# factory.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/rotary_embedding/factory.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `get_rope`, and `get_rotary_pos_embed`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: get_rope / get_rotary_pos_embed factory functions and module-level caches. / 该文件属于运行时算子层。它围绕 `get_rope` 和 `get_rotary_pos_embed` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: module setup and imports / 模块初始化与导入
```python
"""get_rope / get_rotary_pos_embed factory functions and module-level caches."""

from collections import OrderedDict
from typing import Any

import torch

from .base import LinearScalingRotaryEmbedding, RotaryEmbedding
from .mrope import NDRotaryEmbedding, _to_tuple
```
**EN:** This block establishes the module context and imports `collections`, `typing`, `torch`, `.base`, and `.mrope`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `collections`、`typing`、`torch`、`.base` 和 `.mrope`。这些依赖为后续实现提供所需符号。

### Lines 11-13: supporting statements / 辅助语句
```python
_ROPE_DICT: dict[tuple, RotaryEmbedding] = {}
_ND_ROPE_CACHE: "OrderedDict[tuple, NDRotaryEmbedding]" = OrderedDict()
_ROPE_3D_CACHE: "OrderedDict[tuple, tuple[torch.Tensor, torch.Tensor]]" = OrderedDict()
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `_ROPE_DICT`, `_ND_ROPE_CACHE`, and `_ROPE_3D_CACHE`. The code collaborates with `OrderedDict`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `_ROPE_DICT`、`_ND_ROPE_CACHE` 和 `_ROPE_3D_CACHE` 等名称。 代码会与 `OrderedDict` 协同工作。

### Lines 16-82: `get_rope` implementation / `get_rope` 实现
```python
def get_rope(
    head_size: int,
    rotary_dim: int,
    max_position: int,
    base: int | float,
    is_neox_style: bool = True,
    rope_scaling: dict[str, Any] | None = None,
    dtype: torch.dtype | None = None,
    partial_rotary_factor: float = 1.0,
) -> RotaryEmbedding:
    if dtype is None:
        dtype = torch.get_default_dtype()
    if rope_scaling is not None:
        # Transforms every value that is a list into a tuple for caching calls
        rope_scaling_tuple = {
            k: tuple(v) if isinstance(v, list) else v for k, v in rope_scaling.items()
        }
        rope_scaling_args = tuple(rope_scaling_tuple.items())
    else:
        rope_scaling_args = None
    if partial_rotary_factor < 1.0:
        rotary_dim = int(rotary_dim * partial_rotary_factor)
    max_position_embeddings = max_position
    rope_type = None
    if rope_scaling is not None:
        rope_type = rope_scaling.get("rope_type", rope_scaling.get("type", None))
        if rope_type in (None, "default"):
            rope_scaling = None
        elif rope_type == "linear":
            factor = float(rope_scaling.get("factor", 1.0))
            original_max = rope_scaling.get("original_max_position_embeddings", None)
            if original_max is not None:
                max_position_embeddings = max(
                    max_position_embeddings, int(float(original_max) * factor)
                )
    key = (
        head_size,
        rotary_dim,
        max_position_embeddings,
        base,
        is_neox_style,
        rope_scaling_args,
        dtype,
    )
    if key in _ROPE_DICT:
        return _ROPE_DICT[key]

    if rope_scaling is None:
        rotary_emb = RotaryEmbedding(
            head_size, rotary_dim, max_position_embeddings, base, is_neox_style, dtype
        )
    else:
        if rope_type == "linear":
            factor = float(rope_scaling.get("factor", 1.0))
            rotary_emb = LinearScalingRotaryEmbedding(
                head_size=head_size,
                rotary_dim=rotary_dim,
                max_position_embeddings=max_position_embeddings,
                base=base,
                is_neox_style=is_neox_style,
                dtype=dtype,
                scaling_factor=factor,
            )
        else:
            raise ValueError(f"Unknown RoPE scaling {rope_scaling}")
    _ROPE_DICT[key] = rotary_emb
    return rotary_emb
```
**EN:** This block defines function `get_rope`. It retrieves rope. Key calls include `torch.get_default_dtype`, `tuple`, `int`, `rope_scaling.get`, and `RotaryEmbedding`. The implementation branches on conditions. Parameters such as `head_size`, `rotary_dim`, `max_position`, `base`, and `is_neox_style` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_rope`。 它用于获取rope。 关键调用包括 `torch.get_default_dtype`、`tuple`、`int`、`rope_scaling.get` 和 `RotaryEmbedding`。 实现中包含条件分支。 本段逻辑主要由 `head_size`、`rotary_dim`、`max_position`、`base` 和 `is_neox_style` 等参数驱动。

