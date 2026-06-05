# _consolidate_hf_safetensors.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/checkpoint/_consolidate_hf_safetensors.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed checkpointing, planners, and storage helpers. Its main entry points include _FqnData, _OutputFileData, _parse_input_metadata, _write_metadata.
- **用途 (CN)**: 该模块聚焦于分布式检查点、规划器与存储辅助逻辑，其主要入口包括 _FqnData, _OutputFileData, _parse_input_metadata, _write_metadata。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# pyre-strict

import concurrent.futures
import glob
import json
import logging
import math
import os
import struct
import time
from dataclasses import dataclass, field
from typing import Any

import torch
from torch import distributed as dist
from torch.distributed.checkpoint._hf_utils import (
    _gen_file_name,
    _get_dcp_custom_metadata,
    _get_safetensors_file_metadata,
    _metadata_fn,
````

- **L1** EN: Keeps the inline comment or directive: pyre-strict | CN: 保留这一行注释或指令：pyre-strict
- **L2** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L3** EN: Imports module dependencies: `concurrent.futures`. | CN: 导入模块依赖：`concurrent.futures`。
- **L4** EN: Imports module dependencies: `glob`. | CN: 导入模块依赖：`glob`。
- **L5** EN: Imports module dependencies: `json`. | CN: 导入模块依赖：`json`。
- **L6** EN: Imports module dependencies: `logging`. | CN: 导入模块依赖：`logging`。
- **L7** EN: Imports module dependencies: `math`. | CN: 导入模块依赖：`math`。
- **L8** EN: Imports module dependencies: `os`. | CN: 导入模块依赖：`os`。
- **L9** EN: Imports module dependencies: `struct`. | CN: 导入模块依赖：`struct`。
- **L10** EN: Imports module dependencies: `time`. | CN: 导入模块依赖：`time`。
- **L11** EN: Imports selected names from `dataclasses`. | CN: 从 `dataclasses` 导入指定名称。
- **L12** EN: Imports selected names from `typing`. | CN: 从 `typing` 导入指定名称。
- **L13** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L14** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L15** EN: Imports selected names from `torch`. | CN: 从 `torch` 导入指定名称。
- **L16** EN: Imports selected names from `torch.distributed.checkpoint._hf_utils`. | CN: 从 `torch.distributed.checkpoint._hf_utils` 导入指定名称。
- **L17** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L18** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L19** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L20** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。

### Lines 21-40 / 第 21-40 行

````python
    DATA_OFFSETS_KEY,
    DEFAULT_EXTRA_METADATA_KEY,
    DTYPE_KEY,
    SAVED_OFFSETS_KEY,
    SHAPE_KEY,
    SUFFIX,
)


logger: logging.Logger = logging.getLogger(__name__)


@dataclass
class _FqnData:
    """
    Dataclass to store information about a tensor (identified by its fully qualified name).

    Attributes:
        offset_in_file: Byte offset where this tensor's data begins in the output file
        shape_in_file: Shape of the tensor in the output file
````

- **L21** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L22** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L23** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L24** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L25** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L26** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L27** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L28** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L29** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L30** EN: Assigns or updates `logger`. | CN: 对 `logger` 进行赋值或更新。
- **L31** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L32** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L33** EN: Applies decorator `dataclass` to the following definition. | CN: 将装饰器 `dataclass` 应用于后续定义。
- **L34** EN: Defines class `_FqnData`. | CN: 定义类 `_FqnData`。
- **L35** EN: Starts the docstring for the class _FqnData. | CN: 开始定义 class _FqnData 的文档字符串。
- **L36** EN: Continues the docstring text for the class _FqnData. | CN: 继续补充 class _FqnData 的文档字符串内容。
- **L37** EN: Continues the docstring text for the class _FqnData. | CN: 继续补充 class _FqnData 的文档字符串内容。
- **L38** EN: Continues the docstring text for the class _FqnData. | CN: 继续补充 class _FqnData 的文档字符串内容。
- **L39** EN: Continues the docstring text for the class _FqnData. | CN: 继续补充 class _FqnData 的文档字符串内容。
- **L40** EN: Continues the docstring text for the class _FqnData. | CN: 继续补充 class _FqnData 的文档字符串内容。

### Lines 41-60 / 第 41-60 行

````python
        dtype_size: Size of the tensor's data type in bytes
        dtype_str: String representation of the tensor's data type
    """

    offset_in_file: int = 0
    shape_in_file: list[int] = field(default_factory=list)
    dtype_size: int = 0
    dtype_str: str = ""


@dataclass
class _OutputFileData:
    """
    Dataclass to store information about an output safetensors file.

    Attributes:
        metadata_size: Size of the metadata section in bytes
        fqn_data: Dictionary mapping tensor names to their metadata
    """

````

- **L41** EN: Continues the docstring text for the class _FqnData. | CN: 继续补充 class _FqnData 的文档字符串内容。
- **L42** EN: Continues the docstring text for the class _FqnData. | CN: 继续补充 class _FqnData 的文档字符串内容。
- **L43** EN: Closes the docstring for the class _FqnData. | CN: 结束 class _FqnData 的文档字符串。
- **L44** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L45** EN: Assigns or updates `offset_in_file`. | CN: 对 `offset_in_file` 进行赋值或更新。
- **L46** EN: Assigns or updates `shape_in_file`. | CN: 对 `shape_in_file` 进行赋值或更新。
- **L47** EN: Assigns or updates `dtype_size`. | CN: 对 `dtype_size` 进行赋值或更新。
- **L48** EN: Assigns or updates `dtype_str`. | CN: 对 `dtype_str` 进行赋值或更新。
- **L49** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L50** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L51** EN: Applies decorator `dataclass` to the following definition. | CN: 将装饰器 `dataclass` 应用于后续定义。
- **L52** EN: Defines class `_OutputFileData`. | CN: 定义类 `_OutputFileData`。
- **L53** EN: Starts the docstring for the class _OutputFileData. | CN: 开始定义 class _OutputFileData 的文档字符串。
- **L54** EN: Continues the docstring text for the class _OutputFileData. | CN: 继续补充 class _OutputFileData 的文档字符串内容。
- **L55** EN: Continues the docstring text for the class _OutputFileData. | CN: 继续补充 class _OutputFileData 的文档字符串内容。
- **L56** EN: Continues the docstring text for the class _OutputFileData. | CN: 继续补充 class _OutputFileData 的文档字符串内容。
- **L57** EN: Continues the docstring text for the class _OutputFileData. | CN: 继续补充 class _OutputFileData 的文档字符串内容。
- **L58** EN: Continues the docstring text for the class _OutputFileData. | CN: 继续补充 class _OutputFileData 的文档字符串内容。
- **L59** EN: Closes the docstring for the class _OutputFileData. | CN: 结束 class _OutputFileData 的文档字符串。
- **L60** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 61-80 / 第 61-80 行

````python
    metadata_size: int = 0
    fqn_data: dict[str, _FqnData] = field(default_factory=dict)


@dataclass
class _InputFileData:
    """
    Dataclass to store information about an input safetensors file.

    Attributes:
        metadata_size: Size of the metadata section in bytes
        metadata: Json metadata from the safetensors file
    """

    metadata_size: int = 0
    metadata: Any = None


def _parse_input_metadata(
    input_files_data: dict[str, _InputFileData],
````

- **L61** EN: Assigns or updates `metadata_size`. | CN: 对 `metadata_size` 进行赋值或更新。
- **L62** EN: Assigns or updates `fqn_data`. | CN: 对 `fqn_data` 进行赋值或更新。
- **L63** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L64** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L65** EN: Applies decorator `dataclass` to the following definition. | CN: 将装饰器 `dataclass` 应用于后续定义。
- **L66** EN: Defines class `_InputFileData`. | CN: 定义类 `_InputFileData`。
- **L67** EN: Starts the docstring for the class _InputFileData. | CN: 开始定义 class _InputFileData 的文档字符串。
- **L68** EN: Continues the docstring text for the class _InputFileData. | CN: 继续补充 class _InputFileData 的文档字符串内容。
- **L69** EN: Continues the docstring text for the class _InputFileData. | CN: 继续补充 class _InputFileData 的文档字符串内容。
- **L70** EN: Continues the docstring text for the class _InputFileData. | CN: 继续补充 class _InputFileData 的文档字符串内容。
- **L71** EN: Continues the docstring text for the class _InputFileData. | CN: 继续补充 class _InputFileData 的文档字符串内容。
- **L72** EN: Continues the docstring text for the class _InputFileData. | CN: 继续补充 class _InputFileData 的文档字符串内容。
- **L73** EN: Closes the docstring for the class _InputFileData. | CN: 结束 class _InputFileData 的文档字符串。
- **L74** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L75** EN: Assigns or updates `metadata_size`. | CN: 对 `metadata_size` 进行赋值或更新。
- **L76** EN: Assigns or updates `metadata`. | CN: 对 `metadata` 进行赋值或更新。
- **L77** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L78** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L79** EN: Defines function `_parse_input_metadata`. | CN: 定义函数 `_parse_input_metadata`。
- **L80** EN: Continues the implementation inside function `_parse_input_metadata`. | CN: 继续说明函数 `_parse_input_metadata` 内部的实现。

### Lines 81-100 / 第 81-100 行

````python
    output_files_data: dict[str, _OutputFileData],
) -> None:
    """
    Parse metadata from input safetensors files to determine the full tensor shapes and types.

    This function analyzes the metadata from all input files to determine the complete shape
    of each tensor after consolidation. It updates the output_files_data with this information.

    Args:
        input_files_data: dict of metadata from input safetensors files
        output_files_data: Dictionary mapping output file paths to their metadata

    Raises:
        ValueError: If no DCP custom metadata is found in a safetensors file
    """

    from safetensors.torch import _getdtype  # type: ignore[import]

    # Dictionary to track the full size of each tensor across all shards
    fqn_to_size_mapping: dict[str, tuple[list[int], str]] = {}
````

- **L81** EN: Continues the implementation inside function `_parse_input_metadata`. | CN: 继续说明函数 `_parse_input_metadata` 内部的实现。
- **L82** EN: Continues the implementation inside function `_parse_input_metadata`. | CN: 继续说明函数 `_parse_input_metadata` 内部的实现。
- **L83** EN: Starts the docstring for the function _parse_input_metadata. | CN: 开始定义 function _parse_input_metadata 的文档字符串。
- **L84** EN: Continues the docstring text for the function _parse_input_metadata. | CN: 继续补充 function _parse_input_metadata 的文档字符串内容。
- **L85** EN: Continues the docstring text for the function _parse_input_metadata. | CN: 继续补充 function _parse_input_metadata 的文档字符串内容。
- **L86** EN: Continues the docstring text for the function _parse_input_metadata. | CN: 继续补充 function _parse_input_metadata 的文档字符串内容。
- **L87** EN: Continues the docstring text for the function _parse_input_metadata. | CN: 继续补充 function _parse_input_metadata 的文档字符串内容。
- **L88** EN: Continues the docstring text for the function _parse_input_metadata. | CN: 继续补充 function _parse_input_metadata 的文档字符串内容。
- **L89** EN: Continues the docstring text for the function _parse_input_metadata. | CN: 继续补充 function _parse_input_metadata 的文档字符串内容。
- **L90** EN: Continues the docstring text for the function _parse_input_metadata. | CN: 继续补充 function _parse_input_metadata 的文档字符串内容。
- **L91** EN: Continues the docstring text for the function _parse_input_metadata. | CN: 继续补充 function _parse_input_metadata 的文档字符串内容。
- **L92** EN: Continues the docstring text for the function _parse_input_metadata. | CN: 继续补充 function _parse_input_metadata 的文档字符串内容。
- **L93** EN: Continues the docstring text for the function _parse_input_metadata. | CN: 继续补充 function _parse_input_metadata 的文档字符串内容。
- **L94** EN: Continues the docstring text for the function _parse_input_metadata. | CN: 继续补充 function _parse_input_metadata 的文档字符串内容。
- **L95** EN: Closes the docstring for the function _parse_input_metadata. | CN: 结束 function _parse_input_metadata 的文档字符串。
- **L96** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L97** EN: Imports selected names from `safetensors.torch`. | CN: 从 `safetensors.torch` 导入指定名称。
- **L98** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L99** EN: Keeps the inline comment or directive: Dictionary to track the full size of each tensor across all shards | CN: 保留这一行注释或指令：Dictionary to track the full size of each tensor across all shards
- **L100** EN: Assigns or updates `fqn_to_size_mapping`. | CN: 对 `fqn_to_size_mapping` 进行赋值或更新。

### Lines 101-120 / 第 101-120 行

````python

    for file_data in input_files_data.values():
        safetensors_metadata = file_data.metadata
        dcp_sharding_info = _get_dcp_custom_metadata(safetensors_metadata)
        if not dcp_sharding_info:
            raise ValueError(
                "No DCP custom metadata found in safetensors file. The file must be saved with DCP to be consolidated."
            )

        for key, val in safetensors_metadata.items():
            if key == DEFAULT_EXTRA_METADATA_KEY:
                continue

            # Get the shape of this tensor shard and its offset in the full tensor
            sizes = val[SHAPE_KEY]
            offsets = dcp_sharding_info[key][SAVED_OFFSETS_KEY]

            if key not in fqn_to_size_mapping:
                # First time seeing this tensor - calculate its full size by adding offsets to dimensions
                cur_size = [size + offset for size, offset in zip(sizes, offsets)]
