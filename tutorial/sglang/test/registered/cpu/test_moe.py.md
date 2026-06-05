# test_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/cpu/test_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on cpu moe in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 cpu moe 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: Import dependencies
```python
import itertools
import math
import unittest
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 6-8: Import dependencies
```python
import torch

from sglang.srt.layers.amx_utils import CPUQuantMethod
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 10-10: Define module constants
```python
kernel = torch.ops.sgl_kernel
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 12-12: Implement expr logic
```python
torch.manual_seed(128)
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 14-29: Import dependencies
```python
from utils import (
    BLOCK_K,
    BLOCK_N,
    factor_for_scale,
    fp8_max,
    fp8_min,
    native_fp8_fused_moe,
    precision,
    scaled_weight,
    torch_naive_fused_moe,
    torch_w8a8_per_column_fused_moe,
    unpack_and_dequant_awq,
)

from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 31-31: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-b-test-cpu")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 34-64: Define helper: fused moe
```python
def fused_moe(a, w1, w2, score, topk, renormalize, prepack):

    G = 1
    topk_group = 1

    B, D = a.shape
    topk_weights = torch.empty(B, topk, dtype=torch.float32)
    topk_ids = torch.empty(B, topk, dtype=torch.int32)
    topk_weights, topk_ids = kernel.grouped_topk_cpu(
        a, score, topk, renormalize, G, topk_group, 0, None, None
    )

    packed_w1 = kernel.convert_weight_packed(w1) if prepack else w1
    packed_w2 = kernel.convert_weight_packed(w2) if prepack else w2

    inplace = True
    return kernel.fused_experts_cpu(
        a,
        packed_w1,
        packed_w2,
        topk_weights,
        topk_ids,
        inplace,
        CPUQuantMethod.UNQUANT,
        None,
        None,
        None,
        None,
        None,
        prepack,
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 67-67: Define class TestFusedExperts
```python
class TestFusedExperts(CustomTestCase):
```
**EN:** This declaration introduces the `TestFusedExperts` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFusedExperts` 测试类，并说明它通过继承承担的职责。

### Lines 68-91: Declare TestFusedExperts configuration
```python
    M = [2, 114]
    N = [32]
    K = [32]
    E = [4]
    topk = [2]
    renormalize = [False, True]

    M_int8 = [1, 39]
    N_int8 = [128]
    K_int8 = [256]
    E_int8 = [8]
    topk_int8 = [3]

    M_fp8 = [2, 121]
    N_fp8 = [352, 512]
    K_fp8 = [256, 320]
    E_fp8 = [8]
    topk_fp8 = [4]

    M_int4 = [1, 6]
    N_int4 = [512]
    K_int4 = [256]
    E_int4 = [8]
    topk_int4 = [4]
```
**EN:** This block defines class-level settings that are shared across the `TestFusedExperts` test methods.
**CN:** 该代码块定义了 `TestFusedExperts` 各测试方法共享的类级配置。

### Lines 93-106: Define helper: bf16 moe
```python
    def _bf16_moe(self, m, n, k, e, topk, renormalize):
        dtype = torch.bfloat16
        prepack = True

        a = torch.randn((m, k), device="cpu", dtype=dtype) / 10
        w1 = torch.randn((e, 2 * n, k), device="cpu", dtype=dtype) / 10
        w2 = torch.randn((e, k, n), device="cpu", dtype=dtype) / 10
        score = torch.randn((m, e), device="cpu", dtype=dtype)

        torch_output = torch_naive_fused_moe(a, w1, w2, score, topk, renormalize)
        fused_output = fused_moe(a, w1, w2, score, topk, renormalize, prepack)

        atol = rtol = precision[torch_output.dtype]
        torch.testing.assert_close(torch_output, fused_output, atol=atol, rtol=rtol)
```
**EN:** This helper function encapsulates reusable logic inside `TestFusedExperts` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestFusedExperts` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 108-125: Run test: bf16 moe
```python
    def test_bf16_moe(self):
        for params in itertools.product(
            self.M,
            self.N,
            self.K,
            self.E,
            self.topk,
            self.renormalize,
        ):
            with self.subTest(
                m=params[0],
                n=params[1],
                k=params[2],
                e=params[3],
                topk=params[4],
                renormalize=params[5],
            ):
                self._bf16_moe(*params)
```
**EN:** This test method exercises bf16 moe and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 bf16 moe 场景，并验证观测到的行为是否符合预期契约。

### Lines 127-183: Define helper: int8 moe
```python
    def _int8_moe(self, M, N, K, E, topk):
        dtype = torch.bfloat16
        prepack = True

        # Initialize int8 quantization parameters
        int8_factor_for_scale = 1e-2
        int8_max = 127
        int8_min = -128

        # Input tensor
        # M * K
        a = torch.randn((M, K), dtype=dtype) / math.sqrt(K)

        # Generate int8 weights
        w1_fp32 = (torch.rand((E, 2 * N, K), dtype=torch.float32) - 0.5) * 2
        w1 = (w1_fp32 * int8_max).clamp(min=int8_min, max=int8_max).to(torch.int8)

        w2_fp32 = (torch.rand((E, K, N), dtype=torch.float32) - 0.5) * 2
        w2 = (w2_fp32 * int8_max).clamp(min=int8_min, max=int8_max).to(torch.int8)

        # Generate scale for each column (per-column quantization)
        w1_s = torch.rand(E, 2 * N, device=w1_fp32.device) * int8_factor_for_scale
        w2_s = torch.rand(E, K, device=w2_fp32.device) * int8_factor_for_scale

        # Calculate routing
        score = torch.randn((M, E), dtype=dtype)
        score = torch.softmax(score, dim=-1, dtype=torch.float32)
        topk_weight, topk_ids = torch.topk(score, topk)

        ref_out = torch_w8a8_per_column_fused_moe(
            a, w1, w2, w1_s, w2_s, topk_weight, topk_ids, topk
        )

        inplace = True
        packed_w1 = kernel.convert_weight_packed(w1) if prepack else w1
        packed_w2 = kernel.convert_weight_packed(w2) if prepack else w2
        out = kernel.fused_experts_cpu(
            a,
            packed_w1,
            packed_w2,
            topk_weight,
            topk_ids.to(torch.int32),
            inplace,
            CPUQuantMethod.INT8_W8A8,
            w1_s,
            w2_s,
            None,
            None,
            None,
            prepack,
        )

        atol = rtol = precision[ref_out.dtype]
        # Increase the tolerance for large input shapes
        if M > 35:
            atol = rtol = 0.02
        torch.testing.assert_close(ref_out, out, atol=atol, rtol=rtol)
```
**EN:** This helper function encapsulates reusable logic inside `TestFusedExperts` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestFusedExperts` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 185-200: Run test: int8 moe
```python
    def test_int8_moe(self):
        for params in itertools.product(
            self.M_int8,
            self.N_int8,
            self.K_int8,
            self.E_int8,
            self.topk_int8,
        ):
            with self.subTest(
                M=params[0],
                N=params[1],
                K=params[2],
                E=params[3],
                topk=params[4],
            ):
                self._int8_moe(*params)
```
**EN:** This test method exercises int8 moe and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 int8 moe 场景，并验证观测到的行为是否符合预期契约。

### Lines 202-252: Define helper: fp8 moe
```python
    def _fp8_moe(self, M, N, K, E, topk):
        dtype = torch.bfloat16

        a = torch.randn(M, K, dtype=dtype) / math.sqrt(K)

        w1_fp32 = torch.randn(E, 2 * N, K)
        w1 = (w1_fp32 * fp8_max).clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn)

        w2_fp32 = torch.randn(E, K, N)
        w2 = (w2_fp32 * fp8_max).clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn)

        w1s = (
            torch.randn(E, math.ceil(2 * N / BLOCK_N), math.ceil(K / BLOCK_K))
            * factor_for_scale
        )
        w2s = (
            torch.randn(E, math.ceil(K / BLOCK_N), math.ceil(N / BLOCK_K))
            * factor_for_scale
        )

        w1_scaled = scaled_weight(w1, w1s)
        w2_scaled = scaled_weight(w2, w2s)

        score = torch.randn((M, E), dtype=dtype)
        score = torch.softmax(score, dim=-1, dtype=torch.float32)
        topk_weight, topk_ids = torch.topk(score, topk)

        w1 = kernel.convert_weight_packed(w1)
        w2 = kernel.convert_weight_packed(w2)

        ref_out = native_fp8_fused_moe(
            a, w1_scaled, w2_scaled, topk_weight, topk_ids, topk
        )
        out = kernel.fused_experts_cpu(
            a,
            w1,
            w2,
            topk_weight,
            topk_ids.to(torch.int32),
            False,
            CPUQuantMethod.FP8_W8A16,
            w1s,
            w2s,
            None,
            None,
            [BLOCK_N, BLOCK_K],
            True,
        )

        atol = rtol = precision[dtype]
        torch.testing.assert_close(ref_out.bfloat16(), out, atol=atol, rtol=rtol)
