# sampler.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/utils/data/sampler.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements data-loading infrastructure, datapipes, samplers, worker coordination, and dataset traversal utilities. This specific file centers on `sampler.py`. Dataset iteration, batching, and worker orchestration are central concerns here. The implementation is performance-facing and focuses on measurement fidelity or result reporting. It bridges PyTorch values or graphs into visualization-friendly event formats.
- **Purpose (CN) / 用途（中文）**: 实现数据加载基础设施、datapipes、sampler、worker 协调以及数据集遍历工具。 该文件具体围绕 `sampler.py` 展开。 数据集迭代、批处理以及 worker 协调是这里的核心关注点。 该实现面向性能分析，重点在于测量可信度或结果报告。 它把 PyTorch 数值或计算图桥接到适合可视化的事件格式。
## Line-by-Line Analysis / 逐行分析

### Lines 1-22 / 第 1-22 行
```python
# mypy: allow-untyped-defs
import itertools
from collections.abc import Iterable, Iterator, Sequence, Sized
from typing import Generic, TypeVar

import torch


# Note: For benchmarking changes to samplers, see:
# /benchmarks/data/samplers_bench.py
# This benchmark compares the performance of different sampler implementations
# and can be used to evaluate the impact of optimizations.


__all__ = [
    "BatchSampler",
    "RandomSampler",
    "Sampler",
    "SequentialSampler",
    "SubsetRandomSampler",
    "WeightedRandomSampler",
]
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch; standard-library helpers such as itertools, collections.abc:Iterable, collections.abc:Iterator, collections.abc:Sequence. The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. The code is measurement-oriented, so reproducibility and result interpretation matter as much as raw execution.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch；标准库辅助模块，如 itertools, collections.abc:Iterable, collections.abc:Iterator, collections.abc:Sequence。 `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 该代码面向性能测量，因此可复现性和结果解释与原始执行同样重要。

### Lines 25-48 / 第 25-48 行
```python
_T_co = TypeVar("_T_co", covariant=True)


class Sampler(Generic[_T_co]):
    r"""Base class for all Samplers.

    Every Sampler subclass has to provide an :meth:`__iter__` method, providing a
    way to iterate over indices or lists of indices (batches) of dataset elements,
    and may provide a :meth:`__len__` method that returns the length of the returned iterators.

    Example:
        >>> # xdoctest: +SKIP
        >>> class AccedingSequenceLengthSampler(Sampler[int]):
        >>>     def __init__(self, data: List[str]) -> None:
        >>>         self.data = data
        >>>
        >>>     def __len__(self) -> int:
        >>>         return len(self.data)
        >>>
        >>>     def __iter__(self) -> Iterator[int]:
        >>>         sizes = torch.tensor([len(x) for x in self.data])
        >>>         yield from torch.argsort(sizes).tolist()
        >>>
        >>> class AccedingSequenceLengthBatchSampler(Sampler[List[int]]):
```
- **EN**: It introduces or extends class-level abstractions such as `Sampler`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Yield-based control flow indicates that the block participates in lazy iteration instead of eager materialization. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `Sampler` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 基于 `yield` 的控制流说明该代码参与的是惰性迭代，而不是急切地产生结果。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 49-67 / 第 49-67 行
```python
        >>>     def __init__(self, data: List[str], batch_size: int) -> None:
        >>>         self.data = data
        >>>         self.batch_size = batch_size
        >>>
        >>>     def __len__(self) -> int:
        >>>         return (len(self.data) + self.batch_size - 1) // self.batch_size
        >>>
        >>>     def __iter__(self) -> Iterator[List[int]]:
        >>>         sizes = torch.tensor([len(x) for x in self.data])
        >>>         for batch in torch.chunk(torch.argsort(sizes), len(self)):
        >>>             yield batch.tolist()

    .. note:: The :meth:`__len__` method isn't strictly required by
              :class:`~torch.utils.data.DataLoader`, but is expected in any
              calculation involving the length of a :class:`~torch.utils.data.DataLoader`.
    """

    def __iter__(self) -> Iterator[_T_co]:
        raise NotImplementedError
```
- **EN**: It introduces or extends class-level abstractions such as `Sampler`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. Yield-based control flow indicates that the block participates in lazy iteration instead of eager materialization. The tail of the block returns computed state or forwards the next value to the caller. This section supports data traversal, worker behavior, or sample batching in the input pipeline.
- **CN**: 它引入或扩展了 `Sampler` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 基于 `yield` 的控制流说明该代码参与的是惰性迭代，而不是急切地产生结果。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这一段支撑输入流水线中的数据遍历、worker 行为或样本批处理。

