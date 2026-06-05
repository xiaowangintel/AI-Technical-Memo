# hf_storage.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/checkpoint/hf_storage.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed checkpointing, planners, and storage helpers. Its main entry points include HuggingFaceStorageWriter, HuggingFaceStorageReader.
- **用途 (CN)**: 该模块聚焦于分布式检查点、规划器与存储辅助逻辑，其主要入口包括 HuggingFaceStorageWriter, HuggingFaceStorageReader。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs
import dataclasses
import json
import logging
import queue
import threading
from typing import Any

import torch
from torch.distributed.checkpoint import FileSystemReader, FileSystemWriter
from torch.distributed.checkpoint._consolidate_hf_safetensors import (
    consolidate_safetensors_files,
)
from torch.distributed.checkpoint._hf_utils import (
    _gen_file_name,
    _HFStorageInfo,
    _metadata_fn,
    CUSTOM_METADATA_KEY,
    SAVED_OFFSETS_KEY,
    SHARDED_DIR_NAME,
````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Imports module dependencies: `dataclasses`. | CN: 导入模块依赖：`dataclasses`。
- **L3** EN: Imports module dependencies: `json`. | CN: 导入模块依赖：`json`。
- **L4** EN: Imports module dependencies: `logging`. | CN: 导入模块依赖：`logging`。
- **L5** EN: Imports module dependencies: `queue`. | CN: 导入模块依赖：`queue`。
- **L6** EN: Imports module dependencies: `threading`. | CN: 导入模块依赖：`threading`。
- **L7** EN: Imports selected names from `typing`. | CN: 从 `typing` 导入指定名称。
- **L8** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L9** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L10** EN: Imports selected names from `torch.distributed.checkpoint`. | CN: 从 `torch.distributed.checkpoint` 导入指定名称。
- **L11** EN: Imports selected names from `torch.distributed.checkpoint._consolidate_hf_safetensors`. | CN: 从 `torch.distributed.checkpoint._consolidate_hf_safetensors` 导入指定名称。
- **L12** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L13** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L14** EN: Imports selected names from `torch.distributed.checkpoint._hf_utils`. | CN: 从 `torch.distributed.checkpoint._hf_utils` 导入指定名称。
- **L15** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L16** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L17** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L18** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L19** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L20** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。

### Lines 21-40 / 第 21-40 行

````python
    SUFFIX,
)
from torch.distributed.checkpoint.filesystem import SerializationFormat
from torch.distributed.checkpoint.metadata import (
    ChunkStorageMetadata,
    Metadata,
    MetadataIndex,
    StorageMeta,
    TensorProperties,
    TensorStorageMetadata,
)
from torch.distributed.checkpoint.planner import (
    LoadPlan,
    LoadPlanner,
    ReadItem,
    SavePlan,
    SavePlanner,
    WriteItem,
)
from torch.distributed.checkpoint.storage import WriteResult
````

- **L21** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L22** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L23** EN: Imports selected names from `torch.distributed.checkpoint.filesystem`. | CN: 从 `torch.distributed.checkpoint.filesystem` 导入指定名称。
- **L24** EN: Imports selected names from `torch.distributed.checkpoint.metadata`. | CN: 从 `torch.distributed.checkpoint.metadata` 导入指定名称。
- **L25** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L26** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L27** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L28** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L29** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L30** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L31** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L32** EN: Imports selected names from `torch.distributed.checkpoint.planner`. | CN: 从 `torch.distributed.checkpoint.planner` 导入指定名称。
- **L33** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L34** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L35** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L36** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L37** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L38** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L39** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L40** EN: Imports selected names from `torch.distributed.checkpoint.storage`. | CN: 从 `torch.distributed.checkpoint.storage` 导入指定名称。

### Lines 41-60 / 第 41-60 行

````python
from torch.futures import Future


logger: logging.Logger = logging.getLogger(__name__)

__all__ = ["HuggingFaceStorageWriter", "HuggingFaceStorageReader"]


class HuggingFaceStorageWriter(FileSystemWriter):
    """
    A writer that writes to storage in the huggingface safetensors format.
    """

    def __init__(
        self,
        path: str,
        fqn_to_index_mapping: dict[str, int] | None = None,
        thread_count: int = 1,
        save_distributed: bool = False,
        enable_consolidation: bool = False,
````

- **L41** EN: Imports selected names from `torch.futures`. | CN: 从 `torch.futures` 导入指定名称。
- **L42** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L43** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L44** EN: Assigns or updates `logger`. | CN: 对 `logger` 进行赋值或更新。
- **L45** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L46** EN: Assigns or updates `__all__`. | CN: 对 `__all__` 进行赋值或更新。
- **L47** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L48** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L49** EN: Defines class `HuggingFaceStorageWriter`. | CN: 定义类 `HuggingFaceStorageWriter`。
- **L50** EN: Starts the docstring for the class HuggingFaceStorageWriter. | CN: 开始定义 class HuggingFaceStorageWriter 的文档字符串。
- **L51** EN: Continues the docstring text for the class HuggingFaceStorageWriter. | CN: 继续补充 class HuggingFaceStorageWriter 的文档字符串内容。
- **L52** EN: Closes the docstring for the class HuggingFaceStorageWriter. | CN: 结束 class HuggingFaceStorageWriter 的文档字符串。
- **L53** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L54** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L55** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L56** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L57** EN: Assigns or updates `fqn_to_index_mapping`. | CN: 对 `fqn_to_index_mapping` 进行赋值或更新。
- **L58** EN: Assigns or updates `thread_count`. | CN: 对 `thread_count` 进行赋值或更新。
- **L59** EN: Assigns or updates `save_distributed`. | CN: 对 `save_distributed` 进行赋值或更新。
- **L60** EN: Assigns or updates `enable_consolidation`. | CN: 对 `enable_consolidation` 进行赋值或更新。

### Lines 61-80 / 第 61-80 行

````python
        thread_count_consolidation: int = 1,
    ) -> None:
        """
        Initialize the huggingface writer pointing to path.

        Args:
            path: directory where the checkpoint will be read from.
            fqn_to_index_mapping: A mapping from tensor FQN to the index of the file that the tensor should be written to.
                              Indices are from 1 to N, where N is the number of files. If not provided,
                              the tensors will be written to a single file. If none, then all the tensors on the
                              same rank will be written to the same file.
            thread_count: Number of threads to use to write distributed checkpoint. Default to 1.
            save_distributed: If True, save the checkpoint using distributed APIs where every rank saves its own shard.
                        Default is False which assumes rank-0 checkpointing of the full state_dict.
            enable_consolidation: If True, consolidate the sharded checkpoint after saving. The sharded tensors will be
                                saved to path/sharded and the full tensors will be saved to path. Default to False.
            thread_count_consolidation: Number of threads to use for parallel processing of saving data
                                to consolidated output files. Default to 1.
        """

