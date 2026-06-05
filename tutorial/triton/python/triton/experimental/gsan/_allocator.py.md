# _allocator.py — Code Analysis / 代码分析

## Source / 来源

- **Path:** `./python/triton/experimental/gsan/_allocator.py`
- **EN:** This source file at `./python/triton/experimental/gsan/_allocator.py` defines the main symbols `_load_gsan_module`, `_compile_gsan_allocator`, `get_allocator`, `create_mem_pool` for the surrounding Triton subsystem.
- **CN:** 位于 `./python/triton/experimental/gsan/_allocator.py` 的这个源文件为周边 Triton 子系统定义了主要符号 `_load_gsan_module`, `_compile_gsan_allocator`, `get_allocator`, `create_mem_pool`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1
```python
from __future__ import annotations
```
**EN:** At module scope, this block imports annotations from `__future__` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `__future__` 导入 annotations，把当前文件与周边 API 和辅助工具连接起来。

### Lines 3-3
```python
import functools
```
**EN:** At module scope, this block imports functools so later definitions can reuse those modules or symbols.
**CN:** 在模块级作用域中，这段代码导入了 functools，供后续定义复用这些模块或符号。

### Lines 4-4
```python
from pathlib import Path
```
**EN:** At module scope, this block imports Path from `pathlib` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `pathlib` 导入 Path，把当前文件与周边 API 和辅助工具连接起来。

### Lines 5-5
```python
from types import ModuleType
```
**EN:** At module scope, this block imports ModuleType from `types` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `types` 导入 ModuleType，把当前文件与周边 API 和辅助工具连接起来。

### Lines 7-7
```python
from triton.runtime import driver as runtime_driver
```
**EN:** At module scope, this block imports driver as runtime_driver from `triton.runtime` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `triton.runtime` 导入 driver as runtime_driver，把当前文件与周边 API 和辅助工具连接起来。

### Lines 8-8
```python
from triton.runtime.build import compile_module_from_file
```
**EN:** At module scope, this block imports compile_module_from_file from `triton.runtime.build` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `triton.runtime.build` 导入 compile_module_from_file，把当前文件与周边 API 和辅助工具连接起来。

### Lines 10-10
```python
_THIS_DIR = Path(__file__).resolve().parent
```
**EN:** At module scope, this assignment updates `_THIS_DIR` with `Path(__file__).resolve().parent`, establishing state, aliases, or configuration used later.
**CN:** 在模块级作用域中，这段赋值把 `Path(__file__).resolve().parent` 写入 `_THIS_DIR`，为后续逻辑建立状态、别名或配置。

### Lines 11-11
```python
_GSAN_SOURCE_PATH = _THIS_DIR / "src" / "GSanAllocator.cc"
```
**EN:** At module scope, this assignment updates `_GSAN_SOURCE_PATH` with `_THIS_DIR / 'src' / 'GSanAllocator.cc'`, establishing state, aliases, or configuration used later.
**CN:** 在模块级作用域中，这段赋值把 `_THIS_DIR / 'src' / 'GSanAllocator.cc'` 写入 `_GSAN_SOURCE_PATH`，为后续逻辑建立状态、别名或配置。

### Lines 14-15
```python
@functools.lru_cache()
def _load_gsan_module() -> ModuleType:
```
**EN:** At module scope, this header declares the function `_load_gsan_module()`, which is responsible for load gsan module. Decorators: functools.lru_cache().
**CN:** 在模块级作用域中，这段头部声明了函数 `_load_gsan_module()`，它负责处理 load gsan module 相关逻辑。 装饰器包括：functools.lru_cache()。

### Lines 16-17
```python
    if runtime_driver.active.get_current_target().backend != "cuda":
        raise RuntimeError("GSan allocator requires the CUDA backend.")
```
**EN:** Inside function `_load_gsan_module`, this conditional checks `runtime_driver.active.get_current_target().backend != 'cuda'` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `_load_gsan_module` 内部，这段条件语句检查 `runtime_driver.active.get_current_target().backend != 'cuda'`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 19-19
```python
    from triton.backends.nvidia.driver import library_dirs, include_dirs
```
**EN:** Inside function `_load_gsan_module`, this block imports library_dirs, include_dirs from `triton.backends.nvidia.driver` to connect this file with nearby APIs and helpers.
**CN:** 在函数 `_load_gsan_module` 内部，这段代码从 `triton.backends.nvidia.driver` 导入 library_dirs, include_dirs，把当前文件与周边 API 和辅助工具连接起来。

