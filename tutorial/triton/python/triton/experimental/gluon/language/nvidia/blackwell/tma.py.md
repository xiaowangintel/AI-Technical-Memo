# tma.py — Code Analysis / 代码分析

## Source / 来源

- **Path:** `./python/triton/experimental/gluon/language/nvidia/blackwell/tma.py`
- **EN:** This source file at `./python/triton/experimental/gluon/language/nvidia/blackwell/tma.py` defines the main symbols `async_gather`, `_emit_scatter_nonnegative_check`, `async_scatter` for the surrounding Triton subsystem.
- **CN:** 位于 `./python/triton/experimental/gluon/language/nvidia/blackwell/tma.py` 的这个源文件为周边 Triton 子系统定义了主要符号 `async_gather`, `_emit_scatter_nonnegative_check`, `async_scatter`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1
```python
import triton.experimental.gluon.language._core as ttgl
```
**EN:** At module scope, this block imports triton.experimental.gluon.language._core as ttgl so later definitions can reuse those modules or symbols.
**CN:** 在模块级作用域中，这段代码导入了 triton.experimental.gluon.language._core as ttgl，供后续定义复用这些模块或符号。

### Lines 2-2
```python
from triton.experimental.gluon.language._core import builtin
```
**EN:** At module scope, this block imports builtin from `triton.experimental.gluon.language._core` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `triton.experimental.gluon.language._core` 导入 builtin，把当前文件与周边 API 和辅助工具连接起来。

### Lines 3-20
```python
from triton.experimental.gluon.language.nvidia.hopper.tma import (
    async_atomic_add,
    async_atomic_and,
    async_atomic_max,
    async_atomic_min,
    async_atomic_or,
    async_atomic_xor,
    async_copy_global_to_shared,
    async_copy_shared_to_global,
    async_load,
    async_load_im2col,
    async_store,
    store_wait,
    tensor_descriptor,
    tensor_descriptor_type,
    make_tensor_descriptor,
    _emit_alignment_check,
)
```
**EN:** At module scope, this block imports async_atomic_add, async_atomic_and, async_atomic_max, async_atomic_min, async_atomic_or, async_atomic_xor, async_copy_global_to_shared, async_copy_shared_to_global, and 8 more from `triton.experimental.gluon.language.nvidia.hopper.tma` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `triton.experimental.gluon.language.nvidia.hopper.tma` 导入 async_atomic_add, async_atomic_and, async_atomic_max, async_atomic_min, async_atomic_or, async_atomic_xor, async_copy_global_to_shared, async_copy_shared_to_global, and 8 more，把当前文件与周边 API 和辅助工具连接起来。

### Lines 22-40
```python
__all__ = [
    "async_gather",
    "async_scatter",
    "async_atomic_add",
    "async_atomic_and",
    "async_atomic_max",
    "async_atomic_min",
    "async_atomic_or",
    "async_atomic_xor",
    "async_copy_global_to_shared",
    "async_copy_shared_to_global",
    "async_load",
    "async_load_im2col",
    "async_store",
    "store_wait",
    "tensor_descriptor",
    "tensor_descriptor_type",
    "make_tensor_descriptor",
]
```
**EN:** At module scope, this assignment updates `__all__` with `['async_gather', 'async_scatter', 'async_atomic_add', 'async_atomic_and', 'as...`, establishing state, aliases, or configuration used later.
**CN:** 在模块级作用域中，这段赋值把 `['async_gather', 'async_scatter', 'async_atomic_add', 'async_atomic_and', 'as...` 写入 `__all__`，为后续逻辑建立状态、别名或配置。

### Lines 43-44
```python
@builtin
def async_gather(tensor_desc, x_offsets, y_offset, barrier, result, pred=True, multicast=False, _semantic=None):
```
**EN:** At module scope, this header declares the function `async_gather(tensor_desc, x_offsets, y_offset, barrier, result, pred, multicast, _semantic)`, which is responsible for async gather. Decorators: builtin. The docstring says: Asynchronously gather elements from global memory to shared memory using TMA.
**CN:** 在模块级作用域中，这段头部声明了函数 `async_gather(tensor_desc, x_offsets, y_offset, barrier, result, pred, multicast, _semantic)`，它负责处理 async gather 相关逻辑。 装饰器包括：builtin。 文档字符串说明：Asynchronously gather elements from global memory to shared memory using TMA.

### Lines 45-56
```python
    """
    Asynchronously gather elements from global memory to shared memory using TMA.

    Args:
        tensor_desc (tensor_descriptor): The tensor descriptor.
        x_offsets (tensor): 1D tensor of X offsets.
        y_offset (int): Scalar Y offset.
        barrier (shared_memory_descriptor): Barrier that will be signaled when the operation is complete.
        result (tensor_memory_descriptor): Result shared memory, must have NVMMASharedLayout.
        pred (bool): Scalar predicate. Operation is skipped if predicate is False. Defaults to True.
        multicast (bool): Enable multicast.
    """
```
**EN:** Inside function `async_gather`, this docstring documents the surrounding scope. Summary: Asynchronously gather elements from global memory to shared memory using TMA.
**CN:** 在函数 `async_gather` 内部，这段文档字符串用于说明当前作用域。摘要：Asynchronously gather elements from global memory to shared memory using TMA.

