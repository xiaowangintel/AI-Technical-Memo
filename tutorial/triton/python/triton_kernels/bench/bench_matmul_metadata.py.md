# bench_matmul_metadata.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/bench/bench_matmul_metadata.py`
- **Purpose / 用途:** Benchmark driver for bench matmul metadata; it measures performance, sweeps configurations, and reports results for Triton kernels. / 用于 bench matmul metadata 的基准测试驱动；它会测量性能、扫描配置，并报告 Triton 内核的结果。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-10 (module)
```python
 1| import argparse
 2| import dataclasses
 3| import math
 4| import time
 5| 
 6| import torch
 7| 
 8| from triton_kernels.matmul_details._common import _matmul_flops_and_bytes_from_slices
 9| 
10| 
```
**EN:** This block imports `argparse`, `dataclasses`, `math`, `time`, `torch`, `triton_kernels.matmul_details._common (_matmul_flops_and_bytes_from_slices)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `argparse`, `dataclasses`, `math`, `time`, `torch`, `triton_kernels.matmul_details._common (_matmul_flops_and_bytes_from_slices)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 11-12 (FakeTensor)
```python
11| @dataclasses.dataclass(frozen=True)
12| class FakeTensor:
```
**EN:** Defines class `FakeTensor` with decorators `dataclasses.dataclass(frozen=True)` to organize related behavior. Key fields include `shape`, `dtype`. Main methods are `element_size`, `numel`.

**CN:** 定义类 `FakeTensor`，带有装饰器 `dataclasses.dataclass(frozen=True)`，用于组织相关行为。关键字段包括 `shape`, `dtype`。主要方法有 `element_size`, `numel`.

### Block 3 — Lines 13-13 (FakeTensor)
```python
13|     shape: tuple[int, ...]
```
**EN:** Annotated assignment stores `shape` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `shape` and 声明该带注解的字段.

### Block 4 — Lines 14-15 (FakeTensor)
```python
14|     dtype: torch.dtype
15| 
```
**EN:** Annotated assignment stores `dtype` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `dtype` and 声明该带注解的字段.

### Block 5 — Lines 16-16 (element_size)
```python
16|     def element_size(self) -> int:
```
**EN:** Defines function `element_size(self)` for this module. The body mainly returns the computed result. It uses calls such as `torch.empty((), dtype=self.dtype).ele...`, `torch.empty` to implement its workflow.

**CN:** 定义函数 `element_size(self)`，供本模块使用. 主体主要返回计算结果. 其中会调用 `torch.empty((), dtype=self.dtype).ele...`, `torch.empty` 来实现其工作流程.

### Block 6 — Lines 17-18 (element_size)
```python
17|         return torch.empty((), dtype=self.dtype).element_size()
18| 
```
**EN:** Returns `torch.empty((), dtype=self.dtype).element_size()`.

**CN:** 返回 `torch.empty((), dtype=self.dtype).element_size()`.

### Block 7 — Lines 19-19 (numel)
```python
19|     def numel(self) -> int:
```
**EN:** Defines function `numel(self)` for this module. The body mainly returns the computed result. It uses calls such as `math.prod` to implement its workflow.

**CN:** 定义函数 `numel(self)`，供本模块使用. 主体主要返回计算结果. 其中会调用 `math.prod` 来实现其工作流程.

### Block 8 — Lines 20-22 (numel)
```python
20|         return math.prod(self.shape)
21| 
22| 
```
**EN:** Returns `math.prod(self.shape)`.

**CN:** 返回 `math.prod(self.shape)`.

### Block 9 — Lines 23-23 (_old_flops_and_bytes_from_slices)
```python
23| def _old_flops_and_bytes_from_slices(args, M, N, K, X, Y, W, slice_sizes, nbits, batch_size):
```
**EN:** Defines function `_old_flops_and_bytes_from_slices(args, M, N, K, X, Y, W, slice_sizes, nbits, batch_size)` for this module. The body mainly prepares intermediate values; prepares intermediate values; branches on runtime conditions. It uses calls such as `slice_sizes.sum`, `n_tokens.to`, `X.element_size`, `W.element_size`, `Y.element_size` to implement its workflow.

