# bench_zerocopy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/hf3fs/bench_zerocopy.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on hf3fs zerocopy. It primarily measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs. / 该 Python 模块聚焦于 hf3fs zerocopy 相关流程。它主要用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。

## Line-by-Line Analysis / 逐行分析
### Lines 1-140: Imports and setup / 导入与初始化
```python
import threading
import time

import torch
from tqdm import tqdm

from sglang.srt.distributed import (
    get_world_group,
    init_distributed_environment,
    initialize_model_parallel,
)
from sglang.srt.managers.cache_controller import (
    HiCacheController,
    PrefetchOperation,
    StorageOperation,
)
from sglang.srt.mem_cache.allocator import TokenToKVPoolAllocator
from sglang.srt.mem_cache.memory_pool import MHATokenToKVPool
from sglang.srt.mem_cache.memory_pool_host import MHATokenToKVPoolHost

init_distributed_environment(
    world_size=1,
    rank=0,
    distributed_init_method="tcp://127.0.0.1:23456",
    local_rank=0,
    backend="gloo",
)

initialize_model_parallel(
    tensor_model_parallel_size=1,
    pipeline_model_parallel_size=1,
)

group = get_world_group().cpu_group

max_total_num_tokens = 524288
page_size = 64
kv_cache_dtype = torch.bfloat16
layer_num = 64
head_num, head_dim = 8, 128
device = "cuda"
hicache_ratio = 2
hicache_size = 0
hicache_mem_layout = "page_first"
# hicache_mem_layout = "layer_first"
hicache_write_policy = "write_through"
hicache_io_backend = "kernel"
hicache_storage_backend = "hf3fs"
prefetch_threshold = 256

op_size = 1024
op_num = 16

token_to_kv_pool = MHATokenToKVPool(
    max_total_num_tokens,
    page_size=page_size,
    dtype=kv_cache_dtype,
    head_num=head_num,
    head_dim=head_dim,
    layer_num=layer_num,
    device=device,
    enable_memory_saver=True,
)

token_to_kv_pool_allocator = TokenToKVPoolAllocator(
    max_total_num_tokens,
    dtype=kv_cache_dtype,
    device=device,
    kvcache=token_to_kv_pool,
    need_sort=False,
)

kv_cache = token_to_kv_pool_allocator.get_kvcache()
token_to_kv_pool_host = MHATokenToKVPoolHost(
    kv_cache,
    hicache_ratio,
    hicache_size,
    page_size,
    hicache_mem_layout,
)

load_cache_event = threading.Event()
cache_controller = HiCacheController(
    token_to_kv_pool_allocator,
    token_to_kv_pool_host,
    page_size,
    group,
    load_cache_event=load_cache_event,
    write_policy=hicache_write_policy,
    io_backend=hicache_io_backend,
    storage_backend=hicache_storage_backend,
    prefetch_threshold=prefetch_threshold,
)

operations = [
    StorageOperation(
        torch.tensor(list(range(i, i + op_size))),
        list(range(i, i + op_size)),
        hash_value=[f"{j}" for j in range(i, i + op_size, page_size)],
    )
    for i in tqdm(range(0, op_num * op_size, op_size))
]

tik = time.monotonic()
if hicache_mem_layout == "page_first":
    for operation in operations:
        cache_controller.zerocopy_page_backup(operation, batch_size=128)
elif hicache_mem_layout == "layer_first":
    for operation in operations:
        cache_controller.generic_page_backup(operation, batch_size=128)
tok = time.monotonic()
print(f"{tok-tik:.6f} s")

operations = [
    PrefetchOperation(
        f"{i}",
        torch.tensor(list(range(i, i + op_size))),
        list(range(i, i + op_size)),
        f"{i}",
    )
    for i in tqdm(range(0, op_num * op_size, op_size))
]

for operation in operations:
    operation.hash_value = [
        f"{j}"
        for j in range(
            int(operation.last_hash), int(operation.last_hash) + op_size, page_size
        )
    ]

tik = time.monotonic()
if hicache_mem_layout == "page_first":
    for operation in operations:
        cache_controller.zerocopy_page_transfer(operation, batch_size=128)
elif hicache_mem_layout == "layer_first":
    for operation in operations:
        cache_controller.generic_page_transfer(operation, batch_size=128)
tok = time.monotonic()
print(f"{tok-tik:.6f} s")
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and processes tokenized prompts or decoded outputs.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `threading`, `time`
- **Third-party / 第三方依赖**: `torch`, `tqdm`
- **Internal / 项目内部依赖**: `sglang.srt.distributed`, `sglang.srt.managers.cache_controller`, `sglang.srt.mem_cache.allocator`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.memory_pool_host`