```
**EN:** This helper function encapsulates reusable logic inside `TestFusedExperts` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestFusedExperts` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 254-269: Run test: fp8 moe
```python
    def test_fp8_moe(self):
        for params in itertools.product(
            self.M_fp8,
            self.N_fp8,
            self.K_fp8,
            self.E_fp8,
            self.topk_fp8,
        ):
            with self.subTest(
                M=params[0],
                N=params[1],
                K=params[2],
                E=params[3],
                topk=params[4],
            ):
                self._fp8_moe(*params)
```
**EN:** This test method exercises fp8 moe and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 fp8 moe 场景，并验证观测到的行为是否符合预期契约。

### Lines 271-315: Define helper: int4 moe (part 1)
```python
    def _int4_moe(self, M, N, K, E, topk, group_size=128):
        dtype = torch.bfloat16

        a = torch.rand(M, K, dtype=dtype) / math.sqrt(K)

        awq_w13_weight = torch.randint(-127, 128, (E, K, 2 * N // 8)).to(torch.int)
        awq_w13_zero = torch.randint(0, 10, (E, K // group_size, 2 * N // 8)).to(
            torch.int
        )
        awq_w13_scales = torch.rand(E, int(K // group_size), 2 * N).to(torch.bfloat16)

        awq_w2_weight = torch.randint(-127, 128, (E, N, K // 8)).to(torch.int)
        awq_w2_zero = torch.randint(0, 10, (E, N // group_size, K // 8)).to(torch.int)
        awq_w2_scales = torch.rand(E, int(N // group_size), K).to(torch.bfloat16)
        bf16_w13_weight = []
        bf16_w2_weight = []
        for i in range(E):
            bf16_w13_weight_i, _ = unpack_and_dequant_awq(
                awq_w13_weight[i], awq_w13_zero[i], awq_w13_scales[i], 4, 128
            )
            bf16_w2_weight_i, _ = unpack_and_dequant_awq(
                awq_w2_weight[i], awq_w2_zero[i], awq_w2_scales[i], 4, 128
            )
            bf16_w13_weight.append(bf16_w13_weight_i)
            bf16_w2_weight.append(bf16_w2_weight_i)
        bf16_w13_weight = torch.stack(bf16_w13_weight).detach()
        bf16_w2_weight = torch.stack(bf16_w2_weight).detach()

        score = torch.rand((M, E), dtype=dtype)

        ref_out = torch_naive_fused_moe(
            a, bf16_w13_weight, bf16_w2_weight, score, topk, False
        )
        score = torch.softmax(score, dim=-1, dtype=torch.float32)
        topk_weight, topk_ids = torch.topk(score, topk)
        awq_w13_weight_pack, awq_w13_zero_pack, awq_w13_scales_pack = (
            torch.ops.sgl_kernel.convert_weight_packed_scale_zp(
                awq_w13_weight, awq_w13_zero, awq_w13_scales, 0
            )
        )
        awq_w2_weight_pack, awq_w2_zero_pack, awq_w2_scales_pack = (
            torch.ops.sgl_kernel.convert_weight_packed_scale_zp(
                awq_w2_weight, awq_w2_zero, awq_w2_scales, 0
            )
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestFusedExperts` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestFusedExperts` 内部调用，从而让场景结构更清晰。

### Lines 317-331: Define helper: int4 moe (part 2)
```python
        out = kernel.fused_experts_cpu(
            a,
            awq_w13_weight_pack,
            awq_w2_weight_pack,
            topk_weight,
            topk_ids.to(torch.int32),
            False,
            CPUQuantMethod.INT4_W4A8,
            awq_w13_scales_pack,
            awq_w2_scales_pack,
            awq_w13_zero_pack,
            awq_w2_zero_pack,
            None,
            True,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestFusedExperts` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestFusedExperts` 内部调用，从而让场景结构更清晰。

### Lines 333-334: Define helper: int4 moe (part 3)
```python
        atol = rtol = precision[dtype]
        torch.testing.assert_close(ref_out.bfloat16(), out, atol=atol, rtol=rtol)
```
**EN:** This helper function encapsulates reusable logic inside `TestFusedExperts` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestFusedExperts` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 336-351: Run test: int4 moe
```python
    def test_int4_moe(self):
        for params in itertools.product(
            self.M_int4,
            self.N_int4,
            self.K_int4,
            self.E_int4,
            self.topk_int4,
        ):
            with self.subTest(
                M=params[0],
                N=params[1],
                K=params[2],
                E=params[3],
                topk=params[4],
            ):
                self._int4_moe(*params)
```
**EN:** This test method exercises int4 moe and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 int4 moe 场景，并验证观测到的行为是否符合预期契约。

### Lines 354-355: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.layers.amx_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `itertools`, `math`, `torch`, `unittest`, `utils`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
