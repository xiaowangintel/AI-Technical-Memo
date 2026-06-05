# optimizer.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/checkpoint/optimizer.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed checkpointing, planners, and storage helpers. Its main entry points include _ReaderWithOffset, _gen_rank_device, _create_colwise_spec.
- **用途 (CN)**: 该模块聚焦于分布式检查点、规划器与存储辅助逻辑，其主要入口包括 _ReaderWithOffset, _gen_rank_device, _create_colwise_spec。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# Copyright (c) Meta Platforms, Inc. and affiliates

import dataclasses
from collections.abc import Sequence
from typing import cast

import torch
import torch.distributed as dist
from torch._utils import _get_device_module
from torch.distributed._shard.sharded_tensor.api import ShardedTensor
from torch.distributed._shard.sharded_tensor.metadata import (
    TensorProperties as ShardTensorProperties,
)
from torch.distributed._shard.sharded_tensor.shard import Shard
from torch.distributed._shard.sharding_spec.chunk_sharding_spec import ChunkShardingSpec
from torch.distributed.checkpoint._nested_dict import unflatten_state_dict
from torch.distributed.checkpoint.default_planner import DefaultLoadPlanner
from torch.distributed.checkpoint.metadata import (
    BytesStorageMetadata,
    ChunkStorageMetadata,
````

- **L1** EN: Keeps the inline comment or directive: Copyright (c) Meta Platforms, Inc. and affiliates | CN: 保留这一行注释或指令：Copyright (c) Meta Platforms, Inc. and affiliates
- **L2** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L3** EN: Imports module dependencies: `dataclasses`. | CN: 导入模块依赖：`dataclasses`。
- **L4** EN: Imports selected names from `collections.abc`. | CN: 从 `collections.abc` 导入指定名称。
- **L5** EN: Imports selected names from `typing`. | CN: 从 `typing` 导入指定名称。
- **L6** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L7** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L8** EN: Imports module dependencies: `torch.distributed as dist`. | CN: 导入模块依赖：`torch.distributed as dist`。
- **L9** EN: Imports selected names from `torch._utils`. | CN: 从 `torch._utils` 导入指定名称。
- **L10** EN: Imports selected names from `torch.distributed._shard.sharded_tensor.api`. | CN: 从 `torch.distributed._shard.sharded_tensor.api` 导入指定名称。
- **L11** EN: Imports selected names from `torch.distributed._shard.sharded_tensor.metadata`. | CN: 从 `torch.distributed._shard.sharded_tensor.metadata` 导入指定名称。
- **L12** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L13** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L14** EN: Imports selected names from `torch.distributed._shard.sharded_tensor.shard`. | CN: 从 `torch.distributed._shard.sharded_tensor.shard` 导入指定名称。
- **L15** EN: Imports selected names from `torch.distributed._shard.sharding_spec.chunk_sharding_spec`. | CN: 从 `torch.distributed._shard.sharding_spec.chunk_sharding_spec` 导入指定名称。
- **L16** EN: Imports selected names from `torch.distributed.checkpoint._nested_dict`. | CN: 从 `torch.distributed.checkpoint._nested_dict` 导入指定名称。
- **L17** EN: Imports selected names from `torch.distributed.checkpoint.default_planner`. | CN: 从 `torch.distributed.checkpoint.default_planner` 导入指定名称。
- **L18** EN: Imports selected names from `torch.distributed.checkpoint.metadata`. | CN: 从 `torch.distributed.checkpoint.metadata` 导入指定名称。
- **L19** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L20** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。

### Lines 21-40 / 第 21-40 行

````python
    Metadata,
    MetadataIndex,
    STATE_DICT_TYPE,
    TensorProperties,
    TensorStorageMetadata,
)
from torch.distributed.checkpoint.planner import LoadPlan, LoadPlanner
from torch.distributed.checkpoint.planner_helpers import (
    _create_read_items,
    create_read_items_for_chunk_list,
)

# pyrefly: ignore [deprecated]
from torch.distributed.checkpoint.state_dict_loader import load_state_dict
from torch.distributed.checkpoint.storage import StorageReader
from torch.distributed.checkpoint.utils import (
    _element_wise_add,
    _element_wise_sub,
    _normalize_device_info,
)
````

- **L21** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L22** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L23** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L24** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L25** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L26** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L27** EN: Imports selected names from `torch.distributed.checkpoint.planner`. | CN: 从 `torch.distributed.checkpoint.planner` 导入指定名称。
- **L28** EN: Imports selected names from `torch.distributed.checkpoint.planner_helpers`. | CN: 从 `torch.distributed.checkpoint.planner_helpers` 导入指定名称。
- **L29** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L30** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L31** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L32** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L33** EN: Keeps the inline comment or directive: pyrefly: ignore [deprecated] | CN: 保留这一行注释或指令：pyrefly: ignore [deprecated]
- **L34** EN: Imports selected names from `torch.distributed.checkpoint.state_dict_loader`. | CN: 从 `torch.distributed.checkpoint.state_dict_loader` 导入指定名称。
- **L35** EN: Imports selected names from `torch.distributed.checkpoint.storage`. | CN: 从 `torch.distributed.checkpoint.storage` 导入指定名称。
- **L36** EN: Imports selected names from `torch.distributed.checkpoint.utils`. | CN: 从 `torch.distributed.checkpoint.utils` 导入指定名称。
- **L37** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L38** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L39** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L40** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 41-60 / 第 41-60 行

