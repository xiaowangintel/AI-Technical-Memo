# benchmark_per_token_quant_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_per_token_quant_fp8.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, quantization benchmarks, FP8 kernel experiments, centered around `with_triton_mode`, `with_dyn_arg`, `bench_compile`, `calculate_diff`. / 实现与基准测试编排、量化基准、FP8 内核实验相关的逻辑，核心符号包括 `with_triton_mode`, `with_dyn_arg`, `bench_compile`, `calculate_diff`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-15)
```python
import itertools
from collections.abc import Callable
from unittest.mock import patch

import pandas as pd
import torch

from vllm.benchmarks.lib.utils import default_vllm_config
from vllm.model_executor.layers.quantization.input_quant_fp8 import QuantFP8
from vllm.model_executor.layers.quantization.utils.quant_utils import GroupShape
from vllm.triton_utils import triton
from vllm.utils.argparse_utils import FlexibleArgumentParser
from vllm.utils.torch_utils import STR_DTYPE_TO_TORCH_DTYPE
```
**EN:** This block gathers standard-library helpers such as `itertools`, `collections.abc`, `unittest.mock`; third-party packages such as `pandas`, `torch`; project-local modules such as `vllm.benchmarks.lib.utils`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.triton_utils`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `itertools`, `collections.abc`, `unittest.mock`；第三方依赖，如 `pandas`, `torch`；项目内部模块，如 `vllm.benchmarks.lib.utils`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.triton_utils`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Function `with_triton_mode` (lines 18-25)
```python
def with_triton_mode(fn):
    """Temporarily force the Triton fallback path"""

    def wrapped(*args, **kwargs):
        with patch("vllm.platforms.current_platform.is_cuda", return_value=False):
            return fn(*args, **kwargs)

    return wrapped
```
**EN:** `with_triton_mode` Temporarily force the Triton fallback path. It mainly works with `fn` and relies on `patch`, `fn` plus context management to move data through this part of the benchmark pipeline.
**CN:** `with_triton_mode` 的职责是：Temporarily force the Triton fallback path。 它主要处理 `fn`，并结合 `patch`, `fn` 以及 上下文管理 来完成这一段基准测试流程。

### Function `with_dyn_arg` (lines 29-34)
```python
def with_dyn_arg(fn: Callable, arg_index: int, dim_index: int):
    def inner(*args):
        torch._dynamo.mark_dynamic(args[arg_index], dim_index)
        return fn(*args)

    return inner
```
**EN:** `with_dyn_arg` implements a helper used by `benchmark_per_token_quant_fp8.py`. It mainly works with `fn`, `arg_index`, `dim_index` and relies on `torch._dynamo.mark_dynamic`, `fn` plus value production to move data through this part of the benchmark pipeline.
**CN:** `with_dyn_arg` 负责实现 `benchmark_per_token_quant_fp8.py` 使用的辅助逻辑。 它主要处理 `fn`, `arg_index`, `dim_index`，并结合 `torch._dynamo.mark_dynamic`, `fn` 以及 结果返回 来完成这一段基准测试流程。

### Function `bench_compile` (lines 37-42)
```python
def bench_compile(fn: Callable):
    # recompile for different shapes
    fwd = torch.compile(fn, fullgraph=True, dynamic=False)

    # First dim is explicitly dynamic to simulate vLLM usage
    return with_dyn_arg(fwd, 0, 0)
```
**EN:** `bench_compile` implements a helper used by `benchmark_per_token_quant_fp8.py`. It mainly works with `fn` and relies on `torch.compile`, `with_dyn_arg` plus value production to move data through this part of the benchmark pipeline.
**CN:** `bench_compile` 负责实现 `benchmark_per_token_quant_fp8.py` 使用的辅助逻辑。 它主要处理 `fn`，并结合 `torch.compile`, `with_dyn_arg` 以及 结果返回 来完成这一段基准测试流程。

