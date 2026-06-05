# expert_distribution.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/eplb/expert_distribution.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the expert-parallel load balancing part of the SRT runtime and implements logic centered on `expert_distribution`. It exposes primary entry points such as `ExpertDistributionMetrics`, `ExpertDistributionRecorder`, `_ExpertDistributionRecorderNoop`. / 该模块属于 SRT 运行时的专家并行负载均衡部分，主要实现围绕 `expert_distribution` 的逻辑。 它对外提供的主要入口包括 `ExpertDistributionMetrics`, `ExpertDistributionRecorder`, `_ExpertDistributionRecorderNoop`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-46: Module imports, constants, and setup
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================

from __future__ import annotations

import logging
import math
import time
from abc import ABC
from collections import deque
from contextlib import contextmanager
from dataclasses import dataclass
from pathlib import Path
from typing import TYPE_CHECKING, Any, Dict, List, Literal, Optional, Tuple, Type

import einops
import torch
import torch.distributed

from sglang.srt.environ import envs
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.observability.metrics_collector import ExpertDispatchCollector
from sglang.srt.server_args import ServerArgs
from sglang.srt.utils import Withable, get_device, get_int_env_var

if TYPE_CHECKING:
    from sglang.srt.eplb.expert_location import ExpertLocationMetadata

logger = logging.getLogger(__name__)

# --------------------------------------- Entrypoint -----------------------------------------

_OutputMode = Literal["file", "object"]


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；协调分布式通信。

### Lines 47-50: Class ExpertDistributionMetrics
```python
@dataclass
class ExpertDistributionMetrics:
    eplb_balancedness: torch.Tensor

```
**EN:** This range introduces `ExpertDistributionMetrics` and defines the structure or metadata that its methods rely on. In this range it records metrics or tracing signals.
**CN:** 这一段引入 `ExpertDistributionMetrics`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 51-52: Method ExpertDistributionMetrics.copy_to_cpu
```python
    def copy_to_cpu(self):
        self.eplb_balancedness = self.eplb_balancedness.to("cpu", non_blocking=True)
```
**EN:** This callable implements `ExpertDistributionMetrics.copy_to_cpu` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `ExpertDistributionMetrics.copy_to_cpu`，主要用于将数据转换为另一种表示。

### Lines 55-57: Class ExpertDistributionRecorder
```python
class ExpertDistributionRecorder(ABC):
    """Global expert distribution recording"""

```
**EN:** This range introduces `ExpertDistributionRecorder` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "Global expert distribution recording"
**CN:** 这一段引入 `ExpertDistributionRecorder`，并定义其后续方法依赖的结构或元数据。

### Lines 58-74: Method ExpertDistributionRecorder.init_new
```python
    @staticmethod
    def init_new(
        server_args: ServerArgs,
        expert_location_metadata: ExpertLocationMetadata,
        rank: int,
    ):
        if server_args.expert_distribution_recorder_mode is not None:
            assert (
                expert_location_metadata is not None
            ), "ExpertLocationMetadata is required for expert distribution recording. One possible"
            "reason is that you are using a model that does not support expert distribution"
            "recording. Try setting `get_model_config_for_expert_location` in your model."
            return _ExpertDistributionRecorderReal(
                server_args, expert_location_metadata, rank
            )
        else:
            return _ExpertDistributionRecorderNoop()
```
**EN:** This callable implements `ExpertDistributionRecorder.init_new`. It takes `server_args`, `expert_location_metadata`, `rank` and mainly implements init new. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ExpertDistributionRecorder.init_new`。它接收 `server_args`, `expert_location_metadata`, `rank`，主要用于实现 init new 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 76-78: Method ExpertDistributionRecorder.with_current_layer
```python
    @contextmanager
    def with_current_layer(self, layer_idx):
        yield
```
**EN:** This callable implements `ExpertDistributionRecorder.with_current_layer`. It takes `layer_idx` and mainly implements with current layer.
**CN:** 这一可调用对象实现了 `ExpertDistributionRecorder.with_current_layer`。它接收 `layer_idx`，主要用于实现 with current layer 相关逻辑。

### Lines 80-82: Method ExpertDistributionRecorder.with_debug_name
```python
    @contextmanager
    def with_debug_name(self, debug_name):
        yield
```
**EN:** This callable implements `ExpertDistributionRecorder.with_debug_name`. It takes `debug_name` and mainly implements with debug name.
**CN:** 这一可调用对象实现了 `ExpertDistributionRecorder.with_debug_name`。它接收 `debug_name`，主要用于实现 with debug name 相关逻辑。

### Lines 84-86: Method ExpertDistributionRecorder.disable_this_region
```python
    @contextmanager
    def disable_this_region(self):
        yield
```
**EN:** This callable implements `ExpertDistributionRecorder.disable_this_region` and mainly implements disable this region.
**CN:** 这一可调用对象实现了 `ExpertDistributionRecorder.disable_this_region`，主要用于实现 disable this region 相关逻辑。

### Lines 88-90: Method ExpertDistributionRecorder.with_forward_pass
```python
    @contextmanager
    def with_forward_pass(self, forward_pass_id: int, forward_batch: ForwardBatch):
        yield {}
```
**EN:** This callable implements `ExpertDistributionRecorder.with_forward_pass`. It takes `forward_pass_id`, `forward_batch` and mainly implements with forward pass.
**CN:** 这一可调用对象实现了 `ExpertDistributionRecorder.with_forward_pass`。它接收 `forward_pass_id`, `forward_batch`，主要用于实现 with forward pass 相关逻辑。

### Lines 92-93: Method ExpertDistributionRecorder.on_select_experts
```python
    def on_select_experts(self, topk_ids: torch.Tensor):
        pass
```
**EN:** This callable implements `ExpertDistributionRecorder.on_select_experts`. It takes `topk_ids` and mainly implements on select experts.
**CN:** 这一可调用对象实现了 `ExpertDistributionRecorder.on_select_experts`。它接收 `topk_ids`，主要用于实现 on select experts 相关逻辑。

### Lines 95-102: Method ExpertDistributionRecorder.on_deepep_dispatch_normal
```python
    def on_deepep_dispatch_normal(
        self,
        local_physical_count_of_layer: List[int],
        num_tokens_per_rank,
        num_tokens_per_rdma_rank,
        num_tokens_per_expert,
    ):
        pass
```
**EN:** This callable implements `ExpertDistributionRecorder.on_deepep_dispatch_normal`. It takes `local_physical_count_of_layer`, `num_tokens_per_rank`, `num_tokens_per_rdma_rank`, `num_tokens_per_expert` and mainly implements on deepep dispatch normal.
**CN:** 这一可调用对象实现了 `ExpertDistributionRecorder.on_deepep_dispatch_normal`。它接收 `local_physical_count_of_layer`, `num_tokens_per_rank`, `num_tokens_per_rdma_rank`, `num_tokens_per_expert`，主要用于实现 on deepep dispatch normal 相关逻辑。

### Lines 104-107: Method ExpertDistributionRecorder.on_deepep_dispatch_low_latency
```python
    def on_deepep_dispatch_low_latency(
        self, local_physical_count_of_layer: torch.Tensor
    ):
        pass
```
**EN:** This callable implements `ExpertDistributionRecorder.on_deepep_dispatch_low_latency`. It takes `local_physical_count_of_layer` and mainly implements on deepep dispatch low latency.
**CN:** 这一可调用对象实现了 `ExpertDistributionRecorder.on_deepep_dispatch_low_latency`。它接收 `local_physical_count_of_layer`，主要用于实现 on deepep dispatch low latency 相关逻辑。

### Lines 109-110: Method ExpertDistributionRecorder.start_record
```python
    def start_record(self):
        self._on_not_implemented()
```
**EN:** This callable implements `ExpertDistributionRecorder.start_record` and mainly implements start record.
**CN:** 这一可调用对象实现了 `ExpertDistributionRecorder.start_record`，主要用于实现 start record 相关逻辑。

### Lines 112-113: Method ExpertDistributionRecorder.stop_record
```python
    def stop_record(self):
        self._on_not_implemented()
```
**EN:** This callable implements `ExpertDistributionRecorder.stop_record` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `ExpertDistributionRecorder.stop_record`，主要用于将数据转换为另一种表示。

### Lines 115-116: Method ExpertDistributionRecorder.dump_record
```python
    def dump_record(self, output_mode: _OutputMode = "file"):
        self._on_not_implemented()
```
**EN:** This callable implements `ExpertDistributionRecorder.dump_record`. It takes `output_mode` and mainly implements dump record.
**CN:** 这一可调用对象实现了 `ExpertDistributionRecorder.dump_record`。它接收 `output_mode`，主要用于实现 dump record 相关逻辑。

### Lines 118-120: Method ExpertDistributionRecorder.recording
```python
    @property
    def recording(self):
        return False
```
**EN:** This callable implements `ExpertDistributionRecorder.recording` and mainly implements recording.
**CN:** 这一可调用对象实现了 `ExpertDistributionRecorder.recording`，主要用于实现 recording 相关逻辑。

### Lines 122-125: Method ExpertDistributionRecorder._on_not_implemented
```python
    def _on_not_implemented(self):
        raise Exception(
            "Please set ServerArgs.expert_distribution_recorder_mode to use ExpertDistributionRecorder."
        )
```
**EN:** This callable implements `ExpertDistributionRecorder._on_not_implemented` and mainly implements on not implemented. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ExpertDistributionRecorder._on_not_implemented`，主要用于实现 on not implemented 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 128-129: Class _ExpertDistributionRecorderNoop
```python
class _ExpertDistributionRecorderNoop(ExpertDistributionRecorder):
    pass
```
**EN:** This range introduces `_ExpertDistributionRecorderNoop` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_ExpertDistributionRecorderNoop`，并定义其后续方法依赖的结构或元数据。

### Lines 132-132: Class _ExpertDistributionRecorderReal
```python
class _ExpertDistributionRecorderReal(ExpertDistributionRecorder):
```
**EN:** This range introduces `_ExpertDistributionRecorderReal` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_ExpertDistributionRecorderReal`，并定义其后续方法依赖的结构或元数据。

