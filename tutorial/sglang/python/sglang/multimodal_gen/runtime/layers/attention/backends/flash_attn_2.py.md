# flash_attn_2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/attention/backends/flash_attn_2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `FlashAttention2Backend`, and `FlashAttention2Impl`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `FlashAttention2Backend` 和 `FlashAttention2Impl` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 5-17: module setup and imports / 模块初始化与导入
```python
import torch

from sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend import (
    AttentionBackend,
    AttentionImpl,
    AttentionMetadata,
    AttentionMetadataBuilder,
)
from sglang.multimodal_gen.runtime.layers.attention.backends.flash_attn import (
    flash_attn_func,
)
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `torch`, `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`, `sglang.multimodal_gen.runtime.layers.attention.backends.flash_attn`, `sglang.multimodal_gen.runtime.platforms`, and `sglang.multimodal_gen.runtime.utils.logging_utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `torch`、`sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`、`sglang.multimodal_gen.runtime.layers.attention.backends.flash_attn`、`sglang.multimodal_gen.runtime.platforms` 和 `sglang.multimodal_gen.runtime.utils.logging_utils`。这些依赖为后续实现提供所需符号。

### Lines 19-19: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 22-22: `FlashAttention2Backend` class overview / `FlashAttention2Backend` 类概览
```python
class FlashAttention2Backend(AttentionBackend):
```
**EN:** This block defines class `FlashAttention2Backend`. It encapsulates flash attention2 backend behavior. It inherits from `AttentionBackend`.
**CN:** 该代码块定义了类 `FlashAttention2Backend`。 它用于封装 flash attention2 backend 相关行为。 它继承自 `AttentionBackend`。

### Lines 23-23: supporting statements / 辅助语句
```python
    accept_output_buffer: bool = True
```
**EN:** This block gathers supporting statements inside `FlashAttention2Backend`. It updates names such as `accept_output_buffer`.
**CN:** 该代码块汇集了位于 `FlashAttention2Backend` 内部的辅助语句。 它会更新 `accept_output_buffer` 等名称。

### Lines 25-27: `get_supported_head_sizes` implementation / `get_supported_head_sizes` 实现
```python
    @staticmethod
    def get_supported_head_sizes() -> list[int]:
        return [32, 64, 96, 128, 160, 192, 224, 256]
```
**EN:** This block defines method `get_supported_head_sizes` on `FlashAttention2Backend`. It retrieves supported head sizes.
**CN:** 该代码块定义了 `FlashAttention2Backend` 的方法 `get_supported_head_sizes`。 它用于获取supported head sizes。

### Lines 29-31: `get_enum` implementation / `get_enum` 实现
```python
    @staticmethod
    def get_enum() -> AttentionBackendEnum:
        return AttentionBackendEnum.FA2
```
**EN:** This block defines method `get_enum` on `FlashAttention2Backend`. It retrieves enum.
**CN:** 该代码块定义了 `FlashAttention2Backend` 的方法 `get_enum`。 它用于获取enum。

### Lines 33-35: `get_impl_cls` implementation / `get_impl_cls` 实现
```python
    @staticmethod
    def get_impl_cls() -> type["FlashAttention2Impl"]:
        return FlashAttention2Impl
```
**EN:** This block defines method `get_impl_cls` on `FlashAttention2Backend`. It retrieves impl cls.
**CN:** 该代码块定义了 `FlashAttention2Backend` 的方法 `get_impl_cls`。 它用于获取impl cls。

### Lines 37-39: `get_metadata_cls` implementation / `get_metadata_cls` 实现
```python
    @staticmethod
    def get_metadata_cls() -> type["AttentionMetadata"]:
        raise NotImplementedError
```
**EN:** This block defines method `get_metadata_cls` on `FlashAttention2Backend`. It retrieves metadata cls.
**CN:** 该代码块定义了 `FlashAttention2Backend` 的方法 `get_metadata_cls`。 它用于获取metadata cls。

### Lines 41-43: `get_builder_cls` implementation / `get_builder_cls` 实现
```python
    @staticmethod
    def get_builder_cls() -> type["AttentionMetadataBuilder"]:
        raise NotImplementedError
```
**EN:** This block defines method `get_builder_cls` on `FlashAttention2Backend`. It retrieves builder cls.
**CN:** 该代码块定义了 `FlashAttention2Backend` 的方法 `get_builder_cls`。 它用于获取builder cls。

### Lines 46-47: `FlashAttention2Impl` class overview / `FlashAttention2Impl` 类概览
```python
class FlashAttention2Impl(AttentionImpl):
```
**EN:** This block defines class `FlashAttention2Impl`. It encapsulates flash attention2 impl behavior. It inherits from `AttentionImpl`.
**CN:** 该代码块定义了类 `FlashAttention2Impl`。 它用于封装 flash attention2 impl 相关行为。 它继承自 `AttentionImpl`。

### Lines 48-59: `__init__` implementation / `__init__` 实现
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
        self.causal = causal
        self.softmax_scale = softmax_scale
```
**EN:** This block defines method `__init__` on `FlashAttention2Impl`. It initializes the instance state. Parameters such as `num_heads`, `head_size`, `causal`, `softmax_scale`, and `num_kv_heads` drive the behavior in this section.
**CN:** 该代码块定义了 `FlashAttention2Impl` 的方法 `__init__`。 它用于初始化实例状态。 本段逻辑主要由 `num_heads`、`head_size`、`causal`、`softmax_scale` 和 `num_kv_heads` 等参数驱动。

### Lines 61-79: `forward` implementation / `forward` 实现
```python
    def forward(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        attn_metadata: AttentionMetadata,
    ):
        output = flash_attn_func(
            q=query,  # type: ignore[no-untyped-call]
            k=key,
            v=value,
            cu_seqlens_q=None,
            cu_seqlens_k=None,
            max_seqlen_q=None,
            max_seqlen_k=None,
            softmax_scale=self.softmax_scale,
            causal=self.causal,
        )
        return output
```
**EN:** This block defines method `forward` on `FlashAttention2Impl`. It executes function. Key calls include `flash_attn_func`. Parameters such as `query`, `key`, `value`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `FlashAttention2Impl` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `flash_attn_func`。 本段逻辑主要由 `query`、`key`、`value` 和 `attn_metadata` 等参数驱动。

## Key Concepts / 关键概念
- `FlashAttention2Backend`: Primary class that encapsulates flash attention2 backend behavior. / 核心类，用于封装 flash attention2 backend 相关行为。
- `FlashAttention2Impl`: Primary class that encapsulates flash attention2 impl behavior. / 核心类，用于封装 flash attention2 impl 相关行为。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`, `sglang.multimodal_gen.runtime.layers.attention.backends.flash_attn`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 79
