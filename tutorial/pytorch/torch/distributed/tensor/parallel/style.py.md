# style.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/tensor/parallel/style.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed tensor layouts, placements, and operators. Its main entry points include ParallelStyle, ColwiseParallel.
- **用途 (CN)**: 该模块聚焦于分布式张量布局、放置与算子逻辑，其主要入口包括 ParallelStyle, ColwiseParallel。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs
# Copyright (c) Meta Platforms, Inc. and affiliates
from abc import ABC, abstractmethod
from functools import partial
from typing import Any

import torch
import torch.nn as nn
from torch.distributed.tensor import (
    DeviceMesh,
    distribute_module,
    distribute_tensor,
    DTensor,
    Replicate,
    Shard,
)
from torch.distributed.tensor.placement_types import Placement


__all__ = [
````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Keeps the inline comment or directive: Copyright (c) Meta Platforms, Inc. and affiliates | CN: 保留这一行注释或指令：Copyright (c) Meta Platforms, Inc. and affiliates
- **L3** EN: Imports selected names from `abc`. | CN: 从 `abc` 导入指定名称。
- **L4** EN: Imports selected names from `functools`. | CN: 从 `functools` 导入指定名称。
- **L5** EN: Imports selected names from `typing`. | CN: 从 `typing` 导入指定名称。
- **L6** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L7** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L8** EN: Imports module dependencies: `torch.nn as nn`. | CN: 导入模块依赖：`torch.nn as nn`。
- **L9** EN: Imports selected names from `torch.distributed.tensor`. | CN: 从 `torch.distributed.tensor` 导入指定名称。
- **L10** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L11** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L12** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L13** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L14** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L15** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L16** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L17** EN: Imports selected names from `torch.distributed.tensor.placement_types`. | CN: 从 `torch.distributed.tensor.placement_types` 导入指定名称。
- **L18** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L19** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L20** EN: Assigns or updates `__all__`. | CN: 对 `__all__` 进行赋值或更新。

### Lines 21-40 / 第 21-40 行

````python
    "ParallelStyle",
    "RowwiseParallel",
    "SequenceParallel",
    "ColwiseParallel",
    "PrepareModuleInput",
    "PrepareModuleInputOutput",
    "PrepareModuleOutput",
]


class ParallelStyle(ABC):
    """
    The parallel style contract defines how the module or submodule should be parallelized.

    It only defines the ``apply`` method for ``parallelize_module`` to use, this allows maximum
    flexibility for different kind of style implementations.
    """

    src_data_rank: int | None = 0

````

- **L21** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L22** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L23** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L24** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L25** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L26** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L27** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L28** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L29** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L30** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L31** EN: Defines class `ParallelStyle`. | CN: 定义类 `ParallelStyle`。
- **L32** EN: Starts the docstring for the class ParallelStyle. | CN: 开始定义 class ParallelStyle 的文档字符串。
- **L33** EN: Continues the docstring text for the class ParallelStyle. | CN: 继续补充 class ParallelStyle 的文档字符串内容。
- **L34** EN: Continues the docstring text for the class ParallelStyle. | CN: 继续补充 class ParallelStyle 的文档字符串内容。
- **L35** EN: Continues the docstring text for the class ParallelStyle. | CN: 继续补充 class ParallelStyle 的文档字符串内容。
- **L36** EN: Continues the docstring text for the class ParallelStyle. | CN: 继续补充 class ParallelStyle 的文档字符串内容。
- **L37** EN: Closes the docstring for the class ParallelStyle. | CN: 结束 class ParallelStyle 的文档字符串。
- **L38** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L39** EN: Assigns or updates `src_data_rank`. | CN: 对 `src_data_rank` 进行赋值或更新。
- **L40** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 41-60 / 第 41-60 行

````python
    @abstractmethod
    def _apply(self, module: nn.Module, device_mesh: DeviceMesh) -> nn.Module: ...


class ColwiseParallel(ParallelStyle):
    """
    Partition a compatible nn.Module in a column-wise fashion. Currently supports nn.Linear and nn.Embedding.
    Users can compose it together with RowwiseParallel to achieve the sharding of more complicated modules.
    (i.e. MLP, Attention)

    Keyword Args:
        input_layouts (Placement, optional):
            The DTensor layout of input tensor for the nn.Module, this is used to annotate the input tensor to
            become a DTensor. If not specified, we assume the input tensor to be replicated.
        output_layouts (Placement, optional):
            The DTensor layout of the output for the nn.Module, this is used to ensure the output of the nn.Module
            with the user desired layout. If not specified, the output tensor is sharded on the last dimension.
        use_local_output (bool, optional):
            Whether to use local :class:`torch.Tensor` instead of :class:`DTensor` for the module output, default: True.
    Returns:
````

- **L41** EN: Applies decorator `abstractmethod` to the following definition. | CN: 将装饰器 `abstractmethod` 应用于后续定义。
- **L42** EN: Defines function `_apply`. | CN: 定义函数 `_apply`。
- **L43** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L44** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L45** EN: Defines class `ColwiseParallel`. | CN: 定义类 `ColwiseParallel`。
- **L46** EN: Starts the docstring for the class ColwiseParallel. | CN: 开始定义 class ColwiseParallel 的文档字符串。
- **L47** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L48** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L49** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L50** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L51** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L52** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L53** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L54** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L55** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L56** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L57** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L58** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L59** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L60** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。

### Lines 61-80 / 第 61-80 行

````python
        A :class:`ParallelStyle` object that represents Colwise sharding of the nn.Module.

    Example::
        >>> # xdoctest: +SKIP(failing)
        >>> from torch.distributed.tensor.parallel import parallelize_module, ColwiseParallel
        >>> from torch.distributed.device_mesh import init_device_mesh
        >>> ...
        >>> m = Model(...)  # m is a nn.Module that contains a "w1" nn.Linear submodule
        >>> tp_mesh = init_device_mesh("cuda", (8,))
        >>>
        >>> # By default, the input of the "w1" Linear will be converted to Replicated DTensor
        >>> # and the output of "w1" will return :class:`torch.Tensor` that shards on the last dim.
        >>>
        >>> sharded_mod = parallelize_module(m, tp_mesh, {"w1": ColwiseParallel()})
        >>> ...

    .. note:: By default ``ColwiseParallel`` output is sharded on the last dimension if the ``output_layouts`` not
        specified, if there're operators that require specific tensor shape (i.e. before the paired ``RowwiseParallel``),
        keep in mind that if the output is sharded the operator might need to be adjusted to the sharded size.
    """
````

- **L61** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L62** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L63** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L64** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L65** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L66** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L67** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L68** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L69** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L70** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L71** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L72** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L73** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L74** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L75** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L76** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L77** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L78** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L79** EN: Continues the docstring text for the class ColwiseParallel. | CN: 继续补充 class ColwiseParallel 的文档字符串内容。
- **L80** EN: Closes the docstring for the class ColwiseParallel. | CN: 结束 class ColwiseParallel 的文档字符串。

### Lines 81-100 / 第 81-100 行

````python

    def __init__(
        self,
        *,
        input_layouts: Placement | None = None,
        output_layouts: Placement | None = None,
        use_local_output: bool = True,
    ):
        super().__init__()
        self.input_layouts = (input_layouts or Replicate(),)
        self.output_layouts = (output_layouts or Shard(-1),)
        # colwise linear runtime sharding (desired sharding):
        # 1. requires replicate input
        # 2. shard output on last dim
        self.desired_input_layouts = (Replicate(),)
        self.use_local_output = use_local_output

    @staticmethod
    def _prepare_input_fn(
        input_layouts, desired_input_layouts, mod, inputs, device_mesh
````

- **L81** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L82** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L83** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L84** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L85** EN: Assigns or updates `input_layouts`. | CN: 对 `input_layouts` 进行赋值或更新。
- **L86** EN: Assigns or updates `output_layouts`. | CN: 对 `output_layouts` 进行赋值或更新。
- **L87** EN: Assigns or updates `use_local_output`. | CN: 对 `use_local_output` 进行赋值或更新。
- **L88** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L89** EN: Calls `super` as part of the current workflow. | CN: 在当前流程中调用 `super`。
- **L90** EN: Assigns or updates `self.input_layouts`. | CN: 对 `self.input_layouts` 进行赋值或更新。
- **L91** EN: Assigns or updates `self.output_layouts`. | CN: 对 `self.output_layouts` 进行赋值或更新。
- **L92** EN: Keeps the inline comment or directive: colwise linear runtime sharding (desired sharding): | CN: 保留这一行注释或指令：colwise linear runtime sharding (desired sharding):
- **L93** EN: Keeps the inline comment or directive: 1. requires replicate input | CN: 保留这一行注释或指令：1. requires replicate input
- **L94** EN: Keeps the inline comment or directive: 2. shard output on last dim | CN: 保留这一行注释或指令：2. shard output on last dim
- **L95** EN: Assigns or updates `self.desired_input_layouts`. | CN: 对 `self.desired_input_layouts` 进行赋值或更新。
- **L96** EN: Assigns or updates `self.use_local_output`. | CN: 对 `self.use_local_output` 进行赋值或更新。
- **L97** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L98** EN: Applies decorator `staticmethod` to the following definition. | CN: 将装饰器 `staticmethod` 应用于后续定义。
- **L99** EN: Defines function `_prepare_input_fn`. | CN: 定义函数 `_prepare_input_fn`。
- **L100** EN: Continues the implementation inside function `_prepare_input_fn`. | CN: 继续说明函数 `_prepare_input_fn` 内部的实现。

### Lines 101-120 / 第 101-120 行

````python
    ):
        # TODO: figure out dynamo support for instance method and switch this to instance method

        # annotate module input placements/sharding with input_layouts
        input_tensor = inputs[0]
        if not isinstance(input_tensor, DTensor):
            input_tensor = DTensor.from_local(
                input_tensor,
                device_mesh,
                input_layouts,
                run_check=False,
            )

        # transform the input layouts to the desired layouts of ColwiseParallel
        if input_layouts != desired_input_layouts:
            input_tensor = input_tensor.redistribute(
                placements=desired_input_layouts, async_op=True
            )
        return input_tensor

