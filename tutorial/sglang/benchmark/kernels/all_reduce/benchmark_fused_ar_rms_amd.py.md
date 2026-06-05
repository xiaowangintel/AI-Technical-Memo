# benchmark_fused_ar_rms_amd.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/kernels/all_reduce/benchmark_fused_ar_rms_amd.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on kernels all reduce fused ar. It primarily builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. / 该 Python 模块聚焦于 kernels all reduce fused ar 相关流程。它主要用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Line-by-Line Analysis / 逐行分析
### Lines 1-45: Imports and setup / 导入与初始化
```python
"""
Benchmark fused allreduce+rmsnorm on AMD with correctness checks.

This script targets the same fused op used by SGLang:
`tensor_model_parallel_fused_allreduce_rmsnorm`.

It reports:
- eager mode latency (prefill-like)
- graph mode latency (decode-like)
- fused availability (whether fused path returns non-None)
- correctness (fused output matches split allreduce + rmsnorm reference)

Usage example:
  torchrun --nproc_per_node=8 \
    benchmark/kernels/all_reduce/benchmark_fused_ar_rms_amd.py \
    --dtype bfloat16 \
    --prefill-shapes 2048x8192,8192x8192 \
    --decode-shapes 1x8192,4x8192,16x8192 \
    --warmup 10 --iters 30 --repeats 5
"""

import argparse
import csv
import os
import statistics
from typing import Dict, List, Optional, Sequence, Tuple

import torch
import torch.distributed as dist
import torch.nn.functional as F

from sglang.srt.distributed.communication_op import (
    tensor_model_parallel_all_reduce,
    tensor_model_parallel_fused_allreduce_rmsnorm,
)
from sglang.srt.distributed.parallel_state import (
    destroy_distributed_environment,
    destroy_model_parallel,
    graph_capture,
    init_distributed_environment,
    initialize_model_parallel,
    set_custom_all_reduce,
)

Shape = Tuple[int, int]
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

### Lines 48-61: Function `parse_shapes` / 函数 `parse_shapes`
```python
def parse_shapes(raw: str) -> List[Shape]:
    shapes: List[Shape] = []
    for item in [x.strip() for x in raw.split(",") if x.strip()]:
        if "x" not in item:
            raise ValueError(f"Invalid shape '{item}', expected MxN format.")
        m_str, n_str = item.split("x", 1)
        m = int(m_str)
        n = int(n_str)
        if m <= 0 or n <= 0:
            raise ValueError(f"Invalid shape '{item}', both dims must be positive.")
        shapes.append((m, n))
    if not shapes:
        raise ValueError("Empty shape list is not allowed.")
    return shapes
```
**EN:** `parse_shapes` is a function that implements the core logic for this scope. It returns `shapes` to the caller. Notable calls include `x.strip`, `item.split`, `int`.
**CN:** `parse_shapes` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `shapes`。其中较关键的调用包括 `x.strip`, `item.split`, `int`。

### Lines 64-73: Function `dtype_from_name` / 函数 `dtype_from_name`
```python
def dtype_from_name(name: str) -> torch.dtype:
    mapping = {
        "float16": torch.float16,
        "fp16": torch.float16,
        "bfloat16": torch.bfloat16,
        "bf16": torch.bfloat16,
    }
    if name not in mapping:
        raise ValueError(f"Unsupported dtype: {name}")
    return mapping[name]
```
**EN:** `dtype_from_name` is a function that prepares tensors and invokes GPU kernels and computes evaluation scores and aggregate statistics. It returns `mapping[name]` to the caller. Notable calls include `ValueError`.
**CN:** `dtype_from_name` 是一个函数，用于准备张量并调用 GPU 内核、计算评测分数与聚合统计结果。它会向调用方返回 `mapping[name]`。其中较关键的调用包括 `ValueError`。

### Lines 76-89: Function `check_close` / 函数 `check_close`
```python
def check_close(
    a: torch.Tensor, b: torch.Tensor, dtype: torch.dtype
) -> Tuple[bool, str]:
    if dtype == torch.bfloat16:
        rtol, atol = 2e-2, 1.25e-1
    else:
        rtol, atol = 1e-2, 2e-2
    try:
        torch.testing.assert_close(a, b, rtol=rtol, atol=atol)
        return True, "PASS"
    except AssertionError:
        max_diff = torch.max(torch.abs(a - b)).item()
        mean_diff = torch.mean(torch.abs(a - b)).item()
        return False, f"FAIL(max={max_diff:.6f},mean={mean_diff:.6f})"
