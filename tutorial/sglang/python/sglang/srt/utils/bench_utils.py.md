# bench_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/bench_utils.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable runtime utilities centered on `bench_utils` and the surrounding SGLang serving stack. / 提供围绕 `bench_utils` 及其所处 SGLang 服务栈的可复用运行时工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Module setup and shared state / 模块设置与共享状态
```python
import os
import re
import sys
from contextlib import nullcontext

import torch
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `os`, `re`, `sys`, `contextlib`, `torch`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `os`, `re`, `sys`, `contextlib`, `torch`。

### Lines 10-10: Class `suppress_stdout_stderr` declaration / 类 `suppress_stdout_stderr` 声明
```python
class suppress_stdout_stderr:
```
**EN:** This class establishes `suppress_stdout_stderr` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__enter__`, `__exit__`.
**CN:** 该类将 `suppress_stdout_stderr` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__enter__`, `__exit__` 等方法。

### Lines 11-29: Method `suppress_stdout_stderr.__enter__` / 方法 `suppress_stdout_stderr.__enter__`
```python
    def __enter__(self):
        self.outnull_file = open(os.devnull, "w")
        self.errnull_file = open(os.devnull, "w")

        self.old_stdout_fileno_undup = sys.stdout.fileno()
        self.old_stderr_fileno_undup = sys.stderr.fileno()

        self.old_stdout_fileno = os.dup(sys.stdout.fileno())
        self.old_stderr_fileno = os.dup(sys.stderr.fileno())

        self.old_stdout = sys.stdout
        self.old_stderr = sys.stderr

        os.dup2(self.outnull_file.fileno(), self.old_stdout_fileno_undup)
        os.dup2(self.errnull_file.fileno(), self.old_stderr_fileno_undup)

        sys.stdout = self.outnull_file
        sys.stderr = self.errnull_file
        return self
```
**EN:** This method implements `__enter__` on `suppress_stdout_stderr`. It primarily calls `open`, `sys.stdout.fileno`, `sys.stderr.fileno`, `os.dup`, `os.dup2`, `self.outnull_file.fileno` to complete its work. State updates are written into `self.outnull_file`, `self.errnull_file`, `self.old_stdout_fileno_undup`, `self.old_stderr_fileno_undup`, `self.old_stdout_fileno`, `self.old_stderr_fileno`.
**CN:** 该方法（属于 `suppress_stdout_stderr`）实现了 `__enter__`。 它主要通过调用 `open`, `sys.stdout.fileno`, `sys.stderr.fileno`, `os.dup`, `os.dup2`, `self.outnull_file.fileno` 来完成任务。 状态更新主要写入 `self.outnull_file`, `self.errnull_file`, `self.old_stdout_fileno_undup`, `self.old_stderr_fileno_undup`, `self.old_stdout_fileno`, `self.old_stderr_fileno`。

### Lines 31-42: Method `suppress_stdout_stderr.__exit__` / 方法 `suppress_stdout_stderr.__exit__`
```python
    def __exit__(self, *_):
        sys.stdout = self.old_stdout
        sys.stderr = self.old_stderr

        os.dup2(self.old_stdout_fileno, self.old_stdout_fileno_undup)
        os.dup2(self.old_stderr_fileno, self.old_stderr_fileno_undup)

        os.close(self.old_stdout_fileno)
        os.close(self.old_stderr_fileno)

        self.outnull_file.close()
        self.errnull_file.close()
```
**EN:** This method implements `__exit__` on `suppress_stdout_stderr`. It primarily calls `os.dup2`, `os.close`, `self.outnull_file.close`, `self.errnull_file.close` to complete its work. State updates are written into `sys.stdout`, `sys.stderr`.
**CN:** 该方法（属于 `suppress_stdout_stderr`）实现了 `__exit__`。 它主要通过调用 `os.dup2`, `os.close`, `self.outnull_file.close`, `self.errnull_file.close` 来完成任务。 状态更新主要写入 `sys.stdout`, `sys.stderr`。

### Lines 46-157: Function `bench_kineto` / 函数 `bench_kineto`
```python
def bench_kineto(
    fn,
    kernel_names,
    num_tests: int = 30,
    suppress_kineto_output: bool = False,
    trace_path: str = None,
    flush_l2: bool = True,
    with_multiple_kernels: bool = False,
):
    # Conflict with Nsight Systems
    using_nsys = int(os.environ.get("SGLANG_NSYS_PROFILING", 0))

    # By default, flush L2 with an excessive 8GB memset to give the GPU some (literal) chill time without full idle
    flush_l2_size = int(8e9 // 4)

    # For some auto-tuning kernels with prints
    fn()

    # Profile
    suppress = (
        suppress_stdout_stderr
        if suppress_kineto_output and not using_nsys
        else nullcontext
    )
    with suppress():
        schedule = (
            torch.profiler.schedule(wait=0, warmup=1, active=1, repeat=1)
            if not using_nsys
# ... omitted for brevity ...
                        )
                        total_num += int(num_str)
                        break
        kernel_times.append(total_time / total_num)

    return tuple(kernel_times) if is_tuple else kernel_times[0]
```
**EN:** This function implements `bench_kineto`. It primarily calls `int`, `fn`, `isinstance`, `profiler.key_averages.table.split`, `all`, `os.environ.get` to complete its work. State updates are written into `using_nsys`, `flush_l2_size`, `suppress`, `is_tuple`, `prof_lines`, `kernel_names`. The implementation relies on conditional branches, iteration, context-managed resources.
**CN:** 该函数实现了 `bench_kineto`。 它主要通过调用 `int`, `fn`, `isinstance`, `profiler.key_averages.table.split`, `all`, `os.environ.get` 来完成任务。 状态更新主要写入 `using_nsys`, `flush_l2_size`, `suppress`, `is_tuple`, `prof_lines`, `kernel_names`。 实现中使用了条件分支、迭代逻辑、上下文管理资源。

## Key Concepts / 关键概念
- **Classes / 类**: `suppress_stdout_stderr`
- **Functions / 函数**: `bench_kineto`, `__enter__`, `__exit__`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: None / 无
- **External / 外部依赖**: `torch`
- **Standard library / 标准库**: `os`, `re`, `sys`, `contextlib`, `time`
