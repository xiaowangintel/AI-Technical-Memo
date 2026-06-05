# test_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/attention/test_attention.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / attention / test_attention, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / attention / test_attention 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-15)
```python
import random

import pytest
import torch

from tests.kernels.allclose_default import get_default_atol, get_default_rtol
from tests.kernels.utils import opcheck
from vllm import _custom_ops as ops
from vllm.model_executor.layers.attention import Attention, MMEncoderAttention
from vllm.platforms import current_platform
from vllm.utils.mem_utils import get_max_shared_memory_bytes
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as random, pytest, torch; shared test helpers from tests.kernels.allclose_default, tests.kernels.utils; and vLLM components like vllm, vllm.model_executor.layers.attention, vllm.platforms, vllm.utils.mem_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 random、pytest、torch；共享测试辅助模块，例如 tests.kernels.allclose_default、tests.kernels.utils；vLLM 内部组件，例如 vllm、vllm.model_executor.layers.attention、vllm.platforms、vllm.utils.mem_utils。

### Constants and module state (lines 17-41)
```python
FLOAT32_BYTES = torch.finfo(torch.float).bits // 8
# This will change depending on the compute capability.
# - 512 as a buffer
MAX_SEQ_LEN = get_max_shared_memory_bytes() // FLOAT32_BYTES - 512
# There may not be enough gpu memory due to large NUM_BLOCKS.
# Reduce NUM_BLOCKS when it happens.
NUM_BLOCKS = 4321  # Arbitrary values for testing
PARTITION_SIZE = 512
PARTITION_SIZE_ROCM = 256
DTYPES = [torch.bfloat16]
NUM_GEN_SEQS = [7]  # Arbitrary values for testing
NUM_PREFILL_SEQS = [3]  # Arbitrary values for testing
NUM_HEADS = [(40, 40), (64, 8)]  # Arbitrary values for testing

# This should be sync with get_supported_head_sizes() in
# vllm.v1.attention.ops.paged_attn.PagedAttention
HEAD_SIZES = [32, 80, 128, 256]

BLOCK_SIZES = [16, 32]
USE_ALIBI = [False, True]
KV_CACHE_DTYPE = ["auto", "fp8"]
SEEDS = [0]
CUDA_DEVICES = [
    f"cuda:{i}" for i in range(1 if torch.accelerator.device_count() == 1 else 2)
]
```
**EN:** This block centralizes shared constants and parameter grids, including FLOAT32_BYTES, MAX_SEQ_LEN, NUM_BLOCKS, PARTITION_SIZE, PARTITION_SIZE_ROCM. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 FLOAT32_BYTES、MAX_SEQ_LEN、NUM_BLOCKS、PARTITION_SIZE、PARTITION_SIZE_ROCM。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `ref_masked_attention` (lines 44-56)
```python
def ref_masked_attention(
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    scale: float,
    attn_mask: torch.Tensor | None = None,
) -> torch.Tensor:
    attn_weights = scale * torch.einsum("qhd,khd->hqk", query, key).float()
    if attn_mask is not None:
        attn_weights = attn_weights + attn_mask.float()
    attn_weights = torch.softmax(attn_weights, dim=-1).to(value.dtype)
    out = torch.einsum("hqk,khd->qhd", attn_weights, value)
    return out
