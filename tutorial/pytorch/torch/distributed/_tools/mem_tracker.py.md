# mem_tracker.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/_tools/mem_tracker.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed runtime helpers, APIs, and package wiring. Its main entry points include _RefType, _State, _get_mem_divisor, _rounding_fn.
- **用途 (CN)**: 该模块聚焦于分布式运行时辅助逻辑、API 与包级导出，其主要入口包括 _RefType, _State, _get_mem_divisor, _rounding_fn。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
import math
import os
import re
import warnings
from collections.abc import Callable
from copy import deepcopy
from enum import auto, Enum
from functools import partial, wraps
from typing import Any, TYPE_CHECKING
from typing_extensions import Self

import torch
import torch.distributed._tools.fake_collectives
from torch import nn, optim
from torch._guards import active_fake_mode
from torch.distributed._tools.common_utils import get_untyped_storages
from torch.distributed._tools.mod_tracker import ModTracker
from torch.distributed.tensor import DTensor
from torch.optim.optimizer import (
    register_optimizer_step_post_hook,
````

- **L1** EN: Imports module dependencies: `math`. | CN: 导入模块依赖：`math`。
- **L2** EN: Imports module dependencies: `os`. | CN: 导入模块依赖：`os`。
- **L3** EN: Imports module dependencies: `re`. | CN: 导入模块依赖：`re`。
- **L4** EN: Imports module dependencies: `warnings`. | CN: 导入模块依赖：`warnings`。
- **L5** EN: Imports selected names from `collections.abc`. | CN: 从 `collections.abc` 导入指定名称。
- **L6** EN: Imports selected names from `copy`. | CN: 从 `copy` 导入指定名称。
- **L7** EN: Imports selected names from `enum`. | CN: 从 `enum` 导入指定名称。
- **L8** EN: Imports selected names from `functools`. | CN: 从 `functools` 导入指定名称。
- **L9** EN: Imports selected names from `typing`. | CN: 从 `typing` 导入指定名称。
- **L10** EN: Imports selected names from `typing_extensions`. | CN: 从 `typing_extensions` 导入指定名称。
- **L11** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L12** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L13** EN: Imports module dependencies: `torch.distributed._tools.fake_collectives`. | CN: 导入模块依赖：`torch.distributed._tools.fake_collectives`。
- **L14** EN: Imports selected names from `torch`. | CN: 从 `torch` 导入指定名称。
- **L15** EN: Imports selected names from `torch._guards`. | CN: 从 `torch._guards` 导入指定名称。
- **L16** EN: Imports selected names from `torch.distributed._tools.common_utils`. | CN: 从 `torch.distributed._tools.common_utils` 导入指定名称。
- **L17** EN: Imports selected names from `torch.distributed._tools.mod_tracker`. | CN: 从 `torch.distributed._tools.mod_tracker` 导入指定名称。
- **L18** EN: Imports selected names from `torch.distributed.tensor`. | CN: 从 `torch.distributed.tensor` 导入指定名称。
- **L19** EN: Imports selected names from `torch.optim.optimizer`. | CN: 从 `torch.optim.optimizer` 导入指定名称。
- **L20** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。

### Lines 21-40 / 第 21-40 行

````python
    register_optimizer_step_pre_hook,
)
from torch.utils._python_dispatch import TorchDispatchMode
from torch.utils._pytree import tree_flatten, tree_map_only
from torch.utils.weak import WeakIdKeyDictionary, weakref


if TYPE_CHECKING:
    from torch.utils.hooks import RemovableHandle

# This value is hard-coded here:
# https://github.com/pytorch/pytorch/blob/5fba5d83f0703ff8077ab65448a998e9ad6598fd/c10/cuda/CUDACachingAllocator.cpp#L117
_PYTORCH_MIN_ALLOCATE = (
    2**9 if int(os.environ.get("PYTORCH_NO_CUDA_MEMORY_CACHING", 0)) == 0 else 1
)
_TOTAL_KEY = "Total"

__all__ = ["MemTracker"]


````

- **L21** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L22** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L23** EN: Imports selected names from `torch.utils._python_dispatch`. | CN: 从 `torch.utils._python_dispatch` 导入指定名称。
- **L24** EN: Imports selected names from `torch.utils._pytree`. | CN: 从 `torch.utils._pytree` 导入指定名称。
- **L25** EN: Imports selected names from `torch.utils.weak`. | CN: 从 `torch.utils.weak` 导入指定名称。
- **L26** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L27** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L28** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L29** EN: Imports selected names from `torch.utils.hooks`. | CN: 从 `torch.utils.hooks` 导入指定名称。
- **L30** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L31** EN: Keeps the inline comment or directive: This value is hard-coded here: | CN: 保留这一行注释或指令：This value is hard-coded here:
- **L32** EN: Keeps the inline comment or directive: https://github.com/pytorch/pytorch/blob/5fba5d83f0703ff8077ab65448a998e9ad6598fd | CN: 保留这一行注释或指令：https://github.com/pytorch/pytorch/blob/5fba5d83f0703ff8077ab65448a998e9ad6598fd
- **L33** EN: Assigns or updates `_PYTORCH_MIN_ALLOCATE`. | CN: 对 `_PYTORCH_MIN_ALLOCATE` 进行赋值或更新。
- **L34** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L35** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L36** EN: Assigns or updates `_TOTAL_KEY`. | CN: 对 `_TOTAL_KEY` 进行赋值或更新。
- **L37** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L38** EN: Assigns or updates `__all__`. | CN: 对 `__all__` 进行赋值或更新。
- **L39** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L40** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 41-60 / 第 41-60 行

````python
class _RefType(str, Enum):
    """Base Class for defining memory reference types, categorizing tensors based on their usage within a model."""


class _State(str, Enum):
    """Base Class for defining module state to capture snapshots ."""


class _MemRefType(_RefType):
    """
    An enum to define memory reference types, categorizing tensors based on their usage within a model.

        - PARAM: Tensors registered as nn.Parameter within modules.
        - BUFFER: Tensors registered as nn.Buffer within modules.
        - GRAD: Gradients associated with parameters.
        - ACT: Tensors produced during the forward pass and recomputation in activation checkpointing.
        - TMP: Temporary memory used during the backward pass, including gradients of activations.
        - OPT: Tensors holding optimizer states.
        - OTH: Tensors registered via `track_external` that do not fit the above categories.
    """
````

- **L41** EN: Defines class `_RefType`. | CN: 定义类 `_RefType`。
- **L42** EN: Docstring line documenting the class _RefType. | CN: 这是记录 class _RefType 的文档字符串。
- **L43** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L44** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L45** EN: Defines class `_State`. | CN: 定义类 `_State`。
- **L46** EN: Docstring line documenting the class _State. | CN: 这是记录 class _State 的文档字符串。
- **L47** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L48** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L49** EN: Defines class `_MemRefType`. | CN: 定义类 `_MemRefType`。
- **L50** EN: Starts the docstring for the class _MemRefType. | CN: 开始定义 class _MemRefType 的文档字符串。
- **L51** EN: Continues the docstring text for the class _MemRefType. | CN: 继续补充 class _MemRefType 的文档字符串内容。
- **L52** EN: Continues the docstring text for the class _MemRefType. | CN: 继续补充 class _MemRefType 的文档字符串内容。
- **L53** EN: Continues the docstring text for the class _MemRefType. | CN: 继续补充 class _MemRefType 的文档字符串内容。
- **L54** EN: Continues the docstring text for the class _MemRefType. | CN: 继续补充 class _MemRefType 的文档字符串内容。
- **L55** EN: Continues the docstring text for the class _MemRefType. | CN: 继续补充 class _MemRefType 的文档字符串内容。
- **L56** EN: Continues the docstring text for the class _MemRefType. | CN: 继续补充 class _MemRefType 的文档字符串内容。
- **L57** EN: Continues the docstring text for the class _MemRefType. | CN: 继续补充 class _MemRefType 的文档字符串内容。
- **L58** EN: Continues the docstring text for the class _MemRefType. | CN: 继续补充 class _MemRefType 的文档字符串内容。
- **L59** EN: Continues the docstring text for the class _MemRefType. | CN: 继续补充 class _MemRefType 的文档字符串内容。
- **L60** EN: Closes the docstring for the class _MemRefType. | CN: 结束 class _MemRefType 的文档字符串。

### Lines 61-80 / 第 61-80 行

````python

    PARAM = "Parameter"
    BUFFER = "Buffer"
    GRAD = "Gradient"
    ACT = "Activation"
    TEMP = "Temp"
    OPT = "Optstate"
    OTH = "Other"


class _ModState(_State):
    """
    An enum to define the state of a module.

        - PRE_FW: The module is about to run the forward pass.
        - POST_FW: The module has finished running the forward pass.
        - PEAK_FW: The module has reached the peak memory usage during the forward pass.
        - PRE_BW: The module is about to run the backward pass.
        - PRE_FW_AC: The module is about to run the forward pass with activation checkpointing.
        - POST_FW_AC: The module has finished running the forward pass with activation checkpointing.
````

- **L61** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L62** EN: Assigns or updates `PARAM`. | CN: 对 `PARAM` 进行赋值或更新。
- **L63** EN: Assigns or updates `BUFFER`. | CN: 对 `BUFFER` 进行赋值或更新。
- **L64** EN: Assigns or updates `GRAD`. | CN: 对 `GRAD` 进行赋值或更新。
- **L65** EN: Assigns or updates `ACT`. | CN: 对 `ACT` 进行赋值或更新。
- **L66** EN: Assigns or updates `TEMP`. | CN: 对 `TEMP` 进行赋值或更新。
- **L67** EN: Assigns or updates `OPT`. | CN: 对 `OPT` 进行赋值或更新。
- **L68** EN: Assigns or updates `OTH`. | CN: 对 `OTH` 进行赋值或更新。
- **L69** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L70** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L71** EN: Defines class `_ModState`. | CN: 定义类 `_ModState`。
- **L72** EN: Starts the docstring for the class _ModState. | CN: 开始定义 class _ModState 的文档字符串。
- **L73** EN: Continues the docstring text for the class _ModState. | CN: 继续补充 class _ModState 的文档字符串内容。
- **L74** EN: Continues the docstring text for the class _ModState. | CN: 继续补充 class _ModState 的文档字符串内容。
- **L75** EN: Continues the docstring text for the class _ModState. | CN: 继续补充 class _ModState 的文档字符串内容。
- **L76** EN: Continues the docstring text for the class _ModState. | CN: 继续补充 class _ModState 的文档字符串内容。
- **L77** EN: Continues the docstring text for the class _ModState. | CN: 继续补充 class _ModState 的文档字符串内容。
- **L78** EN: Continues the docstring text for the class _ModState. | CN: 继续补充 class _ModState 的文档字符串内容。
- **L79** EN: Continues the docstring text for the class _ModState. | CN: 继续补充 class _ModState 的文档字符串内容。
- **L80** EN: Continues the docstring text for the class _ModState. | CN: 继续补充 class _ModState 的文档字符串内容。

### Lines 81-100 / 第 81-100 行

````python
        - POST_BW: The module has finished running the backward pass.
        - PEAK_BW: The module has reached the peak memory usage during the backward pass.
    """

    PRE_FW = "Pre-Forward"
    POST_FW = "Post-Forward"
    PEAK_FW = "Peak-Forward"
    PRE_BW = "Pre-Backward"
    PRE_FW_AC = "Pre-Forward-AC"
    POST_FW_AC = "Post-Forward-AC"
    POST_BW = "Post-Backward"
    PEAK_BW = "Peak-Backward"


class _ModMemStats:
    """
    A class to store the memory statistics of a module.

    Args:
        mod_fqn (str): The fully qualified name of the module.
````

- **L81** EN: Continues the docstring text for the class _ModState. | CN: 继续补充 class _ModState 的文档字符串内容。
- **L82** EN: Continues the docstring text for the class _ModState. | CN: 继续补充 class _ModState 的文档字符串内容。
- **L83** EN: Closes the docstring for the class _ModState. | CN: 结束 class _ModState 的文档字符串。
- **L84** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L85** EN: Assigns or updates `PRE_FW`. | CN: 对 `PRE_FW` 进行赋值或更新。
- **L86** EN: Assigns or updates `POST_FW`. | CN: 对 `POST_FW` 进行赋值或更新。
- **L87** EN: Assigns or updates `PEAK_FW`. | CN: 对 `PEAK_FW` 进行赋值或更新。
- **L88** EN: Assigns or updates `PRE_BW`. | CN: 对 `PRE_BW` 进行赋值或更新。
- **L89** EN: Assigns or updates `PRE_FW_AC`. | CN: 对 `PRE_FW_AC` 进行赋值或更新。
- **L90** EN: Assigns or updates `POST_FW_AC`. | CN: 对 `POST_FW_AC` 进行赋值或更新。
- **L91** EN: Assigns or updates `POST_BW`. | CN: 对 `POST_BW` 进行赋值或更新。
- **L92** EN: Assigns or updates `PEAK_BW`. | CN: 对 `PEAK_BW` 进行赋值或更新。
- **L93** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L94** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L95** EN: Defines class `_ModMemStats`. | CN: 定义类 `_ModMemStats`。
- **L96** EN: Starts the docstring for the class _ModMemStats. | CN: 开始定义 class _ModMemStats 的文档字符串。
- **L97** EN: Continues the docstring text for the class _ModMemStats. | CN: 继续补充 class _ModMemStats 的文档字符串内容。
- **L98** EN: Continues the docstring text for the class _ModMemStats. | CN: 继续补充 class _ModMemStats 的文档字符串内容。
- **L99** EN: Continues the docstring text for the class _ModMemStats. | CN: 继续补充 class _ModMemStats 的文档字符串内容。
- **L100** EN: Continues the docstring text for the class _ModMemStats. | CN: 继续补充 class _ModMemStats 的文档字符串内容。

### Lines 101-120 / 第 101-120 行

````python
    Attributes:
        mod_fqn (str): The fully qualified name of the module.
        parameter_mem (int): The memory usage of the parameters of the module.
        buffer_mem (int): The memory usage of the buffers of the module.
        input_mem (int): The memory usage of the inputs to the module.
        output_mem (int): The memory usage of the outputs from the module.
        snapshots (Dict[_ModState, Dict[torch.device, Dict[str, int]]]): A dictionary of memory snapshots
        of the module at different states defined by ``_ModState``.
    Note:
        The memory snapshot is stored as a dictionary - Dict[torch.device, Dict[str, int]], where each key is a device,
         and each value is another dictionary with keys as memory reference types defined by `_MemRefType` and
         values as the memory consumed in bytes.
    """

    def __init__(self, mod_fqn: str):
        self.mod_fqn = mod_fqn
        self.parameter_mem: int
        self.buffer_mem: int
        self.input_mem: int
        self.output_mem: int
````

- **L101** EN: Continues the docstring text for the class _ModMemStats. | CN: 继续补充 class _ModMemStats 的文档字符串内容。
- **L102** EN: Continues the docstring text for the class _ModMemStats. | CN: 继续补充 class _ModMemStats 的文档字符串内容。
- **L103** EN: Continues the docstring text for the class _ModMemStats. | CN: 继续补充 class _ModMemStats 的文档字符串内容。
- **L104** EN: Continues the docstring text for the class _ModMemStats. | CN: 继续补充 class _ModMemStats 的文档字符串内容。
- **L105** EN: Continues the docstring text for the class _ModMemStats. | CN: 继续补充 class _ModMemStats 的文档字符串内容。
- **L106** EN: Continues the docstring text for the class _ModMemStats. | CN: 继续补充 class _ModMemStats 的文档字符串内容。
- **L107** EN: Continues the docstring text for the class _ModMemStats. | CN: 继续补充 class _ModMemStats 的文档字符串内容。
- **L108** EN: Continues the docstring text for the class _ModMemStats. | CN: 继续补充 class _ModMemStats 的文档字符串内容。
- **L109** EN: Continues the docstring text for the class _ModMemStats. | CN: 继续补充 class _ModMemStats 的文档字符串内容。
- **L110** EN: Continues the docstring text for the class _ModMemStats. | CN: 继续补充 class _ModMemStats 的文档字符串内容。
- **L111** EN: Continues the docstring text for the class _ModMemStats. | CN: 继续补充 class _ModMemStats 的文档字符串内容。
- **L112** EN: Continues the docstring text for the class _ModMemStats. | CN: 继续补充 class _ModMemStats 的文档字符串内容。
- **L113** EN: Closes the docstring for the class _ModMemStats. | CN: 结束 class _ModMemStats 的文档字符串。
- **L114** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L115** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L116** EN: Assigns or updates `self.mod_fqn`. | CN: 对 `self.mod_fqn` 进行赋值或更新。
- **L117** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L118** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L119** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L120** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。

