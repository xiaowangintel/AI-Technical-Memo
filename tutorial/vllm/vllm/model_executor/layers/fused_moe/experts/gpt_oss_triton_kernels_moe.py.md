# gpt_oss_triton_kernels_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/experts/gpt_oss_triton_kernels_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: backend-specific expert kernels and wrappers; fused Mixture-of-Experts routing, kernels, and runtime helpers / 特定后端的专家内核与封装；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-34 — imports and setup
```python
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm import _custom_ops as ops
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.config import (
    FUSED_MOE_UNQUANTIZED_CONFIG,
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
    RoutingMethodType,
)
from vllm.model_executor.layers.fused_moe.experts.lora_experts_mixin import (
    LoRAExpertsMixin,
)
from vllm.model_executor.layers.fused_moe.topk_weight_and_reduce import (
    TopKWeightAndReduceNoOP,
)
from vllm.model_executor.layers.fused_moe.utils import _resize_cache
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kMxfp4Static,
)
from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
from vllm.utils.import_utils import has_triton_kernels

from ..utils import swiglu_limit_func

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 37-58 — function `_triton_kernel_moe_supports_current_device`
```python
def _triton_kernel_moe_supports_current_device() -> bool:
    # Shared device gate for the OAI Triton MoE expert classes.
    # Platform-aware to avoid ROCm capability aliasing — cap (9, 0)
    # matches both gfx90a (verified) and gfx906 (unverified), so we
    # dispatch on gfx-string helpers instead of the cap tuple on ROCm.
    p = current_platform
    if p.is_cuda():
        cap = p.get_device_capability()
        # Keep the original `(9, 0) <= cap < (11, 0)` window on
        # CUDA (covers Hopper SM90 and Blackwell SM100, excludes
        # SM120) — this PR is ROCm-scoped and the broader CUDA
        # range was not validated.
        return cap is not None and (9, 0) <= (cap.major, cap.minor) < (11, 0)
    if p.is_rocm():
        from vllm.platforms.rocm import on_gfx1x, on_gfx9

        # gfx9 family: gfx90a (MI200), gfx942/gfx950 (MI3xx);
        # on_gfx9() already excludes gfx906/gfx908.
        # gfx1x family: gfx11xx (RDNA3/3.5) and gfx12xx (RDNA4);
        # on_gfx1x() excludes gfx10xx (RDNA1/RDNA2).
        return on_gfx9() or on_gfx1x()
    return False
