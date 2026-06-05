# api.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/_shard/sharded_tensor/api.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on sharding specifications and shard-aware tensor helpers. Its main entry points include ShardedTensorBase, ShardedTensor, _register_remote_shards, _create_tensor_from_params.
- **用途 (CN)**: 该模块聚焦于分片规范与分片张量辅助逻辑，其主要入口包括 ShardedTensorBase, ShardedTensor, _register_remote_shards, _create_tensor_from_params。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs
from __future__ import annotations  # type: ignore[attr-defined]

import copy
import operator
import threading
import warnings
import weakref
from dataclasses import dataclass
from functools import reduce
from typing import cast, TYPE_CHECKING
from typing_extensions import deprecated

import torch
import torch.distributed as dist
import torch.distributed._shard.sharding_spec as shard_spec
from torch._utils import _get_device_module
from torch.distributed import distributed_c10d, rpc
from torch.distributed._shard._utils import DEPRECATE_MSG
from torch.distributed._shard.sharding_spec._internals import (
````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Imports selected names from `__future__`. | CN: 从 `__future__` 导入指定名称。
- **L3** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L4** EN: Imports module dependencies: `copy`. | CN: 导入模块依赖：`copy`。
- **L5** EN: Imports module dependencies: `operator`. | CN: 导入模块依赖：`operator`。
- **L6** EN: Imports module dependencies: `threading`. | CN: 导入模块依赖：`threading`。
- **L7** EN: Imports module dependencies: `warnings`. | CN: 导入模块依赖：`warnings`。
- **L8** EN: Imports module dependencies: `weakref`. | CN: 导入模块依赖：`weakref`。
- **L9** EN: Imports selected names from `dataclasses`. | CN: 从 `dataclasses` 导入指定名称。
- **L10** EN: Imports selected names from `functools`. | CN: 从 `functools` 导入指定名称。
- **L11** EN: Imports selected names from `typing`. | CN: 从 `typing` 导入指定名称。
- **L12** EN: Imports selected names from `typing_extensions`. | CN: 从 `typing_extensions` 导入指定名称。
- **L13** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L14** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L15** EN: Imports module dependencies: `torch.distributed as dist`. | CN: 导入模块依赖：`torch.distributed as dist`。
- **L16** EN: Imports module dependencies: `torch.distributed._shard.sharding_spec as shard_spec`. | CN: 导入模块依赖：`torch.distributed._shard.sharding_spec as shard_spec`。
- **L17** EN: Imports selected names from `torch._utils`. | CN: 从 `torch._utils` 导入指定名称。
- **L18** EN: Imports selected names from `torch.distributed`. | CN: 从 `torch.distributed` 导入指定名称。
- **L19** EN: Imports selected names from `torch.distributed._shard._utils`. | CN: 从 `torch.distributed._shard._utils` 导入指定名称。
- **L20** EN: Imports selected names from `torch.distributed._shard.sharding_spec._internals`. | CN: 从 `torch.distributed._shard.sharding_spec._internals` 导入指定名称。

### Lines 21-40 / 第 21-40 行

````python
    check_tensor,
    validate_non_overlapping_shards_metadata,
)
from torch.distributed._shard.sharding_spec.api import (
    _dispatch_custom_op,
    _has_custom_op,
)
from torch.distributed.remote_device import _remote_device
from torch.utils import _pytree as pytree

from .metadata import ShardedTensorMetadata, TensorProperties
from .reshard import reshard_local_shard, reshuffle_local_shard
from .shard import Shard
from .utils import (
    _flatten_tensor_size,
    _parse_and_validate_remote_device,
    _validate_output_tensor_for_gather,
    build_global_metadata,
    build_metadata_from_local_shards,
)
````

- **L21** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L22** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L23** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L24** EN: Imports selected names from `torch.distributed._shard.sharding_spec.api`. | CN: 从 `torch.distributed._shard.sharding_spec.api` 导入指定名称。
- **L25** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L26** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L27** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L28** EN: Imports selected names from `torch.distributed.remote_device`. | CN: 从 `torch.distributed.remote_device` 导入指定名称。
- **L29** EN: Imports selected names from `torch.utils`. | CN: 从 `torch.utils` 导入指定名称。
- **L30** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L31** EN: Imports selected names from `.metadata`. | CN: 从 `.metadata` 导入指定名称。
- **L32** EN: Imports selected names from `.reshard`. | CN: 从 `.reshard` 导入指定名称。
- **L33** EN: Imports selected names from `.shard`. | CN: 从 `.shard` 导入指定名称。
- **L34** EN: Imports selected names from `.utils`. | CN: 从 `.utils` 导入指定名称。
- **L35** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L36** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L37** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L38** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L39** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L40** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 41-60 / 第 41-60 行

````python


if TYPE_CHECKING:
    from collections.abc import Callable, Sequence

    from torch.distributed._shard.metadata import ShardMetadata


# Tracking for sharded tensor objects.
_sharded_tensor_lock = threading.Lock()
_sharded_tensor_current_id = 0
_sharded_tensor_map: dict[int, weakref.ReferenceType[ShardedTensor]] = {}

# Default sharded ops
_SHARDED_OPS: dict[Callable, Callable] = {}

# Customized user ops
_CUSTOM_SHARDED_OPS: dict[Callable, Callable] = {}


````

- **L41** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L42** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L43** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L44** EN: Imports selected names from `collections.abc`. | CN: 从 `collections.abc` 导入指定名称。
- **L45** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L46** EN: Imports selected names from `torch.distributed._shard.metadata`. | CN: 从 `torch.distributed._shard.metadata` 导入指定名称。
- **L47** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L48** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L49** EN: Keeps the inline comment or directive: Tracking for sharded tensor objects. | CN: 保留这一行注释或指令：Tracking for sharded tensor objects.
- **L50** EN: Assigns or updates `_sharded_tensor_lock`. | CN: 对 `_sharded_tensor_lock` 进行赋值或更新。
- **L51** EN: Assigns or updates `_sharded_tensor_current_id`. | CN: 对 `_sharded_tensor_current_id` 进行赋值或更新。
- **L52** EN: Assigns or updates `_sharded_tensor_map`. | CN: 对 `_sharded_tensor_map` 进行赋值或更新。
- **L53** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L54** EN: Keeps the inline comment or directive: Default sharded ops | CN: 保留这一行注释或指令：Default sharded ops
- **L55** EN: Assigns or updates `_SHARDED_OPS`. | CN: 对 `_SHARDED_OPS` 进行赋值或更新。
- **L56** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L57** EN: Keeps the inline comment or directive: Customized user ops | CN: 保留这一行注释或指令：Customized user ops
- **L58** EN: Assigns or updates `_CUSTOM_SHARDED_OPS`. | CN: 对 `_CUSTOM_SHARDED_OPS` 进行赋值或更新。
- **L59** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L60** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 61-80 / 第 61-80 行

````python
def _register_remote_shards(
    sharded_tensor_id: int, rrefs: list[rpc.RRef[Shard]], rpc_rank: int
):
    with _sharded_tensor_lock:
        if sharded_tensor_id not in _sharded_tensor_map:
            raise RuntimeError(
                f"Could not find sharded_tensor_id: {sharded_tensor_id} in map: {_sharded_tensor_map.keys()}"
            )

        sharded_tensor = _sharded_tensor_map[sharded_tensor_id]()
        if sharded_tensor is None:
            raise RuntimeError("ShardedTensor weakref has been deallocated")
        else:
            sharded_tensor._register_remote_shards(rrefs, rpc_rank)


class ShardedTensorBase(torch.Tensor):
    _sharding_spec: shard_spec.ShardingSpec
    _metadata: ShardedTensorMetadata
    _local_shards: list[Shard]
````

- **L61** EN: Defines function `_register_remote_shards`. | CN: 定义函数 `_register_remote_shards`。
- **L62** EN: Continues the implementation inside function `_register_remote_shards`. | CN: 继续说明函数 `_register_remote_shards` 内部的实现。
- **L63** EN: Continues the implementation inside function `_register_remote_shards`. | CN: 继续说明函数 `_register_remote_shards` 内部的实现。
- **L64** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L65** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L66** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L67** EN: Continues the implementation inside function `_register_remote_shards`. | CN: 继续说明函数 `_register_remote_shards` 内部的实现。
- **L68** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L69** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L70** EN: Assigns or updates `sharded_tensor`. | CN: 对 `sharded_tensor` 进行赋值或更新。
- **L71** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L72** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L73** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L74** EN: Calls `sharded_tensor._register_remote_shards` as part of the current workflow. | CN: 在当前流程中调用 `sharded_tensor._register_remote_shards`。
- **L75** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L76** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L77** EN: Defines class `ShardedTensorBase`. | CN: 定义类 `ShardedTensorBase`。
- **L78** EN: Continues the implementation inside class `ShardedTensorBase`. | CN: 继续说明类 `ShardedTensorBase` 内部的实现。
- **L79** EN: Continues the implementation inside class `ShardedTensorBase`. | CN: 继续说明类 `ShardedTensorBase` 内部的实现。
- **L80** EN: Continues the implementation inside class `ShardedTensorBase`. | CN: 继续说明类 `ShardedTensorBase` 内部的实现。

### Lines 81-100 / 第 81-100 行

````python

    def __new__(cls, sharding_spec: shard_spec.ShardingSpec, *size, **kwargs):
        # Use __new__ to construct a wrapper tensor, for recording tensor
        # properties and logging purposes.
        torch._C._log_api_usage_once("torch.distributed._shard.sharded_tensor")

        # check sharding spec and build sharded tensor metadata
        if not isinstance(sharding_spec, shard_spec.ShardingSpec):
            raise ValueError(f"Expecting ShardingSpec but got: {type(sharding_spec)}")

        sizes = _flatten_tensor_size(size)
        dtype = kwargs["dtype"]
        layout = kwargs["layout"]
        pin_memory = kwargs["pin_memory"]
        requires_grad = kwargs["requires_grad"]

        if dtype is None:
            dtype = torch.get_default_dtype()

        tensor_properties = TensorProperties(
````

- **L81** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L82** EN: Defines function `__new__`. | CN: 定义函数 `__new__`。
- **L83** EN: Keeps the inline comment or directive: Use __new__ to construct a wrapper tensor, for recording tensor | CN: 保留这一行注释或指令：Use __new__ to construct a wrapper tensor, for recording tensor
- **L84** EN: Keeps the inline comment or directive: properties and logging purposes. | CN: 保留这一行注释或指令：properties and logging purposes.
- **L85** EN: Calls `torch._C._log_api_usage_once` as part of the current workflow. | CN: 在当前流程中调用 `torch._C._log_api_usage_once`。
- **L86** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L87** EN: Keeps the inline comment or directive: check sharding spec and build sharded tensor metadata | CN: 保留这一行注释或指令：check sharding spec and build sharded tensor metadata
- **L88** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L89** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L90** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L91** EN: Assigns or updates `sizes`. | CN: 对 `sizes` 进行赋值或更新。
- **L92** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L93** EN: Assigns or updates `layout`. | CN: 对 `layout` 进行赋值或更新。
- **L94** EN: Assigns or updates `pin_memory`. | CN: 对 `pin_memory` 进行赋值或更新。
- **L95** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L96** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L97** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L98** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L99** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L100** EN: Assigns or updates `tensor_properties`. | CN: 对 `tensor_properties` 进行赋值或更新。

### Lines 101-120 / 第 101-120 行

````python
            dtype, layout, requires_grad, pin_memory=pin_memory
        )
        sharded_tensor_metadata = sharding_spec.build_metadata(
            sizes, tensor_properties=tensor_properties
        )

        r = torch.Tensor._make_wrapper_subclass(
            cls,
            sizes,
            dtype=dtype,
            layout=layout,
            pin_memory=pin_memory,
            requires_grad=requires_grad,
        )
        # set sharding spec
        r._sharding_spec = sharding_spec
        # set metadata
        r._metadata = sharded_tensor_metadata
        # set local shards
        r._local_shards = []
````

- **L101** EN: Assigns or updates `dtype, layout, requires_grad, pin_memory`. | CN: 对 `dtype, layout, requires_grad, pin_memory` 进行赋值或更新。
- **L102** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L103** EN: Assigns or updates `sharded_tensor_metadata`. | CN: 对 `sharded_tensor_metadata` 进行赋值或更新。
- **L104** EN: Assigns or updates `sizes, tensor_properties`. | CN: 对 `sizes, tensor_properties` 进行赋值或更新。
- **L105** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L106** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L107** EN: Assigns or updates `r`. | CN: 对 `r` 进行赋值或更新。
- **L108** EN: Continues the implementation inside function `__new__`. | CN: 继续说明函数 `__new__` 内部的实现。
- **L109** EN: Continues the implementation inside function `__new__`. | CN: 继续说明函数 `__new__` 内部的实现。
- **L110** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L111** EN: Assigns or updates `layout`. | CN: 对 `layout` 进行赋值或更新。
- **L112** EN: Assigns or updates `pin_memory`. | CN: 对 `pin_memory` 进行赋值或更新。
- **L113** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L114** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L115** EN: Keeps the inline comment or directive: set sharding spec | CN: 保留这一行注释或指令：set sharding spec
- **L116** EN: Assigns or updates `r._sharding_spec`. | CN: 对 `r._sharding_spec` 进行赋值或更新。
- **L117** EN: Keeps the inline comment or directive: set metadata | CN: 保留这一行注释或指令：set metadata
- **L118** EN: Assigns or updates `r._metadata`. | CN: 对 `r._metadata` 进行赋值或更新。
- **L119** EN: Keeps the inline comment or directive: set local shards | CN: 保留这一行注释或指令：set local shards
- **L120** EN: Assigns or updates `r._local_shards`. | CN: 对 `r._local_shards` 进行赋值或更新。

### Lines 121-140 / 第 121-140 行

````python
        return r

    def metadata(self) -> ShardedTensorMetadata:
        """
        Returns a :class:`ShardedTensorMetadata` object corresponding to the
        metadata for the entire tensor.
        """
        return self._metadata

    def local_shards(self) -> list[Shard]:
        """
        Returns a list of :class:`Shard' corresponding to the
        local shards for this rank. Returns an empty list if the current rank
        does not host any shards for this Tensor.
        """
        return self._local_shards

    @classmethod
    def _init_from_local_shards_and_global_metadata(
        cls,
````

- **L121** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L122** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L123** EN: Defines function `metadata`. | CN: 定义函数 `metadata`。
- **L124** EN: Starts the docstring for the function metadata. | CN: 开始定义 function metadata 的文档字符串。
- **L125** EN: Continues the docstring text for the function metadata. | CN: 继续补充 function metadata 的文档字符串内容。
- **L126** EN: Continues the docstring text for the function metadata. | CN: 继续补充 function metadata 的文档字符串内容。
- **L127** EN: Closes the docstring for the function metadata. | CN: 结束 function metadata 的文档字符串。
- **L128** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L129** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L130** EN: Defines function `local_shards`. | CN: 定义函数 `local_shards`。
- **L131** EN: Starts the docstring for the function local_shards. | CN: 开始定义 function local_shards 的文档字符串。
- **L132** EN: Continues the docstring text for the function local_shards. | CN: 继续补充 function local_shards 的文档字符串内容。
- **L133** EN: Continues the docstring text for the function local_shards. | CN: 继续补充 function local_shards 的文档字符串内容。
- **L134** EN: Continues the docstring text for the function local_shards. | CN: 继续补充 function local_shards 的文档字符串内容。
- **L135** EN: Closes the docstring for the function local_shards. | CN: 结束 function local_shards 的文档字符串。
- **L136** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L137** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L138** EN: Applies decorator `classmethod` to the following definition. | CN: 将装饰器 `classmethod` 应用于后续定义。
- **L139** EN: Defines function `_init_from_local_shards_and_global_metadata`. | CN: 定义函数 `_init_from_local_shards_and_global_metadata`。
- **L140** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。

### Lines 141-160 / 第 141-160 行

````python
        local_shards: list[Shard],
        sharded_tensor_metadata: ShardedTensorMetadata,
        sharding_spec=None,
    ) -> ShardedTensorBase:
        """
        Initialize a ShardedTensorBase with local shards and a global
        ShardedTensorMetadata built on each rank.
        Warning: This API is experimental and subject to change. It does
                 not do cross rank validations, and fully rely on the user
                 for the correctness of sharded_tensor_metadata on each rank
        """
        shards_metadata = sharded_tensor_metadata.shards_metadata
        tensor_properties = sharded_tensor_metadata.tensor_properties

        if len(shards_metadata) == 0:
            raise ValueError("shards_metadata must not be empty!")

        if tensor_properties.layout != torch.strided:
            raise ValueError("Only torch.strided layout is currently supported")

````

- **L141** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L142** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L143** EN: Assigns or updates `sharding_spec`. | CN: 对 `sharding_spec` 进行赋值或更新。
- **L144** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L145** EN: Starts the docstring for the function _init_from_local_shards_and_global_metadata. | CN: 开始定义 function _init_from_local_shards_and_global_metadata 的文档字符串。
- **L146** EN: Continues the docstring text for the function _init_from_local_shards_and_global_metadata. | CN: 继续补充 function _init_from_local_shards_and_global_metadata 的文档字符串内容。
- **L147** EN: Continues the docstring text for the function _init_from_local_shards_and_global_metadata. | CN: 继续补充 function _init_from_local_shards_and_global_metadata 的文档字符串内容。
- **L148** EN: Continues the docstring text for the function _init_from_local_shards_and_global_metadata. | CN: 继续补充 function _init_from_local_shards_and_global_metadata 的文档字符串内容。
- **L149** EN: Continues the docstring text for the function _init_from_local_shards_and_global_metadata. | CN: 继续补充 function _init_from_local_shards_and_global_metadata 的文档字符串内容。
- **L150** EN: Continues the docstring text for the function _init_from_local_shards_and_global_metadata. | CN: 继续补充 function _init_from_local_shards_and_global_metadata 的文档字符串内容。
- **L151** EN: Closes the docstring for the function _init_from_local_shards_and_global_metadata. | CN: 结束 function _init_from_local_shards_and_global_metadata 的文档字符串。
- **L152** EN: Assigns or updates `shards_metadata`. | CN: 对 `shards_metadata` 进行赋值或更新。
- **L153** EN: Assigns or updates `tensor_properties`. | CN: 对 `tensor_properties` 进行赋值或更新。
- **L154** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L155** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L156** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L157** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L158** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L159** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L160** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 161-180 / 第 161-180 行