### Top-level setup (lines 45-45)
```python
torch._dynamo.config.recompile_limit = 8888
```
**EN:** This top-level block prepares shared state such as `torch`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `torch`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `calculate_diff` (lines 48-82)
```python
def calculate_diff(
    batch_size: int,
    hidden_size: int,
    group_shape: GroupShape,
    dtype: torch.dtype,
):
    """Calculate the difference between Inductor and CUDA implementations."""
    device = torch.device("cuda")
    x = torch.randn((batch_size, hidden_size), dtype=dtype, device=device)

    quant_fp8 = QuantFP8(False, group_shape, column_major_scales=False)

    torch_out, torch_scale = bench_compile(quant_fp8.forward_native)(x)
    torch_eager_out, torch_eager_scale = quant_fp8.forward_native(x)
    cuda_out, cuda_scale = quant_fp8.forward_cuda(x)

    try:
        torch.testing.assert_close(
            cuda_out.to(torch.float32),
            torch_out.to(torch.float32),
            rtol=1e-3,
            atol=1e-5,
        )
        torch.testing.assert_close(cuda_scale, torch_scale, rtol=1e-3, atol=1e-5)
        torch.testing.assert_close(
            cuda_out.to(torch.float32),
            torch_eager_out.to(torch.float32),
            rtol=1e-3,
            atol=1e-5,
        )
        torch.testing.assert_close(cuda_scale, torch_eager_scale, rtol=1e-3, atol=1e-5)
        print("✅ All implementations match")
    except AssertionError as e:
        print("❌ Implementations differ")
        print(e)
```
**EN:** `calculate_diff` Calculate the difference between Inductor and CUDA implementations. It mainly works with `batch_size`, `hidden_size`, `group_shape`, `dtype` and relies on `torch.device`, `torch.randn`, `QuantFP8`, `bench_compile`, `quant_fp8.forward_native`, `quant_fp8.forward_cuda` plus error handling to move data through this part of the benchmark pipeline.
**CN:** `calculate_diff` 的职责是：Calculate the difference between Inductor and CUDA implementations。 它主要处理 `batch_size`, `hidden_size`, `group_shape`, `dtype`，并结合 `torch.device`, `torch.randn`, `QuantFP8`, `bench_compile`, `quant_fp8.forward_native`, `quant_fp8.forward_cuda` 以及 异常处理 来完成这一段基准测试流程。

### Top-level setup (lines 85-85)
```python
configs = []
```
**EN:** This top-level block prepares shared state such as `configs`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `configs`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `benchmark_quantization` (lines 89-117)
```python
def benchmark_quantization(
    batch_size,
    hidden_size,
    provider,
    group_shape: GroupShape,
    col_major: bool,
    dtype: torch.dtype,
):
    device = torch.device("cuda")

    x = torch.randn(batch_size, hidden_size, device=device, dtype=dtype)

    quantiles = [0.5, 0.2, 0.8]
    quant_fp8 = QuantFP8(False, group_shape, column_major_scales=col_major)

    if provider == "torch":
        fn = lambda: bench_compile(quant_fp8.forward_native)(x.clone())
    elif provider == "cuda":
        fn = lambda: quant_fp8.forward_cuda(x.clone())
    elif provider == "triton":
        if not group_shape.is_per_group():
            # Triton only supported for per-group
            return 0, 0, 0

        fn = lambda: with_triton_mode(quant_fp8.forward_cuda)(x.clone())

    ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(fn, quantiles=quantiles)

    return 1000 * ms, 1000 * max_ms, 1000 * min_ms
```
**EN:** `benchmark_quantization` coordinates or measures benchmark orchestration. It mainly works with `batch_size`, `hidden_size`, `provider`, `group_shape`, `col_major`, ... and relies on `torch.device`, `torch.randn`, `QuantFP8`, `bench_compile`, `x.clone`, `quant_fp8.forward_cuda` plus branching to move data through this part of the benchmark pipeline.
**CN:** `benchmark_quantization` 负责协调或测量基准测试编排。 它主要处理 `batch_size`, `hidden_size`, `provider`, `group_shape`, `col_major`, ...，并结合 `torch.device`, `torch.randn`, `QuantFP8`, `bench_compile`, `x.clone`, `quant_fp8.forward_cuda` 以及 条件分支 来完成这一段基准测试流程。

