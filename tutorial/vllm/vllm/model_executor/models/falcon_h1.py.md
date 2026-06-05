# falcon_h1.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/falcon_h1.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for falcon_h1, including architecture wrappers and weight loading logic. / 面向推理的 falcon_h1 vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 5-59)
```python
from collections.abc import Iterable
from itertools import islice

import torch
from torch import nn
from transformers import FalconH1Config

from vllm.compilation.decorators import support_torch_compile
from vllm.config import CacheConfig, ModelConfig, VllmConfig
from vllm.distributed import get_tensor_model_parallel_world_size
from vllm.distributed.parallel_state import get_pp_group
from vllm.model_executor.layers.activation import SiluAndMul
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.linear import (
    MergedColumnParallelLinear,
    QKVParallelLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.logits_processor import LogitsProcessor
from vllm.model_executor.layers.mamba.mamba_mixer2 import MambaMixer2
from vllm.model_executor.layers.mamba.mamba_utils import (
    MambaStateCopyFunc,
    MambaStateCopyFuncCalculator,
    MambaStateDtypeCalculator,
    MambaStateShapeCalculator,
)
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.rotary_embedding import get_rope
from vllm.model_executor.layers.vocab_parallel_embedding import (
    ParallelLMHead,
    VocabParallelEmbedding,
)
from vllm.model_executor.model_loader.weight_utils import (
    default_weight_loader,
    maybe_remap_kv_scale_name,
)
from vllm.sequence import IntermediateTensors
from vllm.transformers_utils.config import set_default_rope_theta

from .interfaces import (
    HasInnerState,
    IsHybrid,
    SupportsLoRA,
    SupportsMambaPrefixCaching,
    SupportsPP,
)
from .utils import (
    AutoWeightsLoader,
    PPMissingLayer,
    is_pp_missing_parameter,
    make_empty_intermediate_tensors_factory,
    make_layers,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `FalconH1MLP` (lines 62-101)
```python
class FalconH1MLP(nn.Module):
    def __init__(
        self,
        config: FalconH1Config,
        quant_config: QuantizationConfig | None = None,
        bias: bool = False,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
            input_size=config.hidden_size,
            output_sizes=[config.intermediate_size] * 2,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
        )
        self.down_proj = RowParallelLinear(
            input_size=config.intermediate_size,
            output_size=config.hidden_size,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.down_proj",
        )
        self.tp_size = get_tensor_model_parallel_world_size()
        self.intermediate_size = config.intermediate_size
```
**EN:** Class `FalconH1MLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `FalconH1MLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `FalconH1MLP.__init__` (lines 63-93)
```python
    def __init__(
        self,
        config: FalconH1Config,
        quant_config: QuantizationConfig | None = None,
        bias: bool = False,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
            input_size=config.hidden_size,
            output_sizes=[config.intermediate_size] * 2,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
        )
        self.down_proj = RowParallelLinear(
            input_size=config.intermediate_size,
            output_size=config.hidden_size,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.down_proj",
        )
        self.tp_size = get_tensor_model_parallel_world_size()
        self.intermediate_size = config.intermediate_size
        self.gate_multiplier, self.down_multiplier = config.mlp_multipliers
        if config.hidden_act != "silu":
            raise ValueError(
                f"Unsupported activation: {config.hidden_act}. "
                "Only silu is supported for now."
            )
        self.act_fn = SiluAndMul()
