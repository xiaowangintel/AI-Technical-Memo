# sage_attn3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/attention/backends/sage_attn3.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `SageAttention3Backend`, and `SageAttention3Impl`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `SageAttention3Backend` 和 `SageAttention3Impl` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 5-15: module setup and imports / 模块初始化与导入
```python
import torch
import torch.nn.functional as F
from sageattn3 import sageattn3_blackwell

from sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend import (
    AttentionBackend,
    AttentionImpl,
    AttentionMetadata,
)
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `torch`, `torch.nn.functional`, `sageattn3`, `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`, `sglang.multimodal_gen.runtime.platforms`, and `sglang.multimodal_gen.runtime.utils.logging_utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `torch`、`torch.nn.functional`、`sageattn3`、`sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`、`sglang.multimodal_gen.runtime.platforms` 和 `sglang.multimodal_gen.runtime.utils.logging_utils`。这些依赖为后续实现提供所需符号。

### Lines 17-17: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 20-20: `SageAttention3Backend` class overview / `SageAttention3Backend` 类概览
```python
class SageAttention3Backend(AttentionBackend):
```
**EN:** This block defines class `SageAttention3Backend`. It encapsulates sage attention3 backend behavior. It inherits from `AttentionBackend`.
**CN:** 该代码块定义了类 `SageAttention3Backend`。 它用于封装 sage attention3 backend 相关行为。 它继承自 `AttentionBackend`。

### Lines 21-21: supporting statements / 辅助语句
```python
    accept_output_buffer: bool = True
```
**EN:** This block gathers supporting statements inside `SageAttention3Backend`. It updates names such as `accept_output_buffer`.
**CN:** 该代码块汇集了位于 `SageAttention3Backend` 内部的辅助语句。 它会更新 `accept_output_buffer` 等名称。

### Lines 23-25: `get_supported_head_sizes` implementation / `get_supported_head_sizes` 实现
```python
    @staticmethod
    def get_supported_head_sizes() -> list[int]:
        return [64, 128, 256]
```
**EN:** This block defines method `get_supported_head_sizes` on `SageAttention3Backend`. It retrieves supported head sizes.
**CN:** 该代码块定义了 `SageAttention3Backend` 的方法 `get_supported_head_sizes`。 它用于获取supported head sizes。

### Lines 27-29: `get_enum` implementation / `get_enum` 实现
```python
    @staticmethod
    def get_enum() -> AttentionBackendEnum:
        return AttentionBackendEnum.SAGE_ATTN_3
```
**EN:** This block defines method `get_enum` on `SageAttention3Backend`. It retrieves enum.
**CN:** 该代码块定义了 `SageAttention3Backend` 的方法 `get_enum`。 它用于获取enum。

### Lines 31-33: `get_impl_cls` implementation / `get_impl_cls` 实现
```python
    @staticmethod
    def get_impl_cls() -> type["SageAttention3Impl"]:
        return SageAttention3Impl
```
**EN:** This block defines method `get_impl_cls` on `SageAttention3Backend`. It retrieves impl cls.
**CN:** 该代码块定义了 `SageAttention3Backend` 的方法 `get_impl_cls`。 它用于获取impl cls。

### Lines 35-37: `get_metadata_cls` implementation / `get_metadata_cls` 实现
```python
    @staticmethod
    def get_metadata_cls() -> type["AttentionMetadata"]:
        raise NotImplementedError
```
**EN:** This block defines method `get_metadata_cls` on `SageAttention3Backend`. It retrieves metadata cls.
**CN:** 该代码块定义了 `SageAttention3Backend` 的方法 `get_metadata_cls`。 它用于获取metadata cls。

### Lines 40-40: `SageAttention3Impl` class overview / `SageAttention3Impl` 类概览
```python
class SageAttention3Impl(AttentionImpl):
```
**EN:** This block defines class `SageAttention3Impl`. It encapsulates sage attention3 impl behavior. It inherits from `AttentionImpl`.
**CN:** 该代码块定义了类 `SageAttention3Impl`。 它用于封装 sage attention3 impl 相关行为。 它继承自 `AttentionImpl`。

### Lines 41-41: supporting statements / 辅助语句
```python
    _warned_gqa_fallback_global: bool = False
```
**EN:** This block gathers supporting statements inside `SageAttention3Impl`. It updates names such as `_warned_gqa_fallback_global`.
**CN:** 该代码块汇集了位于 `SageAttention3Impl` 内部的辅助语句。 它会更新 `_warned_gqa_fallback_global` 等名称。

### Lines 43-55: `__init__` implementation / `__init__` 实现
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
        self.dropout = extra_impl_args.get("dropout_p", 0.0)
```
**EN:** This block defines method `__init__` on `SageAttention3Impl`. It initializes the instance state. Key calls include `extra_impl_args.get`. Parameters such as `num_heads`, `head_size`, `causal`, `softmax_scale`, and `num_kv_heads` drive the behavior in this section.
**CN:** 该代码块定义了 `SageAttention3Impl` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `extra_impl_args.get`。 本段逻辑主要由 `num_heads`、`head_size`、`causal`、`softmax_scale` 和 `num_kv_heads` 等参数驱动。

### Lines 57-92: `forward` implementation / `forward` 实现
```python
    def forward(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        attn_metadata: AttentionMetadata,
    ) -> torch.Tensor:
        query = query.transpose(1, 2)
        key = key.transpose(1, 2)
        value = value.transpose(1, 2)
        # SageAttention3's Blackwell kernel assumes MHA (Hq == Hkv). For GQA/MQA
        # (Hq != Hkv), fall back to torch SDPA which supports GQA.
        if key.shape[1] != query.shape[1]:
            if query.shape[1] % key.shape[1] != 0:
                raise ValueError(
                    "GQA/MQA requires query heads to be a multiple of KV heads, "
                    f"got q_heads={query.shape[1]} and kv_heads={key.shape[1]}"
                )
            if not type(self)._warned_gqa_fallback_global:
                logger.warning(
                    "SageAttention3 does not support GQA/MQA (Hq != Hkv); falling back to torch SDPA."
                )
                type(self)._warned_gqa_fallback_global = True
            output = F.scaled_dot_product_attention(
                query,
                key,
                value,
                is_causal=self.causal,
                dropout_p=self.dropout,
                scale=self.softmax_scale,
                enable_gqa=True,
            )
        else:
            output = sageattn3_blackwell(query, key, value, is_causal=self.causal)
        output = output.transpose(1, 2)
        return output
```
**EN:** This block defines method `forward` on `SageAttention3Impl`. It executes function. Key calls include `query.transpose`, `key.transpose`, `value.transpose`, `output.transpose`, and `F.scaled_dot_product_attention`. The implementation branches on conditions. Parameters such as `query`, `key`, `value`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `SageAttention3Impl` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `query.transpose`、`key.transpose`、`value.transpose`、`output.transpose` 和 `F.scaled_dot_product_attention`。 实现中包含条件分支。 本段逻辑主要由 `query`、`key`、`value` 和 `attn_metadata` 等参数驱动。

## Key Concepts / 关键概念
- `SageAttention3Backend`: Primary class that encapsulates sage attention3 backend behavior. / 核心类，用于封装 sage attention3 backend 相关行为。
- `SageAttention3Impl`: Primary class that encapsulates sage attention3 impl behavior. / 核心类，用于封装 sage attention3 impl 相关行为。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `torch`, `torch.nn.functional`, `sageattn3`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 92
