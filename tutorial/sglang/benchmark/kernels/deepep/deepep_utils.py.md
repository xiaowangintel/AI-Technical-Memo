# deepep_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/deepep/deepep_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels deepep. It primarily prepares tensors and invokes GPU kernels, processes tokenized prompts or decoded outputs, and computes evaluation scores and aggregate statistics. / 该 Python 模块聚焦于 kernels deepep 相关流程。它主要用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。

## Line-by-Line Analysis / 逐行分析
### Lines 3-9: Imports and setup / 导入与初始化
```python
import os
import sys
from typing import Optional

import numpy as np
import torch
import torch.distributed as dist
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it prepares tensors and invokes GPU kernels.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会准备张量并调用 GPU 内核。

### Lines 12-33: Function `init_dist` / 函数 `init_dist`
```python
def init_dist(local_rank: int, num_local_ranks: int, args):
    ip = args.master_addr
    port = args.master_port
    num_nodes = args.nnodes
    node_rank = args.node_rank
    assert (num_local_ranks < 8 and num_nodes == 1) or num_local_ranks == 8

    dist.init_process_group(
        backend="nccl",
        init_method=f"tcp://{ip}:{port}",
        world_size=num_nodes * num_local_ranks,
        rank=node_rank * num_local_ranks + local_rank,
    )
    torch.set_default_dtype(torch.bfloat16)
    torch.set_default_device("cuda")
    torch.cuda.set_device(local_rank)

    return (
        dist.get_rank(),
        dist.get_world_size(),
        dist.new_group(list(range(num_local_ranks * num_nodes))),
    )
```
**EN:** `init_dist` is a function that prepares tensors and invokes GPU kernels. It returns `(dist.get_rank(), dist.get_world_size(), dist.new_group(list(range(num_local_ranks * num_nodes))))` to the caller. Notable calls include `dist.init_process_group`, `torch.set_default_dtype`, `torch.set_default_device`.
**CN:** `init_dist` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `(dist.get_rank(), dist.get_world_size(), dist.new_group(list(range(num_local_ranks * num_nodes))))`。其中较关键的调用包括 `dist.init_process_group`, `torch.set_default_dtype`, `torch.set_default_device`。

### Lines 36-40: Function `calc_diff` / 函数 `calc_diff`
```python
def calc_diff(x: torch.Tensor, y: torch.Tensor):
    x, y = x.double() + 1, y.double() + 1
    denominator = (x * x + y * y).sum()
    sim = 2 * (x * y).sum() / denominator
    return (1 - sim).item()
```
**EN:** `calc_diff` is a function that prepares tensors and invokes GPU kernels. It returns `(1 - sim).item()` to the caller. Notable calls include `(x * x + y * y).sum`, `(1 - sim).item`, `x.double`.
**CN:** `calc_diff` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `(1 - sim).item()`。其中较关键的调用包括 `(x * x + y * y).sum`, `(1 - sim).item`, `x.double`。

### Lines 43-50: Function `per_token_cast_to_fp8` / 函数 `per_token_cast_to_fp8`
```python
def per_token_cast_to_fp8(x: torch.Tensor):
    assert x.dim() == 2 and x.size(1) % 128 == 0
    m, n = x.shape
    x_view = x.view(m, -1, 128)
    x_amax = x_view.abs().float().amax(dim=2).view(m, -1).clamp(1e-4)
    return (x_view * (448.0 / x_amax.unsqueeze(2))).to(torch.float8_e4m3fn).view(
        m, n
    ), (x_amax / 448.0).view(m, -1)