````

- **L61** EN: Assigns or updates `thread_count_consolidation`. | CN: 对 `thread_count_consolidation` 进行赋值或更新。
- **L62** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L63** EN: Starts the docstring for the function __init__. | CN: 开始定义 function __init__ 的文档字符串。
- **L64** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L65** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L66** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L67** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L68** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L69** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L70** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L71** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L72** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L73** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L74** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L75** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L76** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L77** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L78** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L79** EN: Closes the docstring for the function __init__. | CN: 结束 function __init__ 的文档字符串。
- **L80** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 81-100 / 第 81-100 行

````python
        super().__init__(
            path=path,
            serialization_format=SerializationFormat.SAFETENSORS,
            thread_count=thread_count,
        )
        self.fqn_to_index_mapping: dict[str, int] | None = fqn_to_index_mapping
        self.save_distributed: bool = save_distributed
        self.enable_consolidation: bool = enable_consolidation
        self.consolidated_output_path: str | None = None
        if self.enable_consolidation:
            self.consolidated_output_path = str(self.path)
            self.path = self.fs.concat_path(self.path, SHARDED_DIR_NAME)
        self.thread_count_consolidation = thread_count_consolidation

    def prepare_global_plan(self, plans: list[SavePlan]) -> list[SavePlan]:
        new_plans = []
        for i, plan in enumerate(plans, start=1):
            storage_data: dict[str, Any] = {}
            if self.fqn_to_index_mapping is not None:
                storage_data["fqn_to_index_mapping"] = self.fqn_to_index_mapping
````

- **L81** EN: Calls `super` as part of the current workflow. | CN: 在当前流程中调用 `super`。
- **L82** EN: Assigns or updates `path`. | CN: 对 `path` 进行赋值或更新。
- **L83** EN: Assigns or updates `serialization_format`. | CN: 对 `serialization_format` 进行赋值或更新。
- **L84** EN: Assigns or updates `thread_count`. | CN: 对 `thread_count` 进行赋值或更新。
- **L85** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L86** EN: Assigns or updates `self.fqn_to_index_mapping`. | CN: 对 `self.fqn_to_index_mapping` 进行赋值或更新。
- **L87** EN: Assigns or updates `self.save_distributed`. | CN: 对 `self.save_distributed` 进行赋值或更新。
- **L88** EN: Assigns or updates `self.enable_consolidation`. | CN: 对 `self.enable_consolidation` 进行赋值或更新。
- **L89** EN: Assigns or updates `self.consolidated_output_path`. | CN: 对 `self.consolidated_output_path` 进行赋值或更新。
- **L90** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L91** EN: Assigns or updates `self.consolidated_output_path`. | CN: 对 `self.consolidated_output_path` 进行赋值或更新。
- **L92** EN: Assigns or updates `self.path`. | CN: 对 `self.path` 进行赋值或更新。
- **L93** EN: Assigns or updates `self.thread_count_consolidation`. | CN: 对 `self.thread_count_consolidation` 进行赋值或更新。
- **L94** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L95** EN: Defines function `prepare_global_plan`. | CN: 定义函数 `prepare_global_plan`。
- **L96** EN: Assigns or updates `new_plans`. | CN: 对 `new_plans` 进行赋值或更新。
- **L97** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L98** EN: Assigns or updates `storage_data`. | CN: 对 `storage_data` 进行赋值或更新。
- **L99** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L100** EN: Continues the implementation inside function `prepare_global_plan`. | CN: 继续说明函数 `prepare_global_plan` 内部的实现。

### Lines 101-120 / 第 101-120 行

````python
            if self.save_distributed:
                storage_data["shard_index"] = i

            new_plans.append(dataclasses.replace(plan, storage_data=storage_data))

        return new_plans

    def write_data(
        self,
        plan: SavePlan,
        planner: SavePlanner,
    ) -> Future[list[WriteResult]]:
        if len(plan.items) == 0:
            fut: Future = Future()
            fut.set_result([])
            return fut

        # storage_plan is a map from key to file index
        storage_data: dict[str, Any] = plan.storage_data
        storage_plan: dict[str, int] | None = None
````

