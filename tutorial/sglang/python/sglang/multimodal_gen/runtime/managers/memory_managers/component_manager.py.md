# component_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/managers/memory_managers/component_manager.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for component manager in the multimodal generation stack. Key symbols include `ComponentUse`, `ResidencyState`, `ResidencyBatch`. / 该模块包含多模态生成体系中与 component manager 相关的运行时支持代码。 关键符号包括 `ComponentUse`, `ResidencyState`, `ResidencyBatch`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: Imports and module setup / 导入与模块初始化
```python
from collections.abc import Callable, Iterator
from contextlib import contextmanager
from dataclasses import dataclass
from functools import lru_cache
from typing import Mapping, MutableMapping, Protocol, Sequence, TypeVar

import torch
import torch.nn as nn

from sglang.multimodal_gen.runtime.managers.memory_managers.component_resident_strategies import (
    ComponentResidencyStrategy,
    LayerwiseOffloadStrategy,
    ResidentStrategy,
    VanillaD2HStrategy,
# ...

logger = init_logger(__name__)

_T = TypeVar("_T")
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 34-60: Class `ComponentUse` / 类 `ComponentUse`
```python
@dataclass(slots=True)
class ComponentUse:
    """Describes one stage/use-site access to a pipeline component."""

    stage_name: str
    # Pipeline module key: transformer / video_dit / text_encoder / ...
    component_name: str
    # Model-specific phase for sequential components, e.g. stage1 or stage2.
    # TODO: Replace this with ordered timeline identity. In an all-sequential
    # pipeline, use-site identity should come from the declared ComponentUse
    # order instead of a per-use `phase` field.
    phase: str | None = None
    # Whether the manager may prepare this component for the next request.
    preferred_ready_after_request: bool = False
# ...
    target_dtype: torch.dtype | None = None
    # Some components are intentionally kept ready between warmup and the first
    # real request to avoid measuring a cold H2D in the user-visible request.
    keep_ready_after_warmup: bool = False
```
**EN:** This class models `ComponentUse`. Describes one stage/use-site access to a pipeline component.
**CN:** 该类实现 `ComponentUse`。 文档字符串指出：Describes one stage/use-site access to a pipeline component.

### Lines 63-78: Class `ResidencyState` / 类 `ResidencyState`
```python
@dataclass(slots=True)
class ResidencyState:
    """
    Necessary internal runtime info of ComponentResidencyManager
    """

    stages: Sequence["ComponentResidencyStage"] = ()
    stage_index: int = -1
    stage_name: str | None = None
    next_stage_name: str | None = None
    current_use: ComponentUse | None = None
    # the ComponentUses of the preceding stages
    future_uses: tuple[ComponentUse, ...] = ()
    batch_is_warmup: bool = False
    manager_mode: str = "static"
    trace_enabled: bool = False
```
**EN:** This class models `ResidencyState`. Necessary internal runtime info of ComponentResidencyManager
**CN:** 该类实现 `ResidencyState`。 文档字符串指出：Necessary internal runtime info of ComponentResidencyManager

### Lines 81-82: Class `ResidencyBatch` / 类 `ResidencyBatch`
```python
class ResidencyBatch(Protocol):
    is_warmup: bool
```
**EN:** This class models `ResidencyBatch` as a specialization of `Protocol`.
**CN:** 该类实现 `ResidencyBatch`，并继承/扩展 `Protocol`。

### Lines 85-88: Class `ComponentResidencyStage` / 类 `ComponentResidencyStage`
```python
class ComponentResidencyStage(Protocol):
    def component_uses(
        self, server_args: ServerArgs, stage_name: str | None = None
    ) -> list[ComponentUse]: ...
```
**EN:** This class models `ComponentResidencyStage` as a specialization of `Protocol`. Important methods include `component_uses`.
**CN:** 该类实现 `ComponentResidencyStage`，并继承/扩展 `Protocol`。 其中较重要的方法包括 `component_uses`。

### Lines 91-94: Class `ComponentResidencyPipeline` / 类 `ComponentResidencyPipeline`
```python
class ComponentResidencyPipeline(Protocol):
    modules: Mapping[str, object]
    _stage_name_mapping: Mapping[str, ComponentResidencyStage]
    component_residency_strategies: MutableMapping[str, "ComponentResidencyStrategy"]
