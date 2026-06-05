# vmoba.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/attention/backends/vmoba.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `VMOBAAttentionBackend`, `VideoMobaAttentionMetadata`, and `VideoMobaAttentionMetadataBuilder`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `VMOBAAttentionBackend`、`VideoMobaAttentionMetadata` 和 `VideoMobaAttentionMetadataBuilder` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 5-23: module setup and imports / 模块初始化与导入
```python
import re
from dataclasses import dataclass

import torch
from einops import rearrange
from kernel.attn.vmoba_attn.vmoba import (
    moba_attn_varlen,
    process_moba_input,
    process_moba_output,
)

from sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend import (
    AttentionBackend,
    AttentionImpl,
    AttentionMetadata,
    AttentionMetadataBuilder,
)
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `re`, `dataclasses`, `torch`, `einops`, `kernel.attn.vmoba_attn.vmoba`, and `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `re`、`dataclasses`、`torch`、`einops`、`kernel.attn.vmoba_attn.vmoba` 和 `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`。这些依赖为后续实现提供所需符号。

### Lines 25-25: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 28-29: `VMOBAAttentionBackend` class overview / `VMOBAAttentionBackend` 类概览
```python
class VMOBAAttentionBackend(AttentionBackend):
```
**EN:** This block defines class `VMOBAAttentionBackend`. It encapsulates vmobaattention backend behavior. It inherits from `AttentionBackend`.
**CN:** 该代码块定义了类 `VMOBAAttentionBackend`。 它用于封装 vmobaattention backend 相关行为。 它继承自 `AttentionBackend`。

### Lines 30-30: supporting statements / 辅助语句
```python
    accept_output_buffer: bool = True
```
**EN:** This block gathers supporting statements inside `VMOBAAttentionBackend`. It updates names such as `accept_output_buffer`.
**CN:** 该代码块汇集了位于 `VMOBAAttentionBackend` 内部的辅助语句。 它会更新 `accept_output_buffer` 等名称。

### Lines 32-34: `get_enum` implementation / `get_enum` 实现
```python
    @staticmethod
    def get_enum() -> AttentionBackendEnum:
        return AttentionBackendEnum.VMOBA_ATTN
```
**EN:** This block defines method `get_enum` on `VMOBAAttentionBackend`. It retrieves enum.
**CN:** 该代码块定义了 `VMOBAAttentionBackend` 的方法 `get_enum`。 它用于获取enum。

### Lines 36-38: `get_impl_cls` implementation / `get_impl_cls` 实现
```python
    @staticmethod
    def get_impl_cls() -> type["VMOBAAttentionImpl"]:
        return VMOBAAttentionImpl
```
**EN:** This block defines method `get_impl_cls` on `VMOBAAttentionBackend`. It retrieves impl cls.
**CN:** 该代码块定义了 `VMOBAAttentionBackend` 的方法 `get_impl_cls`。 它用于获取impl cls。

### Lines 40-42: `get_metadata_cls` implementation / `get_metadata_cls` 实现
```python
    @staticmethod
    def get_metadata_cls() -> type["VideoMobaAttentionMetadata"]:
        return VideoMobaAttentionMetadata
```
**EN:** This block defines method `get_metadata_cls` on `VMOBAAttentionBackend`. It retrieves metadata cls.
**CN:** 该代码块定义了 `VMOBAAttentionBackend` 的方法 `get_metadata_cls`。 它用于获取metadata cls。

### Lines 44-46: `get_builder_cls` implementation / `get_builder_cls` 实现
```python
    @staticmethod
    def get_builder_cls() -> type["VideoMobaAttentionMetadataBuilder"]:
        return VideoMobaAttentionMetadataBuilder
```
**EN:** This block defines method `get_builder_cls` on `VMOBAAttentionBackend`. It retrieves builder cls.
**CN:** 该代码块定义了 `VMOBAAttentionBackend` 的方法 `get_builder_cls`。 它用于获取builder cls。

### Lines 50-50: `VideoMobaAttentionMetadata` class overview / `VideoMobaAttentionMetadata` 类概览
```python
class VideoMobaAttentionMetadata(AttentionMetadata):
```
**EN:** This block defines class `VideoMobaAttentionMetadata`. It encapsulates video moba attention metadata behavior. It inherits from `AttentionMetadata`.
**CN:** 该代码块定义了类 `VideoMobaAttentionMetadata`。 它用于封装 video moba attention metadata 相关行为。 它继承自 `AttentionMetadata`。

