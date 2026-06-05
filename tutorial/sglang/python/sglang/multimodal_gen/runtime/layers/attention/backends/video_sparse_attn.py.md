# video_sparse_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/attention/backends/video_sparse_attn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `VideoSparseAttentionBackend`, `VideoSparseAttentionMetadata`, and `VideoSparseAttentionMetadataBuilder`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `VideoSparseAttentionBackend`、`VideoSparseAttentionMetadata` 和 `VideoSparseAttentionMetadataBuilder` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-8: module setup and imports / 模块初始化与导入
```python
import functools
import math
from dataclasses import dataclass

import torch
```
**EN:** This block establishes the module context and imports `functools`, `math`, `dataclasses`, and `torch`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `functools`、`math`、`dataclasses` 和 `torch`。这些依赖为后续实现提供所需符号。

### Lines 10-28: supporting statements / 辅助语句
```python
try:
    from vsa import video_sparse_attn
except ImportError:
    video_sparse_attn = None

from typing import Any

from sglang.multimodal_gen.runtime.distributed import get_sp_group
from sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend import (
    AttentionBackend,
    AttentionImpl,
    AttentionMetadata,
    AttentionMetadataBuilder,
)
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
VSA_TILE_SIZE = (4, 4, 4)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, and `VSA_TILE_SIZE`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 和 `VSA_TILE_SIZE` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 31-52: `get_tile_partition_indices` implementation / `get_tile_partition_indices` 实现
```python
@functools.lru_cache(maxsize=10)
def get_tile_partition_indices(
    dit_seq_shape: tuple[int, int, int],
    tile_size: tuple[int, int, int],
    device: torch.device,
) -> torch.LongTensor:
    T, H, W = dit_seq_shape
    ts, hs, ws = tile_size
    indices = torch.arange(T * H * W, device=device, dtype=torch.long).reshape(T, H, W)
    ls = []
    for t in range(math.ceil(T / ts)):
        for h in range(math.ceil(H / hs)):
            for w in range(math.ceil(W / ws)):
                ls.append(
                    indices[
                        t * ts : min(t * ts + ts, T),
                        h * hs : min(h * hs + hs, H),
                        w * ws : min(w * ws + ws, W),
                    ].flatten()
                )
    index = torch.cat(ls, dim=0)
    return index
```
**EN:** This block defines function `get_tile_partition_indices`. It retrieves tile partition indices. Key calls include `functools.lru_cache`, `torch.arange.reshape`, `range`, `torch.cat`, and `math.ceil`. The implementation iterates over collections or steps. Parameters such as `dit_seq_shape`, `tile_size`, and `device` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_tile_partition_indices`。 它用于获取tile partition indices。 关键调用包括 `functools.lru_cache`、`torch.arange.reshape`、`range`、`torch.cat` 和 `math.ceil`。 实现中会遍历集合或步骤。 本段逻辑主要由 `dit_seq_shape`、`tile_size` 和 `device` 等参数驱动。

