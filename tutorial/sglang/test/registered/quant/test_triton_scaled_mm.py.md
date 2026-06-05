# test_triton_scaled_mm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/quant/test_triton_scaled_mm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates triton scaled mm behavior in SGLang's quant area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 quant 领域中与 triton scaled mm 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: module imports and dependencies / 模块导入与依赖
```python
import unittest
from typing import Optional

import torch
import torch.testing

from sglang.srt.layers.quantization.fp8_kernel import triton_scaled_mm
from sglang.srt.utils.common import get_device
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `typing`, `torch`, `torch.testing`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `typing`, `torch`, `torch.testing`。

### Lines 12-13: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=11, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=12, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 16-29: function torch scaled mm / 函数 torch scaled mm
```python
def torch_scaled_mm(
    a: torch.Tensor,
    b: torch.Tensor,
    scale_a: torch.Tensor,
    scale_b: torch.Tensor,
    out_dtype: torch.dtype,
    bias: Optional[torch.Tensor] = None,
) -> torch.Tensor:
    """Reference implementation using float32 for stability"""
    out = torch.mm(a.to(torch.float32), b.to(torch.float32))
    out = scale_a.to(torch.float32) * out * scale_b.to(torch.float32).T
    if bias is not None:
        out = out + bias.to(torch.float32)
    return out.to(out_dtype)
```
**EN:** Reference implementation using float32 for stability This block implements `torch_scaled_mm` and captures one focused piece of the module's behavior.
**CN:** Reference implementation using float32 for stability 该代码块实现 `torch_scaled_mm`，承担模块行为中的一个聚焦逻辑片段。

### Lines 32-32: class TestScaledMM declaration / 类 TestScaledMM 声明
```python
class TestScaledMM(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 33-38: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        if not (torch.cuda.is_available() or torch.xpu.is_available()):
            raise unittest.SkipTest("No CUDA or XPU device available")
        cls._device = get_device()
        torch.set_default_device(cls._device)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 40-55: method make inputs / 方法 make inputs
```python
    def _make_inputs(self, M, K, N, in_dtype):
        if in_dtype == torch.int8:
            a = torch.randint(-8, 8, (M, K), dtype=in_dtype, device=self._device)
            b = torch.randint(-8, 8, (K, N), dtype=in_dtype, device=self._device)
        else:  # fp8
            a = torch.clamp(
                0.1 * torch.randn((M, K), dtype=torch.float16, device=self._device),
                -0.3,
                0.3,
            ).to(in_dtype)
            b = torch.clamp(
                0.1 * torch.randn((K, N), dtype=torch.float16, device=self._device),
                -0.3,
                0.3,
            ).to(in_dtype)
        return a, b
```
**EN:** This block implements `_make_inputs` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_inputs`，承担模块行为中的一个聚焦逻辑片段。

### Lines 57-99: test case basic cases / 测试用例 basic cases
```python
    def test_basic_cases(self):
        """Test core functionality with reduced precision requirements"""
        test_configs = [
            (32, 32, 32, torch.int8, torch.float16, False),
            (64, 64, 64, torch.int8, torch.float16, True),
        ]

        try:
            torch.tensor([1.0], dtype=torch.float8_e4m3fn, device=self._device)
            test_configs.append((32, 32, 32, torch.float8_e4m3fn, torch.float16, False))
        except:
            print("FP8 not supported, skipping")

        for M, K, N, in_dtype, out_dtype, with_bias in test_configs:
            with self.subTest(M=M, K=K, N=N, dtype=in_dtype, bias=with_bias):
                print(f"Currently testing with in_dtype: {in_dtype}")
                torch.manual_seed(42)

                input, weight = self._make_inputs(M, K, N, in_dtype)
                scale_a = 0.1 + 0.05 * torch.rand(
                    (M, 1), dtype=torch.float32, device=self._device
                )
                scale_b = 0.1 + 0.05 * torch.rand(
                    (N, 1), dtype=torch.float32, device=self._device
                )
                bias = (
                    0.01 * torch.randn((M, N), dtype=out_dtype, device=self._device)
                    if with_bias
                    else None
                )

                triton_out = triton_scaled_mm(
                    input, weight, scale_a, scale_b, out_dtype, bias
                )
                ref_out = torch_scaled_mm(
                    input, weight, scale_a, scale_b, out_dtype, bias
                )

                # Use relaxed tolerances
                rtol = 0.15 if in_dtype == torch.int8 else 0.25
                atol = 0.1 if in_dtype == torch.int8 else 0.15

                torch.testing.assert_close(triton_out, ref_out, rtol=rtol, atol=atol)
```
**EN:** Test core functionality with reduced precision requirements This test exercises `test_basic_cases` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test core functionality with reduced precision requirements 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_cases`。

### Lines 102-103: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `torch_scaled_mm`: Reference implementation using float32 for stability / 该代码块实现 `torch_scaled_mm`，承担模块行为中的一个聚焦逻辑片段。
- `TestScaledMM`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestScaledMM.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestScaledMM._make_inputs`: This block implements `_make_inputs` and captures one focused piece of the module's behavior. / 该代码块实现 `_make_inputs`，承担模块行为中的一个聚焦逻辑片段。
- `TestScaledMM.test_basic_cases`: Test core functionality with reduced precision requirements / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_cases`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `typing`
- **Third-party modules / 第三方模块**: `torch`, `torch.testing`
- **Internal modules / 内部模块**: `sglang.srt.layers.quantization.fp8_kernel`, `sglang.srt.utils.common`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 103
