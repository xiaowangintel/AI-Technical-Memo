# benchmark_block_fp8_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_block_fp8_gemm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, FP8 kernel experiments, GEMM kernel benchmarks, centered around `DEEPSEEK_V3_SHAPES`, `build_w8a8_block_fp8_runner`, `benchmark_tflops`. / 实现与基准测试编排、FP8 内核实验、GEMM 内核基准相关的逻辑，核心符号包括 `DEEPSEEK_V3_SHAPES`, `build_w8a8_block_fp8_runner`, `benchmark_tflops`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-4)
```python
import os
```
**EN:** This block gathers standard-library helpers such as `os`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `os`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 7-7)
```python
os.environ["VLLM_USE_DEEP_GEMM"] = "0"
```
**EN:** This top-level block prepares shared state such as `os`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `os`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Additional imports (lines 9-9)
```python
import torch
```
**EN:** This block gathers third-party packages such as `torch`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了第三方依赖，如 `torch`。这些依赖构成了后续基准测试逻辑的基础。

### Additional imports (lines 11-11)
```python
from vllm.benchmarks.lib.utils import default_vllm_config
```
**EN:** This block gathers project-local modules such as `vllm.benchmarks.lib.utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了项目内部模块，如 `vllm.benchmarks.lib.utils`。这些依赖构成了后续基准测试逻辑的基础。

### Additional imports (lines 12-14)
```python
from vllm.model_executor.kernels.linear import (
    init_fp8_linear_kernel,
)
```
**EN:** This block gathers project-local modules such as `vllm.model_executor.kernels.linear`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了项目内部模块，如 `vllm.model_executor.kernels.linear`。这些依赖构成了后续基准测试逻辑的基础。

### Additional imports (lines 15-18)
```python
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    GroupShape,
    create_fp8_quant_key,
)
```
**EN:** This block gathers project-local modules such as `vllm.model_executor.layers.quantization.utils.quant_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了项目内部模块，如 `vllm.model_executor.layers.quantization.utils.quant_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Additional imports (lines 19-21)
```python
from vllm.model_executor.layers.quantization.utils.w8a8_utils import (
    CUTLASS_BLOCK_FP8_SUPPORTED,
)
```
**EN:** This block gathers project-local modules such as `vllm.model_executor.layers.quantization.utils.w8a8_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了项目内部模块，如 `vllm.model_executor.layers.quantization.utils.w8a8_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Additional imports (lines 22-22)
```python
from vllm.platforms import current_platform
```
**EN:** This block gathers project-local modules such as `vllm.platforms`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了项目内部模块，如 `vllm.platforms`。这些依赖构成了后续基准测试逻辑的基础。

### Additional imports (lines 23-23)
```python
from vllm.triton_utils import triton as vllm_triton
```
**EN:** This block gathers project-local modules such as `vllm.triton_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了项目内部模块，如 `vllm.triton_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 25-42)
```python
assert current_platform.is_cuda(), (
    "Only support benchmarking w8a8 block fp8 kernel on CUDA device."
)

