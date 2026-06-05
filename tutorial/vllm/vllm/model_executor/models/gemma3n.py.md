# gemma3n.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/gemma3n.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for gemma3n, including architecture wrappers and weight loading logic. / 面向推理的 gemma3n vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 18-61)
```python
from collections.abc import Iterable

import torch
from torch import nn
from transformers.models.gemma3n.configuration_gemma3n import Gemma3nTextConfig

from vllm.compilation.decorators import support_torch_compile
from vllm.config import CacheConfig, VllmConfig
from vllm.distributed import get_tensor_model_parallel_world_size
from vllm.forward_context import get_forward_context
from vllm.logger import init_logger
from vllm.model_executor.layers.activation import (
    _ACTIVATION_REGISTRY,
    GeluAndMul,
    GeluAndMulSparse,
)
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.linear import (
    ColumnParallelLinear,
    MergedColumnParallelLinear,
    QKVParallelLinear,
    ReplicatedLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.logits_processor import LogitsProcessor
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.rotary_embedding import get_rope
from vllm.model_executor.layers.vocab_parallel_embedding import VocabParallelEmbedding
from vllm.model_executor.model_loader.weight_utils import (
    default_weight_loader,
    maybe_remap_kv_scale_name,
)
from vllm.sequence import IntermediateTensors
from vllm.v1.attention.backends.utils import KVSharingFastPrefillMetadata

from .interfaces import SupportsQuant
from .utils import (
    AutoWeightsLoader,
    extract_layer_index,
    is_pp_missing_parameter,
    make_layers,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 65-65)
```python
EPS = torch.tensor(torch.finfo().min)
```
**EN:** This block defines EPS, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 EPS，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Class `Gemma3nAltUp` (lines 68-186)
```python
class Gemma3nAltUp(nn.Module):
    """Alternating updates (Altup)
    The AltUp module wraps transformer layers. The `predict` step modifies the
    input to the transformer layer, and the `correct` step propagates the output
    of the transformer layer to the sparsely updated dimensions.
    See more in the research paper:
    https://proceedings.neurips.cc/paper_files/paper/2023/file/f2059277ac6ce66e7e5543001afa8bb5-Paper-Conference.pdf
    """

    def __init__(
        self,
        hidden_size: int,
        rms_norm_eps: float,
        altup_num_inputs: int,
        altup_coef_clip: float,
        altup_active_idx: int,
        quant_config: QuantizationConfig,
        prefix: str,
    ):
        super().__init__()

        self.altup_num_inputs = altup_num_inputs
        self.altup_active_idx = altup_active_idx
        self.altup_coef_clip = altup_coef_clip
```
**EN:** Class `Gemma3nAltUp` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, _compute_router_modalities, scale_corrected_output, predict, correct.
**CN:** 类 `Gemma3nAltUp` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, _compute_router_modalities, scale_corrected_output, predict, correct。

### Method `Gemma3nAltUp.__init__` (lines 77-126)
```python
    def __init__(
        self,
        hidden_size: int,
        rms_norm_eps: float,
        altup_num_inputs: int,
        altup_coef_clip: float,
        altup_active_idx: int,
        quant_config: QuantizationConfig,
        prefix: str,
    ):
        super().__init__()

        self.altup_num_inputs = altup_num_inputs
        self.altup_active_idx = altup_active_idx
        self.altup_coef_clip = altup_coef_clip

        self.correction_coefs = ReplicatedLinear(
            altup_num_inputs,
            altup_num_inputs,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.correction_coefs",
            return_bias=False,
        )
        self.prediction_coefs = ReplicatedLinear(
            altup_num_inputs,
            altup_num_inputs**2,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.prediction_coefs",
            return_bias=False,
        )
        self.modality_router = ReplicatedLinear(
            hidden_size,
            altup_num_inputs,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.modality_router",
            return_bias=False,
        )
        self.router_norm = RMSNorm(
            hidden_size=hidden_size,
            eps=rms_norm_eps,
        )
        self.router_input_scale = torch.tensor(
            hidden_size**-1.0, dtype=self.modality_router.weight.dtype
        )
        self.correct_output_scale = nn.Parameter(
            torch.zeros(hidden_size, dtype=torch.float32)
        )
