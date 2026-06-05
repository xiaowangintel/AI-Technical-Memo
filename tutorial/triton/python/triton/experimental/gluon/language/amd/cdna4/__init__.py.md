# __init__.py — Code Analysis / 代码分析

## Source / 来源

- **Path:** `./python/triton/experimental/gluon/language/amd/cdna4/__init__.py`
- **EN:** This package initializer at `./python/triton/experimental/gluon/language/amd/cdna4/__init__.py` wires together the public API for `cdna4` by re-exporting or organizing symbols.
- **CN:** 位于 `./python/triton/experimental/gluon/language/amd/cdna4/__init__.py` 的这个包初始化文件通过重新导出或整理符号，为 `cdna4` 组装公共 API。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1
```python
from triton.runtime.jit import constexpr_function
```
**EN:** At module scope, this block imports constexpr_function from `triton.runtime.jit` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `triton.runtime.jit` 导入 constexpr_function，把当前文件与周边 API 和辅助工具连接起来。

### Lines 2-2
```python
from triton._C.libtriton.gluon_ir import get_amd_mfma_scale_layout as _get_mfma_scale_layout
```
**EN:** At module scope, this block imports get_amd_mfma_scale_layout as _get_mfma_scale_layout from `triton._C.libtriton.gluon_ir` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `triton._C.libtriton.gluon_ir` 导入 get_amd_mfma_scale_layout as _get_mfma_scale_layout，把当前文件与周边 API 和辅助工具连接起来。

### Lines 4-4
```python
from ..._core import builtin, int8, uint8, _unwrap_if_constexpr
```
**EN:** At module scope, this block imports builtin, int8, uint8, _unwrap_if_constexpr from `..._core` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `..._core` 导入 builtin, int8, uint8, _unwrap_if_constexpr，把当前文件与周边 API 和辅助工具连接起来。

### Lines 5-5
```python
from ..._layouts import DotOperandLayout
```
**EN:** At module scope, this block imports DotOperandLayout from `..._layouts` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `..._layouts` 导入 DotOperandLayout，把当前文件与周边 API 和辅助工具连接起来。

### Lines 6-6
```python
from .._layouts import AMDMFMALayout
```
**EN:** At module scope, this block imports AMDMFMALayout from `.._layouts` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `.._layouts` 导入 AMDMFMALayout，把当前文件与周边 API 和辅助工具连接起来。

### Lines 7-7
```python
from .._ops import _mma_scaled, _scaled_upcast
```
**EN:** At module scope, this block imports _mma_scaled, _scaled_upcast from `.._ops` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `.._ops` 导入 _mma_scaled, _scaled_upcast，把当前文件与周边 API 和辅助工具连接起来。

### Lines 8-8
```python
from ..cdna3 import _buffer_atomic_rmw_impl, _convert_e8m0_scale_to_bf16
```
**EN:** At module scope, this block imports _buffer_atomic_rmw_impl, _convert_e8m0_scale_to_bf16 from `..cdna3` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `..cdna3` 导入 _buffer_atomic_rmw_impl, _convert_e8m0_scale_to_bf16，把当前文件与周边 API 和辅助工具连接起来。

### Lines 9-9
```python
from ..cdna3 import *  # NOQA: F403
```
**EN:** At module scope, this block imports * from `..cdna3` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `..cdna3` 导入 *，把当前文件与周边 API 和辅助工具连接起来。

### Lines 10-10
```python
from ..cdna3 import __all__ as __cdna3_all
```
**EN:** At module scope, this block imports __all__ as __cdna3_all from `..cdna3` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `..cdna3` 导入 __all__ as __cdna3_all，把当前文件与周边 API 和辅助工具连接起来。

### Lines 11-11
```python
from . import async_copy
```
**EN:** At module scope, this block imports async_copy from `.` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `.` 导入 async_copy，把当前文件与周边 API 和辅助工具连接起来。

### Lines 13-13
```python
__all__ = [*__cdna3_all, "async_copy", "mfma_scaled", "scaled_upcast", "get_mfma_scale_layout"]
```
**EN:** At module scope, this assignment updates `__all__` with `[*__cdna3_all, 'async_copy', 'mfma_scaled', 'scaled_upcast', 'get_mfma_scale_...`, establishing state, aliases, or configuration used later.
**CN:** 在模块级作用域中，这段赋值把 `[*__cdna3_all, 'async_copy', 'mfma_scaled', 'scaled_upcast', 'get_mfma_scale_...` 写入 `__all__`，为后续逻辑建立状态、别名或配置。

