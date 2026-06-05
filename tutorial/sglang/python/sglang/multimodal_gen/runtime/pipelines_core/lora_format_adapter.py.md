# lora_format_adapter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/lora_format_adapter.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for lora format adapter, connecting stages, tensors, and runtime metadata. Key symbols include `LoRAFormat`, `_sample_keys`, `_has_substring_key`. / 该模块负责 lora format adapter 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `LoRAFormat`, `_sample_keys`, `_has_substring_key`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Imports and module setup / 导入与模块初始化
```python
from __future__ import annotations

import logging
from enum import Enum
from typing import Dict, Iterable, Mapping, Optional

import torch
from diffusers.loaders import lora_conversion_utils as lcu

logger = logging.getLogger("LoRAFormatAdapter")
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 13-22: Class `LoRAFormat` / 类 `LoRAFormat`
```python
class LoRAFormat(str, Enum):
    """Supported external LoRA formats before normalization."""

    STANDARD = "standard"
    NON_DIFFUSERS_SD = "non-diffusers-sd"
    QWEN_IMAGE_STANDARD = "qwen-image-standard"
    XLABS_FLUX = "xlabs-ai"
    KOHYA_FLUX = "kohya-flux"
    WAN = "wan"
    AI_TOOLKIT_FLUX = "ai-toolkit-flux"
```
**EN:** This class models `LoRAFormat` as a specialization of `str`, `Enum`. Supported external LoRA formats before normalization.
**CN:** 该类实现 `LoRAFormat`，并继承/扩展 `str`, `Enum`。 文档字符串指出：Supported external LoRA formats before normalization.

### Lines 25-31: Function `_sample_keys` / 函数 `_sample_keys`
```python
def _sample_keys(keys: Iterable[str], k: int = 20) -> list[str]:
    out = []
    for i, key in enumerate(keys):
        if i >= k:
            break
        out.append(key)
    return out
```
**EN:** This function drives `_sample_keys` with inputs such as `keys`, `k`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_sample_keys`，主要处理 `keys`, `k` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 34-35: Function `_has_substring_key` / 函数 `_has_substring_key`
```python
def _has_substring_key(keys: Iterable[str], substr: str) -> bool:
    return any(substr in k for k in keys)
```
**EN:** This function drives `_has_substring_key` with inputs such as `keys`, `substr`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_has_substring_key`，主要处理 `keys`, `substr` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 38-39: Function `_has_prefix_key` / 函数 `_has_prefix_key`
```python
def _has_prefix_key(keys: Iterable[str], prefix: str) -> bool:
    return any(k.startswith(prefix) for k in keys)
```
**EN:** This function drives `_has_prefix_key` with inputs such as `keys`, `prefix`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_has_prefix_key`，主要处理 `keys`, `prefix` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 42-57: Function `_looks_like_xlabs_flux_key` / 函数 `_looks_like_xlabs_flux_key`
```python
def _looks_like_xlabs_flux_key(k: str) -> bool:
    """XLabs FLUX-style keys under double_blocks/single_blocks with lora down/up."""
    if not (k.endswith(".down.weight") or k.endswith(".up.weight")):
        return False

    if not k.startswith(
        (
            "double_blocks.",
            "single_blocks.",
            "diffusion_model.double_blocks",
            "diffusion_model.single_blocks",
        )
    ):
        return False

    return ".processor." in k or ".proj_lora" in k or ".qkv_lora" in k
```
**EN:** This function drives `_looks_like_xlabs_flux_key` with inputs such as `k`. XLabs FLUX-style keys under double_blocks/single_blocks with lora down/up.
**CN:** 这个函数负责 `_looks_like_xlabs_flux_key`，主要处理 `k` 等输入。 文档字符串说明：XLabs FLUX-style keys under double_blocks/single_blocks with lora down/up.

### Lines 60-69: Function `_looks_like_kohya_flux` / 函数 `_looks_like_kohya_flux`
```python
def _looks_like_kohya_flux(state_dict: Mapping[str, torch.Tensor]) -> bool:
    """Kohya FLUX LoRA (flux_lora.py) under lora_unet_double/single_blocks_ prefixes."""
    if not state_dict:
        return False
    keys = state_dict.keys()
    return any(
        k.startswith("lora_unet_double_blocks_")
        or k.startswith("lora_unet_single_blocks_")
        for k in keys
    )
```
**EN:** This function drives `_looks_like_kohya_flux` with inputs such as `state_dict`. Kohya FLUX LoRA (flux_lora.py) under lora_unet_double/single_blocks_ prefixes.
**CN:** 这个函数负责 `_looks_like_kohya_flux`，主要处理 `state_dict` 等输入。 文档字符串说明：Kohya FLUX LoRA (flux_lora.py) under lora_unet_double/single_blocks_ prefixes.

