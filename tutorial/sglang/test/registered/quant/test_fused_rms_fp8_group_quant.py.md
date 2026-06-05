# test_fused_rms_fp8_group_quant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/quant/test_fused_rms_fp8_group_quant.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates fused rms fp8 group quant behavior in SGLang's quant area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 quant 领域中与 fused rms fp8 group quant 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: module imports and dependencies / 模块导入与依赖
```python
import itertools
import unittest

import torch
import torch.nn.functional as F

from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `itertools`, `unittest`, `torch`, `torch.nn.functional`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `itertools`, `unittest`, `torch`, `torch.nn.functional`。

### Lines 10-10: CI registration and metadata / CI 注册与元数据
```python
register_amd_ci(est_time=10, suite="stage-a-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_amd_ci.
**CN:** 该代码块通过 register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 13-19: function fp8 available / 函数 fp8 available
```python
def _fp8_available() -> bool:
    # requirement：1) GPU；2) ROCm；3) torch support float8_e4m3fn
    if not torch.cuda.is_available():
        return False
    if getattr(torch.version, "hip", None) is None:
        return False
    return hasattr(torch, "float8_e4m3fn")
```
**EN:** This block implements `_fp8_available` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_fp8_available`，承担模块行为中的一个聚焦逻辑片段。

### Lines 22-26: function rmsnorm / 函数 rmsnorm
```python
def _rmsnorm(x, weight, eps=1e-6):
    # row-wise RMSNorm
    row_norm = (x * x).sum(dim=-1)
    norm = torch.rsqrt(row_norm / x.shape[1] + eps)
    return x * norm[:, None] * weight[None, :]
```
**EN:** This block implements `_rmsnorm` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_rmsnorm`，承担模块行为中的一个聚焦逻辑片段。

### Lines 29-51: function per token fp8 group quant / 函数 per token fp8 group quant
```python
def _per_token_fp8_group_quant(x, dtype_quant, group_size=128):
    """per token、group-size quant, return (quantized, scale)。"""
    DTYPE_MAX = torch.finfo(dtype_quant).max
    M, N = x.shape

    pad = (group_size - (N % group_size)) % group_size
    if pad:
        x_reshape = F.pad(x, (0, pad, 0, 0), "constant", 0)
    else:
        x_reshape = x

    G = (N + group_size - 1) // group_size
    x_reshape = x_reshape.view(M, G, group_size).to(torch.float32)
    x_max = torch.max(torch.abs(x_reshape), dim=-1, keepdim=True)[0].clamp_min_(1e-10)
    x_scale = x_max / DTYPE_MAX
    inv = 1.0 / x_scale

    x_q = torch.clamp(x_reshape * inv, -DTYPE_MAX, DTYPE_MAX).to(dtype_quant)
    x_q = x_q.view(M, G * group_size)
    if pad:
        x_q = x_q[:, :N]
    x_scale = x_scale.squeeze(-1)  # [M, G]
    return x_q, x_scale
```
**EN:** per token、group-size quant, return (quantized, scale)。 This block implements `_per_token_fp8_group_quant` and captures one focused piece of the module's behavior.
**CN:** per token、group-size quant, return (quantized, scale)。 该代码块实现 `_per_token_fp8_group_quant`，承担模块行为中的一个聚焦逻辑片段。

### Lines 54-66: function upcast fp8 group / 函数 upcast fp8 group
```python
def _upcast_fp8_group(x_q, x_s, out_dtype=torch.float32, group_size=128):
    """unqaunt"""
    M, N = x_q.shape
    G = (N + group_size - 1) // group_size
    pad = (group_size - (N % group_size)) % group_size

    if pad:
        x_q = F.pad(x_q, (0, pad, 0, 0), "constant", 0)

    x_q = x_q.view(M, G, group_size).to(torch.float32)
    x = x_q * x_s.view(M, G, 1)
    x = x.view(M, G * group_size)[:, :N]
    return x.to(out_dtype)
```
**EN:** unqaunt This block implements `_upcast_fp8_group` and captures one focused piece of the module's behavior.
**CN:** unqaunt 该代码块实现 `_upcast_fp8_group`，承担模块行为中的一个聚焦逻辑片段。

### Lines 69-70: class TestFusedRMSFP8GroupQuant declaration / 类 TestFusedRMSFP8GroupQuant 声明
```python
class TestFusedRMSFP8GroupQuant(CustomTestCase):
    #
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 71-75: class-level constants and configuration for `TestFusedRMSFP8GroupQuant` / 类级常量与配置
```python
    DTYPES = [torch.bfloat16, torch.float16]
    # (M, N1, N2)
    SHAPES = [(32, 128, 7168), (128, 7168, 7168)]
    GROUP_SIZE = [128]
    SEEDS = [0]
```
**EN:** This block defines shared names such as `DTYPES`, `SHAPES`, `GROUP_SIZE`, `SEEDS`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `DTYPES`, `SHAPES`, `GROUP_SIZE`, `SEEDS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 77-81: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        if not _fp8_available():
            raise unittest.SkipTest("Skip: ROCm/FP8 is not available")
        torch.set_default_device("cuda")
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 83-93: method run ref / 方法 run ref
```python
    def _run_ref(self, x1, w1, eps1, x2, w2, eps2, res1, dtype_quant, group_size):
        s = x1 + (res1 if res1 is not None else 0)
        y1 = _rmsnorm(s, w1, eps1)
        y2 = _rmsnorm(x2, w2, eps2) if x2 is not None else None
        y1_q, y1_s = _per_token_fp8_group_quant(y1, dtype_quant, group_size)
        return (
            (y1_q, y1_s),
            y1.to(x1.dtype),
            (y2.to(x1.dtype) if y2 is not None else None),
            (s.to(x1.dtype) if res1 is not None else None),
        )
```
**EN:** This block implements `_run_ref` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_ref`，承担模块行为中的一个聚焦逻辑片段。

### Lines 95-138: method case / 方法 case
```python
    def _case(self, M, N1, N2, group_size, dtype, seed):
        torch.manual_seed(seed)
        fp8 = torch.float8_e4m3fn
        device = "cuda"

        x1 = torch.randn(M, N1, dtype=dtype, device=device) / 10
        x2 = torch.randn(M, N2, dtype=dtype, device=device) / 10
        w1 = torch.ones(N1, dtype=torch.float32, device=device)
        w2 = torch.ones(N2, dtype=torch.float32, device=device)
        res1 = torch.randn(M, N1, dtype=dtype, device=device) / 10

        # ref
        (y1_q_ref, y1_s_ref), y1_ref, y2_ref, s_ref = self._run_ref(
            x1, w1, 1e-6, x2, w2, 1e-6, res1, fp8, group_size
        )

        # be tested：aiter fused op
        from aiter.ops.triton.fused_fp8_quant import fused_rms_fp8_group_quant

        (y1_q, y1_s), y1, y2, s = fused_rms_fp8_group_quant(
            x1,
            w1,
            1e-6,
            inp2=x2,
            inp2_weight=w2,
            inp2_epsilon=1e-6,
            group_size=group_size,
            dtype_quant=fp8,
            res1=res1,
            output_unquantized_inp1=True,  # get unqaunt y1
        )

        torch.testing.assert_close(y1, y1_ref, atol=0.1, rtol=0.1)
        torch.testing.assert_close(y2, y2_ref, atol=0.1, rtol=0.1)
        torch.testing.assert_close(s, s_ref, atol=0.1, rtol=0.1)

        # check unquant
        y1_up_ref = _upcast_fp8_group(
            y1_q_ref, y1_s_ref, out_dtype=torch.float32, group_size=group_size
        )
        y1_up = _upcast_fp8_group(
            y1_q, y1_s, out_dtype=torch.float32, group_size=group_size
        )
        torch.testing.assert_close(y1_up, y1_up_ref, atol=0.1, rtol=0.1)
```
**EN:** This block implements `_case` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_case`，承担模块行为中的一个聚焦逻辑片段。

### Lines 140-146: test case fused rms fp8 group quant / 测试用例 fused rms fp8 group quant
```python
    def test_fused_rms_fp8_group_quant(self):
        for params in itertools.product(
            self.SHAPES, self.GROUP_SIZE, self.DTYPES, self.SEEDS
        ):
            (M, N1, N2), g, dtype, seed = params
            with self.subTest(M=M, N1=N1, N2=N2, group_size=g, dtype=dtype, seed=seed):
                self._case(M, N1, N2, g, dtype, seed)
```
**EN:** This test exercises `test_fused_rms_fp8_group_quant` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_rms_fp8_group_quant`。

### Lines 149-150: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_fp8_available`: This block implements `_fp8_available` and captures one focused piece of the module's behavior. / 该代码块实现 `_fp8_available`，承担模块行为中的一个聚焦逻辑片段。
- `_rmsnorm`: This block implements `_rmsnorm` and captures one focused piece of the module's behavior. / 该代码块实现 `_rmsnorm`，承担模块行为中的一个聚焦逻辑片段。
- `_per_token_fp8_group_quant`: per token、group-size quant, return (quantized, scale)。 / 该代码块实现 `_per_token_fp8_group_quant`，承担模块行为中的一个聚焦逻辑片段。
- `_upcast_fp8_group`: unqaunt / 该代码块实现 `_upcast_fp8_group`，承担模块行为中的一个聚焦逻辑片段。
- `TestFusedRMSFP8GroupQuant`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFusedRMSFP8GroupQuant.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestFusedRMSFP8GroupQuant._run_ref`: This block implements `_run_ref` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_ref`，承担模块行为中的一个聚焦逻辑片段。
- `TestFusedRMSFP8GroupQuant._case`: This block implements `_case` and captures one focused piece of the module's behavior. / 该代码块实现 `_case`，承担模块行为中的一个聚焦逻辑片段。
- `TestFusedRMSFP8GroupQuant.test_fused_rms_fp8_group_quant`: This test exercises `test_fused_rms_fp8_group_quant` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_rms_fp8_group_quant`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `itertools`, `unittest`
- **Third-party modules / 第三方模块**: `torch`, `torch.nn.functional`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 150
