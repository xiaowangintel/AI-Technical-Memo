# mps.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/mps.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `MetalExprPrinter`, `MetalOverrides`, `MetalKernel`, and `MetalScheduling`. It exposes functions such as `value_to_metal`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `MetalExprPrinter`、`MetalOverrides`、`MetalKernel`、`MetalScheduling` 等类。同时提供 `value_to_metal` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
# This is not a feature-complete compiler backend
# Just an early prototype that shows that one can compile elementwise ops into a Metal shader
from __future__ import annotations

import functools
import itertools
import logging
import math
from pathlib import Path
from typing import Any, TYPE_CHECKING

import sympy
from sympy.printing.precedence import PRECEDENCE

import torch
from torch.utils._cpp_embed_headers import _embed_headers
from torch.utils._ordered_set import OrderedSet
from torch.utils._sympy.printers import CppPrinter, ExprPrinter as ExprPrinter_
from torch.utils._sympy.value_ranges import ValueRanges

from ..utils import ceildiv, get_bounds_index_expr, get_kernel_metadata
from ..virtualized import ops, OpsWrapper, V
from .common import (
    CSEVariable,
    DeferredLine,
    DTYPE_TO_COMPUTATION_DTYPE,
    IndentedBuffer,
    OpOverrides,
````
- **EN**: Imports dependencies such as `__future__`, `functools`, `itertools`, `logging`, `math`, `pathlib`, and `...+11` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里导入了 `__future__`、`functools`、`itertools`、`logging`、`math`、`pathlib`、`另有11项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。借助符号表达式来推理索引、形状或代数性质。

### Lines 29-56 / 第 29-56 行
````python
    PythonPrinter,
)
from .simd import IterationRangesEntry, SIMDKernel, SIMDScheduling


if TYPE_CHECKING:
    from ..ops_handler import ReductionType, StoreMode
    from ..scheduler import Scheduler, SchedulerNode
    from .common import OpVarT

log = logging.getLogger(__name__)

DTYPE_TO_METAL = {
    torch.bool: "bool",
    torch.int8: "char",
    torch.int16: "short",
    torch.int32: "int",
    torch.int64: "long",
    torch.uint8: "uchar",
    torch.float: "float",
    torch.half: "half",
    torch.bfloat16: "bfloat",
}


def value_to_metal(val: float | int | bool | str | CSEVariable) -> str:
    if isinstance(val, float):
        if val == torch.inf:
````
- **EN**: Imports dependencies such as `.simd`, `..ops_handler`, `..scheduler`, and `.common` for the logic in this range. Introduces function `value_to_metal`. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里导入了 `.simd`、`..ops_handler`、`..scheduler`、`.common` 等依赖，为后续逻辑提供基础能力。这里定义了函数`value_to_metal`。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 57-84 / 第 57-84 行
````python
            return "HUGE_VALF"
        elif val == -torch.inf:
            return "-HUGE_VALF"
        elif val != val:  # Only float that not equal to self is nan
            return "NAN"
        return str(val)
    elif isinstance(val, bool):
        return "true" if val else "false"
    return str(val)


class MetalExprPrinter(ExprPrinter_):
    """Converts sympy expression to Metal code snippet"""

    def _print_FloorDiv(self, expr: sympy.Expr) -> str:
        x, div = expr.args
        x = self.doprint(x)
        div = self.doprint(div)
        if expr.is_integer:
            return f"c10::metal::floor_divide({x}, {div})"
        return f"metal::floor({x}) / ({div})"

    def _print_ModularIndexing(self, expr: sympy.Expr) -> str:
        x, div, mod = expr.args
        # Workaround for Metal compiler bug with fused (x / A) % B, see PR 175481
        use_safe_mod = div == 65536 and (mod & (mod - 1)) != 0

        x = self.doprint(x)
````
- **EN**: Introduces class `MetalExprPrinter`, function `_print_FloorDiv`, function `_print_ModularIndexing`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`MetalExprPrinter`、函数`_print_FloorDiv`、函数`_print_ModularIndexing`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 85-112 / 第 85-112 行
````python
        if div != 1:
            div = self.doprint(div)
            if expr.is_integer:
                x = f"({x}) / ({div})"
            else:
                x = f"metal::floor({x}) / ({div})"
        mod = self.doprint(mod)
        if use_safe_mod:
            return f"c10::metal::safe_mod({x}, {mod})"
        return f"({x}) % ({mod})"

    def _print_Min(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 2:
            raise RuntimeError("metal::min only supported for 2 args")
        # pyrefly: ignore [missing-attribute]
        a, b = map(self._print, expr.args)
        typecast_a = f"static_cast<decltype({a}+{b})>({a})"
        typecast_b = f"static_cast<decltype({a}+{b})>({b})"
        return f"metal::min({typecast_a}, {typecast_b})"

    def _print_Max(self, expr: sympy.Expr) -> str:
        if len(expr.args) != 2:
            raise RuntimeError("metal::max only supported for 2 args")
        # pyrefly: ignore [missing-attribute]
        a, b = map(self._print, expr.args)
        typecast_a = f"static_cast<decltype({a}+{b})>({a})"
        typecast_b = f"static_cast<decltype({a}+{b})>({b})"
        return f"metal::max({typecast_a}, {typecast_b})"
````
- **EN**: Introduces function `_print_Min`, function `_print_Max`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_print_Min`、函数`_print_Max`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 113-140 / 第 113-140 行
````python

    def _print_Abs(self, expr: sympy.Expr) -> str:
        assert len(expr.args) == 1
        # pyrefly: ignore [missing-attribute]
        return f"metal::abs({self._print(expr.args[0])})"

    def _print_RoundToInt(self, expr: sympy.Expr) -> str:
        assert len(expr.args) == 1
        # pyrefly: ignore [missing-attribute]
        return f"static_cast<long>(metal::rint({self._print(expr.args[0])}))"

    def _print_RoundDecimal(self, expr: sympy.Expr) -> str:
        assert len(expr.args) == 2
        number, ndigits = expr.args
        if number.is_integer:
            # ndigits < 0 should have been filtered by the sympy function
            assert ndigits < 0
            raise ValueError(
                f"For integer inputs, only non-negative ndigits are currently supported, but got {ndigits}."
            )
        number_str = self.parenthesize(number, PRECEDENCE["Mul"])
        return f"static_cast<float>(metal::rint(1e{ndigits} * {number_str}) * 1e{-ndigits})"

    def _print_IntTrueDiv(self, expr: sympy.Expr) -> str:
        lhs, rhs = expr.args
        # TODO: This is only accurate up to 2**23
        # pyrefly: ignore [missing-attribute]
        return f"static_cast<float>({self._print(lhs)}) / static_cast<float>({self._print(rhs)})"
````
- **EN**: Introduces function `_print_Abs`, function `_print_RoundToInt`, function `_print_RoundDecimal`, function `_print_IntTrueDiv`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_print_Abs`、函数`_print_RoundToInt`、函数`_print_RoundDecimal`、函数`_print_IntTrueDiv`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-168 / 第 141-168 行
````python

    def _print_PowByNatural(self, expr: sympy.Expr) -> str:
        assert len(expr.args) == 2
        x, y = map(self.doprint, expr.args)
        return f"metal::precise::pow(static_cast<float>({x}), static_cast<float>({y}))"

    def _print_ToFloat(self, expr: sympy.Expr) -> str:
        assert len(expr.args) == 1
        x = self.doprint(expr.args[0])
        return f"static_cast<float>({x})"

    def _print_Float(self, expr: sympy.Expr) -> str:
        if expr.is_integer:
            # sympy considers 0.0 to be integer, but Metal doesn't.
            # this workaround prints the float as an integer
            # xref: https://github.com/sympy/sympy/issues/26620
            return str(int(expr))
        else:
            return str(expr)

    def _print_FloorToInt(self, expr: sympy.Expr) -> str:
        assert len(expr.args) == 1
        x = self.doprint(expr.args[0])
        return f"static_cast<int>(metal::floor(static_cast<float>({x})))"

    _print_floor = _print_FloorToInt

    def _print_TruncToInt(self, expr: sympy.Expr) -> str:
````
- **EN**: Introduces function `_print_PowByNatural`, function `_print_ToFloat`, function `_print_Float`, function `_print_FloorToInt`, function `_print_TruncToInt`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_print_PowByNatural`、函数`_print_ToFloat`、函数`_print_Float`、函数`_print_FloorToInt`、函数`_print_TruncToInt`。保留了指向设计说明、规范或厂商数据手册的注释引用。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 169-196 / 第 169-196 行
````python
        assert len(expr.args) == 1
        x = self.doprint(expr.args[0])
        return f"static_cast<int>(metal::trunc({x}))"

    def _print_OpaqueUnaryFn_log2(self, expr: sympy.Expr) -> str:
        assert len(expr.args) == 1
        x = self.doprint(expr.args[0])
        return f"metal::precise::log2({x})"

    def _print_Where(self, expr: sympy.Expr) -> str:
        c, p, q = (
            self.parenthesize(arg, PRECEDENCE["Atom"] - 0.5) for arg in expr.args
        )
        return f"{c} ? {p} : {q}"


class MetalOverrides(OpOverrides):
    """Implements Metal-specific overrides for ops. Base class emits Python-friendly overrides."""

    @staticmethod
    def to_dtype(
        x: CSEVariable,
        dtype: torch.dtype,
        src_dtype: torch.dtype | None = None,
        use_compute_types: bool = True,
    ) -> str:
        if dtype == torch.double:
            log.warning(
````
- **EN**: Introduces function `_print_OpaqueUnaryFn_log2`, function `_print_Where`, class `MetalOverrides`, function `to_dtype`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`_print_OpaqueUnaryFn_log2`、函数`_print_Where`、类`MetalOverrides`、函数`to_dtype`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 197-224 / 第 197-224 行
````python
                "float64 cast requested, probably from tensorify_python_scalars"
            )
            return f"static_cast<float>({x})"
        return f"static_cast<{DTYPE_TO_METAL[dtype]}>({x})"

    @staticmethod
    def to_dtype_bitcast(
        x: CSEVariable, dtype: torch.dtype, src_dtype: torch.dtype
    ) -> str:
        return f"as_type<{DTYPE_TO_METAL[dtype]}>(static_cast<{DTYPE_TO_METAL[src_dtype]}>({x}))"

    @staticmethod
    def constant(val: bool | float | int, dtype: torch.dtype) -> str:
        return value_to_metal(val)

    @staticmethod
    def index_expr(expr: sympy.Expr, dtype: torch.dtype) -> str:
        idx_str = V.kernel.index_to_str(V.kernel.prepare_indexing(expr))
        var = V.kernel.cse.generate(
            V.kernel.compute, idx_str, bounds=get_bounds_index_expr(expr)
        )
        return ops.to_dtype(var, dtype)

    @staticmethod
    def masked(mask: CSEVariable, body: sympy.Expr, other: CSEVariable) -> str:
        # TODO: Type annotation for other is wrong, it's often float or int
        # TODO: Should it be converted to lambda on MacOS-15+?

````
- **EN**: Introduces function `to_dtype_bitcast`, function `constant`, function `index_expr`, function `masked`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`to_dtype_bitcast`、函数`constant`、函数`index_expr`、函数`masked`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 225-252 / 第 225-252 行
````python
        other_str = value_to_metal(other)
        scoped_body = IndentedBuffer()
        with V.kernel.swap_buffers(scoped_body), scoped_body.indent():
            # Reset the scoped variable counter so that each invocation of the same body
            # generates identical variable names. Without this reset, repeated calls to
            # body() would keep incrementing the counter, resulting in different cache key.
            V.kernel.cse.iter_buffer_ids = itertools.count()
            # Append "_scoped" to the current prefix so each nesting level gets unique vars
            V.kernel.cse.name_prefix += "_scoped"
            rc = body()

        # Compute cache key manually as variable name is needed to actually generate the code
        cache_key = f"{mask}:{scoped_body.getvalue()}:{other_str}"
        var = V.kernel.cse.try_get(cache_key)
        if not var:
            var = V.kernel.cse.newvar(dtype=rc.dtype)
            V.kernel.cse.put(cache_key, var)
            V.kernel.compute.writelines(
                [f"{DTYPE_TO_METAL[rc.dtype]} {var};", f"if ({mask}) {{"]
            )
            with V.kernel.compute.indent():
                V.kernel.compute.splice(scoped_body)
                V.kernel.compute.writeline(
                    f"{var} = static_cast<decltype({var})>({rc});"
                )
            V.kernel.compute.writeline(
                f"}} else {var} = static_cast<decltype({var})>({other_str});"
            )
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `other_str`, `scoped_body`, `rc`, `cache_key`, and `var`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `other_str`、`scoped_body`、`rc`、`cache_key`、`var` 等值。

### Lines 253-280 / 第 253-280 行
````python
        return var

    @staticmethod
    # pyrefly: ignore [bad-override]
    def where(a: OpVarT, b: OpVarT, c: OpVarT) -> str:
        return f"{a} ? {b} : static_cast<decltype({b})>({value_to_metal(c)})"

    @staticmethod
    def remainder(a: OpVarT, b: OpVarT) -> str:
        return f"c10::metal::remainder({a}, {b})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def maximum(a: CSEVariable, b: CSEVariable) -> str:
        typecast_a = f"static_cast<decltype({a}+{b})>({a})"
        typecast_b = f"static_cast<decltype({a}+{b})>({b})"
        return f"c10::metal::max({typecast_a}, {typecast_b})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def minimum(a: CSEVariable, b: CSEVariable) -> str:
        typecast_a = f"static_cast<decltype({a}+{b})>({a})"
        typecast_b = f"static_cast<decltype({a}+{b})>({b})"
        return f"c10::metal::min({typecast_a}, {typecast_b})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def logical_or(a: CSEVariable, b: CSEVariable) -> str:
````
- **EN**: Introduces function `where`, function `remainder`, function `maximum`, function `minimum`, function `logical_or`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `typecast_a`, and `typecast_b`.
- **CN**: 这里定义了函数`where`、函数`remainder`、函数`maximum`、函数`minimum`、函数`logical_or`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `typecast_a`、`typecast_b` 等值。

### Lines 281-308 / 第 281-308 行
````python
        return f"{a} || {b}"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def logical_and(a: CSEVariable, b: CSEVariable) -> str:
        return f"{a} && {b}"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def isnan(x: CSEVariable) -> str:
        return f"metal::isnan({x})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def isinf(x: CSEVariable) -> str:
        return f"metal::isinf({x})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def log(x: CSEVariable) -> str:
        return f"metal::precise::log({x})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def exp(x: CSEVariable) -> str:
        return f"metal::precise::exp({x})"

    @staticmethod
````
- **EN**: Introduces function `logical_and`, function `isnan`, function `isinf`, function `log`, function `exp`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`logical_and`、函数`isnan`、函数`isinf`、函数`log`、函数`exp`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 309-336 / 第 309-336 行
````python
    # pyrefly: ignore [bad-override]
    def abs(x: CSEVariable) -> str:
        return f"metal::abs({x})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def signbit(x: CSEVariable) -> str:
        return f"metal::signbit({x})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def sin(x: CSEVariable) -> str:
        return f"metal::precise::sin({x})"

    @staticmethod
    def sinc(x: CSEVariable) -> str:
        return f"c10::metal::sinc({x})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def cos(x: CSEVariable) -> str:
        return f"metal::precise::cos({x})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def tan(x: CSEVariable) -> str:
        return f"metal::precise::tan({x})"

````
- **EN**: Introduces function `abs`, function `signbit`, function `sin`, function `sinc`, function `cos`, function `tan`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`abs`、函数`signbit`、函数`sin`、函数`sinc`、函数`cos`、函数`tan`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 337-364 / 第 337-364 行
````python
    @staticmethod
    # pyrefly: ignore [bad-override]
    def asin(x: CSEVariable) -> str:
        return f"metal::precise::asin({x})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def acos(x: CSEVariable) -> str:
        return f"metal::precise::acos({x})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def atan(x: CSEVariable) -> str:
        return f"metal::precise::atan({x})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def atan2(x: CSEVariable, y: CSEVariable) -> str:
        return f"::metal::precise::atan2({x}, {y})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def sqrt(x: CSEVariable) -> str:
        return f"metal::precise::sqrt({x})"

    @staticmethod
    def neg(x: CSEVariable) -> str:
        # TODO: Does it rely on undefined behavior?
````
- **EN**: Introduces function `asin`, function `acos`, function `atan`, function `atan2`, function `sqrt`, function `neg`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`asin`、函数`acos`、函数`atan`、函数`atan2`、函数`sqrt`、函数`neg`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 365-392 / 第 365-392 行
````python
        # If so, add special logic for unsigned types
        return f"static_cast<decltype({x})>(-{x})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def rsqrt(x: CSEVariable) -> str:
        return f"metal::precise::rsqrt({x})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def tanh(x: CSEVariable) -> str:
        return f"metal::precise::tanh({x})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def atanh(x: CSEVariable) -> str:
        return f"metal::precise::atanh({x})"

    @staticmethod
    def floordiv(a: CSEVariable, b: CSEVariable) -> str:
        # a and b must be of integer type
        return f"c10::metal::floor_divide({a}, {b})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def floor(x: CSEVariable) -> str:
        return f"metal::floor({x})"

````
- **EN**: Introduces function `rsqrt`, function `tanh`, function `atanh`, function `floordiv`, function `floor`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`rsqrt`、函数`tanh`、函数`atanh`、函数`floordiv`、函数`floor`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 393-420 / 第 393-420 行
````python
    @staticmethod
    # pyrefly: ignore [bad-override]
    def sign(x: CSEVariable) -> str:
        return f"metal::sign({x})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def fmod(a: CSEVariable, b: CSEVariable) -> str:
        typecast_a = f"static_cast<decltype({a}+{b})>({a})"
        typecast_b = f"static_cast<decltype({a}+{b})>({b})"
        return f"metal::fmod({typecast_a}, {typecast_b})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def trunc(x: CSEVariable) -> str:
        return f"metal::trunc({x})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def truncdiv(a: CSEVariable, b: CSEVariable) -> str:
        quot = f"{a} / {b}"
        if (a.dtype is not None and a.dtype.is_floating_point) or (
            b.dtype is not None and b.dtype.is_floating_point
        ):
            return f"metal::trunc({quot})"
        return quot

    @staticmethod
````
- **EN**: Introduces function `sign`, function `fmod`, function `trunc`, function `truncdiv`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`sign`、函数`fmod`、函数`trunc`、函数`truncdiv`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 421-448 / 第 421-448 行
````python
    # pyrefly: ignore [bad-override]
    def ceil(x: CSEVariable) -> str:
        return f"metal::ceil({x})"

    @staticmethod
    def rand(seed: CSEVariable, offset: CSEVariable) -> str:
        V.kernel.headers.add("random")
        return f"c10::metal::rand({seed}, {offset})"

    @staticmethod
    def randn(seed: CSEVariable, offset: CSEVariable) -> str:
        V.kernel.headers.add("random")
        return f"c10::metal::randn({seed}, {offset})"

    @staticmethod
    def randint64(
        seed: CSEVariable, offset: CSEVariable, low: CSEVariable, high: CSEVariable
    ) -> str:
        V.kernel.headers.add("random")
        return f"c10::metal::randint64({seed}, {offset}, {low}, {high})"

    @staticmethod
    # pyrefly: ignore [bad-override]
    def round(x: CSEVariable) -> str:
        return f"metal::rint({x})"

    @staticmethod
    def pow(a: CSEVariable, b: CSEVariable) -> str:
````
- **EN**: Introduces function `ceil`, function `rand`, function `randn`, function `randint64`, function `round`, function `pow`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `seed`.
- **CN**: 这里定义了函数`ceil`、函数`rand`、函数`randn`、函数`randint64`、函数`round`、函数`pow`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `seed` 等值。

### Lines 449-476 / 第 449-476 行
````python
        cast_a = f"static_cast<decltype({a}+{b})>({a})"
        cast_b = f"static_cast<decltype({a}+{b})>({b})"
        return f"metal::pow({cast_a}, {cast_b})"

    def _special_unary(self, a: CSEVariable, name: str) -> str:
        V.kernel.headers.add("special_math")
        return f"c10::metal::{name}({a})"

    def _special_binary(self, a: CSEVariable, b: CSEVariable, name: str) -> str:
        V.kernel.headers.add("special_math")
        return f"c10::metal::{name}({a}, {b})"

    @classmethod
    def _initialize_special_ops(cls) -> None:
        # Unary special ops
        for name in [
            "erf",
            "erfinv",
            "i0",
            "i0e",
            "i1",
            "i1e",
            "digamma",
            "spherical_bessel_j0",
        ]:
            setattr(cls, name, functools.partialmethod(cls._special_unary, name=name))

        cls.lgamma = functools.partialmethod(cls._special_unary, name="log_gamma")  # type: ignore[assignment]
````
- **EN**: Introduces function `_special_unary`, function `_special_binary`, function `_initialize_special_ops`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_special_unary`、函数`_special_binary`、函数`_initialize_special_ops`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 477-504 / 第 477-504 行
````python

        # Unary special ops with forward in method name
        for name in [
            "bessel_j0",
            "bessel_j1",
            "bessel_y0",
            "bessel_y1",
            "modified_bessel_i0",
            "modified_bessel_i1",
            "modified_bessel_k0",
            "modified_bessel_k1",
            "scaled_modified_bessel_k0",
            "scaled_modified_bessel_k1",
        ]:
            setattr(
                cls,
                name,
                functools.partialmethod(cls._special_unary, name=name + "_forward"),
            )

        # Binary special ops
        for name in [
            "polygamma",
            "igamma",
            "igammac",
            "zeta",
        ]:
            setattr(cls, name, functools.partialmethod(cls._special_binary, name=name))
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `MetalOverrides._initialize_special_ops`.
- **CN**: 包含分支、循环或上下文管理等控制流。这一段延续了函数`MetalOverrides._initialize_special_ops` 的具体实现。

### Lines 505-532 / 第 505-532 行
````python

        # Binary special ops with forward in method name
        for name in [
            "chebyshev_polynomial_t",
            "chebyshev_polynomial_u",
            "chebyshev_polynomial_v",
            "chebyshev_polynomial_w",
            "hermite_polynomial_h",
            "hermite_polynomial_he",
            "shifted_chebyshev_polynomial_t",
            "shifted_chebyshev_polynomial_u",
            "shifted_chebyshev_polynomial_v",
            "shifted_chebyshev_polynomial_w",
        ]:
            setattr(
                cls,
                name,
                functools.partialmethod(cls._special_binary, name=name + "_forward"),
            )


MetalOverrides._initialize_pointwise_overrides("mps")
MetalOverrides._initialize_special_ops()


class MetalKernel(SIMDKernel):
    """Implement Metal codegen based on the SIMDKernel abstraction"""

````
- **EN**: Introduces class `MetalKernel`. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`MetalKernel`。包含分支、循环或上下文管理等控制流。

### Lines 533-560 / 第 533-560 行
````python
    overrides = MetalOverrides  # type: ignore[assignment]
    suffix = ";"
    newvar_prefix = "auto "
    max_threadgroup_size = 1024
    simd_group_size = 32
    pexpr = PythonPrinter().doprint
    cexpr = CppPrinter().doprint
    sexpr = MetalExprPrinter().doprint
    kexpr = sexpr
    headers: OrderedSet[str] = OrderedSet(["utils"])
    multistage_reduction_entry: list[IterationRangesEntry] = []

    def __init__(
        self,
        tiling: dict[str, sympy.Expr],
        **kwargs: Any,
    ) -> None:
        super().__init__(tiling, **kwargs)
        self.acc_var_ids = itertools.count()

    def dtype_to_str(self, dtype: torch.dtype) -> str:
        return DTYPE_TO_METAL[dtype]

    def load(self, name: str, index: sympy.Expr) -> CSEVariable:
        """Codegen a load from an InputBuffer"""
        var = self.args.input(name)
        index = self.prepare_indexing(index)
        dtype = V.graph.get_dtype(name)
````
- **EN**: Introduces function `__init__`, function `dtype_to_str`, function `load`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`__init__`、函数`dtype_to_str`、函数`load`。借助符号表达式来推理索引、形状或代数性质。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 561-588 / 第 561-588 行
````python
        line = f"{var}[{self.index_to_str(index)}]"
        if dtype in [torch.float16, torch.bfloat16]:
            # TODO(NS): Figure out the right balance between optype casts
            # op_math_t for half-precision floats should be float32
            # Otherwise it can lead to a correctness issues with eager
            line = f"static_cast<float>({line})"
            dtype = torch.float32
        return self.cse.generate(self.loads, line, dtype=dtype)

    def store(
        self, name: str, index: sympy.Expr, value: CSEVariable, mode: StoreMode = None
    ) -> None:
        var = self.args.output(name)
        index = self.prepare_indexing(index)
        dtype_str = self.dtype_to_str(V.graph.get_dtype(name))
        cast_val = f"static_cast<{dtype_str}>({value})"
        if mode is None:
            line = f"{var}[{self.index_to_str(index)}] = {cast_val};"
        elif mode == "atomic_add":
            self.headers.add("atomic")
            atomic_type = f"c10::metal::AtomicType<{dtype_str}>"
            cast_var = f"reinterpret_cast<device {atomic_type}::type *>({var})"
            line = f"{atomic_type}::atomic_add({cast_var}, {self.index_to_str(index)}, {cast_val});"
        else:
            raise RuntimeError(f"Unimplemented store mode {mode}")
        if self.inside_reduction:
            self.compute.writeline(DeferredLine(name, line))
        else:
````
- **EN**: Introduces function `store`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`store`。借助符号表达式来推理索引、形状或代数性质。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 589-616 / 第 589-616 行
````python
            self.stores.writeline(DeferredLine(name, line))

    def store_reduction(self, name: str, index: sympy.Expr, value: CSEVariable) -> None:
        var = self.args.output(name)
        index = self.prepare_indexing(index)
        dtype_str = self.dtype_to_str(V.graph.get_dtype(name))

        reduction_dim = next(t for t in self.range_trees if t.is_reduction)
        # Only one thread in the reduction group needs to store the results
        line = f"{var}[{self.index_to_str(index)}] = static_cast<{dtype_str}>({value});"
        line = f"if ({reduction_dim.name} == 0) {line}"
        self.stores.writeline(DeferredLine(name, line))

    def _new_idxvar(
        self,
        dtype: str | torch.dtype,
        elem_count: int | None = None,
        default_value: Any | None = None,
        is_threadgroup: bool = True,
        bounds: ValueRanges[Any] = ValueRanges.unknown(),
    ) -> CSEVariable:
        if isinstance(dtype, torch.dtype):
            dtype = self.dtype_to_str(dtype)
        var_name = f"tmp_acc_{next(self.acc_var_ids)}"
        var = V.kernel.create_cse_var(var_name, bounds, dtype)
        var_def = "threadgroup " if is_threadgroup else ""
        var_def += f"{dtype} {var_name}"
        if elem_count:
````
- **EN**: Introduces function `store_reduction`, function `_new_idxvar`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`store_reduction`、函数`_new_idxvar`。借助符号表达式来推理索引、形状或代数性质。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。

### Lines 617-644 / 第 617-644 行
````python
            var_def += f"[{self.sexpr(elem_count)}]"
        if default_value is not None:
            assert not is_threadgroup, "Thread group var can not have default value"
            var_def += f" = {default_value}"
        self.indexing_code.writeline(var_def + self.suffix)
        return var

    def reduction(
        self,
        dtype: torch.dtype,
        src_dtype: torch.dtype,
        reduction_type: ReductionType,
        value: CSEVariable | tuple[CSEVariable, ...],
    ) -> CSEVariable | tuple[CSEVariable, ...]:
        "Caching wrapper around _reduction_nocache"
        cache_key = (src_dtype, reduction_type, value)
        # Return cached reduction
        if cache_key in self.cse.reduction_cache:
            return self.cse.reduction_cache[cache_key]
        result = self._reduction_nocache(dtype, src_dtype, reduction_type, value)
        self.cse.reduction_cache[cache_key] = result  # type: ignore[assignment]
        return result

    def _reduction_nocache(
        self,
        dtype: torch.dtype,
        src_dtype: torch.dtype,
        reduction_type: ReductionType,
````
- **EN**: Introduces function `reduction`, function `_reduction_nocache`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`reduction`、函数`_reduction_nocache`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 645-672 / 第 645-672 行
````python
        value: CSEVariable | tuple[CSEVariable, ...],
    ) -> CSEVariable | tuple[CSEVariable, ...]:
        """Codegen a reduction operation.
        Only sum and prod operations are somewhat reasonable optimized"""
        assert self.inside_reduction
        assert not self._load_mask

        def _unwrap_helper(res3: CSEVariable) -> tuple[CSEVariable, ...]:
            # Uwraps vec3 dtype into individual components
            return OpsWrapper._unwrap(
                [CSEVariable(f"{res3}.{t}", res3.bounds, res3.dtype) for t in "xyz"]
            )

        # Establish reduction buffer size and index expression
        reduction_idx = ""
        acc_buf_size = 1
        for rd in self.range_trees:
            if not rd.is_reduction:
                continue
            if reduction_idx:
                reduction_idx += " + "
            reduction_idx += f"{rd.name} * {acc_buf_size}"

            if isinstance(rd.numel, sympy.Integer):
                acc_buf_size *= rd.numel
            else:
                acc_buf_size *= sympy.Symbol(
                    f"{rd.prefix}numel", integer=True, positive=True
````
- **EN**: Introduces function `_unwrap_helper`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_unwrap_helper`。借助符号表达式来推理索引、形状或代数性质。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 673-700 / 第 673-700 行
````python
                )

        acc_buf_size = sympy.Min(acc_buf_size, self.max_threadgroup_size)
        acc_buf_size_str = self.sexpr(acc_buf_size)
        shmem_buf_size = (
            ceildiv(acc_buf_size, self.simd_group_size)
            if isinstance(acc_buf_size, sympy.Integer)
            else self.simd_group_size
        )

        if reduction_type == "any":
            acc = self._new_idxvar(dtype)
            self.indexing_code.writeline(f"{acc} = false;")
            self.indexing_code.writeline(
                "threadgroup_barrier(metal::mem_flags::mem_threadgroup);"
            )
            self.compute.splice(
                f"""
                if ({value}) {{
                    {acc} = true;
                }}
            """
            )
            self.stores.writeline(
                "threadgroup_barrier(metal::mem_flags::mem_threadgroup);"
            )
            return acc

