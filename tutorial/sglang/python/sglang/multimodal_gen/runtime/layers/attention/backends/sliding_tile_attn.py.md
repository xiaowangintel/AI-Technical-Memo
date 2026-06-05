# sliding_tile_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/attention/backends/sliding_tile_attn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `RangeDict`, `SlidingTileAttentionBackend`, and `SlidingTileAttentionMetadata`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `RangeDict`、`SlidingTileAttentionBackend` 和 `SlidingTileAttentionMetadata` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-25: module setup and imports / 模块初始化与导入
```python
import json
from dataclasses import dataclass
from typing import Any

import torch
from einops import rearrange

from sglang.multimodal_gen.runtime.distributed import get_sp_group
from sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend import (
    AttentionBackend,
    AttentionImpl,
    AttentionMetadata,
    AttentionMetadataBuilder,
)
from sglang.multimodal_gen.runtime.managers.forward_context import (
    ForwardContext,
    get_forward_context,
)
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
from sglang.multimodal_gen.runtime.server_args import get_global_server_args
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.utils import dict_to_3d_list
```
**EN:** This block establishes the module context and imports `json`, `dataclasses`, `typing`, `torch`, `einops`, and `sglang.multimodal_gen.runtime.distributed`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `json`、`dataclasses`、`typing`、`torch`、`einops` 和 `sglang.multimodal_gen.runtime.distributed`。这些依赖为后续实现提供所需符号。

### Lines 27-34: supporting statements / 辅助语句
```python
try:
    from st_attn import sliding_tile_attention

    st_attn_backend_available = True
except Exception:
    st_attn_backend_available = False

logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 37-38: `RangeDict` class overview / `RangeDict` 类概览
```python
class RangeDict(dict):
```
**EN:** This block defines class `RangeDict`. It encapsulates range dict behavior. It inherits from `dict`.
**CN:** 该代码块定义了类 `RangeDict`。 它用于封装 range dict 相关行为。 它继承自 `dict`。

### Lines 39-47: `__getitem__` implementation / `__getitem__` 实现
```python
    def __getitem__(self, item: int) -> str:
        for key in self.keys():
            if isinstance(key, tuple):
                low, high = key
                if low <= item <= high:
                    return str(super().__getitem__(key))
            elif key == item:
                return str(super().__getitem__(key))
        raise KeyError(f"seq_len {item} not supported for STA")
```
**EN:** This block defines method `__getitem__` on `RangeDict`. It handles getitem logic. Key calls include `self.keys`, `KeyError`, `isinstance`, `str`, and `super.__getitem__`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `item` drive the behavior in this section.
**CN:** 该代码块定义了 `RangeDict` 的方法 `__getitem__`。 它用于处理 getitem 相关逻辑。 关键调用包括 `self.keys`、`KeyError`、`isinstance`、`str` 和 `super.__getitem__`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `item` 等参数驱动。

### Lines 50-50: `SlidingTileAttentionBackend` class overview / `SlidingTileAttentionBackend` 类概览
```python
class SlidingTileAttentionBackend(AttentionBackend):
```
**EN:** This block defines class `SlidingTileAttentionBackend`. It encapsulates sliding tile attention backend behavior. It inherits from `AttentionBackend`.
**CN:** 该代码块定义了类 `SlidingTileAttentionBackend`。 它用于封装 sliding tile attention backend 相关行为。 它继承自 `AttentionBackend`。

### Lines 51-51: supporting statements / 辅助语句
```python
    accept_output_buffer: bool = True
```
**EN:** This block gathers supporting statements inside `SlidingTileAttentionBackend`. It updates names such as `accept_output_buffer`.
**CN:** 该代码块汇集了位于 `SlidingTileAttentionBackend` 内部的辅助语句。 它会更新 `accept_output_buffer` 等名称。

### Lines 53-56: `get_supported_head_sizes` implementation / `get_supported_head_sizes` 实现
```python
    @staticmethod
    def get_supported_head_sizes() -> list[int]:
        # TODO(will-refactor): check this
        return [32, 64, 96, 128, 160, 192, 224, 256]
```
**EN:** This block defines method `get_supported_head_sizes` on `SlidingTileAttentionBackend`. It retrieves supported head sizes.
**CN:** 该代码块定义了 `SlidingTileAttentionBackend` 的方法 `get_supported_head_sizes`。 它用于获取supported head sizes。

### Lines 58-60: `get_enum` implementation / `get_enum` 实现
```python
    @staticmethod
    def get_enum() -> AttentionBackendEnum:
        return AttentionBackendEnum.SLIDING_TILE_ATTN
