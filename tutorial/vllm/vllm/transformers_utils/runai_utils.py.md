# runai_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/runai_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements transformer utility helpers related to Runai Utils. / [CN] 实现与 Runai Utils 相关的 Transformers 工具辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 4-12: Imports
```python
import hashlib
import os
import shutil
import signal

from vllm import envs
from vllm.assets.base import get_cache_dir
from vllm.logger import init_logger
from vllm.utils.import_utils import PlaceholderModule
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as `hashlib`, `os`, `shutil`, `signal`, external APIs such as none, and internal vLLM modules such as `vllm`, `vllm.assets.base`, `vllm.logger`, `vllm.utils.import_utils`. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 `hashlib`, `os`, `shutil`, `signal`，外部 API 如 无，以及 vLLM 内部模块如 `vllm`, `vllm.assets.base`, `vllm.logger`, `vllm.utils.import_utils`。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 14-16: Module state and constants
```python
logger = init_logger(__name__)

SUPPORTED_SCHEMES = ["s3://", "gs://", "az://"]
```
**EN:** This block defines module-level constants/defaults such as `logger`, `SUPPORTED_SCHEMES`. They centralize shared parameters for later helpers and classes.
**CN:** 该代码块定义了模块级常量或默认值，例如 `logger`, `SUPPORTED_SCHEMES`。它们把后续辅助函数和类会复用的参数集中到一起。

### Lines 18-24: Try/except block
```python
try:
    from runai_model_streamer import list_safetensors as runai_list_safetensors
    from runai_model_streamer import pull_files as runai_pull_files
except ImportError:
    runai_model_streamer = PlaceholderModule("runai_model_streamer")  # type: ignore[assignment]
    runai_pull_files = runai_model_streamer.placeholder_attr("pull_files")
    runai_list_safetensors = runai_model_streamer.placeholder_attr("list_safetensors")
```
**EN:** This `try`/`except` block provides an import-time compatibility fallback, usually to bridge optional dependencies or Transformers-version differences.
**CN:** 该 `try`/`except` 代码块提供导入时的兼容性回退，通常用于处理可选依赖或 Transformers 版本差异。

### Lines 27-37: Function `list_safetensors`
```python
def list_safetensors(path: str = "") -> list[str]:
    """
    List full file names from object path and filter by allow pattern.

    Args:
        path: The object storage path to list from.

    Returns:
        list[str]: List of full object storage paths allowed by the pattern
    """
    return runai_list_safetensors(path)
```
**EN:** This function implements `list_safetensors`. The docstring states that List full file names from object path and filter by allow pattern. Main inputs include `path`.
**CN:** 该函数实现 `list_safetensors` 相关逻辑。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `path`。

### Lines 40-41: Function `is_runai_obj_uri`
```python
def is_runai_obj_uri(model_or_path: str) -> bool:
    return model_or_path.lower().startswith(tuple(SUPPORTED_SCHEMES))
```
**EN:** This function checks whether runai obj uri satisfies the required condition. Main inputs include `model_or_path`.
**CN:** 该函数用于判断 `is_runai_obj_uri` 对应的条件是否满足。 主要输入参数包括 `model_or_path`。

### Lines 44-100: Class `ObjectStorageModel`
```python
class ObjectStorageModel:
    """
    A class representing an ObjectStorage model mirrored into a
    temporary directory.

    Attributes:
        dir: The temporary created directory.

    Methods:
        pull_files(): Pull model from object storage to the temporary directory.
    """

    def __init__(self, url: str) -> None:
        if envs.VLLM_ASSETS_CACHE_MODEL_CLEAN:
            for sig in (signal.SIGINT, signal.SIGTERM):
                existing_handler = signal.getsignal(sig)
                signal.signal(sig, self._close_by_signal(existing_handler))

        dir_name = os.path.join(
            get_cache_dir(),
            "model_streamer",
            hashlib.sha256(str(url).encode()).hexdigest()[:8],
        )
        os.makedirs(dir_name, exist_ok=True)
        self.dir = dir_name
        logger.debug("Init object storage, model cache path is: %s", dir_name)

    def _close(self) -> None:
        if os.path.exists(self.dir):
            shutil.rmtree(self.dir)

    def _close_by_signal(self, existing_handler=None):
        def new_handler(signum, frame):
            self._close()
            if existing_handler:
                existing_handler(signum, frame)

        return new_handler

    def pull_files(
        self,
        model_path: str = "",
        allow_pattern: list[str] | None = None,
        ignore_pattern: list[str] | None = None,
    ) -> None:
        """
        Pull files from object storage into the temporary directory.

        Args:
            model_path: The object storage path of the model.
            allow_pattern: A list of patterns of which files to pull.
            ignore_pattern: A list of patterns of which files not to pull.

        """
        if not model_path.endswith("/"):
            model_path = model_path + "/"
        runai_pull_files(model_path, self.dir, allow_pattern, ignore_pattern)
```
**EN:** Defines class `ObjectStorageModel`. The class docstring highlights that A class representing an ObjectStorage model mirrored into a Key methods include `__init__`, `_close`, `_close_by_signal`, `pull_files`.
**CN:** 定义类 `ObjectStorageModel`。 类文档字符串进一步说明了它的职责。 关键方法包括 `__init__`, `_close`, `_close_by_signal`, `pull_files`。

## Key Concepts / 关键概念
- **EN:** Internal vLLM imports connect the file to runtime registries, logging, artifact lookup, and model-execution utilities.
- **CN:** vLLM 内部导入会把该文件连接到运行时注册表、日志、模型文件查找以及模型执行工具。

## Dependencies / 依赖关系
- **EN:** Standard library modules: `hashlib`, `os`, `shutil`, `signal`.
- **CN:** 标准库模块：`hashlib`, `os`, `shutil`, `signal`。
- **EN:** External packages: `runai_model_streamer`.
- **CN:** 外部依赖包：`runai_model_streamer`。
- **EN:** Internal modules: `vllm`, `vllm.assets.base`, `vllm.logger`, `vllm.utils.import_utils`.
- **CN:** 内部模块：`vllm`, `vllm.assets.base`, `vllm.logger`, `vllm.utils.import_utils`。