````python
from torch.distributed.distributed_c10d import _get_default_group
from torch.distributed.fsdp._shard_utils import _create_chunk_sharded_tensor
from torch.distributed.remote_device import _remote_device
from torch.distributed.tensor import DTensor


STATE_DICT_2D_LAYOUT = dict[str, tuple[Sequence[int] | None, Sequence[int]]]


# TODO: Update docstrings for optimizer.py
__all__ = [
    "load_sharded_optimizer_state_dict",
]


def _gen_rank_device(global_rank: int, device_type: str = "cuda") -> str:
    if device_type == "cpu":
        return "cpu"
    device_module = _get_device_module(device_type)
    if device_module.is_available():
````

- **L41** EN: Imports selected names from `torch.distributed.distributed_c10d`. | CN: 从 `torch.distributed.distributed_c10d` 导入指定名称。
- **L42** EN: Imports selected names from `torch.distributed.fsdp._shard_utils`. | CN: 从 `torch.distributed.fsdp._shard_utils` 导入指定名称。
- **L43** EN: Imports selected names from `torch.distributed.remote_device`. | CN: 从 `torch.distributed.remote_device` 导入指定名称。
- **L44** EN: Imports selected names from `torch.distributed.tensor`. | CN: 从 `torch.distributed.tensor` 导入指定名称。
- **L45** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L46** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L47** EN: Assigns or updates `STATE_DICT_2D_LAYOUT`. | CN: 对 `STATE_DICT_2D_LAYOUT` 进行赋值或更新。
- **L48** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L49** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L50** EN: Keeps the inline comment or directive: TODO: Update docstrings for optimizer.py | CN: 保留这一行注释或指令：TODO: Update docstrings for optimizer.py
- **L51** EN: Assigns or updates `__all__`. | CN: 对 `__all__` 进行赋值或更新。
- **L52** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L53** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L54** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L55** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L56** EN: Defines function `_gen_rank_device`. | CN: 定义函数 `_gen_rank_device`。
- **L57** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L58** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L59** EN: Assigns or updates `device_module`. | CN: 对 `device_module` 进行赋值或更新。
- **L60** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 61-80 / 第 61-80 行

````python
        return _normalize_device_info(
            device_type, global_rank % device_module.device_count()
        )
    return "cpu"


def _create_colwise_spec(
    pg: dist.ProcessGroup | None = None,
) -> ChunkShardingSpec:
    pg_device_type = dist.distributed_c10d._get_pg_default_device(pg).type
    if pg is None:
        placements = [
            f"rank:{idx}/{_gen_rank_device(idx, pg_device_type)}"
            for idx in range(dist.get_world_size())
        ]
    else:
        placements = [
            f"rank:{idx}/{_gen_rank_device(dist.get_global_rank(pg, idx), pg_device_type)}"
            for idx in range(pg.size())
        ]
````

- **L61** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L62** EN: Continues the implementation inside function `_gen_rank_device`. | CN: 继续说明函数 `_gen_rank_device` 内部的实现。
- **L63** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L64** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L65** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L66** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L67** EN: Defines function `_create_colwise_spec`. | CN: 定义函数 `_create_colwise_spec`。
- **L68** EN: Assigns or updates `pg`. | CN: 对 `pg` 进行赋值或更新。
- **L69** EN: Continues the implementation inside function `_create_colwise_spec`. | CN: 继续说明函数 `_create_colwise_spec` 内部的实现。
- **L70** EN: Assigns or updates `pg_device_type`. | CN: 对 `pg_device_type` 进行赋值或更新。
- **L71** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L72** EN: Assigns or updates `placements`. | CN: 对 `placements` 进行赋值或更新。
- **L73** EN: Continues the implementation inside function `_create_colwise_spec`. | CN: 继续说明函数 `_create_colwise_spec` 内部的实现。
- **L74** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L75** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L76** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L77** EN: Assigns or updates `placements`. | CN: 对 `placements` 进行赋值或更新。
- **L78** EN: Continues the implementation inside function `_create_colwise_spec`. | CN: 继续说明函数 `_create_colwise_spec` 内部的实现。
- **L79** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L80** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 81-100 / 第 81-100 行

````python
    return ChunkShardingSpec(
        dim=0,
        placements=cast(list[_remote_device | str], placements),
    )


def _is_nested_tensor(val: torch.Tensor) -> bool:
    if type(val) is ShardedTensor:
        if len(val.local_shards()) == 0:
            return False
        if type(val.local_shards()[0].tensor) is ShardedTensor:
            return True
        if type(val.local_shards()[0].tensor) is DTensor:
            raise ValueError("Cannot handle DTensor nested inside ShardedTensor")
    elif type(val) is DTensor and (
        type(val._local_tensor) is DTensor or type(val._local_tensor) is ShardedTensor
    ):
        raise ValueError("Cannot handle nested DTensor")
    return False