### Lines 121-140 / 第 121-140 行

````python
        self.local_peak: dict[torch.device, int] = {}
        self.snapshots: dict[_ModState, list[dict[torch.device, dict[str, int]]]] = {}


class _WeakRefInfo:
    """
    Manages memory statistics and device attributes for tensor storages.
    """

    def __init__(
        self, size: int, element_size: int, device: torch.device, reftype: _RefType
    ) -> None:
        """
        Initializes the ``_WeakRefInfo`` object with tensor storage properties.

        Args:
            size (int): The number of elements in the tensor storage.
            element_size (int): The size of each element in the tensor storage.
            device (torch.device): The device on which the tensor is allocated.
            reftype (_RefType): The reference type of the tensor.
````

- **L121** EN: Assigns or updates `self.local_peak`. | CN: 对 `self.local_peak` 进行赋值或更新。
- **L122** EN: Assigns or updates `self.snapshots`. | CN: 对 `self.snapshots` 进行赋值或更新。
- **L123** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L124** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L125** EN: Defines class `_WeakRefInfo`. | CN: 定义类 `_WeakRefInfo`。
- **L126** EN: Starts the docstring for the class _WeakRefInfo. | CN: 开始定义 class _WeakRefInfo 的文档字符串。
- **L127** EN: Continues the docstring text for the class _WeakRefInfo. | CN: 继续补充 class _WeakRefInfo 的文档字符串内容。
- **L128** EN: Closes the docstring for the class _WeakRefInfo. | CN: 结束 class _WeakRefInfo 的文档字符串。
- **L129** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L130** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L131** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L132** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。
- **L133** EN: Starts the docstring for the function __init__. | CN: 开始定义 function __init__ 的文档字符串。
- **L134** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L135** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L136** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L137** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L138** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L139** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。
- **L140** EN: Continues the docstring text for the function __init__. | CN: 继续补充 function __init__ 的文档字符串内容。

### Lines 141-160 / 第 141-160 行

````python
        """
        self.size = size
        self.element_size = element_size
        self.reftype = reftype
        self.device = device
        self.mem_consumed = self._calculate_mem_consumed()

    def _calculate_mem_consumed(self) -> int:
        """
        Calculates the memory consumed by the tensor storage, considering device-specific allocation rules.

        Returns:
            int: The memory consumed in bytes.
        """
        mem = self.size * self.element_size
        if self.device.type == "cuda":
            return math.ceil((mem) / _PYTORCH_MIN_ALLOCATE) * _PYTORCH_MIN_ALLOCATE
        return mem

    def update_mem_consumed(self, st: torch.UntypedStorage) -> int:
````

- **L141** EN: Closes the docstring for the function __init__. | CN: 结束 function __init__ 的文档字符串。
- **L142** EN: Assigns or updates `self.size`. | CN: 对 `self.size` 进行赋值或更新。
- **L143** EN: Assigns or updates `self.element_size`. | CN: 对 `self.element_size` 进行赋值或更新。
- **L144** EN: Assigns or updates `self.reftype`. | CN: 对 `self.reftype` 进行赋值或更新。
- **L145** EN: Assigns or updates `self.device`. | CN: 对 `self.device` 进行赋值或更新。
- **L146** EN: Assigns or updates `self.mem_consumed`. | CN: 对 `self.mem_consumed` 进行赋值或更新。
- **L147** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L148** EN: Defines function `_calculate_mem_consumed`. | CN: 定义函数 `_calculate_mem_consumed`。
- **L149** EN: Starts the docstring for the function _calculate_mem_consumed. | CN: 开始定义 function _calculate_mem_consumed 的文档字符串。
- **L150** EN: Continues the docstring text for the function _calculate_mem_consumed. | CN: 继续补充 function _calculate_mem_consumed 的文档字符串内容。
- **L151** EN: Continues the docstring text for the function _calculate_mem_consumed. | CN: 继续补充 function _calculate_mem_consumed 的文档字符串内容。
- **L152** EN: Continues the docstring text for the function _calculate_mem_consumed. | CN: 继续补充 function _calculate_mem_consumed 的文档字符串内容。
- **L153** EN: Continues the docstring text for the function _calculate_mem_consumed. | CN: 继续补充 function _calculate_mem_consumed 的文档字符串内容。
- **L154** EN: Closes the docstring for the function _calculate_mem_consumed. | CN: 结束 function _calculate_mem_consumed 的文档字符串。
- **L155** EN: Assigns or updates `mem`. | CN: 对 `mem` 进行赋值或更新。
- **L156** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L157** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L158** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L159** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L160** EN: Defines function `update_mem_consumed`. | CN: 定义函数 `update_mem_consumed`。

### Lines 161-180 / 第 161-180 行

````python
        """
        Updates and returns the memory consumed if the storage size has changed.

        Args:
            st (torch.UntypedStorage): The tensor storage to check for size updates.

        Returns:
            int: The updated memory consumed in bytes.
        """
        if st.size() != self.size:
            self.size = st.size()
            self.mem_consumed = self._calculate_mem_consumed()
        return self.mem_consumed

    @classmethod
    def create_winfo(
        cls,
        st: torch.UntypedStorage,
        device: torch.device,
        reftype: _RefType,
````

- **L161** EN: Starts the docstring for the function update_mem_consumed. | CN: 开始定义 function update_mem_consumed 的文档字符串。
- **L162** EN: Continues the docstring text for the function update_mem_consumed. | CN: 继续补充 function update_mem_consumed 的文档字符串内容。
- **L163** EN: Continues the docstring text for the function update_mem_consumed. | CN: 继续补充 function update_mem_consumed 的文档字符串内容。
- **L164** EN: Continues the docstring text for the function update_mem_consumed. | CN: 继续补充 function update_mem_consumed 的文档字符串内容。
- **L165** EN: Continues the docstring text for the function update_mem_consumed. | CN: 继续补充 function update_mem_consumed 的文档字符串内容。
- **L166** EN: Continues the docstring text for the function update_mem_consumed. | CN: 继续补充 function update_mem_consumed 的文档字符串内容。
- **L167** EN: Continues the docstring text for the function update_mem_consumed. | CN: 继续补充 function update_mem_consumed 的文档字符串内容。
- **L168** EN: Continues the docstring text for the function update_mem_consumed. | CN: 继续补充 function update_mem_consumed 的文档字符串内容。
- **L169** EN: Closes the docstring for the function update_mem_consumed. | CN: 结束 function update_mem_consumed 的文档字符串。
- **L170** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L171** EN: Assigns or updates `self.size`. | CN: 对 `self.size` 进行赋值或更新。
- **L172** EN: Assigns or updates `self.mem_consumed`. | CN: 对 `self.mem_consumed` 进行赋值或更新。
- **L173** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L174** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L175** EN: Applies decorator `classmethod` to the following definition. | CN: 将装饰器 `classmethod` 应用于后续定义。
- **L176** EN: Defines function `create_winfo`. | CN: 定义函数 `create_winfo`。
- **L177** EN: Continues the implementation inside function `create_winfo`. | CN: 继续说明函数 `create_winfo` 内部的实现。
- **L178** EN: Continues the implementation inside function `create_winfo`. | CN: 继续说明函数 `create_winfo` 内部的实现。
- **L179** EN: Continues the implementation inside function `create_winfo`. | CN: 继续说明函数 `create_winfo` 内部的实现。
- **L180** EN: Continues the implementation inside function `create_winfo`. | CN: 继续说明函数 `create_winfo` 内部的实现。

### Lines 181-200 / 第 181-200 行

````python
        callback: Callable[[Self, weakref.ref], Any] | None = None,
    ) -> tuple[Self, weakref.ref]:
        """
        Creates a new ``_WeakRefInfo`` instance and a weak reference to a ``torch.UntypedStorage`` object,
        optionally attaching a callback to the weak reference.

        Args:
            st (torch.UntypedStorage): The storage object for which to create the weak reference info.
            device (torch.device): The device associated with the storage object.
            reftype (_RefType): The type of reference, used to categorize the storage.
            callback (Optional[Callable[[Self, weakref.ref]]]): A callback function that is called when
                the storage object is about to be finalized (garbage collected). The callback function
                should accept two arguments: the ``_WeakRefInfo`` instance and the weak reference to the storage.
        Returns:
            Tuple[Self, weakref.ref]: A tuple containing the newly created ``_WeakRefInfo`` instance and the
            weak reference to the storage object. The weak reference may have an attached callback if provided.
        """

        winfo = cls(st.size(), st.element_size(), device, reftype)
        w_st = weakref.ref(st, partial(callback, winfo) if callback else None)
````

- **L181** EN: Assigns or updates `callback`. | CN: 对 `callback` 进行赋值或更新。
- **L182** EN: Continues the implementation inside function `create_winfo`. | CN: 继续说明函数 `create_winfo` 内部的实现。
- **L183** EN: Starts the docstring for the function create_winfo. | CN: 开始定义 function create_winfo 的文档字符串。
- **L184** EN: Continues the docstring text for the function create_winfo. | CN: 继续补充 function create_winfo 的文档字符串内容。
- **L185** EN: Continues the docstring text for the function create_winfo. | CN: 继续补充 function create_winfo 的文档字符串内容。
- **L186** EN: Continues the docstring text for the function create_winfo. | CN: 继续补充 function create_winfo 的文档字符串内容。
- **L187** EN: Continues the docstring text for the function create_winfo. | CN: 继续补充 function create_winfo 的文档字符串内容。
- **L188** EN: Continues the docstring text for the function create_winfo. | CN: 继续补充 function create_winfo 的文档字符串内容。
- **L189** EN: Continues the docstring text for the function create_winfo. | CN: 继续补充 function create_winfo 的文档字符串内容。
- **L190** EN: Continues the docstring text for the function create_winfo. | CN: 继续补充 function create_winfo 的文档字符串内容。
- **L191** EN: Continues the docstring text for the function create_winfo. | CN: 继续补充 function create_winfo 的文档字符串内容。
- **L192** EN: Continues the docstring text for the function create_winfo. | CN: 继续补充 function create_winfo 的文档字符串内容。
- **L193** EN: Continues the docstring text for the function create_winfo. | CN: 继续补充 function create_winfo 的文档字符串内容。
- **L194** EN: Continues the docstring text for the function create_winfo. | CN: 继续补充 function create_winfo 的文档字符串内容。
- **L195** EN: Continues the docstring text for the function create_winfo. | CN: 继续补充 function create_winfo 的文档字符串内容。
- **L196** EN: Continues the docstring text for the function create_winfo. | CN: 继续补充 function create_winfo 的文档字符串内容。
- **L197** EN: Closes the docstring for the function create_winfo. | CN: 结束 function create_winfo 的文档字符串。
- **L198** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L199** EN: Assigns or updates `winfo`. | CN: 对 `winfo` 进行赋值或更新。
- **L200** EN: Assigns or updates `w_st`. | CN: 对 `w_st` 进行赋值或更新。

### Lines 201-220 / 第 201-220 行

````python
        return winfo, w_st


def _get_mem_divisor(units: str) -> int:
    unit_dict = {"B": 1, "KiB": 2**10, "MiB": 2**20, "GiB": 2**30}
    if units in unit_dict:
        return unit_dict[units]
    else:
        raise ValueError(
            f"Unsupported unit: {units}. Supported units are: {', '.join(unit_dict.keys())}"
        )


def _rounding_fn(value: int, divisor: int, precision: int) -> float | int:
    return value if divisor == 1 else round(value / divisor, precision)


def _print_snapshot(snapshot: dict[torch.device, dict[str, int]], units: str) -> None:
    if len(snapshot) == 0:
        print("No memory tracked.")
````

- **L201** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L202** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L203** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L204** EN: Defines function `_get_mem_divisor`. | CN: 定义函数 `_get_mem_divisor`。
- **L205** EN: Assigns or updates `unit_dict`. | CN: 对 `unit_dict` 进行赋值或更新。
- **L206** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L207** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L208** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L209** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L210** EN: Continues the implementation inside function `_get_mem_divisor`. | CN: 继续说明函数 `_get_mem_divisor` 内部的实现。
- **L211** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L212** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L213** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L214** EN: Defines function `_rounding_fn`. | CN: 定义函数 `_rounding_fn`。
- **L215** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L216** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L217** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L218** EN: Defines function `_print_snapshot`. | CN: 定义函数 `_print_snapshot`。
- **L219** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L220** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。

### Lines 221-240 / 第 221-240 行

````python
        return
    divisor = _get_mem_divisor(units)
    for dev, dev_snap in snapshot.items():
        if _rounding_fn(dev_snap[_TOTAL_KEY], divisor, 2) <= 0:
            continue
        print(
            f"Device: {dev}",
            *(
                f"\t{k.value}: {_rounding_fn(v, divisor, 2)} {units}"
                if isinstance(k, _RefType)
                else f"\t{k}: {_rounding_fn(v, divisor, 2)} {units}"
                for k, v in dev_snap.items()
            ),
            sep="\n",
        )


def _print_snapshot_tabular(
    snapshot: dict[torch.device, dict[str, int]], units: str
) -> None:
````

- **L221** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L222** EN: Assigns or updates `divisor`. | CN: 对 `divisor` 进行赋值或更新。
- **L223** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L224** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L225** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L226** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L227** EN: Continues the implementation inside function `_print_snapshot`. | CN: 继续说明函数 `_print_snapshot` 内部的实现。
- **L228** EN: Continues the implementation inside function `_print_snapshot`. | CN: 继续说明函数 `_print_snapshot` 内部的实现。
- **L229** EN: Continues the implementation inside function `_print_snapshot`. | CN: 继续说明函数 `_print_snapshot` 内部的实现。
- **L230** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L231** EN: Continues the implementation inside function `_print_snapshot`. | CN: 继续说明函数 `_print_snapshot` 内部的实现。
- **L232** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L233** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L234** EN: Assigns or updates `sep`. | CN: 对 `sep` 进行赋值或更新。
- **L235** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L236** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L237** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L238** EN: Defines function `_print_snapshot_tabular`. | CN: 定义函数 `_print_snapshot_tabular`。
- **L239** EN: Continues the implementation inside function `_print_snapshot_tabular`. | CN: 继续说明函数 `_print_snapshot_tabular` 内部的实现。
- **L240** EN: Continues the implementation inside function `_print_snapshot_tabular`. | CN: 继续说明函数 `_print_snapshot_tabular` 内部的实现。

### Lines 241-260 / 第 241-260 行

````python
    if len(snapshot) == 0:
        print("No memory tracked.")
        return
    try:
        from tabulate import tabulate
    except ImportError as err:
        raise ImportError(
            "Please install tabulate to use the tabulate option."
        ) from err
    divisor = _get_mem_divisor(units)
    table_data = []
    key_list = list(next(iter(snapshot.values())).keys())
    headers = ["Device"] + [
        f"{key.value}" if isinstance(key, _RefType) else f"{key}" for key in key_list
    ]

    for dev, dev_snap in snapshot.items():
        if _rounding_fn(dev_snap[_TOTAL_KEY], divisor, 2) <= 0:
            continue
        row = [str(dev)]
````

- **L241** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L242** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L243** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L244** EN: Starts a protected block for exception handling. | CN: 开始一个受保护的异常处理代码块。
- **L245** EN: Imports selected names from `tabulate`. | CN: 从 `tabulate` 导入指定名称。
- **L246** EN: Handles a matching exception path. | CN: 处理匹配到的异常路径。
- **L247** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L248** EN: Continues the implementation inside function `_print_snapshot_tabular`. | CN: 继续说明函数 `_print_snapshot_tabular` 内部的实现。
- **L249** EN: Continues the implementation inside function `_print_snapshot_tabular`. | CN: 继续说明函数 `_print_snapshot_tabular` 内部的实现。
- **L250** EN: Assigns or updates `divisor`. | CN: 对 `divisor` 进行赋值或更新。
- **L251** EN: Assigns or updates `table_data`. | CN: 对 `table_data` 进行赋值或更新。
- **L252** EN: Assigns or updates `key_list`. | CN: 对 `key_list` 进行赋值或更新。
- **L253** EN: Assigns or updates `headers`. | CN: 对 `headers` 进行赋值或更新。
- **L254** EN: Continues the implementation inside function `_print_snapshot_tabular`. | CN: 继续说明函数 `_print_snapshot_tabular` 内部的实现。
- **L255** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L256** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L257** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L258** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L259** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L260** EN: Assigns or updates `row`. | CN: 对 `row` 进行赋值或更新。

### Lines 261-280 / 第 261-280 行

````python
        row.extend(f"{_rounding_fn(v, divisor, 2)} {units}" for v in dev_snap.values())
        table_data.append(row)
    print(tabulate(table_data, headers=headers, tablefmt="rst"))


