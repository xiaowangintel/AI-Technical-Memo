# prefetch.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/offloader/prefetch.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Prefetch-based CPU offloading with async prefetching. / 该文件的核心目的为：Prefetch-based CPU offloading with async prefetching.

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 5-10)
```python
"""Prefetch-based CPU offloading with async prefetching.

Uses static buffers and event-based stream forking for torch.compile +
CUDA graph compatibility. Events allow the copy stream to join CUDA
graph captures, ensuring H2D copies are properly captured.
"""
```
**EN:** This opening docstring states the module's intent, expected usage, and any important assumptions before the implementation details begin.
**CN:** 开头的文档字符串先说明模块意图、预期用法以及实现展开前需要了解的重要假设。

### Imports (lines 12-24)
```python
from abc import ABC, abstractmethod

from collections.abc import Generator

from dataclasses import dataclass

from typing import Any

import torch

import torch.nn as nn

import vllm.model_executor.offloader.prefetch_ops  # noqa: F401

from vllm.logger import init_logger

from vllm.model_executor.offloader.base import BaseOffloader, should_pin_memory

from vllm.utils.torch_utils import get_dtype_size
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 26-26)
```python
logger = init_logger(__name__)
```
**EN:** This constant/configuration block defines `logger`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `logger`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Class `ParamInfo` (lines 30-57)
```python
class ParamInfo:
    """Metadata about an offloaded parameter."""

    name: str
    shape: tuple[int, ...]
    stride: tuple[int, ...]
    dtype: torch.dtype

    @property
    def key(self) -> tuple[str, tuple[int, ...], tuple[int, ...], torch.dtype]:
        """Unique key for buffer pool grouping.

        Includes parameter name to prevent different parameters with the same
        shape from sharing buffers within the same layer. Parameters with the
        same name across different layers will share buffers (via slots).

        Includes stride because parameters with same shape but different
        strides need separate buffers to preserve memory layout.
        """
        return (self.name, self.shape, self.stride, self.dtype)

    @property
    def num_bytes(self) -> int:
    # ... omitted for brevity ...
            numel *= dim
        return numel * get_dtype_size(self.dtype)
```
**EN:** Class `ParamInfo` is a structured building block in this module and uses a dataclass-style declaration to store explicit state. Key methods include `key`, `num_bytes`, which define initialization, validation, transformation, or access patterns. The class docstring says: Metadata about an offloaded parameter.
**CN:** 类 `ParamInfo` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `key`, `num_bytes`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Metadata about an offloaded parameter.

### Method `ParamInfo.key` (lines 39-49)
```python
    def key(self) -> tuple[str, tuple[int, ...], tuple[int, ...], torch.dtype]:
        """Unique key for buffer pool grouping.

        Includes parameter name to prevent different parameters with the same
        shape from sharing buffers within the same layer. Parameters with the
        same name across different layers will share buffers (via slots).

        Includes stride because parameters with same shape but different
        strides need separate buffers to preserve memory layout.
        """
        return (self.name, self.shape, self.stride, self.dtype)
```
**EN:** Method `ParamInfo.key` exposes a computed property so callers can access derived state without duplicating logic. The docstring highlights: Unique key for buffer pool grouping.
**CN:** Method `ParamInfo.key` 暴露计算属性，使调用方无需重复逻辑即可访问派生状态。 文档字符串强调：Unique key for buffer pool grouping.

### Method `ParamInfo.num_bytes` (lines 52-57)
```python
    def num_bytes(self) -> int:
        """Size in bytes."""
        numel = 1
        for dim in self.shape:
            numel *= dim
        return numel * get_dtype_size(self.dtype)
```
**EN:** Method `ParamInfo.num_bytes` exposes a computed property so callers can access derived state without duplicating logic. The docstring highlights: Size in bytes. Key calls such as `get_dtype_size` show the concrete execution path.
**CN:** Method `ParamInfo.num_bytes` 暴露计算属性，使调用方无需重复逻辑即可访问派生状态。 文档字符串强调：Size in bytes. 像 `get_dtype_size` 这样的关键调用展示了该代码块的具体执行路径。

### Class `StaticBufferPool` (lines 60-124)
```python
class StaticBufferPool:
    """Pre-allocated GPU buffer pool for offloaded parameters.

    Allocates slot_capacity copies of each unique parameter
    (name, shape, stride, dtype), allowing for double/triple buffering
    during prefetch.

    Buffer slots are reused circularly: layer N uses slot (N % slot_capacity).

    The key includes parameter name to prevent different parameters within
    the same layer from sharing buffers. Parameters with the same name
    across different layers share buffers via the slot mechanism.
    """

    def __init__(
        self,
        param_infos: list[ParamInfo],
        slot_capacity: int,
        device: torch.device,
    ):
        self.slot_capacity = slot_capacity
        self.total_bytes = 0
        self._device = device
    # ... omitted for brevity ...
        key = (name, shape, stride, dtype)
        return self._buffers[key][slot_idx % self.slot_capacity]