````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 701-728 / 第 701-728 行
````python
        self.headers.add("reduction_utils")

        if reduction_type in ["prod", "sum"]:
            acc_dtype = DTYPE_TO_COMPUTATION_DTYPE[src_dtype]
            acc_buf = self._new_idxvar(acc_dtype, shmem_buf_size)
            if not self.multistage_reduction_entry:
                val = value
            else:
                default_val, reduction_op = (
                    (0, "+") if reduction_type == "sum" else (1, "*")
                )
                val = self._new_idxvar(
                    acc_dtype, default_value=default_val, is_threadgroup=False
                )
                self.compute.splice(f"{val} {reduction_op}= {value};")

            return self.cse.generate(
                self.stores,
                f"c10::metal::threadgroup_{reduction_type}({acc_buf}, {val}, {reduction_idx}, {acc_buf_size_str})",
                dtype=DTYPE_TO_COMPUTATION_DTYPE[dtype],
            )
        if reduction_type in ["max", "min"]:
            acc_buf = self._new_idxvar(src_dtype, shmem_buf_size)
            src_metal_type = DTYPE_TO_METAL[src_dtype]
            cast_value = f"static_cast<{src_metal_type}>({value})"
            if not self.multistage_reduction_entry:
                val = cast_value  # type: ignore[assignment]
            else:
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `acc_dtype`, `acc_buf`, `val`, `else`, `dtype`, `src_metal_type`, and `...+1`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `acc_dtype`、`acc_buf`、`val`、`else`、`dtype`、`src_metal_type`、`另有1项` 等值。

