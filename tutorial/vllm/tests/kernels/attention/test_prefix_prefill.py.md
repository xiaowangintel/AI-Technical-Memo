# test_prefix_prefill.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/attention/test_prefix_prefill.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / attention / test_prefix_prefill, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / attention / test_prefix_prefill 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-18)
```python
import math
import random
import time
from collections.abc import Callable

import pytest
import torch
import torch.nn.functional as F

from vllm.platforms import current_platform
from vllm.utils.torch_utils import STR_DTYPE_TO_TORCH_DTYPE, set_random_seed
from vllm.v1.attention.ops.chunked_prefill_paged_decode import (
    chunked_prefill_paged_decode,
)
from vllm.v1.attention.ops.prefix_prefill import context_attention_fwd
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as math, random, time, collections.abc; and vLLM components like vllm.platforms, vllm.utils.torch_utils, vllm.v1.attention.ops.chunked_prefill_paged_decode, vllm.v1.attention.ops.prefix_prefill.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 math、random、time、collections.abc；vLLM 内部组件，例如 vllm.platforms、vllm.utils.torch_utils、vllm.v1.attention.ops.chunked_prefill_paged_decode、vllm.v1.attention.ops.prefix_prefill。

### Constants and module state (lines 20-30)
```python
NUM_HEADS = [64]
NUM_QUERIES_PER_KV = [1, 64]
HEAD_SIZES = [24, 128]
DTYPES = [torch.float16]
CUDA_DEVICES = [
    f"cuda:{i}" for i in range(1 if torch.accelerator.device_count() == 1 else 2)
]
SLIDING_WINDOW = [0, 16, 2048]
KV_CACHE_DTYPES = ["auto", "fp8", "fp8_e5m2"]

OPS = [chunked_prefill_paged_decode, context_attention_fwd]
```
**EN:** This block centralizes shared constants and parameter grids, including NUM_HEADS, NUM_QUERIES_PER_KV, HEAD_SIZES, DTYPES, CUDA_DEVICES. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 NUM_HEADS、NUM_QUERIES_PER_KV、HEAD_SIZES、DTYPES、CUDA_DEVICES。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `create_causal_attention_mask_for_sdpa` (lines 33-70)
```python
def create_causal_attention_mask_for_sdpa(
    query_lens: list[int],
    seq_lens: list[int],
    sliding_window: int = 0,
    device: torch.device = None,
    dtype: torch.dtype = None,
) -> torch.Tensor:
    total_queries = sum(query_lens)
    total_keys = sum(seq_lens)

    # Create a mask filled with -inf
    mask = torch.full(
        (total_queries, total_keys), float("-inf"), device=device, dtype=dtype
    )

    query_start = 0
    key_start = 0

    for query_len, seq_len in zip(query_lens, seq_lens):
        query_end = query_start + query_len
        key_end = key_start + seq_len
        q_indices = torch.arange(query_len, device=device)
        k_indices = torch.arange(seq_len, device=device)
        q_pos_in_seq = seq_len - query_len + q_indices

        valid_mask = k_indices[None, :] <= q_pos_in_seq[:, None]

        if sliding_window > 0:
            valid_mask &= k_indices[None, :] >= (
                q_pos_in_seq[:, None] - sliding_window + 1
            )

        mask[query_start:query_end, key_start:key_end][valid_mask] = 0.0

        query_start = query_end
        key_start = key_end

    return mask
