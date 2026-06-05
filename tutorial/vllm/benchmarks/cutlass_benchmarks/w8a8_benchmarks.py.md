# w8a8_benchmarks.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/cutlass_benchmarks/w8a8_benchmarks.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements W8A8 quantization benchmarks, benchmark orchestration, centered around `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `DEFAULT_TP_SIZES`, `bench_fn`. / 实现与W8A8 量化基准、基准测试编排相关的逻辑，核心符号包括 `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `DEFAULT_TP_SIZES`, `bench_fn`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-22)
```python
import argparse
import copy
import itertools
import pickle as pkl
import time
from collections.abc import Callable, Iterable

import torch
import torch.utils.benchmark as TBenchmark
from torch.utils.benchmark import Measurement as TMeasurement
from utils import make_rand_tensors
from weight_shapes import WEIGHT_SHAPES

from vllm import _custom_ops as ops
from vllm.model_executor.layers.quantization.utils.fp8_utils import (
    w8a8_triton_block_scaled_mm,
)
from vllm.utils.argparse_utils import FlexibleArgumentParser
from vllm.utils.math_utils import cdiv
```
**EN:** This block gathers standard-library helpers such as `argparse`, `copy`, `itertools`, `pickle`, `time`, `collections.abc`; third-party packages such as `torch`, `torch.utils.benchmark`; project-local modules such as `utils`, `weight_shapes`, `vllm`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.utils.argparse_utils`, `vllm.utils.math_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `copy`, `itertools`, `pickle`, `time`, `collections.abc`；第三方依赖，如 `torch`, `torch.utils.benchmark`；项目内部模块，如 `utils`, `weight_shapes`, `vllm`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.utils.argparse_utils`, `vllm.utils.math_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 24-26)
```python
DEFAULT_MODELS = list(WEIGHT_SHAPES.keys())
DEFAULT_BATCH_SIZES = [1, 16, 32, 64, 128, 256, 512]
DEFAULT_TP_SIZES = [1]
```
**EN:** This top-level block prepares shared state such as `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `DEFAULT_TP_SIZES`. It uses `list`, `WEIGHT_SHAPES.keys` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `DEFAULT_TP_SIZES`。它借助 `list`, `WEIGHT_SHAPES.keys` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `bench_fn` (lines 30-46)
```python
def bench_fn(
    label: str, sub_label: str, description: str, fn: Callable, *args, **kwargs
) -> TMeasurement:
    min_run_time = 1

    globals = {
        "args": args,
        "kwargs": kwargs,
        "fn": fn,
    }
    return TBenchmark.Timer(
        stmt="fn(*args, **kwargs)",
        globals=globals,
        label=label,
        sub_label=sub_label,
        description=description,
    ).blocked_autorange(min_run_time=min_run_time)