def _print_state_snapshots(
    snapshots: dict[_State, list[dict[torch.device, dict[str, int]]]], units: str
) -> None:
    for state, snapshot_list in snapshots.items():
        print(f"{state.value}")
        for i, snapshot in enumerate(snapshot_list):
            print(f"# {i + 1}:")
            _print_snapshot(snapshot, units)
    print()


def _print_state_snapshots_tabular(
    snapshots: dict[_State, list[dict[torch.device, dict[str, int]]]], units: str
) -> None:
    try:
````

- **L261** EN: Calls `row.extend` as part of the current workflow. | CN: 在当前流程中调用 `row.extend`。
- **L262** EN: Calls `table_data.append` as part of the current workflow. | CN: 在当前流程中调用 `table_data.append`。
- **L263** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L264** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L265** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L266** EN: Defines function `_print_state_snapshots`. | CN: 定义函数 `_print_state_snapshots`。
- **L267** EN: Continues the implementation inside function `_print_state_snapshots`. | CN: 继续说明函数 `_print_state_snapshots` 内部的实现。
- **L268** EN: Continues the implementation inside function `_print_state_snapshots`. | CN: 继续说明函数 `_print_state_snapshots` 内部的实现。
- **L269** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L270** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L271** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L272** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L273** EN: Calls `_print_snapshot` as part of the current workflow. | CN: 在当前流程中调用 `_print_snapshot`。
- **L274** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L275** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L276** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L277** EN: Defines function `_print_state_snapshots_tabular`. | CN: 定义函数 `_print_state_snapshots_tabular`。
- **L278** EN: Continues the implementation inside function `_print_state_snapshots_tabular`. | CN: 继续说明函数 `_print_state_snapshots_tabular` 内部的实现。
- **L279** EN: Continues the implementation inside function `_print_state_snapshots_tabular`. | CN: 继续说明函数 `_print_state_snapshots_tabular` 内部的实现。
- **L280** EN: Starts a protected block for exception handling. | CN: 开始一个受保护的异常处理代码块。

### Lines 281-300 / 第 281-300 行

````python
        from tabulate import tabulate
    except ImportError as err:
        raise ImportError(
            "Please install tabulate to use the tabulate option."
        ) from err

    table_data = []
    last_state_call = None
    divisor = _get_mem_divisor(units)
    for state, snapshot_list in snapshots.items():
        for i, snapshot in enumerate(snapshot_list):
            state_call = f"{state.value} # {i + 1}"
            for dev, dev_snap in snapshot.items():
                if _rounding_fn(dev_snap[_TOTAL_KEY], divisor, 2) <= 0:
                    continue
                row = {
                    "State & Call": (
                        state_call if state_call != last_state_call else ""
                    ),
                    "Device": str(dev),
````

- **L281** EN: Imports selected names from `tabulate`. | CN: 从 `tabulate` 导入指定名称。
- **L282** EN: Handles a matching exception path. | CN: 处理匹配到的异常路径。
- **L283** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L284** EN: Continues the implementation inside function `_print_state_snapshots_tabular`. | CN: 继续说明函数 `_print_state_snapshots_tabular` 内部的实现。
- **L285** EN: Continues the implementation inside function `_print_state_snapshots_tabular`. | CN: 继续说明函数 `_print_state_snapshots_tabular` 内部的实现。
- **L286** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L287** EN: Assigns or updates `table_data`. | CN: 对 `table_data` 进行赋值或更新。
- **L288** EN: Assigns or updates `last_state_call`. | CN: 对 `last_state_call` 进行赋值或更新。
- **L289** EN: Assigns or updates `divisor`. | CN: 对 `divisor` 进行赋值或更新。
- **L290** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L291** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L292** EN: Assigns or updates `state_call`. | CN: 对 `state_call` 进行赋值或更新。
- **L293** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L294** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L295** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L296** EN: Assigns or updates `row`. | CN: 对 `row` 进行赋值或更新。
- **L297** EN: Continues the implementation inside function `_print_state_snapshots_tabular`. | CN: 继续说明函数 `_print_state_snapshots_tabular` 内部的实现。
- **L298** EN: Continues the implementation inside function `_print_state_snapshots_tabular`. | CN: 继续说明函数 `_print_state_snapshots_tabular` 内部的实现。
- **L299** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L300** EN: Continues the implementation inside function `_print_state_snapshots_tabular`. | CN: 继续说明函数 `_print_state_snapshots_tabular` 内部的实现。

### Lines 301-320 / 第 301-320 行

````python
                }
                last_state_call = state_call
                for k, v in dev_snap.items():
                    row[f"{k.value}" if isinstance(k, _RefType) else f"{k}"] = (
                        f"{_rounding_fn(v, divisor, 2)} {units}"
                    )
                table_data.append(row)
    print(tabulate(table_data, headers="keys", tablefmt="rst"))


class _UpdateType(Enum):
    # These are used for tracking updates to the continuouly maintained memory snapshot.
    # ADD - When a new tensor storage is tracked
    # DEL - When a tensor storage is about to be finalized (garbage collected).
    # REF - When a tensor reference is updated, for instance, the gradients are marked as
    #       generic backward reference types until the grad_hook categorizes them as gradients.
    # SIZE - When a tensor's storage is resized.
    ADD = auto()
    DEL = auto()
    REF = auto()
````

- **L301** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L302** EN: Assigns or updates `last_state_call`. | CN: 对 `last_state_call` 进行赋值或更新。
- **L303** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L304** EN: Continues the implementation inside function `_print_state_snapshots_tabular`. | CN: 继续说明函数 `_print_state_snapshots_tabular` 内部的实现。
- **L305** EN: Continues the implementation inside function `_print_state_snapshots_tabular`. | CN: 继续说明函数 `_print_state_snapshots_tabular` 内部的实现。
- **L306** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L307** EN: Calls `table_data.append` as part of the current workflow. | CN: 在当前流程中调用 `table_data.append`。
- **L308** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L309** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L310** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L311** EN: Defines class `_UpdateType`. | CN: 定义类 `_UpdateType`。
- **L312** EN: Keeps the inline comment or directive: These are used for tracking updates to the continuouly maintained memory snapsho | CN: 保留这一行注释或指令：These are used for tracking updates to the continuouly maintained memory snapsho
- **L313** EN: Keeps the inline comment or directive: ADD - When a new tensor storage is tracked | CN: 保留这一行注释或指令：ADD - When a new tensor storage is tracked
- **L314** EN: Keeps the inline comment or directive: DEL - When a tensor storage is about to be finalized (garbage collected). | CN: 保留这一行注释或指令：DEL - When a tensor storage is about to be finalized (garbage collected).
- **L315** EN: Keeps the inline comment or directive: REF - When a tensor reference is updated, for instance, the gradients are marked | CN: 保留这一行注释或指令：REF - When a tensor reference is updated, for instance, the gradients are marked
- **L316** EN: Keeps the inline comment or directive: generic backward reference types until the grad_hook categorizes them as gradien | CN: 保留这一行注释或指令：generic backward reference types until the grad_hook categorizes them as gradien
- **L317** EN: Keeps the inline comment or directive: SIZE - When a tensor's storage is resized. | CN: 保留这一行注释或指令：SIZE - When a tensor's storage is resized.
- **L318** EN: Assigns or updates `ADD`. | CN: 对 `ADD` 进行赋值或更新。
- **L319** EN: Assigns or updates `DEL`. | CN: 对 `DEL` 进行赋值或更新。
- **L320** EN: Assigns or updates `REF`. | CN: 对 `REF` 进行赋值或更新。

### Lines 321-340 / 第 321-340 行

````python
    SIZE = auto()


class MemTracker(TorchDispatchMode):
    """
    A TorchDispatchMode to track, categorize and attribute the tensor memory created or accessed within its context.

    It categorizes the tracked tensors as parameters, buffers, activations, gradients, temporary memory and optimizer states
    as defined by ``_MemRefType`` within its context. It captures memory `snapshots` for the modules, called within its context,
    at various states defined by ``_ModState``.

    Attributes:
        memory_tracking: A weakref key dictionary to store the memory statistics of each module. Each key
        is a reference to a module, and each value is a ``_ModMemStats`` object that stores the memory
        statistics of the module.

    Note:
        The MemTracker should be used as a context manager. The modules, optimizers, and any other tensors created within
        the context of MemTracker will be tracked by default. Any tensors or stateful objects such as modules, optimizers etc.
        that need to be tracked but are created outside the MemTracker should be registered using the `track_external` method.
````

- **L321** EN: Assigns or updates `SIZE`. | CN: 对 `SIZE` 进行赋值或更新。
- **L322** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L323** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L324** EN: Defines class `MemTracker`. | CN: 定义类 `MemTracker`。
- **L325** EN: Starts the docstring for the class MemTracker. | CN: 开始定义 class MemTracker 的文档字符串。
- **L326** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L327** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L328** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L329** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L330** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L331** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L332** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L333** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L334** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L335** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L336** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L337** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L338** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L339** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L340** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。

### Lines 341-360 / 第 341-360 行

````python
        The `track_external` method should be called before the MemTracker is used. Any tensors created outside the ``MemTracker``
        and not supplied to the `track_external` method will not be tracked by the ``MemTracker``.

    Example usage:

        .. code-block:: python

            module = ...
            optimizer = ...
            inp = ...
            mem_tracker = MemTracker()
            mem_tracker.track_external(module, optimizer, inp)
            with mem_tracker as mt:
                loss = module(inp)
                print("After Forward:")
                mt.display_snapshot("current")
                loss.backward()
                optimizer.step()
                optimizer.zero_grad()
            mt.display_snapshot("peak")
````

- **L341** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L342** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L343** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L344** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L345** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L346** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L347** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L348** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L349** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L350** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L351** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L352** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L353** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L354** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L355** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L356** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L357** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L358** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L359** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L360** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。

### Lines 361-380 / 第 361-380 行

````python
            mt.display_modulewise_snapshots(depth=3, units="MiB")

    Known Limitations:
        - The ``MemTracker`` does not track memory for tensors that bypass the ``TorchDispatchMode`` ex. under ``no_dispatch``.
        - Resizing tensor storages directly by using non-Tensor methods other than using ``torch.Untyped_Storage.resize_``
          is not tracked. File a Github issue if you have use-cases for this.
        - If the tensors are not traceable or wrappable subclasses of ``torch.Tensor``, then the tracker does not know how to
            track their storages. File a Github issue if you have use-cases for this.
        - During AC in the backward pass there might be misattribution between activation and temp memory, but the peak memory
          will be tracked accurately. This will be fixed in the next update by hooking intricately with ``torch.uitls.checkpoint``.
    """

    def __init__(self) -> None:
        self.memory_tracking = WeakIdKeyDictionary()
        self._curr_mem_snap: dict[torch.device, dict[str, int]] = {}
        self._peak_mem: dict[torch.device, int] = {}
        self._peak_mem_snap: dict[torch.device, dict[str, int]] = {}
        self._param_to_grad_hook_handles = WeakIdKeyDictionary()
        self._optimizer_hook_handles: tuple[RemovableHandle, RemovableHandle] | None = (
            None
````

- **L361** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L362** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L363** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L364** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L365** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L366** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L367** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L368** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L369** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L370** EN: Continues the docstring text for the class MemTracker. | CN: 继续补充 class MemTracker 的文档字符串内容。
- **L371** EN: Closes the docstring for the class MemTracker. | CN: 结束 class MemTracker 的文档字符串。
- **L372** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L373** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L374** EN: Assigns or updates `self.memory_tracking`. | CN: 对 `self.memory_tracking` 进行赋值或更新。
- **L375** EN: Assigns or updates `self._curr_mem_snap`. | CN: 对 `self._curr_mem_snap` 进行赋值或更新。
- **L376** EN: Assigns or updates `self._peak_mem`. | CN: 对 `self._peak_mem` 进行赋值或更新。
- **L377** EN: Assigns or updates `self._peak_mem_snap`. | CN: 对 `self._peak_mem_snap` 进行赋值或更新。
- **L378** EN: Assigns or updates `self._param_to_grad_hook_handles`. | CN: 对 `self._param_to_grad_hook_handles` 进行赋值或更新。
- **L379** EN: Assigns or updates `self._optimizer_hook_handles`. | CN: 对 `self._optimizer_hook_handles` 进行赋值或更新。
- **L380** EN: Continues the implementation inside function `__init__`. | CN: 继续说明函数 `__init__` 内部的实现。

### Lines 381-400 / 第 381-400 行

````python
        )
        # Dictionary to store the ``_WeakRefInfo`` instances corresponding to each tensor's storage.
        self._WINFO = WeakIdKeyDictionary()
        self._mod_tracker = ModTracker()
        # This is a general memory tracker which can be used with any ``_RefType`` subclass
        self._ref_class: type[_RefType] = _MemRefType
        # Flags to track if we are in the AC region or optimizer step region
        self._in_opt: bool = False
        self._in_ac: bool = False
        # Weak references to the topmost AC module currently active
        self._ac_mod: weakref.ref | None = None
        self._orig_resize = torch.UntypedStorage.resize_
        self._depth = 0

    def _update_snap(
        self,
        u_type: _UpdateType,
        winfo: _WeakRefInfo,
        old_mem_consumed: int | None = None,
        old_reftype: _RefType | None = None,
````

- **L381** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L382** EN: Keeps the inline comment or directive: Dictionary to store the ``_WeakRefInfo`` instances corresponding to each tensor' | CN: 保留这一行注释或指令：Dictionary to store the ``_WeakRefInfo`` instances corresponding to each tensor'
- **L383** EN: Assigns or updates `self._WINFO`. | CN: 对 `self._WINFO` 进行赋值或更新。
- **L384** EN: Assigns or updates `self._mod_tracker`. | CN: 对 `self._mod_tracker` 进行赋值或更新。
- **L385** EN: Keeps the inline comment or directive: This is a general memory tracker which can be used with any ``_RefType`` subclas | CN: 保留这一行注释或指令：This is a general memory tracker which can be used with any ``_RefType`` subclas
- **L386** EN: Assigns or updates `self._ref_class`. | CN: 对 `self._ref_class` 进行赋值或更新。
- **L387** EN: Keeps the inline comment or directive: Flags to track if we are in the AC region or optimizer step region | CN: 保留这一行注释或指令：Flags to track if we are in the AC region or optimizer step region
- **L388** EN: Assigns or updates `self._in_opt`. | CN: 对 `self._in_opt` 进行赋值或更新。
- **L389** EN: Assigns or updates `self._in_ac`. | CN: 对 `self._in_ac` 进行赋值或更新。
- **L390** EN: Keeps the inline comment or directive: Weak references to the topmost AC module currently active | CN: 保留这一行注释或指令：Weak references to the topmost AC module currently active
- **L391** EN: Assigns or updates `self._ac_mod`. | CN: 对 `self._ac_mod` 进行赋值或更新。
- **L392** EN: Assigns or updates `self._orig_resize`. | CN: 对 `self._orig_resize` 进行赋值或更新。
- **L393** EN: Assigns or updates `self._depth`. | CN: 对 `self._depth` 进行赋值或更新。
- **L394** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L395** EN: Defines function `_update_snap`. | CN: 定义函数 `_update_snap`。
- **L396** EN: Continues the implementation inside function `_update_snap`. | CN: 继续说明函数 `_update_snap` 内部的实现。
- **L397** EN: Continues the implementation inside function `_update_snap`. | CN: 继续说明函数 `_update_snap` 内部的实现。
- **L398** EN: Continues the implementation inside function `_update_snap`. | CN: 继续说明函数 `_update_snap` 内部的实现。
- **L399** EN: Assigns or updates `old_mem_consumed`. | CN: 对 `old_mem_consumed` 进行赋值或更新。
- **L400** EN: Assigns or updates `old_reftype`. | CN: 对 `old_reftype` 进行赋值或更新。

### Lines 401-420 / 第 401-420 行

````python
    ) -> None:
        # Initialize a flag to track if the total memory might drop to zero after updates.
        maybe_zero = False
        # Ensure the device entry exists in the current memory snapshot, initializing if necessary.
        # pyrefly: ignore [no-matching-overload]
        dev_snap = self._curr_mem_snap.setdefault(
            winfo.device, dict.fromkeys(self._ref_class, 0)
        )
        dev_snap.setdefault(_TOTAL_KEY, 0)
        # Handle different types of updates based on the update type (`u_type`).
        if u_type == _UpdateType.ADD:
            # Increase the memory consumed for the specific reference type and update the total.
            dev_snap[winfo.reftype] += winfo.mem_consumed
            dev_snap[_TOTAL_KEY] += winfo.mem_consumed
        elif u_type == _UpdateType.DEL:
            # Decrease the memory consumed for the specific reference type and reduce the total.
            dev_snap[winfo.reftype] -= winfo.mem_consumed
            dev_snap[_TOTAL_KEY] -= winfo.mem_consumed
            maybe_zero = True
        elif u_type == _UpdateType.REF:
````

- **L401** EN: Continues the implementation inside function `_update_snap`. | CN: 继续说明函数 `_update_snap` 内部的实现。
- **L402** EN: Keeps the inline comment or directive: Initialize a flag to track if the total memory might drop to zero after updates. | CN: 保留这一行注释或指令：Initialize a flag to track if the total memory might drop to zero after updates.
- **L403** EN: Assigns or updates `maybe_zero`. | CN: 对 `maybe_zero` 进行赋值或更新。
- **L404** EN: Keeps the inline comment or directive: Ensure the device entry exists in the current memory snapshot, initializing if n | CN: 保留这一行注释或指令：Ensure the device entry exists in the current memory snapshot, initializing if n
- **L405** EN: Keeps the inline comment or directive: pyrefly: ignore [no-matching-overload] | CN: 保留这一行注释或指令：pyrefly: ignore [no-matching-overload]
- **L406** EN: Assigns or updates `dev_snap`. | CN: 对 `dev_snap` 进行赋值或更新。
- **L407** EN: Continues the implementation inside function `_update_snap`. | CN: 继续说明函数 `_update_snap` 内部的实现。
- **L408** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L409** EN: Calls `dev_snap.setdefault` as part of the current workflow. | CN: 在当前流程中调用 `dev_snap.setdefault`。
- **L410** EN: Keeps the inline comment or directive: Handle different types of updates based on the update type (`u_type`). | CN: 保留这一行注释或指令：Handle different types of updates based on the update type (`u_type`).
- **L411** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L412** EN: Keeps the inline comment or directive: Increase the memory consumed for the specific reference type and update the tota | CN: 保留这一行注释或指令：Increase the memory consumed for the specific reference type and update the tota
- **L413** EN: Continues the implementation inside function `_update_snap`. | CN: 继续说明函数 `_update_snap` 内部的实现。
- **L414** EN: Continues the implementation inside function `_update_snap`. | CN: 继续说明函数 `_update_snap` 内部的实现。
- **L415** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L416** EN: Keeps the inline comment or directive: Decrease the memory consumed for the specific reference type and reduce the tota | CN: 保留这一行注释或指令：Decrease the memory consumed for the specific reference type and reduce the tota
- **L417** EN: Continues the implementation inside function `_update_snap`. | CN: 继续说明函数 `_update_snap` 内部的实现。
- **L418** EN: Continues the implementation inside function `_update_snap`. | CN: 继续说明函数 `_update_snap` 内部的实现。
- **L419** EN: Assigns or updates `maybe_zero`. | CN: 对 `maybe_zero` 进行赋值或更新。
- **L420** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。

### Lines 421-440 / 第 421-440 行

````python
            if old_reftype is None:
                raise AssertionError
            # Adjust memory consumption between two reference types within the same device.
            dev_snap[old_reftype] -= winfo.mem_consumed
            dev_snap[winfo.reftype] += winfo.mem_consumed
        elif u_type == _UpdateType.SIZE:
            if old_mem_consumed is None:
                raise AssertionError
            # Adjust the memory consumed for a reference type due to a change in size.
            change = winfo.mem_consumed - old_mem_consumed
            dev_snap[winfo.reftype] += change
            dev_snap[_TOTAL_KEY] += change
            maybe_zero = True
        else:
            raise ValueError(f"Invalid update type: {u_type}")
        # Check if the total memory for the device has dropped to zero.
        if maybe_zero:
            if self._curr_mem_snap[winfo.device][_TOTAL_KEY] == 0:
                # Remove the device entry from the memory snapshot if the total memory is zero.
                del self._curr_mem_snap[winfo.device]
````

- **L421** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L422** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L423** EN: Keeps the inline comment or directive: Adjust memory consumption between two reference types within the same device. | CN: 保留这一行注释或指令：Adjust memory consumption between two reference types within the same device.
- **L424** EN: Continues the implementation inside function `_update_snap`. | CN: 继续说明函数 `_update_snap` 内部的实现。
- **L425** EN: Continues the implementation inside function `_update_snap`. | CN: 继续说明函数 `_update_snap` 内部的实现。
- **L426** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L427** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L428** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L429** EN: Keeps the inline comment or directive: Adjust the memory consumed for a reference type due to a change in size. | CN: 保留这一行注释或指令：Adjust the memory consumed for a reference type due to a change in size.
- **L430** EN: Assigns or updates `change`. | CN: 对 `change` 进行赋值或更新。
- **L431** EN: Continues the implementation inside function `_update_snap`. | CN: 继续说明函数 `_update_snap` 内部的实现。
- **L432** EN: Continues the implementation inside function `_update_snap`. | CN: 继续说明函数 `_update_snap` 内部的实现。
- **L433** EN: Assigns or updates `maybe_zero`. | CN: 对 `maybe_zero` 进行赋值或更新。
- **L434** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L435** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L436** EN: Keeps the inline comment or directive: Check if the total memory for the device has dropped to zero. | CN: 保留这一行注释或指令：Check if the total memory for the device has dropped to zero.
- **L437** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L438** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L439** EN: Keeps the inline comment or directive: Remove the device entry from the memory snapshot if the total memory is zero. | CN: 保留这一行注释或指令：Remove the device entry from the memory snapshot if the total memory is zero.
- **L440** EN: Continues the implementation inside function `_update_snap`. | CN: 继续说明函数 `_update_snap` 内部的实现。

### Lines 441-460 / 第 441-460 行

````python

    def _update_and_maybe_create_winfos(
        self,
        t: torch.Tensor,
        reftype: _RefType,
        update_existing: bool = False,
    ) -> set[_WeakRefInfo]:
        sts = get_untyped_storages(t)
        winfos = set()
        for st in sts:
            # Attempt to retrieve existing ``_WeakRefInfo`` and its weak reference from the tracking dictionary.
            winfo, _ = self._WINFO.get(st, (None, None))
            if winfo is not None:
                # If ``_WeakRefInfo`` exists, check if the reference type needs to be updated.
                old_reftype = winfo.reftype
                if old_reftype != reftype:
                    # Update the reference type and apply changes via ``_update_snap``.
                    winfo.reftype = reftype
                    self._update_snap(_UpdateType.REF, winfo, old_reftype=old_reftype)
                winfos.add(winfo)
````

- **L441** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L442** EN: Defines function `_update_and_maybe_create_winfos`. | CN: 定义函数 `_update_and_maybe_create_winfos`。
- **L443** EN: Continues the implementation inside function `_update_and_maybe_create_winfos`. | CN: 继续说明函数 `_update_and_maybe_create_winfos` 内部的实现。
- **L444** EN: Continues the implementation inside function `_update_and_maybe_create_winfos`. | CN: 继续说明函数 `_update_and_maybe_create_winfos` 内部的实现。
- **L445** EN: Continues the implementation inside function `_update_and_maybe_create_winfos`. | CN: 继续说明函数 `_update_and_maybe_create_winfos` 内部的实现。
- **L446** EN: Assigns or updates `update_existing`. | CN: 对 `update_existing` 进行赋值或更新。
- **L447** EN: Continues the implementation inside function `_update_and_maybe_create_winfos`. | CN: 继续说明函数 `_update_and_maybe_create_winfos` 内部的实现。
- **L448** EN: Assigns or updates `sts`. | CN: 对 `sts` 进行赋值或更新。
- **L449** EN: Assigns or updates `winfos`. | CN: 对 `winfos` 进行赋值或更新。
- **L450** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L451** EN: Keeps the inline comment or directive: Attempt to retrieve existing ``_WeakRefInfo`` and its weak reference from the tr | CN: 保留这一行注释或指令：Attempt to retrieve existing ``_WeakRefInfo`` and its weak reference from the tr
- **L452** EN: Assigns or updates `winfo, _`. | CN: 对 `winfo, _` 进行赋值或更新。
- **L453** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L454** EN: Keeps the inline comment or directive: If ``_WeakRefInfo`` exists, check if the reference type needs to be updated. | CN: 保留这一行注释或指令：If ``_WeakRefInfo`` exists, check if the reference type needs to be updated.
- **L455** EN: Assigns or updates `old_reftype`. | CN: 对 `old_reftype` 进行赋值或更新。
- **L456** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L457** EN: Keeps the inline comment or directive: Update the reference type and apply changes via ``_update_snap``. | CN: 保留这一行注释或指令：Update the reference type and apply changes via ``_update_snap``.
- **L458** EN: Assigns or updates `winfo.reftype`. | CN: 对 `winfo.reftype` 进行赋值或更新。
- **L459** EN: Calls `self._update_snap` as part of the current workflow. | CN: 在当前流程中调用 `self._update_snap`。
- **L460** EN: Calls `winfos.add` as part of the current workflow. | CN: 在当前流程中调用 `winfos.add`。

### Lines 461-480 / 第 461-480 行

````python
            elif update_existing:
                # If no existing ``_WeakRefInfo`` is found and update_existing is True, raise an error.
                raise KeyError("No existing winfo found")
            else:
                # If no existing _WeakRefInfo is found and update_existing is False, create a new ``_WeakRefInfo``.
                winfo, w_st = _WeakRefInfo.create_winfo(
                    st, t.device, reftype, self._delete_callback
                )
                # Store the new ``_WeakRefInfo`` and its weak reference in the tracking dictionary.
                self._WINFO[st] = (winfo, w_st)
                # Update the snapshot for the newly added ``_WeakRefInfo``.
                if winfo.mem_consumed > 0:
                    self._update_snap(_UpdateType.ADD, winfo)
                winfos.add(winfo)
        return winfos

    def _delete_callback(self, winfo: _WeakRefInfo, w_st: weakref.ref) -> None:
        # Callback to be called when the storage object corresponding to the  ``_WeakRefInfo``
        # instance is about to be finalized.
        if winfo.mem_consumed > 0:
````

- **L461** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L462** EN: Keeps the inline comment or directive: If no existing ``_WeakRefInfo`` is found and update_existing is True, raise an e | CN: 保留这一行注释或指令：If no existing ``_WeakRefInfo`` is found and update_existing is True, raise an e
- **L463** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L464** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L465** EN: Keeps the inline comment or directive: If no existing _WeakRefInfo is found and update_existing is False, create a new  | CN: 保留这一行注释或指令：If no existing _WeakRefInfo is found and update_existing is False, create a new 
- **L466** EN: Assigns or updates `winfo, w_st`. | CN: 对 `winfo, w_st` 进行赋值或更新。
- **L467** EN: Continues the implementation inside function `_update_and_maybe_create_winfos`. | CN: 继续说明函数 `_update_and_maybe_create_winfos` 内部的实现。
- **L468** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L469** EN: Keeps the inline comment or directive: Store the new ``_WeakRefInfo`` and its weak reference in the tracking dictionary | CN: 保留这一行注释或指令：Store the new ``_WeakRefInfo`` and its weak reference in the tracking dictionary
- **L470** EN: Assigns or updates `self._WINFO[st]`. | CN: 对 `self._WINFO[st]` 进行赋值或更新。
- **L471** EN: Keeps the inline comment or directive: Update the snapshot for the newly added ``_WeakRefInfo``. | CN: 保留这一行注释或指令：Update the snapshot for the newly added ``_WeakRefInfo``.
- **L472** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L473** EN: Calls `self._update_snap` as part of the current workflow. | CN: 在当前流程中调用 `self._update_snap`。
- **L474** EN: Calls `winfos.add` as part of the current workflow. | CN: 在当前流程中调用 `winfos.add`。
- **L475** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L476** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L477** EN: Defines function `_delete_callback`. | CN: 定义函数 `_delete_callback`。
- **L478** EN: Keeps the inline comment or directive: Callback to be called when the storage object corresponding to the  ``_WeakRefIn | CN: 保留这一行注释或指令：Callback to be called when the storage object corresponding to the  ``_WeakRefIn
- **L479** EN: Keeps the inline comment or directive: instance is about to be finalized. | CN: 保留这一行注释或指令：instance is about to be finalized.
- **L480** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 481-500 / 第 481-500 行

````python
            self._update_snap(_UpdateType.DEL, winfo)

    def _track_resize(self) -> None:
        # Need to monkey-patch this because ``torch.UntypedStorage.resize_`` is not captured
        # by ``TorchDispatchMode``.
        @wraps(self._orig_resize)
        def resize_(st: torch.UntypedStorage, size: int) -> None:
            self._orig_resize(st, size)
            winfo, _ = self._WINFO.get(st, (None, None))
            if winfo is not None and winfo.size != st.size():
                old_mem_consumed = winfo.mem_consumed
                winfo.update_mem_consumed(st)
                self._update_snap(
                    _UpdateType.SIZE, winfo, old_mem_consumed=old_mem_consumed
                )

        torch.UntypedStorage.resize_ = resize_  # type: ignore[method-assign, assignment]

    def _restore_resize(self) -> None:
        torch.UntypedStorage.resize_ = self._orig_resize  # type: ignore[method-assign]
````

- **L481** EN: Calls `self._update_snap` as part of the current workflow. | CN: 在当前流程中调用 `self._update_snap`。
- **L482** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L483** EN: Defines function `_track_resize`. | CN: 定义函数 `_track_resize`。
- **L484** EN: Keeps the inline comment or directive: Need to monkey-patch this because ``torch.UntypedStorage.resize_`` is not captur | CN: 保留这一行注释或指令：Need to monkey-patch this because ``torch.UntypedStorage.resize_`` is not captur
- **L485** EN: Keeps the inline comment or directive: by ``TorchDispatchMode``. | CN: 保留这一行注释或指令：by ``TorchDispatchMode``.
- **L486** EN: Applies decorator `wraps(self._orig_resize)` to the following definition. | CN: 将装饰器 `wraps(self._orig_resize)` 应用于后续定义。
- **L487** EN: Defines function `resize_`. | CN: 定义函数 `resize_`。
- **L488** EN: Calls `self._orig_resize` as part of the current workflow. | CN: 在当前流程中调用 `self._orig_resize`。
- **L489** EN: Assigns or updates `winfo, _`. | CN: 对 `winfo, _` 进行赋值或更新。
- **L490** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L491** EN: Assigns or updates `old_mem_consumed`. | CN: 对 `old_mem_consumed` 进行赋值或更新。
- **L492** EN: Calls `winfo.update_mem_consumed` as part of the current workflow. | CN: 在当前流程中调用 `winfo.update_mem_consumed`。
- **L493** EN: Calls `self._update_snap` as part of the current workflow. | CN: 在当前流程中调用 `self._update_snap`。
- **L494** EN: Assigns or updates `_UpdateType.SIZE, winfo, old_mem_consumed`. | CN: 对 `_UpdateType.SIZE, winfo, old_mem_consumed` 进行赋值或更新。
- **L495** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L496** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L497** EN: Assigns or updates `torch.UntypedStorage.resize_`. | CN: 对 `torch.UntypedStorage.resize_` 进行赋值或更新。
- **L498** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L499** EN: Defines function `_restore_resize`. | CN: 定义函数 `_restore_resize`。
- **L500** EN: Assigns or updates `torch.UntypedStorage.resize_`. | CN: 对 `torch.UntypedStorage.resize_` 进行赋值或更新。

### Lines 501-520 / 第 501-520 行

````python

    def _update_peak_stats(self, peak_state: _State) -> None:
        # We first capture the current memory snapshot of the current tracker state then,
        # We step through each of the modules we have tracked so far in ``memory_tracking``
        #  and check if it is currently active by querying ``_mod_tracker.parents``
        # If it is active, we update the per device peak memory usage for the module
        #  corresponding to the ``_State`` which can be ``PEAK_FW`` or ``PEAK_BW``.
        curr_snap = self._curr_mem_snap

        for mod_stats in self.memory_tracking.values():
            if mod_stats.mod_fqn in self._mod_tracker.parents:
                if peak_state in mod_stats.snapshots:
                    for dev, dev_snap in curr_snap.items():
                        if mod_stats.local_peak.get(dev, 0) < dev_snap[_TOTAL_KEY]:
                            mod_stats.local_peak[dev] = dev_snap[_TOTAL_KEY]
                            mod_stats.snapshots[peak_state][-1][dev] = deepcopy(
                                dev_snap
                            )

        for dev, dev_snap in curr_snap.items():
````

