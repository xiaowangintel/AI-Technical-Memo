# float2.py — Code Analysis / 代码分析

## Source / 来源

- **Path:** `./python/triton/experimental/gluon/language/nvidia/blackwell/float2.py`
- **EN:** This source file at `./python/triton/experimental/gluon/language/nvidia/blackwell/float2.py` defines the main symbols `Float2Tensor`, `_add_f32x2`, `_sub_f32x2`, `_mul_f32x2`, `_fma_f32x2` for the surrounding Triton subsystem.
- **CN:** 位于 `./python/triton/experimental/gluon/language/nvidia/blackwell/float2.py` 的这个源文件为周边 Triton 子系统定义了主要符号 `Float2Tensor`, `_add_f32x2`, `_sub_f32x2`, `_mul_f32x2`, `_fma_f32x2`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1
```python
from triton.experimental.gluon import aggregate
```
**EN:** At module scope, this block imports aggregate from `triton.experimental.gluon` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `triton.experimental.gluon` 导入 aggregate，把当前文件与周边 API 和辅助工具连接起来。

### Lines 2-2
```python
from triton.experimental.gluon.language import _core as ttgl, _standard as stdlib
```
**EN:** At module scope, this block imports _core as ttgl, _standard as stdlib from `triton.experimental.gluon.language` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `triton.experimental.gluon.language` 导入 _core as ttgl, _standard as stdlib，把当前文件与周边 API 和辅助工具连接起来。

### Lines 3-3
```python
from triton.experimental.gluon._runtime import constexpr_function, jit
```
**EN:** At module scope, this block imports constexpr_function, jit from `triton.experimental.gluon._runtime` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `triton.experimental.gluon._runtime` 导入 constexpr_function, jit，把当前文件与周边 API 和辅助工具连接起来。

### Lines 5-12
```python
__all__ = [
    "pack2",
    "unpack2",
    "pack",
    "unpack",
    "fma",
    "Float2Tensor",
]
```
**EN:** At module scope, this assignment updates `__all__` with `['pack2', 'unpack2', 'pack', 'unpack', 'fma', 'Float2Tensor']`, establishing state, aliases, or configuration used later.
**CN:** 在模块级作用域中，这段赋值把 `['pack2', 'unpack2', 'pack', 'unpack', 'fma', 'Float2Tensor']` 写入 `__all__`，为后续逻辑建立状态、别名或配置。

### Lines 15-16
```python
@jit
def _add_f32x2(a, b):
```
**EN:** At module scope, this header declares the function `_add_f32x2(a, b)`, which is responsible for add f32x2. Decorators: jit.
**CN:** 在模块级作用域中，这段头部声明了函数 `_add_f32x2(a, b)`，它负责处理 add f32x2 相关逻辑。 装饰器包括：jit。

### Lines 17-26
```python
    return ttgl.inline_asm_elementwise(
        """
        add.f32x2 $0, $1, $2;
        """,
        "=l,l,l",
        [a, b],
        dtype=ttgl.int64,
        is_pure=True,
        pack=1,
    )
```
**EN:** Inside function `_add_f32x2`, this return statement sends `ttgl.inline_asm_elementwise('\n add.f32x2 $0, $1, $2;\n ', '=l,l,l', [a, b], dtype=ttgl.int64, is...` back to the caller as the result of the current routine.
**CN:** 在函数 `_add_f32x2` 内部，这条返回语句把 `ttgl.inline_asm_elementwise('\n add.f32x2 $0, $1, $2;\n ', '=l,l,l', [a, b], dtype=ttgl.int64, is...` 作为当前过程的结果返回给调用方。

### Lines 29-30
```python
@jit
def _sub_f32x2(a, b):
```
**EN:** At module scope, this header declares the function `_sub_f32x2(a, b)`, which is responsible for sub f32x2. Decorators: jit.
**CN:** 在模块级作用域中，这段头部声明了函数 `_sub_f32x2(a, b)`，它负责处理 sub f32x2 相关逻辑。 装饰器包括：jit。

