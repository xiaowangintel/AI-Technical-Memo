# gdn_linear_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/mamba/gdn_linear_attn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `ChunkGatedDeltaRule`, `GatedDeltaNetAttention` for Mamba/state-space layers and kernels. / 实现 `ChunkGatedDeltaRule`, `GatedDeltaNetAttention`，用于Mamba/状态空间模型层与内核。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-3)
```python
"""Inference-only Qwen3-Next/Qwen3.5 model."""
```
**EN:** This docstring gives the module author's high-level intent: Inference-only Qwen3-Next/Qwen3.5 model. It is useful for reading the rest of the file because later classes and helpers refine this stated purpose.
**CN:** 这个文档字符串给出了模块作者的高层意图：Inference-only Qwen3-Next/Qwen3.5 model. 在阅读后续类和辅助函数时，可以把它视为整个文件的总纲。

### Imports and module setup (lines 5-67)
```python
import torch
from einops import rearrange
from torch import nn
from transformers.activations import ACT2FN

from vllm import envs
from vllm._aiter_ops import rocm_aiter_ops
from vllm.config import (
    VllmConfig,
    get_current_vllm_config,
)
from vllm.distributed import (
    divide,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
)
from vllm.forward_context import ForwardContext, get_forward_context
from vllm.logger import init_logger
from vllm.model_executor.custom_op import CustomOp, PluggableLayer
from vllm.model_executor.layers.fla.ops import (
    chunk_gated_delta_rule as fla_chunk_gated_delta_rule,
)
from vllm.model_executor.layers.fla.ops import (
    fused_post_conv_prep,
    fused_recurrent_gated_delta_rule_packed_decode,
    fused_sigmoid_gating_delta_rule_update,
)
from vllm.model_executor.layers.fla.ops.chunk import l2norm_fwd
from vllm.model_executor.layers.fla.ops.utils import FLA_CHUNK_SIZE
from vllm.model_executor.layers.layernorm import RMSNormGated
from vllm.model_executor.layers.linear import (
    ColumnParallelLinear,
    MergedColumnParallelLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.mamba.abstract import MambaBase
from vllm.model_executor.layers.mamba.mamba_mixer2 import mamba_v2_sharded_weight_loader
from vllm.model_executor.layers.mamba.mamba_utils import (
    MambaStateDtypeCalculator,
    MambaStateShapeCalculator,
    is_conv_state_dim_first,
)
from vllm.model_executor.layers.mamba.ops.causal_conv1d import (
    causal_conv1d_fn,
    causal_conv1d_update,
)
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.model_loader.weight_utils import (
    sharded_weight_loader,
)
from vllm.model_executor.models.utils import extract_layer_index
from vllm.model_executor.utils import set_weight_attrs
from vllm.platforms import current_platform
from vllm.transformers_utils.configs.qwen3_next import Qwen3NextConfig
from vllm.triton_utils import tl, triton
from vllm.utils.torch_utils import (
    LayerNameType,
    _encode_layer_name,
    _resolve_layer_name,
    direct_register_custom_op,
)
from vllm.v1.attention.backends.gdn_attn import GDNAttentionMetadata
from vllm.v1.attention.backends.registry import MambaAttentionBackendEnum
```
**EN:** This opening block pulls in external dependencies such as `torch`, `einops`, `transformers` and internal modules such as `vllm`, `vllm._aiter_ops`, `vllm.config`, `vllm.distributed`, `vllm.forward_context`, `vllm.logger`. That import mix shows the file is part of the Mamba/state-space layers and kernels stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `torch`, `einops`, `transformers`）以及内部模块（如 `vllm`, `vllm._aiter_ops`, `vllm.config`, `vllm.distributed`, `vllm.forward_context`, `vllm.logger`）。这些导入关系表明该文件属于Mamba/状态空间模型层与内核栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 73-83)
```python
GDN_AITER_TRITON_AVAILABLE = rocm_aiter_ops.are_gdn_triton_kernels_available()

if GDN_AITER_TRITON_AVAILABLE:
    from aiter.ops.triton.causal_conv1d_update_single_token import (
        fused_reshape_causal_conv1d_update_single_token as gdn_aiter_fused_reshape_causal_conv1d_update_single_token,  # noqa: E501
    )
    from aiter.ops.triton.gated_delta_net.fused_rearrange_sigmoid_gdr import (
        fused_rearrange_sigmoid_gated_delta_rule as gdn_aiter_fused_rearrange_sigmoid_gated_delta_rule,  # noqa: E501
    )

logger = init_logger(__name__)
```
**EN:** This block defines module-level metadata or constants such as `GDN_AITER_TRITON_AVAILABLE`, `logger`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the Mamba/state-space layers and kernels pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `GDN_AITER_TRITON_AVAILABLE`, `logger`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在Mamba/状态空间模型层与内核流程中复用。