```
**EN:** This helper acts as a reference implementation for masked attention. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数为 masked attention 提供参考实现。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `ref_single_query_cached_kv_attention` (lines 59-111)
```python
def ref_single_query_cached_kv_attention(
    output: torch.Tensor,
    query: torch.Tensor,
    num_queries_per_kv: int,
    key_cache: torch.Tensor,
    value_cache: torch.Tensor,
    block_tables: torch.Tensor,
    seq_lens: torch.Tensor,
    scale: float,
    alibi_slopes: torch.Tensor | None,
) -> None:
    num_query_heads = query.shape[1]
    num_kv_heads = value_cache.shape[1]
    head_size = value_cache.shape[2]
    block_size = value_cache.shape[3]
    num_seqs = query.shape[0]

    block_tables_lst = block_tables.cpu().tolist()
    seq_lens_lst = seq_lens.cpu().tolist()
    for i in range(num_seqs):
        q = query[i].unsqueeze(0)
        block_table = block_tables_lst[i]
        seq_len = int(seq_lens_lst[i])

        keys_lst: list[torch.Tensor] = []
        values_lst: list[torch.Tensor] = []
        for j in range(seq_len):
            block_number = int(block_table[j // block_size])
            block_offset = j % block_size

            k = key_cache[block_number, :, :, block_offset, :]
            k = k.reshape(num_kv_heads, head_size)
            keys_lst.append(k)

            v = value_cache[block_number, :, :, block_offset]
            values_lst.append(v)
        keys = torch.stack(keys_lst, dim=0)
        values = torch.stack(values_lst, dim=0)
        if num_queries_per_kv > 1:
            # Handle MQA and GQA
            keys = torch.repeat_interleave(keys, num_queries_per_kv, dim=1)
            values = torch.repeat_interleave(values, num_queries_per_kv, dim=1)

        alibi_bias = None
        if alibi_slopes is not None:
            # Create the ALiBi bias used in the paged attention kernel.
            position_ids = torch.arange(seq_len).int()
            alibi_bias = (position_ids - seq_len + 1).float()
            alibi_bias = alibi_slopes.view(-1, 1, 1) * alibi_bias.view(1, 1, -1)

        out = ref_masked_attention(q, keys, values, scale, alibi_bias)
        out = out.view(num_query_heads, head_size)
        output[i].copy_(out, non_blocking=True)
```
**EN:** This helper acts as a reference implementation for single query cached KV attention. a simpler reference path is used to validate the optimized kernel path.
**CN:** 该辅助函数为 single query cached KV attention 提供参考实现。 它会使用更直接的参考路径来验证优化后的内核路径。

### Function `test_paged_attention` (lines 114-405)
```python
@pytest.mark.parametrize(
    "version", ["v1", "v2"] if not current_platform.is_rocm() else ["v1", "v2", "rocm"]
)
@pytest.mark.parametrize("num_seqs", NUM_GEN_SEQS)
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("use_alibi", USE_ALIBI)
@pytest.mark.parametrize("block_size", BLOCK_SIZES)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("kv_cache_dtype", KV_CACHE_DTYPE)
@pytest.mark.parametrize("seed", SEEDS)
@pytest.mark.parametrize("device", CUDA_DEVICES)
def test_paged_attention(
    kv_cache_factory,
    version: str,
    num_seqs: int,
    num_heads: tuple[int, int],
    head_size: int,
    use_alibi: bool,
    block_size: int,
    dtype: torch.dtype,
    kv_cache_dtype: str,
    seed: int,
    device: str,
) -> None:
    if (kv_cache_dtype == "fp8" and head_size % 16) or (
        version == "rocm" and head_size not in (64, 128)
    ):
        pytest.skip()

    if (
        version == "rocm"
        and current_platform.is_navi()
        and (
            kv_cache_dtype == "fp8" or head_size != 128 or block_size != 16 or use_alibi
        )
    ):
        pytest.skip()

    global PARTITION_SIZE
# ... excerpt ...
        key_cache_shape = (NUM_BLOCKS, num_kv_heads, head_size // x, block_size, x)
        dequantized_key_cache = torch.empty(
            size=key_cache_shape, dtype=dtype, device=device
        )
        ops.convert_fp8(dequantized_key_cache, key_cache)
        key_cache = dequantized_key_cache

        value_cache_shape = value_cache.shape
        dequantized_value_cache = torch.empty(
            size=value_cache_shape, dtype=dtype, device=device
        )
        ops.convert_fp8(dequantized_value_cache, value_cache)
        value_cache = dequantized_value_cache

    ref_output = torch.empty_like(query)
    ref_single_query_cached_kv_attention(
        ref_output,
        query,
        num_queries_per_kv,
        key_cache,
        value_cache,
        block_tables,
        seq_lens,
        scale,
        alibi_slopes,
    )

    # NOTE(woosuk): Due to the kernel-level differences in the two
    # implementations, there is a small numerical difference in the two
    # outputs. Thus, we use a relaxed tolerance for the test.
    atol = get_default_atol(output) if current_platform.is_rocm() else 1e-3
    rtol = get_default_rtol(output) if current_platform.is_rocm() else 1e-5

    # NOTE(zhaoyang): FP8 KV Cache will introduce quantization error,
    # so we use a relaxed tolerance for the test.
    atol, rtol = 1e-3, 1e-5
    if kv_cache_dtype == "fp8":
        atol, rtol = 1e-2, 1e-5
    torch.testing.assert_close(output, ref_output, atol=atol, rtol=rtol)
```
**EN:** This pytest case verifies paged attention. It is parameterized across 10 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as kv_cache_factory, version, num_seqs, num_heads. unsupported hardware, backend, or configuration combinations are skipped early. it also validates that the custom operator entry point is wired correctly. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 paged attention 的行为。 它通过 10 组参数化输入覆盖多种场景；它会使用诸如 kv_cache_factory、version、num_seqs、num_heads 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；它还会校验自定义算子入口是否正确接线。 由于该代码块较大，这里只展示关键片段。

### Function `ref_multi_query_kv_attention` (lines 408-445)
```python
def ref_multi_query_kv_attention(
    cu_seq_lens: list[int],
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    scale: float,
    alibi_bias: list[torch.Tensor] | None,
    dtype: torch.dtype,
) -> torch.Tensor:
    num_seqs = len(cu_seq_lens) - 1
    ref_outputs: list[torch.Tensor] = []
    if alibi_bias:
        assert len(alibi_bias) == num_seqs
    for i in range(num_seqs):
        start_idx = cu_seq_lens[i]
        end_idx = cu_seq_lens[i + 1]
        seq_len = end_idx - start_idx

        # Create attention mask. ALiBi already includes a tril causal mask.
        if alibi_bias:
            attn_mask = alibi_bias[i]
        else:
            attn_mask = torch.triu(
                torch.ones(seq_len, seq_len, dtype=dtype), diagonal=1
            )
            attn_mask = attn_mask * torch.finfo(dtype).min
            attn_mask = attn_mask.to(dtype=dtype)

        ref_output = ref_masked_attention(
            query[start_idx:end_idx],
            key[start_idx:end_idx],
            value[start_idx:end_idx],
            scale,
            attn_mask=attn_mask,
        )
        ref_outputs.append(ref_output)

    return torch.cat(ref_outputs, dim=0)
```
**EN:** This helper acts as a reference implementation for multi query KV attention. assertions at the end lock in the intended behavior or graph shape. a simpler reference path is used to validate the optimized kernel path.
**CN:** 该辅助函数为 multi query KV attention 提供参考实现。 结尾处的断言会固定预期行为或计算图形态；它会使用更直接的参考路径来验证优化后的内核路径。

### Function `test_num_heads_not_divisible_by_num_kv_heads` (lines 448-460)
```python
@pytest.mark.parametrize("attention_cls", [Attention, MMEncoderAttention])
def test_num_heads_not_divisible_by_num_kv_heads(attention_cls: type) -> None:
    head_size = 64
    scale = float(1.0 / (head_size**0.5))
    num_heads = 16
    num_kv_heads = 5
    with pytest.raises(AssertionError):
        _ = attention_cls(
            num_heads=num_heads,
            head_size=head_size,
            scale=scale,
            num_kv_heads=num_kv_heads,
        )
```
**EN:** This pytest case verifies num heads not divisible by num KV heads. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as attention_cls. the expected failure path is asserted explicitly.
**CN:** 该 pytest 用例验证 num heads not divisible by num KV heads 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 attention_cls 等 fixture 或输入；代码会显式断言预期的失败路径。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `random`
- `pytest`
- `torch`
- `tests.kernels.allclose_default -> get_default_atol, get_default_rtol`
- `tests.kernels.utils -> opcheck`
- `vllm -> _custom_ops`
- `vllm.model_executor.layers.attention -> Attention, MMEncoderAttention`
- `vllm.platforms -> current_platform`
- `vllm.utils.mem_utils -> get_max_shared_memory_bytes`
- `vllm.utils.torch_utils -> set_random_seed`
