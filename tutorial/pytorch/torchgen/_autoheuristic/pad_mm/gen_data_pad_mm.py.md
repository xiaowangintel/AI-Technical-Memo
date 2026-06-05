# gen_data_pad_mm.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torchgen/_autoheuristic/pad_mm/gen_data_pad_mm.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Collects data, trains heuristics, and evaluates learned rules that influence matmul-related generation or tuning decisions.
- **Purpose (CN)**: 收集数据、训练启发式模型，并评估影响矩阵乘相关生成或调优决策的规则。
## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
import csv
import random
import sys
from collections.abc import Generator
from pathlib import Path
from typing import Any


sys.path.append(str(Path(__file__).absolute().parents[1]))

from benchmark_runner import BenchmarkRunner  # type: ignore[import-not-found]
from benchmark_utils import (  # type: ignore[import-not-found]
    fits_in_memory,
    get_mm_tensors,
    get_random_between_pow2,
    set_precision,
    transpose_tensors,
)
```
- **EN**: The import section wires together third-party modules such as benchmark_runner, benchmark_utils; standard-library modules such as csv, random, sys, collections.abc, and 2 more for the logic below.
- **CN**: 导入区把第三方模块，如 benchmark_runner、benchmark_utils；标准库模块，如 csv、random、sys、collections.abc 等共 6 项组织在一起，供下方逻辑使用。

### Lines 19-33
```python
from collect_known_mm_shapes import (
    collect_known_mm_shapes,  # type: ignore[import-not-found]
)

import torch
from torch._inductor.fx_passes.pad_mm import (  # type: ignore[import-not-found]
    get_alignment_size_dtype,
)
from torch._inductor.utils import fresh_cache


class BenchmarkRunnerPadMM(BenchmarkRunner):  # type: ignore[misc, no-any-unimported]
    """
    BenchmarkRunner for pad_mm. Used to generate collect training data with AutoHeuristic to learn a heuristic.
    """
```
- **EN**: The import section wires together PyTorch-local modules such as torch, torch._inductor.fx_passes.pad_mm, torch._inductor.utils; third-party modules such as collect_known_mm_shapes for the logic below. It introduces or extends BenchmarkRunnerPadMM, which hold the primary data model or public surface for this slice of the file. This chunk continues `BenchmarkRunnerPadMM` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries.
- **CN**: 导入区把PyTorch 本地模块，如 torch、torch._inductor.fx_passes.pad_mm、torch._inductor.utils；第三方模块，如 collect_known_mm_shapes组织在一起，供下方逻辑使用。 它引入或扩展了 BenchmarkRunnerPadMM，这些类型承载了本段的核心数据模型或对外接口。 这一段延续了 `BenchmarkRunnerPadMM`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。

### Lines 35-49
```python
    def __init__(self) -> None:
        super().__init__("pad_mm")

        # Add CLI argument for additional shape CSV files
        self.parser.add_argument(
            "--additional-shape-csv",
            nargs="*",
            default=[],
            help="List of CSV files containing additional matrix multiplication shapes (M,K,N,dtype format)",
        )

        # Initialize additional_shape_collections
        self.additional_shape_collections: list[
            list[tuple[int, int, int, torch.dtype, torch.dtype]]
        ] = []
```
- **EN**: This chunk defines `__init__`, which implements one step in the operator code-generation pipeline. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `__init__`，其作用是实现算子代码生成流水线中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 51-63
```python
        # Initialize the shape generator (will be set up after parsing args)
        self.shape_generator = None

    def load_shapes_from_csv(
        self, csv_file: str
    ) -> list[tuple[int, int, int, torch.dtype, torch.dtype]]:
        """Load matrix multiplication shapes from a CSV file in M,K,N,dtype format."""
        shapes = []
        dtype_map = {
            "float16": torch.float16,
            "bfloat16": torch.bfloat16,
            "float32": torch.float32,
        }
```
- **EN**: This chunk defines `load_shapes_from_csv`, which loads external data or stored state into the active pipeline. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `load_shapes_from_csv`，其作用是把外部数据或已存储状态加载到当前流水线中。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 65-81
```python
        try:
            with open(csv_file) as f:
                reader = csv.DictReader(f)
                for row in reader:
                    m = int(row["M"])
                    k = int(row["K"])
                    n = int(row["N"])
                    dtype_str = row["dtype"]

                    if dtype_str in dtype_map:
                        dtype = dtype_map[dtype_str]
                        # Store as (m, k, n, dtype1, dtype2) with same dtype for both
                        shapes.append((m, k, n, dtype, dtype))
                    else:
                        print(
                            f"Warning: Unknown dtype '{dtype_str}' in {csv_file}, skipping row"
                        )