### Lines 729-756 / 第 729-756 行
````python
                lim_fn = "lowest" if reduction_type.endswith("max") else "max"
                limit_val = f"::metal::numeric_limits<{src_metal_type}>::{lim_fn}()"
                val = self._new_idxvar(
                    src_dtype, default_value=limit_val, is_threadgroup=False
                )
                self.compute.splice(
                    f"{val} = ::c10::metal::{reduction_type}({val}, {cast_value});"
                )
            return self.cse.generate(
                self.stores,
                f"c10::metal::threadgroup_{reduction_type}({acc_buf}, {val}, {reduction_idx}, {acc_buf_size_str})",
                dtype=DTYPE_TO_COMPUTATION_DTYPE[dtype],
            )
        if reduction_type in ["argmin", "argmax"]:
            data_acc_buf = self._new_idxvar(src_dtype, shmem_buf_size)
            idx_acc_buf = self._new_idxvar(dtype, shmem_buf_size)
            src_metal_type = DTYPE_TO_METAL[src_dtype]
            cast_value = f"static_cast<{src_metal_type}>({value})"
            if not self.multistage_reduction_entry:
                val = cast_value  # type: ignore[assignment]
                idx_val = f"static_cast<{DTYPE_TO_METAL[dtype]}>({reduction_idx})"
            else:
                lim_fn = "lowest" if reduction_type.endswith("max") else "max"
                limit_val = f"::metal::numeric_limits<{src_metal_type}>::{lim_fn}()"
                val = self._new_idxvar(
                    src_dtype, default_value=limit_val, is_threadgroup=False
                )
                idx_val = self._new_idxvar(dtype, default_value=0, is_threadgroup=False)  # type: ignore[assignment]
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `lim_fn`, `limit_val`, `val`, `dtype`, `data_acc_buf`, `idx_acc_buf`, and `...+4`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `lim_fn`、`limit_val`、`val`、`dtype`、`data_acc_buf`、`idx_acc_buf`、`另有4项` 等值。

