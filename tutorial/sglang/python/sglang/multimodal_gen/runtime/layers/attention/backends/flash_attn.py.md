# flash_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/attention/backends/flash_attn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `FlashAttentionMetadata`, `FlashAttentionMetadataBuilder`, and `FlashAttentionBackend`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `FlashAttentionMetadata`、`FlashAttentionMetadataBuilder` 和 `FlashAttentionBackend` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 3-13: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass
from typing import Any, List, Optional, Tuple

import torch

from sglang.jit_kernel.flash_attention import flash_attn_varlen_func
from sglang.multimodal_gen.runtime.layers.utils import register_custom_op
from sglang.multimodal_gen.runtime.managers.forward_context import get_forward_context
from sglang.multimodal_gen.runtime.platforms import (
    AttentionBackendEnum,
)
```
**EN:** This block establishes the module context and imports `dataclasses`, `typing`, `torch`, `sglang.jit_kernel.flash_attention`, `sglang.multimodal_gen.runtime.layers.utils`, and `sglang.multimodal_gen.runtime.managers.forward_context`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`typing`、`torch`、`sglang.jit_kernel.flash_attention`、`sglang.multimodal_gen.runtime.layers.utils` 和 `sglang.multimodal_gen.runtime.managers.forward_context`。这些依赖为后续实现提供所需符号。

### Lines 16-17: `maybe_contiguous` implementation / `maybe_contiguous` 实现
```python
def maybe_contiguous(x: Optional[torch.Tensor]) -> Optional[torch.Tensor]:
    return x.contiguous() if x is not None and x.stride(-1) != 1 else x
```
**EN:** This block defines function `maybe_contiguous`. It handles maybe contiguous logic. Key calls include `x.contiguous`, and `x.stride`. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了函数 `maybe_contiguous`。 它用于处理 maybe contiguous 相关逻辑。 关键调用包括 `x.contiguous` 和 `x.stride`。 本段逻辑主要由 `x` 等参数驱动。

### Lines 27-84: `flash_attn_varlen_func_fake_out` implementation / `flash_attn_varlen_func_fake_out` 实现
```python
def flash_attn_varlen_func_fake_out(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    cu_seqlens_q: Optional[torch.Tensor] = None,
    cu_seqlens_k: Optional[torch.Tensor] = None,
    max_seqlen_q: Optional[int] = None,
    max_seqlen_k: Optional[int] = None,
    seqused_q: Optional[torch.Tensor] = None,
    seqused_k: Optional[torch.Tensor] = None,
    page_table: Optional[torch.Tensor] = None,
    softmax_scale: Optional[float] = None,
    causal: bool = False,
    qv: Optional[torch.Tensor] = None,
    q_descale: Optional[torch.Tensor] = None,
    k_descale: Optional[torch.Tensor] = None,
    v_descale: Optional[torch.Tensor] = None,
    window_size: Optional[List[int]] = None,
    attention_chunk: int = 0,
    softcap: float = 0.0,
    num_splits: int = 1,
    pack_gqa: Optional[bool] = None,
    sm_margin: int = 0,
    return_softmax_lse: bool = False,
    sinks: Optional[torch.Tensor] = None,
    ver: int = 4,
) -> torch.Tensor:
    assert ver == 4, "only support flash attention v4"
    q, k, v = [maybe_contiguous(t) for t in (q, k, v)]
    num_head, head_dim = q.shape[-2:]
    if cu_seqlens_q is None:
        batch_size, seqlen_q = q.shape[:2]
    else:
        batch_size = cu_seqlens_q.shape[0] - 1
        seqlen_q = None
    head_dim_v = v.shape[-1]

    if cu_seqlens_q is not None:
        assert cu_seqlens_q.shape == (
            batch_size + 1,
        ), "cu_seqlens_q must have shape (batch_size + 1,)"
        assert cu_seqlens_q.dtype == torch.int32, "cu_seqlens_q must be int32"
        assert cu_seqlens_q.stride(0) == 1, "cu_seqlens_q must be contiguous"

    assert q.dtype in [
        torch.float16,
        torch.bfloat16,
    ], "inputs must be float16 or bfloat16"
    assert q.dtype == k.dtype == v.dtype, "inputs must have the same dtype"
    assert head_dim <= 256, "head_dim must be less than or equal to 256"
    alignment = 16 // q.element_size()
    assert head_dim_v % alignment == 0, f"head_dim_v must be divisible by {alignment}"

    q_batch_seqlen_shape = (
        (batch_size, seqlen_q) if cu_seqlens_q is None else (q.shape[0],)
    )
    out = q.new_empty(*q_batch_seqlen_shape, num_head, head_dim_v)
    return out