````

- **L101** EN: Continues the implementation inside function `_prepare_input_fn`. | CN: 继续说明函数 `_prepare_input_fn` 内部的实现。
- **L102** EN: Keeps the inline comment or directive: TODO: figure out dynamo support for instance method and switch this to instance  | CN: 保留这一行注释或指令：TODO: figure out dynamo support for instance method and switch this to instance 
- **L103** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L104** EN: Keeps the inline comment or directive: annotate module input placements/sharding with input_layouts | CN: 保留这一行注释或指令：annotate module input placements/sharding with input_layouts
- **L105** EN: Assigns or updates `input_tensor`. | CN: 对 `input_tensor` 进行赋值或更新。
- **L106** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L107** EN: Assigns or updates `input_tensor`. | CN: 对 `input_tensor` 进行赋值或更新。
- **L108** EN: Continues the implementation inside function `_prepare_input_fn`. | CN: 继续说明函数 `_prepare_input_fn` 内部的实现。
- **L109** EN: Continues the implementation inside function `_prepare_input_fn`. | CN: 继续说明函数 `_prepare_input_fn` 内部的实现。
- **L110** EN: Continues the implementation inside function `_prepare_input_fn`. | CN: 继续说明函数 `_prepare_input_fn` 内部的实现。
- **L111** EN: Assigns or updates `run_check`. | CN: 对 `run_check` 进行赋值或更新。
- **L112** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L113** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L114** EN: Keeps the inline comment or directive: transform the input layouts to the desired layouts of ColwiseParallel | CN: 保留这一行注释或指令：transform the input layouts to the desired layouts of ColwiseParallel
- **L115** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L116** EN: Assigns or updates `input_tensor`. | CN: 对 `input_tensor` 进行赋值或更新。
- **L117** EN: Assigns or updates `placements`. | CN: 对 `placements` 进行赋值或更新。
- **L118** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L119** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L120** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 121-140 / 第 121-140 行

````python
    def _partition_linear_fn(self, name, module, device_mesh):
        # colwise shard weight/bias to Shard(0), weight be Shard(0)
        # means Colwise as Linear is input * weight^T + bias, where
        # weight would become Shard(1)
        for name, param in module.named_parameters():
            dist_param = nn.Parameter(
                distribute_tensor(
                    param, device_mesh, [Shard(0)], src_data_rank=self.src_data_rank
                ),
                requires_grad=param.requires_grad,
            )
            module.register_parameter(name, dist_param)

    def _partition_embedding_fn(self, name, module, device_mesh):
        # colwise shard embedding.weight is straight forward as Shard(1)
        for name, param in module.named_parameters():
            dist_param = nn.Parameter(
                distribute_tensor(
                    param, device_mesh, [Shard(1)], src_data_rank=self.src_data_rank
                ),
````

- **L121** EN: Defines function `_partition_linear_fn`. | CN: 定义函数 `_partition_linear_fn`。
- **L122** EN: Keeps the inline comment or directive: colwise shard weight/bias to Shard(0), weight be Shard(0) | CN: 保留这一行注释或指令：colwise shard weight/bias to Shard(0), weight be Shard(0)
- **L123** EN: Keeps the inline comment or directive: means Colwise as Linear is input * weight^T + bias, where | CN: 保留这一行注释或指令：means Colwise as Linear is input * weight^T + bias, where
- **L124** EN: Keeps the inline comment or directive: weight would become Shard(1) | CN: 保留这一行注释或指令：weight would become Shard(1)
- **L125** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L126** EN: Assigns or updates `dist_param`. | CN: 对 `dist_param` 进行赋值或更新。
- **L127** EN: Calls `distribute_tensor` as part of the current workflow. | CN: 在当前流程中调用 `distribute_tensor`。
- **L128** EN: Continues the implementation inside function `_partition_linear_fn`. | CN: 继续说明函数 `_partition_linear_fn` 内部的实现。
- **L129** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L130** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L131** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L132** EN: Calls `module.register_parameter` as part of the current workflow. | CN: 在当前流程中调用 `module.register_parameter`。
- **L133** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L134** EN: Defines function `_partition_embedding_fn`. | CN: 定义函数 `_partition_embedding_fn`。
- **L135** EN: Keeps the inline comment or directive: colwise shard embedding.weight is straight forward as Shard(1) | CN: 保留这一行注释或指令：colwise shard embedding.weight is straight forward as Shard(1)
- **L136** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L137** EN: Assigns or updates `dist_param`. | CN: 对 `dist_param` 进行赋值或更新。
- **L138** EN: Calls `distribute_tensor` as part of the current workflow. | CN: 在当前流程中调用 `distribute_tensor`。
- **L139** EN: Continues the implementation inside function `_partition_embedding_fn`. | CN: 继续说明函数 `_partition_embedding_fn` 内部的实现。
- **L140** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 141-160 / 第 141-160 行

````python
                requires_grad=param.requires_grad,
            )
            module.register_parameter(name, dist_param)

    @staticmethod
    def _prepare_output_fn(output_layouts, use_local_output, mod, outputs, device_mesh):
        # outputs is a shard on last dimension DTensor, i.e. Shard(-1)
        if outputs.placements != output_layouts:
            outputs = outputs.redistribute(placements=output_layouts, async_op=True)
        # back to local tensor
        return outputs.to_local() if use_local_output else outputs

    def _apply(self, module: nn.Module, device_mesh: DeviceMesh) -> nn.Module:
        if isinstance(module, nn.Linear):
            partition_fn = self._partition_linear_fn
        elif isinstance(module, nn.Embedding):
            partition_fn = self._partition_embedding_fn
        else:
            raise NotImplementedError(
                "ColwiseParallel currently only support nn.Linear and nn.Embedding!"
````

- **L141** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L142** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L143** EN: Calls `module.register_parameter` as part of the current workflow. | CN: 在当前流程中调用 `module.register_parameter`。
- **L144** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L145** EN: Applies decorator `staticmethod` to the following definition. | CN: 将装饰器 `staticmethod` 应用于后续定义。
- **L146** EN: Defines function `_prepare_output_fn`. | CN: 定义函数 `_prepare_output_fn`。
- **L147** EN: Keeps the inline comment or directive: outputs is a shard on last dimension DTensor, i.e. Shard(-1) | CN: 保留这一行注释或指令：outputs is a shard on last dimension DTensor, i.e. Shard(-1)
- **L148** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L149** EN: Assigns or updates `outputs`. | CN: 对 `outputs` 进行赋值或更新。
- **L150** EN: Keeps the inline comment or directive: back to local tensor | CN: 保留这一行注释或指令：back to local tensor
- **L151** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L152** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L153** EN: Defines function `_apply`. | CN: 定义函数 `_apply`。
- **L154** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L155** EN: Assigns or updates `partition_fn`. | CN: 对 `partition_fn` 进行赋值或更新。
- **L156** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L157** EN: Assigns or updates `partition_fn`. | CN: 对 `partition_fn` 进行赋值或更新。
- **L158** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L159** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L160** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。

### Lines 161-180 / 第 161-180 行

````python
            )

        return distribute_module(
            module,
            device_mesh,
            partition_fn,
            partial(
                self._prepare_input_fn,
                self.input_layouts,
                self.desired_input_layouts,
            ),
            partial(
                self._prepare_output_fn, self.output_layouts, self.use_local_output
            ),
        )

    def __repr__(self) -> str:
        tmpstr = self.__class__.__name__ + "("
        tmpstr += f"input_layouts={self.input_layouts}, "
        tmpstr += f"output_layouts={self.output_layouts}, "
````

- **L161** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L162** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L163** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L164** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L165** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L166** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L167** EN: Calls `partial` as part of the current workflow. | CN: 在当前流程中调用 `partial`。
- **L168** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L169** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L170** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L171** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L172** EN: Calls `partial` as part of the current workflow. | CN: 在当前流程中调用 `partial`。
- **L173** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L174** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L175** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L176** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L177** EN: Defines function `__repr__`. | CN: 定义函数 `__repr__`。
- **L178** EN: Assigns or updates `tmpstr`. | CN: 对 `tmpstr` 进行赋值或更新。
- **L179** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L180** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。

### Lines 181-200 / 第 181-200 行

````python
        tmpstr += f"use_local_output={self.use_local_output}"
        tmpstr += ")"
        return tmpstr


