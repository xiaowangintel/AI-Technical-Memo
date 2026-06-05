# _shard_utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/fsdp/_shard_utils.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on Fully Sharded Data Parallel runtime and utilities. Its main entry points include _get_remote_device_str, _create_chunk_sharded_tensor.
- **用途 (CN)**: 该模块聚焦于Fully Sharded Data Parallel（FSDP）运行时与工具，其主要入口包括 _get_remote_device_str, _create_chunk_sharded_tensor。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs
import copy
import itertools
import math

import torch
import torch.distributed as dist
from torch._utils import _get_device_module
from torch.distributed import distributed_c10d
from torch.distributed._shard.sharded_tensor import (
    Shard,
    ShardedTensor,
    ShardedTensorMetadata,
    TensorProperties,
)
from torch.distributed._shard.sharding_spec import ShardMetadata
from torch.distributed.tensor import DeviceMesh, DTensor, Replicate, Shard as DShard


def _get_remote_device_str(rank, device_type, num_devices_per_node):
````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Imports module dependencies: `copy`. | CN: 导入模块依赖：`copy`。
- **L3** EN: Imports module dependencies: `itertools`. | CN: 导入模块依赖：`itertools`。
- **L4** EN: Imports module dependencies: `math`. | CN: 导入模块依赖：`math`。
- **L5** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L6** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L7** EN: Imports module dependencies: `torch.distributed as dist`. | CN: 导入模块依赖：`torch.distributed as dist`。
- **L8** EN: Imports selected names from `torch._utils`. | CN: 从 `torch._utils` 导入指定名称。
- **L9** EN: Imports selected names from `torch.distributed`. | CN: 从 `torch.distributed` 导入指定名称。
- **L10** EN: Imports selected names from `torch.distributed._shard.sharded_tensor`. | CN: 从 `torch.distributed._shard.sharded_tensor` 导入指定名称。
- **L11** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L12** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L13** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L14** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L15** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L16** EN: Imports selected names from `torch.distributed._shard.sharding_spec`. | CN: 从 `torch.distributed._shard.sharding_spec` 导入指定名称。
- **L17** EN: Imports selected names from `torch.distributed.tensor`. | CN: 从 `torch.distributed.tensor` 导入指定名称。
- **L18** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L19** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L20** EN: Defines function `_get_remote_device_str`. | CN: 定义函数 `_get_remote_device_str`。

### Lines 21-40 / 第 21-40 行

````python
    if device_type.lower() == "cpu":
        return f"rank:{rank}/{device_type}"
    elif device_type.lower() == "hpu":
        return f"rank:{rank}/{device_type}:{_get_device_module(device_type).current_device()}"
    else:
        return f"rank:{rank}/{device_type}:{rank % num_devices_per_node}"


def _create_chunk_sharded_tensor(
    tensor: torch.Tensor,
    rank: int,
    world_size: int,
    num_devices_per_node: int,
    pg: dist.ProcessGroup,
    device: torch.device | None = None,
) -> ShardedTensor:
    """
    Shard a tensor to chunks along the first dimension. The local rank will gets its
    corresponding chunk as the local shard to create a ShardedTensor.
    """
````

