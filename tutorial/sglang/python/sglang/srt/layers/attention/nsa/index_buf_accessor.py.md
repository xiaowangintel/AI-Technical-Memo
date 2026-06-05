# index_buf_accessor.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/nsa/index_buf_accessor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module manages index buf accessor logic around KV-cache layout, indexing, or paging for SGLang attention execution. / 该模块管理与 index buf accessor 相关的 KV 缓存布局、索引或分页逻辑，用于 SGLang 注意力执行。
## Line-by-Line Analysis / 逐行分析
### Lines 1-9: imports
```python
from typing import TYPE_CHECKING

import torch
import triton
import triton.language as tl

from sglang.srt.layers.attention.nsa.utils import aiter_can_use_preshuffle_paged_mqa
from sglang.srt.layers.quantization.fp8_kernel import is_fp8_fnuz
from sglang.srt.utils import get_bool_env_var, is_hip
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 11-17: module constants
```python
_is_hip = is_hip()
_is_fp8_fnuz = is_fp8_fnuz()
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
# aiter cp_gather kernel with preshuffle=True is only valid when the indexer
# uses the page_size=64 preshuffle layout (i.e. when the matching MQA gluon path
# is also enabled).
_use_aiter_preshuffle = aiter_can_use_preshuffle_paged_mqa()
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 19-20: conditional branch
```python
if _use_aiter_preshuffle:
    from aiter.ops.cache import cp_gather_indexer_k_quant_cache
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 22-23: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    from sglang.srt.mem_cache.memory_pool import NSATokenToKVPool
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 25-28: docstring
```python
"""
k: data, 128 item per token, fp8
s: scale, 1 item per token, fp32
"""
```
**EN:** Provides inline documentation that explains the scope of the surrounding module or class.
**CN:** 提供内联文档，用于说明周围模块或类的职责范围。

### Lines 31-31: class GetK
```python
class GetK:
```
**EN:** Defines the get k type and the state it exposes to the rest of the attention stack.
**CN:** 定义 get k 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 32-34: method GetK.execute
```python
    @classmethod
    def execute(cls, *args, **kwargs):
        return cls.triton(*args, **kwargs)
```
**EN:** Implements the execute routine used by this attention module.
**CN:** 实现该注意力模块使用的 execute 例程。

### Lines 36-53: method GetK.slow
```python
    @classmethod
    def slow(
        cls, pool: "NSATokenToKVPool", buf, seq_len: int, page_indices: torch.Tensor
    ):
        num_pages = (seq_len + pool.page_size - 1) // pool.page_size
        seq_len_ = num_pages * pool.page_size
        index_k_fp8 = torch.empty(
            (seq_len_, pool.index_head_dim),
            dtype=torch.uint8,
            device=pool.device,
        )
        for i in range(num_pages):
            page_index = page_indices[i]
            index_k_fp8[i * pool.page_size : (i + 1) * pool.page_size] = buf[
                page_index
            ][: pool.page_size * pool.index_head_dim].view(-1, pool.index_head_dim)

        return index_k_fp8[:seq_len]
```
**EN:** Implements the slow routine used by this attention module.
**CN:** 实现该注意力模块使用的 slow 例程。

### Lines 55-83: method GetK.torch_fast
```python
    @classmethod
    def torch_fast(
        cls, pool: "NSATokenToKVPool", buf, seq_len: int, page_indices: torch.Tensor
    ):
        """
        :param page_indices: (num_pages,), int32
        :return: (seq_len, index_head_dim), uint8
        """

        # can handle per 128B instead of per element

        # page_indices: (num_pages,), element := a page index
        buf_numel_per_page = buf.shape[1]

        num_k_bytes_per_page = pool.page_size * pool.index_head_dim
        num_k_bytes_per_token = pool.index_head_dim

        # buf: (num_pages, page_size 64 * head_dim 128 + page_size 64 * fp32_nbytes 4), uint8
        # flat_buf: (whatever,), uint8
        flat_buf = buf.flatten()

        # flat_indices: (num_pages, num_k_bytes_per_page), int32, element := an index into flat_buf that we want to access
        flat_indices = (page_indices * buf_numel_per_page)[:, None] + torch.arange(
            num_k_bytes_per_page, dtype=torch.int32, device="cuda"
        )[None, :]
        flat_indices = flat_indices.flatten()[: seq_len * num_k_bytes_per_token]

        out = flat_buf[flat_indices]
        return out.view(-1, 128)