````

- **L81** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L82** EN: Assigns or updates `dim`. | CN: 对 `dim` 进行赋值或更新。
- **L83** EN: Assigns or updates `placements`. | CN: 对 `placements` 进行赋值或更新。
- **L84** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L85** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L86** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L87** EN: Defines function `_is_nested_tensor`. | CN: 定义函数 `_is_nested_tensor`。
- **L88** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L89** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L90** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L91** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L92** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L93** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L94** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L95** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L96** EN: Calls `type` as part of the current workflow. | CN: 在当前流程中调用 `type`。
- **L97** EN: Continues the implementation inside function `_is_nested_tensor`. | CN: 继续说明函数 `_is_nested_tensor` 内部的实现。
- **L98** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L99** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L100** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 101-120 / 第 101-120 行

````python

def _alloc_tensor(
    props: TensorProperties, size: Sequence[int], device_type: str = "cuda"
) -> torch.Tensor:
    if device_type == "cpu":
        device = cast(torch.device, _get_device_module(device_type).current_device())
    else:
        device = torch.device(
            device_type, _get_device_module(device_type).current_device()
        )

    return torch.empty(
        size=size,
        dtype=props.dtype,
        layout=props.layout,
        requires_grad=props.requires_grad,
        pin_memory=props.pin_memory,
        device=device,
    )

````

- **L101** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L102** EN: Defines function `_alloc_tensor`. | CN: 定义函数 `_alloc_tensor`。
- **L103** EN: Assigns or updates `props`. | CN: 对 `props` 进行赋值或更新。
- **L104** EN: Continues the implementation inside function `_alloc_tensor`. | CN: 继续说明函数 `_alloc_tensor` 内部的实现。
- **L105** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L106** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L107** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L108** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L109** EN: Continues the implementation inside function `_alloc_tensor`. | CN: 继续说明函数 `_alloc_tensor` 内部的实现。
- **L110** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L111** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L112** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L113** EN: Assigns or updates `size`. | CN: 对 `size` 进行赋值或更新。
- **L114** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L115** EN: Assigns or updates `layout`. | CN: 对 `layout` 进行赋值或更新。
- **L116** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L117** EN: Assigns or updates `pin_memory`. | CN: 对 `pin_memory` 进行赋值或更新。
- **L118** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L119** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L120** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 121-140 / 第 121-140 行

````python

def _get_state_dict_2d_layout(
    state_dict: STATE_DICT_TYPE,
) -> tuple[STATE_DICT_2D_LAYOUT, dist.ProcessGroup | None]:
    """
    Load the right TP slice of the optimizer state.

    This is not easy since the per-tensor slicing can't be inferred from checkpoint metadata.
    We take advantage of the model state_dict producing a sliced ST to figure out what we need to load.
    This is pretty fragile and it might be easier for FSDP to compute this info for us.
    Returns a dictionary where keys are the same of the state_dict and the value is a tuple of
    (offset, size) for the current rank TP slice.
    N.B. The state_dict *MUST* come from FSDP.sharded_state_dict.
    """
    specs: STATE_DICT_2D_LAYOUT = {}
    dp_pg: dist.ProcessGroup | None = None
    for key, value in state_dict.items():
        specs[key] = (None, value.size())
        if _is_nested_tensor(value):
            if not len(value.local_shards()) == 1:
````

- **L121** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L122** EN: Defines function `_get_state_dict_2d_layout`. | CN: 定义函数 `_get_state_dict_2d_layout`。
- **L123** EN: Continues the implementation inside function `_get_state_dict_2d_layout`. | CN: 继续说明函数 `_get_state_dict_2d_layout` 内部的实现。
- **L124** EN: Continues the implementation inside function `_get_state_dict_2d_layout`. | CN: 继续说明函数 `_get_state_dict_2d_layout` 内部的实现。
- **L125** EN: Starts the docstring for the function _get_state_dict_2d_layout. | CN: 开始定义 function _get_state_dict_2d_layout 的文档字符串。
- **L126** EN: Continues the docstring text for the function _get_state_dict_2d_layout. | CN: 继续补充 function _get_state_dict_2d_layout 的文档字符串内容。
- **L127** EN: Continues the docstring text for the function _get_state_dict_2d_layout. | CN: 继续补充 function _get_state_dict_2d_layout 的文档字符串内容。
- **L128** EN: Continues the docstring text for the function _get_state_dict_2d_layout. | CN: 继续补充 function _get_state_dict_2d_layout 的文档字符串内容。
- **L129** EN: Continues the docstring text for the function _get_state_dict_2d_layout. | CN: 继续补充 function _get_state_dict_2d_layout 的文档字符串内容。
- **L130** EN: Continues the docstring text for the function _get_state_dict_2d_layout. | CN: 继续补充 function _get_state_dict_2d_layout 的文档字符串内容。
- **L131** EN: Continues the docstring text for the function _get_state_dict_2d_layout. | CN: 继续补充 function _get_state_dict_2d_layout 的文档字符串内容。
- **L132** EN: Continues the docstring text for the function _get_state_dict_2d_layout. | CN: 继续补充 function _get_state_dict_2d_layout 的文档字符串内容。
- **L133** EN: Continues the docstring text for the function _get_state_dict_2d_layout. | CN: 继续补充 function _get_state_dict_2d_layout 的文档字符串内容。
- **L134** EN: Closes the docstring for the function _get_state_dict_2d_layout. | CN: 结束 function _get_state_dict_2d_layout 的文档字符串。
- **L135** EN: Assigns or updates `specs`. | CN: 对 `specs` 进行赋值或更新。
- **L136** EN: Assigns or updates `dp_pg`. | CN: 对 `dp_pg` 进行赋值或更新。
- **L137** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L138** EN: Assigns or updates `specs[key]`. | CN: 对 `specs[key]` 进行赋值或更新。
- **L139** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L140** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 141-160 / 第 141-160 行