- **L101** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L102** EN: Continues the implementation inside function `prepare_global_plan`. | CN: 继续说明函数 `prepare_global_plan` 内部的实现。
- **L103** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L104** EN: Calls `new_plans.append` as part of the current workflow. | CN: 在当前流程中调用 `new_plans.append`。
- **L105** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L106** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L107** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L108** EN: Defines function `write_data`. | CN: 定义函数 `write_data`。
- **L109** EN: Continues the implementation inside function `write_data`. | CN: 继续说明函数 `write_data` 内部的实现。
- **L110** EN: Continues the implementation inside function `write_data`. | CN: 继续说明函数 `write_data` 内部的实现。
- **L111** EN: Continues the implementation inside function `write_data`. | CN: 继续说明函数 `write_data` 内部的实现。
- **L112** EN: Continues the implementation inside function `write_data`. | CN: 继续说明函数 `write_data` 内部的实现。
- **L113** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L114** EN: Assigns or updates `fut`. | CN: 对 `fut` 进行赋值或更新。
- **L115** EN: Calls `fut.set_result` as part of the current workflow. | CN: 在当前流程中调用 `fut.set_result`。
- **L116** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L117** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L118** EN: Keeps the inline comment or directive: storage_plan is a map from key to file index | CN: 保留这一行注释或指令：storage_plan is a map from key to file index
- **L119** EN: Assigns or updates `storage_data`. | CN: 对 `storage_data` 进行赋值或更新。
- **L120** EN: Assigns or updates `storage_plan`. | CN: 对 `storage_plan` 进行赋值或更新。

### Lines 121-140 / 第 121-140 行

````python
        shard_index: int | None = None
        if "fqn_to_index_mapping" in storage_data:
            storage_plan = storage_data["fqn_to_index_mapping"]
        if "shard_index" in storage_data:
            shard_index = storage_data["shard_index"]

        buckets = self._split_by_storage_plan(storage_plan, plan.items)
        highest_index = max(storage_plan.values()) if storage_plan is not None else 1

        file_queue: queue.Queue = queue.Queue()
        for file_index, write_items in buckets.items():
            file_name = _gen_file_name(file_index, highest_index, shard_index)
            file_queue.put(
                (self.fs.concat_path(self.path, file_name), file_name, write_items)
            )

        return super()._write_data(planner, file_queue)

    def finish(self, metadata: Metadata, results: list[list[WriteResult]]) -> None:
        if self.save_distributed and not self.enable_consolidation:
````

- **L121** EN: Assigns or updates `shard_index`. | CN: 对 `shard_index` 进行赋值或更新。
- **L122** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L123** EN: Assigns or updates `storage_plan`. | CN: 对 `storage_plan` 进行赋值或更新。
- **L124** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L125** EN: Assigns or updates `shard_index`. | CN: 对 `shard_index` 进行赋值或更新。
- **L126** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L127** EN: Assigns or updates `buckets`. | CN: 对 `buckets` 进行赋值或更新。
- **L128** EN: Assigns or updates `highest_index`. | CN: 对 `highest_index` 进行赋值或更新。
- **L129** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L130** EN: Assigns or updates `file_queue`. | CN: 对 `file_queue` 进行赋值或更新。
- **L131** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L132** EN: Assigns or updates `file_name`. | CN: 对 `file_name` 进行赋值或更新。
- **L133** EN: Calls `file_queue.put` as part of the current workflow. | CN: 在当前流程中调用 `file_queue.put`。
- **L134** EN: Continues the implementation inside function `write_data`. | CN: 继续说明函数 `write_data` 内部的实现。
- **L135** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L136** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L137** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L138** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L139** EN: Defines function `finish`. | CN: 定义函数 `finish`。
- **L140** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 141-160 / 第 141-160 行

````python
            # if we are saving distributed, without consolidating,
            # then we have no metadata to write because a metadata
            # file with fqn to file mapping doesn't make sense
            # in this case, because fqns will be in multiple files
            logger.info("Not consolidating sharded checkpoint in finish step.")
            return
        if self.save_distributed:
            fqn_to_index_mapping: dict[str, int] = (
                self.fqn_to_index_mapping
                if self.fqn_to_index_mapping is not None
                else dict.fromkeys(metadata.state_dict_metadata.keys(), 1)
            )

            return consolidate_safetensors_files(
                input_dir=str(self.path),
                output_dir=self.consolidated_output_path,  # type: ignore[arg-type]
                num_threads=self.thread_count_consolidation,
                fqn_to_index_mapping=fqn_to_index_mapping,
            )

````

- **L141** EN: Keeps the inline comment or directive: if we are saving distributed, without consolidating, | CN: 保留这一行注释或指令：if we are saving distributed, without consolidating,
- **L142** EN: Keeps the inline comment or directive: then we have no metadata to write because a metadata | CN: 保留这一行注释或指令：then we have no metadata to write because a metadata
- **L143** EN: Keeps the inline comment or directive: file with fqn to file mapping doesn't make sense | CN: 保留这一行注释或指令：file with fqn to file mapping doesn't make sense
- **L144** EN: Keeps the inline comment or directive: in this case, because fqns will be in multiple files | CN: 保留这一行注释或指令：in this case, because fqns will be in multiple files
- **L145** EN: Calls `logger.info` as part of the current workflow. | CN: 在当前流程中调用 `logger.info`。
- **L146** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L147** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L148** EN: Assigns or updates `fqn_to_index_mapping`. | CN: 对 `fqn_to_index_mapping` 进行赋值或更新。
- **L149** EN: Continues the implementation inside function `finish`. | CN: 继续说明函数 `finish` 内部的实现。
- **L150** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L151** EN: Continues the implementation inside function `finish`. | CN: 继续说明函数 `finish` 内部的实现。
- **L152** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L153** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L154** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L155** EN: Assigns or updates `input_dir`. | CN: 对 `input_dir` 进行赋值或更新。
- **L156** EN: Assigns or updates `output_dir`. | CN: 对 `output_dir` 进行赋值或更新。
- **L157** EN: Assigns or updates `num_threads`. | CN: 对 `num_threads` 进行赋值或更新。
- **L158** EN: Assigns or updates `fqn_to_index_mapping`. | CN: 对 `fqn_to_index_mapping` 进行赋值或更新。
- **L159** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L160** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 161-180 / 第 161-180 行