- **L501** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L502** EN: Defines function `_update_peak_stats`. | CN: 定义函数 `_update_peak_stats`。
- **L503** EN: Keeps the inline comment or directive: We first capture the current memory snapshot of the current tracker state then, | CN: 保留这一行注释或指令：We first capture the current memory snapshot of the current tracker state then,
- **L504** EN: Keeps the inline comment or directive: We step through each of the modules we have tracked so far in ``memory_tracking` | CN: 保留这一行注释或指令：We step through each of the modules we have tracked so far in ``memory_tracking`
- **L505** EN: Keeps the inline comment or directive: and check if it is currently active by querying ``_mod_tracker.parents`` | CN: 保留这一行注释或指令：and check if it is currently active by querying ``_mod_tracker.parents``
- **L506** EN: Keeps the inline comment or directive: If it is active, we update the per device peak memory usage for the module | CN: 保留这一行注释或指令：If it is active, we update the per device peak memory usage for the module
- **L507** EN: Keeps the inline comment or directive: corresponding to the ``_State`` which can be ``PEAK_FW`` or ``PEAK_BW``. | CN: 保留这一行注释或指令：corresponding to the ``_State`` which can be ``PEAK_FW`` or ``PEAK_BW``.
- **L508** EN: Assigns or updates `curr_snap`. | CN: 对 `curr_snap` 进行赋值或更新。
- **L509** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L510** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L511** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L512** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L513** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L514** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L515** EN: Assigns or updates `mod_stats.local_peak[dev]`. | CN: 对 `mod_stats.local_peak[dev]` 进行赋值或更新。
- **L516** EN: Continues the implementation inside function `_update_peak_stats`. | CN: 继续说明函数 `_update_peak_stats` 内部的实现。
- **L517** EN: Continues the implementation inside function `_update_peak_stats`. | CN: 继续说明函数 `_update_peak_stats` 内部的实现。
- **L518** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L519** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L520** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。

### Lines 521-540 / 第 521-540 行

````python
            if self._peak_mem.get(dev, 0) < dev_snap[_TOTAL_KEY]:
                self._peak_mem[dev] = dev_snap[_TOTAL_KEY]
                self._peak_mem_snap[dev] = deepcopy(dev_snap)

    def _track(self, reftype: _RefType, t: torch.Tensor) -> None:
        # Get the storages of the tensor and check if we have already tracked them.
        # If yes, then check if the storage size has changed and update the current snapshot.
        # Else create a new ``_WeakRefInfo`` instance and add it to the dictionary.
        sts = get_untyped_storages(t)
        for st in sts:
            winfo, _ = self._WINFO.get(st, (None, None))
            if winfo is not None:
                if winfo.size != st.size():
                    old_mem_consumed = winfo.mem_consumed
                    winfo.update_mem_consumed(st)
                    self._update_snap(
                        _UpdateType.SIZE, winfo, old_mem_consumed=old_mem_consumed
                    )
                return
            else:
````

- **L521** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L522** EN: Assigns or updates `self._peak_mem[dev]`. | CN: 对 `self._peak_mem[dev]` 进行赋值或更新。
- **L523** EN: Assigns or updates `self._peak_mem_snap[dev]`. | CN: 对 `self._peak_mem_snap[dev]` 进行赋值或更新。
- **L524** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L525** EN: Defines function `_track`. | CN: 定义函数 `_track`。
- **L526** EN: Keeps the inline comment or directive: Get the storages of the tensor and check if we have already tracked them. | CN: 保留这一行注释或指令：Get the storages of the tensor and check if we have already tracked them.
- **L527** EN: Keeps the inline comment or directive: If yes, then check if the storage size has changed and update the current snapsh | CN: 保留这一行注释或指令：If yes, then check if the storage size has changed and update the current snapsh
- **L528** EN: Keeps the inline comment or directive: Else create a new ``_WeakRefInfo`` instance and add it to the dictionary. | CN: 保留这一行注释或指令：Else create a new ``_WeakRefInfo`` instance and add it to the dictionary.
- **L529** EN: Assigns or updates `sts`. | CN: 对 `sts` 进行赋值或更新。
- **L530** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L531** EN: Assigns or updates `winfo, _`. | CN: 对 `winfo, _` 进行赋值或更新。
- **L532** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L533** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L534** EN: Assigns or updates `old_mem_consumed`. | CN: 对 `old_mem_consumed` 进行赋值或更新。
- **L535** EN: Calls `winfo.update_mem_consumed` as part of the current workflow. | CN: 在当前流程中调用 `winfo.update_mem_consumed`。
- **L536** EN: Calls `self._update_snap` as part of the current workflow. | CN: 在当前流程中调用 `self._update_snap`。
- **L537** EN: Assigns or updates `_UpdateType.SIZE, winfo, old_mem_consumed`. | CN: 对 `_UpdateType.SIZE, winfo, old_mem_consumed` 进行赋值或更新。
- **L538** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L539** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L540** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。

### Lines 541-560 / 第 541-560 行

````python
                winfo, w_st = _WeakRefInfo.create_winfo(
                    st, t.device, reftype, self._delete_callback
                )
                self._WINFO[st] = (winfo, w_st)
                # Update the current snapshot for the newly added ``_WeakRefInfo``.
                if winfo.mem_consumed > 0:
                    self._update_snap(_UpdateType.ADD, winfo)

    def get_tracker_snapshot(
        self, type: str = "current"
    ) -> dict[torch.device, dict[str, int]]:
        """
        Capture a snapshot of the memory usage breakdown per device, based on the specified type.

        Args:
            type (str): The type of snapshot to capture. Can be "current" for the current memory usage or "peak" for the
                        peak memory usage. Defaults to "current".
        Returns:
            Dict[torch.device, Dict[str, int]]: A dictionary where each key is a torch.device, and each value is another
                                                dictionary. This inner dictionary has keys representing memory reference
````

- **L541** EN: Assigns or updates `winfo, w_st`. | CN: 对 `winfo, w_st` 进行赋值或更新。
- **L542** EN: Continues the implementation inside function `_track`. | CN: 继续说明函数 `_track` 内部的实现。
- **L543** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L544** EN: Assigns or updates `self._WINFO[st]`. | CN: 对 `self._WINFO[st]` 进行赋值或更新。
- **L545** EN: Keeps the inline comment or directive: Update the current snapshot for the newly added ``_WeakRefInfo``. | CN: 保留这一行注释或指令：Update the current snapshot for the newly added ``_WeakRefInfo``.
- **L546** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L547** EN: Calls `self._update_snap` as part of the current workflow. | CN: 在当前流程中调用 `self._update_snap`。
- **L548** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L549** EN: Defines function `get_tracker_snapshot`. | CN: 定义函数 `get_tracker_snapshot`。
- **L550** EN: Assigns or updates `self, type`. | CN: 对 `self, type` 进行赋值或更新。
- **L551** EN: Continues the implementation inside function `get_tracker_snapshot`. | CN: 继续说明函数 `get_tracker_snapshot` 内部的实现。
- **L552** EN: Starts the docstring for the function get_tracker_snapshot. | CN: 开始定义 function get_tracker_snapshot 的文档字符串。
- **L553** EN: Continues the docstring text for the function get_tracker_snapshot. | CN: 继续补充 function get_tracker_snapshot 的文档字符串内容。
- **L554** EN: Continues the docstring text for the function get_tracker_snapshot. | CN: 继续补充 function get_tracker_snapshot 的文档字符串内容。
- **L555** EN: Continues the docstring text for the function get_tracker_snapshot. | CN: 继续补充 function get_tracker_snapshot 的文档字符串内容。
- **L556** EN: Continues the docstring text for the function get_tracker_snapshot. | CN: 继续补充 function get_tracker_snapshot 的文档字符串内容。
- **L557** EN: Continues the docstring text for the function get_tracker_snapshot. | CN: 继续补充 function get_tracker_snapshot 的文档字符串内容。
- **L558** EN: Continues the docstring text for the function get_tracker_snapshot. | CN: 继续补充 function get_tracker_snapshot 的文档字符串内容。
- **L559** EN: Continues the docstring text for the function get_tracker_snapshot. | CN: 继续补充 function get_tracker_snapshot 的文档字符串内容。
- **L560** EN: Continues the docstring text for the function get_tracker_snapshot. | CN: 继续补充 function get_tracker_snapshot 的文档字符串内容。

### Lines 561-580 / 第 561-580 行

````python
                                                types as defined in ``_MemRefType`` and values representing the amount of
                                                memory consumed in bytes.
        Raises:
            ValueError: If an invalid type is specified.
        """
        if type == "current":
            return deepcopy(self._curr_mem_snap)
        elif type == "peak":
            return deepcopy(self._peak_mem_snap)
        else:
            raise ValueError(f"Invalid type {type}")

    def _track_module_params_and_buffers(
        self, module: nn.Module, install_grad_hooks: bool = True
    ) -> tuple[int, int]:
        # Track the parameters and buffers of the module if not already tracked.
        # If the parameters have gradients, track the gradients as well.
        # If install_grad_hooks is True, install a gradient hook on the parameters
        #  to track the gradients, if it has not already been installed.
        # Return the total memory consumed by the parameters and buffers.
````

- **L561** EN: Continues the docstring text for the function get_tracker_snapshot. | CN: 继续补充 function get_tracker_snapshot 的文档字符串内容。
- **L562** EN: Continues the docstring text for the function get_tracker_snapshot. | CN: 继续补充 function get_tracker_snapshot 的文档字符串内容。
- **L563** EN: Continues the docstring text for the function get_tracker_snapshot. | CN: 继续补充 function get_tracker_snapshot 的文档字符串内容。
- **L564** EN: Continues the docstring text for the function get_tracker_snapshot. | CN: 继续补充 function get_tracker_snapshot 的文档字符串内容。
- **L565** EN: Closes the docstring for the function get_tracker_snapshot. | CN: 结束 function get_tracker_snapshot 的文档字符串。
- **L566** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L567** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L568** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L569** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L570** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L571** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L572** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L573** EN: Defines function `_track_module_params_and_buffers`. | CN: 定义函数 `_track_module_params_and_buffers`。
- **L574** EN: Assigns or updates `self, module`. | CN: 对 `self, module` 进行赋值或更新。
- **L575** EN: Continues the implementation inside function `_track_module_params_and_buffers`. | CN: 继续说明函数 `_track_module_params_and_buffers` 内部的实现。
- **L576** EN: Keeps the inline comment or directive: Track the parameters and buffers of the module if not already tracked. | CN: 保留这一行注释或指令：Track the parameters and buffers of the module if not already tracked.
- **L577** EN: Keeps the inline comment or directive: If the parameters have gradients, track the gradients as well. | CN: 保留这一行注释或指令：If the parameters have gradients, track the gradients as well.
- **L578** EN: Keeps the inline comment or directive: If install_grad_hooks is True, install a gradient hook on the parameters | CN: 保留这一行注释或指令：If install_grad_hooks is True, install a gradient hook on the parameters
- **L579** EN: Keeps the inline comment or directive: to track the gradients, if it has not already been installed. | CN: 保留这一行注释或指令：to track the gradients, if it has not already been installed.
- **L580** EN: Keeps the inline comment or directive: Return the total memory consumed by the parameters and buffers. | CN: 保留这一行注释或指令：Return the total memory consumed by the parameters and buffers.

### Lines 581-600 / 第 581-600 行

````python
        def _grad_hook(grad: torch.Tensor) -> None:
            self._update_and_maybe_create_winfos(
                grad,
                _MemRefType.GRAD,
            )

        param_memory = 0
        for param in module.parameters():
            winfos = self._update_and_maybe_create_winfos(
                param,
                _MemRefType.PARAM,
            )
            param_memory += sum(winfo.mem_consumed for winfo in winfos)
            if param.grad is not None:
                self._update_and_maybe_create_winfos(
                    param.grad,
                    _MemRefType.GRAD,
                )
            if (
                self._param_to_grad_hook_handles.get(param, None) is None
````

- **L581** EN: Defines function `_grad_hook`. | CN: 定义函数 `_grad_hook`。
- **L582** EN: Calls `self._update_and_maybe_create_winfos` as part of the current workflow. | CN: 在当前流程中调用 `self._update_and_maybe_create_winfos`。
- **L583** EN: Continues the implementation inside function `_grad_hook`. | CN: 继续说明函数 `_grad_hook` 内部的实现。
- **L584** EN: Continues the implementation inside function `_grad_hook`. | CN: 继续说明函数 `_grad_hook` 内部的实现。
- **L585** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L586** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L587** EN: Assigns or updates `param_memory`. | CN: 对 `param_memory` 进行赋值或更新。
- **L588** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L589** EN: Assigns or updates `winfos`. | CN: 对 `winfos` 进行赋值或更新。
- **L590** EN: Continues the implementation inside function `_track_module_params_and_buffers`. | CN: 继续说明函数 `_track_module_params_and_buffers` 内部的实现。
- **L591** EN: Continues the implementation inside function `_track_module_params_and_buffers`. | CN: 继续说明函数 `_track_module_params_and_buffers` 内部的实现。
- **L592** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L593** EN: Continues the implementation inside function `_track_module_params_and_buffers`. | CN: 继续说明函数 `_track_module_params_and_buffers` 内部的实现。
- **L594** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L595** EN: Calls `self._update_and_maybe_create_winfos` as part of the current workflow. | CN: 在当前流程中调用 `self._update_and_maybe_create_winfos`。
- **L596** EN: Continues the implementation inside function `_track_module_params_and_buffers`. | CN: 继续说明函数 `_track_module_params_and_buffers` 内部的实现。
- **L597** EN: Continues the implementation inside function `_track_module_params_and_buffers`. | CN: 继续说明函数 `_track_module_params_and_buffers` 内部的实现。
- **L598** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L599** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L600** EN: Calls `self._param_to_grad_hook_handles.get` as part of the current workflow. | CN: 在当前流程中调用 `self._param_to_grad_hook_handles.get`。

### Lines 601-620 / 第 601-620 行

````python
                and install_grad_hooks
            ):
                grad_hook_handle = param.register_hook(_grad_hook)
                post_acc_grad_hook_handle = param.register_post_accumulate_grad_hook(
                    lambda p: (_grad_hook(p.grad))
                )
                self._param_to_grad_hook_handles[param] = (
                    grad_hook_handle,
                    post_acc_grad_hook_handle,
                )
        buffer_memory = 0
        for buffer in module.buffers():
            winfos = self._update_and_maybe_create_winfos(
                buffer,
                _MemRefType.BUFFER,
            )
            buffer_memory += sum(winfo.mem_consumed for winfo in winfos)
        return (param_memory, buffer_memory)

    def _track_inputs_or_outputs(self, args: Any) -> int:
````

- **L601** EN: Continues the implementation inside function `_track_module_params_and_buffers`. | CN: 继续说明函数 `_track_module_params_and_buffers` 内部的实现。
- **L602** EN: Continues the implementation inside function `_track_module_params_and_buffers`. | CN: 继续说明函数 `_track_module_params_and_buffers` 内部的实现。
- **L603** EN: Assigns or updates `grad_hook_handle`. | CN: 对 `grad_hook_handle` 进行赋值或更新。
- **L604** EN: Assigns or updates `post_acc_grad_hook_handle`. | CN: 对 `post_acc_grad_hook_handle` 进行赋值或更新。
- **L605** EN: Continues the implementation inside function `_track_module_params_and_buffers`. | CN: 继续说明函数 `_track_module_params_and_buffers` 内部的实现。
- **L606** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L607** EN: Assigns or updates `self._param_to_grad_hook_handles[param]`. | CN: 对 `self._param_to_grad_hook_handles[param]` 进行赋值或更新。
- **L608** EN: Continues the implementation inside function `_track_module_params_and_buffers`. | CN: 继续说明函数 `_track_module_params_and_buffers` 内部的实现。
- **L609** EN: Continues the implementation inside function `_track_module_params_and_buffers`. | CN: 继续说明函数 `_track_module_params_and_buffers` 内部的实现。
- **L610** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L611** EN: Assigns or updates `buffer_memory`. | CN: 对 `buffer_memory` 进行赋值或更新。
- **L612** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L613** EN: Assigns or updates `winfos`. | CN: 对 `winfos` 进行赋值或更新。
- **L614** EN: Continues the implementation inside function `_track_module_params_and_buffers`. | CN: 继续说明函数 `_track_module_params_and_buffers` 内部的实现。
- **L615** EN: Continues the implementation inside function `_track_module_params_and_buffers`. | CN: 继续说明函数 `_track_module_params_and_buffers` 内部的实现。
- **L616** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L617** EN: Continues the implementation inside function `_track_module_params_and_buffers`. | CN: 继续说明函数 `_track_module_params_and_buffers` 内部的实现。
- **L618** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L619** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L620** EN: Defines function `_track_inputs_or_outputs`. | CN: 定义函数 `_track_inputs_or_outputs`。

### Lines 621-640 / 第 621-640 行

