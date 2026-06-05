# eagle_draft_npu_graph_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/graph_runner/eagle_draft_npu_graph_runner.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for backend-specific runtime support inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的后端运行时支持支持。

## Line-by-Line Analysis / 逐行分析
### Lines 14-44: Module setup and shared state / 模块设置与共享状态
```python
"""Run the model with npu graph and torch.compile"""

from __future__ import annotations

import logging
import threading
from typing import TYPE_CHECKING, Dict, Union

import numpy as np
import torch

from sglang.srt.configs.model_config import AttentionArch, is_deepseek_nsa
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.speculative.eagle_draft_cuda_graph_runner import (
    EAGLEDraftCudaGraphRunner,
)

if TYPE_CHECKING:
    from sglang.srt.speculative.eagle_worker import EAGLEWorker

from sglang.srt.utils import is_npu

logger = logging.getLogger(__name__)

if is_npu():
    torch.cuda.CUDAGraph = torch.npu.NPUGraph
    torch.cuda.synchronize = torch.npu.synchronize
    torch.cuda.graph = torch.npu.graph
    torch.cuda.stream = torch.npu.stream
    torch.cuda.Stream = torch.npu.Stream
    torch.cuda.current_stream = torch.npu.current_stream
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `logging`, `threading`, `typing`, `numpy`, `torch`. It also defines symbols such as `logger`, `torch.cuda.CUDAGraph`, `torch.cuda.synchronize`, `torch.cuda.graph`, `torch.cuda.stream`, `torch.cuda.Stream` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `logging`, `threading`, `typing`, `numpy`, `torch`。 同时定义了 `logger`, `torch.cuda.CUDAGraph`, `torch.cuda.synchronize`, `torch.cuda.graph`, `torch.cuda.stream`, `torch.cuda.Stream` 等符号，供后续逻辑使用。

### Lines 47-47: Class `EAGLEDraftNpuGraphRunner` declaration / 类 `EAGLEDraftNpuGraphRunner` 声明
```python
class EAGLEDraftNpuGraphRunner(EAGLEDraftCudaGraphRunner):
```
**EN:** This class establishes `EAGLEDraftNpuGraphRunner` as the main container/coordinator for the surrounding logic. It inherits from `EAGLEDraftCudaGraphRunner`. Its core interface includes methods such as `__init__`, `_init_arch_map`, `_create_graph`, `_capture_init`, `_capture_graph`, `_get_update_attr_name`.
**CN:** 该类将 `EAGLEDraftNpuGraphRunner` 定义为周边逻辑的主要封装体或协调者。 它继承自 `EAGLEDraftCudaGraphRunner`。 其核心接口包括 `__init__`, `_init_arch_map`, `_create_graph`, `_capture_init`, `_capture_graph`, `_get_update_attr_name` 等方法。

### Lines 48-52: Method `EAGLEDraftNpuGraphRunner.__init__` / 方法 `EAGLEDraftNpuGraphRunner.__init__`
```python
    def __init__(self, eagle_worker: EAGLEWorker):
        super().__init__(eagle_worker)
        self.update_attr_name = None
        self.update_attr_type = None
        self._init_arch_map()
```
**EN:** This method implements `__init__` on `EAGLEDraftNpuGraphRunner`. It primarily calls `super.__init__`, `self._init_arch_map`, `super` to complete its work. State updates are written into `self.update_attr_name`, `self.update_attr_type`.
**CN:** 该方法（属于 `EAGLEDraftNpuGraphRunner`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `self._init_arch_map`, `super` 来完成任务。 状态更新主要写入 `self.update_attr_name`, `self.update_attr_type`。

### Lines 54-62: Method `EAGLEDraftNpuGraphRunner._init_arch_map` / 方法 `EAGLEDraftNpuGraphRunner._init_arch_map`
```python
    def _init_arch_map(self):
        self.attr_name: Dict[str, str] = {
            AttentionArch.MLA: "actual_seq_lengths_kv",
            AttentionArch.MHA: "context_lens",
        }
        self.attr_type: Dict[str, Union[list, torch.Tensor]] = {
            AttentionArch.MLA: [],
            AttentionArch.MHA: torch.Tensor(),
        }