### Lines 31-40
```python
    return ttgl.inline_asm_elementwise(
        """
        sub.f32x2 $0, $1, $2;
        """,
        "=l,l,l",
        [a, b],
        dtype=ttgl.int64,
        is_pure=True,
        pack=1,
    )
```
**EN:** Inside function `_sub_f32x2`, this return statement sends `ttgl.inline_asm_elementwise('\n sub.f32x2 $0, $1, $2;\n ', '=l,l,l', [a, b], dtype=ttgl.int64, is...` back to the caller as the result of the current routine.
**CN:** 在函数 `_sub_f32x2` 内部，这条返回语句把 `ttgl.inline_asm_elementwise('\n sub.f32x2 $0, $1, $2;\n ', '=l,l,l', [a, b], dtype=ttgl.int64, is...` 作为当前过程的结果返回给调用方。

### Lines 43-44
```python
@jit
def _mul_f32x2(a, b):
```
**EN:** At module scope, this header declares the function `_mul_f32x2(a, b)`, which is responsible for mul f32x2. Decorators: jit.
**CN:** 在模块级作用域中，这段头部声明了函数 `_mul_f32x2(a, b)`，它负责处理 mul f32x2 相关逻辑。 装饰器包括：jit。

### Lines 45-54
```python
    return ttgl.inline_asm_elementwise(
        """
        mul.f32x2 $0, $1, $2;
        """,
        "=l,l,l",
        [a, b],
        dtype=ttgl.int64,
        is_pure=True,
        pack=1,
    )
```
**EN:** Inside function `_mul_f32x2`, this return statement sends `ttgl.inline_asm_elementwise('\n mul.f32x2 $0, $1, $2;\n ', '=l,l,l', [a, b], dtype=ttgl.int64, is...` back to the caller as the result of the current routine.
**CN:** 在函数 `_mul_f32x2` 内部，这条返回语句把 `ttgl.inline_asm_elementwise('\n mul.f32x2 $0, $1, $2;\n ', '=l,l,l', [a, b], dtype=ttgl.int64, is...` 作为当前过程的结果返回给调用方。

### Lines 57-58
```python
@jit
def _fma_f32x2(a, b, c):
```
**EN:** At module scope, this header declares the function `_fma_f32x2(a, b, c)`, which is responsible for fma f32x2. Decorators: jit.
**CN:** 在模块级作用域中，这段头部声明了函数 `_fma_f32x2(a, b, c)`，它负责处理 fma f32x2 相关逻辑。 装饰器包括：jit。

### Lines 59-68
```python
    return ttgl.inline_asm_elementwise(
        """
        fma.rn.f32x2 $0, $1, $2, $3;
        """,
        "=l,l,l,l",
        [a, b, c],
        dtype=ttgl.int64,
        is_pure=True,
        pack=1,
    )
```
**EN:** Inside function `_fma_f32x2`, this return statement sends `ttgl.inline_asm_elementwise('\n fma.rn.f32x2 $0, $1, $2, $3;\n ', '=l,l,l,l', [a, b, c], dtype=tt...` back to the caller as the result of the current routine.
**CN:** 在函数 `_fma_f32x2` 内部，这条返回语句把 `ttgl.inline_asm_elementwise('\n fma.rn.f32x2 $0, $1, $2, $3;\n ', '=l,l,l,l', [a, b, c], dtype=tt...` 作为当前过程的结果返回给调用方。

### Lines 71-72
```python
@aggregate
class Float2Tensor:
```
**EN:** At module scope, this header defines class `Float2Tensor`, a container for float2 tensor related behavior. Decorators: aggregate.
**CN:** 在模块级作用域中，这段头部定义了类 `Float2Tensor`，用于封装 float2 tensor 相关行为。 装饰器包括：aggregate。

### Lines 73-73
```python
    value: ttgl.tensor
```
**EN:** Inside class `Float2Tensor`, this annotated declaration introduces `value` with type `ttgl.tensor`, documenting expected structure for later use.
**CN:** 在类 `Float2Tensor` 内部，这条带注解的声明为 `value` 指定了类型 `ttgl.tensor`，用来说明后续使用时期望的数据结构。

### Lines 75-76
```python
    @constexpr_function
    def __init__(self, value: ttgl.tensor):
```
**EN:** Inside class `Float2Tensor`, this header declares the function `__init__(self, value)`, which is responsible for object initialization. Decorators: constexpr_function.
**CN:** 在类 `Float2Tensor` 内部，这段头部声明了函数 `__init__(self, value)`，它负责处理 对象初始化 相关逻辑。 装饰器包括：constexpr_function。

