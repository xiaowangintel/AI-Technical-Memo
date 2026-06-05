# autoheuristic.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/autoheuristic/autoheuristic.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module contains heuristic models and helpers for autotuning decisions. It defines classes such as `LocalFeedback`, `InconsistentMetadata`, `AutoHeuristic`, and `AutoHeuristicSelectAlgorithm`.
- **用途（中文）**: 该模块包含自动调优决策所需的启发式模型与辅助逻辑。其中定义了 `LocalFeedback`、`InconsistentMetadata`、`AutoHeuristic`、`AutoHeuristicSelectAlgorithm` 等类。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
import json
import os
from collections.abc import Callable
from functools import partial
from typing import Any

import torch
from torch._inductor.autoheuristic.autoheuristic_utils import (
    AHContext,
    AHMetadata,
    AHOperation,
    Choice,
    CHOICE_COL,
    Feedback,
    FEEDBACK_COL,
    get_metadata_str_from_log,
)
from torch._inductor.autoheuristic.learned_heuristic_controller import (
    LearnedHeuristicController,
)
````
- **EN**: Imports dependencies such as `json`, `os`, `collections.abc`, `functools`, `typing`, `torch`, and `...+2` for the logic in this range.
- **CN**: 这里导入了 `json`、`os`、`collections.abc`、`functools`、`typing`、`torch`、`另有2项` 等依赖，为后续逻辑提供基础能力。

### Lines 21-40 / 第 21-40 行
````python
from torch._inductor.ir import ChoiceCaller
from torch._inductor.runtime.runtime_utils import cache_dir
from torch._inductor.utils import get_gpu_shared_memory


class LocalFeedback:
    """
    To be able to collect data for a choice, a function providing feedback given a choice has to be provided.
    LocalFeedback can be used when AutoHeuristic should immediately run the function to collect feedback for each choice
    (see pad_mm.py, where the autotuning happens locally, for an example).
    """

    def __init__(self, feedback_fn: Callable[[Choice], Feedback]) -> None:
        self.feedback_fn = feedback_fn

    def __call__(self, choice: Choice) -> Feedback:
        return self.feedback_fn(choice)


class InconsistentMetadata(Exception):
````
- **EN**: Imports dependencies such as `torch._inductor.ir`, `torch._inductor.runtime.runtime_utils`, and `torch._inductor.utils` for the logic in this range. Introduces class `LocalFeedback`, function `__init__`, function `__call__`, class `InconsistentMetadata`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `torch._inductor.ir`、`torch._inductor.runtime.runtime_utils`、`torch._inductor.utils` 等依赖，为后续逻辑提供基础能力。这里定义了类`LocalFeedback`、函数`__init__`、函数`__call__`、类`InconsistentMetadata`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 41-60 / 第 41-60 行
````python
    """
    Exception that is thrown when AutoHeuristic tries to log data to a file where the metadata stored in the file does
    not match the metadata it would store if the file didn't exist.
    """


class AutoHeuristic:
    """
    AutoHeuristic is a framework that allows one to collect data, learn a heuristic (i.e. a regression tree) and
    generate the heuristic to code. This class allows one to collect data. The collected data can then be used to train
    a heuristic (see torchgen/autoheuristic/).
    """

    collected_feedback: dict[Choice, Feedback]

    def __init__(
        self,
        fallback: Callable[[], Choice],
        choices: list[Choice],
        feedback: LocalFeedback | None,
````
- **EN**: Introduces class `AutoHeuristic`, function `__init__`. Initializes or updates values such as `collected_feedback`, `fallback`, `choices`, and `feedback`.
- **CN**: 这里定义了类`AutoHeuristic`、函数`__init__`。初始化或更新了 `collected_feedback`、`fallback`、`choices`、`feedback` 等值。

### Lines 61-80 / 第 61-80 行
````python
        context: AHContext,
        name: str,
        augment_context: list[AHOperation] | None = None,
        precondition: Callable[[AHMetadata, AHContext], bool] | None = None,
    ) -> None:
        """
        Initializes an instance of the AutoHeuristic class.

        Args:
            fallback: A callable that returns a Choice when the heuristic is unsure which choice to make, or
            AutoHeuristic is in data collection mode.
            choices: A list of possible choices the heuristic can make.
            feedback: An instance of LocalFeedback that provides feedback for a given choice.
            context: Context to store with each choice and feedback.
            name: A string that identifies the heuristic.
            augment_context: An optional list of AHOperation instances that augment the context.
            precondition: A callable that returns a boolean indicating whether AutoHeuristic should run.
        """
        self.fallback = fallback
        self.choices = choices