### Lines 16-17
```python
@builtin
def mfma_scaled(a, a_scale, a_format, b, b_scale, b_format, acc, _semantic=None):
```
**EN:** At module scope, this header declares the function `mfma_scaled(a, a_scale, a_format, b, b_scale, b_format, acc, _semantic)`, which is responsible for mfma scaled. Decorators: builtin. The docstring says: AMD Scaled MFMA operation.
**CN:** 在模块级作用域中，这段头部声明了函数 `mfma_scaled(a, a_scale, a_format, b, b_scale, b_format, acc, _semantic)`，它负责处理 mfma scaled 相关逻辑。 装饰器包括：builtin。 文档字符串说明：AMD Scaled MFMA operation.

### Lines 18-38
```python
    """
    AMD Scaled MFMA operation.

    ```
    c = a * a_scale @ b * b_scale + acc
    ```

    `a` and `b` use microscaling formats described in
    "OCP Microscaling Formats (MX) Specification":
    https://www.opencompute.org/documents/ocp-microscaling-formats-mx-v1-0-spec-final-pdf.
    Currently supported only on CDNA4 hardware.

    Args:
        a (tensor): The operand A to be multiplied.
        a_scale (Optional[tensor]): Scale factor for operand A.
        a_format (str): Format of the operand A. Available formats: `e2m1`, `e4m3`, `e5m2`.
        b (tensor): The operand B to be multiplied.
        b_scale (Optional[tensor]): Scale factor for operand B.
        b_format (str): Format of the operand B. Available formats: `e2m1`, `e4m3`, `e5m2`.
        acc (tensor): Accumulator tensor.
    """
```
**EN:** Inside function `mfma_scaled`, this docstring documents the surrounding scope. Summary: AMD Scaled MFMA operation.
**CN:** 在函数 `mfma_scaled` 内部，这段文档字符串用于说明当前作用域。摘要：AMD Scaled MFMA operation.

### Lines 39-39
```python
    layout = acc.type.layout
```
**EN:** Inside function `mfma_scaled`, this assignment updates `layout` with `acc.type.layout`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `mfma_scaled` 内部，这段赋值把 `acc.type.layout` 写入 `layout`，为后续逻辑建立状态、别名或配置。

### Lines 40-40
```python
    assert isinstance(layout, AMDMFMALayout), "Expected layout to be an instance of AMDMFMALayout"
```
**EN:** Inside function `mfma_scaled`, this assertion enforces `isinstance(layout, AMDMFMALayout)` so invalid states are caught early during execution.
**CN:** 在函数 `mfma_scaled` 内部，这条断言要求 `isinstance(layout, AMDMFMALayout)` 成立，从而在执行早期捕获非法状态。

### Lines 41-42
```python
    assert (isinstance(a.type.layout, DotOperandLayout) and a.type.layout.parent== layout), \
            "Expected lhs layout to be a DotOperandLayout with parent matching MFMA layout"
```
**EN:** Inside function `mfma_scaled`, this assertion enforces `isinstance(a.type.layout, DotOperandLayout) and a.type.layout.parent == layout` so invalid states are caught early during execution.
**CN:** 在函数 `mfma_scaled` 内部，这条断言要求 `isinstance(a.type.layout, DotOperandLayout) and a.type.layout.parent == layout` 成立，从而在执行早期捕获非法状态。

### Lines 43-44
```python
    assert (isinstance(b.type.layout, DotOperandLayout) and b.type.layout.parent == layout), \
            "Expected rhs layout to be a DotOperandLayout with parent matching MFMA layout"
```
**EN:** Inside function `mfma_scaled`, this assertion enforces `isinstance(b.type.layout, DotOperandLayout) and b.type.layout.parent == layout` so invalid states are caught early during execution.
**CN:** 在函数 `mfma_scaled` 内部，这条断言要求 `isinstance(b.type.layout, DotOperandLayout) and b.type.layout.parent == layout` 成立，从而在执行早期捕获非法状态。

