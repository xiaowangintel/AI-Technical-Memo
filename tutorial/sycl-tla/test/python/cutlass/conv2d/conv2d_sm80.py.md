# conv2d_sm80.py — Code Analysis / 代码分析
## Source / 来源
- **Path / 路径:** `test/python/cutlass/conv2d/conv2d_sm80.py`
- **EN:** Builds the SM80 Conv2D unittest matrix by dynamically attaching many test methods that vary data type, opclass, iterator algorithm, split-K mode, swizzle, and activation.
- **CN:** 通过动态挂载大量测试方法，构建面向 SM80 的 Conv2D 单元测试矩阵，覆盖数据类型、opclass、迭代器算法、split-K、swizzle 和激活函数等维度。

## Line-by-Line Analysis / 逐行分析
### Lines 37-58
```python
import logging
import unittest

import cutlass_cppgen
from cutlass_cppgen.backend.utils.device import device_cc

from conv2d_test_utils import *


cutlass_cppgen.set_log_level(logging.WARNING)
cc = 80


@unittest.skipIf(device_cc() < cc, 'Device compute capability is invalid for SM80 tests.')
class Conv2dSm80(unittest.TestCase):
    """
    Wrapper class to which tests will be added dynamically in __main__
    """
    pass


conv_problems = get_conv_problems()
```
**EN:** The file imports logging, `unittest`, CUTLASS Python bindings, and all helpers from `conv2d_test_utils`. It immediately lowers the CUTLASS codegen log level to warnings, fixes the target compute capability at `80`, and declares an otherwise empty `Conv2dSm80` test case class. The `@unittest.skipIf(...)` decorator prevents the whole suite from running on devices whose detected compute capability is below SM80.

**CN:** 文件导入了 logging、`unittest`、CUTLASS Python 绑定，以及 `conv2d_test_utils` 中的全部辅助函数。随后它把 CUTLASS 代码生成日志级别降到 warning，固定目标算力为 `80`，并声明了一个本身为空的 `Conv2dSm80` 测试类。`@unittest.skipIf(...)` 装饰器会在检测到设备算力低于 SM80 时直接跳过整套测试。