```
**EN:** Implements the torch fast routine used by this attention module.
**CN:** 实现该注意力模块使用的 torch fast 例程。

### Lines 85-100: method GetK.triton
```python
    @classmethod
    def triton(
        cls, pool: "NSATokenToKVPool", buf, seq_len: int, page_indices: torch.Tensor
    ):
        """
        Triton implementation for gathering K data from paged buffer.
        :param page_indices: (num_pages,), int32/int64
        :return: (seq_len, index_head_dim), uint8
        """
        return _get_k_triton(
            buf=buf,
            page_indices=page_indices,
            seq_len=seq_len,
            page_size=pool.page_size,
            index_head_dim=pool.index_head_dim,
        )
```
**EN:** Implements the triton routine used by this attention module.
**CN:** 实现该注意力模块使用的 triton 例程。

### Lines 103-103: class GetS
```python
class GetS:
```
**EN:** Defines the get s type and the state it exposes to the rest of the attention stack.
**CN:** 定义 get s 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 104-106: method GetS.execute
```python
    @classmethod
    def execute(cls, *args, **kwargs):
        return cls.triton(*args, **kwargs)
```
**EN:** Implements the execute routine used by this attention module.
**CN:** 实现该注意力模块使用的 execute 例程。

### Lines 108-125: method GetS.slow
```python
    @classmethod
    def slow(
        cls, pool: "NSATokenToKVPool", buf, seq_len: int, page_indices: torch.Tensor
    ):
        num_pages = (seq_len + pool.page_size - 1) // pool.page_size
        seq_len_ = num_pages * pool.page_size
        assert pool.index_head_dim // pool.quant_block_size == 1
        index_k_scale_fp8 = torch.empty(
            (seq_len_, 4),
            dtype=torch.uint8,
            device=pool.device,
        )
        for i in range(num_pages):
            page_index = page_indices[i]
            index_k_scale_fp8[i * pool.page_size : (i + 1) * pool.page_size] = buf[
                page_index
            ][pool.page_size * pool.index_head_dim :].view(-1, 4)
        return index_k_scale_fp8[:seq_len]
```
**EN:** Implements the slow routine used by this attention module.
**CN:** 实现该注意力模块使用的 slow 例程。

### Lines 127-152: method GetS.torch_fast
```python
    @classmethod
    def torch_fast(
        cls, pool: "NSATokenToKVPool", buf, seq_len: int, page_indices: torch.Tensor
    ):
        """
        :param page_indices: (num_pages,), int32
        :return: (seq_len, index_head_dim // quant_block_size), uint8
        """
        buf_numel_per_page = buf.shape[1]

        num_s_bytes_per_page = buf.shape[1] - pool.page_size * pool.index_head_dim
        num_s_bytes_per_token = pool.index_head_dim // pool.quant_block_size * 4
        s_offset_in_page = pool.page_size * pool.index_head_dim

        flat_buf = buf.flatten()
        flat_indices = (
            (page_indices * buf_numel_per_page)[:, None]
            + torch.arange(num_s_bytes_per_page, dtype=torch.int32, device="cuda")[
                None, :
            ]
            + s_offset_in_page
        )
        flat_indices = flat_indices.flatten()[: seq_len * num_s_bytes_per_token]

        out = flat_buf[flat_indices]
        return out.view(-1, 4)
```
**EN:** Implements the torch fast routine used by this attention module.
**CN:** 实现该注意力模块使用的 torch fast 例程。

### Lines 154-169: method GetS.triton
```python
    @classmethod
    def triton(
        cls, pool: "NSATokenToKVPool", buf, seq_len: int, page_indices: torch.Tensor
    ):
        """
        Triton implementation for gathering S (scale) data from paged buffer.
        :param page_indices: (num_pages,), int32/int64
        :return: (seq_len, 4), uint8
        """
        return _get_s_triton(
            buf=buf,
            page_indices=page_indices,
            seq_len=seq_len,
            page_size=pool.page_size,
            index_head_dim=pool.index_head_dim,
        )
