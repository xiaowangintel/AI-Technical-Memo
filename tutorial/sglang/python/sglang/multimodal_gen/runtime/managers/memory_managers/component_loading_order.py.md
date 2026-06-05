# component_loading_order.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/managers/memory_managers/component_loading_order.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for component loading order in the multimodal generation stack. Key symbols include `ComponentLoadSpec`, `_component_base_name`, `_component_variant_priority`. / 该模块包含多模态生成体系中与 component loading order 相关的运行时支持代码。 关键符号包括 `ComponentLoadSpec`, `_component_base_name`, `_component_variant_priority`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Imports and module setup / 导入与模块初始化
```python
"""Memory-aware ordering for pipeline component weight loads to avoid OOM while loading.

Load the VRAM-intensive components earlier than others

The pipeline owns component selection, path resolution, and actual loading; this
module only ranks already-selected load specs.
"""

import glob
import json
import os
from dataclasses import dataclass

from sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload_components import (
    is_dit_component_name,
    is_image_encoder_component_name,
    is_text_encoder_component_name,
    is_vae_component_name,
)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 22-31: Class `ComponentLoadSpec` / 类 `ComponentLoadSpec`
```python
@dataclass(frozen=True)
class ComponentLoadSpec:
    """One pipeline component that still needs a real weight load."""

    module_name: str
    load_module_name: str
    component_model_path: str
    transformers_or_diffusers: str
    architecture: str | None
    index: int
```
**EN:** This class models `ComponentLoadSpec`. One pipeline component that still needs a real weight load.
**CN:** 该类实现 `ComponentLoadSpec`。 文档字符串指出：One pipeline component that still needs a real weight load.

### Lines 32-34: Top-level configuration / 顶层配置
```python


_WEIGHT_FILE_SUFFIXES = (".bin", ".pt", ".pth")
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 37-41: Function `_component_base_name` / 函数 `_component_base_name`
```python
def _component_base_name(component_name: str) -> str:
    prefix, separator, suffix = component_name.rpartition("_")
    if separator and suffix.isdigit():
        return prefix
    return component_name
```
**EN:** This function drives `_component_base_name` with inputs such as `component_name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_component_base_name`，主要处理 `component_name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 44-48: Function `_component_variant_priority` / 函数 `_component_variant_priority`
```python
def _component_variant_priority(component_name: str) -> int:
    _, separator, suffix = component_name.rpartition("_")
    if separator and suffix.isdigit():
        return -int(suffix)
    return 0
```
**EN:** This function drives `_component_variant_priority` with inputs such as `component_name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_component_variant_priority`，主要处理 `component_name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 51-62: Function `component_load_risk_rank` / 函数 `component_load_risk_rank`
```python
def component_load_risk_rank(component_name: str) -> int:
    """Fallback type rank when checkpoint size cannot be inferred."""
    candidate_names = (component_name, _component_base_name(component_name))
    if any(is_dit_component_name(name) for name in candidate_names):
        return 0
    if any(is_text_encoder_component_name(name) for name in candidate_names):
        return 1
    if any(is_image_encoder_component_name(name) for name in candidate_names):
        return 2
    if any(is_vae_component_name(name) for name in candidate_names):
        return 3
    return 10
```
**EN:** This function drives `component_load_risk_rank` with inputs such as `component_name`. Fallback type rank when checkpoint size cannot be inferred.
**CN:** 这个函数负责 `component_load_risk_rank`，主要处理 `component_name` 等输入。 文档字符串说明：Fallback type rank when checkpoint size cannot be inferred.

### Lines 65-69: Function `_safe_file_size` / 函数 `_safe_file_size`
```python
def _safe_file_size(file_path: str) -> int | None:
    try:
        return os.path.getsize(file_path)
    except OSError:
        return None
```
**EN:** This function drives `_safe_file_size` with inputs such as `file_path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_safe_file_size`，主要处理 `file_path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 72-91: Function `_safetensors_payload_size_bytes` / 函数 `_safetensors_payload_size_bytes`
```python
def _safetensors_payload_size_bytes(file_path: str) -> int | None:
    try:
        with open(file_path, "rb") as f:
            header_size_bytes = f.read(8)
            if len(header_size_bytes) != 8:
                return _safe_file_size(file_path)
            header_size = int.from_bytes(header_size_bytes, "little")
            header = json.loads(f.read(header_size))
    except (OSError, json.JSONDecodeError, ValueError):
        return _safe_file_size(file_path)

    payload_size = 0
    for tensor_name, tensor_info in header.items():
        if tensor_name == "__metadata__":
