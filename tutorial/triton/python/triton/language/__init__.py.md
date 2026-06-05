# __init__.py — Code Analysis / 代码分析

## Source / 来源

- **Path:** `./python/triton/language/__init__.py`
- **EN:** This package initializer at `./python/triton/language/__init__.py` wires together the public API for `language` by re-exporting or organizing symbols.
- **CN:** 位于 `./python/triton/language/__init__.py` 的这个包初始化文件通过重新导出或整理符号，为 `language` 组装公共 API。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1
```python
"""isort:skip_file"""
```
**EN:** At module scope, this docstring documents the surrounding scope. Summary: isort:skip_file
**CN:** 在模块级作用域中，这段文档字符串用于说明当前作用域。摘要：isort:skip_file

### Lines 2-2
```python
# Import order is significant here.
```
**EN:** At module scope, this comment block explains the surrounding logic or records implementation notes.
**CN:** 在模块级作用域中，这段注释用于说明周围逻辑，或记录实现细节。

### Lines 4-4
```python
from . import math
```
**EN:** At module scope, this block imports math from `.` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `.` 导入 math，把当前文件与周边 API 和辅助工具连接起来。

### Lines 5-5
```python
from . import extra
```
**EN:** At module scope, this block imports extra from `.` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `.` 导入 extra，把当前文件与周边 API 和辅助工具连接起来。

### Lines 6-30
```python
from .standard import (
    argmax,
    argmin,
    bitonic_merge,
    cdiv,
    cumprod,
    cumsum,
    flip,
    interleave,
    max,
    min,
    ravel,
    reduce_or,
    sigmoid,
    softmax,
    sort,
    squeeze,
    sum,
    swizzle2d,
    topk,
    unsqueeze,
    xor_sum,
    zeros,
    zeros_like,
)
```
**EN:** At module scope, this block imports argmax, argmin, bitonic_merge, cdiv, cumprod, cumsum, flip, interleave, and 15 more from `.standard` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `.standard` 导入 argmax, argmin, bitonic_merge, cdiv, cumprod, cumsum, flip, interleave, and 15 more，把当前文件与周边 API 和辅助工具连接起来。

### Lines 31-125
```python
from .core import (
    PropagateNan,
    TRITON_MAX_TENSOR_NUMEL,
    aggregate_replace,
    load_tensor_descriptor,
    store_tensor_descriptor,
    make_tensor_descriptor,
    tensor_descriptor,
    tensor_descriptor_type,
    add,
    advance,
    arange,
    associative_scan,
    assume,
    atomic_add,
    atomic_and,
    atomic_cas,
    atomic_max,
    atomic_min,
    atomic_or,
    atomic_xchg,
    atomic_xor,
    bfloat16,
    block_type,
    broadcast,
    broadcast_to,
    cat,
    cast,
    clamp,
    condition,
    const,
    constexpr,
    constexpr_type,
    debug_barrier,
    device_assert,
    device_print,
    dot,
    dot_scaled,
    dtype,
    expand_dims,
    float16,
    float32,
    float64,
    float8e4b15,
    float8e4nv,
    float8e4b8,
    float8e5,
    float8e5b16,
    full,
    gather,
    histogram,
    inline_asm_elementwise,
    int1,
    int16,
    int32,
    int64,
    int8,
    join,
    load,
    make_block_ptr,
    map_elementwise,
    max_constancy,
    max_contiguous,
    maximum,
    minimum,
    mul,
    multiple_of,
    num_programs,
    permute,
    pi32_t,
    pointer_type,
    program_id,
    range,
    reduce,
    reshape,
    slice,
    split,
    static_assert,
    static_print,
    static_range,
    store,
    sub,
    tensor,
    to_tensor,
    trans,
    tuple,
    tuple_type,
    uint16,
    uint32,
    uint64,
    uint8,
    view,
    void,
    where,
)
```
**EN:** At module scope, this block imports PropagateNan, TRITON_MAX_TENSOR_NUMEL, aggregate_replace, load_tensor_descriptor, store_tensor_descriptor, make_tensor_descriptor, tensor_descriptor, tensor_descriptor_type, and 85 more from `.core` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `.core` 导入 PropagateNan, TRITON_MAX_TENSOR_NUMEL, aggregate_replace, load_tensor_descriptor, store_tensor_descriptor, make_tensor_descriptor, tensor_descriptor, tensor_descriptor_type, and 85 more，把当前文件与周边 API 和辅助工具连接起来。

