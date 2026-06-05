# test_amd_nccl_allreduce_determinism.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_amd_nccl_allreduce_determinism.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `AMD NCCL allreduce determinism` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `AMD NCCL allreduce determinism` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Module documentation
````python
"""
Test to confirm non-determinism of default NCCL all-reduce with batch size invariance.

This test uses the default torch.distributed.all_reduce (NCCL) which can be
NON-DETERMINISTIC due to tree-based reduction algorithms that don't guarantee
fixed accumulation order for bfloat16/float16.

This test compares:
1. Default all-reduce (same batch size) - should be DETERMINISTIC
2. Default all-reduce (different batch size) - typically NON-DETERMINISTIC for bfloat16

Usage:
    pytest test_amd_nccl_allreduce_determinism.py
"""
````
**EN:** This section documents the module-level intent or provenance before executable logic begins.
**CN:** 该部分在可执行逻辑开始前说明模块级意图、来源或背景。

### Lines 16-21: Imports and module setup
````python
import multiprocessing as mp
import socket

import pytest
import torch
import torch.distributed as dist
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 24-27: `get_open_port` definition
````python
def get_open_port():
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.bind(("127.0.0.1", 0))
        return s.getsockname()[1]
````
**EN:** This section defines `get_open_port`. It retrieves or computes the `open port` path used by the module.
**CN:** 该部分定义 `get_open_port`。它负责获取或计算模块中与 `open port` 相关的处理路径。

### Lines 30-162: `worker` definition
````python
def worker(world_size, rank, port):
    device = torch.device(f"cuda:{rank}")
    torch.cuda.set_device(device)

    dist.init_process_group(
        backend="nccl",
        init_method=f"tcp://localhost:{port}",
        rank=rank,
        world_size=world_size,
    )

    num_trials = 10

    # Matrix sizes similar to real model layers
    # Format: (batch_size, hidden_dim) - typical tensor shape for all-reduce
    BS = 50  # max batch_size (1..BS)
    hidden_dim = 16384  # hidden dimension / intermediate dimension

    # Different seed per rank - each GPU has DIFFERENT input
    torch.manual_seed(42 + rank)

    # Create fixed inputs for all trials
    # Single request: (hidden_dim,)
    base_input = torch.randn(hidden_dim, dtype=torch.bfloat16, device=device)
    base_input_rand = torch.randn(hidden_dim, dtype=torch.bfloat16, device=device)

    dist.barrier()

    # =========================================================================
    # TEST 1: Default all-reduce (same batch size) - should be DETERMINISTIC
    # =========================================================================
    if rank == 0:
        print(f"\n{'='*70}")
        print("TEST 1: Default NCCL all_reduce (same batch size)")
        print(f"{'='*70}")
    dist.barrier()

    results_allreduce_only = []
    for trial in range(num_trials):
        # Clone the same input
        inp = base_input.clone()

        # Use default NCCL all-reduce
        dist.all_reduce(inp)
        torch.cuda.synchronize()

        # Store checksum
        checksum = inp.view(-1).sum().item()
        first_vals = inp.view(-1)[:5].clone()
        results_allreduce_only.append((checksum, first_vals))

        if rank == 0:
            print(
                f"  Trial {trial+1:2d}: sum={checksum:.6f}, first5={first_vals.tolist()}"
            )

    # Check determinism
    if rank == 0:
        ref_sum, ref_vals = results_allreduce_only[0]
        all_match = True
        for i, (s, vals) in enumerate(results_allreduce_only[1:], 1):
            if abs(ref_sum - s) > 1e-3 or not torch.allclose(ref_vals, vals, rtol=1e-3):
                all_match = False
                print(f"  Trial {i+1} DIFFERS! ref_sum={ref_sum:.6f}, got={s:.6f}")

        if all_match:
            print("  ✓ DEFAULT ALL_REDUCE (fixed BS): DETERMINISTIC (as expected)")
        else:
            print("  ✗ DEFAULT ALL_REDUCE (fixed BS): NON-DETERMINISTIC (unexpected!)")

    dist.barrier()

    # =========================================================================
    # TEST 2: Default all-reduce (different batch size) - typically NON-DETERMINISTIC
    # [a], [a, x], [a, x, x], ...
    # =========================================================================
    if rank == 0:
        print(f"\n{'='*70}")
        print("TEST 2: Default NCCL all_reduce (different batch size)")
        print("Batches: [a], [a,x], [a,x,x], ...")
        print(f"{'='*70}")
    dist.barrier()

    results_allreduce_only = {trial: [] for trial in range(num_trials)}
    for trial in range(num_trials):
        for bs in range(1, BS + 1):
            # Construct batch: (batch_size, hidden_dim)
            # First element is base_input, rest are base_input_rand
            batch = torch.stack([base_input] + [base_input_rand] * (bs - 1), dim=0)
            # Shape: (bs, hidden_dim)

            # Flatten for all-reduce: (bs * hidden_dim,)
            batch_flat = batch.view(-1)

            # Use default NCCL all-reduce
            dist.all_reduce(batch_flat)
            torch.cuda.synchronize()

            # Reshape back to (bs, hidden_dim)
            batch_out = batch_flat.view(bs, hidden_dim)

            # Only compare output corresponding to first request
            out_first_req = batch_out[0].clone()
            checksum = out_first_req.sum().item()
            first_vals = out_first_req[:5].clone()
            results_allreduce_only[trial].append((bs, checksum, first_vals))

            if rank == 0:
                print(
                    f"  Batch size {bs:2d}: sum={checksum:.6f}, first5={first_vals.tolist()}"
                )

    # Check determinism
    if rank == 0:
        for trial in range(num_trials):
            results = results_allreduce_only[trial]

            _, ref_sum, ref_vals = results[0]
            all_match = True
            for _, s, vals in results[1:]:
                if abs(ref_sum - s) > 1e-3 or not torch.allclose(
                    ref_vals, vals, rtol=1e-3
                ):
                    all_match = False

        if all_match:
            print("  ✓ DEFAULT ALL_REDUCE (variant BS): DETERMINISTIC")
        else:
            print("  ✗ DEFAULT ALL_REDUCE (variant BS): NON-DETERMINISTIC")

    dist.barrier()

    dist.destroy_process_group()