```
**EN:** This function defines `_triton_kernel_moe_supports_current_device`. It implements a low-level kernel that works on tiled tensor blocks. Key calls include `p.is_cuda`, `p.is_rocm`, `p.get_device_capability`, `on_gfx9`, `on_gfx1x`. It writes or updates `p`, `cap`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_triton_kernel_moe_supports_current_device`。 它实现了按分块张量执行的底层内核。 关键调用包括 `p.is_cuda`, `p.is_rocm`, `p.get_device_capability`, `on_gfx9`, `on_gfx1x`。 它会写入或更新 `p`, `cap`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 61-209 — function `_patch_make_bitmatrix_metadata`
```python
def _patch_make_bitmatrix_metadata() -> None:
    """Monkey-patch make_bitmatrix_metadata to support non-power-of-2 top_k.

    triton's tl.arange requires a power-of-2 range.  The original kernel
    computes BLOCK_SIZE = BLOCK_PER_TOK * TOKS_PER_ROW (= 32 * top_k).  For
    DeepSeek-V4 with top_k=6 this gives 192, which is not a power of 2 and
    causes a compile error at the first forward pass.

    Fix: define a drop-in replacement kernel that accepts an extra constexpr
    BLOCK_SIZE_PADDED (next power of 2 >= BLOCK_SIZE) and uses it for the
    tl.arange call while keeping the actual BLOCK_SIZE as the stride between
    thread-blocks so that all flat indices into NonzeroIndx stay correct.
    Elements beyond BLOCK_SIZE are masked out (col_indx = 0xffff) and ignored.

    This function is called once at module load time and patches the function
    inside the triton_kernels tensor module so that SparseMatrix.__post_init__
    picks up the fixed version transparently.
    """
    import torch
    import triton
    import triton.language as tl

    try:
        from vllm.third_party.triton_kernels.tensor_details import (
            bitmatrix as _bm,
        )
        from vllm.third_party.triton_kernels.tensor_details.bitmatrix import (
            BitmatrixMetadata,
            _keyed_add,
            cdiv,
        )
        from vllm.third_party.triton_kernels.tensor_details.bitmatrix_details.sum_bitmatrix_rows import (  # noqa: E501
            sum_bitmatrix_rows,
        )
    except ImportError:
        return

    @triton.jit
    def _stage2_pow2(
        ColSortedIndx,
        RowSortedIndx,
        NonzeroIndx,
        n_tokens,
        ColPartialSum,
# ... omitted for brevity ...
    # Also patch the bitmatrix module itself in case it is imported directly.
    _bm.make_bitmatrix_metadata = _make_bitmatrix_metadata_pow2_safe
```
**EN:** This function defines `_patch_make_bitmatrix_metadata`. Monkey-patch make_bitmatrix_metadata to support non-power-of-2 top_k. Key calls include `tl.static_assert`, `tl.program_id`, `tl.arange`, `tl.load.to`, `to`, `tl.sort`. It writes or updates `make_bitmatrix_metadata`, `BLOCK_SIZE`, `nonzero_indx_size`, `pid_m`, `offs_local`, `offs_global`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_patch_make_bitmatrix_metadata`。 该函数/方法的文档字符串直接说明了它的职责。 关键调用包括 `tl.static_assert`, `tl.program_id`, `tl.arange`, `tl.load.to`, `to`, `tl.sort`。 它会写入或更新 `make_bitmatrix_metadata`, `BLOCK_SIZE`, `nonzero_indx_size`, `pid_m`, `offs_local`, `offs_global`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 251-294 — function `pack_bitmatrix`
```python
@triton.jit
def pack_bitmatrix(
    bitmatrix,
    topk_ids,
    n_rows,  # n_rows in bitmatrix / topk_ids
    bm_cols: tl.constexpr,  # n int32_t bitpacks in bitmatrix
    n_expts_act,  # num_topk
    BLOCK_SIZE_M: tl.constexpr,
    BLOCK_SIZE_K: tl.constexpr,
):
    """
    Packs topk_ids into a bitmatrix.
    code reference:
    https://github.com/triton-lang/triton/blob/dd1bbc52b34d202dfe5ffea1e04fb16166c5c04e/python/triton_kernels/bench/distributed.py#L264
    """
    pid_m = tl.program_id(0)
    offsets_m = pid_m * BLOCK_SIZE_M + tl.arange(0, BLOCK_SIZE_M)
    offsets_k = tl.arange(0, BLOCK_SIZE_K)
    offsets = offsets_m[:, None] * n_expts_act + offsets_k[None, :]
    mask = (offsets_m < n_rows)[:, None] & (offsets_k < n_expts_act)[None, :]
    indices = tl.load(topk_ids + offsets, mask=mask, other=-1)
    valid = indices >= 0
    div = indices // 32
    rem = indices % 32
    one = tl.cast(1, tl.uint32)

    # Iterate through all the relevant bitmatrix columns.
    for i in range(bm_cols):
        # When BLOCK_SIZE_K=32, offs is just the column index.
        offs = tl.arange(0, BLOCK_SIZE_K // 32) + i * (BLOCK_SIZE_K // 32)
        # All topks that need to go into this column has the correct bit set.
        # Other bits are 0. x is a 2D tensor.
        # Guard with `valid` to prevent negative indices from producing
        # spurious bits (on HIP, -1 // 32 == 0 and 1 << (-1 % 32) sets
        # bit 31).
        x = tl.where(
            valid[:, :, None] & (div[:, :, None] == offs[None, None, :]),
            (one << rem)[:, :, None],
            0,
        )
        # Reduce x to get a single int32_t bitpack.
        y = tl.reduce_or(x, axis=1)
        bitmatrix_ptrs = bitmatrix + offsets_m[:, None] * bm_cols + offs[None, :]
        tl.store(bitmatrix_ptrs, y, mask=offsets_m[:, None] < n_rows)
```
**EN:** This function defines `pack_bitmatrix`. Packs topk_ids into a bitmatrix. The main inputs are `bitmatrix`, `topk_ids`, `n_rows`, `bm_cols`, `n_expts_act`, `BLOCK_SIZE_M`. Key calls include `tl.program_id`, `tl.arange`, `tl.load`, `tl.cast`, `range`, `tl.where`. It writes or updates `pid_m`, `offsets_m`, `offsets_k`, `offsets`, `mask`, `indices`. The body uses loops to cover different runtime cases.
**CN:** 该函数定义 `pack_bitmatrix`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `bitmatrix`, `topk_ids`, `n_rows`, `bm_cols`, `n_expts_act`, `BLOCK_SIZE_M`。 关键调用包括 `tl.program_id`, `tl.arange`, `tl.load`, `tl.cast`, `range`, `tl.where`。 它会写入或更新 `pid_m`, `offsets_m`, `offsets_k`, `offsets`, `mask`, `indices`。 函数体通过循环来覆盖不同的运行时场景。

### Lines 297-366 — function `triton_kernel_moe_forward`
```python
def triton_kernel_moe_forward(
    hidden_states: torch.Tensor,
    w1,  # Tensor or triton_kernels.Tensor
    w2,  # Tensor or triton_kernels.Tensor
    gating_output: torch.Tensor,
    topk: int,
    renormalize: bool,
    activation: MoEActivation = MoEActivation.SWIGLUOAI,
    quant_config: FusedMoEQuantConfig | None = None,
    apply_router_weight_on_input: bool = False,
    global_num_experts: int = -1,
    expert_map: torch.Tensor | None = None,
    unpadded_N_w1=None,
    unpadded_K_w1=None,
    unpadded_N_w2=None,
    unpadded_K_w2=None,
) -> torch.Tensor:
    from triton_kernels.topk import topk as topk_fn

    sm_first = not renormalize
    logits = gating_output
    if sm_first:
        logits = torch.softmax(logits, dim=-1)
    topk_result = topk_fn(logits, topk, apply_softmax=not sm_first)
    # topk may return a tuple (vals, indx, bitmatrix) or a
    # SparseMatrix depending on the triton_kernels version.
    if isinstance(topk_result, tuple):
        topk_weights, topk_ids_raw, _ = topk_result
    else:
        topk_weights = topk_result.vals
        topk_ids_raw = topk_result.indx

    if expert_map is not None:
        # topk_ids_raw contains global expert IDs - remap to local.
        topk_ids = expert_map[topk_ids_raw.to(torch.long)]
        local_num_experts = w1.shape[0]
        routing_data, gather_idx, scatter_idx = make_routing_data(
            topk_ids, topk_weights, local_num_experts
        )
        # expert_map already applied; pass None downstream.
        effective_expert_map = None
        effective_global_num_experts = local_num_experts
    else:
        topk_ids = topk_ids_raw.to(torch.long)
# ... omitted for brevity ...
        expert_map=effective_expert_map,
    )