````python
        if sharding_spec is None:
            spec = shard_spec._infer_sharding_spec_from_shards_metadata(shards_metadata)
        else:
            spec = sharding_spec

        sharded_tensor_base = ShardedTensorBase.__new__(
            ShardedTensor,
            spec,
            sharded_tensor_metadata.size,
            dtype=tensor_properties.dtype,
            layout=tensor_properties.layout,
            pin_memory=tensor_properties.pin_memory,
            requires_grad=tensor_properties.requires_grad,
        )

        # check if shards_metadata have overlap shards
        validate_non_overlapping_shards_metadata(shards_metadata)

        # check if the shards_metadata is compatible with overall size of the sharded tensor.
        check_tensor(shards_metadata, list(sharded_tensor_metadata.size))
````

- **L161** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L162** EN: Assigns or updates `spec`. | CN: 对 `spec` 进行赋值或更新。
- **L163** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L164** EN: Assigns or updates `spec`. | CN: 对 `spec` 进行赋值或更新。
- **L165** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L166** EN: Assigns or updates `sharded_tensor_base`. | CN: 对 `sharded_tensor_base` 进行赋值或更新。
- **L167** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L168** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L169** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L170** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L171** EN: Assigns or updates `layout`. | CN: 对 `layout` 进行赋值或更新。
- **L172** EN: Assigns or updates `pin_memory`. | CN: 对 `pin_memory` 进行赋值或更新。
- **L173** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L174** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L175** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L176** EN: Keeps the inline comment or directive: check if shards_metadata have overlap shards | CN: 保留这一行注释或指令：check if shards_metadata have overlap shards
- **L177** EN: Calls `validate_non_overlapping_shards_metadata` as part of the current workflow. | CN: 在当前流程中调用 `validate_non_overlapping_shards_metadata`。
- **L178** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L179** EN: Keeps the inline comment or directive: check if the shards_metadata is compatible with overall size of the sharded tens | CN: 保留这一行注释或指令：check if the shards_metadata is compatible with overall size of the sharded tens
- **L180** EN: Calls `check_tensor` as part of the current workflow. | CN: 在当前流程中调用 `check_tensor`。

### Lines 181-200 / 第 181-200 行

````python

        # done validation, add local_shards
        sharded_tensor_base._local_shards = local_shards
        return sharded_tensor_base

    @classmethod
    def __torch_dispatch__(cls, func, types, args=(), kwargs=None):  # type: ignore[override]
        raise RuntimeError(
            f"A {cls.__name__} object is being used from c++ while calling {func.__module__}.{func.__name__} "
            "but the there is no custom __torch_dispatch__ implementation for it."
        )


class ShardedTensor(ShardedTensorBase):
    """
    ShardedTensor is an torch.Tensor subclass to represent Tensors that are sharded
    across multiple devices and multiple processes.

    ShardedTensor is initialized in an SPMD like fashion where each rank
    initializes the ShardedTensor. The ShardedTensor object on each rank
````

- **L181** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L182** EN: Keeps the inline comment or directive: done validation, add local_shards | CN: 保留这一行注释或指令：done validation, add local_shards
- **L183** EN: Assigns or updates `sharded_tensor_base._local_shards`. | CN: 对 `sharded_tensor_base._local_shards` 进行赋值或更新。
- **L184** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L185** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L186** EN: Applies decorator `classmethod` to the following definition. | CN: 将装饰器 `classmethod` 应用于后续定义。
- **L187** EN: Defines function `__torch_dispatch__`. | CN: 定义函数 `__torch_dispatch__`。
- **L188** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L189** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L190** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L191** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L192** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L193** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L194** EN: Defines class `ShardedTensor`. | CN: 定义类 `ShardedTensor`。
- **L195** EN: Starts the docstring for the class ShardedTensor. | CN: 开始定义 class ShardedTensor 的文档字符串。
- **L196** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L197** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L198** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L199** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L200** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。

### Lines 201-220 / 第 201-220 行

````python
    then only stores the local shard for the Tensor and provides global
    metadata for all the shards.

    ShardedTensor doesn't provide any Tensor like operations but is a wrapper
    providing the Tensor representing the local shard and the global metadata.
    Using these, users can build their custom distributed._sharded computations
    on top of this primitive. The local shards are all initialized using the
    create_op specified by tensor_init_params.create_op, e.g., torch.ones, or
    torch.empty

    Args:
        sharding_spec (:class:`torch.distributed._shard.sharding_spec.ShardingSpec`): The specification
            describing how to shard the Tensor.
        size (int...): a sequence of integers defining the shape of the output
            tensor. Can be a variable number of arguments or a collection like a list or tuple.

    Keyword args:
        dtype (:class:`torch.dtype`, optional): the desired data type of returned tensor.
                Default: if ``None``, uses a global default (see :func:`torch.set_default_dtype`).
        layout (:class:`torch.layout`, optional): the desired layout of returned Tensor.
````

- **L201** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L202** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L203** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L204** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L205** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L206** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L207** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L208** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L209** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L210** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L211** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L212** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L213** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L214** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L215** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L216** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L217** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L218** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L219** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L220** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。

### Lines 221-240 / 第 221-240 行

````python
            Default: ``torch.strided``.
        requires_grad (bool, optional): If autograd should record operations on the
            returned tensor. Default: ``False``.
        pin_memory (bool, optional): If set, returned tensor would be allocated in
            the pinned memory. Works only for CPU tensors. Default: ``False``.
        memory_format (:class:`torch.memory_format`, optional): the desired memory format of
            returned Tensor. Default: ``torch.contiguous_format``.
        init_rrefs (bool, optional): Whether or not to initialize
            :class:`torch.distributed.rpc.RRef`s pointing to remote shards.
            Need to initialize the RPC Framework if specified as ``True``.
            Default: ``False``.

    .. note:: ShardedTensor uses collectives to do various operations, i.e. it
        uses all_gather to do cross rank validations. For NCCL-based process
        groups, internal tensor representations of objects must be moved to the
        GPU device before communication takes place. In this case, the device
        used is given by ``torch.cuda.current_device()`` and it is the user's
        responsibility to ensure that this is set so that each rank has an
        individual GPU, via ``torch.cuda.set_device()``

````

- **L221** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L222** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L223** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L224** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L225** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L226** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L227** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L228** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L229** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L230** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L231** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L232** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L233** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L234** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L235** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L236** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L237** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L238** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L239** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。
- **L240** EN: Continues the docstring text for the class ShardedTensor. | CN: 继续补充 class ShardedTensor 的文档字符串内容。

### Lines 241-260 / 第 241-260 行

````python
    """

    def __new__(cls, sharding_spec: shard_spec.ShardingSpec, *size, **kwargs):
        self = super().__new__(cls, sharding_spec, *size, **kwargs)
        return self

    def __init__(
        self,
        sharding_spec: shard_spec.ShardingSpec,
        *size,
        dtype=None,
        layout=torch.strided,
        requires_grad=False,
        pin_memory=False,
        memory_format=torch.contiguous_format,
        process_group=None,
        init_rrefs=False,
    ):
        # prepare initialization, initialize fields like
        # _process_group, _local_shards, etc.
````

- **L241** EN: Closes the docstring for the class ShardedTensor. | CN: 结束 class ShardedTensor 的文档字符串。
- **L242** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L243** EN: Defines function `__new__`. | CN: 定义函数 `__new__`。
- **L244** EN: Assigns or updates `self`. | CN: 对 `self` 进行赋值或更新。
- **L245** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L246** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L247** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L248** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L249** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L250** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L251** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L252** EN: Assigns or updates `layout`. | CN: 对 `layout` 进行赋值或更新。
- **L253** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L254** EN: Assigns or updates `pin_memory`. | CN: 对 `pin_memory` 进行赋值或更新。
- **L255** EN: Assigns or updates `memory_format`. | CN: 对 `memory_format` 进行赋值或更新。
- **L256** EN: Assigns or updates `process_group`. | CN: 对 `process_group` 进行赋值或更新。
- **L257** EN: Assigns or updates `init_rrefs`. | CN: 对 `init_rrefs` 进行赋值或更新。
- **L258** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L259** EN: Keeps the inline comment or directive: prepare initialization, initialize fields like | CN: 保留这一行注释或指令：prepare initialization, initialize fields like
- **L260** EN: Keeps the inline comment or directive: _process_group, _local_shards, etc. | CN: 保留这一行注释或指令：_process_group, _local_shards, etc.

### Lines 261-280 / 第 261-280 行

````python
        self._prepare_init(process_group=process_group, init_rrefs=init_rrefs)

        if layout != torch.strided:
            raise ValueError("Only torch.strided layout is currently supported")

        if memory_format != torch.contiguous_format:
            raise ValueError(
                "Only torch.contiguous_format memory_format is currently supported"
            )

        self._metadata.tensor_properties.memory_format = memory_format

        current_rank = dist.get_rank()  # global rank

        for shard_metadata in self._metadata.shards_metadata:
            rank, device = _parse_and_validate_remote_device(
                self._process_group, shard_metadata.placement
            )
            if rank == current_rank:
                local_tensor = _create_tensor_from_params(
````

- **L261** EN: Calls `self._prepare_init` as part of the current workflow. | CN: 在当前流程中调用 `self._prepare_init`。
- **L262** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L263** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L264** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L265** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L266** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L267** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L268** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L269** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L270** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L271** EN: Assigns or updates `self._metadata.tensor_properties.memory_format`. | CN: 对 `self._metadata.tensor_properties.memory_format` 进行赋值或更新。
- **L272** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L273** EN: Assigns or updates `current_rank`. | CN: 对 `current_rank` 进行赋值或更新。
- **L274** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L275** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L276** EN: Assigns or updates `rank, device`. | CN: 对 `rank, device` 进行赋值或更新。
- **L277** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L278** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L279** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L280** EN: Assigns or updates `local_tensor`. | CN: 对 `local_tensor` 进行赋值或更新。

### Lines 281-300 / 第 281-300 行

````python
                    shard_metadata.shard_sizes,
                    local_device=device,
                    tensor_properties=self._metadata.tensor_properties,
                )
                self._local_shards.append(Shard(local_tensor, shard_metadata))

        # do post initialization (i.e. register sharded_tensor_id, initialize_rpc)
        self._post_init()

    def _prepare_init(self, process_group=None, init_rrefs=False):
        self._init_rrefs = init_rrefs
        self._sharded_tensor_id = None

        self._process_group = self._normalize_pg(process_group)
        self._remote_shards: dict[int, list[rpc.RRef[Shard]]] = {}

    def _post_init(self):
        # Initialize RPC if available.
        if self._init_rrefs:
            with _sharded_tensor_lock:
````

- **L281** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L282** EN: Assigns or updates `local_device`. | CN: 对 `local_device` 进行赋值或更新。
- **L283** EN: Assigns or updates `tensor_properties`. | CN: 对 `tensor_properties` 进行赋值或更新。
- **L284** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L285** EN: Calls `self._local_shards.append` as part of the current workflow. | CN: 在当前流程中调用 `self._local_shards.append`。
- **L286** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L287** EN: Keeps the inline comment or directive: do post initialization (i.e. register sharded_tensor_id, initialize_rpc) | CN: 保留这一行注释或指令：do post initialization (i.e. register sharded_tensor_id, initialize_rpc)
- **L288** EN: Calls `self._post_init` as part of the current workflow. | CN: 在当前流程中调用 `self._post_init`。
- **L289** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L290** EN: Defines function `_prepare_init`. | CN: 定义函数 `_prepare_init`。
- **L291** EN: Assigns or updates `self._init_rrefs`. | CN: 对 `self._init_rrefs` 进行赋值或更新。
- **L292** EN: Assigns or updates `self._sharded_tensor_id`. | CN: 对 `self._sharded_tensor_id` 进行赋值或更新。
- **L293** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L294** EN: Assigns or updates `self._process_group`. | CN: 对 `self._process_group` 进行赋值或更新。
- **L295** EN: Assigns or updates `self._remote_shards`. | CN: 对 `self._remote_shards` 进行赋值或更新。
- **L296** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L297** EN: Defines function `_post_init`. | CN: 定义函数 `_post_init`。
- **L298** EN: Keeps the inline comment or directive: Initialize RPC if available. | CN: 保留这一行注释或指令：Initialize RPC if available.
- **L299** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L300** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。

### Lines 301-320 / 第 301-320 行

````python
                global _sharded_tensor_current_id, _sharded_tensor_map
                self._sharded_tensor_id = _sharded_tensor_current_id
                _sharded_tensor_map[self._sharded_tensor_id] = weakref.ref(self)
                _sharded_tensor_current_id += 1

            if not rpc._is_current_rpc_agent_set():
                raise RuntimeError(
                    "RPC Framework needs to be initialized using"
                    " torch.distributed.rpc.init_rpc if init_rrefs is set to True"
                )
            self._init_rpc()

    def __del__(self):
        # Clean up the global map.
        with _sharded_tensor_lock:
            global _sharded_tensor_current_id, _sharded_tensor_map
            if (
                hasattr(self, "_sharded_tensor_id")
                and self._sharded_tensor_id in _sharded_tensor_map
            ):
````

- **L301** EN: Declares module-level names for reassignment. | CN: 声明需要重新赋值的模块级名称。
- **L302** EN: Assigns or updates `self._sharded_tensor_id`. | CN: 对 `self._sharded_tensor_id` 进行赋值或更新。
- **L303** EN: Assigns or updates `_sharded_tensor_map[self._sharded_tensor_id]`. | CN: 对 `_sharded_tensor_map[self._sharded_tensor_id]` 进行赋值或更新。
- **L304** EN: Continues the implementation inside function `_post_init`. | CN: 继续说明函数 `_post_init` 内部的实现。
- **L305** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L306** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L307** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L308** EN: Continues the implementation inside function `_post_init`. | CN: 继续说明函数 `_post_init` 内部的实现。
- **L309** EN: Continues the implementation inside function `_post_init`. | CN: 继续说明函数 `_post_init` 内部的实现。
- **L310** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L311** EN: Calls `self._init_rpc` as part of the current workflow. | CN: 在当前流程中调用 `self._init_rpc`。
- **L312** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L313** EN: Defines function `__del__`. | CN: 定义函数 `__del__`。
- **L314** EN: Keeps the inline comment or directive: Clean up the global map. | CN: 保留这一行注释或指令：Clean up the global map.
- **L315** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L316** EN: Declares module-level names for reassignment. | CN: 声明需要重新赋值的模块级名称。
- **L317** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L318** EN: Calls `hasattr` as part of the current workflow. | CN: 在当前流程中调用 `hasattr`。
- **L319** EN: Continues the implementation inside function `__del__`. | CN: 继续说明函数 `__del__` 内部的实现。
- **L320** EN: Continues the implementation inside function `__del__`. | CN: 继续说明函数 `__del__` 内部的实现。

### Lines 321-340 / 第 321-340 行

````python
                _sharded_tensor_map.pop(self._sharded_tensor_id)  # type: ignore[call-overload]

    def _init_rpc(self):
        # Validate PG and RPC ranks match.
        pg_rank = dist.get_rank()
        rpc_rank = rpc.get_worker_info().id
        if pg_rank != rpc_rank:
            raise ValueError(
                f"Default ProcessGroup and RPC ranks must be "
                f"the same for ShardedTensor, found process group rank: "
                f"{pg_rank} and RPC rank: {rpc_rank}"
            )

        self._remote_shards = {}

        # Gather all the sharded tensor ids.
        worker_infos = rpc._get_current_rpc_agent().get_worker_infos()
        rank_to_name = {}
        name_to_rank = {}

````

- **L321** EN: Calls `_sharded_tensor_map.pop` as part of the current workflow. | CN: 在当前流程中调用 `_sharded_tensor_map.pop`。
- **L322** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L323** EN: Defines function `_init_rpc`. | CN: 定义函数 `_init_rpc`。
- **L324** EN: Keeps the inline comment or directive: Validate PG and RPC ranks match. | CN: 保留这一行注释或指令：Validate PG and RPC ranks match.
- **L325** EN: Assigns or updates `pg_rank`. | CN: 对 `pg_rank` 进行赋值或更新。
- **L326** EN: Assigns or updates `rpc_rank`. | CN: 对 `rpc_rank` 进行赋值或更新。
- **L327** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L328** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L329** EN: Continues the implementation inside function `_init_rpc`. | CN: 继续说明函数 `_init_rpc` 内部的实现。
- **L330** EN: Continues the implementation inside function `_init_rpc`. | CN: 继续说明函数 `_init_rpc` 内部的实现。
- **L331** EN: Continues the implementation inside function `_init_rpc`. | CN: 继续说明函数 `_init_rpc` 内部的实现。
- **L332** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L333** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L334** EN: Assigns or updates `self._remote_shards`. | CN: 对 `self._remote_shards` 进行赋值或更新。
- **L335** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L336** EN: Keeps the inline comment or directive: Gather all the sharded tensor ids. | CN: 保留这一行注释或指令：Gather all the sharded tensor ids.
- **L337** EN: Assigns or updates `worker_infos`. | CN: 对 `worker_infos` 进行赋值或更新。
- **L338** EN: Assigns or updates `rank_to_name`. | CN: 对 `rank_to_name` 进行赋值或更新。
- **L339** EN: Assigns or updates `name_to_rank`. | CN: 对 `name_to_rank` 进行赋值或更新。
- **L340** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 341-360 / 第 341-360 行

````python
        for worker_info in worker_infos:
            rank_to_name[worker_info.id] = worker_info.name
            name_to_rank[worker_info.name] = worker_info.id

        all_tensor_ids = rpc.api._all_gather(self._sharded_tensor_id)

        # Share the local shards to the entire world.
        futs = []
        rpc_rank = rpc.get_worker_info().id
        for rank in range(dist.get_world_size()):
            # Skip self.
            if rank == dist.get_rank():
                continue

            if len(self.local_shards()) != 0:
                rrefs: list[rpc.RRef[Shard]] = [
                    rpc.RRef(shard) for shard in self.local_shards()
                ]
                fut = rpc.rpc_async(
                    rank,
````

- **L341** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L342** EN: Assigns or updates `rank_to_name[worker_info.id]`. | CN: 对 `rank_to_name[worker_info.id]` 进行赋值或更新。
- **L343** EN: Assigns or updates `name_to_rank[worker_info.name]`. | CN: 对 `name_to_rank[worker_info.name]` 进行赋值或更新。
- **L344** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L345** EN: Assigns or updates `all_tensor_ids`. | CN: 对 `all_tensor_ids` 进行赋值或更新。
- **L346** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L347** EN: Keeps the inline comment or directive: Share the local shards to the entire world. | CN: 保留这一行注释或指令：Share the local shards to the entire world.
- **L348** EN: Assigns or updates `futs`. | CN: 对 `futs` 进行赋值或更新。
- **L349** EN: Assigns or updates `rpc_rank`. | CN: 对 `rpc_rank` 进行赋值或更新。
- **L350** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L351** EN: Keeps the inline comment or directive: Skip self. | CN: 保留这一行注释或指令：Skip self.
- **L352** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L353** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L354** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L355** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L356** EN: Assigns or updates `rrefs`. | CN: 对 `rrefs` 进行赋值或更新。
- **L357** EN: Calls `rpc.RRef` as part of the current workflow. | CN: 在当前流程中调用 `rpc.RRef`。
- **L358** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L359** EN: Assigns or updates `fut`. | CN: 对 `fut` 进行赋值或更新。
- **L360** EN: Continues the implementation inside function `_init_rpc`. | CN: 继续说明函数 `_init_rpc` 内部的实现。

### Lines 361-380 / 第 361-380 行

````python
                    _register_remote_shards,
                    args=(all_tensor_ids[rank_to_name[rank]], rrefs, rpc_rank),
                )
                futs.append(fut)

        torch.futures.wait_all(futs)

        # Barrier for all RPCs to finish on all ranks.
        rpc.api._all_gather(None)

    def _get_preferred_device(self) -> torch.device:
        """
        Return the preferred device to be used when creating tensors for collectives.
        This method takes into account the associated process group
        """
        backend = dist.get_backend(self._process_group)
        if backend == dist.Backend.NCCL:
            return torch.device(torch.cuda.current_device())
        elif backend == dist.Backend.GLOO:
            return torch.device("cpu")