### Lines 757-784 / 第 757-784 行
````python
                idx_var = next(
                    t for t in self.range_tree_nodes.values() if t.is_reduction
                )
                cmp_op = ">" if reduction_type == "argmax" else "<"
                nan_suffix = (
                    f" || ::metal::isnan({value}) "
                    if src_dtype.is_floating_point
                    else ""
                )
                self.compute.splice(f"""
                if ({value} {cmp_op} {val}{nan_suffix}) {{
                    {val} = {value};
                    {idx_val} = {idx_var.name};
                }}
                """)
            return self.cse.generate(
                self.stores,
                f"c10::metal::threadgroup_{reduction_type}({data_acc_buf}, {idx_acc_buf}, "
                f"{val}, {idx_val}, {reduction_idx}, {acc_buf_size_str})",
                dtype=dtype,
            )
        if reduction_type == "welford_reduce":
            if not self.multistage_reduction_entry:
                acc_buf = self._new_idxvar(src_dtype, acc_buf_size)
                self.compute.splice(f"{acc_buf}[{reduction_idx}] = {value};")
                wf_res = self.cse.generate(
                    self.compute,
                    f"c10::metal::threadgroup_{reduction_type}({acc_buf}, {acc_buf_size_str})",
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `idx_var`, `cmp_op`, `nan_suffix`, `dtype`, `acc_buf`, and `wf_res`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `idx_var`、`cmp_op`、`nan_suffix`、`dtype`、`acc_buf`、`wf_res` 等值。

### Lines 785-812 / 第 785-812 行
````python
                    dtype=torch.float32,
                )
                return _unwrap_helper(wf_res)
            acc_buf = self._new_idxvar("float3", acc_buf_size)
            acc_thread_var = f"{acc_buf}[{reduction_idx}]"
            self.indexing_code.splice(f"{acc_thread_var} = 0.0;")
            self.compute.writeline(
                f"{acc_thread_var} = ::c10::metal::welford_combine({acc_thread_var}, float3({value}, 0.0, 1.0));"
            )
            wf_res = self.cse.generate(
                self.stores,
                f"c10::metal::threadgroup_welford_combine({acc_buf}, {acc_buf_size})",
                dtype=torch.float32,
            )
            return _unwrap_helper(wf_res)
        if reduction_type == "welford_combine":
            assert isinstance(value, tuple), "Input to welford combine must be tuple"
            acc_buf = self._new_idxvar("float3", acc_buf_size)
            acc_thread_var = f"{acc_buf}[{reduction_idx}]"
            inp_value = f"float3({value[0]}, {value[1]}, {value[2]})"
            self.indexing_code.splice(f"{acc_thread_var} = 0.0;")
            if self.multistage_reduction_entry:
                self.indexing_code.splice(f"{acc_thread_var} = 0.0;")
                self.compute.writeline(
                    f"{acc_thread_var} = ::c10::metal::welford_combine({acc_thread_var}, {inp_value});"
                )
            else:
                self.compute.writeline(f"{acc_thread_var} = {inp_value};")
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dtype`, `acc_buf`, `acc_thread_var`, `wf_res`, `inp_value`, and `else`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `dtype`、`acc_buf`、`acc_thread_var`、`wf_res`、`inp_value`、`else` 等值。

### Lines 813-840 / 第 813-840 行
````python
            wf_res = self.cse.generate(
                self.stores if self.multistage_reduction_entry else self.compute,
                f"c10::metal::threadgroup_{reduction_type}({acc_buf}, {acc_buf_size_str})",
                dtype=torch.float32,
            )
            return _unwrap_helper(wf_res)
        raise NotImplementedError(reduction_type)

    def codegen_iteration_ranges_entry(self, entry: IterationRangesEntry) -> None:
        index_expr = self.rename_indexing(entry.expr)
        index_str = self.sexpr(index_expr)  # type: ignore[misc]

        if not entry.is_reduction or (
            isinstance(entry.root.numel, sympy.Integer)
            and entry.root.numel <= self.max_threadgroup_size
        ):
            self.indexing_code.writeline(
                f"{self.index_dtype} {entry.name} = {index_str};"
            )
            return

        acc_size = (
            entry.root.numel
            if isinstance(entry.root.numel, sympy.Integer)
            else sympy.Symbol(f"{entry.root.prefix}numel", integer=True, positive=True)
        )

        # Check if we've already generated a loop for this reduction root
````
- **EN**: Introduces function `codegen_iteration_ranges_entry`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`codegen_iteration_ranges_entry`。借助符号表达式来推理索引、形状或代数性质。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 841-868 / 第 841-868 行
````python
        root_already_processed = any(
            e.root is entry.root for e in self.multistage_reduction_entry
        )

        linear_idx_name = f"{entry.root.prefix}_linear_idx"

        if not root_already_processed:
            self.multistage_reduction_entry.append(entry)
            # When reducing the tensor whose size exceeds max threadgroup size
            # loop over extra indices per reduction thread and perform part of the operation
            # using values in the shared memory

            # Use floats so that it doesn't do integer division
            loop_size = (acc_size + float(self.max_threadgroup_size - 1)) // float(
                self.max_threadgroup_size
            )
            loop_size_str = self.sexpr(loop_size)

            root_name = entry.root.name

            self.body.writeline(
                f"for(auto {entry.root.prefix}_cnt = 0; {entry.root.prefix}_cnt < {loop_size_str}; ++{entry.root.prefix}_cnt) {{"
            )
            with self.body.indent():
                if isinstance(acc_size, sympy.Symbol):
                    self.body.writeline(
                        f"{self.index_dtype} {linear_idx_name} = "
                        f"{self.max_threadgroup_size} * {entry.root.prefix}_cnt + {root_name};"
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `root_already_processed`, `linear_idx_name`, `loop_size`, `loop_size_str`, and `root_name`. This range continues the implementation of function `MetalKernel.codegen_iteration_ranges_entry`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。初始化或更新了 `root_already_processed`、`linear_idx_name`、`loop_size`、`loop_size_str`、`root_name` 等值。这一段延续了函数`MetalKernel.codegen_iteration_ranges_entry` 的具体实现。

### Lines 869-896 / 第 869-896 行
````python
                    )
                else:
                    self.body.writeline(
                        f"{self.index_dtype} {linear_idx_name} = {loop_size_str} * {root_name} + {entry.root.prefix}_cnt;"
                    )

                # Check that reduction is performed only within tensor boundary
                if (
                    isinstance(acc_size, sympy.Symbol)
                    or loop_size * self.max_threadgroup_size != acc_size
                ):
                    self.body.writeline(f"if ({linear_idx_name} >= {acc_size}) break;")

                # Compute entry value from linear index by substituting root name
                sub_index_str = index_str.replace(entry.root.name, linear_idx_name)
                self.body.writeline(
                    f"{self.index_dtype} {entry.name} = {sub_index_str};"
                )
        else:
            # root is already processed so just need to compute this entry's value inside the existing loop
            with self.body.indent():
                sub_index_str = index_str.replace(entry.root.name, linear_idx_name)
                self.body.writeline(
                    f"{self.index_dtype} {entry.name} = {sub_index_str};"
                )

    def codegen_body(self) -> None:
        """
````
- **EN**: Introduces function `codegen_body`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, and `sub_index_str`.
- **CN**: 这里定义了函数`codegen_body`。借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`sub_index_str` 等值。

### Lines 897-924 / 第 897-924 行
````python
        Concat output code from index_code, loads, compute, stores,
        suffix into self.body.

        For pointwise kernels, this is called just once at the end.

        For reduction kernels, this generates a loop over the reduction
        axis.
        """
        if self.multistage_reduction_entry:
            with self.body.indent():
                self.body.splice(self.loads)
                self.body.splice(self.compute)
            self.body.writeline("}" * len(self.multistage_reduction_entry))
            # Invalidate variables instantiated inside loop
            # But results of reduction alive. Reduction cache values can be
            # either CSEVariable or tuple of CSEVariables, in which case all
            # variables in the tuple must be preserved
            self.cse.invalidate(
                OrderedSet(
                    v
                    for item in self.cse.reduction_cache.values()
                    for v in (item if isinstance(item, tuple) else (item,))
                )
            )
            # And loop codegen
            while self.multistage_reduction_entry:
                self.multistage_reduction_entry.pop().cache_clear()
        else:
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`. This range continues the implementation of function `MetalKernel.codegen_body`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。这一段延续了函数`MetalKernel.codegen_body` 的具体实现。