```
**EN:** This function defines `triton_kernel_moe_forward`. It executes the main forward/runtime path for this component. The main inputs are `hidden_states`, `w1`, `w2`, `gating_output`, `topk`, `renormalize`. Key calls include `topk_fn`, `isinstance`, `torch.empty_like`, `triton_kernel_fused_experts`, `torch.softmax`, `make_routing_data`. It writes or updates `sm_first`, `logits`, `topk_result`, `output`, `effective_quant_config`, `topk_weights`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `triton_kernel_moe_forward`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `hidden_states`, `w1`, `w2`, `gating_output`, `topk`, `renormalize`。 关键调用包括 `topk_fn`, `isinstance`, `torch.empty_like`, `triton_kernel_fused_experts`, `torch.softmax`, `make_routing_data`。 它会写入或更新 `sm_first`, `logits`, `topk_result`, `output`, `effective_quant_config`, `topk_weights`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 370-467 — function `triton_kernel_fused_experts`
```python
def triton_kernel_fused_experts(
    output_tensor: torch.Tensor,
    hidden_states: torch.Tensor,
    w1,  # Tensor or triton_kernels.Tensor
    w2,  # Tensor or triton_kernels.Tensor
    routing_data,  # RoutingData
    gather_indx,  # GatherIndx
    scatter_indx,  # ScatterIndx
    topk: int,
    activation: MoEActivation = MoEActivation.SWIGLUOAI,
    quant_config: FusedMoEQuantConfig | None = None,
    swiglu_alpha: float = 1.702,
    swiglu_limit: float = 7.0,
    apply_router_weight_on_input: bool = False,
    global_num_experts: int = -1,
    expert_map: torch.Tensor | None = None,
    intermediate_cache: torch.Tensor | None = None,
    a1q_scale: torch.Tensor | None = None,
) -> torch.Tensor:
    """Triton implementation of fused expert computation using OAI kernels."""
    assert activation == MoEActivation.SWIGLUOAI, (
        "Only SWIGLUOAI activation is supported"
    )
    assert quant_config is not None

    # type check, uint8 means mxfp4
    assert hidden_states.dtype == torch.bfloat16
    assert quant_config.w1_bias is None or quant_config.w1_bias.dtype == torch.float32
    assert quant_config.w2_bias is None or quant_config.w2_bias.dtype == torch.float32

    # Shape check, only check non-mxfp4
    assert hidden_states.ndim == 2
    assert hidden_states.shape[-1] == w1.shape[-2]
    assert w2.shape[-1] == w1.shape[1]

    batch_dim = 1
    M, K = hidden_states.shape[-2:]
    E, _, N = w1.shape

    if global_num_experts == -1:
        global_num_experts = E

    if intermediate_cache is None:
        intermediate_cache = torch.empty(
# ... omitted for brevity ...
    output_tensor = output_tensor.view(M, K)
    return output_tensor
```
**EN:** This function defines `triton_kernel_fused_experts`. Triton implementation of fused expert computation using OAI kernels. The main inputs are `output_tensor`, `hidden_states`, `w1`, `w2`, `routing_data`, `gather_indx`. Key calls include `_resize_cache`, `matmul_ogs`, `output_tensor.view`, `torch.empty`, `FusedActivation`, `intermediate_cache.view`. It writes or updates `batch_dim`, `M`, `K`, `E`, `_`, `N`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `triton_kernel_fused_experts`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `output_tensor`, `hidden_states`, `w1`, `w2`, `routing_data`, `gather_indx`。 关键调用包括 `_resize_cache`, `matmul_ogs`, `output_tensor.view`, `torch.empty`, `FusedActivation`, `intermediate_cache.view`。 它会写入或更新 `batch_dim`, `M`, `K`, `E`, `_`, `N`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 470-541 — function `make_routing_data`
```python
def make_routing_data(
    topk_ids: torch.Tensor,
    topk_weights: torch.Tensor,
    num_local_experts: int,
) -> tuple["RoutingData", torch.Tensor, torch.Tensor]:
    topk_ids = topk_ids.to(torch.int16)
    topk_weights = topk_weights.to(torch.bfloat16)

    n_rows, num_topk = topk_ids.size()

    BLOCK_SIZE_M = 512
    BLOCK_SIZE_K = 32

    bm_cols = triton.cdiv(num_local_experts, BLOCK_SIZE_K)  # n_bitpacks
    bitmatrix = torch.zeros(
        (n_rows, bm_cols), dtype=torch.uint32, device=topk_ids.device
    )

    grid = (triton.cdiv(n_rows, BLOCK_SIZE_M),)
    pack_bitmatrix[grid](
        bitmatrix,
        topk_ids,
        n_rows,
        bm_cols,
        num_topk,
        BLOCK_SIZE_M=BLOCK_SIZE_M,
        BLOCK_SIZE_K=BLOCK_SIZE_K,
    )

    bitmatrix_shape = [n_rows, bm_cols * 32]
    bitmatrix_shape_max = [n_rows, None]
    bitmatrix = (
        Bitmatrix(
            bitmatrix, dtype=BIT, shape=bitmatrix_shape, shape_max=bitmatrix_shape_max
        )
        if not use_legacy_triton_kernels
        else Bitmatrix(
            bitmatrix,
            shape=bitmatrix_shape,
            shape_max=bitmatrix_shape_max,
            scratchpad=None,
        )
    )

# ... omitted for brevity ...
    scatter_indx = ScatterIndx(dispatch_indx, combine_indx)
    return routing_data, gather_indx, scatter_indx
```
**EN:** This function defines `make_routing_data`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `topk_ids`, `topk_weights`, `num_local_experts`. Key calls include `topk_ids.to`, `topk_weights.to`, `topk_ids.size`, `triton.cdiv`, `torch.zeros`, `pack_bitmatrix`. It writes or updates `topk_ids`, `topk_weights`, `n_rows`, `num_topk`, `BLOCK_SIZE_M`, `BLOCK_SIZE_K`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `make_routing_data`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `topk_ids`, `topk_weights`, `num_local_experts`。 关键调用包括 `topk_ids.to`, `topk_weights.to`, `topk_ids.size`, `triton.cdiv`, `torch.zeros`, `pack_bitmatrix`。 它会写入或更新 `topk_ids`, `topk_weights`, `n_rows`, `num_topk`, `BLOCK_SIZE_M`, `BLOCK_SIZE_K`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 544-624 — class `BaseOAITritonExperts`
```python
class BaseOAITritonExperts(mk.FusedMoEExpertsModular):
    @property
    def expects_unquantized_inputs(self) -> bool:
        return True

    @staticmethod
    def _supports_current_device() -> bool:
        return _triton_kernel_moe_supports_current_device() and has_triton_kernels()

    @staticmethod
    def _supports_no_act_and_mul() -> bool:
        return False

    @staticmethod
# ... omitted for brevity ...
    ) -> tuple["RoutingData", torch.Tensor, torch.Tensor]:
        return make_routing_data(topk_ids, topk_weights, num_local_experts)
