# benchmark_torch_symm_mem.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/all_reduce/benchmark_torch_symm_mem.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels all reduce torch symm. It primarily measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and loads, filters, or serializes benchmark datasets. / 该 Python 模块聚焦于 kernels all reduce torch symm 相关流程。它主要用于测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集。

## Line-by-Line Analysis / 逐行分析
### Lines 1-49: Imports and setup / 导入与初始化
```python
"""For Now, TORCH_SYMM_MEM is only supported on following limited tp case

SM90: {
    2: 64 * MiB,  # 64 MB
    4: 64 * MiB,  # 64 MB
    6: 128 * MiB,  # 128 MB
    8: 128 * MiB,  # 128 MB
},
SM100: {
    2: 64 * MiB,  # 64 MB
    4: 64 * MiB,  # 64 MB
    6: 128 * MiB,  # 128 MB
    8: 128 * MiB,  # 128 MB
}

export WORLD_SIZE=8
export RANK=0
export MASTER_ADDR=127.0.0.1
export MASTER_PORT=12345

torchrun --nproc_per_node gpu \
--nnodes $WORLD_SIZE \
--node_rank $RANK \
--master_addr $MASTER_ADDR \
--master_port $MASTER_PORT ./benchmark/kernels/all_reduce/benchmark_torch_symm_mem.py
"""

import os
from contextlib import nullcontext
from typing import List

import torch
import torch.distributed as dist
from torch.distributed import ProcessGroup

from sglang.srt.distributed import init_distributed_environment
from sglang.srt.distributed.device_communicators.pynccl import PyNcclCommunicator
from sglang.srt.distributed.device_communicators.torch_symm_mem import (
    TorchSymmMemCommunicator,
)
from sglang.srt.distributed.parallel_state import (
    get_tensor_model_parallel_group,
    graph_capture,
    initialize_model_parallel,
    set_torch_symm_mem_all_reduce,
)
from sglang.utils import is_in_ci

IS_CI = is_in_ci()
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it measures runtime latency, throughput, or other benchmark metrics and prepares tensors and invokes GPU kernels.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

### Lines 52-54: Function `torch_allreduce` / 函数 `torch_allreduce`
```python
def torch_allreduce(torch_input: torch.Tensor, group: ProcessGroup) -> torch.Tensor:
    dist.all_reduce(torch_input, group=group)
    return torch_input
```
**EN:** `torch_allreduce` is a function that prepares tensors and invokes GPU kernels. It returns `torch_input` to the caller. Notable calls include `dist.all_reduce`.
**CN:** `torch_allreduce` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `torch_input`。其中较关键的调用包括 `dist.all_reduce`。

### Lines 57-60: Function `torch_symm_mem_allreduce` / 函数 `torch_symm_mem_allreduce`
```python
def torch_symm_mem_allreduce(
    torch_symm_mem_input: torch.Tensor, torch_symm_mem_comm: TorchSymmMemCommunicator
) -> torch.Tensor:
    return torch_symm_mem_comm.all_reduce(torch_symm_mem_input)
```
**EN:** `torch_symm_mem_allreduce` is a function that prepares tensors and invokes GPU kernels. It returns `torch_symm_mem_comm.all_reduce(torch_symm_mem_input)` to the caller. Notable calls include `torch_symm_mem_comm.all_reduce`.
**CN:** `torch_symm_mem_allreduce` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `torch_symm_mem_comm.all_reduce(torch_symm_mem_input)`。其中较关键的调用包括 `torch_symm_mem_comm.all_reduce`。

### Lines 63-67: Function `pynccl_allreduce` / 函数 `pynccl_allreduce`
```python
def pynccl_allreduce(
    pynccl_input: torch.Tensor, pynccl_comm: PyNcclCommunicator
) -> torch.Tensor:
    pynccl_comm.all_reduce(pynccl_input)
    return pynccl_input
