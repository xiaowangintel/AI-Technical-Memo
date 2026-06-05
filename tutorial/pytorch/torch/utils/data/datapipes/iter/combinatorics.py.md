# combinatorics.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/utils/data/datapipes/iter/combinatorics.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements data-loading infrastructure, datapipes, samplers, worker coordination, and dataset traversal utilities. This specific file centers on `combinatorics.py`. Dataset iteration, batching, and worker orchestration are central concerns here.
- **Purpose (CN) / 用途（中文）**: 实现数据加载基础设施、datapipes、sampler、worker 协调以及数据集遍历工具。 该文件具体围绕 `combinatorics.py` 展开。 数据集迭代、批处理以及 worker 协调是这里的核心关注点。
## Line-by-Line Analysis / 逐行分析

### Lines 1-15 / 第 1-15 行
```python
# mypy: allow-untyped-defs
import random
from collections.abc import Iterator, Sized
from typing import TypeVar

import torch
from torch.utils.data.datapipes._decorator import functional_datapipe
from torch.utils.data.datapipes.datapipe import IterDataPipe
from torch.utils.data.sampler import Sampler, SequentialSampler


__all__ = [
    "SamplerIterDataPipe",
    "ShufflerIterDataPipe",
]
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch, torch.utils.data.datapipes._decorator:functional_datapipe, torch.utils.data.datapipes.datapipe:IterDataPipe, torch.utils.data.sampler:Sampler; standard-library helpers such as random, collections.abc:Iterator, collections.abc:Sized, typing:TypeVar. The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch, torch.utils.data.datapipes._decorator:functional_datapipe, torch.utils.data.datapipes.datapipe:IterDataPipe, torch.utils.data.sampler:Sampler；标准库辅助模块，如 random, collections.abc:Iterator, collections.abc:Sized, typing:TypeVar。 `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 18-32 / 第 18-32 行
```python
_T_co = TypeVar("_T_co", covariant=True)


class SamplerIterDataPipe(IterDataPipe[_T_co]):
    r"""
    Generate sample elements using the provided ``Sampler`` (defaults to :class:`SequentialSampler`).

    Args:
        datapipe: IterDataPipe to sample from
        sampler: Sampler class to generate sample elements from input DataPipe.
            Default is :class:`SequentialSampler` for IterDataPipe
    """

    datapipe: IterDataPipe
    sampler: Sampler
```
- **EN**: It introduces or extends class-level abstractions such as `SamplerIterDataPipe`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `SamplerIterDataPipe` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 34-46 / 第 34-46 行
```python
    def __init__(
        self,
        datapipe: IterDataPipe,
        sampler: type[Sampler] = SequentialSampler,
        sampler_args: tuple | None = None,
        sampler_kwargs: dict | None = None,
    ) -> None:
        # pyrefly: ignore [unsafe-overlap]
        if not isinstance(datapipe, Sized):
            raise AssertionError(
                "Sampler class requires input datapipe implemented `__len__`"
            )
        super().__init__()
```
- **EN**: It introduces or extends class-level abstractions such as `SamplerIterDataPipe`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `SamplerIterDataPipe` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 48-61 / 第 48-61 行
```python
        self.datapipe = datapipe
        self.sampler_args = () if sampler_args is None else sampler_args
        self.sampler_kwargs = {} if sampler_kwargs is None else sampler_kwargs
        self.sampler_kwargs["data_source"] = self.datapipe
        self.sampler = sampler(*self.sampler_args, **self.sampler_kwargs)

    def __iter__(self) -> Iterator[_T_co]:
        return iter(self.sampler)

    def __len__(self) -> int:
        # Dataset has been tested as `Sized`
        if isinstance(self.sampler, Sized):
            return len(self.sampler)
        raise TypeError(f"{type(self).__name__} instance doesn't have valid length")
```
- **EN**: It introduces or extends class-level abstractions such as `SamplerIterDataPipe`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `SamplerIterDataPipe` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 64-77 / 第 64-77 行
```python
@functional_datapipe("shuffle")
class ShufflerIterDataPipe(IterDataPipe[_T_co]):
    r"""
    Shuffle the input DataPipe with a buffer (functional name: ``shuffle``).

    The buffer with ``buffer_size`` is filled with elements from the datapipe first. Then,
    each item will be yielded from the buffer by reservoir sampling via iterator.

    ``buffer_size`` is required to be larger than ``0``. For ``buffer_size == 1``, the
    datapipe is not shuffled. In order to fully shuffle all elements from datapipe,
    ``buffer_size`` is required to be greater than or equal to the size of datapipe.

    When it is used with :class:`torch.utils.data.DataLoader`, the methods to
    set up random seed are different based on :attr:`num_workers`.
```
- **EN**: It introduces or extends class-level abstractions such as `ShufflerIterDataPipe`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Yield-based control flow indicates that the block participates in lazy iteration instead of eager materialization. This section supports data traversal, worker behavior, or sample batching in the input pipeline.
- **CN**: 它引入或扩展了 `ShufflerIterDataPipe` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 基于 `yield` 的控制流说明该代码参与的是惰性迭代，而不是急切地产生结果。 这一段支撑输入流水线中的数据遍历、worker 行为或样本批处理。