### Lines 69-92 / 第 69-92 行
```python
    # NOTE [ Lack of Default `__len__` in Python Abstract Base Classes ]
    #
    # Many times we have an abstract class representing a collection/iterable of
    # data, e.g., `torch.utils.data.Sampler`, with its subclasses optionally
    # implementing a `__len__` method. In such cases, we must make sure to not
    # provide a default implementation, because both straightforward default
    # implementations have their issues:
    #
    #   + `return NotImplemented`:
    #     Calling `len(subclass_instance)` raises:
    #       TypeError: 'NotImplementedType' object cannot be interpreted as an integer
    #
    #   + `raise NotImplementedError`:
    #     This prevents triggering some fallback behavior. E.g., the built-in
    #     `list(X)` tries to call `len(X)` first, and executes a different code
    #     path if the method is not found or `NotImplemented` is returned, while
    #     raising a `NotImplementedError` will propagate and make the call fail
    #     where it could have used `__iter__` to complete the call.
    #
    # Thus, the only two sensible things to do are
    #
    #   + **not** provide a default `__len__`.
    #
    #   + raise a `TypeError` instead, which is what Python uses when users call
```
- **EN**: This chunk continues the implementation of `Sampler`, filling in the details of its control flow or data handling. This range is dominated by comments or directives, so its main role is to document assumptions or tool behavior.
- **CN**: 这一段延续了 `Sampler` 的实现，继续补充其控制流或数据处理细节。 这一段主要由注释或指令构成，因此其主要作用是记录假设或工具行为。

### Lines 93-113 / 第 93-113 行
```python
    #     a method that is not defined on an object.
    #     (@ssnl verifies that this works on at least Python 3.7.)


class SequentialSampler(Sampler[int]):
    r"""Samples elements sequentially, always in the same order.

    Args:
        data_source (Sized): data source to sample from. Must implement __len__.
    """

    data_source: Sized

    def __init__(self, data_source: Sized) -> None:
        self.data_source = data_source

    def __iter__(self) -> Iterator[int]:
        return iter(range(len(self.data_source)))

    def __len__(self) -> int:
        return len(self.data_source)
```
- **EN**: It introduces or extends class-level abstractions such as `SequentialSampler`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `SequentialSampler` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 116-139 / 第 116-139 行
```python
class RandomSampler(Sampler[int]):
    r"""Samples elements randomly. If without replacement, then sample from a shuffled dataset.

    If with replacement, then user can specify :attr:`num_samples` to draw.

    Args:
        data_source (Sized): data source to sample from. Must implement __len__.
        replacement (bool): samples are drawn on-demand with replacement if ``True``, default=``False``
        num_samples (int): number of samples to draw, default=`len(dataset)`.
        generator (Generator): Generator used in sampling.
    """

    data_source: Sized
    replacement: bool

    def __init__(
        self,
        data_source: Sized,
        replacement: bool = False,
        num_samples: int | None = None,
        generator=None,
    ) -> None:
        self.data_source = data_source
        self.replacement = replacement
```
- **EN**: It introduces or extends class-level abstractions such as `RandomSampler`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms.
- **CN**: 它引入或扩展了 `RandomSampler` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。

