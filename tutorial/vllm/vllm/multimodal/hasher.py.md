# hasher.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/multimodal/hasher.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements hasher support for the `multimodal` portion of vLLM. / 为 vLLM 的 `multimodal` 子目录实现与 哈希 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-17)
```python
import functools

import hashlib

import pickle

import uuid

from collections.abc import Callable, Iterable

import numpy as np

import torch

from PIL import Image

import vllm.envs as envs

from vllm.logger import init_logger

from .media import MediaWithBytes
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 19-19)
```python
logger = init_logger(__name__)
```
**EN:** This constant/configuration block defines `logger`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `logger`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Function `_get_hasher_factory` (lines 23-47)
```python
def _get_hasher_factory(algorithm: str) -> Callable[[], "hashlib._Hash"]:
    """
    Get the hasher factory based on the configured algorithm.

    Args:
        algorithm: Hash algorithm name (blake3, sha256, or sha512)

    Returns a callable that creates a new hasher instance.
    Supports blake3 (default), sha256, and sha512 for FIPS compliance.

    See: https://github.com/vllm-project/vllm/issues/18334
    """
    algorithm = algorithm.lower()

    if algorithm == "blake3":
        from blake3 import blake3

        return blake3
    elif algorithm == "sha256":
        return hashlib.sha256
    elif algorithm == "sha512":
        return hashlib.sha512
    else:
        # This should never happen due to env_with_choices validation
        raise ValueError(f"Unsupported hash algorithm: {algorithm}")
```
**EN:** Function `_get_hasher_factory` computes stable identifiers or fingerprints for caching/comparison. The docstring highlights: Get the hasher factory based on the configured algorithm. Key calls such as `algorithm.lower`, `ValueError`, `functools.lru_cache` show the concrete execution path.
**CN:** Function `_get_hasher_factory` 负责计算稳定标识或指纹以支持缓存/比较。 文档字符串强调：Get the hasher factory based on the configured algorithm. 像 `algorithm.lower`, `ValueError`, `functools.lru_cache` 这样的关键调用展示了该代码块的具体执行路径。

### Class `MultiModalHasher` (lines 50-162)
```python
class MultiModalHasher:
    @classmethod
    def serialize_item(cls, obj: object) -> Iterable[bytes | memoryview]:
        # Simple cases
        if isinstance(obj, (bytes, memoryview)):
            return (obj,)
        if isinstance(obj, str):
            return (obj.encode("utf-8"),)
        if isinstance(obj, (int, float)):
            return (np.array(obj).tobytes(),)

        if isinstance(obj, Image.Image):
            exif = obj.getexif()
            if Image.ExifTags.Base.ImageID in exif and isinstance(
                exif[Image.ExifTags.Base.ImageID], uuid.UUID
            ):
                return (exif[Image.ExifTags.Base.ImageID].bytes,)

            data = {"mode": obj.mode, "data": np.asarray(obj)}
            palette = obj.palette
            if palette is not None:
                data["palette"] = palette.palette
                if palette.rawmode is not None:
    # ... omitted for brevity ...

        return hasher.hexdigest()
```
**EN:** Class `MultiModalHasher` is a structured building block in this module. Key methods include `serialize_item`, `iter_item_to_bytes`, `hash_kwargs`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `MultiModalHasher` 是该模块中的结构化构件。 关键方法包括 `serialize_item`, `iter_item_to_bytes`, `hash_kwargs`，它们共同定义初始化、校验、变换或访问模式。

### Method `MultiModalHasher.serialize_item` (lines 52-131)
```python
    def serialize_item(cls, obj: object) -> Iterable[bytes | memoryview]:
        # Simple cases
        if isinstance(obj, (bytes, memoryview)):
            return (obj,)
        if isinstance(obj, str):
            return (obj.encode("utf-8"),)
        if isinstance(obj, (int, float)):
            return (np.array(obj).tobytes(),)

        if isinstance(obj, Image.Image):
            exif = obj.getexif()
            if Image.ExifTags.Base.ImageID in exif and isinstance(
                exif[Image.ExifTags.Base.ImageID], uuid.UUID
            ):
                return (exif[Image.ExifTags.Base.ImageID].bytes,)

            data = {"mode": obj.mode, "data": np.asarray(obj)}
            palette = obj.palette
            if palette is not None:
    # ... omitted for brevity ...

        return (pickle.dumps(obj),)
