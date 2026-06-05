# async_copy.py — Code Analysis / 代码分析

## Source / 来源

- **Path:** `./python/triton/experimental/gluon/language/amd/cdna4/async_copy.py`
- **EN:** This source file at `./python/triton/experimental/gluon/language/amd/cdna4/async_copy.py` defines the main symbols `global_load_to_shared`, `buffer_load_to_shared`, `commit_group`, `wait_group` for the surrounding Triton subsystem.
- **CN:** 位于 `./python/triton/experimental/gluon/language/amd/cdna4/async_copy.py` 的这个源文件为周边 Triton 子系统定义了主要符号 `global_load_to_shared`, `buffer_load_to_shared`, `commit_group`, `wait_group`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1
```python
from ..._core import ir, builtin, _unwrap_if_constexpr
```
**EN:** At module scope, this block imports ir, builtin, _unwrap_if_constexpr from `..._core` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `..._core` 导入 ir, builtin, _unwrap_if_constexpr，把当前文件与周边 API 和辅助工具连接起来。

### Lines 2-2
```python
from ..._semantic import _check
```
**EN:** At module scope, this block imports _check from `..._semantic` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `..._semantic` 导入 _check，把当前文件与周边 API 和辅助工具连接起来。

### Lines 3-3
```python
from ..._layouts import DistributedLayout
```
**EN:** At module scope, this block imports DistributedLayout from `..._layouts` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `..._layouts` 导入 DistributedLayout，把当前文件与周边 API 和辅助工具连接起来。

### Lines 4-4
```python
from ..cdna3 import _verify_buffer_ops
```
**EN:** At module scope, this block imports _verify_buffer_ops from `..cdna3` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `..cdna3` 导入 _verify_buffer_ops，把当前文件与周边 API 和辅助工具连接起来。

### Lines 6-12
```python
__all__ = [
    "global_load_to_shared",
    "buffer_load_to_shared",
    "commit_group",
    "wait_group",
    "load_shared_relaxed",
]
```
**EN:** At module scope, this assignment updates `__all__` with `['global_load_to_shared', 'buffer_load_to_shared', 'commit_group', 'wait_grou...`, establishing state, aliases, or configuration used later.
**CN:** 在模块级作用域中，这段赋值把 `['global_load_to_shared', 'buffer_load_to_shared', 'commit_group', 'wait_grou...` 写入 `__all__`，为后续逻辑建立状态、别名或配置。

### Lines 15-16
```python
@builtin
def global_load_to_shared(dest, ptr, mask=None, other=None, cache_modifier="", _semantic=None):
```
**EN:** At module scope, this header declares the function `global_load_to_shared(dest, ptr, mask, other, cache_modifier, _semantic)`, which is responsible for global load to shared. Decorators: builtin. The docstring says: AMD global load to shared operation.
**CN:** 在模块级作用域中，这段头部声明了函数 `global_load_to_shared(dest, ptr, mask, other, cache_modifier, _semantic)`，它负责处理 global load to shared 相关逻辑。 装饰器包括：builtin。 文档字符串说明：AMD global load to shared operation.

### Lines 17-47
```python
    """
    AMD global load to shared operation. This operation loads data directly
    from global memory to shared memory without going through registers. It
    happens asynchronously and requires a subsequent `async_wait` to ensure the
    data is available in shared memory. Note that this operation does still
    complete in order with ttgl.loads/stores or buffer_loads/stores on CDNA4,
    so interleaving with them will hurt performance.

    Compared to `buffer_load_to_shared`, it requires a tensor pointer which
    supports 64-bit indexing range for each thread in a block, which gives more
    flexibility, but at the cost of higher register pressure and no hardware
    out-of-bound masking support. Prefer to use `buffer_load_to_shared` when
    possible for better performance.

    The underlying hardware instruction uses separate registers for global
    memory address for each thread but the same register for local memory
    address for the whole warp. Therefore, while using this operation
    the following conditions must be met or lowering to LLVM will fail:

    - For the `ptr` layout, size per thread * bits per element must be 128 or 32.
      To get ideal performance, it is recommended to use 128 bits per element.
    - Writes to `dest` must be coalesced.
    - If `dest` is swizzled, it only can be swizzled within warp boundary.

    Args:
        dest (shared_memory_descriptor): Destination shared memory descriptor.
        ptr (pointer tensor): Tensor of pointers to global memory to load from.
        mask (tensor, optional): Mask tensor for predicated loads. Defaults to None.
        other (tensor or scalar, optional): Tensor or scalar providing default values for masked elements. Defaults to None.
        cache_modifier (str): Cache modifier specifier. Defaults to "".
    """
```
**EN:** Inside function `global_load_to_shared`, this docstring documents the surrounding scope. Summary: AMD global load to shared operation.
**CN:** 在函数 `global_load_to_shared` 内部，这段文档字符串用于说明当前作用域。摘要：AMD global load to shared operation.

