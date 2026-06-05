# _hf_utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/checkpoint/_hf_utils.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed checkpointing, planners, and storage helpers. Its main entry points include _HFStorageInfo, _gen_file_name, _get_safetensors_file_metadata.
- **用途 (CN)**: 该模块聚焦于分布式检查点、规划器与存储辅助逻辑，其主要入口包括 _HFStorageInfo, _gen_file_name, _get_safetensors_file_metadata。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
import io
import json
import struct
from dataclasses import dataclass
from typing import Any

import torch


_metadata_fn: str = "model.safetensors.index.json"

FILE_NAME = "model-{cpt_idx}-of-{num_files}"
SHARDED_FILE_NAME = "shard-{shard_idx}-model-{cpt_idx}-of-{num_files}"
SUFFIX = ".safetensors"

# metadata keys
CUSTOM_METADATA_KEY = "DCP_SHARDING_INFO"
DEFAULT_EXTRA_METADATA_KEY = "__metadata__"
SAVED_OFFSETS_KEY = "saved_offsets"
SHAPE_KEY = "shape"
````

- **L1** EN: Imports module dependencies: `io`. | CN: 导入模块依赖：`io`。
- **L2** EN: Imports module dependencies: `json`. | CN: 导入模块依赖：`json`。
- **L3** EN: Imports module dependencies: `struct`. | CN: 导入模块依赖：`struct`。
- **L4** EN: Imports selected names from `dataclasses`. | CN: 从 `dataclasses` 导入指定名称。
- **L5** EN: Imports selected names from `typing`. | CN: 从 `typing` 导入指定名称。
- **L6** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L7** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L8** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L9** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L10** EN: Assigns or updates `_metadata_fn`. | CN: 对 `_metadata_fn` 进行赋值或更新。
- **L11** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L12** EN: Assigns or updates `FILE_NAME`. | CN: 对 `FILE_NAME` 进行赋值或更新。
- **L13** EN: Assigns or updates `SHARDED_FILE_NAME`. | CN: 对 `SHARDED_FILE_NAME` 进行赋值或更新。
- **L14** EN: Assigns or updates `SUFFIX`. | CN: 对 `SUFFIX` 进行赋值或更新。
- **L15** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L16** EN: Keeps the inline comment or directive: metadata keys | CN: 保留这一行注释或指令：metadata keys
- **L17** EN: Assigns or updates `CUSTOM_METADATA_KEY`. | CN: 对 `CUSTOM_METADATA_KEY` 进行赋值或更新。
- **L18** EN: Assigns or updates `DEFAULT_EXTRA_METADATA_KEY`. | CN: 对 `DEFAULT_EXTRA_METADATA_KEY` 进行赋值或更新。
- **L19** EN: Assigns or updates `SAVED_OFFSETS_KEY`. | CN: 对 `SAVED_OFFSETS_KEY` 进行赋值或更新。
- **L20** EN: Assigns or updates `SHAPE_KEY`. | CN: 对 `SHAPE_KEY` 进行赋值或更新。

### Lines 21-40 / 第 21-40 行

````python
DATA_KEY = "data"
DTYPE_KEY = "dtype"
DATA_OFFSETS_KEY = "data_offsets"

DTYPE_MAP = {
    "F16": torch.float16,
    "F32": torch.float32,
    "F64": torch.float64,
    "I8": torch.int8,
    "U8": torch.uint8,
    "I16": torch.int16,
    "I32": torch.int32,
    "I64": torch.int64,
    "BF16": torch.bfloat16,
}

HF_DCP_VERSION: float = 1.0
DCP_VERSION_KEY = "DCP_VERSION"
DCP_SHARDING_INFO_KEY = "DCP_SHARDING_INFO"

````

- **L21** EN: Assigns or updates `DATA_KEY`. | CN: 对 `DATA_KEY` 进行赋值或更新。
- **L22** EN: Assigns or updates `DTYPE_KEY`. | CN: 对 `DTYPE_KEY` 进行赋值或更新。
- **L23** EN: Assigns or updates `DATA_OFFSETS_KEY`. | CN: 对 `DATA_OFFSETS_KEY` 进行赋值或更新。
- **L24** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L25** EN: Assigns or updates `DTYPE_MAP`. | CN: 对 `DTYPE_MAP` 进行赋值或更新。
- **L26** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L27** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L28** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L29** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L30** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L31** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L32** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L33** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L34** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L35** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L36** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L37** EN: Assigns or updates `HF_DCP_VERSION`. | CN: 对 `HF_DCP_VERSION` 进行赋值或更新。
- **L38** EN: Assigns or updates `DCP_VERSION_KEY`. | CN: 对 `DCP_VERSION_KEY` 进行赋值或更新。
- **L39** EN: Assigns or updates `DCP_SHARDING_INFO_KEY`. | CN: 对 `DCP_SHARDING_INFO_KEY` 进行赋值或更新。
- **L40** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 41-60 / 第 41-60 行

