# weight_updater.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/scheduler_components/weight_updater.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements weight updater logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 weight updater 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-6: Import runtime dependencies / 导入运行时依赖
```python
import logging
import traceback
from dataclasses import dataclass, field
from typing import Any, Callable, Tuple
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 8-8: Import runtime dependencies / 导入运行时依赖
```python
import torch
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 10-37: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.constants import (
    GPU_MEMORY_ALL_TYPES,
    GPU_MEMORY_TYPE_CUDA_GRAPH,
    GPU_MEMORY_TYPE_KV_CACHE,
    GPU_MEMORY_TYPE_WEIGHTS,
)
from sglang.srt.managers.io_struct import (
    CheckWeightsReqInput,
    CheckWeightsReqOutput,
    DestroyWeightsUpdateGroupReqInput,
    DestroyWeightsUpdateGroupReqOutput,
    GetWeightsByNameReqInput,
    GetWeightsByNameReqOutput,
    InitWeightsUpdateGroupReqInput,
    InitWeightsUpdateGroupReqOutput,
    ReleaseMemoryOccupationReqInput,
    ReleaseMemoryOccupationReqOutput,
    ResumeMemoryOccupationReqInput,
    ResumeMemoryOccupationReqOutput,
    UpdateWeightFromDiskReqInput,
    UpdateWeightFromDiskReqOutput,
    UpdateWeightsFromDistributedReqInput,
    UpdateWeightsFromDistributedReqOutput,
    UpdateWeightsFromIPCReqInput,
    UpdateWeightsFromIPCReqOutput,
    UpdateWeightsFromTensorReqInput,
    UpdateWeightsFromTensorReqOutput,
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 39-51: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)


@dataclass(kw_only=True, slots=True)
class SchedulerWeightUpdaterManager:
    tp_worker: Any
    draft_worker: Any
    tp_cpu_group: Any
    memory_saver_adapter: Any
    flush_cache: Callable[..., bool]
    is_fully_idle: Callable[..., bool]
    offload_tags: set = field(default_factory=set)
    stashed_model_static_state: Any = None
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 53-58: Implement flush cache after weight update / 实现flush 缓存 after weight update
```python
    def flush_cache_after_weight_update(self, recv_req) -> None:
        if recv_req.flush_cache:
            flush_cache_success = self.flush_cache(
                empty_cache=recv_req.torch_empty_cache
            )
            assert flush_cache_success, "Cache flush failed after updating weights"
```
**EN:** This block implements the method `flush_cache_after_weight_update(recv_req)` on `SchedulerWeightUpdaterManager`. It focuses on handling the weight updater responsibilities represented by `flush_cache_after_weight_update`, so the class can advance the weight updater workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerWeightUpdaterManager` 上的方法 `flush_cache_after_weight_update(recv_req)`。它围绕 `flush_cache_after_weight_update` 所承担的 weight updater 相关职责展开，使该类能够独立推进相应流程。

### Lines 60-70: Implement update weights from disk / 实现update weights from disk
```python
    def update_weights_from_disk(self, recv_req: UpdateWeightFromDiskReqInput):
        """In-place update of the weights from disk."""
        success, message = self.tp_worker.update_weights_from_disk(recv_req)
        tp_success = success
        if success and self.draft_worker is not None:
            success, message = self.draft_worker.update_weights_from_disk(recv_req)
        if tp_success:
            self.flush_cache_after_weight_update(recv_req)
        if not success:
            logger.error(message)
        return UpdateWeightFromDiskReqOutput(success, message, 0)
```
**EN:** This block implements the method `update_weights_from_disk(recv_req)` on `SchedulerWeightUpdaterManager`. It focuses on In-place update of the weights from disk., so the class can advance the weight updater workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerWeightUpdaterManager` 上的方法 `update_weights_from_disk(recv_req)`。它围绕 `update_weights_from_disk` 所承担的 weight updater 相关职责展开，使该类能够独立推进相应流程。

