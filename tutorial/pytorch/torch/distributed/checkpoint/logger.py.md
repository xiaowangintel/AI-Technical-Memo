# logger.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/checkpoint/logger.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed checkpointing, planners, and storage helpers. Its main entry points include _msg_dict_from_dcp_method_args, _get_msg_dict.
- **用途 (CN)**: 该模块聚焦于分布式检查点、规划器与存储辅助逻辑，其主要入口包括 _msg_dict_from_dcp_method_args, _get_msg_dict。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs
import functools
import logging
import time
from collections.abc import Callable
from typing import Any, TypeVar
from typing_extensions import ParamSpec
from uuid import uuid4

import torch.distributed.c10d_logger as c10d_logger
from torch.distributed.checkpoint.logging_handlers import DCP_LOGGER_NAME


logger = logging.getLogger()


__all__: list[str] = []

# pyrefly: ignore [unknown-name]
global _dcp_logger
````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Imports module dependencies: `functools`. | CN: 导入模块依赖：`functools`。
- **L3** EN: Imports module dependencies: `logging`. | CN: 导入模块依赖：`logging`。
- **L4** EN: Imports module dependencies: `time`. | CN: 导入模块依赖：`time`。
- **L5** EN: Imports selected names from `collections.abc`. | CN: 从 `collections.abc` 导入指定名称。
- **L6** EN: Imports selected names from `typing`. | CN: 从 `typing` 导入指定名称。
- **L7** EN: Imports selected names from `typing_extensions`. | CN: 从 `typing_extensions` 导入指定名称。
- **L8** EN: Imports selected names from `uuid`. | CN: 从 `uuid` 导入指定名称。
- **L9** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L10** EN: Imports module dependencies: `torch.distributed.c10d_logger as c10d_logger`. | CN: 导入模块依赖：`torch.distributed.c10d_logger as c10d_logger`。
- **L11** EN: Imports selected names from `torch.distributed.checkpoint.logging_handlers`. | CN: 从 `torch.distributed.checkpoint.logging_handlers` 导入指定名称。
- **L12** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L13** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L14** EN: Assigns or updates `logger`. | CN: 对 `logger` 进行赋值或更新。
- **L15** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L16** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L17** EN: Assigns or updates `__all__`. | CN: 对 `__all__` 进行赋值或更新。
- **L18** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L19** EN: Keeps the inline comment or directive: pyrefly: ignore [unknown-name] | CN: 保留这一行注释或指令：pyrefly: ignore [unknown-name]
- **L20** EN: Declares module-level names for reassignment. | CN: 声明需要重新赋值的模块级名称。

### Lines 21-40 / 第 21-40 行

````python
_dcp_logger = c10d_logger._get_or_create_logger(DCP_LOGGER_NAME)

_T = TypeVar("_T")
_P = ParamSpec("_P")


def _msg_dict_from_dcp_method_args(*args, **kwargs) -> dict[str, Any]:
    """
    Extracts log data from dcp method args
    """
    msg_dict = {}

    # checkpoint ID can be passed in through the serializer or through the checkpoint id directly
    storage_writer = kwargs.get("storage_writer")
    storage_reader = kwargs.get("storage_reader")
    planner = kwargs.get("planner")

    checkpoint_id = kwargs.get("checkpoint_id")
    if not checkpoint_id and (serializer := storage_writer or storage_reader):
        checkpoint_id = getattr(serializer, "checkpoint_id", None)
````

