# test_cpu_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/attention/test_cpu_attn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / attention / test_cpu_attn, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / attention / test_cpu_attn 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-12)
```python
import functools
import math

import pytest
import torch

from vllm.platforms import CpuArchEnum, current_platform
from vllm.utils.torch_utils import set_random_seed
from vllm.v1.attention.backends.cpu_attn import _get_attn_isa
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as functools, math, pytest, torch; and vLLM components like vllm.platforms, vllm.utils.torch_utils, vllm.v1.attention.backends.cpu_attn.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 functools、math、pytest、torch；vLLM 内部组件，例如 vllm.platforms、vllm.utils.torch_utils、vllm.v1.attention.backends.cpu_attn。

### Top-level block starting at line 14 (lines 14-15)
```python
if not current_platform.is_cpu():
    pytest.skip("skipping CPU-only tests", allow_module_level=True)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Imports and shared setup (lines 17-21)
```python
from vllm._custom_ops import (
    cpu_attention_with_kv_cache,
    cpu_attn_get_scheduler_metadata,
    cpu_attn_reshape_and_cache,
)
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in and vLLM components like vllm._custom_ops.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括vLLM 内部组件，例如 vllm._custom_ops。

### Top-level block starting at line 25 (lines 25-26)
```python
if torch.cpu._is_amx_tile_supported():
    torch.cpu._init_amx()
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 29-45)
```python
NUM_HEADS = [
    (4, 4),
    (8, 2),
    (9, 3),
]
HEAD_SIZES = [96, 128, 512]
HEAD_SIZES_VEC16 = [96, 80, 112, 128]
QTYPES = [torch.bfloat16, torch.half, torch.float32]
SLIDING_WINDOWS = [None, 256]
NUM_BLOCKS = [
    1024,
]
SEQ_LENS = [  # (q_len, kv_len)
    [(1, 213), (1, 1), (1, 312), (1, 7), (1, 7812)],  # decode batch
    [(2345, 2345), (5, 5), (3, 16), (134, 5131)],  # prefill batch
    [(992, 2456), (1, 1234), (98, 1145), (1, 4162), (2345, 2345)],  # mixed batch
]
```
**EN:** This block centralizes shared constants and parameter grids, including NUM_HEADS, HEAD_SIZES, HEAD_SIZES_VEC16, QTYPES, SLIDING_WINDOWS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 NUM_HEADS、HEAD_SIZES、HEAD_SIZES_VEC16、QTYPES、SLIDING_WINDOWS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `get_attn_isa` (lines 48-59)
```python
def get_attn_isa(
    block_size: int | None = None,
    dtype: torch.dtype | None = None,
):
    # Delegate to _get_attn_isa so the fallback path applies the same arch
    # gating (e.g. RISC-V RVV is only chosen when the build's hardcoded
    # VLEN=128 kernel is actually present; on VLEN=256 / scalar hosts it
    # correctly falls through to vec/vec16).
    return _get_attn_isa(
        dtype if dtype is not None else torch.bfloat16,
        block_size if block_size else 32,
    )
```
**EN:** This helper function implements the shared logic for attn isa. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 attn isa 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `tensor_cache` (lines 63-70)
```python
@functools.lru_cache(maxsize=128, typed=False)
def tensor_cache(
    elem_num: int,
    dtype: torch.dtype,
) -> torch.Tensor:
    tensor = torch.randn(elem_num, dtype=dtype)

    return tensor
```
**EN:** This helper function implements the shared logic for tensor cache. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 tensor cache 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_get_alibi_slopes` (lines 73-94)
```python
def _get_alibi_slopes(total_num_heads: int) -> torch.Tensor:
    closest_power_of_2 = 2 ** math.floor(math.log2(total_num_heads))
    base = torch.tensor(
        2 ** (-(2 ** -(math.log2(closest_power_of_2) - 3))),
        dtype=torch.float32,
    )
    powers = torch.arange(1, 1 + closest_power_of_2, dtype=torch.int32)
    slopes = torch.pow(base, powers)

    if closest_power_of_2 != total_num_heads:
        extra_base = torch.tensor(
            2 ** (-(2 ** -(math.log2(2 * closest_power_of_2) - 3))),
            dtype=torch.float32,
        )
        num_remaining_heads = min(
            closest_power_of_2, total_num_heads - closest_power_of_2
        )
        extra_powers = torch.arange(
            start=1, end=1 + 2 * num_remaining_heads, step=2, dtype=torch.int32
        )
        slopes = torch.cat([slopes, torch.pow(extra_base, extra_powers)], dim=0)
    return slopes.float()
