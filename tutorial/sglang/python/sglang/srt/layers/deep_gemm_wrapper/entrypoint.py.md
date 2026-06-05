# entrypoint.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/deep_gemm_wrapper/entrypoint.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements linear projections and GEMM backend integration for the SGLang SRT runtime. It exposes symbols such as `grouped_gemm_nt_f8f8bf16_masked`, `_ensure_cuda`, `grouped_gemm_nt_bf16_masked`, and `grouped_gemm_nt_f8f8bf16_contig` and connects them to backend-specific paths such as `CUDA` and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了线性投影与 GEMM 后端集成。它提供了 `grouped_gemm_nt_f8f8bf16_masked`、`_ensure_cuda`、`grouped_gemm_nt_bf16_masked` 以及 `grouped_gemm_nt_f8f8bf16_contig` 等符号，并把这些符号连接到 `CUDA` 和 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Imports, conditional backend setup, and runtime guards
```python
import logging
from contextlib import contextmanager
from typing import Any, Optional, Tuple

import torch

from sglang.srt.environ import envs
from sglang.srt.layers.deep_gemm_wrapper import compile_utils
from sglang.srt.layers.deep_gemm_wrapper.configurer import (  # noqa: F401
    DEEPGEMM_BLACKWELL,
    DEEPGEMM_NEED_TMA_ALIGNED_SCALES,
    DEEPGEMM_SCALE_UE8M0,
    ENABLE_JIT_DEEPGEMM,
)
from sglang.srt.server_args import ServerArgs

logger = logging.getLogger(__name__)

if ENABLE_JIT_DEEPGEMM:
    import deep_gemm
    from deep_gemm.utils.layout import get_mn_major_tma_aligned_tensor  # noqa: F401

_SANITY_CHECK = envs.SGLANG_DEEPGEMM_SANITY_CHECK.get()


# TODO maybe rename these functions
```
**EN:** This section prepares the module namespace. It imports `logging`, `contextlib.contextmanager`, `typing.Any`, `typing.Optional`, `typing.Tuple`, and `torch`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `logger` and `_SANITY_CHECK` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `logging`、`contextlib.contextmanager`、`typing.Any`、`typing.Optional`、`typing.Tuple` 以及 `torch`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `logger` 和 `_SANITY_CHECK` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 27-79: Function `grouped_gemm_nt_f8f8bf16_masked` and its core logic
```python
def grouped_gemm_nt_f8f8bf16_masked(
    lhs: Tuple[torch.Tensor, torch.Tensor],
    rhs: Tuple[torch.Tensor, torch.Tensor],
    out: torch.Tensor,
    masked_m: torch.Tensor,
    expected_m: int,
    overlap_args: Optional[Any] = None,
    max_block_n: int = 256,
    recipe_a: Optional[Tuple[int, int]] = None,
    recipe_b: Optional[Tuple[int, int]] = None,
):
    num_groups, _, k = lhs[0].shape
    _, n, _ = rhs[0].shape
    kernel_type = compile_utils.DeepGemmKernelType.GROUPED_GEMM_NT_F8F8BF16_MASKED

    _sanity_check_input(lhs)
    _sanity_check_input(rhs)

    lhs = _ensure_cuda(lhs)
    rhs = _ensure_cuda(rhs)

    with compile_utils.deep_gemm_execution_hook(
        expected_m, n, k, num_groups, kernel_type
    ):
        with configure_deep_gemm_num_sms(
            overlap_args.num_sms if overlap_args is not None else None
        ):

            fp4_kwargs = {}
            if recipe_a is not None:
                fp4_kwargs["recipe_a"] = recipe_a
            if recipe_b is not None:
                fp4_kwargs["recipe_b"] = recipe_b

            return deep_gemm.fp8_m_grouped_gemm_nt_masked(
                lhs,
                rhs,
                out,
                masked_m,
                expected_m,
                **fp4_kwargs,
                **(
                    dict(
                        enable_overlap=True,
                        max_block_n=max_block_n,
                        signal=overlap_args.signal,
                    )
                    if overlap_args is not None
                    else {}
                ),
            )
```
**EN:** This block defines `grouped_gemm_nt_f8f8bf16_masked` and contains the main logic for this step. It mainly invokes `_sanity_check_input`, `_ensure_cuda`, `compile_utils.deep_gemm_execution_hook`, `configure_deep_gemm_num_sms`, and `deep_gemm.fp8_m_grouped_gemm_nt_masked`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_groups`, `_`, `k`, `n`, and `kernel_type` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `grouped_gemm_nt_f8f8bf16_masked`，并承载这一阶段的核心逻辑。 它主要调用 `_sanity_check_input`、`_ensure_cuda`、`compile_utils.deep_gemm_execution_hook`、`configure_deep_gemm_num_sms` 以及 `deep_gemm.fp8_m_grouped_gemm_nt_masked`，说明该流程会编排底层辅助函数或计算内核。 像 `num_groups`、`_`、`k`、`n` 以及 `kernel_type` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 80-88: Internal helper `_ensure_cuda`
```python
def _ensure_cuda(
    pair: Tuple[torch.Tensor, torch.Tensor],
) -> Tuple[torch.Tensor, torch.Tensor]:
    return (
        pair[0].cuda() if not pair[0].is_cuda else pair[0],
        pair[1].cuda() if not pair[1].is_cuda else pair[1],
    )
