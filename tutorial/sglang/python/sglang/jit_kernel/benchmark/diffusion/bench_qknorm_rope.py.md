# bench_qknorm_rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/benchmark/diffusion/bench_qknorm_rope.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from dataclasses import dataclass". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from dataclasses import dataclass”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Imports and module setup
```python
from dataclasses import dataclass
from typing import Tuple

import torch
import triton
import triton.testing

from sglang.jit_kernel.benchmark.utils import (
    DEFAULT_DEVICE,
    DEFAULT_DTYPE,
    get_benchmark_range,
    run_benchmark_no_cudagraph,
)
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=13, suite="base-b-kernel-benchmark-1-gpu-large")

MAX_SEQ_LEN = 131072
ROPE_BASE = 10000.0


@dataclass(frozen=True)
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 23-50: Class `CaseSpec`
```python
class CaseSpec:
    name: str
    batch_size: int
    num_tokens: int
    num_heads: int
    head_dim: int
    rope_dim: int
    is_neox: bool


BENCH_CASES = (
    CaseSpec("flux_1024", 1, 4096, 24, 128, 128, False),
    CaseSpec("qwen_image_1024", 1, 4096, 32, 128, 128, False),
    CaseSpec("qwen_image_partial", 1, 4096, 32, 128, 64, False),
    # Z-Image-Turbo default 1024x1024 config: dim=3840, num_heads=30 -> head_dim=128.
    CaseSpec("zimage_1024", 1, 4096, 30, 128, 128, False),
    CaseSpec("batch2_medium", 2, 2048, 24, 128, 128, False),
)
CASE_BY_NAME = {case.name: case for case in BENCH_CASES}
CASE_NAMES = get_benchmark_range(
    full_range=[case.name for case in BENCH_CASES],
    ci_range=[case.name for case in BENCH_CASES],
)
LINE_VALS = ["split", "fused"]
LINE_NAMES = ["JIT QKNorm + FlashInfer RoPE", "SGL JIT Fused QKNorm+RoPE"]
STYLES = [("red", "-"), ("blue", "--")]
```
**EN:** This block declares the `CaseSpec` class and establishes the behavior or state it encapsulates. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段声明了 `CaseSpec` 类，并建立其封装的行为或状态。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 51-67: Function `create_cos_sin_cache`
```python
def create_cos_sin_cache(
    rotary_dim: int,
    max_position: int = MAX_SEQ_LEN,
    base: float = ROPE_BASE,
) -> torch.Tensor:
    inv_freq = 1.0 / (
        base
        ** (
            torch.arange(0, rotary_dim, 2, dtype=torch.float32, device=DEFAULT_DEVICE)
            / rotary_dim
        )
    )
    t = torch.arange(max_position, dtype=torch.float32, device=DEFAULT_DEVICE)
    freqs = torch.einsum("i,j->ij", t, inv_freq)
    return torch.cat((freqs.cos(), freqs.sin()), dim=-1)
```
**EN:** This block defines `create_cos_sin_cache`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `create_cos_sin_cache`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 68-119: Function `make_inputs`
```python
def make_inputs(case: CaseSpec) -> dict[str, torch.Tensor | bool]:
    seed = (
        case.batch_size * 1_000_003
        + case.num_tokens * 8191
        + case.num_heads * 127
        + case.head_dim * 17
        + case.rope_dim
    )
    generator = torch.Generator(device=DEFAULT_DEVICE)
    generator.manual_seed(seed)
    return {
        "q": torch.randn(
            case.batch_size * case.num_tokens,
            case.num_heads,
            case.head_dim,
            device=DEFAULT_DEVICE,
            dtype=DEFAULT_DTYPE,
            generator=generator,
        ),
        "k": torch.randn(
            case.batch_size * case.num_tokens,
            case.num_heads,
            case.head_dim,
            device=DEFAULT_DEVICE,
            dtype=DEFAULT_DTYPE,
            generator=generator,
        ),
        "q_weight": torch.randn(
            case.head_dim,
            device=DEFAULT_DEVICE,
            dtype=DEFAULT_DTYPE,
            generator=generator,
        ),
        "k_weight": torch.randn(
            case.head_dim,
            device=DEFAULT_DEVICE,
            dtype=DEFAULT_DTYPE,
            generator=generator,
        ),
        "positions": torch.randint(
            0,
            MAX_SEQ_LEN,
            (case.batch_size * case.num_tokens,),
            device=DEFAULT_DEVICE,
            dtype=torch.int64,
            generator=generator,
        ),
        "cos_sin_cache": create_cos_sin_cache(case.rope_dim),
        "is_neox": case.is_neox,
    }
