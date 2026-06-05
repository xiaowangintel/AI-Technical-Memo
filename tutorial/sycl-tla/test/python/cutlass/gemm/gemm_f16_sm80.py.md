# gemm_f16_sm80.py — Code Analysis / 代码分析

## Source / 来源

- Path / 路径: `test/python/cutlass/gemm/gemm_f16_sm80.py`
- **EN:** Low-level SM80 FP16 GEMM coverage spanning TensorOp kernels, SIMT fallbacks, and StreamK swizzled variants through dynamic test registration.
- **CN:** 该文件通过动态注册的方式，覆盖 SM80 上的 FP16 GEMM：既包含 TensorOp 内核，也包含 SIMT 回退路径和 StreamK swizzle 变体。

## Line-by-Line Analysis / 逐行分析

### Lines 37-49

```python
from functools import partial
import logging
import unittest

import cutlass_cppgen
from cutlass_cppgen.backend.utils.device import device_cc

from utils import LayoutCombination, add_test_gemm


cutlass_cppgen.set_log_level(logging.WARNING)
cc = 80
dtype = cutlass_cppgen.DataType.f16
```
**EN:** The module imports the same registration infrastructure used across the GEMM tests, then fixes the target architecture to SM80 and the operand type to FP16. Logging is reduced to warnings so large generated test suites stay quiet unless something fails.

**CN:** 模块首先导入整套 GEMM 测试注册基础设施，然后把目标架构固定为 SM80，把操作数类型固定为 FP16。日志级别被限制为 warning，以避免大规模动态生成测试时输出过多噪声。

### Lines 51-67

```python
@unittest.skipIf(device_cc() < cc, 'Device compute capability is insufficient for SM80 tests.')
@unittest.skipIf(cutlass_cppgen.utils.datatypes.torch_type(dtype) is None, f'Version of torch installed does not contain a datatype match for {dtype}')
class GemmF16Sm80(unittest.TestCase):
    """
    Wrapper class to which tests will be added dynamically in __main__
    """
    pass


@unittest.skipIf(device_cc() < cc, 'Device compute capability is insufficient for SM80 tests.')
@unittest.skipIf(cutlass_cppgen.utils.datatypes.torch_type(dtype) is None, f'Version of torch installed does not contain a datatype match for {dtype}')
class GemmF16Sm80StreamK(unittest.TestCase):
    """
    Wrapper class to which tests will be added dynamically in __main__
    """
    pass
```
**EN:** There are two wrapper classes: one for ordinary SM80 tests and one dedicated to StreamK variants. Both classes are gated by the same conditions: the current GPU must have compute capability at least 80, and the installed PyTorch must expose an FP16 type that CUTLASS recognizes.

**CN:** 这里定义了两个包装类：一个承载普通 SM80 测试，另一个专门承载 StreamK 变体。两个类都受同样的条件限制：当前 GPU 的计算能力至少为 80，并且安装的 PyTorch 必须提供 CUTLASS 能识别的 FP16 类型。

### Lines 68-71

```python
add_test_specialized = partial(add_test_gemm, element=dtype, cc=cc, cluster_shape=[1, 1, 1])

# Tests using TensorOp
add_test_tensorop = partial(add_test_specialized, opclass=cutlass_cppgen.OpcodeClass.TensorOp)
```
**EN:** `add_test_specialized` captures the cross-cutting defaults (`element`, `cc`, `cluster_shape`), and `add_test_tensorop` narrows that baseline to Tensor Core execution. This keeps the long registration list below compact and readable.

**CN:** `add_test_specialized` 先固化跨测试共享的默认参数（`element`、`cc`、`cluster_shape`），而 `add_test_tensorop` 再把这一基线收缩到 Tensor Core 执行方式。这样下面很长的注册列表就能保持相对紧凑和可读。

### Lines 73-105

