# internlm2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/internlm2.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for internlm2, including architecture wrappers and weight loading logic. / 面向推理的 internlm2 vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-50)
```python
from collections.abc import Iterable
from functools import partial
from itertools import islice
from typing import Any

import torch
from torch import nn
from transformers import PretrainedConfig

from vllm.compilation.decorators import support_torch_compile
from vllm.config import CacheConfig, VllmConfig
from vllm.distributed import (
    get_pp_group,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
    split_tensor_along_last_dim,
    tensor_model_parallel_all_gather,
)
from vllm.model_executor.layers.activation import SiluAndMul
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.linear import (
    MergedColumnParallelLinear,
    QKVParallelLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.logits_processor import LogitsProcessor
from vllm.model_executor.layers.pooler.tokwise import pooler_for_token_classify
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.rotary_embedding import get_rope
from vllm.model_executor.layers.vocab_parallel_embedding import (
    ParallelLMHead,
    VocabParallelEmbedding,
)
from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.sequence import IntermediateTensors

from .interfaces import SupportsLoRA, SupportsPP
from .interfaces_base import default_pooling_type
from .utils import (
    StageMissingLayer,
    is_pp_missing_parameter,
    make_empty_intermediate_tensors_factory,
    make_layers,
    maybe_prefix,
    no_init_weights,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `InternLM2MLP` (lines 53-87)
```python
class InternLM2MLP(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
        )
        self.w2 = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.w2",
        )
        if hidden_act != "silu":
```
**EN:** Class `InternLM2MLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `InternLM2MLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `InternLM2MLP.__init__` (lines 54-81)
```python
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
        )
        self.w2 = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.w2",
        )
        if hidden_act != "silu":
            raise ValueError(
                f"Unsupported activation: {hidden_act}. Only silu is supported for now."
            )
        self.act_fn = SiluAndMul()
```
**EN:** Method `InternLM2MLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternLM2MLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternLM2MLP.forward` (lines 83-87)
```python
    def forward(self, x):
        gate_up, _ = self.gate_up_proj(x)
        x = self.act_fn(gate_up)
        x, _ = self.w2(x)
        return x
