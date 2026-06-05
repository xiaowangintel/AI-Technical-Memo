# metadata.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/_shard/sharded_tensor/metadata.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on sharding specifications and shard-aware tensor helpers. Its main entry points include MEM_FORMAT_ENCODING, TensorProperties.
- **用途 (CN)**: 该模块聚焦于分片规范与分片张量辅助逻辑，其主要入口包括 MEM_FORMAT_ENCODING, TensorProperties。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs
from dataclasses import dataclass, field
from enum import Enum

import torch
from torch.distributed._shard.metadata import ShardMetadata


class MEM_FORMAT_ENCODING(Enum):
    TORCH_CONTIGUOUS_FORMAT = 0
    TORCH_CHANNELS_LAST = 1
    TORCH_PRESERVE_FORMAT = 2


@dataclass
class TensorProperties:
    """Properties used to create :class:`Tensor`"""

    # Regular tensor fields
    dtype: torch.dtype = field(default=torch.get_default_dtype())
````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Imports selected names from `dataclasses`. | CN: 从 `dataclasses` 导入指定名称。
- **L3** EN: Imports selected names from `enum`. | CN: 从 `enum` 导入指定名称。
- **L4** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L5** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L6** EN: Imports selected names from `torch.distributed._shard.metadata`. | CN: 从 `torch.distributed._shard.metadata` 导入指定名称。
- **L7** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L8** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L9** EN: Defines class `MEM_FORMAT_ENCODING`. | CN: 定义类 `MEM_FORMAT_ENCODING`。
- **L10** EN: Assigns or updates `TORCH_CONTIGUOUS_FORMAT`. | CN: 对 `TORCH_CONTIGUOUS_FORMAT` 进行赋值或更新。
- **L11** EN: Assigns or updates `TORCH_CHANNELS_LAST`. | CN: 对 `TORCH_CHANNELS_LAST` 进行赋值或更新。
- **L12** EN: Assigns or updates `TORCH_PRESERVE_FORMAT`. | CN: 对 `TORCH_PRESERVE_FORMAT` 进行赋值或更新。
- **L13** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L14** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L15** EN: Applies decorator `dataclass` to the following definition. | CN: 将装饰器 `dataclass` 应用于后续定义。
- **L16** EN: Defines class `TensorProperties`. | CN: 定义类 `TensorProperties`。
- **L17** EN: Docstring line documenting the class TensorProperties. | CN: 这是记录 class TensorProperties 的文档字符串。
- **L18** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L19** EN: Keeps the inline comment or directive: Regular tensor fields | CN: 保留这一行注释或指令：Regular tensor fields
- **L20** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。

### Lines 21-40 / 第 21-40 行

````python
    layout: torch.layout = field(default=torch.strided)
    requires_grad: bool = False
    memory_format: torch.memory_format = field(default=torch.contiguous_format)
    pin_memory: bool = False

    def __getstate__(self):
        # Since torch.memory_format cannot be pickled!
        memory_format = self.memory_format
        if memory_format == torch.contiguous_format:
            mem_format_encoding = MEM_FORMAT_ENCODING.TORCH_CONTIGUOUS_FORMAT
        elif memory_format == torch.channels_last:
            mem_format_encoding = MEM_FORMAT_ENCODING.TORCH_CHANNELS_LAST
        elif memory_format == torch.preserve_format:
            mem_format_encoding = MEM_FORMAT_ENCODING.TORCH_PRESERVE_FORMAT
        else:
            raise RuntimeError(f"Invalid torch.memory_format: {memory_format}")

        return (
            self.dtype,
            self.layout,
````

- **L21** EN: Assigns or updates `layout`. | CN: 对 `layout` 进行赋值或更新。
- **L22** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L23** EN: Assigns or updates `memory_format`. | CN: 对 `memory_format` 进行赋值或更新。
- **L24** EN: Assigns or updates `pin_memory`. | CN: 对 `pin_memory` 进行赋值或更新。
- **L25** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L26** EN: Defines function `__getstate__`. | CN: 定义函数 `__getstate__`。
- **L27** EN: Keeps the inline comment or directive: Since torch.memory_format cannot be pickled! | CN: 保留这一行注释或指令：Since torch.memory_format cannot be pickled!
- **L28** EN: Assigns or updates `memory_format`. | CN: 对 `memory_format` 进行赋值或更新。
- **L29** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L30** EN: Assigns or updates `mem_format_encoding`. | CN: 对 `mem_format_encoding` 进行赋值或更新。
- **L31** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L32** EN: Assigns or updates `mem_format_encoding`. | CN: 对 `mem_format_encoding` 进行赋值或更新。
- **L33** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L34** EN: Assigns or updates `mem_format_encoding`. | CN: 对 `mem_format_encoding` 进行赋值或更新。
- **L35** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L36** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L37** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L38** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L39** EN: Continues the implementation inside function `__getstate__`. | CN: 继续说明函数 `__getstate__` 内部的实现。
- **L40** EN: Continues the implementation inside function `__getstate__`. | CN: 继续说明函数 `__getstate__` 内部的实现。

### Lines 41-60 / 第 41-60 行

````python
            self.requires_grad,
            mem_format_encoding,
            self.pin_memory,
        )

    def __setstate__(
        self,
        state,
    ):
        (
            self.dtype,
            self.layout,
            self.requires_grad,
            mem_format_encoding,
            self.pin_memory,
        ) = state

        if mem_format_encoding == MEM_FORMAT_ENCODING.TORCH_CONTIGUOUS_FORMAT:
            memory_format = torch.contiguous_format
        elif mem_format_encoding == MEM_FORMAT_ENCODING.TORCH_CHANNELS_LAST:
````

- **L41** EN: Continues the implementation inside function `__getstate__`. | CN: 继续说明函数 `__getstate__` 内部的实现。
- **L42** EN: Continues the implementation inside function `__getstate__`. | CN: 继续说明函数 `__getstate__` 内部的实现。
- **L43** EN: Continues the implementation inside function `__getstate__`. | CN: 继续说明函数 `__getstate__` 内部的实现。
- **L44** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L45** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L46** EN: Defines function `__setstate__`. | CN: 定义函数 `__setstate__`。
- **L47** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L48** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L49** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L50** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L51** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L52** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L53** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L54** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L55** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L56** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L57** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L58** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L59** EN: Assigns or updates `memory_format`. | CN: 对 `memory_format` 进行赋值或更新。
- **L60** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。

### Lines 61-80 / 第 61-80 行

````python
            memory_format = torch.channels_last
        elif mem_format_encoding == MEM_FORMAT_ENCODING.TORCH_PRESERVE_FORMAT:
            memory_format = torch.preserve_format
        else:
            raise RuntimeError(
                f"Invalid torch.memory_format encoding: {mem_format_encoding}"
            )

        self.memory_format = memory_format

    @staticmethod
    def create_from_tensor(tensor: torch.Tensor) -> "TensorProperties":
        return TensorProperties(
            dtype=tensor.dtype,
            layout=tensor.layout,
            requires_grad=tensor.requires_grad,
            memory_format=torch.contiguous_format,
            pin_memory=tensor.is_pinned(),
        )

````

- **L61** EN: Assigns or updates `memory_format`. | CN: 对 `memory_format` 进行赋值或更新。
- **L62** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L63** EN: Assigns or updates `memory_format`. | CN: 对 `memory_format` 进行赋值或更新。
- **L64** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L65** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L66** EN: Continues the implementation inside function `__setstate__`. | CN: 继续说明函数 `__setstate__` 内部的实现。
- **L67** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L68** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L69** EN: Assigns or updates `self.memory_format`. | CN: 对 `self.memory_format` 进行赋值或更新。
- **L70** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L71** EN: Applies decorator `staticmethod` to the following definition. | CN: 将装饰器 `staticmethod` 应用于后续定义。
- **L72** EN: Defines function `create_from_tensor`. | CN: 定义函数 `create_from_tensor`。
- **L73** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L74** EN: Assigns or updates `dtype`. | CN: 对 `dtype` 进行赋值或更新。
- **L75** EN: Assigns or updates `layout`. | CN: 对 `layout` 进行赋值或更新。
- **L76** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L77** EN: Assigns or updates `memory_format`. | CN: 对 `memory_format` 进行赋值或更新。
- **L78** EN: Assigns or updates `pin_memory`. | CN: 对 `pin_memory` 进行赋值或更新。
- **L79** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L80** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 81-94 / 第 81-94 行

````python

@dataclass
class ShardedTensorMetadata:
    """
    Represents metadata for :class:`ShardedTensor`
    """

    # Metadata about each shard of the Tensor
    shards_metadata: list[ShardMetadata] = field(default_factory=list)

    # Size of each dim of the overall Tensor.
    size: torch.Size = field(default=torch.Size([]))

    tensor_properties: TensorProperties = field(default_factory=TensorProperties)
````

- **L81** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L82** EN: Applies decorator `dataclass` to the following definition. | CN: 将装饰器 `dataclass` 应用于后续定义。
- **L83** EN: Defines class `ShardedTensorMetadata`. | CN: 定义类 `ShardedTensorMetadata`。
- **L84** EN: Starts the docstring for the class ShardedTensorMetadata. | CN: 开始定义 class ShardedTensorMetadata 的文档字符串。
- **L85** EN: Continues the docstring text for the class ShardedTensorMetadata. | CN: 继续补充 class ShardedTensorMetadata 的文档字符串内容。
- **L86** EN: Closes the docstring for the class ShardedTensorMetadata. | CN: 结束 class ShardedTensorMetadata 的文档字符串。
- **L87** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L88** EN: Keeps the inline comment or directive: Metadata about each shard of the Tensor | CN: 保留这一行注释或指令：Metadata about each shard of the Tensor
- **L89** EN: Assigns or updates `shards_metadata`. | CN: 对 `shards_metadata` 进行赋值或更新。
- **L90** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L91** EN: Keeps the inline comment or directive: Size of each dim of the overall Tensor. | CN: 保留这一行注释或指令：Size of each dim of the overall Tensor.
- **L92** EN: Assigns or updates `size`. | CN: 对 `size` 进行赋值或更新。
- **L93** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L94** EN: Assigns or updates `tensor_properties`. | CN: 对 `tensor_properties` 进行赋值或更新。

## Key Concepts / 关键概念

- **EN**: sharding specifications and shard-aware tensor helpers  
  **CN**: 分片规范与分片张量辅助逻辑
- **EN**: sharding  
  **CN**: 分片
- **EN**: distributed tensors  
  **CN**: 分布式张量
- **EN**: Primary classes: MEM_FORMAT_ENCODING, TensorProperties, ShardedTensorMetadata  
  **CN**: 主要类：MEM_FORMAT_ENCODING, TensorProperties, ShardedTensorMetadata

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed._shard.metadata`
- **PyTorch / PyTorch**: `torch`
- **Python Stdlib / Python 标准库**: `dataclasses`, `enum`
- **Third-party / 第三方**: None detected / 未检测到

