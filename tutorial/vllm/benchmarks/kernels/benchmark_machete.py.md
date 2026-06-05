# benchmark_machete.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/kernels/benchmark_machete.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, centered around `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `DEFAULT_TP_SIZES`, `NVTX_PROFILE`. / 实现与基准测试编排相关的逻辑，核心符号包括 `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `DEFAULT_TP_SIZES`, `NVTX_PROFILE`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-36)
```python
import argparse
import copy
import itertools
import math
import os
import pickle as pkl
import time
from collections.abc import Callable, Iterable
from dataclasses import dataclass
from itertools import product

import pandas as pd
import torch
import torch.utils.benchmark as TBenchmark
from torch.utils.benchmark import Measurement as TMeasurement
from weight_shapes import WEIGHT_SHAPES

from vllm import _custom_ops as ops
from vllm.model_executor.layers.quantization.utils.marlin_utils import (
    GPTQ_MARLIN_MAX_PARALLEL,
    GPTQ_MARLIN_MIN_THREAD_N,
    marlin_permute_scales,
    marlin_zero_points,
)
from vllm.model_executor.layers.quantization.utils.marlin_utils_test import (
    MarlinWorkspace,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    pack_rows,
    quantize_weights,
)
from vllm.scalar_type import ScalarType, scalar_types
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block gathers standard-library helpers such as `argparse`, `copy`, `itertools`, `math`, `os`, `pickle`; third-party packages such as `pandas`, `torch`, `torch.utils.benchmark`; project-local modules such as `weight_shapes`, `vllm`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils_test`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.scalar_type`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `copy`, `itertools`, `math`, `os`, `pickle`；第三方依赖，如 `pandas`, `torch`, `torch.utils.benchmark`；项目内部模块，如 `weight_shapes`, `vllm`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils_test`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.scalar_type`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 38-45)
```python
DEFAULT_MODELS = ["meta-llama/Llama-3-8b", "meta-llama/Llama-2-70b-hf"]
DEFAULT_BATCH_SIZES = [1, 16, 32, 64, 128, 256, 512, 1024]
DEFAULT_TP_SIZES = [1]

NVTX_PROFILE = os.environ.get("NVTX_PROFILE", False)

if NVTX_PROFILE:
    import nvtx
```
**EN:** This top-level block prepares shared state such as `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `DEFAULT_TP_SIZES`, `NVTX_PROFILE`. It uses `os.environ.get` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `DEFAULT_MODELS`, `DEFAULT_BATCH_SIZES`, `DEFAULT_TP_SIZES`, `NVTX_PROFILE`。它借助 `os.environ.get` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `terse_type_name` (lines 48-56)
```python
def terse_type_name(dt):
    return {
        torch.bfloat16: "bf16",
        torch.float16: "fp16",
        torch.int8: "int8",
        torch.float8_e4m3fn: "fp8",
        torch.float: "float",
        torch.int: "int",
    }[dt]
```
**EN:** `terse_type_name` implements a helper used by `benchmark_machete.py`. It mainly works with `dt` and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `terse_type_name` 负责实现 `benchmark_machete.py` 使用的辅助逻辑。 它主要处理 `dt`，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Class `BenchmarkTensors` (lines 60-70)
```python
class BenchmarkTensors:
    w_ref: torch.Tensor
    a: torch.Tensor

    w_q: torch.Tensor
    group_size: int | None
    wtype: ScalarType
    w_g_s: torch.Tensor
    w_g_zp: torch.Tensor | None
    w_ch_s: torch.Tensor | None
    w_tok_s: torch.Tensor | None
```
**EN:** Class `BenchmarkTensors` packages shared state and related operations for this benchmark module. It extends `object` and exposes methods such as its methods.
**CN:** 类 `BenchmarkTensors` 用于封装该基准模块的共享状态与相关操作。它继承自 `object`，并提供 its methods 等方法。

