# memory_tracker.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/_tools/memory_tracker.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed runtime helpers, APIs, and package wiring. Its main entry points include MemoryProfileDispatchMode, MemoryTracker.
- **用途 (CN)**: 该模块聚焦于分布式运行时辅助逻辑、API 与包级导出，其主要入口包括 MemoryProfileDispatchMode, MemoryTracker。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs
import operator
import pickle
from collections import defaultdict
from collections.abc import Callable, Sequence
from itertools import chain
from typing import Any, no_type_check, TYPE_CHECKING

import torch
import torch.nn as nn
from torch.utils._python_dispatch import TorchDispatchMode


if TYPE_CHECKING:
    from torch.utils.hooks import RemovableHandle


BYTES_PER_MB = 1024 * 1024.0


````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Imports module dependencies: `operator`. | CN: 导入模块依赖：`operator`。
- **L3** EN: Imports module dependencies: `pickle`. | CN: 导入模块依赖：`pickle`。
- **L4** EN: Imports selected names from `collections`. | CN: 从 `collections` 导入指定名称。
- **L5** EN: Imports selected names from `collections.abc`. | CN: 从 `collections.abc` 导入指定名称。
- **L6** EN: Imports selected names from `itertools`. | CN: 从 `itertools` 导入指定名称。
- **L7** EN: Imports selected names from `typing`. | CN: 从 `typing` 导入指定名称。
- **L8** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L9** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L10** EN: Imports module dependencies: `torch.nn as nn`. | CN: 导入模块依赖：`torch.nn as nn`。
- **L11** EN: Imports selected names from `torch.utils._python_dispatch`. | CN: 从 `torch.utils._python_dispatch` 导入指定名称。
- **L12** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L13** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L14** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L15** EN: Imports selected names from `torch.utils.hooks`. | CN: 从 `torch.utils.hooks` 导入指定名称。
- **L16** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L17** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L18** EN: Assigns or updates `BYTES_PER_MB`. | CN: 对 `BYTES_PER_MB` 进行赋值或更新。
- **L19** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L20** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 21-40 / 第 21-40 行

````python
class MemoryProfileDispatchMode(TorchDispatchMode):
    """Run in ``TorchDispatchMode`` to get memory stats at operator level."""

    def __init__(self, memory_tracker) -> None:
        self.memory_tracker = memory_tracker

    def __torch_dispatch__(self, func, types, args=..., kwargs=None):
        rs = func(*args, **kwargs)
        if func is torch.ops.aten.detach.default:
            return rs
        func_name: str = (
            self.memory_tracker._cur_module_name
            + "."
            + func.__name__
            + "_"
            + str(self.memory_tracker._operator_names[func.__name__])
        )
        self.memory_tracker._operator_names[func.__name__] = (
            self.memory_tracker._operator_names[func.__name__] + 1
        )
````

- **L21** EN: Defines class `MemoryProfileDispatchMode`. | CN: 定义类 `MemoryProfileDispatchMode`。
- **L22** EN: Docstring line documenting the class MemoryProfileDispatchMode. | CN: 这是记录 class MemoryProfileDispatchMode 的文档字符串。
- **L23** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L24** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L25** EN: Assigns or updates `self.memory_tracker`. | CN: 对 `self.memory_tracker` 进行赋值或更新。
- **L26** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L27** EN: Defines function `__torch_dispatch__`. | CN: 定义函数 `__torch_dispatch__`。
- **L28** EN: Assigns or updates `rs`. | CN: 对 `rs` 进行赋值或更新。
- **L29** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L30** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L31** EN: Assigns or updates `func_name`. | CN: 对 `func_name` 进行赋值或更新。
- **L32** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L33** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L34** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L35** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L36** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L37** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L38** EN: Assigns or updates `self.memory_tracker._operator_names[func.__name__]`. | CN: 对 `self.memory_tracker._operator_names[func.__name__]` 进行赋值或更新。
- **L39** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L40** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 41-60 / 第 41-60 行

````python
        self.memory_tracker._record_memory_stats(func_name)

        return rs


