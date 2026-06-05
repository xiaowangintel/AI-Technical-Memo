# test_block_int8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/quant/test_block_int8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates block int8 behavior in SGLang's quant area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 quant 领域中与 block int8 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: module imports and dependencies / 模块导入与依赖
```python
import itertools
import unittest

import torch

from sglang.srt.layers.activation import SiluAndMul
from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe import fused_moe
from sglang.srt.layers.moe.topk import TopKConfig, select_experts
from sglang.srt.server_args import ServerArgs, set_global_server_args_for_scheduler
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `itertools`, `unittest`, `torch`, `sglang.srt.layers.activation`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `itertools`, `unittest`, `torch`, `sglang.srt.layers.activation`。

### Lines 13-14: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=44, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=22, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 15-17: supporting source context / 辅助源码上下文
```python


# For test
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 18-41: function native per token group quant int8 / 函数 native per token group quant int8
```python
def native_per_token_group_quant_int8(x, group_size, eps=1e-10, dtype=torch.int8):
    """Function to perform per-token-group quantization on an input tensor `x` using native torch.

    It converts the tensor values into float8 values and returns the
    quantized tensor along with the scaling factor used for quantization.
    Note that only `torch.float8_e4m3fn` is supported for now.
    """
    assert (
        x.shape[-1] % group_size == 0
    ), "the last dimension of `x` cannot be divisible by `group_size`"
    assert x.is_contiguous(), "`x` is not contiguous"

    iinfo = torch.iinfo(dtype)
    int8_min = iinfo.min
    int8_max = iinfo.max

    x_ = x.reshape(x.numel() // group_size, group_size)
    amax = x_.abs().max(dim=-1, keepdim=True)[0].clamp(min=eps).to(torch.float32)
    x_s = amax / int8_max
    x_q = (x_ / x_s).clamp(min=int8_min, max=int8_max).to(dtype)
    x_q = x_q.reshape(x.shape)
    x_s = x_s.reshape(x.shape[:-1] + (x.shape[-1] // group_size,))

    return x_q, x_s
```
**EN:** Function to perform per-token-group quantization on an input tensor `x` using native torch. This block implements `native_per_token_group_quant_int8` and captures one focused piece of the module's behavior.
**CN:** Function to perform per-token-group quantization on an input tensor `x` using native torch. 该代码块实现 `native_per_token_group_quant_int8`，承担模块行为中的一个聚焦逻辑片段。

### Lines 42-44: supporting source context / 辅助源码上下文
```python


# For test
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 45-97: function native w8a8 block int8 matmul / 函数 native w8a8 block int8 matmul
```python
def native_w8a8_block_int8_matmul(A, B, As, Bs, block_size, output_dtype=torch.float16):
    """This function performs matrix multiplication with block-wise quantization using native torch.

    It takes two input tensors `A` and `B` with scales `As` and `Bs`.
    The output is returned in the specified `output_dtype`.
    """

    A = A.to(torch.float32)
    B = B.to(torch.float32)
    assert A.shape[-1] == B.shape[-1]
    assert B.ndim == 2 and B.is_contiguous() and Bs.ndim == 2
    assert len(block_size) == 2
    block_n, block_k = block_size[0], block_size[1]
    assert (A.shape[-1] + block_k - 1) // block_k == As.shape[-1]
    assert A.shape[:-1] == As.shape[:-1]

    M = A.numel() // A.shape[-1]
    N, K = B.shape
    origin_C_shape = A.shape[:-1] + (N,)
    A = A.reshape(M, A.shape[-1])
    As = As.reshape(M, As.shape[-1])
    n_tiles = (N + block_n - 1) // block_n
    k_tiles = (K + block_k - 1) // block_k
    assert n_tiles == Bs.shape[0]
    assert k_tiles == Bs.shape[1]

    C_shape = (M, N)
    C = torch.zeros(C_shape, dtype=torch.float32, device=A.device)

    A_tiles = [A[:, i * block_k : min((i + 1) * block_k, K)] for i in range(k_tiles)]
    B_tiles = [
        [
            B[
                j * block_n : min((j + 1) * block_n, N),
                i * block_k : min((i + 1) * block_k, K),
            ]
            for i in range(k_tiles)
        ]
        for j in range(n_tiles)
    ]
    C_tiles = [C[:, j * block_n : min((j + 1) * block_n, N)] for j in range(n_tiles)]
    As_tiles = [As[:, i : i + 1] for i in range(k_tiles)]

    for i in range(k_tiles):
        for j in range(n_tiles):
            a = A_tiles[i]
            b = B_tiles[j][i]
            c = C_tiles[j]
            s = As_tiles[i] * Bs[j][i]
            c[:, :] += torch.matmul(a, b.t()) * s

    C = C.reshape(origin_C_shape).to(output_dtype)
    return C