````
- **EN**: Initializes or updates values such as `context`, `name`, `augment_context`, `precondition`, `Args`, `fallback`, and `...+2`. This range continues the implementation of function `AutoHeuristic.__init__`.
- **CN**: 初始化或更新了 `context`、`name`、`augment_context`、`precondition`、`Args`、`fallback`、`另有2项` 等值。这一段延续了函数`AutoHeuristic.__init__` 的具体实现。

### Lines 81-100 / 第 81-100 行
````python
        self.feedback = feedback
        self.context = context
        self.name = name
        self.collected_feedback = {}
        self.augment_context = augment_context
        self.metadata = AHMetadata(
            get_gpu_shared_memory(),
            torch.cuda.get_device_capability(),
            self.choices,
            self.name,
        )
        self.precondition = precondition

        if not self.satisfies_precondition():
            return

        if torch._inductor.config.autoheuristic_log_path == "DEFAULT":
            self.log_path = self.get_default_log_path()
        else:
            self.log_path = torch._inductor.config.autoheuristic_log_path
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。

### Lines 101-120 / 第 101-120 行
````python

        if torch._inductor.config.collect_autoheuristic(self.name):
            if self.feedback is not None:
                for choice in self.choices:
                    feedback_val = self.feedback(choice)
                    self.save_data(choice, feedback_val)

    def satisfies_precondition(self) -> bool:
        return self.precondition is None or self.precondition(
            self.metadata, self.context
        )

    def get_choice(self) -> Choice:
        """
        Returns the chosen option based on the value of autoheuristic_use.
        If self.name is one of the comma separated strings in autoheuristic_use,
        it queries a learned heuristic to make a decision. Otherwise, it returns the fallback option.
        """

        if not self.satisfies_precondition():
````
- **EN**: Introduces function `satisfies_precondition`, function `get_choice`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `feedback_val`.
- **CN**: 这里定义了函数`satisfies_precondition`、函数`get_choice`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `feedback_val` 等值。

### Lines 121-140 / 第 121-140 行
````python
            return self.fallback()

        if torch._inductor.config.use_autoheuristic(self.name):
            if self.augment_context is not None:
                self.context.apply_operations(self.augment_context)
            controller = LearnedHeuristicController(
                self.metadata,
                self.context,
            )
            decision = controller.get_decision()
            if decision not in self.choices:
                # TODO(AlnisM): We might want to allow this in the future
                return self.fallback()
            if decision is not None:
                return decision
        return self.fallback()

    def get_top_k_choices(
        self, top_k: int, always_included: list[str] | None = None
    ) -> list[Choice] | None:
````
- **EN**: Introduces function `get_top_k_choices`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `controller`, and `decision`.
- **CN**: 这里定义了函数`get_top_k_choices`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `controller`、`decision` 等值。

### Lines 141-160 / 第 141-160 行
````python
        if not self.satisfies_precondition():
            return None
        if torch._inductor.config.use_autoheuristic(self.name):
            if self.augment_context is not None:
                self.context.apply_operations(self.augment_context)
            controller = LearnedHeuristicController(
                self.metadata,
                self.context,
            )
            choices = controller.get_decisions_ranked(top_k)
            if choices is None:
                return None
            if always_included is not None:
                for choice in always_included:
                    if choice not in choices:
                        choices.append(choice)
            return choices
        return None

    def get_collected_feedback(self, choice: Choice) -> Any:
````
- **EN**: Introduces function `get_collected_feedback`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `controller`, and `choices`.
- **CN**: 这里定义了函数`get_collected_feedback`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `controller`、`choices` 等值。

### Lines 161-180 / 第 161-180 行
````python
        return self.collected_feedback.get(choice, None)

    @staticmethod
    def get_device_identifier() -> str:
        # a heuristic might work well for one GPU, but not for another
        # we store the collected data per GPU model and learn a heuristic per GPU model

        # TODO(AlnisM): just using the device name for now, but the same GPU model can have different names
        device_name = torch.cuda.get_device_name().replace(" ", "_")
        return device_name

    def get_default_log_path(self) -> str:
        device_name = self.get_device_identifier()
        path = f"{cache_dir()}/autoheuristic/{device_name}/"
        os.makedirs(path, exist_ok=True)
        path += f"{self.name}.txt"
        return path

    def serialize_metadata(self) -> str:
        metadata_dict = self.metadata.to_dict()
