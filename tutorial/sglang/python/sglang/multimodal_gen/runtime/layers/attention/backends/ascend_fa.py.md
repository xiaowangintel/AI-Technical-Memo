# ascend_fa.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/attention/backends/ascend_fa.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `AscendFAMetadata`, `AscendFAMetadataBuilder`, and `AscendFABackend`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `AscendFAMetadata`、`AscendFAMetadataBuilder` 和 `AscendFABackend` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass
from typing import Any

import torch

from sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend import (
    AttentionBackend,
    AttentionImpl,
    AttentionMetadata,
    AttentionMetadataBuilder,
)
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `dataclasses`, `typing`, `torch`, `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`, `sglang.multimodal_gen.runtime.platforms`, and `sglang.multimodal_gen.runtime.utils.logging_utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`typing`、`torch`、`sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`、`sglang.multimodal_gen.runtime.platforms` 和 `sglang.multimodal_gen.runtime.utils.logging_utils`。这些依赖为后续实现提供所需符号。

### Lines 15-15: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 19-19: `AscendFAMetadata` class overview / `AscendFAMetadata` 类概览
```python
class AscendFAMetadata:
```
**EN:** This block defines class `AscendFAMetadata`. It encapsulates ascend fametadata behavior.
**CN:** 该代码块定义了类 `AscendFAMetadata`。 它用于封装 ascend fametadata 相关行为。

### Lines 20-20: supporting statements / 辅助语句
```python
    pass
```
**EN:** This block gathers supporting statements inside `AscendFAMetadata`.
**CN:** 该代码块汇集了位于 `AscendFAMetadata` 内部的辅助语句。

### Lines 23-23: `AscendFAMetadataBuilder` class overview / `AscendFAMetadataBuilder` 类概览
```python
class AscendFAMetadataBuilder(AttentionMetadataBuilder):
```
**EN:** This block defines class `AscendFAMetadataBuilder`. It encapsulates ascend fametadata builder behavior. It inherits from `AttentionMetadataBuilder`.
**CN:** 该代码块定义了类 `AscendFAMetadataBuilder`。 它用于封装 ascend fametadata builder 相关行为。 它继承自 `AttentionMetadataBuilder`。

### Lines 24-25: `__init__` implementation / `__init__` 实现
```python
    def __init__(self) -> None:
        pass
```
**EN:** This block defines method `__init__` on `AscendFAMetadataBuilder`. It initializes the instance state.
**CN:** 该代码块定义了 `AscendFAMetadataBuilder` 的方法 `__init__`。 它用于初始化实例状态。

### Lines 27-28: `prepare` implementation / `prepare` 实现
```python
    def prepare(self) -> None:
        pass
```
**EN:** This block defines method `prepare` on `AscendFAMetadataBuilder`. It prepares function.
**CN:** 该代码块定义了 `AscendFAMetadataBuilder` 的方法 `prepare`。 它用于准备函数。

### Lines 30-34: `build` implementation / `build` 实现
```python
    def build(
        self,
        **kwargs: dict[str, Any],
    ) -> AttentionMetadata:
        return AscendFAMetadata()
```
**EN:** This block defines method `build` on `AscendFAMetadataBuilder`. It builds function. Key calls include `AscendFAMetadata`.
**CN:** 该代码块定义了 `AscendFAMetadataBuilder` 的方法 `build`。 它用于构建函数。 关键调用包括 `AscendFAMetadata`。

### Lines 37-38: `AscendFABackend` class overview / `AscendFABackend` 类概览
```python
class AscendFABackend(AttentionBackend):
```
**EN:** This block defines class `AscendFABackend`. It encapsulates ascend fabackend behavior. It inherits from `AttentionBackend`.
**CN:** 该代码块定义了类 `AscendFABackend`。 它用于封装 ascend fabackend 相关行为。 它继承自 `AttentionBackend`。

### Lines 39-41: `get_enum` implementation / `get_enum` 实现
```python
    @staticmethod
    def get_enum() -> AttentionBackendEnum:
        return AttentionBackendEnum.FA
```
**EN:** This block defines method `get_enum` on `AscendFABackend`. It retrieves enum.
**CN:** 该代码块定义了 `AscendFABackend` 的方法 `get_enum`。 它用于获取enum。

### Lines 43-45: `get_impl_cls` implementation / `get_impl_cls` 实现
```python
    @staticmethod
    def get_impl_cls() -> type["AscendFAImpl"]:
        return AscendFAImpl