```
**EN:** Method `Gemma3nAltUp.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma3nAltUp.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma3nAltUp.scale_corrected_output` (lines 133-136)
```python
    def scale_corrected_output(self, corrected: torch.Tensor) -> torch.Tensor:
        return (
            corrected.type_as(self.correct_output_scale) * self.correct_output_scale
        ).type_as(corrected)
```
**EN:** Method `Gemma3nAltUp.scale_corrected_output` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Gemma3nAltUp.scale_corrected_output` 封装了该模块中的一段可复用核心逻辑。

### Class `Gemma3nLaurelBlock` (lines 189-228)
```python
class Gemma3nLaurelBlock(nn.Module):
    """Learned Augmented Residual Layer"""

    def __init__(
        self,
        hidden_size: int,
        laurel_rank: int,
        rms_norm_eps: float,
        *,
        quant_config: QuantizationConfig | None = None,
        prefix: str,
    ) -> None:
        super().__init__()

        self.linear_left = ColumnParallelLinear(
            hidden_size,
            laurel_rank,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.linear_left",
            return_bias=False,
        )
        self.linear_right = RowParallelLinear(
            laurel_rank,
            hidden_size,
```
**EN:** Class `Gemma3nLaurelBlock` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Gemma3nLaurelBlock` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Gemma3nLaurelBlock.__init__` (lines 192-222)
```python
    def __init__(
        self,
        hidden_size: int,
        laurel_rank: int,
        rms_norm_eps: float,
        *,
        quant_config: QuantizationConfig | None = None,
        prefix: str,
    ) -> None:
        super().__init__()

        self.linear_left = ColumnParallelLinear(
            hidden_size,
            laurel_rank,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.linear_left",
            return_bias=False,
        )
        self.linear_right = RowParallelLinear(
            laurel_rank,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.linear_right",
            return_bias=False,
        )
        self.post_laurel_norm = RMSNorm(
            hidden_size=hidden_size,
            eps=rms_norm_eps,
        )
```
**EN:** Method `Gemma3nLaurelBlock.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma3nLaurelBlock.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma3nLaurelBlock.forward` (lines 224-228)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        laurel_x = self.linear_left(x)
        laurel_x = self.linear_right(laurel_x)
        normed_laurel_x = self.post_laurel_norm(laurel_x)
        return x + normed_laurel_x
```
**EN:** Method `Gemma3nLaurelBlock.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Gemma3nLaurelBlock.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Gemma3nMLP` (lines 231-275)
```python
class Gemma3nMLP(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_activation: str,
        activation_sparsity: float = 0.0,
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
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.down_proj",
        )
```
**EN:** Class `Gemma3nMLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Gemma3nMLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Gemma3nMLP.__init__` (lines 232-269)
```python
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_activation: str,
        activation_sparsity: float = 0.0,
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
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.down_proj",
        )
        if hidden_activation != "gelu_pytorch_tanh":
            raise ValueError(
                "Gemma3 uses `gelu_pytorch_tanh` as the hidden activation "
                "function. Please set `hidden_act` and `hidden_activation` to "
                "`gelu_pytorch_tanh`."
            )

        self.act_fn = (
            GeluAndMulSparse(
                activation_sparsity=activation_sparsity, approximate="tanh"
            )
            if activation_sparsity > 0.0
            else GeluAndMul(approximate="tanh")
        )
