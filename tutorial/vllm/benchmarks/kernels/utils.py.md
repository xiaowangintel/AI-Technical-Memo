# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements shared helper utilities, benchmark orchestration, centered around `CudaGraphBenchParams`, `ArgPool`, `Bench`. / 实现与共享辅助工具、基准测试编排相关的逻辑，核心符号包括 `CudaGraphBenchParams`, `ArgPool`, `Bench`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-10)
```python
import dataclasses
from collections.abc import Callable, Iterable
from typing import Any

import torch
import torch.utils.benchmark as TBenchmark
from torch.utils.benchmark import Measurement as TMeasurement
```
**EN:** This block gathers standard-library helpers such as `dataclasses`, `collections.abc`, `typing`; third-party packages such as `torch`, `torch.utils.benchmark`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `dataclasses`, `collections.abc`, `typing`；第三方依赖，如 `torch`, `torch.utils.benchmark`。这些依赖构成了后续基准测试逻辑的基础。

### Class `CudaGraphBenchParams` (lines 14-15)
```python
class CudaGraphBenchParams:
    num_ops_in_cuda_graph: int
```
**EN:** Class `CudaGraphBenchParams` packages shared state and related operations for this benchmark module. It extends `object` and exposes methods such as its methods.
**CN:** 类 `CudaGraphBenchParams` 用于封装该基准模块的共享状态与相关操作。它继承自 `object`，并提供 its methods 等方法。

### Class `ArgPool` (lines 19-31)
```python
class ArgPool:
    """
    When some argument of the benchmarking function is annotated with this type,
    the benchmarking class (BenchMM) will collapse the argument to a pick a
    single value from the given list of values, during function invocation.
    For every invocation during a benchmarking run, it will choose a
    different value from the list.
    """

    values: Iterable[Any]

    def __getitem__(self, index):
        return self.values[index]
```
**EN:** Class `ArgPool` is the main object-oriented wrapper for this module. Its docstring says: When some argument of the benchmarking function is annotated with this type, the benchmarking class (BenchMM) will collapse the argument to a pick a single value from the given list of values, during function invocation. It extends `object` and organizes behavior through `__getitem__`.
**CN:** 类 `ArgPool` 是该模块中的主要面向对象封装。文档字符串指出：When some argument of the benchmarking function is annotated with this type, the benchmarking class (BenchMM) will collapse the argument to a pick a single value from the given list of values, during function invocation。它继承自 `object`，并通过 `__getitem__` 组织行为。

### Class `Bench` (lines 34-214)
```python
class Bench:
    class ArgsIterator:
        def __init__(self, args_list, kwargs_list):
            assert len(args_list) == len(kwargs_list)
            self.args_list = args_list
            self.kwargs_list = kwargs_list
            self.n = len(self.args_list)
            self.idx = 0

        def __next__(self):
            while True:
                yield (self.args_list[self.idx], self.kwargs_list[self.idx])
                self.idx += 1
                self.idx = self.idx % self.n

        def reset(self):
            self.idx = 0

        @property
        def n_args(self):
            return self.n

    def __init__(
        self,
        cuda_graph_params: CudaGraphBenchParams | None,
        label: str,
        sub_label: str,
        description: str,
        fn: Callable,
        *args,
        **kwargs,
    ):
        self.cuda_graph_params = cuda_graph_params
        self.use_cuda_graph = self.cuda_graph_params is not None
    # ... omitted for brevity ...
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        if exc_type:
            print(f"exc type {exc_type}")
            print(f"exc value {exc_value}")
            print(f"exc traceback {traceback}")
```
**EN:** Class `Bench` packages shared state and related operations for this benchmark module. It extends `object` and exposes methods such as `__init__`, `collapse_argpool`, `get_cuda_graph_runner`, `run_cudagrah`, `run_eager`, `run`.
**CN:** 类 `Bench` 用于封装该基准模块的共享状态与相关操作。它继承自 `object`，并提供 `__init__`, `collapse_argpool`, `get_cuda_graph_runner`, `run_cudagrah`, `run_eager`, `run` 等方法。

### Method `Bench.__init__` (lines 56-85)
```python
    def __init__(
        self,
        cuda_graph_params: CudaGraphBenchParams | None,
        label: str,
        sub_label: str,
        description: str,
        fn: Callable,
        *args,
        **kwargs,
    ):
        self.cuda_graph_params = cuda_graph_params
        self.use_cuda_graph = self.cuda_graph_params is not None
        self.label = label
        self.sub_label = sub_label
        self.description = description
        self.fn = fn

        # Process args
        self._args = args
        self._kwargs = kwargs
        self.args_list, self.kwargs_list = self.collapse_argpool(*args, **kwargs)
        self.args_iterator = self.ArgsIterator(self.args_list, self.kwargs_list)

        # Cudagraph runner
        self.g = None
        if self.use_cuda_graph:
            self.g = self.get_cuda_graph_runner()

        # benchmark run params
        self.min_run_time = 1
```
**EN:** `__init__` implements a helper used by `utils.py`. It mainly works with `cuda_graph_params`, `label`, `sub_label`, `description`, `fn`, ... and relies on `self.collapse_argpool`, `self.ArgsIterator`, `self.get_cuda_graph_runner` plus branching to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `utils.py` 使用的辅助逻辑。 它主要处理 `cuda_graph_params`, `label`, `sub_label`, `description`, `fn`, ...，并结合 `self.collapse_argpool`, `self.ArgsIterator`, `self.get_cuda_graph_runner` 以及 条件分支 来完成这一段基准测试流程。

