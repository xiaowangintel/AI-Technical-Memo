# api.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/tensor/parallel/api.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed tensor layouts, placements, and operators. Its main entry points include parallelize_module.
- **用途 (CN)**: 该模块聚焦于分布式张量布局、放置与算子逻辑，其主要入口包括 parallelize_module。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# Copyright (c) Meta Platforms, Inc. and affiliates
import warnings
from fnmatch import fnmatch

import torch
import torch.nn as nn
from torch.distributed.device_mesh import _mesh_resources, DeviceMesh
from torch.distributed.tensor.parallel.style import ParallelStyle


__all__ = ["parallelize_module"]


def parallelize_module(  # type: ignore[return]
    module: nn.Module,
    device_mesh: DeviceMesh | None = None,
    parallelize_plan: ParallelStyle | dict[str, ParallelStyle] | None = None,
    *,
    src_data_rank: int | None = 0,
) -> nn.Module:
````

- **L1** EN: Keeps the inline comment or directive: Copyright (c) Meta Platforms, Inc. and affiliates | CN: 保留这一行注释或指令：Copyright (c) Meta Platforms, Inc. and affiliates
- **L2** EN: Imports module dependencies: `warnings`. | CN: 导入模块依赖：`warnings`。
- **L3** EN: Imports selected names from `fnmatch`. | CN: 从 `fnmatch` 导入指定名称。
- **L4** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L5** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L6** EN: Imports module dependencies: `torch.nn as nn`. | CN: 导入模块依赖：`torch.nn as nn`。
- **L7** EN: Imports selected names from `torch.distributed.device_mesh`. | CN: 从 `torch.distributed.device_mesh` 导入指定名称。
- **L8** EN: Imports selected names from `torch.distributed.tensor.parallel.style`. | CN: 从 `torch.distributed.tensor.parallel.style` 导入指定名称。
- **L9** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L10** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L11** EN: Assigns or updates `__all__`. | CN: 对 `__all__` 进行赋值或更新。
- **L12** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L13** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L14** EN: Defines function `parallelize_module`. | CN: 定义函数 `parallelize_module`。
- **L15** EN: Continues the implementation inside function `parallelize_module`. | CN: 继续说明函数 `parallelize_module` 内部的实现。
- **L16** EN: Assigns or updates `device_mesh`. | CN: 对 `device_mesh` 进行赋值或更新。
- **L17** EN: Assigns or updates `parallelize_plan`. | CN: 对 `parallelize_plan` 进行赋值或更新。
- **L18** EN: Continues the implementation inside function `parallelize_module`. | CN: 继续说明函数 `parallelize_module` 内部的实现。
- **L19** EN: Assigns or updates `src_data_rank`. | CN: 对 `src_data_rank` 进行赋值或更新。
- **L20** EN: Continues the implementation inside function `parallelize_module`. | CN: 继续说明函数 `parallelize_module` 内部的实现。

### Lines 21-40 / 第 21-40 行

