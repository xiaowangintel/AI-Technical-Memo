# bench_reduce.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/bench/bench_reduce.py`
- **Purpose / 用途:** Benchmark driver for bench reduce; it measures performance, sweeps configurations, and reports results for Triton kernels. / 用于 bench reduce 的基准测试驱动；它会测量性能、扫描配置，并报告 Triton 内核的结果。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-8 (module)
```python
1| import argparse
2| import statistics
3| 
4| import torch
5| 
6| from triton_kernels.reduce import reduce, _select_reduce_forward_config
7| 
8| 
```
**EN:** This block imports `argparse`, `statistics`, `torch`, `triton_kernels.reduce (reduce, _select_reduce_forward_config)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `argparse`, `statistics`, `torch`, `triton_kernels.reduce (reduce, _select_reduce_forward_config)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 9-9 (_csv_ints)
```python
9| def _csv_ints(s):
```
**EN:** Defines function `_csv_ints(s)` for this module. The body mainly returns the computed result. It uses calls such as `int`, `s.split` to implement its workflow.

**CN:** 定义函数 `_csv_ints(s)`，供本模块使用. 主体主要返回计算结果. 其中会调用 `int`, `s.split` 来实现其工作流程.

### Block 3 — Lines 10-12 (_csv_ints)
```python
10|     return [int(x) for x in s.split(",") if x]
11| 
12| 
```
**EN:** Returns `[int(x) for x in s.split(',') if x]`.

**CN:** 返回 `[int(x) for x in s.split(',') if x]`.

### Block 4 — Lines 13-13 (_flush_cache)
```python
13| def _flush_cache(cache_killer):
```
**EN:** Defines function `_flush_cache(cache_killer)` for this module. The body mainly branches on runtime conditions. It uses calls such as `cache_killer.add_` to implement its workflow.

**CN:** 定义函数 `_flush_cache(cache_killer)`，供本模块使用. 主体主要根据运行时条件分支. 其中会调用 `cache_killer.add_` 来实现其工作流程.

### Block 5 — Lines 14-17 (_flush_cache)
```python
14|     if cache_killer is not None:
15|         cache_killer.add_(1.0)
16| 
17| 
```
**EN:** Checks `cache_killer is not None`. The true branch mainly invokes `cache_killer.add_`.

**CN:** 检查 `cache_killer is not None`. 真分支主要invokes `cache_killer.add_`.

### Block 6 — Lines 18-18 (bench_reduce)
```python
18| def bench_reduce(k, s0, s1, iters, cache_killer):
```
**EN:** Defines function `bench_reduce(k, s0, s1, iters, cache_killer)` for this module. The body mainly prepares intermediate values; iterates over cases or dimensions; invokes `torch.cuda.synchronize`. It uses calls such as `torch.randn`, `range`, `torch.cuda.synchronize`, `torch.cuda.Event`, `times_ms.sort` to implement its workflow.

**CN:** 定义函数 `bench_reduce(k, s0, s1, iters, cache_killer)`，供本模块使用. 主体主要准备中间值; 遍历测试用例或维度; invokes `torch.cuda.synchronize`. 其中会调用 `torch.randn`, `range`, `torch.cuda.synchronize`, `torch.cuda.Event`, `times_ms.sort` 来实现其工作流程.

### Block 7 — Lines 19-19 (bench_reduce)
```python
19|     x = torch.randn((k, s0, s1), device="cuda", dtype=torch.float32)
```
**EN:** Assigns `x` and calls `torch.randn`.

**CN:** 将 `x`，并调用 `torch.randn`.

### Block 8 — Lines 20-22 (bench_reduce)
```python
20|     for _ in range(10):
21|         _flush_cache(cache_killer)
22|         reduce(x, dim=0, y_dtype=torch.bfloat16)
```
**EN:** Loops over `range(10)` with target `_`. The loop body mainly invokes `_flush_cache`; invokes `reduce`.

**CN:** 遍历 `range(10)` ，目标变量为 `_`. 循环体主要invokes `_flush_cache`; invokes `reduce`.

