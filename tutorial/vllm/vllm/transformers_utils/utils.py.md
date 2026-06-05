# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements transformer utility helpers related to Utils. / [CN] 实现与 Utils 相关的 Transformers 工具辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 4-13: Imports
```python
import json
import os
import struct
from functools import cache
from os import PathLike
from pathlib import Path
from typing import Any

import vllm.envs as envs
from vllm.logger import init_logger
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as `json`, `os`, `struct`, `functools`, `pathlib`, `typing`, external APIs such as none, and internal vLLM modules such as `vllm.envs`, `vllm.logger`. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 `json`, `os`, `struct`, `functools`, `pathlib`, `typing`，外部 API 如 无，以及 vLLM 内部模块如 `vllm.envs`, `vllm.logger`。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 15-15: Module state and constants
```python
logger = init_logger(__name__)
```
**EN:** This block initializes module-level state such as `logger`. The assigned names act as shared context for the rest of the file.
**CN:** 该代码块初始化了模块级状态，例如 `logger`。这些名称为文件其余部分提供共享上下文。

### Lines 18-19: Function `is_s3`
```python
def is_s3(model_or_path: str) -> bool:
    return model_or_path.lower().startswith("s3://")
```
**EN:** This function checks whether s3 satisfies the required condition. Main inputs include `model_or_path`.
**CN:** 该函数用于判断 `is_s3` 对应的条件是否满足。 主要输入参数包括 `model_or_path`。

### Lines 22-23: Function `is_gcs`
```python
def is_gcs(model_or_path: str) -> bool:
    return model_or_path.lower().startswith("gs://")
```
**EN:** This function checks whether gcs satisfies the required condition. Main inputs include `model_or_path`.
**CN:** 该函数用于判断 `is_gcs` 对应的条件是否满足。 主要输入参数包括 `model_or_path`。

### Lines 26-27: Function `is_azure`
```python
def is_azure(model_or_path: str) -> bool:
    return model_or_path.lower().startswith("az://")
```
**EN:** This function checks whether azure satisfies the required condition. Main inputs include `model_or_path`.
**CN:** 该函数用于判断 `is_azure` 对应的条件是否满足。 主要输入参数包括 `model_or_path`。

### Lines 30-31: Function `is_cloud_storage`
```python
def is_cloud_storage(model_or_path: str) -> bool:
    return is_s3(model_or_path) or is_gcs(model_or_path) or is_azure(model_or_path)
```
**EN:** This function checks whether cloud storage satisfies the required condition. Main inputs include `model_or_path`.
**CN:** 该函数用于判断 `is_cloud_storage` 对应的条件是否满足。 主要输入参数包括 `model_or_path`。

### Lines 34-38: Function `without_trust_remote_code`
```python
def without_trust_remote_code(kwargs: dict[str, Any]) -> dict[str, Any]:
    """Return kwargs without trust_remote_code without modifying original dict."""
    if "trust_remote_code" not in kwargs:
        return kwargs
    return {k: v for k, v in kwargs.items() if k != "trust_remote_code"}
```
**EN:** This function implements `without_trust_remote_code`. The docstring states that Return kwargs without trust_remote_code without modifying original dict. Main inputs include `kwargs`.
**CN:** 该函数实现 `without_trust_remote_code` 相关逻辑。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `kwargs`。

### Lines 41-59: Function `modelscope_list_repo_files`
```python
def modelscope_list_repo_files(
    repo_id: str,
    revision: str | None = None,
    token: str | bool | None = None,
) -> list[str]:
    """List files in a modelscope repo."""
    from modelscope.hub.api import HubApi

    api = HubApi()
    api.login(token)
    # same as huggingface_hub.list_repo_files
    files = [
        file["Path"]
        for file in api.get_model_files(
            model_id=repo_id, revision=revision, recursive=True
        )
        if file["Type"] == "blob"
    ]
    return files
```
**EN:** This function implements `modelscope_list_repo_files`. The docstring states that List files in a modelscope repo. Main inputs include `repo_id`, `revision`, `token`.
**CN:** 该函数实现 `modelscope_list_repo_files` 相关逻辑。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `repo_id`, `revision`, `token`。

### Lines 62-67: Function `_maybe_json_dict`
```python
def _maybe_json_dict(path: str | PathLike) -> dict[str, str]:
    with open(path) as f:
        try:
            return json.loads(f.read())
        except Exception:
            return dict[str, str]()