````python
                raise AssertionError("Cannot handle ST with multiple shards")
            if not isinstance(value, ShardedTensor):
                raise AssertionError("Can only handle nested ShardedTensor")
            shard = value.local_shards()[0]
            specs[key] = (
                shard.metadata.shard_offsets,
                shard.metadata.shard_sizes,
            )
            dp_pg = shard.tensor._process_group  # type: ignore[attr-defined]

    return (
        specs,
        dp_pg,
    )


class _ReaderWithOffset(DefaultLoadPlanner):
    translation: dict[MetadataIndex, MetadataIndex]
    state_dict: STATE_DICT_TYPE
    # pyrefly: ignore [bad-override]
````

- **L141** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L142** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L143** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L144** EN: Assigns or updates `shard`. | CN: 对 `shard` 进行赋值或更新。
- **L145** EN: Assigns or updates `specs[key]`. | CN: 对 `specs[key]` 进行赋值或更新。
- **L146** EN: Continues the implementation inside function `_get_state_dict_2d_layout`. | CN: 继续说明函数 `_get_state_dict_2d_layout` 内部的实现。
- **L147** EN: Continues the implementation inside function `_get_state_dict_2d_layout`. | CN: 继续说明函数 `_get_state_dict_2d_layout` 内部的实现。
- **L148** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L149** EN: Assigns or updates `dp_pg`. | CN: 对 `dp_pg` 进行赋值或更新。
- **L150** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L151** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L152** EN: Continues the implementation inside function `_get_state_dict_2d_layout`. | CN: 继续说明函数 `_get_state_dict_2d_layout` 内部的实现。
- **L153** EN: Continues the implementation inside function `_get_state_dict_2d_layout`. | CN: 继续说明函数 `_get_state_dict_2d_layout` 内部的实现。
- **L154** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L155** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L156** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L157** EN: Defines class `_ReaderWithOffset`. | CN: 定义类 `_ReaderWithOffset`。
- **L158** EN: Continues the implementation inside class `_ReaderWithOffset`. | CN: 继续说明类 `_ReaderWithOffset` 内部的实现。
- **L159** EN: Continues the implementation inside class `_ReaderWithOffset`. | CN: 继续说明类 `_ReaderWithOffset` 内部的实现。
- **L160** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-override] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-override]

### Lines 161-180 / 第 161-180 行

````python
    metadata: Metadata

    def __init__(self, fqn_to_offset: dict[str, Sequence[int]]) -> None:
        super().__init__()
        self.fqn_to_offset = fqn_to_offset
        self.metadata = Metadata({})
        self.state_dict = {}
        self.translation = {}

    def create_local_plan(self) -> LoadPlan:
        requests = []
        self.translation = {}
        for fqn, obj in self.state_dict.items():
            md = self.metadata.state_dict_metadata[fqn]
            if not isinstance(obj, ShardedTensor):
                requests += _create_read_items(fqn, md, obj)
                continue

            if fqn not in self.fqn_to_offset:
                requests += _create_read_items(fqn, md, obj)
````

- **L161** EN: Continues the implementation inside class `_ReaderWithOffset`. | CN: 继续说明类 `_ReaderWithOffset` 内部的实现。
- **L162** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L163** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L164** EN: Calls `super` as part of the current workflow. | CN: 在当前流程中调用 `super`。
- **L165** EN: Assigns or updates `self.fqn_to_offset`. | CN: 对 `self.fqn_to_offset` 进行赋值或更新。
- **L166** EN: Assigns or updates `self.metadata`. | CN: 对 `self.metadata` 进行赋值或更新。
- **L167** EN: Assigns or updates `self.state_dict`. | CN: 对 `self.state_dict` 进行赋值或更新。
- **L168** EN: Assigns or updates `self.translation`. | CN: 对 `self.translation` 进行赋值或更新。
- **L169** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L170** EN: Defines function `create_local_plan`. | CN: 定义函数 `create_local_plan`。
- **L171** EN: Assigns or updates `requests`. | CN: 对 `requests` 进行赋值或更新。
- **L172** EN: Assigns or updates `self.translation`. | CN: 对 `self.translation` 进行赋值或更新。
- **L173** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L174** EN: Assigns or updates `md`. | CN: 对 `md` 进行赋值或更新。
- **L175** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L176** EN: Continues the implementation inside function `create_local_plan`. | CN: 继续说明函数 `create_local_plan` 内部的实现。
- **L177** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L178** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L179** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L180** EN: Continues the implementation inside function `create_local_plan`. | CN: 继续说明函数 `create_local_plan` 内部的实现。