class RowwiseParallel(ParallelStyle):
    """
    Partition a compatible nn.Module in a row-wise fashion. Currently supports nn.Linear and nn.Embedding.
    Users can compose it with ColwiseParallel to achieve the sharding of more complicated modules.
    (i.e. MLP, Attention)

    Keyword Args:
        input_layouts (Placement, optional):
            The DTensor layout of input tensor for the nn.Module, this is used to annotate the input tensor to
            become a DTensor. If not specified, we assume the input tensor to be sharded on the last dimension.
        output_layouts (Placement, optional):
            The DTensor layout of the output for the nn.Module, this is used to ensure the output of the nn.Module
            with the user desired layout. If not specified, the output tensor is replicated.
        use_local_output (bool, optional):
            Whether to use local :class:`torch.Tensor` instead of :class:`DTensor` for the module output, default: True.
````

- **L181** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L182** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L183** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L184** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L185** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L186** EN: Defines class `RowwiseParallel`. | CN: 定义类 `RowwiseParallel`。
- **L187** EN: Starts the docstring for the class RowwiseParallel. | CN: 开始定义 class RowwiseParallel 的文档字符串。
- **L188** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L189** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L190** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L191** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L192** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L193** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L194** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L195** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L196** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L197** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L198** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L199** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L200** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。

### Lines 201-220 / 第 201-220 行

````python
    Returns:
        A :class:`ParallelStyle` object that represents Rowwise sharding of the nn.Module.

    Example::
        >>> # xdoctest: +SKIP(failing)
        >>> from torch.distributed.tensor.parallel import parallelize_module, RowwiseParallel
        >>> from torch.distributed.device_mesh import init_device_mesh
        >>> ...
        >>> m = Model(...)  # m is a nn.Module that contains a "w2" nn.Linear submodule
        >>> tp_mesh = init_device_mesh("cuda", (8,))
        >>>
        >>> # By default, the input of the "w2" Linear will be converted to DTensor that shards on the last dim
        >>> # and the output of "w2" will return a replicated :class:`torch.Tensor`.
        >>>
        >>> sharded_mod = parallelize_module(m, tp_mesh, {"w2": RowwiseParallel()}),
        >>> ...
    """

    def __init__(
        self,
````

- **L201** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L202** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L203** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L204** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L205** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L206** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L207** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L208** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L209** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L210** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L211** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L212** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L213** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L214** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L215** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L216** EN: Continues the docstring text for the class RowwiseParallel. | CN: 继续补充 class RowwiseParallel 的文档字符串内容。
- **L217** EN: Closes the docstring for the class RowwiseParallel. | CN: 结束 class RowwiseParallel 的文档字符串。
- **L218** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L219** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L220** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。

### Lines 221-240 / 第 221-240 行

````python
        *,
        input_layouts: Placement | None = None,
        output_layouts: Placement | None = None,
        use_local_output: bool = True,
    ):
        super().__init__()
        self.input_layouts = (input_layouts or Shard(-1),)
        self.output_layouts = (output_layouts or Replicate(),)
        self.use_local_output = use_local_output

    @staticmethod
    def _prepare_input_fn(
        input_layouts, desired_input_layouts, mod, inputs, device_mesh
    ):
        input_tensor = inputs[0]
        if not isinstance(input_tensor, DTensor):
            input_tensor = DTensor.from_local(
                input_tensor,
                device_mesh,
                input_layouts,
````

- **L221** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L222** EN: Assigns or updates `input_layouts`. | CN: 对 `input_layouts` 进行赋值或更新。
- **L223** EN: Assigns or updates `output_layouts`. | CN: 对 `output_layouts` 进行赋值或更新。
- **L224** EN: Assigns or updates `use_local_output`. | CN: 对 `use_local_output` 进行赋值或更新。
- **L225** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L226** EN: Calls `super` as part of the current workflow. | CN: 在当前流程中调用 `super`。
- **L227** EN: Assigns or updates `self.input_layouts`. | CN: 对 `self.input_layouts` 进行赋值或更新。
- **L228** EN: Assigns or updates `self.output_layouts`. | CN: 对 `self.output_layouts` 进行赋值或更新。
- **L229** EN: Assigns or updates `self.use_local_output`. | CN: 对 `self.use_local_output` 进行赋值或更新。
- **L230** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L231** EN: Applies decorator `staticmethod` to the following definition. | CN: 将装饰器 `staticmethod` 应用于后续定义。
- **L232** EN: Defines function `_prepare_input_fn`. | CN: 定义函数 `_prepare_input_fn`。
- **L233** EN: Continues the implementation inside function `_prepare_input_fn`. | CN: 继续说明函数 `_prepare_input_fn` 内部的实现。
- **L234** EN: Continues the implementation inside function `_prepare_input_fn`. | CN: 继续说明函数 `_prepare_input_fn` 内部的实现。
- **L235** EN: Assigns or updates `input_tensor`. | CN: 对 `input_tensor` 进行赋值或更新。
- **L236** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L237** EN: Assigns or updates `input_tensor`. | CN: 对 `input_tensor` 进行赋值或更新。
- **L238** EN: Continues the implementation inside function `_prepare_input_fn`. | CN: 继续说明函数 `_prepare_input_fn` 内部的实现。
- **L239** EN: Continues the implementation inside function `_prepare_input_fn`. | CN: 继续说明函数 `_prepare_input_fn` 内部的实现。
- **L240** EN: Continues the implementation inside function `_prepare_input_fn`. | CN: 继续说明函数 `_prepare_input_fn` 内部的实现。

### Lines 241-260 / 第 241-260 行

````python
                run_check=False,
            )

        if input_layouts != desired_input_layouts:
            input_tensor = input_tensor.redistribute(
                placements=desired_input_layouts, async_op=True
            )
        return input_tensor

    def _partition_linear_fn(self, name, module, device_mesh):
        # Rowwise shard weight to Shard(1), bias to Replicate(), weight be Shard(1)
        # means Rowwise as nn.Linear is input * weight^T + bias, where
        # weight would become Shard(0)
        module.register_parameter(
            "weight",
            nn.Parameter(
                distribute_tensor(
                    module.weight,
                    device_mesh,
                    [Shard(1)],
````

- **L241** EN: Assigns or updates `run_check`. | CN: 对 `run_check` 进行赋值或更新。
- **L242** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L243** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L244** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L245** EN: Assigns or updates `input_tensor`. | CN: 对 `input_tensor` 进行赋值或更新。
- **L246** EN: Assigns or updates `placements`. | CN: 对 `placements` 进行赋值或更新。
- **L247** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L248** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L249** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L250** EN: Defines function `_partition_linear_fn`. | CN: 定义函数 `_partition_linear_fn`。
- **L251** EN: Keeps the inline comment or directive: Rowwise shard weight to Shard(1), bias to Replicate(), weight be Shard(1) | CN: 保留这一行注释或指令：Rowwise shard weight to Shard(1), bias to Replicate(), weight be Shard(1)
- **L252** EN: Keeps the inline comment or directive: means Rowwise as nn.Linear is input * weight^T + bias, where | CN: 保留这一行注释或指令：means Rowwise as nn.Linear is input * weight^T + bias, where
- **L253** EN: Keeps the inline comment or directive: weight would become Shard(0) | CN: 保留这一行注释或指令：weight would become Shard(0)
- **L254** EN: Calls `module.register_parameter` as part of the current workflow. | CN: 在当前流程中调用 `module.register_parameter`。
- **L255** EN: Continues the implementation inside function `_partition_linear_fn`. | CN: 继续说明函数 `_partition_linear_fn` 内部的实现。
- **L256** EN: Calls `nn.Parameter` as part of the current workflow. | CN: 在当前流程中调用 `nn.Parameter`。
- **L257** EN: Calls `distribute_tensor` as part of the current workflow. | CN: 在当前流程中调用 `distribute_tensor`。
- **L258** EN: Continues the implementation inside function `_partition_linear_fn`. | CN: 继续说明函数 `_partition_linear_fn` 内部的实现。
- **L259** EN: Continues the implementation inside function `_partition_linear_fn`. | CN: 继续说明函数 `_partition_linear_fn` 内部的实现。
- **L260** EN: Continues the implementation inside function `_partition_linear_fn`. | CN: 继续说明函数 `_partition_linear_fn` 内部的实现。

### Lines 261-280 / 第 261-280 行

````python
                    src_data_rank=self.src_data_rank,
                ),
                requires_grad=module.weight.requires_grad,
            ),
        )
        if getattr(module, "bias", None) is not None:
            # The Linear module has bias
            module.register_parameter(
                "bias",
                nn.Parameter(
                    distribute_tensor(
                        module.bias,
                        device_mesh,
                        [Replicate()],
                        src_data_rank=self.src_data_rank,
                    ),
                    requires_grad=module.bias.requires_grad,
                ),
            )

````

- **L261** EN: Assigns or updates `src_data_rank`. | CN: 对 `src_data_rank` 进行赋值或更新。
- **L262** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L263** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L264** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L265** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L266** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L267** EN: Keeps the inline comment or directive: The Linear module has bias | CN: 保留这一行注释或指令：The Linear module has bias
- **L268** EN: Calls `module.register_parameter` as part of the current workflow. | CN: 在当前流程中调用 `module.register_parameter`。
- **L269** EN: Continues the implementation inside function `_partition_linear_fn`. | CN: 继续说明函数 `_partition_linear_fn` 内部的实现。
- **L270** EN: Calls `nn.Parameter` as part of the current workflow. | CN: 在当前流程中调用 `nn.Parameter`。
- **L271** EN: Calls `distribute_tensor` as part of the current workflow. | CN: 在当前流程中调用 `distribute_tensor`。
- **L272** EN: Continues the implementation inside function `_partition_linear_fn`. | CN: 继续说明函数 `_partition_linear_fn` 内部的实现。
- **L273** EN: Continues the implementation inside function `_partition_linear_fn`. | CN: 继续说明函数 `_partition_linear_fn` 内部的实现。
- **L274** EN: Continues the implementation inside function `_partition_linear_fn`. | CN: 继续说明函数 `_partition_linear_fn` 内部的实现。
- **L275** EN: Assigns or updates `src_data_rank`. | CN: 对 `src_data_rank` 进行赋值或更新。
- **L276** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L277** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L278** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L279** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L280** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 281-300 / 第 281-300 行

````python
    def _partition_embedding_fn(self, name, module, device_mesh):
        # rowwise shard embedding.weight is Shard(0)
        for name, param in module.named_parameters():
            dist_param = nn.Parameter(
                distribute_tensor(
                    param, device_mesh, [Shard(0)], src_data_rank=self.src_data_rank
                ),
                requires_grad=param.requires_grad,
            )
            module.register_parameter(name, dist_param)

    @staticmethod
    def _prepare_output_fn(output_layouts, use_local_output, mod, outputs, device_mesh):
        # Rowwise sharding produces partial output, depending on output layouts:
        # 1. to replicate -> allreduce
        # 2. to shard -> reduce_scatter
        if outputs.placements != output_layouts:
            outputs = outputs.redistribute(placements=output_layouts, async_op=True)
        # back to local tensor if use_local_output is True
        return outputs.to_local() if use_local_output else outputs
````