- **L21** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L22** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L23** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L24** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L25** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L26** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L27** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L28** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L29** EN: Defines function `_create_chunk_sharded_tensor`. | CN: 定义函数 `_create_chunk_sharded_tensor`。
- **L30** EN: Continues the implementation inside function `_create_chunk_sharded_tensor`. | CN: 继续说明函数 `_create_chunk_sharded_tensor` 内部的实现。
- **L31** EN: Continues the implementation inside function `_create_chunk_sharded_tensor`. | CN: 继续说明函数 `_create_chunk_sharded_tensor` 内部的实现。
- **L32** EN: Continues the implementation inside function `_create_chunk_sharded_tensor`. | CN: 继续说明函数 `_create_chunk_sharded_tensor` 内部的实现。
- **L33** EN: Continues the implementation inside function `_create_chunk_sharded_tensor`. | CN: 继续说明函数 `_create_chunk_sharded_tensor` 内部的实现。
- **L34** EN: Continues the implementation inside function `_create_chunk_sharded_tensor`. | CN: 继续说明函数 `_create_chunk_sharded_tensor` 内部的实现。
- **L35** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L36** EN: Continues the implementation inside function `_create_chunk_sharded_tensor`. | CN: 继续说明函数 `_create_chunk_sharded_tensor` 内部的实现。
- **L37** EN: Starts the docstring for the function _create_chunk_sharded_tensor. | CN: 开始定义 function _create_chunk_sharded_tensor 的文档字符串。
- **L38** EN: Continues the docstring text for the function _create_chunk_sharded_tensor. | CN: 继续补充 function _create_chunk_sharded_tensor 的文档字符串内容。
- **L39** EN: Continues the docstring text for the function _create_chunk_sharded_tensor. | CN: 继续补充 function _create_chunk_sharded_tensor 的文档字符串内容。
- **L40** EN: Closes the docstring for the function _create_chunk_sharded_tensor. | CN: 结束 function _create_chunk_sharded_tensor 的文档字符串。

### Lines 41-60 / 第 41-60 行