```python
add_test_tensorop(cls=GemmF16Sm80, layouts=LayoutCombination.NNN, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
                  element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 32], warp_count=[2, 2, 1], stages=3)
add_test_tensorop(cls=GemmF16Sm80, layouts=LayoutCombination.NNT, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
                  element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 32], warp_count=[2, 2, 1], stages=3)
add_test_tensorop(cls=GemmF16Sm80, layouts=LayoutCombination.NTN, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
                  element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 32], warp_count=[2, 2, 1], stages=3)
add_test_tensorop(cls=GemmF16Sm80, layouts=LayoutCombination.NTT, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
                  element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 32], warp_count=[2, 2, 1], stages=3)
add_test_tensorop(cls=GemmF16Sm80, layouts=LayoutCombination.TNN, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
                  element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 32], warp_count=[2, 2, 1], stages=3)
add_test_tensorop(cls=GemmF16Sm80, layouts=LayoutCombination.TNT, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
                  element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 32], warp_count=[2, 2, 1], stages=3)
add_test_tensorop(cls=GemmF16Sm80, layouts=LayoutCombination.TTN, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
                  element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 32], warp_count=[2, 2, 1], stages=3)
add_test_tensorop(cls=GemmF16Sm80, layouts=LayoutCombination.TTT, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
                  element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 32], warp_count=[2, 2, 1], stages=3)
add_test_tensorop(cls=GemmF16Sm80, layouts=LayoutCombination.TNT, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
                  element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[ 64, 128, 32], warp_count=[1, 2, 1], stages=3)
add_test_tensorop(cls=GemmF16Sm80, layouts=LayoutCombination.TNT, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
                  element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128,  64, 32], warp_count=[2, 1, 1], stages=3)
add_test_tensorop(cls=GemmF16Sm80, layouts=LayoutCombination.TNT, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
                  element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[ 64,  64, 64], warp_count=[1, 1, 1], stages=3)
add_test_tensorop(cls=GemmF16Sm80, layouts=LayoutCombination.TNT, alignments=[4, 4, 8], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
                  element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 32], warp_count=[2, 2, 1], stages=3)
add_test_tensorop(cls=GemmF16Sm80, layouts=LayoutCombination.TNT, alignments=[4, 4, 8], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
                  element_accumulator=cutlass_cppgen.DataType.f16, threadblock_shape=[128, 128, 32], warp_count=[2, 2, 1], stages=3)
add_test_tensorop(cls=GemmF16Sm80, layouts=LayoutCombination.TNT, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
                  element_accumulator=cutlass_cppgen.DataType.f16, threadblock_shape=[128, 128, 32], warp_count=[2, 2, 1], stages=3)
add_test_tensorop(cls=GemmF16Sm80, layouts=LayoutCombination.TNT, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
                  element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[ 64,  64, 64], warp_count=[1, 1, 1], stages=5)
add_test_tensorop(cls=GemmF16Sm80, layouts=LayoutCombination.TNT, alignments=[2, 2, 2], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
                  element_accumulator=cutlass_cppgen.DataType.f16, threadblock_shape=[128, 128, 32], warp_count=[2, 2, 1], stages=3)
```
**EN:** This is the main TensorOp coverage block. The first eight registrations enumerate every layout combination from `NNN` through `TTT` with the same `128x128x32` tile, `warp_count=[2,2,1]`, and FP32 accumulation. The later `TNT`-focused registrations then probe alternative tile shapes, alignment tuples (`[4,4,8]`, `[2,2,2]`), accumulator types (FP32 versus FP16), and stage counts (`3` or `5`). Together they stress both layout handling and kernel-shape tuning.

**CN:** 这是主要的 TensorOp 覆盖块。前八次注册在相同的 `128x128x32` tile、`warp_count=[2,2,1]` 和 FP32 累加设置下，枚举了从 `NNN` 到 `TTT` 的全部布局组合。后面的若干 `TNT` 定向注册则进一步探测不同的 tile 形状、对齐组合（如 `[4,4,8]`、`[2,2,2]`）、累加类型（FP32 与 FP16）以及阶段数（`3` 或 `5`）。整体上同时覆盖了布局处理与内核参数调优。

### Lines 106-119

```python
# Tests using SIMT
add_test_simt = partial(add_test_specialized, opclass=cutlass_cppgen.OpcodeClass.Simt)

add_test_simt(cls=GemmF16Sm80, layouts=LayoutCombination.NNN, alignments=[1, 1, 1], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
              element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 8], warp_count=[2, 2, 1], stages=2)
add_test_simt(cls=GemmF16Sm80, layouts=LayoutCombination.TNN, alignments=[1, 1, 1], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
              element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[ 64, 128, 8], warp_count=[1, 2, 1], stages=2)
add_test_simt(cls=GemmF16Sm80, layouts=LayoutCombination.NTN, alignments=[1, 1, 1], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
              element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128,  64, 8], warp_count=[2, 1, 1], stages=2)
add_test_simt(cls=GemmF16Sm80, layouts=LayoutCombination.TTN, alignments=[1, 1, 1], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
              element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[ 64,  64, 8], warp_count=[1, 1, 1], stages=2)
add_test_simt(cls=GemmF16Sm80, layouts=LayoutCombination.NNT, alignments=[1, 1, 1], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
              element_accumulator=cutlass_cppgen.DataType.f16, threadblock_shape=[128, 128, 8], warp_count=[2, 2, 1], stages=2)
```
**EN:** The SIMT section swaps Tensor Cores out for standard scalar/SIMT execution. All alignments drop to `[1,1,1]`, K-dimension tiles shrink to `8`, and the cases cover several layout combinations, including one `NNT` case that accumulates in FP16 instead of FP32.