- **L21** EN: Assigns or updates `_dcp_logger`. | CN: 对 `_dcp_logger` 进行赋值或更新。
- **L22** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L23** EN: Assigns or updates `_T`. | CN: 对 `_T` 进行赋值或更新。
- **L24** EN: Assigns or updates `_P`. | CN: 对 `_P` 进行赋值或更新。
- **L25** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L26** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L27** EN: Defines function `_msg_dict_from_dcp_method_args`. | CN: 定义函数 `_msg_dict_from_dcp_method_args`。
- **L28** EN: Starts the docstring for the function _msg_dict_from_dcp_method_args. | CN: 开始定义 function _msg_dict_from_dcp_method_args 的文档字符串。
- **L29** EN: Continues the docstring text for the function _msg_dict_from_dcp_method_args. | CN: 继续补充 function _msg_dict_from_dcp_method_args 的文档字符串内容。
- **L30** EN: Closes the docstring for the function _msg_dict_from_dcp_method_args. | CN: 结束 function _msg_dict_from_dcp_method_args 的文档字符串。
- **L31** EN: Assigns or updates `msg_dict`. | CN: 对 `msg_dict` 进行赋值或更新。
- **L32** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L33** EN: Keeps the inline comment or directive: checkpoint ID can be passed in through the serializer or through the checkpoint  | CN: 保留这一行注释或指令：checkpoint ID can be passed in through the serializer or through the checkpoint 
- **L34** EN: Assigns or updates `storage_writer`. | CN: 对 `storage_writer` 进行赋值或更新。
- **L35** EN: Assigns or updates `storage_reader`. | CN: 对 `storage_reader` 进行赋值或更新。
- **L36** EN: Assigns or updates `planner`. | CN: 对 `planner` 进行赋值或更新。
- **L37** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L38** EN: Assigns or updates `checkpoint_id`. | CN: 对 `checkpoint_id` 进行赋值或更新。
- **L39** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L40** EN: Assigns or updates `checkpoint_id`. | CN: 对 `checkpoint_id` 进行赋值或更新。

### Lines 41-60 / 第 41-60 行

````python

    msg_dict["checkpoint_id"] = (
        # pyrefly: ignore [unsupported-operation]
        str(checkpoint_id) if checkpoint_id is not None else checkpoint_id
    )

    # Uniquely identify a _dcp_method_logger wrapped function call.
    msg_dict["uuid"] = str(uuid4().int)

    if storage_writer:
        msg_dict["storage_writer"] = storage_writer.__class__.__name__

    if storage_reader:
        msg_dict["storage_reader"] = storage_reader.__class__.__name__

    if planner:
        msg_dict["planner"] = planner.__class__.__name__

    return msg_dict

````

- **L41** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L42** EN: Continues the implementation inside function `_msg_dict_from_dcp_method_args`. | CN: 继续说明函数 `_msg_dict_from_dcp_method_args` 内部的实现。
- **L43** EN: Keeps the inline comment or directive: pyrefly: ignore [unsupported-operation] | CN: 保留这一行注释或指令：pyrefly: ignore [unsupported-operation]
- **L44** EN: Calls `str` as part of the current workflow. | CN: 在当前流程中调用 `str`。
- **L45** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L46** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L47** EN: Keeps the inline comment or directive: Uniquely identify a _dcp_method_logger wrapped function call. | CN: 保留这一行注释或指令：Uniquely identify a _dcp_method_logger wrapped function call.
- **L48** EN: Continues the implementation inside function `_msg_dict_from_dcp_method_args`. | CN: 继续说明函数 `_msg_dict_from_dcp_method_args` 内部的实现。
- **L49** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L50** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L51** EN: Continues the implementation inside function `_msg_dict_from_dcp_method_args`. | CN: 继续说明函数 `_msg_dict_from_dcp_method_args` 内部的实现。
- **L52** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L53** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L54** EN: Continues the implementation inside function `_msg_dict_from_dcp_method_args`. | CN: 继续说明函数 `_msg_dict_from_dcp_method_args` 内部的实现。
- **L55** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L56** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L57** EN: Continues the implementation inside function `_msg_dict_from_dcp_method_args`. | CN: 继续说明函数 `_msg_dict_from_dcp_method_args` 内部的实现。
- **L58** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L59** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L60** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 61-80 / 第 61-80 行

````python

def _get_msg_dict(func_name, *args, **kwargs) -> dict[str, Any]:
    msg_dict = _msg_dict_from_dcp_method_args(*args, **kwargs)
    msg_dict.update(c10d_logger._get_msg_dict(func_name, *args, **kwargs))

    return msg_dict