### Lines 133-159: Method _ExpertDistributionRecorderReal.__init__
```python
    def __init__(
        self,
        server_args: ServerArgs,
        expert_location_metadata: ExpertLocationMetadata,
        rank: int,
    ):
        self._server_args = server_args
        self._expert_location_metadata = expert_location_metadata

        self._recording = False
        self._disable_all = False
        self._current_forward_pass_id = Withable()
        self._current_layer_idx = Withable()
        self._current_debug_name = Withable()
        self._accumulator = _Accumulator.init_new(
            server_args, expert_location_metadata, rank
        )
        self._single_pass_gatherers = {
            k: _SinglePassGatherer.init_new(server_args, expert_location_metadata, rank)
            for k in self._accumulator.get_single_pass_gatherer_keys()
        }

        if server_args.enable_expert_distribution_metrics:
            logger.info(
                "ExpertDistributionRecorder auto start record since enable_expert_distribution_metrics"
            )
            self.start_record()
```
**EN:** This callable implements `_ExpertDistributionRecorderReal.__init__`. It takes `server_args`, `expert_location_metadata`, `rank` and mainly initializes instance state and defaults. In this range it emits logs for diagnostics; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `_ExpertDistributionRecorderReal.__init__`。它接收 `server_args`, `expert_location_metadata`, `rank`，主要用于初始化实例状态与默认值。 在这一范围内，它会输出日志以便诊断；记录指标或追踪信号。

### Lines 161-162: Method _ExpertDistributionRecorderReal.with_current_layer
```python
    def with_current_layer(self, layer_idx):
        return self._current_layer_idx.with_value(layer_idx)
```
**EN:** This callable implements `_ExpertDistributionRecorderReal.with_current_layer`. It takes `layer_idx` and mainly implements with current layer.
**CN:** 这一可调用对象实现了 `_ExpertDistributionRecorderReal.with_current_layer`。它接收 `layer_idx`，主要用于实现 with current layer 相关逻辑。

### Lines 164-165: Method _ExpertDistributionRecorderReal.with_debug_name
```python
    def with_debug_name(self, debug_name):
        return self._current_debug_name.with_value(debug_name)
```
**EN:** This callable implements `_ExpertDistributionRecorderReal.with_debug_name`. It takes `debug_name` and mainly implements with debug name.
**CN:** 这一可调用对象实现了 `_ExpertDistributionRecorderReal.with_debug_name`。它接收 `debug_name`，主要用于实现 with debug name 相关逻辑。

### Lines 167-175: Method _ExpertDistributionRecorderReal.with_forward_pass
```python
    @contextmanager
    def with_forward_pass(self, forward_pass_id: int, forward_batch: ForwardBatch):
        outputs = {}
        with self._current_forward_pass_id.with_value(forward_pass_id):
            self._on_forward_pass_start(forward_batch)
            try:
                yield outputs
            finally:
                self._on_forward_pass_end(forward_pass_id, outputs)
```
**EN:** This callable implements `_ExpertDistributionRecorderReal.with_forward_pass`. It takes `forward_pass_id`, `forward_batch` and mainly implements with forward pass.
**CN:** 这一可调用对象实现了 `_ExpertDistributionRecorderReal.with_forward_pass`。它接收 `forward_pass_id`, `forward_batch`，主要用于实现 with forward pass 相关逻辑。

### Lines 177-185: Method _ExpertDistributionRecorderReal.disable_this_region
```python
    @contextmanager
    def disable_this_region(self):
        """Context manager to temporarily disable recording."""
        previous_disable_all = self._disable_all
        self._disable_all = True
        try:
            yield
        finally:
            self._disable_all = previous_disable_all
```
**EN:** This callable implements `_ExpertDistributionRecorderReal.disable_this_region` and mainly implements disable this region. The docstring states: "Context manager to temporarily disable recording."
**CN:** 这一可调用对象实现了 `_ExpertDistributionRecorderReal.disable_this_region`，主要用于实现 disable this region 相关逻辑。

### Lines 187-192: Method _ExpertDistributionRecorderReal._on_forward_pass_start
```python
    def _on_forward_pass_start(self, forward_batch: ForwardBatch):
        if not self._recording:
            return
        for gatherer_key, gatherer in self._single_pass_gatherers.items():
            gatherer.reset()
            gatherer.on_forward_pass_start(forward_batch)
```
**EN:** This callable implements `_ExpertDistributionRecorderReal._on_forward_pass_start`. It takes `forward_batch` and mainly implements on forward pass start.
**CN:** 这一可调用对象实现了 `_ExpertDistributionRecorderReal._on_forward_pass_start`。它接收 `forward_batch`，主要用于实现 on forward pass start 相关逻辑。

### Lines 194-201: Method _ExpertDistributionRecorderReal._on_forward_pass_end
```python
    def _on_forward_pass_end(self, forward_pass_id: int, outputs: Dict[str, Any]):
        if not self._recording:
            return
        for gatherer_key, gatherer in self._single_pass_gatherers.items():
            single_pass_data = gatherer.collect()
            self._accumulator.append(
                forward_pass_id, gatherer_key, single_pass_data, outputs
            )
```
**EN:** This callable implements `_ExpertDistributionRecorderReal._on_forward_pass_end`. It takes `forward_pass_id`, `outputs` and mainly implements on forward pass end.
**CN:** 这一可调用对象实现了 `_ExpertDistributionRecorderReal._on_forward_pass_end`。它接收 `forward_pass_id`, `outputs`，主要用于实现 on forward pass end 相关逻辑。

### Lines 203-204: Method _ExpertDistributionRecorderReal.on_select_experts
```python
    def on_select_experts(self, topk_ids: torch.Tensor):
        self._on_hook("on_select_experts", topk_ids=topk_ids)
```
**EN:** This callable implements `_ExpertDistributionRecorderReal.on_select_experts`. It takes `topk_ids` and mainly implements on select experts.
**CN:** 这一可调用对象实现了 `_ExpertDistributionRecorderReal.on_select_experts`。它接收 `topk_ids`，主要用于实现 on select experts 相关逻辑。

### Lines 206-219: Method _ExpertDistributionRecorderReal.on_deepep_dispatch_normal
```python
    def on_deepep_dispatch_normal(
        self,
        local_physical_count_of_layer: List[int],
        num_tokens_per_rank,
        num_tokens_per_rdma_rank,
        num_tokens_per_expert,
    ):
        self._on_hook(
            "on_deepep_dispatch_normal",
            local_physical_count_of_layer=local_physical_count_of_layer,
            num_tokens_per_rank=num_tokens_per_rank,
            num_tokens_per_rdma_rank=num_tokens_per_rdma_rank,
            num_tokens_per_expert=num_tokens_per_expert,
        )
```
**EN:** This callable implements `_ExpertDistributionRecorderReal.on_deepep_dispatch_normal`. It takes `local_physical_count_of_layer`, `num_tokens_per_rank`, `num_tokens_per_rdma_rank`, `num_tokens_per_expert` and mainly implements on deepep dispatch normal.
**CN:** 这一可调用对象实现了 `_ExpertDistributionRecorderReal.on_deepep_dispatch_normal`。它接收 `local_physical_count_of_layer`, `num_tokens_per_rank`, `num_tokens_per_rdma_rank`, `num_tokens_per_expert`，主要用于实现 on deepep dispatch normal 相关逻辑。

### Lines 221-227: Method _ExpertDistributionRecorderReal.on_deepep_dispatch_low_latency
```python
    def on_deepep_dispatch_low_latency(
        self, local_physical_count_of_layer: torch.Tensor
    ):
        self._on_hook(
            "on_deepep_dispatch_low_latency",
            local_physical_count_of_layer=local_physical_count_of_layer,
        )
```
**EN:** This callable implements `_ExpertDistributionRecorderReal.on_deepep_dispatch_low_latency`. It takes `local_physical_count_of_layer` and mainly implements on deepep dispatch low latency.
**CN:** 这一可调用对象实现了 `_ExpertDistributionRecorderReal.on_deepep_dispatch_low_latency`。它接收 `local_physical_count_of_layer`，主要用于实现 on deepep dispatch low latency 相关逻辑。

### Lines 229-241: Method _ExpertDistributionRecorderReal._on_hook
```python
    def _on_hook(self, hook_name: str, **kwargs):
        if self._disable_all:
            return
        if not (
            self._recording or torch.get_device_module().is_current_stream_capturing()
        ):
            return
        gatherer = self._single_pass_gatherers[
            self._accumulator.get_single_pass_gatherer_key(
                self._current_debug_name.value
            )
        ]
        getattr(gatherer, hook_name)(layer_idx=self._current_layer_idx.value, **kwargs)
```
**EN:** This callable implements `_ExpertDistributionRecorderReal._on_hook`. It takes `hook_name`, `**kwargs` and mainly implements on hook.
**CN:** 这一可调用对象实现了 `_ExpertDistributionRecorderReal._on_hook`。它接收 `hook_name`, `**kwargs`，主要用于实现 on hook 相关逻辑。

### Lines 243-251: Method _ExpertDistributionRecorderReal._reset
```python
    def _reset(self):
        """Reset the expert distribution recorder."""
        logger.info("Resetting ExpertDistributionRecorder...")
        assert (
            self._current_layer_idx.value is None
        ), f"{self._current_layer_idx.value=}"
        for gatherer in self._single_pass_gatherers.values():
            gatherer.reset()
        self._accumulator.reset()
```
**EN:** This callable implements `_ExpertDistributionRecorderReal._reset` and mainly applies configuration to mutable state. The docstring states: "Reset the expert distribution recorder." In this range it performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `_ExpertDistributionRecorderReal._reset`，主要用于将配置写入可变状态。 在这一范围内，它会对非法状态执行防御性检查；输出日志以便诊断。

### Lines 253-260: Method _ExpertDistributionRecorderReal.start_record
```python
    def start_record(self):
        """Start recording the expert distribution."""
        if self._recording:
            logger.warning(
                "SGLang server is already recording expert ids. Did you forget to dump the expert ids recorded so far by sending requests to the `/stop_expert_distribution_record` and `/dump_expert_distribution_record` endpoints?"
            )
        self._reset()
        self._recording = True
```
**EN:** This callable implements `_ExpertDistributionRecorderReal.start_record` and mainly implements start record. The docstring states: "Start recording the expert distribution." In this range it emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `_ExpertDistributionRecorderReal.start_record`，主要用于实现 start record 相关逻辑。 在这一范围内，它会输出日志以便诊断。

