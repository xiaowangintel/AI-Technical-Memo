# bench_qknorm_across_heads.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/benchmark/bench_qknorm_across_heads.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import itertools". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import itertools”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Imports and module setup
```python
import itertools
from typing import Tuple

import torch
import triton
import triton.testing
from sgl_kernel import rmsnorm

from sglang.jit_kernel.benchmark.utils import run_benchmark
from sglang.jit_kernel.norm import fused_inplace_qknorm_across_heads
from sglang.srt.utils import get_current_device_stream_fast
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.utils import is_in_ci

register_cuda_ci(est_time=12, suite="base-b-kernel-benchmark-1-gpu-large")

IS_CI = is_in_ci()

alt_stream = torch.cuda.Stream()
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 22-31: Function `sglang_jit_qknorm_across_heads`
```python
def sglang_jit_qknorm_across_heads(
    q: torch.Tensor,
    k: torch.Tensor,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
) -> None:

    fused_inplace_qknorm_across_heads(q, k, q_weight, k_weight)
```
**EN:** This block defines `sglang_jit_qknorm_across_heads`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `sglang_jit_qknorm_across_heads`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 32-46: Function `sglang_aot_qknorm_across_heads`
```python
def sglang_aot_qknorm_across_heads(
    q: torch.Tensor,
    k: torch.Tensor,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
) -> None:

    current_stream = get_current_device_stream_fast()
    alt_stream.wait_stream(current_stream)
    rmsnorm(q, q_weight, out=q)
    with torch.cuda.stream(alt_stream):
        rmsnorm(k, k_weight, out=k)
    current_stream.wait_stream(alt_stream)
```
**EN:** This block defines `sglang_aot_qknorm_across_heads`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `sglang_aot_qknorm_across_heads`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 47-59: Imports and module setup
```python
def flashinfer_qknorm_across_heads(
    q: torch.Tensor,
    k: torch.Tensor,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
) -> None:
    from flashinfer import rmsnorm

    rmsnorm(q, q_weight, out=q)
    rmsnorm(k, k_weight, out=k)


@torch.compile()
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 60-104: Function `torch_impl_qknorm_across_heads`
```python
def torch_impl_qknorm_across_heads(
    q: torch.Tensor,
    k: torch.Tensor,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
    eps: float = 1e-6,
) -> None:
    q_mean = q.float().pow(2).mean(dim=-1, keepdim=True)
    k_mean = k.float().pow(2).mean(dim=-1, keepdim=True)
    q_norm = (q_mean + eps).rsqrt()
    k_norm = (k_mean + eps).rsqrt()
    q.copy_(q.float() * q_norm * q_weight.float())
    k.copy_(k.float() * k_norm * k_weight.float())


DTYPE = torch.bfloat16
DEVICE = "cuda"

if IS_CI:
    BS_RANGE = [16]
    HIDDEN_DIM_RANGE = [1024]
else:
    BS_RANGE = [2**n for n in range(0, 14)]
    HIDDEN_DIM_RANGE = [512, 1024, 2048, 4096, 8192]

LINE_VALS = ["jit", "aot", "flashinfer", "torch"]
LINE_NAMES = ["SGL JIT Kernel", "SGL AOT Kernel", "FlashInfer", "PyTorch"]
STYLES = [("blue", "-"), ("orange", "--"), ("green", "-."), ("red", ":")]

configs = list(itertools.product(BS_RANGE, HIDDEN_DIM_RANGE))


@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["batch_size", "hidden_dim"],
        x_vals=configs,
        line_arg="provider",
        line_vals=LINE_VALS,
        line_names=LINE_NAMES,
        styles=STYLES,
        ylabel="us",
        plot_name="qknorm-across-heads-performance",
        args={},
    )
)
```
**EN:** This block defines `torch_impl_qknorm_across_heads`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `torch_impl_qknorm_across_heads`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 105-123: Function `benchmark`
```python
def benchmark(
    batch_size: int, hidden_dim: int, provider: str
) -> Tuple[float, float, float]:
    q = torch.randn((batch_size, hidden_dim), dtype=DTYPE, device=DEVICE)
    k = torch.randn((batch_size, hidden_dim), dtype=DTYPE, device=DEVICE)
    q_weight = torch.randn(hidden_dim, dtype=DTYPE, device=DEVICE)
    k_weight = torch.randn(hidden_dim, dtype=DTYPE, device=DEVICE)
    FN_MAP = {
        "jit": sglang_jit_qknorm_across_heads,
        "aot": sglang_aot_qknorm_across_heads,
        "flashinfer": flashinfer_qknorm_across_heads,
        "torch": torch_impl_qknorm_across_heads,
    }
    fn = lambda: FN_MAP[provider](q, k, q_weight, k_weight)
    return run_benchmark(fn)


if __name__ == "__main__":
    benchmark.run(print_data=True)
```
**EN:** This block defines `benchmark`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `benchmark`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `itertools`
- `typing -> Tuple`
- `torch`
- `triton`
- `triton.testing`
- `sgl_kernel -> rmsnorm`
- `sglang.jit_kernel.benchmark.utils -> run_benchmark`
- `sglang.jit_kernel.norm -> fused_inplace_qknorm_across_heads`
- `sglang.srt.utils -> get_current_device_stream_fast`
- `sglang.test.ci.ci_register -> register_cuda_ci`
- `sglang.utils -> is_in_ci`
- `flashinfer -> rmsnorm`