```
**EN:** Method `InternLM2MLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `InternLM2MLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `InternLM2Attention` (lines 90-192)
```python
class InternLM2Attention(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        rope_parameters: dict[str, Any] | None = None,
        max_position_embeddings: int = 8192,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        self.tp_size = get_tensor_model_parallel_world_size()
        self.tp_rank = get_tensor_model_parallel_rank()
        self.total_num_heads = num_heads
        assert self.total_num_heads % self.tp_size == 0
        self.num_heads = self.total_num_heads // self.tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= self.tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % self.tp_size == 0
        else:
```
**EN:** Class `InternLM2Attention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, split_qkv, forward.
**CN:** 类 `InternLM2Attention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, split_qkv, forward。

### Method `InternLM2Attention.__init__` (lines 91-156)
```python
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        rope_parameters: dict[str, Any] | None = None,
        max_position_embeddings: int = 8192,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        self.tp_size = get_tensor_model_parallel_world_size()
        self.tp_rank = get_tensor_model_parallel_rank()
        self.total_num_heads = num_heads
        assert self.total_num_heads % self.tp_size == 0
        self.num_heads = self.total_num_heads // self.tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= self.tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % self.tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert self.tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // self.tp_size)
        self.head_dim = hidden_size // self.total_num_heads
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.key_value_groups = int(self.num_heads / self.num_kv_heads)
        self.scaling = self.head_dim**-0.5
        self.max_position_embeddings = max_position_embeddings

        self.wqkv = QKVParallelLinear(
            hidden_size,
            self.head_dim,
            self.total_num_heads,
            self.total_num_kv_heads,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.wqkv",
        )
        self.wo = RowParallelLinear(
            self.total_num_heads * self.head_dim,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.wo",
        )

        self.rotary_emb = get_rope(
            self.head_dim,
            max_position=max_position_embeddings,
            rope_parameters=rope_parameters,
        )
        self.attn = Attention(
            self.num_heads,
            self.head_dim,
# ... truncated for analysis ...
```
**EN:** Method `InternLM2Attention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternLM2Attention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternLM2Attention.split_qkv` (lines 158-180)
```python
    def split_qkv(self, qkv: torch.Tensor):
        seq_len = qkv.shape[0]
        if self.tp_size > 1:
            qkv_map = [self.q_size, self.kv_size, self.kv_size] * self.tp_size
            qkv = tensor_model_parallel_all_gather(qkv)
            qkv = torch.split(qkv, qkv_map, dim=-1)
            qkv = qkv[::3] + qkv[1::3] + qkv[2::3]
            qkv = torch.cat(qkv, dim=-1)

        qkv = qkv.view(
            seq_len, self.total_num_kv_heads, self.key_value_groups + 2, self.head_dim
        )
        q, k, v = torch.split(qkv, [self.key_value_groups, 1, 1], dim=-2)
        q = q.reshape(seq_len, self.q_size * self.tp_size)
        k = k.reshape(seq_len, self.kv_size * self.tp_size)
        v = v.reshape(seq_len, self.kv_size * self.tp_size)

        if self.tp_size > 1:
            splitter = partial(split_tensor_along_last_dim, num_partitions=self.tp_size)
            q = splitter(q)[self.tp_rank]
            k = splitter(k)[self.tp_rank]
            v = splitter(v)[self.tp_rank]
        return q, k, v
```
**EN:** Method `InternLM2Attention.split_qkv` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `InternLM2Attention.split_qkv` 封装了该模块中的一段可复用核心逻辑。

### Method `InternLM2Attention.forward` (lines 182-192)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        qkv, _ = self.wqkv(hidden_states)
        q, k, v = self.split_qkv(qkv)
        q, k = self.rotary_emb(positions, q, k)
        attn_output = self.attn(q, k, v)
        output, _ = self.wo(attn_output)
        return output
```
**EN:** Method `InternLM2Attention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `InternLM2Attention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `InternLMDecoderLayer` (lines 195-246)
```python
class InternLMDecoderLayer(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        max_position_embeddings = getattr(config, "max_position_embeddings", 8192)
        self.attention = InternLM2Attention(
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=config.num_key_value_heads,
            rope_parameters=config.rope_parameters,
            max_position_embeddings=max_position_embeddings,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.attention",
        )
        self.feed_forward = InternLM2MLP(
            hidden_size=self.hidden_size,
            intermediate_size=config.intermediate_size,
            hidden_act=config.hidden_act,
```
**EN:** Class `InternLMDecoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `InternLMDecoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `InternLMDecoderLayer.__init__` (lines 196-224)
```python
    def __init__(
        self,
        config: PretrainedConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        max_position_embeddings = getattr(config, "max_position_embeddings", 8192)
        self.attention = InternLM2Attention(
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=config.num_key_value_heads,
            rope_parameters=config.rope_parameters,
            max_position_embeddings=max_position_embeddings,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.attention",
        )
        self.feed_forward = InternLM2MLP(
            hidden_size=self.hidden_size,
            intermediate_size=config.intermediate_size,
            hidden_act=config.hidden_act,
            quant_config=quant_config,
            prefix=f"{prefix}.feed_forward",
        )
        self.attention_norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.ffn_norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
```
**EN:** Method `InternLMDecoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternLMDecoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternLMDecoderLayer.forward` (lines 226-246)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        residual: torch.Tensor | None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        # Self Attention
        if residual is None:
            residual = hidden_states
            hidden_states = self.attention_norm(hidden_states)
        else:
            hidden_states, residual = self.attention_norm(hidden_states, residual)
        hidden_states = self.attention(
            positions=positions,
            hidden_states=hidden_states,
        )

        # Fully Connected
        hidden_states, residual = self.ffn_norm(hidden_states, residual)
        hidden_states = self.feed_forward(hidden_states)
        return hidden_states, residual
```
**EN:** Method `InternLMDecoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `InternLMDecoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `InternLM2Model` (lines 250-309)
```python
@support_torch_compile
class InternLM2Model(nn.Module):
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        layer_type: type[InternLMDecoderLayer] = InternLMDecoderLayer,
    ):
        super().__init__()

        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.config = config
        self.vocab_size = config.vocab_size
        self.tok_embeddings = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
        )
        self.start_layer, self.end_layer, self.layers = make_layers(
            config.num_hidden_layers,
            lambda prefix: layer_type(
                config, cache_config, quant_config, prefix=prefix
```
**EN:** Class `InternLM2Model` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, embed_input_ids, forward.
**CN:** 类 `InternLM2Model` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, embed_input_ids, forward。

### Method `InternLM2Model.__init__` (lines 251-280)
```python
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        layer_type: type[InternLMDecoderLayer] = InternLMDecoderLayer,
    ):
        super().__init__()

        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.config = config
        self.vocab_size = config.vocab_size
        self.tok_embeddings = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
        )
        self.start_layer, self.end_layer, self.layers = make_layers(
            config.num_hidden_layers,
            lambda prefix: layer_type(
                config, cache_config, quant_config, prefix=prefix
            ),
            prefix=f"{prefix}.layers",
        )
        self.norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.make_empty_intermediate_tensors = make_empty_intermediate_tensors_factory(
            ["hidden_states", "residual"], config.hidden_size
        )