### Lines 262-268: Method _ExpertDistributionRecorderReal.stop_record
```python
    def stop_record(self):
        """Stop recording the expert distribution."""
        if not self._recording:
            logger.warning(
                "SGLang server has not been recording expert ids. Did you forget to start recording by sending request to the `/start_expert_distribution_record` endpoint?"
            )
        self._recording = False
```
**EN:** This callable implements `_ExpertDistributionRecorderReal.stop_record` and mainly converts data into another representation. The docstring states: "Stop recording the expert distribution." In this range it emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `_ExpertDistributionRecorderReal.stop_record`，主要用于将数据转换为另一种表示。 在这一范围内，它会输出日志以便诊断。

### Lines 270-274: Method _ExpertDistributionRecorderReal.dump_record
```python
    def dump_record(self, output_mode: _OutputMode = "file"):
        """Dump the expert distribution record and reset the recorder after dumping."""
        output = self._accumulator.dump(output_mode=output_mode)
        self._reset()
        return output
```
**EN:** This callable implements `_ExpertDistributionRecorderReal.dump_record`. It takes `output_mode` and mainly implements dump record. The docstring states: "Dump the expert distribution record and reset the recorder after dumping."
**CN:** 这一可调用对象实现了 `_ExpertDistributionRecorderReal.dump_record`。它接收 `output_mode`，主要用于实现 dump record 相关逻辑。

### Lines 276-278: Method _ExpertDistributionRecorderReal.recording
```python
    @property
    def recording(self):
        return self._recording
```
**EN:** This callable implements `_ExpertDistributionRecorderReal.recording` and mainly implements recording.
**CN:** 这一可调用对象实现了 `_ExpertDistributionRecorderReal.recording`，主要用于实现 recording 相关逻辑。

### Lines 279-285: Module-level constants and helpers
```python


_global_expert_distribution_recorder: Optional[ExpertDistributionRecorder] = (
    _ExpertDistributionRecorderNoop()
)


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 286-287: Function get_global_expert_distribution_recorder
```python
def get_global_expert_distribution_recorder():
    return _global_expert_distribution_recorder
```
**EN:** This callable implements `get_global_expert_distribution_recorder` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `get_global_expert_distribution_recorder`，主要用于获取某个值或派生视图。

### Lines 290-292: Function set_global_expert_distribution_recorder
```python
def set_global_expert_distribution_recorder(value):
    global _global_expert_distribution_recorder
    _global_expert_distribution_recorder = value
```
**EN:** This callable implements `set_global_expert_distribution_recorder`. It takes `value` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `set_global_expert_distribution_recorder`。它接收 `value`，主要用于将配置写入可变状态。

### Lines 293-297: Module-level logic
```python


# --------------------------------------- SinglePassGatherer -----------------------------------------


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 298-298: Class _SinglePassGatherer
```python
class _SinglePassGatherer(ABC):
```
**EN:** This range introduces `_SinglePassGatherer` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_SinglePassGatherer`，并定义其后续方法依赖的结构或元数据。

### Lines 299-328: Method _SinglePassGatherer.init_new
```python
    @staticmethod
    def init_new(
        server_args: ServerArgs,
        expert_location_metadata: ExpertLocationMetadata,
        rank: int,
    ) -> "_SinglePassGatherer":
        if server_args.expert_distribution_recorder_mode == "per_token":
            return _DetailSinglePassGatherer(
                server_args, expert_location_metadata, rank
            )

        if server_args.expert_distribution_recorder_mode == "stat_approx":
            if server_args.moe_a2a_backend != "none" and (
                server_args.deepep_mode == "normal"
            ):
                return _DeepepNormalSinglePassGatherer(expert_location_metadata, rank)
            else:
                raise NotImplementedError

        if server_args.moe_a2a_backend != "none":
            if server_args.deepep_mode == "normal":
                return _SelectExpertsSinglePassGatherer(expert_location_metadata, rank)
            elif server_args.deepep_mode == "low_latency":
                return _DeepepLowLatencySinglePassGatherer(
                    expert_location_metadata, rank
                )
            else:
                raise NotImplementedError

        return _SelectExpertsSinglePassGatherer(expert_location_metadata, rank)
```
**EN:** This callable implements `_SinglePassGatherer.init_new`. It takes `server_args`, `expert_location_metadata`, `rank` and mainly implements init new. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `_SinglePassGatherer.init_new`。它接收 `server_args`, `expert_location_metadata`, `rank`，主要用于实现 init new 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 330-332: Method _SinglePassGatherer.__init__
```python
    def __init__(self, expert_location_metadata: ExpertLocationMetadata, rank: int):
        self._expert_location_metadata = expert_location_metadata
        self._rank = rank
```
**EN:** This callable implements `_SinglePassGatherer.__init__`. It takes `expert_location_metadata`, `rank` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `_SinglePassGatherer.__init__`。它接收 `expert_location_metadata`, `rank`，主要用于初始化实例状态与默认值。

### Lines 334-335: Method _SinglePassGatherer.on_forward_pass_start
```python
    def on_forward_pass_start(self, forward_batch: ForwardBatch):
        pass
```
**EN:** This callable implements `_SinglePassGatherer.on_forward_pass_start`. It takes `forward_batch` and mainly implements on forward pass start.
**CN:** 这一可调用对象实现了 `_SinglePassGatherer.on_forward_pass_start`。它接收 `forward_batch`，主要用于实现 on forward pass start 相关逻辑。

### Lines 337-338: Method _SinglePassGatherer.on_select_experts
```python
    def on_select_experts(self, layer_idx: int, topk_ids: torch.Tensor):
        pass
```
**EN:** This callable implements `_SinglePassGatherer.on_select_experts`. It takes `layer_idx`, `topk_ids` and mainly implements on select experts.
**CN:** 这一可调用对象实现了 `_SinglePassGatherer.on_select_experts`。它接收 `layer_idx`, `topk_ids`，主要用于实现 on select experts 相关逻辑。

### Lines 340-348: Method _SinglePassGatherer.on_deepep_dispatch_normal
```python
    def on_deepep_dispatch_normal(
        self,
        layer_idx: int,
        local_physical_count_of_layer: List[int],
        num_tokens_per_rank,
        num_tokens_per_rdma_rank,
        num_tokens_per_expert,
    ):
        pass
```
**EN:** This callable implements `_SinglePassGatherer.on_deepep_dispatch_normal`. It takes `layer_idx`, `local_physical_count_of_layer`, `num_tokens_per_rank`, `num_tokens_per_rdma_rank` and mainly implements on deepep dispatch normal.
**CN:** 这一可调用对象实现了 `_SinglePassGatherer.on_deepep_dispatch_normal`。它接收 `layer_idx`, `local_physical_count_of_layer`, `num_tokens_per_rank`, `num_tokens_per_rdma_rank`，主要用于实现 on deepep dispatch normal 相关逻辑。

### Lines 350-353: Method _SinglePassGatherer.on_deepep_dispatch_low_latency
```python
    def on_deepep_dispatch_low_latency(
        self, layer_idx: int, local_physical_count_of_layer: torch.Tensor
    ):
        pass
```
**EN:** This callable implements `_SinglePassGatherer.on_deepep_dispatch_low_latency`. It takes `layer_idx`, `local_physical_count_of_layer` and mainly implements on deepep dispatch low latency.
**CN:** 这一可调用对象实现了 `_SinglePassGatherer.on_deepep_dispatch_low_latency`。它接收 `layer_idx`, `local_physical_count_of_layer`，主要用于实现 on deepep dispatch low latency 相关逻辑。

### Lines 355-356: Method _SinglePassGatherer.reset
```python
    def reset(self):
        raise NotImplementedError
```
**EN:** This callable implements `_SinglePassGatherer.reset` and mainly applies configuration to mutable state. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `_SinglePassGatherer.reset`，主要用于将配置写入可变状态。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 358-359: Method _SinglePassGatherer.collect
```python
    def collect(self) -> Dict:
        raise NotImplementedError
```
**EN:** This callable implements `_SinglePassGatherer.collect` and mainly collects runtime information. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `_SinglePassGatherer.collect`，主要用于收集运行时信息。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 362-365: Class _DetailSinglePassGatherer
```python
class _DetailSinglePassGatherer(_SinglePassGatherer):
    # DeepSeek V3 has this value; should generalize later
    _TOP_K_NUM = 8

```
**EN:** This range introduces `_DetailSinglePassGatherer` and defines the structure or metadata that its methods rely on. In this range it adjusts sampling behavior and decoding controls.
**CN:** 这一段引入 `_DetailSinglePassGatherer`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会调整采样行为与解码控制。

### Lines 366-387: Method _DetailSinglePassGatherer.__init__
```python
    def __init__(
        self,
        server_args: ServerArgs,
        expert_location_metadata: ExpertLocationMetadata,
        rank: int,
    ):
        super().__init__(expert_location_metadata, rank)
        self._metadata: Optional[Dict[str, Any]] = None
        self._topk_ids_of_layer = torch.zeros(
            (
                expert_location_metadata.num_layers,
                # TODO determine the max number
                server_args.chunked_prefill_size * 8,
                self._TOP_K_NUM,
            ),
            dtype=torch.int32,
            device=server_args.device,
        )
        self._misc_objects: List[Dict[str, Any]] = []
        assert (
            not server_args.enable_two_batch_overlap
        ), "DetailSinglePassGatherer does not support TBO yet"
```
**EN:** This callable implements `_DetailSinglePassGatherer.__init__`. It takes `server_args`, `expert_location_metadata`, `rank` and mainly initializes instance state and defaults. In this range it performs defensive checks on invalid state; adjusts sampling behavior and decoding controls.
**CN:** 这一可调用对象实现了 `_DetailSinglePassGatherer.__init__`。它接收 `server_args`, `expert_location_metadata`, `rank`，主要用于初始化实例状态与默认值。 在这一范围内，它会对非法状态执行防御性检查；调整采样行为与解码控制。

### Lines 388-389: Class-level scaffolding for _DetailSinglePassGatherer
```python
        # TODO assert shared experts fusion is disabled, o/w data is wrong