```
**EN:** This helper function implements the shared logic for causal attention mask for sdpa. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 causal attention mask for sdpa 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `create_alibi_causal_mask` (lines 73-98)
```python
def create_alibi_causal_mask(
    query_len: int,
    seq_len: int,
    alibi_slopes: torch.Tensor,
    device: torch.device,
    dtype: torch.dtype,
) -> torch.Tensor:
    query_pos = torch.arange(
        seq_len - query_len, seq_len, device=device, dtype=torch.float32
    )
    key_pos = torch.arange(seq_len, device=device, dtype=torch.float32)

    rel_pos = key_pos[None, :] - query_pos[:, None]

    # Apply ALiBi slopes: [num_heads, query_len, seq_len]
    alibi_bias = alibi_slopes[:, None, None] * rel_pos[None, :, :]
    alibi_bias = alibi_bias.to(dtype)

    # Apply causal mask: prevent attending to future positions
    # causal_mask[i, j] = True if key_pos[j] <= query_pos[i]
    causal_mask = key_pos[None, :] <= query_pos[:, None]
    alibi_bias = alibi_bias.masked_fill(~causal_mask[None, :, :], float("-inf"))

    # Add batch dimension: [1, num_heads, query_len, seq_len]
    # SDPA expects batch dimension even for single sequences
    return alibi_bias.unsqueeze(0)
```
**EN:** This helper function implements the shared logic for ALiBi causal mask. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 ALiBi causal mask 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_contexted_kv_attention` (lines 101-321)
```python
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("num_queries_per_kv", NUM_QUERIES_PER_KV)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("kv_cache_dtype", KV_CACHE_DTYPES)
@pytest.mark.parametrize("device", CUDA_DEVICES)
@pytest.mark.parametrize("sliding_window", SLIDING_WINDOW)
@pytest.mark.parametrize("op", OPS)
@torch.inference_mode()
def test_contexted_kv_attention(
    num_heads: int,
    num_queries_per_kv: int,
    head_size: int,
    sliding_window: int,
    dtype: torch.dtype,
    kv_cache_dtype: str,
    device: str,
    op: Callable,
    block_size: int = 32,
) -> None:
    if "fp8" in kv_cache_dtype and not current_platform.has_device_capability(89):
        pytest.skip(
            "Triton limitation: fp8e4nv data type is not supported on CUDA arch < 89"
        )

    if (
        current_platform.is_rocm()
        and op is chunked_prefill_paged_decode
        and kv_cache_dtype == "fp8_e5m2"
    ):
        pytest.skip("ROCm custom paged attention does not support fp8_e5m2 KV cache")

    set_random_seed(0)
    torch.set_default_device(device)

    # Need this, otherwise when we capture the graph the process
    # for GPU 1 would run on both GPU0 and GPU1 and things would hang
    #
    # see also similar issue: https://github.com/Dao-AILab/flash-attention/issues/523
    torch.accelerator.set_device_index(device)
# ... excerpt ...

    value_sdpa = value[:, :, None, :].expand(
        value.shape[0], num_kv_heads, num_queries_per_kv, value.shape[-1]
    )
    value_sdpa = value_sdpa.permute(1, 2, 0, 3).reshape(
        1, num_heads, sum(seq_lens), head_size
    )

    attn_mask = create_causal_attention_mask_for_sdpa(
        query_lens, seq_lens, sliding_window, device=device, dtype=dtype
    )

    output_ref = F.scaled_dot_product_attention(
        query_sdpa,
        key_sdpa,
        value_sdpa,
        attn_mask=attn_mask,
        dropout_p=0.0,
        scale=scale,
    )
    torch.accelerator.synchronize()
    start_time = time.time()
    output_ref = F.scaled_dot_product_attention(
        query_sdpa,
        key_sdpa,
        value_sdpa,
        attn_mask=attn_mask,
        dropout_p=0.0,
        scale=scale,
    )
    torch.accelerator.synchronize()
    end_time = time.time()
    print(f"PyTorch SDPA Time: {(end_time - start_time) * 1000:.2f} ms")

    # Reshape output back to (num_tokens, num_heads, head_size)
    output_ref = output_ref.view(num_heads, num_tokens, head_size)
    output_ref = output_ref.permute(1, 0, 2).contiguous()
    atol = 1e-3 if "fp8" in kv_cache_dtype else 1e-4
    torch.testing.assert_close(output, output_ref, atol=atol, rtol=0)
```
**EN:** This pytest case verifies contexted KV attention. It is parameterized across 8 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_heads, num_queries_per_kv, head_size, sliding_window. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 contexted KV attention 的行为。 它通过 8 组参数化输入覆盖多种场景；它会使用诸如 num_heads、num_queries_per_kv、head_size、sliding_window 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

