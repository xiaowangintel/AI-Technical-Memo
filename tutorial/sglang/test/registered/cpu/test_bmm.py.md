# test_bmm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/cpu/test_bmm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on cpu bmm in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 cpu bmm 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: Import dependencies
```python
import itertools
import unittest
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 5-11: Import dependencies
```python
import torch
import torch.nn as nn
from utils import precision

from sglang.srt.layers.quantization.fp8_utils import input_to_float8
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 13-13: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-b-test-cpu")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 15-15: Implement expr logic
```python
torch.manual_seed(1234)
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 18-18: Define class Mod
```python
class Mod(nn.Module):
```
**EN:** This declaration introduces the `Mod` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `Mod` 测试类，并说明它通过继承承担的职责。

### Lines 19-21: Define helper: init
```python
    def __init__(self, input_channel, output_channel, has_bias):
        super(Mod, self).__init__()
        self.linear = torch.nn.Linear(input_channel, output_channel, has_bias)
```
**EN:** This helper function encapsulates reusable logic inside `Mod` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `Mod` 内部调用，从而让场景结构更清晰。

### Lines 23-24: Define helper: forward
```python
    def forward(self, x):
        return self.linear(x)
```
**EN:** This helper function encapsulates reusable logic inside `Mod` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `Mod` 内部调用，从而让场景结构更清晰。

### Lines 27-27: Define class TestBmm
```python
class TestBmm(CustomTestCase):
```
**EN:** This declaration introduces the `TestBmm` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestBmm` 测试类，并说明它通过继承承担的职责。

### Lines 28-32: Declare TestBmm configuration
```python
    M = [1, 2, 11, 111]
    N = [128 + 32, 512]
    K = [512 + 32, 128 + 32]
    B = [1, 16, 17]
    chunk = [True, False]
```
**EN:** This block defines class-level settings that are shared across the `TestBmm` test methods.
**CN:** 该代码块定义了 `TestBmm` 各测试方法共享的类级配置。

### Lines 34-47: Define helper: get bmm inputs
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
**EN:** This helper function encapsulates reusable logic inside `TestBmm` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestBmm` 内部调用，从而让场景结构更清晰。

### Lines 49-61: Define helper: bf16 bmm
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
**EN:** This helper function encapsulates reusable logic inside `TestBmm` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestBmm` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 63-76: Define helper: fp8 bmm
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
**EN:** This helper function encapsulates reusable logic inside `TestBmm` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestBmm` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 78-94: Run test: bmm
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
**EN:** This test method exercises bmm and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 bmm 场景，并验证观测到的行为是否符合预期契约。

### Lines 97-98: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。
- Shared base classes centralize common setup, assertions, and diagnostics. / 共享基类集中封装了通用初始化、断言与诊断逻辑。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.layers.quantization.fp8_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `itertools`, `torch`, `torch.nn`, `unittest`, `utils`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