```
**EN:** This callable implements `None._DetailSinglePassGatherer` and mainly implements Detail Single Pass Gatherer. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `None._DetailSinglePassGatherer`，主要用于实现 Detail Single Pass Gatherer 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 390-399: Method _DetailSinglePassGatherer.on_forward_pass_start
```python
    def on_forward_pass_start(self, forward_batch: ForwardBatch):
        assert self._metadata is None
        self._metadata = dict(
            # TODO pr-chain
            # rids=forward_batch.rids,
            input_ids=forward_batch.input_ids.cpu().tolist(),
            positions=forward_batch.positions.cpu().tolist(),
            extend_seq_lens=forward_batch.extend_seq_lens_cpu,
            forward_mode=forward_batch.forward_mode.value,
        )
```
**EN:** This callable implements `_DetailSinglePassGatherer.on_forward_pass_start`. It takes `forward_batch` and mainly implements on forward pass start. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `_DetailSinglePassGatherer.on_forward_pass_start`。它接收 `forward_batch`，主要用于实现 on forward pass start 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 401-404: Method _DetailSinglePassGatherer.on_select_experts
```python
    def on_select_experts(self, layer_idx: int, topk_ids: torch.Tensor):
        self._topk_ids_of_layer[layer_idx, : topk_ids.shape[0], : topk_ids.shape[1]] = (
            topk_ids
        )
```
**EN:** This callable implements `_DetailSinglePassGatherer.on_select_experts`. It takes `layer_idx`, `topk_ids` and mainly implements on select experts.
**CN:** 这一可调用对象实现了 `_DetailSinglePassGatherer.on_select_experts`。它接收 `layer_idx`, `topk_ids`，主要用于实现 on select experts 相关逻辑。

### Lines 406-421: Method _DetailSinglePassGatherer.on_deepep_dispatch_normal
```python
    def on_deepep_dispatch_normal(
        self,
        layer_idx: int,
        local_physical_count_of_layer: List[int],
        num_tokens_per_rank,
        num_tokens_per_rdma_rank,
        num_tokens_per_expert,
    ):
        self._misc_objects.append(
            dict(
                layer_id=layer_idx,
                num_tokens_per_rank=num_tokens_per_rank.cpu().tolist(),
                num_tokens_per_rdma_rank=num_tokens_per_rdma_rank.cpu().tolist(),
                num_tokens_per_expert=num_tokens_per_expert.cpu().tolist(),
            )
        )
```
**EN:** This callable implements `_DetailSinglePassGatherer.on_deepep_dispatch_normal`. It takes `layer_idx`, `local_physical_count_of_layer`, `num_tokens_per_rank`, `num_tokens_per_rdma_rank` and mainly implements on deepep dispatch normal.
**CN:** 这一可调用对象实现了 `_DetailSinglePassGatherer.on_deepep_dispatch_normal`。它接收 `layer_idx`, `local_physical_count_of_layer`, `num_tokens_per_rank`, `num_tokens_per_rdma_rank`，主要用于实现 on deepep dispatch normal 相关逻辑。

### Lines 423-426: Method _DetailSinglePassGatherer.reset
```python
    def reset(self):
        self._topk_ids_of_layer[...] = -1
        self._misc_objects.clear()
        self._metadata = None
```
**EN:** This callable implements `_DetailSinglePassGatherer.reset` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `_DetailSinglePassGatherer.reset`，主要用于将配置写入可变状态。

### Lines 428-443: Method _DetailSinglePassGatherer.collect
```python
    def collect(self) -> Dict:
        num_tokens = len(self._metadata["input_ids"])

        global_physical_count = _convert_per_token_to_global_physical_count(
            num_tokens,
            num_layers=self._expert_location_metadata.num_layers,
            num_physical_experts=self._expert_location_metadata.num_physical_experts,
            _topk_ids_of_layer=self._topk_ids_of_layer,
        )

        return dict(
            **self._metadata,
            topk_ids_of_layer=self._topk_ids_of_layer[:, :num_tokens, :].clone().cpu(),
            misc_objects=self._misc_objects,
            global_physical_count=global_physical_count,
        )
```
**EN:** This callable implements `_DetailSinglePassGatherer.collect` and mainly collects runtime information.
**CN:** 这一可调用对象实现了 `_DetailSinglePassGatherer.collect`，主要用于收集运行时信息。

### Lines 446-446: Class _LayerBasedCpuSinglePassGatherer
```python
class _LayerBasedCpuSinglePassGatherer(_SinglePassGatherer):
```
**EN:** This range introduces `_LayerBasedCpuSinglePassGatherer` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_LayerBasedCpuSinglePassGatherer`，并定义其后续方法依赖的结构或元数据。

### Lines 447-449: Method _LayerBasedCpuSinglePassGatherer.__init__
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self._objects_of_layer = {}
```
**EN:** This callable implements `_LayerBasedCpuSinglePassGatherer.__init__`. It takes `*args`, `**kwargs` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `_LayerBasedCpuSinglePassGatherer.__init__`。它接收 `*args`, `**kwargs`，主要用于初始化实例状态与默认值。

### Lines 451-458: Method _LayerBasedCpuSinglePassGatherer._on_layer_data
```python
    def _on_layer_data(self, layer_idx: int, objects: List[int]):
        assert 0 <= layer_idx < self._expert_location_metadata.num_layers
        if layer_idx in self._objects_of_layer:
            self._objects_of_layer[layer_idx] = _list_sum(
                self._objects_of_layer[layer_idx], objects
            )
        else:
            self._objects_of_layer[layer_idx] = objects
```
**EN:** This callable implements `_LayerBasedCpuSinglePassGatherer._on_layer_data`. It takes `layer_idx`, `objects` and mainly implements on layer data. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `_LayerBasedCpuSinglePassGatherer._on_layer_data`。它接收 `layer_idx`, `objects`，主要用于实现 on layer data 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 460-461: Method _LayerBasedCpuSinglePassGatherer.reset
```python
    def reset(self):
        self._objects_of_layer.clear()
```
**EN:** This callable implements `_LayerBasedCpuSinglePassGatherer.reset` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `_LayerBasedCpuSinglePassGatherer.reset`，主要用于将配置写入可变状态。

### Lines 463-468: Method _LayerBasedCpuSinglePassGatherer._collect_objects
```python
    def _collect_objects(self, pad_len: int) -> torch.Tensor:
        data = [
            self._objects_of_layer.get(layer_index) or ([0] * pad_len)
            for layer_index in range(self._expert_location_metadata.num_layers)
        ]
        return torch.tensor(data)
```
**EN:** This callable implements `_LayerBasedCpuSinglePassGatherer._collect_objects`. It takes `pad_len` and mainly collects runtime information.
**CN:** 这一可调用对象实现了 `_LayerBasedCpuSinglePassGatherer._collect_objects`。它接收 `pad_len`，主要用于收集运行时信息。

### Lines 471-472: Function _list_sum
```python
def _list_sum(a: List, b: List) -> List:
    return [x + y for x, y in zip(a, b, strict=True)]
```
**EN:** This callable implements `_list_sum`. It takes `a`, `b` and mainly implements list sum.
**CN:** 这一可调用对象实现了 `_list_sum`。它接收 `a`, `b`，主要用于实现 list sum 相关逻辑。

### Lines 475-475: Class _LayerBasedGpuSinglePassGatherer
```python
class _LayerBasedGpuSinglePassGatherer(_SinglePassGatherer):
```
**EN:** This range introduces `_LayerBasedGpuSinglePassGatherer` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_LayerBasedGpuSinglePassGatherer`，并定义其后续方法依赖的结构或元数据。

### Lines 476-493: Method _LayerBasedGpuSinglePassGatherer.__init__
```python
    def __init__(self, *args, enable_global_physical_experts: bool, **kwargs):
        super().__init__(*args, **kwargs)

        device = get_device()

        self._enable_global_physical_experts = enable_global_physical_experts
        self._data = torch.zeros(
            (
                self._expert_location_metadata.num_layers,
                (
                    self._expert_location_metadata.num_physical_experts
                    if enable_global_physical_experts
                    else self._expert_location_metadata.num_local_physical_experts
                ),
            ),
            dtype=torch.int,
            device=device,
        )
```
**EN:** This callable implements `_LayerBasedGpuSinglePassGatherer.__init__`. It takes `*args`, `**kwargs` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `_LayerBasedGpuSinglePassGatherer.__init__`。它接收 `*args`, `**kwargs`，主要用于初始化实例状态与默认值。

### Lines 495-496: Method _LayerBasedGpuSinglePassGatherer.reset
```python
    def reset(self):
        self._data[...] = 0
```
**EN:** This callable implements `_LayerBasedGpuSinglePassGatherer.reset` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `_LayerBasedGpuSinglePassGatherer.reset`，主要用于将配置写入可变状态。

### Lines 498-510: Method _LayerBasedGpuSinglePassGatherer.collect
```python
    def collect(self) -> Dict:
        if self._enable_global_physical_experts:
            global_physical_count = self._data
        else:
            # Can optimize if bottleneck
            global_physical_count = _convert_local_to_global_physical_count(
                self._data,
                rank=self._rank,
                num_local_physical_experts=self._expert_location_metadata.num_local_physical_experts,
                num_physical_experts=self._expert_location_metadata.num_physical_experts,
            )

        return dict(global_physical_count=global_physical_count)
```
**EN:** This callable implements `_LayerBasedGpuSinglePassGatherer.collect` and mainly collects runtime information.
**CN:** 这一可调用对象实现了 `_LayerBasedGpuSinglePassGatherer.collect`，主要用于收集运行时信息。