```
**EN:** This method implements `_init_arch_map` on `EAGLEDraftNpuGraphRunner`. It primarily calls `torch.Tensor` to complete its work. State updates are written into `self.attr_name`, `self.attr_type`.
**CN:** 该方法（属于 `EAGLEDraftNpuGraphRunner`）实现了 `_init_arch_map`。 它主要通过调用 `torch.Tensor` 来完成任务。 状态更新主要写入 `self.attr_name`, `self.attr_type`。

### Lines 64-65: Method `EAGLEDraftNpuGraphRunner._create_graph` / 方法 `EAGLEDraftNpuGraphRunner._create_graph`
```python
    def _create_graph(self):
        return torch.npu.NPUGraph()
```
**EN:** This method implements `_create_graph` on `EAGLEDraftNpuGraphRunner`. It primarily calls `torch.npu.NPUGraph` to complete its work.
**CN:** 该方法（属于 `EAGLEDraftNpuGraphRunner`）实现了 `_create_graph`。 它主要通过调用 `torch.npu.NPUGraph` 来完成任务。

### Lines 67-71: Method `EAGLEDraftNpuGraphRunner._capture_init` / 方法 `EAGLEDraftNpuGraphRunner._capture_init`
```python
    def _capture_init(self, run_once_fn):
        for _ in range(2):
            torch.npu.synchronize()
            self.model_runner.tp_group.barrier()
            run_once_fn()
```
**EN:** This method implements `_capture_init` on `EAGLEDraftNpuGraphRunner`. It primarily calls `range`, `torch.npu.synchronize`, `self.model_runner.tp_group.barrier`, `run_once_fn` to complete its work. The implementation relies on iteration.
**CN:** 该方法（属于 `EAGLEDraftNpuGraphRunner`）实现了 `_capture_init`。 它主要通过调用 `range`, `torch.npu.synchronize`, `self.model_runner.tp_group.barrier`, `run_once_fn` 来完成任务。 实现中使用了迭代逻辑。

### Lines 73-78: Method `EAGLEDraftNpuGraphRunner._capture_graph` / 方法 `EAGLEDraftNpuGraphRunner._capture_graph`
```python
    def _capture_graph(self, graph, pool, stream, run_once_fn):
        with torch.npu.graph(
            graph, pool=pool, stream=stream, auto_dispatch_capture=True
        ):
            out = run_once_fn()
        return out
```
**EN:** This method implements `_capture_graph` on `EAGLEDraftNpuGraphRunner`. It primarily calls `torch.npu.graph`, `run_once_fn` to complete its work. State updates are written into `out`. The implementation relies on context-managed resources.
**CN:** 该方法（属于 `EAGLEDraftNpuGraphRunner`）实现了 `_capture_graph`。 它主要通过调用 `torch.npu.graph`, `run_once_fn` 来完成任务。 状态更新主要写入 `out`。 实现中使用了上下文管理资源。

### Lines 80-81: Method `EAGLEDraftNpuGraphRunner._get_update_attr_name` / 方法 `EAGLEDraftNpuGraphRunner._get_update_attr_name`
```python
    def _get_update_attr_name(self):
        return self.attr_name[AttentionArch.MLA]
```
**EN:** This method implements `_get_update_attr_name` on `EAGLEDraftNpuGraphRunner`.
**CN:** 该方法（属于 `EAGLEDraftNpuGraphRunner`）实现了 `_get_update_attr_name`。

### Lines 83-84: Method `EAGLEDraftNpuGraphRunner._get_update_attr_type` / 方法 `EAGLEDraftNpuGraphRunner._get_update_attr_type`
```python
    def _get_update_attr_type(self):
        return self.attr_type[AttentionArch.MLA]