```
**EN:** This block defines method `get_enum` on `SlidingTileAttentionBackend`. It retrieves enum.
**CN:** 该代码块定义了 `SlidingTileAttentionBackend` 的方法 `get_enum`。 它用于获取enum。

### Lines 62-64: `get_impl_cls` implementation / `get_impl_cls` 实现
```python
    @staticmethod
    def get_impl_cls() -> type["SlidingTileAttentionImpl"]:
        return SlidingTileAttentionImpl
```
**EN:** This block defines method `get_impl_cls` on `SlidingTileAttentionBackend`. It retrieves impl cls.
**CN:** 该代码块定义了 `SlidingTileAttentionBackend` 的方法 `get_impl_cls`。 它用于获取impl cls。

### Lines 66-68: `get_metadata_cls` implementation / `get_metadata_cls` 实现
```python
    @staticmethod
    def get_metadata_cls() -> type["SlidingTileAttentionMetadata"]:
        return SlidingTileAttentionMetadata
```
**EN:** This block defines method `get_metadata_cls` on `SlidingTileAttentionBackend`. It retrieves metadata cls.
**CN:** 该代码块定义了 `SlidingTileAttentionBackend` 的方法 `get_metadata_cls`。 它用于获取metadata cls。

### Lines 70-72: `get_builder_cls` implementation / `get_builder_cls` 实现
```python
    @staticmethod
    def get_builder_cls() -> type["SlidingTileAttentionMetadataBuilder"]:
        return SlidingTileAttentionMetadataBuilder
```
**EN:** This block defines method `get_builder_cls` on `SlidingTileAttentionBackend`. It retrieves builder cls.
**CN:** 该代码块定义了 `SlidingTileAttentionBackend` 的方法 `get_builder_cls`。 它用于获取builder cls。

### Lines 76-76: `SlidingTileAttentionMetadata` class overview / `SlidingTileAttentionMetadata` 类概览
```python
class SlidingTileAttentionMetadata(AttentionMetadata):
```
**EN:** This block defines class `SlidingTileAttentionMetadata`. It encapsulates sliding tile attention metadata behavior. It inherits from `AttentionMetadata`.
**CN:** 该代码块定义了类 `SlidingTileAttentionMetadata`。 它用于封装 sliding tile attention metadata 相关行为。 它继承自 `AttentionMetadata`。

### Lines 77-80: supporting statements / 辅助语句
```python
    current_timestep: int
    STA_param: list[
        list[Any]
    ]  # each timestep with one metadata, shape [num_layers, num_heads]
```
**EN:** This block gathers supporting statements inside `SlidingTileAttentionMetadata`. It updates names such as `current_timestep`, and `STA_param`.
**CN:** 该代码块汇集了位于 `SlidingTileAttentionMetadata` 内部的辅助语句。 它会更新 `current_timestep` 和 `STA_param` 等名称。

### Lines 83-84: `SlidingTileAttentionMetadataBuilder` class overview / `SlidingTileAttentionMetadataBuilder` 类概览
```python
class SlidingTileAttentionMetadataBuilder(AttentionMetadataBuilder):
```
**EN:** This block defines class `SlidingTileAttentionMetadataBuilder`. It encapsulates sliding tile attention metadata builder behavior. It inherits from `AttentionMetadataBuilder`.
**CN:** 该代码块定义了类 `SlidingTileAttentionMetadataBuilder`。 它用于封装 sliding tile attention metadata builder 相关行为。 它继承自 `AttentionMetadataBuilder`。

### Lines 85-86: `__init__` implementation / `__init__` 实现
```python
    def __init__(self):
        pass
```
**EN:** This block defines method `__init__` on `SlidingTileAttentionMetadataBuilder`. It initializes the instance state.
**CN:** 该代码块定义了 `SlidingTileAttentionMetadataBuilder` 的方法 `__init__`。 它用于初始化实例状态。

### Lines 88-89: `prepare` implementation / `prepare` 实现
```python
    def prepare(self):
        pass