### Lines 72-79: Function `_looks_like_non_diffusers_sd` / 函数 `_looks_like_non_diffusers_sd`
```python
def _looks_like_non_diffusers_sd(state_dict: Mapping[str, torch.Tensor]) -> bool:
    """Classic non-diffusers SD LoRA (Kohya/A1111/sd-scripts)."""
    if not state_dict:
        return False
    keys = state_dict.keys()
    return all(
        k.startswith(("lora_unet_", "lora_te_", "lora_te1_", "lora_te2_")) for k in keys
    )
```
**EN:** This function drives `_looks_like_non_diffusers_sd` with inputs such as `state_dict`. Classic non-diffusers SD LoRA (Kohya/A1111/sd-scripts).
**CN:** 这个函数负责 `_looks_like_non_diffusers_sd`，主要处理 `state_dict` 等输入。 文档字符串说明：Classic non-diffusers SD LoRA (Kohya/A1111/sd-scripts).

### Lines 82-95: Function `_looks_like_wan_lora` / 函数 `_looks_like_wan_lora`
```python
def _looks_like_wan_lora(state_dict: Mapping[str, torch.Tensor]) -> bool:
    """Wan2.2 distill LoRAs (Wan-AI / Wan2.2-Distill-Loras style)."""
    if not state_dict:
        return False

    for k in state_dict.keys():
        if not k.startswith("diffusion_model.blocks."):
            continue
        if ".lora_down" not in k and ".lora_up" not in k:
            continue
        if ".cross_attn." in k or ".self_attn." in k or ".ffn." in k or ".norm3." in k:
            return True

    return False
```
**EN:** This function drives `_looks_like_wan_lora` with inputs such as `state_dict`. Wan2.2 distill LoRAs (Wan-AI / Wan2.2-Distill-Loras style).
**CN:** 这个函数负责 `_looks_like_wan_lora`，主要处理 `state_dict` 等输入。 文档字符串说明：Wan2.2 distill LoRAs (Wan-AI / Wan2.2-Distill-Loras style).

### Lines 98-105: Function `_looks_like_qwen_image` / 函数 `_looks_like_qwen_image`
```python
def _looks_like_qwen_image(state_dict: Mapping[str, torch.Tensor]) -> bool:
    keys = list(state_dict.keys())
    if not keys:
        return False
    return _has_prefix_key(keys, "transformer.transformer_blocks.") and (
        _has_substring_key(keys, ".lora.down.weight")
        or _has_substring_key(keys, ".lora.up.weight")
    )
```
**EN:** This function drives `_looks_like_qwen_image` with inputs such as `state_dict`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_looks_like_qwen_image`，主要处理 `state_dict` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 108-131: Function `_looks_like_ai_toolkit_flux_lora` / 函数 `_looks_like_ai_toolkit_flux_lora`
```python
def _looks_like_ai_toolkit_flux_lora(state_dict: Mapping[str, torch.Tensor]) -> bool:
    """Detect ai-toolkit/ComfyUI trained Flux LoRA with double_blocks/single_blocks naming.

    Key patterns: double_blocks.{N}.img_attn.proj.lora_A.weight
    """
    keys = list(state_dict.keys())
    if not keys:
        return False

    has_double_blocks = any(
        k.startswith("double_blocks.")
        or k.startswith("base_model.model.double_blocks.")
        for k in keys
    )
# ...
        keys, ".lora_B"
    )

    return (has_double_blocks or has_single_blocks) and has_lora_ab