### Lines 925-952 / 第 925-952 行
````python
            self.body.splice(self.loads)
            self.body.splice(self.compute)
        self.body.splice(self.stores)
        self.loads.clear()
        self.compute.clear()
        self.stores.clear()

    def codegen_kernel(self, name: str = "generated_kernel") -> str:
        """Called at the end to generate a final kernel string"""
        self.codegen_body()
        code = IndentedBuffer()
        fn_name = name

        if V.graph.cpp_wrapper:
            code.writeline('(R"MTL(')
        else:
            code.writeline("compile_mps_shader('''")

        idx_vars = self.active_range_trees()
        with code.indent():
            if not V.graph.cpp_wrapper:
                for header in self.headers:
                    code.writeline(f"#include <c10/metal/{header}.h>")
            else:
                headers = [
                    f"#include <c10/metal/{header}.h>" for header in self.headers
                ]
                header_contents = _embed_headers(
````
- **EN**: Introduces function `codegen_kernel`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `code`, `fn_name`, `else`, `idx_vars`, `headers`, and `header_contents`.
- **CN**: 这里定义了函数`codegen_kernel`。包含分支、循环或上下文管理等控制流。初始化或更新了 `code`、`fn_name`、`else`、`idx_vars`、`headers`、`header_contents` 等值。

### Lines 953-980 / 第 953-980 行
````python
                    headers,
                    [Path(__file__).parent.parent.parent / "include"],
                    OrderedSet(),  # type: ignore[arg-type]
                )
                code.writeline(header_contents)

            if self.inside_reduction:
                total_reduction_size = math.prod(
                    t.numel for t in self.range_trees if t.is_reduction
                )
                # If using dynamic shapes, set the threadgroup size to be the
                # max possible size
                threadgroup_size = (
                    min(total_reduction_size, self.max_threadgroup_size)
                    if isinstance(total_reduction_size, sympy.Integer)
                    else self.max_threadgroup_size
                )
                code.writeline(
                    f"[[max_total_threads_per_threadgroup({threadgroup_size})]]"
                )
            code.writeline(f"kernel void {fn_name}(")
            with code.indent():
                for outer, inner in self.args.output_buffers.items():
                    if outer in self.removed_buffers:
                        continue
                    dtype_str = self.dtype_to_str(V.graph.get_dtype(outer))
                    code.writeline(f"device {dtype_str}* {inner},")
                for outer, inner in self.args.input_buffers.items():