````python
FORMAT_KEY = "format"
FORMAT_VALUE = "pt"

NUM_BYTES_FOR_HEADER_LEN = 8

SHARDED_DIR_NAME = "sharded"


@dataclass
class _HFStorageInfo:
    """This is the per entry storage info."""

    relative_path: str
    shape: torch.Size
    dtype: torch.dtype


def _gen_file_name(
    index: int, largest_index: int, shard_index: int | None = None
) -> str:
````

- **L41** EN: Assigns or updates `FORMAT_KEY`. | CN: 对 `FORMAT_KEY` 进行赋值或更新。
- **L42** EN: Assigns or updates `FORMAT_VALUE`. | CN: 对 `FORMAT_VALUE` 进行赋值或更新。
- **L43** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L44** EN: Assigns or updates `NUM_BYTES_FOR_HEADER_LEN`. | CN: 对 `NUM_BYTES_FOR_HEADER_LEN` 进行赋值或更新。
- **L45** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L46** EN: Assigns or updates `SHARDED_DIR_NAME`. | CN: 对 `SHARDED_DIR_NAME` 进行赋值或更新。
- **L47** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L48** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L49** EN: Applies decorator `dataclass` to the following definition. | CN: 将装饰器 `dataclass` 应用于后续定义。
- **L50** EN: Defines class `_HFStorageInfo`. | CN: 定义类 `_HFStorageInfo`。
- **L51** EN: Docstring line documenting the class _HFStorageInfo. | CN: 这是记录 class _HFStorageInfo 的文档字符串。
- **L52** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L53** EN: Continues the implementation inside class `_HFStorageInfo`. | CN: 继续说明类 `_HFStorageInfo` 内部的实现。
- **L54** EN: Continues the implementation inside class `_HFStorageInfo`. | CN: 继续说明类 `_HFStorageInfo` 内部的实现。
- **L55** EN: Continues the implementation inside class `_HFStorageInfo`. | CN: 继续说明类 `_HFStorageInfo` 内部的实现。
- **L56** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L57** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L58** EN: Defines function `_gen_file_name`. | CN: 定义函数 `_gen_file_name`。
- **L59** EN: Assigns or updates `index`. | CN: 对 `index` 进行赋值或更新。
- **L60** EN: Continues the implementation inside function `_gen_file_name`. | CN: 继续说明函数 `_gen_file_name` 内部的实现。

### Lines 61-80 / 第 61-80 行

````python
    if shard_index is not None:
        return (
            SHARDED_FILE_NAME.format(
                shard_idx=f"{shard_index}".zfill(5),
                cpt_idx=f"{index}".zfill(5),
                num_files=f"{largest_index}".zfill(5),
            )
            + SUFFIX
        )
    else:
        return (
            FILE_NAME.format(
                cpt_idx=f"{index}".zfill(5), num_files=f"{largest_index}".zfill(5)
            )
            + SUFFIX
        )


def _get_safetensors_file_metadata(file_bytes: io.IOBase) -> tuple[Any, int]:
    # this uses the same logic that's done in HF code base
````

- **L61** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L62** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L63** EN: Calls `SHARDED_FILE_NAME.format` as part of the current workflow. | CN: 在当前流程中调用 `SHARDED_FILE_NAME.format`。
- **L64** EN: Assigns or updates `shard_idx`. | CN: 对 `shard_idx` 进行赋值或更新。
- **L65** EN: Assigns or updates `cpt_idx`. | CN: 对 `cpt_idx` 进行赋值或更新。
- **L66** EN: Assigns or updates `num_files`. | CN: 对 `num_files` 进行赋值或更新。
- **L67** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L68** EN: Continues the implementation inside function `_gen_file_name`. | CN: 继续说明函数 `_gen_file_name` 内部的实现。
- **L69** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L70** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L71** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L72** EN: Calls `FILE_NAME.format` as part of the current workflow. | CN: 在当前流程中调用 `FILE_NAME.format`。
- **L73** EN: Assigns or updates `cpt_idx`. | CN: 对 `cpt_idx` 进行赋值或更新。
- **L74** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L75** EN: Continues the implementation inside function `_gen_file_name`. | CN: 继续说明函数 `_gen_file_name` 内部的实现。
- **L76** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L77** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L78** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L79** EN: Defines function `_get_safetensors_file_metadata`. | CN: 定义函数 `_get_safetensors_file_metadata`。
- **L80** EN: Keeps the inline comment or directive: this uses the same logic that's done in HF code base | CN: 保留这一行注释或指令：this uses the same logic that's done in HF code base