```
**EN:** This method implements `_get_update_attr_type` on `EAGLEDraftNpuGraphRunner`.
**CN:** 该方法（属于 `EAGLEDraftNpuGraphRunner`）实现了 `_get_update_attr_type`。

### Lines 86-94: Method `EAGLEDraftNpuGraphRunner._replay_update` / 方法 `EAGLEDraftNpuGraphRunner._replay_update`
```python
    def _replay_update(self, seq_lens_list):
        if isinstance(self.update_attr_type, torch.Tensor):
            seq_lens = torch.from_numpy(np.array(seq_lens_list).astype(np.int32))

        self.graphs[self.bs].update(
            cpu_update_input=[
                {self.update_attr_name: seq_lens} for seq_lens in seq_lens_list
            ]
        )
```
**EN:** This method implements `_replay_update` on `EAGLEDraftNpuGraphRunner`. It primarily calls `isinstance`, `self.graphs.update`, `torch.from_numpy`, `np.array.astype`, `np.array` to complete its work. State updates are written into `seq_lens`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `EAGLEDraftNpuGraphRunner`）实现了 `_replay_update`。 它主要通过调用 `isinstance`, `self.graphs.update`, `torch.from_numpy`, `np.array.astype`, `np.array` 来完成任务。 状态更新主要写入 `seq_lens`。 实现中使用了条件分支。

### Lines 96-112: Method `EAGLEDraftNpuGraphRunner._replay` / 方法 `EAGLEDraftNpuGraphRunner._replay`
```python
    def _replay(self, forward_batch: ForwardBatch):
        self.update_attr_name = self._get_update_attr_name()
        self.update_attr_type = self._get_update_attr_type()
        if not is_deepseek_nsa(self.model_runner.model_config.hf_config):
            seq_lens_for_each_draft_step = []
            for speculative_step_id in range(self.speculative_num_steps - 1):
                seq_lens_cpu = forward_batch.seq_lens_cpu + speculative_step_id + 1
                seq_lens = seq_lens_cpu.tolist() + [0] * (self.bs - self.raw_bs)
                seq_lens_for_each_draft_step.append(seq_lens)
            thread = threading.Thread(
                target=self._replay_update, args=(seq_lens_for_each_draft_step,)
            )
            thread.start()
            self.graphs[self.bs].replay()
            thread.join()
        else:
            self.graphs[self.bs].replay()
```
**EN:** This method implements `_replay` on `EAGLEDraftNpuGraphRunner`. It primarily calls `self._get_update_attr_name`, `self._get_update_attr_type`, `is_deepseek_nsa`, `range`, `threading.Thread`, `thread.start` to complete its work. State updates are written into `self.update_attr_name`, `self.update_attr_type`, `seq_lens_for_each_draft_step`, `thread`, `seq_lens_cpu`, `seq_lens`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `EAGLEDraftNpuGraphRunner`）实现了 `_replay`。 它主要通过调用 `self._get_update_attr_name`, `self._get_update_attr_type`, `is_deepseek_nsa`, `range`, `threading.Thread`, `thread.start` 来完成任务。 状态更新主要写入 `self.update_attr_name`, `self.update_attr_type`, `seq_lens_for_each_draft_step`, `thread`, `seq_lens_cpu`, `seq_lens`。 实现中使用了条件分支、迭代逻辑。

### Lines 114-115: Method `EAGLEDraftNpuGraphRunner._cache_loc_dtype` / 方法 `EAGLEDraftNpuGraphRunner._cache_loc_dtype`
```python
    def _cache_loc_dtype(self):
        return torch.int32
```
**EN:** This method implements `_cache_loc_dtype` on `EAGLEDraftNpuGraphRunner`.
**CN:** 该方法（属于 `EAGLEDraftNpuGraphRunner`）实现了 `_cache_loc_dtype`。

## Key Concepts / 关键概念
- **Classes / 类**: `EAGLEDraftNpuGraphRunner`
- **Functions / 函数**: `__init__`, `_init_arch_map`, `_create_graph`, `_capture_init`, `_capture_graph`, `_get_update_attr_name`, `_get_update_attr_type`, `_replay_update`
- **Themes / 主题**: `runner`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.configs.model_config`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.speculative.eagle_draft_cuda_graph_runner`, `sglang.srt.utils`, `sglang.srt.speculative.eagle_worker`
- **External / 外部依赖**: `numpy`, `torch`
- **Standard library / 标准库**: `__future__`, `logging`, `threading`, `typing`