````

- **L101** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L102** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L103** EN: Assigns or updates `safetensors_metadata`. | CN: 对 `safetensors_metadata` 进行赋值或更新。
- **L104** EN: Assigns or updates `dcp_sharding_info`. | CN: 对 `dcp_sharding_info` 进行赋值或更新。
- **L105** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L106** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L107** EN: Continues the implementation inside function `_parse_input_metadata`. | CN: 继续说明函数 `_parse_input_metadata` 内部的实现。
- **L108** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L109** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L110** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L111** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L112** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L113** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L114** EN: Keeps the inline comment or directive: Get the shape of this tensor shard and its offset in the full tensor | CN: 保留这一行注释或指令：Get the shape of this tensor shard and its offset in the full tensor
- **L115** EN: Assigns or updates `sizes`. | CN: 对 `sizes` 进行赋值或更新。
- **L116** EN: Assigns or updates `offsets`. | CN: 对 `offsets` 进行赋值或更新。
- **L117** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L118** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L119** EN: Keeps the inline comment or directive: First time seeing this tensor - calculate its full size by adding offsets to dim | CN: 保留这一行注释或指令：First time seeing this tensor - calculate its full size by adding offsets to dim
- **L120** EN: Assigns or updates `cur_size`. | CN: 对 `cur_size` 进行赋值或更新。

### Lines 121-140 / 第 121-140 行

````python
                fqn_to_size_mapping[key] = (cur_size, val[DTYPE_KEY])
            else:
                # We've seen this tensor before - update its size if this shard extends beyond current known dimensions
                cur_size = fqn_to_size_mapping[key][0]
                for i in range(len(sizes)):
                    cur_size[i] = max(cur_size[i], sizes[i] + offsets[i])

    # Now that we know the full size of each tensor, populate the output file data
    for fqn, tensor_info in fqn_to_size_mapping.items():
        tensor_size = tensor_info[0]
        dtype_str = tensor_info[1]
        for output_data in output_files_data.values():
            # Add this tensor to the output file if it's already assigned there
            if fqn in output_data.fqn_data:
                output_data.fqn_data[fqn] = _FqnData(
                    shape_in_file=tensor_size,
                    dtype_size=torch.finfo(_getdtype(dtype_str)).bits
                    // 8,  # Convert bits to bytes
                    dtype_str=dtype_str,
                )
````

- **L121** EN: Assigns or updates `fqn_to_size_mapping[key]`. | CN: 对 `fqn_to_size_mapping[key]` 进行赋值或更新。
- **L122** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L123** EN: Keeps the inline comment or directive: We've seen this tensor before - update its size if this shard extends beyond cur | CN: 保留这一行注释或指令：We've seen this tensor before - update its size if this shard extends beyond cur
- **L124** EN: Assigns or updates `cur_size`. | CN: 对 `cur_size` 进行赋值或更新。
- **L125** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L126** EN: Assigns or updates `cur_size[i]`. | CN: 对 `cur_size[i]` 进行赋值或更新。
- **L127** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L128** EN: Keeps the inline comment or directive: Now that we know the full size of each tensor, populate the output file data | CN: 保留这一行注释或指令：Now that we know the full size of each tensor, populate the output file data
- **L129** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L130** EN: Assigns or updates `tensor_size`. | CN: 对 `tensor_size` 进行赋值或更新。
- **L131** EN: Assigns or updates `dtype_str`. | CN: 对 `dtype_str` 进行赋值或更新。
- **L132** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L133** EN: Keeps the inline comment or directive: Add this tensor to the output file if it's already assigned there | CN: 保留这一行注释或指令：Add this tensor to the output file if it's already assigned there
- **L134** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L135** EN: Assigns or updates `output_data.fqn_data[fqn]`. | CN: 对 `output_data.fqn_data[fqn]` 进行赋值或更新。
- **L136** EN: Assigns or updates `shape_in_file`. | CN: 对 `shape_in_file` 进行赋值或更新。
- **L137** EN: Assigns or updates `dtype_size`. | CN: 对 `dtype_size` 进行赋值或更新。
- **L138** EN: Continues the implementation inside function `_parse_input_metadata`. | CN: 继续说明函数 `_parse_input_metadata` 内部的实现。
- **L139** EN: Assigns or updates `dtype_str`. | CN: 对 `dtype_str` 进行赋值或更新。
- **L140** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 141-160 / 第 141-160 行

````python


def _write_metadata(
    output_files_data: dict[str, _OutputFileData],
) -> None:
    """
    Write metadata to the beginning of each output safetensors file.

    This function writes the metadata section to each output file, including information
    about tensor shapes, data types, and offsets. It also updates the offset_in_file
    field for each tensor in the output_files_data.

    Args:
        output_files_data: Dictionary mapping output file paths to their metadata
    """
    # Process each output file
    for file_path, output_data in output_files_data.items():
        with open(file_path, "wb") as f:
            metadata = {}
            curr_offset = 0
````

- **L141** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L142** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L143** EN: Defines function `_write_metadata`. | CN: 定义函数 `_write_metadata`。
- **L144** EN: Continues the implementation inside function `_write_metadata`. | CN: 继续说明函数 `_write_metadata` 内部的实现。
- **L145** EN: Continues the implementation inside function `_write_metadata`. | CN: 继续说明函数 `_write_metadata` 内部的实现。
- **L146** EN: Starts the docstring for the function _write_metadata. | CN: 开始定义 function _write_metadata 的文档字符串。
- **L147** EN: Continues the docstring text for the function _write_metadata. | CN: 继续补充 function _write_metadata 的文档字符串内容。
- **L148** EN: Continues the docstring text for the function _write_metadata. | CN: 继续补充 function _write_metadata 的文档字符串内容。
- **L149** EN: Continues the docstring text for the function _write_metadata. | CN: 继续补充 function _write_metadata 的文档字符串内容。
- **L150** EN: Continues the docstring text for the function _write_metadata. | CN: 继续补充 function _write_metadata 的文档字符串内容。
- **L151** EN: Continues the docstring text for the function _write_metadata. | CN: 继续补充 function _write_metadata 的文档字符串内容。
- **L152** EN: Continues the docstring text for the function _write_metadata. | CN: 继续补充 function _write_metadata 的文档字符串内容。
- **L153** EN: Continues the docstring text for the function _write_metadata. | CN: 继续补充 function _write_metadata 的文档字符串内容。
- **L154** EN: Continues the docstring text for the function _write_metadata. | CN: 继续补充 function _write_metadata 的文档字符串内容。
- **L155** EN: Closes the docstring for the function _write_metadata. | CN: 结束 function _write_metadata 的文档字符串。
- **L156** EN: Keeps the inline comment or directive: Process each output file | CN: 保留这一行注释或指令：Process each output file
- **L157** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L158** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L159** EN: Assigns or updates `metadata`. | CN: 对 `metadata` 进行赋值或更新。
- **L160** EN: Assigns or updates `curr_offset`. | CN: 对 `curr_offset` 进行赋值或更新。

### Lines 161-180 / 第 161-180 行

````python

            # Calculate offsets for each tensor in the file
            for fqn, fqn_data in output_data.fqn_data.items():
                # Calculate the end offset by multiplying all dimensions and the data type size
                end_offset = (
                    curr_offset
                    + math.prod(fqn_data.shape_in_file) * fqn_data.dtype_size
                )

                # Store metadata for this tensor
                metadata[fqn] = {
                    SHAPE_KEY: fqn_data.shape_in_file,
                    DTYPE_KEY: fqn_data.dtype_str,
                    DATA_OFFSETS_KEY: [
                        curr_offset,
                        end_offset,
                    ],  # Start and end byte offsets
                }
                # Store the offset for later use when writing the actual tensor data
                fqn_data.offset_in_file = curr_offset
````

- **L161** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L162** EN: Keeps the inline comment or directive: Calculate offsets for each tensor in the file | CN: 保留这一行注释或指令：Calculate offsets for each tensor in the file
- **L163** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L164** EN: Keeps the inline comment or directive: Calculate the end offset by multiplying all dimensions and the data type size | CN: 保留这一行注释或指令：Calculate the end offset by multiplying all dimensions and the data type size
- **L165** EN: Assigns or updates `end_offset`. | CN: 对 `end_offset` 进行赋值或更新。
- **L166** EN: Continues the implementation inside function `_write_metadata`. | CN: 继续说明函数 `_write_metadata` 内部的实现。
- **L167** EN: Continues the implementation inside function `_write_metadata`. | CN: 继续说明函数 `_write_metadata` 内部的实现。
- **L168** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L169** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L170** EN: Keeps the inline comment or directive: Store metadata for this tensor | CN: 保留这一行注释或指令：Store metadata for this tensor
- **L171** EN: Assigns or updates `metadata[fqn]`. | CN: 对 `metadata[fqn]` 进行赋值或更新。
- **L172** EN: Continues the implementation inside function `_write_metadata`. | CN: 继续说明函数 `_write_metadata` 内部的实现。
- **L173** EN: Continues the implementation inside function `_write_metadata`. | CN: 继续说明函数 `_write_metadata` 内部的实现。
- **L174** EN: Continues the implementation inside function `_write_metadata`. | CN: 继续说明函数 `_write_metadata` 内部的实现。
- **L175** EN: Continues the implementation inside function `_write_metadata`. | CN: 继续说明函数 `_write_metadata` 内部的实现。
- **L176** EN: Continues the implementation inside function `_write_metadata`. | CN: 继续说明函数 `_write_metadata` 内部的实现。
- **L177** EN: Continues the implementation inside function `_write_metadata`. | CN: 继续说明函数 `_write_metadata` 内部的实现。
- **L178** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L179** EN: Keeps the inline comment or directive: Store the offset for later use when writing the actual tensor data | CN: 保留这一行注释或指令：Store the offset for later use when writing the actual tensor data
- **L180** EN: Assigns or updates `fqn_data.offset_in_file`. | CN: 对 `fqn_data.offset_in_file` 进行赋值或更新。

### Lines 181-200 / 第 181-200 行

````python

                # Update current offset for the next tensor
                curr_offset = end_offset

            # Convert metadata to JSON and encode as bytes
            json_metadata = json.dumps(metadata)
            json_bytes = json_metadata.encode("utf-8")

            # Write the metadata size as an 8-byte unsigned integer (little-endian)
            size_in_bytes = len(json_bytes)
            header_len = struct.pack("<Q", size_in_bytes)

            # Write the header length and metadata to the file
            f.write(header_len)
            f.write(json_bytes)

            # Store the total metadata size (header + JSON) for later use
            output_data.metadata_size = f.tell()


````

- **L181** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L182** EN: Keeps the inline comment or directive: Update current offset for the next tensor | CN: 保留这一行注释或指令：Update current offset for the next tensor
- **L183** EN: Assigns or updates `curr_offset`. | CN: 对 `curr_offset` 进行赋值或更新。
- **L184** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L185** EN: Keeps the inline comment or directive: Convert metadata to JSON and encode as bytes | CN: 保留这一行注释或指令：Convert metadata to JSON and encode as bytes
- **L186** EN: Assigns or updates `json_metadata`. | CN: 对 `json_metadata` 进行赋值或更新。
- **L187** EN: Assigns or updates `json_bytes`. | CN: 对 `json_bytes` 进行赋值或更新。
- **L188** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L189** EN: Keeps the inline comment or directive: Write the metadata size as an 8-byte unsigned integer (little-endian) | CN: 保留这一行注释或指令：Write the metadata size as an 8-byte unsigned integer (little-endian)
- **L190** EN: Assigns or updates `size_in_bytes`. | CN: 对 `size_in_bytes` 进行赋值或更新。
- **L191** EN: Assigns or updates `header_len`. | CN: 对 `header_len` 进行赋值或更新。
- **L192** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L193** EN: Keeps the inline comment or directive: Write the header length and metadata to the file | CN: 保留这一行注释或指令：Write the header length and metadata to the file
- **L194** EN: Calls `f.write` as part of the current workflow. | CN: 在当前流程中调用 `f.write`。
- **L195** EN: Calls `f.write` as part of the current workflow. | CN: 在当前流程中调用 `f.write`。
- **L196** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L197** EN: Keeps the inline comment or directive: Store the total metadata size (header + JSON) for later use | CN: 保留这一行注释或指令：Store the total metadata size (header + JSON) for later use
- **L198** EN: Assigns or updates `output_data.metadata_size`. | CN: 对 `output_data.metadata_size` 进行赋值或更新。
- **L199** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L200** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 201-220 / 第 201-220 行

````python
def _read_tensor_data(
    f,
    start_offset: int,
    end_offset: int,
    metadata_size: int,
) -> bytes:
    """
    Read a specific byte range of tensor data from an open safetensors file.

    Args:
        f: An open file object (handle) for the safetensors file
        start_offset: Start offset of tensor data within the data section
        end_offset: End offset of tensor data within the data section
        metadata_size: Size of the metadata header

    Returns:
        Raw tensor data as bytes
    """
    absolute_start = metadata_size + start_offset
    length = end_offset - start_offset