### Function `fi_chunk_gated_delta_rule` (lines 86-132)
```python
def fi_chunk_gated_delta_rule(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    g: torch.Tensor,
    beta: torch.Tensor,
    initial_state: torch.Tensor,
    output_final_state: bool,
    cu_seqlens: torch.Tensor | None = None,
    use_qk_l2norm_in_kernel: bool = True,
):
    from flashinfer.gdn_prefill import (
        chunk_gated_delta_rule as chunk_gated_delta_rule_fi,
    )

    if use_qk_l2norm_in_kernel:
        q = l2norm_fwd(q)
        k = l2norm_fwd(k)

    # use flashinfer implementation
    q = q.squeeze(0).contiguous()
    k = k.squeeze(0).contiguous()
    v = v.squeeze(0).contiguous()

    g = g.squeeze(0).contiguous()
    beta = beta.squeeze(0).contiguous()
    fi_state = initial_state.to(torch.float32)
    fi_g = g.to(torch.float32)
    fi_beta = beta.to(torch.float32)
    result = chunk_gated_delta_rule_fi(
        q=q,
        k=k,
        v=v,
        g=torch.exp(fi_g),
        beta=fi_beta,
        initial_state=fi_state,
        output_final_state=output_final_state,
        cu_seqlens=cu_seqlens,
    )
    # FlashInfer returns (output, state) when output_final_state=True,
    # or just output when output_final_state=False.
    # Unsqueeze back to 4D (1, L, H, D) to match fla output format
    if output_final_state:
        output, final_state = result
        return output.unsqueeze(0), final_state
    else:
        return result.unsqueeze(0), None
```
**EN:** Defines function `fi_chunk_gated_delta_rule` with signature `fi_chunk_gated_delta_rule(q: torch.Tensor, k: torch.Tensor, v: torch.Tensor, g: torch.Tensor, beta: torch.Tensor, initial_state: torch.Tensor, output_final_state: bool, cu_seqlens: torch.Tensor | None=None, use_qk_l2norm_in_kernel: bool=True)`. It mainly works with `q`, `k`, `v`, `g`, `beta`, `initial_state`, `output_final_state`, `cu_seqlens`; implements one step of the Mamba/SSM execution path. The body uses branching, tensor/kernel operations. Key calls include `q.squeeze.contiguous`, `k.squeeze.contiguous`, `v.squeeze.contiguous`, `g.squeeze.contiguous`, `beta.squeeze.contiguous`, `initial_state.to`.
**CN:** 定义函数 `fi_chunk_gated_delta_rule`，其签名为 `fi_chunk_gated_delta_rule(q: torch.Tensor, k: torch.Tensor, v: torch.Tensor, g: torch.Tensor, beta: torch.Tensor, initial_state: torch.Tensor, output_final_state: bool, cu_seqlens: torch.Tensor | None=None, use_qk_l2norm_in_kernel: bool=True)`。它主要围绕 `q`, `k`, `v`, `g`, `beta`, `initial_state`, `output_final_state`, `cu_seqlens` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断、张量或内核操作。关键调用包括 `q.squeeze.contiguous`, `k.squeeze.contiguous`, `v.squeeze.contiguous`, `g.squeeze.contiguous`, `beta.squeeze.contiguous`, `initial_state.to`。

### Function `gdn_attention_core` (lines 1434-1471)
```python
def gdn_attention_core(
    qkv_or_qkvz: torch.Tensor,
    b_or_ba: torch.Tensor,
    a_or_z_out: torch.Tensor,
    core_attn_out: torch.Tensor,
    fast_kernel: bool,
    layer_name: LayerNameType,
) -> None:
    """Custom op dispatching to _forward_core or _forward_core_rocm.

    Handles conv1d + recurrent attention only; input/output projections
    are performed by the caller.

    When ``fast_kernel=False`` (standard path):
        qkv_or_qkvz is [q, k, v], b_or_ba is b, a_or_z_out is a (read-only).
    When ``fast_kernel=True`` (AITER Triton fast path, ROCm only):
        qkv_or_qkvz is [q, k, v, z], b_or_ba is [b, a], a_or_z_out is the
        z output buffer (mutated in-place).

    ``core_attn_out`` is always mutated in-place.
    """
    layer_name = _resolve_layer_name(layer_name)
    forward_context: ForwardContext = get_forward_context()
    self = forward_context.no_compile_layers[layer_name]
    if fast_kernel:
        self._forward_core_rocm(
            qkvz=qkv_or_qkvz,
            ba=b_or_ba,
            z_out=a_or_z_out,
            core_attn_out=core_attn_out,
        )
    else:
        self._forward_core(
            mixed_qkv=qkv_or_qkvz,
            b=b_or_ba,
            a=a_or_z_out,
            core_attn_out=core_attn_out,
        )
```
**EN:** Defines function `gdn_attention_core` with signature `gdn_attention_core(qkv_or_qkvz: torch.Tensor, b_or_ba: torch.Tensor, a_or_z_out: torch.Tensor, core_attn_out: torch.Tensor, fast_kernel: bool, layer_name: LayerNameType) -> None`. It mainly works with `qkv_or_qkvz`, `b_or_ba`, `a_or_z_out`, `core_attn_out`, `fast_kernel`, `layer_name`; implements one step of the Mamba/SSM execution path. The body uses branching. Key calls include `_resolve_layer_name`, `get_forward_context`, `self._forward_core_rocm`, `self._forward_core`.
**CN:** 定义函数 `gdn_attention_core`，其签名为 `gdn_attention_core(qkv_or_qkvz: torch.Tensor, b_or_ba: torch.Tensor, a_or_z_out: torch.Tensor, core_attn_out: torch.Tensor, fast_kernel: bool, layer_name: LayerNameType) -> None`。它主要围绕 `qkv_or_qkvz`, `b_or_ba`, `a_or_z_out`, `core_attn_out`, `fast_kernel`, `layer_name` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断。关键调用包括 `_resolve_layer_name`, `get_forward_context`, `self._forward_core_rocm`, `self._forward_core`。