### Class `TypeConfig` (lines 74-81)
```python
class TypeConfig:
    act_type: torch.dtype
    weight_type: ScalarType
    output_type: torch.dtype | None
    group_scale_type: torch.dtype | None
    group_zero_type: torch.dtype | None
    channel_scale_type: torch.dtype | None
    token_scale_type: torch.dtype | None
```
**EN:** Class `TypeConfig` packages shared state and related operations for this benchmark module. It extends `object` and exposes methods such as its methods.
**CN:** 类 `TypeConfig` 用于封装该基准模块的共享状态与相关操作。它继承自 `object`，并提供 its methods 等方法。

### Function `rand_data` (lines 84-88)
```python
def rand_data(shape, dtype=torch.float16, scale=1):
    if dtype.is_floating_point:
        return (scale * torch.rand(shape, device="cuda") - 0.3).to(dtype)
    else:
        return torch.randint(-15, 15, shape, dtype=dtype, device="cuda")
```
**EN:** `rand_data` implements a helper used by `benchmark_machete.py`. It mainly works with `shape`, `dtype`, `scale` and relies on `to`, `torch.rand`, `torch.randint` plus branching to move data through this part of the benchmark pipeline.
**CN:** `rand_data` 负责实现 `benchmark_machete.py` 使用的辅助逻辑。 它主要处理 `shape`, `dtype`, `scale`，并结合 `to`, `torch.rand`, `torch.randint` 以及 条件分支 来完成这一段基准测试流程。

### Function `quantize_and_pack` (lines 91-111)
```python
def quantize_and_pack(
    atype: torch.dtype,
    w: torch.Tensor,
    wtype: ScalarType,
    stype: torch.dtype | None,
    group_size: int | None,
    zero_points: bool = False,
):
    assert wtype.is_integer(), "TODO: support floating point weights"

    w_ref, w_q, w_s, w_zp = quantize_weights(
        w,
        wtype,
        group_size=group_size,
        zero_points=zero_points,
        # to match how the kernel applies zps
        ref_zero_points_after_scales=True,
    )

    w_q = pack_rows(w_q, wtype.size_bits, *w_q.shape)
    return w_ref, w_q, w_s, w_zp
```
**EN:** `quantize_and_pack` coordinates or measures quantization benchmarks. It mainly works with `atype`, `w`, `wtype`, `stype`, `group_size`, ... and relies on `wtype.is_integer`, `quantize_weights`, `pack_rows` plus value production to move data through this part of the benchmark pipeline.
**CN:** `quantize_and_pack` 负责协调或测量量化基准。 它主要处理 `atype`, `w`, `wtype`, `stype`, `group_size`, ...，并结合 `wtype.is_integer`, `quantize_weights`, `pack_rows` 以及 结果返回 来完成这一段基准测试流程。

