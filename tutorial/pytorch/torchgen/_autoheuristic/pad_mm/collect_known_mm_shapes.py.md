# collect_known_mm_shapes.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torchgen/_autoheuristic/pad_mm/collect_known_mm_shapes.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Collects data, trains heuristics, and evaluates learned rules that influence matmul-related generation or tuning decisions.
- **Purpose (CN)**: 收集数据、训练启发式模型，并评估影响矩阵乘相关生成或调优决策的规则。
## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```python
import argparse
import csv
import sys
from pathlib import Path


# Add parent directory to path for imports
sys.path.append(str(Path(__file__).absolute().parents[1]))
sys.path.append(
    str(
        Path(__file__).absolute().parents[3]
        / "benchmarks"
        / "dynamo"
        / "microbenchmarks"
    )
)
```
- **EN**: The import section wires together standard-library modules such as argparse, csv, sys, pathlib for the logic below.
- **CN**: 导入区把标准库模块，如 argparse、csv、sys、pathlib组织在一起，供下方逻辑使用。

### Lines 18-32
```python
from operator_inp_utils import (  # type: ignore[import-not-found]
    deserialize_args,
    OperatorInputsLoader,
)

import torch
from torch._inductor.fx_passes.pad_mm import (
    get_alignment_size_dtype,  # type: ignore[import-not-found]
)
from torch._subclasses.fake_tensor import FakeTensorMode


def is_aligned(dim: int, align_size: int) -> bool:
    """Check if dimension is aligned to the given alignment size."""
    return dim % align_size == 0
```
- **EN**: The import section wires together PyTorch-local modules such as torch, torch._inductor.fx_passes.pad_mm, torch._subclasses.fake_tensor; third-party modules such as operator_inp_utils for the logic below. This chunk defines `is_aligned`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 导入区把PyTorch 本地模块，如 torch、torch._inductor.fx_passes.pad_mm、torch._subclasses.fake_tensor；第三方模块，如 operator_inp_utils组织在一起，供下方逻辑使用。 这一段定义了 `is_aligned`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 35-51
```python
def extract_mm_shapes_from_loader(
    loader: OperatorInputsLoader,
) -> list[tuple[int, int, int, torch.dtype, torch.dtype]]:
    """Extract matrix multiplication shapes from an OperatorInputsLoader using deserialize_args with FakeTensorMode."""
    shapes = []

    # Matrix multiplication operators to look for
    mm_operators = ["aten.mm.default", "aten.addmm.default", "aten.bmm.default"]

    # Use FakeTensorMode to avoid instantiating actual tensors
    with FakeTensorMode():
        for op_name in mm_operators:
            if op_name not in loader.operator_db:
                continue

            # Count shapes extracted from this operator
            shape_count = 0
```
- **EN**: This chunk defines `extract_mm_shapes_from_loader`, which loads external data or stored state into the active pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `extract_mm_shapes_from_loader`，其作用是把外部数据或已存储状态加载到当前流水线中。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 53-70
```python
            # Access the raw string data directly from operator_db and reuse existing parsing
            for input_str in loader.operator_db[op_name]:
                try:
                    # Use deserialize_args to parse inputs - will create fake tensors
                    args, kwargs = deserialize_args(input_str)

                    if op_name == "aten.mm.default":
                        # mm(input, mat2) -> result
                        if len(args) >= 2:
                            a, b = args[0], args[1]
                            if isinstance(a, torch.Tensor) and isinstance(
                                b, torch.Tensor
                            ):
                                a_shape, a_dtype = tuple(a.shape), a.dtype
                                b_shape, b_dtype = tuple(b.shape), b.dtype
                                if len(a_shape) == 2 and len(b_shape) == 2:
                                    m, k = a_shape
                                    k2, n = b_shape