class MemoryTracker:
    """
    Collect and plot the memory stats at operator level.

    Includes ``memories_allocated``, ``memories_active`` and ``memories_reserved``.
    It also prints a summary for the top 20 operators that generate the most memories.

    Example usage:

        >>> # xdoctest: +SKIP(failing)
        >>> net.cuda()
        >>> input = input.cuda()

        >>> mem_tracker = MemoryTracker()
        >>> mem_tracker.start_monitor(net)
````

- **L41** EN: Calls `self.memory_tracker._record_memory_stats` as part of the current workflow. | CN: 在当前流程中调用 `self.memory_tracker._record_memory_stats`。
- **L42** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L43** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L44** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L45** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L46** EN: Defines class `MemoryTracker`. | CN: 定义类 `MemoryTracker`。
- **L47** EN: Starts the docstring for the class MemoryTracker. | CN: 开始定义 class MemoryTracker 的文档字符串。
- **L48** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L49** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L50** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L51** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L52** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L53** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L54** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L55** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L56** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L57** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L58** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L59** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L60** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。

### Lines 61-80 / 第 61-80 行

````python

        >>> net.zero_grad(True)
        >>> loss = net(input)
        >>> if isinstance(loss, dict):
        >>>    loss = loss['out']
        >>> loss.sum().backward()
        >>> net.zero_grad(set_to_none=True)

        >>> mem_tracker.stop()
        >>> mem_tracker.summary()
        >>> mem_tracker.show_traces()
    """

    def __init__(self) -> None:
        torch._C._log_api_usage_once("torch.distributed.memory_tracker")
        self._hooks: list[RemovableHandle] = []
        self._operator_names: dict[str, int] = defaultdict(int)
        self.memories_allocated: dict[int, dict[str, float]] = defaultdict()
        self.memories_active: dict[int, dict[str, float]] = defaultdict()
        self.memories_reserved: dict[int, dict[str, float]] = defaultdict()
````

- **L61** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L62** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L63** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L64** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L65** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L66** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L67** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L68** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L69** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L70** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L71** EN: Continues the docstring text for the class MemoryTracker. | CN: 继续补充 class MemoryTracker 的文档字符串内容。
- **L72** EN: Closes the docstring for the class MemoryTracker. | CN: 结束 class MemoryTracker 的文档字符串。
- **L73** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L74** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L75** EN: Calls `torch._C._log_api_usage_once` as part of the current workflow. | CN: 在当前流程中调用 `torch._C._log_api_usage_once`。
- **L76** EN: Assigns or updates `self._hooks`. | CN: 对 `self._hooks` 进行赋值或更新。
- **L77** EN: Assigns or updates `self._operator_names`. | CN: 对 `self._operator_names` 进行赋值或更新。
- **L78** EN: Assigns or updates `self.memories_allocated`. | CN: 对 `self.memories_allocated` 进行赋值或更新。
- **L79** EN: Assigns or updates `self.memories_active`. | CN: 对 `self.memories_active` 进行赋值或更新。
- **L80** EN: Assigns or updates `self.memories_reserved`. | CN: 对 `self.memories_reserved` 进行赋值或更新。

### Lines 81-100 / 第 81-100 行

````python
        self._markers: dict[str, int] = defaultdict(int)
        self._cur_module_name: str = ""
        self._op_index: int = 0
        self._num_alloc_retries: int = 0
        self._device_module = torch.get_device_module()

    @no_type_check
    def start_monitor(self, root_module: nn.Module) -> None:
        """
        Register module hooks and entering ``MemoryProfileDispatchMode``.

        This enables operator level memory stats can be tracked during module runtime.
        """
        self._clear_state()
        root_module.__setattr__("_memory_tracker_is_root", True)
        for name, m in root_module.named_modules():
            if m is not root_module:
                m.__setattr__("_memory_tracker_is_root", False)
            # fused_proxy_group does not support hooks
            if ".fused_proxy_grouped_embedding_bag" in name:
````

- **L81** EN: Assigns or updates `self._markers`. | CN: 对 `self._markers` 进行赋值或更新。
- **L82** EN: Assigns or updates `self._cur_module_name`. | CN: 对 `self._cur_module_name` 进行赋值或更新。
- **L83** EN: Assigns or updates `self._op_index`. | CN: 对 `self._op_index` 进行赋值或更新。
- **L84** EN: Assigns or updates `self._num_alloc_retries`. | CN: 对 `self._num_alloc_retries` 进行赋值或更新。
- **L85** EN: Assigns or updates `self._device_module`. | CN: 对 `self._device_module` 进行赋值或更新。
- **L86** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L87** EN: Applies decorator `no_type_check` to the following definition. | CN: 将装饰器 `no_type_check` 应用于后续定义。
- **L88** EN: Defines function `start_monitor`. | CN: 定义函数 `start_monitor`。
- **L89** EN: Starts the docstring for the function start_monitor. | CN: 开始定义 function start_monitor 的文档字符串。
- **L90** EN: Continues the docstring text for the function start_monitor. | CN: 继续补充 function start_monitor 的文档字符串内容。
- **L91** EN: Continues the docstring text for the function start_monitor. | CN: 继续补充 function start_monitor 的文档字符串内容。
- **L92** EN: Continues the docstring text for the function start_monitor. | CN: 继续补充 function start_monitor 的文档字符串内容。
- **L93** EN: Closes the docstring for the function start_monitor. | CN: 结束 function start_monitor 的文档字符串。
- **L94** EN: Calls `self._clear_state` as part of the current workflow. | CN: 在当前流程中调用 `self._clear_state`。
- **L95** EN: Calls `root_module.__setattr__` as part of the current workflow. | CN: 在当前流程中调用 `root_module.__setattr__`。
- **L96** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L97** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L98** EN: Calls `m.__setattr__` as part of the current workflow. | CN: 在当前流程中调用 `m.__setattr__`。
- **L99** EN: Keeps the inline comment or directive: fused_proxy_group does not support hooks | CN: 保留这一行注释或指令：fused_proxy_group does not support hooks
- **L100** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 101-120 / 第 101-120 行

