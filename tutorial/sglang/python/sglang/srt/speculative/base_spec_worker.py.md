# base_spec_worker.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/base_spec_worker.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements speculative decoding infrastructure, such as workers, metadata objects, CUDA-graph runners, or shared utilities. It coordinates draft-model execution with target-model verification to improve throughput. / 该文件实现了推测解码基础设施，例如 worker、元数据对象、CUDA Graph 运行器或共享工具函数。它通过协调草稿模型执行与目标模型校验来提升吞吐。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9: Module header, imports, and shared constants
```python
from __future__ import annotations

from abc import ABC, abstractmethod
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from sglang.srt.managers.tp_worker import TpModelWorker
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 10-10: BaseDraftWorker class declaration
```python
class BaseDraftWorker(ABC):
```
**EN:** This block declares the `BaseDraftWorker` class, which exists to produce draft tokens or draft-side state. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `BaseDraftWorker` 类，其职责是生成草稿 token 或草稿侧状态。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 11-13: BaseDraftWorker.draft method
```python
    @abstractmethod
    def draft():
        pass
```
**EN:** This block uses `BaseDraftWorker.draft` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BaseDraftWorker.draft` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 15-17: BaseDraftWorker.draft_extend method
```python
    @abstractmethod
    def draft_extend():
        pass
```
**EN:** This block uses `BaseDraftWorker.draft_extend` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BaseDraftWorker.draft_extend` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 20-20: BaseSpecWorker class declaration
```python
class BaseSpecWorker(ABC):
```
**EN:** This block declares the `BaseSpecWorker` class, which exists to coordinate staged runtime execution. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `BaseSpecWorker` 类，其职责是协调分阶段运行时执行。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 21-24: BaseSpecWorker.target_worker method
```python
    @property
    @abstractmethod
    def target_worker(self) -> TpModelWorker:
        pass
```
**EN:** This block uses `BaseSpecWorker.target_worker` to coordinate staged runtime execution. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BaseSpecWorker.target_worker` 来协调分阶段运行时执行。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 26-29: BaseSpecWorker.draft_worker method
```python
    @property
    @abstractmethod
    def draft_worker(self) -> BaseDraftWorker:
        pass
```
**EN:** This block uses `BaseSpecWorker.draft_worker` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BaseSpecWorker.draft_worker` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 31-34: BaseSpecWorker.clear_cache_pool method
```python
    @abstractmethod
    def clear_cache_pool(self):
        # TODO: move this abstract method to BaseTpWorker and call through self.model_runner
        pass
```
**EN:** This block uses `BaseSpecWorker.clear_cache_pool` to clear cached state or buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BaseSpecWorker.clear_cache_pool` 来清理缓存状态或缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 36-42: BaseSpecWorker.on_verify_complete_cpu method
```python
    def on_verify_complete_cpu(self, num_correct_drafts_per_req: list[int]) -> None:
        """Hook called after verify finishes and accept counts are on CPU.

        Default no-op. Adaptive-aware workers override this to feed the
        controller without forcing a GPU→CPU sync in the worker hot path.
        """
        pass
```
**EN:** This block uses `BaseSpecWorker.on_verify_complete_cpu` to verify draft results against the target model. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BaseSpecWorker.on_verify_complete_cpu` 来根据目标模型校验草稿结果。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- Worker coordination / Worker 协调
- Speculative decoding / 推测解码
- Integration with model workers / 与模型 worker 集成

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.managers.tp_worker`
### External / 外部
- `__future__`
- `abc` (stdlib)
- `typing` (stdlib)
