# benchmark_quant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_quant.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, quantization benchmarks, centered around `main`. / 实现与基准测试编排、量化基准相关的逻辑，核心符号包括 `main`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-10)
```python
import time

import torch

from vllm import _custom_ops as ops
from vllm.utils.argparse_utils import FlexibleArgumentParser
from vllm.utils.torch_utils import STR_DTYPE_TO_TORCH_DTYPE, set_random_seed
```
**EN:** This block gathers standard-library helpers such as `time`; third-party packages such as `torch`; project-local modules such as `vllm`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `time`；第三方依赖，如 `torch`；项目内部模块，如 `vllm`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Function `main` (lines 14-59)
```python
def main(
    num_tokens: int,
    hidden_size: int,
    static_scale: bool,
    quant_dtype: torch.dtype,
    dtype: torch.dtype,
    seed: int = 0,
    do_profile: bool = False,
    num_warmup_iters: int = 5,
    num_iters: int = 100,
) -> None:
    set_random_seed(seed)
    torch.set_default_device("cuda")

    x = torch.randn(num_tokens, hidden_size, dtype=dtype)
    scale = torch.randn(1, 1, dtype=torch.float32) if static_scale else None

    def run_cuda_benchmark(num_iters: int, profile: bool = False) -> float:
        torch.accelerator.synchronize()
        if profile:
            torch.cuda.cudart().cudaProfilerStart()
        start_time = time.perf_counter()

        for _ in range(num_iters):
            if quant_dtype == torch.int8:
                ops.scaled_int8_quant(x, scale)
            else:
                ops.scaled_fp8_quant(x, scale)
        torch.accelerator.synchronize()

        end_time = time.perf_counter()
        if profile:
            torch.cuda.cudart().cudaProfilerStop()
        return (end_time - start_time) / num_iters

    # Warmup.
    print("Warming up...")
    run_benchmark = run_cuda_benchmark
    run_benchmark(num_iters=num_warmup_iters, profile=False)

    # Benchmark.
    if do_profile:
        latency = run_benchmark(num_iters=1, profile=True)
    else:
        latency = run_benchmark(num_iters=num_iters, profile=False)
    print(f"Kernel running time: {latency * 1000000:.3f} us")
```
**EN:** `main` serves as the main entry point for this script. It mainly works with `num_tokens`, `hidden_size`, `static_scale`, `quant_dtype`, `dtype`, ... and relies on `set_random_seed`, `torch.set_default_device`, `torch.randn`, `torch.accelerator.synchronize`, `torch.cuda.cudart.cudaProfilerStart`, `torch.cuda.cudart` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 `num_tokens`, `hidden_size`, `static_scale`, `quant_dtype`, `dtype`, ...，并结合 `set_random_seed`, `torch.set_default_device`, `torch.randn`, `torch.accelerator.synchronize`, `torch.cuda.cudart.cudaProfilerStart`, `torch.cuda.cudart` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Entry point (lines 62-108)
```python
if __name__ == "__main__":

    def to_torch_dtype(dt):
        if dt == "int8":
            return torch.int8
        if dt == "fp8":
            return torch.float8_e4m3fn
        raise ValueError(f"Unsupported dtype: {dt}")

    parser = FlexibleArgumentParser(
        description="Benchmark the quantization (fp8 or int8) kernel."
    )
    parser.add_argument("--num-tokens", type=int, default=4096)
    parser.add_argument("--hidden-size", type=int, default=8192)
    parser.add_argument("--static-scale", action="store_true")
    parser.add_argument(
        "--quant-dtype", type=str, choices=["fp8", "int8"], default="int8"
    )
    parser.add_argument(
        "--dtype", type=str, choices=["half", "bfloat16", "float"], default="half"
    )

    parser.add_argument("--seed", type=int, default=0)
    parser.add_argument("--profile", action="store_true")
    parser.add_argument("--num-warmup-iters", type=int, default=5)
    parser.add_argument(
        "--num-iters",
        type=int,
        default=100,
        help="Number of benchmark iterations. "
        "If --profile is set, this number is ignored",
    )

    args = parser.parse_args()
    print(args)

    main(
        num_tokens=args.num_tokens,
        hidden_size=args.hidden_size,
        static_scale=args.static_scale,
        quant_dtype=to_torch_dtype(args.quant_dtype),
        dtype=STR_DTYPE_TO_TORCH_DTYPE[args.dtype],
        seed=args.seed,
        do_profile=args.profile,
        num_warmup_iters=args.num_warmup_iters,
        num_iters=args.num_iters,
    )
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `ValueError`, `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `print`, `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `ValueError`, `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `print`, `main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。
- **EN:** Performance metrics: the script emphasizes latency/throughput measurement rather than model quality.
- **CN:** 性能指标：该脚本更强调延迟/吞吐量测量，而不是模型质量。

## Dependencies / 依赖关系
- **EN:** Standard library: `time`.
- **CN:** 标准库依赖：`time`。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: `vllm`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`.
- **CN:** 内部模块：`vllm`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`。
