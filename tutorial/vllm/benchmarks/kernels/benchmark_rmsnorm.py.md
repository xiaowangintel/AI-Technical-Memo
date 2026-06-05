# benchmark_rmsnorm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_rmsnorm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, RMSNorm kernel benchmarks, centered around `HuggingFaceRMSNorm`, `rmsnorm_naive`, `rmsnorm_flashinfer`, `rmsnorm_vllm`. / 实现与基准测试编排、RMSNorm 内核基准相关的逻辑，核心符号包括 `HuggingFaceRMSNorm`, `rmsnorm_naive`, `rmsnorm_flashinfer`, `rmsnorm_vllm`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-11)
```python
import itertools

import torch
from flashinfer.norm import fused_add_rmsnorm, rmsnorm
from torch import nn

from vllm import _custom_ops as vllm_ops
from vllm.triton_utils import triton
```
**EN:** This block gathers standard-library helpers such as `itertools`; third-party packages such as `torch`, `flashinfer.norm`; project-local modules such as `vllm`, `vllm.triton_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `itertools`；第三方依赖，如 `torch`, `flashinfer.norm`；项目内部模块，如 `vllm`, `vllm.triton_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Class `HuggingFaceRMSNorm` (lines 14-37)
```python
class HuggingFaceRMSNorm(nn.Module):
    def __init__(self, hidden_size: int, eps: float = 1e-6) -> None:
        super().__init__()
        self.weight = nn.Parameter(torch.ones(hidden_size))
        self.variance_epsilon = eps

    def forward(
        self,
        x: torch.Tensor,
        residual: torch.Tensor | None = None,
    ) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]:
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
```
**EN:** Class `HuggingFaceRMSNorm` packages shared state and related operations for this benchmark module. It extends `nn.Module` and exposes methods such as `__init__`, `forward`.
**CN:** 类 `HuggingFaceRMSNorm` 用于封装该基准模块的共享状态与相关操作。它继承自 `nn.Module`，并提供 `__init__`, `forward` 等方法。

### Method `HuggingFaceRMSNorm.__init__` (lines 15-18)
```python
    def __init__(self, hidden_size: int, eps: float = 1e-6) -> None:
        super().__init__()
        self.weight = nn.Parameter(torch.ones(hidden_size))
        self.variance_epsilon = eps
```
**EN:** `__init__` implements a helper used by `benchmark_rmsnorm.py`. It mainly works with `hidden_size`, `eps` and relies on `super.__init__`, `super`, `nn.Parameter`, `torch.ones` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `benchmark_rmsnorm.py` 使用的辅助逻辑。 它主要处理 `hidden_size`, `eps`，并结合 `super.__init__`, `super`, `nn.Parameter`, `torch.ones` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `HuggingFaceRMSNorm.forward` (lines 20-37)
```python
    def forward(
        self,
        x: torch.Tensor,
        residual: torch.Tensor | None = None,
    ) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]:
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
```
**EN:** `forward` implements a helper used by `benchmark_rmsnorm.py`. It mainly works with `x`, `residual` and relies on `x.to`, `residual.to`, `x.pow.mean`, `x.pow`, `torch.rsqrt` plus branching to move data through this part of the benchmark pipeline.
**CN:** `forward` 负责实现 `benchmark_rmsnorm.py` 使用的辅助逻辑。 它主要处理 `x`, `residual`，并结合 `x.to`, `residual.to`, `x.pow.mean`, `x.pow`, `torch.rsqrt` 以及 条件分支 来完成这一段基准测试流程。

### Function `rmsnorm_naive` (lines 40-61)
```python
def rmsnorm_naive(
    x: torch.Tensor,
    weight: torch.Tensor,
    residual: torch.Tensor | None = None,
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
```
**EN:** `rmsnorm_naive` coordinates or measures RMSNorm kernel benchmarks. It mainly works with `x`, `weight`, `residual`, `eps` and relies on `HuggingFaceRMSNorm`, `nn.Parameter`, `naive_norm.to`, `x.view`, `residual.view`, `naive_norm` plus branching to move data through this part of the benchmark pipeline.
**CN:** `rmsnorm_naive` 负责协调或测量RMSNorm 内核基准。 它主要处理 `x`, `weight`, `residual`, `eps`，并结合 `HuggingFaceRMSNorm`, `nn.Parameter`, `naive_norm.to`, `x.view`, `residual.view`, `naive_norm` 以及 条件分支 来完成这一段基准测试流程。