````

- **L201** EN: Defines function `_read_tensor_data`. | CN: 定义函数 `_read_tensor_data`。
- **L202** EN: Continues the implementation inside function `_read_tensor_data`. | CN: 继续说明函数 `_read_tensor_data` 内部的实现。
- **L203** EN: Continues the implementation inside function `_read_tensor_data`. | CN: 继续说明函数 `_read_tensor_data` 内部的实现。
- **L204** EN: Continues the implementation inside function `_read_tensor_data`. | CN: 继续说明函数 `_read_tensor_data` 内部的实现。
- **L205** EN: Continues the implementation inside function `_read_tensor_data`. | CN: 继续说明函数 `_read_tensor_data` 内部的实现。
- **L206** EN: Continues the implementation inside function `_read_tensor_data`. | CN: 继续说明函数 `_read_tensor_data` 内部的实现。
- **L207** EN: Starts the docstring for the function _read_tensor_data. | CN: 开始定义 function _read_tensor_data 的文档字符串。
- **L208** EN: Continues the docstring text for the function _read_tensor_data. | CN: 继续补充 function _read_tensor_data 的文档字符串内容。
- **L209** EN: Continues the docstring text for the function _read_tensor_data. | CN: 继续补充 function _read_tensor_data 的文档字符串内容。
- **L210** EN: Continues the docstring text for the function _read_tensor_data. | CN: 继续补充 function _read_tensor_data 的文档字符串内容。
- **L211** EN: Continues the docstring text for the function _read_tensor_data. | CN: 继续补充 function _read_tensor_data 的文档字符串内容。
- **L212** EN: Continues the docstring text for the function _read_tensor_data. | CN: 继续补充 function _read_tensor_data 的文档字符串内容。
- **L213** EN: Continues the docstring text for the function _read_tensor_data. | CN: 继续补充 function _read_tensor_data 的文档字符串内容。
- **L214** EN: Continues the docstring text for the function _read_tensor_data. | CN: 继续补充 function _read_tensor_data 的文档字符串内容。
- **L215** EN: Continues the docstring text for the function _read_tensor_data. | CN: 继续补充 function _read_tensor_data 的文档字符串内容。
- **L216** EN: Continues the docstring text for the function _read_tensor_data. | CN: 继续补充 function _read_tensor_data 的文档字符串内容。
- **L217** EN: Continues the docstring text for the function _read_tensor_data. | CN: 继续补充 function _read_tensor_data 的文档字符串内容。
- **L218** EN: Closes the docstring for the function _read_tensor_data. | CN: 结束 function _read_tensor_data 的文档字符串。
- **L219** EN: Assigns or updates `absolute_start`. | CN: 对 `absolute_start` 进行赋值或更新。
- **L220** EN: Assigns or updates `length`. | CN: 对 `length` 进行赋值或更新。

### Lines 221-240 / 第 221-240 行

````python

    f.seek(absolute_start)
    return f.read(length)


def _process_output_file(
    output_file: str,
    output_data: _OutputFileData,
    input_files_data: dict[str, _InputFileData],
) -> None:
    """
    Process a single output file by writing tensor data from input files using direct reads.

    This function is designed to be run in parallel for different output files.

    Args:
        output_file: Path to the output file
        output_data: Metadata for the output file
        input_files_data: Dictionary mapping input file paths to their metadata
    """
````

- **L221** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L222** EN: Calls `f.seek` as part of the current workflow. | CN: 在当前流程中调用 `f.seek`。
- **L223** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L224** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L225** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L226** EN: Defines function `_process_output_file`. | CN: 定义函数 `_process_output_file`。
- **L227** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。
- **L228** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。
- **L229** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。
- **L230** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。
- **L231** EN: Starts the docstring for the function _process_output_file. | CN: 开始定义 function _process_output_file 的文档字符串。
- **L232** EN: Continues the docstring text for the function _process_output_file. | CN: 继续补充 function _process_output_file 的文档字符串内容。
- **L233** EN: Continues the docstring text for the function _process_output_file. | CN: 继续补充 function _process_output_file 的文档字符串内容。
- **L234** EN: Continues the docstring text for the function _process_output_file. | CN: 继续补充 function _process_output_file 的文档字符串内容。
- **L235** EN: Continues the docstring text for the function _process_output_file. | CN: 继续补充 function _process_output_file 的文档字符串内容。
- **L236** EN: Continues the docstring text for the function _process_output_file. | CN: 继续补充 function _process_output_file 的文档字符串内容。
- **L237** EN: Continues the docstring text for the function _process_output_file. | CN: 继续补充 function _process_output_file 的文档字符串内容。
- **L238** EN: Continues the docstring text for the function _process_output_file. | CN: 继续补充 function _process_output_file 的文档字符串内容。
- **L239** EN: Continues the docstring text for the function _process_output_file. | CN: 继续补充 function _process_output_file 的文档字符串内容。
- **L240** EN: Closes the docstring for the function _process_output_file. | CN: 结束 function _process_output_file 的文档字符串。

### Lines 241-260 / 第 241-260 行

````python

    sorted_tensors = sorted(
        output_data.fqn_data.items(), key=lambda x: x[1].offset_in_file
    )

    file_handles = {}
    dcp_metadata = {}
    for safetensors_file, file_data in input_files_data.items():
        dcp_metadata[safetensors_file] = _get_dcp_custom_metadata(file_data.metadata)

    try:
        # Open all input files for reading
        for safetensors_file in input_files_data:
            file_handles[safetensors_file] = open(safetensors_file, "rb")  # noqa: SIM115

        with open(output_file, "r+b") as output_stream:
            output_stream.seek(0, os.SEEK_END)
            # Process each tensor in sequential output order
            for tensor_fqn, tensor_fqn_data in sorted_tensors:
                full_tensor_mv = memoryview(
````

- **L241** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L242** EN: Assigns or updates `sorted_tensors`. | CN: 对 `sorted_tensors` 进行赋值或更新。
- **L243** EN: Calls `output_data.fqn_data.items` as part of the current workflow. | CN: 在当前流程中调用 `output_data.fqn_data.items`。
- **L244** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L245** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L246** EN: Assigns or updates `file_handles`. | CN: 对 `file_handles` 进行赋值或更新。
- **L247** EN: Assigns or updates `dcp_metadata`. | CN: 对 `dcp_metadata` 进行赋值或更新。
- **L248** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L249** EN: Assigns or updates `dcp_metadata[safetensors_file]`. | CN: 对 `dcp_metadata[safetensors_file]` 进行赋值或更新。
- **L250** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L251** EN: Starts a protected block for exception handling. | CN: 开始一个受保护的异常处理代码块。
- **L252** EN: Keeps the inline comment or directive: Open all input files for reading | CN: 保留这一行注释或指令：Open all input files for reading
- **L253** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L254** EN: Assigns or updates `file_handles[safetensors_file]`. | CN: 对 `file_handles[safetensors_file]` 进行赋值或更新。
- **L255** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L256** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L257** EN: Calls `output_stream.seek` as part of the current workflow. | CN: 在当前流程中调用 `output_stream.seek`。
- **L258** EN: Keeps the inline comment or directive: Process each tensor in sequential output order | CN: 保留这一行注释或指令：Process each tensor in sequential output order
- **L259** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L260** EN: Assigns or updates `full_tensor_mv`. | CN: 对 `full_tensor_mv` 进行赋值或更新。

### Lines 261-280 / 第 261-280 行

````python
                    bytearray(
                        math.prod(tensor_fqn_data.shape_in_file)
                        * tensor_fqn_data.dtype_size
                    )
                )

                # Process each input safetensors file
                for safetensors_file in input_files_data:
                    file_metadata = input_files_data[safetensors_file].metadata
                    input_metadata_size = input_files_data[
                        safetensors_file
                    ].metadata_size

                    if tensor_fqn not in file_metadata:
                        continue

                    metadata = file_metadata[tensor_fqn]

                    data_offsets = metadata[DATA_OFFSETS_KEY]

````

- **L261** EN: Calls `bytearray` as part of the current workflow. | CN: 在当前流程中调用 `bytearray`。
- **L262** EN: Calls `math.prod` as part of the current workflow. | CN: 在当前流程中调用 `math.prod`。
- **L263** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。
- **L264** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L265** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L266** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L267** EN: Keeps the inline comment or directive: Process each input safetensors file | CN: 保留这一行注释或指令：Process each input safetensors file
- **L268** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L269** EN: Assigns or updates `file_metadata`. | CN: 对 `file_metadata` 进行赋值或更新。
- **L270** EN: Assigns or updates `input_metadata_size`. | CN: 对 `input_metadata_size` 进行赋值或更新。
- **L271** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。
- **L272** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。
- **L273** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L274** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L275** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L276** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L277** EN: Assigns or updates `metadata`. | CN: 对 `metadata` 进行赋值或更新。
- **L278** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L279** EN: Assigns or updates `data_offsets`. | CN: 对 `data_offsets` 进行赋值或更新。
- **L280** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 281-300 / 第 281-300 行

````python
                    # Use explicit reads to fetch tensor data efficiently
                    data_to_write = _read_tensor_data(
                        file_handles[safetensors_file],
                        data_offsets[0],
                        data_offsets[1],
                        input_metadata_size,
                    )

                    # Get the offsets of this tensor shard within the full tensor
                    # pyrefly: ignore [unsupported-operation]
                    fqn_custom_metadata = dcp_metadata[safetensors_file][tensor_fqn]  # type: ignore[index]
                    offsets_of_tensor_being_read = fqn_custom_metadata[
                        SAVED_OFFSETS_KEY
                    ]  # type: ignore[index]

                    # Write this tensor shard to the appropriate position in the output file
                    _write_sub_tensor_to_file_optimized(
                        full_tensor_mv,
                        data_to_write,
                        tensor_fqn_data.dtype_size,  # Size of each element in bytes
````

- **L281** EN: Keeps the inline comment or directive: Use explicit reads to fetch tensor data efficiently | CN: 保留这一行注释或指令：Use explicit reads to fetch tensor data efficiently
- **L282** EN: Assigns or updates `data_to_write`. | CN: 对 `data_to_write` 进行赋值或更新。
- **L283** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。
- **L284** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。
- **L285** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。
- **L286** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。
- **L287** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L288** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L289** EN: Keeps the inline comment or directive: Get the offsets of this tensor shard within the full tensor | CN: 保留这一行注释或指令：Get the offsets of this tensor shard within the full tensor
- **L290** EN: Keeps the inline comment or directive: pyrefly: ignore [unsupported-operation] | CN: 保留这一行注释或指令：pyrefly: ignore [unsupported-operation]
- **L291** EN: Assigns or updates `fqn_custom_metadata`. | CN: 对 `fqn_custom_metadata` 进行赋值或更新。
- **L292** EN: Assigns or updates `offsets_of_tensor_being_read`. | CN: 对 `offsets_of_tensor_being_read` 进行赋值或更新。
- **L293** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。
- **L294** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。
- **L295** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L296** EN: Keeps the inline comment or directive: Write this tensor shard to the appropriate position in the output file | CN: 保留这一行注释或指令：Write this tensor shard to the appropriate position in the output file
- **L297** EN: Calls `_write_sub_tensor_to_file_optimized` as part of the current workflow. | CN: 在当前流程中调用 `_write_sub_tensor_to_file_optimized`。
- **L298** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。
- **L299** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。
- **L300** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。

### Lines 301-320 / 第 301-320 行

````python
                        tensor_fqn_data.shape_in_file,  # Full tensor shape
                        offsets_of_tensor_being_read,  # Where this shard belongs in the full tensor
                        metadata[SHAPE_KEY],  # Shape of this shard
                    )

                output_stream.write(full_tensor_mv)

    finally:
        for f in file_handles.values():
            f.close()


def _write_data(
    input_files_data: dict[str, _InputFileData],
    output_files_data: dict[str, _OutputFileData],
    num_threads: int = 1,
) -> None:
    """
    Write tensor data from input files to the output files using memory mapping.

````

- **L301** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。
- **L302** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。
- **L303** EN: Continues the implementation inside function `_process_output_file`. | CN: 继续说明函数 `_process_output_file` 内部的实现。
- **L304** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L305** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L306** EN: Calls `output_stream.write` as part of the current workflow. | CN: 在当前流程中调用 `output_stream.write`。
- **L307** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L308** EN: Runs cleanup logic regardless of exceptions. | CN: 无论是否出现异常，都执行清理逻辑。
- **L309** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L310** EN: Calls `f.close` as part of the current workflow. | CN: 在当前流程中调用 `f.close`。
- **L311** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L312** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L313** EN: Defines function `_write_data`. | CN: 定义函数 `_write_data`。
- **L314** EN: Continues the implementation inside function `_write_data`. | CN: 继续说明函数 `_write_data` 内部的实现。
- **L315** EN: Continues the implementation inside function `_write_data`. | CN: 继续说明函数 `_write_data` 内部的实现。
- **L316** EN: Assigns or updates `num_threads`. | CN: 对 `num_threads` 进行赋值或更新。
- **L317** EN: Continues the implementation inside function `_write_data`. | CN: 继续说明函数 `_write_data` 内部的实现。
- **L318** EN: Starts the docstring for the function _write_data. | CN: 开始定义 function _write_data 的文档字符串。
- **L319** EN: Continues the docstring text for the function _write_data. | CN: 继续补充 function _write_data 的文档字符串内容。
- **L320** EN: Continues the docstring text for the function _write_data. | CN: 继续补充 function _write_data 的文档字符串内容。

### Lines 321-340 / 第 321-340 行

````python
    This function reads tensor data from each input file and writes it to the appropriate
    position in the output files based on the tensor's offsets. When num_threads > 1,
    the work is split across threads with each thread handling a different output file.

    Args:
        input_files_data: Dictionary mapping input file paths to their metadata
        output_files_data: Dictionary mapping output file paths to their metadata
        num_threads: Number of threads to use for parallel processing
    """
    if num_threads <= 1 or len(output_files_data) <= 1:
        # Sequential processing
        for output_file, output_data in output_files_data.items():
            _process_output_file(output_file, output_data, input_files_data)
    else:
        # Parallel processing with ThreadPoolExecutor
        with concurrent.futures.ThreadPoolExecutor(
            max_workers=min(num_threads, len(output_files_data))
        ) as executor:
            futures = []
            for output_file, output_data in output_files_data.items():