# DeepSeek-V3 weight shapes
DEEPSEEK_V3_SHAPES = [
    (512 + 64, 7168),
    (2112, 7168),
    ((128 + 64) * 128, 7168),
    (128 * (128 + 128), 512),
    (7168, 16384),
    (7168, 18432),
    (18432 * 2, 7168),
    (24576, 1536),
    (12288, 7168),
    (4096, 7168),
    (7168, 2048),
]
```
**EN:** This top-level block prepares shared state such as `DEEPSEEK_V3_SHAPES`. It uses `current_platform.is_cuda` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `DEEPSEEK_V3_SHAPES`。它借助 `current_platform.is_cuda` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `build_w8a8_block_fp8_runner` (lines 46-94)
```python
def build_w8a8_block_fp8_runner(M, N, K, block_size, device, use_cutlass):
    """Build runner function for w8a8 block fp8 matmul."""
    factor_for_scale = 1e-2

    fp8_info = torch.finfo(torch.float8_e4m3fn)
    fp8_max, fp8_min = fp8_info.max, fp8_info.min

    # Create random input tensor (bfloat16, will be quantized by W8A8BlockFp8LinearOp)
    A_ref = (torch.rand(M, K, dtype=torch.bfloat16, device=device) - 0.5) * 2 * fp8_max

    # Create quantized weight tensor
    B_ref = (torch.rand(N, K, dtype=torch.bfloat16, device=device) - 0.5) * 2 * fp8_max
    B = B_ref.clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn)

    # Create weight scales
    block_n, block_k = block_size[0], block_size[1]
    n_tiles = (N + block_n - 1) // block_n
    k_tiles = (K + block_k - 1) // block_k

    Bs = (
        torch.rand(n_tiles, k_tiles, dtype=torch.float32, device=device)
        * factor_for_scale
    )

    # Create W8A8BlockFp8LinearOp instance
    weight_group_shape = GroupShape(block_n, block_k)
    act_quant_group_shape = GroupShape(1, block_k)  # Per-token, per-group quantization

    linear_op = init_fp8_linear_kernel(
        weight_quant_key=create_fp8_quant_key(
            static=True, group_shape=weight_group_shape
        ),
        activation_quant_key=create_fp8_quant_key(
            static=False, group_shape=act_quant_group_shape
    # ... omitted for brevity ...
            input=A_ref,
            weight=B,
            weight_scale=Bs,
            input_scale=None,
            bias=None,
        )

    return run
```
**EN:** `build_w8a8_block_fp8_runner` Build runner function for w8a8 block fp8 matmul. It mainly works with `M`, `N`, `K`, `block_size`, `device`, ... and relies on `torch.finfo`, `torch.rand`, `B_ref.clamp.to`, `B_ref.clamp`, `GroupShape`, `init_fp8_linear_kernel` plus value production to move data through this part of the benchmark pipeline.
**CN:** `build_w8a8_block_fp8_runner` 的职责是：Build runner function for w8a8 block fp8 matmul。 它主要处理 `M`, `N`, `K`, `block_size`, `device`, ...，并结合 `torch.finfo`, `torch.rand`, `B_ref.clamp.to`, `B_ref.clamp`, `GroupShape`, `init_fp8_linear_kernel` 以及 结果返回 来完成这一段基准测试流程。

### Top-level setup (lines 98-102)
```python
available_providers = ["torch-bf16", "w8a8-block-fp8-triton"]
plot_title = "BF16 vs W8A8 Block FP8 GEMMs"

if CUTLASS_BLOCK_FP8_SUPPORTED:
    available_providers.append("w8a8-block-fp8-cutlass")
```
**EN:** This top-level block prepares shared state such as `available_providers`, `plot_title`. It uses `available_providers.append` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `available_providers`, `plot_title`。它借助 `available_providers.append` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `benchmark_tflops` (lines 118-148)
```python
def benchmark_tflops(batch_size, provider, N, K, block_size=(128, 128)):
    M = batch_size
    device = "cuda"

    quantiles = [0.5, 0.2, 0.8]

    if provider == "torch-bf16":
        a = torch.randn((M, K), device=device, dtype=torch.bfloat16)
        b = torch.randn((N, K), device=device, dtype=torch.bfloat16)
        ms, min_ms, max_ms = vllm_triton.testing.do_bench_cudagraph(
            lambda: torch.nn.functional.linear(a, b), quantiles=quantiles
        )
    elif provider == "w8a8-block-fp8-triton":
        run_w8a8_triton = build_w8a8_block_fp8_runner(
            M, N, K, block_size, device, use_cutlass=False
        )
        ms, min_ms, max_ms = vllm_triton.testing.do_bench_cudagraph(
            lambda: run_w8a8_triton(), quantiles=quantiles
        )
    elif provider == "w8a8-block-fp8-cutlass":
        run_w8a8_cutlass = build_w8a8_block_fp8_runner(
            M, N, K, block_size, device, use_cutlass=True
        )
        ms, min_ms, max_ms = vllm_triton.testing.do_bench_cudagraph(
            lambda: run_w8a8_cutlass(), quantiles=quantiles
        )
    else:
        raise ValueError(f"Unknown provider: {provider}")

    to_tflops = lambda t_ms: (2 * M * N * K) * 1e-12 / (t_ms * 1e-3)
    return to_tflops(ms), to_tflops(max_ms), to_tflops(min_ms)
```
**EN:** `benchmark_tflops` coordinates or measures benchmark orchestration. It mainly works with `batch_size`, `provider`, `N`, `K`, `block_size` and relies on `torch.randn`, `vllm_triton.testing.do_bench_cudagraph`, `torch.nn.functional.linear`, `build_w8a8_block_fp8_runner`, `run_w8a8_triton`, `run_w8a8_cutlass` plus branching to move data through this part of the benchmark pipeline.
**CN:** `benchmark_tflops` 负责协调或测量基准测试编排。 它主要处理 `batch_size`, `provider`, `N`, `K`, `block_size`，并结合 `torch.randn`, `vllm_triton.testing.do_bench_cudagraph`, `torch.nn.functional.linear`, `build_w8a8_block_fp8_runner`, `run_w8a8_triton`, `run_w8a8_cutlass` 以及 条件分支 来完成这一段基准测试流程。

### Entry point (lines 151-167)
```python
if __name__ == "__main__":
    block_size = (128, 128)

    for N, K in DEEPSEEK_V3_SHAPES:
        print(f"\nBenchmarking DeepSeek-V3, N={N} K={K}")

        print(f"TFLOP/s comparison (block_size={block_size}):")
        benchmark_tflops.run(
            print_data=True,
            # show_plots=False,
            # save_path=f"bench_w8a8_block_fp8_tflops_n{N}_k{K}",
            N=N,
            K=K,
            block_size=block_size,
        )

    print("\nBenchmark finished!")
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `print`, `benchmark_tflops.run` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `print`, `benchmark_tflops.run` 等调用串起来，使模块能够从命令行直接启动。

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
- **EN:** Standard library: `os`.
- **CN:** 标准库依赖：`os`。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: `vllm.benchmarks.lib.utils`, `vllm.model_executor.kernels.linear`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, `vllm.platforms`, `vllm.triton_utils`.
- **CN:** 内部模块：`vllm.benchmarks.lib.utils`, `vllm.model_executor.kernels.linear`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.layers.quantization.utils.w8a8_utils`, `vllm.platforms`, `vllm.triton_utils`。
