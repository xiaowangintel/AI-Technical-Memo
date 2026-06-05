# _size_hinting.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/experimental/_size_hinting.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Holds experimental tracing, shape reasoning, and transformation infrastructure built on FX.
- **Purpose (CN)**: 存放基于 FX 的实验性跟踪、形状推理与变换基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25
```python
"""
Size hinting utilities for symbolic shape expressions.

This module contains the core logic for resolving symbolic expressions to
concrete integer hints. Two strategies are provided:

- _guarding_hint_or_throw_base: strict, only uses backed symbol hints, throws on
  unbacked symbols. Use for correctness-critical guarding decisions.
- _optimization_hint_base: permissive, uses heuristics and fallbacks for unbacked
  symbols. Use for performance optimization decisions.
"""

from __future__ import annotations

import logging
import sys
from typing import Any, TYPE_CHECKING

import sympy

from torch.utils._sympy.numbers import int_oo


log = logging.getLogger(__name__)
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 26-43
```python
# Maximum number of free symbols in an expression before we skip
# sympy.factor() in optimization_hint process for unbacked.
# Factoring polynomials with many variables is expensive.
SYMPY_FACTOR_MAX_FREE_SYMBOLS = 50

if TYPE_CHECKING:
    from torch.fx.experimental.symbolic_shapes import ShapeEnv


def _sympy_subs(expr: sympy.Basic, replacements: dict[sympy.Expr, Any]) -> sympy.Basic:
    """
    When the passed replacement symbol v is a string, it is converted to a symbol with name v that
    have the same replaced expression integer and nonnegative properties.
    """

    def to_symbol(replaced: sympy.Expr, replacement: sympy.Expr | str) -> sympy.Symbol:
        if not isinstance(replaced, sympy.Expr):
            raise AssertionError(
```
- **EN**: This typing-only branch imports symbols for static analysis without affecting runtime behavior.
- **CN**: 这个仅用于类型检查的分支会导入静态分析所需符号，而不会影响运行时行为。

### Lines 44-66
```python
                f"Expected sympy.Expr key, got {type(replaced)}: {replaced}"
            )
        if isinstance(replacement, str):
            return sympy.Symbol(
                replacement,
                integer=replaced.is_integer,  # type: ignore[attr-defined]
                nonnegative=replaced.is_nonnegative,  # type: ignore[attr-defined]
            )
        else:
            return replacement

    # xreplace is faster than subs, but is way more picky
    return sympy.sympify(expr).xreplace(
        {k: to_symbol(k, v) for k, v in replacements.items()}
    )


def _maybe_realize_expr(
    expr: sympy.Basic, nan_fallback: int | None
) -> int | bool | None:
    """
    Handle special sympy values in hinting APIs.
```
- **EN**: This module-level block helps manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 67-84
```python
    Returns:
        - True/False for sympy.true/sympy.false (preserves bool type)
        - Raises ValueError for complex numbers
        - sys.maxsize for positive infinity
        - -sys.maxsize for negative infinity
        - fallback for NaN
        - None if no special handling needed
    """
    if expr is sympy.true:
        return True
    if expr is sympy.false:
        return False

    try:
        return int(expr)
    except (TypeError, ValueError):
        pass