````

- **L321** EN: Continues the docstring text for the function _write_data. | CN: 继续补充 function _write_data 的文档字符串内容。
- **L322** EN: Continues the docstring text for the function _write_data. | CN: 继续补充 function _write_data 的文档字符串内容。
- **L323** EN: Continues the docstring text for the function _write_data. | CN: 继续补充 function _write_data 的文档字符串内容。
- **L324** EN: Continues the docstring text for the function _write_data. | CN: 继续补充 function _write_data 的文档字符串内容。
- **L325** EN: Continues the docstring text for the function _write_data. | CN: 继续补充 function _write_data 的文档字符串内容。
- **L326** EN: Continues the docstring text for the function _write_data. | CN: 继续补充 function _write_data 的文档字符串内容。
- **L327** EN: Continues the docstring text for the function _write_data. | CN: 继续补充 function _write_data 的文档字符串内容。
- **L328** EN: Continues the docstring text for the function _write_data. | CN: 继续补充 function _write_data 的文档字符串内容。
- **L329** EN: Closes the docstring for the function _write_data. | CN: 结束 function _write_data 的文档字符串。
- **L330** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L331** EN: Keeps the inline comment or directive: Sequential processing | CN: 保留这一行注释或指令：Sequential processing
- **L332** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L333** EN: Calls `_process_output_file` as part of the current workflow. | CN: 在当前流程中调用 `_process_output_file`。
- **L334** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L335** EN: Keeps the inline comment or directive: Parallel processing with ThreadPoolExecutor | CN: 保留这一行注释或指令：Parallel processing with ThreadPoolExecutor
- **L336** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L337** EN: Assigns or updates `max_workers`. | CN: 对 `max_workers` 进行赋值或更新。
- **L338** EN: Continues the implementation inside function `_write_data`. | CN: 继续说明函数 `_write_data` 内部的实现。
- **L339** EN: Assigns or updates `futures`. | CN: 对 `futures` 进行赋值或更新。
- **L340** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。

### Lines 341-360 / 第 341-360 行

````python
                futures.append(
                    executor.submit(
                        _process_output_file,
                        output_file,
                        output_data,
                        input_files_data,
                    )
                )

            # Wait for all futures to complete
            for future in concurrent.futures.as_completed(futures):
                # Handle any exceptions that might have occurred
                try:
                    future.result()
                except Exception as e:
                    print(f"Error processing output file: {e}")
                    raise


def _write_sub_tensor_to_file_optimized(
````

- **L341** EN: Calls `futures.append` as part of the current workflow. | CN: 在当前流程中调用 `futures.append`。
- **L342** EN: Calls `executor.submit` as part of the current workflow. | CN: 在当前流程中调用 `executor.submit`。
- **L343** EN: Continues the implementation inside function `_write_data`. | CN: 继续说明函数 `_write_data` 内部的实现。
- **L344** EN: Continues the implementation inside function `_write_data`. | CN: 继续说明函数 `_write_data` 内部的实现。
- **L345** EN: Continues the implementation inside function `_write_data`. | CN: 继续说明函数 `_write_data` 内部的实现。
- **L346** EN: Continues the implementation inside function `_write_data`. | CN: 继续说明函数 `_write_data` 内部的实现。
- **L347** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L348** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L349** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L350** EN: Keeps the inline comment or directive: Wait for all futures to complete | CN: 保留这一行注释或指令：Wait for all futures to complete
- **L351** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L352** EN: Keeps the inline comment or directive: Handle any exceptions that might have occurred | CN: 保留这一行注释或指令：Handle any exceptions that might have occurred
- **L353** EN: Starts a protected block for exception handling. | CN: 开始一个受保护的异常处理代码块。
- **L354** EN: Calls `future.result` as part of the current workflow. | CN: 在当前流程中调用 `future.result`。
- **L355** EN: Handles a matching exception path. | CN: 处理匹配到的异常路径。
- **L356** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L357** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L358** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L359** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L360** EN: Defines function `_write_sub_tensor_to_file_optimized`. | CN: 定义函数 `_write_sub_tensor_to_file_optimized`。

### Lines 361-380 / 第 361-380 行

````python
    full_tensor_mv: memoryview,
    sub_tensor_bytes: bytes,
    element_size: int,
    tensor_shape: list[int],
    sub_tensor_offsets: list[int],
    sub_tensor_shape: list[int],
) -> None:
    """
    Optimized version that writes the maximum number of contiguous bytes possible.

    Uses a unified algorithm that calculates the maximum contiguous bytes that can be
    written in each iteration and continues until the entire subtensor is written.
    Handles all sharding patterns efficiently:
    - Full sub-tensor at once for row-wise sharding
    - Row-by-row for column-wise sharding
    - Optimized chunks for other patterns

    Args:
        full_tensor_mv: Buffer to write the full tensor to
        sub_tensor_bytes: Raw tensor data as bytes
````

- **L361** EN: Continues the implementation inside function `_write_sub_tensor_to_file_optimized`. | CN: 继续说明函数 `_write_sub_tensor_to_file_optimized` 内部的实现。
- **L362** EN: Continues the implementation inside function `_write_sub_tensor_to_file_optimized`. | CN: 继续说明函数 `_write_sub_tensor_to_file_optimized` 内部的实现。
- **L363** EN: Continues the implementation inside function `_write_sub_tensor_to_file_optimized`. | CN: 继续说明函数 `_write_sub_tensor_to_file_optimized` 内部的实现。
- **L364** EN: Continues the implementation inside function `_write_sub_tensor_to_file_optimized`. | CN: 继续说明函数 `_write_sub_tensor_to_file_optimized` 内部的实现。
- **L365** EN: Continues the implementation inside function `_write_sub_tensor_to_file_optimized`. | CN: 继续说明函数 `_write_sub_tensor_to_file_optimized` 内部的实现。
- **L366** EN: Continues the implementation inside function `_write_sub_tensor_to_file_optimized`. | CN: 继续说明函数 `_write_sub_tensor_to_file_optimized` 内部的实现。
- **L367** EN: Continues the implementation inside function `_write_sub_tensor_to_file_optimized`. | CN: 继续说明函数 `_write_sub_tensor_to_file_optimized` 内部的实现。
- **L368** EN: Starts the docstring for the function _write_sub_tensor_to_file_optimized. | CN: 开始定义 function _write_sub_tensor_to_file_optimized 的文档字符串。
- **L369** EN: Continues the docstring text for the function _write_sub_tensor_to_file_optimized. | CN: 继续补充 function _write_sub_tensor_to_file_optimized 的文档字符串内容。
- **L370** EN: Continues the docstring text for the function _write_sub_tensor_to_file_optimized. | CN: 继续补充 function _write_sub_tensor_to_file_optimized 的文档字符串内容。
- **L371** EN: Continues the docstring text for the function _write_sub_tensor_to_file_optimized. | CN: 继续补充 function _write_sub_tensor_to_file_optimized 的文档字符串内容。
- **L372** EN: Continues the docstring text for the function _write_sub_tensor_to_file_optimized. | CN: 继续补充 function _write_sub_tensor_to_file_optimized 的文档字符串内容。
- **L373** EN: Continues the docstring text for the function _write_sub_tensor_to_file_optimized. | CN: 继续补充 function _write_sub_tensor_to_file_optimized 的文档字符串内容。
- **L374** EN: Continues the docstring text for the function _write_sub_tensor_to_file_optimized. | CN: 继续补充 function _write_sub_tensor_to_file_optimized 的文档字符串内容。
- **L375** EN: Continues the docstring text for the function _write_sub_tensor_to_file_optimized. | CN: 继续补充 function _write_sub_tensor_to_file_optimized 的文档字符串内容。
- **L376** EN: Continues the docstring text for the function _write_sub_tensor_to_file_optimized. | CN: 继续补充 function _write_sub_tensor_to_file_optimized 的文档字符串内容。
- **L377** EN: Continues the docstring text for the function _write_sub_tensor_to_file_optimized. | CN: 继续补充 function _write_sub_tensor_to_file_optimized 的文档字符串内容。
- **L378** EN: Continues the docstring text for the function _write_sub_tensor_to_file_optimized. | CN: 继续补充 function _write_sub_tensor_to_file_optimized 的文档字符串内容。
- **L379** EN: Continues the docstring text for the function _write_sub_tensor_to_file_optimized. | CN: 继续补充 function _write_sub_tensor_to_file_optimized 的文档字符串内容。
- **L380** EN: Continues the docstring text for the function _write_sub_tensor_to_file_optimized. | CN: 继续补充 function _write_sub_tensor_to_file_optimized 的文档字符串内容。

### Lines 381-400 / 第 381-400 行

````python
        element_size: Size of each element in bytes
        tensor_shape: Shape of the full tensor
        sub_tensor_offsets: Starting offsets of the sub-tensor within the full tensor
        sub_tensor_shape: Shape of the sub-tensor
    """
    # Handle empty tensors
    if not tensor_shape or not sub_tensor_shape:
        return

    # Calculate tensor strides for efficient indexing
    tensor_strides = [1]
    for i in range(len(tensor_shape) - 1, 0, -1):
        tensor_strides.insert(0, tensor_strides[0] * tensor_shape[i])

    sub_tensor_strides = [1]
    for i in range(len(sub_tensor_shape) - 1, 0, -1):
        sub_tensor_strides.insert(0, sub_tensor_strides[0] * sub_tensor_shape[i])

    total_elements = math.prod(sub_tensor_shape)

````

- **L381** EN: Continues the docstring text for the function _write_sub_tensor_to_file_optimized. | CN: 继续补充 function _write_sub_tensor_to_file_optimized 的文档字符串内容。
- **L382** EN: Continues the docstring text for the function _write_sub_tensor_to_file_optimized. | CN: 继续补充 function _write_sub_tensor_to_file_optimized 的文档字符串内容。
- **L383** EN: Continues the docstring text for the function _write_sub_tensor_to_file_optimized. | CN: 继续补充 function _write_sub_tensor_to_file_optimized 的文档字符串内容。
- **L384** EN: Continues the docstring text for the function _write_sub_tensor_to_file_optimized. | CN: 继续补充 function _write_sub_tensor_to_file_optimized 的文档字符串内容。
- **L385** EN: Closes the docstring for the function _write_sub_tensor_to_file_optimized. | CN: 结束 function _write_sub_tensor_to_file_optimized 的文档字符串。
- **L386** EN: Keeps the inline comment or directive: Handle empty tensors | CN: 保留这一行注释或指令：Handle empty tensors
- **L387** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L388** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L389** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L390** EN: Keeps the inline comment or directive: Calculate tensor strides for efficient indexing | CN: 保留这一行注释或指令：Calculate tensor strides for efficient indexing
- **L391** EN: Assigns or updates `tensor_strides`. | CN: 对 `tensor_strides` 进行赋值或更新。
- **L392** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L393** EN: Calls `tensor_strides.insert` as part of the current workflow. | CN: 在当前流程中调用 `tensor_strides.insert`。
- **L394** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L395** EN: Assigns or updates `sub_tensor_strides`. | CN: 对 `sub_tensor_strides` 进行赋值或更新。
- **L396** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L397** EN: Calls `sub_tensor_strides.insert` as part of the current workflow. | CN: 在当前流程中调用 `sub_tensor_strides.insert`。
- **L398** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L399** EN: Assigns or updates `total_elements`. | CN: 对 `total_elements` 进行赋值或更新。
- **L400** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 401-420 / 第 401-420 行

````python
    elements_written = 0
    while elements_written < total_elements:
        # Convert linear index to multi-dimensional indices
        temp_idx = elements_written
        indices = []
        for dim_size in reversed(sub_tensor_shape):
            indices.append(temp_idx % dim_size)
            temp_idx //= dim_size
        indices.reverse()

        # Calculate maximum contiguous elements we can write from this position
        max_contiguous = _calculate_max_contiguous_elements(
            indices, sub_tensor_shape, tensor_shape
        )

        # Calculate source position in bytes
        src_pos = sum(idx * stride for idx, stride in zip(indices, sub_tensor_strides))
        src_byte_offset = src_pos * element_size

        # Calculate destination position in bytes
