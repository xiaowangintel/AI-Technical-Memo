# bench_rmsnorm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/benchmark/bench_rmsnorm.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This benchmark script measures `RMSNorm` kernels across shapes, providers, and dtypes, then reports latency or speedup. / 该基准脚本围绕 `RMSNorm` 内核在不同形状、实现提供方和数据类型下进行测量，并输出延迟或加速比。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: Comments and local context
````python
# Benchmarks SGLang RMSNorm kernels versus vLLM and FlashInfer across
# (batch_size, seq_len, hidden_size) and prints speed-up.
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 3-16: Imports and module setup
````python
import argparse
import itertools
import os
import re
from typing import List, Optional, Tuple, Union

import sgl_kernel
import torch
import torch.nn as nn
import triton
import triton.testing
from sgl_kernel.utils import is_arch_support_pdl

from sglang.utils import is_in_ci
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 17-18: Comments and local context
````python

# Optional imports
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 19-26: Optional dependency handling
````python
try:
    from flashinfer.norm import fused_add_rmsnorm, rmsnorm

    FLASHINFER_AVAILABLE = True
except ImportError:
    fused_add_rmsnorm = None
    rmsnorm = None
    FLASHINFER_AVAILABLE = False
````
**EN:** This block attempts an operation that may fail, then provides fallback behavior so the module can keep working in reduced mode.
**CN:** 该部分尝试执行可能失败的操作，并提供回退行为，使模块能够以降级模式继续工作。

### Lines 28-34: Optional dependency handling
````python
try:
    from vllm import _custom_ops as vllm_ops

    VLLM_AVAILABLE = True
except ImportError:
    vllm_ops = None
    VLLM_AVAILABLE = False
````
**EN:** This block attempts an operation that may fail, then provides fallback behavior so the module can keep working in reduced mode.
**CN:** 该部分尝试执行可能失败的操作，并提供回退行为，使模块能够以降级模式继续工作。

### Lines 36-36: Constants and configuration
````python
IS_CI = is_in_ci()
````
**EN:** This block defines shared constants or configuration values such as `IS_CI`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `IS_CI`），供后续函数或控制流程复用。

### Lines 39-44: `str2int_list` definition
````python
def str2int_list(arg: str) -> List[int]:
    if arg in ("", None):
        return []
    if re.fullmatch(r"\d+(,\d+)*", arg.strip()) is None:
        raise argparse.ArgumentTypeError(f"Bad int list: {arg}")
    return [int(x) for x in arg.split(",")]
````
**EN:** This section defines `str2int_list` and implements the core logic associated with str2int list.
**CN:** 该部分定义 `str2int_list`，并实现与 str2int list 相关的核心逻辑。