````python
    """
    Apply Tensor Parallelism in PyTorch by parallelizing modules or sub-modules based on a user-specified plan.

    We parallelize module or sub_modules based on a parallelize_plan. The parallelize_plan contains
    :class:`ParallelStyle`, which indicates how user wants the module or sub_module
    to be parallelized.

    User can also specify different parallel style per module fully qualified name (FQN).

    Note that ``parallelize_module`` only accepts a 1-D :class:`DeviceMesh`, if you have a 2-D or N-D :class:`DeviceMesh`,
    slice the DeviceMesh to a 1-D sub DeviceMesh first then pass to this API(i.e. ``device_mesh[\"tp\"]``)

    Args:
        module (:class:`nn.Module`):
            Module to be parallelized.
        device_mesh (:class:`DeviceMesh`, optional):
            Object which describes the mesh topology of devices for the DTensor.
            If not specified, the call must be under a DeviceMesh context.
        parallelize_plan (Union[:class:`ParallelStyle`, Dict[str, :class:`ParallelStyle`]], optional):
            The plan used to parallelize the module. It can be either a
````

- **L21** EN: Starts the docstring for the function parallelize_module. | CN: 开始定义 function parallelize_module 的文档字符串。
- **L22** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L23** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L24** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L25** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L26** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L27** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L28** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L29** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L30** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L31** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L32** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L33** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L34** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L35** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L36** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L37** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L38** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L39** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L40** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。

### Lines 41-60 / 第 41-60 行

````python
            :class:`ParallelStyle` object which contains how we prepare
            input/output for Tensor Parallelism or it can be a dict of module
            FQN and its corresponding :class:`ParallelStyle` object. If not
            specified, the call will do nothing at the moment.
    Keyword args:
        src_data_rank (int, optional): the rank of the source data for the logical/global tensor, it is used by
            :meth:`distribute_tensor` to scatter/broadcast the shards/replicas to other ranks. By default,
            we use ``group_rank=0`` on each DeviceMesh dimension as the source data to preserve the single-device
            semantic. If passing ``None`` explicitly, :meth:`parallelize_module` simply uses its local data instead
            of trying to preserve the single-device semantic via scatter/broadcast. Default: 0
    Return:
        A :class:`nn.Module` object parallelized.

    Example::
        >>> # xdoctest: +SKIP("distributed")
        >>> from torch.distributed.tensor.parallel import parallelize_module, ColwiseParallel
        >>> from torch.distributed.device_mesh import init_device_mesh
        >>>
        >>> # Define the module.
        >>> m = Model(...)
````

- **L41** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L42** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L43** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L44** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L45** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L46** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L47** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L48** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L49** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L50** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L51** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L52** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L53** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L54** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L55** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L56** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L57** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L58** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L59** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L60** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。

### Lines 61-80 / 第 61-80 行

````python
        >>> tp_mesh = init_device_mesh("cuda", (8,))
        >>> m = parallelize_module(m, tp_mesh, {"w1": ColwiseParallel(), "w2": RowwiseParallel()})
        >>>

    .. note:: For complex module architecture like Attention, MLP layers, we recommend composing
        different ParallelStyles together (i.e. ``ColwiseParallel`` and ``RowwiseParallel``) and pass
        as a parallelize_plan, to achieves the desired sharding computation.
    """
    torch._C._log_api_usage_once("torch.distributed.tensor.parallel.parallelize_module")

    device_mesh = device_mesh or _mesh_resources.get_current_mesh()

    if parallelize_plan is None:
        warnings.warn(
            "No parallelize_plan is provided and auto-parallel is not supported "
            "at the moment, so this parallelize_module call will do nothing.",
            stacklevel=2,
        )
        return module

````

- **L61** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L62** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L63** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L64** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L65** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L66** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L67** EN: Continues the docstring text for the function parallelize_module. | CN: 继续补充 function parallelize_module 的文档字符串内容。
- **L68** EN: Closes the docstring for the function parallelize_module. | CN: 结束 function parallelize_module 的文档字符串。
- **L69** EN: Calls `torch._C._log_api_usage_once` as part of the current workflow. | CN: 在当前流程中调用 `torch._C._log_api_usage_once`。
- **L70** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L71** EN: Assigns or updates `device_mesh`. | CN: 对 `device_mesh` 进行赋值或更新。
- **L72** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L73** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L74** EN: Calls `warnings.warn` as part of the current workflow. | CN: 在当前流程中调用 `warnings.warn`。
- **L75** EN: Continues the implementation inside function `parallelize_module`. | CN: 继续说明函数 `parallelize_module` 内部的实现。
- **L76** EN: Continues the implementation inside function `parallelize_module`. | CN: 继续说明函数 `parallelize_module` 内部的实现。
- **L77** EN: Assigns or updates `stacklevel`. | CN: 对 `stacklevel` 进行赋值或更新。
- **L78** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L79** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L80** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 81-100 / 第 81-100 行