### Lines 126-127
```python
from .math import (umulhi, exp, exp2, fma, log, log2, cos, rsqrt, sin, sqrt, sqrt_rn, abs, fdiv, div_rn, erf, floor,
                   ceil)
```
**EN:** At module scope, this block imports umulhi, exp, exp2, fma, log, log2, cos, rsqrt, and 9 more from `.math` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `.math` 导入 umulhi, exp, exp2, fma, log, log2, cos, rsqrt, and 9 more，把当前文件与周边 API 和辅助工具连接起来。

### Lines 128-139
```python
from .random import (
    pair_uniform_to_normal,
    philox,
    philox_impl,
    rand,
    rand4x,
    randint,
    randint4x,
    randn,
    randn4x,
    uint_to_uniform_float,
)
```
**EN:** At module scope, this block imports pair_uniform_to_normal, philox, philox_impl, rand, rand4x, randint, randint4x, randn, and 2 more from `.random` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `.random` 导入 pair_uniform_to_normal, philox, philox_impl, rand, rand4x, randint, randint4x, randn, and 2 more，把当前文件与周边 API 和辅助工具连接起来。

### Lines 140-140
```python
from . import target_info
```
**EN:** At module scope, this block imports target_info from `.` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `.` 导入 target_info，把当前文件与周边 API 和辅助工具连接起来。

### Lines 142-286
```python
__all__ = [
    "PropagateNan",
    "TRITON_MAX_TENSOR_NUMEL",
    "load_tensor_descriptor",
    "store_tensor_descriptor",
    "make_tensor_descriptor",
    "tensor_descriptor",
    "abs",
    "add",
    "advance",
    "arange",
    "argmax",
    "argmin",
    "associative_scan",
    "assume",
    "atomic_add",
    "atomic_and",
    "atomic_cas",
    "atomic_max",
    "atomic_min",
    "atomic_or",
    "atomic_xchg",
    "atomic_xor",
    "bfloat16",
    "bitonic_merge",
    "block_type",
    "broadcast",
    "broadcast_to",
    "cat",
    "cast",
    "cdiv",
    "ceil",
    "clamp",
    "condition",
    "const",
    "constexpr",
    "constexpr_type",
    "cos",
    "cumprod",
    "cumsum",
    "debug_barrier",
    "device_assert",
    "device_print",
    "div_rn",
    "dot",
    "dot_scaled",
    "dtype",
    "erf",
    "exp",
    "exp2",
    "expand_dims",
    "extra",
    "fdiv",
    "flip",
    "float16",
    "float32",
    "float64",
    "float8e4b15",
    "float8e4nv",
    "float8e4b8",
    "float8e5",
    "float8e5b16",
    "floor",
    "fma",
    "full",
    "gather",
    "histogram",
    "inline_asm_elementwise",
    "interleave",
    "int1",
    "int16",
    "int32",
    "int64",
    "int8",
    "join",
    "load",
    "log",
    "log2",
    "make_block_ptr",
    "map_elementwise",
    "math",
    "max",
    "max_constancy",
    "max_contiguous",
    "maximum",
    "min",
    "minimum",
    "mul",
    "multiple_of",
    "num_programs",
    "pair_uniform_to_normal",
    "permute",
    "philox",
    "philox_impl",
    "pi32_t",
    "pointer_type",
    "program_id",
    "rand",
    "rand4x",
    "randint",
    "randint4x",
    "randn",
    "randn4x",
    "range",
    "ravel",
    "reduce",
    "reduce_or",
    "reshape",
    "rsqrt",
    "slice",
    "sigmoid",
    "sin",
    "softmax",
    "sort",
    "split",
    "sqrt",
    "sqrt_rn",
    "squeeze",
    "static_assert",
    "static_print",
    "static_range",
    "store",
    "sub",
    "sum",
    "swizzle2d",
    "target_info",
    "tensor",
    "topk",
    "to_tensor",
    "trans",
    "tuple",
    "uint16",
    "uint32",
    "uint64",
    "uint8",
    "uint_to_uniform_float",
    "umulhi",
    "unsqueeze",
    "view",
    "void",
    "where",
    "xor_sum",
    "zeros",
    "zeros_like",
]
```
**EN:** At module scope, this assignment updates `__all__` with `['PropagateNan', 'TRITON_MAX_TENSOR_NUMEL', 'load_tensor_descriptor', 'store_...`, establishing state, aliases, or configuration used later.
**CN:** 在模块级作用域中，这段赋值把 `['PropagateNan', 'TRITON_MAX_TENSOR_NUMEL', 'load_tensor_descriptor', 'store_...` 写入 `__all__`，为后续逻辑建立状态、别名或配置。

