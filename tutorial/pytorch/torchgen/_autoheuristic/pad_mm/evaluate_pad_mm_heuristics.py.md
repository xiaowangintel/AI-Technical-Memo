# evaluate_pad_mm_heuristics.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torchgen/_autoheuristic/pad_mm/evaluate_pad_mm_heuristics.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Collects data, trains heuristics, and evaluates learned rules that influence matmul-related generation or tuning decisions.
- **Purpose (CN)**: 收集数据、训练启发式模型，并评估影响矩阵乘相关生成或调优决策的规则。
## Line-by-Line Analysis / 逐行分析

### Lines 1-21
```python
#!/usr/bin/env python3

import argparse
import csv
import functools

import torch
from torch._inductor.autoheuristic.autoheuristic_utils import AHContext, AHMetadata
from torch._inductor.fx_passes.pad_mm import get_alignment_size_dtype
from torch._inductor.runtime.benchmarking import benchmarker
from torch._inductor.utils import get_gpu_shared_memory


def fits_in_memory(dtype, m: int, k: int, n: int) -> bool:
    threshold_memory = torch.cuda.get_device_properties(0).total_memory / 4
    return dtype.itemsize * (m * k + k * n + m * n) < threshold_memory


def set_precision(dtype, float32_precision: str = "highest") -> None:
    precision = float32_precision if dtype == torch.float32 else "high"
    torch.set_float32_matmul_precision(precision)
```
- **EN**: The import section wires together PyTorch-local modules such as torch, torch._inductor.autoheuristic.autoheuristic_utils, torch._inductor.fx_passes.pad_mm, torch._inductor.runtime.benchmarking, and 1 more; standard-library modules such as argparse, csv, functools for the logic below. This chunk defines `set_precision`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 导入区把PyTorch 本地模块，如 torch、torch._inductor.autoheuristic.autoheuristic_utils、torch._inductor.fx_passes.pad_mm、torch._inductor.runtime.benchmarking 等共 5 项；标准库模块，如 argparse、csv、functools组织在一起，供下方逻辑使用。 这一段定义了 `set_precision`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 24-44
```python
def get_heuristic_decision(m: int, k: int, n: int, dtype: torch.dtype) -> str | None:
    from torch._inductor.autoheuristic.autoheuristic import AutoHeuristic, LocalFeedback
    from torch._inductor.fx_passes.pad_mm import (
        get_alignment_size,
        get_context,
        get_padded_length,
        pad_mm_operations,
        pad_mm_precondition,
    )

    torch._inductor.config.autoheuristic_use.pad_mm = True

    if not torch._inductor.config.run_autoheuristic("pad_mm"):
        return None

    a = torch.randn(m, k, dtype=dtype, device="cuda")
    b = torch.randn(k, n, dtype=dtype, device="cuda")

    m_padded_length = get_padded_length(m, get_alignment_size(a))
    k_padded_length = get_padded_length(k, get_alignment_size(a))
    n_padded_length = get_padded_length(n, get_alignment_size(b))
```
- **EN**: The import section wires together PyTorch-local modules such as torch._inductor.autoheuristic.autoheuristic, torch._inductor.fx_passes.pad_mm for the logic below. This chunk defines `get_heuristic_decision`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 导入区把PyTorch 本地模块，如 torch._inductor.autoheuristic.autoheuristic、torch._inductor.fx_passes.pad_mm组织在一起，供下方逻辑使用。 这一段定义了 `get_heuristic_decision`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 46-70
```python
    context = get_context(
        a,
        b,
        mat1_pre_padded=False,
        mat2_pre_padded=False,
        m_padded_length=m_padded_length,
        k_padded_length=k_padded_length,
        n_padded_length=n_padded_length,
    )

    def dummy_feedback(choice: str) -> float:
        return 1.0

    def fallback() -> str:
        return "no_decision"

    autoheuristic = AutoHeuristic(
        fallback=fallback,
        choices=["orig", "pad"],
        feedback=LocalFeedback(dummy_feedback),
        context=context,
        name="pad_mm",
        augment_context=pad_mm_operations(),
        precondition=pad_mm_precondition,
    )
```
- **EN**: This chunk defines `fallback`, which implements one step in the operator code-generation pipeline. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `fallback`，其作用是实现算子代码生成流水线中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 72-93
```python
    choice = autoheuristic.get_choice()
    return choice


def benchmark_both_choices(
    m: int,
    k: int,
    n: int,
    dtype: torch.dtype,
    num_reps: int = 3,
    float32_precision: str = "highest",
) -> tuple[float, float]:
    set_precision(dtype, float32_precision)
    a = torch.randn(m, k, dtype=dtype, device="cuda")
    b = torch.randn(k, n, dtype=dtype, device="cuda")

    # Use existing benchmarking infrastructure with proper cache management
    # benchmarker returns time in milliseconds, so convert to seconds for consistency
    orig_time_ms = benchmarker.benchmark(
        torch.mm, fn_args=(a, b), rep=num_reps, is_vetted_benchmarking=True
    )
    orig_time = orig_time_ms / 1000.0  # Convert ms to seconds
```
- **EN**: This chunk defines `benchmark_both_choices`, which measures behavior so implementations or heuristics can be compared. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `benchmark_both_choices`，其作用是测量行为，以便比较不同实现或启发式规则。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 95-116
```python
    from torch._inductor.fx_passes.pad_mm import (
        get_alignment_size,
        get_padded_length,
        pad_mm,
    )

    m_padded_length = get_padded_length(a.shape[0], get_alignment_size(a))
    k_padded_length = get_padded_length(a.shape[1], get_alignment_size(a))
    n_padded_length = get_padded_length(b.shape[1], get_alignment_size(b))

    if m_padded_length == 0 and k_padded_length == 0 and n_padded_length == 0:
        return orig_time, orig_time

    pad_time_ms = benchmarker.benchmark(
        pad_mm,
        fn_args=(a, b, m_padded_length, k_padded_length, n_padded_length),
        rep=num_reps,
        is_vetted_benchmarking=True,
    )
    pad_time = pad_time_ms / 1000.0  # Convert ms to seconds

    return orig_time, pad_time
```
- **EN**: The import section wires together PyTorch-local modules such as torch._inductor.fx_passes.pad_mm for the logic below. This chunk continues `benchmark_both_choices` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 导入区把PyTorch 本地模块，如 torch._inductor.fx_passes.pad_mm组织在一起，供下方逻辑使用。 这一段延续了 `benchmark_both_choices`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 119-144
```python
def load_shapes_from_csv(csv_file: str) -> list:
    shapes = []
    with open(csv_file) as f:
        reader = csv.DictReader(f)
        for row in reader:
            m, k, n = int(row["M"]), int(row["K"]), int(row["N"])
            dtype_str = row["dtype"]

            if dtype_str == "float16":
                dtype = torch.float16
            elif dtype_str == "bfloat16":
                dtype = torch.bfloat16
            elif dtype_str == "float32":
                dtype = torch.float32
            else:
                continue

            shapes.append((m, k, n, dtype))

    print(f"Loaded {len(shapes)} shapes from {csv_file}")
    return shapes


@functools.cache
def get_shared_mem_size():
    return get_gpu_shared_memory()
```
- **EN**: Decorators such as @functools.cache modify how the following Python definitions are exported, wrapped, or dataclass-managed. This chunk defines `get_shared_mem_size`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @functools.cache 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 这一段定义了 `get_shared_mem_size`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 147-173
```python
def check_shape_passes_precondition(m: int, k: int, n: int, dtype: torch.dtype) -> bool:
    """
    Check if a shape passes the same precondition used by the actual pad_mm AutoHeuristics.

    This uses the exact same pad_mm_precondition function that the AutoHeuristic system
    uses, avoiding hardcoded magic numbers by delegating to the source of truth.
    """
    from torch._inductor.autoheuristic.autoheuristic_utils import pad_mm_precondition

    shared_memory = get_shared_mem_size()
    device_capa = torch.cuda.get_device_capability()

    # Create the same metadata and context that AutoHeuristics uses
    metadata = AHMetadata(
        shared_memory=shared_memory,
        device_capa=device_capa,
        choices=["orig", "pad"],  # Required but not used for precondition check
        name="pad_mm",  # Required but not used for precondition check
    )

    context = AHContext()
    context.add_feature("m", m)
    context.add_feature("k", k)
    context.add_feature("n", n)

    # Use the actual pad_mm_precondition function - no hardcoded values!
    return pad_mm_precondition(metadata, context)
```
- **EN**: The import section wires together PyTorch-local modules such as torch._inductor.autoheuristic.autoheuristic_utils for the logic below. This chunk defines `check_shape_passes_precondition`, which verifies invariants and catches incorrect states early. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 导入区把PyTorch 本地模块，如 torch._inductor.autoheuristic.autoheuristic_utils组织在一起，供下方逻辑使用。 这一段定义了 `check_shape_passes_precondition`，其作用是验证不变量，并尽早捕获错误状态。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 176-202
```python
def filter_shapes(shapes: list) -> list:
    filtered = []
    aligned_count = 0
    precondition_failed_count = 0
    memory_count = 0

    for m, k, n, dtype in shapes:
        # Check if already aligned
        align_size = get_alignment_size_dtype(dtype)
        is_aligned = all((dim % align_size == 0) for dim in [m, k, n])

        if is_aligned:
            aligned_count += 1
            continue

        # Check if passes the actual precondition used by pad_mm AutoHeuristics
        if not check_shape_passes_precondition(m, k, n, dtype):
            precondition_failed_count += 1
            continue

        # Check if fits in memory
        if not fits_in_memory(dtype, m, k, n):
            memory_count += 1
            continue

        # This shape is suitable for evaluation
        filtered.append((m, k, n, dtype))
```
- **EN**: This chunk defines `filter_shapes`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `filter_shapes`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 204-231
```python
    print("Filtering results:")
    print(f"  Already aligned (skipped): {aligned_count}")
    print(f"  Failed pad_mm_precondition (skipped): {precondition_failed_count}")
    print(f"  Too large for memory (skipped): {memory_count}")
    print(f"  Suitable for evaluation: {len(filtered)}")

    return filtered


def main():
    parser = argparse.ArgumentParser(
        description="Evaluate trained AutoHeuristics for pad_mm optimization"
    )
    parser.add_argument("csv_file", help="Path to CSV file with M,K,N,dtype columns")
    parser.add_argument(
        "--num-reps", type=int, default=3, help="Benchmark repetitions (default: 3)"
    )
    parser.add_argument(
        "--device", type=int, default=None, help="CUDA device (default: current)"
    )
    parser.add_argument(
        "--max-shapes",
        type=int,
        default=10000,
        help="Max shapes to test (default: 10000)",
    )
    parser.add_argument(
        "--float32_matmul_precision",
```
- **EN**: This chunk defines `main`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `main`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 232-255
```python
        type=str,
        choices=["high", "highest"],
        default="highest",
        help="Matmul precision for float32 (default: highest). Non-fp32 always uses 'high'.",
    )

    args = parser.parse_args()

    torch.set_default_device("cuda")
    if args.device is not None:
        torch.cuda.set_device(args.device)

    print(f"Using CUDA device: {torch.cuda.current_device()}")
    print()

    shapes = load_shapes_from_csv(args.csv_file)
    if not shapes:
        print("No shapes found!")
        return

    shapes = filter_shapes(shapes)
    if not shapes:
        print("No suitable shapes found!")
        return
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 257-283
```python
    if len(shapes) > args.max_shapes:
        shapes = shapes[: args.max_shapes]
        print(f"Limited to first {args.max_shapes} shapes")

    print(f"Evaluating {len(shapes)} shapes with {args.num_reps} reps each")
    print()

    total_decisions = 0
    correct_decisions = 0
    true_positives = 0  # Chose pad, should pad
    true_negatives = 0  # Chose orig, should orig
    false_positives = 0  # Chose pad, should orig
    false_negatives = 0  # Chose orig, should pad
    no_decision_shapes = 0

    tp_speedups = []  # Speed-up percentages for true positives
    fp_slowdowns = []  # Speed-down percentages for false positives

    # Track non-confident decisions and confident decisions by dtype
    no_decision_shape_list = []  # List of (M, K, N, dtype) where heuristic chose no_decision
    confident_by_dtype = {}  # Count of confident decisions by dtype

    for i, (m, k, n, dtype) in enumerate(shapes, 1):
        print(f"Shape {i}/{len(shapes)}: M={m}, K={k}, N={n}, dtype={dtype}")

        heuristic_choice = get_heuristic_decision(m, k, n, dtype)
        print(f"  Heuristic: {heuristic_choice}")
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 285-312
```python
        orig_time, pad_time = benchmark_both_choices(
            m, k, n, dtype, args.num_reps, args.float32_matmul_precision
        )
        ground_truth = "pad" if pad_time < orig_time else "orig"

        print(f"  Times: orig={orig_time:.6f}s, pad={pad_time:.6f}s")
        print(f"  Ground truth: {ground_truth}")

        if heuristic_choice == "no_decision":
            # Heuristic punted to benchmarking - this is correct behavior for small/uncertain shapes
            no_decision_shapes += 1
            no_decision_shape_list.append((m, k, n, dtype))
            print("  Heuristic chose to benchmark (conservative)")
        else:
            # Heuristic made a confident decision - evaluate accuracy
            total_decisions += 1
            # Track confident decisions by dtype
            dtype_str = str(dtype).replace("torch.", "")
            confident_by_dtype[dtype_str] = confident_by_dtype.get(dtype_str, 0) + 1
            if heuristic_choice == ground_truth:
                correct_decisions += 1
                print("  ✓ CORRECT")
                if heuristic_choice == "pad":
                    true_positives += 1  # Correctly chose pad
                    # Calculate speed-up: (orig_time - pad_time) / orig_time * 100
                    speedup = (orig_time - pad_time) / orig_time * 100
                    tp_speedups.append(speedup)
                    print(f"    Speed-up: {speedup:.1f}%")
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 313-338
```python
                else:
                    true_negatives += 1  # Correctly chose orig
            else:
                print("  ✗ WRONG")
                if heuristic_choice == "pad" and ground_truth == "orig":
                    false_positives += 1
                    # Calculate speed-down: (pad_time - orig_time) / orig_time * 100
                    slowdown = (pad_time - orig_time) / orig_time * 100
                    fp_slowdowns.append(slowdown)
                    print(f"    Speed-down: {slowdown:.1f}%")
                elif heuristic_choice == "orig" and ground_truth == "pad":
                    false_negatives += 1

        print(f"  Confidence Rate: {total_decisions}/{i}")
        if total_decisions > 0:
            accuracy = correct_decisions / total_decisions * 100
            tp_rate = true_positives / total_decisions * 100
            tn_rate = true_negatives / total_decisions * 100
            fp_rate = false_positives / total_decisions * 100
            fn_rate = false_negatives / total_decisions * 100

            # Compute average speedup/slowdown
            avg_tp_speedup = sum(tp_speedups) / len(tp_speedups) if tp_speedups else 0
            avg_fp_slowdown = (
                sum(fp_slowdowns) / len(fp_slowdowns) if fp_slowdowns else 0
            )
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 340-366
```python
            print(
                f"  Accuracy: {correct_decisions}/{total_decisions} ({accuracy:.1f}%) "
                f"| TP: {tp_rate:.1f}% (avg speedup: {avg_tp_speedup:.1f}%) "
                f"| TN: {tn_rate:.1f}% "
                f"| FP: {fp_rate:.1f}% (avg slowdown: {avg_fp_slowdown:.1f}%)"
                f"| FN: {fn_rate:.1f}%"
            )

        print()

    print("=== FINAL RESULTS ===")
    print(f"Confident decisions: {total_decisions}")
    print(f"#Shapes without confident decisions: {no_decision_shapes}")

    if total_decisions > 0:
        accuracy = correct_decisions / total_decisions * 100
        tp_rate = true_positives / total_decisions * 100
        tn_rate = true_negatives / total_decisions * 100
        fp_rate = false_positives / total_decisions * 100
        fn_rate = false_negatives / total_decisions * 100

        avg_tp_speedup = sum(tp_speedups) / len(tp_speedups) if tp_speedups else 0
        avg_fp_slowdown = sum(fp_slowdowns) / len(fp_slowdowns) if fp_slowdowns else 0

        print(
            f"\nConfident decision accuracy: {accuracy:.1f}% ({correct_decisions}/{total_decisions})"
        )
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 368-390
```python
        if tp_speedups:
            print(
                f"True Positives (chose pad, should pad): {tp_rate:.1f}% ({true_positives}) "
                f"| Avg speed-up: {avg_tp_speedup:.1f}%"
            )
        else:
            print(
                f"True Positives (chose pad, should pad): {tp_rate:.1f}% ({true_positives})"
            )

        print(
            f"True Negatives (chose orig, should orig): {tn_rate:.1f}% ({true_negatives})"
        )

        if fp_slowdowns:
            print(
                f"False Positives (chose pad, should orig): {fp_rate:.1f}% ({false_positives}) "
                f"| Avg speed-down: {avg_fp_slowdown:.1f}%"
            )
        else:
            print(
                f"False Positives (chose pad, should orig): {fp_rate:.1f}% ({false_positives})"
            )
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。