### Lines 55-61: `get_reverse_tile_partition_indices` implementation / `get_reverse_tile_partition_indices` 实现
```python
@functools.lru_cache(maxsize=10)
def get_reverse_tile_partition_indices(
    dit_seq_shape: tuple[int, int, int],
    tile_size: tuple[int, int, int],
    device: torch.device,
) -> torch.LongTensor:
    return torch.argsort(get_tile_partition_indices(dit_seq_shape, tile_size, device))
```
**EN:** This block defines function `get_reverse_tile_partition_indices`. It retrieves reverse tile partition indices. Key calls include `functools.lru_cache`, `torch.argsort`, and `get_tile_partition_indices`. Parameters such as `dit_seq_shape`, `tile_size`, and `device` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_reverse_tile_partition_indices`。 它用于获取reverse tile partition indices。 关键调用包括 `functools.lru_cache`、`torch.argsort` 和 `get_tile_partition_indices`。 本段逻辑主要由 `dit_seq_shape`、`tile_size` 和 `device` 等参数驱动。

### Lines 64-105: `construct_variable_block_sizes` implementation / `construct_variable_block_sizes` 实现
```python
@functools.lru_cache(maxsize=10)
def construct_variable_block_sizes(
    dit_seq_shape: tuple[int, int, int],
    num_tiles: tuple[int, int, int],
    device: torch.device,
) -> torch.LongTensor:
    """
    Compute the number of valid (non‑padded) tokens inside every
    (ts_t × ts_h × ts_w) tile after padding ‑‑ flattened in the order
    (t‑tile, h‑tile, w‑tile) that `rearrange` uses.

    Returns
    -------
    torch.LongTensor  # shape: [∏ full_window_size]
    """
    # unpack
    t, h, w = dit_seq_shape
    ts_t, ts_h, ts_w = VSA_TILE_SIZE
    n_t, n_h, n_w = num_tiles

    def _sizes(dim_len: int, tile: int, n_tiles: int) -> torch.LongTensor:
        """Vector with the size of each tile along one dimension."""
        sizes = torch.full((n_tiles,), tile, dtype=torch.int, device=device)
        # size of last (possibly partial) tile
        remainder = dim_len - (n_tiles - 1) * tile
        sizes[-1] = remainder if remainder > 0 else tile
        return sizes

    t_sizes = _sizes(t, ts_t, n_t)  # [n_t]
    h_sizes = _sizes(h, ts_h, n_h)  # [n_h]
    w_sizes = _sizes(w, ts_w, n_w)  # [n_w]

    # broadcast‑multiply to get voxels per tile, then flatten
    block_sizes = (
        t_sizes[:, None, None]  # [n_t, 1,   1]
        * h_sizes[None, :, None]  # [1,   n_h, 1]
        * w_sizes[None, None, :]  # [1,   1,   n_w]
    ).reshape(
        -1
    )  # [n_t * n_h * n_w]

    return block_sizes
```
**EN:** This block defines function `construct_variable_block_sizes`. Compute the number of valid (non‑padded) tokens inside every (ts_t × ts_h × ts_w) tile after padding ‑‑ flattened in the order (t‑tile, h‑tile, w‑tile) that `rearrange` uses. Returns ------- torch.LongTensor # shape: [∏ full_window_size] Key calls include `functools.lru_cache`, `_sizes`, `reshape`, and `torch.full`. Parameters such as `dit_seq_shape`, `num_tiles`, and `device` drive the behavior in this section.
**CN:** 该代码块定义了函数 `construct_variable_block_sizes`。 它用于处理 construct variable block sizes 相关逻辑。 关键调用包括 `functools.lru_cache`、`_sizes`、`reshape` 和 `torch.full`。 本段逻辑主要由 `dit_seq_shape`、`num_tiles` 和 `device` 等参数驱动。

### Lines 108-123: `get_non_pad_index` implementation / `get_non_pad_index` 实现
```python
@functools.lru_cache(maxsize=10)
def get_non_pad_index(
    variable_block_sizes: torch.LongTensor,
    max_block_size: int,
):
    n_win = variable_block_sizes.shape[0]
    device = variable_block_sizes.device
    starts_pad = torch.arange(n_win, device=device) * max_block_size
    index_pad = (
        starts_pad[:, None] + torch.arange(max_block_size, device=device)[None, :]
    )
    index_mask = (
        torch.arange(max_block_size, device=device)[None, :]
        < variable_block_sizes[:, None]
    )
    return index_pad[index_mask]
```
**EN:** This block defines function `get_non_pad_index`. It retrieves non pad index. Key calls include `functools.lru_cache`, and `torch.arange`. Parameters such as `variable_block_sizes`, and `max_block_size` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_non_pad_index`。 它用于获取non pad index。 关键调用包括 `functools.lru_cache` 和 `torch.arange`。 本段逻辑主要由 `variable_block_sizes` 和 `max_block_size` 等参数驱动。

### Lines 126-127: `VideoSparseAttentionBackend` class overview / `VideoSparseAttentionBackend` 类概览
```python
class VideoSparseAttentionBackend(AttentionBackend):
```
**EN:** This block defines class `VideoSparseAttentionBackend`. It encapsulates video sparse attention backend behavior. It inherits from `AttentionBackend`.
**CN:** 该代码块定义了类 `VideoSparseAttentionBackend`。 它用于封装 video sparse attention backend 相关行为。 它继承自 `AttentionBackend`。

### Lines 128-128: supporting statements / 辅助语句
```python
    accept_output_buffer: bool = True
```
**EN:** This block gathers supporting statements inside `VideoSparseAttentionBackend`. It updates names such as `accept_output_buffer`.
**CN:** 该代码块汇集了位于 `VideoSparseAttentionBackend` 内部的辅助语句。 它会更新 `accept_output_buffer` 等名称。