```
**EN:** Method `InternLM2Model.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternLM2Model.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternLM2Model.embed_input_ids` (lines 282-283)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.tok_embeddings(input_ids)
```
**EN:** Method `InternLM2Model.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `InternLM2Model.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `InternLM2Model.forward` (lines 285-309)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor | IntermediateTensors:
        if get_pp_group().is_first_rank:
            if inputs_embeds is not None:
                hidden_states = inputs_embeds
            else:
                hidden_states = self.embed_input_ids(input_ids)
            residual = None
        else:
            assert intermediate_tensors is not None
            hidden_states = intermediate_tensors["hidden_states"]
            residual = intermediate_tensors["residual"]
        for layer in islice(self.layers, self.start_layer, self.end_layer):
            hidden_states, residual = layer(positions, hidden_states, residual)
        if not get_pp_group().is_last_rank:
            return IntermediateTensors(
                {"hidden_states": hidden_states, "residual": residual}
            )
        hidden_states, _ = self.norm(hidden_states, residual)
        return hidden_states
```
**EN:** Method `InternLM2Model.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `InternLM2Model.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `InternLM2ForCausalLM` (lines 312-404)
```python
class InternLM2ForCausalLM(nn.Module, SupportsPP, SupportsLoRA):
    packed_modules_mapping = {
        "wqkv": ["wqkv"],
        "gate_up_proj": ["w1", "w3"],
    }

    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        model_type: type[InternLM2Model] = InternLM2Model,
    ):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        self.config = config
        self.quant_config = quant_config

        self.model = model_type(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )
        self.output = ParallelLMHead(
            config.vocab_size,
```
**EN:** Class `InternLM2ForCausalLM` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsPP, SupportsLoRA. Key methods include __init__, embed_input_ids, forward, compute_logits, load_weights.
**CN:** 类 `InternLM2ForCausalLM` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsPP、SupportsLoRA。 关键方法包括 __init__, embed_input_ids, forward, compute_logits, load_weights。

### Method `InternLM2ForCausalLM.__init__` (lines 318-346)
```python
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        model_type: type[InternLM2Model] = InternLM2Model,
    ):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        self.config = config
        self.quant_config = quant_config

        self.model = model_type(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )
        self.output = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "output"),
        )
        if self.config.tie_word_embeddings:
            self.output.weight = self.model.tok_embeddings.weight
        self.logits_processor = LogitsProcessor(config.vocab_size)
        self.make_empty_intermediate_tensors = (
            self.model.make_empty_intermediate_tensors
        )
```
**EN:** Method `InternLM2ForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternLM2ForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternLM2ForCausalLM.embed_input_ids` (lines 348-349)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.model.embed_input_ids(input_ids)
```
**EN:** Method `InternLM2ForCausalLM.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `InternLM2ForCausalLM.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `InternLM2ForCausalLM.forward` (lines 351-361)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor:
        hidden_states = self.model(
            input_ids, positions, intermediate_tensors, inputs_embeds
        )
        return hidden_states