````
- **EN**: Touches the filesystem to load, validate, or store compiler artifacts. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这一段会访问文件系统，用于加载、校验或保存编译产物。借助符号表达式来推理索引、形状或代数性质。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。

### Lines 981-1008 / 第 981-1008 行
````python
                    dtype = V.graph.get_dtype(outer)
                    # MPS does not support float64, but scalar inputs are fine
                    if dtype == torch.float64:
                        outer_buf = V.graph.try_get_buffer(outer)
                        if outer_buf is None or outer_buf.get_size() != []:
                            raise RuntimeError("float64 is not supported by MPS")
                        dtype_str = "float"
                    else:
                        dtype_str = self.dtype_to_str(dtype)
                    code.writeline(f"constant {dtype_str}* {inner},")
                for inner in self.args.sizevars.values():
                    code.writeline(f"constant long& {inner},")

                # Write dynamic values as inputs
                for idx_var in idx_vars:
                    if isinstance(idx_var.numel, sympy.Integer):
                        pass
                    else:
                        code.writeline(f"constant long& {idx_var.prefix}numel,")

                # Add error buffer parameter if error header is used
                if "error" in self.headers:
                    code.writeline("device c10::metal::ErrorMessages* error_buf,")

                assert len(idx_vars) < 4, "Up to 3 index variables are supported"
                thread_pos_dtype = (
                    f"uint{len(idx_vars)}" if len(idx_vars) > 1 else "uint"
                )
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1009-1036 / 第 1009-1036 行
````python
                thread_pos_var_name = (
                    idx_vars[0].name if len(idx_vars) == 1 else "thread_pos"
                )
                thread_pos_suffix = "," if self.inside_reduction else ""
                code.writeline(
                    f"{thread_pos_dtype} {thread_pos_var_name} [[thread_position_in_grid]]{thread_pos_suffix}"
                )
                if self.inside_reduction:
                    code.writeline(
                        f"{thread_pos_dtype} group_pos [[thread_position_in_threadgroup]]"
                    )
            code.writeline(") {")
            with code.indent():
                if len(idx_vars) > 1:
                    for idx, var in enumerate(idx_vars):
                        code.writeline(
                            f"auto {var.name} = thread_pos.{chr(120 + idx)};"
                        )
                code.splice(self.indexing_code)
                code.splice(self.body)
            code.writeline("}")

        if V.graph.cpp_wrapper:
            code.writeline(')MTL");')
        else:
            code.writeline("''')")

        return code.getvalue()
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `thread_pos_var_name`, `thread_pos_suffix`, and `else`. This range continues the implementation of function `MetalKernel.codegen_kernel`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `thread_pos_var_name`、`thread_pos_suffix`、`else` 等值。这一段延续了函数`MetalKernel.codegen_kernel` 的具体实现。