### Lines 72-75: Implement init weights update group / 实现init weights update group
```python
    def init_weights_update_group(self, recv_req: InitWeightsUpdateGroupReqInput):
        """Initialize the online model parameter update group."""
        success, message = self.tp_worker.init_weights_update_group(recv_req)
        return InitWeightsUpdateGroupReqOutput(success, message)
```
**EN:** This block implements the method `init_weights_update_group(recv_req)` on `SchedulerWeightUpdaterManager`. It focuses on Initialize the online model parameter update group., so the class can advance the weight updater workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerWeightUpdaterManager` 上的方法 `init_weights_update_group(recv_req)`。它围绕 `init_weights_update_group` 所承担的 weight updater 相关职责展开，使该类能够独立推进相应流程。

### Lines 77-83: Implement destroy weights update group / 实现destroy weights update group
```python
    def destroy_weights_update_group(
        self,
        recv_req: DestroyWeightsUpdateGroupReqInput,
    ):
        """Destroy the online model parameter update group."""
        success, message = self.tp_worker.destroy_weights_update_group(recv_req)
        return DestroyWeightsUpdateGroupReqOutput(success, message)
```
**EN:** This block implements the method `destroy_weights_update_group(recv_req)` on `SchedulerWeightUpdaterManager`. It focuses on Destroy the online model parameter update group., so the class can advance the weight updater workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerWeightUpdaterManager` 上的方法 `destroy_weights_update_group(recv_req)`。它围绕 `destroy_weights_update_group` 所承担的 weight updater 相关职责展开，使该类能够独立推进相应流程。

### Lines 85-95: Implement update weights from distributed / 实现update weights from distributed
```python
    def update_weights_from_distributed(
        self,
        recv_req: UpdateWeightsFromDistributedReqInput,
    ) -> Tuple[bool, str]:
        """Update the online model parameter."""
        success, message = self.tp_worker.update_weights_from_distributed(recv_req)
        if success:
            self.flush_cache_after_weight_update(recv_req)
        else:
            logger.error(message)
        return UpdateWeightsFromDistributedReqOutput(success, message)
```
**EN:** This block implements the method `update_weights_from_distributed(recv_req)` on `SchedulerWeightUpdaterManager`. It focuses on Update the online model parameter., so the class can advance the weight updater workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerWeightUpdaterManager` 上的方法 `update_weights_from_distributed(recv_req)`。它围绕 `update_weights_from_distributed` 所承担的 weight updater 相关职责展开，使该类能够独立推进相应流程。

### Lines 97-109: Implement update weights from tensor / 实现update weights from 张量
```python
    def update_weights_from_tensor(self, recv_req: UpdateWeightsFromTensorReqInput):
        """Update the online model parameter from tensors."""
        if recv_req.disable_draft_model:
            worker = self.tp_worker
        else:
            worker = self.draft_worker or self.tp_worker
        success, message = worker.update_weights_from_tensor(recv_req)
        if success:
            self.flush_cache_after_weight_update(recv_req)
        else:
            logger.error(message)
        torch.distributed.barrier(group=self.tp_cpu_group)
        return UpdateWeightsFromTensorReqOutput(success, message)
```
**EN:** This block implements the method `update_weights_from_tensor(recv_req)` on `SchedulerWeightUpdaterManager`. It focuses on Update the online model parameter from tensors., so the class can advance the weight updater workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerWeightUpdaterManager` 上的方法 `update_weights_from_tensor(recv_req)`。它围绕 `update_weights_from_tensor` 所承担的 weight updater 相关职责展开，使该类能够独立推进相应流程。

### Lines 111-122: Implement update weights from ipc / 实现update weights from ipc
```python
    def update_weights_from_ipc(self, recv_req: UpdateWeightsFromIPCReqInput):
        """Update the online model parameter from IPC for checkpoint-engine integration."""
        success, message = self.tp_worker.update_weights_from_ipc(recv_req)
        tp_success = success
        if success and self.draft_worker is not None:
            success, message = self.draft_worker.update_weights_from_ipc(recv_req)
        if tp_success:
            self.flush_cache_after_weight_update(recv_req)
        if not success:
            logger.error(message)
        torch.distributed.barrier(group=self.tp_cpu_group)
        return UpdateWeightsFromIPCReqOutput(success, message)
```
**EN:** This block implements the method `update_weights_from_ipc(recv_req)` on `SchedulerWeightUpdaterManager`. It focuses on Update the online model parameter from IPC for checkpoint-engine integration., so the class can advance the weight updater workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerWeightUpdaterManager` 上的方法 `update_weights_from_ipc(recv_req)`。它围绕 `update_weights_from_ipc` 所承担的 weight updater 相关职责展开，使该类能够独立推进相应流程。

### Lines 124-126: Implement get weights by name / 实现get weights by name
```python
    def get_weights_by_name(self, recv_req: GetWeightsByNameReqInput):
        parameter = self.tp_worker.get_weights_by_name(recv_req)
        return GetWeightsByNameReqOutput(parameter)
