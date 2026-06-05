# test_flashinfer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/attention/test_flashinfer.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / attention / test_flashinfer, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / attention / test_flashinfer 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 5-8)
```python
import pytest

from vllm.platforms import current_platform
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest; and vLLM components like vllm.platforms, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest；vLLM 内部组件，例如 vllm.platforms、vllm.utils.torch_utils。

### Top-level block starting at line 10 (lines 10-16)
```python
try:
    import flashinfer
except ImportError:
    if current_platform.is_rocm():
        pytest.skip(
            "flashinfer is not supported for vLLM on ROCm.", allow_module_level=True
        )
```
**EN:** This top-level `Try` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `Try` 代码块执行周边测试所依赖的辅助逻辑。

### Imports and shared setup (lines 18-18)
```python
import torch
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as torch.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 torch。

### Constants and module state (lines 20-26)
```python
NUM_HEADS = [(32, 8), (6, 1)]
HEAD_SIZES = [128, 256]
BLOCK_SIZES = [16, 32]
DTYPES = [torch.bfloat16]
NUM_BLOCKS = 32768  # Large enough to test overflow in index calculation.
SOFT_CAPS = [None, 30.0]
SLIDING_WINDOWS = [None, 64]
```
**EN:** This block centralizes shared constants and parameter grids, including NUM_HEADS, HEAD_SIZES, BLOCK_SIZES, DTYPES, NUM_BLOCKS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 NUM_HEADS、HEAD_SIZES、BLOCK_SIZES、DTYPES、NUM_BLOCKS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `ref_paged_attn` (lines 29-84)
```python
def ref_paged_attn(
    query: torch.Tensor,
    key_cache: torch.Tensor,
    value_cache: torch.Tensor,
    query_lens: list[int],
    kv_lens: list[int],
    block_tables: torch.Tensor,
    scale: float,
    sliding_window: int | None = None,
    soft_cap: float | None = None,
) -> torch.Tensor:
    num_seqs = len(query_lens)
    block_tables = block_tables.cpu().numpy()
    _, block_size, num_kv_heads, head_size = key_cache.shape

    outputs: list[torch.Tensor] = []
    start_idx = 0
    for i in range(num_seqs):
        query_len = query_lens[i]
        kv_len = kv_lens[i]
        q = query[start_idx : start_idx + query_len]
        q *= scale

        num_kv_blocks = (kv_len + block_size - 1) // block_size
        block_indices = block_tables[i, :num_kv_blocks]

        k = key_cache[block_indices].view(-1, num_kv_heads, head_size)
        k = k[:kv_len]
        v = value_cache[block_indices].view(-1, num_kv_heads, head_size)
        v = v[:kv_len]

        if q.shape[1] != k.shape[1]:
            k = torch.repeat_interleave(k, q.shape[1] // k.shape[1], dim=1)
            v = torch.repeat_interleave(v, q.shape[1] // v.shape[1], dim=1)
        attn = torch.einsum("qhd,khd->hqk", q, k).float()
        empty_mask = torch.ones(query_len, kv_len)
        mask = torch.triu(empty_mask, diagonal=kv_len - query_len + 1).bool()
        if sliding_window is not None:
            sliding_window_mask = (
                torch.triu(
                    empty_mask, diagonal=kv_len - (query_len + sliding_window) + 1
                )
                .bool()
                .logical_not()
            )
            mask |= sliding_window_mask
        if soft_cap is not None:
            attn = soft_cap * torch.tanh(attn / soft_cap)
        attn.masked_fill_(mask, float("-inf"))
        attn = torch.softmax(attn, dim=-1).to(v.dtype)
        out = torch.einsum("hqk,khd->qhd", attn, v)

        outputs.append(out)
        start_idx += query_len

    return torch.cat(outputs, dim=0)
```
**EN:** This helper acts as a reference implementation for paged attn. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数为 paged attn 提供参考实现。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_make_paged_kv_metadata` (lines 87-120)
```python
def _make_paged_kv_metadata(
    kv_lens: list[int],
    block_size: int,
    num_blocks: int,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor, torch.Tensor]:
    """Build paged-KV metadata tensors for fast_plan_decode tests.

    Returns:
        kv_indptr          – CPU int32, shape [num_seqs + 1]
        kv_indices         – CUDA int32, shape [total_blocks]
        kv_last_page_lens  – CPU int32, shape [num_seqs]
        block_tables       – CUDA int32, shape [num_seqs, max_blocks_per_seq]
    """
    num_seqs = len(kv_lens)
    max_blocks = (max(kv_lens) + block_size - 1) // block_size
    block_tables = torch.randint(
        0, num_blocks, (num_seqs, max_blocks), dtype=torch.int32, device="cuda"
    )

    indptr_list = [0]
    indices_list: list[int] = []
    last_lens_list: list[int] = []
    for i, seq_len in enumerate(kv_lens):
        n = (seq_len + block_size - 1) // block_size
        indices_list.extend(block_tables[i, :n].cpu().tolist())
        indptr_list.append(indptr_list[-1] + n)
        last_lens_list.append(seq_len % block_size or block_size)

    return (
        torch.tensor(indptr_list, dtype=torch.int32, device="cpu"),
        torch.tensor(indices_list, dtype=torch.int32, device="cuda"),
        torch.tensor(last_lens_list, dtype=torch.int32, device="cpu"),
        block_tables,
    )