### Lines 51-70: supporting statements / 辅助语句
```python
    current_timestep: int

    temporal_chunk_size: int
    temporal_topk: int
    spatial_chunk_size: tuple[int, int]
    spatial_topk: int
    st_chunk_size: tuple[int, int, int]
    st_topk: int

    moba_select_mode: str
    moba_threshold: float
    moba_threshold_type: str
    patch_resolution: list[int]

    first_full_step: int = 12
    first_full_layer: int = 0
    # temporal_layer -> spatial_layer -> st_layer
    temporal_layer: int = 1
    spatial_layer: int = 1
    st_layer: int = 1
```
**EN:** This block gathers supporting statements inside `VideoMobaAttentionMetadata`. It updates names such as `current_timestep`, `temporal_chunk_size`, `temporal_topk`, `spatial_chunk_size`, `spatial_topk`, and `st_chunk_size`.
**CN:** 该代码块汇集了位于 `VideoMobaAttentionMetadata` 内部的辅助语句。 它会更新 `current_timestep`、`temporal_chunk_size`、`temporal_topk`、`spatial_chunk_size`、`spatial_topk` 和 `st_chunk_size` 等名称。

### Lines 73-88: `pad_input` implementation / `pad_input` 实现
```python
def pad_input(hidden_states, indices, batch, seqlen):
    """
    Arguments:
        hidden_states: (total_nnz, ...), where total_nnz = number of tokens in selected in attention_mask.
        indices: (total_nnz), the indices that represent the non-masked tokens of the original padded input sequence.
        batch: int, batch size for the padded sequence.
        seqlen: int, maximum sequence length for the padded sequence.
    Return:
        hidden_states: (batch, seqlen, ...)
    """
    dim = hidden_states.shape[1:]
    output = torch.zeros(
        (batch * seqlen), *dim, device=hidden_states.device, dtype=hidden_states.dtype
    )
    output[indices] = hidden_states
    return rearrange(output, "(b s) ... -> b s ...", b=batch)
```
**EN:** This block defines function `pad_input`. Arguments: hidden_states: (total_nnz, ...), where total_nnz = number of tokens in selected in attention_mask. indices: (total_nnz), the indices that represent the non-masked tokens of the original padded input sequence. Key calls include `torch.zeros`, and `rearrange`. Parameters such as `hidden_states`, `indices`, `batch`, and `seqlen` drive the behavior in this section.
**CN:** 该代码块定义了函数 `pad_input`。 它用于处理 pad input 相关逻辑。 关键调用包括 `torch.zeros` 和 `rearrange`。 本段逻辑主要由 `hidden_states`、`indices`、`batch` 和 `seqlen` 等参数驱动。

### Lines 91-92: `VideoMobaAttentionMetadataBuilder` class overview / `VideoMobaAttentionMetadataBuilder` 类概览
```python
class VideoMobaAttentionMetadataBuilder(AttentionMetadataBuilder):
```
**EN:** This block defines class `VideoMobaAttentionMetadataBuilder`. It encapsulates video moba attention metadata builder behavior. It inherits from `AttentionMetadataBuilder`.
**CN:** 该代码块定义了类 `VideoMobaAttentionMetadataBuilder`。 它用于封装 video moba attention metadata builder 相关行为。 它继承自 `AttentionMetadataBuilder`。

### Lines 93-94: `__init__` implementation / `__init__` 实现
```python
    def __init__(self):
        pass
```
**EN:** This block defines method `__init__` on `VideoMobaAttentionMetadataBuilder`. It initializes the instance state.
**CN:** 该代码块定义了 `VideoMobaAttentionMetadataBuilder` 的方法 `__init__`。 它用于初始化实例状态。

### Lines 96-97: `prepare` implementation / `prepare` 实现
```python
    def prepare(self):
        pass
```
**EN:** This block defines method `prepare` on `VideoMobaAttentionMetadataBuilder`. It prepares function.
**CN:** 该代码块定义了 `VideoMobaAttentionMetadataBuilder` 的方法 `prepare`。 它用于准备函数。

