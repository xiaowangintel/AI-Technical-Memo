# test_deepgemm_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/attention/test_deepgemm_attention.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / attention / test_deepgemm_attention, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / attention / test_deepgemm_attention 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-18)
```python
import random

import pytest
import torch

from vllm.platforms import current_platform
from vllm.utils.deep_gemm import (
    _ceil_to_ue8m0,
    calc_diff,
    fp8_fp4_mqa_logits,
    fp8_fp4_paged_mqa_logits,
    get_num_sms,
    get_paged_mqa_logits_metadata,
)
from vllm.utils.import_utils import has_deep_gemm
from vllm.utils.math_utils import cdiv
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as random, pytest, torch; and vLLM components like vllm.platforms, vllm.utils.deep_gemm, vllm.utils.import_utils, vllm.utils.math_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 random、pytest、torch；vLLM 内部组件，例如 vllm.platforms、vllm.utils.deep_gemm、vllm.utils.import_utils、vllm.utils.math_utils。

### Function `kv_cache_cast_to_fp8` (lines 21-39)
```python
def kv_cache_cast_to_fp8(x: torch.Tensor) -> torch.Tensor:
    # x: (num_blocks, block_size, 1, head_dim)
    num_blocks, block_size, num_heads, head_dim = x.shape
    assert num_heads == 1
    x_amax = x.abs().float().amax(dim=3, keepdim=True).clamp(1e-4)
    sf = x_amax / 448.0
    x_scaled = (x * (1.0 / sf)).to(torch.float8_e4m3fn)
    x_fp8 = torch.empty(
        (num_blocks, block_size * (head_dim + 4)),
        device=x.device,
        dtype=torch.uint8,
    )
    x_fp8[:, : block_size * head_dim] = x_scaled.view(
        num_blocks, block_size * head_dim
    ).view(dtype=torch.uint8)
    x_fp8[:, block_size * head_dim :] = sf.view(num_blocks, block_size).view(
        dtype=torch.uint8
    )
    return x_fp8.view(num_blocks, block_size, num_heads, head_dim + 4)
```
**EN:** This helper function implements the shared logic for KV cache cast to FP8. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 KV cache cast to FP8 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `per_custom_dims_cast_to_fp8` (lines 42-50)
```python
def per_custom_dims_cast_to_fp8(
    x: torch.Tensor, dims: tuple, use_ue8m0: bool
) -> tuple[torch.Tensor, torch.Tensor]:
    excluded_dims = tuple([i for i in range(x.dim()) if i not in set(dims)])
    x_amax = x.abs().float().amax(dim=excluded_dims, keepdim=True).clamp(1e-4)
    sf = x_amax / 448.0
    sf = _ceil_to_ue8m0(sf) if use_ue8m0 else sf
    x_scaled = (x * (1.0 / sf)).to(torch.float8_e4m3fn)
    return x_scaled, sf.squeeze()
```
**EN:** This helper function implements the shared logic for per custom dims cast to FP8. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 per custom dims cast to FP8 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `_generate_cp_test_data` (lines 53-63)
```python
def _generate_cp_test_data(seq_len: int, seq_len_kv: int):
    assert seq_len_kv % seq_len == 0 and seq_len % 2 == 0
    chunk_size = seq_len // 2
    cp_size = seq_len_kv // seq_len
    cp_id = cp_size // 3
    ks = torch.zeros(seq_len, dtype=torch.int, device="cuda")
    ke = torch.zeros(seq_len, dtype=torch.int, device="cuda")
    for i in range(chunk_size):
        ke[i] = cp_id * chunk_size + i
        ke[i + chunk_size] = (cp_size * 2 - 1 - cp_id) * chunk_size + i
    return ks, ke
```
**EN:** This helper function implements the shared logic for generate cp test data. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 generate cp test data 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `_ref_fp8_mqa_logits` (lines 66-90)
```python
def _ref_fp8_mqa_logits(
    q: torch.Tensor,
    kv: torch.Tensor,
    weights: torch.Tensor,
    cu_seqlen_ks: torch.Tensor,
    cu_seqlen_ke: torch.Tensor,
):
    seq_len_kv = kv.shape[0]

    k = kv
    q = q.float()
    k = k.float()

    mask_lo = (
        torch.arange(0, seq_len_kv, device="cuda")[None, :] >= cu_seqlen_ks[:, None]
    )
    mask_hi = (
        torch.arange(0, seq_len_kv, device="cuda")[None, :] < cu_seqlen_ke[:, None]
    )
    mask = mask_lo & mask_hi
    score = torch.einsum("mhd,nd->hmn", q, k)
    logits = (score.relu() * weights.unsqueeze(-1).transpose(0, 1)).sum(dim=0)
    logits = logits.masked_fill(~mask, float("-inf"))

    return logits
