# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module collects reusable helper functions, type adapters, and lightweight abstractions shared by multiple SRT model implementations. / 该模块汇总多个 SRT 模型实现共享的辅助函数、类型适配器和轻量抽象。

## Line-by-Line Analysis / 逐行分析

### Lines 14-37: Module imports
```python
from __future__ import annotations

import itertools
from collections.abc import Iterable, Mapping
from dataclasses import dataclass, field
from functools import lru_cache
from typing import TYPE_CHECKING, Any, Optional, Tuple

import numpy as np
import torch
import triton
import triton.language as tl

from sglang.jit_kernel.norm import can_use_fused_inplace_qknorm, fused_inplace_qknorm
from sglang.srt.environ import envs
from sglang.srt.layers.radix_attention import RadixAttention
from sglang.srt.layers.utils.cp_utils import is_prefill_context_parallel_enabled
from sglang.srt.mem_cache.swa_memory_pool import SWAKVPool
from sglang.srt.model_executor.cuda_graph_runner import get_is_capture_mode
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_loader.weight_utils import default_weight_loader
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import get_current_device_stream_fast, is_cuda, is_hip
from sglang.srt.utils.custom_op import register_custom_op
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、SGLang 运行时组件。

### Lines 39-40: Top-level if
```python
if TYPE_CHECKING:
    from sglang.srt.layers.layernorm import RMSNorm
```
**EN:** Applies conditional top-level logic so the module can adapt to optional dependencies, feature flags, or environment-specific behavior.
**CN:** 应用条件化的顶层逻辑，使模块能够适配可选依赖、功能开关或特定环境行为。

### Lines 42-42: Top-level assign
```python
_is_cuda = is_cuda()
```
**EN:** Defines or updates _is_cuda, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cuda，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 43-43: Top-level assign
```python
_is_hip = is_hip()
```
**EN:** Defines or updates _is_hip, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_hip，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 45-45: Top-level assign
```python
WeightsMapping = Mapping[str, Optional[str]]
```
**EN:** Defines or updates WeightsMapping, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 WeightsMapping，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 46-46: Top-level expr
```python
"""If a key maps to a value of `None`, the corresponding weight is ignored."""
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 49-52: Class `WeightsMapper` overview
```python
@dataclass
class WeightsMapper:
    """Maps the name of each weight if they match the following patterns."""
```
**EN:** Defines `WeightsMapper` as a reusable runtime type derived from no explicit base class. The class groups 5 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `WeightsMapper`，其继承关系为 no explicit base class。这个类组织了 5 个方法，用于实现模型相关行为。

### Lines 53-55: Class `WeightsMapper` attributes
```python
    orig_to_new_substr: WeightsMapping = field(default_factory=dict)
    orig_to_new_prefix: WeightsMapping = field(default_factory=dict)
    orig_to_new_suffix: WeightsMapping = field(default_factory=dict)
```
**EN:** Defines class-level attributes and metadata that shape how `WeightsMapper` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `WeightsMapper` 在运行时的行为。

### Lines 57-62: Method `WeightsMapper.__or__`
```python
    def __or__(self, other: "WeightsMapper") -> "WeightsMapper":
        return WeightsMapper(
            orig_to_new_substr={**self.orig_to_new_substr, **other.orig_to_new_substr},
            orig_to_new_prefix={**self.orig_to_new_prefix, **other.orig_to_new_prefix},
            orig_to_new_suffix={**self.orig_to_new_suffix, **other.orig_to_new_suffix},
        )
```
**EN:** This method implements `__or__(other: ...)` and implements a focused helper that supports the surrounding runtime flow inside `WeightsMapper`.
**CN:** 这个方法实现了 `__or__(other: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `WeightsMapper` 内部调用。

### Lines 64-95: Method `WeightsMapper._map_name`
```python
    def _map_name(self, key: str) -> Optional[str]:
        for substr, new_key in sorted(
            self.orig_to_new_substr.items(), key=lambda i: len(i[0]), reverse=True
        ):
            if substr in key:
                if new_key is None:
                    return None

                key = key.replace(substr, new_key, 1)
                break

        for prefix, new_key in sorted(
            self.orig_to_new_prefix.items(), key=lambda i: len(i[0]), reverse=True
        ):
            if key.startswith(prefix):
                if new_key is None:
                    return None

                key = key.replace(prefix, new_key, 1)
                break

        for suffix, new_key in sorted(
            self.orig_to_new_suffix.items(), key=lambda i: len(i[0]), reverse=True
        ):
            if key.endswith(suffix):
                if new_key is None:
                    return None

                key = new_key.join(key.rsplit(suffix, 1))
                break

        return key
