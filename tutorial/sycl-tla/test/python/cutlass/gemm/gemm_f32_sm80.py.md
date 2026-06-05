# gemm_f32_sm80.py — Code Analysis / 代码分析

## Source / 来源

- Path / 路径: `test/python/cutlass/gemm/gemm_f32_sm80.py`
- **EN:** Low-level SM80 FP32 GEMM registration that covers TensorOp, SIMT, and a small StreamK smoke test with dynamically generated unit tests.
- **CN:** 该文件通过动态生成单元测试的方式，覆盖 SM80 上的 FP32 GEMM，包括 TensorOp、SIMT，以及一小组 StreamK 冒烟测试。

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
dtype = cutlass_cppgen.DataType.f32
```
**EN:** The module imports the shared registration helpers, selects SM80 as the target architecture, and uses `cutlass_cppgen.DataType.f32` as the canonical type for inputs, accumulation, and output unless overridden later.

**CN:** 模块导入共享的测试注册辅助工具，把目标架构设为 SM80，并把 `cutlass_cppgen.DataType.f32` 作为默认类型；除非后面显式覆盖，否则输入、累加和输出都使用这一类型。

### Lines 52-67

```python
@unittest.skipIf(device_cc() < cc, 'Device compute capability is insufficient for SM80 tests.')
@unittest.skipIf(cutlass_cppgen.utils.datatypes.torch_type(dtype) is None, f'Version of torch installed does not contain a datatype match for {dtype}')
class GemmF32Sm80(unittest.TestCase):
    """
    Wrapper class to which tests will be added dynamically in __main__
    """
    pass


@unittest.skipIf(device_cc() < cc, 'Device compute capability is insufficient for SM80 tests.')
@unittest.skipIf(cutlass_cppgen.utils.datatypes.torch_type(dtype) is None, f'Version of torch installed does not contain a datatype match for {dtype}')
class GemmF32Sm80StreamK(unittest.TestCase):
    """
    Wrapper class to which tests will be added dynamically in __main__
    """
    pass
```
**EN:** As in the FP16 and FP64 SM80 files, two wrapper classes are declared: one for the main suite and one for StreamK. Both are protected by the same capability and torch-dtype checks.

**CN:** 与 SM80 的 FP16 和 FP64 文件相同，这里也声明了两个包装类：一个承载主测试集，一个承载 StreamK 测试。二者都受相同的硬件能力与 torch 数据类型检查保护。

### Lines 70-73

```python
add_test_specialized = partial(add_test_gemm, element=dtype, cc=cc, cluster_shape=[1, 1, 1])

# Tests using TensorOp
add_test_tensorop = partial(add_test_specialized, opclass=cutlass_cppgen.OpcodeClass.TensorOp)
```
**EN:** `add_test_specialized` captures the SM80/F32 baseline, and `add_test_tensorop` specializes it to Tensor Core execution. This keeps later registrations short while still exposing the full parameter set in each concrete case.

**CN:** `add_test_specialized` 固定了 SM80/F32 的公共基线，而 `add_test_tensorop` 再把它收窄为 Tensor Core 执行。这样后面的具体注册既保持简洁，又保留了完整参数信息。

### Lines 75-82

```python
add_test_tensorop(cls=GemmF32Sm80, layouts=LayoutCombination.NNN, alignments=[4, 4, 4], element_output=dtype, element_C=dtype,
                  element_accumulator=dtype, threadblock_shape=[128, 128, 32], warp_count=[2, 2, 1], stages=3)
add_test_tensorop(cls=GemmF32Sm80, layouts=LayoutCombination.NNT, alignments=[4, 4, 4], element_output=dtype, element_C=dtype,
                  element_accumulator=dtype, threadblock_shape=[128, 128, 32], warp_count=[2, 2, 1], stages=3)
add_test_tensorop(cls=GemmF32Sm80, layouts=LayoutCombination.NTN, alignments=[4, 4, 4], element_output=dtype, element_C=dtype,
                  element_accumulator=dtype, threadblock_shape=[ 64, 128, 32], warp_count=[1, 2, 1], stages=3)
add_test_tensorop(cls=GemmF32Sm80, layouts=LayoutCombination.NTN, alignments=[4, 4, 4], element_output=dtype, element_C=dtype,
                  element_accumulator=dtype, threadblock_shape=[ 64,  64, 32], warp_count=[1, 1, 1], stages=4)
```
**EN:** The TensorOp section registers four cases. `NNN` and `NNT` use a `128x128x32` tile, while `NTN` is tested with both `64x128x32` and `64x64x32` tiles. All of them use alignment `[4,4,4]`, FP32 accumulation, and explicit warp decompositions.

**CN:** TensorOp 部分共注册四个用例。`NNN` 和 `NNT` 使用 `128x128x32` tile，而 `NTN` 还额外测试 `64x128x32` 与 `64x64x32` 两种 tile。所有这些用例都采用 `[4,4,4]` 对齐、FP32 累加以及显式给定的 warp 拆分。

### Lines 83-95

```python
# Tests using SIMT
add_test_simt = partial(add_test_specialized, opclass=cutlass_cppgen.OpcodeClass.Simt)

