# bench_moe_ep_post_reorder.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_moe_ep_post_reorder.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `MoE expert parallel post reorder` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `MoE expert parallel post reorder` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Imports and module setup
````python
import torch
import triton

from sglang.srt.layers.moe.ep_moe.kernels import post_reorder_triton_kernel
from sglang.utils import is_in_ci
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 7-7: Constants and configuration
````python
IS_CI = is_in_ci()
````
**EN:** This block defines shared constants or configuration values such as `IS_CI`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `IS_CI`），供后续函数或控制流程复用。

### Lines 8-9: Comments and local context
````python

# CI environment uses simplified parameters
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 10-13: Conditional logic
````python
if IS_CI:
    batch_sizes = [64, 128]  # Only test 2 values in CI
else:
    batch_sizes = [64, 128, 256, 512, 640, 768, 1024, 2048, 4096]
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 15-15: Constants and configuration
````python
configs = [(bs,) for bs in batch_sizes]
````
**EN:** This block defines shared constants or configuration values such as `configs`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `configs`），供后续函数或控制流程复用。

### Lines 18-81: `benchmark` definition
````python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["batch_size"],
        x_vals=[list(_) for _ in configs],
        line_arg="provider",
        line_vals=["triton"],
        line_names=["Triton Kernel"],
        styles=[("orange", "-")],
        ylabel="us",
        plot_name="ep-moe-post-reorder-performance",
        args={},
    )
)
def benchmark(batch_size, provider):
    dtype = torch.bfloat16
    device = torch.device("cuda")
    hidden_size, topk, start_expert_id, end_expert_id, block_size = 4096, 8, 0, 255, 512

    def alloc_tensors():
        down_output = torch.randn(
            batch_size * topk, hidden_size, dtype=dtype, device=device
        )
        output = torch.zeros(batch_size, hidden_size, dtype=dtype, device=device)
        src2dst = torch.randint(
            0, batch_size * topk, (batch_size, topk), dtype=torch.int32, device=device
        )
        topk_ids = torch.randint(
            start_expert_id,
            end_expert_id + 1,
            (batch_size, topk),
            dtype=torch.int32,
            device=device,
        )
        topk_weights = torch.rand(batch_size, topk, dtype=dtype, device=device)
        return down_output, output, src2dst, topk_ids, topk_weights

    quantiles = [0.5, 0.2, 0.8]

    if provider == "triton":
        d_out, out, s2d, tk_ids, tk_weights = alloc_tensors()

        def run_triton():
            post_reorder_triton_kernel[(batch_size,)](
                d_out.view(-1),
                out.view(-1),
                s2d.view(-1),
                tk_ids.view(-1),
                tk_weights.view(-1),
                start_expert_id,
                end_expert_id,
                topk,
                hidden_size,
                0,
                block_size,
            )

        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            run_triton, quantiles=quantiles
        )

    else:
        raise ValueError(f"Unknown provider: {provider}")

    return 1000 * ms, 1000 * max_ms, 1000 * min_ms
````
**EN:** This section defines `benchmark` and implements the core logic associated with benchmark. Benchmark helpers from Triton are used to measure runtime behavior.
**CN:** 该部分定义 `benchmark`，并实现与 benchmark 相关的核心逻辑。 这里使用 Triton 的基准辅助工具来测量运行时行为。

### Lines 84-85: Command-line entry point
````python
if __name__ == "__main__":
    benchmark.run(print_data=True)
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Benchmark driver / 基准驱动脚本
- **Primary symbols / 主要符号**: `benchmark`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, Triton benchmarking / Triton 基准工具
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.srt.layers.moe.ep_moe.kernels`, `sglang.utils`
- **External / 外部**: `torch`, `triton`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
