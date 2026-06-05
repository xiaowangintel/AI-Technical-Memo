# layerwise_offload_components.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/managers/memory_managers/layerwise_offload_components.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for layerwise offload components in the multimodal generation stack. Key symbols include `is_dit_component_name`, `is_text_encoder_component_name`, `is_image_encoder_component_name`. / 该模块包含多模态生成体系中与 layerwise offload components 相关的运行时支持代码。 关键符号包括 `is_dit_component_name`, `is_text_encoder_component_name`, `is_image_encoder_component_name`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-49: Imports and module setup / 导入与模块初始化
```python
from collections.abc import Collection, Sequence

LAYERWISE_OFFLOAD_ALL_COMPONENTS = "all"
LAYERWISE_OFFLOAD_DIT_GROUP = "dit"
LAYERWISE_OFFLOAD_TEXT_ENCODER_GROUP = "text_encoder"
LAYERWISE_OFFLOAD_IMAGE_ENCODER_GROUP = "image_encoder"
LAYERWISE_OFFLOAD_VAE_GROUP = "vae"
LAYERWISE_OFFLOAD_DEFAULT_GROUP = "default"

# Components whose layerwise policy has been validated as a better default than
# component-level CPU offload when the user has not pinned their placement.
LAYERWISE_OFFLOAD_DEFAULT_GROUP_COMPONENTS = (
    LAYERWISE_OFFLOAD_TEXT_ENCODER_GROUP,
    LAYERWISE_OFFLOAD_IMAGE_ENCODER_GROUP,
# ...
    "text_encoder_cpu_offload",
    "image_encoder_cpu_offload",
    "vae_cpu_offload",
)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 52-53: Function `is_dit_component_name` / 函数 `is_dit_component_name`
```python
def is_dit_component_name(component_name: str) -> bool:
    return component_name in DIT_COMPONENT_NAMES
```
**EN:** This function drives `is_dit_component_name` with inputs such as `component_name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `is_dit_component_name`，主要处理 `component_name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 56-59: Function `is_text_encoder_component_name` / 函数 `is_text_encoder_component_name`
```python
def is_text_encoder_component_name(component_name: str) -> bool:
    return component_name.startswith("text_encoder") or component_name.endswith(
        "text_encoder"
    )
```
**EN:** This function drives `is_text_encoder_component_name` with inputs such as `component_name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `is_text_encoder_component_name`，主要处理 `component_name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 62-63: Function `is_image_encoder_component_name` / 函数 `is_image_encoder_component_name`
```python
def is_image_encoder_component_name(component_name: str) -> bool:
    return component_name == "image_encoder"
```
**EN:** This function drives `is_image_encoder_component_name` with inputs such as `component_name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `is_image_encoder_component_name`，主要处理 `component_name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 66-67: Function `is_vae_component_name` / 函数 `is_vae_component_name`
```python
def is_vae_component_name(component_name: str) -> bool:
    return component_name in VAE_COMPONENT_NAMES
```
**EN:** This function drives `is_vae_component_name` with inputs such as `component_name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `is_vae_component_name`，主要处理 `component_name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 70-80: Function `layerwise_component_matches_selection` / 函数 `layerwise_component_matches_selection`
```python
def layerwise_component_matches_selection(
    component_name: str,
    selected_component_name: str,
) -> bool:
    """if the provided component_name (unnormalized, e.g., text_encoder_2)  matches with the selected_component_name (normalized)"""
    if selected_component_name == LAYERWISE_OFFLOAD_TEXT_ENCODER_GROUP:
        return is_text_encoder_component_name(component_name)
    if selected_component_name == LAYERWISE_OFFLOAD_VAE_GROUP:
        # `vae` is a default-policy selector; AV-side decoders remain explicit-only
        return component_name in DEFAULT_LAYERWISE_VAE_COMPONENT_NAMES
    return component_name == selected_component_name