### Lines 61-115
```python
# Tests for optimized & analytic
for conv_kind in ["fprop", "wgrad", "dgrad"]:
    # F16, simt
    add_test(
        Conv2dSm80, cc, conv_kind, conv_problems, cutlass_cppgen.DataType.f16, cutlass_cppgen.DataType.f32, cutlass_cppgen.DataType.f16,
        opclass="simt", threadblock_shape=[128, 128, 8],
        warp_count=[4, 2, 1], stages=2, instruction_shape=[1, 1, 1])
    # F16, tensor op
    add_test(
        Conv2dSm80, cc, conv_kind, conv_problems, cutlass_cppgen.DataType.f16, cutlass_cppgen.DataType.f32, cutlass_cppgen.DataType.f16,
        opclass="tensor_op", threadblock_shape=[128, 128, 64],
        warp_count=[2, 2, 1], stages=3, instruction_shape=[16, 8, 16])
    # F16, tensor op, analytic iterator
    add_test(
        Conv2dSm80, cc, conv_kind, conv_problems, cutlass_cppgen.DataType.f16, cutlass_cppgen.DataType.f16, cutlass_cppgen.DataType.f16,
        opclass="tensor_op", threadblock_shape=[128, 128, 64],
        warp_count=[2, 2, 1], stages=3, instruction_shape=[16, 8, 16], iterator_algorithm="analytic")
    # F16, tensor op, f32 output
    add_test(
        Conv2dSm80, cc, conv_kind, conv_problems, cutlass_cppgen.DataType.f16, cutlass_cppgen.DataType.f32, cutlass_cppgen.DataType.f32,
        opclass="tensor_op", threadblock_shape=[128, 128, 64],
        warp_count=[2, 2, 1], stages=3, instruction_shape=[16, 8, 16])
    # F16, tensor op, different tile description
    add_test(
        Conv2dSm80, cc, conv_kind, conv_problems, cutlass_cppgen.DataType.f16, cutlass_cppgen.DataType.f32, cutlass_cppgen.DataType.f16,
        opclass="tensor_op", threadblock_shape=[128, 64, 32],
        warp_count=[2, 2, 1], stages=3, instruction_shape=[16, 8, 8])
    # F32, simt
    add_test(
        Conv2dSm80, cc, conv_kind, conv_problems, cutlass_cppgen.DataType.f32, cutlass_cppgen.DataType.f32, cutlass_cppgen.DataType.f32,
        opclass="simt", threadblock_shape=[128, 128, 8],
        warp_count=[4, 2, 1], stages=4, instruction_shape=[1, 1, 1])
    # Tf32, tensorop
    add_test(
        Conv2dSm80, cc, conv_kind, conv_problems, cutlass_cppgen.DataType.f32, cutlass_cppgen.DataType.f32, cutlass_cppgen.DataType.f32,
        opclass="tensor_op", threadblock_shape=[128, 128, 16],
        warp_count=[2, 2, 1], stages=3, instruction_shape=[16, 8, 8]
    )
    # Split-K
    add_test(
        Conv2dSm80, cc, conv_kind, conv_problems, cutlass_cppgen.DataType.f16, cutlass_cppgen.DataType.f32, cutlass_cppgen.DataType.f16,
        opclass="tensor_op", threadblock_shape=[128, 128, 64],
        warp_count=[2, 2, 1], stages=3, instruction_shape=[16, 8, 16], split_k_mode="serial",
        split_k_slices=2)
    add_test(
        Conv2dSm80, cc, conv_kind, conv_problems, cutlass_cppgen.DataType.f16, cutlass_cppgen.DataType.f32, cutlass_cppgen.DataType.f16,
        opclass="tensor_op", threadblock_shape=[128, 128, 64],
        warp_count=[2, 2, 1], stages=3, instruction_shape=[16, 8, 16], split_k_mode="parallel",
        split_k_slices=5)
    # Swizzling functor
    add_test(
        Conv2dSm80, cc, conv_kind, conv_problems, cutlass_cppgen.DataType.f16, cutlass_cppgen.DataType.f32, cutlass_cppgen.DataType.f16,
        opclass="tensor_op", threadblock_shape=[128, 64, 32],
        warp_count=[2, 2, 1], stages=3, instruction_shape=[16, 8, 8], swizzle=4)

```
**EN:** This loop is the heart of the suite. For each convolution kind (`fprop`, `wgrad`, `dgrad`), the code calls `add_test()` repeatedly to synthesize test methods covering FP16 SIMT, FP16 Tensor Core, analytic iterators, FP32 outputs, alternate tile shapes, FP32 SIMT, TF32 Tensor Core, serial/parallel split-K, and a swizzled kernel. Because `add_test()` installs methods on the class by name, this block declaratively defines a large matrix without writing dozens of manual test functions.

**CN:** 这段循环是测试套件的核心。对于每一种卷积类型（`fprop`、`wgrad`、`dgrad`），代码都会重复调用 `add_test()`，动态生成覆盖 FP16 SIMT、FP16 Tensor Core、analytic 迭代器、FP32 输出、不同 tile 形状、FP32 SIMT、TF32 Tensor Core、串行/并行 split-K 以及 swizzle kernel 的测试方法。由于 `add_test()` 会按名字把方法安装到类上，这里可以用声明式方式定义庞大的测试矩阵，而不需要手写几十个测试函数。

