# _decompositions.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/tensor/_decompositions.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed tensor layouts, placements, and operators. Its main entry points include PlacementTrackingMode, DecompShardingStrategy, _infer_schema_info_from_op, _extract_input_specs.
- **用途 (CN)**: 该模块聚焦于分布式张量布局、放置与算子逻辑，其主要入口包括 PlacementTrackingMode, DecompShardingStrategy, _infer_schema_info_from_op, _extract_input_specs。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs
"""
Decomposition-based sharding propagation for DTensor.

When an operator doesn't have a registered sharding strategy, we derive one by
tracing through its decomposition. The decomposed ops (which do have strategies)
determine how placements propagate through the original op.
"""

from __future__ import annotations

import itertools
from typing import Any, TYPE_CHECKING

import torch
from torch._decomp import decomposition_table
from torch.distributed._functional_collectives import _are_we_tracing
from torch.distributed.device_mesh import DeviceMesh
from torch.distributed.tensor._dtensor_spec import DTensorSpec
from torch.distributed.tensor._op_schema import OpSchema, OpStrategy, RuntimeSchemaInfo
````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Starts the docstring for the module. | CN: 开始定义 module 的文档字符串。
- **L3** EN: Continues the docstring text for the module. | CN: 继续补充 module 的文档字符串内容。
- **L4** EN: Continues the docstring text for the module. | CN: 继续补充 module 的文档字符串内容。
- **L5** EN: Continues the docstring text for the module. | CN: 继续补充 module 的文档字符串内容。
- **L6** EN: Continues the docstring text for the module. | CN: 继续补充 module 的文档字符串内容。
- **L7** EN: Continues the docstring text for the module. | CN: 继续补充 module 的文档字符串内容。
- **L8** EN: Closes the docstring for the module. | CN: 结束 module 的文档字符串。
- **L9** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L10** EN: Imports selected names from `__future__`. | CN: 从 `__future__` 导入指定名称。
- **L11** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L12** EN: Imports module dependencies: `itertools`. | CN: 导入模块依赖：`itertools`。
- **L13** EN: Imports selected names from `typing`. | CN: 从 `typing` 导入指定名称。
- **L14** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L15** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L16** EN: Imports selected names from `torch._decomp`. | CN: 从 `torch._decomp` 导入指定名称。
- **L17** EN: Imports selected names from `torch.distributed._functional_collectives`. | CN: 从 `torch.distributed._functional_collectives` 导入指定名称。
- **L18** EN: Imports selected names from `torch.distributed.device_mesh`. | CN: 从 `torch.distributed.device_mesh` 导入指定名称。
- **L19** EN: Imports selected names from `torch.distributed.tensor._dtensor_spec`. | CN: 从 `torch.distributed.tensor._dtensor_spec` 导入指定名称。
- **L20** EN: Imports selected names from `torch.distributed.tensor._op_schema`. | CN: 从 `torch.distributed.tensor._op_schema` 导入指定名称。

### Lines 21-40 / 第 21-40 行

````python
from torch.distributed.tensor._utils import try_find_mesh_from_args
from torch.distributed.tensor.placement_types import (
    _StridedShard,
    Placement,
    Replicate,
    Shard,
)
from torch.fx.experimental.symbolic_shapes import GuardOnDataDependentSymNode
from torch.utils._python_dispatch import TorchDispatchMode


def _infer_schema_info_from_op(op: OpOverload) -> RuntimeSchemaInfo:
    """Infer RuntimeSchemaInfo from an operator's schema for decomposition ops"""
    schema = op._schema

    # Find first non-tensor positional arg index
    static_argnum = None
    for i, arg in enumerate(schema.arguments):
        if arg.kwarg_only:
            break
````

- **L21** EN: Imports selected names from `torch.distributed.tensor._utils`. | CN: 从 `torch.distributed.tensor._utils` 导入指定名称。
- **L22** EN: Imports selected names from `torch.distributed.tensor.placement_types`. | CN: 从 `torch.distributed.tensor.placement_types` 导入指定名称。
- **L23** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L24** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L25** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L26** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L27** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L28** EN: Imports selected names from `torch.fx.experimental.symbolic_shapes`. | CN: 从 `torch.fx.experimental.symbolic_shapes` 导入指定名称。
- **L29** EN: Imports selected names from `torch.utils._python_dispatch`. | CN: 从 `torch.utils._python_dispatch` 导入指定名称。
- **L30** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L31** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L32** EN: Defines function `_infer_schema_info_from_op`. | CN: 定义函数 `_infer_schema_info_from_op`。
- **L33** EN: Docstring line documenting the function _infer_schema_info_from_op. | CN: 这是记录 function _infer_schema_info_from_op 的文档字符串。
- **L34** EN: Assigns or updates `schema`. | CN: 对 `schema` 进行赋值或更新。
- **L35** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L36** EN: Keeps the inline comment or directive: Find first non-tensor positional arg index | CN: 保留这一行注释或指令：Find first non-tensor positional arg index
- **L37** EN: Assigns or updates `static_argnum`. | CN: 对 `static_argnum` 进行赋值或更新。
- **L38** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L39** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L40** EN: Exits the current loop immediately. | CN: 立即退出当前循环。

### Lines 41-60 / 第 41-60 行