````

- **L361** EN: Continues the implementation inside function `_init_rpc`. | CN: 继续说明函数 `_init_rpc` 内部的实现。
- **L362** EN: Assigns or updates `args`. | CN: 对 `args` 进行赋值或更新。
- **L363** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L364** EN: Calls `futs.append` as part of the current workflow. | CN: 在当前流程中调用 `futs.append`。
- **L365** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L366** EN: Calls `torch.futures.wait_all` as part of the current workflow. | CN: 在当前流程中调用 `torch.futures.wait_all`。
- **L367** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L368** EN: Keeps the inline comment or directive: Barrier for all RPCs to finish on all ranks. | CN: 保留这一行注释或指令：Barrier for all RPCs to finish on all ranks.
- **L369** EN: Calls `rpc.api._all_gather` as part of the current workflow. | CN: 在当前流程中调用 `rpc.api._all_gather`。
- **L370** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L371** EN: Defines function `_get_preferred_device`. | CN: 定义函数 `_get_preferred_device`。
- **L372** EN: Starts the docstring for the function _get_preferred_device. | CN: 开始定义 function _get_preferred_device 的文档字符串。
- **L373** EN: Continues the docstring text for the function _get_preferred_device. | CN: 继续补充 function _get_preferred_device 的文档字符串内容。
- **L374** EN: Continues the docstring text for the function _get_preferred_device. | CN: 继续补充 function _get_preferred_device 的文档字符串内容。
- **L375** EN: Closes the docstring for the function _get_preferred_device. | CN: 结束 function _get_preferred_device 的文档字符串。
- **L376** EN: Assigns or updates `backend`. | CN: 对 `backend` 进行赋值或更新。
- **L377** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L378** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L379** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L380** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

### Lines 381-400 / 第 381-400 行

````python
        else:
            backend_config = dist.BackendConfig(backend)
            for device, backend_str in backend_config.get_device_backend_map().items():
                if backend_str == backend and device != "cpu":
                    return torch.device(
                        device, _get_device_module(device).current_device()
                    )
        return torch.device("cpu")

    def gather(  # type: ignore[override]
        self,
        dst: int = 0,
        out: torch.Tensor | None = None,
        enforce_dtype: bool = False,
        dtype: torch.dtype | None = None,
    ) -> None:
        """
        Creates a full :class:`Tensor` on rank ``dst`` by gathering all shards of the
        sharded tensor.

````

- **L381** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L382** EN: Assigns or updates `backend_config`. | CN: 对 `backend_config` 进行赋值或更新。
- **L383** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L384** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L385** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L386** EN: Continues the implementation inside function `_get_preferred_device`. | CN: 继续说明函数 `_get_preferred_device` 内部的实现。
- **L387** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L388** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L389** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L390** EN: Defines function `gather`. | CN: 定义函数 `gather`。
- **L391** EN: Continues the implementation inside function `gather`. | CN: 继续说明函数 `gather` 内部的实现。
- **L392** EN: Assigns or updates `dst`. | CN: 对 `dst` 进行赋值或更新。
- **L393** EN: Assigns or updates `out`. | CN: 对 `out` 进行赋值或更新。
- **L394** EN: Assigns or updates `enforce_dtype`. | CN: 对 `enforce_dtype` 进行赋值或更新。
- **L395** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L396** EN: Continues the implementation inside function `gather`. | CN: 继续说明函数 `gather` 内部的实现。
- **L397** EN: Starts the docstring for the function gather. | CN: 开始定义 function gather 的文档字符串。
- **L398** EN: Continues the docstring text for the function gather. | CN: 继续补充 function gather 的文档字符串内容。
- **L399** EN: Continues the docstring text for the function gather. | CN: 继续补充 function gather 的文档字符串内容。
- **L400** EN: Continues the docstring text for the function gather. | CN: 继续补充 function gather 的文档字符串内容。

### Lines 401-420 / 第 401-420 行

````python
        The API needs to be called on all ranks in SPMD fashion. All ranks should have
        the same ``dst``. ``out`` should be a tensor of the same size as the overall
        size of the sharded tensor on ``dst`` and ``None`` on all other ranks.

        Args:
            dst(int): The rank where full tensor is constructed.
                Default: 0
            out (:class `torch.Tensor`, optional): The output full tensor.
                Must to be provided ONLY on ``dst`` rank.
                Default: ``None``
            enforce_dtype (bool): Deprecated, please use dtype instead.  Force the
                gathered tensors to be the same type as input and output.
            dtype (torch.dtype): Force the gathered tensors to be this dtype.
                Default: ``None``
        """

        def shard_size(shard_md):
            return reduce(operator.mul, shard_md.shard_sizes)  # type: ignore[attr-defined]

        if enforce_dtype:
````

- **L401** EN: Continues the docstring text for the function gather. | CN: 继续补充 function gather 的文档字符串内容。
- **L402** EN: Continues the docstring text for the function gather. | CN: 继续补充 function gather 的文档字符串内容。
- **L403** EN: Continues the docstring text for the function gather. | CN: 继续补充 function gather 的文档字符串内容。
- **L404** EN: Continues the docstring text for the function gather. | CN: 继续补充 function gather 的文档字符串内容。
- **L405** EN: Continues the docstring text for the function gather. | CN: 继续补充 function gather 的文档字符串内容。
- **L406** EN: Continues the docstring text for the function gather. | CN: 继续补充 function gather 的文档字符串内容。
- **L407** EN: Continues the docstring text for the function gather. | CN: 继续补充 function gather 的文档字符串内容。
- **L408** EN: Continues the docstring text for the function gather. | CN: 继续补充 function gather 的文档字符串内容。
- **L409** EN: Continues the docstring text for the function gather. | CN: 继续补充 function gather 的文档字符串内容。
- **L410** EN: Continues the docstring text for the function gather. | CN: 继续补充 function gather 的文档字符串内容。
- **L411** EN: Continues the docstring text for the function gather. | CN: 继续补充 function gather 的文档字符串内容。
- **L412** EN: Continues the docstring text for the function gather. | CN: 继续补充 function gather 的文档字符串内容。
- **L413** EN: Continues the docstring text for the function gather. | CN: 继续补充 function gather 的文档字符串内容。
- **L414** EN: Continues the docstring text for the function gather. | CN: 继续补充 function gather 的文档字符串内容。
- **L415** EN: Closes the docstring for the function gather. | CN: 结束 function gather 的文档字符串。
- **L416** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L417** EN: Defines function `shard_size`. | CN: 定义函数 `shard_size`。
- **L418** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L419** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L420** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 421-440 / 第 421-440 行

````python
            warnings.warn(
                "`enforce_dtype` is deprecated. Please use `dtype` instead.",
                FutureWarning,
                stacklevel=2,
            )

        rank = dist.get_rank(self._process_group)
        full_size = self.metadata().size
        _validate_output_tensor_for_gather(rank, dst, full_size, out)

        local_shards = self.local_shards()
        world_size = dist.get_world_size(self._process_group)
        rank_sizes = [0 for _ in range(world_size)]
        max_rank_size = 0
        shard_placement: dict[ShardMetadata, tuple[int, int]] = {}
        # collect sizes
        for shard_md in self.metadata().shards_metadata:
            shard_rank = cast(_remote_device, shard_md.placement).rank()
            if shard_rank is None:
                raise AssertionError
````

- **L421** EN: Calls `warnings.warn` as part of the current workflow. | CN: 在当前流程中调用 `warnings.warn`。
- **L422** EN: Continues the implementation inside function `gather`. | CN: 继续说明函数 `gather` 内部的实现。
- **L423** EN: Continues the implementation inside function `gather`. | CN: 继续说明函数 `gather` 内部的实现。
- **L424** EN: Assigns or updates `stacklevel`. | CN: 对 `stacklevel` 进行赋值或更新。
- **L425** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L426** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L427** EN: Assigns or updates `rank`. | CN: 对 `rank` 进行赋值或更新。
- **L428** EN: Assigns or updates `full_size`. | CN: 对 `full_size` 进行赋值或更新。
- **L429** EN: Calls `_validate_output_tensor_for_gather` as part of the current workflow. | CN: 在当前流程中调用 `_validate_output_tensor_for_gather`。
- **L430** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L431** EN: Assigns or updates `local_shards`. | CN: 对 `local_shards` 进行赋值或更新。
- **L432** EN: Assigns or updates `world_size`. | CN: 对 `world_size` 进行赋值或更新。
- **L433** EN: Assigns or updates `rank_sizes`. | CN: 对 `rank_sizes` 进行赋值或更新。
- **L434** EN: Assigns or updates `max_rank_size`. | CN: 对 `max_rank_size` 进行赋值或更新。
- **L435** EN: Assigns or updates `shard_placement`. | CN: 对 `shard_placement` 进行赋值或更新。
- **L436** EN: Keeps the inline comment or directive: collect sizes | CN: 保留这一行注释或指令：collect sizes
- **L437** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L438** EN: Assigns or updates `shard_rank`. | CN: 对 `shard_rank` 进行赋值或更新。
- **L439** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L440** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。

### Lines 441-460 / 第 441-460 行

````python

            shard_placement[shard_md] = (shard_rank, rank_sizes[shard_rank])
            rank_sizes[shard_rank] += shard_size(shard_md)
            max_rank_size = max(max_rank_size, rank_sizes[shard_rank])

        gather_list: list[torch.Tensor] | None
        if rank == dst:
            if out is None:
                raise AssertionError
            if enforce_dtype:
                # enforce_dtype is deprecated.  Do it for backward compatibility.
                dtype = out.dtype
            # TODO make it as a view of out tensor
            gather_list = [
                torch.empty((max_rank_size,), device=out.device, dtype=dtype)
                for _ in range(world_size)
            ]
        else:
            gather_list = None

````

- **L441** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L442** EN: Assigns or updates `shard_placement[shard_md]`. | CN: 对 `shard_placement[shard_md]` 进行赋值或更新。
- **L443** EN: Continues the implementation inside function `gather`. | CN: 继续说明函数 `gather` 内部的实现。
- **L444** EN: Assigns or updates `max_rank_size`. | CN: 对 `max_rank_size` 进行赋值或更新。
- **L445** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L446** EN: Continues the implementation inside function `gather`. | CN: 继续说明函数 `gather` 内部的实现。
- **L447** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L448** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L449** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L450** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L451** EN: Keeps the inline comment or directive: enforce_dtype is deprecated.  Do it for backward compatibility. | CN: 保留这一行注释或指令：enforce_dtype is deprecated.  Do it for backward compatibility.
- **L452** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L453** EN: Keeps the inline comment or directive: TODO make it as a view of out tensor | CN: 保留这一行注释或指令：TODO make it as a view of out tensor
- **L454** EN: Assigns or updates `gather_list`. | CN: 对 `gather_list` 进行赋值或更新。
- **L455** EN: Calls `torch.empty` as part of the current workflow. | CN: 在当前流程中调用 `torch.empty`。
- **L456** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L457** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L458** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L459** EN: Assigns or updates `gather_list`. | CN: 对 `gather_list` 进行赋值或更新。
- **L460** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 461-480 / 第 461-480 行

````python
        with torch.no_grad():
            if enforce_dtype and len(local_shards) > 0:
                # enforce_dtype is deprecated.  Do it for backward compatibility.
                dtype = local_shards[0].tensor.dtype
            data = torch.empty(
                max_rank_size, device=self._get_preferred_device(), dtype=dtype
            )

            for shard in local_shards:
                src = shard.tensor.flatten()
                if src.nelement() == 0:
                    warnings.warn(
                        "Gathering a tensor with zero elements on rank " + str(rank),
                        stacklevel=2,
                    )
                    continue
                shard_offset = shard_placement[shard.metadata][1]
                data[shard_offset : shard_offset + src.numel()].copy_(src)

        dist.gather(
````

- **L461** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L462** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L463** EN: Keeps the inline comment or directive: enforce_dtype is deprecated.  Do it for backward compatibility. | CN: 保留这一行注释或指令：enforce_dtype is deprecated.  Do it for backward compatibility.
- **L464** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L465** EN: Assigns or updates `data`. | CN: 对 `data` 进行赋值或更新。
- **L466** EN: Assigns or updates `max_rank_size, device`. | CN: 对 `max_rank_size, device` 进行赋值或更新。
- **L467** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L468** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L469** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L470** EN: Assigns or updates `src`. | CN: 对 `src` 进行赋值或更新。
- **L471** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L472** EN: Calls `warnings.warn` as part of the current workflow. | CN: 在当前流程中调用 `warnings.warn`。
- **L473** EN: Continues the implementation inside function `gather`. | CN: 继续说明函数 `gather` 内部的实现。
- **L474** EN: Assigns or updates `stacklevel`. | CN: 对 `stacklevel` 进行赋值或更新。
- **L475** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L476** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L477** EN: Assigns or updates `shard_offset`. | CN: 对 `shard_offset` 进行赋值或更新。
- **L478** EN: Continues the implementation inside function `gather`. | CN: 继续说明函数 `gather` 内部的实现。
- **L479** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L480** EN: Calls `dist.gather` as part of the current workflow. | CN: 在当前流程中调用 `dist.gather`。

### Lines 481-500 / 第 481-500 行

````python
            tensor=data,
            gather_list=gather_list,
            dst=dst,
            group=self._process_group,
        )
        if rank != dst:
            return
        # In _validate_output_tensor_for_gather, we raise if out == None and rank == dst
        out = cast(torch.Tensor, out)
        if gather_list is None:
            raise AssertionError

        full_size = self.metadata().size
        dims = len(full_size)
        for shard_md in self.metadata().shards_metadata:
            rank, rank_offset = shard_placement[shard_md]
            tensor = gather_list[rank]
            tensor = tensor[rank_offset : rank_offset + shard_size(shard_md)]
            tensor = tensor.view(shard_md.shard_sizes)

````

- **L481** EN: Assigns or updates `tensor`. | CN: 对 `tensor` 进行赋值或更新。
- **L482** EN: Assigns or updates `gather_list`. | CN: 对 `gather_list` 进行赋值或更新。
- **L483** EN: Assigns or updates `dst`. | CN: 对 `dst` 进行赋值或更新。
- **L484** EN: Assigns or updates `group`. | CN: 对 `group` 进行赋值或更新。
- **L485** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L486** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L487** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L488** EN: Keeps the inline comment or directive: In _validate_output_tensor_for_gather, we raise if out == None and rank == dst | CN: 保留这一行注释或指令：In _validate_output_tensor_for_gather, we raise if out == None and rank == dst
- **L489** EN: Assigns or updates `out`. | CN: 对 `out` 进行赋值或更新。
- **L490** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L491** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L492** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L493** EN: Assigns or updates `full_size`. | CN: 对 `full_size` 进行赋值或更新。
- **L494** EN: Assigns or updates `dims`. | CN: 对 `dims` 进行赋值或更新。
- **L495** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L496** EN: Assigns or updates `rank, rank_offset`. | CN: 对 `rank, rank_offset` 进行赋值或更新。
- **L497** EN: Assigns or updates `tensor`. | CN: 对 `tensor` 进行赋值或更新。
- **L498** EN: Assigns or updates `tensor`. | CN: 对 `tensor` 进行赋值或更新。
- **L499** EN: Assigns or updates `tensor`. | CN: 对 `tensor` 进行赋值或更新。
- **L500** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 501-520 / 第 501-520 行

````python
            out_narrow_view = out
            for dim in range(dims):
                out_narrow_view = out_narrow_view.narrow(
                    dim,
                    shard_md.shard_offsets[dim],
                    shard_md.shard_sizes[dim],
                )

            out_narrow_view.copy_(tensor)

    def cpu(
        self, memory_format=torch.preserve_format, process_group=None
    ) -> ShardedTensor:
        """
        Returns a copy of this object in CPU memory.

        If this ShardedTensor is already on CPU memory, then no copy is
        performed and original object is returned.

        .. note:: When moving a ShardedTensor from GPU to CPU, the ShardedTensor might
````

- **L501** EN: Assigns or updates `out_narrow_view`. | CN: 对 `out_narrow_view` 进行赋值或更新。
- **L502** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L503** EN: Assigns or updates `out_narrow_view`. | CN: 对 `out_narrow_view` 进行赋值或更新。
- **L504** EN: Continues the implementation inside function `gather`. | CN: 继续说明函数 `gather` 内部的实现。
- **L505** EN: Continues the implementation inside function `gather`. | CN: 继续说明函数 `gather` 内部的实现。
- **L506** EN: Continues the implementation inside function `gather`. | CN: 继续说明函数 `gather` 内部的实现。
- **L507** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L508** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L509** EN: Calls `out_narrow_view.copy_` as part of the current workflow. | CN: 在当前流程中调用 `out_narrow_view.copy_`。
- **L510** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L511** EN: Defines function `cpu`. | CN: 定义函数 `cpu`。
- **L512** EN: Assigns or updates `self, memory_format`. | CN: 对 `self, memory_format` 进行赋值或更新。
- **L513** EN: Continues the implementation inside function `cpu`. | CN: 继续说明函数 `cpu` 内部的实现。
- **L514** EN: Starts the docstring for the function cpu. | CN: 开始定义 function cpu 的文档字符串。
- **L515** EN: Continues the docstring text for the function cpu. | CN: 继续补充 function cpu 的文档字符串内容。
- **L516** EN: Continues the docstring text for the function cpu. | CN: 继续补充 function cpu 的文档字符串内容。
- **L517** EN: Continues the docstring text for the function cpu. | CN: 继续补充 function cpu 的文档字符串内容。
- **L518** EN: Continues the docstring text for the function cpu. | CN: 继续补充 function cpu 的文档字符串内容。
- **L519** EN: Continues the docstring text for the function cpu. | CN: 继续补充 function cpu 的文档字符串内容。
- **L520** EN: Continues the docstring text for the function cpu. | CN: 继续补充 function cpu 的文档字符串内容。

### Lines 521-540 / 第 521-540 行

````python
            need to be managed by a different type of ProcessGroup(i.e. ProcessGroupGloo),
            it is the user's responsibility to explicitly pass in a new process_group that
            is compatible with CPU.
        """
        # TODO: make this a __torch_function__ op once ShardedTensor becomes a
        # torch.Tensor subclass, see https://github.com/pytorch/pytorch/issues/75402
        if (
            memory_format != torch.preserve_format
            and memory_format != torch.contiguous_format
        ):
            raise RuntimeError(
                "Only `torch.contiguous_format` or "
                "`torch.preserve_format` is supported!"
            )
        all_on_cpu = True
        for meta in self.metadata().shards_metadata:
            all_on_cpu &= meta.placement.device().type == "cpu"  # type: ignore[union-attr]

        # if every shard is already on CPU, return the original object
        if all_on_cpu:
````

- **L521** EN: Continues the docstring text for the function cpu. | CN: 继续补充 function cpu 的文档字符串内容。
- **L522** EN: Continues the docstring text for the function cpu. | CN: 继续补充 function cpu 的文档字符串内容。
- **L523** EN: Continues the docstring text for the function cpu. | CN: 继续补充 function cpu 的文档字符串内容。
- **L524** EN: Closes the docstring for the function cpu. | CN: 结束 function cpu 的文档字符串。
- **L525** EN: Keeps the inline comment or directive: TODO: make this a __torch_function__ op once ShardedTensor becomes a | CN: 保留这一行注释或指令：TODO: make this a __torch_function__ op once ShardedTensor becomes a
- **L526** EN: Keeps the inline comment or directive: torch.Tensor subclass, see https://github.com/pytorch/pytorch/issues/75402 | CN: 保留这一行注释或指令：torch.Tensor subclass, see https://github.com/pytorch/pytorch/issues/75402
- **L527** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L528** EN: Continues the implementation inside function `cpu`. | CN: 继续说明函数 `cpu` 内部的实现。
- **L529** EN: Continues the implementation inside function `cpu`. | CN: 继续说明函数 `cpu` 内部的实现。
- **L530** EN: Continues the implementation inside function `cpu`. | CN: 继续说明函数 `cpu` 内部的实现。
- **L531** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L532** EN: Continues the implementation inside function `cpu`. | CN: 继续说明函数 `cpu` 内部的实现。
- **L533** EN: Continues the implementation inside function `cpu`. | CN: 继续说明函数 `cpu` 内部的实现。
- **L534** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L535** EN: Assigns or updates `all_on_cpu`. | CN: 对 `all_on_cpu` 进行赋值或更新。
- **L536** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L537** EN: Continues the implementation inside function `cpu`. | CN: 继续说明函数 `cpu` 内部的实现。
- **L538** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L539** EN: Keeps the inline comment or directive: if every shard is already on CPU, return the original object | CN: 保留这一行注释或指令：if every shard is already on CPU, return the original object
- **L540** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 541-560 / 第 541-560 行

````python
            return self

        # if not, returns a copy of this object on CPU
        list_shards: list[Shard] = []
        # move all local shards to cpu, and change metadata
        for shard in self._local_shards:
            cpu_tensor = shard.tensor.cpu(memory_format=memory_format)  # type: ignore[call-arg]
            metadata = copy.deepcopy(shard.metadata)
            metadata.placement._device = torch.device("cpu")  # type: ignore[union-attr]
            list_shards.append(Shard(cpu_tensor, metadata))

        st_meta = copy.deepcopy(self.metadata())
        for meta in st_meta.shards_metadata:
            if meta.placement.device().type != "cpu":  # type: ignore[union-attr]
                meta.placement._device = torch.device("cpu")  # type: ignore[union-attr]

        pg = self._process_group if process_group is None else process_group
        st_cpu = ShardedTensor._init_from_local_shards_and_global_metadata(
            list_shards,
            sharded_tensor_metadata=st_meta,
````

- **L541** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L542** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L543** EN: Keeps the inline comment or directive: if not, returns a copy of this object on CPU | CN: 保留这一行注释或指令：if not, returns a copy of this object on CPU
- **L544** EN: Assigns or updates `list_shards`. | CN: 对 `list_shards` 进行赋值或更新。
- **L545** EN: Keeps the inline comment or directive: move all local shards to cpu, and change metadata | CN: 保留这一行注释或指令：move all local shards to cpu, and change metadata
- **L546** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L547** EN: Assigns or updates `cpu_tensor`. | CN: 对 `cpu_tensor` 进行赋值或更新。
- **L548** EN: Assigns or updates `metadata`. | CN: 对 `metadata` 进行赋值或更新。
- **L549** EN: Assigns or updates `metadata.placement._device`. | CN: 对 `metadata.placement._device` 进行赋值或更新。
- **L550** EN: Calls `list_shards.append` as part of the current workflow. | CN: 在当前流程中调用 `list_shards.append`。
- **L551** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L552** EN: Assigns or updates `st_meta`. | CN: 对 `st_meta` 进行赋值或更新。
- **L553** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L554** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L555** EN: Assigns or updates `meta.placement._device`. | CN: 对 `meta.placement._device` 进行赋值或更新。
- **L556** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L557** EN: Assigns or updates `pg`. | CN: 对 `pg` 进行赋值或更新。
- **L558** EN: Assigns or updates `st_cpu`. | CN: 对 `st_cpu` 进行赋值或更新。
- **L559** EN: Continues the implementation inside function `cpu`. | CN: 继续说明函数 `cpu` 内部的实现。
- **L560** EN: Assigns or updates `sharded_tensor_metadata`. | CN: 对 `sharded_tensor_metadata` 进行赋值或更新。

### Lines 561-580 / 第 561-580 行

````python
            process_group=pg,
            init_rrefs=self._init_rrefs,
        )
        return st_cpu

    def cuda(
        self,
        device=None,
        non_blocking=False,
        memory_format=torch.preserve_format,
        process_group=None,
    ) -> ShardedTensor:
        """
        Returns a copy of this object in CUDA memory, if the original ShardedTensor
        is on CPU, we will move the local shard to the current GPU device of each
        process in a SPMD fashion.
        If this ShardedTensor is already on CUDA memory and local shards on each rank are
        already on current device, we still returns a new ShardedTensor object with new
        metadata, but no underlying data movements are performed.
        .. note:: When moving a ShardedTensor from CPU to GPU, the ShardedTensor might
````

- **L561** EN: Assigns or updates `process_group`. | CN: 对 `process_group` 进行赋值或更新。
- **L562** EN: Assigns or updates `init_rrefs`. | CN: 对 `init_rrefs` 进行赋值或更新。
- **L563** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L564** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L565** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L566** EN: Defines function `cuda`. | CN: 定义函数 `cuda`。
- **L567** EN: Continues the implementation inside function `cuda`. | CN: 继续说明函数 `cuda` 内部的实现。
- **L568** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L569** EN: Assigns or updates `non_blocking`. | CN: 对 `non_blocking` 进行赋值或更新。
- **L570** EN: Assigns or updates `memory_format`. | CN: 对 `memory_format` 进行赋值或更新。
- **L571** EN: Assigns or updates `process_group`. | CN: 对 `process_group` 进行赋值或更新。
- **L572** EN: Continues the implementation inside function `cuda`. | CN: 继续说明函数 `cuda` 内部的实现。
- **L573** EN: Starts the docstring for the function cuda. | CN: 开始定义 function cuda 的文档字符串。
- **L574** EN: Continues the docstring text for the function cuda. | CN: 继续补充 function cuda 的文档字符串内容。
- **L575** EN: Continues the docstring text for the function cuda. | CN: 继续补充 function cuda 的文档字符串内容。
- **L576** EN: Continues the docstring text for the function cuda. | CN: 继续补充 function cuda 的文档字符串内容。
- **L577** EN: Continues the docstring text for the function cuda. | CN: 继续补充 function cuda 的文档字符串内容。
- **L578** EN: Continues the docstring text for the function cuda. | CN: 继续补充 function cuda 的文档字符串内容。
- **L579** EN: Continues the docstring text for the function cuda. | CN: 继续补充 function cuda 的文档字符串内容。
- **L580** EN: Continues the docstring text for the function cuda. | CN: 继续补充 function cuda 的文档字符串内容。

### Lines 581-600 / 第 581-600 行

````python
            need to be managed by a different type of ProcessGroup(i.e. ProcessGroupNCCL),
            it is the user's responsibility to explicitly pass in a new process_group that
            is compatible with GPU.
        """
        if (
            memory_format != torch.preserve_format
            and memory_format != torch.contiguous_format
        ):
            raise RuntimeError(
                "Only `torch.contiguous_format` or "
                "`torch.preserve_format` is supported!"
            )

        if device is not None:
            device = torch.device(device) if isinstance(device, str) else device
            if not (
                isinstance(device, torch.device)
                and device.index == torch.cuda.current_device()
            ):
                raise AssertionError(
````

- **L581** EN: Continues the docstring text for the function cuda. | CN: 继续补充 function cuda 的文档字符串内容。
- **L582** EN: Continues the docstring text for the function cuda. | CN: 继续补充 function cuda 的文档字符串内容。
- **L583** EN: Continues the docstring text for the function cuda. | CN: 继续补充 function cuda 的文档字符串内容。
- **L584** EN: Closes the docstring for the function cuda. | CN: 结束 function cuda 的文档字符串。
- **L585** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L586** EN: Continues the implementation inside function `cuda`. | CN: 继续说明函数 `cuda` 内部的实现。
- **L587** EN: Continues the implementation inside function `cuda`. | CN: 继续说明函数 `cuda` 内部的实现。
- **L588** EN: Continues the implementation inside function `cuda`. | CN: 继续说明函数 `cuda` 内部的实现。
- **L589** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L590** EN: Continues the implementation inside function `cuda`. | CN: 继续说明函数 `cuda` 内部的实现。
- **L591** EN: Continues the implementation inside function `cuda`. | CN: 继续说明函数 `cuda` 内部的实现。
- **L592** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L593** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L594** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L595** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L596** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L597** EN: Calls `isinstance` as part of the current workflow. | CN: 在当前流程中调用 `isinstance`。
- **L598** EN: Continues the implementation inside function `cuda`. | CN: 继续说明函数 `cuda` 内部的实现。
- **L599** EN: Continues the implementation inside function `cuda`. | CN: 继续说明函数 `cuda` 内部的实现。
- **L600** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。

### Lines 601-620 / 第 601-620 行

````python
                    """Only device without device id (e.g. "cpu" or "cuda") is expected for ShardedTensor!"""
                )

        current_device = torch.device(torch.cuda.current_device())
        # returns a copy of ShardedTensor on CUDA current device
        list_shards: list[Shard] = []
        # move all local shards to current device, and change metadata
        # if local shards already on the current device, there's no
        # real data movement, only the metadata are copied.
        for shard in self._local_shards:
            cuda_tensor = shard.tensor.cuda(
                device=current_device,
                non_blocking=non_blocking,
                memory_format=memory_format,
            )  # type: ignore[call-arg]
            metadata = copy.deepcopy(shard.metadata)
            metadata.placement._device = current_device  # type: ignore[union-attr]

            list_shards.append(Shard(cuda_tensor, metadata))

````

- **L601** EN: Continues the implementation inside function `cuda`. | CN: 继续说明函数 `cuda` 内部的实现。
- **L602** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L603** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L604** EN: Assigns or updates `current_device`. | CN: 对 `current_device` 进行赋值或更新。
- **L605** EN: Keeps the inline comment or directive: returns a copy of ShardedTensor on CUDA current device | CN: 保留这一行注释或指令：returns a copy of ShardedTensor on CUDA current device
- **L606** EN: Assigns or updates `list_shards`. | CN: 对 `list_shards` 进行赋值或更新。
- **L607** EN: Keeps the inline comment or directive: move all local shards to current device, and change metadata | CN: 保留这一行注释或指令：move all local shards to current device, and change metadata
- **L608** EN: Keeps the inline comment or directive: if local shards already on the current device, there's no | CN: 保留这一行注释或指令：if local shards already on the current device, there's no
- **L609** EN: Keeps the inline comment or directive: real data movement, only the metadata are copied. | CN: 保留这一行注释或指令：real data movement, only the metadata are copied.
- **L610** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L611** EN: Assigns or updates `cuda_tensor`. | CN: 对 `cuda_tensor` 进行赋值或更新。
- **L612** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L613** EN: Assigns or updates `non_blocking`. | CN: 对 `non_blocking` 进行赋值或更新。
- **L614** EN: Assigns or updates `memory_format`. | CN: 对 `memory_format` 进行赋值或更新。
- **L615** EN: Continues the implementation inside function `cuda`. | CN: 继续说明函数 `cuda` 内部的实现。
- **L616** EN: Assigns or updates `metadata`. | CN: 对 `metadata` 进行赋值或更新。
- **L617** EN: Assigns or updates `metadata.placement._device`. | CN: 对 `metadata.placement._device` 进行赋值或更新。
- **L618** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L619** EN: Calls `list_shards.append` as part of the current workflow. | CN: 在当前流程中调用 `list_shards.append`。
- **L620** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 621-640 / 第 621-640 行

````python
        st_meta = copy.deepcopy(self.metadata())
        for meta in st_meta.shards_metadata:
            if meta.placement.device().type != "cuda":  # type: ignore[union-attr]
                meta.placement._device = current_device  # type: ignore[union-attr]

        pg = self._process_group if process_group is None else process_group
        # we need to use `init_from_local_shards` to communicate between ranks
        # and update the sharding spec/shards metadata.
        st_cuda = ShardedTensor._init_from_local_shards_and_global_metadata(
            list_shards,
            sharded_tensor_metadata=st_meta,
            process_group=pg,
            init_rrefs=self._init_rrefs,
        )
        return st_cuda

    def to(self, *args, **kwargs) -> ShardedTensor:
        current_device: torch.device
        if self._local_shards:
            current_device = self._local_shards[0].tensor.device
````

- **L621** EN: Assigns or updates `st_meta`. | CN: 对 `st_meta` 进行赋值或更新。
- **L622** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L623** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L624** EN: Assigns or updates `meta.placement._device`. | CN: 对 `meta.placement._device` 进行赋值或更新。
- **L625** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L626** EN: Assigns or updates `pg`. | CN: 对 `pg` 进行赋值或更新。
- **L627** EN: Keeps the inline comment or directive: we need to use `init_from_local_shards` to communicate between ranks | CN: 保留这一行注释或指令：we need to use `init_from_local_shards` to communicate between ranks
- **L628** EN: Keeps the inline comment or directive: and update the sharding spec/shards metadata. | CN: 保留这一行注释或指令：and update the sharding spec/shards metadata.
- **L629** EN: Assigns or updates `st_cuda`. | CN: 对 `st_cuda` 进行赋值或更新。
- **L630** EN: Continues the implementation inside function `cuda`. | CN: 继续说明函数 `cuda` 内部的实现。
- **L631** EN: Assigns or updates `sharded_tensor_metadata`. | CN: 对 `sharded_tensor_metadata` 进行赋值或更新。
- **L632** EN: Assigns or updates `process_group`. | CN: 对 `process_group` 进行赋值或更新。
- **L633** EN: Assigns or updates `init_rrefs`. | CN: 对 `init_rrefs` 进行赋值或更新。
- **L634** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L635** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L636** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L637** EN: Defines function `to`. | CN: 定义函数 `to`。
- **L638** EN: Continues the implementation inside function `to`. | CN: 继续说明函数 `to` 内部的实现。
- **L639** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L640** EN: Assigns or updates `current_device`. | CN: 对 `current_device` 进行赋值或更新。

### Lines 641-660 / 第 641-660 行

````python
        elif self._process_group._get_backend_name() == "gloo":
            current_device = torch.device("cpu")
        else:
            current_device = torch.device(torch.cuda.current_device())
        current_dtype = self.dtype
        device_to = current_device
        dtype_to = current_dtype
        if len(args) == 1:
            if isinstance(args[0], torch.dtype):
                dtype_to = args[0]
            elif isinstance(args[0], torch.device):
                device_to = args[0]
            elif isinstance(args[0], (str, int)):
                device_to = torch.device(args[0])
            elif isinstance(args[0], torch.Tensor):
                dtype_to = args[0].dtype
                device_to = args[0].device
            else:
                raise RuntimeError(f"ShardedTensor.to() have wrong arguments: {args}")
        elif len(args) == 2:
````

- **L641** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L642** EN: Assigns or updates `current_device`. | CN: 对 `current_device` 进行赋值或更新。
- **L643** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L644** EN: Assigns or updates `current_device`. | CN: 对 `current_device` 进行赋值或更新。
- **L645** EN: Assigns or updates `current_dtype`. | CN: 对 `current_dtype` 进行赋值或更新。
- **L646** EN: Assigns or updates `device_to`. | CN: 对 `device_to` 进行赋值或更新。
- **L647** EN: Assigns or updates `dtype_to`. | CN: 对 `dtype_to` 进行赋值或更新。
- **L648** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L649** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L650** EN: Assigns or updates `dtype_to`. | CN: 对 `dtype_to` 进行赋值或更新。
- **L651** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L652** EN: Assigns or updates `device_to`. | CN: 对 `device_to` 进行赋值或更新。
- **L653** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L654** EN: Assigns or updates `device_to`. | CN: 对 `device_to` 进行赋值或更新。
- **L655** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L656** EN: Assigns or updates `dtype_to`. | CN: 对 `dtype_to` 进行赋值或更新。
- **L657** EN: Assigns or updates `device_to`. | CN: 对 `device_to` 进行赋值或更新。
- **L658** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L659** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L660** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。

### Lines 661-680 / 第 661-680 行

````python
            device_to, dtype_to = args
        else:
            dtype_to = kwargs.get("dtype", current_dtype)
            device_to = kwargs.get("device", current_device)

        device_to = (
            torch.device(device_to) if isinstance(device_to, (str, int)) else device_to
        )

        if device_to.type == "cuda":
            # if device_to set to cuda, set to current device even
            # if user specify the device index.
            current_idx = torch.cuda.current_device()
            if device_to.index != current_idx:
                warnings.warn(
                    "ShardedTensor.to only move tensor to its current device"
                    "If you want to put to different device, use `reshard` instead.",
                    stacklevel=2,
                )
            device_to = torch.device(current_idx)
````

