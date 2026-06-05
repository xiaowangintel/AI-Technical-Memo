# quantization_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/utils/quantization_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for quantization utils in the multimodal generation stack. Key symbols include `normalize_flat_modelopt_quant_config`, `_infer_nvfp4_group_size_from_tensors`, `_resolve_quant_method_name`. / 该模块包含多模态生成体系中与 quantization utils 相关的运行时支持代码。 关键符号包括 `normalize_flat_modelopt_quant_config`, `_infer_nvfp4_group_size_from_tensors`, `_resolve_quant_method_name`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Imports and module setup / 导入与模块初始化
```python
import glob
import json
import os
import re
from pathlib import Path
from typing import Any, Dict, List, Optional

from safetensors import safe_open

from sglang.multimodal_gen.runtime.layers.quantization import (
    QuantizationConfig,
    get_quantization_config,
)
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 19-36: Function `normalize_flat_modelopt_quant_config` / 函数 `normalize_flat_modelopt_quant_config`
```python
def normalize_flat_modelopt_quant_config(
    quant_cfg: dict[str, Any] | None,
) -> dict[str, Any] | None:
    """Fill required diffusers fields for flat ModelOpt component configs."""
    if not isinstance(quant_cfg, dict) or quant_cfg.get("quant_method") != "modelopt":
        return quant_cfg

    quant_algo = str(
        quant_cfg.get("quant_algo")
        or quant_cfg.get("quantization", {}).get("quant_algo")
        or ""
    ).upper()
    if not quant_algo:
        return quant_cfg

    normalized = dict(quant_cfg)
    normalized.setdefault("quant_type", quant_algo)
    return normalized
```
**EN:** This function drives `normalize_flat_modelopt_quant_config` with inputs such as `quant_cfg`. Fill required diffusers fields for flat ModelOpt component configs.
**CN:** 这个函数负责 `normalize_flat_modelopt_quant_config`，主要处理 `quant_cfg` 等输入。 文档字符串说明：Fill required diffusers fields for flat ModelOpt component configs.

### Lines 39-67: Function `_infer_nvfp4_group_size_from_tensors` / 函数 `_infer_nvfp4_group_size_from_tensors`
```python
def _infer_nvfp4_group_size_from_tensors(weight, scale) -> Optional[int]:
    """Infer NVFP4 group_size from serialized weight/scale tensor shapes."""
    weight_shape = tuple(getattr(weight, "shape", ()))
    scale_shape = tuple(getattr(scale, "shape", ()))
    if len(weight_shape) < 2:
        return None

    input_size = int(weight_shape[1]) * 2
    if input_size <= 0:
        return None

    candidate_num_groups: list[int] = []
    if len(scale_shape) >= 2:
        candidate_num_groups.append(int(scale_shape[-1]))
# ...
        if input_size % num_groups == 0:
            return input_size // num_groups

    return None