```
**EN:** `pynccl_allreduce` is a function that prepares tensors and invokes GPU kernels. It returns `pynccl_input` to the caller. Notable calls include `pynccl_comm.all_reduce`.
**CN:** `pynccl_allreduce` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `pynccl_input`。其中较关键的调用包括 `pynccl_comm.all_reduce`。

### Lines 70-99: Function `_bench_graph_time` / 函数 `_bench_graph_time`
```python
def _bench_graph_time(func, inp_randn, warmup_loop=2, graph_loop=10, test_loop=10):
    graph_input = inp_randn.clone()
    with graph_capture() as graph_capture_context:
        graph = torch.cuda.CUDAGraph()
        with torch.cuda.graph(graph, stream=graph_capture_context.stream):
            for _ in range(graph_loop):
                graph_out = func(graph_input)

    graph.replay()
    func_output = graph_out.clone()

    for _ in range(warmup_loop):
        graph.replay()
    torch.cuda.synchronize()

    start_event = torch.cuda.Event(enable_timing=True)
    end_event = torch.cuda.Event(enable_timing=True)

    latencies: List[float] = []
    for _ in range(test_loop):
        torch.cuda.synchronize()
        dist.barrier()
        start_event.record()
        graph.replay()
        end_event.record()
        end_event.synchronize()
        latencies.append(start_event.elapsed_time(end_event))
    func_cost_us = sum(latencies) / len(latencies) / graph_loop * 1000
    graph.reset()
    return func_output, func_cost_us
```
**EN:** `_bench_graph_time` is a function that prepares tensors and invokes GPU kernels. It returns `(func_output, func_cost_us)` to the caller. Notable calls include `inp_randn.clone`, `graph.replay`, `graph_out.clone`.
**CN:** `_bench_graph_time` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `(func_output, func_cost_us)`。其中较关键的调用包括 `inp_randn.clone`, `graph.replay`, `graph_out.clone`。

### Lines 102-121: Function `_bench_eager_time` / 函数 `_bench_eager_time`
```python
def _bench_eager_time(func, inp_randn, warmup_loop=2, test_loop=10):
    eager_input = inp_randn.clone()
    eager_output = func(eager_input)
    func_output = eager_output.clone()

    for _ in range(warmup_loop):
        func(eager_input)
    torch.cuda.synchronize()

    start_event = torch.cuda.Event(enable_timing=True)
    end_event = torch.cuda.Event(enable_timing=True)
    torch.cuda.synchronize()
    start_event.record()
    for _ in range(test_loop):
        func(eager_input)
    end_event.record()
    torch.cuda.synchronize()
    func_cost_us = start_event.elapsed_time(end_event) / test_loop * 1000

    return func_output, func_cost_us
```
**EN:** `_bench_eager_time` is a function that prepares tensors and invokes GPU kernels. It returns `(func_output, func_cost_us)` to the caller. Notable calls include `inp_randn.clone`, `func`, `eager_output.clone`.
**CN:** `_bench_eager_time` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `(func_output, func_cost_us)`。其中较关键的调用包括 `inp_randn.clone`, `func`, `eager_output.clone`。

### Lines 124-137: Function `get_torch_prof_ctx` / 函数 `get_torch_prof_ctx`
```python
def get_torch_prof_ctx(do_prof: bool):
    ctx = (
        torch.profiler.profile(
            activities=[
                torch.profiler.ProfilerActivity.CPU,
                torch.profiler.ProfilerActivity.CUDA,
            ],
            record_shapes=True,
            with_stack=True,
        )
        if do_prof
        else nullcontext()
    )
    return ctx
```
**EN:** `get_torch_prof_ctx` is a function that prepares tensors and invokes GPU kernels. It returns `ctx` to the caller. Notable calls include `torch.profiler.profile`, `nullcontext`.
**CN:** `get_torch_prof_ctx` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `ctx`。其中较关键的调用包括 `torch.profiler.profile`, `nullcontext`。

### Lines 140-145: Function `human_readable_size` / 函数 `human_readable_size`
```python
def human_readable_size(size, decimal_places=1):
    for unit in ["B", "KiB", "MiB", "GiB", "TiB", "PiB"]:
        if size < 1024.0 or unit == "PiB":
            break
        size /= 1024.0
    return f"{size:.{decimal_places}f} {unit}"
```
**EN:** `human_readable_size` is a function that implements the core logic for this scope. It returns `f'{size:.{decimal_places}f} {unit}'` to the caller.
**CN:** `human_readable_size` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `f'{size:.{decimal_places}f} {unit}'`。

### Lines 148-152: Imports and setup / 导入与初始化
```python
try:
    from tabulate import tabulate
except ImportError:
    print("tabulate not installed, skipping table printing")
    tabulate = None
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。