```
- **EN**: This chunk continues `extract_mm_shapes_from_loader` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Exception-handling structure makes cleanup or fallback behavior explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `extract_mm_shapes_from_loader`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 异常处理结构使清理逻辑或回退行为保持明确。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 71-88
```python
                                    if k == k2:  # Valid matrix multiplication
                                        shapes.append((m, k, n, a_dtype, b_dtype))
                                        shape_count += 1

                    elif op_name == "aten.addmm.default":
                        # addmm(bias, input, mat2) -> result
                        if len(args) >= 3:
                            _, a, b = args[0], args[1], args[2]
                            if isinstance(a, torch.Tensor) and isinstance(
                                b, torch.Tensor
                            ):
                                a_shape, a_dtype = tuple(a.shape), a.dtype
                                b_shape, b_dtype = tuple(b.shape), b.dtype
                                if len(a_shape) == 2 and len(b_shape) == 2:
                                    m, k = a_shape
                                    k2, n = b_shape
                                    if k == k2:  # Valid matrix multiplication
                                        shapes.append((m, k, n, a_dtype, b_dtype))
```
- **EN**: This chunk continues `extract_mm_shapes_from_loader` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `extract_mm_shapes_from_loader`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 89-106
```python
                                        shape_count += 1

                    elif op_name == "aten.bmm.default":
                        # bmm(input, mat2) -> result (batch matrix multiplication)
                        if len(args) >= 2:
                            a, b = args[0], args[1]
                            if isinstance(a, torch.Tensor) and isinstance(
                                b, torch.Tensor
                            ):
                                a_shape, a_dtype = tuple(a.shape), a.dtype
                                b_shape, b_dtype = tuple(b.shape), b.dtype
                                if len(a_shape) == 3 and len(b_shape) == 3:
                                    batch1, m, k = a_shape
                                    batch2, k2, n = b_shape
                                    if (
                                        batch1 == batch2 and k == k2
                                    ):  # Valid batch matrix multiplication
                                        shapes.append((m, k, n, a_dtype, b_dtype))
```
- **EN**: This chunk continues `extract_mm_shapes_from_loader` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `extract_mm_shapes_from_loader`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 107-122
```python
                                        shape_count += 1

                except Exception:
                    # Skip invalid inputs
                    continue

            print(f"    Extracted {shape_count} shapes from {op_name}")

    return shapes


def filter_unaligned_shapes(
    shapes: list[tuple[int, int, int, torch.dtype, torch.dtype]],
) -> list[tuple[int, int, int, torch.dtype, torch.dtype]]:
    """Filter shapes to keep only those that are not completely aligned (so padding is relevant)."""
    filtered_shapes = []
```
- **EN**: This chunk defines `filter_unaligned_shapes`, which implements one step in the operator code-generation pipeline. Exception-handling structure makes cleanup or fallback behavior explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `filter_unaligned_shapes`，其作用是实现算子代码生成流水线中的一个步骤。 异常处理结构使清理逻辑或回退行为保持明确。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 124-138
```python
    for m, k, n, dtype1, dtype2 in shapes:
        # Use the primary dtype for alignment calculation (assume both dtypes are similar for alignment purposes)
        dtype = dtype1
        try:
            align_size = get_alignment_size_dtype(dtype)

            # Only keep shapes where not all dimensions are aligned
            if not all(is_aligned(dim, align_size) for dim in [m, k, n]):
                filtered_shapes.append((m, k, n, dtype1, dtype2))

        except Exception:
            # If we can't get alignment size, skip this shape
            continue

    return filtered_shapes
```
- **EN**: This chunk continues `filter_unaligned_shapes` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Exception-handling structure makes cleanup or fallback behavior explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `filter_unaligned_shapes`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 异常处理结构使清理逻辑或回退行为保持明确。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 141-151
```python
def collect_known_mm_shapes() -> list[tuple[int, int, int, torch.dtype, torch.dtype]]:
    """
    Collect known matrix multiplication shapes from HuggingFace, TIMM, and TorchBench datasets.

    Returns:
        List of tuples containing (m, k, n, dtype1, dtype2) for matrix multiplication shapes
        that are not completely aligned (so padding is relevant).
    """
    all_shapes = []

    loaders = []
```
- **EN**: This chunk defines `collect_known_mm_shapes`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `collect_known_mm_shapes`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 153-164
```python
    # Try to load each dataset
    try:
        hf_loader = OperatorInputsLoader.get_huggingface_loader()
        loaders.append(("HuggingFace", hf_loader))
    except Exception as e:
        print(f"Warning: Could not load HuggingFace dataset: {e}")

    try:
        timm_loader = OperatorInputsLoader.get_timm_loader()
        loaders.append(("TIMM", timm_loader))
    except Exception as e:
        print(f"Warning: Could not load TIMM dataset: {e}")
