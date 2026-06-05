# benchmark_norm_router_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_norm_router_gemm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, GEMM kernel benchmarks, multimodal processing benchmarks, centered around `HIDDEN_SIZE`, `NUM_EXPERTS_CHOICES`, `RMS_EPS`, `unfused_norm_router_gemm`. / 实现与基准测试编排、GEMM 内核基准、多模态处理基准相关的逻辑，核心符号包括 `HIDDEN_SIZE`, `NUM_EXPERTS_CHOICES`, `RMS_EPS`, `unfused_norm_router_gemm`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 3-15)
```python
"""Benchmark and correctness check for ``ops.dsv4_norm_router_gemm``.

Two implementations are compared:

  1. ``unfused``   — ``vllm_ops.rms_norm`` then ``ops.dsv3_router_gemm``,
                     i.e. the current vLLM hot path (two kernel launches).
  2. ``fused``     — ``ops.dsv4_norm_router_gemm``, the new single-kernel
                     fused path.

Both produce ``(normed_x: bf16, router_logits: fp32)``.  The correctness
check verifies that ``fused`` and ``unfused`` agree to within ~1 bf16
ULP — that is the precision floor for this op.
"""
```
**EN:** The module docstring introduces Benchmark and correctness check for ``ops.dsv4_norm_router_gemm``. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Benchmark and correctness check for ``ops.dsv4_norm_router_gemm`` 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 17-22)
```python
import argparse

import torch

from vllm import _custom_ops as vllm_ops
from vllm.triton_utils import triton
```
**EN:** This block gathers standard-library helpers such as `argparse`; third-party packages such as `torch`; project-local modules such as `vllm`, `vllm.triton_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`；第三方依赖，如 `torch`；项目内部模块，如 `vllm`, `vllm.triton_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 28-30)
```python
HIDDEN_SIZE = 7168
NUM_EXPERTS_CHOICES = (384,)
RMS_EPS = 1e-6
```
**EN:** This top-level block prepares shared state such as `HIDDEN_SIZE`, `NUM_EXPERTS_CHOICES`, `RMS_EPS`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `HIDDEN_SIZE`, `NUM_EXPERTS_CHOICES`, `RMS_EPS`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `unfused_norm_router_gemm` (lines 33-46)
```python
def unfused_norm_router_gemm(
    x: torch.Tensor,
    norm_weight: torch.Tensor,
    gate_weight: torch.Tensor,
    eps: float,
) -> tuple[torch.Tensor, torch.Tensor]:
    # Call ``_C::rms_norm`` directly (mirroring ``_dsv4_pro_norm_gate``'s
    # fallback path) so the benchmarked baseline doesn't inherit any
    # Python wrapper overhead or risk falling through to the native
    # eager-primitive ``RMSNorm.forward_native`` path.
    normed = torch.empty_like(x)
    torch.ops._C.rms_norm(normed, x, norm_weight, eps)
    logits = vllm_ops.dsv3_router_gemm(normed, gate_weight, torch.float32)
    return normed, logits
```
**EN:** `unfused_norm_router_gemm` implements GEMM kernel benchmarks. It mainly works with `x`, `norm_weight`, `gate_weight`, `eps` and relies on `torch.empty_like`, `torch.ops._C.rms_norm`, `vllm_ops.dsv3_router_gemm` plus value production to move data through this part of the benchmark pipeline.
**CN:** `unfused_norm_router_gemm` 负责实现GEMM 内核基准。 它主要处理 `x`, `norm_weight`, `gate_weight`, `eps`，并结合 `torch.empty_like`, `torch.ops._C.rms_norm`, `vllm_ops.dsv3_router_gemm` 以及 结果返回 来完成这一段基准测试流程。

### Function `fused_norm_router_gemm` (lines 49-55)
```python
def fused_norm_router_gemm(
    x: torch.Tensor,
    norm_weight: torch.Tensor,
    gate_weight: torch.Tensor,
    eps: float,
) -> tuple[torch.Tensor, torch.Tensor]:
    return vllm_ops.dsv4_norm_router_gemm(x, norm_weight, gate_weight, eps)