```
**EN:** Method `Gemma3nMLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma3nMLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma3nMLP.forward` (lines 271-275)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        gate_up, _ = self.gate_up_proj(x)
        x = self.act_fn(gate_up)
        x, _ = self.down_proj(x)
        return x
```
**EN:** Method `Gemma3nMLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Gemma3nMLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Gemma3nAttention` (lines 278-427)
```python
class Gemma3nAttention(nn.Module):
    def __init__(
        self,
        config: Gemma3nTextConfig,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        head_dim: int,
        max_position_embeddings: int,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.hidden_size = hidden_size
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = num_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % tp_size == 0
```
**EN:** Class `Gemma3nAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Gemma3nAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Gemma3nAttention.__init__` (lines 279-402)
```python
    def __init__(
        self,
        config: Gemma3nTextConfig,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        head_dim: int,
        max_position_embeddings: int,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.hidden_size = hidden_size
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = num_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        self.head_dim = head_dim
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim

        self.qkv_proj = QKVParallelLinear(
            hidden_size,
            self.head_dim,
            self.total_num_heads,
            self.total_num_kv_heads,
            bias=config.attention_bias,
            quant_config=quant_config,
            prefix=f"{prefix}.qkv_proj",
        )
        self.o_proj = RowParallelLinear(
            self.total_num_heads * self.head_dim,
            hidden_size,
            bias=config.attention_bias,
            quant_config=quant_config,
            prefix=f"{prefix}.o_proj",
        )
        self.q_norm = RMSNorm(hidden_size=self.head_dim, eps=config.rms_norm_eps)
        self.k_norm = RMSNorm(hidden_size=self.head_dim, eps=config.rms_norm_eps)
        self.v_norm = RMSNorm(
            hidden_size=self.head_dim, eps=config.rms_norm_eps, has_weight=False
        )

        layer_idx = extract_layer_index(prefix)
        layer_type = config.layer_types[layer_idx]
        is_sliding = layer_type == "sliding_attention"
        self.sliding_window = config.sliding_window if is_sliding else None

# ... truncated for analysis ...
```
**EN:** Method `Gemma3nAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma3nAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma3nAttention.forward` (lines 404-427)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        **kwargs,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)

        q = q.unflatten(-1, (self.num_heads, self.head_dim))
        q = self.q_norm(q)
        q = q.flatten(-2, -1)
        k = k.unflatten(-1, (self.num_kv_heads, self.head_dim))
        k = self.k_norm(k)
        k = k.flatten(-2, -1)
        v = v.unflatten(-1, (self.num_kv_heads, self.head_dim))
        v = self.v_norm(v)
        v = v.flatten(-2, -1)

        q, k = self.rotary_emb(positions, q, k)
        attn_output = self.attn(q, k, v)

        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** Method `Gemma3nAttention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Gemma3nAttention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Gemma3nDecoderLayer` (lines 430-570)