```
**EN:** This helper function implements the shared logic for get ALiBi slopes. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 get ALiBi slopes 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `ref_paged_attn` (lines 97-183)
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
    alibi_slopes: torch.Tensor | None = None,
    s_aux: torch.Tensor | None = None,
) -> torch.Tensor:
    num_seqs = len(query_lens)
    block_tables = block_tables.cpu().numpy()
    _, block_size, num_kv_heads, head_size = key_cache.shape
    dtype = query.dtype

    outputs: list[torch.Tensor] = []
    start_idx = 0

    if alibi_slopes is not None:
        alibi_slopes = alibi_slopes[:, None, None]

    if s_aux is not None:
        s_aux = s_aux.float()
        s_aux = s_aux[:, None, None]

    for i in range(num_seqs):
        query_len = query_lens[i]
        kv_len = kv_lens[i]
        q = query[start_idx : start_idx + query_len].float()
        q *= scale

        num_kv_blocks = (kv_len + block_size - 1) // block_size
        block_indices = block_tables[i, :num_kv_blocks]

        k = key_cache[block_indices].view(-1, num_kv_heads, head_size)
        k = k[:kv_len].float()
        v = value_cache[block_indices].view(-1, num_kv_heads, head_size)
# ... excerpt ...

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

        if alibi_slopes is not None:
            q_start_pos = kv_len - query_len
            q_pos = q_start_pos + torch.arange(0, query_len)[None, :, None]
            kv_pos = torch.arange(0, kv_len)[None, None, :]
            dist = q_pos - kv_pos
            alibi_bias = -alibi_slopes * dist
            attn += alibi_bias

        attn.masked_fill_(mask, float("-inf"))

        if s_aux is not None:
            s_aux_ext = s_aux.repeat(1, query_len, 1)
            attn = torch.cat((s_aux_ext, attn), dim=-1)

        attn = torch.softmax(attn, dim=-1)

        if s_aux is not None:
            attn = attn[:, :, 1:]

        out = torch.einsum("hqk,khd->qhd", attn, v).to(dtype=dtype)

        outputs.append(out)
        start_idx += query_len

    return torch.cat(outputs, dim=0)
```
**EN:** This helper acts as a reference implementation for paged attn. It keeps the surrounding module logic factored into a reusable unit. Only the key portions are shown here because the block is large.
**CN:** 该辅助函数为 paged attn 提供参考实现。 它把周边模块中的共用逻辑封装成可复用单元。 由于该代码块较大，这里只展示关键片段。

### Constants and module state (lines 186-187)
```python
_FP8_ATOL = {"fp8_e4m3": 0.2, "fp8_e5m2": 0.3}
_FP8_RTOL = 0.1
```
**EN:** This block centralizes shared constants and parameter grids, including _FP8_ATOL, _FP8_RTOL. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 _FP8_ATOL、_FP8_RTOL。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `varlen_with_paged_kv` (lines 190-415)
```python
@torch.inference_mode()
def varlen_with_paged_kv(
    seq_lens: list[tuple[int, int]],
    num_heads: tuple[int, int],
    head_size: int,
    sliding_window: int | None,
    dtype: torch.dtype,
    block_size: int,
    soft_cap: float | None,
    num_blocks: int,
    use_alibi: bool,
    use_sink: bool,
    isa: str,
    kv_cache_dtype: str = "auto",
    k_scale: float = 1.0,
    v_scale: float = 1.0,
) -> None:
    set_random_seed(0)
    num_seqs = len(seq_lens)
    query_lens = [x[0] for x in seq_lens]
    kv_lens = [x[1] for x in seq_lens]
    num_query_heads = num_heads[0]
    num_kv_heads = num_heads[1]
    assert num_query_heads % num_kv_heads == 0
    max_kv_len = max(kv_lens)
    window_size = (sliding_window - 1, 0) if sliding_window is not None else (-1, -1)
    scale = head_size**-0.5
    token_num = sum(query_lens)

    # for n heads the set of slopes is the geometric sequence that starts
    # 2^(-8/n)
    alibi_slopes = _get_alibi_slopes(num_query_heads) if use_alibi else None

    s_aux = (
        15 * torch.rand((num_query_heads,), dtype=torch.bfloat16) if use_sink else None
    )

    is_fp8 = kv_cache_dtype != "auto"
    if is_fp8 and current_platform.get_cpu_architecture() != CpuArchEnum.X86:
        pytest.skip("FP8 KV cache only supported on x86")
# ... excerpt ...
            value_cache=ref_value_cache,
            output=ref_output,
            query_start_loc=cu_query_lens,
            seq_lens=kv_lens_tensor,
            scale=scale,
            causal=True,
            alibi_slopes=alibi_slopes,
            sliding_window=window_size,
            block_table=block_tables,
            softcap=soft_cap if soft_cap is not None else 0,
            scheduler_metadata=metadata,
            s_aux=s_aux,
        )
        atol = _FP8_ATOL[kv_cache_dtype]
        rtol = _FP8_RTOL
    else:
        ref_output = ref_paged_attn(
            query=query,
            key_cache=key_cache,
            value_cache=value_cache,
            query_lens=query_lens,
            kv_lens=kv_lens,
            block_tables=block_tables,
            scale=scale,
            sliding_window=sliding_window,
            soft_cap=soft_cap,
            alibi_slopes=alibi_slopes,
            s_aux=s_aux,
        )
        atol, rtol = 1.5e-2, 1e-2

    (
        torch.testing.assert_close(out_with_split, ref_output, atol=atol, rtol=rtol),
        f"{torch.max(torch.abs(out_with_split - ref_output))}",
    )
    (
        torch.testing.assert_close(out_without_split, ref_output, atol=atol, rtol=rtol),
        f"{torch.max(torch.abs(out_without_split - ref_output))}",
    )
```
**EN:** This helper function implements the shared logic for varlen with paged KV. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该辅助函数实现了 varlen with paged KV 所需的共享逻辑。 不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