```
**EN:** `fused_norm_router_gemm` implements GEMM kernel benchmarks. It mainly works with `x`, `norm_weight`, `gate_weight`, `eps` and relies on `vllm_ops.dsv4_norm_router_gemm` plus value production to move data through this part of the benchmark pipeline.
**CN:** `fused_norm_router_gemm` 负责实现GEMM 内核基准。 它主要处理 `x`, `norm_weight`, `gate_weight`, `eps`，并结合 `vllm_ops.dsv4_norm_router_gemm` 以及 结果返回 来完成这一段基准测试流程。

### Function `_make_inputs` (lines 58-67)
```python
def _make_inputs(num_tokens: int, num_experts: int, hidden_size: int, seed: int = 0):
    torch.manual_seed(seed)
    device = "cuda"
    x = torch.randn(num_tokens, hidden_size, dtype=torch.bfloat16, device=device)
    norm_w = torch.randn(hidden_size, dtype=torch.bfloat16, device=device)
    gate_w = torch.randn(num_experts, hidden_size, dtype=torch.bfloat16, device=device)
    # Down-scale gate_w so the GEMV output stays in a representable range.
    gate_w = gate_w / float(hidden_size) ** 0.5
    norm_w = (norm_w * 0.1) + 1.0
    return x, norm_w, gate_w
```
**EN:** `_make_inputs` implements a helper used by `benchmark_norm_router_gemm.py`. It mainly works with `num_tokens`, `num_experts`, `hidden_size`, `seed` and relies on `torch.manual_seed`, `torch.randn`, `float` plus value production to move data through this part of the benchmark pipeline.
**CN:** `_make_inputs` 负责实现 `benchmark_norm_router_gemm.py` 使用的辅助逻辑。 它主要处理 `num_tokens`, `num_experts`, `hidden_size`, `seed`，并结合 `torch.manual_seed`, `torch.randn`, `float` 以及 结果返回 来完成这一段基准测试流程。

### Function `calculate_diff` (lines 70-116)
```python
def calculate_diff(
    num_tokens: int,
    num_experts: int,
    hidden_size: int = HIDDEN_SIZE,
    normed_atol: float = 2e-3,
    logits_atol: float = 1e-2,
    rtol: float = 1e-2,
) -> None:
    x, norm_w, gate_w = _make_inputs(num_tokens, num_experts, hidden_size)

    normed_unfused, logits_unfused = unfused_norm_router_gemm(
        x.clone(), norm_w, gate_w, RMS_EPS
    )
    normed_fused, logits_fused = fused_norm_router_gemm(
        x.clone(), norm_w, gate_w, RMS_EPS
    )

    def _max_abs(a, b):
        return (a.float() - b.float()).abs().max().item()

    print(f"\n=== M={num_tokens} E={num_experts} H={hidden_size} ===")
    print(f"normed_x  |fused - unfused| = {_max_abs(normed_fused, normed_unfused):.3e}")
    print(f"logits    |fused - unfused| = {_max_abs(logits_fused, logits_unfused):.3e}")

    ok_normed = torch.allclose(
        normed_fused.float(),
        normed_unfused.float(),
        atol=normed_atol,
        rtol=rtol,
    )
    ok_logits = torch.allclose(
        logits_fused.float(),
        logits_unfused.float(),
        atol=logits_atol,
        rtol=rtol,
    )
    if ok_normed and ok_logits:
        print(
            f"OK   fused vs unfused within "
            f"normed_atol={normed_atol:.0e} logits_atol={logits_atol:.0e} "
            f"rtol={rtol:.0e}"
        )
    else:
        print(
            f"FAIL normed_ok={ok_normed} logits_ok={ok_logits}; "
            f"see max-abs values above"
        )