```
**EN:** This class defines `BaseOAITritonExperts`. It inherits from `mk.FusedMoEExpertsModular`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Important methods include `expects_unquantized_inputs`, `supports_expert_map`, `moe_problem_size`, `finalize_weight_and_reduce_impl`. Key calls include `a1.size`, `topk_ids.size`, `TopKWeightAndReduceNoOP`, `make_routing_data`, `_triton_kernel_moe_supports_current_device`, `has_triton_kernels`. It writes or updates `SUPPORTED_W_A`, `E`, `_`, `N`, `K`, `M`.
**CN:** 该类定义了 `BaseOAITritonExperts`。 它继承自 `mk.FusedMoEExpertsModular`。 它管理专家分发、融合内核或 MoE 特有的张量流。 重要方法包括 `expects_unquantized_inputs`, `supports_expert_map`, `moe_problem_size`, `finalize_weight_and_reduce_impl`。 关键调用包括 `a1.size`, `topk_ids.size`, `TopKWeightAndReduceNoOP`, `make_routing_data`, `_triton_kernel_moe_supports_current_device`, `has_triton_kernels`。 它会写入或更新 `SUPPORTED_W_A`, `E`, `_`, `N`, `K`, `M`。

### Lines 545-547 — method `BaseOAITritonExperts.expects_unquantized_inputs`
```python
    @property
    def expects_unquantized_inputs(self) -> bool:
        return True