### Lines 48-48
```python
    _check(ptr.type.is_block(), lambda: "expected ptr to be a tensor")
```
**EN:** Inside function `global_load_to_shared`, this expression evaluates `_check` mainly for its side effects or registration behavior.
**CN:** 在函数 `global_load_to_shared` 内部，这条表达式计算 `_check`，主要目的是触发副作用或完成注册行为。

### Lines 49-49
```python
    _check(isinstance(ptr.type.layout, DistributedLayout), lambda: "expected ptr type layout to be a DistributedLayout")
```
**EN:** Inside function `global_load_to_shared`, this expression evaluates `_check` mainly for its side effects or registration behavior.
**CN:** 在函数 `global_load_to_shared` 内部，这条表达式计算 `_check`，主要目的是触发副作用或完成注册行为。

### Lines 50-52
```python
    _check(
        dest.shape == ptr.shape, lambda:
        f"expected dest shape to match pointer shape but got dest.shape = {dest.shape}, pointer.shape = {ptr.shape}")
```
**EN:** Inside function `global_load_to_shared`, this expression evaluates `_check` mainly for its side effects or registration behavior.
**CN:** 在函数 `global_load_to_shared` 内部，这条表达式计算 `_check`，主要目的是触发副作用或完成注册行为。

### Lines 54-54
```python
    mask = _unwrap_if_constexpr(mask)
```
**EN:** Inside function `global_load_to_shared`, this assignment updates `mask` with `_unwrap_if_constexpr(mask)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `global_load_to_shared` 内部，这段赋值把 `_unwrap_if_constexpr(mask)` 写入 `mask`，为后续逻辑建立状态、别名或配置。

### Lines 55-56
```python
    if mask is not None:
        ptr, mask = _semantic.broadcast_impl_value(ptr, mask)
```
**EN:** Inside function `global_load_to_shared`, this conditional checks `mask is not None` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `global_load_to_shared` 内部，这段条件语句检查 `mask is not None`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 57-57
```python
    other = _unwrap_if_constexpr(other)
```
**EN:** Inside function `global_load_to_shared`, this assignment updates `other` with `_unwrap_if_constexpr(other)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `global_load_to_shared` 内部，这段赋值把 `_unwrap_if_constexpr(other)` 写入 `other`，为后续逻辑建立状态、别名或配置。

### Lines 58-61
```python
    if other is not None:
        other = _semantic.to_tensor(other)
        other = _semantic.cast(other, ptr.dtype.element_ty)
        ptr, other = _semantic.broadcast_impl_value(ptr, other)
```
**EN:** Inside function `global_load_to_shared`, this conditional checks `other is not None` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `global_load_to_shared` 内部，这段条件语句检查 `other is not None`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 63-63
```python
    cache_modifier = _semantic._str_to_load_cache_modifier(cache_modifier)
```
**EN:** Inside function `global_load_to_shared`, this assignment updates `cache_modifier` with `_semantic._str_to_load_cache_modifier(cache_modifier)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `global_load_to_shared` 内部，这段赋值把 `_semantic._str_to_load_cache_modifier(cache_modifier)` 写入 `cache_modifier`，为后续逻辑建立状态、别名或配置。

### Lines 64-64
```python
    mask_handle = mask.handle if mask is not None else ir.value()