### Lines 140-158 / 第 140-158 行
```python
        self._num_samples = num_samples
        self.generator = generator

        if not isinstance(self.replacement, bool):
            raise TypeError(
                f"replacement should be a boolean value, but got replacement={self.replacement}"
            )

        if not isinstance(self.num_samples, int) or self.num_samples <= 0:
            raise ValueError(
                f"num_samples should be a positive integer value, but got num_samples={self.num_samples}"
            )

    @property
    def num_samples(self) -> int:
        # dataset size might change at runtime
        if self._num_samples is None:
            return len(self.data_source)
        return self._num_samples
```
- **EN**: It introduces or extends class-level abstractions such as `RandomSampler`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `RandomSampler` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 160-183 / 第 160-183 行
```python
    def __iter__(self) -> Iterator[int]:
        n = len(self.data_source)
        if self.generator is None:
            seed = int(torch.empty((), dtype=torch.int64).random_().item())
            generator = torch.Generator()
            generator.manual_seed(seed)
        else:
            generator = self.generator

        if self.replacement:
            for _ in range(self.num_samples // 32):
                yield from torch.randint(
                    high=n, size=(32,), dtype=torch.int64, generator=generator
                ).tolist()
            yield from torch.randint(
                high=n,
                size=(self.num_samples % 32,),
                dtype=torch.int64,
                generator=generator,
            ).tolist()
        else:
            for _ in range(self.num_samples // n):
                yield from torch.randperm(n, generator=generator).tolist()
            yield from torch.randperm(n, generator=generator).tolist()[
```
- **EN**: It introduces or extends class-level abstractions such as `RandomSampler`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Yield-based control flow indicates that the block participates in lazy iteration instead of eager materialization.
- **CN**: 它引入或扩展了 `RandomSampler` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 基于 `yield` 的控制流说明该代码参与的是惰性迭代，而不是急切地产生结果。

### Lines 184-203 / 第 184-203 行
```python
                : self.num_samples % n
            ]

    def __len__(self) -> int:
        return self.num_samples


class SubsetRandomSampler(Sampler[int]):
    r"""Samples elements randomly from a given list of indices, without replacement.

    Args:
        indices (sequence): a sequence of indices
        generator (Generator): Generator used in sampling.
    """

    indices: Sequence[int]

    def __init__(self, indices: Sequence[int], generator=None) -> None:
        self.indices = indices
        self.generator = generator
```
- **EN**: It introduces or extends class-level abstractions such as `RandomSampler`, `SubsetRandomSampler`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `RandomSampler`, `SubsetRandomSampler` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 205-222 / 第 205-222 行
```python
    def __iter__(self) -> Iterator[int]:
        for i in torch.randperm(len(self.indices), generator=self.generator).tolist():
            yield self.indices[i]

    def __len__(self) -> int:
        return len(self.indices)


class WeightedRandomSampler(Sampler[int]):
    r"""Samples elements from ``[0,..,len(weights)-1]`` with given probabilities (weights).

    Args:
        weights (sequence)   : a sequence of weights, not necessary summing up to one
        num_samples (int): number of samples to draw
        replacement (bool): if ``True``, samples are drawn with replacement.
            If not, they are drawn without replacement, which means that when a
            sample index is drawn for a row, it cannot be drawn again for that row.
        generator (Generator): Generator used in sampling.
```
- **EN**: It introduces or extends class-level abstractions such as `SubsetRandomSampler`, `WeightedRandomSampler`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Yield-based control flow indicates that the block participates in lazy iteration instead of eager materialization. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `SubsetRandomSampler`, `WeightedRandomSampler` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 基于 `yield` 的控制流说明该代码参与的是惰性迭代，而不是急切地产生结果。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 224-242 / 第 224-242 行
```python
    Example:
        >>> # xdoctest: +IGNORE_WANT("non-deterministic")
        >>> list(
        ...     WeightedRandomSampler(
        ...         [0.1, 0.9, 0.4, 0.7, 3.0, 0.6], 5, replacement=True
        ...     )
        ... )
        [4, 4, 1, 4, 5]
        >>> list(
        ...     WeightedRandomSampler(
        ...         [0.9, 0.4, 0.05, 0.2, 0.3, 0.1], 5, replacement=False
        ...     )
        ... )
        [0, 1, 4, 3, 2]
    """

    weights: torch.Tensor
    num_samples: int
    replacement: bool
```
- **EN**: It introduces or extends class-level abstractions such as `WeightedRandomSampler`, which organize state and behavior for this subsystem.
- **CN**: 它引入或扩展了 `WeightedRandomSampler` 等类级抽象，用于组织该子系统的状态与行为。

