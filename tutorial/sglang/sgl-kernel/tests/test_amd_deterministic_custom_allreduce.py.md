# test_amd_deterministic_custom_allreduce.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_amd_deterministic_custom_allreduce.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `AMD deterministic custom allreduce` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `AMD deterministic custom allreduce` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Module documentation
````python
"""
Test deterministic custom all-reduce kernel behavior with batch size invariance.

This test uses the 1-stage all-reduce kernel which is inherently deterministic
due to fixed accumulation ordering (each GPU reads all data from all GPUs and
reduces locally in a fixed order - no atomics, no race conditions).

Note: This is NOT a reduce-scatter + all-gather (RS+AG) approach.

This test compares:
1. Deterministic kernel (same batch size)
2. Deterministic kernel (different batch size)

Usage:
    pytest test_amd_deterministic_custom_allreduce.py
"""
````
**EN:** This section documents the module-level intent or provenance before executable logic begins.
**CN:** 该部分在可执行逻辑开始前说明模块级意图、来源或背景。

### Lines 18-25: Imports and module setup
````python
import multiprocessing as mp
import socket

import pytest
import torch
import torch.distributed as dist

from sglang.srt.environ import envs
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 28-31: `get_open_port` definition
````python
def get_open_port():
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.bind(("127.0.0.1", 0))
        return s.getsockname()[1]
````
**EN:** This section defines `get_open_port`. It retrieves or computes the `open port` path used by the module.
**CN:** 该部分定义 `get_open_port`。它负责获取或计算模块中与 `open port` 相关的处理路径。

### Lines 34-205: `worker` definition
````python
def worker(world_size, rank, port):
    envs.SGLANG_USE_1STAGE_ALLREDUCE.set("1")
    device = torch.device(f"cuda:{rank}")
    torch.cuda.set_device(device)

    dist.init_process_group(
        backend="nccl",
        init_method=f"tcp://localhost:{port}",
        rank=rank,
        world_size=world_size,
    )

    # Try to import and use deterministic kernel
    try:
        from torch.distributed import new_group

        from sglang.srt.distributed.device_communicators.custom_all_reduce import (
            CustomAllreduce,
        )

        # Create gloo group for custom AR
        dist.barrier()
        ar_group = new_group(backend="gloo")
        dist.barrier()

        custom_ar = CustomAllreduce(group=ar_group, device=device)

        if custom_ar is None or custom_ar.disabled:
            if rank == 0:
                print("✗ Custom AR not available or disabled")
            dist.destroy_process_group()
            return
    except Exception as e:
        if rank == 0:
            print(f"✗ Failed to initialize deterministic kernel: {e}")
            import traceback

            traceback.print_exc()
        dist.destroy_process_group()
        return

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

    # Check if inputs fit in buffer
    # Buffer size is max_size bytes, input size is numel * element_size bytes
    input_size_bytes = base_input.numel() * base_input.element_size()
    if input_size_bytes > custom_ar.max_size and rank == 0:
        print(
            f"Warning: Input size ({input_size_bytes/(1024*1024):.1f} MB) exceeds buffer size ({custom_ar.max_size/(1024*1024):.1f} MB)"
        )
        print("  Using unregistered mode (will copy to buffer)")

    dist.barrier()

    # =========================================================================
    # TEST 1: Deterministic kernel (same batch size) - should be DETERMINISTIC
    # =========================================================================
    if rank == 0:
        print(f"\n{'='*70}")
        print("TEST 1: Deterministic kernel (same batch size)")
        print(f"{'='*70}")
    dist.barrier()

    results_allreduce_only = []
    for trial in range(num_trials):
        # Clone the same input
        inp = base_input.clone()

        result = custom_ar.custom_all_reduce(inp)
        torch.cuda.synchronize()

        # Store checksum
        checksum = result.view(-1).sum().item()
        first_vals = result.view(-1)[:5].clone()
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
            print("  ✓ DETERMINISTIC KERNEL (fixed BS): DETERMINISTIC (as expected)")
        else:
            print(
                "  ✗ DETERMINISTIC KERNEL (fixed BS): NON-DETERMINISTIC (unexpected!)"
            )

    dist.barrier()

    # =========================================================================
    # TEST 2: Deterministic kernel (different batch size) - should be DETERMINISTIC
    # [a], [a, x], [a, x, x], ...
    # =========================================================================
    if rank == 0:
        print(f"\n{'='*70}")
        print("TEST 2: Deterministic kernel (different batch size)")
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

            result_flat = custom_ar.custom_all_reduce(batch_flat)
            torch.cuda.synchronize()

            # Reshape back to (bs, hidden_dim)
            batch_out = result_flat.view(bs, hidden_dim)

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
            print("  ✓ DETERMINISTIC KERNEL (variant BS): DETERMINISTIC")
        else:
            print("  ✗ DETERMINISTIC KERNEL (variant BS): NON-DETERMINISTIC")

    dist.barrier()

    dist.destroy_process_group()
````
**EN:** This section defines `worker` and implements the core logic associated with worker.
**CN:** 该部分定义 `worker`，并实现与 worker 相关的核心逻辑。

### Lines 208-238: `main` definition
````python
def main():
    world_size = 8
    available_gpus = torch.cuda.device_count()

    print("=" * 70)
    print("Deterministic Kernel All-Reduce Determinism Test")
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

### Lines 241-247: `test_deterministic_custom_allreduce` definition
````python
@pytest.mark.skipif(
    not torch.cuda.is_available() or torch.cuda.device_count() < 2,
    reason="Requires at least 2 CUDA GPUs",
)
def test_deterministic_custom_allreduce():
    """Test that deterministic custom all-reduce produces consistent results."""
    main()
````
**EN:** This section defines the test `test_deterministic_custom_allreduce`. It sets up inputs, runs the target path, and checks the expected result. Docstring summary: Test that deterministic custom all-reduce produces consistent results.
**CN:** 该部分定义测试 `test_deterministic_custom_allreduce`。它会准备输入、执行目标路径，并检查预期结果。 文档字符串摘要：Test that deterministic custom all-reduce produces consistent results.

### Lines 250-251: Command-line entry point
````python
if __name__ == "__main__":
    main()
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `get_open_port`, `worker`, `main`, `test_deterministic_custom_allreduce`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.srt.distributed.device_communicators.custom_all_reduce`, `sglang.srt.environ`
- **External / 外部**: `multiprocessing`, `pytest`, `socket`, `torch`, `torch.distributed`, `traceback`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