### Lines 513-513: Class _SelectExpertsSinglePassGatherer
```python
class _SelectExpertsSinglePassGatherer(_LayerBasedGpuSinglePassGatherer):
```
**EN:** This range introduces `_SelectExpertsSinglePassGatherer` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_SelectExpertsSinglePassGatherer`，并定义其后续方法依赖的结构或元数据。

### Lines 514-515: Method _SelectExpertsSinglePassGatherer.__init__
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs, enable_global_physical_experts=True)
```
**EN:** This callable implements `_SelectExpertsSinglePassGatherer.__init__`. It takes `*args`, `**kwargs` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `_SelectExpertsSinglePassGatherer.__init__`。它接收 `*args`, `**kwargs`，主要用于初始化实例状态与默认值。

### Lines 516-517: Class-level scaffolding for _SelectExpertsSinglePassGatherer
```python

    # can optimize (e.g. fuse / compile)
```
**EN:** This callable implements `None._SelectExpertsSinglePassGatherer` and mainly implements Select Experts Single Pass Gatherer. In this range it prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `None._SelectExpertsSinglePassGatherer`，主要用于实现 Select Experts Single Pass Gatherer 相关逻辑。 在这一范围内，它会处理与编译相关的行为。

### Lines 518-523: Method _SelectExpertsSinglePassGatherer.on_select_experts
```python
    def on_select_experts(self, layer_idx: int, topk_ids: torch.Tensor):
        topk_ids = topk_ids.flatten()
        mask = topk_ids != -1
        self._data[layer_idx, :].scatter_add_(
            dim=0, index=topk_ids.masked_fill(~mask, 0).long(), src=mask.int()
        )
```
**EN:** This callable implements `_SelectExpertsSinglePassGatherer.on_select_experts`. It takes `layer_idx`, `topk_ids` and mainly implements on select experts.
**CN:** 这一可调用对象实现了 `_SelectExpertsSinglePassGatherer.on_select_experts`。它接收 `layer_idx`, `topk_ids`，主要用于实现 on select experts 相关逻辑。

### Lines 526-526: Class _DeepepNormalSinglePassGatherer
```python
class _DeepepNormalSinglePassGatherer(_LayerBasedCpuSinglePassGatherer):
```
**EN:** This range introduces `_DeepepNormalSinglePassGatherer` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_DeepepNormalSinglePassGatherer`，并定义其后续方法依赖的结构或元数据。

### Lines 527-533: Method _DeepepNormalSinglePassGatherer.__init__
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        if torch.distributed.get_rank() == 0:
            logger.info(
                "DeepepNormalSinglePassGatherer gathers approximate statistics. "
                "If used with small batch size, consider using expert_distribution_recorder_mode=stat."
            )
```
**EN:** This callable implements `_DeepepNormalSinglePassGatherer.__init__`. It takes `*args`, `**kwargs` and mainly initializes instance state and defaults. In this range it emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `_DeepepNormalSinglePassGatherer.__init__`。它接收 `*args`, `**kwargs`，主要用于初始化实例状态与默认值。 在这一范围内，它会输出日志以便诊断；协调分布式通信。

### Lines 535-544: Method _DeepepNormalSinglePassGatherer.on_deepep_dispatch_normal
```python
    def on_deepep_dispatch_normal(
        self,
        layer_idx: int,
        local_physical_count_of_layer: List[int],
        num_tokens_per_rank,
        num_tokens_per_rdma_rank,
        num_tokens_per_expert,
    ):
        assert isinstance(local_physical_count_of_layer, list)
        self._on_layer_data(layer_idx, local_physical_count_of_layer)
```
**EN:** This callable implements `_DeepepNormalSinglePassGatherer.on_deepep_dispatch_normal`. It takes `layer_idx`, `local_physical_count_of_layer`, `num_tokens_per_rank`, `num_tokens_per_rdma_rank` and mainly implements on deepep dispatch normal. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `_DeepepNormalSinglePassGatherer.on_deepep_dispatch_normal`。它接收 `layer_idx`, `local_physical_count_of_layer`, `num_tokens_per_rank`, `num_tokens_per_rdma_rank`，主要用于实现 on deepep dispatch normal 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 546-556: Method _DeepepNormalSinglePassGatherer.collect
```python
    def collect(self) -> Dict:
        local_physical_count = super()._collect_objects(
            pad_len=self._expert_location_metadata.num_local_physical_experts
        )
        global_physical_count = _convert_local_to_global_physical_count(
            local_physical_count,
            rank=self._rank,
            num_local_physical_experts=self._expert_location_metadata.num_local_physical_experts,
            num_physical_experts=self._expert_location_metadata.num_physical_experts,
        )
        return dict(global_physical_count=global_physical_count)
```
**EN:** This callable implements `_DeepepNormalSinglePassGatherer.collect` and mainly collects runtime information.
**CN:** 这一可调用对象实现了 `_DeepepNormalSinglePassGatherer.collect`，主要用于收集运行时信息。

### Lines 559-559: Class _DeepepLowLatencySinglePassGatherer
```python
class _DeepepLowLatencySinglePassGatherer(_LayerBasedGpuSinglePassGatherer):
```
**EN:** This range introduces `_DeepepLowLatencySinglePassGatherer` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_DeepepLowLatencySinglePassGatherer`，并定义其后续方法依赖的结构或元数据。

### Lines 560-561: Method _DeepepLowLatencySinglePassGatherer.__init__
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs, enable_global_physical_experts=False)
```
**EN:** This callable implements `_DeepepLowLatencySinglePassGatherer.__init__`. It takes `*args`, `**kwargs` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `_DeepepLowLatencySinglePassGatherer.__init__`。它接收 `*args`, `**kwargs`，主要用于初始化实例状态与默认值。

### Lines 563-567: Method _DeepepLowLatencySinglePassGatherer.on_deepep_dispatch_low_latency
```python
    def on_deepep_dispatch_low_latency(
        self, layer_idx: int, local_physical_count_of_layer: torch.Tensor
    ):
        # Most naive implementation, can optimize later
        self._data[layer_idx, :] += local_physical_count_of_layer
```
**EN:** This callable implements `_DeepepLowLatencySinglePassGatherer.on_deepep_dispatch_low_latency`. It takes `layer_idx`, `local_physical_count_of_layer` and mainly implements on deepep dispatch low latency.
**CN:** 这一可调用对象实现了 `_DeepepLowLatencySinglePassGatherer.on_deepep_dispatch_low_latency`。它接收 `layer_idx`, `local_physical_count_of_layer`，主要用于实现 on deepep dispatch low latency 相关逻辑。

### Lines 570-588: Function _convert_per_token_to_global_physical_count
```python
def _convert_per_token_to_global_physical_count(
    num_tokens: int,
    num_layers: int,
    num_physical_experts: int,
    _topk_ids_of_layer: torch.Tensor,
) -> torch.Tensor:
    topk_ids_layer_major = _topk_ids_of_layer[:, :num_tokens, :].reshape(num_layers, -1)
    mask = topk_ids_layer_major != -1

    index = topk_ids_layer_major.masked_fill(~mask, 0).long()
    src = mask.int()

    ans = torch.zeros(
        (num_layers, num_physical_experts),
        dtype=_topk_ids_of_layer.dtype,
        device=_topk_ids_of_layer.device,
    )
    ans.scatter_add_(dim=1, index=index, src=src)
    return ans
```
**EN:** This callable implements `_convert_per_token_to_global_physical_count`. It takes `num_tokens`, `num_layers`, `num_physical_experts`, `_topk_ids_of_layer` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `_convert_per_token_to_global_physical_count`。它接收 `num_tokens`, `num_layers`, `num_physical_experts`, `_topk_ids_of_layer`，主要用于将数据转换为另一种表示。

### Lines 591-605: Function _convert_local_to_global_physical_count
```python
def _convert_local_to_global_physical_count(
    local_physical_count: torch.Tensor,
    rank: int,
    num_local_physical_experts: int,
    num_physical_experts: int,
) -> torch.Tensor:
    dtype = local_physical_count.dtype
    device = local_physical_count.device
    num_layers, _ = local_physical_count.shape

    ans = torch.zeros((num_layers, num_physical_experts), dtype=dtype, device=device)
    ans[
        :, num_local_physical_experts * rank : num_local_physical_experts * (rank + 1)
    ] = local_physical_count
    return ans
```
**EN:** This callable implements `_convert_local_to_global_physical_count`. It takes `local_physical_count`, `rank`, `num_local_physical_experts`, `num_physical_experts` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `_convert_local_to_global_physical_count`。它接收 `local_physical_count`, `rank`, `num_local_physical_experts`, `num_physical_experts`，主要用于将数据转换为另一种表示。

### Lines 606-612: Module-level constants and helpers
```python


# --------------------------------------- Accumulator -----------------------------------------

_SINGLE_PASS_GATHERER_KEY_PRIMARY = "primary"


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 613-613: Class _Accumulator
```python
class _Accumulator(ABC):
```
**EN:** This range introduces `_Accumulator` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_Accumulator`，并定义其后续方法依赖的结构或元数据。

### Lines 614-622: Method _Accumulator.init_new
```python
    @staticmethod
    def init_new(
        server_args: ServerArgs,
        expert_location_metadata: ExpertLocationMetadata,
        rank: int,
    ) -> "_Accumulator":
        return _Accumulator.get_class(server_args)(
            server_args, expert_location_metadata, rank
        )
```
**EN:** This callable implements `_Accumulator.init_new`. It takes `server_args`, `expert_location_metadata`, `rank` and mainly implements init new.
**CN:** 这一可调用对象实现了 `_Accumulator.init_new`。它接收 `server_args`, `expert_location_metadata`, `rank`，主要用于实现 init new 相关逻辑。

### Lines 624-631: Method _Accumulator.get_class
```python
    @staticmethod
    def get_class(server_args: ServerArgs) -> Type["_Accumulator"]:
        return {
            "stat": _StatAccumulator,
            "stat_approx": _StatAccumulator,
            "per_pass": _DetailAccumulator,
            "per_token": _DetailAccumulator,
        }[server_args.expert_distribution_recorder_mode]
```
**EN:** This callable implements `_Accumulator.get_class`. It takes `server_args` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `_Accumulator.get_class`。它接收 `server_args`，主要用于获取某个值或派生视图。