```
**EN:** This block defines function `flash_attn_varlen_func_fake_out`. It handles flash attn varlen func fake out logic. Key calls include `q.new_empty`, `maybe_contiguous`, `q.element_size`, and `cu_seqlens_q.stride`. The implementation branches on conditions. Parameters such as `q`, `k`, `v`, `cu_seqlens_q`, and `cu_seqlens_k` drive the behavior in this section.
**CN:** 该代码块定义了函数 `flash_attn_varlen_func_fake_out`。 它用于处理 flash attn varlen func fake out 相关逻辑。 关键调用包括 `q.new_empty`、`maybe_contiguous`、`q.element_size` 和 `cu_seqlens_q.stride`。 实现中包含条件分支。 本段逻辑主要由 `q`、`k`、`v`、`cu_seqlens_q` 和 `cu_seqlens_k` 等参数驱动。

### Lines 87-153: `flash_attn_varlen_func_fake_out_lse` implementation / `flash_attn_varlen_func_fake_out_lse` 实现
```python
def flash_attn_varlen_func_fake_out_lse(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    cu_seqlens_q: Optional[torch.Tensor] = None,
    cu_seqlens_k: Optional[torch.Tensor] = None,
    max_seqlen_q: Optional[int] = None,
    max_seqlen_k: Optional[int] = None,
    seqused_q: Optional[torch.Tensor] = None,
    seqused_k: Optional[torch.Tensor] = None,
    page_table: Optional[torch.Tensor] = None,
    softmax_scale: Optional[float] = None,
    causal: bool = False,
    qv: Optional[torch.Tensor] = None,
    q_descale: Optional[torch.Tensor] = None,
    k_descale: Optional[torch.Tensor] = None,
    v_descale: Optional[torch.Tensor] = None,
    window_size: Optional[List[int]] = None,
    attention_chunk: int = 0,
    softcap: float = 0.0,
    num_splits: int = 1,
    pack_gqa: Optional[bool] = None,
    sm_margin: int = 0,
    return_softmax_lse: bool = True,
    sinks: Optional[torch.Tensor] = None,
    ver: int = 4,
) -> Tuple[torch.Tensor, torch.Tensor]:
    assert ver == 4, "only support flash attention v4"
    q, k, v = [maybe_contiguous(t) for t in (q, k, v)]
    num_head, head_dim = q.shape[-2:]
    if cu_seqlens_q is None:
        batch_size, seqlen_q = q.shape[:2]
        total_q = batch_size * seqlen_q
    else:
        batch_size = cu_seqlens_q.shape[0] - 1
        seqlen_q = None
        total_q = q.shape[0]
    head_dim_v = v.shape[-1]

    if cu_seqlens_q is not None:
        assert cu_seqlens_q.shape == (
            batch_size + 1,
        ), "cu_seqlens_q must have shape (batch_size + 1,)"
        assert cu_seqlens_q.dtype == torch.int32, "cu_seqlens_q must be int32"
        assert cu_seqlens_q.stride(0) == 1, "cu_seqlens_q must be contiguous"

    assert q.dtype in [
        torch.float16,
        torch.bfloat16,
    ], "inputs must be float16 or bfloat16"
    assert q.dtype == k.dtype == v.dtype, "inputs must have the same dtype"
    assert head_dim <= 256, "head_dim must be less than or equal to 256"
    alignment = 16 // q.element_size()
    assert head_dim_v % alignment == 0, f"head_dim_v must be divisible by {alignment}"

    q_batch_seqlen_shape = (
        (batch_size, seqlen_q) if cu_seqlens_q is None else (total_q,)
    )
    lse_shape = (
        (batch_size, num_head, seqlen_q)
        if cu_seqlens_q is None
        else (num_head, total_q)
    )

    out = q.new_empty(*q_batch_seqlen_shape, num_head, head_dim_v)
    lse = q.new_empty(lse_shape, dtype=torch.float32)
    return out, lse
