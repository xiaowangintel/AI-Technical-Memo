# test_mamba_ssm_ssd.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/mamba/test_mamba_ssm_ssd.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / mamba / test_mamba_ssm_ssd, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / mamba / test_mamba_ssm_ssd 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-13)
```python
import pytest
import torch
import torch.nn.functional as F
from einops import rearrange, repeat

from vllm.model_executor.layers.mamba.ops.ssd_combined import (
    mamba_chunk_scan_combined_varlen,
)
from vllm.utils.torch_utils import set_random_seed
from vllm.v1.attention.backends.mamba2_attn import compute_varlen_chunk_metadata
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch, torch.nn.functional, einops; and vLLM components like vllm.model_executor.layers.mamba.ops.ssd_combined, vllm.utils.torch_utils, vllm.v1.attention.backends.mamba2_attn.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch、torch.nn.functional、einops；vLLM 内部组件，例如 vllm.model_executor.layers.mamba.ops.ssd_combined、vllm.utils.torch_utils、vllm.v1.attention.backends.mamba2_attn。

### Function `segsum` (lines 21-30)
```python
def segsum(x):
    """Calculates segment sum."""
    T = x.size(-1)
    x = repeat(x, "... d -> ... d e", e=T)
    mask = torch.tril(torch.ones(T, T, device=x.device, dtype=bool), diagonal=-1)
    x = x.masked_fill(~mask, 0)
    x_segsum = torch.cumsum(x, dim=-2)
    mask = torch.tril(torch.ones(T, T, device=x.device, dtype=bool), diagonal=0)
    x_segsum = x_segsum.masked_fill(~mask, -torch.inf)
    return x_segsum
```
**EN:** This helper function implements the shared logic for segsum. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 segsum 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `ssd_minimal_discrete` (lines 33-81)
```python
def ssd_minimal_discrete(X, A, B, C, block_len, initial_states=None):
    """
    Arguments:
        X: (batch, length, n_heads, d_head)
        A: (batch, length, n_heads)
        B: (batch, length, n_heads, d_state)
        C: (batch, length, n_heads, d_state)
    Return:
        Y: (batch, length, n_heads, d_head)
    """
    assert X.dtype == A.dtype == B.dtype == C.dtype
    assert X.shape[1] % block_len == 0

    # Rearrange into blocks/chunks
    X, A, B, C = (
        rearrange(x, "b (c l) ... -> b c l ...", l=block_len) for x in (X, A, B, C)
    )

    A = rearrange(A, "b c l h -> b h c l")
    A_cumsum = torch.cumsum(A, dim=-1)

    # 1. Compute the output for each intra-chunk (diagonal blocks)
    L = torch.exp(segsum(A))
    Y_diag = torch.einsum("bclhn,bcshn,bhcls,bcshp->bclhp", C, B, L, X)

    # 2. Compute the state for each intra-chunk
    # (right term of low-rank factorization of off-diagonal blocks; B terms)
    decay_states = torch.exp(A_cumsum[:, :, :, -1:] - A_cumsum)
    states = torch.einsum("bclhn,bhcl,bclhp->bchpn", B, decay_states, X)

    # 3. Compute the inter-chunk SSM recurrence; produces correct SSM states at
    #    chunk boundaries
    # (middle term of factorization of off-diag blocks; A terms)
    if initial_states is None:
        initial_states = torch.zeros_like(states[:, :1])
    states = torch.cat([initial_states, states], dim=1)
    decay_chunk = torch.exp(segsum(F.pad(A_cumsum[:, :, :, -1], (1, 0))))
    new_states = torch.einsum("bhzc,bchpn->bzhpn", decay_chunk, states)
    states, final_state = new_states[:, :-1], new_states[:, -1]

    # 4. Compute state -> output conversion per chunk
    # (left term of low-rank factorization of off-diagonal blocks; C terms)
    state_decay_out = torch.exp(A_cumsum)
    Y_off = torch.einsum("bclhn,bchpn,bhcl->bclhp", C, states, state_decay_out)

    # Add output of intra-chunk and inter-chunk terms
    # (diagonal and off-diagonal blocks)
    Y = rearrange(Y_diag + Y_off, "b c l h p -> b (c l) h p")
    return Y, final_state