```
**EN:** This helper function implements the shared logic for ref FP8 mqa logits. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 ref FP8 mqa logits 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_deepgemm_fp8_mqa_logits` (lines 93-150)
```python
@pytest.mark.skipif(not current_platform.is_cuda(), reason="CUDA only")
@pytest.mark.skipif(not has_deep_gemm(), reason="DeepGEMM not available")
@pytest.mark.skipif(
    not current_platform.has_device_capability(90), reason="SM90 and SM100 only"
)
@pytest.mark.parametrize("clean_logits", [True, False])
def test_deepgemm_fp8_mqa_logits(clean_logits: bool):
    torch.manual_seed(0)
    random.seed(0)
    num_heads, head_dim = 32, 128
    for seq_len in (512,):
        for seq_len_kv in (1024,):
            for disable_cp in (False, True):
                q = torch.randn(
                    seq_len,
                    num_heads,
                    head_dim,
                    device="cuda",
                    dtype=torch.bfloat16,
                )
                kv = torch.randn(
                    seq_len_kv, head_dim, device="cuda", dtype=torch.bfloat16
                )
                weights = torch.randn(
                    seq_len, num_heads, device="cuda", dtype=torch.float32
                )

                if disable_cp:
                    ks = torch.zeros(seq_len, dtype=torch.int, device="cuda")
                    ke = torch.arange(seq_len, dtype=torch.int, device="cuda") + (
                        seq_len_kv - seq_len
                    )
                else:
                    ks, ke = _generate_cp_test_data(seq_len, seq_len_kv)

                q_fp8 = q.to(torch.float8_e4m3fn)
                kv_fp8 = per_custom_dims_cast_to_fp8(kv, (0,), False)
                logits = fp8_fp4_mqa_logits(
                    (q_fp8, None), kv_fp8, weights, ks, ke, clean_logits=clean_logits
                )

                ref_logits = _ref_fp8_mqa_logits(
                    q=q,
                    kv=kv,
                    weights=weights,
                    cu_seqlen_ks=ks,
                    cu_seqlen_ke=ke,
                )
                ref_neginf_mask = ref_logits == float("-inf")

                if clean_logits:
                    neginf_mask = logits == float("-inf")
                    assert torch.equal(neginf_mask, ref_neginf_mask)

                ref_logits = ref_logits.masked_fill(ref_neginf_mask, 0)
                logits = logits.masked_fill(ref_neginf_mask, 0)
                diff = calc_diff(logits, ref_logits)
                assert diff < 1e-3, f"{diff=}"
```
**EN:** This pytest case verifies deepgemm FP8 mqa logits. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as clean_logits. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 deepgemm FP8 mqa logits 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 clean_logits 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Function `_ref_fp8_fp4_paged_mqa_logits` (lines 153-200)
```python
def _ref_fp8_fp4_paged_mqa_logits(
    q: torch.Tensor,
    kv_cache: torch.Tensor,
    weights: torch.Tensor,
    context_lens: torch.Tensor,
    block_tables: torch.Tensor,
    max_model_len: int,
):
    batch_size, next_n, _, _ = q.size()
    _, block_size, _, _ = kv_cache.size()
    logits = torch.full(
        [batch_size * next_n, max_model_len],
        float("-inf"),
        device=q.device,
        dtype=torch.float32,
    )
    context_lens_list = context_lens.tolist()
    for i in range(batch_size):
        context_len = context_lens_list[i]
        q_offsets = torch.arange(context_len - next_n, context_len, device="cuda")
        weight_slice = (
            weights[i * next_n : (i + 1) * next_n, :].transpose(0, 1).contiguous()
        )
        for block_rk in range(cdiv(context_len, block_size)):
            block_idx = block_tables[i][block_rk]
            qx, kx = q[i], kv_cache[block_idx]
            k_offsets = torch.arange(
                block_rk * block_size,
                (block_rk + 1) * block_size,
                device="cuda",
            )
            mask = (k_offsets[None, :] < context_len) & (
                k_offsets[None, :] <= q_offsets[:, None]
            )
            s = torch.where(
                mask[None, :, :],
                (qx.transpose(0, 1) @ kx.transpose(0, 1).transpose(1, 2)).to(
                    logits.dtype
                ),
                float("-inf"),
            )
            s = torch.relu(s) * weight_slice[..., None]
            s = s.sum(dim=0)
            logits[
                i * next_n : (i + 1) * next_n,
                block_rk * block_size : (block_rk + 1) * block_size,
            ] = torch.where(k_offsets[None, :] <= q_offsets[:, None], s, float("-inf"))
    return logits
```
**EN:** This helper function implements the shared logic for ref FP8 fp4 paged mqa logits. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 ref FP8 fp4 paged mqa logits 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_deepgemm_fp8_fp4_paged_mqa_logits` (lines 203-310)
```python
@pytest.mark.skipif(not current_platform.is_cuda(), reason="CUDA only")
@pytest.mark.skipif(not has_deep_gemm(), reason="DeepGEMM not available")
@pytest.mark.skipif(
    not current_platform.has_device_capability(90), reason="SM90 and SM100 only"
)
def test_deepgemm_fp8_fp4_paged_mqa_logits():
    # NOTE: clean_logits=True is incompatible with the 2D context_lens
    # required by csrc/apis/attention.hpp; only the False path is exercised.
    clean_logits = False
    torch.manual_seed(0)
    random.seed(0)

    max_model_len = 4096
    for batch_size, next_n in [(4, 1), (2, 2)]:
        for heads, index_dim in [(32, 128)]:
            for avg_kv in (2048,):
                num_blocks, blocksize = max_model_len * 2, 64

                q = torch.randn(
                    (batch_size, next_n, heads, index_dim),
                    device="cuda",
                    dtype=torch.bfloat16,
                )
                kv_cache = torch.randn(
                    (num_blocks, blocksize, 1, index_dim),
                    device="cuda",
                    dtype=torch.bfloat16,
                )
                weights = torch.randn(
                    (batch_size * next_n, heads),
                    device="cuda",
                    dtype=torch.float32,
                )

                context_lens = (
                    torch.randint(int(0.8 * avg_kv), int(1.2 * avg_kv), (batch_size,))
                    .cuda()
                    .to(torch.int32)
                )
                max_block_len = (
# ... excerpt ...
                    context_lens_2d, blocksize, get_num_sms()
                )
                logits = fp8_fp4_paged_mqa_logits(
                    (q_fp8, None),
                    kv_cache_fp8,
                    weights,
                    context_lens_2d,
                    block_tables,
                    schedule_metadata,
                    max_model_len,
                    clean_logits=clean_logits,
                )

                ref_logits = _ref_fp8_fp4_paged_mqa_logits(
                    q,
                    kv_cache,
                    weights,
                    context_lens,
                    block_tables,
                    max_model_len,
                )

                positions = (
                    torch.arange(max_model_len, device="cuda")
                    .unsqueeze(0)
                    .expand(batch_size * next_n, -1)
                )
                row_indices = torch.arange(batch_size * next_n, device="cuda") // next_n
                next_n_offset = (
                    torch.arange(batch_size * next_n, device="cuda") % next_n
                )
                mask = positions <= (
                    context_lens[row_indices] - next_n + next_n_offset
                ).unsqueeze(1)

                logits = logits.masked_fill(~mask, 0)
                ref_logits = ref_logits.masked_fill(~mask, 0)
                diff = calc_diff(logits, ref_logits)
                assert diff < 1e-3, f"{diff=}"
```
**EN:** This pytest case verifies deepgemm FP8 fp4 paged mqa logits. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 deepgemm FP8 fp4 paged mqa logits 的行为。 不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `random`
- `pytest`
- `torch`
- `vllm.platforms -> current_platform`
- `vllm.utils.deep_gemm -> _ceil_to_ue8m0, calc_diff, fp8_fp4_mqa_logits, fp8_fp4_paged_mqa_logits, get_num_sms, get_paged_mqa_logits_metadata`
- `vllm.utils.import_utils -> has_deep_gemm`
- `vllm.utils.math_utils -> cdiv`