```
**EN:** This block defines function `flash_attn_varlen_func_fake_out_lse`. It handles flash attn varlen func fake out lse logic. Key calls include `q.new_empty`, `maybe_contiguous`, `q.element_size`, and `cu_seqlens_q.stride`. The implementation branches on conditions. Parameters such as `q`, `k`, `v`, `cu_seqlens_q`, and `cu_seqlens_k` drive the behavior in this section.
**CN:** 该代码块定义了函数 `flash_attn_varlen_func_fake_out_lse`。 它用于处理 flash attn varlen func fake out lse 相关逻辑。 关键调用包括 `q.new_empty`、`maybe_contiguous`、`q.element_size` 和 `cu_seqlens_q.stride`。 实现中包含条件分支。 本段逻辑主要由 `q`、`k`、`v`、`cu_seqlens_q` 和 `cu_seqlens_k` 等参数驱动。

### Lines 161-222: `flash_attn_varlen_func_op` implementation / `flash_attn_varlen_func_op` 实现
```python
@register_custom_op(fake_impl=flash_attn_varlen_func_fake_out)
def flash_attn_varlen_func_op(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    cu_seqlens_q: Optional[torch.Tensor] = None,
    cu_seqlens_k: Optional[torch.Tensor] = None,
    max_seqlen_q: Optional[int] = None,
    max_seqlen_k: Optional[int] = None,
    seqused_q: Optional[torch.Tensor] = None,
    seqused_k: Optional[torch.Tensor] = None,
    page_table: Optional[torch.Tensor] = None,
    softmax_scale: Optional[float] = None,
    causal: bool = False,
    qv: Optional[torch.Tensor] = None,
    q_descale: Optional[torch.Tensor] = None,
    k_descale: Optional[torch.Tensor] = None,
    v_descale: Optional[torch.Tensor] = None,
    window_size: Optional[List[int]] = None,
    attention_chunk: int = 0,
    softcap: float = 0.0,
    num_splits: int = 1,
    pack_gqa: Optional[bool] = None,
    sm_margin: int = 0,
    return_softmax_lse: bool = False,
    sinks: Optional[torch.Tensor] = None,
    ver: int = 4,
) -> torch.Tensor:
    if window_size is None:
        window_size = [-1, -1]
    if return_softmax_lse:
        raise ValueError(
            "flash_attn_varlen_func_op is out-only op; return_softmax_lse must be False. "
            "Use flash_attn_varlen_func_op_lse for (out, lse)."
        )
    return flash_attn_varlen_func(
        q,
        k,
        v,
        cu_seqlens_q=cu_seqlens_q,
        cu_seqlens_k=cu_seqlens_k,
        max_seqlen_q=max_seqlen_q,
        max_seqlen_k=max_seqlen_k,
        seqused_q=seqused_q,
        seqused_k=seqused_k,
        page_table=page_table,
        softmax_scale=softmax_scale,
        causal=causal,
        qv=qv,
        q_descale=q_descale,
        k_descale=k_descale,
        v_descale=v_descale,
        window_size=tuple(window_size),
        attention_chunk=attention_chunk,
        softcap=softcap,
        num_splits=num_splits,
        pack_gqa=pack_gqa,
        sm_margin=sm_margin,
        return_softmax_lse=False,
        sinks=sinks,
        ver=ver,
    )