```
**EN:** Class `StaticBufferPool` is a structured building block in this module. Key methods include `__init__`, `get_buffer`, which define initialization, validation, transformation, or access patterns. The class docstring says: Pre-allocated GPU buffer pool for offloaded parameters.
**CN:** 类 `StaticBufferPool` 是该模块中的结构化构件。 关键方法包括 `__init__`, `get_buffer`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Pre-allocated GPU buffer pool for offloaded parameters.

### Method `StaticBufferPool.__init__` (lines 74-112)
```python
    def __init__(
        self,
        param_infos: list[ParamInfo],
        slot_capacity: int,
        device: torch.device,
    ):
        self.slot_capacity = slot_capacity
        self.total_bytes = 0
        self._device = device

        # Group by (shape, stride, dtype) - only allocate unique combinations
        unique_params: dict[tuple, ParamInfo] = {}
        for info in param_infos:
            if info.key not in unique_params:
                unique_params[info.key] = info

        # Allocate buffers: key -> list of tensors (one per slot)
        self._buffers: dict[tuple, list[torch.Tensor]] = {}
        for key, info in unique_params.items():
    # ... omitted for brevity ...
            self.total_bytes / 1e9,
        )
```
**EN:** Method `StaticBufferPool.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `unique_params.items`, `range`, `torch.empty_strided`, `slot_tensors.append`, `logger.debug` show the concrete execution path.
**CN:** Method `StaticBufferPool.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `unique_params.items`, `range`, `torch.empty_strided`, `slot_tensors.append`, `logger.debug` 这样的关键调用展示了该代码块的具体执行路径。

### Method `StaticBufferPool.get_buffer` (lines 114-124)
```python
    def get_buffer(
        self,
        name: str,
        shape: tuple[int, ...],
        stride: tuple[int, ...],
        dtype: torch.dtype,
        slot_idx: int,
    ) -> torch.Tensor:
        """Get a static buffer for the given name/shape/stride/dtype/slot."""
        key = (name, shape, stride, dtype)
        return self._buffers[key][slot_idx % self.slot_capacity]
```
**EN:** Method `StaticBufferPool.get_buffer` provides a reusable helper around the module's main workflow. The docstring highlights: Get a static buffer for the given name/shape/stride/dtype/slot.
**CN:** Method `StaticBufferPool.get_buffer` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Get a static buffer for the given name/shape/stride/dtype/slot.

### Class `PrefetchOffloader` (lines 127-365)
```python
class PrefetchOffloader(BaseOffloader):
    """Prefetching-based offloader with group-based layer selection.

    Groups layers and uses async H2D prefetch to hide transfer latency.
    Uses static buffers and stream synchronization for torch.compile and
    CUDA graph compatibility.

    Args:
        group_size: Group every N layers together.
        num_in_group: Offload this many layers per group (last N of each group).
        prefetch_step: Number of layers to prefetch ahead.
        mode: Offload mode ("cpu" is currently supported).
    """

    def __init__(
        self,
        group_size: int,
        num_in_group: int,
        prefetch_step: int,
        offload_params: set[str] | None = None,
        mode: str = "cpu",
    ):
        self.group_size = group_size
    # ... omitted for brevity ...
        for i in range(min(self.prefetch_step, len(self.module_offloaders))):
            self.module_offloaders[i].start_onload_to_static()
```
**EN:** Class `PrefetchOffloader` is a structured building block in this module. It inherits from `BaseOffloader`. Key methods include `__init__`, `wrap_modules`, `_hook_module_forward`, `_wait_for_layer`, `sync_prev_onload`, `_start_prefetch`, which define initialization, validation, transformation, or access patterns. The class docstring says: Prefetching-based offloader with group-based layer selection.
**CN:** 类 `PrefetchOffloader` 是该模块中的结构化构件，继承自 `BaseOffloader`。 关键方法包括 `__init__`, `wrap_modules`, `_hook_module_forward`, `_wait_for_layer`, `sync_prev_onload`, `_start_prefetch`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Prefetching-based offloader with group-based layer selection.

### Method `PrefetchOffloader.__init__` (lines 141-161)
```python
    def __init__(
        self,
        group_size: int,
        num_in_group: int,
        prefetch_step: int,
        offload_params: set[str] | None = None,
        mode: str = "cpu",
    ):
        self.group_size = group_size
        self.num_in_group = num_in_group
        self.prefetch_step = prefetch_step
        self.offload_params = offload_params or set()
        self.mode = mode

        # Copy stream for async H2D transfers
        self.copy_stream = torch.cuda.Stream()

        # Module offloaders and buffer pool (populated in wrap_modules/post_init)
        self.module_offloaders: list[_ModuleOffloader] = []
        self.buffer_pool: StaticBufferPool | None = None
        self.total_offloaded_bytes = 0
```
**EN:** Method `PrefetchOffloader.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `set`, `torch.cuda.Stream` show the concrete execution path.
**CN:** Method `PrefetchOffloader.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `set`, `torch.cuda.Stream` 这样的关键调用展示了该代码块的具体执行路径。

### Method `PrefetchOffloader.wrap_modules` (lines 163-207)
```python
    def wrap_modules(
        self,
        modules_generator: Generator[nn.Module, None, None],
    ) -> list[nn.Module]:
        """Wrap modules with prefetch offloading logic."""
        assert len(self.module_offloaders) == 0, (
            "wrap_modules should only be called once"
        )

        all_modules = []
        offload_modules = []

        for module_index, module in enumerate(modules_generator):
            all_modules.append(module)

            # Select layers to offload based on group pattern
            # Offload last num_in_group layers of each group_size
            if module_index % self.group_size >= self.group_size - self.num_in_group:
                if self.offload_params:
    # ... omitted for brevity ...

        return all_modules