### Lines 21-27
```python
    return compile_module_from_file(
        src_path=str(_GSAN_SOURCE_PATH),
        name="gsan_allocator",
        library_dirs=library_dirs(),
        include_dirs=include_dirs,
        libraries=["libcuda.so.1"],
    )
```
**EN:** Inside function `_load_gsan_module`, this return statement sends `compile_module_from_file(src_path=str(_GSAN_SOURCE_PATH), name='gsan_allocator', library_dirs=lib...` back to the caller as the result of the current routine.
**CN:** 在函数 `_load_gsan_module` 内部，这条返回语句把 `compile_module_from_file(src_path=str(_GSAN_SOURCE_PATH), name='gsan_allocator', library_dirs=lib...` 作为当前过程的结果返回给调用方。

### Lines 30-32
```python
@functools.lru_cache()
def _compile_gsan_allocator() -> str:
    # __file__ for a compiled module is the so file
```
**EN:** At module scope, this header declares the function `_compile_gsan_allocator()`, which is responsible for compile gsan allocator. Decorators: functools.lru_cache().
**CN:** 在模块级作用域中，这段头部声明了函数 `_compile_gsan_allocator()`，它负责处理 compile gsan allocator 相关逻辑。 装饰器包括：functools.lru_cache()。

### Lines 33-33
```python
    return _load_gsan_module().__file__
```
**EN:** Inside function `_compile_gsan_allocator`, this return statement sends `_load_gsan_module().__file__` back to the caller as the result of the current routine.
**CN:** 在函数 `_compile_gsan_allocator` 内部，这条返回语句把 `_load_gsan_module().__file__` 作为当前过程的结果返回给调用方。

### Lines 36-37
```python
@functools.lru_cache()
def get_allocator():
```
**EN:** At module scope, this header declares the function `get_allocator()`, which is responsible for get allocator. Decorators: functools.lru_cache().
**CN:** 在模块级作用域中，这段头部声明了函数 `get_allocator()`，它负责处理 get allocator 相关逻辑。 装饰器包括：functools.lru_cache()。

### Lines 38-38
```python
    from torch.cuda.memory import CUDAPluggableAllocator
```
**EN:** Inside function `get_allocator`, this block imports CUDAPluggableAllocator from `torch.cuda.memory` to connect this file with nearby APIs and helpers.
**CN:** 在函数 `get_allocator` 内部，这段代码从 `torch.cuda.memory` 导入 CUDAPluggableAllocator，把当前文件与周边 API 和辅助工具连接起来。

### Lines 39-39
```python
    so_name = _compile_gsan_allocator()
```
**EN:** Inside function `get_allocator`, this assignment updates `so_name` with `_compile_gsan_allocator()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `get_allocator` 内部，这段赋值把 `_compile_gsan_allocator()` 写入 `so_name`，为后续逻辑建立状态、别名或配置。

### Lines 40-40
```python
    return CUDAPluggableAllocator(so_name, "gsanMalloc", "gsanFree")
```
**EN:** Inside function `get_allocator`, this return statement sends `CUDAPluggableAllocator(so_name, 'gsanMalloc', 'gsanFree')` back to the caller as the result of the current routine.
**CN:** 在函数 `get_allocator` 内部，这条返回语句把 `CUDAPluggableAllocator(so_name, 'gsanMalloc', 'gsanFree')` 作为当前过程的结果返回给调用方。

### Lines 43-43
```python
def create_mem_pool():
```
**EN:** At module scope, this header declares the function `create_mem_pool()`, which is responsible for create mem pool.
**CN:** 在模块级作用域中，这段头部声明了函数 `create_mem_pool()`，它负责处理 create mem pool 相关逻辑。

### Lines 44-44
```python
    from torch.cuda.memory import MemPool
```
**EN:** Inside function `create_mem_pool`, this block imports MemPool from `torch.cuda.memory` to connect this file with nearby APIs and helpers.
**CN:** 在函数 `create_mem_pool` 内部，这段代码从 `torch.cuda.memory` 导入 MemPool，把当前文件与周边 API 和辅助工具连接起来。

### Lines 45-45
```python
    return MemPool(get_allocator().allocator())
