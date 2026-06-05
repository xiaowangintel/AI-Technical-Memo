# benchmark_fp8_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_fp8_gemm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, FP8 kernel experiments, GEMM kernel benchmarks, centered around `PROVIDER_CFGS`, `_quant_weight_fp8`, `build_fp8_runner`, `benchmark`. / 实现与基准测试编排、FP8 内核实验、GEMM 内核基准相关的逻辑，核心符号包括 `PROVIDER_CFGS`, `_quant_weight_fp8`, `build_fp8_runner`, `benchmark`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-12)
```python
import argparse
import copy
import itertools

import torch
from weight_shapes import WEIGHT_SHAPES

from vllm._custom_ops import cutlass_scaled_mm as vllm_scaled_mm
from vllm._custom_ops import scaled_fp8_quant as vllm_scaled_fp8_quant
from vllm.triton_utils import triton
```
**EN:** This block gathers standard-library helpers such as `argparse`, `copy`, `itertools`; third-party packages such as `torch`; project-local modules such as `weight_shapes`, `vllm._custom_ops`, `vllm.triton_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `copy`, `itertools`；第三方依赖，如 `torch`；项目内部模块，如 `weight_shapes`, `vllm._custom_ops`, `vllm.triton_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 14-42)
```python
PROVIDER_CFGS = {
    "torch-bf16": dict(enabled=True),
    "fp8-tensor-w-token-a": dict(
        w="tensor", a="token", no_a_quant=False, enabled=False
    ),
    "fp8-tensor-w-tensor-a": dict(
        w="tensor", a="tensor", no_a_quant=False, enabled=True
    ),
    "fp8-channel-w-token-a": dict(
        w="channel", a="token", no_a_quant=False, enabled=True
    ),
    "fp8-channel-w-tensor-a": dict(
        w="channel", a="tensor", no_a_quant=False, enabled=False
    ),
    "fp8-tensor-w-token-a-noquant": dict(
        w="tensor", a="token", no_a_quant=True, enabled=False
    ),
    "fp8-tensor-w-tensor-a-noquant": dict(
        w="tensor", a="tensor", no_a_quant=True, enabled=True
    ),
    "fp8-channel-w-token-a-noquant": dict(
        w="channel", a="token", no_a_quant=True, enabled=True
    ),
    "fp8-channel-w-tensor-a-noquant": dict(
        w="channel", a="tensor", no_a_quant=True, enabled=False
    ),
}

_enabled = [k for k, v in PROVIDER_CFGS.items() if v["enabled"]]
```
**EN:** This top-level block prepares shared state such as `PROVIDER_CFGS`, `_enabled`. It uses `dict`, `PROVIDER_CFGS.items` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `PROVIDER_CFGS`, `_enabled`。它借助 `dict`, `PROVIDER_CFGS.items` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `_quant_weight_fp8` (lines 45-51)
```python
def _quant_weight_fp8(b: torch.Tensor, w_type: str, device: str):
    if w_type == "tensor":
        scale_b = torch.ones(1, device=device, dtype=torch.float32)
        b_fp8, scale_b_fp8 = vllm_scaled_fp8_quant(b, scale_b)
    else:
        b_fp8, scale_b_fp8 = vllm_scaled_fp8_quant(b, use_per_token_if_dynamic=True)
    return b_fp8.t(), scale_b_fp8
```
**EN:** `_quant_weight_fp8` coordinates or measures FP8 kernel experiments. It mainly works with `b`, `w_type`, `device` and relies on `torch.ones`, `vllm_scaled_fp8_quant`, `b_fp8.t` plus branching to move data through this part of the benchmark pipeline.
**CN:** `_quant_weight_fp8` 负责协调或测量FP8 内核实验。 它主要处理 `b`, `w_type`, `device`，并结合 `torch.ones`, `vllm_scaled_fp8_quant`, `b_fp8.t` 以及 条件分支 来完成这一段基准测试流程。

