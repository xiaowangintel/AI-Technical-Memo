# graph_settings.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/utils/data/graph_settings.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements data-loading infrastructure, datapipes, samplers, worker coordination, and dataset traversal utilities. This specific file centers on `graph_settings.py`. Dataset iteration, batching, and worker orchestration are central concerns here.
- **Purpose (CN) / 用途（中文）**: 实现数据加载基础设施、datapipes、sampler、worker 协调以及数据集遍历工具。 该文件具体围绕 `graph_settings.py` 展开。 数据集迭代、批处理以及 worker 协调是这里的核心关注点。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行
```python
# mypy: allow-untyped-defs
import inspect
import warnings
from typing import Any
from typing_extensions import deprecated

import torch
from torch.utils.data.datapipes.iter.sharding import (
    _ShardingIterDataPipe,
    SHARDING_PRIORITIES,
)
from torch.utils.data.graph import DataPipe, DataPipeGraph, traverse_dps
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch, torch.utils.data.datapipes.iter.sharding:_ShardingIterDataPipe, torch.utils.data.datapipes.iter.sharding:SHARDING_PRIORITIES, torch.utils.data.graph:DataPipe; standard-library helpers such as inspect, warnings, typing:Any; external packages such as typing_extensions:deprecated. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch, torch.utils.data.datapipes.iter.sharding:_ShardingIterDataPipe, torch.utils.data.datapipes.iter.sharding:SHARDING_PRIORITIES, torch.utils.data.graph:DataPipe；标准库辅助模块，如 inspect, warnings, typing:Any；外部包，如 typing_extensions:deprecated。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 15-25 / 第 15-25 行
```python
__all__ = [
    "apply_random_seed",
    "apply_sharding",
    "apply_shuffle_seed",
    "apply_shuffle_settings",
    "get_all_graph_pipes",
]


def get_all_graph_pipes(graph: DataPipeGraph) -> list[DataPipe]:
    return _get_all_graph_pipes_helper(graph, set())
```
- **EN**: The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. Key callable entry points in this range include `get_all_graph_pipes`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 这一段的重要可调用入口包括 `get_all_graph_pipes`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 28-38 / 第 28-38 行
```python
def _get_all_graph_pipes_helper(
    graph: DataPipeGraph, id_cache: set[int]
) -> list[DataPipe]:
    results: list[DataPipe] = []
    for dp_id, (datapipe, sub_graph) in graph.items():
        if dp_id in id_cache:
            continue
        id_cache.add(dp_id)
        results.append(datapipe)
        results.extend(_get_all_graph_pipes_helper(sub_graph, id_cache))
    return results
```
- **EN**: Key callable entry points in this range include `_get_all_graph_pipes_helper`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `_get_all_graph_pipes_helper`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 41-55 / 第 41-55 行
```python
def _is_sharding_datapipe(datapipe: DataPipe) -> bool:
    return isinstance(datapipe, _ShardingIterDataPipe) or (
        hasattr(datapipe, "apply_sharding")
        and inspect.ismethod(datapipe.apply_sharding)
    )


def apply_sharding(
    datapipe: DataPipe,
    num_of_instances: int,
    instance_id: int,
    sharding_group=SHARDING_PRIORITIES.DEFAULT,
) -> DataPipe:
    r"""
    Apply dynamic sharding over the ``sharding_filter`` DataPipe that has a method ``apply_sharding``.
```
- **EN**: Key callable entry points in this range include `_is_sharding_datapipe`, `apply_sharding`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `_is_sharding_datapipe`, `apply_sharding`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 57-74 / 第 57-74 行
```python
    RuntimeError will be raised when multiple ``sharding_filter`` are presented in the same branch.
    """
    graph = traverse_dps(datapipe)

    def _helper(graph, prev_applied=None) -> None:
        for dp, sub_graph in graph.values():
            applied = None
            if _is_sharding_datapipe(dp):
                if prev_applied is not None:
                    raise RuntimeError(
                        "Sharding twice on a single pipeline is likely unintended and will cause data loss. "
                        f"Sharding already applied to {prev_applied} while trying to apply to {dp}"
                    )
                # For BC, only provide sharding_group if accepted
                sig = inspect.signature(dp.apply_sharding)
                if len(sig.parameters) < 3:
                    dp.apply_sharding(num_of_instances, instance_id)
                else:
```
- **EN**: Key callable entry points in this range include `apply_sharding`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `apply_sharding`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 75-85 / 第 75-85 行
```python
                    dp.apply_sharding(
                        num_of_instances, instance_id, sharding_group=sharding_group
                    )
                applied = dp
            if applied is None:
                applied = prev_applied
            _helper(sub_graph, applied)

    _helper(graph)

    return datapipe
```
- **EN**: Key callable entry points in this range include `apply_sharding`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `apply_sharding`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 88-102 / 第 88-102 行
```python
def _is_shuffle_datapipe(datapipe: DataPipe) -> bool:
    return (
        hasattr(datapipe, "set_shuffle")
        and hasattr(datapipe, "set_seed")
        and inspect.ismethod(datapipe.set_shuffle)
        and inspect.ismethod(datapipe.set_seed)
    )