```
**EN:** Inside function `global_load_to_shared`, this assignment updates `mask_handle` with `mask.handle if mask is not None else ir.value()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `global_load_to_shared` 内部，这段赋值把 `mask.handle if mask is not None else ir.value()` 写入 `mask_handle`，为后续逻辑建立状态、别名或配置。

### Lines 65-65
```python
    other_handle = other.handle if other is not None else ir.value()
```
**EN:** Inside function `global_load_to_shared`, this assignment updates `other_handle` with `other.handle if other is not None else ir.value()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `global_load_to_shared` 内部，这段赋值把 `other.handle if other is not None else ir.value()` 写入 `other_handle`，为后续逻辑建立状态、别名或配置。

### Lines 66-67
```python
    _semantic.builder.create_async_copy_global_to_local(dest.handle, ptr.handle, mask_handle, other_handle,
                                                        cache_modifier, ir.EVICTION_POLICY.NORMAL, False)
```
**EN:** Inside function `global_load_to_shared`, this expression evaluates `_semantic.builder.create_async_copy_global_to_local` mainly for its side effects or registration behavior.
**CN:** 在函数 `global_load_to_shared` 内部，这条表达式计算 `_semantic.builder.create_async_copy_global_to_local`，主要目的是触发副作用或完成注册行为。

### Lines 70-71
```python
@builtin
def buffer_load_to_shared(dest, ptr, offsets, mask=None, other=None, cache_modifier="", _semantic=None):
```
**EN:** At module scope, this header declares the function `buffer_load_to_shared(dest, ptr, offsets, mask, other, cache_modifier, _semantic)`, which is responsible for buffer load to shared. Decorators: builtin. The docstring says: AMD buffer load to shared operation.
**CN:** 在模块级作用域中，这段头部声明了函数 `buffer_load_to_shared(dest, ptr, offsets, mask, other, cache_modifier, _semantic)`，它负责处理 buffer load to shared 相关逻辑。 装饰器包括：builtin。 文档字符串说明：AMD buffer load to shared operation.

### Lines 72-103
```python
    """
    AMD buffer load to shared operation. Buffer load is similar to global load
    but it accesses global memory via a scalar base pointer and a tensor of
    32-bit offsets instead of a tensor of pointers. This operation loads data
    directly from global memory to shared memory without going through
    registers. It happens asynchronously and requires a subsequent `async_wait`
    to ensure thedata is available in shared memory. Note that this operation
    does still complete in order with ttgl.loads/stores or buffer_loads/stores
    on CDNA4, so interleaving with them will hurt performance.

    Compared to `global_load_to_shared`, it has better performance and also
    supports hardware out-of-bound masking. But it strictly requires a
    32-bit offset instead of a 64-bit tensor pointer.

    The underlying hardware instruction uses separate registers for global
    memory address for each thread but the same register for local memory
    address for the whole warp. Therefore, while using this operation
    the following conditions must be met or lowering to LLVM will fail:

    - For the `offsets` layout, size per thread * bits per element must be 128 or 32.
      To get ideal performance, it is recommended to use 128 bits per element.
    - Writes to `dest` must be coalesced.
    - If `dest` is swizzled, it only can be swizzled within warp boundary.

    Args:
        dest (shared_memory_descriptor): Destination shared memory descriptor.
        ptr (pointer to scalar): Global memory scalar base pointer to load from.
        offsets (tensor): Offsets tensor for the load operation.
        mask (tensor, optional): Mask tensor for predicated loads. Defaults to None.
        other (tensor or scalar, optional): Tensor or scalar providing default values for masked elements. Defaults to None.
        cache_modifier (str): Cache modifier specifier. Defaults to "".
    """
```
**EN:** Inside function `buffer_load_to_shared`, this docstring documents the surrounding scope. Summary: AMD buffer load to shared operation.
**CN:** 在函数 `buffer_load_to_shared` 内部，这段文档字符串用于说明当前作用域。摘要：AMD buffer load to shared operation.

### Lines 104-105
```python
    _check(isinstance(offsets.type.layout, DistributedLayout),
           lambda: "expected offsets type layout to be a DistributedLayout")