```
**EN:** This block defines `_ensure_cuda` and contains the main logic for this step. It mainly invokes `pair.cuda`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_ensure_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `pair.cuda`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 89-111: Function `grouped_gemm_nt_bf16_masked` and its core logic
```python
def grouped_gemm_nt_bf16_masked(
    a: torch.Tensor,
    b: torch.Tensor,
    d: torch.Tensor,
    masked_m: torch.Tensor,
    expected_m: int,
):
    num_groups, _, k = a.shape
    _, n, _ = b.shape
    kernel_type = compile_utils.DeepGemmKernelType.GROUPED_GEMM_NT_BF16_MASKED

    with compile_utils.deep_gemm_execution_hook(
        expected_m, n, k, num_groups, kernel_type
    ):
        return deep_gemm.m_grouped_bf16_gemm_nt_masked(
            a,
            b,
            d,
            masked_m,
            expected_m,
        )
```
**EN:** This block defines `grouped_gemm_nt_bf16_masked` and contains the main logic for this step. It mainly invokes `compile_utils.deep_gemm_execution_hook` and `deep_gemm.m_grouped_bf16_gemm_nt_masked`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_groups`, `_`, `k`, `n`, and `kernel_type` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `grouped_gemm_nt_bf16_masked`，并承载这一阶段的核心逻辑。 它主要调用 `compile_utils.deep_gemm_execution_hook` 和 `deep_gemm.m_grouped_bf16_gemm_nt_masked`，说明该流程会编排底层辅助函数或计算内核。 像 `num_groups`、`_`、`k`、`n` 以及 `kernel_type` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 112-141: Function `grouped_gemm_nt_f8f8bf16_contig` and its core logic
```python
def grouped_gemm_nt_f8f8bf16_contig(
    lhs: Tuple[torch.Tensor, torch.Tensor],
    rhs: Tuple[torch.Tensor, torch.Tensor],
    out: torch.Tensor,
    m_indices: torch.Tensor,
    recipe_a: Optional[Tuple[int, int]] = None,
    recipe_b: Optional[Tuple[int, int]] = None,
):
    m, k = lhs[0].shape
    num_groups, n, _ = rhs[0].shape
    kernel_type = compile_utils.DeepGemmKernelType.GROUPED_GEMM_NT_F8F8BF16_CONTIG

    if m == 0:
        return

    _sanity_check_input(lhs)
    _sanity_check_input(rhs)

    fp4_kwargs = {}
    if recipe_a is not None:
        fp4_kwargs["recipe_a"] = recipe_a
    if recipe_b is not None:
        fp4_kwargs["recipe_b"] = recipe_b

    with compile_utils.deep_gemm_execution_hook(m, n, k, num_groups, kernel_type):
        deep_gemm.m_grouped_fp8_gemm_nt_contiguous(
            lhs, rhs, out, m_indices, **fp4_kwargs
        )
```
**EN:** This block defines `grouped_gemm_nt_f8f8bf16_contig` and contains the main logic for this step. It mainly invokes `_sanity_check_input`, `compile_utils.deep_gemm_execution_hook`, and `deep_gemm.m_grouped_fp8_gemm_nt_contiguous`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `m`, `k`, `num_groups`, `n`, and `_` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `grouped_gemm_nt_f8f8bf16_contig`，并承载这一阶段的核心逻辑。 它主要调用 `_sanity_check_input`、`compile_utils.deep_gemm_execution_hook` 以及 `deep_gemm.m_grouped_fp8_gemm_nt_contiguous`，说明该流程会编排底层辅助函数或计算内核。 像 `m`、`k`、`num_groups`、`n` 以及 `_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 142-152: Function `grouped_gemm_nt_bf16_contig` and its core logic
```python
def grouped_gemm_nt_bf16_contig(
    a: torch.Tensor, b: torch.Tensor, d: torch.Tensor, m_indices: torch.Tensor
):
    m, k = a.shape
    num_groups, n, _ = b.shape
    kernel_type = compile_utils.DeepGemmKernelType.GROUPED_GEMM_NT_BF16_CONTIG

    with compile_utils.deep_gemm_execution_hook(m, n, k, num_groups, kernel_type):
        deep_gemm.m_grouped_bf16_gemm_nt_contiguous(a, b, d, m_indices)
