# base_structured_sparsifier.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/pruning/_experimental/pruner/base_structured_sparsifier.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements pruning and sparsity utilities that remove or mask parameters while preserving model structure. This specific file centers on `base_structured_sparsifier.py`. Runtime configuration, environment overrides, or feature gating are important in this module.
- **Purpose (CN) / 用途（中文）**: 实现剪枝与稀疏化工具，在保留模型结构的同时移除或屏蔽参数。 该文件具体围绕 `base_structured_sparsifier.py` 展开。 运行时配置、环境变量覆盖或特性开关是此模块的重要主题。
## Line-by-Line Analysis / 逐行分析

### Lines 1-24 / 第 1-24 行
```python
# mypy: allow-untyped-defs
from collections.abc import Callable
from itertools import chain
from operator import getitem

import torch
import torch.nn.functional as F
from torch import nn
from torch.ao.pruning.sparsifier.base_sparsifier import BaseSparsifier
from torch.fx import symbolic_trace
from torch.nn.utils import parametrize

from .match_utils import apply_match, MatchAllNode
from .parametrization import BiasHook, FakeStructuredSparsity, module_contains_param
from .prune_functions import (
    prune_conv2d,
    prune_conv2d_activation_conv2d,
    prune_conv2d_activation_pool_conv2d,
    prune_conv2d_conv2d,
    prune_conv2d_pool_activation_conv2d,
    prune_conv2d_pool_flatten_linear,
    prune_linear,
    prune_linear_activation_linear,
    prune_linear_linear,
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch, torch.nn.functional, torch:nn, torch.ao.pruning.sparsifier.base_sparsifier:BaseSparsifier; standard-library helpers such as collections.abc:Callable, itertools:chain, operator:getitem. Hook registration/order is important because the value comes from when callbacks fire, not just what they compute.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch, torch.nn.functional, torch:nn, torch.ao.pruning.sparsifier.base_sparsifier:BaseSparsifier；标准库辅助模块，如 collections.abc:Callable, itertools:chain, operator:getitem。 钩子的价值不仅在于计算什么，更在于何时触发以及触发顺序。

### Lines 25-48 / 第 25-48 行
```python
    prune_lstm_output_layernorm_linear,
    prune_lstm_output_linear,
)


def _get_supported_structured_pruning_modules():
    SUPPORTED_STRUCTURED_PRUNING_MODULES = {  # added to config if None given
        nn.Linear,
        nn.Conv2d,
        nn.LSTM,
    }
    return SUPPORTED_STRUCTURED_PRUNING_MODULES


def _get_supported_activation_functions():
    SUPPORTED_ACTIVATION_FUNCTIONS = {
        F.relu,
        F.rrelu,
        F.hardtanh,
        F.relu6,
        F.sigmoid,
        F.hardsigmoid,
        F.tanh,
        F.silu,
```
- **EN**: Key callable entry points in this range include `_get_supported_structured_pruning_modules`, `_get_supported_activation_functions`. They package a focused unit of behavior behind named helpers or APIs. Named constants such as `SUPPORTED_STRUCTURED_PRUNING_MODULES`, `SUPPORTED_ACTIVATION_FUNCTIONS` centralize shared configuration or sentinel values. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. The logic controls configuration state, making precedence and override rules especially important.
- **CN**: 这一段的重要可调用入口包括 `_get_supported_structured_pruning_modules`, `_get_supported_activation_functions`，它们把聚焦的行为封装成具名辅助函数或 API。 `SUPPORTED_STRUCTURED_PRUNING_MODULES, SUPPORTED_ACTIVATION_FUNCTIONS` 等具名常量把共享配置或哨兵值集中定义在一起。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 该逻辑控制配置状态，因此优先级和覆盖规则尤为重要。

### Lines 49-63 / 第 49-63 行
```python
        F.mish,
        F.hardswish,
        F.elu,
        F.celu,
        F.selu,
        F.hardshrink,
        F.leaky_relu,
        F.logsigmoid,
        F.softplus,
        F.prelu,
        F.softsign,
        F.tanhshrink,
        F.gelu,
    }
    return SUPPORTED_ACTIVATION_FUNCTIONS
```
- **EN**: Key callable entry points in this range include `_get_supported_activation_functions`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `_get_supported_activation_functions`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 66-89 / 第 66-89 行
```python
def _get_supported_activation_modules():
    SUPPORTED_ACTIVATION_MODULES = {
        nn.ReLU,
        nn.RReLU,
        nn.Hardtanh,
        nn.ReLU6,
        nn.Sigmoid,
        nn.Hardsigmoid,
        nn.Tanh,
        nn.SiLU,
        nn.Mish,
        nn.Hardswish,
        nn.ELU,
        nn.CELU,
        nn.SELU,
        nn.Hardshrink,
        nn.LeakyReLU,
        nn.LogSigmoid,
        nn.Softplus,
        nn.PReLU,
        nn.Softsign,
        nn.Tanhshrink,
        nn.GELU,
    }