### Function `test_varlen_with_paged_kv_normal_vec` (lines 418-457)
```python
@pytest.mark.parametrize("kv_cache_dtype", ["auto", "fp8_e4m3", "fp8_e5m2"])
@pytest.mark.parametrize("seq_lens", SEQ_LENS)
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("block_size", [96, 128])
@pytest.mark.parametrize("sliding_window", SLIDING_WINDOWS)
@pytest.mark.parametrize("dtype", QTYPES)
@pytest.mark.parametrize("soft_cap", [None])
@pytest.mark.parametrize("num_blocks", NUM_BLOCKS)
@pytest.mark.parametrize("use_alibi", [False])
@pytest.mark.parametrize("use_sink", [False])
@pytest.mark.parametrize("isa", ["vec"])
def test_varlen_with_paged_kv_normal_vec(
    seq_lens: list[tuple[int, int]],
    num_heads: tuple[int, int],
    head_size: int,
    sliding_window: int | None,
    dtype: torch.dtype,
    block_size: int,
    soft_cap: float | None,
    num_blocks: int,
    use_alibi: bool,
    use_sink: bool,
    isa: str,
    kv_cache_dtype: str,
) -> None:
    varlen_with_paged_kv(
        seq_lens=seq_lens,
        num_heads=num_heads,
        head_size=head_size,
        sliding_window=sliding_window,
        dtype=dtype,
        block_size=block_size,
        soft_cap=soft_cap,
        num_blocks=num_blocks,
        use_alibi=use_alibi,
        use_sink=use_sink,
        isa=isa,
        kv_cache_dtype=kv_cache_dtype,
    )
```
**EN:** This pytest case verifies varlen with paged KV normal vec. It is parameterized across 12 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as seq_lens, num_heads, head_size, sliding_window.
**CN:** 该 pytest 用例验证 varlen with paged KV normal vec 的行为。 它通过 12 组参数化输入覆盖多种场景；它会使用诸如 seq_lens、num_heads、head_size、sliding_window 等 fixture 或输入。