```
**EN:** This method defines `expects_unquantized_inputs`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `expects_unquantized_inputs`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 575-576 — method `BaseOAITritonExperts.supports_expert_map`
```python
    def supports_expert_map(self) -> bool:
        return True
```
**EN:** This method defines `supports_expert_map`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `supports_expert_map`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 578-612 — method `BaseOAITritonExperts.moe_problem_size`
```python
    def moe_problem_size(
        self,
        a1: torch.Tensor,
        w1: torch.Tensor,
        w2: torch.Tensor,
        topk_ids: torch.Tensor,
    ) -> tuple[int, int, int, int, int]:
        """
        Extract the MoE problem size from the given tensor arguments:
        - a: The hidden states, input to the MoE layer.
        - w1: The first set of expert weights.
        - w2: The second set of expert weights.
        - topk_ids: The topk ids.
        Note: extracting the problem shape from the weight and activation
        tensors is not obvious.  It needs to be done this way specifically
        due to subtle issues with particular kernels, e.g. the int4 kernels
        divide the trailing dimension by two, so it's not "correct" to
        extract N or K from the trailing dimension of w1 or w2.  Similarly,
        some kernels transpose the weights, so this needs to be kept in mind.
        Note: This implementation covers most cases. However, if experts
        require a specialized implementation, like MarlinExperts, they are free
        to override this function.
        """
        assert len(w1.shape) == 3 and len(w2.shape) == 3
        E, _, N = w1.shape
        K = a1.size(-1)

        assert a1.dim() == 2
        assert topk_ids.size(0) == a1.size(0), f"{topk_ids.size(0)} != {a1.size(0)}"
        M = a1.size(0)

        assert topk_ids.dim() == 2
        topk = topk_ids.size(1)

        return E, M, N, K, topk