### Function `gdn_attention_core_fake` (lines 1474-1483)
```python
def gdn_attention_core_fake(
    qkv_or_qkvz: torch.Tensor,
    b_or_ba: torch.Tensor,
    a_or_z_out: torch.Tensor,
    core_attn_out: torch.Tensor,
    fast_kernel: bool,
    layer_name: LayerNameType,
) -> None:
    """Fake implementation for torch.compile."""
    return
```
**EN:** Defines function `gdn_attention_core_fake` with signature `gdn_attention_core_fake(qkv_or_qkvz: torch.Tensor, b_or_ba: torch.Tensor, a_or_z_out: torch.Tensor, core_attn_out: torch.Tensor, fast_kernel: bool, layer_name: LayerNameType) -> None`. It mainly works with `qkv_or_qkvz`, `b_or_ba`, `a_or_z_out`, `core_attn_out`, `fast_kernel`, `layer_name`; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `gdn_attention_core_fake`，其签名为 `gdn_attention_core_fake(qkv_or_qkvz: torch.Tensor, b_or_ba: torch.Tensor, a_or_z_out: torch.Tensor, core_attn_out: torch.Tensor, fast_kernel: bool, layer_name: LayerNameType) -> None`。它主要围绕 `qkv_or_qkvz`, `b_or_ba`, `a_or_z_out`, `core_attn_out`, `fast_kernel`, `layer_name` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Function `fused_gdn_gating_kernel` (lines 1495-1527)
```python
def fused_gdn_gating_kernel(
    g,
    beta_output,
    A_log,
    a,
    b,
    dt_bias,
    seq_len,
    NUM_HEADS: tl.constexpr,
    beta: tl.constexpr,
    threshold: tl.constexpr,
    BLK_HEADS: tl.constexpr,
):
    i_b, i_s, i_d = tl.program_id(0), tl.program_id(1), tl.program_id(2)
    head_off = i_d * BLK_HEADS + tl.arange(0, BLK_HEADS)
    off = i_b * seq_len * NUM_HEADS + i_s * NUM_HEADS + head_off
    mask = head_off < NUM_HEADS
    blk_A_log = tl.load(A_log + head_off, mask=mask)
    blk_a = tl.load(a + off, mask=mask)
    blk_b = tl.load(b + off, mask=mask)
    blk_bias = tl.load(dt_bias + head_off, mask=mask)
    # If the model is loaded in fp16, without the .float() here, A might be -inf
    x = blk_a.to(tl.float32) + blk_bias.to(tl.float32)
    softplus_x = tl.where(
        beta * x <= threshold, (1 / beta) * tl.log(1 + tl.exp(beta * x)), x
    )
    blk_g = -tl.exp(blk_A_log.to(tl.float32)) * softplus_x
    tl.store(g + off, blk_g.to(g.dtype.element_ty), mask=mask)
    # compute beta_output = sigmoid(b)
    blk_beta_output = tl.sigmoid(blk_b.to(tl.float32))
    tl.store(
        beta_output + off, blk_beta_output.to(beta_output.dtype.element_ty), mask=mask
    )
```
**EN:** Defines function `fused_gdn_gating_kernel` with signature `fused_gdn_gating_kernel(g, beta_output, A_log, a, b, dt_bias, seq_len, NUM_HEADS: tl.constexpr, beta: tl.constexpr, threshold: tl.constexpr, BLK_HEADS: tl.constexpr)`. It mainly works with `g`, `beta_output`, `A_log`, `a`, `b`, `dt_bias`, `seq_len`, `NUM_HEADS`; implements one step of the Mamba/SSM execution path. The body uses tensor/kernel operations. Key calls include `tl.load`, `tl.where`, `tl.store`, `tl.sigmoid`, `tl.program_id`, `tl.arange`.
**CN:** 定义函数 `fused_gdn_gating_kernel`，其签名为 `fused_gdn_gating_kernel(g, beta_output, A_log, a, b, dt_bias, seq_len, NUM_HEADS: tl.constexpr, beta: tl.constexpr, threshold: tl.constexpr, BLK_HEADS: tl.constexpr)`。它主要围绕 `g`, `beta_output`, `A_log`, `a`, `b`, `dt_bias`, `seq_len`, `NUM_HEADS` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含张量或内核操作。关键调用包括 `tl.load`, `tl.where`, `tl.store`, `tl.sigmoid`, `tl.program_id`, `tl.arange`。