```
**EN:** This function drives `_looks_like_ai_toolkit_flux_lora` with inputs such as `state_dict`. Detect ai-toolkit/ComfyUI trained Flux LoRA with double_blocks/single_blocks naming.
**CN:** 这个函数负责 `_looks_like_ai_toolkit_flux_lora`，主要处理 `state_dict` 等输入。 文档字符串说明：Detect ai-toolkit/ComfyUI trained Flux LoRA with double_blocks/single_blocks naming.

### Lines 134-165: Function `detect_lora_format_from_state_dict` / 函数 `detect_lora_format_from_state_dict`
```python
def detect_lora_format_from_state_dict(
    state_dict: Mapping[str, torch.Tensor],
) -> LoRAFormat:
    """Classify LoRA format by key patterns only."""
    keys = list(state_dict.keys())
    if not keys:
        return LoRAFormat.STANDARD

    if _looks_like_ai_toolkit_flux_lora(state_dict):
        return LoRAFormat.AI_TOOLKIT_FLUX

    if _has_substring_key(keys, ".lora_A") or _has_substring_key(keys, ".lora_B"):
        return LoRAFormat.STANDARD

# ...
    if _has_substring_key(keys, ".lora.down") or _has_substring_key(keys, ".lora_up"):
        return LoRAFormat.NON_DIFFUSERS_SD

    return LoRAFormat.STANDARD
```
**EN:** This function drives `detect_lora_format_from_state_dict` with inputs such as `state_dict`. Classify LoRA format by key patterns only.
**CN:** 这个函数负责 `detect_lora_format_from_state_dict`，主要处理 `state_dict` 等输入。 文档字符串说明：Classify LoRA format by key patterns only.

### Lines 168-188: Function `_convert_qwen_image_standard` / 函数 `_convert_qwen_image_standard`
```python
def _convert_qwen_image_standard(
    state_dict: Mapping[str, torch.Tensor],
    log: logging.Logger,
) -> Dict[str, torch.Tensor]:
    """Qwen-Image: transformer.*.lora.down/up -> transformer_blocks.*.lora_A/B."""
    out: Dict[str, torch.Tensor] = {}

    for name, tensor in state_dict.items():
        new_name = name

        if new_name.startswith("transformer."):
            new_name = new_name[len("transformer.") :]

        if new_name.endswith(".lora.down.weight"):
# ...

        out[new_name] = tensor

    return out
```
**EN:** This function drives `_convert_qwen_image_standard` with inputs such as `state_dict`, `log`. Qwen-Image: transformer.*.lora.down/up -> transformer_blocks.*.lora_A/B.
**CN:** 这个函数负责 `_convert_qwen_image_standard`，主要处理 `state_dict`, `log` 等输入。 文档字符串说明：Qwen-Image: transformer.*.lora.down/up -> transformer_blocks.*.lora_A/B.

### Lines 191-218: Function `_convert_non_diffusers_sd_simple` / 函数 `_convert_non_diffusers_sd_simple`
```python
def _convert_non_diffusers_sd_simple(
    state_dict: Mapping[str, torch.Tensor],
    log: logging.Logger,
) -> Dict[str, torch.Tensor]:
    """Generic down/up -> A/B conversion for non-diffusers SD-like formats."""
    out: Dict[str, torch.Tensor] = {}

    for name, tensor in state_dict.items():
        new_name = name

        if "lora_down.weight" in new_name:
            new_name = new_name.replace("lora_down.weight", "lora_A.weight")
        elif "lora_up.weight" in new_name:
            new_name = new_name.replace("lora_up.weight", "lora_B.weight")
# ...
        "sample keys (<=20): %s",
        ", ".join(sample),
    )
    return out
```
**EN:** This function drives `_convert_non_diffusers_sd_simple` with inputs such as `state_dict`, `log`. Generic down/up -> A/B conversion for non-diffusers SD-like formats.
**CN:** 这个函数负责 `_convert_non_diffusers_sd_simple`，主要处理 `state_dict`, `log` 等输入。 文档字符串说明：Generic down/up -> A/B conversion for non-diffusers SD-like formats.

### Lines 221-250: Function `_convert_with_diffusers_utils_if_available` / 函数 `_convert_with_diffusers_utils_if_available`
```python
def _convert_with_diffusers_utils_if_available(
    state_dict: Mapping[str, torch.Tensor],
    log: logging.Logger,
) -> Optional[Dict[str, torch.Tensor]]:
    """Use diffusers.lora_conversion_utils if available."""
    try:
        if hasattr(lcu, "maybe_convert_state_dict"):
            converted = lcu.maybe_convert_state_dict(  # type: ignore[attr-defined]
                state_dict
            )
        else:
            converted = dict(state_dict)

        if not isinstance(converted, dict):