### Lines 47-70: `HuggingFaceRMSNorm` definition
````python
class HuggingFaceRMSNorm(nn.Module):
    def __init__(self, hidden_size: int, eps: float = 1e-6) -> None:
        super().__init__()
        self.weight = nn.Parameter(torch.ones(hidden_size))
        self.variance_epsilon = eps

    def forward(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        orig_dtype = x.dtype
        x = x.to(torch.float32)
        if residual is not None:
            x = x + residual.to(torch.float32)
            residual = x.to(orig_dtype)

        variance = x.pow(2).mean(dim=-1, keepdim=True)
        x = x * torch.rsqrt(variance + self.variance_epsilon)
        x = x.to(orig_dtype) * self.weight
        if residual is None:
            return x
        else:
            return x, residual
````
**EN:** This section defines the class `HuggingFaceRMSNorm`, grouping related state and behavior around Hugging Face RMSNorm.
**CN:** 该部分定义类 `HuggingFaceRMSNorm`，把与 Hugging Face RMSNorm 相关的状态和行为组织在一起。

### Lines 73-94: `rmsnorm_naive` definition
````python
def rmsnorm_naive(
    x: torch.Tensor,
    weight: torch.Tensor,
    residual: Optional[torch.Tensor] = None,
    eps: float = 1e-6,
):
    naive_norm = HuggingFaceRMSNorm(x.shape[-1], eps=eps)
    naive_norm.weight = nn.Parameter(weight)
    naive_norm = naive_norm.to(x.device)

    orig_shape = x.shape
    x = x.view(-1, x.shape[-1])
    if residual is not None:
        residual = residual.view(-1, residual.shape[-1])

    output = naive_norm(x, residual)

    if isinstance(output, tuple):
        output = (output[0].view(orig_shape), output[1].view(orig_shape))
    else:
        output = output.view(orig_shape)
    return output
````
**EN:** This section defines `rmsnorm_naive` and implements the core logic associated with RMSNorm naive.
**CN:** 该部分定义 `rmsnorm_naive`，并实现与 RMSNorm naive 相关的核心逻辑。

### Lines 97-122: `rmsnorm_flashinfer` definition
````python
def rmsnorm_flashinfer(
    x: torch.Tensor,
    weight: torch.Tensor,
    residual: Optional[torch.Tensor] = None,
    eps: float = 1e-6,
):
    if not FLASHINFER_AVAILABLE:
        # Fallback to naive implementation if FlashInfer is not available
        return rmsnorm_naive(x, weight, residual, eps)

    orig_shape = x.shape
    x = x.view(-1, x.shape[-1])
    if residual is not None:
        residual = residual.view(-1, residual.shape[-1])

    if residual is not None:
        fused_add_rmsnorm(x, residual, weight, eps)
        output = (x, residual)
    else:
        output = rmsnorm(x, weight, eps)

    if isinstance(output, tuple):
        output = (output[0].view(orig_shape), output[1].view(orig_shape))
    else:
        output = output.view(orig_shape)
    return output
````
**EN:** This section defines `rmsnorm_flashinfer` and implements the core logic associated with RMSNorm flashinfer.
**CN:** 该部分定义 `rmsnorm_flashinfer`，并实现与 RMSNorm flashinfer 相关的核心逻辑。

### Lines 125-152: `rmsnorm_vllm` definition
````python
def rmsnorm_vllm(
    x: torch.Tensor,
    weight: torch.Tensor,
    residual: Optional[torch.Tensor] = None,
    eps: float = 1e-6,
):
    if not VLLM_AVAILABLE:
        # Fallback to naive implementation if vLLM is not available
        return rmsnorm_naive(x, weight, residual, eps)

    orig_shape = x.shape
    x = x.view(-1, x.shape[-1])
    if residual is not None:
        residual = residual.view(-1, residual.shape[-1])

    if residual is not None:
        vllm_ops.fused_add_rms_norm(x, residual, weight, eps)
        output = (x, residual)
    else:
        out = torch.empty_like(x)
        vllm_ops.rms_norm(out, x, weight, eps)
        output = out

    if isinstance(output, tuple):
        output = (output[0].view(orig_shape), output[1].view(orig_shape))
    else:
        output = output.view(orig_shape)
    return output
````
**EN:** This section defines `rmsnorm_vllm` and implements the core logic associated with RMSNorm vllm.
**CN:** 该部分定义 `rmsnorm_vllm`，并实现与 RMSNorm vllm 相关的核心逻辑。

### Lines 155-182: `rmsnorm_sglang` definition
````python
def rmsnorm_sglang(
    x: torch.Tensor,
    weight: torch.Tensor,
    residual: Optional[torch.Tensor] = None,
    eps: float = 1e-6,
    enable_pdl: Optional[bool] = None,
):
    orig_shape = x.shape
    x = x.view(-1, x.shape[-1])
    if residual is not None:
        residual = residual.view(-1, residual.shape[-1])

    if enable_pdl is None:
        enable_pdl = is_arch_support_pdl()

    if residual is not None:
        sgl_kernel.fused_add_rmsnorm(x, residual, weight, eps, enable_pdl=enable_pdl)
        output = (x, residual)
    else:
        out = torch.empty_like(x)
        sgl_kernel.rmsnorm(x, weight, eps, out=out, enable_pdl=enable_pdl)
        output = out

    if isinstance(output, tuple):
        output = (output[0].view(orig_shape), output[1].view(orig_shape))
    else:
        output = output.view(orig_shape)
    return output
````
**EN:** This section defines `rmsnorm_sglang` and implements the core logic associated with RMSNorm sglang.
**CN:** 该部分定义 `rmsnorm_sglang`，并实现与 RMSNorm sglang 相关的核心逻辑。

### Lines 185-235: `calculate_diff` definition
````python
def calculate_diff(batch_size, seq_len, hidden_size, use_residual=True):
    dtype = torch.bfloat16
    x = torch.randn(batch_size, seq_len, hidden_size, dtype=dtype, device="cuda")
    weight = torch.ones(hidden_size, dtype=dtype, device="cuda")
    residual = torch.randn_like(x) if use_residual else None

    output_naive = rmsnorm_naive(
        x.clone(), weight, residual.clone() if residual is not None else None
    )
    output_flashinfer = rmsnorm_flashinfer(
        x.clone(), weight, residual.clone() if residual is not None else None
    )
    output_vllm = rmsnorm_vllm(
        x.clone(), weight, residual.clone() if residual is not None else None
    )
    output_sglang = rmsnorm_sglang(
        x.clone(), weight, residual.clone() if residual is not None else None
    )

    if use_residual:
        output_naive = output_naive[0]
        output_flashinfer = output_flashinfer[0]
        output_vllm = output_vllm[0]
        output_sglang = output_sglang[0]

    print(f"Naive output={output_naive}")
    if FLASHINFER_AVAILABLE:
        print(f"FlashInfer output={output_flashinfer}")
    else:
        print("FlashInfer not available, skipped")
    if VLLM_AVAILABLE:
        print(f"VLLM output={output_vllm}")
    else:
        print("vLLM not available, skipped")
    print(f"SGLang output={output_sglang}")

    # Only compare available implementations
    all_match = torch.allclose(output_naive, output_sglang, atol=1e-2, rtol=1e-2)
    if FLASHINFER_AVAILABLE:
        all_match = all_match and torch.allclose(
            output_naive, output_flashinfer, atol=1e-2, rtol=1e-2
        )
    if VLLM_AVAILABLE:
        all_match = all_match and torch.allclose(
            output_naive, output_vllm, atol=1e-2, rtol=1e-2
        )

    if all_match:
        print("✅ All available implementations match")
    else:
        print("❌ Implementations differ")
````
**EN:** This section defines `calculate_diff`. It calculates the `diff` path used by the module.
**CN:** 该部分定义 `calculate_diff`。它负责计算模块中与 `diff` 相关的处理路径。

### Lines 236-238: Comments and local context
````python


# CI environment uses simplified parameters
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 239-246: Conditional logic
````python
if IS_CI:
    default_batch_sizes = [1]  # Single batch size for CI
    default_seq_lens = [64]  # Single sequence length for CI
    default_hidden_sizes = [4096]  # Single hidden size for CI
else:
    default_batch_sizes = [2**i for i in range(0, 7, 2)]  # 1, 4, 16, 64
    default_seq_lens = [2**i for i in range(6, 11, 1)]  # 64, 128, 256, 512, 1024
    default_hidden_sizes = [32 * 128, 48 * 128]  # 4096, 6144
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 249-250: `make_configs` definition
````python
def make_configs(bsizes: List[int], slens: List[int], hsizes: List[int]) -> List[Tuple]:
    return list(itertools.product(bsizes, slens, hsizes))
````
**EN:** This section defines `make_configs`. It constructs the `configs` path used by the module.
**CN:** 该部分定义 `make_configs`。它负责构造模块中与 `configs` 相关的处理路径。

### Lines 251-253: Comments and local context
````python


# Filter providers based on availability
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 254-256: Constants and configuration
````python
available_providers = ["huggingface", "sglang"]
available_names = ["HuggingFace", "SGL Kernel"]
available_styles = [("blue", "-"), ("orange", "-")]
````
**EN:** This block defines shared constants or configuration values such as `available_providers`, `available_names`, `available_styles`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `available_providers`, `available_names`, `available_styles`），供后续函数或控制流程复用。

### Lines 258-261: Conditional logic
````python
if FLASHINFER_AVAILABLE:
    available_providers.insert(-1, "flashinfer")
    available_names.insert(-1, "FlashInfer")
    available_styles.insert(-1, ("green", "-"))
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 263-266: Conditional logic
````python
if VLLM_AVAILABLE:
    available_providers.insert(-1, "vllm")
    available_names.insert(-1, "vLLM")
    available_styles.insert(-1, ("red", "-"))
````
**EN:** This section selects different behavior according to runtime conditions such as platform, environment, or feature availability.
**CN:** 该部分根据平台、环境或功能可用性等运行时条件选择不同的行为。

### Lines 269-362: `benchmark` definition
````python
@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["batch_size", "seq_len", "hidden_size"],
        x_vals=[],
        line_arg="provider",
        line_vals=available_providers,
        line_names=available_names,
        styles=available_styles,
        ylabel="µs (median)  or  × (speed-up)",
        plot_name="rmsnorm-performance",
        args={},
    )
)
def benchmark(batch_size, seq_len, hidden_size, provider, use_residual):
    device = torch.device("cuda")
    dtype = torch.bfloat16

    x = torch.randn(batch_size, seq_len, hidden_size, dtype=dtype, device=device)
    weight = torch.ones(hidden_size, dtype=dtype, device=device)
    residual = torch.randn_like(x) if use_residual else None

    # timing helper
    def timed(fn):
        for _ in range(5):
            fn()
        torch.cuda.synchronize()
        ms, qmin, qmax = triton.testing.do_bench_cudagraph(
            fn, quantiles=[0.5, 0.2, 0.8]
        )
        return 1000 * ms, 1000 * qmax, 1000 * qmin

    if provider == "huggingface":
        return timed(
            lambda: rmsnorm_naive(
                x.clone(),
                weight,
                residual.clone() if residual is not None else None,
            )
        )
    elif provider == "flashinfer":
        if not FLASHINFER_AVAILABLE:
            return (0, 0, 0)
        return timed(
            lambda: rmsnorm_flashinfer(
                x.clone(),
                weight,
                residual.clone() if residual is not None else None,
            )
        )
    elif provider == "vllm":
        if not VLLM_AVAILABLE:
            return (0, 0, 0)
        return timed(
            lambda: rmsnorm_vllm(
                x.clone(),
                weight,
                residual.clone() if residual is not None else None,
            )
        )
    elif provider == "sglang":
        return timed(
            lambda: rmsnorm_sglang(
                x.clone(),
                weight,
                residual.clone() if residual is not None else None,
            )
        )

    # provider == "speedup"
    if VLLM_AVAILABLE:
        t_ref, _, _ = timed(
            lambda: rmsnorm_vllm(
                x.clone(),
                weight,
                residual.clone() if residual is not None else None,
            )
        )
    else:
        t_ref, _, _ = timed(
            lambda: rmsnorm_naive(
                x.clone(),
                weight,
                residual.clone() if residual is not None else None,
            )
        )
    t_sgl, _, _ = timed(
        lambda: rmsnorm_sglang(
            x.clone(),
            weight,
            residual.clone() if residual is not None else None,
        )
    )
    spd = t_ref / t_sgl if t_ref > 0 else 1.0
    return (spd, spd, spd)
