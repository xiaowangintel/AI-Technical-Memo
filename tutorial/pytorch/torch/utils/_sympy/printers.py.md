# printers.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/utils/_sympy/printers.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements utility logic that supports broader PyTorch execution, packaging, or developer workflows. This specific file centers on `printers.py`. It bridges PyTorch values or graphs into visualization-friendly event formats.
- **Purpose (CN) / 用途（中文）**: 实现支撑更广泛 PyTorch 执行、打包或开发者工作流的辅助逻辑。 该文件具体围绕 `printers.py` 展开。 它把 PyTorch 数值或计算图桥接到适合可视化的事件格式。
## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第 1-30 行
```python
import sys

import sympy
from sympy.printing.precedence import PRECEDENCE, precedence
from sympy.printing.str import StrPrinter


INDEX_TYPE = "int64_t"
INDEX_TYPE_MAX = (1 << 63) - 1
INDEX_TYPE_MIN = -1 << 63


# This printer contains rules that are supposed to be generic for both C/C++ and
# Python
class ExprPrinter(StrPrinter):
    # override this so that _print_FloorDiv is used
    printmethod = "_torch_sympystr"

    def _print_Mul(self, expr: sympy.Expr) -> str:
        return self.stringify(expr.args, "*", precedence(expr))

    def _print_Not(self, expr: sympy.Expr) -> str:
        # pyrefly: ignore [missing-attribute]
        return f"not ({self._print(expr.args[0])})"

    def _print_Add(self, expr: sympy.Expr, order: str | None = None) -> str:
        return self.stringify(expr.args, " + ", precedence(expr))

    def _print_Relational(self, expr: sympy.Expr) -> str:
        return self.stringify(expr.args, f" {expr.rel_op} ", precedence(expr))
```
- **EN**: It introduces or extends class-level abstractions such as `ExprPrinter`, which organize state and behavior for this subsystem. Named constants such as `INDEX_TYPE`, `INDEX_TYPE_MAX`, `INDEX_TYPE_MIN` centralize shared configuration or sentinel values. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `ExprPrinter` 等类级抽象，用于组织该子系统的状态与行为。 `INDEX_TYPE, INDEX_TYPE_MAX, INDEX_TYPE_MIN` 等具名常量把共享配置或哨兵值集中定义在一起。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 32-55 / 第 32-55 行
```python
    def _print_BitwiseFn_bitwise_and(self, expr: sympy.Expr) -> str:
        return self.stringify(expr.args, " & ", PRECEDENCE["BitwiseAnd"])

    def _print_BitwiseFn_bitwise_or(self, expr: sympy.Expr) -> str:
        return self.stringify(expr.args, " | ", PRECEDENCE["BitwiseOr"])

    def _print_BitwiseFn_bitwise_xor(self, expr: sympy.Expr) -> str:
        return self.stringify(expr.args, " ^ ", PRECEDENCE["BitwiseXor"])

    # NB: this is OK to put here, because Mod is only defined for positive
    # numbers, and so across C/Python its behavior is consistent
    def _print_Mod(self, expr: sympy.Expr) -> str:
        return self.stringify(expr.args, " % ", PRECEDENCE["Atom"] - 0.5)

    def _print_FloatTrueDiv(self, expr: sympy.Expr) -> str:
        s = self.stringify(expr.args, " / ", PRECEDENCE["Atom"] - 0.5)
        return f"({s})"

    def _print_CleanDiv(self, expr: sympy.Expr) -> str:
        return self._print_FloorDiv(expr)

    def _print_Identity(self, expr: sympy.Expr) -> str:
        # pyrefly: ignore [missing-attribute]
        return self._print(expr.args[0])
```
- **EN**: It introduces or extends class-level abstractions such as `ExprPrinter`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `ExprPrinter` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 57-87 / 第 57-87 行
```python
    def _print_Float(self, expr: sympy.Expr) -> str:
        if expr._prec == 53:
            # IEEE-754 double precision have 53 bits. SymPy prints them with
            # 15 digits, but we need 17 for round-trip correctness
            return str(sympy.Float(expr, dps=17))
        else:
            # We don't use other precisions in pytorch
            return str(expr)

    # This must be implemented because sympy will collect x * x into Pow(x, 2), without
    # any explicit intervention.  We print it just like x * x, notably, we
    # never generate sympy.Pow with floats.
    #
    # NB: this pow by natural, you should never have used builtin sympy.pow
    # for FloatPow, and a symbolic exponent should be PowByNatural.  These
    # means exp is guaranteed to be integer.
    def _print_Pow(self, expr: sympy.Expr) -> str:
        base, exp = expr.args
        if exp != int(exp):
            raise AssertionError(exp)
        exp = int(exp)
        if exp < 0:
            raise AssertionError(f"exponent must be non-negative, got {exp}")
        if exp > 0:
            return self.stringify([base] * exp, "*", PRECEDENCE["Mul"])
        return "1"

    # Explicit NotImplemented functions are to prevent default sympy printing
    # behavior, which will just barf out ToFloat(...) to your IR.  The error
    # message is better here because it tells you which printer class it needs
    # to go in.
```
- **EN**: It introduces or extends class-level abstractions such as `ExprPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `ExprPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 89-118 / 第 89-118 行
```python
    def _print_ToFloat(self, expr: sympy.Expr) -> str:
        raise NotImplementedError(f"_print_ToFloat not implemented for {type(self)}")

    def _print_Infinity(self, expr: sympy.Expr) -> str:
        raise NotImplementedError(f"_print_Infinity not implemented for {type(self)}")

    def _print_NegativeInfinity(self, expr: sympy.Expr) -> str:
        raise NotImplementedError(
            f"_print_NegativeInfinity not implemented for {type(self)}"
        )

    def _print_NaN(self, expr: sympy.Expr) -> str:
        raise NotImplementedError(f"_print_NaN not implemented for {type(self)}")

    def _print_FloorDiv(self, expr: sympy.Expr) -> str:
        raise NotImplementedError(f"_print_FloorDiv not implemented for {type(self)}")

    def _print_PythonMod(self, expr: sympy.Expr) -> str:
        raise NotImplementedError(f"_print_PythonMod not implemented for {type(self)}")

    def _print_IntTrueDiv(self, expr: sympy.Expr) -> str:
        raise NotImplementedError(f"_print_IntTrueDiv not implemented for {type(self)}")

    def _print_PowByNatural(self, expr: sympy.Expr) -> str:
        raise NotImplementedError(
            f"_print_PowByNatural not implemented for {type(self)}"
        )

    def _print_FloatPow(self, expr: sympy.Expr) -> str:
        raise NotImplementedError(f"_print_FloatPow not implemented for {type(self)}")
```
- **EN**: It introduces or extends class-level abstractions such as `ExprPrinter`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 它引入或扩展了 `ExprPrinter` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 120-139 / 第 120-139 行
```python
    def _print_TruncToInt(self, expr: sympy.Expr) -> str:
        raise NotImplementedError(f"_print_TruncToInt not implemented for {type(self)}")

    def _print_RoundToInt(self, expr: sympy.Expr) -> str:
        raise NotImplementedError(f"_print_RoundToInt not implemented for {type(self)}")

    def _print_RoundDecimal(self, expr: sympy.Expr) -> str:
        raise NotImplementedError(
            f"_print_RoundDecimal not implemented for {type(self)}"
        )

    # NB: Some float operations are INTENTIONALLY not implemented for
    # printers.  You can implement them as a quick unblock, but it is better
    # to ask yourself why we haven't done this computation in the Tensor
    # universe instead

    def _print_TruncToFloat(self, expr: sympy.Expr) -> str:
        raise NotImplementedError(
            f"_print_TruncToFloat not implemented for {type(self)}"
        )
```
- **EN**: It introduces or extends class-level abstractions such as `ExprPrinter`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 它引入或扩展了 `ExprPrinter` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 142-166 / 第 142-166 行
```python
class PythonPrinter(ExprPrinter):
    def _print_ToFloat(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("ToFloat expects exactly one argument")
        # NB: We use sym_float here because the printer is used for cache
        # serialization, and cache guards get evaluated with SymInt to
        # propagate guards to the parent ShapeEnv.  However, this comes at a
        # runtime cost for guards involving float.  If this is unacceptable
        # overhead, what you want to do is have two separate printers for
        # SymInt, one for when the inputs are guaranteed to be int, and
        # another for when they could be SymInt.
        #
        # NB: sym_min/sym_max also have this problem, but I chose not to fix
        # those.
        #
        # See https://github.com/pytorch/pytorch/issues/142507 for more
        # context.
        # pyrefly: ignore [missing-attribute]
        return f"torch.sym_float({self._print(expr.args[0])})"

    def _print_And(self, expr: sympy.Expr) -> str:
        return self.stringify(expr.args, " and ", precedence(expr))

    def _print_Or(self, expr: sympy.Expr) -> str:
        return self.stringify(expr.args, " or ", precedence(expr))
```
- **EN**: It introduces or extends class-level abstractions such as `PythonPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `PythonPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 168-197 / 第 168-197 行
```python
    def _print_ModularIndexing(self, expr: sympy.Expr) -> str:
        x, div, mod = (
            self.parenthesize(arg, PRECEDENCE["Atom"] - 0.5) for arg in expr.args
        )
        if div != "1":
            x = f"({x} // {div})"
        return f"({x} % {mod})"

    def _print_Infinity(self, expr: sympy.Expr) -> str:
        return "math.inf"

    def _print_NegativeInfinity(self, expr: sympy.Expr) -> str:
        return "-math.inf"

    def _print_NaN(self, expr: sympy.Expr) -> str:
        return "math.nan"

    # WARNING: this is dangerous for Triton, which has C-style modulus
    def _print_PythonMod(self, expr: sympy.Expr) -> str:
        return self.stringify(expr.args, " % ", PRECEDENCE["Atom"] - 0.5)

    # WARNING: this is dangerous for Triton, which has C-style modulus
    def _print_FloorDiv(self, expr: sympy.Expr) -> str:
        x, div = (self.parenthesize(arg, PRECEDENCE["Atom"] - 0.5) for arg in expr.args)
        return f"{x} // {div}"

    # WARNING: this is dangerous for Triton, when lhs, rhs > 2**53, Python
    # does a special algorithm
    def _print_IntTrueDiv(self, expr: sympy.Expr) -> str:
        return self.stringify(expr.args, " / ", PRECEDENCE["Atom"] - 0.5)
```
- **EN**: It introduces or extends class-level abstractions such as `PythonPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `PythonPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 199-226 / 第 199-226 行
```python
    def _helper_sqrt(self, expr: sympy.Expr) -> str:
        # NB: We use torch._sym_sqrt here instead of math.sqrt because the
        # guard expression may be evaluated with SymInt/SymFloat inputs (e.g.
        # during cache hit re-evaluation in evaluate_guards_expression).
        # math.sqrt on a SymFloat triggers evaluate_expr which forces
        # concretization/specialization of the symbol, creating spurious
        # guards that didn't exist in the original program.
        # torch._sym_sqrt properly propagates through the symbolic system
        # without forcing specialization.
        # See https://github.com/pytorch/pytorch/issues/152435
        # pyrefly: ignore [missing-attribute]
        return f"torch._sym_sqrt({self._print(expr)})"

    def _print_OpaqueUnaryFn_sqrt(self, expr: sympy.Expr) -> str:
        return self._helper_sqrt(expr.args[0])

    def _print_FloatPow(self, expr: sympy.Expr) -> str:
        return self.stringify(expr.args, " ** ", PRECEDENCE["Pow"])

    # TODO: Not sure this works with Triton, even when base/exp are integral
    def _print_PowByNatural(self, expr: sympy.Expr) -> str:
        return self.stringify(expr.args, " ** ", PRECEDENCE["Pow"])

    def _print_floor(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("floor expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"math.floor({self._print(expr.args[0])})"
```
- **EN**: It introduces or extends class-level abstractions such as `PythonPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `PythonPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 228-257 / 第 228-257 行
```python
    def _print_FloorToInt(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("FloorToInt expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"math.floor({self._print(expr.args[0])})"

    def _print_TruncToInt(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("TruncToInt expects exactly one argument")
        # This also could have been int(), they'll do the same thing for float
        # pyrefly: ignore [missing-attribute]
        return f"math.trunc({self._print(expr.args[0])})"

    def _print_ceiling(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("ceiling expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"math.ceil({self._print(expr.args[0])})"

    def _print_CeilToInt(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("CeilToInt expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"math.ceil({self._print(expr.args[0])})"

    def _print_Abs(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("Abs expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"abs({self._print(expr.args[0])})"
```
- **EN**: It introduces or extends class-level abstractions such as `PythonPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `PythonPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 259-284 / 第 259-284 行
```python
    # NB: It's expected that we've made explicit any promotion in the sympy
    # expression, so it doesn't matter that Python max/min doesn't perform
    # promotion
    def _print_Max(self, expr: sympy.Expr) -> str:
        if len(expr.args) < 2:
            raise AssertionError("Max expects at least two arguments")
        # pyrefly: ignore [missing-attribute]
        return f"max({', '.join(map(self._print, expr.args))})"

    def _print_Min(self, expr: sympy.Expr) -> str:
        if len(expr.args) < 2:
            raise AssertionError("Min expects at least two arguments")
        # pyrefly: ignore [missing-attribute]
        return f"min({', '.join(map(self._print, expr.args))})"

    def _print_OpaqueUnaryFn_cos(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("cos expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"math.cos({self._print(expr.args[0])})"

    def _print_OpaqueUnaryFn_cosh(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("cosh expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"math.cosh({self._print(expr.args[0])})"
```
- **EN**: It introduces or extends class-level abstractions such as `PythonPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `PythonPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 286-314 / 第 286-314 行
```python
    def _print_OpaqueUnaryFn_acos(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("acos expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"math.acos({self._print(expr.args[0])})"

    def _print_OpaqueUnaryFn_sin(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("sin expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"math.sin({self._print(expr.args[0])})"

    def _print_OpaqueUnaryFn_sinh(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("sinh expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"math.sinh({self._print(expr.args[0])})"

    def _print_OpaqueUnaryFn_asin(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("asin expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"math.asin({self._print(expr.args[0])})"

    def _print_OpaqueUnaryFn_tan(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("tan expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"math.tan({self._print(expr.args[0])})"
```
- **EN**: It introduces or extends class-level abstractions such as `PythonPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `PythonPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 316-338 / 第 316-338 行
```python
    def _print_OpaqueUnaryFn_tanh(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("tanh expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"math.tanh({self._print(expr.args[0])})"

    def _print_OpaqueUnaryFn_atan(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("atan expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"math.atan({self._print(expr.args[0])})"

    def _print_OpaqueUnaryFn_log2(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("log2 expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"math.log2({self._print(expr.args[0])})"

    def _print_RoundToInt(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("RoundToInt expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"round({self._print(expr.args[0])})"
```
- **EN**: It introduces or extends class-level abstractions such as `PythonPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `PythonPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 340-367 / 第 340-367 行
```python
    def _print_RoundDecimal(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 2:
            raise AssertionError("RoundDecimal expects exactly two arguments")
        number, ndigits = expr.args
        if not isinstance(ndigits, sympy.Integer):
            raise TypeError("ndigits must be an instance of sympy.Integer")
        # pyrefly: ignore [missing-attribute]
        return f"round({self._print(number)}, {ndigits})"

    def _print_Piecewise(self, expr: sympy.Expr) -> str:
        # Convert Piecewise(expr_cond_pairs) to nested ternary expressions
        # Piecewise((e1, c1), (e2, c2), ..., (eN, cN))
        # becomes: e1 if c1 else (e2 if c2 else (... else eN))
        result: str | None = None
        for expr_i, cond_i in reversed(expr.args):
            # pyrefly: ignore [missing-attribute]
            expr_str = self._print(expr_i)
            if cond_i == True:  # noqa: E712
                # This is the default case
                result = expr_str
            else:
                # pyrefly: ignore [missing-attribute]
                cond_str = self._print(cond_i)
                if result is None:
                    result = expr_str
                else:
                    result = f"({expr_str} if {cond_str} else {result})"
        return result if result else "0"
```
- **EN**: It introduces or extends class-level abstractions such as `PythonPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `PythonPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 370-391 / 第 370-391 行
```python
class CppPrinter(ExprPrinter):
    def _print_Integer(self, expr: sympy.Expr) -> str:
        suffix = "LL" if sys.platform in ["darwin", "win32"] else "L"
        i = int(expr)
        if i > INDEX_TYPE_MAX or i < INDEX_TYPE_MIN:
            raise OverflowError(f"{i} too big to convert to {INDEX_TYPE}")
        elif i == INDEX_TYPE_MIN:
            if i != (-1) << 63:
                raise AssertionError("unexpected minimum index type value")
            # Writing -9223372036854775808L makes the value overflow
            # as it is parsed as -(9223372036854775808L) by the C/C++ compiler
            return f"(-1{suffix} << 63)"
        return f"{i}{suffix}"

    def _print_Where(self, expr: sympy.Expr) -> str:
        c, p, q = (
            self.parenthesize(arg, PRECEDENCE["Atom"] - 0.5) for arg in expr.args
        )
        return f"{c} ? {p} : {q}"

    def _print_Or(self, expr: sympy.Expr) -> str:
        return self.stringify(expr.args, " || ", precedence(expr))
```
- **EN**: It introduces or extends class-level abstractions such as `CppPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `CppPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 393-421 / 第 393-421 行
```python
    def _print_Piecewise(self, expr: sympy.Expr) -> str:
        # Convert Piecewise(expr_cond_pairs) to nested ternary operators
        # Piecewise((e1, c1), (e2, c2), ..., (eN, cN))
        # becomes: c1 ? e1 : (c2 ? e2 : (... : eN))
        result: str | None = None
        for expr_i, cond_i in reversed(expr.args):
            expr_str = self.parenthesize(expr_i, PRECEDENCE["Atom"] - 0.5)
            if cond_i == True:  # noqa: E712
                # This is the default case
                result = expr_str
            else:
                cond_str = self.parenthesize(cond_i, PRECEDENCE["Atom"] - 0.5)
                if result is None:
                    result = expr_str
                else:
                    result = f"{cond_str} ? {expr_str} : {result}"
        return f"({result})" if result else "0"

    def _print_ModularIndexing(self, expr: sympy.Expr) -> str:
        x, div, mod = expr.args
        x = self.doprint(x)
        if div != 1:
            div = self.doprint(div)
            if expr.is_integer:
                x = f"c10::div_floor_integer(static_cast<int64_t>({x}), static_cast<int64_t>({div}))"
            else:
                x = f"c10::div_floor_floating(static_cast<double>({x}), static_cast<double>({div}))"
        mod = self.doprint(mod)
        return f"(static_cast<{INDEX_TYPE}>({x}) % static_cast<{INDEX_TYPE}>({mod}))"
```
- **EN**: It introduces or extends class-level abstractions such as `CppPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `CppPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 423-450 / 第 423-450 行
```python
    def _print_FloorDiv(self, expr: sympy.Expr) -> str:
        x, div = expr.args
        x = self.doprint(x)
        div = self.doprint(div)
        if expr.is_integer:
            return f"c10::div_floor_integer(static_cast<int64_t>({x}), static_cast<int64_t>({div}))"
        return f"c10::div_floor_floating(static_cast<double>({x}), static_cast<double>({div}))"

    def _print_floor(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("floor expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        r = f"std::floor({self._print(expr.args[0])})"
        return f"static_cast<{INDEX_TYPE}>({r})" if expr.is_integer else r

    def _print_FloorToInt(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("FloorToInt expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        r = f"std::floor({self._print(expr.args[0])})"
        return f"static_cast<{INDEX_TYPE}>({r})" if expr.is_integer else r

    def _print_TruncToInt(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("TruncToInt expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        r = f"std::trunc({self._print(expr.args[0])})"
        return f"static_cast<{INDEX_TYPE}>({r})"
```
- **EN**: It introduces or extends class-level abstractions such as `CppPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `CppPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 452-474 / 第 452-474 行
```python
    def _print_TruncToFloat(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("TruncToFloat expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"std::trunc({self._print(expr.args[0])})"

    def _print_ToFloat(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("ToFloat expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"static_cast<double>({self._print(expr.args[0])})"

    def _print_PythonMod(self, expr: sympy.Expr) -> str:
        x, div = expr.args
        x = self.doprint(x)
        div = self.doprint(div)
        return f"c10::div_mod({x}, {div})"

    def _print_IntTrueDiv(self, expr: sympy.Expr) -> str:
        lhs, rhs = expr.args
        # TODO: This is only accurate up to 2**53
        # pyrefly: ignore [missing-attribute]
        return f"static_cast<double>({self._print(lhs)}) / static_cast<double>({self._print(rhs)})"
```
- **EN**: It introduces or extends class-level abstractions such as `CppPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `CppPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 476-507 / 第 476-507 行
```python
    # TODO: PowByNatural: we need to implement our own int-int pow.  Do NOT
    # use std::pow, that operates on floats
    def _print_PowByNatural(self, expr: sympy.Expr) -> str:
        # Implement the special-case of 2**x for now
        base, exp = expr.args
        if base == 2:
            # pyrefly: ignore [missing-attribute]
            return f"(1 << ({self._print(exp)}))"
        raise NotImplementedError(
            f"_print_PowByNatural not implemented for {type(self)}"
        )

    def _print_FloatPow(self, expr: sympy.Expr) -> str:
        base, exp = expr.args
        # pyrefly: ignore [missing-attribute]
        return f"std::pow({self._print(base)}, {self._print(exp)})"

    def _print_Pow(self, expr: sympy.Expr) -> str:
        # Uses float constants to perform FP div
        base, exp = expr.args

        if exp == 0.5 or exp == -0.5:
            # pyrefly: ignore [missing-attribute]
            base = self._print(base)
            return f"std::sqrt({base})" if exp == 0.5 else f"1.0/std::sqrt({base})"
        if exp.is_integer:
            exp = int(exp)
            if exp > 0:
                r = self.stringify([base] * exp, "*", PRECEDENCE["Mul"])
            elif exp < -1:
                r = (
                    "1.0/("
```
- **EN**: It introduces or extends class-level abstractions such as `CppPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `CppPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 508-535 / 第 508-535 行
```python
                    + self.stringify([base] * abs(exp), "*", PRECEDENCE["Mul"])
                    + ")"
                )
            elif exp == -1:
                # pyrefly: ignore [missing-attribute]
                r = "1.0/" + self._print(base)
            else:  # exp == 0
                r = "1.0"

            return f"static_cast<{INDEX_TYPE}>({r})" if expr.is_integer else r
        else:
            # TODO: float vs double
            return f"std::pow({base}, {float(exp)})"

    def _print_Rational(self, expr: sympy.Expr) -> str:
        # Uses float constants to perform FP div
        if expr.q == 1:
            r = f"{expr.p}"
        else:
            r = f"{expr.p}.0/{expr.q}.0"
        return f"static_cast<{INDEX_TYPE}>({r})" if expr.is_integer else r

    def _print_ceiling(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("ceiling expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        r = f"std::ceil({self._print(expr.args[0])})"
        return f"static_cast<{INDEX_TYPE}>({r})" if expr.is_integer else r
```
- **EN**: It introduces or extends class-level abstractions such as `CppPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `CppPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 537-562 / 第 537-562 行
```python
    def _print_CeilToInt(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("CeilToInt expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        r = f"std::ceil({self._print(expr.args[0])})"
        return f"static_cast<{INDEX_TYPE}>({r})" if expr.is_integer else r

    def _print_Min(self, expr: sympy.Expr) -> str:
        # pyrefly: ignore [missing-attribute]
        args = [self._print(a) for a in expr.args]
        if len(args) == 2:
            return f"std::min(static_cast<{INDEX_TYPE}>({args[0]}), static_cast<{INDEX_TYPE}>({args[1]}))"
        else:
            # Initializer list overload
            il = "{" + ", ".join(args) + "}"
            return f"std::min<{INDEX_TYPE}>({il})"

    def _print_Max(self, expr: sympy.Expr) -> str:
        # pyrefly: ignore [missing-attribute]
        args = [self._print(a) for a in expr.args]
        if len(args) == 2:
            return f"std::max(static_cast<{INDEX_TYPE}>({args[0]}), static_cast<{INDEX_TYPE}>({args[1]}))"
        else:
            # Initializer list overload
            il = "{" + ", ".join(args) + "}"
            return f"std::max<{INDEX_TYPE}>({il})"
```
- **EN**: It introduces or extends class-level abstractions such as `CppPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `CppPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 564-592 / 第 564-592 行
```python
    def _print_Abs(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("Abs expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"std::abs({self._print(expr.args[0])})"

    def _print_OpaqueUnaryFn_cos(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("cos expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"std::cos({self._print(expr.args[0])})"

    def _print_OpaqueUnaryFn_cosh(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("cosh expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"std::cosh({self._print(expr.args[0])})"

    def _print_OpaqueUnaryFn_acos(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("acos expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"std::acos({self._print(expr.args[0])})"

    def _print_OpaqueUnaryFn_sin(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("sin expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"math.sin({self._print(expr.args[0])})"
```
- **EN**: It introduces or extends class-level abstractions such as `CppPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `CppPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 594-622 / 第 594-622 行
```python
    def _print_OpaqueUnaryFn_sinh(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("sinh expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"std::sinh({self._print(expr.args[0])})"

    def _print_OpaqueUnaryFn_asin(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("asin expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"std::asin({self._print(expr.args[0])})"

    def _print_OpaqueUnaryFn_tan(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("tan expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"std::tan({self._print(expr.args[0])})"

    def _print_OpaqueUnaryFn_tanh(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("tanh expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"std::tanh({self._print(expr.args[0])})"

    def _print_OpaqueUnaryFn_atan(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("atan expects exactly one argument")
        # pyrefly: ignore [missing-attribute]
        return f"std::atan({self._print(expr.args[0])})"
```
- **EN**: It introduces or extends class-level abstractions such as `CppPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `CppPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 624-654 / 第 624-654 行
```python
    def _print_OpaqueUnaryFn_sqrt(self, expr: sympy.Expr) -> str:
        # pyrefly: ignore [missing-attribute]
        return f"std::sqrt({self._print(expr.args[0])})"

    def _print_OpaqueUnaryFn_log2(self, expr: sympy.Expr) -> str:
        # pyrefly: ignore [missing-attribute]
        return f"std::log2({self._print(expr.args[0])})"

    def _print_RoundToInt(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 1:
            raise AssertionError("RoundToInt expects exactly one argument")
        # TODO: dispatch to llrint depending on index type
        # pyrefly: ignore [missing-attribute]
        return f"std::lrint({self._print(expr.args[0])})"

    def _print_RoundDecimal(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 2:
            raise AssertionError("RoundDecimal expects exactly two arguments")
        number, ndigits = expr.args
        if number.is_integer:
            # ndigits < 0 should have been filtered by the sympy function
            if ndigits >= 0:
                raise AssertionError("ndigits must be negative for integer inputs")
            raise ValueError(
                f"For integer inputs, only non-negative ndigits are currently supported, but got {ndigits}."
            )
        number_str = self.parenthesize(number, PRECEDENCE["Mul"])
        return f"static_cast<double>(std::nearbyint(1e{ndigits} * {number_str}) * 1e{-ndigits})"

    def _print_BooleanTrue(self, expr: sympy.Expr) -> str:
        return "true"
```
- **EN**: It introduces or extends class-level abstractions such as `CppPrinter`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `CppPrinter` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 656-666 / 第 656-666 行
```python
    def _print_BooleanFalse(self, expr: sympy.Expr) -> str:
        return "false"

    def _print_Infinity(self, expr: sympy.Expr) -> str:
        return "std::numeric_limits<double>::infinity()"

    def _print_NegativeInfinity(self, expr: sympy.Expr) -> str:
        return f"-{self._print_Infinity(expr)}"

    def _print_NaN(self, expr: sympy.Expr) -> str:
        return "std::numeric_limits<double>::quiet_NaN()"
```
- **EN**: It introduces or extends class-level abstractions such as `CppPrinter`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `CppPrinter` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements utility logic that supports broader PyTorch execution, packaging, or developer workflows.
  - CN: 实现支撑更广泛 PyTorch 执行、打包或开发者工作流的辅助逻辑。
- **Configuration management**
  - EN: Centralizes feature flags, environment overrides, and runtime-config wiring.
  - CN: 集中管理特性开关、环境变量覆盖以及运行时配置接线。
- **TensorBoard integration**
  - EN: Exports model, scalar, and graph information into TensorBoard-consumable formats.
  - CN: 把模型、标量与图信息导出为 TensorBoard 可消费的格式。
- **Serialization safety**
  - EN: Keeps persisted data exchange bounded, predictable, and easier to validate.
  - CN: 让持久化数据交换保持有界、可预测且更易验证。
- **ExprPrinter**
  - EN: `ExprPrinter` is one of the main classes that structures the file's behavior.
  - CN: `ExprPrinter` 是组织该文件行为的核心类之一。
- **PythonPrinter**
  - EN: `PythonPrinter` is one of the main classes that structures the file's behavior.
  - CN: `PythonPrinter` 是组织该文件行为的核心类之一。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `sys`
- **Third-party packages / 第三方包**: `sympy`, `sympy.printing.precedence:PRECEDENCE`, `sympy.printing.precedence:precedence`, `sympy.printing.str:StrPrinter`
- **Primary symbols / 核心符号**: `ExprPrinter`, `PythonPrinter`, `CppPrinter`
