# bench_rotary_embedding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_rotary_embedding.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `rotary embedding` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `rotary embedding` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and module setup
````python
import itertools
import os

import torch
import triton
from sgl_kernel.testing.rotary_embedding import (
    FlashInferRotaryEmbedding,
    FusedSetKVBufferArg,
    MHATokenToKVPool,
    RotaryEmbedding,
    create_inputs,
)

from sglang.srt.utils.bench_utils import bench_kineto
from sglang.utils import is_in_ci
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 17-17: Constants and configuration
````python
IS_CI = is_in_ci()
````
**EN:** This block defines shared constants or configuration values such as `IS_CI`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `IS_CI`），供后续函数或控制流程复用。

### Lines 18-19: Comments and local context
````python

# CI environment uses simplified parameters
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 20-32: Conditional logic
````python
if IS_CI:
    batch_seq_configs = [(1, 1)]  # Single config for CI
    save_kv_configs = [False]  # Single option for CI
else:
    batch_seq_configs = [
        (1, 1),
        (32, 1),
        (128, 1),
        (512, 1),
        (2, 512),
        (4, 4096),
    ]
    save_kv_configs = [False, True]
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 34-38: Constants and configuration
````python
configs = [
    (batch_size, seq_len, save_kv_cache)
    for batch_size, seq_len in batch_seq_configs
    for save_kv_cache in save_kv_configs
]
````
**EN:** This block defines shared constants or configuration values such as `configs`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `configs`），供后续函数或控制流程复用。

### Lines 41-104: `benchmark` definition
````python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["batch_size", "seq_len", "save_kv_cache"],
        x_vals=configs,
        line_arg="provider",
        line_vals=["sglang"],
        line_names=["SGL Kernel"],
        styles=[("green", "-")],
        ylabel="us",
        plot_name="bench_rotary_embedding",
        args={},
    )
)
def benchmark(batch_size, seq_len, save_kv_cache, provider):
    device = torch.device("cuda")

    num_q_heads = 32
    num_kv_heads = 8
    head_size = 64
    dtype = torch.bfloat16

    config = dict(
        head_size=head_size,
        rotary_dim=64,
        max_position_embeddings=4096,
        base=8000,
        is_neox_style=True,
        dtype=dtype,
    )
    rope_flashinfer = FlashInferRotaryEmbedding(**config).to(device)
    pool_flashinfer = MHATokenToKVPool(head_num=num_kv_heads, head_dim=head_size)

    inputs = create_inputs(
        head_size=head_size,
        batch_size=batch_size,
        seq_len=seq_len,
        device=device,
        dtype=dtype,
        num_q_heads=num_q_heads,
        num_kv_heads=num_kv_heads,
    )

    query_flashinfer, key_flashinfer = inputs["query"].clone(), inputs["key"].clone()

    bench_fn = lambda: rope_flashinfer.forward_cuda(
        inputs["pos_ids"],
        query_flashinfer,
        key_flashinfer,
        fused_set_kv_buffer_arg=(
            FusedSetKVBufferArg(
                value=inputs["value"],
                k_buffer=pool_flashinfer.k_buffer[0].view(-1, num_kv_heads * head_size),
                v_buffer=pool_flashinfer.v_buffer[0].view(-1, num_kv_heads * head_size),
                k_scale=None,
                v_scale=None,
                cache_loc=inputs["out_cache_loc"],
            )
            if save_kv_cache
            else None
        ),
    )

    time_s = bench_kineto(bench_fn, kernel_names="BatchQKApplyRotaryPosIds")
    return time_s * 1e6
````
**EN:** This section defines `benchmark` and implements the core logic associated with benchmark. Benchmark helpers from Triton are used to measure runtime behavior.
**CN:** 该部分定义 `benchmark`，并实现与 benchmark 相关的核心逻辑。 这里使用 Triton 的基准辅助工具来测量运行时行为。

### Lines 107-108: Command-line entry point
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
- **Internal / 内部**: `sgl_kernel.testing.rotary_embedding`, `sglang.srt.utils.bench_utils`, `sglang.utils`
- **External / 外部**: `itertools`, `os`, `torch`, `triton`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
