# gemma4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/gemma4.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for gemma4, including architecture wrappers and weight loading logic. / 面向推理的 gemma4 vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 21-83)
```python
from collections.abc import Iterable
from dataclasses import replace
from itertools import islice

import regex as re
import torch
from torch import nn

from vllm.compilation.decorators import support_torch_compile
from vllm.config import CacheConfig, VllmConfig
from vllm.distributed import (
    get_pp_group,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
)
from vllm.forward_context import get_forward_context
from vllm.logger import init_logger
from vllm.model_executor.layers.activation import get_act_and_mul_fn
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.fused_moe import (
    FusedMoE,
    GateLinear,
    fused_moe_make_expert_params_mapping,
)
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
from vllm.model_executor.layers.vocab_parallel_embedding import (
    ParallelLMHead,
    VocabParallelEmbedding,
)
from vllm.model_executor.model_loader.weight_utils import (
    default_weight_loader,
    maybe_remap_kv_scale_name,
)
from vllm.platforms import current_platform
from vllm.sequence import IntermediateTensors
from vllm.triton_utils import tl, triton
from vllm.v1.attention.backends.utils import KVSharingFastPrefillMetadata

from .interfaces import (
    EagleModelMixin,
    MixtureOfExperts,
    SupportsEagle3,
    SupportsLoRA,
    SupportsPP,
)
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    extract_layer_index,
    is_pp_missing_parameter,
    make_layers,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Function `_remap_gemma4_expert_weight_name` (lines 88-89)
```python
def _remap_gemma4_expert_weight_name(name: str) -> str:
    return re.sub(r"(?<!\.moe)\.experts\.(\d+)\.", r".moe.experts.\1.", name)
```
**EN:** Function `_remap_gemma4_expert_weight_name` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Function `_remap_gemma4_expert_weight_name` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Function `_gemma4_routing_kernel` (lines 93-155)
```python
@triton.jit
def _gemma4_routing_kernel(
    gating_ptr,
    per_expert_scale_ptr,
    topk_weights_ptr,
    topk_ids_ptr,
    E: tl.constexpr,
    K: tl.constexpr,
    BLOCK_E: tl.constexpr,
):
    pid = tl.program_id(0)
    offs_e = tl.arange(0, BLOCK_E)
    valid = offs_e < E

    logits = tl.load(
        gating_ptr + pid * E + offs_e,
        mask=valid,
        other=-float("inf"),
    ).to(tl.float32)

    max_l = tl.max(logits, axis=0)

    # Float32 → ascending-sortable bijection
    MIN32 = -2147483648
    logit_bits = logits.to(tl.int32, bitcast=True)
    sign_b = logit_bits >> 31
    key = tl.where(sign_b == 0, logit_bits ^ -1, logit_bits ^ MIN32)
    key = tl.where(valid, key, 0x7FFFFFFF)
    sk64 = key.to(tl.int64) & 0x00000000FFFFFFFF
    packed = (sk64 << 32) | offs_e.to(tl.int64)
    sorted_p = tl.sort(packed, descending=False)

    # Vectorized extraction of ALL sorted elements — no K-loop, no cross-lane reductions
    all_keys = ((sorted_p >> 32) & 0x00000000FFFFFFFF).to(tl.int32)
    all_ids = (sorted_p & 0x00000000FFFFFFFF).to(tl.int32)

    # Inverse bijection: recover original logit bits
    sign_k = all_keys >> 31
    all_bits = tl.where(sign_k < 0, all_keys ^ -1, all_keys ^ MIN32)
    all_logits = all_bits.to(tl.float32, bitcast=True)

    # Compute raw_exp for ALL BLOCK_E elements — vectorized, ~2 VALU clocks
    all_raw_exp = tl.math.exp2((all_logits - max_l) * 1.4426950408889634)

    # Sum only top-K for renorm — ONE masked reduction
    top_mask = offs_e < K
    renorm_raw = tl.sum(tl.where(top_mask, all_raw_exp, 0.0), axis=0)
    renorm_raw = tl.where(renorm_raw > 0.0, renorm_raw, 1.0)
    inv_renorm = 1.0 / renorm_raw

    # Load scales for top-K only (masked gather; scale array is tiny → L1 cached)
    all_scales = tl.load(
        per_expert_scale_ptr + all_ids.to(tl.int64),
        mask=top_mask,
        other=1.0,
    ).to(tl.float32)

    # Final weights: vectorized multiply (only top-K will be stored)
    all_weights = (all_raw_exp * inv_renorm * all_scales).to(tl.float32)

