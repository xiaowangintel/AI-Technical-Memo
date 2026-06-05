# test_custom_all_reduce.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_custom_all_reduce.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: """"". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“"""”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-80: Imports and module setup
```python
"""
Correctness test for the JIT custom all-reduce (v2) kernel.

The test compares the JIT custom all-reduce output against NCCL all-reduce
for various tensor sizes and dtypes, in both eager and CUDA-graph modes.

Usage:
    python -m pytest test_jit_custom_all_reduce.py -v

This file doubles as the torchrun worker script.  The test class launches
    torchrun --nproc_per_node=N <this_file>
and asserts that all worker processes exit successfully.
"""

from __future__ import annotations

import itertools
import logging
import multiprocessing as mp
import os
from typing import Dict, Optional, Tuple

import pytest
import torch
import torch.distributed as dist

import sglang.srt.distributed.parallel_state as ps
from sglang.jit_kernel.all_reduce import (
    AllReduceAlgo,
    _jit_custom_all_reduce_pull_module,
    _jit_custom_all_reduce_push_module,
)
from sglang.jit_kernel.tests.utils import multiprocess_main, multiprocess_test
from sglang.srt.distributed.device_communicators.custom_all_reduce_v2 import (
    CustomAllReduceV2,
)
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(
    est_time=300,
    suite="base-b-kernel-unit-8-gpu-h200",
)
register_cuda_ci(
    est_time=300,
    suite="nightly-kernel-8-gpu-h200",
    nightly=True,
)

# ---------------------------------------------------------------------------
# Test parameters (shared between test class and worker)
# ---------------------------------------------------------------------------

TEST_SIZES = [
    16,
    32,
    512,
    1024,
    1024 + 16,  # weird case
    4 * 1024,
    32 * 1024,
    256 * 1024,
    2 * 1024 * 1024,  # 2M elements
    4 * 1024 * 1024,  # 4M elements
]
TEST_DTYPES = [torch.float16, torch.bfloat16, torch.float32]
SHOTS = [
    AllReduceAlgo.ONE_SHOT_PULL,
    AllReduceAlgo.ONE_SHOT_PUSH,
    AllReduceAlgo.TWO_SHOT_PULL,
]
USE_GRAPH_OPTIONS = [True, False]
TEST_CONFIG = itertools.product(TEST_SIZES, TEST_DTYPES, SHOTS, USE_GRAPH_OPTIONS)
TEST_LAYERS = 4
TEST_LOOP = 16

# ---------------------------------------------------------------------------
# Test class (runs via pytest, launches torchrun subprocesses)
# ---------------------------------------------------------------------------
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 81-85: Function `_compile_one`
```python
def _compile_one(dtype: torch.dtype, world_size: int):
    _jit_custom_all_reduce_push_module(dtype, world_size)
    _jit_custom_all_reduce_pull_module(dtype, world_size)
```
**EN:** This block defines `_compile_one`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_compile_one`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 86-101: Function `_precompile_kernels`
```python
def _precompile_kernels() -> None:
    # NOTE: even when device count < 8, we should be able to compile all
    process_map: Dict[Tuple[torch.dtype, int], mp.Process] = {}
    COMPILE_SPACE = itertools.product(TEST_DTYPES, [2, 3, 4, 5, 6, 7, 8])
    mp.set_start_method("spawn")
    for config in COMPILE_SPACE:
        process_map[config] = mp.Process(target=_compile_one, args=config)
    for process in process_map.values():
        process.start()
    for (dtype, world_size), process in process_map.items():
        process.join()
        if process.exitcode != 0:
            raise RuntimeError(f"Custom All Reduce {world_size=} {dtype=} failed")


@pytest.mark.parametrize("nproc", [1, 2, 3, 4, 5, 6, 7, 8])
```
**EN:** This block defines `_precompile_kernels`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_precompile_kernels`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 102-118: Function `test_custom_allreduce`
```python
def test_custom_allreduce(nproc: int) -> None:
    if nproc == 1:  # NOTE: special case to speed up tests
        return _precompile_kernels()

    device_count = torch.cuda.device_count()
    if device_count < nproc:
        pytest.skip(
            f"Requires at least {nproc} GPUs, but only {device_count} available"
        )
    multiprocess_test(__file__, nproc)