### Lines 130-132: `get_supported_head_sizes` implementation / `get_supported_head_sizes` 实现
```python
    @staticmethod
    def get_supported_head_sizes() -> list[int]:
        return [64, 128]
```
**EN:** This block defines method `get_supported_head_sizes` on `VideoSparseAttentionBackend`. It retrieves supported head sizes.
**CN:** 该代码块定义了 `VideoSparseAttentionBackend` 的方法 `get_supported_head_sizes`。 它用于获取supported head sizes。

### Lines 134-136: `get_enum` implementation / `get_enum` 实现
```python
    @staticmethod
    def get_enum() -> AttentionBackendEnum:
        return AttentionBackendEnum.VIDEO_SPARSE_ATTN
```
**EN:** This block defines method `get_enum` on `VideoSparseAttentionBackend`. It retrieves enum.
**CN:** 该代码块定义了 `VideoSparseAttentionBackend` 的方法 `get_enum`。 它用于获取enum。

### Lines 138-140: `get_impl_cls` implementation / `get_impl_cls` 实现
```python
    @staticmethod
    def get_impl_cls() -> type["VideoSparseAttentionImpl"]:
        return VideoSparseAttentionImpl
```
**EN:** This block defines method `get_impl_cls` on `VideoSparseAttentionBackend`. It retrieves impl cls.
**CN:** 该代码块定义了 `VideoSparseAttentionBackend` 的方法 `get_impl_cls`。 它用于获取impl cls。

### Lines 142-144: `get_metadata_cls` implementation / `get_metadata_cls` 实现
```python
    @staticmethod
    def get_metadata_cls() -> type["VideoSparseAttentionMetadata"]:
        return VideoSparseAttentionMetadata
```
**EN:** This block defines method `get_metadata_cls` on `VideoSparseAttentionBackend`. It retrieves metadata cls.
**CN:** 该代码块定义了 `VideoSparseAttentionBackend` 的方法 `get_metadata_cls`。 它用于获取metadata cls。

### Lines 146-148: `get_builder_cls` implementation / `get_builder_cls` 实现
```python
    @staticmethod
    def get_builder_cls() -> type["VideoSparseAttentionMetadataBuilder"]:
        return VideoSparseAttentionMetadataBuilder
```
**EN:** This block defines method `get_builder_cls` on `VideoSparseAttentionBackend`. It retrieves builder cls.
**CN:** 该代码块定义了 `VideoSparseAttentionBackend` 的方法 `get_builder_cls`。 它用于获取builder cls。

### Lines 152-152: `VideoSparseAttentionMetadata` class overview / `VideoSparseAttentionMetadata` 类概览
```python
class VideoSparseAttentionMetadata(AttentionMetadata):
```
**EN:** This block defines class `VideoSparseAttentionMetadata`. It encapsulates video sparse attention metadata behavior. It inherits from `AttentionMetadata`.
**CN:** 该代码块定义了类 `VideoSparseAttentionMetadata`。 它用于封装 video sparse attention metadata 相关行为。 它继承自 `AttentionMetadata`。

### Lines 153-168: supporting statements / 辅助语句
```python
    current_timestep: int
    dit_seq_shape: list[int]
    VSA_sparsity: float
    num_tiles: list[int]
    total_seq_length: int
    tile_partition_indices: torch.LongTensor
    reverse_tile_partition_indices: torch.LongTensor
    variable_block_sizes: torch.LongTensor
    non_pad_index: torch.LongTensor

    # adaption for FastWan2.1-T2V-1.3B-Diffusers
    # Sequence lengths for the forward batch
    # Maximum sequence length for query
    max_seqlen_q: int = 1
    # Maximum sequence length for key
    max_seqlen_k: int = 0
```
**EN:** This block gathers supporting statements inside `VideoSparseAttentionMetadata`. It updates names such as `current_timestep`, `dit_seq_shape`, `VSA_sparsity`, `num_tiles`, `total_seq_length`, and `tile_partition_indices`.
**CN:** 该代码块汇集了位于 `VideoSparseAttentionMetadata` 内部的辅助语句。 它会更新 `current_timestep`、`dit_seq_shape`、`VSA_sparsity`、`num_tiles`、`total_seq_length` 和 `tile_partition_indices` 等名称。