````python
        # writing a model.index.safetensors.json file with fqn to file mapping
        # for the rank-0 checkpointing case
        metadata_to_write = {}
        storage_md = {}
        total_size = 0
        for wr_list in results:
            storage_md.update(
                {wr.index.fqn: wr.storage_data.relative_path for wr in wr_list}
            )
            total_size += sum([wr.storage_data.length for wr in wr_list])
        metadata_to_write["metadata"] = {"total_size": total_size}
        metadata_to_write["weight_map"] = storage_md

        metadata_path = self.fs.concat_path(self.path, f"{_metadata_fn}")
        with self.fs.create_stream(metadata_path, "w") as metadata_file:
            json.dump(metadata_to_write, metadata_file, indent=2)

    def _split_by_storage_plan(
        self, storage_plan: dict[str, int] | None, items: list[WriteItem]
    ) -> dict[int, list[WriteItem]]:
````

- **L161** EN: Keeps the inline comment or directive: writing a model.index.safetensors.json file with fqn to file mapping | CN: 保留这一行注释或指令：writing a model.index.safetensors.json file with fqn to file mapping
- **L162** EN: Keeps the inline comment or directive: for the rank-0 checkpointing case | CN: 保留这一行注释或指令：for the rank-0 checkpointing case
- **L163** EN: Assigns or updates `metadata_to_write`. | CN: 对 `metadata_to_write` 进行赋值或更新。
- **L164** EN: Assigns or updates `storage_md`. | CN: 对 `storage_md` 进行赋值或更新。
- **L165** EN: Assigns or updates `total_size`. | CN: 对 `total_size` 进行赋值或更新。
- **L166** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L167** EN: Calls `storage_md.update` as part of the current workflow. | CN: 在当前流程中调用 `storage_md.update`。
- **L168** EN: Continues the implementation inside function `finish`. | CN: 继续说明函数 `finish` 内部的实现。
- **L169** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L170** EN: Continues the implementation inside function `finish`. | CN: 继续说明函数 `finish` 内部的实现。
- **L171** EN: Continues the implementation inside function `finish`. | CN: 继续说明函数 `finish` 内部的实现。
- **L172** EN: Continues the implementation inside function `finish`. | CN: 继续说明函数 `finish` 内部的实现。
- **L173** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L174** EN: Assigns or updates `metadata_path`. | CN: 对 `metadata_path` 进行赋值或更新。
- **L175** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L176** EN: Calls `json.dump` as part of the current workflow. | CN: 在当前流程中调用 `json.dump`。
- **L177** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L178** EN: Defines function `_split_by_storage_plan`. | CN: 定义函数 `_split_by_storage_plan`。
- **L179** EN: Continues the implementation inside function `_split_by_storage_plan`. | CN: 继续说明函数 `_split_by_storage_plan` 内部的实现。
- **L180** EN: Continues the implementation inside function `_split_by_storage_plan`. | CN: 继续说明函数 `_split_by_storage_plan` 内部的实现。

### Lines 181-200 / 第 181-200 行

````python
        # storage_plan is a map from key to index
        if storage_plan is None:
            return {1: items}

        buckets = {}
        for item in items:
            key = item.index.fqn

            idx = storage_plan[key]
            if idx not in buckets:
                buckets[idx] = [item]
            else:
                buckets[idx].append(item)

        return buckets

    @property
    def metadata_path(self) -> str:
        return _metadata_fn

````

- **L181** EN: Keeps the inline comment or directive: storage_plan is a map from key to index | CN: 保留这一行注释或指令：storage_plan is a map from key to index
- **L182** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L183** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L184** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L185** EN: Assigns or updates `buckets`. | CN: 对 `buckets` 进行赋值或更新。
- **L186** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L187** EN: Assigns or updates `key`. | CN: 对 `key` 进行赋值或更新。
- **L188** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L189** EN: Assigns or updates `idx`. | CN: 对 `idx` 进行赋值或更新。
- **L190** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L191** EN: Assigns or updates `buckets[idx]`. | CN: 对 `buckets[idx]` 进行赋值或更新。
- **L192** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L193** EN: Continues the implementation inside function `_split_by_storage_plan`. | CN: 继续说明函数 `_split_by_storage_plan` 内部的实现。
- **L194** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L195** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L196** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L197** EN: Applies decorator `property` to the following definition. | CN: 将装饰器 `property` 应用于后续定义。
- **L198** EN: Defines function `metadata_path`. | CN: 定义函数 `metadata_path`。
- **L199** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L200** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 201-220 / 第 201-220 行

````python

