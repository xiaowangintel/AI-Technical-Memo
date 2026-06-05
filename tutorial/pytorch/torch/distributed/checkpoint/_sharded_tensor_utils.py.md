# _sharded_tensor_utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/checkpoint/_sharded_tensor_utils.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed checkpointing, planners, and storage helpers. Its main entry points include _flatten_sharded_tensors.
- **用途 (CN)**: 该模块聚焦于分布式检查点、规划器与存储辅助逻辑，其主要入口包括 _flatten_sharded_tensors。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# Copyright (c) Meta Platforms, Inc. and affiliates

import copy
from typing import TYPE_CHECKING

import torch.distributed as dist
from torch.distributed._shard.sharded_tensor import Shard, ShardedTensor, ShardMetadata
from torch.distributed.checkpoint.metadata import STATE_DICT_TYPE
from torch.distributed.remote_device import _remote_device

from ._traverse import OBJ_PATH, set_element, STATE_DICT_ITEM, traverse_state_dict
from .utils import _element_wise_add, _normalize_device_info


if TYPE_CHECKING:
    from torch.distributed._shard.sharded_tensor.metadata import ShardedTensorMetadata


# TODO: We need to refactor this code.
def _flatten_sharded_tensors(state_dict: STATE_DICT_TYPE) -> STATE_DICT_TYPE:
````

- **L1** EN: Keeps the inline comment or directive: Copyright (c) Meta Platforms, Inc. and affiliates | CN: 保留这一行注释或指令：Copyright (c) Meta Platforms, Inc. and affiliates
- **L2** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L3** EN: Imports module dependencies: `copy`. | CN: 导入模块依赖：`copy`。
- **L4** EN: Imports selected names from `typing`. | CN: 从 `typing` 导入指定名称。
- **L5** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L6** EN: Imports module dependencies: `torch.distributed as dist`. | CN: 导入模块依赖：`torch.distributed as dist`。
- **L7** EN: Imports selected names from `torch.distributed._shard.sharded_tensor`. | CN: 从 `torch.distributed._shard.sharded_tensor` 导入指定名称。
- **L8** EN: Imports selected names from `torch.distributed.checkpoint.metadata`. | CN: 从 `torch.distributed.checkpoint.metadata` 导入指定名称。
- **L9** EN: Imports selected names from `torch.distributed.remote_device`. | CN: 从 `torch.distributed.remote_device` 导入指定名称。
- **L10** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L11** EN: Imports selected names from `._traverse`. | CN: 从 `._traverse` 导入指定名称。
- **L12** EN: Imports selected names from `.utils`. | CN: 从 `.utils` 导入指定名称。
- **L13** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L14** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L15** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L16** EN: Imports selected names from `torch.distributed._shard.sharded_tensor.metadata`. | CN: 从 `torch.distributed._shard.sharded_tensor.metadata` 导入指定名称。
- **L17** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L18** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L19** EN: Keeps the inline comment or directive: TODO: We need to refactor this code. | CN: 保留这一行注释或指令：TODO: We need to refactor this code.
- **L20** EN: Defines function `_flatten_sharded_tensors`. | CN: 定义函数 `_flatten_sharded_tensors`。

### Lines 21-40 / 第 21-40 行

````python
    r"""
    Transform ``state_dict`` by flattening all nested ShardedTensor instances found.

    The resulting ShardedTensor instances are only correct regarding the local shard and
    MUST not be used for any other purpose but checkpointing, as no operator will work with them.

    This function should be used in conjunction with a state_dict produced by FSDP's
    StateDictType.SHARDED_STATE_DICT methods.
    """
    new_state_dict: STATE_DICT_TYPE = {}

    def rewrite_dict(path: OBJ_PATH, value: STATE_DICT_ITEM) -> None:
        if not isinstance(value, ShardedTensor):
            set_element(new_state_dict, path, value)
            return
        shards = value.local_shards()

        if len(shards) == 0:
            return
        if len(shards) != 1:
````

