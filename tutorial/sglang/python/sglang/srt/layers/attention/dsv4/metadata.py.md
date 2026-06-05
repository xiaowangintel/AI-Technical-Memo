# metadata.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/dsv4/metadata.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines the metadata structures that carry execution metadata through the attention pipeline. / 该模块定义 metadata 结构，用于在注意力流水线中传递执行元数据。
## Line-by-Line Analysis / 逐行分析
### Lines 1-10: imports
```python
from __future__ import annotations

import warnings
from dataclasses import dataclass, field, fields
from typing import TYPE_CHECKING, Any, List, Optional

import torch

from sglang.srt.environ import envs
from sglang.srt.utils import is_hip
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 12-13: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    pass
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 16-50: docstring
```python
"""
Some comments on the common terms used in DeepSeekV4Backend:

topk_lengths:
    NOTE: TL;DR: topk_lengths == seq_lens
    The FlashMLA sparse decode kernel will attend to `k` tokens for each query.
    `topk_lengths` indicates how many tokens each query will attend to.
    This should be named as `seq_lens`, but we simply follow the naming convention.

page_table:
    The page table indicates which pages each request is assigned to.
    Each value in the page table is the page index in the TokenToKVPool.
    This page index is irrelevant to the actual `page_size`.

page_indices:
    The real indices used to index into the KV cache.
    This can be computed from the `page_table` and `page_size`.
    e.g. page_indices[i, j] = page_table[i, j // page_size] * page_size + (j % page_size)
    For sparse C4 top-512 attention, the indices will be selected from the C4 page indices.
    In implementation, we don't materialize the full C4 `page_indices`,
    but calculate them from `page_table` on-the-fly in the attention kernel.

positions:
    The position of the last token for each request.
    For compress token, the positions must be times of compress ratio.
    For example, for C4, raw_position=11 will trigger a compression,
    But the RoPE's position, during compression, must be 8 instead of 11.

Some other notes:
    c4_ / c128_: means "compressed by 4" / "compressed by 128".
    c4_page_size: page_size // 4
    c4_seq_lens: seq_lens // 4, but bounded by at least 1, due to flash_mla requirement.
    c4_sparse: means "compressed by 4" but only attend to top-512 tokens.
               all related length will be clipped to 512.
"""
```
**EN:** Provides inline documentation that explains the scope of the surrounding module or class.
**CN:** 提供内联文档，用于说明周围模块或类的职责范围。

### Lines 53-94: function copy_metadata
```python
def copy_metadata(
    *,
    src,
    dst,
    check_eq_fields: List[str],
    copy_fields: List[str],
    assign_fields: Optional[List[str]] = None,
):
    assign_fields = assign_fields or []

    for field_name in check_eq_fields:
        src_val = getattr(src, field_name)
        dst_val = getattr(dst, field_name)
        assert src_val == dst_val, f"{field_name=} {src_val=} {dst_val=}"

    for field_name in copy_fields:
        src_val = getattr(src, field_name)
        dst_val = getattr(dst, field_name)
        if src_val is None and dst_val is None:
            continue
        assert dst_val is not None, f"{field_name=} {src_val=} {dst_val=}"
        if hasattr(dst_val, "copy_"):
            dst_val.copy_(src_val)
        else:
            warnings.warn(
                f"{field_name=} {type(dst_val)=} does not have copy_, use setattr"
            )
            setattr(dst, field_name, src_val)

    for field_name in assign_fields:
        setattr(dst, field_name, getattr(src, field_name))

    provided_fields = check_eq_fields + copy_fields + assign_fields
    provided_fields_unique = set(provided_fields)
    assert len(provided_fields) == len(
        provided_fields_unique
    ), f"{provided_fields=} has dup"
    all_fields = {f.name for f in fields(src)}
    provided_fields = set(provided_fields)
    assert (
        provided_fields == all_fields
    ), f"{provided_fields - all_fields=}, {all_fields - provided_fields=}"
```
**EN:** Implements the copy metadata routine used by this attention module.
**CN:** 实现该注意力模块使用的 copy metadata 例程。

### Lines 97-104: class PagedIndexerMetadata
```python
@dataclass
class PagedIndexerMetadata:
    page_size: int
    page_table: torch.Tensor
    c4_seq_lens: torch.Tensor
    deep_gemm_metadata: Any = field(init=False, repr=False)
    topk_metadata: torch.Tensor = field(init=False, repr=False)