```
- **EN**: This block continues `_maybe_realize_expr` and works to initialize learnable tensors and related state. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_maybe_realize_expr`，用于初始化可学习张量及相关状态。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 85-108
```python
    if isinstance(expr, sympy.Expr):
        if expr.has(sympy.I):
            raise ValueError(
                f"_maybe_realize_expr received a complex expression: {expr}. "
                "Tensor dimensions cannot be complex numbers."
            )
        if expr in (int_oo, sympy.oo):
            return sys.maxsize
        if expr in (-int_oo, -sympy.oo):
            return -sys.maxsize
        if nan_fallback is not None and (expr is sympy.nan or expr.has(sympy.nan)):
            return nan_fallback

    return None


def _guarding_hint_or_throw_base(
    shape_env: ShapeEnv,
    expr: sympy.Expr | sympy.Basic | int | bool,
    precomputed_replacements: dict[sympy.Expr, sympy.Symbol],
) -> int | bool:
    """
    Return a concrete integer hint for an expression that is safe to use for guarding.
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 109-126
```python
    This function evaluates the expression using only backed-symbols hints. Unlike
    _optimization_hint_base(), this function does NOT use heuristics or fallback values
    for unbacked symbols.

    Use this when you need a hint value that will be used for a guarding decision.

    Args:
        shape_env: The ShapeEnv instance.
        expr: A sympy expression or integer to evaluate.
        precomputed_replacements: Precomputed replacements for PRECOMPUTED_SIZE symbols.

    Returns:
        The concrete integer value of the expression based on backed symbol hints.

    Raises:
        GuardOnDataDependentSymNode: If the expression contains unbacked symbols
        (data-dependent values) that cannot be resolved to concrete values.