**CN:** 定义函数 `_old_flops_and_bytes_from_slices(args, M, N, K, X, Y, W, slice_sizes, nbits, batch_size)`，供本模块使用. 主体主要准备中间值; 准备中间值; 根据运行时条件分支. 其中会调用 `slice_sizes.sum`, `n_tokens.to`, `X.element_size`, `W.element_size`, `Y.element_size` 来实现其工作流程.

### Block 10 — Lines 24-24 (_old_flops_and_bytes_from_slices)
```python
24|     n_tokens = slice_sizes.sum()
```
**EN:** Assigns `n_tokens` and calls `slice_sizes.sum`.

**CN:** 将 `n_tokens`，并调用 `slice_sizes.sum`.

### Block 11 — Lines 25-26 (_old_flops_and_bytes_from_slices)
```python
25|     z = 1 if args["RAGGED_DIMENSION"] == "K" else batch_size
26| 
```
**EN:** Assigns `z` and uses conditional expression `1 if args['RAGGED_DIMENSION'] == 'K' else batch_size`.

**CN:** 将 `z` and 使用条件表达式 `1 if args['RAGGED_DIMENSION'] == 'K' else batch_size`.

### Block 12 — Lines 27-43 (_old_flops_and_bytes_from_slices)
```python
27|     if args["RAGGED_DIMENSION"] == "K":
28|         flops = n_tokens.to(torch.float64) * (2.0 * M * N * z)
29|         n_x_bytes = n_tokens * X.shape[-2] * X.element_size()
30|         n_y_bytes = Y.numel() * Y.element_size() * (2 if args["OutAcc"] is not None else 1)
31|         n_w_bytes = n_tokens * W.shape[-1] * W.element_size()
32|     else:
33|         if M is None:
34|             flops = n_tokens.to(torch.float64) * (2.0 * N * K * z)
35|         elif K is None:
36|             flops = n_tokens.to(torch.float64) * (2.0 * M * N * z)
37|         else:
38|             flops = torch.empty((), dtype=torch.float64, device=slice_sizes.device)
39|             flops.fill_(2.0 * M * N * K * z)
40|         n_x_bytes = n_tokens * X.shape[-1] * X.element_size()
41|         n_y_bytes = n_tokens * Y.shape[-1] * Y.element_size()
42|         n_w_bytes = (W.numel() * W.element_size() // slice_sizes.numel()) * (slice_sizes > 0).sum()
43| 
```
**EN:** Checks `args['RAGGED_DIMENSION'] == 'K'`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch branches on runtime conditions; prepares intermediate values.

**CN:** 检查 `args['RAGGED_DIMENSION'] == 'K'`. 真分支主要准备中间值; 准备中间值；而 else 分支根据运行时条件分支; 准备中间值.

### Block 13 — Lines 44-46 (_old_flops_and_bytes_from_slices)
```python
44|     return {f"flops{nbits}": flops, "bytes": n_x_bytes + n_y_bytes + n_w_bytes}
45| 
46| 
```
**EN:** Returns `{f'flops{nbits}': flops, 'bytes': n_x_bytes + n_y_bytes + n_w_bytes}`.

**CN:** 返回 `{f'flops{nbits}': flops, 'bytes': n_x_bytes + n_y_bytes + n_w_bytes}`.

### Block 14 — Lines 47-47 (_make_slice_sizes)
```python
47| def _make_slice_sizes(n_slices, max_slice_size, active_fraction, device):
```
**EN:** Defines function `_make_slice_sizes(n_slices, max_slice_size, active_fraction, device)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `max`, `torch.zeros`, `torch.randint`, `min`, `torch.randperm` to implement its workflow.

**CN:** 定义函数 `_make_slice_sizes(n_slices, max_slice_size, active_fraction, device)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `max`, `torch.zeros`, `torch.randint`, `min`, `torch.randperm` 来实现其工作流程.

