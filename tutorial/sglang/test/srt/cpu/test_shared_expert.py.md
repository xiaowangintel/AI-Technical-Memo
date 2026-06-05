# test_shared_expert.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/cpu/test_shared_expert.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates shared expert behavior in SGLang's srt / cpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / CPU 领域中与 shared expert 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: module imports and dependencies / 模块导入与依赖
```python
import itertools
import math
import unittest

import torch
from utils import (
    BLOCK_K,
    BLOCK_N,
    factor_for_scale,
    fp8_max,
    fp8_min,
    per_token_quant_int8,
    precision,
    scaled_weight,
    torch_naive_moe,
    torch_w8a8_per_column_moe,
)

from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `itertools`, `math`, `unittest`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `itertools`, `math`, `unittest`, `torch`。

### Lines 21-21: supporting statements / 辅助语句
```python
torch.manual_seed(1234)
```
**EN:** This block performs supporting work through calls such as `manual_seed`, preparing state for nearby definitions.
**CN:** 该代码块通过 `manual_seed` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 24-24: class TestSharedExpert declaration / 类 TestSharedExpert 声明
```python
class TestSharedExpert(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 25-33: class-level constants and configuration for `TestSharedExpert` / 类级常量与配置
```python
    M = [2, 121]
    N = [32, 32 * 4]
    K = [32, 32 * 2]
    routed_scaling_factor = [16]
    apply_scaling_factor = [True, False]

    M_fp8 = [2, 12]
    N_fp8 = [512]
    K_fp8 = [256]
```
**EN:** This block defines shared names such as `M`, `N`, `K`, `routed_scaling_factor`, `apply_scaling_factor`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `M`, `N`, `K`, `routed_scaling_factor`, `apply_scaling_factor` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 35-74: method bf16 shared expert / 方法 bf16 shared expert
```python
    def _bf16_shared_expert(self, m, n, k, routed_scaling_factor, apply_scaling_factor):
        dtype = torch.bfloat16

        hidden_states = torch.randn(m, k, dtype=dtype) / k
        w1 = torch.randn(2 * n, k, dtype=dtype)
        w2 = torch.randn(k, n, dtype=dtype)
        fused_output = (
            torch.randn(m, k, dtype=dtype) / k if apply_scaling_factor else None
        )
        routed_scaling_factor = routed_scaling_factor if apply_scaling_factor else None

        # fused moe mutates content in hs
        hidden_states2 = hidden_states.clone()

        # bfloat16
        ref = torch_naive_moe(
            hidden_states,
            w1,
            w2,
            fused_output,
            routed_scaling_factor,
            output_dtype=dtype,
        )
        out = torch.ops.sgl_kernel.shared_expert_cpu(
            hidden_states2,
            w1,
            w2,
            fused_output,
            routed_scaling_factor,
            True,
            False,
            False,
            None,
            None,
            None,
            False,
        )

        atol = rtol = precision[ref.dtype]
        torch.testing.assert_close(ref, out, atol=atol, rtol=rtol)
```
**EN:** This block implements `_bf16_shared_expert` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_bf16_shared_expert`，承担模块行为中的一个聚焦逻辑片段。

### Lines 76-91: test case bf16 shared expert / 测试用例 bf16 shared expert
```python
    def test_bf16_shared_expert(self):
        for params in itertools.product(
            self.M,
            self.N,
            self.K,
            self.routed_scaling_factor,
            self.apply_scaling_factor,
        ):
            with self.subTest(
                m=params[0],
                n=params[1],
                k=params[2],
                routed_scaling_factor=params[3],
                apply_scaling_factor=params[4],
            ):
                self._bf16_shared_expert(*params)
```
**EN:** This test exercises `test_bf16_shared_expert` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bf16_shared_expert`。

### Lines 93-134: method int8 shared expert / 方法 int8 shared expert
```python
    def _int8_shared_expert(self, m, n, k, routed_scaling_factor, apply_scaling_factor):
        dtype = torch.bfloat16

        hidden_states = torch.randn(m, k, dtype=dtype) / k
        w1 = torch.randn(2 * n, k, dtype=dtype)
        w2 = torch.randn(k, n, dtype=dtype)
        fused_output = (
            torch.randn(m, k, dtype=dtype) / k if apply_scaling_factor else None
        )
        routed_scaling_factor = routed_scaling_factor if apply_scaling_factor else None

        # fused moe mutates content in hs
        hidden_states2 = hidden_states.clone()

        w1_q, w1_s = per_token_quant_int8(w1)
        w2_q, w2_s = per_token_quant_int8(w2)
        ref = torch_w8a8_per_column_moe(
            hidden_states,
            w1_q,
            w2_q,
            w1_s,
            w2_s,
            fused_output,
            routed_scaling_factor,
        )
        out = torch.ops.sgl_kernel.shared_expert_cpu(
            hidden_states2,
            w1_q,
            w2_q,
            fused_output,
            routed_scaling_factor,
            True,
            True,
            False,
            w1_s,
            w2_s,
            None,
            False,
        )

        atol = rtol = precision[ref.dtype]
        torch.testing.assert_close(ref, out, atol=atol, rtol=rtol)
```
**EN:** This block implements `_int8_shared_expert` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_int8_shared_expert`，承担模块行为中的一个聚焦逻辑片段。

### Lines 136-151: test case int8 shared expert / 测试用例 int8 shared expert
```python
    def test_int8_shared_expert(self):
        for params in itertools.product(
            self.M,
            self.N,
            self.K,
            self.routed_scaling_factor,
            self.apply_scaling_factor,
        ):
            with self.subTest(
                m=params[0],
                n=params[1],
                k=params[2],
                routed_scaling_factor=params[3],
                apply_scaling_factor=params[4],
            ):
                self._int8_shared_expert(*params)
```
**EN:** This test exercises `test_int8_shared_expert` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_int8_shared_expert`。

### Lines 153-211: method fp8 shared expert / 方法 fp8 shared expert
```python
    def _fp8_shared_expert(self, m, n, k, routed_scaling_factor, apply_scaling_factor):
        dtype = torch.bfloat16

        hidden_states = torch.randn(m, k, dtype=dtype) / math.sqrt(k)

        w1_fp32 = torch.randn(1, 2 * n, k)
        w1 = (w1_fp32 * fp8_max).clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn)

        w2_fp32 = torch.randn(1, k, n)
        w2 = (w2_fp32 * fp8_max).clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn)

        w1s = torch.randn(1, 2 * n // BLOCK_N, k // BLOCK_K) * factor_for_scale
        w2s = torch.randn(1, k // BLOCK_N, n // BLOCK_K) * factor_for_scale

        w1_scaled = scaled_weight(w1, w1s).view(2 * n, k)
        w2_scaled = scaled_weight(w2, w2s).view(k, n)

        # change back to 2D
        w1, w2 = w1.squeeze(0), w2.squeeze(0)
        w1s, w2s = w1s.squeeze(0), w2s.squeeze(0)
        w1_scaled, w2_scaled = w1_scaled.squeeze(0), w2_scaled.squeeze(0)

        fused_output = (
            torch.randn(m, k, dtype=dtype) / math.sqrt(k)
            if apply_scaling_factor
            else None
        )
        routed_scaling_factor = routed_scaling_factor if apply_scaling_factor else None
        hidden_states2 = hidden_states.clone()

        # ref with bfloat16
        ref = torch_naive_moe(
            hidden_states,
            w1_scaled,
            w2_scaled,
            fused_output,
            routed_scaling_factor,
            output_dtype=dtype,
        )

        w1 = torch.ops.sgl_kernel.convert_weight_packed(w1)  # [2N, K]
        w2 = torch.ops.sgl_kernel.convert_weight_packed(w2)  # [K, N]
        out = torch.ops.sgl_kernel.shared_expert_cpu(
            hidden_states2,
            w1,
            w2,
            fused_output,
            routed_scaling_factor,
            True,
            False,
            True,
            w1s,
            w2s,
            [BLOCK_N, BLOCK_K],
            True,
        )

        atol = rtol = precision[ref.dtype]
        torch.testing.assert_close(ref, out, atol=atol, rtol=rtol)
```
**EN:** This block implements `_fp8_shared_expert` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_fp8_shared_expert`，承担模块行为中的一个聚焦逻辑片段。

### Lines 213-228: test case fp8 shared expert / 测试用例 fp8 shared expert
```python
    def test_fp8_shared_expert(self):
        for params in itertools.product(
            self.M_fp8,
            self.N_fp8,
            self.K_fp8,
            self.routed_scaling_factor,
            self.apply_scaling_factor,
        ):
            with self.subTest(
                m=params[0],
                n=params[1],
                k=params[2],
                routed_scaling_factor=params[3],
                apply_scaling_factor=params[4],
            ):
                self._fp8_shared_expert(*params)
```
**EN:** This test exercises `test_fp8_shared_expert` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fp8_shared_expert`。

### Lines 231-232: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestSharedExpert`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSharedExpert._bf16_shared_expert`: This block implements `_bf16_shared_expert` and captures one focused piece of the module's behavior. / 该代码块实现 `_bf16_shared_expert`，承担模块行为中的一个聚焦逻辑片段。
- `TestSharedExpert.test_bf16_shared_expert`: This test exercises `test_bf16_shared_expert` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bf16_shared_expert`。
- `TestSharedExpert._int8_shared_expert`: This block implements `_int8_shared_expert` and captures one focused piece of the module's behavior. / 该代码块实现 `_int8_shared_expert`，承担模块行为中的一个聚焦逻辑片段。
- `TestSharedExpert.test_int8_shared_expert`: This test exercises `test_int8_shared_expert` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_int8_shared_expert`。
- `TestSharedExpert._fp8_shared_expert`: This block implements `_fp8_shared_expert` and captures one focused piece of the module's behavior. / 该代码块实现 `_fp8_shared_expert`，承担模块行为中的一个聚焦逻辑片段。
- `TestSharedExpert.test_fp8_shared_expert`: This test exercises `test_fp8_shared_expert` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fp8_shared_expert`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `itertools`, `math`, `unittest`
- **Third-party modules / 第三方模块**: `torch`, `utils`
- **Internal modules / 内部模块**: `sglang.test.test_utils`

- **Total lines / 总行数**: 232
