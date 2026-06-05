# loss.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/tensor/parallel/loss.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed tensor layouts, placements, and operators. Its main entry points include loss_parallel, _find_all_reduce_mesh_dim.
- **用途 (CN)**: 该模块聚焦于分布式张量布局、放置与算子逻辑，其主要入口包括 loss_parallel, _find_all_reduce_mesh_dim。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs
# Copyright (c) Meta Platforms, Inc. and affiliates
import contextlib
from typing import cast

import torch
import torch._prims_common as utils
import torch.distributed._functional_collectives as funcol
import torch.distributed.distributed_c10d as c10d
from torch import Tensor
from torch.distributed.device_mesh import DeviceMesh
from torch.distributed.tensor import DTensor, Replicate, Shard
from torch.distributed.tensor._dtensor_spec import DTensorSpec, TensorMeta
from torch.distributed.tensor._ops._embedding_ops import _MaskPartial
from torch.distributed.tensor._ops._math_ops import (
    _skip_dim,
    Reduction,
    replicate_reduction_dims,
)
from torch.distributed.tensor._ops.utils import normalize_dim
````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Keeps the inline comment or directive: Copyright (c) Meta Platforms, Inc. and affiliates | CN: 保留这一行注释或指令：Copyright (c) Meta Platforms, Inc. and affiliates
- **L3** EN: Imports module dependencies: `contextlib`. | CN: 导入模块依赖：`contextlib`。
- **L4** EN: Imports selected names from `typing`. | CN: 从 `typing` 导入指定名称。
- **L5** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L6** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L7** EN: Imports module dependencies: `torch._prims_common as utils`. | CN: 导入模块依赖：`torch._prims_common as utils`。
- **L8** EN: Imports module dependencies: `torch.distributed._functional_collectives as funcol`. | CN: 导入模块依赖：`torch.distributed._functional_collectives as funcol`。
- **L9** EN: Imports module dependencies: `torch.distributed.distributed_c10d as c10d`. | CN: 导入模块依赖：`torch.distributed.distributed_c10d as c10d`。
- **L10** EN: Imports selected names from `torch`. | CN: 从 `torch` 导入指定名称。
- **L11** EN: Imports selected names from `torch.distributed.device_mesh`. | CN: 从 `torch.distributed.device_mesh` 导入指定名称。
- **L12** EN: Imports selected names from `torch.distributed.tensor`. | CN: 从 `torch.distributed.tensor` 导入指定名称。
- **L13** EN: Imports selected names from `torch.distributed.tensor._dtensor_spec`. | CN: 从 `torch.distributed.tensor._dtensor_spec` 导入指定名称。
- **L14** EN: Imports selected names from `torch.distributed.tensor._ops._embedding_ops`. | CN: 从 `torch.distributed.tensor._ops._embedding_ops` 导入指定名称。
- **L15** EN: Imports selected names from `torch.distributed.tensor._ops._math_ops`. | CN: 从 `torch.distributed.tensor._ops._math_ops` 导入指定名称。
- **L16** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L17** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L18** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L19** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L20** EN: Imports selected names from `torch.distributed.tensor._ops.utils`. | CN: 从 `torch.distributed.tensor._ops.utils` 导入指定名称。

### Lines 21-40 / 第 21-40 行

````python
from torch.distributed.tensor.placement_types import Placement


aten = torch.ops.aten


__all__ = ["loss_parallel"]


@contextlib.contextmanager
def loss_parallel():
    """
    A context manager that enables loss parallelism, where efficient parallelized loss computation
    can be performed when the input is sharded on the class dimension. Currently only the cross-entropy
    loss is supported.

    Within this context manager, one can use :func:`~torch.nn.functional.cross_entropy` or
    :class:`~torch.nn.CrossEntropyLoss` as usual, with the following assumptions on the input parameters.
    The corresponding ``backward()`` call, if any, also needs to happen under this context manager.

````

- **L21** EN: Imports selected names from `torch.distributed.tensor.placement_types`. | CN: 从 `torch.distributed.tensor.placement_types` 导入指定名称。
- **L22** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L23** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L24** EN: Assigns or updates `aten`. | CN: 对 `aten` 进行赋值或更新。
- **L25** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L26** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L27** EN: Assigns or updates `__all__`. | CN: 对 `__all__` 进行赋值或更新。
- **L28** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L29** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L30** EN: Applies decorator `contextlib.contextmanager` to the following definition. | CN: 将装饰器 `contextlib.contextmanager` 应用于后续定义。
- **L31** EN: Defines function `loss_parallel`. | CN: 定义函数 `loss_parallel`。
- **L32** EN: Starts the docstring for the function loss_parallel. | CN: 开始定义 function loss_parallel 的文档字符串。
- **L33** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L34** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L35** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L36** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L37** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L38** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L39** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L40** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。

### Lines 41-60 / 第 41-60 行

````python
    Args:
        input (:class:`DTensor`):
            Input logits. Assumed to be sharded on the class dimension.
        target (Union[:class:`torch.Tensor`, :class:`DTensor`]):
            Must be ground truth class indices (class probabilities currently not supported).
            Assumed to be replicated across the ``DeviceMesh``.
        weight (Union[:class:`torch.Tensor`, :class:`DTensor`], optional):
            If given, assumed to be replicated across the ``DeviceMesh``.
        label_smoothing:
            Currently not supported.

    Returns:
        A replicated :class:`DTensor`.

    Example:
        A sharded DTensor is manually created here to showcase the usage.
        In practice, it is usually the output of a TP module.

        >>> # xdoctest: +SKIP("distributed")
        >>> from torch.distributed.tensor.parallel import loss_parallel
````

- **L41** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L42** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L43** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L44** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L45** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L46** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L47** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L48** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L49** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L50** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L51** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L52** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L53** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L54** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L55** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L56** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L57** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L58** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L59** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L60** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。

### Lines 61-80 / 第 61-80 行

````python
        >>> from torch.distributed.device_mesh import init_device_mesh
        >>> ...
        >>> device_mesh = init_device_mesh("cuda", (8,))
        >>> input = torch.randn(4, 16, device="cuda", requires_grad=True)
        >>> dist_input = distribute_tensor(input, device_mesh, placements=[Shard(1)])
        >>> target = torch.randint(16, (4,), device="cuda")
        >>> with loss_parallel():
        >>>     loss = F.cross_entropy(dist_input, target, reduction="mean")
        >>>     loss.backward()
        >>> ...
    """
    _enable_custom_loss_ops()

    yield

    _disable_custom_loss_ops()


# Currently only needs to support one dimensional DeviceMesh; in general return
# the mesh_dim with placements[mesh_dim].is_shard(dim)
````

- **L61** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L62** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L63** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L64** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L65** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L66** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L67** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L68** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L69** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L70** EN: Continues the docstring text for the function loss_parallel. | CN: 继续补充 function loss_parallel 的文档字符串内容。
- **L71** EN: Closes the docstring for the function loss_parallel. | CN: 结束 function loss_parallel 的文档字符串。
- **L72** EN: Calls `_enable_custom_loss_ops` as part of the current workflow. | CN: 在当前流程中调用 `_enable_custom_loss_ops`。
- **L73** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L74** EN: Yields a value from a generator-style callable. | CN: 从生成器风格的可调用对象中产出一个值。
- **L75** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L76** EN: Calls `_disable_custom_loss_ops` as part of the current workflow. | CN: 在当前流程中调用 `_disable_custom_loss_ops`。
- **L77** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L78** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L79** EN: Keeps the inline comment or directive: Currently only needs to support one dimensional DeviceMesh; in general return | CN: 保留这一行注释或指令：Currently only needs to support one dimensional DeviceMesh; in general return
- **L80** EN: Keeps the inline comment or directive: the mesh_dim with placements[mesh_dim].is_shard(dim) | CN: 保留这一行注释或指令：the mesh_dim with placements[mesh_dim].is_shard(dim)

### Lines 81-100 / 第 81-100 行

````python
def _find_all_reduce_mesh_dim(placements: tuple[Placement, ...], dim: int) -> int:
    if not len(placements) == 1:
        raise ValueError(
            "Currently loss_parallel() only supports input on one-dimensional DeviceMesh."
        )
    if not placements[0].is_shard(dim):
        raise ValueError(
            f"loss_parallel() should be enabled only when the input tensor is sharded on dimension {dim}."
        )
    return 0