# ...
            "falling back to internal converters. Error: %s",
            exc,
        )
        return None
```
**EN:** This function drives `_convert_with_diffusers_utils_if_available` with inputs such as `state_dict`, `log`. Use diffusers.lora_conversion_utils if available.
**CN:** 这个函数负责 `_convert_with_diffusers_utils_if_available`，主要处理 `state_dict`, `log` 等输入。 文档字符串说明：Use diffusers.lora_conversion_utils if available.

### Lines 253-286: Function `_convert_via_diffusers_candidates` / 函数 `_convert_via_diffusers_candidates`
```python
def _convert_via_diffusers_candidates(
    state_dict: Mapping[str, torch.Tensor],
    candidate_names: tuple[str, ...],
    log: logging.Logger,
    unavailable_warning: str,
    no_converter_warning: str,
    success_info: str,
    all_failed_warning: str,
) -> Dict[str, torch.Tensor]:
    """Try multiple named converters in lora_conversion_utils, use the first that works."""
    converters = [
        (n, getattr(lcu, n)) for n in candidate_names if callable(getattr(lcu, n, None))
    ]
    if not converters:
# ...
            last_err = exc

    log.warning(all_failed_warning.format(last_err=last_err))
    return dict(state_dict)
```
**EN:** This function drives `_convert_via_diffusers_candidates` with inputs such as `state_dict`, `candidate_names`, `log`, `unavailable_warning`. Try multiple named converters in lora_conversion_utils, use the first that works.
**CN:** 这个函数负责 `_convert_via_diffusers_candidates`，主要处理 `state_dict`, `candidate_names`, `log`, `unavailable_warning` 等输入。 文档字符串说明：Try multiple named converters in lora_conversion_utils, use the first that works.

### Lines 289-314: Function `_convert_xlabs_ai_via_diffusers` / 函数 `_convert_xlabs_ai_via_diffusers`
```python
def _convert_xlabs_ai_via_diffusers(
    state_dict: Mapping[str, torch.Tensor],
    log: logging.Logger,
) -> Dict[str, torch.Tensor]:
    """Convert XLabs FLUX LoRA via diffusers helpers."""
    return _convert_via_diffusers_candidates(
        state_dict,
        (
            "_convert_xlabs_flux_lora_to_diffusers",
            "convert_xlabs_lora_state_dict_to_diffusers",
            "convert_xlabs_lora_to_diffusers",
            "convert_xlabs_flux_lora_to_diffusers",
        ),
        log=log,
# ...
            "[LoRAFormatAdapter] All XLabs FLUX converters failed; "
            "last error: {last_err}"
        ),
    )
```
**EN:** This function drives `_convert_xlabs_ai_via_diffusers` with inputs such as `state_dict`, `log`. Convert XLabs FLUX LoRA via diffusers helpers.
**CN:** 这个函数负责 `_convert_xlabs_ai_via_diffusers`，主要处理 `state_dict`, `log` 等输入。 文档字符串说明：Convert XLabs FLUX LoRA via diffusers helpers.

### Lines 317-338: Function `_convert_kohya_flux_via_diffusers` / 函数 `_convert_kohya_flux_via_diffusers`
```python
def _convert_kohya_flux_via_diffusers(
    state_dict: Mapping[str, torch.Tensor],
    log: logging.Logger,
) -> Dict[str, torch.Tensor]:
    """Convert Kohya FLUX LoRA via diffusers helpers."""
    return _convert_via_diffusers_candidates(
        state_dict,
        (
            "_convert_kohya_flux_lora_to_diffusers",
            "convert_kohya_flux_lora_to_diffusers",
        ),
        log=log,
        unavailable_warning=(
            "[LoRAFormatAdapter] Kohya FLUX detected but diffusers is unavailable."
# ...
            "[LoRAFormatAdapter] Kohya FLUX conversion failed; "
            "last error: {last_err}"
        ),
    )