### Function `test_varlen_with_paged_kv_normal_amx` (lines 460-500)
```python
@pytest.mark.parametrize("kv_cache_dtype", ["auto", "fp8_e4m3", "fp8_e5m2"])
@pytest.mark.parametrize("seq_lens", SEQ_LENS)
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("block_size", [96, 128])
@pytest.mark.parametrize("sliding_window", SLIDING_WINDOWS)
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@pytest.mark.parametrize("soft_cap", [None])
@pytest.mark.parametrize("num_blocks", NUM_BLOCKS)
@pytest.mark.parametrize("use_alibi", [False])
@pytest.mark.parametrize("use_sink", [False])
@pytest.mark.parametrize("isa", ["amx"])
@pytest.mark.skipif(not torch.cpu._is_amx_tile_supported(), reason="no AMX support.")
def test_varlen_with_paged_kv_normal_amx(
    seq_lens: list[tuple[int, int]],
    num_heads: tuple[int, int],
    head_size: int,
    sliding_window: int | None,
    dtype: torch.dtype,
    block_size: int,
    soft_cap: float | None,
    num_blocks: int,
    use_alibi: bool,
    use_sink: bool,
    isa: str,
    kv_cache_dtype: str,
) -> None:
    varlen_with_paged_kv(
        seq_lens=seq_lens,
        num_heads=num_heads,
        head_size=head_size,
        sliding_window=sliding_window,
        dtype=dtype,
        block_size=block_size,
        soft_cap=soft_cap,
        num_blocks=num_blocks,
        use_alibi=use_alibi,
        use_sink=use_sink,
        isa=isa,
        kv_cache_dtype=kv_cache_dtype,
    )
```
**EN:** This pytest case verifies varlen with paged KV normal amx. It is parameterized across 12 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as seq_lens, num_heads, head_size, sliding_window. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 varlen with paged KV normal amx 的行为。 它通过 12 组参数化输入覆盖多种场景；它会使用诸如 seq_lens、num_heads、head_size、sliding_window 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_varlen_with_paged_kv_normal_vec16` (lines 503-539)
```python
@pytest.mark.parametrize("seq_lens", SEQ_LENS)
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES_VEC16)
@pytest.mark.parametrize("block_size", [48])
@pytest.mark.parametrize("sliding_window", SLIDING_WINDOWS)
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@pytest.mark.parametrize("soft_cap", [None])
@pytest.mark.parametrize("num_blocks", NUM_BLOCKS)
@pytest.mark.parametrize("use_alibi", [False])
@pytest.mark.parametrize("use_sink", [False])
@pytest.mark.parametrize("isa", ["vec16"])
def test_varlen_with_paged_kv_normal_vec16(
    seq_lens: list[tuple[int, int]],
    num_heads: tuple[int, int],
    head_size: int,
    sliding_window: int | None,
    dtype: torch.dtype,
    block_size: int,
    soft_cap: float | None,
    num_blocks: int,
    use_alibi: bool,
    use_sink: bool,
    isa: str,
) -> None:
    varlen_with_paged_kv(
        seq_lens=seq_lens,
        num_heads=num_heads,
        head_size=head_size,
        sliding_window=sliding_window,
        dtype=dtype,
        block_size=block_size,
        soft_cap=soft_cap,
        num_blocks=num_blocks,
        use_alibi=use_alibi,
        use_sink=use_sink,
        isa=isa,
    )
```
**EN:** This pytest case verifies varlen with paged KV normal vec16. It is parameterized across 11 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as seq_lens, num_heads, head_size, sliding_window.
**CN:** 该 pytest 用例验证 varlen with paged KV normal vec16 的行为。 它通过 11 组参数化输入覆盖多种场景；它会使用诸如 seq_lens、num_heads、head_size、sliding_window 等 fixture 或输入。

### Function `test_varlen_with_paged_kv_normal_neon` (lines 542-582)
```python
@pytest.mark.parametrize("seq_lens", SEQ_LENS)
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("block_size", [96, 128])
@pytest.mark.parametrize("sliding_window", SLIDING_WINDOWS)
@pytest.mark.parametrize("dtype", QTYPES)
@pytest.mark.parametrize("soft_cap", [None])
@pytest.mark.parametrize("num_blocks", NUM_BLOCKS)
@pytest.mark.parametrize("use_alibi", [False])
@pytest.mark.parametrize("use_sink", [False])
@pytest.mark.parametrize("isa", ["neon"])
@pytest.mark.skipif(
    current_platform.get_cpu_architecture() != CpuArchEnum.ARM,
    reason="Not an Arm CPU.",
)
def test_varlen_with_paged_kv_normal_neon(
    seq_lens: list[tuple[int, int]],
    num_heads: tuple[int, int],
    head_size: int,
    sliding_window: int | None,
    dtype: torch.dtype,
    block_size: int,
    soft_cap: float | None,
    num_blocks: int,
    use_alibi: bool,
    use_sink: bool,
    isa: str,
) -> None:
    varlen_with_paged_kv(
        seq_lens=seq_lens,
        num_heads=num_heads,
        head_size=head_size,
        sliding_window=sliding_window,
        dtype=dtype,
        block_size=block_size,
        soft_cap=soft_cap,
        num_blocks=num_blocks,
        use_alibi=use_alibi,
        use_sink=use_sink,
        isa=isa,
    )
