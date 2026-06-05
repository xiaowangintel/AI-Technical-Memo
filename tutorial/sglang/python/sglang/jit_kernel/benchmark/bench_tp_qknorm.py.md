# bench_tp_qknorm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/benchmark/bench_tp_qknorm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: Imports and module setup
```python
from __future__ import annotations

import argparse
import os

import torch
import torch.distributed as dist

import sglang.srt.distributed.parallel_state as ps
from sglang.jit_kernel.all_reduce import (
    fused_parallel_qknorm,
    get_fused_parallel_qknorm_max_occupancy,
)
from sglang.jit_kernel.utils import get_ci_test_range
from sglang.srt.distributed.device_communicators.custom_all_reduce_v2 import (
    CustomAllReduceV2,
)
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(
    est_time=120,
    suite="base-b-kernel-benchmark-1-gpu-large",
    disabled="requires multi-GPU, self-skips in CI",
)

Q_K_DIMS = [(6144, 1024)]
DTYPE = torch.bfloat16
EPS = 1e-6
BATCH_SIZES = get_ci_test_range([2**i for i in range(15)], [1, 64, 1024])
NUM_LAYERS = 8
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 33-39: Function `parse_args`
```python
def parse_args():
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument("--warmup", type=int, default=10)
    parser.add_argument("--iters", type=int, default=100)
    return parser.parse_args()
```
**EN:** This block defines `parse_args`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `parse_args`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 40-75: Function `init_distributed`
```python
def init_distributed():
    local_rank = int(os.environ["LOCAL_RANK"])
    world_size = int(os.environ["WORLD_SIZE"])
    rank = local_rank
    device = torch.device(f"cuda:{rank}")
    torch.cuda.set_device(device)

    dist.init_process_group(backend="gloo")
    ps._WORLD = coord = ps.init_world_group(
        ranks=list(range(world_size)),
        local_rank=local_rank,
        backend="nccl",
    )

    cpu_group = coord.cpu_group
    max_occupancy = get_fused_parallel_qknorm_max_occupancy(
        DTYPE, world_size, Q_K_DIMS[0][0], Q_K_DIMS[0][1]
    )
    if rank == 0:
        print(f"Max occupancy for fused_parallel_qknorm: {max_occupancy} blocks/SM")

    props = torch.cuda.get_device_properties(device)
    comm = CustomAllReduceV2(
        cpu_group,
        device,
        max_pull_size=0,
        max_push_size=8 * max(BATCH_SIZES),
        max_push_blocks=props.multi_processor_count * max_occupancy,
    )
    comm_ = CustomAllReduceV2(cpu_group, device)
    if comm.disabled or comm_.disabled:
        raise RuntimeError("JIT CustomAllReduceV2 is disabled on this system")
    return rank, world_size, device, cpu_group, comm, comm_


@torch.inference_mode()
```
**EN:** This block defines `init_distributed`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `init_distributed`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 76-97: Function `bench_one`
```python
def bench_one(fn, warmup: int, iters: int) -> float:
    for _ in range(warmup):
        fn(0)
    torch.cuda.synchronize()

    graph = torch.cuda.CUDAGraph()
    with torch.cuda.graph(graph):
        for i in range(NUM_LAYERS):
            fn(i)

    graph.replay()
    start = torch.cuda.Event(enable_timing=True)
    end = torch.cuda.Event(enable_timing=True)
    graph.replay()
    start.record()
    for i in range(iters):
        graph.replay()
    end.record()
    torch.cuda.synchronize()
    return start.elapsed_time(end) * 1000.0 / (iters * NUM_LAYERS)
```
**EN:** This block defines `bench_one`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `bench_one`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 98-112: Imports and module setup
```python
def rmsnorm_baseline(
    comm_,
    q: torch.Tensor,
    k: torch.Tensor,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
    world_size: int,
) -> None:
    from sglang.srt.models.minimax_m2 import rms_apply_serial, rms_sumsq_serial

    sum_sq = rms_sumsq_serial(q, k)
    sum_sq = comm_.custom_all_reduce(sum_sq)
    rms_apply_serial(q, k, q_weight, k_weight, sum_sq, world_size, EPS)
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 113-170: Function `main`
```python
def main():
    args = parse_args()
    rank, world_size, device, _, comm, comm_ = init_distributed()
    torch.cuda.set_stream(torch.cuda.Stream())

    if rank == 0:
        print(
            f"{'q_dim':>8} {'k_dim':>8} {'batch':>8} {'fused_us':>12} {'baseline_us':>12}"
        )

    for q_dim, k_dim in Q_K_DIMS:
        local_q_dim = q_dim // world_size
        local_k_dim = k_dim // world_size
        for batch_size in BATCH_SIZES:
            q = torch.randn(
                NUM_LAYERS, batch_size, local_q_dim, device=device, dtype=DTYPE
            )
            k = torch.randn(
                NUM_LAYERS, batch_size, local_k_dim, device=device, dtype=DTYPE
            )
            q_weight = torch.randn(NUM_LAYERS, local_q_dim, device=device, dtype=DTYPE)
            k_weight = torch.randn(NUM_LAYERS, local_k_dim, device=device, dtype=DTYPE)

            def run_fused(i: int):
                fused_parallel_qknorm(
                    comm.obj,
                    q[i],
                    k[i],
                    q_weight[i],
                    k_weight[i],
                    EPS,
                )

            def run_baseline(i: int):
                rmsnorm_baseline(
                    comm_,
                    q[i],
                    k[i],
                    q_weight[i],
                    k_weight[i],
                    world_size,
                )

            fused_us = bench_one(run_fused, args.warmup, args.iters)
            baseline_us = bench_one(run_baseline, args.warmup, args.iters)

            if rank == 0:
                print(
                    f"{q_dim:8d} {k_dim:8d} {batch_size:8d} "
                    f"{fused_us:12.1f} {baseline_us:12.1f}"
                )

    comm.close()
    dist.destroy_process_group()


if __name__ == "__main__":
    main()
```
**EN:** This block defines `main`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `main`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `argparse`
- `os`
- `torch`
- `torch.distributed as dist`
- `sglang.srt.distributed.parallel_state as ps`
- `sglang.jit_kernel.all_reduce -> (`
- `sglang.jit_kernel.utils -> get_ci_test_range`
- `sglang.srt.distributed.device_communicators.custom_all_reduce_v2 -> (`
- `sglang.test.ci.ci_register -> register_cuda_ci`
- `sglang.srt.models.minimax_m2 -> rms_apply_serial`