```
**EN:** Implements the triton routine used by this attention module.
**CN:** 实现该注意力模块使用的 triton 例程。

### Lines 172-172: class GetKAndS
```python
class GetKAndS:
```
**EN:** Defines the get kand s type and the state it exposes to the rest of the attention stack.
**CN:** 定义 get kand s 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 173-181: method GetKAndS.execute
```python
    @classmethod
    def execute(cls, *args, **kwargs):
        # The aiter path uses cp_gather_indexer_k_quant_cache(preshuffle=True),
        # which only matches the layout produced when the rest of the indexer
        # is on the page_size=64 preshuffle path. Otherwise fall back to the
        # triton implementation (which works on the page_size=1 legacy layout).
        if _use_aiter_preshuffle:
            return cls.aiter(*args, **kwargs)
        return cls.triton(*args, **kwargs)
```
**EN:** Implements the execute routine used by this attention module.
**CN:** 实现该注意力模块使用的 execute 例程。

### Lines 183-223: method GetKAndS.aiter
```python
    @classmethod
    def aiter(
        cls,
        pool: "NSATokenToKVPool",
        buf: torch.Tensor,
        page_indices: torch.Tensor,
        seq_len_tensor: torch.Tensor,
        seq_len_sum: int,
        max_seq_len: int,
    ):
        from sglang.srt.layers.quantization.fp8_kernel import fp8_dtype

        page_size = pool.page_size
        index_head_dim = pool.index_head_dim
        quant_block_size = pool.quant_block_size
        scale_elems = index_head_dim // quant_block_size

        kv_cache = buf.view(-1, page_size, index_head_dim + scale_elems * 4).view(
            fp8_dtype
        )
        dst_k = torch.empty(
            (seq_len_sum, index_head_dim), dtype=torch.uint8, device=buf.device
        )
        dst_scale = torch.empty(
            (seq_len_sum, scale_elems * 4), dtype=torch.uint8, device=buf.device
        )

        cu_seq_lens = torch.zeros(
            seq_len_tensor.shape[0] + 1, dtype=torch.int32, device=buf.device
        )
        torch.cumsum(seq_len_tensor.to(torch.int32), dim=0, out=cu_seq_lens[1:])

        cp_gather_indexer_k_quant_cache(
            kv_cache,
            dst_k.view(fp8_dtype),
            dst_scale,
            page_indices.to(torch.int32),
            cu_seq_lens,
            preshuffle=True,
        )
        return dst_k, dst_scale
```
**EN:** Implements the aiter routine used by this attention module.
**CN:** 实现该注意力模块使用的 aiter 例程。

### Lines 225-253: method GetKAndS.triton
```python
    @classmethod
    def triton(
        cls,
        pool: "NSATokenToKVPool",
        buf: torch.Tensor,
        page_indices: torch.Tensor,
        seq_len_tensor: torch.Tensor,
        seq_len_sum: int,
        max_seq_len: int,
    ):
        """
        Triton implementation for gathering both K and S data from paged buffer in a single call.
        :param page_indices: (num_pages,), int32/int64
        :param seq_len_tensor: (num_pages,), int32/int64
        :param seq_len_sum: sum of all sequence len, int32
        :param max_seq_len: max of all sequence len, int32
        :return: tuple of (k_fp8, k_scale) where
                 k_fp8: (seq_len, index_head_dim), uint8
                 k_scale: (seq_len, 4), uint8
        """
        return _get_k_and_s_triton(
            buf=buf,
            page_indices=page_indices,
            seq_lens=seq_len_tensor,
            seq_len_sum=seq_len_sum,
            max_seq_len=max_seq_len,
            page_size=pool.page_size,
            index_head_dim=pool.index_head_dim,
        )
```
**EN:** Implements the triton routine used by this attention module.
**CN:** 实现该注意力模块使用的 triton 例程。

### Lines 256-256: class SetK
```python
class SetK:
```
**EN:** Defines the set k type and the state it exposes to the rest of the attention stack.
**CN:** 定义 set k 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 257-259: method SetK.execute
```python
    @classmethod
    def execute(cls, *args, buf, **kwargs):
        return cls.torch_fast(*args, **kwargs, buf=buf)