### Block 15 — Lines 48-48 (_make_slice_sizes)
```python
48|     n_active = max(1, min(n_slices, round(n_slices * active_fraction)))
```
**EN:** Assigns `n_active` and calls `max`.

**CN:** 将 `n_active`，并调用 `max`.

### Block 16 — Lines 49-49 (_make_slice_sizes)
```python
49|     slice_sizes = torch.zeros((n_slices, ), dtype=torch.int32, device=device)
```
**EN:** Assigns `slice_sizes` and calls `torch.zeros`.

**CN:** 将 `slice_sizes`，并调用 `torch.zeros`.

### Block 17 — Lines 50-50 (_make_slice_sizes)
```python
50|     active = torch.randperm(n_slices, device=device)[:n_active]
```
**EN:** Assigns `active` and evaluates `torch.randperm(n_slices, device=device)[:n_active]`.

**CN:** 将 `active` and 计算 `torch.randperm(n_slices, device=device)[:n_active]`.

### Block 18 — Lines 51-51 (_make_slice_sizes)
```python
51|     slice_sizes[active] = torch.randint(1, max_slice_size + 1, (n_active, ), dtype=torch.int32, device=device)
```
**EN:** Assigns `slice_sizes[active]` and calls `torch.randint`.

**CN:** 将 `slice_sizes[active]`，并调用 `torch.randint`.

### Block 19 — Lines 52-54 (_make_slice_sizes)
```python
52|     return slice_sizes
53| 
54| 
```
**EN:** Returns `slice_sizes`.

**CN:** 返回 `slice_sizes`.

### Block 20 — Lines 55-55 (_make_case)
```python
55| def _make_case(args, mode, device):
```
**EN:** Defines function `_make_case(args, mode, device)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `getattr`, `_make_slice_sizes`, `FakeTensor`, `object` to implement its workflow.

**CN:** 定义函数 `_make_case(args, mode, device)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `getattr`, `_make_slice_sizes`, `FakeTensor`, `object` 来实现其工作流程.

### Block 21 — Lines 56-56 (_make_case)
```python
56|     dtype = getattr(torch, args.dtype)
```
**EN:** Assigns `dtype` and calls `getattr`.

**CN:** 将 `dtype`，并调用 `getattr`.

### Block 22 — Lines 57-57 (_make_case)
```python
57|     slice_sizes = _make_slice_sizes(args.n_slices, args.max_slice_size, args.active_fraction, device)
```
**EN:** Assigns `slice_sizes` and calls `_make_slice_sizes`.

**CN:** 将 `slice_sizes`，并调用 `_make_slice_sizes`.

### Block 23 — Lines 58-59 (_make_case)
```python
58|     total_capacity = args.n_slices * args.max_slice_size
59| 
```
**EN:** Assigns `total_capacity` and evaluates `args.n_slices * args.max_slice_size`.

**CN:** 将 `total_capacity` and 计算 `args.n_slices * args.max_slice_size`.

### Block 24 — Lines 60-66 (_make_case)
```python
60|     if mode == "ragged_k":
61|         metadata_args = {"RAGGED_DIMENSION": "K", "OutAcc": object() if args.out_acc else None}
62|         X = FakeTensor((args.m, total_capacity), dtype)
63|         Y = FakeTensor((args.m, args.n), dtype)
64|         W = FakeTensor((total_capacity, args.n), dtype)
65|         return metadata_args, args.m, args.n, None, X, Y, W, slice_sizes, dtype.itemsize * 8, 1
66| 
```
**EN:** Checks `mode == 'ragged_k'`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `mode == 'ragged_k'`. 真分支主要准备中间值; 准备中间值.

### Block 25 — Lines 67-67 (_make_case)
```python
67|     metadata_args = {"RAGGED_DIMENSION": "M", "OutAcc": None}
```
**EN:** Assigns `metadata_args` and builds a dictionary.

**CN:** 将 `metadata_args` and 构造一个字典.

### Block 26 — Lines 68-68 (_make_case)
```python
68|     X = FakeTensor((total_capacity, args.k), dtype)
```
**EN:** Assigns `X` and calls `FakeTensor`.