````python
        if arg.type.kind() != "TensorType" and static_argnum is None:
            static_argnum = i
            break

    # Find keyword-only args that aren't tensors
    kwarg_only_names = []
    for arg in schema.arguments:
        if arg.kwarg_only and arg.type.kind() != "TensorType":
            kwarg_only_names.append(arg.name)

    kwargs = {}
    if static_argnum is not None:
        kwargs["static_argnum"] = static_argnum
    if kwarg_only_names:
        # pyrefly: ignore [unsupported-operation]
        kwargs["static_kwargkey"] = kwarg_only_names

    # pyrefly: ignore [bad-argument-type]
    return RuntimeSchemaInfo(**kwargs)

````

- **L41** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L42** EN: Assigns or updates `static_argnum`. | CN: 对 `static_argnum` 进行赋值或更新。
- **L43** EN: Exits the current loop immediately. | CN: 立即退出当前循环。
- **L44** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L45** EN: Keeps the inline comment or directive: Find keyword-only args that aren't tensors | CN: 保留这一行注释或指令：Find keyword-only args that aren't tensors
- **L46** EN: Assigns or updates `kwarg_only_names`. | CN: 对 `kwarg_only_names` 进行赋值或更新。
- **L47** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L48** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L49** EN: Calls `kwarg_only_names.append` as part of the current workflow. | CN: 在当前流程中调用 `kwarg_only_names.append`。
- **L50** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L51** EN: Assigns or updates `kwargs`. | CN: 对 `kwargs` 进行赋值或更新。
- **L52** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L53** EN: Continues the implementation inside function `_infer_schema_info_from_op`. | CN: 继续说明函数 `_infer_schema_info_from_op` 内部的实现。
- **L54** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L55** EN: Keeps the inline comment or directive: pyrefly: ignore [unsupported-operation] | CN: 保留这一行注释或指令：pyrefly: ignore [unsupported-operation]
- **L56** EN: Continues the implementation inside function `_infer_schema_info_from_op`. | CN: 继续说明函数 `_infer_schema_info_from_op` 内部的实现。
- **L57** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L58** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-argument-type] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-argument-type]
- **L59** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L60** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 61-80 / 第 61-80 行

````python

from torch.utils._pytree import tree_any, tree_flatten, tree_map, tree_map_only


if TYPE_CHECKING:
    from torch._ops import OpOverload
    from torch.distributed.tensor._sharding_prop import ShardingPropagator


def _extract_input_specs(op_schema: OpSchema) -> tuple[DTensorSpec | object, ...]:
    return op_schema.args_schema + tuple(op_schema.kwargs_schema.values())


class PlacementTrackingMode(TorchDispatchMode):
    """
    TorchDispatchMode that tracks DTensor placements through op execution.

    Used during decomposition tracing: intercepts each op, propagates sharding
    via the ShardingPropagator, and records output placements on the result tensors.
    """
````

- **L61** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L62** EN: Imports selected names from `torch.utils._pytree`. | CN: 从 `torch.utils._pytree` 导入指定名称。
- **L63** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L64** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L65** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L66** EN: Imports selected names from `torch._ops`. | CN: 从 `torch._ops` 导入指定名称。
- **L67** EN: Imports selected names from `torch.distributed.tensor._sharding_prop`. | CN: 从 `torch.distributed.tensor._sharding_prop` 导入指定名称。
- **L68** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L69** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L70** EN: Defines function `_extract_input_specs`. | CN: 定义函数 `_extract_input_specs`。
- **L71** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L72** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L73** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L74** EN: Defines class `PlacementTrackingMode`. | CN: 定义类 `PlacementTrackingMode`。
- **L75** EN: Starts the docstring for the class PlacementTrackingMode. | CN: 开始定义 class PlacementTrackingMode 的文档字符串。
- **L76** EN: Continues the docstring text for the class PlacementTrackingMode. | CN: 继续补充 class PlacementTrackingMode 的文档字符串内容。
- **L77** EN: Continues the docstring text for the class PlacementTrackingMode. | CN: 继续补充 class PlacementTrackingMode 的文档字符串内容。
- **L78** EN: Continues the docstring text for the class PlacementTrackingMode. | CN: 继续补充 class PlacementTrackingMode 的文档字符串内容。
- **L79** EN: Continues the docstring text for the class PlacementTrackingMode. | CN: 继续补充 class PlacementTrackingMode 的文档字符串内容。
- **L80** EN: Closes the docstring for the class PlacementTrackingMode. | CN: 结束 class PlacementTrackingMode 的文档字符串。

### Lines 81-100 / 第 81-100 行

````python

    def __init__(self, sharding_prop: ShardingPropagator, mesh: DeviceMesh):
        super().__init__()
        self.sharding_prop = sharding_prop
        self.mesh = mesh

    def __torch_dispatch__(self, func, types, args=(), kwargs=None):
        args_schema, kwargs_schema = tree_map(
            lambda x: getattr(x, "_spec", x) if isinstance(x, torch.Tensor) else x,
            (args, kwargs or {}),
        )

        if not tree_any(
            lambda x: isinstance(x, DTensorSpec), (args_schema, kwargs_schema)
        ):
            raise NotImplementedError(f"No DTensorSpec found in args/kwargs for {func}")

        # Set schema_info so the LRU cache key includes static args
        op_schema = OpSchema(func, args_schema, kwargs_schema)
        schema_info = self.sharding_prop.op_to_schema_info.get(func)
