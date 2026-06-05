# profile_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/profile_utils.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable SGLang runtime helpers for profiling and performance inspection helpers. / 为 SGLang 运行时提供面向性能剖析与性能观测辅助逻辑的可复用辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Module setup and shared state / 模块设置与共享状态
```python
import logging
import os
import time
from abc import ABC
from dataclasses import dataclass
from pathlib import Path
from typing import Callable, Dict, List, Optional

import torch

from sglang.srt.distributed.parallel_state_wrapper import ParallelState
from sglang.srt.managers.io_struct import ProfileReqOutput
from sglang.srt.model_executor.forward_batch_info import ForwardMode
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import is_npu

_is_npu = is_npu()
if _is_npu:
    import torch_npu

    patches = [
        ["profiler.profile", torch_npu.profiler.profile],
        ["profiler.ProfilerActivity.CUDA", torch_npu.profiler.ProfilerActivity.NPU],
        ["profiler.ProfilerActivity.CPU", torch_npu.profiler.ProfilerActivity.CPU],
    ]
    torch_npu._apply_patches(patches)

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `logging`, `os`, `time`, `abc`, `dataclasses`, `pathlib`. It also defines symbols such as `_is_npu`, `patches`, `logger` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `logging`, `os`, `time`, `abc`, `dataclasses`, `pathlib`。 同时定义了 `_is_npu`, `patches`, `logger` 等符号，供后续逻辑使用。

### Lines 31-31: Class `ProfileManager` declaration / 类 `ProfileManager` 声明
```python
class ProfileManager:
```
**EN:** This class establishes `ProfileManager` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `step`, `configure`, `manual_start`, `manual_stop`, `_do_start`.
**CN:** 该类将 `ProfileManager` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `step`, `configure`, `manual_start`, `manual_stop`, `_do_start` 等方法。

### Lines 32-41: Method `ProfileManager.__init__` / 方法 `ProfileManager.__init__`
```python
    def __init__(self, ps: ParallelState, cpu_group):
        self.stage_based_trigger = _StageBasedTrigger(
            on_start=self._do_start,
            on_stop=self._do_stop,
        )
        self.ps = ps
        self.cpu_group = cpu_group
        self.first_rank_in_node = ps.gpu_id == get_global_server_args().base_gpu_id
        self.profiler_kwargs = None
        self.profiler = None
```
**EN:** This method implements `__init__` on `ProfileManager`. It primarily calls `_StageBasedTrigger`, `get_global_server_args` to complete its work. State updates are written into `self.stage_based_trigger`, `self.ps`, `self.cpu_group`, `self.first_rank_in_node`, `self.profiler_kwargs`, `self.profiler`.
**CN:** 该方法（属于 `ProfileManager`）实现了 `__init__`。 它主要通过调用 `_StageBasedTrigger`, `get_global_server_args` 来完成任务。 状态更新主要写入 `self.stage_based_trigger`, `self.ps`, `self.cpu_group`, `self.first_rank_in_node`, `self.profiler_kwargs`, `self.profiler`。

### Lines 43-48: Method `ProfileManager.step` / 方法 `ProfileManager.step`
```python
    def step(self, forward_mode: ForwardMode):
        stage = _get_stage_from_forward_mode(forward_mode)
        if stage is None:
            return

        self.stage_based_trigger.step(stage=stage)
