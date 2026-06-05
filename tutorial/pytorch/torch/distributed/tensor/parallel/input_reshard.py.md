# input_reshard.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/tensor/parallel/input_reshard.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed tensor layouts, placements, and operators. Its main entry points include input_reshard, _pack_hook_tp.
- **用途 (CN)**: 该模块聚焦于分布式张量布局、放置与算子逻辑，其主要入口包括 input_reshard, _pack_hook_tp。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# Copyright (c) Meta Platforms, Inc. and affiliates
from functools import partial
from typing import Any

import torch
from torch.distributed.tensor import DeviceMesh, DTensor, Replicate, Shard
from torch.distributed.tensor.placement_types import _is_shard_like


__all__ = [
    "input_reshard",
]


def input_reshard(
    module: torch.nn.Module,
    tp_device_mesh: DeviceMesh,
    input_reshard_dim: int | None = None,
) -> torch.nn.Module:
    """
````

- **L1** EN: Keeps the inline comment or directive: Copyright (c) Meta Platforms, Inc. and affiliates | CN: 保留这一行注释或指令：Copyright (c) Meta Platforms, Inc. and affiliates
- **L2** EN: Imports selected names from `functools`. | CN: 从 `functools` 导入指定名称。
- **L3** EN: Imports selected names from `typing`. | CN: 从 `typing` 导入指定名称。
- **L4** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L5** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L6** EN: Imports selected names from `torch.distributed.tensor`. | CN: 从 `torch.distributed.tensor` 导入指定名称。
- **L7** EN: Imports selected names from `torch.distributed.tensor.placement_types`. | CN: 从 `torch.distributed.tensor.placement_types` 导入指定名称。
- **L8** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L9** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L10** EN: Assigns or updates `__all__`. | CN: 对 `__all__` 进行赋值或更新。
- **L11** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L12** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L13** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L14** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L15** EN: Defines function `input_reshard`. | CN: 定义函数 `input_reshard`。
- **L16** EN: Continues the implementation inside function `input_reshard`. | CN: 继续说明函数 `input_reshard` 内部的实现。
- **L17** EN: Continues the implementation inside function `input_reshard`. | CN: 继续说明函数 `input_reshard` 内部的实现。
- **L18** EN: Assigns or updates `input_reshard_dim`. | CN: 对 `input_reshard_dim` 进行赋值或更新。
- **L19** EN: Continues the implementation inside function `input_reshard`. | CN: 继续说明函数 `input_reshard` 内部的实现。
- **L20** EN: Starts the docstring for the function input_reshard. | CN: 开始定义 function input_reshard 的文档字符串。

### Lines 21-40 / 第 21-40 行

````python
    Register hooks to an nn.Module for input resharding, enabling sharding and restoration during backward computation.

    Register hooks to an nn.Module with input resharding so that we can shard
    per the given `tp_device_mesh` and `input_reshard_dim` and restore the
    input back when recomputing the activations in the backward. The reason
    why we can do this is that for Tensor Parallel(TP), the input are same
    across all TP ranks.

    Args:
        module (:class:`nn.Module`):
            Module to be registered with input resharding.
        tp_device_mesh (:class:`DeviceMesh`):
            Object which describes the mesh topology
            of devices for Tensor Parallel.
        input_reshard_dim (Optional[int]):
            The dimension of where we perform the sharding
            of input. If set None, there is no sharding of input.
            Default: None

    Return:
````

- **L21** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L22** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L23** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L24** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L25** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L26** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L27** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L28** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L29** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L30** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L31** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L32** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L33** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L34** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L35** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L36** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L37** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L38** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L39** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L40** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。

### Lines 41-60 / 第 41-60 行

````python
        A :class:`nn.Module` object registered with TP input resharding.
    """
    if input_reshard_dim is None:
        return module

    cx: torch.autograd.graph.saved_tensors_hooks | None = None

    def input_reshard_forward_pre_hook(_: torch.nn.Module, _i: tuple[Any, ...]) -> None:
        saved_tensor_hooks = torch.autograd.graph.saved_tensors_hooks(
            partial(_pack_hook_tp, tp_device_mesh, input_reshard_dim),
            partial(_unpack_hook_tp, tp_device_mesh, input_reshard_dim),
        )
        saved_tensor_hooks.__enter__()
        nonlocal cx
        cx = saved_tensor_hooks  # type: ignore[name-defined]

    def input_reshard_backward_hook(
        _: torch.nn.Module, _i: tuple[Any, ...], _o: Any
    ) -> Any:
        nonlocal cx