class HuggingFaceStorageReader(FileSystemReader):
    """
    A reader that reads a checkpoint in the huggingface safetensors format.
    """

    def __init__(self, path: str, thread_count: int = 1) -> None:
        """
        Initialize the huggingface reader pointing to path.

        Args:
            path: directory where the checkpoint will be read from.
            thread_count: Number of threads to use to read distributed checkpoint. Default to 1.
        """

        super().__init__(path=path)
        self.thread_count = thread_count

    def _process_read_request(self, f, req: ReadItem, planner: LoadPlanner) -> None:
        """Helper function to process a single read request."""
````

- **L201** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L202** EN: Defines class `HuggingFaceStorageReader`. | CN: 定义类 `HuggingFaceStorageReader`。
- **L203** EN: Starts the docstring for the class HuggingFaceStorageReader. | CN: 开始定义 class HuggingFaceStorageReader 的文档字符串。
- **L204** EN: Continues the docstring text for the class HuggingFaceStorageReader. | CN: 继续补充 class HuggingFaceStorageReader 的文档字符串内容。
- **L205** EN: Closes the docstring for the class HuggingFaceStorageReader. | CN: 结束 class HuggingFaceStorageReader 的文档字符串。
- **L206** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L207** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L208** EN: Starts the docstring for the function __init__. | CN: 开始定义 function __init__ 的文档字符串。
- **L209** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L210** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L211** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L212** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L213** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L214** EN: Closes the docstring for the function __init__. | CN: 结束 function __init__ 的文档字符串。
- **L215** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L216** EN: Calls `super` as part of the current workflow. | CN: 在当前流程中调用 `super`。
- **L217** EN: Assigns or updates `self.thread_count`. | CN: 对 `self.thread_count` 进行赋值或更新。
- **L218** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L219** EN: Defines function `_process_read_request`. | CN: 定义函数 `_process_read_request`。
- **L220** EN: Docstring line documenting the function _process_read_request. | CN: 这是记录 function _process_read_request 的文档字符串。

### Lines 221-240 / 第 221-240 行

````python
        # Create slices for each dimension based on offsets and lengths
        slices = tuple(
            slice(offset, offset + length)
            for offset, length in zip(req.storage_offsets, req.lengths)
        )
        tensor = f.get_slice(req.storage_index.fqn)[slices]
        target_tensor = planner.resolve_tensor(req).detach()

        if target_tensor.size() != tensor.size():
            raise AssertionError(
                f"req {req.storage_index} mismatch sizes {target_tensor.size()} vs {tensor.size()}"
            )

        target_tensor.copy_(tensor)
        planner.commit_tensor(req, target_tensor)

    def _read_files_from_queue(
        self,
        file_queue: queue.Queue,
        result_queue: queue.Queue,
````

- **L221** EN: Keeps the inline comment or directive: Create slices for each dimension based on offsets and lengths | CN: 保留这一行注释或指令：Create slices for each dimension based on offsets and lengths
- **L222** EN: Assigns or updates `slices`. | CN: 对 `slices` 进行赋值或更新。
- **L223** EN: Calls `slice` as part of the current workflow. | CN: 在当前流程中调用 `slice`。
- **L224** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L225** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L226** EN: Assigns or updates `tensor`. | CN: 对 `tensor` 进行赋值或更新。
- **L227** EN: Assigns or updates `target_tensor`. | CN: 对 `target_tensor` 进行赋值或更新。
- **L228** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L229** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L230** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L231** EN: Continues the implementation inside function `_process_read_request`. | CN: 继续说明函数 `_process_read_request` 内部的实现。
- **L232** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L233** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L234** EN: Calls `target_tensor.copy_` as part of the current workflow. | CN: 在当前流程中调用 `target_tensor.copy_`。
- **L235** EN: Calls `planner.commit_tensor` as part of the current workflow. | CN: 在当前流程中调用 `planner.commit_tensor`。
- **L236** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L237** EN: Defines function `_read_files_from_queue`. | CN: 定义函数 `_read_files_from_queue`。
- **L238** EN: Continues the implementation inside function `_read_files_from_queue`. | CN: 继续说明函数 `_read_files_from_queue` 内部的实现。
- **L239** EN: Continues the implementation inside function `_read_files_from_queue`. | CN: 继续说明函数 `_read_files_from_queue` 内部的实现。
- **L240** EN: Continues the implementation inside function `_read_files_from_queue`. | CN: 继续说明函数 `_read_files_from_queue` 内部的实现。

### Lines 241-260 / 第 241-260 行

````python
        planner: LoadPlanner,
    ) -> None:
        from safetensors import safe_open  # type: ignore[import]

        try:
            while True:
                file_name, reqs = file_queue.get_nowait()
                with safe_open(filename=file_name, framework="pt") as f:
                    for req in reqs:
                        self._process_read_request(f, req, planner)
                result_queue.put(True)  # Signal that this file has been processed
        except queue.Empty:
            pass

    def read_data(self, plan: LoadPlan, planner: LoadPlanner) -> Future[None]:
        from safetensors import safe_open  # type: ignore[import]

        per_file: dict[str, list[ReadItem]] = {}

        for read_item in plan.items:
````