```
**EN:** This block defines function `flash_attn_varlen_func_op`. It handles flash attn varlen func op logic. Key calls include `register_custom_op`, `flash_attn_varlen_func`, `ValueError`, and `tuple`. The implementation branches on conditions. Parameters such as `q`, `k`, `v`, `cu_seqlens_q`, and `cu_seqlens_k` drive the behavior in this section.
**CN:** 该代码块定义了函数 `flash_attn_varlen_func_op`。 它用于处理 flash attn varlen func op 相关逻辑。 关键调用包括 `register_custom_op`、`flash_attn_varlen_func`、`ValueError` 和 `tuple`。 实现中包含条件分支。 本段逻辑主要由 `q`、`k`、`v`、`cu_seqlens_q` 和 `cu_seqlens_k` 等参数驱动。

### Lines 225-286: `flash_attn_varlen_func_op_lse` implementation / `flash_attn_varlen_func_op_lse` 实现
```python
@register_custom_op(fake_impl=flash_attn_varlen_func_fake_out_lse)
def flash_attn_varlen_func_op_lse(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    cu_seqlens_q: Optional[torch.Tensor] = None,
    cu_seqlens_k: Optional[torch.Tensor] = None,
    max_seqlen_q: Optional[int] = None,
    max_seqlen_k: Optional[int] = None,
    seqused_q: Optional[torch.Tensor] = None,
    seqused_k: Optional[torch.Tensor] = None,
    page_table: Optional[torch.Tensor] = None,
    softmax_scale: Optional[float] = None,
    causal: bool = False,
    qv: Optional[torch.Tensor] = None,
    q_descale: Optional[torch.Tensor] = None,
    k_descale: Optional[torch.Tensor] = None,
    v_descale: Optional[torch.Tensor] = None,
    window_size: Optional[List[int]] = None,
    attention_chunk: int = 0,
    softcap: float = 0.0,
    num_splits: int = 1,
    pack_gqa: Optional[bool] = None,
    sm_margin: int = 0,
    return_softmax_lse: bool = True,
    sinks: Optional[torch.Tensor] = None,
    ver: int = 4,
) -> Tuple[torch.Tensor, torch.Tensor]:
    if window_size is None:
        window_size = [-1, -1]
    if not return_softmax_lse:
        raise ValueError(
            "flash_attn_varlen_func_op_lse is out+lse op; return_softmax_lse must be True. "
            "Use flash_attn_varlen_func_op for out-only."
        )
    return flash_attn_varlen_func(
        q,
        k,
        v,
        cu_seqlens_q=cu_seqlens_q,
        cu_seqlens_k=cu_seqlens_k,
        max_seqlen_q=max_seqlen_q,
        max_seqlen_k=max_seqlen_k,
        seqused_q=seqused_q,
        seqused_k=seqused_k,
        page_table=page_table,
        softmax_scale=softmax_scale,
        causal=causal,
        qv=qv,
        q_descale=q_descale,
        k_descale=k_descale,
        v_descale=v_descale,
        window_size=tuple(window_size),
        attention_chunk=attention_chunk,
        softcap=softcap,
        num_splits=num_splits,
        pack_gqa=pack_gqa,
        sm_margin=sm_margin,
        return_softmax_lse=True,
        sinks=sinks,
        ver=ver,
    )
```
**EN:** This block defines function `flash_attn_varlen_func_op_lse`. It handles flash attn varlen func op lse logic. Key calls include `register_custom_op`, `flash_attn_varlen_func`, `ValueError`, and `tuple`. The implementation branches on conditions. Parameters such as `q`, `k`, `v`, `cu_seqlens_q`, and `cu_seqlens_k` drive the behavior in this section.
**CN:** 该代码块定义了函数 `flash_attn_varlen_func_op_lse`。 它用于处理 flash attn varlen func op lse 相关逻辑。 关键调用包括 `register_custom_op`、`flash_attn_varlen_func`、`ValueError` 和 `tuple`。 实现中包含条件分支。 本段逻辑主要由 `q`、`k`、`v`、`cu_seqlens_q` 和 `cu_seqlens_k` 等参数驱动。

### Lines 289-296: supporting statements / 辅助语句
```python
from sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend import (
    AttentionBackend,
    AttentionImpl,
    AttentionMetadata,
    AttentionMetadataBuilder,
)

fa_ver = 3
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `fa_ver`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `fa_ver` 等名称。

### Lines 299-301: `set_fa_ver` implementation / `set_fa_ver` 实现
```python
def set_fa_ver(ver: int) -> None:
    global fa_ver
    fa_ver = ver
```
**EN:** This block defines function `set_fa_ver`. It configures fa ver. Parameters such as `ver` drive the behavior in this section.
**CN:** 该代码块定义了函数 `set_fa_ver`。 它用于设置fa ver。 本段逻辑主要由 `ver` 等参数驱动。