````python
        # Calculate the memory consumed by the inputs or outputs of the module.
        input_or_output_memory = 0

        def add_inps_or_outs(t: torch.Tensor) -> None:
            nonlocal input_or_output_memory
            sts = get_untyped_storages(t)
            for st in sts:
                winfo, _ = self._WINFO.get(st, (None, None))
                if winfo is not None:
                    input_or_output_memory += winfo.mem_consumed

        tree_map_only(torch.Tensor, add_inps_or_outs, args)
        return input_or_output_memory

    def _pre_fw_hook(self, module: nn.Module, inputs: Any) -> None:
        # This is installed as a pre-fwd user hook with ``ModTracker.`` Based on the following cases we
        # set the state and capture the memory snapshot for the module.
        # Case 1: If the module is not in the ``memory_tracking`` dictionary, we track the parameters, buffers,
        #         input and output memory of the module. Create a new ``_ModMemStats`` instance for the module
        #         and add it to the ``memory_tracking`` dictionary.
````

- **L621** EN: Keeps the inline comment or directive: Calculate the memory consumed by the inputs or outputs of the module. | CN: 保留这一行注释或指令：Calculate the memory consumed by the inputs or outputs of the module.
- **L622** EN: Assigns or updates `input_or_output_memory`. | CN: 对 `input_or_output_memory` 进行赋值或更新。
- **L623** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L624** EN: Defines function `add_inps_or_outs`. | CN: 定义函数 `add_inps_or_outs`。
- **L625** EN: Declares enclosing-scope names for reassignment. | CN: 声明需要重新赋值的外层作用域名称。
- **L626** EN: Assigns or updates `sts`. | CN: 对 `sts` 进行赋值或更新。
- **L627** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L628** EN: Assigns or updates `winfo, _`. | CN: 对 `winfo, _` 进行赋值或更新。
- **L629** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L630** EN: Continues the implementation inside function `add_inps_or_outs`. | CN: 继续说明函数 `add_inps_or_outs` 内部的实现。
- **L631** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L632** EN: Calls `tree_map_only` as part of the current workflow. | CN: 在当前流程中调用 `tree_map_only`。
- **L633** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L634** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L635** EN: Defines function `_pre_fw_hook`. | CN: 定义函数 `_pre_fw_hook`。
- **L636** EN: Keeps the inline comment or directive: This is installed as a pre-fwd user hook with ``ModTracker.`` Based on the follo | CN: 保留这一行注释或指令：This is installed as a pre-fwd user hook with ``ModTracker.`` Based on the follo
- **L637** EN: Keeps the inline comment or directive: set the state and capture the memory snapshot for the module. | CN: 保留这一行注释或指令：set the state and capture the memory snapshot for the module.
- **L638** EN: Keeps the inline comment or directive: Case 1: If the module is not in the ``memory_tracking`` dictionary, we track the | CN: 保留这一行注释或指令：Case 1: If the module is not in the ``memory_tracking`` dictionary, we track the
- **L639** EN: Keeps the inline comment or directive: input and output memory of the module. Create a new ``_ModMemStats`` instance fo | CN: 保留这一行注释或指令：input and output memory of the module. Create a new ``_ModMemStats`` instance fo
- **L640** EN: Keeps the inline comment or directive: and add it to the ``memory_tracking`` dictionary. | CN: 保留这一行注释或指令：and add it to the ``memory_tracking`` dictionary.

### Lines 641-660 / 第 641-660 行

````python
        # Case 2: If the module is already in the ``memory_tracking`` dictionary and we are in backward, this means
        #         we are in the AC region. We check if this is the top most module in the AC region. If it is,
        #         we store a weak reference and set the flag ``_in_ac`` to True.
        # Case 3: If the module is already in the ``memory_tracking`` dictionary and we are in forward, this means
        #         this module is called for the second time. If it is a root module, that means we are in the next
        #         iteration and we error out. If it is not a root module, that means it's a submodule that is being
        #         used multiple times in the same iteration, which we allow and track.
        # For Case 1 and 3, we also initialize the ``local_peak`` and ``PEAK_FW`` snapshot for the module.
        mod_name = self._mod_tracker.get_known_fqn(module)
        if mod_name is None:
            raise AssertionError
        if module not in self.memory_tracking:
            mod_stats = _ModMemStats(mod_name)
            param_mem, buffer_mem = self._track_module_params_and_buffers(
                module, install_grad_hooks=True
            )
            input_mem = self._track_inputs_or_outputs(inputs)
            mod_stats.parameter_mem = param_mem
            mod_stats.buffer_mem = buffer_mem
            mod_stats.input_mem = input_mem
````

- **L641** EN: Keeps the inline comment or directive: Case 2: If the module is already in the ``memory_tracking`` dictionary and we ar | CN: 保留这一行注释或指令：Case 2: If the module is already in the ``memory_tracking`` dictionary and we ar
- **L642** EN: Keeps the inline comment or directive: we are in the AC region. We check if this is the top most module in the AC regio | CN: 保留这一行注释或指令：we are in the AC region. We check if this is the top most module in the AC regio
- **L643** EN: Keeps the inline comment or directive: we store a weak reference and set the flag ``_in_ac`` to True. | CN: 保留这一行注释或指令：we store a weak reference and set the flag ``_in_ac`` to True.
- **L644** EN: Keeps the inline comment or directive: Case 3: If the module is already in the ``memory_tracking`` dictionary and we ar | CN: 保留这一行注释或指令：Case 3: If the module is already in the ``memory_tracking`` dictionary and we ar
- **L645** EN: Keeps the inline comment or directive: this module is called for the second time. If it is a root module, that means we | CN: 保留这一行注释或指令：this module is called for the second time. If it is a root module, that means we
- **L646** EN: Keeps the inline comment or directive: iteration and we error out. If it is not a root module, that means it's a submod | CN: 保留这一行注释或指令：iteration and we error out. If it is not a root module, that means it's a submod
- **L647** EN: Keeps the inline comment or directive: used multiple times in the same iteration, which we allow and track. | CN: 保留这一行注释或指令：used multiple times in the same iteration, which we allow and track.
- **L648** EN: Keeps the inline comment or directive: For Case 1 and 3, we also initialize the ``local_peak`` and ``PEAK_FW`` snapshot | CN: 保留这一行注释或指令：For Case 1 and 3, we also initialize the ``local_peak`` and ``PEAK_FW`` snapshot
- **L649** EN: Assigns or updates `mod_name`. | CN: 对 `mod_name` 进行赋值或更新。
- **L650** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L651** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L652** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L653** EN: Assigns or updates `mod_stats`. | CN: 对 `mod_stats` 进行赋值或更新。
- **L654** EN: Assigns or updates `param_mem, buffer_mem`. | CN: 对 `param_mem, buffer_mem` 进行赋值或更新。
- **L655** EN: Assigns or updates `module, install_grad_hooks`. | CN: 对 `module, install_grad_hooks` 进行赋值或更新。
- **L656** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L657** EN: Assigns or updates `input_mem`. | CN: 对 `input_mem` 进行赋值或更新。
- **L658** EN: Assigns or updates `mod_stats.parameter_mem`. | CN: 对 `mod_stats.parameter_mem` 进行赋值或更新。
- **L659** EN: Assigns or updates `mod_stats.buffer_mem`. | CN: 对 `mod_stats.buffer_mem` 进行赋值或更新。
- **L660** EN: Assigns or updates `mod_stats.input_mem`. | CN: 对 `mod_stats.input_mem` 进行赋值或更新。

### Lines 661-680 / 第 661-680 行

````python
            self.memory_tracking[module] = mod_stats
            state = _ModState.PRE_FW

        elif self._mod_tracker.is_bw:
            mod_stats = self.memory_tracking[module]
            state = _ModState.PRE_FW_AC
            if self._ac_mod is None:
                self._ac_mod = weakref.ref(module)
                self._in_ac = True
        else:
            parents = set(self._mod_tracker.parents) - {mod_name}
            if len(parents) == 1 and "Global" in parents:
                raise NotImplementedError(
                    "MemTracker does not support memory tracking for multiple iterative calls."
                    " Either use ``reset_mod_stats`` to clear module memory stats for the previous iteration"
                    " or file a github issue if you need this feature."
                )
            mod_stats = self.memory_tracking[module]
            state = _ModState.PRE_FW
            input_mem = self._track_inputs_or_outputs(inputs)
````

- **L661** EN: Assigns or updates `self.memory_tracking[module]`. | CN: 对 `self.memory_tracking[module]` 进行赋值或更新。
- **L662** EN: Assigns or updates `state`. | CN: 对 `state` 进行赋值或更新。
- **L663** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L664** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L665** EN: Assigns or updates `mod_stats`. | CN: 对 `mod_stats` 进行赋值或更新。
- **L666** EN: Assigns or updates `state`. | CN: 对 `state` 进行赋值或更新。
- **L667** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L668** EN: Assigns or updates `self._ac_mod`. | CN: 对 `self._ac_mod` 进行赋值或更新。
- **L669** EN: Assigns or updates `self._in_ac`. | CN: 对 `self._in_ac` 进行赋值或更新。
- **L670** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L671** EN: Assigns or updates `parents`. | CN: 对 `parents` 进行赋值或更新。
- **L672** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L673** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L674** EN: Continues the implementation inside function `_pre_fw_hook`. | CN: 继续说明函数 `_pre_fw_hook` 内部的实现。
- **L675** EN: Continues the implementation inside function `_pre_fw_hook`. | CN: 继续说明函数 `_pre_fw_hook` 内部的实现。
- **L676** EN: Continues the implementation inside function `_pre_fw_hook`. | CN: 继续说明函数 `_pre_fw_hook` 内部的实现。
- **L677** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L678** EN: Assigns or updates `mod_stats`. | CN: 对 `mod_stats` 进行赋值或更新。
- **L679** EN: Assigns or updates `state`. | CN: 对 `state` 进行赋值或更新。
- **L680** EN: Assigns or updates `input_mem`. | CN: 对 `input_mem` 进行赋值或更新。

### Lines 681-700 / 第 681-700 行

````python
            mod_stats.mod_fqn = mod_name
            mod_stats.input_mem = input_mem

        mem_snapshot = self.get_tracker_snapshot()
        if state == _ModState.PRE_FW:
            mod_stats.local_peak = {
                dev: dev_snap[_TOTAL_KEY] for dev, dev_snap in mem_snapshot.items()
            }
            mod_stats.snapshots.setdefault(_ModState.PEAK_FW, []).append(mem_snapshot)
        mod_stats.snapshots.setdefault(state, []).append(deepcopy(mem_snapshot))

    def _post_fw_hook(self, module: nn.Module, inputs: Any, outputs: Any) -> None:
        # This is installed as a post-fwd user hook with ``ModTracker``. Based on the following cases we
        # set the state and capture the memory snapshot for the module.
        # Case 1: This is called in backward, which means we are in the AC region. If this is the top most module
        #         in the AC region, we set the flag ``_in_ac`` to False.
        # Case 2: This is called in forward so we calculate the output memory
        #         of the module and update its mod_stats.
        mod_stats = self.memory_tracking[module]
        if self._mod_tracker.is_bw:
````

- **L681** EN: Assigns or updates `mod_stats.mod_fqn`. | CN: 对 `mod_stats.mod_fqn` 进行赋值或更新。
- **L682** EN: Assigns or updates `mod_stats.input_mem`. | CN: 对 `mod_stats.input_mem` 进行赋值或更新。
- **L683** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L684** EN: Assigns or updates `mem_snapshot`. | CN: 对 `mem_snapshot` 进行赋值或更新。
- **L685** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L686** EN: Assigns or updates `mod_stats.local_peak`. | CN: 对 `mod_stats.local_peak` 进行赋值或更新。
- **L687** EN: Continues the implementation inside function `_pre_fw_hook`. | CN: 继续说明函数 `_pre_fw_hook` 内部的实现。
- **L688** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L689** EN: Calls `mod_stats.snapshots.setdefault` as part of the current workflow. | CN: 在当前流程中调用 `mod_stats.snapshots.setdefault`。
- **L690** EN: Calls `mod_stats.snapshots.setdefault` as part of the current workflow. | CN: 在当前流程中调用 `mod_stats.snapshots.setdefault`。
- **L691** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L692** EN: Defines function `_post_fw_hook`. | CN: 定义函数 `_post_fw_hook`。
- **L693** EN: Keeps the inline comment or directive: This is installed as a post-fwd user hook with ``ModTracker``. Based on the foll | CN: 保留这一行注释或指令：This is installed as a post-fwd user hook with ``ModTracker``. Based on the foll
- **L694** EN: Keeps the inline comment or directive: set the state and capture the memory snapshot for the module. | CN: 保留这一行注释或指令：set the state and capture the memory snapshot for the module.
- **L695** EN: Keeps the inline comment or directive: Case 1: This is called in backward, which means we are in the AC region. If this | CN: 保留这一行注释或指令：Case 1: This is called in backward, which means we are in the AC region. If this
- **L696** EN: Keeps the inline comment or directive: in the AC region, we set the flag ``_in_ac`` to False. | CN: 保留这一行注释或指令：in the AC region, we set the flag ``_in_ac`` to False.
- **L697** EN: Keeps the inline comment or directive: Case 2: This is called in forward so we calculate the output memory | CN: 保留这一行注释或指令：Case 2: This is called in forward so we calculate the output memory
- **L698** EN: Keeps the inline comment or directive: of the module and update its mod_stats. | CN: 保留这一行注释或指令：of the module and update its mod_stats.
- **L699** EN: Assigns or updates `mod_stats`. | CN: 对 `mod_stats` 进行赋值或更新。
- **L700** EN: Starts a conditional branch. | CN: 开始一个条件分支。

### Lines 701-720 / 第 701-720 行

````python
            state = _ModState.POST_FW_AC
            if self._ac_mod is not None and self._ac_mod() is module:
                self._ac_mod = None
                self._in_ac = False
        else:
            state = _ModState.POST_FW
            output_mem = self._track_inputs_or_outputs(outputs)
            mod_stats.output_mem = output_mem
        mod_stats.snapshots.setdefault(state, []).append(self.get_tracker_snapshot())

    def _pre_bw_hook(self, module: nn.Module, args: Any) -> None:
        # This is installed as a pre-bwd user hook with ``ModTracker``. We set the state and capture the
        # snapshot for the module. We also initialize the ``local_peak`` and ``PEAK_BW`` snapshot for it.
        # If the module is None, we skip the hook.
        # This can happen since this installed inside a multi-grad hook on the module's output tensors
        # and the module itself may not be alive during backward.
        if module is None:
            warnings.warn("Module is None. Skipping PRE_BW hook.", stacklevel=2)
            return
        mod_stats = self.memory_tracking[module]
````

- **L701** EN: Assigns or updates `state`. | CN: 对 `state` 进行赋值或更新。
- **L702** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L703** EN: Assigns or updates `self._ac_mod`. | CN: 对 `self._ac_mod` 进行赋值或更新。
- **L704** EN: Assigns or updates `self._in_ac`. | CN: 对 `self._in_ac` 进行赋值或更新。
- **L705** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L706** EN: Assigns or updates `state`. | CN: 对 `state` 进行赋值或更新。
- **L707** EN: Assigns or updates `output_mem`. | CN: 对 `output_mem` 进行赋值或更新。
- **L708** EN: Assigns or updates `mod_stats.output_mem`. | CN: 对 `mod_stats.output_mem` 进行赋值或更新。
- **L709** EN: Calls `mod_stats.snapshots.setdefault` as part of the current workflow. | CN: 在当前流程中调用 `mod_stats.snapshots.setdefault`。
- **L710** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L711** EN: Defines function `_pre_bw_hook`. | CN: 定义函数 `_pre_bw_hook`。
- **L712** EN: Keeps the inline comment or directive: This is installed as a pre-bwd user hook with ``ModTracker``. We set the state a | CN: 保留这一行注释或指令：This is installed as a pre-bwd user hook with ``ModTracker``. We set the state a
- **L713** EN: Keeps the inline comment or directive: snapshot for the module. We also initialize the ``local_peak`` and ``PEAK_BW`` s | CN: 保留这一行注释或指令：snapshot for the module. We also initialize the ``local_peak`` and ``PEAK_BW`` s
- **L714** EN: Keeps the inline comment or directive: If the module is None, we skip the hook. | CN: 保留这一行注释或指令：If the module is None, we skip the hook.
- **L715** EN: Keeps the inline comment or directive: This can happen since this installed inside a multi-grad hook on the module's ou | CN: 保留这一行注释或指令：This can happen since this installed inside a multi-grad hook on the module's ou
- **L716** EN: Keeps the inline comment or directive: and the module itself may not be alive during backward. | CN: 保留这一行注释或指令：and the module itself may not be alive during backward.
- **L717** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L718** EN: Calls `warnings.warn` as part of the current workflow. | CN: 在当前流程中调用 `warnings.warn`。
- **L719** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L720** EN: Assigns or updates `mod_stats`. | CN: 对 `mod_stats` 进行赋值或更新。

