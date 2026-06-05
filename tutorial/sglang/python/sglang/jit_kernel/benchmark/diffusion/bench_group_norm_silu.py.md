# bench_group_norm_silu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/benchmark/diffusion/bench_group_norm_silu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import argparse". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import argparse”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Imports and module setup
```python
import argparse
import csv
import statistics
import sys
from dataclasses import dataclass
from pathlib import Path
from typing import Callable

import torch
import torch.nn.functional as F
import triton.testing

from sglang.jit_kernel.diffusion.triton.group_norm_silu import triton_group_norm_silu
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.utils import is_in_ci

register_cuda_ci(
    est_time=45,
    suite="base-b-kernel-benchmark-1-gpu-large",
    disabled="standalone benchmark",
)

DEVICE = "cuda"
EPS = 1e-5
QUANTILES = [0.5, 0.2, 0.8]


@dataclass(frozen=True)
```
**EN:** This block imports dependencies and prepares names used later in the module. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 29-44: Class `Case`
```python
class Case:
    name: str
    shape: tuple[int, ...]
    num_groups: int


CASES = [
    Case("token_2d", (4, 128), 32),
    Case("image_2d", (2, 64, 32, 32), 32),
    Case("video_3d_small", (1, 64, 4, 16, 16), 32),
    Case("threshold_3d", (1, 128, 1, 256, 256), 32),
    Case("hunyuan_video_large", (1, 128, 20, 256, 256), 32),
]
CASE_BY_NAME = {case.name: case for case in CASES}
```
**EN:** This block declares the `Case` class and establishes the behavior or state it encapsulates. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段声明了 `Case` 类，并建立其封装的行为或状态。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 45-56: Function `dtype_from_name`
```python
def dtype_from_name(name: str) -> torch.dtype:
    mapping = {
        "bf16": torch.bfloat16,
        "bfloat16": torch.bfloat16,
        "fp16": torch.float16,
        "float16": torch.float16,
        "fp32": torch.float32,
        "float32": torch.float32,
    }
    return mapping[name]
```
**EN:** This block defines `dtype_from_name`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `dtype_from_name`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 57-65: Function `dtype_name`
```python
def dtype_name(dtype: torch.dtype) -> str:
    mapping = {
        torch.bfloat16: "bf16",
        torch.float16: "fp16",
        torch.float32: "fp32",
    }
    return mapping[dtype]
```
**EN:** This block defines `dtype_name`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `dtype_name`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 66-69: Function `parse_dtypes`
```python
def parse_dtypes(text: str) -> list[torch.dtype]:
    return [dtype_from_name(item.strip()) for item in text.split(",") if item.strip()]
```
**EN:** This block defines `parse_dtypes`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `parse_dtypes`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 70-79: Function `parse_cases`
```python
def parse_cases(text: str) -> list[Case]:
    if text == "all":
        return CASES
    names = [item.strip() for item in text.split(",") if item.strip()]
    missing = sorted(set(names) - CASE_BY_NAME.keys())
    if missing:
        raise ValueError(f"Unknown cases: {missing}")
    return [CASE_BY_NAME[name] for name in names]
```
**EN:** This block defines `parse_cases`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `parse_cases`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 80-87: Function `tolerance`
```python
def tolerance(dtype: torch.dtype) -> tuple[float, float]:
    if dtype == torch.float32:
        return 1e-5, 1e-5
    if dtype == torch.bfloat16:
        return 7e-2, 2e-2
    return 3e-3, 3e-3
```
**EN:** This block defines `tolerance`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `tolerance`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 88-96: Function `native_group_norm_silu`
```python
def native_group_norm_silu(
    x: torch.Tensor,
    weight: torch.Tensor,
    bias: torch.Tensor,
    num_groups: int,
) -> torch.Tensor:
    return F.silu(F.group_norm(x, num_groups, weight=weight, bias=bias, eps=EPS))
```
**EN:** This block defines `native_group_norm_silu`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `native_group_norm_silu`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 97-105: Function `make_inputs`
```python
def make_inputs(case: Case, dtype: torch.dtype) -> tuple[torch.Tensor, ...]:
    generator = torch.Generator(device=DEVICE)
    generator.manual_seed(len(case.shape) * 1009 + case.shape[1] * 17 + case.num_groups)
    x = torch.randn(case.shape, device=DEVICE, dtype=dtype, generator=generator)
    weight = torch.randn(case.shape[1], device=DEVICE, dtype=dtype, generator=generator)
    bias = torch.randn(case.shape[1], device=DEVICE, dtype=dtype, generator=generator)
    return x, weight, bias
```
**EN:** This block defines `make_inputs`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `make_inputs`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 106-115: Function `do_bench_us`
```python
def do_bench_us(fn: Callable[[], object], warmup: int, rep: int) -> tuple[float, ...]:
    median_ms, p20_ms, p80_ms = triton.testing.do_bench(
        fn,
        quantiles=QUANTILES,
        warmup=warmup,
        rep=rep,
    )
    return median_ms * 1000.0, p20_ms * 1000.0, p80_ms * 1000.0
```
**EN:** This block defines `do_bench_us`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `do_bench_us`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 116-119: Function `summarize`
```python
def summarize(values: list[float]) -> float:
    return statistics.median(values)
```
**EN:** This block defines `summarize`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `summarize`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 120-177: Function `run_case`
```python
def run_case(
    case: Case,
    dtype: torch.dtype,
    rounds: int,
    warmup: int,
    rep: int,
) -> dict[str, object]:
    x, weight, bias = make_inputs(case, dtype)

    with torch.inference_mode():
        actual = triton_group_norm_silu(
            x, weight, bias, num_groups=case.num_groups, eps=EPS
        )
        expected = native_group_norm_silu(x, weight, bias, case.num_groups)
        atol, rtol = tolerance(dtype)
        torch.testing.assert_close(actual, expected, atol=atol, rtol=rtol)

        native_stats = []
        fused_stats = []
        for _ in range(rounds):
            native_stats.append(
                do_bench_us(
                    lambda: native_group_norm_silu(x, weight, bias, case.num_groups),
                    warmup=warmup,
                    rep=rep,
                )
            )
            fused_stats.append(
                do_bench_us(
                    lambda: triton_group_norm_silu(
                        x, weight, bias, num_groups=case.num_groups, eps=EPS
                    ),
                    warmup=warmup,
                    rep=rep,
                )
            )

    native_median_us = summarize([stats[0] for stats in native_stats])
    fused_median_us = summarize([stats[0] for stats in fused_stats])
    torch.cuda.empty_cache()
    return {
        "case": case.name,
        "shape": "x".join(str(dim) for dim in case.shape),
        "groups": case.num_groups,
        "dtype": dtype_name(dtype),
        "native_median_us": native_median_us,
        "native_p20_us": summarize([stats[1] for stats in native_stats]),
        "native_p80_us": summarize([stats[2] for stats in native_stats]),
        "fused_median_us": fused_median_us,
        "fused_p20_us": summarize([stats[1] for stats in fused_stats]),
        "fused_p80_us": summarize([stats[2] for stats in fused_stats]),
        "speedup": native_median_us / fused_median_us,
        "rounds": rounds,
        "warmup": warmup,
        "rep": rep,
    }
```
**EN:** This block defines `run_case`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `run_case`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 178-204: Function `run_profile`
```python
def run_profile(case: Case, dtype: torch.dtype, provider: str, iters: int) -> None:
    x, weight, bias = make_inputs(case, dtype)

    if provider == "native":

        def fn() -> torch.Tensor:
            return native_group_norm_silu(x, weight, bias, case.num_groups)

    elif provider == "fused":

        def fn() -> torch.Tensor:
            return triton_group_norm_silu(
                x, weight, bias, num_groups=case.num_groups, eps=EPS
            )

    else:
        raise ValueError(f"Unknown provider: {provider}")

    with torch.inference_mode():
        for _ in range(5):
            fn()
        torch.cuda.synchronize()
        for _ in range(iters):
            fn()
        torch.cuda.synchronize()
```
**EN:** This block defines `run_profile`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `run_profile`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 205-213: Function `write_csv`
```python
def write_csv(rows: list[dict[str, object]], output_path: Path) -> None:
    output_path.parent.mkdir(parents=True, exist_ok=True)
    fieldnames = list(rows[0].keys()) if rows else []
    with output_path.open("w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(rows)
```
**EN:** This block defines `write_csv`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `write_csv`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 214-237: Function `print_rows`
```python
def print_rows(rows: list[dict[str, object]]) -> None:
    header = (
        "case",
        "dtype",
        "shape",
        "native_us",
        "fused_us",
        "speedup",
    )
    print("| " + " | ".join(header) + " |")
    print("|---|---|---|---:|---:|---:|")
    for row in rows:
        print(
            "| {case} | {dtype} | {shape} | {native:.2f} | {fused:.2f} | {speedup:.3f}x |".format(
                case=row["case"],
                dtype=row["dtype"],
                shape=row["shape"],
                native=row["native_median_us"],
                fused=row["fused_median_us"],
                speedup=row["speedup"],
            )
        )
```
**EN:** This block defines `print_rows`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `print_rows`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 238-281: Function `main`
```python
def main() -> None:
    parser = argparse.ArgumentParser(
        description="Benchmark fused GroupNorm+SiLU against PyTorch GroupNorm+SiLU."
    )
    parser.add_argument("--cases", default="all")
    parser.add_argument("--dtypes", default="bf16,fp16")
    parser.add_argument("--rounds", type=int, default=3)
    parser.add_argument("--warmup", type=int, default=25)
    parser.add_argument("--rep", type=int, default=100)
    parser.add_argument("--output-csv", default="")
    parser.add_argument("--profile-provider", choices=["native", "fused"], default="")
    parser.add_argument("--profile-iters", type=int, default=20)
    args = parser.parse_args()

    if not torch.cuda.is_available():
        raise RuntimeError("CUDA is required for this benchmark.")

    cases = parse_cases(args.cases)
    dtypes = parse_dtypes(args.dtypes)

    if args.profile_provider:
        if len(cases) != 1 or len(dtypes) != 1:
            raise ValueError(
                "--profile-provider requires exactly one case and one dtype"
            )
        run_profile(cases[0], dtypes[0], args.profile_provider, args.profile_iters)
        return

    rows = []
    for case in cases:
        for dtype in dtypes:
            rows.append(run_case(case, dtype, args.rounds, args.warmup, args.rep))

    print_rows(rows)
    if args.output_csv:
        write_csv(rows, Path(args.output_csv))
        print(f"Wrote {args.output_csv}")


if __name__ == "__main__":
    if is_in_ci():
        print("Skipping bench_group_norm_silu.py in CI")
        sys.exit(0)
    main()
```
**EN:** This block defines `main`, including its interface and main implementation logic. Given the path, it likely contributes to performance measurement or comparison. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `main`，包括其接口以及主要实现逻辑。 结合路径来看，它很可能用于性能测量或对比。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `argparse`
- `csv`
- `statistics`
- `sys`
- `dataclasses -> dataclass`
- `pathlib -> Path`
- `typing -> Callable`
- `torch`
- `torch.nn.functional as F`
- `triton.testing`
- `sglang.jit_kernel.diffusion.triton.group_norm_silu -> triton_group_norm_silu`
- `sglang.test.ci.ci_register -> register_cuda_ci`
- `sglang.utils -> is_in_ci`
