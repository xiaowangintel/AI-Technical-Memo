# test_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/cpu/test_gemm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates gemm behavior in SGLang's srt / cpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / CPU 领域中与 gemm 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: module imports and dependencies / 模块导入与依赖
```python
import itertools
import unittest

# TODO: use interface in cpu.py
import torch
import torch.nn as nn
from utils import (
    convert_weight,
    native_w8a8_per_token_matmul,
    per_token_quant_int8,
    precision,
    unpack_and_dequant_awq,
    unpack_and_dequant_gptq,
)

from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `itertools`, `unittest`, `torch`, `torch.nn`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `itertools`, `unittest`, `torch`, `torch.nn`。

### Lines 18-18: supporting statements / 辅助语句
```python
torch.manual_seed(1234)
```
**EN:** This block performs supporting work through calls such as `manual_seed`, preparing state for nearby definitions.
**CN:** 该代码块通过 `manual_seed` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 21-21: class Mod declaration / 类 Mod 声明
```python
class Mod(nn.Module):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `nn.Module`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `nn.Module`。

### Lines 22-24: method init / 方法 init
```python
    def __init__(self, input_channel, output_channel, has_bias):
        super(Mod, self).__init__()
        self.linear = torch.nn.Linear(input_channel, output_channel, has_bias)
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 26-27: method forward / 方法 forward
```python
    def forward(self, x):
        return self.linear(x)