### Lines 721-740 / 第 721-740 行

````python
        mem_snapshot = self.get_tracker_snapshot()
        mod_stats.local_peak = {
            dev: dev_snap[_TOTAL_KEY] for dev, dev_snap in mem_snapshot.items()
        }
        mod_stats.snapshots.setdefault(_ModState.PEAK_BW, []).append(mem_snapshot)
        mod_stats.snapshots.setdefault(_ModState.PRE_BW, []).append(
            deepcopy(mem_snapshot)
        )

    def _post_bw_hook(self, module: nn.Module, args: Any) -> None:
        # This is installed as a post-bwd user hook with ``ModTracker``. We set the state and capture the
        # snapshot for the module if it is not None.
        # This can happen since this installed inside a multi-grad hook on the module's input tensors
        # and the module itself may not be alive during backward.
        if module is None:
            warnings.warn("Module is None. Skipping POST_BW hook.", stacklevel=2)
            return
        mod_stats = self.memory_tracking[module]
        mod_stats.snapshots.setdefault(_ModState.POST_BW, []).append(
            self.get_tracker_snapshot()
````

- **L721** EN: Assigns or updates `mem_snapshot`. | CN: 对 `mem_snapshot` 进行赋值或更新。
- **L722** EN: Assigns or updates `mod_stats.local_peak`. | CN: 对 `mod_stats.local_peak` 进行赋值或更新。
- **L723** EN: Continues the implementation inside function `_pre_bw_hook`. | CN: 继续说明函数 `_pre_bw_hook` 内部的实现。
- **L724** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L725** EN: Calls `mod_stats.snapshots.setdefault` as part of the current workflow. | CN: 在当前流程中调用 `mod_stats.snapshots.setdefault`。
- **L726** EN: Calls `mod_stats.snapshots.setdefault` as part of the current workflow. | CN: 在当前流程中调用 `mod_stats.snapshots.setdefault`。
- **L727** EN: Calls `deepcopy` as part of the current workflow. | CN: 在当前流程中调用 `deepcopy`。
- **L728** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L729** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L730** EN: Defines function `_post_bw_hook`. | CN: 定义函数 `_post_bw_hook`。
- **L731** EN: Keeps the inline comment or directive: This is installed as a post-bwd user hook with ``ModTracker``. We set the state  | CN: 保留这一行注释或指令：This is installed as a post-bwd user hook with ``ModTracker``. We set the state 
- **L732** EN: Keeps the inline comment or directive: snapshot for the module if it is not None. | CN: 保留这一行注释或指令：snapshot for the module if it is not None.
- **L733** EN: Keeps the inline comment or directive: This can happen since this installed inside a multi-grad hook on the module's in | CN: 保留这一行注释或指令：This can happen since this installed inside a multi-grad hook on the module's in
- **L734** EN: Keeps the inline comment or directive: and the module itself may not be alive during backward. | CN: 保留这一行注释或指令：and the module itself may not be alive during backward.
- **L735** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L736** EN: Calls `warnings.warn` as part of the current workflow. | CN: 在当前流程中调用 `warnings.warn`。
- **L737** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L738** EN: Assigns or updates `mod_stats`. | CN: 对 `mod_stats` 进行赋值或更新。
- **L739** EN: Calls `mod_stats.snapshots.setdefault` as part of the current workflow. | CN: 在当前流程中调用 `mod_stats.snapshots.setdefault`。
- **L740** EN: Calls `self.get_tracker_snapshot` as part of the current workflow. | CN: 在当前流程中调用 `self.get_tracker_snapshot`。

### Lines 741-760 / 第 741-760 行

````python
        )

    def _track_optimizer_states(
        self, reftype: _RefType, optimizer: optim.Optimizer
    ) -> None:
        for states in optimizer.state.values():
            for val in states.values():
                if isinstance(val, torch.Tensor):
                    self._update_and_maybe_create_winfos(
                        val,
                        reftype,
                    )

    def _register_global_optimizer_hook(self) -> None:
        # Register a hook on the optimizer step to track the optimizer states.
        # The pre-hook is to set the flag ``_in_opt`` to True. The post-hook unsets the flag,
        # and also tracks any optimizer states that are created during the optimizer step.
        def _opt_step_pre_hook(
            optimizer: optim.Optimizer, args: Any, kwargs: Any
        ) -> None:
````

- **L741** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L742** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L743** EN: Defines function `_track_optimizer_states`. | CN: 定义函数 `_track_optimizer_states`。
- **L744** EN: Continues the implementation inside function `_track_optimizer_states`. | CN: 继续说明函数 `_track_optimizer_states` 内部的实现。
- **L745** EN: Continues the implementation inside function `_track_optimizer_states`. | CN: 继续说明函数 `_track_optimizer_states` 内部的实现。
- **L746** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L747** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L748** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L749** EN: Calls `self._update_and_maybe_create_winfos` as part of the current workflow. | CN: 在当前流程中调用 `self._update_and_maybe_create_winfos`。
- **L750** EN: Continues the implementation inside function `_track_optimizer_states`. | CN: 继续说明函数 `_track_optimizer_states` 内部的实现。
- **L751** EN: Continues the implementation inside function `_track_optimizer_states`. | CN: 继续说明函数 `_track_optimizer_states` 内部的实现。
- **L752** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L753** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L754** EN: Defines function `_register_global_optimizer_hook`. | CN: 定义函数 `_register_global_optimizer_hook`。
- **L755** EN: Keeps the inline comment or directive: Register a hook on the optimizer step to track the optimizer states. | CN: 保留这一行注释或指令：Register a hook on the optimizer step to track the optimizer states.
- **L756** EN: Keeps the inline comment or directive: The pre-hook is to set the flag ``_in_opt`` to True. The post-hook unsets the fl | CN: 保留这一行注释或指令：The pre-hook is to set the flag ``_in_opt`` to True. The post-hook unsets the fl
- **L757** EN: Keeps the inline comment or directive: and also tracks any optimizer states that are created during the optimizer step. | CN: 保留这一行注释或指令：and also tracks any optimizer states that are created during the optimizer step.
- **L758** EN: Defines function `_opt_step_pre_hook`. | CN: 定义函数 `_opt_step_pre_hook`。
- **L759** EN: Continues the implementation inside function `_opt_step_pre_hook`. | CN: 继续说明函数 `_opt_step_pre_hook` 内部的实现。
- **L760** EN: Continues the implementation inside function `_opt_step_pre_hook`. | CN: 继续说明函数 `_opt_step_pre_hook` 内部的实现。

### Lines 761-780 / 第 761-780 行

````python
            self._in_opt = True

        def _opt_step_post_hook(
            optimizer: optim.Optimizer, args: Any, kwargs: Any
        ) -> None:
            self._track_optimizer_states(_MemRefType.OPT, optimizer)
            self._in_opt = False

        self._optimizer_hook_handles = (
            register_optimizer_step_pre_hook(_opt_step_pre_hook),
            register_optimizer_step_post_hook(_opt_step_post_hook),
        )

    def _deregister_param_and_optimizer_hooks(self) -> None:
        for (
            grad_hook_handle,
            post_acc_grad_hook_handle,
        ) in self._param_to_grad_hook_handles.values():
            grad_hook_handle.remove()
            post_acc_grad_hook_handle.remove()
````

- **L761** EN: Assigns or updates `self._in_opt`. | CN: 对 `self._in_opt` 进行赋值或更新。
- **L762** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L763** EN: Defines function `_opt_step_post_hook`. | CN: 定义函数 `_opt_step_post_hook`。
- **L764** EN: Continues the implementation inside function `_opt_step_post_hook`. | CN: 继续说明函数 `_opt_step_post_hook` 内部的实现。
- **L765** EN: Continues the implementation inside function `_opt_step_post_hook`. | CN: 继续说明函数 `_opt_step_post_hook` 内部的实现。
- **L766** EN: Calls `self._track_optimizer_states` as part of the current workflow. | CN: 在当前流程中调用 `self._track_optimizer_states`。
- **L767** EN: Assigns or updates `self._in_opt`. | CN: 对 `self._in_opt` 进行赋值或更新。
- **L768** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L769** EN: Assigns or updates `self._optimizer_hook_handles`. | CN: 对 `self._optimizer_hook_handles` 进行赋值或更新。
- **L770** EN: Calls `register_optimizer_step_pre_hook` as part of the current workflow. | CN: 在当前流程中调用 `register_optimizer_step_pre_hook`。
- **L771** EN: Calls `register_optimizer_step_post_hook` as part of the current workflow. | CN: 在当前流程中调用 `register_optimizer_step_post_hook`。
- **L772** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L773** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L774** EN: Defines function `_deregister_param_and_optimizer_hooks`. | CN: 定义函数 `_deregister_param_and_optimizer_hooks`。
- **L775** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L776** EN: Continues the implementation inside function `_deregister_param_and_optimizer_hooks`. | CN: 继续说明函数 `_deregister_param_and_optimizer_hooks` 内部的实现。
- **L777** EN: Continues the implementation inside function `_deregister_param_and_optimizer_hooks`. | CN: 继续说明函数 `_deregister_param_and_optimizer_hooks` 内部的实现。
- **L778** EN: Continues the implementation inside function `_deregister_param_and_optimizer_hooks`. | CN: 继续说明函数 `_deregister_param_and_optimizer_hooks` 内部的实现。
- **L779** EN: Calls `grad_hook_handle.remove` as part of the current workflow. | CN: 在当前流程中调用 `grad_hook_handle.remove`。
- **L780** EN: Calls `post_acc_grad_hook_handle.remove` as part of the current workflow. | CN: 在当前流程中调用 `post_acc_grad_hook_handle.remove`。

### Lines 781-800 / 第 781-800 行

````python
        self._param_to_grad_hook_handles.clear()

        if self._optimizer_hook_handles is not None:
            for handle in self._optimizer_hook_handles:
                handle.remove()
            self._optimizer_hook_handles = None

    def track_external(
        self, *external: nn.Module | optim.Optimizer | torch.Tensor
    ) -> None:
        """
        Track tensors and stateful objects like modules, optimizers etc. that are created outside the MemTracker.

        This method should be called before the ``MemTracker`` is used. Any tensors that are not module parameters, buffers,
        gradients activations, or optimizer states will be categorized as ``Other``. If you want them categorized with a
        custom name, please file a GitHub issue. Any tensors created outside the MemTracker and not supplied to this
        method will not be be tracked by ``MemTracker``.

        Args:
            *external (Union[nn.Module, optim.Optimizer, torch.Tensor]): The external modules, optimizers, and
````

- **L781** EN: Calls `self._param_to_grad_hook_handles.clear` as part of the current workflow. | CN: 在当前流程中调用 `self._param_to_grad_hook_handles.clear`。
- **L782** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L783** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L784** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L785** EN: Calls `handle.remove` as part of the current workflow. | CN: 在当前流程中调用 `handle.remove`。
- **L786** EN: Assigns or updates `self._optimizer_hook_handles`. | CN: 对 `self._optimizer_hook_handles` 进行赋值或更新。
- **L787** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L788** EN: Defines function `track_external`. | CN: 定义函数 `track_external`。
- **L789** EN: Continues the implementation inside function `track_external`. | CN: 继续说明函数 `track_external` 内部的实现。
- **L790** EN: Continues the implementation inside function `track_external`. | CN: 继续说明函数 `track_external` 内部的实现。
- **L791** EN: Starts the docstring for the function track_external. | CN: 开始定义 function track_external 的文档字符串。
- **L792** EN: Continues the docstring text for the function track_external. | CN: 继续补充 function track_external 的文档字符串内容。
- **L793** EN: Continues the docstring text for the function track_external. | CN: 继续补充 function track_external 的文档字符串内容。
- **L794** EN: Continues the docstring text for the function track_external. | CN: 继续补充 function track_external 的文档字符串内容。
- **L795** EN: Continues the docstring text for the function track_external. | CN: 继续补充 function track_external 的文档字符串内容。
- **L796** EN: Continues the docstring text for the function track_external. | CN: 继续补充 function track_external 的文档字符串内容。
- **L797** EN: Continues the docstring text for the function track_external. | CN: 继续补充 function track_external 的文档字符串内容。
- **L798** EN: Continues the docstring text for the function track_external. | CN: 继续补充 function track_external 的文档字符串内容。
- **L799** EN: Continues the docstring text for the function track_external. | CN: 继续补充 function track_external 的文档字符串内容。
- **L800** EN: Continues the docstring text for the function track_external. | CN: 继续补充 function track_external 的文档字符串内容。

### Lines 801-820 / 第 801-820 行

````python
                                                                         tensors to be tracked.
        """
        flat_external, _ = tree_flatten(external)
        for obj in flat_external:
            if isinstance(obj, torch.Tensor):
                self._update_and_maybe_create_winfos(
                    obj,
                    _MemRefType.OTH,
                )
            elif isinstance(obj, torch.nn.Module):
                self._track_module_params_and_buffers(obj, install_grad_hooks=False)
            elif isinstance(obj, optim.Optimizer):
                self._track_optimizer_states(_MemRefType.OPT, obj)
            elif obj is None:
                continue
            else:
                raise TypeError(
                    f"Object of type {type(obj)} is not supported for tracking. "
                    f"Only stateful objects like modules, optimizers, and tensors are supported."
                )
````

- **L801** EN: Continues the docstring text for the function track_external. | CN: 继续补充 function track_external 的文档字符串内容。
- **L802** EN: Closes the docstring for the function track_external. | CN: 结束 function track_external 的文档字符串。
- **L803** EN: Assigns or updates `flat_external, _`. | CN: 对 `flat_external, _` 进行赋值或更新。
- **L804** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L805** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L806** EN: Calls `self._update_and_maybe_create_winfos` as part of the current workflow. | CN: 在当前流程中调用 `self._update_and_maybe_create_winfos`。
- **L807** EN: Continues the implementation inside function `track_external`. | CN: 继续说明函数 `track_external` 内部的实现。
- **L808** EN: Continues the implementation inside function `track_external`. | CN: 继续说明函数 `track_external` 内部的实现。
- **L809** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L810** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L811** EN: Calls `self._track_module_params_and_buffers` as part of the current workflow. | CN: 在当前流程中调用 `self._track_module_params_and_buffers`。
- **L812** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L813** EN: Calls `self._track_optimizer_states` as part of the current workflow. | CN: 在当前流程中调用 `self._track_optimizer_states`。
- **L814** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L815** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L816** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L817** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L818** EN: Continues the implementation inside function `track_external`. | CN: 继续说明函数 `track_external` 内部的实现。
- **L819** EN: Continues the implementation inside function `track_external`. | CN: 继续说明函数 `track_external` 内部的实现。
- **L820** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。

### Lines 821-840 / 第 821-840 行

