# test_fp4_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/kernels/test_fp4_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates fp4 moe behavior in SGLang's kernels area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 kernels 领域中与 fp4 moe 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting source context / 辅助源码上下文
```python
# SPDX-License-Identifier: Apache-2.0
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 2-15: module imports and dependencies / 模块导入与依赖
```python
from typing import Callable

import pytest
import torch
from flashinfer import fp4_quantize, scaled_fp4_grouped_quantize
from flashinfer.fused_moe import cutlass_fused_moe as flashinfer_cutlass_fused_moe
from sgl_kernel import silu_and_mul
from torch.nn import functional as F

from sglang.jit_kernel.nvfp4 import scaled_fp4_quant
from sglang.srt.layers.moe.cutlass_moe import cutlass_moe_fp4
from sglang.srt.layers.moe.cutlass_moe_params import CutlassMoEParams, CutlassMoEType
from sglang.srt.layers.moe.topk import TopKConfig, select_experts
from sglang.test.ci.ci_register import register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `typing`, `pytest`, `torch`, `flashinfer`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `typing`, `pytest`, `torch`, `flashinfer`。

### Lines 17-30: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=300, suite="nightly-4-gpu-b200", nightly=True)

if torch.cuda.get_device_capability() < (10, 0):
    pytest.skip(
        reason="Nvfp4 Requires compute capability of 10 or above.",
        allow_module_level=True,
    )

kE2M1ToFloat = torch.tensor(
    [0.0, 0.5, 1.0, 1.5, 2.0, 3.0, 4.0, 6.0], dtype=torch.float32
)

FLOAT8_E4M3_MAX = 448.0
FLOAT4_E2M1_MAX = 6.0
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, get_device_capability, skip.
**CN:** 该代码块通过 register_cuda_ci, get_device_capability, skip 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 33-40: function convert swizzled to linear / 函数 convert swizzled to linear
```python
def convert_swizzled_to_linear(a_sf_swizzled: torch.Tensor, m, k, block_size):
    m_tiles = (m + 128 - 1) // 128
    f = block_size * 4
    k_tiles = (k + f - 1) // f
    tmp = torch.reshape(a_sf_swizzled, (1, m_tiles, k_tiles, 32, 4, 4))
    tmp = torch.permute(tmp, (0, 1, 4, 3, 2, 5))
    out = tmp.reshape(m_tiles * 128, k_tiles * f // block_size)
    return out[0:m, 0:k]
```
**EN:** This block implements `convert_swizzled_to_linear` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `convert_swizzled_to_linear`，承担模块行为中的一个聚焦逻辑片段。

### Lines 43-59: function dequantize nvfp4 to dtype / 函数 dequantize nvfp4 to dtype
```python
def dequantize_nvfp4_to_dtype(
    tensor_fp4, tensor_sf, global_scale, dtype, device, block_size=16
):
    """Dequantize the fp4 tensor back to high precision."""
    # Two fp4 values are packed into one uint8.
    assert tensor_fp4.dtype == torch.uint8
    m, packed_k = tensor_fp4.shape
    k = packed_k * 2
    tensor_f32 = break_fp4_bytes(tensor_fp4, dtype)
    tensor_f32 = tensor_f32.reshape(m, k // block_size, block_size)
    tensor_sf = tensor_sf.view(torch.float8_e4m3fn)
    tensor_sf = convert_swizzled_to_linear(tensor_sf, m, k, block_size)
    tensor_sf_dtype = tensor_sf.to(torch.float32) / global_scale

    # scale the tensor
    out = (tensor_f32 * tensor_sf_dtype.unsqueeze(-1)).reshape(m, k)
    return out.to(dtype=dtype)
```
**EN:** Dequantize the fp4 tensor back to high precision. This block implements `dequantize_nvfp4_to_dtype` and captures one focused piece of the module's behavior.
**CN:** Dequantize the fp4 tensor back to high precision. 该代码块实现 `dequantize_nvfp4_to_dtype`，承担模块行为中的一个聚焦逻辑片段。

