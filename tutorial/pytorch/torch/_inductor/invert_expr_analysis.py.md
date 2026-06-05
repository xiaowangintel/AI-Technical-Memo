# invert_expr_analysis.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/invert_expr_analysis.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `Term`. It exposes functions such as `static_eq`, `generate_inverse_formula`, `parse_terms`, `parse_single_term`, `analyze_expression_properties`, `check_invertibility`, and `...+1`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `Term` 等类。同时提供 `static_eq`、`generate_inverse_formula`、`parse_terms`、`parse_single_term`、`analyze_expression_properties`、`check_invertibility`、`另有1项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
from dataclasses import dataclass

import sympy

from torch._inductor.utils import _IntLike, argsort_sym
from torch.utils._sympy.functions import FloorDiv, ModularIndexing

from .virtualized import V


def static_eq(a: _IntLike, b: _IntLike) -> bool:
    return V.graph.sizevars.statically_known_equals(a, b)


````
- **EN**: Imports dependencies such as `dataclasses`, `sympy`, `torch._inductor.utils`, `torch.utils._sympy.functions`, and `.virtualized` for the logic in this range. Introduces function `static_eq`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `dataclasses`、`sympy`、`torch._inductor.utils`、`torch.utils._sympy.functions`、`.virtualized` 等依赖，为后续逻辑提供基础能力。这里定义了函数`static_eq`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 15-28 / 第 15-28 行
````python
@dataclass
class Term:
    coefficient: _IntLike
    range: _IntLike | None  # None for unbounded
    original_expr: sympy.Expr
    reconstruction_multiplier: _IntLike  # The multiplier needed for reconstruction


def generate_inverse_formula(expr: sympy.Expr, var: sympy.Symbol) -> sympy.Expr | None:
    """
     Analyze an expression to see if it matches a specific invertible pattern that we
     know how to reverse.

     We're looking for expressions that are sums of terms where each term extracts a
````
- **EN**: Introduces class `Term`, function `generate_inverse_formula`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里定义了类`Term`、函数`generate_inverse_formula`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。借助符号表达式来推理索引、形状或代数性质。

### Lines 29-42 / 第 29-42 行
````python
     distinct bounded range from the input variable, like:

         y = c₀*a₀ + c₁*a₁ + c₂*a₂ + ... + cₙ*aₙ

     where each aᵢ must be one of these specific patterns:
     - ModularIndexing(var, divisor, modulo)
     - FloorDiv(ModularIndexing(var, 1, modulo), divisor)
     - FloorDiv(var, divisor)
     - var (the variable itself)

     The key pattern we need is:
     - Coefficients are strictly decreasing: c₀ > c₁ > c₂ > ... > cₙ
     - Each coefficient matches the product of ranges of later terms (mixed-radix property)
     - Each term extracts a bounded range, creating non-overlapping "slots"
````
- **EN**: Initializes or updates values such as `y`. This range continues the implementation of function `generate_inverse_formula`.
- **CN**: 初始化或更新了 `y` 等值。这一段延续了函数`generate_inverse_formula` 的具体实现。

### Lines 43-56 / 第 43-56 行
````python

     If we find this pattern, we can generate the reconstruction transformation that
     decomposes the variable and rebuilds it using the correct multipliers.

     EXAMPLE:
     Input: 100*((p//100)) + 10*((p%100)//10) + (p%10)

     Returns the reconstruction expression:
         remainder₀ = p
         component₀ = remainder₀ // 100          # hundreds digit
         remainder₁ = remainder₀ % 100
         component₁ = remainder₁ // 10           # tens digit
         remainder₂ = remainder₁ % 10
         component₂ = remainder₂                 # ones digit
````
- **EN**: Initializes or updates values such as `EXAMPLE`, and `Input`. This range continues the implementation of function `generate_inverse_formula`.
- **CN**: 初始化或更新了 `EXAMPLE`、`Input` 等值。这一段延续了函数`generate_inverse_formula` 的具体实现。

### Lines 57-70 / 第 57-70 行
````python
         result = component₀*100 + component₁*10 + component₂*1

    This decomposes p into its components and rebuilds it using the original
     multipliers, which should equal the input expression.

     Args:
         expr: Expression to analyze (sum of terms with ModularIndexing, FloorDiv, etc.)
         var: The variable being decomposed

     Returns:
         None if not invertible, or the reconstruction expression

     References:
         Mixed-radix systems: https://en.wikipedia.org/wiki/Mixed_radix
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Initializes or updates values such as `result`, `Args`, `expr`, `var`, `Returns`, and `References`. This range continues the implementation of function `generate_inverse_formula`.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。初始化或更新了 `result`、`Args`、`expr`、`var`、`Returns`、`References` 等值。这一段延续了函数`generate_inverse_formula` 的具体实现。

### Lines 71-84 / 第 71-84 行
````python
    """
    # Step 1: Parse all terms
    terms = parse_terms(expr, var)
    if not terms:
        return None

    # Step 2: Sort by coefficient (descending)
    coeffs = [t.coefficient for t in terms]
    idxs = reversed(argsort_sym(V.graph.sizevars.shape_env, coeffs))
    terms = [terms[i] for i in idxs]

    # Step 3: Check invertibility conditions
    if not check_invertibility(terms):
        return None
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `terms`, `coeffs`, and `idxs`. This range continues the implementation of function `generate_inverse_formula`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `terms`、`coeffs`、`idxs` 等值。这一段延续了函数`generate_inverse_formula` 的具体实现。

### Lines 85-98 / 第 85-98 行
````python

    return generate_reconstruction_expr(terms, var)