````

- **L41** EN: Continues the docstring text for the function input_reshard. | CN: 继续补充 function input_reshard 的文档字符串内容。
- **L42** EN: Closes the docstring for the function input_reshard. | CN: 结束 function input_reshard 的文档字符串。
- **L43** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L44** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L45** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L46** EN: Assigns or updates `cx`. | CN: 对 `cx` 进行赋值或更新。
- **L47** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L48** EN: Defines function `input_reshard_forward_pre_hook`. | CN: 定义函数 `input_reshard_forward_pre_hook`。
- **L49** EN: Assigns or updates `saved_tensor_hooks`. | CN: 对 `saved_tensor_hooks` 进行赋值或更新。
- **L50** EN: Calls `partial` as part of the current workflow. | CN: 在当前流程中调用 `partial`。
- **L51** EN: Calls `partial` as part of the current workflow. | CN: 在当前流程中调用 `partial`。
- **L52** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L53** EN: Calls `saved_tensor_hooks.__enter__` as part of the current workflow. | CN: 在当前流程中调用 `saved_tensor_hooks.__enter__`。
- **L54** EN: Declares enclosing-scope names for reassignment. | CN: 声明需要重新赋值的外层作用域名称。
- **L55** EN: Assigns or updates `cx`. | CN: 对 `cx` 进行赋值或更新。
- **L56** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L57** EN: Defines function `input_reshard_backward_hook`. | CN: 定义函数 `input_reshard_backward_hook`。
- **L58** EN: Continues the implementation inside function `input_reshard_backward_hook`. | CN: 继续说明函数 `input_reshard_backward_hook` 内部的实现。
- **L59** EN: Continues the implementation inside function `input_reshard_backward_hook`. | CN: 继续说明函数 `input_reshard_backward_hook` 内部的实现。
- **L60** EN: Declares enclosing-scope names for reassignment. | CN: 声明需要重新赋值的外层作用域名称。

### Lines 61-80 / 第 61-80 行

````python
        cx.__exit__()  # type: ignore[name-defined, union-attr]

    module.register_forward_pre_hook(input_reshard_forward_pre_hook)
    module.register_forward_hook(input_reshard_backward_hook)
    return module


def _pack_hook_tp(mesh: DeviceMesh, input_reshard_dim: int, x: torch.Tensor) -> Any:
    """Hook function called after FWD to shard input."""
    if isinstance(x, DTensor) and all(p.is_replicate() for p in x._spec.placements):
        return x.redistribute(device_mesh=mesh, placements=[Shard(input_reshard_dim)])
    elif (
        not isinstance(x, DTensor)
        and isinstance(x, torch.Tensor)
        and x.numel() >= mesh.size()
    ):
        return (
            DTensor.from_local(x, device_mesh=mesh)
            .redistribute(device_mesh=mesh, placements=[Shard(input_reshard_dim)])
            .to_local()
````

- **L61** EN: Calls `cx.__exit__` as part of the current workflow. | CN: 在当前流程中调用 `cx.__exit__`。
- **L62** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L63** EN: Calls `module.register_forward_pre_hook` as part of the current workflow. | CN: 在当前流程中调用 `module.register_forward_pre_hook`。
- **L64** EN: Calls `module.register_forward_hook` as part of the current workflow. | CN: 在当前流程中调用 `module.register_forward_hook`。
- **L65** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L66** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L67** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L68** EN: Defines function `_pack_hook_tp`. | CN: 定义函数 `_pack_hook_tp`。
- **L69** EN: Docstring line documenting the function _pack_hook_tp. | CN: 这是记录 function _pack_hook_tp 的文档字符串。
- **L70** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L71** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L72** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L73** EN: Continues the implementation inside function `_pack_hook_tp`. | CN: 继续说明函数 `_pack_hook_tp` 内部的实现。
- **L74** EN: Continues the implementation inside function `_pack_hook_tp`. | CN: 继续说明函数 `_pack_hook_tp` 内部的实现。
- **L75** EN: Continues the implementation inside function `_pack_hook_tp`. | CN: 继续说明函数 `_pack_hook_tp` 内部的实现。
- **L76** EN: Continues the implementation inside function `_pack_hook_tp`. | CN: 继续说明函数 `_pack_hook_tp` 内部的实现。
- **L77** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L78** EN: Calls `DTensor.from_local` as part of the current workflow. | CN: 在当前流程中调用 `DTensor.from_local`。
- **L79** EN: Continues the implementation inside function `_pack_hook_tp`. | CN: 继续说明函数 `_pack_hook_tp` 内部的实现。
- **L80** EN: Continues the implementation inside function `_pack_hook_tp`. | CN: 继续说明函数 `_pack_hook_tp` 内部的实现。

### Lines 81-100 / 第 81-100 行

````python
        )
    else:
        return x


