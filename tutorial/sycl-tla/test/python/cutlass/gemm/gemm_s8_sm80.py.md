# gemm_s8_sm80.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/python/cutlass/gemm/gemm_s8_sm80.py`
- **Purpose (EN):** Registers SM80 INT8 GEMM tests across TensorOp, SIMT, and StreamK variants, using dynamic unittest method generation.
- **用途 (CN):** 通过动态生成 unittest 方法，为 SM80 INT8 GEMM 注册 TensorOp、SIMT 与 StreamK 多种测试变体。

## Line-by-Line Analysis / 逐行分析

### Lines 33-45 / 第 33-45 行

```python
"""
Low-level functionality tests for GEMM with S8 operands on SM80
"""

from functools import partial
import logging
import unittest

import cutlass_cppgen
from cutlass_cppgen.backend.utils.device import device_cc

from utils import LayoutCombination, add_test_gemm

```

**EN:** The file follows the same registration-only pattern as the other architecture-specific suites.
It imports CUTLASS bindings, the device capability helper, and the local layout/test-registration utilities needed to build lots of cases declaratively.

**CN:** 该文件和其他按架构划分的测试一样，属于“只注册、不直接实现测试体”的模式。
它导入 CUTLASS 绑定、设备能力查询函数，以及用于声明式批量生成测试的本地布局/注册辅助工具。

### Lines 47-70 / 第 47-70 行

```python
cutlass_cppgen.set_log_level(logging.WARNING)
cc = 80
dtype = cutlass_cppgen.DataType.s8


@unittest.skipIf(device_cc() < cc, 'Device compute capability is insufficient for SM80 tests.')
@unittest.skipIf(cutlass_cppgen.utils.datatypes.torch_type(dtype) is None, f'Version of torch installed does not contain a datatype match for {dtype}')
class GemmS8Sm80(unittest.TestCase):
    """
    Wrapper class to which tests will be added dynamically in __main__
    """
    pass


@unittest.skipIf(device_cc() < cc, 'Device compute capability is insufficient for SM80 tests.')
@unittest.skipIf(cutlass_cppgen.utils.datatypes.torch_type(dtype) is None, f'Version of torch installed does not contain a datatype match for {dtype}')
class GemmS8Sm80StreamK(unittest.TestCase):
    """
    Wrapper class to which tests will be added dynamically in __main__
    """
    pass


add_test_specialized = partial(add_test_gemm, element=dtype, cc=cc, cluster_shape=[1, 1, 1])
```

**EN:** Logging is reduced, `cc` is set to 80, and the base datatype is `s8`.
The file defines two unittest containers: `GemmS8Sm80` for the regular suite and `GemmS8Sm80StreamK` for StreamK-specific cases.
`add_test_specialized` binds shared parameters such as `element=dtype`, `cc=80`, and the default cluster shape `[1,1,1]`, but intentionally leaves `cls` open so later calls can target either unittest class.

**CN:** 文件先收敛日志，把 `cc` 设为 80，并把基础数据类型定为 `s8`。
接着定义两个 unittest 容器：`GemmS8Sm80` 用于常规测试集合，`GemmS8Sm80StreamK` 专门容纳 StreamK 相关用例。
`add_test_specialized` 预绑定了共享参数，例如 `element=dtype`、`cc=80` 和默认 cluster `[1,1,1]`，但刻意不绑定 `cls`，以便后续调用可分别挂载到两个测试类上。

### Lines 72-81 / 第 72-81 行

```python
# Tests using TensorOp
add_test_tensorop = partial(add_test_specialized, opclass=cutlass_cppgen.OpcodeClass.TensorOp)

add_test_tensorop(cls=GemmS8Sm80, layouts=LayoutCombination.TNN, alignments=[16, 16, 16],  element_output=cutlass_cppgen.DataType.s8, element_C=cutlass_cppgen.DataType.s8,
                  element_accumulator=cutlass_cppgen.DataType.s32, threadblock_shape=[256, 128, 64], warp_count=[4, 2, 1], stages=3)
add_test_tensorop(cls=GemmS8Sm80, layouts=LayoutCombination.TNT, alignments=[16, 16, 16],  element_output=cutlass_cppgen.DataType.s8, element_C=cutlass_cppgen.DataType.s8,
                  element_accumulator=cutlass_cppgen.DataType.s32, threadblock_shape=[128, 256, 64], warp_count=[2, 4, 1], stages=3)
add_test_tensorop(cls=GemmS8Sm80, layouts=LayoutCombination.TNN, alignments=[16, 16,  4], element_output=cutlass_cppgen.DataType.s32, element_C=cutlass_cppgen.DataType.s32,
                  element_accumulator=cutlass_cppgen.DataType.s32, threadblock_shape=[ 64,  64, 64], warp_count=[1, 1, 1], stages=4)

```

**EN:** The TensorOp section adds three INT8 kernel variants.
Two cases output INT8 into INT8 C tensors, but use different `TNN`/`TNT` layout combinations and different warp-count aspect ratios to match their threadblock shapes.
The third TensorOp case switches output and C to `s32` and uses a smaller `64x64x64` tile with four stages, covering an accumulator-style output path.

**CN:** TensorOp 段注册了三个 INT8 内核变体。
其中前两个用例把结果写回 INT8 的 C 张量，但分别采用 `TNN`/`TNT` 布局组合，以及与 threadblock 形状相匹配的不同 warp 宽高比。
第三个 TensorOp 用例把输出与 C 都切换为 `s32`，并改用较小的 `64x64x64` tile 和 4 个流水阶段，从而覆盖“累加结果保留为 32 位”的路径。