### Lines 85-171: `get_rotary_pos_embed` implementation / `get_rotary_pos_embed` 实现
```python
def get_rotary_pos_embed(
    rope_sizes,
    hidden_size,
    heads_num,
    rope_dim_list,
    rope_theta,
    theta_rescale_factor=1.0,
    interpolation_factor=1.0,
    shard_dim: int = 0,
    dtype: torch.dtype = torch.float32,
    start_frame: int = 0,
    device: torch.device | str | None = None,
) -> tuple[torch.Tensor, torch.Tensor]:
    """
    Generate rotary positional embeddings for the given sizes.

    Args:
        rope_sizes: Tuple of dimensions (t, h, w)
        hidden_size: Hidden dimension size
        heads_num: Number of attention heads
        rope_dim_list: List of dimensions for each axis, or None
        rope_theta: Base for frequency calculations
        theta_rescale_factor: Rescale factor for theta. Defaults to 1.0
        interpolation_factor: Factor to scale positions. Defaults to 1.0
        shard_dim: Which dimension to shard for sequence parallelism. Defaults to 0.

    Returns:
        Tuple of (cos, sin) tensors for rotary embeddings
    """

    target_ndim = 3
    head_dim = hidden_size // heads_num

    if rope_dim_list is None:
        rope_dim_list = [head_dim // target_ndim for _ in range(target_ndim)]

    assert (
        sum(rope_dim_list) == head_dim
    ), "sum(rope_dim_list) should equal to head_dim of attention layer"

    # Get SP info - now handled within NDRotaryEmbedding
    # sp_group = get_sp_group()
    # sp_rank = sp_group.rank_in_group
    # sp_world_size = sp_group.world_size

    # Simple LRU cache keyed by parameters
    global _ND_ROPE_CACHE
    key = (
        tuple(rope_dim_list),
        float(rope_theta),
        (
            tuple(theta_rescale_factor)
            if isinstance(theta_rescale_factor, list)
            else float(theta_rescale_factor)
        ),
        (
            tuple(interpolation_factor)
            if isinstance(interpolation_factor, list)
            else float(interpolation_factor)
        ),
        dtype,
    )

    cache_hit = key in _ND_ROPE_CACHE
    if cache_hit:
        rope_emb = _ND_ROPE_CACHE.pop(key)
        _ND_ROPE_CACHE[key] = rope_emb  # move to end (most-recent)
    else:
        rope_emb = NDRotaryEmbedding(
            rope_dim_list=rope_dim_list,
            rope_theta=rope_theta,
            theta_rescale_factor=theta_rescale_factor,
            interpolation_factor=interpolation_factor,
            dtype=dtype,
        )
        _ND_ROPE_CACHE[key] = rope_emb
        if len(_ND_ROPE_CACHE) > 16:
            # pop least-recently-used
            _ND_ROPE_CACHE.pop(next(iter(_ND_ROPE_CACHE)))

    freqs_cos, freqs_sin = rope_emb.forward_from_grid(
        grid_size=_to_tuple(rope_sizes, dim=3),
        shard_dim=shard_dim,
        start_frame=start_frame,
        device=device,
    )
    return freqs_cos, freqs_sin
```
**EN:** This block defines function `get_rotary_pos_embed`. Generate rotary positional embeddings for the given sizes. Args: rope_sizes: Tuple of dimensions (t, h, w) hidden_size: Hidden dimension size heads_num: Number of attention heads rope_dim_list: List of dimensions for each axis, or None rope_theta: Base for frequency calculations theta_rescale_factor: Rescale factor for theta. Key calls include `rope_emb.forward_from_grid`, `sum`, `tuple`, `float`, and `_ND_ROPE_CACHE.pop`. The implementation branches on conditions. Parameters such as `rope_sizes`, `hidden_size`, `heads_num`, `rope_dim_list`, and `rope_theta` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_rotary_pos_embed`。 它用于获取rotary pos embed。 关键调用包括 `rope_emb.forward_from_grid`、`sum`、`tuple`、`float` 和 `_ND_ROPE_CACHE.pop`。 实现中包含条件分支。 本段逻辑主要由 `rope_sizes`、`hidden_size`、`heads_num`、`rope_dim_list` 和 `rope_theta` 等参数驱动。

## Key Concepts / 关键概念
- `get_rope`: Top-level function that retrieves rope. / 顶层函数，用于获取rope。
- `get_rotary_pos_embed`: Generate rotary positional embeddings for the given sizes. / 顶层函数，用于获取rotary pos embed。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `.base`, `.mrope`

- **Total lines / 总行数**: 171