```
**EN:** This function drives `layerwise_component_matches_selection` with inputs such as `component_name`, `selected_component_name`. if the provided component_name (unnormalized, e.g., text_encoder_2)  matches with the selected_component_name (normalized)
**CN:** 这个函数负责 `layerwise_component_matches_selection`，主要处理 `component_name`, `selected_component_name` 等输入。 文档字符串说明：if the provided component_name (unnormalized, e.g., text_encoder_2)  matches with the selected_component_name (normalized)

### Lines 83-90: Function `layerwise_component_matches_any_selection` / 函数 `layerwise_component_matches_any_selection`
```python
def layerwise_component_matches_any_selection(
    component_name: str,
    selected_component_names: Collection[str],
) -> bool:
    return any(
        layerwise_component_matches_selection(component_name, selected_component_name)
        for selected_component_name in selected_component_names
    )
```
**EN:** This function drives `layerwise_component_matches_any_selection` with inputs such as `component_name`, `selected_component_names`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `layerwise_component_matches_any_selection`，主要处理 `component_name`, `selected_component_names` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 93-121: Function `cpu_offload_flags_for_layerwise_components` / 函数 `cpu_offload_flags_for_layerwise_components`
```python
def cpu_offload_flags_for_layerwise_components(
    component_names: Sequence[str],
) -> tuple[str, ...]:
    component_names = normalize_layerwise_offload_components(component_names) or []
    if LAYERWISE_OFFLOAD_ALL_COMPONENTS in component_names:
        return CPU_OFFLOAD_FLAG_NAMES

    flag_names: list[str] = []
    if LAYERWISE_OFFLOAD_DIT_GROUP in component_names:
        flag_names.append("dit_cpu_offload")

    for component_name in component_names:
        if component_name == LAYERWISE_OFFLOAD_DIT_GROUP:
            continue
# ...
        if flag_name not in flag_names:
            flag_names.append(flag_name)

    return tuple(flag_names)
```
**EN:** This function drives `cpu_offload_flags_for_layerwise_components` with inputs such as `component_names`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `cpu_offload_flags_for_layerwise_components`，主要处理 `component_names` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 124-127: Function `expand_layerwise_offload_component_group` / 函数 `expand_layerwise_offload_component_group`
```python
def expand_layerwise_offload_component_group(component_name: str) -> tuple[str, ...]:
    if component_name == LAYERWISE_OFFLOAD_DEFAULT_GROUP:
        return LAYERWISE_OFFLOAD_DEFAULT_GROUP_COMPONENTS
    return (component_name,)
```
**EN:** This function drives `expand_layerwise_offload_component_group` with inputs such as `component_name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `expand_layerwise_offload_component_group`，主要处理 `component_name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 130-157: Function `normalize_layerwise_offload_components` / 函数 `normalize_layerwise_offload_components`
```python
def normalize_layerwise_offload_components(
    component_names: str | Sequence[str] | None,
) -> list[str] | None:
    if component_names is None:
        return None

    raw_components = (
        [component_names] if isinstance(component_names, str) else component_names
    )
    normalized_components: list[str] = []
    for raw_component in raw_components:
        if not isinstance(raw_component, str):
            raise ValueError(
                f"Invalid layerwise offload component name: {raw_component}."
# ...
                if expanded_component_name not in normalized_components:
                    normalized_components.append(expanded_component_name)

    return normalized_components or None
```
**EN:** This function drives `normalize_layerwise_offload_components` with inputs such as `component_names`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `normalize_layerwise_offload_components`，主要处理 `component_names` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- Image generation flow / 图像生成流程
- Video generation flow / 视频生成流程
- Audio generation flow / 音频生成流程
- Symbol `is_dit_component_name` anchors the module API / 符号 `is_dit_component_name` 构成该模块的核心 API
- Symbol `is_text_encoder_component_name` anchors the module API / 符号 `is_text_encoder_component_name` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc`