```
**EN:** This helper function implements the shared logic for ssd minimal discrete. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 ssd minimal discrete 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `generate_random_inputs` (lines 84-94)
```python
def generate_random_inputs(batch_size, seqlen, n_heads, d_head, itype, device="cuda"):
    set_random_seed(0)
    A = -torch.exp(torch.rand(n_heads, dtype=itype, device=device))
    dt = F.softplus(
        torch.randn(batch_size, seqlen, n_heads, dtype=itype, device=device) - 4
    )
    X = torch.randn((batch_size, seqlen, n_heads, d_head), dtype=itype, device=device)
    B = torch.randn((batch_size, seqlen, n_heads, d_head), dtype=itype, device=device)
    C = torch.randn((batch_size, seqlen, n_heads, d_head), dtype=itype, device=device)

    return A, dt, X, B, C
```
**EN:** This helper function implements the shared logic for generate random inputs. randomness is controlled so the scenario remains reproducible.
**CN:** 该辅助函数实现了 generate random inputs 所需的共享逻辑。 代码会控制随机性以保证场景可复现。

### Function `generate_continuous_batched_examples` (lines 97-188)
```python
def generate_continuous_batched_examples(
    example_lens_by_batch,
    num_examples,
    full_length,
    last_taken,
    exhausted,
    n_heads,
    d_head,
    itype,
    device="cuda",
    return_naive_ref=True,
):
    # this function generates a random examples of certain length
    # and then cut according to "example_lens_by_batch" and feed
    # them in continuous batches to the kernels.
    # If if return_naive_ref=True, the naive torch implementation
    # ssd_minimal_discrete will be used to compute and return
    # reference output.

    # generate the full-length example
    A, dt, X, B, C = generate_random_inputs(
        num_examples, full_length, n_heads, d_head, itype
    )

    if return_naive_ref:
        Y_min, final_state_min = ssd_minimal_discrete(
            X * dt.unsqueeze(-1), A * dt, B, C, block_len=full_length // 4
        )

    # internal function that outputs a cont batch of examples
    # given a tuple of lengths for each example in the batch
    # e.g., example_lens=(8, 4) means take 8 samples from first eg,
    #       4 examples from second eg, etc
    def get_continuous_batch(example_lens: tuple[int, ...]):
        indices = []
        for i, x in enumerate(example_lens):
            c = last_taken.get(i, 0)
            indices.append((c, c + x))
            last_taken[i] = (c + x) % full_length
            exhausted[i] = last_taken[i] == 0
# ... excerpt ...

    IND_E = None
    for spec in example_lens_by_batch:
        # get the (maybe partial) example seen in this cont batch
        dt2, X2, B2, C2 = get_continuous_batch(spec)

        # get the metadata
        cu_seqlens = torch.tensor((0,) + spec, device=device).cumsum(dim=0)
        seq_idx = torch.zeros(
            cu_seqlens[-1], dtype=torch.int32, device=cu_seqlens.device
        )
        for i, (srt, end) in enumerate(
            zip(
                cu_seqlens,
                cu_seqlens[1:],
            )
        ):
            seq_idx[srt:end] = i

        # for cont batch
        if IND_E is None:
            IND_S = [0 for _ in range(len(spec))]
        else:
            IND_S = [x % full_length for x in IND_E]
        IND_E = [end_boundary(x + y) for x, y in zip(IND_S, spec)]

        # varlen has implicit batch=1
        dt2 = dt2.squeeze(0)
        X2 = X2.squeeze(0)
        B2 = B2.squeeze(0)
        C2 = C2.squeeze(0)
        yield (
            [Y_min[s, IND_S[s] : IND_E[s]] for s in range(num_examples)]
            if return_naive_ref
            else None,
            cu_seqlens,
            seq_idx,
            (A, dt2, X2, B2, C2),
        )