def _dcp_method_logger(
    log_exceptions: bool = False, **wrapper_kwargs: Any
) -> Callable[[Callable[_P, _T]], Callable[_P, _T]]:  # pyre-ignore
    """This method decorator logs the start, end, and exception of wrapped events."""

    def decorator(func: Callable[_P, _T]):
        @functools.wraps(func)
        def wrapper(*args: _P.args, **kwargs: _P.kwargs) -> _T:
            msg_dict = _get_msg_dict(
                func.__name__, *args, **{**wrapper_kwargs, **kwargs}
            )

````

- **L61** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L62** EN: Defines function `_get_msg_dict`. | CN: 定义函数 `_get_msg_dict`。
- **L63** EN: Assigns or updates `msg_dict`. | CN: 对 `msg_dict` 进行赋值或更新。
- **L64** EN: Calls `msg_dict.update` as part of the current workflow. | CN: 在当前流程中调用 `msg_dict.update`。
- **L65** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L66** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L67** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L68** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L69** EN: Defines function `_dcp_method_logger`. | CN: 定义函数 `_dcp_method_logger`。
- **L70** EN: Assigns or updates `log_exceptions`. | CN: 对 `log_exceptions` 进行赋值或更新。
- **L71** EN: Continues the implementation inside function `_dcp_method_logger`. | CN: 继续说明函数 `_dcp_method_logger` 内部的实现。
- **L72** EN: Docstring line documenting the function _dcp_method_logger. | CN: 这是记录 function _dcp_method_logger 的文档字符串。
- **L73** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L74** EN: Defines function `decorator`. | CN: 定义函数 `decorator`。
- **L75** EN: Applies decorator `functools.wraps(func)` to the following definition. | CN: 将装饰器 `functools.wraps(func)` 应用于后续定义。
- **L76** EN: Defines function `wrapper`. | CN: 定义函数 `wrapper`。
- **L77** EN: Assigns or updates `msg_dict`. | CN: 对 `msg_dict` 进行赋值或更新。
- **L78** EN: Continues the implementation inside function `wrapper`. | CN: 继续说明函数 `wrapper` 内部的实现。
- **L79** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L80** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 81-100 / 第 81-100 行

````python
            # log start event
            msg_dict["event"] = "start"
            t0 = time.time_ns()
            msg_dict["time"] = t0
            msg_dict["log_exceptions"] = log_exceptions
            _dcp_logger.debug(msg_dict)

            # exceptions
            try:
                result = func(*args, **kwargs)
            except BaseException as error:
                if log_exceptions:
                    msg_dict["event"] = "exception"
                    msg_dict["error"] = f"{error}"
                    msg_dict["time"] = time.time_ns()
                    _dcp_logger.error(msg_dict)
                raise

            # end event
            msg_dict["event"] = "end"
````

- **L81** EN: Keeps the inline comment or directive: log start event | CN: 保留这一行注释或指令：log start event
- **L82** EN: Continues the implementation inside function `wrapper`. | CN: 继续说明函数 `wrapper` 内部的实现。
- **L83** EN: Assigns or updates `t0`. | CN: 对 `t0` 进行赋值或更新。
- **L84** EN: Continues the implementation inside function `wrapper`. | CN: 继续说明函数 `wrapper` 内部的实现。
- **L85** EN: Continues the implementation inside function `wrapper`. | CN: 继续说明函数 `wrapper` 内部的实现。
- **L86** EN: Calls `_dcp_logger.debug` as part of the current workflow. | CN: 在当前流程中调用 `_dcp_logger.debug`。
- **L87** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L88** EN: Keeps the inline comment or directive: exceptions | CN: 保留这一行注释或指令：exceptions
- **L89** EN: Starts a protected block for exception handling. | CN: 开始一个受保护的异常处理代码块。
- **L90** EN: Assigns or updates `result`. | CN: 对 `result` 进行赋值或更新。
- **L91** EN: Handles a matching exception path. | CN: 处理匹配到的异常路径。
- **L92** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L93** EN: Continues the implementation inside function `wrapper`. | CN: 继续说明函数 `wrapper` 内部的实现。
- **L94** EN: Continues the implementation inside function `wrapper`. | CN: 继续说明函数 `wrapper` 内部的实现。
- **L95** EN: Continues the implementation inside function `wrapper`. | CN: 继续说明函数 `wrapper` 内部的实现。
- **L96** EN: Calls `_dcp_logger.error` as part of the current workflow. | CN: 在当前流程中调用 `_dcp_logger.error`。
- **L97** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L98** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L99** EN: Keeps the inline comment or directive: end event | CN: 保留这一行注释或指令：end event
- **L100** EN: Continues the implementation inside function `wrapper`. | CN: 继续说明函数 `wrapper` 内部的实现。