add_test_simt(cls=GemmF32Sm80, layouts=LayoutCombination.NNN, alignments=[1, 1, 1], element_output=dtype, element_C=dtype,
              element_accumulator=dtype, threadblock_shape=[128, 128, 8], warp_count=[2, 2, 1], stages=2)
add_test_simt(cls=GemmF32Sm80, layouts=LayoutCombination.TNN, alignments=[1, 1, 1], element_output=dtype, element_C=dtype,
              element_accumulator=dtype, threadblock_shape=[ 64, 128, 8], warp_count=[1, 2, 1], stages=2)
add_test_simt(cls=GemmF32Sm80, layouts=LayoutCombination.NTN, alignments=[1, 1, 1], element_output=dtype, element_C=dtype,
              element_accumulator=dtype, threadblock_shape=[128,  64, 8], warp_count=[2, 1, 1], stages=2)
add_test_simt(cls=GemmF32Sm80, layouts=LayoutCombination.TTN, alignments=[1, 1, 1], element_output=dtype, element_C=dtype,
              element_accumulator=dtype, threadblock_shape=[ 64,  64, 8], warp_count=[1, 1, 1], stages=2)
add_test_simt(cls=GemmF32Sm80, layouts=LayoutCombination.NNT, alignments=[1, 1, 1], element_output=dtype, element_C=dtype,
              element_accumulator=dtype, threadblock_shape=[128, 128, 8], warp_count=[2, 2, 1], stages=2)
```
**EN:** The SIMT block adds five fallback cases with alignment `[1,1,1]` and smaller K-dimension tiles (`8`). Together they cover `NNN`, `TNN`, `NTN`, `TTN`, and `NNT`, which gives the non-TensorOp path a representative layout spread.

**CN:** SIMT 块增加了五个回退用例，对齐统一为 `[1,1,1]`，K 维 tile 缩小为 `8`。这些用例共同覆盖 `NNN`、`TNN`、`NTN`、`TTN` 和 `NNT`，为非 TensorOp 路径提供了有代表性的布局分布。

### Lines 97-100

```python
# Stream K tests
add_test_streamk = partial(add_test_specialized, opclass=cutlass_cppgen.OpcodeClass.TensorOp, swizzle=cutlass_cppgen.swizzle.ThreadblockSwizzleStreamK)
add_test_streamk(cls=GemmF32Sm80StreamK, layouts=LayoutCombination.TTN, alignments=[4, 4, 4], element_output=dtype, element_C=dtype,
                 element_accumulator=dtype, threadblock_shape=[128, 128, 32], warp_count=[2, 2, 1], stages=3)
```
**EN:** The StreamK coverage is intentionally lightweight: it registers a single `TTN` TensorOp case using the StreamK swizzle. That is enough to ensure the specialized scheduling path is at least exercised for FP32 on SM80.

**CN:** StreamK 覆盖刻意保持轻量：这里只注册了一个采用 StreamK swizzle 的 `TTN` TensorOp 用例。这已经足以保证 FP32 在 SM80 上的这条特殊调度路径至少被执行一次。

### Lines 103-104

```python
if __name__ == '__main__':
    unittest.main()
```
**EN:** The final lines provide the standard `unittest` launcher.

**CN:** 最后两行提供标准的 `unittest` 启动入口。

## Key Concepts / 关键概念

- **EN:** The file uses the same dynamic registration pattern as the other SM80 suites, but with FP32-specific tile/alignment choices.
  **CN:** 该文件沿用其他 SM80 测试的动态注册模式，但采用了 FP32 专属的 tile 与对齐选择。
- **EN:** TensorOp and SIMT are both covered so the suite checks optimized and fallback execution paths.
  **CN:** 文件同时覆盖 TensorOp 与 SIMT，因此既检查优化路径，也检查回退路径。
- **EN:** The StreamK section is a smoke test rather than a full combinatorial sweep.
  **CN:** StreamK 部分属于冒烟测试，而不是完整的组合式穷举。
- **EN:** All main numeric roles remain FP32, making this a same-precision GEMM validation file.
  **CN:** 主要数值角色全部保持 FP32，因此这是一个同精度 GEMM 验证文件。

## Dependencies / 依赖关系

- **EN:** External: `cutlass_cppgen` supplies FP32 datatypes, TensorOp/SIMT opclass enums, and the StreamK swizzle.
  **CN:** 外部依赖：`cutlass_cppgen` 提供 FP32 数据类型、TensorOp/SIMT 的 opclass 枚举以及 StreamK swizzle。
- **EN:** Hardware gate: `device_cc` requires an SM80-or-newer GPU.
  **CN:** 硬件门控：`device_cc` 要求当前 GPU 至少为 SM80。
- **EN:** Internal helpers: `LayoutCombination` and `add_test_gemm` define layout triples and generate the actual unit-test methods.
  **CN:** 内部依赖：`LayoutCombination` 和 `add_test_gemm` 负责定义布局三元组并生成实际的单元测试方法。
- **EN:** Standard library: `partial`, `logging`, and `unittest` support concise registration and test execution.
  **CN:** 标准库依赖：`partial`、`logging` 与 `unittest` 用于简化注册过程和执行测试。