```
**EN:** This class models `ComponentResidencyPipeline` as a specialization of `Protocol`.
**CN:** 该类实现 `ComponentResidencyPipeline`，并继承/扩展 `Protocol`。

### Lines 97-110: Function `should_cpu_offload_component` / 函数 `should_cpu_offload_component`
```python
def should_cpu_offload_component(
    component_name: str, module: nn.Module, server_args: ServerArgs
) -> bool:
    if server_args.use_fsdp_inference or is_fsdp_managed_module(module):
        return False
    if is_dit_component_name(component_name):
        return bool(server_args.dit_cpu_offload)
    if is_text_encoder_component_name(component_name):
        return bool(server_args.text_encoder_cpu_offload)
    if is_image_encoder_component_name(component_name):
        return bool(server_args.image_encoder_cpu_offload)
    if is_vae_component_name(component_name):
        return bool(server_args.vae_cpu_offload)
    return False
```
**EN:** This function drives `should_cpu_offload_component` with inputs such as `component_name`, `module`, `server_args`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `should_cpu_offload_component`，主要处理 `component_name`, `module`, `server_args` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 113-122: Function `build_component_residency_strategy` / 函数 `build_component_residency_strategy`
```python
def build_component_residency_strategy(
    component_name: str,
    module: nn.Module,
    server_args: ServerArgs,
) -> ComponentResidencyStrategy:
    if is_layerwise_offloaded_module(module):
        return LayerwiseOffloadStrategy()
    if should_cpu_offload_component(component_name, module, server_args):
        return VanillaD2HStrategy()
    return ResidentStrategy()
```
**EN:** This function drives `build_component_residency_strategy` with inputs such as `component_name`, `module`, `server_args`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `build_component_residency_strategy`，主要处理 `component_name`, `module`, `server_args` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 125-651: Class `ComponentResidencyManager` / 类 `ComponentResidencyManager`
```python
class ComponentResidencyManager:
    """Executor-owned component lifecycle coordinator. Provide hooks for a PipelineExecutor

    Hooks are called around executor progress:
        before request: collect a flat ordered ComponentUse timeline.
        before stage: update current/next stage context only.
        begin use: finish previous active use, prepare current use, wait until ready.
        end use: finish or keep current use, then prefetch the next heavy timeline use.
        finish request: finish active use and schedule preferred next-request prefetch.

    The manager instance is global and rebound to the active pipeline before request execution.
    This manager is designed only for sequential execution order for now
    """

# ...
        if not torch.get_device_module().is_available():
            return
        torch.get_device_module().empty_cache()
        self._trace("empty_cache", use, strategy, module, detail="after_release")
```
**EN:** This class models `ComponentResidencyManager`. Executor-owned component lifecycle coordinator. Important methods include `__init__`, `enabled`, `refresh_pipeline`, `refresh_server_args`.
**CN:** 该类实现 `ComponentResidencyManager`。 文档字符串指出：Executor-owned component lifecycle coordinator. 其中较重要的方法包括 `__init__`, `enabled`, `refresh_pipeline`, `refresh_server_args`。

### Lines 652-654: Top-level configuration / 顶层配置
```python


_GLOBAL_COMPONENT_RESIDENCY_MANAGER: ComponentResidencyManager | None = None
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 657-671: Function `get_global_component_residency_manager` / 函数 `get_global_component_residency_manager`
```python
def get_global_component_residency_manager(
    pipeline: ComponentResidencyPipeline,
    server_args: ServerArgs,
) -> ComponentResidencyManager:
    global _GLOBAL_COMPONENT_RESIDENCY_MANAGER

    if _GLOBAL_COMPONENT_RESIDENCY_MANAGER is None:
        _GLOBAL_COMPONENT_RESIDENCY_MANAGER = ComponentResidencyManager(
            pipeline, server_args
        )
    else:
        _GLOBAL_COMPONENT_RESIDENCY_MANAGER.refresh_server_args(server_args)
    _GLOBAL_COMPONENT_RESIDENCY_MANAGER.refresh_pipeline(pipeline)

    return _GLOBAL_COMPONENT_RESIDENCY_MANAGER
```
**EN:** This function drives `get_global_component_residency_manager` with inputs such as `pipeline`, `server_args`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_global_component_residency_manager`，主要处理 `pipeline`, `server_args` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Artifact storage management / 产物存储管理
- Caching strategy / 缓存策略
- Image generation flow / 图像生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.managers.memory_managers.component_resident_strategies`, `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload`, `sglang.multimodal_gen.runtime.managers.memory_managers.layerwise_offload_components`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `torch`, `torch.nn`
- **Stdlib / 标准库**: `collections.abc`, `contextlib`, `dataclasses`, `functools`, `typing`
