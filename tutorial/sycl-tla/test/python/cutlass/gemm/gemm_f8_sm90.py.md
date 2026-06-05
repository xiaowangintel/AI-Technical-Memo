# gemm_f8_sm90.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/python/cutlass/gemm/gemm_f8_sm90.py`
- **Purpose (EN):** Registers dynamic SM90 FP8 GEMM unit tests for E4M3 and E5M2 kernels, covering TensorOp and a small SIMT sanity case.
- **用途 (CN):** 为 SM90 上的 E4M3 与 E5M2 FP8 GEMM 内核注册动态单元测试，覆盖 TensorOp 配置以及一个简化的 SIMT 校验用例。

## Line-by-Line Analysis / 逐行分析

### Lines 33-45 / 第 33-45 行

```python
"""
Low-level functionality tests for GEMM with S8 operands on SM90
"""

from functools import partial
import logging
import unittest

import cutlass_cppgen
from cutlass_cppgen.backend.utils.device import device_cc

from utils import LayoutCombination, add_test_gemm

```

**EN:** The module-level docstring says “S8 operands on SM90”, but the implementation below actually builds FP8 tests. That makes the docstring look inherited or stale.
The imports establish the pattern used across this directory: `partial` for pre-binding parameters, `unittest` for discovery, CUTLASS Python bindings for kernel descriptions, and local helpers from `utils` for layout enums and dynamic test attachment.

**CN:** 模块级文档字符串写的是“SM90 上的 S8 操作数”，但下面真正注册的是 FP8 测试，因此这段说明看起来像是沿用旧文件后未更新的注释。
这些导入体现了该目录的通用写法：用 `partial` 预绑定参数，用 `unittest` 做测试发现，用 CUTLASS Python 绑定描述内核，再通过本地 `utils` 中的布局枚举与动态挂载函数生成测试方法。

### Lines 47-61 / 第 47-61 行

```python
cutlass_cppgen.set_log_level(logging.WARNING)
cc = 90
dtype = cutlass_cppgen.DataType.e4m3


@unittest.skipIf(device_cc() < cc, 'Device compute capability is insufficient for SM90 tests.')
@unittest.skipIf(cutlass_cppgen.utils.datatypes.torch_type(dtype) is None, f'Version of torch installed does not contain a datatype match for {dtype}')
class GemmF8E4M3Sm90(unittest.TestCase):
    """
    Wrapper class to which tests will be added dynamically in __main__
    """
    pass


add_test_specialized = partial(add_test_gemm, cls=GemmF8E4M3Sm90, element=dtype, compilation_modes=['nvcc'])
```

**EN:** The file reduces CUTLASS logging noise, fixes the target architecture at compute capability 90, and starts with FP8 E4M3 as the operand type.
The two `skipIf` decorators protect the suite from running when the device is older than SM90 or when the installed PyTorch build lacks an FP8 mapping for this datatype.
`GemmF8E4M3Sm90` is intentionally empty; it is only a container that will receive generated test methods. The `partial` call binds the class, datatype, and `nvcc` compilation mode into `add_test_gemm`.

**CN:** 文件先把 CUTLASS 日志降到 warning，再把目标架构固定为计算能力 90，并以 FP8 E4M3 作为第一组操作数类型。
两个 `skipIf` 装饰器用于防御式跳过：如果设备不是 SM90，或当前 PyTorch 版本没有该 FP8 类型的映射，就不执行这组测试。
`GemmF8E4M3Sm90` 本身是空类，只作为动态生成测试方法的挂载容器；后面的 `partial` 则把类、数据类型和 `nvcc` 编译模式预先绑定到 `add_test_gemm`。

### Lines 63-84 / 第 63-84 行

```python
add_test_tensorop = partial(add_test_specialized, opclass=cutlass_cppgen.OpcodeClass.TensorOp)

# Test with 1x1x1 clusters
add_test_tensorop(layouts=LayoutCombination.TNT, alignments=[16, 16, 16], element_output=cutlass_cppgen.DataType.e4m3,
                  element_accumulator=cutlass_cppgen.DataType.f32, cluster_shape=[1, 1, 1], threadblock_shape=[128, 128, 128], stages=None)

# Tests with different cluster shapes
add_test_tensorop(layouts=LayoutCombination.TNT, alignments=[16, 16, 16], element_output=cutlass_cppgen.DataType.e4m3,
                  element_accumulator=cutlass_cppgen.DataType.f32, cluster_shape=[2, 2, 1], threadblock_shape=[128, 128, 128], stages=None)
add_test_tensorop(layouts=LayoutCombination.TNT, alignments=[16, 16, 16], element_output=cutlass_cppgen.DataType.e4m3,
                  element_accumulator=cutlass_cppgen.DataType.f32, cluster_shape=[1, 4, 1], threadblock_shape=[128, 128, 128], stages=None)

# Tests with warp-specialized ping-pong schedule
add_test_tensorop(layouts=LayoutCombination.TNT, alignments=[16, 16, 16], element_output=cutlass_cppgen.DataType.e4m3,
                  element_accumulator=cutlass_cppgen.DataType.f32, cluster_shape=[2, 1, 1], threadblock_shape=[128, 128, 128], stages=None,
                  kernel_schedule=cutlass_cppgen.KernelScheduleType.TmaWarpSpecializedPingpong,
                  epilogue_schedule=cutlass_cppgen.EpilogueScheduleType.TmaWarpSpecialized)

# Tests for SIMT
add_test_simt = partial(add_test_specialized, opclass=cutlass_cppgen.OpcodeClass.Simt)
add_test_simt(layouts=LayoutCombination.TNN, alignments=[1, 1, 1], element_output=cutlass_cppgen.DataType.e4m3,
              element_accumulator=cutlass_cppgen.DataType.f32, cluster_shape=[1, 1, 1], threadblock_shape=[64, 32, 8], stages=2)
```

