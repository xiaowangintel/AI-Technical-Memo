# torchrec_sharding_example.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/tensor/examples/torchrec_sharding_example.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed tensor layouts, placements, and operators. Its main entry points include LocalShardsWrapper, get_device_type, run_torchrec_row_wise_even_sharding_example.
- **用途 (CN)**: 该模块聚焦于分布式张量布局、放置与算子逻辑，其主要入口包括 LocalShardsWrapper, get_device_type, run_torchrec_row_wise_even_sharding_example。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs
"""
The following example demonstrates how to represent torchrec's embedding
sharding with the DTensor API.
"""

import argparse
import os
from functools import cached_property
from typing import TYPE_CHECKING

import torch
from torch.distributed.checkpoint.metadata import (
    ChunkStorageMetadata,
    TensorProperties,
    TensorStorageMetadata,
)
from torch.distributed.tensor import (
    DeviceMesh,
    DTensor,
````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Starts the docstring for the module. | CN: 开始定义 module 的文档字符串。
- **L3** EN: Continues the docstring text for the module. | CN: 继续补充 module 的文档字符串内容。
- **L4** EN: Continues the docstring text for the module. | CN: 继续补充 module 的文档字符串内容。
- **L5** EN: Closes the docstring for the module. | CN: 结束 module 的文档字符串。
- **L6** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L7** EN: Imports module dependencies: `argparse`. | CN: 导入模块依赖：`argparse`。
- **L8** EN: Imports module dependencies: `os`. | CN: 导入模块依赖：`os`。
- **L9** EN: Imports selected names from `functools`. | CN: 从 `functools` 导入指定名称。
- **L10** EN: Imports selected names from `typing`. | CN: 从 `typing` 导入指定名称。
- **L11** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L12** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L13** EN: Imports selected names from `torch.distributed.checkpoint.metadata`. | CN: 从 `torch.distributed.checkpoint.metadata` 导入指定名称。
- **L14** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L15** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L16** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L17** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L18** EN: Imports selected names from `torch.distributed.tensor`. | CN: 从 `torch.distributed.tensor` 导入指定名称。
- **L19** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L20** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。

### Lines 21-40 / 第 21-40 行

````python
    init_device_mesh,
    Replicate,
    Shard,
)
from torch.distributed.tensor.debug import visualize_sharding


if TYPE_CHECKING:
    from torch.distributed.tensor.placement_types import Placement


def get_device_type():
    return (
        "cuda"
        if torch.cuda.is_available() and torch.cuda.device_count() >= 4
        else "cpu"
    )


aten = torch.ops.aten
````

- **L21** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L22** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L23** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L24** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L25** EN: Imports selected names from `torch.distributed.tensor.debug`. | CN: 从 `torch.distributed.tensor.debug` 导入指定名称。
- **L26** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L27** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L28** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L29** EN: Imports selected names from `torch.distributed.tensor.placement_types`. | CN: 从 `torch.distributed.tensor.placement_types` 导入指定名称。
- **L30** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L31** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L32** EN: Defines function `get_device_type`. | CN: 定义函数 `get_device_type`。
- **L33** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L34** EN: Continues the implementation inside function `get_device_type`. | CN: 继续说明函数 `get_device_type` 内部的实现。
- **L35** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L36** EN: Continues the implementation inside function `get_device_type`. | CN: 继续说明函数 `get_device_type` 内部的实现。
- **L37** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L38** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L39** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L40** EN: Assigns or updates `aten`. | CN: 对 `aten` 进行赋值或更新。

### Lines 41-60 / 第 41-60 行

````python
supported_ops = [aten.view.default, aten._to_copy.default]


# this torch.Tensor subclass is a wrapper around all local shards associated
# with a single sharded embedding table.
class LocalShardsWrapper(torch.Tensor):
    local_shards: list[torch.Tensor]
    storage_meta: TensorStorageMetadata

    @staticmethod
    def __new__(
        cls, local_shards: list[torch.Tensor], offsets: list[torch.Size]
    ) -> "LocalShardsWrapper":
        if len(local_shards) <= 0:
            raise AssertionError
        if len(local_shards) != len(offsets):
            raise AssertionError
        if local_shards[0].ndim != 2:
            raise AssertionError
        # we calculate the total tensor size by "concat" on second tensor dimension
````

- **L41** EN: Assigns or updates `supported_ops`. | CN: 对 `supported_ops` 进行赋值或更新。
- **L42** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L43** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L44** EN: Keeps the inline comment or directive: this torch.Tensor subclass is a wrapper around all local shards associated | CN: 保留这一行注释或指令：this torch.Tensor subclass is a wrapper around all local shards associated
- **L45** EN: Keeps the inline comment or directive: with a single sharded embedding table. | CN: 保留这一行注释或指令：with a single sharded embedding table.
- **L46** EN: Defines class `LocalShardsWrapper`. | CN: 定义类 `LocalShardsWrapper`。
- **L47** EN: Continues the implementation inside class `LocalShardsWrapper`. | CN: 继续说明类 `LocalShardsWrapper` 内部的实现。
- **L48** EN: Continues the implementation inside class `LocalShardsWrapper`. | CN: 继续说明类 `LocalShardsWrapper` 内部的实现。
- **L49** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L50** EN: Applies decorator `staticmethod` to the following definition. | CN: 将装饰器 `staticmethod` 应用于后续定义。
- **L51** EN: Defines function `__new__`. | CN: 定义函数 `__new__`。
- **L52** EN: Continues the implementation inside function `__new__`. | CN: 继续说明函数 `__new__` 内部的实现。
- **L53** EN: Continues the implementation inside function `__new__`. | CN: 继续说明函数 `__new__` 内部的实现。
- **L54** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L55** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L56** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L57** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L58** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L59** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L60** EN: Keeps the inline comment or directive: we calculate the total tensor size by "concat" on second tensor dimension | CN: 保留这一行注释或指令：we calculate the total tensor size by "concat" on second tensor dimension

### Lines 61-80 / 第 61-80 行