```
**EN:** This helper function implements the shared logic for make paged KV metadata. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 make paged KV metadata 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_make_cg_decode_wrapper` (lines 123-146)
```python
def _make_cg_decode_wrapper(
    num_seqs: int,
    kv_indices_buffer: torch.Tensor,
    workspace_buffer: torch.Tensor,
    use_tensor_cores: bool = True,
) -> "flashinfer.BatchDecodeWithPagedKVCacheWrapper":
    """Create a cudagraph-enabled BatchDecodeWithPagedKVCacheWrapper.

    *kv_indices_buffer* is shared with the caller so that fast_plan_decode
    can avoid the device-to-device index copy on subsequent (cudagraph) calls.
    """
    return flashinfer.BatchDecodeWithPagedKVCacheWrapper(
        workspace_buffer,
        "NHD",
        use_cuda_graph=True,
        paged_kv_indptr_buffer=torch.zeros(
            num_seqs + 1, dtype=torch.int32, device="cuda"
        ),
        paged_kv_indices_buffer=kv_indices_buffer,
        paged_kv_last_page_len_buffer=torch.zeros(
            num_seqs, dtype=torch.int32, device="cuda"
        ),
        use_tensor_cores=use_tensor_cores,
    )
```
**EN:** This helper function implements the shared logic for make cg decode wrapper. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 make cg decode wrapper 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_fast_decode_plan_importable` (lines 149-157)
```python
def test_fast_decode_plan_importable() -> None:
    """fast_decode_plan must be importable from flashinfer.decode.

    This is a forward-compatibility smoke test: if FlashInfer reorganises its
    public API the import will fail before any other test does.
    """
    from flashinfer.decode import fast_decode_plan  # noqa: F401

    assert callable(fast_decode_plan)
```
**EN:** This pytest case verifies fast decode plan importable. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 fast decode plan importable 的行为。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_fast_plan_decode_warmup_uses_full_plan` (lines 160-204)
```python
@pytest.mark.parametrize("dtype", DTYPES)
@torch.inference_mode
def test_fast_plan_decode_warmup_uses_full_plan(dtype: torch.dtype) -> None:
    """On the first call fast_plan_decode must route through self.plan() and
    flip vllm_first_call to False on the wrapper object."""
    from unittest.mock import patch

    from vllm.v1.attention.backends.flashinfer import fast_plan_decode

    torch.set_default_device("cuda")
    set_random_seed(0)

    kv_lens = [128, 64]
    block_size = 16
    num_seqs = len(kv_lens)
    num_query_heads, num_kv_heads = 8, 2
    head_size = 128

    kv_indptr, kv_indices, kv_last_page_lens, _ = _make_paged_kv_metadata(
        kv_lens, block_size, NUM_BLOCKS
    )

    workspace = torch.empty(128 * 1024 * 1024, dtype=torch.int8)
    wrapper = _make_cg_decode_wrapper(num_seqs, kv_indices.clone(), workspace)

    assert getattr(wrapper, "vllm_first_call", True) is True

    with patch.object(wrapper, "plan", wraps=wrapper.plan) as mock_plan:
        fast_plan_decode(
            wrapper,
            indptr_cpu=kv_indptr,
            indices=kv_indices,
            last_page_len_cpu=kv_last_page_lens,
            num_qo_heads=num_query_heads,
            num_kv_heads=num_kv_heads,
            head_dim=head_size,
            page_size=block_size,
            q_data_type=dtype,
            kv_data_type=dtype,
        )
        mock_plan.assert_called_once()

    assert wrapper.vllm_first_call is False, (
        "vllm_first_call should be False after the first fast_plan_decode call"
    )
```
**EN:** This pytest case verifies fast plan decode warmup uses full plan. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as dtype. assertions at the end lock in the intended behavior or graph shape. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 fast plan decode warmup uses full plan 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 dtype 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态；代码会控制随机性以保证场景可复现。