```
**EN:** Method `FalconH1MLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `FalconH1MLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `FalconH1MLP.forward` (lines 95-101)
```python
    def forward(self, x):
        x, _ = self.gate_up_proj(x)
        x[:, : self.intermediate_size // self.tp_size] *= self.gate_multiplier
        x = self.act_fn(x)
        x, _ = self.down_proj(x)
        x = x * self.down_multiplier
        return x
```
**EN:** Method `FalconH1MLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `FalconH1MLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `FalconH1SSMDecoderLayer` (lines 104-212)
```python
class FalconH1SSMDecoderLayer(nn.Module):
    def __init__(
        self,
        config: FalconH1Config,
        model_config: ModelConfig | None = None,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.tp_size = get_tensor_model_parallel_world_size()

        self.d_ssm = (
            int(config.mamba_expand * config.hidden_size)
            if config.mamba_d_ssm is None
            else config.mamba_d_ssm
        )

        self.mamba = MambaMixer2(
            hidden_size=config.hidden_size,
            ssm_state_size=config.mamba_d_state,
            conv_kernel_size=config.mamba_d_conv,
            intermediate_size=self.d_ssm,
            use_conv_bias=config.mamba_conv_bias,
```
**EN:** Class `FalconH1SSMDecoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, _init_mup_vector, forward.
**CN:** 类 `FalconH1SSMDecoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, _init_mup_vector, forward。

### Method `FalconH1SSMDecoderLayer.__init__` (lines 105-144)
```python
    def __init__(
        self,
        config: FalconH1Config,
        model_config: ModelConfig | None = None,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.tp_size = get_tensor_model_parallel_world_size()

        self.d_ssm = (
            int(config.mamba_expand * config.hidden_size)
            if config.mamba_d_ssm is None
            else config.mamba_d_ssm
        )

        self.mamba = MambaMixer2(
            hidden_size=config.hidden_size,
            ssm_state_size=config.mamba_d_state,
            conv_kernel_size=config.mamba_d_conv,
            intermediate_size=self.d_ssm,
            use_conv_bias=config.mamba_conv_bias,
            use_bias=config.mamba_proj_bias,
            n_groups=config.mamba_n_groups,
            num_heads=config.mamba_n_heads,
            head_dim=config.mamba_d_head,
            rms_norm_eps=config.rms_norm_eps,
            activation=config.hidden_act,
            model_config=model_config,
            cache_config=cache_config,
            quant_config=quant_config,
            use_rms_norm=config.mamba_rms_norm,
            prefix=f"{prefix}.mixer",
        )
        # n_groups is overridden later by `MambaMixer2`
        self.groups_time_state_size = self.mamba.n_groups * config.mamba_d_state
        self.zxbcdt_multipliers = config.ssm_multipliers
        self._init_mup_vector()
```
**EN:** Method `FalconH1SSMDecoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `FalconH1SSMDecoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `FalconH1SSMDecoderLayer.forward` (lines 202-212)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        residual: torch.Tensor | None,
        **kwargs,
    ):
        output = self.mamba(
            hidden_states,
            mup_vector=self.mup_vector,
        )
        return output, residual
```
**EN:** Method `FalconH1SSMDecoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `FalconH1SSMDecoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `FalconH1AttentionDecoderLayer` (lines 215-316)
```python
class FalconH1AttentionDecoderLayer(nn.Module):
    def __init__(
        self,
        config: FalconH1Config,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        set_default_rope_theta(config, default_theta=1e11)
        max_position_embeddings = getattr(config, "max_position_embeddings", 8192)
        self.hidden_size = config.hidden_size
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = config.num_attention_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = config.num_key_value_heads
        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert tp_size % self.total_num_kv_heads == 0
```
**EN:** Class `FalconH1AttentionDecoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, self_attention, forward.
**CN:** 类 `FalconH1AttentionDecoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, self_attention, forward。

### Method `FalconH1AttentionDecoderLayer.__init__` (lines 216-288)
```python
    def __init__(
        self,
        config: FalconH1Config,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        set_default_rope_theta(config, default_theta=1e11)
        max_position_embeddings = getattr(config, "max_position_embeddings", 8192)
        self.hidden_size = config.hidden_size
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = config.num_attention_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = config.num_key_value_heads
        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        self.head_dim = (
            config.hidden_size // self.total_num_heads
            if getattr(config, "head_dim", None) is None
            else config.head_dim
        )
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5
        self.max_position_embeddings = max_position_embeddings

        rotary_dim = getattr(config, "attn_rotary_emb", self.head_dim)
        config.rope_parameters["partial_rotary_factor"] = rotary_dim / self.head_dim

        self.rotary_emb = get_rope(
            head_size=self.head_dim,
            max_position=max_position_embeddings,
            rope_parameters=config.rope_parameters,
            is_neox_style=True,
            dtype=None,  # see impl of get_rope
        )

        self.qkv_proj = QKVParallelLinear(
            config.hidden_size,
            self.head_dim,
            self.total_num_heads,
            self.total_num_kv_heads,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.qkv_proj",
        )
        self.o_proj = RowParallelLinear(
            self.total_num_heads * self.head_dim,
            config.hidden_size,
            bias=False,
            quant_config=quant_config,
# ... truncated for analysis ...
```
**EN:** Method `FalconH1AttentionDecoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `FalconH1AttentionDecoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `FalconH1AttentionDecoderLayer.self_attention` (lines 290-303)
```python
    def self_attention(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        **kwargs,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        k = k * self.key_multiplier

        q, k = self.rotary_emb(positions, q, k)
        attn_output = self.attn(q, k, v)
        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** Method `FalconH1AttentionDecoderLayer.self_attention` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FalconH1AttentionDecoderLayer.self_attention` 封装了该模块中的一段可复用核心逻辑。

### Method `FalconH1AttentionDecoderLayer.forward` (lines 305-316)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        residual: torch.Tensor | None,
        **kwargs,
    ):
        hidden_states = self.self_attention(
            positions=positions,
            hidden_states=hidden_states,
        )
        return hidden_states, residual
```
**EN:** Method `FalconH1AttentionDecoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `FalconH1AttentionDecoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `FalconH1ParallelHybrid` (lines 319-415)
```python
class FalconH1ParallelHybrid(nn.Module):
    """
    A hybrid decoder layer for FalconH1 where the input is processed
    in parallel through both the self-attention branch and the SSM (Mamba)
    branch. Their outputs are then summed to produce the final hidden state.

    This layer uses:
      - FalconH1AttentionDecoderLayer for the multi-head self-attention branch.
      - FalconH1SSMDecoderLayer for the state-space (Mamba) branch.
    """

    def __init__(
        self,
        config: FalconH1Config,
        layer_idx: int,
        model_config: ModelConfig | None = None,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        # Instantiate the attention branch
        self.self_attn = FalconH1AttentionDecoderLayer(
            config=config,
```
**EN:** Class `FalconH1ParallelHybrid` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `FalconH1ParallelHybrid` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `FalconH1ParallelHybrid.__init__` (lines 330-373)
```python
    def __init__(
        self,
        config: FalconH1Config,
        layer_idx: int,
        model_config: ModelConfig | None = None,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        # Instantiate the attention branch
        self.self_attn = FalconH1AttentionDecoderLayer(
            config=config,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=prefix,
        )

        # In V1 all attention/ssm layers must have
        # different index in prefix
        ssm_layer_idx = config.num_hidden_layers + layer_idx
        ssm_prefix = prefix.split(".")[0] + f".{ssm_layer_idx}"

        # Instantiate the SSM branch
        self.mamba = FalconH1SSMDecoderLayer(
            config=config,
            model_config=model_config,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=ssm_prefix,
        )
        self.ssm_out_multiplier = config.ssm_out_multiplier
        self.ssm_in_multiplier = config.ssm_in_multiplier

        self.attention_in_multiplier = config.attention_in_multiplier
        self.attn_out_multiplier = config.attention_out_multiplier

        self.feed_forward = FalconH1MLP(
            config, quant_config=quant_config, prefix=f"{prefix}.feed_forward"
        )

        self.input_layernorm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.pre_ff_layernorm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
```
**EN:** Method `FalconH1ParallelHybrid.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `FalconH1ParallelHybrid.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `FalconH1ParallelHybrid.forward` (lines 375-415)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        **kwargs,
    ):
        residual = hidden_states
        hidden_states = self.input_layernorm(hidden_states)
        # Process input through the attention branch.
        # FalconH1AttentionDecoderLayer expects positions, hidden_states,
        # kv_cache, attn_metadata, and residual.
        attn_hidden, _ = self.self_attn(
            positions=positions,
            hidden_states=hidden_states * self.attention_in_multiplier,
            residual=residual,
            **kwargs,
        )

        # Process input through the SSM branch.
        # FalconH1SSMDecoderLayer expects hidden_states, attn_metadata,
        # residual, and sequence_idx.
        ssm_hidden, _ = self.mamba(
            hidden_states=hidden_states * self.ssm_in_multiplier,
            residual=residual,
            **kwargs,
        )
        # Sum the outputs from both branches.
        # We assume both branches produce outputs of the same
        # dimensionality (config.hidden_size).
        hidden_states = (attn_hidden * self.attn_out_multiplier) + (
            ssm_hidden * self.ssm_out_multiplier
        )
        hidden_states = hidden_states + residual

        # feed-forward
        residual = hidden_states
        hidden_states = self.pre_ff_layernorm(hidden_states)
        hidden_states = self.feed_forward(hidden_states)
        hidden_states = residual + hidden_states

        return hidden_states
```
**EN:** Method `FalconH1ParallelHybrid.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `FalconH1ParallelHybrid.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `FalconH1Model` (lines 419-554)
```python
@support_torch_compile
class FalconH1Model(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config: FalconH1Config = vllm_config.model_config.hf_config
        model_config = vllm_config.model_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.config = config

        self.vocab_size = config.vocab_size

        if get_pp_group().is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                self.vocab_size,
                config.hidden_size,
            )
            self.embedding_multiplier = config.embedding_multiplier
        else:
            self.embed_tokens = PPMissingLayer()
            self.embedding_multiplier = 1.0

        def get_layer(prefix: str):
            layer_idx = int(prefix.rsplit(".", 1)[1])
```
**EN:** Class `FalconH1Model` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, embed_input_ids, forward, load_weights.
**CN:** 类 `FalconH1Model` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, embed_input_ids, forward, load_weights。

### Method `FalconH1Model.__init__` (lines 420-462)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config: FalconH1Config = vllm_config.model_config.hf_config
        model_config = vllm_config.model_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.config = config

        self.vocab_size = config.vocab_size

        if get_pp_group().is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                self.vocab_size,
                config.hidden_size,
            )
            self.embedding_multiplier = config.embedding_multiplier
        else:
            self.embed_tokens = PPMissingLayer()
            self.embedding_multiplier = 1.0

        def get_layer(prefix: str):
            layer_idx = int(prefix.rsplit(".", 1)[1])
            layer_class = FalconH1ParallelHybrid
            return layer_class(
                config,
                layer_idx,
                model_config,
                cache_config,
                quant_config=quant_config,
                prefix=prefix,
            )

        self.start_layer, self.end_layer, self.layers = make_layers(
            config.num_hidden_layers, get_layer, prefix=f"{prefix}.layers"
        )
        self.make_empty_intermediate_tensors = make_empty_intermediate_tensors_factory(
            ["hidden_states", "residual"], config.hidden_size
        )
        if get_pp_group().is_last_rank:
            self.final_layernorm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        else:
            self.final_layernorm = PPMissingLayer()
```
**EN:** Method `FalconH1Model.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `FalconH1Model.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `FalconH1Model.embed_input_ids` (lines 464-465)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embed_tokens(input_ids)
```
**EN:** Method `FalconH1Model.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `FalconH1Model.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `FalconH1Model.forward` (lines 467-497)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor:
        if get_pp_group().is_first_rank:
            if inputs_embeds is not None:
                hidden_states = inputs_embeds * self.embedding_multiplier
            else:
                hidden_states = (
                    self.embed_input_ids(input_ids) * self.embedding_multiplier
                )
        else:
            assert intermediate_tensors is not None
            hidden_states = intermediate_tensors["hidden_states"]

        for layer in islice(self.layers, self.start_layer, self.end_layer):
            hidden_states = layer(
                positions=positions,
                hidden_states=hidden_states,
            )
        if not get_pp_group().is_last_rank:
            return IntermediateTensors(
                {
                    "hidden_states": hidden_states,
                }
            )
        hidden_states = self.final_layernorm(hidden_states)
        return hidden_states
```
**EN:** Method `FalconH1Model.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `FalconH1Model.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Method `FalconH1Model.load_weights` (lines 499-554)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
        ]

        params_dict = dict(self.named_parameters())
        loaded_params: set[str] = set()
        for name, loaded_weight in weights:
            if "rotary_emb.inv_freq" in name:
                continue

            if "A_log" in name:
                name = name.replace("A_log", "A")

            if "mamba" in name:
                name = name.replace("mamba", "mamba.mamba")

            if "scale" in name:
                # Remapping the name of kv-scale.
                name = maybe_remap_kv_scale_name(name, params_dict)
                if name is None:
                    continue

            for param_name, weight_name, shard_id in stacked_params_mapping:
                if weight_name not in name:
                    continue

                name = name.replace(weight_name, param_name)
                # Skip loading extra bias for GPTQ models.
                if name.endswith(".bias") and name not in params_dict:
                    continue
                # Skip layers on other devices.
                if is_pp_missing_parameter(name, self):
                    continue
                param = params_dict[name]
                weight_loader = param.weight_loader
                weight_loader(param, loaded_weight, shard_id)
                break
            else:
                # Skip loading extra bias for GPTQ models.
                if name.endswith(".bias") and name not in params_dict:
                    continue
                if is_pp_missing_parameter(name, self):
                    continue

                param = params_dict[name]
                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                weight_loader(param, loaded_weight)
            loaded_params.add(name)

        return loaded_params
```
**EN:** Method `FalconH1Model.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `FalconH1Model.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `FalconH1ForCausalLM` (lines 557-697)
```python
class FalconH1ForCausalLM(
    nn.Module,
    HasInnerState,
    SupportsLoRA,
    SupportsPP,
    IsHybrid,
    SupportsMambaPrefixCaching,
):
    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
        "gate_up_proj": ["gate_proj", "up_proj"],
    }

    embedding_modules = {
        "embed_tokens": "input_embeddings",
        "lm_head": "output_embeddings",
    }

    @classmethod
    def get_mamba_state_dtype_from_config(
        cls,
        vllm_config: "VllmConfig",
    ) -> tuple[torch.dtype, torch.dtype]:
        return MambaStateDtypeCalculator.mamba2_state_dtype(
            vllm_config.model_config.dtype,
```
**EN:** Class `FalconH1ForCausalLM` is a structural model block in the vLLM execution graph. It inherits from nn.Module, HasInnerState, SupportsLoRA, SupportsPP, IsHybrid, SupportsMambaPrefixCaching. Key methods include get_mamba_state_dtype_from_config, get_mamba_state_shape_from_config, get_mamba_state_copy_func, __init__, embed_input_ids, forward.
**CN:** 类 `FalconH1ForCausalLM` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、HasInnerState、SupportsLoRA、SupportsPP、IsHybrid、SupportsMambaPrefixCaching。 关键方法包括 get_mamba_state_dtype_from_config, get_mamba_state_shape_from_config, get_mamba_state_copy_func, __init__, embed_input_ids, forward。

### Method `FalconH1ForCausalLM.get_mamba_state_dtype_from_config` (lines 576-584)
```python
    @classmethod
    def get_mamba_state_dtype_from_config(
        cls,
        vllm_config: "VllmConfig",
    ) -> tuple[torch.dtype, torch.dtype]:
        return MambaStateDtypeCalculator.mamba2_state_dtype(
            vllm_config.model_config.dtype,
            vllm_config.cache_config.mamba_cache_dtype,
            vllm_config.cache_config.mamba_ssm_cache_dtype,
        )
```
**EN:** Method `FalconH1ForCausalLM.get_mamba_state_dtype_from_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FalconH1ForCausalLM.get_mamba_state_dtype_from_config` 封装了该模块中的一段可复用核心逻辑。

### Method `FalconH1ForCausalLM.get_mamba_state_shape_from_config` (lines 587-618)
```python
    @classmethod
    def get_mamba_state_shape_from_config(
        cls,
        vllm_config: "VllmConfig",
    ) -> tuple[tuple[int, int], tuple[int, int, int]]:
        """Calculate shapes for Mamba's convolutional and state caches.

        Args:
            vllm_config: vLLM config

        Returns:
            Tuple containing:
            - conv_state_shape: Shape for convolutional state cache
            - temporal_state_shape: Shape for state space model cache
        """
        parallel_config = vllm_config.parallel_config
        hf_config = vllm_config.model_config.hf_config

        intermediate_size = (
            int(hf_config.mamba_expand * hf_config.hidden_size)
            if hf_config.mamba_d_ssm is None
            else hf_config.mamba_d_ssm
        )

        return MambaStateShapeCalculator.mamba2_state_shape(
            intermediate_size=intermediate_size,
            tp_world_size=parallel_config.tensor_parallel_size,
            n_groups=hf_config.mamba_n_groups,
            num_heads=hf_config.mamba_n_heads,
            head_dim=hf_config.mamba_d_head,
            state_size=hf_config.mamba_d_state,
            conv_kernel=hf_config.mamba_d_conv,
        )
```
**EN:** Method `FalconH1ForCausalLM.get_mamba_state_shape_from_config` encapsulates a focused piece of reusable logic inside this module. The docstring says: Calculate shapes for Mamba's convolutional and state caches.
**CN:** Method `FalconH1ForCausalLM.get_mamba_state_shape_from_config` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Calculate shapes for Mamba's convolutional and state caches。

### Method `FalconH1ForCausalLM.get_mamba_state_copy_func` (lines 621-622)
```python
    @classmethod
    def get_mamba_state_copy_func(cls) -> tuple[MambaStateCopyFunc, MambaStateCopyFunc]:
        return MambaStateCopyFuncCalculator.mamba2_state_copy_func()
```
**EN:** Method `FalconH1ForCausalLM.get_mamba_state_copy_func` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `FalconH1ForCausalLM.get_mamba_state_copy_func` 封装了该模块中的一段可复用核心逻辑。

### Method `FalconH1ForCausalLM.__init__` (lines 624-662)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        config = vllm_config.model_config.hf_config
        self.vllm_config = vllm_config
        self.model_config = vllm_config.model_config

        scheduler_config = vllm_config.scheduler_config

        self.quant_config = vllm_config.quant_config

        super().__init__()
        self.config = config
        self.scheduler_config = scheduler_config
        self.model = FalconH1Model(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )
        self.tie_word_embeddings = config.tie_word_embeddings

        if get_pp_group().is_last_rank:
            self.lm_head = ParallelLMHead(
                config.vocab_size,
                config.hidden_size,
                prefix=maybe_prefix(prefix, "lm_head"),
            )
            self.lm_head_multiplier = config.lm_head_multiplier
            if self.tie_word_embeddings:
                self.lm_head = self.lm_head.tie_weights(self.model.embed_tokens)
            # Used to track and store by the Mamba cache between steps.

            self.logits_processor = LogitsProcessor(
                config.vocab_size,
                config.vocab_size,
                scale=config.lm_head_multiplier,
            )
        else:
            self.lm_head = PPMissingLayer()

        self.make_empty_intermediate_tensors = (
            self.model.make_empty_intermediate_tensors
        )
```
**EN:** Method `FalconH1ForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `FalconH1ForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

## Key Concepts / 关键概念
- **Attention / 注意力**
  - **EN:** Attention layers model token-to-token or token-to-feature interactions inside the architecture.
  - **CN:** 注意力层负责建模 token 与 token，或 token 与特征之间的交互。
- **LoRA support / LoRA 支持**
  - **EN:** The file is aware of low-rank adaptation interfaces or compatible parameter layouts.
  - **CN:** 该文件考虑了 LoRA 接口或兼容的低秩参数布局。
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from collections.abc import Iterable`, `from itertools import islice`
- **Third-party / 第三方**: `import torch`, `from torch import nn`, `from transformers import FalconH1Config`
- **vLLM internal / vLLM 内部依赖**: `from vllm.compilation.decorators import support_torch_compile`, `from vllm.config import CacheConfig, ModelConfig, VllmConfig`, `from vllm.distributed import get_tensor_model_parallel_world_size`, `from vllm.distributed.parallel_state import get_pp_group`, `from vllm.model_executor.layers.activation import SiluAndMul`, `from vllm.model_executor.layers.attention import Attention`, `from vllm.model_executor.layers.layernorm import RMSNorm`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.layers.mamba.mamba_mixer2 import MambaMixer2`, `from vllm.model_executor.layers.mamba.mamba_utils import (`, `from vllm.model_executor.layers.quantization import QuantizationConfig`
- **Module note / 模块说明**: **EN:** Inference-only FalconH1 model. **CN:** 模块文档字符串给出的原始说明是：Inference-only FalconH1 model.。