- **L281** EN: Defines function `_partition_embedding_fn`. | CN: 定义函数 `_partition_embedding_fn`。
- **L282** EN: Keeps the inline comment or directive: rowwise shard embedding.weight is Shard(0) | CN: 保留这一行注释或指令：rowwise shard embedding.weight is Shard(0)
- **L283** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L284** EN: Assigns or updates `dist_param`. | CN: 对 `dist_param` 进行赋值或更新。
- **L285** EN: Calls `distribute_tensor` as part of the current workflow. | CN: 在当前流程中调用 `distribute_tensor`。
- **L286** EN: Continues the implementation inside function `_partition_embedding_fn`. | CN: 继续说明函数 `_partition_embedding_fn` 内部的实现。
- **L287** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L288** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L289** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L290** EN: Calls `module.register_parameter` as part of the current workflow. | CN: 在当前流程中调用 `module.register_parameter`。
- **L291** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L292** EN: Applies decorator `staticmethod` to the following definition. | CN: 将装饰器 `staticmethod` 应用于后续定义。
- **L293** EN: Defines function `_prepare_output_fn`. | CN: 定义函数 `_prepare_output_fn`。
- **L294** EN: Keeps the inline comment or directive: Rowwise sharding produces partial output, depending on output layouts: | CN: 保留这一行注释或指令：Rowwise sharding produces partial output, depending on output layouts:
- **L295** EN: Keeps the inline comment or directive: 1. to replicate -> allreduce | CN: 保留这一行注释或指令：1. to replicate -> allreduce
- **L296** EN: Keeps the inline comment or directive: 2. to shard -> reduce_scatter | CN: 保留这一行注释或指令：2. to shard -> reduce_scatter
- **L297** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L298** EN: Assigns or updates `outputs`. | CN: 对 `outputs` 进行赋值或更新。
- **L299** EN: Keeps the inline comment or directive: back to local tensor if use_local_output is True | CN: 保留这一行注释或指令：back to local tensor if use_local_output is True
- **L300** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

### Lines 301-320 / 第 301-320 行

````python

    def _apply(self, module: nn.Module, device_mesh: DeviceMesh) -> nn.Module:
        if isinstance(module, nn.Linear):
            partition_fn = self._partition_linear_fn
            # rowwise linear runtime sharding requires input tensor shard on last dim
            self.desired_input_layouts: tuple[Placement, ...] = (Shard(-1),)
        elif isinstance(module, nn.Embedding):
            partition_fn = self._partition_embedding_fn
            # rowwise embedding runtime sharding requires input tensor replicated
            self.desired_input_layouts = (Replicate(),)
        else:
            raise NotImplementedError(
                "RowwiseParallel currently only support nn.Linear and nn.Embedding!"
            )

        return distribute_module(
            module,
            device_mesh,
            partition_fn,
            partial(
````

- **L301** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L302** EN: Defines function `_apply`. | CN: 定义函数 `_apply`。
- **L303** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L304** EN: Assigns or updates `partition_fn`. | CN: 对 `partition_fn` 进行赋值或更新。
- **L305** EN: Keeps the inline comment or directive: rowwise linear runtime sharding requires input tensor shard on last dim | CN: 保留这一行注释或指令：rowwise linear runtime sharding requires input tensor shard on last dim
- **L306** EN: Assigns or updates `self.desired_input_layouts`. | CN: 对 `self.desired_input_layouts` 进行赋值或更新。
- **L307** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L308** EN: Assigns or updates `partition_fn`. | CN: 对 `partition_fn` 进行赋值或更新。
- **L309** EN: Keeps the inline comment or directive: rowwise embedding runtime sharding requires input tensor replicated | CN: 保留这一行注释或指令：rowwise embedding runtime sharding requires input tensor replicated
- **L310** EN: Assigns or updates `self.desired_input_layouts`. | CN: 对 `self.desired_input_layouts` 进行赋值或更新。
- **L311** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L312** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L313** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L314** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L315** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L316** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L317** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L318** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L319** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L320** EN: Calls `partial` as part of the current workflow. | CN: 在当前流程中调用 `partial`。

### Lines 321-340 / 第 321-340 行

````python
                self._prepare_input_fn,
                self.input_layouts,
                self.desired_input_layouts,
            ),
            partial(
                self._prepare_output_fn, self.output_layouts, self.use_local_output
            ),
        )

    def __repr__(self) -> str:
        tmpstr = self.__class__.__name__ + "("
        tmpstr += f"input_layouts={self.input_layouts}, "
        tmpstr += f"output_layouts={self.output_layouts}, "
        tmpstr += f"use_local_output={self.use_local_output}"
        tmpstr += ")"
        return tmpstr


class SequenceParallel(ParallelStyle):
    """
````

- **L321** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L322** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L323** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L324** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L325** EN: Calls `partial` as part of the current workflow. | CN: 在当前流程中调用 `partial`。
- **L326** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L327** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L328** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L329** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L330** EN: Defines function `__repr__`. | CN: 定义函数 `__repr__`。
- **L331** EN: Assigns or updates `tmpstr`. | CN: 对 `tmpstr` 进行赋值或更新。
- **L332** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L333** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L334** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L335** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L336** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L337** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L338** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L339** EN: Defines class `SequenceParallel`. | CN: 定义类 `SequenceParallel`。
- **L340** EN: Starts the docstring for the class SequenceParallel. | CN: 开始定义 class SequenceParallel 的文档字符串。

### Lines 341-360 / 第 341-360 行

````python
    SequenceParallel replicates a compatible ``nn.Module`` parameters and runs the sharded computation with
    input sharded on the sequence dimension. This currently supports ``nn.LayerNorm``, ``nn.Dropout``, and the
    `RMSNorm python implementation <https://github.com/facebookresearch/llama/blob/main/llama/model.py#L34>`__

    This style implements the operation that is described in the paper
    `Reducing Activation Recomputation in Large Transformer Models <https://arxiv.org/abs/2205.05198>`__

    If the input passed in to this ``nn.Module`` is a :class:`torch.Tensor`, it assumes that the input is already sharded
    on the sequence dimension and converts the input to a :class:`DTensor` sharded on the sequence dimension. If the input
    passed in to this ``nn.Module`` is already a :class:`DTensor` but is not sharded on the sequence dimension, it would
    redistribute the input to be sharded on the sequence dimension.

    The output of the ``nn.Module`` will be sharded on the sequence dimension.

    Keyword Args:
        sequence_dim (int, optional):
            The sequence dimension of the input tensor for the ``nn.Module``, this is used to annotate the input tensor to
            become a DTensor that is sharded on the sequence dimension, default: 1.
        use_local_output (bool, optional):
            Whether to use local :class:`torch.Tensor` instead of :class:`DTensor` for the module output, default: False.
````

- **L341** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L342** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L343** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L344** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L345** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L346** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L347** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L348** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L349** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L350** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L351** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L352** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L353** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L354** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L355** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L356** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L357** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L358** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L359** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L360** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。

### Lines 361-380 / 第 361-380 行

````python
    Returns:
        A :class:`ParallelStyle` object that represents Sequence Parallel of the ``nn.Module``.

    Example::
        >>> # xdoctest: +SKIP(failing)
        >>> from torch.distributed.tensor.parallel import parallelize_module, SequenceParallel
        >>> from torch.distributed.device_mesh import init_device_mesh
        >>> ...
        >>> m = Model(...)  # m is a nn.Module that contains a "norm" nn.LayerNorm submodule
        >>> tp_mesh = init_device_mesh("cuda", (8,))
        >>>
        >>> # By default, the input of the "norm" will be converted to DTensor that shards on the sequence dim
        >>> # and the output of "norm" will return a sharded on sequence dimension :class:`DTensor`.
        >>>
        >>> sharded_mod = parallelize_module(m, tp_mesh, {"norm": SequenceParallel()}),
        >>> ...

    .. note:: SequenceParallel style assumes ones initialization if there are weights in the nn.Module (i.e.
        ``nn.LayerNorm`` or ``RMSNorm``, and they by default have ones initialization). If you have custom
        inits for the weights on those modules, you need to broadcast the weights before/after parallelizing
````

- **L361** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L362** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L363** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L364** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L365** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L366** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L367** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L368** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L369** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L370** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L371** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L372** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L373** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L374** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L375** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L376** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L377** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L378** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L379** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L380** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。

### Lines 381-400 / 第 381-400 行

````python
        to ensure that they are replicated.
    """

    def __init__(self, *, sequence_dim: int = 1, use_local_output: bool = False):
        super().__init__()
        self.sequence_sharding = (Shard(sequence_dim),)
        self.use_local_output = use_local_output

    def _replicate_module_fn(
        self, name: str, module: nn.Module, device_mesh: DeviceMesh
    ):
        for p_name, param in module.named_parameters():
            # simple replication with fixed ones_ init from LayerNorm/RMSNorm, which allow
            # us to simply just use from_local
            replicated_param = torch.nn.Parameter(
                DTensor.from_local(param, device_mesh, [Replicate()], run_check=False)
            )
            module.register_parameter(p_name, replicated_param)

    @staticmethod
````

- **L381** EN: Continues the docstring text for the class SequenceParallel. | CN: 继续补充 class SequenceParallel 的文档字符串内容。
- **L382** EN: Closes the docstring for the class SequenceParallel. | CN: 结束 class SequenceParallel 的文档字符串。
- **L383** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L384** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L385** EN: Calls `super` as part of the current workflow. | CN: 在当前流程中调用 `super`。
- **L386** EN: Assigns or updates `self.sequence_sharding`. | CN: 对 `self.sequence_sharding` 进行赋值或更新。
- **L387** EN: Assigns or updates `self.use_local_output`. | CN: 对 `self.use_local_output` 进行赋值或更新。
- **L388** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L389** EN: Defines function `_replicate_module_fn`. | CN: 定义函数 `_replicate_module_fn`。
- **L390** EN: Continues the implementation inside function `_replicate_module_fn`. | CN: 继续说明函数 `_replicate_module_fn` 内部的实现。
- **L391** EN: Continues the implementation inside function `_replicate_module_fn`. | CN: 继续说明函数 `_replicate_module_fn` 内部的实现。
- **L392** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L393** EN: Keeps the inline comment or directive: simple replication with fixed ones_ init from LayerNorm/RMSNorm, which allow | CN: 保留这一行注释或指令：simple replication with fixed ones_ init from LayerNorm/RMSNorm, which allow
- **L394** EN: Keeps the inline comment or directive: us to simply just use from_local | CN: 保留这一行注释或指令：us to simply just use from_local
- **L395** EN: Assigns or updates `replicated_param`. | CN: 对 `replicated_param` 进行赋值或更新。
- **L396** EN: Calls `DTensor.from_local` as part of the current workflow. | CN: 在当前流程中调用 `DTensor.from_local`。
- **L397** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L398** EN: Calls `module.register_parameter` as part of the current workflow. | CN: 在当前流程中调用 `module.register_parameter`。
- **L399** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L400** EN: Applies decorator `staticmethod` to the following definition. | CN: 将装饰器 `staticmethod` 应用于后续定义。

