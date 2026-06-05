# bench_fp8_blockwise_group_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_fp8_blockwise_group_gemm.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `FP8 blockwise group GEMM` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `FP8 blockwise group GEMM` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Imports and module setup
````python
import argparse
import random
from dataclasses import dataclass
from typing import List, Tuple

import deep_gemm
import torch
from sgl_kernel import fp8_blockwise_scaled_grouped_mm

from sglang.utils import is_in_ci
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 12-12: Constants and configuration
````python
IS_CI = is_in_ci()
````
**EN:** This block defines shared constants or configuration values such as `IS_CI`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `IS_CI`），供后续函数或控制流程复用。

### Lines 15-16: `get_m_alignment_for_contiguous_layout` definition
````python
def get_m_alignment_for_contiguous_layout():
    return 128
````
**EN:** This section defines `get_m_alignment_for_contiguous_layout`. It retrieves or computes the `m alignment for contiguous layout` path used by the module.
**CN:** 该部分定义 `get_m_alignment_for_contiguous_layout`。它负责获取或计算模块中与 `m alignment for contiguous layout` 相关的处理路径。

### Lines 19-20: `ceil_div` definition
````python
def ceil_div(x: int, y: int) -> int:
    return (x + y - 1) // y
````
**EN:** This section defines `ceil_div` and implements the core logic associated with ceil div.
**CN:** 该部分定义 `ceil_div`，并实现与 ceil div 相关的核心逻辑。

### Lines 23-31: `per_token_cast_to_fp8` definition
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

### Lines 34-46: `per_block_cast_to_fp8` definition
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