### Lines 77-77
```python
        self.value = value
```
**EN:** Inside class `Float2Tensor` and function `__init__`, this assignment updates `self.value` with `value`, establishing state, aliases, or configuration used later.
**CN:** 在类 `Float2Tensor`、函数 `__init__` 内部，这段赋值把 `value` 写入 `self.value`，为后续逻辑建立状态、别名或配置。

### Lines 79-80
```python
    @jit
    def __add__(self, rhs):
```
**EN:** Inside class `Float2Tensor`, this header declares the function `__add__(self, rhs)`, which is responsible for add. Decorators: jit.
**CN:** 在类 `Float2Tensor` 内部，这段头部声明了函数 `__add__(self, rhs)`，它负责处理 add 相关逻辑。 装饰器包括：jit。

### Lines 81-81
```python
        ttgl.static_assert(isinstance(rhs, Float2Tensor), "rhs must be a Float2Tensor")
```
**EN:** Inside class `Float2Tensor` and function `__add__`, this expression evaluates `ttgl.static_assert` mainly for its side effects or registration behavior.
**CN:** 在类 `Float2Tensor`、函数 `__add__` 内部，这条表达式计算 `ttgl.static_assert`，主要目的是触发副作用或完成注册行为。

### Lines 82-82
```python
        return Float2Tensor(_add_f32x2(self.value, rhs.value))
```
**EN:** Inside class `Float2Tensor` and function `__add__`, this return statement sends `Float2Tensor(_add_f32x2(self.value, rhs.value))` back to the caller as the result of the current routine.
**CN:** 在类 `Float2Tensor`、函数 `__add__` 内部，这条返回语句把 `Float2Tensor(_add_f32x2(self.value, rhs.value))` 作为当前过程的结果返回给调用方。

### Lines 84-85
```python
    @jit
    def __sub__(self, rhs):
```
**EN:** Inside class `Float2Tensor`, this header declares the function `__sub__(self, rhs)`, which is responsible for sub. Decorators: jit.
**CN:** 在类 `Float2Tensor` 内部，这段头部声明了函数 `__sub__(self, rhs)`，它负责处理 sub 相关逻辑。 装饰器包括：jit。

### Lines 86-86
```python
        ttgl.static_assert(isinstance(rhs, Float2Tensor), "rhs must be a Float2Tensor")
```
**EN:** Inside class `Float2Tensor` and function `__sub__`, this expression evaluates `ttgl.static_assert` mainly for its side effects or registration behavior.
**CN:** 在类 `Float2Tensor`、函数 `__sub__` 内部，这条表达式计算 `ttgl.static_assert`，主要目的是触发副作用或完成注册行为。

### Lines 87-87
```python
        return Float2Tensor(_sub_f32x2(self.value, rhs.value))
```
**EN:** Inside class `Float2Tensor` and function `__sub__`, this return statement sends `Float2Tensor(_sub_f32x2(self.value, rhs.value))` back to the caller as the result of the current routine.
**CN:** 在类 `Float2Tensor`、函数 `__sub__` 内部，这条返回语句把 `Float2Tensor(_sub_f32x2(self.value, rhs.value))` 作为当前过程的结果返回给调用方。

### Lines 89-90
```python
    @jit
    def __mul__(self, rhs):
```
**EN:** Inside class `Float2Tensor`, this header declares the function `__mul__(self, rhs)`, which is responsible for mul. Decorators: jit.
**CN:** 在类 `Float2Tensor` 内部，这段头部声明了函数 `__mul__(self, rhs)`，它负责处理 mul 相关逻辑。 装饰器包括：jit。

### Lines 91-91
```python
        ttgl.static_assert(isinstance(rhs, Float2Tensor), "rhs must be a Float2Tensor")
```
**EN:** Inside class `Float2Tensor` and function `__mul__`, this expression evaluates `ttgl.static_assert` mainly for its side effects or registration behavior.
**CN:** 在类 `Float2Tensor`、函数 `__mul__` 内部，这条表达式计算 `ttgl.static_assert`，主要目的是触发副作用或完成注册行为。