**CN:** SIMT 部分把 Tensor Core 替换成普通标量/SIMT 执行路径。所有对齐都降为 `[1,1,1]`，K 维 tile 缩小到 `8`，并覆盖若干布局组合；其中有一个 `NNT` 用例会使用 FP16 而非 FP32 进行累加。

### Lines 121-125

```python
add_test_streamk = partial(add_test_specialized, opclass=cutlass_cppgen.OpcodeClass.TensorOp, swizzle=cutlass_cppgen.swizzle.ThreadblockSwizzleStreamK)
add_test_streamk(cls=GemmF16Sm80StreamK, layouts=LayoutCombination.NNN, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
                 element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 32], warp_count=[2, 2, 1], stages=3)
add_test_streamk(cls=GemmF16Sm80StreamK, layouts=LayoutCombination.TNT, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16, element_C=cutlass_cppgen.DataType.f16,
                 element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[ 64,  64, 64], warp_count=[1, 1, 1], stages=5)
```
**EN:** The StreamK section keeps TensorOp execution but changes the swizzle to `ThreadblockSwizzleStreamK`. Only two representative cases are registered, making this a targeted smoke test for StreamK scheduling rather than an exhaustive matrix of configurations.

**CN:** StreamK 部分仍然采用 TensorOp 执行，但把 swizzle 改成 `ThreadblockSwizzleStreamK`。这里只注册了两个代表性用例，因此它更像是对 StreamK 调度的定向冒烟测试，而不是穷举所有配置。

### Lines 127-128

```python
if __name__ == '__main__':
    unittest.main()
```
**EN:** The standard `unittest.main()` hook closes the file.

**CN:** 文件最后以标准的 `unittest.main()` 入口收尾。

## Key Concepts / 关键概念

- **EN:** Three execution styles are covered: TensorOp, SIMT, and TensorOp+StreamK.
  **CN:** 文件覆盖了三种执行风格：TensorOp、SIMT，以及 TensorOp+StreamK。
- **EN:** The registration list is intentionally dense because the real test bodies are generated by `add_test_gemm`.
  **CN:** 注册列表之所以很密集，是因为真正的测试函数体由 `add_test_gemm` 动态生成。
- **EN:** Alignment, tile shape, warp decomposition, accumulator type, and pipeline stages are all treated as first-class test parameters.
  **CN:** 对齐、tile 形状、warp 拆分、累加类型和流水线阶段数都被当作一等测试参数来覆盖。
- **EN:** SM80 gating ensures the suite only runs where these kernel families are expected to exist.
  **CN:** SM80 的硬件门控保证这些内核家族只在预期支持的平台上运行。

## Dependencies / 依赖关系

- **EN:** External: `cutlass_cppgen` provides FP16 datatypes, `OpcodeClass`, and the StreamK swizzle functor.
  **CN:** 外部依赖：`cutlass_cppgen` 提供 FP16 数据类型、`OpcodeClass` 以及 StreamK swizzle functor。
- **EN:** Hardware check: `device_cc` decides whether the active GPU is new enough for SM80 kernels.
  **CN:** 硬件检查依赖：`device_cc` 用来判断当前 GPU 是否足以支持 SM80 内核。
- **EN:** Internal helpers: `LayoutCombination` enumerates row/column-major triples and `add_test_gemm` attaches generated tests to the wrapper classes.
  **CN:** 内部依赖：`LayoutCombination` 枚举 A/B/C 的行列主序组合，`add_test_gemm` 则把生成的测试挂接到包装类上。
- **EN:** Standard library: `partial`, `logging`, and `unittest` support configuration reuse, log suppression, and test discovery.
  **CN:** 标准库依赖：`partial`、`logging` 和 `unittest` 分别支持配置复用、日志抑制与测试发现。
