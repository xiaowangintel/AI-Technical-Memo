# bench_per_token_group_quant_8bit.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_per_token_group_quant_8bit.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `per token group quantization 8bit` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `per token group quantization 8bit` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Imports and module setup
````python
import itertools
import os
import time
from functools import partial
from pathlib import Path

import torch
import triton
from sgl_kernel.test_utils import create_per_token_group_quant_test_data

from sglang.srt.layers.quantization.fp8_kernel import (
    create_per_token_group_quant_fp8_output_scale,
)
from sglang.srt.layers.quantization.fp8_kernel import (
    per_token_group_quant_8bit as triton_per_token_group_quant_8bit,
)
from sglang.srt.layers.quantization.fp8_kernel import (
    sglang_per_token_group_quant_8bit,
)
from sglang.srt.utils import is_hip
from sglang.srt.utils.bench_utils import bench_kineto
from sglang.utils import is_in_ci
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 24-27: Constants and configuration
````python
IS_CI = is_in_ci()

_is_hip = is_hip()
fp8_type_ = torch.float8_e4m3fnuz if _is_hip else torch.float8_e4m3fn
````
**EN:** This block defines shared constants or configuration values such as `IS_CI`, `_is_hip`, `fp8_type_`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `IS_CI`, `_is_hip`, `fp8_type_`），供后续函数或控制流程复用。

### Lines 30-30: Constants and configuration
````python
mode_concentrated = IS_CI or (os.environ.get("SGLANG_BENCH_MODE", "") == "concentrated")
````
**EN:** This block defines shared constants or configuration values such as `mode_concentrated`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `mode_concentrated`），供后续函数或控制流程复用。

### Lines 32-185: Conditional logic
````python
if int(os.environ.get("SGLANG_NSYS_PROFILING", "0")):
    configs = [
        [
            768 * 8,
            2048,
            128,
            48,
            fp8_type_,
            dict(
                column_major_scales=True,
                scale_tma_aligned=True,
                scale_ue8m0=True,
                fuse_silu_and_mul=True,
                # masked_layout_mode=None,
                masked_layout_mode="balanced",
                # masked_layout_mode="extreme",
            ),
        ]
    ]
elif mode_concentrated:
    configs = list(
        itertools.product(
            [768],
            [1536, 7168, 16384],
            [128],
            [None],
            [fp8_type_],
            [
                dict(
                    column_major_scales=True,
                    scale_tma_aligned=True,
                    scale_ue8m0=True,
                    fuse_silu_and_mul=False,
                    masked_layout_mode=None,
                ),
            ],
        )
    ) + list(
        itertools.product(
            [768 * 8],
            [2048],
            [128],
            [48],
            [fp8_type_],
            [
                dict(
                    column_major_scales=True,
                    scale_tma_aligned=True,
                    scale_ue8m0=True,
                    fuse_silu_and_mul=True,
                    masked_layout_mode=None,
                ),
                dict(
                    column_major_scales=True,
                    scale_tma_aligned=True,
                    scale_ue8m0=True,
                    fuse_silu_and_mul=True,
                    masked_layout_mode="balanced",
                ),
                dict(
                    column_major_scales=True,
                    scale_tma_aligned=True,
                    scale_ue8m0=True,
                    fuse_silu_and_mul=True,
                    masked_layout_mode="imbalanced",
                ),
                dict(
                    column_major_scales=True,
                    scale_tma_aligned=True,
                    scale_ue8m0=True,
                    fuse_silu_and_mul=True,
                    masked_layout_mode="extreme",
                ),
            ],
        )
    )