def parse_terms(expr: sympy.Expr, var: sympy.Symbol) -> list[Term] | None:
    """Parse expression into terms."""
    if not isinstance(expr, sympy.Add):
        # Single term
        term = parse_single_term(expr, var)
        return [term] if term else []

    terms = []
    for arg in expr.args:
        term = parse_single_term(arg, var)
````
- **EN**: Introduces function `parse_terms`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`parse_terms`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 99-112 / 第 99-112 行
````python
        if term:
            terms.append(term)
        else:
            return None  # If any term fails to parse, fail completely

    return terms


def parse_single_term(term: sympy.Expr, var: sympy.Symbol) -> Term | None:
    """Parse a single term and extract coefficient, range, and reconstruction multiplier."""
    # Extract coefficient and expression parts
    coefficient, expr_parts = term.as_coeff_mul()

    if len(expr_parts) == 0:
````
- **EN**: Introduces function `parse_single_term`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`parse_single_term`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 113-126 / 第 113-126 行
````python
        # Pure constant term
        return Term(
            coefficient=coefficient,
            range=1,
            original_expr=1,
            reconstruction_multiplier=0,
        )
    elif len(expr_parts) == 1:
        expr = expr_parts[0]
    else:
        # Multiple non-constant factors, too complex
        return None

    # Now determine the range and reconstruction multiplier
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `coefficient`, `range`, `original_expr`, `reconstruction_multiplier`, `expr`, and `else`. This range continues the implementation of function `parse_single_term`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `coefficient`、`range`、`original_expr`、`reconstruction_multiplier`、`expr`、`else` 等值。这一段延续了函数`parse_single_term` 的具体实现。

### Lines 127-140 / 第 127-140 行
````python
    range_val, reconstruction_multiplier = analyze_expression_properties(expr, var)
    if reconstruction_multiplier is None:
        return None

    return Term(
        coefficient=coefficient,
        range=range_val,
        original_expr=expr,
        reconstruction_multiplier=reconstruction_multiplier,
    )


def analyze_expression_properties(
    expr: sympy.Expr, var: sympy.Symbol
````
- **EN**: Introduces function `analyze_expression_properties`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`analyze_expression_properties`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-154 / 第 141-154 行
````python
) -> tuple[_IntLike | None, _IntLike | None]:
    """Analyze an expression to determine its range and reconstruction multiplier."""
    # ModularIndexing(var, divisor, modulo) = (var // divisor) % modulo
    if isinstance(expr, ModularIndexing):
        x, div, mod = expr.args
        if static_eq(x, var):
            return mod, div  # Range is mod, multiplier is div

    # FloorDiv cases
    if isinstance(expr, FloorDiv):
        base, divisor = expr.args

        # FloorDiv(ModularIndexing(var, 1, mod), div) = (var % mod) // div
        if isinstance(base, ModularIndexing):
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `analyze_expression_properties`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`analyze_expression_properties` 的具体实现。

### Lines 155-168 / 第 155-168 行
````python
            x, inner_div, mod = base.args
            if static_eq(x, var) and static_eq(inner_div, 1):
                range_val = FloorDiv(mod, divisor)
                return range_val, divisor  # Range is mod//div, multiplier is div

        # FloorDiv(var, divisor) = var // divisor (unbounded)
        elif static_eq(base, var):
            return None, divisor  # Unbounded range, multiplier is div

    return None, None


def check_invertibility(terms: list[Term]) -> bool:
    """Check if the terms represent an invertible transformation."""
````
- **EN**: Introduces function `check_invertibility`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `range_val`.
- **CN**: 这里定义了函数`check_invertibility`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `range_val` 等值。

### Lines 169-182 / 第 169-182 行
````python
    if not terms:
        return False

    # Coefficients must be strictly decreasing
    coeffs = [t.coefficient for t in terms]
    if argsort_sym(V.graph.sizevars.shape_env, coeffs) != list(
        reversed(range(len(coeffs)))
    ):
        return False

    # Check mixed-radix property: each coeff[i] = coeff[i+1] * range[i+1]
    expected_coeff = 1
    for term in reversed(terms):
        if not static_eq(term.coefficient, expected_coeff):
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `coeffs`, and `expected_coeff`. This range continues the implementation of function `check_invertibility`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `coeffs`、`expected_coeff` 等值。这一段延续了函数`check_invertibility` 的具体实现。

### Lines 183-196 / 第 183-196 行
````python
            return False
        if term.range is not None:
            expected_coeff *= term.range

    return True


def generate_reconstruction_expr(terms: list[Term], var: sympy.Symbol) -> sympy.Expr:
    y = var
    reconstruction = sympy.S.Zero
    remainder = y

    for i, term in enumerate(terms):
        if i < len(terms) - 1:
````
- **EN**: Introduces function `generate_reconstruction_expr`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`generate_reconstruction_expr`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 197-205 / 第 197-205 行
````python
            component = FloorDiv(remainder, term.coefficient)
            remainder = ModularIndexing(remainder, 1, term.coefficient)
        else:
            # Last term should also divide by its coefficient
            component = FloorDiv(remainder, term.coefficient)

        reconstruction += component * term.reconstruction_multiplier

    return reconstruction
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `component`, `remainder`, and `else`. This range continues the implementation of function `generate_reconstruction_expr`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `component`、`remainder`、`else` 等值。这一段延续了函数`generate_reconstruction_expr` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Primary classes: `Term`  
  **CN**: 主要类：`Term`
- **EN**: Primary functions: `static_eq`, `generate_inverse_formula`, `parse_terms`, `parse_single_term`, `analyze_expression_properties`, `check_invertibility`, and `...+1`  
  **CN**: 主要函数：`static_eq`、`generate_inverse_formula`、`parse_terms`、`parse_single_term`、`analyze_expression_properties`、`check_invertibility`、`另有1项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch._inductor.utils`, `torch.utils._sympy.functions`, `.virtualized`
