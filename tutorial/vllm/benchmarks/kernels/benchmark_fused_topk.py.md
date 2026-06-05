# benchmark_fused_topk.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_fused_topk.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, sampling/top-k benchmarks, centered around `torch_topk`, `get_benchmark`. / 实现与基准测试编排、采样与 top-k 基准相关的逻辑，核心符号包括 `torch_topk`, `get_benchmark`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-10)
```python
import itertools

import torch

from vllm.model_executor.layers.fused_moe.router.fused_topk_router import fused_topk
from vllm.triton_utils import triton
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block gathers standard-library helpers such as `itertools`; third-party packages such as `torch`; project-local modules such as `vllm.model_executor.layers.fused_moe.router.fused_topk_router`, `vllm.triton_utils`, `vllm.utils.argparse_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `itertools`；第三方依赖，如 `torch`；项目内部模块，如 `vllm.model_executor.layers.fused_moe.router.fused_topk_router`, `vllm.triton_utils`, `vllm.utils.argparse_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 12-15)
```python
num_tokens_range = [2**i for i in range(0, 8, 2)]
num_experts_range = [16, 32, 64, 128, 256, 512]
topk_range = [3, 4]
configs = list(itertools.product(num_tokens_range, num_experts_range, topk_range))
```
**EN:** This top-level block prepares shared state such as `num_tokens_range`, `num_experts_range`, `topk_range`, `configs`. It uses `range`, `list`, `itertools.product` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `num_tokens_range`, `num_experts_range`, `topk_range`, `configs`。它借助 `range`, `list`, `itertools.product` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `torch_topk` (lines 18-33)
```python
def torch_topk(
    gating_output: torch.Tensor,
    topk: int,
    renormalize: bool,
    scoring_func: str = "softmax",
):
    if scoring_func == "softmax":
        scores = torch.softmax(gating_output.float(), dim=-1)
    else:
        scores = torch.sigmoid(gating_output.float())
    topk_weights, topk_ids = torch.topk(scores, k=topk, dim=-1)

    if renormalize:
        topk_weights = topk_weights / topk_weights.sum(dim=-1, keepdim=True)

    return topk_weights, topk_ids
```
**EN:** `torch_topk` implements sampling/top-k benchmarks. It mainly works with `gating_output`, `topk`, `renormalize`, `scoring_func` and relies on `torch.softmax`, `gating_output.float`, `torch.sigmoid`, `torch.topk`, `topk_weights.sum` plus branching to move data through this part of the benchmark pipeline.
**CN:** `torch_topk` 负责实现采样与 top-k 基准。 它主要处理 `gating_output`, `topk`, `renormalize`, `scoring_func`，并结合 `torch.softmax`, `gating_output.float`, `torch.sigmoid`, `torch.topk`, `topk_weights.sum` 以及 条件分支 来完成这一段基准测试流程。

### Function `get_benchmark` (lines 36-87)
```python
def get_benchmark(scoring_func):
    @triton.testing.perf_report(
        triton.testing.Benchmark(
            x_names=["num_tokens", "num_experts", "topk"],
            x_vals=[list(_) for _ in configs],
            line_arg="provider",
            line_vals=["torch", "vllm"],
            line_names=["Torch", "vLLM"],
            styles=[("blue", "-"), ("red", "-")],
            ylabel="us",
            plot_name=f"fused-topk-perf-{scoring_func}",
            args={},
        )
    )
    def benchmark(num_tokens, num_experts, topk, provider):
        dtype = torch.bfloat16
        hidden_size = 1024
        renormalize = True
        hidden_states = torch.randn(
            (num_tokens, hidden_size), dtype=dtype, device="cuda"
        )
        gating_output = torch.randn(
            (num_tokens, num_experts), dtype=dtype, device="cuda"
        )

        quantiles = [0.5, 0.2, 0.8]

        if provider == "torch":
            ms, min_ms, max_ms = triton.testing.do_bench(
                lambda: torch_topk(
                    gating_output=gating_output,
                    topk=topk,
                    renormalize=renormalize,
                    scoring_func=scoring_func,
    # ... omitted for brevity ...
                    scoring_func=scoring_func,
                ),
                quantiles=quantiles,
            )

        return 1000 * ms, 1000 * max_ms, 1000 * min_ms

    return benchmark
```
**EN:** `get_benchmark` coordinates or measures benchmark orchestration. It mainly works with `scoring_func` and relies on `torch.randn`, `triton.testing.do_bench`, `torch_topk`, `fused_topk`, `triton.testing.perf_report`, `triton.testing.Benchmark` plus branching to move data through this part of the benchmark pipeline.
**CN:** `get_benchmark` 负责协调或测量基准测试编排。 它主要处理 `scoring_func`，并结合 `torch.randn`, `triton.testing.do_bench`, `torch_topk`, `fused_topk`, `triton.testing.perf_report`, `triton.testing.Benchmark` 以及 条件分支 来完成这一段基准测试流程。

### Entry point (lines 90-99)
```python
if __name__ == "__main__":
    parser = FlexibleArgumentParser(description="Benchmark the MoE topk kernel.")
    parser.add_argument("--scoring-func", type=str, default="softmax")
    parser.add_argument("--save-path", type=str, default="./configs/fused_topk/")
    args = parser.parse_args()

    # Get the benchmark function
    benchmark = get_benchmark(args.scoring_func)
    # Run performance benchmark
    benchmark.run(print_data=True, save_path=args.save_path)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `get_benchmark`, `benchmark.run` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `get_benchmark`, `benchmark.run` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Result presentation: the module aggregates measurements and renders tables or plots.
- **CN:** 结果呈现：该模块会汇总测量结果并生成表格或图形。
- **EN:** MoE specialization: the benchmark focuses on expert routing, expert GEMMs, or fused MoE kernels.
- **CN:** MoE 专项：该基准聚焦专家路由、专家 GEMM 或融合 MoE 内核。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `itertools`.
- **CN:** 标准库依赖：`itertools`。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: `vllm.model_executor.layers.fused_moe.router.fused_topk_router`, `vllm.triton_utils`, `vllm.utils.argparse_utils`.
- **CN:** 内部模块：`vllm.model_executor.layers.fused_moe.router.fused_topk_router`, `vllm.triton_utils`, `vllm.utils.argparse_utils`。
