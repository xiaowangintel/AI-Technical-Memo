# learned_heuristic_controller.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/autoheuristic/learned_heuristic_controller.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module contains heuristic models and helpers for autotuning decisions. It defines classes such as `LearnedHeuristicController`. It exposes functions such as `find_and_instantiate_subclasses`.
- **用途（中文）**: 该模块包含自动调优决策所需的启发式模型与辅助逻辑。其中定义了 `LearnedHeuristicController` 等类。同时提供 `find_and_instantiate_subclasses` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
import importlib
import inspect
import pkgutil
from collections import defaultdict
from typing import Any

from torch._inductor.autoheuristic.autoheuristic_utils import (
    AHContext,
    AHMetadata,
    Choice,
)
from torch._inductor.autoheuristic.learnedheuristic_interface import LearnedHeuristic


````
- **EN**: Imports dependencies such as `importlib`, `inspect`, `pkgutil`, `collections`, `typing`, `torch._inductor.autoheuristic.autoheuristic_utils`, and `...+1` for the logic in this range.
- **CN**: 这里导入了 `importlib`、`inspect`、`pkgutil`、`collections`、`typing`、`torch._inductor.autoheuristic.autoheuristic_utils`、`另有1项` 等依赖，为后续逻辑提供基础能力。

### Lines 15-28 / 第 15-28 行
````python
def find_and_instantiate_subclasses(
    package_name: str, base_class: Any
) -> list[LearnedHeuristic]:
    instances = []

    package = importlib.import_module(package_name)
    for _, module_name, _ in pkgutil.walk_packages(
        package.__path__, package.__name__ + "."
    ):
        try:
            module_basename = module_name.split(".")[-1]
            if not module_basename.startswith("_"):
                # learned heuristics start with an underscore
                continue
````
- **EN**: Introduces function `find_and_instantiate_subclasses`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `package_name`, `instances`, `package`, `try`, and `module_basename`.
- **CN**: 这里定义了函数`find_and_instantiate_subclasses`。包含分支、循环或上下文管理等控制流。初始化或更新了 `package_name`、`instances`、`package`、`try`、`module_basename` 等值。

### Lines 29-42 / 第 29-42 行
````python
            module = importlib.import_module(module_name)

            # look for classes that are subclasses of base_class
            for _name, obj in inspect.getmembers(module):
                if (
                    inspect.isclass(obj)
                    and issubclass(obj, base_class)
                    and obj != base_class
                ):
                    instance = obj()
                    instances.append(instance)
        except Exception as e:
            print(f"Error processing module {module_name}: {e}")

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `module`, and `instance`. This range continues the implementation of function `find_and_instantiate_subclasses`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `module`、`instance` 等值。这一段延续了函数`find_and_instantiate_subclasses` 的具体实现。

### Lines 43-56 / 第 43-56 行
````python
    return instances


class LearnedHeuristicController:
    """
    Class that finds and instantiates all learned heuristics. It also provides
    a way to get the decision of a learned heuristic.
    """

    existing_heuristics: dict[str, list[LearnedHeuristic]] = defaultdict(list)
    """
    A dictionary that stores all the learned heuristics for each optimization.
    The key is the optimization name, and the value is a list of LearnedHeuristic objects.
    """
````
- **EN**: Introduces class `LearnedHeuristicController`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `existing_heuristics`.
- **CN**: 这里定义了类`LearnedHeuristicController`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `existing_heuristics` 等值。

### Lines 57-70 / 第 57-70 行
````python

    heuristics_initialized: bool = False
    """
    A flag that indicates whether the learned heuristics have been initialized.
    Set to true when the get_decision() function is called for the first time.
    """

    def __init__(
        self,
        metadata: AHMetadata,
        context: AHContext,
    ) -> None:
        self.metadata = metadata
        self.context = context
````
- **EN**: Introduces function `__init__`. Initializes or updates values such as `heuristics_initialized`, `metadata`, and `context`.
- **CN**: 这里定义了函数`__init__`。初始化或更新了 `heuristics_initialized`、`metadata`、`context` 等值。

### Lines 71-84 / 第 71-84 行
````python

    def get_heuristics(self, name: str) -> list[LearnedHeuristic]:
        """
        Returns a list of learned heuristics for the given optimization name.
        """

        if not LearnedHeuristicController.heuristics_initialized:
            # learned heuristics are generated into the following package
            learned_heuristics_package = "torch._inductor.autoheuristic.artifacts"

            # learned heuristics have to be of type LearnedHeuristic
            base_class = LearnedHeuristic
            found_heuristics = find_and_instantiate_subclasses(
                learned_heuristics_package, base_class
````
- **EN**: Introduces function `get_heuristics`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `learned_heuristics_package`, `base_class`, and `found_heuristics`.
- **CN**: 这里定义了函数`get_heuristics`。包含分支、循环或上下文管理等控制流。初始化或更新了 `learned_heuristics_package`、`base_class`、`found_heuristics` 等值。

### Lines 85-98 / 第 85-98 行
````python
            )

            for learned_heuristic in found_heuristics:
                opt_name = learned_heuristic.get_name()
                LearnedHeuristicController.existing_heuristics[opt_name].append(
                    learned_heuristic
                )
            LearnedHeuristicController.heuristics_initialized = True

        return LearnedHeuristicController.existing_heuristics[name]

    def get_decision(self) -> Choice | None:
        """
        Returns the decision made by the learned heuristic or None if no heuristic was found or the heuristic is unsure
````
- **EN**: Introduces function `get_decision`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `opt_name`.
- **CN**: 这里定义了函数`get_decision`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `opt_name` 等值。

### Lines 99-112 / 第 99-112 行
````python
        which choice to make.
        """

        heuristics = self.get_heuristics(self.metadata.name)
        for heuristic in heuristics:
            if heuristic.check_precondition(self.metadata, self.context):
                return heuristic.get_decision(self.context, self.metadata.choices)
        return None

    def get_decisions_ranked(self, top_k: int) -> list[Choice] | None:
        heuristics = self.get_heuristics(self.metadata.name)
        for heuristic in heuristics:
            if heuristic.check_precondition(self.metadata, self.context):
                choices = heuristic.get_decisions_ranked(self.context)
````
- **EN**: Introduces function `get_decisions_ranked`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `heuristics`, and `choices`.
- **CN**: 这里定义了函数`get_decisions_ranked`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `heuristics`、`choices` 等值。

### Lines 113-119 / 第 113-119 行
````python
                if choices is None:
                    return None
                avail_choices = [
                    choice for choice in choices if choice in self.metadata.choices
                ]
                return avail_choices[:top_k]
        return None
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `avail_choices`. This range continues the implementation of function `LearnedHeuristicController.get_decisions_ranked`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `avail_choices` 等值。这一段延续了函数`LearnedHeuristicController.get_decisions_ranked` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Contains heuristic models and helpers for autotuning decisions  
  **CN**: 包含自动调优决策所需的启发式模型与辅助逻辑
- **EN**: Primary classes: `LearnedHeuristicController`  
  **CN**: 主要类：`LearnedHeuristicController`
- **EN**: Primary functions: `find_and_instantiate_subclasses`  
  **CN**: 主要函数：`find_and_instantiate_subclasses`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `importlib`, `inspect`, `pkgutil`, `collections`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch._inductor.autoheuristic.autoheuristic_utils`, `torch._inductor.autoheuristic.learnedheuristic_interface`
