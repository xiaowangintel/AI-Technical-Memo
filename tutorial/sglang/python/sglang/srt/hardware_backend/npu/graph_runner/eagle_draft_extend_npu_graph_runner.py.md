# eagle_draft_extend_npu_graph_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/graph_runner/eagle_draft_extend_npu_graph_runner.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for backend-specific runtime support inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的后端运行时支持支持。

## Line-by-Line Analysis / 逐行分析
### Lines 14-30: Module setup and shared state / 模块设置与共享状态
```python
"""Run the model with npu graph and torch.compile."""

from __future__ import annotations

import threading
from typing import TYPE_CHECKING

import torch

from sglang.srt.configs.model_config import is_deepseek_nsa
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.speculative.eagle_draft_extend_cuda_graph_runner import (
    EAGLEDraftExtendCudaGraphRunner,
)

if TYPE_CHECKING:
    from sglang.srt.speculative.eagle_worker import EAGLEWorker
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `threading`, `typing`, `torch`, `sglang.srt.configs.model_config`, `sglang.srt.model_executor.forward_batch_info`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `threading`, `typing`, `torch`, `sglang.srt.configs.model_config`, `sglang.srt.model_executor.forward_batch_info`。

### Lines 33-33: Class `EAGLEDraftExtendNpuGraphRunner` declaration / 类 `EAGLEDraftExtendNpuGraphRunner` 声明
```python
class EAGLEDraftExtendNpuGraphRunner(EAGLEDraftExtendCudaGraphRunner):
```
**EN:** This class establishes `EAGLEDraftExtendNpuGraphRunner` as the main container/coordinator for the surrounding logic. It inherits from `EAGLEDraftExtendCudaGraphRunner`. Its core interface includes methods such as `__init__`, `_create_graph`, `_cache_loc_dtype`, `_capture_init`, `_capture_graph`, `_replay_update`.
**CN:** 该类将 `EAGLEDraftExtendNpuGraphRunner` 定义为周边逻辑的主要封装体或协调者。 它继承自 `EAGLEDraftExtendCudaGraphRunner`。 其核心接口包括 `__init__`, `_create_graph`, `_cache_loc_dtype`, `_capture_init`, `_capture_graph`, `_replay_update` 等方法。

### Lines 34-35: Method `EAGLEDraftExtendNpuGraphRunner.__init__` / 方法 `EAGLEDraftExtendNpuGraphRunner.__init__`
```python
    def __init__(self, eagle_worker: EAGLEWorker):
        super().__init__(eagle_worker)
```
**EN:** This method implements `__init__` on `EAGLEDraftExtendNpuGraphRunner`. It primarily calls `super.__init__`, `super` to complete its work.
**CN:** 该方法（属于 `EAGLEDraftExtendNpuGraphRunner`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `super` 来完成任务。

### Lines 37-38: Method `EAGLEDraftExtendNpuGraphRunner._create_graph` / 方法 `EAGLEDraftExtendNpuGraphRunner._create_graph`
```python
    def _create_graph(self):
        return torch.npu.NPUGraph()
```
**EN:** This method implements `_create_graph` on `EAGLEDraftExtendNpuGraphRunner`. It primarily calls `torch.npu.NPUGraph` to complete its work.
**CN:** 该方法（属于 `EAGLEDraftExtendNpuGraphRunner`）实现了 `_create_graph`。 它主要通过调用 `torch.npu.NPUGraph` 来完成任务。

### Lines 40-41: Method `EAGLEDraftExtendNpuGraphRunner._cache_loc_dtype` / 方法 `EAGLEDraftExtendNpuGraphRunner._cache_loc_dtype`
```python
    def _cache_loc_dtype(self):
        return torch.int32
```
**EN:** This method implements `_cache_loc_dtype` on `EAGLEDraftExtendNpuGraphRunner`.
**CN:** 该方法（属于 `EAGLEDraftExtendNpuGraphRunner`）实现了 `_cache_loc_dtype`。

### Lines 43-47: Method `EAGLEDraftExtendNpuGraphRunner._capture_init` / 方法 `EAGLEDraftExtendNpuGraphRunner._capture_init`
```python
    def _capture_init(self, run_once_fn):
        for _ in range(2):
            torch.npu.synchronize()
            self.model_runner.tp_group.barrier()
            run_once_fn()