```
**EN:** Implements the execute routine used by this attention module.
**CN:** 实现该注意力模块使用的 execute 例程。

### Lines 261-275: method SetK.slow
```python
    @classmethod
    def slow(
        cls,
        pool: "NSATokenToKVPool",
        buf: torch.Tensor,
        loc: torch.Tensor,
        index_k: torch.Tensor,
    ):
        for i in range(len(loc)):
            page_index = loc[i] // pool.page_size
            offset = loc[i] % pool.page_size
            buf[
                page_index,
                offset * pool.index_head_dim : (offset + 1) * pool.index_head_dim,
            ] = index_k[i].view(torch.uint8)
```
**EN:** Implements the slow routine used by this attention module.
**CN:** 实现该注意力模块使用的 slow 例程。

### Lines 277-303: method SetK.torch_fast
```python
    @classmethod
    def torch_fast(
        cls,
        pool: "NSATokenToKVPool",
        buf: torch.Tensor,
        loc: torch.Tensor,
        index_k: torch.Tensor,
    ):
        (num_tokens_to_write,) = loc.shape
        buf_numel_per_page = buf.shape[1]
        num_k_bytes_per_token = pool.index_head_dim

        # loc: (num_tokens_to_write,), int32, element := the token index to write to
        loc_page_index = loc // pool.page_size
        loc_token_offset_in_page = loc % pool.page_size

        flat_buf = buf.flatten()
        flat_indices = (
            (loc_page_index * buf_numel_per_page)[:, None]
            + (loc_token_offset_in_page * num_k_bytes_per_token)[:, None]
            + torch.arange(num_k_bytes_per_token, dtype=torch.int32, device="cuda")[
                None, :
            ]
        )
        num_k_bytes_total = num_tokens_to_write * num_k_bytes_per_token
        flat_indices = flat_indices.flatten()[:num_k_bytes_total]
        flat_buf[flat_indices] = index_k.view(torch.uint8).flatten()
```
**EN:** Implements the torch fast routine used by this attention module.
**CN:** 实现该注意力模块使用的 torch fast 例程。

### Lines 306-306: class SetS
```python
class SetS:
```
**EN:** Defines the set s type and the state it exposes to the rest of the attention stack.
**CN:** 定义 set s 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 307-309: method SetS.execute
```python
    @classmethod
    def execute(cls, *args, buf, **kwargs):
        return cls.torch_fast(*args, **kwargs, buf=buf)
```
**EN:** Implements the execute routine used by this attention module.
**CN:** 实现该注意力模块使用的 execute 例程。

### Lines 311-325: method SetS.slow
```python
    @classmethod
    def slow(
        cls,
        pool: "NSATokenToKVPool",
        buf: torch.Tensor,
        loc: torch.Tensor,
        index_k_scale: torch.Tensor,
    ):
        for i in range(len(loc)):
            page_index = loc[i] // pool.page_size
            offset = loc[i] % pool.page_size
            start = pool.page_size * pool.index_head_dim
            buf[page_index, start + offset * 4 : start + (offset + 1) * 4] = (
                index_k_scale[i].view(torch.uint8)
            )
```
**EN:** Implements the slow routine used by this attention module.
**CN:** 实现该注意力模块使用的 slow 例程。

### Lines 327-355: method SetS.torch_fast
```python
    @classmethod
    def torch_fast(
        cls,
        pool: "NSATokenToKVPool",
        buf: torch.Tensor,
        loc: torch.Tensor,
        index_k_scale: torch.Tensor,
    ):
        (num_tokens_to_write,) = loc.shape
        buf_numel_per_page = buf.shape[1]
        num_s_bytes_per_token = 4
        s_offset_in_page = pool.page_size * pool.index_head_dim

        # loc: (num_tokens_to_write,), int32, element := the token index to write to
        loc_page_index = loc // pool.page_size
        loc_token_offset_in_page = loc % pool.page_size

        flat_buf = buf.flatten()
        flat_indices = (
            (loc_page_index * buf_numel_per_page)[:, None]
            + s_offset_in_page
            + (loc_token_offset_in_page * num_s_bytes_per_token)[:, None]
            + torch.arange(num_s_bytes_per_token, dtype=torch.int32, device="cuda")[
                None, :
            ]
        )
        number_s_bytes_total = num_tokens_to_write * num_s_bytes_per_token
        flat_indices = flat_indices.flatten()[:number_s_bytes_total]
        flat_buf[flat_indices] = index_k_scale.view(torch.uint8).flatten()