### Lines 46-46
```python
    a_format = _unwrap_if_constexpr(a_format)
```
**EN:** Inside function `mfma_scaled`, this assignment updates `a_format` with `_unwrap_if_constexpr(a_format)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `mfma_scaled` 内部，这段赋值把 `_unwrap_if_constexpr(a_format)` 写入 `a_format`，为后续逻辑建立状态、别名或配置。

### Lines 47-47
```python
    b_format = _unwrap_if_constexpr(b_format)
```
**EN:** Inside function `mfma_scaled`, this assignment updates `b_format` with `_unwrap_if_constexpr(b_format)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `mfma_scaled` 内部，这段赋值把 `_unwrap_if_constexpr(b_format)` 写入 `b_format`，为后续逻辑建立状态、别名或配置。

### Lines 48-48
```python
    a_scale = _unwrap_if_constexpr(a_scale)
```
**EN:** Inside function `mfma_scaled`, this assignment updates `a_scale` with `_unwrap_if_constexpr(a_scale)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `mfma_scaled` 内部，这段赋值把 `_unwrap_if_constexpr(a_scale)` 写入 `a_scale`，为后续逻辑建立状态、别名或配置。

### Lines 49-49
```python
    b_scale = _unwrap_if_constexpr(b_scale)
```
**EN:** Inside function `mfma_scaled`, this assignment updates `b_scale` with `_unwrap_if_constexpr(b_scale)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `mfma_scaled` 内部，这段赋值把 `_unwrap_if_constexpr(b_scale)` 写入 `b_scale`，为后续逻辑建立状态、别名或配置。

### Lines 50-50
```python
    assert a_format in {"e2m1", "e4m3", "e5m2"}, f"Unsupported lhs_format: {a_format}"
```
**EN:** Inside function `mfma_scaled`, this assertion enforces `a_format in {'e2m1', 'e4m3', 'e5m2'}` so invalid states are caught early during execution.
**CN:** 在函数 `mfma_scaled` 内部，这条断言要求 `a_format in {'e2m1', 'e4m3', 'e5m2'}` 成立，从而在执行早期捕获非法状态。

### Lines 51-51
```python
    assert b_format in {"e2m1", "e4m3", "e5m2"}, f"Unsupported rhs_format: {b_format}"
```
**EN:** Inside function `mfma_scaled`, this assertion enforces `b_format in {'e2m1', 'e4m3', 'e5m2'}` so invalid states are caught early during execution.
**CN:** 在函数 `mfma_scaled` 内部，这条断言要求 `b_format in {'e2m1', 'e4m3', 'e5m2'}` 成立，从而在执行早期捕获非法状态。

### Lines 53-53
```python
    return _mma_scaled(a, a_scale, a_format, b, b_scale, b_format, acc, get_mfma_scale_layout, _semantic)
```
**EN:** Inside function `mfma_scaled`, this return statement sends `_mma_scaled(a, a_scale, a_format, b, b_scale, b_format, acc, get_mfma_scale_layout, _semantic)` back to the caller as the result of the current routine.
**CN:** 在函数 `mfma_scaled` 内部，这条返回语句把 `_mma_scaled(a, a_scale, a_format, b, b_scale, b_format, acc, get_mfma_scale_layout, _semantic)` 作为当前过程的结果返回给调用方。

### Lines 56-57
```python
@builtin
def scaled_upcast(src, scale, elem_type, axis=None, _semantic=None):
```
**EN:** At module scope, this header declares the function `scaled_upcast(src, scale, elem_type, axis, _semantic)`, which is responsible for scaled upcast. Decorators: builtin. The docstring says: Upcast an fp4 or fp8 tensor and fold raw E8M0 scale payload into the CDNA4 scaled-upcast op.
**CN:** 在模块级作用域中，这段头部声明了函数 `scaled_upcast(src, scale, elem_type, axis, _semantic)`，它负责处理 scaled upcast 相关逻辑。 装饰器包括：builtin。 文档字符串说明：Upcast an fp4 or fp8 tensor and fold raw E8M0 scale payload into the CDNA4 scaled-upcast op.

