# numbers.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/utils/_sympy/numbers.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements utility logic that supports broader PyTorch execution, packaging, or developer workflows. This specific file centers on `numbers.py`. Key abstractions such as `IntInfinity, NegativeIntInfinity` shape the public surface of the file.
- **Purpose (CN) / 用途（中文）**: 实现支撑更广泛 PyTorch 执行、打包或开发者工作流的辅助逻辑。 该文件具体围绕 `numbers.py` 展开。 `IntInfinity, NegativeIntInfinity` 等关键抽象塑造了该文件的公开接口。
## Line-by-Line Analysis / 逐行分析

### Lines 1-23 / 第 1-23 行
```python
# mypy: allow-untyped-defs
import mpmath.libmp as mlib  # type: ignore[import-untyped]
import sympy
from sympy import Expr
from sympy.core.decorators import _sympifyit
from sympy.core.expr import AtomicExpr
from sympy.core.numbers import Number
from sympy.core.parameters import global_parameters
from sympy.core.singleton import S, Singleton


# pyrefly: ignore [invalid-inheritance]
class IntInfinity(Number, metaclass=Singleton):
    r"""Positive integer infinite quantity.

    Integer infinity is a value in an extended integers which
    is greater than all other integers.  We distinguish it from
    sympy's existing notion of infinity in that it reports that
    it is_integer.

    Infinity is a singleton, and can be accessed by ``S.IntInfinity``,
    or can be imported as ``int_oo``.
    """
```
- **EN**: This block establishes the module dependencies, pulling in external packages such as mpmath.libmp, sympy, sympy:Expr, sympy.core.decorators:_sympifyit. It introduces or extends class-level abstractions such as `IntInfinity`, which organize state and behavior for this subsystem. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段建立模块依赖，引入了外部包，如 mpmath.libmp, sympy, sympy:Expr, sympy.core.decorators:_sympifyit。 它引入或扩展了 `IntInfinity` 等类级抽象，用于组织该子系统的状态与行为。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 25-46 / 第 25-46 行
```python
    # NB: We can't actually mark this as infinite, as integer and infinite are
    # inconsistent assumptions in sympy.  We also report that we are complex,
    # different from sympy.oo

    is_integer = True
    is_commutative = True
    is_number = True
    is_extended_real = True
    is_comparable = True
    is_extended_positive = True
    is_prime = False

    # Ensure we get dispatched to before plain numbers
    _op_priority = 100.0

    __slots__ = ()

    def __new__(cls):
        return AtomicExpr.__new__(cls)

    def _sympystr(self, printer) -> str:
        return "int_oo"
```
- **EN**: It introduces or extends class-level abstractions such as `IntInfinity`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `IntInfinity` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 48-69 / 第 48-69 行
```python
    def _eval_subs(self, old, new):
        if self == old:
            return new

    # We could do these, not sure about it
    """
    def _eval_evalf(self, prec=None):
        return Float('inf')

    def evalf(self, prec=None, **options):
        return self._eval_evalf(prec)
    """

    @_sympifyit("other", NotImplemented)
    def __add__(self, other):
        if isinstance(other, Number) and global_parameters.evaluate:
            if other in (S.Infinity, S.NegativeInfinity):
                return other
            if other in (S.NegativeIntInfinity, S.NaN):
                return S.NaN
            return self
        return Number.__add__(self, other)
```
- **EN**: It introduces or extends class-level abstractions such as `IntInfinity`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `IntInfinity` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 71-87 / 第 71-87 行
```python
    __radd__ = __add__

    @_sympifyit("other", NotImplemented)
    def __sub__(self, other):
        if isinstance(other, Number) and global_parameters.evaluate:
            if other is S.Infinity:
                return S.NegativeInfinity
            if other is S.NegativeInfinity:
                return S.Infinity
            if other in (S.IntInfinity, S.NaN):
                return S.NaN
            return self
        return Number.__sub__(self, other)

    @_sympifyit("other", NotImplemented)
    def __rsub__(self, other):
        return (-self).__add__(other)
```
- **EN**: It introduces or extends class-level abstractions such as `IntInfinity`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `IntInfinity` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 89-112 / 第 89-112 行
```python
    @_sympifyit("other", NotImplemented)
    def __mul__(self, other):
        if isinstance(other, Number) and global_parameters.evaluate:
            if other.is_zero or other is S.NaN:
                return S.NaN
            if other.is_extended_positive:
                return self
            return S.NegativeIntInfinity
        return Number.__mul__(self, other)

    __rmul__ = __mul__

    @_sympifyit("other", NotImplemented)
    def __truediv__(self, other):
        if isinstance(other, Number) and global_parameters.evaluate:
            if other in (
                S.Infinity,
                S.IntInfinity,
                S.NegativeInfinity,
                S.NegativeIntInfinity,
                S.NaN,
            ):
                return S.NaN
            if other.is_extended_nonnegative:
```
- **EN**: It introduces or extends class-level abstractions such as `IntInfinity`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `IntInfinity` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 113-133 / 第 113-133 行
```python
                return S.Infinity  # truediv produces float
            return S.NegativeInfinity  # truediv produces float
        return Number.__truediv__(self, other)

    def __abs__(self):
        return S.IntInfinity

    def __neg__(self):
        return S.NegativeIntInfinity

    def _eval_power(self, expt):
        if expt.is_extended_positive:
            return S.IntInfinity
        if expt.is_extended_negative:
            return S.Zero
        if expt is S.NaN:
            return S.NaN
        if expt is S.ComplexInfinity:
            return S.NaN
        if expt.is_extended_real is False and expt.is_number:
            from sympy.functions.elementary.complexes import re
```
- **EN**: It introduces or extends class-level abstractions such as `IntInfinity`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `IntInfinity` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 135-155 / 第 135-155 行
```python
            expt_real = re(expt)
            if expt_real.is_positive:
                return S.ComplexInfinity
            if expt_real.is_negative:
                return S.Zero
            if expt_real.is_zero:
                return S.NaN

            return self ** expt.evalf()

    def _as_mpf_val(self, prec):
        return mlib.finf

    def __hash__(self):
        return super().__hash__()

    def __eq__(self, other):
        return other is S.IntInfinity

    def __ne__(self, other):
        return other is not S.IntInfinity
```
- **EN**: It introduces or extends class-level abstractions such as `IntInfinity`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `IntInfinity` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 157-179 / 第 157-179 行
```python
    def __gt__(self, other):
        if other is S.Infinity:
            return sympy.false  # sympy.oo > int_oo
        elif other is S.IntInfinity:
            return sympy.false  # consistency with sympy.oo
        else:
            return sympy.true

    def __ge__(self, other):
        if other is S.Infinity:
            return sympy.false  # sympy.oo > int_oo
        elif other is S.IntInfinity:
            return sympy.true  # consistency with sympy.oo
        else:
            return sympy.true

    def __lt__(self, other):
        if other is S.Infinity:
            return sympy.true  # sympy.oo > int_oo
        elif other is S.IntInfinity:
            return sympy.false  # consistency with sympy.oo
        else:
            return sympy.false
```
- **EN**: It introduces or extends class-level abstractions such as `IntInfinity`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `IntInfinity` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 181-201 / 第 181-201 行
```python
    def __le__(self, other):
        if other is S.Infinity:
            return sympy.true  # sympy.oo > int_oo
        elif other is S.IntInfinity:
            return sympy.true  # consistency with sympy.oo
        else:
            return sympy.false

    @_sympifyit("other", NotImplemented)
    def __mod__(self, other):
        if not isinstance(other, Expr):
            return NotImplemented
        return S.NaN

    __rmod__ = __mod__

    def floor(self):
        return self

    def ceiling(self):
        return self
```
- **EN**: It introduces or extends class-level abstractions such as `IntInfinity`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `IntInfinity` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 204-222 / 第 204-222 行
```python
int_oo = S.IntInfinity


def is_infinite(expr) -> bool:
    """Check if an expression is any type of infinity (positive or negative).

    This handles both sympy's built-in infinities (oo, -oo) and PyTorch's
    integer infinities (int_oo, -int_oo).

    Note: We cannot rely on sympy's is_finite property because IntInfinity
    and NegativeIntInfinity have is_integer=True, which implies is_finite=True
    in sympy's assumption system.
    """
    return expr in (
        S.Infinity,
        S.NegativeInfinity,
        S.IntInfinity,
        S.NegativeIntInfinity,
    )
```
- **EN**: Key callable entry points in this range include `is_infinite`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `is_infinite`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 225-247 / 第 225-247 行
```python
# pyrefly: ignore [invalid-inheritance]
class NegativeIntInfinity(Number, metaclass=Singleton):
    """Negative integer infinite quantity.

    NegativeInfinity is a singleton, and can be accessed
    by ``S.NegativeInfinity``.

    See Also
    ========

    IntInfinity
    """

    # Ensure we get dispatched to before plain numbers
    _op_priority = 100.0

    is_integer = True
    is_extended_real = True
    is_commutative = True
    is_comparable = True
    is_extended_negative = True
    is_number = True
    is_prime = False
```
- **EN**: It introduces or extends class-level abstractions such as `NegativeIntInfinity`, which organize state and behavior for this subsystem. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 它引入或扩展了 `NegativeIntInfinity` 等类级抽象，用于组织该子系统的状态与行为。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 249-267 / 第 249-267 行
```python
    __slots__ = ()

    def __new__(cls):
        return AtomicExpr.__new__(cls)

    def _eval_subs(self, old, new):
        if self == old:
            return new

    def _sympystr(self, printer) -> str:
        return "-int_oo"

    """
    def _eval_evalf(self, prec=None):
        return Float('-inf')

    def evalf(self, prec=None, **options):
        return self._eval_evalf(prec)
    """
```
- **EN**: It introduces or extends class-level abstractions such as `NegativeIntInfinity`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `NegativeIntInfinity` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 269-289 / 第 269-289 行
```python
    @_sympifyit("other", NotImplemented)
    def __add__(self, other):
        if isinstance(other, Number) and global_parameters.evaluate:
            if other is S.Infinity:
                return S.Infinity
            if other in (S.IntInfinity, S.NaN):
                return S.NaN
            return self
        return Number.__add__(self, other)

    __radd__ = __add__

    @_sympifyit("other", NotImplemented)
    def __sub__(self, other):
        if isinstance(other, Number) and global_parameters.evaluate:
            if other is S.NegativeInfinity:
                return S.Infinity
            if other in (S.NegativeIntInfinity, S.NaN):
                return S.NaN
            return self
        return Number.__sub__(self, other)
```
- **EN**: It introduces or extends class-level abstractions such as `NegativeIntInfinity`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `NegativeIntInfinity` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 291-305 / 第 291-305 行
```python
    @_sympifyit("other", NotImplemented)
    def __rsub__(self, other):
        return (-self).__add__(other)

    @_sympifyit("other", NotImplemented)
    def __mul__(self, other):
        if isinstance(other, Number) and global_parameters.evaluate:
            if other.is_zero or other is S.NaN:
                return S.NaN
            if other.is_extended_positive:
                return self
            return S.IntInfinity
        return Number.__mul__(self, other)

    __rmul__ = __mul__
```
- **EN**: It introduces or extends class-level abstractions such as `NegativeIntInfinity`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `NegativeIntInfinity` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 307-327 / 第 307-327 行
```python
    @_sympifyit("other", NotImplemented)
    def __truediv__(self, other):
        if isinstance(other, Number) and global_parameters.evaluate:
            if other in (
                S.Infinity,
                S.IntInfinity,
                S.NegativeInfinity,
                S.NegativeIntInfinity,
                S.NaN,
            ):
                return S.NaN
            if other.is_extended_nonnegative:
                return self
            return S.Infinity  # truediv returns float
        return Number.__truediv__(self, other)

    def __abs__(self):
        return S.IntInfinity

    def __neg__(self):
        return S.IntInfinity
```
- **EN**: It introduces or extends class-level abstractions such as `NegativeIntInfinity`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `NegativeIntInfinity` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 329-344 / 第 329-344 行
```python
    def _eval_power(self, expt):
        if expt.is_number:
            if expt in (
                S.NaN,
                S.Infinity,
                S.NegativeInfinity,
                S.IntInfinity,
                S.NegativeIntInfinity,
            ):
                return S.NaN

            if isinstance(expt, sympy.Integer) and expt.is_extended_positive:
                if expt.is_odd:
                    return S.NegativeIntInfinity
                else:
                    return S.IntInfinity
```
- **EN**: It introduces or extends class-level abstractions such as `NegativeIntInfinity`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `NegativeIntInfinity` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 346-368 / 第 346-368 行
```python
            inf_part = S.IntInfinity**expt
            s_part = S.NegativeOne**expt
            if inf_part == 0 and s_part.is_finite:
                return inf_part
            if (
                inf_part is S.ComplexInfinity
                and s_part.is_finite
                and not s_part.is_zero
            ):
                return S.ComplexInfinity
            return s_part * inf_part

    def _as_mpf_val(self, prec):
        return mlib.fninf

    def __hash__(self):
        return super().__hash__()

    def __eq__(self, other):
        return other is S.NegativeIntInfinity

    def __ne__(self, other):
        return other is not S.NegativeIntInfinity
```
- **EN**: It introduces or extends class-level abstractions such as `NegativeIntInfinity`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `NegativeIntInfinity` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 370-392 / 第 370-392 行
```python
    def __gt__(self, other):
        if other is S.NegativeInfinity:
            return sympy.true  # -sympy.oo < -int_oo
        elif other is S.NegativeIntInfinity:
            return sympy.false  # consistency with sympy.oo
        else:
            return sympy.false

    def __ge__(self, other):
        if other is S.NegativeInfinity:
            return sympy.true  # -sympy.oo < -int_oo
        elif other is S.NegativeIntInfinity:
            return sympy.true  # consistency with sympy.oo
        else:
            return sympy.false

    def __lt__(self, other):
        if other is S.NegativeInfinity:
            return sympy.false  # -sympy.oo < -int_oo
        elif other is S.NegativeIntInfinity:
            return sympy.false  # consistency with sympy.oo
        else:
            return sympy.true
```
- **EN**: It introduces or extends class-level abstractions such as `NegativeIntInfinity`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `NegativeIntInfinity` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 394-417 / 第 394-417 行
```python
    def __le__(self, other):
        if other is S.NegativeInfinity:
            return sympy.false  # -sympy.oo < -int_oo
        elif other is S.NegativeIntInfinity:
            return sympy.true  # consistency with sympy.oo
        else:
            return sympy.true

    @_sympifyit("other", NotImplemented)
    def __mod__(self, other):
        if not isinstance(other, Expr):
            return NotImplemented
        return S.NaN

    __rmod__ = __mod__

    def floor(self):
        return self

    def ceiling(self):
        return self

    def as_powers_dict(self):
        return {S.NegativeOne: 1, S.IntInfinity: 1}
```
- **EN**: It introduces or extends class-level abstractions such as `NegativeIntInfinity`, which organize state and behavior for this subsystem. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `NegativeIntInfinity` 等类级抽象，用于组织该子系统的状态与行为。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements utility logic that supports broader PyTorch execution, packaging, or developer workflows.
  - CN: 实现支撑更广泛 PyTorch 执行、打包或开发者工作流的辅助逻辑。
