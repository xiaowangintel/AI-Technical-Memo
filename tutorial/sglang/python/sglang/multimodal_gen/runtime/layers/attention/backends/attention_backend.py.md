# attention_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/attention/backends/attention_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `AttentionBackend`, `AttentionMetadata`, and `AttentionMetadataBuilder`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `AttentionBackend`、`AttentionMetadata` 和 `AttentionMetadataBuilder` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 6-8: module setup and imports / 模块初始化与导入
```python
from abc import ABC, abstractmethod
from dataclasses import dataclass, fields
from typing import TYPE_CHECKING, Any, Generic, Protocol, TypeVar
```
**EN:** This block establishes the module context and imports `abc`, `dataclasses`, and `typing`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `abc`、`dataclasses` 和 `typing`。这些依赖为后续实现提供所需符号。

### Lines 10-16: supporting statements / 辅助语句
```python
if TYPE_CHECKING:
    pass

import torch

from sglang.kernel_api_logging import wrap_method_with_debug_kernel_once
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
```
**EN:** This block gathers supporting statements at module scope.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。

### Lines 19-24: `AttentionBackend` class overview / `AttentionBackend` 类概览
```python
class AttentionBackend(ABC):
    """Abstract class for attention backends."""

    # For some attention backends, we allocate an output tensor before
    # calling the custom op. When piecewise cudagraph is enabled, this
    # makes sure the output tensor is allocated inside the cudagraph.
```
**EN:** This block defines class `AttentionBackend`. Abstract class for attention backends. It inherits from `ABC`.
**CN:** 该代码块定义了类 `AttentionBackend`。 它用于封装 attention backend 相关行为。 它继承自 `ABC`。

### Lines 25-25: supporting statements / 辅助语句
```python
    accept_output_buffer: bool = False
```
**EN:** This block gathers supporting statements inside `AttentionBackend`. It updates names such as `accept_output_buffer`.
**CN:** 该代码块汇集了位于 `AttentionBackend` 内部的辅助语句。 它会更新 `accept_output_buffer` 等名称。

### Lines 27-30: `get_enum` implementation / `get_enum` 实现
```python
    @staticmethod
    @abstractmethod
    def get_enum() -> AttentionBackendEnum:
        raise NotImplementedError
```
**EN:** This block defines method `get_enum` on `AttentionBackend`. It retrieves enum.
**CN:** 该代码块定义了 `AttentionBackend` 的方法 `get_enum`。 它用于获取enum。

### Lines 32-35: `get_impl_cls` implementation / `get_impl_cls` 实现
```python
    @staticmethod
    @abstractmethod
    def get_impl_cls() -> type["AttentionImpl"]:
        raise NotImplementedError
```
**EN:** This block defines method `get_impl_cls` on `AttentionBackend`. It retrieves impl cls.
**CN:** 该代码块定义了 `AttentionBackend` 的方法 `get_impl_cls`。 它用于获取impl cls。

### Lines 37-40: `get_metadata_cls` implementation / `get_metadata_cls` 实现
```python
    @staticmethod
    @abstractmethod
    def get_metadata_cls() -> type["AttentionMetadata"]:
        raise NotImplementedError
```
**EN:** This block defines method `get_metadata_cls` on `AttentionBackend`. It retrieves metadata cls.
**CN:** 该代码块定义了 `AttentionBackend` 的方法 `get_metadata_cls`。 它用于获取metadata cls。

### Lines 51-54: `get_builder_cls` implementation / `get_builder_cls` 实现
```python
    @staticmethod
    @abstractmethod
    def get_builder_cls() -> type["AttentionMetadataBuilder"]:
        return None
```
**EN:** This block defines method `get_builder_cls` on `AttentionBackend`. It retrieves builder cls.
**CN:** 该代码块定义了 `AttentionBackend` 的方法 `get_builder_cls`。 它用于获取builder cls。

### Lines 58-61: `AttentionMetadata` class overview / `AttentionMetadata` 类概览
```python
class AttentionMetadata:
    """Attention metadata for prefill and decode batched together."""

    # Current step of diffusion process
```
**EN:** This block defines class `AttentionMetadata`. Attention metadata for prefill and decode batched together.
**CN:** 该代码块定义了类 `AttentionMetadata`。 它用于封装 attention metadata 相关行为。

### Lines 62-62: supporting statements / 辅助语句
```python
    current_timestep: int
```
**EN:** This block gathers supporting statements inside `AttentionMetadata`. It updates names such as `current_timestep`.
**CN:** 该代码块汇集了位于 `AttentionMetadata` 内部的辅助语句。 它会更新 `current_timestep` 等名称。