### Lines 58-67
```python
    """
    Upcast an fp4 or fp8 tensor and fold raw E8M0 scale payload into the
    CDNA4 scaled-upcast op.

    The scale tensor must use raw E8M0 payload in `int8` or `uint8`, and must
    already have the expanded output shape and scaled-upcast result layout.
    For fp4 inputs, that is the canonical unpacked layout implied by `src`
    and `axis`. `elem_type` must be `fp16` or `bf16`. CDNA4 converts those
    bytes to the internal `bf16` scale form expected by the AMD op.
    """
```
**EN:** Inside function `scaled_upcast`, this docstring documents the surrounding scope. Summary: Upcast an fp4 or fp8 tensor and fold raw E8M0 scale payload into the CDNA4 scaled-upcast op.
**CN:** 在函数 `scaled_upcast` 内部，这段文档字符串用于说明当前作用域。摘要：Upcast an fp4 or fp8 tensor and fold raw E8M0 scale payload into the CDNA4 scaled-upcast op.

### Lines 68-68
```python
    axis = _unwrap_if_constexpr(axis)
```
**EN:** Inside function `scaled_upcast`, this assignment updates `axis` with `_unwrap_if_constexpr(axis)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `scaled_upcast` 内部，这段赋值把 `_unwrap_if_constexpr(axis)` 写入 `axis`，为后续逻辑建立状态、别名或配置。

### Lines 69-69
```python
    elem_type = _unwrap_if_constexpr(elem_type)
```
**EN:** Inside function `scaled_upcast`, this assignment updates `elem_type` with `_unwrap_if_constexpr(elem_type)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `scaled_upcast` 内部，这段赋值把 `_unwrap_if_constexpr(elem_type)` 写入 `elem_type`，为后续逻辑建立状态、别名或配置。

### Lines 70-71
```python
    assert scale.dtype in (int8, uint8), \
        f"Expected scale to use raw E8M0 payload in int8/uint8 but got {scale.dtype}"
```
**EN:** Inside function `scaled_upcast`, this assertion enforces `scale.dtype in (int8, uint8)` so invalid states are caught early during execution.
**CN:** 在函数 `scaled_upcast` 内部，这条断言要求 `scale.dtype in (int8, uint8)` 成立，从而在执行早期捕获非法状态。

### Lines 72-72
```python
    scale = _convert_e8m0_scale_to_bf16(scale, _semantic=_semantic)
```
**EN:** Inside function `scaled_upcast`, this assignment updates `scale` with `_convert_e8m0_scale_to_bf16(scale, _semantic=_semantic)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `scaled_upcast` 内部，这段赋值把 `_convert_e8m0_scale_to_bf16(scale, _semantic=_semantic)` 写入 `scale`，为后续逻辑建立状态、别名或配置。

### Lines 73-73
```python
    return _scaled_upcast(src, scale, elem_type, axis, _semantic)
```
**EN:** Inside function `scaled_upcast`, this return statement sends `_scaled_upcast(src, scale, elem_type, axis, _semantic)` back to the caller as the result of the current routine.
**CN:** 在函数 `scaled_upcast` 内部，这条返回语句把 `_scaled_upcast(src, scale, elem_type, axis, _semantic)` 作为当前过程的结果返回给调用方。

### Lines 76-76
```python
def _get_mfma_scale_layout_impl(*args, **kwargs):
```
**EN:** At module scope, this header declares the function `_get_mfma_scale_layout_impl(*args, **kwargs)`, which is responsible for get mfma scale layout impl.
**CN:** 在模块级作用域中，这段头部声明了函数 `_get_mfma_scale_layout_impl(*args, **kwargs)`，它负责处理 get mfma scale layout impl 相关逻辑。

### Lines 77-77
```python
    return _get_mfma_scale_layout(*args, **kwargs)