```
**EN:** This method implements `step` on `ProfileManager`. It primarily calls `_get_stage_from_forward_mode`, `self.stage_based_trigger.step` to complete its work. State updates are written into `stage`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `ProfileManager`）实现了 `step`。 它主要通过调用 `_get_stage_from_forward_mode`, `self.stage_based_trigger.step` 来完成任务。 状态更新主要写入 `stage`。 实现中使用了条件分支。

### Lines 50-91: Method `ProfileManager.configure` / 方法 `ProfileManager.configure`
```python
    def configure(
        self,
        *,
        output_dir: Optional[str],
        start_step: Optional[int],
        num_steps: Optional[int],
        activities: Optional[List[str]],
        with_stack: Optional[bool],
        record_shapes: Optional[bool],
        profile_by_stage: bool,
        profile_id: str,
        merge_profiles: bool,
        profile_prefix: str,
        profile_stages: Optional[List[str]] = None,
    ):
        # not supported yet
        assert start_step is None
        assert (
            profile_by_stage
        ), "only support profile_by_stage=true now"  # `false` can be easily supported
        assert not merge_profiles

        if output_dir is None:
            output_dir = os.getenv("SGLANG_TORCH_PROFILER_DIR", "/tmp")
        if activities is None:
            activities = ["CPU", "GPU"]

        self.profiler_kwargs = dict(
# ... omitted for brevity ...
        self.stage_based_trigger.configure(
            num_steps=num_steps,
            interesting_stages=profile_stages or ["prefill", "decode"],
        )

        return ProfileReqOutput(success=True, message="Succeeded")
```
**EN:** This method implements `configure` on `ProfileManager`. It primarily calls `dict`, `self.stage_based_trigger.configure`, `ProfileReqOutput`, `os.getenv` to complete its work. State updates are written into `self.profiler_kwargs`, `output_dir`, `activities`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `ProfileManager`）实现了 `configure`。 它主要通过调用 `dict`, `self.stage_based_trigger.configure`, `ProfileReqOutput`, `os.getenv` 来完成任务。 状态更新主要写入 `self.profiler_kwargs`, `output_dir`, `activities`。 实现中使用了条件分支。

### Lines 93-94: Method `ProfileManager.manual_start` / 方法 `ProfileManager.manual_start`
```python
    def manual_start(self):
        raise NotImplementedError("manually start is only supported yet")
```
**EN:** This method implements `manual_start` on `ProfileManager`. It primarily calls `NotImplementedError` to complete its work.
**CN:** 该方法（属于 `ProfileManager`）实现了 `manual_start`。 它主要通过调用 `NotImplementedError` 来完成任务。

### Lines 96-97: Method `ProfileManager.manual_stop` / 方法 `ProfileManager.manual_stop`
```python
    def manual_stop(self):
        raise NotImplementedError("manually stop is only supported yet")
```
**EN:** This method implements `manual_stop` on `ProfileManager`. It primarily calls `NotImplementedError` to complete its work.
**CN:** 该方法（属于 `ProfileManager`）实现了 `manual_stop`。 它主要通过调用 `NotImplementedError` 来完成任务。

### Lines 99-114: Method `ProfileManager._do_start` / 方法 `ProfileManager._do_start`
```python
    def _do_start(self, stage: Optional[str] = None):
        logger.info(
            f"Profiling starts{f' for {stage}' if stage else ''}. "
            f"Traces will be saved to: {self.profiler_kwargs['output_dir']} "
            f"(with profile id: {self.profiler_kwargs['profile_id']})",
        )

        assert self.profiler is None
        self.profiler = _ProfilerBase.create(
            **self.profiler_kwargs,
            ps=self.ps,
            cpu_group=self.cpu_group,
            first_rank_in_node=self.first_rank_in_node,
            output_suffix=f"-{stage}" if stage else "",
        )
        self.profiler.start()
```
**EN:** This method implements `_do_start` on `ProfileManager`. It primarily calls `logger.info`, `_ProfilerBase.create`, `self.profiler.start` to complete its work. State updates are written into `self.profiler`.
**CN:** 该方法（属于 `ProfileManager`）实现了 `_do_start`。 它主要通过调用 `logger.info`, `_ProfilerBase.create`, `self.profiler.start` 来完成任务。 状态更新主要写入 `self.profiler`。

### Lines 116-122: Method `ProfileManager._do_stop` / 方法 `ProfileManager._do_stop`
```python
    def _do_stop(self):
        logger.info("Stop profiling...")
        self.profiler.stop()
        logger.info(
            f"Profiling done. Traces are saved to: {self.profiler_kwargs['output_dir']}"
        )
        self.profiler = None
```
**EN:** This method implements `_do_stop` on `ProfileManager`. It primarily calls `logger.info`, `self.profiler.stop` to complete its work. State updates are written into `self.profiler`.
**CN:** 该方法（属于 `ProfileManager`）实现了 `_do_stop`。 它主要通过调用 `logger.info`, `self.profiler.stop` 来完成任务。 状态更新主要写入 `self.profiler`。

### Lines 125-133: Function `_get_stage_from_forward_mode` / 函数 `_get_stage_from_forward_mode`
```python
def _get_stage_from_forward_mode(forward_mode: ForwardMode):
    if forward_mode.is_prefill():
        return "prefill"
    elif forward_mode.is_decode():
        return "decode"
    elif forward_mode.is_idle():
        return None
    else:
        raise RuntimeError(f"unsupported profile stage: {forward_mode=}")
```
**EN:** This function implements `_get_stage_from_forward_mode`. It primarily calls `forward_mode.is_prefill`, `forward_mode.is_decode`, `forward_mode.is_idle`, `RuntimeError` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_get_stage_from_forward_mode`。 它主要通过调用 `forward_mode.is_prefill`, `forward_mode.is_decode`, `forward_mode.is_idle`, `RuntimeError` 来完成任务。 实现中使用了条件分支。

### Lines 139-139: Class `_StageBasedTrigger` declaration / 类 `_StageBasedTrigger` 声明
```python
class _StageBasedTrigger:
```
**EN:** This class establishes `_StageBasedTrigger` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `configure`, `step`.
**CN:** 该类将 `_StageBasedTrigger` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `configure`, `step` 等方法。

### Lines 140-142: Class `_StageConfig` declaration / 类 `_StageConfig` 声明
```python
    @dataclass
    class _StageConfig:
        target_count: int
```
**EN:** This class establishes `_StageConfig` as a compact data container for the surrounding logic. The main stored fields include `target_count`.
**CN:** 该类将 `_StageConfig` 定义为周边逻辑的紧凑的数据容器。 其主要存储字段包括 `target_count`。

### Lines 144-147: Class `_RunningState` declaration / 类 `_RunningState` 声明
```python
    @dataclass
    class _RunningState:
        curr_stage: str
        curr_count: int
```
**EN:** This class establishes `_RunningState` as a compact data container for the surrounding logic. The main stored fields include `curr_stage`, `curr_count`.
**CN:** 该类将 `_RunningState` 定义为周边逻辑的紧凑的数据容器。 其主要存储字段包括 `curr_stage`, `curr_count`。

### Lines 149-155: Method `_StageBasedTrigger.__init__` / 方法 `_StageBasedTrigger.__init__`
```python
    def __init__(self, on_start: Callable, on_stop: Callable):
        self.on_start = on_start
        self.on_stop = on_stop

        self.running_state: Optional[_StageBasedTrigger._RunningState] = None
        # When a stage is in the dict, it means it is being or should be executed
        self.stage_configs: Dict[str, _StageBasedTrigger._StageConfig] = {}
```
**EN:** This method implements `__init__` on `_StageBasedTrigger`. State updates are written into `self.on_start`, `self.on_stop`, `self.running_state`, `self.stage_configs`.
**CN:** 该方法（属于 `_StageBasedTrigger`）实现了 `__init__`。 状态更新主要写入 `self.on_start`, `self.on_stop`, `self.running_state`, `self.stage_configs`。

### Lines 157-162: Method `_StageBasedTrigger.configure` / 方法 `_StageBasedTrigger.configure`
```python
    def configure(self, num_steps: int, interesting_stages: List[str]):
        assert self.running_state is None
        self.stage_configs = {
            stage: self._StageConfig(target_count=num_steps)
            for stage in interesting_stages
        }
```
**EN:** This method implements `configure` on `_StageBasedTrigger`. It primarily calls `self._StageConfig` to complete its work. State updates are written into `self.stage_configs`.
**CN:** 该方法（属于 `_StageBasedTrigger`）实现了 `configure`。 它主要通过调用 `self._StageConfig` 来完成任务。 状态更新主要写入 `self.stage_configs`。

### Lines 164-189: Method `_StageBasedTrigger.step` / 方法 `_StageBasedTrigger.step`
```python
    def step(self, stage: str):
        # Incr counter
        if (s := self.running_state) is not None:
            s.curr_count += 1

        # Maybe stop
        if ((s := self.running_state) is not None) and (
            (s.curr_count > self.stage_configs[s.curr_stage].target_count)
            or (stage != s.curr_stage)
        ):
            del self.stage_configs[s.curr_stage]
            self.running_state = None
            self.on_stop()

        # Maybe start
        if (self.running_state is None) and (stage in self.stage_configs):
            self.running_state = self._RunningState(
                curr_stage=stage,
                curr_count=0,
            )
            self.on_start(stage=stage)

        # Sanity check
        assert (self.running_state is not None) == (stage in self.stage_configs)
        if (s := self.running_state) is not None:
            assert s.curr_stage == stage
```
**EN:** This method implements `step` on `_StageBasedTrigger`. It primarily calls `self.on_stop`, `self._RunningState`, `self.on_start` to complete its work. State updates are written into `s.curr_count`, `self.running_state`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `_StageBasedTrigger`）实现了 `step`。 它主要通过调用 `self.on_stop`, `self._RunningState`, `self.on_start` 来完成任务。 状态更新主要写入 `s.curr_count`, `self.running_state`。 实现中使用了条件分支。

### Lines 195-195: Class `_ProfilerBase` declaration / 类 `_ProfilerBase` 声明
```python
class _ProfilerBase(ABC):
```
**EN:** This class establishes `_ProfilerBase` as the main container/coordinator for the surrounding logic. It inherits from `ABC`. Its core interface includes methods such as `create`, `start`, `stop`.
**CN:** 该类将 `_ProfilerBase` 定义为周边逻辑的主要封装体或协调者。 它继承自 `ABC`。 其核心接口包括 `create`, `start`, `stop` 等方法。

### Lines 196-215: Method `_ProfilerBase.create` / 方法 `_ProfilerBase.create`
```python
    @staticmethod
    def create(activities, with_stack, record_shapes, **kwargs):
        inners = []
        if ("CPU" in activities) or ("GPU" in activities):
            inners.append(
                _ProfilerTorch(
                    **kwargs,
                    activities=activities,
                    with_stack=with_stack,
                    record_shapes=record_shapes,
                )
            )
        if "MEM" in activities:
            inners.append(_ProfilerMemory(**kwargs))
        if "CUDA_PROFILER" in activities:
            inners.append(_ProfilerCudart(**kwargs))
        if "RPD" in activities:  # for ROCM
            inners.append(_ProfilerRPD(**kwargs))

        return _ProfilerList(inners)
```
**EN:** This method implements `create` on `_ProfilerBase`. It primarily calls `_ProfilerList`, `inners.append`, `_ProfilerTorch`, `_ProfilerMemory`, `_ProfilerCudart`, `_ProfilerRPD` to complete its work. State updates are written into `inners`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `_ProfilerBase`）实现了 `create`。 它主要通过调用 `_ProfilerList`, `inners.append`, `_ProfilerTorch`, `_ProfilerMemory`, `_ProfilerCudart`, `_ProfilerRPD` 来完成任务。 状态更新主要写入 `inners`。 实现中使用了条件分支。

### Lines 217-218: Method `_ProfilerBase.start` / 方法 `_ProfilerBase.start`
```python
    def start(self):
        raise NotImplementedError
```
**EN:** This method implements `start` on `_ProfilerBase`.
**CN:** 该方法（属于 `_ProfilerBase`）实现了 `start`。

### Lines 220-221: Method `_ProfilerBase.stop` / 方法 `_ProfilerBase.stop`
```python
    def stop(self):
        raise NotImplementedError
```
**EN:** This method implements `stop` on `_ProfilerBase`.
**CN:** 该方法（属于 `_ProfilerBase`）实现了 `stop`。

### Lines 224-224: Class `_ProfilerList` declaration / 类 `_ProfilerList` 声明
```python
class _ProfilerList(_ProfilerBase):
```
**EN:** This class establishes `_ProfilerList` as the main container/coordinator for the surrounding logic. It inherits from `_ProfilerBase`. Its core interface includes methods such as `__init__`, `start`, `stop`.
**CN:** 该类将 `_ProfilerList` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_ProfilerBase`。 其核心接口包括 `__init__`, `start`, `stop` 等方法。

### Lines 225-226: Method `_ProfilerList.__init__` / 方法 `_ProfilerList.__init__`
```python
    def __init__(self, inners: List[_ProfilerBase]):
        self.inners = inners
```
**EN:** This method implements `__init__` on `_ProfilerList`. State updates are written into `self.inners`.
**CN:** 该方法（属于 `_ProfilerList`）实现了 `__init__`。 状态更新主要写入 `self.inners`。

### Lines 228-230: Method `_ProfilerList.start` / 方法 `_ProfilerList.start`
```python
    def start(self):
        for inner in self.inners:
            inner.start()
```
**EN:** This method implements `start` on `_ProfilerList`. It primarily calls `inner.start` to complete its work. The implementation relies on iteration.
**CN:** 该方法（属于 `_ProfilerList`）实现了 `start`。 它主要通过调用 `inner.start` 来完成任务。 实现中使用了迭代逻辑。

### Lines 232-234: Method `_ProfilerList.stop` / 方法 `_ProfilerList.stop`
```python
    def stop(self):
        for inner in self.inners:
            inner.stop()
```
**EN:** This method implements `stop` on `_ProfilerList`. It primarily calls `inner.stop` to complete its work. The implementation relies on iteration.
**CN:** 该方法（属于 `_ProfilerList`）实现了 `stop`。 它主要通过调用 `inner.stop` 来完成任务。 实现中使用了迭代逻辑。

### Lines 237-237: Class `_ProfilerConcreteBase` declaration / 类 `_ProfilerConcreteBase` 声明
```python
class _ProfilerConcreteBase(_ProfilerBase):
```
**EN:** This class establishes `_ProfilerConcreteBase` as the main container/coordinator for the surrounding logic. It inherits from `_ProfilerBase`. Its core interface includes methods such as `__init__`.
**CN:** 该类将 `_ProfilerConcreteBase` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_ProfilerBase`。 其核心接口包括 `__init__` 等方法。

### Lines 238-254: Method `_ProfilerConcreteBase.__init__` / 方法 `_ProfilerConcreteBase.__init__`
```python
    def __init__(
        self,
        output_dir: str,
        output_prefix: str,
        output_suffix: str,
        profile_id: str,
        ps: ParallelState,
        cpu_group,
        first_rank_in_node: bool,
    ):
        self.output_dir = output_dir
        self.output_prefix = output_prefix
        self.output_suffix = output_suffix
        self.profile_id = profile_id
        self.ps = ps
        self.cpu_group = cpu_group
        self.first_rank_in_node = first_rank_in_node
```
**EN:** This method implements `__init__` on `_ProfilerConcreteBase`. State updates are written into `self.output_dir`, `self.output_prefix`, `self.output_suffix`, `self.profile_id`, `self.ps`, `self.cpu_group`.
**CN:** 该方法（属于 `_ProfilerConcreteBase`）实现了 `__init__`。 状态更新主要写入 `self.output_dir`, `self.output_prefix`, `self.output_suffix`, `self.profile_id`, `self.ps`, `self.cpu_group`。

### Lines 257-257: Class `_ProfilerTorch` declaration / 类 `_ProfilerTorch` 声明
```python
class _ProfilerTorch(_ProfilerConcreteBase):
```
**EN:** This class establishes `_ProfilerTorch` as the main container/coordinator for the surrounding logic. It inherits from `_ProfilerConcreteBase`. Its core interface includes methods such as `__init__`, `start`, `stop`.
**CN:** 该类将 `_ProfilerTorch` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_ProfilerConcreteBase`。 其核心接口包括 `__init__`, `start`, `stop` 等方法。

### Lines 258-262: Method `_ProfilerTorch.__init__` / 方法 `_ProfilerTorch.__init__`
```python
    def __init__(self, with_stack: bool, record_shapes: bool, activities, **kwargs):
        super().__init__(**kwargs)
        self.with_stack = with_stack
        self.record_shapes = record_shapes
        self.activities = activities
```
**EN:** This method implements `__init__` on `_ProfilerTorch`. It primarily calls `super.__init__`, `super` to complete its work. State updates are written into `self.with_stack`, `self.record_shapes`, `self.activities`.
**CN:** 该方法（属于 `_ProfilerTorch`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `super` 来完成任务。 状态更新主要写入 `self.with_stack`, `self.record_shapes`, `self.activities`。

### Lines 264-285: Method `_ProfilerTorch.start` / 方法 `_ProfilerTorch.start`
```python
    def start(self):
        activity_map = {
            "CPU": torch.profiler.ProfilerActivity.CPU,
            "GPU": torch.profiler.ProfilerActivity.CUDA,
        }
        torchprof_activities = [
            activity_map[a] for a in self.activities if a in activity_map
        ]

        self.torch_profiler = torch.profiler.profile(
            activities=torchprof_activities,
            with_stack=self.with_stack if self.with_stack is not None else True,
            record_shapes=(
                self.record_shapes if self.record_shapes is not None else False
            ),
            on_trace_ready=(
                None
                if not _is_npu
                else torch_npu.profiler.tensorboard_trace_handler(self.output_dir)
            ),
        )
        self.torch_profiler.start()
```
**EN:** This method implements `start` on `_ProfilerTorch`. It primarily calls `torch.profiler.profile`, `self.torch_profiler.start`, `torch_npu.profiler.tensorboard_trace_handler` to complete its work. State updates are written into `activity_map`, `torchprof_activities`, `self.torch_profiler`.
**CN:** 该方法（属于 `_ProfilerTorch`）实现了 `start`。 它主要通过调用 `torch.profiler.profile`, `self.torch_profiler.start`, `torch_npu.profiler.tensorboard_trace_handler` 来完成任务。 状态更新主要写入 `activity_map`, `torchprof_activities`, `self.torch_profiler`。

### Lines 287-313: Method `_ProfilerTorch.stop` / 方法 `_ProfilerTorch.stop`
```python
    def stop(self):
        Path(self.output_dir).mkdir(parents=True, exist_ok=True)

        self.torch_profiler.stop()
        if not _is_npu:
            # Build filename with only non-zero ranks to maintain backward compatibility
            filename_parts = [self.profile_id, f"TP-{self.ps.tp_rank}"]

            # Only add other ranks if parallelism is enabled (size > 1)
            if self.ps.dp_size > 1:
                filename_parts.append(f"DP-{self.ps.dp_rank}")
            if self.ps.pp_size > 1:
                filename_parts.append(f"PP-{self.ps.pp_rank}")
            if self.ps.moe_ep_size > 1:
                filename_parts.append(f"EP-{self.ps.moe_ep_rank}")

            filename = (
                (self.output_prefix + "-" if self.output_prefix else "")
                + "-".join(filename_parts)
                + self.output_suffix
                + ".trace.json.gz"
            )

            self.torch_profiler.export_chrome_trace(
                os.path.join(self.output_dir, filename)
            )
        torch.distributed.barrier(self.cpu_group)
```
**EN:** This method implements `stop` on `_ProfilerTorch`. It primarily calls `Path.mkdir`, `self.torch_profiler.stop`, `torch.distributed.barrier`, `self.torch_profiler.export_chrome_trace`, `Path`, `filename_parts.append` to complete its work. State updates are written into `filename_parts`, `filename`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `_ProfilerTorch`）实现了 `stop`。 它主要通过调用 `Path.mkdir`, `self.torch_profiler.stop`, `torch.distributed.barrier`, `self.torch_profiler.export_chrome_trace`, `Path`, `filename_parts.append` 来完成任务。 状态更新主要写入 `filename_parts`, `filename`。 实现中使用了条件分支。

### Lines 318-318: Class `_ProfilerMemory` declaration / 类 `_ProfilerMemory` 声明
```python
class _ProfilerMemory(_ProfilerConcreteBase):
```
**EN:** This class establishes `_ProfilerMemory` as the main container/coordinator for the surrounding logic. It inherits from `_ProfilerConcreteBase`. Its core interface includes methods such as `start`, `stop`.
**CN:** 该类将 `_ProfilerMemory` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_ProfilerConcreteBase`。 其核心接口包括 `start`, `stop` 等方法。

### Lines 319-320: Method `_ProfilerMemory.start` / 方法 `_ProfilerMemory.start`
```python
    def start(self):
        torch.cuda.memory._record_memory_history(max_entries=100000)
```
**EN:** This method implements `start` on `_ProfilerMemory`. It primarily calls `torch.cuda.memory._record_memory_history` to complete its work.
**CN:** 该方法（属于 `_ProfilerMemory`）实现了 `start`。 它主要通过调用 `torch.cuda.memory._record_memory_history` 来完成任务。

### Lines 322-333: Method `_ProfilerMemory.stop` / 方法 `_ProfilerMemory.stop`
```python
    def stop(self):
        Path(self.output_dir).mkdir(parents=True, exist_ok=True)

        memory_profile_path = os.path.join(
            self.output_dir,
            str(time.time())
            + f"-TP-{self.ps.tp_rank}-memory"
            + self.output_suffix
            + ".pickle",
        )
        torch.cuda.memory._dump_snapshot(memory_profile_path)
        torch.cuda.memory._record_memory_history(enabled=None)
```
**EN:** This method implements `stop` on `_ProfilerMemory`. It primarily calls `Path.mkdir`, `os.path.join`, `torch.cuda.memory._dump_snapshot`, `torch.cuda.memory._record_memory_history`, `Path`, `str` to complete its work. State updates are written into `memory_profile_path`.
**CN:** 该方法（属于 `_ProfilerMemory`）实现了 `stop`。 它主要通过调用 `Path.mkdir`, `os.path.join`, `torch.cuda.memory._dump_snapshot`, `torch.cuda.memory._record_memory_history`, `Path`, `str` 来完成任务。 状态更新主要写入 `memory_profile_path`。

### Lines 336-336: Class `_ProfilerCudart` declaration / 类 `_ProfilerCudart` 声明
```python
class _ProfilerCudart(_ProfilerConcreteBase):
```
**EN:** This class establishes `_ProfilerCudart` as the main container/coordinator for the surrounding logic. It inherits from `_ProfilerConcreteBase`. Its core interface includes methods such as `start`, `stop`.
**CN:** 该类将 `_ProfilerCudart` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_ProfilerConcreteBase`。 其核心接口包括 `start`, `stop` 等方法。

### Lines 337-340: Method `_ProfilerCudart.start` / 方法 `_ProfilerCudart.start`
```python
    def start(self):
        if self.first_rank_in_node:
            logger.info(f"Call cudaProfilerStart")
            torch.cuda.cudart().cudaProfilerStart()
```
**EN:** This method implements `start` on `_ProfilerCudart`. It primarily calls `logger.info`, `torch.cuda.cudart.cudaProfilerStart`, `torch.cuda.cudart` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `_ProfilerCudart`）实现了 `start`。 它主要通过调用 `logger.info`, `torch.cuda.cudart.cudaProfilerStart`, `torch.cuda.cudart` 来完成任务。 实现中使用了条件分支。

### Lines 342-345: Method `_ProfilerCudart.stop` / 方法 `_ProfilerCudart.stop`
```python
    def stop(self):
        if self.first_rank_in_node:
            logger.info(f"Call cudaProfilerStop")
            torch.cuda.cudart().cudaProfilerStop()
```
**EN:** This method implements `stop` on `_ProfilerCudart`. It primarily calls `logger.info`, `torch.cuda.cudart.cudaProfilerStop`, `torch.cuda.cudart` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `_ProfilerCudart`）实现了 `stop`。 它主要通过调用 `logger.info`, `torch.cuda.cudart.cudaProfilerStop`, `torch.cuda.cudart` 来完成任务。 实现中使用了条件分支。

### Lines 348-348: Class `_ProfilerRPD` declaration / 类 `_ProfilerRPD` 声明
```python
class _ProfilerRPD(_ProfilerConcreteBase):
```
**EN:** This class establishes `_ProfilerRPD` as the main container/coordinator for the surrounding logic. It inherits from `_ProfilerConcreteBase`. Its core interface includes methods such as `start`, `stop`.
**CN:** 该类将 `_ProfilerRPD` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_ProfilerConcreteBase`。 其核心接口包括 `start`, `stop` 等方法。

### Lines 349-378: Method `_ProfilerRPD.start` / 方法 `_ProfilerRPD.start`
```python
    def start(self):
        Path(self.output_dir).mkdir(parents=True, exist_ok=True)

        from rpdTracerControl import rpdTracerControl

        rpdTracerControl.skipCreate()

        self.rpd_profile_path = os.path.join(
            self.output_dir,
            "rpd-" + str(time.time()) + f"-TP-{self.ps.tp_rank}" + ".trace.json.gz",
        )

        if self.ps.tp_rank == 0:
            import sqlite3

            from rocpd.schema import RocpdSchema

            if os.path.exists("trace.rpd"):
                os.unlink("trace.rpd")
            schema = RocpdSchema()
            connection = sqlite3.connect("trace.rpd")
            schema.writeSchema(connection)
            connection.commit()
            del connection
        torch.distributed.barrier(self.cpu_group)

        self.rpd_profiler = rpdTracerControl()
        self.rpd_profiler.setPythonTrace(True)
        self.rpd_profiler.start()
        self.rpd_profiler.rangePush("", "rpd profile range", "")
```
**EN:** This method implements `start` on `_ProfilerRPD`. It primarily calls `Path.mkdir`, `rpdTracerControl.skipCreate`, `os.path.join`, `torch.distributed.barrier`, `rpdTracerControl`, `self.rpd_profiler.setPythonTrace` to complete its work. State updates are written into `self.rpd_profile_path`, `self.rpd_profiler`, `schema`, `connection`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `_ProfilerRPD`）实现了 `start`。 它主要通过调用 `Path.mkdir`, `rpdTracerControl.skipCreate`, `os.path.join`, `torch.distributed.barrier`, `rpdTracerControl`, `self.rpd_profiler.setPythonTrace` 来完成任务。 状态更新主要写入 `self.rpd_profile_path`, `self.rpd_profiler`, `schema`, `connection`。 实现中使用了条件分支。

### Lines 380-389: Method `_ProfilerRPD.stop` / 方法 `_ProfilerRPD.stop`
```python
    def stop(self):
        self.rpd_profiler.rangePop()
        self.rpd_profiler.stop()
        self.rpd_profiler.flush()

        torch.distributed.barrier(self.cpu_group)
        if self.ps.tp_rank == 0:
            from sglang.srt.utils.rpd_utils import rpd_to_chrome_trace

            rpd_to_chrome_trace("trace.rpd", self.rpd_profile_path)
```
**EN:** This method implements `stop` on `_ProfilerRPD`. It primarily calls `self.rpd_profiler.rangePop`, `self.rpd_profiler.stop`, `self.rpd_profiler.flush`, `torch.distributed.barrier`, `rpd_to_chrome_trace` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `_ProfilerRPD`）实现了 `stop`。 它主要通过调用 `self.rpd_profiler.rangePop`, `self.rpd_profiler.stop`, `self.rpd_profiler.flush`, `torch.distributed.barrier`, `rpd_to_chrome_trace` 来完成任务。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Classes / 类**: `ProfileManager`, `_StageBasedTrigger`, `_ProfilerBase`, `_ProfilerList`, `_ProfilerConcreteBase`, `_ProfilerTorch`, `_ProfilerMemory`, `_ProfilerCudart`
- **Functions / 函数**: `_get_stage_from_forward_mode`, `__init__`, `step`, `configure`, `manual_start`, `manual_stop`, `_do_start`, `_do_stop`
- **Themes / 主题**: `profile`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.distributed.parallel_state_wrapper`, `sglang.srt.managers.io_struct`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.srt.utils.rpd_utils`
- **External / 外部依赖**: `torch`, `torch_npu`, `rpdTracerControl`, `rocpd.schema`
- **Standard library / 标准库**: `logging`, `os`, `time`, `abc`, `dataclasses`, `pathlib`, `typing`, `sqlite3`