### Lines 62-83: function break fp4 bytes / 函数 break fp4 bytes
```python
def break_fp4_bytes(a, dtype):
    assert a.dtype == torch.uint8
    m, n = a.shape

    # Vectorized nibble processing
    a_flat = a.flatten()
    high = (a_flat & 0xF0) >> 4  # Upper nibbles
    low = a_flat & 0x0F  # Lower nibbles

    # Combine nibbles for batch processing
    combined = torch.stack((low, high), dim=1).flatten()

    # Vectorized sign and magnitude extraction
    signs = (combined & 0x08).to(torch.bool)  # Sign bits
    abs_vals = (combined & 0x07).to(torch.long)  # Magnitude indices

    # Device-aware lookup and sign application
    kE2M1 = kE2M1ToFloat.to(device=a.device)
    values = kE2M1[abs_vals] * torch.where(signs, -1.0, 1.0)

    # Reshape to final form
    return values.reshape(m, n * 2).to(dtype=dtype)
```
**EN:** This block implements `break_fp4_bytes` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `break_fp4_bytes`，承担模块行为中的一个聚焦逻辑片段。

### Lines 86-91: function compute routing / 函数 compute routing
```python
def compute_routing(router_logits: torch.Tensor, top_k: int):
    routing_weights = torch.softmax(router_logits, dim=1, dtype=torch.float)
    routing_weights, selected_experts = torch.topk(routing_weights, top_k, dim=-1)
    routing_weights /= routing_weights.sum(dim=-1, keepdim=True)
    routing_weights = routing_weights.float()
    return routing_weights, selected_experts
```
**EN:** This block implements `compute_routing` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `compute_routing`，承担模块行为中的一个聚焦逻辑片段。

### Lines 94-114: function prepare inputs / 函数 prepare inputs
```python
def prepare_inputs(
    hidden_states: torch.Tensor,
    router_logits: torch.Tensor,
    num_experts: int,
    topk: int,
):
    routing_weights, topk_idx = compute_routing(router_logits, topk)

    masked_m = []
    for i in range(num_experts):
        mask = topk_idx.view(-1) == i
        masked_m.append(mask.sum())

    masked_m = torch.tensor(masked_m, dtype=torch.int32)
    hidden_states_3d = torch.empty(
        (num_experts, max(masked_m), hidden_states.shape[1]), dtype=hidden_states.dtype
    )
    for i in range(num_experts):
        hidden_states_3d[i, : masked_m[i], :] = hidden_states[topk_idx.view(-1) == i]

    return hidden_states_3d, masked_m, topk_idx, routing_weights
```
**EN:** This block implements `prepare_inputs` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `prepare_inputs`，承担模块行为中的一个聚焦逻辑片段。

### Lines 117-128: module-level constants and configuration / 模块级常量与配置
```python
MNK_FACTORS = [
    (2, 1024, 1024),
    (2, 1024, 1536),
    (2, 3072, 1024),
    (2, 3072, 1536),
    (64, 1024, 1024),
    (64, 1024, 1536),
    (64, 3072, 1024),
    (64, 2048, 1024),
    (224, 1024, 1024),
    (224, 1024, 1536),
]
```
**EN:** This block defines shared names such as `MNK_FACTORS`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `MNK_FACTORS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 129-131: supporting source context / 辅助源码上下文
```python


# Reference implementation of torch_moe
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 132-150: function torch moe / 函数 torch moe
```python
def torch_moe(a, w1, w2, score, topk, expert_map):
    B, D = a.shape
    a = a.view(B, -1, D).repeat(1, topk, 1).reshape(-1, D)
    out = torch.zeros(B * topk, w2.shape[1], dtype=a.dtype, device=a.device)
    score = torch.softmax(score, dim=-1, dtype=torch.float32)
    topk_weight, topk_ids = torch.topk(score, topk)
    topk_weight = topk_weight.view(-1)
    topk_ids = topk_ids.view(-1)
    if expert_map is not None:
        topk_ids = expert_map[topk_ids]
    for i in range(w1.shape[0]):
        mask = topk_ids == i
        if mask.sum():
            out[mask] = silu_and_mul(a[mask] @ w1[i].transpose(0, 1)) @ w2[i].transpose(
                0, 1
            )
    return (
        out.view(B, -1, w2.shape[1]) * topk_weight.view(B, -1, 1).to(out.dtype)
    ).sum(dim=1)
```
**EN:** This block implements `torch_moe` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `torch_moe`，承担模块行为中的一个聚焦逻辑片段。

