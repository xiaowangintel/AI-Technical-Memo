# _testing_utils.py — Code Analysis / 代码分析

## Source / 来源

- **Path:** `./python/triton/experimental/gsan/_testing_utils.py`
- **EN:** This source file at `./python/triton/experimental/gsan/_testing_utils.py` defines the main symbols `nanosleep`, `atomic_poll`, `shadow_cell_tensor_from_address`, `shadow_cell_from_address` for the surrounding Triton subsystem.
- **CN:** 位于 `./python/triton/experimental/gsan/_testing_utils.py` 的这个源文件为周边 Triton 子系统定义了主要符号 `nanosleep`, `atomic_poll`, `shadow_cell_tensor_from_address`, `shadow_cell_from_address`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1
```python
from __future__ import annotations
```
**EN:** At module scope, this block imports annotations from `__future__` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `__future__` 导入 annotations，把当前文件与周边 API 和辅助工具连接起来。

### Lines 3-3
```python
import torch
```
**EN:** At module scope, this block imports torch so later definitions can reuse those modules or symbols.
**CN:** 在模块级作用域中，这段代码导入了 torch，供后续定义复用这些模块或符号。

### Lines 4-4
```python
import triton
```
**EN:** At module scope, this block imports triton so later definitions can reuse those modules or symbols.
**CN:** 在模块级作用域中，这段代码导入了 triton，供后续定义复用这些模块或符号。

### Lines 5-5
```python
import triton.language as tl
```
**EN:** At module scope, this block imports triton.language as tl so later definitions can reuse those modules or symbols.
**CN:** 在模块级作用域中，这段代码导入了 triton.language as tl，供后续定义复用这些模块或符号。

### Lines 7-7
```python
from ._allocator import get_global_state_pointer
```
**EN:** At module scope, this block imports get_global_state_pointer from `._allocator` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `._allocator` 导入 get_global_state_pointer，把当前文件与周边 API 和辅助工具连接起来。

### Lines 8-8
```python
from triton._C.libtriton.gsan_testing import thread_state_address, SHADOW_GRANULARITY_BYTES, PER_DEVICE_STATE_STRIDE_BYTES, GLOBAL_STATE_SIZE_BYTES, shadow_cell_address, thread_state_stride_bytes, SHADOW_CELL_SIZE_BYTES
```
**EN:** At module scope, this block imports thread_state_address, SHADOW_GRANULARITY_BYTES, PER_DEVICE_STATE_STRIDE_BYTES, GLOBAL_STATE_SIZE_BYTES, shadow_cell_address, thread_state_stride_bytes, SHADOW_CELL_SIZE_BYTES from `triton._C.libtriton.gsan_testing` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `triton._C.libtriton.gsan_testing` 导入 thread_state_address, SHADOW_GRANULARITY_BYTES, PER_DEVICE_STATE_STRIDE_BYTES, GLOBAL_STATE_SIZE_BYTES, shadow_cell_address, thread_state_stride_bytes, SHADOW_CELL_SIZE_BYTES，把当前文件与周边 API 和辅助工具连接起来。

### Lines 9-9
```python
from ._testing import (decode_global_state_tensor, decode_shadow_cell_tensor, decode_thread_state_tensor)
```
**EN:** At module scope, this block imports decode_global_state_tensor, decode_shadow_cell_tensor, decode_thread_state_tensor from `._testing` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `._testing` 导入 decode_global_state_tensor, decode_shadow_cell_tensor, decode_thread_state_tensor，把当前文件与周边 API 和辅助工具连接起来。

### Lines 10-10
```python
from ._utils import uint8_cuda_tensor_from_ptr
```
**EN:** At module scope, this block imports uint8_cuda_tensor_from_ptr from `._utils` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `._utils` 导入 uint8_cuda_tensor_from_ptr，把当前文件与周边 API 和辅助工具连接起来。

### Lines 13-14
```python
@triton.jit
def nanosleep(duration):
```
**EN:** At module scope, this header declares the function `nanosleep(duration)`, which is responsible for nanosleep. Decorators: triton.jit.
**CN:** 在模块级作用域中，这段头部声明了函数 `nanosleep(duration)`，它负责处理 nanosleep 相关逻辑。 装饰器包括：triton.jit。