### Function `create_bench_tensors` (lines 114-177)
```python
def create_bench_tensors(
    shape: tuple[int, int, int], types: TypeConfig, group_size: int | None
) -> list[BenchmarkTensors]:
    m, n, k = shape

    # we want to make sure that weights don't fit into L2 cache between runs so
    #  we construct enough weights to exceed L2 cache, which is 50mb on a H100
    #  so we target total weight size > 2*50mb
    num_weights = math.ceil(
        2 * 50 * 1024**2 * 8 / (k * n * types.weight_type.size_bits)
    )

    a = rand_data((m, k), types.act_type, scale=5)

    benchmark_tensors: list[BenchmarkTensors] = []
    for _ in range(num_weights):
        w = rand_data((k, n), types.act_type, scale=5)

        if types.group_scale_type is not None:
            w = w.to(types.group_scale_type)
        if w.dtype.itemsize == 1:
            w = w.to(torch.float16)

        w_ref, w_q_packed, w_s, w_zp = quantize_and_pack(
            a.dtype,
            w,
            types.weight_type,
            types.group_scale_type,
            group_size,
            types.group_zero_type is not None,
        )

        if not a.dtype.is_floating_point:
            aiinfo = torch.iinfo(a.dtype)
    # ... omitted for brevity ...
                w_g_zp=w_zp,
                group_size=group_size,
                w_ch_s=w_ch_s,
                w_tok_s=w_tok_s,
            )
        )

    return benchmark_tensors
```
**EN:** `create_bench_tensors` constructs shared runtime objects for the benchmark. It mainly works with `shape`, `types`, `group_size` and relies on `math.ceil`, `rand_data`, `range`, `w.to`, `quantize_and_pack`, `torch.iinfo` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `create_bench_tensors` 负责构建基准测试共享的运行时对象。 它主要处理 `shape`, `types`, `group_size`，并结合 `math.ceil`, `rand_data`, `range`, `w.to`, `quantize_and_pack`, `torch.iinfo` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `torch_matmul_f16_create_bench_fn` (lines 180-186)
```python
def torch_matmul_f16_create_bench_fn(bt: BenchmarkTensors) -> Callable:
    a = bt.a
    w = bt.w_ref.to(bt.a.dtype)  # use float reference tensor
    if a.dtype not in [torch.float16, torch.bfloat16]:
        a = a.to(torch.float16)
        w = w.to(torch.float16)
    return lambda: torch.matmul(a, w)
```
**EN:** `torch_matmul_f16_create_bench_fn` implements a helper used by `benchmark_machete.py`. It mainly works with `bt` and relies on `bt.w_ref.to`, `a.to`, `w.to`, `torch.matmul` plus branching to move data through this part of the benchmark pipeline.
**CN:** `torch_matmul_f16_create_bench_fn` 负责实现 `benchmark_machete.py` 使用的辅助逻辑。 它主要处理 `bt`，并结合 `bt.w_ref.to`, `a.to`, `w.to`, `torch.matmul` 以及 条件分支 来完成这一段基准测试流程。

### Function `cutlass_scaled_mm_create_bench_fn` (lines 189-199)
```python
def cutlass_scaled_mm_create_bench_fn(bt: BenchmarkTensors) -> Callable:
    if bt.w_ch_s is not None and bt.w_tok_s is not None:
        scale_a = bt.w_tok_s.to(torch.float32)
        scale_b = bt.w_ch_s.to(torch.float32)
    else:
        scale_a = torch.tensor(1.0, dtype=torch.float32, device=bt.a.device)
        scale_b = torch.tensor(1.0, dtype=torch.float32, device=bt.a.device)
    w_col_major = bt.w_ref.to(bt.a.dtype).t().contiguous().t()
    return lambda: ops.cutlass_scaled_mm(
        bt.a, w_col_major, scale_a, scale_b, out_dtype=torch.float16
    )
```
**EN:** `cutlass_scaled_mm_create_bench_fn` implements multimodal processing benchmarks. It mainly works with `bt` and relies on `bt.w_tok_s.to`, `bt.w_ch_s.to`, `torch.tensor`, `bt.w_ref.to.t.contiguous.t`, `bt.w_ref.to.t.contiguous`, `bt.w_ref.to.t` plus branching to move data through this part of the benchmark pipeline.
**CN:** `cutlass_scaled_mm_create_bench_fn` 负责实现多模态处理基准。 它主要处理 `bt`，并结合 `bt.w_tok_s.to`, `bt.w_ch_s.to`, `torch.tensor`, `bt.w_ref.to.t.contiguous.t`, `bt.w_ref.to.t.contiguous`, `bt.w_ref.to.t` 以及 条件分支 来完成这一段基准测试流程。