```
- **EN**: This block continues `_guarding_hint_or_throw_base` and works to track symbolic shape constraints and shape-dependent reasoning. It also iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `_guarding_hint_or_throw_base`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会遍历输入、节点或簿记结构。

### Lines 127-152
```python
    See Also:
        _optimization_hint_base: For cases where fallback/heuristic values are acceptable
            for unbacked symbols.
    """
    from torch.fx.experimental.symbolic_shapes import (
        has_free_unbacked_symbols,
        symbol_is_type,
        SymT,
    )

    # sympy.expand() doesn't work with boolean expressions like Or/And
    if isinstance(expr, sympy.Expr):
        expr = sympy.expand(expr).xreplace(shape_env.replacements)
    else:
        expr = sympy.sympify(expr).xreplace(shape_env.replacements)

    if isinstance(expr, sympy.Expr):
        expr = expr.expand(identity=True)

    result = _maybe_realize_expr(expr, None)
    if result is not None:
        return result

    if not isinstance(expr, sympy.Basic):
        raise RuntimeError("isinstance(expr, sympy.Basic)", expr, type(expr))
```
- **EN**: This block continues `_guarding_hint_or_throw_base` and works to track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `_guarding_hint_or_throw_base`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 153-178
```python
    if any(symbol_is_type(s, SymT.PRECOMPUTED_SIZE) for s in expr.free_symbols):  # type: ignore[attr-defined]
        expr = _sympy_subs(expr, precomputed_replacements)

    # TODO do we need sympy_subs, or just xreplace
    expr = _sympy_subs(expr, shape_env.backed_var_to_val)
    if isinstance(expr, sympy.Expr):
        expr = expr.expand(identity=True)

    if has_free_unbacked_symbols(expr):
        # Note: we could do better here and call
        # _maybe_evaluate_static(orig_expr, compute_hint=True)
        # but is it worth the overhead? probably not.
        raise shape_env._make_data_dependent_error(expr, expr)

    result = _maybe_realize_expr(expr, None)
    if result is None:
        raise RuntimeError("unexpected None!", expr)
    return result


def _get_unbacked_replacements(shape_env: ShapeEnv) -> dict[sympy.Expr, sympy.Expr]:
    """Builds a mapping from unbacked expressions to canonical equivalents
    using a union-find algorithm over deferred runtime asserts.
    Used by optimization_hint to resolve unbacked symbols to consistent values."""
    from collections import defaultdict
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 179-203
```python
    from torch.fx.experimental.symbolic_shapes import has_free_unbacked_symbols
    from torch.utils._ordered_set import OrderedSet

    if shape_env._unbacked_replacements is not None:
        return shape_env._unbacked_replacements

    class CanonicalExprFinder:
        """
        A disjoint-set/union-find data structure that can return the
        "canonical" expression for a group of equivalent expressions.
        - The canonical expression must come from the input eq_graph.
        - The heuristics used to choose a leader determines which
        expression becomes the canonical expression.
        """

        def __init__(self, eq_graph: dict[sympy.Expr, OrderedSet[sympy.Expr]]) -> None:
            self.eq_graph = eq_graph
            self.expressions = list(eq_graph.keys())
            self.reverse_expressions = {
                expr: i for i, expr in enumerate(self.expressions)
            }
            self.leader = list(range(len(self.expressions)))
            self.size = [1] * len(self.expressions)
            self._build_canonical_expr_mapping()
```
- **EN**: Defines the `_get_unbacked_replacements` function; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`_get_unbacked_replacements` 函数；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 204-230
```python
        def _build_canonical_expr_mapping(self) -> None:
            for expr, edges in self.eq_graph.items():
                for adj in edges:
                    self.union_expr(expr, adj)

        def union_expr(self, a: sympy.Expr, b: sympy.Expr) -> bool:
            return self.union(self.reverse_expressions[a], self.reverse_expressions[b])

        def union(self, a: int, b: int) -> bool:
            rootA = self.find(a)
            rootB = self.find(b)
            if rootA == rootB:
                return False
            leader, other = self.choose_leader(rootA, rootB)
            self.leader[other] = leader
            self.size[leader] += self.size[other]
            return True

        def find_expr(self, expr: sympy.Expr) -> sympy.Expr:
            parent = self.find(self.reverse_expressions[expr])
            return self.expressions[parent]

        def find(self, x: int) -> int:
            if self.leader[x] != x:
                self.leader[x] = self.find(self.leader[x])
            return self.leader[x]
```
- **EN**: Declares `CanonicalExprFinder`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `CanonicalExprFinder`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 231-256
```python
        def choose_leader(self, a: int, b: int) -> tuple[int, int]:
            """
            The leader will become the canonical expression.
            Returns a (leader, follower) tuple.

            Heuristics:
            1. Backed expression or constants preferred over unbacked expr
            2. Simpler sub-expr when one contains the other
            3. Higher frequency across equalities from deferred runtime assertions
            4. Size of the set
            5. Fallback to sympy.Basic.compare
            """

            def _choose(x: int, y: int) -> bool:
                lhs, rhs = self.expressions[x], self.expressions[y]

                any_unbacked_lhs = has_free_unbacked_symbols(lhs)
                any_unbacked_rhs = has_free_unbacked_symbols(rhs)
                if any_unbacked_lhs != any_unbacked_rhs:
                    return bool(any_unbacked_rhs)

                if lhs.has(rhs):
                    return False
                elif rhs.has(lhs):
                    return True
```
- **EN**: Defines the `_get_unbacked_replacements.CanonicalExprFinder.choose_leader` method; this block introduces logic that hand work to a compiler/backend pipeline.
- **CN**: 定义`_get_unbacked_replacements.CanonicalExprFinder.choose_leader` 方法；该代码块引入了用于将工作移交给编译器或后端流水线的逻辑。

### Lines 257-283
```python
                degrees_lhs = len(self.eq_graph[lhs])
                degrees_rhs = len(self.eq_graph[rhs])
                if degrees_lhs != degrees_rhs:
                    return degrees_lhs > degrees_rhs

                if self.size[x] != self.size[y]:
                    return self.size[x] > self.size[y]

                return lhs.compare(rhs) == -1

            if _choose(a, b):
                return a, b
            return b, a

    # Build an undirected graph using ShapeEnv's deferred runtime assertions.
    shape_env._equality_graph = defaultdict(OrderedSet)
    for assertions in shape_env.deferred_runtime_asserts.values():
        for assertion in assertions:
            if not isinstance(assertion.expr, sympy.Equality):
                continue
            lhs = sympy.sympify(assertion.expr.lhs)
            rhs = sympy.sympify(assertion.expr.rhs)
            shape_env._equality_graph[lhs].add(rhs)
            shape_env._equality_graph[rhs].add(lhs)

    uf = CanonicalExprFinder(shape_env._equality_graph)
```
- **EN**: This block continues `_get_unbacked_replacements` and works to track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_get_unbacked_replacements`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 284-301
```python
    shape_env._unbacked_replacements = {}
    for expr in shape_env._equality_graph:
        canonical_expr = uf.find_expr(expr)
        if expr != canonical_expr:
            shape_env._unbacked_replacements[expr] = canonical_expr

    return shape_env._unbacked_replacements


def _sub_unbacked_exprs(shape_env: ShapeEnv, expr: sympy.Expr) -> sympy.Expr:
    """Substitute unbacked expressions with canonical equivalents.
    Used by optimization_hint to maximize consistency when hinting unbacked symbols."""
    replacements = _get_unbacked_replacements(shape_env)

    # consider making this threshold configurable
    sub_cnt_limit = 30
    sub_cnt = 0
    while sub_cnt < sub_cnt_limit:
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 302-326
```python
        new_expr = expr.subs(replacements)
        if new_expr == expr:
            break
        if len(new_expr.free_symbols) <= SYMPY_FACTOR_MAX_FREE_SYMBOLS:
            expr = sympy.factor(new_expr)
        else:
            expr = new_expr
        sub_cnt += 1
    else:
        log.warning("Substitution limit (%d) reached w/ %s", sub_cnt_limit, expr)

    expr = _sympy_subs(expr, shape_env.backed_var_to_val)
    expr = _sympy_subs(expr, shape_env.var_to_hint_override)
    return expr


def _optimization_hint_base(
    shape_env: ShapeEnv,
    expr: sympy.Expr | int,
    precomputed_replacements: dict[sympy.Expr, sympy.Symbol],
    fallback: int | None = None,
) -> int:
    """
    Return a concrete integer hint for an expression using heuristics.
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 327-352
```python
    This function should be used for non-guarding based optimizations.
    It will hint unbacked symbols using user provided optimization hints.
    If not provided, fallback will be used along with some heuristics
    that try to maximize consistency with the shape environment.

    Args:
        shape_env: The ShapeEnv instance.
        expr: A sympy expression or integer to evaluate.
        precomputed_replacements: Precomputed replacements for PRECOMPUTED_SIZE symbols.
        fallback: Fallback value for unbacked symbols. If None, reads from config.

    Returns:
        A concrete integer hint for the expression.
    """
    from torch.fx.experimental.symbolic_shapes import (
        has_free_unbacked_symbols,
        symbol_is_type,
        SymT,
    )

    # Read config at call time to respect runtime patches (e.g., in tests)
    if fallback is None:
        from torch._inductor.config import unbacked_symint_fallback

        fallback = unbacked_symint_fallback
```
- **EN**: This block continues `_optimization_hint_base` and works to track symbolic shape constraints and shape-dependent reasoning. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `_optimization_hint_base`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会根据运行时条件分支处理。