````

- **L81** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L82** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L83** EN: Calls `super` as part of the current workflow. | CN: 在当前流程中调用 `super`。
- **L84** EN: Assigns or updates `self.sharding_prop`. | CN: 对 `self.sharding_prop` 进行赋值或更新。
- **L85** EN: Assigns or updates `self.mesh`. | CN: 对 `self.mesh` 进行赋值或更新。
- **L86** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L87** EN: Defines function `__torch_dispatch__`. | CN: 定义函数 `__torch_dispatch__`。
- **L88** EN: Assigns or updates `args_schema, kwargs_schema`. | CN: 对 `args_schema, kwargs_schema` 进行赋值或更新。
- **L89** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L90** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L91** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L92** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L93** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L94** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L95** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L96** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L97** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L98** EN: Keeps the inline comment or directive: Set schema_info so the LRU cache key includes static args | CN: 保留这一行注释或指令：Set schema_info so the LRU cache key includes static args
- **L99** EN: Assigns or updates `op_schema`. | CN: 对 `op_schema` 进行赋值或更新。
- **L100** EN: Assigns or updates `schema_info`. | CN: 对 `schema_info` 进行赋值或更新。

### Lines 101-120 / 第 101-120 行

````python
        if schema_info is None:
            schema_info = (
                self.sharding_prop.op_to_schema_info_for_single_dim_strategy.get(func)
            )
        if schema_info is not None:
            op_schema.schema_info = schema_info
            op_schema._recompute_comparison_key()

        if _are_we_tracing():
            output_sharding = self.sharding_prop.propagate_op_sharding_non_cached(
                op_schema
            )
        else:
            output_sharding = self.sharding_prop.propagate_op_sharding(op_schema)

        if (
            output_sharding.needs_redistribute  # pyrefly: ignore [missing-attribute]
            and (
                redistribute_schema
                := output_sharding.redistribute_schema  # pyrefly: ignore [missing-attribute]
````

- **L101** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L102** EN: Assigns or updates `schema_info`. | CN: 对 `schema_info` 进行赋值或更新。
- **L103** EN: Calls `self.sharding_prop.op_to_schema_info_for_single_dim_strategy.get` as part of the current workflow. | CN: 在当前流程中调用 `self.sharding_prop.op_to_schema_info_for_single_dim_strategy.get`。
- **L104** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L105** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L106** EN: Assigns or updates `op_schema.schema_info`. | CN: 对 `op_schema.schema_info` 进行赋值或更新。
- **L107** EN: Calls `op_schema._recompute_comparison_key` as part of the current workflow. | CN: 在当前流程中调用 `op_schema._recompute_comparison_key`。
- **L108** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L109** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L110** EN: Assigns or updates `output_sharding`. | CN: 对 `output_sharding` 进行赋值或更新。
- **L111** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L112** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L113** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L114** EN: Assigns or updates `output_sharding`. | CN: 对 `output_sharding` 进行赋值或更新。
- **L115** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L116** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L117** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L118** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L119** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L120** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。

### Lines 121-140 / 第 121-140 行

````python
            )
            is not None
        ):
            # a pure .needs_redistribute check is too broad; we want to ban redistribution,
            # but this flag is set for view ops that convert global shape -> local shape args.
            # During decomposition tracing on meta tensors at global shape, the shape adjustment
            # is irrelevant — only reject true redistribution.
            for orig, desired in zip(
                op_schema.args_spec,
                redistribute_schema.args_spec,  # pyrefly: ignore [missing-attribute]
            ):
                if orig.placements != desired.placements:
                    raise RuntimeError(
                        f"Decomposition requires redistribution for {func}"
                    )

        out = func(*args, **kwargs)
        # pyrefly: ignore [missing-attribute]
        self._record_output_specs(out, output_sharding.output_spec)
        return out
````

- **L121** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L122** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L123** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L124** EN: Keeps the inline comment or directive: a pure .needs_redistribute check is too broad; we want to ban redistribution, | CN: 保留这一行注释或指令：a pure .needs_redistribute check is too broad; we want to ban redistribution,
- **L125** EN: Keeps the inline comment or directive: but this flag is set for view ops that convert global shape -> local shape args. | CN: 保留这一行注释或指令：but this flag is set for view ops that convert global shape -> local shape args.
- **L126** EN: Keeps the inline comment or directive: During decomposition tracing on meta tensors at global shape, the shape adjustme | CN: 保留这一行注释或指令：During decomposition tracing on meta tensors at global shape, the shape adjustme
- **L127** EN: Keeps the inline comment or directive: is irrelevant — only reject true redistribution. | CN: 保留这一行注释或指令：is irrelevant — only reject true redistribution.
- **L128** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L129** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L130** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L131** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L132** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L133** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L134** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L135** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L136** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L137** EN: Assigns or updates `out`. | CN: 对 `out` 进行赋值或更新。
- **L138** EN: Keeps the inline comment or directive: pyrefly: ignore [missing-attribute] | CN: 保留这一行注释或指令：pyrefly: ignore [missing-attribute]
- **L139** EN: Calls `self._record_output_specs` as part of the current workflow. | CN: 在当前流程中调用 `self._record_output_specs`。
- **L140** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