### Lines 116-133
```python
# Tests for few channels and fixed channels
# F16, tensor op, few channels
for c, tb, stage, inst in zip([2, 1],
                                [[128, 128, 64], [128, 128, 32]],
                                [3, 2],
                                [[16, 8, 16], [16, 8, 8]]):
    add_test(
        Conv2dSm80, cc, "fprop", conv2d_few_channel_problemsizes(c), cutlass_cppgen.DataType.f16, cutlass_cppgen.DataType.f32, cutlass_cppgen.DataType.f16,
        opclass="tensor_op", threadblock_shape=tb,
        warp_count=[2, 2, 1], stages=stage, instruction_shape=inst, iterator_algorithm="few_channels"
    )
# F16, tensor op, fixed channels
for c in [8, 4, 2]:
    add_test(
        Conv2dSm80, cc, "fprop", conv2d_few_channel_problemsizes(c), cutlass_cppgen.DataType.f16, cutlass_cppgen.DataType.f32, cutlass_cppgen.DataType.f16,
        opclass="tensor_op", threadblock_shape=[128, 128, 64],
        warp_count=[2, 2, 1], stages=3, instruction_shape=[16, 8, 16], iterator_algorithm="fixed_channels"
    )
```
**EN:** After the main loop, the suite adds special-case forward-propagation tests for few-channel and fixed-channel iterators. The first loop zips channel counts together with matching threadblock shapes, stage counts, and instruction shapes; the second loop reuses one tile configuration while varying the channel count for `fixed_channels` coverage.

**CN:** 主循环之后，测试套件又补充了面向少通道和固定通道迭代器的前向传播特例测试。第一组循环把通道数与对应的 threadblock 形状、stage 数和 instruction shape 配套组合；第二组循环则在复用同一 tile 配置的同时变化通道数，用于覆盖 `fixed_channels` 场景。

### Lines 135-146
```python
# Test activations
for activation in ["relu", "leaky_relu"]:
    for split_k_mode, split_k_slices in zip(["parallel", "serial", "parallel"], [1, 7, 5]):
        add_test(
            Conv2dSm80, cc, "fprop", conv_problems, cutlass_cppgen.DataType.f16, cutlass_cppgen.DataType.f32, cutlass_cppgen.DataType.f16,
            opclass="tensor_op", threadblock_shape=[128, 128, 64],
            warp_count=[2, 2, 1], stages=3, instruction_shape=[16, 8, 16], split_k_mode=split_k_mode,
            split_k_slices=split_k_slices, activation=activation)


if __name__ == '__main__':
    unittest.main()
```
**EN:** The last generated tests focus on epilogue activations. For both `relu` and `leaky_relu`, the file sweeps several split-K configurations and registers each combination through `add_test()`. The `__main__` guard simply delegates execution to `unittest.main()`.

**CN:** 最后一批动态生成的测试聚焦于 epilogue 激活函数。对于 `relu` 和 `leaky_relu`，文件会遍历若干 split-K 配置，并通过 `add_test()` 注册每个组合。末尾的 `__main__` 保护则直接把执行权交给 `unittest.main()`。

## Key Concepts / 关键概念
- **EN:** Dynamic unittest generation: test methods are created programmatically instead of being written one by one.
  **CN:** 动态单元测试生成：测试方法不是逐个手写，而是通过程序自动创建。
- **EN:** Architecture gating: the whole class is skipped when the device cannot satisfy SM80 requirements.
  **CN:** 架构门控：如果设备不满足 SM80 要求，整类测试会被跳过。
- **EN:** Coverage matrix construction: convolution kind, datatype, tile shape, iterator, split-K mode, and activation are all treated as orthogonal dimensions.
  **CN:** 覆盖矩阵构造：卷积类型、数据类型、tile 形状、迭代器、split-K 模式和激活函数都被作为正交维度组合。

## Dependencies / 依赖关系
- **EN:** `cutlass_cppgen` provides datatype enums and the underlying Conv2D plan machinery configured by helper code.
  **CN:** `cutlass_cppgen` 提供数据类型枚举，以及由辅助代码配置的底层 Conv2D plan 机制。
- **EN:** `cutlass_cppgen.backend.utils.device.device_cc` is used to detect the actual GPU compute capability for the `skipIf` gate.
  **CN:** `cutlass_cppgen.backend.utils.device.device_cc` 用于检测真实 GPU 算力，以支持 `skipIf` 跳过逻辑。
- **EN:** `conv2d_test_utils` contributes `add_test()`, `get_conv_problems()`, and `conv2d_few_channel_problemsizes()`, which together provide both the test factory and the input problem sets.
  **CN:** `conv2d_test_utils` 提供了 `add_test()`、`get_conv_problems()` 和 `conv2d_few_channel_problemsizes()`，共同组成测试工厂与输入问题集。