### Lines 305-307: `FlashAttentionMetadata` class overview / `FlashAttentionMetadata` 类概览
```python
class FlashAttentionMetadata:
    # Sequence lengths for the forward batch
    # Maximum sequence length for query
```
**EN:** This block defines class `FlashAttentionMetadata`. It encapsulates flash attention metadata behavior.
**CN:** 该代码块定义了类 `FlashAttentionMetadata`。 它用于封装 flash attention metadata 相关行为。

### Lines 308-314: supporting statements / 辅助语句
```python
    max_seqlen_q: int = 1
    # Maximum sequence length for key
    max_seqlen_k: int = 0
    # Cumulative sequence lengths for query
    cu_seqlens_q: torch.Tensor = None
    # Cumulative sequence lengths for key
    cu_seqlens_k: torch.Tensor = None
```
**EN:** This block gathers supporting statements inside `FlashAttentionMetadata`. It updates names such as `max_seqlen_q`, `max_seqlen_k`, `cu_seqlens_q`, and `cu_seqlens_k`.
**CN:** 该代码块汇集了位于 `FlashAttentionMetadata` 内部的辅助语句。 它会更新 `max_seqlen_q`、`max_seqlen_k`、`cu_seqlens_q` 和 `cu_seqlens_k` 等名称。

### Lines 317-317: `FlashAttentionMetadataBuilder` class overview / `FlashAttentionMetadataBuilder` 类概览
```python
class FlashAttentionMetadataBuilder(AttentionMetadataBuilder):
```
**EN:** This block defines class `FlashAttentionMetadataBuilder`. It encapsulates flash attention metadata builder behavior. It inherits from `AttentionMetadataBuilder`.
**CN:** 该代码块定义了类 `FlashAttentionMetadataBuilder`。 它用于封装 flash attention metadata builder 相关行为。 它继承自 `AttentionMetadataBuilder`。

### Lines 318-319: `__init__` implementation / `__init__` 实现
```python
    def __init__(self) -> None:
        pass
```
**EN:** This block defines method `__init__` on `FlashAttentionMetadataBuilder`. It initializes the instance state.
**CN:** 该代码块定义了 `FlashAttentionMetadataBuilder` 的方法 `__init__`。 它用于初始化实例状态。

### Lines 321-322: `prepare` implementation / `prepare` 实现
```python
    def prepare(self) -> None:
        pass
```
**EN:** This block defines method `prepare` on `FlashAttentionMetadataBuilder`. It prepares function.
**CN:** 该代码块定义了 `FlashAttentionMetadataBuilder` 的方法 `prepare`。 它用于准备函数。

### Lines 324-330: `build` implementation / `build` 实现
```python
    def build(  # type: ignore
        self,
        raw_latent_shape=list,
        **kwargs: dict[str, Any],
    ) -> FlashAttentionMetadata:
        # TODO: put empty values here to be set at first-run, since the q_len calculation can be complicated
        return FlashAttentionMetadata(max_seqlen_q=None, max_seqlen_k=None)
```
**EN:** This block defines method `build` on `FlashAttentionMetadataBuilder`. It builds function. Key calls include `FlashAttentionMetadata`. Parameters such as `raw_latent_shape` drive the behavior in this section.
**CN:** 该代码块定义了 `FlashAttentionMetadataBuilder` 的方法 `build`。 它用于构建函数。 关键调用包括 `FlashAttentionMetadata`。 本段逻辑主要由 `raw_latent_shape` 等参数驱动。

### Lines 333-333: `FlashAttentionBackend` class overview / `FlashAttentionBackend` 类概览
```python
class FlashAttentionBackend(AttentionBackend):
```
**EN:** This block defines class `FlashAttentionBackend`. It encapsulates flash attention backend behavior. It inherits from `AttentionBackend`.
**CN:** 该代码块定义了类 `FlashAttentionBackend`。 它用于封装 flash attention backend 相关行为。 它继承自 `AttentionBackend`。

### Lines 334-334: supporting statements / 辅助语句
```python
    accept_output_buffer: bool = True
```
**EN:** This block gathers supporting statements inside `FlashAttentionBackend`. It updates names such as `accept_output_buffer`.
**CN:** 该代码块汇集了位于 `FlashAttentionBackend` 内部的辅助语句。 它会更新 `accept_output_buffer` 等名称。