```
**EN:** This block defines method `prepare` on `SlidingTileAttentionMetadataBuilder`. It prepares function.
**CN:** 该代码块定义了 `SlidingTileAttentionMetadataBuilder` 的方法 `prepare`。 它用于准备函数。

### Lines 91-104: `build` implementation / `build` 实现
```python
    def build(  # type: ignore
        self,
        STA_param: list[list[Any]],
        current_timestep: int,
        **kwargs: dict[str, Any],
    ) -> SlidingTileAttentionMetadata:
        param = STA_param
        if param is None:
            return SlidingTileAttentionMetadata(
                current_timestep=current_timestep, STA_param=[]
            )
        return SlidingTileAttentionMetadata(
            current_timestep=current_timestep, STA_param=param[current_timestep]
        )
```
**EN:** This block defines method `build` on `SlidingTileAttentionMetadataBuilder`. It builds function. Key calls include `SlidingTileAttentionMetadata`. The implementation branches on conditions. Parameters such as `STA_param`, and `current_timestep` drive the behavior in this section.
**CN:** 该代码块定义了 `SlidingTileAttentionMetadataBuilder` 的方法 `build`。 它用于构建函数。 关键调用包括 `SlidingTileAttentionMetadata`。 实现中包含条件分支。 本段逻辑主要由 `STA_param` 和 `current_timestep` 等参数驱动。

### Lines 107-108: `SlidingTileAttentionImpl` class overview / `SlidingTileAttentionImpl` 类概览
```python
class SlidingTileAttentionImpl(AttentionImpl):
```
**EN:** This block defines class `SlidingTileAttentionImpl`. It encapsulates sliding tile attention impl behavior. It inherits from `AttentionImpl`.
**CN:** 该代码块定义了类 `SlidingTileAttentionImpl`。 它用于封装 sliding tile attention impl 相关行为。 它继承自 `AttentionImpl`。

### Lines 109-150: `__init__` implementation / `__init__` 实现
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
        if not st_attn_backend_available:
            raise ValueError("st attn not supported")
        # TODO(will-refactor): for now this is the mask strategy, but maybe we should
        # have a more general config for STA?
        mask_strategy_file_path = (
            get_global_server_args().attention_backend_config.mask_strategy_file_path
        )
        if mask_strategy_file_path is None:
            raise ValueError("SGLANG_DIFFUSION_ATTENTION_CONFIG is not set")

        # TODO(kevin): get mask strategy for different STA modes
        with open(mask_strategy_file_path) as f:
            mask_strategy = json.load(f)
        self.mask_strategy = dict_to_3d_list(mask_strategy)

        self.prefix = prefix
        sp_group = get_sp_group()
        self.sp_size = sp_group.world_size
        # STA config
        self.STA_base_tile_size = [6, 8, 8]
        self.dit_seq_shape_mapping = RangeDict(
            {
                (115200, 115456): "30x48x80",
                82944: "36x48x48",
                69120: "18x48x80",
            }
        )
        self.full_window_mapping = {
            "30x48x80": [5, 6, 10],
            "36x48x48": [6, 6, 6],
            "18x48x80": [3, 6, 10],
        }
```
**EN:** This block defines method `__init__` on `SlidingTileAttentionImpl`. It initializes the instance state. Key calls include `dict_to_3d_list`, `get_sp_group`, `RangeDict`, `ValueError`, and `open`. The implementation branches on conditions, uses context-managed resources. Parameters such as `num_heads`, `head_size`, `causal`, `softmax_scale`, and `num_kv_heads` drive the behavior in this section.
**CN:** 该代码块定义了 `SlidingTileAttentionImpl` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `dict_to_3d_list`、`get_sp_group`、`RangeDict`、`ValueError` 和 `open`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `num_heads`、`head_size`、`causal`、`softmax_scale` 和 `num_kv_heads` 等参数驱动。

### Lines 152-162: `tile` implementation / `tile` 实现
```python
    def tile(self, x: torch.Tensor) -> torch.Tensor:
        return rearrange(
            x,
            "b (n_t ts_t n_h ts_h n_w ts_w) h d -> b (n_t n_h n_w ts_t ts_h ts_w) h d",
            n_t=self.full_window_size[0],
            n_h=self.full_window_size[1],
            n_w=self.full_window_size[2],
            ts_t=self.STA_base_tile_size[0],
            ts_h=self.STA_base_tile_size[1],
            ts_w=self.STA_base_tile_size[2],
        )
```
**EN:** This block defines method `tile` on `SlidingTileAttentionImpl`. It handles tile logic. Key calls include `rearrange`. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `SlidingTileAttentionImpl` 的方法 `tile`。 它用于处理 tile 相关逻辑。 关键调用包括 `rearrange`。 本段逻辑主要由 `x` 等参数驱动。

