# test_rccl_multi_gpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/amd/test_rccl_multi_gpu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `test_rccl_multi_gpu` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `test_rccl_multi_gpu` 流程，主要负责CI 编排、测试执行。它属于 `amd` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-5: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Simple RCCL test for multi-GPU communication.
This test verifies that RCCL can initialize and communicate across multiple GPUs.
"""
```
**EN:** Simple RCCL test for multi-GPU communication.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 7-11: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import os
import sys

import torch
import torch.distributed as dist
```
**EN:** This block loads os, sys, torch, torch.distributed. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 os, sys, torch, torch.distributed。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 14-57: Defines the `test_rccl_allreduce` routine / 定义 `test_rccl_allreduce` 例程
```python
def test_rccl_allreduce():
    """Test basic RCCL allreduce operation across all GPUs."""
    if not torch.cuda.is_available():
        print("CUDA not available, skipping test")
        sys.exit(1)

    # Initialize process group with NCCL (RCCL on AMD)
    dist.init_process_group(backend="nccl")

    rank = dist.get_rank()
    world_size = dist.get_world_size()

    print(f"[Rank {rank}/{world_size}] Initialized successfully")

    # Set device
    device = torch.device(f"cuda:{rank}")
    torch.cuda.set_device(device)

    print(f"[Rank {rank}] Device: {torch.cuda.get_device_name(device)}")
    print(
        f"[Rank {rank}] Device memory: {torch.cuda.get_device_properties(device).total_memory / 1e9:.2f} GB"
    )

    # Create a tensor and perform allreduce
    tensor = torch.ones(1000, device=device) * rank
    print(f"[Rank {rank}] Before allreduce: tensor sum = {tensor.sum().item()}")

    dist.all_reduce(tensor, op=dist.ReduceOp.SUM)

    expected_sum = sum(range(world_size)) * 1000
    actual_sum = tensor.sum().item()

    print(
        f"[Rank {rank}] After allreduce: tensor sum = {actual_sum}, expected = {expected_sum}"
    )

    if abs(actual_sum - expected_sum) < 0.1:
        print(f"[Rank {rank}] ✓ RCCL allreduce test PASSED")
        dist.destroy_process_group()
        sys.exit(0)
    else:
        print(f"[Rank {rank}] ✗ RCCL allreduce test FAILED")
        dist.destroy_process_group()
        sys.exit(1)
```
**EN:** This block defines `test_rccl_allreduce`. Test basic RCCL allreduce operation across all GPUs. It takes no explicit parameters at the top level. Internally it emits status messages, controls process exit status.
**CN:** 该代码块定义了 `test_rccl_allreduce`。其文档字符串说明了该逻辑的预期职责。它在顶层定义中不接收显式参数。内部会输出状态信息、控制进程退出状态。

### Lines 60-61: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    test_rccl_allreduce()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **Environment management** / 环境管理
- **Process control** / 进程控制

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `sys`
- **Third-party modules / 第三方模块**: `torch`, `torch.distributed`
