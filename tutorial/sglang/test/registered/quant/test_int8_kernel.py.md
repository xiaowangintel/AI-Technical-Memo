# test_int8_kernel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/quant/test_int8_kernel.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates int8 kernel behavior in SGLang's quant area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 quant 领域中与 int8 kernel 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: module imports and dependencies / 模块导入与依赖
```python
import itertools
import unittest

import torch

from sglang.srt.layers.activation import SiluAndMul
from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe import fused_moe
from sglang.srt.layers.moe.topk import TopKConfig, select_experts
from sglang.srt.layers.quantization.int8_kernel import per_token_quant_int8
from sglang.srt.server_args import ServerArgs, set_global_server_args_for_scheduler
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `itertools`, `unittest`, `torch`, `sglang.srt.layers.activation`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `itertools`, `unittest`, `torch`, `sglang.srt.layers.activation`。

### Lines 14-14: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=15, stage="base-b", runner_config="1-gpu-small")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 17-36: function native w8a8 per token matmul / 函数 native w8a8 per token matmul
```python
def native_w8a8_per_token_matmul(A, B, As, Bs, output_dtype=torch.float16):
    """Matrix multiplication function that supports per-token input quantization and per-column weight quantization"""
    A = A.to(torch.float32)
    B = B.to(torch.float32)

    assert A.shape[-1] == B.shape[-1], "Dimension mismatch"
    assert B.ndim == 2 and B.is_contiguous(), "B must be a 2D contiguous tensor"

    # Reshape input
    M = A.numel() // A.shape[-1]
    B = B.t()  # Transpose weight matrix
    N, K = B.shape
    origin_C_shape = A.shape[:-1] + (K,)
    A = A.reshape(M, N)

    # As is per-token [M, 1], Bs is per-column [1, K]
    C = torch.matmul(A, B)  # [M, K]
    C = As * C * Bs.view(1, -1)  # Broadcast per-column scale

    return C.reshape(origin_C_shape).to(output_dtype)
```
**EN:** Matrix multiplication function that supports per-token input quantization and per-column weight quantization This block implements `native_w8a8_per_token_matmul` and captures one focused piece of the module's behavior.
**CN:** Matrix multiplication function that supports per-token input quantization and per-column weight quantization 该代码块实现 `native_w8a8_per_token_matmul`，承担模块行为中的一个聚焦逻辑片段。

### Lines 39-79: function torch w8a8 per column moe / 函数 torch w8a8 per column moe
```python
def torch_w8a8_per_column_moe(a, w1, w2, w1_s, w2_s, score, topk):
    """This function performs fused moe with per-column int8 quantization using native torch."""

    set_global_server_args_for_scheduler(ServerArgs(model_path="dummy"))

    B, D = a.shape
    # Perform per-token quantization
    a_q, a_s = per_token_quant_int8(a)
    # Repeat tokens to match topk
    a_q = a_q.view(B, -1, D).repeat(1, topk, 1).reshape(-1, D)
    # Also repeat the scale
    a_s = a_s.view(B, -1, 1).repeat(1, topk, 1).reshape(-1, 1)  # [B*topk, 1]

    out = torch.zeros(B * topk, w2.shape[1], dtype=a.dtype, device=a.device)

    # Calculate routing
    score = torch.softmax(score, dim=-1, dtype=torch.float32)
    topk_weight, topk_ids = torch.topk(score, topk)
    topk_weight = topk_weight.view(-1)
    topk_ids = topk_ids.view(-1)
    # Process each expert
    for i in range(w1.shape[0]):
        mask = topk_ids == i
        if mask.sum():
            # First MLP layer: note that a_s is now per-token
            inter_out = native_w8a8_per_token_matmul(
                a_q[mask], w1[i], a_s[mask], w1_s[i], output_dtype=a.dtype
            )
            # Activation function
            act_out = SiluAndMul().forward_native(inter_out)
            # Quantize activation output with per-token
            act_out_q, act_out_s = per_token_quant_int8(act_out)

            # Second MLP layer
            out[mask] = native_w8a8_per_token_matmul(
                act_out_q, w2[i], act_out_s, w2_s[i], output_dtype=a.dtype
            )
    # Apply routing weights and sum
    return (
        out.view(B, -1, w2.shape[1]) * topk_weight.view(B, -1, 1).to(out.dtype)
    ).sum(dim=1)