### Lines 1037-1064 / 第 1037-1064 行
````python

    def call_kernel(
        self, name: str, node: Any = None, deallocate_ws: bool = True
    ) -> None:
        """
        Codegens a call to this kernel
        """
        wrapper = V.graph.wrapper_code
        # Make sure sizevars has been computed
        for v in self.args.sizevars:
            wrapper.ensure_size_computed(v)

        _, call_args, _, arg_types = self.args.python_argdefs()
        arg_name_to_type = {
            str(call_arg): arg_type for call_arg, arg_type in zip(call_args, arg_types)
        }

        args = [*self.args.output_buffers.keys(), *self.args.input_buffers.keys()]
        args = [arg for arg in args if arg not in self.removed_buffers]
        args += [str(v) for v in self.args.sizevars]
        arg_types = [arg_name_to_type[arg] for arg in args]

        # Add any dynamic ints as inputs
        for tree in self.range_trees:
            if isinstance(tree.numel, (sympy.Integer, int)):
                # Don't need to pass in integers as inputs
                continue
            elif isinstance(tree.numel, sympy.Symbol):
````
- **EN**: Introduces function `call_kernel`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `wrapper`, `arg_name_to_type`, `args`, and `arg_types`.
- **CN**: 这里定义了函数`call_kernel`。借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。初始化或更新了 `wrapper`、`arg_name_to_type`、`args`、`arg_types` 等值。

### Lines 1065-1092 / 第 1065-1092 行
````python
                expr = tree.numel
            else:
                expr = V.graph.wrapper_code.generate_numel_expr(name, tree).inner

            if not tree.is_reduction or self.inside_reduction:
                args.append(str(expr))
                arg_types.append(int)

        expr_printer = self.cexpr if V.graph.cpp_wrapper else self.pexpr

        def format_threads(threads: list[str], kwarg: str) -> str:
            if V.graph.cpp_wrapper:
                threads = [f"static_cast<uint64_t>({t})" for t in threads]
                return f"{{{', '.join(threads)}}}"
            else:
                return f"{kwarg}=[{', '.join(threads)}]"

        # For reduction kernels, limit the maximum size over reduction dimensions to
        # a maximum threadgroup size
        if len(self.active_range_trees()) > 0:
            threads = [
                expr_printer(
                    sympy.Min(v.numel, self.max_threadgroup_size)  # type: ignore[misc]
                    if v.is_reduction
                    else v.numel
                )
                for v in self.active_range_trees()
            ]