````

- **L401** EN: Assigns or updates `elements_written`. | CN: 对 `elements_written` 进行赋值或更新。
- **L402** EN: Starts a loop controlled by a condition. | CN: 开始一个由条件控制的循环。
- **L403** EN: Keeps the inline comment or directive: Convert linear index to multi-dimensional indices | CN: 保留这一行注释或指令：Convert linear index to multi-dimensional indices
- **L404** EN: Assigns or updates `temp_idx`. | CN: 对 `temp_idx` 进行赋值或更新。
- **L405** EN: Assigns or updates `indices`. | CN: 对 `indices` 进行赋值或更新。
- **L406** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L407** EN: Calls `indices.append` as part of the current workflow. | CN: 在当前流程中调用 `indices.append`。
- **L408** EN: Continues the implementation inside function `_write_sub_tensor_to_file_optimized`. | CN: 继续说明函数 `_write_sub_tensor_to_file_optimized` 内部的实现。
- **L409** EN: Calls `indices.reverse` as part of the current workflow. | CN: 在当前流程中调用 `indices.reverse`。
- **L410** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L411** EN: Keeps the inline comment or directive: Calculate maximum contiguous elements we can write from this position | CN: 保留这一行注释或指令：Calculate maximum contiguous elements we can write from this position
- **L412** EN: Assigns or updates `max_contiguous`. | CN: 对 `max_contiguous` 进行赋值或更新。
- **L413** EN: Continues the implementation inside function `_write_sub_tensor_to_file_optimized`. | CN: 继续说明函数 `_write_sub_tensor_to_file_optimized` 内部的实现。
- **L414** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L415** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L416** EN: Keeps the inline comment or directive: Calculate source position in bytes | CN: 保留这一行注释或指令：Calculate source position in bytes
- **L417** EN: Assigns or updates `src_pos`. | CN: 对 `src_pos` 进行赋值或更新。
- **L418** EN: Assigns or updates `src_byte_offset`. | CN: 对 `src_byte_offset` 进行赋值或更新。
- **L419** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L420** EN: Keeps the inline comment or directive: Calculate destination position in bytes | CN: 保留这一行注释或指令：Calculate destination position in bytes

### Lines 421-440 / 第 421-440 行

````python
        dest_indices = [
            idx + offset for idx, offset in zip(indices, sub_tensor_offsets)
        ]
        dest_pos = sum(
            idx * stride for idx, stride in zip(dest_indices, tensor_strides)
        )
        dest_byte_offset = dest_pos * element_size

        # Write the contiguous chunk
        bytes_to_write = max_contiguous * element_size
        chunk_data = sub_tensor_bytes[
            src_byte_offset : src_byte_offset + bytes_to_write
        ]
        full_tensor_mv[dest_byte_offset : dest_byte_offset + bytes_to_write] = (
            chunk_data
        )

        elements_written += max_contiguous


````

- **L421** EN: Assigns or updates `dest_indices`. | CN: 对 `dest_indices` 进行赋值或更新。
- **L422** EN: Continues the implementation inside function `_write_sub_tensor_to_file_optimized`. | CN: 继续说明函数 `_write_sub_tensor_to_file_optimized` 内部的实现。
- **L423** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L424** EN: Assigns or updates `dest_pos`. | CN: 对 `dest_pos` 进行赋值或更新。
- **L425** EN: Continues the implementation inside function `_write_sub_tensor_to_file_optimized`. | CN: 继续说明函数 `_write_sub_tensor_to_file_optimized` 内部的实现。
- **L426** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L427** EN: Assigns or updates `dest_byte_offset`. | CN: 对 `dest_byte_offset` 进行赋值或更新。
- **L428** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L429** EN: Keeps the inline comment or directive: Write the contiguous chunk | CN: 保留这一行注释或指令：Write the contiguous chunk
- **L430** EN: Assigns or updates `bytes_to_write`. | CN: 对 `bytes_to_write` 进行赋值或更新。
- **L431** EN: Assigns or updates `chunk_data`. | CN: 对 `chunk_data` 进行赋值或更新。
- **L432** EN: Continues the implementation inside function `_write_sub_tensor_to_file_optimized`. | CN: 继续说明函数 `_write_sub_tensor_to_file_optimized` 内部的实现。
- **L433** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L434** EN: Assigns or updates `full_tensor_mv[dest_byte_offset`. | CN: 对 `full_tensor_mv[dest_byte_offset` 进行赋值或更新。
- **L435** EN: Continues the implementation inside function `_write_sub_tensor_to_file_optimized`. | CN: 继续说明函数 `_write_sub_tensor_to_file_optimized` 内部的实现。
- **L436** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L437** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L438** EN: Continues the implementation inside function `_write_sub_tensor_to_file_optimized`. | CN: 继续说明函数 `_write_sub_tensor_to_file_optimized` 内部的实现。
- **L439** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L440** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 441-460 / 第 441-460 行

````python
def _calculate_max_contiguous_elements(
    indices: list[int],
    sub_tensor_shape: list[int],
    tensor_shape: list[int],
) -> int:
    """
    Calculate the maximum number of contiguous elements that can be written from current position.

    This determines the largest chunk by checking how elements are laid out in memory
    and finding natural boundaries where contiguity breaks.

    Args:
        indices: Current position indices in the sub-tensor
        sub_tensor_shape: Shape of the sub-tensor being written
        tensor_shape: Shape of the full tensor

    Raises:
        ValueError: If input lists are empty, have mismatched lengths, or contain invalid values
    """
    # Validate input lists are not empty
````

- **L441** EN: Defines function `_calculate_max_contiguous_elements`. | CN: 定义函数 `_calculate_max_contiguous_elements`。
- **L442** EN: Continues the implementation inside function `_calculate_max_contiguous_elements`. | CN: 继续说明函数 `_calculate_max_contiguous_elements` 内部的实现。
- **L443** EN: Continues the implementation inside function `_calculate_max_contiguous_elements`. | CN: 继续说明函数 `_calculate_max_contiguous_elements` 内部的实现。
- **L444** EN: Continues the implementation inside function `_calculate_max_contiguous_elements`. | CN: 继续说明函数 `_calculate_max_contiguous_elements` 内部的实现。
- **L445** EN: Continues the implementation inside function `_calculate_max_contiguous_elements`. | CN: 继续说明函数 `_calculate_max_contiguous_elements` 内部的实现。
- **L446** EN: Starts the docstring for the function _calculate_max_contiguous_elements. | CN: 开始定义 function _calculate_max_contiguous_elements 的文档字符串。
- **L447** EN: Continues the docstring text for the function _calculate_max_contiguous_elements. | CN: 继续补充 function _calculate_max_contiguous_elements 的文档字符串内容。
- **L448** EN: Continues the docstring text for the function _calculate_max_contiguous_elements. | CN: 继续补充 function _calculate_max_contiguous_elements 的文档字符串内容。
- **L449** EN: Continues the docstring text for the function _calculate_max_contiguous_elements. | CN: 继续补充 function _calculate_max_contiguous_elements 的文档字符串内容。
- **L450** EN: Continues the docstring text for the function _calculate_max_contiguous_elements. | CN: 继续补充 function _calculate_max_contiguous_elements 的文档字符串内容。
- **L451** EN: Continues the docstring text for the function _calculate_max_contiguous_elements. | CN: 继续补充 function _calculate_max_contiguous_elements 的文档字符串内容。
- **L452** EN: Continues the docstring text for the function _calculate_max_contiguous_elements. | CN: 继续补充 function _calculate_max_contiguous_elements 的文档字符串内容。
- **L453** EN: Continues the docstring text for the function _calculate_max_contiguous_elements. | CN: 继续补充 function _calculate_max_contiguous_elements 的文档字符串内容。
- **L454** EN: Continues the docstring text for the function _calculate_max_contiguous_elements. | CN: 继续补充 function _calculate_max_contiguous_elements 的文档字符串内容。
- **L455** EN: Continues the docstring text for the function _calculate_max_contiguous_elements. | CN: 继续补充 function _calculate_max_contiguous_elements 的文档字符串内容。
- **L456** EN: Continues the docstring text for the function _calculate_max_contiguous_elements. | CN: 继续补充 function _calculate_max_contiguous_elements 的文档字符串内容。
- **L457** EN: Continues the docstring text for the function _calculate_max_contiguous_elements. | CN: 继续补充 function _calculate_max_contiguous_elements 的文档字符串内容。
- **L458** EN: Continues the docstring text for the function _calculate_max_contiguous_elements. | CN: 继续补充 function _calculate_max_contiguous_elements 的文档字符串内容。
- **L459** EN: Closes the docstring for the function _calculate_max_contiguous_elements. | CN: 结束 function _calculate_max_contiguous_elements 的文档字符串。
- **L460** EN: Keeps the inline comment or directive: Validate input lists are not empty | CN: 保留这一行注释或指令：Validate input lists are not empty

### Lines 461-480 / 第 461-480 行

````python
    if not indices or not sub_tensor_shape or not tensor_shape:
        raise ValueError("Input lists cannot be empty")

    # Validate all lists have the same length (same number of dimensions)
    if not (len(indices) == len(sub_tensor_shape) == len(tensor_shape)):
        raise ValueError(
            f"All input lists must have the same length. Got indices: {len(indices)}, "
            f"sub_tensor_shape: {len(sub_tensor_shape)}, tensor_shape: {len(tensor_shape)}"
        )

    # Validate indices are within bounds of sub_tensor_shape
    for i, (idx, sub_dim) in enumerate(zip(indices, sub_tensor_shape)):
        if idx >= sub_dim:
            raise ValueError(
                f"Index {idx} at dimension {i} is out of bounds for sub-tensor shape {sub_tensor_shape}"
            )

    # Validate sub_tensor dimensions don't exceed tensor dimensions
    for i, (sub_dim, tensor_dim) in enumerate(zip(sub_tensor_shape, tensor_shape)):
        if sub_dim > tensor_dim:
````

- **L461** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L462** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L463** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L464** EN: Keeps the inline comment or directive: Validate all lists have the same length (same number of dimensions) | CN: 保留这一行注释或指令：Validate all lists have the same length (same number of dimensions)
- **L465** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L466** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L467** EN: Continues the implementation inside function `_calculate_max_contiguous_elements`. | CN: 继续说明函数 `_calculate_max_contiguous_elements` 内部的实现。
- **L468** EN: Continues the implementation inside function `_calculate_max_contiguous_elements`. | CN: 继续说明函数 `_calculate_max_contiguous_elements` 内部的实现。
- **L469** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L470** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L471** EN: Keeps the inline comment or directive: Validate indices are within bounds of sub_tensor_shape | CN: 保留这一行注释或指令：Validate indices are within bounds of sub_tensor_shape
- **L472** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L473** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L474** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L475** EN: Continues the implementation inside function `_calculate_max_contiguous_elements`. | CN: 继续说明函数 `_calculate_max_contiguous_elements` 内部的实现。
- **L476** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L477** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L478** EN: Keeps the inline comment or directive: Validate sub_tensor dimensions don't exceed tensor dimensions | CN: 保留这一行注释或指令：Validate sub_tensor dimensions don't exceed tensor dimensions
- **L479** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L480** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 481-500 / 第 481-500 行

````python
            raise ValueError(
                f"Sub-tensor dimension {sub_dim} at position {i} exceeds tensor dimension {tensor_dim}"
            )

    # Start with elements remaining in the last dimension
    max_contiguous = sub_tensor_shape[-1] - indices[-1]

    # Check if we can extend across multiple dimensions
    # We can write across dimension boundaries if we're writing complete "rows"
    # and the layout in destination tensor maintains contiguity

    # For 2D case: check if we can write multiple complete rows
    if len(sub_tensor_shape) >= 2:
        # If we're at the start of a row and can write complete rows
        if indices[-1] == 0:  # At start of last dimension (column)
            rows_remaining = sub_tensor_shape[-2] - indices[-2]  # Rows left to write

            # Check if writing complete rows maintains contiguity in destination
            # This is true for row-wise sharding or when sub-tensor spans full width
            if sub_tensor_shape[-1] == tensor_shape[-1]:  # Full width
````