def _cast_to_dtensor(
    tensor, placements: tuple[Placement, ...], mesh: DeviceMesh
) -> DTensor:
    if isinstance(tensor, DTensor):
        if tensor.placements == placements:
            return tensor
        else:
            raise RuntimeError(f"Expected {placements} but got {tensor.placements}.")
````

- **L81** EN: Defines function `_find_all_reduce_mesh_dim`. | CN: 定义函数 `_find_all_reduce_mesh_dim`。
- **L82** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L83** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L84** EN: Continues the implementation inside function `_find_all_reduce_mesh_dim`. | CN: 继续说明函数 `_find_all_reduce_mesh_dim` 内部的实现。
- **L85** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L86** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L87** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L88** EN: Continues the implementation inside function `_find_all_reduce_mesh_dim`. | CN: 继续说明函数 `_find_all_reduce_mesh_dim` 内部的实现。
- **L89** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L90** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L91** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L92** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L93** EN: Defines function `_cast_to_dtensor`. | CN: 定义函数 `_cast_to_dtensor`。
- **L94** EN: Continues the implementation inside function `_cast_to_dtensor`. | CN: 继续说明函数 `_cast_to_dtensor` 内部的实现。
- **L95** EN: Continues the implementation inside function `_cast_to_dtensor`. | CN: 继续说明函数 `_cast_to_dtensor` 内部的实现。
- **L96** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L97** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L98** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L99** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L100** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。

### Lines 101-120 / 第 101-120 行

````python
    elif isinstance(tensor, torch.Tensor):
        return DTensor.from_local(
            tensor, device_mesh=mesh, placements=placements, run_check=False
        )
    else:
        raise TypeError(f"Unsupported type {type(tensor)}")


def _propagate_tensor_meta(
    op_call: torch._ops.OpOverload,
    args: tuple[object, ...],
    kwargs: dict[str, object],
) -> TensorMeta:
    op_info = DTensor._op_dispatcher.unwrap_to_op_info(op_call, args, kwargs)
    if op_info.schema is None:
        raise AssertionError(
            "op_info.schema should not be None after unwrap_to_op_info"
        )
    tensor_meta = DTensor._op_dispatcher.sharding_propagator._propagate_tensor_meta(
        op_info.schema
````

- **L101** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L102** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L103** EN: Assigns or updates `tensor, device_mesh`. | CN: 对 `tensor, device_mesh` 进行赋值或更新。
- **L104** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L105** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L106** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L107** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L108** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L109** EN: Defines function `_propagate_tensor_meta`. | CN: 定义函数 `_propagate_tensor_meta`。
- **L110** EN: Continues the implementation inside function `_propagate_tensor_meta`. | CN: 继续说明函数 `_propagate_tensor_meta` 内部的实现。
- **L111** EN: Continues the implementation inside function `_propagate_tensor_meta`. | CN: 继续说明函数 `_propagate_tensor_meta` 内部的实现。
- **L112** EN: Continues the implementation inside function `_propagate_tensor_meta`. | CN: 继续说明函数 `_propagate_tensor_meta` 内部的实现。
- **L113** EN: Continues the implementation inside function `_propagate_tensor_meta`. | CN: 继续说明函数 `_propagate_tensor_meta` 内部的实现。
- **L114** EN: Assigns or updates `op_info`. | CN: 对 `op_info` 进行赋值或更新。
- **L115** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L116** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L117** EN: Continues the implementation inside function `_propagate_tensor_meta`. | CN: 继续说明函数 `_propagate_tensor_meta` 内部的实现。
- **L118** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L119** EN: Assigns or updates `tensor_meta`. | CN: 对 `tensor_meta` 进行赋值或更新。
- **L120** EN: Continues the implementation inside function `_propagate_tensor_meta`. | CN: 继续说明函数 `_propagate_tensor_meta` 内部的实现。

### Lines 121-140 / 第 121-140 行

````python
    )
    if isinstance(tensor_meta, TensorMeta):
        return tensor_meta
    elif isinstance(tensor_meta, tuple):
        # pyrefly: ignore [bad-return]
        return tensor_meta[0]
    else:
        raise RuntimeError(f"Unexpected tensor meta type: {type(tensor_meta)}.")


# NOTE: The implementation follows torch._decomp.decomposition._log_softmax,
# with all_reduce manually inserted to perform distributed computation.
def _log_softmax(x, dim, half_to_float, mesh, mesh_dim):
    if half_to_float:
        if x.dtype != torch.half:
            raise AssertionError
    computation_dtype, result_dtype = utils.elementwise_dtypes(
        x, type_promotion_kind=utils.ELEMENTWISE_TYPE_PROMOTION_KIND.DEFAULT
    )
    x = x.to(dtype=computation_dtype, memory_format=torch.contiguous_format)
````

- **L121** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L122** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L123** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L124** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L125** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-return] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-return]
- **L126** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L127** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L128** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L129** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L130** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L131** EN: Keeps the inline comment or directive: NOTE: The implementation follows torch._decomp.decomposition._log_softmax, | CN: 保留这一行注释或指令：NOTE: The implementation follows torch._decomp.decomposition._log_softmax,
- **L132** EN: Keeps the inline comment or directive: with all_reduce manually inserted to perform distributed computation. | CN: 保留这一行注释或指令：with all_reduce manually inserted to perform distributed computation.
- **L133** EN: Defines function `_log_softmax`. | CN: 定义函数 `_log_softmax`。
- **L134** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L135** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L136** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L137** EN: Assigns or updates `computation_dtype, result_dtype`. | CN: 对 `computation_dtype, result_dtype` 进行赋值或更新。
- **L138** EN: Assigns or updates `x, type_promotion_kind`. | CN: 对 `x, type_promotion_kind` 进行赋值或更新。
- **L139** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L140** EN: Assigns or updates `x`. | CN: 对 `x` 进行赋值或更新。

### Lines 141-160 / 第 141-160 行

````python
    if x.numel() == 0:
        shifted = x
    else:
        x_max = torch.amax(x, dim, keepdim=True)
        x_max = funcol.all_reduce(
            x_max, reduceOp=c10d.ReduceOp.MAX.name, group=(mesh, mesh_dim)
        )
        shifted = x - x_max
    shifted_sumexp = torch.sum(torch.exp(shifted), dim, keepdim=True)
    shifted_sumexp = funcol.all_reduce(
        shifted_sumexp, reduceOp=c10d.ReduceOp.SUM.name, group=(mesh, mesh_dim)
    )
    shifted_logsumexp = torch.log(shifted_sumexp)
    result = shifted - shifted_logsumexp
    if not half_to_float:
        result = result.to(result_dtype)
    return result