```
**EN:** Implements the torch fast routine used by this attention module.
**CN:** 实现该注意力模块使用的 torch fast 例程。

### Lines 358-358: class SetKAndS
```python
class SetKAndS:
```
**EN:** Defines the set kand s type and the state it exposes to the rest of the attention stack.
**CN:** 定义 set kand s 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 359-378: method SetKAndS.execute
```python
    @classmethod
    def execute(cls, *args, buf, **kwargs):
        if 0:
            # print("SetK, SetS comparison test")
            buf_cloned = buf.clone()
            cls.vanilla(*args, **kwargs, buf=buf)
            cls.triton(*args, **kwargs, buf=buf_cloned)

            def _clear_token_0(target):
                target[0, :128] = target[0, 64 * 128 : 64 * 128 + 4] = 0

            _clear_token_0(buf)
            _clear_token_0(buf_cloned)

            assert torch.all(
                buf == buf_cloned
            ), f"{buf=} {buf_cloned=} {kwargs['loc'].to_list()=}"
            return

        cls.triton(*args, **kwargs, buf=buf)
```
**EN:** Implements the execute routine used by this attention module.
**CN:** 实现该注意力模块使用的 execute 例程。

### Lines 380-383: method SetKAndS.vanilla
```python
    @classmethod
    def vanilla(cls, pool, buf, loc, index_k, index_k_scale):
        SetK.execute(pool=pool, buf=buf, loc=loc, index_k=index_k)
        SetS.execute(pool=pool, buf=buf, loc=loc, index_k_scale=index_k_scale)
```
**EN:** Implements the vanilla routine used by this attention module.
**CN:** 实现该注意力模块使用的 vanilla 例程。

### Lines 385-395: method SetKAndS.triton
```python
    @classmethod
    def triton(cls, pool, buf, loc, index_k, index_k_scale):
        loc = loc.to(torch.int64)

        _set_k_and_s_triton(
            buf=buf,
            loc=loc,
            index_k=index_k,
            index_k_scale=index_k_scale,
            page_size=pool.page_size,
        )
```
**EN:** Implements the triton routine used by this attention module.
**CN:** 实现该注意力模块使用的 triton 例程。

### Lines 398-472: function _set_k_and_s_triton
```python
def _set_k_and_s_triton(
    buf: torch.Tensor,
    loc: torch.Tensor,
    index_k: torch.Tensor,
    index_k_scale: torch.Tensor,
    page_size: int,
):
    """
    :param buf: (num_pages, page_size 64 * (128B data + 4B scale)), uint8
    :param loc: (num_tokens_to_write,), int, element := the token index to write to
    :param index_k: (num_tokens_to_write, 128 elem), fp8
    :param index_k_scale: (num_tokens_to_write, 1 elem), fp32
    :return:
    """
    num_pages, buf_numel_per_page = buf.shape
    (num_tokens_to_write,) = loc.shape
    num_tokens_to_write_, index_head_dim = index_k.shape

    # Handle both 1D (num_tokens,) and 2D (num_tokens, 1) shapes for index_k_scale
    if index_k_scale.ndim == 1:
        num_tokens_to_write__ = index_k_scale.shape[0]
        scale_dim = 1
    elif index_k_scale.ndim == 2:
        num_tokens_to_write__, scale_dim = index_k_scale.shape
    else:
        raise ValueError(
            f"index_k_scale must be 1D or 2D, got shape {index_k_scale.shape}"
        )
    assert buf_numel_per_page == page_size * (128 + 4)
    assert num_tokens_to_write == num_tokens_to_write_ == num_tokens_to_write__
    assert index_head_dim == 128
    assert scale_dim == 1