### Lines 92-92
```python
        return Float2Tensor(_mul_f32x2(self.value, rhs.value))
```
**EN:** Inside class `Float2Tensor` and function `__mul__`, this return statement sends `Float2Tensor(_mul_f32x2(self.value, rhs.value))` back to the caller as the result of the current routine.
**CN:** 在类 `Float2Tensor`、函数 `__mul__` 内部，这条返回语句把 `Float2Tensor(_mul_f32x2(self.value, rhs.value))` 作为当前过程的结果返回给调用方。

### Lines 94-95
```python
    @jit
    def sum(self, axis: ttgl.constexpr):
```
**EN:** Inside class `Float2Tensor`, this header declares the function `sum(self, axis)`, which is responsible for sum. Decorators: jit.
**CN:** 在类 `Float2Tensor` 内部，这段头部声明了函数 `sum(self, axis)`，它负责处理 sum 相关逻辑。 装饰器包括：jit。

### Lines 96-96
```python
        return Float2Tensor(ttgl.reduce(self.value, axis=axis, combine_fn=_add_f32x2))
```
**EN:** Inside class `Float2Tensor` and function `sum`, this return statement sends `Float2Tensor(ttgl.reduce(self.value, axis=axis, combine_fn=_add_f32x2))` back to the caller as the result of the current routine.
**CN:** 在类 `Float2Tensor`、函数 `sum` 内部，这条返回语句把 `Float2Tensor(ttgl.reduce(self.value, axis=axis, combine_fn=_add_f32x2))` 作为当前过程的结果返回给调用方。

### Lines 99-100
```python
@jit
def pack2(x0, x1):
```
**EN:** At module scope, this header declares the function `pack2(x0, x1)`, which is responsible for pack2. Decorators: jit.
**CN:** 在模块级作用域中，这段头部声明了函数 `pack2(x0, x1)`，它负责处理 pack2 相关逻辑。 装饰器包括：jit。

### Lines 101-110
```python
    value = ttgl.inline_asm_elementwise(
        """
        mov.b64 $0, { $1, $2 };
        """,
        "=l,r,r",
        [x0, x1],
        dtype=ttgl.int64,
        is_pure=True,
        pack=1,
    )
```
**EN:** Inside function `pack2`, this assignment updates `value` with `ttgl.inline_asm_elementwise('\n mov.b64 $0, { $1, $2 };\n ', '=l,r,r', [x0, x...`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `pack2` 内部，这段赋值把 `ttgl.inline_asm_elementwise('\n mov.b64 $0, { $1, $2 };\n ', '=l,r,r', [x0, x...` 写入 `value`，为后续逻辑建立状态、别名或配置。

### Lines 111-111
```python
    return Float2Tensor(value)
```
**EN:** Inside function `pack2`, this return statement sends `Float2Tensor(value)` back to the caller as the result of the current routine.
**CN:** 在函数 `pack2` 内部，这条返回语句把 `Float2Tensor(value)` 作为当前过程的结果返回给调用方。

### Lines 114-115
```python
@jit
def unpack2(x):
```
**EN:** At module scope, this header declares the function `unpack2(x)`, which is responsible for unpack2. Decorators: jit.
**CN:** 在模块级作用域中，这段头部声明了函数 `unpack2(x)`，它负责处理 unpack2 相关逻辑。 装饰器包括：jit。

### Lines 116-125
```python
    return ttgl.inline_asm_elementwise(
        """
        mov.b64 { $0, $1 }, $2;
        """,
        "=r,=r,l",
        [x.value],
        dtype=[ttgl.float32, ttgl.float32],
        is_pure=True,
        pack=1,
    )
```
**EN:** Inside function `unpack2`, this return statement sends `ttgl.inline_asm_elementwise('\n mov.b64 { $0, $1 }, $2;\n ', '=r,=r,l', [x.value], dtype=[ttgl.fl...` back to the caller as the result of the current routine.
**CN:** 在函数 `unpack2` 内部，这条返回语句把 `ttgl.inline_asm_elementwise('\n mov.b64 { $0, $1 }, $2;\n ', '=r,=r,l', [x.value], dtype=[ttgl.fl...` 作为当前过程的结果返回给调用方。