- **L481** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L482** EN: Continues the implementation inside function `_calculate_max_contiguous_elements`. | CN: 继续说明函数 `_calculate_max_contiguous_elements` 内部的实现。
- **L483** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L484** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L485** EN: Keeps the inline comment or directive: Start with elements remaining in the last dimension | CN: 保留这一行注释或指令：Start with elements remaining in the last dimension
- **L486** EN: Assigns or updates `max_contiguous`. | CN: 对 `max_contiguous` 进行赋值或更新。
- **L487** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L488** EN: Keeps the inline comment or directive: Check if we can extend across multiple dimensions | CN: 保留这一行注释或指令：Check if we can extend across multiple dimensions
- **L489** EN: Keeps the inline comment or directive: We can write across dimension boundaries if we're writing complete "rows" | CN: 保留这一行注释或指令：We can write across dimension boundaries if we're writing complete "rows"
- **L490** EN: Keeps the inline comment or directive: and the layout in destination tensor maintains contiguity | CN: 保留这一行注释或指令：and the layout in destination tensor maintains contiguity
- **L491** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L492** EN: Keeps the inline comment or directive: For 2D case: check if we can write multiple complete rows | CN: 保留这一行注释或指令：For 2D case: check if we can write multiple complete rows
- **L493** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L494** EN: Keeps the inline comment or directive: If we're at the start of a row and can write complete rows | CN: 保留这一行注释或指令：If we're at the start of a row and can write complete rows
- **L495** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L496** EN: Assigns or updates `rows_remaining`. | CN: 对 `rows_remaining` 进行赋值或更新。
- **L497** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L498** EN: Keeps the inline comment or directive: Check if writing complete rows maintains contiguity in destination | CN: 保留这一行注释或指令：Check if writing complete rows maintains contiguity in destination
- **L499** EN: Keeps the inline comment or directive: This is true for row-wise sharding or when sub-tensor spans full width | CN: 保留这一行注释或指令：This is true for row-wise sharding or when sub-tensor spans full width
- **L500** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 501-520 / 第 501-520 行

````python
                max_contiguous = rows_remaining * sub_tensor_shape[-1]

            # For higher dimensions, check if we can extend further
            if len(sub_tensor_shape) >= 3 and indices[-2] == 0:
                # Check if we can write complete 2D slices
                remaining_in_dim = sub_tensor_shape[-3] - indices[-3]
                if (
                    sub_tensor_shape[-1] == tensor_shape[-1]
                    and sub_tensor_shape[-2] == tensor_shape[-2]
                ):
                    max_contiguous = (
                        remaining_in_dim * sub_tensor_shape[-2] * sub_tensor_shape[-1]
                    )

    return max_contiguous


def _write_overall_metadata_file(
    output_dir: str,
    output_files_data: dict[str, _OutputFileData],
````

- **L501** EN: Assigns or updates `max_contiguous`. | CN: 对 `max_contiguous` 进行赋值或更新。
- **L502** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L503** EN: Keeps the inline comment or directive: For higher dimensions, check if we can extend further | CN: 保留这一行注释或指令：For higher dimensions, check if we can extend further
- **L504** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L505** EN: Keeps the inline comment or directive: Check if we can write complete 2D slices | CN: 保留这一行注释或指令：Check if we can write complete 2D slices
- **L506** EN: Assigns or updates `remaining_in_dim`. | CN: 对 `remaining_in_dim` 进行赋值或更新。
- **L507** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L508** EN: Continues the implementation inside function `_calculate_max_contiguous_elements`. | CN: 继续说明函数 `_calculate_max_contiguous_elements` 内部的实现。
- **L509** EN: Continues the implementation inside function `_calculate_max_contiguous_elements`. | CN: 继续说明函数 `_calculate_max_contiguous_elements` 内部的实现。
- **L510** EN: Continues the implementation inside function `_calculate_max_contiguous_elements`. | CN: 继续说明函数 `_calculate_max_contiguous_elements` 内部的实现。
- **L511** EN: Assigns or updates `max_contiguous`. | CN: 对 `max_contiguous` 进行赋值或更新。
- **L512** EN: Continues the implementation inside function `_calculate_max_contiguous_elements`. | CN: 继续说明函数 `_calculate_max_contiguous_elements` 内部的实现。
- **L513** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L514** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L515** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L516** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L517** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L518** EN: Defines function `_write_overall_metadata_file`. | CN: 定义函数 `_write_overall_metadata_file`。
- **L519** EN: Continues the implementation inside function `_write_overall_metadata_file`. | CN: 继续说明函数 `_write_overall_metadata_file` 内部的实现。
- **L520** EN: Continues the implementation inside function `_write_overall_metadata_file`. | CN: 继续说明函数 `_write_overall_metadata_file` 内部的实现。

### Lines 521-540 / 第 521-540 行

````python
) -> None:
    """
    Write the overall metadata file that maps tensor names to their file locations.

    This creates a model.safetensors.index.json file that HuggingFace models use
    to locate tensors across multiple files.

    Args:
        output_dir: Directory where the metadata file will be written
        output_files_data: Dictionary mapping output file paths to their metadata
    """
    total_size = 0
    weight_map = {}
    for output_path, value in output_files_data.items():
        for fqn, fqn_data in value.fqn_data.items():
            total_size += math.prod(fqn_data.shape_in_file) * fqn_data.dtype_size
            weight_map[fqn] = os.path.basename(output_path)

    metadata_to_write: dict[str, Any] = {}
    metadata_to_write["metadata"] = {"total_size": total_size}
````

- **L521** EN: Continues the implementation inside function `_write_overall_metadata_file`. | CN: 继续说明函数 `_write_overall_metadata_file` 内部的实现。
- **L522** EN: Starts the docstring for the function _write_overall_metadata_file. | CN: 开始定义 function _write_overall_metadata_file 的文档字符串。
- **L523** EN: Continues the docstring text for the function _write_overall_metadata_file. | CN: 继续补充 function _write_overall_metadata_file 的文档字符串内容。
- **L524** EN: Continues the docstring text for the function _write_overall_metadata_file. | CN: 继续补充 function _write_overall_metadata_file 的文档字符串内容。
- **L525** EN: Continues the docstring text for the function _write_overall_metadata_file. | CN: 继续补充 function _write_overall_metadata_file 的文档字符串内容。
- **L526** EN: Continues the docstring text for the function _write_overall_metadata_file. | CN: 继续补充 function _write_overall_metadata_file 的文档字符串内容。
- **L527** EN: Continues the docstring text for the function _write_overall_metadata_file. | CN: 继续补充 function _write_overall_metadata_file 的文档字符串内容。
- **L528** EN: Continues the docstring text for the function _write_overall_metadata_file. | CN: 继续补充 function _write_overall_metadata_file 的文档字符串内容。
- **L529** EN: Continues the docstring text for the function _write_overall_metadata_file. | CN: 继续补充 function _write_overall_metadata_file 的文档字符串内容。
- **L530** EN: Continues the docstring text for the function _write_overall_metadata_file. | CN: 继续补充 function _write_overall_metadata_file 的文档字符串内容。
- **L531** EN: Closes the docstring for the function _write_overall_metadata_file. | CN: 结束 function _write_overall_metadata_file 的文档字符串。
- **L532** EN: Assigns or updates `total_size`. | CN: 对 `total_size` 进行赋值或更新。
- **L533** EN: Assigns or updates `weight_map`. | CN: 对 `weight_map` 进行赋值或更新。
- **L534** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L535** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L536** EN: Continues the implementation inside function `_write_overall_metadata_file`. | CN: 继续说明函数 `_write_overall_metadata_file` 内部的实现。
- **L537** EN: Assigns or updates `weight_map[fqn]`. | CN: 对 `weight_map[fqn]` 进行赋值或更新。
- **L538** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L539** EN: Assigns or updates `metadata_to_write`. | CN: 对 `metadata_to_write` 进行赋值或更新。
- **L540** EN: Continues the implementation inside function `_write_overall_metadata_file`. | CN: 继续说明函数 `_write_overall_metadata_file` 内部的实现。

### Lines 541-560 / 第 541-560 行

````python
    metadata_to_write["weight_map"] = weight_map

    metadata_path = os.path.join(output_dir, f"{_metadata_fn}")
    with open(metadata_path, "w") as metadata_file:
        json.dump(metadata_to_write, metadata_file, indent=2)


def _consolidate_safetensors_files(
    input_dir: str,
    output_dir: str,
    fqn_to_file_mapping: dict[str, str],
    num_threads: int,
) -> dict[str, _OutputFileData]:
    output_files_data: dict[str, _OutputFileData] = {}
    # Create multiple output files based on the provided mapping
    for fqn, filename in fqn_to_file_mapping.items():
        output_path = os.path.join(output_dir, filename)

        if output_path not in output_files_data:
            output_files_data[output_path] = _OutputFileData(fqn_data={fqn: _FqnData()})
````

- **L541** EN: Continues the implementation inside function `_write_overall_metadata_file`. | CN: 继续说明函数 `_write_overall_metadata_file` 内部的实现。
- **L542** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L543** EN: Assigns or updates `metadata_path`. | CN: 对 `metadata_path` 进行赋值或更新。
- **L544** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L545** EN: Calls `json.dump` as part of the current workflow. | CN: 在当前流程中调用 `json.dump`。
- **L546** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L547** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L548** EN: Defines function `_consolidate_safetensors_files`. | CN: 定义函数 `_consolidate_safetensors_files`。
- **L549** EN: Continues the implementation inside function `_consolidate_safetensors_files`. | CN: 继续说明函数 `_consolidate_safetensors_files` 内部的实现。
- **L550** EN: Continues the implementation inside function `_consolidate_safetensors_files`. | CN: 继续说明函数 `_consolidate_safetensors_files` 内部的实现。
- **L551** EN: Continues the implementation inside function `_consolidate_safetensors_files`. | CN: 继续说明函数 `_consolidate_safetensors_files` 内部的实现。
- **L552** EN: Continues the implementation inside function `_consolidate_safetensors_files`. | CN: 继续说明函数 `_consolidate_safetensors_files` 内部的实现。
- **L553** EN: Continues the implementation inside function `_consolidate_safetensors_files`. | CN: 继续说明函数 `_consolidate_safetensors_files` 内部的实现。
- **L554** EN: Assigns or updates `output_files_data`. | CN: 对 `output_files_data` 进行赋值或更新。
- **L555** EN: Keeps the inline comment or directive: Create multiple output files based on the provided mapping | CN: 保留这一行注释或指令：Create multiple output files based on the provided mapping
- **L556** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L557** EN: Assigns or updates `output_path`. | CN: 对 `output_path` 进行赋值或更新。
- **L558** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L559** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L560** EN: Assigns or updates `output_files_data[output_path]`. | CN: 对 `output_files_data[output_path]` 进行赋值或更新。

### Lines 561-580 / 第 561-580 行

````python
        else:
            output_files_data[output_path].fqn_data[fqn] = _FqnData()

    # Find all safetensors files in the input directory
    safetensors_files = glob.glob(os.path.join(input_dir, f"*{SUFFIX}"))

    # Read metadata from all input files
    input_files_data: dict[str, _InputFileData] = {}
    for safetensor_file in safetensors_files:
        with open(safetensor_file, "rb") as f:
            metadata, size = _get_safetensors_file_metadata(f)
            input_files_data[safetensor_file] = _InputFileData(
                metadata_size=size, metadata=metadata
            )
    # Step 1: Parse metadata to determine tensor shapes and types
    _parse_input_metadata(input_files_data, output_files_data)

    # Step 2: Write metadata headers to output files
    _write_metadata(output_files_data)
    # Step 3: Write actual tensor data from input files to output files
````

- **L561** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L562** EN: Assigns or updates `output_files_data[output_path].fqn_data[fqn]`. | CN: 对 `output_files_data[output_path].fqn_data[fqn]` 进行赋值或更新。
- **L563** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L564** EN: Keeps the inline comment or directive: Find all safetensors files in the input directory | CN: 保留这一行注释或指令：Find all safetensors files in the input directory
- **L565** EN: Assigns or updates `safetensors_files`. | CN: 对 `safetensors_files` 进行赋值或更新。
- **L566** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L567** EN: Keeps the inline comment or directive: Read metadata from all input files | CN: 保留这一行注释或指令：Read metadata from all input files
- **L568** EN: Assigns or updates `input_files_data`. | CN: 对 `input_files_data` 进行赋值或更新。
- **L569** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L570** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L571** EN: Assigns or updates `metadata, size`. | CN: 对 `metadata, size` 进行赋值或更新。
- **L572** EN: Assigns or updates `input_files_data[safetensor_file]`. | CN: 对 `input_files_data[safetensor_file]` 进行赋值或更新。
- **L573** EN: Assigns or updates `metadata_size`. | CN: 对 `metadata_size` 进行赋值或更新。
- **L574** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L575** EN: Keeps the inline comment or directive: Step 1: Parse metadata to determine tensor shapes and types | CN: 保留这一行注释或指令：Step 1: Parse metadata to determine tensor shapes and types
- **L576** EN: Calls `_parse_input_metadata` as part of the current workflow. | CN: 在当前流程中调用 `_parse_input_metadata`。
- **L577** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L578** EN: Keeps the inline comment or directive: Step 2: Write metadata headers to output files | CN: 保留这一行注释或指令：Step 2: Write metadata headers to output files
- **L579** EN: Calls `_write_metadata` as part of the current workflow. | CN: 在当前流程中调用 `_write_metadata`。
- **L580** EN: Keeps the inline comment or directive: Step 3: Write actual tensor data from input files to output files | CN: 保留这一行注释或指令：Step 3: Write actual tensor data from input files to output files

### Lines 581-600 / 第 581-600 行

````python
    _write_data(input_files_data, output_files_data, num_threads)

    return output_files_data