### Lines 244-262 / 第 244-262 行
```python
    def __init__(
        self,
        weights: Sequence[float],
        num_samples: int,
        replacement: bool = True,
        generator=None,
    ) -> None:
        if (
            not isinstance(num_samples, int)
            or isinstance(num_samples, bool)
            or num_samples <= 0
        ):
            raise ValueError(
                f"num_samples should be a positive integer value, but got num_samples={num_samples}"
            )
        if not isinstance(replacement, bool):
            raise ValueError(
                f"replacement should be a boolean value, but got replacement={replacement}"
            )
```
- **EN**: It introduces or extends class-level abstractions such as `WeightedRandomSampler`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 它引入或扩展了 `WeightedRandomSampler` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 264-283 / 第 264-283 行
```python
        weights_tensor = torch.as_tensor(weights, dtype=torch.double)
        if len(weights_tensor.shape) != 1:
            raise ValueError(
                "weights should be a 1d sequence but given "
                f"weights have shape {tuple(weights_tensor.shape)}"
            )

        self.weights = weights_tensor
        self.num_samples = num_samples
        self.replacement = replacement
        self.generator = generator

    def __iter__(self) -> Iterator[int]:
        rand_tensor = torch.multinomial(
            self.weights, self.num_samples, self.replacement, generator=self.generator
        )
        yield from iter(rand_tensor.tolist())

    def __len__(self) -> int:
        return self.num_samples
```
- **EN**: It introduces or extends class-level abstractions such as `WeightedRandomSampler`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. Yield-based control flow indicates that the block participates in lazy iteration instead of eager materialization. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `WeightedRandomSampler` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 基于 `yield` 的控制流说明该代码参与的是惰性迭代，而不是急切地产生结果。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 286-306 / 第 286-306 行
```python
class BatchSampler(Sampler[list[int]]):
    r"""Wraps another sampler to yield a mini-batch of indices.

    Args:
        sampler (Sampler or Iterable): Base sampler. Can be any iterable object
        batch_size (int): Size of mini-batch.
        drop_last (bool): If ``True``, the sampler will drop the last batch if
            its size would be less than ``batch_size``

    Example:
        >>> list(
        ...     BatchSampler(
        ...         SequentialSampler(range(10)), batch_size=3, drop_last=False
        ...     )
        ... )
        [[0, 1, 2], [3, 4, 5], [6, 7, 8], [9]]
        >>> list(
        ...     BatchSampler(SequentialSampler(range(10)), batch_size=3, drop_last=True)
        ... )
        [[0, 1, 2], [3, 4, 5], [6, 7, 8]]
    """
```
- **EN**: It introduces or extends class-level abstractions such as `BatchSampler`, which organize state and behavior for this subsystem. Yield-based control flow indicates that the block participates in lazy iteration instead of eager materialization.
- **CN**: 它引入或扩展了 `BatchSampler` 等类级抽象，用于组织该子系统的状态与行为。 基于 `yield` 的控制流说明该代码参与的是惰性迭代，而不是急切地产生结果。