### Lines 128-129
```python
@constexpr_function
def _get_split_shape(shape, axis):
```
**EN:** At module scope, this header declares the function `_get_split_shape(shape, axis)`, which is responsible for get split shape. Decorators: constexpr_function.
**CN:** 在模块级作用域中，这段头部声明了函数 `_get_split_shape(shape, axis)`，它负责处理 get split shape 相关逻辑。 装饰器包括：constexpr_function。

### Lines 130-130
```python
    shape = [d for d in shape]
```
**EN:** Inside function `_get_split_shape`, this assignment updates `shape` with `[d for d in shape]`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `_get_split_shape` 内部，这段赋值把 `[d for d in shape]` 写入 `shape`，为后续逻辑建立状态、别名或配置。

### Lines 131-131
```python
    assert shape[axis] >= 2, f"not enough elements to pack along axis {axis}"
```
**EN:** Inside function `_get_split_shape`, this assertion enforces `shape[axis] >= 2` so invalid states are caught early during execution.
**CN:** 在函数 `_get_split_shape` 内部，这条断言要求 `shape[axis] >= 2` 成立，从而在执行早期捕获非法状态。

### Lines 132-132
```python
    shape[axis] //= 2
```
**EN:** Inside function `_get_split_shape`, this assignment updates `shape[axis]` with `2`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `_get_split_shape` 内部，这段赋值把 `2` 写入 `shape[axis]`，为后续逻辑建立状态、别名或配置。

### Lines 133-133
```python
    shape.insert(axis + 1, 2)
```
**EN:** Inside function `_get_split_shape`, this expression evaluates `shape.insert` mainly for its side effects or registration behavior.
**CN:** 在函数 `_get_split_shape` 内部，这条表达式计算 `shape.insert`，主要目的是触发副作用或完成注册行为。

### Lines 134-134
```python
    permute = list(range(len(shape)))
```
**EN:** Inside function `_get_split_shape`, this assignment updates `permute` with `list(range(len(shape)))`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `_get_split_shape` 内部，这段赋值把 `list(range(len(shape)))` 写入 `permute`，为后续逻辑建立状态、别名或配置。

### Lines 135-135
```python
    permute[axis + 1], permute[len(permute) - 1] = permute[len(permute) - 1], permute[axis + 1]
```
**EN:** Inside function `_get_split_shape`, this assignment updates `(permute[axis + 1], permute[len(permute) - 1])` with `(permute[len(permute) - 1], permute[axis + 1])`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `_get_split_shape` 内部，这段赋值把 `(permute[len(permute) - 1], permute[axis + 1])` 写入 `(permute[axis + 1], permute[len(permute) - 1])`，为后续逻辑建立状态、别名或配置。

### Lines 136-136
```python
    return ttgl.tuple(shape), ttgl.tuple(permute)
```
**EN:** Inside function `_get_split_shape`, this return statement sends `(ttgl.tuple(shape), ttgl.tuple(permute))` back to the caller as the result of the current routine.
**CN:** 在函数 `_get_split_shape` 内部，这条返回语句把 `(ttgl.tuple(shape), ttgl.tuple(permute))` 作为当前过程的结果返回给调用方。

### Lines 139-140
```python
@constexpr_function
def _get_join_shape(shape, axis):
```
**EN:** At module scope, this header declares the function `_get_join_shape(shape, axis)`, which is responsible for get join shape. Decorators: constexpr_function.
**CN:** 在模块级作用域中，这段头部声明了函数 `_get_join_shape(shape, axis)`，它负责处理 get join shape 相关逻辑。 装饰器包括：constexpr_function。

### Lines 141-141
```python
    shape = [d for d in shape]
```
**EN:** Inside function `_get_join_shape`, this assignment updates `shape` with `[d for d in shape]`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `_get_join_shape` 内部，这段赋值把 `[d for d in shape]` 写入 `shape`，为后续逻辑建立状态、别名或配置。

### Lines 142-142
```python
    shape[axis] *= 2
```
**EN:** Inside function `_get_join_shape`, this assignment updates `shape[axis]` with `2`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `_get_join_shape` 内部，这段赋值把 `2` 写入 `shape[axis]`，为后续逻辑建立状态、别名或配置。

