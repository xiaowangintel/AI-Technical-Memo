# benchmark_cpu_fused_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/cpu/benchmark_cpu_fused_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, CPU execution benchmarks, mixture-of-experts benchmarks, centered around `ISA_CHOICES`, `main`. / 实现与基准测试编排、CPU 执行基准、混合专家基准相关的逻辑，核心符号包括 `ISA_CHOICES`, `main`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-11)
```python
import sys
import time

import numpy as np
import torch

from vllm.utils.argparse_utils import FlexibleArgumentParser
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block gathers standard-library helpers such as `sys`, `time`; third-party packages such as `numpy`, `torch`; project-local modules such as `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `sys`, `time`；第三方依赖，如 `numpy`, `torch`；项目内部模块，如 `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 14-27)
```python
try:
    from vllm._custom_ops import cpu_fused_moe, cpu_prepack_moe_weight
except (ImportError, AttributeError) as e:
    print("ERROR: CPU fused MoE operations are not available on this platform.")
    print("This benchmark requires x86 CPU with proper vLLM CPU extensions compiled.")
    print(
        "The cpu_fused_moe kernel is typically available on Linux x86_64 "
        "with AVX2/AVX512."
    )
    print(f"Import error: {e}")
    sys.exit(1)

# ISA selection following test_cpu_fused_moe.py pattern
ISA_CHOICES = ["amx", "vec"] if torch.cpu._is_amx_tile_supported() else ["vec"]
```
**EN:** This top-level block prepares shared state such as `ISA_CHOICES`. It uses `print`, `sys.exit`, `torch.cpu._is_amx_tile_supported` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `ISA_CHOICES`。它借助 `print`, `sys.exit`, `torch.cpu._is_amx_tile_supported` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `main` (lines 31-121)
```python
def main(
    batch_size: int,
    expert_num: int,
    hidden_size: int,
    intermediate_size: int,
    topk_num: int,
    use_bias: bool = False,
    dtype: torch.dtype = torch.bfloat16,
    activation: str = "silu",
    isa: str = "vec",
    seed: int = 0,
    iters: int = 20,
) -> None:
    set_random_seed(seed)
    # up_dim = 2 * intermediate_size for gate + up projection
    up_dim = 2 * intermediate_size

    input_tensor = torch.randn((batch_size, hidden_size), dtype=dtype) / (
        0.5 * hidden_size**0.5
    )

    w13 = torch.randn((expert_num, up_dim, hidden_size), dtype=dtype) / (
        0.5 * hidden_size**0.5
    )
    w2 = torch.randn((expert_num, hidden_size, intermediate_size), dtype=dtype) / (
        0.5 * intermediate_size**0.5
    )

    w13_bias = None
    w2_bias = None
    if use_bias:
        w13_bias = torch.randn((expert_num, up_dim), dtype=dtype) / (0.5 * up_dim**0.5)
        w2_bias = torch.randn((expert_num, hidden_size), dtype=dtype) / (
            0.5 * hidden_size**0.5
    # ... omitted for brevity ...
    # Calculate throughput metrics
    # FLOPs estimation: 2 * batch * topk * (hidden * up_dim + intermediate * hidden)
    flops_per_token = (
        2 * topk_num * (hidden_size * up_dim + intermediate_size * hidden_size)
    )
    total_flops = batch_size * flops_per_token
    tflops = total_flops / (time_mean * 1e-3) / 1e12
    print(f"\tthroughput (TFLOP/s) = {tflops:.4f}")
```
**EN:** `main` serves as the main entry point for this script. It mainly works with `batch_size`, `expert_num`, `hidden_size`, `intermediate_size`, `topk_num`, ... and relies on `set_random_seed`, `torch.randn`, `torch.softmax`, `torch.topk`, `topk_ids.to`, `cpu_prepack_moe_weight` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 `batch_size`, `expert_num`, `hidden_size`, `intermediate_size`, `topk_num`, ...，并结合 `set_random_seed`, `torch.randn`, `torch.softmax`, `torch.topk`, `topk_ids.to`, `cpu_prepack_moe_weight` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Entry point (lines 124-175)
```python
if __name__ == "__main__":
    parser = FlexibleArgumentParser(description="Benchmark the CPU fused MoE kernel.")
    parser.add_argument("--batch-size", type=int, default=64)
    parser.add_argument("--expert-num", type=int, default=8)
    parser.add_argument("--hidden-size", type=int, default=2880)
    parser.add_argument("--intermediate-size", type=int, default=2880)
    parser.add_argument(
        "--topk-num",
        type=int,
        default=None,
        help="Number of experts to route each token to (default: expert_num // 2)",
    )
    parser.add_argument("--use-bias", action="store_true")
    parser.add_argument(
        "--activation",
        type=str,
        choices=["silu", "swigluoai"],
        default="silu",
        help="Activation function",
    )
    parser.add_argument(
        "--isa",
        type=str,
        choices=ISA_CHOICES,
        default=ISA_CHOICES[0],
        help=f"ISA to use (available: {ISA_CHOICES})",
    )
    parser.add_argument("--seed", type=int, default=0)
    parser.add_argument("--iters", type=int, default=20)

    args = parser.parse_args()

    # Default topk_num to expert_num // 2, minimum 1
    topk_num = (
    # ... omitted for brevity ...
        topk_num=topk_num,
        use_bias=args.use_bias,
        dtype=torch.bfloat16,  # Following test_cpu_fused_moe.py
        activation=args.activation,
        isa=args.isa,
        seed=args.seed,
        iters=args.iters,
    )
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `max`, `print`, `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `max`, `print`, `main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** MoE specialization: the benchmark focuses on expert routing, expert GEMMs, or fused MoE kernels.
- **CN:** MoE 专项：该基准聚焦专家路由、专家 GEMM 或融合 MoE 内核。
- **EN:** Performance metrics: the script emphasizes latency/throughput measurement rather than model quality.
- **CN:** 性能指标：该脚本更强调延迟/吞吐量测量，而不是模型质量。

## Dependencies / 依赖关系
- **EN:** Standard library: `sys`, `time`.
- **CN:** 标准库依赖：`sys`, `time`。
- **EN:** Third-party packages: `numpy`, `torch`.
- **CN:** 第三方依赖：`numpy`, `torch`。
- **EN:** Internal modules: `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`.
- **CN:** 内部模块：`vllm.utils.argparse_utils`, `vllm.utils.torch_utils`。