```
- **EN**: This chunk continues `collect_known_mm_shapes` and expands its control flow, data preparation, or emitted structure. Exception-handling structure makes cleanup or fallback behavior explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `collect_known_mm_shapes`，继续展开其控制流、数据准备或生成结构。 异常处理结构使清理逻辑或回退行为保持明确。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 166-182
```python
    try:
        torchbench_loader = OperatorInputsLoader.get_torchbench_loader()
        loaders.append(("TorchBench", torchbench_loader))
    except Exception as e:
        print(f"Warning: Could not load TorchBench dataset: {e}")

    # Extract shapes from each loader
    for dataset_name, loader in loaders:
        print(f"Extracting shapes from {dataset_name}...")

        shapes = extract_mm_shapes_from_loader(loader)
        print(f"Found {len(shapes)} matrix multiplication shapes from {dataset_name}")
        all_shapes.extend(shapes)

    # Remove duplicates
    unique_shapes = list(set(all_shapes))
    print(f"Total unique shapes before filtering: {len(unique_shapes)}")
```
- **EN**: This chunk continues `collect_known_mm_shapes` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Exception-handling structure makes cleanup or fallback behavior explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `collect_known_mm_shapes`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 异常处理结构使清理逻辑或回退行为保持明确。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 184-194
```python
    # Filter for unaligned shapes only
    filtered_shapes = filter_unaligned_shapes(unique_shapes)
    print(f"Shapes after filtering for unaligned: {len(filtered_shapes)}")

    return filtered_shapes


def main(output_file="mm_shapes.csv"):
    shapes = collect_known_mm_shapes()

    print(f"\nCollected {len(shapes)} real-world matrix multiplication shapes")
```
- **EN**: This chunk defines `main`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `main`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 196-209
```python
    # Convert dtype objects to strings and filter for desired dtypes
    dtype_map = {
        torch.float16: "float16",
        torch.bfloat16: "bfloat16",
        torch.float32: "float32",
    }

    # Convert to desired format and filter dtypes
    csv_rows = []
    for m, k, n, dtype1, dtype2 in shapes:
        # Use the first dtype and convert to string
        if dtype1 in dtype_map:
            dtype_str = dtype_map[dtype1]
            csv_rows.append([m, k, n, dtype_str])
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 211-228
```python
    # Save to CSV file
    with open(output_file, "w", newline="") as csvfile:
        writer = csv.writer(csvfile)
        # Write header
        writer.writerow(["M", "K", "N", "dtype"])
        # Write data rows
        writer.writerows(csv_rows)

    print(f"Saved matrix multiplication shapes to {output_file}")


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Collect matrix multiplication shapes from real-world datasets and save to CSV"
    )
    parser.add_argument(
        "--output",
        "-o",
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 229-235
```python
        type=str,
        default="mm_shapes.csv",
        help="Output CSV filename (default: mm_shapes.csv)",
    )

    args = parser.parse_args()
    main(args.output)
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

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
- **Serialization**
  - EN: Moves runtime data to and from persistent archive/container representations.
  - CN: 在运行时数据与持久化归档/容器表示之间进行转换。
- **is_aligned**
  - EN: `is_aligned` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `is_aligned` 是本文件声明、导出或驱动的显著符号之一。
- **extract_mm_shapes_from_loader**
  - EN: `extract_mm_shapes_from_loader` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `extract_mm_shapes_from_loader` 是本文件声明、导出或驱动的显著符号之一。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torch`, `torch._inductor.fx_passes.pad_mm`, `torch._subclasses.fake_tensor`
- **Third-party modules / 第三方模块**: `operator_inp_utils`
- **Standard library / 标准库**: `argparse`, `csv`, `sys`, `pathlib`
- **Primary symbols / 核心符号**: `is_aligned`, `extract_mm_shapes_from_loader`, `filter_unaligned_shapes`, `collect_known_mm_shapes`, `main`
