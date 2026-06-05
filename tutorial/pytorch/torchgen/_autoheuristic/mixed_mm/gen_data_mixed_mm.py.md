# gen_data_mixed_mm.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torchgen/_autoheuristic/mixed_mm/gen_data_mixed_mm.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Collects data, trains heuristics, and evaluates learned rules that influence matmul-related generation or tuning decisions.
- **Purpose (CN)**: 收集数据、训练启发式模型，并评估影响矩阵乘相关生成或调优决策的规则。
## Line-by-Line Analysis / 逐行分析

### Lines 1-8
```python
# mypy: ignore-errors
import random
import sys
from pathlib import Path
from typing import Any


sys.path.append(str(Path(__file__).absolute().parents[1]))
```
- **EN**: The import section wires together standard-library modules such as random, sys, pathlib, typing for the logic below.
- **CN**: 导入区把标准库模块，如 random、sys、pathlib、typing组织在一起，供下方逻辑使用。

### Lines 10-18
```python
from benchmark_runner import BenchmarkRunner  # type: ignore[import-not-found]
from benchmark_utils import (  # type: ignore[import-not-found]
    fits_in_memory,
    get_mm_tensors,
    get_random_between_pow2,
)

import torch
from torch._inductor.utils import fresh_cache
```
- **EN**: The import section wires together PyTorch-local modules such as torch, torch._inductor.utils; third-party modules such as benchmark_runner, benchmark_utils for the logic below.
- **CN**: 导入区把PyTorch 本地模块，如 torch、torch._inductor.utils；第三方模块，如 benchmark_runner、benchmark_utils组织在一起，供下方逻辑使用。

