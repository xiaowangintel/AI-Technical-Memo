# sharded_state_loader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/model_loader/sharded_state_loader.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements sharded state loader support for the `model_loader` portion of vLLM. / 为 vLLM 的 `model_loader` 子目录实现与 shardedstate加载器 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-24)
```python
import collections

import glob

import os

import time

from collections.abc import Generator

from copy import copy

from typing import Any

import torch

from torch import nn

from vllm.config import ModelConfig

from vllm.config.load import LoadConfig

from vllm.logger import init_logger

from vllm.model_executor.model_loader.base_loader import BaseModelLoader

from vllm.model_executor.model_loader.weight_utils import (
    download_weights_from_hf,
    runai_safetensors_weights_iterator,
)

from vllm.transformers_utils.s3_utils import glob as s3_glob

from vllm.transformers_utils.utils import is_s3
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 26-26)
```python
logger = init_logger(__name__)
```
**EN:** This constant/configuration block defines `logger`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `logger`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Class `ShardedStateLoader` (lines 29-214)
```python
class ShardedStateLoader(BaseModelLoader):
    """
    Model loader that directly loads each worker's model state dict, which
    enables a fast load path for large tensor-parallel models where each worker
    only needs to read its own shard rather than the entire checkpoint. See
    `examples/features/sharded_state/save_sharded_state_offline.py` for creating
    a sharded checkpoint.
    """

    DEFAULT_PATTERN = "model-rank-{rank}-part-{part}.safetensors"

    def __init__(self, load_config: LoadConfig):
        super().__init__(load_config)

        extra_config = (
            {}
            if load_config.model_loader_extra_config is None
            else copy(load_config.model_loader_extra_config)
        )
        self.pattern = extra_config.pop("pattern", self.DEFAULT_PATTERN)
        if extra_config:
            raise ValueError(
                f"Unexpected extra config keys for load format "
    # ... omitted for brevity ...
                os.path.join(path, filename),
            )
```
**EN:** Class `ShardedStateLoader` is a structured building block in this module. It inherits from `BaseModelLoader`. Key methods include `__init__`, `_filter_subtensors`, `_prepare_weights`, `download_model`, `load_weights`, `iterate_over_files`, which define initialization, validation, transformation, or access patterns. The class docstring says: Model loader that directly loads each worker's model state dict, which enables a fast load path for large tensor-parallel models where each worker only needs to read its own sha...
**CN:** 类 `ShardedStateLoader` 是该模块中的结构化构件，继承自 `BaseModelLoader`。 关键方法包括 `__init__`, `_filter_subtensors`, `_prepare_weights`, `download_model`, `load_weights`, `iterate_over_files`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Model loader that directly loads each worker's model state dict, which enables a fast load path for large tensor-parallel models where each worker only needs to read its own sha...

### Method `ShardedStateLoader.__init__` (lines 40-54)
```python
    def __init__(self, load_config: LoadConfig):
        super().__init__(load_config)

        extra_config = (
            {}
            if load_config.model_loader_extra_config is None
            else copy(load_config.model_loader_extra_config)
        )
        self.pattern = extra_config.pop("pattern", self.DEFAULT_PATTERN)
        if extra_config:
            raise ValueError(
                f"Unexpected extra config keys for load format "
                f"{load_config.load_format}: "
                f"{load_config.model_loader_extra_config.keys()}"
            )
```
**EN:** Method `ShardedStateLoader.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `super().__init__`, `super`, `copy`, `extra_config.pop`, `ValueError` show the concrete execution path.
**CN:** Method `ShardedStateLoader.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `super().__init__`, `super`, `copy`, `extra_config.pop`, `ValueError` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ShardedStateLoader._filter_subtensors` (lines 57-92)
```python
    def _filter_subtensors(
        tensors: dict[str, torch.Tensor],
    ) -> dict[str, torch.Tensor]:
        """
        Filter out all tensors that share the same memory or a subset of the
        memory of another tensor.
        """
        same_storage_groups: dict[Any, list[tuple[str, torch.Tensor]]] = (
            collections.defaultdict(list)
        )
        for key, tensor in tensors.items():
            if tensor.numel():
                ptr = tensor.untyped_storage().data_ptr()
                same_storage_groups[tensor.device, ptr].append((key, tensor))

        def get_end_ptr(tensor: torch.Tensor) -> int:
            return tensor.view(-1)[-1].data_ptr() + tensor.element_size()

        result: dict[str, torch.Tensor] = {}
    # ... omitted for brevity ...
                    result[k] = t
        return result
```
**EN:** Method `ShardedStateLoader._filter_subtensors` provides a reusable helper around the module's main workflow. The docstring highlights: Filter out all tensors that share the same memory or a subset of the memory of another tensor. Key calls such as `collections.defaultdict`, `tensors.items`, `tensor.numel`, `tensor.untyped_storage().data_ptr`, `tensor.untyped_storage` show the concrete execution path.
**CN:** Method `ShardedStateLoader._filter_subtensors` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Filter out all tensors that share the same memory or a subset of the memory of another tensor. 像 `collections.defaultdict`, `tensors.items`, `tensor.numel`, `tensor.untyped_storage().data_ptr`, `tensor.untyped_storage` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ShardedStateLoader.download_model` (lines 107-108)
```python
    def download_model(self, model_config: ModelConfig) -> None:
        self._prepare_weights(model_config.model, model_config.revision)