### Function `test_contexted_kv_attention_alibi` (lines 324-581)
```python
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("num_queries_per_kv", NUM_QUERIES_PER_KV)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("kv_cache_dtype", KV_CACHE_DTYPES)
@pytest.mark.parametrize("device", CUDA_DEVICES)
@pytest.mark.parametrize("op", OPS)
@torch.inference_mode()
def test_contexted_kv_attention_alibi(
    num_heads: int,
    num_queries_per_kv: int,
    head_size: int,
    dtype: torch.dtype,
    kv_cache_dtype: str,
    device: str,
    op: Callable,
    block_size: int = 32,
) -> None:
    if "fp8" in kv_cache_dtype and not current_platform.has_device_capability(89):
        pytest.skip(
            "Triton limitation: fp8e4nv data type is not supported on CUDA arch < 89"
        )

    if (
        current_platform.is_rocm()
        and op is chunked_prefill_paged_decode
        and kv_cache_dtype == "fp8_e5m2"
    ):
        pytest.skip("ROCm custom paged attention does not support fp8_e5m2 KV cache")

    set_random_seed(0)
    torch.set_default_device(device)

    # Need this, otherwise when we capture the graph the process
    # for GPU 1 would run on both GPU0 and GPU1 and things would hang
    #
    # see also similar issue: https://github.com/Dao-AILab/flash-attention/issues/523
    torch.accelerator.set_device_index(device)

    def _get_alibi_slopes(total_num_heads: int) -> torch.Tensor:
# ... excerpt ...
            q_sdpa.permute(1, 2, 0, 3)
            .reshape(1, num_heads, query_len, head_size)
            .contiguous()
        )

        k_sdpa = (
            k.permute(1, 2, 0, 3).reshape(1, num_heads, seq_len, head_size).contiguous()
        )
        v_sdpa = (
            v.permute(1, 2, 0, 3).reshape(1, num_heads, seq_len, head_size).contiguous()
        )

        # Create ALiBi causal mask for this sequence using utility function
        alibi_mask = create_alibi_causal_mask(
            query_len, seq_len, alibi_slopes, device, dtype
        )

        # Compute attention
        out = F.scaled_dot_product_attention(
            q_sdpa,
            k_sdpa,
            v_sdpa,
            attn_mask=alibi_mask,
            dropout_p=0.0,
            scale=scale,
        )

        # Reshape output back to [query_len, num_heads, head_size]
        out = out.view(num_heads, query_len, head_size).permute(1, 0, 2)
        output_ref[query_start:query_end].copy_(out)

        query_start = query_end
        key_start = key_end

    torch.accelerator.synchronize()
    end_time = time.time()
    print(f"PyTorch SDPA Time: {(end_time - start_time) * 1000:.2f} ms")
    atol = 1e-3 if "fp8" in kv_cache_dtype else 1e-6
    torch.testing.assert_close(output, output_ref, atol=atol, rtol=0)
```
**EN:** This pytest case verifies contexted KV attention ALiBi. It is parameterized across 7 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_heads, num_queries_per_kv, head_size, dtype. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 contexted KV attention ALiBi 的行为。 它通过 7 组参数化输入覆盖多种场景；它会使用诸如 num_heads、num_queries_per_kv、head_size、dtype 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

