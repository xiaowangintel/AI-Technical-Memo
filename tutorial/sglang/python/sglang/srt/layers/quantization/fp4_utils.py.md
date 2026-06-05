# fp4_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/fp4_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains supporting code for fp4 utils quantization in the SGLang SRT layer stack. / 该模块包含 SGLang SRT 层栈中与 FP4 工具 量化相关的支撑代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: module imports and setup
```python
from __future__ import annotations

import logging
from enum import Enum
from typing import TYPE_CHECKING, Optional

import torch

from sglang.srt.utils.common import is_sm100_supported, is_sm120_supported
from sglang.srt.utils.custom_op import register_custom_op_from_extern
```
**EN:** This block imports __future__, enum, flashinfer, logging, sglang.srt.server_args, sglang.srt.utils.common, sglang.srt.utils.custom_op and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, enum, flashinfer, logging, sglang.srt.server_args, sglang.srt.utils.common, sglang.srt.utils.custom_op 等依赖，并为当前量化实现准备模块命名空间。

### Lines 12-13: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.server_args import ServerArgs
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 15-15: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 18-18: initialize fp4_quantize
```python
fp4_quantize = None
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as fp4_quantize.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 fp4_quantize。

### Lines 19-86: guarded import or fallback path
```python
try:
    from flashinfer import fp4_quantize as _flashinfer_fp4_quantize

    _flashinfer_fp4_quantize_backend = "cute-dsl" if is_sm100_supported() else "cuda"

    def _round_up(x: int, y: int) -> int:
        return ((x + y - 1) // y) * y

    def _flashinfer_fp4_quantize_impl(
        input: torch.Tensor,
        global_scale: Optional[torch.Tensor] = None,
        sf_vec_size: int = 16,
        sf_use_ue8m0: bool = False,
        is_sf_swizzled_layout: bool = True,
        is_sf_8x4_layout: bool = False,
        enable_pdl: Optional[bool] = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        return _flashinfer_fp4_quantize(
            input=input,
            global_scale=global_scale,
            sf_vec_size=sf_vec_size,
            sf_use_ue8m0=sf_use_ue8m0,
            is_sf_swizzled_layout=is_sf_swizzled_layout,
            is_sf_8x4_layout=is_sf_8x4_layout,
            enable_pdl=enable_pdl,
            backend=_flashinfer_fp4_quantize_backend,
        )

    def _flashinfer_fp4_quantize_fake(
        input: torch.Tensor,
        global_scale: Optional[torch.Tensor] = None,
        sf_vec_size: int = 16,
        sf_use_ue8m0: bool = False,
        is_sf_swizzled_layout: bool = True,
        is_sf_8x4_layout: bool = False,
        enable_pdl: Optional[bool] = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        is_column_major = input.stride(-2) == 1
        if is_column_major:
            m = input.shape[-1]
            K = input.shape[-2]
        else:
            m = input.numel() // input.shape[-1]
            K = input.shape[-1]
        if is_column_major:
            x_q = input.new_empty((*input.shape[:-2], K // 2, m), dtype=torch.uint8)
        else:
            x_q = input.new_empty((*input.shape[:-1], K // 2), dtype=torch.uint8)
        if is_sf_swizzled_layout:
            row_size = 8 if is_sf_8x4_layout else 128
            sf_rows = _round_up(m, row_size)
            sf_cols = _round_up(K // sf_vec_size, 4)
        else:
            sf_rows = m
            sf_cols = K // sf_vec_size
        if is_column_major:
            sf = input.new_empty((sf_cols, sf_rows), dtype=torch.uint8)
        else:
            sf = input.new_empty((sf_rows, sf_cols), dtype=torch.uint8)
        return x_q, sf

    fp4_quantize = register_custom_op_from_extern(
        _flashinfer_fp4_quantize_impl,
        op_name="flashinfer_fp4_quantize",
        fake_impl=_flashinfer_fp4_quantize_fake,
    )
except ImportError:
    fp4_quantize = None
```
**EN:** This block uses exception handling to provide a fallback implementation when optional functionality is unavailable.
**CN:** 该代码块通过异常处理在可选功能不可用时提供后备实现。

### Lines 89-91: class Fp4GemmRunnerBackend: definition
```python
class Fp4GemmRunnerBackend(Enum):
    """Enum for FP4 GEMM runner backend selection."""
```
**EN:** This block declares `Fp4GemmRunnerBackend`, a supporting class for the quantization stack. It organizes behaviors such as is_auto, is_cutlass, is_flashinfer_cudnn, is_flashinfer_cutlass.
**CN:** 该代码块声明 `Fp4GemmRunnerBackend`，它是量化栈中的支撑类，组织了 is_auto, is_cutlass, is_flashinfer_cudnn, is_flashinfer_cutlass 等行为。

### Lines 92-92: Fp4GemmRunnerBackend member: initialize AUTO
```python
    AUTO = "auto"
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as AUTO.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 AUTO。

### Lines 93-93: Fp4GemmRunnerBackend member: initialize CUTLASS
```python
    CUTLASS = "cutlass"
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as CUTLASS.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 CUTLASS。

### Lines 94-94: Fp4GemmRunnerBackend member: initialize FLASHINFER_CUDNN
```python
    FLASHINFER_CUDNN = "flashinfer_cudnn"
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as FLASHINFER_CUDNN.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 FLASHINFER_CUDNN。

### Lines 95-95: Fp4GemmRunnerBackend member: initialize FLASHINFER_CUTEDSL
```python
    FLASHINFER_CUTEDSL = "flashinfer_cutedsl"
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as FLASHINFER_CUTEDSL.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 FLASHINFER_CUTEDSL。

### Lines 96-96: Fp4GemmRunnerBackend member: initialize FLASHINFER_CUTLASS
```python
    FLASHINFER_CUTLASS = "flashinfer_cutlass"
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as FLASHINFER_CUTLASS.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 FLASHINFER_CUTLASS。

### Lines 97-97: Fp4GemmRunnerBackend member: initialize FLASHINFER_TRTLLM
```python
    FLASHINFER_TRTLLM = "flashinfer_trtllm"
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as FLASHINFER_TRTLLM.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 FLASHINFER_TRTLLM。

### Lines 99-100: Fp4GemmRunnerBackend.is_auto()
```python
    def is_auto(self) -> bool:
        return self == Fp4GemmRunnerBackend.AUTO
```
**EN:** This block defines `Fp4GemmRunnerBackend.is_auto()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `Fp4GemmRunnerBackend.is_auto()`，用于检查供上层控制流使用的条件。

### Lines 102-103: Fp4GemmRunnerBackend.is_cutlass()
```python
    def is_cutlass(self) -> bool:
        return self == Fp4GemmRunnerBackend.CUTLASS
```
**EN:** This block defines `Fp4GemmRunnerBackend.is_cutlass()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `Fp4GemmRunnerBackend.is_cutlass()`，用于检查供上层控制流使用的条件。

### Lines 105-106: Fp4GemmRunnerBackend.is_flashinfer_cudnn()
```python
    def is_flashinfer_cudnn(self) -> bool:
        return self == Fp4GemmRunnerBackend.FLASHINFER_CUDNN
```
**EN:** This block defines `Fp4GemmRunnerBackend.is_flashinfer_cudnn()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `Fp4GemmRunnerBackend.is_flashinfer_cudnn()`，用于检查供上层控制流使用的条件。

### Lines 108-109: Fp4GemmRunnerBackend.is_flashinfer_cutlass()
```python
    def is_flashinfer_cutlass(self) -> bool:
        return self == Fp4GemmRunnerBackend.FLASHINFER_CUTLASS
```
**EN:** This block defines `Fp4GemmRunnerBackend.is_flashinfer_cutlass()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `Fp4GemmRunnerBackend.is_flashinfer_cutlass()`，用于检查供上层控制流使用的条件。

### Lines 111-112: Fp4GemmRunnerBackend.is_flashinfer_trtllm()
```python
    def is_flashinfer_trtllm(self) -> bool:
        return self == Fp4GemmRunnerBackend.FLASHINFER_TRTLLM
```
**EN:** This block defines `Fp4GemmRunnerBackend.is_flashinfer_trtllm()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `Fp4GemmRunnerBackend.is_flashinfer_trtllm()`，用于检查供上层控制流使用的条件。

### Lines 114-115: Fp4GemmRunnerBackend.is_flashinfer_cutedsl()
```python
    def is_flashinfer_cutedsl(self) -> bool:
        return self == Fp4GemmRunnerBackend.FLASHINFER_CUTEDSL
```
**EN:** This block defines `Fp4GemmRunnerBackend.is_flashinfer_cutedsl()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `Fp4GemmRunnerBackend.is_flashinfer_cutedsl()`，用于检查供上层控制流使用的条件。

### Lines 117-118: Fp4GemmRunnerBackend.is_flashinfer()
```python
    def is_flashinfer(self) -> bool:
        return self.value.startswith("flashinfer_")
```
**EN:** This block defines `Fp4GemmRunnerBackend.is_flashinfer()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `Fp4GemmRunnerBackend.is_flashinfer()`，用于检查供上层控制流使用的条件。

### Lines 120-135: Fp4GemmRunnerBackend.get_flashinfer_backend()
```python
    def get_flashinfer_backend(self) -> str:
        """Get the backend string to pass to FlashInfer's mm_fp4 API.

        This remaps SGLang's user-facing backend names to FlashInfer's API names.
        Examples:
            'flashinfer_trtllm' -> 'trtllm'
            'flashinfer_cutlass' -> 'cutlass'
            'flashinfer_cudnn' -> 'cudnn'
            'flashinfer_cutedsl' -> 'cute-dsl'
        """
        if self == Fp4GemmRunnerBackend.FLASHINFER_CUTEDSL:
            return "cute-dsl"
        if self.value.startswith("flashinfer_"):
            return self.value.removeprefix("flashinfer_")
        else:
            return self.value
```
**EN:** This block defines `Fp4GemmRunnerBackend.get_flashinfer_backend()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `Fp4GemmRunnerBackend.get_flashinfer_backend()`，用于为调用方获取或计算派生值。

### Lines 138-138: annotate FP4_GEMM_RUNNER_BACKEND
```python
FP4_GEMM_RUNNER_BACKEND: Fp4GemmRunnerBackend | None = None
```
**EN:** This block declares and initializes the annotated symbol `FP4_GEMM_RUNNER_BACKEND`.
**CN:** 该代码块声明并初始化带类型注解的符号 `FP4_GEMM_RUNNER_BACKEND`。

### Lines 141-157: initialize_fp4_gemm_config()
```python
def initialize_fp4_gemm_config(server_args: ServerArgs) -> None:
    """Initialize FP4 GEMM configuration from server args."""
    global FP4_GEMM_RUNNER_BACKEND

    backend = server_args.fp4_gemm_runner_backend
    if backend == "auto":
        if is_sm120_supported():
            # flashinfer_cutlass produces NaN in dense MLP layers with
            # heterogeneous batches on SM120 (Blackwell).  cudnn is stable.
            # See: https://github.com/sgl-project/sglang/issues/20043
            backend = "flashinfer_cudnn"
        elif is_sm100_supported():
            backend = "flashinfer_cutedsl"
        else:
            backend = "flashinfer_cutlass"

    FP4_GEMM_RUNNER_BACKEND = Fp4GemmRunnerBackend(backend)
```
**EN:** This block defines `initialize_fp4_gemm_config()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `initialize_fp4_gemm_config()`，用于实现量化栈中的可复用模块逻辑。

### Lines 160-165: get_fp4_gemm_runner_backend()
```python
def get_fp4_gemm_runner_backend() -> Fp4GemmRunnerBackend:
    """Get the current FP4 GEMM runner backend."""
    global FP4_GEMM_RUNNER_BACKEND
    if FP4_GEMM_RUNNER_BACKEND is None:
        FP4_GEMM_RUNNER_BACKEND = Fp4GemmRunnerBackend.AUTO
    return FP4_GEMM_RUNNER_BACKEND
```
**EN:** This block defines `get_fp4_gemm_runner_backend()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `get_fp4_gemm_runner_backend()`，用于为调用方获取或计算派生值。

## Key Concepts / 关键概念
- `Fp4GemmRunnerBackend`: A supporting class that structures file-level quantization behavior. / `Fp4GemmRunnerBackend` 是一个支撑类，用于组织该文件中的量化行为。
- `initialize_fp4_gemm_config()` : A public function that implements reusable module logic for the quantization stack. / `initialize_fp4_gemm_config()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `get_fp4_gemm_runner_backend()` : A public function that retrieves or computes a derived value for callers. / `get_fp4_gemm_runner_backend()`：一个公开函数，用于为调用方获取或计算派生值。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `enum`, `flashinfer`, `logging`, `torch`, `typing`
- **Internal / 内部**: `sglang.srt.server_args`, `sglang.srt.utils.common`, `sglang.srt.utils.custom_op`