### Lines 141-160 / 第 141-160 行

````python

    def _record_output_specs(self, output: Any, output_spec: DTensorSpec | Any) -> None:
        if isinstance(output, torch.Tensor) and output_spec is not None:
            output._spec = output_spec  # pyrefly: ignore [missing-attribute]
        elif isinstance(output, (tuple, list)) and isinstance(
            output_spec, (tuple, list)
        ):
            for t, s in zip(output, output_spec):
                self._record_output_specs(t, s)


class DecompShardingStrategy:
    """
    Generates sharding strategies for ops by tracing through their decompositions.

    For each candidate input placement combination, runs the decomposition on meta
    tensors under PlacementTrackingMode to determine the output placement. These
    single-dimension strategies are then expanded to the full mesh.
    """

````

- **L141** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L142** EN: Defines function `_record_output_specs`. | CN: 定义函数 `_record_output_specs`。
- **L143** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L144** EN: Assigns or updates `output._spec`. | CN: 对 `output._spec` 进行赋值或更新。
- **L145** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L146** EN: Continues the implementation inside function `_record_output_specs`. | CN: 继续说明函数 `_record_output_specs` 内部的实现。
- **L147** EN: Continues the implementation inside function `_record_output_specs`. | CN: 继续说明函数 `_record_output_specs` 内部的实现。
- **L148** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L149** EN: Calls `self._record_output_specs` as part of the current workflow. | CN: 在当前流程中调用 `self._record_output_specs`。
- **L150** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L151** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L152** EN: Defines class `DecompShardingStrategy`. | CN: 定义类 `DecompShardingStrategy`。
- **L153** EN: Starts the docstring for the class DecompShardingStrategy. | CN: 开始定义 class DecompShardingStrategy 的文档字符串。
- **L154** EN: Continues the docstring text for the class DecompShardingStrategy. | CN: 继续补充 class DecompShardingStrategy 的文档字符串内容。
- **L155** EN: Continues the docstring text for the class DecompShardingStrategy. | CN: 继续补充 class DecompShardingStrategy 的文档字符串内容。
- **L156** EN: Continues the docstring text for the class DecompShardingStrategy. | CN: 继续补充 class DecompShardingStrategy 的文档字符串内容。
- **L157** EN: Continues the docstring text for the class DecompShardingStrategy. | CN: 继续补充 class DecompShardingStrategy 的文档字符串内容。
- **L158** EN: Continues the docstring text for the class DecompShardingStrategy. | CN: 继续补充 class DecompShardingStrategy 的文档字符串内容。
- **L159** EN: Closes the docstring for the class DecompShardingStrategy. | CN: 结束 class DecompShardingStrategy 的文档字符串。
- **L160** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 161-180 / 第 161-180 行

````python
    def __init__(self, sharding_prop: ShardingPropagator):
        self.sharding_prop = sharding_prop
        # Cache fake meshes per device type to avoid repeated allocation.
        # A fake size-1 mesh ensures identical strategy computation across all ranks
        # during decomposition tracing, avoiding potential SPMD divergence.
        # False negatives are avoided (all sizes % 1 == 0), while false positives
        # are caught on expansion to the real, multi-dim device mesh.
        self._fake_meshes: dict[str, DeviceMesh] = {}

    def _get_fake_mesh(self, device_type: str) -> DeviceMesh:
        fake_mesh = self._fake_meshes.get(device_type)
        if fake_mesh is None:
            fake_mesh = DeviceMesh(device_type, [0], _init_backend=False, _rank=0)
            self._fake_meshes[device_type] = fake_mesh
        return fake_mesh

    @staticmethod
    def has_decomp(op: OpOverload) -> bool:
        # Check if op has a decomposition (explicit or CIA)
        return op in decomposition_table or op._can_decompose()
````

- **L161** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L162** EN: Assigns or updates `self.sharding_prop`. | CN: 对 `self.sharding_prop` 进行赋值或更新。
- **L163** EN: Keeps the inline comment or directive: Cache fake meshes per device type to avoid repeated allocation. | CN: 保留这一行注释或指令：Cache fake meshes per device type to avoid repeated allocation.
- **L164** EN: Keeps the inline comment or directive: A fake size-1 mesh ensures identical strategy computation across all ranks | CN: 保留这一行注释或指令：A fake size-1 mesh ensures identical strategy computation across all ranks
- **L165** EN: Keeps the inline comment or directive: during decomposition tracing, avoiding potential SPMD divergence. | CN: 保留这一行注释或指令：during decomposition tracing, avoiding potential SPMD divergence.
- **L166** EN: Keeps the inline comment or directive: False negatives are avoided (all sizes % 1 == 0), while false positives | CN: 保留这一行注释或指令：False negatives are avoided (all sizes % 1 == 0), while false positives
- **L167** EN: Keeps the inline comment or directive: are caught on expansion to the real, multi-dim device mesh. | CN: 保留这一行注释或指令：are caught on expansion to the real, multi-dim device mesh.
- **L168** EN: Assigns or updates `self._fake_meshes`. | CN: 对 `self._fake_meshes` 进行赋值或更新。
- **L169** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L170** EN: Defines function `_get_fake_mesh`. | CN: 定义函数 `_get_fake_mesh`。
- **L171** EN: Assigns or updates `fake_mesh`. | CN: 对 `fake_mesh` 进行赋值或更新。
- **L172** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L173** EN: Assigns or updates `fake_mesh`. | CN: 对 `fake_mesh` 进行赋值或更新。
- **L174** EN: Assigns or updates `self._fake_meshes[device_type]`. | CN: 对 `self._fake_meshes[device_type]` 进行赋值或更新。
- **L175** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L176** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L177** EN: Applies decorator `staticmethod` to the following definition. | CN: 将装饰器 `staticmethod` 应用于后续定义。
- **L178** EN: Defines function `has_decomp`. | CN: 定义函数 `has_decomp`。
- **L179** EN: Keeps the inline comment or directive: Check if op has a decomposition (explicit or CIA) | CN: 保留这一行注释或指令：Check if op has a decomposition (explicit or CIA)
- **L180** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