```python
class Gemma3nDecoderLayer(nn.Module):
    def __init__(
        self,
        config: Gemma3nTextConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        assert isinstance(config, Gemma3nTextConfig)
        self.altup_active_idx = config.altup_active_idx
        assert config.altup_correct_scale

        self.altup = Gemma3nAltUp(
            hidden_size=config.hidden_size,
            rms_norm_eps=config.rms_norm_eps,
            altup_num_inputs=config.altup_num_inputs,
            altup_coef_clip=config.altup_coef_clip,
            altup_active_idx=config.altup_active_idx,
            quant_config=quant_config,
            prefix=f"{prefix}.altup",
        )
        self.self_attn = Gemma3nAttention(
            config=config,
            hidden_size=config.hidden_size,
```
**EN:** Class `Gemma3nDecoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Gemma3nDecoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Gemma3nDecoderLayer.__init__` (lines 431-524)
```python
    def __init__(
        self,
        config: Gemma3nTextConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        assert isinstance(config, Gemma3nTextConfig)
        self.altup_active_idx = config.altup_active_idx
        assert config.altup_correct_scale

        self.altup = Gemma3nAltUp(
            hidden_size=config.hidden_size,
            rms_norm_eps=config.rms_norm_eps,
            altup_num_inputs=config.altup_num_inputs,
            altup_coef_clip=config.altup_coef_clip,
            altup_active_idx=config.altup_active_idx,
            quant_config=quant_config,
            prefix=f"{prefix}.altup",
        )
        self.self_attn = Gemma3nAttention(
            config=config,
            hidden_size=config.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=config.num_key_value_heads,
            head_dim=config.head_dim,
            max_position_embeddings=config.max_position_embeddings,
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
        )
        self.mlp = Gemma3nMLP(
            hidden_size=config.hidden_size,
            # NOTE: Matformer https://github.com/huggingface/transformers/blob/a52478253bbe522a420e88ea3940d4d98a935300/src/transformers/models/gemma3n/modular_gemma3n.py#L258 # noqa: E501
            intermediate_size=config.intermediate_size[extract_layer_index(prefix)],
            hidden_activation=config.hidden_activation,
            quant_config=quant_config,
            activation_sparsity=config.activation_sparsity_pattern[
                extract_layer_index(prefix)
            ],
            prefix=f"{prefix}.mlp",
        )
        self.laurel = Gemma3nLaurelBlock(
            hidden_size=config.hidden_size,
            laurel_rank=config.laurel_rank,
            rms_norm_eps=config.rms_norm_eps,
            quant_config=quant_config,
            prefix=f"{prefix}.laurel",
        )

        # NOTE(rob): should be ColumnParallelLinear and RowParallelLinear
        # But, we need to add per_layer_input_gate(x) to per_layer_input.
        # per_layer_input cannot be sharded, so we replicate for now.
        self.per_layer_input_gate = ReplicatedLinear(
            config.hidden_size,
            config.hidden_size_per_layer_input,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.per_layer_input_gate",
# ... truncated for analysis ...
```
**EN:** Method `Gemma3nDecoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma3nDecoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma3nDecoderLayer.forward` (lines 526-570)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        per_layer_input: torch.Tensor,
        **kwargs,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        # ActUp (predict).
        predictions = self.altup.predict(hidden_states)
        active_prediction = predictions[self.altup_active_idx]
        active_prediction_normed = self.input_layernorm(active_prediction)
        laurel_output = self.laurel(active_prediction_normed)

        # Attention.
        attn = self.self_attn(
            positions=positions,
            hidden_states=active_prediction_normed,
            **kwargs,
        )
        attn = self.post_attention_layernorm(attn)
        attn_gated = attn + active_prediction
        attn_laurel = (attn_gated + laurel_output) / torch.sqrt(torch.tensor(2.0))

        # MLP.
        attn_norm = self.pre_feedforward_layernorm(attn_laurel)
        attn_ffw = self.mlp(attn_norm)
        attn_ffw_norm = self.post_feedforward_layernorm(attn_ffw)
        attn_ffw_laurel_gated = attn_laurel + attn_ffw_norm

        # ActUp (connect).
        corrected_predictions = self.altup.correct(predictions, attn_ffw_laurel_gated)
        first_prediction = corrected_predictions[self.altup_active_idx]
        first_prediction = self.altup.scale_corrected_output(first_prediction)

        # per_layer_input_gate adapted from jax.numpy.einsum("btd,dp->btp", ...)
        first_prediction = self.per_layer_input_gate(first_prediction)
        first_prediction = self.act_fn(first_prediction)
        first_prediction = torch.mul(first_prediction, per_layer_input)

        # per_layer_projection adapted from jax.numpy.einsum("btp,pd->btd", ...)
        first_prediction = self.per_layer_projection(first_prediction)
        first_prediction = self.post_per_layer_input_norm(first_prediction)
        corrected_predictions[1:] += first_prediction

        return corrected_predictions