### Lines 181-200 / 第 181-200 行

````python
                continue

            offset = self.fqn_to_offset[fqn]

            if not len(obj.local_shards()) == 1:
                raise AssertionError("Expected exactly one local shard")
            original_shard = obj.local_shards()[0]
            local_chunks = [
                ChunkStorageMetadata(
                    offsets=torch.Size(
                        _element_wise_add(original_shard.metadata.shard_offsets, offset)
                    ),
                    sizes=torch.Size(original_shard.metadata.shard_sizes),
                )
            ]

            reqs = create_read_items_for_chunk_list(
                fqn, cast(TensorStorageMetadata, md), local_chunks
            )
            # TODO: The ReadItems will have a displaced MetadataIndex, fix it.
````

- **L181** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L182** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L183** EN: Assigns or updates `offset`. | CN: 对 `offset` 进行赋值或更新。
- **L184** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L185** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L186** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L187** EN: Assigns or updates `original_shard`. | CN: 对 `original_shard` 进行赋值或更新。
- **L188** EN: Assigns or updates `local_chunks`. | CN: 对 `local_chunks` 进行赋值或更新。
- **L189** EN: Calls `ChunkStorageMetadata` as part of the current workflow. | CN: 在当前流程中调用 `ChunkStorageMetadata`。
- **L190** EN: Assigns or updates `offsets`. | CN: 对 `offsets` 进行赋值或更新。
- **L191** EN: Calls `_element_wise_add` as part of the current workflow. | CN: 在当前流程中调用 `_element_wise_add`。
- **L192** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L193** EN: Assigns or updates `sizes`. | CN: 对 `sizes` 进行赋值或更新。
- **L194** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L195** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L196** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L197** EN: Assigns or updates `reqs`. | CN: 对 `reqs` 进行赋值或更新。
- **L198** EN: Continues the implementation inside function `create_local_plan`. | CN: 继续说明函数 `create_local_plan` 内部的实现。
- **L199** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L200** EN: Keeps the inline comment or directive: TODO: The ReadItems will have a displaced MetadataIndex, fix it. | CN: 保留这一行注释或指令：TODO: The ReadItems will have a displaced MetadataIndex, fix it.

### Lines 201-220 / 第 201-220 行

````python
            # TODO: we should change _create_sharded_read_items to have more ergonomic API
            for ri in reqs:
                if ri.dest_index.offset is None:
                    raise AssertionError("dest_index.offset must not be None")
                original_offset = _element_wise_sub(ri.dest_index.offset, offset)
                original_index = dataclasses.replace(
                    ri.dest_index, offset=torch.Size(original_offset)
                )
                self.translation[ri.dest_index] = original_index

            requests += reqs
        return LoadPlan(requests)

    def lookup_tensor(self, index: MetadataIndex) -> torch.Tensor:
        return super().lookup_tensor(self.translation.get(index, index))