### Lines 15-15
```python
    duration = tl.to_tensor(duration)
```
**EN:** Inside function `nanosleep`, this assignment updates `duration` with `tl.to_tensor(duration)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `nanosleep` 内部，这段赋值把 `tl.to_tensor(duration)` 写入 `duration`，为后续逻辑建立状态、别名或配置。

### Lines 16-16
```python
    tl.inline_asm_elementwise("nanosleep.u32 $1; mov.b32 $0, 0;", "=r, r", [duration], tl.int32, is_pure=False, pack=1)
```
**EN:** Inside function `nanosleep`, this expression evaluates `tl.inline_asm_elementwise` mainly for its side effects or registration behavior.
**CN:** 在函数 `nanosleep` 内部，这条表达式计算 `tl.inline_asm_elementwise`，主要目的是触发副作用或完成注册行为。

### Lines 19-20
```python
@triton.jit
def atomic_poll(ptr, expect, sem: tl.constexpr = "relaxed", scope: tl.constexpr = "gpu"):
```
**EN:** At module scope, this header declares the function `atomic_poll(ptr, expect, sem, scope)`, which is responsible for atomic poll. Decorators: triton.jit.
**CN:** 在模块级作用域中，这段头部声明了函数 `atomic_poll(ptr, expect, sem, scope)`，它负责处理 atomic poll 相关逻辑。 装饰器包括：triton.jit。

### Lines 21-22
```python
    while tl.atomic_add(ptr, 0, sem=sem, scope=scope) != expect:
        nanosleep(100)
```
**EN:** Inside function `atomic_poll`, this loop keeps running while `tl.atomic_add(ptr, 0, sem=sem, scope=scope) != expect` remains true.
**CN:** 在函数 `atomic_poll` 内部，这段循环会在 `tl.atomic_add(ptr, 0, sem=sem, scope=scope) != expect` 为真时持续执行。

### Lines 25-25
```python
def shadow_cell_tensor_from_address(real_address: int, *, device_index: int | None = None) -> torch.Tensor:
```
**EN:** At module scope, this header declares the function `shadow_cell_tensor_from_address(real_address, *, device_index)`, which is responsible for shadow cell tensor from address.
**CN:** 在模块级作用域中，这段头部声明了函数 `shadow_cell_tensor_from_address(real_address, *, device_index)`，它负责处理 shadow cell tensor from address 相关逻辑。

### Lines 26-27
```python
    if device_index is None:
        device_index = torch.cuda.current_device()
```
**EN:** Inside function `shadow_cell_tensor_from_address`, this conditional checks `device_index is None` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `shadow_cell_tensor_from_address` 内部，这段条件语句检查 `device_index is None`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 28-28
```python
    shadow_ptr = shadow_cell_address(real_address)
```
**EN:** Inside function `shadow_cell_tensor_from_address`, this assignment updates `shadow_ptr` with `shadow_cell_address(real_address)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `shadow_cell_tensor_from_address` 内部，这段赋值把 `shadow_cell_address(real_address)` 写入 `shadow_ptr`，为后续逻辑建立状态、别名或配置。

### Lines 29-29
```python
    return uint8_cuda_tensor_from_ptr(shadow_ptr, SHADOW_CELL_SIZE_BYTES, device_index)
```
**EN:** Inside function `shadow_cell_tensor_from_address`, this return statement sends `uint8_cuda_tensor_from_ptr(shadow_ptr, SHADOW_CELL_SIZE_BYTES, device_index)` back to the caller as the result of the current routine.
**CN:** 在函数 `shadow_cell_tensor_from_address` 内部，这条返回语句把 `uint8_cuda_tensor_from_ptr(shadow_ptr, SHADOW_CELL_SIZE_BYTES, device_index)` 作为当前过程的结果返回给调用方。

### Lines 32-32
```python
def shadow_cell_from_address(real_address: int, *, device_index: int | None = None):
```
**EN:** At module scope, this header declares the function `shadow_cell_from_address(real_address, *, device_index)`, which is responsible for shadow cell from address.
**CN:** 在模块级作用域中，这段头部声明了函数 `shadow_cell_from_address(real_address, *, device_index)`，它负责处理 shadow cell from address 相关逻辑。

### Lines 33-33
```python
    return decode_shadow_cell_tensor(shadow_cell_tensor_from_address(real_address, device_index=device_index))
```
**EN:** Inside function `shadow_cell_from_address`, this return statement sends `decode_shadow_cell_tensor(shadow_cell_tensor_from_address(real_address, device_index=device_index))` back to the caller as the result of the current routine.
**CN:** 在函数 `shadow_cell_from_address` 内部，这条返回语句把 `decode_shadow_cell_tensor(shadow_cell_tensor_from_address(real_address, device_index=device_index))` 作为当前过程的结果返回给调用方。