### Lines 81-100 / 第 81-100 行

````python
    # https://github.com/2404589803/huggingface_hub/blob/main/src/huggingface_hub/hf_api.py#L5308
    # and follows their documentation on how their files are serialized
    # https://huggingface.co/docs/safetensors/index#format

    header_len_bytes = file_bytes.read(NUM_BYTES_FOR_HEADER_LEN)
    header_len = struct.unpack("<Q", header_len_bytes)[0]
    header_json = file_bytes.read(header_len)
    metadata = json.loads(header_json)
    return (metadata, header_len + NUM_BYTES_FOR_HEADER_LEN)


def _get_dtype(dtype_str: str) -> torch.dtype:
    try:
        dtype = DTYPE_MAP[dtype_str]
    except KeyError:
        dtype = torch.get_default_dtype()

    return dtype


````

- **L81** EN: Keeps the inline comment or directive: https://github.com/2404589803/huggingface_hub/blob/main/src/huggingface_hub/hf_a | CN: 保留这一行注释或指令：https://github.com/2404589803/huggingface_hub/blob/main/src/huggingface_hub/hf_a
- **L82** EN: Keeps the inline comment or directive: and follows their documentation on how their files are serialized | CN: 保留这一行注释或指令：and follows their documentation on how their files are serialized
- **L83** EN: Keeps the inline comment or directive: https://huggingface.co/docs/safetensors/index#format | CN: 保留这一行注释或指令：https://huggingface.co/docs/safetensors/index#format
- **L84** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L85** EN: Assigns or updates `header_len_bytes`. | CN: 对 `header_len_bytes` 进行赋值或更新。
- **L86** EN: Assigns or updates `header_len`. | CN: 对 `header_len` 进行赋值或更新。
- **L87** EN: Assigns or updates `header_json`. | CN: 对 `header_json` 进行赋值或更新。
- **L88** EN: Assigns or updates `metadata`. | CN: 对 `metadata` 进行赋值或更新。
- **L89** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L90** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L91** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L92** EN: Defines function `_get_dtype`. | CN: 定义函数 `_get_dtype`。
- **L93** EN: Starts a protected block for exception handling. | CN: 开始一个受保护的异常处理代码块。
- **L94** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L95** EN: Handles a matching exception path. | CN: 处理匹配到的异常路径。
- **L96** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L97** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L98** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L99** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L100** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 101-106 / 第 101-106 行

````python
def _get_dcp_custom_metadata(metadata: Any) -> Any | None:
    if DEFAULT_EXTRA_METADATA_KEY in metadata:
        custom_metadata = metadata[DEFAULT_EXTRA_METADATA_KEY]
        if CUSTOM_METADATA_KEY in custom_metadata:
            return json.loads(custom_metadata[CUSTOM_METADATA_KEY])
    return None
````

- **L101** EN: Defines function `_get_dcp_custom_metadata`. | CN: 定义函数 `_get_dcp_custom_metadata`。
- **L102** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L103** EN: Assigns or updates `custom_metadata`. | CN: 对 `custom_metadata` 进行赋值或更新。
- **L104** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L105** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L106** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

## Key Concepts / 关键概念

- **EN**: distributed checkpointing, planners, and storage helpers  
  **CN**: 分布式检查点、规划器与存储辅助逻辑
- **EN**: sharding  
  **CN**: 分片
- **EN**: Primary classes: _HFStorageInfo  
  **CN**: 主要类：_HFStorageInfo
- **EN**: Core callables: _gen_file_name, _get_safetensors_file_metadata, _get_dtype, _get_dcp_custom_metadata  
  **CN**: 核心可调用对象：_gen_file_name, _get_safetensors_file_metadata, _get_dtype, _get_dcp_custom_metadata

## Dependencies / 依赖关系

- **Internal / 内部**: None detected / 未检测到
- **PyTorch / PyTorch**: `torch`
- **Python Stdlib / Python 标准库**: `dataclasses`, `io`, `json`, `struct`, `typing`
- **Third-party / 第三方**: None detected / 未检测到