### Lines 99-149: `build` implementation / `build` 实现
```python
    def build(  # type: ignore
        self,
        current_timestep: int,
        raw_latent_shape: tuple[int, int, int],
        patch_size: tuple[int, int, int],
        temporal_chunk_size: int,
        temporal_topk: int,
        spatial_chunk_size: tuple[int, int],
        spatial_topk: int,
        st_chunk_size: tuple[int, int, int],
        st_topk: int,
        moba_select_mode: str = "threshold",
        moba_threshold: float = 0.25,
        moba_threshold_type: str = "query_head",
        device: torch.device = None,
        first_full_layer: int = 0,
        first_full_step: int = 12,
        temporal_layer: int = 1,
        spatial_layer: int = 1,
        st_layer: int = 1,
        **kwargs,
    ) -> VideoMobaAttentionMetadata:
        if device is None:
            device = torch.device("cpu")
        assert (
            raw_latent_shape[0] % patch_size[0] == 0
            and raw_latent_shape[1] % patch_size[1] == 0
            and raw_latent_shape[2] % patch_size[2] == 0
        ), f"spatial patch_resolution {raw_latent_shape} should be divisible by patch_size {patch_size}"
        patch_resolution = [
            t // pt for t, pt in zip(raw_latent_shape, patch_size, strict=False)
        ]

        return VideoMobaAttentionMetadata(
            current_timestep=current_timestep,
            temporal_chunk_size=temporal_chunk_size,
            temporal_topk=temporal_topk,
            spatial_chunk_size=spatial_chunk_size,
            spatial_topk=spatial_topk,
            st_chunk_size=st_chunk_size,
            st_topk=st_topk,
            moba_select_mode=moba_select_mode,
            moba_threshold=moba_threshold,
            moba_threshold_type=moba_threshold_type,
            patch_resolution=patch_resolution,
            first_full_layer=first_full_layer,
            first_full_step=first_full_step,
            temporal_layer=temporal_layer,
            spatial_layer=spatial_layer,
            st_layer=st_layer,
        )
```
**EN:** This block defines method `build` on `VideoMobaAttentionMetadataBuilder`. It builds function. Key calls include `VideoMobaAttentionMetadata`, `torch.device`, and `zip`. The implementation branches on conditions. Parameters such as `current_timestep`, `raw_latent_shape`, `patch_size`, `temporal_chunk_size`, and `temporal_topk` drive the behavior in this section.
**CN:** 该代码块定义了 `VideoMobaAttentionMetadataBuilder` 的方法 `build`。 它用于构建函数。 关键调用包括 `VideoMobaAttentionMetadata`、`torch.device` 和 `zip`。 实现中包含条件分支。 本段逻辑主要由 `current_timestep`、`raw_latent_shape`、`patch_size`、`temporal_chunk_size` 和 `temporal_topk` 等参数驱动。

### Lines 152-153: `VMOBAAttentionImpl` class overview / `VMOBAAttentionImpl` 类概览
```python
class VMOBAAttentionImpl(AttentionImpl):
```
**EN:** This block defines class `VMOBAAttentionImpl`. It encapsulates vmobaattention impl behavior. It inherits from `AttentionImpl`.
**CN:** 该代码块定义了类 `VMOBAAttentionImpl`。 它用于封装 vmobaattention impl 相关行为。 它继承自 `AttentionImpl`。

### Lines 154-167: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        num_heads,
        head_size,
        softmax_scale,
        causal=False,
        num_kv_heads=None,
        prefix="",
        **extra_impl_args,
    ) -> None:
        self.prefix = prefix
        self.layer_idx = self._get_layer_idx(prefix)

        self.pad_input = pad_input
```
**EN:** This block defines method `__init__` on `VMOBAAttentionImpl`. It initializes the instance state. Key calls include `self._get_layer_idx`. Parameters such as `num_heads`, `head_size`, `softmax_scale`, `causal`, and `num_kv_heads` drive the behavior in this section.
**CN:** 该代码块定义了 `VMOBAAttentionImpl` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `self._get_layer_idx`。 本段逻辑主要由 `num_heads`、`head_size`、`softmax_scale`、`causal` 和 `num_kv_heads` 等参数驱动。

### Lines 169-173: `_get_layer_idx` implementation / `_get_layer_idx` 实现
```python
    def _get_layer_idx(self, prefix: str) -> int | None:
        match = re.search(r"blocks\.(\d+)", prefix)
        if not match:
            raise ValueError(f"Invalid prefix: {prefix}")
        return int(match.group(1))
```
**EN:** This block defines method `_get_layer_idx` on `VMOBAAttentionImpl`. It retrieves layer idx. Key calls include `re.search`, `int`, `ValueError`, and `match.group`. The implementation branches on conditions. Parameters such as `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `VMOBAAttentionImpl` 的方法 `_get_layer_idx`。 它用于获取layer idx。 关键调用包括 `re.search`、`int`、`ValueError` 和 `match.group`。 实现中包含条件分支。 本段逻辑主要由 `prefix` 等参数驱动。

