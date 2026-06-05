# test_bmm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/cpu/test_bmm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates bmm behavior in SGLang's srt / cpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / CPU 领域中与 bmm 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: module imports and dependencies / 模块导入与依赖
```python
import itertools
import unittest

# TODO: use interface in cpu.py
import torch
import torch.nn as nn
from utils import precision

from sglang.srt.layers.quantization.fp8_utils import input_to_float8
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `itertools`, `unittest`, `torch`, `torch.nn`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `itertools`, `unittest`, `torch`, `torch.nn`。

### Lines 12-12: supporting statements / 辅助语句
```python
torch.manual_seed(1234)
```
**EN:** This block performs supporting work through calls such as `manual_seed`, preparing state for nearby definitions.
**CN:** 该代码块通过 `manual_seed` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 15-15: class Mod declaration / 类 Mod 声明
```python
class Mod(nn.Module):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `nn.Module`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `nn.Module`。

### Lines 16-18: method init / 方法 init
```python
    def __init__(self, input_channel, output_channel, has_bias):
        super(Mod, self).__init__()
        self.linear = torch.nn.Linear(input_channel, output_channel, has_bias)
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 20-21: method forward / 方法 forward
```python
    def forward(self, x):
        return self.linear(x)
```
**EN:** This block implements `forward` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `forward`，承担模块行为中的一个聚焦逻辑片段。

### Lines 24-24: class TestBmm declaration / 类 TestBmm 声明
```python
class TestBmm(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 25-29: class-level constants and configuration for `TestBmm` / 类级常量与配置
```python
    M = [1, 2, 11, 111]
    N = [128 + 32, 512]
    K = [512 + 32, 128 + 32]
    B = [1, 16, 17]
    chunk = [True, False]
```
**EN:** This block defines shared names such as `M`, `N`, `K`, `B`, `chunk`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `M`, `N`, `K`, `B`, `chunk` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 31-44: method get bmm inputs / 方法 get bmm inputs
```python
    def _get_bmm_inputs(self, B, M, N, K, chunk, dtype):
        if chunk:
            mat1 = (
                torch.randn(M, B, K + 64, dtype=dtype).narrow(2, 0, K).transpose_(0, 1)
            )
            mat2 = torch.randn(B, N, K, dtype=dtype).transpose_(1, 2)
            mat3 = (
                torch.randn(M, B, N + 64, dtype=dtype).narrow(2, 0, N).transpose_(0, 1)
            )
        else:
            mat1 = torch.randn(M, B, K, dtype=dtype).transpose_(0, 1)
            mat2 = torch.randn(B, N, K, dtype=dtype).transpose_(1, 2)
            mat3 = torch.randn(M, B, N, dtype=dtype).transpose_(0, 1)
        return mat1, mat2, mat3
```
**EN:** This block implements `_get_bmm_inputs` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_get_bmm_inputs`，承担模块行为中的一个聚焦逻辑片段。

### Lines 46-58: method bf16 bmm / 方法 bf16 bmm
```python
    def _bf16_bmm(self, B, M, N, K, chunk, dtype=torch.bfloat16):
        mat1, mat2, mat3 = self._get_bmm_inputs(B, M, N, K, chunk, dtype)
        ref = torch.bmm(mat1, mat2)
        mat2_t = mat2.transpose_(1, 2)
        mat3.zero_()
        torch.ops.sgl_kernel.bmm_cpu(mat3, mat1, mat2, False, None)
        atol = rtol = precision[ref.dtype]
        torch.testing.assert_close(ref, mat3, atol=atol, rtol=rtol)

        packed_B = torch.ops.sgl_kernel.convert_weight_packed(mat2_t)
        mat3.zero_()
        torch.ops.sgl_kernel.bmm_cpu(mat3, mat1, packed_B, True, None)
        torch.testing.assert_close(ref, mat3, atol=atol, rtol=rtol)
```
**EN:** This block implements `_bf16_bmm` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_bf16_bmm`，承担模块行为中的一个聚焦逻辑片段。

### Lines 60-73: method fp8 bmm / 方法 fp8 bmm
```python
    def _fp8_bmm(self, B, M, N, K, chunk, dtype=torch.bfloat16):
        mat1, mat2, mat3 = self._get_bmm_inputs(B, M, N, K, chunk, dtype)
        mat2_q, mat2_s = input_to_float8(mat2)
        ref = torch.bmm(mat1, mat2_q.to(torch.bfloat16)) * mat2_s
        mat2_q_t = mat2_q.transpose_(1, 2).contiguous()
        mat3.zero_()
        atol = rtol = precision[ref.dtype]
        torch.ops.sgl_kernel.bmm_cpu(mat3, mat1, mat2_q_t, False, mat2_s)
        torch.testing.assert_close(ref, mat3, atol=atol, rtol=rtol)

        packed_B_q = torch.ops.sgl_kernel.convert_weight_packed(mat2_q_t)
        mat3.zero_()
        torch.ops.sgl_kernel.bmm_cpu(mat3, mat1, packed_B_q, True, mat2_s)
        torch.testing.assert_close(ref, mat3, atol=atol, rtol=rtol)
```
**EN:** This block implements `_fp8_bmm` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_fp8_bmm`，承担模块行为中的一个聚焦逻辑片段。

### Lines 75-91: test case bmm / 测试用例 bmm
```python
    def test_bmm(self):
        for params in itertools.product(
            self.B,
            self.M,
            self.N,
            self.K,
            self.chunk,
        ):
            with self.subTest(
                B=params[0],
                M=params[1],
                N=params[2],
                K=params[3],
                chunk=params[4],
            ):
                self._bf16_bmm(*params)
                self._fp8_bmm(*params)
```
**EN:** This test exercises `test_bmm` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bmm`。

### Lines 94-95: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `Mod`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestBmm`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `Mod.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。
- `Mod.forward`: This block implements `forward` and captures one focused piece of the module's behavior. / 该代码块实现 `forward`，承担模块行为中的一个聚焦逻辑片段。
- `TestBmm._get_bmm_inputs`: This block implements `_get_bmm_inputs` and captures one focused piece of the module's behavior. / 该代码块实现 `_get_bmm_inputs`，承担模块行为中的一个聚焦逻辑片段。
- `TestBmm._bf16_bmm`: This block implements `_bf16_bmm` and captures one focused piece of the module's behavior. / 该代码块实现 `_bf16_bmm`，承担模块行为中的一个聚焦逻辑片段。
- `TestBmm._fp8_bmm`: This block implements `_fp8_bmm` and captures one focused piece of the module's behavior. / 该代码块实现 `_fp8_bmm`，承担模块行为中的一个聚焦逻辑片段。
- `TestBmm.test_bmm`: This test exercises `test_bmm` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bmm`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `itertools`, `unittest`
- **Third-party modules / 第三方模块**: `torch`, `torch.nn`, `utils`
- **Internal modules / 内部模块**: `sglang.srt.layers.quantization.fp8_utils`, `sglang.test.test_utils`

- **Total lines / 总行数**: 95