```
**EN:** `calculate_diff` implements a helper used by `benchmark_norm_router_gemm.py`. It mainly works with `num_tokens`, `num_experts`, `hidden_size`, `normed_atol`, `logits_atol`, ... and relies on `_make_inputs`, `unfused_norm_router_gemm`, `x.clone`, `fused_norm_router_gemm`, `abs.max.item`, `abs.max` plus branching to move data through this part of the benchmark pipeline.
**CN:** `calculate_diff` 负责实现 `benchmark_norm_router_gemm.py` 使用的辅助逻辑。 它主要处理 `num_tokens`, `num_experts`, `hidden_size`, `normed_atol`, `logits_atol`, ...，并结合 `_make_inputs`, `unfused_norm_router_gemm`, `x.clone`, `fused_norm_router_gemm`, `abs.max.item`, `abs.max` 以及 条件分支 来完成这一段基准测试流程。

### Function `get_benchmark` (lines 119-153)
```python
def get_benchmark():
    # Only num_tokens varies (DSV4-Pro hard-codes E=384); single-axis
    # sweep yields a clean line plot with M on the x-axis.
    num_experts = NUM_EXPERTS_CHOICES[0]

    @triton.testing.perf_report(
        triton.testing.Benchmark(
            x_names=["num_tokens"],
            x_vals=list(range(1, 17)),
            line_arg="provider",
            line_vals=["unfused", "fused"],
            line_names=["unfused (rms+dsv3)", "fused (dsv4)"],
            styles=[("green", "-"), ("red", "-")],
            ylabel="us",
            plot_name=f"norm-router-gemm-E{num_experts}-H{HIDDEN_SIZE}",
            args={},
        )
    )
    def benchmark(num_tokens, provider):
        x, norm_w, gate_w = _make_inputs(num_tokens, num_experts, HIDDEN_SIZE)

        quantiles = [0.5, 0.2, 0.8]
        if provider == "unfused":
            fn = lambda: unfused_norm_router_gemm(  # noqa: E731
                x, norm_w, gate_w, RMS_EPS
            )
        else:
            fn = lambda: fused_norm_router_gemm(  # noqa: E731
                x, norm_w, gate_w, RMS_EPS
            )

        ms, min_ms, max_ms = triton.testing.do_bench(fn, quantiles=quantiles)
        return 1000 * ms, 1000 * max_ms, 1000 * min_ms

    return benchmark
```
**EN:** `get_benchmark` coordinates or measures benchmark orchestration. It mainly works with no explicit parameters and relies on `_make_inputs`, `unfused_norm_router_gemm`, `fused_norm_router_gemm`, `triton.testing.do_bench`, `triton.testing.perf_report`, `triton.testing.Benchmark` plus branching to move data through this part of the benchmark pipeline.
**CN:** `get_benchmark` 负责协调或测量基准测试编排。 它主要处理 无显式参数，并结合 `_make_inputs`, `unfused_norm_router_gemm`, `fused_norm_router_gemm`, `triton.testing.do_bench`, `triton.testing.perf_report`, `triton.testing.Benchmark` 以及 条件分支 来完成这一段基准测试流程。

### Function `main` (lines 156-179)
```python
def main() -> None:
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--save-path",
        type=str,
        default="./configs/norm_router_gemm/",
    )
    parser.add_argument(
        "--skip-bench",
        action="store_true",
        help="Run only the correctness check, not the perf sweep.",
    )
    args = parser.parse_args()

    # Correctness sweep over the full fast-path range M=1..16.
    for m in range(1, 17):
        for e in NUM_EXPERTS_CHOICES:
            calculate_diff(num_tokens=m, num_experts=e, hidden_size=HIDDEN_SIZE)

    if args.skip_bench:
        return

    benchmark = get_benchmark()
    benchmark.run(print_data=True, save_path=args.save_path)
```
**EN:** `main` serves as the main entry point for this script. It mainly works with no explicit parameters and relies on `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `range`, `calculate_diff`, `get_benchmark` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 无显式参数，并结合 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`, `range`, `calculate_diff`, `get_benchmark` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Entry point (lines 182-183)
```python
if __name__ == "__main__":
    main()
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `main` 等调用串起来，使模块能够从命令行直接启动。

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
- **EN:** Standard library: `argparse`.
- **CN:** 标准库依赖：`argparse`。
- **EN:** Third-party packages: `torch`.
- **CN:** 第三方依赖：`torch`。
- **EN:** Internal modules: `vllm`, `vllm.triton_utils`.
- **CN:** 内部模块：`vllm`, `vllm.triton_utils`。