```
**EN:** This function drives `_infer_nvfp4_group_size_from_tensors` with inputs such as `weight`, `scale`. Infer NVFP4 group_size from serialized weight/scale tensor shapes.
**CN:** 这个函数负责 `_infer_nvfp4_group_size_from_tensors`，主要处理 `weight`, `scale` 等输入。 文档字符串说明：Infer NVFP4 group_size from serialized weight/scale tensor shapes.

### Lines 70-89: Function `_resolve_quant_method_name` / 函数 `_resolve_quant_method_name`
```python
def _resolve_quant_method_name(quant_cfg: dict) -> str:
    quant_cfg = normalize_flat_modelopt_quant_config(quant_cfg) or quant_cfg
    quant_method = quant_cfg.get("quant_method")
    if quant_method != "modelopt":
        return quant_method

    quant_algo = (
        quant_cfg.get("quant_algo")
        or quant_cfg.get("quantization", {}).get("quant_algo")
        or ""
    ).upper()
    if quant_algo == "MIXED_PRECISION":
        raise ValueError(
            "ModelOpt mixed precision is not supported by the current SGLang diffusion runtime."
# ...
        return "modelopt_fp8"
    if "FP4" in quant_algo or "NVFP4" in quant_algo:
        return "modelopt_fp4"
    raise ValueError(f"Unsupported ModelOpt quant_algo for diffusion: {quant_algo}")
```
**EN:** This function drives `_resolve_quant_method_name` with inputs such as `quant_cfg`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_resolve_quant_method_name`，主要处理 `quant_cfg` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 92-96: Function `_load_quant_cls` / 函数 `_load_quant_cls`
```python
def _load_quant_cls(quant_cfg: dict):
    quant_method = _resolve_quant_method_name(quant_cfg)
    if not quant_method:
        raise ValueError("Missing quant_method in quantization config.")
    return get_quantization_config(quant_method)
```
**EN:** This function drives `_load_quant_cls` with inputs such as `quant_cfg`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_load_quant_cls`，主要处理 `quant_cfg` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 99-116: Function `find_quant_modelslim_config` / 函数 `find_quant_modelslim_config`
```python
def find_quant_modelslim_config(model_config, component_model_path):
    # Try exact name first, then glob for variant filenames (e.g. after repack)
    quant_config_file = Path(component_model_path, "quant_model_description.json")
    if not quant_config_file.is_file():
        candidates = sorted(
            Path(component_model_path).glob("quant_model_description*.json")
        )
        quant_config_file = candidates[0] if candidates else None

    quant_cfg = None
    if quant_config_file is not None and Path(quant_config_file).is_file():
        with open(quant_config_file) as f:
            quant_cfg = json.load(f)
        # This field is required for flagless model loading but is not present in
        # modelslim model description, so we're adding it here manually.
        quant_cfg["quant_method"] = "modelslim"

    return quant_cfg
```
**EN:** This function drives `find_quant_modelslim_config` with inputs such as `model_config`, `component_model_path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `find_quant_modelslim_config`，主要处理 `model_config`, `component_model_path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 119-123: Function `replace_prefix` / 函数 `replace_prefix`
```python
def replace_prefix(key: str, prefix_mapping: dict[str, str]) -> str:
    for prefix, new_prefix in prefix_mapping.items():
        if key.startswith(prefix):
            key = key.replace(prefix, new_prefix, 1)
    return key
```
**EN:** This function drives `replace_prefix` with inputs such as `key`, `prefix_mapping`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `replace_prefix`，主要处理 `key`, `prefix_mapping` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 126-198: Function `get_quant_config` / 函数 `get_quant_config`
```python
def get_quant_config(
    model_config,
    component_model_path: str,
    packed_modules_mapping: Dict[str, List[str]] = {},
    reverse_param_names_mapping: Dict[str, List[str]] = {},
    remap_prefix: Dict[str, str] | None = None,
) -> QuantizationConfig:
    quant_cfg = find_quant_modelslim_config(model_config, component_model_path)
    if quant_cfg is not None:
        quant_cls = _load_quant_cls(quant_cfg)
        return quant_cls.from_config(quant_cfg, reverse_param_names_mapping)

    if "quantization_config" not in model_config:
        return None
# ...
            ]
            config["quantization"]["exclude_modules"] = exclude_modules
        config["packed_modules_mapping"] = packed_modules_mapping
        return quant_cls.from_config(config)
```
**EN:** This function drives `get_quant_config` with inputs such as `model_config`, `component_model_path`, `packed_modules_mapping`, `reverse_param_names_mapping`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_quant_config`，主要处理 `model_config`, `component_model_path`, `packed_modules_mapping`, `reverse_param_names_mapping` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 201-208: Function `handle_fp8_metadata_format` / 函数 `handle_fp8_metadata_format`
```python
def handle_fp8_metadata_format(quant_config_dict):
    layers = quant_config_dict.get("layers", {})
    if any(
        isinstance(v, dict) and "float8" in v.get("format", "") for v in layers.values()
    ):
        quant_config_dict["quant_method"] = "fp8"
        quant_config_dict["activation_scheme"] = "dynamic"
    return quant_config_dict
```
**EN:** This function drives `handle_fp8_metadata_format` with inputs such as `quant_config_dict`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `handle_fp8_metadata_format`，主要处理 `quant_config_dict` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 211-259: Function `get_quant_config_from_safetensors_metadata` / 函数 `get_quant_config_from_safetensors_metadata`
```python
def get_quant_config_from_safetensors_metadata(
    file_path: str,
) -> Optional[QuantizationConfig]:
    """Extract quantization config from a safetensors file's metadata header.
    Returns None if no recognizable quantization metadata is found.
    """
    metadata = get_metadata_from_safetensors_file(file_path)
    if not metadata:
        return None

    quant_config_str = metadata.get("_quantization_metadata")
    quant_config_dict = None
    if quant_config_str:
        try:
# ...
        logger.debug(f"Get quantization config from safetensors file: {file_path}")
        return config
    except Exception as _e:
        return None
```
**EN:** This function drives `get_quant_config_from_safetensors_metadata` with inputs such as `file_path`. Extract quantization config from a safetensors file's metadata header.
**CN:** 这个函数负责 `get_quant_config_from_safetensors_metadata`，主要处理 `file_path` 等输入。 文档字符串说明：Extract quantization config from a safetensors file's metadata header.