```
**EN:** This function performs matrix multiplication with block-wise quantization using native torch. This block implements `native_w8a8_block_int8_matmul` and captures one focused piece of the module's behavior.
**CN:** This function performs matrix multiplication with block-wise quantization using native torch. 该代码块实现 `native_w8a8_block_int8_matmul`，承担模块行为中的一个聚焦逻辑片段。

### Lines 98-100: supporting source context / 辅助源码上下文
```python


# For test
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 101-132: function torch w8a8 block int8 moe / 函数 torch w8a8 block int8 moe
```python
def torch_w8a8_block_int8_moe(a, w1, w2, w1_s, w2_s, topk_output, topk, block_shape):
    """This function performs fused moe with block-wise quantization using native torch."""

    set_global_server_args_for_scheduler(ServerArgs(model_path="dummy"))

    B, D = a.shape
    a = a.view(B, -1, D).repeat(1, topk, 1).reshape(-1, D)
    out = torch.zeros(B * topk, w2.shape[1], dtype=a.dtype, device=a.device)
    # Use topk_output instead of torch.topk for consistent equal-value handling
    # moeTopK kernel and torch.topk may differ in tie-breaking for equal values
    topk_weight, topk_ids = topk_output.topk_weights, topk_output.topk_ids

    topk_weight = topk_weight.view(-1)
    topk_ids = topk_ids.view(-1)

    _, block_k = block_shape[0], block_shape[1]
    a_q, a_s = native_per_token_group_quant_int8(a, block_k)
    for i in range(w1.shape[0]):
        mask = topk_ids == i
        if mask.sum():
            inter_out = native_w8a8_block_int8_matmul(
                a_q[mask], w1[i], a_s[mask], w1_s[i], block_shape, output_dtype=a.dtype
            )
            act_out = SiluAndMul().forward_native(inter_out)
            act_out_q, act_out_s = native_per_token_group_quant_int8(act_out, block_k)
            act_out = act_out.to(torch.float32)
            out[mask] = native_w8a8_block_int8_matmul(
                act_out_q, w2[i], act_out_s, w2_s[i], block_shape, output_dtype=a.dtype
            )
    return (
        out.view(B, -1, w2.shape[1]) * topk_weight.view(B, -1, 1).to(out.dtype)
    ).sum(dim=1)
```
**EN:** This function performs fused moe with block-wise quantization using native torch. This block implements `torch_w8a8_block_int8_moe` and captures one focused piece of the module's behavior.
**CN:** This function performs fused moe with block-wise quantization using native torch. 该代码块实现 `torch_w8a8_block_int8_moe`，承担模块行为中的一个聚焦逻辑片段。

### Lines 135-135: class TestW8A8BlockINT8FusedMoE declaration / 类 TestW8A8BlockINT8FusedMoE 声明
```python
class TestW8A8BlockINT8FusedMoE(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 136-144: class-level constants and configuration for `TestW8A8BlockINT8FusedMoE` / 类级常量与配置
```python
    DTYPES = [torch.half, torch.bfloat16]
    M = [1, 33, 64, 222]
    N = [128, 1024]
    K = [256, 4096]
    E = [8, 24]
    TOP_KS = [2, 6]
    # BLOCK_SIZE = [[64, 64], [64, 128], [128, 64], [128, 128]]
    BLOCK_SIZE = [[128, 128]]
    SEEDS = [0]
