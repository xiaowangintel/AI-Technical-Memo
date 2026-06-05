# test_shared_expert.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/cpu/test_shared_expert.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on cpu shared expert in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 cpu shared expert 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Import dependencies
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

from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 22-22: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-b-test-cpu")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 24-24: Implement expr logic
```python
torch.manual_seed(1234)
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 27-27: Define class TestSharedExpert
```python
class TestSharedExpert(CustomTestCase):
```
**EN:** This declaration introduces the `TestSharedExpert` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSharedExpert` 测试类，并说明它通过继承承担的职责。

### Lines 28-36: Declare TestSharedExpert configuration
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
**EN:** This block defines class-level settings that are shared across the `TestSharedExpert` test methods.
**CN:** 该代码块定义了 `TestSharedExpert` 各测试方法共享的类级配置。

### Lines 38-77: Define helper: bf16 shared expert
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
**EN:** This helper function encapsulates reusable logic inside `TestSharedExpert` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestSharedExpert` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 79-94: Run test: bf16 shared expert
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
**EN:** This test method exercises bf16 shared expert and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 bf16 shared expert 场景，并验证观测到的行为是否符合预期契约。

### Lines 96-137: Define helper: int8 shared expert
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
**EN:** This helper function encapsulates reusable logic inside `TestSharedExpert` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestSharedExpert` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 139-154: Run test: int8 shared expert
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
**EN:** This test method exercises int8 shared expert and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 int8 shared expert 场景，并验证观测到的行为是否符合预期契约。

### Lines 156-214: Define helper: fp8 shared expert
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
**EN:** This helper function encapsulates reusable logic inside `TestSharedExpert` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestSharedExpert` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 216-231: Run test: fp8 shared expert
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
**EN:** This test method exercises fp8 shared expert and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 fp8 shared expert 场景，并验证观测到的行为是否符合预期契约。

### Lines 234-235: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `itertools`, `math`, `torch`, `unittest`, `utils`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