```
**EN:** This method defines `moe_problem_size`. Extract the MoE problem size from the given tensor arguments: - a: The hidden states, input to the MoE layer. The main inputs are `a1`, `w1`, `w2`, `topk_ids`. Key calls include `a1.size`, `topk_ids.size`, `a1.dim`, `topk_ids.dim`, `len`. It writes or updates `E`, `_`, `N`, `K`, `M`, `topk`.
**CN:** 该方法定义 `moe_problem_size`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `a1`, `w1`, `w2`, `topk_ids`。 关键调用包括 `a1.size`, `topk_ids.size`, `a1.dim`, `topk_ids.dim`, `len`。 它会写入或更新 `E`, `_`, `N`, `K`, `M`, `topk`。

### Lines 614-616 — method `BaseOAITritonExperts.finalize_weight_and_reduce_impl`
```python
    def finalize_weight_and_reduce_impl(self) -> mk.TopKWeightAndReduce:
        # Weight application and reduction happens in the fused_experts kernel.
        return TopKWeightAndReduceNoOP()
```
**EN:** This method defines `finalize_weight_and_reduce_impl`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Key calls include `TopKWeightAndReduceNoOP`.
**CN:** 该方法定义 `finalize_weight_and_reduce_impl`。 它管理专家分发、融合内核或 MoE 特有的张量流。 关键调用包括 `TopKWeightAndReduceNoOP`。

### Lines 627-705 — class `OAITritonExperts`
```python
class OAITritonExperts(BaseOAITritonExperts):
    """OAI Triton-based fused MoE expert implementation."""

    @staticmethod
    def _supports_activation(activation: MoEActivation) -> bool:
        return activation == MoEActivation.SWIGLUOAI

    @staticmethod
    def activation_format() -> mk.FusedMoEActivationFormat:
        return mk.FusedMoEActivationFormat.Standard

    def workspace_shapes(
        self,
        M: int,
# ... omitted for brevity ...
            a1q_scale=a1q_scale,
        )