- **L661** EN: Assigns or updates `device_to, dtype_to`. | CN: 对 `device_to, dtype_to` 进行赋值或更新。
- **L662** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L663** EN: Assigns or updates `dtype_to`. | CN: 对 `dtype_to` 进行赋值或更新。
- **L664** EN: Assigns or updates `device_to`. | CN: 对 `device_to` 进行赋值或更新。
- **L665** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L666** EN: Assigns or updates `device_to`. | CN: 对 `device_to` 进行赋值或更新。
- **L667** EN: Calls `torch.device` as part of the current workflow. | CN: 在当前流程中调用 `torch.device`。
- **L668** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L669** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L670** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L671** EN: Keeps the inline comment or directive: if device_to set to cuda, set to current device even | CN: 保留这一行注释或指令：if device_to set to cuda, set to current device even
- **L672** EN: Keeps the inline comment or directive: if user specify the device index. | CN: 保留这一行注释或指令：if user specify the device index.
- **L673** EN: Assigns or updates `current_idx`. | CN: 对 `current_idx` 进行赋值或更新。
- **L674** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L675** EN: Calls `warnings.warn` as part of the current workflow. | CN: 在当前流程中调用 `warnings.warn`。
- **L676** EN: Continues the implementation inside function `to`. | CN: 继续说明函数 `to` 内部的实现。
- **L677** EN: Continues the implementation inside function `to`. | CN: 继续说明函数 `to` 内部的实现。
- **L678** EN: Assigns or updates `stacklevel`. | CN: 对 `stacklevel` 进行赋值或更新。
- **L679** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L680** EN: Assigns or updates `device_to`. | CN: 对 `device_to` 进行赋值或更新。

### Lines 681-700 / 第 681-700 行

````python

        copy_tensor = kwargs.get("copy", False)
        non_blocking = kwargs.get("non_blocking", False)
        memory_format = kwargs.get("memory_format", torch.preserve_format)
        process_group = kwargs.get("process_group")

        if (
            not copy_tensor
            and dtype_to == current_dtype
            and device_to == current_device
        ):
            # already have correct dtype and device, return itself
            return self

        # returns a copy of ShardedTensor on CUDA current device
        list_shards: list[Shard] = []

        for shard in self._local_shards:
            new_tensor = shard.tensor.to(  # type: ignore[call-overload]
                device=device_to,
````

- **L681** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L682** EN: Assigns or updates `copy_tensor`. | CN: 对 `copy_tensor` 进行赋值或更新。
- **L683** EN: Assigns or updates `non_blocking`. | CN: 对 `non_blocking` 进行赋值或更新。
- **L684** EN: Assigns or updates `memory_format`. | CN: 对 `memory_format` 进行赋值或更新。
- **L685** EN: Assigns or updates `process_group`. | CN: 对 `process_group` 进行赋值或更新。
- **L686** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L687** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L688** EN: Continues the implementation inside function `to`. | CN: 继续说明函数 `to` 内部的实现。
- **L689** EN: Continues the implementation inside function `to`. | CN: 继续说明函数 `to` 内部的实现。
- **L690** EN: Continues the implementation inside function `to`. | CN: 继续说明函数 `to` 内部的实现。
- **L691** EN: Continues the implementation inside function `to`. | CN: 继续说明函数 `to` 内部的实现。
- **L692** EN: Keeps the inline comment or directive: already have correct dtype and device, return itself | CN: 保留这一行注释或指令：already have correct dtype and device, return itself
- **L693** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L694** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L695** EN: Keeps the inline comment or directive: returns a copy of ShardedTensor on CUDA current device | CN: 保留这一行注释或指令：returns a copy of ShardedTensor on CUDA current device
- **L696** EN: Assigns or updates `list_shards`. | CN: 对 `list_shards` 进行赋值或更新。
- **L697** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L698** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L699** EN: Assigns or updates `new_tensor`. | CN: 对 `new_tensor` 进行赋值或更新。
- **L700** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。

### Lines 701-720 / 第 701-720 行

````python
                dtype=dtype_to,
                non_blocking=non_blocking,
                copy=copy_tensor,
                memory_format=memory_format,
            )
            metadata = copy.deepcopy(shard.metadata)
            if metadata.placement is not None:
                metadata.placement._device = device_to
            list_shards.append(Shard(new_tensor, metadata))

        # update metadata
        st_meta = copy.deepcopy(self.metadata())
        st_meta.tensor_properties.dtype = dtype_to
        for meta in st_meta.shards_metadata:
            meta.placement._device = device_to  # type: ignore[union-attr]

        pg = self._process_group if process_group is None else process_group
        # we need to use `init_from_local_shards` to communicate between ranks
        # and update the sharding spec/shards metadata.
        st_to = ShardedTensor._init_from_local_shards_and_global_metadata(
````

- **L701** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L702** EN: Assigns or updates `non_blocking`. | CN: 对 `non_blocking` 进行赋值或更新。
- **L703** EN: Assigns or updates `copy`. | CN: 对 `copy` 进行赋值或更新。
- **L704** EN: Assigns or updates `memory_format`. | CN: 对 `memory_format` 进行赋值或更新。
- **L705** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L706** EN: Assigns or updates `metadata`. | CN: 对 `metadata` 进行赋值或更新。
- **L707** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L708** EN: Assigns or updates `metadata.placement._device`. | CN: 对 `metadata.placement._device` 进行赋值或更新。
- **L709** EN: Calls `list_shards.append` as part of the current workflow. | CN: 在当前流程中调用 `list_shards.append`。
- **L710** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L711** EN: Keeps the inline comment or directive: update metadata | CN: 保留这一行注释或指令：update metadata
- **L712** EN: Assigns or updates `st_meta`. | CN: 对 `st_meta` 进行赋值或更新。
- **L713** EN: Assigns or updates `st_meta.tensor_properties.dtype`. | CN: 对 `st_meta.tensor_properties.dtype` 进行赋值或更新。
- **L714** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L715** EN: Assigns or updates `meta.placement._device`. | CN: 对 `meta.placement._device` 进行赋值或更新。
- **L716** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L717** EN: Assigns or updates `pg`. | CN: 对 `pg` 进行赋值或更新。
- **L718** EN: Keeps the inline comment or directive: we need to use `init_from_local_shards` to communicate between ranks | CN: 保留这一行注释或指令：we need to use `init_from_local_shards` to communicate between ranks
- **L719** EN: Keeps the inline comment or directive: and update the sharding spec/shards metadata. | CN: 保留这一行注释或指令：and update the sharding spec/shards metadata.
- **L720** EN: Assigns or updates `st_to`. | CN: 对 `st_to` 进行赋值或更新。

### Lines 721-740 / 第 721-740 行

````python
            list_shards,
            sharded_tensor_metadata=st_meta,
            process_group=pg,
            init_rrefs=self._init_rrefs,
        )
        return st_to

    @classmethod
    def _normalize_pg(
        cls, process_group: dist.ProcessGroup | None
    ) -> dist.ProcessGroup:
        if process_group is not None:
            return process_group
        return distributed_c10d._get_default_group()

    @classmethod
    def _init_from_local_shards(
        cls,
        local_shards: list[Shard],
        *global_size,
````

- **L721** EN: Continues the implementation inside function `to`. | CN: 继续说明函数 `to` 内部的实现。
- **L722** EN: Assigns or updates `sharded_tensor_metadata`. | CN: 对 `sharded_tensor_metadata` 进行赋值或更新。
- **L723** EN: Assigns or updates `process_group`. | CN: 对 `process_group` 进行赋值或更新。
- **L724** EN: Assigns or updates `init_rrefs`. | CN: 对 `init_rrefs` 进行赋值或更新。
- **L725** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L726** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L727** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L728** EN: Applies decorator `classmethod` to the following definition. | CN: 将装饰器 `classmethod` 应用于后续定义。
- **L729** EN: Defines function `_normalize_pg`. | CN: 定义函数 `_normalize_pg`。
- **L730** EN: Continues the implementation inside function `_normalize_pg`. | CN: 继续说明函数 `_normalize_pg` 内部的实现。
- **L731** EN: Continues the implementation inside function `_normalize_pg`. | CN: 继续说明函数 `_normalize_pg` 内部的实现。
- **L732** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L733** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L734** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L735** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L736** EN: Applies decorator `classmethod` to the following definition. | CN: 将装饰器 `classmethod` 应用于后续定义。
- **L737** EN: Defines function `_init_from_local_shards`. | CN: 定义函数 `_init_from_local_shards`。
- **L738** EN: Continues the implementation inside function `_init_from_local_shards`. | CN: 继续说明函数 `_init_from_local_shards` 内部的实现。
- **L739** EN: Continues the implementation inside function `_init_from_local_shards`. | CN: 继续说明函数 `_init_from_local_shards` 内部的实现。
- **L740** EN: Continues the implementation inside function `_init_from_local_shards`. | CN: 继续说明函数 `_init_from_local_shards` 内部的实现。

### Lines 741-760 / 第 741-760 行

````python
        process_group=None,
        init_rrefs=False,
    ):
        # recalc metadata handles special ST creation cases like each rank only has tensor available
        # caller need to provide None on the unknown dimension of the global size
        # We will change None into zeros and go through the same amount of checks as before to create ST
        # and use all_gather to calculate the offsets and global size for metadata
        # It is compatible with the current use case since, conventionally we don't pass None as global size
        # Therefore the old path won't trigger the new feature
        recalc_metadata = False
        for dim in global_size:
            if dim is None:
                recalc_metadata = True
        if recalc_metadata:
            global_size = tuple(
                0 if dim_size is None else dim_size for dim_size in global_size
            )
        # STEP 1: Validate the Shardmetadatas locally
        process_group = cls._normalize_pg(process_group)
        current_rank = dist.get_rank()  # intentional to get global rank
````

- **L741** EN: Assigns or updates `process_group`. | CN: 对 `process_group` 进行赋值或更新。
- **L742** EN: Assigns or updates `init_rrefs`. | CN: 对 `init_rrefs` 进行赋值或更新。
- **L743** EN: Continues the implementation inside function `_init_from_local_shards`. | CN: 继续说明函数 `_init_from_local_shards` 内部的实现。
- **L744** EN: Keeps the inline comment or directive: recalc metadata handles special ST creation cases like each rank only has tensor | CN: 保留这一行注释或指令：recalc metadata handles special ST creation cases like each rank only has tensor
- **L745** EN: Keeps the inline comment or directive: caller need to provide None on the unknown dimension of the global size | CN: 保留这一行注释或指令：caller need to provide None on the unknown dimension of the global size
- **L746** EN: Keeps the inline comment or directive: We will change None into zeros and go through the same amount of checks as befor | CN: 保留这一行注释或指令：We will change None into zeros and go through the same amount of checks as befor
- **L747** EN: Keeps the inline comment or directive: and use all_gather to calculate the offsets and global size for metadata | CN: 保留这一行注释或指令：and use all_gather to calculate the offsets and global size for metadata
- **L748** EN: Keeps the inline comment or directive: It is compatible with the current use case since, conventionally we don't pass N | CN: 保留这一行注释或指令：It is compatible with the current use case since, conventionally we don't pass N
- **L749** EN: Keeps the inline comment or directive: Therefore the old path won't trigger the new feature | CN: 保留这一行注释或指令：Therefore the old path won't trigger the new feature
- **L750** EN: Assigns or updates `recalc_metadata`. | CN: 对 `recalc_metadata` 进行赋值或更新。
- **L751** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L752** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L753** EN: Assigns or updates `recalc_metadata`. | CN: 对 `recalc_metadata` 进行赋值或更新。
- **L754** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L755** EN: Assigns or updates `global_size`. | CN: 对 `global_size` 进行赋值或更新。
- **L756** EN: Continues the implementation inside function `_init_from_local_shards`. | CN: 继续说明函数 `_init_from_local_shards` 内部的实现。
- **L757** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L758** EN: Keeps the inline comment or directive: STEP 1: Validate the Shardmetadatas locally | CN: 保留这一行注释或指令：STEP 1: Validate the Shardmetadatas locally
- **L759** EN: Assigns or updates `process_group`. | CN: 对 `process_group` 进行赋值或更新。
- **L760** EN: Assigns or updates `current_rank`. | CN: 对 `current_rank` 进行赋值或更新。

### Lines 761-780 / 第 761-780 行

````python
        world_size = dist.get_world_size(process_group)

        local_sharded_tensor_metadata: ShardedTensorMetadata | None = None
        global_tensor_size = _flatten_tensor_size(global_size)

        if len(local_shards) > 0:
            local_sharded_tensor_metadata = build_metadata_from_local_shards(
                local_shards, global_tensor_size, current_rank, process_group
            )

        # STEP 2. Validate metadata across ranks, and build a global sharded tensor
        # metadata by gathering local ShardedTensorMetadata
        gathered_metadatas: list[ShardedTensorMetadata | None] = []
        if world_size > 1:
            gathered_metadatas = [None for _ in range(world_size)]

            dist.all_gather_object(
                gathered_metadatas, local_sharded_tensor_metadata, group=process_group
            )
        else:
````

- **L761** EN: Assigns or updates `world_size`. | CN: 对 `world_size` 进行赋值或更新。
- **L762** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L763** EN: Assigns or updates `local_sharded_tensor_metadata`. | CN: 对 `local_sharded_tensor_metadata` 进行赋值或更新。
- **L764** EN: Assigns or updates `global_tensor_size`. | CN: 对 `global_tensor_size` 进行赋值或更新。
- **L765** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L766** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L767** EN: Assigns or updates `local_sharded_tensor_metadata`. | CN: 对 `local_sharded_tensor_metadata` 进行赋值或更新。
- **L768** EN: Continues the implementation inside function `_init_from_local_shards`. | CN: 继续说明函数 `_init_from_local_shards` 内部的实现。
- **L769** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L770** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L771** EN: Keeps the inline comment or directive: STEP 2. Validate metadata across ranks, and build a global sharded tensor | CN: 保留这一行注释或指令：STEP 2. Validate metadata across ranks, and build a global sharded tensor
- **L772** EN: Keeps the inline comment or directive: metadata by gathering local ShardedTensorMetadata | CN: 保留这一行注释或指令：metadata by gathering local ShardedTensorMetadata
- **L773** EN: Assigns or updates `gathered_metadatas`. | CN: 对 `gathered_metadatas` 进行赋值或更新。
- **L774** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L775** EN: Assigns or updates `gathered_metadatas`. | CN: 对 `gathered_metadatas` 进行赋值或更新。
- **L776** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L777** EN: Calls `dist.all_gather_object` as part of the current workflow. | CN: 在当前流程中调用 `dist.all_gather_object`。
- **L778** EN: Assigns or updates `gathered_metadatas, local_sharded_tensor_metadata, group`. | CN: 对 `gathered_metadatas, local_sharded_tensor_metadata, group` 进行赋值或更新。
- **L779** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L780** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。

### Lines 781-800 / 第 781-800 行

