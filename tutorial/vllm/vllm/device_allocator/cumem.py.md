# cumem.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/device_allocator/cumem.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements cumem support for the `device_allocator` portion of vLLM. / 为 vLLM 的 `device_allocator` 子目录实现与 cumem 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 11-22)
```python
import dataclasses

import gc

import os

from collections.abc import Callable, Iterator

from contextlib import contextmanager

from typing import Any

import torch

from vllm.logger import init_logger

from vllm.utils.platform_utils import is_pin_memory_available

from vllm.utils.system_utils import find_loaded_library
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 24-48)
```python
logger = init_logger(__name__)

cumem_available = False

libcudart: Any = None

HandleType = tuple[int, int, int, int]
```
**EN:** This constant/configuration block defines `logger`, `cumem_available`, `libcudart`, `HandleType`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `logger`, `cumem_available`, `libcudart`, `HandleType`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Class `AllocationData` (lines 52-55)
```python
class AllocationData:
    handle: HandleType
    tag: str
    cpu_backup_tensor: torch.Tensor | None = None
```
**EN:** Class `AllocationData` is a structured building block in this module and uses a dataclass-style declaration to store explicit state.
**CN:** 类 `AllocationData` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。

### Function `create_and_map` (lines 58-59)
```python
def create_and_map(allocation_handle: HandleType) -> None:
    python_create_and_map(*allocation_handle)
```
**EN:** Function `create_and_map` constructs derived objects, runtime state, or helper structures. Key calls such as `python_create_and_map` show the concrete execution path.
**CN:** Function `create_and_map` 负责构造派生对象、运行时状态或辅助结构。 像 `python_create_and_map` 这样的关键调用展示了该代码块的具体执行路径。

### Function `unmap_and_release` (lines 62-63)
```python
def unmap_and_release(allocation_handle: HandleType) -> None:
    python_unmap_and_release(*allocation_handle)
```
**EN:** Function `unmap_and_release` provides a reusable helper around the module's main workflow. Key calls such as `python_unmap_and_release` show the concrete execution path.
**CN:** Function `unmap_and_release` 为模块主流程提供可复用的辅助逻辑。 像 `python_unmap_and_release` 这样的关键调用展示了该代码块的具体执行路径。

### Function `get_pluggable_allocator` (lines 66-74)
```python
def get_pluggable_allocator(
    python_malloc_fn: Callable[[HandleType], None],
    python_free_func: Callable[[int], HandleType],
) -> torch.cuda.memory.CUDAPluggableAllocator:
    init_module(python_malloc_fn, python_free_func)
    new_alloc = torch.cuda.memory.CUDAPluggableAllocator(
        lib_name, "my_malloc", "my_free"
    )
    return new_alloc
```
**EN:** Function `get_pluggable_allocator` provides a reusable helper around the module's main workflow. Key calls such as `init_module`, `torch.cuda.memory.CUDAPluggableAllocator` show the concrete execution path.
**CN:** Function `get_pluggable_allocator` 为模块主流程提供可复用的辅助逻辑。 像 `init_module`, `torch.cuda.memory.CUDAPluggableAllocator` 这样的关键调用展示了该代码块的具体执行路径。

### Function `use_memory_pool_with_allocator` (lines 78-87)
```python
def use_memory_pool_with_allocator(
    python_malloc_fn: Callable[[HandleType], None],
    python_free_func: Callable[[int], HandleType],
) -> Iterator[
    tuple[torch.cuda.memory.MemPool, torch.cuda.memory.CUDAPluggableAllocator]
]:
    new_alloc = get_pluggable_allocator(python_malloc_fn, python_free_func)
    mem_pool = torch.cuda.memory.MemPool(new_alloc._allocator)
    with torch.cuda.memory.use_mem_pool(mem_pool):
        yield mem_pool, new_alloc