```
**EN:** This class defines `OAITritonExperts`. It inherits from `BaseOAITritonExperts`. OAI Triton-based fused MoE expert implementation. Important methods include `apply`, `activation_format`, `workspace_shapes`. Key calls include `self.adjust_N_for_activation`, `self._make_routing_data`, `topk_ids.size`, `triton_kernel_fused_experts`. It writes or updates `activation_out_dim`, `workspace1`, `workspace2`, `output`, `local_num_experts`, `routing_data`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `OAITritonExperts`。 它继承自 `BaseOAITritonExperts`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `apply`, `activation_format`, `workspace_shapes`。 关键调用包括 `self.adjust_N_for_activation`, `self._make_routing_data`, `topk_ids.size`, `triton_kernel_fused_experts`。 它会写入或更新 `activation_out_dim`, `workspace1`, `workspace2`, `output`, `local_num_experts`, `routing_data`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 656-705 — method `OAITritonExperts.apply`
```python
    def apply(
        self,
        output: torch.Tensor,
        hidden_states: torch.Tensor,
        w1: torch.Tensor,
        w2: torch.Tensor,
        topk_weights: torch.Tensor,
        topk_ids: torch.Tensor,
        activation: MoEActivation,
        global_num_experts: int,
        expert_map: torch.Tensor | None,
        a1q_scale: torch.Tensor | None,
        a2_scale: torch.Tensor | None,
        workspace13: torch.Tensor,
        workspace2: torch.Tensor,
        expert_tokens_meta: mk.ExpertTokensMetadata | None,
        apply_router_weight_on_input: bool,
    ):
        if self.quant_config is None:
            self.quant_config: FusedMoEQuantConfig = FUSED_MOE_UNQUANTIZED_CONFIG

        if expert_map is not None:
            topk_ids = expert_map[topk_ids]

        local_num_experts = w1.shape[0]
        if global_num_experts == -1:
            global_num_experts = local_num_experts

        routing_data, gather_indx, scatter_indx = self._make_routing_data(
            topk_ids, topk_weights, local_num_experts
        )

        topk = topk_ids.size(1)
        triton_kernel_fused_experts(
            output,
            hidden_states,
            w1,
            w2,
            routing_data,
            gather_indx,
            scatter_indx,
            topk=topk,
            activation=activation,
            quant_config=self.quant_config,
# ... omitted for brevity ...
            a1q_scale=a1q_scale,
        )
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`. Key calls include `self._make_routing_data`, `topk_ids.size`, `triton_kernel_fused_experts`. It writes or updates `local_num_experts`, `routing_data`, `gather_indx`, `scatter_indx`, `topk`, `quant_config`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `output`, `hidden_states`, `w1`, `w2`, `topk_weights`, `topk_ids`。 关键调用包括 `self._make_routing_data`, `topk_ids.size`, `triton_kernel_fused_experts`。 它会写入或更新 `local_num_experts`, `routing_data`, `gather_indx`, `scatter_indx`, `topk`, `quant_config`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Backend-specific expert kernels and wrappers / [CN] 特定后端的专家内核与封装
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `_triton_kernel_moe_supports_current_device`, `_patch_make_bitmatrix_metadata`, `pack_bitmatrix`, `triton_kernel_moe_forward` / [CN] 核心符号：`_triton_kernel_moe_supports_current_device`, `_patch_make_bitmatrix_metadata`, `pack_bitmatrix`, `triton_kernel_moe_forward`

## Dependencies / 依赖关系
- **External**: `torch`, `utils` / **外部依赖**: `torch`, `utils`
- **Internal**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.lora_experts_mixin`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms` / **内部依赖**: `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.lora_experts_mixin`, `vllm.model_executor.layers.fused_moe.topk_weight_and_reduce`, `vllm.model_executor.layers.fused_moe.utils`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`
- **Runtime traits**: platform-aware dispatch, custom C++/CUDA ops, Triton kernels, distributed collectives / **运行时特征**: platform-aware dispatch, custom C++/CUDA ops, Triton kernels, distributed collectives