**EN:** `add_test_tensorop` further specializes the helper to Tensor Core kernels. The next calls enumerate several representative configurations instead of writing explicit test methods.
The first block uses a standard `1x1x1` cluster. Two more calls keep the same tile size but vary cluster shape, which exercises SM90 cluster launch behavior.
Another call enables the warp-specialized ping-pong/TMA schedule pair, and the final block switches to `OpcodeClass.Simt` with scalar alignment and a much smaller tile to validate a non-TensorOp path.

**CN:** `add_test_tensorop` 在上一层 `partial` 基础上继续固定为 Tensor Core 内核，后续多个调用用“参数枚举”取代了手写测试函数。
第一组是标准的 `1x1x1` cluster；接着两组保持 threadblock 不变，只改变 cluster 形状，用来覆盖 SM90 的 cluster 启动形态。
再下一组打开 warp-specialized 的 ping-pong/TMA 调度组合；最后一组切到 `OpcodeClass.Simt`，使用标量对齐和更小的 tile，验证非 TensorOp 路径也能工作。

### Lines 87-112 / 第 87-112 行

```python
#
# Add a test for E5M2
#
dtype = cutlass_cppgen.DataType.e5m2


@unittest.skipIf(device_cc() < cc, 'Device compute capability is insufficient for SM90 tests.')
@unittest.skipIf(cutlass_cppgen.utils.datatypes.torch_type(dtype) is None, f'Version of torch installed does not contain a datatype match for {dtype}')
class GemmF8E5M2Sm90(unittest.TestCase):
    """
    Wrapper class to which tests will be added dynamically in __main__
    """
    pass


add_test_specialized = partial(add_test_gemm, cls=GemmF8E5M2Sm90, element=dtype, compilation_modes=['nvcc'])

add_test_tensorop = partial(add_test_specialized, opclass=cutlass_cppgen.OpcodeClass.TensorOp)

# Tests with 1x1x1 clusters
add_test_tensorop(layouts=LayoutCombination.TNN, alignments=[16, 16, 16], element_output=dtype,
                  element_accumulator=cutlass_cppgen.DataType.f32, cluster_shape=[1, 1, 1], threadblock_shape=[128, 128, 128], stages=3)


if __name__ == '__main__':
    unittest.main()
```

**EN:** The file then reassigns `dtype` to FP8 E5M2 and repeats the same pattern with a second wrapper class.
This second class gets its own skip guards and its own `partial`, so E5M2 tests are isolated from the E4M3 class in unittest output.
Only one E5M2 TensorOp case is registered here, using `TNN`, 128³ threadblocks, and three pipeline stages. The `__main__` guard delegates execution to `unittest.main()`.

**CN:** 随后文件把 `dtype` 改为 FP8 E5M2，并用同样的结构再定义第二组包装类。
这组测试拥有独立的跳过条件与独立的 `partial` 绑定，因此在 unittest 输出中会与 E4M3 用例分开显示。
这里仅注册了一个 E5M2 TensorOp 用例：使用 `TNN` 布局、128³ threadblock 和 3 个流水阶段。末尾的 `__main__` 分支直接调用 `unittest.main()` 执行测试。

## Key Concepts / 关键概念

- **Dynamic test registration / 动态测试注册:** The file never writes concrete test methods by hand; it generates them through repeated `add_test_gemm(...)` calls. / 文件没有手写具体测试方法，而是通过重复调用 `add_test_gemm(...)` 动态生成。
- **Hardware and dtype gating / 硬件与数据类型门控:** Execution is guarded by both compute capability checks and PyTorch datatype availability checks. / 执行前同时检查设备计算能力和 PyTorch 是否支持对应数据类型。
- **TensorOp vs. SIMT coverage / TensorOp 与 SIMT 覆盖:** Most cases target Tensor Cores, while one SIMT case keeps a baseline software-style path exercised. / 大多数用例覆盖 Tensor Core，另有一个 SIMT 用例保留基础的非 TensorOp 路径。
- **FP8 variant split / FP8 变体拆分:** E4M3 and E5M2 are represented as separate unittest classes, making failures easier to localize. / E4M3 与 E5M2 分别放在不同的 unittest 类中，便于定位失败来源。

## Dependencies / 依赖关系

- **`functools.partial` + `unittest` / `functools.partial` + `unittest`:** Provide parameter binding and Python test discovery/execution. / 提供参数预绑定以及 Python 测试发现/执行能力。
- **`cutlass_cppgen` / `cutlass_cppgen`:** Supplies datatype enums, opcode classes, schedule enums, and runtime helpers used to define each GEMM case. / 提供数据类型枚举、Opcode 类、调度枚举和定义 GEMM 用例所需的运行时辅助能力。
- **`device_cc` / `device_cc`:** Reports device compute capability so the suite can skip unsupported GPUs. / 返回设备计算能力，供测试在不支持的 GPU 上自动跳过。
- **Local `utils` helpers / 本地 `utils` 辅助函数:** `LayoutCombination` enumerates operand layouts and `add_test_gemm` attaches generated tests to the class. / `LayoutCombination` 枚举操作数布局，`add_test_gemm` 负责把生成的测试方法挂到类上。