```
**EN:** This method implements `_capture_init` on `EAGLEDraftExtendNpuGraphRunner`. It primarily calls `range`, `torch.npu.synchronize`, `self.model_runner.tp_group.barrier`, `run_once_fn` to complete its work. The implementation relies on iteration.
**CN:** 该方法（属于 `EAGLEDraftExtendNpuGraphRunner`）实现了 `_capture_init`。 它主要通过调用 `range`, `torch.npu.synchronize`, `self.model_runner.tp_group.barrier`, `run_once_fn` 来完成任务。 实现中使用了迭代逻辑。

### Lines 49-54: Method `EAGLEDraftExtendNpuGraphRunner._capture_graph` / 方法 `EAGLEDraftExtendNpuGraphRunner._capture_graph`
```python
    def _capture_graph(self, graph, pool, stream, run_once_fn):
        with torch.npu.graph(
            graph, pool=pool, stream=stream, auto_dispatch_capture=True
        ):
            out = run_once_fn()
        return out
```
**EN:** This method implements `_capture_graph` on `EAGLEDraftExtendNpuGraphRunner`. It primarily calls `torch.npu.graph`, `run_once_fn` to complete its work. State updates are written into `out`. The implementation relies on context-managed resources.
**CN:** 该方法（属于 `EAGLEDraftExtendNpuGraphRunner`）实现了 `_capture_graph`。 它主要通过调用 `torch.npu.graph`, `run_once_fn` 来完成任务。 状态更新主要写入 `out`。 实现中使用了上下文管理资源。

### Lines 56-59: Method `EAGLEDraftExtendNpuGraphRunner._replay_update` / 方法 `EAGLEDraftExtendNpuGraphRunner._replay_update`
```python
    def _replay_update(self, seq_lens):
        self.graphs[self.bs].update(
            cpu_update_input=[{"actual_seq_lengths_kv": seq_lens}]
        )
```
**EN:** This method implements `_replay_update` on `EAGLEDraftExtendNpuGraphRunner`. It primarily calls `self.graphs.update` to complete its work.
**CN:** 该方法（属于 `EAGLEDraftExtendNpuGraphRunner`）实现了 `_replay_update`。 它主要通过调用 `self.graphs.update` 来完成任务。

### Lines 61-71: Method `EAGLEDraftExtendNpuGraphRunner._replay` / 方法 `EAGLEDraftExtendNpuGraphRunner._replay`
```python
    def _replay(self, forward_batch: ForwardBatch):
        if not is_deepseek_nsa(self.model_runner.model_config.hf_config):
            seq_lens = forward_batch.seq_lens_cpu.tolist() + [0] * (
                self.bs - self.raw_bs
            )
            thread = threading.Thread(target=self._replay_update, args=(seq_lens,))
            thread.start()
            self.graphs[self.bs].replay()
            thread.join()
        else:
            self.graphs[self.bs].replay()
```
**EN:** This method implements `_replay` on `EAGLEDraftExtendNpuGraphRunner`. It primarily calls `is_deepseek_nsa`, `threading.Thread`, `thread.start`, `self.graphs.replay`, `thread.join`, `forward_batch.seq_lens_cpu.tolist` to complete its work. State updates are written into `seq_lens`, `thread`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `EAGLEDraftExtendNpuGraphRunner`）实现了 `_replay`。 它主要通过调用 `is_deepseek_nsa`, `threading.Thread`, `thread.start`, `self.graphs.replay`, `thread.join`, `forward_batch.seq_lens_cpu.tolist` 来完成任务。 状态更新主要写入 `seq_lens`, `thread`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Classes / 类**: `EAGLEDraftExtendNpuGraphRunner`
- **Functions / 函数**: `__init__`, `_create_graph`, `_cache_loc_dtype`, `_capture_init`, `_capture_graph`, `_replay_update`, `_replay`
- **Themes / 主题**: `runner`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.configs.model_config`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.speculative.eagle_draft_extend_cuda_graph_runner`, `sglang.srt.speculative.eagle_worker`
- **External / 外部依赖**: `torch`
- **Standard library / 标准库**: `__future__`, `threading`, `typing`