### Function `marlin_create_bench_fn` (lines 202-258)
```python
def marlin_create_bench_fn(bt: BenchmarkTensors) -> Callable:
    device = bt.a.device

    workspace = MarlinWorkspace(
        bt.w_ref.shape[1], GPTQ_MARLIN_MIN_THREAD_N, GPTQ_MARLIN_MAX_PARALLEL
    )

    if bt.w_g_zp is None:
        w_zp = torch.empty(0, dtype=torch.int, device=device)
    else:
        w_zp = marlin_zero_points(
            bt.w_g_zp, bt.w_ref.shape[0], bt.w_ref.shape[1], bt.wtype.size_bits
        )

    if bt.group_size is None:
        w_s = torch.tensor([], device="cuda", dtype=torch.half)
    else:
        w_s = marlin_permute_scales(
            bt.w_g_s, bt.w_ref.shape[0], bt.w_ref.shape[1], bt.group_size
        )

    sort_indices = torch.empty(0, dtype=torch.int, device=device)
    g_idx = torch.empty(0, dtype=torch.int, device=device)
    w_q = ops.gptq_marlin_repack(
        bt.w_q, sort_indices, bt.w_ref.shape[0], bt.w_ref.shape[1], bt.wtype.size_bits
    )

    if bt.a.dtype.is_floating_point:
        assert bt.w_ch_s is None
        assert bt.w_tok_s is None
        assert bt.group_size is not None

        fn = lambda: ops.marlin_gemm(
            a=bt.a,
    # ... omitted for brevity ...
            is_zp_float=False,
        )
    else:
        assert bt.a.dtype == torch.int8
        assert bt.wtype == scalar_types.uint4b8
        raise NotImplementedError("QQQ is not supported anymore")

    return fn
```
**EN:** `marlin_create_bench_fn` implements Marlin kernel experiments. It mainly works with `bt` and relies on `MarlinWorkspace`, `torch.empty`, `marlin_zero_points`, `torch.tensor`, `marlin_permute_scales`, `ops.gptq_marlin_repack` plus branching to move data through this part of the benchmark pipeline.
**CN:** `marlin_create_bench_fn` 负责实现Marlin 内核实验。 它主要处理 `bt`，并结合 `MarlinWorkspace`, `torch.empty`, `marlin_zero_points`, `torch.tensor`, `marlin_permute_scales`, `ops.gptq_marlin_repack` 以及 条件分支 来完成这一段基准测试流程。

### Function `machete_create_bench_fn` (lines 261-284)
```python
def machete_create_bench_fn(
    bt: BenchmarkTensors, out_type=torch.dtype, schedule=None
) -> Callable:
    w_q = bt.w_q.t().contiguous().t()  # make col major
    w_q = ops.machete_prepack_B(
        w_q, bt.a.dtype, bt.wtype, None if bt.w_g_s is None else bt.w_g_s.dtype
    )

    w_g_zp = bt.w_g_zp
    if w_g_zp is not None:
        w_g_zp = -1 * bt.w_g_s * (w_g_zp.to(bt.w_g_s.dtype))

    return lambda: ops.machete_mm(
        a=bt.a,
        b_q=w_q,
        b_type=bt.wtype,
        b_group_scales=bt.w_g_s,
        b_group_zeros=w_g_zp,
        b_group_size=bt.group_size,
        b_channel_scales=bt.w_ch_s,
        a_token_scales=bt.w_tok_s,
        out_type=out_type,
        schedule=schedule,
    )
```
**EN:** `machete_create_bench_fn` implements a helper used by `benchmark_machete.py`. It mainly works with `bt`, `out_type`, `schedule` and relies on `bt.w_q.t.contiguous.t`, `bt.w_q.t.contiguous`, `bt.w_q.t`, `ops.machete_prepack_B`, `w_g_zp.to`, `ops.machete_mm` plus branching to move data through this part of the benchmark pipeline.
**CN:** `machete_create_bench_fn` 负责实现 `benchmark_machete.py` 使用的辅助逻辑。 它主要处理 `bt`, `out_type`, `schedule`，并结合 `bt.w_q.t.contiguous.t`, `bt.w_q.t.contiguous`, `bt.w_q.t`, `ops.machete_prepack_B`, `w_g_zp.to`, `ops.machete_mm` 以及 条件分支 来完成这一段基准测试流程。