### Lines 181-200 / 第 181-200 行

````python

    def ensure_schema_info(self, op: OpOverload) -> None:
        """
        Register schema_info for decomposition op on first invocation.
        Needed for correct shard prop cache key.
        """
        if op not in self.sharding_prop.op_to_schema_info:
            schema_info = _infer_schema_info_from_op(op)
            self.sharding_prop.op_to_schema_info[op] = schema_info

    def propagate_strategy(
        self,
        op_schema: OpSchema,
    ) -> OpStrategy | None:
        if not tree_any(
            lambda x: isinstance(x, DTensorSpec),
            (op_schema.args_schema, op_schema.kwargs_schema),
        ):
            return None

````

- **L181** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L182** EN: Defines function `ensure_schema_info`. | CN: 定义函数 `ensure_schema_info`。
- **L183** EN: Starts the docstring for the function ensure_schema_info. | CN: 开始定义 function ensure_schema_info 的文档字符串。
- **L184** EN: Continues the docstring text for the function ensure_schema_info. | CN: 继续补充 function ensure_schema_info 的文档字符串内容。
- **L185** EN: Continues the docstring text for the function ensure_schema_info. | CN: 继续补充 function ensure_schema_info 的文档字符串内容。
- **L186** EN: Closes the docstring for the function ensure_schema_info. | CN: 结束 function ensure_schema_info 的文档字符串。
- **L187** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L188** EN: Assigns or updates `schema_info`. | CN: 对 `schema_info` 进行赋值或更新。
- **L189** EN: Assigns or updates `self.sharding_prop.op_to_schema_info[op]`. | CN: 对 `self.sharding_prop.op_to_schema_info[op]` 进行赋值或更新。
- **L190** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L191** EN: Defines function `propagate_strategy`. | CN: 定义函数 `propagate_strategy`。
- **L192** EN: Continues the implementation inside function `propagate_strategy`. | CN: 继续说明函数 `propagate_strategy` 内部的实现。
- **L193** EN: Continues the implementation inside function `propagate_strategy`. | CN: 继续说明函数 `propagate_strategy` 内部的实现。
- **L194** EN: Continues the implementation inside function `propagate_strategy`. | CN: 继续说明函数 `propagate_strategy` 内部的实现。
- **L195** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L196** EN: Continues the implementation inside function `propagate_strategy`. | CN: 继续说明函数 `propagate_strategy` 内部的实现。
- **L197** EN: Continues the implementation inside function `propagate_strategy`. | CN: 继续说明函数 `propagate_strategy` 内部的实现。
- **L198** EN: Continues the implementation inside function `propagate_strategy`. | CN: 继续说明函数 `propagate_strategy` 内部的实现。
- **L199** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L200** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 201-220 / 第 201-220 行

````python
        candidate_placements = self._get_candidate_placements(op_schema)
        mesh = try_find_mesh_from_args(
            op_schema.op,
            op_schema.args_schema + tuple(op_schema.kwargs_schema.values()),
        )

        fake_mesh = self._get_fake_mesh(mesh.device_type)
        single_dim_strategies = []
        output_placements: list[Placement | tuple[Placement, ...]] = []
        for input_placements in candidate_placements:
            try:
                output = self._propagate_through_decomp(
                    op_schema,
                    input_placements,
                    fake_mesh,
                )
            except NotImplementedError:
                return None
            except GuardOnDataDependentSymNode:
                return None
````