```
**EN:** Inside function `_get_mfma_scale_layout_impl`, this return statement sends `_get_mfma_scale_layout(*args, **kwargs)` back to the caller as the result of the current routine.
**CN:** 在函数 `_get_mfma_scale_layout_impl` 内部，这条返回语句把 `_get_mfma_scale_layout(*args, **kwargs)` 作为当前过程的结果返回给调用方。

### Lines 80-80
```python
_get_mfma_scale_layout_impl.__triton_builtin__ = True
```
**EN:** At module scope, this assignment updates `_get_mfma_scale_layout_impl.__triton_builtin__` with `True`, establishing state, aliases, or configuration used later.
**CN:** 在模块级作用域中，这段赋值把 `True` 写入 `_get_mfma_scale_layout_impl.__triton_builtin__`，为后续逻辑建立状态、别名或配置。

### Lines 83-84
```python
@constexpr_function
def get_mfma_scale_layout(dot_operand_layout, shape, scale_factor=32):
```
**EN:** At module scope, this header declares the function `get_mfma_scale_layout(dot_operand_layout, shape, scale_factor)`, which is responsible for get mfma scale layout. Decorators: constexpr_function. The docstring says: Get the scale layout for MFMA scaled operands.
**CN:** 在模块级作用域中，这段头部声明了函数 `get_mfma_scale_layout(dot_operand_layout, shape, scale_factor)`，它负责处理 get mfma scale layout 相关逻辑。 装饰器包括：constexpr_function。 文档字符串说明：Get the scale layout for MFMA scaled operands.

### Lines 85-93
```python
    """ Get the scale layout for MFMA scaled operands.

    Args:
        dot_operand_layout (DotOperandLayout): The dot operand layout.
        shape (List[int]): The shape of the scale tensor.
        scale_factor (int): The scale factor.
    Return:
        layout (DistributedLinearLayout): The scale layout.
    """
```
**EN:** Inside function `get_mfma_scale_layout`, this docstring documents the surrounding scope. Summary: Get the scale layout for MFMA scaled operands.
**CN:** 在函数 `get_mfma_scale_layout` 内部，这段文档字符串用于说明当前作用域。摘要：Get the scale layout for MFMA scaled operands.

### Lines 94-94
```python
    assert scale_factor == 32, "Only scale factor 32 is supported for CDNA4 Scaled MFMA"
```
**EN:** Inside function `get_mfma_scale_layout`, this assertion enforces `scale_factor == 32` so invalid states are caught early during execution.
**CN:** 在函数 `get_mfma_scale_layout` 内部，这条断言要求 `scale_factor == 32` 成立，从而在执行早期捕获非法状态。

### Lines 95-95
```python
    op_idx = dot_operand_layout.operand_index
```
**EN:** Inside function `get_mfma_scale_layout`, this assignment updates `op_idx` with `dot_operand_layout.operand_index`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `get_mfma_scale_layout` 内部，这段赋值把 `dot_operand_layout.operand_index` 写入 `op_idx`，为后续逻辑建立状态、别名或配置。

### Lines 96-96
```python
    parent = dot_operand_layout.parent
```
**EN:** Inside function `get_mfma_scale_layout`, this assignment updates `parent` with `dot_operand_layout.parent`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `get_mfma_scale_layout` 内部，这段赋值把 `dot_operand_layout.parent` 写入 `parent`，为后续逻辑建立状态、别名或配置。

### Lines 97-97
```python
    assert isinstance(parent, AMDMFMALayout), "Expected parent to be an instance of AMDMFMALayout"
```
**EN:** Inside function `get_mfma_scale_layout`, this assertion enforces `isinstance(parent, AMDMFMALayout)` so invalid states are caught early during execution.
**CN:** 在函数 `get_mfma_scale_layout` 内部，这条断言要求 `isinstance(parent, AMDMFMALayout)` 成立，从而在执行早期捕获非法状态。

### Lines 98-98
```python
    mdim = parent.instr_shape[0]
```
**EN:** Inside function `get_mfma_scale_layout`, this assignment updates `mdim` with `parent.instr_shape[0]`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `get_mfma_scale_layout` 内部，这段赋值把 `parent.instr_shape[0]` 写入 `mdim`，为后续逻辑建立状态、别名或配置。

### Lines 99-99
```python
    tiles_per_warp = parent.tiles_per_warp
```
**EN:** Inside function `get_mfma_scale_layout`, this assignment updates `tiles_per_warp` with `parent.tiles_per_warp`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `get_mfma_scale_layout` 内部，这段赋值把 `parent.tiles_per_warp` 写入 `tiles_per_warp`，为后续逻辑建立状态、别名或配置。

### Lines 100-100
```python
    warps_per_cta = parent.warps_per_cta
```
**EN:** Inside function `get_mfma_scale_layout`, this assignment updates `warps_per_cta` with `parent.warps_per_cta`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `get_mfma_scale_layout` 内部，这段赋值把 `parent.warps_per_cta` 写入 `warps_per_cta`，为后续逻辑建立状态、别名或配置。

### Lines 101-101
```python
    return _get_mfma_scale_layout_impl(op_idx, shape, mdim, tiles_per_warp, warps_per_cta)