### Lines 633-641: Method _Accumulator.__init__
```python
    def __init__(
        self,
        server_args: ServerArgs,
        expert_location_metadata: ExpertLocationMetadata,
        rank: int,
    ):
        self._server_args = server_args
        self._expert_location_metadata = expert_location_metadata
        self._rank = rank
```
**EN:** This callable implements `_Accumulator.__init__`. It takes `server_args`, `expert_location_metadata`, `rank` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `_Accumulator.__init__`。它接收 `server_args`, `expert_location_metadata`, `rank`，主要用于初始化实例状态与默认值。

### Lines 643-644: Method _Accumulator.get_single_pass_gatherer_keys
```python
    def get_single_pass_gatherer_keys(self):
        return [_SINGLE_PASS_GATHERER_KEY_PRIMARY]
```
**EN:** This callable implements `_Accumulator.get_single_pass_gatherer_keys` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `_Accumulator.get_single_pass_gatherer_keys`，主要用于获取某个值或派生视图。

### Lines 646-647: Method _Accumulator.get_single_pass_gatherer_key
```python
    def get_single_pass_gatherer_key(self, debug_name: Optional[str]):
        return _SINGLE_PASS_GATHERER_KEY_PRIMARY
```
**EN:** This callable implements `_Accumulator.get_single_pass_gatherer_key`. It takes `debug_name` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `_Accumulator.get_single_pass_gatherer_key`。它接收 `debug_name`，主要用于获取某个值或派生视图。

### Lines 649-656: Method _Accumulator.append
```python
    def append(
        self,
        forward_pass_id: int,
        gatherer_key: str,
        single_pass_data: Dict,
        outputs: Dict[str, Any],
    ):
        pass
```
**EN:** This callable implements `_Accumulator.append`. It takes `forward_pass_id`, `gatherer_key`, `single_pass_data`, `outputs` and mainly implements append.
**CN:** 这一可调用对象实现了 `_Accumulator.append`。它接收 `forward_pass_id`, `gatherer_key`, `single_pass_data`, `outputs`，主要用于实现 append 相关逻辑。

### Lines 658-659: Method _Accumulator.reset
```python
    def reset(self):
        pass
```
**EN:** This callable implements `_Accumulator.reset` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `_Accumulator.reset`，主要用于将配置写入可变状态。

### Lines 661-662: Method _Accumulator.dump
```python
    def dump(self, output_mode: _OutputMode):
        pass
```
**EN:** This callable implements `_Accumulator.dump`. It takes `output_mode` and mainly implements dump.
**CN:** 这一可调用对象实现了 `_Accumulator.dump`。它接收 `output_mode`，主要用于实现 dump 相关逻辑。

### Lines 665-665: Class _UtilizationRateAccumulatorMixin
```python
class _UtilizationRateAccumulatorMixin(_Accumulator):
```
**EN:** This range introduces `_UtilizationRateAccumulatorMixin` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_UtilizationRateAccumulatorMixin`，并定义其后续方法依赖的结构或元数据。

### Lines 666-678: Method _UtilizationRateAccumulatorMixin.__init__
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        self._enable = self._server_args.enable_expert_distribution_metrics

        if self._enable:
            self.window_sizes = [10, 100, 1000]
            self._history = _DequeCollection(maxlens=self.window_sizes)
            self._rank = torch.distributed.get_rank()
            self._expert_dispatch_collector = ExpertDispatchCollector(
                self._expert_location_metadata.ep_size
            )
            self._metric_heatmap_collection_counter = 0
```
**EN:** This callable implements `_UtilizationRateAccumulatorMixin.__init__`. It takes `*args`, `**kwargs` and mainly initializes instance state and defaults. In this range it coordinates distributed communication; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `_UtilizationRateAccumulatorMixin.__init__`。它接收 `*args`, `**kwargs`，主要用于初始化实例状态与默认值。 在这一范围内，它会协调分布式通信；记录指标或追踪信号。

### Lines 680-691: Method _UtilizationRateAccumulatorMixin.append
```python
    def append(
        self,
        forward_pass_id: int,
        gatherer_key: str,
        single_pass_data: Dict,
        outputs: Dict[str, Any],
    ):
        super().append(forward_pass_id, gatherer_key, single_pass_data, outputs)
        if self._enable:
            return self._append_utilization_rate(
                forward_pass_id, single_pass_data["global_physical_count"], outputs
            )
```
**EN:** This callable implements `_UtilizationRateAccumulatorMixin.append`. It takes `forward_pass_id`, `gatherer_key`, `single_pass_data`, `outputs` and mainly implements append.
**CN:** 这一可调用对象实现了 `_UtilizationRateAccumulatorMixin.append`。它接收 `forward_pass_id`, `gatherer_key`, `single_pass_data`, `outputs`，主要用于实现 append 相关逻辑。

### Lines 693-696: Method _UtilizationRateAccumulatorMixin.reset
```python
    def reset(self):
        super().reset()
        if self._enable:
            self._history.clear()
```
**EN:** This callable implements `_UtilizationRateAccumulatorMixin.reset` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `_UtilizationRateAccumulatorMixin.reset`，主要用于将配置写入可变状态。

### Lines 698-738: Method _UtilizationRateAccumulatorMixin._append_utilization_rate
```python
    def _append_utilization_rate(
        self,
        forward_pass_id: int,
        single_pass_global_physical_count: torch.Tensor,
        outputs: Dict[str, Any],
    ):
        gpu_physical_count = compute_gpu_physical_count(
            single_pass_global_physical_count,
            num_gpu=self._expert_location_metadata.ep_size,
        )
        gpu_physical_count = gpu_physical_count.to(self._server_args.device)
        torch.distributed.reduce(
            gpu_physical_count, dst=0, op=torch.distributed.ReduceOp.SUM
        )

        if self._rank == 0:
            self._handle_metric_eplb_heatmap(gpu_physical_count)

            utilization_rate_gpu = torch.mean(
                compute_utilization_rate(gpu_physical_count)
            )
            if envs.SGLANG_ENABLE_EPLB_BALANCEDNESS_METRIC.get():
                print(f"hi {self._rank=} {utilization_rate_gpu=}")
                outputs["metrics"] = ExpertDistributionMetrics(
                    eplb_balancedness=utilization_rate_gpu,
                )
            else:
                # TODO maybe refactor this part to also avoid a `.item()` gpu->cpu sync
                utilization_rate_cpu = utilization_rate_gpu.item()
                self._history.append(utilization_rate_cpu)

                gpu_physical_count_sum = gpu_physical_count.sum().item()

                logger.info(
                    f"[Expert Balancedness] "
                    f"forward_pass_id={forward_pass_id} "
                    f"current_pass_balancedness={utilization_rate_cpu:.03f} "
                    f"{''.join(f'last_{size}_average_balancedness={value:.03f} ' for size, value in self._history.mean().items())} "
                    f"gpu_physical_count_sum={gpu_physical_count_sum}"
                    # f"current_pass_per_layer={[round(x, 2) for x in utilization_rate_tensor.cpu().tolist()]}"
                )
```
**EN:** This callable implements `_UtilizationRateAccumulatorMixin._append_utilization_rate`. It takes `forward_pass_id`, `single_pass_global_physical_count`, `outputs` and mainly implements append utilization rate. In this range it emits logs for diagnostics; coordinates distributed communication; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `_UtilizationRateAccumulatorMixin._append_utilization_rate`。它接收 `forward_pass_id`, `single_pass_global_physical_count`, `outputs`，主要用于实现 append utilization rate 相关逻辑。 在这一范围内，它会输出日志以便诊断；协调分布式通信；读取环境变量驱动的配置。

### Lines 739-740: Class-level scaffolding for _UtilizationRateAccumulatorMixin
```python

    # TODO refactor
```
**EN:** This callable implements `None._UtilizationRateAccumulatorMixin` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `None._UtilizationRateAccumulatorMixin`，主要用于将数据转换为另一种表示。

### Lines 741-761: Method _UtilizationRateAccumulatorMixin._handle_metric_eplb_heatmap
```python
    def _handle_metric_eplb_heatmap(self, gpu_physical_count: torch.Tensor):
        # sglang:eplb_gpu_physical_count metric is disabled if SGLANG_EPLB_HEATMAP_COLLECTION_INTERVAL <= 0
        interval = get_int_env_var("SGLANG_EPLB_HEATMAP_COLLECTION_INTERVAL", 0)
        if interval > 0 and self._metric_heatmap_collection_counter % interval == 0:
            for layer_idx in range(self._expert_location_metadata.num_layers):
                count_of_layer = (
                    self._expert_dispatch_collector.eplb_gpu_physical_count.labels(
                        layer=str(layer_idx)
                    )
                )
                # Exclude the +Inf bucket.
                assert (
                    self._expert_location_metadata.ep_size
                    == len(count_of_layer._buckets) - 1
                ), f"{self._expert_location_metadata.ep_size=}, {len(count_of_layer._buckets)=}"
                for gpu_rank in range(self._expert_location_metadata.ep_size):
                    count = gpu_physical_count[layer_idx, gpu_rank]
                    if count > 0:
                        count_of_layer._sum.inc(count * gpu_rank)
                        count_of_layer._buckets[gpu_rank].inc(count)
        self._metric_heatmap_collection_counter += 1
```
**EN:** This callable implements `_UtilizationRateAccumulatorMixin._handle_metric_eplb_heatmap`. It takes `gpu_physical_count` and mainly implements handle metric eplb heatmap. In this range it performs defensive checks on invalid state; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `_UtilizationRateAccumulatorMixin._handle_metric_eplb_heatmap`。它接收 `gpu_physical_count`，主要用于实现 handle metric eplb heatmap 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；记录指标或追踪信号。

### Lines 764-764: Class _DequeCollection
```python
class _DequeCollection:
```
**EN:** This range introduces `_DequeCollection` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_DequeCollection`，并定义其后续方法依赖的结构或元数据。

### Lines 765-766: Method _DequeCollection.__init__
```python
    def __init__(self, maxlens: List[int]):
        self._dequeues = [deque(maxlen=maxlen) for maxlen in maxlens]
