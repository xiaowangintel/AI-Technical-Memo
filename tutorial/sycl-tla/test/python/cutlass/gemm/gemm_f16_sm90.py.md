# gemm_f16_sm90.py — Code Analysis / 代码分析

## Source / 来源

- Path / 路径: `test/python/cutlass/gemm/gemm_f16_sm90.py`
- **EN:** Low-level SM90 FP16 GEMM coverage for Hopper-class features such as cluster shapes, schedule policies, SIMT fallback, and void-C kernels.
- **CN:** 该文件为 SM90 上的 FP16 GEMM 提供低层覆盖，重点体现 Hopper 相关特性，例如 cluster shape、调度策略、SIMT 回退路径以及 void-C 内核。

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
dtype = cutlass_cppgen.DataType.f16
```
**EN:** The imports and top-level constants match the rest of the suite, but the target architecture is now SM90 and the operand type remains FP16. This file is aimed at Hopper-era configuration features that do not exist in the SM80 file.

**CN:** 导入方式和顶层常量与同类文件一致，但这里的目标架构变成了 SM90，操作数类型仍是 FP16。相较于 SM80 文件，这里更关注 Hopper 时代才出现的一些配置特性。

### Lines 51-57

```python
@unittest.skipIf(device_cc() < cc, 'Device compute capability is insufficient for SM90 tests.')
@unittest.skipIf(cutlass_cppgen.utils.datatypes.torch_type(dtype) is None, f'Version of torch installed does not contain a datatype match for {dtype}')
class GemmF16Sm90(unittest.TestCase):
    """
    Wrapper class to which tests will be added dynamically in __main__
    """
    pass
```
**EN:** Unlike the SM80 file, SM90 only needs one wrapper class because the specialized Hopper coverage is kept in a single suite. The decorators again require both sufficient compute capability and a matching PyTorch FP16 type.

**CN:** 与 SM80 文件不同，SM90 这里只需要一个包装类，因为 Hopper 专项覆盖都集中在同一个测试集合中。装饰器依旧要求足够高的计算能力以及 PyTorch 提供匹配的 FP16 类型。

### Lines 60-63

```python
add_test_specialized = partial(add_test_gemm, cls=GemmF16Sm90, element=dtype,
                               warp_count=None, compilation_modes=['nvcc'])

add_test_tensorop = partial(add_test_specialized, opclass=cutlass_cppgen.OpcodeClass.TensorOp)
```
**EN:** `add_test_specialized` fixes the generated class, the element type, and `compilation_modes=["nvcc"]`, while leaving `warp_count=None` so the helper or backend can infer the preferred warp decomposition. `add_test_tensorop` then narrows the path to Tensor Core kernels.

**CN:** `add_test_specialized` 固定了目标测试类、元素类型以及 `compilation_modes=["nvcc"]`，同时把 `warp_count` 留为 `None`，让辅助函数或后端去推断更合适的 warp 拆分。`add_test_tensorop` 则进一步把路径限定为 Tensor Core 内核。

### Lines 65-86

```python
# Tests with 1x1x1 clusters
add_test_unit_cluster = partial(add_test_tensorop, cluster_shape=[1, 1, 1])
add_test_unit_cluster(layouts=LayoutCombination.NNN, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16,
                      element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 32], stages=3)
add_test_unit_cluster(layouts=LayoutCombination.NNT, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16,
                      element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 32], stages=None)
add_test_unit_cluster(layouts=LayoutCombination.NTN, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16,
                      element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 32], stages=None)
add_test_unit_cluster(layouts=LayoutCombination.NTT, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16,
                      element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 32], stages=None)
add_test_unit_cluster(layouts=LayoutCombination.TNN, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16,
                      element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 32], stages=None)
add_test_unit_cluster(layouts=LayoutCombination.TNT, alignments=[4, 4, 8], element_output=cutlass_cppgen.DataType.f16,
                      element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 32], stages=None)
add_test_unit_cluster(layouts=LayoutCombination.TNT, alignments=[4, 4, 8], element_output=cutlass_cppgen.DataType.f16,
                      element_accumulator=cutlass_cppgen.DataType.f16, threadblock_shape=[128, 128, 32], stages=None)
add_test_unit_cluster(layouts=LayoutCombination.TNT, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16,
                      element_accumulator=cutlass_cppgen.DataType.f16, threadblock_shape=[128, 128, 32], stages=None)
add_test_unit_cluster(layouts=LayoutCombination.TNT, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16,
                      element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[ 64,  64, 64], stages=5)
add_test_unit_cluster(layouts=LayoutCombination.TNT, alignments=[2, 2, 2], element_output=cutlass_cppgen.DataType.f16,
                      element_accumulator=cutlass_cppgen.DataType.f16, threadblock_shape=[128, 128, 32], stages=None)