# ... truncated for analysis ...
```
**EN:** Function `_gemma4_routing_kernel` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `_gemma4_routing_kernel` 封装了该模块中的一段可复用核心逻辑。

### Function `gemma4_fused_routing_kernel_triton` (lines 158-180)
```python
def gemma4_fused_routing_kernel_triton(
    gating_output: torch.Tensor,
    topk: int,
    per_expert_scale: torch.Tensor,
    num_warps: int = 1,
) -> tuple[torch.Tensor, torch.Tensor]:
    gating_output = gating_output.contiguous()
    per_expert_scale = per_expert_scale.contiguous()
    T, E = gating_output.shape
    weights = torch.empty(T, topk, dtype=torch.float32, device=gating_output.device)
    ids = torch.empty(T, topk, dtype=torch.int32, device=gating_output.device)
    BLOCK_E = triton.next_power_of_2(E)
    _gemma4_routing_kernel[(T,)](
        gating_output,
        per_expert_scale,
        weights,
        ids,
        E,
        topk,
        BLOCK_E,
        num_warps=num_warps,
    )
    return weights, ids
```
**EN:** Function `gemma4_fused_routing_kernel_triton` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `gemma4_fused_routing_kernel_triton` 封装了该模块中的一段可复用核心逻辑。

### Function `gemma4_routing_function_torch` (lines 183-203)
```python
def gemma4_routing_function_torch(
    gating_output: torch.Tensor,
    topk: int,
    per_expert_scale: torch.Tensor,
) -> tuple[torch.Tensor, torch.Tensor]:
    _, topk_ids = torch.topk(gating_output, k=topk, dim=-1)
    router_probabilities = torch.nn.functional.softmax(gating_output, dim=-1)
    indicator = torch.nn.functional.one_hot(
        topk_ids, num_classes=gating_output.size(-1)
    ).sum(dim=-2)
    gate_weights = indicator * router_probabilities
    renorm_factor = torch.sum(gate_weights, dim=-1, keepdim=True)
    renorm_factor = torch.where(renorm_factor > 0.0, renorm_factor, 1.0)
    dispatch_weights = gate_weights / renorm_factor

    topk_weights = dispatch_weights.gather(1, topk_ids)

    # Fold per_expert_scale into routing weights
    expert_scales = per_expert_scale[topk_ids].to(topk_weights.dtype)
    topk_weights = topk_weights * expert_scales
    return topk_weights.to(torch.float32), topk_ids.to(torch.int32)