### Method `Bench.collapse_argpool` (lines 87-119)
```python
    def collapse_argpool(self, *args, **kwargs):
        argpool_args = [arg for arg in args if isinstance(arg, ArgPool)] + [
            arg for arg in kwargs.values() if isinstance(arg, ArgPool)
        ]
        if len(argpool_args) == 0:
            return [args], [kwargs]

        # Make sure all argpools are of the same size
        argpool_size = len(argpool_args[0].values)
        assert all([argpool_size == len(arg.values) for arg in argpool_args])

        # create copies of the args
        args_list = []
        kwargs_list = []
        for _ in range(argpool_size):
            args_list.append(args)
            kwargs_list.append(kwargs.copy())

        for i in range(argpool_size):
            # collapse args; Just pick the ith value
            args_list[i] = tuple(
                [arg[i] if isinstance(arg, ArgPool) else arg for arg in args_list[i]]
            )

            # collapse kwargs
            kwargs_i = kwargs_list[i]
            arg_pool_keys = [k for k, v in kwargs_i.items() if isinstance(v, ArgPool)]
            for k in arg_pool_keys:
                # again just pick the ith value
                kwargs_i[k] = kwargs_i[k][i]
            kwargs_list[i] = kwargs_i

        return args_list, kwargs_list
```
**EN:** `collapse_argpool` implements a helper used by `utils.py`. It mainly works with `*args`, `**kwargs` and relies on `isinstance`, `kwargs.values`, `len`, `all`, `range`, `args_list.append` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `collapse_argpool` 负责实现 `utils.py` 使用的辅助逻辑。 它主要处理 `*args`, `**kwargs`，并结合 `isinstance`, `kwargs.values`, `len`, `all`, `range`, `args_list.append` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Method `Bench.get_cuda_graph_runner` (lines 121-142)
```python
    def get_cuda_graph_runner(self):
        assert self.use_cuda_graph
        assert self.args_iterator is not None

        num_graph_ops = self.cuda_graph_params.num_ops_in_cuda_graph

        # warmup
        args_it = self.args_iterator.__next__()
        for _ in range(2):
            args, kwargs = next(args_it)
            self.fn(*args, **kwargs)

        self.args_iterator.reset()
        args_it = self.args_iterator.__next__()
        stream = torch.cuda.Stream()
        with torch.cuda.stream(stream):
            g = torch.cuda.CUDAGraph()
            with torch.cuda.graph(g):
                for _ in range(num_graph_ops):
                    args, kwargs = next(args_it)
                    self.fn(*args, **kwargs)
        return g
```
**EN:** `get_cuda_graph_runner` implements a helper used by `utils.py`. It mainly works with no explicit parameters and relies on `self.args_iterator.__next__`, `range`, `next`, `self.fn`, `self.args_iterator.reset`, `torch.cuda.Stream` plus iteration and context management to move data through this part of the benchmark pipeline.
**CN:** `get_cuda_graph_runner` 负责实现 `utils.py` 使用的辅助逻辑。 它主要处理 无显式参数，并结合 `self.args_iterator.__next__`, `range`, `next`, `self.fn`, `self.args_iterator.reset`, `torch.cuda.Stream` 以及 循环迭代、上下文管理 来完成这一段基准测试流程。

### Method `Bench.run_cudagrah` (lines 144-157)
```python
    def run_cudagrah(self) -> TMeasurement:
        assert self.use_cuda_graph
        globals = {"g": self.g}

        return TBenchmark.Timer(
            stmt="g.replay()",
            globals=globals,
            label=(
                f"{self.label}"
                f" | cugraph {self.cuda_graph_params.num_ops_in_cuda_graph} ops"
            ),
            sub_label=self.sub_label,
            description=self.description,
        ).blocked_autorange(min_run_time=self.min_run_time)
```
**EN:** `run_cudagrah` implements a helper used by `utils.py`. It mainly works with no explicit parameters and relies on `TBenchmark.Timer.blocked_autorange`, `TBenchmark.Timer` plus value production to move data through this part of the benchmark pipeline.
**CN:** `run_cudagrah` 负责实现 `utils.py` 使用的辅助逻辑。 它主要处理 无显式参数，并结合 `TBenchmark.Timer.blocked_autorange`, `TBenchmark.Timer` 以及 结果返回 来完成这一段基准测试流程。

## Key Concepts / 关键概念
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。

## Dependencies / 依赖关系
- **EN:** Standard library: `dataclasses`, `collections.abc`, `typing`.
- **CN:** 标准库依赖：`dataclasses`, `collections.abc`, `typing`。
- **EN:** Third-party packages: `torch`, `torch.utils.benchmark`.
- **CN:** 第三方依赖：`torch`, `torch.utils.benchmark`。
- **EN:** Internal modules: none.
- **CN:** 内部模块：无。