```
- **EN**: This chunk continues `load_shapes_from_csv` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Exception-handling structure makes cleanup or fallback behavior explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `load_shapes_from_csv`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 异常处理结构使清理逻辑或回退行为保持明确。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 83-97
```python
            print(f"Loaded {len(shapes)} shapes from {csv_file}")
        except Exception as e:
            print(f"Error loading shapes from {csv_file}: {e}")

        return shapes

    def setup_shape_collections(self, csv_files: list[str]) -> None:
        """Setup additional shape collections from CSV files and built-in collection."""
        self.additional_shape_collections = []

        # Load shapes from provided CSV files first
        for csv_file in csv_files:
            shapes = self.load_shapes_from_csv(csv_file)
            if shapes:
                self.additional_shape_collections.append(shapes)
```
- **EN**: This chunk defines `setup_shape_collections`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Exception-handling structure makes cleanup or fallback behavior explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `setup_shape_collections`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 异常处理结构使清理逻辑或回退行为保持明确。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 99-116
```python
        self.additional_shape_collections.append(collect_known_mm_shapes())
        self.shape_generator = self.generate_mm_shapes()

    def generate_mm_shapes(self) -> Generator[tuple[int, int, int, Any], None, None]:
        """Generator that yields (m, k, n, dtype) tuples for matrix multiplication.

        First exhausts all shapes from additional_shape_collections, then generates random shapes.
        Only yields unaligned shapes since external CSV shapes may not be pre-filtered.
        """
        # Phase 1: Use all shapes from additional shape collections
        for collection in self.additional_shape_collections:
            for m, k, n, dtype1, _ in collection:
                # Filter for unaligned shapes only (external CSVs may not be pre-filtered)
                align_size = get_alignment_size_dtype(dtype1)
                if not all(self.is_aligned(dim, align_size) for dim in [m, k, n]):
                    # Check if it fits in memory
                    if fits_in_memory(dtype1, m, k, n):
                        yield (m, k, n, dtype1)
```
- **EN**: This chunk defines `generate_mm_shapes`, which generates derived code, tables, or analysis artifacts from higher-level metadata. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `generate_mm_shapes`，其作用是根据高层元数据生成派生代码、表或分析产物。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 118-135
```python
        # Phase 2: Generate infinite random shapes

        while True:
            # Generate random dtype
            dtype_choices = [torch.float16, torch.bfloat16, torch.float32]
            dtype = random.choices(dtype_choices)[0]

            # Generate random shape for this dtype
            uniform = random.choices([True, False])[0]
            align_size = get_alignment_size_dtype(dtype)

            # Keep trying until we get a valid unaligned shape that fits in memory
            while True:
                if uniform:
                    m = random.randint(1, 65536)
                    k = random.randint(1, 65536)
                    n = random.randint(1, 65536)
                else:
```
- **EN**: This chunk continues `generate_mm_shapes` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `generate_mm_shapes`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 136-152
```python
                    m = self.get_random_dim()
                    k = self.get_random_dim()
                    n = self.get_random_dim()

                # Skip if all dimensions are aligned (we need unaligned for padding to be relevant)
                if all(self.is_aligned(dim, align_size) for dim in [m, k, n]):
                    continue

                # Check if it fits in memory
                if fits_in_memory(dtype, m, k, n):
                    yield (m, k, n, dtype)
                    break

    def create_input(self) -> tuple[Any, ...]:
        # Get the next shape from the generator
        m, k, n, dtype = next(self.shape_generator)
        set_precision(dtype)
```
- **EN**: This chunk defines `create_input`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `create_input`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 154-166
```python
        (transpose_left, transpose_right) = transpose_tensors()
        prepadded_left = self.prepadded()
        prepadded_right = self.prepadded()
        return (
            m,
            k,
            n,
            transpose_left,
            transpose_right,
            dtype,
            prepadded_left,
            prepadded_right,
        )
```
- **EN**: This chunk continues `create_input` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `create_input`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 168-185
```python
    def run_benchmark(
        self,
        m: int,
        k: int,
        n: int,
        transpose_left: bool,
        transpose_right: bool,
        dtype: Any,
        prepadded_left: bool,
        prepadded_right: bool,
    ) -> None:
        a, b = get_mm_tensors(
            m,
            k,
            n,
            transpose_left,
            transpose_right,
            dtype_left=dtype,
```
- **EN**: This chunk defines `run_benchmark`, which measures behavior so implementations or heuristics can be compared. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `run_benchmark`，其作用是测量行为，以便比较不同实现或启发式规则。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 186-200
```python
            dtype_right=dtype,
        )

        print("Benchmarking the following input:")
        print(f"m={m} k={k} n={n} dtype={dtype}")
        print(f"transpose_left={transpose_left} transpose_right={transpose_right}")
        print(f"prepadded_left={prepadded_left} prepadded_right={prepadded_right}")

        with fresh_cache():

            def mm(a: Any, b: Any) -> Any:
                return torch.mm(a, b)

            def mm_mat1_prepadded(a: Any, b: Any) -> Any:
                return torch.mm(a + 1, b)