# ... omitted 31 lines ...
    _set_k_and_s_triton_kernel[(num_tokens_to_write,)](
        buf_fp8,
        buf_fp32,
        loc,
        index_k,
        index_k_scale,
        index_k.stride(0),
        PAGE_SIZE=page_size,
        BUF_NUMEL_PER_PAGE=buf_numel_per_page,
        NUM_K_ELEMS_PER_TOKEN=index_head_dim,
        S_OFFSET_NBYTES_IN_PAGE=page_size * index_head_dim,
    )
```
**EN:** Implements the set k and s triton routine used by this attention module.
**CN:** 实现该注意力模块使用的 set k and s triton 例程。

### Lines 475-515: function _set_k_and_s_triton_kernel
```python
@triton.jit
def _set_k_and_s_triton_kernel(
    buf_fp8_ptr,
    buf_fp32_ptr,
    loc_ptr,
    index_k_ptr,
    index_k_scale_ptr,
    index_k_ptr_stride_0,
    PAGE_SIZE: tl.constexpr,
    BUF_NUMEL_PER_PAGE: tl.constexpr,
    NUM_K_ELEMS_PER_TOKEN: tl.constexpr,
    S_OFFSET_NBYTES_IN_PAGE: tl.constexpr,
):
    token_id = tl.program_id(0)

    loc = tl.load(loc_ptr + token_id)

    in_k_offsets = token_id * index_k_ptr_stride_0 + tl.arange(0, NUM_K_ELEMS_PER_TOKEN)

    # no need for `mask`, since we read 128B for k and 4B for scale, both pow of 2
    k = tl.load(index_k_ptr + in_k_offsets)
    k_scale = tl.load(index_k_scale_ptr + token_id)

    loc_page_index = loc // PAGE_SIZE
    loc_token_offset_in_page = loc % PAGE_SIZE

    out_k_offsets = (
        loc_page_index * BUF_NUMEL_PER_PAGE
        + loc_token_offset_in_page * NUM_K_ELEMS_PER_TOKEN
        + tl.arange(0, NUM_K_ELEMS_PER_TOKEN)
    )

    # "//4" b/c it is fp32 instead of uint8
    out_s_offset = (
        loc_page_index * BUF_NUMEL_PER_PAGE // 4
        + S_OFFSET_NBYTES_IN_PAGE // 4
        + loc_token_offset_in_page
    )

    tl.store(buf_fp8_ptr + out_k_offsets, k)
    tl.store(buf_fp32_ptr + out_s_offset, k_scale)
```
**EN:** Implements the set k and s triton kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 set k and s triton kernel 例程。

### Lines 518-553: function _get_k_triton
```python
def _get_k_triton(
    buf: torch.Tensor,
    page_indices: torch.Tensor,
    seq_len: int,
    page_size: int,
    index_head_dim: int,
):
    """
    Gather K (key) data from paged buffer using Triton.

    :param buf: (num_pages, page_size * 128 + page_size * 4), uint8
    :param page_indices: (num_pages,), int32/int64
    :param seq_len: int, number of tokens to gather
    :param page_size: int, typically 64
    :param index_head_dim: int, typically 128
    :return: (seq_len, index_head_dim), uint8
    """
    num_pages, buf_numel_per_page = buf.shape

    # Allocate output
    out = torch.empty((seq_len, index_head_dim), dtype=torch.uint8, device=buf.device)

    # Launch kernel with one thread per token
    grid = (seq_len,)
    _get_k_triton_kernel[grid](
        buf,
        page_indices,
        out,
        seq_len,
        page_size,
        buf_numel_per_page,
        index_head_dim,
        BLOCK_SIZE=128,
    )

    return out