### Lines 171-172: `VideoSparseAttentionMetadataBuilder` class overview / `VideoSparseAttentionMetadataBuilder` 类概览
```python
class VideoSparseAttentionMetadataBuilder(AttentionMetadataBuilder):
```
**EN:** This block defines class `VideoSparseAttentionMetadataBuilder`. It encapsulates video sparse attention metadata builder behavior. It inherits from `AttentionMetadataBuilder`.
**CN:** 该代码块定义了类 `VideoSparseAttentionMetadataBuilder`。 它用于封装 video sparse attention metadata builder 相关行为。 它继承自 `AttentionMetadataBuilder`。

### Lines 173-174: `__init__` implementation / `__init__` 实现
```python
    def __init__(self):
        pass
```
**EN:** This block defines method `__init__` on `VideoSparseAttentionMetadataBuilder`. It initializes the instance state.
**CN:** 该代码块定义了 `VideoSparseAttentionMetadataBuilder` 的方法 `__init__`。 它用于初始化实例状态。

### Lines 176-177: `prepare` implementation / `prepare` 实现
```python
    def prepare(self):
        pass
```
**EN:** This block defines method `prepare` on `VideoSparseAttentionMetadataBuilder`. It prepares function.
**CN:** 该代码块定义了 `VideoSparseAttentionMetadataBuilder` 的方法 `prepare`。 它用于准备函数。

### Lines 179-225: `build` implementation / `build` 实现
```python
    def build(  # type: ignore
        self,
        current_timestep: int,
        raw_latent_shape: tuple[int, int, int],
        patch_size: tuple[int, int, int],
        VSA_sparsity: float,
        device: torch.device,
        **kwargs: dict[str, Any],
    ) -> VideoSparseAttentionMetadata:
        patch_size = patch_size
        dit_seq_shape = (
            raw_latent_shape[0] // patch_size[0],
            raw_latent_shape[1] // patch_size[1],
            raw_latent_shape[2] // patch_size[2],
        )

        num_tiles = (
            math.ceil(dit_seq_shape[0] / VSA_TILE_SIZE[0]),
            math.ceil(dit_seq_shape[1] / VSA_TILE_SIZE[1]),
            math.ceil(dit_seq_shape[2] / VSA_TILE_SIZE[2]),
        )
        total_seq_length = math.prod(dit_seq_shape)

        tile_partition_indices = get_tile_partition_indices(
            dit_seq_shape, VSA_TILE_SIZE, device
        )
        reverse_tile_partition_indices = get_reverse_tile_partition_indices(
            dit_seq_shape, VSA_TILE_SIZE, device
        )
        variable_block_sizes = construct_variable_block_sizes(
            dit_seq_shape, num_tiles, device
        )
        non_pad_index = get_non_pad_index(
            variable_block_sizes, math.prod(VSA_TILE_SIZE)
        )

        return VideoSparseAttentionMetadata(
            current_timestep=current_timestep,
            dit_seq_shape=dit_seq_shape,  # type: ignore
            VSA_sparsity=VSA_sparsity,  # type: ignore
            num_tiles=num_tiles,  # type: ignore
            total_seq_length=total_seq_length,  # type: ignore
            tile_partition_indices=tile_partition_indices,  # type: ignore
            reverse_tile_partition_indices=reverse_tile_partition_indices,
            variable_block_sizes=variable_block_sizes,
            non_pad_index=non_pad_index,
        )
```
**EN:** This block defines method `build` on `VideoSparseAttentionMetadataBuilder`. It builds function. Key calls include `math.prod`, `get_tile_partition_indices`, `get_reverse_tile_partition_indices`, `construct_variable_block_sizes`, and `get_non_pad_index`. Parameters such as `current_timestep`, `raw_latent_shape`, `patch_size`, `VSA_sparsity`, and `device` drive the behavior in this section.
**CN:** 该代码块定义了 `VideoSparseAttentionMetadataBuilder` 的方法 `build`。 它用于构建函数。 关键调用包括 `math.prod`、`get_tile_partition_indices`、`get_reverse_tile_partition_indices`、`construct_variable_block_sizes` 和 `get_non_pad_index`。 本段逻辑主要由 `current_timestep`、`raw_latent_shape`、`patch_size`、`VSA_sparsity` 和 `device` 等参数驱动。