def apply_shuffle_settings(datapipe: DataPipe, shuffle: bool | None = None) -> DataPipe:
    r"""
    Traverse the graph of ``DataPipes`` to find and set shuffle attribute.

    Apply the method to each `DataPipe` that has APIs of ``set_shuffle``
    and ``set_seed``.
```
- **EN**: Key callable entry points in this range include `_is_shuffle_datapipe`, `apply_shuffle_settings`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `_is_shuffle_datapipe`, `apply_shuffle_settings`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 104-121 / 第 104-121 行
```python
    Args:
        datapipe: DataPipe that needs to set shuffle attribute
        shuffle: Shuffle option (default: ``None`` and no-op to the graph)
    """
    if shuffle is None:
        return datapipe

    graph = traverse_dps(datapipe)
    all_pipes = get_all_graph_pipes(graph)
    shufflers = [pipe for pipe in all_pipes if _is_shuffle_datapipe(pipe)]
    if not shufflers and shuffle:
        warnings.warn(
            "`shuffle=True` was set, but the datapipe does not contain a `Shuffler`. Adding one at the end. "
            "Be aware that the default buffer size might not be sufficient for your task.",
            stacklevel=2,
        )
        datapipe = datapipe.shuffle()
        shufflers = [
```
- **EN**: Key callable entry points in this range include `apply_shuffle_settings`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `apply_shuffle_settings`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 122-137 / 第 122-137 行
```python
            datapipe,
        ]

    for shuffler in shufflers:
        shuffler.set_shuffle(shuffle)

    return datapipe


@deprecated(
    "`apply_shuffle_seed` is deprecated since 1.12 and will be removed in the future releases. "
    "Please use `apply_random_seed` instead.",
    category=FutureWarning,
)
def apply_shuffle_seed(datapipe: DataPipe, rng: Any) -> DataPipe:
    return apply_random_seed(datapipe, rng)
```
- **EN**: Key callable entry points in this range include `apply_shuffle_settings`, `apply_shuffle_seed`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `apply_shuffle_settings`, `apply_shuffle_seed`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 140-157 / 第 140-157 行
```python
def _is_random_datapipe(datapipe: DataPipe) -> bool:
    return hasattr(datapipe, "set_seed") and inspect.ismethod(datapipe.set_seed)


def apply_random_seed(datapipe: DataPipe, rng: torch.Generator) -> DataPipe:
    r"""
    Traverse the graph of ``DataPipes`` to find random ``DataPipe`` with an API of ``set_seed``.

    Then set the random seed based on the provided RNG to those ``DataPipe``.

    Args:
        datapipe: DataPipe that needs to set randomness
        rng: Random number generator to generate random seeds
    """
    graph = traverse_dps(datapipe)
    all_pipes = get_all_graph_pipes(graph)
    # Using a set to track id of DataPipe to prevent setting randomness per DataPipe more than once.
    # And, `id` is used in case of unhashable DataPipe
```
- **EN**: Key callable entry points in this range include `_is_random_datapipe`, `apply_random_seed`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `_is_random_datapipe`, `apply_random_seed`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 158-173 / 第 158-173 行
```python
    cache = set()
    random_datapipes = []
    for pipe in all_pipes:
        if id(pipe) in cache:
            continue
        if _is_random_datapipe(pipe):
            random_datapipes.append(pipe)
            cache.add(id(pipe))

    for pipe in random_datapipes:
        random_seed = int(
            torch.empty((), dtype=torch.int64).random_(generator=rng).item()
        )
        pipe.set_seed(random_seed)

    return datapipe
```
- **EN**: Key callable entry points in this range include `apply_random_seed`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `apply_random_seed`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements data-loading infrastructure, datapipes, samplers, worker coordination, and dataset traversal utilities.
  - CN: 实现数据加载基础设施、datapipes、sampler、worker 协调以及数据集遍历工具。
- **Graph rewriting**
  - EN: Rewrites module or FX graph structure so later execution follows a transformed pipeline.
  - CN: 改写模块或 FX 图结构，使后续执行遵循转换后的流水线。
- **DataPipes**
  - EN: Builds composable input pipelines that lazily transform or route dataset elements.
  - CN: 构建可组合的输入流水线，以惰性方式变换或路由数据集元素。
- **TensorBoard integration**
  - EN: Exports model, scalar, and graph information into TensorBoard-consumable formats.
  - CN: 把模型、标量与图信息导出为 TensorBoard 可消费的格式。
- **Serialization safety**
  - EN: Keeps persisted data exchange bounded, predictable, and easier to validate.
  - CN: 让持久化数据交换保持有界、可预测且更易验证。
- **get_all_graph_pipes**
  - EN: `get_all_graph_pipes` is a representative function that exposes or coordinates an important action in this module.
  - CN: `get_all_graph_pipes` 是一个具有代表性的函数，用于暴露或协调此模块中的关键动作。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.utils.data.datapipes.iter.sharding:_ShardingIterDataPipe`, `torch.utils.data.datapipes.iter.sharding:SHARDING_PRIORITIES`, `torch.utils.data.graph:DataPipe`, `torch.utils.data.graph:DataPipeGraph`, `torch.utils.data.graph:traverse_dps`
- **Python standard library / Python 标准库**: `inspect`, `warnings`, `typing:Any`
- **Third-party packages / 第三方包**: `typing_extensions:deprecated`
- **Explicit exports / 显式导出**: `apply_random_seed`, `apply_sharding`, `apply_shuffle_seed`, `apply_shuffle_settings`, `get_all_graph_pipes`
- **Primary symbols / 核心符号**: `get_all_graph_pipes`, `_get_all_graph_pipes_helper`, `_is_sharding_datapipe`, `apply_sharding`, `_is_shuffle_datapipe`, `apply_shuffle_settings`, `apply_shuffle_seed`, `_is_random_datapipe`, `apply_random_seed`