### Lines 36-36
```python
def global_state_tensor(*, device_index: int | None = None) -> torch.Tensor:
```
**EN:** At module scope, this header declares the function `global_state_tensor(*, device_index)`, which is responsible for global state tensor.
**CN:** 在模块级作用域中，这段头部声明了函数 `global_state_tensor(*, device_index)`，它负责处理 global state tensor 相关逻辑。

### Lines 37-38
```python
    if device_index is None:
        device_index = torch.cuda.current_device()
```
**EN:** Inside function `global_state_tensor`, this conditional checks `device_index is None` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `global_state_tensor` 内部，这段条件语句检查 `device_index is None`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 39-39
```python
    ptr = get_global_state_pointer() + device_index * PER_DEVICE_STATE_STRIDE_BYTES
```
**EN:** Inside function `global_state_tensor`, this assignment updates `ptr` with `get_global_state_pointer() + device_index * PER_DEVICE_STATE_STRIDE_BYTES`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `global_state_tensor` 内部，这段赋值把 `get_global_state_pointer() + device_index * PER_DEVICE_STATE_STRIDE_BYTES` 写入 `ptr`，为后续逻辑建立状态、别名或配置。

### Lines 40-40
```python
    return uint8_cuda_tensor_from_ptr(ptr, GLOBAL_STATE_SIZE_BYTES, device_index)
```
**EN:** Inside function `global_state_tensor`, this return statement sends `uint8_cuda_tensor_from_ptr(ptr, GLOBAL_STATE_SIZE_BYTES, device_index)` back to the caller as the result of the current routine.
**CN:** 在函数 `global_state_tensor` 内部，这条返回语句把 `uint8_cuda_tensor_from_ptr(ptr, GLOBAL_STATE_SIZE_BYTES, device_index)` 作为当前过程的结果返回给调用方。

### Lines 43-43
```python
def global_state(*, device_index: int | None = None):
```
**EN:** At module scope, this header declares the function `global_state(*, device_index)`, which is responsible for global state.
**CN:** 在模块级作用域中，这段头部声明了函数 `global_state(*, device_index)`，它负责处理 global state 相关逻辑。

### Lines 44-44
```python
    return decode_global_state_tensor(global_state_tensor(device_index=device_index))
```
**EN:** Inside function `global_state`, this return statement sends `decode_global_state_tensor(global_state_tensor(device_index=device_index))` back to the caller as the result of the current routine.
**CN:** 在函数 `global_state` 内部，这条返回语句把 `decode_global_state_tensor(global_state_tensor(device_index=device_index))` 作为当前过程的结果返回给调用方。

### Lines 47-47
```python
def thread_state_tensor(smid: int, *, device_index: int | None = None) -> torch.Tensor:
```
**EN:** At module scope, this header declares the function `thread_state_tensor(smid, *, device_index)`, which is responsible for thread state tensor.
**CN:** 在模块级作用域中，这段头部声明了函数 `thread_state_tensor(smid, *, device_index)`，它负责处理 thread state tensor 相关逻辑。

### Lines 48-49
```python
    if device_index is None:
        device_index = torch.cuda.current_device()
```
**EN:** Inside function `thread_state_tensor`, this conditional checks `device_index is None` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `thread_state_tensor` 内部，这段条件语句检查 `device_index is None`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 50-50
```python
    gs = global_state(device_index=device_index)
```
**EN:** Inside function `thread_state_tensor`, this assignment updates `gs` with `global_state(device_index=device_index)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `thread_state_tensor` 内部，这段赋值把 `global_state(device_index=device_index)` 写入 `gs`，为后续逻辑建立状态、别名或配置。

### Lines 51-56
```python
    ptr = thread_state_address(
        get_global_state_pointer() + device_index * PER_DEVICE_STATE_STRIDE_BYTES,
        gs.num_threads,
        gs.clock_buffer_size,
        smid,
    )
