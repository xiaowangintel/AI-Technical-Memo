# ddp.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/tensor/parallel/ddp.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed tensor layouts, placements, and operators. Its main entry points include _get_submodule_n_params, _update_module_param.
- **用途 (CN)**: 该模块聚焦于分布式张量布局、放置与算子逻辑，其主要入口包括 _get_submodule_n_params, _update_module_param。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs
from typing import Any

import torch.nn as nn
from torch.distributed.tensor.parallel._data_parallel_utils import (
    _flatten_tensor,
    _unflatten_tensor,
)


__all__ = []  # type: ignore[var-annotated]


def _get_submodule_n_params(module: nn.Module, path: str):
    """
    Get submodule and the direct path of parameter from the module
    """
    if "." in path:
        path_list = path.split(".")
        parent_module_path = ".".join(path_list[:-1])
````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Imports selected names from `typing`. | CN: 从 `typing` 导入指定名称。
- **L3** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L4** EN: Imports module dependencies: `torch.nn as nn`. | CN: 导入模块依赖：`torch.nn as nn`。
- **L5** EN: Imports selected names from `torch.distributed.tensor.parallel._data_parallel_utils`. | CN: 从 `torch.distributed.tensor.parallel._data_parallel_utils` 导入指定名称。
- **L6** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L7** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L8** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L9** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L10** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L11** EN: Assigns or updates `__all__`. | CN: 对 `__all__` 进行赋值或更新。
- **L12** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L13** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L14** EN: Defines function `_get_submodule_n_params`. | CN: 定义函数 `_get_submodule_n_params`。
- **L15** EN: Starts the docstring for the function _get_submodule_n_params. | CN: 开始定义 function _get_submodule_n_params 的文档字符串。
- **L16** EN: Continues the docstring text for the function _get_submodule_n_params. | CN: 继续补充 function _get_submodule_n_params 的文档字符串内容。
- **L17** EN: Closes the docstring for the function _get_submodule_n_params. | CN: 结束 function _get_submodule_n_params 的文档字符串。
- **L18** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L19** EN: Assigns or updates `path_list`. | CN: 对 `path_list` 进行赋值或更新。
- **L20** EN: Assigns or updates `parent_module_path`. | CN: 对 `parent_module_path` 进行赋值或更新。

### Lines 21-40 / 第 21-40 行

````python
        module = module.get_submodule(parent_module_path)
        path = path_list[-1]
    return module, path


def _update_module_param(param_list: list[tuple[nn.Module, str, nn.Parameter]]):
    """
    Update parameters within the module
    """
    for item in param_list:
        parent_module, module_path, t = item
        if not hasattr(parent_module, module_path):
            raise AssertionError
        delattr(parent_module, module_path)
        setattr(parent_module, module_path, t)


def _reconstruct_dtensor(module: nn.Module, _input: Any):
    """
    Reconstruct DTensor parameters from local tensors
````

- **L21** EN: Assigns or updates `module`. | CN: 对 `module` 进行赋值或更新。
- **L22** EN: Assigns or updates `path`. | CN: 对 `path` 进行赋值或更新。
- **L23** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L24** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L25** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L26** EN: Defines function `_update_module_param`. | CN: 定义函数 `_update_module_param`。
- **L27** EN: Starts the docstring for the function _update_module_param. | CN: 开始定义 function _update_module_param 的文档字符串。
- **L28** EN: Continues the docstring text for the function _update_module_param. | CN: 继续补充 function _update_module_param 的文档字符串内容。
- **L29** EN: Closes the docstring for the function _update_module_param. | CN: 结束 function _update_module_param 的文档字符串。
- **L30** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L31** EN: Assigns or updates `parent_module, module_path, t`. | CN: 对 `parent_module, module_path, t` 进行赋值或更新。
- **L32** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L33** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L34** EN: Calls `delattr` as part of the current workflow. | CN: 在当前流程中调用 `delattr`。
- **L35** EN: Calls `setattr` as part of the current workflow. | CN: 在当前流程中调用 `setattr`。
- **L36** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L37** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L38** EN: Defines function `_reconstruct_dtensor`. | CN: 定义函数 `_reconstruct_dtensor`。
- **L39** EN: Starts the docstring for the function _reconstruct_dtensor. | CN: 开始定义 function _reconstruct_dtensor 的文档字符串。
- **L40** EN: Continues the docstring text for the function _reconstruct_dtensor. | CN: 继续补充 function _reconstruct_dtensor 的文档字符串内容。

### Lines 41-60 / 第 41-60 行

````python
    """
    param_list = []
    # TODO: To add perf optimizations to this iterations
    for name, t in module.named_parameters():
        if hasattr(t, "_st_info"):
            dtensor = _unflatten_tensor(t, t._st_info)
            param_list.append((*_get_submodule_n_params(module, name), dtensor))
    _update_module_param(param_list)  # type: ignore[arg-type]


