# test_cutedsl_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_cutedsl_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_cutedsl_moe, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_cutedsl_moe 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 5-7)
```python
import pytest

from vllm.platforms import current_platform
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest; and vLLM components like vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest；vLLM 内部组件，例如 vllm.platforms。

### Top-level block starting at line 9 (lines 9-13)
```python
if not current_platform.has_device_capability(100):
    pytest.skip(
        reason="Nvfp4 Requires compute capability of 10 or above.",
        allow_module_level=True,
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Imports and shared setup (lines 15-28)
```python
import torch
from flashinfer import fp4_quantize
from torch.nn import functional as F

from vllm.model_executor.layers.activation import SiluAndMul
from vllm.model_executor.layers.fused_moe.experts.flashinfer_cutedsl_batched_moe import (  # noqa: E501
    flashinfer_cutedsl_moe_masked,
)
from vllm.utils.flashinfer import (
    flashinfer_cutedsl_grouped_gemm_nt_masked as cutedsl_gmm_masked,
)
from vllm.utils.flashinfer import (
    scaled_fp4_grouped_quantize,
)
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as torch, flashinfer, torch.nn; and vLLM components like vllm.model_executor.layers.activation, vllm.model_executor.layers.fused_moe.experts.flashinfer_cutedsl_batched_moe, vllm.utils.flashinfer.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 torch、flashinfer、torch.nn；vLLM 内部组件，例如 vllm.model_executor.layers.activation、vllm.model_executor.layers.fused_moe.experts.flashinfer_cutedsl_batched_moe、vllm.utils.flashinfer。

### Constants and module state (lines 30-35)
```python
kE2M1ToFloat = torch.tensor(
    [0.0, 0.5, 1.0, 1.5, 2.0, 3.0, 4.0, 6.0], dtype=torch.float32
)

FLOAT8_E4M3_MAX = 448.0
FLOAT4_E2M1_MAX = 6.0
```
**EN:** This block centralizes shared constants and parameter grids, including FLOAT8_E4M3_MAX, FLOAT4_E2M1_MAX. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 FLOAT8_E4M3_MAX、FLOAT4_E2M1_MAX。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `convert_swizzled_to_linear` (lines 38-45)
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
**EN:** This helper function implements the shared logic for convert swizzled to linear. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 convert swizzled to linear 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `dequantize_nvfp4_to_dtype` (lines 48-64)
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
**EN:** This helper function implements the shared logic for dequantize nvfp4 to dtype. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 dequantize nvfp4 to dtype 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `break_fp4_bytes` (lines 67-88)
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
**EN:** This helper function implements the shared logic for break fp4 bytes. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 break fp4 bytes 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `generate_balanced_routing` (lines 91-123)
```python
def generate_balanced_routing(
    hidden_states: torch.Tensor, num_experts: int, top_k: int
):
    """
    Generate routing weights and topk indices such that every expert is active.
    Returns routing_weights, topk_idx
    """

    num_tokens, hidden_dim = hidden_states.shape
    #   num_tokens = batch_size * seq_len

    # First, assign at least one token per expert
    tokens_per_expert = torch.arange(num_tokens) % num_experts
    tokens_per_expert = tokens_per_expert[torch.randperm(num_tokens)]  # shuffle

    # Each token has top_k experts — start with one guaranteed expert
    topk_idx = torch.full((num_tokens, top_k), -1, dtype=torch.long)
    topk_idx[:, 0] = tokens_per_expert

    # For remaining top_k - 1 experts, pick randomly (allowing repeats)
    if top_k > 1:
        random_choices = torch.randint(0, num_experts, (num_tokens, top_k - 1))
        topk_idx[:, 1:] = random_choices

    # Normalize routing weights so each token's weights sum to 1
    routing_weights = torch.rand(num_tokens, top_k)
    routing_weights /= routing_weights.sum(dim=-1, keepdim=True)

    # Reshape back if needed
    routing_weights = routing_weights.view(num_tokens, top_k)
    topk_idx = topk_idx.view(num_tokens, top_k)

    return routing_weights, topk_idx
```
**EN:** This helper function implements the shared logic for generate balanced routing. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 generate balanced routing 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `prepare_inputs` (lines 126-152)
```python
def prepare_inputs(
    hidden_states: torch.Tensor,
    router_logits: torch.Tensor,
    num_experts: int,
    topk: int,
):
    routing_weights, topk_idx = generate_balanced_routing(
        router_logits, num_experts, topk
    )

    masked_m = []
    for i in range(num_experts):
        mask = topk_idx.view(-1) == i
        masked_m.append(mask.sum())

    masked_m = torch.tensor(masked_m, dtype=torch.int32)
    # Initialize the hidden_states_3d with ones instead of empty to avoid nan
    # issue.
    hidden_states_3d = torch.ones(
        (num_experts, max(masked_m), hidden_states.shape[1]),
        dtype=hidden_states.dtype,
        device=hidden_states.device,
    )
    for i in range(num_experts):
        hidden_states_3d[i, : masked_m[i], :] = hidden_states[topk_idx.view(-1) == i]

    return hidden_states_3d, masked_m, topk_idx, routing_weights
```
**EN:** This helper function implements the shared logic for prepare inputs. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 prepare inputs 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 155-166)
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
**EN:** This block centralizes shared constants and parameter grids, including MNK_FACTORS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 MNK_FACTORS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `torch_moe` (lines 170-188)
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
            out[mask] = SiluAndMul()(a[mask] @ w1[i].transpose(0, 1)) @ w2[i].transpose(
                0, 1
            )
    return (
        out.view(B, -1, w2.shape[1]) * topk_weight.view(B, -1, 1).to(out.dtype)
    ).sum(dim=1)
```
**EN:** This helper function implements the shared logic for torch MoE. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 torch MoE 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `torch_moe_nvfp4` (lines 191-220)
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
**EN:** This helper function implements the shared logic for torch MoE nvfp4. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 torch MoE nvfp4 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `grouped_gemm_ref` (lines 223-308)
```python
def grouped_gemm_ref(
    hidden_states_expanded: torch.Tensor,
    hidden_states_3d: torch.Tensor,
    weights: torch.Tensor,
    topk_idx: torch.Tensor,
    masked_m: torch.Tensor,
    B: int,
    topk: int,
    num_experts: int,
    *,
    block_size: int = 16,
) -> torch.Tensor:
    """
    Computes the reference grouped GEMM (fp4 quantized per-expert loop),
    computes flashinfer grouped GEMM (for scale consistency),
    and returns ONLY the repacked reference output: out_ref.

    Returns:
        out_ref: Tensor [num_experts, max_m, n_out]
    """
    device_hs = hidden_states_expanded.device
    device_w = weights.device
    out_dtype = weights.dtype
    n_out = weights.shape[1]

    # Flattened reference output (B*topk, n_out)
    out = torch.zeros((B * topk, n_out), dtype=out_dtype, device=device_w)

    # Per-expert reference compute loop
    for i in range(num_experts):
        mask = topk_idx.view(-1) == i
        if mask.any():
            lhs = hidden_states_expanded[mask]
            rhs = weights[i]

            a_amax = lhs.abs().max().to(torch.float32).to(device_hs)
            b_amax = rhs.abs().max().to(torch.float32).to(device_w)

            a_gs = FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX / a_amax
            b_gs = FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX / b_amax
# ... excerpt ...
                a_gs,
                dtype=lhs.dtype,
                device=device_hs,
                block_size=block_size,
            )
            rhs_in_dtype = dequantize_nvfp4_to_dtype(
                rhsq,
                rhsq_sf,
                b_gs,
                dtype=rhs.dtype,
                device=device_w,
                block_size=block_size,
            )

            out[mask] = lhs_in_dtype @ rhs_in_dtype.t()

    # Determine per-expert max_m
    max_m_val = int(masked_m.max().item())

    # Repack into [num_experts, max_m, n_out]
    out_ref = torch.zeros(
        (num_experts, max_m_val, n_out),
        dtype=out.dtype,
        device=out.device,
    )
    expert_slot = [0] * num_experts

    for i, expert_id in enumerate(topk_idx.view(-1).tolist()):
        slot = expert_slot[expert_id]
        if slot < max_m_val:
            out_ref[expert_id, slot, :] = out[i]
            expert_slot[expert_id] += 1
        else:
            raise IndexError(
                f"Expert {expert_id} exceeded max slots ({max_m_val}). "
                "Increase max_m or check masked_m."
            )

    return out_ref
```
**EN:** This helper function implements the shared logic for grouped gemm ref. It keeps the surrounding module logic factored into a reusable unit. Only the key portions are shown here because the block is large.
**CN:** 该辅助函数实现了 grouped gemm ref 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。 由于该代码块较大，这里只展示关键片段。

### Function `flashinfer_cutedsl_grouped_gemm_nt_masked` (lines 311-367)
```python
def flashinfer_cutedsl_grouped_gemm_nt_masked(
    hidden_states: torch.Tensor,  # 3d
    input_global_scale: torch.Tensor,  # (l,)
    weights: torch.Tensor,
    w_global_scale: torch.Tensor,  # (l,)
    masked_m: torch.Tensor,
):
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
        torch.full((num_experts,), n, device=weights.device, dtype=torch.int32),
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

    cutedsl_gmm_masked(
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
**EN:** This helper function implements the shared logic for flashinfer cutedsl grouped gemm nt masked. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 flashinfer cutedsl grouped gemm nt masked 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_flashinfer_cutedsl_moe_masked` (lines 370-523)
```python
@pytest.mark.parametrize("bs, hidden_dim, inter_dim", [(2, 128, 256), (16, 128, 512)])
@pytest.mark.parametrize("topk", [1, 2, 4])
@torch.inference_mode()
def test_flashinfer_cutedsl_moe_masked(
    bs: int, hidden_dim: int, inter_dim: int, topk: int
):
    torch.manual_seed(42)
    device = "cuda"
    num_experts = 8
    hidden_states = (
        torch.randn(bs, hidden_dim, dtype=torch.bfloat16, device=device) / 5.0
    )
    w1 = (
        torch.randn(
            num_experts, 2 * inter_dim, hidden_dim, dtype=torch.bfloat16, device=device
        )
        / 10.0
    )
    w2 = (
        torch.randn(
            num_experts, hidden_dim, inter_dim, dtype=torch.bfloat16, device=device
        )
        / 10.0
    )
    router_logits = torch.randn(bs, num_experts, dtype=torch.float32)

    hidden_states_expanded = (
        hidden_states.view(bs, -1, hidden_dim)
        .repeat(1, topk, 1)
        .reshape(-1, hidden_dim)
    )
    hidden_states_3d, masked_m, topk_idx, routing_weights = prepare_inputs(
        hidden_states_expanded, router_logits, num_experts, topk
    )

    w1_amax = w1.abs().amax(dim=(1, 2)).to(torch.float32).to(w1.device)
    w2_amax = w2.abs().amax(dim=(1, 2)).to(torch.float32).to(w2.device)
    input_global_scale = torch.ones(
        (num_experts,), dtype=torch.float32, device=hidden_states.device
    )
# ... excerpt ...
            w1_blockscale_sliced,
            w1_global_scale[idx],
            dtype=w1.dtype,
            device=w1.device,
            block_size=16,
        )
        w2_d[idx] = dequantize_nvfp4_to_dtype(
            w2_fp4_sliced,
            w2_blockscale_sliced,
            w2_global_scale[idx],
            dtype=w2.dtype,
            device=w2.device,
            block_size=16,
        )

    ref_output = torch_moe_nvfp4(
        a_in_dtype,
        w1_d,
        w2_d,
        topk,
        routing_weights.to(a_in_dtype.device),
        topk_idx.to(a_in_dtype.device),
    )
    out_weighted = torch.zeros_like(ref_output, device=out.device, dtype=out.dtype)

    positions = torch.nonzero(masked_m[topk_idx], as_tuple=False)
    rows, cols = positions[:, 0], positions[:, 1]
    experts = topk_idx[rows, cols]
    for i in range(num_experts):
        mask = experts == i
        if mask.any():
            idx = torch.nonzero(mask, as_tuple=False).squeeze(-1)
            r, c = rows[idx], cols[idx]
            out_weighted[r] += out[i, : len(r), :] * routing_weights[r, c].to(
                out.device
            ).unsqueeze(-1)
    torch.testing.assert_close(
        out_weighted.cpu(), ref_output.cpu(), atol=2e-1, rtol=2e-1
    )
```
**EN:** This pytest case verifies flashinfer cutedsl MoE masked. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as bs, hidden_dim, inter_dim, topk. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 flashinfer cutedsl MoE masked 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 bs、hidden_dim、inter_dim、topk 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

### Function `test_grouped_gemm_nt_masked` (lines 526-583)
```python
@pytest.mark.parametrize(
    "bs, hidden_dim, inter_dim, topk", [(2, 128, 256, 2), (16, 128, 512, 5)]
)
@torch.inference_mode()
def test_grouped_gemm_nt_masked(
    bs: int, hidden_dim: int, inter_dim: int, topk: int
) -> None:
    torch.manual_seed(42)
    B = bs
    D = hidden_dim
    N = inter_dim
    # CuteDSL group gemm has issue when not all experts are active.
    # i.e. masked = [2, 3, 0, 0, 1] where the 2nd and 3rd experts are inactive
    # see https://github.com/flashinfer-ai/flashinfer/issues/1856
    num_experts = bs
    hidden_states = torch.randn(B, D, dtype=torch.bfloat16, device="cuda")
    weights = torch.randn(num_experts, N, D, dtype=torch.bfloat16, device="cuda")
    router_logits = torch.randn(B, num_experts, dtype=torch.float32)

    hidden_states_expanded = (
        hidden_states.view(B, -1, D).repeat(1, topk, 1).reshape(-1, D)
    )
    hidden_states_3d, masked_m, topk_idx, _ = prepare_inputs(
        hidden_states_expanded, router_logits, num_experts, topk
    )

    a_amax = (
        hidden_states_3d.abs()
        .amax(dim=(1, 2))
        .to(torch.float32)
        .to(hidden_states.device)
    )
    b_amax = weights.abs().amax(dim=(1, 2)).to(torch.float32).to(weights.device)
    a_gs = FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX / a_amax
    b_gs = FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX / b_amax
    out_flashinfer = flashinfer_cutedsl_grouped_gemm_nt_masked(
        hidden_states_3d.to(hidden_states.device), a_gs, weights, b_gs, masked_m
    )
    # reference
    out_ref = grouped_gemm_ref(
        hidden_states_expanded=hidden_states_expanded,
        hidden_states_3d=hidden_states_3d,
        weights=weights,
        topk_idx=topk_idx,
        masked_m=masked_m,
        B=B,
        topk=topk,
        num_experts=num_experts,
    )
    # Note: just to compare the masked position due to cutedsl may write nan
    # into unmasked position.
    for i in range(num_experts):
        torch.testing.assert_close(
            out_flashinfer.permute(2, 0, 1)[i, : masked_m[i]],
            out_ref.to(out_flashinfer.device)[i, : masked_m[i]],
            atol=1e-1,
            rtol=1e-1,
        )
```
**EN:** This pytest case verifies grouped gemm nt masked. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as bs, hidden_dim, inter_dim, topk. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 grouped gemm nt masked 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 bs、hidden_dim、inter_dim、topk 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Top-level block starting at line 586 (lines 586-588)
```python
if __name__ == "__main__":
    test_flashinfer_cutedsl_moe_masked(16, 128, 512, 4)
    test_grouped_gemm_nt_masked(16, 128, 512, 4)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `vllm.platforms -> current_platform`
- `torch`
- `flashinfer -> fp4_quantize`
- `torch.nn -> functional`
- `vllm.model_executor.layers.activation -> SiluAndMul`
- `vllm.model_executor.layers.fused_moe.experts.flashinfer_cutedsl_batched_moe -> flashinfer_cutedsl_moe_masked`
- `vllm.utils.flashinfer -> flashinfer_cutedsl_grouped_gemm_nt_masked`
- `vllm.utils.flashinfer -> scaled_fp4_grouped_quantize`