### Lines 79-96 / 第 79-96 行
```python
    For single-process mode (:attr:`num_workers == 0`), the random seed is set before
    the :class:`~torch.utils.data.DataLoader` in the main process. For multi-process
    mode (:attr:`num_worker > 0`), `worker_init_fn` is used to set up a random seed
    for each worker process.

    Args:
        datapipe: The IterDataPipe being shuffled
        buffer_size: The buffer size for shuffling (default to ``10000``)
        unbatch_level: Specifies if it is necessary to unbatch source data before
            applying the shuffle

    Example:
        >>> # xdoctest: +SKIP
        >>> from torchdata.datapipes.iter import IterableWrapper
        >>> dp = IterableWrapper(range(10))
        >>> shuffle_dp = dp.shuffle()
        >>> list(shuffle_dp)
        [0, 4, 1, 6, 3, 2, 9, 5, 7, 8]
```
- **EN**: It introduces or extends class-level abstractions such as `ShufflerIterDataPipe`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. This section supports data traversal, worker behavior, or sample batching in the input pipeline.
- **CN**: 它引入或扩展了 `ShufflerIterDataPipe` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 这一段支撑输入流水线中的数据遍历、worker 行为或样本批处理。

### Lines 97-114 / 第 97-114 行
```python
    """

    datapipe: IterDataPipe[_T_co]
    buffer_size: int
    _buffer: list[_T_co]
    _enabled: bool
    _seed: int | None
    _rng: random.Random

    def __init__(
        self,
        datapipe: IterDataPipe[_T_co],
        *,
        buffer_size: int = 10000,
        unbatch_level: int = 0,
    ) -> None:
        super().__init__()
        # TODO: Performance optimization
```
- **EN**: It introduces or extends class-level abstractions such as `ShufflerIterDataPipe`, which organize state and behavior for this subsystem. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `ShufflerIterDataPipe` 等类级抽象，用于组织该子系统的状态与行为。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 115-130 / 第 115-130 行
```python
        #       buffer can be a fixed size and remove expensive `append()` and `len()` operations
        self._buffer: list[_T_co] = []
        if buffer_size <= 0:
            raise AssertionError("buffer_size should be larger than 0")
        if unbatch_level == 0:
            self.datapipe = datapipe
        else:
            self.datapipe = datapipe.unbatch(unbatch_level=unbatch_level)
        self.buffer_size = buffer_size
        self._enabled = True
        self._seed = None
        self._rng = random.Random()

    def set_shuffle(self, shuffle=True):
        self._enabled = shuffle
        return self
```
- **EN**: It introduces or extends class-level abstractions such as `ShufflerIterDataPipe`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `ShufflerIterDataPipe` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 132-149 / 第 132-149 行
```python
    def set_seed(self, seed: int):
        self._seed = seed
        return self

    def __iter__(self) -> Iterator[_T_co]:
        if not self._enabled:
            yield from self.datapipe
        else:
            for x in self.datapipe:
                if len(self._buffer) == self.buffer_size:
                    idx = self._rng.randint(0, len(self._buffer) - 1)
                    val, self._buffer[idx] = self._buffer[idx], x
                    yield val
                else:
                    self._buffer.append(x)
            while self._buffer:
                idx = self._rng.randint(0, len(self._buffer) - 1)
                yield self._buffer.pop(idx)
```
- **EN**: It introduces or extends class-level abstractions such as `ShufflerIterDataPipe`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Yield-based control flow indicates that the block participates in lazy iteration instead of eager materialization. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `ShufflerIterDataPipe` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 基于 `yield` 的控制流说明该代码参与的是惰性迭代，而不是急切地产生结果。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 151-163 / 第 151-163 行
```python
    def __len__(self) -> int:
        # pyrefly: ignore [unsafe-overlap]
        if isinstance(self.datapipe, Sized):
            return len(self.datapipe)
        raise TypeError(f"{type(self).__name__} instance doesn't have valid length")

    def reset(self) -> None:
        self._buffer = []
        if self._enabled:
            if self._seed is None:
                self._seed = int(torch.empty((), dtype=torch.int64).random_().item())
            self._rng.seed(self._seed)
            self._seed = None
```
- **EN**: It introduces or extends class-level abstractions such as `ShufflerIterDataPipe`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `ShufflerIterDataPipe` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 165-178 / 第 165-178 行
```python
    def __getstate__(self):
        state = (
            self.datapipe,
            self.buffer_size,
            self._enabled,
            self._seed,
            self._buffer,
            self._rng.getstate(),
            self._valid_iterator_id,
            self._number_of_samples_yielded,
        )
        if IterDataPipe.getstate_hook is not None:
            return IterDataPipe.getstate_hook(state)
        return state
```
- **EN**: It introduces or extends class-level abstractions such as `ShufflerIterDataPipe`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Yield-based control flow indicates that the block participates in lazy iteration instead of eager materialization. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `ShufflerIterDataPipe` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 基于 `yield` 的控制流说明该代码参与的是惰性迭代，而不是急切地产生结果。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 180-195 / 第 180-195 行
```python
    def __setstate__(self, state):
        (
            self.datapipe,
            self.buffer_size,
            self._enabled,
            self._seed,
            self._buffer,
            rng_state,
            self._valid_iterator_id,
            self._number_of_samples_yielded,
        ) = state
        self._rng = random.Random()
        self._rng.setstate(rng_state)

    def __del__(self) -> None:
        self._buffer.clear()
```
- **EN**: It introduces or extends class-level abstractions such as `ShufflerIterDataPipe`, which organize state and behavior for this subsystem. Yield-based control flow indicates that the block participates in lazy iteration instead of eager materialization. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 它引入或扩展了 `ShufflerIterDataPipe` 等类级抽象，用于组织该子系统的状态与行为。 基于 `yield` 的控制流说明该代码参与的是惰性迭代，而不是急切地产生结果。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

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
- **Hooks and callbacks**
  - EN: Coordinates callback registration and ordered execution around framework events.
  - CN: 协调回调注册以及围绕框架事件的有序执行。
- **SamplerIterDataPipe**
  - EN: `SamplerIterDataPipe` is one of the main classes that structures the file's behavior.
  - CN: `SamplerIterDataPipe` 是组织该文件行为的核心类之一。
- **ShufflerIterDataPipe**
  - EN: `ShufflerIterDataPipe` is one of the main classes that structures the file's behavior.
  - CN: `ShufflerIterDataPipe` 是组织该文件行为的核心类之一。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.utils.data.datapipes._decorator:functional_datapipe`, `torch.utils.data.datapipes.datapipe:IterDataPipe`, `torch.utils.data.sampler:Sampler`, `torch.utils.data.sampler:SequentialSampler`
- **Python standard library / Python 标准库**: `random`, `collections.abc:Iterator`, `collections.abc:Sized`, `typing:TypeVar`
- **Explicit exports / 显式导出**: `SamplerIterDataPipe`, `ShufflerIterDataPipe`
- **Primary symbols / 核心符号**: `SamplerIterDataPipe`, `ShufflerIterDataPipe`