```
**EN:** The first TensorOp group keeps `cluster_shape=[1,1,1]` and explores layout, alignment, accumulator precision, and stage selection. Most registrations use a `128x128x32` tile; one `NNN` case pins `stages=3`, many others leave `stages=None` to accept automatic stage selection, and one `64x64x64` `TNT` case explicitly raises the pipeline depth to `5`.

**CN:** 第一组 TensorOp 测试固定 `cluster_shape=[1,1,1]`，重点探索布局、对齐、累加精度以及 stage 选择。大多数注册使用 `128x128x32` tile；其中一个 `NNN` 用例固定 `stages=3`，很多其他用例把 `stages=None` 留给后端自动选择，还有一个 `64x64x64` 的 `TNT` 用例显式把流水线深度提高到 `5`。

### Lines 88-105

```python
# Tests with different cluster shapes
add_test_cluster_shape = partial(add_test_tensorop, threadblock_shape=[64, 128, 64], stages=None)
add_test_cluster_shape(layouts=LayoutCombination.TTN, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16,
                       element_accumulator=cutlass_cppgen.DataType.f16, cluster_shape=[2, 2, 1])
add_test_cluster_shape(layouts=LayoutCombination.TNN, alignments=[8, 8, 4], element_output=cutlass_cppgen.DataType.f32,
                       element_accumulator=cutlass_cppgen.DataType.f32, cluster_shape=[2, 2, 1])
add_test_cluster_shape(layouts=LayoutCombination.NTN, alignments=[8, 8, 4], element_output=cutlass_cppgen.DataType.f32,
                       element_accumulator=cutlass_cppgen.DataType.f32, cluster_shape=[2, 2, 1])
add_test_cluster_shape(layouts=LayoutCombination.NNN, alignments=[8, 8, 4], element_output=cutlass_cppgen.DataType.f32,
                       element_accumulator=cutlass_cppgen.DataType.f32, cluster_shape=[2, 2, 1])
add_test_cluster_shape(layouts=LayoutCombination.TTN, alignments=[8, 8, 4], element_output=cutlass_cppgen.DataType.f32,
                       element_accumulator=cutlass_cppgen.DataType.f32, cluster_shape=[1, 4, 1])
add_test_cluster_shape(layouts=LayoutCombination.TTN, alignments=[8, 8, 4], element_output=cutlass_cppgen.DataType.f32,
                       element_accumulator=cutlass_cppgen.DataType.f32, cluster_shape=[2, 4, 1])
add_test_cluster_shape(layouts=LayoutCombination.TTN, alignments=[8, 8, 4], element_output=cutlass_cppgen.DataType.f32,
                       element_accumulator=cutlass_cppgen.DataType.f32, cluster_shape=[4, 1, 1])
add_test_cluster_shape(layouts=LayoutCombination.TTN, alignments=[8, 8, 4], element_output=cutlass_cppgen.DataType.f32,
                       element_accumulator=cutlass_cppgen.DataType.f32, cluster_shape=[4, 2, 1])
```
**EN:** The next block focuses on cluster shapes. It fixes a `64x128x64` threadblock and varies `cluster_shape` across `[2,2,1]`, `[1,4,1]`, `[2,4,1]`, `[4,1,1]`, and `[4,2,1]`. The first case keeps FP16 output/accumulation, while the later ones switch to FP32 output/accumulation for additional epilogue coverage.

**CN:** 下一块聚焦 cluster shape。本段固定 `64x128x64` 的 threadblock，然后把 `cluster_shape` 扫描为 `[2,2,1]`、`[1,4,1]`、`[2,4,1]`、`[4,1,1]` 和 `[4,2,1]`。第一个用例保持 FP16 输出/累加，后面若干用例则切换到 FP32 输出/累加，以补充 epilogue 相关覆盖。

### Lines 107-130

```python
# Tests for different schedule modes
add_test_schedule = partial(add_test_specialized, layouts=LayoutCombination.TTN, alignments=[8, 8, 4],
                            element_output=cutlass_cppgen.DataType.f32, element_accumulator=cutlass_cppgen.DataType.f32,
                            opclass=cutlass_cppgen.OpcodeClass.TensorOp, threadblock_shape=[128, 128, 64], stages=None)
