# benchmark_nvfp4_qutlass.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_nvfp4_qutlass.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, NVFP4 quantization benchmarks, centered around `PROVIDER_CFGS`, `get_hadamard_matrix`, `_quant_weight_nvfp4`, `build_nvfp4_runner`. / 实现与基准测试编排、NVFP4 量化基准相关的逻辑，核心符号包括 `PROVIDER_CFGS`, `get_hadamard_matrix`, `_quant_weight_nvfp4`, `build_nvfp4_runner`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 20-31)
```python
import argparse
import copy
import itertools

import torch
from compressed_tensors.transform.utils.hadamard import deterministic_hadamard_matrix
from weight_shapes import WEIGHT_SHAPES

from vllm import _custom_ops as ops  # use existing nvfp4 gemm in vllm
from vllm._custom_ops import fusedQuantizeNv
from vllm.model_executor.layers.quantization.qutlass_utils import to_blocked
from vllm.triton_utils import triton
```
**EN:** This block gathers standard-library helpers such as `argparse`, `copy`, `itertools`; third-party packages such as `torch`, `compressed_tensors.transform.utils.hadamard`; project-local modules such as `weight_shapes`, `vllm`, `vllm._custom_ops`, `vllm.model_executor.layers.quantization.qutlass_utils`, `vllm.triton_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `copy`, `itertools`；第三方依赖，如 `torch`, `compressed_tensors.transform.utils.hadamard`；项目内部模块，如 `weight_shapes`, `vllm`, `vllm._custom_ops`, `vllm.model_executor.layers.quantization.qutlass_utils`, `vllm.triton_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 33-39)
```python
PROVIDER_CFGS = {
    "torch-bf16": dict(enabled=True),
    "nvfp4": dict(no_a_quant=False, enabled=True),
    "nvfp4-noquant": dict(no_a_quant=True, enabled=True),
}

_enabled = [k for k, v in PROVIDER_CFGS.items() if v["enabled"]]
```
**EN:** This top-level block prepares shared state such as `PROVIDER_CFGS`, `_enabled`. It uses `dict`, `PROVIDER_CFGS.items` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `PROVIDER_CFGS`, `_enabled`。它借助 `dict`, `PROVIDER_CFGS.items` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `get_hadamard_matrix` (lines 42-46)
```python
def get_hadamard_matrix(group_size: int, dtype: torch.dtype, device: torch.device):
    return (
        deterministic_hadamard_matrix(group_size, dtype=dtype, device=device)
        * group_size**-0.5
    )
```
**EN:** `get_hadamard_matrix` implements a helper used by `benchmark_nvfp4_qutlass.py`. It mainly works with `group_size`, `dtype`, `device` and relies on `deterministic_hadamard_matrix` plus value production to move data through this part of the benchmark pipeline.
**CN:** `get_hadamard_matrix` 负责实现 `benchmark_nvfp4_qutlass.py` 使用的辅助逻辑。 它主要处理 `group_size`, `dtype`, `device`，并结合 `deterministic_hadamard_matrix` 以及 结果返回 来完成这一段基准测试流程。

### Function `_quant_weight_nvfp4` (lines 49-64)
```python
def _quant_weight_nvfp4(
    b: torch.Tensor,
    forward_hadamard_matrix: torch.Tensor,
    global_scale: torch.Tensor,
    device: str,
    M: int,
    N: int,
    K: int,
):
    weight_hf_e2m1, weight_hf_e8m0 = fusedQuantizeNv(
        b, forward_hadamard_matrix, global_scale
    )
    weight_hf_scale_block = to_blocked(weight_hf_e8m0, backend="triton").view(
        -1, K // 16
    )
    return weight_hf_e2m1, weight_hf_scale_block
```
**EN:** `_quant_weight_nvfp4` coordinates or measures NVFP4 quantization benchmarks. It mainly works with `b`, `forward_hadamard_matrix`, `global_scale`, `device`, `M`, ... and relies on `fusedQuantizeNv`, `to_blocked.view`, `to_blocked` plus value production to move data through this part of the benchmark pipeline.
**CN:** `_quant_weight_nvfp4` 负责协调或测量NVFP4 量化基准。 它主要处理 `b`, `forward_hadamard_matrix`, `global_scale`, `device`, `M`, ...，并结合 `fusedQuantizeNv`, `to_blocked.view`, `to_blocked` 以及 结果返回 来完成这一段基准测试流程。

