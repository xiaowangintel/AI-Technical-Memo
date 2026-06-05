# benchmark_marlin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_marlin.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, Marlin kernel experiments, centered around `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `ACT_ORDER_OPTS`, `K_FULL_OPTS`. / 实现与基准测试编排、Marlin 内核实验相关的逻辑，核心符号包括 `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `ACT_ORDER_OPTS`, `K_FULL_OPTS`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-38)
```python
import torch
import torch.utils.benchmark as benchmark
from benchmark_shapes import WEIGHT_SHAPES

from vllm import _custom_ops as ops
from vllm.model_executor.layers.quantization.utils.allspark_utils import (
    ALLSPARK_AMPERE_M_CUBLAS_THRESHOLD,
    ALLSPARK_SUPPORTED_QUANT_TYPES,
)
from vllm.model_executor.layers.quantization.utils.marlin_utils import (
    GPTQ_MARLIN_MAX_PARALLEL,
    GPTQ_MARLIN_MIN_THREAD_N,
    MARLIN_SUPPORTED_GROUP_SIZES,
    query_marlin_supported_quant_types,
)
from vllm.model_executor.layers.quantization.utils.marlin_utils_fp4 import (
    FP4_MARLIN_SUPPORTED_GROUP_SIZES,
    rand_marlin_weight_fp4_like,
)
from vllm.model_executor.layers.quantization.utils.marlin_utils_fp8 import (
    marlin_quant_fp8_torch,
)
from vllm.model_executor.layers.quantization.utils.marlin_utils_test import (
    MarlinWorkspace,
    awq_marlin_quantize,
    marlin_quantize,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    gptq_pack,
    gptq_quantize_weights,
    quantize_weights,
    sort_weights,
)
from vllm.scalar_type import ScalarType, scalar_types
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block gathers third-party packages such as `torch`, `torch.utils.benchmark`; project-local modules such as `benchmark_shapes`, `vllm`, `vllm.model_executor.layers.quantization.utils.allspark_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils_fp4`, `vllm.model_executor.layers.quantization.utils.marlin_utils_fp8`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了第三方依赖，如 `torch`, `torch.utils.benchmark`；项目内部模块，如 `benchmark_shapes`, `vllm`, `vllm.model_executor.layers.quantization.utils.allspark_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils_fp4`, `vllm.model_executor.layers.quantization.utils.marlin_utils_fp8`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 40-44)
```python
DEFAULT_MODELS = ["meta-llama/Llama-2-7b-hf/TP1"]
DEFAULT_BATCH_SIZES = [1, 16, 32, 64, 128, 256, 512, 1024, 2048, 4096, 8192]