### Function `fused_gdn_gating` (lines 1530-1563)
```python
def fused_gdn_gating(
    A_log: torch.Tensor,
    a: torch.Tensor,
    b: torch.Tensor,
    dt_bias: torch.Tensor,
    beta: float = 1.0,
    threshold: float = 20.0,
) -> tuple[torch.Tensor, torch.Tensor]:
    """
    Fused computation of g and beta for Gated Delta Net.
    g = -self.A_log.float().exp() * F.softplus(a.float() + self.dt_bias)
    beta_output = b.sigmoid()
    TODO maybe use torch.compile to replace this triton kernel
    """
    batch, num_heads = a.shape
    seq_len = 1
    grid = (batch, seq_len, triton.cdiv(num_heads, 8))
    g = torch.empty(1, batch, num_heads, dtype=torch.float32, device=a.device)
    beta_output = torch.empty(1, batch, num_heads, dtype=b.dtype, device=b.device)
    fused_gdn_gating_kernel[grid](
        g,
        beta_output,
        A_log,
        a,
        b,
        dt_bias,
        seq_len,
        num_heads,
        beta,
        threshold,
        8,
        num_warps=1,
    )
    return g, beta_output
```
**EN:** Defines function `fused_gdn_gating` with signature `fused_gdn_gating(A_log: torch.Tensor, a: torch.Tensor, b: torch.Tensor, dt_bias: torch.Tensor, beta: float=1.0, threshold: float=20.0) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `A_log`, `a`, `b`, `dt_bias`, `beta`, `threshold`; implements one step of the Mamba/SSM execution path. The body uses tensor/kernel operations. Key calls include `torch.empty`, `fused_gdn_gating_kernel`, `triton.cdiv`.
**CN:** 定义函数 `fused_gdn_gating`，其签名为 `fused_gdn_gating(A_log: torch.Tensor, a: torch.Tensor, b: torch.Tensor, dt_bias: torch.Tensor, beta: float=1.0, threshold: float=20.0) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `A_log`, `a`, `b`, `dt_bias`, `beta`, `threshold` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含张量或内核操作。关键调用包括 `torch.empty`, `fused_gdn_gating_kernel`, `triton.cdiv`。

### Class `ChunkGatedDeltaRule` overview (lines 136-235)
```python
class ChunkGatedDeltaRule(CustomOp):
    def __init__(self) -> None:
        super().__init__()
        additional_config = get_current_vllm_config().additional_config
        assert isinstance(additional_config, dict)
        backend_cfg = additional_config.get("gdn_prefill_backend", "auto")
        backend = str(backend_cfg).strip().lower()

        supports_flashinfer = (
            current_platform.is_cuda() and current_platform.is_device_capability(90)
        )

        if backend == "flashinfer":
            use_flashinfer = supports_flashinfer
            if not use_flashinfer:
                logger.warning_once(
                    "GDN prefill backend 'flashinfer' is selected but "
                    "cannot use this kernel on the current platform. "
                    "Falling back to Triton/FLA."
                )
        elif backend == "triton":
            use_flashinfer = False
        else:
            use_flashinfer = supports_flashinfer
```
**EN:** Defines class `ChunkGatedDeltaRule` with base classes `CustomOp` and decorators `CustomOp.register('chunk_gated_delta_rule')`. It acts as a Mamba/SSM module building block and exposes 3 direct methods, with notable entries `__init__`, `forward_cuda`, `forward_native`.
**CN:** 定义类 `ChunkGatedDeltaRule`，其基类为 `CustomOp`，装饰器为 `CustomOp.register('chunk_gated_delta_rule')`。它在整体实现中充当Mamba/状态空间模块构件，并直接暴露 3 个方法，较重要的包括 `__init__`, `forward_cuda`, `forward_native`。

### Method `ChunkGatedDeltaRule.__init__` (lines 137-173)
```python
    def __init__(self) -> None:
        super().__init__()
        additional_config = get_current_vllm_config().additional_config
        assert isinstance(additional_config, dict)
        backend_cfg = additional_config.get("gdn_prefill_backend", "auto")
        backend = str(backend_cfg).strip().lower()

        supports_flashinfer = (
            current_platform.is_cuda() and current_platform.is_device_capability(90)
        )

        if backend == "flashinfer":
            use_flashinfer = supports_flashinfer
            if not use_flashinfer:
                logger.warning_once(
                    "GDN prefill backend 'flashinfer' is selected but "
                    "cannot use this kernel on the current platform. "
                    "Falling back to Triton/FLA."
                )
        elif backend == "triton":
            use_flashinfer = False
        else:
            use_flashinfer = supports_flashinfer

        if use_flashinfer:
            logger.info_once("Using FlashInfer GDN prefill kernel")
            logger.info_once(
                "FlashInfer GDN prefill kernel is JIT-compiled; first run may "
                "take a while to compile. Set `--gdn-prefill-backend triton` to "
                "avoid JIT compile time.",
            )
        else:
            logger.info_once("Using Triton/FLA GDN prefill kernel")

        self._forward_method = (
            self.forward_cuda if use_flashinfer else self.forward_native
        )
```
**EN:** Defines function `ChunkGatedDeltaRule.__init__` with signature `__init__(self) -> None`. It mainly works with object context only; initializes the object state and cached resources. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `super.__init__`, `isinstance`, `additional_config.get`, `str.strip.lower`, `get_current_vllm_config`, `current_platform.is_cuda`.
**CN:** 定义函数 `ChunkGatedDeltaRule.__init__`，其签名为 `__init__(self) -> None`。它主要围绕 仅依赖对象上下文 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `super.__init__`, `isinstance`, `additional_config.get`, `str.strip.lower`, `get_current_vllm_config`, `current_platform.is_cuda`。