- **Quantization pipeline**
  - EN: Tracks calibration statistics, quantization parameters, or graph/module rewrites for lower-precision execution.
  - CN: 跟踪校准统计信息、量化参数，或面向低精度执行的图/模块改写。
- **IntInfinity**
  - EN: `IntInfinity` is one of the main classes that structures the file's behavior.
  - CN: `IntInfinity` 是组织该文件行为的核心类之一。
- **NegativeIntInfinity**
  - EN: `NegativeIntInfinity` is one of the main classes that structures the file's behavior.
  - CN: `NegativeIntInfinity` 是组织该文件行为的核心类之一。
- **is_infinite**
  - EN: `is_infinite` is a representative function that exposes or coordinates an important action in this module.
  - CN: `is_infinite` 是一个具有代表性的函数，用于暴露或协调此模块中的关键动作。
## Dependencies / 依赖关系

- **Third-party packages / 第三方包**: `mpmath.libmp`, `sympy`, `sympy:Expr`, `sympy.core.decorators:_sympifyit`, `sympy.core.expr:AtomicExpr`, `sympy.core.numbers:Number`, `sympy.core.parameters:global_parameters`, `sympy.core.singleton:S`, `sympy.core.singleton:Singleton`
- **Primary symbols / 核心符号**: `IntInfinity`, `NegativeIntInfinity`, `is_infinite`