```
**EN:** This pytest case verifies varlen with paged KV normal neon. It is parameterized across 11 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as seq_lens, num_heads, head_size, sliding_window. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 varlen with paged KV normal neon 的行为。 它通过 11 组参数化输入覆盖多种场景；它会使用诸如 seq_lens、num_heads、head_size、sliding_window 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_varlen_with_paged_kv_normal_rvv` (lines 585-628)
```python
@pytest.mark.parametrize("kv_cache_dtype", ["auto", "fp8_e4m3"])
@pytest.mark.parametrize("seq_lens", SEQ_LENS)
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("head_size", HEAD_SIZES)
@pytest.mark.parametrize("block_size", [96, 128])
@pytest.mark.parametrize("sliding_window", SLIDING_WINDOWS)
@pytest.mark.parametrize("dtype", QTYPES)
@pytest.mark.parametrize("soft_cap", [None])
@pytest.mark.parametrize("num_blocks", NUM_BLOCKS)
@pytest.mark.parametrize("use_alibi", [False])
@pytest.mark.parametrize("use_sink", [False])
@pytest.mark.parametrize("isa", ["rvv"])
@pytest.mark.skipif(
    current_platform.get_cpu_architecture() != CpuArchEnum.RISCV,
    reason="Not a RISC-V CPU.",
)
def test_varlen_with_paged_kv_normal_rvv(
    seq_lens: list[tuple[int, int]],
    num_heads: tuple[int, int],
    head_size: int,
    sliding_window: int | None,
    dtype: torch.dtype,
    block_size: int,
    soft_cap: float | None,
    num_blocks: int,
    use_alibi: bool,
    use_sink: bool,
    isa: str,
    kv_cache_dtype: str,
) -> None:
    varlen_with_paged_kv(
        seq_lens=seq_lens,
        num_heads=num_heads,
        head_size=head_size,
        sliding_window=sliding_window,
        dtype=dtype,
        block_size=block_size,
        soft_cap=soft_cap,
        num_blocks=num_blocks,
        use_alibi=use_alibi,
        use_sink=use_sink,
        isa=isa,
        kv_cache_dtype=kv_cache_dtype,
    )
```
**EN:** This pytest case verifies varlen with paged KV normal rvv. It is parameterized across 12 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as seq_lens, num_heads, head_size, sliding_window. unsupported hardware, backend, or configuration combinations are skipped early.
**CN:** 该 pytest 用例验证 varlen with paged KV normal rvv 的行为。 它通过 12 组参数化输入覆盖多种场景；它会使用诸如 seq_lens、num_heads、head_size、sliding_window 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过。

### Function `test_varlen_with_paged_kv_softcap` (lines 631-670)
```python
@pytest.mark.parametrize("kv_cache_dtype", ["auto", "fp8_e4m3"])
@pytest.mark.parametrize("seq_lens", SEQ_LENS)
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("head_size", [96])
@pytest.mark.parametrize("block_size", [128])
@pytest.mark.parametrize("sliding_window", SLIDING_WINDOWS)
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@pytest.mark.parametrize("soft_cap", [50])
@pytest.mark.parametrize("num_blocks", NUM_BLOCKS)
@pytest.mark.parametrize("use_alibi", [False])
@pytest.mark.parametrize("use_sink", [False])
@pytest.mark.parametrize("isa", [get_attn_isa()])
def test_varlen_with_paged_kv_softcap(
    seq_lens: list[tuple[int, int]],
    num_heads: tuple[int, int],
    head_size: int,
    sliding_window: int | None,
    dtype: torch.dtype,
    block_size: int,
    soft_cap: float | None,
    num_blocks: int,
    use_alibi: bool,
    use_sink: bool,
    isa: str,
    kv_cache_dtype: str,
) -> None:
    varlen_with_paged_kv(
        seq_lens=seq_lens,
        num_heads=num_heads,
        head_size=head_size,
        sliding_window=sliding_window,
        dtype=dtype,
        block_size=block_size,
        soft_cap=soft_cap,
        num_blocks=num_blocks,
        use_alibi=use_alibi,
        use_sink=use_sink,
        isa=isa,
        kv_cache_dtype=kv_cache_dtype,
    )
```
**EN:** This pytest case verifies varlen with paged KV softcap. It is parameterized across 12 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as seq_lens, num_heads, head_size, sliding_window.
**CN:** 该 pytest 用例验证 varlen with paged KV softcap 的行为。 它通过 12 组参数化输入覆盖多种场景；它会使用诸如 seq_lens、num_heads、head_size、sliding_window 等 fixture 或输入。