add_test_schedule(
    cluster_shape=[1, 1, 1],
    kernel_schedule=cutlass_cppgen.KernelScheduleType.TmaWarpSpecializedPingpong,
    epilogue_schedule=cutlass_cppgen.EpilogueScheduleType.TmaWarpSpecialized
)
add_test_schedule(
    cluster_shape=[1, 1, 1],
    kernel_schedule=cutlass_cppgen.KernelScheduleType.TmaWarpSpecializedCooperative,
    epilogue_schedule=cutlass_cppgen.EpilogueScheduleType.TmaWarpSpecializedCooperative
)
add_test_schedule(
    cluster_shape=[2, 1, 1],
    kernel_schedule=cutlass_cppgen.KernelScheduleType.TmaWarpSpecializedPingpong,
    epilogue_schedule=cutlass_cppgen.EpilogueScheduleType.TmaWarpSpecialized
)
add_test_schedule(
    cluster_shape=[2, 1, 1],
    kernel_schedule=cutlass_cppgen.KernelScheduleType.TmaWarpSpecializedCooperative,
    epilogue_schedule=cutlass_cppgen.EpilogueScheduleType.TmaWarpSpecializedCooperative
)
```
**EN:** This schedule-oriented group fixes the layout to `TTN`, alignments to `[8,8,4]`, and the tile to `128x128x64`, then compares two Hopper schedule families: ping-pong and cooperative. Each schedule pair is exercised with both `cluster_shape=[1,1,1]` and `[2,1,1]`, so the file explicitly tests scheduling policy as a configuration dimension.

**CN:** 这一组以调度策略为核心：它固定 `TTN` 布局、`[8,8,4]` 对齐以及 `128x128x64` tile，然后比较 Hopper 上两类调度：ping-pong 与 cooperative。每种调度又分别测试 `cluster_shape=[1,1,1]` 和 `[2,1,1]`，因此该文件把调度策略本身当作一个独立的配置维度来验证。

### Lines 132-143

```python
# Tests using SIMT
add_test_simt = partial(add_test_specialized, opclass=cutlass_cppgen.OpcodeClass.Simt, alignments=[1, 1, 1], cluster_shape=[1, 1, 1], stages=2)
add_test_simt(layouts=LayoutCombination.NNN, element_output=cutlass_cppgen.DataType.f16, element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 8])
add_test_simt(layouts=LayoutCombination.TNN, element_output=cutlass_cppgen.DataType.f16, element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[ 64, 128, 8])
add_test_simt(layouts=LayoutCombination.NTN, element_output=cutlass_cppgen.DataType.f16, element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128,  64, 8])
add_test_simt(layouts=LayoutCombination.TTN, element_output=cutlass_cppgen.DataType.f16, element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[ 64,  64, 8])
add_test_simt(layouts=LayoutCombination.NNT, element_output=cutlass_cppgen.DataType.f16, element_accumulator=cutlass_cppgen.DataType.f16, threadblock_shape=[128, 128, 8])

# Tests with void-C kernels
add_test_cluster_shape(layouts=LayoutCombination.NNT, alignments=[8, 8, 8], element_output=cutlass_cppgen.DataType.f16,
                       element_accumulator=cutlass_cppgen.DataType.f32, threadblock_shape=[128, 128, 32], stages=None,
                       cluster_shape=[2, 1, 1], element_C=cutlass_cppgen.DataType.void)
```
**EN:** The final registrations add a SIMT smoke suite and one special TensorOp case with `element_C=cutlass_cppgen.DataType.void`. That `void-C` case means the generated kernel does not read a real `C` tensor, effectively testing the “no source C” epilogue path.

**CN:** 最后一组注册先加入一套 SIMT 冒烟测试，然后增加一个特殊的 TensorOp 用例：`element_C=cutlass_cppgen.DataType.void`。这个 `void-C` 设定表示生成的内核不会真正读取 `C` 张量，本质上是在测试“没有源 `C` 输入”的 epilogue 路径。

### Lines 145-146

```python
if __name__ == '__main__':
    unittest.main()
```
**EN:** The file ends with the usual `unittest.main()` launcher.

**CN:** 文件最后仍然是标准的 `unittest.main()` 启动入口。

## Key Concepts / 关键概念

- **EN:** SM90 coverage expands beyond plain layout/tile sweeps into Hopper-specific cluster and schedule controls.
  **CN:** SM90 的覆盖范围已经从普通布局/tile 扫描扩展到 Hopper 专有的 cluster 与 schedule 控制。
- **EN:** `stages=None` is used intentionally in many registrations so the backend can pick an appropriate pipeline depth.
  **CN:** 很多注册故意把 `stages` 设为 `None`，以便后端自动选择合适的流水线深度。
- **EN:** The file mixes FP16 and FP32 accumulation/output choices to cover more epilogue and numeric pathways.
  **CN:** 该文件交替使用 FP16 与 FP32 的累加/输出设置，以覆盖更多 epilogue 与数值路径。
- **EN:** `element_C=void` is a notable special case because it exercises GEMM variants that do not consume an input C matrix.
  **CN:** `element_C=void` 是一个很重要的特殊情况，因为它覆盖了“不读取输入 C 矩阵”的 GEMM 变体。

## Dependencies / 依赖关系

- **EN:** External: `cutlass_cppgen` contributes `DataType`, `OpcodeClass`, `KernelScheduleType`, and `EpilogueScheduleType` for Hopper-specific configuration.
  **CN:** 外部依赖：`cutlass_cppgen` 提供 `DataType`、`OpcodeClass`、`KernelScheduleType` 与 `EpilogueScheduleType`，用于 Hopper 专项配置。
- **EN:** Hardware gating: `device_cc` ensures only SM90-capable GPUs run this suite.
  **CN:** 硬件门控依赖：`device_cc` 确保只有支持 SM90 的 GPU 才会运行这组测试。
- **EN:** Internal helpers: `LayoutCombination` and `add_test_gemm` remain the core mechanism for declarative registration.
  **CN:** 内部依赖：`LayoutCombination` 与 `add_test_gemm` 仍然是声明式注册测试的核心机制。
- **EN:** Build mode dependency: the suite forces `nvcc`, unlike the Intel Xe BF16 files that use `dpcpp`.
  **CN:** 构建模式依赖：该文件强制使用 `nvcc`，这一点与采用 `dpcpp` 的 Intel Xe BF16 文件不同。