```
**EN:** `bench_fn` implements a helper used by `w8a8_benchmarks.py`. It mainly works with `label`, `sub_label`, `description`, `fn`, `*args`, ... and relies on `TBenchmark.Timer.blocked_autorange`, `TBenchmark.Timer` plus value production to move data through this part of the benchmark pipeline.
**CN:** `bench_fn` 负责实现 `w8a8_benchmarks.py` 使用的辅助逻辑。 它主要处理 `label`, `sub_label`, `description`, `fn`, `*args`, ...，并结合 `TBenchmark.Timer.blocked_autorange`, `TBenchmark.Timer` 以及 结果返回 来完成这一段基准测试流程。

### Function `bench_int8` (lines 49-101)
```python
def bench_int8(
    dtype: torch.dtype,
    m: int,
    k: int,
    n: int,
    label: str,
    sub_label: str,
    bench_kernels: list[str] | None = None,
) -> Iterable[TMeasurement]:
    """Benchmark INT8-based kernels."""
    assert dtype == torch.int8
    a, b = make_rand_tensors(torch.int8, m, n, k)
    scale_a = torch.tensor(1.0, device="cuda", dtype=torch.float32)
    scale_b = torch.tensor(1.0, device="cuda", dtype=torch.float32)
    bias = torch.zeros((n,), device="cuda", dtype=torch.bfloat16)
    azp = torch.zeros((m,), device="cuda", dtype=torch.int32)
    azp_adj = torch.zeros((n,), device="cuda", dtype=torch.int32)

    bench_fns = {
        "pytorch_bf16_bf16_bf16_matmul-no-scales": lambda: torch.mm(
            a.to(dtype=torch.bfloat16), b.to(dtype=torch.bfloat16)
        ),
        "pytorch_fp16_fp16_fp16_matmul-no-scales": lambda: torch.mm(
            a.to(dtype=torch.float16), b.to(dtype=torch.float16)
        ),
        "cutlass_i8_i8_bf16_scaled_mm": lambda: ops.cutlass_scaled_mm(
            a, b, scale_a, scale_b, torch.bfloat16
        ),
        "cutlass_i8_i8_bf16_scaled_mm_bias": lambda: ops.cutlass_scaled_mm(
            a, b, scale_a, scale_b, torch.bfloat16, bias
        ),
        "cutlass_i8_i8_bf16_scaled_mm_azp": lambda: ops.cutlass_scaled_mm_azp(
            a, b, scale_a, scale_b, torch.bfloat16, azp_adj
        ),
    # ... omitted for brevity ...
    timers = []
    for name, fn in bench_fns.items():
        # If bench_kernels is None, run all. Otherwise, run only exact matches.
        if bench_kernels is None or name in bench_kernels:
            print(f"Running {name}")
            timers.append(bench_fn(label, sub_label, name, fn))

    return timers
```
**EN:** `bench_int8` Benchmark INT8-based kernels. It mainly works with `dtype`, `m`, `k`, `n`, `label`, ... and relies on `make_rand_tensors`, `torch.tensor`, `torch.zeros`, `torch.mm`, `a.to`, `b.to` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `bench_int8` 的职责是：Benchmark INT8-based kernels。 它主要处理 `dtype`, `m`, `k`, `n`, `label`, ...，并结合 `make_rand_tensors`, `torch.tensor`, `torch.zeros`, `torch.mm`, `a.to`, `b.to` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `bench_fp8` (lines 104-176)
```python
def bench_fp8(
    dtype: torch.dtype,
    m: int,
    k: int,
    n: int,
    label: str,
    sub_label: str,
    bench_kernels: list[str] | None = None,
) -> Iterable[TMeasurement]:
    """Benchmark FP8-based kernels."""
    assert dtype == torch.float8_e4m3fn
    a, b = make_rand_tensors(torch.float8_e4m3fn, m, n, k)
    a_cont = a.contiguous()
    scale_a = torch.tensor(1.0, device="cuda", dtype=torch.float32)
    scale_b = torch.tensor(1.0, device="cuda", dtype=torch.float32)

    block_scale_a = torch.rand((m, cdiv(k, 128)), device="cuda", dtype=torch.float32)
    block_scale_b = torch.rand(
        cdiv(k, 128), cdiv(n, 128), device="cuda", dtype=torch.float32
    )
    block_scale_a_M_major = block_scale_a.t().contiguous().t()
    block_scale_b_K_major = block_scale_b.t().contiguous().t()
    bias = torch.zeros((n,), device="cuda", dtype=torch.bfloat16)

    print(m, k, n)

    bench_fns = {
        "pytorch_bf16_bf16_bf16_matmul-no-scales": lambda: torch.mm(
            a.to(dtype=torch.bfloat16), b.to(dtype=torch.bfloat16)
        ),
        "pytorch_fp16_fp16_fp16_matmul-no-scales": lambda: torch.mm(
            a.to(dtype=torch.float16), b.to(dtype=torch.float16)
        ),
        "pytorch_fp8_fp8_fp16_scaled_mm": lambda: torch._scaled_mm(
    # ... omitted for brevity ...
    timers = []
    for name, fn in bench_fns.items():
        # If bench_kernels is None, run all. Otherwise, run only exact matches.
        if bench_kernels is None or name in bench_kernels:
            print(f"Running {name}")
            timers.append(bench_fn(label, sub_label, name, fn))

    return timers
```
**EN:** `bench_fp8` Benchmark FP8-based kernels. It mainly works with `dtype`, `m`, `k`, `n`, `label`, ... and relies on `make_rand_tensors`, `a.contiguous`, `torch.tensor`, `torch.rand`, `cdiv`, `block_scale_a.t.contiguous.t` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `bench_fp8` 的职责是：Benchmark FP8-based kernels。 它主要处理 `dtype`, `m`, `k`, `n`, `label`, ...，并结合 `make_rand_tensors`, `a.contiguous`, `torch.tensor`, `torch.rand`, `cdiv`, `block_scale_a.t.contiguous.t` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `bench` (lines 179-192)
```python
def bench(
    dtype: torch.dtype,
    m: int,
    k: int,
    n: int,
    label: str,
    sub_label: str,
    bench_kernels: list[str] | None = None,
) -> Iterable[TMeasurement]:
    if dtype == torch.int8:
        return bench_int8(dtype, m, k, n, label, sub_label, bench_kernels)
    if dtype == torch.float8_e4m3fn:
        return bench_fp8(dtype, m, k, n, label, sub_label, bench_kernels)
    raise ValueError("unsupported type")
```
**EN:** `bench` implements a helper used by `w8a8_benchmarks.py`. It mainly works with `dtype`, `m`, `k`, `n`, `label`, ... and relies on `bench_int8`, `bench_fp8`, `ValueError` plus branching to move data through this part of the benchmark pipeline.
**CN:** `bench` 负责实现 `w8a8_benchmarks.py` 使用的辅助逻辑。 它主要处理 `dtype`, `m`, `k`, `n`, `label`, ...，并结合 `bench_int8`, `bench_fp8`, `ValueError` 以及 条件分支 来完成这一段基准测试流程。

### Function `print_timers` (lines 196-198)
```python
def print_timers(timers: Iterable[TMeasurement]):
    compare = TBenchmark.Compare(timers)
    compare.print()
```
**EN:** `print_timers` formats results for display or export. It mainly works with `timers` and relies on `TBenchmark.Compare`, `compare.print` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `print_timers` 负责格式化结果以便展示或导出。 它主要处理 `timers`，并结合 `TBenchmark.Compare`, `compare.print` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Function `run` (lines 201-219)
```python
def run(
    dtype: torch.dtype,
    MKNs: Iterable[tuple[int, int, int]],
    bench_kernels: list[str] | None = None,
) -> Iterable[TMeasurement]:
    results = []
    for m, k, n in MKNs:
        timers = bench(
            dtype,
            m,
            k,
            n,
            f"scaled-{dtype}-gemm",
            f"MKN=({m}x{k}x{n})",
            bench_kernels=bench_kernels,
        )
        print_timers(timers)
        results.extend(timers)
    return results
```
**EN:** `run` implements a helper used by `w8a8_benchmarks.py`. It mainly works with `dtype`, `MKNs`, `bench_kernels` and relies on `bench`, `print_timers`, `results.extend` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `run` 负责实现 `w8a8_benchmarks.py` 使用的辅助逻辑。 它主要处理 `dtype`, `MKNs`, `bench_kernels`，并结合 `bench`, `print_timers`, `results.extend` 以及 循环迭代 来完成这一段基准测试流程。

### Function `make_output` (lines 222-234)
```python
def make_output(
    data: Iterable[TMeasurement],
    MKNs: Iterable[tuple[int, int, int]],
    base_description: str,
    timestamp=None,
):
    print(f"== All Results {base_description} ====")
    print_timers(data)

    # pickle all the results
    timestamp = int(time.time()) if timestamp is None else timestamp
    with open(f"{base_description}-{timestamp}.pkl", "wb") as f:
        pkl.dump(data, f)
```
**EN:** `make_output` implements a helper used by `w8a8_benchmarks.py`. It mainly works with `data`, `MKNs`, `base_description`, `timestamp` and relies on `print`, `print_timers`, `int`, `time.time`, `open`, `pkl.dump` plus context management to move data through this part of the benchmark pipeline.
**CN:** `make_output` 负责实现 `w8a8_benchmarks.py` 使用的辅助逻辑。 它主要处理 `data`, `MKNs`, `base_description`, `timestamp`，并结合 `print`, `print_timers`, `int`, `time.time`, `open`, `pkl.dump` 以及 上下文管理 来完成这一段基准测试流程。

### Function `run_square_bench` (lines 237-241)
```python
def run_square_bench(args):
    dim_sizes = list(range(args.dim_start, args.dim_end + 1, args.dim_increment))
    MKNs = list(zip(dim_sizes, dim_sizes, dim_sizes))
    data = run(args.dtype, MKNs, bench_kernels=args.kernels)
    make_output(data, MKNs, f"square_bench-{args.dtype}")
```
**EN:** `run_square_bench` implements a helper used by `w8a8_benchmarks.py`. It mainly works with `args` and relies on `list`, `range`, `zip`, `run`, `make_output` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `run_square_bench` 负责实现 `w8a8_benchmarks.py` 使用的辅助逻辑。 它主要处理 `args`，并结合 `list`, `range`, `zip`, `run`, `make_output` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Function `run_range_bench` (lines 244-252)
```python
def run_range_bench(args):
    dim_sizes = list(range(args.dim_start, args.dim_end, args.dim_increment))
    n = len(dim_sizes)
    Ms = [args.m_constant] * n if args.m_constant is not None else dim_sizes
    Ks = [args.k_constant] * n if args.k_constant is not None else dim_sizes
    Ns = [args.n_constant] * n if args.n_constant is not None else dim_sizes
    MKNs = list(zip(Ms, Ks, Ns))
    data = run(args.dtype, MKNs, bench_kernels=args.kernels)
    make_output(data, MKNs, f"range_bench-{args.dtype}")
```
**EN:** `run_range_bench` implements a helper used by `w8a8_benchmarks.py`. It mainly works with `args` and relies on `list`, `range`, `len`, `zip`, `run`, `make_output` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `run_range_bench` 负责实现 `w8a8_benchmarks.py` 使用的辅助逻辑。 它主要处理 `args`，并结合 `list`, `range`, `len`, `zip`, `run`, `make_output` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Function `run_model_bench` (lines 255-293)
```python
def run_model_bench(args):
    print("Benchmarking models:")
    for i, model in enumerate(args.models):
        print(f"[{i}]  {model}")

    def model_shapes(model_name: str, tp_size: int) -> list[tuple[int, int]]:
        KNs = []
        for KN, tp_split_dim in copy.deepcopy(WEIGHT_SHAPES[model_name]):
            KN[tp_split_dim] = KN[tp_split_dim] // tp_size
            KNs.append(KN)
        return KNs

    model_bench_data = []
    models_tps = list(itertools.product(args.models, args.tp_sizes))
    for model, tp_size in models_tps:
        Ms = args.batch_sizes
        KNs = model_shapes(model, tp_size)
        MKNs = []
        for m in Ms:
            for k, n in KNs:
                MKNs.append((m, k, n))

        data = run(args.dtype, MKNs, bench_kernels=args.kernels)
        model_bench_data.append(data)

    # Print all results
    for data, model_tp in zip(model_bench_data, models_tps):
        model, tp_size = model_tp
        print(f"== Results {args.dtype} {model}-TP{tp_size} ====")
        print_timers(data)

    timestamp = int(time.time())

    all_data = []
    for d in model_bench_data:
        all_data.extend(d)
    # pickle all data
    with open(f"model_bench-{args.dtype}-{timestamp}.pkl", "wb") as f:
        pkl.dump(all_data, f)
```
**EN:** `run_model_bench` implements a helper used by `w8a8_benchmarks.py`. It mainly works with `args` and relies on `print`, `enumerate`, `copy.deepcopy`, `KNs.append`, `list`, `itertools.product` plus iteration and context management to move data through this part of the benchmark pipeline.
**CN:** `run_model_bench` 负责实现 `w8a8_benchmarks.py` 使用的辅助逻辑。 它主要处理 `args`，并结合 `print`, `enumerate`, `copy.deepcopy`, `KNs.append`, `list`, `itertools.product` 以及 循环迭代、上下文管理 来完成这一段基准测试流程。

### Entry point (lines 296-372)
```python
if __name__ == "__main__":

    def to_torch_dtype(dt):
        if dt == "int8":
            return torch.int8
        if dt == "fp8":
            return torch.float8_e4m3fn
        raise ValueError("unsupported dtype")

    parser = FlexibleArgumentParser(
        description="""
Benchmark Cutlass GEMM.

    To run square GEMMs:
        python3 ./benchmarks/cutlass_benchmarks/w8a8_benchmarks.py --dtype fp8 square_bench --dim-start 128 --dim-end 512 --dim-increment 64
    
    To run constant N and K and sweep M:
        python3 ./benchmarks/cutlass_benchmarks/w8a8_benchmarks.py --dtype fp8 range_bench --dim-start 128 --dim-end 512 --dim-increment 64 --n-constant 16384 --k-constant 16384
    
    To run dimensions from a model:
        python3 ./benchmarks/cutlass_benchmarks/w8a8_benchmarks.py --dtype fp8 model_bench --models meta-llama/Llama-2-7b-hf --batch-sizes 16 --tp-sizes 1
    
    Output:
        - a .pkl file, that is a list of raw torch.benchmark.utils.Measurements for the pytorch and cutlass implementations for the various GEMMs.
            """,  # noqa: E501
        formatter_class=argparse.RawTextHelpFormatter,
    )

    parser.add_argument(
        "--dtype",
        type=to_torch_dtype,
        required=True,
        help="Available options are ['int8', 'fp8']",
    )
# ... omitted for brevity ...
    )
    model_parser.add_argument(
        "--batch-sizes", nargs="+", type=int, default=DEFAULT_BATCH_SIZES
    )
    model_parser.set_defaults(func=run_model_bench)

    args = parser.parse_args()
    args.func(args)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `ValueError`, `FlexibleArgumentParser`, `parser.add_argument`, `parser.add_subparsers`, `subparsers.add_parser`, `square_parser.add_argument` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `ValueError`, `FlexibleArgumentParser`, `parser.add_argument`, `parser.add_subparsers`, `subparsers.add_parser`, `square_parser.add_argument` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`, `copy`, `itertools`, `pickle`, `time`, `collections.abc`.
- **CN:** 标准库依赖：`argparse`, `copy`, `itertools`, `pickle`, `time`, `collections.abc`。
- **EN:** Third-party packages: `torch`, `torch.utils.benchmark`.
- **CN:** 第三方依赖：`torch`, `torch.utils.benchmark`。
- **EN:** Internal modules: `utils`, `weight_shapes`, `vllm`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.utils.argparse_utils`, `vllm.utils.math_utils`.
- **CN:** 内部模块：`utils`, `weight_shapes`, `vllm`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.utils.argparse_utils`, `vllm.utils.math_utils`。
