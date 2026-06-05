# bench_client.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/hf3fs/bench_client.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on hf3fs client. It primarily measures runtime latency, throughput, or other benchmark metrics, coordinates asynchronous or parallel execution, and prepares tensors and invokes GPU kernels. / 该 Python 模块聚焦于 hf3fs client 相关流程。它主要用于测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行、准备张量并调用 GPU 内核。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Imports and setup / 导入与初始化
```python
import concurrent.futures
import logging
import random
import time
from typing import List

import torch
from tqdm import tqdm

from sglang.srt.mem_cache.storage.hf3fs.hf3fs_usrbio_client import Hf3fsUsrBioClient
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it coordinates asynchronous or parallel execution and prepares tensors and invokes GPU kernels.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会协调异步或并行执行、准备张量并调用 GPU 内核。

### Lines 13-23: Function `print_stats` / 函数 `print_stats`
```python
def print_stats(x: List[int]):
    x = sorted(x)
    lenx = len(x)
    print(
        f"mean = {sum(x)/len(x):.2f}, "
        f"min = {min(x):.2f}, "
        f"p25 = {x[int(lenx*0.25)]:.2f}, "
        f"p50 = {x[int(lenx*0.5)]:.2f}, "
        f"p75 = {x[int(lenx*0.75)]:.2f}, "
        f"max = {max(x):.2f}"
    )
```
**EN:** `print_stats` is a function that implements the core logic for this scope. Notable calls include `sorted`, `len`, `print`.
**CN:** `print_stats` 是一个函数，用于实现当前作用域的核心逻辑。其中较关键的调用包括 `sorted`, `len`, `print`。

### Lines 26-63: Function `test` / 函数 `test`
```python
def test():
    # /path/to/hf3fs
    file_path = "/data/bench.bin"
    file_size = 1 << 40
    bytes_per_page = 16 << 20
    entries = 32
    file_ops = Hf3fsUsrBioClient(file_path, file_size, bytes_per_page, entries, 5)

    print("test batch_read / batch_write")
    num_pages = 128
    dtype = torch.bfloat16
    numel = bytes_per_page // dtype.itemsize
    offsets = list(range(file_size // bytes_per_page))
    random.shuffle(offsets)
    offsets = offsets[:num_pages]
    offsets = [i * bytes_per_page for i in offsets]
    tensor_writes = [
        torch.randn(numel, dtype=dtype)
        for _ in tqdm(range(num_pages), desc="prepare tensor")
    ]
    for i in tqdm(range(0, num_pages, file_ops.entries), desc="batch_write"):
        results = file_ops.batch_write(
            offsets[i : i + file_ops.entries], tensor_writes[i : i + file_ops.entries]
        )
        assert all([result == numel * dtype.itemsize for result in results])
    tensor_reads = [
        torch.empty(numel, dtype=dtype)
        for _ in tqdm(range(num_pages), desc="prepare tensor")
    ]
    for i in tqdm(range(0, num_pages, file_ops.entries), desc="batch_read"):
        results = file_ops.batch_read(
            offsets[i : i + file_ops.entries], tensor_reads[i : i + file_ops.entries]
        )
        assert all([result == numel * dtype.itemsize for result in results])
    assert all([torch.allclose(r, w) for r, w in zip(tensor_reads, tensor_writes)])

    file_ops.close()
    print("test done")
```
**EN:** `test` is a function that prepares tensors and invokes GPU kernels. Notable calls include `Hf3fsUsrBioClient`, `print`, `list`.
**CN:** `test` 是一个函数，用于准备张量并调用 GPU 内核。其中较关键的调用包括 `Hf3fsUsrBioClient`, `print`, `list`。

### Lines 66-152: Function `bench` / 函数 `bench`
```python
def bench():
    file_path = "/data/bench.bin"
    file_size = 1 << 40
    bytes_per_page = 16 << 20
    entries = 8
    numjobs = 16

    dtype = torch.bfloat16
    numel = bytes_per_page // dtype.itemsize

    file_ops = [
        Hf3fsUsrBioClient(file_path, file_size, bytes_per_page, entries, 5)
        for _ in range(numjobs)
    ]

    num_page = entries

    offsets = list(range(file_size // bytes_per_page))
    tensors_write = [torch.randn(numel, dtype=dtype)] * num_page
    tensors_read = [torch.empty(numel, dtype=dtype)] * num_page
    random.shuffle(offsets)

    warmup = 50
    iteration = 100

    executor = concurrent.futures.ThreadPoolExecutor(max_workers=numjobs)

    w_bw = []
    w_size = num_page * numjobs * bytes_per_page / (1 << 30)
    for i in tqdm(range(warmup + iteration), desc="Benchmarking write (GB/s)"):
        _offsets = [
            [
                offset * bytes_per_page
                for offset in offsets[
                    (i * numjobs + j) * num_page : (i * numjobs + j + 1) * num_page
                ]
            ]
            for j in range(numjobs)
        ]
        tik = time.perf_counter()
        futures = [
            executor.submit(file_ops[j].batch_write, offset, tensors_write)
            for j, offset in enumerate(_offsets)
        ]
        results = [future.result() for future in futures]
        tok = time.perf_counter()
        if i < warmup:
            continue
        w_bw.append(w_size / (tok - tik))
        results = [
            _result == bytes_per_page for result in results for _result in result
        ]
        assert all(results)
    print_stats(w_bw)

    r_bw = []
    r_size = w_size
    for i in tqdm(range(warmup + iteration), desc="Benchmarking read (GB/s)"):
        _offsets = [
            [
                offset * bytes_per_page
                for offset in offsets[
                    (i * numjobs + j) * num_page : (i * numjobs + j + 1) * num_page
                ]
            ]
            for j in range(numjobs)
        ]
        tik = time.perf_counter()
        futures = [
            executor.submit(file_ops[j].batch_read, offset, tensors_read)
            for j, offset in enumerate(_offsets)
        ]
        results = [future.result() for future in futures]
        tok = time.perf_counter()
        if i < warmup:
            continue
        r_bw.append(r_size / (tok - tik))
        results = [
            _result == bytes_per_page for result in results for _result in result
        ]
        assert all(results)
    print_stats(r_bw)

    executor.shutdown(wait=True)
    for _file_ops in file_ops:
        _file_ops.close()
    print("bench done")
```
**EN:** `bench` is a function that measures runtime latency, throughput, or other benchmark metrics, coordinates asynchronous or parallel execution, and prepares tensors and invokes GPU kernels. Notable calls include `list`, `random.shuffle`, `concurrent.futures.ThreadPoolExecutor`.
**CN:** `bench` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行、准备张量并调用 GPU 内核。其中较关键的调用包括 `list`, `random.shuffle`, `concurrent.futures.ThreadPoolExecutor`。

### Lines 155-158: Function `main` / 函数 `main`
```python
def main():
    logging.basicConfig(level=logging.INFO)
    test()
    bench()
```
**EN:** `main` is a function that implements the core logic for this scope. Notable calls include `logging.basicConfig`, `test`, `bench`.
**CN:** `main` 是一个函数，用于实现当前作用域的核心逻辑。其中较关键的调用包括 `logging.basicConfig`, `test`, `bench`。

### Lines 161-162: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    main()
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Concurrency / 并发**: Coordinates multiple requests or workers at the same time. / 同时协调多个请求或工作单元。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `concurrent.futures`, `logging`, `random`, `time`, `typing`
- **Third-party / 第三方依赖**: `torch`, `tqdm`
- **Internal / 项目内部依赖**: `sglang.srt.mem_cache.storage.hf3fs.hf3fs_usrbio_client`