```
**EN:** Method `InternLM2ForCausalLM.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `InternLM2ForCausalLM.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Method `InternLM2ForCausalLM.compute_logits` (lines 363-368)
```python
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        logits = self.logits_processor(self.output, hidden_states)
        return logits
```
**EN:** Method `InternLM2ForCausalLM.compute_logits` converts hidden states into prediction-ready outputs, often by applying a head, post-processing, or sampling logic.
**CN:** Method `InternLM2ForCausalLM.compute_logits` 将隐藏状态转成可预测输出，通常会经过输出头、后处理或采样逻辑。

### Class `InternLM2ForRewardModel` (lines 408-459)
```python
@default_pooling_type(tok_pooling_type="ALL")
class InternLM2ForRewardModel(InternLM2ForCausalLM):
    is_pooling_model = True

    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        model_type: type[InternLM2Model] = InternLM2Model,
    ):
        with no_init_weights(
            self,
            lambda mod: StageMissingLayer("output", mod),
            targets=(LogitsProcessor, ParallelLMHead),
        ):
            super().__init__(
                vllm_config=vllm_config,
                prefix=prefix,
                model_type=model_type,
            )

        config = vllm_config.model_config.hf_config
        self.head_dtype = vllm_config.model_config.head_dtype
```
**EN:** Class `InternLM2ForRewardModel` organizes related behavior for this model family or helper component. It inherits from InternLM2ForCausalLM. Key methods include __init__, forward.
**CN:** 类 `InternLM2ForRewardModel` 用于组织该模型族或辅助组件的相关行为。 它继承自 InternLM2ForCausalLM。 关键方法包括 __init__, forward。

### Method `InternLM2ForRewardModel.__init__` (lines 411-445)
```python
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        model_type: type[InternLM2Model] = InternLM2Model,
    ):
        with no_init_weights(
            self,
            lambda mod: StageMissingLayer("output", mod),
            targets=(LogitsProcessor, ParallelLMHead),
        ):
            super().__init__(
                vllm_config=vllm_config,
                prefix=prefix,
                model_type=model_type,
            )

        config = vllm_config.model_config.hf_config
        self.head_dtype = vllm_config.model_config.head_dtype

        self.v_head = RowParallelLinear(
            config.hidden_size,
            1,
            bias=False,
            input_is_parallel=False,
            params_dtype=self.head_dtype,
            prefix=maybe_prefix(prefix, "v_head"),
            return_bias=False,
        )

        pooler_config = vllm_config.model_config.pooler_config
        assert pooler_config is not None

        self.pooler = pooler_for_token_classify(pooler_config)
```
**EN:** Method `InternLM2ForRewardModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternLM2ForRewardModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternLM2ForRewardModel.forward` (lines 447-459)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor | IntermediateTensors:
        hidden_states = self.model(
            input_ids, positions, intermediate_tensors, inputs_embeds
        )
        hidden_states = hidden_states.to(self.head_dtype)
        logits = self.v_head(hidden_states)
        return logits
```
**EN:** Method `InternLM2ForRewardModel.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `InternLM2ForRewardModel.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

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
- **Standard library / 标准库**: `from collections.abc import Iterable`, `from functools import partial`, `from itertools import islice`, `from typing import Any`
- **Third-party / 第三方**: `import torch`, `from torch import nn`, `from transformers import PretrainedConfig`
- **vLLM internal / vLLM 内部依赖**: `from vllm.compilation.decorators import support_torch_compile`, `from vllm.config import CacheConfig, VllmConfig`, `from vllm.distributed import (`, `from vllm.model_executor.layers.activation import SiluAndMul`, `from vllm.model_executor.layers.attention import Attention`, `from vllm.model_executor.layers.layernorm import RMSNorm`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.layers.pooler.tokwise import pooler_for_token_classify`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.rotary_embedding import get_rope`, `from vllm.model_executor.layers.vocab_parallel_embedding import (`