````python
    # note: The RNG tracker will be initialized in distribute_tensor() call if it hasn't
    # been initialized.

    if isinstance(parallelize_plan, ParallelStyle):
        parallelize_plan.src_data_rank = src_data_rank
        return parallelize_plan._apply(module, device_mesh)
    elif isinstance(parallelize_plan, dict):
        for module_path, parallelize_style in parallelize_plan.items():
            if module_path == "":
                # shortcut: empty string means to apply the plan to the current module
                parallelize_module(module, device_mesh, parallelize_style)
                continue

            path_splits = module_path.split(".")
            # Instead of blindly popping tokens, first check the match,
            # we only consume/pop the token if we found a match.
            token = path_splits[0]

            matched_children = list(
                filter(
````

- **L81** EN: Keeps the inline comment or directive: note: The RNG tracker will be initialized in distribute_tensor() call if it hasn | CN: 保留这一行注释或指令：note: The RNG tracker will be initialized in distribute_tensor() call if it hasn
- **L82** EN: Keeps the inline comment or directive: been initialized. | CN: 保留这一行注释或指令：been initialized.
- **L83** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L84** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L85** EN: Assigns or updates `parallelize_plan.src_data_rank`. | CN: 对 `parallelize_plan.src_data_rank` 进行赋值或更新。
- **L86** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L87** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L88** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L89** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L90** EN: Keeps the inline comment or directive: shortcut: empty string means to apply the plan to the current module | CN: 保留这一行注释或指令：shortcut: empty string means to apply the plan to the current module
- **L91** EN: Calls `parallelize_module` as part of the current workflow. | CN: 在当前流程中调用 `parallelize_module`。
- **L92** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L93** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L94** EN: Assigns or updates `path_splits`. | CN: 对 `path_splits` 进行赋值或更新。
- **L95** EN: Keeps the inline comment or directive: Instead of blindly popping tokens, first check the match, | CN: 保留这一行注释或指令：Instead of blindly popping tokens, first check the match,
- **L96** EN: Keeps the inline comment or directive: we only consume/pop the token if we found a match. | CN: 保留这一行注释或指令：we only consume/pop the token if we found a match.
- **L97** EN: Assigns or updates `token`. | CN: 对 `token` 进行赋值或更新。
- **L98** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L99** EN: Assigns or updates `matched_children`. | CN: 对 `matched_children` 进行赋值或更新。
- **L100** EN: Calls `filter` as part of the current workflow. | CN: 在当前流程中调用 `filter`。

### Lines 101-120 / 第 101-120 行

````python
                    # `t[0]` is child name
                    lambda t: fnmatch(t[0], token),
                    module.named_children(),
                )
            )
            if not matched_children:
                # No match at this level. Log a warning and process next plan entry.
                warnings.warn(
                    f"Parallelize plan key '{module_path}' could not be resolved: "
                    f"no submodule matching token '{token}' in module {module}, "
                    f"skipping this plan entry.",
                    stacklevel=2,
                )
                continue

            # Now that we have a match, we can consume the token.
            path_splits.pop(0)
            # apply the plan to all matched submodules
            for _, submodule in matched_children:
                if path_splits:
````

- **L101** EN: Keeps the inline comment or directive: `t[0]` is child name | CN: 保留这一行注释或指令：`t[0]` is child name
- **L102** EN: Continues the implementation inside function `parallelize_module`. | CN: 继续说明函数 `parallelize_module` 内部的实现。
- **L103** EN: Calls `module.named_children` as part of the current workflow. | CN: 在当前流程中调用 `module.named_children`。
- **L104** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L105** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L106** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L107** EN: Keeps the inline comment or directive: No match at this level. Log a warning and process next plan entry. | CN: 保留这一行注释或指令：No match at this level. Log a warning and process next plan entry.
- **L108** EN: Calls `warnings.warn` as part of the current workflow. | CN: 在当前流程中调用 `warnings.warn`。
- **L109** EN: Continues the implementation inside function `parallelize_module`. | CN: 继续说明函数 `parallelize_module` 内部的实现。
- **L110** EN: Continues the implementation inside function `parallelize_module`. | CN: 继续说明函数 `parallelize_module` 内部的实现。
- **L111** EN: Continues the implementation inside function `parallelize_module`. | CN: 继续说明函数 `parallelize_module` 内部的实现。
- **L112** EN: Assigns or updates `stacklevel`. | CN: 对 `stacklevel` 进行赋值或更新。
- **L113** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L114** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L115** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L116** EN: Keeps the inline comment or directive: Now that we have a match, we can consume the token. | CN: 保留这一行注释或指令：Now that we have a match, we can consume the token.
- **L117** EN: Calls `path_splits.pop` as part of the current workflow. | CN: 在当前流程中调用 `path_splits.pop`。
- **L118** EN: Keeps the inline comment or directive: apply the plan to all matched submodules | CN: 保留这一行注释或指令：apply the plan to all matched submodules
- **L119** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L120** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 121-140 / 第 121-140 行

