# test_norm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/cpu/test_norm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates norm behavior in SGLang's srt / cpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / CPU 领域中与 norm 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: module imports and dependencies / 模块导入与依赖
```python
import itertools
import unittest
from typing import Optional, Tuple, Union

import torch
from utils import make_non_contiguous, parametrize, precision

from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `itertools`, `unittest`, `typing`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `itertools`, `unittest`, `typing`, `torch`。

### Lines 10-10: supporting statements / 辅助语句
```python
torch.manual_seed(1234)
```
**EN:** This block performs supporting work through calls such as `manual_seed`, preparing state for nearby definitions.
**CN:** 该代码块通过 `manual_seed` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 13-14: class TestNorm declaration / 类 TestNorm 声明
```python
class TestNorm(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 15-34: method forward native / 方法 forward native
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
**EN:** This block implements `_forward_native` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_forward_native`，承担模块行为中的一个聚焦逻辑片段。

### Lines 36-37: method norm / 方法 norm
```python
    def _norm(self, x, eps):
        return x * torch.rsqrt(x.pow(2).mean(-1, keepdim=True) + eps)
```
**EN:** This block implements `_norm` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_norm`，承担模块行为中的一个聚焦逻辑片段。

### Lines 39-44: method gemma3 rmsnorm native / 方法 gemma3 rmsnorm native
```python
    def _gemma3_rmsnorm_native(
        self, x: torch.Tensor, weight: torch.Tensor, variance_epsilon: float = 1e-6
    ):
        output = self._norm(x.float(), variance_epsilon)
        output = output * (1.0 + weight.float())
        return output.type_as(x)
```
**EN:** This block implements `_gemma3_rmsnorm_native` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_gemma3_rmsnorm_native`，承担模块行为中的一个聚焦逻辑片段。

### Lines 46-63: method gemma rmsnorm native / 方法 gemma rmsnorm native
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
**EN:** This block implements `_gemma_rmsnorm_native` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_gemma_rmsnorm_native`，承担模块行为中的一个聚焦逻辑片段。

### Lines 65-97: test case norm / 测试用例 norm
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
**EN:** This test exercises `test_norm` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_norm`。

### Lines 99-132: test case norm 3d / 测试用例 norm 3d
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
**EN:** This test exercises `test_norm_3d` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_norm_3d`。

### Lines 134-150: test case l2norm / 测试用例 l2norm
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
**EN:** This test exercises `test_l2norm` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_l2norm`。

### Lines 152-184: test case gemma rmsnorm / 测试用例 gemma rmsnorm
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
**EN:** This test exercises `test_gemma_rmsnorm` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gemma_rmsnorm`。

### Lines 186-205: test case gemma3 rmsnorm / 测试用例 gemma3 rmsnorm
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
**EN:** This test exercises `test_gemma3_rmsnorm` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gemma3_rmsnorm`。

### Lines 207-218: method gemma4 rmsnorm native / 方法 gemma4 rmsnorm native
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
**EN:** This block implements `_gemma4_rmsnorm_native` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_gemma4_rmsnorm_native`，承担模块行为中的一个聚焦逻辑片段。

### Lines 220-254: test case gemma4 rmsnorm / 测试用例 gemma4 rmsnorm
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
**EN:** This test exercises `test_gemma4_rmsnorm` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gemma4_rmsnorm`。

### Lines 257-257: class TestFusedRMSNormGated declaration / 类 TestFusedRMSNormGated 声明
```python
class TestFusedRMSNormGated(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 258-260: class-level constants and configuration for `TestFusedRMSNormGated` / 类级常量与配置
```python
    M = [4096, 1024]
    N = [4096, 4096 + 13]
    dtype = [torch.float16, torch.bfloat16]
```
**EN:** This block defines shared names such as `M`, `N`, `dtype`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `M`, `N`, `dtype` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 262-277: method forward native / 方法 forward native
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
**EN:** This block implements `_forward_native` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_forward_native`，承担模块行为中的一个聚焦逻辑片段。

### Lines 279-295: method norm test / 方法 norm test
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
**EN:** This block implements `_norm_test` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_norm_test`，承担模块行为中的一个聚焦逻辑片段。

### Lines 297-300: test case norm / 测试用例 norm
```python
    def test_norm(self):
        for params in itertools.product(self.M, self.N, self.dtype):
            with self.subTest(m=params[0], n=params[1], dtype=params[2]):
                self._norm_test(*params)
```
**EN:** This test exercises `test_norm` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_norm`。

### Lines 303-304: class TestLayerNorm declaration / 类 TestLayerNorm 声明
```python
class TestLayerNorm(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 305-325: method forward native / 方法 forward native
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
**EN:** This block implements `_forward_native` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_forward_native`，承担模块行为中的一个聚焦逻辑片段。

### Lines 327-376: test case norm input 2d / 测试用例 norm input 2d
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
**EN:** This test exercises `test_norm_input_2d` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_norm_input_2d`。

### Lines 378-428: test case norm input 3d / 测试用例 norm input 3d
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
**EN:** This test exercises `test_norm_input_3d` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_norm_input_3d`。

### Lines 431-432: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestNorm`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFusedRMSNormGated`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLayerNorm`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestNorm._forward_native`: This block implements `_forward_native` and captures one focused piece of the module's behavior. / 该代码块实现 `_forward_native`，承担模块行为中的一个聚焦逻辑片段。
- `TestNorm._norm`: This block implements `_norm` and captures one focused piece of the module's behavior. / 该代码块实现 `_norm`，承担模块行为中的一个聚焦逻辑片段。
- `TestNorm._gemma3_rmsnorm_native`: This block implements `_gemma3_rmsnorm_native` and captures one focused piece of the module's behavior. / 该代码块实现 `_gemma3_rmsnorm_native`，承担模块行为中的一个聚焦逻辑片段。
- `TestNorm._gemma_rmsnorm_native`: This block implements `_gemma_rmsnorm_native` and captures one focused piece of the module's behavior. / 该代码块实现 `_gemma_rmsnorm_native`，承担模块行为中的一个聚焦逻辑片段。
- `TestNorm.test_norm`: This test exercises `test_norm` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_norm`。
- `TestNorm.test_norm_3d`: This test exercises `test_norm_3d` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_norm_3d`。
- `TestNorm.test_l2norm`: This test exercises `test_l2norm` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_l2norm`。
- `TestNorm.test_gemma_rmsnorm`: This test exercises `test_gemma_rmsnorm` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gemma_rmsnorm`。
- `TestNorm.test_gemma3_rmsnorm`: This test exercises `test_gemma3_rmsnorm` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gemma3_rmsnorm`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `itertools`, `unittest`, `typing`
- **Third-party modules / 第三方模块**: `torch`, `utils`
- **Internal modules / 内部模块**: `sglang.test.test_utils`

- **Total lines / 总行数**: 432