````python
                continue
            # hook ordering with other hooks added by users is not managed, so
            # the memory stats tracked here may not completely accurate.
            h1 = m.register_forward_pre_hook(self._create_pre_forward_hook(name))
            h2 = m.register_forward_hook(self._create_post_forward_hook(name))
            # it does not work well with jagged tensor somehow, the root cause is not
            # clear and remove it for now as it does not really capture important info.
            # h3 = m.register_backward_hook(self._create_backward_hook(name))
            self._hooks.extend([h1, h2])
        self._device_module.empty_cache()
        if getattr(self, "profile_mode", None) is not None:
            raise AssertionError
        self.profile_mode = MemoryProfileDispatchMode(self)
        self.profile_mode.__enter__()

    @no_type_check
    def stop(self) -> None:
        """
        Remove module hooks and exit ``MemoryProfileDispatchMode`` to stop tracking memory stats at operator level.

````

- **L101** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L102** EN: Keeps the inline comment or directive: hook ordering with other hooks added by users is not managed, so | CN: 保留这一行注释或指令：hook ordering with other hooks added by users is not managed, so
- **L103** EN: Keeps the inline comment or directive: the memory stats tracked here may not completely accurate. | CN: 保留这一行注释或指令：the memory stats tracked here may not completely accurate.
- **L104** EN: Assigns or updates `h1`. | CN: 对 `h1` 进行赋值或更新。
- **L105** EN: Assigns or updates `h2`. | CN: 对 `h2` 进行赋值或更新。
- **L106** EN: Keeps the inline comment or directive: it does not work well with jagged tensor somehow, the root cause is not | CN: 保留这一行注释或指令：it does not work well with jagged tensor somehow, the root cause is not
- **L107** EN: Keeps the inline comment or directive: clear and remove it for now as it does not really capture important info. | CN: 保留这一行注释或指令：clear and remove it for now as it does not really capture important info.
- **L108** EN: Keeps the inline comment or directive: h3 = m.register_backward_hook(self._create_backward_hook(name)) | CN: 保留这一行注释或指令：h3 = m.register_backward_hook(self._create_backward_hook(name))
- **L109** EN: Calls `self._hooks.extend` as part of the current workflow. | CN: 在当前流程中调用 `self._hooks.extend`。
- **L110** EN: Calls `self._device_module.empty_cache` as part of the current workflow. | CN: 在当前流程中调用 `self._device_module.empty_cache`。
- **L111** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L112** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L113** EN: Assigns or updates `self.profile_mode`. | CN: 对 `self.profile_mode` 进行赋值或更新。
- **L114** EN: Calls `self.profile_mode.__enter__` as part of the current workflow. | CN: 在当前流程中调用 `self.profile_mode.__enter__`。
- **L115** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L116** EN: Applies decorator `no_type_check` to the following definition. | CN: 将装饰器 `no_type_check` 应用于后续定义。
- **L117** EN: Defines function `stop`. | CN: 定义函数 `stop`。
- **L118** EN: Starts the docstring for the function stop. | CN: 开始定义 function stop 的文档字符串。
- **L119** EN: Continues the docstring text for the function stop. | CN: 继续补充 function stop 的文档字符串内容。
- **L120** EN: Continues the docstring text for the function stop. | CN: 继续补充 function stop 的文档字符串内容。

### Lines 121-140 / 第 121-140 行

````python
        Get some aggregated stats when the memory_tracker() is enabled, like ``num_alloc_retries``.
        """
        self._num_alloc_retries = self._device_module.memory_stats().get(
            "num_alloc_retries", 0
        )

        for h in self._hooks:
            h.remove()
        self._hooks.clear()
        if getattr(self, "profile_mode", None) is None:
            raise AssertionError
        self.profile_mode.__exit__(None, None, None)
        self.profile_mode = None

    @no_type_check
    def summary(self, top: int = 20) -> None:
        """
        Print out the top operators that generate the most memories.

        The number of the top operators can be configured.
````

- **L121** EN: Continues the docstring text for the function stop. | CN: 继续补充 function stop 的文档字符串内容。
- **L122** EN: Closes the docstring for the function stop. | CN: 结束 function stop 的文档字符串。
- **L123** EN: Assigns or updates `self._num_alloc_retries`. | CN: 对 `self._num_alloc_retries` 进行赋值或更新。
- **L124** EN: Continues the implementation inside function `stop`. | CN: 继续说明函数 `stop` 内部的实现。
- **L125** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L126** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L127** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L128** EN: Calls `h.remove` as part of the current workflow. | CN: 在当前流程中调用 `h.remove`。
- **L129** EN: Calls `self._hooks.clear` as part of the current workflow. | CN: 在当前流程中调用 `self._hooks.clear`。
- **L130** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L131** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L132** EN: Calls `self.profile_mode.__exit__` as part of the current workflow. | CN: 在当前流程中调用 `self.profile_mode.__exit__`。
- **L133** EN: Assigns or updates `self.profile_mode`. | CN: 对 `self.profile_mode` 进行赋值或更新。
- **L134** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L135** EN: Applies decorator `no_type_check` to the following definition. | CN: 将装饰器 `no_type_check` 应用于后续定义。
- **L136** EN: Defines function `summary`. | CN: 定义函数 `summary`。
- **L137** EN: Starts the docstring for the function summary. | CN: 开始定义 function summary 的文档字符串。
- **L138** EN: Continues the docstring text for the function summary. | CN: 继续补充 function summary 的文档字符串内容。
- **L139** EN: Continues the docstring text for the function summary. | CN: 继续补充 function summary 的文档字符串内容。
- **L140** EN: Continues the docstring text for the function summary. | CN: 继续补充 function summary 的文档字符串内容。

### Lines 141-160 / 第 141-160 行

````python
        """
        op_diff: dict[str, float] = defaultdict(float)
        op_name, previous_allocated_memory = self.memories_allocated[0]
        for i in range(1, self._op_index):
            op_name, current_allocated_memory = self.memories_allocated[i]
            op_diff[op_name] = current_allocated_memory - previous_allocated_memory
            previous_allocated_memory = current_allocated_memory

        print("------------------------------------------------")
        print(f"The number of alloc retries are: {self._num_alloc_retries}")
        print(f"Top {top} ops that generates memory are:")
        for k, v in sorted(op_diff.items(), key=operator.itemgetter(1), reverse=True)[
            :top
        ]:
            print(f"{k}: {v}MB")
        print("------------------------------------------------")

    @no_type_check
    def show_traces(self, path: str = "") -> None:
        import matplotlib.pyplot as plt