```
**EN:** Method `MultiModalHasher.serialize_item` provides a reusable helper around the module's main workflow. Key calls such as `isinstance`, `obj.encode`, `np.array(obj).tobytes`, `np.array`, `obj.getexif` show the concrete execution path.
**CN:** Method `MultiModalHasher.serialize_item` 为模块主流程提供可复用的辅助逻辑。 像 `isinstance`, `obj.encode`, `np.array(obj).tobytes`, `np.array`, `obj.getexif` 这样的关键调用展示了该代码块的具体执行路径。

### Method `MultiModalHasher.iter_item_to_bytes` (lines 134-151)
```python
    def iter_item_to_bytes(
        cls,
        key: str,
        obj: object,
    ) -> Iterable[bytes | memoryview]:
        if obj is None:
            yield key.encode("utf-8")
            return
        # Recursive cases
        if isinstance(obj, (list, tuple)):
            for i, elem in enumerate(obj):
                yield from cls.iter_item_to_bytes(f"{key}.{i}", elem)
        elif isinstance(obj, dict):
            for k, v in obj.items():
                yield from cls.iter_item_to_bytes(f"{key}.{k}", v)
        else:
            yield key.encode("utf-8")
            yield from cls.serialize_item(obj)
```
**EN:** Method `MultiModalHasher.iter_item_to_bytes` provides a reusable helper around the module's main workflow. Key calls such as `key.encode`, `isinstance`, `enumerate`, `cls.iter_item_to_bytes`, `obj.items` show the concrete execution path.
**CN:** Method `MultiModalHasher.iter_item_to_bytes` 为模块主流程提供可复用的辅助逻辑。 像 `key.encode`, `isinstance`, `enumerate`, `cls.iter_item_to_bytes`, `obj.items` 这样的关键调用展示了该代码块的具体执行路径。

### Method `MultiModalHasher.hash_kwargs` (lines 154-162)
```python
    def hash_kwargs(cls, **kwargs: object) -> str:
        hasher_factory = _get_hasher_factory(envs.VLLM_MM_HASHER_ALGORITHM)
        hasher = hasher_factory()

        for k, v in sorted(kwargs.items(), key=lambda kv: kv[0]):
            for bytes_ in cls.iter_item_to_bytes(k, v):
                hasher.update(bytes_)

        return hasher.hexdigest()
```
**EN:** Method `MultiModalHasher.hash_kwargs` parses configuration, arguments, or structured metadata. Key calls such as `_get_hasher_factory`, `hasher_factory`, `sorted`, `kwargs.items`, `cls.iter_item_to_bytes` show the concrete execution path.
**CN:** Method `MultiModalHasher.hash_kwargs` 负责解析配置、参数或结构化元数据。 像 `_get_hasher_factory`, `hasher_factory`, `sorted`, `kwargs.items`, `cls.iter_item_to_bytes` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **Multimodality / 多模态**
  - **EN:** The code contains modality-specific paths for text, image, audio, or video data.
  - **CN:** 代码包含面向文本、图像、音频或视频数据的模态专用路径。
- **Caching / 缓存**
  - **EN:** Some definitions are designed to reuse computed state and avoid repeated work.
  - **CN:** 部分定义旨在复用已计算状态，避免重复工作。
- **Configuration / 配置**
  - **EN:** Configuration objects and validation rules centralize feature toggles and runtime settings.
  - **CN:** 配置对象与校验规则集中管理功能开关和运行时设置。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import functools`, `import hashlib`, `import pickle`, `import uuid`, `from collections.abc import Callable, Iterable`
- **Third-party / 第三方**: `import numpy as np`, `import torch`, `from PIL import Image`
- **vLLM internal / vLLM 内部依赖**: `import vllm.envs as envs`, `from vllm.logger import init_logger`, `from .media import MediaWithBytes`