```
**EN:** Inside function `create_mem_pool`, this return statement sends `MemPool(get_allocator().allocator())` back to the caller as the result of the current routine.
**CN:** 在函数 `create_mem_pool` 内部，这条返回语句把 `MemPool(get_allocator().allocator())` 作为当前过程的结果返回给调用方。

### Lines 48-48
```python
def gsan_malloc(size: int, device: int, stream: int = 0) -> int:
```
**EN:** At module scope, this header declares the function `gsan_malloc(size, device, stream)`, which is responsible for gsan malloc.
**CN:** 在模块级作用域中，这段头部声明了函数 `gsan_malloc(size, device, stream)`，它负责处理 gsan malloc 相关逻辑。

### Lines 49-49
```python
    module = _load_gsan_module()
```
**EN:** Inside function `gsan_malloc`, this assignment updates `module` with `_load_gsan_module()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `gsan_malloc` 内部，这段赋值把 `_load_gsan_module()` 写入 `module`，为后续逻辑建立状态、别名或配置。

### Lines 50-50
```python
    return module.malloc(size, device, stream)
```
**EN:** Inside function `gsan_malloc`, this return statement sends `module.malloc(size, device, stream)` back to the caller as the result of the current routine.
**CN:** 在函数 `gsan_malloc` 内部，这条返回语句把 `module.malloc(size, device, stream)` 作为当前过程的结果返回给调用方。

### Lines 53-53
```python
def gsan_free(ptr: int, device: int, size: int = 0, stream: int = 0) -> None:
```
**EN:** At module scope, this header declares the function `gsan_free(ptr, device, size, stream)`, which is responsible for gsan free.
**CN:** 在模块级作用域中，这段头部声明了函数 `gsan_free(ptr, device, size, stream)`，它负责处理 gsan free 相关逻辑。

### Lines 54-54
```python
    module = _load_gsan_module()
```
**EN:** Inside function `gsan_free`, this assignment updates `module` with `_load_gsan_module()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `gsan_free` 内部，这段赋值把 `_load_gsan_module()` 写入 `module`，为后续逻辑建立状态、别名或配置。

### Lines 55-55
```python
    module.free(ptr, device, size, stream)
```
**EN:** Inside function `gsan_free`, this expression evaluates `module.free` mainly for its side effects or registration behavior.
**CN:** 在函数 `gsan_free` 内部，这条表达式计算 `module.free`，主要目的是触发副作用或完成注册行为。

### Lines 58-58
```python
def get_reserve_pointer() -> int:
```
**EN:** At module scope, this header declares the function `get_reserve_pointer()`, which is responsible for get reserve pointer.
**CN:** 在模块级作用域中，这段头部声明了函数 `get_reserve_pointer()`，它负责处理 get reserve pointer 相关逻辑。

### Lines 59-59
```python
    return _load_gsan_module().get_reserve_pointer()
```
**EN:** Inside function `get_reserve_pointer`, this return statement sends `_load_gsan_module().get_reserve_pointer()` back to the caller as the result of the current routine.
**CN:** 在函数 `get_reserve_pointer` 内部，这条返回语句把 `_load_gsan_module().get_reserve_pointer()` 作为当前过程的结果返回给调用方。

### Lines 62-62
```python
def get_reserve_size() -> int:
```
**EN:** At module scope, this header declares the function `get_reserve_size()`, which is responsible for get reserve size.
**CN:** 在模块级作用域中，这段头部声明了函数 `get_reserve_size()`，它负责处理 get reserve size 相关逻辑。

### Lines 63-63
```python
    return _load_gsan_module().get_reserve_size()
```
**EN:** Inside function `get_reserve_size`, this return statement sends `_load_gsan_module().get_reserve_size()` back to the caller as the result of the current routine.
**CN:** 在函数 `get_reserve_size` 内部，这条返回语句把 `_load_gsan_module().get_reserve_size()` 作为当前过程的结果返回给调用方。

### Lines 66-66
```python
def get_global_state_pointer() -> int:
```
**EN:** At module scope, this header declares the function `get_global_state_pointer()`, which is responsible for get global state pointer.
**CN:** 在模块级作用域中，这段头部声明了函数 `get_global_state_pointer()`，它负责处理 get global state pointer 相关逻辑。

### Lines 67-67
```python
    return _load_gsan_module().get_global_state_pointer()