### Lines 308-331 / 第 308-331 行
```python
    def __init__(
        self,
        sampler: Sampler[int] | Iterable[int],
        batch_size: int,
        drop_last: bool,
    ) -> None:
        # Since collections.abc.Iterable does not check for `__getitem__`, which
        # is one way for an object to be an iterable, we don't do an `isinstance`
        # check here.
        if (
            not isinstance(batch_size, int)
            or isinstance(batch_size, bool)
            or batch_size <= 0
        ):
            raise ValueError(
                f"batch_size should be a positive integer value, but got batch_size={batch_size}"
            )
        if not isinstance(drop_last, bool):
            raise ValueError(
                f"drop_last should be a boolean value, but got drop_last={drop_last}"
            )
        self.sampler = sampler
        self.batch_size = batch_size
        self.drop_last = drop_last
```
- **EN**: It introduces or extends class-level abstractions such as `BatchSampler`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 它引入或扩展了 `BatchSampler` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 333-354 / 第 333-354 行
```python
    def __iter__(self) -> Iterator[list[int]]:
        sampler_iter = iter(self.sampler)
        if self.drop_last:
            # Create multiple references to the same iterator
            args = [sampler_iter] * self.batch_size
            for batch_droplast in zip(*args, strict=False):
                yield [*batch_droplast]
        else:
            batch = [*itertools.islice(sampler_iter, self.batch_size)]
            while batch:
                yield batch
                batch = [*itertools.islice(sampler_iter, self.batch_size)]

    def __len__(self) -> int:
        # Can only be called if self.sampler has __len__ implemented
        # We cannot enforce this condition, so we turn off typechecking for the
        # implementation below.
        # Somewhat related: see NOTE [ Lack of Default `__len__` in Python Abstract Base Classes ]
        if self.drop_last:
            return len(self.sampler) // self.batch_size  # type: ignore[arg-type]
        else:
            return (len(self.sampler) + self.batch_size - 1) // self.batch_size  # type: ignore[arg-type]
```
- **EN**: It introduces or extends class-level abstractions such as `BatchSampler`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Yield-based control flow indicates that the block participates in lazy iteration instead of eager materialization. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `BatchSampler` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 基于 `yield` 的控制流说明该代码参与的是惰性迭代，而不是急切地产生结果。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements data-loading infrastructure, datapipes, samplers, worker coordination, and dataset traversal utilities.
  - CN: 实现数据加载基础设施、datapipes、sampler、worker 协调以及数据集遍历工具。
- **Graph rewriting**
  - EN: Rewrites module or FX graph structure so later execution follows a transformed pipeline.
  - CN: 改写模块或 FX 图结构，使后续执行遵循转换后的流水线。
- **Numeric comparison**
  - EN: Helps compare reference and transformed executions to diagnose quantization effects.
  - CN: 帮助比较参考执行与转换后执行，从而诊断量化带来的影响。
- **Data loading**
  - EN: Coordinates dataset traversal, batching, multiprocessing workers, and sample collation.
  - CN: 协调数据集遍历、批处理、多进程 worker 以及样本拼装。
- **Benchmarking**
  - EN: Focuses on repeatable measurement, result comparison, and performance diagnostics.
  - CN: 聚焦可重复的性能测量、结果比较与性能诊断。
- **TensorBoard integration**
  - EN: Exports model, scalar, and graph information into TensorBoard-consumable formats.
  - CN: 把模型、标量与图信息导出为 TensorBoard 可消费的格式。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`
- **Python standard library / Python 标准库**: `itertools`, `collections.abc:Iterable`, `collections.abc:Iterator`, `collections.abc:Sequence`, `collections.abc:Sized`, `typing:Generic`, `typing:TypeVar`
- **Explicit exports / 显式导出**: `BatchSampler`, `RandomSampler`, `Sampler`, `SequentialSampler`, `SubsetRandomSampler`, `WeightedRandomSampler`
- **Primary symbols / 核心符号**: `Sampler`, `SequentialSampler`, `RandomSampler`, `SubsetRandomSampler`, `WeightedRandomSampler`, `BatchSampler`