### Lines 143-143
```python
    permute = list(range(len(shape)))
```
**EN:** Inside function `_get_join_shape`, this assignment updates `permute` with `list(range(len(shape)))`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `_get_join_shape` 内部，这段赋值把 `list(range(len(shape)))` 写入 `permute`，为后续逻辑建立状态、别名或配置。

### Lines 144-144
```python
    permute.insert(axis + 1, len(permute))
```
**EN:** Inside function `_get_join_shape`, this expression evaluates `permute.insert` mainly for its side effects or registration behavior.
**CN:** 在函数 `_get_join_shape` 内部，这条表达式计算 `permute.insert`，主要目的是触发副作用或完成注册行为。

### Lines 145-145
```python
    return ttgl.tuple(shape), ttgl.tuple(permute)
```
**EN:** Inside function `_get_join_shape`, this return statement sends `(ttgl.tuple(shape), ttgl.tuple(permute))` back to the caller as the result of the current routine.
**CN:** 在函数 `_get_join_shape` 内部，这条返回语句把 `(ttgl.tuple(shape), ttgl.tuple(permute))` 作为当前过程的结果返回给调用方。

### Lines 148-149
```python
@jit
def pack(x, axis):
```
**EN:** At module scope, this header declares the function `pack(x, axis)`, which is responsible for pack. Decorators: jit.
**CN:** 在模块级作用域中，这段头部声明了函数 `pack(x, axis)`，它负责处理 pack 相关逻辑。 装饰器包括：jit。

### Lines 150-150
```python
    sp: ttgl.constexpr = _get_split_shape(x.shape, axis)
```
**EN:** Inside function `pack`, this assignment updates `sp` with `_get_split_shape(x.shape, axis)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `pack` 内部，这段赋值把 `_get_split_shape(x.shape, axis)` 写入 `sp`，为后续逻辑建立状态、别名或配置。

### Lines 151-151
```python
    x0, x1 = x.reshape(*sp[0]).permute(*sp[1]).split()
```
**EN:** Inside function `pack`, this assignment updates `(x0, x1)` with `x.reshape(*sp[0]).permute(*sp[1]).split()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `pack` 内部，这段赋值把 `x.reshape(*sp[0]).permute(*sp[1]).split()` 写入 `(x0, x1)`，为后续逻辑建立状态、别名或配置。

### Lines 152-152
```python
    return pack2(x0, x1)
```
**EN:** Inside function `pack`, this return statement sends `pack2(x0, x1)` back to the caller as the result of the current routine.
**CN:** 在函数 `pack` 内部，这条返回语句把 `pack2(x0, x1)` 作为当前过程的结果返回给调用方。

### Lines 155-156
```python
@jit
def unpack(x, axis):
```
**EN:** At module scope, this header declares the function `unpack(x, axis)`, which is responsible for unpack. Decorators: jit.
**CN:** 在模块级作用域中，这段头部声明了函数 `unpack(x, axis)`，它负责处理 unpack 相关逻辑。 装饰器包括：jit。

### Lines 157-157
```python
    shape: ttgl.constexpr = x.value.shape
```
**EN:** Inside function `unpack`, this assignment updates `shape` with `x.value.shape`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `unpack` 内部，这段赋值把 `x.value.shape` 写入 `shape`，为后续逻辑建立状态、别名或配置。

### Lines 158-158
```python
    sp: ttgl.constexpr = _get_join_shape(shape, axis)
```
**EN:** Inside function `unpack`, this assignment updates `sp` with `_get_join_shape(shape, axis)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `unpack` 内部，这段赋值把 `_get_join_shape(shape, axis)` 写入 `sp`，为后续逻辑建立状态、别名或配置。

### Lines 159-159
```python
    x0, x1 = unpack2(x)
```
**EN:** Inside function `unpack`, this assignment updates `(x0, x1)` with `unpack2(x)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `unpack` 内部，这段赋值把 `unpack2(x)` 写入 `(x0, x1)`，为后续逻辑建立状态、别名或配置。

### Lines 160-160
```python
    return ttgl.join(x0, x1).permute(*sp[1]).reshape(*sp[0])
```
**EN:** Inside function `unpack`, this return statement sends `ttgl.join(x0, x1).permute(*sp[1]).reshape(*sp[0])` back to the caller as the result of the current routine.
**CN:** 在函数 `unpack` 内部，这条返回语句把 `ttgl.join(x0, x1).permute(*sp[1]).reshape(*sp[0])` 作为当前过程的结果返回给调用方。

