# benchmark_nvfp4_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_nvfp4_gemm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, NVFP4 quantization benchmarks, GEMM kernel benchmarks, centered around `FLOAT4_E2M1_MAX`, `FLOAT8_E4M3_MAX`, `PROVIDER_CFGS`, `_quant_weight_nvfp4`. / 实现与基准测试编排、NVFP4 量化基准、GEMM 内核基准相关的逻辑，核心符号包括 `FLOAT4_E2M1_MAX`, `FLOAT8_E4M3_MAX`, `PROVIDER_CFGS`, `_quant_weight_nvfp4`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-14)
```python
import argparse
import copy
import itertools
import os

import torch
from weight_shapes import WEIGHT_SHAPES

from vllm import _custom_ops as ops
from vllm.platforms import current_platform
from vllm.scalar_type import scalar_types
from vllm.triton_utils import triton
```
**EN:** This block gathers standard-library helpers such as `argparse`, `copy`, `itertools`, `os`; third-party packages such as `torch`; project-local modules such as `weight_shapes`, `vllm`, `vllm.platforms`, `vllm.scalar_type`, `vllm.triton_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `copy`, `itertools`, `os`；第三方依赖，如 `torch`；项目内部模块，如 `weight_shapes`, `vllm`, `vllm.platforms`, `vllm.scalar_type`, `vllm.triton_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 16-50)
```python
if not current_platform.has_device_capability(100):
    raise RuntimeError("NVFP4 requires compute capability of 10.0 (Blackwell)")


FLOAT4_E2M1_MAX = scalar_types.float4_e2m1f.max()
FLOAT8_E4M3_MAX = torch.finfo(torch.float8_e4m3fn).max

PROVIDER_CFGS = {
    "torch-bf16": dict(enabled=True),
    "nvfp4": dict(no_a_quant=False, enabled=True),
    "nvfp4-noquant": dict(no_a_quant=True, enabled=True),
    "fbgemm-nvfp4": dict(fbgemm=True, no_a_quant=False, enabled=True),
    "fbgemm-nvfp4-noquant": dict(fbgemm=True, no_a_quant=True, enabled=True),
}

_needs_fbgemm = any(
    v.get("fbgemm", False) for v in PROVIDER_CFGS.values() if v.get("enabled", False)
)
if _needs_fbgemm:
    try:
        from fbgemm_gpu.experimental.gemm.triton_gemm.fp4_quantize import (
            triton_scale_nvfp4_quant,
        )
    except ImportError:
        print(
            "WARNING: FBGEMM providers are enabled but fbgemm_gpu is not installed. "
            "These providers will be skipped. Please install fbgemm_gpu with: "
            "'pip install fbgemm-gpu-genai' to run them."
        )
        # Disable FBGEMM providers so the benchmark can run.
        for cfg in PROVIDER_CFGS.values():
            if cfg.get("fbgemm"):
                cfg["enabled"] = False

_enabled = [k for k, v in PROVIDER_CFGS.items() if v["enabled"]]
```
**EN:** This top-level block prepares shared state such as `FLOAT4_E2M1_MAX`, `FLOAT8_E4M3_MAX`, `PROVIDER_CFGS`, `_needs_fbgemm`, `_enabled`. It uses `current_platform.has_device_capability`, `RuntimeError`, `scalar_types.float4_e2m1f.max`, `torch.finfo`, `dict`, `any` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `FLOAT4_E2M1_MAX`, `FLOAT8_E4M3_MAX`, `PROVIDER_CFGS`, `_needs_fbgemm`, `_enabled`。它借助 `current_platform.has_device_capability`, `RuntimeError`, `scalar_types.float4_e2m1f.max`, `torch.finfo`, `dict`, `any` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `_quant_weight_nvfp4` (lines 53-61)
```python
def _quant_weight_nvfp4(b: torch.Tensor, device: str, cfg):
    # Compute global scale for weight
    b_amax = torch.abs(b).max().to(torch.float32)
    b_global_scale = FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX / b_amax
    if "fbgemm" in cfg and cfg["fbgemm"]:
        b_fp4, scale_b_fp4 = triton_scale_nvfp4_quant(b, b_global_scale)
    else:
        b_fp4, scale_b_fp4 = ops.scaled_fp4_quant(b, b_global_scale)
    return b_fp4, scale_b_fp4, b_global_scale
```
**EN:** `_quant_weight_nvfp4` coordinates or measures NVFP4 quantization benchmarks. It mainly works with `b`, `device`, `cfg` and relies on `torch.abs.max.to`, `torch.abs.max`, `torch.abs`, `triton_scale_nvfp4_quant`, `ops.scaled_fp4_quant` plus branching to move data through this part of the benchmark pipeline.
**CN:** `_quant_weight_nvfp4` 负责协调或测量NVFP4 量化基准。 它主要处理 `b`, `device`, `cfg`，并结合 `torch.abs.max.to`, `torch.abs.max`, `torch.abs`, `triton_scale_nvfp4_quant`, `ops.scaled_fp4_quant` 以及 条件分支 来完成这一段基准测试流程。

