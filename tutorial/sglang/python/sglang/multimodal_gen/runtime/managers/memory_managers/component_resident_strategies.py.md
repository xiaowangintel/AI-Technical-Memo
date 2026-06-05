# component_resident_strategies.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/managers/memory_managers/component_resident_strategies.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for component resident strategies in the multimodal generation stack. Key symbols include `_module_to_local_device`, `_module_reference_tensor`, `_module_ready_on_local_device`. / 该模块包含多模态生成体系中与 component resident strategies 相关的运行时支持代码。 关键符号包括 `_module_to_local_device`, `_module_reference_tensor`, `_module_ready_on_local_device`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: Imports and module setup / 导入与模块初始化
```python
"""
Basic Component Resident Strategy Utilities for defining usage of components, to let ComponentResidencyManager to coordinate
"""

from __future__ import annotations

from typing import TYPE_CHECKING

import torch
import torch.nn as nn

from sglang.multimodal_gen.runtime.distributed import get_local_torch_device
from sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload import (
    LayerwiseOffloadableModuleMixin,
# ...
        ResidencyState,
    )

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 28-39: Function `_module_to_local_device` / 函数 `_module_to_local_device`
```python
def _module_to_local_device(
    module: nn.Module, *, dtype: torch.dtype | None = None
) -> None:
    device = get_local_torch_device()
    tensor = _module_reference_tensor(module)
    if tensor is not None and tensor.device == device:
        if dtype is None or tensor.dtype == dtype:
            return
    if dtype is None:
        module.to(device, non_blocking=True)
    else:
        module.to(device, dtype=dtype, non_blocking=True)
```
**EN:** This function drives `_module_to_local_device` with inputs such as `module`, `dtype`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_module_to_local_device`，主要处理 `module`, `dtype` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 42-46: Function `_module_reference_tensor` / 函数 `_module_reference_tensor`
```python
def _module_reference_tensor(module: nn.Module) -> torch.Tensor | None:
    tensor = next(module.parameters(), None)
    if tensor is None:
        tensor = next(module.buffers(), None)
    return tensor
```
**EN:** This function drives `_module_reference_tensor` with inputs such as `module`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_module_reference_tensor`，主要处理 `module` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 49-57: Function `_module_ready_on_local_device` / 函数 `_module_ready_on_local_device`
```python
def _module_ready_on_local_device(
    module: nn.Module, *, dtype: torch.dtype | None = None
) -> bool:
    tensor = _module_reference_tensor(module)
    if tensor is None:
        return True
    if tensor.device != get_local_torch_device():
        return False
    return dtype is None or tensor.dtype == dtype
```
**EN:** This function drives `_module_ready_on_local_device` with inputs such as `module`, `dtype`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_module_ready_on_local_device`，主要处理 `module`, `dtype` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 60-61: Function `is_fsdp_managed_module` / 函数 `is_fsdp_managed_module`
```python
def is_fsdp_managed_module(module: nn.Module) -> bool:
    return module.__class__.__name__.startswith("FSDP")
```
**EN:** This function drives `is_fsdp_managed_module` with inputs such as `module`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `is_fsdp_managed_module`，主要处理 `module` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 64-138: Class `ComponentResidencyStrategy` / 类 `ComponentResidencyStrategy`
```python
class ComponentResidencyStrategy:
    """Baseclass for describing how a component should be treated (regarding where its weights locates)

    e.g., a LayerwiseOffloadStrategy would override:
        enter: to prefetch some layers before DiT is used, and
        exits: to release GPU weight snapshot after DiT is used
    to achieve desired behavior

    """

    name = "resident"

    def prepare_for_use(
        self,
# ...
        pass

    def exit(self, module: nn.Module, next_module: nn.Module | None = None) -> None:
        pass
```
**EN:** This class models `ComponentResidencyStrategy`. Baseclass for describing how a component should be treated (regarding where its weights locates) Important methods include `prepare_for_use`, `wait_for_use`, `finish_use`, `prepare_after_request`.
**CN:** 该类实现 `ComponentResidencyStrategy`。 文档字符串指出：Baseclass for describing how a component should be treated (regarding where its weights locates) 其中较重要的方法包括 `prepare_for_use`, `wait_for_use`, `finish_use`, `prepare_after_request`。

### Lines 141-152: Class `ResidentStrategy` / 类 `ResidentStrategy`
```python
class ResidentStrategy(ComponentResidencyStrategy):
    name = "resident"

    def prepare_for_use(
        self,
        module: nn.Module,
        use: ComponentUse,
        state: ResidencyState,
    ) -> None:
        if is_fsdp_managed_module(module):
            return
        _module_to_local_device(module, dtype=use.target_dtype)
```
**EN:** This class models `ResidentStrategy` as a specialization of `ComponentResidencyStrategy`. Important methods include `prepare_for_use`.
**CN:** 该类实现 `ResidentStrategy`，并继承/扩展 `ComponentResidencyStrategy`。 其中较重要的方法包括 `prepare_for_use`。

