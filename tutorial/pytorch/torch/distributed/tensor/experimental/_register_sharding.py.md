# _register_sharding.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/tensor/experimental/_register_sharding.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed tensor layouts, placements, and operators. Its main entry points include register_sharding.
- **用途 (CN)**: 该模块聚焦于分布式张量布局、放置与算子逻辑，其主要入口包括 register_sharding。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs
# Copyright (c) Meta Platforms, Inc. and affiliates
from collections.abc import Callable, Sequence
from functools import partial

import torch
from torch._ops import OpOverload
from torch.distributed.tensor import DTensor
from torch.distributed.tensor._op_schema import (
    OpSchema,
    OpStrategy,
    PlacementList,
    RuntimeSchemaInfo,
    StrategyType,
    TupleStrategy,
)
from torch.distributed.tensor._ops.utils import expand_to_full_mesh_op_strategy


__all__ = ["register_sharding"]
````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Keeps the inline comment or directive: Copyright (c) Meta Platforms, Inc. and affiliates | CN: 保留这一行注释或指令：Copyright (c) Meta Platforms, Inc. and affiliates
- **L3** EN: Imports selected names from `collections.abc`. | CN: 从 `collections.abc` 导入指定名称。
- **L4** EN: Imports selected names from `functools`. | CN: 从 `functools` 导入指定名称。
- **L5** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L6** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L7** EN: Imports selected names from `torch._ops`. | CN: 从 `torch._ops` 导入指定名称。
- **L8** EN: Imports selected names from `torch.distributed.tensor`. | CN: 从 `torch.distributed.tensor` 导入指定名称。
- **L9** EN: Imports selected names from `torch.distributed.tensor._op_schema`. | CN: 从 `torch.distributed.tensor._op_schema` 导入指定名称。
- **L10** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L11** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L12** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L13** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L14** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L15** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L16** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L17** EN: Imports selected names from `torch.distributed.tensor._ops.utils`. | CN: 从 `torch.distributed.tensor._ops.utils` 导入指定名称。
- **L18** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L19** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L20** EN: Assigns or updates `__all__`. | CN: 对 `__all__` 进行赋值或更新。

### Lines 21-40 / 第 21-40 行

````python


def register_sharding(op: OpOverload | list[OpOverload]):
    """
    :meth:`register_sharding` is an experimental API that allows users to register sharding
    strategies for an operator when the tensor inputs and outputs are DTensor.
    It can be useful when: (1) there doesn't exist a default sharding strategy for ``op``,
    e.g. when ``op`` is a custom operator that is not supported by :class:`DTensor`; (2)
    when users would like to overwrite default sharding strategies of existing operators.

    Args:
        op (Union[OpOverload, List[OpOverload]]):
            An op or a list of ops to register the customized sharding function.

    Returns:
        A function decorator which can be used to wrap a function that defines the sharding
        strategy for the operator specified in ``op``. The defined sharding strategy will be
        registered to DTensor and will override the default sharding strategy if DTensor has
        already implemented the operator. The customized sharding function takes the same inputs
        as the original op (except that if an arg is a :class:`torch.Tensor`, it will be
````

- **L21** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L22** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L23** EN: Defines function `register_sharding`. | CN: 定义函数 `register_sharding`。
- **L24** EN: Starts the docstring for the function register_sharding. | CN: 开始定义 function register_sharding 的文档字符串。
- **L25** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L26** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L27** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L28** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L29** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L30** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L31** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L32** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L33** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L34** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L35** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L36** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L37** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L38** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L39** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L40** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。

### Lines 41-60 / 第 41-60 行

````python
        replaced by a tensor-like object that DTensor uses internally). The function should
        return a sequence of 2-tuples, each specifying acceptable output placements and its
        corresponding input placements.

    Example:
        >>> # xdoctest: +SKIP("distributed")
        >>> @register_sharding(aten._softmax.default)
        >>> def custom_softmax_sharding(x, dim, half_to_float):
        >>>     softmax_dim = dim if dim >= 0 else dim + x.ndim
        >>>     acceptable_shardings = []
        >>>
        >>>     all_replicate = ([Replicate()], [Replicate(), None, None])
        >>>     acceptable_shardings.append(all_replicate)
        >>>
        >>>     for sharding_dim in range(x.ndim):
        >>>         if sharding_dim != softmax_dim:
        >>>             all_sharded = (
        >>>                 [Shard(sharding_dim)],
        >>>                 [Shard(sharding_dim), None, None],
        >>>             )
````

- **L41** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L42** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L43** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L44** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L45** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L46** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L47** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L48** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L49** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L50** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L51** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L52** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L53** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L54** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L55** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L56** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L57** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L58** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L59** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L60** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。

### Lines 61-80 / 第 61-80 行

````python
        >>>             acceptable_shardings.append(all_sharded)
        >>>
        >>>     return acceptable_shardings

    .. note:: This API is currently experimental and subject to change
    """

    def custom_strategy(
        custom_sharding_fn: Callable[
            ..., Sequence[tuple[PlacementList, PlacementList]]
        ],
        op_schema: OpSchema,
    ) -> StrategyType:
        def strategy_to_spec(strategy: object) -> object:
            if isinstance(strategy, OpStrategy):
                # take the output spec from the first strategy
                return strategy.strategies[0].output_spec
            elif isinstance(strategy, TupleStrategy):
                return tuple(strategy_to_spec(s) for s in strategy.children)
            else:
````

- **L61** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L62** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L63** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L64** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L65** EN: Continues the docstring text for the function register_sharding. | CN: 继续补充 function register_sharding 的文档字符串内容。
- **L66** EN: Closes the docstring for the function register_sharding. | CN: 结束 function register_sharding 的文档字符串。
- **L67** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L68** EN: Defines function `custom_strategy`. | CN: 定义函数 `custom_strategy`。
- **L69** EN: Continues the implementation inside function `custom_strategy`. | CN: 继续说明函数 `custom_strategy` 内部的实现。
- **L70** EN: Continues the implementation inside function `custom_strategy`. | CN: 继续说明函数 `custom_strategy` 内部的实现。
- **L71** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L72** EN: Continues the implementation inside function `custom_strategy`. | CN: 继续说明函数 `custom_strategy` 内部的实现。
- **L73** EN: Continues the implementation inside function `custom_strategy`. | CN: 继续说明函数 `custom_strategy` 内部的实现。
- **L74** EN: Defines function `strategy_to_spec`. | CN: 定义函数 `strategy_to_spec`。
- **L75** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L76** EN: Keeps the inline comment or directive: take the output spec from the first strategy | CN: 保留这一行注释或指令：take the output spec from the first strategy
- **L77** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L78** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L79** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L80** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。

### Lines 81-100 / 第 81-100 行

````python
                return strategy

        mesh = op_schema.get_mesh_from_args()

        args_schema = tuple(strategy_to_spec(i) for i in op_schema.args_schema)
        kwargs_schema = {
            k: strategy_to_spec(v) for k, v in op_schema.kwargs_schema.items()
        }

        acceptable_shardings = custom_sharding_fn(*args_schema, **kwargs_schema)

        single_mesh_dim_strategies: list[PlacementList] = []
        for output_specs, input_specs in acceptable_shardings:
            single_mesh_dim_strategies.append(output_specs + input_specs)

        # TODO: handle out variant ops
        return expand_to_full_mesh_op_strategy(
            mesh,
            op_schema,
            single_mesh_dim_strategies,
````

- **L81** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L82** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L83** EN: Assigns or updates `mesh`. | CN: 对 `mesh` 进行赋值或更新。
- **L84** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L85** EN: Assigns or updates `args_schema`. | CN: 对 `args_schema` 进行赋值或更新。
- **L86** EN: Assigns or updates `kwargs_schema`. | CN: 对 `kwargs_schema` 进行赋值或更新。
- **L87** EN: Continues the implementation inside function `custom_strategy`. | CN: 继续说明函数 `custom_strategy` 内部的实现。
- **L88** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L89** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L90** EN: Assigns or updates `acceptable_shardings`. | CN: 对 `acceptable_shardings` 进行赋值或更新。
- **L91** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L92** EN: Assigns or updates `single_mesh_dim_strategies`. | CN: 对 `single_mesh_dim_strategies` 进行赋值或更新。
- **L93** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L94** EN: Calls `single_mesh_dim_strategies.append` as part of the current workflow. | CN: 在当前流程中调用 `single_mesh_dim_strategies.append`。
- **L95** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L96** EN: Keeps the inline comment or directive: TODO: handle out variant ops | CN: 保留这一行注释或指令：TODO: handle out variant ops
- **L97** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L98** EN: Continues the implementation inside function `custom_strategy`. | CN: 继续说明函数 `custom_strategy` 内部的实现。
- **L99** EN: Continues the implementation inside function `custom_strategy`. | CN: 继续说明函数 `custom_strategy` 内部的实现。
- **L100** EN: Continues the implementation inside function `custom_strategy`. | CN: 继续说明函数 `custom_strategy` 内部的实现。

### Lines 101-120 / 第 101-120 行

````python
            input_index=len(op_schema.op._schema.returns),
            inplace_op=op_schema.is_inplace_op(),
        )

    def wrapper(custom_sharding_fn):
        def derive_schema_info(op):
            # NOTE: without user directly providing RuntimeSchemaInfo, for now
            #       we create it in a conservative fashion as follows:
            #       1. let static_argnum be the first int argument
            #       2. let static_kwargkey include all the int type kwargs
            #       3. always set needs_pytree=True
            static_argnum = 100
            static_kwargkey: list[str] = []
            for i, arg in enumerate(op._schema.arguments):
                if isinstance(arg.type, torch.IntType) or (
                    isinstance(arg.type, torch.OptionalType)
                    and isinstance(arg.type.getElementType(), torch.IntType)
                ):
                    static_argnum = min(i, static_argnum)
                    if arg.kwarg_only:
````

- **L101** EN: Assigns or updates `input_index`. | CN: 对 `input_index` 进行赋值或更新。
- **L102** EN: Assigns or updates `inplace_op`. | CN: 对 `inplace_op` 进行赋值或更新。
- **L103** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L104** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L105** EN: Defines function `wrapper`. | CN: 定义函数 `wrapper`。
- **L106** EN: Defines function `derive_schema_info`. | CN: 定义函数 `derive_schema_info`。
- **L107** EN: Keeps the inline comment or directive: NOTE: without user directly providing RuntimeSchemaInfo, for now | CN: 保留这一行注释或指令：NOTE: without user directly providing RuntimeSchemaInfo, for now
- **L108** EN: Keeps the inline comment or directive: we create it in a conservative fashion as follows: | CN: 保留这一行注释或指令：we create it in a conservative fashion as follows:
- **L109** EN: Keeps the inline comment or directive: 1. let static_argnum be the first int argument | CN: 保留这一行注释或指令：1. let static_argnum be the first int argument
- **L110** EN: Keeps the inline comment or directive: 2. let static_kwargkey include all the int type kwargs | CN: 保留这一行注释或指令：2. let static_kwargkey include all the int type kwargs
- **L111** EN: Keeps the inline comment or directive: 3. always set needs_pytree=True | CN: 保留这一行注释或指令：3. always set needs_pytree=True
- **L112** EN: Assigns or updates `static_argnum`. | CN: 对 `static_argnum` 进行赋值或更新。
- **L113** EN: Assigns or updates `static_kwargkey`. | CN: 对 `static_kwargkey` 进行赋值或更新。
- **L114** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L115** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L116** EN: Calls `isinstance` as part of the current workflow. | CN: 在当前流程中调用 `isinstance`。
- **L117** EN: Continues the implementation inside function `derive_schema_info`. | CN: 继续说明函数 `derive_schema_info` 内部的实现。
- **L118** EN: Continues the implementation inside function `derive_schema_info`. | CN: 继续说明函数 `derive_schema_info` 内部的实现。
- **L119** EN: Assigns or updates `static_argnum`. | CN: 对 `static_argnum` 进行赋值或更新。
- **L120** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 121-136 / 第 121-136 行

````python
                        static_kwargkey.append(arg.name)
            return RuntimeSchemaInfo(
                static_argnum, static_kwargkey or None, needs_pytree=True
            )

        overloads = op if isinstance(op, list) else [op]
        for overload in overloads:
            DTensor._op_dispatcher.sharding_propagator.register_op_strategy(
                overload,
                partial(custom_strategy, custom_sharding_fn),
                derive_schema_info(overload),
            )

        return custom_sharding_fn

    return wrapper
````

- **L121** EN: Calls `static_kwargkey.append` as part of the current workflow. | CN: 在当前流程中调用 `static_kwargkey.append`。
- **L122** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L123** EN: Assigns or updates `static_argnum, static_kwargkey or None, needs_pytree`. | CN: 对 `static_argnum, static_kwargkey or None, needs_pytree` 进行赋值或更新。
- **L124** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L125** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L126** EN: Assigns or updates `overloads`. | CN: 对 `overloads` 进行赋值或更新。
- **L127** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L128** EN: Calls `DTensor._op_dispatcher.sharding_propagator.register_op_strategy` as part of the current workflow. | CN: 在当前流程中调用 `DTensor._op_dispatcher.sharding_propagator.register_op_strategy`。
- **L129** EN: Continues the implementation inside function `wrapper`. | CN: 继续说明函数 `wrapper` 内部的实现。
- **L130** EN: Calls `partial` as part of the current workflow. | CN: 在当前流程中调用 `partial`。
- **L131** EN: Calls `derive_schema_info` as part of the current workflow. | CN: 在当前流程中调用 `derive_schema_info`。
- **L132** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L133** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L134** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L135** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L136** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

## Key Concepts / 关键概念

- **EN**: distributed tensor layouts, placements, and operators  
  **CN**: 分布式张量布局、放置与算子逻辑
- **EN**: sharding  
  **CN**: 分片
- **EN**: distributed tensors  
  **CN**: 分布式张量
- **EN**: placements  
  **CN**: 放置规则
- **EN**: Core callables: register_sharding  
  **CN**: 核心可调用对象：register_sharding

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed.tensor`, `torch.distributed.tensor._op_schema`, `torch.distributed.tensor._ops.utils`
- **PyTorch / PyTorch**: `torch`, `torch._ops`
- **Python Stdlib / Python 标准库**: `collections.abc`, `functools`
- **Third-party / 第三方**: None detected / 未检测到