def consolidate_safetensors_files(
    input_dir: str,
    output_dir: str,
    fqn_to_index_mapping: dict[str, int],
    num_threads: int = 1,
) -> None:
    """
    Main function to consolidate sharded safetensors files into one or more output files.

    This function orchestrates the entire consolidation process:
    1. Sets up the output file structure based on the fqn_to_index_mapping
    2. Finds all safetensors files in the input directory
    3. Parses metadata from all input files
    4. Writes metadata to the output files
    5. Writes tensor data from input files to output files
````

- **L581** EN: Calls `_write_data` as part of the current workflow. | CN: 在当前流程中调用 `_write_data`。
- **L582** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L583** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L584** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L585** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L586** EN: Defines function `consolidate_safetensors_files`. | CN: 定义函数 `consolidate_safetensors_files`。
- **L587** EN: Continues the implementation inside function `consolidate_safetensors_files`. | CN: 继续说明函数 `consolidate_safetensors_files` 内部的实现。
- **L588** EN: Continues the implementation inside function `consolidate_safetensors_files`. | CN: 继续说明函数 `consolidate_safetensors_files` 内部的实现。
- **L589** EN: Continues the implementation inside function `consolidate_safetensors_files`. | CN: 继续说明函数 `consolidate_safetensors_files` 内部的实现。
- **L590** EN: Assigns or updates `num_threads`. | CN: 对 `num_threads` 进行赋值或更新。
- **L591** EN: Continues the implementation inside function `consolidate_safetensors_files`. | CN: 继续说明函数 `consolidate_safetensors_files` 内部的实现。
- **L592** EN: Starts the docstring for the function consolidate_safetensors_files. | CN: 开始定义 function consolidate_safetensors_files 的文档字符串。
- **L593** EN: Continues the docstring text for the function consolidate_safetensors_files. | CN: 继续补充 function consolidate_safetensors_files 的文档字符串内容。
- **L594** EN: Continues the docstring text for the function consolidate_safetensors_files. | CN: 继续补充 function consolidate_safetensors_files 的文档字符串内容。
- **L595** EN: Continues the docstring text for the function consolidate_safetensors_files. | CN: 继续补充 function consolidate_safetensors_files 的文档字符串内容。
- **L596** EN: Continues the docstring text for the function consolidate_safetensors_files. | CN: 继续补充 function consolidate_safetensors_files 的文档字符串内容。
- **L597** EN: Continues the docstring text for the function consolidate_safetensors_files. | CN: 继续补充 function consolidate_safetensors_files 的文档字符串内容。
- **L598** EN: Continues the docstring text for the function consolidate_safetensors_files. | CN: 继续补充 function consolidate_safetensors_files 的文档字符串内容。
- **L599** EN: Continues the docstring text for the function consolidate_safetensors_files. | CN: 继续补充 function consolidate_safetensors_files 的文档字符串内容。
- **L600** EN: Continues the docstring text for the function consolidate_safetensors_files. | CN: 继续补充 function consolidate_safetensors_files 的文档字符串内容。

### Lines 601-620 / 第 601-620 行

````python
    6. Writes overall model.index.safetensors.json file with weight map

    Args:
        input_dir: Directory containing sharded safetensors files
        output_dir: Directory where consolidated files will be written
        fqn_to_index_mapping: Optional mapping of tensor names to output file indices.
                             If None, all tensors will be consolidated into a single file.
        num_threads: Number of threads to use for parallel processing of saving data to output files.
    """
    start_time = time.time()
    logger.info(
        "Consolidating safetensors files from %s to %s. Beginning at time %f",
        input_dir,
        output_dir,
        start_time,
    )

    max_index = max(fqn_to_index_mapping.values())
    fqn_to_file_mapping = {
        fqn: _gen_file_name(idx, max_index) for fqn, idx in fqn_to_index_mapping.items()
````

- **L601** EN: Continues the docstring text for the function consolidate_safetensors_files. | CN: 继续补充 function consolidate_safetensors_files 的文档字符串内容。
- **L602** EN: Continues the docstring text for the function consolidate_safetensors_files. | CN: 继续补充 function consolidate_safetensors_files 的文档字符串内容。
- **L603** EN: Continues the docstring text for the function consolidate_safetensors_files. | CN: 继续补充 function consolidate_safetensors_files 的文档字符串内容。
- **L604** EN: Continues the docstring text for the function consolidate_safetensors_files. | CN: 继续补充 function consolidate_safetensors_files 的文档字符串内容。
- **L605** EN: Continues the docstring text for the function consolidate_safetensors_files. | CN: 继续补充 function consolidate_safetensors_files 的文档字符串内容。
- **L606** EN: Continues the docstring text for the function consolidate_safetensors_files. | CN: 继续补充 function consolidate_safetensors_files 的文档字符串内容。
- **L607** EN: Continues the docstring text for the function consolidate_safetensors_files. | CN: 继续补充 function consolidate_safetensors_files 的文档字符串内容。
- **L608** EN: Continues the docstring text for the function consolidate_safetensors_files. | CN: 继续补充 function consolidate_safetensors_files 的文档字符串内容。
- **L609** EN: Closes the docstring for the function consolidate_safetensors_files. | CN: 结束 function consolidate_safetensors_files 的文档字符串。
- **L610** EN: Assigns or updates `start_time`. | CN: 对 `start_time` 进行赋值或更新。
- **L611** EN: Calls `logger.info` as part of the current workflow. | CN: 在当前流程中调用 `logger.info`。
- **L612** EN: Continues the implementation inside function `consolidate_safetensors_files`. | CN: 继续说明函数 `consolidate_safetensors_files` 内部的实现。
- **L613** EN: Continues the implementation inside function `consolidate_safetensors_files`. | CN: 继续说明函数 `consolidate_safetensors_files` 内部的实现。
- **L614** EN: Continues the implementation inside function `consolidate_safetensors_files`. | CN: 继续说明函数 `consolidate_safetensors_files` 内部的实现。
- **L615** EN: Continues the implementation inside function `consolidate_safetensors_files`. | CN: 继续说明函数 `consolidate_safetensors_files` 内部的实现。
- **L616** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L617** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L618** EN: Assigns or updates `max_index`. | CN: 对 `max_index` 进行赋值或更新。
- **L619** EN: Assigns or updates `fqn_to_file_mapping`. | CN: 对 `fqn_to_file_mapping` 进行赋值或更新。
- **L620** EN: Continues the implementation inside function `consolidate_safetensors_files`. | CN: 继续说明函数 `consolidate_safetensors_files` 内部的实现。

### Lines 621-640 / 第 621-640 行

````python
    }

    output_files_data = _consolidate_safetensors_files(
        input_dir, output_dir, fqn_to_file_mapping, num_threads
    )

    # Step 4: Write overall model.index.safetensors.json file with weight map
    _write_overall_metadata_file(output_dir, output_files_data)

    logger.info("Done consolidating. Took %.2f secs.", time.time() - start_time)


def consolidate_safetensors_files_on_every_rank(
    input_dir: str,
    output_dir: str,
    fqn_to_index_mapping: dict[str, int],
    num_threads: int = 1,
    process_group: dist.ProcessGroup | None = None,
) -> None:
    """
````

- **L621** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L622** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L623** EN: Assigns or updates `output_files_data`. | CN: 对 `output_files_data` 进行赋值或更新。
- **L624** EN: Continues the implementation inside function `consolidate_safetensors_files`. | CN: 继续说明函数 `consolidate_safetensors_files` 内部的实现。
- **L625** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L626** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L627** EN: Keeps the inline comment or directive: Step 4: Write overall model.index.safetensors.json file with weight map | CN: 保留这一行注释或指令：Step 4: Write overall model.index.safetensors.json file with weight map
- **L628** EN: Calls `_write_overall_metadata_file` as part of the current workflow. | CN: 在当前流程中调用 `_write_overall_metadata_file`。
- **L629** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L630** EN: Calls `logger.info` as part of the current workflow. | CN: 在当前流程中调用 `logger.info`。
- **L631** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L632** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L633** EN: Defines function `consolidate_safetensors_files_on_every_rank`. | CN: 定义函数 `consolidate_safetensors_files_on_every_rank`。
- **L634** EN: Continues the implementation inside function `consolidate_safetensors_files_on_every_rank`. | CN: 继续说明函数 `consolidate_safetensors_files_on_every_rank` 内部的实现。
- **L635** EN: Continues the implementation inside function `consolidate_safetensors_files_on_every_rank`. | CN: 继续说明函数 `consolidate_safetensors_files_on_every_rank` 内部的实现。
- **L636** EN: Continues the implementation inside function `consolidate_safetensors_files_on_every_rank`. | CN: 继续说明函数 `consolidate_safetensors_files_on_every_rank` 内部的实现。
- **L637** EN: Assigns or updates `num_threads`. | CN: 对 `num_threads` 进行赋值或更新。
- **L638** EN: Assigns or updates `process_group`. | CN: 对 `process_group` 进行赋值或更新。
- **L639** EN: Continues the implementation inside function `consolidate_safetensors_files_on_every_rank`. | CN: 继续说明函数 `consolidate_safetensors_files_on_every_rank` 内部的实现。
- **L640** EN: Starts the docstring for the function consolidate_safetensors_files_on_every_rank. | CN: 开始定义 function consolidate_safetensors_files_on_every_rank 的文档字符串。

### Lines 641-660 / 第 641-660 行

````python
    Consolidate sharded safetensors files across multiple ranks, with each rank handling a subset of output files.

    This function distributes the consolidation work by assigning output files to different ranks.
    All tensors with the same index in fqn_to_index_mapping are processed by the same rank,
    as they belong to the same output file.

    If process_group is provided, rank and world_size will be derived from it. Otherwise,
    they will be automatically detected from the distributed environment if available.

    Args:
        input_dir: Directory containing sharded safetensors files
        output_dir: Directory where consolidated files will be written
        fqn_to_index_mapping: Mapping of tensor names to output file indices
        num_threads: Number of threads to use for parallel processing on each rank
        process_group: PyTorch distributed process group (default: None, will use default group)
    """

    start_time = time.time()
    # Derive rank and world_size from process_group or default distributed environment
    if dist.is_available() and dist.is_initialized():
````

- **L641** EN: Continues the docstring text for the function consolidate_safetensors_files_on_every_rank. | CN: 继续补充 function consolidate_safetensors_files_on_every_rank 的文档字符串内容。
- **L642** EN: Continues the docstring text for the function consolidate_safetensors_files_on_every_rank. | CN: 继续补充 function consolidate_safetensors_files_on_every_rank 的文档字符串内容。
- **L643** EN: Continues the docstring text for the function consolidate_safetensors_files_on_every_rank. | CN: 继续补充 function consolidate_safetensors_files_on_every_rank 的文档字符串内容。
- **L644** EN: Continues the docstring text for the function consolidate_safetensors_files_on_every_rank. | CN: 继续补充 function consolidate_safetensors_files_on_every_rank 的文档字符串内容。
- **L645** EN: Continues the docstring text for the function consolidate_safetensors_files_on_every_rank. | CN: 继续补充 function consolidate_safetensors_files_on_every_rank 的文档字符串内容。
- **L646** EN: Continues the docstring text for the function consolidate_safetensors_files_on_every_rank. | CN: 继续补充 function consolidate_safetensors_files_on_every_rank 的文档字符串内容。
- **L647** EN: Continues the docstring text for the function consolidate_safetensors_files_on_every_rank. | CN: 继续补充 function consolidate_safetensors_files_on_every_rank 的文档字符串内容。
- **L648** EN: Continues the docstring text for the function consolidate_safetensors_files_on_every_rank. | CN: 继续补充 function consolidate_safetensors_files_on_every_rank 的文档字符串内容。
- **L649** EN: Continues the docstring text for the function consolidate_safetensors_files_on_every_rank. | CN: 继续补充 function consolidate_safetensors_files_on_every_rank 的文档字符串内容。
- **L650** EN: Continues the docstring text for the function consolidate_safetensors_files_on_every_rank. | CN: 继续补充 function consolidate_safetensors_files_on_every_rank 的文档字符串内容。
- **L651** EN: Continues the docstring text for the function consolidate_safetensors_files_on_every_rank. | CN: 继续补充 function consolidate_safetensors_files_on_every_rank 的文档字符串内容。
- **L652** EN: Continues the docstring text for the function consolidate_safetensors_files_on_every_rank. | CN: 继续补充 function consolidate_safetensors_files_on_every_rank 的文档字符串内容。
- **L653** EN: Continues the docstring text for the function consolidate_safetensors_files_on_every_rank. | CN: 继续补充 function consolidate_safetensors_files_on_every_rank 的文档字符串内容。
- **L654** EN: Continues the docstring text for the function consolidate_safetensors_files_on_every_rank. | CN: 继续补充 function consolidate_safetensors_files_on_every_rank 的文档字符串内容。
- **L655** EN: Continues the docstring text for the function consolidate_safetensors_files_on_every_rank. | CN: 继续补充 function consolidate_safetensors_files_on_every_rank 的文档字符串内容。
- **L656** EN: Closes the docstring for the function consolidate_safetensors_files_on_every_rank. | CN: 结束 function consolidate_safetensors_files_on_every_rank 的文档字符串。
- **L657** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L658** EN: Assigns or updates `start_time`. | CN: 对 `start_time` 进行赋值或更新。
- **L659** EN: Keeps the inline comment or directive: Derive rank and world_size from process_group or default distributed environment | CN: 保留这一行注释或指令：Derive rank and world_size from process_group or default distributed environment
- **L660** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 661-680 / 第 661-680 行

