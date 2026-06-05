# bench_storage.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/hf3fs/bench_storage.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on hf3fs storage. It primarily sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. / 该 Python 模块聚焦于 hf3fs storage 相关流程。它主要用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Imports and setup / 导入与初始化
```python
import json
import logging
import os
import random
import time
from typing import List

import torch
from tqdm import tqdm

from sglang.srt.mem_cache.storage.hf3fs.mini_3fs_metadata_server import (
    Hf3fsLocalMetadataClient,
)
from sglang.srt.mem_cache.storage.hf3fs.storage_hf3fs import HiCacheHF3FS
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it prepares tensors and invokes GPU kernels and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集。

### Lines 17-27: Function `print_stats` / 函数 `print_stats`
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

### Lines 30-126: Function `test` / 函数 `test`
```python
def test():
    # Qwen3-32B
    layer_num = 64
    head_num, head_dim = 8, 128
    kv_lora_rank, qk_rope_head_dim = 0, 0
    store_dtype = torch.bfloat16
    tokens_per_page = 64

    file_path_prefix = "/data/test"
    file_size = 128 << 20
    numjobs = 16
    bytes_per_page = 16 << 20
    entries = 2
    dtype = store_dtype

    config_path = os.getenv(HiCacheHF3FS.default_env_var)
    assert config_path
    try:
        with open(config_path, "w") as f:
            json.dump(
                {
                    "file_path_prefix": file_path_prefix,
                    "file_size": file_size,
                    "numjobs": numjobs,
                    "entries": entries,
                },
                f,
            )
    except Exception as e:
        raise RuntimeError(f"Failed to dump config to {config_path}: {str(e)}")
    hicache_hf3fs = HiCacheHF3FS.from_env_config(bytes_per_page, dtype)

    numel = 2 * tokens_per_page * layer_num * head_num * head_dim
    assert numel * dtype.itemsize == bytes_per_page

    num_pages = 10
    tensors = {}
    for i in range(num_pages):
        k = f"key_{i}"
        v = torch.randn((numel,)).to(dtype=dtype)
        ok = hicache_hf3fs.set(k, v)
        if i < (file_size // bytes_per_page):
            assert ok, f"Failed to insert {k}"
        else:
            assert not ok
        tensors[k] = v
    assert hicache_hf3fs.get("key_8") is None
    assert hicache_hf3fs.get("key_9") is None

    start = 0
    for i in range(start, start + hicache_hf3fs.num_pages):
        k = f"key_{i}"
        assert hicache_hf3fs.exists(k)
        out = hicache_hf3fs.get(k)
        assert out is not None
        v = tensors[k]
        assert torch.allclose(v, out, atol=1e-3), f"Tensor mismatch for {k}"

    assert not hicache_hf3fs.exists("not_exists")

    hicache_hf3fs.delete("key_7")
    v2 = torch.randn((numel,)).to(dtype=dtype)
    assert hicache_hf3fs.set("key_new", v2)
    assert torch.allclose(hicache_hf3fs.get("key_new"), v2, atol=1e-3)

    hicache_hf3fs.clear()
    assert (
        len(hicache_hf3fs.metadata_client.rank_metadata.free_pages)
        == hicache_hf3fs.metadata_client.rank_metadata.num_pages
    )

    # batch
    num_pages = 10
    tensors = {}
    keys = []
    values = []
    for i in range(num_pages):
        k = f"key_{i}"
        keys.append(k)
        v = torch.randn((numel,)).to(dtype=dtype)
        values.append(v)

    ok = hicache_hf3fs.batch_set(keys, values)
    assert not ok
    assert hicache_hf3fs.get("key_8") is None
    assert hicache_hf3fs.get("key_9") is None

    results = hicache_hf3fs.batch_get(keys[: hicache_hf3fs.num_pages])
    for result, key, value in zip(
        results, keys[: hicache_hf3fs.num_pages], values[: hicache_hf3fs.num_pages]
    ):
        assert torch.allclose(value, result, atol=1e-3), f"Tensor mismatch for {key}"

    hicache_hf3fs.close()
    os.remove(hicache_hf3fs.file_path)

    print("All test cases passed.")
```
**EN:** `test` is a function that sends requests to serving or OpenAI-compatible APIs, prepares tensors and invokes GPU kernels, and loads, filters, or serializes benchmark datasets. Notable calls include `os.getenv`, `HiCacheHF3FS.from_env_config`, `range`.
**CN:** `test` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集。其中较关键的调用包括 `os.getenv`, `HiCacheHF3FS.from_env_config`, `range`。

### Lines 129-192: Function `bench` / 函数 `bench`
```python
def bench():
    # Qwen3-32B
    layer_num = 64
    head_num, head_dim = 8, 128
    kv_lora_rank, qk_rope_head_dim = 0, 0
    store_dtype = torch.bfloat16
    tokens_per_page = 64

    file_path = "/data/test.bin"
    file_size = 1 << 40
    numjobs = 16
    bytes_per_page = 16 << 20
    entries = 8
    dtype = store_dtype
    hicache_hf3fs = HiCacheHF3FS(
        rank=0,
        file_path=file_path,
        file_size=file_size,
        numjobs=numjobs,
        bytes_per_page=bytes_per_page,
        entries=entries,
        dtype=dtype,
        metadata_client=Hf3fsLocalMetadataClient(),
    )

    numel = 2 * tokens_per_page * layer_num * head_num * head_dim
    assert numel * dtype.itemsize == bytes_per_page

    num_page = 128
    values = [torch.randn((numel,)).to(dtype=dtype) for _ in tqdm(range(num_page))]

    warmup = 50
    iteration = 100

    w_bw = []
    w_size = num_page * bytes_per_page / (1 << 30)
    for i in tqdm(range(warmup + iteration), desc="Benchmarking write (GB/s)"):
        keys = [f"{j}" for j in range(i * num_page, (i + 1) * num_page)]
        tik = time.perf_counter()
        ok = hicache_hf3fs.batch_set(keys, values)
        tok = time.perf_counter()
        if i < warmup:
            continue
        w_bw.append(w_size / (tok - tik))
        assert ok
    print_stats(w_bw)

    r_bw = []
    r_size = num_page * bytes_per_page / (1 << 30)
    for i in tqdm(range(warmup + iteration), desc="Benchmarking read (GB/s)"):
        keys = random.sample(
            list(hicache_hf3fs.metadata_client.rank_metadata.key_to_index.keys()),
            num_page,
        )
        tik = time.perf_counter()
        results = hicache_hf3fs.batch_get(keys)
        tok = time.perf_counter()
        if i < warmup:
            continue
        r_bw.append(r_size / (tok - tik))
        assert all([r is not None for r in results])
    print_stats(r_bw)

    hicache_hf3fs.close()
```
**EN:** `bench` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. Notable calls include `HiCacheHF3FS`, `tqdm`, `print_stats`.
**CN:** `bench` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。其中较关键的调用包括 `HiCacheHF3FS`, `tqdm`, `print_stats`。

### Lines 195-247: Function `allclose` / 函数 `allclose`
```python
def allclose():
    # Qwen3-32B
    layer_num = 64
    head_num, head_dim = 8, 128
    kv_lora_rank, qk_rope_head_dim = 0, 0
    store_dtype = torch.bfloat16
    tokens_per_page = 64

    file_path = "/data/test.bin"
    file_size = 1 << 40
    numjobs = 16
    bytes_per_page = 16 << 20
    entries = 8
    dtype = store_dtype
    hicache_hf3fs = HiCacheHF3FS(
        rank=0,
        file_path=file_path,
        file_size=file_size,
        numjobs=numjobs,
        bytes_per_page=bytes_per_page,
        entries=entries,
        dtype=dtype,
        metadata_client=Hf3fsLocalMetadataClient(),
    )

    numel = 2 * tokens_per_page * layer_num * head_num * head_dim
    assert numel * dtype.itemsize == bytes_per_page

    num_page = 128
    values = [torch.randn((numel,)).to(dtype=dtype) for _ in tqdm(range(num_page))]

    iteration = 100

    for i in tqdm(range(iteration), desc="Benchmarking write (GB/s)"):
        keys = [f"{j}" for j in range(i * num_page, (i + 1) * num_page)]
        ok = hicache_hf3fs.batch_set(keys, values)
        assert ok

    read_keys, read_results = [], []
    for i in tqdm(range(iteration), desc="Benchmarking read (GB/s)"):
        keys = random.sample(
            list(hicache_hf3fs.metadata_client.rank_metadata.key_to_index.keys()),
            num_page,
        )
        results = hicache_hf3fs.batch_get(keys)
        read_keys.extend(keys)
        read_results.extend(results)
        assert all([r is not None for r in results])

    for key, result in tqdm(zip(read_keys, read_results)):
        assert torch.allclose(values[int(key) % num_page], result, atol=1e-3)

    hicache_hf3fs.close()
```
**EN:** `allclose` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. Notable calls include `HiCacheHF3FS`, `tqdm`, `hicache_hf3fs.close`.
**CN:** `allclose` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。其中较关键的调用包括 `HiCacheHF3FS`, `tqdm`, `hicache_hf3fs.close`。

### Lines 250-254: Function `main` / 函数 `main`
```python
def main():
    logging.basicConfig(level=logging.INFO)
    test()
    bench()
    allclose()
```
**EN:** `main` is a function that implements the core logic for this scope. Notable calls include `logging.basicConfig`, `test`, `bench`.
**CN:** `main` 是一个函数，用于实现当前作用域的核心逻辑。其中较关键的调用包括 `logging.basicConfig`, `test`, `bench`。

### Lines 257-258: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    main()
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `logging`, `os`, `random`, `time`, `typing`
- **Third-party / 第三方依赖**: `torch`, `tqdm`
- **Internal / 项目内部依赖**: `sglang.srt.mem_cache.storage.hf3fs.mini_3fs_metadata_server`, `sglang.srt.mem_cache.storage.hf3fs.storage_hf3fs`
