# test_triton_prefill_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/attention/test_triton_prefill_attention.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / attention / test_triton_prefill_attention, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / attention / test_triton_prefill_attention 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-9)
```python
import pytest
import torch
import torch.nn.functional as F

from vllm.platforms import current_platform
from vllm.v1.attention.ops.triton_prefill_attention import context_attention_fwd
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch, torch.nn.functional; and vLLM components like vllm.platforms, vllm.v1.attention.ops.triton_prefill_attention.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch、torch.nn.functional；vLLM 内部组件，例如 vllm.platforms、vllm.v1.attention.ops.triton_prefill_attention。

### Constants and module state (lines 11-11)
```python
DEVICE_TYPE = current_platform.device_type
```
**EN:** This block centralizes shared constants and parameter grids, including DEVICE_TYPE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 DEVICE_TYPE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `ref_masked_attention` (lines 14-75)
```python
def ref_masked_attention(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    is_causal: bool = True,
    sliding_window_q: int | None = None,
    sliding_window_k: int | None = None,
) -> torch.Tensor:
    """Reference implementation using PyTorch SDPA."""
    # q, k, v: [total_tokens, num_heads, head_dim]
    # SDPA expects [batch, num_heads, seq_len, head_dim]

    total_tokens = q.shape[0]

    # Add batch dimension and transpose
    q = q.unsqueeze(0).transpose(1, 2)  # [1, num_heads, total_tokens, head_dim]
    k = k.unsqueeze(0).transpose(1, 2)  # [1, num_heads, total_tokens, head_dim]
    v = v.unsqueeze(0).transpose(1, 2)  # [1, num_heads, total_tokens, head_dim]

    # Create attention mask if needed
    attn_mask = None
    use_causal = is_causal

    # If we have sliding window or need custom masking, create explicit mask
    sliding_window_q = sliding_window_q if sliding_window_q is not None else 0
    sliding_window_k = sliding_window_k if sliding_window_k is not None else 0
    if (sliding_window_q > 0) or (sliding_window_k > 0):
        # Position indices
        pos_q = torch.arange(total_tokens, device=q.device).unsqueeze(1)
        pos_k = torch.arange(total_tokens, device=q.device).unsqueeze(0)

        # Start with valid mask (False = no masking)
        mask = torch.ones(
            (total_tokens, total_tokens), dtype=torch.bool, device=q.device
        )

        # Apply causal mask
        if is_causal:
            mask = mask & (pos_q >= pos_k)

        # Apply sliding window masks
        sliding_window_mask = torch.ones_like(mask)
        if sliding_window_q > 0:
            sliding_window_mask &= pos_q - pos_k <= sliding_window_q

        if sliding_window_k > 0:
            sliding_window_mask &= pos_k - pos_q <= sliding_window_k

        mask = mask & sliding_window_mask

        attn_mask = torch.where(mask, 0.0, float("-inf")).to(q.dtype)
        use_causal = False  # Don't use is_causal when providing explicit mask

    # Use SDPA
    output = F.scaled_dot_product_attention(
        q, k, v, attn_mask=attn_mask, is_causal=use_causal, dropout_p=0.0
    )

    # Convert back to original shape: [total_tokens, num_heads, head_dim]
    output = output.transpose(1, 2).squeeze(0)

    return output
```
**EN:** This helper acts as a reference implementation for masked attention. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数为 masked attention 提供参考实现。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_context_attention` (lines 78-153)
```python
@pytest.mark.parametrize("B", [5])
@pytest.mark.parametrize("max_seq_len", [1024])
@pytest.mark.parametrize("H_Q", [32])
@pytest.mark.parametrize("H_KV", [32, 8])
@pytest.mark.parametrize("D", [128])
@pytest.mark.parametrize("is_causal", [True, False])
@pytest.mark.parametrize("dtype", [torch.float32, torch.bfloat16])
def test_context_attention(
    B: int,
    max_seq_len: int,
    H_Q: int,
    H_KV: int,
    D: int,
    is_causal: bool,
    dtype: torch.dtype,
):
    """Test basic context attention without sliding window."""
    torch.manual_seed(42)

    # Generate random sequence lengths for each batch
    seq_lens = torch.randint(
        max_seq_len // 2, max_seq_len + 1, (B,), device=DEVICE_TYPE
    )
    total_tokens = seq_lens.sum().item()

    # Create batch start locations
    b_start_loc = torch.zeros(B, dtype=torch.int32, device=DEVICE_TYPE)
    b_start_loc[1:] = torch.cumsum(seq_lens[:-1], dim=0)

    # Create input tensors
    q = torch.randn(total_tokens, H_Q, D, dtype=dtype, device=DEVICE_TYPE)
    k = torch.randn(total_tokens, H_KV, D, dtype=dtype, device=DEVICE_TYPE)
    v = torch.randn(total_tokens, H_KV, D, dtype=dtype, device=DEVICE_TYPE)
    o = torch.zeros_like(q)

    # Call Triton kernel
    context_attention_fwd(
        q,
        k,
        v,
        o,
        b_start_loc,
        seq_lens,
        max_seq_len,
        is_causal=is_causal,
        sliding_window_q=None,
        sliding_window_k=None,
    )

    # Compute reference output for each sequence in batch
    o_ref = torch.zeros_like(q)
    for i in range(B):
        start = b_start_loc[i].item()
        end = start + seq_lens[i].item()

        q_seq = q[start:end]
        k_seq = k[start:end]
        v_seq = v[start:end]

        # Expand KV heads if using GQA
        if H_Q != H_KV:
            kv_group_num = H_Q // H_KV
            k_seq = k_seq.repeat_interleave(kv_group_num, dim=1)
            v_seq = v_seq.repeat_interleave(kv_group_num, dim=1)

        o_ref[start:end] = ref_masked_attention(
            q_seq,
            k_seq,
            v_seq,
            is_causal=is_causal,
            sliding_window_q=None,
            sliding_window_k=None,
        )

    # Compare outputs
    torch.testing.assert_close(o, o_ref, rtol=1e-2, atol=1e-2)
```
**EN:** This pytest case verifies context attention. It is parameterized across 7 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as B, max_seq_len, H_Q, H_KV. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 context attention 的行为。 它通过 7 组参数化输入覆盖多种场景；它会使用诸如 B、max_seq_len、H_Q、H_KV 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