```
**EN:** This block defines method `get_impl_cls` on `AscendFABackend`. It retrieves impl cls.
**CN:** 该代码块定义了 `AscendFABackend` 的方法 `get_impl_cls`。 它用于获取impl cls。

### Lines 47-49: `get_metadata_cls` implementation / `get_metadata_cls` 实现
```python
    @staticmethod
    def get_metadata_cls() -> type["AttentionMetadata"]:
        raise NotImplementedError
```
**EN:** This block defines method `get_metadata_cls` on `AscendFABackend`. It retrieves metadata cls.
**CN:** 该代码块定义了 `AscendFABackend` 的方法 `get_metadata_cls`。 它用于获取metadata cls。

### Lines 51-53: `get_builder_cls` implementation / `get_builder_cls` 实现
```python
    @staticmethod
    def get_builder_cls() -> type["AttentionMetadataBuilder"]:
        return AscendFAMetadataBuilder
```
**EN:** This block defines method `get_builder_cls` on `AscendFABackend`. It retrieves builder cls.
**CN:** 该代码块定义了 `AscendFABackend` 的方法 `get_builder_cls`。 它用于获取builder cls。

### Lines 56-57: `AscendFAImpl` class overview / `AscendFAImpl` 类概览
```python
class AscendFAImpl(AttentionImpl):
```
**EN:** This block defines class `AscendFAImpl`. It encapsulates ascend faimpl behavior. It inherits from `AttentionImpl`.
**CN:** 该代码块定义了类 `AscendFAImpl`。 它用于封装 ascend faimpl 相关行为。 它继承自 `AttentionImpl`。

### Lines 58-69: `__init__` implementation / `__init__` 实现
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
**EN:** This block defines method `__init__` on `AscendFAImpl`. It initializes the instance state. Parameters such as `num_heads`, `head_size`, `causal`, `softmax_scale`, and `num_kv_heads` drive the behavior in this section.
**CN:** 该代码块定义了 `AscendFAImpl` 的方法 `__init__`。 它用于初始化实例状态。 本段逻辑主要由 `num_heads`、`head_size`、`causal`、`softmax_scale` 和 `num_kv_heads` 等参数驱动。

### Lines 71-104: `forward` implementation / `forward` 实现
```python
    def forward(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        attn_metadata: AttentionMetadata,
        return_softmax_lse: bool = False,
    ) -> torch.Tensor:
        mask = None
        num_heads, num_key_value_heads = query.shape[2], key.shape[2]
        if self.causal:
            seq_len = query.shape[1]
            mask = torch.triu(
                torch.ones(seq_len, seq_len, device=query.device), diagonal=1
            ).bool()
        # transpose to bs, heads, seq_len, head_dim
        query = query.transpose(1, 2)
        key = key.transpose(1, 2)
        value = value.transpose(1, 2)
        output, lse = torch.ops.npu.npu_fused_infer_attention_score(
            query,
            key,
            value,
            num_heads=num_heads,
            num_key_value_heads=num_key_value_heads,
            scale=self.softmax_scale,
            input_layout="BNSD",
            softmax_lse_flag=return_softmax_lse,
            atten_mask=mask,
        )
        output = output.transpose(1, 2)
        if return_softmax_lse:
            return output, lse
        return output
```
**EN:** This block defines method `forward` on `AscendFAImpl`. It executes function. Key calls include `query.transpose`, `key.transpose`, `value.transpose`, `torch.ops.npu.npu_fused_infer_attention_score`, and `output.transpose`. The implementation branches on conditions. Parameters such as `query`, `key`, `value`, `attn_metadata`, and `return_softmax_lse` drive the behavior in this section.
**CN:** 该代码块定义了 `AscendFAImpl` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `query.transpose`、`key.transpose`、`value.transpose`、`torch.ops.npu.npu_fused_infer_attention_score` 和 `output.transpose`。 实现中包含条件分支。 本段逻辑主要由 `query`、`key`、`value`、`attn_metadata` 和 `return_softmax_lse` 等参数驱动。

## Key Concepts / 关键概念
- `AscendFAMetadata`: Primary class that encapsulates ascend fametadata behavior. / 核心类，用于封装 ascend fametadata 相关行为。
- `AscendFAMetadataBuilder`: Primary class that encapsulates ascend fametadata builder behavior. / 核心类，用于封装 ascend fametadata builder 相关行为。
- `AscendFABackend`: Primary class that encapsulates ascend fabackend behavior. / 核心类，用于封装 ascend fabackend 相关行为。
- `AscendFAImpl`: Primary class that encapsulates ascend faimpl behavior. / 核心类，用于封装 ascend faimpl 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 104