```
**EN:** Method `PrefetchOffloader.wrap_modules` provides a reusable helper around the module's main workflow. The docstring highlights: Wrap modules with prefetch offloading logic. Key calls such as `len`, `enumerate`, `all_modules.append`, `module.named_parameters`, `any` show the concrete execution path.
**CN:** Method `PrefetchOffloader.wrap_modules` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Wrap modules with prefetch offloading logic. 像 `len`, `enumerate`, `all_modules.append`, `module.named_parameters`, `any` 这样的关键调用展示了该代码块的具体执行路径。

### Method `PrefetchOffloader._hook_module_forward` (lines 209-241)
```python
    def _hook_module_forward(self, index: int, module: nn.Module):
        """Hook module's forward with torch.compile-compatible sync."""
        original_forward = module.forward

        def forward(*args, **kwargs):
            # Temporarily restore original forward to avoid recursion
            module.forward = original_forward

            # Wait for this layer's prefetch to complete
            # mutates_args on input_tensor creates data dependency for torch.compile
            input_tensor = args[0] if args else kwargs.get("hidden_states")
            torch.ops.vllm.wait_prefetch(input_tensor, index)

            # No parameter swapping needed - parameters already point to
            # GPU static buffers (set in assign_static_buffer)
            output = original_forward(*args, **kwargs)

            # Start prefetch for next layer (circular)
            # mutates_args on output_tensor creates ordering dependency
    # ... omitted for brevity ...

        module.forward = forward
```
**EN:** Method `PrefetchOffloader._hook_module_forward` provides a reusable helper around the module's main workflow. The docstring highlights: Hook module's forward with torch.compile-compatible sync. Key calls such as `kwargs.get`, `torch.ops.vllm.wait_prefetch`, `original_forward`, `len`, `isinstance` show the concrete execution path.
**CN:** Method `PrefetchOffloader._hook_module_forward` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Hook module's forward with torch.compile-compatible sync. 像 `kwargs.get`, `torch.ops.vllm.wait_prefetch`, `original_forward`, `len`, `isinstance` 这样的关键调用展示了该代码块的具体执行路径。

### Method `PrefetchOffloader.sync_prev_onload` (lines 275-282)
```python
    def sync_prev_onload(self):
        """Sync previous onload operations.

        Ensures any H2D copies in flight on copy_stream complete before
        the compute stream continues. Call this before CUDA graph
        capture/replay or when synchronization is needed.
        """
        torch.cuda.current_stream().wait_stream(self.copy_stream)
```
**EN:** Method `PrefetchOffloader.sync_prev_onload` handles loading or retrieval of external/internal data. The docstring highlights: Sync previous onload operations. Key calls such as `torch.cuda.current_stream().wait_stream`, `torch.cuda.current_stream` show the concrete execution path.
**CN:** Method `PrefetchOffloader.sync_prev_onload` 负责加载或获取外部/内部数据。 文档字符串强调：Sync previous onload operations. 像 `torch.cuda.current_stream().wait_stream`, `torch.cuda.current_stream` 这样的关键调用展示了该代码块的具体执行路径。

### Method `PrefetchOffloader.join_after_forward` (lines 289-308)
```python
    def join_after_forward(self):
        """Join copy_stream after model forward completes.

        Call this after the model forward pass but before CUDA graph capture
        ends. This ensures copy_stream is rejoined for any prefetches started
        during the forward pass.

        We join ALL layers that have _prefetch_in_capture=True, meaning their
        prefetch was started during capture but not yet waited on (joined).
        This handles both full and piecewise cudagraph modes correctly:
        - Full mode: joins layers 0..prefetch_step-1 (prefetched by last layers)
        - Piecewise mode: joins only layers prefetched by THIS subgraph's layers
        """
        if not self.module_offloaders:
            return
        # Join all layers whose prefetch was started in capture but not waited on
        for offloader in self.module_offloaders:
            if offloader._prefetch_in_capture:
                torch.cuda.current_stream().wait_event(offloader._copy_done_event)
                offloader._prefetch_in_capture = False
```
**EN:** Method `PrefetchOffloader.join_after_forward` provides a reusable helper around the module's main workflow. The docstring highlights: Join copy_stream after model forward completes. Key calls such as `torch.cuda.current_stream().wait_event`, `torch.cuda.current_stream` show the concrete execution path.
**CN:** Method `PrefetchOffloader.join_after_forward` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Join copy_stream after model forward completes. 像 `torch.cuda.current_stream().wait_event`, `torch.cuda.current_stream` 这样的关键调用展示了该代码块的具体执行路径。

### Method `PrefetchOffloader.post_init` (lines 310-365)
```python
    def post_init(self):
        """Allocate static buffer pool and start initial prefetches.

        Note: Parameters have already been offloaded to CPU during wrap_modules()
        (in _CpuParamOffloader.__init__), so GPU memory is available for the
        static buffer pool.
        """
        # Sync CPU storage with current param.data BEFORE collecting param info.
        # This is needed because process_weights_after_loading may have:
        # 1. Transformed weights (quantization, transpose, etc.)
        # 2. Created new CPU tensors via device_loading_context
        # Our _cpu_storage would be stale otherwise.
        for offloader in self.module_offloaders:
            offloader.sync_cpu_storage()

        # Collect parameter info (now using synced CPU storage)
        param_infos: list[ParamInfo] = []
        device: torch.device | None = None

    # ... omitted for brevity ...
        for i in range(min(self.prefetch_step, len(self.module_offloaders))):
            self.module_offloaders[i].start_onload_to_static()