```
**EN:** Inside function `buffer_load_to_shared`, this expression evaluates `_check` mainly for its side effects or registration behavior.
**CN:** 在函数 `buffer_load_to_shared` 内部，这条表达式计算 `_check`，主要目的是触发副作用或完成注册行为。

### Lines 106-106
```python
    _verify_buffer_ops(ptr, offsets, mask, other)
```
**EN:** Inside function `buffer_load_to_shared`, this expression evaluates `_verify_buffer_ops` mainly for its side effects or registration behavior.
**CN:** 在函数 `buffer_load_to_shared` 内部，这条表达式计算 `_verify_buffer_ops`，主要目的是触发副作用或完成注册行为。

### Lines 108-108
```python
    mask = _unwrap_if_constexpr(mask)
```
**EN:** Inside function `buffer_load_to_shared`, this assignment updates `mask` with `_unwrap_if_constexpr(mask)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `buffer_load_to_shared` 内部，这段赋值把 `_unwrap_if_constexpr(mask)` 写入 `mask`，为后续逻辑建立状态、别名或配置。

### Lines 109-110
```python
    if mask is not None:
        offsets, mask = _semantic.broadcast_impl_value(offsets, mask)
```
**EN:** Inside function `buffer_load_to_shared`, this conditional checks `mask is not None` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `buffer_load_to_shared` 内部，这段条件语句检查 `mask is not None`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 111-111
```python
    other = _unwrap_if_constexpr(other)
```
**EN:** Inside function `buffer_load_to_shared`, this assignment updates `other` with `_unwrap_if_constexpr(other)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `buffer_load_to_shared` 内部，这段赋值把 `_unwrap_if_constexpr(other)` 写入 `other`，为后续逻辑建立状态、别名或配置。

### Lines 112-115
```python
    if other is not None:
        other = _semantic.to_tensor(other)
        other = _semantic.cast(other, ptr.type.scalar.element_ty)
        offsets, other = _semantic.broadcast_impl_value(offsets, other)
```
**EN:** Inside function `buffer_load_to_shared`, this conditional checks `other is not None` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `buffer_load_to_shared` 内部，这段条件语句检查 `other is not None`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 117-117
```python
    mask = mask.handle if mask is not None else ir.value()
```
**EN:** Inside function `buffer_load_to_shared`, this assignment updates `mask` with `mask.handle if mask is not None else ir.value()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `buffer_load_to_shared` 内部，这段赋值把 `mask.handle if mask is not None else ir.value()` 写入 `mask`，为后续逻辑建立状态、别名或配置。

### Lines 118-118
```python
    other = other.handle if other is not None else ir.value()
```
**EN:** Inside function `buffer_load_to_shared`, this assignment updates `other` with `other.handle if other is not None else ir.value()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `buffer_load_to_shared` 内部，这段赋值把 `other.handle if other is not None else ir.value()` 写入 `other`，为后续逻辑建立状态、别名或配置。

### Lines 119-119
```python
    stride = ir.value()
```
**EN:** Inside function `buffer_load_to_shared`, this assignment updates `stride` with `ir.value()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `buffer_load_to_shared` 内部，这段赋值把 `ir.value()` 写入 `stride`，为后续逻辑建立状态、别名或配置。

### Lines 120-120
```python
    cache_modifier = _semantic._str_to_load_cache_modifier(cache_modifier)
```
**EN:** Inside function `buffer_load_to_shared`, this assignment updates `cache_modifier` with `_semantic._str_to_load_cache_modifier(cache_modifier)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `buffer_load_to_shared` 内部，这段赋值把 `_semantic._str_to_load_cache_modifier(cache_modifier)` 写入 `cache_modifier`，为后续逻辑建立状态、别名或配置。

### Lines 122-123
```python
    _semantic.builder.create_buffer_load_to_local(dest.handle, ptr.handle, offsets.handle, mask, other, stride,
                                                  cache_modifier)