### Function `build_nvfp4_runner` (lines 64-122)
```python
def build_nvfp4_runner(cfg, a, b, dtype, device):
    b_fp4, scale_b_fp4, b_global_scale = _quant_weight_nvfp4(b, device, cfg)

    # Compute global scale for activation
    # NOTE: This is generally provided ahead-of-time by the model checkpoint.
    a_amax = torch.abs(a).max().to(torch.float32)
    a_global_scale = FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX / a_amax

    # Alpha for the GEMM operation
    alpha = 1.0 / (a_global_scale * b_global_scale)
    if "fbgemm" in cfg and cfg["fbgemm"]:
        if cfg["no_a_quant"]:
            a_fp4, scale_a_fp4 = triton_scale_nvfp4_quant(a, a_global_scale)

            def run():
                return torch.ops.fbgemm.f4f4bf16(
                    a_fp4,
                    b_fp4,
                    scale_a_fp4,
                    scale_b_fp4,
                    global_scale=alpha,
                    use_mx=False,
                )

            return run
        else:

            def run():
                a_fp4, scale_a_fp4 = triton_scale_nvfp4_quant(a, a_global_scale)
                return torch.ops.fbgemm.f4f4bf16(
                    a_fp4,
                    b_fp4,
                    scale_a_fp4,
                    scale_b_fp4,
    # ... omitted for brevity ...
    # Quantize activation on-the-fly
    def run():
        a_fp4, scale_a_fp4 = ops.scaled_fp4_quant(a, a_global_scale)
        return ops.cutlass_scaled_fp4_mm(
            a_fp4, b_fp4, scale_a_fp4, scale_b_fp4, alpha, dtype
        )

    return run
```
**EN:** `build_nvfp4_runner` coordinates or measures NVFP4 quantization benchmarks. It mainly works with `cfg`, `a`, `b`, `dtype`, `device` and relies on `_quant_weight_nvfp4`, `torch.abs.max.to`, `torch.abs.max`, `torch.abs`, `triton_scale_nvfp4_quant`, `torch.ops.fbgemm.f4f4bf16` plus branching to move data through this part of the benchmark pipeline.
**CN:** `build_nvfp4_runner` 负责协调或测量NVFP4 量化基准。 它主要处理 `cfg`, `a`, `b`, `dtype`, `device`，并结合 `_quant_weight_nvfp4`, `torch.abs.max.to`, `torch.abs.max`, `torch.abs`, `triton_scale_nvfp4_quant`, `torch.ops.fbgemm.f4f4bf16` 以及 条件分支 来完成这一段基准测试流程。

### Function `benchmark` (lines 138-160)
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
        run_quant = build_nvfp4_runner(cfg, a, b, dtype, device)
        ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
            lambda: run_quant(), quantiles=quantiles
        )

    to_tflops = lambda t_ms: (2 * M * N * K) * 1e-12 / (t_ms * 1e-3)
    return to_tflops(ms), to_tflops(max_ms), to_tflops(min_ms)
```
**EN:** `benchmark` coordinates or measures benchmark orchestration. It mainly works with `batch_size`, `provider`, `N`, `K` and relies on `torch.randn`, `triton.testing.do_bench_cudagraph`, `torch.nn.functional.linear`, `build_nvfp4_runner`, `run_quant`, `to_tflops` plus branching to move data through this part of the benchmark pipeline.
**CN:** `benchmark` 负责协调或测量基准测试编排。 它主要处理 `batch_size`, `provider`, `N`, `K`，并结合 `torch.randn`, `triton.testing.do_bench_cudagraph`, `torch.nn.functional.linear`, `build_nvfp4_runner`, `run_quant`, `to_tflops` 以及 条件分支 来完成这一段基准测试流程。

### Function `prepare_shapes` (lines 163-170)
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

### Entry point (lines 173-198)
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
        print(f"{model}, N={N} K={K}, BF16 vs NVFP4 GEMMs TFLOP/s:")
        save_dir = f"bench_nvfp4_res_n{N}_k{K}"
        os.makedirs(save_dir, exist_ok=True)

        benchmark.run(
            print_data=True,
            show_plots=True,
            save_path=save_dir,
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
- **EN:** Standard library: `argparse`, `copy`, `itertools`, `os`.
- **CN:** 标准库依赖：`argparse`, `copy`, `itertools`, `os`。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: `weight_shapes`, `vllm`, `vllm.platforms`, `vllm.scalar_type`, `vllm.triton_utils`.
- **CN:** 内部模块：`weight_shapes`, `vllm`, `vllm.platforms`, `vllm.scalar_type`, `vllm.triton_utils`。