```
**EN:** Implements the get k triton routine used by this attention module.
**CN:** 实现该注意力模块使用的 get k triton 例程。

### Lines 556-593: function _get_k_triton_kernel
```python
@triton.jit
def _get_k_triton_kernel(
    buf_ptr,
    page_indices_ptr,
    out_ptr,
    seq_len: tl.constexpr,
    page_size: tl.constexpr,
    buf_numel_per_page: tl.constexpr,
    index_head_dim: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    """
    Each program handles one token (seq_len tokens total).
    Loads 128 bytes from the appropriate page.
    """
    token_id = tl.program_id(0)

    # Calculate which page and offset within page
    page_idx = token_id // page_size
    token_offset_in_page = token_id % page_size

    # Load the page index from page_indices
    page_index = tl.load(page_indices_ptr + page_idx)

    # Calculate source offset in buf
    # buf[page_index, token_offset_in_page * index_head_dim : ...]
    src_base_offset = (
        page_index * buf_numel_per_page + token_offset_in_page * index_head_dim
    )

    # Load 128 bytes (index_head_dim elements)
    offsets = tl.arange(0, BLOCK_SIZE)
    mask = offsets < index_head_dim
    data = tl.load(buf_ptr + src_base_offset + offsets, mask=mask)

    # Store to output
    dst_offset = token_id * index_head_dim
    tl.store(out_ptr + dst_offset + offsets, data, mask=mask)
```
**EN:** Implements the get k triton kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 get k triton kernel 例程。

### Lines 596-631: function _get_s_triton
```python
def _get_s_triton(
    buf: torch.Tensor,
    page_indices: torch.Tensor,
    seq_len: int,
    page_size: int,
    index_head_dim: int,
):
    """
    Gather S (scale) data from paged buffer using Triton.

    :param buf: (num_pages, page_size * 128 + page_size * 4), uint8
    :param page_indices: (num_pages,), int32/int64
    :param seq_len: int, number of tokens to gather
    :param page_size: int, typically 64
    :param index_head_dim: int, typically 128
    :return: (seq_len, 4), uint8 (representing fp32 scale)
    """
    num_pages, buf_numel_per_page = buf.shape
    s_offset_in_page = page_size * index_head_dim  # Scales start after K data

    # Allocate output
    out = torch.empty((seq_len, 4), dtype=torch.uint8, device=buf.device)

    # Launch kernel with one thread per token
    grid = (seq_len,)
    _get_s_triton_kernel[grid](
        buf,
        page_indices,
        out,
        seq_len,
        page_size,
        buf_numel_per_page,
        s_offset_in_page,
    )

    return out
```
**EN:** Implements the get s triton routine used by this attention module.
**CN:** 实现该注意力模块使用的 get s triton 例程。

### Lines 634-670: function _get_s_triton_kernel
```python
@triton.jit
def _get_s_triton_kernel(
    buf_ptr,
    page_indices_ptr,
    out_ptr,
    seq_len: tl.constexpr,
    page_size: tl.constexpr,
    buf_numel_per_page: tl.constexpr,
    s_offset_in_page: tl.constexpr,
):
    """
    Each program handles one token (seq_len tokens total).
    Loads 4 bytes (fp32 scale) from the appropriate page.
    """
    token_id = tl.program_id(0)

    # Calculate which page and offset within page
    page_idx = token_id // page_size
    token_offset_in_page = token_id % page_size

    # Load the page index from page_indices
    page_index = tl.load(page_indices_ptr + page_idx)

    # Calculate source offset in buf
    # Scales are stored after K data: page_size * index_head_dim offset
    # buf[page_index, s_offset_in_page + token_offset_in_page * 4 : ...]
    src_base_offset = (
        page_index * buf_numel_per_page + s_offset_in_page + token_offset_in_page * 4
    )

    # Load 4 bytes (fp32 scale)
    offsets = tl.arange(0, 4)
    data = tl.load(buf_ptr + src_base_offset + offsets)

    # Store to output
    dst_offset = token_id * 4
    tl.store(out_ptr + dst_offset + offsets, data)
```
**EN:** Implements the get s triton kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 get s triton kernel 例程。

### Lines 673-736: function _get_k_and_s_triton
```python
def _get_k_and_s_triton(
    buf: torch.Tensor,
    page_indices: torch.Tensor,
    seq_lens: torch.Tensor,
    seq_len_sum: int,
    max_seq_len: int,
    page_size: int,
    index_head_dim: int,
):
    """
    Fused gather of both K (key) and S (scale) data from paged buffer using Triton.
    This is more efficient than calling GetK and GetS separately.

    :param buf: (num_pages, page_size * 128 + page_size * 4), uint8
    :param page_indices: (num_pages,), int32/int64
    :param seq_lens: tensor of sequence lens, int64
    :param seq_len_sum: sum of all sequence len, int32
    :param max_seq_len: max of sequence len, int32
    :param page_size: int, typically 64
    :param index_head_dim: int, typically 128
    :return: tuple of (k_out, s_out) where
             k_out: (seq_len, index_head_dim), uint8
             s_out: (seq_len, 4), uint8
    """
    # Allocate outputs
    k_out = torch.empty(
        (seq_len_sum, index_head_dim), dtype=torch.uint8, device=buf.device
    )
    s_out = torch.empty((seq_len_sum, 4), dtype=torch.uint8, device=buf.device)

    _, buf_numel_per_page = buf.shape
    _, page_indice_batch_offset = page_indices.shape
# ... omitted 20 lines ...
        seq_len_ptr=seq_lens,
        seq_len_num_pow=seq_num_pow2,
        page_size=page_size,
        buf_numel_per_page=buf_numel_per_page,
        index_head_dim=index_head_dim,
        s_offset_in_page=s_offset_in_page,
        page_indice_batch_offset=page_indice_batch_offset,
        BLOCK_SIZE=BLOCK_SIZE,
        BLOCK_SIZE_K=BLOCK_SIZE_K,
    )

    return k_out, s_out