### Lines 153-182: function torch moe nvfp4 / 函数 torch moe nvfp4
```python
def torch_moe_nvfp4(a, w1, w2, topk, topk_weight, topk_ids):
    B, D = a.shape
    a = a.view(B, -1, D).repeat(1, topk, 1).reshape(-1, D)
    out = torch.zeros(B * topk, w2.shape[1], dtype=a.dtype, device=a.device)

    topk_weight = topk_weight.view(-1)
    topk_ids = topk_ids.view(-1)

    for i in range(w1.shape[0]):
        mask = topk_ids == i
        if mask.sum():
            m = w1[i].shape[0]
            assert m % 2 == 0
            # Note: w1 and w3 are swapped!
            w3_expert, w1_expert = w1[i][m // 2 :, :], w1[i][: m // 2, :]
            inter = F.silu(a[mask] @ w1_expert.t()) * (a[mask] @ w3_expert.t())
            inter_gs = torch.tensor(1.0).cuda()
            inter_q, inter_blockscale = fp4_quantize(inter, inter_gs)
            inter = dequantize_nvfp4_to_dtype(
                inter_q,
                inter_blockscale,
                inter_gs,
                dtype=inter.dtype,
                device=inter.device,
                block_size=16,
            ).cuda()
            out[mask] = inter @ w2[i].transpose(0, 1)
    return (
        out.view(B, -1, w2.shape[1]) * topk_weight.view(B, -1, 1).to(out.dtype)
    ).sum(dim=1)
```
**EN:** This block implements `torch_moe_nvfp4` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `torch_moe_nvfp4`，承担模块行为中的一个聚焦逻辑片段。

### Lines 185-243: function flashinfer cutedsl grouped gemm nt masked / 函数 flashinfer cutedsl grouped gemm nt masked
```python
def flashinfer_cutedsl_grouped_gemm_nt_masked(
    hidden_states: torch.Tensor,  # 3d
    input_global_scale: torch.Tensor,  # (l,)
    weights: torch.Tensor,
    w_global_scale: torch.Tensor,  # (l,)
    masked_m: torch.Tensor,
):
    from flashinfer.cute_dsl.blockscaled_gemm import grouped_gemm_nt_masked

    # hidden_states: [l, m, k]
    # weights: [l, n, k]
    aq, aq_sf = scaled_fp4_grouped_quantize(
        hidden_states,
        masked_m.to(hidden_states.device),
        input_global_scale,
    )
    num_experts, n, k = weights.shape
    bq, bq_sf = scaled_fp4_grouped_quantize(
        weights,
        torch.ones(num_experts, device=weights.device, dtype=torch.int32) * n,
        w_global_scale,
    )

    out = torch.zeros(
        (num_experts, max(masked_m), n), dtype=weights.dtype, device=aq.device
    )
    out = out.permute(1, 2, 0)  # requirement of kernel
    sf_vec_size = 16
    ab_dtype = "float4_e2m1fn"
    sf_dtype = "float8_e4m3fn"
    c_dtype = "bfloat16"
    alpha = 1.0 / (input_global_scale * w_global_scale).to(out.dtype).view(
        1, 1, num_experts
    )

    def get_cute_dtype(input: torch.Tensor) -> str:
        if input.dtype == torch.bfloat16:
            return "bfloat16"
        elif input.dtype == torch.float16:
            return "float16"
        elif input.dtype == torch.float32:
            return "float32"
        else:
            raise ValueError(f"Unsupported cute dtype {input.dtype}")

    grouped_gemm_nt_masked(
        (aq, aq_sf),
        (bq, bq_sf),
        out,
        masked_m.to(aq.device),
        ab_dtype=ab_dtype,
        sf_dtype=sf_dtype,
        c_dtype=c_dtype,
        sf_vec_size=sf_vec_size,
        alpha=alpha,
        alpha_dtype=get_cute_dtype(alpha),
    )

    return out
```
**EN:** This block implements `flashinfer_cutedsl_grouped_gemm_nt_masked` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `flashinfer_cutedsl_grouped_gemm_nt_masked`，承担模块行为中的一个聚焦逻辑片段。