- **L201** EN: Assigns or updates `candidate_placements`. | CN: 对 `candidate_placements` 进行赋值或更新。
- **L202** EN: Assigns or updates `mesh`. | CN: 对 `mesh` 进行赋值或更新。
- **L203** EN: Continues the implementation inside function `propagate_strategy`. | CN: 继续说明函数 `propagate_strategy` 内部的实现。
- **L204** EN: Continues the implementation inside function `propagate_strategy`. | CN: 继续说明函数 `propagate_strategy` 内部的实现。
- **L205** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L206** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L207** EN: Assigns or updates `fake_mesh`. | CN: 对 `fake_mesh` 进行赋值或更新。
- **L208** EN: Assigns or updates `single_dim_strategies`. | CN: 对 `single_dim_strategies` 进行赋值或更新。
- **L209** EN: Assigns or updates `output_placements`. | CN: 对 `output_placements` 进行赋值或更新。
- **L210** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L211** EN: Starts a protected block for exception handling. | CN: 开始一个受保护的异常处理代码块。
- **L212** EN: Assigns or updates `output`. | CN: 对 `output` 进行赋值或更新。
- **L213** EN: Continues the implementation inside function `propagate_strategy`. | CN: 继续说明函数 `propagate_strategy` 内部的实现。
- **L214** EN: Continues the implementation inside function `propagate_strategy`. | CN: 继续说明函数 `propagate_strategy` 内部的实现。
- **L215** EN: Continues the implementation inside function `propagate_strategy`. | CN: 继续说明函数 `propagate_strategy` 内部的实现。
- **L216** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L217** EN: Handles a matching exception path. | CN: 处理匹配到的异常路径。
- **L218** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L219** EN: Handles a matching exception path. | CN: 处理匹配到的异常路径。
- **L220** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

### Lines 221-240 / 第 221-240 行

````python
            except (RuntimeError, KeyError, IndexError):
                # TODO(pianpwk): RuntimeError is raised when redistribution is detected; switch to a custom error type
                # Runtime/KeyError/IndexError can also occur in view ops
                continue

            output_placements = (
                [output] if not isinstance(output, tuple) else list(output)
            )
            single_dim_strategies.append(output_placements + list(input_placements))

        if not single_dim_strategies:
            raise AssertionError(
                "Sharding propagation should have produced at least Replicate() strategy"
            )

        n_outputs = len(output_placements)
        strategy_schema = self.sharding_prop._wrap_with_op_strategy(op_schema)
        # Import here to avoid circular import at module load time
        from torch.distributed.tensor._ops.utils import expand_to_full_mesh_op_strategy

````

- **L221** EN: Handles a matching exception path. | CN: 处理匹配到的异常路径。
- **L222** EN: Keeps the inline comment or directive: TODO(pianpwk): RuntimeError is raised when redistribution is detected; switch to | CN: 保留这一行注释或指令：TODO(pianpwk): RuntimeError is raised when redistribution is detected; switch to
- **L223** EN: Keeps the inline comment or directive: Runtime/KeyError/IndexError can also occur in view ops | CN: 保留这一行注释或指令：Runtime/KeyError/IndexError can also occur in view ops
- **L224** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L225** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L226** EN: Assigns or updates `output_placements`. | CN: 对 `output_placements` 进行赋值或更新。
- **L227** EN: Continues the implementation inside function `propagate_strategy`. | CN: 继续说明函数 `propagate_strategy` 内部的实现。
- **L228** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L229** EN: Calls `single_dim_strategies.append` as part of the current workflow. | CN: 在当前流程中调用 `single_dim_strategies.append`。
- **L230** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L231** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L232** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L233** EN: Continues the implementation inside function `propagate_strategy`. | CN: 继续说明函数 `propagate_strategy` 内部的实现。
- **L234** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L235** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L236** EN: Assigns or updates `n_outputs`. | CN: 对 `n_outputs` 进行赋值或更新。
- **L237** EN: Assigns or updates `strategy_schema`. | CN: 对 `strategy_schema` 进行赋值或更新。
- **L238** EN: Keeps the inline comment or directive: Import here to avoid circular import at module load time | CN: 保留这一行注释或指令：Import here to avoid circular import at module load time
- **L239** EN: Imports selected names from `torch.distributed.tensor._ops.utils`. | CN: 从 `torch.distributed.tensor._ops.utils` 导入指定名称。
- **L240** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 241-260 / 第 241-260 行

````python
        return expand_to_full_mesh_op_strategy(
            mesh, strategy_schema, single_dim_strategies, input_index=n_outputs
        )

    def _propagate_through_decomp(
        self,
        op_schema: OpSchema,
        placement: tuple[Placement | None],
        mesh: DeviceMesh,
    ) -> Placement | tuple[Placement, ...]:
        op = op_schema.op
        if op in decomposition_table:
            decomp_fn = decomposition_table[op]
        elif op._can_decompose():
            decomp_fn = op.decompose
        else:
            raise NotImplementedError(f"No decomposition found for {op}")

        placement_iter = iter(placement)

````

- **L241** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L242** EN: Assigns or updates `mesh, strategy_schema, single_dim_strategies, input_index`. | CN: 对 `mesh, strategy_schema, single_dim_strategies, input_index` 进行赋值或更新。
- **L243** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L244** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L245** EN: Defines function `_propagate_through_decomp`. | CN: 定义函数 `_propagate_through_decomp`。
- **L246** EN: Continues the implementation inside function `_propagate_through_decomp`. | CN: 继续说明函数 `_propagate_through_decomp` 内部的实现。
- **L247** EN: Continues the implementation inside function `_propagate_through_decomp`. | CN: 继续说明函数 `_propagate_through_decomp` 内部的实现。
- **L248** EN: Continues the implementation inside function `_propagate_through_decomp`. | CN: 继续说明函数 `_propagate_through_decomp` 内部的实现。
- **L249** EN: Continues the implementation inside function `_propagate_through_decomp`. | CN: 继续说明函数 `_propagate_through_decomp` 内部的实现。
- **L250** EN: Continues the implementation inside function `_propagate_through_decomp`. | CN: 继续说明函数 `_propagate_through_decomp` 内部的实现。
- **L251** EN: Assigns or updates `op`. | CN: 对 `op` 进行赋值或更新。
- **L252** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L253** EN: Assigns or updates `decomp_fn`. | CN: 对 `decomp_fn` 进行赋值或更新。
- **L254** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L255** EN: Assigns or updates `decomp_fn`. | CN: 对 `decomp_fn` 进行赋值或更新。
- **L256** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L257** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L258** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L259** EN: Assigns or updates `placement_iter`. | CN: 对 `placement_iter` 进行赋值或更新。
- **L260** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 261-280 / 第 261-280 行