````python
    chunks = tensor.chunk(world_size, dim=0)
    if len(chunks) > rank:
        local_shard = chunks[rank].clone()
        offsets = [0 for _ in tensor.size()]
        offsets[0] = math.ceil(tensor.size()[0] / world_size) * rank
        local_shards = [Shard.from_tensor_and_offsets(local_shard, offsets, rank)]
    else:
        local_shards = []

    # Create a ShardedTensor without invoking communication.
    chunk_sizes = [list(chunk.size()) for chunk in chunks]
    dim0_offsets = [0] + list(
        itertools.accumulate([chunk_size[0] for chunk_size in chunk_sizes])
    )[:-1]
    offsets = [0] * (len(chunk_sizes[0]) - 1)
    chunk_offsets = [[d0] + offsets for d0 in dim0_offsets]
    device_type = (
        distributed_c10d._get_pg_default_device(pg).type
        if device is None
        else device.type
````

- **L41** EN: Assigns or updates `chunks`. | CN: 对 `chunks` 进行赋值或更新。
- **L42** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L43** EN: Assigns or updates `local_shard`. | CN: 对 `local_shard` 进行赋值或更新。
- **L44** EN: Assigns or updates `offsets`. | CN: 对 `offsets` 进行赋值或更新。
- **L45** EN: Assigns or updates `offsets[0]`. | CN: 对 `offsets[0]` 进行赋值或更新。
- **L46** EN: Assigns or updates `local_shards`. | CN: 对 `local_shards` 进行赋值或更新。
- **L47** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L48** EN: Assigns or updates `local_shards`. | CN: 对 `local_shards` 进行赋值或更新。
- **L49** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L50** EN: Keeps the inline comment or directive: Create a ShardedTensor without invoking communication. | CN: 保留这一行注释或指令：Create a ShardedTensor without invoking communication.
- **L51** EN: Assigns or updates `chunk_sizes`. | CN: 对 `chunk_sizes` 进行赋值或更新。
- **L52** EN: Assigns or updates `dim0_offsets`. | CN: 对 `dim0_offsets` 进行赋值或更新。
- **L53** EN: Calls `itertools.accumulate` as part of the current workflow. | CN: 在当前流程中调用 `itertools.accumulate`。
- **L54** EN: Continues the implementation inside function `_create_chunk_sharded_tensor`. | CN: 继续说明函数 `_create_chunk_sharded_tensor` 内部的实现。
- **L55** EN: Assigns or updates `offsets`. | CN: 对 `offsets` 进行赋值或更新。
- **L56** EN: Assigns or updates `chunk_offsets`. | CN: 对 `chunk_offsets` 进行赋值或更新。
- **L57** EN: Assigns or updates `device_type`. | CN: 对 `device_type` 进行赋值或更新。
- **L58** EN: Calls `distributed_c10d._get_pg_default_device` as part of the current workflow. | CN: 在当前流程中调用 `distributed_c10d._get_pg_default_device`。
- **L59** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L60** EN: Continues the implementation inside function `_create_chunk_sharded_tensor`. | CN: 继续说明函数 `_create_chunk_sharded_tensor` 内部的实现。

### Lines 61-80 / 第 61-80 行

````python
    )
    placements = [
        _get_remote_device_str(
            dist.get_global_rank(pg, r),
            device_type,
            num_devices_per_node,
        )
        for r in range(len(chunk_sizes))
    ]
    if len(chunk_sizes) != len(chunk_offsets) or len(chunk_sizes) != len(placements):
        raise AssertionError(
            f"Expected chunk_sizes, chunk_offsets, and placements to have the same length, "
            f"got {len(chunk_sizes)}, {len(chunk_offsets)}, {len(placements)}"
        )
    shard_metadata = [
        ShardMetadata(offset, size, placement)
        for offset, size, placement in zip(chunk_offsets, chunk_sizes, placements)
    ]
    sharded_tensor_metadata = ShardedTensorMetadata(
        shards_metadata=shard_metadata,
````

- **L61** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L62** EN: Assigns or updates `placements`. | CN: 对 `placements` 进行赋值或更新。
- **L63** EN: Calls `_get_remote_device_str` as part of the current workflow. | CN: 在当前流程中调用 `_get_remote_device_str`。
- **L64** EN: Calls `dist.get_global_rank` as part of the current workflow. | CN: 在当前流程中调用 `dist.get_global_rank`。
- **L65** EN: Continues the implementation inside function `_create_chunk_sharded_tensor`. | CN: 继续说明函数 `_create_chunk_sharded_tensor` 内部的实现。
- **L66** EN: Continues the implementation inside function `_create_chunk_sharded_tensor`. | CN: 继续说明函数 `_create_chunk_sharded_tensor` 内部的实现。
- **L67** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L68** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L69** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L70** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L71** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L72** EN: Continues the implementation inside function `_create_chunk_sharded_tensor`. | CN: 继续说明函数 `_create_chunk_sharded_tensor` 内部的实现。
- **L73** EN: Continues the implementation inside function `_create_chunk_sharded_tensor`. | CN: 继续说明函数 `_create_chunk_sharded_tensor` 内部的实现。
- **L74** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L75** EN: Assigns or updates `shard_metadata`. | CN: 对 `shard_metadata` 进行赋值或更新。
- **L76** EN: Calls `ShardMetadata` as part of the current workflow. | CN: 在当前流程中调用 `ShardMetadata`。
- **L77** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L78** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L79** EN: Assigns or updates `sharded_tensor_metadata`. | CN: 对 `sharded_tensor_metadata` 进行赋值或更新。
- **L80** EN: Assigns or updates `shards_metadata`. | CN: 对 `shards_metadata` 进行赋值或更新。

### Lines 81-100 / 第 81-100 行

````python
        size=tensor.size(),
        tensor_properties=TensorProperties(
            dtype=tensor.dtype,
            layout=tensor.layout,
            requires_grad=False,
            memory_format=torch.contiguous_format,
            pin_memory=tensor.is_pinned(),
        ),
    )
    return ShardedTensor._init_from_local_shards_and_global_metadata(
        local_shards, sharded_tensor_metadata=sharded_tensor_metadata, process_group=pg
    )