```
**EN:** This callable implements `_DequeCollection.__init__`. It takes `maxlens` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `_DequeCollection.__init__`。它接收 `maxlens`，主要用于初始化实例状态与默认值。

### Lines 768-770: Method _DequeCollection.append
```python
    def append(self, value):
        for d in self._dequeues:
            d.append(value)
```
**EN:** This callable implements `_DequeCollection.append`. It takes `value` and mainly implements append.
**CN:** 这一可调用对象实现了 `_DequeCollection.append`。它接收 `value`，主要用于实现 append 相关逻辑。

### Lines 772-774: Method _DequeCollection.clear
```python
    def clear(self):
        for d in self._dequeues:
            d.clear()
```
**EN:** This callable implements `_DequeCollection.clear` and mainly implements clear.
**CN:** 这一可调用对象实现了 `_DequeCollection.clear`，主要用于实现 clear 相关逻辑。

### Lines 776-777: Method _DequeCollection.mean
```python
    def mean(self) -> Dict[int, float]:
        return {d.maxlen: sum(d) / len(d) for d in self._dequeues}
```
**EN:** This callable implements `_DequeCollection.mean` and mainly implements mean.
**CN:** 这一可调用对象实现了 `_DequeCollection.mean`，主要用于实现 mean 相关逻辑。

### Lines 780-780: Class _DetailAccumulator
```python
class _DetailAccumulator(_UtilizationRateAccumulatorMixin):
```
**EN:** This range introduces `_DetailAccumulator` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_DetailAccumulator`，并定义其后续方法依赖的结构或元数据。

### Lines 781-783: Method _DetailAccumulator.__init__
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self._records = []
```
**EN:** This callable implements `_DetailAccumulator.__init__`. It takes `*args`, `**kwargs` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `_DetailAccumulator.__init__`。它接收 `*args`, `**kwargs`，主要用于初始化实例状态与默认值。

### Lines 785-788: Method _DetailAccumulator.get_single_pass_gatherer_keys
```python
    def get_single_pass_gatherer_keys(self):
        if False:  # TODO `server_args.enable_two_batch_overlap`
            return [_SINGLE_PASS_GATHERER_KEY_PRIMARY, "child_a", "child_b"]
        return super().get_single_pass_gatherer_keys()
```
**EN:** This callable implements `_DetailAccumulator.get_single_pass_gatherer_keys` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `_DetailAccumulator.get_single_pass_gatherer_keys`，主要用于获取某个值或派生视图。

### Lines 790-793: Method _DetailAccumulator.get_single_pass_gatherer_key
```python
    def get_single_pass_gatherer_key(self, debug_name: Optional[str]):
        if False:  # TODO `server_args.enable_two_batch_overlap`
            return debug_name or _SINGLE_PASS_GATHERER_KEY_PRIMARY
        return super().get_single_pass_gatherer_key(debug_name)
```
**EN:** This callable implements `_DetailAccumulator.get_single_pass_gatherer_key`. It takes `debug_name` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `_DetailAccumulator.get_single_pass_gatherer_key`。它接收 `debug_name`，主要用于获取某个值或派生视图。

### Lines 795-820: Method _DetailAccumulator.append
```python
    def append(
        self,
        forward_pass_id: int,
        gatherer_key: str,
        single_pass_data: Dict,
        outputs: Dict[str, Any],
    ):
        super().append(forward_pass_id, gatherer_key, single_pass_data, outputs)

        def _process_object(obj):
            if isinstance(obj, torch.Tensor):
                return obj.cpu().clone()
            return obj

        single_pass_data_processed = {
            k: _process_object(v) for k, v in single_pass_data.items()
        }

        self._records.append(
            dict(
                forward_pass_id=forward_pass_id,
                rank=self._rank,
                gatherer_key=gatherer_key,
                **single_pass_data_processed,
            )
        )
```
**EN:** This callable implements `_DetailAccumulator.append`. It takes `forward_pass_id`, `gatherer_key`, `single_pass_data`, `outputs` and mainly implements append.
**CN:** 这一可调用对象实现了 `_DetailAccumulator.append`。它接收 `forward_pass_id`, `gatherer_key`, `single_pass_data`, `outputs`，主要用于实现 append 相关逻辑。

### Lines 822-824: Method _DetailAccumulator.reset
```python
    def reset(self):
        super().reset()
        self._records.clear()
```
**EN:** This callable implements `_DetailAccumulator.reset` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `_DetailAccumulator.reset`，主要用于将配置写入可变状态。

### Lines 826-835: Method _DetailAccumulator.dump
```python
    def dump(self, output_mode: _OutputMode):
        assert output_mode == "file"
        output = dict(
            records=self._records,
            # NOTE: This may change during recording, so here we say it is the "last" one
            last_physical_to_logical_map=self._expert_location_metadata.physical_to_logical_map,
        )
        _dump_to_file(
            f"expert_distribution_recorder_{time.time()}_{self._rank}.pt", output
        )
```
**EN:** This callable implements `_DetailAccumulator.dump`. It takes `output_mode` and mainly implements dump. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `_DetailAccumulator.dump`。它接收 `output_mode`，主要用于实现 dump 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 838-838: Class _StatAccumulator
```python
class _StatAccumulator(_UtilizationRateAccumulatorMixin):
```
**EN:** This range introduces `_StatAccumulator` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_StatAccumulator`，并定义其后续方法依赖的结构或元数据。

### Lines 839-851: Method _StatAccumulator.__init__
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self._global_physical_count_of_buffered_step = _Buffer.init_new(
            item_shape=(
                self._expert_location_metadata.num_layers,
                # Cannot use local_physical_count to support select_experts
                self._expert_location_metadata.num_physical_experts,
            ),
            buffer_size=self._server_args.expert_distribution_recorder_buffer_size,
            dtype=torch.int32,
            device=self._server_args.device,
        )
        self._first_dump = True
```
**EN:** This callable implements `_StatAccumulator.__init__`. It takes `*args`, `**kwargs` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `_StatAccumulator.__init__`。它接收 `*args`, `**kwargs`，主要用于初始化实例状态与默认值。

### Lines 853-864: Method _StatAccumulator.append
```python
    def append(
        self,
        forward_pass_id: int,
        gatherer_key: str,
        single_pass_data: Dict,
        outputs: Dict[str, Any],
    ):
        super().append(forward_pass_id, gatherer_key, single_pass_data, outputs)
        # Can optimize if overhead here is large
        self._global_physical_count_of_buffered_step.append(
            single_pass_data["global_physical_count"]
        )
```
**EN:** This callable implements `_StatAccumulator.append`. It takes `forward_pass_id`, `gatherer_key`, `single_pass_data`, `outputs` and mainly implements append.
**CN:** 这一可调用对象实现了 `_StatAccumulator.append`。它接收 `forward_pass_id`, `gatherer_key`, `single_pass_data`, `outputs`，主要用于实现 append 相关逻辑。

### Lines 866-868: Method _StatAccumulator.reset
```python
    def reset(self):
        super().reset()
        self._global_physical_count_of_buffered_step.reset()
```
**EN:** This callable implements `_StatAccumulator.reset` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `_StatAccumulator.reset`，主要用于将配置写入可变状态。

### Lines 870-898: Method _StatAccumulator.dump
```python
    def dump(self, output_mode: _OutputMode):
        logical_count_of_buffered_step = _convert_global_physical_count_to_logical_count(
            self._global_physical_count_of_buffered_step.get_all(),
            num_layers=self._expert_location_metadata.num_layers,
            num_logical_experts=self._expert_location_metadata.num_logical_experts,
            physical_to_logical_map=self._expert_location_metadata.physical_to_logical_map,
        )

        if self._first_dump:
            self._first_dump = False
            torch.get_device_module().empty_cache()

        torch.distributed.all_reduce(
            logical_count_of_buffered_step, op=torch.distributed.ReduceOp.SUM
        )

        output = dict(
            rank=self._rank,
            logical_count=logical_count_of_buffered_step,
            average_utilization_rate_over_window=self._get_global_average_utilization_rate(),
        )

        if output_mode == "file":
            if self._rank == 0:
                _dump_to_file(f"expert_distribution_recorder_{time.time()}.pt", output)
        elif output_mode == "object":
            return output
        else:
            raise NotImplementedError
```
**EN:** This callable implements `_StatAccumulator.dump`. It takes `output_mode` and mainly implements dump. In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `_StatAccumulator.dump`。它接收 `output_mode`，主要用于实现 dump 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 900-923: Method _StatAccumulator._get_global_average_utilization_rate
```python
    def _get_global_average_utilization_rate(self):
        if not self._enable or math.isclose(
            self._server_args.eplb_min_rebalancing_utilization_threshold, 1.0
        ):
            return None

        if self._rank == 0:
            utilization_mean_rates = self._history.mean()
            window_index = self.window_sizes[-1]
            average_utilization_rate_over_window = (
                utilization_mean_rates[window_index]
                if window_index in utilization_mean_rates
                else 0
            )

            avg_rate_tensor = torch.tensor(
                [average_utilization_rate_over_window],
                dtype=torch.float32,
                device="cuda",
            )
        else:
            avg_rate_tensor = torch.empty(1, dtype=torch.float32, device="cuda")
        torch.distributed.broadcast(avg_rate_tensor, src=0)
        return avg_rate_tensor.item()
```
**EN:** This callable implements `_StatAccumulator._get_global_average_utilization_rate` and mainly retrieves a value or derived view. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `_StatAccumulator._get_global_average_utilization_rate`，主要用于获取某个值或派生视图。 在这一范围内，它会协调分布式通信。

### Lines 926-932: Function _dump_to_file
```python
def _dump_to_file(name, data):
    save_dir = Path(envs.SGLANG_EXPERT_DISTRIBUTION_RECORDER_DIR.get())
    path_output = save_dir / name
    logger.info(f"Write expert distribution to {path_output}")
    if not save_dir.exists():
        save_dir.mkdir(parents=True, exist_ok=True)
    torch.save(data, str(path_output))
```
**EN:** This callable implements `_dump_to_file`. It takes `name`, `data` and mainly converts data into another representation. In this range it emits logs for diagnostics; reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `_dump_to_file`。它接收 `name`, `data`，主要用于将数据转换为另一种表示。 在这一范围内，它会输出日志以便诊断；读取环境变量驱动的配置。

### Lines 935-935: Class _Buffer
```python
class _Buffer:
```
**EN:** This range introduces `_Buffer` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_Buffer`，并定义其后续方法依赖的结构或元数据。

