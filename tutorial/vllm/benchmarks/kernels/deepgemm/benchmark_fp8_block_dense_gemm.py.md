# benchmark_fp8_block_dense_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/deepgemm/benchmark_fp8_block_dense_gemm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, FP8 kernel experiments, GEMM kernel benchmarks, centered around `benchmark_shape`, `format_table_row`, `print_table`, `format_speedup`. / 实现与基准测试编排、FP8 内核实验、GEMM 内核基准相关的逻辑，核心符号包括 `benchmark_shape`, `format_table_row`, `print_table`, `format_speedup`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-19)
```python
import time

import torch

from vllm import _custom_ops as ops
from vllm.model_executor.layers.quantization.utils.fp8_utils import (
    per_token_group_quant_fp8,
    w8a8_triton_block_scaled_mm,
)
from vllm.triton_utils import triton
from vllm.utils.deep_gemm import (
    calc_diff,
    fp8_gemm_nt,
    per_block_cast_to_fp8,
)
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block gathers standard-library helpers such as `time`; third-party packages such as `torch`; project-local modules such as `vllm`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.triton_utils`, `vllm.utils.deep_gemm`, `vllm.utils.torch_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `time`；第三方依赖，如 `torch`；项目内部模块，如 `vllm`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.triton_utils`, `vllm.utils.deep_gemm`, `vllm.utils.torch_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Function `benchmark_shape` (lines 22-183)
```python
def benchmark_shape(
    m: int,
    n: int,
    k: int,
    warmup: int = 100,
    repeat: int = 10000,
    verbose: bool = False,
) -> dict:
    """Benchmark all implementations for a specific (m, n, k) shape."""
    if verbose:
        print(f"\n=== Benchmarking shape: m={m}, n={n}, k={k} ===")

    # Create test tensors
    A = torch.randn((m, k), device="cuda", dtype=torch.bfloat16)
    B = torch.randn((n, k), device="cuda", dtype=torch.bfloat16)

    # Reference result in BF16
    torch.accelerator.synchronize()
    C_ref = A @ B.t()

    # Pre-quantize B for all implementations
    # (weights can be pre-quantized offline)
    B_deepgemm, B_scale_deepgemm = per_block_cast_to_fp8(B, [128, 128], use_ue8m0=True)
    B_vllm, B_scale_vllm = per_block_cast_to_fp8(B, [128, 128], use_ue8m0=True)

    # Block size configuration
    block_size = [128, 128]

    # Pre-quantize A for all implementations
    A_deepgemm, A_scale_deepgemm = per_token_group_quant_fp8(
        A, block_size[1], column_major_scales=True, tma_aligned_scales=True
    )
    C_deepgemm = torch.empty((m, n), device="cuda", dtype=torch.bfloat16)
    A_vllm, A_scale_vllm = per_token_group_quant_fp8(A, block_size[1])
    # ... omitted for brevity ...
    )
    if verbose:
        print(
            f"vLLM CUTLASS is {cutlass_vs_triton:.2f}x "
            f"{'faster' if cutlass_vs_triton > 1 else 'slower'} than vLLM Triton"
        )

    return benchmark_results
```
**EN:** `benchmark_shape` Benchmark all implementations for a specific (m, n, k) shape. It mainly works with `m`, `n`, `k`, `warmup`, `repeat`, ... and relies on `print`, `torch.randn`, `torch.accelerator.synchronize`, `B.t`, `per_block_cast_to_fp8`, `per_token_group_quant_fp8` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `benchmark_shape` 的职责是：Benchmark all implementations for a specific (m, n, k) shape。 它主要处理 `m`, `n`, `k`, `warmup`, `repeat`, ...，并结合 `print`, `torch.randn`, `torch.accelerator.synchronize`, `B.t`, `per_block_cast_to_fp8`, `per_token_group_quant_fp8` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `format_table_row` (lines 186-188)
```python
def format_table_row(values, widths):
    """Format a row with specified column widths."""
    return "| " + " | ".join(f"{val:{w}}" for val, w in zip(values, widths)) + " |"
