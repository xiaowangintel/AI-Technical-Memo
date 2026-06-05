# benchmark_grouped_gemm_cutlass.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_grouped_gemm_cutlass.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, GEMM kernel benchmarks, multimodal processing benchmarks, centered around `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `DEFAULT_TP_SIZES`, `PER_ACT_TOKEN_OPTS`. / 实现与基准测试编排、GEMM 内核基准、多模态处理基准相关的逻辑，核心符号包括 `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `DEFAULT_TP_SIZES`, `PER_ACT_TOKEN_OPTS`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-22)
```python
import torch
import torch.utils.benchmark as benchmark
from benchmark_shapes import WEIGHT_SHAPES_MOE

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from tests.kernels.moe.utils import make_dummy_moe_config
from vllm import _custom_ops as ops
from vllm.config import ParallelConfig, VllmConfig, set_current_vllm_config
from vllm.model_executor.layers.fused_moe.all2all_utils import (
    maybe_make_prepare_finalize,
)
from vllm.model_executor.layers.fused_moe.config import fp8_w8a8_moe_quant_config
from vllm.model_executor.layers.fused_moe.experts.cutlass_moe import CutlassExpertsFp8
from vllm.model_executor.layers.fused_moe.fused_moe import (
    fused_experts,
    fused_topk,
)
from vllm.utils.argparse_utils import FlexibleArgumentParser
from vllm.v1.worker.workspace import init_workspace_manager
```
**EN:** This block gathers third-party packages such as `torch`, `torch.utils.benchmark`; project-local modules such as `benchmark_shapes`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `tests.kernels.moe.utils`, `vllm`, `vllm.config`, `vllm.model_executor.layers.fused_moe.all2all_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了第三方依赖，如 `torch`, `torch.utils.benchmark`；项目内部模块，如 `benchmark_shapes`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `tests.kernels.moe.utils`, `vllm`, `vllm.config`, `vllm.model_executor.layers.fused_moe.all2all_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 24-34)
```python
DEFAULT_MODELS = [
    "mistralai/Mixtral-8x7B-Instruct-v0.1",
    "deepseek-ai/DeepSeek-V2-Lite",
    "ibm-granite/granite-3.0-1b-a400m",
    "ibm-granite/granite-3.0-3b-a800m",
]
DEFAULT_BATCH_SIZES = [1, 4, 8, 16, 32, 64, 128, 256, 512]
DEFAULT_TP_SIZES = [1]

PER_ACT_TOKEN_OPTS = [False]
PER_OUT_CH_OPTS = [False]
```
**EN:** This top-level block prepares shared state such as `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `DEFAULT_TP_SIZES`, `PER_ACT_TOKEN_OPTS`, `PER_OUT_CH_OPTS`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `DEFAULT_TP_SIZES`, `PER_ACT_TOKEN_OPTS`, `PER_OUT_CH_OPTS`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `to_fp8` (lines 37-41)
```python
def to_fp8(tensor: torch.Tensor):
    finfo = torch.finfo(torch.float8_e4m3fn)
    return torch.round(tensor.clamp(min=finfo.min, max=finfo.max)).to(
        dtype=torch.float8_e4m3fn
    )
```
**EN:** `to_fp8` coordinates or measures FP8 kernel experiments. It mainly works with `tensor` and relies on `torch.finfo`, `torch.round.to`, `torch.round`, `tensor.clamp` plus value production to move data through this part of the benchmark pipeline.
**CN:** `to_fp8` 负责协调或测量FP8 内核实验。 它主要处理 `tensor`，并结合 `torch.finfo`, `torch.round.to`, `torch.round`, `tensor.clamp` 以及 结果返回 来完成这一段基准测试流程。

### Function `bench_run` (lines 44-361)
```python
def bench_run(
    results: list[benchmark.Measurement],
    model: str,
    num_experts: int,
    topk: int,
    per_act_token: bool,
    per_out_ch: bool,
    mkn: tuple[int, int, int],
):
    init_workspace_manager(torch.accelerator.current_device_index())
    label = "Quant Matmul"

    sub_label = (
        "{}, num_experts={}, topk={}, per_act_token={} per_out_ch={}, MKN=({})".format(
            model, num_experts, topk, per_act_token, per_out_ch, mkn
        )
    )

    print(f"Testing: {sub_label}")

    (m, k, n) = mkn

    dtype = torch.half

    a = torch.randn((m, k), device="cuda", dtype=dtype) / 10
    w1 = torch.randn((num_experts, 2 * n, k), device="cuda", dtype=dtype) / 10
    w2 = torch.randn((num_experts, k, n), device="cuda", dtype=dtype) / 10

    _, a_scale = ops.scaled_fp8_quant(a)

    w1_q = torch.empty(
        (num_experts, 2 * n, k), device="cuda", dtype=torch.float8_e4m3fn
    )
    w2_q = torch.empty((num_experts, k, n), device="cuda", dtype=torch.float8_e4m3fn)
    # ... omitted for brevity ...
        benchmark.Timer(
            stmt="replay_graph(cutlass_graph, num_runs)",
            globals=globals,
            label=label,
            sub_label=sub_label,
            description="grouped_gemm_moe_cuda_graphs",
        ).blocked_autorange(min_run_time=min_run_time)
    )