### Function `compute_geomean_speedups` (lines 121-156)
```python
def compute_geomean_speedups(
    df: pd.DataFrame,
    baseline_col: str,
    speedup_cols: list[str],
    groupby_cols: list[str] | None = None,
) -> pd.DataFrame:
    """
    Compute geometric mean speedups over a baseline column.

    Args:
        df: Input dataframe
        baseline_col: Column to use as baseline
        speedup_cols: Columns to compute speedups for
        groupby_cols: Columns to group by. If None, compute over entire df.

    Returns:
        pd.DataFrame with geometric mean speedups
    """
    from scipy.stats import gmean

    def geo_speedup(group: pd.DataFrame) -> pd.Series:
        ratios = {
            col: (group[baseline_col] / group[col]).values for col in speedup_cols
        }
        return pd.Series({col: gmean(vals) for col, vals in ratios.items()})

    if groupby_cols is None:
        result = geo_speedup(df).to_frame().T
    else:
        result = (
            df.groupby(groupby_cols)
            .apply(geo_speedup, include_groups=False)
            .reset_index()
        )

    return result
```
**EN:** `compute_geomean_speedups` Compute geometric mean speedups over a baseline column. It mainly works with `df`, `baseline_col`, `speedup_cols`, `groupby_cols` and relies on `pd.Series`, `gmean`, `ratios.items`, `geo_speedup.to_frame`, `geo_speedup`, `df.groupby.apply.reset_index` plus branching to move data through this part of the benchmark pipeline.
**CN:** `compute_geomean_speedups` 的职责是：Compute geometric mean speedups over a baseline column。 它主要处理 `df`, `baseline_col`, `speedup_cols`, `groupby_cols`，并结合 `pd.Series`, `gmean`, `ratios.items`, `geo_speedup.to_frame`, `geo_speedup`, `df.groupby.apply.reset_index` 以及 条件分支 来完成这一段基准测试流程。

### Entry point (lines 159-272)
```python
if __name__ == "__main__":
    parser = FlexibleArgumentParser(
        description="Benchmark the various implementations of QuantFP8 (dynamic-only)"
    )
    parser.add_argument("-c", "--check", action="store_true")
    parser.add_argument(
        "--dtype", type=str, choices=["half", "bfloat16", "float"], default="bfloat16"
    )
    parser.add_argument(
        "--hidden-sizes",
        type=int,
        nargs="+",
        default=[896, 1024, 2048, 4096, 7168],
        help="Hidden sizes to benchmark",
    )
    parser.add_argument(
        "--batch-sizes",
        type=int,
        nargs="+",
        default=[1, 16, 128, 512, 1024],
        help="Batch sizes to benchmark",
    )
    parser.add_argument(
        "--group-sizes",
        type=int,
        nargs="+",
        default=None,
        help="Group sizes for GroupShape(1,N) to benchmark. "
        "Use 0 for PER_TENSOR, -1 for PER_TOKEN (default: 0,-1,64,128)",
    )
    parser.add_argument(
        "--no-column-major",
        action="store_true",
        help="Disable column-major scales testing",
    # ... omitted for brevity ...
        df,
        baseline_col="Torch (Compiled)",
        speedup_cols=["CUDA", "Triton"],
        groupby_cols=["col_major", "group_shape"],
    )

    print("Speedup over Torch (Compiled)")
    print(geo_table_grouped.to_string(index=False))
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `group_shapes.append`, `GroupShape`, `itertools.product` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `group_shapes.append`, `GroupShape`, `itertools.product` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Result presentation: the module aggregates measurements and renders tables or plots.
- **CN:** 结果呈现：该模块会汇总测量结果并生成表格或图形。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `itertools`, `collections.abc`, `unittest.mock`.
- **CN:** 标准库依赖：`itertools`, `collections.abc`, `unittest.mock`。
- **EN:** Third-party packages: `pandas`, `torch`.
- **CN:** 第三方依赖：`pandas`, `torch`。
- **EN:** Internal modules: `vllm.benchmarks.lib.utils`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.triton_utils`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`.
- **CN:** 内部模块：`vllm.benchmarks.lib.utils`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.triton_utils`, `vllm.utils.argparse_utils`, `vllm.utils.torch_utils`。