def load_sharded_optimizer_state_dict(
    model_state_dict: STATE_DICT_TYPE,
    optimizer_key: str,
````

- **L201** EN: Keeps the inline comment or directive: TODO: we should change _create_sharded_read_items to have more ergonomic API | CN: 保留这一行注释或指令：TODO: we should change _create_sharded_read_items to have more ergonomic API
- **L202** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L203** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L204** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L205** EN: Assigns or updates `original_offset`. | CN: 对 `original_offset` 进行赋值或更新。
- **L206** EN: Assigns or updates `original_index`. | CN: 对 `original_index` 进行赋值或更新。
- **L207** EN: Assigns or updates `ri.dest_index, offset`. | CN: 对 `ri.dest_index, offset` 进行赋值或更新。
- **L208** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L209** EN: Assigns or updates `self.translation[ri.dest_index]`. | CN: 对 `self.translation[ri.dest_index]` 进行赋值或更新。
- **L210** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L211** EN: Continues the implementation inside function `create_local_plan`. | CN: 继续说明函数 `create_local_plan` 内部的实现。
- **L212** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L213** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L214** EN: Defines function `lookup_tensor`. | CN: 定义函数 `lookup_tensor`。
- **L215** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L216** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L217** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L218** EN: Defines function `load_sharded_optimizer_state_dict`. | CN: 定义函数 `load_sharded_optimizer_state_dict`。
- **L219** EN: Continues the implementation inside function `load_sharded_optimizer_state_dict`. | CN: 继续说明函数 `load_sharded_optimizer_state_dict` 内部的实现。
- **L220** EN: Continues the implementation inside function `load_sharded_optimizer_state_dict`. | CN: 继续说明函数 `load_sharded_optimizer_state_dict` 内部的实现。

### Lines 221-240 / 第 221-240 行

````python
    storage_reader: StorageReader,
    planner: LoadPlanner | None = None,
) -> STATE_DICT_TYPE:
    """
    Load a state_dict in conjunction with FSDP sharded optimizer state.

    This is the current recommended way to checkpoint FSDP.
    >>> # xdoctest: +SKIP
    >>> import torch.distributed.checkpoint as dist_cp
    >>> # Save
    >>> model: torch.nn.Model
    >>> optim_params = model.parameters()
    >>> optim = torch.optim.SGD(optim_params, lr=0.01)
    >>> # Save
    >>> with FSDP.state_dict_type(model, StateDictType.SHARDED_STATE_DICT):
    >>>     state_dict = {
    >>>         "optimizer": FSDP.optim_state_dict(model, optim),
    >>>         "model": model.state_dict()
    >>>     }
    >>>     dist_cp.save_state_dict(
````

- **L221** EN: Continues the implementation inside function `load_sharded_optimizer_state_dict`. | CN: 继续说明函数 `load_sharded_optimizer_state_dict` 内部的实现。
- **L222** EN: Assigns or updates `planner`. | CN: 对 `planner` 进行赋值或更新。
- **L223** EN: Continues the implementation inside function `load_sharded_optimizer_state_dict`. | CN: 继续说明函数 `load_sharded_optimizer_state_dict` 内部的实现。
- **L224** EN: Starts the docstring for the function load_sharded_optimizer_state_dict. | CN: 开始定义 function load_sharded_optimizer_state_dict 的文档字符串。
- **L225** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L226** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L227** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L228** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L229** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L230** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L231** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L232** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L233** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L234** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L235** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L236** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L237** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L238** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L239** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L240** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。

### Lines 241-260 / 第 241-260 行

````python
    >>>         state_dict=optim_state,
    >>>         storage_writer=dist_cp.FileSystemWriter("checkpoint"),
    >>>         planner=dist_cp.DefaultSavePlanner(),
    >>>     )
    >>>
    >>> # Load
    >>> with FSDP.state_dict_type(model_tp, StateDictType.SHARDED_STATE_DICT):
    >>>     model_state_dict = model_tp.state_dict()
    >>>     checkpoint = {
    >>>         "model": model_state_dict
    >>>     }
    >>>     dist_cp.load_state_dict(
    >>>         state_dict=checkpoint,
    >>>         storage_reader=dist_cp.FileSystemReader(checkpoint_file),
    >>>         planner=dist_cp.DefaultLoadPlanner(),
    >>>     )
    >>>     model.load_state_dict(checkpoint["model_state"])
    >>>
    >>>     optim_state = dist_cp.load_sharded_optimizer_state_dict(
    >>>         model_state_dict,
````

- **L241** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L242** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L243** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L244** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L245** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L246** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L247** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L248** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L249** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L250** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L251** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L252** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L253** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L254** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L255** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L256** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L257** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L258** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L259** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L260** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。

### Lines 261-280 / 第 261-280 行

````python
    >>>         optimizer_key="optimizer",
    >>>         storage_reader=dist_cp.FileSystemReader("checkpoint"),
    >>>     )
    >>>
    >>>     flattened_osd = FSDP.optim_state_dict_to_load(
    >>>        model, optim, optim_state["optimizer"]
    >>>     )
    >>>
    >>>     optim.load_state_dict(flattened_osd)
    """
    metadata = storage_reader.read_metadata()

    layout_specs, dp_pg = _get_state_dict_2d_layout(model_state_dict)
    dp_pg_device_type = dist.distributed_c10d._get_pg_default_device(dp_pg).type
    device_module = _get_device_module(dp_pg_device_type)

    if dp_pg is None:
        placements = []
        for i in range(dist.get_world_size()):
            device_info = _normalize_device_info(
````

- **L261** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L262** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L263** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L264** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L265** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L266** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L267** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L268** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L269** EN: Continues the docstring text for the function load_sharded_optimizer_state_dict. | CN: 继续补充 function load_sharded_optimizer_state_dict 的文档字符串内容。
- **L270** EN: Closes the docstring for the function load_sharded_optimizer_state_dict. | CN: 结束 function load_sharded_optimizer_state_dict 的文档字符串。
- **L271** EN: Assigns or updates `metadata`. | CN: 对 `metadata` 进行赋值或更新。
- **L272** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L273** EN: Assigns or updates `layout_specs, dp_pg`. | CN: 对 `layout_specs, dp_pg` 进行赋值或更新。
- **L274** EN: Assigns or updates `dp_pg_device_type`. | CN: 对 `dp_pg_device_type` 进行赋值或更新。
- **L275** EN: Assigns or updates `device_module`. | CN: 对 `device_module` 进行赋值或更新。
- **L276** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L277** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L278** EN: Assigns or updates `placements`. | CN: 对 `placements` 进行赋值或更新。
- **L279** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L280** EN: Assigns or updates `device_info`. | CN: 对 `device_info` 进行赋值或更新。

### Lines 281-300 / 第 281-300 行

````python
                dp_pg_device_type, i % device_module.device_count()
            )
            placements.append(f"rank:{i}/{device_info}")
        sharding_spec = ChunkShardingSpec(dim=0, placements=placements)  # type: ignore[arg-type]
    else:
        sharding_spec = _create_colwise_spec(dp_pg)

    # Create a state_dict for optimizer state
    state_dict: STATE_DICT_TYPE = {}

    fqn_to_offset: dict[str, Sequence[int]] = {}
    for key, value in metadata.state_dict_metadata.items():
        key_path = metadata.planner_data[key]
        if key_path[0] != optimizer_key:
            continue

        if isinstance(value, BytesStorageMetadata):
            state_dict[key] = "<bytes_io>"
            continue