````
- **EN**: Introduces function `get_device_identifier`, function `get_default_log_path`, function `serialize_metadata`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`get_device_identifier`、函数`get_default_log_path`、函数`serialize_metadata`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。

### Lines 181-200 / 第 181-200 行
````python
        (
            num_features,
            cat_features,
        ) = self.context.get_numerical_and_categorical_features()
        metadata_dict["numerical_features"] = num_features
        metadata_dict["categorical_features"] = cat_features
        return json.dumps(metadata_dict)

    def save_data(self, choice: Choice, feedback_val: Feedback) -> None:
        self.collected_feedback[choice] = feedback_val
        log_path = self.log_path

        lines = []
        log_exists = os.path.exists(log_path)
        if log_exists:
            # if log already exists, make sure it is consistent
            metadata = self.serialize_metadata()
            existing_metadata = get_metadata_str_from_log(self.log_path)
            if existing_metadata != metadata:
                raise InconsistentMetadata(
````
- **EN**: Introduces function `save_data`. Serializes or deserializes JSON data as part of persistence or interchange. Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`save_data`。通过 JSON 的序列化或反序列化来完成持久化或数据交换。这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 201-220 / 第 201-220 行
````python
                    "Given metadata does not match existing metadata"
                )
        else:
            lines.append(self.serialize_metadata())
            feature_header = self.context.get_feature_names_csv()
            header = feature_header + "," + CHOICE_COL + "," + FEEDBACK_COL
            lines.append(header)

        line = ""
        feature_values = self.context.get_feature_values_csv()
        line += feature_values + "," + choice + "," + str(feedback_val)
        lines.append(line)

        with open(log_path, "a") as f:
            f.write("\n".join(lines) + "\n")


class AutoHeuristicSelectAlgorithm(AutoHeuristic):
    """
    AutoHeuristicSelectAlgorithm is a subclass of AutoHeuristic that allows one to collect data and learn a heuristic
````
- **EN**: Introduces class `AutoHeuristicSelectAlgorithm`. Touches the filesystem to load, validate, or store compiler artifacts. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `feature_header`, `header`, `line`, and `feature_values`.
- **CN**: 这里定义了类`AutoHeuristicSelectAlgorithm`。这一段会访问文件系统，用于加载、校验或保存编译产物。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`feature_header`、`header`、`line`、`feature_values` 等值。

### Lines 221-240 / 第 221-240 行
````python
    when one wants to use AutoHeuristic for kernel choice selection.
    """

    def __init__(
        self,
        fallback: Callable[[], ChoiceCaller | None],
        choices: list[ChoiceCaller],
        input_nodes: list[Any],
        context: AHContext,
        name: str,
        augment_context: list[AHOperation] | None = None,
        precondition: Callable[[AHMetadata, AHContext], bool] | None = None,
    ) -> None:
        """
        The arguments choices, input_nodes and name have to match the ones used in the call to
        autotune_select_algorithm(), e.g. if the following call is made
        autotune_select_algorithm(name, choices, input_nodes, layout), the same name, choices and input_nodes
        have to be used here.
        """
        self.input_nodes = input_nodes
````
- **EN**: Introduces function `__init__`. Initializes or updates values such as `fallback`, `choices`, `input_nodes`, `context`, `name`, `augment_context`, and `...+1`.
- **CN**: 这里定义了函数`__init__`。初始化或更新了 `fallback`、`choices`、`input_nodes`、`context`、`name`、`augment_context`、`另有1项` 等值。

