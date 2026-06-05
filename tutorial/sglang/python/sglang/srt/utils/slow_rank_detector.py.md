# slow_rank_detector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/slow_rank_detector.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable runtime utilities centered on `slow_rank_detector` and the surrounding SGLang serving stack. / 提供围绕 `slow_rank_detector` 及其所处 SGLang 服务栈的可复用运行时工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Module setup and shared state / 模块设置与共享状态
```python
import logging
from typing import Any, Dict, List

import torch
import torch.distributed as dist
import triton

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `logging`, `typing`, `torch`, `torch.distributed`, `triton`. It also defines symbols such as `logger` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `logging`, `typing`, `torch`, `torch.distributed`, `triton`。 同时定义了 `logger` 等符号，供后续逻辑使用。

### Lines 11-23: Function `execute` / 函数 `execute`
```python
def execute():
    if dist.get_rank() == 0:
        logger.info(f"[slow_rank_detector] Start benchmarking...")

    local_metrics = {
        bench_name: _compute_local_metric(bench_name) for bench_name in _BENCH_NAMES
    }

    all_metrics = [None for _ in range(dist.get_world_size())]
    dist.gather_object(local_metrics, all_metrics if dist.get_rank() == 0 else None)

    if dist.get_rank() == 0:
        _analyze_metrics(all_metrics)
```
**EN:** This function implements `execute`. It primarily calls `dist.gather_object`, `dist.get_rank`, `logger.info`, `_compute_local_metric`, `_analyze_metrics`, `range` to complete its work. State updates are written into `local_metrics`, `all_metrics`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `execute`。 它主要通过调用 `dist.gather_object`, `dist.get_rank`, `logger.info`, `_compute_local_metric`, `_analyze_metrics`, `range` 来完成任务。 状态更新主要写入 `local_metrics`, `all_metrics`。 实现中使用了条件分支。

### Lines 26-26: Class `_GemmExecutor` declaration / 类 `_GemmExecutor` 声明
```python
class _GemmExecutor:
```
**EN:** This class establishes `_GemmExecutor` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `__call__`.
**CN:** 该类将 `_GemmExecutor` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `__call__` 等方法。

### Lines 27-29: Method `_GemmExecutor.__init__` / 方法 `_GemmExecutor.__init__`
```python
    def __init__(self):
        self.lhs = torch.randn((8192, 8192), dtype=torch.bfloat16, device="cuda")
        self.rhs = torch.randn((8192, 8192), dtype=torch.bfloat16, device="cuda")
```
**EN:** This method implements `__init__` on `_GemmExecutor`. It primarily calls `torch.randn` to complete its work. State updates are written into `self.lhs`, `self.rhs`.
**CN:** 该方法（属于 `_GemmExecutor`）实现了 `__init__`。 它主要通过调用 `torch.randn` 来完成任务。 状态更新主要写入 `self.lhs`, `self.rhs`。

### Lines 31-32: Method `_GemmExecutor.__call__` / 方法 `_GemmExecutor.__call__`
```python
    def __call__(self):
        self.lhs @ self.rhs
```
**EN:** This method implements `__call__` on `_GemmExecutor`.
**CN:** 该方法（属于 `_GemmExecutor`）实现了 `__call__`。

### Lines 35-35: Class `_ElementwiseExecutor` declaration / 类 `_ElementwiseExecutor` 声明
```python
class _ElementwiseExecutor:
```
**EN:** This class establishes `_ElementwiseExecutor` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `__call__`.
**CN:** 该类将 `_ElementwiseExecutor` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `__call__` 等方法。

### Lines 36-39: Method `_ElementwiseExecutor.__init__` / 方法 `_ElementwiseExecutor.__init__`
```python
    def __init__(self):
        self.value = torch.randint(
            0, 10000, (128 * 1024**2,), dtype=torch.int32, device="cuda"
        )