```
**EN:** Inside function `get_global_state_pointer`, this return statement sends `_load_gsan_module().get_global_state_pointer()` back to the caller as the result of the current routine.
**CN:** 在函数 `get_global_state_pointer` 内部，这条返回语句把 `_load_gsan_module().get_global_state_pointer()` 作为当前过程的结果返回给调用方。

### Lines 70-70
```python
def get_runtime_state_layout(device: int) -> dict[str, int]:
```
**EN:** At module scope, this header declares the function `get_runtime_state_layout(device)`, which is responsible for get runtime state layout.
**CN:** 在模块级作用域中，这段头部声明了函数 `get_runtime_state_layout(device)`，它负责处理 get runtime state layout 相关逻辑。

### Lines 71-71
```python
    module = _load_gsan_module()
```
**EN:** Inside function `get_runtime_state_layout`, this assignment updates `module` with `_load_gsan_module()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `get_runtime_state_layout` 内部，这段赋值把 `_load_gsan_module()` 写入 `module`，为后续逻辑建立状态、别名或配置。

### Lines 72-72
```python
    return module.get_runtime_state_layout(device)
```
**EN:** Inside function `get_runtime_state_layout`, this return statement sends `module.get_runtime_state_layout(device)` back to the caller as the result of the current routine.
**CN:** 在函数 `get_runtime_state_layout` 内部，这条返回语句把 `module.get_runtime_state_layout(device)` 作为当前过程的结果返回给调用方。

### Lines 75-75
```python
def export_allocation_handles(ptr: int) -> tuple[int, int, int]:
```
**EN:** At module scope, this header declares the function `export_allocation_handles(ptr)`, which is responsible for export allocation handles.
**CN:** 在模块级作用域中，这段头部声明了函数 `export_allocation_handles(ptr)`，它负责处理 export allocation handles 相关逻辑。

### Lines 76-76
```python
    module = _load_gsan_module()
```
**EN:** Inside function `export_allocation_handles`, this assignment updates `module` with `_load_gsan_module()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `export_allocation_handles` 内部，这段赋值把 `_load_gsan_module()` 写入 `module`，为后续逻辑建立状态、别名或配置。

### Lines 77-77
```python
    return module.export_allocation_handles(ptr)
```
**EN:** Inside function `export_allocation_handles`, this return statement sends `module.export_allocation_handles(ptr)` back to the caller as the result of the current routine.
**CN:** 在函数 `export_allocation_handles` 内部，这条返回语句把 `module.export_allocation_handles(ptr)` 作为当前过程的结果返回给调用方。

### Lines 80-80
```python
def import_allocation_handles(real_fd: int, shadow_fd: int, alloc_size: int, device: int) -> int:
```
**EN:** At module scope, this header declares the function `import_allocation_handles(real_fd, shadow_fd, alloc_size, device)`, which is responsible for import allocation handles.
**CN:** 在模块级作用域中，这段头部声明了函数 `import_allocation_handles(real_fd, shadow_fd, alloc_size, device)`，它负责处理 import allocation handles 相关逻辑。

### Lines 81-81
```python
    module = _load_gsan_module()
```
**EN:** Inside function `import_allocation_handles`, this assignment updates `module` with `_load_gsan_module()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `import_allocation_handles` 内部，这段赋值把 `_load_gsan_module()` 写入 `module`，为后续逻辑建立状态、别名或配置。

### Lines 82-82
```python
    return module.import_allocation_handles(real_fd, shadow_fd, alloc_size, device)
```
**EN:** Inside function `import_allocation_handles`, this return statement sends `module.import_allocation_handles(real_fd, shadow_fd, alloc_size, device)` back to the caller as the result of the current routine.
**CN:** 在函数 `import_allocation_handles` 内部，这条返回语句把 `module.import_allocation_handles(real_fd, shadow_fd, alloc_size, device)` 作为当前过程的结果返回给调用方。

### Lines 85-85
```python
def export_runtime_state_handle(device: int) -> tuple[int, int]:
```
**EN:** At module scope, this header declares the function `export_runtime_state_handle(device)`, which is responsible for export runtime state handle.
**CN:** 在模块级作用域中，这段头部声明了函数 `export_runtime_state_handle(device)`，它负责处理 export runtime state handle 相关逻辑。