### Function `rmsnorm_flashinfer` (lines 64-85)
```python
def rmsnorm_flashinfer(
    x: torch.Tensor,
    weight: torch.Tensor,
    residual: torch.Tensor | None = None,
    eps: float = 1e-6,
):
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
```
**EN:** `rmsnorm_flashinfer` coordinates or measures RMSNorm kernel benchmarks. It mainly works with `x`, `weight`, `residual`, `eps` and relies on `x.view`, `residual.view`, `fused_add_rmsnorm`, `rmsnorm`, `isinstance`, `output.view` plus branching to move data through this part of the benchmark pipeline.
**CN:** `rmsnorm_flashinfer` 负责协调或测量RMSNorm 内核基准。 它主要处理 `x`, `weight`, `residual`, `eps`，并结合 `x.view`, `residual.view`, `fused_add_rmsnorm`, `rmsnorm`, `isinstance`, `output.view` 以及 条件分支 来完成这一段基准测试流程。

### Function `rmsnorm_vllm` (lines 88-111)
```python
def rmsnorm_vllm(
    x: torch.Tensor,
    weight: torch.Tensor,
    residual: torch.Tensor | None = None,
    eps: float = 1e-6,
):
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
```
**EN:** `rmsnorm_vllm` coordinates or measures RMSNorm kernel benchmarks. It mainly works with `x`, `weight`, `residual`, `eps` and relies on `x.view`, `residual.view`, `vllm_ops.fused_add_rms_norm`, `torch.empty_like`, `vllm_ops.rms_norm`, `isinstance` plus branching to move data through this part of the benchmark pipeline.
**CN:** `rmsnorm_vllm` 负责协调或测量RMSNorm 内核基准。 它主要处理 `x`, `weight`, `residual`, `eps`，并结合 `x.view`, `residual.view`, `vllm_ops.fused_add_rms_norm`, `torch.empty_like`, `vllm_ops.rms_norm`, `isinstance` 以及 条件分支 来完成这一段基准测试流程。

### Function `calculate_diff` (lines 114-144)
```python
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

    if use_residual:
        output_naive = output_naive[0]
        output_flashinfer = output_flashinfer[0]
        output_vllm = output_vllm[0]

    print(f"Naive output={output_naive}")
    print(f"FlashInfer output={output_flashinfer}")
    print(f"vLLM output={output_vllm}")

    if torch.allclose(
        output_naive, output_flashinfer, atol=1e-2, rtol=1e-2
    ) and torch.allclose(output_naive, output_vllm, atol=1e-2, rtol=1e-2):
        print("✅ All implementations match")
    else:
        print("❌ Implementations differ")
```
**EN:** `calculate_diff` implements a helper used by `benchmark_rmsnorm.py`. It mainly works with `batch_size`, `seq_len`, `hidden_size`, `use_residual` and relies on `torch.randn`, `torch.ones`, `torch.randn_like`, `rmsnorm_naive`, `x.clone`, `residual.clone` plus branching to move data through this part of the benchmark pipeline.
**CN:** `calculate_diff` 负责实现 `benchmark_rmsnorm.py` 使用的辅助逻辑。 它主要处理 `batch_size`, `seq_len`, `hidden_size`, `use_residual`，并结合 `torch.randn`, `torch.ones`, `torch.randn_like`, `rmsnorm_naive`, `x.clone`, `residual.clone` 以及 条件分支 来完成这一段基准测试流程。