````python
        cat_tensor_shape = list(local_shards[0].shape)
        if len(local_shards) > 1:  # column-wise sharding
            for shard_size in [s.shape for s in local_shards[1:]]:
                cat_tensor_shape[1] += shard_size[1]

        # according to DCP, each chunk is expected to have the same properties of the
        # TensorStorageMetadata that includes it. Vice versa, the wrapper's properties
        # should also be the same with that of its first chunk.
        wrapper_properties = TensorProperties.create_from_tensor(local_shards[0])
        wrapper_shape = torch.Size(cat_tensor_shape)
        chunks_meta = [
            ChunkStorageMetadata(o, s.shape) for s, o in zip(local_shards, offsets)
        ]

        r = torch.Tensor._make_wrapper_subclass(
            cls,
            wrapper_shape,
        )
        r.shards = local_shards
        r.storage_meta = TensorStorageMetadata(
````

- **L61** EN: Assigns or updates `cat_tensor_shape`. | CN: 对 `cat_tensor_shape` 进行赋值或更新。
- **L62** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L63** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L64** EN: Continues the implementation inside function `__new__`. | CN: 继续说明函数 `__new__` 内部的实现。
- **L65** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L66** EN: Keeps the inline comment or directive: according to DCP, each chunk is expected to have the same properties of the | CN: 保留这一行注释或指令：according to DCP, each chunk is expected to have the same properties of the
- **L67** EN: Keeps the inline comment or directive: TensorStorageMetadata that includes it. Vice versa, the wrapper's properties | CN: 保留这一行注释或指令：TensorStorageMetadata that includes it. Vice versa, the wrapper's properties
- **L68** EN: Keeps the inline comment or directive: should also be the same with that of its first chunk. | CN: 保留这一行注释或指令：should also be the same with that of its first chunk.
- **L69** EN: Assigns or updates `wrapper_properties`. | CN: 对 `wrapper_properties` 进行赋值或更新。
- **L70** EN: Assigns or updates `wrapper_shape`. | CN: 对 `wrapper_shape` 进行赋值或更新。
- **L71** EN: Assigns or updates `chunks_meta`. | CN: 对 `chunks_meta` 进行赋值或更新。
- **L72** EN: Calls `ChunkStorageMetadata` as part of the current workflow. | CN: 在当前流程中调用 `ChunkStorageMetadata`。
- **L73** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L74** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L75** EN: Assigns or updates `r`. | CN: 对 `r` 进行赋值或更新。
- **L76** EN: Continues the implementation inside function `__new__`. | CN: 继续说明函数 `__new__` 内部的实现。
- **L77** EN: Continues the implementation inside function `__new__`. | CN: 继续说明函数 `__new__` 内部的实现。
- **L78** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L79** EN: Assigns or updates `r.shards`. | CN: 对 `r.shards` 进行赋值或更新。
- **L80** EN: Assigns or updates `r.storage_meta`. | CN: 对 `r.storage_meta` 进行赋值或更新。

### Lines 81-100 / 第 81-100 行

````python
            properties=wrapper_properties,
            size=wrapper_shape,
            chunks=chunks_meta,
        )

        return r

    # necessary for ops dispatching from this subclass to its local shards
    @classmethod
    def __torch_dispatch__(cls, func, types, args=(), kwargs=None):  # type: ignore[override]
        kwargs = kwargs or {}

        # TODO: we shall continually extend this function to support more ops if needed
        if func in supported_ops:
            res_shards_list = [
                func(shard, *args[1:], **kwargs)
                # pyrefly: ignore [bad-index]
                for shard in args[0].shards
            ]
            # pyrefly: ignore [bad-index]
````

- **L81** EN: Assigns or updates `properties`. | CN: 对 `properties` 进行赋值或更新。
- **L82** EN: Assigns or updates `size`. | CN: 对 `size` 进行赋值或更新。
- **L83** EN: Assigns or updates `chunks`. | CN: 对 `chunks` 进行赋值或更新。
- **L84** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L85** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L86** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L87** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L88** EN: Keeps the inline comment or directive: necessary for ops dispatching from this subclass to its local shards | CN: 保留这一行注释或指令：necessary for ops dispatching from this subclass to its local shards
- **L89** EN: Applies decorator `classmethod` to the following definition. | CN: 将装饰器 `classmethod` 应用于后续定义。
- **L90** EN: Defines function `__torch_dispatch__`. | CN: 定义函数 `__torch_dispatch__`。
- **L91** EN: Assigns or updates `kwargs`. | CN: 对 `kwargs` 进行赋值或更新。
- **L92** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L93** EN: Keeps the inline comment or directive: TODO: we shall continually extend this function to support more ops if needed | CN: 保留这一行注释或指令：TODO: we shall continually extend this function to support more ops if needed
- **L94** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L95** EN: Assigns or updates `res_shards_list`. | CN: 对 `res_shards_list` 进行赋值或更新。
- **L96** EN: Calls `func` as part of the current workflow. | CN: 在当前流程中调用 `func`。
- **L97** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-index] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-index]
- **L98** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L99** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L100** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-index] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-index]

### Lines 101-120 / 第 101-120 行

````python
            return LocalShardsWrapper(res_shards_list, args[0].shard_offsets)
        else:
            raise NotImplementedError(
                f"{func} is not supported for LocalShardsWrapper!"
            )

    @property
    def shards(self) -> list[torch.Tensor]:
        return self.local_shards

    @shards.setter
    def shards(self, local_shards: list[torch.Tensor]):
        self.local_shards = local_shards

    @cached_property
    def shard_sizes(self) -> list[torch.Size]:
        return [chunk.sizes for chunk in self.storage_meta.chunks]

    @cached_property
    def shard_offsets(self) -> list[torch.Size]:
````

- **L101** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L102** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L103** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L104** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L105** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L106** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L107** EN: Applies decorator `property` to the following definition. | CN: 将装饰器 `property` 应用于后续定义。
- **L108** EN: Defines function `shards`. | CN: 定义函数 `shards`。
- **L109** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L110** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L111** EN: Applies decorator `shards.setter` to the following definition. | CN: 将装饰器 `shards.setter` 应用于后续定义。
- **L112** EN: Defines function `shards`. | CN: 定义函数 `shards`。
- **L113** EN: Assigns or updates `self.local_shards`. | CN: 对 `self.local_shards` 进行赋值或更新。
- **L114** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L115** EN: Applies decorator `cached_property` to the following definition. | CN: 将装饰器 `cached_property` 应用于后续定义。
- **L116** EN: Defines function `shard_sizes`. | CN: 定义函数 `shard_sizes`。
- **L117** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L118** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L119** EN: Applies decorator `cached_property` to the following definition. | CN: 将装饰器 `cached_property` 应用于后续定义。
- **L120** EN: Defines function `shard_offsets`. | CN: 定义函数 `shard_offsets`。

### Lines 121-140 / 第 121-140 行

````python
        return [chunk.offsets for chunk in self.storage_meta.chunks]


def run_torchrec_row_wise_even_sharding_example(rank, world_size):
    # row-wise even sharding example:
    #   One table is evenly sharded by rows within the global ProcessGroup.
    #   In our example, the table's num_embedding is 8, and the embedding dim is 16
    #   The global ProcessGroup has 4 ranks, so each rank will have one 2 by 16 local
    #   shard.

    # device mesh is a representation of the worker ranks
    # create a 1-D device mesh that includes every rank
    device_type = get_device_type()
    device = torch.device(device_type)
    device_mesh = init_device_mesh(device_type=device_type, mesh_shape=(world_size,))

    # manually create the embedding table's local shards
    num_embeddings = 8
    embedding_dim = 16
    # tensor shape
````

- **L121** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L122** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L123** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L124** EN: Defines function `run_torchrec_row_wise_even_sharding_example`. | CN: 定义函数 `run_torchrec_row_wise_even_sharding_example`。
- **L125** EN: Keeps the inline comment or directive: row-wise even sharding example: | CN: 保留这一行注释或指令：row-wise even sharding example:
- **L126** EN: Keeps the inline comment or directive: One table is evenly sharded by rows within the global ProcessGroup. | CN: 保留这一行注释或指令：One table is evenly sharded by rows within the global ProcessGroup.
- **L127** EN: Keeps the inline comment or directive: In our example, the table's num_embedding is 8, and the embedding dim is 16 | CN: 保留这一行注释或指令：In our example, the table's num_embedding is 8, and the embedding dim is 16
- **L128** EN: Keeps the inline comment or directive: The global ProcessGroup has 4 ranks, so each rank will have one 2 by 16 local | CN: 保留这一行注释或指令：The global ProcessGroup has 4 ranks, so each rank will have one 2 by 16 local
- **L129** EN: Keeps the inline comment or directive: shard. | CN: 保留这一行注释或指令：shard.
- **L130** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L131** EN: Keeps the inline comment or directive: device mesh is a representation of the worker ranks | CN: 保留这一行注释或指令：device mesh is a representation of the worker ranks
- **L132** EN: Keeps the inline comment or directive: create a 1-D device mesh that includes every rank | CN: 保留这一行注释或指令：create a 1-D device mesh that includes every rank
- **L133** EN: Assigns or updates `device_type`. | CN: 对 `device_type` 进行赋值或更新。
- **L134** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L135** EN: Assigns or updates `device_mesh`. | CN: 对 `device_mesh` 进行赋值或更新。
- **L136** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L137** EN: Keeps the inline comment or directive: manually create the embedding table's local shards | CN: 保留这一行注释或指令：manually create the embedding table's local shards
- **L138** EN: Assigns or updates `num_embeddings`. | CN: 对 `num_embeddings` 进行赋值或更新。
- **L139** EN: Assigns or updates `embedding_dim`. | CN: 对 `embedding_dim` 进行赋值或更新。
- **L140** EN: Keeps the inline comment or directive: tensor shape | CN: 保留这一行注释或指令：tensor shape