### Lines 228-229: `VideoSparseAttentionImpl` class overview / `VideoSparseAttentionImpl` 类概览
```python
class VideoSparseAttentionImpl(AttentionImpl):
```
**EN:** This block defines class `VideoSparseAttentionImpl`. It encapsulates video sparse attention impl behavior. It inherits from `AttentionImpl`.
**CN:** 该代码块定义了类 `VideoSparseAttentionImpl`。 它用于封装 video sparse attention impl 相关行为。 它继承自 `AttentionImpl`。

### Lines 230-242: `__init__` implementation / `__init__` 实现
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
        self.prefix = prefix
        sp_group = get_sp_group()
        self.sp_size = sp_group.world_size
```
**EN:** This block defines method `__init__` on `VideoSparseAttentionImpl`. It initializes the instance state. Key calls include `get_sp_group`. Parameters such as `num_heads`, `head_size`, `causal`, `softmax_scale`, and `num_kv_heads` drive the behavior in this section.
**CN:** 该代码块定义了 `VideoSparseAttentionImpl` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `get_sp_group`。 本段逻辑主要由 `num_heads`、`head_size`、`causal`、`softmax_scale` 和 `num_kv_heads` 等参数驱动。

### Lines 244-266: `tile` implementation / `tile` 实现
```python
    def tile(
        self,
        x: torch.Tensor,
        num_tiles: list[int],
        tile_partition_indices: torch.LongTensor,
        non_pad_index: torch.LongTensor,
    ) -> torch.Tensor:
        t_padded_size = num_tiles[0] * VSA_TILE_SIZE[0]
        h_padded_size = num_tiles[1] * VSA_TILE_SIZE[1]
        w_padded_size = num_tiles[2] * VSA_TILE_SIZE[2]

        x_padded = torch.zeros(
            (
                x.shape[0],
                t_padded_size * h_padded_size * w_padded_size,
                x.shape[-2],
                x.shape[-1],
            ),
            device=x.device,
            dtype=x.dtype,
        )
        x_padded[:, non_pad_index] = x[:, tile_partition_indices]
        return x_padded
```
**EN:** This block defines method `tile` on `VideoSparseAttentionImpl`. It handles tile logic. Key calls include `torch.zeros`. Parameters such as `x`, `num_tiles`, `tile_partition_indices`, and `non_pad_index` drive the behavior in this section.
**CN:** 该代码块定义了 `VideoSparseAttentionImpl` 的方法 `tile`。 它用于处理 tile 相关逻辑。 关键调用包括 `torch.zeros`。 本段逻辑主要由 `x`、`num_tiles`、`tile_partition_indices` 和 `non_pad_index` 等参数驱动。

### Lines 268-275: `untile` implementation / `untile` 实现
```python
    def untile(
        self,
        x: torch.Tensor,
        reverse_tile_partition_indices: torch.LongTensor,
        non_pad_index: torch.LongTensor,
    ) -> torch.Tensor:
        x = x[:, non_pad_index][:, reverse_tile_partition_indices]
        return x
```
**EN:** This block defines method `untile` on `VideoSparseAttentionImpl`. It handles untile logic. Parameters such as `x`, `reverse_tile_partition_indices`, and `non_pad_index` drive the behavior in this section.
**CN:** 该代码块定义了 `VideoSparseAttentionImpl` 的方法 `untile`。 它用于处理 untile 相关逻辑。 本段逻辑主要由 `x`、`reverse_tile_partition_indices` 和 `non_pad_index` 等参数驱动。

### Lines 277-287: `preprocess_qkv` implementation / `preprocess_qkv` 实现
```python
    def preprocess_qkv(
        self,
        qkv: torch.Tensor,
        attn_metadata: VideoSparseAttentionMetadata,
    ) -> torch.Tensor:
        return self.tile(
            qkv,
            attn_metadata.num_tiles,
            attn_metadata.tile_partition_indices,
            attn_metadata.non_pad_index,
        )