```
**EN:** Implements the get k and s triton routine used by this attention module.
**CN:** 实现该注意力模块使用的 get k and s triton 例程。

### Lines 739-818: function _get_k_and_s_triton_kernel
```python
@triton.jit
def _get_k_and_s_triton_kernel(
    buf_ptr,
    page_indices_ptr,
    k_out_ptr,
    s_out_ptr,
    seq_len_ptr,
    seq_len_num_pow: tl.constexpr,
    page_size: tl.constexpr,
    buf_numel_per_page: tl.constexpr,
    index_head_dim: tl.constexpr,
    s_offset_in_page: tl.constexpr,
    page_indice_batch_offset,
    BLOCK_SIZE: tl.constexpr,
    BLOCK_SIZE_K: tl.constexpr,
):
    """
    Fused kernel that gathers both K and S data in a single pass.
    Each program handles one token (seq_len tokens total).
    Loads 128 bytes (K) + 4 bytes (S) from the appropriate page.
    """
    batch_id = tl.program_id(0)
    block_token_start = tl.program_id(1) * BLOCK_SIZE
    thread_idx = tl.program_id(2)

    # Define the token range within the block and the K dimension range handled by the thread.
    token_ids_in_block = tl.arange(0, BLOCK_SIZE)
    token_ids = block_token_start + token_ids_in_block
    k_offsets = thread_idx * BLOCK_SIZE_K + tl.arange(0, BLOCK_SIZE_K)

    seq_len = tl.load(seq_len_ptr + batch_id)
    token_valid_mask = token_ids < seq_len
# ... omitted 36 lines ...
    s_src_base_offset = page_index * buf_numel_per_page + s_src_token_offset

    s_offsets = tl.arange(0, 4)
    s_load_addr = buf_ptr + s_src_base_offset[:, None] + s_offsets[None, :]
    s_mask = token_valid_mask[:, None] & (s_offsets[None, :] < 4)
    s_data = tl.load(s_load_addr, mask=s_mask, other=0)

    # Store S to output
    s_dst_token_offset = batch_token_offset + token_ids
    s_dst_base_offset = s_dst_token_offset * 4
    s_store_addr = s_out_ptr + s_dst_base_offset[:, None] + s_offsets[None, :]
    tl.store(s_store_addr, s_data, mask=s_mask)
```
**EN:** Implements the get k and s triton kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 get k and s triton kernel 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Paged attention layouts / **CN:** 分页注意力布局
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理

## Dependencies / 依赖关系
- `typing.TYPE_CHECKING`
- `torch`
- `triton`
- `triton.language`
- `sglang.srt.layers.attention.nsa.utils.aiter_can_use_preshuffle_paged_mqa`
- `sglang.srt.layers.quantization.fp8_kernel.is_fp8_fnuz`
- `sglang.srt.utils.get_bool_env_var`
- `sglang.srt.utils.is_hip`
- `aiter.ops.cache.cp_gather_indexer_k_quant_cache`
- `sglang.srt.mem_cache.memory_pool.NSATokenToKVPool`
- `sglang.srt.layers.quantization.fp8_kernel.fp8_dtype`