```
**EN:** This method implements `__init__` on `_ElementwiseExecutor`. It primarily calls `torch.randint` to complete its work. State updates are written into `self.value`.
**CN:** 该方法（属于 `_ElementwiseExecutor`）实现了 `__init__`。 它主要通过调用 `torch.randint` 来完成任务。 状态更新主要写入 `self.value`。

### Lines 41-42: Method `_ElementwiseExecutor.__call__` / 方法 `_ElementwiseExecutor.__call__`
```python
    def __call__(self):
        self.value += 1
```
**EN:** This method implements `__call__` on `_ElementwiseExecutor`. State updates are written into `self.value`.
**CN:** 该方法（属于 `_ElementwiseExecutor`）实现了 `__call__`。 状态更新主要写入 `self.value`。

### Lines 45-50: Constants and shared state / 常量与共享状态
```python
_EXECUTOR_CLS_OF_BENCH = {
    "gemm": _GemmExecutor,
    "elementwise": _ElementwiseExecutor,
}

_BENCH_NAMES = list(_EXECUTOR_CLS_OF_BENCH.keys())
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `_EXECUTOR_CLS_OF_BENCH`, `_BENCH_NAMES`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `_EXECUTOR_CLS_OF_BENCH`, `_BENCH_NAMES`。

### Lines 53-56: Function `_compute_local_metric` / 函数 `_compute_local_metric`
```python
def _compute_local_metric(bench_name):
    executor = _EXECUTOR_CLS_OF_BENCH[bench_name]()
    ms = triton.testing.do_bench_cudagraph(executor, return_mode="mean", rep=20)
    return ms
```
**EN:** This function implements `_compute_local_metric`. It primarily calls `_EXECUTOR_CLS_OF_BENCH`, `triton.testing.do_bench_cudagraph` to complete its work. State updates are written into `executor`, `ms`.
**CN:** 该函数实现了 `_compute_local_metric`。 它主要通过调用 `_EXECUTOR_CLS_OF_BENCH`, `triton.testing.do_bench_cudagraph` 来完成任务。 状态更新主要写入 `executor`, `ms`。

### Lines 59-71: Function `_analyze_metrics` / 函数 `_analyze_metrics`
```python
def _analyze_metrics(all_metrics: List[Dict[str, Any]]):
    for bench_name in _BENCH_NAMES:
        time_of_rank = torch.tensor([m[bench_name] for m in all_metrics])
        speed_of_rank = 1 / time_of_rank
        rel_speed_of_rank = speed_of_rank / speed_of_rank.max()
        slowest_rel_speed = rel_speed_of_rank.min().item()
        logger.info(
            f"[slow_rank_detector] {bench_name=} {slowest_rel_speed=} {rel_speed_of_rank=} {time_of_rank=}"
        )
        if slowest_rel_speed < 0.9:
            logger.warning(
                "[slow_rank_detector] Some ranks are too slow compared with others"
            )
```
**EN:** This function implements `_analyze_metrics`. It primarily calls `torch.tensor`, `rel_speed_of_rank.min.item`, `logger.info`, `speed_of_rank.max`, `logger.warning`, `rel_speed_of_rank.min` to complete its work. State updates are written into `time_of_rank`, `speed_of_rank`, `rel_speed_of_rank`, `slowest_rel_speed`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `_analyze_metrics`。 它主要通过调用 `torch.tensor`, `rel_speed_of_rank.min.item`, `logger.info`, `speed_of_rank.max`, `logger.warning`, `rel_speed_of_rank.min` 来完成任务。 状态更新主要写入 `time_of_rank`, `speed_of_rank`, `rel_speed_of_rank`, `slowest_rel_speed`。 实现中使用了条件分支、迭代逻辑。

## Key Concepts / 关键概念
- **Classes / 类**: `_GemmExecutor`, `_ElementwiseExecutor`
- **Functions / 函数**: `execute`, `_compute_local_metric`, `_analyze_metrics`, `__init__`, `__call__`, `__init__`, `__call__`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: None / 无
- **External / 外部依赖**: `torch`, `torch.distributed`, `triton`
- **Standard library / 标准库**: `logging`, `typing`