### Lines 163-164
```python
@jit
def full_like(x, fill_value):
```
**EN:** At module scope, this header declares the function `full_like(x, fill_value)`, which is responsible for full like. Decorators: jit.
**CN:** 在模块级作用域中，这段头部声明了函数 `full_like(x, fill_value)`，它负责处理 full like 相关逻辑。 装饰器包括：jit。

### Lines 165-165
```python
    ttgl.static_assert(fill_value.dtype == ttgl.float32, "fill_value must be a float32")
```
**EN:** Inside function `full_like`, this expression evaluates `ttgl.static_assert` mainly for its side effects or registration behavior.
**CN:** 在函数 `full_like` 内部，这条表达式计算 `ttgl.static_assert`，主要目的是触发副作用或完成注册行为。

### Lines 166-166
```python
    fill = stdlib.full_like(x.value, fill_value, dtype=ttgl.float32)
```
**EN:** Inside function `full_like`, this assignment updates `fill` with `stdlib.full_like(x.value, fill_value, dtype=ttgl.float32)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `full_like` 内部，这段赋值把 `stdlib.full_like(x.value, fill_value, dtype=ttgl.float32)` 写入 `fill`，为后续逻辑建立状态、别名或配置。

### Lines 167-167
```python
    return pack2(fill, fill)
```
**EN:** Inside function `full_like`, this return statement sends `pack2(fill, fill)` back to the caller as the result of the current routine.
**CN:** 在函数 `full_like` 内部，这条返回语句把 `pack2(fill, fill)` 作为当前过程的结果返回给调用方。

### Lines 170-171
```python
@jit
def fma(a, b, c):
```
**EN:** At module scope, this header declares the function `fma(a, b, c)`, which is responsible for fma. Decorators: jit.
**CN:** 在模块级作用域中，这段头部声明了函数 `fma(a, b, c)`，它负责处理 fma 相关逻辑。 装饰器包括：jit。

### Lines 172-172
```python
    return Float2Tensor(_fma_f32x2(a.value, b.value, c.value))
```
**EN:** Inside function `fma`, this return statement sends `Float2Tensor(_fma_f32x2(a.value, b.value, c.value))` back to the caller as the result of the current routine.
**CN:** 在函数 `fma` 内部，这条返回语句把 `Float2Tensor(_fma_f32x2(a.value, b.value, c.value))` 作为当前过程的结果返回给调用方。

## Key Concepts / 关键概念

- **EN:** Path theme: `python/triton/experimental/gluon/language/nvidia/blackwell` places this module in Triton's triton / experimental / gluon / language / nvidia / blackwell area.
  **CN:** 路径主题：`python/triton/experimental/gluon/language/nvidia/blackwell` 表明该模块位于 Triton 的 triton / experimental / gluon / language / nvidia / blackwell 领域。
- **EN:** Primary classes: `Float2Tensor`.
  **CN:** 主要类：`Float2Tensor`。
- **EN:** Primary functions: `_add_f32x2`, `_sub_f32x2`, `_mul_f32x2`, `_fma_f32x2`, `pack2`, `unpack2`, `_get_split_shape`, `_get_join_shape`, `pack`, `unpack`.
  **CN:** 主要函数：`_add_f32x2`, `_sub_f32x2`, `_mul_f32x2`, `_fma_f32x2`, `pack2`, `unpack2`, `_get_split_shape`, `_get_join_shape`, `pack`, `unpack`。
- **EN:** Frontend language model: this module shapes Triton language semantics, types, or helper intrinsics.
  **CN:** 前端语言模型：该模块塑造 Triton 语言语义、类型或辅助内建函数。
- **EN:** Experimental API: this module lives under `experimental`, so its interfaces may evolve quickly.
  **CN:** 实验性 API：该模块位于 `experimental` 下，其接口可能会快速演进。

## Dependencies / 依赖关系

- **EN:** Internal Triton modules: triton.experimental.gluon, triton.experimental.gluon.language, triton.experimental.gluon._runtime.
  **CN:** Triton 内部模块：triton.experimental.gluon, triton.experimental.gluon.language, triton.experimental.gluon._runtime。
