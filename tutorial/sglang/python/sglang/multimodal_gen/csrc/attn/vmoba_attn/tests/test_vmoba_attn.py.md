# test_vmoba_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/csrc/attn/vmoba_attn/tests/test_vmoba_attn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the attention kernel layer. It centers on `generate_test_data`, and `test_moba_attn_varlen_forward`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于注意力内核层。它围绕 `generate_test_data` 和 `test_moba_attn_varlen_forward` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 3-7: module setup and imports / 模块初始化与导入
```python
import random

import pytest
import torch
from sglang.multimodal_gen.csrc.attn.vmoba_attn.vmoba import moba_attn_varlen
```
**EN:** This block establishes the module context and imports `random`, `pytest`, `torch`, and `sglang.multimodal_gen.csrc.attn.vmoba_attn.vmoba`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `random`、`pytest`、`torch` 和 `sglang.multimodal_gen.csrc.attn.vmoba_attn.vmoba`。这些依赖为后续实现提供所需符号。

### Lines 9-68: `generate_test_data` implementation / `generate_test_data` 实现
```python
def generate_test_data(
    batch_size, total_seqlen, num_heads, head_dim, dtype, device="cuda"
):
    """
    Generates random data for testing the variable-length attention function.
    """
    torch.manual_seed(42)
    random.seed(42)
    torch.cuda.manual_seed_all(42)

    # Generate sequence lengths for each item in the batch
    if batch_size > 1:
        # Ensure sequence lengths are reasonably distributed
        avg_seqlen = total_seqlen // batch_size
        seqlens = [
            random.randint(avg_seqlen // 2, avg_seqlen + avg_seqlen // 2)
            for _ in range(batch_size - 1)
        ]
        remaining_len = total_seqlen - sum(seqlens)
        if remaining_len > 0:
            seqlens.append(remaining_len)
        else:  # Adjust if sum exceeds total_seqlen
            seqlens.append(avg_seqlen)
            current_sum = sum(seqlens)
            seqlens[-1] -= current_sum - total_seqlen
        # Ensure all lengths are positive
        seqlens = [max(1, s) for s in seqlens]
        # Final adjustment to match total_seqlen
        seqlens[-1] += total_seqlen - sum(seqlens)

    else:
        seqlens = [total_seqlen]

    cu_seqlens = torch.tensor(
        [0] + list(torch.cumsum(torch.tensor(seqlens), 0)),
        device=device,
        dtype=torch.int32,
    )
    max_seqlen = max(seqlens) if seqlens else 0

    q = torch.randn(
        (total_seqlen, num_heads, head_dim),
        dtype=dtype,
        device=device,
        requires_grad=False,
    )
    k = torch.randn(
        (total_seqlen, num_heads, head_dim),
        dtype=dtype,
        device=device,
        requires_grad=False,
    )
    v = torch.randn(
        (total_seqlen, num_heads, head_dim),
        dtype=dtype,
        device=device,
        requires_grad=False,
    )

    return q, k, v, cu_seqlens, max_seqlen
```
**EN:** This block defines function `generate_test_data`. Generates random data for testing the variable-length attention function. Key calls include `torch.manual_seed`, `random.seed`, `torch.cuda.manual_seed_all`, `torch.tensor`, and `torch.randn`. The implementation branches on conditions. Parameters such as `batch_size`, `total_seqlen`, `num_heads`, `head_dim`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了函数 `generate_test_data`。 它用于生成test data。 关键调用包括 `torch.manual_seed`、`random.seed`、`torch.cuda.manual_seed_all`、`torch.tensor` 和 `torch.randn`。 实现中包含条件分支。 本段逻辑主要由 `batch_size`、`total_seqlen`、`num_heads`、`head_dim` 和 `dtype` 等参数驱动。