### Lines 141-160 / 第 141-160 行

````python
    local_shard_shape = torch.Size(
        [num_embeddings // world_size, embedding_dim]  # (local_rows, local_cols)
    )
    # tensor offset
    local_shard_offset = torch.Size((rank * 2, embedding_dim))
    # tensor
    local_tensor = torch.randn(local_shard_shape, device=device)
    # row-wise sharding: one shard per rank
    # create the local shards wrapper
    # pyrefly: ignore [no-matching-overload]
    local_shards_wrapper = LocalShardsWrapper(
        local_shards=[local_tensor],
        offsets=[local_shard_offset],
    )

    ###########################################################################
    # example 1: transform local_shards into DTensor
    # usage in TorchRec:
    #   ShardedEmbeddingCollection stores model parallel params in
    #   _model_parallel_name_to_sharded_tensor which is initialized in
````

- **L141** EN: Assigns or updates `local_shard_shape`. | CN: 对 `local_shard_shape` 进行赋值或更新。
- **L142** EN: Continues the implementation inside function `run_torchrec_row_wise_even_sharding_example`. | CN: 继续说明函数 `run_torchrec_row_wise_even_sharding_example` 内部的实现。
- **L143** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L144** EN: Keeps the inline comment or directive: tensor offset | CN: 保留这一行注释或指令：tensor offset
- **L145** EN: Assigns or updates `local_shard_offset`. | CN: 对 `local_shard_offset` 进行赋值或更新。
- **L146** EN: Keeps the inline comment or directive: tensor | CN: 保留这一行注释或指令：tensor
- **L147** EN: Assigns or updates `local_tensor`. | CN: 对 `local_tensor` 进行赋值或更新。
- **L148** EN: Keeps the inline comment or directive: row-wise sharding: one shard per rank | CN: 保留这一行注释或指令：row-wise sharding: one shard per rank
- **L149** EN: Keeps the inline comment or directive: create the local shards wrapper | CN: 保留这一行注释或指令：create the local shards wrapper
- **L150** EN: Keeps the inline comment or directive: pyrefly: ignore [no-matching-overload] | CN: 保留这一行注释或指令：pyrefly: ignore [no-matching-overload]
- **L151** EN: Assigns or updates `local_shards_wrapper`. | CN: 对 `local_shards_wrapper` 进行赋值或更新。
- **L152** EN: Assigns or updates `local_shards`. | CN: 对 `local_shards` 进行赋值或更新。
- **L153** EN: Assigns or updates `offsets`. | CN: 对 `offsets` 进行赋值或更新。
- **L154** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L155** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L156** EN: Keeps the inline comment or directive: ########################################################################## | CN: 保留这一行注释或指令：##########################################################################
- **L157** EN: Keeps the inline comment or directive: example 1: transform local_shards into DTensor | CN: 保留这一行注释或指令：example 1: transform local_shards into DTensor
- **L158** EN: Keeps the inline comment or directive: usage in TorchRec: | CN: 保留这一行注释或指令：usage in TorchRec:
- **L159** EN: Keeps the inline comment or directive: ShardedEmbeddingCollection stores model parallel params in | CN: 保留这一行注释或指令：ShardedEmbeddingCollection stores model parallel params in
- **L160** EN: Keeps the inline comment or directive: _model_parallel_name_to_sharded_tensor which is initialized in | CN: 保留这一行注释或指令：_model_parallel_name_to_sharded_tensor which is initialized in

### Lines 161-180 / 第 161-180 行

````python
    #   _initialize_torch_state() and torch.Tensor params are transformed
    #   into ShardedTensor by ShardedTensor._init_from_local_shards().
    #
    #   This allows state_dict() to always return ShardedTensor objects.

    # this is the sharding placement we use in DTensor to represent row-wise sharding
    # row_wise_sharding_placements means that the global tensor is sharded by first dim
    # over the 1-d mesh.
    row_wise_sharding_placements: list[Placement] = [Shard(0)]

    # create a DTensor from the local shard
    dtensor = DTensor.from_local(
        local_shards_wrapper, device_mesh, row_wise_sharding_placements, run_check=False
    )

    # display the DTensor's sharding
    visualize_sharding(dtensor, header="Row-wise even sharding example in DTensor")

    ###########################################################################
    # example 2: transform DTensor into local_shards
````

- **L161** EN: Keeps the inline comment or directive: _initialize_torch_state() and torch.Tensor params are transformed | CN: 保留这一行注释或指令：_initialize_torch_state() and torch.Tensor params are transformed
- **L162** EN: Keeps the inline comment or directive: into ShardedTensor by ShardedTensor._init_from_local_shards(). | CN: 保留这一行注释或指令：into ShardedTensor by ShardedTensor._init_from_local_shards().
- **L163** EN: Keeps the inline comment or directive: comment | CN: 保留这一行注释或指令：comment
- **L164** EN: Keeps the inline comment or directive: This allows state_dict() to always return ShardedTensor objects. | CN: 保留这一行注释或指令：This allows state_dict() to always return ShardedTensor objects.
- **L165** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L166** EN: Keeps the inline comment or directive: this is the sharding placement we use in DTensor to represent row-wise sharding | CN: 保留这一行注释或指令：this is the sharding placement we use in DTensor to represent row-wise sharding
- **L167** EN: Keeps the inline comment or directive: row_wise_sharding_placements means that the global tensor is sharded by first di | CN: 保留这一行注释或指令：row_wise_sharding_placements means that the global tensor is sharded by first di
- **L168** EN: Keeps the inline comment or directive: over the 1-d mesh. | CN: 保留这一行注释或指令：over the 1-d mesh.
- **L169** EN: Assigns or updates `row_wise_sharding_placements`. | CN: 对 `row_wise_sharding_placements` 进行赋值或更新。
- **L170** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L171** EN: Keeps the inline comment or directive: create a DTensor from the local shard | CN: 保留这一行注释或指令：create a DTensor from the local shard
- **L172** EN: Assigns or updates `dtensor`. | CN: 对 `dtensor` 进行赋值或更新。
- **L173** EN: Assigns or updates `local_shards_wrapper, device_mesh, row_wise_sharding_placements, run_check`. | CN: 对 `local_shards_wrapper, device_mesh, row_wise_sharding_placements, run_check` 进行赋值或更新。
- **L174** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L175** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L176** EN: Keeps the inline comment or directive: display the DTensor's sharding | CN: 保留这一行注释或指令：display the DTensor's sharding
- **L177** EN: Calls `visualize_sharding` as part of the current workflow. | CN: 在当前流程中调用 `visualize_sharding`。
- **L178** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L179** EN: Keeps the inline comment or directive: ########################################################################## | CN: 保留这一行注释或指令：##########################################################################
- **L180** EN: Keeps the inline comment or directive: example 2: transform DTensor into local_shards | CN: 保留这一行注释或指令：example 2: transform DTensor into local_shards

### Lines 181-200 / 第 181-200 行

````python
    # usage in TorchRec:
    #   In ShardedEmbeddingCollection's load_state_dict pre hook
    #   _pre_load_state_dict_hook, if the source param is a ShardedTensor
    #   then we need to transform it into its local_shards.

    # transform DTensor into LocalShardsWrapper
    dtensor_local_shards = dtensor.to_local()
    if not isinstance(dtensor_local_shards, LocalShardsWrapper):
        raise AssertionError
    shard_tensor = dtensor_local_shards.shards[0]
    if not torch.equal(shard_tensor, local_tensor):
        raise AssertionError
    if dtensor_local_shards.shard_sizes[0] != local_shard_shape:  # unwrap shape
        raise AssertionError
    if dtensor_local_shards.shard_offsets[0] != local_shard_offset:  # unwrap offset
        raise AssertionError


def run_torchrec_row_wise_uneven_sharding_example(rank, world_size):
    # row-wise uneven sharding example:
````

- **L181** EN: Keeps the inline comment or directive: usage in TorchRec: | CN: 保留这一行注释或指令：usage in TorchRec:
- **L182** EN: Keeps the inline comment or directive: In ShardedEmbeddingCollection's load_state_dict pre hook | CN: 保留这一行注释或指令：In ShardedEmbeddingCollection's load_state_dict pre hook
- **L183** EN: Keeps the inline comment or directive: _pre_load_state_dict_hook, if the source param is a ShardedTensor | CN: 保留这一行注释或指令：_pre_load_state_dict_hook, if the source param is a ShardedTensor
- **L184** EN: Keeps the inline comment or directive: then we need to transform it into its local_shards. | CN: 保留这一行注释或指令：then we need to transform it into its local_shards.
- **L185** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L186** EN: Keeps the inline comment or directive: transform DTensor into LocalShardsWrapper | CN: 保留这一行注释或指令：transform DTensor into LocalShardsWrapper
- **L187** EN: Assigns or updates `dtensor_local_shards`. | CN: 对 `dtensor_local_shards` 进行赋值或更新。
- **L188** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L189** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L190** EN: Assigns or updates `shard_tensor`. | CN: 对 `shard_tensor` 进行赋值或更新。
- **L191** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L192** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L193** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L194** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L195** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L196** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L197** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L198** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L199** EN: Defines function `run_torchrec_row_wise_uneven_sharding_example`. | CN: 定义函数 `run_torchrec_row_wise_uneven_sharding_example`。
- **L200** EN: Keeps the inline comment or directive: row-wise uneven sharding example: | CN: 保留这一行注释或指令：row-wise uneven sharding example:

### Lines 201-220 / 第 201-220 行

````python
    #   One table is unevenly sharded by rows within the global ProcessGroup.
    #   In our example, the table's num_embedding is 8, and the embedding dim is 16
    #   The global ProcessGroup has 4 ranks, and each rank will have the local shard
    #   of shape:
    #       rank 0: [1, 16]
    #       rank 1: [3, 16]
    #       rank 2: [1, 16]
    #       rank 3: [3, 16]

    # device mesh is a representation of the worker ranks
    # create a 1-D device mesh that includes every rank
    device_type = get_device_type()
    device = torch.device(device_type)
    device_mesh = init_device_mesh(device_type=device_type, mesh_shape=(world_size,))

    # manually create the embedding table's local shards
    num_embeddings = 8
    embedding_dim = 16
    emb_table_shape = torch.Size([num_embeddings, embedding_dim])
    # tensor shape
````

- **L201** EN: Keeps the inline comment or directive: One table is unevenly sharded by rows within the global ProcessGroup. | CN: 保留这一行注释或指令：One table is unevenly sharded by rows within the global ProcessGroup.
- **L202** EN: Keeps the inline comment or directive: In our example, the table's num_embedding is 8, and the embedding dim is 16 | CN: 保留这一行注释或指令：In our example, the table's num_embedding is 8, and the embedding dim is 16
- **L203** EN: Keeps the inline comment or directive: The global ProcessGroup has 4 ranks, and each rank will have the local shard | CN: 保留这一行注释或指令：The global ProcessGroup has 4 ranks, and each rank will have the local shard
- **L204** EN: Keeps the inline comment or directive: of shape: | CN: 保留这一行注释或指令：of shape:
- **L205** EN: Keeps the inline comment or directive: rank 0: [1, 16] | CN: 保留这一行注释或指令：rank 0: [1, 16]
- **L206** EN: Keeps the inline comment or directive: rank 1: [3, 16] | CN: 保留这一行注释或指令：rank 1: [3, 16]
- **L207** EN: Keeps the inline comment or directive: rank 2: [1, 16] | CN: 保留这一行注释或指令：rank 2: [1, 16]
- **L208** EN: Keeps the inline comment or directive: rank 3: [3, 16] | CN: 保留这一行注释或指令：rank 3: [3, 16]
- **L209** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L210** EN: Keeps the inline comment or directive: device mesh is a representation of the worker ranks | CN: 保留这一行注释或指令：device mesh is a representation of the worker ranks
- **L211** EN: Keeps the inline comment or directive: create a 1-D device mesh that includes every rank | CN: 保留这一行注释或指令：create a 1-D device mesh that includes every rank
- **L212** EN: Assigns or updates `device_type`. | CN: 对 `device_type` 进行赋值或更新。
- **L213** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L214** EN: Assigns or updates `device_mesh`. | CN: 对 `device_mesh` 进行赋值或更新。
- **L215** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L216** EN: Keeps the inline comment or directive: manually create the embedding table's local shards | CN: 保留这一行注释或指令：manually create the embedding table's local shards
- **L217** EN: Assigns or updates `num_embeddings`. | CN: 对 `num_embeddings` 进行赋值或更新。
- **L218** EN: Assigns or updates `embedding_dim`. | CN: 对 `embedding_dim` 进行赋值或更新。
- **L219** EN: Assigns or updates `emb_table_shape`. | CN: 对 `emb_table_shape` 进行赋值或更新。
- **L220** EN: Keeps the inline comment or directive: tensor shape | CN: 保留这一行注释或指令：tensor shape

### Lines 221-240 / 第 221-240 行

````python
    local_shard_shape = (
        torch.Size([1, embedding_dim])
        if rank % 2 == 0
        else torch.Size([3, embedding_dim])
    )
    # tensor offset
    local_shard_offset = torch.Size((rank // 2 * 4 + rank % 2 * 1, embedding_dim))
    # tensor
    local_tensor = torch.randn(local_shard_shape, device=device)
    # local shards
    # row-wise sharding: one shard per rank
    # create the local shards wrapper
    # pyrefly: ignore [no-matching-overload]
    local_shards_wrapper = LocalShardsWrapper(
        local_shards=[local_tensor],
        offsets=[local_shard_offset],
    )

    ###########################################################################
    # example 1: transform local_shards into DTensor
````

- **L221** EN: Assigns or updates `local_shard_shape`. | CN: 对 `local_shard_shape` 进行赋值或更新。
- **L222** EN: Calls `torch.Size` as part of the current workflow. | CN: 在当前流程中调用 `torch.Size`。
- **L223** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L224** EN: Continues the implementation inside function `run_torchrec_row_wise_uneven_sharding_example`. | CN: 继续说明函数 `run_torchrec_row_wise_uneven_sharding_example` 内部的实现。
- **L225** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L226** EN: Keeps the inline comment or directive: tensor offset | CN: 保留这一行注释或指令：tensor offset
- **L227** EN: Assigns or updates `local_shard_offset`. | CN: 对 `local_shard_offset` 进行赋值或更新。
- **L228** EN: Keeps the inline comment or directive: tensor | CN: 保留这一行注释或指令：tensor
- **L229** EN: Assigns or updates `local_tensor`. | CN: 对 `local_tensor` 进行赋值或更新。
- **L230** EN: Keeps the inline comment or directive: local shards | CN: 保留这一行注释或指令：local shards
- **L231** EN: Keeps the inline comment or directive: row-wise sharding: one shard per rank | CN: 保留这一行注释或指令：row-wise sharding: one shard per rank
- **L232** EN: Keeps the inline comment or directive: create the local shards wrapper | CN: 保留这一行注释或指令：create the local shards wrapper
- **L233** EN: Keeps the inline comment or directive: pyrefly: ignore [no-matching-overload] | CN: 保留这一行注释或指令：pyrefly: ignore [no-matching-overload]
- **L234** EN: Assigns or updates `local_shards_wrapper`. | CN: 对 `local_shards_wrapper` 进行赋值或更新。
- **L235** EN: Assigns or updates `local_shards`. | CN: 对 `local_shards` 进行赋值或更新。
- **L236** EN: Assigns or updates `offsets`. | CN: 对 `offsets` 进行赋值或更新。
- **L237** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L238** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L239** EN: Keeps the inline comment or directive: ########################################################################## | CN: 保留这一行注释或指令：##########################################################################
- **L240** EN: Keeps the inline comment or directive: example 1: transform local_shards into DTensor | CN: 保留这一行注释或指令：example 1: transform local_shards into DTensor

### Lines 241-260 / 第 241-260 行

````python
    # create the DTensorMetadata which torchrec should provide
    row_wise_sharding_placements: list[Placement] = [Shard(0)]

    # note: for uneven sharding, we need to specify the shape and stride because
    # DTensor would assume even sharding and compute shape/stride based on the
    # assumption. Torchrec needs to pass in this information explicitly.
    # shape/stride are global tensor's shape and stride
    dtensor = DTensor.from_local(
        local_shards_wrapper,  # a torch.Tensor subclass
        device_mesh,  # DeviceMesh
        row_wise_sharding_placements,  # List[Placement]
        run_check=False,
        shape=emb_table_shape,  # this is required for uneven sharding
        stride=(embedding_dim, 1),
    )
    # so far visualize_sharding() cannot print correctly for unevenly sharded DTensor
    # because it relies on offset computation which assumes even sharding.
    visualize_sharding(dtensor, header="Row-wise uneven sharding example in DTensor")
    # check the dtensor has the correct shape and stride on all ranks
    if dtensor.shape != emb_table_shape:
````

- **L241** EN: Keeps the inline comment or directive: create the DTensorMetadata which torchrec should provide | CN: 保留这一行注释或指令：create the DTensorMetadata which torchrec should provide
- **L242** EN: Assigns or updates `row_wise_sharding_placements`. | CN: 对 `row_wise_sharding_placements` 进行赋值或更新。
- **L243** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L244** EN: Keeps the inline comment or directive: note: for uneven sharding, we need to specify the shape and stride because | CN: 保留这一行注释或指令：note: for uneven sharding, we need to specify the shape and stride because
- **L245** EN: Keeps the inline comment or directive: DTensor would assume even sharding and compute shape/stride based on the | CN: 保留这一行注释或指令：DTensor would assume even sharding and compute shape/stride based on the
- **L246** EN: Keeps the inline comment or directive: assumption. Torchrec needs to pass in this information explicitly. | CN: 保留这一行注释或指令：assumption. Torchrec needs to pass in this information explicitly.
- **L247** EN: Keeps the inline comment or directive: shape/stride are global tensor's shape and stride | CN: 保留这一行注释或指令：shape/stride are global tensor's shape and stride
- **L248** EN: Assigns or updates `dtensor`. | CN: 对 `dtensor` 进行赋值或更新。
- **L249** EN: Continues the implementation inside function `run_torchrec_row_wise_uneven_sharding_example`. | CN: 继续说明函数 `run_torchrec_row_wise_uneven_sharding_example` 内部的实现。
- **L250** EN: Continues the implementation inside function `run_torchrec_row_wise_uneven_sharding_example`. | CN: 继续说明函数 `run_torchrec_row_wise_uneven_sharding_example` 内部的实现。
- **L251** EN: Continues the implementation inside function `run_torchrec_row_wise_uneven_sharding_example`. | CN: 继续说明函数 `run_torchrec_row_wise_uneven_sharding_example` 内部的实现。
- **L252** EN: Assigns or updates `run_check`. | CN: 对 `run_check` 进行赋值或更新。
- **L253** EN: Assigns or updates `shape`. | CN: 对 `shape` 进行赋值或更新。
- **L254** EN: Assigns or updates `stride`. | CN: 对 `stride` 进行赋值或更新。
- **L255** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L256** EN: Keeps the inline comment or directive: so far visualize_sharding() cannot print correctly for unevenly sharded DTensor | CN: 保留这一行注释或指令：so far visualize_sharding() cannot print correctly for unevenly sharded DTensor
- **L257** EN: Keeps the inline comment or directive: because it relies on offset computation which assumes even sharding. | CN: 保留这一行注释或指令：because it relies on offset computation which assumes even sharding.
- **L258** EN: Calls `visualize_sharding` as part of the current workflow. | CN: 在当前流程中调用 `visualize_sharding`。
- **L259** EN: Keeps the inline comment or directive: check the dtensor has the correct shape and stride on all ranks | CN: 保留这一行注释或指令：check the dtensor has the correct shape and stride on all ranks
- **L260** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 261-280 / 第 261-280 行

````python
        raise AssertionError
    if dtensor.stride() != (embedding_dim, 1):
        raise AssertionError

    ###########################################################################
    # example 2: transform DTensor into local_shards
    # note: DTensor.to_local() always returns a LocalShardsWrapper
    dtensor_local_shards = dtensor.to_local()
    if not isinstance(dtensor_local_shards, LocalShardsWrapper):
        raise AssertionError
    shard_tensor = dtensor_local_shards.shards[0]
    if not torch.equal(shard_tensor, local_tensor):
        raise AssertionError
    if dtensor_local_shards.shard_sizes[0] != local_shard_shape:  # unwrap shape
        raise AssertionError
    if dtensor_local_shards.shard_offsets[0] != local_shard_offset:  # unwrap offset
        raise AssertionError


def run_torchrec_table_wise_sharding_example(rank, world_size):
````

- **L261** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L262** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L263** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L264** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L265** EN: Keeps the inline comment or directive: ########################################################################## | CN: 保留这一行注释或指令：##########################################################################
- **L266** EN: Keeps the inline comment or directive: example 2: transform DTensor into local_shards | CN: 保留这一行注释或指令：example 2: transform DTensor into local_shards
- **L267** EN: Keeps the inline comment or directive: note: DTensor.to_local() always returns a LocalShardsWrapper | CN: 保留这一行注释或指令：note: DTensor.to_local() always returns a LocalShardsWrapper
- **L268** EN: Assigns or updates `dtensor_local_shards`. | CN: 对 `dtensor_local_shards` 进行赋值或更新。
- **L269** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L270** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L271** EN: Assigns or updates `shard_tensor`. | CN: 对 `shard_tensor` 进行赋值或更新。
- **L272** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L273** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L274** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L275** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L276** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L277** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L278** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L279** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L280** EN: Defines function `run_torchrec_table_wise_sharding_example`. | CN: 定义函数 `run_torchrec_table_wise_sharding_example`。

### Lines 281-300 / 第 281-300 行

````python
    # table-wise example:
    #   each rank in the global ProcessGroup holds one different table.
    #   In our example, the table's num_embedding is 8, and the embedding dim is 16
    #   The global ProcessGroup has 4 ranks, so each rank will have one 8 by 16 complete
    #   table as its local shard.

    device_type = get_device_type()
    device = torch.device(device_type)
    # note: without initializing this mesh, the following local_tensor will be put on
    # device cuda:0.
    init_device_mesh(device_type=device_type, mesh_shape=(world_size,))

    # manually create the embedding table's local shards
    num_embeddings = 8
    embedding_dim = 16
    emb_table_shape = torch.Size([num_embeddings, embedding_dim])

    # for table i, if the current rank holds the table, then the local shard is
    # a LocalShardsWrapper containing the tensor; otherwise the local shard is
    # an empty torch.Tensor
````

- **L281** EN: Keeps the inline comment or directive: table-wise example: | CN: 保留这一行注释或指令：table-wise example:
- **L282** EN: Keeps the inline comment or directive: each rank in the global ProcessGroup holds one different table. | CN: 保留这一行注释或指令：each rank in the global ProcessGroup holds one different table.
- **L283** EN: Keeps the inline comment or directive: In our example, the table's num_embedding is 8, and the embedding dim is 16 | CN: 保留这一行注释或指令：In our example, the table's num_embedding is 8, and the embedding dim is 16
- **L284** EN: Keeps the inline comment or directive: The global ProcessGroup has 4 ranks, so each rank will have one 8 by 16 complete | CN: 保留这一行注释或指令：The global ProcessGroup has 4 ranks, so each rank will have one 8 by 16 complete
- **L285** EN: Keeps the inline comment or directive: table as its local shard. | CN: 保留这一行注释或指令：table as its local shard.
- **L286** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L287** EN: Assigns or updates `device_type`. | CN: 对 `device_type` 进行赋值或更新。
- **L288** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L289** EN: Keeps the inline comment or directive: note: without initializing this mesh, the following local_tensor will be put on | CN: 保留这一行注释或指令：note: without initializing this mesh, the following local_tensor will be put on
- **L290** EN: Keeps the inline comment or directive: device cuda:0. | CN: 保留这一行注释或指令：device cuda:0.
- **L291** EN: Calls `init_device_mesh` as part of the current workflow. | CN: 在当前流程中调用 `init_device_mesh`。
- **L292** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L293** EN: Keeps the inline comment or directive: manually create the embedding table's local shards | CN: 保留这一行注释或指令：manually create the embedding table's local shards
- **L294** EN: Assigns or updates `num_embeddings`. | CN: 对 `num_embeddings` 进行赋值或更新。
- **L295** EN: Assigns or updates `embedding_dim`. | CN: 对 `embedding_dim` 进行赋值或更新。
- **L296** EN: Assigns or updates `emb_table_shape`. | CN: 对 `emb_table_shape` 进行赋值或更新。
- **L297** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L298** EN: Keeps the inline comment or directive: for table i, if the current rank holds the table, then the local shard is | CN: 保留这一行注释或指令：for table i, if the current rank holds the table, then the local shard is
- **L299** EN: Keeps the inline comment or directive: a LocalShardsWrapper containing the tensor; otherwise the local shard is | CN: 保留这一行注释或指令：a LocalShardsWrapper containing the tensor; otherwise the local shard is
- **L300** EN: Keeps the inline comment or directive: an empty torch.Tensor | CN: 保留这一行注释或指令：an empty torch.Tensor

### Lines 301-320 / 第 301-320 行

````python
    table_to_shards = {}  # map {table_id: local shard of table_id}
    table_to_local_tensor = {}  # map {table_id: local tensor of table_id}
    # create 4 embedding tables and place them on different ranks
    # each rank will hold one complete table, and the dict will store
    # the corresponding local shard.
    for i in range(world_size):
        # tensor
        local_tensor = (
            torch.randn(*emb_table_shape, device=device)
            if rank == i
            else torch.empty(0, device=device)
        )
        table_to_local_tensor[i] = local_tensor
        # tensor offset
        local_shard_offset = torch.Size((0, 0))
        # wrap local shards into a wrapper
        local_shards_wrapper = (
            # pyrefly: ignore [no-matching-overload]
            LocalShardsWrapper(
                local_shards=[local_tensor],
````

- **L301** EN: Assigns or updates `table_to_shards`. | CN: 对 `table_to_shards` 进行赋值或更新。
- **L302** EN: Assigns or updates `table_to_local_tensor`. | CN: 对 `table_to_local_tensor` 进行赋值或更新。
- **L303** EN: Keeps the inline comment or directive: create 4 embedding tables and place them on different ranks | CN: 保留这一行注释或指令：create 4 embedding tables and place them on different ranks
- **L304** EN: Keeps the inline comment or directive: each rank will hold one complete table, and the dict will store | CN: 保留这一行注释或指令：each rank will hold one complete table, and the dict will store
- **L305** EN: Keeps the inline comment or directive: the corresponding local shard. | CN: 保留这一行注释或指令：the corresponding local shard.
- **L306** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L307** EN: Keeps the inline comment or directive: tensor | CN: 保留这一行注释或指令：tensor
- **L308** EN: Assigns or updates `local_tensor`. | CN: 对 `local_tensor` 进行赋值或更新。
- **L309** EN: Calls `torch.randn` as part of the current workflow. | CN: 在当前流程中调用 `torch.randn`。
- **L310** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L311** EN: Continues the implementation inside function `run_torchrec_table_wise_sharding_example`. | CN: 继续说明函数 `run_torchrec_table_wise_sharding_example` 内部的实现。
- **L312** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L313** EN: Assigns or updates `table_to_local_tensor[i]`. | CN: 对 `table_to_local_tensor[i]` 进行赋值或更新。
- **L314** EN: Keeps the inline comment or directive: tensor offset | CN: 保留这一行注释或指令：tensor offset
- **L315** EN: Assigns or updates `local_shard_offset`. | CN: 对 `local_shard_offset` 进行赋值或更新。
- **L316** EN: Keeps the inline comment or directive: wrap local shards into a wrapper | CN: 保留这一行注释或指令：wrap local shards into a wrapper
- **L317** EN: Assigns or updates `local_shards_wrapper`. | CN: 对 `local_shards_wrapper` 进行赋值或更新。
- **L318** EN: Keeps the inline comment or directive: pyrefly: ignore [no-matching-overload] | CN: 保留这一行注释或指令：pyrefly: ignore [no-matching-overload]
- **L319** EN: Calls `LocalShardsWrapper` as part of the current workflow. | CN: 在当前流程中调用 `LocalShardsWrapper`。
- **L320** EN: Assigns or updates `local_shards`. | CN: 对 `local_shards` 进行赋值或更新。

### Lines 321-340 / 第 321-340 行

````python
                offsets=[local_shard_offset],
            )
            if rank == i
            else local_tensor
        )
        table_to_shards[i] = local_shards_wrapper

    ###########################################################################
    # example 1: transform local_shards into DTensor
    table_to_dtensor = {}  # same purpose as _model_parallel_name_to_sharded_tensor
    table_wise_sharding_placements = [Replicate()]  # table-wise sharding

    for table_id, local_shards in table_to_shards.items():
        # create a submesh that only contains the rank we place the table
        # note that we cannot use ``init_device_mesh'' to create a submesh
        # so we choose to use the `DeviceMesh` api to directly create a DeviceMesh
        device_submesh = DeviceMesh(
            device_type=device_type,
            mesh=torch.tensor(
                [table_id], dtype=torch.int64
````

- **L321** EN: Assigns or updates `offsets`. | CN: 对 `offsets` 进行赋值或更新。
- **L322** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L323** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L324** EN: Continues the implementation inside function `run_torchrec_table_wise_sharding_example`. | CN: 继续说明函数 `run_torchrec_table_wise_sharding_example` 内部的实现。
- **L325** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L326** EN: Assigns or updates `table_to_shards[i]`. | CN: 对 `table_to_shards[i]` 进行赋值或更新。
- **L327** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L328** EN: Keeps the inline comment or directive: ########################################################################## | CN: 保留这一行注释或指令：##########################################################################
- **L329** EN: Keeps the inline comment or directive: example 1: transform local_shards into DTensor | CN: 保留这一行注释或指令：example 1: transform local_shards into DTensor
- **L330** EN: Assigns or updates `table_to_dtensor`. | CN: 对 `table_to_dtensor` 进行赋值或更新。
- **L331** EN: Assigns or updates `table_wise_sharding_placements`. | CN: 对 `table_wise_sharding_placements` 进行赋值或更新。
- **L332** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L333** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L334** EN: Keeps the inline comment or directive: create a submesh that only contains the rank we place the table | CN: 保留这一行注释或指令：create a submesh that only contains the rank we place the table
- **L335** EN: Keeps the inline comment or directive: note that we cannot use ``init_device_mesh'' to create a submesh | CN: 保留这一行注释或指令：note that we cannot use ``init_device_mesh'' to create a submesh
- **L336** EN: Keeps the inline comment or directive: so we choose to use the `DeviceMesh` api to directly create a DeviceMesh | CN: 保留这一行注释或指令：so we choose to use the `DeviceMesh` api to directly create a DeviceMesh
- **L337** EN: Assigns or updates `device_submesh`. | CN: 对 `device_submesh` 进行赋值或更新。
- **L338** EN: Assigns or updates `device_type`. | CN: 对 `device_type` 进行赋值或更新。
- **L339** EN: Assigns or updates `mesh`. | CN: 对 `mesh` 进行赋值或更新。
- **L340** EN: Continues the implementation inside function `run_torchrec_table_wise_sharding_example`. | CN: 继续说明函数 `run_torchrec_table_wise_sharding_example` 内部的实现。

### Lines 341-360 / 第 341-360 行

````python
            ),  # table ``table_id`` is placed on rank ``table_id``
        )
        # create a DTensor from the local shard for the current table
        # note: for uneven sharding, we need to specify the shape and stride because
        # DTensor would assume even sharding and compute shape/stride based on the
        # assumption. Torchrec needs to pass in this information explicitly.
        dtensor = DTensor.from_local(
            local_shards,
            device_submesh,
            table_wise_sharding_placements,
            run_check=False,
            shape=emb_table_shape,  # this is required for uneven sharding
            stride=(embedding_dim, 1),
        )
        table_to_dtensor[table_id] = dtensor

    # print each table's sharding
    for table_id, dtensor in table_to_dtensor.items():
        visualize_sharding(
            dtensor,
````

- **L341** EN: Continues the implementation inside function `run_torchrec_table_wise_sharding_example`. | CN: 继续说明函数 `run_torchrec_table_wise_sharding_example` 内部的实现。
- **L342** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L343** EN: Keeps the inline comment or directive: create a DTensor from the local shard for the current table | CN: 保留这一行注释或指令：create a DTensor from the local shard for the current table
- **L344** EN: Keeps the inline comment or directive: note: for uneven sharding, we need to specify the shape and stride because | CN: 保留这一行注释或指令：note: for uneven sharding, we need to specify the shape and stride because
- **L345** EN: Keeps the inline comment or directive: DTensor would assume even sharding and compute shape/stride based on the | CN: 保留这一行注释或指令：DTensor would assume even sharding and compute shape/stride based on the
- **L346** EN: Keeps the inline comment or directive: assumption. Torchrec needs to pass in this information explicitly. | CN: 保留这一行注释或指令：assumption. Torchrec needs to pass in this information explicitly.
- **L347** EN: Assigns or updates `dtensor`. | CN: 对 `dtensor` 进行赋值或更新。
- **L348** EN: Continues the implementation inside function `run_torchrec_table_wise_sharding_example`. | CN: 继续说明函数 `run_torchrec_table_wise_sharding_example` 内部的实现。
- **L349** EN: Continues the implementation inside function `run_torchrec_table_wise_sharding_example`. | CN: 继续说明函数 `run_torchrec_table_wise_sharding_example` 内部的实现。
- **L350** EN: Continues the implementation inside function `run_torchrec_table_wise_sharding_example`. | CN: 继续说明函数 `run_torchrec_table_wise_sharding_example` 内部的实现。
- **L351** EN: Assigns or updates `run_check`. | CN: 对 `run_check` 进行赋值或更新。
- **L352** EN: Assigns or updates `shape`. | CN: 对 `shape` 进行赋值或更新。
- **L353** EN: Assigns or updates `stride`. | CN: 对 `stride` 进行赋值或更新。
- **L354** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L355** EN: Assigns or updates `table_to_dtensor[table_id]`. | CN: 对 `table_to_dtensor[table_id]` 进行赋值或更新。
- **L356** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L357** EN: Keeps the inline comment or directive: print each table's sharding | CN: 保留这一行注释或指令：print each table's sharding
- **L358** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L359** EN: Calls `visualize_sharding` as part of the current workflow. | CN: 在当前流程中调用 `visualize_sharding`。
- **L360** EN: Continues the implementation inside function `run_torchrec_table_wise_sharding_example`. | CN: 继续说明函数 `run_torchrec_table_wise_sharding_example` 内部的实现。

### Lines 361-380 / 第 361-380 行

````python
            header=f"Table-wise sharding example in DTensor for Table {table_id}",
        )
        # check the dtensor has the correct shape and stride on all ranks
        if dtensor.shape != emb_table_shape:
            raise AssertionError
        if dtensor.stride() != (embedding_dim, 1):
            raise AssertionError

    ###########################################################################
    # example 2: transform DTensor into torch.Tensor
    for table_id, local_tensor in table_to_local_tensor.items():
        # important: note that DTensor.to_local() always returns an empty torch.Tensor
        # no matter what was passed to DTensor._local_tensor.
        dtensor_local_shards = table_to_dtensor[table_id].to_local()
        if rank == table_id:
            if not isinstance(dtensor_local_shards, LocalShardsWrapper):
                raise AssertionError
            shard_tensor = dtensor_local_shards.shards[0]
            if not torch.equal(shard_tensor, local_tensor):  # unwrap tensor
                raise AssertionError
````

- **L361** EN: Assigns or updates `header`. | CN: 对 `header` 进行赋值或更新。
- **L362** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L363** EN: Keeps the inline comment or directive: check the dtensor has the correct shape and stride on all ranks | CN: 保留这一行注释或指令：check the dtensor has the correct shape and stride on all ranks
- **L364** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L365** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L366** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L367** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L368** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L369** EN: Keeps the inline comment or directive: ########################################################################## | CN: 保留这一行注释或指令：##########################################################################
- **L370** EN: Keeps the inline comment or directive: example 2: transform DTensor into torch.Tensor | CN: 保留这一行注释或指令：example 2: transform DTensor into torch.Tensor
- **L371** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L372** EN: Keeps the inline comment or directive: important: note that DTensor.to_local() always returns an empty torch.Tensor | CN: 保留这一行注释或指令：important: note that DTensor.to_local() always returns an empty torch.Tensor
- **L373** EN: Keeps the inline comment or directive: no matter what was passed to DTensor._local_tensor. | CN: 保留这一行注释或指令：no matter what was passed to DTensor._local_tensor.
- **L374** EN: Assigns or updates `dtensor_local_shards`. | CN: 对 `dtensor_local_shards` 进行赋值或更新。
- **L375** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L376** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L377** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L378** EN: Assigns or updates `shard_tensor`. | CN: 对 `shard_tensor` 进行赋值或更新。
- **L379** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L380** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。

### Lines 381-400 / 第 381-400 行

````python
            if dtensor_local_shards.shard_sizes[0] != emb_table_shape:  # unwrap shape
                raise AssertionError
            if dtensor_local_shards.shard_offsets[0] != torch.Size(
                (0, 0)
            ):  # unwrap offset
                raise AssertionError
        else:
            if dtensor_local_shards.numel() != 0:
                raise AssertionError


def run_example(rank, world_size, example_name):
    # the dict that stores example code
    name_to_example_code = {
        "row-wise-even": run_torchrec_row_wise_even_sharding_example,
        "row-wise-uneven": run_torchrec_row_wise_uneven_sharding_example,
        "table-wise": run_torchrec_table_wise_sharding_example,
    }
    if example_name not in name_to_example_code:
        print(f"example for {example_name} does not exist!")
````

- **L381** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L382** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L383** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L384** EN: Continues the implementation inside function `run_torchrec_table_wise_sharding_example`. | CN: 继续说明函数 `run_torchrec_table_wise_sharding_example` 内部的实现。
- **L385** EN: Continues the implementation inside function `run_torchrec_table_wise_sharding_example`. | CN: 继续说明函数 `run_torchrec_table_wise_sharding_example` 内部的实现。
- **L386** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L387** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L388** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L389** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L390** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L391** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L392** EN: Defines function `run_example`. | CN: 定义函数 `run_example`。
- **L393** EN: Keeps the inline comment or directive: the dict that stores example code | CN: 保留这一行注释或指令：the dict that stores example code
- **L394** EN: Assigns or updates `name_to_example_code`. | CN: 对 `name_to_example_code` 进行赋值或更新。
- **L395** EN: Continues the implementation inside function `run_example`. | CN: 继续说明函数 `run_example` 内部的实现。
- **L396** EN: Continues the implementation inside function `run_example`. | CN: 继续说明函数 `run_example` 内部的实现。
- **L397** EN: Continues the implementation inside function `run_example`. | CN: 继续说明函数 `run_example` 内部的实现。
- **L398** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L399** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L400** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。

### Lines 401-420 / 第 401-420 行

````python
        return

    # the example to run
    example_func = name_to_example_code[example_name]

    # set manual seed
    torch.manual_seed(0)

    # run the example
    example_func(rank, world_size)


if __name__ == "__main__":
    # this script is launched via torchrun which automatically manages ProcessGroup
    rank = int(os.environ["RANK"])
    world_size = int(os.environ["WORLD_SIZE"])
    if world_size != 4:  # our example uses 4 worker ranks
        raise AssertionError
    # parse the arguments
    parser = argparse.ArgumentParser(
````

- **L401** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L402** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L403** EN: Keeps the inline comment or directive: the example to run | CN: 保留这一行注释或指令：the example to run
- **L404** EN: Assigns or updates `example_func`. | CN: 对 `example_func` 进行赋值或更新。
- **L405** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L406** EN: Keeps the inline comment or directive: set manual seed | CN: 保留这一行注释或指令：set manual seed
- **L407** EN: Calls `torch.manual_seed` as part of the current workflow. | CN: 在当前流程中调用 `torch.manual_seed`。
- **L408** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L409** EN: Keeps the inline comment or directive: run the example | CN: 保留这一行注释或指令：run the example
- **L410** EN: Calls `example_func` as part of the current workflow. | CN: 在当前流程中调用 `example_func`。
- **L411** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L412** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L413** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L414** EN: Keeps the inline comment or directive: this script is launched via torchrun which automatically manages ProcessGroup | CN: 保留这一行注释或指令：this script is launched via torchrun which automatically manages ProcessGroup
- **L415** EN: Assigns or updates `rank`. | CN: 对 `rank` 进行赋值或更新。
- **L416** EN: Assigns or updates `world_size`. | CN: 对 `world_size` 进行赋值或更新。
- **L417** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L418** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L419** EN: Keeps the inline comment or directive: parse the arguments | CN: 保留这一行注释或指令：parse the arguments
- **L420** EN: Assigns or updates `parser`. | CN: 对 `parser` 进行赋值或更新。

### Lines 421-433 / 第 421-433 行

````python
        description="torchrec sharding examples",
        formatter_class=argparse.RawTextHelpFormatter,
    )
    example_prompt = (
        "choose one sharding example from below:\n"
        "\t1. row-wise-even;\n"
        "\t2. row-wise-uneven\n"
        "\t3. table-wise\n"
        "e.g. you want to try the row-wise even sharding example, please input 'row-wise-even'\n"
    )
    parser.add_argument("-e", "--example", help=example_prompt, required=True)
    args = parser.parse_args()
    run_example(rank, world_size, args.example)
````

- **L421** EN: Assigns or updates `description`. | CN: 对 `description` 进行赋值或更新。
- **L422** EN: Assigns or updates `formatter_class`. | CN: 对 `formatter_class` 进行赋值或更新。
- **L423** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L424** EN: Assigns or updates `example_prompt`. | CN: 对 `example_prompt` 进行赋值或更新。
- **L425** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L426** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L427** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L428** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L429** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L430** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L431** EN: Calls `parser.add_argument` as part of the current workflow. | CN: 在当前流程中调用 `parser.add_argument`。
- **L432** EN: Assigns or updates `args`. | CN: 对 `args` 进行赋值或更新。
- **L433** EN: Calls `run_example` as part of the current workflow. | CN: 在当前流程中调用 `run_example`。

## Key Concepts / 关键概念

- **EN**: distributed tensor layouts, placements, and operators  
  **CN**: 分布式张量布局、放置与算子逻辑
- **EN**: device mesh  
  **CN**: 设备网格
- **EN**: sharding  
  **CN**: 分片
- **EN**: checkpointing  
  **CN**: 检查点
- **EN**: distributed tensors  
  **CN**: 分布式张量
- **EN**: placements  
  **CN**: 放置规则
- **EN**: distributed stores  
  **CN**: 分布式存储
- **EN**: communication hooks  
  **CN**: 通信钩子

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed.checkpoint.metadata`, `torch.distributed.tensor`, `torch.distributed.tensor.debug`, `torch.distributed.tensor.placement_types`
- **PyTorch / PyTorch**: `torch`
- **Python Stdlib / Python 标准库**: `argparse`, `functools`, `os`, `typing`
- **Third-party / 第三方**: None detected / 未检测到