### Lines 246-325: helper routine check moe (part 1/2) / 辅助流程 check moe（第 1/2 部分）
```python
def check_moe(
    m: int,
    n: int,
    k: int,
    e: int,
    topk: int,
    dtype: torch.dtype,
    moe_impl: Callable,
    flip_w13: bool,
):
    torch.manual_seed(7)
    a = torch.randn((m, k), device="cuda", dtype=dtype) / 10
    w1 = torch.randn((e, 2 * n, k), device="cuda", dtype=dtype) / 10
    quant_blocksize = 16
    round_up = lambda x, y: (x + y - 1) // y * y
    sf_w1_2n = round_up(2 * n, 128)
    sf_w1_k = round_up(k // quant_blocksize, 4)
    w1_blockscale = torch.empty(
        (e, sf_w1_2n, sf_w1_k), device="cuda", dtype=torch.float8_e4m3fn
    )

    w2 = torch.randn((e, k, n), device="cuda", dtype=dtype) / 10
    sf_w2_k = round_up(k, 128)
    sf_w2_n = round_up(n // quant_blocksize, 4)
    w2_blockscale = torch.empty(
        (e, sf_w2_k, sf_w2_n), device="cuda", dtype=torch.float8_e4m3fn
    )

    w1_q = torch.empty((e, 2 * n, k // 2), device="cuda", dtype=torch.uint8)
    w2_q = torch.empty((e, k, n // 2), device="cuda", dtype=torch.uint8)
    w1_gs = torch.empty((e,), device="cuda", dtype=torch.float32)
    w2_gs = torch.empty((e,), device="cuda", dtype=torch.float32)

    for expert in range(e):
        w1_amax = torch.abs(w1).max().to(torch.float32)
        w2_amax = torch.abs(w2).max().to(torch.float32)
        w1_gs[expert] = FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX / w1_amax
        w2_gs[expert] = FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX / w2_amax

        w1_q[expert], w1_blockscale[expert] = scaled_fp4_quant(
            w1[expert], w1_gs[expert]
        )

        w2_q[expert], w2_blockscale[expert] = scaled_fp4_quant(
            w2[expert], w2_gs[expert]
        )

    score = torch.randn((m, e), device="cuda", dtype=dtype)

    topk_output = select_experts(
        hidden_states=a,
        router_logits=score,
        topk_config=TopKConfig(top_k=topk, renormalize=False),
    )
    topk_weights, topk_ids, _ = topk_output

    a1_gs = torch.ones((e,), device="cuda", dtype=torch.float32)
    a2_gs = torch.ones((e,), device="cuda", dtype=torch.float32)
    test_output = moe_impl(
        a=a,
        topk_weights=topk_weights,
        topk_ids=topk_ids,
        w1_q=w1_q,
        w2_q=w2_q,
        a1_gs=a1_gs,
        w1_blockscale=w1_blockscale,
        w1_alphas=(1 / w1_gs),
        a2_gs=a2_gs,
        w2_blockscale=w2_blockscale,
        w2_alphas=(1 / w2_gs),
    )

    # Reference check:
    a_global_scale = (
        (FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX) / torch.amax(a.flatten(), dim=-1)
    ).to(torch.float32)
    a_fp4, a_scale_interleaved = scaled_fp4_quant(a, a_global_scale)
    _, m_k = a_fp4.shape
    a_in_dtype = dequantize_nvfp4_to_dtype(
        a_fp4,
```
**EN:** This helper encapsulates `check_moe` so the surrounding tests can reuse setup, execution, or validation logic. This chunk continues part 1 of the same logical block.
**CN:** 该辅助函数封装了 `check_moe`，以便周围测试复用准备、执行或校验逻辑。 这一段对应同一逻辑块的第 1 部分。