### Method `ChunkGatedDeltaRule.forward_cuda` (lines 175-205)
```python
    def forward_cuda(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        g: torch.Tensor,
        beta: torch.Tensor,
        initial_state: torch.Tensor,
        output_final_state: bool,
        cu_seqlens: torch.Tensor | None = None,
        chunk_indices: torch.Tensor | None = None,
        chunk_offsets: torch.Tensor | None = None,
        use_qk_l2norm_in_kernel: bool = True,
        core_attn_out: torch.Tensor | None = None,
    ):
        o, final_state = fi_chunk_gated_delta_rule(
            q=q,
            k=k,
            v=v,
            g=g,
            beta=beta,
            initial_state=initial_state,
            output_final_state=output_final_state,
            cu_seqlens=cu_seqlens,
            use_qk_l2norm_in_kernel=use_qk_l2norm_in_kernel,
        )
        if core_attn_out is not None:
            o_flat = o.squeeze(0).reshape(-1)
            co_flat = core_attn_out.reshape(-1)
            co_flat[: o_flat.numel()].copy_(o_flat)
        return o, final_state
```
**EN:** Defines function `ChunkGatedDeltaRule.forward_cuda` with signature `forward_cuda(self, q: torch.Tensor, k: torch.Tensor, v: torch.Tensor, g: torch.Tensor, beta: torch.Tensor, initial_state: torch.Tensor, output_final_state: bool, cu_seqlens: torch.Tensor | None=None, chunk_indices: torch.Tensor | None=None, chunk_offsets: torch.Tensor | None=None, use_qk_l2norm_in_kernel: bool=True, core_attn_out: torch.Tensor | None=None)`. It mainly works with `q`, `k`, `v`, `g`, `beta`, `initial_state`, `output_final_state`, `cu_seqlens`; implements one step of the Mamba/SSM execution path. The body uses branching. Key calls include `fi_chunk_gated_delta_rule`, `o.squeeze.reshape`, `core_attn_out.reshape`, `co_flat.copy_`, `o.squeeze`, `o_flat.numel`.
**CN:** 定义函数 `ChunkGatedDeltaRule.forward_cuda`，其签名为 `forward_cuda(self, q: torch.Tensor, k: torch.Tensor, v: torch.Tensor, g: torch.Tensor, beta: torch.Tensor, initial_state: torch.Tensor, output_final_state: bool, cu_seqlens: torch.Tensor | None=None, chunk_indices: torch.Tensor | None=None, chunk_offsets: torch.Tensor | None=None, use_qk_l2norm_in_kernel: bool=True, core_attn_out: torch.Tensor | None=None)`。它主要围绕 `q`, `k`, `v`, `g`, `beta`, `initial_state`, `output_final_state`, `cu_seqlens` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断。关键调用包括 `fi_chunk_gated_delta_rule`, `o.squeeze.reshape`, `core_attn_out.reshape`, `co_flat.copy_`, `o.squeeze`, `o_flat.numel`。

### Method `ChunkGatedDeltaRule.forward_native` (lines 207-235)
```python
    def forward_native(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        g: torch.Tensor,
        beta: torch.Tensor,
        initial_state: torch.Tensor,
        output_final_state: bool,
        cu_seqlens: torch.Tensor | None = None,
        chunk_indices: torch.Tensor | None = None,
        chunk_offsets: torch.Tensor | None = None,
        use_qk_l2norm_in_kernel: bool = True,
        core_attn_out: torch.Tensor | None = None,
    ):
        return fla_chunk_gated_delta_rule(
            q=q,
            k=k,
            v=v,
            g=g,
            beta=beta,
            initial_state=initial_state,
            output_final_state=output_final_state,
            cu_seqlens=cu_seqlens,
            chunk_indices=chunk_indices,
            chunk_offsets=chunk_offsets,
            use_qk_l2norm_in_kernel=use_qk_l2norm_in_kernel,
            core_attn_out=core_attn_out,
        )
```
**EN:** Defines function `ChunkGatedDeltaRule.forward_native` with signature `forward_native(self, q: torch.Tensor, k: torch.Tensor, v: torch.Tensor, g: torch.Tensor, beta: torch.Tensor, initial_state: torch.Tensor, output_final_state: bool, cu_seqlens: torch.Tensor | None=None, chunk_indices: torch.Tensor | None=None, chunk_offsets: torch.Tensor | None=None, use_qk_l2norm_in_kernel: bool=True, core_attn_out: torch.Tensor | None=None)`. It mainly works with `q`, `k`, `v`, `g`, `beta`, `initial_state`, `output_final_state`, `cu_seqlens`; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include `fla_chunk_gated_delta_rule`.
**CN:** 定义函数 `ChunkGatedDeltaRule.forward_native`，其签名为 `forward_native(self, q: torch.Tensor, k: torch.Tensor, v: torch.Tensor, g: torch.Tensor, beta: torch.Tensor, initial_state: torch.Tensor, output_final_state: bool, cu_seqlens: torch.Tensor | None=None, chunk_indices: torch.Tensor | None=None, chunk_offsets: torch.Tensor | None=None, use_qk_l2norm_in_kernel: bool=True, core_attn_out: torch.Tensor | None=None)`。它主要围绕 `q`, `k`, `v`, `g`, `beta`, `initial_state`, `output_final_state`, `cu_seqlens` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `fla_chunk_gated_delta_rule`。