ACT_ORDER_OPTS = [False, True]
K_FULL_OPTS = [False, True]
```
**EN:** This top-level block prepares shared state such as `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `ACT_ORDER_OPTS`, `K_FULL_OPTS`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `ACT_ORDER_OPTS`, `K_FULL_OPTS`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `bench_run` (lines 47-267)
```python
def bench_run(
    results: list[benchmark.Measurement],
    model: str,
    act_order: bool,
    is_k_full: bool,
    quant_type: ScalarType,
    group_size: int,
    size_m: int,
    size_k: int,
    size_n: int,
):
    label = "Quant Matmul"
    sub_label = "{}, act={} k_full={}, q={}, g={}, MKN=({}x{}x{})".format(
        model, act_order, is_k_full, str(quant_type), group_size, size_m, size_k, size_n
    )
    print(f"Testing: {sub_label}")

    a = torch.randn(size_m, size_k).to(torch.half).cuda()
    b = torch.rand(size_k, size_n).to(torch.half).cuda()
    has_zp = quant_type in [scalar_types.uint4, scalar_types.uint8]
    if act_order and (group_size == -1 or group_size == size_k or has_zp):
        return
    if size_k % group_size != 0:
        return

    repack_supported = group_size in MARLIN_SUPPORTED_GROUP_SIZES
    allspark_supported = (
        quant_type in ALLSPARK_SUPPORTED_QUANT_TYPES
        and group_size == -1
        and not act_order
        and is_k_full
    )

    def gen_marlin_params():
    # ... omitted for brevity ...
            benchmark.Timer(
                stmt="output = allspark_w8a16_gemm(a, qw_reorder, s_reorder, zp_reorder, size_n, group_size, sm_count, sm_version, CUBLAS_M_THRESHOLD, False, True)",  # noqa: E501
                globals=globals,
                label=label,
                sub_label=sub_label,
                description="allspark_w8a16_gemm_fp32",
            ).blocked_autorange(min_run_time=min_run_time)
        )
```
**EN:** `bench_run` implements a helper used by `benchmark_marlin.py`. It mainly works with `results`, `model`, `act_order`, `is_k_full`, `quant_type`, ... and relies on `format`, `str`, `print`, `torch.randn.to.cuda`, `torch.randn.to`, `torch.randn` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `bench_run` 负责实现 `benchmark_marlin.py` 使用的辅助逻辑。 它主要处理 `results`, `model`, `act_order`, `is_k_full`, `quant_type`, ...，并结合 `format`, `str`, `print`, `torch.randn.to.cuda`, `torch.randn.to`, `torch.randn` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `main` (lines 270-337)
```python
def main(args):
    print("Benchmarking models:")
    for i, model in enumerate(args.models):
        print(f"[{i}]  {model}")
    results: list[benchmark.Measurement] = []

    for model in args.models:
        for layer in WEIGHT_SHAPES[model]:
            size_k = layer[0]
            size_n = layer[1]

            if len(args.limit_k) > 0 and size_k not in args.limit_k:
                continue

            if len(args.limit_n) > 0 and size_n not in args.limit_n:
                continue

            for act_order in ACT_ORDER_OPTS:
                if (
                    len(args.limit_act_order) > 0
                    and act_order not in args.limit_act_order
                ):
                    continue

                for is_k_full in K_FULL_OPTS:
                    if (
                        len(args.limit_k_full) > 0
                        and is_k_full not in args.limit_k_full
                    ):
                        continue

                    for quant_type in query_marlin_supported_quant_types():
                        if (
                            len(args.limit_num_bits) > 0
    # ... omitted for brevity ...
                                    group_size,
                                    size_m,
                                    size_k,
                                    size_n,
                                )

    compare = benchmark.Compare(results)
    compare.print()
```
**EN:** `main` serves as the main entry point for this script. It mainly works with `args` and relies on `print`, `enumerate`, `len`, `query_marlin_supported_quant_types`, `bench_run`, `benchmark.Compare` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 `args`，并结合 `print`, `enumerate`, `len`, `query_marlin_supported_quant_types`, `bench_run`, `benchmark.Compare` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Entry point (lines 343-365)
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
        choices=WEIGHT_SHAPES.keys(),
    )
    parser.add_argument(
        "--batch-sizes", nargs="+", type=int, default=DEFAULT_BATCH_SIZES
    )
    parser.add_argument("--limit-k", nargs="+", type=int, default=[])
    parser.add_argument("--limit-n", nargs="+", type=int, default=[])
    parser.add_argument("--limit-group-size", nargs="+", type=int, default=[])
    parser.add_argument("--limit-num-bits", nargs="+", type=int, default=[])
    parser.add_argument("--limit-act-order", nargs="+", type=int, default=[])
    parser.add_argument("--limit-k-full", nargs="+", type=int, default=[])

    args = parser.parse_args()
    main(args)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `FlexibleArgumentParser`, `parser.add_argument`, `WEIGHT_SHAPES.keys`, `parser.parse_args`, `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `FlexibleArgumentParser`, `parser.add_argument`, `WEIGHT_SHAPES.keys`, `parser.parse_args`, `main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: none.
- **CN:** 标准库依赖：无。
- **EN:** Third-party packages: `torch`, `torch.utils.benchmark`.
- **CN:** 第三方依赖：`torch`, `torch.utils.benchmark`。
- **EN:** Internal modules: `benchmark_shapes`, `vllm`, `vllm.model_executor.layers.quantization.utils.allspark_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils_fp4`, `vllm.model_executor.layers.quantization.utils.marlin_utils_fp8`.
- **CN:** 内部模块：`benchmark_shapes`, `vllm`, `vllm.model_executor.layers.quantization.utils.allspark_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils_fp4`, `vllm.model_executor.layers.quantization.utils.marlin_utils_fp8`。