```
**EN:** Dataclass-style container that stores structured runtime state for paged indexer metadata.
**CN:** 该数据类风格的容器用于存储 paged indexer metadata 的结构化运行时状态。

### Lines 105-134: method PagedIndexerMetadata.__post_init__
```python
    def __post_init__(self):
        if envs.SGLANG_FP8_PAGED_MQA_LOGITS_TORCH.get():
            self.deep_gemm_metadata = None
        else:
            import deep_gemm

            if envs.SGLANG_OPT_USE_JIT_INDEXER_METADATA.get():
                from sglang.jit_kernel.deepseek_v4 import get_paged_mqa_logits_metadata
            else:
                from deep_gemm import get_paged_mqa_logits_metadata

            _c4 = self.c4_seq_lens.to(torch.int32)
            if _c4.dim() == 1:
                _c4 = _c4.unsqueeze(-1)
            self.deep_gemm_metadata = get_paged_mqa_logits_metadata(
                _c4,
                self.c4_page_size,
                deep_gemm.get_num_sms(),
            )

            assert isinstance(self.deep_gemm_metadata, torch.Tensor)

        from sglang.jit_kernel.deepseek_v4 import plan_topk_v2

        if envs.SGLANG_OPT_USE_TOPK_V2.get():
            self.topk_metadata = plan_topk_v2(self.c4_seq_lens)
        else:
            self.topk_metadata = torch.empty((0,))

        assert self.page_size == 256, "the system hardcodes page_size=256"
```
**EN:** Implements the post init routine used by this attention module.
**CN:** 实现该注意力模块使用的 post init 例程。

### Lines 136-138: method PagedIndexerMetadata.c4_page_size
```python
    @property
    def c4_page_size(self) -> int:
        return self.page_size // 4
```
**EN:** Implements the c4 page size routine used by this attention module.
**CN:** 实现该注意力模块使用的 c4 page size 例程。

### Lines 140-142: method PagedIndexerMetadata.max_seq_len
```python
    @property
    def max_seq_len(self) -> int:
        return self.page_table.shape[1] * self.page_size
```
**EN:** Implements the max seq len routine used by this attention module.
**CN:** 实现该注意力模块使用的 max seq len 例程。

### Lines 144-146: method PagedIndexerMetadata.max_c4_seq_len
```python
    @property
    def max_c4_seq_len(self) -> int:
        return self.page_table.shape[1] * self.c4_page_size
```
**EN:** Implements the max c4 seq len routine used by this attention module.
**CN:** 实现该注意力模块使用的 max c4 seq len 例程。

### Lines 148-159: method PagedIndexerMetadata.copy_
```python
    def copy_(self, other: "PagedIndexerMetadata"):
        if is_hip():
            copy_fields = ["page_table", "c4_seq_lens"]
        else:
            copy_fields = ["page_table", "c4_seq_lens", "deep_gemm_metadata"]
        copy_fields += ["topk_metadata"]
        copy_metadata(
            src=other,
            dst=self,
            check_eq_fields=["page_size"],
            copy_fields=copy_fields,
        )
```
**EN:** Implements the copy routine used by this attention module.
**CN:** 实现该注意力模块使用的 copy 例程。

### Lines 162-165: function maybe_copy_inplace
```python
def maybe_copy_inplace(dst, *, src) -> None:
    assert type(src) == type(dst)
    if dst is not None:
        dst.copy_(src)
```
**EN:** Implements the maybe copy inplace routine used by this attention module.
**CN:** 实现该注意力模块使用的 maybe copy inplace 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Flash-style fused attention kernels / **CN:** Flash 风格融合注意力内核
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** Paged attention layouts / **CN:** 分页注意力布局
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理
- **EN:** Position encoding handling / **CN:** 位置编码处理

## Dependencies / 依赖关系
- `__future__.annotations`
- `warnings`
- `dataclasses.dataclass`
- `dataclasses.field`
- `dataclasses.fields`
- `typing.TYPE_CHECKING`
- `typing.Any`
- `typing.List`
- `typing.Optional`
- `torch`
- `sglang.srt.environ.envs`
- `sglang.srt.utils.is_hip`
- `sglang.jit_kernel.deepseek_v4.plan_topk_v2`
- `deep_gemm`
- `sglang.jit_kernel.deepseek_v4.get_paged_mqa_logits_metadata`
- `deep_gemm.get_paged_mqa_logits_metadata`