### Lines 155-167: Function `print_markdown_table` / 函数 `print_markdown_table`
```python
def print_markdown_table(data):
    if tabulate is not None:
        print(tabulate(data, headers="keys", tablefmt="github"))
        return
    headers = data[0].keys()
    header_row = "| " + " | ".join(headers) + " |"
    separator = "| " + " | ".join(["---"] * len(headers)) + " |"
    rows = []
    for item in data:
        row = "| " + " | ".join(str(item[key]) for key in headers) + " |"
        rows.append(row)
    markdown_table = "\n".join([header_row, separator] + rows)
    print(markdown_table)
```
**EN:** `print_markdown_table` is a function that implements the core logic for this scope. Notable calls include `data[0].keys`, `'\n'.join`, `print`.
**CN:** `print_markdown_table` 是一个函数，用于实现当前作用域的核心逻辑。其中较关键的调用包括 `data[0].keys`, `'\n'.join`, `print`。

### Lines 170-248: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    import logging

    logging.basicConfig(
        level=logging.INFO,
        format="%(asctime)s - %(levelname)s - %(message)s",
        datefmt="%Y-%m-%d %H:%M:%S",
        force=True,
    )
    if not dist.is_initialized():
        dist.init_process_group(backend="nccl")
    world, world_size = dist.group.WORLD, dist.get_world_size()
    rank = dist.get_rank()
    torch.cuda.set_device(rank % 8)
    device = torch.cuda.current_device()
    set_torch_symm_mem_all_reduce(True)
    init_distributed_environment(
        world_size=world_size,
        rank=rank,
        local_rank=rank % 8,
    )
    initialize_model_parallel(tensor_model_parallel_size=world_size)
    group = get_tensor_model_parallel_group().device_group
    cpu_group = get_tensor_model_parallel_group().cpu_group
    pynccl_comm = get_tensor_model_parallel_group().pynccl_comm
    torch_symm_mem_comm = get_tensor_model_parallel_group().torch_symm_mem_comm
    dist.barrier()
    profile = False
    dtype = torch.bfloat16
    ctx = get_torch_prof_ctx(profile)
    result = []

    with ctx:
        if IS_CI:
            i_range = range(10, 11)
        else:
            i_range = range(10, 20)
        for i in i_range:
            sz = 2**i
            if sz * dtype.itemsize > 2**24:
                break
            inp_randn = torch.randint(1, 16, (sz,), dtype=dtype, device=device)

            memory = torch.empty_like(inp_randn)
            memory_out = torch.empty_like(memory)
            torch_eager_output, torch_eager_time = _bench_eager_time(
                lambda inp: torch_allreduce(inp, group), inp_randn
            )
            symm_mem_eager_output, symm_mem_eager_time = _bench_eager_time(
                lambda inp: torch_symm_mem_allreduce(inp, torch_symm_mem_comm),
                inp_randn,
            )
            symm_mem_graph_output, symm_mem_graph_time = _bench_graph_time(
                lambda inp: torch_symm_mem_allreduce(inp, torch_symm_mem_comm),
                inp_randn,
            )
            # since pynccl is inplace op, this return result is not correct if graph loop > 1
            _, pynccl_graph_time = _bench_graph_time(
                lambda inp: pynccl_allreduce(inp, pynccl_comm), inp_randn
            )
            torch.testing.assert_close(torch_eager_output, symm_mem_graph_output)
            torch.testing.assert_close(torch_eager_output, symm_mem_eager_output)
            result.append(
                {
                    "msg_size": human_readable_size(inp_randn.nbytes),
                    "torch eager time": torch_eager_time,
                    "symm mem eager time": symm_mem_eager_time,
                    "symm mem graph time": symm_mem_graph_time,
                    "pynccl graph time": pynccl_graph_time,
                }
            )
            if rank == 0:
                print(f"sz={sz}, dtype={dtype}: correctness check PASS!")
    if rank == 0:
        print_markdown_table(result)
    if profile:
        prof_dir = f"prof/torch_symm_mem"
        os.makedirs(prof_dir, exist_ok=True)
        ctx.export_chrome_trace(f"{prof_dir}/trace_rank{dist.get_rank()}.json.gz")
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it prepares tensors and invokes GPU kernels and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `contextlib`, `typing`, `logging`
- **Third-party / 第三方依赖**: `torch`, `torch.distributed`, `tabulate`
- **Internal / 项目内部依赖**: `sglang.srt.distributed`, `sglang.srt.distributed.device_communicators.pynccl`, `sglang.srt.distributed.device_communicators.torch_symm_mem`, `sglang.srt.distributed.parallel_state`, `sglang.utils`