```
**EN:** This block defines shared names such as `DTYPES`, `M`, `N`, `K`, `E`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `DTYPES`, `M`, `N`, `K`, `E` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 146-150: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA is not available")
        torch.set_default_device("cuda")
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 152-209: method w8a8 block int8 fused moe / 方法 w8a8 block int8 fused moe
```python
    def _w8a8_block_int8_fused_moe(self, M, N, K, E, topk, block_size, dtype, seed):
        torch.manual_seed(seed)
        # NOTE(HandH1998): to avoid overflow when out_dtype = torch.half
        factor_for_scale = 1e-2
        int8_info = torch.iinfo(torch.int8)
        int8_max, int8_min = int8_info.max, int8_info.min

        a = torch.randn((M, K), dtype=dtype) / 10

        w1_fp32 = (torch.rand((E, 2 * N, K), dtype=torch.float32) - 0.5) * 2 * int8_max
        w1 = w1_fp32.clamp(min=int8_min, max=int8_max).to(torch.int8)

        w2_fp32 = (torch.rand((E, K, N), dtype=torch.float32) - 0.5) * 2 * int8_max
        w2 = w2_fp32.clamp(min=int8_min, max=int8_max).to(torch.int8)

        block_n, block_k = block_size[0], block_size[1]
        n_tiles_w1 = (2 * N + block_n - 1) // block_n
        n_tiles_w2 = (K + block_n - 1) // block_n
        k_tiles_w1 = (K + block_k - 1) // block_k
        k_tiles_w2 = (N + block_k - 1) // block_k

        w1_s = (
            torch.rand((E, n_tiles_w1, k_tiles_w1), dtype=torch.float32)
            * factor_for_scale
        )
        w2_s = (
            torch.rand((E, n_tiles_w2, k_tiles_w2), dtype=torch.float32)
            * factor_for_scale
        )

        score = torch.randn((M, E), dtype=dtype)

        topk_output = select_experts(
            hidden_states=a,
            router_logits=score,
            topk_config=TopKConfig(top_k=topk, renormalize=False),
        )

        with torch.inference_mode():
            ref_out = torch_w8a8_block_int8_moe(
                a, w1, w2, w1_s, w2_s, topk_output, topk, block_size
            )
            out = fused_moe(
                a,
                w1,
                w2,
                topk_output,
                use_int8_w8a8=True,
                w1_scale=w1_s,
                w2_scale=w2_s,
                block_shape=block_size,
            )

        self.assertTrue(
            torch.mean(torch.abs(out.to(torch.float32) - ref_out.to(torch.float32)))
            / torch.mean(torch.abs(ref_out.to(torch.float32)))
            < 0.02
        )
```
**EN:** This block implements `_w8a8_block_int8_fused_moe` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_w8a8_block_int8_fused_moe`，承担模块行为中的一个聚焦逻辑片段。

### Lines 211-232: test case w8a8 block int8 fused moe / 测试用例 w8a8 block int8 fused moe
```python
    def test_w8a8_block_int8_fused_moe(self):
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
                self._w8a8_block_int8_fused_moe(*params)
```
**EN:** This test exercises `test_w8a8_block_int8_fused_moe` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_w8a8_block_int8_fused_moe`。

### Lines 235-236: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `native_per_token_group_quant_int8`: Function to perform per-token-group quantization on an input tensor `x` using native torch. / 该代码块实现 `native_per_token_group_quant_int8`，承担模块行为中的一个聚焦逻辑片段。
- `native_w8a8_block_int8_matmul`: This function performs matrix multiplication with block-wise quantization using native torch. / 该代码块实现 `native_w8a8_block_int8_matmul`，承担模块行为中的一个聚焦逻辑片段。
- `torch_w8a8_block_int8_moe`: This function performs fused moe with block-wise quantization using native torch. / 该代码块实现 `torch_w8a8_block_int8_moe`，承担模块行为中的一个聚焦逻辑片段。
- `TestW8A8BlockINT8FusedMoE`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestW8A8BlockINT8FusedMoE.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestW8A8BlockINT8FusedMoE._w8a8_block_int8_fused_moe`: This block implements `_w8a8_block_int8_fused_moe` and captures one focused piece of the module's behavior. / 该代码块实现 `_w8a8_block_int8_fused_moe`，承担模块行为中的一个聚焦逻辑片段。
- `TestW8A8BlockINT8FusedMoE.test_w8a8_block_int8_fused_moe`: This test exercises `test_w8a8_block_int8_fused_moe` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_w8a8_block_int8_fused_moe`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `itertools`, `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.layers.activation`, `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe`, `sglang.srt.layers.moe.topk`, `sglang.srt.server_args`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 236