### Lines 392-412
```python
        print(
            f"False Negatives (chose orig, should pad): {fn_rate:.1f}% ({false_negatives})"
        )
    else:
        print("No confident decisions made!")

    total_evaluated = total_decisions + no_decision_shapes
    if total_evaluated > 0:
        print(
            f"\nConfidence rate: ({total_decisions}/{total_evaluated} made confident decisions)"
        )

    # Print shapes where AutoHeuristics did not make a confident decision
    print(f"\n=== NON-CONFIDENT DECISIONS ({len(no_decision_shape_list)}) ===")
    if no_decision_shape_list:
        print("Shapes where AutoHeuristics chose 'no_decision' (non-confident):")
        for m, k, n, dtype in no_decision_shape_list:
            dtype_str = str(dtype).replace("torch.", "")
            print(f"  M={m}, K={k}, N={n}, dtype={dtype_str}")
    else:
        print("All shapes had confident decisions!")
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 414-426
```python
    # Print confident decisions by dtype
    print("\n=== CONFIDENT DECISIONS BY DTYPE ===")
    if confident_by_dtype:
        print("Number of confident decisions per dtype:")
        for dtype_str, count in sorted(confident_by_dtype.items()):
            print(f"  {dtype_str}: {count} confident decisions")
        print(f"Total confident decisions: {sum(confident_by_dtype.values())}")
    else:
        print("No confident decisions made!")