````python

    def display_snapshot(
        self, type: str = "current", units: str = "B", tabulate: bool = False
    ) -> None:
        """
        Display the memory usage breakdown snapshot of the tracker based on the specified type and units.

        Keyword args:
            type (str): The type of snapshot to display. Can be "current" for the current memory usage or "peak" for the
                        peak memory usage. Defaults to "current".
            units (str): The units to use for displaying memory usage. Defaults to "B". Supports ["B", "KiB", "MiB", "GiB"].
            tabulate (bool): Whether to display the snapshot in a tabular format. Defaults to False.
        """
        snapshot = self.get_tracker_snapshot(type)
        if tabulate:
            _print_snapshot_tabular(snapshot, units)
        else:
            _print_snapshot(snapshot, units)

    def display_modulewise_snapshots(
````

- **L821** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L822** EN: Defines function `display_snapshot`. | CN: 定义函数 `display_snapshot`。
- **L823** EN: Assigns or updates `self, type`. | CN: 对 `self, type` 进行赋值或更新。
- **L824** EN: Continues the implementation inside function `display_snapshot`. | CN: 继续说明函数 `display_snapshot` 内部的实现。
- **L825** EN: Starts the docstring for the function display_snapshot. | CN: 开始定义 function display_snapshot 的文档字符串。
- **L826** EN: Continues the docstring text for the function display_snapshot. | CN: 继续补充 function display_snapshot 的文档字符串内容。
- **L827** EN: Continues the docstring text for the function display_snapshot. | CN: 继续补充 function display_snapshot 的文档字符串内容。
- **L828** EN: Continues the docstring text for the function display_snapshot. | CN: 继续补充 function display_snapshot 的文档字符串内容。
- **L829** EN: Continues the docstring text for the function display_snapshot. | CN: 继续补充 function display_snapshot 的文档字符串内容。
- **L830** EN: Continues the docstring text for the function display_snapshot. | CN: 继续补充 function display_snapshot 的文档字符串内容。
- **L831** EN: Continues the docstring text for the function display_snapshot. | CN: 继续补充 function display_snapshot 的文档字符串内容。
- **L832** EN: Continues the docstring text for the function display_snapshot. | CN: 继续补充 function display_snapshot 的文档字符串内容。
- **L833** EN: Closes the docstring for the function display_snapshot. | CN: 结束 function display_snapshot 的文档字符串。
- **L834** EN: Assigns or updates `snapshot`. | CN: 对 `snapshot` 进行赋值或更新。
- **L835** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L836** EN: Calls `_print_snapshot_tabular` as part of the current workflow. | CN: 在当前流程中调用 `_print_snapshot_tabular`。
- **L837** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L838** EN: Calls `_print_snapshot` as part of the current workflow. | CN: 在当前流程中调用 `_print_snapshot`。
- **L839** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L840** EN: Defines function `display_modulewise_snapshots`. | CN: 定义函数 `display_modulewise_snapshots`。

### Lines 841-860 / 第 841-860 行

````python
        self, depth: int = 2, units: str = "B", tabulate: bool = False
    ) -> None:
        """
        Print per device memory breakdown snapshot for each module called within MemTracker.

        Snapshots are displayed for the states defined by ``_ModState``.
        The module hierarchy is displayed up to the specified depth.

        Keyword Args:
            depth (int, optional): The depth of the module hierarchy to display. Defaults to 2.
            units (str, optional): The units to use for memory tracking. Defaults to "B". Supports ["B", "KiB", "MiB", "GiB"].
            tabulate (bool, optional): Whether to display the snapshot in a tabular format. Defaults to False.
        """

        def natural_sort_key(s: str) -> list[int | str]:
            return [
                int(text) if text.isdigit() else text.lower()
                for text in re.split("([0-9]+)", s)
            ]

````

- **L841** EN: Assigns or updates `self, depth`. | CN: 对 `self, depth` 进行赋值或更新。
- **L842** EN: Continues the implementation inside function `display_modulewise_snapshots`. | CN: 继续说明函数 `display_modulewise_snapshots` 内部的实现。
- **L843** EN: Starts the docstring for the function display_modulewise_snapshots. | CN: 开始定义 function display_modulewise_snapshots 的文档字符串。
- **L844** EN: Continues the docstring text for the function display_modulewise_snapshots. | CN: 继续补充 function display_modulewise_snapshots 的文档字符串内容。
- **L845** EN: Continues the docstring text for the function display_modulewise_snapshots. | CN: 继续补充 function display_modulewise_snapshots 的文档字符串内容。
- **L846** EN: Continues the docstring text for the function display_modulewise_snapshots. | CN: 继续补充 function display_modulewise_snapshots 的文档字符串内容。
- **L847** EN: Continues the docstring text for the function display_modulewise_snapshots. | CN: 继续补充 function display_modulewise_snapshots 的文档字符串内容。
- **L848** EN: Continues the docstring text for the function display_modulewise_snapshots. | CN: 继续补充 function display_modulewise_snapshots 的文档字符串内容。
- **L849** EN: Continues the docstring text for the function display_modulewise_snapshots. | CN: 继续补充 function display_modulewise_snapshots 的文档字符串内容。
- **L850** EN: Continues the docstring text for the function display_modulewise_snapshots. | CN: 继续补充 function display_modulewise_snapshots 的文档字符串内容。
- **L851** EN: Continues the docstring text for the function display_modulewise_snapshots. | CN: 继续补充 function display_modulewise_snapshots 的文档字符串内容。
- **L852** EN: Continues the docstring text for the function display_modulewise_snapshots. | CN: 继续补充 function display_modulewise_snapshots 的文档字符串内容。
- **L853** EN: Closes the docstring for the function display_modulewise_snapshots. | CN: 结束 function display_modulewise_snapshots 的文档字符串。
- **L854** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L855** EN: Defines function `natural_sort_key`. | CN: 定义函数 `natural_sort_key`。
- **L856** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L857** EN: Calls `int` as part of the current workflow. | CN: 在当前流程中调用 `int`。
- **L858** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L859** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L860** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 861-880 / 第 861-880 行

````python
        for mod_stats in sorted(
            self.memory_tracking.values(),
            key=lambda m_stats: natural_sort_key(m_stats.mod_fqn),
        ):
            mod_fqn = mod_stats.mod_fqn
            mod_depth = mod_fqn.count(".") + 1
            if mod_depth > depth:
                continue
            print(f"Module:  {mod_fqn}")
            if tabulate:
                _print_state_snapshots_tabular(mod_stats.snapshots, units)
            else:
                _print_state_snapshots(mod_stats.snapshots, units)

    def reset_mod_stats(self) -> None:
        """
        Reset all the module memory stats. Clears ``memory_tracking`` dictionary.
        """
        self.memory_tracking.clear()

````

- **L861** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L862** EN: Calls `self.memory_tracking.values` as part of the current workflow. | CN: 在当前流程中调用 `self.memory_tracking.values`。
- **L863** EN: Assigns or updates `key`. | CN: 对 `key` 进行赋值或更新。
- **L864** EN: Continues the implementation inside function `display_modulewise_snapshots`. | CN: 继续说明函数 `display_modulewise_snapshots` 内部的实现。
- **L865** EN: Assigns or updates `mod_fqn`. | CN: 对 `mod_fqn` 进行赋值或更新。
- **L866** EN: Assigns or updates `mod_depth`. | CN: 对 `mod_depth` 进行赋值或更新。
- **L867** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L868** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L869** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L870** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L871** EN: Calls `_print_state_snapshots_tabular` as part of the current workflow. | CN: 在当前流程中调用 `_print_state_snapshots_tabular`。
- **L872** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L873** EN: Calls `_print_state_snapshots` as part of the current workflow. | CN: 在当前流程中调用 `_print_state_snapshots`。
- **L874** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L875** EN: Defines function `reset_mod_stats`. | CN: 定义函数 `reset_mod_stats`。
- **L876** EN: Starts the docstring for the function reset_mod_stats. | CN: 开始定义 function reset_mod_stats 的文档字符串。
- **L877** EN: Continues the docstring text for the function reset_mod_stats. | CN: 继续补充 function reset_mod_stats 的文档字符串内容。
- **L878** EN: Closes the docstring for the function reset_mod_stats. | CN: 结束 function reset_mod_stats 的文档字符串。
- **L879** EN: Calls `self.memory_tracking.clear` as part of the current workflow. | CN: 在当前流程中调用 `self.memory_tracking.clear`。
- **L880** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 881-900 / 第 881-900 行

````python
    def __enter__(self) -> "MemTracker":
        if self._depth == 0:
            self._register_global_optimizer_hook()
            self._mod_tracker.register_user_hooks(
                self._pre_fw_hook,
                self._post_fw_hook,
                self._pre_bw_hook,
                self._post_bw_hook,
            )
            self._track_resize()
            self._peak_mem_snap = self.get_tracker_snapshot()
            self._peak_mem = {
                dev: dev_snap[_TOTAL_KEY]
                for dev, dev_snap in self._peak_mem_snap.items()
            }
            self._mod_tracker.__enter__()
        super().__enter__()
        self._depth += 1
        return self

````

- **L881** EN: Defines function `__enter__`. | CN: 定义函数 `__enter__`。
- **L882** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L883** EN: Calls `self._register_global_optimizer_hook` as part of the current workflow. | CN: 在当前流程中调用 `self._register_global_optimizer_hook`。
- **L884** EN: Calls `self._mod_tracker.register_user_hooks` as part of the current workflow. | CN: 在当前流程中调用 `self._mod_tracker.register_user_hooks`。
- **L885** EN: Continues the implementation inside function `__enter__`. | CN: 继续说明函数 `__enter__` 内部的实现。
- **L886** EN: Continues the implementation inside function `__enter__`. | CN: 继续说明函数 `__enter__` 内部的实现。
- **L887** EN: Continues the implementation inside function `__enter__`. | CN: 继续说明函数 `__enter__` 内部的实现。
- **L888** EN: Continues the implementation inside function `__enter__`. | CN: 继续说明函数 `__enter__` 内部的实现。
- **L889** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L890** EN: Calls `self._track_resize` as part of the current workflow. | CN: 在当前流程中调用 `self._track_resize`。
- **L891** EN: Assigns or updates `self._peak_mem_snap`. | CN: 对 `self._peak_mem_snap` 进行赋值或更新。
- **L892** EN: Assigns or updates `self._peak_mem`. | CN: 对 `self._peak_mem` 进行赋值或更新。
- **L893** EN: Continues the implementation inside function `__enter__`. | CN: 继续说明函数 `__enter__` 内部的实现。
- **L894** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L895** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L896** EN: Calls `self._mod_tracker.__enter__` as part of the current workflow. | CN: 在当前流程中调用 `self._mod_tracker.__enter__`。
- **L897** EN: Calls `super` as part of the current workflow. | CN: 在当前流程中调用 `super`。
- **L898** EN: Continues the implementation inside function `__enter__`. | CN: 继续说明函数 `__enter__` 内部的实现。
- **L899** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L900** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 901-920 / 第 901-920 行

````python
    # pyrefly: ignore [bad-override]
    def __exit__(self, *args: Any) -> None:
        self._depth -= 1
        if self._depth == 0:
            self._deregister_param_and_optimizer_hooks()
            self._mod_tracker.clear_user_hooks()
            self._restore_resize()
            self._mod_tracker.__exit__(*args)
        super().__exit__(*args)

    def __torch_dispatch__(self, func, types, args=(), kwargs=None):  # type: ignore[no-untyped-def]
        # When running this mode with DTensor, ordinarily all modes will
        # run **before** subclasses get a chance to run.
        # Returning NotImplemented here gives us a chance to let DTensor
        # run and desugar into local tensor ops, before `MemTracker` sees them.
        if any(t == DTensor for t in types):
            return NotImplemented
        if (
            func is torch.ops._c10d_functional.wait_tensor.default
            and active_fake_mode()
````

- **L901** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-override] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-override]
- **L902** EN: Defines function `__exit__`. | CN: 定义函数 `__exit__`。
- **L903** EN: Continues the implementation inside function `__exit__`. | CN: 继续说明函数 `__exit__` 内部的实现。
- **L904** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L905** EN: Calls `self._deregister_param_and_optimizer_hooks` as part of the current workflow. | CN: 在当前流程中调用 `self._deregister_param_and_optimizer_hooks`。
- **L906** EN: Calls `self._mod_tracker.clear_user_hooks` as part of the current workflow. | CN: 在当前流程中调用 `self._mod_tracker.clear_user_hooks`。
- **L907** EN: Calls `self._restore_resize` as part of the current workflow. | CN: 在当前流程中调用 `self._restore_resize`。
- **L908** EN: Calls `self._mod_tracker.__exit__` as part of the current workflow. | CN: 在当前流程中调用 `self._mod_tracker.__exit__`。
- **L909** EN: Calls `super` as part of the current workflow. | CN: 在当前流程中调用 `super`。
- **L910** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L911** EN: Defines function `__torch_dispatch__`. | CN: 定义函数 `__torch_dispatch__`。
- **L912** EN: Keeps the inline comment or directive: When running this mode with DTensor, ordinarily all modes will | CN: 保留这一行注释或指令：When running this mode with DTensor, ordinarily all modes will
- **L913** EN: Keeps the inline comment or directive: run **before** subclasses get a chance to run. | CN: 保留这一行注释或指令：run **before** subclasses get a chance to run.
- **L914** EN: Keeps the inline comment or directive: Returning NotImplemented here gives us a chance to let DTensor | CN: 保留这一行注释或指令：Returning NotImplemented here gives us a chance to let DTensor
- **L915** EN: Keeps the inline comment or directive: run and desugar into local tensor ops, before `MemTracker` sees them. | CN: 保留这一行注释或指令：run and desugar into local tensor ops, before `MemTracker` sees them.
- **L916** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L917** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L918** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L919** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L920** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。

### Lines 921-940 / 第 921-940 行

````python
        ):
            # N.B: This is a hacky way to override the Meta IMPL of wait_tensor. The original impl returns
            # a new tensor which does not happen in eager mode, when a wait_tensor is called.
            # pyrefly: ignore [bad-index]
            res = args[0]
        else:
            res = func(*args, **kwargs or {})
        # If we are tracking an optimizer state, we use the optimizer reference type.
        # If we are in backward region and not in AC region, we use the backward reference type.
        # Else we use the forward reference type.
        if self._in_opt:
            reftype = _MemRefType.OPT
        elif self._mod_tracker.is_bw and not self._in_ac:
            reftype = _MemRefType.TEMP
        else:
            reftype = _MemRefType.ACT
        tree_map_only(torch.Tensor, partial(self._track, reftype), res)
        peak_state = _ModState.PEAK_BW if self._mod_tracker.is_bw else _ModState.PEAK_FW
        self._update_peak_stats(peak_state)
        return res
````

- **L921** EN: Continues the implementation inside function `__torch_dispatch__`. | CN: 继续说明函数 `__torch_dispatch__` 内部的实现。
- **L922** EN: Keeps the inline comment or directive: N.B: This is a hacky way to override the Meta IMPL of wait_tensor. The original  | CN: 保留这一行注释或指令：N.B: This is a hacky way to override the Meta IMPL of wait_tensor. The original 
- **L923** EN: Keeps the inline comment or directive: a new tensor which does not happen in eager mode, when a wait_tensor is called. | CN: 保留这一行注释或指令：a new tensor which does not happen in eager mode, when a wait_tensor is called.
- **L924** EN: Keeps the inline comment or directive: pyrefly: ignore [bad-index] | CN: 保留这一行注释或指令：pyrefly: ignore [bad-index]
- **L925** EN: Assigns or updates `res`. | CN: 对 `res` 进行赋值或更新。
- **L926** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L927** EN: Assigns or updates `res`. | CN: 对 `res` 进行赋值或更新。
- **L928** EN: Keeps the inline comment or directive: If we are tracking an optimizer state, we use the optimizer reference type. | CN: 保留这一行注释或指令：If we are tracking an optimizer state, we use the optimizer reference type.
- **L929** EN: Keeps the inline comment or directive: If we are in backward region and not in AC region, we use the backward reference | CN: 保留这一行注释或指令：If we are in backward region and not in AC region, we use the backward reference
- **L930** EN: Keeps the inline comment or directive: Else we use the forward reference type. | CN: 保留这一行注释或指令：Else we use the forward reference type.
- **L931** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L932** EN: Assigns or updates `reftype`. | CN: 对 `reftype` 进行赋值或更新。
- **L933** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L934** EN: Assigns or updates `reftype`. | CN: 对 `reftype` 进行赋值或更新。
- **L935** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L936** EN: Assigns or updates `reftype`. | CN: 对 `reftype` 进行赋值或更新。
- **L937** EN: Calls `tree_map_only` as part of the current workflow. | CN: 在当前流程中调用 `tree_map_only`。
- **L938** EN: Assigns or updates `peak_state`. | CN: 对 `peak_state` 进行赋值或更新。
- **L939** EN: Calls `self._update_peak_stats` as part of the current workflow. | CN: 在当前流程中调用 `self._update_peak_stats`。
- **L940** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

## Key Concepts / 关键概念

- **EN**: distributed runtime helpers, APIs, and package wiring  
  **CN**: 分布式运行时辅助逻辑、API 与包级导出
- **EN**: checkpointing  
  **CN**: 检查点
- **EN**: collective communication  
  **CN**: 集合通信
- **EN**: distributed tensors  
  **CN**: 分布式张量
- **EN**: distributed stores  
  **CN**: 分布式存储
- **EN**: optimizer coordination  
  **CN**: 优化器协同
- **EN**: communication hooks  
  **CN**: 通信钩子
- **EN**: Primary classes: _RefType, _State, _MemRefType, _ModState, _ModMemStats  
  **CN**: 主要类：_RefType, _State, _MemRefType, _ModState, _ModMemStats

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed._tools.common_utils`, `torch.distributed._tools.fake_collectives`, `torch.distributed._tools.mod_tracker`, `torch.distributed.tensor`
- **PyTorch / PyTorch**: `torch`, `torch._guards`, `torch.optim.optimizer`, `torch.utils._python_dispatch`, `torch.utils._pytree`, `torch.utils.hooks`, `torch.utils.weak`
- **Python Stdlib / Python 标准库**: `collections.abc`, `copy`, `enum`, `functools`, `math`, `os`, `re`, `typing`, `warnings`
- **Third-party / 第三方**: `tabulate`, `typing_extensions`

