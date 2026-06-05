# api.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/_shard/sharding_plan/api.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on sharding specifications and shard-aware tensor helpers. Its main entry points include ShardingPlan, ShardingPlanner.
- **用途 (CN)**: 该模块聚焦于分片规范与分片张量辅助逻辑，其主要入口包括 ShardingPlan, ShardingPlanner。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
import abc
from dataclasses import dataclass

import torch.nn as nn
from torch.distributed._shard.sharder import Sharder
from torch.distributed._shard.sharding_spec import ShardingSpec


@dataclass
class ShardingPlan:
    """
    Representation of a sharding plan, describes how to shard a module
    across hosts. `plan` is used to shard module parameters according to the spec provided,
    `output_plan` and `return_local_tensor` are optional, they are used to specify the output
    layout of a module with a spec, and when to convert back to data parallel fashion.

    Args:
        plan (Dict[str, Union[:class:`torch.distributed._shard.sharding_spec.ShardingSpec`,
              :class:`torch.distributed._shard.sharder.Sharder`]):
            a dict describes how to shard a module, there're currently two ways to shard a module:
````

- **L1** EN: Imports module dependencies: `abc`. | CN: 导入模块依赖：`abc`。
- **L2** EN: Imports selected names from `dataclasses`. | CN: 从 `dataclasses` 导入指定名称。
- **L3** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L4** EN: Imports module dependencies: `torch.nn as nn`. | CN: 导入模块依赖：`torch.nn as nn`。
- **L5** EN: Imports selected names from `torch.distributed._shard.sharder`. | CN: 从 `torch.distributed._shard.sharder` 导入指定名称。
- **L6** EN: Imports selected names from `torch.distributed._shard.sharding_spec`. | CN: 从 `torch.distributed._shard.sharding_spec` 导入指定名称。
- **L7** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L8** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L9** EN: Applies decorator `dataclass` to the following definition. | CN: 将装饰器 `dataclass` 应用于后续定义。
- **L10** EN: Defines class `ShardingPlan`. | CN: 定义类 `ShardingPlan`。
- **L11** EN: Starts the docstring for the class ShardingPlan. | CN: 开始定义 class ShardingPlan 的文档字符串。
- **L12** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L13** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L14** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L15** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L16** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L17** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L18** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L19** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L20** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。

### Lines 21-40 / 第 21-40 行

````python
                1. directly shard a module parameter by a `ShardingSpec`, keyed by the name of
                   a parameter to a `ShardingSpec`.
                2. shard a submodule by applying a `Sharder` on it, keyed by the name of a module
                   to a `Sharder` object.
        output_plan (Dict[str, :class:`torch.distributed._shard.sharding_spec.ShardingSpec`), optional):
            a dict specifies the layout of a module's output which produces a ShardedTensor,
            keyed by the name of module to ShardingSpec("" in key means the root module).
            Default: `None`
        return_local_tensor (List[str], optional): a list of string, each element enables
            a module's sharded output to be returned as a Tensor from its local shards to
            ensure further processing in a data parallel fashion. ("" in list means the
            root module).
            Default: None
    Example:
      Suppose we want to shard a module with two linear layers and then run it with DDP, we also
      want to convert the output of the second linear layer back to DDP, we can do it as follows:

        >>> # xdoctest: +REQUIRES(module:torch._C._distributed_c10d)
        >>> class MyModule(nn.Module):
        >>>     def __init__(self) -> None:
````

- **L21** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L22** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L23** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L24** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L25** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L26** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L27** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L28** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L29** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L30** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L31** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L32** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L33** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L34** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L35** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L36** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L37** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L38** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L39** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L40** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。

### Lines 41-60 / 第 41-60 行

````python
        >>>        super().__init__()
        >>>        self.fc1 = nn.Linear()
        >>>        self.gelu = nn.GELU()
        >>>        self.fc2 = nn.Linear()
        >>>        self.relu = nn.Linear()
        >>>
        >>>     def forward(self, input):
        >>>         return self.relu(self.fc2(self.gelu(self.fc1(input))))


        >>> # xdoctest: +SKIP("Undefined spec1, spec2)
        >>> sharding_plan = ShardingPlan(
        >>>    plan={
        >>>        "fc1.weight": spec1,
        >>>        "fc2.weight": spec2
        >>>    },
        >>>    output_plan={
        >>>        "fc2": output_spec
        >>>    },
        >>>    return_local_tensor=["fc2"]
````

- **L41** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L42** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L43** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L44** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L45** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L46** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L47** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L48** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L49** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L50** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L51** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L52** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L53** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L54** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L55** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L56** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L57** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L58** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L59** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L60** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。

### Lines 61-80 / 第 61-80 行

````python
        >>> )
    """

    plan: dict[str, ShardingSpec | Sharder]
    output_plan: dict[str, ShardingSpec] | None = None
    return_local_tensor: list[str] | None = None