````python
            gathered_metadatas = [local_sharded_tensor_metadata]

        global_sharded_tensor_metadata = build_global_metadata(
            gathered_metadatas, recalc_metadata=recalc_metadata
        )
        if recalc_metadata:
            # for recalc use cases, we only support rw for now, limit the blast radius
            # will modify here once we support more sharding type
            if not (
                len(local_shards) > 0
                and len(global_sharded_tensor_metadata.shards_metadata) > current_rank
            ):
                raise AssertionError(
                    f"# for metadata recalculation, local_shards must be larger than 0 "
                    f"actual:{len(local_shards)}, # glb metadata must be greater than any rank id, "
                    f"# metadata:{len(global_sharded_tensor_metadata.shards_metadata)}, rank id:{current_rank}"
                )
            local_md = [
                shard_md
                for shard_md in global_sharded_tensor_metadata.shards_metadata
````

- **L781** EN: Assigns or updates `gathered_metadatas`. | CN: 对 `gathered_metadatas` 进行赋值或更新。
- **L782** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L783** EN: Assigns or updates `global_sharded_tensor_metadata`. | CN: 对 `global_sharded_tensor_metadata` 进行赋值或更新。
- **L784** EN: Assigns or updates `gathered_metadatas, recalc_metadata`. | CN: 对 `gathered_metadatas, recalc_metadata` 进行赋值或更新。
- **L785** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L786** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L787** EN: Keeps the inline comment or directive: for recalc use cases, we only support rw for now, limit the blast radius | CN: 保留这一行注释或指令：for recalc use cases, we only support rw for now, limit the blast radius
- **L788** EN: Keeps the inline comment or directive: will modify here once we support more sharding type | CN: 保留这一行注释或指令：will modify here once we support more sharding type
- **L789** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L790** EN: Calls `len` as part of the current workflow. | CN: 在当前流程中调用 `len`。
- **L791** EN: Continues the implementation inside function `_init_from_local_shards`. | CN: 继续说明函数 `_init_from_local_shards` 内部的实现。
- **L792** EN: Continues the implementation inside function `_init_from_local_shards`. | CN: 继续说明函数 `_init_from_local_shards` 内部的实现。
- **L793** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L794** EN: Continues the implementation inside function `_init_from_local_shards`. | CN: 继续说明函数 `_init_from_local_shards` 内部的实现。
- **L795** EN: Continues the implementation inside function `_init_from_local_shards`. | CN: 继续说明函数 `_init_from_local_shards` 内部的实现。
- **L796** EN: Continues the implementation inside function `_init_from_local_shards`. | CN: 继续说明函数 `_init_from_local_shards` 内部的实现。
- **L797** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L798** EN: Assigns or updates `local_md`. | CN: 对 `local_md` 进行赋值或更新。
- **L799** EN: Continues the implementation inside function `_init_from_local_shards`. | CN: 继续说明函数 `_init_from_local_shards` 内部的实现。
- **L800** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。

### Lines 801-820 / 第 801-820 行

````python
                if shard_md.placement.rank() == current_rank
            ]
            if len(local_md) != 1:
                raise AssertionError(
                    f"should has and only has one metadata for local rank, actual:{local_md}"
                )
            local_shards[0].metadata = local_md[0]
        tensor_properties = global_sharded_tensor_metadata.tensor_properties

        # STEP 3: Validation done, create the actual ShardedTensor and populate fields
        # prepare initialization
        spec = shard_spec._infer_sharding_spec_from_shards_metadata(
            global_sharded_tensor_metadata.shards_metadata
        )
        sharded_tensor = cls.__new__(
            cls,
            spec,
            global_sharded_tensor_metadata.size,
            dtype=tensor_properties.dtype,
            layout=tensor_properties.layout,
````

- **L801** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L802** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L803** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L804** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L805** EN: Continues the implementation inside function `_init_from_local_shards`. | CN: 继续说明函数 `_init_from_local_shards` 内部的实现。
- **L806** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L807** EN: Assigns or updates `local_shards[0].metadata`. | CN: 对 `local_shards[0].metadata` 进行赋值或更新。
- **L808** EN: Assigns or updates `tensor_properties`. | CN: 对 `tensor_properties` 进行赋值或更新。
- **L809** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L810** EN: Keeps the inline comment or directive: STEP 3: Validation done, create the actual ShardedTensor and populate fields | CN: 保留这一行注释或指令：STEP 3: Validation done, create the actual ShardedTensor and populate fields
- **L811** EN: Keeps the inline comment or directive: prepare initialization | CN: 保留这一行注释或指令：prepare initialization
- **L812** EN: Assigns or updates `spec`. | CN: 对 `spec` 进行赋值或更新。
- **L813** EN: Continues the implementation inside function `_init_from_local_shards`. | CN: 继续说明函数 `_init_from_local_shards` 内部的实现。
- **L814** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L815** EN: Assigns or updates `sharded_tensor`. | CN: 对 `sharded_tensor` 进行赋值或更新。
- **L816** EN: Continues the implementation inside function `_init_from_local_shards`. | CN: 继续说明函数 `_init_from_local_shards` 内部的实现。
- **L817** EN: Continues the implementation inside function `_init_from_local_shards`. | CN: 继续说明函数 `_init_from_local_shards` 内部的实现。
- **L818** EN: Continues the implementation inside function `_init_from_local_shards`. | CN: 继续说明函数 `_init_from_local_shards` 内部的实现。
- **L819** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L820** EN: Assigns or updates `layout`. | CN: 对 `layout` 进行赋值或更新。

### Lines 821-840 / 第 821-840 行

````python
            pin_memory=tensor_properties.pin_memory,
            requires_grad=tensor_properties.requires_grad,
        )
        sharded_tensor._prepare_init(process_group=process_group, init_rrefs=init_rrefs)

        # attach local_shards to the ShardedTensor created
        sharded_tensor._local_shards = local_shards

        # run post initialization, i.e. map registration, rpc initialization
        sharded_tensor._post_init()
        return sharded_tensor

    @classmethod
    @deprecated(DEPRECATE_MSG, category=FutureWarning)
    def _init_from_local_tensor(
        cls,
        local_tensor: torch.Tensor,
        sharding_spec: shard_spec.ShardingSpec,
        *global_size: Sequence[int],
        process_group: dist.ProcessGroup | None = None,
````

- **L821** EN: Assigns or updates `pin_memory`. | CN: 对 `pin_memory` 进行赋值或更新。
- **L822** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L823** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L824** EN: Calls `sharded_tensor._prepare_init` as part of the current workflow. | CN: 在当前流程中调用 `sharded_tensor._prepare_init`。
- **L825** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L826** EN: Keeps the inline comment or directive: attach local_shards to the ShardedTensor created | CN: 保留这一行注释或指令：attach local_shards to the ShardedTensor created
- **L827** EN: Assigns or updates `sharded_tensor._local_shards`. | CN: 对 `sharded_tensor._local_shards` 进行赋值或更新。
- **L828** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L829** EN: Keeps the inline comment or directive: run post initialization, i.e. map registration, rpc initialization | CN: 保留这一行注释或指令：run post initialization, i.e. map registration, rpc initialization
- **L830** EN: Calls `sharded_tensor._post_init` as part of the current workflow. | CN: 在当前流程中调用 `sharded_tensor._post_init`。
- **L831** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L832** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L833** EN: Applies decorator `classmethod` to the following definition. | CN: 将装饰器 `classmethod` 应用于后续定义。
- **L834** EN: Applies decorator `deprecated(DEPRECATE_MSG, category=FutureWarning)` to the following definition. | CN: 将装饰器 `deprecated(DEPRECATE_MSG, category=FutureWarning)` 应用于后续定义。
- **L835** EN: Defines function `_init_from_local_tensor`. | CN: 定义函数 `_init_from_local_tensor`。
- **L836** EN: Continues the implementation inside function `_init_from_local_tensor`. | CN: 继续说明函数 `_init_from_local_tensor` 内部的实现。
- **L837** EN: Continues the implementation inside function `_init_from_local_tensor`. | CN: 继续说明函数 `_init_from_local_tensor` 内部的实现。
- **L838** EN: Continues the implementation inside function `_init_from_local_tensor`. | CN: 继续说明函数 `_init_from_local_tensor` 内部的实现。
- **L839** EN: Continues the implementation inside function `_init_from_local_tensor`. | CN: 继续说明函数 `_init_from_local_tensor` 内部的实现。
- **L840** EN: Assigns or updates `process_group`. | CN: 对 `process_group` 进行赋值或更新。

### Lines 841-860 / 第 841-860 行

````python
        init_rrefs=False,
    ) -> ShardedTensor:
        """
        Initialize a ShardedTensor given only one local tensor, global sharded tensor
        size and sharding spec on each rank.

        Args:
            local_tensor (Tensor): Single tensor of local shard stored in each rank.
            sharding_spec (:class:`torch.distributed._shard.sharding_spec.ShardingSpec`):
                The specification describing how to shard the Tensor.
            global_size (Sequence[int]): Size of the sharded tensor.
            process_group (ProcessGroup, optional): The process group to aggregate on.
                Default: None
            init_rrefs (bool, optional): Whether or not to initialize
                :class:`torch.distributed.rpc.RRef`s pointing to remote shards.
                Need to initialize the RPC Framework if specified as ``True``.
                Default: ``False``.

        Returns:
            A :class:`ShardedTensor` sharded based on the given sharding_spec with local
````

- **L841** EN: Assigns or updates `init_rrefs`. | CN: 对 `init_rrefs` 进行赋值或更新。
- **L842** EN: Continues the implementation inside function `_init_from_local_tensor`. | CN: 继续说明函数 `_init_from_local_tensor` 内部的实现。
- **L843** EN: Starts the docstring for the function _init_from_local_tensor. | CN: 开始定义 function _init_from_local_tensor 的文档字符串。
- **L844** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L845** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L846** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L847** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L848** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L849** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L850** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L851** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L852** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L853** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L854** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L855** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L856** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L857** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L858** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L859** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L860** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。

### Lines 861-880 / 第 861-880 行

````python
                tensor stored in the current rank.

        Examples:
            >>> # xdoctest: +SKIP
            >>> # All tensors below are of torch.int64 type.
            >>> # We have 2 process groups, 2 ranks.
            >>> tensor = torch.arange(2, dtype=torch.int64) + 1 + 2 * rank
            >>> local_tensor = torch.unsqueeze(torch.cat([tensor, tensor + 2]))
            >>> local_tensor
            tensor([[1, 2, 3, 4]]) # Rank 0
            tensor([[3, 4, 5, 6]]) # Rank 1
            >>> sharding_dim = 0
            >>> sharding_spec = ChunkShardingSpec(
                    dim=sharding_dim,
                    placements=[
                        "rank:0/cuda:0",
                        "rank:1/cuda:1",
                    ],
                )
            >>> st = ShardedTensor._init_from_local_tensor(
````

- **L861** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L862** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L863** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L864** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L865** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L866** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L867** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L868** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L869** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L870** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L871** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L872** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L873** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L874** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L875** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L876** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L877** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L878** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L879** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L880** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。

### Lines 881-900 / 第 881-900 行

````python
            ...     local_tensor, sharding_spec, [2, 4]
            ... )
            >>> st
            ShardedTensor(
                ShardedTensorMetadata(
                    shards_metadata=[
                        ShardMetadata(shard_offsets=[0, 0], shard_sizes=[1, 4], placement=rank:0/cuda:0),
                        ShardMetadata(shard_offsets=[1, 0], shard_sizes=[1, 4], placement=rank:1/cuda:1),
                    ],
                    size=torch.Size([2, 4])
            )
            >>> st.local_tensor()
            tensor([1, 2, 3, 4]) # Rank 0
            tensor([3, 4, 5, 6]) # Rank 1

        Warning: This API is experimental and subject to change. It lacks of a fully across
                 rank validations, and we only validate the local shard on the current rank.
                 We fully rely on the user to ensure local tensor is sharded based on the
                 sharding spec.
        """
````

- **L881** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L882** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L883** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L884** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L885** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L886** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L887** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L888** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L889** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L890** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L891** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L892** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L893** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L894** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L895** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L896** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L897** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L898** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L899** EN: Continues the docstring text for the function _init_from_local_tensor. | CN: 继续补充 function _init_from_local_tensor 的文档字符串内容。
- **L900** EN: Closes the docstring for the function _init_from_local_tensor. | CN: 结束 function _init_from_local_tensor 的文档字符串。

### Lines 901-920 / 第 901-920 行

````python
        if not local_tensor.is_contiguous():
            raise ValueError("local_tensor is not a contiguous Tensor.")

        global_tensor_size = _flatten_tensor_size(global_size)
        tensor_properties = TensorProperties(
            dtype=local_tensor.dtype,
            layout=local_tensor.layout,
            requires_grad=local_tensor.requires_grad,
            memory_format=torch.contiguous_format,
            pin_memory=local_tensor.is_pinned(),
        )
        sharded_tensor_metadata = sharding_spec.build_metadata(
            global_tensor_size, tensor_properties
        )

        process_group = cls._normalize_pg(process_group)
        current_rank = dist.get_rank()  # intentional to get global rank

        local_shards: list[Shard] = []
        for shard_metadata in sharded_tensor_metadata.shards_metadata:
````

- **L901** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L902** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L903** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L904** EN: Assigns or updates `global_tensor_size`. | CN: 对 `global_tensor_size` 进行赋值或更新。
- **L905** EN: Assigns or updates `tensor_properties`. | CN: 对 `tensor_properties` 进行赋值或更新。
- **L906** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L907** EN: Assigns or updates `layout`. | CN: 对 `layout` 进行赋值或更新。
- **L908** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L909** EN: Assigns or updates `memory_format`. | CN: 对 `memory_format` 进行赋值或更新。
- **L910** EN: Assigns or updates `pin_memory`. | CN: 对 `pin_memory` 进行赋值或更新。
- **L911** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L912** EN: Assigns or updates `sharded_tensor_metadata`. | CN: 对 `sharded_tensor_metadata` 进行赋值或更新。
- **L913** EN: Continues the implementation inside function `_init_from_local_tensor`. | CN: 继续说明函数 `_init_from_local_tensor` 内部的实现。
- **L914** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L915** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L916** EN: Assigns or updates `process_group`. | CN: 对 `process_group` 进行赋值或更新。
- **L917** EN: Assigns or updates `current_rank`. | CN: 对 `current_rank` 进行赋值或更新。
- **L918** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L919** EN: Assigns or updates `local_shards`. | CN: 对 `local_shards` 进行赋值或更新。
- **L920** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。

### Lines 921-940 / 第 921-940 行

````python
            rank, _device = _parse_and_validate_remote_device(
                process_group, shard_metadata.placement
            )
            if rank == current_rank:
                local_shards.append(Shard(local_tensor, shard_metadata))

        # TODO: figure out what the API should behave when some rank have no shard
        # see https://github.com/pytorch/pytorch/issues/7313
        return ShardedTensor._init_from_local_shards_and_global_metadata(
            local_shards,
            sharded_tensor_metadata,
            process_group=process_group,
            init_rrefs=init_rrefs,
            sharding_spec=sharding_spec,
        )

    @classmethod
    def _init_from_local_shards_and_global_metadata(  # type: ignore[override]
        cls,
        local_shards: list[Shard],
````

- **L921** EN: Assigns or updates `rank, _device`. | CN: 对 `rank, _device` 进行赋值或更新。
- **L922** EN: Continues the implementation inside function `_init_from_local_tensor`. | CN: 继续说明函数 `_init_from_local_tensor` 内部的实现。
- **L923** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L924** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L925** EN: Calls `local_shards.append` as part of the current workflow. | CN: 在当前流程中调用 `local_shards.append`。
- **L926** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L927** EN: Keeps the inline comment or directive: TODO: figure out what the API should behave when some rank have no shard | CN: 保留这一行注释或指令：TODO: figure out what the API should behave when some rank have no shard
- **L928** EN: Keeps the inline comment or directive: see https://github.com/pytorch/pytorch/issues/7313 | CN: 保留这一行注释或指令：see https://github.com/pytorch/pytorch/issues/7313
- **L929** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L930** EN: Continues the implementation inside function `_init_from_local_tensor`. | CN: 继续说明函数 `_init_from_local_tensor` 内部的实现。
- **L931** EN: Continues the implementation inside function `_init_from_local_tensor`. | CN: 继续说明函数 `_init_from_local_tensor` 内部的实现。
- **L932** EN: Assigns or updates `process_group`. | CN: 对 `process_group` 进行赋值或更新。
- **L933** EN: Assigns or updates `init_rrefs`. | CN: 对 `init_rrefs` 进行赋值或更新。
- **L934** EN: Assigns or updates `sharding_spec`. | CN: 对 `sharding_spec` 进行赋值或更新。
- **L935** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L936** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L937** EN: Applies decorator `classmethod` to the following definition. | CN: 将装饰器 `classmethod` 应用于后续定义。
- **L938** EN: Defines function `_init_from_local_shards_and_global_metadata`. | CN: 定义函数 `_init_from_local_shards_and_global_metadata`。
- **L939** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L940** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。

### Lines 941-960 / 第 941-960 行

````python
        sharded_tensor_metadata: ShardedTensorMetadata,
        process_group=None,
        init_rrefs=False,
        sharding_spec=None,
    ) -> ShardedTensor:
        """
        Initialize a ShardedTensor with local shards and a global
        ShardedTensorMetadata built on each rank.

        Warning: This API is experimental and subject to change. It does
                 not do cross rank validations, and fully rely on the user
                 for the correctness of sharded_tensor_metadata on each rank
        """
        process_group = cls._normalize_pg(process_group)
        current_rank = dist.get_rank()  # intentional to get global rank

        shards_metadata = sharded_tensor_metadata.shards_metadata

        local_shard_metadatas = []

````

- **L941** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L942** EN: Assigns or updates `process_group`. | CN: 对 `process_group` 进行赋值或更新。
- **L943** EN: Assigns or updates `init_rrefs`. | CN: 对 `init_rrefs` 进行赋值或更新。
- **L944** EN: Assigns or updates `sharding_spec`. | CN: 对 `sharding_spec` 进行赋值或更新。
- **L945** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L946** EN: Starts the docstring for the function _init_from_local_shards_and_global_metadata. | CN: 开始定义 function _init_from_local_shards_and_global_metadata 的文档字符串。
- **L947** EN: Continues the docstring text for the function _init_from_local_shards_and_global_metadata. | CN: 继续补充 function _init_from_local_shards_and_global_metadata 的文档字符串内容。
- **L948** EN: Continues the docstring text for the function _init_from_local_shards_and_global_metadata. | CN: 继续补充 function _init_from_local_shards_and_global_metadata 的文档字符串内容。
- **L949** EN: Continues the docstring text for the function _init_from_local_shards_and_global_metadata. | CN: 继续补充 function _init_from_local_shards_and_global_metadata 的文档字符串内容。
- **L950** EN: Continues the docstring text for the function _init_from_local_shards_and_global_metadata. | CN: 继续补充 function _init_from_local_shards_and_global_metadata 的文档字符串内容。
- **L951** EN: Continues the docstring text for the function _init_from_local_shards_and_global_metadata. | CN: 继续补充 function _init_from_local_shards_and_global_metadata 的文档字符串内容。
- **L952** EN: Continues the docstring text for the function _init_from_local_shards_and_global_metadata. | CN: 继续补充 function _init_from_local_shards_and_global_metadata 的文档字符串内容。
- **L953** EN: Closes the docstring for the function _init_from_local_shards_and_global_metadata. | CN: 结束 function _init_from_local_shards_and_global_metadata 的文档字符串。
- **L954** EN: Assigns or updates `process_group`. | CN: 对 `process_group` 进行赋值或更新。
- **L955** EN: Assigns or updates `current_rank`. | CN: 对 `current_rank` 进行赋值或更新。
- **L956** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L957** EN: Assigns or updates `shards_metadata`. | CN: 对 `shards_metadata` 进行赋值或更新。
- **L958** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L959** EN: Assigns or updates `local_shard_metadatas`. | CN: 对 `local_shard_metadatas` 进行赋值或更新。
- **L960** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 961-980 / 第 961-980 行

````python
        # collect local shard metadatas from the global sharded_tensor_metadata
        for shard_metadata in shards_metadata:  # type: ignore[attr-defined]
            rank, local_device = _parse_and_validate_remote_device(
                process_group, shard_metadata.placement
            )

            if current_rank == rank:
                local_shard_metadatas.append(shard_metadata)

        if len(local_shards) != len(local_shard_metadatas):
            raise RuntimeError(
                f"Number of local shards ({len(local_shards)}) does not match number of local "
                f"shards metadata in sharded_tensor_metadata ({len(local_shard_metadatas)}) "
                f"on rank ({current_rank}) "
            )

        shards_metadata = sharded_tensor_metadata.shards_metadata
        tensor_properties = sharded_tensor_metadata.tensor_properties

        if len(shards_metadata) == 0:
````

- **L961** EN: Keeps the inline comment or directive: collect local shard metadatas from the global sharded_tensor_metadata | CN: 保留这一行注释或指令：collect local shard metadatas from the global sharded_tensor_metadata
- **L962** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L963** EN: Assigns or updates `rank, local_device`. | CN: 对 `rank, local_device` 进行赋值或更新。
- **L964** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L965** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L966** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L967** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L968** EN: Calls `local_shard_metadatas.append` as part of the current workflow. | CN: 在当前流程中调用 `local_shard_metadatas.append`。
- **L969** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L970** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L971** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L972** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L973** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L974** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L975** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L976** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L977** EN: Assigns or updates `shards_metadata`. | CN: 对 `shards_metadata` 进行赋值或更新。
- **L978** EN: Assigns or updates `tensor_properties`. | CN: 对 `tensor_properties` 进行赋值或更新。
- **L979** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L980** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 981-1000 / 第 981-1000 行

````python
            raise ValueError("shards_metadata must not be empty!")

        if tensor_properties.layout != torch.strided:
            raise ValueError("Only torch.strided layout is currently supported")

        if sharding_spec is None:
            spec = shard_spec._infer_sharding_spec_from_shards_metadata(shards_metadata)
        else:
            spec = sharding_spec

        sharded_tensor = ShardedTensor.__new__(
            ShardedTensor,
            spec,
            sharded_tensor_metadata.size,
            dtype=tensor_properties.dtype,
            layout=tensor_properties.layout,
            pin_memory=tensor_properties.pin_memory,
            requires_grad=tensor_properties.requires_grad,
        )

````