else:
    configs = list(
        itertools.product(
            [1, 4, 16, 64, 256, 768, 2048, 8192, 16384],
            [1536, 7168, 16384],
            [128],
            [None],
            [fp8_type_],
            [
                dict(
                    column_major_scales=False,
                    scale_tma_aligned=False,
                    scale_ue8m0=False,
                    fuse_silu_and_mul=False,
                    masked_layout_mode=None,
                ),
                dict(
                    column_major_scales=True,
                    scale_tma_aligned=False,
                    scale_ue8m0=False,
                    fuse_silu_and_mul=False,
                    masked_layout_mode=None,
                ),
                dict(
                    column_major_scales=True,
                    scale_tma_aligned=True,
                    scale_ue8m0=False,
                    fuse_silu_and_mul=False,
                    masked_layout_mode=None,
                ),
                dict(
                    column_major_scales=True,
                    scale_tma_aligned=True,
                    scale_ue8m0=True,
                    fuse_silu_and_mul=False,
                    masked_layout_mode=None,
                ),
            ],
        )
    ) + list(
        itertools.product(
            [1 * 8, 4 * 8, 64 * 8, 256 * 8, 768 * 8],
            [2048],
            [128],
            [8, 16, 32, 48],
            [fp8_type_],
            [
                dict(
                    column_major_scales=True,
                    scale_tma_aligned=True,
                    scale_ue8m0=True,
                    fuse_silu_and_mul=True,
                    masked_layout_mode=None,
                ),
                dict(
                    column_major_scales=True,
                    scale_tma_aligned=True,
                    scale_ue8m0=True,
                    fuse_silu_and_mul=True,
                    masked_layout_mode="balanced",
                ),
                dict(
                    column_major_scales=True,
                    scale_tma_aligned=True,
                    scale_ue8m0=True,
                    fuse_silu_and_mul=True,
                    masked_layout_mode="imbalanced",
                ),
                dict(
                    column_major_scales=True,
                    scale_tma_aligned=True,
                    scale_ue8m0=True,
                    fuse_silu_and_mul=True,
                    masked_layout_mode="extreme",
                ),
            ],
        )
    )
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 188-241: `benchmark` definition
````python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=[
            "num_tokens",
            "hidden_dim",
            "group_size",
            "num_ranks",
            "dst_dtype",
            "flags",
        ],
        x_vals=configs,
        line_arg="provider",
        line_vals=["triton", "sglang"],
        # Triton has multi kernels and we only report the time for the core one
        line_names=["Triton (Inaccurate)", "SGL Kernel"],
        styles=[("blue", "-"), ("green", "-")],
        ylabel="us",
        plot_name="per-token-group-quant-8bit-performance",
        args={},
    )
)
def benchmark(
    num_tokens, hidden_dim, group_size, num_ranks, dst_dtype, flags, provider
):
    print(
        f"Testing: {num_tokens=} {hidden_dim=} {group_size=} {num_ranks=} {dst_dtype=} {flags=} {provider=}"
    )

    x, masked_m = create_per_token_group_quant_test_data(
        num_tokens=num_tokens, hidden_dim=hidden_dim, num_ranks=num_ranks, flags=flags
    )

    fn, kernel_names = {
        "triton": (
            triton_per_token_group_quant_8bit,
            "_per_token_group_quant_8bit|_silu_and_mul_post_quant_kernel",
        ),
        "sglang": (
            partial(sglang_per_token_group_quant_8bit, enable_v2=True),
            "per_token_group_quant_8bit_kernel",
        ),
    }[provider]
    bench_fn = lambda: fn(
        x=x,
        masked_m=masked_m,
        group_size=group_size,
        dst_dtype=dst_dtype,
        **{k: v for k, v in flags.items() if k not in ["masked_layout_mode"]},
    )

    time_s = bench_kineto(
        bench_fn, kernel_names=kernel_names, num_tests=300 if mode_concentrated else 30
    )
    return time_s * 1e6
````
**EN:** This section defines `benchmark` and implements the core logic associated with benchmark. Benchmark helpers from Triton are used to measure runtime behavior.
**CN:** 该部分定义 `benchmark`，并实现与 benchmark 相关的核心逻辑。 这里使用 Triton 的基准辅助工具来测量运行时行为。

### Lines 244-245: Command-line entry point
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
- **Internal / 内部**: `sgl_kernel.test_utils`, `sglang.srt.layers.quantization.fp8_kernel`, `sglang.srt.utils`, `sglang.srt.utils.bench_utils`, `sglang.utils`
- **External / 外部**: `functools`, `itertools`, `os`, `pathlib`, `time`, `torch`, `triton`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