### Function `cutlass_w4a8_create_bench_fn` (lines 287-303)
```python
def cutlass_w4a8_create_bench_fn(
    bt: BenchmarkTensors, out_type=torch.dtype, schedule=None
) -> Callable:
    w_q = bt.w_q.t().contiguous().t()  # make col major
    w_q = ops.cutlass_encode_and_reorder_int4b(w_q)
    # expects fp8 scales
    w_s = ops.cutlass_pack_scale_fp8(bt.w_g_s.to(torch.float8_e4m3fn))

    return lambda: ops.cutlass_w4a8_mm(
        a=bt.a,
        b_q=w_q,
        b_group_scales=w_s,
        b_group_size=bt.group_size,
        b_channel_scales=bt.w_ch_s,
        a_token_scales=bt.w_tok_s,
        maybe_schedule=schedule,
    )
```
**EN:** `cutlass_w4a8_create_bench_fn` implements a helper used by `benchmark_machete.py`. It mainly works with `bt`, `out_type`, `schedule` and relies on `bt.w_q.t.contiguous.t`, `bt.w_q.t.contiguous`, `bt.w_q.t`, `ops.cutlass_encode_and_reorder_int4b`, `ops.cutlass_pack_scale_fp8`, `bt.w_g_s.to` plus value production to move data through this part of the benchmark pipeline.
**CN:** `cutlass_w4a8_create_bench_fn` 负责实现 `benchmark_machete.py` 使用的辅助逻辑。 它主要处理 `bt`, `out_type`, `schedule`，并结合 `bt.w_q.t.contiguous.t`, `bt.w_q.t.contiguous`, `bt.w_q.t`, `ops.cutlass_encode_and_reorder_int4b`, `ops.cutlass_pack_scale_fp8`, `bt.w_g_s.to` 以及 结果返回 来完成这一段基准测试流程。

### Function `bench_fns` (lines 311-331)
```python
def bench_fns(label: str, sub_label: str, description: str, fns: list[Callable]):
    min_run_time = 1 if not NVTX_PROFILE else 0.1
    res = TBenchmark.Timer(
        stmt="""
        for fn in fns:
            fn()
        """,
        globals={"fns": fns},
        label=label,
        sub_label=sub_label,
        description=description,
    ).blocked_autorange(min_run_time=min_run_time)

    if NVTX_PROFILE:
        with (
            nvtx.annotate("mm-bench"),
            nvtx.annotate(f"{label}|{sub_label}|{description}"),
        ):
            fns[0]()

    return res
```
**EN:** `bench_fns` implements a helper used by `benchmark_machete.py`. It mainly works with `label`, `sub_label`, `description`, `fns` and relies on `TBenchmark.Timer.blocked_autorange`, `TBenchmark.Timer`, `nvtx.annotate`, `fns` plus branching and context management to move data through this part of the benchmark pipeline.
**CN:** `bench_fns` 负责实现 `benchmark_machete.py` 使用的辅助逻辑。 它主要处理 `label`, `sub_label`, `description`, `fns`，并结合 `TBenchmark.Timer.blocked_autorange`, `TBenchmark.Timer`, `nvtx.annotate`, `fns` 以及 条件分支、上下文管理 来完成这一段基准测试流程。