````
- **EN**: Introduces function `format_threads`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`format_threads`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1093-1120 / 第 1093-1120 行
````python

            args.append(format_threads(threads, "threads"))
            arg_types.append(list)
        else:
            if V.graph.cpp_wrapper:
                raise RuntimeError("We should always have threads?")

        if self.inside_reduction:
            threads = [
                expr_printer(sympy.Min(v.numel, self.max_threadgroup_size))  # type: ignore[misc]
                if v.is_reduction
                else "1"
                for v in self.active_range_trees()
            ]
            args.append(format_threads(threads, "group_size"))
            arg_types.append(list)
        else:
            if V.graph.cpp_wrapper:
                # Add a None so that we always have a group_size in the
                # arguments. We won't use it if the value is None.
                args += [None]  # type: ignore[list-item]
                arg_types.append(None)

        # Add error buffer index if error reporting is used
        # TODO(malfet) Figure out how to do it for aoti
        if "error" in self.headers and not V.graph.cpp_wrapper:
            args.append(
                f"error_buf_idx={len([arg for arg in args if arg is not None and '=' not in arg])}"
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, and `threads`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`threads` 等值。

### Lines 1121-1148 / 第 1121-1148 行
````python
            )

        wrapper.generate_kernel_call(
            name,
            args,
            device=torch.device("mps"),
            triton=False,
            arg_types=arg_types,
        )

    def device_assert_async(self, cond: CSEVariable, msg: str) -> None:
        if V.graph.cpp_wrapper:
            self.cse.generate(self.compute, f"if (!{cond}) return", assignment=False)
        else:
            self.headers.add("error")
            self.compute.writelines(
                [
                    f"if (!{cond}) {{",
                    f"    TORCH_REPORT_ERROR(error_buf, {repr(msg)});",
                    "    return;",
                    "}",
                ]
            )

    def check_bounds(
        self, expr: sympy.Expr, size: sympy.Expr, lower: bool, upper: bool
    ) -> None:
        if not (lower or upper):
````
- **EN**: Introduces function `device_assert_async`, function `check_bounds`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`device_assert_async`、函数`check_bounds`。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。

### Lines 1149-1176 / 第 1149-1176 行
````python
            return

        expr_str = self.index_to_str(expr)
        size_str = self.index_to_str(size)

        # Generate bounds checking with error reporting
        # TODO(malfet): Is upper bound inclusive or exclusive?
        if lower and upper:
            # Check both lower and upper bounds
            condition = f"({expr_str} < 0 || {expr_str} >= {size_str})"
        elif lower:
            condition = f"{expr_str} < 0"
        else:
            condition = f"{expr_str} >= {size_str}"

        # Generate error reporting code
        if V.graph.cpp_wrapper:
            self.cse.generate(
                self.compute, f"if ({condition}) return", assignment=False
            )
        else:
            # Add error header for error reporting
            self.headers.add("error")
            self.compute.writelines(
                [
                    f"if ({condition}) {{",
                    f'    TORCH_REPORT_ERROR(error_buf, "Index ", {expr_str}, " out of range [0, ", {size_str}, ")");',
                    "    return;",
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `expr_str`, `size_str`, `condition`, and `else`. This range continues the implementation of function `MetalKernel.check_bounds`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `expr_str`、`size_str`、`condition`、`else` 等值。这一段延续了函数`MetalKernel.check_bounds` 的具体实现。

### Lines 1177-1204 / 第 1177-1204 行
````python
                    "}",
                ]
            )


class MetalScheduling(SIMDScheduling):
    kernel_type = MetalKernel  # type: ignore[assignment]
    _kernel_fn_counter: int = 0

    def __init__(self, scheduler: Scheduler | None) -> None:
        super().__init__(scheduler)

    def define_kernel(
        self, src_code: str, node_schedule: list[SchedulerNode], kernel: MetalKernel
    ) -> str:
        wrapper = V.graph.wrapper_code
        if src_code in wrapper.src_to_kernel:
            return wrapper.src_to_kernel[src_code]

        if V.graph.cpp_wrapper:
            # C++ path: one library per kernel (each has a single "generated_kernel" function)
            mps_lib_name = f"mps_lib_{wrapper.next_kernel_suffix()}"
            kernel_name = mps_lib_name
            wrapper.src_to_kernel[src_code] = kernel_name
            src_code = f"const char* {mps_lib_name}_source = " + src_code
            origins, detailed_origins = get_kernel_metadata(node_schedule, wrapper)
            wrapper.define_kernel(
                mps_lib_name, src_code, f"{origins}\n{detailed_origins}", gpu=False
````
- **EN**: Introduces class `MetalScheduling`, function `__init__`, function `define_kernel`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`MetalScheduling`、函数`__init__`、函数`define_kernel`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1205-1232 / 第 1205-1232 行
````python
            )
            return kernel_name

        # Python path: register kernel with async_compile; wait() will compile all
        # accumulated Metal kernels into a single library and replace each placeholder.
        fn_name = f"generated_kernel_{self._kernel_fn_counter}"
        self._kernel_fn_counter += 1
        wrapper.src_to_kernel[src_code] = fn_name

        # Extract Metal source from compile_mps_shader('''...''') call
        metal_src_start = "compile_mps_shader('''"
        start = src_code.index(metal_src_start) + len(metal_src_start)
        end = src_code.rindex("''')")
        metal_src = src_code[start:end]

        # Strip #include lines and rename the kernel function
        body_lines = []
        for line in metal_src.split("\n"):
            if line.strip().startswith("#include"):
                continue
            body_lines.append(
                line.replace("kernel void generated_kernel(", f"kernel void {fn_name}(")
            )

        headers_repr = repr(sorted(kernel.headers))
        wrapper.header.writeline(f"{fn_name} = async_compile.metal({fn_name!r}, '''")
        for line in body_lines:
            wrapper.header.writeline(line)
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `fn_name`, `metal_src_start`, `start`, `end`, `metal_src`, `body_lines`, and `...+1`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `fn_name`、`metal_src_start`、`start`、`end`、`metal_src`、`body_lines`、`另有1项` 等值。

### Lines 1233-1235 / 第 1233-1235 行
````python
        wrapper.header.writeline(f"''', {headers_repr})")

        return fn_name
````
- **EN**: Includes returns or checks that define the contract of this code path. This range continues the implementation of function `MetalScheduling.define_kernel`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`MetalScheduling.define_kernel` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Scheduling decisions  
  **CN**: 调度决策
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Device capability modeling  
  **CN**: 设备能力建模

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `functools`, `itertools`, `logging`, `math`, `pathlib`, `typing`
- **Third-party / 第三方**: `sympy`, `sympy.printing.precedence`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.utils._cpp_embed_headers`, `torch.utils._ordered_set`, `torch.utils._sympy.printers`, `torch.utils._sympy.value_ranges`, `..utils`, `..virtualized`, `.common`, `.simd`, `..ops_handler`, `..scheduler`