### Lines 262-268: Function `get_metadata_from_safetensors_file` / 函数 `get_metadata_from_safetensors_file`
```python
def get_metadata_from_safetensors_file(file_path: str):
    try:
        with safe_open(file_path, framework="pt", device="cpu") as f:
            metadata = f.metadata()
            return metadata
    except Exception as e:
        logger.warning(e)
```
**EN:** This function drives `get_metadata_from_safetensors_file` with inputs such as `file_path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_metadata_from_safetensors_file`，主要处理 `file_path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 271-457: Function `_build_nvfp4_config_from_safetensors_files` / 函数 `_build_nvfp4_config_from_safetensors_files`
```python
def _build_nvfp4_config_from_safetensors_files(
    file_paths: list[str],
    param_names_mapping_dict: Optional[dict] = None,
    reverse_param_names_mapping_dict: Optional[dict] = None,
    fallback_group_size: Optional[int] = None,
) -> Optional[QuantizationConfig]:
    """Build a single NVFP4 config by aggregating metadata across multiple files.

    Some checkpoints split BF16 fallback layers and NVFP4 layers across multiple
    safetensors. Building the config from only the first matching file can
    incorrectly exclude layers that are quantized in a later shard.
    """
    group_size = None
    quantized_bfl_modules: set[str] = set()
# ...
            ", ".join(files_with_nvfp4_signal),
            e,
        )
        return None
```
**EN:** This function drives `_build_nvfp4_config_from_safetensors_files` with inputs such as `file_paths`, `param_names_mapping_dict`, `reverse_param_names_mapping_dict`, `fallback_group_size`. Build a single NVFP4 config by aggregating metadata across multiple files.
**CN:** 这个函数负责 `_build_nvfp4_config_from_safetensors_files`，主要处理 `file_paths`, `param_names_mapping_dict`, `reverse_param_names_mapping_dict`, `fallback_group_size` 等输入。 文档字符串说明：Build a single NVFP4 config by aggregating metadata across multiple files.

### Lines 460-472: Function `build_nvfp4_config_from_safetensors` / 函数 `build_nvfp4_config_from_safetensors`
```python
def build_nvfp4_config_from_safetensors(
    file_path: str,
    param_names_mapping_dict: Optional[dict] = None,
    reverse_param_names_mapping_dict: Optional[dict] = None,
    fallback_group_size: Optional[int] = None,
) -> Optional[QuantizationConfig]:
    """Backward-compatible wrapper for a single safetensors file."""
    return _build_nvfp4_config_from_safetensors_files(
        [file_path],
        param_names_mapping_dict,
        reverse_param_names_mapping_dict,
        fallback_group_size,
    )
```
**EN:** This function drives `build_nvfp4_config_from_safetensors` with inputs such as `file_path`, `param_names_mapping_dict`, `reverse_param_names_mapping_dict`, `fallback_group_size`. Backward-compatible wrapper for a single safetensors file.
**CN:** 这个函数负责 `build_nvfp4_config_from_safetensors`，主要处理 `file_path`, `param_names_mapping_dict`, `reverse_param_names_mapping_dict`, `fallback_group_size` 等输入。 文档字符串说明：Backward-compatible wrapper for a single safetensors file.

### Lines 475-486: Function `build_nvfp4_config_from_safetensors_list` / 函数 `build_nvfp4_config_from_safetensors_list`
```python
def build_nvfp4_config_from_safetensors_list(
    file_paths: list[str],
    param_names_mapping_dict: Optional[dict] = None,
    reverse_param_names_mapping_dict: Optional[dict] = None,
    fallback_group_size: Optional[int] = None,
) -> Optional[QuantizationConfig]:
    return _build_nvfp4_config_from_safetensors_files(
        file_paths,
        param_names_mapping_dict,
        reverse_param_names_mapping_dict,
        fallback_group_size,
    )
```
**EN:** This function drives `build_nvfp4_config_from_safetensors_list` with inputs such as `file_paths`, `param_names_mapping_dict`, `reverse_param_names_mapping_dict`, `fallback_group_size`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `build_nvfp4_config_from_safetensors_list`，主要处理 `file_paths`, `param_names_mapping_dict`, `reverse_param_names_mapping_dict`, `fallback_group_size` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- Quantization workflow / 量化工作流
- Model/component loading / 模型/组件加载
- Symbol `normalize_flat_modelopt_quant_config` anchors the module API / 符号 `normalize_flat_modelopt_quant_config` 构成该模块的核心 API
- Symbol `_infer_nvfp4_group_size_from_tensors` anchors the module API / 符号 `_infer_nvfp4_group_size_from_tensors` 构成该模块的核心 API
- Symbol `_resolve_quant_method_name` anchors the module API / 符号 `_resolve_quant_method_name` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.layers.quantization`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.loader.utils`
- **External / 外部**: `safetensors`
- **Stdlib / 标准库**: `glob`, `json`, `os`, `re`, `pathlib`, `typing`