```
**EN:** This private function implements `_maybe_json_dict`. Main inputs include `path`.
**CN:** 该私有函数实现 `_maybe_json_dict` 相关逻辑。 主要输入参数包括 `path`。

### Lines 70-79: Function `_maybe_space_split_dict`
```python
def _maybe_space_split_dict(path: str | PathLike) -> dict[str, str]:
    parsed_dict = dict[str, str]()
    with open(path) as f:
        for line in f.readlines():
            try:
                model_name, redirect_name = line.strip().split()
                parsed_dict[model_name] = redirect_name
            except Exception:
                pass
    return parsed_dict
```
**EN:** This private function implements `_maybe_space_split_dict`. Main inputs include `path`.
**CN:** 该私有函数实现 `_maybe_space_split_dict` 相关逻辑。 主要输入参数包括 `path`。

### Lines 83-106: Function `maybe_model_redirect`
```python
def maybe_model_redirect(model: str) -> str:
    """
    Use model_redirect to redirect the model name to a local folder.

    :param model: hf model name
    :return: maybe redirect to a local folder
    """

    model_redirect_path = envs.VLLM_MODEL_REDIRECT_PATH

    if not model_redirect_path:
        return model

    if not Path(model_redirect_path).exists():
        return model

    redirect_dict = _maybe_json_dict(model_redirect_path) or _maybe_space_split_dict(
        model_redirect_path
    )
    if redirect_model := redirect_dict.get(model):
        logger.info("model redirect: [ %s ] -> [ %s ]", model, redirect_model)
        return redirect_model

    return model
```
**EN:** This function implements `maybe_model_redirect`. The docstring states that Use model_redirect to redirect the model name to a local folder. Main inputs include `model`. Decorators such as `cache` modify caching, validation, or dispatch behavior.
**CN:** 该函数实现 `maybe_model_redirect` 相关逻辑。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `model`。 装饰器如 `cache` 会影响缓存、校验或分发行为。

### Lines 109-113: Function `parse_safetensors_file_metadata`
```python
def parse_safetensors_file_metadata(path: str | PathLike) -> dict[str, Any]:
    with open(path, "rb") as f:
        length_of_metadata = struct.unpack("<Q", f.read(8))[0]
        metadata = json.loads(f.read(length_of_metadata).decode("utf-8"))
        return metadata
```
**EN:** This function parses safetensors file metadata. Main inputs include `path`.
**CN:** 该函数负责完成 `parse_safetensors_file_metadata` 对应的核心步骤。 主要输入参数包括 `path`。

### Lines 116-123: Function `convert_model_repo_to_path`
```python
def convert_model_repo_to_path(model_repo: str) -> str:
    """When VLLM_USE_MODELSCOPE is True convert a model
    repository string to a Path str."""
    if not envs.VLLM_USE_MODELSCOPE or Path(model_repo).exists():
        return model_repo
    from modelscope.utils.file_utils import get_model_cache_root

    return os.path.join(get_model_cache_root(), model_repo)
```
**EN:** This function converts model repo to path. The docstring states that When VLLM_USE_MODELSCOPE is True convert a model Main inputs include `model_repo`.
**CN:** 该函数负责完成 `convert_model_repo_to_path` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `model_repo`。

## Key Concepts / 关键概念
- **EN:** Internal vLLM imports connect the file to runtime registries, logging, artifact lookup, and model-execution utilities.
- **CN:** vLLM 内部导入会把该文件连接到运行时注册表、日志、模型文件查找以及模型执行工具。

## Dependencies / 依赖关系
- **EN:** Standard library modules: `json`, `os`, `struct`, `functools`, `pathlib`, `typing`.
- **CN:** 标准库模块：`json`, `os`, `struct`, `functools`, `pathlib`, `typing`。
- **EN:** External packages: `modelscope.hub.api`, `modelscope.utils.file_utils`.
- **CN:** 外部依赖包：`modelscope.hub.api`, `modelscope.utils.file_utils`。
- **EN:** Internal modules: `vllm.envs`, `vllm.logger`.
- **CN:** 内部模块：`vllm.envs`, `vllm.logger`。