````python
        def to_meta(x):
            p = next(placement_iter)
            if isinstance(x, DTensorSpec):
                # pyrefly: ignore [missing-attribute]
                meta = torch.empty(x.shape, dtype=x.tensor_meta.dtype, device="meta")
                # pyrefly: ignore [missing-attribute]
                meta._spec = DTensorSpec(mesh, (p,), tensor_meta=x.tensor_meta)
                return meta
            return x

        # Disable LocalTensorMode during decomposition tracing to prevent
        # interference with meta tensor operations
        from torch.distributed._local_tensor import maybe_disable_local_tensor_mode

        with maybe_disable_local_tensor_mode():
            # Create meta tensors and run decomposition outside LocalTensorMode
            args_meta = tree_map(to_meta, op_schema.args_schema)
            kwargs_meta = tree_map(to_meta, op_schema.kwargs_schema)

            with PlacementTrackingMode(self.sharding_prop, mesh):
````

- **L261** EN: Defines function `to_meta`. | CN: 定义函数 `to_meta`。
- **L262** EN: Assigns or updates `p`. | CN: 对 `p` 进行赋值或更新。
- **L263** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L264** EN: Keeps the inline comment or directive: pyrefly: ignore [missing-attribute] | CN: 保留这一行注释或指令：pyrefly: ignore [missing-attribute]
- **L265** EN: Assigns or updates `meta`. | CN: 对 `meta` 进行赋值或更新。
- **L266** EN: Keeps the inline comment or directive: pyrefly: ignore [missing-attribute] | CN: 保留这一行注释或指令：pyrefly: ignore [missing-attribute]
- **L267** EN: Assigns or updates `meta._spec`. | CN: 对 `meta._spec` 进行赋值或更新。
- **L268** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L269** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L270** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L271** EN: Keeps the inline comment or directive: Disable LocalTensorMode during decomposition tracing to prevent | CN: 保留这一行注释或指令：Disable LocalTensorMode during decomposition tracing to prevent
- **L272** EN: Keeps the inline comment or directive: interference with meta tensor operations | CN: 保留这一行注释或指令：interference with meta tensor operations
- **L273** EN: Imports selected names from `torch.distributed._local_tensor`. | CN: 从 `torch.distributed._local_tensor` 导入指定名称。
- **L274** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L275** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L276** EN: Keeps the inline comment or directive: Create meta tensors and run decomposition outside LocalTensorMode | CN: 保留这一行注释或指令：Create meta tensors and run decomposition outside LocalTensorMode
- **L277** EN: Assigns or updates `args_meta`. | CN: 对 `args_meta` 进行赋值或更新。
- **L278** EN: Assigns or updates `kwargs_meta`. | CN: 对 `kwargs_meta` 进行赋值或更新。
- **L279** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L280** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。

### Lines 281-300 / 第 281-300 行

````python
                output = decomp_fn(*args_meta, **kwargs_meta)

        def get_placement(t):
            if isinstance(t, torch.Tensor):
                spec = getattr(t, "_spec", None)
                return spec.placements[0] if spec else None
            return None

        result = tree_map(get_placement, output)
        if isinstance(result, (tuple, list)):
            flat = [p for p in result if p is not None]
            return flat[0] if len(flat) == 1 else tuple(flat)
        return result

    @staticmethod
    def _get_candidate_placements(
        op_schema: OpSchema,
    ) -> list[tuple[Placement | None]]:
        tensor_specs = _extract_input_specs(op_schema)
        flat_specs, _ = tree_flatten(list(tensor_specs))
````

- **L281** EN: Assigns or updates `output`. | CN: 对 `output` 进行赋值或更新。
- **L282** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L283** EN: Defines function `get_placement`. | CN: 定义函数 `get_placement`。
- **L284** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L285** EN: Assigns or updates `spec`. | CN: 对 `spec` 进行赋值或更新。
- **L286** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L287** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L288** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L289** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L290** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L291** EN: Assigns or updates `flat`. | CN: 对 `flat` 进行赋值或更新。
- **L292** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L293** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L294** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L295** EN: Applies decorator `staticmethod` to the following definition. | CN: 将装饰器 `staticmethod` 应用于后续定义。
- **L296** EN: Defines function `_get_candidate_placements`. | CN: 定义函数 `_get_candidate_placements`。
- **L297** EN: Continues the implementation inside function `_get_candidate_placements`. | CN: 继续说明函数 `_get_candidate_placements` 内部的实现。
- **L298** EN: Continues the implementation inside function `_get_candidate_placements`. | CN: 继续说明函数 `_get_candidate_placements` 内部的实现。
- **L299** EN: Assigns or updates `tensor_specs`. | CN: 对 `tensor_specs` 进行赋值或更新。
- **L300** EN: Assigns or updates `flat_specs, _`. | CN: 对 `flat_specs, _` 进行赋值或更新。