def _log_softmax_handler(
````

- **L141** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L142** EN: Assigns or updates `shifted`. | CN: 对 `shifted` 进行赋值或更新。
- **L143** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L144** EN: Assigns or updates `x_max`. | CN: 对 `x_max` 进行赋值或更新。
- **L145** EN: Assigns or updates `x_max`. | CN: 对 `x_max` 进行赋值或更新。
- **L146** EN: Assigns or updates `x_max, reduceOp`. | CN: 对 `x_max, reduceOp` 进行赋值或更新。
- **L147** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L148** EN: Assigns or updates `shifted`. | CN: 对 `shifted` 进行赋值或更新。
- **L149** EN: Assigns or updates `shifted_sumexp`. | CN: 对 `shifted_sumexp` 进行赋值或更新。
- **L150** EN: Assigns or updates `shifted_sumexp`. | CN: 对 `shifted_sumexp` 进行赋值或更新。
- **L151** EN: Assigns or updates `shifted_sumexp, reduceOp`. | CN: 对 `shifted_sumexp, reduceOp` 进行赋值或更新。
- **L152** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L153** EN: Assigns or updates `shifted_logsumexp`. | CN: 对 `shifted_logsumexp` 进行赋值或更新。
- **L154** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L155** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L156** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L157** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L158** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L159** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L160** EN: Defines function `_log_softmax_handler`. | CN: 定义函数 `_log_softmax_handler`。

### Lines 161-180 / 第 161-180 行

````python
    op_call: torch._ops.OpOverload,
    args: tuple[object, ...],
    kwargs: dict[str, object],
) -> object:
    x = cast(DTensor, args[0])
    dim = cast(int, args[1])
    half_to_float = cast(bool, args[2])

    spec = x._spec
    dim = normalize_dim(dim, x.dim())
    mesh_dim = _find_all_reduce_mesh_dim(spec.placements, dim)

    output_tensor_meta = _propagate_tensor_meta(op_call, args, kwargs)

    res = _log_softmax(x._local_tensor, dim, half_to_float, spec.mesh, mesh_dim)

    res_spec = DTensorSpec(
        spec.mesh,
        spec.placements,
        tensor_meta=output_tensor_meta,
````

- **L161** EN: Continues the implementation inside function `_log_softmax_handler`. | CN: 继续说明函数 `_log_softmax_handler` 内部的实现。
- **L162** EN: Continues the implementation inside function `_log_softmax_handler`. | CN: 继续说明函数 `_log_softmax_handler` 内部的实现。
- **L163** EN: Continues the implementation inside function `_log_softmax_handler`. | CN: 继续说明函数 `_log_softmax_handler` 内部的实现。
- **L164** EN: Continues the implementation inside function `_log_softmax_handler`. | CN: 继续说明函数 `_log_softmax_handler` 内部的实现。
- **L165** EN: Assigns or updates `x`. | CN: 对 `x` 进行赋值或更新。
- **L166** EN: Assigns or updates `dim`. | CN: 对 `dim` 进行赋值或更新。
- **L167** EN: Assigns or updates `half_to_float`. | CN: 对 `half_to_float` 进行赋值或更新。
- **L168** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L169** EN: Assigns or updates `spec`. | CN: 对 `spec` 进行赋值或更新。
- **L170** EN: Assigns or updates `dim`. | CN: 对 `dim` 进行赋值或更新。
- **L171** EN: Assigns or updates `mesh_dim`. | CN: 对 `mesh_dim` 进行赋值或更新。
- **L172** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L173** EN: Assigns or updates `output_tensor_meta`. | CN: 对 `output_tensor_meta` 进行赋值或更新。
- **L174** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L175** EN: Assigns or updates `res`. | CN: 对 `res` 进行赋值或更新。
- **L176** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L177** EN: Assigns or updates `res_spec`. | CN: 对 `res_spec` 进行赋值或更新。
- **L178** EN: Continues the implementation inside function `_log_softmax_handler`. | CN: 继续说明函数 `_log_softmax_handler` 内部的实现。
- **L179** EN: Continues the implementation inside function `_log_softmax_handler`. | CN: 继续说明函数 `_log_softmax_handler` 内部的实现。
- **L180** EN: Assigns or updates `tensor_meta`. | CN: 对 `tensor_meta` 进行赋值或更新。

### Lines 181-200 / 第 181-200 行

````python
    )

    # pyrefly: ignore [bad-argument-type]
    return DTensor(
        # pyrefly: ignore [bad-argument-count]
        res,
        res_spec,
        # pyrefly: ignore [unexpected-keyword]
        requires_grad=res.requires_grad,
    )


# NOTE: As explained below at _nll_loss_and_log_softmax_backward, the
# _log_softmax_backward_handler does not actually do any computation.
def _log_softmax_backward_handler(
    op_call: torch._ops.OpOverload,
    args: tuple[object, ...],
    kwargs: dict[str, object],
) -> object:
    grad_output = cast(DTensor, args[0])
````

- **L181** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L182** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L183** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-argument-type] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-argument-type]
- **L184** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L185** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-argument-count] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-argument-count]
- **L186** EN: Continues the implementation inside function `_log_softmax_handler`. | CN: 继续说明函数 `_log_softmax_handler` 内部的实现。
- **L187** EN: Continues the implementation inside function `_log_softmax_handler`. | CN: 继续说明函数 `_log_softmax_handler` 内部的实现。
- **L188** EN: Keeps the inline comment or directive: pyrefly: ignore [unexpected-keyword] | CN: 保留这一行注释或指令：pyrefly: ignore [unexpected-keyword]
- **L189** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L190** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L191** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L192** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L193** EN: Keeps the inline comment or directive: NOTE: As explained below at _nll_loss_and_log_softmax_backward, the | CN: 保留这一行注释或指令：NOTE: As explained below at _nll_loss_and_log_softmax_backward, the
- **L194** EN: Keeps the inline comment or directive: _log_softmax_backward_handler does not actually do any computation. | CN: 保留这一行注释或指令：_log_softmax_backward_handler does not actually do any computation.
- **L195** EN: Defines function `_log_softmax_backward_handler`. | CN: 定义函数 `_log_softmax_backward_handler`。
- **L196** EN: Continues the implementation inside function `_log_softmax_backward_handler`. | CN: 继续说明函数 `_log_softmax_backward_handler` 内部的实现。
- **L197** EN: Continues the implementation inside function `_log_softmax_backward_handler`. | CN: 继续说明函数 `_log_softmax_backward_handler` 内部的实现。
- **L198** EN: Continues the implementation inside function `_log_softmax_backward_handler`. | CN: 继续说明函数 `_log_softmax_backward_handler` 内部的实现。
- **L199** EN: Continues the implementation inside function `_log_softmax_backward_handler`. | CN: 继续说明函数 `_log_softmax_backward_handler` 内部的实现。
- **L200** EN: Assigns or updates `grad_output`. | CN: 对 `grad_output` 进行赋值或更新。

### Lines 201-220 / 第 201-220 行

````python
    input_dtype = cast(torch.dtype, args[3])
    return grad_output.to(input_dtype)


# NOTE: The implementation follows torch._decomp.decomposition._nll_loss_forward,
# with customized communication inserted to perform distributed computation.
def _nll_loss_forward(
    x: Tensor,
    target: Tensor,
    weight: Tensor | None,
    local_weight: Tensor | None,
    reduction: int,
    ignore_index: int,
    input_shape: torch.Size,
    channel_dim: int,
    mesh: DeviceMesh,
    mesh_dim: int,
) -> tuple[Tensor, Tensor]:
    n_dims = x.dim()
    channel_dim = 1
````

- **L201** EN: Assigns or updates `input_dtype`. | CN: 对 `input_dtype` 进行赋值或更新。
- **L202** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L203** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L204** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L205** EN: Keeps the inline comment or directive: NOTE: The implementation follows torch._decomp.decomposition._nll_loss_forward, | CN: 保留这一行注释或指令：NOTE: The implementation follows torch._decomp.decomposition._nll_loss_forward,
- **L206** EN: Keeps the inline comment or directive: with customized communication inserted to perform distributed computation. | CN: 保留这一行注释或指令：with customized communication inserted to perform distributed computation.
- **L207** EN: Defines function `_nll_loss_forward`. | CN: 定义函数 `_nll_loss_forward`。
- **L208** EN: Continues the implementation inside function `_nll_loss_forward`. | CN: 继续说明函数 `_nll_loss_forward` 内部的实现。
- **L209** EN: Continues the implementation inside function `_nll_loss_forward`. | CN: 继续说明函数 `_nll_loss_forward` 内部的实现。
- **L210** EN: Continues the implementation inside function `_nll_loss_forward`. | CN: 继续说明函数 `_nll_loss_forward` 内部的实现。
- **L211** EN: Continues the implementation inside function `_nll_loss_forward`. | CN: 继续说明函数 `_nll_loss_forward` 内部的实现。
- **L212** EN: Continues the implementation inside function `_nll_loss_forward`. | CN: 继续说明函数 `_nll_loss_forward` 内部的实现。
- **L213** EN: Continues the implementation inside function `_nll_loss_forward`. | CN: 继续说明函数 `_nll_loss_forward` 内部的实现。
- **L214** EN: Continues the implementation inside function `_nll_loss_forward`. | CN: 继续说明函数 `_nll_loss_forward` 内部的实现。
- **L215** EN: Continues the implementation inside function `_nll_loss_forward`. | CN: 继续说明函数 `_nll_loss_forward` 内部的实现。
- **L216** EN: Continues the implementation inside function `_nll_loss_forward`. | CN: 继续说明函数 `_nll_loss_forward` 内部的实现。
- **L217** EN: Continues the implementation inside function `_nll_loss_forward`. | CN: 继续说明函数 `_nll_loss_forward` 内部的实现。
- **L218** EN: Continues the implementation inside function `_nll_loss_forward`. | CN: 继续说明函数 `_nll_loss_forward` 内部的实现。
- **L219** EN: Assigns or updates `n_dims`. | CN: 对 `n_dims` 进行赋值或更新。
- **L220** EN: Assigns or updates `channel_dim`. | CN: 对 `channel_dim` 进行赋值或更新。