````

- **L281** EN: Continues the implementation inside function `load_sharded_optimizer_state_dict`. | CN: 继续说明函数 `load_sharded_optimizer_state_dict` 内部的实现。
- **L282** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L283** EN: Calls `placements.append` as part of the current workflow. | CN: 在当前流程中调用 `placements.append`。
- **L284** EN: Assigns or updates `sharding_spec`. | CN: 对 `sharding_spec` 进行赋值或更新。
- **L285** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L286** EN: Assigns or updates `sharding_spec`. | CN: 对 `sharding_spec` 进行赋值或更新。
- **L287** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L288** EN: Keeps the inline comment or directive: Create a state_dict for optimizer state | CN: 保留这一行注释或指令：Create a state_dict for optimizer state
- **L289** EN: Assigns or updates `state_dict`. | CN: 对 `state_dict` 进行赋值或更新。
- **L290** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L291** EN: Assigns or updates `fqn_to_offset`. | CN: 对 `fqn_to_offset` 进行赋值或更新。
- **L292** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L293** EN: Assigns or updates `key_path`. | CN: 对 `key_path` 进行赋值或更新。
- **L294** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L295** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L296** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L297** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L298** EN: Assigns or updates `state_dict[key]`. | CN: 对 `state_dict[key]` 进行赋值或更新。
- **L299** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L300** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 301-320 / 第 301-320 行

````python
        # value: TensorStorageMetadata
        if value.size.numel() == 1:
            state_dict[key] = _alloc_tensor(
                value.properties, value.size, dp_pg_device_type
            )
        elif dp_pg is None:
            state_dict[key] = _create_chunk_sharded_tensor(
                _alloc_tensor(value.properties, value.size, dp_pg_device_type),
                rank=dist.get_rank(),
                world_size=dist.get_world_size(),
                num_devices_per_node=device_module.device_count(),
                pg=_get_default_group(),
            )
        else:
            spec_key = key_path[2]
            alloc_size = layout_specs.get(spec_key, (None, value.size))[1]

            properties = ShardTensorProperties(
                dtype=value.properties.dtype,
                layout=value.properties.layout,
````

- **L301** EN: Keeps the inline comment or directive: value: TensorStorageMetadata | CN: 保留这一行注释或指令：value: TensorStorageMetadata
- **L302** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L303** EN: Assigns or updates `state_dict[key]`. | CN: 对 `state_dict[key]` 进行赋值或更新。
- **L304** EN: Continues the implementation inside function `load_sharded_optimizer_state_dict`. | CN: 继续说明函数 `load_sharded_optimizer_state_dict` 内部的实现。
- **L305** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L306** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L307** EN: Assigns or updates `state_dict[key]`. | CN: 对 `state_dict[key]` 进行赋值或更新。
- **L308** EN: Calls `_alloc_tensor` as part of the current workflow. | CN: 在当前流程中调用 `_alloc_tensor`。
- **L309** EN: Assigns or updates `rank`. | CN: 对 `rank` 进行赋值或更新。
- **L310** EN: Assigns or updates `world_size`. | CN: 对 `world_size` 进行赋值或更新。
- **L311** EN: Assigns or updates `num_devices_per_node`. | CN: 对 `num_devices_per_node` 进行赋值或更新。
- **L312** EN: Assigns or updates `pg`. | CN: 对 `pg` 进行赋值或更新。
- **L313** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L314** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L315** EN: Assigns or updates `spec_key`. | CN: 对 `spec_key` 进行赋值或更新。
- **L316** EN: Assigns or updates `alloc_size`. | CN: 对 `alloc_size` 进行赋值或更新。
- **L317** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L318** EN: Assigns or updates `properties`. | CN: 对 `properties` 进行赋值或更新。
- **L319** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L320** EN: Assigns or updates `layout`. | CN: 对 `layout` 进行赋值或更新。

### Lines 321-340 / 第 321-340 行

````python
                requires_grad=value.properties.requires_grad,
                memory_format=value.properties.memory_format,
                pin_memory=value.properties.pin_memory,
            )

            st_md = sharding_spec.build_metadata(torch.Size(alloc_size), properties)
            local_shards = []
            current_rank = dist.get_rank(dp_pg)
            for shard_md in st_md.shards_metadata:
                if cast(_remote_device, shard_md.placement).rank() != current_rank:
                    continue
                local_shards.append(
                    Shard(
                        tensor=_alloc_tensor(
                            value.properties, shard_md.shard_sizes, dp_pg_device_type
                        ),
                        metadata=shard_md,
                    )
                )

````

