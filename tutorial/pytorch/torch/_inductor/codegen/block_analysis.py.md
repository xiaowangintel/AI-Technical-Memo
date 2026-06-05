# block_analysis.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/block_analysis.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `BlockPatternMatcher`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `BlockPatternMatcher` 等类。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
import collections
import functools
import textwrap

import sympy
from sympy import Expr, Symbol

from torch.utils._ordered_set import OrderedSet
from torch.utils._sympy.functions import FloorDiv, ModularIndexing

from ..utils import sympy_dot, sympy_subs
from ..virtualized import V


````
- **EN**: Imports dependencies such as `collections`, `functools`, `textwrap`, `sympy`, `torch.utils._ordered_set`, `torch.utils._sympy.functions`, and `...+2` for the logic in this range. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里导入了 `collections`、`functools`、`textwrap`、`sympy`、`torch.utils._ordered_set`、`torch.utils._sympy.functions`、`另有2项` 等依赖，为后续逻辑提供基础能力。借助符号表达式来推理索引、形状或代数性质。

### Lines 15-28 / 第 15-28 行
````python
class BlockPatternMatcher:
    """
    Matches block indexing expressions.
    """

    _indexing_wild_signed_int = functools.partial(
        sympy.Wild, properties=[lambda x: x.is_integer]
    )
    _indexing_wild_unsigned_int = functools.partial(
        sympy.Wild, properties=[lambda x: x.is_integer and x.is_nonnegative]
    )

    @classmethod
    def get_subexpr_involving_symbol(cls, expr: Expr, symbol: Symbol) -> Expr:
````
- **EN**: Introduces class `BlockPatternMatcher`, function `get_subexpr_involving_symbol`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Initializes or updates values such as `_indexing_wild_signed_int`, and `_indexing_wild_unsigned_int`.
- **CN**: 这里定义了类`BlockPatternMatcher`、函数`get_subexpr_involving_symbol`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。初始化或更新了 `_indexing_wild_signed_int`、`_indexing_wild_unsigned_int` 等值。

### Lines 29-42 / 第 29-42 行
````python
        """
        Given a sympy expression, return the subexpression comprised only of terms
        involving the specified symbol.

        For example, if `expr` is `x * 5 + x ** 2 + y * 2 + 5`, and `symbol` is `x`,
        this returns `x * 5 + x ** 2`.
        """
        expr = cls._preprocess(expr)
        return sympy.S.Zero + sum(
            term for term in sympy.Add.make_args(expr) if symbol in term.free_symbols
        )

    @staticmethod
    def factor_index_expr(expr: sympy.Expr, index_var: Symbol) -> sympy.Expr:
````
- **EN**: Introduces function `factor_index_expr`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`factor_index_expr`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 43-56 / 第 43-56 行
````python
        """
        Given an index expression, factor the expression around
        - FloorDiv(index_var, ...)
        - ModularIndexing(index_var, ...)
        - xindex

        e.g. FloorDiv(index_var, d0)*s0 + FloorDiv(index_var, d0)*s1 ->
        FloorDiv(index_var, d0) * (s0 + s1)
        """
        centres = OrderedSet()
        for sub in sympy.preorder_traversal(expr):
            if isinstance(sub, FloorDiv) and sub.args[0] == index_var:
                centres.add(sub)
            elif isinstance(sub, ModularIndexing) and sub.args[0] == index_var:
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `centres`. This range continues the implementation of function `BlockPatternMatcher.factor_index_expr`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。初始化或更新了 `centres` 等值。这一段延续了函数`BlockPatternMatcher.factor_index_expr` 的具体实现。

### Lines 57-70 / 第 57-70 行
````python
                centres.add(sub)
        centres.add(index_var)

        expr_out = expr
        for c in centres:
            expr_out = sympy.collect(expr_out, c)
        return expr_out

    @staticmethod
    def get_slice_numels(dims: list[Expr]) -> list[Expr]:
        """
        Compute the cumulative size of each dimension's slice.
        This proceeds from the last dim up to the second.
        """