```
**EN:** Function `gemma4_routing_function_torch` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `gemma4_routing_function_torch` 封装了该模块中的一段可复用核心逻辑。

### Class `Gemma4MLP` (lines 218-248)
```python
class Gemma4MLP(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_activation: str,
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
        self.act_fn = get_act_and_mul_fn(hidden_activation)
```
**EN:** Class `Gemma4MLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Gemma4MLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Gemma4MLP.__init__` (lines 219-242)
```python
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_activation: str,
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
        self.act_fn = get_act_and_mul_fn(hidden_activation)
```
**EN:** Method `Gemma4MLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma4MLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma4MLP.forward` (lines 244-248)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        gate_up, _ = self.gate_up_proj(x)
        x = self.act_fn(gate_up)
        x, _ = self.down_proj(x)
        return x
```
**EN:** Method `Gemma4MLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Gemma4MLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Gemma4Router` (lines 251-298)
```python
class Gemma4Router(nn.Module):
    """Router for Gemma4 MoE that preprocesses input before projection.

    Applies RMSNorm (no learned weight), root_size scaling
    (hidden_size^{-0.5}), then a learned per-dimension scale before
    projecting to expert logits.

    This preprocessing is applied ONLY to the router's input, not to
    the expert MLPs' input.
    """

    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size

        # RMSNorm without learned weight — pure normalization only
        self.norm = RMSNorm(self.hidden_size, eps=config.rms_norm_eps, has_weight=False)
        # Per-dimension learned scale, applied after norm + root_size
        self.scale = nn.Parameter(torch.ones(self.hidden_size))
        # Constant 1/sqrt(hidden_size) scaling factor
```
**EN:** Class `Gemma4Router` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Gemma4Router` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Gemma4Router.__init__` (lines 262-290)
```python
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size

        # RMSNorm without learned weight — pure normalization only
        self.norm = RMSNorm(self.hidden_size, eps=config.rms_norm_eps, has_weight=False)
        # Per-dimension learned scale, applied after norm + root_size
        self.scale = nn.Parameter(torch.ones(self.hidden_size))
        # Constant 1/sqrt(hidden_size) scaling factor
        self.register_buffer(
            "root_size",
            torch.tensor(self.hidden_size**-0.5),
            persistent=False,
        )
        # Project to expert logits; replicated across TP for consistent routing
        # GateLinear supports bf16 W/A → fp32 output, which is important
        # because the topk kernel often needs fp32 for stable routing.
        self.proj = GateLinear(
            self.hidden_size,
            config.num_experts,
            bias=False,
            out_dtype=torch.float32,
            prefix=f"{prefix}.proj",
        )
```
**EN:** Method `Gemma4Router.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma4Router.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma4Router.forward` (lines 292-298)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        """Returns raw router logits [T, E]."""
        x = self.norm(x)
        x = x * self.root_size.to(x.dtype)
        x = x * self.scale.to(x.dtype)
        router_logits, _ = self.proj(x)
        return router_logits
```
**EN:** Method `Gemma4Router.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Returns raw router logits [T, E].
**CN:** Method `Gemma4Router.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Returns raw router logits [T, E]。

### Class `Gemma4MoE` (lines 301-365)
```python
class Gemma4MoE(nn.Module):
    """Mixture of Experts for Gemma4 using vLLM's FusedMoE.

    Wraps FusedMoE with custom routing. The router projection is
    external (Gemma4Router) — this class only handles expert dispatch.

    Gemma4 routing: softmax over ALL experts → top-k → renormalize.
    per_expert_scale is folded into routing weights for mathematical
    correctness with FusedMoE's fused kernel.
    """

    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        self.num_experts = config.num_experts

        # Per-expert output scale folded into routing weights so that
        # FusedMoE's fused kernel computes: Σ_e (expert_e * w_e * scale_e)
        self.per_expert_scale = nn.Parameter(torch.ones(config.num_experts))
```
**EN:** Class `Gemma4MoE` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Gemma4MoE` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Gemma4MoE.__init__` (lines 312-362)
```python
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        self.num_experts = config.num_experts

        # Per-expert output scale folded into routing weights so that
        # FusedMoE's fused kernel computes: Σ_e (expert_e * w_e * scale_e)
        self.per_expert_scale = nn.Parameter(torch.ones(config.num_experts))

        # Gemma4 routing: softmax over ALL experts → top-k → renormalize.
        # FusedMoE's built-in fused_topk scopes softmax differently, so
        # a custom routing function is needed for numerical correctness.
        # NOTE: self.per_expert_scale is read at call time (not captured into
        # a local) so that torch.func.functional_call parameter substitution
        # reaches the routing function correctly.
        def routing_function(
            hidden_states: torch.Tensor,
            gating_output: torch.Tensor,
            topk: int,
            renormalize: bool,
        ) -> tuple[torch.Tensor, torch.Tensor]:
            if current_platform.is_cuda_alike() or current_platform.is_xpu():
                return gemma4_fused_routing_kernel_triton(
                    gating_output, topk, self.per_expert_scale
                )

            return gemma4_routing_function_torch(
                gating_output, topk, self.per_expert_scale
            )

        # FusedMoE experts with custom Gemma4 routing
        self.experts = FusedMoE(
            num_experts=config.num_experts,
            top_k=config.top_k_experts,
            hidden_size=config.hidden_size,
            intermediate_size=getattr(
                config,
                "moe_intermediate_size",
                getattr(config, "expert_intermediate_size", None),
            ),
            renormalize=True,
            quant_config=quant_config,
            prefix=f"{prefix}.experts",
            custom_routing_function=routing_function,
            activation="gelu_tanh",
        )
```
**EN:** Method `Gemma4MoE.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma4MoE.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma4MoE.forward` (lines 364-365)
```python
    def forward(self, x: torch.Tensor, router_logits: torch.Tensor) -> torch.Tensor:
        return self.experts(x, router_logits)
```
**EN:** Method `Gemma4MoE.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Gemma4MoE.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Gemma4Attention` (lines 368-539)
```python
class Gemma4Attention(nn.Module):
    def __init__(
        self,
        config,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        head_dim: int,
        max_position_embeddings: int,
        use_k_eq_v: bool = False,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        attn_logits_soft_cap: float | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.hidden_size = hidden_size
        self.use_k_eq_v = use_k_eq_v

        tp_size = get_tensor_model_parallel_world_size()
        self.tp_rank = get_tensor_model_parallel_rank()
        self.total_num_heads = num_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
```
**EN:** Class `Gemma4Attention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Gemma4Attention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Gemma4Attention.__init__` (lines 369-503)
```python
    def __init__(
        self,
        config,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        head_dim: int,
        max_position_embeddings: int,
        use_k_eq_v: bool = False,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        attn_logits_soft_cap: float | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.hidden_size = hidden_size
        self.use_k_eq_v = use_k_eq_v

        tp_size = get_tensor_model_parallel_world_size()
        self.tp_rank = get_tensor_model_parallel_rank()
        self.total_num_heads = num_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= tp_size:
            assert self.total_num_kv_heads % tp_size == 0
        else:
            assert tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        self.head_dim = head_dim
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        # Gemma4 uses scaling=1.0.
        # Unlike Gemma2/3, query_pre_attn_scalar is NOT used here;
        # Q/K norms with learnable weights handle scaling implicitly.
        self.scaling = 1.0

        # QKVParallelLinear handles GQA correctly for all layer types.
        # k_eq_v layers load K weights into both K and V slots via
        # _weight_iterator remapping — no structural difference needed.
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

        # Q/K norms: output = norm(x) * weight (learnable per-head scale)
        self.q_norm = RMSNorm(self.head_dim, eps=config.rms_norm_eps)
# ... truncated for analysis ...
```
**EN:** Method `Gemma4Attention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma4Attention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma4Attention.forward` (lines 505-539)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        **kwargs,
    ) -> torch.Tensor:
        # Unified QKV path (works for both k_eq_v and standard layers).
        # For k_eq_v, K weights are loaded into both K and V slots of
        # qkv_proj, so V == K automatically.
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)

        # Q norm (always applied)
        q = q.unflatten(-1, (self.num_heads, self.head_dim))
        q = self.q_norm(q)
        q = q.flatten(-2, -1)

        if not self.is_kv_shared_layer:
            # Non-shared: apply K norm + RoPE, V norm
            k = k.unflatten(-1, (self.num_kv_heads, self.head_dim))
            k = self.k_norm(k)
            k = k.flatten(-2, -1)
            q, k = self.rotary_emb(positions, q, k)

            v = v.unflatten(-1, (self.num_kv_heads, self.head_dim))
            v = self.v_norm(v)
            v = v.flatten(-2, -1)
        else:
            # Shared: only apply RoPE to Q
            q = self.rotary_emb(positions, q, k)[0]

        attn_output = self.attn(q, k, v)
        output, _ = self.o_proj(attn_output)

        return output
```
**EN:** Method `Gemma4Attention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Gemma4Attention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Gemma4DecoderLayer` (lines 542-756)
```python
class Gemma4DecoderLayer(nn.Module):
    def __init__(
        self,
        config,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        self.hidden_size_per_layer_input = getattr(
            config, "hidden_size_per_layer_input", 0
        )

        layer_idx = extract_layer_index(prefix)
        self.layer_idx = layer_idx

        # Gemma4 uses different head dimensions for sliding vs full attention
        layer_type = config.layer_types[layer_idx]
        self.is_full_attention = layer_type == "full_attention"
        if self.is_full_attention:
            head_dim = getattr(config, "global_head_dim", config.head_dim)
        else:
            head_dim = config.head_dim
```
**EN:** Class `Gemma4DecoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Gemma4DecoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Gemma4DecoderLayer.__init__` (lines 543-694)
```python
    def __init__(
        self,
        config,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        self.hidden_size_per_layer_input = getattr(
            config, "hidden_size_per_layer_input", 0
        )

        layer_idx = extract_layer_index(prefix)
        self.layer_idx = layer_idx

        # Gemma4 uses different head dimensions for sliding vs full attention
        layer_type = config.layer_types[layer_idx]
        self.is_full_attention = layer_type == "full_attention"
        if self.is_full_attention:
            head_dim = getattr(config, "global_head_dim", config.head_dim)
        else:
            head_dim = config.head_dim

        # Determine if this full-attention layer uses k_eq_v
        # (laptop variant: no v_proj, K reused as V on full attention layers)
        use_k_eq_v = self.is_full_attention and getattr(
            config, "attention_k_eq_v", False
        )

        # For k_eq_v full-attention layers, use num_global_key_value_heads
        # as the KV head count when k_eq_v is enabled.
        if use_k_eq_v:
            num_kv_heads = getattr(
                config, "num_global_key_value_heads", config.num_key_value_heads
            )
        else:
            num_kv_heads = config.num_key_value_heads

        self.self_attn = Gemma4Attention(
            config=config,
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=num_kv_heads,
            head_dim=head_dim,
            max_position_embeddings=config.max_position_embeddings,
            use_k_eq_v=use_k_eq_v,
            cache_config=cache_config,
            quant_config=quant_config,
            attn_logits_soft_cap=getattr(config, "attn_logit_softcapping", None),
            prefix=f"{prefix}.self_attn",
        )

        # Compute per-layer intermediate_size from config.
        # When use_double_wide_mlp is set, intermediate_size doubles for
        # KV-shared layers (layers >= first_kv_shared_layer_idx).
        first_kv_shared_layer_idx = config.num_hidden_layers - getattr(
            config, "num_kv_shared_layers", 0
        )
        is_kv_shared_layer = layer_idx >= first_kv_shared_layer_idx > 0
# ... truncated for analysis ...
```
**EN:** Method `Gemma4DecoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma4DecoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma4DecoderLayer.forward` (lines 696-756)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        residual: torch.Tensor | None,
        per_layer_input: torch.Tensor | None = None,
        **kwargs,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        # Gemma4 residual pattern:
        # 1. input_norm(x) → attn → post_attn_norm → ADD residual
        # 2. pre_ff_norm → mlp → post_ff_norm → ADD residual
        residual = hidden_states

        hidden_states = self.input_layernorm(residual)

        hidden_states = self.self_attn(
            positions=positions,
            hidden_states=hidden_states,
            **kwargs,
        )

        hidden_states = self.post_attention_layernorm(hidden_states)
        hidden_states = hidden_states + residual
        residual = hidden_states

        # MLP runs unconditionally (same inputs for MoE and non-MoE)
        hidden_states = self.pre_feedforward_layernorm(hidden_states)
        hidden_states = self.mlp(hidden_states)

        if self.enable_moe_block:
            hidden_states_1 = self.post_feedforward_layernorm_1(hidden_states)

            # Router and MoE experts see the residual (pre-MLP state),
            # matching the HF transformers forward path
            router_logits = self.router(residual)
            hidden_states_2 = self.pre_feedforward_layernorm_2(residual)
            hidden_states_2 = self.moe(hidden_states_2, router_logits)
            hidden_states_2 = self.post_feedforward_layernorm_2(hidden_states_2)

            # Combine MLP and MoE outputs
            hidden_states = hidden_states_1 + hidden_states_2

        hidden_states = self.post_feedforward_layernorm(hidden_states)
        hidden_states = hidden_states + residual

        # Apply PLE (Per-Layer Embedding) if configured
        if per_layer_input is not None and self.per_layer_input_gate is not None:
            gate = self.per_layer_input_gate(hidden_states)
            gate = torch.nn.functional.gelu(gate, approximate="tanh")
            gated_per_layer = gate * per_layer_input
            per_layer_contribution = self.per_layer_projection(gated_per_layer)
            per_layer_contribution = self.post_per_layer_input_norm(
                per_layer_contribution
            )
            hidden_states = hidden_states + per_layer_contribution

        # Apply layer scalar for full-attention layers
        # Apply per-layer scalar (all text layers)
        hidden_states = hidden_states * self.layer_scalar

# ... truncated for analysis ...
```
**EN:** Method `Gemma4DecoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Gemma4DecoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Gemma4SelfDecoderLayers` (lines 787-917)
```python
@support_torch_compile(
    enable_if=lambda vllm_config: vllm_config.cache_config.kv_sharing_fast_prefill
)
class Gemma4SelfDecoderLayers(nn.Module):
    """Compiled wrapper: embedding + non-KV-shared layers (YOCO first half).

    Owns the embedding and PLE modules so they are inside the compiled
    graph. Gemma4Model delegates embedding methods here.
    """

    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        decoder_layers: list[Gemma4DecoderLayer],
        layer_idx_start: int,
        embed_tokens: VocabParallelEmbedding,
        normalizer: torch.Tensor,
        embed_tokens_per_layer: VocabParallelEmbedding | None,
        embed_scale_per_layer: torch.Tensor | None,
        per_layer_model_projection: ColumnParallelLinear | None,
        per_layer_projection_norm: RMSNorm | None,
        per_layer_input_scale: torch.Tensor | None,
        per_layer_projection_scale: torch.Tensor | None,
```
**EN:** Class `Gemma4SelfDecoderLayers` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, embed_input_ids, get_per_layer_inputs, project_per_layer_inputs, forward.
**CN:** 类 `Gemma4SelfDecoderLayers` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, embed_input_ids, get_per_layer_inputs, project_per_layer_inputs, forward。

### Method `Gemma4SelfDecoderLayers.__init__` (lines 794-832)
```python
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        decoder_layers: list[Gemma4DecoderLayer],
        layer_idx_start: int,
        embed_tokens: VocabParallelEmbedding,
        normalizer: torch.Tensor,
        embed_tokens_per_layer: VocabParallelEmbedding | None,
        embed_scale_per_layer: torch.Tensor | None,
        per_layer_model_projection: ColumnParallelLinear | None,
        per_layer_projection_norm: RMSNorm | None,
        per_layer_input_scale: torch.Tensor | None,
        per_layer_projection_scale: torch.Tensor | None,
    ):
        super().__init__()
        self.decoder_layers = decoder_layers
        self.layer_idx_start = layer_idx_start

        config = _get_text_config(vllm_config.model_config.hf_config)
        self.config = config
        self.hidden_size_per_layer_input = getattr(
            config, "hidden_size_per_layer_input", 0
        )
        self.vocab_size_per_layer_input = getattr(
            config, "vocab_size_per_layer_input", config.vocab_size
        )

        # Shared references to modules owned by Gemma4Model — must be
        # inside this nn.Module so torch.compile captures them.
        self.embed_tokens = embed_tokens
        self.normalizer = normalizer
        self.embed_tokens_per_layer = embed_tokens_per_layer
        self.embed_scale_per_layer = embed_scale_per_layer
        self.per_layer_model_projection = per_layer_model_projection
        self.per_layer_projection_norm = per_layer_projection_norm
        self.per_layer_input_scale = per_layer_input_scale
        self.per_layer_projection_scale = per_layer_projection_scale
```
**EN:** Method `Gemma4SelfDecoderLayers.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma4SelfDecoderLayers.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma4SelfDecoderLayers.embed_input_ids` (lines 834-835)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embed_tokens(input_ids) * self.normalizer
```
**EN:** Method `Gemma4SelfDecoderLayers.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Gemma4SelfDecoderLayers.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Class `Gemma4CrossDecoderLayers` (lines 923-952)
```python
@support_torch_compile(
    enable_if=lambda vllm_config: vllm_config.cache_config.kv_sharing_fast_prefill
)
class Gemma4CrossDecoderLayers(nn.Module):
    """Cross-decoder layers (YOCO second half, KV-shared)."""

    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        decoder_layers: list[Gemma4DecoderLayer],
        layer_idx_start: int,
    ):
        super().__init__()
        self.decoder_layers = decoder_layers
        self.layer_idx_start = layer_idx_start

    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        per_layer_inputs: torch.Tensor | None = None,
        **kwargs,
    ) -> torch.Tensor:
```
**EN:** Class `Gemma4CrossDecoderLayers` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Gemma4CrossDecoderLayers` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Gemma4CrossDecoderLayers.__init__` (lines 926-936)
```python
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        decoder_layers: list[Gemma4DecoderLayer],
        layer_idx_start: int,
    ):
        super().__init__()
        self.decoder_layers = decoder_layers
        self.layer_idx_start = layer_idx_start
```
**EN:** Method `Gemma4CrossDecoderLayers.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma4CrossDecoderLayers.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma4CrossDecoderLayers.forward` (lines 938-952)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        per_layer_inputs: torch.Tensor | None = None,
        **kwargs,
    ) -> torch.Tensor:
        return _run_decoder_layers(
            self.decoder_layers,
            self.layer_idx_start,
            positions,
            hidden_states,
            per_layer_inputs,
            **kwargs,
        )
```
**EN:** Method `Gemma4CrossDecoderLayers.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Gemma4CrossDecoderLayers.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Gemma4Model` (lines 958-1503)
```python
@support_torch_compile(
    enable_if=lambda vllm_config: not vllm_config.cache_config.kv_sharing_fast_prefill
)
class Gemma4Model(nn.Module, EagleModelMixin):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = _get_text_config(vllm_config.model_config.hf_config)
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config
        self.config = config
        self.quant_config = quant_config

        # PLE config values (default to 0 if not present — disables PLE)
        self.hidden_size_per_layer_input = getattr(
            config, "hidden_size_per_layer_input", 0
        )
        self.vocab_size_per_layer_input = getattr(
            config, "vocab_size_per_layer_input", config.vocab_size
        )

        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=f"{prefix}.embed_tokens",
```
**EN:** Class `Gemma4Model` is a structural model block in the vLLM execution graph. It inherits from nn.Module, EagleModelMixin. Key methods include __init__, embed_input_ids, get_per_layer_inputs, project_per_layer_inputs, fast_prefill_forward, forward.
**CN:** 类 `Gemma4Model` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、EagleModelMixin。 关键方法包括 __init__, embed_input_ids, get_per_layer_inputs, project_per_layer_inputs, fast_prefill_forward, forward。