```
**EN:** This helper function implements the shared logic for generate continuous batched examples. It keeps the surrounding module logic factored into a reusable unit. Only the key portions are shown here because the block is large.
**CN:** 该辅助函数实现了 generate continuous batched examples 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。 由于该代码块较大，这里只展示关键片段。

### Function `test_mamba_chunk_scan_single_example` (lines 191-254)
```python
@pytest.mark.parametrize("itype", [torch.float32, torch.bfloat16])
@pytest.mark.parametrize("n_heads", [4, 16, 32])
@pytest.mark.parametrize("d_head", [5, 8, 32, 128])
@pytest.mark.parametrize("seq_len_chunk_size", [(112, 16), (128, 32)])
def test_mamba_chunk_scan_single_example(d_head, n_heads, seq_len_chunk_size, itype):
    # this tests the kernels on a single example (bs=1)

    # TODO: the bfloat16 case requires higher thresholds. To be investigated

    if itype == torch.bfloat16:
        atol, rtol = 5e-2, 5e-2
    else:
        atol, rtol = 8e-3, 5e-3

    # set seed
    batch_size = 1  # batch_size
    # ssd_minimal_discrete requires chunk_size divide seqlen
    # - this is only required for generating the reference seqs,
    #   it is not an operational limitation.
    seqlen, chunk_size = seq_len_chunk_size

    A, dt, X, B, C = generate_random_inputs(batch_size, seqlen, n_heads, d_head, itype)

    Y_min, final_state_min = ssd_minimal_discrete(
        X * dt.unsqueeze(-1), A * dt, B, C, chunk_size
    )

    cu_seqlens = torch.tensor((0, seqlen), device="cuda").cumsum(dim=0)
    cu_chunk_seqlens, last_chunk_indices, seq_idx_chunks = (
        compute_varlen_chunk_metadata(cu_seqlens, chunk_size)
    )
    # varlen has implicit batch=1
    X = X.squeeze(0)
    dt = dt.squeeze(0)
    A = A.squeeze(0)
    B = B.squeeze(0)
    C = C.squeeze(0)
    Y = torch.empty_like(X)
    final_state = mamba_chunk_scan_combined_varlen(
        X,
        dt,
        A,
        B,
        C,
        chunk_size,
        cu_seqlens=cu_seqlens.to(torch.int32),
        cu_chunk_seqlens=cu_chunk_seqlens,
        last_chunk_indices=last_chunk_indices,
        seq_idx=seq_idx_chunks,
        out=Y,
        D=None,
    )

    # just test the last in sequence
    torch.testing.assert_close(Y[-1], Y_min[0, -1], atol=atol, rtol=rtol)

    # just test the last head
    # NOTE, in the kernel we always cast states to fp32
    torch.testing.assert_close(
        final_state[:, -1].to(torch.float32),
        final_state_min[:, -1].to(torch.float32),
        atol=atol,
        rtol=rtol,
    )
```
**EN:** This pytest case verifies mamba chunk scan single example. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as d_head, n_heads, seq_len_chunk_size, itype. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 mamba chunk scan single example 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 d_head、n_heads、seq_len_chunk_size、itype 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较。

### Function `test_mamba_chunk_scan_cont_batch` (lines 257-346)
```python
@pytest.mark.parametrize("itype", [torch.float32])
@pytest.mark.parametrize("n_heads", [4, 8])
@pytest.mark.parametrize("d_head", [5, 16, 32])
@pytest.mark.parametrize(
    "seq_len_chunk_size_cases",
    [
        # small-ish chunk_size (8)
        (64, 8, 2, [(64, 32), (64, 32)]),
        (64, 8, 2, [(8, 8), (8, 8), (8, 8)]),  # chunk size boundary
        (
            64,
            8,
            2,
            [(4, 4), (4, 4), (4, 4), (4, 4)],
        ),  # chunk_size larger than cont batches
        (64, 8, 5, [(64, 32, 16, 8, 8)]),
        # large-ish chunk_size (256)
        (64, 256, 1, [(5,), (1,), (1,), (1,)]),  # irregular sizes with small sequences
        (
            64,
            256,
            2,
            [(5, 30), (1, 2), (1, 2), (1, 2)],
        ),  # irregular sizes with small sequences
        # we also need to test some large seqlen
        # to catch errors with init states decay
        (768, 128, 2, [(138, 225), (138, 225)]),
    ],
)
def test_mamba_chunk_scan_cont_batch(d_head, n_heads, seq_len_chunk_size_cases, itype):
    # this test with multiple examples in a continuous batch
    # (i.e. chunked prefill)

    seqlen, chunk_size, num_examples, cases = seq_len_chunk_size_cases

    # This test can have larger error for longer sequences
    if seqlen > 256:
        atol, rtol = 1e-2, 5e-3
    else:
        atol, rtol = 5e-3, 5e-3
# ... excerpt ...
        B,
        C,
    ) in generate_continuous_batched_examples(
        cases, num_examples, seqlen, last_taken, exhausted, n_heads, d_head, itype
    ):
        cu_chunk_seqlens, last_chunk_indices, seq_idx_chunks = (
            compute_varlen_chunk_metadata(cu_seqlens, chunk_size)
        )

        Y = torch.empty_like(X)
        new_states = mamba_chunk_scan_combined_varlen(
            X,
            dt,
            A,
            B,
            C,
            chunk_size,
            cu_seqlens=cu_seqlens.to(torch.int32),
            cu_chunk_seqlens=cu_chunk_seqlens,
            last_chunk_indices=last_chunk_indices,
            seq_idx=seq_idx_chunks,
            out=Y,
            D=None,
            initial_states=states,
        )

        # just test the last in sequence
        for i in range(num_examples):
            # just test one dim and dstate
            Y_eg = Y[cu_seqlens[i] : cu_seqlens[i + 1], 0, 0]
            Y_min_eg = Y_min[i][:, 0, 0]
            torch.testing.assert_close(Y_eg, Y_min_eg, atol=atol, rtol=rtol)

        # update states
        states = new_states
        for i, clear in exhausted.items():
            if clear:
                states[i].fill_(0.0)
                exhausted[i] = False