### Function `test_varlen_with_paged_kv_alibi` (lines 673-712)
```python
@pytest.mark.parametrize("kv_cache_dtype", ["auto", "fp8_e4m3"])
@pytest.mark.parametrize("seq_lens", SEQ_LENS)
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("head_size", [96])
@pytest.mark.parametrize("block_size", [128])
@pytest.mark.parametrize("sliding_window", SLIDING_WINDOWS)
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@pytest.mark.parametrize("soft_cap", [None])
@pytest.mark.parametrize("num_blocks", NUM_BLOCKS)
@pytest.mark.parametrize("use_alibi", [True])
@pytest.mark.parametrize("use_sink", [False])
@pytest.mark.parametrize("isa", [get_attn_isa()])
def test_varlen_with_paged_kv_alibi(
    seq_lens: list[tuple[int, int]],
    num_heads: tuple[int, int],
    head_size: int,
    sliding_window: int | None,
    dtype: torch.dtype,
    block_size: int,
    soft_cap: float | None,
    num_blocks: int,
    use_alibi: bool,
    use_sink: bool,
    isa: str,
    kv_cache_dtype: str,
) -> None:
    varlen_with_paged_kv(
        seq_lens=seq_lens,
        num_heads=num_heads,
        head_size=head_size,
        sliding_window=sliding_window,
        dtype=dtype,
        block_size=block_size,
        soft_cap=soft_cap,
        num_blocks=num_blocks,
        use_alibi=use_alibi,
        use_sink=use_sink,
        isa=isa,
        kv_cache_dtype=kv_cache_dtype,
    )
```
**EN:** This pytest case verifies varlen with paged KV ALiBi. It is parameterized across 12 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as seq_lens, num_heads, head_size, sliding_window.
**CN:** 该 pytest 用例验证 varlen with paged KV ALiBi 的行为。 它通过 12 组参数化输入覆盖多种场景；它会使用诸如 seq_lens、num_heads、head_size、sliding_window 等 fixture 或输入。

### Function `test_varlen_with_paged_kv_sink` (lines 715-754)
```python
@pytest.mark.parametrize("kv_cache_dtype", ["auto", "fp8_e4m3"])
@pytest.mark.parametrize("seq_lens", SEQ_LENS)
@pytest.mark.parametrize("num_heads", NUM_HEADS)
@pytest.mark.parametrize("head_size", [96])
@pytest.mark.parametrize("block_size", [128])
@pytest.mark.parametrize("sliding_window", SLIDING_WINDOWS)
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@pytest.mark.parametrize("soft_cap", [None])
@pytest.mark.parametrize("num_blocks", NUM_BLOCKS)
@pytest.mark.parametrize("use_alibi", [False])
@pytest.mark.parametrize("use_sink", [True])
@pytest.mark.parametrize("isa", [get_attn_isa()])
def test_varlen_with_paged_kv_sink(
    seq_lens: list[tuple[int, int]],
    num_heads: tuple[int, int],
    head_size: int,
    sliding_window: int | None,
    dtype: torch.dtype,
    block_size: int,
    soft_cap: float | None,
    num_blocks: int,
    use_alibi: bool,
    use_sink: bool,
    isa: str,
    kv_cache_dtype: str,
) -> None:
    varlen_with_paged_kv(
        seq_lens=seq_lens,
        num_heads=num_heads,
        head_size=head_size,
        sliding_window=sliding_window,
        dtype=dtype,
        block_size=block_size,
        soft_cap=soft_cap,
        num_blocks=num_blocks,
        use_alibi=use_alibi,
        use_sink=use_sink,
        isa=isa,
        kv_cache_dtype=kv_cache_dtype,
    )
```
**EN:** This pytest case verifies varlen with paged KV sink. It is parameterized across 12 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as seq_lens, num_heads, head_size, sliding_window.
**CN:** 该 pytest 用例验证 varlen with paged KV sink 的行为。 它通过 12 组参数化输入覆盖多种场景；它会使用诸如 seq_lens、num_heads、head_size、sliding_window 等 fixture 或输入。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `functools`
- `math`
- `pytest`
- `torch`
- `vllm.platforms -> CpuArchEnum, current_platform`
- `vllm.utils.torch_utils -> set_random_seed`
- `vllm.v1.attention.backends.cpu_attn -> _get_attn_isa`
- `vllm._custom_ops -> cpu_attention_with_kv_cache, cpu_attn_get_scheduler_metadata, cpu_attn_reshape_and_cache`