### Function `build_nvfp4_runner` (lines 67-112)
```python
def build_nvfp4_runner(cfg, a, b, forward_hadamard_matrix, dtype, device, M, N, K):
    alpha = torch.tensor([1.0], device="cuda")
    global_scale = torch.tensor([1.0], device="cuda")
    weight_hf_e2m1, weight_hf_scale_block = _quant_weight_nvfp4(
        b, forward_hadamard_matrix, global_scale, device, M, N, K
    )

    if cfg["no_a_quant"]:
        # Pre-quantize activation
        input_hf_e2m1, input_hf_e8m0 = fusedQuantizeNv(
            a, forward_hadamard_matrix, global_scale
        )
        input_hf_scale_block = to_blocked(input_hf_e8m0, backend="triton").view(
            -1, K // 16
        )

        def run():
            return ops.cutlass_scaled_fp4_mm(
                input_hf_e2m1,
                weight_hf_e2m1,
                input_hf_scale_block,
                weight_hf_scale_block,
                alpha,
                torch.bfloat16,
            )

        return run

    # Quantize activation on-the-fly
    def run():
        input_hf_e2m1, input_hf_e8m0 = fusedQuantizeNv(
            a, forward_hadamard_matrix, global_scale
        )
        input_hf_scale_block = to_blocked(input_hf_e8m0, backend="triton").view(
            -1, K // 16
        )
        return ops.cutlass_scaled_fp4_mm(
            input_hf_e2m1,
            weight_hf_e2m1,
            input_hf_scale_block,
            weight_hf_scale_block,
            alpha,
            torch.bfloat16,
        )

    return run
```
**EN:** `build_nvfp4_runner` coordinates or measures NVFP4 quantization benchmarks. It mainly works with `cfg`, `a`, `b`, `forward_hadamard_matrix`, `dtype`, ... and relies on `torch.tensor`, `_quant_weight_nvfp4`, `fusedQuantizeNv`, `to_blocked.view`, `to_blocked`, `ops.cutlass_scaled_fp4_mm` plus branching to move data through this part of the benchmark pipeline.
**CN:** `build_nvfp4_runner` 负责协调或测量NVFP4 量化基准。 它主要处理 `cfg`, `a`, `b`, `forward_hadamard_matrix`, `dtype`, ...，并结合 `torch.tensor`, `_quant_weight_nvfp4`, `fusedQuantizeNv`, `to_blocked.view`, `to_blocked`, `ops.cutlass_scaled_fp4_mm` 以及 条件分支 来完成这一段基准测试流程。