- **L241** EN: Continues the implementation inside function `_read_files_from_queue`. | CN: 继续说明函数 `_read_files_from_queue` 内部的实现。
- **L242** EN: Continues the implementation inside function `_read_files_from_queue`. | CN: 继续说明函数 `_read_files_from_queue` 内部的实现。
- **L243** EN: Imports selected names from `safetensors`. | CN: 从 `safetensors` 导入指定名称。
- **L244** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L245** EN: Starts a protected block for exception handling. | CN: 开始一个受保护的异常处理代码块。
- **L246** EN: Starts a loop controlled by a condition. | CN: 开始一个由条件控制的循环。
- **L247** EN: Assigns or updates `file_name, reqs`. | CN: 对 `file_name, reqs` 进行赋值或更新。
- **L248** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L249** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L250** EN: Calls `self._process_read_request` as part of the current workflow. | CN: 在当前流程中调用 `self._process_read_request`。
- **L251** EN: Calls `result_queue.put` as part of the current workflow. | CN: 在当前流程中调用 `result_queue.put`。
- **L252** EN: Handles a matching exception path. | CN: 处理匹配到的异常路径。
- **L253** EN: Keeps an intentionally empty block. | CN: 保留一个有意为空的代码块。
- **L254** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L255** EN: Defines function `read_data`. | CN: 定义函数 `read_data`。
- **L256** EN: Imports selected names from `safetensors`. | CN: 从 `safetensors` 导入指定名称。
- **L257** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L258** EN: Assigns or updates `per_file`. | CN: 对 `per_file` 进行赋值或更新。
- **L259** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L260** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。

### Lines 261-280 / 第 261-280 行

````python
            item_md: _HFStorageInfo = self.storage_data[read_item.storage_index]
            file_name = item_md.relative_path
            per_file.setdefault(file_name, []).append(read_item)

        if self.thread_count <= 1 or len(per_file) <= 1:
            for file_name, reqs in per_file.items():
                with safe_open(filename=file_name, framework="pt") as f:
                    for req in reqs:
                        self._process_read_request(f, req, planner)
        else:
            # Use parallel implementation with thread pool
            file_queue: queue.Queue = queue.Queue()
            result_queue: queue.Queue = queue.Queue()

            # Fill the queue with files to process
            for file_name, reqs in per_file.items():
                file_queue.put((file_name, reqs))

            # Create and start worker threads
            threads = []
````

- **L261** EN: Assigns or updates `item_md`. | CN: 对 `item_md` 进行赋值或更新。
- **L262** EN: Assigns or updates `file_name`. | CN: 对 `file_name` 进行赋值或更新。
- **L263** EN: Calls `per_file.setdefault` as part of the current workflow. | CN: 在当前流程中调用 `per_file.setdefault`。
- **L264** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L265** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L266** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L267** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L268** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L269** EN: Calls `self._process_read_request` as part of the current workflow. | CN: 在当前流程中调用 `self._process_read_request`。
- **L270** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L271** EN: Keeps the inline comment or directive: Use parallel implementation with thread pool | CN: 保留这一行注释或指令：Use parallel implementation with thread pool
- **L272** EN: Assigns or updates `file_queue`. | CN: 对 `file_queue` 进行赋值或更新。
- **L273** EN: Assigns or updates `result_queue`. | CN: 对 `result_queue` 进行赋值或更新。
- **L274** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L275** EN: Keeps the inline comment or directive: Fill the queue with files to process | CN: 保留这一行注释或指令：Fill the queue with files to process
- **L276** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L277** EN: Calls `file_queue.put` as part of the current workflow. | CN: 在当前流程中调用 `file_queue.put`。
- **L278** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L279** EN: Keeps the inline comment or directive: Create and start worker threads | CN: 保留这一行注释或指令：Create and start worker threads
- **L280** EN: Assigns or updates `threads`. | CN: 对 `threads` 进行赋值或更新。

### Lines 281-300 / 第 281-300 行

````python
            num_threads = min(self.thread_count, len(per_file))
            for _ in range(num_threads):
                t = threading.Thread(
                    target=self._read_files_from_queue,
                    args=(file_queue, result_queue, planner),
                )
                t.start()
                threads.append(t)

            # Wait for all threads to complete
            for t in threads:
                t.join()

            # Check if all files were processed
            processed_count = 0
            try:
                while True:
                    result_queue.get_nowait()
                    processed_count += 1
            except queue.Empty:
````

- **L281** EN: Assigns or updates `num_threads`. | CN: 对 `num_threads` 进行赋值或更新。
- **L282** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L283** EN: Assigns or updates `t`. | CN: 对 `t` 进行赋值或更新。
- **L284** EN: Assigns or updates `target`. | CN: 对 `target` 进行赋值或更新。
- **L285** EN: Assigns or updates `args`. | CN: 对 `args` 进行赋值或更新。
- **L286** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L287** EN: Calls `t.start` as part of the current workflow. | CN: 在当前流程中调用 `t.start`。
- **L288** EN: Calls `threads.append` as part of the current workflow. | CN: 在当前流程中调用 `threads.append`。
- **L289** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L290** EN: Keeps the inline comment or directive: Wait for all threads to complete | CN: 保留这一行注释或指令：Wait for all threads to complete
- **L291** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L292** EN: Calls `t.join` as part of the current workflow. | CN: 在当前流程中调用 `t.join`。
- **L293** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L294** EN: Keeps the inline comment or directive: Check if all files were processed | CN: 保留这一行注释或指令：Check if all files were processed
- **L295** EN: Assigns or updates `processed_count`. | CN: 对 `processed_count` 进行赋值或更新。
- **L296** EN: Starts a protected block for exception handling. | CN: 开始一个受保护的异常处理代码块。
- **L297** EN: Starts a loop controlled by a condition. | CN: 开始一个由条件控制的循环。
- **L298** EN: Calls `result_queue.get_nowait` as part of the current workflow. | CN: 在当前流程中调用 `result_queue.get_nowait`。
- **L299** EN: Continues the implementation inside function `read_data`. | CN: 继续说明函数 `read_data` 内部的实现。
- **L300** EN: Handles a matching exception path. | CN: 处理匹配到的异常路径。