### Method `Gemma4Model.__init__` (lines 959-1158)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = _get_text_config(vllm_config.model_config.hf_config)
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config
        self.config = config
        self.quant_config = quant_config

        # PLE config values (default to 0 if not present — disables PLE)
        self.hidden_size_per_layer_input = getattr(
            config, "hidden_size_per_layer_input", 0
        )
        self.vocab_size_per_layer_input = getattr(
            config, "vocab_size_per_layer_input", config.vocab_size
        )

        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=f"{prefix}.embed_tokens",
        )

        # Per-Layer Embedding (PLE) components
        if (
            self.hidden_size_per_layer_input is not None
            and self.hidden_size_per_layer_input > 0
        ):
            total_ple_dim = self.hidden_size_per_layer_input * config.num_hidden_layers
            self.embed_tokens_per_layer = VocabParallelEmbedding(
                self.vocab_size_per_layer_input,
                total_ple_dim,
                quant_config=quant_config,
                prefix=f"{prefix}.embed_tokens_per_layer",
            )
            # Scaled embedding factor (from config, not hardcoded)
            # Register as buffer so it moves to GPU with the model
            # and interacts correctly with torch.compile AOT caching.
            self.register_buffer(
                "embed_scale_per_layer",
                torch.tensor(self.hidden_size_per_layer_input**0.5),
                persistent=False,
            )
            # Projection: hidden_size → total_ple_dim
            # ColumnParallelLinear with gather_output=True
            self.per_layer_model_projection = ColumnParallelLinear(
                config.hidden_size,
                total_ple_dim,
                bias=False,
                gather_output=True,
                return_bias=False,
                quant_config=quant_config,
                prefix=f"{prefix}.per_layer_model_projection",
            )
            # PLE projection norm: output = norm(x) * weight
            self.per_layer_projection_norm = RMSNorm(
                self.hidden_size_per_layer_input,
                eps=config.rms_norm_eps,
            )
            # Scale factor for combining projection + per_layer_inputs
