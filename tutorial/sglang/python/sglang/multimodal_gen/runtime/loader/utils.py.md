# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/loader/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module collects shared utility helpers that are reused across the multimodal generation stack. Key symbols include `set_default_torch_dtype`, `get_param_names_mapping`, `hf_to_custom_state_dict`. / 该模块汇总了多模态生成栈中可复用的通用工具函数。 关键符号包括 `set_default_torch_dtype`, `get_param_names_mapping`, `hf_to_custom_state_dict`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
"""Utilities for selecting and loading models."""

import contextlib
import glob
import os
import re
from collections import defaultdict
from collections.abc import Callable, Iterator
from typing import Any, Dict, Type

import torch
# ...
    torch.float8_e4m3fn,
    torch.float8_e5m2,
    torch.int8,
}
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 29-37: Function `set_default_torch_dtype` / 函数 `set_default_torch_dtype`
```python
@contextlib.contextmanager
def set_default_torch_dtype(dtype: torch.dtype):
    """Sets the default torch dtype to the given dtype."""
    old_dtype = torch.get_default_dtype()
    torch.set_default_dtype(dtype)
    try:
        yield
    finally:
        torch.set_default_dtype(old_dtype)
```
**EN:** This function drives `set_default_torch_dtype` with inputs such as `dtype`. Sets the default torch dtype to the given dtype.
**CN:** 这个函数负责 `set_default_torch_dtype`，主要处理 `dtype` 等输入。 文档字符串说明：Sets the default torch dtype to the given dtype.

### Lines 40-99: Function `get_param_names_mapping` / 函数 `get_param_names_mapping`
```python
def get_param_names_mapping(
    mapping_dict: dict[str, str | tuple[str, int, int]],
) -> Callable[[str], tuple[str, Any, Any]]:
    """
    Creates a mapping function that transforms parameter names using regex patterns.

    Args:
        mapping_dict (Dict[str, str]): Dictionary mapping regex patterns to replacement patterns

    Returns:
        Callable[[str], str]: A function that maps parameter names from source to target format
    """

    def mapping_fn(name: str) -> tuple[str, Any, Any]:
# ...

        return name, merge_index, total_split_params

    return mapping_fn
```
**EN:** This function drives `get_param_names_mapping` with inputs such as `mapping_dict`. Creates a mapping function that transforms parameter names using regex patterns.
**CN:** 这个函数负责 `get_param_names_mapping`，主要处理 `mapping_dict` 等输入。 文档字符串说明：Creates a mapping function that transforms parameter names using regex patterns.

### Lines 102-175: Function `hf_to_custom_state_dict` / 函数 `hf_to_custom_state_dict`
```python
def hf_to_custom_state_dict(
    hf_param_sd: dict[str, torch.Tensor] | Iterator[tuple[str, torch.Tensor]],
    param_names_mapping: Callable[[str], tuple[str, Any, Any]],
    valid_target_names: set[str] | None = None,
) -> tuple[dict[str, torch.Tensor], dict[str, tuple[str, Any, Any]]]:
    """
    Converts a Hugging Face parameter state dictionary to a custom parameter state dictionary.

    Args:
        hf_param_sd (Dict[str, torch.Tensor]): The Hugging Face parameter state dictionary
        param_names_mapping (Callable[[str], tuple[str, Any, Any]]): A function that maps parameter names from source to target format

    Returns:
        custom_param_sd (Dict[str, torch.Tensor]): The custom formatted parameter state dict
# ...
                    full_tensor.dtype,
                )
        custom_param_sd[target_param_name] = full_tensor
    return custom_param_sd, reverse_param_names_mapping
```
**EN:** This function drives `hf_to_custom_state_dict` with inputs such as `hf_param_sd`, `param_names_mapping`, `valid_target_names`. Converts a Hugging Face parameter state dictionary to a custom parameter state dictionary.
**CN:** 这个函数负责 `hf_to_custom_state_dict`，主要处理 `hf_param_sd`, `param_names_mapping`, `valid_target_names` 等输入。 文档字符串说明：Converts a Hugging Face parameter state dictionary to a custom parameter state dictionary.

### Lines 178-189: Class `skip_init_modules` / 类 `skip_init_modules`
```python
class skip_init_modules:
    def __enter__(self):
        # Save originals
        self._orig_reset = {}
        for cls in (nn.Linear, nn.Conv1d, nn.Conv2d, nn.Conv3d):
            self._orig_reset[cls] = cls.reset_parameters
            cls.reset_parameters = lambda self: None  # skip init

    def __exit__(self, exc_type, exc_value, traceback):
        # restore originals
        for cls, orig in self._orig_reset.items():
            cls.reset_parameters = orig
```
**EN:** This class models `skip_init_modules`. Important methods include `__enter__`, `__exit__`.
**CN:** 该类实现 `skip_init_modules`。 其中较重要的方法包括 `__enter__`, `__exit__`。