```
**EN:** `bench_run` implements a helper used by `benchmark_grouped_gemm_cutlass.py`. It mainly works with `results`, `model`, `num_experts`, `topk`, `per_act_token`, ... and relies on `init_workspace_manager`, `torch.accelerator.current_device_index`, `format`, `print`, `torch.randn`, `ops.scaled_fp8_quant` plus iteration and context management to move data through this part of the benchmark pipeline.
**CN:** `bench_run` 负责实现 `benchmark_grouped_gemm_cutlass.py` 使用的辅助逻辑。 它主要处理 `results`, `model`, `num_experts`, `topk`, `per_act_token`, ...，并结合 `init_workspace_manager`, `torch.accelerator.current_device_index`, `format`, `print`, `torch.randn`, `ops.scaled_fp8_quant` 以及 循环迭代、上下文管理 来完成这一段基准测试流程。

### Function `main` (lines 364-404)
```python
def main(args):
    # Initialize workspace manager (required for CUTLASS MoE kernels)
    device = torch.device("cuda:0")
    init_workspace_manager(device)

    print("Benchmarking models:")
    for i, model in enumerate(args.models):
        print(f"[{i}]  {model}")

    results: list[benchmark.Measurement] = []

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

                for per_act_token in PER_ACT_TOKEN_OPTS:
                    for per_out_ch in PER_OUT_CH_OPTS:
                        for size_m in DEFAULT_BATCH_SIZES:
                            mkn = (size_m, size_k, size_n)
                            bench_run(
                                results,
                                model,
                                num_experts,
                                topk,
                                per_act_token,
                                per_out_ch,
                                mkn,
                            )

    compare = benchmark.Compare(results)
    compare.print()
```
**EN:** `main` serves as the main entry point for this script. It mainly works with `args` and relies on `torch.device`, `init_workspace_manager`, `print`, `enumerate`, `len`, `bench_run` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 `args`，并结合 `torch.device`, `init_workspace_manager`, `print`, `enumerate`, `len`, `bench_run` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Entry point (lines 407-429)
```python
if __name__ == "__main__":
    parser = FlexibleArgumentParser(
        description="Benchmark Marlin across specified models/shapes/batches"
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
    parser.add_argument("--limit-num-groups", nargs="+", type=int, default=[])
    parser.add_argument("--limit-per-act-token", nargs="+", type=int, default=[])
    parser.add_argument("--limit-per-out-ch", nargs="+", type=int, default=[])

    args = parser.parse_args()
    main(args)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `FlexibleArgumentParser`, `parser.add_argument`, `WEIGHT_SHAPES_MOE.keys`, `parser.parse_args`, `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `FlexibleArgumentParser`, `parser.add_argument`, `WEIGHT_SHAPES_MOE.keys`, `parser.parse_args`, `main` 等调用串起来，使模块能够从命令行直接启动。

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
- **EN:** Third-party packages: `torch`, `torch.utils.benchmark`.
- **CN:** 第三方依赖：`torch`, `torch.utils.benchmark`。
- **EN:** Internal modules: `benchmark_shapes`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `tests.kernels.moe.utils`, `vllm`, `vllm.config`, `vllm.model_executor.layers.fused_moe.all2all_utils`.
- **CN:** 内部模块：`benchmark_shapes`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `tests.kernels.moe.utils`, `vllm`, `vllm.config`, `vllm.model_executor.layers.fused_moe.all2all_utils`。
