# gemm_f64_sm80.py — Code Analysis / 代码分析

## Source / 来源

- Path / 路径: `test/python/cutlass/gemm/gemm_f64_sm80.py`
- **EN:** Low-level SM80 FP64 GEMM registration with TensorOp, SIMT, and StreamK coverage tuned for the heavier cost of double-precision kernels.
- **CN:** 该文件为 SM80 上的 FP64 GEMM 注册低层测试，覆盖 TensorOp、SIMT 与 StreamK，并针对双精度内核更高的代价采用了更保守的 tile 配置。

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
dtype = cutlass_cppgen.DataType.f64
```
**EN:** The shared GEMM-test infrastructure is imported again, but this time the selected scalar type is `f64`. As in similar files, logging is muted to warnings and `cc` is fixed to 80.

**CN:** 这里再次导入共享的 GEMM 测试基础设施，但这次选定的标量类型是 `f64`。和相邻文件一样，日志被限制到 warning，而 `cc` 固定为 80。

### Lines 52-67

```python
@unittest.skipIf(device_cc() < cc, 'Device compute capability is insufficient for SM80 tests.')
@unittest.skipIf(cutlass_cppgen.utils.datatypes.torch_type(dtype) is None, f'Version of torch installed does not contain a datatype match for {dtype}')
class GemmF64Sm80(unittest.TestCase):
    """
    Wrapper class to which tests will be added dynamically in __main__
    """
    pass


@unittest.skipIf(device_cc() < cc, 'Device compute capability is insufficient for SM80 tests.')
@unittest.skipIf(cutlass_cppgen.utils.datatypes.torch_type(dtype) is None, f'Version of torch installed does not contain a datatype match for {dtype}')
class GemmF64Sm80StreamK(unittest.TestCase):
    """
    Wrapper class to which tests will be added dynamically in __main__
    """
    pass
```
**EN:** Two wrapper classes divide the suite into standard kernels and StreamK variants. The decorators ensure the suite is only active on SM80-capable hardware with a PyTorch build that knows how to represent FP64 for these tests.

**CN:** 两个包装类把测试集划分为标准内核与 StreamK 变体。装饰器保证这组测试只会在具备 SM80 能力、且 PyTorch 能正确表示 FP64 的环境中启用。

### Lines 70-73

```python
add_test_specialized = partial(add_test_gemm, element=dtype, cc=cc, cluster_shape=[1, 1, 1])

# Tests using TensorOp
add_test_tensorop = partial(add_test_specialized, opclass=cutlass_cppgen.OpcodeClass.TensorOp)
```
**EN:** The base partial captures the common FP64-on-SM80 settings, and `add_test_tensorop` narrows them to TensorOp kernels. This mirrors the structure of the FP16 and FP32 files.

**CN:** 基础 `partial` 固定了 FP64-on-SM80 的公共配置，`add_test_tensorop` 再把它收窄为 TensorOp 内核。这一组织方式与 FP16、FP32 文件保持一致。

### Lines 75-80

```python
add_test_tensorop(cls=GemmF64Sm80, layouts=LayoutCombination.NNN, alignments=[1, 1, 1], element_output=dtype, element_C=dtype,
                  element_accumulator=dtype, threadblock_shape=[128, 128, 16], warp_count=[4, 2, 1], stages=3)
add_test_tensorop(cls=GemmF64Sm80, layouts=LayoutCombination.NTN, alignments=[1, 1, 1], element_output=dtype, element_C=dtype,
                  element_accumulator=dtype, threadblock_shape=[ 64,  64, 16], warp_count=[2, 2, 1], stages=4)
add_test_tensorop(cls=GemmF64Sm80, layouts=LayoutCombination.TTN, alignments=[1, 1, 1], element_output=dtype, element_C=dtype,
                  element_accumulator=dtype, threadblock_shape=[ 32,  32, 16], warp_count=[2, 1, 1], stages=5)
```
**EN:** Only three TensorOp cases are registered, and their tiles get progressively smaller (`128x128x16`, `64x64x16`, `32x32x16`). That smaller geometry reflects the higher cost and different throughput profile of FP64 GEMM. Every case keeps alignment `[1,1,1]` and uses FP64 throughout.

**CN:** TensorOp 部分只注册了三个用例，而且 tile 会逐步变小（`128x128x16`、`64x64x16`、`32x32x16`）。这种更小的几何规模体现了 FP64 GEMM 计算代价更高、吞吐特性不同。所有用例都保持 `[1,1,1]` 对齐，并在整个路径中统一使用 FP64。

### Lines 82-94

```python
# Tests using SIMT
add_test_simt = partial(add_test_specialized, opclass=cutlass_cppgen.OpcodeClass.Simt)

add_test_simt(cls=GemmF64Sm80, layouts=LayoutCombination.NNN, alignments=[1, 1, 1], element_output=dtype, element_C=dtype,
              element_accumulator=dtype, threadblock_shape=[128, 128, 8], warp_count=[2, 2, 1], stages=2)