````

- **L141** EN: Closes the docstring for the function summary. | CN: 结束 function summary 的文档字符串。
- **L142** EN: Assigns or updates `op_diff`. | CN: 对 `op_diff` 进行赋值或更新。
- **L143** EN: Assigns or updates `op_name, previous_allocated_memory`. | CN: 对 `op_name, previous_allocated_memory` 进行赋值或更新。
- **L144** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L145** EN: Assigns or updates `op_name, current_allocated_memory`. | CN: 对 `op_name, current_allocated_memory` 进行赋值或更新。
- **L146** EN: Assigns or updates `op_diff[op_name]`. | CN: 对 `op_diff[op_name]` 进行赋值或更新。
- **L147** EN: Assigns or updates `previous_allocated_memory`. | CN: 对 `previous_allocated_memory` 进行赋值或更新。
- **L148** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L149** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L150** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L151** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L152** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L153** EN: Continues the implementation inside function `summary`. | CN: 继续说明函数 `summary` 内部的实现。
- **L154** EN: Continues the implementation inside function `summary`. | CN: 继续说明函数 `summary` 内部的实现。
- **L155** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L156** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L157** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L158** EN: Applies decorator `no_type_check` to the following definition. | CN: 将装饰器 `no_type_check` 应用于后续定义。
- **L159** EN: Defines function `show_traces`. | CN: 定义函数 `show_traces`。
- **L160** EN: Imports module dependencies: `matplotlib.pyplot as plt`. | CN: 导入模块依赖：`matplotlib.pyplot as plt`。

### Lines 161-180 / 第 161-180 行

````python

        def _plot_figure(x, y_values, labels):
            min_val = min(chain.from_iterable(y_values)) * 0.999
            max_val = max(chain.from_iterable(y_values)) * 1.001
            plt.figure()
            for y, label in zip(y_values, labels):
                plt.plot(x, y, label=label)
            plt.xlabel("# Operator Calls")
            plt.ylabel("Memory (MB)")
            plt.legend()
            for marker_name, marker in self._markers.items():
                if marker_name == "fw_bw_boundary":
                    plt.plot(
                        [marker, marker],
                        [min_val, max_val],
                        "r",
                        lw=2,
                        label=marker_name,
                    )
                else:
````