def _unpack_hook_tp(mesh: DeviceMesh, input_reshard_dim: int, x: Any) -> torch.Tensor:
    """Hook function called before activation recomputing in BWD to restore input."""
    if (
        isinstance(x, DTensor)
        and len(x._spec.placements) == 1
        and _is_shard_like(x._spec.placements[0])
    ):
        return x.redistribute(device_mesh=mesh, placements=[Replicate()])
    elif (
        not isinstance(x, DTensor)
        and isinstance(x, torch.Tensor)
        and x.numel() >= mesh.size()
    ):
        return (
            DTensor.from_local(
````

- **L81** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L82** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L83** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L84** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L85** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L86** EN: Defines function `_unpack_hook_tp`. | CN: 定义函数 `_unpack_hook_tp`。
- **L87** EN: Docstring line documenting the function _unpack_hook_tp. | CN: 这是记录 function _unpack_hook_tp 的文档字符串。
- **L88** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L89** EN: Calls `isinstance` as part of the current workflow. | CN: 在当前流程中调用 `isinstance`。
- **L90** EN: Continues the implementation inside function `_unpack_hook_tp`. | CN: 继续说明函数 `_unpack_hook_tp` 内部的实现。
- **L91** EN: Continues the implementation inside function `_unpack_hook_tp`. | CN: 继续说明函数 `_unpack_hook_tp` 内部的实现。
- **L92** EN: Continues the implementation inside function `_unpack_hook_tp`. | CN: 继续说明函数 `_unpack_hook_tp` 内部的实现。
- **L93** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L94** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L95** EN: Continues the implementation inside function `_unpack_hook_tp`. | CN: 继续说明函数 `_unpack_hook_tp` 内部的实现。
- **L96** EN: Continues the implementation inside function `_unpack_hook_tp`. | CN: 继续说明函数 `_unpack_hook_tp` 内部的实现。
- **L97** EN: Continues the implementation inside function `_unpack_hook_tp`. | CN: 继续说明函数 `_unpack_hook_tp` 内部的实现。
- **L98** EN: Continues the implementation inside function `_unpack_hook_tp`. | CN: 继续说明函数 `_unpack_hook_tp` 内部的实现。
- **L99** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L100** EN: Calls `DTensor.from_local` as part of the current workflow. | CN: 在当前流程中调用 `DTensor.from_local`。

### Lines 101-107 / 第 101-107 行

````python
                x, device_mesh=mesh, placements=[Shard(input_reshard_dim)]
            )
            .redistribute(device_mesh=mesh, placements=[Replicate()])
            .to_local()
        )
    else:
        return x
````

- **L101** EN: Assigns or updates `x, device_mesh`. | CN: 对 `x, device_mesh` 进行赋值或更新。
- **L102** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L103** EN: Continues the implementation inside function `_unpack_hook_tp`. | CN: 继续说明函数 `_unpack_hook_tp` 内部的实现。
- **L104** EN: Continues the implementation inside function `_unpack_hook_tp`. | CN: 继续说明函数 `_unpack_hook_tp` 内部的实现。
- **L105** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L106** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L107** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

## Key Concepts / 关键概念

- **EN**: distributed tensor layouts, placements, and operators  
  **CN**: 分布式张量布局、放置与算子逻辑
- **EN**: device mesh  
  **CN**: 设备网格
- **EN**: sharding  
  **CN**: 分片
- **EN**: distributed tensors  
  **CN**: 分布式张量
- **EN**: placements  
  **CN**: 放置规则
- **EN**: distributed stores  
  **CN**: 分布式存储
- **EN**: communication hooks  
  **CN**: 通信钩子
- **EN**: distributed autograd  
  **CN**: 分布式自动求导

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed.tensor`, `torch.distributed.tensor.placement_types`
- **PyTorch / PyTorch**: `torch`
- **Python Stdlib / Python 标准库**: `functools`, `typing`
- **Third-party / 第三方**: None detected / 未检测到

