# bench_cutlass_mla.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_cutlass_mla.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `cutlass MLA` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `cutlass MLA` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Imports and module setup
````python
import argparse
import copy
import itertools
import os

import torch
import triton
from sgl_kernel import cutlass_mla_decode, cutlass_mla_get_workspace_size

from sglang.srt.utils import get_device_capability
from sglang.utils import is_in_ci
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 13-13: Constants and configuration
````python
IS_CI = is_in_ci()
````
**EN:** This block defines shared constants or configuration values such as `IS_CI`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `IS_CI`），供后续函数或控制流程复用。

### Lines 14-15: Comments and local context
````python

# CI environment uses simplified parameters
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 16-21: Conditional logic
````python
if IS_CI:
    bs_range = [1]  # Single batch size for CI
    qlen_range = [64]  # Single sequence length for CI
else:
    bs_range = [1, 8, 32, 64, 128, 256]
    qlen_range = [1, 64, 128, 256, 512, 1024, 2048, 4096, 8192]
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 23-23: Constants and configuration
````python
configs = list(itertools.product(bs_range, qlen_range))
````
**EN:** This block defines shared constants or configuration values such as `configs`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `configs`），供后续函数或控制流程复用。

### Lines 26-124: `benchmark` definition
````python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["batch_size", "seq_len"],
        x_vals=configs,
        x_log=False,
        line_arg="provider",
        line_vals=[
            "128 heads",
            "64 heads",
            "32 heads",
            "16 heads",
        ],
        line_names=[
            "128 heads",
            "64 heads",
            "32 heads",
            "16 heads",
        ],
        styles=[("green", "-"), ("green", "--"), ("blue", "-"), ("blue", "--")],
        ylabel="GB/s",
        plot_name="cutlass mla",
        args={},
    )
)
def benchmark(batch_size, seq_len, provider, block_size, num_kv_splits):
    d = 576
    dn = 64
    dv = 512

    h_q_map = {
        "128": 128,
        "64": 64,
        "32": 32,
        "16": 16,
    }
    parsed_h_q = next(
        (value for key, value in h_q_map.items() if key in provider), None
    )

    if parsed_h_q is None:
        raise ValueError(f"Unknown head configuration in provider: {provider}")
    h_q = parsed_h_q

    seq_lens = torch.full((batch_size,), seq_len, dtype=torch.int32, device="cuda")
    max_seq_len = seq_lens.max().item()
    block_num = (max_seq_len + block_size - 1) // block_size

    # Pad block_num so that small blocks can be packed into full 128-sized CUTLASS tiles.
    # One 128-wide tile can hold (128 // block_size) small blocks.
    pack_factor = 128 // block_size
    block_num = ((block_num + pack_factor - 1) // pack_factor) * pack_factor

    qn = (
        torch.randn(h_q, batch_size, d - dn, dtype=torch.bfloat16, device="cuda")
        * 100.0
    )
    qr = torch.randn(batch_size, h_q, dn, dtype=torch.bfloat16, device="cuda") * 100.0
    block_table = torch.randint(
        0,
        batch_size * block_num,
        (batch_size, block_num),
        dtype=torch.int32,
        device="cuda",
    )

    kv_cache = torch.randn(
        block_table.numel(), block_size, d, dtype=torch.bfloat16, device="cuda"
    )

    workspace_size = cutlass_mla_get_workspace_size(
        block_num * block_size, batch_size, num_kv_splits=num_kv_splits
    )
    workspace = torch.empty(workspace_size, device="cuda", dtype=torch.uint8)

    quantiles = [0.5, 0.2, 0.8]
    ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
        lambda: cutlass_mla_decode(
            qn.transpose(0, 1),
            qr,
            kv_cache,
            seq_lens,
            block_table,
            workspace,
            1.44,
            num_kv_splits,
        ),
        quantiles=quantiles,
    )

    q_size = qn.numel() * qn.element_size() + qr.numel() * qr.element_size()

    gbps = (
        lambda ms: (
            q_size + q_size * dv / d + kv_cache.numel() * kv_cache.element_size()
        )
        * 1e-9
        / (ms * 1e-3)
    )
    return gbps(ms), gbps(max_ms), gbps(min_ms)
````
**EN:** This section defines `benchmark` and implements the core logic associated with benchmark. Benchmark helpers from Triton are used to measure runtime behavior.
**CN:** 该部分定义 `benchmark`，并实现与 benchmark 相关的核心逻辑。 这里使用 Triton 的基准辅助工具来测量运行时行为。

### Lines 127-175: Command-line entry point
````python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--block-sizes",
        nargs="+",
        type=int,
        default=[1, 32, 64, 128],
        help="List of batch sizes",
    )
    parser.add_argument(
        "--num-kv-splits",
        nargs="+",
        type=int,
        default=[-1],
        help="List of batch sizes",
    )
    args = parser.parse_args()

    # Skip in CI environment or unsupported architectures
    if IS_CI:
        major, minor = get_device_capability()
        if major is None or major < 10:  # Requires compute capability 10.0+
            print("Skipping Cutlass MLA benchmark in CI environment")
            if major is not None:
                print(
                    f"Cutlass MLA requires compute capability 10.0+, but found {major}.{minor}"
                )
            else:
                print("Could not determine device capability")
        else:
            for block_size in args.block_sizes:
                for kv_split in args.num_kv_splits:
                    print(f"block_size={block_size}, num_kv_splits={kv_split}: ")
                    benchmark.run(
                        print_data=True,
                        block_size=block_size,
                        num_kv_splits=kv_split,
                    )
            print("Benchmark finished!")
    else:
        for block_size in args.block_sizes:
            for kv_split in args.num_kv_splits:
                print(f"block_size={block_size}, num_kv_splits={kv_split}: ")
                benchmark.run(
                    print_data=True,
                    block_size=block_size,
                    num_kv_splits=kv_split,
                )
        print("Benchmark finished!")
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Benchmark driver / 基准驱动脚本
- **Primary symbols / 主要符号**: `benchmark`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, Triton benchmarking / Triton 基准工具, CLI argument parsing / 命令行参数解析
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sglang.srt.utils`, `sglang.utils`
- **External / 外部**: `argparse`, `copy`, `itertools`, `os`, `torch`, `triton`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