````python
                    # we haven't reached the leaf, apply in dict style
                    leaf_path = ".".join(path_splits)  # rest of the path after `token`
                    parallelize_module(
                        submodule,
                        device_mesh,
                        {leaf_path: parallelize_style},
                        src_data_rank=src_data_rank,
                    )
                else:
                    # otherwise, directly apply style to this submodule
                    parallelize_module(
                        submodule,
                        device_mesh,
                        parallelize_style,
                        src_data_rank=src_data_rank,
                    )
        return module
    else:
        raise TypeError(  # pyre-ignore[7]
            "Expect Union[ParallelStyle, Dict[str, ParallelStyle]] for"
````

- **L121** EN: Keeps the inline comment or directive: we haven't reached the leaf, apply in dict style | CN: 保留这一行注释或指令：we haven't reached the leaf, apply in dict style
- **L122** EN: Assigns or updates `leaf_path`. | CN: 对 `leaf_path` 进行赋值或更新。
- **L123** EN: Calls `parallelize_module` as part of the current workflow. | CN: 在当前流程中调用 `parallelize_module`。
- **L124** EN: Continues the implementation inside function `parallelize_module`. | CN: 继续说明函数 `parallelize_module` 内部的实现。
- **L125** EN: Continues the implementation inside function `parallelize_module`. | CN: 继续说明函数 `parallelize_module` 内部的实现。
- **L126** EN: Continues the implementation inside function `parallelize_module`. | CN: 继续说明函数 `parallelize_module` 内部的实现。
- **L127** EN: Assigns or updates `src_data_rank`. | CN: 对 `src_data_rank` 进行赋值或更新。
- **L128** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L129** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L130** EN: Keeps the inline comment or directive: otherwise, directly apply style to this submodule | CN: 保留这一行注释或指令：otherwise, directly apply style to this submodule
- **L131** EN: Calls `parallelize_module` as part of the current workflow. | CN: 在当前流程中调用 `parallelize_module`。
- **L132** EN: Continues the implementation inside function `parallelize_module`. | CN: 继续说明函数 `parallelize_module` 内部的实现。
- **L133** EN: Continues the implementation inside function `parallelize_module`. | CN: 继续说明函数 `parallelize_module` 内部的实现。
- **L134** EN: Continues the implementation inside function `parallelize_module`. | CN: 继续说明函数 `parallelize_module` 内部的实现。
- **L135** EN: Assigns or updates `src_data_rank`. | CN: 对 `src_data_rank` 进行赋值或更新。
- **L136** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L137** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L138** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L139** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L140** EN: Continues the implementation inside function `parallelize_module`. | CN: 继续说明函数 `parallelize_module` 内部的实现。

### Lines 141-142 / 第 141-142 行

````python
            f" parallelize_plan, {type(parallelize_plan)} found!"
        )
````

- **L141** EN: Continues the implementation inside function `parallelize_module`. | CN: 继续说明函数 `parallelize_module` 内部的实现。
- **L142** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

## Key Concepts / 关键概念

- **EN**: distributed tensor layouts, placements, and operators  
  **CN**: 分布式张量布局、放置与算子逻辑
- **EN**: device mesh  
  **CN**: 设备网格
- **EN**: sharding  
  **CN**: 分片
- **EN**: broadcast  
  **CN**: 广播
- **EN**: distributed tensors  
  **CN**: 分布式张量
- **EN**: Core callables: parallelize_module  
  **CN**: 核心可调用对象：parallelize_module

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed.device_mesh`, `torch.distributed.tensor.parallel.style`
- **PyTorch / PyTorch**: `torch`, `torch.nn`
- **Python Stdlib / Python 标准库**: `fnmatch`, `warnings`
- **Third-party / 第三方**: None detected / 未检测到