```
**EN:** This block implements `forward` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `forward`，承担模块行为中的一个聚焦逻辑片段。

### Lines 30-30: class TestGemm declaration / 类 TestGemm 声明
```python
class TestGemm(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 31-50: class-level constants and configuration for `TestGemm` / 类级常量与配置
```python
    M = [1, 101]
    N = [16, 32 * 13]
    K = [32 * 16]
    has_bias = [False, True]

    M_int8 = [2, 128]
    N_int8 = [32 * 12]
    K_int8 = [32 * 17]

    M_fp8 = [1, 11]
    N_fp8 = [128, 224]
    K_fp8 = [512, 576]

    M_awq = [1, 32]
    N_awq = [4096]
    K_awq = [4096]

    M_gptq = [1, 32]
    N_gptq = [4096]
    K_gptq = [4096]
```
**EN:** This block defines shared names such as `M`, `N`, `K`, `has_bias`, `M_int8`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `M`, `N`, `K`, `has_bias`, `M_int8` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 52-75: method bf16 gemm / 方法 bf16 gemm
```python
    def _bf16_gemm(self, M, N, K, has_bias):

        mat1 = torch.randn(M, K, dtype=torch.bfloat16)
        mat2 = torch.randn(N, K, dtype=torch.bfloat16)

        ref = torch.matmul(mat1.float(), mat2.float().t())
        if has_bias:
            bias = torch.randn(N, dtype=torch.float32)
            ref.add_(bias.bfloat16())

        ref = ref.bfloat16()

        out = torch.ops.sgl_kernel.weight_packed_linear(
            mat1, mat2, bias if has_bias else None, False
        )

        packed_mat2 = torch.ops.sgl_kernel.convert_weight_packed(mat2)
        out2 = torch.ops.sgl_kernel.weight_packed_linear(
            mat1, packed_mat2, bias if has_bias else None, True
        )

        atol = rtol = precision[ref.dtype]
        torch.testing.assert_close(ref, out, atol=atol, rtol=rtol)
        torch.testing.assert_close(ref, out2, atol=atol, rtol=rtol)
```
**EN:** This block implements `_bf16_gemm` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_bf16_gemm`，承担模块行为中的一个聚焦逻辑片段。

### Lines 77-90: test case bf16 gemm / 测试用例 bf16 gemm
```python
    def test_bf16_gemm(self):
        for params in itertools.product(
            self.M,
            self.N,
            self.K,
            self.has_bias,
        ):
            with self.subTest(
                M=params[0],
                N=params[1],
                K=params[2],
                has_bias=params[3],
            ):
                self._bf16_gemm(*params)
```
**EN:** This test exercises `test_bf16_gemm` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bf16_gemm`。

### Lines 92-121: method bf16 gemm with small oc / 方法 bf16 gemm with small oc
```python
    def _bf16_gemm_with_small_oc(self, M, N, K, has_bias, use_post_sigmul):
        use_post_sigmul = use_post_sigmul and N == 1
        mat_mul = (
            None if not use_post_sigmul else torch.randn(M, 2 * K, dtype=torch.bfloat16)
        )
        mat1 = torch.randn(M, K, dtype=torch.bfloat16)
        mat2 = torch.randn(N, K, dtype=torch.bfloat16)

        ref = torch.nn.functional.linear(mat1, mat2)
        if has_bias:
            bias = torch.randn(N, dtype=torch.float32)
            ref.add_(bias)
        if use_post_sigmul:
            ref = torch.nn.functional.sigmoid(ref) * mat_mul
            out = torch.ops.sgl_kernel.fused_linear_sigmoid_mul(
                mat1,
                torch.ops.sgl_kernel.convert_weight_packed(mat2),
                bias if has_bias else None,
                True,
                mat_mul if use_post_sigmul else None,
            )
        else:
            out = torch.ops.sgl_kernel.weight_packed_linear(
                mat1,
                torch.ops.sgl_kernel.convert_weight_packed(mat2),
                bias if has_bias else None,
                True,
            )
        atol = rtol = precision[ref.dtype]
        torch.testing.assert_close(ref, out, atol=atol, rtol=rtol)
```
**EN:** This block implements `_bf16_gemm_with_small_oc` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_bf16_gemm_with_small_oc`，承担模块行为中的一个聚焦逻辑片段。

### Lines 123-134: test case bf16 gemm with small oc / 测试用例 bf16 gemm with small oc
```python
    def test_bf16_gemm_with_small_oc(self):
        for params in itertools.product(
            [1, 8, 32, 1024], [12, 1], self.K, self.has_bias, [False, True]
        ):
            with self.subTest(
                M=params[0],
                N=params[1],
                K=params[2],
                has_bias=params[3],
                use_post_sigmul=params[4],
            ):
                self._bf16_gemm_with_small_oc(*params)
```
**EN:** This test exercises `test_bf16_gemm_with_small_oc` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bf16_gemm_with_small_oc`。

### Lines 136-164: method int8 gemm / 方法 int8 gemm
```python
    def _int8_gemm(self, M, N, K, has_bias):
        dtype = torch.bfloat16
        A = torch.randn((M, K), dtype=dtype) / 10
        Aq, As = per_token_quant_int8(A)

        factor_for_scale = 1e-2
        int8_max = 127
        int8_min = -128

        B = (torch.rand((N, K), dtype=torch.float32) - 0.5) * 2
        Bq = (B * int8_max).clamp(min=int8_min, max=int8_max).to(torch.int8)
        Bs = torch.rand(N) * factor_for_scale

        bias = torch.randn(N) if has_bias else None
        ref_out = native_w8a8_per_token_matmul(Aq, Bq, As, Bs, bias, dtype)

        atol = rtol = precision[ref_out.dtype]

        Aq2, As2 = torch.ops.sgl_kernel.per_token_quant_int8_cpu(A)
        out = torch.ops.sgl_kernel.int8_scaled_mm_cpu(
            Aq2, Bq, As2, Bs, bias if has_bias else None, torch.bfloat16, False
        )
        torch.testing.assert_close(ref_out, out, atol=atol, rtol=rtol)

        # test the fused version
        fused_out = torch.ops.sgl_kernel.int8_scaled_mm_with_quant(
            A, Bq, Bs, bias if has_bias else None, torch.bfloat16, False
        )
        torch.testing.assert_close(ref_out, fused_out, atol=atol, rtol=rtol)
```
**EN:** This block implements `_int8_gemm` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_int8_gemm`，承担模块行为中的一个聚焦逻辑片段。

### Lines 166-179: test case int8 gemm / 测试用例 int8 gemm
```python
    def test_int8_gemm(self):
        for params in itertools.product(
            self.M_int8,
            self.N_int8,
            self.K_int8,
            self.has_bias,
        ):
            with self.subTest(
                M=params[0],
                N=params[1],
                K=params[2],
                has_bias=params[3],
            ):
                self._int8_gemm(*params)
```
**EN:** This test exercises `test_int8_gemm` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_int8_gemm`。

### Lines 181-223: method fp8 gemm / 方法 fp8 gemm
```python
    def _fp8_gemm(self, M, N, K, has_bias):
        prepack = True
        chunk = False
        scale_block_size_N = 64
        scale_block_size_K = 128
        assert scale_block_size_N <= N
        assert scale_block_size_K <= K
        A_dtype = torch.bfloat16

        model = Mod(K, N, has_bias).eval()
        if chunk:
            data = torch.randn(M, K + 6, dtype=A_dtype).narrow(1, 0, K)
        else:
            data = torch.randn(M, K, dtype=A_dtype)

        weight = model.linear.weight  # (N, K)

        if has_bias:
            bias = model.linear.bias

        fp8_weight, scales, dq_weight = convert_weight(
            weight, [scale_block_size_N, scale_block_size_K], A_dtype
        )

        if has_bias:
            ref = torch.matmul(data.to(A_dtype), dq_weight.T) + bias.to(A_dtype)
        else:
            ref = torch.matmul(data.to(A_dtype), dq_weight.T)

        if prepack:
            fp8_weight = torch.ops.sgl_kernel.convert_weight_packed(fp8_weight)

        opt = torch.ops.sgl_kernel.fp8_scaled_mm_cpu(
            data,
            fp8_weight,
            scales,
            [scale_block_size_N, scale_block_size_K],
            bias if has_bias else None,
            data.dtype,
            prepack,
        )
        atol = rtol = precision[ref.dtype]
        torch.testing.assert_close(ref, opt, atol=atol, rtol=rtol)
```
**EN:** This block implements `_fp8_gemm` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_fp8_gemm`，承担模块行为中的一个聚焦逻辑片段。

### Lines 225-238: test case fp8 gemm / 测试用例 fp8 gemm
```python
    def test_fp8_gemm(self):
        for params in itertools.product(
            self.M_fp8,
            self.N_fp8,
            self.K_fp8,
            self.has_bias,
        ):
            with self.subTest(
                M=params[0],
                N=params[1],
                K=params[2],
                has_bias=params[3],
            ):
                self._fp8_gemm(*params)
```
**EN:** This test exercises `test_fp8_gemm` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fp8_gemm`。

### Lines 240-270: method int4 awq gemm / 方法 int4 awq gemm
```python
    def _int4_awq_gemm(self, M, N, K, group_size, has_bias):
        awq_weight = torch.randint(-128, 128, (K, N // 8)).to(torch.int)
        awq_zero = torch.randint(0, 10, (K // group_size, N // 8)).to(torch.int)
        awq_scales = torch.rand(int(K // group_size), N).to(torch.bfloat16)
        bf16_weight, _ = unpack_and_dequant_awq(
            awq_weight, awq_zero, awq_scales, 4, 128
        )
        if has_bias:
            bias = torch.rand(bf16_weight.shape[0]).to(torch.float)
        else:
            bias = None
        x = torch.rand(M, bf16_weight.size(-1)).to(torch.bfloat16)
        ref_res = torch.nn.functional.linear(
            x, bf16_weight, bias=bias.to(torch.bfloat16) if has_bias else None
        )

        packed_weight, packed_zero, packed_scales = (
            torch.ops.sgl_kernel.convert_weight_packed_scale_zp(
                awq_weight, awq_zero, awq_scales, 0
            )
        )
        target_res = torch.ops.sgl_kernel.int4_scaled_mm_cpu(
            x,
            packed_weight,
            packed_zero,
            packed_scales,
            bias,
        )

        atol = rtol = precision[ref_res.dtype]
        torch.testing.assert_close(ref_res, target_res, atol=atol, rtol=rtol)
```
**EN:** This block implements `_int4_awq_gemm` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_int4_awq_gemm`，承担模块行为中的一个聚焦逻辑片段。

### Lines 272-283: test case int4 awq gemm / 测试用例 int4 awq gemm
```python
    def test_int4_awq_gemm(self):
        for params in itertools.product(
            self.M_awq, self.N_awq, self.K_awq, [128], self.has_bias
        ):
            with self.subTest(
                M=params[0],
                N=params[1],
                K=params[2],
                group_size=params[3],
                has_bias=params[4],
            ):
                self._int4_awq_gemm(*params)
```
**EN:** This test exercises `test_int4_awq_gemm` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_int4_awq_gemm`。

### Lines 285-315: method int4 gptq gemm / 方法 int4 gptq gemm
```python
    def _int4_gptq_gemm(self, M, N, K, group_size, has_bias):
        torch.manual_seed(127)
        gptq_weight = torch.randint(-128, 128, (K // 8, N)).to(torch.int)
        gptq_zero = torch.randint(0, 10, (K // group_size, N // 8)).to(torch.int)
        gptq_scales = torch.rand(int(K // group_size), N).to(torch.bfloat16) // 10

        bf16_weight = unpack_and_dequant_gptq(gptq_weight, gptq_zero, gptq_scales)
        if has_bias:
            bias = torch.rand(bf16_weight.shape[0]).to(torch.float)
        else:
            bias = None
        x = torch.rand(M, bf16_weight.size(-1)).to(torch.bfloat16)
        ref_res = torch.nn.functional.linear(
            x, bf16_weight, bias=bias.to(torch.bfloat16) if has_bias else None
        )

        packed_weight, packed_zero, packed_scales = (
            torch.ops.sgl_kernel.convert_weight_packed_scale_zp(
                gptq_weight, gptq_zero, gptq_scales, 1
            )
        )
        target_res = torch.ops.sgl_kernel.int4_scaled_mm_cpu(
            x,
            packed_weight,
            packed_zero,
            packed_scales,
            bias,
        )

        atol = rtol = precision[ref_res.dtype]
        torch.testing.assert_close(ref_res, target_res, atol=atol, rtol=rtol)
```
**EN:** This block implements `_int4_gptq_gemm` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_int4_gptq_gemm`，承担模块行为中的一个聚焦逻辑片段。

### Lines 317-328: test case int4 gptq gemm / 测试用例 int4 gptq gemm
```python
    def test_int4_gptq_gemm(self):
        for params in itertools.product(
            self.M_gptq, self.N_gptq, self.K_gptq, [128], self.has_bias
        ):
            with self.subTest(
                M=params[0],
                N=params[1],
                K=params[2],
                group_size=params[3],
                has_bias=params[4],
            ):
                self._int4_gptq_gemm(*params)
```
**EN:** This test exercises `test_int4_gptq_gemm` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_int4_gptq_gemm`。

### Lines 331-332: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `Mod`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestGemm`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `Mod.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。
- `Mod.forward`: This block implements `forward` and captures one focused piece of the module's behavior. / 该代码块实现 `forward`，承担模块行为中的一个聚焦逻辑片段。
- `TestGemm._bf16_gemm`: This block implements `_bf16_gemm` and captures one focused piece of the module's behavior. / 该代码块实现 `_bf16_gemm`，承担模块行为中的一个聚焦逻辑片段。
- `TestGemm.test_bf16_gemm`: This test exercises `test_bf16_gemm` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bf16_gemm`。
- `TestGemm._bf16_gemm_with_small_oc`: This block implements `_bf16_gemm_with_small_oc` and captures one focused piece of the module's behavior. / 该代码块实现 `_bf16_gemm_with_small_oc`，承担模块行为中的一个聚焦逻辑片段。
- `TestGemm.test_bf16_gemm_with_small_oc`: This test exercises `test_bf16_gemm_with_small_oc` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bf16_gemm_with_small_oc`。
- `TestGemm._int8_gemm`: This block implements `_int8_gemm` and captures one focused piece of the module's behavior. / 该代码块实现 `_int8_gemm`，承担模块行为中的一个聚焦逻辑片段。
- `TestGemm.test_int8_gemm`: This test exercises `test_int8_gemm` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_int8_gemm`。
- `TestGemm._fp8_gemm`: This block implements `_fp8_gemm` and captures one focused piece of the module's behavior. / 该代码块实现 `_fp8_gemm`，承担模块行为中的一个聚焦逻辑片段。
- `TestGemm.test_fp8_gemm`: This test exercises `test_fp8_gemm` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fp8_gemm`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `itertools`, `unittest`
- **Third-party modules / 第三方模块**: `torch`, `torch.nn`, `utils`
- **Internal modules / 内部模块**: `sglang.test.test_utils`

- **Total lines / 总行数**: 332