```
**EN:** `format_table_row` Format a row with specified column widths. It mainly works with `values`, `widths` and relies on `join`, `zip` plus value production to move data through this part of the benchmark pipeline.
**CN:** `format_table_row` 的职责是：Format a row with specified column widths。 它主要处理 `values`, `widths`，并结合 `join`, `zip` 以及 结果返回 来完成这一段基准测试流程。

### Function `print_table` (lines 191-211)
```python
def print_table(headers, rows, title=None):
    """Print a table with headers and rows."""
    if title:
        print(f"\n{title}")

    # Calculate column widths based on headers and data
    widths = [
        max(len(str(h)), max(len(str(row[i])) for row in rows))
        for i, h in enumerate(headers)
    ]

    # Create separator line
    separator = "+-" + "-+-".join("-" * w for w in widths) + "-+"

    # Print table
    print(separator)
    print(format_table_row(headers, widths))
    print(separator)
    for row in rows:
        print(format_table_row(row, widths))
    print(separator)
```
**EN:** `print_table` Print a table with headers and rows. It mainly works with `headers`, `rows`, `title` and relies on `print`, `max`, `len`, `str`, `enumerate`, `join` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `print_table` 的职责是：Print a table with headers and rows。 它主要处理 `headers`, `rows`, `title`，并结合 `print`, `max`, `len`, `str`, `enumerate`, `join` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `format_speedup` (lines 214-216)
```python
def format_speedup(value):
    """Format speedup value with indicator if it's faster or slower."""
    return f"{value:.2f}x {'faster' if value > 1.0 else 'slower'}"
```
**EN:** `format_speedup` Format speedup value with indicator if it's faster or slower. It mainly works with `value` and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `format_speedup` 的职责是：Format speedup value with indicator if it's faster or slower。 它主要处理 `value`，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Function `run_benchmarks` (lines 219-430)
```python
def run_benchmarks(verbose: bool = False):
    """Run benchmarks for a set of common shapes."""
    print("===== STARTING FP8 GEMM BENCHMARK =====")

    # Make sure we're using the GPU
    if not torch.cuda.is_available():
        print("CUDA not available! Tests require GPU.")
        return

    # Print system information
    print(f"PyTorch version: {torch.__version__}")
    print(f"CUDA version: {torch.version.cuda}")
    print(f"Triton version: {triton.__version__}")
    print(f"Using device: {torch.cuda.get_device_name()}")

    # Enable TF32 for better performance
    torch.backends.cuda.matmul.allow_tf32 = True
    torch.backends.cudnn.allow_tf32 = True

    # Set seeds for reproducibility
    set_random_seed(42)
    # Define benchmark shapes (m, n, k)
    shapes = [
        (8, 4096, 7168),
        (8, 7168, 18432),
        (8, 18432, 7168),
        (64, 4096, 7168),
        (64, 7168, 18432),
        (64, 18432, 7168),
        (64, 24576, 1536),
        (64, 32768, 512),
        (64, 7168, 16384),
        (128, 4096, 7168),
        (128, 7168, 18432),
    # ... omitted for brevity ...
            avg_diff[impl] += result["implementations"][impl]["diff"]["Reference"]

    diff_headers = ["Implementation", "Avg Diff vs Reference"]
    diff_rows = []
    for impl in implementations:
        diff_rows.append([impl, f"{avg_diff[impl] / num_shapes:.6f}"])

    print_table(diff_headers, diff_rows)
```
**EN:** `run_benchmarks` Run benchmarks for a set of common shapes. It mainly works with `verbose` and relies on `print`, `torch.cuda.is_available`, `torch.cuda.get_device_name`, `set_random_seed`, `benchmark_shape`, `all_results.append` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `run_benchmarks` 的职责是：Run benchmarks for a set of common shapes。 它主要处理 `verbose`，并结合 `print`, `torch.cuda.is_available`, `torch.cuda.get_device_name`, `set_random_seed`, `benchmark_shape`, `all_results.append` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Entry point (lines 433-434)
```python
if __name__ == "__main__":
    run_benchmarks(verbose=False)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `run_benchmarks` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `run_benchmarks` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `time`.
- **CN:** 标准库依赖：`time`。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: `vllm`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.triton_utils`, `vllm.utils.deep_gemm`, `vllm.utils.torch_utils`.
- **CN:** 内部模块：`vllm`, `vllm.model_executor.layers.quantization.utils.fp8_utils`, `vllm.triton_utils`, `vllm.utils.deep_gemm`, `vllm.utils.torch_utils`。