- **L21** EN: Starts the docstring for the function _flatten_sharded_tensors. | CN: 开始定义 function _flatten_sharded_tensors 的文档字符串。
- **L22** EN: Continues the docstring text for the function _flatten_sharded_tensors. | CN: 继续补充 function _flatten_sharded_tensors 的文档字符串内容。
- **L23** EN: Continues the docstring text for the function _flatten_sharded_tensors. | CN: 继续补充 function _flatten_sharded_tensors 的文档字符串内容。
- **L24** EN: Continues the docstring text for the function _flatten_sharded_tensors. | CN: 继续补充 function _flatten_sharded_tensors 的文档字符串内容。
- **L25** EN: Continues the docstring text for the function _flatten_sharded_tensors. | CN: 继续补充 function _flatten_sharded_tensors 的文档字符串内容。
- **L26** EN: Continues the docstring text for the function _flatten_sharded_tensors. | CN: 继续补充 function _flatten_sharded_tensors 的文档字符串内容。
- **L27** EN: Continues the docstring text for the function _flatten_sharded_tensors. | CN: 继续补充 function _flatten_sharded_tensors 的文档字符串内容。
- **L28** EN: Continues the docstring text for the function _flatten_sharded_tensors. | CN: 继续补充 function _flatten_sharded_tensors 的文档字符串内容。
- **L29** EN: Closes the docstring for the function _flatten_sharded_tensors. | CN: 结束 function _flatten_sharded_tensors 的文档字符串。
- **L30** EN: Assigns or updates `new_state_dict`. | CN: 对 `new_state_dict` 进行赋值或更新。
- **L31** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L32** EN: Defines function `rewrite_dict`. | CN: 定义函数 `rewrite_dict`。
- **L33** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L34** EN: Calls `set_element` as part of the current workflow. | CN: 在当前流程中调用 `set_element`。
- **L35** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L36** EN: Assigns or updates `shards`. | CN: 对 `shards` 进行赋值或更新。
- **L37** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L38** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L39** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L40** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 41-60 / 第 41-60 行

````python
            set_element(new_state_dict, path, value)
            return

        outer_shard = shards[0]

        inner_st = outer_shard.tensor
        if not isinstance(inner_st, ShardedTensor):
            set_element(new_state_dict, path, value)
            return

        if len(inner_st.local_shards()) != 1:
            raise ValueError("Cannot handle inner tensor with more than 1 shard")
        inner_shard = inner_st.local_shards()[0]

        local_shards = [
            Shard(
                tensor=inner_shard.tensor,
                metadata=ShardMetadata(
                    shard_offsets=_element_wise_add(
                        outer_shard.metadata.shard_offsets,
````

- **L41** EN: Calls `set_element` as part of the current workflow. | CN: 在当前流程中调用 `set_element`。
- **L42** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L43** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L44** EN: Assigns or updates `outer_shard`. | CN: 对 `outer_shard` 进行赋值或更新。
- **L45** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L46** EN: Assigns or updates `inner_st`. | CN: 对 `inner_st` 进行赋值或更新。
- **L47** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L48** EN: Calls `set_element` as part of the current workflow. | CN: 在当前流程中调用 `set_element`。
- **L49** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L50** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L51** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L52** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L53** EN: Assigns or updates `inner_shard`. | CN: 对 `inner_shard` 进行赋值或更新。
- **L54** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L55** EN: Assigns or updates `local_shards`. | CN: 对 `local_shards` 进行赋值或更新。
- **L56** EN: Calls `Shard` as part of the current workflow. | CN: 在当前流程中调用 `Shard`。
- **L57** EN: Assigns or updates `tensor`. | CN: 对 `tensor` 进行赋值或更新。
- **L58** EN: Assigns or updates `metadata`. | CN: 对 `metadata` 进行赋值或更新。
- **L59** EN: Assigns or updates `shard_offsets`. | CN: 对 `shard_offsets` 进行赋值或更新。
- **L60** EN: Continues the implementation inside function `rewrite_dict`. | CN: 继续说明函数 `rewrite_dict` 内部的实现。

### Lines 61-80 / 第 61-80 行

````python
                        inner_shard.metadata.shard_offsets,
                    ),
                    shard_sizes=inner_shard.metadata.shard_sizes,
                    placement=f"rank:{dist.get_rank()}/{inner_shard.tensor.device}",
                ),
            )
        ]

        st_meta: ShardedTensorMetadata = copy.deepcopy(value.metadata())
        other_rank = 0 if dist.get_rank() > 0 else 1
        device_info = _normalize_device_info(inner_shard.tensor.device.type, 0)

        # Remove the outer ST shard the inner ST covers
        for i, shard_md in enumerate(st_meta.shards_metadata):
            if shard_md.shard_offsets == outer_shard.metadata.shard_offsets:
                st_meta.shards_metadata.pop(i)
                break

        # Attribute other rank for the other shards
        for shard_md in st_meta.shards_metadata:
````

- **L61** EN: Continues the implementation inside function `rewrite_dict`. | CN: 继续说明函数 `rewrite_dict` 内部的实现。
- **L62** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L63** EN: Assigns or updates `shard_sizes`. | CN: 对 `shard_sizes` 进行赋值或更新。
- **L64** EN: Assigns or updates `placement`. | CN: 对 `placement` 进行赋值或更新。
- **L65** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L66** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L67** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L68** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L69** EN: Assigns or updates `st_meta`. | CN: 对 `st_meta` 进行赋值或更新。
- **L70** EN: Assigns or updates `other_rank`. | CN: 对 `other_rank` 进行赋值或更新。
- **L71** EN: Assigns or updates `device_info`. | CN: 对 `device_info` 进行赋值或更新。
- **L72** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L73** EN: Keeps the inline comment or directive: Remove the outer ST shard the inner ST covers | CN: 保留这一行注释或指令：Remove the outer ST shard the inner ST covers
- **L74** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L75** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L76** EN: Calls `st_meta.shards_metadata.pop` as part of the current workflow. | CN: 在当前流程中调用 `st_meta.shards_metadata.pop`。
- **L77** EN: Exits the current loop immediately. | CN: 立即退出当前循环。
- **L78** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L79** EN: Keeps the inline comment or directive: Attribute other rank for the other shards | CN: 保留这一行注释或指令：Attribute other rank for the other shards
- **L80** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。

