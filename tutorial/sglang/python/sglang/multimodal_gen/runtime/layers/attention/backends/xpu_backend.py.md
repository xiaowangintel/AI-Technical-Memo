# xpu_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/attention/backends/xpu_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `XPUAttentionBackend`, `XPUAttentionImpl`, and `_get_cu_seqlens`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `XPUAttentionBackend`、`XPUAttentionImpl` 和 `_get_cu_seqlens` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-8: module setup and imports / 模块初始化与导入
```python
from functools import lru_cache

import torch

from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
```
**EN:** This block establishes the module context and imports `functools`, `torch`, and `sglang.multimodal_gen.runtime.platforms`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `functools`、`torch` 和 `sglang.multimodal_gen.runtime.platforms`。这些依赖为后续实现提供所需符号。

### Lines 10-25: supporting statements / 辅助语句
```python
try:
    from sgl_kernel.flash_attn import flash_attn_varlen_func

    flash_attn_func = flash_attn_varlen_func
except ImportError as e:
    raise e

from sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend import (
    AttentionBackend,
    AttentionImpl,
    AttentionMetadata,
    AttentionMetadataBuilder,
)
from sglang.multimodal_gen.runtime.layers.attention.backends.flash_attn import (
    FlashAttentionMetadataBuilder,
)
```
**EN:** This block gathers supporting statements at module scope.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。

### Lines 28-28: `XPUAttentionBackend` class overview / `XPUAttentionBackend` 类概览
```python
class XPUAttentionBackend(AttentionBackend):
```
**EN:** This block defines class `XPUAttentionBackend`. It encapsulates xpuattention backend behavior. It inherits from `AttentionBackend`.
**CN:** 该代码块定义了类 `XPUAttentionBackend`。 它用于封装 xpuattention backend 相关行为。 它继承自 `AttentionBackend`。

### Lines 29-29: supporting statements / 辅助语句
```python
    accept_output_buffer: bool = True
```
**EN:** This block gathers supporting statements inside `XPUAttentionBackend`. It updates names such as `accept_output_buffer`.
**CN:** 该代码块汇集了位于 `XPUAttentionBackend` 内部的辅助语句。 它会更新 `accept_output_buffer` 等名称。

### Lines 31-33: `get_supported_head_sizes` implementation / `get_supported_head_sizes` 实现
```python
    @staticmethod
    def get_supported_head_sizes() -> list[int]:
        return [64, 96, 128, 192, 256]
```
**EN:** This block defines method `get_supported_head_sizes` on `XPUAttentionBackend`. It retrieves supported head sizes.
**CN:** 该代码块定义了 `XPUAttentionBackend` 的方法 `get_supported_head_sizes`。 它用于获取supported head sizes。

### Lines 35-37: `get_enum` implementation / `get_enum` 实现
```python
    @staticmethod
    def get_enum() -> AttentionBackendEnum:
        return AttentionBackendEnum.FA
```
**EN:** This block defines method `get_enum` on `XPUAttentionBackend`. It retrieves enum.
**CN:** 该代码块定义了 `XPUAttentionBackend` 的方法 `get_enum`。 它用于获取enum。

### Lines 39-41: `get_impl_cls` implementation / `get_impl_cls` 实现
```python
    @staticmethod
    def get_impl_cls() -> type["XPUAttentionImpl"]:
        return XPUAttentionImpl
```
**EN:** This block defines method `get_impl_cls` on `XPUAttentionBackend`. It retrieves impl cls.
**CN:** 该代码块定义了 `XPUAttentionBackend` 的方法 `get_impl_cls`。 它用于获取impl cls。

### Lines 43-46: `get_metadata_cls` implementation / `get_metadata_cls` 实现
```python
    @staticmethod
    def get_metadata_cls() -> type["AttentionMetadata"]:
        """XPU backend does not require special metadata."""
        return AttentionMetadata
```
**EN:** This block defines method `get_metadata_cls` on `XPUAttentionBackend`. XPU backend does not require special metadata.
**CN:** 该代码块定义了 `XPUAttentionBackend` 的方法 `get_metadata_cls`。 它用于获取metadata cls。

### Lines 48-50: `get_builder_cls` implementation / `get_builder_cls` 实现
```python
    @staticmethod
    def get_builder_cls() -> type["AttentionMetadataBuilder"]:
        return FlashAttentionMetadataBuilder
```
**EN:** This block defines method `get_builder_cls` on `XPUAttentionBackend`. It retrieves builder cls.
**CN:** 该代码块定义了 `XPUAttentionBackend` 的方法 `get_builder_cls`。 它用于获取builder cls。