### Lines 289-289
```python
def str_to_ty(name, c):
```
**EN:** At module scope, this header declares the function `str_to_ty(name, c)`, which is responsible for str to ty.
**CN:** 在模块级作用域中，这段头部声明了函数 `str_to_ty(name, c)`，它负责处理 str to ty 相关逻辑。

### Lines 290-290
```python
    from builtins import tuple
```
**EN:** Inside function `str_to_ty`, this block imports tuple from `builtins` to connect this file with nearby APIs and helpers.
**CN:** 在函数 `str_to_ty` 内部，这段代码从 `builtins` 导入 tuple，把当前文件与周边 API 和辅助工具连接起来。

### Lines 292-294
```python
    if isinstance(name, tuple):
        fields = type(name).__dict__.get("_fields", None)
        return tuple_type([str_to_ty(x, c) for x in name], fields)
```
**EN:** Inside function `str_to_ty`, this conditional checks `isinstance(name, tuple)` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `str_to_ty` 内部，这段条件语句检查 `isinstance(name, tuple)`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 296-303
```python
    if name[0] == "*":
        name = name[1:]
        const = False
        if name[0] == "k":
            name = name[1:]
            const = True
        ty = str_to_ty(name, c)
        return pointer_type(element_ty=ty, const=const)
```
**EN:** Inside function `str_to_ty`, this conditional checks `name[0] == '*'` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `str_to_ty` 内部，这段条件语句检查 `name[0] == '*'`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 305-344
```python
    if name.startswith("tensordesc"):
        # Determine mode from type name: tensordesc_im2col vs tensordesc
        is_im2col = name.startswith("tensordesc_im2col")

        inner = name.split("<")[1].rstrip(">")
        dtype, rest = inner.split("[", maxsplit=1)
        block_shape, rest = rest.split("]", maxsplit=1)
        block_shape = [int(s.strip()) for s in block_shape.rstrip("]").split(",")]
        # For im2col, parse optional input_rank=N (e.g., ",input_rank=4,layout")
        tensor_rank = None
        import re as _re
        rank_match = _re.search(r",input_rank=(\d+)", rest)
        if rank_match:
            tensor_rank = int(rank_match.group(1))
            rest = rest[:rank_match.start()] + rest[rank_match.end():]
        layout_str = rest.lstrip(",")
        is_gluon = len(layout_str)
        dtype = str_to_ty(dtype, None)
        # For im2col with tensor_rank, use it for shape/stride types; otherwise use block_shape ndim
        ndim = tensor_rank if (is_im2col and tensor_rank is not None) else len(block_shape)
        shape_type = tuple_type([int32] * ndim)
        # FIXME: Last dim stride should be constexpr(1)
        stride_type = tuple_type(([int64] * ndim))
        block = block_type(dtype, block_shape)
        if is_gluon:
            from triton.experimental.gluon.language._layouts import NVMMASharedLayout, PaddedSharedLayout, SwizzledSharedLayout
            from triton.experimental.gluon.language.nvidia.hopper.tma import tensor_descriptor_type as nvidia_tensor_descriptor_type
            from triton.experimental.gluon.language.nvidia.hopper.tma import tensor_descriptor_im2col_type as nvidia_tensor_descriptor_im2col_type
            from triton.experimental.gluon.language.amd.gfx1250.tdm import tensor_descriptor_type as amd_tensor_descriptor_type
            layout = eval(
                layout_str,
                dict(NVMMASharedLayout=NVMMASharedLayout, PaddedSharedLayout=PaddedSharedLayout,
                     SwizzledSharedLayout=SwizzledSharedLayout))
            if isinstance(layout, NVMMASharedLayout):
                if is_im2col:
                    return nvidia_tensor_descriptor_im2col_type(block, shape_type, stride_type, layout)
                return nvidia_tensor_descriptor_type(block, shape_type, stride_type, layout)
            else:
                return amd_tensor_descriptor_type(block, shape_type, stride_type, layout)
        return tensor_descriptor_type(block, shape_type, stride_type)
```
**EN:** Inside function `str_to_ty`, this conditional checks `name.startswith('tensordesc')` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `str_to_ty` 内部，这段条件语句检查 `name.startswith('tensordesc')`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 346-347
```python
    if name.startswith("constexpr"):
        return constexpr_type(c)
```
**EN:** Inside function `str_to_ty`, this conditional checks `name.startswith('constexpr')` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `str_to_ty` 内部，这段条件语句检查 `name.startswith('constexpr')`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 349-370
```python
    tys = {
        "fp8e4nv": float8e4nv,
        "fp8e4b8": float8e4b8,
        "fp8e5": float8e5,
        "fp8e5b16": float8e5b16,
        "fp8e4b15": float8e4b15,
        "fp16": float16,
        "bf16": bfloat16,
        "fp32": float32,
        "fp64": float64,
        "i1": int1,
        "i8": int8,
        "i16": int16,
        "i32": int32,
        "i64": int64,
        "u1": int1,
        "u8": uint8,
        "u16": uint16,
        "u32": uint32,
        "u64": uint64,
        "B": int1,
    }
```
**EN:** Inside function `str_to_ty`, this assignment updates `tys` with `{'fp8e4nv': float8e4nv, 'fp8e4b8': float8e4b8, 'fp8e5': float8e5, 'fp8e5b16':...`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `str_to_ty` 内部，这段赋值把 `{'fp8e4nv': float8e4nv, 'fp8e4b8': float8e4b8, 'fp8e5': float8e5, 'fp8e5b16':...` 写入 `tys`，为后续逻辑建立状态、别名或配置。