### Lines 401-420 / 第 401-420 行

````python
    def _prepare_input_fn(sequence_sharding, mod, inputs, device_mesh):
        input_tensor = inputs[0]
        if isinstance(input_tensor, DTensor):
            # if the passed in input DTensor is not sharded on the sequence dim, we need to redistribute it
            if input_tensor.placements != sequence_sharding:
                input_tensor = input_tensor.redistribute(
                    placements=sequence_sharding, async_op=True
                )
            return input_tensor
        elif isinstance(input_tensor, torch.Tensor):
            # assume the input passed in already sharded on the sequence dim and create the DTensor
            return DTensor.from_local(
                input_tensor, device_mesh, sequence_sharding, run_check=False
            )
        else:
            raise ValueError(
                f"expecting input of {mod} to be a torch.Tensor or DTensor, but got {input_tensor}"
            )

    @staticmethod
````

- **L401** EN: Defines function `_prepare_input_fn`. | CN: 定义函数 `_prepare_input_fn`。
- **L402** EN: Assigns or updates `input_tensor`. | CN: 对 `input_tensor` 进行赋值或更新。
- **L403** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L404** EN: Keeps the inline comment or directive: if the passed in input DTensor is not sharded on the sequence dim, we need to re | CN: 保留这一行注释或指令：if the passed in input DTensor is not sharded on the sequence dim, we need to re
- **L405** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L406** EN: Assigns or updates `input_tensor`. | CN: 对 `input_tensor` 进行赋值或更新。
- **L407** EN: Assigns or updates `placements`. | CN: 对 `placements` 进行赋值或更新。
- **L408** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L409** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L410** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L411** EN: Keeps the inline comment or directive: assume the input passed in already sharded on the sequence dim and create the DT | CN: 保留这一行注释或指令：assume the input passed in already sharded on the sequence dim and create the DT
- **L412** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L413** EN: Assigns or updates `input_tensor, device_mesh, sequence_sharding, run_check`. | CN: 对 `input_tensor, device_mesh, sequence_sharding, run_check` 进行赋值或更新。
- **L414** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L415** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L416** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L417** EN: Continues the implementation inside function `_prepare_input_fn`. | CN: 继续说明函数 `_prepare_input_fn` 内部的实现。
- **L418** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L419** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L420** EN: Applies decorator `staticmethod` to the following definition. | CN: 将装饰器 `staticmethod` 应用于后续定义。

### Lines 421-440 / 第 421-440 行

````python
    def _prepare_output_fn(use_local_output, mod, outputs, device_mesh):
        return outputs.to_local() if use_local_output else outputs

    def _apply(self, module: nn.Module, device_mesh: DeviceMesh) -> nn.Module:
        return distribute_module(
            module,
            device_mesh,
            self._replicate_module_fn,
            partial(self._prepare_input_fn, self.sequence_sharding),
            partial(self._prepare_output_fn, self.use_local_output),
        )

    def __repr__(self) -> str:
        tmpstr = self.__class__.__name__ + "("
        if len(self.sequence_sharding) == 1:
            tmpstr += f"sequence_dim={self.sequence_sharding[0].dim}, "
        tmpstr += f"use_local_output={self.use_local_output}"
        tmpstr += ")"
        return tmpstr

````

- **L421** EN: Defines function `_prepare_output_fn`. | CN: 定义函数 `_prepare_output_fn`。
- **L422** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L423** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L424** EN: Defines function `_apply`. | CN: 定义函数 `_apply`。
- **L425** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L426** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L427** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L428** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L429** EN: Calls `partial` as part of the current workflow. | CN: 在当前流程中调用 `partial`。
- **L430** EN: Calls `partial` as part of the current workflow. | CN: 在当前流程中调用 `partial`。
- **L431** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L432** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L433** EN: Defines function `__repr__`. | CN: 定义函数 `__repr__`。
- **L434** EN: Assigns or updates `tmpstr`. | CN: 对 `tmpstr` 进行赋值或更新。
- **L435** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L436** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L437** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L438** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L439** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L440** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 441-460 / 第 441-460 行

````python

class PrepareModuleInput(ParallelStyle):
    """
    Configure the nn.Module's inputs to convert the input tensors of the nn.Module to DTensors at runtime according to
    ``input_layouts``, and perform layout redistribution according to the ``desired_input_layouts``.

    Keyword Args:
        input_layouts (Union[Placement, Tuple[Optional[Placement]]]):
            The DTensor layouts of input tensors for the nn.Module, this is used to convert the input tensors to
            DTensors. If some inputs are not torch.Tensor or no need to convert to DTensors, ``None`` need to be specified
            as a placeholder. default: None.
        desired_input_layouts (Union[Placement, Tuple[Optional[Placement]]]):
            The desired DTensor layout of input tensors for the nn.Module, this is used to ensure the inputs of the nn.Module
            have the desired DTensor layouts. This argument needs to have the same length with ``input_layouts``. default: None.
        input_kwarg_layouts (Dict[str, Placement]):
            The DTensor layouts of input kwargs for the nn.Module, this is used to convert the input kwarg tensors to DTensors.
            default: None
        desired_input_kwarg_layouts: (Dict[str, Placement]):
            The desired DTensor layout of input kwargs for the nn.Module, this is used to ensure the inputs of the nn.Module
            have the desired DTensor layouts. default: None.
````

- **L441** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L442** EN: Defines class `PrepareModuleInput`. | CN: 定义类 `PrepareModuleInput`。
- **L443** EN: Starts the docstring for the class PrepareModuleInput. | CN: 开始定义 class PrepareModuleInput 的文档字符串。
- **L444** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L445** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L446** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L447** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L448** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L449** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L450** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L451** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L452** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L453** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L454** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L455** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L456** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L457** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L458** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L459** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L460** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。

### Lines 461-480 / 第 461-480 行

````python
        use_local_output (bool, optional):
            Whether to use local :class:`torch.Tensor` instead of :class:`DTensor` for the module inputs, default: False.
    Returns:
        A :class:`ParallelStyle` object that prepares the sharding layouts of the nn.Module's inputs.

    Example::
        >>> # xdoctest: +SKIP(failing)
        >>> from torch.distributed.tensor.parallel import parallelize_module, PrepareModuleInput
        >>> from torch.distributed.device_mesh import init_device_mesh
        >>> ...
        >>> block = TransformerBlock(...)  # block is a nn.Module that contains an "attn" Attention submodule
        >>> tp_mesh = init_device_mesh("cuda", (8,))
        >>>
        >>> # According to the style specified below, the first input of attn will be annotated to Sharded DTensor
        >>> # and then redistributed to Replicated DTensor.
        >>> parallelize_module(
        >>>     block, # this can be a submodule or module
        >>>     tp_mesh,
        >>>     parallelize_plan={
        >>>         "attn": PrepareModuleInput(
````

- **L461** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L462** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L463** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L464** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L465** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L466** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L467** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L468** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L469** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L470** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L471** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L472** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L473** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L474** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L475** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L476** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L477** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L478** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L479** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L480** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。

### Lines 481-500 / 第 481-500 行

````python
        >>>             input_layouts=(Shard(0), None, None, ...),
        >>>             desired_input_layouts=(Replicate(), None, None, ...)
        >>>         ),
        >>>     }
        >>> )
    """

    def __init__(
        self,
        *,
        input_layouts: Placement | tuple[Placement | None, ...] | None = None,
        desired_input_layouts: Placement | tuple[Placement | None, ...] | None = None,
        input_kwarg_layouts: dict[str, Placement] | None = None,
        desired_input_kwarg_layouts: dict[str, Placement] | None = None,
        use_local_output: bool = False,
    ):
        self.input_layouts = (
            (input_layouts,) if isinstance(input_layouts, Placement) else input_layouts
        )
        self.desired_input_layouts = (
````

- **L481** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L482** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L483** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L484** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L485** EN: Continues the docstring text for the class PrepareModuleInput. | CN: 继续补充 class PrepareModuleInput 的文档字符串内容。
- **L486** EN: Closes the docstring for the class PrepareModuleInput. | CN: 结束 class PrepareModuleInput 的文档字符串。
- **L487** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L488** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L489** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L490** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L491** EN: Assigns or updates `input_layouts`. | CN: 对 `input_layouts` 进行赋值或更新。
- **L492** EN: Assigns or updates `desired_input_layouts`. | CN: 对 `desired_input_layouts` 进行赋值或更新。
- **L493** EN: Assigns or updates `input_kwarg_layouts`. | CN: 对 `input_kwarg_layouts` 进行赋值或更新。
- **L494** EN: Assigns or updates `desired_input_kwarg_layouts`. | CN: 对 `desired_input_kwarg_layouts` 进行赋值或更新。
- **L495** EN: Assigns or updates `use_local_output`. | CN: 对 `use_local_output` 进行赋值或更新。
- **L496** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L497** EN: Assigns or updates `self.input_layouts`. | CN: 对 `self.input_layouts` 进行赋值或更新。
- **L498** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L499** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L500** EN: Assigns or updates `self.desired_input_layouts`. | CN: 对 `self.desired_input_layouts` 进行赋值或更新。

### Lines 501-520 / 第 501-520 行

````python
            (desired_input_layouts,)
            if isinstance(desired_input_layouts, Placement)
            else desired_input_layouts
        )
        self.use_local_output = use_local_output
        if self.input_layouts is not None:
            if self.desired_input_layouts is None:
                raise AssertionError("desired module inputs should not be None!")
            if len(self.input_layouts) != len(self.desired_input_layouts):
                raise AssertionError(
                    "input_layouts and desired_input_layouts should have same length!"
                )
        self.with_kwargs = input_kwarg_layouts is not None
        self.input_kwarg_layouts = input_kwarg_layouts or {}
        self.desired_input_kwarg_layouts = desired_input_kwarg_layouts or {}
        if self.with_kwargs:
            if len(self.input_kwarg_layouts) != len(self.desired_input_kwarg_layouts):
                raise AssertionError(
                    "input_kwarg_layouts and desired_input_kwarg_layouts should have same length!"
                )