### Lines 175-259: `forward` implementation / `forward` 实现
```python
    def forward(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        attn_metadata: AttentionMetadata,
    ) -> torch.Tensor:
        """
        query: [B, L, H, D]
        key:   [B, L, H, D]
        value: [B, L, H, D]
        attn_metadata: AttentionMetadata
        """
        batch_size, sequence_length, num_heads, head_dim = query.shape

        # select chunk type according to layer idx:
        loop_layer_num = (
            attn_metadata.temporal_layer
            + attn_metadata.spatial_layer
            + attn_metadata.st_layer
        )
        moba_layer = self.layer_idx - attn_metadata.first_full_layer
        if moba_layer % loop_layer_num < attn_metadata.temporal_layer:
            moba_chunk_size = attn_metadata.temporal_chunk_size
            moba_topk = attn_metadata.temporal_topk
        elif (
            moba_layer % loop_layer_num
            < attn_metadata.temporal_layer + attn_metadata.spatial_layer
        ):
            moba_chunk_size = attn_metadata.spatial_chunk_size
            moba_topk = attn_metadata.spatial_topk
        elif (
            moba_layer % loop_layer_num
            < attn_metadata.temporal_layer
            + attn_metadata.spatial_layer
            + attn_metadata.st_layer
        ):
            moba_chunk_size = attn_metadata.st_chunk_size
            moba_topk = attn_metadata.st_topk

        query, chunk_size = process_moba_input(
            query, attn_metadata.patch_resolution, moba_chunk_size
        )
        key, chunk_size = process_moba_input(
            key, attn_metadata.patch_resolution, moba_chunk_size
        )
        value, chunk_size = process_moba_input(
            value, attn_metadata.patch_resolution, moba_chunk_size
        )
        max_seqlen = query.shape[1]
        indices_q = torch.arange(
            0, query.shape[0] * query.shape[1], device=query.device
        )
        cu_seqlens = torch.arange(
            0,
            query.shape[0] * query.shape[1] + 1,
            query.shape[1],
            dtype=torch.int32,
            device=query.device,
        )
        query = rearrange(query, "b s ... -> (b s) ...")
        key = rearrange(key, "b s ... -> (b s) ...")
        value = rearrange(value, "b s ... -> (b s) ...")

        # current_timestep=attn_metadata.current_timestep
        hidden_states = moba_attn_varlen(
            query,
            key,
            value,
            cu_seqlens=cu_seqlens,
            max_seqlen=max_seqlen,
            moba_chunk_size=chunk_size,
            moba_topk=moba_topk,
            select_mode=attn_metadata.moba_select_mode,
            simsum_threshold=attn_metadata.moba_threshold,
            threshold_type=attn_metadata.moba_threshold_type,
        )
        hidden_states = self.pad_input(
            hidden_states, indices_q, batch_size, sequence_length
        )
        hidden_states = process_moba_output(
            hidden_states, attn_metadata.patch_resolution, moba_chunk_size
        )

        return hidden_states
```
**EN:** This block defines method `forward` on `VMOBAAttentionImpl`. query: [B, L, H, D] key: [B, L, H, D] value: [B, L, H, D] attn_metadata: AttentionMetadata Key calls include `process_moba_input`, `torch.arange`, `rearrange`, `moba_attn_varlen`, and `self.pad_input`. The implementation branches on conditions. Parameters such as `query`, `key`, `value`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `VMOBAAttentionImpl` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `process_moba_input`、`torch.arange`、`rearrange`、`moba_attn_varlen` 和 `self.pad_input`。 实现中包含条件分支。 本段逻辑主要由 `query`、`key`、`value` 和 `attn_metadata` 等参数驱动。

## Key Concepts / 关键概念
- `VMOBAAttentionBackend`: Primary class that encapsulates vmobaattention backend behavior. / 核心类，用于封装 vmobaattention backend 相关行为。
- `VideoMobaAttentionMetadata`: Primary class that encapsulates video moba attention metadata behavior. / 核心类，用于封装 video moba attention metadata 相关行为。
- `pad_input`: Arguments: hidden_states: (total_nnz, ...), where total_nnz = number of tokens in selected in attention_mask. / 顶层函数，用于处理 pad input 相关逻辑。
- `VideoMobaAttentionMetadataBuilder`: Primary class that encapsulates video moba attention metadata builder behavior. / 核心类，用于封装 video moba attention metadata builder 相关行为。
- `VMOBAAttentionImpl`: Primary class that encapsulates vmobaattention impl behavior. / 核心类，用于封装 vmobaattention impl 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `re`, `dataclasses`
- **Third-party / 第三方依赖**: `torch`, `einops`, `kernel.attn.vmoba_attn.vmoba`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 259