### Lines 336-338: `get_supported_head_sizes` implementation / `get_supported_head_sizes` 实现
```python
    @staticmethod
    def get_supported_head_sizes() -> list[int]:
        return [32, 64, 96, 128, 160, 192, 224, 256]
```
**EN:** This block defines method `get_supported_head_sizes` on `FlashAttentionBackend`. It retrieves supported head sizes.
**CN:** 该代码块定义了 `FlashAttentionBackend` 的方法 `get_supported_head_sizes`。 它用于获取supported head sizes。

### Lines 340-342: `get_enum` implementation / `get_enum` 实现
```python
    @staticmethod
    def get_enum() -> AttentionBackendEnum:
        return AttentionBackendEnum.FA
```
**EN:** This block defines method `get_enum` on `FlashAttentionBackend`. It retrieves enum.
**CN:** 该代码块定义了 `FlashAttentionBackend` 的方法 `get_enum`。 它用于获取enum。

### Lines 344-346: `get_impl_cls` implementation / `get_impl_cls` 实现
```python
    @staticmethod
    def get_impl_cls() -> type["FlashAttentionImpl"]:
        return FlashAttentionImpl
```
**EN:** This block defines method `get_impl_cls` on `FlashAttentionBackend`. It retrieves impl cls.
**CN:** 该代码块定义了 `FlashAttentionBackend` 的方法 `get_impl_cls`。 它用于获取impl cls。

### Lines 348-350: `get_metadata_cls` implementation / `get_metadata_cls` 实现
```python
    @staticmethod
    def get_metadata_cls() -> type["AttentionMetadata"]:
        raise NotImplementedError
```
**EN:** This block defines method `get_metadata_cls` on `FlashAttentionBackend`. It retrieves metadata cls.
**CN:** 该代码块定义了 `FlashAttentionBackend` 的方法 `get_metadata_cls`。 它用于获取metadata cls。

### Lines 352-354: `get_builder_cls` implementation / `get_builder_cls` 实现
```python
    @staticmethod
    def get_builder_cls() -> type["AttentionMetadataBuilder"]:
        return FlashAttentionMetadataBuilder
```
**EN:** This block defines method `get_builder_cls` on `FlashAttentionBackend`. It retrieves builder cls.
**CN:** 该代码块定义了 `FlashAttentionBackend` 的方法 `get_builder_cls`。 它用于获取builder cls。

### Lines 357-357: `FlashAttentionImpl` class overview / `FlashAttentionImpl` 类概览
```python
class FlashAttentionImpl(AttentionImpl):
```
**EN:** This block defines class `FlashAttentionImpl`. It encapsulates flash attention impl behavior. It inherits from `AttentionImpl`.
**CN:** 该代码块定义了类 `FlashAttentionImpl`。 它用于封装 flash attention impl 相关行为。 它继承自 `AttentionImpl`。

### Lines 358-373: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        causal: bool,
        softmax_scale: float,
        num_kv_heads: int | None = None,
        prefix: str = "",
        **extra_impl_args,
    ) -> None:
        self.num_heads = num_heads
        self.num_kv_heads = num_kv_heads
        self.head_size = head_size
        self.causal = causal
        self.softmax_scale = softmax_scale
        self.attention_metadata = FlashAttentionMetadata()