### Lines 221-240 / 第 221-240 行

````python
    if n_dims < 2:
        channel_dim = 0

    def _weight_view(weight: Tensor) -> Tensor:
        if n_dims > 1:
            shape = [
                1,
            ] * n_dims
            shape[channel_dim] = weight.shape[0]
            w = weight.view(shape)
        else:
            w = weight
        return w

    if weight is not None:
        w = _weight_view(weight)
        if local_weight is None:
            raise AssertionError
        local_w = _weight_view(local_weight)
        x = x * local_w
````

- **L221** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L222** EN: Assigns or updates `channel_dim`. | CN: 对 `channel_dim` 进行赋值或更新。
- **L223** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L224** EN: Defines function `_weight_view`. | CN: 定义函数 `_weight_view`。
- **L225** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L226** EN: Assigns or updates `shape`. | CN: 对 `shape` 进行赋值或更新。
- **L227** EN: Continues the implementation inside function `_weight_view`. | CN: 继续说明函数 `_weight_view` 内部的实现。
- **L228** EN: Continues the implementation inside function `_weight_view`. | CN: 继续说明函数 `_weight_view` 内部的实现。
- **L229** EN: Assigns or updates `shape[channel_dim]`. | CN: 对 `shape[channel_dim]` 进行赋值或更新。
- **L230** EN: Assigns or updates `w`. | CN: 对 `w` 进行赋值或更新。
- **L231** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L232** EN: Assigns or updates `w`. | CN: 对 `w` 进行赋值或更新。
- **L233** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L234** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L235** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L236** EN: Assigns or updates `w`. | CN: 对 `w` 进行赋值或更新。
- **L237** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L238** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L239** EN: Assigns or updates `local_w`. | CN: 对 `local_w` 进行赋值或更新。
- **L240** EN: Assigns or updates `x`. | CN: 对 `x` 进行赋值或更新。

### Lines 241-260 / 第 241-260 行

````python
    safe_target = torch.where(target != ignore_index, target, 0)
    safe_target_ = safe_target.unsqueeze(channel_dim)

    # The following code block is a distributed version of
    # result = -torch.gather(self, channel_dim, safe_target_).squeeze(channel_dim)
    partial_placement = _MaskPartial(offset_shape=input_shape, offset_dim=channel_dim)
    safe_target_partial_ = partial_placement._partition_value(
        safe_target_, mesh, mesh_dim
    )
    result_partial = torch.gather(x, channel_dim, safe_target_partial_)
    # an all_reduce happens here
    result_reduced = partial_placement._reduce_value(result_partial, mesh, mesh_dim)
    result = -result_reduced.squeeze(channel_dim)

    result = torch.where(target != ignore_index, result, 0)

    if reduction == Reduction.NONE.value and n_dims > 1:
        total_weight = x.new_full((), 0.0)
        return result, total_weight

````

- **L241** EN: Assigns or updates `safe_target`. | CN: 对 `safe_target` 进行赋值或更新。
- **L242** EN: Assigns or updates `safe_target_`. | CN: 对 `safe_target_` 进行赋值或更新。
- **L243** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L244** EN: Keeps the inline comment or directive: The following code block is a distributed version of | CN: 保留这一行注释或指令：The following code block is a distributed version of
- **L245** EN: Keeps the inline comment or directive: result = -torch.gather(self, channel_dim, safe_target_).squeeze(channel_dim) | CN: 保留这一行注释或指令：result = -torch.gather(self, channel_dim, safe_target_).squeeze(channel_dim)
- **L246** EN: Assigns or updates `partial_placement`. | CN: 对 `partial_placement` 进行赋值或更新。
- **L247** EN: Assigns or updates `safe_target_partial_`. | CN: 对 `safe_target_partial_` 进行赋值或更新。
- **L248** EN: Continues the implementation inside function `_nll_loss_forward`. | CN: 继续说明函数 `_nll_loss_forward` 内部的实现。
- **L249** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L250** EN: Assigns or updates `result_partial`. | CN: 对 `result_partial` 进行赋值或更新。
- **L251** EN: Keeps the inline comment or directive: an all_reduce happens here | CN: 保留这一行注释或指令：an all_reduce happens here
- **L252** EN: Assigns or updates `result_reduced`. | CN: 对 `result_reduced` 进行赋值或更新。
- **L253** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L254** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L255** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L256** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L257** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L258** EN: Assigns or updates `total_weight`. | CN: 对 `total_weight` 进行赋值或更新。
- **L259** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L260** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 261-280 / 第 261-280 行

````python
    if weight is not None:
        new_shape = list(x.shape)
        new_shape[channel_dim] = -1
        # pyrefly: ignore [unbound-name]
        w = w.expand(new_shape)
        wsum = torch.gather(w, channel_dim, safe_target_).squeeze(channel_dim)
        wsum = torch.where(target != ignore_index, wsum, 0)
        total_weight = wsum.sum()
    else:
        total_weight = (target != ignore_index).sum().to(x)

    # NOTE: this is correct only on 1D DeviceMesh; o/w additional
    #       all-reduce on result and total_weight is needed
    if reduction == Reduction.SUM.value:
        result = result.sum()
    elif reduction == Reduction.MEAN.value:
        result = result.sum() / total_weight

    return result, total_weight

````

- **L261** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L262** EN: Assigns or updates `new_shape`. | CN: 对 `new_shape` 进行赋值或更新。
- **L263** EN: Assigns or updates `new_shape[channel_dim]`. | CN: 对 `new_shape[channel_dim]` 进行赋值或更新。
- **L264** EN: Keeps the inline comment or directive: pyrefly: ignore [unbound-name] | CN: 保留这一行注释或指令：pyrefly: ignore [unbound-name]
- **L265** EN: Assigns or updates `w`. | CN: 对 `w` 进行赋值或更新。
- **L266** EN: Assigns or updates `wsum`. | CN: 对 `wsum` 进行赋值或更新。
- **L267** EN: Assigns or updates `wsum`. | CN: 对 `wsum` 进行赋值或更新。
- **L268** EN: Assigns or updates `total_weight`. | CN: 对 `total_weight` 进行赋值或更新。
- **L269** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L270** EN: Assigns or updates `total_weight`. | CN: 对 `total_weight` 进行赋值或更新。
- **L271** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L272** EN: Keeps the inline comment or directive: NOTE: this is correct only on 1D DeviceMesh; o/w additional | CN: 保留这一行注释或指令：NOTE: this is correct only on 1D DeviceMesh; o/w additional
- **L273** EN: Keeps the inline comment or directive: all-reduce on result and total_weight is needed | CN: 保留这一行注释或指令：all-reduce on result and total_weight is needed
- **L274** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L275** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L276** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L277** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L278** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L279** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L280** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 281-300 / 第 281-300 行

````python