### Lines 86-86
```python
    module = _load_gsan_module()
```
**EN:** Inside function `export_runtime_state_handle`, this assignment updates `module` with `_load_gsan_module()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `export_runtime_state_handle` 内部，这段赋值把 `_load_gsan_module()` 写入 `module`，为后续逻辑建立状态、别名或配置。

### Lines 87-87
```python
    return module.export_runtime_state_handle(device)
```
**EN:** Inside function `export_runtime_state_handle`, this return statement sends `module.export_runtime_state_handle(device)` back to the caller as the result of the current routine.
**CN:** 在函数 `export_runtime_state_handle` 内部，这条返回语句把 `module.export_runtime_state_handle(device)` 作为当前过程的结果返回给调用方。

### Lines 90-90
```python
def import_runtime_state_handle(fd: int, alloc_size: int, peer_device: int, device: int) -> None:
```
**EN:** At module scope, this header declares the function `import_runtime_state_handle(fd, alloc_size, peer_device, device)`, which is responsible for import runtime state handle.
**CN:** 在模块级作用域中，这段头部声明了函数 `import_runtime_state_handle(fd, alloc_size, peer_device, device)`，它负责处理 import runtime state handle 相关逻辑。

### Lines 91-91
```python
    module = _load_gsan_module()
```
**EN:** Inside function `import_runtime_state_handle`, this assignment updates `module` with `_load_gsan_module()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `import_runtime_state_handle` 内部，这段赋值把 `_load_gsan_module()` 写入 `module`，为后续逻辑建立状态、别名或配置。

### Lines 92-92
```python
    module.import_runtime_state_handle(fd, alloc_size, peer_device, device)
```
**EN:** Inside function `import_runtime_state_handle`, this expression evaluates `module.import_runtime_state_handle` mainly for its side effects or registration behavior.
**CN:** 在函数 `import_runtime_state_handle` 内部，这条表达式计算 `module.import_runtime_state_handle`，主要目的是触发副作用或完成注册行为。

### Lines 95-95
```python
def free_allocation(ptr: int, device: int) -> None:
```
**EN:** At module scope, this header declares the function `free_allocation(ptr, device)`, which is responsible for free allocation.
**CN:** 在模块级作用域中，这段头部声明了函数 `free_allocation(ptr, device)`，它负责处理 free allocation 相关逻辑。

### Lines 96-96
```python
    gsan_free(ptr, device, size=0, stream=0)
```
**EN:** Inside function `free_allocation`, this expression evaluates `gsan_free` mainly for its side effects or registration behavior.
**CN:** 在函数 `free_allocation` 内部，这条表达式计算 `gsan_free`，主要目的是触发副作用或完成注册行为。

## Key Concepts / 关键概念

- **EN:** Path theme: `python/triton/experimental/gsan` places this module in Triton's triton / experimental / gsan area.
  **CN:** 路径主题：`python/triton/experimental/gsan` 表明该模块位于 Triton 的 triton / experimental / gsan 领域。
- **EN:** Primary functions: `_load_gsan_module`, `_compile_gsan_allocator`, `get_allocator`, `create_mem_pool`, `gsan_malloc`, `gsan_free`, `get_reserve_pointer`, `get_reserve_size`, `get_global_state_pointer`, `get_runtime_state_layout`.
  **CN:** 主要函数：`_load_gsan_module`, `_compile_gsan_allocator`, `get_allocator`, `create_mem_pool`, `gsan_malloc`, `gsan_free`, `get_reserve_pointer`, `get_reserve_size`, `get_global_state_pointer`, `get_runtime_state_layout`。
- **EN:** Experimental API: this module lives under `experimental`, so its interfaces may evolve quickly.
  **CN:** 实验性 API：该模块位于 `experimental` 下，其接口可能会快速演进。

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: __future__, functools, pathlib, types, torch.cuda.memory.
  **CN:** 标准库依赖：__future__, functools, pathlib, types, torch.cuda.memory。
- **EN:** Internal Triton modules: triton.runtime, triton.runtime.build, triton.backends.nvidia.driver.
  **CN:** Triton 内部模块：triton.runtime, triton.runtime.build, triton.backends.nvidia.driver。
