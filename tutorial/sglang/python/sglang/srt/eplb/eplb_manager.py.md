# eplb_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/eplb/eplb_manager.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the expert-parallel load balancing part of the SRT runtime and implements logic centered on `eplb_manager`. It exposes primary entry points such as `EPLBManager`, `_chunk_list`. / 该模块属于 SRT 运行时的专家并行负载均衡部分，主要实现围绕 `eplb_manager` 的逻辑。 它对外提供的主要入口包括 `EPLBManager`, `_chunk_list`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Module imports, constants, and setup
```python
import logging
import time
from typing import TYPE_CHECKING, List

import torch.cuda

from sglang.srt.eplb.expert_distribution import get_global_expert_distribution_recorder
from sglang.srt.eplb.expert_location import ExpertLocationMetadata

if TYPE_CHECKING:
    from sglang.srt.model_executor.model_runner import ModelRunner

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断。

### Lines 16-16: Class EPLBManager
```python
class EPLBManager:
```
**EN:** This range introduces `EPLBManager` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `EPLBManager`，并定义其后续方法依赖的结构或元数据。

### Lines 17-39: Method EPLBManager.__init__
```python
    def __init__(self, model_runner: "ModelRunner"):
        super().__init__()
        self._model_runner = model_runner
        self._server_args = model_runner.server_args
        self._rebalance_layers_per_chunk = (
            self._server_args.eplb_rebalance_layers_per_chunk
        )
        self._rebalance_num_iterations = self._server_args.eplb_rebalance_num_iterations

        # Otherwise, the circular buffer will contain stale data. If the case is needed, it can be implemented.
        assert (
            self._server_args.eplb_rebalance_num_iterations
            >= self._server_args.expert_distribution_recorder_buffer_size
        ), "eplb_rebalance_num_iterations must be greater than expert_distribution_recorder_buffer_size"

        if not get_global_expert_distribution_recorder().recording:
            get_global_expert_distribution_recorder().start_record()

        logger.info(
            f"[EPLBManager] system started, will rebalance per {self._rebalance_num_iterations} iterations."
        )

        self._main_generator = self._entrypoint()
```
**EN:** This callable implements `EPLBManager.__init__`. It takes `model_runner` and mainly initializes instance state and defaults. In this range it performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `EPLBManager.__init__`。它接收 `model_runner`，主要用于初始化实例状态与默认值。 在这一范围内，它会对非法状态执行防御性检查；输出日志以便诊断。

### Lines 41-42: Method EPLBManager.on_forward_pass_end
```python
    def on_forward_pass_end(self):
        next(self._main_generator)
```
**EN:** This callable implements `EPLBManager.on_forward_pass_end` and mainly implements on forward pass end.
**CN:** 这一可调用对象实现了 `EPLBManager.on_forward_pass_end`，主要用于实现 on forward pass end 相关逻辑。

### Lines 44-45: Method EPLBManager.reset_generator
```python
    def reset_generator(self):
        self._main_generator = self._entrypoint()
```
**EN:** This callable implements `EPLBManager.reset_generator` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `EPLBManager.reset_generator`，主要用于将数据转换为另一种表示。

### Lines 46-47: Class-level scaffolding for EPLBManager
```python

    # can be more complex if needed
```
**EN:** This callable implements `None.EPLBManager` and mainly implements EPLBManager.
**CN:** 这一可调用对象实现了 `None.EPLBManager`，主要用于实现 EPLBManager 相关逻辑。

### Lines 48-53: Method EPLBManager._entrypoint
```python
    def _entrypoint(self):
        while True:
            for _ in range(self._rebalance_num_iterations):
                yield

            yield from self.rebalance()
