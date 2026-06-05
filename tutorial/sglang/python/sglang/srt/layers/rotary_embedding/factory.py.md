# factory.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/rotary_embedding/factory.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements rotary position embedding logic with embedding lookup and preprocessing for the SGLang SRT runtime. It exposes symbols such as `_get_rope_param`, `get_rope`, `get_rope_cpu`, and `get_rope_wrapper` and connects them to backend-specific paths such as `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了旋转位置编码逻辑，并结合嵌入查找与预处理。它提供了 `_get_rope_param`、`get_rope`、`get_rope_cpu` 以及 `get_rope_wrapper` 等符号，并把这些符号连接到 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: Imports, constants, and runtime setup
```python
"""Factory functions: get_rope, get_rope_cpu, get_rope_wrapper."""

from __future__ import annotations

import logging
from typing import Any, Dict, Optional, Tuple

import torch

from sglang.srt.layers.rotary_embedding.base import (
    LinearScalingRotaryEmbedding,
    RotaryEmbedding,
)
from sglang.srt.layers.rotary_embedding.mrope import (
    MRotaryEmbedding,
    YaRNScalingMRotaryEmbedding,
)
from sglang.srt.layers.rotary_embedding.rope_variant import (
    DeepseekScalingRotaryEmbedding,
    DualChunkRotaryEmbedding,
    DynamicNTKAlphaRotaryEmbedding,
    DynamicNTKScalingRotaryEmbedding,
    FourierRotaryEmbedding,
    Gemma4RotaryEmbedding,
    Llama3RotaryEmbedding,
    Phi3LongRoPEScaledRotaryEmbedding,
)
from sglang.srt.layers.rotary_embedding.yarn import YaRNScalingRotaryEmbedding
from sglang.srt.utils import get_bool_env_var, is_hip

logger = logging.getLogger(__name__)
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `logging`, `typing.Any`, `typing.Dict`, `typing.Optional`, and `typing.Tuple`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `logger` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`logging`、`typing.Any`、`typing.Dict`、`typing.Optional` 以及 `typing.Tuple`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `logger` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 34-53: Internal helper `_get_rope_param`
```python
def _get_rope_param(rope_scaling, key, default, scaling_type):
    """Get a parameter from rope_scaling dict, warn if missing.

    In transformers v5, config.rope_scaling is an alias for rope_parameters
    which may be non-None even for models with no actual scaling (rope_type=default).
    When a required key is missing, this logs a warning instead of silently
    defaulting, to make config mismatches easier to debug.
    """
    if key in rope_scaling:
        return rope_scaling[key]
    logger.warning(
        "rope_scaling (type=%s) missing key '%s', defaulting to %s. "
        "This may indicate a v5 config issue — check model accuracy.",
        scaling_type,
        key,
        default,
    )
    return default