**CN:** 将 `X`，并调用 `FakeTensor`.

### Block 27 — Lines 69-69 (_make_case)
```python
69|     Y = FakeTensor((total_capacity, args.n), dtype)
```
**EN:** Assigns `Y` and calls `FakeTensor`.

**CN:** 将 `Y`，并调用 `FakeTensor`.

### Block 28 — Lines 70-70 (_make_case)
```python
70|     W = FakeTensor((args.n_slices, args.k, args.n), dtype)
```
**EN:** Assigns `W` and calls `FakeTensor`.

**CN:** 将 `W`，并调用 `FakeTensor`.

### Block 29 — Lines 71-73 (_make_case)
```python
71|     return metadata_args, None, args.n, args.k, X, Y, W, slice_sizes, dtype.itemsize * 8, args.batch_size
72| 
73| 
```
**EN:** Returns `(metadata_args, None, args.n, args.k, X, Y, W, slice_sizes, dtype.itemsize * ...`.

**CN:** 返回 `(metadata_args, None, args.n, args.k, X, Y, W, slice_sizes, dtype.itemsize * ...`.

### Block 30 — Lines 74-74 (_bench)
```python
74| def _bench(fn, iters, warmup, device):
```
**EN:** Defines function `_bench(fn, iters, warmup, device)` for this module. The body mainly iterates over cases or dimensions; invokes `torch.cuda.synchronize`; prepares intermediate values. It uses calls such as `range`, `torch.cuda.synchronize`, `torch.cuda.Event`, `time.perf_counter`, `start_event.record` to implement its workflow.

**CN:** 定义函数 `_bench(fn, iters, warmup, device)`，供本模块使用. 主体主要遍历测试用例或维度; invokes `torch.cuda.synchronize`; 准备中间值. 其中会调用 `range`, `torch.cuda.synchronize`, `torch.cuda.Event`, `time.perf_counter`, `start_event.record` 来实现其工作流程.

### Block 31 — Lines 75-76 (_bench)
```python
75|     for _ in range(warmup):
76|         fn()
```
**EN:** Loops over `range(warmup)` with target `_`. The loop body mainly invokes `fn`.

**CN:** 遍历 `range(warmup)` ，目标变量为 `_`. 循环体主要invokes `fn`.

### Block 32 — Lines 77-78 (_bench)
```python
77|     torch.cuda.synchronize(device)
78| 
```
**EN:** Calls `torch.cuda.synchronize` for side effects, registration, or validation.

**CN:** 调用 `torch.cuda.synchronize` ，用于副作用、注册或校验。

### Block 33 — Lines 79-79 (_bench)
```python
79|     start_event = torch.cuda.Event(enable_timing=True)
```
**EN:** Assigns `start_event` and calls `torch.cuda.Event`.

**CN:** 将 `start_event`，并调用 `torch.cuda.Event`.

### Block 34 — Lines 80-80 (_bench)
```python
80|     end_event = torch.cuda.Event(enable_timing=True)
```
**EN:** Assigns `end_event` and calls `torch.cuda.Event`.

**CN:** 将 `end_event`，并调用 `torch.cuda.Event`.

### Block 35 — Lines 81-81 (_bench)
```python
81|     wall_start = time.perf_counter()
```
**EN:** Assigns `wall_start` and calls `time.perf_counter`.

**CN:** 将 `wall_start`，并调用 `time.perf_counter`.

### Block 36 — Lines 82-82 (_bench)
```python
82|     start_event.record()
```
**EN:** Calls `start_event.record` for side effects, registration, or validation.

**CN:** 调用 `start_event.record` ，用于副作用、注册或校验。

### Block 37 — Lines 83-83 (_bench)
```python
83|     last = None
```
**EN:** Assigns `last` and stores constant `None`.

**CN:** 将 `last` and 保存常量 `None`.

### Block 38 — Lines 84-85 (_bench)
```python
84|     for _ in range(iters):
85|         last = fn()
```
**EN:** Loops over `range(iters)` with target `_`. The loop body mainly prepares intermediate values.