```
**EN:** This block defines `grouped_gemm_nt_bf16_contig` and contains the main logic for this step. It mainly invokes `compile_utils.deep_gemm_execution_hook` and `deep_gemm.m_grouped_bf16_gemm_nt_contiguous`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `m`, `k`, `num_groups`, `n`, and `_` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `grouped_gemm_nt_bf16_contig`，并承载这一阶段的核心逻辑。 它主要调用 `compile_utils.deep_gemm_execution_hook` 和 `deep_gemm.m_grouped_bf16_gemm_nt_contiguous`，说明该流程会编排底层辅助函数或计算内核。 像 `m`、`k`、`num_groups`、`n` 以及 `_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 153-173: Function `gemm_nt_f8f8bf16` and its core logic
```python
def gemm_nt_f8f8bf16(
    lhs: Tuple[torch.Tensor, torch.Tensor],
    rhs: Tuple[torch.Tensor, torch.Tensor],
    out: torch.Tensor,
):
    m, k = lhs[0].shape
    n, _ = rhs[0].shape
    num_groups = 1
    kernel_type = compile_utils.DeepGemmKernelType.GEMM_NT_F8F8BF16

    _sanity_check_input(lhs)
    _sanity_check_input(rhs)

    with compile_utils.deep_gemm_execution_hook(m, n, k, num_groups, kernel_type):
        deep_gemm.fp8_gemm_nt(
            lhs,
            rhs,
            out,
        )
```
**EN:** This block defines `gemm_nt_f8f8bf16` and contains the main logic for this step. It mainly invokes `_sanity_check_input`, `compile_utils.deep_gemm_execution_hook`, and `deep_gemm.fp8_gemm_nt`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `m`, `k`, `n`, `_`, and `num_groups` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `gemm_nt_f8f8bf16`，并承载这一阶段的核心逻辑。 它主要调用 `_sanity_check_input`、`compile_utils.deep_gemm_execution_hook` 以及 `deep_gemm.fp8_gemm_nt`，说明该流程会编排底层辅助函数或计算内核。 像 `m`、`k`、`n`、`_` 以及 `num_groups` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 174-187: Function `gemm_nt_bf16bf16f32` and its core logic
```python
def gemm_nt_bf16bf16f32(
    lhs: torch.Tensor,
    rhs: torch.Tensor,
    out: torch.Tensor,
):
    m, k = lhs.shape
    n, _ = rhs.shape
    num_groups = 1
    kernel_type = compile_utils.DeepGemmKernelType.GEMM_NT_BF16BF16F32

    with compile_utils.deep_gemm_execution_hook(m, n, k, num_groups, kernel_type):
        deep_gemm.bf16_gemm_nt(lhs, rhs, out)
```
**EN:** This block defines `gemm_nt_bf16bf16f32` and contains the main logic for this step. It mainly invokes `compile_utils.deep_gemm_execution_hook` and `deep_gemm.bf16_gemm_nt`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `m`, `k`, `n`, `_`, and `num_groups` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `gemm_nt_bf16bf16f32`，并承载这一阶段的核心逻辑。 它主要调用 `compile_utils.deep_gemm_execution_hook` 和 `deep_gemm.bf16_gemm_nt`，说明该流程会编排底层辅助函数或计算内核。 像 `m`、`k`、`n`、`_` 以及 `num_groups` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 188-191: `update_deep_gemm_config` step for deep runtime integration GEMM config
```python
def update_deep_gemm_config(gpu_id: int, server_args: ServerArgs):
    compile_utils.update_deep_gemm_config(gpu_id, server_args)
```
**EN:** This block defines `update_deep_gemm_config` and contains the main logic for this step. It mainly invokes `compile_utils.update_deep_gemm_config`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `update_deep_gemm_config`，并承载这一阶段的核心逻辑。 它主要调用 `compile_utils.update_deep_gemm_config`，说明该流程会编排底层辅助函数或计算内核。