### Function `test_fast_plan_decode_matches_full_plan` (lines 207-287)
```python
@pytest.mark.parametrize("kv_lens", [[1328, 18, 463], [1, 54, 293, 70]])
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("block_size", BLOCK_SIZES)
@pytest.mark.parametrize("dtype", DTYPES)
@torch.inference_mode
def test_fast_plan_decode_matches_full_plan(
    kv_lens: list[int],
    num_heads: tuple[int, int],
    head_size: int,
    block_size: int,
    dtype: torch.dtype,
) -> None:
    """fast_plan_decode's cudagraph path (delegating to FlashInfer's
    fast_decode_plan) must produce attention output numerically identical to
    a standard plan() call.

    Both the warmup call (self.plan) and the subsequent fast call
    (fast_decode_plan) are verified against the same reference.
    """
    from vllm.v1.attention.backends.flashinfer import fast_plan_decode

    torch.set_default_device("cuda")
    set_random_seed(0)
    num_seqs = len(kv_lens)
    num_query_heads, num_kv_heads = num_heads

    query = torch.randn(num_seqs, num_query_heads, head_size, dtype=dtype)
    key_value_cache = torch.randn(
        NUM_BLOCKS, 2, block_size, num_kv_heads, head_size, dtype=dtype
    )

    kv_indptr, kv_indices, kv_last_page_lens, _ = _make_paged_kv_metadata(
        kv_lens, block_size, NUM_BLOCKS
    )

    # Reference output via the standard plan()
    workspace_ref = torch.empty(128 * 1024 * 1024, dtype=torch.int8)
    ref_wrapper = flashinfer.BatchDecodeWithPagedKVCacheWrapper(
        workspace_ref, "NHD", use_tensor_cores=True
# ... excerpt ...
        kv_indptr,
        kv_indices,
        kv_last_page_lens,
        num_query_heads,
        num_kv_heads,
        head_size,
        block_size,
        "NONE",
        q_data_type=dtype,
        kv_data_type=dtype,
    )
    ref_output = ref_wrapper.run(query, key_value_cache)

    # CUDAGraph wrapper exercised through fast_plan_decode
    kv_indices_buf = kv_indices.clone()
    workspace_cg = torch.empty(128 * 1024 * 1024, dtype=torch.int8)
    cg_wrapper = _make_cg_decode_wrapper(num_seqs, kv_indices_buf, workspace_cg)

    plan_kwargs: dict = dict(
        indptr_cpu=kv_indptr,
        indices=kv_indices_buf,
        last_page_len_cpu=kv_last_page_lens,
        num_qo_heads=num_query_heads,
        num_kv_heads=num_kv_heads,
        head_dim=head_size,
        page_size=block_size,
        q_data_type=dtype,
        kv_data_type=dtype,
    )

    # First call – warmup path (routes through self.plan)
    fast_plan_decode(cg_wrapper, **plan_kwargs)
    warmup_output = cg_wrapper.run(query, key_value_cache)
    torch.testing.assert_close(warmup_output, ref_output, atol=1e-2, rtol=1e-2)

    # Second call – fast path (routes through fast_decode_plan from FlashInfer)
    fast_plan_decode(cg_wrapper, **plan_kwargs)
    fast_output = cg_wrapper.run(query, key_value_cache)
    torch.testing.assert_close(fast_output, ref_output, atol=1e-2, rtol=1e-2)
```
**EN:** This pytest case verifies fast plan decode matches full plan. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as kv_lens, num_heads, head_size, block_size. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 fast plan decode matches full plan 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 kv_lens、num_heads、head_size、block_size 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