**CN:** 遍历 `range(iters)` ，目标变量为 `_`. 循环体主要准备中间值.

### Block 39 — Lines 86-86 (_bench)
```python
86|     end_event.record()
```
**EN:** Calls `end_event.record` for side effects, registration, or validation.

**CN:** 调用 `end_event.record` ，用于副作用、注册或校验。

### Block 40 — Lines 87-87 (_bench)
```python
87|     torch.cuda.synchronize(device)
```
**EN:** Calls `torch.cuda.synchronize` for side effects, registration, or validation.

**CN:** 调用 `torch.cuda.synchronize` ，用于副作用、注册或校验。

### Block 41 — Lines 88-90 (_bench)
```python
88|     wall_s = time.perf_counter() - wall_start
89|     # Touch the result once after timing to catch bad kernels without putting .item()
90|     # synchronization in the measured loop.
```
**EN:** Assigns `wall_s` and evaluates `time.perf_counter() - wall_start`.

**CN:** 将 `wall_s` and 计算 `time.perf_counter() - wall_start`.

### Block 42 — Lines 91-92 (_bench)
```python
91|     for value in last.values():
92|         assert value.numel() == 1
```
**EN:** Loops over `last.values()` with target `value`. The loop body mainly checks invariants.

**CN:** 遍历 `last.values()` ，目标变量为 `value`. 循环体主要检查不变量.

### Block 43 — Lines 93-95 (_bench)
```python
93|     return start_event.elapsed_time(end_event) * 1000.0 / iters, wall_s * 1e6 / iters
94| 
95| 
```
**EN:** Returns `(start_event.elapsed_time(end_event) * 1000.0 / iters, wall_s * 1000000.0 / i...`.

**CN:** 返回 `(start_event.elapsed_time(end_event) * 1000.0 / iters, wall_s * 1000000.0 / i...`.

### Block 44 — Lines 96-96 (_run_case)
```python
96| def _run_case(args, mode, device):
```
**EN:** Defines function `_run_case(args, mode, device)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `_make_case`, `old_fn`, `new_fn`, `torch.cuda.synchronize`, `torch.testing.assert_close` to implement its workflow.

**CN:** 定义函数 `_run_case(args, mode, device)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `_make_case`, `old_fn`, `new_fn`, `torch.cuda.synchronize`, `torch.testing.assert_close` 来实现其工作流程.

### Block 45 — Lines 97-97 (_run_case)
```python
97|     case = _make_case(args, mode, device)
```
**EN:** Assigns `case` and calls `_make_case`.

**CN:** 将 `case`，并调用 `_make_case`.

### Block 46 — Lines 98-99 (_run_case)
```python
98|     metadata_args, M, N, K, X, Y, W, slice_sizes, nbits, batch_size = case
99| 
```
**EN:** Assigns `metadata_args`, `M`, `N`, `K`, `X`, `Y` and references `case`.

**CN:** 将 `metadata_args`, `M`, `N`, `K`, `X`, `Y` and 引用 `case`.

### Block 47 — Lines 100-100 (old_fn)
```python
100|     def old_fn():
```
**EN:** Defines function `old_fn()` for this module. The body mainly returns the computed result. It uses calls such as `_old_flops_and_bytes_from_slices` to implement its workflow.

**CN:** 定义函数 `old_fn()`，供本模块使用. 主体主要返回计算结果. 其中会调用 `_old_flops_and_bytes_from_slices` 来实现其工作流程.

### Block 48 — Lines 101-102 (old_fn)
```python
101|         return _old_flops_and_bytes_from_slices(metadata_args, M, N, K, X, Y, W, slice_sizes, nbits, batch_size)
102| 
```
**EN:** Returns `_old_flops_and_bytes_from_slices(metadata_args, M, N, K, X, Y, W, slice_sizes...`.

**CN:** 返回 `_old_flops_and_bytes_from_slices(metadata_args, M, N, K, X, Y, W, slice_sizes...`.