### Top-level setup (lines 334-335)
```python
_SWEEP_SCHEDULES_RESULTS: pd.DataFrame | None = None
_SWEEP_SCHEDULES_RESULTS_CSV: str | None = None
```
**EN:** This top-level block prepares shared state such as `_SWEEP_SCHEDULES_RESULTS`, `_SWEEP_SCHEDULES_RESULTS_CSV`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `_SWEEP_SCHEDULES_RESULTS`, `_SWEEP_SCHEDULES_RESULTS_CSV`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `bench` (lines 338-478)
```python
def bench(
    types: TypeConfig,
    group_size: int,
    m: int,
    k: int,
    n: int,
    label: str,
    sub_label: str,
    sweep_schedules: bool = True,
) -> list[TMeasurement]:
    benchmark_tensors = create_bench_tensors((m, n, k), types, group_size)
    sub_label += f", L={len(benchmark_tensors)}"

    name_type_string = f"W{types.weight_type}" + f"-A{terse_type_name(types.act_type)}"
    if types.group_scale_type is not None:
        name_type_string += f"-GS{terse_type_name(types.group_scale_type)}"
    if types.group_zero_type is not None:
        name_type_string += f"-GZ{terse_type_name(types.group_zero_type)}"
    if group_size is not None:
        name_type_string += f"-G{group_size}"
    if types.channel_scale_type is not None:
        name_type_string += f"-CS{terse_type_name(types.channel_scale_type)}"
    if types.token_scale_type is not None:
        name_type_string += f"-TS{terse_type_name(types.token_scale_type)}"

    timers = []
    # pytorch impl
    timers.append(
        bench_fns(
            label,
            sub_label,
            "torch.matmul (fp16)",
            [torch_matmul_f16_create_bench_fn(bt) for bt in benchmark_tensors],
        )
    # ... omitted for brevity ...
            print(f"  {res.median:5.5} ", schedule)
            if not best or res.median < best.median:
                best = res
                best_schedule = schedule
        print("Best schedule:", best_schedule)
        timers.append(best)

    return timers
```
**EN:** `bench` implements a helper used by `benchmark_machete.py`. It mainly works with `types`, `group_size`, `m`, `k`, `n`, ... and relies on `create_bench_tensors`, `len`, `terse_type_name`, `timers.append`, `bench_fns`, `torch_matmul_f16_create_bench_fn` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `bench` 负责实现 `benchmark_machete.py` 使用的辅助逻辑。 它主要处理 `types`, `group_size`, `m`, `k`, `n`, ...，并结合 `create_bench_tensors`, `len`, `terse_type_name`, `timers.append`, `bench_fns`, `torch_matmul_f16_create_bench_fn` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `print_timers` (lines 482-484)
```python
def print_timers(timers: list[TMeasurement]):
    compare = TBenchmark.Compare(timers)
    compare.print()
```
**EN:** `print_timers` formats results for display or export. It mainly works with `timers` and relies on `TBenchmark.Compare`, `compare.print` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `print_timers` 负责格式化结果以便展示或导出。 它主要处理 `timers`，并结合 `TBenchmark.Compare`, `compare.print` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Function `run` (lines 487-515)
```python
def run(args, MKNs: Iterable[tuple[int, int, int]]) -> Iterable[TMeasurement]:
    types = TypeConfig(
        act_type=args.act_type,
        weight_type=scalar_types.uint4b8
        if args.group_zero_type is None
        else scalar_types.uint4,
        output_type=args.out_type,
        group_scale_type=args.group_scale_type,
        group_zero_type=args.group_zero_type,
        channel_scale_type=args.channel_scale_type,
        token_scale_type=args.token_scale_type,
    )

    results: list[TMeasurement] = []
    for m, k, n in MKNs:
        timers = bench(
            types,
            args.group_size,
            m,
            k,
            n,
            f"{args.act_type}-gemm",
            f"MKN=({m}x{k}x{n})",
            sweep_schedules=args.sweep_schedules,
        )
        print_timers(timers)
        results.extend(timers)

    return results
```
**EN:** `run` implements a helper used by `benchmark_machete.py`. It mainly works with `args`, `MKNs` and relies on `TypeConfig`, `bench`, `print_timers`, `results.extend` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `run` 负责实现 `benchmark_machete.py` 使用的辅助逻辑。 它主要处理 `args`, `MKNs`，并结合 `TypeConfig`, `bench`, `print_timers`, `results.extend` 以及 循环迭代 来完成这一段基准测试流程。