def _localize_dtensor(
    module: nn.Module, *_: Any, ignored_params: set[nn.Parameter] | None = None
):
    """
    Convert DTensor parameters to local tensors
    """
    if ignored_params is None:
        ignored_params = set()
    param_list = []
    for name, param in module.named_parameters():
````

- **L41** EN: Closes the docstring for the function _reconstruct_dtensor. | CN: 结束 function _reconstruct_dtensor 的文档字符串。
- **L42** EN: Assigns or updates `param_list`. | CN: 对 `param_list` 进行赋值或更新。
- **L43** EN: Keeps the inline comment or directive: TODO: To add perf optimizations to this iterations | CN: 保留这一行注释或指令：TODO: To add perf optimizations to this iterations
- **L44** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L45** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L46** EN: Assigns or updates `dtensor`. | CN: 对 `dtensor` 进行赋值或更新。
- **L47** EN: Calls `param_list.append` as part of the current workflow. | CN: 在当前流程中调用 `param_list.append`。
- **L48** EN: Calls `_update_module_param` as part of the current workflow. | CN: 在当前流程中调用 `_update_module_param`。
- **L49** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L50** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L51** EN: Defines function `_localize_dtensor`. | CN: 定义函数 `_localize_dtensor`。
- **L52** EN: Assigns or updates `module`. | CN: 对 `module` 进行赋值或更新。
- **L53** EN: Continues the implementation inside function `_localize_dtensor`. | CN: 继续说明函数 `_localize_dtensor` 内部的实现。
- **L54** EN: Starts the docstring for the function _localize_dtensor. | CN: 开始定义 function _localize_dtensor 的文档字符串。
- **L55** EN: Continues the docstring text for the function _localize_dtensor. | CN: 继续补充 function _localize_dtensor 的文档字符串内容。
- **L56** EN: Closes the docstring for the function _localize_dtensor. | CN: 结束 function _localize_dtensor 的文档字符串。
- **L57** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L58** EN: Assigns or updates `ignored_params`. | CN: 对 `ignored_params` 进行赋值或更新。
- **L59** EN: Assigns or updates `param_list`. | CN: 对 `param_list` 进行赋值或更新。
- **L60** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。

### Lines 61-80 / 第 61-80 行

````python
        if param in ignored_params:
            continue
        t, sharding_info = _flatten_tensor(param)
        if sharding_info is not None:
            t = nn.Parameter(t)
            t._st_info = sharding_info  # type: ignore[attr-defined]
            param_list.append((*_get_submodule_n_params(module, name), t))
    _update_module_param(param_list)  # type: ignore[arg-type]


