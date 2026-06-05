# test_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/cpu/test_gemm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on cpu gemm in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 cpu gemm 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: Import dependencies
```python
import itertools
import unittest
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 5-17: Import dependencies
```python
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

from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 19-19: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-b-test-cpu")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 21-21: Implement expr logic
```python
torch.manual_seed(1234)
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 24-24: Define class Mod
```python
class Mod(nn.Module):
```
**EN:** This declaration introduces the `Mod` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `Mod` 测试类，并说明它通过继承承担的职责。

### Lines 25-27: Define helper: init
```python
    def __init__(self, input_channel, output_channel, has_bias):
        super(Mod, self).__init__()
        self.linear = torch.nn.Linear(input_channel, output_channel, has_bias)
```
**EN:** This helper function encapsulates reusable logic inside `Mod` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `Mod` 内部调用，从而让场景结构更清晰。

### Lines 29-30: Define helper: forward
```python
    def forward(self, x):
        return self.linear(x)
```
**EN:** This helper function encapsulates reusable logic inside `Mod` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `Mod` 内部调用，从而让场景结构更清晰。

### Lines 33-33: Define class TestGemm
```python
class TestGemm(CustomTestCase):
```
**EN:** This declaration introduces the `TestGemm` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestGemm` 测试类，并说明它通过继承承担的职责。

### Lines 34-53: Declare TestGemm configuration
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
**EN:** This block defines class-level settings that are shared across the `TestGemm` test methods.
**CN:** 该代码块定义了 `TestGemm` 各测试方法共享的类级配置。

### Lines 55-78: Define helper: bf16 gemm
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
**EN:** This helper function encapsulates reusable logic inside `TestGemm` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGemm` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 80-93: Run test: bf16 gemm
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
**EN:** This test method exercises bf16 gemm and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 bf16 gemm 场景，并验证观测到的行为是否符合预期契约。

### Lines 95-124: Define helper: bf16 gemm with small oc
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
**EN:** This helper function encapsulates reusable logic inside `TestGemm` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGemm` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 126-137: Run test: bf16 gemm with small oc
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
**EN:** This test method exercises bf16 gemm with small oc and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 bf16 gemm with small oc 场景，并验证观测到的行为是否符合预期契约。

### Lines 139-167: Define helper: int8 gemm
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
**EN:** This helper function encapsulates reusable logic inside `TestGemm` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGemm` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 169-182: Run test: int8 gemm
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
**EN:** This test method exercises int8 gemm and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 int8 gemm 场景，并验证观测到的行为是否符合预期契约。

### Lines 184-226: Define helper: fp8 gemm
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
**EN:** This helper function encapsulates reusable logic inside `TestGemm` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGemm` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 228-241: Run test: fp8 gemm
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
**EN:** This test method exercises fp8 gemm and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 fp8 gemm 场景，并验证观测到的行为是否符合预期契约。

### Lines 243-273: Define helper: int4 awq gemm
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
**EN:** This helper function encapsulates reusable logic inside `TestGemm` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGemm` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 275-286: Run test: int4 awq gemm
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
**EN:** This test method exercises int4 awq gemm and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 int4 awq gemm 场景，并验证观测到的行为是否符合预期契约。

### Lines 288-318: Define helper: int4 gptq gemm
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
**EN:** This helper function encapsulates reusable logic inside `TestGemm` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGemm` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 320-331: Run test: int4 gptq gemm
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
**EN:** This test method exercises int4 gptq gemm and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 int4 gptq gemm 场景，并验证观测到的行为是否符合预期契约。

### Lines 334-335: Expose unittest entrypoint
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
- External and stdlib modules / 外部与标准库模块: `itertools`, `torch`, `torch.nn`, `unittest`, `utils`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