def _nll_loss_forward_handler(
    op_call: torch._ops.OpOverload,
    args: tuple[object, ...],
    kwargs: dict[str, object],
) -> object:
    x = cast(DTensor, args[0])
    target = args[1]
    weight = args[2]
    reduction = cast(int, args[3])
    ignore_index = cast(int, args[4])

    channel_dim = 1 if x.dim() >= 2 else 0
    spec = x._spec
    mesh_dim = _find_all_reduce_mesh_dim(spec.placements, channel_dim)

    # Check user input: if target and weight are not DTensors, convert them to DTensors;
    # if they are DTensors, check that they have the desired placements.
    target_placements = _skip_dim(
        replicate_reduction_dims(spec.placements, [channel_dim]), channel_dim
````

- **L281** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L282** EN: Defines function `_nll_loss_forward_handler`. | CN: 定义函数 `_nll_loss_forward_handler`。
- **L283** EN: Continues the implementation inside function `_nll_loss_forward_handler`. | CN: 继续说明函数 `_nll_loss_forward_handler` 内部的实现。
- **L284** EN: Continues the implementation inside function `_nll_loss_forward_handler`. | CN: 继续说明函数 `_nll_loss_forward_handler` 内部的实现。
- **L285** EN: Continues the implementation inside function `_nll_loss_forward_handler`. | CN: 继续说明函数 `_nll_loss_forward_handler` 内部的实现。
- **L286** EN: Continues the implementation inside function `_nll_loss_forward_handler`. | CN: 继续说明函数 `_nll_loss_forward_handler` 内部的实现。
- **L287** EN: Assigns or updates `x`. | CN: 对 `x` 进行赋值或更新。
- **L288** EN: Assigns or updates `target`. | CN: 对 `target` 进行赋值或更新。
- **L289** EN: Assigns or updates `weight`. | CN: 对 `weight` 进行赋值或更新。
- **L290** EN: Assigns or updates `reduction`. | CN: 对 `reduction` 进行赋值或更新。
- **L291** EN: Assigns or updates `ignore_index`. | CN: 对 `ignore_index` 进行赋值或更新。
- **L292** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L293** EN: Assigns or updates `channel_dim`. | CN: 对 `channel_dim` 进行赋值或更新。
- **L294** EN: Assigns or updates `spec`. | CN: 对 `spec` 进行赋值或更新。
- **L295** EN: Assigns or updates `mesh_dim`. | CN: 对 `mesh_dim` 进行赋值或更新。
- **L296** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L297** EN: Keeps the inline comment or directive: Check user input: if target and weight are not DTensors, convert them to DTensor | CN: 保留这一行注释或指令：Check user input: if target and weight are not DTensors, convert them to DTensor
- **L298** EN: Keeps the inline comment or directive: if they are DTensors, check that they have the desired placements. | CN: 保留这一行注释或指令：if they are DTensors, check that they have the desired placements.
- **L299** EN: Assigns or updates `target_placements`. | CN: 对 `target_placements` 进行赋值或更新。
- **L300** EN: Calls `replicate_reduction_dims` as part of the current workflow. | CN: 在当前流程中调用 `replicate_reduction_dims`。

### Lines 301-320 / 第 301-320 行

````python
    )
    all_replicate_placements = (Replicate(),) * spec.mesh.ndim
    target = _cast_to_dtensor(target, target_placements, spec.mesh)
    local_weight = None
    if weight is not None:
        weight = _cast_to_dtensor(weight, all_replicate_placements, spec.mesh)
        # For local computation, both (replicated) weight and (sharded) local_weight
        # are needed in _nll_loss_forward(). local_weight is generated here using
        # DTensor API, without incurring any communication.
        sharded_placements = [
            Shard(0) if i == mesh_dim else Replicate() for i in range(spec.mesh.ndim)
        ]
        local_weight = weight.redistribute(spec.mesh, sharded_placements)._local_tensor
        if local_weight.shape[0] != x._local_tensor.shape[channel_dim]:
            raise AssertionError

    if reduction == Reduction.NONE.value:
        output_placements = target_placements
    else:
        output_placements = all_replicate_placements
````

- **L301** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L302** EN: Assigns or updates `all_replicate_placements`. | CN: 对 `all_replicate_placements` 进行赋值或更新。
- **L303** EN: Assigns or updates `target`. | CN: 对 `target` 进行赋值或更新。
- **L304** EN: Assigns or updates `local_weight`. | CN: 对 `local_weight` 进行赋值或更新。
- **L305** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L306** EN: Assigns or updates `weight`. | CN: 对 `weight` 进行赋值或更新。
- **L307** EN: Keeps the inline comment or directive: For local computation, both (replicated) weight and (sharded) local_weight | CN: 保留这一行注释或指令：For local computation, both (replicated) weight and (sharded) local_weight
- **L308** EN: Keeps the inline comment or directive: are needed in _nll_loss_forward(). local_weight is generated here using | CN: 保留这一行注释或指令：are needed in _nll_loss_forward(). local_weight is generated here using
- **L309** EN: Keeps the inline comment or directive: DTensor API, without incurring any communication. | CN: 保留这一行注释或指令：DTensor API, without incurring any communication.
- **L310** EN: Assigns or updates `sharded_placements`. | CN: 对 `sharded_placements` 进行赋值或更新。
- **L311** EN: Calls `Shard` as part of the current workflow. | CN: 在当前流程中调用 `Shard`。
- **L312** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L313** EN: Assigns or updates `local_weight`. | CN: 对 `local_weight` 进行赋值或更新。
- **L314** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L315** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L316** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L317** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L318** EN: Assigns or updates `output_placements`. | CN: 对 `output_placements` 进行赋值或更新。
- **L319** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L320** EN: Assigns or updates `output_placements`. | CN: 对 `output_placements` 进行赋值或更新。

### Lines 321-340 / 第 321-340 行

````python

    # tensor inputs to _propagate_tensor_meta need to be DTensors
    # pyrefly: ignore [bad-assignment]
    args = list(args)
    # pyrefly: ignore [unsupported-operation]
    args[1], args[2] = target, weight
    output_tensor_meta = _propagate_tensor_meta(op_call, tuple(args), kwargs)

    result, total_weight = _nll_loss_forward(
        x._local_tensor,
        target._local_tensor,
        weight._local_tensor if weight is not None else None,
        local_weight,
        reduction,
        ignore_index,
        x.shape,
        channel_dim,
        spec.mesh,
        mesh_dim,
    )
````

- **L321** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L322** EN: Keeps the inline comment or directive: tensor inputs to _propagate_tensor_meta need to be DTensors | CN: 保留这一行注释或指令：tensor inputs to _propagate_tensor_meta need to be DTensors
- **L323** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-assignment] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-assignment]
- **L324** EN: Assigns or updates `args`. | CN: 对 `args` 进行赋值或更新。
- **L325** EN: Keeps the inline comment or directive: pyrefly: ignore [unsupported-operation] | CN: 保留这一行注释或指令：pyrefly: ignore [unsupported-operation]
- **L326** EN: Assigns or updates `args[1], args[2]`. | CN: 对 `args[1], args[2]` 进行赋值或更新。
- **L327** EN: Assigns or updates `output_tensor_meta`. | CN: 对 `output_tensor_meta` 进行赋值或更新。
- **L328** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L329** EN: Assigns or updates `result, total_weight`. | CN: 对 `result, total_weight` 进行赋值或更新。
- **L330** EN: Continues the implementation inside function `_nll_loss_forward_handler`. | CN: 继续说明函数 `_nll_loss_forward_handler` 内部的实现。
- **L331** EN: Continues the implementation inside function `_nll_loss_forward_handler`. | CN: 继续说明函数 `_nll_loss_forward_handler` 内部的实现。
- **L332** EN: Continues the implementation inside function `_nll_loss_forward_handler`. | CN: 继续说明函数 `_nll_loss_forward_handler` 内部的实现。
- **L333** EN: Continues the implementation inside function `_nll_loss_forward_handler`. | CN: 继续说明函数 `_nll_loss_forward_handler` 内部的实现。
- **L334** EN: Continues the implementation inside function `_nll_loss_forward_handler`. | CN: 继续说明函数 `_nll_loss_forward_handler` 内部的实现。
- **L335** EN: Continues the implementation inside function `_nll_loss_forward_handler`. | CN: 继续说明函数 `_nll_loss_forward_handler` 内部的实现。
- **L336** EN: Continues the implementation inside function `_nll_loss_forward_handler`. | CN: 继续说明函数 `_nll_loss_forward_handler` 内部的实现。
- **L337** EN: Continues the implementation inside function `_nll_loss_forward_handler`. | CN: 继续说明函数 `_nll_loss_forward_handler` 内部的实现。
- **L338** EN: Continues the implementation inside function `_nll_loss_forward_handler`. | CN: 继续说明函数 `_nll_loss_forward_handler` 内部的实现。
- **L339** EN: Continues the implementation inside function `_nll_loss_forward_handler`. | CN: 继续说明函数 `_nll_loss_forward_handler` 内部的实现。
- **L340** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 341-360 / 第 341-360 行