- **L161** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L162** EN: Defines function `_plot_figure`. | CN: 定义函数 `_plot_figure`。
- **L163** EN: Assigns or updates `min_val`. | CN: 对 `min_val` 进行赋值或更新。
- **L164** EN: Assigns or updates `max_val`. | CN: 对 `max_val` 进行赋值或更新。
- **L165** EN: Calls `plt.figure` as part of the current workflow. | CN: 在当前流程中调用 `plt.figure`。
- **L166** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L167** EN: Calls `plt.plot` as part of the current workflow. | CN: 在当前流程中调用 `plt.plot`。
- **L168** EN: Calls `plt.xlabel` as part of the current workflow. | CN: 在当前流程中调用 `plt.xlabel`。
- **L169** EN: Calls `plt.ylabel` as part of the current workflow. | CN: 在当前流程中调用 `plt.ylabel`。
- **L170** EN: Calls `plt.legend` as part of the current workflow. | CN: 在当前流程中调用 `plt.legend`。
- **L171** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L172** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L173** EN: Calls `plt.plot` as part of the current workflow. | CN: 在当前流程中调用 `plt.plot`。
- **L174** EN: Continues the implementation inside function `_plot_figure`. | CN: 继续说明函数 `_plot_figure` 内部的实现。
- **L175** EN: Continues the implementation inside function `_plot_figure`. | CN: 继续说明函数 `_plot_figure` 内部的实现。
- **L176** EN: Continues the implementation inside function `_plot_figure`. | CN: 继续说明函数 `_plot_figure` 内部的实现。
- **L177** EN: Assigns or updates `lw`. | CN: 对 `lw` 进行赋值或更新。
- **L178** EN: Assigns or updates `label`. | CN: 对 `label` 进行赋值或更新。
- **L179** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L180** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。

### Lines 181-200 / 第 181-200 行

````python
                    plt.plot(
                        [marker, marker],
                        [min_val, max_val],
                        "k-",
                        lw=2,
                        label=marker_name,
                    )

        if path != "":
            self.load(path)

        y_1 = [gb for (name, gb) in self.memories_allocated.values()]
        y_2 = [gb for (name, gb) in self.memories_active.values()]
        y_3 = [gb for (name, gb) in self.memories_reserved.values()]
        x = list(range(len(y_1)))
        # Split figures when there is big difference between
        # "reserved_memory" and "allocated_memory" or "active_memory".
        _plot_figure(
            x,
            [list(y_1), list(y_2), list(y_3)],
````

- **L181** EN: Calls `plt.plot` as part of the current workflow. | CN: 在当前流程中调用 `plt.plot`。
- **L182** EN: Continues the implementation inside function `_plot_figure`. | CN: 继续说明函数 `_plot_figure` 内部的实现。
- **L183** EN: Continues the implementation inside function `_plot_figure`. | CN: 继续说明函数 `_plot_figure` 内部的实现。
- **L184** EN: Continues the implementation inside function `_plot_figure`. | CN: 继续说明函数 `_plot_figure` 内部的实现。
- **L185** EN: Assigns or updates `lw`. | CN: 对 `lw` 进行赋值或更新。
- **L186** EN: Assigns or updates `label`. | CN: 对 `label` 进行赋值或更新。
- **L187** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L188** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L189** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L190** EN: Calls `self.load` as part of the current workflow. | CN: 在当前流程中调用 `self.load`。
- **L191** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L192** EN: Assigns or updates `y_1`. | CN: 对 `y_1` 进行赋值或更新。
- **L193** EN: Assigns or updates `y_2`. | CN: 对 `y_2` 进行赋值或更新。
- **L194** EN: Assigns or updates `y_3`. | CN: 对 `y_3` 进行赋值或更新。
- **L195** EN: Assigns or updates `x`. | CN: 对 `x` 进行赋值或更新。
- **L196** EN: Keeps the inline comment or directive: Split figures when there is big difference between | CN: 保留这一行注释或指令：Split figures when there is big difference between
- **L197** EN: Keeps the inline comment or directive: "reserved_memory" and "allocated_memory" or "active_memory". | CN: 保留这一行注释或指令："reserved_memory" and "allocated_memory" or "active_memory".
- **L198** EN: Calls `_plot_figure` as part of the current workflow. | CN: 在当前流程中调用 `_plot_figure`。
- **L199** EN: Continues the implementation inside function `show_traces`. | CN: 继续说明函数 `show_traces` 内部的实现。
- **L200** EN: Continues the implementation inside function `show_traces`. | CN: 继续说明函数 `show_traces` 内部的实现。

### Lines 201-220 / 第 201-220 行

````python
            ["allocated_memory", "active_memory", "reserved_memory"],
        )
        _plot_figure(x, [list(y_1)], ["allocated_memory"])
        _plot_figure(x, [list(y_2)], ["active_memory"])
        _plot_figure(x, [list(y_3)], ["reserved_memory"])

    def save_stats(self, path: str) -> None:
        """Save the stats using pickle during runtime if users want to plot the traces in other places like notebook."""
        stats = {
            "memories_allocated": self.memories_allocated,
            "memories_active": self.memories_active,
            "memories_reserved": self.memories_reserved,
            "markers": self._markers,
            "num_alloc_retries": self._num_alloc_retries,
        }

        with open(path, "wb") as f:
            pickle.dump(stats, f, pickle.HIGHEST_PROTOCOL)

    def load(self, path: str) -> None:
````

- **L201** EN: Continues the implementation inside function `show_traces`. | CN: 继续说明函数 `show_traces` 内部的实现。
- **L202** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L203** EN: Calls `_plot_figure` as part of the current workflow. | CN: 在当前流程中调用 `_plot_figure`。
- **L204** EN: Calls `_plot_figure` as part of the current workflow. | CN: 在当前流程中调用 `_plot_figure`。
- **L205** EN: Calls `_plot_figure` as part of the current workflow. | CN: 在当前流程中调用 `_plot_figure`。
- **L206** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L207** EN: Defines function `save_stats`. | CN: 定义函数 `save_stats`。
- **L208** EN: Docstring line documenting the function save_stats. | CN: 这是记录 function save_stats 的文档字符串。
- **L209** EN: Assigns or updates `stats`. | CN: 对 `stats` 进行赋值或更新。
- **L210** EN: Continues the implementation inside function `save_stats`. | CN: 继续说明函数 `save_stats` 内部的实现。
- **L211** EN: Continues the implementation inside function `save_stats`. | CN: 继续说明函数 `save_stats` 内部的实现。
- **L212** EN: Continues the implementation inside function `save_stats`. | CN: 继续说明函数 `save_stats` 内部的实现。
- **L213** EN: Continues the implementation inside function `save_stats`. | CN: 继续说明函数 `save_stats` 内部的实现。
- **L214** EN: Continues the implementation inside function `save_stats`. | CN: 继续说明函数 `save_stats` 内部的实现。
- **L215** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L216** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L217** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L218** EN: Calls `pickle.dump` as part of the current workflow. | CN: 在当前流程中调用 `pickle.dump`。
- **L219** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L220** EN: Defines function `load`. | CN: 定义函数 `load`。

### Lines 221-240 / 第 221-240 行

````python
        """Load the pickled memory stats to plot the traces or print the summary."""
        with open(path, "rb") as f:
            stats = pickle.load(f)

        self.memories_allocated = stats["memories_allocated"]
        self.memories_active = stats["memories_active"]
        self.memories_reserved = stats["memories_reserved"]
        self._markers = stats["markers"]
        self._num_alloc_retries = stats["num_alloc_retries"]

    def _create_pre_forward_hook(self, name: str) -> Callable:
        """Prefix operator name with current module and 'forward', and insert 'fw_start' marker at forward pass start."""

        def _pre_forward_hook(module: nn.Module, inputs: Any) -> None:
            self._cur_module_name = f"{name}.forward"
            if (
                hasattr(module, "_memory_tracker_is_root")
                and module._memory_tracker_is_root
            ):
                self._add_marker("fw_start")
````

- **L221** EN: Docstring line documenting the function load. | CN: 这是记录 function load 的文档字符串。
- **L222** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L223** EN: Assigns or updates `stats`. | CN: 对 `stats` 进行赋值或更新。
- **L224** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L225** EN: Assigns or updates `self.memories_allocated`. | CN: 对 `self.memories_allocated` 进行赋值或更新。
- **L226** EN: Assigns or updates `self.memories_active`. | CN: 对 `self.memories_active` 进行赋值或更新。
- **L227** EN: Assigns or updates `self.memories_reserved`. | CN: 对 `self.memories_reserved` 进行赋值或更新。
- **L228** EN: Assigns or updates `self._markers`. | CN: 对 `self._markers` 进行赋值或更新。
- **L229** EN: Assigns or updates `self._num_alloc_retries`. | CN: 对 `self._num_alloc_retries` 进行赋值或更新。
- **L230** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L231** EN: Defines function `_create_pre_forward_hook`. | CN: 定义函数 `_create_pre_forward_hook`。
- **L232** EN: Docstring line documenting the function _create_pre_forward_hook. | CN: 这是记录 function _create_pre_forward_hook 的文档字符串。
- **L233** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L234** EN: Defines function `_pre_forward_hook`. | CN: 定义函数 `_pre_forward_hook`。
- **L235** EN: Assigns or updates `self._cur_module_name`. | CN: 对 `self._cur_module_name` 进行赋值或更新。
- **L236** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L237** EN: Calls `hasattr` as part of the current workflow. | CN: 在当前流程中调用 `hasattr`。
- **L238** EN: Continues the implementation inside function `_pre_forward_hook`. | CN: 继续说明函数 `_pre_forward_hook` 内部的实现。
- **L239** EN: Continues the implementation inside function `_pre_forward_hook`. | CN: 继续说明函数 `_pre_forward_hook` 内部的实现。
- **L240** EN: Calls `self._add_marker` as part of the current workflow. | CN: 在当前流程中调用 `self._add_marker`。

### Lines 241-260 / 第 241-260 行

