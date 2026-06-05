# runai_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/runai_utils.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable runtime utilities centered on `runai_utils` and the surrounding SGLang serving stack. / 提供围绕 `runai_utils` 及其所处 SGLang 服务栈的可复用运行时工具。

## Line-by-Line Analysis / 逐行分析
### Lines 5-14: Module setup and shared state / 模块设置与共享状态
```python
import hashlib
import logging
import os
from pathlib import Path

from sglang.srt.environ import envs

logger = logging.getLogger(__name__)

SUPPORTED_SCHEMES = ["s3://", "gs://", "az://"]
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `hashlib`, `logging`, `os`, `pathlib`, `sglang.srt.environ`. It also defines symbols such as `logger`, `SUPPORTED_SCHEMES` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `hashlib`, `logging`, `os`, `pathlib`, `sglang.srt.environ`。 同时定义了 `logger`, `SUPPORTED_SCHEMES` 等符号，供后续逻辑使用。

### Lines 33-45: Function `list_safetensors` / 函数 `list_safetensors`
```python
def list_safetensors(path: str = "") -> list[str]:
    """
    List full file names from object path and filter by allow pattern.

    Args:
        path: The object storage path to list from.

    Returns:
        list[str]: List of full object storage paths allowed by the pattern
    """
    from runai_model_streamer import list_safetensors as runai_list_safetensors

    return runai_list_safetensors(path)
```
**EN:** This function implements `list_safetensors`. It primarily calls `runai_list_safetensors` to complete its work.
**CN:** 该函数实现了 `list_safetensors`。 它主要通过调用 `runai_list_safetensors` 来完成任务。

### Lines 48-50: Function `is_runai_obj_uri` / 函数 `is_runai_obj_uri`
```python
def is_runai_obj_uri(model_or_path: str | Path) -> bool:
    # Cast to str to handle pathlib.Path inputs which lack string methods (like .lower)
    return str(model_or_path).lower().startswith(tuple(SUPPORTED_SCHEMES))
```
**EN:** This function implements `is_runai_obj_uri`. It primarily calls `str.lower.startswith`, `tuple`, `str.lower`, `str` to complete its work.
**CN:** 该函数实现了 `is_runai_obj_uri`。 它主要通过调用 `str.lower.startswith`, `tuple`, `str.lower`, `str` 来完成任务。

### Lines 53-69: Class `ObjectStorageModel` declaration / 类 `ObjectStorageModel` 声明
```python
class ObjectStorageModel:
    """
    Model loader that uses Runai Model Streamer to load a model.

      Supports object storage (S3, GCS) with lazy weight streaming.

      Configuration (via load_config.model_loader_extra_config):
          - distributed (bool): Enable distributed streaming
          - concurrency (int): Number of concurrent downloads
          - memory_limit (int): Memory limit for streaming buffer

      Note: Metadata files must be pre-downloaded via
      ObjectStorageModel.download_and_get_path() before instantiation.

    Attributes:
        dir: The temporary created directory.
    """
```
**EN:** This class establishes `ObjectStorageModel` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `__enter__`, `__exit__`, `pull_files`, `download_and_get_path`, `get_path`.
**CN:** 该类将 `ObjectStorageModel` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `__enter__`, `__exit__`, `pull_files`, `download_and_get_path`, `get_path` 等方法。

### Lines 71-76: Method `ObjectStorageModel.__init__` / 方法 `ObjectStorageModel.__init__`
```python
    def __init__(self, url: str) -> None:
        self.dir = ObjectStorageModel.get_path(url)

        from runai_model_streamer import ObjectStorageModel as RunaiObjectStorageModel

        self._runai_obj = RunaiObjectStorageModel(model_path=url, dst=self.dir)
```
**EN:** This method implements `__init__` on `ObjectStorageModel`. It primarily calls `ObjectStorageModel.get_path`, `RunaiObjectStorageModel` to complete its work. State updates are written into `self.dir`, `self._runai_obj`.
**CN:** 该方法（属于 `ObjectStorageModel`）实现了 `__init__`。 它主要通过调用 `ObjectStorageModel.get_path`, `RunaiObjectStorageModel` 来完成任务。 状态更新主要写入 `self.dir`, `self._runai_obj`。

### Lines 78-79: Method `ObjectStorageModel.__enter__` / 方法 `ObjectStorageModel.__enter__`
```python
    def __enter__(self):
        return self