````

- **L501** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L502** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L503** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L504** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L505** EN: Assigns or updates `self.use_local_output`. | CN: 对 `self.use_local_output` 进行赋值或更新。
- **L506** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L507** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L508** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L509** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L510** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L511** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L512** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L513** EN: Assigns or updates `self.with_kwargs`. | CN: 对 `self.with_kwargs` 进行赋值或更新。
- **L514** EN: Assigns or updates `self.input_kwarg_layouts`. | CN: 对 `self.input_kwarg_layouts` 进行赋值或更新。
- **L515** EN: Assigns or updates `self.desired_input_kwarg_layouts`. | CN: 对 `self.desired_input_kwarg_layouts` 进行赋值或更新。
- **L516** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L517** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L518** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L519** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L520** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 521-540 / 第 521-540 行

````python

    def _prepare_input_arg(
        self,
        input: Any,
        mesh: DeviceMesh,
        input_layout: Placement | None,
        desired_layout: Placement | None,
    ):
        if input_layout is not None:
            if isinstance(input, DTensor):
                # TODO: re-enable the check once we fix the compile path
                # assert inp.placements[0] == input_layout
                dt_inp = input
            else:
                if not isinstance(input, torch.Tensor):
                    raise AssertionError("expecting input to be a torch.Tensor!")
                dt_inp = DTensor.from_local(
                    input, mesh, (input_layout,), run_check=False
                )

````

- **L521** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L522** EN: Defines function `_prepare_input_arg`. | CN: 定义函数 `_prepare_input_arg`。
- **L523** EN: Continues the implementation inside function `_prepare_input_arg`. | CN: 继续说明函数 `_prepare_input_arg` 内部的实现。
- **L524** EN: Continues the implementation inside function `_prepare_input_arg`. | CN: 继续说明函数 `_prepare_input_arg` 内部的实现。
- **L525** EN: Continues the implementation inside function `_prepare_input_arg`. | CN: 继续说明函数 `_prepare_input_arg` 内部的实现。
- **L526** EN: Continues the implementation inside function `_prepare_input_arg`. | CN: 继续说明函数 `_prepare_input_arg` 内部的实现。
- **L527** EN: Continues the implementation inside function `_prepare_input_arg`. | CN: 继续说明函数 `_prepare_input_arg` 内部的实现。
- **L528** EN: Continues the implementation inside function `_prepare_input_arg`. | CN: 继续说明函数 `_prepare_input_arg` 内部的实现。
- **L529** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L530** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L531** EN: Keeps the inline comment or directive: TODO: re-enable the check once we fix the compile path | CN: 保留这一行注释或指令：TODO: re-enable the check once we fix the compile path
- **L532** EN: Keeps the inline comment or directive: assert inp.placements[0] == input_layout | CN: 保留这一行注释或指令：assert inp.placements[0] == input_layout
- **L533** EN: Assigns or updates `dt_inp`. | CN: 对 `dt_inp` 进行赋值或更新。
- **L534** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L535** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L536** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L537** EN: Assigns or updates `dt_inp`. | CN: 对 `dt_inp` 进行赋值或更新。
- **L538** EN: Continues the implementation inside function `_prepare_input_arg`. | CN: 继续说明函数 `_prepare_input_arg` 内部的实现。
- **L539** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L540** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 541-560 / 第 541-560 行

````python
            if desired_layout is not None and input_layout != desired_layout:
                dt_inp = dt_inp.redistribute(placements=(desired_layout,))

            return dt_inp.to_local() if self.use_local_output else dt_inp
        else:
            return input

    def _prepare_input_fn(self, inputs, device_mesh):
        if self.input_layouts is None:
            return inputs
        prepared_inputs = []
        if not isinstance(inputs, tuple):
            inputs = (inputs,)
        if len(inputs) != len(self.input_layouts):
            raise ValueError("module inputs and input_layouts should have same length!")

        if self.desired_input_layouts is None:
            raise AssertionError("desired module inputs should not be None!")

        for inp, input_layout, desired_layout in zip(
````

- **L541** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L542** EN: Assigns or updates `dt_inp`. | CN: 对 `dt_inp` 进行赋值或更新。
- **L543** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L544** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L545** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L546** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L547** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L548** EN: Defines function `_prepare_input_fn`. | CN: 定义函数 `_prepare_input_fn`。
- **L549** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L550** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L551** EN: Assigns or updates `prepared_inputs`. | CN: 对 `prepared_inputs` 进行赋值或更新。
- **L552** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L553** EN: Assigns or updates `inputs`. | CN: 对 `inputs` 进行赋值或更新。
- **L554** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L555** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L556** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L557** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L558** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L559** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L560** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。

### Lines 561-580 / 第 561-580 行

````python
            inputs, self.input_layouts, self.desired_input_layouts
        ):
            prepared_inputs.append(
                self._prepare_input_arg(inp, device_mesh, input_layout, desired_layout)
            )
        return tuple(prepared_inputs)

    def _prepare_input_kwarg_fn(self, inputs, kwarg_inputs, device_mesh):
        prepared_arg_inputs = self._prepare_input_fn(inputs, device_mesh)
        prepared_kwarg_inputs = {}
        for kwarg_key in kwarg_inputs:
            kwarg_val = kwarg_inputs[kwarg_key]
            input_layout = self.input_kwarg_layouts.get(kwarg_key)
            desired_input_layout = self.desired_input_kwarg_layouts.get(kwarg_key)

            prepared_kwarg_inputs[kwarg_key] = self._prepare_input_arg(
                kwarg_val, device_mesh, input_layout, desired_input_layout
            )

        return (prepared_arg_inputs, prepared_kwarg_inputs)
````

- **L561** EN: Continues the implementation inside function `_prepare_input_fn`. | CN: 继续说明函数 `_prepare_input_fn` 内部的实现。
- **L562** EN: Continues the implementation inside function `_prepare_input_fn`. | CN: 继续说明函数 `_prepare_input_fn` 内部的实现。
- **L563** EN: Calls `prepared_inputs.append` as part of the current workflow. | CN: 在当前流程中调用 `prepared_inputs.append`。
- **L564** EN: Calls `self._prepare_input_arg` as part of the current workflow. | CN: 在当前流程中调用 `self._prepare_input_arg`。
- **L565** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L566** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L567** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L568** EN: Defines function `_prepare_input_kwarg_fn`. | CN: 定义函数 `_prepare_input_kwarg_fn`。
- **L569** EN: Assigns or updates `prepared_arg_inputs`. | CN: 对 `prepared_arg_inputs` 进行赋值或更新。
- **L570** EN: Assigns or updates `prepared_kwarg_inputs`. | CN: 对 `prepared_kwarg_inputs` 进行赋值或更新。
- **L571** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L572** EN: Assigns or updates `kwarg_val`. | CN: 对 `kwarg_val` 进行赋值或更新。
- **L573** EN: Assigns or updates `input_layout`. | CN: 对 `input_layout` 进行赋值或更新。
- **L574** EN: Assigns or updates `desired_input_layout`. | CN: 对 `desired_input_layout` 进行赋值或更新。
- **L575** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L576** EN: Assigns or updates `prepared_kwarg_inputs[kwarg_key]`. | CN: 对 `prepared_kwarg_inputs[kwarg_key]` 进行赋值或更新。
- **L577** EN: Continues the implementation inside function `_prepare_input_kwarg_fn`. | CN: 继续说明函数 `_prepare_input_kwarg_fn` 内部的实现。
- **L578** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L579** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L580** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

### Lines 581-600 / 第 581-600 行

````python

    def _apply(self, module: nn.Module, device_mesh: DeviceMesh) -> nn.Module:
        if self.with_kwargs:
            module.register_forward_pre_hook(
                lambda _, inputs, kwargs: self._prepare_input_kwarg_fn(
                    inputs, kwargs, device_mesh
                ),
                with_kwargs=True,
            )  # type: ignore[misc]
        else:
            module.register_forward_pre_hook(
                lambda _, inputs: self._prepare_input_fn(inputs, device_mesh)
            )  # type: ignore[misc, call-arg]
        return module

    def __repr__(self) -> str:
        tmpstr = self.__class__.__name__ + "("
        tmpstr += f"input_layouts={self.input_layouts}, "
        tmpstr += f"desired_input_layouts={self.desired_input_layouts}, "
        tmpstr += f"input_kwarg_layouts={self.input_kwarg_layouts}, "
````

- **L581** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L582** EN: Defines function `_apply`. | CN: 定义函数 `_apply`。
- **L583** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L584** EN: Calls `module.register_forward_pre_hook` as part of the current workflow. | CN: 在当前流程中调用 `module.register_forward_pre_hook`。
- **L585** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L586** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L587** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L588** EN: Assigns or updates `with_kwargs`. | CN: 对 `with_kwargs` 进行赋值或更新。
- **L589** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L590** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L591** EN: Calls `module.register_forward_pre_hook` as part of the current workflow. | CN: 在当前流程中调用 `module.register_forward_pre_hook`。
- **L592** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L593** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L594** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L595** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L596** EN: Defines function `__repr__`. | CN: 定义函数 `__repr__`。
- **L597** EN: Assigns or updates `tmpstr`. | CN: 对 `tmpstr` 进行赋值或更新。
- **L598** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L599** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L600** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。

### Lines 601-620 / 第 601-620 行

````python
        tmpstr += f"desired_input_kwarg_layouts={self.desired_input_kwarg_layouts}, "
        tmpstr += f"use_local_output={self.use_local_output}"
        tmpstr += ")"
        return tmpstr