### Lines 192-204: Function `configure_deep_gemm_num_sms` and its core logic
```python
@contextmanager
def configure_deep_gemm_num_sms(num_sms):
    if num_sms is None or not ENABLE_JIT_DEEPGEMM:
        yield
    else:
        original_num_sms = deep_gemm.get_num_sms()
        deep_gemm.set_num_sms(num_sms)
        try:
            yield
        finally:
            deep_gemm.set_num_sms(original_num_sms)
```
**EN:** This block defines `configure_deep_gemm_num_sms` and contains the main logic for this step. Decorators like `contextmanager` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `deep_gemm.get_num_sms` and `deep_gemm.set_num_sms`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `original_num_sms` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `configure_deep_gemm_num_sms`，并承载这一阶段的核心逻辑。 像 `contextmanager` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `deep_gemm.get_num_sms` 和 `deep_gemm.set_num_sms`，说明该流程会编排底层辅助函数或计算内核。 像 `original_num_sms` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 205-217: Internal helper `_sanity_check_input`
```python
def _sanity_check_input(x_fp8: Tuple[torch.Tensor, torch.Tensor]):
    if not _SANITY_CHECK:
        return

    x, x_scale = x_fp8

    if x_scale.dtype == torch.int:
        return

    from sglang.srt.layers.quantization.fp8_utils import ceil_to_ue8m0

    x_scale_ceil = ceil_to_ue8m0(x_scale)
    assert torch.all(x_scale == x_scale_ceil), f"{x_scale=} {x_scale_ceil=}"
```
**EN:** This block defines `_sanity_check_input` and contains the main logic for this step. It mainly invokes `ceil_to_ue8m0` and `torch.all`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `x`, `x_scale`, and `x_scale_ceil` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_sanity_check_input`，并承载这一阶段的核心逻辑。 它主要调用 `ceil_to_ue8m0` 和 `torch.all`，说明该流程会编排底层辅助函数或计算内核。 像 `x`、`x_scale` 以及 `x_scale_ceil` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `grouped_gemm_nt_f8f8bf16_masked`, `_ensure_cuda`, `grouped_gemm_nt_bf16_masked`, `grouped_gemm_nt_f8f8bf16_contig`, and `grouped_gemm_nt_bf16_contig`. / **主要符号**：核心入口包括 `grouped_gemm_nt_f8f8bf16_masked`、`_ensure_cuda`、`grouped_gemm_nt_bf16_masked`、`grouped_gemm_nt_f8f8bf16_contig` 以及 `grouped_gemm_nt_bf16_contig`。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Projection layers**: Focuses on matrix multiplication, weight layout, and projection-oriented wrappers. / **投影层**：关注矩阵乘法、权重布局与面向投影的包装层。

## Dependencies / 依赖关系
- **Standard library**: `logging`, `contextlib.contextmanager`, `typing.Any`, `typing.Optional`, and `typing.Tuple` / **标准库**：`logging`、`contextlib.contextmanager`、`typing.Any`、`typing.Optional` 以及 `typing.Tuple`
- **Third-party**: `torch`, `deep_gemm`, and `deep_gemm.utils.layout.get_mn_major_tma_aligned_tensor` / **第三方依赖**：`torch`、`deep_gemm` 以及 `deep_gemm.utils.layout.get_mn_major_tma_aligned_tensor`
- **Internal SGLang modules**: `sglang.srt.environ.envs`, `sglang.srt.layers.deep_gemm_wrapper.compile_utils`, `sglang.srt.layers.deep_gemm_wrapper.configurer.DEEPGEMM_BLACKWELL`, `sglang.srt.layers.deep_gemm_wrapper.configurer.DEEPGEMM_NEED_TMA_ALIGNED_SCALES`, `sglang.srt.layers.deep_gemm_wrapper.configurer.DEEPGEMM_SCALE_UE8M0`, `sglang.srt.layers.deep_gemm_wrapper.configurer.ENABLE_JIT_DEEPGEMM`, `sglang.srt.server_args.ServerArgs`, and `sglang.srt.layers.quantization.fp8_utils.ceil_to_ue8m0` / **SGLang 内部模块**：`sglang.srt.environ.envs`、`sglang.srt.layers.deep_gemm_wrapper.compile_utils`、`sglang.srt.layers.deep_gemm_wrapper.configurer.DEEPGEMM_BLACKWELL`、`sglang.srt.layers.deep_gemm_wrapper.configurer.DEEPGEMM_NEED_TMA_ALIGNED_SCALES`、`sglang.srt.layers.deep_gemm_wrapper.configurer.DEEPGEMM_SCALE_UE8M0`、`sglang.srt.layers.deep_gemm_wrapper.configurer.ENABLE_JIT_DEEPGEMM`、`sglang.srt.server_args.ServerArgs` 以及 `sglang.srt.layers.quantization.fp8_utils.ceil_to_ue8m0`