```
**EN:** This block defines method `preprocess_qkv` on `VideoSparseAttentionImpl`. It handles preprocess qkv logic. Key calls include `self.tile`. Parameters such as `qkv`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `VideoSparseAttentionImpl` 的方法 `preprocess_qkv`。 它用于处理 preprocess qkv 相关逻辑。 关键调用包括 `self.tile`。 本段逻辑主要由 `qkv` 和 `attn_metadata` 等参数驱动。

### Lines 289-298: `postprocess_output` implementation / `postprocess_output` 实现
```python
    def postprocess_output(
        self,
        output: torch.Tensor,
        attn_metadata: VideoSparseAttentionMetadata,
    ) -> torch.Tensor:
        return self.untile(
            output,
            attn_metadata.reverse_tile_partition_indices,
            attn_metadata.non_pad_index,
        )
```
**EN:** This block defines method `postprocess_output` on `VideoSparseAttentionImpl`. It handles postprocess output logic. Key calls include `self.untile`. Parameters such as `output`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `VideoSparseAttentionImpl` 的方法 `postprocess_output`。 它用于处理 postprocess output 相关逻辑。 关键调用包括 `self.untile`。 本段逻辑主要由 `output` 和 `attn_metadata` 等参数驱动。

### Lines 300-332: `forward` implementation / `forward` 实现
```python
    def forward(  # type: ignore[override]
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        gate_compress: torch.Tensor,
        attn_metadata: VideoSparseAttentionMetadata,
    ) -> torch.Tensor:
        query = query.transpose(1, 2).contiguous()
        key = key.transpose(1, 2).contiguous()
        value = value.transpose(1, 2).contiguous()
        gate_compress = gate_compress.transpose(1, 2).contiguous()

        VSA_sparsity = attn_metadata.VSA_sparsity

        cur_topk = math.ceil(
            (1 - VSA_sparsity)
            * (attn_metadata.total_seq_length / math.prod(VSA_TILE_SIZE))
        )

        if video_sparse_attn is None:
            raise NotImplementedError("video_sparse_attn is not installed")
        hidden_states = video_sparse_attn(
            query,
            key,
            value,
            variable_block_sizes=attn_metadata.variable_block_sizes,
            topk=cur_topk,
            block_size=VSA_TILE_SIZE,
            compress_attn_weight=gate_compress,
        ).transpose(1, 2)

        return hidden_states
```
**EN:** This block defines method `forward` on `VideoSparseAttentionImpl`. It executes function. Key calls include `query.transpose.contiguous`, `key.transpose.contiguous`, `value.transpose.contiguous`, `gate_compress.transpose.contiguous`, and `math.ceil`. The implementation branches on conditions. Parameters such as `query`, `key`, `value`, `gate_compress`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `VideoSparseAttentionImpl` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `query.transpose.contiguous`、`key.transpose.contiguous`、`value.transpose.contiguous`、`gate_compress.transpose.contiguous` 和 `math.ceil`。 实现中包含条件分支。 本段逻辑主要由 `query`、`key`、`value`、`gate_compress` 和 `attn_metadata` 等参数驱动。

## Key Concepts / 关键概念
- `get_tile_partition_indices`: Top-level function that retrieves tile partition indices. / 顶层函数，用于获取tile partition indices。
- `get_reverse_tile_partition_indices`: Top-level function that retrieves reverse tile partition indices. / 顶层函数，用于获取reverse tile partition indices。
- `construct_variable_block_sizes`: Compute the number of valid (non‑padded) tokens inside every (ts_t × ts_h × ts_w) tile after padding ‑‑ flattened in the order (t‑tile, h‑tile, w‑tile) that `rearrange` uses. / 顶层函数，用于处理 construct variable block sizes 相关逻辑。
- `get_non_pad_index`: Top-level function that retrieves non pad index. / 顶层函数，用于获取non pad index。
- `VideoSparseAttentionBackend`: Primary class that encapsulates video sparse attention backend behavior. / 核心类，用于封装 video sparse attention backend 相关行为。
- `VideoSparseAttentionMetadata`: Primary class that encapsulates video sparse attention metadata behavior. / 核心类，用于封装 video sparse attention metadata 相关行为。
- `VideoSparseAttentionMetadataBuilder`: Primary class that encapsulates video sparse attention metadata builder behavior. / 核心类，用于封装 video sparse attention metadata builder 相关行为。
- `VideoSparseAttentionImpl`: Primary class that encapsulates video sparse attention impl behavior. / 核心类，用于封装 video sparse attention impl 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`, `math`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`, `vsa`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 332