```
**EN:** Method `ShardedStateLoader.download_model` handles loading or retrieval of external/internal data. Key calls such as `self._prepare_weights` show the concrete execution path.
**CN:** Method `ShardedStateLoader.download_model` 负责加载或获取外部/内部数据。 像 `self._prepare_weights` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ShardedStateLoader.load_weights` (lines 110-162)
```python
    def load_weights(self, model: nn.Module, model_config: ModelConfig) -> None:
        from vllm.distributed import get_tensor_model_parallel_rank

        model_weights = model_config.model
        if model_weights_override := model_config.model_weights:
            model_weights = model_weights_override
        local_model_path = model_weights

        rank = get_tensor_model_parallel_rank()
        pattern = os.path.join(
            local_model_path,
            self.pattern.format(rank=rank, part="*"),
        )

        filepaths = []
        if is_s3(local_model_path):
            file_pattern = f"*{self.pattern.format(rank=rank, part='*')}"
            filepaths = s3_glob(path=local_model_path, allow_pattern=[file_pattern])
        else:
    # ... omitted for brevity ...
        if state_dict:
            raise ValueError(f"Missing keys {tuple(state_dict)} in loaded state!")
```
**EN:** Method `ShardedStateLoader.load_weights` handles loading or retrieval of external/internal data. Key calls such as `get_tensor_model_parallel_rank`, `os.path.join`, `self.pattern.format`, `is_s3`, `s3_glob` show the concrete execution path.
**CN:** Method `ShardedStateLoader.load_weights` 负责加载或获取外部/内部数据。 像 `get_tensor_model_parallel_rank`, `os.path.join`, `self.pattern.format`, `is_s3`, `s3_glob` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ShardedStateLoader.iterate_over_files` (lines 164-176)
```python
    def iterate_over_files(
        self, paths
    ) -> Generator[tuple[str, torch.Tensor], None, None]:
        if self.load_config.load_format == "runai_streamer_sharded":
            yield from runai_safetensors_weights_iterator(paths, True)
        else:
            from safetensors.torch import safe_open

            for path in paths:
                with safe_open(path, framework="pt") as f:
                    for key in f.keys():  # noqa: SIM118
                        tensor = f.get_tensor(key)
                        yield key, tensor
```
**EN:** Method `ShardedStateLoader.iterate_over_files` provides a reusable helper around the module's main workflow. Key calls such as `runai_safetensors_weights_iterator`, `safe_open`, `f.keys`, `f.get_tensor` show the concrete execution path.
**CN:** Method `ShardedStateLoader.iterate_over_files` 为模块主流程提供可复用的辅助逻辑。 像 `runai_safetensors_weights_iterator`, `safe_open`, `f.keys`, `f.get_tensor` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ShardedStateLoader.save_model` (lines 179-214)
```python
    def save_model(
        model: torch.nn.Module,
        path: str,
        pattern: str | None = None,
        max_size: int | None = None,
    ) -> None:
        from safetensors.torch import save_file

        from vllm.distributed import get_tensor_model_parallel_rank

        if pattern is None:
            pattern = ShardedStateLoader.DEFAULT_PATTERN
        rank = get_tensor_model_parallel_rank()
        part_idx = 0
        total_size = 0
        state_dict = ShardedStateLoader._filter_subtensors(model.state_dict())
        state_dict_part: dict[str, torch.Tensor] = {}
        for key, tensor in state_dict.items():
            param_size = tensor.nelement() * tensor.element_size()
    # ... omitted for brevity ...
                os.path.join(path, filename),
            )
```
**EN:** Method `ShardedStateLoader.save_model` serializes data and writes it to a target representation. Key calls such as `get_tensor_model_parallel_rank`, `ShardedStateLoader._filter_subtensors`, `model.state_dict`, `state_dict.items`, `tensor.nelement` show the concrete execution path.
**CN:** Method `ShardedStateLoader.save_model` 负责序列化数据并写入目标表示。 像 `get_tensor_model_parallel_rank`, `ShardedStateLoader._filter_subtensors`, `model.state_dict`, `state_dict.items`, `tensor.nelement` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import collections`, `import glob`, `import os`, `import time`, `from collections.abc import Generator`, `from copy import copy`, `from typing import Any`
- **Third-party / 第三方**: `import torch`, `from torch import nn`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import ModelConfig`, `from vllm.config.load import LoadConfig`, `from vllm.logger import init_logger`, `from vllm.model_executor.model_loader.base_loader import BaseModelLoader`, `from vllm.model_executor.model_loader.weight_utils import download_weights_from_hf, runai_safetensors_weights_iterator`, `from vllm.transformers_utils.s3_utils import glob as s3_glob`, `from vllm.transformers_utils.utils import is_s3`