```
**EN:** This block defines method `__init__` on `FlashAttentionImpl`. It initializes the instance state. Key calls include `FlashAttentionMetadata`. Parameters such as `num_heads`, `head_size`, `causal`, `softmax_scale`, and `num_kv_heads` drive the behavior in this section.
**CN:** 该代码块定义了 `FlashAttentionImpl` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `FlashAttentionMetadata`。 本段逻辑主要由 `num_heads`、`head_size`、`causal`、`softmax_scale` 和 `num_kv_heads` 等参数驱动。

### Lines 375-445: `forward` implementation / `forward` 实现
```python
    def forward(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        attn_metadata: AttentionMetadata = None,
        *,
        return_softmax_lse: bool = False,
    ):
        attn_metadata: FlashAttentionMetadata = get_forward_context().attn_metadata
        if attn_metadata is not None and attn_metadata.max_seqlen_q is None:
            attn_metadata.max_seqlen_q = query.shape[1]
            attn_metadata.max_seqlen_k = key.shape[1]
            max_seqlen_q = attn_metadata.max_seqlen_q
            max_seqlen_k = attn_metadata.max_seqlen_k
        else:
            max_seqlen_q = query.shape[1]
            max_seqlen_k = key.shape[1]

        # FA version selection:
        # - fa_ver == 3: call python function (can return Tensor or (Tensor, Tensor) depending on flag)
        # - fa_ver == 4: call custom ops with FIXED return schema
        if fa_ver == 3:
            flash_attn_op = flash_attn_varlen_func
            output = flash_attn_op(
                q=query,
                k=key,
                v=value,
                cu_seqlens_q=None,
                cu_seqlens_k=None,
                max_seqlen_q=max_seqlen_q,
                max_seqlen_k=max_seqlen_k,
                softmax_scale=self.softmax_scale,
                causal=self.causal,
                return_softmax_lse=return_softmax_lse,
                ver=fa_ver,
            )
            return output

        if fa_ver == 4:
            if return_softmax_lse:
                out_tensor, softmax_lse = flash_attn_varlen_func_op_lse(
                    q=query,
                    k=key,
                    v=value,
                    cu_seqlens_q=None,
                    cu_seqlens_k=None,
                    max_seqlen_q=max_seqlen_q,
                    max_seqlen_k=max_seqlen_k,
                    softmax_scale=self.softmax_scale,
                    causal=self.causal,
                    return_softmax_lse=True,
                    ver=fa_ver,
                )
                return out_tensor, softmax_lse
            out_tensor = flash_attn_varlen_func_op(
                q=query,
                k=key,
                v=value,
                cu_seqlens_q=None,
                cu_seqlens_k=None,
                max_seqlen_q=max_seqlen_q,
                max_seqlen_k=max_seqlen_k,
                softmax_scale=self.softmax_scale,
                causal=self.causal,
                return_softmax_lse=False,
                ver=fa_ver,
            )
            return out_tensor

        raise ValueError(f"flash attention version {fa_ver} is not supported.")
```
**EN:** This block defines method `forward` on `FlashAttentionImpl`. It executes function. Key calls include `ValueError`, `get_forward_context`, `flash_attn_op`, `flash_attn_varlen_func_op`, and `flash_attn_varlen_func_op_lse`. The implementation branches on conditions. Parameters such as `query`, `key`, `value`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `FlashAttentionImpl` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `ValueError`、`get_forward_context`、`flash_attn_op`、`flash_attn_varlen_func_op` 和 `flash_attn_varlen_func_op_lse`。 实现中包含条件分支。 本段逻辑主要由 `query`、`key`、`value` 和 `attn_metadata` 等参数驱动。

## Key Concepts / 关键概念
- `maybe_contiguous`: Top-level function that handles maybe contiguous logic. / 顶层函数，用于处理 maybe contiguous 相关逻辑。
- `flash_attn_varlen_func_fake_out`: Top-level function that handles flash attn varlen func fake out logic. / 顶层函数，用于处理 flash attn varlen func fake out 相关逻辑。
- `flash_attn_varlen_func_fake_out_lse`: Top-level function that handles flash attn varlen func fake out lse logic. / 顶层函数，用于处理 flash attn varlen func fake out lse 相关逻辑。
- `flash_attn_varlen_func_op`: Top-level function that handles flash attn varlen func op logic. / 顶层函数，用于处理 flash attn varlen func op 相关逻辑。
- `flash_attn_varlen_func_op_lse`: Top-level function that handles flash attn varlen func op lse logic. / 顶层函数，用于处理 flash attn varlen func op lse 相关逻辑。
- `set_fa_ver`: Top-level function that configures fa ver. / 顶层函数，用于设置fa ver。
- `FlashAttentionMetadata`: Primary class that encapsulates flash attention metadata behavior. / 核心类，用于封装 flash attention metadata 相关行为。
- `FlashAttentionMetadataBuilder`: Primary class that encapsulates flash attention metadata builder behavior. / 核心类，用于封装 flash attention metadata builder 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.jit_kernel.flash_attention`, `sglang.multimodal_gen.runtime.layers.utils`, `sglang.multimodal_gen.runtime.managers.forward_context`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`

- **Total lines / 总行数**: 445