```
**EN:** This pytest case verifies mamba chunk scan cont batch. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as d_head, n_heads, seq_len_chunk_size_cases, itype. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 mamba chunk scan cont batch 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 d_head、n_heads、seq_len_chunk_size_cases、itype 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

### Function `test_mamba_chunk_scan_cont_batch_prefill_chunking` (lines 349-569)
```python
@pytest.mark.parametrize("chunk_size", [8, 256])
@pytest.mark.parametrize(
    "seqlens",
    [(16, 20), (270, 88, 212, 203)],
)
def test_mamba_chunk_scan_cont_batch_prefill_chunking(chunk_size, seqlens):
    # This test verifies the correctness of the chunked prefill implementation
    # in the mamba2 ssd kernels, by comparing concatenation (in the sequence
    # dimension) of chunked results with the full sequence result.
    # It is different from test_mamba_chunk_scan_cont_batch by:
    # 1. Not using the naive torch implementation (ssd_minimal_discrete) to get
    #    reference outputs. Instead, it compares chunked kernel outputs to full
    #    sequence kernel outputs. This is the most straightforward way to
    #    assert chunked prefill correctness.
    # 2. It focuses on cases where sequences change in the middle of mamba
    #    chunks, and not necessarily on chunk boundaries.

    max_seqlen = max(seqlens)
    # This test can have larger error for longer sequences
    if max_seqlen > 256:
        atol, rtol = 1e-2, 5e-3
    else:
        atol, rtol = 5e-3, 5e-3

    num_sequences = len(seqlens)
    n_heads = 16
    d_head = 64
    itype = torch.float32

    # hold state during the cutting process so we know if an
    # example has been exhausted and needs to cycle
    last_taken: dict = {}  # map: eg -> pointer to last taken sample
    exhausted: dict = {}  # map: eg -> boolean indicating example is exhausted
    _, cu_seqlens, seq_idx, (A, dt, X, B, C) = next(
        generate_continuous_batched_examples(
            [seqlens],
            num_sequences,
            max_seqlen,
            last_taken,
            exhausted,
# ... excerpt ...
        chunk_size,
        cu_seqlens=remaining_chunked_cu_seqlens.to(torch.int32),
        cu_chunk_seqlens=cu_chunk_seqlens,
        last_chunk_indices=last_chunk_indices,
        seq_idx=seq_idx_chunks,
        out=Y_chunked,
        D=None,
        initial_states=partial_state,
    )
    Y = concat_batch_f(Y_partial, Y_chunked)

    # kernel chunked is same as kernel overall
    for i in range(num_sequences):
        Y_seq = Y[cu_seqlens[i] : cu_seqlens[i + 1], ...]
        Y_ref_seq = Y_ref[cu_seqlens[i] : cu_seqlens[i + 1], ...]
        torch.testing.assert_close(
            Y_seq[: chunked_seqlens[i], ...],
            Y_ref_seq[: chunked_seqlens[i], ...],
            atol=atol,
            rtol=rtol,
            msg=lambda x, i=i: f"seq{i} output part1 " + x,
        )
        torch.testing.assert_close(
            Y_seq[chunked_seqlens[i] :, ...],
            Y_ref_seq[chunked_seqlens[i] :, ...],
            atol=atol,
            rtol=rtol,
            msg=lambda x, i=i: f"seq{i} output part2 " + x,
        )

        state_seq = state_chunked[i]
        state_seq_ref = state_ref[i]
        torch.testing.assert_close(
            state_seq,
            state_seq_ref,
            atol=atol,
            rtol=rtol,
            msg=lambda x, i=i: f"seq{i} state " + x,
        )
```
**EN:** This pytest case verifies mamba chunk scan cont batch prefill chunking. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as chunk_size, seqlens. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 mamba chunk scan cont batch prefill chunking 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 chunk_size、seqlens 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `torch.nn.functional`
- `einops -> rearrange, repeat`
- `vllm.model_executor.layers.mamba.ops.ssd_combined -> mamba_chunk_scan_combined_varlen`
- `vllm.utils.torch_utils -> set_random_seed`
- `vllm.v1.attention.backends.mamba2_attn -> compute_varlen_chunk_metadata`