```
**EN:** This method implements `__enter__` on `ObjectStorageModel`.
**CN:** 该方法（属于 `ObjectStorageModel`）实现了 `__enter__`。

### Lines 81-82: Method `ObjectStorageModel.__exit__` / 方法 `ObjectStorageModel.__exit__`
```python
    def __exit__(self, exc_type, exc_val, exc_tb):
        return self._runai_obj.__exit__(exc_type, exc_val, exc_tb)
```
**EN:** This method implements `__exit__` on `ObjectStorageModel`. It primarily calls `self._runai_obj.__exit__` to complete its work.
**CN:** 该方法（属于 `ObjectStorageModel`）实现了 `__exit__`。 它主要通过调用 `self._runai_obj.__exit__` 来完成任务。

### Lines 84-95: Method `ObjectStorageModel.pull_files` / 方法 `ObjectStorageModel.pull_files`
```python
    def pull_files(
        self,
        allow_pattern: list[str] | None = None,
        ignore_pattern: list[str] | None = None,
    ) -> None:
        """Pull files from object storage into the local cache directory.

        Args:
            allow_pattern: File patterns to include (e.g. ["*.json"]).
            ignore_pattern: File patterns to exclude.
        """
        self._runai_obj.pull_files(allow_pattern, ignore_pattern)
```
**EN:** This method implements `pull_files` on `ObjectStorageModel`. It primarily calls `self._runai_obj.pull_files` to complete its work.
**CN:** 该方法（属于 `ObjectStorageModel`）实现了 `pull_files`。 它主要通过调用 `self._runai_obj.pull_files` 来完成任务。

### Lines 97-115: Method `ObjectStorageModel.download_and_get_path` / 方法 `ObjectStorageModel.download_and_get_path`
```python
    @classmethod
    def download_and_get_path(cls, model_path: str) -> str:
        """
        Downloads the model metadata (excluding heavy weights) and returns
        the local directory path. Safe for concurrent usage by multiple processes
        """
        with cls(url=model_path) as downloader:
            downloader.pull_files(
                ignore_pattern=[
                    "*.pt",
                    "*.safetensors",
                    "*.bin",
                    "*.tensors",
                    "*.pth",
                ],
            )
            cache_dir = downloader.dir
            logger.info(f"Runai Model : {cache_dir}, metadata ready.")
        return cache_dir
```
**EN:** This method implements `download_and_get_path` on `ObjectStorageModel`. It primarily calls `cls`, `downloader.pull_files`, `logger.info` to complete its work. State updates are written into `cache_dir`. The implementation relies on context-managed resources.
**CN:** 该方法（属于 `ObjectStorageModel`）实现了 `download_and_get_path`。 它主要通过调用 `cls`, `downloader.pull_files`, `logger.info` 来完成任务。 状态更新主要写入 `cache_dir`。 实现中使用了上下文管理资源。

### Lines 117-132: Method `ObjectStorageModel.get_path` / 方法 `ObjectStorageModel.get_path`
```python
    @classmethod
    def get_path(cls, model_path: str) -> str:
        """
        Returns the local directory path.
        """
        model_hash = hashlib.sha256(str(model_path).encode()).hexdigest()[:16]
        base_dir = envs.SGLANG_CACHE_DIR.get()

        # Ensure base cache dir exists
        os.makedirs(os.path.join(base_dir, "model_streamer"), exist_ok=True)

        return os.path.join(
            base_dir,
            "model_streamer",
            model_hash,
        )
```
**EN:** This method implements `get_path` on `ObjectStorageModel`. It primarily calls `envs.SGLANG_CACHE_DIR.get`, `os.makedirs`, `os.path.join`, `hashlib.sha256.hexdigest`, `hashlib.sha256`, `str.encode` to complete its work. State updates are written into `model_hash`, `base_dir`.
**CN:** 该方法（属于 `ObjectStorageModel`）实现了 `get_path`。 它主要通过调用 `envs.SGLANG_CACHE_DIR.get`, `os.makedirs`, `os.path.join`, `hashlib.sha256.hexdigest`, `hashlib.sha256`, `str.encode` 来完成任务。 状态更新主要写入 `model_hash`, `base_dir`。

## Key Concepts / 关键概念
- **Classes / 类**: `ObjectStorageModel`
- **Functions / 函数**: `list_safetensors`, `is_runai_obj_uri`, `__init__`, `__enter__`, `__exit__`, `pull_files`, `download_and_get_path`, `get_path`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.environ`
- **External / 外部依赖**: `runai_model_streamer`
- **Standard library / 标准库**: `hashlib`, `logging`, `os`, `pathlib`