```
**EN:** This block defines `_get_rope_param` and contains the main logic for this step. It mainly invokes `logger.warning`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_get_rope_param`，并承载这一阶段的核心逻辑。 它主要调用 `logger.warning`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 54-62: Module constants and feature-gating logic
```python
_is_hip = is_hip()
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip

if _use_aiter:
    from aiter.rotary_embedding import get_rope as aiter_get_rope

_ROPE_DICT: Dict[Tuple, RotaryEmbedding] = {}
```
**EN:** This section prepares the module namespace. It imports `aiter.rotary_embedding.get_rope`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_hip`, `_use_aiter`, and `_ROPE_DICT` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `aiter.rotary_embedding.get_rope`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_hip`、`_use_aiter` 以及 `_ROPE_DICT` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 63-344: `get_rope` getter for RoPE
```python
def get_rope(
    head_size: int,
    rotary_dim: int,
    max_position: int,
    base: int,
    is_neox_style: bool = True,
    rope_scaling: Optional[Dict[str, Any]] = None,
    dtype: Optional[torch.dtype] = None,
    partial_rotary_factor: float = 1.0,
    dual_chunk_attention_config: Optional[Dict[str, Any]] = None,
) -> RotaryEmbedding:
    if dtype is None:
        dtype = torch.get_default_dtype()
    if rope_scaling is not None:
        rope_scaling_tuple = {
            k: tuple(v) if isinstance(v, list) else v for k, v in rope_scaling.items()
        }
        rope_scaling_args = tuple(rope_scaling_tuple.items())
    else:
        rope_scaling_args = None

    if dual_chunk_attention_config is not None:
        dual_chunk_attention_tuple = {
            k: tuple(v) if isinstance(v, list) else v
            for k, v in dual_chunk_attention_config.items()
            if k != "sparse_attention_config"
        }
        dual_chunk_attention_args = tuple(dual_chunk_attention_tuple.items())
    else:
        dual_chunk_attention_args = None

    if partial_rotary_factor < 1.0:
        rotary_dim = int(rotary_dim * partial_rotary_factor)
    key = (
        head_size,
        rotary_dim,
        max_position,
        base,
        is_neox_style,
        rope_scaling_args,
        dual_chunk_attention_args,
        dtype,
    )
    if key in _ROPE_DICT:
        return _ROPE_DICT[key]

    if dual_chunk_attention_config is not None:
        extra_kwargs = {
            k: v
            for k, v in dual_chunk_attention_config.items()
            if k in ("chunk_size", "local_size")
        }
        rotary_emb = DualChunkRotaryEmbedding(
            head_size,
            rotary_dim,
            max_position,
            base,
            is_neox_style,
            dtype,
            **extra_kwargs,
        )
    elif rope_scaling is None:
        rotary_emb = RotaryEmbedding(
            head_size, rotary_dim, max_position, base, is_neox_style, dtype
        )
    else:
        if "rope_type" in rope_scaling:
            scaling_type = rope_scaling["rope_type"]
        elif "type" in rope_scaling:
            scaling_type = rope_scaling["type"]
        else:
            raise ValueError(
                f"Unknown RoPE scaling type, rope_scaling is {rope_scaling}"
            )

        if scaling_type == "llama3":
            scaling_factor = _get_rope_param(rope_scaling, "factor", 1.0, scaling_type)
            low_freq_factor = _get_rope_param(
                rope_scaling, "low_freq_factor", 1.0, scaling_type
            )
            high_freq_factor = _get_rope_param(
                rope_scaling, "high_freq_factor", 4.0, scaling_type
            )
            original_max_position = _get_rope_param(
                rope_scaling,
                "original_max_position_embeddings",
                max_position,
                scaling_type,
            )
            rotary_emb = Llama3RotaryEmbedding(
                head_size,
                rotary_dim,
                max_position,
                base,
                is_neox_style,
                dtype,
                scaling_factor,
                low_freq_factor,
                high_freq_factor,
                original_max_position,
            )
        elif scaling_type == "default":
            if "mrope_section" in rope_scaling:
                rotary_emb = MRotaryEmbedding(
                    head_size,
                    rotary_dim,
                    max_position,
                    base,
                    is_neox_style,
                    dtype,
                    mrope_section=rope_scaling["mrope_section"],
                    mrope_interleaved=rope_scaling.get("mrope_interleaved", False),
                    mrope_interleaved_glm=rope_scaling.get(
                        "mrope_interleaved_glm", False
                    ),
                )
            elif rope_scaling.get("use_fope", False):
                rotary_emb = FourierRotaryEmbedding(
                    head_size,
                    rotary_dim,
                    max_position,
                    base,
                    is_neox_style,
                    dtype,
                    num_kv_heads=rope_scaling["num_kv_heads"],
                    fope_init_factor=rope_scaling.get("fope_init_factor", 0.1),
                    fope_sep_head=rope_scaling.get("fope_sep_head", True),
                    num_inv_freq=rope_scaling.get("num_inv_freq", None),
                )
            else:
                rotary_emb = RotaryEmbedding(
                    head_size,
                    rotary_dim,
                    max_position,
                    base,
                    is_neox_style,
                    dtype,
                )
        elif scaling_type == "linear":
            scaling_factor = _get_rope_param(rope_scaling, "factor", 1.0, scaling_type)
            rotary_emb = LinearScalingRotaryEmbedding(
                head_size,
                rotary_dim,
                max_position,
                base,
                is_neox_style,
                scaling_factor,
                dtype,
            )
        elif scaling_type == "dynamic":
            scaling_factor = _get_rope_param(rope_scaling, "factor", 1.0, scaling_type)
            if "alpha" in rope_scaling:
                rotary_emb = DynamicNTKAlphaRotaryEmbedding(
                    head_size,
                    rotary_dim,
                    max_position,
                    base,
                    is_neox_style,
                    rope_scaling["alpha"],
                    dtype,
                )
            else:
                rotary_emb = DynamicNTKScalingRotaryEmbedding(
                    head_size,
                    rotary_dim,
                    max_position,
                    base,
                    is_neox_style,
                    scaling_factor,
                    dtype,
                )
        elif scaling_type == "yarn":
            scaling_factor = _get_rope_param(rope_scaling, "factor", 1.0, scaling_type)
            original_max_position = _get_rope_param(
                rope_scaling,
                "original_max_position_embeddings",
                max_position,
                scaling_type,
            )
            extra_kwargs = {
                k: v
                for k, v in rope_scaling.items()
                if k
                in ("extrapolation_factor", "attn_factor", "beta_fast", "beta_slow")
            }
            extra_kwargs["truncate"] = rope_scaling.get("truncate", True)
            if "mrope_section" in rope_scaling:
                rotary_emb = YaRNScalingMRotaryEmbedding(
                    head_size,
                    rotary_dim,
                    original_max_position,
                    base,
                    is_neox_style,
                    scaling_factor,
                    dtype,
                    mrope_section=rope_scaling["mrope_section"],
                    mrope_interleaved=rope_scaling.get("mrope_interleaved", False),
                    **extra_kwargs,
                )
            else:
                rotary_emb = YaRNScalingRotaryEmbedding(
                    head_size,
                    rotary_dim,
                    original_max_position,
                    base,
                    is_neox_style,
                    scaling_factor,
                    dtype,
                    **extra_kwargs,
                )
        elif scaling_type == "deepseek_yarn":
            scaling_factor = _get_rope_param(rope_scaling, "factor", 1.0, scaling_type)
            original_max_position = _get_rope_param(
                rope_scaling,
                "original_max_position_embeddings",
                max_position,
                scaling_type,
            )
            extra_kwargs = {
                k: v
                for k, v in rope_scaling.items()
                if k
                in (
                    "extrapolation_factor",
                    "attn_factor",
                    "beta_fast",
                    "beta_slow",
                    "mscale",
                    "mscale_all_dim",
                )
            }
            rotary_emb = DeepseekScalingRotaryEmbedding(
                head_size,
                rotary_dim,
                original_max_position,
                base,
                is_neox_style,
                scaling_factor,
                dtype,
                **extra_kwargs,
            )
        elif scaling_type == "longrope":
            short_factor = rope_scaling["short_factor"]
            long_factor = rope_scaling["long_factor"]
            original_max_position = _get_rope_param(
                rope_scaling,
                "original_max_position_embeddings",
                max_position,
                scaling_type,
            )
            extra_kwargs = {
                k: v
                for k, v in rope_scaling.items()
                if k in ("short_mscale", "long_mscale")
            }
            rotary_emb = Phi3LongRoPEScaledRotaryEmbedding(
                head_size,
                rotary_dim,
                max_position,
                original_max_position,
                base,
                is_neox_style,
                dtype,
                short_factor,
                long_factor,
                **extra_kwargs,
            )
        elif scaling_type == "proportional":
            rotary_emb = Gemma4RotaryEmbedding(
                head_size,
                rotary_dim,
                max_position,
                base,
                is_neox_style,
                dtype,
            )
        else:
            raise ValueError(f"Unknown RoPE scaling type {scaling_type}")
    _ROPE_DICT[key] = rotary_emb
    return rotary_emb
```
**EN:** This block defines `get_rope` and contains the main logic for this step. It mainly invokes `torch.get_default_dtype`, `tuple`, `int`, `DualChunkRotaryEmbedding`, and `rope_scaling_tuple.items`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `key`, `dtype`, `rope_scaling_tuple`, `rope_scaling_args`, and `dual_chunk_attention_tuple` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_rope`，并承载这一阶段的核心逻辑。 它主要调用 `torch.get_default_dtype`、`tuple`、`int`、`DualChunkRotaryEmbedding` 以及 `rope_scaling_tuple.items`，说明该流程会编排底层辅助函数或计算内核。 像 `key`、`dtype`、`rope_scaling_tuple`、`rope_scaling_args` 以及 `dual_chunk_attention_tuple` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 345-416: `get_rope_cpu` getter for RoPE CPU
```python
def get_rope_cpu(
    head_size: int,
    rotary_dim: int,
    max_position: int,
    base: int,
    is_neox_style: bool = True,
    rope_scaling: Optional[Dict[str, Any]] = None,
    dtype: Optional[torch.dtype] = None,
    partial_rotary_factor: float = 1.0,
    device: Optional[str] = None,
) -> RotaryEmbedding:
    if dtype is None:
        dtype = torch.get_default_dtype()
    if rope_scaling is not None:
        rope_scaling_tuple = {
            k: tuple(v) if isinstance(v, list) else v for k, v in rope_scaling.items()
        }
        rope_scaling_args = tuple(rope_scaling_tuple.items())
    else:
        rope_scaling_args = None
    if partial_rotary_factor < 1.0:
        rotary_dim = int(rotary_dim * partial_rotary_factor)
    key = (
        head_size,
        rotary_dim,
        max_position,
        base,
        is_neox_style,
        rope_scaling_args,
        dtype,
    )
    if key in _ROPE_DICT:
        return _ROPE_DICT[key]

    assert rope_scaling is not None
    scaling_type = rope_scaling["rope_type"]
    assert (
        scaling_type == "deepseek_yarn"
    ), "Only deepseek_yarn is supported for CPU for now"

    scaling_factor = _get_rope_param(rope_scaling, "factor", 1.0, scaling_type)
    original_max_position = _get_rope_param(
        rope_scaling, "original_max_position_embeddings", max_position, scaling_type
    )
    extra_kwargs = {
        k: v
        for k, v in rope_scaling.items()
        if k
        in (
            "extrapolation_factor",
            "attn_factor",
            "beta_fast",
            "beta_slow",
            "mscale",
            "mscale_all_dim",
        )
    }
    extra_kwargs["device"] = device
    rotary_emb = DeepseekScalingRotaryEmbedding(
        head_size,
        rotary_dim,
        original_max_position,
        base,
        is_neox_style,
        scaling_factor,
        dtype,
        **extra_kwargs,
    )
    _ROPE_DICT[key] = rotary_emb
    return rotary_emb
