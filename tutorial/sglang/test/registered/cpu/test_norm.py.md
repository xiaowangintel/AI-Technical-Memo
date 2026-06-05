# test_norm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/cpu/test_norm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on cpu norm in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 cpu norm 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Import dependencies
```python
import itertools
import unittest
from typing import Optional, Tuple, Union

import torch
from utils import make_non_contiguous, parametrize, precision

from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 11-11: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-b-test-cpu")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 13-13: Implement expr logic
```python
torch.manual_seed(1234)
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 16-16: Define class TestNorm
```python
class TestNorm(CustomTestCase):
```
**EN:** This declaration introduces the `TestNorm` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNorm` 测试类，并说明它通过继承承担的职责。

### Lines 18-37: Define helper: forward native
```python
    def _forward_native(
        self,
        x: torch.Tensor,
        weight: torch.Tensor,
        variance_epsilon: float = 1e-6,
        residual: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        orig_dtype = x.dtype
        x = x.to(torch.float32)
        if residual is not None:
            x = x + residual.to(torch.float32)
            residual = x.to(orig_dtype)

        variance = x.pow(2).mean(dim=-1, keepdim=True)
        x = x * torch.rsqrt(variance + variance_epsilon)
        x = x.to(orig_dtype) * weight
        if residual is None:
            return x
        else:
            return x, residual
```
**EN:** This helper function encapsulates reusable logic inside `TestNorm` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestNorm` 内部调用，从而让场景结构更清晰。

### Lines 39-40: Define helper: norm
```python
    def _norm(self, x, eps):
        return x * torch.rsqrt(x.pow(2).mean(-1, keepdim=True) + eps)
```
**EN:** This helper function encapsulates reusable logic inside `TestNorm` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestNorm` 内部调用，从而让场景结构更清晰。

### Lines 42-47: Define helper: gemma3 rmsnorm native
```python
    def _gemma3_rmsnorm_native(
        self, x: torch.Tensor, weight: torch.Tensor, variance_epsilon: float = 1e-6
    ):
        output = self._norm(x.float(), variance_epsilon)
        output = output * (1.0 + weight.float())
        return output.type_as(x)
```
**EN:** This helper function encapsulates reusable logic inside `TestNorm` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestNorm` 内部调用，从而让场景结构更清晰。

### Lines 49-66: Define helper: gemma rmsnorm native
```python
    def _gemma_rmsnorm_native(
        self,
        x: torch.Tensor,
        weight: torch.Tensor,
        variance_epsilon: float = 1e-6,
        residual: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        orig_dtype = x.dtype
        if residual is not None:
            x = x + residual
            residual = x

        x = x.float()
        variance = x.pow(2).mean(dim=-1, keepdim=True)
        x = x * torch.rsqrt(variance + variance_epsilon)
        x = x * (1.0 + weight.float())
        x = x.to(orig_dtype)
        return x if residual is None else (x, residual)
```
**EN:** This helper function encapsulates reusable logic inside `TestNorm` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestNorm` 内部调用，从而让场景结构更清晰。

### Lines 68-100: Run test: norm
```python
    @parametrize(
        m=[4096, 1024],
        n=[4096, 4109],
        dtype=[torch.float16, torch.bfloat16],
    )
    def test_norm(self, m, n, dtype):

        x = torch.randn([m, n], dtype=dtype)
        x = make_non_contiguous(x)
        hidden_size = x.size(-1)
        weight = torch.randn(hidden_size, dtype=dtype)
        variance_epsilon = 1e-6

        out = torch.ops.sgl_kernel.rmsnorm_cpu(x, weight, variance_epsilon)
        ref_out = self._forward_native(x, weight, variance_epsilon)

        atol = rtol = precision[ref_out.dtype]
        torch.testing.assert_close(ref_out, out, atol=atol, rtol=rtol)

        ref_x = x.clone()
        residual = torch.randn([m, hidden_size], dtype=dtype)
        ref_residual = residual.clone()

        torch.ops.sgl_kernel.fused_add_rmsnorm_cpu(
            x, residual, weight, variance_epsilon
        )

        ref_x, ref_residual = self._forward_native(
            ref_x, weight, variance_epsilon, ref_residual
        )

        torch.testing.assert_close(x, ref_x, atol=atol, rtol=rtol)
        torch.testing.assert_close(residual, ref_residual, atol=atol, rtol=rtol)
```
**EN:** This test method exercises norm and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 norm 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 102-135: Run test: norm 3d
```python
    @parametrize(
        l=[1, 2],
        m=[4096, 1024],
        n=[4096, 4109],
        dtype=[torch.float16, torch.bfloat16],
    )
    def test_norm_3d(self, l, m, n, dtype):

        x = torch.randn([l, m, n], dtype=dtype)
        x = make_non_contiguous(x)
        hidden_size = x.size(-1)
        weight = torch.randn(hidden_size, dtype=dtype)
        variance_epsilon = 1e-6

        out = torch.ops.sgl_kernel.rmsnorm_cpu(x, weight, variance_epsilon)
        ref_out = self._forward_native(x, weight, variance_epsilon)

        atol = rtol = precision[ref_out.dtype]
        torch.testing.assert_close(ref_out, out, atol=atol, rtol=rtol)

        ref_x = x.clone()
        residual = torch.randn([l, m, hidden_size], dtype=dtype)
        ref_residual = residual.clone()

        torch.ops.sgl_kernel.fused_add_rmsnorm_cpu(
            x, residual, weight, variance_epsilon
        )

        ref_x, ref_residual = self._forward_native(
            ref_x, weight, variance_epsilon, ref_residual
        )

        torch.testing.assert_close(x, ref_x, atol=atol, rtol=rtol)
        torch.testing.assert_close(residual, ref_residual, atol=atol, rtol=rtol)
```
**EN:** This test method exercises norm 3d and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 norm 3d 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 137-153: Run test: l2norm
```python
    @parametrize(
        m=[4096, 1024],
        n=[4096, 4109],
        dtype=[torch.float16, torch.bfloat16],
    )
    def test_l2norm(self, m, n, dtype):

        x = torch.randn([m, n], dtype=dtype)
        hidden_size = x.size(-1)
        fake_ones_weight = torch.ones(hidden_size, dtype=dtype)
        variance_epsilon = 1e-6

        out = torch.ops.sgl_kernel.l2norm_cpu(x, variance_epsilon)
        ref_out = self._forward_native(x, fake_ones_weight, variance_epsilon)

        atol = rtol = precision[ref_out.dtype]
        torch.testing.assert_close(ref_out, out, atol=atol, rtol=rtol)
```
**EN:** This test method exercises l2norm and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 l2norm 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 155-187: Run test: gemma rmsnorm
```python
    @parametrize(
        m=[4096, 1024],
        n=[4096, 4109],
        dtype=[torch.float16, torch.bfloat16],
    )
    def test_gemma_rmsnorm(self, m, n, dtype):

        x = torch.randn([m, n], dtype=dtype)
        x = make_non_contiguous(x)
        hidden_size = x.size(-1)
        weight = torch.randn(hidden_size, dtype=dtype)
        variance_epsilon = 1e-6

        out = torch.ops.sgl_kernel.gemma_rmsnorm_cpu(x, weight, variance_epsilon)
        ref_out = self._gemma_rmsnorm_native(x, weight, variance_epsilon)

        atol = rtol = precision[ref_out.dtype]
        torch.testing.assert_close(ref_out, out, atol=atol, rtol=rtol)

        ref_x = x.clone()
        residual = torch.randn([m, hidden_size], dtype=dtype)
        ref_residual = residual.clone()

        torch.ops.sgl_kernel.gemma_fused_add_rmsnorm_cpu(
            x, residual, weight, variance_epsilon
        )

        ref_x, ref_residual = self._gemma_rmsnorm_native(
            ref_x, weight, variance_epsilon, ref_residual
        )

        torch.testing.assert_close(x, ref_x, atol=atol, rtol=rtol)
        torch.testing.assert_close(residual, ref_residual, atol=atol, rtol=rtol)
```
**EN:** This test method exercises gemma rmsnorm and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gemma rmsnorm 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 189-208: Run test: gemma3 rmsnorm
```python
    @parametrize(
        m=[4096, 1024],
        n=[4096, 4109],
        dtype=[torch.float16, torch.bfloat16],
    )
    def test_gemma3_rmsnorm(self, m, n, dtype):
        x_list = [
            torch.randn([m, n], dtype=dtype),
            torch.randn([1, m, 2, n], dtype=dtype),
        ]
        for x in x_list:
            x = make_non_contiguous(x)
            hidden_size = x.size(-1)
            weight = torch.randn(hidden_size, dtype=dtype)
            variance_epsilon = 1e-6
            out = torch.ops.sgl_kernel.gemma3_rmsnorm_cpu(x, weight, variance_epsilon)
            ref_out = self._gemma3_rmsnorm_native(x, weight, variance_epsilon)

            atol = rtol = precision[ref_out.dtype]
            torch.testing.assert_close(ref_out, out, atol=atol, rtol=rtol)
```
**EN:** This test method exercises gemma3 rmsnorm and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gemma3 rmsnorm 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 210-221: Define helper: gemma4 rmsnorm native
```python
    def _gemma4_rmsnorm_native(
        self,
        x: torch.Tensor,
        weight: torch.Tensor,
        variance_epsilon: float = 1e-6,
        scale_shift: float = 0.0,
        with_scale: bool = True,
    ):
        output = self._norm(x.float(), variance_epsilon)
        if with_scale:
            output = output * (weight.float() + scale_shift)
        return output.type_as(x)
```
**EN:** This helper function encapsulates reusable logic inside `TestNorm` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestNorm` 内部调用，从而让场景结构更清晰。

### Lines 223-257: Run test: gemma4 rmsnorm
```python
    @parametrize(
        m=[4096, 1024],
        n=[4096, 4109],
        dtype=[torch.float16, torch.bfloat16],
    )
    def test_gemma4_rmsnorm(self, m, n, dtype):
        for scale_shift, with_scale in [
            (0.0, True),
            (1.0, True),
            (0.0, False),
            (1.0, False),
        ]:
            x_list = [
                torch.randn([m, n], dtype=dtype),
                torch.randn([4, m, n], dtype=dtype),
            ]
            # Add non-block-contiguous 3D input
            base = torch.randn([4, 2 * m, n], dtype=dtype)
            x_list.append(base[:, :m, :])

            for x in x_list:
                x = make_non_contiguous(x)
                hidden_size = x.size(-1)
                weight = torch.randn(hidden_size, dtype=dtype)
                variance_epsilon = 1e-6

                out = torch.ops.sgl_kernel.gemma4_rmsnorm_cpu(
                    x, weight, variance_epsilon, scale_shift, with_scale
                )
                ref_out = self._gemma4_rmsnorm_native(
                    x, weight, variance_epsilon, scale_shift, with_scale
                )

                atol = rtol = precision[ref_out.dtype]
                torch.testing.assert_close(ref_out, out, atol=atol, rtol=rtol)
```
**EN:** This test method exercises gemma4 rmsnorm and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gemma4 rmsnorm 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 260-260: Define class TestFusedRMSNormGated
```python
class TestFusedRMSNormGated(CustomTestCase):
```
**EN:** This declaration introduces the `TestFusedRMSNormGated` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFusedRMSNormGated` 测试类，并说明它通过继承承担的职责。

### Lines 261-263: Declare TestFusedRMSNormGated configuration
```python
    M = [4096, 1024]
    N = [4096, 4096 + 13]
    dtype = [torch.float16, torch.bfloat16]
```
**EN:** This block defines class-level settings that are shared across the `TestFusedRMSNormGated` test methods.
**CN:** 该代码块定义了 `TestFusedRMSNormGated` 各测试方法共享的类级配置。

### Lines 265-280: Define helper: forward native
```python
    def _forward_native(
        self,
        hidden_states: torch.Tensor,
        weight: torch.Tensor,
        variance_epsilon: float = 1e-6,
        gate: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        input_dtype = hidden_states.dtype
        hidden_states = hidden_states.to(torch.float32)
        variance = hidden_states.pow(2).mean(-1, keepdim=True)
        # Norm before gate
        hidden_states = hidden_states * torch.rsqrt(variance + variance_epsilon)
        hidden_states = weight * hidden_states.to(input_dtype)
        hidden_states = hidden_states * torch.nn.functional.silu(gate.to(torch.float32))

        return hidden_states.to(input_dtype)
```
**EN:** This helper function encapsulates reusable logic inside `TestFusedRMSNormGated` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestFusedRMSNormGated` 内部调用，从而让场景结构更清晰。

### Lines 282-298: Define helper: norm test
```python
    def _norm_test(self, m, n, dtype):

        x = torch.randn([m, n], dtype=dtype)
        x = make_non_contiguous(x)
        batch_size = x.size(0)
        hidden_size = x.size(-1)
        weight = torch.randn(hidden_size, dtype=dtype)
        variance_epsilon = 1e-6
        gate = torch.randn([batch_size, hidden_size], dtype=dtype)

        out = torch.ops.sgl_kernel.fused_rmsnorm_gated_cpu(
            x, weight, gate, variance_epsilon
        )
        ref_out = self._forward_native(x, weight, variance_epsilon, gate)

        atol = rtol = precision[ref_out.dtype] * 2
        torch.testing.assert_close(ref_out, out, atol=atol, rtol=rtol)
```
**EN:** This helper function encapsulates reusable logic inside `TestFusedRMSNormGated` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestFusedRMSNormGated` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 300-303: Run test: norm
```python
    def test_norm(self):
        for params in itertools.product(self.M, self.N, self.dtype):
            with self.subTest(m=params[0], n=params[1], dtype=params[2]):
                self._norm_test(*params)
```
**EN:** This test method exercises norm and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 norm 场景，并验证观测到的行为是否符合预期契约。

### Lines 306-306: Define class TestLayerNorm
```python
class TestLayerNorm(CustomTestCase):
```
**EN:** This declaration introduces the `TestLayerNorm` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestLayerNorm` 测试类，并说明它通过继承承担的职责。

### Lines 308-328: Define helper: forward native
```python
    def _forward_native(
        self,
        x: torch.Tensor,
        weight: torch.Tensor,
        variance_epsilon: float,
        residual: Optional[torch.Tensor] = None,
        bias: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        orig_dtype = x.dtype
        x = x.to(torch.float32)
        if residual is not None:
            x = x + residual.to(torch.float32)
            residual = x.to(orig_dtype)

        variance, mean = torch.var_mean(x, dim=-1, keepdim=True, correction=0)
        x = (x - mean) * torch.rsqrt(variance + variance_epsilon)
        x = x * weight.to(torch.float32)
        if bias is not None:
            x = x + bias.to(torch.float32)
        x = x.to(orig_dtype)
        return x if residual is None else (x, residual)
```
**EN:** This helper function encapsulates reusable logic inside `TestLayerNorm` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestLayerNorm` 内部调用，从而让场景结构更清晰。

### Lines 330-379: Run test: norm input 2d
```python
    @parametrize(
        m=[4096, 1024],
        n=[4096, 4109],
        dtype=[torch.float16, torch.bfloat16],
    )
    def test_norm_input_2d(self, m: int, n: int, dtype: torch.dtype) -> None:
        x = torch.randn([m, n], dtype=dtype)
        x = make_non_contiguous(x)
        hidden_size = x.size(-1)
        weight = torch.randn(hidden_size, dtype=dtype)
        bias = torch.randn(hidden_size, dtype=dtype)
        variance_epsilon = 1e-6

        ln_out = torch.ops.sgl_kernel.layernorm_cpu(x, weight, None, variance_epsilon)
        ref_ln_out = self._forward_native(x, weight, variance_epsilon)

        atol = rtol = precision[ref_ln_out.dtype]
        torch.testing.assert_close(ln_out, ref_ln_out, atol=atol, rtol=rtol)

        ln_out = torch.ops.sgl_kernel.layernorm_cpu(x, weight, bias, variance_epsilon)
        ref_ln_out = self._forward_native(
            x, weight, variance_epsilon, residual=None, bias=bias
        )
        torch.testing.assert_close(ln_out, ref_ln_out, atol=atol, rtol=rtol)

        residual = torch.randn([m, hidden_size], dtype=dtype)
        ref_residual = residual.clone()

        add_ln_out = torch.ops.sgl_kernel.fused_add_layernorm_cpu(
            x, residual, weight, None, variance_epsilon
        )
        ref_add_ln_out, ref_residual = self._forward_native(
            x, weight, variance_epsilon, residual=ref_residual
        )

        torch.testing.assert_close(add_ln_out, ref_add_ln_out, atol=atol, rtol=rtol)
        torch.testing.assert_close(residual, ref_residual, atol=atol, rtol=rtol)

        residual = torch.randn([m, hidden_size], dtype=dtype)
        ref_residual = residual.clone()

        add_ln_out = torch.ops.sgl_kernel.fused_add_layernorm_cpu(
            x, residual, weight, bias, variance_epsilon
        )
        ref_add_ln_out, ref_residual = self._forward_native(
            x, weight, variance_epsilon, residual=ref_residual, bias=bias
        )

        torch.testing.assert_close(add_ln_out, ref_add_ln_out, atol=atol, rtol=rtol)
        torch.testing.assert_close(residual, ref_residual, atol=atol, rtol=rtol)
```
**EN:** This test method exercises norm input 2d and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 norm input 2d 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 381-431: Run test: norm input 3d
```python
    @parametrize(
        l=[4096, 1024],
        m=[1, 4],
        n=[4096, 4109, 2304],
        dtype=[torch.float16, torch.bfloat16],
    )
    def test_norm_input_3d(self, l: int, m: int, n: int, dtype: torch.dtype) -> None:
        x = torch.randn([l, m, n], dtype=dtype)
        x = make_non_contiguous(x)
        hidden_size = x.size(-1)
        weight = torch.randn(hidden_size, dtype=dtype)
        bias = torch.randn(hidden_size, dtype=dtype)
        variance_epsilon = 1e-6

        ln_out = torch.ops.sgl_kernel.layernorm_cpu(x, weight, None, variance_epsilon)
        ref_ln_out = self._forward_native(x, weight, variance_epsilon)

        atol = rtol = precision[ref_ln_out.dtype]
        torch.testing.assert_close(ln_out, ref_ln_out, atol=atol, rtol=rtol)

        ln_out = torch.ops.sgl_kernel.layernorm_cpu(x, weight, bias, variance_epsilon)
        ref_ln_out = self._forward_native(
            x, weight, variance_epsilon, residual=None, bias=bias
        )
        torch.testing.assert_close(ln_out, ref_ln_out, atol=atol, rtol=rtol)

        residual = torch.randn([l, m, hidden_size], dtype=dtype)
        ref_residual = residual.clone()

        add_ln_out = torch.ops.sgl_kernel.fused_add_layernorm_cpu(
            x, residual, weight, None, variance_epsilon
        )
        ref_add_ln_out, ref_residual = self._forward_native(
            x, weight, variance_epsilon, ref_residual
        )

        torch.testing.assert_close(add_ln_out, ref_add_ln_out, atol=atol, rtol=rtol)
        torch.testing.assert_close(residual, ref_residual, atol=atol, rtol=rtol)

        residual = torch.randn([l, m, hidden_size], dtype=dtype)
        ref_residual = residual.clone()

        add_ln_out = torch.ops.sgl_kernel.fused_add_layernorm_cpu(
            x, residual, weight, bias, variance_epsilon
        )
        ref_add_ln_out, ref_residual = self._forward_native(
            x, weight, variance_epsilon, residual=ref_residual, bias=bias
        )

        torch.testing.assert_close(add_ln_out, ref_add_ln_out, atol=atol, rtol=rtol)
        torch.testing.assert_close(residual, ref_residual, atol=atol, rtol=rtol)
```
**EN:** This test method exercises norm input 3d and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 norm input 3d 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 434-435: Expose unittest entrypoint
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
- External and stdlib modules / 外部与标准库模块: `itertools`, `torch`, `typing`, `unittest`, `utils`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