````
**EN:** This section defines `worker` and implements the core logic associated with worker.
**CN:** 该部分定义 `worker`，并实现与 worker 相关的核心逻辑。

### Lines 165-195: `main` definition
````python
def main():
    world_size = 8
    available_gpus = torch.cuda.device_count()

    print("=" * 70)
    print("Default NCCL All-Reduce Determinism Test")
    print("=" * 70)
    print(f"Available GPUs: {available_gpus}")
    print(f"Using world_size: {world_size}")

    if available_gpus < world_size:
        print(
            f"WARNING: Only {available_gpus} GPUs available, using {available_gpus} instead"
        )
        world_size = available_gpus

    if world_size < 2:
        print("ERROR: Need at least 2 GPUs for this test")
        return

    mp.set_start_method("spawn", force=True)
    port = get_open_port()

    procs = []
    for rank in range(world_size):
        p = mp.Process(target=worker, args=(world_size, rank, port))
        p.start()
        procs.append(p)

    for p in procs:
        p.join()
````
**EN:** This section defines `main` and implements the core logic associated with main.
**CN:** 该部分定义 `main`，并实现与 main 相关的核心逻辑。

### Lines 198-204: `test_nccl_allreduce_determinism` definition
````python
@pytest.mark.skipif(
    not torch.cuda.is_available() or torch.cuda.device_count() < 2,
    reason="Requires at least 2 CUDA GPUs",
)
def test_nccl_allreduce_determinism():
    """Test NCCL all-reduce determinism behavior with varying batch sizes."""
    main()
````
**EN:** This section defines the test `test_nccl_allreduce_determinism`. It sets up inputs, runs the target path, and checks the expected result. Docstring summary: Test NCCL all-reduce determinism behavior with varying batch sizes.
**CN:** 该部分定义测试 `test_nccl_allreduce_determinism`。它会准备输入、执行目标路径，并检查预期结果。 文档字符串摘要：Test NCCL all-reduce determinism behavior with varying batch sizes.

### Lines 207-208: Command-line entry point
````python
if __name__ == "__main__":
    main()
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `get_open_port`, `worker`, `main`, `test_nccl_allreduce_determinism`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: None obvious / 无明显内部依赖
- **External / 外部**: `multiprocessing`, `pytest`, `socket`, `torch`, `torch.distributed`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