```
**EN:** This block defines `get_rope_cpu` and contains the main logic for this step. It mainly invokes `_get_rope_param`, `DeepseekScalingRotaryEmbedding`, `torch.get_default_dtype`, `tuple`, and `int`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `key`, `scaling_type`, `scaling_factor`, `original_max_position`, and `extra_kwargs` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_rope_cpu`，并承载这一阶段的核心逻辑。 它主要调用 `_get_rope_param`、`DeepseekScalingRotaryEmbedding`、`torch.get_default_dtype`、`tuple` 以及 `int`，说明该流程会编排底层辅助函数或计算内核。 像 `key`、`scaling_type`、`scaling_factor`、`original_max_position` 以及 `extra_kwargs` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 417-451: `get_rope_wrapper` getter for RoPE wrapper logic
```python
def get_rope_wrapper(
    head_size: int,
    rotary_dim: int,
    max_position: int,
    base: int,
    is_neox_style: bool = True,
    rope_scaling: Optional[Dict[str, Any]] = None,
    dtype: Optional[torch.dtype] = None,
    partial_rotary_factor: float = 1.0,
    device: Optional[str] = None,
):
    if device != "cpu":
        wrapper = aiter_get_rope if _use_aiter else get_rope
        return wrapper(
            head_size,
            rotary_dim,
            max_position,
            base,
            is_neox_style,
            rope_scaling,
            dtype,
            partial_rotary_factor,
        )

    return get_rope_cpu(
        head_size,
        rotary_dim,
        max_position,
        base,
        is_neox_style,
        rope_scaling,
        dtype,
        partial_rotary_factor,
        device,
    )
```
**EN:** This block defines `get_rope_wrapper` and contains the main logic for this step. It mainly invokes `get_rope_cpu` and `wrapper`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `wrapper` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_rope_wrapper`，并承载这一阶段的核心逻辑。 它主要调用 `get_rope_cpu` 和 `wrapper`，说明该流程会编排底层辅助函数或计算内核。 像 `wrapper` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `_get_rope_param`, `get_rope`, `get_rope_cpu`, and `get_rope_wrapper`. / **主要符号**：核心入口包括 `_get_rope_param`、`get_rope`、`get_rope_cpu` 以及 `get_rope_wrapper`。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Position encoding**: Describes how rotary embeddings or related position transforms are applied. / **位置编码**：说明如何应用旋转位置编码或相关位置变换。
- **Projection layers**: Focuses on matrix multiplication, weight layout, and projection-oriented wrappers. / **投影层**：关注矩阵乘法、权重布局与面向投影的包装层。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `logging`, `typing.Any`, `typing.Dict`, `typing.Optional`, and `typing.Tuple` / **标准库**：`__future__.annotations`、`logging`、`typing.Any`、`typing.Dict`、`typing.Optional` 以及 `typing.Tuple`
- **Third-party**: `torch` and `aiter.rotary_embedding.get_rope` / **第三方依赖**：`torch` 和 `aiter.rotary_embedding.get_rope`
- **Internal SGLang modules**: `sglang.srt.layers.rotary_embedding.base.LinearScalingRotaryEmbedding`, `sglang.srt.layers.rotary_embedding.base.RotaryEmbedding`, `sglang.srt.layers.rotary_embedding.mrope.MRotaryEmbedding`, `sglang.srt.layers.rotary_embedding.mrope.YaRNScalingMRotaryEmbedding`, `sglang.srt.layers.rotary_embedding.rope_variant.DeepseekScalingRotaryEmbedding`, `sglang.srt.layers.rotary_embedding.rope_variant.DualChunkRotaryEmbedding`, `sglang.srt.layers.rotary_embedding.rope_variant.DynamicNTKAlphaRotaryEmbedding`, `sglang.srt.layers.rotary_embedding.rope_variant.DynamicNTKScalingRotaryEmbedding`, `sglang.srt.layers.rotary_embedding.rope_variant.FourierRotaryEmbedding`, `sglang.srt.layers.rotary_embedding.rope_variant.Gemma4RotaryEmbedding`, `sglang.srt.layers.rotary_embedding.rope_variant.Llama3RotaryEmbedding`, and `sglang.srt.layers.rotary_embedding.rope_variant.Phi3LongRoPEScaledRotaryEmbedding` / **SGLang 内部模块**：`sglang.srt.layers.rotary_embedding.base.LinearScalingRotaryEmbedding`、`sglang.srt.layers.rotary_embedding.base.RotaryEmbedding`、`sglang.srt.layers.rotary_embedding.mrope.MRotaryEmbedding`、`sglang.srt.layers.rotary_embedding.mrope.YaRNScalingMRotaryEmbedding`、`sglang.srt.layers.rotary_embedding.rope_variant.DeepseekScalingRotaryEmbedding`、`sglang.srt.layers.rotary_embedding.rope_variant.DualChunkRotaryEmbedding`、`sglang.srt.layers.rotary_embedding.rope_variant.DynamicNTKAlphaRotaryEmbedding`、`sglang.srt.layers.rotary_embedding.rope_variant.DynamicNTKScalingRotaryEmbedding`、`sglang.srt.layers.rotary_embedding.rope_variant.FourierRotaryEmbedding`、`sglang.srt.layers.rotary_embedding.rope_variant.Gemma4RotaryEmbedding`、`sglang.srt.layers.rotary_embedding.rope_variant.Llama3RotaryEmbedding` 以及 `sglang.srt.layers.rotary_embedding.rope_variant.Phi3LongRoPEScaledRotaryEmbedding`
