# gemm_mixed_sm80.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/python/cutlass/gemm/gemm_mixed_sm80.py`
- **Purpose (EN):** Defines SM80 mixed-precision GEMM tests that start from FP16 kernels and selectively upcast one operand from INT8.
- **用途 (CN):** 定义 SM80 上的混合精度 GEMM 测试：以内核基准类型 FP16 为主，并选择性地把一个输入操作数改为 INT8。

## Line-by-Line Analysis / 逐行分析

### Lines 33-45 / 第 33-45 行

```python
"""
Low-level functionality tests for GEMM with mixed operands on SM80
"""

from functools import partial
import logging
import unittest

import cutlass_cppgen
from cutlass_cppgen.backend.utils.device import device_cc

from utils import LayoutCombination, add_test_gemm

```

**EN:** The module docstring accurately states that this file covers “mixed operands on SM80”.
Its imports mirror the structure used by the other GEMM test registration files: `partial`, logging, unittest, CUTLASS bindings, a device capability query, and the local helper utilities.

**CN:** 模块文档字符串明确说明该文件测试的是“SM80 上的混合操作数”。
导入结构与其他 GEMM 测试注册文件一致：`partial`、logging、unittest、CUTLASS 绑定、设备能力查询，以及本地工具函数。

### Lines 47-63 / 第 47-63 行

```python
cutlass_cppgen.set_log_level(logging.WARNING)
cc = 80
dtype =cutlass_cppgen.DataType.f16


@unittest.skipIf(device_cc() < cc, 'Device compute capability is insufficient for SM80 tests.')
@unittest.skipIf(cutlass_cppgen.utils.datatypes.torch_type(dtype) is None, f'Version of torch installed does not contain a datatype match for {dtype}')
class GemmMixedSm80(unittest.TestCase):
    """
    Wrapper class to which tests will be added dynamically in __main__
    """
    pass


add_test_mixed = partial(add_test_gemm, cls=GemmMixedSm80, element=dtype, cc=cc, cluster_shape=[1, 1, 1],
                         opclass=cutlass_cppgen.OpcodeClass.TensorOp, threadblock_shape=[128, 128, 64],
                         warp_count=[2, 2, 1], stages=3, element_accumulator=cutlass_cppgen.DataType.f32)
```

**EN:** CUTLASS logging is reduced to warnings, the target architecture is fixed at SM80, and `dtype` is set to `f16` as the base datatype.
The empty `GemmMixedSm80` class is guarded by the usual compute-capability and PyTorch-datatype checks.
`add_test_mixed` pre-binds almost every common kernel property: class, base element type, target CC, TensorOp execution, threadblock shape `128x128x64`, warp count `[2,2,1]`, three pipeline stages, and FP32 accumulation.

**CN:** 文件先把 CUTLASS 日志降为 warning，把目标架构固定为 SM80，并将 `dtype` 设为 `f16` 作为基础数据类型。
空壳类 `GemmMixedSm80` 仍然挂着常见的两层保护：检查计算能力，以及检查 PyTorch 是否支持该类型。
`add_test_mixed` 预绑定了几乎所有公共内核属性：所属类、基础元素类型、目标 CC、TensorOp 执行方式、`128x128x64` 的 threadblock、`[2,2,1]` warp 数、3 个流水阶段以及 FP32 累加。

### Lines 65-75 / 第 65-75 行

```python
# Test with upcast on A
add_test_mixed(element_A=cutlass_cppgen.DataType.s8, alignments=[16, 8, 8], layouts=LayoutCombination.TNT)
add_test_mixed(element_A=cutlass_cppgen.DataType.s8, alignments=[16, 8, 8], layouts=LayoutCombination.TNN)

# Test with upcast on B
add_test_mixed(element_B=cutlass_cppgen.DataType.s8, alignments=[8, 16, 8], layouts=LayoutCombination.TNT)
add_test_mixed(element_B=cutlass_cppgen.DataType.s8, alignments=[8, 16, 8], layouts=LayoutCombination.TNN)


if __name__ == '__main__':
    unittest.main()
```

**EN:** The remaining lines enumerate the actual mixed-operand cases.
Two calls override `element_A` to `s8`, which means operand A is upcast into the FP16/FP32 computation path; the tests cover both `TNT` and `TNN` layouts.
Two more calls instead override `element_B` to `s8`, again covering both layouts. The alignment triplets differ depending on which operand is INT8, reflecting per-operand alignment expectations.
The module ends with a standard `unittest.main()` guard.

**CN:** 余下代码就是具体的混合操作数组合。
前两次调用把 `element_A` 改成 `s8`，表示 A 操作数会在后续 FP16/FP32 计算路径中被提升处理；测试同时覆盖 `TNT` 与 `TNN` 两种布局。
再后两次调用把 `element_B` 改成 `s8`，同样覆盖两种布局。由于哪一侧是 INT8 会影响访问约束，所以对齐三元组也随之变化。
文件末尾照例用 `unittest.main()` 作为入口。

## Key Concepts / 关键概念

- **Mixed operand typing / 混合操作数类型:** The helper keeps the overall kernel in an FP16/FP32 regime while selectively overriding one input tensor to INT8. / 辅助函数让整体内核保持 FP16/FP32 路径，同时选择性地把一个输入张量改成 INT8。
- **Common-config partial / 公共配置的 partial:** A single `partial` call removes repetition across all four mixed test variants. / 通过一次 `partial` 绑定去掉四个混合测试变体中的重复配置。
- **Layout coverage / 布局覆盖:** Both `TNT` and `TNN` are exercised for A-upcast and B-upcast cases. / 无论是 A 提升还是 B 提升，都覆盖了 `TNT` 与 `TNN` 两种布局。

## Dependencies / 依赖关系

- **Python stdlib / Python 标准库:** `functools.partial`, `logging`, and `unittest` support parameter binding, logging control, and test discovery. / `functools.partial`、`logging` 与 `unittest` 分别负责参数绑定、日志控制和测试发现。
- **`cutlass_cppgen` / `cutlass_cppgen`:** Provides datatype enums and opcode classes used to declare mixed GEMM kernels. / 提供声明混合 GEMM 内核所需的数据类型枚举与 Opcode 类。
- **`device_cc` / `device_cc`:** Supplies the SM version check used by the decorators. / 提供装饰器所需的 SM 版本检查。
- **Local `utils` / 本地 `utils`:** Supplies `LayoutCombination` and the dynamic `add_test_gemm` registration helper. / 提供 `LayoutCombination` 以及动态注册测试的 `add_test_gemm` 辅助函数。