````
- **EN**: Introduces function `get_slice_numels`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_slice_numels`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 71-84 / 第 71-84 行
````python
        numels = collections.deque([sympy.S.One])
        for dim in dims[:0:-1]:
            numel = dim * numels[0]
            numels.appendleft(numel)
        return [*numels]

    @staticmethod
    def _preprocess(expr: Expr) -> Expr:
        # Remove any Identity nodes, e.g. expand x + (5 * y) to x + 5 * y.
        # Disable mul and multinomial as those expansions affect op trees:
        # e.g. sympy expects to match Mul(a, b), but expansion has simplified
        # to Add(...). Even though they may be algebraically equivalent,
        # sympy `match` performs structural pattern matching
        return expr.expand(mul=False, multinomial=False, identity=True)
````
- **EN**: Introduces function `_preprocess`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_preprocess`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 85-98 / 第 85-98 行
````python

    @classmethod
    def match_mod_div_block_expr(
        cls,
        index: Expr,
        index_var: Symbol,
        numel: Expr,
        num_dims: int,
    ) -> tuple[list[Expr], list[Expr], list[Expr]] | None:
        """
        Matches modular indexing expressions, converting them to implied block dimensions and strides.
        See triton.py for more information.

        Warning: this function requires that `index`, `numel` and any other sympy
````
- **EN**: Introduces function `match_mod_div_block_expr`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里定义了函数`match_mod_div_block_expr`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 99-112 / 第 99-112 行
````python
        expression does not have precomputed replacements since otherwise block
        pattern matching may fail.
        See [Note: Precomputed replacements with BlockPatternMatch]
        """
        index = cls._preprocess(index)

        # Pattern match to find the strides and offset.
        wild_unsigned_int = functools.partial(
            cls._indexing_wild_unsigned_int, exclude=[index_var]
        )
        wild_signed_int = functools.partial(
            cls._indexing_wild_signed_int, exclude=[index_var]
        )
        dims: list[Expr] = [
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Initializes or updates values such as `index`, `wild_unsigned_int`, `wild_signed_int`, and `dims`. This range continues the implementation of function `BlockPatternMatcher.match_mod_div_block_expr`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。初始化或更新了 `index`、`wild_unsigned_int`、`wild_signed_int`、`dims` 等值。这一段延续了函数`BlockPatternMatcher.match_mod_div_block_expr` 的具体实现。

### Lines 113-126 / 第 113-126 行
````python
            wild_unsigned_int(f"dim_mod{idx}") for idx in range(num_dims)
        ]
        strides: list[Expr] = [
            wild_signed_int(f"stride_mod{idx}") for idx in range(num_dims)
        ]

        # The first dimension's index is computed by division.
        # The remaining are computed by modulo.
        slice_numels = cls.get_slice_numels(dims[:num_dims])
        block_index_exprs = [FloorDiv(index_var, slice_numels[0])] + [
            ModularIndexing(index_var, numel, dim)
            for dim, numel in zip(dims[1:], slice_numels[1:])
        ]

````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `strides`, `slice_numels`, and `block_index_exprs`. This range continues the implementation of function `BlockPatternMatcher.match_mod_div_block_expr`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。初始化或更新了 `strides`、`slice_numels`、`block_index_exprs` 等值。这一段延续了函数`BlockPatternMatcher.match_mod_div_block_expr` 的具体实现。

### Lines 127-140 / 第 127-140 行
````python
        # Calculate a linear index from block indices.
        match_expr = sympy_dot(strides, block_index_exprs)

        # Heuristic: if the number of dimensions is high, check that the minimum requirements
        # are met before attempting an expensive full match. see triton.py:match_mod_div_block
        # for more details. In short, here we check that each subexpression in sympy.Add contains
        # only FloorDiv or ModularIndexing expressions.
        if num_dims >= 5:
            stride = sympy.symbols("stride", cls=wild_signed_int)
            denom, other = sympy.symbols("denominator other", cls=wild_unsigned_int)
            mod_div_pattern = stride * ModularIndexing(index_var, denom, other)
            floor_div_pattern = stride * FloorDiv(index_var, denom)
            first_dim_floor_div_matched = False
            match_failed = False
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `match_expr`, `stride`, `mod_div_pattern`, `floor_div_pattern`, `first_dim_floor_div_matched`, and `match_failed`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含分支、循环或上下文管理等控制流。初始化或更新了 `match_expr`、`stride`、`mod_div_pattern`、`floor_div_pattern`、`first_dim_floor_div_matched`、`match_failed` 等值。