### Block 9 — Lines 23-24 (bench_reduce)
```python
23|     torch.cuda.synchronize()
24| 
```
**EN:** Calls `torch.cuda.synchronize` for side effects, registration, or validation.

**CN:** 调用 `torch.cuda.synchronize` ，用于副作用、注册或校验。

### Block 10 — Lines 25-25 (bench_reduce)
```python
25|     times_ms = []
```
**EN:** Assigns `times_ms` and builds a list.

**CN:** 将 `times_ms` and 构造一个列表.

### Block 11 — Lines 26-26 (bench_reduce)
```python
26|     start = torch.cuda.Event(enable_timing=True)
```
**EN:** Assigns `start` and calls `torch.cuda.Event`.

**CN:** 将 `start`，并调用 `torch.cuda.Event`.

### Block 12 — Lines 27-27 (bench_reduce)
```python
27|     end = torch.cuda.Event(enable_timing=True)
```
**EN:** Assigns `end` and calls `torch.cuda.Event`.

**CN:** 将 `end`，并调用 `torch.cuda.Event`.

### Block 13 — Lines 28-34 (bench_reduce)
```python
28|     for _ in range(iters):
29|         _flush_cache(cache_killer)
30|         start.record()
31|         reduce(x, dim=0, y_dtype=torch.bfloat16)
32|         end.record()
33|         torch.cuda.synchronize()
34|         times_ms.append(start.elapsed_time(end))
```
**EN:** Loops over `range(iters)` with target `_`. The loop body mainly invokes `_flush_cache`; invokes `start.record`.

**CN:** 遍历 `range(iters)` ，目标变量为 `_`. 循环体主要invokes `_flush_cache`; invokes `start.record`.

### Block 14 — Lines 35-35 (bench_reduce)
```python
35|     times_ms.sort()
```
**EN:** Calls `times_ms.sort` for side effects, registration, or validation.

**CN:** 调用 `times_ms.sort` ，用于副作用、注册或校验。

### Block 15 — Lines 36-38 (bench_reduce)
```python
36|     return statistics.median(times_ms), statistics.mean(times_ms), times_ms[int(0.9 * (iters - 1))]
37| 
38| 
```
**EN:** Returns `(statistics.median(times_ms), statistics.mean(times_ms), times_ms[int(0.9 * (...`.

**CN:** 返回 `(statistics.median(times_ms), statistics.mean(times_ms), times_ms[int(0.9 * (...`.