````python
    out_spec = DTensorSpec(spec.mesh, output_placements, tensor_meta=output_tensor_meta)

    return (
        # pyrefly: ignore [bad-argument-type]
        DTensor(
            # pyrefly: ignore [bad-argument-count]
            result,
            out_spec,
            # pyrefly: ignore [unexpected-keyword]
            requires_grad=result.requires_grad,
        ),
        total_weight,
    )


# NOTE: The backward computation of cross_entropy goes through two steps:
# backward for nll_loss and then backward for log_softmax. In loss parallel,
# the two steps are fused into the following function (called by _nll_loss_backward_handler)
# to avoid communication when target contains class indices not class probabilities.
# Also note that the _log_softmax_backward_handler does not perform computation.
````

- **L341** EN: Assigns or updates `out_spec`. | CN: 对 `out_spec` 进行赋值或更新。
- **L342** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L343** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L344** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-argument-type] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-argument-type]
- **L345** EN: Calls `DTensor` as part of the current workflow. | CN: 在当前流程中调用 `DTensor`。
- **L346** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-argument-count] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-argument-count]
- **L347** EN: Continues the implementation inside function `_nll_loss_forward_handler`. | CN: 继续说明函数 `_nll_loss_forward_handler` 内部的实现。
- **L348** EN: Continues the implementation inside function `_nll_loss_forward_handler`. | CN: 继续说明函数 `_nll_loss_forward_handler` 内部的实现。
- **L349** EN: Keeps the inline comment or directive: pyrefly: ignore [unexpected-keyword] | CN: 保留这一行注释或指令：pyrefly: ignore [unexpected-keyword]
- **L350** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L351** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L352** EN: Continues the implementation inside function `_nll_loss_forward_handler`. | CN: 继续说明函数 `_nll_loss_forward_handler` 内部的实现。
- **L353** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L354** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L355** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L356** EN: Keeps the inline comment or directive: NOTE: The backward computation of cross_entropy goes through two steps: | CN: 保留这一行注释或指令：NOTE: The backward computation of cross_entropy goes through two steps:
- **L357** EN: Keeps the inline comment or directive: backward for nll_loss and then backward for log_softmax. In loss parallel, | CN: 保留这一行注释或指令：backward for nll_loss and then backward for log_softmax. In loss parallel,
- **L358** EN: Keeps the inline comment or directive: the two steps are fused into the following function (called by _nll_loss_backwar | CN: 保留这一行注释或指令：the two steps are fused into the following function (called by _nll_loss_backwar
- **L359** EN: Keeps the inline comment or directive: to avoid communication when target contains class indices not class probabilitie | CN: 保留这一行注释或指令：to avoid communication when target contains class indices not class probabilitie
- **L360** EN: Keeps the inline comment or directive: Also note that the _log_softmax_backward_handler does not perform computation. | CN: 保留这一行注释或指令：Also note that the _log_softmax_backward_handler does not perform computation.

### Lines 361-380 / 第 361-380 行

````python
# The implementation resembles _nll_loss_backward and _log_softmax_backward_data
# from torch._decomp.decomposition.
def _nll_loss_and_log_softmax_backward(
    grad_output: Tensor,
    x: Tensor,
    target: Tensor,
    weight: Tensor | None,
    reduction: int,
    ignore_index: int,
    total_weight: Tensor,
    input_shape: torch.Size,
    channel_dim: int,
    mesh: DeviceMesh,
    mesh_dim: int,
) -> Tensor:
    channel_dim = 0 if x.dim() < 2 else 1
    if reduction == Reduction.MEAN.value:
        grad_output = grad_output / total_weight

    target = target.unsqueeze(channel_dim)
````

- **L361** EN: Keeps the inline comment or directive: The implementation resembles _nll_loss_backward and _log_softmax_backward_data | CN: 保留这一行注释或指令：The implementation resembles _nll_loss_backward and _log_softmax_backward_data
- **L362** EN: Keeps the inline comment or directive: from torch._decomp.decomposition. | CN: 保留这一行注释或指令：from torch._decomp.decomposition.
- **L363** EN: Defines function `_nll_loss_and_log_softmax_backward`. | CN: 定义函数 `_nll_loss_and_log_softmax_backward`。
- **L364** EN: Continues the implementation inside function `_nll_loss_and_log_softmax_backward`. | CN: 继续说明函数 `_nll_loss_and_log_softmax_backward` 内部的实现。
- **L365** EN: Continues the implementation inside function `_nll_loss_and_log_softmax_backward`. | CN: 继续说明函数 `_nll_loss_and_log_softmax_backward` 内部的实现。
- **L366** EN: Continues the implementation inside function `_nll_loss_and_log_softmax_backward`. | CN: 继续说明函数 `_nll_loss_and_log_softmax_backward` 内部的实现。
- **L367** EN: Continues the implementation inside function `_nll_loss_and_log_softmax_backward`. | CN: 继续说明函数 `_nll_loss_and_log_softmax_backward` 内部的实现。
- **L368** EN: Continues the implementation inside function `_nll_loss_and_log_softmax_backward`. | CN: 继续说明函数 `_nll_loss_and_log_softmax_backward` 内部的实现。
- **L369** EN: Continues the implementation inside function `_nll_loss_and_log_softmax_backward`. | CN: 继续说明函数 `_nll_loss_and_log_softmax_backward` 内部的实现。
- **L370** EN: Continues the implementation inside function `_nll_loss_and_log_softmax_backward`. | CN: 继续说明函数 `_nll_loss_and_log_softmax_backward` 内部的实现。
- **L371** EN: Continues the implementation inside function `_nll_loss_and_log_softmax_backward`. | CN: 继续说明函数 `_nll_loss_and_log_softmax_backward` 内部的实现。
- **L372** EN: Continues the implementation inside function `_nll_loss_and_log_softmax_backward`. | CN: 继续说明函数 `_nll_loss_and_log_softmax_backward` 内部的实现。
- **L373** EN: Continues the implementation inside function `_nll_loss_and_log_softmax_backward`. | CN: 继续说明函数 `_nll_loss_and_log_softmax_backward` 内部的实现。
- **L374** EN: Continues the implementation inside function `_nll_loss_and_log_softmax_backward`. | CN: 继续说明函数 `_nll_loss_and_log_softmax_backward` 内部的实现。
- **L375** EN: Continues the implementation inside function `_nll_loss_and_log_softmax_backward`. | CN: 继续说明函数 `_nll_loss_and_log_softmax_backward` 内部的实现。
- **L376** EN: Assigns or updates `channel_dim`. | CN: 对 `channel_dim` 进行赋值或更新。
- **L377** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L378** EN: Assigns or updates `grad_output`. | CN: 对 `grad_output` 进行赋值或更新。
- **L379** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L380** EN: Assigns or updates `target`. | CN: 对 `target` 进行赋值或更新。

### Lines 381-400 / 第 381-400 行

````python
    safe_target = torch.where(target != ignore_index, target, 0)
    grad_input = torch.zeros_like(x)

    # The following code block is a distributed version of
    # grad_input = torch.scatter(grad_input, channel_dim, safe_target, -1.0)
    partial_placement = _MaskPartial(offset_shape=input_shape, offset_dim=channel_dim)
    safe_target = safe_target.squeeze(channel_dim).flatten()
    masked_safe_target = partial_placement._partition_value(safe_target, mesh, mesh_dim)
    # only update grad_input to -1 if not masked
    if partial_placement.mask_buffer.data is None:
        raise AssertionError
    grad_update = partial_placement.mask_buffer.data.to(grad_input.dtype) - 1.0
    arange_1d = torch.arange(
        masked_safe_target.shape[0], device=masked_safe_target.device
    )
    # The first two cases with x.dim() <= 2 are for aten.nll_loss_backward.default;
    # the last case is for aten.nll_loss2d_backward.default.
    if x.dim() == 1:
        grad_input[masked_safe_target] = grad_update
    elif x.dim() == 2:
````

- **L381** EN: Assigns or updates `safe_target`. | CN: 对 `safe_target` 进行赋值或更新。
- **L382** EN: Assigns or updates `grad_input`. | CN: 对 `grad_input` 进行赋值或更新。
- **L383** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L384** EN: Keeps the inline comment or directive: The following code block is a distributed version of | CN: 保留这一行注释或指令：The following code block is a distributed version of
- **L385** EN: Keeps the inline comment or directive: grad_input = torch.scatter(grad_input, channel_dim, safe_target, -1.0) | CN: 保留这一行注释或指令：grad_input = torch.scatter(grad_input, channel_dim, safe_target, -1.0)
- **L386** EN: Assigns or updates `partial_placement`. | CN: 对 `partial_placement` 进行赋值或更新。
- **L387** EN: Assigns or updates `safe_target`. | CN: 对 `safe_target` 进行赋值或更新。
- **L388** EN: Assigns or updates `masked_safe_target`. | CN: 对 `masked_safe_target` 进行赋值或更新。
- **L389** EN: Keeps the inline comment or directive: only update grad_input to -1 if not masked | CN: 保留这一行注释或指令：only update grad_input to -1 if not masked
- **L390** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L391** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L392** EN: Assigns or updates `grad_update`. | CN: 对 `grad_update` 进行赋值或更新。
- **L393** EN: Assigns or updates `arange_1d`. | CN: 对 `arange_1d` 进行赋值或更新。
- **L394** EN: Assigns or updates `masked_safe_target.shape[0], device`. | CN: 对 `masked_safe_target.shape[0], device` 进行赋值或更新。
- **L395** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L396** EN: Keeps the inline comment or directive: The first two cases with x.dim() <= 2 are for aten.nll_loss_backward.default; | CN: 保留这一行注释或指令：The first two cases with x.dim() <= 2 are for aten.nll_loss_backward.default;
- **L397** EN: Keeps the inline comment or directive: the last case is for aten.nll_loss2d_backward.default. | CN: 保留这一行注释或指令：the last case is for aten.nll_loss2d_backward.default.
- **L398** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L399** EN: Assigns or updates `grad_input[masked_safe_target]`. | CN: 对 `grad_input[masked_safe_target]` 进行赋值或更新。
- **L400** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。

### Lines 401-420 / 第 401-420 行

````python
        grad_input[arange_1d, masked_safe_target] = grad_update
    else:
        grad_input_t = grad_input.transpose(channel_dim, -1)
        intermidate_shape = grad_input_t.shape
        grad_input_2d = grad_input_t.reshape(-1, x.shape[channel_dim])
        grad_input_2d[arange_1d, masked_safe_target] = grad_update
        grad_input = grad_input_2d.view(intermidate_shape).transpose(channel_dim, -1)

    if grad_input.dim() > grad_output.dim() > 0:
        grad_output = grad_output.unsqueeze(channel_dim)

    if weight is not None:
        new_shape = [1 for _ in range(x.dim())]
        new_shape[channel_dim] = weight.shape[0]
        weight = weight.reshape(new_shape)
        # In order for fused computation to work, the following line is rewritten.
        # grad_output = grad_output * weight
        new_shape = list(x.shape)
        new_shape[channel_dim] = -1
        w = weight.expand(new_shape)
````

- **L401** EN: Assigns or updates `grad_input[arange_1d, masked_safe_target]`. | CN: 对 `grad_input[arange_1d, masked_safe_target]` 进行赋值或更新。
- **L402** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L403** EN: Assigns or updates `grad_input_t`. | CN: 对 `grad_input_t` 进行赋值或更新。
- **L404** EN: Assigns or updates `intermidate_shape`. | CN: 对 `intermidate_shape` 进行赋值或更新。
- **L405** EN: Assigns or updates `grad_input_2d`. | CN: 对 `grad_input_2d` 进行赋值或更新。
- **L406** EN: Assigns or updates `grad_input_2d[arange_1d, masked_safe_target]`. | CN: 对 `grad_input_2d[arange_1d, masked_safe_target]` 进行赋值或更新。
- **L407** EN: Assigns or updates `grad_input`. | CN: 对 `grad_input` 进行赋值或更新。
- **L408** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L409** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L410** EN: Assigns or updates `grad_output`. | CN: 对 `grad_output` 进行赋值或更新。
- **L411** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L412** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L413** EN: Assigns or updates `new_shape`. | CN: 对 `new_shape` 进行赋值或更新。
- **L414** EN: Assigns or updates `new_shape[channel_dim]`. | CN: 对 `new_shape[channel_dim]` 进行赋值或更新。
- **L415** EN: Assigns or updates `weight`. | CN: 对 `weight` 进行赋值或更新。
- **L416** EN: Keeps the inline comment or directive: In order for fused computation to work, the following line is rewritten. | CN: 保留这一行注释或指令：In order for fused computation to work, the following line is rewritten.
- **L417** EN: Keeps the inline comment or directive: grad_output = grad_output * weight | CN: 保留这一行注释或指令：grad_output = grad_output * weight
- **L418** EN: Assigns or updates `new_shape`. | CN: 对 `new_shape` 进行赋值或更新。
- **L419** EN: Assigns or updates `new_shape[channel_dim]`. | CN: 对 `new_shape[channel_dim]` 进行赋值或更新。
- **L420** EN: Assigns or updates `w`. | CN: 对 `w` 进行赋值或更新。

### Lines 421-440 / 第 421-440 行

````python
        w_target = torch.gather(w, channel_dim, target)
        grad_output = grad_output * w_target

    grad_output = torch.where(target != ignore_index, grad_output, 0)

    # NOTE: Instead of directly returning the grad_input as grad_output for log_softmax,
    # here we perform backward computation for log_softmax altogether to avoid the
    # otherwise extra all_gather communication.
    # return grad_input * grad_output
    return (grad_input + torch.exp(x)) * grad_output


def _nll_loss_backward_handler(
    op_call: torch._ops.OpOverload,
    args: tuple[object, ...],
    kwargs: dict[str, object],
) -> object:
    grad_output = cast(DTensor, args[0])
    x = cast(DTensor, args[1])
    target = args[2]
````

- **L421** EN: Assigns or updates `w_target`. | CN: 对 `w_target` 进行赋值或更新。
- **L422** EN: Assigns or updates `grad_output`. | CN: 对 `grad_output` 进行赋值或更新。
- **L423** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L424** EN: Assigns or updates `grad_output`. | CN: 对 `grad_output` 进行赋值或更新。
- **L425** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L426** EN: Keeps the inline comment or directive: NOTE: Instead of directly returning the grad_input as grad_output for log_softma | CN: 保留这一行注释或指令：NOTE: Instead of directly returning the grad_input as grad_output for log_softma
- **L427** EN: Keeps the inline comment or directive: here we perform backward computation for log_softmax altogether to avoid the | CN: 保留这一行注释或指令：here we perform backward computation for log_softmax altogether to avoid the
- **L428** EN: Keeps the inline comment or directive: otherwise extra all_gather communication. | CN: 保留这一行注释或指令：otherwise extra all_gather communication.
- **L429** EN: Keeps the inline comment or directive: return grad_input * grad_output | CN: 保留这一行注释或指令：return grad_input * grad_output
- **L430** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L431** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L432** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L433** EN: Defines function `_nll_loss_backward_handler`. | CN: 定义函数 `_nll_loss_backward_handler`。
- **L434** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L435** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L436** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L437** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L438** EN: Assigns or updates `grad_output`. | CN: 对 `grad_output` 进行赋值或更新。
- **L439** EN: Assigns or updates `x`. | CN: 对 `x` 进行赋值或更新。
- **L440** EN: Assigns or updates `target`. | CN: 对 `target` 进行赋值或更新。

### Lines 441-460 / 第 441-460 行

````python
    weight = args[3]
    reduction = cast(int, args[4])
    ignore_index = cast(int, args[5])
    total_weight = cast(Tensor, args[6])

    channel_dim = 1 if x.dim() >= 2 else 0
    spec = x._spec
    mesh_dim = _find_all_reduce_mesh_dim(spec.placements, channel_dim)

    # if target and weight are not DTensors, convert them to DTensors
    target_placements = _skip_dim(
        replicate_reduction_dims(spec.placements, [channel_dim]), channel_dim
    )
    all_replicate_placements = (Replicate(),) * spec.mesh.ndim
    target = _cast_to_dtensor(target, target_placements, spec.mesh)
    if weight is not None:
        weight = _cast_to_dtensor(weight, all_replicate_placements, spec.mesh)

    # tensor inputs to _propagate_tensor_meta need to be DTensors
    # pyrefly: ignore [bad-assignment]
````

- **L441** EN: Assigns or updates `weight`. | CN: 对 `weight` 进行赋值或更新。
- **L442** EN: Assigns or updates `reduction`. | CN: 对 `reduction` 进行赋值或更新。
- **L443** EN: Assigns or updates `ignore_index`. | CN: 对 `ignore_index` 进行赋值或更新。
- **L444** EN: Assigns or updates `total_weight`. | CN: 对 `total_weight` 进行赋值或更新。
- **L445** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L446** EN: Assigns or updates `channel_dim`. | CN: 对 `channel_dim` 进行赋值或更新。
- **L447** EN: Assigns or updates `spec`. | CN: 对 `spec` 进行赋值或更新。
- **L448** EN: Assigns or updates `mesh_dim`. | CN: 对 `mesh_dim` 进行赋值或更新。
- **L449** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L450** EN: Keeps the inline comment or directive: if target and weight are not DTensors, convert them to DTensors | CN: 保留这一行注释或指令：if target and weight are not DTensors, convert them to DTensors
- **L451** EN: Assigns or updates `target_placements`. | CN: 对 `target_placements` 进行赋值或更新。
- **L452** EN: Calls `replicate_reduction_dims` as part of the current workflow. | CN: 在当前流程中调用 `replicate_reduction_dims`。
- **L453** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L454** EN: Assigns or updates `all_replicate_placements`. | CN: 对 `all_replicate_placements` 进行赋值或更新。
- **L455** EN: Assigns or updates `target`. | CN: 对 `target` 进行赋值或更新。
- **L456** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L457** EN: Assigns or updates `weight`. | CN: 对 `weight` 进行赋值或更新。
- **L458** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L459** EN: Keeps the inline comment or directive: tensor inputs to _propagate_tensor_meta need to be DTensors | CN: 保留这一行注释或指令：tensor inputs to _propagate_tensor_meta need to be DTensors
- **L460** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-assignment] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-assignment]