### Lines 353-378
```python
    # to have expanded (Identity free) expr stored in original
    if isinstance(expr, sympy.Expr):
        expr = expr.expand(identity=True)

    original = expr
    # sympy.expand() doesn't work with boolean expressions like Or/And
    if isinstance(expr, sympy.Expr):
        expr = expr.xreplace(shape_env.replacements)
    else:
        expr = sympy.sympify(expr).xreplace(shape_env.replacements)

    result = _maybe_realize_expr(expr, fallback)
    if result is not None:
        return result

    if isinstance(expr, sympy.Expr):
        expr = expr.expand(identity=True)

    # Replace backed symbols with their hints, leaving unbacked symbols alone.
    result = _maybe_realize_expr(expr, None)
    if result is not None:
        return result

    if not isinstance(expr, sympy.Expr):
        raise RuntimeError("isinstance(expr, sympy.Expr)", expr)
```
- **EN**: This block continues `_optimization_hint_base` and works to track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_optimization_hint_base`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 379-396
```python
    if any(symbol_is_type(s, SymT.PRECOMPUTED_SIZE) for s in expr.free_symbols):  # type: ignore[attr-defined]
        expr = _sympy_subs(expr, precomputed_replacements)

    expr = _sympy_subs(expr, shape_env.backed_var_to_val)
    if isinstance(expr, sympy.Expr):
        expr = expr.expand(identity=True)

    result = _maybe_realize_expr(expr, fallback)
    if result is not None:
        return result

    expr = _sympy_subs(expr, shape_env.var_to_hint_override)

    result = _maybe_realize_expr(expr, fallback)
    if result is not None:
        return result

    # If unbacked symbols remain, try to substitute them using heuristics