### Block 16 — Lines 39-39 (main)
```python
39| def main():
```
**EN:** Defines function `main()` for this module. The body mainly prepares intermediate values; invokes `parser.add_argument`; invokes `parser.add_argument`. It uses calls such as `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `print`, `_csv_ints` to implement its workflow.

**CN:** 定义函数 `main()`，供本模块使用. 主体主要准备中间值; invokes `parser.add_argument`; invokes `parser.add_argument`. 其中会调用 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `print`, `_csv_ints` 来实现其工作流程.

### Block 17 — Lines 40-40 (main)
```python
40|     parser = argparse.ArgumentParser(description="Benchmark wide-S1 reduce_forward shapes.")
```
**EN:** Assigns `parser` and calls `argparse.ArgumentParser`.

**CN:** 将 `parser`，并调用 `argparse.ArgumentParser`.

### Block 18 — Lines 41-41 (main)
```python
41|     parser.add_argument("--ks", default="1,2,3,4,5,6,7,8")
```
**EN:** Calls `parser.add_argument` for side effects, registration, or validation.

**CN:** 调用 `parser.add_argument` ，用于副作用、注册或校验。

### Block 19 — Lines 42-42 (main)
```python
42|     parser.add_argument("--s0s", default="1,2,4,8,16,32,64,128,256")
```
**EN:** Calls `parser.add_argument` for side effects, registration, or validation.

**CN:** 调用 `parser.add_argument` ，用于副作用、注册或校验。

### Block 20 — Lines 43-43 (main)
```python
43|     parser.add_argument("--s1s", default="1024,2048,4096,8192,16384,32768")
```
**EN:** Calls `parser.add_argument` for side effects, registration, or validation.

**CN:** 调用 `parser.add_argument` ，用于副作用、注册或校验。

### Block 21 — Lines 44-44 (main)
```python
44|     parser.add_argument("--iters", type=int, default=80)
```
**EN:** Calls `parser.add_argument` for side effects, registration, or validation.

**CN:** 调用 `parser.add_argument` ，用于副作用、注册或校验。

### Block 22 — Lines 45-50 (main)
```python
45|     parser.add_argument(
46|         "--flush-mb",
47|         type=int,
48|         default=512,
49|         help="Touch this many MiB before each measured reduce. Set to 0 to benchmark hot-cache repeats.",
50|     )
```
**EN:** Calls `parser.add_argument` for side effects, registration, or validation.

**CN:** 调用 `parser.add_argument` ，用于副作用、注册或校验。

### Block 23 — Lines 51-52 (main)
```python
51|     args = parser.parse_args()
52| 
```
**EN:** Assigns `args` and calls `parser.parse_args`.

**CN:** 将 `args`，并调用 `parser.parse_args`.

### Block 24 — Lines 53-53 (main)
```python
53|     cache_killer = None
```
**EN:** Assigns `cache_killer` and stores constant `None`.

**CN:** 将 `cache_killer` and 保存常量 `None`.

### Block 25 — Lines 54-58 (main)
```python
54|     if args.flush_mb > 0:
55|         n_elements = args.flush_mb * 1024 * 1024 // torch.empty((), dtype=torch.float32).element_size()
56|         cache_killer = torch.empty(n_elements, device="cuda", dtype=torch.float32)
57|         cache_killer.zero_()
58| 
```
**EN:** Checks `args.flush_mb > 0`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `args.flush_mb > 0`. 真分支主要准备中间值; 准备中间值.

### Block 26 — Lines 59-59 (main)
```python
59|     print("K,S0,Y_S1,BLOCK_S0,BLOCK_S1,median_ms,mean_ms,p90_ms", flush=True)
```
**EN:** Calls `print` for side effects, registration, or validation.

**CN:** 调用 `print` ，用于副作用、注册或校验。

### Block 27 — Lines 60-69 (main)
```python
60|     for s1 in _csv_ints(args.s1s):
61|         for k in _csv_ints(args.ks):
62|             for s0 in _csv_ints(args.s0s):
63|                 opt_flags = _select_reduce_forward_config(s0, s1, 1, k, False)
64|                 median_ms, mean_ms, p90_ms = bench_reduce(k, s0, s1, args.iters, cache_killer)
65|                 print(
66|                     f"{k},{s0},{s1},{opt_flags.block_s0},{opt_flags.block_x_s1},{median_ms:.6f},{mean_ms:.6f},{p90_ms:.6f}",
67|                     flush=True)
68| 
69| 
```
**EN:** Loops over `_csv_ints(args.s1s)` with target `s1`. The loop body mainly iterates over cases or dimensions.

**CN:** 遍历 `_csv_ints(args.s1s)` ，目标变量为 `s1`. 循环体主要遍历测试用例或维度.

### Block 28 — Lines 70-71 (module)
```python
70| if __name__ == "__main__":
71|     main()
```
**EN:** Checks `__name__ == '__main__'`. The true branch mainly invokes `main`.

**CN:** 检查 `__name__ == '__main__'`. 真分支主要invokes `main`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `_csv_ints`, `_flush_cache`, `bench_reduce`, `main`.
  **CN:** 主要符号：`_csv_ints`, `_flush_cache`, `bench_reduce`, `main`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Reduction logic combines partial values across dimensions or shards.
  **CN:** 归约逻辑会沿着维度或分片合并部分结果。
- **EN:** Benchmark-oriented code sweeps parameter spaces and records performance metrics.
  **CN:** 面向基准测试的代码会扫描参数空间并记录性能指标。

## Dependencies / 依赖关系
- **EN:** External modules: `argparse`, `statistics`, `torch`.
  **CN:** 外部模块：`argparse`, `statistics`, `torch`。
- **EN:** Internal modules: `triton_kernels.reduce (reduce, _select_reduce_forward_config)`.
  **CN:** 内部模块：`triton_kernels.reduce (reduce, _select_reduce_forward_config)`。
