# benchmark_nvfp4_quant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_nvfp4_quant.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, NVFP4 quantization benchmarks, quantization benchmarks, centered around `FLOAT4_E2M1_MAX`, `FLOAT8_E4M3_MAX`, `PROVIDER_CFGS`, `compute_global_scale`. / 实现与基准测试编排、NVFP4 量化基准、量化基准相关的逻辑，核心符号包括 `FLOAT4_E2M1_MAX`, `FLOAT8_E4M3_MAX`, `PROVIDER_CFGS`, `compute_global_scale`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-14)
```python
import argparse
import copy
import itertools

import torch
from weight_shapes import WEIGHT_SHAPES

from vllm import _custom_ops as ops
from vllm.platforms import current_platform
from vllm.scalar_type import scalar_types
from vllm.triton_utils import triton
from vllm.utils.flashinfer import flashinfer_fp4_quantize
```
**EN:** This block gathers standard-library helpers such as `argparse`, `copy`, `itertools`; third-party packages such as `torch`; project-local modules such as `weight_shapes`, `vllm`, `vllm.platforms`, `vllm.scalar_type`, `vllm.triton_utils`, `vllm.utils.flashinfer`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `copy`, `itertools`；第三方依赖，如 `torch`；项目内部模块，如 `weight_shapes`, `vllm`, `vllm.platforms`, `vllm.scalar_type`, `vllm.triton_utils`, `vllm.utils.flashinfer`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 16-31)
```python
if not current_platform.has_device_capability(100):
    raise RuntimeError("NVFP4 requires compute capability of 10.0 (Blackwell)")

FLOAT4_E2M1_MAX = scalar_types.float4_e2m1f.max()
FLOAT8_E4M3_MAX = torch.finfo(torch.float8_e4m3fn).max

PROVIDER_CFGS = {
    "vllm": dict(backend="vllm", is_sf_swizzled_layout=False, enabled=True),
    "vllm-swizzle": dict(backend="vllm", is_sf_swizzled_layout=True, enabled=True),
    "flashinfer": dict(backend="flashinfer", is_sf_swizzled_layout=False, enabled=True),
    "flashinfer-swizzle": dict(
        backend="flashinfer", is_sf_swizzled_layout=True, enabled=True
    ),
}

_enabled = [k for k, v in PROVIDER_CFGS.items() if v["enabled"]]
```
**EN:** This top-level block prepares shared state such as `FLOAT4_E2M1_MAX`, `FLOAT8_E4M3_MAX`, `PROVIDER_CFGS`, `_enabled`. It uses `current_platform.has_device_capability`, `RuntimeError`, `scalar_types.float4_e2m1f.max`, `torch.finfo`, `dict`, `PROVIDER_CFGS.items` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `FLOAT4_E2M1_MAX`, `FLOAT8_E4M3_MAX`, `PROVIDER_CFGS`, `_enabled`。它借助 `current_platform.has_device_capability`, `RuntimeError`, `scalar_types.float4_e2m1f.max`, `torch.finfo`, `dict`, `PROVIDER_CFGS.items` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `compute_global_scale` (lines 34-37)
```python
def compute_global_scale(tensor: torch.Tensor) -> torch.Tensor:
    """Compute global scale for FP4 quantization."""
    amax = torch.abs(tensor).max().to(torch.float32)
    return FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX / amax
```
**EN:** `compute_global_scale` Compute global scale for FP4 quantization. It mainly works with `tensor` and relies on `torch.abs.max.to`, `torch.abs.max`, `torch.abs` plus value production to move data through this part of the benchmark pipeline.
**CN:** `compute_global_scale` 的职责是：Compute global scale for FP4 quantization。 它主要处理 `tensor`，并结合 `torch.abs.max.to`, `torch.abs.max`, `torch.abs` 以及 结果返回 来完成这一段基准测试流程。

### Function `benchmark` (lines 53-103)
```python
def benchmark(batch_size, provider, N, K):
    M = batch_size
    device = "cuda"
    dtype = torch.bfloat16

    # Create input tensor
    a = torch.randn((M, K), device=device, dtype=dtype)

    # Compute global scale for activation
    a_global_scale = compute_global_scale(a)

    quantiles = [0.5, 0.2, 0.8]

    cfg = PROVIDER_CFGS[provider]

    if cfg["backend"] == "vllm":
        # vLLM's FP4 quantization
        if cfg["is_sf_swizzled_layout"]:
            ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
                lambda: ops.scaled_fp4_quant(
                    a, a_global_scale, is_sf_swizzled_layout=True
                ),
                quantiles=quantiles,
            )
        else:
            ms, min_ms, max_ms = triton.testing.do_bench_cudagraph(
                lambda: ops.scaled_fp4_quant(
                    a, a_global_scale, is_sf_swizzled_layout=False
                ),
                quantiles=quantiles,
            )
    elif cfg["backend"] == "flashinfer":
        # FlashInfer's FP4 quantization
        if cfg["is_sf_swizzled_layout"]:
    # ... omitted for brevity ...
                    a, a_global_scale, is_sf_swizzled_layout=False
                ),
                quantiles=quantiles,
            )

    # Convert ms to us for better readability at small batch sizes
    to_us = lambda t_ms: t_ms * 1000
    return to_us(ms), to_us(max_ms), to_us(min_ms)
```
**EN:** `benchmark` coordinates or measures benchmark orchestration. It mainly works with `batch_size`, `provider`, `N`, `K` and relies on `torch.randn`, `compute_global_scale`, `triton.testing.do_bench_cudagraph`, `ops.scaled_fp4_quant`, `flashinfer_fp4_quantize`, `to_us` plus branching to move data through this part of the benchmark pipeline.
**CN:** `benchmark` 负责协调或测量基准测试编排。 它主要处理 `batch_size`, `provider`, `N`, `K`，并结合 `torch.randn`, `compute_global_scale`, `triton.testing.do_bench_cudagraph`, `ops.scaled_fp4_quant`, `flashinfer_fp4_quantize`, `to_us` 以及 条件分支 来完成这一段基准测试流程。