```
**EN:** `check_close` is a function that prepares tensors and invokes GPU kernels. It returns `(True, 'PASS')` to the caller. Notable calls include `torch.testing.assert_close`, `torch.max(torch.abs(a - b)).item`, `torch.mean(torch.abs(a - b)).item`.
**CN:** `check_close` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `(True, 'PASS')`。其中较关键的调用包括 `torch.testing.assert_close`, `torch.max(torch.abs(a - b)).item`, `torch.mean(torch.abs(a - b)).item`。

### Lines 92-125: Function `_measure_us` / 函数 `_measure_us`
```python
def _measure_us(
    fn,
    warmup: int,
    iters: int,
    repeats: int,
    device: torch.device,
) -> Tuple[float, Dict[str, float]]:
    for _ in range(warmup):
        fn()
    torch.cuda.synchronize()

    start_event = torch.cuda.Event(enable_timing=True)
    end_event = torch.cuda.Event(enable_timing=True)
    samples_us: List[float] = []

    for _ in range(max(1, repeats)):
        _barrier(device)
        torch.cuda.synchronize()
        start_event.record()
        for _ in range(iters):
            fn()
        end_event.record()
        end_event.synchronize()
        samples_us.append(start_event.elapsed_time(end_event) * 1000.0 / iters)

    sorted_samples = sorted(samples_us)
    p50 = float(statistics.median(sorted_samples))
    p95 = float(sorted_samples[int((len(sorted_samples) - 1) * 0.95)])
    return p50, {
        "p50_us": p50,
        "p95_us": p95,
        "min_us": float(sorted_samples[0]),
        "max_us": float(sorted_samples[-1]),
    }
