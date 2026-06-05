# sdpa.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/attention/backends/sdpa.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `SDPABackend`, and `SDPAImpl`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `SDPABackend` 和 `SDPAImpl` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 5-16: module setup and imports / 模块初始化与导入
```python
from contextlib import nullcontext

import torch
from torch.nn.attention import SDPBackend, sdpa_kernel

from sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend import (  # FlashAttentionMetadata,
    AttentionBackend,
    AttentionImpl,
    AttentionMetadata,
)
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `contextlib`, `torch`, `torch.nn.attention`, `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`, `sglang.multimodal_gen.runtime.platforms`, and `sglang.multimodal_gen.runtime.utils.logging_utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `contextlib`、`torch`、`torch.nn.attention`、`sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`、`sglang.multimodal_gen.runtime.platforms` 和 `sglang.multimodal_gen.runtime.utils.logging_utils`。这些依赖为后续实现提供所需符号。

### Lines 18-25: supporting statements / 辅助语句
```python
logger = init_logger(__name__)

_PYTORCH_DEFAULT_CUDA_SDP_BACKENDS = [
    SDPBackend.CUDNN_ATTENTION,
    SDPBackend.FLASH_ATTENTION,
    SDPBackend.EFFICIENT_ATTENTION,
    SDPBackend.MATH,
]
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, and `_PYTORCH_DEFAULT_CUDA_SDP_BACKENDS`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 和 `_PYTORCH_DEFAULT_CUDA_SDP_BACKENDS` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 28-29: `SDPABackend` class overview / `SDPABackend` 类概览
```python
class SDPABackend(AttentionBackend):
```
**EN:** This block defines class `SDPABackend`. It encapsulates sdpabackend behavior. It inherits from `AttentionBackend`.
**CN:** 该代码块定义了类 `SDPABackend`。 它用于封装 sdpabackend 相关行为。 它继承自 `AttentionBackend`。

### Lines 30-30: supporting statements / 辅助语句
```python
    accept_output_buffer: bool = True
```
**EN:** This block gathers supporting statements inside `SDPABackend`. It updates names such as `accept_output_buffer`.
**CN:** 该代码块汇集了位于 `SDPABackend` 内部的辅助语句。 它会更新 `accept_output_buffer` 等名称。

### Lines 32-34: `get_supported_head_sizes` implementation / `get_supported_head_sizes` 实现
```python
    @staticmethod
    def get_supported_head_sizes() -> list[int]:
        return [32, 64, 96, 128, 160, 192, 224, 256]
```
**EN:** This block defines method `get_supported_head_sizes` on `SDPABackend`. It retrieves supported head sizes.
**CN:** 该代码块定义了 `SDPABackend` 的方法 `get_supported_head_sizes`。 它用于获取supported head sizes。

### Lines 36-38: `get_enum` implementation / `get_enum` 实现
```python
    @staticmethod
    def get_enum() -> AttentionBackendEnum:
        return AttentionBackendEnum.TORCH_SDPA
```
**EN:** This block defines method `get_enum` on `SDPABackend`. It retrieves enum.
**CN:** 该代码块定义了 `SDPABackend` 的方法 `get_enum`。 它用于获取enum。

### Lines 40-42: `get_impl_cls` implementation / `get_impl_cls` 实现
```python
    @staticmethod
    def get_impl_cls() -> type["SDPAImpl"]:
        return SDPAImpl
```
**EN:** This block defines method `get_impl_cls` on `SDPABackend`. It retrieves impl cls.
**CN:** 该代码块定义了 `SDPABackend` 的方法 `get_impl_cls`。 它用于获取impl cls。

### Lines 49-50: `SDPAImpl` class overview / `SDPAImpl` 类概览
```python
class SDPAImpl(AttentionImpl):
```
**EN:** This block defines class `SDPAImpl`. It encapsulates sdpaimpl behavior. It inherits from `AttentionImpl`.
**CN:** 该代码块定义了类 `SDPAImpl`。 它用于封装 sdpaimpl 相关行为。 它继承自 `AttentionImpl`。

### Lines 51-64: `__init__` implementation / `__init__` 实现
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
        self.allow_cudnn_sdp = bool(extra_impl_args.get("allow_cudnn_sdp", False))
```
**EN:** This block defines method `__init__` on `SDPAImpl`. It initializes the instance state. Key calls include `extra_impl_args.get`, and `bool`. Parameters such as `num_heads`, `head_size`, `causal`, `softmax_scale`, and `num_kv_heads` drive the behavior in this section.
**CN:** 该代码块定义了 `SDPAImpl` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `extra_impl_args.get` 和 `bool`。 本段逻辑主要由 `num_heads`、`head_size`、`causal`、`softmax_scale` 和 `num_kv_heads` 等参数驱动。

### Lines 66-95: `forward` implementation / `forward` 实现
```python
    def forward(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        attn_metadata: AttentionMetadata,
    ) -> torch.Tensor:
        # transpose to bs, heads, seq_len, head_dim
        query = query.transpose(1, 2)
        key = key.transpose(1, 2)
        value = value.transpose(1, 2)
        attn_kwargs = {
            "attn_mask": None,
            "dropout_p": self.dropout,
            "is_causal": self.causal,
            "scale": self.softmax_scale,
        }
        if query.shape[1] != key.shape[1]:
            attn_kwargs["enable_gqa"] = True
        sdpa_context = (
            sdpa_kernel(_PYTORCH_DEFAULT_CUDA_SDP_BACKENDS)
            if self.allow_cudnn_sdp and query.device.type == "cuda"
            else nullcontext()
        )
        with sdpa_context:
            output = torch.nn.functional.scaled_dot_product_attention(
                query, key, value, **attn_kwargs
            )
        output = output.transpose(1, 2)
        return output
```
**EN:** This block defines method `forward` on `SDPAImpl`. It executes function. Key calls include `query.transpose`, `key.transpose`, `value.transpose`, `output.transpose`, and `sdpa_kernel`. The implementation branches on conditions, uses context-managed resources. Parameters such as `query`, `key`, `value`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `SDPAImpl` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `query.transpose`、`key.transpose`、`value.transpose`、`output.transpose` 和 `sdpa_kernel`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `query`、`key`、`value` 和 `attn_metadata` 等参数驱动。

## Key Concepts / 关键概念
- `SDPABackend`: Primary class that encapsulates sdpabackend behavior. / 核心类，用于封装 sdpabackend 相关行为。
- `SDPAImpl`: Primary class that encapsulates sdpaimpl behavior. / 核心类，用于封装 sdpaimpl 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `contextlib`
- **Third-party / 第三方依赖**: `torch`, `torch.nn.attention`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 95
