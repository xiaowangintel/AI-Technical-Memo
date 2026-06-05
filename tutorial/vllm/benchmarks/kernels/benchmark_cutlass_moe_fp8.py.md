# benchmark_cutlass_moe_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_cutlass_moe_fp8.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, mixture-of-experts benchmarks, FP8 kernel experiments, centered around `WEIGHT_SHAPES_MOE`, `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `DEFAULT_TP_SIZES`. / 实现与基准测试编排、混合专家基准、FP8 内核实验相关的逻辑，核心符号包括 `WEIGHT_SHAPES_MOE`, `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `DEFAULT_TP_SIZES`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 3-7)
```python
"""
Benchmark the performance of the cutlass_moe_fp8 kernel vs the triton_moe
kernel. Both kernels take in fp8 quantized weights and 16-bit activations,
but use different quantization strategies and backends.
"""
```
**EN:** The module docstring introduces Benchmark the performance of the cutlass_moe_fp8 kernel vs the triton_moe kernel. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Benchmark the performance of the cutlass_moe_fp8 kernel vs the triton_moe kernel 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 9-23)
```python
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from tests.kernels.moe.utils import make_dummy_moe_config
from vllm import _custom_ops as ops
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.all2all_utils import (
    maybe_make_prepare_finalize,
)
from vllm.model_executor.layers.fused_moe.config import fp8_w8a8_moe_quant_config
from vllm.model_executor.layers.fused_moe.experts.cutlass_moe import CutlassExpertsFp8
from vllm.model_executor.layers.fused_moe.fused_moe import fused_experts, fused_topk
from vllm.platforms import current_platform
from vllm.utils.argparse_utils import FlexibleArgumentParser
from vllm.v1.worker.workspace import init_workspace_manager
```
**EN:** This block gathers third-party packages such as `torch`; project-local modules such as `vllm.model_executor.layers.fused_moe.modular_kernel`, `tests.kernels.moe.utils`, `vllm`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.all2all_utils`, `vllm.model_executor.layers.fused_moe.config`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了第三方依赖，如 `torch`；项目内部模块，如 `vllm.model_executor.layers.fused_moe.modular_kernel`, `tests.kernels.moe.utils`, `vllm`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.all2all_utils`, `vllm.model_executor.layers.fused_moe.config`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 27-55)
```python
WEIGHT_SHAPES_MOE = {
    "mixtral-8x7b": [
        [8, 2, 4096, 14336],
    ],
    "deepseek-v2": [
        [160, 6, 5120, 12288],
    ],
    "custom-small": [
        [8, 2, 2048, 7168],
    ],
    "glm45-fp8": [
        [128, 8, 4096, 1408],
    ],
    "Llama-4-Maverick-17B-128E-Instruct-FP8": [
        [128, 1, 5120, 8192],
    ],
}

DEFAULT_MODELS = [
    "mixtral-8x7b",
]

DEFAULT_BATCH_SIZES = [4, 8, 16, 32, 64, 128, 256, 512, 1024, 2048]
DEFAULT_TP_SIZES = [1]

PER_ACT_TOKEN_OPTS = [False, True]
PER_OUT_CH_OPTS = [False, True]

FP8_DTYPE = current_platform.fp8_dtype()
```
**EN:** This top-level block prepares shared state such as `WEIGHT_SHAPES_MOE`, `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `DEFAULT_TP_SIZES`, `PER_ACT_TOKEN_OPTS`, `PER_OUT_CH_OPTS`. It uses `current_platform.fp8_dtype` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `WEIGHT_SHAPES_MOE`, `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `DEFAULT_TP_SIZES`, `PER_ACT_TOKEN_OPTS`, `PER_OUT_CH_OPTS`。它借助 `current_platform.fp8_dtype` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `bench_run` (lines 58-238)
```python
def bench_run(
    results: list,
    model: str,
    num_experts: int,
    topk: int,
    per_act_token: bool,
    per_out_ch: bool,
    mkn: tuple[int, int, int],
):
    init_workspace_manager(torch.accelerator.current_device_index())
    (m, k, n) = mkn

    dtype = torch.half
    device = "cuda"

    # Create input activations
    a = torch.randn((m, k), device=device, dtype=dtype) / 10

    # Create weights
    w1 = torch.randn((num_experts, 2 * n, k), device=device, dtype=dtype) / 10
    w2 = torch.randn((num_experts, k, n), device=device, dtype=dtype) / 10

    # Create FP8 quantized weights and scales for both kernels
    w1_fp8q = torch.empty((num_experts, 2 * n, k), device=device, dtype=FP8_DTYPE)
    w2_fp8q = torch.empty((num_experts, k, n), device=device, dtype=FP8_DTYPE)

    # Create scales based on quantization strategy
    if per_out_ch:
        # Per-channel quantization
        w1_scale = torch.empty(
            (num_experts, 2 * n, 1), device=device, dtype=torch.float32
        )
        w2_scale = torch.empty((num_experts, k, 1), device=device, dtype=torch.float32)
    else:
    # ... omitted for brevity ...
    triton_time_us = triton_graph_time * 1000
    cutlass_time_us = cutlass_graph_time * 1000

    return {
        "batch_size": m,
        "triton_time_us": triton_time_us,
        "cutlass_time_us": cutlass_time_us,
    }
```
**EN:** `bench_run` implements a helper used by `benchmark_cutlass_moe_fp8.py`. It mainly works with `results`, `model`, `num_experts`, `topk`, `per_act_token`, ... and relies on `init_workspace_manager`, `torch.accelerator.current_device_index`, `torch.randn`, `torch.empty`, `range`, `ops.scaled_fp8_quant` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `bench_run` 负责实现 `benchmark_cutlass_moe_fp8.py` 使用的辅助逻辑。 它主要处理 `results`, `model`, `num_experts`, `topk`, `per_act_token`, ...，并结合 `init_workspace_manager`, `torch.accelerator.current_device_index`, `torch.randn`, `torch.empty`, `range`, `ops.scaled_fp8_quant` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Function `main` (lines 241-305)
```python
def main(args):
    # Initialize workspace manager (required for CUTLASS MoE kernels)
    device = torch.device("cuda:0")
    init_workspace_manager(device)

    print("Benchmarking models:")
    for i, model in enumerate(args.models):
        print(f"[{i}]  {model}")

    all_results = []

    for model in args.models:
        for tp in args.tp_sizes:
            for layer in WEIGHT_SHAPES_MOE[model]:
                num_experts = layer[0]
                topk = layer[1]
                size_k = layer[2]
                size_n = layer[3] // tp

                if len(args.limit_k) > 0 and size_k not in args.limit_k:
                    continue

                if len(args.limit_n) > 0 and size_n not in args.limit_n:
                    continue

                for per_act_token in args.per_act_token_opts:
                    for per_out_ch in args.per_out_ch_opts:
                        print(
                            f"\n=== {model}, experts={num_experts}, topk={topk},"
                            f"per_act={per_act_token}, per_out_ch={per_out_ch} ==="
                        )

                        config_results = []
                        for size_m in args.batch_sizes:
    # ... omitted for brevity ...
                                    f"{result['batch_size']:<12}"
                                    f"{result['triton_time_us']:<15.2f}"
                                    f"{result['cutlass_time_us']:<15.2f}"
                                )

                            all_results.extend(config_results)

    print(f"\nTotal benchmarks completed: {len(all_results)}")
```
**EN:** `main` serves as the main entry point for this script. It mainly works with `args` and relies on `torch.device`, `init_workspace_manager`, `print`, `enumerate`, `len`, `bench_run` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 `args`，并结合 `torch.device`, `init_workspace_manager`, `print`, `enumerate`, `len`, `bench_run` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Entry point (lines 308-352)
```python
if __name__ == "__main__":
    parser = FlexibleArgumentParser(
        description="""Benchmark CUTLASS FP8 MOE vs Triton FP8 FUSED MOE
         across specified models/shapes/batches

        Example usage:
        python benchmark_cutlass_moe_fp8.py  \
            --model "Llama-4-Maverick-17B-128E-Instruct-FP8"  \
            --tp-sizes 8 \
            --batch-size 2 4 8  \
            --per-act-token-opts false \
            --per-out-ch-opts false

        """
    )
    parser.add_argument(
        "--models",
        nargs="+",
        type=str,
        default=DEFAULT_MODELS,
        choices=WEIGHT_SHAPES_MOE.keys(),
    )
    parser.add_argument("--tp-sizes", nargs="+", type=int, default=DEFAULT_TP_SIZES)
    parser.add_argument(
        "--batch-sizes", nargs="+", type=int, default=DEFAULT_BATCH_SIZES
    )
    parser.add_argument("--limit-k", nargs="+", type=int, default=[])
    parser.add_argument("--limit-n", nargs="+", type=int, default=[])
    parser.add_argument(
        "--per-act-token-opts",
        nargs="+",
        type=lambda x: x.lower() == "true",
        default=[False, True],
        help="Per-activation token quantization options (true/false)",
    )
    parser.add_argument(
        "--per-out-ch-opts",
        nargs="+",
        type=lambda x: x.lower() == "true",
        default=[False, True],
        help="Per-output channel quantization options (true/false)",
    )

    args = parser.parse_args()
    main(args)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `FlexibleArgumentParser`, `parser.add_argument`, `WEIGHT_SHAPES_MOE.keys`, `x.lower`, `parser.parse_args`, `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `FlexibleArgumentParser`, `parser.add_argument`, `WEIGHT_SHAPES_MOE.keys`, `x.lower`, `parser.parse_args`, `main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** MoE specialization: the benchmark focuses on expert routing, expert GEMMs, or fused MoE kernels.
- **CN:** MoE 专项：该基准聚焦专家路由、专家 GEMM 或融合 MoE 内核。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: none.
- **CN:** 标准库依赖：无。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: `vllm.model_executor.layers.fused_moe.modular_kernel`, `tests.kernels.moe.utils`, `vllm`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.all2all_utils`, `vllm.model_executor.layers.fused_moe.config`.
- **CN:** 内部模块：`vllm.model_executor.layers.fused_moe.modular_kernel`, `tests.kernels.moe.utils`, `vllm`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.all2all_utils`, `vllm.model_executor.layers.fused_moe.config`。