```
**EN:** This callable implements `EPLBManager._entrypoint` and mainly implements entrypoint. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `EPLBManager._entrypoint`，主要用于实现 entrypoint 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 55-94: Method EPLBManager.rebalance
```python
    def rebalance(self):
        logger.info("[EPLBManager] rebalance start")

        enable_timing = self._rebalance_layers_per_chunk is None

        if enable_timing:
            torch.get_device_module().synchronize()
            time_start = time.time()

        dump_record_output = get_global_expert_distribution_recorder().dump_record(
            output_mode="object"
        )
        logical_count = dump_record_output["logical_count"]
        average_utilization_rate_over_window = dump_record_output[
            "average_utilization_rate_over_window"
        ]

        # Check whether rebalancing is needed
        if not self._check_rebalance_needed(average_utilization_rate_over_window):
            return

        expert_location_metadata = ExpertLocationMetadata.init_by_eplb(
            self._server_args, self._model_runner.model_config, logical_count
        )

        update_layer_ids_chunks = self._compute_update_layer_ids_chunks()
        for chunk_index, update_layer_ids in enumerate(update_layer_ids_chunks):
            if len(update_layer_ids_chunks) > 1:
                yield
            self._model_runner.update_expert_location(
                expert_location_metadata,
                update_layer_ids=update_layer_ids,
            )

        msg = f"[EPLBManager] rebalance end"
        if enable_timing:
            torch.get_device_module().synchronize()
            time_end = time.time()
            msg += f" time={time_end - time_start:.3f}s"
        logger.info(msg)
```
**EN:** This callable implements `EPLBManager.rebalance` and mainly implements rebalance. In this range it emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `EPLBManager.rebalance`，主要用于实现 rebalance 相关逻辑。 在这一范围内，它会输出日志以便诊断。

### Lines 96-109: Method EPLBManager._check_rebalance_needed
```python
    def _check_rebalance_needed(self, average_utilization_rate_over_window):
        if average_utilization_rate_over_window is None:
            return True

        if (
            average_utilization_rate_over_window
            > self._server_args.eplb_min_rebalancing_utilization_threshold
        ):
            logger.info(
                f"[EPLBManager] Skipped ep rebalancing: current GPU utilization {average_utilization_rate_over_window:.2f} > minimum rebalance threshold {self._server_args.eplb_min_rebalancing_utilization_threshold:.2f}"
            )
            return False

        return True
```
**EN:** This callable implements `EPLBManager._check_rebalance_needed`. It takes `average_utilization_rate_over_window` and mainly checks preconditions and compatibility. In this range it emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `EPLBManager._check_rebalance_needed`。它接收 `average_utilization_rate_over_window`，主要用于检查前置条件与兼容性。 在这一范围内，它会输出日志以便诊断。

### Lines 111-116: Method EPLBManager._compute_update_layer_ids_chunks
```python
    def _compute_update_layer_ids_chunks(self) -> List[List[int]]:
        all_layer_ids = sorted(
            list(self._model_runner.model.routed_experts_weights_of_layer.keys())
        )
        chunk_size = self._rebalance_layers_per_chunk or 1000000
        return list(_chunk_list(all_layer_ids, chunk_size=chunk_size))
```
**EN:** This callable implements `EPLBManager._compute_update_layer_ids_chunks` and mainly updates existing runtime state. In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `EPLBManager._compute_update_layer_ids_chunks`，主要用于更新现有运行时状态。 在这一范围内，它会管理模型权重或检查点。

### Lines 119-121: Function _chunk_list
```python
def _chunk_list(items: List, chunk_size):
    for start_index in range(0, len(items), chunk_size):
        yield items[start_index : start_index + chunk_size]
```
**EN:** This callable implements `_chunk_list`. It takes `items`, `chunk_size` and mainly implements chunk list.
**CN:** 这一可调用对象实现了 `_chunk_list`。它接收 `items`, `chunk_size`，主要用于实现 chunk list 相关逻辑。

## Key Concepts / 关键概念
- `EPLBManager`: core class or state container / 核心类或状态容器
- `_chunk_list`: implements chunk list / 实现 chunk list 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `time`, `typing`
- **Third-party / 第三方**: `torch.cuda`
- **Internal modules / 内部模块**: `sglang.srt.eplb.expert_distribution`, `sglang.srt.eplb.expert_location`, `sglang.srt.model_executor.model_runner`