### Class `GatedDeltaNetAttention` overview (lines 239-1431)
```python
class GatedDeltaNetAttention(PluggableLayer, MambaBase):
    @property
    def mamba_type(self) -> MambaAttentionBackendEnum:
        return MambaAttentionBackendEnum.GDN_ATTN

    def get_state_dtype(self) -> tuple[torch.dtype, torch.dtype]:
        return MambaStateDtypeCalculator.gated_delta_net_state_dtype(
            self.model_config.dtype,
            self.cache_config.mamba_cache_dtype,
            self.cache_config.mamba_ssm_cache_dtype,
        )

    def get_state_shape(self) -> tuple[tuple[int, ...], tuple[int, ...]]:
        return MambaStateShapeCalculator.gated_delta_net_state_shape(
            self.tp_size,
            self.num_k_heads,
            self.num_v_heads,
            self.head_k_dim,
            self.head_v_dim,
            self.conv_kernel_size,
            self.num_spec,
        )

    def __init__(
        self,
```
**EN:** Defines class `GatedDeltaNetAttention` with base classes `PluggableLayer`, `MambaBase` and decorators `PluggableLayer.register('gated_delta_net_attention')`. It acts as an attention-related layer or kernel wrapper and exposes 20 direct methods, with notable entries `mamba_type`, `get_state_dtype`, `get_state_shape`, `__init__`, `create_qkvz_proj`, `create_ba_proj`.
**CN:** 定义类 `GatedDeltaNetAttention`，其基类为 `PluggableLayer`, `MambaBase`，装饰器为 `PluggableLayer.register('gated_delta_net_attention')`。它在整体实现中充当与注意力相关的层或内核封装，并直接暴露 20 个方法，较重要的包括 `mamba_type`, `get_state_dtype`, `get_state_shape`, `__init__`, `create_qkvz_proj`, `create_ba_proj`。

### Method `GatedDeltaNetAttention.__init__` (lines 262-408)
```python
    def __init__(
        self,
        config: Qwen3NextConfig,
        vllm_config: VllmConfig,
        prefix: str = "",
        gqa_interleaved_layout=False,
    ) -> None:
        super().__init__()
        self.tp_size = get_tensor_model_parallel_world_size()
        self.tp_rank = get_tensor_model_parallel_rank()
        self.hidden_size = config.hidden_size
        self.num_v_heads = config.linear_num_value_heads
        self.num_k_heads = config.linear_num_key_heads
        self.head_k_dim = config.linear_key_head_dim
        self.head_v_dim = config.linear_value_head_dim
        self.key_dim = self.head_k_dim * self.num_k_heads
        self.value_dim = self.head_v_dim * self.num_v_heads

        self.conv_kernel_size = config.linear_conv_kernel_dim
        self.layer_idx = extract_layer_index(prefix)
        self.activation = config.hidden_act
        self.act = ACT2FN[config.hidden_act]
        self.layer_norm_epsilon = config.rms_norm_eps
        self.prefix = prefix
        self.config = config
        self.model_config = vllm_config.model_config
        self.cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config
        self.speculative_config = vllm_config.speculative_config
        self.num_spec = (
            self.speculative_config.num_speculative_tokens
            if self.speculative_config
            else 0
        )
        self.gqa_interleaved_layout = gqa_interleaved_layout
        if current_platform.is_xpu():
            self._forward_method = self.forward_xpu
        elif current_platform.is_cpu():
# ... truncated for analysis ...
            quant_config=quant_config,
            prefix=f"{prefix}.out_proj",
        )

        self.chunk_gated_delta_rule = ChunkGatedDeltaRule()
        self._prefill_kernels_warmed_up = False
        self.enable_packed_recurrent_decode = (
            envs.VLLM_ENABLE_FLA_PACKED_RECURRENT_DECODE
        )

        compilation_config = get_current_vllm_config().compilation_config
        if prefix in compilation_config.static_forward_context:
            raise ValueError(f"Duplicate layer name: {prefix}")
        compilation_config.static_forward_context[prefix] = self
```
**EN:** Defines function `GatedDeltaNetAttention.__init__` with signature `__init__(self, config: Qwen3NextConfig, vllm_config: VllmConfig, prefix: str='', gqa_interleaved_layout=False) -> None`. It mainly works with `config`, `vllm_config`, `prefix`, `gqa_interleaved_layout`; initializes the object state and cached resources. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `super.__init__`, `get_tensor_model_parallel_world_size`, `get_tensor_model_parallel_rank`, `extract_layer_index`, `current_platform.is_xpu`, `ColumnParallelLinear`.
**CN:** 定义函数 `GatedDeltaNetAttention.__init__`，其签名为 `__init__(self, config: Qwen3NextConfig, vllm_config: VllmConfig, prefix: str='', gqa_interleaved_layout=False) -> None`。它主要围绕 `config`, `vllm_config`, `prefix`, `gqa_interleaved_layout` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `super.__init__`, `get_tensor_model_parallel_world_size`, `get_tensor_model_parallel_rank`, `extract_layer_index`, `current_platform.is_xpu`, `ColumnParallelLinear`。