### Lines 301-320 / 第 301-320 行

````python
                pass

            if processed_count != len(per_file):
                raise AssertionError(
                    f"Not all files were processed: {processed_count} out of {len(per_file)}"
                )

        fut: Future = Future()
        fut.set_result(None)
        return fut

    # pyrefly: ignore [bad-override]
    def read_metadata(self) -> Metadata:
        from safetensors import safe_open  # type: ignore[import]
        from safetensors.torch import _getdtype  # type: ignore[import]

        state_dict_metadata: dict[str, TensorStorageMetadata] = {}
        storage_data: dict[MetadataIndex, _HFStorageInfo] = {}

        safetensors_files = []
````

- **L301** EN: Keeps an intentionally empty block. | CN: 保留一个有意为空的代码块。
- **L302** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L303** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L304** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L305** EN: Continues the implementation inside function `read_data`. | CN: 继续说明函数 `read_data` 内部的实现。
- **L306** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L307** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L308** EN: Assigns or updates `fut`. | CN: 对 `fut` 进行赋值或更新。
- **L309** EN: Calls `fut.set_result` as part of the current workflow. | CN: 在当前流程中调用 `fut.set_result`。
- **L310** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L311** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L312** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-override] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-override]
- **L313** EN: Defines function `read_metadata`. | CN: 定义函数 `read_metadata`。
- **L314** EN: Imports selected names from `safetensors`. | CN: 从 `safetensors` 导入指定名称。
- **L315** EN: Imports selected names from `safetensors.torch`. | CN: 从 `safetensors.torch` 导入指定名称。
- **L316** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L317** EN: Assigns or updates `state_dict_metadata`. | CN: 对 `state_dict_metadata` 进行赋值或更新。
- **L318** EN: Assigns or updates `storage_data`. | CN: 对 `storage_data` 进行赋值或更新。
- **L319** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L320** EN: Assigns or updates `safetensors_files`. | CN: 对 `safetensors_files` 进行赋值或更新。

### Lines 321-340 / 第 321-340 行

````python
        for file in self.fs.ls(self.path):
            if file.endswith(SUFFIX):
                safetensors_files.append(file)

        for safetensor_file in safetensors_files:
            with safe_open(safetensor_file, framework="pt") as f:
                keys = f.keys()
                extra_metadata = f.metadata()

                dcp_sharding_info = None
                if extra_metadata and extra_metadata.get(CUSTOM_METADATA_KEY):
                    dcp_sharding_info = json.loads(
                        extra_metadata.get(CUSTOM_METADATA_KEY)
                    )

                for key in keys:
                    shape = f.get_slice(key).get_shape()
                    dtype = f.get_slice(key).get_dtype()
                    # construct state_dict_metadata
                    if dcp_sharding_info is not None:
````

- **L321** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L322** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L323** EN: Calls `safetensors_files.append` as part of the current workflow. | CN: 在当前流程中调用 `safetensors_files.append`。
- **L324** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L325** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L326** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L327** EN: Assigns or updates `keys`. | CN: 对 `keys` 进行赋值或更新。
- **L328** EN: Assigns or updates `extra_metadata`. | CN: 对 `extra_metadata` 进行赋值或更新。
- **L329** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L330** EN: Assigns or updates `dcp_sharding_info`. | CN: 对 `dcp_sharding_info` 进行赋值或更新。
- **L331** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L332** EN: Assigns or updates `dcp_sharding_info`. | CN: 对 `dcp_sharding_info` 进行赋值或更新。
- **L333** EN: Calls `extra_metadata.get` as part of the current workflow. | CN: 在当前流程中调用 `extra_metadata.get`。
- **L334** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L335** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L336** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L337** EN: Assigns or updates `shape`. | CN: 对 `shape` 进行赋值或更新。
- **L338** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L339** EN: Keeps the inline comment or directive: construct state_dict_metadata | CN: 保留这一行注释或指令：construct state_dict_metadata
- **L340** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 341-360 / 第 341-360 行