- **L321** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L322** EN: Assigns or updates `memory_format`. | CN: 对 `memory_format` 进行赋值或更新。
- **L323** EN: Assigns or updates `pin_memory`. | CN: 对 `pin_memory` 进行赋值或更新。
- **L324** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L325** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L326** EN: Assigns or updates `st_md`. | CN: 对 `st_md` 进行赋值或更新。
- **L327** EN: Assigns or updates `local_shards`. | CN: 对 `local_shards` 进行赋值或更新。
- **L328** EN: Assigns or updates `current_rank`. | CN: 对 `current_rank` 进行赋值或更新。
- **L329** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L330** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L331** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L332** EN: Calls `local_shards.append` as part of the current workflow. | CN: 在当前流程中调用 `local_shards.append`。
- **L333** EN: Calls `Shard` as part of the current workflow. | CN: 在当前流程中调用 `Shard`。
- **L334** EN: Assigns or updates `tensor`. | CN: 对 `tensor` 进行赋值或更新。
- **L335** EN: Continues the implementation inside function `load_sharded_optimizer_state_dict`. | CN: 继续说明函数 `load_sharded_optimizer_state_dict` 内部的实现。
- **L336** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L337** EN: Assigns or updates `metadata`. | CN: 对 `metadata` 进行赋值或更新。
- **L338** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L339** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L340** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 341-360 / 第 341-360 行

````python
            st = ShardedTensor._init_from_local_shards_and_global_metadata(
                local_shards, st_md, process_group=dp_pg
            )

            if spec_key in layout_specs and layout_specs[spec_key][0] is not None:
                fqn_to_offset[key] = cast(Sequence[int], layout_specs[spec_key][0])

            state_dict[key] = st

    # Whether we unflatten before or after doesn't matter
    load_state_dict(
        state_dict=state_dict,
        storage_reader=storage_reader,
        # FIXME the type of planner is wrong in load_state_dict
        planner=_ReaderWithOffset(fqn_to_offset) if dp_pg is not None else planner,
    )

    state_dict = unflatten_state_dict(state_dict, metadata.planner_data)

    return state_dict
````

- **L341** EN: Assigns or updates `st`. | CN: 对 `st` 进行赋值或更新。
- **L342** EN: Assigns or updates `local_shards, st_md, process_group`. | CN: 对 `local_shards, st_md, process_group` 进行赋值或更新。
- **L343** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L344** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L345** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L346** EN: Assigns or updates `fqn_to_offset[key]`. | CN: 对 `fqn_to_offset[key]` 进行赋值或更新。
- **L347** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L348** EN: Assigns or updates `state_dict[key]`. | CN: 对 `state_dict[key]` 进行赋值或更新。
- **L349** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L350** EN: Keeps the inline comment or directive: Whether we unflatten before or after doesn't matter | CN: 保留这一行注释或指令：Whether we unflatten before or after doesn't matter
- **L351** EN: Calls `load_state_dict` as part of the current workflow. | CN: 在当前流程中调用 `load_state_dict`。
- **L352** EN: Assigns or updates `state_dict`. | CN: 对 `state_dict` 进行赋值或更新。
- **L353** EN: Assigns or updates `storage_reader`. | CN: 对 `storage_reader` 进行赋值或更新。
- **L354** EN: Keeps the inline comment or directive: FIXME the type of planner is wrong in load_state_dict | CN: 保留这一行注释或指令：FIXME the type of planner is wrong in load_state_dict
- **L355** EN: Assigns or updates `planner`. | CN: 对 `planner` 进行赋值或更新。
- **L356** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L357** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L358** EN: Assigns or updates `state_dict`. | CN: 对 `state_dict` 进行赋值或更新。
- **L359** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L360** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

## Key Concepts / 关键概念

- **EN**: distributed checkpointing, planners, and storage helpers  
  **CN**: 分布式检查点、规划器与存储辅助逻辑
- **EN**: process groups  
  **CN**: 进程组
- **EN**: sharding  
  **CN**: 分片
- **EN**: checkpointing  
  **CN**: 检查点
- **EN**: distributed tensors  
  **CN**: 分布式张量
- **EN**: placements  
  **CN**: 放置规则
- **EN**: optimizer coordination  
  **CN**: 优化器协同
- **EN**: state dict handling  
  **CN**: state_dict 处理

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed`, `torch.distributed._shard.sharded_tensor.api`, `torch.distributed._shard.sharded_tensor.metadata`, `torch.distributed._shard.sharded_tensor.shard`, `torch.distributed._shard.sharding_spec.chunk_sharding_spec`, `torch.distributed.checkpoint._nested_dict`, `torch.distributed.checkpoint.default_planner`, `torch.distributed.checkpoint.metadata`, `torch.distributed.checkpoint.planner`, `torch.distributed.checkpoint.planner_helpers`, `torch.distributed.checkpoint.state_dict_loader`, `torch.distributed.checkpoint.storage`, `torch.distributed.checkpoint.utils`, `torch.distributed.distributed_c10d`, `torch.distributed.fsdp._shard_utils`, `torch.distributed.remote_device`, `torch.distributed.tensor`
- **PyTorch / PyTorch**: `torch`, `torch._utils`
- **Python Stdlib / Python 标准库**: `collections.abc`, `dataclasses`, `typing`
- **Third-party / 第三方**: None detected / 未检测到