### Block 49 — Lines 103-103 (new_fn)
```python
103|     def new_fn():
```
**EN:** Defines function `new_fn()` for this module. The body mainly returns the computed result. It uses calls such as `_matmul_flops_and_bytes_from_slices` to implement its workflow.

**CN:** 定义函数 `new_fn()`，供本模块使用. 主体主要返回计算结果. 其中会调用 `_matmul_flops_and_bytes_from_slices` 来实现其工作流程.

### Block 50 — Lines 104-105 (new_fn)
```python
104|         return _matmul_flops_and_bytes_from_slices(metadata_args, M, N, K, X, Y, W, slice_sizes, nbits, batch_size)
105| 
```
**EN:** Returns `_matmul_flops_and_bytes_from_slices(metadata_args, M, N, K, X, Y, W, slice_si...`.

**CN:** 返回 `_matmul_flops_and_bytes_from_slices(metadata_args, M, N, K, X, Y, W, slice_si...`.

### Block 51 — Lines 106-106 (_run_case)
```python
106|     old = old_fn()
```
**EN:** Assigns `old` and calls `old_fn`.

**CN:** 将 `old`，并调用 `old_fn`.

### Block 52 — Lines 107-107 (_run_case)
```python
107|     new = new_fn()
```
**EN:** Assigns `new` and calls `new_fn`.

**CN:** 将 `new`，并调用 `new_fn`.

### Block 53 — Lines 108-108 (_run_case)
```python
108|     torch.cuda.synchronize(device)
```
**EN:** Calls `torch.cuda.synchronize` for side effects, registration, or validation.

**CN:** 调用 `torch.cuda.synchronize` ，用于副作用、注册或校验。

### Block 54 — Lines 109-109 (_run_case)
```python
109|     torch.testing.assert_close(new[f"flops{nbits}"].cpu(), old[f"flops{nbits}"].cpu(), rtol=0, atol=0)
```
**EN:** Calls `torch.testing.assert_close` for side effects, registration, or validation.

**CN:** 调用 `torch.testing.assert_close` ，用于副作用、注册或校验。

### Block 55 — Lines 110-111 (_run_case)
```python
110|     torch.testing.assert_close(new["bytes"].cpu(), old["bytes"].to(torch.int64).cpu(), rtol=0, atol=0)
111| 
```
**EN:** Calls `torch.testing.assert_close` for side effects, registration, or validation.

**CN:** 调用 `torch.testing.assert_close` ，用于副作用、注册或校验。

### Block 56 — Lines 112-112 (_run_case)
```python
112|     old_gpu_us, old_wall_us = _bench(old_fn, args.iters, args.warmup, device)
```
**EN:** Assigns `old_gpu_us`, `old_wall_us` and calls `_bench`.

**CN:** 将 `old_gpu_us`, `old_wall_us`，并调用 `_bench`.

### Block 57 — Lines 113-113 (_run_case)
```python
113|     new_gpu_us, new_wall_us = _bench(new_fn, args.iters, args.warmup, device)
```
**EN:** Assigns `new_gpu_us`, `new_wall_us` and calls `_bench`.

**CN:** 将 `new_gpu_us`, `new_wall_us`，并调用 `_bench`.

### Block 58 — Lines 114-121 (_run_case)
```python
114|     print(
115|         f"{mode},{args.n_slices},{args.max_slice_size},{args.active_fraction},"
116|         f"{old_gpu_us:.3f},{new_gpu_us:.3f},{old_gpu_us / new_gpu_us:.2f},"
117|         f"{old_wall_us:.3f},{new_wall_us:.3f},{old_wall_us / new_wall_us:.2f}",
118|         flush=True,
119|     )
120| 
121| 
```
**EN:** Calls `print` for side effects, registration, or validation.

**CN:** 调用 `print` ，用于副作用、注册或校验。