if __name__ == "__main__":
    main()
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

## Key Concepts / 关键概念

- **Torch operator code generation**
  - EN: Builds derived code artifacts from declarative operator metadata.
  - CN: 从声明式算子元数据构建派生代码产物。
- **Learned heuristics**
  - EN: Collects data or trains rules that guide performance-sensitive decisions.
  - CN: 收集数据或训练规则，以指导性能敏感的决策。
- **Benchmarking**
  - EN: Measures runtime behavior to compare implementations or generate tuning data.
  - CN: 测量运行时行为，用于比较实现或生成调优数据。
- **fits_in_memory**
  - EN: `fits_in_memory` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `fits_in_memory` 是本文件声明、导出或驱动的显著符号之一。
- **set_precision**
  - EN: `set_precision` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `set_precision` 是本文件声明、导出或驱动的显著符号之一。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torch`, `torch._inductor.autoheuristic.autoheuristic_utils`, `torch._inductor.fx_passes.pad_mm`, `torch._inductor.runtime.benchmarking`, `torch._inductor.utils`, `torch._inductor.autoheuristic.autoheuristic`
- **Standard library / 标准库**: `argparse`, `csv`, `functools`
- **Primary symbols / 核心符号**: `fits_in_memory`, `set_precision`, `get_heuristic_decision`, `dummy_feedback`, `fallback`, `benchmark_both_choices`, `load_shapes_from_csv`, `get_shared_mem_size`, `check_shape_passes_precondition`, `filter_shapes`