### Function `test_context_attention_sliding_window` (lines 156-232)
```python
@pytest.mark.parametrize("B", [4])
@pytest.mark.parametrize("max_seq_len", [1024])
@pytest.mark.parametrize("H_Q", [32])
@pytest.mark.parametrize("H_KV", [32, 8])
@pytest.mark.parametrize("D", [128])
@pytest.mark.parametrize("sliding_window", [(32, 32), (32, 0), (0, 32)])
@pytest.mark.parametrize("dtype", [torch.float32, torch.bfloat16])
def test_context_attention_sliding_window(
    B: int,
    max_seq_len: int,
    H_Q: int,
    H_KV: int,
    D: int,
    sliding_window: tuple[int, int],
    dtype: torch.dtype,
):
    sliding_window_q, sliding_window_k = sliding_window
    """Test context attention with sliding window."""
    torch.manual_seed(42)

    # Generate random sequence lengths for each batch
    seq_lens = torch.randint(
        max_seq_len // 2, max_seq_len + 1, (B,), device=DEVICE_TYPE
    )
    total_tokens = seq_lens.sum().item()

    # Create batch start locations
    b_start_loc = torch.zeros(B, dtype=torch.int32, device=DEVICE_TYPE)
    b_start_loc[1:] = torch.cumsum(seq_lens[:-1], dim=0)

    # Create input tensors
    q = torch.randn(total_tokens, H_Q, D, dtype=dtype, device=DEVICE_TYPE)
    k = torch.randn(total_tokens, H_KV, D, dtype=dtype, device=DEVICE_TYPE)
    v = torch.randn(total_tokens, H_KV, D, dtype=dtype, device=DEVICE_TYPE)
    o = torch.zeros_like(q)

    # Call Triton kernel
    context_attention_fwd(
        q,
        k,
        v,
        o,
        b_start_loc,
        seq_lens,
        max_seq_len,
        is_causal=False,
        sliding_window_q=sliding_window_q,
        sliding_window_k=sliding_window_k,
    )

    # Compute reference output for each sequence in batch
    o_ref = torch.zeros_like(q)
    for i in range(B):
        start = b_start_loc[i].item()
        end = start + seq_lens[i].item()

        q_seq = q[start:end]
        k_seq = k[start:end]
        v_seq = v[start:end]

        # Expand KV heads if using GQA
        if H_Q != H_KV:
            kv_group_num = H_Q // H_KV
            k_seq = k_seq.repeat_interleave(kv_group_num, dim=1)
            v_seq = v_seq.repeat_interleave(kv_group_num, dim=1)

        o_ref[start:end] = ref_masked_attention(
            q_seq,
            k_seq,
            v_seq,
            is_causal=False,
            sliding_window_q=sliding_window_q if sliding_window_q > 0 else None,
            sliding_window_k=sliding_window_k if sliding_window_k > 0 else None,
        )

    # Compare outputs
    torch.testing.assert_close(o, o_ref, rtol=2e-2, atol=2e-2)
```
**EN:** This pytest case verifies context attention sliding window. It is parameterized across 7 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as B, max_seq_len, H_Q, H_KV. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 context attention sliding window 的行为。 它通过 7 组参数化输入覆盖多种场景；它会使用诸如 B、max_seq_len、H_Q、H_KV 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `torch.nn.functional`
- `vllm.platforms -> current_platform`
- `vllm.v1.attention.ops.triton_prefill_attention -> context_attention_fwd`