add_test_simt(cls=GemmF64Sm80, layouts=LayoutCombination.TNN, alignments=[1, 1, 1], element_output=dtype, element_C=dtype,
              element_accumulator=dtype, threadblock_shape=[ 64, 128, 8], warp_count=[1, 2, 1], stages=2)
add_test_simt(cls=GemmF64Sm80, layouts=LayoutCombination.NTN, alignments=[1, 1, 1], element_output=dtype, element_C=dtype,
              element_accumulator=dtype, threadblock_shape=[128,  64, 8], warp_count=[2, 1, 1], stages=2)
add_test_simt(cls=GemmF64Sm80, layouts=LayoutCombination.TTN, alignments=[1, 1, 1], element_output=dtype, element_C=dtype,
              element_accumulator=dtype, threadblock_shape=[ 64,  64, 8], warp_count=[1, 1, 1], stages=2)
add_test_simt(cls=GemmF64Sm80, layouts=LayoutCombination.NNT, alignments=[1, 1, 1], element_output=dtype, element_C=dtype,
              element_accumulator=dtype, threadblock_shape=[128, 128, 8], warp_count=[2, 2, 1], stages=2)
```
**EN:** The SIMT block adds five fallback cases with the usual `K=8` tiles and several layout combinations. Compared with the TensorOp section, this provides a broader layout sweep but on simpler scalar execution.

**CN:** SIMT 块增加了五个回退用例，沿用常见的 `K=8` tile，并覆盖多种布局组合。相较 TensorOp 部分，这一块在更简单的标量执行路径上提供了更宽的布局覆盖。

### Lines 96-99

```python
# Stream K tests
add_test_streamk = partial(add_test_specialized, opclass=cutlass_cppgen.OpcodeClass.TensorOp, swizzle=cutlass_cppgen.swizzle.ThreadblockSwizzleStreamK)
add_test_streamk(cls=GemmF64Sm80StreamK, layouts=LayoutCombination.NTT, alignments=[1, 1, 1], element_output=dtype, element_C=dtype,
                 element_accumulator=dtype, threadblock_shape=[128, 128, 16], warp_count=[4, 2, 1], stages=3)
```
**EN:** StreamK coverage is reduced to one `NTT` TensorOp case. As with the FP32 file, this is a focused smoke test to ensure the StreamK path exists and runs for double precision.

**CN:** StreamK 覆盖被压缩为一个 `NTT` 的 TensorOp 用例。和 FP32 文件一样，这里更像是一个定向的冒烟测试，用来确认双精度下的 StreamK 路径能够生成并运行。

### Lines 102-103

```python
if __name__ == '__main__':
    unittest.main()
```
**EN:** The file ends with the standard `unittest.main()` entry point.

**CN:** 文件最后以标准的 `unittest.main()` 入口结束。

## Key Concepts / 关键概念

- **EN:** FP64 coverage is intentionally smaller and uses smaller tiles because double-precision kernels are more resource-intensive.
  **CN:** 由于双精度内核更消耗资源，FP64 覆盖规模刻意更小，tile 也更保守。
- **EN:** All alignments remain `[1,1,1]`, unlike FP16/FP32 suites that can exploit wider vector-friendly alignments.
  **CN:** 所有对齐都保持为 `[1,1,1]`，不同于 FP16/FP32 文件那样可以利用更宽的向量化对齐。
- **EN:** TensorOp, SIMT, and StreamK are still all represented, so the file checks both feature breadth and precision-specific behavior.
  **CN:** 尽管规模更小，文件仍然同时包含 TensorOp、SIMT 和 StreamK，因此既覆盖功能广度，也关注精度相关行为。
- **EN:** The file validates same-precision FP64 input, accumulation, C, and output handling end to end.
  **CN:** 该文件端到端验证了 FP64 输入、累加、C 和输出都保持同精度的处理路径。

## Dependencies / 依赖关系

- **EN:** External: `cutlass_cppgen` provides FP64 datatypes, opclass enums, and StreamK support.
  **CN:** 外部依赖：`cutlass_cppgen` 提供 FP64 数据类型、opclass 枚举以及 StreamK 支持。
- **EN:** Hardware gate: `device_cc` enforces SM80-or-newer execution.
  **CN:** 硬件门控：`device_cc` 强制要求 SM80 或更新架构。
- **EN:** Internal helpers: `LayoutCombination` and `add_test_gemm` supply layout presets and generated test methods.
  **CN:** 内部依赖：`LayoutCombination` 与 `add_test_gemm` 提供布局预设和生成式测试方法。
- **EN:** Standard library: `partial`, `logging`, and `unittest` support reusable configuration and test discovery.
  **CN:** 标准库依赖：`partial`、`logging` 与 `unittest` 用于配置复用和测试发现。