```
**EN:** `per_token_cast_to_fp8` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. It returns `((x_view * (448.0 / x_amax.unsqueeze(2))).to(torch.float8_e4m3fn).view(m, n), (x_amax / 448.0).vi...` to the caller. Notable calls include `x.view`, `x_view.abs().float().amax(dim=2).view(m, -1).clamp`, `(x_view * (448.0 / x_amax.unsqueeze(2))).to(torch.float8_e4m3fn).view`.
**CN:** `per_token_cast_to_fp8` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `((x_view * (448.0 / x_amax.unsqueeze(2))).to(torch.float8_e4m3fn).view(m, n), (x_amax / 448.0).vi...`。其中较关键的调用包括 `x.view`, `x_view.abs().float().amax(dim=2).view(m, -1).clamp`, `(x_view * (448.0 / x_amax.unsqueeze(2))).to(torch.float8_e4m3fn).view`。

### Lines 53-56: Function `per_token_cast_back` / 函数 `per_token_cast_back`
```python
def per_token_cast_back(x_fp8: torch.Tensor, x_scales: torch.Tensor):
    x_fp32 = x_fp8.to(torch.float32).view(x_fp8.size(0), -1, 128)
    x_scales = x_scales.view(x_fp8.size(0), -1, 1)
    return (x_fp32 * x_scales).view(x_fp8.shape).to(torch.bfloat16)
```
**EN:** `per_token_cast_back` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. It returns `(x_fp32 * x_scales).view(x_fp8.shape).to(torch.bfloat16)` to the caller. Notable calls include `x_fp8.to(torch.float32).view`, `x_scales.view`, `(x_fp32 * x_scales).view(x_fp8.shape).to`.
**CN:** `per_token_cast_back` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `(x_fp32 * x_scales).view(x_fp8.shape).to(torch.bfloat16)`。其中较关键的调用包括 `x_fp8.to(torch.float32).view`, `x_scales.view`, `(x_fp32 * x_scales).view(x_fp8.shape).to`。

### Lines 59-71: Function `inplace_unique` / 函数 `inplace_unique`
```python
def inplace_unique(x: torch.Tensor, num_slots: int):
    assert x.dim() == 2
    mask = x < 0
    x_padded = x.masked_fill(mask, num_slots)
    bin_count = torch.zeros((x.size(0), num_slots + 1), dtype=x.dtype, device=x.device)
    bin_count.scatter_add_(1, x_padded, torch.ones_like(x_padded))
    bin_count = bin_count[:, :num_slots]
    sorted_bin_count, sorted_bin_idx = torch.sort(bin_count, dim=-1, descending=True)
    sorted_bin_idx.masked_fill_(sorted_bin_count == 0, -1)
    sorted_bin_idx = torch.sort(sorted_bin_idx, descending=True, dim=-1).values
    x[:, :].fill_(-1)
    valid_len = min(num_slots, x.size(1))
    x[:, :valid_len] = sorted_bin_idx[:, :valid_len]
```
**EN:** `inplace_unique` is a function that prepares tensors and invokes GPU kernels. Notable calls include `x.masked_fill`, `torch.zeros`, `bin_count.scatter_add_`.
**CN:** `inplace_unique` 是一个函数，用于准备张量并调用 GPU 内核。其中较关键的调用包括 `x.masked_fill`, `torch.zeros`, `bin_count.scatter_add_`。

### Lines 74-81: Function `create_grouped_scores` / 函数 `create_grouped_scores`
```python
def create_grouped_scores(
    scores: torch.Tensor, group_idx: torch.Tensor, num_groups: int
):
    num_tokens, num_experts = scores.shape
    scores = scores.view(num_tokens, num_groups, -1)
    mask = torch.zeros((num_tokens, num_groups), dtype=torch.bool, device=scores.device)
    mask = mask.scatter_(1, group_idx, True).unsqueeze(-1).expand_as(scores)
    return (scores * mask).view(num_tokens, num_experts)
```
**EN:** `create_grouped_scores` is a function that prepares tensors and invokes GPU kernels, processes tokenized prompts or decoded outputs, and computes evaluation scores and aggregate statistics. It returns `(scores * mask).view(num_tokens, num_experts)` to the caller. Notable calls include `scores.view`, `torch.zeros`, `mask.scatter_(1, group_idx, True).unsqueeze(-1).expand_as`.
**CN:** `create_grouped_scores` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。它会向调用方返回 `(scores * mask).view(num_tokens, num_experts)`。其中较关键的调用包括 `scores.view`, `torch.zeros`, `mask.scatter_(1, group_idx, True).unsqueeze(-1).expand_as`。

### Lines 84-111: Function `bench` / 函数 `bench`
```python
def bench(fn, num_warmups: int = 20, num_tests: int = 30, post_fn=None):
    # Flush L2 cache with 256 MB data
    torch.cuda.synchronize()
    cache = torch.empty(int(256e6 // 4), dtype=torch.int, device="cuda")

    # Warmup
    for _ in range(num_warmups):
        fn()

    # Flush L2
    cache.zero_()

    # Testing
    start_events = [torch.cuda.Event(enable_timing=True) for _ in range(num_tests)]
    end_events = [torch.cuda.Event(enable_timing=True) for _ in range(num_tests)]
    for i in range(num_tests):
        # Record
        start_events[i].record()
        fn()
        end_events[i].record()
        if post_fn is not None:
            post_fn()
    torch.cuda.synchronize()

    times = np.array(
        [s.elapsed_time(e) / 1e3 for s, e in zip(start_events, end_events)]
    )[1:]
    return np.average(times), np.min(times), np.max(times)
```
**EN:** `bench` is a function that prepares tensors and invokes GPU kernels. It returns `(np.average(times), np.min(times), np.max(times))` to the caller. Notable calls include `torch.cuda.synchronize`, `torch.empty`, `range`.
**CN:** `bench` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `(np.average(times), np.min(times), np.max(times))`。其中较关键的调用包括 `torch.cuda.synchronize`, `torch.empty`, `range`。

### Lines 114-114: Class `empty_suppress` declaration / 类 `empty_suppress` 声明
```python
class empty_suppress:
```
**EN:** This block introduces class `empty_suppress`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `empty_suppress`，用于通过统一接口组织相关行为。

### Lines 115-116: Method `empty_suppress.__enter__` / 方法 `empty_suppress.__enter__`
```python
    def __enter__(self):
        return self
```
**EN:** `empty_suppress.__enter__` is a method that implements the core logic for this scope. It returns `self` to the caller.
**CN:** `empty_suppress.__enter__` 是一个方法，用于实现当前作用域的核心逻辑。它会向调用方返回 `self`。

### Lines 118-119: Method `empty_suppress.__exit__` / 方法 `empty_suppress.__exit__`
```python
    def __exit__(self, *_):
        pass
```
**EN:** `empty_suppress.__exit__` is a method that implements the core logic for this scope.
**CN:** `empty_suppress.__exit__` 是一个方法，用于实现当前作用域的核心逻辑。

### Lines 122-122: Class `suppress_stdout_stderr` declaration / 类 `suppress_stdout_stderr` 声明
```python
class suppress_stdout_stderr:
```
**EN:** This block introduces class `suppress_stdout_stderr`, which groups related behavior behind a shared interface.
**CN:** 该代码块引入类 `suppress_stdout_stderr`，用于通过统一接口组织相关行为。

### Lines 123-141: Method `suppress_stdout_stderr.__enter__` / 方法 `suppress_stdout_stderr.__enter__`
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
**EN:** `suppress_stdout_stderr.__enter__` is a method that reads or writes local files and intermediate artifacts. It returns `self` to the caller. Notable calls include `open`, `sys.stdout.fileno`, `sys.stderr.fileno`.
**CN:** `suppress_stdout_stderr.__enter__` 是一个方法，用于读写本地文件及中间产物。它会向调用方返回 `self`。其中较关键的调用包括 `open`, `sys.stdout.fileno`, `sys.stderr.fileno`。

### Lines 143-154: Method `suppress_stdout_stderr.__exit__` / 方法 `suppress_stdout_stderr.__exit__`
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
**EN:** `suppress_stdout_stderr.__exit__` is a method that implements the core logic for this scope. Notable calls include `os.dup2`, `os.close`, `self.outnull_file.close`.
**CN:** `suppress_stdout_stderr.__exit__` 是一个方法，用于实现当前作用域的核心逻辑。其中较关键的调用包括 `os.dup2`, `os.close`, `self.outnull_file.close`。

### Lines 157-214: Function `bench_kineto` / 函数 `bench_kineto`
```python
def bench_kineto(
    fn,
    kernel_names,
    num_tests: int = 30,
    suppress_kineto_output: bool = False,
    trace_path: Optional[str] = None,
    barrier_comm_profiling: bool = False,
):
    # Profile
    suppress = suppress_stdout_stderr if suppress_kineto_output else empty_suppress
    with suppress():
        schedule = torch.profiler.schedule(wait=0, warmup=1, active=1, repeat=1)
        with torch.profiler.profile(
            activities=[torch.profiler.ProfilerActivity.CUDA], schedule=schedule
        ) as prof:
            for i in range(2):
                # NOTES: use a large kernel and a barrier to eliminate the unbalanced CPU launch overhead
                if barrier_comm_profiling:
                    lhs = torch.randn((8192, 8192), dtype=torch.float, device="cuda")
                    rhs = torch.randn((8192, 8192), dtype=torch.float, device="cuda")
                    lhs @ rhs
                    dist.all_reduce(torch.ones(1, dtype=torch.float, device="cuda"))
                for _ in range(num_tests):
                    fn()
                prof.step()

    # Parse the profiling table
    assert isinstance(kernel_names, str) or isinstance(kernel_names, tuple)
    is_tupled = isinstance(kernel_names, tuple)
    prof_lines = (
        prof.key_averages()
        .table(sort_by="cuda_time_total", max_name_column_width=100)
        .split("\n")
    )
    kernel_names = (kernel_names,) if isinstance(kernel_names, str) else kernel_names
    assert all([isinstance(name, str) for name in kernel_names])
    for name in kernel_names:
        assert (
            sum([name in line for line in prof_lines]) == 1
        ), f"Errors of the kernel {name} in the profiling table"

    # Save chrome traces
    if trace_path is not None:
        prof.export_chrome_trace(trace_path)

    # Return average kernel times
    units = {"ms": 1e3, "us": 1e6}
    kernel_times = []
    for name in kernel_names:
        for line in prof_lines:
            if name in line:
                time_str = line.split()[-2]
                for unit, scale in units.items():
                    if unit in time_str:
                        kernel_times.append(float(time_str.replace(unit, "")) / scale)
                        break
                break
    return tuple(kernel_times) if is_tupled else kernel_times[0]
```
**EN:** `bench_kineto` is a function that prepares tensors and invokes GPU kernels. It returns `tuple(kernel_times) if is_tupled else kernel_times[0]` to the caller. Notable calls include `isinstance`, `prof.key_averages().table(sort_by='cuda_time_total', max_name_column_width=100).split`, `all`.
**CN:** `bench_kineto` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `tuple(kernel_times) if is_tupled else kernel_times[0]`。其中较关键的调用包括 `isinstance`, `prof.key_averages().table(sort_by='cuda_time_total', max_name_column_width=100).split`, `all`。

### Lines 217-218: Function `hash_tensor` / 函数 `hash_tensor`
```python
def hash_tensor(t: torch.Tensor):
    return t.view(torch.int64).sum().item()
```
**EN:** `hash_tensor` is a function that prepares tensors and invokes GPU kernels. It returns `t.view(torch.int64).sum().item()` to the caller. Notable calls include `t.view(torch.int64).sum().item`, `t.view(torch.int64).sum`, `t.view`.
**CN:** `hash_tensor` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `t.view(torch.int64).sum().item()`。其中较关键的调用包括 `t.view(torch.int64).sum().item`, `t.view(torch.int64).sum`, `t.view`。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `sys`, `typing`
- **Third-party / 第三方依赖**: `numpy`, `torch`, `torch.distributed`