### Lines 461-480 / 第 461-480 行

````python
    args = list(args)
    # pyrefly: ignore [unsupported-operation]
    args[2], args[3] = target, weight
    # pyrefly: ignore [unsupported-operation]
    args[6] = _cast_to_dtensor(total_weight, all_replicate_placements, spec.mesh)
    output_tensor_meta = _propagate_tensor_meta(op_call, tuple(args), kwargs)

    result = _nll_loss_and_log_softmax_backward(
        grad_output._local_tensor,
        x._local_tensor,
        target._local_tensor,
        weight._local_tensor if weight is not None else None,
        reduction,
        ignore_index,
        total_weight,
        x.shape,
        channel_dim,
        spec.mesh,
        mesh_dim,
    )
````

- **L461** EN: Assigns or updates `args`. | CN: 对 `args` 进行赋值或更新。
- **L462** EN: Keeps the inline comment or directive: pyrefly: ignore [unsupported-operation] | CN: 保留这一行注释或指令：pyrefly: ignore [unsupported-operation]
- **L463** EN: Assigns or updates `args[2], args[3]`. | CN: 对 `args[2], args[3]` 进行赋值或更新。
- **L464** EN: Keeps the inline comment or directive: pyrefly: ignore [unsupported-operation] | CN: 保留这一行注释或指令：pyrefly: ignore [unsupported-operation]
- **L465** EN: Assigns or updates `args[6]`. | CN: 对 `args[6]` 进行赋值或更新。
- **L466** EN: Assigns or updates `output_tensor_meta`. | CN: 对 `output_tensor_meta` 进行赋值或更新。
- **L467** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L468** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L469** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L470** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L471** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L472** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L473** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L474** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L475** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L476** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L477** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L478** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L479** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L480** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 481-500 / 第 481-500 行