### Lines 57-58
```python
    if _semantic.builder.options.enable_iisan:
        _emit_alignment_check(tensor_desc, (y_offset, ), "async_gather", "y_offset", _semantic=_semantic)
```
**EN:** Inside function `async_gather`, this conditional checks `_semantic.builder.options.enable_iisan` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `async_gather` 内部，这段条件语句检查 `_semantic.builder.options.enable_iisan`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 60-60
```python
    pred = _semantic.to_tensor(pred)
```
**EN:** Inside function `async_gather`, this assignment updates `pred` with `_semantic.to_tensor(pred)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `async_gather` 内部，这段赋值把 `_semantic.to_tensor(pred)` 写入 `pred`，为后续逻辑建立状态、别名或配置。

### Lines 61-61
```python
    y_offset = _semantic.to_tensor(y_offset)
```
**EN:** Inside function `async_gather`, this assignment updates `y_offset` with `_semantic.to_tensor(y_offset)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `async_gather` 内部，这段赋值把 `_semantic.to_tensor(y_offset)` 写入 `y_offset`，为后续逻辑建立状态、别名或配置。

### Lines 62-62
```python
    multicast = ttgl._unwrap_if_constexpr(multicast)
```
**EN:** Inside function `async_gather`, this assignment updates `multicast` with `ttgl._unwrap_if_constexpr(multicast)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `async_gather` 内部，这段赋值把 `ttgl._unwrap_if_constexpr(multicast)` 写入 `multicast`，为后续逻辑建立状态、别名或配置。

### Lines 63-64
```python
    _semantic.builder.create_async_tma_gather(tensor_desc.handle, x_offsets.handle, y_offset.handle, barrier.handle,
                                              result.handle, pred.handle, multicast)
```
**EN:** Inside function `async_gather`, this expression evaluates `_semantic.builder.create_async_tma_gather` mainly for its side effects or registration behavior.
**CN:** 在函数 `async_gather` 内部，这条表达式计算 `_semantic.builder.create_async_tma_gather`，主要目的是触发副作用或完成注册行为。

### Lines 67-67
```python
def _emit_scatter_nonnegative_check(x_offsets, y_offset, _semantic=None):
```
**EN:** At module scope, this header declares the function `_emit_scatter_nonnegative_check(x_offsets, y_offset, _semantic)`, which is responsible for emit scatter nonnegative check.
**CN:** 在模块级作用域中，这段头部声明了函数 `_emit_scatter_nonnegative_check(x_offsets, y_offset, _semantic)`，它负责处理 emit scatter nonnegative check 相关逻辑。

### Lines 68-68
```python
    y_offset = ttgl.to_tensor(y_offset, _semantic=_semantic)
```
**EN:** Inside function `_emit_scatter_nonnegative_check`, this assignment updates `y_offset` with `ttgl.to_tensor(y_offset, _semantic=_semantic)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `_emit_scatter_nonnegative_check` 内部，这段赋值把 `ttgl.to_tensor(y_offset, _semantic=_semantic)` 写入 `y_offset`，为后续逻辑建立状态、别名或配置。

### Lines 69-69
```python
    zero = ttgl.to_tensor(0, _semantic=_semantic)
```
**EN:** Inside function `_emit_scatter_nonnegative_check`, this assignment updates `zero` with `ttgl.to_tensor(0, _semantic=_semantic)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `_emit_scatter_nonnegative_check` 内部，这段赋值把 `ttgl.to_tensor(0, _semantic=_semantic)` 写入 `zero`，为后续逻辑建立状态、别名或配置。

### Lines 71-71
```python
    is_nonnegative = y_offset.__ge__(zero, _semantic=_semantic)
```
**EN:** Inside function `_emit_scatter_nonnegative_check`, this assignment updates `is_nonnegative` with `y_offset.__ge__(zero, _semantic=_semantic)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `_emit_scatter_nonnegative_check` 内部，这段赋值把 `y_offset.__ge__(zero, _semantic=_semantic)` 写入 `is_nonnegative`，为后续逻辑建立状态、别名或配置。

### Lines 72-72
```python
    ttgl.device_assert(is_nonnegative, "async_scatter y_offset cannot be negative", _semantic=_semantic)
```
**EN:** Inside function `_emit_scatter_nonnegative_check`, this expression evaluates `ttgl.device_assert` mainly for its side effects or registration behavior.
**CN:** 在函数 `_emit_scatter_nonnegative_check` 内部，这条表达式计算 `ttgl.device_assert`，主要目的是触发副作用或完成注册行为。