### Lines 326-365: helper routine check moe (part 2/2) / 辅助流程 check moe（第 2/2 部分）
```python
        a_scale_interleaved,
        a_global_scale,
        dtype=a.dtype,
        device=a.device,
        block_size=quant_blocksize,
    )

    w1_d = torch.empty((e, 2 * n, k), device="cuda", dtype=dtype)
    w2_d = torch.empty((e, k, n), device="cuda", dtype=dtype)

    for idx in range(0, e):
        w1_d[idx] = dequantize_nvfp4_to_dtype(
            w1_q[idx],
            w1_blockscale[idx],
            w1_gs[idx],
            dtype=w1.dtype,
            device=w1.device,
            block_size=quant_blocksize,
        )
        w2_d[idx] = dequantize_nvfp4_to_dtype(
            w2_q[idx],
            w2_blockscale[idx],
            w2_gs[idx],
            dtype=w2.dtype,
            device=w2.device,
            block_size=quant_blocksize,
        )

    if flip_w13:
        dim = -2
        size = w1_d.size(dim)
        assert size % 2 == 0, f"Expected even size in dim {dim}, got {size}"
        half = size // 2
        # Reorder weight
        w1, w3 = w1_d.split(half, dim=dim)
        w1_d = torch.cat([w3, w1], dim=dim).contiguous()

    torch_output = torch_moe(a_in_dtype, w1_d, w2_d, score, topk, None)

    torch.testing.assert_close(torch_output, test_output, atol=1e-1, rtol=1e-1)
```
**EN:** This helper encapsulates `check_moe` so the surrounding tests can reuse setup, execution, or validation logic. This chunk continues part 2 of the same logical block.
**CN:** 该辅助函数封装了 `check_moe`，以便周围测试复用准备、执行或校验逻辑。 这一段对应同一逻辑块的第 2 部分。

### Lines 368-412: test case cutlass fp4 moe no graph / 测试用例 cutlass fp4 moe no graph
```python
@pytest.mark.parametrize("m,n,k", MNK_FACTORS)
@pytest.mark.parametrize("e", [40, 64, 256])
@pytest.mark.parametrize("topk", [1, 6, 8])
@pytest.mark.parametrize("dtype", [torch.half, torch.bfloat16])
@torch.inference_mode()
def test_cutlass_fp4_moe_no_graph(
    m: int, n: int, k: int, e: int, topk: int, dtype: torch.dtype
):
    def cutlass_moe_impl(
        a,
        topk_weights,
        topk_ids,
        w1_q,
        w2_q,
        a1_gs,
        w1_blockscale,
        w1_alphas,
        a2_gs,
        w2_blockscale,
        w2_alphas,
    ):
        params = CutlassMoEParams(
            CutlassMoEType.BlockscaledFP4,
            device=a.device,
            num_experts=e,
            intermediate_size_per_partition=n,  # n
            hidden_size=k,
        )  # k
        return cutlass_moe_fp4(
            a=a,
            a1_gscale=a1_gs,
            w1_fp4=w1_q,
            w1_blockscale=w1_blockscale,
            w1_alphas=w1_alphas,
            a2_gscale=a2_gs,
            w2_fp4=w2_q,
            w2_blockscale=w2_blockscale,
            w2_alphas=w2_alphas,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            params=params,
            apply_router_weight_on_input=False,
        )

    check_moe(m, n, k, e, topk, dtype, cutlass_moe_impl, flip_w13=False)
```
**EN:** This test exercises `test_cutlass_fp4_moe_no_graph` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_cutlass_fp4_moe_no_graph`。

### Lines 415-453: test case flashinfer fp4 moe no graph / 测试用例 flashinfer fp4 moe no graph
```python
@pytest.mark.parametrize("m,n,k", MNK_FACTORS)
@pytest.mark.parametrize("e", [40, 64, 256])
@pytest.mark.parametrize("topk", [1, 6, 8])
@pytest.mark.parametrize("dtype", [torch.half, torch.bfloat16])
@torch.inference_mode()
def test_flashinfer_fp4_moe_no_graph(
    m: int, n: int, k: int, e: int, topk: int, dtype: torch.dtype
):
    def flashinfer_moe_impl(
        a,
        topk_weights,
        topk_ids,
        w1_q,
        w2_q,
        a1_gs,
        w1_blockscale,
        w1_alphas,
        a2_gs,
        w2_blockscale,
        w2_alphas,
    ):
        return flashinfer_cutlass_fused_moe(
            a,
            topk_ids.to(torch.int),
            topk_weights,
            w1_q.view(torch.long),
            w2_q.view(torch.long),
            a.dtype,
            quant_scales=[
                a1_gs,
                w1_blockscale.view(torch.int32),
                w1_alphas,
                a2_gs,
                w2_blockscale.view(torch.int32),
                w2_alphas,
            ],
        )[0]

    check_moe(m, n, k, e, topk, dtype, flashinfer_moe_impl, flip_w13=True)
