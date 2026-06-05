# test_fp8_kernel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/quant/test_fp8_kernel.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates fp8 kernel behavior in SGLang's quant area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 quant 领域中与 fp8 kernel 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch

from sglang.srt.layers.quantization.fp8_kernel import (
    per_token_group_quant_fp8,
    w8a8_block_fp8_matmul,
)
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `sglang.srt.layers.quantization.fp8_kernel`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `sglang.srt.layers.quantization.fp8_kernel`, `sglang.test.ci.ci_register`。

### Lines 12-12: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=10, stage="base-b", runner_config="1-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 14-14: module imports and dependencies / 模块导入与依赖
```python
from sglang.srt.utils import get_device, is_cuda, is_xpu
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.srt.utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.srt.utils`。

### Lines 16-19: module-level constants and configuration / 模块级常量与配置
```python
_is_cuda = is_cuda()
_is_xpu = is_xpu()

device = get_device()
```
**EN:** This block defines shared names such as `_is_cuda`, `_is_xpu`, `device`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `_is_cuda`, `_is_xpu`, `device` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 22-22: class TestFP8Base declaration / 类 TestFP8Base 声明
```python
class TestFP8Base(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 23-31: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.M = 256
        # test non-aligned
        cls.N = 1024 + 64
        cls.K = 512
        cls.group_size = 128
        cls.quant_type = torch.float8_e4m3fn
        cls.output_type = torch.bfloat16
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 33-54: method make A / 方法 make A
```python
    @staticmethod
    def _make_A(M, K, group_size, out_dtype):
        quant_A = torch.rand(
            M, K // group_size, group_size, dtype=torch.float32, device=device
        )
        # -1 ~ 1
        quant_A = quant_A * 2 - 1
        # scaling abs max to fmax
        finfo = torch.finfo(out_dtype)
        fmax = finfo.max
        scaling = fmax / quant_A.abs().amax(-1, keepdim=True)
        quant_A *= scaling
        quant_A = quant_A.to(out_dtype).to(torch.float32)

        # create scale and A
        scale = torch.rand(M, K // group_size, dtype=torch.float32, device=device)
        scale /= fmax
        A = quant_A * scale[..., None]

        A = A.reshape(M, K)
        quant_A = quant_A.reshape(M, K).to(out_dtype)
        return A, quant_A, scale
```
**EN:** This block implements `_make_A` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_A`，承担模块行为中的一个聚焦逻辑片段。

### Lines 56-96: method make B / 方法 make B
```python
    @staticmethod
    def _make_B(K, N, group_size, out_dtype):
        def _aligned_size(a, b):
            return (a + b - 1) // b * b

        K_aligned = _aligned_size(K, group_size)
        N_aligned = _aligned_size(N, group_size)

        quant_B = torch.rand(
            K_aligned // group_size,
            group_size,
            N_aligned // group_size,
            group_size,
            dtype=torch.float32,
            device=device,
        )
        quant_B = quant_B * 2 - 1

        # scaling abs max to fmax
        finfo = torch.finfo(out_dtype)
        fmax = finfo.max
        scaling = fmax / quant_B.abs().amax((1, 3), keepdim=True)
        quant_B *= scaling
        quant_B = quant_B.to(out_dtype).to(torch.float32)

        scale = torch.rand(
            K_aligned // group_size,
            1,
            N_aligned // group_size,
            1,
            dtype=torch.float32,
            device=device,
        )
        scale /= fmax

        B = quant_B * scale

        B = B.reshape(K_aligned, N_aligned)[:K, :N]
        quant_B = quant_B.reshape(K_aligned, N_aligned).to(out_dtype)[:K, :N]
        scale = scale.reshape(K_aligned // group_size, N_aligned // group_size)
        return B, quant_B, scale
```
**EN:** This block implements `_make_B` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_B`，承担模块行为中的一个聚焦逻辑片段。

### Lines 99-99: class TestPerTokenGroupQuantFP8 declaration / 类 TestPerTokenGroupQuantFP8 声明
```python
class TestPerTokenGroupQuantFP8(TestFP8Base):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestFP8Base`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestFP8Base`。

### Lines 100-113: test case per token group quant fp8 / 测试用例 per token group quant fp8
```python
    def test_per_token_group_quant_fp8(self):
        if _is_cuda and torch.cuda.get_device_capability()[0] < 9:
            return

        A, A_quant_gt, scale_gt = self._make_A(
            M=self.M, K=self.K, group_size=self.group_size, out_dtype=self.quant_type
        )
        A_quant, scale = per_token_group_quant_fp8(
            x=A.to(torch.bfloat16), group_size=self.group_size
        )
        torch.testing.assert_close(scale, scale_gt)
        diff = (A_quant.to(torch.float16) - A_quant_gt.to(torch.float16)).abs()
        diff_count = (diff > 1e-5).count_nonzero()
        assert diff_count / diff.numel() < 1e-4
```
**EN:** This test exercises `test_per_token_group_quant_fp8` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_per_token_group_quant_fp8`。

### Lines 116-116: class TestW8A8BlockFP8Matmul declaration / 类 TestW8A8BlockFP8Matmul 声明
```python
class TestW8A8BlockFP8Matmul(TestFP8Base):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestFP8Base`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestFP8Base`。

### Lines 117-141: test case w8a8 block fp8 matmul / 测试用例 w8a8 block fp8 matmul
```python
    def test_w8a8_block_fp8_matmul(self):
        if _is_cuda and torch.cuda.get_device_capability()[0] < 9:
            return
        elif _is_xpu:
            # XPU doesn't provide traditional capability info like CUDA
            pass
        else:
            return

        A, A_quant_gt, A_scale_gt = self._make_A(
            M=self.M, K=self.K, group_size=self.group_size, out_dtype=self.quant_type
        )
        B, B_quant_gt, B_scale_gt = self._make_B(
            K=self.K, N=self.N, group_size=self.group_size, out_dtype=self.quant_type
        )
        C_gt = A.to(self.output_type) @ B.to(self.output_type)
        C = w8a8_block_fp8_matmul(
            A=A_quant_gt,
            B=B_quant_gt.T.contiguous(),
            As=A_scale_gt,
            Bs=B_scale_gt.T.contiguous(),
            block_size=[128, 128],
            output_dtype=self.output_type,
        )
        torch.testing.assert_close(C, C_gt, atol=0.5, rtol=1e-4)
```
**EN:** This test exercises `test_w8a8_block_fp8_matmul` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_w8a8_block_fp8_matmul`。

### Lines 144-145: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestFP8Base`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestPerTokenGroupQuantFP8`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestW8A8BlockFP8Matmul`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFP8Base.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestFP8Base._make_A`: This block implements `_make_A` and captures one focused piece of the module's behavior. / 该代码块实现 `_make_A`，承担模块行为中的一个聚焦逻辑片段。
- `TestFP8Base._make_B`: This block implements `_make_B` and captures one focused piece of the module's behavior. / 该代码块实现 `_make_B`，承担模块行为中的一个聚焦逻辑片段。
- `TestPerTokenGroupQuantFP8.test_per_token_group_quant_fp8`: This test exercises `test_per_token_group_quant_fp8` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_per_token_group_quant_fp8`。
- `TestW8A8BlockFP8Matmul.test_w8a8_block_fp8_matmul`: This test exercises `test_w8a8_block_fp8_matmul` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_w8a8_block_fp8_matmul`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.layers.quantization.fp8_kernel`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`, `sglang.srt.utils`

- **Total lines / 总行数**: 145