# ... truncated for analysis ...
```
**EN:** Method `Gemma4Model.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma4Model.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma4Model.embed_input_ids` (lines 1160-1161)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.self_decoder.embed_input_ids(input_ids)
```
**EN:** Method `Gemma4Model.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Gemma4Model.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Class `Gemma4ForCausalLM` (lines 1506-1721)
```python
class Gemma4ForCausalLM(
    nn.Module, SupportsLoRA, SupportsPP, MixtureOfExperts, SupportsEagle3
):
    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_prefix={
            # Gemma4ForConditionalGeneration already loads the text stack
            # from `model.language_model.*`. We reuse that same checkpoint
            # and adapter naming for the text-only Gemma4ForCausalLM path,
            # so LoRA keys from the conditional wrapper map onto `model.*`.
            "model.language_model.": "model.",
        },
        orig_to_new_substr={
            # Gemma4ForConditionalGeneration names MoE adapter targets under
            # `...moe.experts.*`, while the text-only model exposes them
            # under `...moe.*`.
            ".moe.experts.gate_up_proj": ".moe.gate_up_proj",
            ".moe.experts.down_proj": ".moe.down_proj",
        },
    )
    # Note: qkv_proj packing applies to non-k_eq_v layers (sliding
    # attention and full attention without k_eq_v). k_eq_v layers use
    # separate q_proj + k_proj without packing.
    packed_modules_mapping = {
        "qkv_proj": [
            "q_proj",
```
**EN:** Class `Gemma4ForCausalLM` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsLoRA, SupportsPP, MixtureOfExperts, SupportsEagle3. Key methods include __init__, embed_input_ids, forward, compute_logits, load_weights.
**CN:** 类 `Gemma4ForCausalLM` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsLoRA、SupportsPP、MixtureOfExperts、SupportsEagle3。 关键方法包括 __init__, embed_input_ids, forward, compute_logits, load_weights。

### Method `Gemma4ForCausalLM.__init__` (lines 1540-1594)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        config = _get_text_config(vllm_config.model_config.hf_config)
        quant_config = vllm_config.quant_config

        super().__init__()
        self.config = config
        self.quant_config = quant_config
        self.model = Gemma4Model(
            vllm_config=vllm_config,
            prefix=maybe_prefix(prefix, "model"),
        )

        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "lm_head"),
        )
        if config.tie_word_embeddings:
            self.lm_head = self.lm_head.tie_weights(self.model.embed_tokens)

        self.logits_processor = LogitsProcessor(
            config.vocab_size,
            soft_cap=getattr(config, "final_logit_softcapping", None),
        )
        self.make_empty_intermediate_tensors = (
            self.model.make_empty_intermediate_tensors
        )

        # --- MixtureOfExperts protocol ---
        self.expert_weights: list[list[torch.Tensor]] = []
        self.moe_layers: list[nn.Module] = []
        example_moe: Gemma4MoE | None = None

        for layer in self.model.layers:
            if hasattr(layer, "moe") and isinstance(layer.moe, Gemma4MoE):
                example_moe = layer.moe
                self.moe_layers.append(layer.moe.experts)

        self.num_moe_layers = len(self.moe_layers)

        if example_moe is not None:
            self.num_logical_experts = example_moe.num_experts
            self.num_physical_experts = example_moe.num_experts
            self.num_local_physical_experts = example_moe.num_experts
            self.num_routed_experts = example_moe.num_experts
        else:
            self.num_logical_experts = 0
            self.num_physical_experts = 0
            self.num_local_physical_experts = 0
            self.num_routed_experts = 0

        self.num_expert_groups = 1
        self.num_shared_experts = 0
        self.num_redundant_experts = 0
```
**EN:** Method `Gemma4ForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma4ForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma4ForCausalLM.embed_input_ids` (lines 1596-1597)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.model.embed_input_ids(input_ids)
```
**EN:** Method `Gemma4ForCausalLM.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Gemma4ForCausalLM.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

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
- **Standard library / 标准库**: `from collections.abc import Iterable`, `from dataclasses import replace`, `from itertools import islice`, `import regex as re`
- **Third-party / 第三方**: `import torch`, `from torch import nn`
- **vLLM internal / vLLM 内部依赖**: `from vllm.compilation.decorators import support_torch_compile`, `from vllm.config import CacheConfig, VllmConfig`, `from vllm.distributed import (`, `from vllm.forward_context import get_forward_context`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.activation import get_act_and_mul_fn`, `from vllm.model_executor.layers.attention import Attention`, `from vllm.model_executor.layers.fused_moe import (`, `from vllm.model_executor.layers.layernorm import RMSNorm`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.layers.quantization import QuantizationConfig`
- **Module note / 模块说明**: **EN:** Gemma 4 model implementation for vLLM. **CN:** 模块文档字符串给出的原始说明是：Gemma 4 model implementation for vLLM.。