### Lines 49-87: `construct_contiguous_grouped` definition
````python
def construct_contiguous_grouped(
    num_groups: int, expected_m_per_group: int, k: int, n: int
) -> Tuple[
    int,
    Tuple[torch.Tensor, torch.Tensor],
    Tuple[torch.Tensor, torch.Tensor],
    torch.Tensor,
    torch.Tensor,
    torch.Tensor,
]:
    alignment = get_m_alignment_for_contiguous_layout()
    group_ms = [int(expected_m_per_group) for _ in range(num_groups)]
    m = sum([ceil_div(x, alignment) * alignment for x in group_ms])

    x = torch.randn((m, k), device="cuda", dtype=torch.bfloat16)
    y = torch.randn((num_groups, n, k), device="cuda", dtype=torch.bfloat16)
    m_indices = torch.empty(m, device="cuda", dtype=torch.int32)
    out = torch.empty((m, n), device="cuda", dtype=torch.bfloat16)

    start = 0
    for i, group_m in enumerate(group_ms):
        actual_end = start + group_m
        aligned_end = start + ceil_div(group_m, alignment) * alignment
        m_indices[start:actual_end] = i
        m_indices[actual_end:aligned_end] = -1
        start = aligned_end

    assert m % 4 == 0, f"TMA alignment error: {m}"
    x_fp8 = per_token_cast_to_fp8(x)
    y_fp8 = (
        torch.empty_like(y, dtype=torch.float8_e4m3fn),
        torch.empty(
            (num_groups, ceil_div(n, 128), k // 128), device="cuda", dtype=torch.float
        ),
    )
    for i in range(num_groups):
        y_fp8[0][i], y_fp8[1][i] = per_block_cast_to_fp8(y[i])

    return m, x_fp8, y_fp8, m_indices, out
````
**EN:** This section defines `construct_contiguous_grouped` and implements the core logic associated with construct contiguous grouped. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义 `construct_contiguous_grouped`，并实现与 construct contiguous grouped 相关的核心逻辑。 它还会在继续执行前进行显式断言或形状检查。

### Lines 90-123: `bench_deepgemm` definition
````python
def bench_deepgemm(
    expected_m_per_group: int,
    n: int,
    k: int,
    num_groups: int,
    num_warmup: int,
    num_run: int,
) -> Tuple[float, int]:
    # construct tensors
    m, x_fp8, y_fp8, m_indices, out = construct_contiguous_grouped(
        num_groups, expected_m_per_group, k, n
    )

    def run_deepgemm():
        deep_gemm.m_grouped_fp8_gemm_nt_contiguous(x_fp8, y_fp8, out, m_indices)

    # warmup
    for _ in range(num_warmup):
        run_deepgemm()
    torch.cuda.synchronize()

    # run
    start_event = torch.cuda.Event(enable_timing=True)
    end_event = torch.cuda.Event(enable_timing=True)
    latencies: list[float] = []
    start_event.record()
    for _ in range(num_run):
        run_deepgemm()
    end_event.record()
    end_event.synchronize()
    torch.cuda.synchronize()
    avg = start_event.elapsed_time(end_event) / num_run * 1000  # us

    return avg, m
````
**EN:** This section defines `bench_deepgemm`. It benchmarks the `deepgemm` path used by the module.
**CN:** 该部分定义 `bench_deepgemm`。它负责基准测试模块中与 `deepgemm` 相关的处理路径。

### Lines 126-243: `bench_cutlass` definition
````python
def bench_cutlass(
    expected_m_per_group: int,
    n: int,
    k: int,
    num_groups: int,
    num_warmup: int,
    num_run: int,
) -> Tuple[float, int]:
    device = "cuda"
    alignment = 16
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

    # TODO(@TianQiLin666666): Unique group_ms in all bench function
    group_ms = [
        alignment * ceil_div(int(expected_m_per_group), alignment)
        for _ in range(num_groups)
    ]
    for g in range(num_groups):
        m_g = group_ms[g]
        expert_offsets[g + 1] = expert_offsets[g] + m_g
        problem_sizes[g][:] = torch.tensor([m_g, n_g, k_g], device=device)

        a_g, a_scale = per_token_cast_to_fp8(torch.randn((m_g, k_g), device=device))
        b_g, b_scale = per_block_cast_to_fp8(torch.randn((n_g, k_g), device=device).t())
        a_tensors.append(a_g)
        b_tensors.append(b_g)
        a_scales_tensors.append(a_scale)
        b_scales_tensors.append(b_scale)

    a_stack = torch.empty(
        (expert_offsets[-1], k_g), device=device, dtype=torch.float8_e4m3fn
    )
    b_stack = torch.empty(
        (num_groups, n_g, k_g), device=device, dtype=torch.float8_e4m3fn
    )

    for g in range(num_groups):
        a_stack[expert_offsets[g] : expert_offsets[g + 1]] = a_tensors[g]
        b_stack[g] = b_tensors[g].t()
    b_stack = b_stack.transpose(1, 2)

    a_scale_stack = torch.empty(
        (expert_offsets[-1], k_g // 128), device=device, dtype=torch.float32
    )
    b_scale_stack = torch.empty(
        (num_groups, n_g // 128, k_g // 128), device=device, dtype=torch.float32
    )

    for g in range(num_groups):
        a_scale_stack[expert_offsets[g] : expert_offsets[g + 1]] = a_scales_tensors[g]
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
**EN:** This section defines `bench_cutlass`. It benchmarks the `cutlass` path used by the module.
**CN:** 该部分定义 `bench_cutlass`。它负责基准测试模块中与 `cutlass` 相关的处理路径。

### Lines 246-254: `bench_sglang_triton` definition
````python
def bench_sglang_triton(
    expected_m_per_group: int,
    n: int,
    k: int,
    num_groups: int,
    num_warmup: int,
    num_run: int,
) -> Tuple[float, int]:
    pass
````
**EN:** This section defines `bench_sglang_triton`. It benchmarks the `sglang triton` path used by the module.
**CN:** 该部分定义 `bench_sglang_triton`。它负责基准测试模块中与 `sglang triton` 相关的处理路径。

### Lines 257-261: Constants and configuration
````python
benchmark_kernels = {
    "deepgemm": bench_deepgemm,
    "cutlass": bench_cutlass,
    # "triton": bench_sglang_triton,
}
````
**EN:** This block defines shared constants or configuration values such as `benchmark_kernels`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `benchmark_kernels`），供后续函数或控制流程复用。

### Lines 264-269: `ShapeArg` definition
````python
@dataclass
class ShapeArg:
    expected_m_per_group: int
    n: int
    k: int
    num_groups: int
````
**EN:** This section defines the class `ShapeArg`, grouping related state and behavior around Shape Arg.
**CN:** 该部分定义类 `ShapeArg`，把与 Shape Arg 相关的状态和行为组织在一起。

### Lines 272-290: `benchmark_one_shape` definition
````python
def benchmark_one_shape(
    shape_args: List[ShapeArg],
    num_warmup: int,
    num_run: int,
):
    for shape in shape_args:
        print(
            f"\nBenchmark: expected_m_per_group={shape.expected_m_per_group}, n={shape.n}, k={shape.k}, num_groups={shape.num_groups}"
        )
        for kernel_name, kernel_func in benchmark_kernels.items():
            average_time, m = kernel_func(
                shape.expected_m_per_group,
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

### Lines 293-336: `main` definition
````python
def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--num-warmup", type=int, default=3)
    parser.add_argument("--num-run", type=int, default=10)

    # CI environment uses simplified parameters
    if IS_CI:
        shape_args = [
            # Only test one simple shape in CI
            ShapeArg(expected_m_per_group=128, n=512, k=7168, num_groups=256),
        ]
    else:
        shape_args = [
            # Prefill, DeepSeek-R1, gateup, chunk_size = 4096, TP = 8
            ShapeArg(expected_m_per_group=128, n=512, k=7168, num_groups=256),
            # Prefill, DeepSeek-R1, gateup, chunk_size = 8192, TP = 8
            ShapeArg(expected_m_per_group=256, n=512, k=7168, num_groups=256),
            # Prefill, DeepSeek-R1, gateup, chunk_size = 8192, TP = 16
            ShapeArg(expected_m_per_group=256, n=256, k=7168, num_groups=256),
            # Prefill, DeepSeek-R1, gateup, chunk_size = 16384, TP = 16
            ShapeArg(expected_m_per_group=512, n=256, k=7168, num_groups=256),
            # Decode, DeepSeek-R1, gateup, bs = 32, TP = 8
            ShapeArg(expected_m_per_group=1, n=512, k=7168, num_groups=256),
            # Decode, DeepSeek-R1, gateup, bs = 64, TP = 16
            ShapeArg(expected_m_per_group=2, n=256, k=7168, num_groups=256),
            # Prefill, DeepSeek-R1, gateup, chunk_size = 8192, EP = 8
            ShapeArg(expected_m_per_group=256, n=4096, k=7168, num_groups=32),
            # Prefill, DeepSeek-R1, gateup, chunk_size = 16384, EP = 16
            ShapeArg(expected_m_per_group=512, n=4096, k=7168, num_groups=16),
            # Decode, DeepSeek-R1, gateup, bs = 128, EP = 8
            ShapeArg(expected_m_per_group=4, n=4096, k=7168, num_groups=32),
            # Decode, DeepSeek-R1, gateup, bs = 256, EP = 16
            ShapeArg(expected_m_per_group=8, n=4096, k=7168, num_groups=16),
            # Prefill, Qwen3-235B-A22B-FP8, gateup, chunk_size = 16384, TP = 4
            ShapeArg(expected_m_per_group=1024, n=768, k=4096, num_groups=128),
            # Prefill, Qwen3-235B-A22B-FP8, down, chunk_size = 16384, TP = 4
            ShapeArg(expected_m_per_group=1024, n=4096, k=384, num_groups=128),
            # Decode, Qwen3-235B-A22B-FP8, gateup, bs = 256, TP = 4
            ShapeArg(expected_m_per_group=16, n=768, k=4096, num_groups=128),
            # Decode, Qwen3-235B-A22B-FP8, down, bs = 256, TP = 4
            ShapeArg(expected_m_per_group=16, n=4096, k=384, num_groups=128),
        ]
    args = parser.parse_args()
    benchmark_one_shape(shape_args, args.num_warmup, args.num_run)
````
**EN:** This section defines `main` and implements the core logic associated with main.
**CN:** 该部分定义 `main`，并实现与 main 相关的核心逻辑。

### Lines 339-340: Command-line entry point
````python
if __name__ == "__main__":
    main()
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Benchmark driver / 基准驱动脚本
- **Primary symbols / 主要符号**: `get_m_alignment_for_contiguous_layout`, `ceil_div`, `per_token_cast_to_fp8`, `per_block_cast_to_fp8`, `construct_contiguous_grouped`, `bench_deepgemm`, `bench_cutlass`, `bench_sglang_triton`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, Triton benchmarking / Triton 基准工具, CLI argument parsing / 命令行参数解析
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sglang.utils`
- **External / 外部**: `argparse`, `dataclasses`, `deep_gemm`, `random`, `torch`, `typing`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