````python

        return _pre_forward_hook

    def _create_post_forward_hook(self, name: str) -> Callable:
        """Insert the marker 'fw_bw_boundary' at the boundary of forward and backward pass."""

        def _post_forward_hook(
            module: nn.Module,
            inputs: Sequence[torch.Tensor],
            outputs: Sequence[torch.Tensor],
        ) -> None:
            if (
                hasattr(module, "_memory_tracker_is_root")
                and module._memory_tracker_is_root
            ):
                self._add_marker("fw_bw_boundary")

        return _post_forward_hook

    def _create_backward_hook(self, name: str) -> Callable:
````

- **L241** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L242** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L243** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L244** EN: Defines function `_create_post_forward_hook`. | CN: 定义函数 `_create_post_forward_hook`。
- **L245** EN: Docstring line documenting the function _create_post_forward_hook. | CN: 这是记录 function _create_post_forward_hook 的文档字符串。
- **L246** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L247** EN: Defines function `_post_forward_hook`. | CN: 定义函数 `_post_forward_hook`。
- **L248** EN: Continues the implementation inside function `_post_forward_hook`. | CN: 继续说明函数 `_post_forward_hook` 内部的实现。
- **L249** EN: Continues the implementation inside function `_post_forward_hook`. | CN: 继续说明函数 `_post_forward_hook` 内部的实现。
- **L250** EN: Continues the implementation inside function `_post_forward_hook`. | CN: 继续说明函数 `_post_forward_hook` 内部的实现。
- **L251** EN: Continues the implementation inside function `_post_forward_hook`. | CN: 继续说明函数 `_post_forward_hook` 内部的实现。
- **L252** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L253** EN: Calls `hasattr` as part of the current workflow. | CN: 在当前流程中调用 `hasattr`。
- **L254** EN: Continues the implementation inside function `_post_forward_hook`. | CN: 继续说明函数 `_post_forward_hook` 内部的实现。
- **L255** EN: Continues the implementation inside function `_post_forward_hook`. | CN: 继续说明函数 `_post_forward_hook` 内部的实现。
- **L256** EN: Calls `self._add_marker` as part of the current workflow. | CN: 在当前流程中调用 `self._add_marker`。
- **L257** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L258** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L259** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L260** EN: Defines function `_create_backward_hook`. | CN: 定义函数 `_create_backward_hook`。

### Lines 261-280 / 第 261-280 行

````python
        """Insert the current module name with backward prefix for the operator name."""

        def _backward_hook(
            module: nn.Module, grad_input: torch.Tensor, grad_output: torch.Tensor
        ) -> None:
            self._cur_module_name = f"{name}.backward"

        return _backward_hook

    @no_type_check
    def _record_memory_stats(self, fn_name: str) -> None:
        """
        Record current memory allocated, current memory active and current memory reserved.

        The memory stats dict is indexed with ``self._op_index``.
        """
        memory_allocated: float = self._device_module.memory_allocated() / BYTES_PER_MB
        memory_reserved: float = self._device_module.memory_reserved() / BYTES_PER_MB
        memory_active: float = (
            self._device_module.memory_stats().get("active_bytes.all.current", 0)
````

- **L261** EN: Docstring line documenting the function _create_backward_hook. | CN: 这是记录 function _create_backward_hook 的文档字符串。
- **L262** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L263** EN: Defines function `_backward_hook`. | CN: 定义函数 `_backward_hook`。
- **L264** EN: Continues the implementation inside function `_backward_hook`. | CN: 继续说明函数 `_backward_hook` 内部的实现。
- **L265** EN: Continues the implementation inside function `_backward_hook`. | CN: 继续说明函数 `_backward_hook` 内部的实现。
- **L266** EN: Assigns or updates `self._cur_module_name`. | CN: 对 `self._cur_module_name` 进行赋值或更新。
- **L267** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L268** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L269** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L270** EN: Applies decorator `no_type_check` to the following definition. | CN: 将装饰器 `no_type_check` 应用于后续定义。
- **L271** EN: Defines function `_record_memory_stats`. | CN: 定义函数 `_record_memory_stats`。
- **L272** EN: Starts the docstring for the function _record_memory_stats. | CN: 开始定义 function _record_memory_stats 的文档字符串。
- **L273** EN: Continues the docstring text for the function _record_memory_stats. | CN: 继续补充 function _record_memory_stats 的文档字符串内容。
- **L274** EN: Continues the docstring text for the function _record_memory_stats. | CN: 继续补充 function _record_memory_stats 的文档字符串内容。
- **L275** EN: Continues the docstring text for the function _record_memory_stats. | CN: 继续补充 function _record_memory_stats 的文档字符串内容。
- **L276** EN: Closes the docstring for the function _record_memory_stats. | CN: 结束 function _record_memory_stats 的文档字符串。
- **L277** EN: Assigns or updates `memory_allocated`. | CN: 对 `memory_allocated` 进行赋值或更新。
- **L278** EN: Assigns or updates `memory_reserved`. | CN: 对 `memory_reserved` 进行赋值或更新。
- **L279** EN: Assigns or updates `memory_active`. | CN: 对 `memory_active` 进行赋值或更新。
- **L280** EN: Calls `self._device_module.memory_stats` as part of the current workflow. | CN: 在当前流程中调用 `self._device_module.memory_stats`。

### Lines 281-300 / 第 281-300 行

````python
            / BYTES_PER_MB
        )
        self.memories_allocated[self._op_index] = (fn_name, memory_allocated)
        self.memories_reserved[self._op_index] = (fn_name, memory_reserved)
        self.memories_active[self._op_index] = (fn_name, memory_active)
        self._op_index += 1

    def _add_marker(self, marker_name: str) -> None:
        """Set the marker's x-axis value."""
        marker_val = len(self.memories_allocated.values())
        self._markers[marker_name] = marker_val

    def _clear_state(self) -> None:
        """Clear states when start_monitor() is called."""
        self._operator_names.clear()
        self.memories_allocated.clear()
        self.memories_active.clear()
        self.memories_reserved.clear()
        self._markers.clear()
        self._cur_module_name = ""