```
- **EN**: This block continues `_optimization_hint_base` and works to track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_optimization_hint_base`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 397-416
```python
    # that maximize consistency with the shape environment.
    if has_free_unbacked_symbols(expr):
        # Make sure to substitute with the factored version
        # e.g. 10*(s0 + u0) instead of 10*s0 + 10*u0
        if (
            isinstance(original, sympy.Expr)
            and len(original.free_symbols) <= SYMPY_FACTOR_MAX_FREE_SYMBOLS
        ):
            original = sympy.factor(original)
        expr = _sub_unbacked_exprs(shape_env, original)

    # For multiple expressions that depend on an unbacked symint,
    # we want to compute them consistently for a size hint we have chosen.
    # So, recursively compute expressions via size hints of contained symbols.
    # For example: u1 * u2 - 10 ==> fallback * fallback - 10

    if not isinstance(expr, sympy.Expr):
        raise RuntimeError(f"Expected sympy Expr, got {type(expr)}: {expr}")
    free_symbols = expr.free_symbols
```
- **EN**: This block continues `_optimization_hint_base` and works to track symbolic shape constraints and shape-dependent reasoning. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_optimization_hint_base`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 417-443
```python
    # Constrain fallback per-symbol based on var_to_range bounds
    size_dict = {}
    for s in free_symbols:
        sym_fallback = fallback
        vr = shape_env.var_to_range.get(s, None)
        if vr is not None:
            if isinstance(vr.lower, (int, sympy.Integer)):
                sym_fallback = max(sym_fallback, int(vr.lower))
            if isinstance(vr.upper, (int, sympy.Integer)):
                sym_fallback = min(sym_fallback, int(vr.upper))
        size_dict[s] = sym_fallback

    try:
        final_result = expr.subs(size_dict)
    except ZeroDivisionError:
        # Expressions like ModularIndexing(x, u1, 4) crash during subs()
        # when u1 is substituted with 0, because sympy eagerly evaluates
        # (x // 0) % 4.  This can happen when an unbacked symbol with
        # var_to_range lower=0 is used as a divisor (e.g. from
        # _dynamic_reshape_indexer) and the fallback also maps to 0.
        # Return fallback in that case.
        return fallback if fallback is not None else 0

    final_result = _maybe_realize_expr(final_result, fallback)
    if final_result is None:
        raise RuntimeError(f"Failed to realize expression to int: {expr}")
```
- **EN**: This block continues `_optimization_hint_base` and works to track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `_optimization_hint_base`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 444-444
```python
    return final_result
```
- **EN**: This block continues `_optimization_hint_base` and works to manipulate FX graphs, tracing helpers, or transformation utilities. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `_optimization_hint_base`，用于操作 FX 图、跟踪辅助逻辑或变换工具。 同时它还会计算并返回中间值或结果。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Guards / 守卫**
  - EN: Runtime predicates preserve correctness when cached compiled graphs are reused.
  - CN: 运行时谓词用于在复用缓存编译图时保持正确性。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch.utils._sympy.numbers`, `torch.fx.experimental.symbolic_shapes`, `torch.utils._ordered_set`, `torch._inductor.config`
- **Standard library / 标准库**: `__future__`, `logging`, `sys`, `typing`, `collections`
- **Third-party packages / 第三方包**: `sympy`
- **Primary symbols / 核心符号**: `SYMPY_FACTOR_MAX_FREE_SYMBOLS`, `_sympy_subs`, `_maybe_realize_expr`, `_guarding_hint_or_throw_base`, `_get_unbacked_replacements`, `_sub_unbacked_exprs`, `_optimization_hint_base`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