def _pre_dp_module_transform(module: nn.Module):
    """
    Enable the composability between Tensor Parallelism (TP) and Data
    Parallelism(DP) in PyTorch when using DDP. We need to convert Parameters which
    are DTensors to local tensors before wrapping with data parallelism API.
    We then register two hooks, one for converting local tensors back to DTensor
    preforward and one to convert DTensors back to tensors after Forward. By
    integrating this way, we avoid any special handling of DTensor parameters by DDP
    and get DTensor's gradients propagated back to DP, e.g. gradient buckets of DDP.

````

- **L61** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L62** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L63** EN: Assigns or updates `t, sharding_info`. | CN: 对 `t, sharding_info` 进行赋值或更新。
- **L64** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L65** EN: Assigns or updates `t`. | CN: 对 `t` 进行赋值或更新。
- **L66** EN: Assigns or updates `t._st_info`. | CN: 对 `t._st_info` 进行赋值或更新。
- **L67** EN: Calls `param_list.append` as part of the current workflow. | CN: 在当前流程中调用 `param_list.append`。
- **L68** EN: Calls `_update_module_param` as part of the current workflow. | CN: 在当前流程中调用 `_update_module_param`。
- **L69** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L70** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L71** EN: Defines function `_pre_dp_module_transform`. | CN: 定义函数 `_pre_dp_module_transform`。
- **L72** EN: Starts the docstring for the function _pre_dp_module_transform. | CN: 开始定义 function _pre_dp_module_transform 的文档字符串。
- **L73** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L74** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L75** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L76** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L77** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L78** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L79** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L80** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。

### Lines 81-100 / 第 81-100 行

````python
    For now, this API only works with ``DistributedDataParallel``. It will later support
    other DP methods such as FSDP.

    Args:
        module (:class:`nn.Module`):
            Module which has been applied TP on.

    Example::
        >>> # xdoctest: +SKIP("distributed")
        >>> from torch.distributed.tensor.parallel import parallelize_module, PairwiseParallel
        >>> from torch.nn.parallel import DistributedDataParallel as DDP
        >>> from torch.distributed.tensor.parallel.ddp import pre_dp_module_transform
        >>>
        >>> # Define the module.
        >>> m = module(...)
        >>> parallelize_module(m, PairwiseParallel())
        >>> m = pre_dp_module_transform(m)
        >>> m = DDP(m)
        >>>
    """
````

- **L81** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L82** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L83** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L84** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L85** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L86** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L87** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L88** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L89** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L90** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L91** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L92** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L93** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L94** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L95** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L96** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L97** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L98** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L99** EN: Continues the docstring text for the function _pre_dp_module_transform. | CN: 继续补充 function _pre_dp_module_transform 的文档字符串内容。
- **L100** EN: Closes the docstring for the function _pre_dp_module_transform. | CN: 结束 function _pre_dp_module_transform 的文档字符串。

### Lines 101-105 / 第 101-105 行

````python

    _localize_dtensor(module, None, None)
    # TODO: To add test cases and ensure that it works for nested modules
    module.register_forward_pre_hook(_reconstruct_dtensor)
    module.register_forward_hook(_localize_dtensor)
````

- **L101** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L102** EN: Calls `_localize_dtensor` as part of the current workflow. | CN: 在当前流程中调用 `_localize_dtensor`。
- **L103** EN: Keeps the inline comment or directive: TODO: To add test cases and ensure that it works for nested modules | CN: 保留这一行注释或指令：TODO: To add test cases and ensure that it works for nested modules
- **L104** EN: Calls `module.register_forward_pre_hook` as part of the current workflow. | CN: 在当前流程中调用 `module.register_forward_pre_hook`。
- **L105** EN: Calls `module.register_forward_hook` as part of the current workflow. | CN: 在当前流程中调用 `module.register_forward_hook`。

## Key Concepts / 关键概念

- **EN**: distributed tensor layouts, placements, and operators  
  **CN**: 分布式张量布局、放置与算子逻辑
- **EN**: sharding  
  **CN**: 分片
- **EN**: distributed tensors  
  **CN**: 分布式张量
- **EN**: communication hooks  
  **CN**: 通信钩子
- **EN**: Core callables: _get_submodule_n_params, _update_module_param, _reconstruct_dtensor, _localize_dtensor, _pre_dp_module_transform  
  **CN**: 核心可调用对象：_get_submodule_n_params, _update_module_param, _reconstruct_dtensor, _localize_dtensor, _pre_dp_module_transform

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed.tensor.parallel._data_parallel_utils`
- **PyTorch / PyTorch**: `torch.nn`
- **Python Stdlib / Python 标准库**: `typing`
- **Third-party / 第三方**: None detected / 未检测到