```
**EN:** `_measure_us` is a function that prepares tensors and invokes GPU kernels and loads, filters, or serializes benchmark datasets. It returns `(p50, {'p50_us': p50, 'p95_us': p95, 'min_us': float(sorted_samples[0]), 'max_us': float(sorted_s...` to the caller. Notable calls include `range`, `torch.cuda.synchronize`, `torch.cuda.Event`.
**CN:** `_measure_us` 是一个函数，用于准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集。它会向调用方返回 `(p50, {'p50_us': p50, 'p95_us': p95, 'min_us': float(sorted_samples[0]), 'max_us': float(sorted_s...`。其中较关键的调用包括 `range`, `torch.cuda.synchronize`, `torch.cuda.Event`。

### Lines 128-132: Function `_barrier` / 函数 `_barrier`
```python
def _barrier(device: torch.device):
    try:
        dist.barrier(device_ids=[device.index])
    except TypeError:
        dist.barrier()
```
**EN:** `_barrier` is a function that prepares tensors and invokes GPU kernels. Notable calls include `dist.barrier`.
**CN:** `_barrier` 是一个函数，用于准备张量并调用 GPU 内核。其中较关键的调用包括 `dist.barrier`。

### Lines 135-139: Function `_mean_across_ranks` / 函数 `_mean_across_ranks`
```python
def _mean_across_ranks(value: float, device: torch.device) -> float:
    t = torch.tensor([value], dtype=torch.float64, device=device)
    dist.all_reduce(t, op=dist.ReduceOp.SUM)
    t /= dist.get_world_size()
    return float(t.item())
```
**EN:** `_mean_across_ranks` is a function that prepares tensors and invokes GPU kernels. It returns `float(t.item())` to the caller. Notable calls include `torch.tensor`, `dist.all_reduce`, `dist.get_world_size`.
**CN:** `_mean_across_ranks` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `float(t.item())`。其中较关键的调用包括 `torch.tensor`, `dist.all_reduce`, `dist.get_world_size`。

### Lines 142-145: Function `_all_true_across_ranks` / 函数 `_all_true_across_ranks`
```python
def _all_true_across_ranks(value: bool, device: torch.device) -> bool:
    t = torch.tensor([1 if value else 0], dtype=torch.int32, device=device)
    dist.all_reduce(t, op=dist.ReduceOp.MIN)
    return bool(int(t.item()))
```
**EN:** `_all_true_across_ranks` is a function that prepares tensors and invokes GPU kernels. It returns `bool(int(t.item()))` to the caller. Notable calls include `torch.tensor`, `dist.all_reduce`, `bool`.
**CN:** `_all_true_across_ranks` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `bool(int(t.item()))`。其中较关键的调用包括 `torch.tensor`, `dist.all_reduce`, `bool`。

### Lines 148-168: Function `_make_inputs` / 函数 `_make_inputs`
```python
def _make_inputs(
    shape: Shape,
    dtype: torch.dtype,
    seed: int,
    residual_mode: str,
    rank: int,
    device: torch.device,
) -> Tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    m, n = shape
    torch.manual_seed(seed + rank * 17)
    x = torch.randn((m, n), dtype=torch.float32, device=device).to(dtype)
    if residual_mode == "self":
        residual = x.clone()
    elif residual_mode == "random":
        residual = torch.randn((m, n), dtype=torch.float32, device=device).to(dtype)
    elif residual_mode == "zero":
        residual = torch.zeros((m, n), dtype=dtype, device=device)
    else:
        raise ValueError(f"Unknown residual_mode: {residual_mode}")
    weight = torch.randn((n,), dtype=torch.float32, device=device).to(dtype)
    return x, residual, weight
```
**EN:** `_make_inputs` is a function that prepares tensors and invokes GPU kernels. It returns `(x, residual, weight)` to the caller. Notable calls include `torch.manual_seed`, `torch.randn((m, n), dtype=torch.float32, device=device).to`, `torch.randn((n,), dtype=torch.float32, device=device).to`.
**CN:** `_make_inputs` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `(x, residual, weight)`。其中较关键的调用包括 `torch.manual_seed`, `torch.randn((m, n), dtype=torch.float32, device=device).to`, `torch.randn((n,), dtype=torch.float32, device=device).to`。

### Lines 171-182: Function `_split_reference` / 函数 `_split_reference`
```python
def _split_reference(
    x: torch.Tensor, residual: torch.Tensor, weight: torch.Tensor, eps: float
) -> Tuple[torch.Tensor, torch.Tensor]:
    ar_out = tensor_model_parallel_all_reduce(x.clone())
    residual_out = ar_out + residual
    out = F.rms_norm(
        input=residual_out,
        normalized_shape=(residual_out.shape[-1],),
        weight=weight,
        eps=eps,
    )
    return out, residual_out
```
**EN:** `_split_reference` is a function that prepares tensors and invokes GPU kernels. It returns `(out, residual_out)` to the caller. Notable calls include `tensor_model_parallel_all_reduce`, `F.rms_norm`, `x.clone`.
**CN:** `_split_reference` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `(out, residual_out)`。其中较关键的调用包括 `tensor_model_parallel_all_reduce`, `F.rms_norm`, `x.clone`。

### Lines 185-231: Function `bench_eager` / 函数 `bench_eager`
```python
def bench_eager(
    x: torch.Tensor,
    residual: torch.Tensor,
    weight: torch.Tensor,
    eps: float,
    warmup: int,
    iters: int,
    repeats: int,
) -> Dict[str, object]:
    split_fn = lambda: _split_reference(x, residual, weight, eps)
    split_us, split_stats = _measure_us(split_fn, warmup, iters, repeats, x.device)

    fused_probe = tensor_model_parallel_fused_allreduce_rmsnorm(
        x.clone(), residual.clone(), weight, eps
    )
    fused_available = fused_probe is not None

    fused_us: Optional[float] = None
    fused_stats: Optional[Dict[str, float]] = None
    if fused_available:
        fused_fn = lambda: tensor_model_parallel_fused_allreduce_rmsnorm(
            x, residual, weight, eps
        )
        fused_us, fused_stats = _measure_us(fused_fn, warmup, iters, repeats, x.device)

    ref_out, ref_residual = _split_reference(x, residual, weight, eps)
    if fused_available:
        fused_out, fused_residual = tensor_model_parallel_fused_allreduce_rmsnorm(
            x.clone(), residual.clone(), weight, eps
        )
        out_ok, out_detail = check_close(fused_out, ref_out, x.dtype)
        res_ok, res_detail = check_close(fused_residual, ref_residual, x.dtype)
        correctness_ok = out_ok and res_ok
        correctness_detail = f"out={out_detail}, residual={res_detail}"
    else:
        correctness_ok = True
        correctness_detail = "SKIP(fused_unavailable)"

    return {
        "split_us": split_us,
        "split_stats": split_stats,
        "fused_available": fused_available,
        "fused_us": fused_us,
        "fused_stats": fused_stats,
        "correctness_ok": correctness_ok,
        "correctness_detail": correctness_detail,
    }
```
**EN:** `bench_eager` is a function that prepares tensors and invokes GPU kernels. It returns `{'split_us': split_us, 'split_stats': split_stats, 'fused_available': fused_available, 'fused_us'...` to the caller. Notable calls include `_measure_us`, `tensor_model_parallel_fused_allreduce_rmsnorm`, `_split_reference`.
**CN:** `bench_eager` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `{'split_us': split_us, 'split_stats': split_stats, 'fused_available': fused_available, 'fused_us'...`。其中较关键的调用包括 `_measure_us`, `tensor_model_parallel_fused_allreduce_rmsnorm`, `_split_reference`。

### Lines 234-310: Function `bench_graph` / 函数 `bench_graph`
```python
def bench_graph(
    x: torch.Tensor,
    residual: torch.Tensor,
    weight: torch.Tensor,
    eps: float,
    warmup: int,
    iters: int,
    repeats: int,
) -> Dict[str, object]:
    split_x = x.clone()
    split_res = residual.clone()
    split_graph_out: Optional[torch.Tensor] = None

    with graph_capture() as gc:
        split_graph = torch.cuda.CUDAGraph()
        with torch.cuda.graph(split_graph, stream=gc.stream):
            split_graph_out, _ = _split_reference(split_x, split_res, weight, eps)

    def split_replay():
        split_graph.replay()

    split_us, split_stats = _measure_us(split_replay, warmup, iters, repeats, x.device)

    fused_probe = tensor_model_parallel_fused_allreduce_rmsnorm(
        x.clone(), residual.clone(), weight, eps
    )
    fused_available = fused_probe is not None

    fused_us: Optional[float] = None
    fused_stats: Optional[Dict[str, float]] = None
    fused_graph_out: Optional[torch.Tensor] = None
    fused_graph_residual: Optional[torch.Tensor] = None

    if fused_available:
        fused_x = x.clone()
        fused_res = residual.clone()
        with graph_capture() as gc:
            fused_graph = torch.cuda.CUDAGraph()
            with torch.cuda.graph(fused_graph, stream=gc.stream):
                fused_graph_out, fused_graph_residual = (
                    tensor_model_parallel_fused_allreduce_rmsnorm(
                        fused_x, fused_res, weight, eps
                    )
                )

        def fused_replay():
            fused_graph.replay()

        fused_us, fused_stats = _measure_us(
            fused_replay, warmup, iters, repeats, x.device
        )

    ref_out, ref_residual = _split_reference(x, residual, weight, eps)
    if (
        fused_available
        and fused_graph_out is not None
        and fused_graph_residual is not None
    ):
        fused_graph.replay()
        torch.cuda.synchronize()
        out_ok, out_detail = check_close(fused_graph_out, ref_out, x.dtype)
        res_ok, res_detail = check_close(fused_graph_residual, ref_residual, x.dtype)
        correctness_ok = out_ok and res_ok
        correctness_detail = f"out={out_detail}, residual={res_detail}"
    else:
        correctness_ok = True
        correctness_detail = "SKIP(fused_unavailable)"

    return {
        "split_us": split_us,
        "split_stats": split_stats,
        "fused_available": fused_available,
        "fused_us": fused_us,
        "fused_stats": fused_stats,
        "correctness_ok": correctness_ok,
        "correctness_detail": correctness_detail,
    }
```
**EN:** `bench_graph` is a function that prepares tensors and invokes GPU kernels. It returns `{'split_us': split_us, 'split_stats': split_stats, 'fused_available': fused_available, 'fused_us'...` to the caller. Notable calls include `x.clone`, `residual.clone`, `_measure_us`.
**CN:** `bench_graph` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `{'split_us': split_us, 'split_stats': split_stats, 'fused_available': fused_available, 'fused_us'...`。其中较关键的调用包括 `x.clone`, `residual.clone`, `_measure_us`。

### Lines 313-315: Function `_shape_bytes` / 函数 `_shape_bytes`
```python
def _shape_bytes(shape: Shape, dtype: torch.dtype) -> int:
    m, n = shape
    return m * n * torch.tensor([], dtype=dtype).element_size()
```
**EN:** `_shape_bytes` is a function that prepares tensors and invokes GPU kernels. It returns `m * n * torch.tensor([], dtype=dtype).element_size()` to the caller. Notable calls include `torch.tensor([], dtype=dtype).element_size`, `torch.tensor`.
**CN:** `_shape_bytes` 是一个函数，用于准备张量并调用 GPU 内核。它会向调用方返回 `m * n * torch.tensor([], dtype=dtype).element_size()`。其中较关键的调用包括 `torch.tensor([], dtype=dtype).element_size`, `torch.tensor`。

### Lines 318-364: Function `parse_args` / 函数 `parse_args`
```python
def parse_args():
    parser = argparse.ArgumentParser(
        description="Benchmark fused allreduce+rmsnorm (prefill eager + decode graph)."
    )
    parser.add_argument(
        "--dtype",
        type=str,
        default="bf16",
        choices=["fp16", "bf16", "float16", "bfloat16"],
    )
    parser.add_argument("--eps", type=float, default=1e-6)
    parser.add_argument("--seed", type=int, default=1234)
    parser.add_argument(
        "--residual-mode",
        type=str,
        default="self",
        choices=["self", "random", "zero"],
        help="Use residual=x (self) to match aiter test behavior by default.",
    )
    parser.add_argument(
        "--prefill-shapes",
        type=str,
        default="2048x2880,2048x8192,8192x8192,16384x8192",
        help="Comma-separated MxN shapes for eager mode.",
    )
    parser.add_argument(
        "--decode-shapes",
        type=str,
        default="1x2880,4x2880,16x2880,1x8192,2x8192,4x8192,8x8192,16x8192",
        help="Comma-separated MxN shapes for graph mode.",
    )
    parser.add_argument("--warmup", type=int, default=10)
    parser.add_argument("--iters", type=int, default=30)
    parser.add_argument("--repeats", type=int, default=5)
    parser.add_argument(
        "--mode",
        type=str,
        default="both",
        choices=["eager", "graph", "both"],
    )
    parser.add_argument(
        "--csv-out",
        type=str,
        default=None,
        help="Optional output CSV path (written on rank 0 only).",
    )
    return parser.parse_args()
```
**EN:** `parse_args` is a function that builds command-line arguments and runtime configuration, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. It returns `parser.parse_args()` to the caller. Notable calls include `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`.
**CN:** `parse_args` 是一个函数，用于构建命令行参数与运行时配置、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。它会向调用方返回 `parser.parse_args()`。其中较关键的调用包括 `argparse.ArgumentParser`, `parser.add_argument`, `parser.parse_args`。

### Lines 367-532: Function `main` / 函数 `main`
```python
def main():
    args = parse_args()
    dtype = dtype_from_name(args.dtype)
    rank = int(os.environ.get("RANK", "0"))
    world_size = int(os.environ.get("WORLD_SIZE", "1"))
    local_rank = int(os.environ.get("LOCAL_RANK", str(rank)))
    torch.cuda.set_device(local_rank % torch.cuda.device_count())
    device = torch.device(f"cuda:{local_rank % torch.cuda.device_count()}")

    set_custom_all_reduce(True)
    init_distributed_environment(
        world_size=world_size,
        rank=rank,
        local_rank=local_rank,
        distributed_init_method="env://",
        backend="nccl",
    )
    initialize_model_parallel(tensor_model_parallel_size=world_size)

    prefill_shapes = parse_shapes(args.prefill_shapes)
    decode_shapes = parse_shapes(args.decode_shapes)

    if rank == 0:
        print(
            "Config: "
            f"world_size={world_size}, dtype={dtype}, residual_mode={args.residual_mode}, "
            f"warmup={args.warmup}, iters={args.iters}, repeats={args.repeats}"
        )

    run_modes: Sequence[str]
    if args.mode == "both":
        run_modes = ("eager", "graph")
    else:
        run_modes = (args.mode,)
    csv_rows: List[Dict[str, object]] = []

    for mode in run_modes:
        shapes = prefill_shapes if mode == "eager" else decode_shapes
        if rank == 0:
            phase_name = "prefill(eager)" if mode == "eager" else "decode(graph)"
            print("\n" + "=" * 120)
            print(f"Mode: {phase_name}")
            print(
                "| Shape | Input bytes/rank | Split p50 (us) | Fused p50 (us) | Speedup | Fused available | Correctness |"
            )
            print(
                "|:------|-----------------:|---------------:|---------------:|--------:|:----------------|:------------|"
            )

        for shape in shapes:
            x, residual, weight = _make_inputs(
                shape=shape,
                dtype=dtype,
                seed=args.seed,
                residual_mode=args.residual_mode,
                rank=rank,
                device=device,
            )

            if mode == "eager":
                metrics = bench_eager(
                    x=x,
                    residual=residual,
                    weight=weight,
                    eps=args.eps,
                    warmup=args.warmup,
                    iters=args.iters,
                    repeats=args.repeats,
                )
            else:
                metrics = bench_graph(
                    x=x,
                    residual=residual,
                    weight=weight,
                    eps=args.eps,
                    warmup=args.warmup,
                    iters=args.iters,
                    repeats=args.repeats,
                )

            split_us = _mean_across_ranks(float(metrics["split_us"]), device)
            fused_available = _all_true_across_ranks(
                bool(metrics["fused_available"]), device
            )
            correctness_ok = _all_true_across_ranks(
                bool(metrics["correctness_ok"]), device
            )

            fused_us: Optional[float] = None
            if fused_available and metrics["fused_us"] is not None:
                fused_us = _mean_across_ranks(float(metrics["fused_us"]), device)

            if rank == 0:
                m, n = shape
                shape_str = f"{m}x{n}"
                bytes_per_rank = _shape_bytes(shape, dtype)
                if fused_us is not None and fused_us > 0:
                    speedup = split_us / fused_us
                    speedup_str = f"{speedup:.3f}x"
                    fused_str = f"{fused_us:.1f}"
                else:
                    speedup_str = "N/A"
                    fused_str = "N/A"
                correctness_text = (
                    "PASS" if correctness_ok else str(metrics["correctness_detail"])
                )
                print(
                    f"| {shape_str} | {bytes_per_rank} | {split_us:.1f} | {fused_str} | "
                    f"{speedup_str} | {str(fused_available)} | {correctness_text} |"
                )
                csv_rows.append(
                    {
                        "mode": mode,
                        "shape": shape_str,
                        "m": m,
                        "n": n,
                        "bytes_per_rank": bytes_per_rank,
                        "split_p50_us": split_us,
                        "fused_p50_us": fused_us if fused_us is not None else "",
                        "speedup_split_over_fused": (
                            split_us / fused_us
                            if fused_us is not None and fused_us > 0
                            else ""
                        ),
                        "fused_available": fused_available,
                        "correctness_ok": correctness_ok,
                        "correctness_detail": correctness_text,
                        "dtype": str(dtype),
                        "world_size": world_size,
                        "residual_mode": args.residual_mode,
                        "warmup": args.warmup,
                        "iters": args.iters,
                        "repeats": args.repeats,
                    }
                )

    if rank == 0 and args.csv_out:
        os.makedirs(os.path.dirname(args.csv_out) or ".", exist_ok=True)
        fieldnames = [
            "mode",
            "shape",
            "m",
            "n",
            "bytes_per_rank",
            "split_p50_us",
            "fused_p50_us",
            "speedup_split_over_fused",
            "fused_available",
            "correctness_ok",
            "correctness_detail",
            "dtype",
            "world_size",
            "residual_mode",
            "warmup",
            "iters",
            "repeats",
        ]
        with open(args.csv_out, "w", newline="", encoding="utf-8") as f:
            writer = csv.DictWriter(f, fieldnames=fieldnames)
            writer.writeheader()
            writer.writerows(csv_rows)
        print(f"\nSaved CSV to: {args.csv_out}")

    _barrier(device)
    destroy_model_parallel()
    destroy_distributed_environment()
```
**EN:** `main` is a function that builds command-line arguments and runtime configuration, prepares tensors and invokes GPU kernels, and loads, filters, or serializes benchmark datasets. Notable calls include `parse_args`, `dtype_from_name`, `int`.
**CN:** `main` 是一个函数，用于构建命令行参数与运行时配置、准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集。其中较关键的调用包括 `parse_args`, `dtype_from_name`, `int`。

### Lines 535-536: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    main()
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `csv`, `os`, `statistics`, `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.distributed`, `torch.nn.functional`
- **Internal / 项目内部依赖**: `sglang.srt.distributed.communication_op`, `sglang.srt.distributed.parallel_state`