def _create_chunk_dtensor(
    tensor: torch.Tensor,
    rank: int,
    device_mesh: DeviceMesh,
) -> DTensor:
    """
````

- **L81** EN: Assigns or updates `size`. | CN: 对 `size` 进行赋值或更新。
- **L82** EN: Assigns or updates `tensor_properties`. | CN: 对 `tensor_properties` 进行赋值或更新。
- **L83** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L84** EN: Assigns or updates `layout`. | CN: 对 `layout` 进行赋值或更新。
- **L85** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L86** EN: Assigns or updates `memory_format`. | CN: 对 `memory_format` 进行赋值或更新。
- **L87** EN: Assigns or updates `pin_memory`. | CN: 对 `pin_memory` 进行赋值或更新。
- **L88** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L89** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L90** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L91** EN: Assigns or updates `local_shards, sharded_tensor_metadata`. | CN: 对 `local_shards, sharded_tensor_metadata` 进行赋值或更新。
- **L92** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L93** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L94** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L95** EN: Defines function `_create_chunk_dtensor`. | CN: 定义函数 `_create_chunk_dtensor`。
- **L96** EN: Continues the implementation inside function `_create_chunk_dtensor`. | CN: 继续说明函数 `_create_chunk_dtensor` 内部的实现。
- **L97** EN: Continues the implementation inside function `_create_chunk_dtensor`. | CN: 继续说明函数 `_create_chunk_dtensor` 内部的实现。
- **L98** EN: Continues the implementation inside function `_create_chunk_dtensor`. | CN: 继续说明函数 `_create_chunk_dtensor` 内部的实现。
- **L99** EN: Continues the implementation inside function `_create_chunk_dtensor`. | CN: 继续说明函数 `_create_chunk_dtensor` 内部的实现。
- **L100** EN: Starts the docstring for the function _create_chunk_dtensor. | CN: 开始定义 function _create_chunk_dtensor 的文档字符串。

### Lines 101-120 / 第 101-120 行

````python
    Shard a tensor to chunks along the first dimension. The local rank will gets its
    corresponding chunk as the local tensor to create a DTensor.
    """
    # We need to explicitly call .detach() to return a new tensor detached from the current graph.
    tensor = tensor.detach().clone()

    # FSDP placements: [Shard(0)]
    # HSDP placements: [Replicate(), Shard(0)]
    replicate_placements = [Replicate() for _ in range(device_mesh.ndim)]
    shard_placements = [Replicate() for _ in range(device_mesh.ndim)]
    shard_placements[-1] = DShard(0)  # type: ignore[call-overload]

    return DTensor.from_local(
        tensor, device_mesh, replicate_placements, run_check=False
    ).redistribute(
        placements=shard_placements,
    )


def _all_gather_dtensor(
````

- **L101** EN: Continues the docstring text for the function _create_chunk_dtensor. | CN: 继续补充 function _create_chunk_dtensor 的文档字符串内容。
- **L102** EN: Continues the docstring text for the function _create_chunk_dtensor. | CN: 继续补充 function _create_chunk_dtensor 的文档字符串内容。
- **L103** EN: Closes the docstring for the function _create_chunk_dtensor. | CN: 结束 function _create_chunk_dtensor 的文档字符串。
- **L104** EN: Keeps the inline comment or directive: We need to explicitly call .detach() to return a new tensor detached from the cu | CN: 保留这一行注释或指令：We need to explicitly call .detach() to return a new tensor detached from the cu
- **L105** EN: Assigns or updates `tensor`. | CN: 对 `tensor` 进行赋值或更新。
- **L106** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L107** EN: Keeps the inline comment or directive: FSDP placements: [Shard(0)] | CN: 保留这一行注释或指令：FSDP placements: [Shard(0)]
- **L108** EN: Keeps the inline comment or directive: HSDP placements: [Replicate(), Shard(0)] | CN: 保留这一行注释或指令：HSDP placements: [Replicate(), Shard(0)]
- **L109** EN: Assigns or updates `replicate_placements`. | CN: 对 `replicate_placements` 进行赋值或更新。
- **L110** EN: Assigns or updates `shard_placements`. | CN: 对 `shard_placements` 进行赋值或更新。
- **L111** EN: Continues the implementation inside function `_create_chunk_dtensor`. | CN: 继续说明函数 `_create_chunk_dtensor` 内部的实现。
- **L112** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L113** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L114** EN: Assigns or updates `tensor, device_mesh, replicate_placements, run_check`. | CN: 对 `tensor, device_mesh, replicate_placements, run_check` 进行赋值或更新。
- **L115** EN: Continues the implementation inside function `_create_chunk_dtensor`. | CN: 继续说明函数 `_create_chunk_dtensor` 内部的实现。
- **L116** EN: Assigns or updates `placements`. | CN: 对 `placements` 进行赋值或更新。
- **L117** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L118** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L119** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L120** EN: Defines function `_all_gather_dtensor`. | CN: 定义函数 `_all_gather_dtensor`。

### Lines 121-139 / 第 121-139 行

````python
    tensor: DTensor,
    root_mesh: DeviceMesh | None,
) -> torch.Tensor:
    """
    All gather a DTensor in its sharded dimension and return the local tensor.
    """
    if root_mesh != tensor.device_mesh:
        raise AssertionError("The device mesh of a tensor should be a root mesh.")

    placements = list(copy.deepcopy(tensor.placements))
    # FSDP placements: [Shard(0)] -> [Replicate()]
    # HSDP placements: [Replicate(), Shard(0)] -> [Replicate(), Replicate()]
    placements[-1] = Replicate()
    tensor = tensor.redistribute(
        device_mesh=tensor.device_mesh,
        placements=placements,
    )

    return tensor.to_local()