```
**EN:** This block implements the method `get_weights_by_name(recv_req)` on `SchedulerWeightUpdaterManager`. It focuses on handling the weight updater responsibilities represented by `get_weights_by_name`, so the class can advance the weight updater workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerWeightUpdaterManager` 上的方法 `get_weights_by_name(recv_req)`。它围绕 `get_weights_by_name` 所承担的 weight updater 相关职责展开，使该类能够独立推进相应流程。

### Lines 128-157: Implement release memory occupation / 实现release memory occupation
```python
    def release_memory_occupation(self, recv_req: ReleaseMemoryOccupationReqInput):
        assert (
            self.is_fully_idle()
        ), "release_memory_occupation should be called only when server is idle."

        tags = recv_req.tags

        if tags is None or len(tags) == 0:
            tags = GPU_MEMORY_ALL_TYPES

        for tag in tags:
            self.offload_tags.add(tag)

        if GPU_MEMORY_TYPE_KV_CACHE in tags:
            self.memory_saver_adapter.pause(GPU_MEMORY_TYPE_KV_CACHE)
            self.flush_cache()

        if GPU_MEMORY_TYPE_WEIGHTS in tags:
            self.stashed_model_static_state = _export_static_state(
                self.tp_worker.model_runner.model
            )
            torch.distributed.barrier(self.tp_cpu_group)
            self.memory_saver_adapter.pause(GPU_MEMORY_TYPE_WEIGHTS)

        if GPU_MEMORY_TYPE_CUDA_GRAPH in tags:
            self.memory_saver_adapter.pause(GPU_MEMORY_TYPE_CUDA_GRAPH)

        torch.get_device_module().synchronize()

        return ReleaseMemoryOccupationReqOutput()
```
**EN:** This block implements the method `release_memory_occupation(recv_req)` on `SchedulerWeightUpdaterManager`. It focuses on handling the weight updater responsibilities represented by `release_memory_occupation`, so the class can advance the weight updater workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerWeightUpdaterManager` 上的方法 `release_memory_occupation(recv_req)`。它围绕 `release_memory_occupation` 所承担的 weight updater 相关职责展开，使该类能够独立推进相应流程。

### Lines 159-183: Implement resume memory occupation / 实现resume memory occupation
```python
    def resume_memory_occupation(self, recv_req: ResumeMemoryOccupationReqInput):
        tags = recv_req.tags

        if tags is None or len(tags) == 0:
            tags = GPU_MEMORY_ALL_TYPES

        for tag in tags:
            self.offload_tags.remove(tag)

        if GPU_MEMORY_TYPE_CUDA_GRAPH in tags:
            self.memory_saver_adapter.resume(GPU_MEMORY_TYPE_CUDA_GRAPH)

        if GPU_MEMORY_TYPE_WEIGHTS in tags:
            self.memory_saver_adapter.resume(GPU_MEMORY_TYPE_WEIGHTS)
            torch.distributed.barrier(self.tp_cpu_group)
            _import_static_state(
                self.tp_worker.model_runner.model,
                self.stashed_model_static_state,
            )
            del self.stashed_model_static_state

        if GPU_MEMORY_TYPE_KV_CACHE in tags:
            self.memory_saver_adapter.resume(GPU_MEMORY_TYPE_KV_CACHE)

        return ResumeMemoryOccupationReqOutput()
```
**EN:** This block implements the method `resume_memory_occupation(recv_req)` on `SchedulerWeightUpdaterManager`. It focuses on handling the weight updater responsibilities represented by `resume_memory_occupation`, so the class can advance the weight updater workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerWeightUpdaterManager` 上的方法 `resume_memory_occupation(recv_req)`。它围绕 `resume_memory_occupation` 所承担的 weight updater 相关职责展开，使该类能够独立推进相应流程。