### Function `prepare_shapes` (lines 106-113)
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

### Function `_test_accuracy_once` (lines 116-149)
```python
def _test_accuracy_once(
    M: int, K: int, dtype: torch.dtype, device: str, is_sf_swizzled_layout: bool
):
    """Test accuracy between vLLM and FlashInfer FP4 quantization."""
    # Create input tensor
    a = torch.randn((M, K), device=device, dtype=dtype)

    # Compute global scale
    a_global_scale = compute_global_scale(a)

    # vLLM quantization
    vllm_fp4, vllm_scale = ops.scaled_fp4_quant(
        a, a_global_scale, is_sf_swizzled_layout=is_sf_swizzled_layout
    )

    # FlashInfer quantization (with swizzled layout to match vLLM's output)
    flashinfer_fp4, flashinfer_scale = flashinfer_fp4_quantize(
        a, a_global_scale, is_sf_swizzled_layout=is_sf_swizzled_layout
    )
    flashinfer_scale = flashinfer_scale.view(torch.float8_e4m3fn)

    # Compare outputs
    torch.testing.assert_close(
        vllm_fp4,
        flashinfer_fp4,
    )
    # Compare scales
    torch.testing.assert_close(
        vllm_scale,
        flashinfer_scale,
    )
    print(
        f"M={M}, K={K}, dtype={dtype}, is_sf_swizzled_layout={is_sf_swizzled_layout}: PASSED"  # noqa: E501
    )
```
**EN:** `_test_accuracy_once` Test accuracy between vLLM and FlashInfer FP4 quantization. It mainly works with `M`, `K`, `dtype`, `device`, `is_sf_swizzled_layout` and relies on `torch.randn`, `compute_global_scale`, `ops.scaled_fp4_quant`, `flashinfer_fp4_quantize`, `flashinfer_scale.view`, `torch.testing.assert_close` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `_test_accuracy_once` 的职责是：Test accuracy between vLLM and FlashInfer FP4 quantization。 它主要处理 `M`, `K`, `dtype`, `device`, `is_sf_swizzled_layout`，并结合 `torch.randn`, `compute_global_scale`, `ops.scaled_fp4_quant`, `flashinfer_fp4_quantize`, `flashinfer_scale.view`, `torch.testing.assert_close` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Function `test_accuracy` (lines 152-170)
```python
def test_accuracy():
    """Run accuracy tests across various shapes."""
    print("\n" + "=" * 60)
    print("Running accuracy tests: vLLM vs FlashInfer")
    print("=" * 60)

    device = "cuda"
    dtype = torch.bfloat16

    # Test various batch sizes and hidden dimensions
    Ms = [1, 1024]
    Ks = [4096]

    for is_sf_swizzled_layout in [True, False]:
        for M in Ms:
            for K in Ks:
                _test_accuracy_once(M, K, dtype, device, is_sf_swizzled_layout)

    print("\nAll accuracy tests passed!")
```
**EN:** `test_accuracy` Run accuracy tests across various shapes. It mainly works with no explicit parameters and relies on `print`, `_test_accuracy_once` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `test_accuracy` 的职责是：Run accuracy tests across various shapes。 它主要处理 无显式参数，并结合 `print`, `_test_accuracy_once` 以及 循环迭代 来完成这一段基准测试流程。

### Entry point (lines 173-210)
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Benchmark NVFP4 quantization: vLLM vs FlashInfer"
    )
    parser.add_argument(
        "--models",
        nargs="+",
        type=str,
        default=["meta-llama/Llama-3.3-70B-Instruct"],
        choices=list(WEIGHT_SHAPES.keys()),
    )
    parser.add_argument("--tp-sizes", nargs="+", type=int, default=[1])
    parser.add_argument(
        "--save-path",
        type=str,
        default=None,
        help="Path to save benchmark results",
    )
    parser.add_argument(
        "--accuracy",
        action="store_true",
        help="Run accuracy tests",
    )
    args = parser.parse_args()

    if args.accuracy:
        test_accuracy()

    for K, N, model in prepare_shapes(args):
        print(f"\n{model}, N={N} K={K}")
        benchmark.run(
            print_data=True,
            save_path=args.save_path,
            N=N,
            K=K,
        )

    print("\nBenchmark finished!")
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `argparse.ArgumentParser`, `parser.add_argument`, `list`, `WEIGHT_SHAPES.keys`, `parser.parse_args`, `test_accuracy` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `argparse.ArgumentParser`, `parser.add_argument`, `list`, `WEIGHT_SHAPES.keys`, `parser.parse_args`, `test_accuracy` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Result presentation: the module aggregates measurements and renders tables or plots.
- **CN:** 结果呈现：该模块会汇总测量结果并生成表格或图形。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。
- **EN:** Performance metrics: the script emphasizes latency/throughput measurement rather than model quality.
- **CN:** 性能指标：该脚本更强调延迟/吞吐量测量，而不是模型质量。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`, `copy`, `itertools`.
- **CN:** 标准库依赖：`argparse`, `copy`, `itertools`。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: `weight_shapes`, `vllm`, `vllm.platforms`, `vllm.scalar_type`, `vllm.triton_utils`, `vllm.utils.flashinfer`.
- **CN:** 内部模块：`weight_shapes`, `vllm`, `vllm.platforms`, `vllm.scalar_type`, `vllm.triton_utils`, `vllm.utils.flashinfer`。