````python
    # the output sharding is the same as input sharding: Shard(channel_dim) on mesh_dim
    out_spec = DTensorSpec(
        spec.mesh,
        spec.placements,
        tensor_meta=output_tensor_meta,
    )

    # pyrefly: ignore [bad-argument-type]
    return DTensor(
        # pyrefly: ignore [bad-argument-count]
        result,
        out_spec,
        # pyrefly: ignore [unexpected-keyword]
        requires_grad=result.requires_grad,
    )


customized_loss_ops = {
    aten._log_softmax.default: _log_softmax_handler,
    aten._log_softmax_backward_data.default: _log_softmax_backward_handler,
````

- **L481** EN: Keeps the inline comment or directive: the output sharding is the same as input sharding: Shard(channel_dim) on mesh_di | CN: 保留这一行注释或指令：the output sharding is the same as input sharding: Shard(channel_dim) on mesh_di
- **L482** EN: Assigns or updates `out_spec`. | CN: 对 `out_spec` 进行赋值或更新。
- **L483** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L484** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L485** EN: Assigns or updates `tensor_meta`. | CN: 对 `tensor_meta` 进行赋值或更新。
- **L486** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L487** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L488** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-argument-type] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-argument-type]
- **L489** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L490** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-argument-count] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-argument-count]
- **L491** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L492** EN: Continues the implementation inside function `_nll_loss_backward_handler`. | CN: 继续说明函数 `_nll_loss_backward_handler` 内部的实现。
- **L493** EN: Keeps the inline comment or directive: pyrefly: ignore [unexpected-keyword] | CN: 保留这一行注释或指令：pyrefly: ignore [unexpected-keyword]
- **L494** EN: Assigns or updates `requires_grad`. | CN: 对 `requires_grad` 进行赋值或更新。
- **L495** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L496** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L497** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L498** EN: Assigns or updates `customized_loss_ops`. | CN: 对 `customized_loss_ops` 进行赋值或更新。
- **L499** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L500** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。

### Lines 501-514 / 第 501-514 行

````python
    aten.nll_loss_forward.default: _nll_loss_forward_handler,
    aten.nll_loss2d_forward.default: _nll_loss_forward_handler,
    aten.nll_loss_backward.default: _nll_loss_backward_handler,
    aten.nll_loss2d_backward.default: _nll_loss_backward_handler,
}


def _enable_custom_loss_ops():
    DTensor._op_dispatcher._custom_op_handlers.update(customized_loss_ops)


def _disable_custom_loss_ops():
    for custom_op in customized_loss_ops:
        DTensor._op_dispatcher._custom_op_handlers.pop(custom_op)
````

- **L501** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L502** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L503** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L504** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L505** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L506** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L507** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L508** EN: Defines function `_enable_custom_loss_ops`. | CN: 定义函数 `_enable_custom_loss_ops`。
- **L509** EN: Calls `DTensor._op_dispatcher._custom_op_handlers.update` as part of the current workflow. | CN: 在当前流程中调用 `DTensor._op_dispatcher._custom_op_handlers.update`。
- **L510** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L511** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L512** EN: Defines function `_disable_custom_loss_ops`. | CN: 定义函数 `_disable_custom_loss_ops`。
- **L513** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L514** EN: Calls `DTensor._op_dispatcher._custom_op_handlers.pop` as part of the current workflow. | CN: 在当前流程中调用 `DTensor._op_dispatcher._custom_op_handlers.pop`。

## Key Concepts / 关键概念

- **EN**: distributed tensor layouts, placements, and operators  
  **CN**: 分布式张量布局、放置与算子逻辑
- **EN**: device mesh  
  **CN**: 设备网格
- **EN**: sharding  
  **CN**: 分片
- **EN**: collective communication  
  **CN**: 集合通信
- **EN**: all-reduce  
  **CN**: all-reduce 聚合
- **EN**: all-gather  
  **CN**: all-gather 聚合
- **EN**: distributed tensors  
  **CN**: 分布式张量
- **EN**: placements  
  **CN**: 放置规则

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed._functional_collectives`, `torch.distributed.device_mesh`, `torch.distributed.distributed_c10d`, `torch.distributed.tensor`, `torch.distributed.tensor._dtensor_spec`, `torch.distributed.tensor._ops._embedding_ops`, `torch.distributed.tensor._ops._math_ops`, `torch.distributed.tensor._ops.utils`, `torch.distributed.tensor.placement_types`
- **PyTorch / PyTorch**: `torch`, `torch._prims_common`
- **Python Stdlib / Python 标准库**: `contextlib`, `typing`
- **Third-party / 第三方**: None detected / 未检测到