### Lines 192-194: Function `_normalize_component_type` / 函数 `_normalize_component_type`
```python
def _normalize_component_type(module_type: str) -> str:
    """Normalize module types like 'text_encoder_2' -> 'text_encoder'."""
    return re.sub(r"_\d+$", "", module_type)
```
**EN:** This function drives `_normalize_component_type` with inputs such as `module_type`. Normalize module types like 'text_encoder_2' -> 'text_encoder'.
**CN:** 这个函数负责 `_normalize_component_type`，主要处理 `module_type` 等输入。 文档字符串说明：Normalize module types like 'text_encoder_2' -> 'text_encoder'.

### Lines 197-206: Function `_clean_hf_config_inplace` / 函数 `_clean_hf_config_inplace`
```python
def _clean_hf_config_inplace(model_config: dict) -> None:
    """Remove common extraneous HF fields if present."""
    for key in (
        "_name_or_path",
        "transformers_version",
        "model_type",
        "tokenizer_class",
        "torch_dtype",
    ):
        model_config.pop(key, None)
```
**EN:** This function drives `_clean_hf_config_inplace` with inputs such as `model_config`. Remove common extraneous HF fields if present.
**CN:** 这个函数负责 `_clean_hf_config_inplace`，主要处理 `model_config` 等输入。 文档字符串说明：Remove common extraneous HF fields if present.

### Lines 209-240: Function `_try_redownload_missing_shards` / 函数 `_try_redownload_missing_shards`
```python
def _try_redownload_missing_shards(model_path: str, missing: list[str]) -> bool:
    """Try to re-download missing safetensors shards from HuggingFace Hub.

    Parses the repo_id and revision from the HF cache path structure
    (models--{org}--{repo}/snapshots/{revision}) and calls hf_hub_download
    for each missing shard. Returns True if all shards were recovered.
    """
    try:
        from huggingface_hub import hf_hub_download

        match = re.search(
            r"models--([^/\\]+)--([^/\\]+)[/\\]snapshots[/\\]([^/\\]+)", model_path
        )
        if not match:
# ...
        return True
    except Exception as e:
        logger.warning("Auto-repair failed: %s", e)
        return False
```
**EN:** This function drives `_try_redownload_missing_shards` with inputs such as `model_path`, `missing`. Try to re-download missing safetensors shards from HuggingFace Hub.
**CN:** 这个函数负责 `_try_redownload_missing_shards`，主要处理 `model_path`, `missing` 等输入。 文档字符串说明：Try to re-download missing safetensors shards from HuggingFace Hub.

### Lines 243-278: Function `_list_safetensors_files` / 函数 `_list_safetensors_files`
```python
def _list_safetensors_files(model_path: str) -> list[str]:
    """List all .safetensors files under a directory.

    If a safetensors index file is present, verifies that every shard listed
    in the index actually exists on disk. Missing shards are first repaired
    automatically via HuggingFace Hub (if the path is an HF cache entry);
    if repair fails a clear RuntimeError is raised.
    """
    found = sorted(glob.glob(os.path.join(str(model_path), "*.safetensors")))

    index_path = os.path.join(
        str(model_path), "diffusion_pytorch_model.safetensors.index.json"
    )
    if os.path.exists(index_path):
# ...
                    f"`huggingface-cli download {os.path.basename(model_path)}`)."
                )

    return found
```
**EN:** This function drives `_list_safetensors_files` with inputs such as `model_path`. List all .safetensors files under a directory.
**CN:** 这个函数负责 `_list_safetensors_files`，主要处理 `model_path` 等输入。 文档字符串说明：List all .safetensors files under a directory.

### Lines 279-281: Top-level configuration / 顶层配置
```python


BYTES_PER_GB = 1024**3
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 284-300: Function `get_memory_usage_of_component` / 函数 `get_memory_usage_of_component`
```python
def get_memory_usage_of_component(module) -> float | None:
    """
    returned value is in GB, rounded to 2 decimal digits
    """
    if not isinstance(module, nn.Module):
        return None
    if hasattr(module, "get_memory_footprint"):
        usage = module.get_memory_footprint() / BYTES_PER_GB
    else:
        # manually
        param_size = sum(p.numel() * p.element_size() for p in module.parameters())
        buffer_size = sum(b.numel() * b.element_size() for b in module.buffers())

        total_size_bytes = param_size + buffer_size
        usage = total_size_bytes / (1024**3)

    return round(usage, 2)
```
**EN:** This function drives `get_memory_usage_of_component` with inputs such as `module`. returned value is in GB, rounded to 2 decimal digits
**CN:** 这个函数负责 `get_memory_usage_of_component`，主要处理 `module` 等输入。 文档字符串说明：returned value is in GB, rounded to 2 decimal digits

### Lines 301-304: Top-level configuration / 顶层配置
```python


# component name ->  ComponentLoader class
component_name_to_loader_cls: Dict[str, Type[Any]] = {}
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Quantization workflow / 量化工作流
- Model/component loading / 模型/组件加载
- Caching strategy / 缓存策略
- Command-line interface / 命令行接口

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `torch`, `huggingface_hub`
- **Stdlib / 标准库**: `contextlib`, `glob`, `os`, `re`, `collections`, `collections.abc`, `typing`, `json`