```
**EN:** This function performs fused moe with per-column int8 quantization using native torch. This block implements `torch_w8a8_per_column_moe` and captures one focused piece of the module's behavior.
**CN:** This function performs fused moe with per-column int8 quantization using native torch. 该代码块实现 `torch_w8a8_per_column_moe`，承担模块行为中的一个聚焦逻辑片段。

### Lines 82-82: class TestW8A8Int8FusedMoE declaration / 类 TestW8A8Int8FusedMoE 声明
```python
class TestW8A8Int8FusedMoE(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 83-91: class-level constants and configuration for `TestW8A8Int8FusedMoE` / 类级常量与配置
```python
    DTYPES = [torch.half, torch.bfloat16]
    M = [1, 33]
    N = [128, 1024]
    K = [256, 4096]
    E = [8]
    TOP_KS = [2, 6]
    BLOCK_SIZE = [[64, 64], [64, 128], [128, 64], [128, 128]]
    BLOCK_SIZE = [[128, 128]]
    SEEDS = [0]
```
**EN:** This block defines shared names such as `DTYPES`, `M`, `N`, `K`, `E`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `DTYPES`, `M`, `N`, `K`, `E` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 93-97: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA is not available")
        torch.set_default_device("cuda")
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 99-148: method w8a8 int8 fused moe / 方法 w8a8 int8 fused moe
```python
    def _w8a8_int8_fused_moe(self, M, N, K, E, topk, block_size, dtype, seed):
        torch.manual_seed(seed)
        # Initialize int8 quantization parameters
        factor_for_scale = 1e-2
        int8_max = 127
        int8_min = -128

        # Input tensor
        # M * K
        a = torch.randn((M, K), dtype=dtype) / 10

        # Generate int8 weights
        w1_fp32 = (torch.rand((E, 2 * N, K), dtype=torch.float32) - 0.5) * 2
        w1 = (w1_fp32 * int8_max).clamp(min=int8_min, max=int8_max).to(torch.int8)

        w2_fp32 = (torch.rand((E, K, N), dtype=torch.float32) - 0.5) * 2
        w2 = (w2_fp32 * int8_max).clamp(min=int8_min, max=int8_max).to(torch.int8)

        # Generate scale for each column (per-column quantization)
        w1_s = torch.rand(E, 2 * N, device=w1_fp32.device) * factor_for_scale
        w2_s = torch.rand(E, K, device=w2_fp32.device) * factor_for_scale
        score = torch.randn((M, E), dtype=dtype)

        with torch.inference_mode():
            ref_out = torch_w8a8_per_column_moe(a, w1, w2, w1_s, w2_s, score, topk)
            topk_output = select_experts(
                hidden_states=a,
                router_logits=score,
                topk_config=TopKConfig(top_k=topk, renormalize=False),
            )
            out = fused_moe(
                a,
                w1,
                w2,
                topk_output,
                use_fp8_w8a8=False,  # Not using fp8
                use_int8_w8a16=False,  # Not using int8-w8a16
                use_int8_w8a8=True,  # Using int8-w8a8
                per_channel_quant=True,
                w1_scale=w1_s,
                w2_scale=w2_s,
                block_shape=None,  # Not using block quantization
            )

        # Check results
        self.assertTrue(
            torch.mean(torch.abs(out.to(torch.float32) - ref_out.to(torch.float32)))
            / torch.mean(torch.abs(ref_out.to(torch.float32)))
            < 0.05
        )
```
**EN:** This block implements `_w8a8_int8_fused_moe` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_w8a8_int8_fused_moe`，承担模块行为中的一个聚焦逻辑片段。

### Lines 150-171: test case w8a8 int8 fused moe / 测试用例 w8a8 int8 fused moe
```python
    def test_w8a8_int8_fused_moe(self):
        for params in itertools.product(
            self.M,
            self.N,
            self.K,
            self.E,
            self.TOP_KS,
            self.BLOCK_SIZE,
            self.DTYPES,
            self.SEEDS,
        ):
            with self.subTest(
                M=params[0],
                N=params[1],
                K=params[2],
                E=params[3],
                topk=params[4],
                block_size=params[5],
                dtype=params[6],
                seed=params[7],
            ):
                self._w8a8_int8_fused_moe(*params)
```
**EN:** This test exercises `test_w8a8_int8_fused_moe` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_w8a8_int8_fused_moe`。

### Lines 174-175: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `native_w8a8_per_token_matmul`: Matrix multiplication function that supports per-token input quantization and per-column weight quantization / 该代码块实现 `native_w8a8_per_token_matmul`，承担模块行为中的一个聚焦逻辑片段。
- `torch_w8a8_per_column_moe`: This function performs fused moe with per-column int8 quantization using native torch. / 该代码块实现 `torch_w8a8_per_column_moe`，承担模块行为中的一个聚焦逻辑片段。
- `TestW8A8Int8FusedMoE`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestW8A8Int8FusedMoE.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestW8A8Int8FusedMoE._w8a8_int8_fused_moe`: This block implements `_w8a8_int8_fused_moe` and captures one focused piece of the module's behavior. / 该代码块实现 `_w8a8_int8_fused_moe`，承担模块行为中的一个聚焦逻辑片段。
- `TestW8A8Int8FusedMoE.test_w8a8_int8_fused_moe`: This test exercises `test_w8a8_int8_fused_moe` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_w8a8_int8_fused_moe`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `itertools`, `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.layers.activation`, `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe`, `sglang.srt.layers.moe.topk`, `sglang.srt.layers.quantization.int8_kernel`, `sglang.srt.server_args`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 175