class ShardingPlanner(abc.ABC):
    """
    Default ShardingPlanner interface, can be extended and
    implement advanced sharding strategies.
    """

    @abc.abstractmethod
    def build_plan(self, module: nn.Module) -> ShardingPlan:
        """
        Given a nn.Module, define how to shard the module across
        ranks, return a ShardingPlan
        Args:
````

- **L61** EN: Continues the docstring text for the class ShardingPlan. | CN: 继续补充 class ShardingPlan 的文档字符串内容。
- **L62** EN: Closes the docstring for the class ShardingPlan. | CN: 结束 class ShardingPlan 的文档字符串。
- **L63** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L64** EN: Continues the implementation inside class `ShardingPlan`. | CN: 继续说明类 `ShardingPlan` 内部的实现。
- **L65** EN: Assigns or updates `output_plan`. | CN: 对 `output_plan` 进行赋值或更新。
- **L66** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L67** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L68** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L69** EN: Defines class `ShardingPlanner`. | CN: 定义类 `ShardingPlanner`。
- **L70** EN: Starts the docstring for the class ShardingPlanner. | CN: 开始定义 class ShardingPlanner 的文档字符串。
- **L71** EN: Continues the docstring text for the class ShardingPlanner. | CN: 继续补充 class ShardingPlanner 的文档字符串内容。
- **L72** EN: Continues the docstring text for the class ShardingPlanner. | CN: 继续补充 class ShardingPlanner 的文档字符串内容。
- **L73** EN: Closes the docstring for the class ShardingPlanner. | CN: 结束 class ShardingPlanner 的文档字符串。
- **L74** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L75** EN: Applies decorator `abc.abstractmethod` to the following definition. | CN: 将装饰器 `abc.abstractmethod` 应用于后续定义。
- **L76** EN: Defines function `build_plan`. | CN: 定义函数 `build_plan`。
- **L77** EN: Starts the docstring for the function build_plan. | CN: 开始定义 function build_plan 的文档字符串。
- **L78** EN: Continues the docstring text for the function build_plan. | CN: 继续补充 function build_plan 的文档字符串内容。
- **L79** EN: Continues the docstring text for the function build_plan. | CN: 继续补充 function build_plan 的文档字符串内容。
- **L80** EN: Continues the docstring text for the function build_plan. | CN: 继续补充 function build_plan 的文档字符串内容。

### Lines 81-86 / 第 81-86 行

````python
            module (:class:`torch.nn.Module`):
                The module to apply sharding to.
        Returns:
            A :class:`torch.distributed._shard.sharding_plan.ShardingPlan` object that
            represents how to shard the module.
        """
````

- **L81** EN: Continues the docstring text for the function build_plan. | CN: 继续补充 function build_plan 的文档字符串内容。
- **L82** EN: Continues the docstring text for the function build_plan. | CN: 继续补充 function build_plan 的文档字符串内容。
- **L83** EN: Continues the docstring text for the function build_plan. | CN: 继续补充 function build_plan 的文档字符串内容。
- **L84** EN: Continues the docstring text for the function build_plan. | CN: 继续补充 function build_plan 的文档字符串内容。
- **L85** EN: Continues the docstring text for the function build_plan. | CN: 继续补充 function build_plan 的文档字符串内容。
- **L86** EN: Closes the docstring for the function build_plan. | CN: 结束 function build_plan 的文档字符串。

## Key Concepts / 关键概念

- **EN**: sharding specifications and shard-aware tensor helpers  
  **CN**: 分片规范与分片张量辅助逻辑
- **EN**: sharding  
  **CN**: 分片
- **EN**: distributed tensors  
  **CN**: 分布式张量
- **EN**: Primary classes: ShardingPlan, ShardingPlanner  
  **CN**: 主要类：ShardingPlan, ShardingPlanner

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed._shard.sharder`, `torch.distributed._shard.sharding_spec`
- **PyTorch / PyTorch**: `torch.nn`
- **Python Stdlib / Python 标准库**: `abc`, `dataclasses`
- **Third-party / 第三方**: None detected / 未检测到