### Block 59 — Lines 122-122 (main)
```python
122| def main():
```
**EN:** Defines function `main()` for this module. The body mainly prepares intermediate values; invokes `parser.add_argument`; invokes `parser.add_argument`. It uses calls such as `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `torch.cuda.set_device`, `torch.device` to implement its workflow.

**CN:** 定义函数 `main()`，供本模块使用. 主体主要准备中间值; invokes `parser.add_argument`; invokes `parser.add_argument`. 其中会调用 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `torch.cuda.set_device`, `torch.device` 来实现其工作流程.

### Block 60 — Lines 123-123 (main)
```python
123|     parser = argparse.ArgumentParser(description="Benchmark ragged matmul launch-metadata flops/bytes counters.")
```
**EN:** Assigns `parser` and calls `argparse.ArgumentParser`.

**CN:** 将 `parser`，并调用 `argparse.ArgumentParser`.

### Block 61 — Lines 124-124 (main)
```python
124|     parser.add_argument("--mode", choices=("ragged_m", "ragged_k", "both"), default="both")
```
**EN:** Calls `parser.add_argument` for side effects, registration, or validation.

**CN:** 调用 `parser.add_argument` ，用于副作用、注册或校验。

### Block 62 — Lines 125-125 (main)
```python
125|     parser.add_argument("--dtype", choices=("float16", "bfloat16"), default="float16")
```
**EN:** Calls `parser.add_argument` for side effects, registration, or validation.

**CN:** 调用 `parser.add_argument` ，用于副作用、注册或校验。

### Block 63 — Lines 126-126 (main)
```python
126|     parser.add_argument("--device", type=int, default=0)
```
**EN:** Calls `parser.add_argument` for side effects, registration, or validation.

**CN:** 调用 `parser.add_argument` ，用于副作用、注册或校验。

### Block 64 — Lines 127-127 (main)
```python
127|     parser.add_argument("--m", type=int, default=128)
```
**EN:** Calls `parser.add_argument` for side effects, registration, or validation.

**CN:** 调用 `parser.add_argument` ，用于副作用、注册或校验。

### Block 65 — Lines 128-128 (main)
```python
128|     parser.add_argument("--n", type=int, default=8192)
```
**EN:** Calls `parser.add_argument` for side effects, registration, or validation.

**CN:** 调用 `parser.add_argument` ，用于副作用、注册或校验。

### Block 66 — Lines 129-129 (main)
```python
129|     parser.add_argument("--k", type=int, default=8192)
```
**EN:** Calls `parser.add_argument` for side effects, registration, or validation.

**CN:** 调用 `parser.add_argument` ，用于副作用、注册或校验。

### Block 67 — Lines 130-130 (main)
```python
130|     parser.add_argument("--batch-size", type=int, default=1)
```
**EN:** Calls `parser.add_argument` for side effects, registration, or validation.

**CN:** 调用 `parser.add_argument` ，用于副作用、注册或校验。

### Block 68 — Lines 131-131 (main)
```python
131|     parser.add_argument("--n-slices", type=int, default=256)
```
**EN:** Calls `parser.add_argument` for side effects, registration, or validation.

**CN:** 调用 `parser.add_argument` ，用于副作用、注册或校验。

### Block 69 — Lines 132-132 (main)
```python
132|     parser.add_argument("--max-slice-size", type=int, default=256)
```
**EN:** Calls `parser.add_argument` for side effects, registration, or validation.

**CN:** 调用 `parser.add_argument` ，用于副作用、注册或校验。

### Block 70 — Lines 133-133 (main)
```python
133|     parser.add_argument("--active-fraction", type=float, default=0.8)
```
**EN:** Calls `parser.add_argument` for side effects, registration, or validation.

**CN:** 调用 `parser.add_argument` ，用于副作用、注册或校验。

### Block 71 — Lines 134-134 (main)
```python
134|     parser.add_argument("--out-acc", action="store_true")
```
**EN:** Calls `parser.add_argument` for side effects, registration, or validation.

**CN:** 调用 `parser.add_argument` ，用于副作用、注册或校验。

### Block 72 — Lines 135-135 (main)
```python
135|     parser.add_argument("--warmup", type=int, default=25)
```
**EN:** Calls `parser.add_argument` for side effects, registration, or validation.

**CN:** 调用 `parser.add_argument` ，用于副作用、注册或校验。

### Block 73 — Lines 136-136 (main)
```python
136|     parser.add_argument("--iters", type=int, default=500)
```
**EN:** Calls `parser.add_argument` for side effects, registration, or validation.

**CN:** 调用 `parser.add_argument` ，用于副作用、注册或校验。

### Block 74 — Lines 137-138 (main)
```python
137|     args = parser.parse_args()
138| 
```
**EN:** Assigns `args` and calls `parser.parse_args`.

**CN:** 将 `args`，并调用 `parser.parse_args`.

### Block 75 — Lines 139-139 (main)
```python
139|     torch.cuda.set_device(args.device)
```
**EN:** Calls `torch.cuda.set_device` for side effects, registration, or validation.

**CN:** 调用 `torch.cuda.set_device` ，用于副作用、注册或校验。

### Block 76 — Lines 140-140 (main)
```python
140|     device = torch.device("cuda", args.device)
```
**EN:** Assigns `device` and calls `torch.device`.

**CN:** 将 `device`，并调用 `torch.device`.

### Block 77 — Lines 141-142 (main)
```python
141|     torch.manual_seed(0)
142| 
```
**EN:** Calls `torch.manual_seed` for side effects, registration, or validation.

**CN:** 调用 `torch.manual_seed` ，用于副作用、注册或校验。

### Block 78 — Lines 143-143 (main)
```python
143|     modes = ("ragged_m", "ragged_k") if args.mode == "both" else (args.mode, )
```
**EN:** Assigns `modes` and uses conditional expression `('ragged_m', 'ragged_k') if args.mode == 'both' else (arg...`.

**CN:** 将 `modes` and 使用条件表达式 `('ragged_m', 'ragged_k') if args.mode == 'both' else (arg...`.

### Block 79 — Lines 144-146 (main)
```python
144|     print(
145|         "mode,n_slices,max_slice_size,active_fraction,old_gpu_us,new_gpu_us,gpu_speedup,old_wall_us,new_wall_us,wall_speedup"
146|     )
```
**EN:** Calls `print` for side effects, registration, or validation.

**CN:** 调用 `print` ，用于副作用、注册或校验。

### Block 80 — Lines 147-150 (main)
```python
147|     for mode in modes:
148|         _run_case(args, mode, device)
149| 
150| 
```
**EN:** Loops over `modes` with target `mode`. The loop body mainly invokes `_run_case`.

**CN:** 遍历 `modes` ，目标变量为 `mode`. 循环体主要invokes `_run_case`.

### Block 81 — Lines 151-152 (module)
```python
151| if __name__ == "__main__":
152|     main()
```
**EN:** Checks `__name__ == '__main__'`. The true branch mainly invokes `main`.

**CN:** 检查 `__name__ == '__main__'`. 真分支主要invokes `main`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `FakeTensor`, `_old_flops_and_bytes_from_slices`, `_make_slice_sizes`, `_make_case`, `_bench`, `_run_case`, `main`.
  **CN:** 主要符号：`FakeTensor`, `_old_flops_and_bytes_from_slices`, `_make_slice_sizes`, `_make_case`, `_bench`, `_run_case`, `main`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Benchmark-oriented code sweeps parameter spaces and records performance metrics.
  **CN:** 面向基准测试的代码会扫描参数空间并记录性能指标。
- **EN:** Ragged tensor metadata is used when rows or slices have non-uniform sizes.
  **CN:** 当行或切片大小不一致时，会使用 ragged tensor 元数据。

## Dependencies / 依赖关系
- **EN:** External modules: `argparse`, `dataclasses`, `math`, `time`, `torch`.
  **CN:** 外部模块：`argparse`, `dataclasses`, `math`, `time`, `torch`。
- **EN:** Internal modules: `triton_kernels.matmul_details._common (_matmul_flops_and_bytes_from_slices)`.
  **CN:** 内部模块：`triton_kernels.matmul_details._common (_matmul_flops_and_bytes_from_slices)`。
