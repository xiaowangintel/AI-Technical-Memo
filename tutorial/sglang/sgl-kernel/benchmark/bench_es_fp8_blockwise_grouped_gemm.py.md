# bench_es_fp8_blockwise_grouped_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_es_fp8_blockwise_grouped_gemm.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `expert specialization FP8 blockwise grouped GEMM` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `expert specialization FP8 blockwise grouped GEMM` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Imports and module setup
````python
import argparse
import random
from dataclasses import dataclass
from typing import List, Tuple

import numpy as np
import torch
from sgl_kernel import (
    es_fp8_blockwise_scaled_grouped_mm,
    fp8_blockwise_scaled_grouped_mm,
)
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 13-13: Top-level logic
````python
random.seed(28)
````
**EN:** This top-level block contains executable module logic that does not fit into a simpler category.
**CN:** 该顶层代码块包含不适合归入更简单类别的模块执行逻辑。

### Lines 16-17: `ceil_div` definition
````python
def ceil_div(x: int, y: int) -> int:
    return (x + y - 1) // y
````
**EN:** This section defines `ceil_div` and implements the core logic associated with ceil div.
**CN:** 该部分定义 `ceil_div`，并实现与 ceil div 相关的核心逻辑。

### Lines 20-28: `per_token_cast_to_fp8` definition
````python
def per_token_cast_to_fp8(x: torch.Tensor) -> Tuple[torch.Tensor, torch.Tensor]:
    assert x.dim() == 2
    m, n = x.shape
    pad_size = (128 - (n % 128)) % 128
    x = torch.nn.functional.pad(x, (0, pad_size), value=0) if pad_size > 0 else x
    x_view = x.view(m, -1, 128)
    x_amax = x_view.abs().float().amax(dim=2).view(m, -1).clamp(1e-4)
    fp8_data = (x_view * (448.0 / x_amax.unsqueeze(2))).to(torch.float8_e4m3fn)
    return fp8_data.view(m, n + pad_size)[:, :n], (x_amax / 448.0).view(m, -1)
````
**EN:** This section defines `per_token_cast_to_fp8` and implements the core logic associated with per token cast to FP8. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `per_token_cast_to_fp8`，并实现与 per token cast to FP8 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