### Function `make_output` (lines 519-531)
```python
def make_output(
    data: list[TMeasurement],
    MKNs: Iterable[tuple[int, int, int]],
    base_description: str,
    timestamp=None,
):
    print(f"== All Results {base_description} ====")
    print_timers(data)

    # pickle all the results
    timestamp = int(time.time()) if timestamp is None else timestamp
    with open(f"{base_description}-{timestamp}.pkl", "wb") as f:
        pkl.dump(data, f)
```
**EN:** `make_output` implements a helper used by `benchmark_machete.py`. It mainly works with `data`, `MKNs`, `base_description`, `timestamp` and relies on `print`, `print_timers`, `int`, `time.time`, `open`, `pkl.dump` plus context management to move data through this part of the benchmark pipeline.
**CN:** `make_output` 负责实现 `benchmark_machete.py` 使用的辅助逻辑。 它主要处理 `data`, `MKNs`, `base_description`, `timestamp`，并结合 `print`, `print_timers`, `int`, `time.time`, `open`, `pkl.dump` 以及 上下文管理 来完成这一段基准测试流程。

### Function `run_square_bench` (lines 537-542)
```python
def run_square_bench(args):
    dim_sizes = list(range(args.dim_start, args.dim_end + 1, args.dim_increment))
    MKNs = list(zip(dim_sizes, dim_sizes, dim_sizes))
    data = run(args.dtype, args.sweep_schedules, MKNs)

    make_output(data, MKNs, f"square_bench-{args.dtype}")
```
**EN:** `run_square_bench` implements a helper used by `benchmark_machete.py`. It mainly works with `args` and relies on `list`, `range`, `zip`, `run`, `make_output` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `run_square_bench` 负责实现 `benchmark_machete.py` 使用的辅助逻辑。 它主要处理 `args`，并结合 `list`, `range`, `zip`, `run`, `make_output` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Function `run_range_bench` (lines 545-558)
```python
def run_range_bench(args):
    m_start, k_start, n_start = (int(x) for x in args.dim_start.split(","))
    m_end, k_end, n_end = (int(x) for x in args.dim_end.split(","))
    m_increment, k_increment, n_increment = (
        int(x) for x in args.dim_increment.split(",")
    )
    Ms = list(range(m_start, m_end + 1, m_increment))
    Ks = list(range(k_start, k_end + 1, k_increment))
    Ns = list(range(n_start, n_end + 1, n_increment))
    MKNs = list(product(Ms, Ks, Ns))

    data = run(args.dtype, args.sweep_schedules, MKNs)

    make_output(data, MKNs, f"range_bench-{args.dtype}")
```
**EN:** `run_range_bench` implements a helper used by `benchmark_machete.py`. It mainly works with `args` and relies on `int`, `args.dim_start.split`, `args.dim_end.split`, `args.dim_increment.split`, `list`, `range` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `run_range_bench` 负责实现 `benchmark_machete.py` 使用的辅助逻辑。 它主要处理 `args`，并结合 `int`, `args.dim_start.split`, `args.dim_end.split`, `args.dim_increment.split`, `list`, `range` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Function `run_model_bench` (lines 561-610)
```python
def run_model_bench(args):
    print("Benchmarking models:")
    for i, model in enumerate(args.models):
        print(f"[{i}]  {model}")

    def model_shapes(model_name: str, tp_size: int) -> list[tuple[int, int]]:
        KNs = []
        for KN, tp_split_dim in copy.deepcopy(WEIGHT_SHAPES[model_name]):
            KN[tp_split_dim] = KN[tp_split_dim] // tp_size
            KNs.append(KN)
        return KNs

    model_bench_data = []
    models_tps = list(itertools.product(args.models, args.tp_sizes))
    for model, tp_size in models_tps:
        Ms = args.batch_sizes
        KNs = model_shapes(model, tp_size)
        MKNs = []
        for m in Ms:
            for k, n in KNs:
                MKNs.append((m, k, n))

        data = run(args, MKNs)
        model_bench_data.append(data)

    type_string = f"{args.act_type}"

    # Print all results
    for data, model_tp in zip(model_bench_data, models_tps):
        model, tp_size = model_tp
        print(f"== Results {type_string} {model}-TP{tp_size} ====")
        print_timers(data)

    timestr = time.strftime("%Y%m%d-%H%M%S")
    # ... omitted for brevity ...
        args_dict.pop("func")
        pkl.dump(
            {
                "args": args_dict,
                "results": all_results,
            },
            f,
        )
```
**EN:** `run_model_bench` implements a helper used by `benchmark_machete.py`. It mainly works with `args` and relies on `print`, `enumerate`, `copy.deepcopy`, `KNs.append`, `list`, `itertools.product` plus iteration and context management to move data through this part of the benchmark pipeline.
**CN:** `run_model_bench` 负责实现 `benchmark_machete.py` 使用的辅助逻辑。 它主要处理 `args`，并结合 `print`, `enumerate`, `copy.deepcopy`, `KNs.append`, `list`, `itertools.product` 以及 循环迭代、上下文管理 来完成这一段基准测试流程。