class PrepareModuleOutput(ParallelStyle):
    """
    Configure the nn.Module's outputs to convert the output tensors of the nn.Module to DTensors at runtime according to
    ``output_layouts``, and perform layout redistribution according to the ``desired_output_layouts``.

    Keyword Args:
        output_layouts (Union[Placement, Tuple[Placement]]):
            The DTensor layouts of output tensors for the nn.Module, this is used to convert the output tensors to
            DTensors if they are :class:`torch.Tensor`. If some outputs are not torch.Tensor or no need to convert to DTensors,
            ``None`` need to be specified as a placeholder.
        desired_output_layouts (Union[Placement, Tuple[Placement]]):
            The desired DTensor layouts of output tensors for the nn.Module, this is used to ensure the outputs of the nn.Module
            have the desired DTensor layouts.
        use_local_output (bool, optional):
````

- **L601** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L602** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L603** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L604** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L605** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L606** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L607** EN: Defines class `PrepareModuleOutput`. | CN: 定义类 `PrepareModuleOutput`。
- **L608** EN: Starts the docstring for the class PrepareModuleOutput. | CN: 开始定义 class PrepareModuleOutput 的文档字符串。
- **L609** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L610** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L611** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L612** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L613** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L614** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L615** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L616** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L617** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L618** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L619** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L620** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。

### Lines 621-640 / 第 621-640 行

````python
            Whether to use local :class:`torch.Tensor` instead of :class:`DTensor` for the module outputs, default: True.
    Returns:
        A ParallelStyle object that prepares the sharding layouts of the nn.Module's outputs.

    Example::
        >>> # xdoctest: +SKIP(failing)
        >>> from torch.distributed.tensor.parallel import parallelize_module, PrepareModuleOutput
        >>> from torch.distributed.device_mesh import init_device_mesh
        >>> ...
        >>> block = TransformerBlock(...)  # block is a nn.Module that contains an "attn" Attention submodule
        >>> tp_mesh = init_device_mesh("cuda", (8,))
        >>>
        >>> # According to the style specified below, the output of the TransformerBlock will be converted to Replicated DTensor
        >>> # and then redistributed to Sharded DTensor.
        >>> parallelize_module(
        >>>     block, # this can be a submodule or module
        >>>     tp_mesh,
        >>>     parallelize_plan = PrepareModuleOutput(
        >>>         output_layouts=Replicate(),
        >>>         desired_output_layouts=Shard(0)
````

- **L621** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L622** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L623** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L624** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L625** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L626** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L627** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L628** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L629** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L630** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L631** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L632** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L633** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L634** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L635** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L636** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L637** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L638** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L639** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L640** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。

### Lines 641-660 / 第 641-660 行

````python
        >>>     )
        >>> )
    """

    def __init__(
        self,
        *,
        output_layouts: Placement | tuple[Placement | None, ...],
        desired_output_layouts: Placement | tuple[Placement, ...],
        use_local_output: bool = True,
    ):
        self.output_layouts = (
            (output_layouts,)
            if isinstance(output_layouts, Placement)
            else output_layouts
        )
        self.desired_output_layouts = (
            (desired_output_layouts,)
            if isinstance(desired_output_layouts, Placement)
            else desired_output_layouts
````

- **L641** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L642** EN: Continues the docstring text for the class PrepareModuleOutput. | CN: 继续补充 class PrepareModuleOutput 的文档字符串内容。
- **L643** EN: Closes the docstring for the class PrepareModuleOutput. | CN: 结束 class PrepareModuleOutput 的文档字符串。
- **L644** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L645** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L646** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L647** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L648** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L649** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L650** EN: Assigns or updates `use_local_output`. | CN: 对 `use_local_output` 进行赋值或更新。
- **L651** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L652** EN: Assigns or updates `self.output_layouts`. | CN: 对 `self.output_layouts` 进行赋值或更新。
- **L653** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L654** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L655** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L656** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L657** EN: Assigns or updates `self.desired_output_layouts`. | CN: 对 `self.desired_output_layouts` 进行赋值或更新。
- **L658** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L659** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L660** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。

### Lines 661-680 / 第 661-680 行

````python
        )
        self.use_local_output = use_local_output
        if len(self.output_layouts) != len(self.desired_output_layouts):
            raise AssertionError(
                "output_layouts and desired_output_layouts should have same length!"
            )

    def _prepare_out_fn(self, outputs, device_mesh):
        prepared_outputs = []
        if not isinstance(outputs, tuple):
            outputs = (outputs,)
        if len(outputs) != len(self.output_layouts):
            raise ValueError(
                "module outputs and output_layouts should have same length!"
            )

        for out, out_layout, desired_out_layout in zip(
            outputs, self.output_layouts, self.desired_output_layouts
        ):
            if out_layout is not None:
````

- **L661** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L662** EN: Assigns or updates `self.use_local_output`. | CN: 对 `self.use_local_output` 进行赋值或更新。
- **L663** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L664** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L665** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L666** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L667** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L668** EN: Defines function `_prepare_out_fn`. | CN: 定义函数 `_prepare_out_fn`。
- **L669** EN: Assigns or updates `prepared_outputs`. | CN: 对 `prepared_outputs` 进行赋值或更新。
- **L670** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L671** EN: Assigns or updates `outputs`. | CN: 对 `outputs` 进行赋值或更新。
- **L672** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L673** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L674** EN: Continues the implementation inside function `_prepare_out_fn`. | CN: 继续说明函数 `_prepare_out_fn` 内部的实现。
- **L675** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L676** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L677** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L678** EN: Continues the implementation inside function `_prepare_out_fn`. | CN: 继续说明函数 `_prepare_out_fn` 内部的实现。
- **L679** EN: Continues the implementation inside function `_prepare_out_fn`. | CN: 继续说明函数 `_prepare_out_fn` 内部的实现。
- **L680** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 681-700 / 第 681-700 行

````python
                if isinstance(out, DTensor):
                    # TODO: re-enable the check once we fix the compile path
                    # assert out.placements[0] == out_layout
                    dt_out = out
                else:
                    dt_out = DTensor.from_local(
                        out, device_mesh, (out_layout,), run_check=False
                    )

                if out_layout != desired_out_layout:
                    dt_out = dt_out.redistribute(placements=(desired_out_layout,))
                prepared_outputs.append(
                    dt_out.to_local() if self.use_local_output else dt_out
                )
            else:
                prepared_outputs.append(out)
        if len(prepared_outputs) == 1:
            return prepared_outputs[0]
        else:
            return tuple(prepared_outputs)
````

- **L681** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L682** EN: Keeps the inline comment or directive: TODO: re-enable the check once we fix the compile path | CN: 保留这一行注释或指令：TODO: re-enable the check once we fix the compile path
- **L683** EN: Keeps the inline comment or directive: assert out.placements[0] == out_layout | CN: 保留这一行注释或指令：assert out.placements[0] == out_layout
- **L684** EN: Assigns or updates `dt_out`. | CN: 对 `dt_out` 进行赋值或更新。
- **L685** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L686** EN: Assigns or updates `dt_out`. | CN: 对 `dt_out` 进行赋值或更新。
- **L687** EN: Continues the implementation inside function `_prepare_out_fn`. | CN: 继续说明函数 `_prepare_out_fn` 内部的实现。
- **L688** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L689** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L690** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L691** EN: Assigns or updates `dt_out`. | CN: 对 `dt_out` 进行赋值或更新。
- **L692** EN: Calls `prepared_outputs.append` as part of the current workflow. | CN: 在当前流程中调用 `prepared_outputs.append`。
- **L693** EN: Calls `dt_out.to_local` as part of the current workflow. | CN: 在当前流程中调用 `dt_out.to_local`。
- **L694** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L695** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L696** EN: Calls `prepared_outputs.append` as part of the current workflow. | CN: 在当前流程中调用 `prepared_outputs.append`。
- **L697** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L698** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L699** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L700** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

### Lines 701-720 / 第 701-720 行

````python

    def _apply(self, module: nn.Module, device_mesh: DeviceMesh) -> nn.Module:
        module.register_forward_hook(
            lambda _, inputs, outputs: self._prepare_out_fn(outputs, device_mesh)
        )  # type: ignore[misc, call-arg]
        return module

    def __repr__(self) -> str:
        tmpstr = self.__class__.__name__ + "("
        tmpstr += f"output_layouts={self.output_layouts}, "
        tmpstr += f"desired_output_layouts={self.desired_output_layouts}, "
        tmpstr += f"use_local_output={self.use_local_output}"
        tmpstr += ")"
        return tmpstr


