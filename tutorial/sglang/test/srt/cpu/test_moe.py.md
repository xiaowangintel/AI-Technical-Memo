# test_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/cpu/test_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates moe behavior in SGLang's srt / cpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / CPU 领域中与 moe 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: module imports and dependencies / 模块导入与依赖
```python
import itertools
import math
import unittest

# TODO: use interface in cpu.py
import torch

from sglang.srt.layers.amx_utils import CPUQuantMethod
```
**EN:** This block imports the modules needed by the rest of the file, including `itertools`, `math`, `unittest`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `itertools`, `math`, `unittest`, `torch`。

### Lines 10-12: module-level constants and configuration / 模块级常量与配置
```python
kernel = torch.ops.sgl_kernel

torch.manual_seed(128)
```
**EN:** This block defines shared names such as `kernel`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `kernel` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 14-28: module imports and dependencies / 模块导入与依赖
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

from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `utils`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `utils`, `sglang.test.test_utils`。

### Lines 31-61: function fused moe / 函数 fused moe
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
**EN:** This block implements `fused_moe` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `fused_moe`，承担模块行为中的一个聚焦逻辑片段。

### Lines 64-64: class TestFusedExperts declaration / 类 TestFusedExperts 声明
```python
class TestFusedExperts(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 65-88: class-level constants and configuration for `TestFusedExperts` / 类级常量与配置
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
**EN:** This block defines shared names such as `M`, `N`, `K`, `E`, `topk`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `M`, `N`, `K`, `E`, `topk` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 90-103: method bf16 moe / 方法 bf16 moe
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
**EN:** This block implements `_bf16_moe` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_bf16_moe`，承担模块行为中的一个聚焦逻辑片段。

### Lines 105-122: test case bf16 moe / 测试用例 bf16 moe
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
**EN:** This test exercises `test_bf16_moe` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bf16_moe`。

### Lines 124-180: method int8 moe / 方法 int8 moe
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
**EN:** This block implements `_int8_moe` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_int8_moe`，承担模块行为中的一个聚焦逻辑片段。

### Lines 182-197: test case int8 moe / 测试用例 int8 moe
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
**EN:** This test exercises `test_int8_moe` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_int8_moe`。

### Lines 199-249: method fp8 moe / 方法 fp8 moe
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
**EN:** This block implements `_fp8_moe` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_fp8_moe`，承担模块行为中的一个聚焦逻辑片段。

### Lines 251-266: test case fp8 moe / 测试用例 fp8 moe
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
**EN:** This test exercises `test_fp8_moe` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fp8_moe`。

### Lines 268-331: method int4 moe / 方法 int4 moe
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

        atol = rtol = precision[dtype]
        torch.testing.assert_close(ref_out.bfloat16(), out, atol=atol, rtol=rtol)
```
**EN:** This block implements `_int4_moe` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_int4_moe`，承担模块行为中的一个聚焦逻辑片段。

### Lines 333-348: test case int4 moe / 测试用例 int4 moe
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
**EN:** This test exercises `test_int4_moe` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_int4_moe`。

### Lines 351-352: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `fused_moe`: This block implements `fused_moe` and captures one focused piece of the module's behavior. / 该代码块实现 `fused_moe`，承担模块行为中的一个聚焦逻辑片段。
- `TestFusedExperts`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFusedExperts._bf16_moe`: This block implements `_bf16_moe` and captures one focused piece of the module's behavior. / 该代码块实现 `_bf16_moe`，承担模块行为中的一个聚焦逻辑片段。
- `TestFusedExperts.test_bf16_moe`: This test exercises `test_bf16_moe` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bf16_moe`。
- `TestFusedExperts._int8_moe`: This block implements `_int8_moe` and captures one focused piece of the module's behavior. / 该代码块实现 `_int8_moe`，承担模块行为中的一个聚焦逻辑片段。
- `TestFusedExperts.test_int8_moe`: This test exercises `test_int8_moe` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_int8_moe`。
- `TestFusedExperts._fp8_moe`: This block implements `_fp8_moe` and captures one focused piece of the module's behavior. / 该代码块实现 `_fp8_moe`，承担模块行为中的一个聚焦逻辑片段。
- `TestFusedExperts.test_fp8_moe`: This test exercises `test_fp8_moe` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fp8_moe`。
- `TestFusedExperts._int4_moe`: This block implements `_int4_moe` and captures one focused piece of the module's behavior. / 该代码块实现 `_int4_moe`，承担模块行为中的一个聚焦逻辑片段。
- `TestFusedExperts.test_int4_moe`: This test exercises `test_int4_moe` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_int4_moe`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `itertools`, `math`, `unittest`
- **Third-party modules / 第三方模块**: `torch`, `utils`
- **Internal modules / 内部模块**: `sglang.srt.layers.amx_utils`, `sglang.test.test_utils`

- **Total lines / 总行数**: 352