- **L981** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L982** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L983** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L984** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L985** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L986** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L987** EN: Assigns or updates `spec`. | CN: 对 `spec` 进行赋值或更新。
- **L988** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L989** EN: Assigns or updates `spec`. | CN: 对 `spec` 进行赋值或更新。
- **L990** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L991** EN: Assigns or updates `sharded_tensor`. | CN: 对 `sharded_tensor` 进行赋值或更新。
- **L992** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L993** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L994** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L995** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L996** EN: Assigns or updates `layout`. | CN: 对 `layout` 进行赋值或更新。
- **L997** EN: Assigns or updates `pin_memory`. | CN: 对 `pin_memory` 进行赋值或更新。
- **L998** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L999** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1000** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 1001-1020 / 第 1001-1020 行

````python
        def _raise_if_mismatch(expected, actual, prop_name, rank, is_property=False):
            tensor_property_or_metadata = (
                "tensor property" if is_property else "local ShardMetadata"
            )
            if expected != actual:
                raise ValueError(
                    f"Local shards' tensor {prop_name} property is incompatible with "
                    f"{tensor_property_or_metadata} on rank {rank}: "
                    f"{tensor_property_or_metadata} {prop_name}={expected}, "
                    f"local shard tensor {prop_name}={actual}."
                )

        for shard in local_shards:
            shard_meta = shard.metadata
            local_shard_tensor = shard.tensor
            placement = shard_meta.placement
            if placement is None:
                raise AssertionError("Must specify placement for `Shard`!")
            rank = placement.rank()
            local_device = placement.device()
````

- **L1001** EN: Defines function `_raise_if_mismatch`. | CN: 定义函数 `_raise_if_mismatch`。
- **L1002** EN: Assigns or updates `tensor_property_or_metadata`. | CN: 对 `tensor_property_or_metadata` 进行赋值或更新。
- **L1003** EN: Continues the implementation inside function `_raise_if_mismatch`. | CN: 继续说明函数 `_raise_if_mismatch` 内部的实现。
- **L1004** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1005** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L1006** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L1007** EN: Continues the implementation inside function `_raise_if_mismatch`. | CN: 继续说明函数 `_raise_if_mismatch` 内部的实现。
- **L1008** EN: Continues the implementation inside function `_raise_if_mismatch`. | CN: 继续说明函数 `_raise_if_mismatch` 内部的实现。
- **L1009** EN: Continues the implementation inside function `_raise_if_mismatch`. | CN: 继续说明函数 `_raise_if_mismatch` 内部的实现。
- **L1010** EN: Continues the implementation inside function `_raise_if_mismatch`. | CN: 继续说明函数 `_raise_if_mismatch` 内部的实现。
- **L1011** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1012** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1013** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L1014** EN: Assigns or updates `shard_meta`. | CN: 对 `shard_meta` 进行赋值或更新。
- **L1015** EN: Assigns or updates `local_shard_tensor`. | CN: 对 `local_shard_tensor` 进行赋值或更新。
- **L1016** EN: Assigns or updates `placement`. | CN: 对 `placement` 进行赋值或更新。
- **L1017** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L1018** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L1019** EN: Assigns or updates `rank`. | CN: 对 `rank` 进行赋值或更新。
- **L1020** EN: Assigns or updates `local_device`. | CN: 对 `local_device` 进行赋值或更新。

### Lines 1021-1040 / 第 1021-1040 行

````python

            _raise_if_mismatch(
                tensor_properties.layout,
                local_shard_tensor.layout,
                "layout",
                rank,
                True,
            )
            if not local_shard_tensor.is_contiguous():
                raise ValueError(
                    "Only torch.contiguous_format memory_format is currently supported"
                )

            _raise_if_mismatch(
                shard_meta.shard_sizes,
                list(local_shard_tensor.size()),
                "size",
                rank,
            )
            _raise_if_mismatch(
````

- **L1021** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1022** EN: Calls `_raise_if_mismatch` as part of the current workflow. | CN: 在当前流程中调用 `_raise_if_mismatch`。
- **L1023** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1024** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1025** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1026** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1027** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1028** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1029** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L1030** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L1031** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1032** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1033** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1034** EN: Calls `_raise_if_mismatch` as part of the current workflow. | CN: 在当前流程中调用 `_raise_if_mismatch`。
- **L1035** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1036** EN: Calls `list` as part of the current workflow. | CN: 在当前流程中调用 `list`。
- **L1037** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1038** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1039** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1040** EN: Calls `_raise_if_mismatch` as part of the current workflow. | CN: 在当前流程中调用 `_raise_if_mismatch`。

### Lines 1041-1060 / 第 1041-1060 行

````python
                tensor_properties.pin_memory,
                local_shard_tensor.is_pinned(),
                "pin_memory",
                rank,
                True,
            )
            _raise_if_mismatch(local_device, local_shard_tensor.device, "device", rank)
            _raise_if_mismatch(
                tensor_properties.dtype,
                local_shard_tensor.dtype,
                "dtype",
                rank,
                True,
            )
            _raise_if_mismatch(
                tensor_properties.requires_grad,
                local_shard_tensor.requires_grad,
                "requires_grad",
                rank,
                True,
````

- **L1041** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1042** EN: Calls `local_shard_tensor.is_pinned` as part of the current workflow. | CN: 在当前流程中调用 `local_shard_tensor.is_pinned`。
- **L1043** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1044** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1045** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1046** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1047** EN: Calls `_raise_if_mismatch` as part of the current workflow. | CN: 在当前流程中调用 `_raise_if_mismatch`。
- **L1048** EN: Calls `_raise_if_mismatch` as part of the current workflow. | CN: 在当前流程中调用 `_raise_if_mismatch`。
- **L1049** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1050** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1051** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1052** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1053** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1054** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1055** EN: Calls `_raise_if_mismatch` as part of the current workflow. | CN: 在当前流程中调用 `_raise_if_mismatch`。
- **L1056** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1057** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1058** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1059** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。
- **L1060** EN: Continues the implementation inside function `_init_from_local_shards_and_global_metadata`. | CN: 继续说明函数 `_init_from_local_shards_and_global_metadata` 内部的实现。

### Lines 1061-1080 / 第 1061-1080 行

````python
            )

        # check if shards_metadata have overlap shards
        validate_non_overlapping_shards_metadata(shards_metadata)

        # check if the shards_metadata is compatible with overall size of the sharded tensor.
        check_tensor(shards_metadata, list(sharded_tensor_metadata.size))

        # done validation, add local_shards
        sharded_tensor._local_shards = local_shards
        sharded_tensor._prepare_init(process_group=process_group, init_rrefs=init_rrefs)

        # run post initialization, i.e. map registration, rpc initialization
        sharded_tensor._post_init()
        return sharded_tensor

    def sharding_spec(self) -> shard_spec.ShardingSpec:
        """
        Returns the ShardingSpec for the tensor.
        """
````

- **L1061** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1062** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1063** EN: Keeps the inline comment or directive: check if shards_metadata have overlap shards | CN: 保留这一行注释或指令：check if shards_metadata have overlap shards
- **L1064** EN: Calls `validate_non_overlapping_shards_metadata` as part of the current workflow. | CN: 在当前流程中调用 `validate_non_overlapping_shards_metadata`。
- **L1065** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1066** EN: Keeps the inline comment or directive: check if the shards_metadata is compatible with overall size of the sharded tens | CN: 保留这一行注释或指令：check if the shards_metadata is compatible with overall size of the sharded tens
- **L1067** EN: Calls `check_tensor` as part of the current workflow. | CN: 在当前流程中调用 `check_tensor`。
- **L1068** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1069** EN: Keeps the inline comment or directive: done validation, add local_shards | CN: 保留这一行注释或指令：done validation, add local_shards
- **L1070** EN: Assigns or updates `sharded_tensor._local_shards`. | CN: 对 `sharded_tensor._local_shards` 进行赋值或更新。
- **L1071** EN: Calls `sharded_tensor._prepare_init` as part of the current workflow. | CN: 在当前流程中调用 `sharded_tensor._prepare_init`。
- **L1072** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1073** EN: Keeps the inline comment or directive: run post initialization, i.e. map registration, rpc initialization | CN: 保留这一行注释或指令：run post initialization, i.e. map registration, rpc initialization
- **L1074** EN: Calls `sharded_tensor._post_init` as part of the current workflow. | CN: 在当前流程中调用 `sharded_tensor._post_init`。
- **L1075** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L1076** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1077** EN: Defines function `sharding_spec`. | CN: 定义函数 `sharding_spec`。
- **L1078** EN: Starts the docstring for the function sharding_spec. | CN: 开始定义 function sharding_spec 的文档字符串。
- **L1079** EN: Continues the docstring text for the function sharding_spec. | CN: 继续补充 function sharding_spec 的文档字符串内容。
- **L1080** EN: Closes the docstring for the function sharding_spec. | CN: 结束 function sharding_spec 的文档字符串。

### Lines 1081-1100 / 第 1081-1100 行

````python
        return self._sharding_spec

    @deprecated(DEPRECATE_MSG, category=FutureWarning)
    def reshard(self, resharding_spec: shard_spec.ShardingSpec) -> ShardedTensor:
        """
        Reshard a sharded tensor given the ``resharding_spec``. For now, we only support
        single local shard.

        If ``resharding_spec`` is same as the original one, this becomes a no-op.
        If only ``resharding_spec`` shares the same sharding dim with the original one,
        we swap local shards directly.
        For more generic cases, we merge different shards across different ranks and split
        the local shards based on the ``resharding_spec`` via `all_to_all` collective API.

        Args:
            resharding_spec (:class:`torch.distributed._shard.sharding_spec.ShardingSpec`): The
                specification describing how the tensor is sharded.

        Returns:
            A :class:`ShardedTensor` object whose local shards are resharded.
````

- **L1081** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L1082** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1083** EN: Applies decorator `deprecated(DEPRECATE_MSG, category=FutureWarning)` to the following definition. | CN: 将装饰器 `deprecated(DEPRECATE_MSG, category=FutureWarning)` 应用于后续定义。
- **L1084** EN: Defines function `reshard`. | CN: 定义函数 `reshard`。
- **L1085** EN: Starts the docstring for the function reshard. | CN: 开始定义 function reshard 的文档字符串。
- **L1086** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1087** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1088** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1089** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1090** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1091** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1092** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1093** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1094** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1095** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1096** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1097** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1098** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1099** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1100** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。

### Lines 1101-1120 / 第 1101-1120 行

````python

        Examples:
            >>> # xdoctest: +SKIP
            >>> # We have 2 process groups, 2 ranks.
            >>> tensor = torch.arange(4, dtype=torch.int64) + 1 + 2 * rank
            >>> tensor = torch.stack([tensor, tensor])
            >>> tensor
            tensor([[1, 2, 3, 4], [1, 2, 3, 4]]) # Rank 0
            tensor([[3, 4, 5, 6], [3, 4, 5, 6]]) # Rank 1
            tensor([[5, 6, 7, 8], [5, 6, 7, 8]]) # Rank 2
            tensor([[7, 8, 9, 10], [7, 8, 9, 10]]) # Rank 3
            >>> sharding_dim = 0
            >>> spec = ChunkShardingSpec(
                    dim=sharding_dim,
                    placements=[
                        "rank:0/cuda:0",
                        "rank:1/cuda:1",
                        "rank:2/cuda:2",
                        "rank:3/cuda:3",
                    ],
````

- **L1101** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1102** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1103** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1104** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1105** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1106** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1107** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1108** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1109** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1110** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1111** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1112** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1113** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1114** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1115** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1116** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1117** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1118** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1119** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1120** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。

### Lines 1121-1140 / 第 1121-1140 行

````python
                )
            >>> current_offsets = [0] * 2
            >>> current_offsets[0] = rank * 2
            >>> shard_metadata = ShardMetadata(
                    shard_offsets=copy.deepcopy(current_offsets),
                    shard_sizes=tensor.size(),
                    placement=spec.placements[rank],
                )
            >>> local_shards = [
                    Shard(
                        tensor=tensor,
                        metadata=shard_metadata,
                    )
                ]
            >>> st = ShardedTensor._init_from_local_shards(local_shards, tensor.size())
            >>> sharding_dim = 1
            >>> resharding_spec = ChunkShardingSpec(
                    dim=sharding_dim,
                    placements=[
                        "rank:0/cuda:0",
````

- **L1121** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1122** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1123** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1124** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1125** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1126** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1127** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1128** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1129** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1130** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1131** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1132** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1133** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1134** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1135** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1136** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1137** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1138** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1139** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1140** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。

### Lines 1141-1160 / 第 1141-1160 行

````python
                        "rank:1/cuda:1",
                        "rank:2/cuda:2",
                        "rank:3/cuda:3",
                    ],
                )
            >>> st.reshard(resharding_spec)
            >>> tensor = st.local_shards()[0].tensor
            >>> tensor
            tensor([[1], [1], [3], [3], [5], [5], [7], [7]]) # Rank 0
            tensor([[2], [2], [4], [4], [6], [6], [8], [8]]) # Rank 1
            tensor([[3], [3], [5], [5], [7], [7], [9], [9]]) # Rank 2
            tensor([[4], [4], [6], [6], [8], [8], [10], [10]]) # Rank 3
        """
        if not isinstance(
            resharding_spec, shard_spec.ChunkShardingSpec
        ) or not isinstance(self._sharding_spec, shard_spec.ChunkShardingSpec):
            raise NotImplementedError("Only ChunkShardingSpec supported for reshard.")

        num_local_shards = len(self.local_shards())
        if num_local_shards != 1:
````

- **L1141** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1142** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1143** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1144** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1145** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1146** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1147** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1148** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1149** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1150** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1151** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1152** EN: Continues the docstring text for the function reshard. | CN: 继续补充 function reshard 的文档字符串内容。
- **L1153** EN: Closes the docstring for the function reshard. | CN: 结束 function reshard 的文档字符串。
- **L1154** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L1155** EN: Continues the implementation inside function `reshard`. | CN: 继续说明函数 `reshard` 内部的实现。
- **L1156** EN: Continues the implementation inside function `reshard`. | CN: 继续说明函数 `reshard` 内部的实现。
- **L1157** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L1158** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1159** EN: Assigns or updates `num_local_shards`. | CN: 对 `num_local_shards` 进行赋值或更新。
- **L1160** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 1161-1180 / 第 1161-1180 行

````python
            raise NotImplementedError(
                f"Only single local shard supported for reshard. Number of shards: {num_local_shards}"
            )

        if self._sharding_spec.dim == resharding_spec.dim:  # type: ignore[attr-defined]
            if self._sharding_spec.placements == resharding_spec.placements:  # type: ignore[attr-defined]
                return self
            else:
                local_shards, shards_metadata = reshuffle_local_shard(
                    self.local_tensor(),
                    self.size(),  # type: ignore[arg-type]
                    self._sharding_spec,
                    resharding_spec,
                    self._process_group,
                )
        else:
            local_shards, shards_metadata = reshard_local_shard(
                self.local_tensor(),
                self.size(),  # type: ignore[arg-type]
                self._sharding_spec,
````

- **L1161** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L1162** EN: Continues the implementation inside function `reshard`. | CN: 继续说明函数 `reshard` 内部的实现。
- **L1163** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1164** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1165** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L1166** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L1167** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L1168** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L1169** EN: Assigns or updates `local_shards, shards_metadata`. | CN: 对 `local_shards, shards_metadata` 进行赋值或更新。
- **L1170** EN: Calls `self.local_tensor` as part of the current workflow. | CN: 在当前流程中调用 `self.local_tensor`。
- **L1171** EN: Calls `self.size` as part of the current workflow. | CN: 在当前流程中调用 `self.size`。
- **L1172** EN: Continues the implementation inside function `reshard`. | CN: 继续说明函数 `reshard` 内部的实现。
- **L1173** EN: Continues the implementation inside function `reshard`. | CN: 继续说明函数 `reshard` 内部的实现。
- **L1174** EN: Continues the implementation inside function `reshard`. | CN: 继续说明函数 `reshard` 内部的实现。
- **L1175** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1176** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L1177** EN: Assigns or updates `local_shards, shards_metadata`. | CN: 对 `local_shards, shards_metadata` 进行赋值或更新。
- **L1178** EN: Calls `self.local_tensor` as part of the current workflow. | CN: 在当前流程中调用 `self.local_tensor`。
- **L1179** EN: Calls `self.size` as part of the current workflow. | CN: 在当前流程中调用 `self.size`。
- **L1180** EN: Continues the implementation inside function `reshard`. | CN: 继续说明函数 `reshard` 内部的实现。

### Lines 1181-1200 / 第 1181-1200 行

````python
                resharding_spec,
                self._process_group,
            )
        self._local_shards = local_shards
        self._metadata.shards_metadata = shards_metadata
        self._sharding_spec = resharding_spec
        return self

    def local_tensor(self) -> torch.Tensor:
        """
        Return local tensor for a sharded_tensor. For now we only support single local shard.

        Returns:
            A :class:`torch.Tensor` of the local shard.
        """
        num_local_shards = len(self.local_shards())
        if num_local_shards != 1:
            raise NotImplementedError(
                f"Only single local shard is supported. Number of shards: {num_local_shards}"
            )
````

- **L1181** EN: Continues the implementation inside function `reshard`. | CN: 继续说明函数 `reshard` 内部的实现。
- **L1182** EN: Continues the implementation inside function `reshard`. | CN: 继续说明函数 `reshard` 内部的实现。
- **L1183** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1184** EN: Assigns or updates `self._local_shards`. | CN: 对 `self._local_shards` 进行赋值或更新。
- **L1185** EN: Assigns or updates `self._metadata.shards_metadata`. | CN: 对 `self._metadata.shards_metadata` 进行赋值或更新。
- **L1186** EN: Assigns or updates `self._sharding_spec`. | CN: 对 `self._sharding_spec` 进行赋值或更新。
- **L1187** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L1188** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1189** EN: Defines function `local_tensor`. | CN: 定义函数 `local_tensor`。
- **L1190** EN: Starts the docstring for the function local_tensor. | CN: 开始定义 function local_tensor 的文档字符串。
- **L1191** EN: Continues the docstring text for the function local_tensor. | CN: 继续补充 function local_tensor 的文档字符串内容。
- **L1192** EN: Continues the docstring text for the function local_tensor. | CN: 继续补充 function local_tensor 的文档字符串内容。
- **L1193** EN: Continues the docstring text for the function local_tensor. | CN: 继续补充 function local_tensor 的文档字符串内容。
- **L1194** EN: Continues the docstring text for the function local_tensor. | CN: 继续补充 function local_tensor 的文档字符串内容。
- **L1195** EN: Closes the docstring for the function local_tensor. | CN: 结束 function local_tensor 的文档字符串。
- **L1196** EN: Assigns or updates `num_local_shards`. | CN: 对 `num_local_shards` 进行赋值或更新。
- **L1197** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L1198** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L1199** EN: Continues the implementation inside function `local_tensor`. | CN: 继续说明函数 `local_tensor` 内部的实现。
- **L1200** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 1201-1220 / 第 1201-1220 行

