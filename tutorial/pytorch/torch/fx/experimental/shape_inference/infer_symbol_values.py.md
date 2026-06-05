# infer_symbol_values.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/experimental/shape_inference/infer_symbol_values.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Holds experimental tracing, shape reasoning, and transformation infrastructure built on FX.
- **Purpose (CN)**: 存放基于 FX 的实验性跟踪、形状推理与变换基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```python
import re
from collections import defaultdict
from typing import Any

import numpy as np
import sympy as sp

import torch


square_brackets_pattern = r"\[([^]]+)\]"
parentheses_pattern = r"\((.*?)\)"
s_pattern = r"s\d+"
```
- **EN**: This module-level block helps manipulate FX graphs, tracing helpers, or transformation utilities.
- **CN**: 这个模块级代码块用于操作 FX 图、跟踪辅助逻辑或变换工具。

### Lines 16-33
```python
def infer_symbol_values(
    symints: list[torch.SymInt | int],
    init_symints: list[torch.SymInt | int],
    symbol_idx_dict: dict[str, int],
    padding_constraints: defaultdict[torch.SymInt, list[sp.Expr | int]],
    constraint: str,
) -> None:
    if constraint.find("non-singleton") != -1:
        left_expression, right_expression = re.findall(parentheses_pattern, constraint)
        calculate_value(left_expression, right_expression, symints, symbol_idx_dict)

    elif constraint.find("first two dimensions of batch2 tensor to be") != -1:
        matches = re.findall(square_brackets_pattern, constraint)
        left_expression, right_expression = (
            matches[i].split(",")[1].strip() for i in (0, 1)
        )
        calculate_value(left_expression, right_expression, symints, symbol_idx_dict)
```
- **EN**: Defines the `infer_symbol_values` function; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`infer_symbol_values` 函数；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 34-46
```python
    elif constraint.find("a and b must have same reduction dim") != -1:
        matches = re.findall(square_brackets_pattern, constraint)
        left_expression = matches[0].split(",")[1].strip()
        right_expression = matches[1].split(",")[0].strip()
        calculate_value(left_expression, right_expression, symints, symbol_idx_dict)

    elif constraint.find("Split sizes add up to") != -1:
        match_1 = re.search(r"to\s+(.*?)\s+but", constraint)
        extracted_value_1 = match_1.group(1) if match_1 else None
        match_2 = re.search(r"of\s+(.*?)$", constraint)
        extracted_value_2 = match_2.group(1) if match_2 else None
        calculate_value(extracted_value_1, extracted_value_2, symints, symbol_idx_dict)
```
- **EN**: This block continues `infer_symbol_values` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `infer_symbol_values`，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 47-62
```python
    elif constraint.find("is invalid for input of size") != -1:
        matches = re.findall(square_brackets_pattern, constraint)
        left_elements = matches[0].split(",")
        left_equation = sp.sympify(1)
        left_num = 1
        right_equation = sp.sympify(constraint.split("size")[1].strip())

        for left_element in left_elements:
            if sp.sympify(left_element) == sp.sympify("-1"):
                continue
            elif sp.sympify(left_element).is_number:
                left_num *= int(left_element)
            else:
                left_equation *= sp.sympify(left_element)
        right_equation = sp.cancel(right_equation / left_equation)
```
- **EN**: This block continues `infer_symbol_values` and works to manipulate FX graphs, tracing helpers, or transformation utilities. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `infer_symbol_values`，用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 63-74
```python
        right_vars = list(right_equation.free_symbols)
        for right_var in right_vars:
            if sp.sympify(right_var) == sp.sympify("s0"):
                right_equation = sp.cancel(right_equation / right_var)
                right_vars.remove(right_var)  # noqa: B909

        var = right_vars[0]
        idx = symbol_idx_dict[str(var)]
        if var not in padding_constraints:
            padding_constraints[var].append(right_equation)
        update_equation(
            symints,
```
- **EN**: This block continues `infer_symbol_values` and works to track symbolic shape constraints and shape-dependent reasoning. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `infer_symbol_values`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 75-86
```python
            init_symints,
            padding_constraints,
            padding_constraints[var][0],  # type: ignore[arg-type]
            left_num,
            var,
            idx,
        )


def calculate_value(
    left_expression: str | Any | None,
    right_expression: str | Any | None,
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。

### Lines 87-98
```python
    symints: list[torch.SymInt | int],
    symbol_idx_dict: dict[str, int],
) -> None:
    var, val = solve_equation(left_expression, right_expression)
    idx = symbol_idx_dict[var]
    pre_equation = sp.sympify(f"{symints[idx]}")
    symints[idx] = pre_equation.subs(sp.sympify(var), val)


def solve_equation(
    left_expression: str | Any | None,
    right_expression: str | Any | None,
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。

### Lines 99-110
```python
) -> tuple[str, int]:
    expression = f"{left_expression} - {right_expression}"
    var = re.findall(s_pattern, expression)[0]
    if re.findall(parentheses_pattern, expression):
        sub_expression = re.findall(parentheses_pattern, expression)[0]
        var, coeff = sub_expression.split("//")
        x = sp.symbols("x")
        sub_equation = sp.sympify(f"{var} - {coeff} * {x}")
        modified_equation = (
            sp.sympify(x) + sp.sympify(expression) - sp.sympify(sub_expression)
        )
```
- **EN**: This block continues `solve_equation` and works to manipulate FX graphs, tracing helpers, or transformation utilities. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `solve_equation`，用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会根据运行时条件分支处理。

### Lines 111-122
```python
        solution = sp.solve((modified_equation, sub_equation), (x, var))
        return (var, int(solution[sp.sympify(var)]))
    else:
        solution = sp.solve(expression, var)
        val = int(solution[0])
        return (var, val)


def update_equation(
    symints: list[torch.SymInt | int],
    init_symints: list[torch.SymInt | int],
    padding_constraints: defaultdict[torch.SymInt, list[sp.Expr | int]],
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果。

### Lines 123-134
```python
    init_eq: sp.Expr,
    new_mod_num: int,
    var: torch.SymInt,
    idx: int,
) -> None:
    padding_constraints[var].append(new_mod_num)
    mod_num = np.lcm.reduce(padding_constraints[var][1:])  # type: ignore[arg-type]
    eq = mod_num * init_symints[idx]
    eq_const = [arg for arg in init_eq.args if arg.is_number]
    if eq_const:
        rem = int(eq_const[0] % mod_num)
        eq -= rem
```
- **EN**: This block continues `update_equation` and works to track symbolic shape constraints and shape-dependent reasoning. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `update_equation`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会根据运行时条件分支处理。

### Lines 135-135
```python
    symints[idx] = eq
```
- **EN**: This block continues `update_equation` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `update_equation`，用于跟踪符号形状约束以及依赖形状的推理。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`
- **Standard library / 标准库**: `re`, `collections`, `typing`
- **Third-party packages / 第三方包**: `numpy`, `sympy`
- **Primary symbols / 核心符号**: `infer_symbol_values`, `calculate_value`, `solve_equation`, `update_equation`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