### Lines 74-74
```python
    is_nonnegative = x_offsets.__ge__(zero, _semantic=_semantic)
```
**EN:** Inside function `_emit_scatter_nonnegative_check`, this assignment updates `is_nonnegative` with `x_offsets.__ge__(zero, _semantic=_semantic)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `_emit_scatter_nonnegative_check` 内部，这段赋值把 `x_offsets.__ge__(zero, _semantic=_semantic)` 写入 `is_nonnegative`，为后续逻辑建立状态、别名或配置。

### Lines 75-75
```python
    ttgl.device_assert(is_nonnegative, "async_scatter x_offsets cannot have any negative elements", _semantic=_semantic)
```
**EN:** Inside function `_emit_scatter_nonnegative_check`, this expression evaluates `ttgl.device_assert` mainly for its side effects or registration behavior.
**CN:** 在函数 `_emit_scatter_nonnegative_check` 内部，这条表达式计算 `ttgl.device_assert`，主要目的是触发副作用或完成注册行为。

### Lines 78-79
```python
@builtin
def async_scatter(tensor_desc, x_offsets, y_offset, src, _semantic=None):
```
**EN:** At module scope, this header declares the function `async_scatter(tensor_desc, x_offsets, y_offset, src, _semantic)`, which is responsible for async scatter. Decorators: builtin. The docstring says: Asynchronously scatter elements from shared memory to global memory using TMA.
**CN:** 在模块级作用域中，这段头部声明了函数 `async_scatter(tensor_desc, x_offsets, y_offset, src, _semantic)`，它负责处理 async scatter 相关逻辑。 装饰器包括：builtin。 文档字符串说明：Asynchronously scatter elements from shared memory to global memory using TMA.

### Lines 80-88
```python
    """
    Asynchronously scatter elements from shared memory to global memory using TMA.

    Args:
        tensor_desc (tensor_descriptor): The tensor descriptor.
        x_offsets (tensor): 1D tensor of X offsets.
        y_offset (int): Scalar Y offset.
        src (tensor_memory_descriptor): The source data, must be in NVMMASharedLayout.
    """
```
**EN:** Inside function `async_scatter`, this docstring documents the surrounding scope. Summary: Asynchronously scatter elements from shared memory to global memory using TMA.
**CN:** 在函数 `async_scatter` 内部，这段文档字符串用于说明当前作用域。摘要：Asynchronously scatter elements from shared memory to global memory using TMA.

### Lines 89-91
```python
    if _semantic.builder.options.enable_iisan:
        _emit_alignment_check(tensor_desc, (y_offset, ), "async_scatter", "y_offset", _semantic=_semantic)
        _emit_scatter_nonnegative_check(x_offsets, y_offset, _semantic=_semantic)
```
**EN:** Inside function `async_scatter`, this conditional checks `_semantic.builder.options.enable_iisan` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `async_scatter` 内部，这段条件语句检查 `_semantic.builder.options.enable_iisan`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 93-93
```python
    y_offset = _semantic.to_tensor(y_offset)
```
**EN:** Inside function `async_scatter`, this assignment updates `y_offset` with `_semantic.to_tensor(y_offset)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `async_scatter` 内部，这段赋值把 `_semantic.to_tensor(y_offset)` 写入 `y_offset`，为后续逻辑建立状态、别名或配置。

### Lines 94-94
```python
    _semantic.builder.create_async_tma_scatter(tensor_desc.handle, x_offsets.handle, y_offset.handle, src.handle)
```
**EN:** Inside function `async_scatter`, this expression evaluates `_semantic.builder.create_async_tma_scatter` mainly for its side effects or registration behavior.
**CN:** 在函数 `async_scatter` 内部，这条表达式计算 `_semantic.builder.create_async_tma_scatter`，主要目的是触发副作用或完成注册行为。

## Key Concepts / 关键概念

- **EN:** Path theme: `python/triton/experimental/gluon/language/nvidia/blackwell` places this module in Triton's triton / experimental / gluon / language / nvidia / blackwell area.
  **CN:** 路径主题：`python/triton/experimental/gluon/language/nvidia/blackwell` 表明该模块位于 Triton 的 triton / experimental / gluon / language / nvidia / blackwell 领域。
- **EN:** Primary functions: `async_gather`, `_emit_scatter_nonnegative_check`, `async_scatter`.
  **CN:** 主要函数：`async_gather`, `_emit_scatter_nonnegative_check`, `async_scatter`。
- **EN:** Frontend language model: this module shapes Triton language semantics, types, or helper intrinsics.
  **CN:** 前端语言模型：该模块塑造 Triton 语言语义、类型或辅助内建函数。
- **EN:** Experimental API: this module lives under `experimental`, so its interfaces may evolve quickly.
  **CN:** 实验性 API：该模块位于 `experimental` 下，其接口可能会快速演进。

## Dependencies / 依赖关系

- **EN:** Internal Triton modules: triton.experimental.gluon.language._core, triton.experimental.gluon.language.nvidia.hopper.tma.
  **CN:** Triton 内部模块：triton.experimental.gluon.language._core, triton.experimental.gluon.language.nvidia.hopper.tma。