````python
                        offset = dcp_sharding_info[key][SAVED_OFFSETS_KEY]
                    else:
                        offset = [0] * len(shape)

                    if key not in state_dict_metadata:
                        state_dict_metadata[key] = TensorStorageMetadata(
                            properties=TensorProperties(dtype=_getdtype(dtype)),
                            size=torch.Size(
                                [saved + offset for saved, offset in zip(shape, offset)]
                            ),
                            chunks=[
                                ChunkStorageMetadata(
                                    offsets=torch.Size(offset),
                                    sizes=torch.Size(shape),
                                )
                            ],
                        )
                    else:
                        state_dict_metadata[key].chunks.append(
                            ChunkStorageMetadata(
````

- **L341** EN: Assigns or updates `offset`. | CN: 对 `offset` 进行赋值或更新。
- **L342** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L343** EN: Assigns or updates `offset`. | CN: 对 `offset` 进行赋值或更新。
- **L344** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L345** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L346** EN: Assigns or updates `state_dict_metadata[key]`. | CN: 对 `state_dict_metadata[key]` 进行赋值或更新。
- **L347** EN: Assigns or updates `properties`. | CN: 对 `properties` 进行赋值或更新。
- **L348** EN: Assigns or updates `size`. | CN: 对 `size` 进行赋值或更新。
- **L349** EN: Continues the implementation inside function `read_metadata`. | CN: 继续说明函数 `read_metadata` 内部的实现。
- **L350** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L351** EN: Assigns or updates `chunks`. | CN: 对 `chunks` 进行赋值或更新。
- **L352** EN: Calls `ChunkStorageMetadata` as part of the current workflow. | CN: 在当前流程中调用 `ChunkStorageMetadata`。
- **L353** EN: Assigns or updates `offsets`. | CN: 对 `offsets` 进行赋值或更新。
- **L354** EN: Assigns or updates `sizes`. | CN: 对 `sizes` 进行赋值或更新。
- **L355** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L356** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L357** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L358** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L359** EN: Continues the implementation inside function `read_metadata`. | CN: 继续说明函数 `read_metadata` 内部的实现。
- **L360** EN: Calls `ChunkStorageMetadata` as part of the current workflow. | CN: 在当前流程中调用 `ChunkStorageMetadata`。

### Lines 361-380 / 第 361-380 行

````python
                                torch.Size(offset), sizes=torch.Size(shape)
                            )
                        )
                        size = list(state_dict_metadata[key].size)
                        for i in range(len(size)):
                            size[i] = max(size[i], shape[i] + offset[i])
                        state_dict_metadata[key].size = torch.Size(size)

                    # construct storage data
                    if dcp_sharding_info is not None:
                        metadata_index = MetadataIndex(
                            fqn=key, offset=dcp_sharding_info[key][SAVED_OFFSETS_KEY]
                        )
                    else:
                        metadata_index = MetadataIndex(fqn=key, offset=[0] * len(shape))
                    storage_data[metadata_index] = _HFStorageInfo(
                        relative_path=safetensor_file,
                        shape=torch.Size(shape),
                        dtype=_getdtype(dtype),
                    )
````

- **L361** EN: Calls `torch.Size` as part of the current workflow. | CN: 在当前流程中调用 `torch.Size`。
- **L362** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L363** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L364** EN: Assigns or updates `size`. | CN: 对 `size` 进行赋值或更新。
- **L365** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L366** EN: Assigns or updates `size[i]`. | CN: 对 `size[i]` 进行赋值或更新。
- **L367** EN: Assigns or updates `state_dict_metadata[key].size`. | CN: 对 `state_dict_metadata[key].size` 进行赋值或更新。
- **L368** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L369** EN: Keeps the inline comment or directive: construct storage data | CN: 保留这一行注释或指令：construct storage data
- **L370** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L371** EN: Assigns or updates `metadata_index`. | CN: 对 `metadata_index` 进行赋值或更新。
- **L372** EN: Assigns or updates `fqn`. | CN: 对 `fqn` 进行赋值或更新。
- **L373** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L374** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L375** EN: Assigns or updates `metadata_index`. | CN: 对 `metadata_index` 进行赋值或更新。
- **L376** EN: Assigns or updates `storage_data[metadata_index]`. | CN: 对 `storage_data[metadata_index]` 进行赋值或更新。
- **L377** EN: Assigns or updates `relative_path`. | CN: 对 `relative_path` 进行赋值或更新。
- **L378** EN: Assigns or updates `shape`. | CN: 对 `shape` 进行赋值或更新。
- **L379** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L380** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 381-391 / 第 381-391 行

````python

        metadata = Metadata(
            state_dict_metadata=state_dict_metadata,  # type: ignore[arg-type]
            storage_data=storage_data,
        )

        if getattr(metadata, "storage_meta", None) is None:
            metadata.storage_meta = StorageMeta()
        metadata.storage_meta.load_id = self.load_id  # type: ignore[union-attr]

        return metadata
````

- **L381** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L382** EN: Assigns or updates `metadata`. | CN: 对 `metadata` 进行赋值或更新。
- **L383** EN: Assigns or updates `state_dict_metadata`. | CN: 对 `state_dict_metadata` 进行赋值或更新。
- **L384** EN: Assigns or updates `storage_data`. | CN: 对 `storage_data` 进行赋值或更新。
- **L385** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L386** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L387** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L388** EN: Assigns or updates `metadata.storage_meta`. | CN: 对 `metadata.storage_meta` 进行赋值或更新。
- **L389** EN: Assigns or updates `metadata.storage_meta.load_id`. | CN: 对 `metadata.storage_meta.load_id` 进行赋值或更新。
- **L390** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L391** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

## Key Concepts / 关键概念

- **EN**: distributed checkpointing, planners, and storage helpers  
  **CN**: 分布式检查点、规划器与存储辅助逻辑
- **EN**: sharding  
  **CN**: 分片
- **EN**: checkpointing  
  **CN**: 检查点
- **EN**: state dict handling  
  **CN**: state_dict 处理
- **EN**: Primary classes: HuggingFaceStorageWriter, HuggingFaceStorageReader  
  **CN**: 主要类：HuggingFaceStorageWriter, HuggingFaceStorageReader

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed.checkpoint`, `torch.distributed.checkpoint._consolidate_hf_safetensors`, `torch.distributed.checkpoint._hf_utils`, `torch.distributed.checkpoint.filesystem`, `torch.distributed.checkpoint.metadata`, `torch.distributed.checkpoint.planner`, `torch.distributed.checkpoint.storage`
- **PyTorch / PyTorch**: `torch`, `torch.futures`
- **Python Stdlib / Python 标准库**: `dataclasses`, `json`, `logging`, `queue`, `threading`, `typing`
- **Third-party / 第三方**: `safetensors`, `safetensors.torch`