class PrepareModuleInputOutput(ParallelStyle):
    """
    Configure the nn.Module's inputs (and outputs) to convert the input tensors (and output tensors, respectively) of the nn.Module
    to DTensors at runtime according to ``input_layouts`` (and output_layouts, respectively), and perform layout redistribution
````

- **L701** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L702** EN: Defines function `_apply`. | CN: 定义函数 `_apply`。
- **L703** EN: Calls `module.register_forward_hook` as part of the current workflow. | CN: 在当前流程中调用 `module.register_forward_hook`。
- **L704** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L705** EN: Continues the implementation inside function `_apply`. | CN: 继续说明函数 `_apply` 内部的实现。
- **L706** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L707** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L708** EN: Defines function `__repr__`. | CN: 定义函数 `__repr__`。
- **L709** EN: Assigns or updates `tmpstr`. | CN: 对 `tmpstr` 进行赋值或更新。
- **L710** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L711** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L712** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L713** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L714** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L715** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L716** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L717** EN: Defines class `PrepareModuleInputOutput`. | CN: 定义类 `PrepareModuleInputOutput`。
- **L718** EN: Starts the docstring for the class PrepareModuleInputOutput. | CN: 开始定义 class PrepareModuleInputOutput 的文档字符串。
- **L719** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L720** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。

### Lines 721-740 / 第 721-740 行

````python
    according to the ``desired_input_layouts`` (and ``desired_output_layouts``, respectively). This is a combination of
    :class:`PrepareModuleInput` and :class:`PrepareModuleOutput`.

    Keyword Args:
        input_layouts (Union[Placement, Tuple[Optional[Placement]]]):
            The DTensor layouts of input tensors for the nn.Module, this is used to convert the input tensors to
            DTensors. If some inputs are not torch.Tensor or no need to convert to DTensors, ``None`` need to be specified
            as a placeholder. default: None.
        desired_input_layouts (Union[Placement, Tuple[Optional[Placement]]]):
            The desired DTensor layout of input tensors for the nn.Module, this is used to ensure the inputs of the nn.Module
            have the desired DTensor layouts. This argument needs to have the same length with ``input_layouts``. default: None.
        input_kwarg_layouts (Dict[str, Placement]):
            The DTensor layouts of input kwargs for the nn.Module, this is used to convert the input kwarg tensors to DTensors.
            default: None
        desired_input_kwarg_layouts: (Dict[str, Placement]):
            The desired DTensor layout of input kwargs for the nn.Module, this is used to ensure the inputs of the nn.Module
            have the desired DTensor layouts. default: None.
        use_local_input (bool, optional):
            Whether to use local :class:`torch.Tensor` instead of :class:`DTensor` for the module inputs, default: False.
        output_layouts (Union[Placement, Tuple[Placement]]):
````

- **L721** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L722** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L723** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L724** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L725** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L726** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L727** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L728** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L729** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L730** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L731** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L732** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L733** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L734** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L735** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L736** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L737** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L738** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L739** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L740** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。

### Lines 741-760 / 第 741-760 行

````python
            The DTensor layouts of output tensors for the nn.Module, this is used to convert the output tensors to
            DTensors if they are :class:`torch.Tensor`. If some outputs are not torch.Tensor or no need to convert to DTensors,
            ``None`` need to be specified as a placeholder.
        desired_output_layouts (Union[Placement, Tuple[Placement]]):
            The desired DTensor layouts of output tensors for the nn.Module, this is used to ensure the outputs of the nn.Module
            have the desired DTensor layouts.
        use_local_output (bool, optional):
            Whether to use local :class:`torch.Tensor` instead of :class:`DTensor` for the module outputs, default: True.
    Returns:
        A :class:`ParallelStyle` object that prepares the sharding layouts of the nn.Module's inputs and outputs.

    Example::
        >>> # xdoctest: +SKIP(failing)
        >>> from torch.distributed.tensor.parallel import parallelize_module, PrepareModuleInputOutput
        >>> from torch.distributed.device_mesh import init_device_mesh
        >>> ...
        >>> block = TransformerBlock(...)  # block is a nn.Module that contains an "attn" Attention submodule
        >>> tp_mesh = init_device_mesh("cuda", (8,))
        >>>
        >>> # According to the style specified below, the first input of attn will be annotated as Sharded DTensor
````

- **L741** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L742** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L743** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L744** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L745** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L746** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L747** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L748** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L749** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L750** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L751** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L752** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L753** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L754** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L755** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L756** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L757** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L758** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L759** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L760** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。

### Lines 761-780 / 第 761-780 行

````python
        >>> # and then redistributed to Replicated DTensor, and the output of the TransformerBlock will be annotated
        >>> # as Replicated DTensor and then redistributed to Sharded DTensor.
        >>> parallelize_module(
        >>>     block, # this can be a submodule or module
        >>>     tp_mesh,
        >>>     parallelize_plan={
        >>>         "attn": PrepareModuleInputOutput(
        >>>             input_layouts=(Shard(0), None, None, ...),
        >>>             desired_input_layouts=(Replicate(), None, None, ...),
        >>>             output_layouts=Replicate(),
        >>>             desired_output_layouts=Shard(0),
        >>>         ),
        >>>     }
        >>> )
    """

    def __init__(
        self,
        *,
        input_layouts: Placement | tuple[Placement | None, ...] | None = None,
````

- **L761** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L762** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L763** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L764** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L765** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L766** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L767** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L768** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L769** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L770** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L771** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L772** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L773** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L774** EN: Continues the docstring text for the class PrepareModuleInputOutput. | CN: 继续补充 class PrepareModuleInputOutput 的文档字符串内容。
- **L775** EN: Closes the docstring for the class PrepareModuleInputOutput. | CN: 结束 class PrepareModuleInputOutput 的文档字符串。
- **L776** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L777** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L778** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L779** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L780** EN: Assigns or updates `input_layouts`. | CN: 对 `input_layouts` 进行赋值或更新。

### Lines 781-800 / 第 781-800 行

````python
        desired_input_layouts: Placement | tuple[Placement | None, ...] | None = None,
        input_kwarg_layouts: dict[str, Placement] | None = None,
        desired_input_kwarg_layouts: dict[str, Placement] | None = None,
        use_local_input: bool = False,
        output_layouts: Placement | tuple[Placement | None, ...],
        desired_output_layouts: Placement | tuple[Placement, ...],
        use_local_output: bool = True,
    ):
        self.prepare_module_input = PrepareModuleInput(
            input_layouts=input_layouts,
            desired_input_layouts=desired_input_layouts,
            input_kwarg_layouts=input_kwarg_layouts,
            desired_input_kwarg_layouts=desired_input_kwarg_layouts,
            use_local_output=use_local_input,
        )
        self.prepare_module_output = PrepareModuleOutput(
            output_layouts=output_layouts,
            desired_output_layouts=desired_output_layouts,
            use_local_output=use_local_output,
        )
````

- **L781** EN: Assigns or updates `desired_input_layouts`. | CN: 对 `desired_input_layouts` 进行赋值或更新。
- **L782** EN: Assigns or updates `input_kwarg_layouts`. | CN: 对 `input_kwarg_layouts` 进行赋值或更新。
- **L783** EN: Assigns or updates `desired_input_kwarg_layouts`. | CN: 对 `desired_input_kwarg_layouts` 进行赋值或更新。
- **L784** EN: Assigns or updates `use_local_input`. | CN: 对 `use_local_input` 进行赋值或更新。
- **L785** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L786** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L787** EN: Assigns or updates `use_local_output`. | CN: 对 `use_local_output` 进行赋值或更新。
- **L788** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L789** EN: Assigns or updates `self.prepare_module_input`. | CN: 对 `self.prepare_module_input` 进行赋值或更新。
- **L790** EN: Assigns or updates `input_layouts`. | CN: 对 `input_layouts` 进行赋值或更新。
- **L791** EN: Assigns or updates `desired_input_layouts`. | CN: 对 `desired_input_layouts` 进行赋值或更新。
- **L792** EN: Assigns or updates `input_kwarg_layouts`. | CN: 对 `input_kwarg_layouts` 进行赋值或更新。
- **L793** EN: Assigns or updates `desired_input_kwarg_layouts`. | CN: 对 `desired_input_kwarg_layouts` 进行赋值或更新。
- **L794** EN: Assigns or updates `use_local_output`. | CN: 对 `use_local_output` 进行赋值或更新。
- **L795** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L796** EN: Assigns or updates `self.prepare_module_output`. | CN: 对 `self.prepare_module_output` 进行赋值或更新。
- **L797** EN: Assigns or updates `output_layouts`. | CN: 对 `output_layouts` 进行赋值或更新。
- **L798** EN: Assigns or updates `desired_output_layouts`. | CN: 对 `desired_output_layouts` 进行赋值或更新。
- **L799** EN: Assigns or updates `use_local_output`. | CN: 对 `use_local_output` 进行赋值或更新。
- **L800** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 801-820 / 第 801-820 行

````python

    def _apply(self, module: nn.Module, device_mesh: DeviceMesh) -> nn.Module:
        self.prepare_module_input._apply(module, device_mesh)
        self.prepare_module_output._apply(module, device_mesh)

        return module

    def __repr__(self) -> str:
        tmpstr = self.__class__.__name__ + "("
        tmpstr += f"input_layouts={self.prepare_module_input.input_layouts}, "
        tmpstr += (
            f"desired_input_layouts={self.prepare_module_input.desired_input_layouts}, "
        )
        tmpstr += (
            f"input_kwarg_layouts={self.prepare_module_input.input_kwarg_layouts}, "
        )
        tmpstr += f"desired_input_kwarg_layouts={self.prepare_module_input.desired_input_kwarg_layouts}, "
        tmpstr += f"use_local_input={self.prepare_module_input.use_local_output}, "
        tmpstr += f"output_layouts={self.prepare_module_output.output_layouts}, "
        tmpstr += f"desired_output_layouts={self.prepare_module_output.desired_output_layouts}, "
````

- **L801** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L802** EN: Defines function `_apply`. | CN: 定义函数 `_apply`。
- **L803** EN: Calls `self.prepare_module_input._apply` as part of the current workflow. | CN: 在当前流程中调用 `self.prepare_module_input._apply`。
- **L804** EN: Calls `self.prepare_module_output._apply` as part of the current workflow. | CN: 在当前流程中调用 `self.prepare_module_output._apply`。
- **L805** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L806** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L807** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L808** EN: Defines function `__repr__`. | CN: 定义函数 `__repr__`。
- **L809** EN: Assigns or updates `tmpstr`. | CN: 对 `tmpstr` 进行赋值或更新。
- **L810** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L811** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L812** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L813** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L814** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L815** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L816** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L817** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L818** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L819** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L820** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。

### Lines 821-823 / 第 821-823 行

````python
        tmpstr += f"use_local_output={self.prepare_module_output.use_local_output}"
        tmpstr += ")"
        return tmpstr
````

- **L821** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L822** EN: Continues the implementation inside function `__repr__`. | CN: 继续说明函数 `__repr__` 内部的实现。
- **L823** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

## Key Concepts / 关键概念

- **EN**: distributed tensor layouts, placements, and operators  
  **CN**: 分布式张量布局、放置与算子逻辑
- **EN**: device mesh  
  **CN**: 设备网格
- **EN**: sharding  
  **CN**: 分片
- **EN**: reduce-scatter  
  **CN**: reduce-scatter 操作
- **EN**: broadcast  
  **CN**: 广播
- **EN**: distributed tensors  
  **CN**: 分布式张量
- **EN**: placements  
  **CN**: 放置规则
- **EN**: communication hooks  
  **CN**: 通信钩子

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed.tensor`, `torch.distributed.tensor.placement_types`
- **PyTorch / PyTorch**: `torch`, `torch.nn`
- **Python Stdlib / Python 标准库**: `abc`, `functools`, `typing`
- **Third-party / 第三方**: None detected / 未检测到