# ...
        if not isinstance(offsets, list) or len(offsets) != 2:
            return _safe_file_size(file_path)
        payload_size += offsets[1] - offsets[0]
    return payload_size
```
**EN:** This function drives `_safetensors_payload_size_bytes` with inputs such as `file_path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_safetensors_payload_size_bytes`，主要处理 `file_path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 94-109: Function `_safetensors_files_from_index` / 函数 `_safetensors_files_from_index`
```python
def _safetensors_files_from_index(component_model_path: str) -> list[str]:
    indexed_files: set[str] = set()
    index_paths = sorted(
        glob.glob(os.path.join(component_model_path, "*.safetensors.index.json"))
    )
    for index_path in index_paths:
        try:
            with open(index_path) as f:
                weight_map = json.load(f).get("weight_map", {})
        except (OSError, json.JSONDecodeError):
            continue
        for shard_name in weight_map.values():
            shard_path = os.path.join(component_model_path, shard_name)
            if os.path.isfile(shard_path):
                indexed_files.add(shard_path)
    return sorted(indexed_files)
```
**EN:** This function drives `_safetensors_files_from_index` with inputs such as `component_model_path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_safetensors_files_from_index`，主要处理 `component_model_path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 112-123: Function `_list_component_safetensors_files` / 函数 `_list_component_safetensors_files`
```python
def _list_component_safetensors_files(component_model_path: str) -> list[str]:
    if os.path.isfile(component_model_path):
        if component_model_path.endswith(".safetensors"):
            return [component_model_path]
        return []
    if not os.path.isdir(component_model_path):
        return []

    indexed_files = _safetensors_files_from_index(component_model_path)
    if indexed_files:
        return indexed_files
    return sorted(glob.glob(os.path.join(component_model_path, "*.safetensors")))
```
**EN:** This function drives `_list_component_safetensors_files` with inputs such as `component_model_path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_list_component_safetensors_files`，主要处理 `component_model_path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 126-157: Function `infer_component_weight_size_bytes` / 函数 `infer_component_weight_size_bytes`
```python
def infer_component_weight_size_bytes(component_model_path: str) -> int | None:
    """Infer checkpoint payload size from safetensors without materializing tensors."""
    safetensors_files = _list_component_safetensors_files(component_model_path)
    if safetensors_files:
        sizes = [
            size
            for size in (
                _safetensors_payload_size_bytes(file_path)
                for file_path in safetensors_files
            )
            if size is not None
        ]
        return sum(sizes) if sizes else None

# ...
        for size in (_safe_file_size(file_path) for file_path in weight_files)
        if size is not None
    ]
    return sum(sizes) if sizes else None
```
**EN:** This function drives `infer_component_weight_size_bytes` with inputs such as `component_model_path`. Infer checkpoint payload size from safetensors without materializing tensors.
**CN:** 这个函数负责 `infer_component_weight_size_bytes`，主要处理 `component_model_path` 等输入。 文档字符串说明：Infer checkpoint payload size from safetensors without materializing tensors.

### Lines 160-174: Function `order_component_load_specs` / 函数 `order_component_load_specs`
```python
def order_component_load_specs(
    component_specs: list[ComponentLoadSpec],
) -> list[ComponentLoadSpec]:
    # load larger weight payloads before small helpers to reduce startup peak OOMs
    return sorted(
        component_specs,
        key=lambda spec: (
            # 1. model size inferred from checkpoints
            -(infer_component_weight_size_bytes(spec.component_model_path) or 0),
            # 2. infer from component name
            component_load_risk_rank(spec.load_module_name),
            _component_variant_priority(spec.load_module_name),
            spec.index,
        ),
    )
```
**EN:** This function drives `order_component_load_specs` with inputs such as `component_specs`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `order_component_load_specs`，主要处理 `component_specs` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- Pipeline orchestration / 流水线编排
- Image generation flow / 图像生成流程
- Symbol `ComponentLoadSpec` anchors the module API / 符号 `ComponentLoadSpec` 构成该模块的核心 API
- Symbol `_component_base_name` anchors the module API / 符号 `_component_base_name` 构成该模块的核心 API
- Symbol `_component_variant_priority` anchors the module API / 符号 `_component_variant_priority` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload_components`
- **Stdlib / 标准库**: `glob`, `json`, `os`, `dataclasses`