```
- **EN**: This chunk defines `mm_mat1_prepadded`, which implements one step in the operator code-generation pipeline. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `mm_mat1_prepadded`，其作用是实现算子代码生成流水线中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 202-217
```python
            def mm_mat2_prepadded(a: Any, b: Any) -> Any:
                return torch.mm(a, b + 1)

            def mm_mat1_mat2_prepadded(a: Any, b: Any) -> Any:
                return torch.mm(a + 1, b + 1)

            if prepadded_left and prepadded_right:
                cf = torch.compile(mm_mat1_mat2_prepadded)
            elif prepadded_left:
                cf = torch.compile(mm_mat1_prepadded)
            elif prepadded_right:
                cf = torch.compile(mm_mat2_prepadded)
            else:
                cf = torch.compile(mm)
            cf(a, b)
            torch.compiler.reset()
```
- **EN**: This chunk defines `mm_mat1_mat2_prepadded`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `mm_mat1_mat2_prepadded`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 219-234
```python
    def get_random_dim(
        self, min_power2: int = 1, max_power2: int = 16, p_unaligned: float = 0.25
    ) -> int:
        aligned = random.choices([True, False], [1 - p_unaligned, p_unaligned])[0]
        if aligned:
            return 2 ** random.randint(min_power2, max_power2)  # type: ignore[no-any-return]
        else:
            # choose a random number between 2^i and 2^(i+1)
            return get_random_between_pow2(min_power2, max_power2)  # type: ignore[no-any-return]

    def is_aligned(self, dim: int, align_size: int) -> bool:
        return dim % align_size == 0

    def prepadded(self, p_prepadded: float = 0.2) -> bool:
        # p_prepadded: probability that a tensor is "prepadded", i.e. pad_mm excludes time it takes to pad from benchmarking
        return random.choices([True, False], [p_prepadded, 1 - p_prepadded])[0]
```
- **EN**: This chunk defines `prepadded`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `prepadded`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 236-248
```python
    def run(self) -> None:
        """Override run to setup shape collections before running."""
        import time

        from tqdm import tqdm

        torch.set_default_device("cuda")
        args = self.parser.parse_args()

        # Setup shape collections based on CLI arguments
        self.setup_shape_collections(args.additional_shape_csv)

        # Set up torch configuration (copied from parent run method)
```
- **EN**: The import section wires together third-party modules such as tqdm; standard-library modules such as time for the logic below. This chunk defines `run`, which implements one step in the operator code-generation pipeline. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 导入区把第三方模块，如 tqdm；标准库模块，如 time组织在一起，供下方逻辑使用。 这一段定义了 `run`，其作用是实现算子代码生成流水线中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 250-265
```python
        if args.use_heuristic:
            torch._inductor.config.autoheuristic_use.pad_mm = True
            torch._inductor.config.autoheuristic_collect.pad_mm = False
        else:
            torch._inductor.config.autoheuristic_use.pad_mm = False
            torch._inductor.config.autoheuristic_collect.pad_mm = True
        torch._inductor.config.autoheuristic_log_path = args.o
        if args.device is not None:
            torch.cuda.set_device(args.device)
        random.seed(time.time())

        # Run the main benchmarking loop
        for _ in tqdm(range(args.num_samples)):
            input = self.create_input()
            for _ in range(args.num_reps):
                self.run_benchmark(*input)
```
- **EN**: This chunk continues `run` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `run`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 268-270
```python
if __name__ == "__main__":
    runner = BenchmarkRunnerPadMM()
    runner.run()
```
- **EN**: This chunk continues `run` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `run`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

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
- **Compilation pipeline**
  - EN: Connects transforms to ahead-of-time tracing, graph capture, or compilation flows.
  - CN: 把各类变换接入提前追踪、图捕获或编译流程。
- **BenchmarkRunnerPadMM**
  - EN: `BenchmarkRunnerPadMM` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `BenchmarkRunnerPadMM` 是本文件声明、导出或驱动的显著符号之一。
- **__init__**
  - EN: `__init__` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `__init__` 是本文件声明、导出或驱动的显著符号之一。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torch`, `torch._inductor.fx_passes.pad_mm`, `torch._inductor.utils`
- **Third-party modules / 第三方模块**: `benchmark_runner`, `benchmark_utils`, `collect_known_mm_shapes`, `tqdm`
- **Standard library / 标准库**: `csv`, `random`, `sys`, `collections.abc`, `pathlib`, `typing`, `time`
- **Primary symbols / 核心符号**: `BenchmarkRunnerPadMM`, `__init__`, `load_shapes_from_csv`, `setup_shape_collections`, `generate_mm_shapes`, `create_input`, `run_benchmark`, `mm`, `mm_mat1_prepadded`, `mm_mat2_prepadded`
- **Note / 说明**: 14 imports were detected; only the first few are listed for readability. / 检测到 14 个导入项，为便于阅读这里只展示前若干项。