```
**EN:** Inside function `get_mfma_scale_layout`, this return statement sends `_get_mfma_scale_layout_impl(op_idx, shape, mdim, tiles_per_warp, warps_per_cta)` back to the caller as the result of the current routine.
**CN:** 在函数 `get_mfma_scale_layout` 内部，这条返回语句把 `_get_mfma_scale_layout_impl(op_idx, shape, mdim, tiles_per_warp, warps_per_cta)` 作为当前过程的结果返回给调用方。

### Lines 104-107
```python
"""
buffer_atomic_rmw of cnda4 shares the same signature and functionalities as cdna3.buffer_atomic_rmw.
The cdna4 version additionally supports `fadd` with `bf16`.
"""
```
**EN:** At module scope, this docstring documents the surrounding scope. Summary: buffer_atomic_rmw of cnda4 shares the same signature and functionalities as cdna3.buffer_atomic_rmw.
**CN:** 在模块级作用域中，这段文档字符串用于说明当前作用域。摘要：buffer_atomic_rmw of cnda4 shares the same signature and functionalities as cdna3.buffer_atomic_rmw.

### Lines 110-111
```python
@builtin
def buffer_atomic_max(ptr, offsets, value, mask=None, sem=None, scope=None, _semantic=None):
```
**EN:** At module scope, this header declares the function `buffer_atomic_max(ptr, offsets, value, mask, sem, scope, _semantic)`, which is responsible for buffer atomic max. Decorators: builtin.
**CN:** 在模块级作用域中，这段头部声明了函数 `buffer_atomic_max(ptr, offsets, value, mask, sem, scope, _semantic)`，它负责处理 buffer atomic max 相关逻辑。 装饰器包括：builtin。

### Lines 112-113
```python
    return _buffer_atomic_rmw_impl('max', ptr, offsets, value, "cdna4", mask=mask, sem=sem, scope=scope,
                                   _semantic=_semantic)