````python
        rank = dist.get_rank(group=process_group)
        world_size = dist.get_world_size(group=process_group)
    else:
        # Default to single process mode if distributed is not initialized
        rank = 0
        world_size = 1
        logger.warning(
            "Distributed environment not initialized. Running in single process mode."
        )
    logger.info(
        "Rank %d/%d: Consolidating safetensors files from %s to %s",
        rank,
        world_size,
        input_dir,
        output_dir,
    )

    # Find all unique indices in the mapping
    unique_indices = set(fqn_to_index_mapping.values())

````

- **L661** EN: Assigns or updates `rank`. | CN: 对 `rank` 进行赋值或更新。
- **L662** EN: Assigns or updates `world_size`. | CN: 对 `world_size` 进行赋值或更新。
- **L663** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L664** EN: Keeps the inline comment or directive: Default to single process mode if distributed is not initialized | CN: 保留这一行注释或指令：Default to single process mode if distributed is not initialized
- **L665** EN: Assigns or updates `rank`. | CN: 对 `rank` 进行赋值或更新。
- **L666** EN: Assigns or updates `world_size`. | CN: 对 `world_size` 进行赋值或更新。
- **L667** EN: Calls `logger.warning` as part of the current workflow. | CN: 在当前流程中调用 `logger.warning`。
- **L668** EN: Continues the implementation inside function `consolidate_safetensors_files_on_every_rank`. | CN: 继续说明函数 `consolidate_safetensors_files_on_every_rank` 内部的实现。
- **L669** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L670** EN: Calls `logger.info` as part of the current workflow. | CN: 在当前流程中调用 `logger.info`。
- **L671** EN: Continues the implementation inside function `consolidate_safetensors_files_on_every_rank`. | CN: 继续说明函数 `consolidate_safetensors_files_on_every_rank` 内部的实现。
- **L672** EN: Continues the implementation inside function `consolidate_safetensors_files_on_every_rank`. | CN: 继续说明函数 `consolidate_safetensors_files_on_every_rank` 内部的实现。
- **L673** EN: Continues the implementation inside function `consolidate_safetensors_files_on_every_rank`. | CN: 继续说明函数 `consolidate_safetensors_files_on_every_rank` 内部的实现。
- **L674** EN: Continues the implementation inside function `consolidate_safetensors_files_on_every_rank`. | CN: 继续说明函数 `consolidate_safetensors_files_on_every_rank` 内部的实现。
- **L675** EN: Continues the implementation inside function `consolidate_safetensors_files_on_every_rank`. | CN: 继续说明函数 `consolidate_safetensors_files_on_every_rank` 内部的实现。
- **L676** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L677** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L678** EN: Keeps the inline comment or directive: Find all unique indices in the mapping | CN: 保留这一行注释或指令：Find all unique indices in the mapping
- **L679** EN: Assigns or updates `unique_indices`. | CN: 对 `unique_indices` 进行赋值或更新。
- **L680** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 681-700 / 第 681-700 行

````python
    # Distribute indices across ranks
    indices_for_this_rank = []
    for idx in unique_indices:
        # Simple distribution: index % world_size == rank
        if idx % world_size == rank:
            indices_for_this_rank.append(idx)

    logger.info(
        "Rank %d: Assigned %d output files out of %d total files",
        rank,
        len(indices_for_this_rank),
        len(unique_indices),
    )

    # Filter the fqn_to_index_mapping to only include tensors for this rank
    filtered_mapping = {
        fqn: idx
        for fqn, idx in fqn_to_index_mapping.items()
        if idx in indices_for_this_rank
    }
````

- **L681** EN: Keeps the inline comment or directive: Distribute indices across ranks | CN: 保留这一行注释或指令：Distribute indices across ranks
- **L682** EN: Assigns or updates `indices_for_this_rank`. | CN: 对 `indices_for_this_rank` 进行赋值或更新。
- **L683** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L684** EN: Keeps the inline comment or directive: Simple distribution: index % world_size == rank | CN: 保留这一行注释或指令：Simple distribution: index % world_size == rank
- **L685** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L686** EN: Calls `indices_for_this_rank.append` as part of the current workflow. | CN: 在当前流程中调用 `indices_for_this_rank.append`。
- **L687** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L688** EN: Calls `logger.info` as part of the current workflow. | CN: 在当前流程中调用 `logger.info`。
- **L689** EN: Continues the implementation inside function `consolidate_safetensors_files_on_every_rank`. | CN: 继续说明函数 `consolidate_safetensors_files_on_every_rank` 内部的实现。
- **L690** EN: Continues the implementation inside function `consolidate_safetensors_files_on_every_rank`. | CN: 继续说明函数 `consolidate_safetensors_files_on_every_rank` 内部的实现。
- **L691** EN: Calls `len` as part of the current workflow. | CN: 在当前流程中调用 `len`。
- **L692** EN: Calls `len` as part of the current workflow. | CN: 在当前流程中调用 `len`。
- **L693** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L694** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L695** EN: Keeps the inline comment or directive: Filter the fqn_to_index_mapping to only include tensors for this rank | CN: 保留这一行注释或指令：Filter the fqn_to_index_mapping to only include tensors for this rank
- **L696** EN: Assigns or updates `filtered_mapping`. | CN: 对 `filtered_mapping` 进行赋值或更新。
- **L697** EN: Continues the implementation inside function `consolidate_safetensors_files_on_every_rank`. | CN: 继续说明函数 `consolidate_safetensors_files_on_every_rank` 内部的实现。
- **L698** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L699** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L700** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 701-720 / 第 701-720 行

````python

    output_files_data: dict[str, _OutputFileData] = {}
    if filtered_mapping:
        # Convert index mapping to filename mapping
        max_index = max(unique_indices)
        filtered_filename_mapping = {}
        for fqn, idx in filtered_mapping.items():
            filename = _gen_file_name(idx, max_index)
            filtered_filename_mapping[fqn] = filename

        # Call the existing consolidation function with the filtered mapping
        output_files_data = _consolidate_safetensors_files(
            input_dir=input_dir,
            output_dir=output_dir,
            fqn_to_file_mapping=filtered_filename_mapping,
            num_threads=num_threads,
        )

    logger.info(
        "Rank %d: Done consolidating. Processed %d unique indices in %.2f secs.",
````

- **L701** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L702** EN: Assigns or updates `output_files_data`. | CN: 对 `output_files_data` 进行赋值或更新。
- **L703** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L704** EN: Keeps the inline comment or directive: Convert index mapping to filename mapping | CN: 保留这一行注释或指令：Convert index mapping to filename mapping
- **L705** EN: Assigns or updates `max_index`. | CN: 对 `max_index` 进行赋值或更新。
- **L706** EN: Assigns or updates `filtered_filename_mapping`. | CN: 对 `filtered_filename_mapping` 进行赋值或更新。
- **L707** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L708** EN: Assigns or updates `filename`. | CN: 对 `filename` 进行赋值或更新。
- **L709** EN: Assigns or updates `filtered_filename_mapping[fqn]`. | CN: 对 `filtered_filename_mapping[fqn]` 进行赋值或更新。
- **L710** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L711** EN: Keeps the inline comment or directive: Call the existing consolidation function with the filtered mapping | CN: 保留这一行注释或指令：Call the existing consolidation function with the filtered mapping
- **L712** EN: Assigns or updates `output_files_data`. | CN: 对 `output_files_data` 进行赋值或更新。
- **L713** EN: Assigns or updates `input_dir`. | CN: 对 `input_dir` 进行赋值或更新。
- **L714** EN: Assigns or updates `output_dir`. | CN: 对 `output_dir` 进行赋值或更新。
- **L715** EN: Assigns or updates `fqn_to_file_mapping`. | CN: 对 `fqn_to_file_mapping` 进行赋值或更新。
- **L716** EN: Assigns or updates `num_threads`. | CN: 对 `num_threads` 进行赋值或更新。
- **L717** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L718** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L719** EN: Calls `logger.info` as part of the current workflow. | CN: 在当前流程中调用 `logger.info`。
- **L720** EN: Continues the implementation inside function `consolidate_safetensors_files_on_every_rank`. | CN: 继续说明函数 `consolidate_safetensors_files_on_every_rank` 内部的实现。

### Lines 721-740 / 第 721-740 行

````python
        rank,
        len(indices_for_this_rank),
        time.time() - start_time,
    )

    # Wait for all ranks to complete and gather output_files_data on rank 0
    if dist.is_available() and dist.is_initialized():
        gathered_output_files_data: list[dict[str, _OutputFileData]] | None = (
            [{} for _ in range(world_size)] if rank == 0 else None
        )
        dist.gather_object(
            output_files_data,
            gathered_output_files_data,
            dst=0,
            group=process_group,
        )

        if rank == 0:
            # Merge all output_files_data from all ranks
            all_output_files_data: dict[str, _OutputFileData] = {}
````

- **L721** EN: Continues the implementation inside function `consolidate_safetensors_files_on_every_rank`. | CN: 继续说明函数 `consolidate_safetensors_files_on_every_rank` 内部的实现。
- **L722** EN: Calls `len` as part of the current workflow. | CN: 在当前流程中调用 `len`。
- **L723** EN: Calls `time.time` as part of the current workflow. | CN: 在当前流程中调用 `time.time`。
- **L724** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L725** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L726** EN: Keeps the inline comment or directive: Wait for all ranks to complete and gather output_files_data on rank 0 | CN: 保留这一行注释或指令：Wait for all ranks to complete and gather output_files_data on rank 0
- **L727** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L728** EN: Assigns or updates `gathered_output_files_data`. | CN: 对 `gathered_output_files_data` 进行赋值或更新。
- **L729** EN: Continues the implementation inside function `consolidate_safetensors_files_on_every_rank`. | CN: 继续说明函数 `consolidate_safetensors_files_on_every_rank` 内部的实现。
- **L730** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L731** EN: Calls `dist.gather_object` as part of the current workflow. | CN: 在当前流程中调用 `dist.gather_object`。
- **L732** EN: Continues the implementation inside function `consolidate_safetensors_files_on_every_rank`. | CN: 继续说明函数 `consolidate_safetensors_files_on_every_rank` 内部的实现。
- **L733** EN: Continues the implementation inside function `consolidate_safetensors_files_on_every_rank`. | CN: 继续说明函数 `consolidate_safetensors_files_on_every_rank` 内部的实现。
- **L734** EN: Assigns or updates `dst`. | CN: 对 `dst` 进行赋值或更新。
- **L735** EN: Assigns or updates `group`. | CN: 对 `group` 进行赋值或更新。
- **L736** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L737** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L738** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L739** EN: Keeps the inline comment or directive: Merge all output_files_data from all ranks | CN: 保留这一行注释或指令：Merge all output_files_data from all ranks
- **L740** EN: Assigns or updates `all_output_files_data`. | CN: 对 `all_output_files_data` 进行赋值或更新。

### Lines 741-749 / 第 741-749 行

````python
            if gathered_output_files_data is None:
                raise AssertionError
            for rank_data in gathered_output_files_data:
                all_output_files_data.update(rank_data)

            _write_overall_metadata_file(output_dir, all_output_files_data)
            logger.info("Rank 0: Wrote overall metadata file.")
            logger.info("Total time taken: %.2f secs.", time.time() - start_time)
        dist.barrier(group=process_group)
````

- **L741** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L742** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L743** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L744** EN: Calls `all_output_files_data.update` as part of the current workflow. | CN: 在当前流程中调用 `all_output_files_data.update`。
- **L745** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L746** EN: Calls `_write_overall_metadata_file` as part of the current workflow. | CN: 在当前流程中调用 `_write_overall_metadata_file`。
- **L747** EN: Calls `logger.info` as part of the current workflow. | CN: 在当前流程中调用 `logger.info`。
- **L748** EN: Calls `logger.info` as part of the current workflow. | CN: 在当前流程中调用 `logger.info`。
- **L749** EN: Calls `dist.barrier` as part of the current workflow. | CN: 在当前流程中调用 `dist.barrier`。

## Key Concepts / 关键概念

- **EN**: distributed checkpointing, planners, and storage helpers  
  **CN**: 分布式检查点、规划器与存储辅助逻辑
- **EN**: process groups  
  **CN**: 进程组
- **EN**: sharding  
  **CN**: 分片
- **EN**: checkpointing  
  **CN**: 检查点
- **EN**: distributed stores  
  **CN**: 分布式存储
- **EN**: Primary classes: _FqnData, _OutputFileData, _InputFileData  
  **CN**: 主要类：_FqnData, _OutputFileData, _InputFileData
- **EN**: Core callables: _parse_input_metadata, _write_metadata, _read_tensor_data, _process_output_file, _write_data  
  **CN**: 核心可调用对象：_parse_input_metadata, _write_metadata, _read_tensor_data, _process_output_file, _write_data

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed.checkpoint._hf_utils`
- **PyTorch / PyTorch**: `torch`
- **Python Stdlib / Python 标准库**: `concurrent.futures`, `dataclasses`, `glob`, `json`, `logging`, `math`, `os`, `struct`, `time`, `typing`
- **Third-party / 第三方**: `safetensors.torch`