```
**EN:** Function `use_memory_pool_with_allocator` provides a reusable helper around the module's main workflow. Key calls such as `get_pluggable_allocator`, `torch.cuda.memory.MemPool`, `torch.cuda.memory.use_mem_pool` show the concrete execution path.
**CN:** Function `use_memory_pool_with_allocator` 为模块主流程提供可复用的辅助逻辑。 像 `get_pluggable_allocator`, `torch.cuda.memory.MemPool`, `torch.cuda.memory.use_mem_pool` 这样的关键调用展示了该代码块的具体执行路径。

### Class `CuMemAllocator` (lines 90-312)
```python
class CuMemAllocator:
    """
    A singleton class that manages a memory pool for CUDA tensors.
    The memory in this pool can be offloaded or discarded when the
    allocator sleeps.

    Inside the `use_memory_pool(tag)` context, all tensors created will
    be allocated in the memory pool, and has the same tag as the
    tag passed to the context.

    When we call `sleep`, all tensors with the specified tag will be
    offloaded to CPU memory, and the rest of the tensors will be discarded.
    When we call `wake_up`, all tensors that are previously offloaded
    will be loaded back to GPU memory, and the rest of the tensors will
    have empty memory.

    Why it needs to be a singleton?
    When allocated tensors are garbage collected, PyTorch will call
    the free callback, which will call the `python_free_callback` method.
    The C-extension uses a global variable to store the function of an
    instance of this class. If we create multiple instances of this class,
    the global variable will be overwritten and the free callback will
    not work as expected.
    # ... omitted for brevity ...
            sum_bytes += handle[1]
        return sum_bytes
```
**EN:** Class `CuMemAllocator` is a structured building block in this module. Key methods include `get_instance`, `__init__`, `_python_malloc_callback`, `_python_free_callback`, `sleep`, `wake_up`, which define initialization, validation, transformation, or access patterns. The class docstring says: A singleton class that manages a memory pool for CUDA tensors.
**CN:** 类 `CuMemAllocator` 是该模块中的结构化构件。 关键方法包括 `get_instance`, `__init__`, `_python_malloc_callback`, `_python_free_callback`, `sleep`, `wake_up`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：A singleton class that manages a memory pool for CUDA tensors.

### Method `CuMemAllocator.get_instance` (lines 119-128)
```python
    def get_instance() -> "CuMemAllocator":
        """
        CuMemAllocator is a singleton class.
        We cannot call the constructor directly.
        Call this method to get the instance.
        """
        assert cumem_available, "cumem allocator is not available"
        if CuMemAllocator.instance is None:
            CuMemAllocator.instance = CuMemAllocator()
        return CuMemAllocator.instance
```
**EN:** Method `CuMemAllocator.get_instance` provides a reusable helper around the module's main workflow. The docstring highlights: CuMemAllocator is a singleton class. Key calls such as `CuMemAllocator` show the concrete execution path.
**CN:** Method `CuMemAllocator.get_instance` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：CuMemAllocator is a singleton class. 像 `CuMemAllocator` 这样的关键调用展示了该代码块的具体执行路径。

### Method `CuMemAllocator.__init__` (lines 130-138)
```python
    def __init__(self):
        self.pointer_to_data: dict[int, AllocationData] = {}
        self.current_tag: str = CuMemAllocator.default_tag
        self.allocator_and_pools: dict[str, Any] = {}
        # Creating strong references to the two callbacks here to prevent
        # these ephemeral bound-method objects being garbage collected.
        # See discussions in https://github.com/vllm-project/vllm/pull/22724
        self.python_malloc_callback = self._python_malloc_callback
        self.python_free_callback = self._python_free_callback
```
**EN:** Method `CuMemAllocator.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls.
**CN:** Method `CuMemAllocator.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。

### Method `CuMemAllocator.sleep` (lines 171-219)
```python
    def sleep(self, offload_tags: tuple[str, ...] | str | None = None) -> None:
        """
        Put the allocator in sleep mode.
        All data in the memory allocation with the specified tag will be
        offloaded to CPU memory, and others will be discarded.

        :param offload_tags: The tags of the memory allocation that will be
            offloaded. The rest of the memory allocation will be discarded.
        """
        if offload_tags is None:
            # by default, allocated tensors are offloaded
            # when the allocator sleeps
            offload_tags = (CuMemAllocator.default_tag,)
        elif isinstance(offload_tags, str):
            offload_tags = (offload_tags,)

        assert isinstance(offload_tags, tuple)

        total_bytes = 0
    # ... omitted for brevity ...
        gc.collect()
        torch.cuda.empty_cache()
```
**EN:** Method `CuMemAllocator.sleep` provides a reusable helper around the module's main workflow. The docstring highlights: Put the allocator in sleep mode. Key calls such as `isinstance`, `self.pointer_to_data.items`, `torch.empty`, `is_pin_memory_available`, `cpu_backup_tensor.data_ptr` show the concrete execution path.
**CN:** Method `CuMemAllocator.sleep` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Put the allocator in sleep mode. 像 `isinstance`, `self.pointer_to_data.items`, `torch.empty`, `is_pin_memory_available`, `cpu_backup_tensor.data_ptr` 这样的关键调用展示了该代码块的具体执行路径。

### Method `CuMemAllocator.wake_up` (lines 221-243)
```python
    def wake_up(self, tags: list[str] | None = None) -> None:
        """
        Wake up the allocator from sleep mode.
        All data that is previously offloaded will be loaded back to GPU
        memory, and the rest of the data will have empty memory.

        :param tags: The tags of the memory allocation that will be loaded
            back to GPU memory. If None, all memory allocation will be loaded
            back to GPU memory.
        """
        for ptr, data in self.pointer_to_data.items():
            if tags is None or data.tag in tags:
                handle = data.handle
                create_and_map(handle)
                if data.cpu_backup_tensor is not None:
                    cpu_backup_tensor = data.cpu_backup_tensor
                    if cpu_backup_tensor is not None:
                        size_in_bytes = (
                            cpu_backup_tensor.numel() * cpu_backup_tensor.element_size()
    # ... omitted for brevity ...
                        libcudart.cudaMemcpy(ptr, cpu_ptr, size_in_bytes)
                        data.cpu_backup_tensor = None
```
**EN:** Method `CuMemAllocator.wake_up` provides a reusable helper around the module's main workflow. The docstring highlights: Wake up the allocator from sleep mode. Key calls such as `self.pointer_to_data.items`, `create_and_map`, `cpu_backup_tensor.numel`, `cpu_backup_tensor.element_size`, `cpu_backup_tensor.data_ptr` show the concrete execution path.
**CN:** Method `CuMemAllocator.wake_up` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Wake up the allocator from sleep mode. 像 `self.pointer_to_data.items`, `create_and_map`, `cpu_backup_tensor.numel`, `cpu_backup_tensor.element_size`, `cpu_backup_tensor.data_ptr` 这样的关键调用展示了该代码块的具体执行路径。

### Method `CuMemAllocator.use_memory_pool` (lines 246-302)
```python
    def use_memory_pool(self, tag: str | None = None):
        """
        A context manager to use the memory pool.
        All memory allocation created inside the context will be allocated
        in the memory pool, and has the specified tag.

        :param tag: The tag of the memory allocation. If None, the default tag
            will be used.
        """
        if tag is None:
            tag = CuMemAllocator.default_tag

        assert isinstance(tag, str)

        # Expandable segments are incompatible with the memory pool used for
        # sleep mode (see https://github.com/pytorch/pytorch/issues/147851).
        # If the user has enabled expandable segments via
        # PYTORCH_CUDA_ALLOC_CONF, temporarily disable them for the duration
        # of the memory pool context and restore on exit.
    # ... omitted for brevity ...
            if expandable_was_enabled:
                torch.cuda.memory._set_allocator_settings("expandable_segments:True")
```
**EN:** Method `CuMemAllocator.use_memory_pool` provides a reusable helper around the module's main workflow. The docstring highlights: A context manager to use the memory pool. Key calls such as `isinstance`, `os.environ.get`, `torch.cuda.memory._set_allocator_settings`, `use_memory_pool_with_allocator`, `data[0].snapshot` show the concrete execution path.
**CN:** Method `CuMemAllocator.use_memory_pool` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：A context manager to use the memory pool. 像 `isinstance`, `os.environ.get`, `torch.cuda.memory._set_allocator_settings`, `use_memory_pool_with_allocator`, `data[0].snapshot` 这样的关键调用展示了该代码块的具体执行路径。

### Method `CuMemAllocator.get_current_usage` (lines 304-312)
```python
    def get_current_usage(self) -> int:
        """
        Get the total number of bytes allocated in the memory pool.
        """
        sum_bytes: int = 0
        for ptr, data in self.pointer_to_data.items():
            handle = data.handle
            sum_bytes += handle[1]
        return sum_bytes
```
**EN:** Method `CuMemAllocator.get_current_usage` provides a reusable helper around the module's main workflow. The docstring highlights: Get the total number of bytes allocated in the memory pool. Key calls such as `self.pointer_to_data.items` show the concrete execution path.
**CN:** Method `CuMemAllocator.get_current_usage` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Get the total number of bytes allocated in the memory pool. 像 `self.pointer_to_data.items` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **Structured types / 结构化类型**
  - **EN:** Dataclasses, typed dictionaries, or aliases are used to make runtime contracts explicit.
  - **CN:** 通过 dataclass、类型化字典或别名来显式表达运行时契约。
- **Caching / 缓存**
  - **EN:** Some definitions are designed to reuse computed state and avoid repeated work.
  - **CN:** 部分定义旨在复用已计算状态，避免重复工作。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import dataclasses`, `import gc`, `import os`, `from collections.abc import Callable, Iterator`, `from contextlib import contextmanager`, `from typing import Any`
- **Third-party / 第三方**: `import torch`
- **vLLM internal / vLLM 内部依赖**: `from vllm.logger import init_logger`, `from vllm.utils.platform_utils import is_pin_memory_available`, `from vllm.utils.system_utils import find_loaded_library`