### Lines 141-154 / 第 141-154 行
````python
            for arg in sympy.Add.make_args(index):
                if arg.match(floor_div_pattern):
                    # There should only be a single FloorDiv(index, denom) expression
                    # corresponding to the first dimension
                    if first_dim_floor_div_matched:
                        match_failed = True
                        break
                    first_dim_floor_div_matched = True
                elif arg.match(mod_div_pattern):
                    continue
                else:
                    match_failed = True
                    break

````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `match_failed`, `first_dim_floor_div_matched`, and `else`. This range continues the implementation of function `BlockPatternMatcher.match_mod_div_block_expr`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。初始化或更新了 `match_failed`、`first_dim_floor_div_matched`、`else` 等值。这一段延续了函数`BlockPatternMatcher.match_mod_div_block_expr` 的具体实现。

### Lines 155-168 / 第 155-168 行
````python
            if match_failed:
                return None

        # Pattern match.
        match = index.match(match_expr)
        if match is None:
            return None

        # Provide default values for unmatched dims and strides.
        for dim in dims[1:]:
            if dim not in match:
                match[dim] = sympy.S.One
        for stride in strides[1:]:
            if stride not in match:
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `match`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `match` 等值。

### Lines 169-182 / 第 169-182 行
````python
                match[stride] = sympy.S.Zero

        # Replace wildcards with matched expressions.
        dims = [dims[0]] + [match[dim] for dim in dims[1:]]
        strides = [match[stride] for stride in strides]
        slice_numels = cls.get_slice_numels(dims)
        block_index_exprs = [sympy_subs(expr, match) for expr in block_index_exprs]

        sizevars = V.graph.sizevars

        # The leading dimension is not directly matched in our expression.
        # We solve for it by dividing the range tree numel by the product of
        # all other dimensions. We quit if they are not known to be divisible.
        assert dims[0] not in match, "Expected not to match the leading dimension!"
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `dims`, `strides`, `slice_numels`, `block_index_exprs`, and `sizevars`. This range continues the implementation of function `BlockPatternMatcher.match_mod_div_block_expr`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `dims`、`strides`、`slice_numels`、`block_index_exprs`、`sizevars` 等值。这一段延续了函数`BlockPatternMatcher.match_mod_div_block_expr` 的具体实现。

### Lines 183-196 / 第 183-196 行
````python
        if not sizevars.statically_known_multiple_of(numel, slice_numels[0]):
            return None
        dims[0] = numel / slice_numels[0]

        # Sanity check that we can recover the index from the matched subexpressions.
        matched_index = sympy_dot(strides, block_index_exprs)
        assert sizevars.statically_known_equals(
            matched_index,
            index,
        ), textwrap.dedent(
            f"""
            Invalid match!
            Index: {index}
            Matched expression: {matched_index}
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `matched_index`, and `Index`. This range continues the implementation of function `BlockPatternMatcher.match_mod_div_block_expr`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `matched_index`、`Index` 等值。这一段延续了函数`BlockPatternMatcher.match_mod_div_block_expr` 的具体实现。

### Lines 197-210 / 第 197-210 行
````python
            """
        )

        return dims, strides, block_index_exprs

    @classmethod
    def match_affine_block_expr(
        cls,
        index: Expr,
        index_var: Symbol,
    ) -> Expr | None:
        """
        Matches simple expressions of the form stride * index, returning the
        stride.
````
- **EN**: Introduces function `match_affine_block_expr`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`match_affine_block_expr`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 211-218 / 第 211-218 行
````python
        """
        index = cls._preprocess(index)
        stride = cls._indexing_wild_signed_int(name="stride", exclude=[index_var])
        m = index.match(index_var * stride)
        if m is None:
            return None

        return m[stride]
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `index`, `stride`, and `m`. This range continues the implementation of function `BlockPatternMatcher.match_affine_block_expr`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `index`、`stride`、`m` 等值。这一段延续了函数`BlockPatternMatcher.match_affine_block_expr` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Primary classes: `BlockPatternMatcher`  
  **CN**: 主要类：`BlockPatternMatcher`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections`, `functools`, `textwrap`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch.utils._ordered_set`, `torch.utils._sympy.functions`, `..utils`, `..virtualized`