````
**EN:** This section defines `benchmark` and implements the core logic associated with benchmark. Benchmark helpers from Triton are used to measure runtime behavior.
**CN:** 该部分定义 `benchmark`，并实现与 benchmark 相关的核心逻辑。 这里使用 Triton 的基准辅助工具来测量运行时行为。

### Lines 365-395: Command-line entry point
````python
if __name__ == "__main__":
    p = argparse.ArgumentParser("RMSNorm kernel benchmark")
    p.add_argument("--batch_sizes", type=str2int_list, default=default_batch_sizes)
    p.add_argument("--seq_lens", type=str2int_list, default=default_seq_lens)
    p.add_argument("--hidden_sizes", type=str2int_list, default=default_hidden_sizes)
    p.add_argument(
        "--use_residual", action="store_true", help="Whether to use residual connection"
    )
    p.add_argument("--verify_only", action="store_true")
    args = p.parse_args()

    # coerce lists
    if isinstance(args.batch_sizes, str):
        args.batch_sizes = str2int_list(args.batch_sizes)
    if isinstance(args.seq_lens, str):
        args.seq_lens = str2int_list(args.seq_lens)
    if isinstance(args.hidden_sizes, str):
        args.hidden_sizes = str2int_list(args.hidden_sizes)

    # patch perf_report grid
    benchmark_grid = make_configs(args.batch_sizes, args.seq_lens, args.hidden_sizes)
    if hasattr(benchmark, "benchmarks"):
        benchmark.benchmarks.x_vals = benchmark_grid
    else:
        benchmark.benchmark.x_vals = benchmark_grid

    if args.verify_only:
        ok = calculate_diff(4, 128, args.hidden_sizes[0], args.use_residual)
        print("✅ sanity pass" if ok else "❌ mismatch")
    else:
        benchmark.run(print_data=True, use_residual=args.use_residual)
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Benchmark driver / 基准驱动脚本
- **Primary symbols / 主要符号**: `str2int_list`, `HuggingFaceRMSNorm`, `rmsnorm_naive`, `rmsnorm_flashinfer`, `rmsnorm_vllm`, `rmsnorm_sglang`, `calculate_diff`, `make_configs`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, Triton benchmarking / Triton 基准工具, CLI argument parsing / 命令行参数解析
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sgl_kernel.utils`, `sglang.utils`
- **External / 外部**: `argparse`, `flashinfer.norm`, `itertools`, `os`, `re`, `torch`, `torch.nn`, `triton`, `triton.testing`, `typing`, `vllm`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