### Lines 155-314: Class `SnapshotModuleResidency` / 类 `SnapshotModuleResidency`
```python
class SnapshotModuleResidency:
    """Reusable snapshot-based module residency primitive.

    This helper only knows how to:
    - keep CPU parameter/buffer snapshots,
    - prefetch a module (H2D) to the local device on a CUDA side stream
    - release a module by rebinding tensors to those snapshots,
    - track and wait for readiness events.

    It deliberately does not know about pipeline stages, phases, or model-specific
    ordering. Strategy subclasses decide when each primitive is called.
    """

    def __init__(self, *, pin_cpu_memory: bool, enable_async_prefetch: bool) -> None:
# ...
            module.to(get_local_torch_device(), non_blocking=True)
            event = torch.get_device_module().Event()
            event.record(prefetch_stream)
        self._ready_events[component_name] = event
```
**EN:** This class models `SnapshotModuleResidency`. Reusable snapshot-based module residency primitive. Important methods include `__init__`, `is_on_gpu`, `is_ready`, `wait_ready`.
**CN:** 该类实现 `SnapshotModuleResidency`。 文档字符串指出：Reusable snapshot-based module residency primitive. 其中较重要的方法包括 `__init__`, `is_on_gpu`, `is_ready`, `wait_ready`。

### Lines 317-390: Class `SnapshotStrategy` / 类 `SnapshotStrategy`
```python
class SnapshotStrategy(ComponentResidencyStrategy):
    """Snapshot residency: async H2D before use and light snapshot release after use."""

    name = "snapshot"

    def __init__(
        self,
        *,
        pin_cpu_memory: bool,
        enable_async_prefetch: bool,
        copy_runtime_buffers_on_release: bool = False,
    ) -> None:
        self._snapshot_residency = SnapshotModuleResidency(
            pin_cpu_memory=pin_cpu_memory,
# ...
        use: ComponentUse,
        state: ResidencyState,
    ) -> None:
        self.prepare_for_use(module, use, state)
```
**EN:** This class models `SnapshotStrategy` as a specialization of `ComponentResidencyStrategy`. Snapshot residency: async H2D before use and light snapshot release after use. Important methods include `__init__`, `capture`, `is_ready`, `record_ready`.
**CN:** 该类实现 `SnapshotStrategy`，并继承/扩展 `ComponentResidencyStrategy`。 文档字符串指出：Snapshot residency: async H2D before use and light snapshot release after use. 其中较重要的方法包括 `__init__`, `capture`, `is_ready`, `record_ready`。

### Lines 393-484: Class `VanillaD2HStrategy` / 类 `VanillaD2HStrategy`
```python
class VanillaD2HStrategy(ComponentResidencyStrategy):
    """A strategy that performs native torch D2H and H2D for a component"""

    name = "vanilla"

    def __init__(self) -> None:
        self._prefetch_stream: object | None = None
        self._ready_events: dict[str, object] = {}

    def prepare_for_use(
        self,
        module: nn.Module,
        use: ComponentUse,
        state: ResidencyState,
# ...
            self.wait_for_use(module, use, state)
            return
        if not preferred:
            self.finish_use(module, use, state)
```
**EN:** This class models `VanillaD2HStrategy` as a specialization of `ComponentResidencyStrategy`. A strategy that performs native torch D2H and H2D for a component Important methods include `__init__`, `prepare_for_use`, `wait_for_use`, `prefetch_for_use`.
**CN:** 该类实现 `VanillaD2HStrategy`，并继承/扩展 `ComponentResidencyStrategy`。 文档字符串指出：A strategy that performs native torch D2H and H2D for a component 其中较重要的方法包括 `__init__`, `prepare_for_use`, `wait_for_use`, `prefetch_for_use`。

### Lines 487-508: Class `LayerwiseOffloadStrategy` / 类 `LayerwiseOffloadStrategy`
```python
class LayerwiseOffloadStrategy(ComponentResidencyStrategy):
    """A wrapper around LayerwiseOffloadManager to fit in a ComponentResidencyStrategy"""

    name = "layerwise"

    def enter(self, module: nn.Module) -> None:
        if isinstance(module, LayerwiseOffloadableModuleMixin):
            module.prepare_for_next_req()

    def exit(self, module: nn.Module, next_module: nn.Module | None = None) -> None:
        if not isinstance(module, LayerwiseOffloadableModuleMixin):
            return
        for manager in module.layerwise_offload_managers:
            manager.release_all()
# ...
        use: ComponentUse,
        state: ResidencyState,
    ) -> None:
        self.prepare_for_use(module, use, state)
```
**EN:** This class models `LayerwiseOffloadStrategy` as a specialization of `ComponentResidencyStrategy`. A wrapper around LayerwiseOffloadManager to fit in a ComponentResidencyStrategy Important methods include `enter`, `exit`, `prepare_after_request`.
**CN:** 该类实现 `LayerwiseOffloadStrategy`，并继承/扩展 `ComponentResidencyStrategy`。 文档字符串指出：A wrapper around LayerwiseOffloadManager to fit in a ComponentResidencyStrategy 其中较重要的方法包括 `enter`, `exit`, `prepare_after_request`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Artifact storage management / 产物存储管理
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.managers.memory_managers.component_manager`
- **External / 外部**: `__future__`, `torch`, `torch.nn`
- **Stdlib / 标准库**: `typing`