### Function `test_flashinfer_decode_with_paged_kv` (lines 290-382)
```python
@pytest.mark.parametrize("kv_lens", [[1328, 18, 463], [1, 54, 293, 70]])
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("block_size", BLOCK_SIZES)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("soft_cap", SOFT_CAPS)
@pytest.mark.parametrize("sliding_window", SLIDING_WINDOWS)
@torch.inference_mode
def test_flashinfer_decode_with_paged_kv(
    kv_lens: list[int],
    num_heads: tuple[int, int],
    head_size: int,
    dtype: torch.dtype,
    block_size: int,
    soft_cap: float | None,
    sliding_window: int | None,
) -> None:
    torch.set_default_device("cuda")
    set_random_seed(0)
    num_seqs = len(kv_lens)
    num_query_heads = num_heads[0]
    num_kv_heads = num_heads[1]
    assert num_query_heads % num_kv_heads == 0
    max_kv_len = max(kv_lens)
    scale = head_size**-0.5

    query = torch.randn(num_seqs, num_query_heads, head_size, dtype=dtype)

    key_value_cache = torch.randn(
        NUM_BLOCKS, 2, block_size, num_kv_heads, head_size, dtype=dtype
    )
    key_cache = key_value_cache[:, 0, :, :, :].squeeze(1)
    value_cache = key_value_cache[:, 1, :, :, :].squeeze(1)

    max_num_blocks_per_seq = (max_kv_len + block_size - 1) // block_size
    block_tables = torch.randint(
        0, NUM_BLOCKS, (num_seqs, max_num_blocks_per_seq), dtype=torch.int32
    )

    kv_indptr = [0]
# ... excerpt ...
    kv_indices = torch.tensor(kv_indices, dtype=torch.int32)
    kv_last_page_lens = torch.tensor(kv_last_page_lens, dtype=torch.int32)

    workspace_buffer = torch.empty(128 * 1024 * 1024, dtype=torch.int8)
    wrapper = flashinfer.BatchDecodeWithPagedKVCacheWrapper(
        workspace_buffer, "NHD", use_tensor_cores=True
    )
    wrapper.plan(
        kv_indptr,
        kv_indices,
        kv_last_page_lens,
        num_query_heads,
        num_kv_heads,
        head_size,
        block_size,
        "NONE",
        window_left=sliding_window - 1 if sliding_window is not None else -1,
        q_data_type=dtype,
        kv_data_type=dtype,
        logits_soft_cap=soft_cap,
    )

    output = wrapper.run(query, key_value_cache)

    ref_output = ref_paged_attn(
        query=query,
        key_cache=key_cache,
        value_cache=value_cache,
        query_lens=[1] * num_seqs,
        kv_lens=kv_lens,
        block_tables=block_tables,
        scale=scale,
        soft_cap=soft_cap,
        sliding_window=sliding_window,
    )
    (
        torch.testing.assert_close(output, ref_output, atol=1e-2, rtol=1e-2),
        f"{torch.max(torch.abs(output - ref_output))}",
    )
```
**EN:** This pytest case verifies flashinfer decode with paged KV. It is parameterized across 7 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as kv_lens, num_heads, head_size, dtype. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 flashinfer decode with paged KV 的行为。 它通过 7 组参数化输入覆盖多种场景；它会使用诸如 kv_lens、num_heads、head_size、dtype 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