### Lines 53-61: `_get_cu_seqlens` implementation / `_get_cu_seqlens` 实现
```python
@lru_cache(maxsize=128)
def _get_cu_seqlens(device_index: int, bsz: int, seqlen: int) -> torch.Tensor:
    return torch.arange(
        0,
        (bsz + 1) * seqlen,
        step=seqlen,
        device=torch.device("xpu", device_index),
        dtype=torch.int32,
    )
```
**EN:** This block defines function `_get_cu_seqlens`. It retrieves cu seqlens. Key calls include `lru_cache`, `torch.arange`, and `torch.device`. Parameters such as `device_index`, `bsz`, and `seqlen` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_get_cu_seqlens`。 它用于获取cu seqlens。 关键调用包括 `lru_cache`、`torch.arange` 和 `torch.device`。 本段逻辑主要由 `device_index`、`bsz` 和 `seqlen` 等参数驱动。

### Lines 64-65: `XPUAttentionImpl` class overview / `XPUAttentionImpl` 类概览
```python
class XPUAttentionImpl(AttentionImpl):
```
**EN:** This block defines class `XPUAttentionImpl`. It encapsulates xpuattention impl behavior. It inherits from `AttentionImpl`.
**CN:** 该代码块定义了类 `XPUAttentionImpl`。 它用于封装 xpuattention impl 相关行为。 它继承自 `AttentionImpl`。

### Lines 66-80: `__init__` implementation / `__init__` 实现
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
```
**EN:** This block defines method `__init__` on `XPUAttentionImpl`. It initializes the instance state. Parameters such as `num_heads`, `head_size`, `causal`, `softmax_scale`, and `num_kv_heads` drive the behavior in this section.
**CN:** 该代码块定义了 `XPUAttentionImpl` 的方法 `__init__`。 它用于初始化实例状态。 本段逻辑主要由 `num_heads`、`head_size`、`causal`、`softmax_scale` 和 `num_kv_heads` 等参数驱动。

### Lines 82-122: `forward` implementation / `forward` 实现
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
        bsz, seqlen_q, nheads_q, d = tuple(query.shape)
        _, seqlen_k, nheads_k, _ = tuple(key.shape)

        max_seqlen_q = seqlen_q
        max_seqlen_k = seqlen_k

        q_ = query.contiguous().reshape(bsz * seqlen_q, nheads_q, d)
        k_ = key.contiguous().reshape(bsz * seqlen_k, nheads_k, d)
        v_ = value.contiguous().reshape(bsz * seqlen_k, nheads_k, d)
        cu_q = _get_cu_seqlens(q_.device.index, bsz, seqlen_q)
        cu_k = _get_cu_seqlens(q_.device.index, bsz, seqlen_k)

        out = flash_attn_func(
            q=q_,
            k=k_,
            v=v_,
            cu_seqlens_q=cu_q,
            cu_seqlens_k=cu_k,
            max_seqlen_q=max_seqlen_q,
            max_seqlen_k=max_seqlen_k,
            softmax_scale=self.softmax_scale,
            causal=self.causal,
            return_softmax_lse=return_softmax_lse,
        )

        if return_softmax_lse:
            out_tensor, softmax_lse = out[:2]
            result = out_tensor.reshape(bsz, seqlen_q, nheads_q, d)
            return result, softmax_lse

        result = out.reshape(bsz, seqlen_q, nheads_q, d)
        return result
```
**EN:** This block defines method `forward` on `XPUAttentionImpl`. It executes function. Key calls include `tuple`, `query.contiguous.reshape`, `key.contiguous.reshape`, `value.contiguous.reshape`, and `_get_cu_seqlens`. The implementation branches on conditions. Parameters such as `query`, `key`, `value`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `XPUAttentionImpl` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `tuple`、`query.contiguous.reshape`、`key.contiguous.reshape`、`value.contiguous.reshape` 和 `_get_cu_seqlens`。 实现中包含条件分支。 本段逻辑主要由 `query`、`key`、`value` 和 `attn_metadata` 等参数驱动。

## Key Concepts / 关键概念
- `XPUAttentionBackend`: Primary class that encapsulates xpuattention backend behavior. / 核心类，用于封装 xpuattention backend 相关行为。
- `_get_cu_seqlens`: Top-level function that retrieves cu seqlens. / 顶层函数，用于获取cu seqlens。
- `XPUAttentionImpl`: Primary class that encapsulates xpuattention impl behavior. / 核心类，用于封装 xpuattention impl 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`
- **Third-party / 第三方依赖**: `torch`, `sgl_kernel.flash_attn`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`, `sglang.multimodal_gen.runtime.layers.attention.backends.flash_attn`

- **Total lines / 总行数**: 122