### Method `GatedDeltaNetAttention.forward_cuda` (lines 723-783)
```python
    def forward_cuda(
        self,
        hidden_states: torch.Tensor,
        output: torch.Tensor,
    ):
        """
        Forward pass with three parts:
        1. Input projection
        2. Core attention (custom op)
        3. Output projection
        """
        num_tokens = hidden_states.size(0)
        # ============================================================
        # Part 1: Input Projection
        # ============================================================
        mixed_qkvz, _ = self.in_proj_qkvz(hidden_states)
        ba, _ = self.in_proj_ba(hidden_states)

        if self.gqa_interleaved_layout:
            # Qwen3-Next: unpack the interleaved GQA layout
            query, key, value, z, b, a = self.fix_query_key_value_ordering(
                mixed_qkvz, ba
            )
            query, key, value = map(
                lambda x: rearrange(x, "l p d -> l (p d)"), (query, key, value)
            )
            mixed_qkv = torch.cat((query, key, value), dim=-1)
        else:
            # Qwen3.5: weights are already in [q, k, v, z] and [b, a] order
            qkv_size = (self.key_dim * 2 + self.value_dim) // self.tp_size
            z_size = self.value_dim // self.tp_size
            mixed_qkv, z = mixed_qkvz.split([qkv_size, z_size], dim=-1)
            z = z.reshape(z.size(0), -1, self.head_v_dim)
            b, a = ba.chunk(2, dim=-1)
            b = b.contiguous()
            a = a.contiguous()

        # ============================================================
# ... truncated for analysis ...

        torch.ops.vllm.gdn_attention_core(
            mixed_qkv,
            b,
            a,
            core_attn_out,
            fast_kernel=False,
            layer_name=_encode_layer_name(self.prefix),
        )

        # ============================================================
        # Part 3: Output Projection
        # ============================================================
        self._output_projection(core_attn_out, z, output, num_tokens)
```
**EN:** Defines function `GatedDeltaNetAttention.forward_cuda` with signature `forward_cuda(self, hidden_states: torch.Tensor, output: torch.Tensor)`. It mainly works with `hidden_states`, `output`; implements one step of the Mamba/SSM execution path. The body uses branching, tensor/kernel operations. Key calls include `hidden_states.size`, `self.in_proj_qkvz`, `self.in_proj_ba`, `torch.zeros`, `torch.ops.vllm.gdn_attention_core`, `self._output_projection`.
**CN:** 定义函数 `GatedDeltaNetAttention.forward_cuda`，其签名为 `forward_cuda(self, hidden_states: torch.Tensor, output: torch.Tensor)`。它主要围绕 `hidden_states`, `output` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断、张量或内核操作。关键调用包括 `hidden_states.size`, `self.in_proj_qkvz`, `self.in_proj_ba`, `torch.zeros`, `torch.ops.vllm.gdn_attention_core`, `self._output_projection`。

### Method `GatedDeltaNetAttention.forward_xpu` (lines 785-832)
```python
    def forward_xpu(
        self,
        hidden_states: torch.Tensor,
        output: torch.Tensor,
    ):
        """
        Forward pass with three parts:
        1. Input projection
        2. Core attention (custom op)
        3. Output projection
        """
        num_tokens = hidden_states.size(0)

        # ============================================================
        # Part 1: Input Projection
        # ============================================================
        projected_states_qkvz, _ = self.in_proj_qkvz(hidden_states)
        projected_states_ba, _ = self.in_proj_ba(hidden_states)

        # ============================================================
        # Part 2: Core Attention
        # ============================================================
        core_attn_out = torch.zeros(
            (num_tokens, self.num_v_heads // self.tp_size, self.head_v_dim),
            dtype=hidden_states.dtype,
            device=hidden_states.device,
        )
        z = torch.empty_like(core_attn_out)

        torch.ops.vllm.gdn_attention_core_xpu(
            core_attn_out,
            z,
            projected_states_qkvz,
            projected_states_ba,
            self.prefix,
        )

        # ============================================================
        # Part 3: Output Projection
        # ============================================================
        z_shape_og = z.shape
        # Reshape input data into 2D tensor
        core_attn_out = core_attn_out.reshape(-1, core_attn_out.shape[-1])
        z = z.reshape(-1, z.shape[-1])
        core_attn_out = self.norm(core_attn_out, z)
        core_attn_out = core_attn_out.reshape(z_shape_og)
        core_attn_out = core_attn_out.flatten(-2)  # ... h d -> ... (h d)
        output[:num_tokens], _ = self.out_proj(core_attn_out)
```
**EN:** Defines function `GatedDeltaNetAttention.forward_xpu` with signature `forward_xpu(self, hidden_states: torch.Tensor, output: torch.Tensor)`. It mainly works with `hidden_states`, `output`; implements one step of the Mamba/SSM execution path. The body uses tensor/kernel operations. Key calls include `hidden_states.size`, `self.in_proj_qkvz`, `self.in_proj_ba`, `torch.zeros`, `torch.empty_like`, `torch.ops.vllm.gdn_attention_core_xpu`.
**CN:** 定义函数 `GatedDeltaNetAttention.forward_xpu`，其签名为 `forward_xpu(self, hidden_states: torch.Tensor, output: torch.Tensor)`。它主要围绕 `hidden_states`, `output` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含张量或内核操作。关键调用包括 `hidden_states.size`, `self.in_proj_qkvz`, `self.in_proj_ba`, `torch.zeros`, `torch.empty_like`, `torch.ops.vllm.gdn_attention_core_xpu`。