### Lines 241-260 / 第 241-260 行
````python
        self.choicestr2choice: dict[str, ChoiceCaller] = {}
        for choice in choices:
            self.choicestr2choice[choice.autoheuristic_id()] = choice
        choices_str = list(self.choicestr2choice.keys())

        def fallback_str() -> str:
            fallback_choice = fallback()
            if fallback_choice is None:
                # TODO: Find a nicer way to handle this
                return "unsure"
            return fallback_choice.autoheuristic_id()

        super().__init__(
            fallback_str,
            choices_str,
            None,
            context,
            name,
            augment_context,
            precondition,
````
- **EN**: Introduces function `fallback_str`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `choices_str`, and `fallback_choice`.
- **CN**: 这里定义了函数`fallback_str`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `choices_str`、`fallback_choice` 等值。

### Lines 261-280 / 第 261-280 行
````python
        )

        if (
            torch._inductor.config.collect_autoheuristic(self.name)
            and self.satisfies_precondition()
        ):
            self.register_global_feedback(input_nodes, choices)

    def register_global_feedback(
        self, input_nodes: list[Any], choices: list[ChoiceCaller]
    ) -> None:
        """
        Registers a callback in select_algorithm, which is called with the timing of each choice.
        """

        from torch._inductor.select_algorithm import (
            add_feedback_saver,
            create_inputs_key,
            create_precompile_key,
        )
````
- **EN**: Imports dependencies such as `torch._inductor.select_algorithm` for the logic in this range. Introduces function `register_global_feedback`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._inductor.select_algorithm` 等依赖，为后续逻辑提供基础能力。这里定义了函数`register_global_feedback`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 281-300 / 第 281-300 行
````python

        def store_global_feedback(
            ah_inputs_key: str,
            ah_precompile_key: str,
            timings: dict[ChoiceCaller, float],
            name: str,
            input_nodes: list[Any],
            choices: list[ChoiceCaller],
            profiled_time: Callable[[], dict[ChoiceCaller, float]],
            precompile_times: dict[ChoiceCaller, float],
        ) -> None:
            current_inputs_key = create_inputs_key(input_nodes)
            if current_inputs_key != ah_inputs_key:
                return
            current_precompile_key = create_precompile_key(
                name, current_inputs_key, choices
            )
            if current_precompile_key != ah_precompile_key:
                return
            for choice, time in timings.items():
````
- **EN**: Introduces function `store_global_feedback`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ah_inputs_key`, `ah_precompile_key`, `timings`, `name`, `input_nodes`, `choices`, and `...+4`.
- **CN**: 这里定义了函数`store_global_feedback`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `ah_inputs_key`、`ah_precompile_key`、`timings`、`name`、`input_nodes`、`choices`、`另有4项` 等值。

### Lines 301-318 / 第 301-318 行
````python
                self.save_data(choice.autoheuristic_id(), time)

        inputs_key = create_inputs_key(input_nodes)
        precompile_key = create_precompile_key(self.name, inputs_key, choices)
        feedback_saver = partial(store_global_feedback, inputs_key, precompile_key)
        add_feedback_saver(feedback_saver)

    def get_choice_caller(self) -> ChoiceCaller | None:
        choice = self.get_choice()
        return self.choicestr2choice.get(choice, None)

    def get_top_k_choices_caller(
        self, top_k: int, always_included: list[str] | None = None
    ) -> list[ChoiceCaller] | None:
        choices = self.get_top_k_choices(top_k, always_included)
        if choices is None:
            return None
        return [self.choicestr2choice[choice] for choice in choices]
````
- **EN**: Introduces function `get_choice_caller`, function `get_top_k_choices_caller`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `inputs_key`, `precompile_key`, `feedback_saver`, `choice`, and `choices`.
- **CN**: 这里定义了函数`get_choice_caller`、函数`get_top_k_choices_caller`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `inputs_key`、`precompile_key`、`feedback_saver`、`choice`、`choices` 等值。

## Key Concepts / 关键概念
- **EN**: Contains heuristic models and helpers for autotuning decisions  
  **CN**: 包含自动调优决策所需的启发式模型与辅助逻辑
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Profiling data handling  
  **CN**: 性能画像数据处理
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Autotuning support  
  **CN**: 自动调优支持
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Primary classes: `LocalFeedback`, `InconsistentMetadata`, `AutoHeuristic`, and `AutoHeuristicSelectAlgorithm`  
  **CN**: 主要类：`LocalFeedback`、`InconsistentMetadata`、`AutoHeuristic`、`AutoHeuristicSelectAlgorithm`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `collections.abc`, `functools`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor.autoheuristic.autoheuristic_utils`, `torch._inductor.autoheuristic.learned_heuristic_controller`, `torch._inductor.ir`, `torch._inductor.runtime.runtime_utils`, `torch._inductor.utils`, `torch._inductor.select_algorithm`