### Lines 301-320 / 第 301-320 行

````python

        # Step 1: Collect unique placements across all DTensorSpec inputs
        all_placements: set[Placement] = {Replicate()}
        tree_map_only(
            DTensorSpec,
            lambda spec: all_placements.update(spec.placements),
            flat_specs,
        )

        # Step 2: For each input, use the placement set, but expand Shard/StridedShard to all tensor dims
        candidates: list[list[Placement | None]] = []
        for spec in flat_specs:
            if not isinstance(spec, DTensorSpec):
                candidates.append([None])
            else:
                options = set(all_placements)
                for p in all_placements:
                    if isinstance(p, _StridedShard):
                        options |= {
                            _StridedShard(i, split_factor=p.split_factor)
````

- **L301** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L302** EN: Keeps the inline comment or directive: Step 1: Collect unique placements across all DTensorSpec inputs | CN: 保留这一行注释或指令：Step 1: Collect unique placements across all DTensorSpec inputs
- **L303** EN: Assigns or updates `all_placements`. | CN: 对 `all_placements` 进行赋值或更新。
- **L304** EN: Calls `tree_map_only` as part of the current workflow. | CN: 在当前流程中调用 `tree_map_only`。
- **L305** EN: Continues the implementation inside function `_get_candidate_placements`. | CN: 继续说明函数 `_get_candidate_placements` 内部的实现。
- **L306** EN: Continues the implementation inside function `_get_candidate_placements`. | CN: 继续说明函数 `_get_candidate_placements` 内部的实现。
- **L307** EN: Continues the implementation inside function `_get_candidate_placements`. | CN: 继续说明函数 `_get_candidate_placements` 内部的实现。
- **L308** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L309** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L310** EN: Keeps the inline comment or directive: Step 2: For each input, use the placement set, but expand Shard/StridedShard to  | CN: 保留这一行注释或指令：Step 2: For each input, use the placement set, but expand Shard/StridedShard to 
- **L311** EN: Assigns or updates `candidates`. | CN: 对 `candidates` 进行赋值或更新。
- **L312** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L313** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L314** EN: Calls `candidates.append` as part of the current workflow. | CN: 在当前流程中调用 `candidates.append`。
- **L315** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L316** EN: Assigns or updates `options`. | CN: 对 `options` 进行赋值或更新。
- **L317** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L318** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L319** EN: Continues the implementation inside function `_get_candidate_placements`. | CN: 继续说明函数 `_get_candidate_placements` 内部的实现。
- **L320** EN: Calls `_StridedShard` as part of the current workflow. | CN: 在当前流程中调用 `_StridedShard`。

### Lines 321-328 / 第 321-328 行

````python
                            for i in range(spec.ndim)
                        }
                    elif isinstance(p, Shard):
                        options |= {Shard(i) for i in range(spec.ndim)}
                candidates.append(list(options))

        # pyrefly: ignore [bad-argument-type, no-matching-overload]
        return list(itertools.product(*candidates))
````

- **L321** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L322** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L323** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L324** EN: Continues the implementation inside function `_get_candidate_placements`. | CN: 继续说明函数 `_get_candidate_placements` 内部的实现。
- **L325** EN: Calls `candidates.append` as part of the current workflow. | CN: 在当前流程中调用 `candidates.append`。
- **L326** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L327** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-argument-type, no-matching-overload] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-argument-type, no-matching-overload]
- **L328** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

## Key Concepts / 关键概念

- **EN**: distributed tensor layouts, placements, and operators  
  **CN**: 分布式张量布局、放置与算子逻辑
- **EN**: device mesh  
  **CN**: 设备网格
- **EN**: sharding  
  **CN**: 分片
- **EN**: collective communication  
  **CN**: 集合通信
- **EN**: distributed tensors  
  **CN**: 分布式张量
- **EN**: placements  
  **CN**: 放置规则
- **EN**: Primary classes: PlacementTrackingMode, DecompShardingStrategy  
  **CN**: 主要类：PlacementTrackingMode, DecompShardingStrategy
- **EN**: Core callables: _infer_schema_info_from_op, _extract_input_specs  
  **CN**: 核心可调用对象：_infer_schema_info_from_op, _extract_input_specs

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed._functional_collectives`, `torch.distributed._local_tensor`, `torch.distributed.device_mesh`, `torch.distributed.tensor._dtensor_spec`, `torch.distributed.tensor._op_schema`, `torch.distributed.tensor._ops.utils`, `torch.distributed.tensor._sharding_prop`, `torch.distributed.tensor._utils`, `torch.distributed.tensor.placement_types`
- **PyTorch / PyTorch**: `torch`, `torch._decomp`, `torch._ops`, `torch.fx.experimental.symbolic_shapes`, `torch.utils._python_dispatch`, `torch.utils._pytree`
- **Python Stdlib / Python 标准库**: `__future__`, `itertools`, `typing`
- **Third-party / 第三方**: None detected / 未检测到

