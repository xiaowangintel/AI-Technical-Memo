# test_kv_head_stride_canonicalization.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/attention/test_kv_head_stride_canonicalization.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Unit tests for canonicalize_singleton_dim_strides. / 该文件的文档字符串表明其用途：`unit tests for canonicalize_singleton_dim_strides`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 3-23)
```python
"""Unit tests for canonicalize_singleton_dim_strides.

Background
----------
When num_kv_heads_per_rank == 1 (e.g. Qwen3.5-397B with TP=8 → 1 KV head
per rank), PyTorch's is_contiguous() returns True for *any* stride on the
size-1 dimension.  The KV cache allocator can therefore produce a tensor
where that singleton dim has stride = 1 element (2 bytes for bf16) instead
of the canonical product-of-remaining-dims value.

CUDA TMA (used by FlashInfer XQA SM90 and Flash-Attention 3/4 on H100+)
requires all non-outermost strides to be multiples of 16 bytes.  A 2-byte
stride triggers cudaErrorIllegalInstruction.

canonicalize_singleton_dim_strides() patches degenerate strides on all
size-1 dimensions via torch.as_strided — zero-copy.

The degenerate stride manifests at different positions in different backends:
- FlashInfer: stride(-3) after kv_cache.permute() → shape [..., 1, B, D]
- FlashAttention: stride(-2) after kv_cache.unbind(0) → shape [N, B, 1, D]
"""
```
**EN:** Module docstring that declares the scope of the file: Unit tests for canonicalize_singleton_dim_strides.
**CN:** 模块文档字符串直接说明了文件范围：`unit tests for canonicalize_singleton_dim_strides`。

### Imports and setup / 导入与设置 (lines 25-27)
```python
import torch

from vllm.utils.torch_utils import canonicalize_singleton_dim_strides
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `torch`. vLLM modules under test include `vllm.utils.torch_utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `torch`。 被测试的 vLLM 模块包括 `vllm.utils.torch_utils`。

### _inject_degenerate_stride (lines 34-39)
```python
def _inject_degenerate_stride(t: torch.Tensor, dim: int) -> torch.Tensor:
    """Return a view of t with a degenerate (stride=1) on a size-1 dim."""
    assert t.shape[dim] == 1, f"dim {dim} must have size 1"
    strides = list(t.stride())
    strides[dim] = 1  # inject the bug
    return t.as_strided(t.shape, strides)
```
**EN:** Helper function `_inject_degenerate_stride` encapsulates reusable logic for `inject degenerate stride`. Inputs: `t, dim`. Key calls include `list, t.as_strided, t.stride`. It includes 1 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_inject_degenerate_stride` 封装了与 `inject degenerate stride` 相关的可复用逻辑。 输入参数：`t, dim`。 关键调用包括 `list, t.as_strided, t.stride`。 其中包含 1 个内部断言，用于保护前置假设。

### TestCanonicalizeSingletonDimStrides (lines 47-162)
```python
class TestCanonicalizeSingletonDimStrides:
    def test_flashinfer_layout_dim_neg3(self):
        """FlashInfer path: degenerate stride at dim -3 (num_kv_heads)."""
        # Shape after permute: [num_blocks, 2, num_kv_heads, block_size, head_size]
        num_blocks, block_size, head_size = 64, 16, 128
        t = torch.zeros(num_blocks, 2, 1, block_size, head_size, dtype=torch.bfloat16)
        t_deg = _inject_degenerate_stride(t, dim=-3)

        assert t_deg.stride(-3) == 1  # confirm degenerate
        assert t_deg.is_contiguous()  # PyTorch doesn't notice
        fixed = canonicalize_singleton_dim_strides(t_deg)
        assert fixed.stride(-3) == block_size * head_size  # canonical = 2048
        assert fixed.stride(-2) == head_size  # inner dims unchanged
        assert fixed.stride(-1) == 1
    def test_flash_attn_layout_dim_neg2(self):
    # ... excerpt omitted for brevity ...
        assert t_deg.stride(-2) == 1
        assert t_deg.is_contiguous()
        assert fixed.stride(-2) == head_size  # canonical = 128
        assert result is t
        assert result.stride() == original_strides
        # Outer stride should be unchanged (not a size-1 dim)
        assert fixed.stride(0) == t_sliced.stride(0)
        # Inner degenerate stride should be fixed
        assert fixed.stride(-3) == 16 * 128
```
**EN:** Class `TestCanonicalizeSingletonDimStrides` groups 10 test method(s). Representative scenarios: `test_flashinfer_layout_dim_neg3, test_flash_attn_layout_dim_neg2, test_canonical_strides_returned_as_is, test_multi_kv_heads_unchanged, test_data_pointer_preserved, test_multiple_singleton_dims, ...`.
**CN:** 类 `TestCanonicalizeSingletonDimStrides` 组织了 10 个测试方法。 代表性场景：`test_flashinfer_layout_dim_neg3, test_flash_attn_layout_dim_neg2, test_canonical_strides_returned_as_is, test_multi_kv_heads_unchanged, test_data_pointer_preserved, test_multiple_singleton_dims, ...`。

## Key Concepts / 关键概念
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖

## Dependencies / 依赖关系
- **EN:** External libraries: `torch`.
- **CN:** 外部库：`torch`。
- **EN:** vLLM modules under test: `vllm.utils.torch_utils`.
- **CN:** 被测试的 vLLM 模块：`vllm.utils.torch_utils`。