```
- **EN**: Key callable entry points in this range include `_get_supported_activation_modules`. They package a focused unit of behavior behind named helpers or APIs. Named constants such as `SUPPORTED_ACTIVATION_MODULES` centralize shared configuration or sentinel values.
- **CN**: 这一段的重要可调用入口包括 `_get_supported_activation_modules`，它们把聚焦的行为封装成具名辅助函数或 API。 `SUPPORTED_ACTIVATION_MODULES` 等具名常量把共享配置或哨兵值集中定义在一起。

### Lines 90-113 / 第 90-113 行
```python
    return SUPPORTED_ACTIVATION_MODULES


def _get_default_structured_pruning_patterns() -> dict[
    tuple[type[nn.Module] | Callable | MatchAllNode | str, ...],
    Callable[..., None],
]:
    """
    Returns the patterns for conv2d / linear conversion for each element in the activation functions/modules defined above.
    """
    patterns: dict[
        tuple[type[nn.Module] | Callable | MatchAllNode | str, ...],
        Callable[..., None],
    ] = {
        # linear -> linear
        (nn.Linear, "output"): prune_linear,
        (nn.Linear, nn.Linear): prune_linear_linear,
        # conv2d -> conv2d
        (nn.Conv2d, "output"): prune_conv2d,
        (nn.Conv2d, nn.Conv2d): prune_conv2d_conv2d,
        # TODO LSTM Structured pruning does not support returned state currently.
        # Should find a way to explicitly match getitem(0) instead of getitem.
        # This will also require changing the pruning function.
        # lstm -> getitem(0) -> linear
```
- **EN**: Key callable entry points in this range include `_get_supported_activation_modules`, `_get_default_structured_pruning_patterns`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `_get_supported_activation_modules`, `_get_default_structured_pruning_patterns`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 114-137 / 第 114-137 行
```python
        (nn.LSTM, getitem, nn.Linear): prune_lstm_output_linear,
        # lstm -> getitem(0) -> layernorm -> linear
        (nn.LSTM, getitem, nn.LayerNorm, nn.Linear): prune_lstm_output_layernorm_linear,
    }

    for activation in chain(
        _get_supported_activation_functions(), _get_supported_activation_modules()
    ):
        patterns.update(
            {
                # linear -> activation -> linear
                (nn.Linear, activation, nn.Linear): prune_linear_activation_linear,
                # conv2d -> activation -> conv2d
                (nn.Conv2d, activation, nn.Conv2d): prune_conv2d_activation_conv2d,
                # conv2d -> activation -> pool -> conv2d
                (
                    nn.Conv2d,
                    activation,
                    nn.AvgPool2d,
                    nn.Conv2d,
                ): prune_conv2d_activation_pool_conv2d,
                (
                    nn.Conv2d,
                    activation,
```
- **EN**: Key callable entry points in this range include `_get_default_structured_pruning_patterns`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way.
- **CN**: 这一段的重要可调用入口包括 `_get_default_structured_pruning_patterns`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。

### Lines 138-161 / 第 138-161 行
```python
                    F.avg_pool2d,
                    nn.Conv2d,
                ): prune_conv2d_activation_pool_conv2d,
                (
                    nn.Conv2d,
                    activation,
                    nn.MaxPool2d,
                    nn.Conv2d,
                ): prune_conv2d_activation_pool_conv2d,
                (
                    nn.Conv2d,
                    activation,
                    F.max_pool2d,
                    nn.Conv2d,
                ): prune_conv2d_activation_pool_conv2d,
                # conv2d -> pool -> activation -> conv2d
                (
                    nn.Conv2d,
                    nn.AvgPool2d,
                    activation,
                    nn.Conv2d,
                ): prune_conv2d_pool_activation_conv2d,
                (
                    nn.Conv2d,
```
- **EN**: Key callable entry points in this range include `_get_default_structured_pruning_patterns`. They package a focused unit of behavior behind named helpers or APIs.
- **CN**: 这一段的重要可调用入口包括 `_get_default_structured_pruning_patterns`，它们把聚焦的行为封装成具名辅助函数或 API。

### Lines 162-185 / 第 162-185 行
```python
                    F.avg_pool2d,
                    activation,
                    nn.Conv2d,
                ): prune_conv2d_pool_activation_conv2d,
                (
                    nn.Conv2d,
                    nn.MaxPool2d,
                    activation,
                    nn.Conv2d,
                ): prune_conv2d_pool_activation_conv2d,
                (
                    nn.Conv2d,
                    F.max_pool2d,
                    activation,
                    nn.Conv2d,
                ): prune_conv2d_pool_activation_conv2d,
                # conv2d -> adaptive pool -> flatten -> linear
                (
                    nn.Conv2d,
                    nn.AdaptiveAvgPool2d,
                    nn.Flatten,
                    nn.Linear,
                ): prune_conv2d_pool_flatten_linear,
                (
```
- **EN**: Key callable entry points in this range include `_get_default_structured_pruning_patterns`. They package a focused unit of behavior behind named helpers or APIs.
- **CN**: 这一段的重要可调用入口包括 `_get_default_structured_pruning_patterns`，它们把聚焦的行为封装成具名辅助函数或 API。

### Lines 186-205 / 第 186-205 行
```python
                    nn.Conv2d,
                    nn.AdaptiveAvgPool2d,
                    torch.flatten,
                    nn.Linear,
                ): prune_conv2d_pool_flatten_linear,
                (
                    nn.Conv2d,
                    nn.AdaptiveMaxPool2d,
                    nn.Flatten,
                    nn.Linear,
                ): prune_conv2d_pool_flatten_linear,
                (
                    nn.Conv2d,
                    nn.AdaptiveMaxPool2d,
                    torch.flatten,
                    nn.Linear,
                ): prune_conv2d_pool_flatten_linear,
            }
        )
    return patterns