### Entry point (lines 613-745)
```python
if __name__ == "__main__":

    def to_torch_dtype(dt):
        return {
            "bfloat16": torch.bfloat16,
            "float16": torch.float16,
            "int8": torch.int8,
            "float8_e4m3fn": torch.float8_e4m3fn,
            "int": torch.int,
            "float": torch.float,
        }[dt]

    class ToTorchDtype(argparse.Action):
        def __call__(self, parser, namespace, values, option_string=None):
            setattr(namespace, self.dest, to_torch_dtype(values))

    parser = FlexibleArgumentParser(
        description="""
Benchmark Machete GEMM.

    To run square GEMMs:
        python3 ./benchmarks/kernels/benchmark_machete.py --dtype float16 square_bench --dim-start 128 --dim-end 512 --dim-increment 64
    
    To run constant N and K and sweep M:
        python3 ./benchmarks/kernels/benchmark_machete.py --dtype float16 range_bench --dim-start 128 --dim-end 512 --dim-increment 64 --n-constant 16384 --k-constant 16384
    
    To run dimensions from a model:
        python3 ./benchmarks/kernels/benchmark_machete.py --dtype float16 model_bench --models meta-llama/Llama-2-7b-hf --batch-sizes 16 --tp-sizes 1
    
    Output:
        - a .pkl file, that is a list of raw torch.benchmark.utils.Measurements for the pytorch and cutlass implementations for the various GEMMs.
            """,  # noqa: E501
        formatter_class=argparse.RawTextHelpFormatter,
    )
# ... omitted for brevity ...

    args = parser.parse_args()

    _SWEEP_SCHEDULES_RESULTS_CSV = args.sweep_csv_out
    args.func(args)

    if _SWEEP_SCHEDULES_RESULTS is not None:
        _SWEEP_SCHEDULES_RESULTS.to_csv(_SWEEP_SCHEDULES_RESULTS_CSV)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `setattr`, `to_torch_dtype`, `FlexibleArgumentParser`, `parser.add_argument`, `parser.add_subparsers`, `subparsers.add_parser` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `setattr`, `to_torch_dtype`, `FlexibleArgumentParser`, `parser.add_argument`, `parser.add_subparsers`, `subparsers.add_parser` 等调用串起来，使模块能够从命令行直接启动。

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
- **EN:** Standard library: `argparse`, `copy`, `itertools`, `math`, `os`, `pickle`.
- **CN:** 标准库依赖：`argparse`, `copy`, `itertools`, `math`, `os`, `pickle`。
- **EN:** Third-party packages: `pandas`, `torch`, `torch.utils.benchmark`.
- **CN:** 第三方依赖：`pandas`, `torch`, `torch.utils.benchmark`。
- **EN:** Internal modules: `weight_shapes`, `vllm`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils_test`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.scalar_type`.
- **CN:** 内部模块：`weight_shapes`, `vllm`, `vllm.model_executor.layers.quantization.utils.marlin_utils`, `vllm.model_executor.layers.quantization.utils.marlin_utils_test`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.scalar_type`。