### Lines 71-136: `test_moba_attn_varlen_forward` implementation / `test_moba_attn_varlen_forward` 实现
```python
@pytest.mark.parametrize("batch_size", [1, 2])
@pytest.mark.parametrize("total_seqlen", [512, 1024])
@pytest.mark.parametrize("num_heads", [8])
@pytest.mark.parametrize("head_dim", [64])
@pytest.mark.parametrize("moba_chunk_size", [64])
@pytest.mark.parametrize("moba_topk", [2, 4])
@pytest.mark.parametrize("select_mode", ["topk", "threshold"])
@pytest.mark.parametrize("threshold_type", ["query_head", "head_global", "overall"])
@pytest.mark.parametrize("dtype", [torch.float32, torch.float16, torch.bfloat16])
def test_moba_attn_varlen_forward(
    batch_size,
    total_seqlen,
    num_heads,
    head_dim,
    moba_chunk_size,
    moba_topk,
    select_mode,
    threshold_type,
    dtype,
):
    """
    Tests the forward pass of moba_attn_varlen for basic correctness.
    It checks output shape, dtype, and for the presence of NaNs/Infs.
    """
    if dtype == torch.float32:
        pytest.skip("float32 is not supported in flash attention")

    q, k, v, cu_seqlens, max_seqlen = generate_test_data(
        batch_size, total_seqlen, num_heads, head_dim, dtype
    )

    # Ensure chunk size is not larger than the smallest sequence length
    min_seqlen = (cu_seqlens[1:] - cu_seqlens[:-1]).min().item()
    if moba_chunk_size > min_seqlen:
        pytest.skip(
            "moba_chunk_size is larger than the minimum sequence length in the batch"
        )

    try:
        output = moba_attn_varlen(
            q=q,
            k=k,
            v=v,
            cu_seqlens=cu_seqlens,
            max_seqlen=max_seqlen,
            moba_chunk_size=moba_chunk_size,
            moba_topk=moba_topk,
            select_mode=select_mode,
            threshold_type=threshold_type,
            simsum_threshold=0.5,  # A reasonable default for threshold mode
        )
    except Exception as e:
        pytest.fail(f"moba_attn_varlen forward pass failed with exception: {e}")

    # 1. Check output shape
    assert (
        output.shape == q.shape
    ), f"Expected output shape {q.shape}, but got {output.shape}"

    # 2. Check output dtype
    assert (
        output.dtype == q.dtype
    ), f"Expected output dtype {q.dtype}, but got {output.dtype}"

    # 3. Check for NaNs or Infs in the output
    assert torch.all(torch.isfinite(output)), "Output contains NaN or Inf values"
```
**EN:** This block defines function `test_moba_attn_varlen_forward`. Tests the forward pass of moba_attn_varlen for basic correctness. It checks output shape, dtype, and for the presence of NaNs/Infs. Key calls include `pytest.mark.parametrize`, `generate_test_data`, `min.item`, `torch.all`, and `pytest.skip`. The implementation branches on conditions, handles exceptional paths. Parameters such as `batch_size`, `total_seqlen`, `num_heads`, `head_dim`, and `moba_chunk_size` drive the behavior in this section.
**CN:** 该代码块定义了函数 `test_moba_attn_varlen_forward`。 它用于处理 test moba attn varlen forward 相关逻辑。 关键调用包括 `pytest.mark.parametrize`、`generate_test_data`、`min.item`、`torch.all` 和 `pytest.skip`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `batch_size`、`total_seqlen`、`num_heads`、`head_dim` 和 `moba_chunk_size` 等参数驱动。

## Key Concepts / 关键概念
- `generate_test_data`: Generates random data for testing the variable-length attention function. / 顶层函数，用于生成test data。
- `test_moba_attn_varlen_forward`: Tests the forward pass of moba_attn_varlen for basic correctness. / 顶层函数，用于处理 test moba attn varlen forward 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `random`
- **Third-party / 第三方依赖**: `pytest`, `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.csrc.attn.vmoba_attn.vmoba`

- **Total lines / 总行数**: 136
