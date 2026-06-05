# gemm_f64_sm90.py — Code Analysis / 代码分析

## Source / 来源

- Path / 路径: `test/python/cutlass/gemm/gemm_f64_sm90.py`
- **EN:** A compact SM90 FP64 GEMM smoke suite that reuses one common partial and registers just a few TensorOp and SIMT cases.
- **CN:** 这是一个紧凑的 SM90 FP64 GEMM 冒烟测试文件：它复用一个公共 `partial`，只注册少量 TensorOp 和 SIMT 用例。

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
cc = 90
dtype = cutlass_cppgen.DataType.f64
```
**EN:** The top of the file follows the same pattern as other GEMM suites: shared imports, warning-only logging, `cc = 90`, and `dtype = f64`. The file is intentionally short because it aims for basic Hopper FP64 coverage rather than an exhaustive matrix.

**CN:** 文件开头延续了其他 GEMM 测试的一贯写法：共享导入、warning 级别日志、`cc = 90` 以及 `dtype = f64`。整个文件很短，说明它的目标是提供 Hopper 上 FP64 的基础覆盖，而不是穷举所有组合。

### Lines 52-58

```python
@unittest.skipIf(device_cc() < cc, 'Device compute capability is insufficient for SM90 tests.')
@unittest.skipIf(cutlass_cppgen.utils.datatypes.torch_type(dtype) is None, f'Version of torch installed does not contain a datatype match for {dtype}')
class GemmF64Sm90(unittest.TestCase):
    """
    Wrapper class to which tests will be added dynamically in __main__
    """
    pass
```
**EN:** There is only one wrapper class, and it is guarded by the expected SM90 capability and torch-dtype checks. As elsewhere, the class body is empty because generated methods will be attached later.

**CN:** 这里同样只有一个包装类，并通过 SM90 能力检查与 torch 数据类型检查进行保护。和其他文件一样，类体本身为空，因为真正的方法会在后面动态挂接。

### Lines 61-62

```python
add_test_specialized = partial(add_test_gemm, cls=GemmF64Sm90, alignments=[1, 1, 1], cluster_shape=[1, 1, 1],
                               element=dtype, element_output=dtype, element_accumulator=dtype, compilation_modes=['nvcc'])
```
**EN:** This base partial is more aggressive about fixing defaults than most other files: it locks the class, alignments, cluster shape, element type, output type, accumulator type, and compilation mode (`nvcc`) all at once. The concrete registrations only need to specify opclass, layout, tile shape, and stage count.

**CN:** 这个基础 `partial` 比多数同类文件更“激进”地一次性固定了默认参数：它同时锁定测试类、对齐、cluster shape、元素类型、输出类型、累加类型以及编译模式（`nvcc`）。因此后续具体注册只需要给出 opclass、布局、tile 形状和 stage 数。

### Lines 64-67

```python
add_test_specialized(opclass=cutlass_cppgen.OpcodeClass.TensorOp, layouts=LayoutCombination.NNT, threadblock_shape=[128, 128, 32], stages=3)
add_test_specialized(opclass=cutlass_cppgen.OpcodeClass.TensorOp, layouts=LayoutCombination.TNN, threadblock_shape=[128, 128, 32], stages=3)
add_test_specialized(    opclass=cutlass_cppgen.OpcodeClass.Simt, layouts=LayoutCombination.NNN, threadblock_shape=[128, 128,  8], stages=2)
add_test_specialized(    opclass=cutlass_cppgen.OpcodeClass.Simt, layouts=LayoutCombination.TTT, threadblock_shape=[ 64, 128,  8], stages=2)
```
**EN:** The actual suite contains four cases: two TensorOp registrations (`NNT` and `TNN`) using `128x128x32` tiles, and two SIMT registrations (`NNN` and `TTT`) using `128x128x8` and `64x128x8` tiles. This makes the file a minimal cross-section of Hopper FP64 behavior instead of a large combinational sweep.

**CN:** 真正的测试集合只有四个用例：两个 TensorOp 注册（`NNT` 与 `TNN`），都使用 `128x128x32` tile；再加两个 SIMT 注册（`NNN` 与 `TTT`），分别使用 `128x128x8` 与 `64x128x8` tile。也就是说，这个文件提供的是 Hopper FP64 行为的最小横截面，而不是大规模组合式扫描。

### Lines 70-71

```python
if __name__ == '__main__':
    unittest.main()
```
**EN:** The closing `unittest.main()` call enables direct execution.

**CN:** 结尾的 `unittest.main()` 使该文件可以直接执行。

## Key Concepts / 关键概念

- **EN:** This is the smallest suite in the requested set; it is designed as a smoke test, not a broad parameter study.
  **CN:** 在本次要求的文件集合中，这是最精简的一组；它的定位是冒烟测试，而不是广泛的参数研究。
- **EN:** A single common partial eliminates repetition by fixing almost every shared FP64/SM90 parameter up front.
  **CN:** 通过一个公共 `partial` 预先固定几乎所有共享的 FP64/SM90 参数，文件最大程度减少了重复代码。
- **EN:** Both TensorOp and SIMT are still represented, so the suite touches optimized and fallback execution paths.
  **CN:** 文件仍然同时包含 TensorOp 与 SIMT，因此既触达优化路径，也触达回退路径。
- **EN:** All numeric roles stay in FP64, making this a pure same-precision validation path.
  **CN:** 所有数值角色都保持 FP64，因此这是一个纯粹的同精度验证路径。

## Dependencies / 依赖关系

- **EN:** External: `cutlass_cppgen` provides FP64 datatypes and both TensorOp/SIMT opclass enums.
  **CN:** 外部依赖：`cutlass_cppgen` 提供 FP64 数据类型以及 TensorOp/SIMT 的 opclass 枚举。
- **EN:** Hardware gate: `device_cc` ensures only SM90-capable devices run the suite.
  **CN:** 硬件门控：`device_cc` 确保只有支持 SM90 的设备才会运行这组测试。
- **EN:** Internal helpers: `LayoutCombination` and `add_test_gemm` remain responsible for layout encoding and generated test creation.
  **CN:** 内部依赖：`LayoutCombination` 与 `add_test_gemm` 仍然负责布局编码和生成式测试创建。
- **EN:** Build dependency: the file hardcodes `compilation_modes=["nvcc"]`, so it assumes the CUDA nvcc path is available.
  **CN:** 构建依赖：该文件硬编码了 `compilation_modes=["nvcc"]`，因此默认依赖 CUDA 的 nvcc 编译路径可用。
