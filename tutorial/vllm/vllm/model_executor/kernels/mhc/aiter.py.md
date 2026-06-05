# aiter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/mhc/aiter.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the AITER MHC kernel path and wrapper logic. / 实现 AITER 的 MHC 内核路径与包装逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-5)
```python
import torch

from vllm.utils.torch_utils import direct_register_custom_op
```
**EN:** This import block loads `torch`, `vllm.utils.torch_utils`, establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `aiter.py`.
**CN:** 该导入代码块加载了 `torch`, `vllm.utils.torch_utils`，为 `aiter.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Function `mhc_pre_aiter` (lines 8-55)
```python
def mhc_pre_aiter(
    residual: torch.Tensor,
    fn: torch.Tensor,
    hc_scale: torch.Tensor,
    hc_base: torch.Tensor,
    rms_eps: float,
    hc_pre_eps: float,
    hc_sinkhorn_eps: float,
    hc_post_mult_value: float,
    sinkhorn_repeat: int,
    n_splits: int = 1,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    """
    Forward pass for mHC pre block.

    Args:
        residual: shape (..., hc_mult, hidden_size), dtype torch.bfloat16
        fn: shape (hc_mult3, hc_mult * hidden_size), dtype torch.float32
        hc_scale: shape (3,), dtype torch.float32
        hc_base: shape (hc_mult3,), dtype torch.float32
        rms_eps: RMS normalization epsilon
        hc_pre_eps: pre-mix epsilon
        hc_sinkhorn_eps: sinkhorn epsilon
        hc_post_mult_value: post-mix multiplier value
        sinkhorn_repeat: number of sinkhorn iterations
        n_splits: split-k factor;

    Returns:
        post_mix: shape (..., hc_mult), dtype torch.float32
        comb_mix: shape (..., hc_mult, hc_mult), dtype torch.float32
        layer_input: shape (..., hidden_size), dtype torch.bfloat16
    """

    hidden_size = residual.shape[-1]
    assert hidden_size % 256 == 0
    from vllm._aiter_ops import rocm_aiter_ops

    return rocm_aiter_ops.mhc_pre(
        residual,
        fn,
        hc_scale,
        hc_base,
        rms_eps,
        hc_pre_eps,
        hc_sinkhorn_eps,
        hc_post_mult_value,
        sinkhorn_repeat,
    )
```
**EN:** This helper implements `mhc_pre_aiter`. Forward pass for mHC pre block. Internally it relies on calls such as `mhc_pre`.
**CN:** 该函数 `mhc_pre_aiter` 封装了此模块中的一段关键运行时逻辑，重点处理 mhc pre aiter 相关工作。 它内部会调用 `mhc_pre` 等例程。

### Function `_mhc_pre_aiter_fake` (lines 58-96)
```python
def _mhc_pre_aiter_fake(
    residual: torch.Tensor,
    fn: torch.Tensor,
    hc_scale: torch.Tensor,
    hc_base: torch.Tensor,
    rms_eps: float,
    hc_pre_eps: float,
    hc_sinkhorn_eps: float,
    hc_post_mult_value: float,
    sinkhorn_repeat: int,
    n_splits: int = 1,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    hc_mult = residual.shape[-2]
    hidden_size = residual.shape[-1]
    outer_shape = residual.shape[:-2]

    # Create empty tensors with correct shapes for meta device / shape inference
    post_mix = torch.empty(
        *outer_shape,
        hc_mult,
        1,
        dtype=torch.float32,
        device=residual.device,
    )
    comb_mix = torch.empty(
        *outer_shape,
        hc_mult,
        hc_mult,
        dtype=torch.float32,
        device=residual.device,
    )
    layer_input = torch.empty(
        *outer_shape,
        hidden_size,
        dtype=torch.bfloat16,
        device=residual.device,
    )

    return post_mix, comb_mix, layer_input
```
**EN:** This helper implements `_mhc_pre_aiter_fake`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `empty`.
**CN:** 该函数 `_mhc_pre_aiter_fake` 封装了此模块中的一段关键运行时逻辑，重点处理 mhc pre aiter fake 相关工作。 它内部会调用 `empty` 等例程。

### Function `mhc_post_aiter` (lines 99-115)
```python
def mhc_post_aiter(
    x: torch.Tensor,
    residual: torch.Tensor,
    post_layer_mix: torch.Tensor,
    comb_res_mix: torch.Tensor,
) -> torch.Tensor:
    hidden_size = residual.shape[-1]

    assert hidden_size % 256 == 0
    from vllm._aiter_ops import rocm_aiter_ops

    return rocm_aiter_ops.mhc_post(
        x,
        residual,
        post_layer_mix,
        comb_res_mix,
    )
```
**EN:** This helper implements `mhc_post_aiter`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `mhc_post`.
**CN:** 该函数 `mhc_post_aiter` 封装了此模块中的一段关键运行时逻辑，重点处理 mhc post aiter 相关工作。 它内部会调用 `mhc_post` 等例程。

### Function `_mhc_post_aiter_fake` (lines 118-124)
```python
def _mhc_post_aiter_fake(
    x: torch.Tensor,
    residual: torch.Tensor,
    post_layer_mix: torch.Tensor,
    comb_res_mix: torch.Tensor,
) -> torch.Tensor:
    return torch.empty_like(residual)
```
**EN:** This helper implements `_mhc_post_aiter_fake`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `empty_like`.
**CN:** 该函数 `_mhc_post_aiter_fake` 封装了此模块中的一段关键运行时逻辑，重点处理 mhc post aiter fake 相关工作。 它内部会调用 `empty_like` 等例程。

### Top-level block (lines 127-132)
```python
direct_register_custom_op(
    op_name="mhc_pre_aiter",
    op_func=mhc_pre_aiter,
    mutates_args=[],
    fake_impl=_mhc_pre_aiter_fake,
)
```
**EN:** This top-level control-flow block coordinates module initialization before the main kernels are used.
**CN:** 这个顶层控制流代码块在主内核使用前协调模块初始化流程。

### Top-level block (lines 133-138)
```python
direct_register_custom_op(
    op_name="mhc_post_aiter",
    op_func=mhc_post_aiter,
    mutates_args=[],
    fake_impl=_mhc_post_aiter_fake,
)
```
**EN:** This top-level control-flow block coordinates module initialization before the main kernels are used.
**CN:** 这个顶层控制流代码块在主内核使用前协调模块初始化流程。

## Key Concepts / 关键概念
- **AITER backend / AITER 后端**
  - **EN:** The module routes execution through AITER custom operators or kernel helpers.
  - **CN:** 该模块通过 AITER 自定义算子或内核辅助逻辑完成执行。
- **ROCm support / ROCm 支持**
  - **EN:** The implementation targets ROCm-capable devices and their kernel constraints.
  - **CN:** 该实现面向 ROCm 设备及其内核约束。
- **MHC kernels / MHC 内核**
  - **EN:** The implementation provides backend-specific MHC kernels and wrappers.
  - **CN:** 该实现提供面向不同后端的 MHC 内核与包装层。
- **Kernel registration / 内核注册**
  - **EN:** The file registers operators, backends, or preset configurations with PyTorch/vLLM.
  - **CN:** 该文件向 PyTorch/vLLM 注册算子、后端或预设配置。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm.utils.torch_utils`
- **External / 外部依赖**: `torch`