```
**EN:** This function drives `_convert_kohya_flux_via_diffusers` with inputs such as `state_dict`, `log`. Convert Kohya FLUX LoRA via diffusers helpers.
**CN:** 这个函数负责 `_convert_kohya_flux_via_diffusers`，主要处理 `state_dict`, `log` 等输入。 文档字符串说明：Convert Kohya FLUX LoRA via diffusers helpers.

### Lines 341-487: Function `_convert_ai_toolkit_flux_lora` / 函数 `_convert_ai_toolkit_flux_lora`
```python
def _convert_ai_toolkit_flux_lora(
    state_dict: Mapping[str, torch.Tensor],
    log: logging.Logger,
) -> Dict[str, torch.Tensor]:
    """Convert ai-toolkit/ComfyUI trained Flux LoRA to SGLang format.

    Handles the naming convention conversion:
    - double_blocks.{N}.img_attn.qkv -> transformer_blocks.{N}.attn.to_q/k/v
    - double_blocks.{N}.txt_attn.qkv -> transformer_blocks.{N}.attn.add_q/k/v_proj
    - double_blocks.{N}.img_attn.proj -> transformer_blocks.{N}.attn.to_out.0
    - double_blocks.{N}.txt_attn.proj -> transformer_blocks.{N}.attn.to_add_out
    - double_blocks -> transformer_blocks
    - single_blocks -> single_transformer_blocks
    """
# ...
        "sample keys (<=20): %s",
        ", ".join(sample),
    )
    return final_out
```
**EN:** This function drives `_convert_ai_toolkit_flux_lora` with inputs such as `state_dict`, `log`. Convert ai-toolkit/ComfyUI trained Flux LoRA to SGLang format.
**CN:** 这个函数负责 `_convert_ai_toolkit_flux_lora`，主要处理 `state_dict`, `log` 等输入。 文档字符串说明：Convert ai-toolkit/ComfyUI trained Flux LoRA to SGLang format.

### Lines 490-536: Function `convert_lora_state_dict_by_format` / 函数 `convert_lora_state_dict_by_format`
```python
def convert_lora_state_dict_by_format(
    state_dict: Mapping[str, torch.Tensor],
    fmt: LoRAFormat,
    log: logging.Logger,
) -> Dict[str, torch.Tensor]:
    """Normalize a raw LoRA state_dict into A/B + .weight naming."""
    if fmt == LoRAFormat.QWEN_IMAGE_STANDARD:
        return _convert_qwen_image_standard(state_dict, log)

    if fmt == LoRAFormat.AI_TOOLKIT_FLUX:
        return _convert_ai_toolkit_flux_lora(state_dict, log)

    if fmt == LoRAFormat.XLABS_FLUX:
        converted = _convert_xlabs_ai_via_diffusers(state_dict, log)
# ...
        "[LoRAFormatAdapter] format %s not handled specially, returning as-is",
        fmt,
    )
    return dict(state_dict)
```
**EN:** This function drives `convert_lora_state_dict_by_format` with inputs such as `state_dict`, `fmt`, `log`. Normalize a raw LoRA state_dict into A/B + .weight naming.
**CN:** 这个函数负责 `convert_lora_state_dict_by_format`，主要处理 `state_dict`, `fmt`, `log` 等输入。 文档字符串说明：Normalize a raw LoRA state_dict into A/B + .weight naming.

### Lines 539-569: Function `normalize_lora_state_dict` / 函数 `normalize_lora_state_dict`
```python
def normalize_lora_state_dict(
    state_dict: Mapping[str, torch.Tensor],
    logger: Optional[logging.Logger] = None,
) -> Dict[str, torch.Tensor]:
    """Normalize any supported LoRA format into a single canonical layout."""
    log = logger or globals()["logger"]

    keys = list(state_dict.keys())
    log.info(
        "[LoRAFormatAdapter] normalize_lora_state_dict called, #keys=%d",
        len(keys),
    )
    if keys:
        log.info(
# ...
            ", ".join(_sample_keys(norm_keys, 20)),
        )

    return normalized
```
**EN:** This function drives `normalize_lora_state_dict` with inputs such as `state_dict`, `logger`. Normalize any supported LoRA format into a single canonical layout.
**CN:** 这个函数负责 `normalize_lora_state_dict`，主要处理 `state_dict`, `logger` 等输入。 文档字符串说明：Normalize any supported LoRA format into a single canonical layout.

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Model/component loading / 模型/组件加载
- Pipeline orchestration / 流水线编排
- Image generation flow / 图像生成流程
- LoRA adaptation support / LoRA 适配支持

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `torch`, `diffusers.loaders`
- **Stdlib / 标准库**: `logging`, `enum`, `typing`