### Lines 64-74: `asdict_zerocopy` implementation / `asdict_zerocopy` 实现
```python
    def asdict_zerocopy(self, skip_fields: set[str] | None = None) -> dict[str, Any]:
        """Similar to dataclasses.asdict, but avoids deepcopying."""
        if skip_fields is None:
            skip_fields = set()
        # Note that if we add dataclasses as fields, they will need
        # similar handling.
        return {
            field.name: getattr(self, field.name)
            for field in fields(self)
            if field.name not in skip_fields
        }
```
**EN:** This block defines method `asdict_zerocopy` on `AttentionMetadata`. Similar to dataclasses.asdict, but avoids deepcopying. Key calls include `set`, `getattr`, and `fields`. The implementation branches on conditions. Parameters such as `skip_fields` drive the behavior in this section.
**CN:** 该代码块定义了 `AttentionMetadata` 的方法 `asdict_zerocopy`。 它用于处理 asdict zerocopy 相关逻辑。 关键调用包括 `set`、`getattr` 和 `fields`。 实现中包含条件分支。 本段逻辑主要由 `skip_fields` 等参数驱动。

### Lines 77-77: supporting statements / 辅助语句
```python
T = TypeVar("T", bound=AttentionMetadata)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `T`. The code collaborates with `TypeVar`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `T` 等名称。 代码会与 `TypeVar` 协同工作。

### Lines 80-82: `AttentionMetadataBuilder` class overview / `AttentionMetadataBuilder` 类概览
```python
class AttentionMetadataBuilder(ABC, Generic[T]):
    """Abstract class for attention metadata builders."""
```
**EN:** This block defines class `AttentionMetadataBuilder`. Abstract class for attention metadata builders. It inherits from `ABC`, and `Generic`.
**CN:** 该代码块定义了类 `AttentionMetadataBuilder`。 它用于封装 attention metadata builder 相关行为。 它继承自 `ABC` 和 `Generic`。

### Lines 83-86: `__init__` implementation / `__init__` 实现
```python
    @abstractmethod
    def __init__(self) -> None:
        """Create the builder, remember some configuration and parameters."""
        raise NotImplementedError
```
**EN:** This block defines method `__init__` on `AttentionMetadataBuilder`. Create the builder, remember some configuration and parameters.
**CN:** 该代码块定义了 `AttentionMetadataBuilder` 的方法 `__init__`。 它用于初始化实例状态。

### Lines 88-91: `prepare` implementation / `prepare` 实现
```python
    @abstractmethod
    def prepare(self) -> None:
        """Prepare for one batch."""
        raise NotImplementedError
```
**EN:** This block defines method `prepare` on `AttentionMetadataBuilder`. Prepare for one batch.
**CN:** 该代码块定义了 `AttentionMetadataBuilder` 的方法 `prepare`。 它用于准备函数。

### Lines 93-99: `build` implementation / `build` 实现
```python
    @abstractmethod
    def build(
        self,
        **kwargs: dict[str, Any],
    ) -> AttentionMetadata:
        """Build attention metadata with on-device tensors."""
        raise NotImplementedError
```
**EN:** This block defines method `build` on `AttentionMetadataBuilder`. Build attention metadata with on-device tensors.
**CN:** 该代码块定义了 `AttentionMetadataBuilder` 的方法 `build`。 它用于构建函数。

### Lines 102-103: `AttentionLayer` class overview / `AttentionLayer` 类概览
```python
class AttentionLayer(Protocol):
```
**EN:** This block defines class `AttentionLayer`. It encapsulates attention layer behavior. It inherits from `Protocol`.
**CN:** 该代码块定义了类 `AttentionLayer`。 它用于封装 attention layer 相关行为。 它继承自 `Protocol`。

### Lines 104-107: supporting statements / 辅助语句
```python
    _k_scale: torch.Tensor
    _v_scale: torch.Tensor
    _k_scale_float: float
    _v_scale_float: float
```
**EN:** This block gathers supporting statements inside `AttentionLayer`. It updates names such as `_k_scale`, `_v_scale`, `_k_scale_float`, and `_v_scale_float`.
**CN:** 该代码块汇集了位于 `AttentionLayer` 内部的辅助语句。 它会更新 `_k_scale`、`_v_scale`、`_k_scale_float` 和 `_v_scale_float` 等名称。

### Lines 109-116: `forward` implementation / `forward` 实现
```python
    def forward(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        kv_cache: torch.Tensor,
        attn_metadata: AttentionMetadata,
    ) -> torch.Tensor: ...
```
**EN:** This block defines method `forward` on `AttentionLayer`. It executes function. Parameters such as `query`, `key`, `value`, `kv_cache`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `AttentionLayer` 的方法 `forward`。 它用于执行前向计算函数。 本段逻辑主要由 `query`、`key`、`value`、`kv_cache` 和 `attn_metadata` 等参数驱动。

### Lines 119-120: `AttentionImpl` class overview / `AttentionImpl` 类概览
```python
class AttentionImpl(ABC, Generic[T]):
```
**EN:** This block defines class `AttentionImpl`. It encapsulates attention impl behavior. It inherits from `ABC`, and `Generic`.
**CN:** 该代码块定义了类 `AttentionImpl`。 它用于封装 attention impl 相关行为。 它继承自 `ABC` 和 `Generic`。

### Lines 121-132: `__init__` implementation / `__init__` 实现
```python
    @abstractmethod
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        softmax_scale: float,
        causal: bool = False,
        num_kv_heads: int | None = None,
        prefix: str = "",
        **extra_impl_args,
    ) -> None:
        raise NotImplementedError