### Function `test_flashinfer_prefill_with_paged_kv` (lines 385-487)
```python
@pytest.mark.parametrize("seq_lens", [[(1, 1328), (5, 18), (129, 463)]])
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("block_size", BLOCK_SIZES)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("soft_cap", SOFT_CAPS)
@pytest.mark.parametrize("sliding_window", SLIDING_WINDOWS)
@torch.inference_mode
def test_flashinfer_prefill_with_paged_kv(
    seq_lens: list[tuple[int, int]],
    num_heads: tuple[int, int],
    head_size: int,
    dtype: torch.dtype,
    block_size: int,
    soft_cap: float | None,
    sliding_window: int | None,
) -> None:
    torch.set_default_device("cuda")
    set_random_seed(0)
    num_seqs = len(seq_lens)
    query_lens = [x[0] for x in seq_lens]
    kv_lens = [x[1] for x in seq_lens]
    num_query_heads = num_heads[0]
    num_kv_heads = num_heads[1]
    assert num_query_heads % num_kv_heads == 0
    max_kv_len = max(kv_lens)
    scale = head_size**-0.5

    query = torch.randn(sum(query_lens), num_query_heads, head_size, dtype=dtype)
    key_value_cache = torch.randn(
        NUM_BLOCKS, 2, block_size, num_kv_heads, head_size, dtype=dtype
    )
    key_cache = key_value_cache[:, 0, :, :, :].squeeze(1)
    value_cache = key_value_cache[:, 1, :, :, :].squeeze(1)

    # Normalize the scale of the key and value caches to mitigate
    # numerical instability.
    key_cache /= head_size**0.5
    value_cache /= head_size**0.5

# ... excerpt ...
    kv_last_page_lens = torch.tensor(kv_last_page_lens, dtype=torch.int32)

    workspace_buffer = torch.empty(128 * 1024 * 1024, dtype=torch.int8)
    wrapper = flashinfer.BatchPrefillWithPagedKVCacheWrapper(workspace_buffer, "NHD")
    wrapper.plan(
        qo_indptr,
        kv_indptr,
        kv_indices,
        kv_last_page_lens,
        num_query_heads,
        num_kv_heads,
        head_size,
        block_size,
        window_left=sliding_window - 1 if sliding_window is not None else -1,
        q_data_type=dtype,
        kv_data_type=dtype,
        logits_soft_cap=soft_cap,
    )

    output = wrapper.run(
        query,
        key_value_cache,
    )

    ref_output = ref_paged_attn(
        query=query,
        key_cache=key_cache,
        value_cache=value_cache,
        query_lens=query_lens,
        kv_lens=kv_lens,
        block_tables=block_tables,
        scale=scale,
        soft_cap=soft_cap,
        sliding_window=sliding_window,
    )
    (
        torch.testing.assert_close(output, ref_output, atol=5e-2, rtol=1e-2),
        f"{torch.max(torch.abs(output - ref_output))}",
    )
```
**EN:** This pytest case verifies flashinfer prefill with paged KV. It is parameterized across 7 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as seq_lens, num_heads, head_size, dtype. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 flashinfer prefill with paged KV 的行为。 它通过 7 组参数化输入覆盖多种场景；它会使用诸如 seq_lens、num_heads、head_size、dtype 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

### Function `test_flashinfer_prefill_with_paged_fp8_kv` (lines 490-595)
```python
@pytest.mark.parametrize("seq_lens", [[(1, 132), (5, 18)]])
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("block_size", BLOCK_SIZES)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("soft_cap", SOFT_CAPS)
def test_flashinfer_prefill_with_paged_fp8_kv(
    seq_lens: list[tuple[int, int]],
    num_heads: tuple[int, int],
    head_size: int,
    dtype: torch.dtype,
    block_size: int,
    soft_cap: float | None,
) -> None:
    pytest.skip("TODO: fix the accuracy issue")
    torch.set_default_device("cuda")
    set_random_seed(0)
    num_seqs = len(seq_lens)
    query_lens = [x[0] for x in seq_lens]
    kv_lens = [x[1] for x in seq_lens]
    num_query_heads = num_heads[0]
    num_kv_heads = num_heads[1]
    assert num_query_heads % num_kv_heads == 0
    max_kv_len = max(kv_lens)
    scale = head_size**-0.5

    kv_cache_dtype = torch.float8_e4m3fn

    query = torch.randn(sum(query_lens), num_query_heads, head_size, dtype=dtype)
    NUM_BLOCKS_FP8 = 2048
    key_value_cache = torch.randn(
        NUM_BLOCKS_FP8, 2, block_size, num_kv_heads, head_size, dtype=dtype
    )
    key_cache, value_cache = torch.chunk(key_value_cache, 2, dim=1)
    key_cache /= head_size**0.5
    value_cache /= head_size**0.5

    k_scale = key_cache.amax().item() / 448.0
    v_scale = value_cache.amax().item() / 448.0

# ... excerpt ...
    kv_indptr = torch.tensor(kv_indptr, dtype=torch.int32)
    kv_indices = torch.tensor(kv_indices, dtype=torch.int32)
    kv_last_page_lens = torch.tensor(kv_last_page_lens, dtype=torch.int32)

    workspace_buffer = torch.empty(128 * 1024 * 1024, dtype=torch.int8)
    wrapper = flashinfer.BatchPrefillWithPagedKVCacheWrapper(workspace_buffer, "NHD")
    wrapper.plan(
        qo_indptr,
        kv_indptr,
        kv_indices,
        kv_last_page_lens,
        num_query_heads,
        num_kv_heads,
        head_size,
        block_size,
        q_data_type=dtype,
        kv_data_type=kv_cache_dtype,
        logits_soft_cap=soft_cap,
    )

    output = wrapper.run(query, kv_cache_fp8, k_scale=k_scale, v_scale=v_scale)

    ref_output = ref_paged_attn(
        query=query,
        key_cache=key_cache.squeeze(1),
        value_cache=value_cache.squeeze(1),
        query_lens=query_lens,
        kv_lens=kv_lens,
        block_tables=block_tables,
        scale=scale,
        soft_cap=soft_cap,
    )
    del query
    del block_tables
    # verify prefill fp8
    (
        torch.testing.assert_close(output, ref_output, atol=5e-2, rtol=1e-2),
        f"{torch.max(torch.abs(output - ref_output))}",
    )
```
**EN:** This pytest case verifies flashinfer prefill with paged FP8 KV. It is parameterized across 6 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as seq_lens, num_heads, head_size, dtype. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 flashinfer prefill with paged FP8 KV 的行为。 它通过 6 组参数化输入覆盖多种场景；它会使用诸如 seq_lens、num_heads、head_size、dtype 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