```
**EN:** Inside function `buffer_atomic_max`, this return statement sends `_buffer_atomic_rmw_impl('max', ptr, offsets, value, 'cdna4', mask=mask, sem=sem, scope=scope, _se...` back to the caller as the result of the current routine.
**CN:** 在函数 `buffer_atomic_max` 内部，这条返回语句把 `_buffer_atomic_rmw_impl('max', ptr, offsets, value, 'cdna4', mask=mask, sem=sem, scope=scope, _se...` 作为当前过程的结果返回给调用方。

### Lines 116-118
```python
@builtin
def buffer_atomic_min(ptr, offsets, value, mask=None, sem=None, scope=None, _semantic=None):
```
**EN:** At module scope, this header declares the function `buffer_atomic_min(ptr, offsets, value, mask, sem, scope, _semantic)`, which is responsible for buffer atomic min. Decorators: builtin.
**CN:** 在模块级作用域中，这段头部声明了函数 `buffer_atomic_min(ptr, offsets, value, mask, sem, scope, _semantic)`，它负责处理 buffer atomic min 相关逻辑。 装饰器包括：builtin。

### Lines 119-120
```python
    return _buffer_atomic_rmw_impl('min', ptr, offsets, value, "cdna4", mask=mask, sem=sem, scope=scope,
                                   _semantic=_semantic)
```
**EN:** Inside function `buffer_atomic_min`, this return statement sends `_buffer_atomic_rmw_impl('min', ptr, offsets, value, 'cdna4', mask=mask, sem=sem, scope=scope, _se...` back to the caller as the result of the current routine.
**CN:** 在函数 `buffer_atomic_min` 内部，这条返回语句把 `_buffer_atomic_rmw_impl('min', ptr, offsets, value, 'cdna4', mask=mask, sem=sem, scope=scope, _se...` 作为当前过程的结果返回给调用方。

### Lines 123-125
```python
@builtin
def buffer_atomic_add(ptr, offsets, value, mask=None, sem=None, scope=None, _semantic=None):
```
**EN:** At module scope, this header declares the function `buffer_atomic_add(ptr, offsets, value, mask, sem, scope, _semantic)`, which is responsible for buffer atomic add. Decorators: builtin.
**CN:** 在模块级作用域中，这段头部声明了函数 `buffer_atomic_add(ptr, offsets, value, mask, sem, scope, _semantic)`，它负责处理 buffer atomic add 相关逻辑。 装饰器包括：builtin。

### Lines 126-127
```python
    return _buffer_atomic_rmw_impl('add', ptr, offsets, value, "cdna4", mask=mask, sem=sem, scope=scope,
                                   _semantic=_semantic)
```
**EN:** Inside function `buffer_atomic_add`, this return statement sends `_buffer_atomic_rmw_impl('add', ptr, offsets, value, 'cdna4', mask=mask, sem=sem, scope=scope, _se...` back to the caller as the result of the current routine.
**CN:** 在函数 `buffer_atomic_add` 内部，这条返回语句把 `_buffer_atomic_rmw_impl('add', ptr, offsets, value, 'cdna4', mask=mask, sem=sem, scope=scope, _se...` 作为当前过程的结果返回给调用方。

### Lines 130-132
```python
@builtin
def buffer_atomic_and(ptr, offsets, value, mask=None, sem=None, scope=None, _semantic=None):
```
**EN:** At module scope, this header declares the function `buffer_atomic_and(ptr, offsets, value, mask, sem, scope, _semantic)`, which is responsible for buffer atomic and. Decorators: builtin.
**CN:** 在模块级作用域中，这段头部声明了函数 `buffer_atomic_and(ptr, offsets, value, mask, sem, scope, _semantic)`，它负责处理 buffer atomic and 相关逻辑。 装饰器包括：builtin。

### Lines 133-134
```python
    return _buffer_atomic_rmw_impl('and', ptr, offsets, value, "cdna4", mask=mask, sem=sem, scope=scope,
                                   _semantic=_semantic)
```
**EN:** Inside function `buffer_atomic_and`, this return statement sends `_buffer_atomic_rmw_impl('and', ptr, offsets, value, 'cdna4', mask=mask, sem=sem, scope=scope, _se...` back to the caller as the result of the current routine.
**CN:** 在函数 `buffer_atomic_and` 内部，这条返回语句把 `_buffer_atomic_rmw_impl('and', ptr, offsets, value, 'cdna4', mask=mask, sem=sem, scope=scope, _se...` 作为当前过程的结果返回给调用方。

### Lines 137-139
```python
@builtin
def buffer_atomic_or(ptr, offsets, value, mask=None, sem=None, scope=None, _semantic=None):
```
**EN:** At module scope, this header declares the function `buffer_atomic_or(ptr, offsets, value, mask, sem, scope, _semantic)`, which is responsible for buffer atomic or. Decorators: builtin.
**CN:** 在模块级作用域中，这段头部声明了函数 `buffer_atomic_or(ptr, offsets, value, mask, sem, scope, _semantic)`，它负责处理 buffer atomic or 相关逻辑。 装饰器包括：builtin。

### Lines 140-141
```python
    return _buffer_atomic_rmw_impl('or', ptr, offsets, value, "cdna4", mask=mask, sem=sem, scope=scope,
                                   _semantic=_semantic)
```
**EN:** Inside function `buffer_atomic_or`, this return statement sends `_buffer_atomic_rmw_impl('or', ptr, offsets, value, 'cdna4', mask=mask, sem=sem, scope=scope, _sem...` back to the caller as the result of the current routine.
**CN:** 在函数 `buffer_atomic_or` 内部，这条返回语句把 `_buffer_atomic_rmw_impl('or', ptr, offsets, value, 'cdna4', mask=mask, sem=sem, scope=scope, _sem...` 作为当前过程的结果返回给调用方。

### Lines 144-146
```python
@builtin
def buffer_atomic_xor(ptr, offsets, value, mask=None, sem=None, scope=None, _semantic=None):
```
**EN:** At module scope, this header declares the function `buffer_atomic_xor(ptr, offsets, value, mask, sem, scope, _semantic)`, which is responsible for buffer atomic xor. Decorators: builtin.
**CN:** 在模块级作用域中，这段头部声明了函数 `buffer_atomic_xor(ptr, offsets, value, mask, sem, scope, _semantic)`，它负责处理 buffer atomic xor 相关逻辑。 装饰器包括：builtin。

### Lines 147-148
```python
    return _buffer_atomic_rmw_impl('xor', ptr, offsets, value, "cdna4", mask=mask, sem=sem, scope=scope,
                                   _semantic=_semantic)
```
**EN:** Inside function `buffer_atomic_xor`, this return statement sends `_buffer_atomic_rmw_impl('xor', ptr, offsets, value, 'cdna4', mask=mask, sem=sem, scope=scope, _se...` back to the caller as the result of the current routine.
**CN:** 在函数 `buffer_atomic_xor` 内部，这条返回语句把 `_buffer_atomic_rmw_impl('xor', ptr, offsets, value, 'cdna4', mask=mask, sem=sem, scope=scope, _se...` 作为当前过程的结果返回给调用方。

### Lines 151-153
```python
@builtin
def buffer_atomic_xchg(ptr, offsets, value, mask=None, sem=None, scope=None, _semantic=None):
```
**EN:** At module scope, this header declares the function `buffer_atomic_xchg(ptr, offsets, value, mask, sem, scope, _semantic)`, which is responsible for buffer atomic xchg. Decorators: builtin.
**CN:** 在模块级作用域中，这段头部声明了函数 `buffer_atomic_xchg(ptr, offsets, value, mask, sem, scope, _semantic)`，它负责处理 buffer atomic xchg 相关逻辑。 装饰器包括：builtin。

### Lines 154-155
```python
    return _buffer_atomic_rmw_impl('xchg', ptr, offsets, value, "cdna4", mask=mask, sem=sem, scope=scope,
                                   _semantic=_semantic)
```
**EN:** Inside function `buffer_atomic_xchg`, this return statement sends `_buffer_atomic_rmw_impl('xchg', ptr, offsets, value, 'cdna4', mask=mask, sem=sem, scope=scope, _s...` back to the caller as the result of the current routine.
**CN:** 在函数 `buffer_atomic_xchg` 内部，这条返回语句把 `_buffer_atomic_rmw_impl('xchg', ptr, offsets, value, 'cdna4', mask=mask, sem=sem, scope=scope, _s...` 作为当前过程的结果返回给调用方。

## Key Concepts / 关键概念

- **EN:** Path theme: `python/triton/experimental/gluon/language/amd/cdna4` places this module in Triton's triton / experimental / gluon / language / amd / cdna4 area.
  **CN:** 路径主题：`python/triton/experimental/gluon/language/amd/cdna4` 表明该模块位于 Triton 的 triton / experimental / gluon / language / amd / cdna4 领域。
- **EN:** Primary functions: `mfma_scaled`, `scaled_upcast`, `_get_mfma_scale_layout_impl`, `get_mfma_scale_layout`, `buffer_atomic_max`, `buffer_atomic_min`, `buffer_atomic_add`, `buffer_atomic_and`, `buffer_atomic_or`, `buffer_atomic_xor`.
  **CN:** 主要函数：`mfma_scaled`, `scaled_upcast`, `_get_mfma_scale_layout_impl`, `get_mfma_scale_layout`, `buffer_atomic_max`, `buffer_atomic_min`, `buffer_atomic_add`, `buffer_atomic_and`, `buffer_atomic_or`, `buffer_atomic_xor`。
- **EN:** API aggregation: `__init__.py` files usually control what sibling modules expose to users.
  **CN:** `__init__.py` 通常承担 API 聚合职责，决定同级模块向用户暴露什么。
- **EN:** Frontend language model: this module shapes Triton language semantics, types, or helper intrinsics.
  **CN:** 前端语言模型：该模块塑造 Triton 语言语义、类型或辅助内建函数。
- **EN:** Experimental API: this module lives under `experimental`, so its interfaces may evolve quickly.
  **CN:** 实验性 API：该模块位于 `experimental` 下，其接口可能会快速演进。

## Dependencies / 依赖关系

- **EN:** Internal Triton modules: triton.runtime.jit, ..._core, ..._layouts, .._layouts, .._ops, ..cdna3, ..
  **CN:** Triton 内部模块：triton.runtime.jit, ..._core, ..._layouts, .._layouts, .._ops, ..cdna3, .。
- **EN:** Native/C-extension bindings: triton._C.libtriton.gluon_ir.
  **CN:** 原生/C 扩展绑定：triton._C.libtriton.gluon_ir。