```
**EN:** Method `Gemma3nDecoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Gemma3nDecoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Gemma3nSelfDecoder` (lines 577-739)
```python
@support_torch_compile(
    enable_if=lambda vllm_config: vllm_config.cache_config.kv_sharing_fast_prefill
)
class Gemma3nSelfDecoder(nn.Module):
    """
    Includes altup embedding and self decoder layers
    """

    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        decoder_layers: list[Gemma3nDecoderLayer],
        layer_idx_start: int,
    ):
        super().__init__()
        self.decoder_layers = decoder_layers
        self.layer_idx_start = layer_idx_start

        config = vllm_config.model_config.hf_config
        self.config = config
        quant_config = vllm_config.quant_config

        self.embed_tokens = VocabParallelEmbedding(
```
**EN:** Class `Gemma3nSelfDecoder` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, get_per_layer_input_embeddings, get_per_layer_inputs, embed_input_ids, altup_embed, forward.
**CN:** 类 `Gemma3nSelfDecoder` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, get_per_layer_input_embeddings, get_per_layer_inputs, embed_input_ids, altup_embed, forward。

### Method `Gemma3nSelfDecoder.__init__` (lines 582-652)
```python
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        decoder_layers: list[Gemma3nDecoderLayer],
        layer_idx_start: int,
    ):
        super().__init__()
        self.decoder_layers = decoder_layers
        self.layer_idx_start = layer_idx_start

        config = vllm_config.model_config.hf_config
        self.config = config
        quant_config = vllm_config.quant_config

        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=f"{prefix}.embed_tokens",
        )
        self.embed_scale = torch.tensor(
            config.hidden_size**0.5,
            dtype=self.embed_tokens.weight.dtype,
        )
        # Additional per-layer embeddings (PLE)
        self.embed_tokens_per_layer = VocabParallelEmbedding(
            config.vocab_size_per_layer_input,
            config.num_hidden_layers * config.hidden_size_per_layer_input,
            quant_config=quant_config,
            prefix=f"{prefix}.per_layer_embed_tokens",
        )
        self.embed_scale_per_layer = torch.tensor(
            config.hidden_size_per_layer_input**0.5,
            dtype=self.embed_tokens.weight.dtype,
        )
        self.per_layer_model_projection = ColumnParallelLinear(
            config.hidden_size,
            config.num_hidden_layers * config.hidden_size_per_layer_input,
            bias=False,
            gather_output=True,
            return_bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.per_layer_model_projection",
        )
        self.per_layer_projection_norm = RMSNorm(
            hidden_size=config.hidden_size_per_layer_input,
            eps=config.rms_norm_eps,
        )
        self.per_layer_input_scale = torch.rsqrt(torch.tensor(2.0)).to(
            self.embed_tokens.weight.dtype
        )
        self.per_layer_projection_scale = torch.tensor(
            config.hidden_size**0.5,
            dtype=self.embed_tokens.weight.dtype,
        )
        self.altup_projections = nn.ModuleList(
            [
                ColumnParallelLinear(
# ... truncated for analysis ...
```
**EN:** Method `Gemma3nSelfDecoder.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma3nSelfDecoder.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma3nSelfDecoder.get_per_layer_input_embeddings` (lines 654-667)
```python
    def get_per_layer_input_embeddings(self, input_ids: torch.Tensor) -> torch.Tensor:
        # Deal with the fact that vocab_size_per_layer_input < vocab_size
        # which causes us to have some out of vocab tokens by setting
        # those token ids to 0. This matches the HF implementation.
        per_layer_inputs_mask = torch.logical_and(
            input_ids >= 0, input_ids < self.config.vocab_size_per_layer_input
        )
        per_layer_inputs_tokens = torch.where(
            per_layer_inputs_mask, input_ids, torch.zeros_like(input_ids)
        )
        return (
            self.embed_tokens_per_layer(per_layer_inputs_tokens)
            * self.embed_scale_per_layer
        )
```
**EN:** Method `Gemma3nSelfDecoder.get_per_layer_input_embeddings` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Gemma3nSelfDecoder.get_per_layer_input_embeddings` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Class `Gemma3nCrossDecoder` (lines 746-783)
```python
@support_torch_compile(
    enable_if=lambda vllm_config: vllm_config.cache_config.kv_sharing_fast_prefill
)
class Gemma3nCrossDecoder(nn.Module):
    """
    Cross-decoder layers
    """

    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        decoder_layers: list[Gemma3nDecoderLayer],
        layer_idx_start: int,
    ):
        super().__init__()
        self.decoder_layers = decoder_layers
        self.layer_idx_start = layer_idx_start

    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        per_layer_inputs: torch.Tensor,
```
**EN:** Class `Gemma3nCrossDecoder` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Gemma3nCrossDecoder` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Gemma3nCrossDecoder.__init__` (lines 751-761)
```python
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        decoder_layers: list[Gemma3nDecoderLayer],
        layer_idx_start: int,
    ):
        super().__init__()
        self.decoder_layers = decoder_layers
        self.layer_idx_start = layer_idx_start
```
**EN:** Method `Gemma3nCrossDecoder.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma3nCrossDecoder.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma3nCrossDecoder.forward` (lines 763-783)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        per_layer_inputs: torch.Tensor,
        **kwargs,
    ) -> torch.Tensor:
        # [altnum_inputs, num_tokens, hidden_size]
        hidden_states = hidden_states.permute(2, 0, 1)
        for idx, layer in enumerate(self.decoder_layers):
            layer_idx = idx + self.layer_idx_start
            # [altup_num_inputs, num_tokens, hidden_size]
            hidden_states = layer(
                positions=positions,
                hidden_states=hidden_states,
                per_layer_input=per_layer_inputs[:, layer_idx, :],
                **kwargs,
            )
        # [num_tokens, hidden_size, altnum_inputs]
        hidden_states = hidden_states.permute(1, 2, 0)
        return hidden_states
```
**EN:** Method `Gemma3nCrossDecoder.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Gemma3nCrossDecoder.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Gemma3nTextModel` (lines 790-1100)
```python
@support_torch_compile(
    enable_if=lambda vllm_config: not vllm_config.cache_config.kv_sharing_fast_prefill
)
class Gemma3nTextModel(nn.Module, SupportsQuant):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config
        self.config = config
        self.quant_config = quant_config

        self.altup_unembed_projections = nn.ModuleList(
            [
                ColumnParallelLinear(
                    config.hidden_size,
                    config.hidden_size,
                    bias=False,
                    gather_output=True,
                    return_bias=False,
                    quant_config=quant_config,
                    prefix=f"{prefix}.altup_unembed_projections.{idx - 1}",
                )
                for idx in range(1, self.config.altup_num_inputs)
            ]
```
**EN:** Class `Gemma3nTextModel` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsQuant. Key methods include __init__, embed_tokens, get_per_layer_input_embeddings, embed_input_ids, fast_prefill_forward, normal_forward.
**CN:** 类 `Gemma3nTextModel` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsQuant。 关键方法包括 __init__, embed_tokens, get_per_layer_input_embeddings, embed_input_ids, fast_prefill_forward, normal_forward。

### Method `Gemma3nTextModel.__init__` (lines 791-876)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config
        self.config = config
        self.quant_config = quant_config

        self.altup_unembed_projections = nn.ModuleList(
            [
                ColumnParallelLinear(
                    config.hidden_size,
                    config.hidden_size,
                    bias=False,
                    gather_output=True,
                    return_bias=False,
                    quant_config=quant_config,
                    prefix=f"{prefix}.altup_unembed_projections.{idx - 1}",
                )
                for idx in range(1, self.config.altup_num_inputs)
            ]
        )

        # Allocate config.num_kv_shared_layers layers for self-decoder
        self.start_layer, self.end_layer, self.layers = make_layers(
            config.num_hidden_layers,
            lambda prefix: Gemma3nDecoderLayer(
                config, cache_config, quant_config, prefix=prefix
            ),
            prefix=f"{prefix}.layers",
        )

        first_kv_shared_layer_idx = (
            config.num_hidden_layers - config.num_kv_shared_layers
        )

        # NOTE(sarckk): importing this top level seems to cause issues
        # during running of tests.
        from vllm.compilation.backends import set_model_tag

        # Layer idx 0-19 are self-decoder layers in You Only Cache Once (YOCO)
        with set_model_tag("self_decoder"):
            self.self_decoder = Gemma3nSelfDecoder(
                vllm_config=vllm_config,
                prefix=f"{prefix}.self_decoder",
                decoder_layers=self.layers[:first_kv_shared_layer_idx],
                layer_idx_start=0,
            )
        # Layer idx 20-30 are cross-decoder layers in YOCO
        with set_model_tag("cross_decoder"):
            self.cross_decoder = Gemma3nCrossDecoder(
                vllm_config=vllm_config,
                prefix=f"{prefix}.cross_decoder",
                decoder_layers=self.layers[first_kv_shared_layer_idx:],
                layer_idx_start=first_kv_shared_layer_idx,
            )

        self.norm = RMSNorm(
            config.hidden_size,
            eps=config.rms_norm_eps,
# ... truncated for analysis ...
```
**EN:** Method `Gemma3nTextModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma3nTextModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma3nTextModel.embed_tokens` (lines 879-880)
```python
    @property
    def embed_tokens(self):
        return self.self_decoder.embed_tokens
```
**EN:** Method `Gemma3nTextModel.embed_tokens` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Gemma3nTextModel.embed_tokens` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Class `Gemma3nForCausalLM` (lines 1103-1166)
```python
class Gemma3nForCausalLM(nn.Module):
    packed_modules_mapping = {
        "qkv_proj": [
            "q_proj",
            "k_proj",
            "v_proj",
        ],
        "gate_up_proj": [
            "gate_proj",
            "up_proj",
        ],
    }

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        config = vllm_config.model_config.hf_config

        super().__init__()
        self.config = config
        self.cache_config = vllm_config.cache_config
        self.model = Gemma3nTextModel(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )
        self.logits_processor = LogitsProcessor(
            config.vocab_size, soft_cap=config.final_logit_softcapping
        )
```
**EN:** Class `Gemma3nForCausalLM` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, embed_input_ids, forward, compute_logits, load_weights.
**CN:** 类 `Gemma3nForCausalLM` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, embed_input_ids, forward, compute_logits, load_weights。

### Method `Gemma3nForCausalLM.__init__` (lines 1116-1127)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        config = vllm_config.model_config.hf_config

        super().__init__()
        self.config = config
        self.cache_config = vllm_config.cache_config
        self.model = Gemma3nTextModel(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )
        self.logits_processor = LogitsProcessor(
            config.vocab_size, soft_cap=config.final_logit_softcapping
        )
```
**EN:** Method `Gemma3nForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma3nForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma3nForCausalLM.embed_input_ids` (lines 1129-1130)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.model.embed_input_ids(input_ids)
```
**EN:** Method `Gemma3nForCausalLM.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Gemma3nForCausalLM.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

## Key Concepts / 关键概念
- **Attention / 注意力**
  - **EN:** Attention layers model token-to-token or token-to-feature interactions inside the architecture.
  - **CN:** 注意力层负责建模 token 与 token，或 token 与特征之间的交互。
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。
- **KV cache / KV 缓存**
  - **EN:** The runtime stores key/value state so decoding can reuse prior computation efficiently.
  - **CN:** 运行时会保存 key/value 状态，以便解码阶段高效复用历史计算。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from collections.abc import Iterable`
- **Third-party / 第三方**: `import torch`, `from torch import nn`, `from transformers.models.gemma3n.configuration_gemma3n import Gemma3nTextConfig`
- **vLLM internal / vLLM 内部依赖**: `from vllm.compilation.decorators import support_torch_compile`, `from vllm.config import CacheConfig, VllmConfig`, `from vllm.distributed import get_tensor_model_parallel_world_size`, `from vllm.forward_context import get_forward_context`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.activation import (`, `from vllm.model_executor.layers.attention import Attention`, `from vllm.model_executor.layers.layernorm import RMSNorm`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.rotary_embedding import get_rope`