```
- **EN**: Key callable entry points in this range include `_get_default_structured_pruning_patterns`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `_get_default_structured_pruning_patterns`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 208-225 / 第 208-225 行
```python
class BaseStructuredSparsifier(BaseSparsifier):
    r"""Base class for structured pruning.

    Abstract methods that need to be implemented:
        - update_mask: Function to compute a new mask for all keys in the
            `groups` attribute.

    Args:
        - defaults [dict]: default configurations will be attached to the
            configuration. Only the keys that don't exist in the `config` will
            be updated.
    """

    def __init__(self, defaults, patterns=None):
        super().__init__(defaults)
        if patterns is None:
            patterns = _get_default_structured_pruning_patterns()
        self.patterns = patterns
```
- **EN**: It introduces or extends class-level abstractions such as `BaseStructuredSparsifier`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The logic controls configuration state, making precedence and override rules especially important.
- **CN**: 它引入或扩展了 `BaseStructuredSparsifier` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该逻辑控制配置状态，因此优先级和覆盖规则尤为重要。

### Lines 227-244 / 第 227-244 行
```python
    def make_config_from_model(
        self,
        model: nn.Module,
        SUPPORTED_MODULES: set[type] | None = None,
    ) -> None:
        if SUPPORTED_MODULES is None:
            SUPPORTED_MODULES = _get_supported_structured_pruning_modules()
        super().make_config_from_model(model, SUPPORTED_MODULES=SUPPORTED_MODULES)

    def _prepare(self, *args, **kwargs) -> None:
        r"""This function will attach the FakeStructuredSparsity parameterizations
        and BiasHooks at the appropriate points in the model.
        """
        for config in self.groups:
            module = config["module"]
            tensor_name = config["tensor_name"]
            parametrization = config.get("parametrization", FakeStructuredSparsity)
            tensor = getattr(module, tensor_name)
```
- **EN**: It introduces or extends class-level abstractions such as `BaseStructuredSparsifier`, which organize state and behavior for this subsystem. Named constants such as `SUPPORTED_MODULES` centralize shared configuration or sentinel values. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The logic controls configuration state, making precedence and override rules especially important.
- **CN**: 它引入或扩展了 `BaseStructuredSparsifier` 等类级抽象，用于组织该子系统的状态与行为。 `SUPPORTED_MODULES` 等具名常量把共享配置或哨兵值集中定义在一起。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该逻辑控制配置状态，因此优先级和覆盖规则尤为重要。

### Lines 246-268 / 第 246-268 行
```python
            mask = config.get(
                "mask",
                torch.ones(tensor.shape[0], dtype=torch.bool, device=tensor.device),
            )
            self.state[config["tensor_fqn"]]["mask"] = mask
            parametrize.register_parametrization(
                module, tensor_name, parametrization(mask)
            )

            # if linear / conv, we add in bias hooks
            if isinstance(module, (nn.Linear, nn.Conv2d)):
                prune_bias = config.get("prune_bias", True)
                if module.bias is not None:
                    module.register_parameter(
                        "_bias", nn.Parameter(module.bias.detach())
                    )
                    # pyrefly: ignore [bad-assignment]
                    module.bias = None
                    module.prune_bias = prune_bias

                module.register_forward_hook(
                    BiasHook(module.parametrizations.weight[0], prune_bias)  # type: ignore[union-attr, index]
                )