```
**EN:** This test exercises `test_flashinfer_fp4_moe_no_graph` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_flashinfer_fp4_moe_no_graph`。

### Lines 456-458: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    test_cutlass_fp4_moe_no_graph(224, 1024, 1024, 256, 8, torch.half)
    test_flashinfer_fp4_moe_no_graph(224, 1024, 1024, 256, 8, torch.half)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `convert_swizzled_to_linear`: This block implements `convert_swizzled_to_linear` and captures one focused piece of the module's behavior. / 该代码块实现 `convert_swizzled_to_linear`，承担模块行为中的一个聚焦逻辑片段。
- `dequantize_nvfp4_to_dtype`: Dequantize the fp4 tensor back to high precision. / 该代码块实现 `dequantize_nvfp4_to_dtype`，承担模块行为中的一个聚焦逻辑片段。
- `break_fp4_bytes`: This block implements `break_fp4_bytes` and captures one focused piece of the module's behavior. / 该代码块实现 `break_fp4_bytes`，承担模块行为中的一个聚焦逻辑片段。
- `compute_routing`: This block implements `compute_routing` and captures one focused piece of the module's behavior. / 该代码块实现 `compute_routing`，承担模块行为中的一个聚焦逻辑片段。
- `prepare_inputs`: This block implements `prepare_inputs` and captures one focused piece of the module's behavior. / 该代码块实现 `prepare_inputs`，承担模块行为中的一个聚焦逻辑片段。
- `torch_moe`: This block implements `torch_moe` and captures one focused piece of the module's behavior. / 该代码块实现 `torch_moe`，承担模块行为中的一个聚焦逻辑片段。
- `torch_moe_nvfp4`: This block implements `torch_moe_nvfp4` and captures one focused piece of the module's behavior. / 该代码块实现 `torch_moe_nvfp4`，承担模块行为中的一个聚焦逻辑片段。
- `flashinfer_cutedsl_grouped_gemm_nt_masked`: This block implements `flashinfer_cutedsl_grouped_gemm_nt_masked` and captures one focused piece of the module's behavior. / 该代码块实现 `flashinfer_cutedsl_grouped_gemm_nt_masked`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party modules / 第三方模块**: `pytest`, `torch`, `flashinfer`, `flashinfer.fused_moe`, `sgl_kernel`, `torch.nn`
- **Internal modules / 内部模块**: `sglang.jit_kernel.nvfp4`, `sglang.srt.layers.moe.cutlass_moe`, `sglang.srt.layers.moe.cutlass_moe_params`, `sglang.srt.layers.moe.topk`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 458
