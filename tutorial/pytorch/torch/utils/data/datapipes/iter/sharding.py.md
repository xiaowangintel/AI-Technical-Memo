# sharding.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/utils/data/datapipes/iter/sharding.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements data-loading infrastructure, datapipes, samplers, worker coordination, and dataset traversal utilities. This specific file centers on `sharding.py`. Dataset iteration, batching, and worker orchestration are central concerns here.
- **Purpose (CN) / 用途（中文）**: 实现数据加载基础设施、datapipes、sampler、worker 协调以及数据集遍历工具。 该文件具体围绕 `sharding.py` 展开。 数据集迭代、批处理以及 worker 协调是这里的核心关注点。
## Line-by-Line Analysis / 逐行分析

### Lines 1-13 / 第 1-13 行
```python
# mypy: allow-untyped-defs
from collections.abc import Sized
from enum import IntEnum
from typing import NoReturn

from torch.utils.data.datapipes._decorator import functional_datapipe
from torch.utils.data.datapipes.datapipe import IterDataPipe


__all__ = [
    "SHARDING_PRIORITIES",
    "ShardingFilterIterDataPipe",
]
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch.utils.data.datapipes._decorator:functional_datapipe, torch.utils.data.datapipes.datapipe:IterDataPipe; standard-library helpers such as collections.abc:Sized, enum:IntEnum, typing:NoReturn. The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch.utils.data.datapipes._decorator:functional_datapipe, torch.utils.data.datapipes.datapipe:IterDataPipe；标准库辅助模块，如 collections.abc:Sized, enum:IntEnum, typing:NoReturn。 `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 16-29 / 第 16-29 行
```python
class SHARDING_PRIORITIES(IntEnum):
    DEFAULT = 1
    DISTRIBUTED = 2
    MULTIPROCESSING = 3


class _ShardingIterDataPipe(IterDataPipe):
    def apply_sharding(
        self,
        num_of_instances: int,
        instance_id: int,
        sharding_group: SHARDING_PRIORITIES,
    ) -> NoReturn:
        raise NotImplementedError
```
- **EN**: It introduces or extends class-level abstractions such as `SHARDING_PRIORITIES`, `_ShardingIterDataPipe`, which organize state and behavior for this subsystem. Named constants such as `DEFAULT`, `DISTRIBUTED`, `MULTIPROCESSING` centralize shared configuration or sentinel values. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `SHARDING_PRIORITIES`, `_ShardingIterDataPipe` 等类级抽象，用于组织该子系统的状态与行为。 `DEFAULT, DISTRIBUTED, MULTIPROCESSING` 等具名常量把共享配置或哨兵值集中定义在一起。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 32-42 / 第 32-42 行
```python
@functional_datapipe("sharding_filter")
class ShardingFilterIterDataPipe(_ShardingIterDataPipe):
    r"""
    Wrapper that allows DataPipe to be sharded (functional name: ``sharding_filter``).

    After ``apply_sharding`` is called, each instance of the DataPipe (on different workers) will have every `n`-th element of the
    original DataPipe, where `n` equals to the number of instances.

    Args:
        source_datapipe: Iterable DataPipe that will be sharded
    """
```
- **EN**: It introduces or extends class-level abstractions such as `ShardingFilterIterDataPipe`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `ShardingFilterIterDataPipe` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 44-61 / 第 44-61 行
```python
    def __init__(
        self, source_datapipe: IterDataPipe, sharding_group_filter=None
    ) -> None:
        super().__init__()
        self.source_datapipe = source_datapipe
        self.sharding_group_filter = sharding_group_filter
        self.groups: dict[int, tuple[int, int]] = {}
        self.num_of_instances = 1
        self.instance_id = 0
        self._update_num_of_instances()

    def apply_sharding(
        self, num_of_instances, instance_id, sharding_group=SHARDING_PRIORITIES.DEFAULT
    ):
        if instance_id >= num_of_instances:
            raise ValueError(
                f"instance_id({instance_id}) should be smaller than num_of_instances({num_of_instances})"
            )
```
- **EN**: It introduces or extends class-level abstractions such as `ShardingFilterIterDataPipe`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `ShardingFilterIterDataPipe` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 62-73 / 第 62-73 行
```python
        if sharding_group == SHARDING_PRIORITIES.DEFAULT:
            if len(self.groups) and SHARDING_PRIORITIES.DEFAULT not in self.groups:
                raise RuntimeError(
                    "ShardingFilter cannot mix DEFAULT and non DEFAULT groups"
                )
        else:
            if SHARDING_PRIORITIES.DEFAULT in self.groups:
                raise RuntimeError(
                    "ShardingFilter cannot mix DEFAULT and non DEFAULT groups"
                )
        self.groups[sharding_group] = (num_of_instances, instance_id)
        self._update_num_of_instances()
```
- **EN**: It introduces or extends class-level abstractions such as `ShardingFilterIterDataPipe`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `ShardingFilterIterDataPipe` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 75-89 / 第 75-89 行
```python
    def _update_num_of_instances(self) -> None:
        sorted_sharding_groups = [
            self.groups[key]
            for key in sorted(self.groups.keys())
            if self.sharding_group_filter is None or key == self.sharding_group_filter
        ]

        sorted_sharding_groups.reverse()

        self.num_of_instances = 1
        self.instance_id = 0

        for group_num_of_instances, group_instance_id in sorted_sharding_groups:
            self.instance_id += self.num_of_instances * group_instance_id
            self.num_of_instances *= group_num_of_instances
```
- **EN**: It introduces or extends class-level abstractions such as `ShardingFilterIterDataPipe`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `ShardingFilterIterDataPipe` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 91-106 / 第 91-106 行
```python
    def __iter__(self):
        for i, item in enumerate(self.source_datapipe):
            if i % self.num_of_instances == self.instance_id:
                yield item

    def __len__(self) -> int:
        # pyrefly: ignore [unsafe-overlap]
        if isinstance(self.source_datapipe, Sized):
            return len(self.source_datapipe) // self.num_of_instances + (
                1
                if (
                    self.instance_id < len(self.source_datapipe) % self.num_of_instances
                )
                else 0
            )
        raise TypeError(f"{type(self).__name__} instance doesn't have valid length")
```
- **EN**: It introduces or extends class-level abstractions such as `ShardingFilterIterDataPipe`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. Yield-based control flow indicates that the block participates in lazy iteration instead of eager materialization. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `ShardingFilterIterDataPipe` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 基于 `yield` 的控制流说明该代码参与的是惰性迭代，而不是急切地产生结果。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements data-loading infrastructure, datapipes, samplers, worker coordination, and dataset traversal utilities.
  - CN: 实现数据加载基础设施、datapipes、sampler、worker 协调以及数据集遍历工具。