```
**EN:** This block defines method `__init__` on `AttentionImpl`. It initializes the instance state. Parameters such as `num_heads`, `head_size`, `softmax_scale`, `causal`, and `num_kv_heads` drive the behavior in this section.
**CN:** 该代码块定义了 `AttentionImpl` 的方法 `__init__`。 它用于初始化实例状态。 本段逻辑主要由 `num_heads`、`head_size`、`softmax_scale`、`causal` 和 `num_kv_heads` 等参数驱动。

### Lines 134-144: `preprocess_qkv` implementation / `preprocess_qkv` 实现
```python
    def preprocess_qkv(self, qkv: torch.Tensor, attn_metadata: T) -> torch.Tensor:
        """Preprocess QKV tensor before performing attention operation.

        Default implementation returns the tensor unchanged.
        Subclasses can override this to implement custom preprocessing
        like reshaping, tiling, scaling, or other transformations.

        Called AFTER all_to_all for distributed attention

        """
        return qkv
```
**EN:** This block defines method `preprocess_qkv` on `AttentionImpl`. Preprocess QKV tensor before performing attention operation. Default implementation returns the tensor unchanged. Parameters such as `qkv`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `AttentionImpl` 的方法 `preprocess_qkv`。 它用于处理 preprocess qkv 相关逻辑。 本段逻辑主要由 `qkv` 和 `attn_metadata` 等参数驱动。

### Lines 146-161: `postprocess_output` implementation / `postprocess_output` 实现
```python
    def postprocess_output(
        self,
        output: torch.Tensor,
        attn_metadata: T,
    ) -> torch.Tensor:
        """Postprocess the output tensor after the attention operation.

        Default implementation returns the tensor unchanged.
        Subclasses can override this to implement custom postprocessing
        like untiling, scaling, or other transformations.

        Called BEFORE all_to_all for distributed attention

        """

        return output
```
**EN:** This block defines method `postprocess_output` on `AttentionImpl`. Postprocess the output tensor after the attention operation. Default implementation returns the tensor unchanged. Parameters such as `output`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `AttentionImpl` 的方法 `postprocess_output`。 它用于处理 postprocess output 相关逻辑。 本段逻辑主要由 `output` 和 `attn_metadata` 等参数驱动。

### Lines 163-171: `forward` implementation / `forward` 实现
```python
    @abstractmethod
    def forward(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        attn_metadata: T,
    ) -> torch.Tensor:
        raise NotImplementedError
```
**EN:** This block defines method `forward` on `AttentionImpl`. It executes function. Parameters such as `query`, `key`, `value`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `AttentionImpl` 的方法 `forward`。 它用于执行前向计算函数。 本段逻辑主要由 `query`、`key`、`value` 和 `attn_metadata` 等参数驱动。

### Lines 174-179: `wrap_attention_impl_forward` implementation / `wrap_attention_impl_forward` 实现
```python
def wrap_attention_impl_forward(attn_impl: AttentionImpl) -> AttentionImpl:
    return wrap_method_with_debug_kernel_once(
        attn_impl,
        "forward",
        op_name=f"diffusion.attn_impl.{attn_impl.__class__.__name__}.forward",
    )
```
**EN:** This block defines function `wrap_attention_impl_forward`. It handles wrap attention impl forward logic. Key calls include `wrap_method_with_debug_kernel_once`. Parameters such as `attn_impl` drive the behavior in this section.
**CN:** 该代码块定义了函数 `wrap_attention_impl_forward`。 它用于处理 wrap attention impl forward 相关逻辑。 关键调用包括 `wrap_method_with_debug_kernel_once`。 本段逻辑主要由 `attn_impl` 等参数驱动。

## Key Concepts / 关键概念
- `AttentionBackend`: Abstract class for attention backends. / 核心类，用于封装 attention backend 相关行为。
- `AttentionMetadata`: Attention metadata for prefill and decode batched together. / 核心类，用于封装 attention metadata 相关行为。
- `AttentionMetadataBuilder`: Abstract class for attention metadata builders. / 核心类，用于封装 attention metadata builder 相关行为。
- `AttentionLayer`: Primary class that encapsulates attention layer behavior. / 核心类，用于封装 attention layer 相关行为。
- `AttentionImpl`: Primary class that encapsulates attention impl behavior. / 核心类，用于封装 attention impl 相关行为。
- `wrap_attention_impl_forward`: Top-level function that handles wrap attention impl forward logic. / 顶层函数，用于处理 wrap attention impl forward 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `abc`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.kernel_api_logging`, `sglang.multimodal_gen.runtime.platforms`

- **Total lines / 总行数**: 179