### Lines 101-120 / 第 101-120 行

````python
            t1 = time.time_ns()
            msg_dict["time"] = time.time_ns()
            msg_dict["times_spent"] = t1 - t0
            _dcp_logger.debug(msg_dict)

            return result

        return wrapper

    return decorator


def _init_logger(rank: int):
    logger.setLevel(logging.INFO)
    ch = logging.StreamHandler()
    ch.setLevel(logging.INFO)
    formatter = logging.Formatter(
        f"[{rank}] %(asctime)s - %(name)s - %(levelname)s - %(message)s"
    )
    ch.setFormatter(formatter)
````

- **L101** EN: Assigns or updates `t1`. | CN: 对 `t1` 进行赋值或更新。
- **L102** EN: Continues the implementation inside function `wrapper`. | CN: 继续说明函数 `wrapper` 内部的实现。
- **L103** EN: Continues the implementation inside function `wrapper`. | CN: 继续说明函数 `wrapper` 内部的实现。
- **L104** EN: Calls `_dcp_logger.debug` as part of the current workflow. | CN: 在当前流程中调用 `_dcp_logger.debug`。
- **L105** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L106** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L107** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L108** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L109** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L110** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L111** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L112** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L113** EN: Defines function `_init_logger`. | CN: 定义函数 `_init_logger`。
- **L114** EN: Calls `logger.setLevel` as part of the current workflow. | CN: 在当前流程中调用 `logger.setLevel`。
- **L115** EN: Assigns or updates `ch`. | CN: 对 `ch` 进行赋值或更新。
- **L116** EN: Calls `ch.setLevel` as part of the current workflow. | CN: 在当前流程中调用 `ch.setLevel`。
- **L117** EN: Assigns or updates `formatter`. | CN: 对 `formatter` 进行赋值或更新。
- **L118** EN: Continues the implementation inside function `_init_logger`. | CN: 继续说明函数 `_init_logger` 内部的实现。
- **L119** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L120** EN: Calls `ch.setFormatter` as part of the current workflow. | CN: 在当前流程中调用 `ch.setFormatter`。

### Lines 121-121 / 第 121-121 行

````python
    logger.addHandler(ch)
````

- **L121** EN: Calls `logger.addHandler` as part of the current workflow. | CN: 在当前流程中调用 `logger.addHandler`。

## Key Concepts / 关键概念

- **EN**: distributed checkpointing, planners, and storage helpers  
  **CN**: 分布式检查点、规划器与存储辅助逻辑
- **EN**: checkpointing  
  **CN**: 检查点
- **EN**: Core callables: _msg_dict_from_dcp_method_args, _get_msg_dict, _dcp_method_logger, _init_logger  
  **CN**: 核心可调用对象：_msg_dict_from_dcp_method_args, _get_msg_dict, _dcp_method_logger, _init_logger

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed.c10d_logger`, `torch.distributed.checkpoint.logging_handlers`
- **PyTorch / PyTorch**: None detected / 未检测到
- **Python Stdlib / Python 标准库**: `collections.abc`, `functools`, `logging`, `time`, `typing`, `uuid`
- **Third-party / 第三方**: `typing_extensions`