```
**EN:** Inside function `buffer_load_to_shared`, this expression evaluates `_semantic.builder.create_buffer_load_to_local` mainly for its side effects or registration behavior.
**CN:** 在函数 `buffer_load_to_shared` 内部，这条表达式计算 `_semantic.builder.create_buffer_load_to_local`，主要目的是触发副作用或完成注册行为。

### Lines 126-127
```python
@builtin
def commit_group(_semantic=None):
```
**EN:** At module scope, this header declares the function `commit_group(_semantic)`, which is responsible for commit group. Decorators: builtin. The docstring says: Commit oustanding async operations.
**CN:** 在模块级作用域中，这段头部声明了函数 `commit_group(_semantic)`，它负责处理 commit group 相关逻辑。 装饰器包括：builtin。 文档字符串说明：Commit oustanding async operations.

### Lines 128-132
```python
    """
    Commit oustanding async operations.

    This finalizes a set of async copy operations which can be waited upon via `wait_group`.
    """
```
**EN:** Inside function `commit_group`, this docstring documents the surrounding scope. Summary: Commit oustanding async operations.
**CN:** 在函数 `commit_group` 内部，这段文档字符串用于说明当前作用域。摘要：Commit oustanding async operations.

### Lines 133-133
```python
    _semantic.builder.create_async_commit_group()
```
**EN:** Inside function `commit_group`, this expression evaluates `_semantic.builder.create_async_commit_group` mainly for its side effects or registration behavior.
**CN:** 在函数 `commit_group` 内部，这条表达式计算 `_semantic.builder.create_async_commit_group`，主要目的是触发副作用或完成注册行为。

### Lines 136-137
```python
@builtin
def wait_group(num_outstanding=0, _semantic=None):
```
**EN:** At module scope, this header declares the function `wait_group(num_outstanding, _semantic)`, which is responsible for wait group. Decorators: builtin. The docstring says: Wait for outstanding commit groups.
**CN:** 在模块级作用域中，这段头部声明了函数 `wait_group(num_outstanding, _semantic)`，它负责处理 wait group 相关逻辑。 装饰器包括：builtin。 文档字符串说明：Wait for outstanding commit groups.

### Lines 138-145
```python
    """
    Wait for outstanding commit groups. It will block until the number of
    outstanding commit groups is less than or equal to `num_outstanding`. Note that uncommited
    async operations will be waited upon even if `num_outstanding` is 0.

    Args:
        num_outstanding (int): The number of outstanding commit groups to wait for. Defaults to 0.
    """
```
**EN:** Inside function `wait_group`, this docstring documents the surrounding scope. Summary: Wait for outstanding commit groups.
**CN:** 在函数 `wait_group` 内部，这段文档字符串用于说明当前作用域。摘要：Wait for outstanding commit groups.

### Lines 146-146
```python
    num_outstanding = _unwrap_if_constexpr(num_outstanding)
```
**EN:** Inside function `wait_group`, this assignment updates `num_outstanding` with `_unwrap_if_constexpr(num_outstanding)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `wait_group` 内部，这段赋值把 `_unwrap_if_constexpr(num_outstanding)` 写入 `num_outstanding`，为后续逻辑建立状态、别名或配置。

### Lines 147-147
```python
    _semantic.builder.create_async_wait_group(num_outstanding)
```
**EN:** Inside function `wait_group`, this expression evaluates `_semantic.builder.create_async_wait_group` mainly for its side effects or registration behavior.
**CN:** 在函数 `wait_group` 内部，这条表达式计算 `_semantic.builder.create_async_wait_group`，主要目的是触发副作用或完成注册行为。

### Lines 150-151
```python
@builtin
def load_shared_relaxed(smem, layout, _semantic=None):
```
**EN:** At module scope, this header declares the function `load_shared_relaxed(smem, layout, _semantic)`, which is responsible for load shared relaxed. Decorators: builtin. The docstring says: Load a tensor from shared memory with extra hints for the underlying compiler to avoid emitting unnecessary waits bef...
**CN:** 在模块级作用域中，这段头部声明了函数 `load_shared_relaxed(smem, layout, _semantic)`，它负责处理 load shared relaxed 相关逻辑。 装饰器包括：builtin。 文档字符串说明：Load a tensor from shared memory with extra hints for the underlying compiler to avoid emitting unnecessary waits bef...