### Lines 371-371
```python
    return tys[name]
```
**EN:** Inside function `str_to_ty`, this return statement sends `tys[name]` back to the caller as the result of the current routine.
**CN:** 在函数 `str_to_ty` 内部，这条返回语句把 `tys[name]` 作为当前过程的结果返回给调用方。

## Key Concepts / 关键概念

- **EN:** Path theme: `python/triton/language` places this module in Triton's triton / language area.
  **CN:** 路径主题：`python/triton/language` 表明该模块位于 Triton 的 triton / language 领域。
- **EN:** Primary functions: `str_to_ty`.
  **CN:** 主要函数：`str_to_ty`。
- **EN:** API aggregation: `__init__.py` files usually control what sibling modules expose to users.
  **CN:** `__init__.py` 通常承担 API 聚合职责，决定同级模块向用户暴露什么。
- **EN:** Frontend language model: this module shapes Triton language semantics, types, or helper intrinsics.
  **CN:** 前端语言模型：该模块塑造 Triton 语言语义、类型或辅助内建函数。

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: builtins, re.
  **CN:** 标准库依赖：builtins, re。
- **EN:** Internal Triton modules: ., .standard, .core, .math, .random, triton.experimental.gluon.language._layouts, triton.experimental.gluon.language.nvidia.hopper.tma, triton.experimental.gluon.language.amd.gfx1250.tdm.
  **CN:** Triton 内部模块：., .standard, .core, .math, .random, triton.experimental.gluon.language._layouts, triton.experimental.gluon.language.nvidia.hopper.tma, triton.experimental.gluon.language.amd.gfx1250.tdm。