```
**EN:** Method `PrefetchOffloader.post_init` constructs derived objects, runtime state, or helper structures. The docstring highlights: Allocate static buffer pool and start initial prefetches. Key calls such as `offloader.sync_cpu_storage`, `param_infos.extend`, `offloader.get_param_infos`, `StaticBufferPool`, `enumerate` show the concrete execution path.
**CN:** Method `PrefetchOffloader.post_init` 负责构造派生对象、运行时状态或辅助结构。 文档字符串强调：Allocate static buffer pool and start initial prefetches. 像 `offloader.sync_cpu_storage`, `param_infos.extend`, `offloader.get_param_infos`, `StaticBufferPool`, `enumerate` 这样的关键调用展示了该代码块的具体执行路径。

### Class `_ModuleOffloader` (lines 368-543)
```python
class _ModuleOffloader:
    """Manages offloading for a single module.

    Uses static buffers from a shared pool instead of dynamic allocation.
    """

    def __init__(
        self,
        mode: str,
        module: nn.Module,
        copy_stream: torch.cuda.Stream,
        whitelist_param_names: list[str],
        layer_idx: int,
    ):
        self.mode = mode
        self.module = module
        self.device = next(module.parameters()).device
        self.copy_stream = copy_stream
        self.layer_idx = layer_idx
        self.offloaded_bytes = 0

        # Event to signal when H2D copy to static buffer is complete.
        # Used for per-layer synchronization (both eager and capture modes).
    # ... omitted for brevity ...
        # Events recorded during capture become invalid after capture ends.
        self._event_valid_for_eager = not torch.cuda.is_current_stream_capturing()
```
**EN:** Class `_ModuleOffloader` is a structured building block in this module. Key methods include `__init__`, `post_init`, `sync_cpu_storage`, `get_param_infos`, `assign_buffer_slot`, `start_onload_to_static`, which define initialization, validation, transformation, or access patterns. The class docstring says: Manages offloading for a single module.
**CN:** 类 `_ModuleOffloader` 是该模块中的结构化构件。 关键方法包括 `__init__`, `post_init`, `sync_cpu_storage`, `get_param_infos`, `assign_buffer_slot`, `start_onload_to_static`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Manages offloading for a single module.

### Method `_ModuleOffloader.__init__` (lines 374-421)
```python
    def __init__(
        self,
        mode: str,
        module: nn.Module,
        copy_stream: torch.cuda.Stream,
        whitelist_param_names: list[str],
        layer_idx: int,
    ):
        self.mode = mode
        self.module = module
        self.device = next(module.parameters()).device
        self.copy_stream = copy_stream
        self.layer_idx = layer_idx
        self.offloaded_bytes = 0

        # Event to signal when H2D copy to static buffer is complete.
        # Used for per-layer synchronization (both eager and capture modes).
        self._copy_done_event = torch.cuda.Event()

    # ... omitted for brevity ...
            for name in whitelist_param_names
        }
```
**EN:** Method `_ModuleOffloader.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `next`, `module.parameters`, `torch.cuda.Event`, `torch.device`, `dict` show the concrete execution path.
**CN:** Method `_ModuleOffloader.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `next`, `module.parameters`, `torch.cuda.Event`, `torch.device`, `dict` 这样的关键调用展示了该代码块的具体执行路径。

### Method `_ModuleOffloader.post_init` (lines 423-427)
```python
    def post_init(self):
        """Collect total offloaded bytes (offloading already done in __init__)."""
        for param_offloader in self._param_offloaders.values():
            param_offloader.post_init()
            self.offloaded_bytes += param_offloader.offloaded_bytes