### Function `build_fp8_runner` (lines 54-86)
```python
def build_fp8_runner(cfg, a, b, dtype, device):
    b_fp8, scale_b_fp8 = _quant_weight_fp8(b, cfg["w"], device)

    scale_a_const = (
        torch.ones(1, device=device, dtype=torch.float32)
        if cfg["a"] == "tensor"
        else None
    )

    if cfg["no_a_quant"]:
        if cfg["a"] == "tensor":
            a_fp8, scale_a_fp8 = vllm_scaled_fp8_quant(a, scale_a_const)
        else:
            a_fp8, scale_a_fp8 = vllm_scaled_fp8_quant(a, use_per_token_if_dynamic=True)

        def run():
            return vllm_scaled_mm(a_fp8, b_fp8, scale_a_fp8, scale_b_fp8, dtype)

        return run

    if cfg["a"] == "tensor":

        def run():
            a_fp8, scale_a_fp8 = vllm_scaled_fp8_quant(a, scale_a_const)
            return vllm_scaled_mm(a_fp8, b_fp8, scale_a_fp8, scale_b_fp8, dtype)

    else:

        def run():
            a_fp8, scale_a_fp8 = vllm_scaled_fp8_quant(a, use_per_token_if_dynamic=True)
            return vllm_scaled_mm(a_fp8, b_fp8, scale_a_fp8, scale_b_fp8, dtype)

    return run
```
**EN:** `build_fp8_runner` coordinates or measures FP8 kernel experiments. It mainly works with `cfg`, `a`, `b`, `dtype`, `device` and relies on `_quant_weight_fp8`, `torch.ones`, `vllm_scaled_fp8_quant`, `vllm_scaled_mm` plus branching to move data through this part of the benchmark pipeline.
**CN:** `build_fp8_runner` 负责协调或测量FP8 内核实验。 它主要处理 `cfg`, `a`, `b`, `dtype`, `device`，并结合 `_quant_weight_fp8`, `torch.ones`, `vllm_scaled_fp8_quant`, `vllm_scaled_mm` 以及 条件分支 来完成这一段基准测试流程。

### Function `benchmark` (lines 102-124)
```python
def benchmark(batch_size, provider, N, K):
    M = batch_size
    device = "cuda"
    dtype = torch.bfloat16

    a = torch.randn((M, K), device=device, dtype=dtype)
    b = torch.randn((N, K), device=device, dtype=dtype)

    quantiles = [0.5, 0.2, 0.8]

    if provider == "torch-bf16":
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: torch.nn.functional.linear(a, b), quantiles=quantiles
        )
    else:
        cfg = PROVIDER_CFGS[provider]
        run_quant = build_fp8_runner(cfg, a, b, dtype, device)
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: run_quant(), quantiles=quantiles
        )

    to_tflops = lambda t_ms: (2 * M * N * K) * 1e-12 / (t_ms * 1e-3)
    return to_tflops(ms), to_tflops(max_ms), to_tflops(min_ms)
```
**EN:** `benchmark` coordinates or measures benchmark orchestration. It mainly works with `batch_size`, `provider`, `N`, `K` and relies on `torch.randn`, `triton.testing.do_bench_cudagraph`, `torch.nn.functional.linear`, `build_fp8_runner`, `run_quant`, `to_tflops` plus branching to move data through this part of the benchmark pipeline.
**CN:** `benchmark` 负责协调或测量基准测试编排。 它主要处理 `batch_size`, `provider`, `N`, `K`，并结合 `torch.randn`, `triton.testing.do_bench_cudagraph`, `torch.nn.functional.linear`, `build_fp8_runner`, `run_quant`, `to_tflops` 以及 条件分支 来完成这一段基准测试流程。

### Function `prepare_shapes` (lines 127-134)
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

### Entry point (lines 137-159)
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--models",
        nargs="+",
        type=str,
        default=["meta-llama/Llama-3.1-8B-Instruct"],
        choices=list(WEIGHT_SHAPES.keys()),
    )
    parser.add_argument("--tp-sizes", nargs="+", type=int, default=[1])
    args = parser.parse_args()

    for K, N, model in prepare_shapes(args):
        print(f"{model}, N={N} K={K}, BF16 vs FP8 GEMMs TFLOP/s:")
        benchmark.run(
            print_data=True,
            show_plots=True,
            save_path=f"bench_fp8_res_n{N}_k{K}",
            N=N,
            K=K,
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
- **EN:** Result presentation: the module aggregates measurements and renders tables or plots.
- **CN:** 结果呈现：该模块会汇总测量结果并生成表格或图形。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`, `copy`, `itertools`.
- **CN:** 标准库依赖：`argparse`, `copy`, `itertools`。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: `weight_shapes`, `vllm._custom_ops`, `vllm.triton_utils`.
- **CN:** 内部模块：`weight_shapes`, `vllm._custom_ops`, `vllm.triton_utils`。