```
- **EN**: It introduces or extends class-level abstractions such as `BaseStructuredSparsifier`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The logic controls configuration state, making precedence and override rules especially important.
- **CN**: 它引入或扩展了 `BaseStructuredSparsifier` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该逻辑控制配置状态，因此优先级和覆盖规则尤为重要。

### Lines 270-288 / 第 270-288 行
```python
    def prune(self) -> None:
        r"""
        This function will FX symbolically trace the model and then find instances of the patterns
        defined in self.patterns (by default SUPPORTED_STRUCTURED_PRUNING_PATTERNS ).

        For each pattern, it will apply to corresponding conversion function, which will modify the output
        and input size expected by the modules within the pattern
        """

        self.traced = symbolic_trace(self.model)
        modules = dict(self.traced.named_modules())

        # Right now we check for matches simply by iterating across all the patterns
        # if this is slow we can store patterns in a trie-structure and modify this code for faster lookup
        for node in self.traced.graph.nodes:
            for pattern, convert_fn in self.patterns.items():
                matched = apply_match(modules, pattern, node, [])
                if matched is None:
                    continue
```
- **EN**: It introduces or extends class-level abstractions such as `BaseStructuredSparsifier`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way.
- **CN**: 它引入或扩展了 `BaseStructuredSparsifier` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。

### Lines 290-310 / 第 290-310 行
```python
                # pyrefly: ignore [no-matching-overload]
                first_module = modules.get(node.target)
                # check if first module exists and has appropriate parameterization, otherwise skip
                if (
                    first_module is not None
                    and parametrize.is_parametrized(first_module)
                    and module_contains_param(first_module, FakeStructuredSparsity)
                ):
                    convert_block = []
                    for node in matched:
                        if node.op == "call_module":
                            convert_block.append(modules.get(node.target))
                        elif node.op == "call_function":
                            convert_block.append(node.target)
                    convert_fn(*convert_block)

        for module in self.traced.modules():
            if module_contains_param(module, FakeStructuredSparsity):
                raise Exception(  # noqa: TRY002
                    f"Error: {module} still contains FakeStructuredSparsity parametrizations!"
                )
```
- **EN**: It introduces or extends class-level abstractions such as `BaseStructuredSparsifier`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 它引入或扩展了 `BaseStructuredSparsifier` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 312-314 / 第 312-314 行
```python
        self.traced.graph.lint()
        self.traced.recompile()
        return self.traced  # type: ignore[return-value]
```
- **EN**: It introduces or extends class-level abstractions such as `BaseStructuredSparsifier`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `BaseStructuredSparsifier` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements pruning and sparsity utilities that remove or mask parameters while preserving model structure.
  - CN: 实现剪枝与稀疏化工具，在保留模型结构的同时移除或屏蔽参数。
- **Graph rewriting**
  - EN: Rewrites module or FX graph structure so later execution follows a transformed pipeline.
  - CN: 改写模块或 FX 图结构，使后续执行遵循转换后的流水线。
- **Pruning and sparsity**
  - EN: Controls parameter masking, sparse layouts, or pruning schedules that reduce effective model size.
  - CN: 控制参数掩码、稀疏布局或剪枝计划，以降低模型的有效规模。
- **Configuration management**
  - EN: Centralizes feature flags, environment overrides, and runtime-config wiring.
  - CN: 集中管理特性开关、环境变量覆盖以及运行时配置接线。
- **Pytree processing**
  - EN: Manipulates nested Python container structure while preserving leaves and shape metadata.
  - CN: 在保留叶子节点与结构元数据的同时处理嵌套 Python 容器。
- **Hooks and callbacks**
  - EN: Coordinates callback registration and ordered execution around framework events.
  - CN: 协调回调注册以及围绕框架事件的有序执行。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.nn.functional`, `torch:nn`, `torch.ao.pruning.sparsifier.base_sparsifier:BaseSparsifier`, `torch.fx:symbolic_trace`, `torch.nn.utils:parametrize`, `.match_utils:apply_match`, `.match_utils:MatchAllNode`, `.parametrization:BiasHook`, `.parametrization:FakeStructuredSparsity`, `.parametrization:module_contains_param`, `.prune_functions:prune_conv2d`, `.prune_functions:prune_conv2d_activation_conv2d`, `.prune_functions:prune_conv2d_activation_pool_conv2d`, `.prune_functions:prune_conv2d_conv2d`
- **Python standard library / Python 标准库**: `collections.abc:Callable`, `itertools:chain`, `operator:getitem`
- **Primary symbols / 核心符号**: `BaseStructuredSparsifier`, `_get_supported_structured_pruning_modules`, `_get_supported_activation_functions`, `_get_supported_activation_modules`, `_get_default_structured_pruning_patterns`