````

- **L281** EN: Continues the implementation inside function `_record_memory_stats`. | CN: 继续说明函数 `_record_memory_stats` 内部的实现。
- **L282** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L283** EN: Assigns or updates `self.memories_allocated[self._op_index]`. | CN: 对 `self.memories_allocated[self._op_index]` 进行赋值或更新。
- **L284** EN: Assigns or updates `self.memories_reserved[self._op_index]`. | CN: 对 `self.memories_reserved[self._op_index]` 进行赋值或更新。
- **L285** EN: Assigns or updates `self.memories_active[self._op_index]`. | CN: 对 `self.memories_active[self._op_index]` 进行赋值或更新。
- **L286** EN: Continues the implementation inside function `_record_memory_stats`. | CN: 继续说明函数 `_record_memory_stats` 内部的实现。
- **L287** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L288** EN: Defines function `_add_marker`. | CN: 定义函数 `_add_marker`。
- **L289** EN: Docstring line documenting the function _add_marker. | CN: 这是记录 function _add_marker 的文档字符串。
- **L290** EN: Assigns or updates `marker_val`. | CN: 对 `marker_val` 进行赋值或更新。
- **L291** EN: Assigns or updates `self._markers[marker_name]`. | CN: 对 `self._markers[marker_name]` 进行赋值或更新。
- **L292** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L293** EN: Defines function `_clear_state`. | CN: 定义函数 `_clear_state`。
- **L294** EN: Docstring line documenting the function _clear_state. | CN: 这是记录 function _clear_state 的文档字符串。
- **L295** EN: Calls `self._operator_names.clear` as part of the current workflow. | CN: 在当前流程中调用 `self._operator_names.clear`。
- **L296** EN: Calls `self.memories_allocated.clear` as part of the current workflow. | CN: 在当前流程中调用 `self.memories_allocated.clear`。
- **L297** EN: Calls `self.memories_active.clear` as part of the current workflow. | CN: 在当前流程中调用 `self.memories_active.clear`。
- **L298** EN: Calls `self.memories_reserved.clear` as part of the current workflow. | CN: 在当前流程中调用 `self.memories_reserved.clear`。
- **L299** EN: Calls `self._markers.clear` as part of the current workflow. | CN: 在当前流程中调用 `self._markers.clear`。
- **L300** EN: Assigns or updates `self._cur_module_name`. | CN: 对 `self._cur_module_name` 进行赋值或更新。

### Lines 301-302 / 第 301-302 行

````python
        self._op_index = 0
        self._num_alloc_retries = 0
````

- **L301** EN: Assigns or updates `self._op_index`. | CN: 对 `self._op_index` 进行赋值或更新。
- **L302** EN: Assigns or updates `self._num_alloc_retries`. | CN: 对 `self._num_alloc_retries` 进行赋值或更新。

## Key Concepts / 关键概念

- **EN**: distributed runtime helpers, APIs, and package wiring  
  **CN**: 分布式运行时辅助逻辑、API 与包级导出
- **EN**: communication hooks  
  **CN**: 通信钩子
- **EN**: Primary classes: MemoryProfileDispatchMode, MemoryTracker  
  **CN**: 主要类：MemoryProfileDispatchMode, MemoryTracker

## Dependencies / 依赖关系

- **Internal / 内部**: None detected / 未检测到
- **PyTorch / PyTorch**: `torch`, `torch.nn`, `torch.utils._python_dispatch`, `torch.utils.hooks`
- **Python Stdlib / Python 标准库**: `collections`, `collections.abc`, `itertools`, `operator`, `pickle`, `typing`
- **Third-party / 第三方**: `matplotlib.pyplot`