````python
        return self.local_shards()[0].tensor

    @classmethod
    @deprecated(DEPRECATE_MSG, category=FutureWarning)
    def __torch_function__(cls, func, types, args=(), kwargs=None):
        def dispatch(st: ShardedTensor, func: Callable):
            # Dispatch to custom user provided op first if it exists.
            if func in _CUSTOM_SHARDED_OPS:
                return _CUSTOM_SHARDED_OPS[func](types, args, kwargs, st._process_group)

            # Dispatch to custom sharding spec op if it has one.
            if _has_custom_op(st._sharding_spec, func):
                return _dispatch_custom_op(
                    st._sharding_spec, func, types, args, kwargs, st._process_group
                )

            if func in _SHARDED_OPS:
                return _SHARDED_OPS[func](types, args, kwargs, st._process_group)

            raise RuntimeError(
````

- **L1201** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L1202** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1203** EN: Applies decorator `classmethod` to the following definition. | CN: 将装饰器 `classmethod` 应用于后续定义。
- **L1204** EN: Applies decorator `deprecated(DEPRECATE_MSG, category=FutureWarning)` to the following definition. | CN: 将装饰器 `deprecated(DEPRECATE_MSG, category=FutureWarning)` 应用于后续定义。
- **L1205** EN: Defines function `__torch_function__`. | CN: 定义函数 `__torch_function__`。
- **L1206** EN: Defines function `dispatch`. | CN: 定义函数 `dispatch`。
- **L1207** EN: Keeps the inline comment or directive: Dispatch to custom user provided op first if it exists. | CN: 保留这一行注释或指令：Dispatch to custom user provided op first if it exists.
- **L1208** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L1209** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L1210** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1211** EN: Keeps the inline comment or directive: Dispatch to custom sharding spec op if it has one. | CN: 保留这一行注释或指令：Dispatch to custom sharding spec op if it has one.
- **L1212** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L1213** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L1214** EN: Continues the implementation inside function `dispatch`. | CN: 继续说明函数 `dispatch` 内部的实现。
- **L1215** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1216** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1217** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L1218** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L1219** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1220** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。

### Lines 1221-1240 / 第 1221-1240 行

````python
                f"torch function '{func.__name__}', with args: {args} and "
                f"kwargs: {kwargs} not supported for ShardedTensor!"
            )

        # Find ShardedTensor instance to get process_group and sharding_spec.
        st_instance = None

        def find_sharded_tensor(e):
            nonlocal st_instance
            if st_instance is None and isinstance(e, ShardedTensor):
                st_instance = e

        pytree.tree_map_(find_sharded_tensor, args)
        pytree.tree_map_(find_sharded_tensor, kwargs)

        if st_instance is not None:
            return dispatch(st_instance, func)

        raise RuntimeError(
            f"torch function '{func.__name__}', with args: {args} and "
````

- **L1221** EN: Continues the implementation inside function `dispatch`. | CN: 继续说明函数 `dispatch` 内部的实现。
- **L1222** EN: Continues the implementation inside function `dispatch`. | CN: 继续说明函数 `dispatch` 内部的实现。
- **L1223** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1224** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1225** EN: Keeps the inline comment or directive: Find ShardedTensor instance to get process_group and sharding_spec. | CN: 保留这一行注释或指令：Find ShardedTensor instance to get process_group and sharding_spec.
- **L1226** EN: Assigns or updates `st_instance`. | CN: 对 `st_instance` 进行赋值或更新。
- **L1227** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1228** EN: Defines function `find_sharded_tensor`. | CN: 定义函数 `find_sharded_tensor`。
- **L1229** EN: Declares enclosing-scope names for reassignment. | CN: 声明需要重新赋值的外层作用域名称。
- **L1230** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L1231** EN: Assigns or updates `st_instance`. | CN: 对 `st_instance` 进行赋值或更新。
- **L1232** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1233** EN: Calls `pytree.tree_map_` as part of the current workflow. | CN: 在当前流程中调用 `pytree.tree_map_`。
- **L1234** EN: Calls `pytree.tree_map_` as part of the current workflow. | CN: 在当前流程中调用 `pytree.tree_map_`。
- **L1235** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1236** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L1237** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L1238** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1239** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L1240** EN: Continues the implementation inside function `__torch_function__`. | CN: 继续说明函数 `__torch_function__` 内部的实现。

### Lines 1241-1260 / 第 1241-1260 行

````python
            f"kwargs: {kwargs} not supported for ShardedTensor!"
        )

    def is_pinned(self) -> bool:  # type: ignore[override]
        """
        Returns True if the sharded tensor (each local shard) resides in pinned memory.
        """
        return self._metadata.tensor_properties.pin_memory

    def _register_remote_shards(
        self, remote_shards: list[rpc.RRef[Shard]], rpc_rank: int
    ):
        self._remote_shards[rpc_rank] = remote_shards

    def remote_shards(self) -> dict[int, list[rpc.RRef[Shard]]]:
        """
        Returns a Dict[int, RRef] with keys being the RPC rank and values
        being RRefs to shards on that rank. Need to initialize the
        RPC framework for this functionality.

````

- **L1241** EN: Continues the implementation inside function `__torch_function__`. | CN: 继续说明函数 `__torch_function__` 内部的实现。
- **L1242** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1243** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1244** EN: Defines function `is_pinned`. | CN: 定义函数 `is_pinned`。
- **L1245** EN: Starts the docstring for the function is_pinned. | CN: 开始定义 function is_pinned 的文档字符串。
- **L1246** EN: Continues the docstring text for the function is_pinned. | CN: 继续补充 function is_pinned 的文档字符串内容。
- **L1247** EN: Closes the docstring for the function is_pinned. | CN: 结束 function is_pinned 的文档字符串。
- **L1248** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L1249** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1250** EN: Defines function `_register_remote_shards`. | CN: 定义函数 `_register_remote_shards`。
- **L1251** EN: Continues the implementation inside function `_register_remote_shards`. | CN: 继续说明函数 `_register_remote_shards` 内部的实现。
- **L1252** EN: Continues the implementation inside function `_register_remote_shards`. | CN: 继续说明函数 `_register_remote_shards` 内部的实现。
- **L1253** EN: Assigns or updates `self._remote_shards[rpc_rank]`. | CN: 对 `self._remote_shards[rpc_rank]` 进行赋值或更新。
- **L1254** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1255** EN: Defines function `remote_shards`. | CN: 定义函数 `remote_shards`。
- **L1256** EN: Starts the docstring for the function remote_shards. | CN: 开始定义 function remote_shards 的文档字符串。
- **L1257** EN: Continues the docstring text for the function remote_shards. | CN: 继续补充 function remote_shards 的文档字符串内容。
- **L1258** EN: Continues the docstring text for the function remote_shards. | CN: 继续补充 function remote_shards 的文档字符串内容。
- **L1259** EN: Continues the docstring text for the function remote_shards. | CN: 继续补充 function remote_shards 的文档字符串内容。
- **L1260** EN: Continues the docstring text for the function remote_shards. | CN: 继续补充 function remote_shards 的文档字符串内容。

### Lines 1261-1280 / 第 1261-1280 行

````python
        Raises an exception if ShardedTensor was created with ``init_rrefs=False``
        """
        if not self._init_rrefs:
            raise RuntimeError(
                "ShardedTensor created with init_rrefs=False, no RRefs to remote shards available"
            )
        return self._remote_shards

    def __hash__(self):
        return id(self)

    def __repr__(self) -> str:  # type: ignore[override]
        return f"ShardedTensor({self._metadata})"

    @dataclass
    class ProcessGroupState:
        """
        State for ser-de of process group
        """

````

- **L1261** EN: Continues the docstring text for the function remote_shards. | CN: 继续补充 function remote_shards 的文档字符串内容。
- **L1262** EN: Closes the docstring for the function remote_shards. | CN: 结束 function remote_shards 的文档字符串。
- **L1263** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L1264** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L1265** EN: Continues the implementation inside function `remote_shards`. | CN: 继续说明函数 `remote_shards` 内部的实现。
- **L1266** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1267** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L1268** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1269** EN: Defines function `__hash__`. | CN: 定义函数 `__hash__`。
- **L1270** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L1271** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1272** EN: Defines function `__repr__`. | CN: 定义函数 `__repr__`。
- **L1273** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L1274** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1275** EN: Applies decorator `dataclass` to the following definition. | CN: 将装饰器 `dataclass` 应用于后续定义。
- **L1276** EN: Defines class `ProcessGroupState`. | CN: 定义类 `ProcessGroupState`。
- **L1277** EN: Starts the docstring for the class ProcessGroupState. | CN: 开始定义 class ProcessGroupState 的文档字符串。
- **L1278** EN: Continues the docstring text for the class ProcessGroupState. | CN: 继续补充 class ProcessGroupState 的文档字符串内容。
- **L1279** EN: Closes the docstring for the class ProcessGroupState. | CN: 结束 class ProcessGroupState 的文档字符串。
- **L1280** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 1281-1300 / 第 1281-1300 行

````python
        local_rank: int
        global_rank: int
        local_world_size: int
        global_world_size: int

    def __getstate__(self):
        pg_state = ShardedTensor.ProcessGroupState(
            distributed_c10d.get_rank(self._process_group),
            distributed_c10d.get_rank(),
            distributed_c10d.get_world_size(self._process_group),
            distributed_c10d.get_world_size(),
        )

        return (
            self._local_shards,
            self._metadata,
            pg_state,
            self._sharding_spec,
            self._init_rrefs,
        )
````

- **L1281** EN: Continues the implementation inside class `ProcessGroupState`. | CN: 继续说明类 `ProcessGroupState` 内部的实现。
- **L1282** EN: Continues the implementation inside class `ProcessGroupState`. | CN: 继续说明类 `ProcessGroupState` 内部的实现。
- **L1283** EN: Continues the implementation inside class `ProcessGroupState`. | CN: 继续说明类 `ProcessGroupState` 内部的实现。
- **L1284** EN: Continues the implementation inside class `ProcessGroupState`. | CN: 继续说明类 `ProcessGroupState` 内部的实现。
- **L1285** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1286** EN: Defines function `__getstate__`. | CN: 定义函数 `__getstate__`。
- **L1287** EN: Assigns or updates `pg_state`. | CN: 对 `pg_state` 进行赋值或更新。
- **L1288** EN: Calls `distributed_c10d.get_rank` as part of the current workflow. | CN: 在当前流程中调用 `distributed_c10d.get_rank`。
- **L1289** EN: Calls `distributed_c10d.get_rank` as part of the current workflow. | CN: 在当前流程中调用 `distributed_c10d.get_rank`。
- **L1290** EN: Calls `distributed_c10d.get_world_size` as part of the current workflow. | CN: 在当前流程中调用 `distributed_c10d.get_world_size`。
- **L1291** EN: Calls `distributed_c10d.get_world_size` as part of the current workflow. | CN: 在当前流程中调用 `distributed_c10d.get_world_size`。
- **L1292** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1293** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1294** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L1295** EN: Continues the implementation inside function `__getstate__`. | CN: 继续说明函数 `__getstate__` 内部的实现。
- **L1296** EN: Continues the implementation inside function `__getstate__`. | CN: 继续说明函数 `__getstate__` 内部的实现。
- **L1297** EN: Continues the implementation inside function `__getstate__`. | CN: 继续说明函数 `__getstate__` 内部的实现。
- **L1298** EN: Continues the implementation inside function `__getstate__`. | CN: 继续说明函数 `__getstate__` 内部的实现。
- **L1299** EN: Continues the implementation inside function `__getstate__`. | CN: 继续说明函数 `__getstate__` 内部的实现。
- **L1300** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 1301-1320 / 第 1301-1320 行

````python

    def __setstate__(self, state):
        self._sharded_tensor_id = None
        if not distributed_c10d.is_initialized():
            raise RuntimeError(
                "Need to initialize default process group using "
                '"init_process_group" before loading ShardedTensor'
            )

        (
            self._local_shards,
            self._metadata,
            pg_state,
            self._sharding_spec,
            self._init_rrefs,
        ) = state

        # Setup process group
        from torch.distributed._shard.api import _get_current_process_group

````

- **L1301** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1302** EN: Defines function `__setstate__`. | CN: 定义函数 `__setstate__`。
- **L1303** EN: Assigns or updates `self._sharded_tensor_id`. | CN: 对 `self._sharded_tensor_id` 进行赋值或更新。
- **L1304** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L1305** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L1306** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L1307** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L1308** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1309** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1310** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L1311** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L1312** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L1313** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L1314** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L1315** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L1316** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L1317** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1318** EN: Keeps the inline comment or directive: Setup process group | CN: 保留这一行注释或指令：Setup process group
- **L1319** EN: Imports selected names from `torch.distributed._shard.api`. | CN: 从 `torch.distributed._shard.api` 导入指定名称。
- **L1320** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 1321-1340 / 第 1321-1340 行

````python
        self._process_group = _get_current_process_group()

        # Validate process group.
        local_rank = distributed_c10d.get_rank(self._process_group)
        if pg_state.local_rank != local_rank:
            raise RuntimeError(
                f"Local rank at save time was {pg_state.local_rank}, but at "
                f"load time was {local_rank}"
            )

        global_rank = distributed_c10d.get_rank()
        if pg_state.global_rank != global_rank:
            raise RuntimeError(
                f"Global rank at save time was {pg_state.global_rank}, but at "
                f"load time was {global_rank}"
            )

        local_world_size = distributed_c10d.get_world_size(self._process_group)
        if pg_state.local_world_size != local_world_size:
            raise RuntimeError(
````

- **L1321** EN: Assigns or updates `self._process_group`. | CN: 对 `self._process_group` 进行赋值或更新。
- **L1322** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1323** EN: Keeps the inline comment or directive: Validate process group. | CN: 保留这一行注释或指令：Validate process group.
- **L1324** EN: Assigns or updates `local_rank`. | CN: 对 `local_rank` 进行赋值或更新。
- **L1325** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L1326** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L1327** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L1328** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L1329** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1330** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1331** EN: Assigns or updates `global_rank`. | CN: 对 `global_rank` 进行赋值或更新。
- **L1332** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L1333** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L1334** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L1335** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L1336** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1337** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1338** EN: Assigns or updates `local_world_size`. | CN: 对 `local_world_size` 进行赋值或更新。
- **L1339** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L1340** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。

### Lines 1341-1360 / 第 1341-1360 行

````python
                f"Local world size at save time was {pg_state.local_world_size}, "
                f"but at load time was {local_world_size}"
            )

        global_world_size = distributed_c10d.get_world_size()
        if pg_state.global_world_size != global_world_size:
            raise RuntimeError(
                f"Global world size at save time was {pg_state.global_world_size}, "
                f"but at load time was {global_world_size}"
            )

        self._post_init()


def _create_tensor_from_params(
    *size, local_device, tensor_properties: TensorProperties
):
    """Helper to construct tensor from size, device and common params."""
    dtype = tensor_properties.dtype
    layout = tensor_properties.layout
````

- **L1341** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L1342** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L1343** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1344** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1345** EN: Assigns or updates `global_world_size`. | CN: 对 `global_world_size` 进行赋值或更新。
- **L1346** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L1347** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L1348** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L1349** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L1350** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L1351** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1352** EN: Calls `self._post_init` as part of the current workflow. | CN: 在当前流程中调用 `self._post_init`。
- **L1353** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1354** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1355** EN: Defines function `_create_tensor_from_params`. | CN: 定义函数 `_create_tensor_from_params`。
- **L1356** EN: Continues the implementation inside function `_create_tensor_from_params`. | CN: 继续说明函数 `_create_tensor_from_params` 内部的实现。
- **L1357** EN: Continues the implementation inside function `_create_tensor_from_params`. | CN: 继续说明函数 `_create_tensor_from_params` 内部的实现。
- **L1358** EN: Docstring line documenting the function _create_tensor_from_params. | CN: 这是记录 function _create_tensor_from_params 的文档字符串。
- **L1359** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L1360** EN: Assigns or updates `layout`. | CN: 对 `layout` 进行赋值或更新。

### Lines 1361-1373 / 第 1361-1373 行

````python
    requires_grad = tensor_properties.requires_grad
    memory_format = tensor_properties.memory_format
    pin_memory = tensor_properties.pin_memory

    return torch.empty(
        *size,
        dtype=dtype,
        layout=layout,
        device=local_device,
        requires_grad=requires_grad,
        memory_format=memory_format,
        pin_memory=pin_memory,
    )
````

- **L1361** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L1362** EN: Assigns or updates `memory_format`. | CN: 对 `memory_format` 进行赋值或更新。
- **L1363** EN: Assigns or updates `pin_memory`. | CN: 对 `pin_memory` 进行赋值或更新。
- **L1364** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L1365** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L1366** EN: Continues the implementation inside function `_create_tensor_from_params`. | CN: 继续说明函数 `_create_tensor_from_params` 内部的实现。
- **L1367** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L1368** EN: Assigns or updates `layout`. | CN: 对 `layout` 进行赋值或更新。
- **L1369** EN: Assigns or updates `device`. | CN: 对 `device` 进行赋值或更新。
- **L1370** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L1371** EN: Assigns or updates `memory_format`. | CN: 对 `memory_format` 进行赋值或更新。
- **L1372** EN: Assigns or updates `pin_memory`. | CN: 对 `pin_memory` 进行赋值或更新。
- **L1373** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

## Key Concepts / 关键概念

- **EN**: sharding specifications and shard-aware tensor helpers  
  **CN**: 分片规范与分片张量辅助逻辑
- **EN**: process groups  
  **CN**: 进程组
- **EN**: sharding  
  **CN**: 分片
- **EN**: RPC  
  **CN**: RPC
- **EN**: remote references  
  **CN**: 远程引用
- **EN**: collective communication  
  **CN**: 集合通信
- **EN**: all-gather  
  **CN**: all-gather 聚合
- **EN**: distributed tensors  
  **CN**: 分布式张量

## Dependencies / 依赖关系

- **Internal / 内部**: `.metadata`, `.reshard`, `.shard`, `.utils`, `torch.distributed`, `torch.distributed._shard._utils`, `torch.distributed._shard.api`, `torch.distributed._shard.metadata`, `torch.distributed._shard.sharding_spec`, `torch.distributed._shard.sharding_spec._internals`, `torch.distributed._shard.sharding_spec.api`, `torch.distributed.remote_device`
- **PyTorch / PyTorch**: `torch`, `torch._utils`, `torch.utils`
- **Python Stdlib / Python 标准库**: `__future__`, `collections.abc`, `copy`, `dataclasses`, `functools`, `operator`, `threading`, `typing`, `warnings`, `weakref`
- **Third-party / 第三方**: `typing_extensions`