# ---------------------------------------------------------------------------
# Worker logic (executed by each torchrun process)
# ---------------------------------------------------------------------------
```
**EN:** This block defines `test_custom_allreduce`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_custom_allreduce`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 119-149: Function `init_distributed`
```python
def init_distributed():
    """Initialize distributed groups via torchrun env vars.

    Returns (rank, device, cpu_group, nccl_group, comm).
    """
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
    nccl_group = coord.device_group
    assert nccl_group is not None

    max_size = max(TEST_SIZES) * 4
    comm = CustomAllReduceV2(cpu_group, device, max_size, max_size)
    if comm.disabled:
        raise RuntimeError("JIT CustomAllReduceV2 is disabled on this system")

    return rank, device, cpu_group, nccl_group, comm


@torch.inference_mode()
```
**EN:** This block defines `init_distributed`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `init_distributed`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 150-207: Function `worker_test`
```python
def worker_test(
    device: torch.device,
    nccl_group: dist.ProcessGroup,
    comm: CustomAllReduceV2,
    size: int,
    dtype: torch.dtype,
    use_graph: bool,
    algo: AllReduceAlgo,
) -> Optional[RuntimeError]:
    comm.override_algo = algo

    def get_run_graph_fn():
        graph = torch.cuda.CUDAGraph()
        graph_inp = torch.zeros((TEST_LAYERS, size), dtype=dtype, device=device)
        out_jits = []
        with comm.capture():
            with torch.cuda.graph(graph):
                for i in range(TEST_LAYERS):
                    out_jits.append(comm.custom_all_reduce(graph_inp[i]))
                out_jit = torch.stack(out_jits)
        torch.cuda.synchronize()

        def run_graph(x: torch.Tensor) -> torch.Tensor:
            graph_inp.copy_(x)
            graph.replay()
            return out_jit.clone()

        return run_graph

    def get_run_eager_fn():
        def run_eager(x: torch.Tensor) -> torch.Tensor:
            eager_inp = x.clone()
            out_eagers = []
            for i in range(TEST_LAYERS):
                out_eagers.append(comm.custom_all_reduce(eager_inp[i]))
                torch.cuda.synchronize()
            return torch.stack(out_eagers)

        return run_eager

    run_fn = get_run_graph_fn() if use_graph else get_run_eager_fn()
    num_errors = 0
    for _ in range(TEST_LOOP):
        # NOTE: 15 * 8 < 128, which is the precision limit for bf16
        inp = torch.randint(0, 16, (TEST_LAYERS, size), dtype=dtype, device=device)
        assert comm.should_custom_ar(inp[0])
        out_ref = inp.clone()
        dist.all_reduce(out_ref, group=nccl_group)
        out_jit = run_fn(inp)
        num_errors += not torch.all(out_jit == out_ref)
    if num_errors > 0:
        return RuntimeError(
            f"Test failed for {size=}, {dtype=}, {algo=}, "
            f"{use_graph=} with {num_errors} errors. "
        )
    return None
```
**EN:** This block defines `worker_test`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `worker_test`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 208-239: Function `worker_main`
```python
def worker_main() -> None:
    """Entry point for each torchrun worker process."""
    rank, device, cpu_group, nccl_group, comm = init_distributed()

    torch.cuda.set_stream(torch.cuda.Stream())

    logging.disable(logging.INFO)  # Suppress internal logging for cleaner test output
    items = list(enumerate(TEST_CONFIG))
    for i, (size, dtype, algo, use_graph) in items:
        error = worker_test(device, nccl_group, comm, size, dtype, use_graph, algo)
        if error is not None:
            print(
                f"Worker {rank} failed for {size=}, {dtype=}, "
                f"{algo=}, {use_graph=}, iteration={i}\n"
                f"Error: {error}"
            )
        # communicate the result to rank 0 for logging
        result = torch.tensor([int(error is not None)])
        dist.all_reduce(result, group=cpu_group)
        failed = bool(result.item())
        if failed:
            raise RuntimeError(
                f"Test failed on rank {rank} for config: "
                f"{size=}, {dtype=}, {algo=}, {use_graph=}"
            )

    comm.close()
    dist.destroy_process_group()


if __name__ == "__main__":
    multiprocess_main(__file__, worker_main)
```
**EN:** This block defines `worker_main`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `worker_main`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `itertools`
- `logging`
- `multiprocessing as mp`
- `os`
- `typing -> Dict`
- `pytest`
- `torch`
- `torch.distributed as dist`
- `sglang.srt.distributed.parallel_state as ps`
- `sglang.jit_kernel.all_reduce -> (`
- `sglang.jit_kernel.tests.utils -> multiprocess_main`
- `sglang.srt.distributed.device_communicators.custom_all_reduce_v2 -> (`
- `sglang.test.ci.ci_register -> register_cuda_ci`