### Lines 164-175: `untile` implementation / `untile` 实现
```python
    def untile(self, x: torch.Tensor) -> torch.Tensor:
        x = rearrange(
            x,
            "b (n_t n_h n_w ts_t ts_h ts_w) h d -> b (n_t ts_t n_h ts_h n_w ts_w) h d",
            n_t=self.full_window_size[0],
            n_h=self.full_window_size[1],
            n_w=self.full_window_size[2],
            ts_t=self.STA_base_tile_size[0],
            ts_h=self.STA_base_tile_size[1],
            ts_w=self.STA_base_tile_size[2],
        )
        return x
```
**EN:** This block defines method `untile` on `SlidingTileAttentionImpl`. It handles untile logic. Key calls include `rearrange`. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `SlidingTileAttentionImpl` 的方法 `untile`。 它用于处理 untile 相关逻辑。 关键调用包括 `rearrange`。 本段逻辑主要由 `x` 等参数驱动。

### Lines 177-191: `preprocess_qkv` implementation / `preprocess_qkv` 实现
```python
    def preprocess_qkv(
        self,
        qkv: torch.Tensor,
        attn_metadata: AttentionMetadata,
    ) -> torch.Tensor:
        img_sequence_length = qkv.shape[1]
        self.dit_seq_shape_str = self.dit_seq_shape_mapping[img_sequence_length]
        self.full_window_size = self.full_window_mapping[self.dit_seq_shape_str]
        self.dit_seq_shape_int = list(map(int, self.dit_seq_shape_str.split("x")))
        self.img_seq_length = (
            self.dit_seq_shape_int[0]
            * self.dit_seq_shape_int[1]
            * self.dit_seq_shape_int[2]
        )
        return self.tile(qkv)
```
**EN:** This block defines method `preprocess_qkv` on `SlidingTileAttentionImpl`. It handles preprocess qkv logic. Key calls include `list`, `self.tile`, `map`, and `self.dit_seq_shape_str.split`. Parameters such as `qkv`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `SlidingTileAttentionImpl` 的方法 `preprocess_qkv`。 它用于处理 preprocess qkv 相关逻辑。 关键调用包括 `list`、`self.tile`、`map` 和 `self.dit_seq_shape_str.split`。 本段逻辑主要由 `qkv` 和 `attn_metadata` 等参数驱动。

### Lines 193-198: `postprocess_output` implementation / `postprocess_output` 实现
```python
    def postprocess_output(
        self,
        output: torch.Tensor,
        attn_metadata: SlidingTileAttentionMetadata,
    ) -> torch.Tensor:
        return self.untile(output)
```
**EN:** This block defines method `postprocess_output` on `SlidingTileAttentionImpl`. It handles postprocess output logic. Key calls include `self.untile`. Parameters such as `output`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `SlidingTileAttentionImpl` 的方法 `postprocess_output`。 它用于处理 postprocess output 相关逻辑。 关键调用包括 `self.untile`。 本段逻辑主要由 `output` 和 `attn_metadata` 等参数驱动。