````

- **L121** EN: Continues the implementation inside function `_all_gather_dtensor`. | CN: 继续说明函数 `_all_gather_dtensor` 内部的实现。
- **L122** EN: Continues the implementation inside function `_all_gather_dtensor`. | CN: 继续说明函数 `_all_gather_dtensor` 内部的实现。
- **L123** EN: Continues the implementation inside function `_all_gather_dtensor`. | CN: 继续说明函数 `_all_gather_dtensor` 内部的实现。
- **L124** EN: Starts the docstring for the function _all_gather_dtensor. | CN: 开始定义 function _all_gather_dtensor 的文档字符串。
- **L125** EN: Continues the docstring text for the function _all_gather_dtensor. | CN: 继续补充 function _all_gather_dtensor 的文档字符串内容。
- **L126** EN: Closes the docstring for the function _all_gather_dtensor. | CN: 结束 function _all_gather_dtensor 的文档字符串。
- **L127** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L128** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L129** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L130** EN: Assigns or updates `placements`. | CN: 对 `placements` 进行赋值或更新。
- **L131** EN: Keeps the inline comment or directive: FSDP placements: [Shard(0)] -> [Replicate()] | CN: 保留这一行注释或指令：FSDP placements: [Shard(0)] -> [Replicate()]
- **L132** EN: Keeps the inline comment or directive: HSDP placements: [Replicate(), Shard(0)] -> [Replicate(), Replicate()] | CN: 保留这一行注释或指令：HSDP placements: [Replicate(), Shard(0)] -> [Replicate(), Replicate()]
- **L133** EN: Continues the implementation inside function `_all_gather_dtensor`. | CN: 继续说明函数 `_all_gather_dtensor` 内部的实现。
- **L134** EN: Assigns or updates `tensor`. | CN: 对 `tensor` 进行赋值或更新。
- **L135** EN: Assigns or updates `device_mesh`. | CN: 对 `device_mesh` 进行赋值或更新。
- **L136** EN: Assigns or updates `placements`. | CN: 对 `placements` 进行赋值或更新。
- **L137** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L138** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L139** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

## Key Concepts / 关键概念

- **EN**: Fully Sharded Data Parallel runtime and utilities  
  **CN**: Fully Sharded Data Parallel（FSDP）运行时与工具
- **EN**: process groups  
  **CN**: 进程组
- **EN**: device mesh  
  **CN**: 设备网格
- **EN**: sharding  
  **CN**: 分片
- **EN**: all-gather  
  **CN**: all-gather 聚合
- **EN**: distributed tensors  
  **CN**: 分布式张量
- **EN**: placements  
  **CN**: 放置规则
- **EN**: Core callables: _get_remote_device_str, _create_chunk_sharded_tensor, _create_chunk_dtensor, _all_gather_dtensor  
  **CN**: 核心可调用对象：_get_remote_device_str, _create_chunk_sharded_tensor, _create_chunk_dtensor, _all_gather_dtensor

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed`, `torch.distributed._shard.sharded_tensor`, `torch.distributed._shard.sharding_spec`, `torch.distributed.tensor`
- **PyTorch / PyTorch**: `torch`, `torch._utils`
- **Python Stdlib / Python 标准库**: `copy`, `itertools`, `math`
- **Third-party / 第三方**: None detected / 未检测到