### Function `test_contexted_kv_attention_f32` (lines 591-620)
```python
@pytest.mark.optional
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("num_queries_per_kv", NUM_QUERIES_PER_KV)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("dtype", [torch.float32])
@pytest.mark.parametrize("kv_cache_dtype", KV_CACHE_DTYPES)
@pytest.mark.parametrize("device", CUDA_DEVICES)
@pytest.mark.parametrize("sliding_window", SLIDING_WINDOW)
@pytest.mark.parametrize("op", OPS)
@torch.inference_mode()
def test_contexted_kv_attention_f32(
    num_heads: int,
    num_queries_per_kv: int,
    head_size: int,
    sliding_window: int,
    dtype: torch.dtype,
    kv_cache_dtype: str,
    device: str,
    op: Callable,
) -> None:
    test_contexted_kv_attention(
        num_heads,
        num_queries_per_kv,
        head_size,
        sliding_window,
        dtype,
        kv_cache_dtype,
        device,
        op,
    )
```
**EN:** This pytest case verifies contexted KV attention f32. It is parameterized across 8 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_heads, num_queries_per_kv, head_size, sliding_window.
**CN:** 该 pytest 用例验证 contexted KV attention f32 的行为。 它通过 8 组参数化输入覆盖多种场景；它会使用诸如 num_heads、num_queries_per_kv、head_size、sliding_window 等 fixture 或输入。

### Function `test_contexted_kv_attention_alibi_f32` (lines 623-643)
```python
@pytest.mark.optional
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("num_queries_per_kv", NUM_QUERIES_PER_KV)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("dtype", [torch.float32])
@pytest.mark.parametrize("kv_cache_dtype", KV_CACHE_DTYPES)
@pytest.mark.parametrize("device", CUDA_DEVICES)
@pytest.mark.parametrize("op", OPS)
@torch.inference_mode()
def test_contexted_kv_attention_alibi_f32(
    num_heads: int,
    num_queries_per_kv: int,
    head_size: int,
    dtype: torch.dtype,
    kv_cache_dtype: str,
    device: str,
    op: Callable,
) -> None:
    test_contexted_kv_attention_alibi(
        num_heads, num_queries_per_kv, head_size, dtype, kv_cache_dtype, device, op
    )
```
**EN:** This pytest case verifies contexted KV attention ALiBi f32. It is parameterized across 7 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_heads, num_queries_per_kv, head_size, dtype.
**CN:** 该 pytest 用例验证 contexted KV attention ALiBi f32 的行为。 它通过 7 组参数化输入覆盖多种场景；它会使用诸如 num_heads、num_queries_per_kv、head_size、dtype 等 fixture 或输入。

### Function `test_qwen3_nonstandard_block_size` (lines 646-674)
```python
@pytest.mark.parametrize("head_size", [128])
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("device", CUDA_DEVICES)
@pytest.mark.parametrize("op", OPS)
@torch.inference_mode()
def test_qwen3_nonstandard_block_size(
    head_size: int,
    dtype: torch.dtype,
    device: str,
    op: Callable,
) -> None:
    """
    A separate test function specifically added
    for Qwen3-Next-80B (Block Size 544).
    """
    if not current_platform.is_rocm():
        pytest.skip("544 block size optimization is only for ROCm.")

    test_contexted_kv_attention(
        num_heads=64,
        num_queries_per_kv=1,
        head_size=head_size,
        block_size=544,
        sliding_window=0,
        dtype=dtype,
        kv_cache_dtype="auto",
        device=device,
        op=op,
    )
```
**EN:** This pytest case verifies qwen3 nonstandard block size. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as head_size, dtype, device, op. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 qwen3 nonstandard block size 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 head_size、dtype、device、op 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `math`
- `random`
- `time`
- `collections.abc -> Callable`
- `pytest`
- `torch`
- `torch.nn.functional`
- `vllm.platforms -> current_platform`
- `vllm.utils.torch_utils -> STR_DTYPE_TO_TORCH_DTYPE, set_random_seed`
- `vllm.v1.attention.ops.chunked_prefill_paged_decode -> chunked_prefill_paged_decode`
- `vllm.v1.attention.ops.prefix_prefill -> context_attention_fwd`