### Function `benchmark` (lines 145-170)
```python
def benchmark(batch_size, provider, N, K, had_size):
    M = batch_size
    device = "cuda"
    dtype = torch.bfloat16

    a = torch.randn((M, K), device=device, dtype=dtype)
    b = torch.randn((N, K), device=device, dtype=dtype)
    forward_hadamard_matrix = get_hadamard_matrix(had_size, dtype, device)

    quantiles = [0.5, 0.2, 0.8]

    if provider == "torch-bf16":
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: torch.nn.functional.linear(a, b), rep=200, quantiles=quantiles
        )
    else:
        cfg = PROVIDER_CFGS[provider]
        run_quant = build_nvfp4_runner(
            cfg, a, b, forward_hadamard_matrix, dtype, device, M, N, K
        )
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: run_quant(), rep=200, quantiles=quantiles
        )

    to_tflops = lambda t_ms: (2 * M * N * K) * 1e-12 / (t_ms * 1e-3)
    return to_tflops(ms), to_tflops(max_ms), to_tflops(min_ms)
```
**EN:** `benchmark` coordinates or measures benchmark orchestration. It mainly works with `batch_size`, `provider`, `N`, `K`, `had_size` and relies on `torch.randn`, `get_hadamard_matrix`, `triton.testing.do_bench_cudagraph`, `torch.nn.functional.linear`, `build_nvfp4_runner`, `run_quant` plus branching to move data through this part of the benchmark pipeline.
**CN:** `benchmark` 负责协调或测量基准测试编排。 它主要处理 `batch_size`, `provider`, `N`, `K`, `had_size`，并结合 `torch.randn`, `get_hadamard_matrix`, `triton.testing.do_bench_cudagraph`, `torch.nn.functional.linear`, `build_nvfp4_runner`, `run_quant` 以及 条件分支 来完成这一段基准测试流程。

### Function `prepare_shapes` (lines 173-180)
```python
def prepare_shapes(args):
    out = []
    for model, tp_size in itertools.product(args.models, args.tp_sizes):
        for KN, tp_dim in copy.deepcopy(WEIGHT_SHAPES[model]):
            KN[tp_dim] //= tp_size
            KN.append(model)
            out.append(KN)
    return out
```
**EN:** `prepare_shapes` implements shape enumeration utilities. It mainly works with `args` and relies on `itertools.product`, `copy.deepcopy`, `KN.append`, `out.append` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `prepare_shapes` 负责实现形状枚举工具。 它主要处理 `args`，并结合 `itertools.product`, `copy.deepcopy`, `KN.append`, `out.append` 以及 循环迭代 来完成这一段基准测试流程。

### Entry point (lines 183-207)
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--models",
        nargs="+",
        type=str,
        default=["meta-llama/Llama-3.3-70B-Instruct"],
        choices=list(WEIGHT_SHAPES.keys()),
    )
    parser.add_argument("--tp-sizes", nargs="+", type=int, default=[1])
    args = parser.parse_args()

    for K, N, model in prepare_shapes(args):
        for had_size in [16, 32, 64, 128]:
            print(f"{model}, N={N} K={K}, HAD={had_size}, BF16 vs NVFP4 GEMMs TFLOP/s:")
            benchmark.run(
                print_data=True,
                show_plots=True,
                save_path=f"bench_nvfp4_res_n{N}_k{K}",
                N=N,
                K=K,
                had_size=had_size,
            )

    print("Benchmark finished!")
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `argparse.ArgumentParser`, `parser.add_argument`, `list`, `WEIGHT_SHAPES.keys`, `parser.parse_args`, `prepare_shapes` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `argparse.ArgumentParser`, `parser.add_argument`, `list`, `WEIGHT_SHAPES.keys`, `parser.parse_args`, `prepare_shapes` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Serving path benchmarking: the script interacts with HTTP services, servers, or request proxies.
- **CN:** 服务路径基准：脚本会与 HTTP 服务、服务器或请求代理交互。
- **EN:** Result presentation: the module aggregates measurements and renders tables or plots.
- **CN:** 结果呈现：该模块会汇总测量结果并生成表格或图形。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`, `copy`, `itertools`.
- **CN:** 标准库依赖：`argparse`, `copy`, `itertools`。
- **EN:** Third-party packages: `torch`, `compressed_tensors.transform.utils.hadamard`.
- **CN:** 第三方依赖：`torch`, `compressed_tensors.transform.utils.hadamard`。
- **EN:** Internal modules: `weight_shapes`, `vllm`, `vllm._custom_ops`, `vllm.model_executor.layers.quantization.qutlass_utils`, `vllm.triton_utils`.
- **CN:** 内部模块：`weight_shapes`, `vllm`, `vllm._custom_ops`, `vllm.model_executor.layers.quantization.qutlass_utils`, `vllm.triton_utils`。