### Lines 152-163
```python
    """
    Load a tensor from shared memory with extra hints for the underlying
    compiler to avoid emitting unnecessary waits before loading from the target
    shared memory.

    Args:
        smem (shared_memory_descriptor): Shared memory descriptor to load from.
        layout (DistributedLayout): The destination layout of the tensor.

    Returns:
        tensor: A Gluon tensor containing the loaded data.
    """
```
**EN:** Inside function `load_shared_relaxed`, this docstring documents the surrounding scope. Summary: Load a tensor from shared memory with extra hints for the underlying compiler to avoid emitting unnecessary waits bef...
**CN:** 在函数 `load_shared_relaxed` 内部，这段文档字符串用于说明当前作用域。摘要：Load a tensor from shared memory with extra hints for the underlying compiler to avoid emitting unnecessary waits bef...

### Lines 164-164
```python
    SYNCED_VIA_WAIT_ATTR_NAME = "ttg.amdg.syncedViaAsyncWait"
```
**EN:** Inside function `load_shared_relaxed`, this assignment updates `SYNCED_VIA_WAIT_ATTR_NAME` with `'ttg.amdg.syncedViaAsyncWait'`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `load_shared_relaxed` 内部，这段赋值把 `'ttg.amdg.syncedViaAsyncWait'` 写入 `SYNCED_VIA_WAIT_ATTR_NAME`，为后续逻辑建立状态、别名或配置。

### Lines 166-166
```python
    layout = _unwrap_if_constexpr(layout)
```
**EN:** Inside function `load_shared_relaxed`, this assignment updates `layout` with `_unwrap_if_constexpr(layout)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `load_shared_relaxed` 内部，这段赋值把 `_unwrap_if_constexpr(layout)` 写入 `layout`，为后续逻辑建立状态、别名或配置。

### Lines 167-167
```python
    ret = _semantic.shared_load(smem, layout)
```
**EN:** Inside function `load_shared_relaxed`, this assignment updates `ret` with `_semantic.shared_load(smem, layout)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `load_shared_relaxed` 内部，这段赋值把 `_semantic.shared_load(smem, layout)` 写入 `ret`，为后续逻辑建立状态、别名或配置。

### Lines 168-168
```python
    ret.handle.set_attr(SYNCED_VIA_WAIT_ATTR_NAME, _semantic.builder.get_bool_attr(True))
```
**EN:** Inside function `load_shared_relaxed`, this expression evaluates `ret.handle.set_attr` mainly for its side effects or registration behavior.
**CN:** 在函数 `load_shared_relaxed` 内部，这条表达式计算 `ret.handle.set_attr`，主要目的是触发副作用或完成注册行为。

### Lines 169-169
```python
    return ret
```
**EN:** Inside function `load_shared_relaxed`, this return statement sends `ret` back to the caller as the result of the current routine.
**CN:** 在函数 `load_shared_relaxed` 内部，这条返回语句把 `ret` 作为当前过程的结果返回给调用方。

## Key Concepts / 关键概念

- **EN:** Path theme: `python/triton/experimental/gluon/language/amd/cdna4` places this module in Triton's triton / experimental / gluon / language / amd / cdna4 area.
  **CN:** 路径主题：`python/triton/experimental/gluon/language/amd/cdna4` 表明该模块位于 Triton 的 triton / experimental / gluon / language / amd / cdna4 领域。
- **EN:** Primary functions: `global_load_to_shared`, `buffer_load_to_shared`, `commit_group`, `wait_group`, `load_shared_relaxed`.
  **CN:** 主要函数：`global_load_to_shared`, `buffer_load_to_shared`, `commit_group`, `wait_group`, `load_shared_relaxed`。
- **EN:** Frontend language model: this module shapes Triton language semantics, types, or helper intrinsics.
  **CN:** 前端语言模型：该模块塑造 Triton 语言语义、类型或辅助内建函数。
- **EN:** Experimental API: this module lives under `experimental`, so its interfaces may evolve quickly.
  **CN:** 实验性 API：该模块位于 `experimental` 下，其接口可能会快速演进。

## Dependencies / 依赖关系

- **EN:** Internal Triton modules: ..._core, ..._semantic, ..._layouts, ..cdna3.
  **CN:** Triton 内部模块：..._core, ..._semantic, ..._layouts, ..cdna3。