```
**EN:** This method implements `_map_name(key: ...)` and implements a focused helper that supports the surrounding runtime flow inside `WeightsMapper`.
**CN:** 这个方法实现了 `_map_name(key: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `WeightsMapper` 内部调用。

### Lines 97-104: Method `WeightsMapper.apply`
```python
    def apply(
        self, weights: Iterable[tuple[str, torch.Tensor]]
    ) -> Iterable[tuple[str, torch.Tensor]]:
        return (
            (out_name, data)
            for name, data in weights
            if (out_name := self._map_name(name)) is not None
        )
```
**EN:** This method implements `apply(weights: ...)` and implements a focused helper that supports the surrounding runtime flow inside `WeightsMapper`.
**CN:** 这个方法实现了 `apply(weights: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `WeightsMapper` 内部调用。

### Lines 106-111: Method `WeightsMapper.apply_list`
```python
    def apply_list(self, values: list[str]) -> list[str]:
        return [
            out_name
            for name in values
            if (out_name := self._map_name(name)) is not None
        ]
```
**EN:** This method implements `apply_list(values: ...)` and implements a focused helper that supports the surrounding runtime flow inside `WeightsMapper`.
**CN:** 这个方法实现了 `apply_list(values: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `WeightsMapper` 内部调用。

### Lines 113-118: Method `WeightsMapper.apply_dict`
```python
    def apply_dict(self, values: dict[str, Any]) -> dict[str, Any]:
        return {
            out_name: value
            for name, value in values.items()
            if (out_name := self._map_name(name)) is not None
        }
```
**EN:** This method implements `apply_dict(values: ...)` and implements a focused helper that supports the surrounding runtime flow inside `WeightsMapper`.
**CN:** 这个方法实现了 `apply_dict(values: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `WeightsMapper` 内部调用。

### Lines 121-121: Class `AutoWeightsLoader` overview
```python
class AutoWeightsLoader:
```
**EN:** Defines `AutoWeightsLoader` as a reusable runtime type derived from no explicit base class. The class groups 8 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `AutoWeightsLoader`，其继承关系为 no explicit base class。这个类组织了 8 个方法，用于实现模型相关行为。

### Lines 122-127: Class `AutoWeightsLoader` attributes
```python
    ROTARY_EMBEDS_UNUSED_WEIGHTS = [
        "rotary_pos_emb.inv_freq",
        "rotary_emb.inv_freq",
        "rotary_emb.cos_cached",
        "rotary_emb.sin_cached",
    ]
```
**EN:** Defines class-level attributes and metadata that shape how `AutoWeightsLoader` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `AutoWeightsLoader` 在运行时的行为。

### Lines 129-145: Method `AutoWeightsLoader.__init__`
```python
    def __init__(
        self,
        module: torch.nn.Module,
        *,
        skip_prefixes: list[str] | None = None,
        skip_substrs: list[str] | None = None,
        ignore_unexpected_prefixes: list[str] | None = None,
        ignore_unexpected_suffixes: list[str] | None = None,
    ) -> None:
        self.module = module
        self.skip_prefixes = list(skip_prefixes or [])
        self.skip_substrs = [
            *(skip_substrs or []),
            *self.ROTARY_EMBEDS_UNUSED_WEIGHTS,
        ]
        self.ignore_unexpected_prefixes = list(ignore_unexpected_prefixes or [])
        self.ignore_unexpected_suffixes = list(ignore_unexpected_suffixes or [])
```
**EN:** This method implements `__init__(module: ..., *, skip_prefixes: ...=..., skip_substrs: ...=..., ignore_unexpected_prefixes: ...=..., ignore_unexpected_suffixes: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(module: ..., *, skip_prefixes: ...=..., skip_substrs: ...=..., ignore_unexpected_prefixes: ...=..., ignore_unexpected_suffixes: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 147-159: Method `AutoWeightsLoader._groupby_prefix`
```python
    def _groupby_prefix(
        self,
        weights: Iterable[tuple[str, torch.Tensor]],
    ) -> Iterable[tuple[str, Iterable[tuple[str, torch.Tensor]]]]:
        weights_by_parts = (
            (weight_name.split(".", 1), weight_data)
            for weight_name, weight_data in weights
        )
        for prefix, group in itertools.groupby(weights_by_parts, key=lambda x: x[0][0]):
            yield prefix, (
                ("" if len(parts) == 1 else parts[1], weight_data)
                for parts, weight_data in group
            )
```
**EN:** This method implements `_groupby_prefix(weights: ...)` and implements a focused helper that supports the surrounding runtime flow inside `AutoWeightsLoader`.
**CN:** 这个方法实现了 `_groupby_prefix(weights: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `AutoWeightsLoader` 内部调用。

### Lines 161-167: Method `AutoWeightsLoader._get_qualname`
```python
    @staticmethod
    def _get_qualname(prefix: str, rest: str) -> str:
        if prefix == "":
            return rest
        if rest == "":
            return prefix
        return f"{prefix}.{rest}"
```
**EN:** This method implements `_get_qualname(rest: ...)` and returns a cached or derived object used elsewhere in the pipeline Decorators: staticmethod.
**CN:** 这个方法实现了 `_get_qualname(rest: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果 装饰器：staticmethod。

### Lines 169-172: Method `AutoWeightsLoader._can_skip`
```python
    def _can_skip(self, qualname: str) -> bool:
        return any(qualname.startswith(p) for p in self.skip_prefixes) or any(
            sub in qualname for sub in self.skip_substrs
        )
```
**EN:** This method implements `_can_skip(qualname: ...)` and implements a focused helper that supports the surrounding runtime flow inside `AutoWeightsLoader`.
**CN:** 这个方法实现了 `_can_skip(qualname: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `AutoWeightsLoader` 内部调用。

### Lines 174-177: Method `AutoWeightsLoader._can_ignore_unexpected`
```python
    def _can_ignore_unexpected(self, qualname: str) -> bool:
        return any(
            qualname.startswith(p) for p in self.ignore_unexpected_prefixes
        ) or any(qualname.endswith(s) for s in self.ignore_unexpected_suffixes)
```
**EN:** This method implements `_can_ignore_unexpected(qualname: ...)` and implements a focused helper that supports the surrounding runtime flow inside `AutoWeightsLoader`.
**CN:** 这个方法实现了 `_can_ignore_unexpected(qualname: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `AutoWeightsLoader` 内部调用。

### Lines 179-199: Method `AutoWeightsLoader._load_param`
```python
    def _load_param(
        self,
        base_prefix: str,
        param: torch.nn.Parameter,
        weights: Iterable[tuple[str, torch.Tensor]],
    ) -> Iterable[str]:
        for weight_name, weight_data in weights:
            weight_qualname = self._get_qualname(base_prefix, weight_name)
            if self._can_skip(weight_qualname):
                continue
            if weight_name != "":
                if self._can_ignore_unexpected(weight_qualname):
                    continue
                raise ValueError(
                    f"Attempted to load nested weight {weight_qualname!r} "
                    f"into parameter {base_prefix!r}"
                )

            weight_loader = getattr(param, "weight_loader", default_weight_loader)
            weight_loader(param, weight_data)
            yield weight_qualname
```
**EN:** This method implements `_load_param(base_prefix: ..., param: ..., weights: ...)` and implements a focused helper that supports the surrounding runtime flow inside `AutoWeightsLoader`.
**CN:** 这个方法实现了 `_load_param(base_prefix: ..., param: ..., weights: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `AutoWeightsLoader` 内部调用。

### Lines 201-260: Method `AutoWeightsLoader._load_module`
```python
    def _load_module(
        self,
        base_prefix: str,
        module: torch.nn.Module,
        weights: Iterable[tuple[str, torch.Tensor]],
    ) -> Iterable[str]:
        if module.__class__.__name__ == "PPMissingLayer":
            return

        if module is not self.module:
            module_load_weights = getattr(module, "load_weights", None)
            if callable(module_load_weights):
                loaded = module_load_weights(weights)
                if loaded is not None:
                    yield from (
                        self._get_qualname(base_prefix, loaded_name)
                        for loaded_name in loaded
                    )
                return

        child_modules = dict(module.named_children())
        child_params = dict(module.named_parameters(recurse=False))
        child_buffers = dict(module.named_buffers(recurse=False))
        for child_prefix, child_weights in self._groupby_prefix(weights):
            prefix = self._get_qualname(base_prefix, child_prefix)
            if child_prefix in child_modules:
                if self._can_skip(prefix + "."):
                    continue
                yield from self._load_module(
                    prefix,
                    child_modules[child_prefix],
                    child_weights,
                )
                continue

            if child_prefix in child_params:
                if self._can_skip(prefix):
                    continue
                yield from self._load_param(
                    prefix, child_params[child_prefix], child_weights
# ... truncated for brevity ...
```
**EN:** This method implements `_load_module(base_prefix: ..., module: ..., weights: ...)` and implements a focused helper that supports the surrounding runtime flow inside `AutoWeightsLoader`.
**CN:** 这个方法实现了 `_load_module(base_prefix: ..., module: ..., weights: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `AutoWeightsLoader` 内部调用。

### Lines 262-273: Method `AutoWeightsLoader.load_weights`
```python
    def load_weights(
        self,
        weights: Iterable[tuple[str, torch.Tensor]],
        *,
        mapper: WeightsMapper | None = None,
    ) -> set[str]:
        if mapper is not None:
            weights = mapper.apply(weights)
        weights = (
            (name, weight) for name, weight in weights if not self._can_skip(name)
        )
        return set(self._load_module("", self.module, weights))
```
**EN:** This method implements `load_weights(weights: ..., *, mapper: ...=...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ..., *, mapper: ...=...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 276-284: Function `enable_fused_set_kv_buffer`
```python
def enable_fused_set_kv_buffer(forward_batch: ForwardBatch):
    """Enable fused set_kv_buffer only on CUDA with bfloat16 KV cache."""
    return (
        _is_cuda
        and hasattr(forward_batch.token_to_kv_pool, "dtype")
        and forward_batch.token_to_kv_pool.dtype == torch.bfloat16
        and not isinstance(forward_batch.token_to_kv_pool, SWAKVPool)
        and not is_prefill_context_parallel_enabled()
    ) or (_is_hip and not is_prefill_context_parallel_enabled())
```
**EN:** This function implements `enable_fused_set_kv_buffer(forward_batch: ...)` and Enable fused set_kv_buffer only on CUDA with bfloat16 KV cache.
**CN:** 这个函数实现了 `enable_fused_set_kv_buffer(forward_batch: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 287-327: Function `create_fused_set_kv_buffer_arg`
```python
def create_fused_set_kv_buffer_arg(
    value: torch.Tensor,
    layer: RadixAttention,
    forward_batch: ForwardBatch,
):
    from sglang.jit_kernel.rope import FusedSetKVBufferArg

    layer_id = layer.layer_id
    token_to_kv_pool = forward_batch.token_to_kv_pool

    k_buffer = token_to_kv_pool.get_key_buffer(layer_id)
    v_buffer = token_to_kv_pool.get_value_buffer(layer_id)

    if not _is_hip:
        assert layer.k_scale is None and layer.v_scale is None, "scale not supported"
        return FusedSetKVBufferArg(
            value=value,
            k_buffer=k_buffer.view(k_buffer.shape[0], -1),
            v_buffer=v_buffer.view(v_buffer.shape[0], -1),
            cache_loc=forward_batch.out_cache_loc,
        )
    else:
        page_size = token_to_kv_pool.page_size
        slot_mapping_swa = (
            token_to_kv_pool.full_to_swa_index_mapping.long()
            if layer.sliding_window_size > 0
            else None
        )
        return {
            "v": value.view(-1, layer.tp_v_head_num, layer.v_head_dim),
            "k_scale": layer.k_scale,
            "v_scale": layer.v_scale,
            "key_cache": k_buffer.view(
                -1, page_size, layer.tp_k_head_num, layer.qk_head_dim
            ),
            "value_cache": v_buffer.view(
                -1, page_size, layer.tp_v_head_num, layer.v_head_dim
            ),
            "slot_mapping": forward_batch.out_cache_loc,
            "swa_slot_mapping": slot_mapping_swa,
# ... truncated for brevity ...
```
**EN:** This function implements `create_fused_set_kv_buffer_arg(value: ..., layer: ..., forward_batch: ...)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `create_fused_set_kv_buffer_arg(value: ..., layer: ..., forward_batch: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 330-333: Function `permute_inv`
```python
def permute_inv(perm: torch.Tensor) -> torch.Tensor:
    inv_perm = torch.empty_like(perm)
    inv_perm[perm] = torch.arange(perm.numel(), device=perm.device, dtype=perm.dtype)
    return inv_perm
```
**EN:** This function implements `permute_inv(perm: ...)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `permute_inv(perm: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 336-355: Function `compute_cu_seqlens_from_grid_numpy`
```python
def compute_cu_seqlens_from_grid_numpy(grid_thw: torch.Tensor) -> torch.Tensor:
    """
    Compute cu_seqlens from grid_thw using NumPy.

    grid_thw: [T, 3] int tensor on CPU.
              columns: [repeat_count, H, W]
    Returns:
        cu_seqlens: 1D int32 tensor on CPU, shape [N + 1]
    """
    assert (
        grid_thw.device.type == "cpu"
    ), "compute_cu_seqlens_from_grid_numpy expects a CPU tensor"
    arr = grid_thw.numpy()

    cu_seqlens = np.repeat(arr[:, 1] * arr[:, 2], arr[:, 0]).cumsum(
        axis=0, dtype=np.int32
    )
    cu_seqlens = np.concatenate([np.zeros(1, dtype=np.int32), cu_seqlens])
    cu_seqlens = torch.from_numpy(cu_seqlens)
    return cu_seqlens
```
**EN:** This function implements `compute_cu_seqlens_from_grid_numpy(grid_thw: ...)` and Compute cu_seqlens from grid_thw using NumPy.
**CN:** 这个函数实现了 `compute_cu_seqlens_from_grid_numpy(grid_thw: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 358-361: Class `RotaryPosMixin` overview
```python
class RotaryPosMixin:

    @staticmethod
    @lru_cache(maxsize=1024)
```
**EN:** Defines `RotaryPosMixin` as a reusable runtime type derived from no explicit base class. The class groups 1 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `RotaryPosMixin`，其继承关系为 no explicit base class。这个类组织了 1 个方法，用于实现模型相关行为。

### Lines 360-391: Method `RotaryPosMixin.rot_pos_ids`
```python
    @staticmethod
    @lru_cache(maxsize=1024)
    def rot_pos_ids(h: int, w: int, spatial_merge_size: int) -> torch.Tensor:
        if isinstance(h, torch.Tensor):
            h = int(h.item())
        if isinstance(w, torch.Tensor):
            w = int(w.item())
        if isinstance(spatial_merge_size, torch.Tensor):
            spatial_merge_size = int(spatial_merge_size.item())
        hpos_ids = np.broadcast_to(np.arange(h).reshape(h, 1), (h, w))
        h_div = h // spatial_merge_size
        w_div = w // spatial_merge_size
        hpos_ids = hpos_ids.reshape(
            h_div,
            spatial_merge_size,
            w_div,
            spatial_merge_size,
        )
        hpos_ids = hpos_ids.transpose(0, 2, 1, 3)
        hpos_ids = hpos_ids.flatten()

        wpos_ids = np.broadcast_to(np.arange(w).reshape(1, w), (h, w))
        wpos_ids = wpos_ids.reshape(
            h_div,
            spatial_merge_size,
            w_div,
            spatial_merge_size,
        )
        wpos_ids = wpos_ids.transpose(0, 2, 1, 3)
        wpos_ids = wpos_ids.flatten()

        return torch.from_numpy(np.stack([hpos_ids, wpos_ids], axis=-1))
```
**EN:** This method implements `rot_pos_ids(w: ..., spatial_merge_size: ...)` and implements a focused helper that supports the surrounding runtime flow inside `RotaryPosMixin` Decorators: staticmethod, lru_cache(maxsize=1024).
**CN:** 这个方法实现了 `rot_pos_ids(w: ..., spatial_merge_size: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `RotaryPosMixin` 内部调用 装饰器：staticmethod, lru_cache(maxsize=1024)。

### Lines 394-413: Function `_reshape_for_qk_norm`
```python
def _reshape_for_qk_norm(x: torch.Tensor, head_dim: int) -> torch.Tensor:
    """Reshape a (..., H*D) tensor into (..., H, D) ahead of QK RMSNorm.

    On CUDA with the inductor piecewise-cuda-graph compiler, return a
    stride-preserving view so inductor can fuse this reshape with the
    subsequent RMSNorm (and any upstream/downstream FP8 quant) into a
    single triton kernel -- the original motivation of #21734.

    Everywhere else (ROCm, or CUDA with the eager PCG fallback), use the
    flat 2D reshape that forces a copy when the input is a non-contiguous
    QKV-split stride-trick view. ROCm's RMSNorm kernels assume contiguous
    inputs and fault on strided tensors (root cause of the #21734 revert
    in #23159).
    """
    if (
        _is_cuda
        and get_global_server_args().piecewise_cuda_graph_compiler == "inductor"
    ):
        return x.view(*x.shape[:-1], -1, head_dim)
    return x.reshape(-1, head_dim)
```
**EN:** This function implements `_reshape_for_qk_norm(x: ..., head_dim: ...)` and Reshape a (..., H*D) tensor into (..., H, D) ahead of QK RMSNorm.
**CN:** 这个函数实现了 `_reshape_for_qk_norm(x: ..., head_dim: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 416-480: Function `apply_qk_norm`
```python
def apply_qk_norm(
    q: torch.Tensor,
    k: torch.Tensor,
    q_norm: RMSNorm,
    k_norm: RMSNorm,
    head_dim: int,
    alt_stream: Optional[torch.cuda.Stream] = None,
    allow_inplace: bool = True,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """
    Apply QK normalization for query and key tensors.
    If eligible, we will use JIT fused inplace QK normalization for better performance.

    Args:
        q: Query tensor of shape [batch_size, ...]
        k: Key tensor of shape [batch_size, ...]
        q_norm: RMSNorm layer for query normalization
        k_norm: RMSNorm layer for key normalization
        head_dim: Dimension of each attention head
        alt_stream: Optional alternative CUDA stream for overlapping computation
        allow_inplace: Whether to allow inplace normalization. (True for better performance)

    Returns:
        Tuple of normalized query and key tensors
    """

    batch_size = q.size(0)
    q_eps = q_norm.variance_epsilon
    k_eps = k_norm.variance_epsilon
    if (
        _is_cuda  # TODO(dark): have not tested on ROCm or other backends
        and allow_inplace  # TODO(dark): this can be relaxed if needed
        and (q_eps == k_eps)  # TODO(dark): this can also be relaxed
        and not envs.SGLANG_ENABLE_DETERMINISTIC_INFERENCE.get()
        and get_global_server_args().piecewise_cuda_graph_compiler
        != "inductor"  # let inductor fuse QK norm
        and can_use_fused_inplace_qknorm(head_dim, q.dtype)
    ):
        fused_inplace_qknorm(
            q=q.view(batch_size, -1, head_dim),
# ... truncated for brevity ...
```
**EN:** This function implements `apply_qk_norm(q: ..., k: ..., q_norm: ..., k_norm: ..., head_dim: ..., alt_stream: ...=..., ...)` and Apply QK normalization for query and key tensors.
**CN:** 这个函数实现了 `apply_qk_norm(q: ..., k: ..., q_norm: ..., k_norm: ..., head_dim: ..., alt_stream: ...=..., ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 489-528: Function `_fused_qk_gemma_rmsnorm_kernel`
```python
@triton.jit
def _fused_qk_gemma_rmsnorm_kernel(
    Q_ptr,
    K_ptr,
    Q_out_ptr,
    K_out_ptr,
    QW_ptr,
    KW_ptr,
    q_stride,
    k_stride,
    k_rows,
    HEAD_DIM: tl.constexpr,
    BLOCK_HD: tl.constexpr,
    EPS: tl.constexpr,
    FP16: tl.constexpr,
):
    pid = tl.program_id(0)
    cols = tl.arange(0, BLOCK_HD)
    mask = cols < HEAD_DIM
    out_dtype = tl.float16 if FP16 else tl.bfloat16

    # Q norm (every block) — use q_stride to handle non-contiguous input
    q_off = pid * q_stride + cols
    q = tl.load(Q_ptr + q_off, mask=mask, other=0.0).to(tl.float32)
    w_q = tl.load(QW_ptr + cols, mask=mask, other=0.0).to(tl.float32)
    q_var = tl.sum(q * q, axis=0) / HEAD_DIM
    q_normed = (q * tl.rsqrt(q_var + EPS) * (w_q + 1.0)).to(out_dtype)
    # output is always contiguous
    q_out_off = pid * HEAD_DIM + cols
    tl.store(Q_out_ptr + q_out_off, q_normed, mask=mask)

    # K norm (first k_rows blocks only) — use k_stride for input
    if pid < k_rows:
        k_off = pid * k_stride + cols
        k = tl.load(K_ptr + k_off, mask=mask, other=0.0).to(tl.float32)
        w_k = tl.load(KW_ptr + cols, mask=mask, other=0.0).to(tl.float32)
        k_var = tl.sum(k * k, axis=0) / HEAD_DIM
        k_normed = (k * tl.rsqrt(k_var + EPS) * (w_k + 1.0)).to(out_dtype)
        k_out_off = pid * HEAD_DIM + cols
        tl.store(K_out_ptr + k_out_off, k_normed, mask=mask)
```
**EN:** This function implements `_fused_qk_gemma_rmsnorm_kernel(Q_ptr, K_ptr, Q_out_ptr, K_out_ptr, QW_ptr, KW_ptr, ...)` and implements a focused helper that supports the surrounding runtime flow Decorators: triton.jit.
**CN:** 这个函数实现了 `_fused_qk_gemma_rmsnorm_kernel(Q_ptr, K_ptr, Q_out_ptr, K_out_ptr, QW_ptr, KW_ptr, ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑 装饰器：triton.jit。

### Lines 531-573: Function `fused_qk_gemma_rmsnorm`
```python
def fused_qk_gemma_rmsnorm(
    q: torch.Tensor,
    k: torch.Tensor,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
    eps: float,
    head_dim: int,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """Fused QK GemmaRMSNorm — single Triton kernel for both q_norm and k_norm.

    grid = q_rows; every block processes its Q row, and the first k_rows
    blocks also process K.  No torch.cat, no slice, no tl.where.
    Passes input strides to the kernel so non-contiguous tensors (e.g. from
    qkv.split()) are read correctly without an extra .contiguous() copy.
    """
    q_flat = q.reshape(-1, head_dim)
    k_flat = k.reshape(-1, head_dim)

    q_rows = q_flat.shape[0]
    k_rows = k_flat.shape[0]

    q_out = torch.empty(q_rows, head_dim, dtype=q.dtype, device=q.device)
    k_out = torch.empty(k_rows, head_dim, dtype=k.dtype, device=k.device)

    BLOCK_HD = triton.next_power_of_2(head_dim)

    _fused_qk_gemma_rmsnorm_kernel[(q_rows,)](
        q_flat,
        k_flat,
        q_out,
        k_out,
        q_weight,
        k_weight,
        q_flat.stride(0),
        k_flat.stride(0),
        k_rows,
        HEAD_DIM=head_dim,
        BLOCK_HD=BLOCK_HD,
        EPS=eps,
        FP16=(q.dtype == torch.float16),
# ... truncated for brevity ...
```
**EN:** This function implements `fused_qk_gemma_rmsnorm(q: ..., k: ..., q_weight: ..., k_weight: ..., eps: ..., head_dim: ...)` and Fused QK GemmaRMSNorm — single Triton kernel for both q_norm and k_norm.
**CN:** 这个函数实现了 `fused_qk_gemma_rmsnorm(q: ..., k: ..., q_weight: ..., k_weight: ..., eps: ..., head_dim: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 577-577: Top-level assign
```python
fused_inplace_qknorm = register_custom_op(fused_inplace_qknorm, mutates_args=["q", "k"])
```
**EN:** Defines or updates fused_inplace_qknorm, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 fused_inplace_qknorm，为后续代码提供模块级配置、元数据或缓存对象。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `__future__.annotations`
- `itertools`
- `collections.abc.Iterable`
- `collections.abc.Mapping`
- `dataclasses.dataclass`
- `dataclasses.field`
- `functools.lru_cache`
- `typing.TYPE_CHECKING`
- `typing.Any`
- `typing.Optional`
- `typing.Tuple`
- `numpy`
- `torch`
- `triton`
- `triton.language`
- `sglang.jit_kernel.norm.can_use_fused_inplace_qknorm`
- `sglang.jit_kernel.norm.fused_inplace_qknorm`
- `sglang.srt.environ.envs`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.utils.cp_utils.is_prefill_context_parallel_enabled`
- `sglang.srt.mem_cache.swa_memory_pool.SWAKVPool`
- `sglang.srt.model_executor.cuda_graph_runner.get_is_capture_mode`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.get_current_device_stream_fast`
- `sglang.srt.utils.is_cuda`
- `sglang.srt.utils.is_hip`
- `sglang.srt.utils.custom_op.register_custom_op`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.jit_kernel.rope.FusedSetKVBufferArg`