### Lines 21-30
```python
class BenchmarkRunnerMixedMM(BenchmarkRunner):  # type: ignore[misc, no-any-unimported]
    """
    BenchmarkRunner for mixed mm. Used to generate collect training data with AutoHeuristic to learn a heuristic.
    Currently, we are generating inputs with the following restrictions:
    - m <= 128, and n and k >= 1024 (for these inputs one of the triton kernels wins in most cases)
    - k % 256 == 0 (if k is not a multiple of the block size, this can have a huge negative impact on performance)
    - mat1 not transposed
    - mat2 transposed
    This allows us to learn a heuristic that works well e.g. for gpt-fast.
    """
```
- **EN**: It introduces or extends BenchmarkRunnerMixedMM, which hold the primary data model or public surface for this slice of the file. This chunk continues `BenchmarkRunnerMixedMM` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 它引入或扩展了 BenchmarkRunnerMixedMM，这些类型承载了本段的核心数据模型或对外接口。 这一段延续了 `BenchmarkRunnerMixedMM`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 32-39
```python
    def __init__(self) -> None:
        super().__init__("mixed_mm")

    def create_input(self) -> tuple[Any, ...]:
        dtype1, dtype2 = self.get_dtypes()
        m, k, n = self.get_m_k_n(dtype1)
        transpose_left, transpose_right = False, True
        return (m, k, n, transpose_left, transpose_right, dtype1, dtype2)
```
- **EN**: This chunk defines `create_input`, which implements one step in the operator code-generation pipeline. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `create_input`，其作用是实现算子代码生成流水线中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 41-52
```python
    def run_benchmark(
        self,
        m: int,
        k: int,
        n: int,
        transpose_left: bool,
        transpose_right: bool,
        dtype_left: Any,
        dtype_right: Any,
    ) -> Any:
        a, b = get_mm_tensors(
            m,
```
- **EN**: This chunk defines `run_benchmark`, which measures behavior so implementations or heuristics can be compared. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `run_benchmark`，其作用是测量行为，以便比较不同实现或启发式规则。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 53-62
```python
            k,
            n,
            transpose_left,
            transpose_right,
            dtype_left=dtype_left,
            dtype_right=torch.float32,
        )
        b = b.to(dtype=dtype_right)

        with fresh_cache():
```
- **EN**: This chunk continues `run_benchmark` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `run_benchmark`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 64-69
```python
            def mixed_mm(A, B):
                return torch.mm(A, B.to(A.dtype))

            cf = torch.compile(mixed_mm, mode="max-autotune-no-cudagraphs")
            cf(a, b)
            torch.compiler.reset()
```
- **EN**: This chunk defines `mixed_mm`, which implements one step in the operator code-generation pipeline. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `mixed_mm`，其作用是实现算子代码生成流水线中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 71-79
```python
    def random_multiple_of_128(self, min_num=7, max_num=17):
        ran_pow2 = random.randint(min_num, max_num - 1)
        start = (2**ran_pow2) // 128
        end = (2 ** (ran_pow2 + 1)) // 128
        random_multiple = random.randint(start, end)
        return random_multiple * 128

    def get_random_pow2(self, min_power2: int, max_power2: int):
        return 2 ** random.randint(min_power2, max_power2)
```
- **EN**: This chunk defines `get_random_pow2`, which implements one step in the operator code-generation pipeline. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `get_random_pow2`，其作用是实现算子代码生成流水线中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 81-89
```python
    def get_distr_type(self) -> str:
        # 85%: choose a random multiple of 128 between 2^10 and 2^17
        # 10%: choose a random power of 2 between 2^10 and 2^17 favoring larger values
        #  4%: choose a random number between 1024 and 131072
        #  1%: choose a random number between 2^i and 2^(i+1) with i in [10, 16]
        return random.choices(
            ["mult_128", "pow2", "uniform", "uniform-between-pow2"],
            [0.85, 0.1, 0.04, 0.01],
        )[0]
```
- **EN**: This chunk defines `get_distr_type`, which implements one step in the operator code-generation pipeline. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `get_distr_type`，其作用是实现算子代码生成流水线中的一个步骤。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 91-102
```python
    def get_random_dim(self):
        distr_type = self.get_distr_type()
        if distr_type == "mult_128":
            return self.random_multiple_of_128(min_num=10, max_num=17)
        if distr_type == "pow2":
            return self.get_random_pow2(min_power2=10, max_power2=17)
        elif distr_type == "uniform-between-pow2":
            return get_random_between_pow2(min_power2=10, max_power2=17)
        elif distr_type == "uniform":
            return random.randint(1024, 131072)
        print(f"random_type {distr_type} not supported")
        sys.exit(1)
```
- **EN**: This chunk defines `get_random_dim`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `get_random_dim`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 104-112
```python
    def get_random_num_small(self) -> int:
        pow2 = random.choices([True, False], [0.75, 0.25])[0]
        if pow2:
            return 2 ** random.randint(1, 7)
        else:
            return get_random_between_pow2(1, 7)

    def get_m_k_n(self, dtype: Any) -> tuple[int, int, int]:
        numel_max = 2**31
```
- **EN**: This chunk defines `get_m_k_n`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `get_m_k_n`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 114-123
```python
        # repeat until tensors fit in memory
        while True:
            m = self.get_random_num_small()
            k = self.get_random_dim()
            n = self.get_random_dim()
            if k % 256 != 0:
                continue

            if not (k >= 1024 and n >= 1024):
                raise AssertionError("k and n must be at least 1024")
```
- **EN**: This chunk continues `get_m_k_n` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `get_m_k_n`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 125-130
```python
            if m * k >= numel_max or m * n >= numel_max or k * n >= numel_max:
                # autotuning will not happen for tensors that are this large
                continue

            if fits_in_memory(dtype, m, k, n):
                return (m, k, n)
```
- **EN**: This chunk continues `get_m_k_n` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `get_m_k_n`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 132-141
```python
    def get_dtypes(self) -> Any:
        while True:
            dtype_floats = [torch.float16, torch.bfloat16]
            dtype_ints = [torch.int8, torch.uint8]
            mat1_dtype = random.choices(dtype_floats)[0]
            mat2_dtype = random.choices(dtype_ints)[0]
            if mat1_dtype == torch.bfloat16 and mat2_dtype == torch.uint8:
                # this combination seems to cause issues with mixed_mm
                continue
            return (mat1_dtype, mat2_dtype)
```
- **EN**: This chunk defines `get_dtypes`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `get_dtypes`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 144-146
```python
if __name__ == "__main__":
    runner = BenchmarkRunnerMixedMM()
    runner.run()
```
- **EN**: This chunk continues `get_dtypes` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `get_dtypes`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

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
- **BenchmarkRunnerMixedMM**
  - EN: `BenchmarkRunnerMixedMM` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `BenchmarkRunnerMixedMM` 是本文件声明、导出或驱动的显著符号之一。
- **__init__**
  - EN: `__init__` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `__init__` 是本文件声明、导出或驱动的显著符号之一。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torch`, `torch._inductor.utils`
- **Third-party modules / 第三方模块**: `benchmark_runner`, `benchmark_utils`
- **Standard library / 标准库**: `random`, `sys`, `pathlib`, `typing`
- **Primary symbols / 核心符号**: `BenchmarkRunnerMixedMM`, `__init__`, `create_input`, `run_benchmark`, `mixed_mm`, `random_multiple_of_128`, `get_random_pow2`, `get_distr_type`, `get_random_dim`, `get_random_num_small`