### Function `test_flashinfer_decode_with_paged_fp8_kv` (lines 598-701)
```python
@pytest.mark.parametrize("kv_lens", [[1328, 18, 463], [1, 54, 293, 70]])
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("block_size", BLOCK_SIZES)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("soft_cap", SOFT_CAPS)
@pytest.mark.skip(reason="TODO: fix the accuracy issue")
@torch.inference_mode
def test_flashinfer_decode_with_paged_fp8_kv(
    kv_lens: list[int],
    num_heads: tuple[int, int],
    head_size: int,
    dtype: torch.dtype,
    block_size: int,
    soft_cap: float | None,
) -> None:
    # test doesn't work for num_heads = (16,16)
    torch.set_default_device("cuda")
    set_random_seed(0)
    num_seqs = len(kv_lens)
    num_query_heads = num_heads[0]
    num_kv_heads = num_heads[1]
    assert num_query_heads % num_kv_heads == 0
    max_kv_len = max(kv_lens)
    scale = head_size**-0.5
    use_tensor_cores = True
    kv_cache_dtype = torch.float8_e4m3fn

    query = torch.randn(num_seqs, num_query_heads, head_size, dtype=dtype)
    NUM_BLOCKS_FP8 = 2048
    key_value_cache = torch.randn(
        NUM_BLOCKS_FP8, 2, block_size, num_kv_heads, head_size, dtype=dtype
    )
    key_cache, value_cache = torch.chunk(key_value_cache, 2, dim=1)
    key_cache /= head_size**0.5
    value_cache /= head_size**0.5

    k_scale = key_cache.amax().item() / 448.0
    v_scale = value_cache.amax().item() / 448.0

# ... excerpt ...
    kv_indices = torch.tensor(kv_indices, dtype=torch.int32)
    kv_last_page_lens = torch.tensor(kv_last_page_lens, dtype=torch.int32)

    workspace_buffer = torch.empty(128 * 1024 * 1024, dtype=torch.int8)
    wrapper = flashinfer.BatchDecodeWithPagedKVCacheWrapper(
        workspace_buffer, "NHD", use_tensor_cores=use_tensor_cores
    )
    wrapper.plan(
        kv_indptr,
        kv_indices,
        kv_last_page_lens,
        num_query_heads,
        num_kv_heads,
        head_size,
        block_size,
        "NONE",
        q_data_type=dtype,
        kv_data_type=kv_cache_dtype,
        logits_soft_cap=soft_cap,
    )
    output = wrapper.run(query, kv_cache_fp8, k_scale=k_scale, v_scale=v_scale)
    key_cache = key_value_cache[:, 0, :, :, :].squeeze(1)
    value_cache = key_value_cache[:, 1, :, :, :].squeeze(1)

    ref_output = ref_paged_attn(
        query=query,
        key_cache=key_cache,
        value_cache=value_cache,
        query_lens=[1] * num_seqs,
        kv_lens=kv_lens,
        block_tables=block_tables,
        scale=scale,
        soft_cap=soft_cap,
    )
    # Temporary fix: Increasing the tolerance. Seems like a flashinfer issue
    (
        torch.testing.assert_close(output, ref_output, atol=2e-2, rtol=1e-2),
        f"{torch.max(torch.abs(output - ref_output))}",
    )
```
**EN:** This pytest case verifies flashinfer decode with paged FP8 KV. It is parameterized across 6 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as kv_lens, num_heads, head_size, dtype. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 flashinfer decode with paged FP8 KV 的行为。 它通过 6 组参数化输入覆盖多种场景；它会使用诸如 kv_lens、num_heads、head_size、dtype 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `vllm.platforms -> current_platform`
- `vllm.utils.torch_utils -> set_random_seed`
- `torch`