```
**EN:** Method `_ModuleOffloader.post_init` constructs derived objects, runtime state, or helper structures. The docstring highlights: Collect total offloaded bytes (offloading already done in __init__). Key calls such as `self._param_offloaders.values`, `param_offloader.post_init` show the concrete execution path.
**CN:** Method `_ModuleOffloader.post_init` 负责构造派生对象、运行时状态或辅助结构。 文档字符串强调：Collect total offloaded bytes (offloading already done in __init__). 像 `self._param_offloaders.values`, `param_offloader.post_init` 这样的关键调用展示了该代码块的具体执行路径。

### Method `_ModuleOffloader.sync_cpu_storage` (lines 429-458)
```python
    def sync_cpu_storage(self):
        """Sync CPU storage with current param.data.

        Called after process_weights_after_loading to ensure _cpu_storage
        contains the final processed weights, not stale pre-loading data.

        Parameters whose underlying nn.Parameter was deleted by
        process_weights_after_loading (e.g. transient KV-cache scale params)
        are pruned from self._param_offloaders so they do not participate in
        buffer-pool allocation or prefetching.
        """
        for param_offloader in self._param_offloaders.values():
            param_offloader.sync_cpu_storage()

        # Remove offloaders whose parameter was deleted during
        # process_weights_after_loading (e.g. k_scale / v_scale).
        deleted = [
            name
            for name, offloader in self._param_offloaders.items()
    # ... omitted for brevity ...
            for name in deleted:
                del self._param_offloaders[name]
```
**EN:** Method `_ModuleOffloader.sync_cpu_storage` provides a reusable helper around the module's main workflow. The docstring highlights: Sync CPU storage with current param.data. Key calls such as `self._param_offloaders.values`, `param_offloader.sync_cpu_storage`, `self._param_offloaders.items`, `getattr`, `logger.debug` show the concrete execution path.
**CN:** Method `_ModuleOffloader.sync_cpu_storage` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Sync CPU storage with current param.data. 像 `self._param_offloaders.values`, `param_offloader.sync_cpu_storage`, `self._param_offloaders.items`, `getattr`, `logger.debug` 这样的关键调用展示了该代码块的具体执行路径。

### Method `_ModuleOffloader.get_param_infos` (lines 460-478)
```python
    def get_param_infos(self) -> list[ParamInfo]:
        """Get parameter metadata for buffer pool allocation.

        Note: sync_cpu_storage() must be called before this method to ensure
        _cpu_storage reflects the final processed weights (after quantization).
        """
        infos = []
        for name, offloader in self._param_offloaders.items():
            cpu_storage = offloader._cpu_storage
            assert cpu_storage is not None, "CPU storage not initialized"
            infos.append(
                ParamInfo(
                    name=name,
                    shape=tuple(cpu_storage.shape),
                    stride=tuple(cpu_storage.stride()),
                    dtype=cpu_storage.dtype,
                )
            )
        return infos
```
**EN:** Method `_ModuleOffloader.get_param_infos` provides a reusable helper around the module's main workflow. The docstring highlights: Get parameter metadata for buffer pool allocation. Key calls such as `self._param_offloaders.items`, `infos.append`, `ParamInfo`, `tuple`, `cpu_storage.stride` show the concrete execution path.
**CN:** Method `_ModuleOffloader.get_param_infos` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Get parameter metadata for buffer pool allocation. 像 `self._param_offloaders.items`, `infos.append`, `ParamInfo`, `tuple`, `cpu_storage.stride` 这样的关键调用展示了该代码块的具体执行路径。

### Method `_ModuleOffloader.assign_buffer_slot` (lines 480-501)
```python
    def assign_buffer_slot(self, pool: StaticBufferPool, slot_idx: int):
        """Assign this module to a buffer slot in the pool.

        Also assigns static GPU buffers to each parameter offloader,
        which moves the parameter data to point to the GPU buffer.
        """
        self._buffer_pool = pool
        self._buffer_slot_idx = slot_idx

        # Assign static buffers to parameters
        # Use CPU storage shape/stride/dtype since param.data is now empty
        for name, offloader in self._param_offloaders.items():
            cpu_storage = offloader._cpu_storage
            assert cpu_storage is not None, "CPU storage not initialized"
            buffer = pool.get_buffer(
                name=name,
                shape=tuple(cpu_storage.shape),
                stride=tuple(cpu_storage.stride()),
                dtype=cpu_storage.dtype,
                slot_idx=slot_idx,
            )
            offloader.assign_static_buffer(buffer)
```
**EN:** Method `_ModuleOffloader.assign_buffer_slot` provides a reusable helper around the module's main workflow. The docstring highlights: Assign this module to a buffer slot in the pool. Key calls such as `self._param_offloaders.items`, `pool.get_buffer`, `tuple`, `cpu_storage.stride`, `offloader.assign_static_buffer` show the concrete execution path.
**CN:** Method `_ModuleOffloader.assign_buffer_slot` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Assign this module to a buffer slot in the pool. 像 `self._param_offloaders.items`, `pool.get_buffer`, `tuple`, `cpu_storage.stride`, `offloader.assign_static_buffer` 这样的关键调用展示了该代码块的具体执行路径。

### Method `_ModuleOffloader.start_onload_to_static` (lines 503-543)
```python
    def start_onload_to_static(self):
        """Start async copy from CPU storage to GPU buffer.

        Uses event-based forking to join copy_stream to CUDA graph capture.
        This ensures H2D copies are properly captured when recording a graph.

        IMPORTANT: We must wait for the compute stream before copying, because
        the previous layer's forward may still be using the buffer (GPU ops are
        async). Without this sync, we could overwrite the buffer while it's
        being read.
        """
        assert self._buffer_pool is not None, "Buffer pool not assigned"

        # Track if this prefetch is being captured (for _wait_for_layer logic)
        self._prefetch_in_capture = torch.cuda.is_current_stream_capturing()

        # Fork: record event on compute stream, copy_stream waits on it
        # This joins copy_stream to any active CUDA graph capture
        fork_event = torch.cuda.Event()
    # ... omitted for brevity ...
        # Events recorded during capture become invalid after capture ends.
        self._event_valid_for_eager = not torch.cuda.is_current_stream_capturing()
```
**EN:** Method `_ModuleOffloader.start_onload_to_static` handles loading or retrieval of external/internal data. The docstring highlights: Start async copy from CPU storage to GPU buffer. Key calls such as `torch.cuda.is_current_stream_capturing`, `torch.cuda.Event`, `torch.cuda.current_stream().record_event`, `torch.cuda.current_stream`, `self.copy_stream.wait_event` show the concrete execution path.
**CN:** Method `_ModuleOffloader.start_onload_to_static` 负责加载或获取外部/内部数据。 文档字符串强调：Start async copy from CPU storage to GPU buffer. 像 `torch.cuda.is_current_stream_capturing`, `torch.cuda.Event`, `torch.cuda.current_stream().record_event`, `torch.cuda.current_stream`, `self.copy_stream.wait_event` 这样的关键调用展示了该代码块的具体执行路径。

### Class `_BaseParamOffloader` (lines 546-597)
```python
class _BaseParamOffloader(ABC):
    """Base class for parameter offloading strategies."""

    # CPU storage for offloaded parameters (set by subclasses)
    _cpu_storage: torch.Tensor | None
    # GPU buffer reference (set by subclasses when using static buffers)
    _gpu_buffer: torch.Tensor | None

    @staticmethod
    def create(mode: str, **kwargs) -> "_BaseParamOffloader":
        """Factory method to create appropriate offloader for mode."""
        if mode == "cpu":
            return _CpuParamOffloader(**kwargs)
        else:
            raise ValueError(f"Unknown offload mode: {mode}")

    def __init__(self, module: nn.Module, param_name: str):
        self._module = module
        self._param_name = param_name
        self.offloaded_bytes = 0
        self._cpu_storage = None
        self._gpu_buffer = None

    # ... omitted for brevity ...
        """Point parameter data to GPU static buffer."""
        pass
```
**EN:** Class `_BaseParamOffloader` is a structured building block in this module. It inherits from `ABC`. Key methods include `create`, `__init__`, `_param`, `post_init`, `sync_cpu_storage`, `assign_static_buffer`, which define initialization, validation, transformation, or access patterns. The class docstring says: Base class for parameter offloading strategies.
**CN:** 类 `_BaseParamOffloader` 是该模块中的结构化构件，继承自 `ABC`。 关键方法包括 `create`, `__init__`, `_param`, `post_init`, `sync_cpu_storage`, `assign_static_buffer`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Base class for parameter offloading strategies.

### Method `_BaseParamOffloader.create` (lines 555-560)
```python
    def create(mode: str, **kwargs) -> "_BaseParamOffloader":
        """Factory method to create appropriate offloader for mode."""
        if mode == "cpu":
            return _CpuParamOffloader(**kwargs)
        else:
            raise ValueError(f"Unknown offload mode: {mode}")
```
**EN:** Method `_BaseParamOffloader.create` constructs derived objects, runtime state, or helper structures. The docstring highlights: Factory method to create appropriate offloader for mode. Key calls such as `_CpuParamOffloader`, `ValueError` show the concrete execution path.
**CN:** Method `_BaseParamOffloader.create` 负责构造派生对象、运行时状态或辅助结构。 文档字符串强调：Factory method to create appropriate offloader for mode. 像 `_CpuParamOffloader`, `ValueError` 这样的关键调用展示了该代码块的具体执行路径。

### Method `_BaseParamOffloader.__init__` (lines 562-567)
```python
    def __init__(self, module: nn.Module, param_name: str):
        self._module = module
        self._param_name = param_name
        self.offloaded_bytes = 0
        self._cpu_storage = None
        self._gpu_buffer = None
```
**EN:** Method `_BaseParamOffloader.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls.
**CN:** Method `_BaseParamOffloader.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。

### Method `_BaseParamOffloader._param` (lines 570-579)
```python
    def _param(self) -> nn.Parameter:
        """Get the parameter being offloaded.

        Supports dotted names (e.g. 'self_attn.qkv_proj.weight') by
        traversing the module hierarchy.
        """
        obj: Any = self._module
        for attr in self._param_name.split("."):
            obj = getattr(obj, attr)
        return obj
```
**EN:** Method `_BaseParamOffloader._param` exposes a computed property so callers can access derived state without duplicating logic. The docstring highlights: Get the parameter being offloaded. Key calls such as `self._param_name.split`, `getattr` show the concrete execution path.
**CN:** Method `_BaseParamOffloader._param` 暴露计算属性，使调用方无需重复逻辑即可访问派生状态。 文档字符串强调：Get the parameter being offloaded. 像 `self._param_name.split`, `getattr` 这样的关键调用展示了该代码块的具体执行路径。

### Method `_BaseParamOffloader.post_init` (lines 581-583)
```python
    def post_init(self):
        """Initialize offloading (move parameter to storage)."""
        return
```
**EN:** Method `_BaseParamOffloader.post_init` constructs derived objects, runtime state, or helper structures. The docstring highlights: Initialize offloading (move parameter to storage).
**CN:** Method `_BaseParamOffloader.post_init` 负责构造派生对象、运行时状态或辅助结构。 文档字符串强调：Initialize offloading (move parameter to storage).

### Method `_BaseParamOffloader.sync_cpu_storage` (lines 586-592)
```python
    def sync_cpu_storage(self) -> None:
        """Sync CPU storage with current param.data.

        Called after process_weights_after_loading to update _cpu_storage
        with the final processed weights.
        """
        pass
```
**EN:** Method `_BaseParamOffloader.sync_cpu_storage` provides a reusable helper around the module's main workflow. The docstring highlights: Sync CPU storage with current param.data.
**CN:** Method `_BaseParamOffloader.sync_cpu_storage` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Sync CPU storage with current param.data.

### Method `_BaseParamOffloader.assign_static_buffer` (lines 595-597)
```python
    def assign_static_buffer(self, gpu_buffer: torch.Tensor) -> None:
        """Point parameter data to GPU static buffer."""
        pass
```
**EN:** Method `_BaseParamOffloader.assign_static_buffer` provides a reusable helper around the module's main workflow. The docstring highlights: Point parameter data to GPU static buffer.
**CN:** Method `_BaseParamOffloader.assign_static_buffer` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Point parameter data to GPU static buffer.

### Class `_CpuParamOffloader` (lines 600-745)
```python
class _CpuParamOffloader(_BaseParamOffloader):
    """Offload parameter to pinned CPU memory.

    Uses GPU static buffers as the actual parameter, with CPU storage
    kept separately. This ensures torch.compile sees GPU tensors at trace time.

    The offloading happens in two phases:
    1. __init__() - copies GPU data to CPU, frees GPU memory immediately
    2. assign_static_buffer() - points param.data to GPU static buffer
    """

    def __init__(self, module: nn.Module, param_name: str):
        super().__init__(module, param_name)
        self._cpu_storage: torch.Tensor | None = None
        self._gpu_buffer: torch.Tensor | None = None  # Store reference to GPU buffer
        # Set to True if the underlying nn.Parameter was deleted by
        # process_weights_after_loading (e.g. transient KV-cache scale params
        # such as k_scale/v_scale created by BaseKVCacheMethod.create_weights
        # and deleted after copying into permanent _k_scale buffers).
        self._param_deleted: bool = False

        # Offload to CPU immediately to free GPU memory during model loading
        self._offload_to_cpu_internal()
    # ... omitted for brevity ...
        """No-op: offloading done in offload_to_cpu/assign_static_buffer."""
        pass
```
**EN:** Class `_CpuParamOffloader` is a structured building block in this module. It inherits from `_BaseParamOffloader`. Key methods include `__init__`, `_offload_to_cpu_internal`, `_update_cpu_storage_from_param`, `assign_static_buffer`, `sync_cpu_storage`, `post_init`, which define initialization, validation, transformation, or access patterns. The class docstring says: Offload parameter to pinned CPU memory.
**CN:** 类 `_CpuParamOffloader` 是该模块中的结构化构件，继承自 `_BaseParamOffloader`。 关键方法包括 `__init__`, `_offload_to_cpu_internal`, `_update_cpu_storage_from_param`, `assign_static_buffer`, `sync_cpu_storage`, `post_init`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Offload parameter to pinned CPU memory.

### Method `_CpuParamOffloader.__init__` (lines 611-622)
```python
    def __init__(self, module: nn.Module, param_name: str):
        super().__init__(module, param_name)
        self._cpu_storage: torch.Tensor | None = None
        self._gpu_buffer: torch.Tensor | None = None  # Store reference to GPU buffer
        # Set to True if the underlying nn.Parameter was deleted by
        # process_weights_after_loading (e.g. transient KV-cache scale params
        # such as k_scale/v_scale created by BaseKVCacheMethod.create_weights
        # and deleted after copying into permanent _k_scale buffers).
        self._param_deleted: bool = False

        # Offload to CPU immediately to free GPU memory during model loading
        self._offload_to_cpu_internal()
```
**EN:** Method `_CpuParamOffloader.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `super().__init__`, `super`, `self._offload_to_cpu_internal` show the concrete execution path.
**CN:** Method `_CpuParamOffloader.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `super().__init__`, `super`, `self._offload_to_cpu_internal` 这样的关键调用展示了该代码块的具体执行路径。

### Method `_CpuParamOffloader._offload_to_cpu_internal` (lines 624-651)
```python
    def _offload_to_cpu_internal(self):
        """Copy parameter data to pinned CPU storage and free GPU memory.

        This replaces param.data with CPU storage, allowing weight loading
        to continue writing to CPU memory. GPU memory is freed when the
        original GPU tensor is garbage collected.
        """
        param = self._param
        pin_memory = should_pin_memory()

        # Create pinned CPU storage and copy current GPU data
        self._cpu_storage = torch.empty_strided(
            size=param.data.size(),
            stride=param.data.stride(),
            dtype=param.data.dtype,
            layout=param.data.layout,
            device="cpu",
            pin_memory=pin_memory,
        )
    # ... omitted for brevity ...
        # and frees GPU memory when the original GPU tensor is garbage collected
        param.data = self._cpu_storage
```
**EN:** Method `_CpuParamOffloader._offload_to_cpu_internal` handles loading or retrieval of external/internal data. The docstring highlights: Copy parameter data to pinned CPU storage and free GPU memory. Key calls such as `should_pin_memory`, `torch.empty_strided`, `param.data.size`, `param.data.stride`, `self._cpu_storage.copy_` show the concrete execution path.
**CN:** Method `_CpuParamOffloader._offload_to_cpu_internal` 负责加载或获取外部/内部数据。 文档字符串强调：Copy parameter data to pinned CPU storage and free GPU memory. 像 `should_pin_memory`, `torch.empty_strided`, `param.data.size`, `param.data.stride`, `self._cpu_storage.copy_` 这样的关键调用展示了该代码块的具体执行路径。

### Method `_CpuParamOffloader._update_cpu_storage_from_param` (lines 653-685)
```python
    def _update_cpu_storage_from_param(self) -> None:
        """Update _cpu_storage from current param.data, ensuring pinned memory.

        After process_weights_after_loading, device_loading_context creates
        non-pinned CPU tensors via `p.data = p.data.to("cpu")`. Using
        non-pinned memory with `copy_(src, non_blocking=True)` causes CUDA to
        perform a stream synchronization before the copy, breaking the
        event-based fork synchronization and potentially allowing the copy
        to overwrite the GPU buffer while the compute stream still reads it.

        This method ensures _cpu_storage always uses pinned memory when
        available, re-pinning if necessary.
        """
        param = self._param

        if param.data.device.type == "cpu":
            if should_pin_memory() and not param.data.is_pinned():
                pinned = torch.empty_strided(
                    size=param.data.size(),
    # ... omitted for brevity ...
            assert self._cpu_storage is not None
            self._cpu_storage.copy_(param.data)
```
**EN:** Method `_CpuParamOffloader._update_cpu_storage_from_param` provides a reusable helper around the module's main workflow. The docstring highlights: Update _cpu_storage from current param.data, ensuring pinned memory. Key calls such as `should_pin_memory`, `param.data.is_pinned`, `torch.empty_strided`, `param.data.size`, `param.data.stride` show the concrete execution path.
**CN:** Method `_CpuParamOffloader._update_cpu_storage_from_param` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Update _cpu_storage from current param.data, ensuring pinned memory. 像 `should_pin_memory`, `param.data.is_pinned`, `torch.empty_strided`, `param.data.size`, `param.data.stride` 这样的关键调用展示了该代码块的具体执行路径。

### Method `_CpuParamOffloader.assign_static_buffer` (lines 687-716)
```python
    def assign_static_buffer(self, gpu_buffer: torch.Tensor) -> None:
        """Point parameter data to GPU static buffer.

        This is called after weight loading AND process_weights_after_loading
        complete. At this point:
        - param.data may have been replaced by device_loading_context
          (which creates new CPU tensors after quantization processing)
        - We need to update _cpu_storage to point to current param.data
          so that prefetch copies the processed weights, not stale data
        - Then point param.data to the GPU buffer for torch.compile
        """
        assert self._cpu_storage is not None, (
            "_offload_to_cpu_internal() must be called before assign_static_buffer()"
        )

        # Get current parameter (may have been replaced by
        # process_weights_after_loading)
        param = self._param

    # ... omitted for brevity ...
        # Point parameter to static GPU buffer - this is what torch.compile sees
        param.data = gpu_buffer
```
**EN:** Method `_CpuParamOffloader.assign_static_buffer` provides a reusable helper around the module's main workflow. The docstring highlights: Point parameter data to GPU static buffer. Key calls such as `self._update_cpu_storage_from_param` show the concrete execution path.
**CN:** Method `_CpuParamOffloader.assign_static_buffer` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Point parameter data to GPU static buffer. 像 `self._update_cpu_storage_from_param` 这样的关键调用展示了该代码块的具体执行路径。

### Method `_CpuParamOffloader.sync_cpu_storage` (lines 718-741)
```python
    def sync_cpu_storage(self) -> None:
        """Sync CPU storage with current param.data.

        Called after process_weights_after_loading to update _cpu_storage
        with the final processed weights. This is critical because:
        1. process_weights_after_loading may transform weights (quantization)
        2. device_loading_context creates NEW CPU tensors when moving back
        3. Our old _cpu_storage would have pre-processed or stale data

        If the parameter no longer exists on the module (e.g. transient
        KV-cache scale parameters such as k_scale/v_scale that are created
        by BaseKVCacheMethod.create_weights() and then deleted by
        process_weights_after_loading() after copying their values into
        permanent _k_scale buffers), the offloader marks itself as deleted
        and skips the sync.  The caller (_ModuleOffloader.sync_cpu_storage)
        is responsible for removing these stale entries.
        """
        try:
            self._update_cpu_storage_from_param()
    # ... omitted for brevity ...
            self._param_deleted = True
            self._cpu_storage = None
```
**EN:** Method `_CpuParamOffloader.sync_cpu_storage` provides a reusable helper around the module's main workflow. The docstring highlights: Sync CPU storage with current param.data. Key calls such as `self._update_cpu_storage_from_param` show the concrete execution path.
**CN:** Method `_CpuParamOffloader.sync_cpu_storage` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Sync CPU storage with current param.data. 像 `self._update_cpu_storage_from_param` 这样的关键调用展示了该代码块的具体执行路径。

### Method `_CpuParamOffloader.post_init` (lines 743-745)
```python
    def post_init(self):
        """No-op: offloading done in offload_to_cpu/assign_static_buffer."""
        pass
```
**EN:** Method `_CpuParamOffloader.post_init` constructs derived objects, runtime state, or helper structures. The docstring highlights: No-op: offloading done in offload_to_cpu/assign_static_buffer.
**CN:** Method `_CpuParamOffloader.post_init` 负责构造派生对象、运行时状态或辅助结构。 文档字符串强调：No-op: offloading done in offload_to_cpu/assign_static_buffer.

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
- **Benchmarking/data prep / 基准测试/数据准备**
  - **EN:** The file includes utilities for preparing benchmark inputs or measuring runtime behavior.
  - **CN:** 该文件包含用于准备基准测试输入或测量运行时行为的工具。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from abc import ABC, abstractmethod`, `from collections.abc import Generator`, `from dataclasses import dataclass`, `from typing import Any`
- **Third-party / 第三方**: `import torch`, `import torch.nn as nn`
- **vLLM internal / vLLM 内部依赖**: `import vllm.model_executor.offloader.prefetch_ops`, `from vllm.logger import init_logger`, `from vllm.model_executor.offloader.base import BaseOffloader, should_pin_memory`, `from vllm.utils.torch_utils import get_dtype_size`