### Lines 185-194: Implement check weights / 实现检查 weights
```python
    def check_weights(self, recv_req: CheckWeightsReqInput):
        try:
            payload = self.tp_worker.model_runner.check_weights(action=recv_req.action)
            return CheckWeightsReqOutput(
                success=True, message="Success.", payload=payload
            )
        except Exception as e:
            logger.warning(f"check_weights see error: {e}")
            traceback.print_exc()
            return CheckWeightsReqOutput(success=False, message=f"{e}")
```
**EN:** This block implements the method `check_weights(recv_req)` on `SchedulerWeightUpdaterManager`. It focuses on handling the weight updater responsibilities represented by `check_weights`, so the class can advance the weight updater workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerWeightUpdaterManager` 上的方法 `check_weights(recv_req)`。它围绕 `check_weights` 所承担的 weight updater 相关职责展开，使该类能够独立推进相应流程。

### Lines 196-206: Implement save remote model / 实现save remote 模型
```python
    def save_remote_model(self, params):
        url = params["url"]

        self.tp_worker.model_runner.save_remote_model(url)

        if self.draft_worker is not None:
            draft_url = params.get("draft_url", None)
            assert (
                draft_url is not None
            ), "draft_url must be provided when draft model is enabled"
            self.draft_worker.model_runner.save_remote_model(draft_url)
```
**EN:** This block implements the method `save_remote_model(params)` on `SchedulerWeightUpdaterManager`. It focuses on handling the weight updater responsibilities represented by `save_remote_model`, so the class can advance the weight updater workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerWeightUpdaterManager` 上的方法 `save_remote_model(params)`。它围绕 `save_remote_model` 所承担的 weight updater 相关职责展开，使该类能够独立推进相应流程。

### Lines 208-213: Implement save sharded model / 实现save sharded 模型
```python
    def save_sharded_model(self, params):
        self.tp_worker.model_runner.save_sharded_model(
            path=params["path"],
            pattern=params["pattern"],
            max_size=params["max_size"],
        )
```
**EN:** This block implements the method `save_sharded_model(params)` on `SchedulerWeightUpdaterManager`. It focuses on handling the weight updater responsibilities represented by `save_sharded_model`, so the class can advance the weight updater workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerWeightUpdaterManager` 上的方法 `save_sharded_model(params)`。它围绕 `save_sharded_model` 所承担的 weight updater 相关职责展开，使该类能够独立推进相应流程。

### Lines 216-221: Implement export static state / 实现export static 状态
```python
def _export_static_state(model):
    return dict(
        buffers=[
            (name, buffer.detach().clone()) for name, buffer in model.named_buffers()
        ]
    )
```
**EN:** This block implements the function `_export_static_state(model)`. It focuses on handling the weight updater responsibilities represented by `_export_static_state`, providing reusable behavior for the weight updater pipeline.
**CN:** 该代码块实现函数 `_export_static_state(model)`。它围绕 `_export_static_state` 所承担的 weight updater 相关职责展开，为对应处理链路提供可复用能力。

### Lines 224-228: Implement import static state / 实现import static 状态
```python
def _import_static_state(model, static_params):
    with torch.inference_mode():
        self_named_buffers = dict(model.named_buffers())
        for name, tensor in static_params["buffers"]:
            self_named_buffers[name][...] = tensor
```
**EN:** This block implements the function `_import_static_state(model, static_params)`. It focuses on handling the weight updater responsibilities represented by `_import_static_state`, providing reusable behavior for the weight updater pipeline.
**CN:** 该代码块实现函数 `_import_static_state(model, static_params)`。它围绕 `_import_static_state` 所承担的 weight updater 相关职责展开，为对应处理链路提供可复用能力。

## Key Concepts / 关键概念
- **Core types / 核心类型**: SchedulerWeightUpdaterManager
- **Main callables / 主要可调用对象**: _export_static_state, _import_static_state
- **Domain focus / 领域焦点**: weight updater / weight updater
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: dataclasses, logging, traceback, typing
- **Third-party / 第三方库**: __future__, torch
- **Local Modules / 本地模块**: sglang.srt.constants, sglang.srt.managers.io_struct