### Top-level setup (lines 147-150)
```python
batch_size_range = [2**i for i in range(0, 7, 2)]
seq_length_range = [2**i for i in range(6, 11, 1)]
head_num_range = [32, 48]
configs = list(itertools.product(head_num_range, batch_size_range, seq_length_range))
```
**EN:** This top-level block prepares shared state such as `batch_size_range`, `seq_length_range`, `head_num_range`, `configs`. It uses `range`, `list`, `itertools.product` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `batch_size_range`, `seq_length_range`, `head_num_range`, `configs`。它借助 `range`, `list`, `itertools.product` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `get_benchmark` (lines 153-207)
```python
def get_benchmark(use_residual):
    @triton.testing.perf_report(
        triton.testing.Benchmark(
            x_names=["head_num", "batch_size", "seq_len"],
            x_vals=[list(_) for _ in configs],
            line_arg="provider",
            line_vals=["huggingface", "flashinfer", "vllm"],
            line_names=["HuggingFace", "FlashInfer", "vLLM"],
            styles=[("blue", "-"), ("green", "-"), ("red", "-")],
            ylabel="us",
            plot_name=f"rmsnorm-perf-{'with' if use_residual else 'without'}-residual",
            args={},
        )
    )
    def benchmark(head_num, batch_size, seq_len, provider):
        dtype = torch.bfloat16
        hidden_size = head_num * 128  # assuming head_dim = 128

        x = torch.randn(batch_size, seq_len, hidden_size, dtype=dtype, device="cuda")
        weight = torch.ones(hidden_size, dtype=dtype, device="cuda")
        residual = torch.randn_like(x) if use_residual else None

        quantiles = [0.5, 0.2, 0.8]

        if provider == "huggingface":
            ms, min_ms, max_ms = triton.testing.do_bench(
                lambda: rmsnorm_naive(
                    x.clone(),
                    weight,
                    residual.clone() if residual is not None else None,
                ),
                quantiles=quantiles,
            )
        elif provider == "flashinfer":
    # ... omitted for brevity ...
                    residual.clone() if residual is not None else None,
                ),
                quantiles=quantiles,
            )

        return 1000 * ms, 1000 * max_ms, 1000 * min_ms

    return benchmark
```
**EN:** `get_benchmark` coordinates or measures benchmark orchestration. It mainly works with `use_residual` and relies on `torch.randn`, `torch.ones`, `torch.randn_like`, `triton.testing.do_bench`, `rmsnorm_naive`, `x.clone` plus branching to move data through this part of the benchmark pipeline.
**CN:** `get_benchmark` 负责协调或测量基准测试编排。 它主要处理 `use_residual`，并结合 `torch.randn`, `torch.ones`, `torch.randn_like`, `triton.testing.do_bench`, `rmsnorm_naive`, `x.clone` 以及 条件分支 来完成这一段基准测试流程。

### Entry point (lines 210-255)
```python
if __name__ == "__main__":
    import argparse

    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--batch-size",
        type=int,
        default=4,
        help="Batch size",
    )
    parser.add_argument(
        "--seq-len",
        type=int,
        default=128,
        help="Sequence length",
    )
    parser.add_argument(
        "--hidden-size",
        type=int,
        default=4096,
        help="Hidden size (2nd dimension) of the sequence",
    )
    parser.add_argument(
        "--use-residual", action="store_true", help="Whether to use residual connection"
    )
    parser.add_argument(
        "--save-path",
        type=str,
        default="./configs/rmsnorm/",
        help="Path to save rmsnorm benchmark results",
    )

    args = parser.parse_args()

    # Run correctness test
    calculate_diff(
        batch_size=args.batch_size,
        seq_len=args.seq_len,
        hidden_size=args.hidden_size,
        use_residual=args.use_residual,
    )

    # Get the benchmark function with proper use_residual setting
    benchmark = get_benchmark(args.use_residual)
    # Run performance benchmark
    benchmark.run(print_data=True, save_path=args.save_path)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `calculate_diff`, `get_benchmark`, `benchmark.run` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `calculate_diff`, `get_benchmark`, `benchmark.run` 等调用串起来，使模块能够从命令行直接启动。

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
- **EN:** Standard library: `itertools`.
- **CN:** 标准库依赖：`itertools`。
- **EN:** Third-party packages: `torch`, `flashinfer.norm`.
- **CN:** 第三方依赖：`torch`, `flashinfer.norm`。
- **EN:** Internal modules: `vllm`, `vllm.triton_utils`.
- **CN:** 内部模块：`vllm`, `vllm.triton_utils`。