### Method `GatedDeltaNetAttention.forward_cpu` (lines 834-882)
```python
    def forward_cpu(
        self,
        hidden_states: torch.Tensor,
        output: torch.Tensor,
    ):
        assert not hasattr(self, "in_proj_qkv"), "lora isn't supported on CPU."

        mixed_qkvz, _ = self.in_proj_qkvz(hidden_states)
        ba, _ = self.in_proj_ba(hidden_states)

        if self.gqa_interleaved_layout:
            # Qwen3-Next: unpack the interleaved GQA layout
            query, key, value, z, b, a = self.fix_query_key_value_ordering(
                mixed_qkvz, ba
            )
            query, key, value = map(
                lambda x: rearrange(x, "l p d -> l (p d)"), (query, key, value)
            )
            mixed_qkv = torch.cat((query, key, value), dim=-1)
        else:
            # Qwen3.5: weights are already in [q, k, v, z] and [b, a] order
            qkv_size = (self.key_dim * 2 + self.value_dim) // self.tp_size
            z_size = self.value_dim // self.tp_size
            mixed_qkv, z = mixed_qkvz.split([qkv_size, z_size], dim=-1)
            z = z.reshape(z.size(0), -1, self.head_v_dim)
            b, a = ba.chunk(2, dim=-1)

        num_tokens = hidden_states.size(0)
        core_attn_out = torch.zeros(
            (num_tokens, self.num_v_heads // self.tp_size, self.head_v_dim),
            dtype=hidden_states.dtype,
            device=hidden_states.device,
        )

        torch.ops.vllm.cpu_gdn_attention_core(
            mixed_qkv,
            b,
            a,
            core_attn_out,
            _encode_layer_name(self.prefix),
        )

        z_shape_og = z.shape
        core_attn_out = core_attn_out.reshape(-1, core_attn_out.shape[-1])
        z = z.reshape(-1, z.shape[-1])
        core_attn_out = self.norm(core_attn_out, z)
        core_attn_out = core_attn_out.reshape(z_shape_og)
        core_attn_out = core_attn_out.flatten(-2)  # ... h d -> ... (h d)
        output[:num_tokens], _ = self.out_proj(core_attn_out)
```
**EN:** Defines function `GatedDeltaNetAttention.forward_cpu` with signature `forward_cpu(self, hidden_states: torch.Tensor, output: torch.Tensor)`. It mainly works with `hidden_states`, `output`; implements one step of the Mamba/SSM execution path. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `self.in_proj_qkvz`, `self.in_proj_ba`, `hidden_states.size`, `torch.zeros`, `torch.ops.vllm.cpu_gdn_attention_core`, `core_attn_out.reshape`.
**CN:** 定义函数 `GatedDeltaNetAttention.forward_cpu`，其签名为 `forward_cpu(self, hidden_states: torch.Tensor, output: torch.Tensor)`。它主要围绕 `hidden_states`, `output` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `self.in_proj_qkvz`, `self.in_proj_ba`, `hidden_states.size`, `torch.zeros`, `torch.ops.vllm.cpu_gdn_attention_core`, `core_attn_out.reshape`。

## Key Concepts / 关键概念
- **EN:** The file belongs to the Mamba/SSM path, so recurrent state updates, scan-style kernels, or short convolutions are recurring themes.
  **CN:** 该文件位于 Mamba/状态空间模型路径中，因此循环状态更新、scan 风格内核或短卷积是反复出现的主题。
- **EN:** Top-level classes include `ChunkGatedDeltaRule`, `GatedDeltaNetAttention`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `ChunkGatedDeltaRule`, `GatedDeltaNetAttention`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `fi_chunk_gated_delta_rule`, `gdn_attention_core`, `gdn_attention_core_fake`, `fused_gdn_gating_kernel`, `fused_gdn_gating` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `fi_chunk_gated_delta_rule`, `gdn_attention_core`, `gdn_attention_core_fake`, `fused_gdn_gating_kernel`, `fused_gdn_gating` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `torch`, `einops`, `transformers`
- **Internal / 内部**: `vllm`, `vllm._aiter_ops`, `vllm.config`, `vllm.distributed`, `vllm.forward_context`, `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.fla.ops`, `vllm.model_executor.layers.fla.ops.chunk`, `vllm.model_executor.layers.fla.ops.utils`, `vllm.model_executor.layers.layernorm`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.mamba.abstract`, `vllm.model_executor.layers.mamba.mamba_mixer2`, `vllm.model_executor.layers.mamba.mamba_utils`, `vllm.model_executor.layers.mamba.ops.causal_conv1d`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.model_loader.weight_utils`, `vllm.model_executor.models.utils`, `vllm.model_executor.utils`