### Lines 200-316: `forward` implementation / `forward` 实现
```python
    def forward(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        attn_metadata: SlidingTileAttentionMetadata,
    ) -> torch.Tensor:
        if self.mask_strategy is None:
            raise ValueError("mask_strategy cannot be None for SlidingTileAttention")
        if self.mask_strategy[0] is None:
            raise ValueError("mask_strategy[0] cannot be None for SlidingTileAttention")

        timestep = attn_metadata.current_timestep
        forward_context: ForwardContext = get_forward_context()
        forward_batch = forward_context.forward_batch
        if forward_batch is None:
            raise ValueError("forward_batch cannot be None")
        # pattern:'.double_blocks.0.attn.impl' or '.single_blocks.0.attn.impl'
        layer_idx = int(self.prefix.split(".")[-3])
        if attn_metadata.STA_param is None or len(attn_metadata.STA_param) <= layer_idx:
            raise ValueError("Invalid STA_param")
        STA_param = attn_metadata.STA_param[layer_idx]

        text_length = q.shape[1] - self.img_seq_length
        has_text = text_length > 0

        query = q.transpose(1, 2).contiguous()
        key = k.transpose(1, 2).contiguous()
        value = v.transpose(1, 2).contiguous()

        head_num = query.size(1)
        sp_group = get_sp_group()
        current_rank = sp_group.rank_in_group
        start_head = current_rank * head_num

        # searching or tuning mode
        if len(STA_param) < head_num * sp_group.world_size:
            sparse_attn_hidden_states_all = []
            full_mask_window = STA_param[-1]
            for window_size in STA_param[:-1]:
                sparse_hidden_states = sliding_tile_attention(
                    query,
                    key,
                    value,
                    [window_size] * head_num,
                    text_length,
                    has_text,
                    self.dit_seq_shape_str,
                ).transpose(1, 2)
                sparse_attn_hidden_states_all.append(sparse_hidden_states)

            hidden_states = sliding_tile_attention(
                query,
                key,
                value,
                [full_mask_window] * head_num,
                text_length,
                has_text,
                self.dit_seq_shape_str,
            ).transpose(1, 2)

            attn_L2_loss = []
            attn_L1_loss = []
            # average loss across all heads
            for sparse_attn_hidden_states in sparse_attn_hidden_states_all:
                # L2 loss
                attn_L2_loss_ = (
                    torch.mean(
                        (sparse_attn_hidden_states.float() - hidden_states.float())
                        ** 2,
                        dim=[0, 1, 3],
                    )
                    .cpu()
                    .numpy()
                )
                attn_L2_loss_ = [round(float(x), 6) for x in attn_L2_loss_]
                attn_L2_loss.append(attn_L2_loss_)
                # L1 loss
                attn_L1_loss_ = (
                    torch.mean(
                        torch.abs(
                            sparse_attn_hidden_states.float() - hidden_states.float()
                        ),
                        dim=[0, 1, 3],
                    )
                    .cpu()
                    .numpy()
                )
                attn_L1_loss_ = [round(float(x), 6) for x in attn_L1_loss_]
                attn_L1_loss.append(attn_L1_loss_)

            layer_loss_save = {"L2_loss": attn_L2_loss, "L1_loss": attn_L1_loss}

            if forward_batch.is_cfg_negative:
                if forward_batch.mask_search_final_result_neg is not None:
                    forward_batch.mask_search_final_result_neg[timestep].append(
                        layer_loss_save
                    )
            else:
                if forward_batch.mask_search_final_result_pos is not None:
                    forward_batch.mask_search_final_result_pos[timestep].append(
                        layer_loss_save
                    )
        else:
            windows = [STA_param[head_idx + start_head] for head_idx in range(head_num)]

            hidden_states = sliding_tile_attention(
                query,
                key,
                value,
                windows,
                text_length,
                has_text,
                self.dit_seq_shape_str,
            ).transpose(1, 2)

        return hidden_states
```
**EN:** This block defines method `forward` on `SlidingTileAttentionImpl`. It executes function. Key calls include `get_forward_context`, `int`, `q.transpose.contiguous`, `k.transpose.contiguous`, and `v.transpose.contiguous`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `q`, `k`, `v`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `SlidingTileAttentionImpl` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `get_forward_context`、`int`、`q.transpose.contiguous`、`k.transpose.contiguous` 和 `v.transpose.contiguous`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `q`、`k`、`v` 和 `attn_metadata` 等参数驱动。

## Key Concepts / 关键概念
- `RangeDict`: Primary class that encapsulates range dict behavior. / 核心类，用于封装 range dict 相关行为。
- `SlidingTileAttentionBackend`: Primary class that encapsulates sliding tile attention backend behavior. / 核心类，用于封装 sliding tile attention backend 相关行为。
- `SlidingTileAttentionMetadata`: Primary class that encapsulates sliding tile attention metadata behavior. / 核心类，用于封装 sliding tile attention metadata 相关行为。
- `SlidingTileAttentionMetadataBuilder`: Primary class that encapsulates sliding tile attention metadata builder behavior. / 核心类，用于封装 sliding tile attention metadata builder 相关行为。
- `SlidingTileAttentionImpl`: Primary class that encapsulates sliding tile attention impl behavior. / 核心类，用于封装 sliding tile attention impl 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`, `einops`, `st_attn`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`, `sglang.multimodal_gen.runtime.managers.forward_context`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.utils`

- **Total lines / 总行数**: 316