```
**EN:** Inside function `thread_state_tensor`, this assignment updates `ptr` with `thread_state_address(get_global_state_pointer() + device_index * PER_DEVICE_S...`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `thread_state_tensor` 内部，这段赋值把 `thread_state_address(get_global_state_pointer() + device_index * PER_DEVICE_S...` 写入 `ptr`，为后续逻辑建立状态、别名或配置。

### Lines 57-57
```python
    size = thread_state_stride_bytes(gs.num_threads, gs.clock_buffer_size)
```
**EN:** Inside function `thread_state_tensor`, this assignment updates `size` with `thread_state_stride_bytes(gs.num_threads, gs.clock_buffer_size)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `thread_state_tensor` 内部，这段赋值把 `thread_state_stride_bytes(gs.num_threads, gs.clock_buffer_size)` 写入 `size`，为后续逻辑建立状态、别名或配置。

### Lines 58-58
```python
    return uint8_cuda_tensor_from_ptr(ptr, size, device_index)
```
**EN:** Inside function `thread_state_tensor`, this return statement sends `uint8_cuda_tensor_from_ptr(ptr, size, device_index)` back to the caller as the result of the current routine.
**CN:** 在函数 `thread_state_tensor` 内部，这条返回语句把 `uint8_cuda_tensor_from_ptr(ptr, size, device_index)` 作为当前过程的结果返回给调用方。

### Lines 61-61
```python
def thread_state_from_smid(smid: int, *, device_index: int | None = None):
```
**EN:** At module scope, this header declares the function `thread_state_from_smid(smid, *, device_index)`, which is responsible for thread state from smid.
**CN:** 在模块级作用域中，这段头部声明了函数 `thread_state_from_smid(smid, *, device_index)`，它负责处理 thread state from smid 相关逻辑。

### Lines 62-63
```python
    if device_index is None:
        device_index = torch.cuda.current_device()
```
**EN:** Inside function `thread_state_from_smid`, this conditional checks `device_index is None` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `thread_state_from_smid` 内部，这段条件语句检查 `device_index is None`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 64-64
```python
    gs = global_state(device_index=device_index)
```
**EN:** Inside function `thread_state_from_smid`, this assignment updates `gs` with `global_state(device_index=device_index)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `thread_state_from_smid` 内部，这段赋值把 `global_state(device_index=device_index)` 写入 `gs`，为后续逻辑建立状态、别名或配置。

### Lines 65-69
```python
    return decode_thread_state_tensor(
        thread_state_tensor(smid, device_index=device_index),
        gs.num_threads,
        gs.clock_buffer_size,
    )
```
**EN:** Inside function `thread_state_from_smid`, this return statement sends `decode_thread_state_tensor(thread_state_tensor(smid, device_index=device_index), gs.num_threads, ...` back to the caller as the result of the current routine.
**CN:** 在函数 `thread_state_from_smid` 内部，这条返回语句把 `decode_thread_state_tensor(thread_state_tensor(smid, device_index=device_index), gs.num_threads, ...` 作为当前过程的结果返回给调用方。

### Lines 72-72
```python
def shadow_tensor_for(real: torch.Tensor) -> torch.Tensor:
```
**EN:** At module scope, this header declares the function `shadow_tensor_for(real)`, which is responsible for shadow tensor for.
**CN:** 在模块级作用域中，这段头部声明了函数 `shadow_tensor_for(real)`，它负责处理 shadow tensor for 相关逻辑。

### Lines 73-73
```python
    shadow_ptr = shadow_cell_address(real.data_ptr())
```
**EN:** Inside function `shadow_tensor_for`, this assignment updates `shadow_ptr` with `shadow_cell_address(real.data_ptr())`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `shadow_tensor_for` 内部，这段赋值把 `shadow_cell_address(real.data_ptr())` 写入 `shadow_ptr`，为后续逻辑建立状态、别名或配置。

### Lines 74-74
```python
    nbytes = real.untyped_storage().nbytes()
```
**EN:** Inside function `shadow_tensor_for`, this assignment updates `nbytes` with `real.untyped_storage().nbytes()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `shadow_tensor_for` 内部，这段赋值把 `real.untyped_storage().nbytes()` 写入 `nbytes`，为后续逻辑建立状态、别名或配置。

### Lines 75-75
```python
    num_cells = triton.cdiv(nbytes, SHADOW_GRANULARITY_BYTES)
```
**EN:** Inside function `shadow_tensor_for`, this assignment updates `num_cells` with `triton.cdiv(nbytes, SHADOW_GRANULARITY_BYTES)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `shadow_tensor_for` 内部，这段赋值把 `triton.cdiv(nbytes, SHADOW_GRANULARITY_BYTES)` 写入 `num_cells`，为后续逻辑建立状态、别名或配置。

### Lines 76-76
```python
    shadow_size = num_cells * SHADOW_CELL_SIZE_BYTES
```
**EN:** Inside function `shadow_tensor_for`, this assignment updates `shadow_size` with `num_cells * SHADOW_CELL_SIZE_BYTES`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `shadow_tensor_for` 内部，这段赋值把 `num_cells * SHADOW_CELL_SIZE_BYTES` 写入 `shadow_size`，为后续逻辑建立状态、别名或配置。

### Lines 77-77
```python
    return uint8_cuda_tensor_from_ptr(shadow_ptr, shadow_size, real.device.index)
```
**EN:** Inside function `shadow_tensor_for`, this return statement sends `uint8_cuda_tensor_from_ptr(shadow_ptr, shadow_size, real.device.index)` back to the caller as the result of the current routine.
**CN:** 在函数 `shadow_tensor_for` 内部，这条返回语句把 `uint8_cuda_tensor_from_ptr(shadow_ptr, shadow_size, real.device.index)` 作为当前过程的结果返回给调用方。

### Lines 80-81
```python
@triton.jit
def store_one_i32(ptr):
```
**EN:** At module scope, this header declares the function `store_one_i32(ptr)`, which is responsible for store one i32. Decorators: triton.jit.
**CN:** 在模块级作用域中，这段头部声明了函数 `store_one_i32(ptr)`，它负责处理 store one i32 相关逻辑。 装饰器包括：triton.jit。

### Lines 82-82
```python
    tl.store(ptr, 1)
```
**EN:** Inside function `store_one_i32`, this expression evaluates `tl.store` mainly for its side effects or registration behavior.
**CN:** 在函数 `store_one_i32` 内部，这条表达式计算 `tl.store`，主要目的是触发副作用或完成注册行为。

### Lines 85-86
```python
@triton.jit
def load_one_i32(ptr, out_ptr):
```
**EN:** At module scope, this header declares the function `load_one_i32(ptr, out_ptr)`, which is responsible for load one i32. Decorators: triton.jit.
**CN:** 在模块级作用域中，这段头部声明了函数 `load_one_i32(ptr, out_ptr)`，它负责处理 load one i32 相关逻辑。 装饰器包括：triton.jit。

### Lines 87-87
```python
    value = tl.load(ptr)
```
**EN:** Inside function `load_one_i32`, this assignment updates `value` with `tl.load(ptr)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `load_one_i32` 内部，这段赋值把 `tl.load(ptr)` 写入 `value`，为后续逻辑建立状态、别名或配置。

### Lines 88-88
```python
    tl.store(out_ptr, value)
```
**EN:** Inside function `load_one_i32`, this expression evaluates `tl.store` mainly for its side effects or registration behavior.
**CN:** 在函数 `load_one_i32` 内部，这条表达式计算 `tl.store`，主要目的是触发副作用或完成注册行为。

## Key Concepts / 关键概念

- **EN:** Path theme: `python/triton/experimental/gsan` places this module in Triton's triton / experimental / gsan area.
  **CN:** 路径主题：`python/triton/experimental/gsan` 表明该模块位于 Triton 的 triton / experimental / gsan 领域。
- **EN:** Primary functions: `nanosleep`, `atomic_poll`, `shadow_cell_tensor_from_address`, `shadow_cell_from_address`, `global_state_tensor`, `global_state`, `thread_state_tensor`, `thread_state_from_smid`, `shadow_tensor_for`, `store_one_i32`.
  **CN:** 主要函数：`nanosleep`, `atomic_poll`, `shadow_cell_tensor_from_address`, `shadow_cell_from_address`, `global_state_tensor`, `global_state`, `thread_state_tensor`, `thread_state_from_smid`, `shadow_tensor_for`, `store_one_i32`。
- **EN:** Experimental API: this module lives under `experimental`, so its interfaces may evolve quickly.
  **CN:** 实验性 API：该模块位于 `experimental` 下，其接口可能会快速演进。

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: __future__, torch.
  **CN:** 标准库依赖：__future__, torch。
- **EN:** Internal Triton modules: triton, triton.language, ._allocator, ._testing, ._utils.
  **CN:** Triton 内部模块：triton, triton.language, ._allocator, ._testing, ._utils。
- **EN:** Native/C-extension bindings: triton._C.libtriton.gsan_testing.
  **CN:** 原生/C 扩展绑定：triton._C.libtriton.gsan_testing。