### Lines 81-100 / 第 81-100 行

````python
            shard_md.placement = _remote_device(f"rank:{other_rank}/{device_info}")

        # Add other inner shards from the inner tensor
        for inner_md in inner_st.metadata().shards_metadata:
            if inner_md.shard_offsets != inner_shard.metadata.shard_offsets:
                st_meta.shards_metadata.append(
                    ShardMetadata(
                        shard_offsets=_element_wise_add(
                            outer_shard.metadata.shard_offsets,
                            inner_md.shard_offsets,
                        ),
                        shard_sizes=inner_md.shard_sizes,
                        placement=f"rank:{other_rank}/{device_info}",
                    )
                )

        # Finally add this shard
        st_meta.shards_metadata.append(local_shards[0].metadata)

        st = ShardedTensor._init_from_local_shards_and_global_metadata(
````

- **L81** EN: Assigns or updates `shard_md.placement`. | CN: 对 `shard_md.placement` 进行赋值或更新。
- **L82** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L83** EN: Keeps the inline comment or directive: Add other inner shards from the inner tensor | CN: 保留这一行注释或指令：Add other inner shards from the inner tensor
- **L84** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L85** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L86** EN: Calls `st_meta.shards_metadata.append` as part of the current workflow. | CN: 在当前流程中调用 `st_meta.shards_metadata.append`。
- **L87** EN: Calls `ShardMetadata` as part of the current workflow. | CN: 在当前流程中调用 `ShardMetadata`。
- **L88** EN: Assigns or updates `shard_offsets`. | CN: 对 `shard_offsets` 进行赋值或更新。
- **L89** EN: Continues the implementation inside function `rewrite_dict`. | CN: 继续说明函数 `rewrite_dict` 内部的实现。
- **L90** EN: Continues the implementation inside function `rewrite_dict`. | CN: 继续说明函数 `rewrite_dict` 内部的实现。
- **L91** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L92** EN: Assigns or updates `shard_sizes`. | CN: 对 `shard_sizes` 进行赋值或更新。
- **L93** EN: Assigns or updates `placement`. | CN: 对 `placement` 进行赋值或更新。
- **L94** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L95** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L96** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L97** EN: Keeps the inline comment or directive: Finally add this shard | CN: 保留这一行注释或指令：Finally add this shard
- **L98** EN: Calls `st_meta.shards_metadata.append` as part of the current workflow. | CN: 在当前流程中调用 `st_meta.shards_metadata.append`。
- **L99** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L100** EN: Assigns or updates `st`. | CN: 对 `st` 进行赋值或更新。

### Lines 101-107 / 第 101-107 行

````python
            local_shards=local_shards,
            sharded_tensor_metadata=st_meta,
        )
        set_element(new_state_dict, path, st)

    traverse_state_dict(state_dict, rewrite_dict)
    return new_state_dict
````

- **L101** EN: Assigns or updates `local_shards`. | CN: 对 `local_shards` 进行赋值或更新。
- **L102** EN: Assigns or updates `sharded_tensor_metadata`. | CN: 对 `sharded_tensor_metadata` 进行赋值或更新。
- **L103** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L104** EN: Calls `set_element` as part of the current workflow. | CN: 在当前流程中调用 `set_element`。
- **L105** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L106** EN: Calls `traverse_state_dict` as part of the current workflow. | CN: 在当前流程中调用 `traverse_state_dict`。
- **L107** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

## Key Concepts / 关键概念

- **EN**: distributed checkpointing, planners, and storage helpers  
  **CN**: 分布式检查点、规划器与存储辅助逻辑
- **EN**: sharding  
  **CN**: 分片
- **EN**: checkpointing  
  **CN**: 检查点
- **EN**: distributed tensors  
  **CN**: 分布式张量
- **EN**: placements  
  **CN**: 放置规则
- **EN**: state dict handling  
  **CN**: state_dict 处理
- **EN**: Core callables: _flatten_sharded_tensors  
  **CN**: 核心可调用对象：_flatten_sharded_tensors

## Dependencies / 依赖关系

- **Internal / 内部**: `._traverse`, `.utils`, `torch.distributed`, `torch.distributed._shard.sharded_tensor`, `torch.distributed._shard.sharded_tensor.metadata`, `torch.distributed.checkpoint.metadata`, `torch.distributed.remote_device`
- **PyTorch / PyTorch**: None detected / 未检测到
- **Python Stdlib / Python 标准库**: `copy`, `typing`
- **Third-party / 第三方**: None detected / 未检测到