- **Data loading**
  - EN: Coordinates dataset traversal, batching, multiprocessing workers, and sample collation.
  - CN: 协调数据集遍历、批处理、多进程 worker 以及样本拼装。
- **DataPipes**
  - EN: Builds composable input pipelines that lazily transform or route dataset elements.
  - CN: 构建可组合的输入流水线，以惰性方式变换或路由数据集元素。
- **SHARDING_PRIORITIES**
  - EN: `SHARDING_PRIORITIES` is one of the main classes that structures the file's behavior.
  - CN: `SHARDING_PRIORITIES` 是组织该文件行为的核心类之一。
- **_ShardingIterDataPipe**
  - EN: `_ShardingIterDataPipe` is one of the main classes that structures the file's behavior.
  - CN: `_ShardingIterDataPipe` 是组织该文件行为的核心类之一。
- **Public exports**
  - EN: The module explicitly curates part of its public API through `__all__`.
  - CN: 该模块通过 `__all__` 显式整理了部分公开 API。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch.utils.data.datapipes._decorator:functional_datapipe`, `torch.utils.data.datapipes.datapipe:IterDataPipe`
- **Python standard library / Python 标准库**: `collections.abc:Sized`, `enum:IntEnum`, `typing:NoReturn`
- **Explicit exports / 显式导出**: `SHARDING_PRIORITIES`, `ShardingFilterIterDataPipe`
- **Primary symbols / 核心符号**: `SHARDING_PRIORITIES`, `_ShardingIterDataPipe`, `ShardingFilterIterDataPipe`