### Lines 936-941: Method _Buffer.init_new
```python
    @staticmethod
    def init_new(item_shape: Tuple, buffer_size: int, dtype, device):
        if buffer_size < 0:
            return _InfiniteBuffer(item_shape, dtype=dtype, device=device)
        else:
            return _CircularBuffer(item_shape, buffer_size, dtype=dtype, device=device)
```
**EN:** This callable implements `_Buffer.init_new`. It takes `item_shape`, `buffer_size`, `dtype`, `device` and mainly implements init new.
**CN:** 这一可调用对象实现了 `_Buffer.init_new`。它接收 `item_shape`, `buffer_size`, `dtype`, `device`，主要用于实现 init new 相关逻辑。

### Lines 943-944: Method _Buffer.append
```python
    def append(self, value: torch.Tensor):
        raise NotImplementedError
```
**EN:** This callable implements `_Buffer.append`. It takes `value` and mainly implements append. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `_Buffer.append`。它接收 `value`，主要用于实现 append 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 946-947: Method _Buffer.get_all
```python
    def get_all(self) -> torch.Tensor:
        raise NotImplementedError
```
**EN:** This callable implements `_Buffer.get_all` and mainly retrieves a value or derived view. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `_Buffer.get_all`，主要用于获取某个值或派生视图。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 949-950: Method _Buffer.reset
```python
    def reset(self):
        raise NotImplementedError
```
**EN:** This callable implements `_Buffer.reset` and mainly applies configuration to mutable state. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `_Buffer.reset`，主要用于将配置写入可变状态。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 953-953: Class _CircularBuffer
```python
class _CircularBuffer(_Buffer):
```
**EN:** This range introduces `_CircularBuffer` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_CircularBuffer`，并定义其后续方法依赖的结构或元数据。

### Lines 954-958: Method _CircularBuffer.__init__
```python
    def __init__(self, item_shape: Tuple, buffer_size: int, dtype, device):
        self._buffer = torch.zeros(
            (buffer_size, *item_shape), dtype=dtype, device=device
        )
        self._curr_index = 0
```
**EN:** This callable implements `_CircularBuffer.__init__`. It takes `item_shape`, `buffer_size`, `dtype`, `device` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `_CircularBuffer.__init__`。它接收 `item_shape`, `buffer_size`, `dtype`, `device`，主要用于初始化实例状态与默认值。

### Lines 960-962: Method _CircularBuffer.append
```python
    def append(self, value: torch.Tensor):
        self._buffer[self._curr_index] = value
        self._curr_index = (self._curr_index + 1) % len(self._buffer)
```
**EN:** This callable implements `_CircularBuffer.append`. It takes `value` and mainly implements append.
**CN:** 这一可调用对象实现了 `_CircularBuffer.append`。它接收 `value`，主要用于实现 append 相关逻辑。

### Lines 964-965: Method _CircularBuffer.get_all
```python
    def get_all(self) -> torch.Tensor:
        return self._buffer
```
**EN:** This callable implements `_CircularBuffer.get_all` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `_CircularBuffer.get_all`，主要用于获取某个值或派生视图。

### Lines 967-968: Method _CircularBuffer.reset
```python
    def reset(self):
        self._buffer[...] = 0
```
**EN:** This callable implements `_CircularBuffer.reset` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `_CircularBuffer.reset`，主要用于将配置写入可变状态。

### Lines 971-971: Class _InfiniteBuffer
```python
class _InfiniteBuffer(_Buffer):
```
**EN:** This range introduces `_InfiniteBuffer` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_InfiniteBuffer`，并定义其后续方法依赖的结构或元数据。

### Lines 972-975: Method _InfiniteBuffer.__init__
```python
    def __init__(self, item_shape: Tuple, dtype, device):
        self._item_shape = item_shape
        self._buffer = torch.zeros((128, *item_shape), dtype=dtype, device=device)
        self._size = 0
```
**EN:** This callable implements `_InfiniteBuffer.__init__`. It takes `item_shape`, `dtype`, `device` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `_InfiniteBuffer.__init__`。它接收 `item_shape`, `dtype`, `device`，主要用于初始化实例状态与默认值。

### Lines 977-990: Method _InfiniteBuffer.append
```python
    def append(self, value: torch.Tensor):
        curr_buffer_size = len(self._buffer)
        dtype = self._buffer.dtype
        device = self._buffer.device

        if self._size == curr_buffer_size:
            new_buffer = torch.zeros(
                (2 * curr_buffer_size, *self._item_shape), dtype=dtype, device=device
            )
            new_buffer[:curr_buffer_size] = self._buffer
            self._buffer = new_buffer

        self._buffer[self._size] = value
        self._size += 1
```
**EN:** This callable implements `_InfiniteBuffer.append`. It takes `value` and mainly implements append.
**CN:** 这一可调用对象实现了 `_InfiniteBuffer.append`。它接收 `value`，主要用于实现 append 相关逻辑。

### Lines 992-993: Method _InfiniteBuffer.get_all
```python
    def get_all(self) -> torch.Tensor:
        return self._buffer[: self._size]
```
**EN:** This callable implements `_InfiniteBuffer.get_all` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `_InfiniteBuffer.get_all`，主要用于获取某个值或派生视图。

### Lines 995-997: Method _InfiniteBuffer.reset
```python
    def reset(self):
        self._buffer[...] = 0
        self._size = 0
```
**EN:** This callable implements `_InfiniteBuffer.reset` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `_InfiniteBuffer.reset`，主要用于将配置写入可变状态。

### Lines 1000-1020: Function _convert_global_physical_count_to_logical_count
```python
def _convert_global_physical_count_to_logical_count(
    # (whatever, num_layers, num_physical_experts)
    global_physical_count: torch.Tensor,
    num_layers: int,
    num_logical_experts: int,
    physical_to_logical_map: torch.Tensor,
):
    dim_extra, _, _ = global_physical_count.shape
    dtype = global_physical_count.dtype
    device = global_physical_count.device
    logical_count = torch.zeros(
        (dim_extra, num_layers, num_logical_experts), dtype=dtype, device=device
    )
    logical_count.scatter_add_(
        dim=2,
        index=physical_to_logical_map.unsqueeze(0)
        .expand(dim_extra, -1, -1)
        .to(torch.int64),
        src=global_physical_count,
    )
    return logical_count
```
**EN:** This callable implements `_convert_global_physical_count_to_logical_count`. It takes `global_physical_count`, `num_layers`, `num_logical_experts`, `physical_to_logical_map` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `_convert_global_physical_count_to_logical_count`。它接收 `global_physical_count`, `num_layers`, `num_logical_experts`, `physical_to_logical_map`，主要用于将数据转换为另一种表示。

### Lines 1023-1033: Function compute_gpu_physical_count
```python
def compute_gpu_physical_count(
    physical_count_of_whatever: torch.Tensor,  # (..., num_layer, num_physical_expert)
    num_gpu: int,
):
    """output: gpu_physical_count_of_batch (..., num_layer, num_gpu)"""
    return einops.reduce(
        physical_count_of_whatever,
        "... num_layer (num_gpu num_expert_per_gpu) -> ... num_layer num_gpu",
        "sum",
        num_gpu=num_gpu,
    )
```
**EN:** This callable implements `compute_gpu_physical_count`. It takes `physical_count_of_whatever`, `num_gpu` and mainly implements compute gpu physical count. The docstring states: "output: gpu_physical_count_of_batch (..., num_layer, num_gpu)"
**CN:** 这一可调用对象实现了 `compute_gpu_physical_count`。它接收 `physical_count_of_whatever`, `num_gpu`，主要用于实现 compute gpu physical count 相关逻辑。

### Lines 1036-1051: Function compute_utilization_rate
```python
def compute_utilization_rate(
    gpu_physical_count_of_batch: torch.Tensor,  # (..., num_layer, num_gpu)
):
    """output: utilization_rate (..., num_layer)"""
    gpu_physical_count_of_batch = gpu_physical_count_of_batch.float()
    max_gpu_physical_count = einops.reduce(
        gpu_physical_count_of_batch,
        "... num_layer num_gpu -> ... num_layer",
        "max",
    )
    avg_gpu_physical_count = einops.reduce(
        gpu_physical_count_of_batch,
        "... num_layer num_gpu -> ... num_layer",
        "mean",
    )
    return (avg_gpu_physical_count + 1e-5) / (max_gpu_physical_count + 1e-5)
```
**EN:** This callable implements `compute_utilization_rate`. It takes `gpu_physical_count_of_batch` and mainly implements compute utilization rate. The docstring states: "output: utilization_rate (..., num_layer)"
**CN:** 这一可调用对象实现了 `compute_utilization_rate`。它接收 `gpu_physical_count_of_batch`，主要用于实现 compute utilization rate 相关逻辑。

## Key Concepts / 关键概念
- `ExpertDistributionMetrics`: core class or state container / 核心类或状态容器
- `ExpertDistributionRecorder`: core class or state container / 核心类或状态容器
- `_ExpertDistributionRecorderNoop`: core class or state container / 核心类或状态容器
- `_ExpertDistributionRecorderReal`: core class or state container / 核心类或状态容器
- `get_global_expert_distribution_recorder`: retrieves a value or derived view / 获取某个值或派生视图
- `set_global_expert_distribution_recorder`: applies configuration to mutable state / 将配置写入可变状态
- `_SinglePassGatherer`: core class or state container / 核心类或状态容器
- `_DetailSinglePassGatherer`: core class or state container / 核心类或状态容器
- `_LayerBasedCpuSinglePassGatherer`: core class or state container / 核心类或状态容器
- `_list_sum`: implements list sum / 实现 list sum 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `math`, `time`, `abc`, `collections`, `contextlib`, `dataclasses`, `pathlib`, `typing`
- **Third-party / 第三方**: `einops`, `torch`, `torch.distributed`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.observability.metrics_collector`, `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.srt.eplb.expert_location`