```
**EN:** This block defines `make_inputs`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `make_inputs`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 120-128: Function `clone_inputs`
```python
def clone_inputs(
    inputs: dict[str, torch.Tensor | bool],
) -> dict[str, torch.Tensor | bool]:
    out: dict[str, torch.Tensor | bool] = {}
    for key, value in inputs.items():
        out[key] = value.clone() if isinstance(value, torch.Tensor) else value
    return out
```
**EN:** This block defines `clone_inputs`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `clone_inputs`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 129-152: Imports and module setup
```python
def split_qknorm_rope(inputs: dict[str, torch.Tensor | bool]) -> None:
    from flashinfer.rope import apply_rope_with_cos_sin_cache_inplace

    from sglang.jit_kernel.norm import fused_inplace_qknorm

    q = inputs["q"]
    k = inputs["k"]
    q_weight = inputs["q_weight"]
    k_weight = inputs["k_weight"]
    positions = inputs["positions"]
    cos_sin_cache = inputs["cos_sin_cache"]
    is_neox = bool(inputs["is_neox"])

    fused_inplace_qknorm(q, k, q_weight, k_weight)
    apply_rope_with_cos_sin_cache_inplace(
        positions=positions,
        query=q.view(q.shape[0], -1),
        key=k.view(k.shape[0], -1),
        head_size=q.shape[-1],
        cos_sin_cache=cos_sin_cache,
        is_neox=is_neox,
    )
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 153-180: Imports and module setup
```python
def fused_qknorm_rope(inputs: dict[str, torch.Tensor | bool]) -> None:
    from sglang.jit_kernel.diffusion.qknorm_rope import fused_inplace_qknorm_rope

    fused_inplace_qknorm_rope(
        inputs["q"],
        inputs["k"],
        inputs["q_weight"],
        inputs["k_weight"],
        inputs["cos_sin_cache"],
        inputs["positions"],
        is_neox=bool(inputs["is_neox"]),
        rope_dim=inputs["cos_sin_cache"].shape[-1],
    )


@triton.testing.perf_report(
    triton.testing.Benchmark(
        x_names=["case_name"],
        x_vals=CASE_NAMES,
        line_arg="provider",
        line_vals=LINE_VALS,
        line_names=LINE_NAMES,
        styles=STYLES,
        ylabel="us",
        plot_name="diffusion-qknorm-rope-performance",
        args={},
    )
)
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 181-190: Function `benchmark`
```python
def benchmark(case_name: str, provider: str) -> Tuple[float, float, float]:
    case = CASE_BY_NAME[case_name]
    inputs = make_inputs(case)
    fn = split_qknorm_rope if provider == "split" else fused_qknorm_rope
    return run_benchmark_no_cudagraph(lambda: fn(inputs))


if __name__ == "__main__":
    print("Running diffusion qknorm + rope performance benchmark...")
    benchmark.run(print_data=True)
```
**EN:** This block defines `benchmark`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `benchmark`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `dataclasses -> dataclass`
- `typing -> Tuple`
- `torch`
- `triton`
- `triton.testing`
- `sglang.jit_kernel.benchmark.utils -> (`
- `sglang.test.ci.ci_register -> register_cuda_ci`
- `flashinfer.rope -> apply_rope_with_cos_sin_cache_inplace`
- `sglang.jit_kernel.norm -> fused_inplace_qknorm`
- `sglang.jit_kernel.diffusion.qknorm_rope -> fused_inplace_qknorm_rope`