### Lines 82-103 / 第 82-103 行

```python
# Tests using SIMT
add_test_simt = partial(add_test_specialized, opclass=cutlass_cppgen.OpcodeClass.Simt)

add_test_simt(cls=GemmS8Sm80, layouts=LayoutCombination.NNN, alignments=[1, 1, 1],  element_output=cutlass_cppgen.DataType.s8, element_C=cutlass_cppgen.DataType.s8,
              element_accumulator=cutlass_cppgen.DataType.s32, threadblock_shape=[128, 128, 8], warp_count=[2, 2, 1], stages=2)
add_test_simt(cls=GemmS8Sm80, layouts=LayoutCombination.TNN, alignments=[1, 1, 1],  element_output=cutlass_cppgen.DataType.s8, element_C=cutlass_cppgen.DataType.s8,
              element_accumulator=cutlass_cppgen.DataType.s32, threadblock_shape=[ 64, 128, 8], warp_count=[1, 2, 1], stages=2)
add_test_simt(cls=GemmS8Sm80, layouts=LayoutCombination.NTN, alignments=[1, 1, 1],  element_output=cutlass_cppgen.DataType.s8, element_C=cutlass_cppgen.DataType.s8,
              element_accumulator=cutlass_cppgen.DataType.s32, threadblock_shape=[128,  64, 8], warp_count=[2, 1, 1], stages=2)
add_test_simt(cls=GemmS8Sm80, layouts=LayoutCombination.TTN, alignments=[1, 1, 1], element_output=cutlass_cppgen.DataType.s32, element_C=cutlass_cppgen.DataType.s32,
              element_accumulator=cutlass_cppgen.DataType.s32, threadblock_shape=[ 64,  64, 8], warp_count=[1, 1, 1], stages=2)
add_test_simt(cls=GemmS8Sm80, layouts=LayoutCombination.NNT, alignments=[1, 1, 1], element_output=cutlass_cppgen.DataType.s32, element_C=cutlass_cppgen.DataType.s32,
              element_accumulator=cutlass_cppgen.DataType.s32, threadblock_shape=[128, 128, 8], warp_count=[2, 2, 1], stages=2)

# Stream K tests
add_test_streamk = partial(add_test_specialized, opclass=cutlass_cppgen.OpcodeClass.TensorOp, swizzle=cutlass_cppgen.swizzle.ThreadblockSwizzleStreamK)
add_test_streamk(cls=GemmS8Sm80StreamK, layouts=LayoutCombination.TNT, alignments=[16, 16, 16], element_output=cutlass_cppgen.DataType.s8, element_C=cutlass_cppgen.DataType.s8,
                 element_accumulator=cutlass_cppgen.DataType.s32, threadblock_shape=[128, 256, 64], warp_count=[2, 4, 1], stages=3)


if __name__ == '__main__':
    unittest.main()
```

**EN:** The SIMT section broadens coverage to five layout/output combinations with scalar alignment `1` and small `k=8` tiles.
Some SIMT cases keep INT8 outputs, while others use `s32` outputs/C tensors, which checks epilogue behavior under different accumulation-storage choices.
The final `add_test_streamk(...)` call targets the second unittest class and binds `ThreadblockSwizzleStreamK`, isolating a StreamK TensorOp case from the regular suite.
The module ends with `unittest.main()`.

**CN:** SIMT 段把覆盖范围扩展到五种布局/输出组合，统一使用标量对齐 `1` 与较小的 `k=8` tile。
这些 SIMT 用例中，有的仍写回 INT8，有的则把输出和 C 张量设为 `s32`，从而检查不同累加/存储组合下的 epilogue 行为。
最后一条 `add_test_streamk(...)` 调用把 `ThreadblockSwizzleStreamK` 绑定到第二个 unittest 类上，从常规测试集中独立出一个 StreamK TensorOp 用例。
模块结尾使用 `unittest.main()`。

## Key Concepts / 关键概念

- **Two test containers / 两个测试容器:** Regular kernels and StreamK kernels are split into separate unittest classes for clearer reporting. / 常规内核与 StreamK 内核被拆到不同 unittest 类中，报告更清晰。
- **INT8 output variants / INT8 输出变体:** The suite checks both INT8-output and INT32-output forms of INT8 GEMM. / 该套件同时检查 INT8 GEMM 的 INT8 输出形式与 INT32 输出形式。
- **Execution-path breadth / 执行路径广度:** TensorOp, SIMT, and StreamK each get explicit coverage on SM80. / SM80 上的 TensorOp、SIMT、StreamK 都有明确覆盖。

## Dependencies / 依赖关系

- **`cutlass_cppgen` / `cutlass_cppgen`:** Provides datatype enums, opcode classes, and the StreamK swizzle functor. / 提供数据类型枚举、Opcode 类以及 StreamK swizzle functor。
- **`device_cc` / `device_cc`:** Used by `skipIf` decorators to gate execution by GPU capability. / 被 `skipIf` 装饰器用于按 GPU 能力门控执行。
- **Local `utils` / 本地 `utils`:** Supplies layout presets and the dynamic GEMM-test factory. / 提供布局预设和动态 GEMM 测试工厂函数。
- **`unittest` / `unittest`:** Hosts the generated methods and supplies discovery via `unittest.main()`. / 承载动态生成的方法，并通过 `unittest.main()` 提供发现与执行。