### Lines 31-43: `per_block_cast_to_fp8` definition
````python
def per_block_cast_to_fp8(x: torch.Tensor) -> Tuple[torch.Tensor, torch.Tensor]:
    assert x.dim() == 2
    m, n = x.shape
    x_padded = torch.zeros(
        (ceil_div(m, 128) * 128, ceil_div(n, 128) * 128), dtype=x.dtype, device=x.device
    )
    x_padded[:m, :n] = x
    x_view = x_padded.view(-1, 128, x_padded.size(1) // 128, 128)
    x_amax = x_view.abs().float().amax(dim=(1, 3), keepdim=True).clamp(1e-4)
    x_scaled = (x_view * (448.0 / x_amax)).to(torch.float8_e4m3fn)
    return x_scaled.view_as(x_padded)[:m, :n].contiguous(), (x_amax / 448.0).view(
        x_view.size(0), x_view.size(2)
    )
````
**EN:** This section defines `per_block_cast_to_fp8` and implements the core logic associated with per block cast to FP8. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `per_block_cast_to_fp8`，并实现与 per block cast to FP8 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

### Lines 46-58: `create_unbalanced_expert_token_distribution` definition
````python
def create_unbalanced_expert_token_distribution(
    batch_size: int, topk: int, num_experts: int
):
    expert_ids = np.random.randint(0, num_experts, size=(batch_size * topk,)).tolist()
    expert_to_count = dict()
    for expert_id in range(num_experts):
        expert_to_count[expert_id] = 0
    for expert_id in expert_ids:
        expert_to_count[expert_id] += 1
    group_ms = []
    for expert_id in range(num_experts):
        group_ms.append(expert_to_count[expert_id])
    return group_ms
````
**EN:** This section defines `create_unbalanced_expert_token_distribution` and implements the core logic associated with create unbalanced expert token distribution.
**CN:** 该部分定义 `create_unbalanced_expert_token_distribution`，并实现与 create unbalanced expert token distribution 相关的核心逻辑。

### Lines 61-171: `bench_es` definition
````python
def bench_es(
    group_ms: List[int],
    n: int,
    k: int,
    num_groups: int,
    num_warmup: int,
    num_run: int,
) -> Tuple[float, int]:
    device = "cuda"
    alignment = 128
    n_g = ceil_div(n, alignment) * alignment
    k_g = ceil_div(k, alignment) * alignment
    out_dtype = torch.bfloat16

    expert_offsets = torch.zeros((num_groups + 1), device=device, dtype=torch.int32)
    problem_sizes = torch.zeros((num_groups, 3), device=device, dtype=torch.int32)

    a_tensors = []
    b_tensors = []
    a_scales_tensors = []
    b_scales_tensors = []
    if False:
        print("Token Distributtion: ", group_ms[0:num_groups])
        print("Token Count: ", sum(group_ms[0:num_groups]))
    for g in range(num_groups):
        m_g = group_ms[g]
        expert_offsets[g + 1] = expert_offsets[g] + m_g
        problem_sizes[g][:] = torch.tensor([m_g, n_g, k_g], device=device)
        if m_g != 0:
            a_g, a_scale = per_token_cast_to_fp8(torch.randn((m_g, k_g), device=device))
            a_tensors.append(a_g)
            a_scales_tensors.append(a_scale)

        b_g, b_scale = per_block_cast_to_fp8(torch.randn((n_g, k_g), device=device).t())
        b_tensors.append(b_g)
        b_scales_tensors.append(b_scale)

    a_stack = torch.empty(
        (expert_offsets[-1], k_g), device=device, dtype=torch.float8_e4m3fn
    )
    b_stack = torch.empty(
        (num_groups, n_g, k_g), device=device, dtype=torch.float8_e4m3fn
    )

    _aux_idx = 0
    for g in range(num_groups):
        if group_ms[g] != 0:
            a_stack[expert_offsets[g] : expert_offsets[g + 1]] = a_tensors[_aux_idx]
            _aux_idx += 1
        b_stack[g] = b_tensors[g].t()
    b_stack = b_stack.transpose(1, 2)

    a_scale_stack = torch.empty(
        (expert_offsets[-1], k_g // 128), device=device, dtype=torch.float32
    )
    b_scale_stack = torch.empty(
        (num_groups, n_g // 128, k_g // 128), device=device, dtype=torch.float32
    )

    _aux_idx = 0
    for g in range(num_groups):
        if group_ms[g] != 0:
            a_scale_stack[expert_offsets[g] : expert_offsets[g + 1]] = a_scales_tensors[
                _aux_idx
            ]
            _aux_idx += 1
        b_scale_stack[g] = b_scales_tensors[g].t()
    b_scale_stack = b_scale_stack.transpose(1, 2)

    workspace = torch.empty((1024 * 1024 * 1024), device=device, dtype=torch.uint8)
    c_out = torch.empty((expert_offsets[-1], n_g), device=device, dtype=out_dtype)
    a_strides = torch.full(
        (num_groups,), a_stack.stride(0), device=device, dtype=torch.int64
    )
    d_strides = torch.full(
        (num_groups,), c_out.stride(0), device=device, dtype=torch.int64
    )

    def run_cutlass():
        es_fp8_blockwise_scaled_grouped_mm(
            c_out,
            a_stack,
            b_stack,
            a_scale_stack,
            b_scale_stack,
            a_strides,
            a_strides,
            d_strides,
            problem_sizes,
            expert_offsets[:-1],
            workspace,
        )

    run_cutlass()
    # warmup
    for _ in range(num_warmup):
        run_cutlass()
    torch.cuda.synchronize()

    # run
    start_event = torch.cuda.Event(enable_timing=True)
    end_event = torch.cuda.Event(enable_timing=True)
    start_event.record()
    for _ in range(num_run):
        run_cutlass()
    end_event.record()
    end_event.synchronize()
    torch.cuda.synchronize()
    avg = start_event.elapsed_time(end_event) / num_run * 1000  # us

    return avg, expert_offsets[-1]
````
**EN:** This section defines `bench_es`. It benchmarks the `expert specialization` path used by the module.
**CN:** 该部分定义 `bench_es`。它负责基准测试模块中与 `expert specialization` 相关的处理路径。

### Lines 174-295: `bench_sgl` definition
````python
def bench_sgl(
    group_ms: List[int],
    n: int,
    k: int,
    num_groups: int,
    num_warmup: int,
    num_run: int,
) -> Tuple[float, int]:
    device = "cuda"
    alignment = 128
    n_g = ceil_div(n, alignment) * alignment
    k_g = ceil_div(k, alignment) * alignment
    out_dtype = torch.bfloat16

    expert_offsets = torch.zeros((num_groups + 1), device=device, dtype=torch.int32)
    problem_sizes = torch.zeros((num_groups, 3), device=device, dtype=torch.int32)
    layout_sfa = torch.zeros((num_groups, 5), device=device, dtype=torch.int32)
    layout_sfb = torch.zeros((num_groups, 5), device=device, dtype=torch.int32)

    a_tensors = []
    b_tensors = []
    a_scales_tensors = []
    b_scales_tensors = []

    for g in range(num_groups):
        m_g = group_ms[g]
        expert_offsets[g + 1] = expert_offsets[g] + m_g
        problem_sizes[g][:] = torch.tensor([m_g, n_g, k_g], device=device)
        if m_g != 0:
            a_g, a_scale = per_token_cast_to_fp8(torch.randn((m_g, k_g), device=device))
            a_tensors.append(a_g)
            a_scales_tensors.append(a_scale)

        b_g, b_scale = per_block_cast_to_fp8(torch.randn((n_g, k_g), device=device).t())
        b_tensors.append(b_g)
        b_scales_tensors.append(b_scale)

    a_stack = torch.empty(
        (expert_offsets[-1], k_g), device=device, dtype=torch.float8_e4m3fn
    )
    b_stack = torch.empty(
        (num_groups, n_g, k_g), device=device, dtype=torch.float8_e4m3fn
    )

    _aux_idx = 0
    for g in range(num_groups):
        if group_ms[g] != 0:
            a_stack[expert_offsets[g] : expert_offsets[g + 1]] = a_tensors[_aux_idx]
            _aux_idx += 1
        b_stack[g] = b_tensors[g].t()
    b_stack = b_stack.transpose(1, 2)

    a_scale_stack = torch.empty(
        (expert_offsets[-1], k_g // 128), device=device, dtype=torch.float32
    )
    b_scale_stack = torch.empty(
        (num_groups, n_g // 128, k_g // 128), device=device, dtype=torch.float32
    )

    _aux_idx = 0
    for g in range(num_groups):
        if group_ms[g] != 0:
            a_scale_stack[expert_offsets[g] : expert_offsets[g + 1]] = a_scales_tensors[
                _aux_idx
            ]
            _aux_idx += 1
        b_scale_stack[g] = b_scales_tensors[g].t()
    b_scale_stack = b_scale_stack.transpose(1, 2)

    c_out = torch.empty((expert_offsets[-1], n_g), device=device, dtype=out_dtype)
    a_strides = torch.full(
        (num_groups,), a_stack.stride(0), device=device, dtype=torch.int64
    )
    c_strides = torch.full(
        (num_groups,), c_out.stride(0), device=device, dtype=torch.int64
    )
    workspace = torch.empty((1024 * 1024 * 1024), device=device, dtype=torch.uint8)
    a_ptrs = torch.empty((num_groups,), device=device, dtype=torch.int64)
    b_ptrs = torch.empty((num_groups,), device=device, dtype=torch.int64)
    out_ptrs = torch.empty((num_groups,), device=device, dtype=torch.int64)
    a_scales_ptrs = torch.empty((num_groups,), device=device, dtype=torch.int64)
    b_scales_ptrs = torch.empty((num_groups,), device=device, dtype=torch.int64)

    def run_cutlass():
        fp8_blockwise_scaled_grouped_mm(
            c_out,
            a_ptrs,
            b_ptrs,
            out_ptrs,
            a_scales_ptrs,
            b_scales_ptrs,
            a_stack,
            b_stack,
            a_scale_stack,
            b_scale_stack,
            a_strides,
            a_strides,
            c_strides,
            layout_sfa,
            layout_sfb,
            problem_sizes,
            expert_offsets[:-1],
            workspace,
        )

    # warmup
    for _ in range(num_warmup):
        run_cutlass()
    torch.cuda.synchronize()

    # run
    start_event = torch.cuda.Event(enable_timing=True)
    end_event = torch.cuda.Event(enable_timing=True)
    start_event.record()
    for _ in range(num_run):
        run_cutlass()
    end_event.record()
    end_event.synchronize()
    torch.cuda.synchronize()
    avg = start_event.elapsed_time(end_event) / num_run * 1000  # us

    return avg, expert_offsets[-1]
````
**EN:** This section defines `bench_sgl`. It benchmarks the `SGL` path used by the module.
**CN:** 该部分定义 `bench_sgl`。它负责基准测试模块中与 `SGL` 相关的处理路径。

### Lines 298-298: Constants and configuration
````python
benchmark_kernels = {"es": bench_es, "sgl-kernel": bench_sgl}
````
**EN:** This block defines shared constants or configuration values such as `benchmark_kernels`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `benchmark_kernels`），供后续函数或控制流程复用。

### Lines 301-305: `ShapeArg` definition
````python
@dataclass
class ShapeArg:
    n: int
    k: int
    num_groups: int
````
**EN:** This section defines the class `ShapeArg`, grouping related state and behavior around Shape Arg.
**CN:** 该部分定义类 `ShapeArg`，把与 Shape Arg 相关的状态和行为组织在一起。

### Lines 308-343: `benchmark_one_shape` definition
````python
def benchmark_one_shape(
    shape_args: List[ShapeArg],
    num_warmup: int,
    num_run: int,
):
    for shape in shape_args:
        for batch_size in [
            128,
            256,
            384,
            512,
            640,
            768,
            896,
            1024,
            1280,
            1536,
            2048,
            3072,
        ]:
            group_ms = create_unbalanced_expert_token_distribution(
                batch_size, 8, shape.num_groups
            )
            print(
                f"\nBenchmark: batch_size={batch_size}, n={shape.n}, k={shape.k}, num_groups={shape.num_groups}"
            )
            for kernel_name, kernel_func in benchmark_kernels.items():
                average_time, m = kernel_func(
                    group_ms,
                    shape.n,
                    shape.k,
                    shape.num_groups,
                    num_warmup,
                    num_run,
                )
                print(f"{kernel_name}: {average_time} us")
````
**EN:** This section defines `benchmark_one_shape` and implements the core logic associated with benchmark one shape.
**CN:** 该部分定义 `benchmark_one_shape`，并实现与 benchmark one shape 相关的核心逻辑。

### Lines 346-369: `main` definition
````python
def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--num-warmup", type=int, default=3)
    parser.add_argument("--num-run", type=int, default=20)
    shape_args = [
        # DeepSeek-R1, gateup, TP = 8
        ShapeArg(n=512, k=7168, num_groups=256),
        # DeepSeek-R1, down, TP = 8
        ShapeArg(n=7168, k=256, num_groups=256),
        # DeepSeek-R1, gateup, TP = 4
        ShapeArg(n=1024, k=7168, num_groups=256),
        # DeepSeek-R1, down, TP = 4
        ShapeArg(n=7168, k=512, num_groups=256),
        # Qwen3-235B-A22B-FP8, gateup, TP = 4
        ShapeArg(n=768, k=4096, num_groups=128),
        # Qwen3-235B-A22B-FP8, down, TP = 4
        ShapeArg(n=4096, k=384, num_groups=128),
        # Qwen3-235B-A22B-FP8, gateup, TP = 2
        ShapeArg(n=1536, k=4096, num_groups=128),
        # Qwen3-235B-A22B-FP8, down, TP = 2
        ShapeArg(n=4096, k=768, num_groups=128),
    ]
    args = parser.parse_args()
    benchmark_one_shape(shape_args, args.num_warmup, args.num_run)
````
**EN:** This section defines `main` and implements the core logic associated with main.
**CN:** 该部分定义 `main`，并实现与 main 相关的核心逻辑。

### Lines 372-373: Command-line entry point
````python
if __name__ == "__main__":
    main()
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Benchmark driver / 基准驱动脚本
- **Primary symbols / 主要符号**: `ceil_div`, `per_token_cast_to_fp8`, `per_block_cast_to_fp8`, `create_unbalanced_expert_token_distribution`, `bench_es`, `bench_sgl`, `ShapeArg`, `benchmark_one_shape`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, CLI argument parsing / 命令行参数解析
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`
- **External / 外部**: `argparse`, `dataclasses`, `numpy`, `random`, `torch`, `typing`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
