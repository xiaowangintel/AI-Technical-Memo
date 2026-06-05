# simulator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/schedule_simulator/simulator.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on request scheduling simulation. It mainly models scheduler behavior so engineers can reason about queueing and routing decisions offline. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于请求调度模拟。它主要用于对调度器行为建模，使工程师可以离线分析排队与路由决策。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Import dependencies and shared types / 导入依赖与共享类型
```python
from dataclasses import dataclass
from typing import Any, Dict, List, Optional

from sglang.srt.debug_utils.schedule_simulator.gpu_state import GPUState, StepRecord
from sglang.srt.debug_utils.schedule_simulator.metrics import MetricRecorder
from sglang.srt.debug_utils.schedule_simulator.request import SimRequest
from sglang.srt.debug_utils.schedule_simulator.routers.base import RouterPolicy
from sglang.srt.debug_utils.schedule_simulator.schedulers.base import SchedulerPolicy
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 12-12: Define class `SimulationResult` and class context / 定义类 `SimulationResult`及类上下文
```python
class SimulationResult:
```
**EN:** This section introduces `SimulationResult`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `SimulationResult`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 13-14: Declare fields for `SimulationResult` such as `step_records`, `summary` / 为 `SimulationResult` 声明字段，例如 `step_records`, `summary`
```python
    step_records: List[StepRecord]
    summary: Dict[str, Any]
```
**EN:** These lines declare the state carried by `SimulationResult`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `SimulationResult` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 17-17: Define class `Simulator` and class context / 定义类 `Simulator`及类上下文
```python
class Simulator:
```
**EN:** This section introduces `Simulator`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `Simulator`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 18-38: Implement method `__init__` for `Simulator` / 为 `Simulator` 实现方法 `__init__`
```python
    def __init__(
        self,
        num_gpus_per_engine: int,
        router: RouterPolicy,
        scheduler: SchedulerPolicy,
        recorders: Optional[List[MetricRecorder]] = None,
        log_level: int = 0,
        max_total_tokens: int = 100000,
        stop_criteria: str = "all_done",
        max_steps: Optional[int] = None,
    ):
        self.num_gpus_per_engine = num_gpus_per_engine
        self.router = router
        self.scheduler = scheduler
        self.recorders = recorders or []
        self.log_level = log_level
        self.max_total_tokens = max_total_tokens
        self.stop_criteria = stop_criteria
        self.max_steps = max_steps
        self.gpu_states: List[GPUState] = []
        self.step = 0
```
**EN:** Method `__init__` implements behavior on `Simulator`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__init__` 为 `Simulator` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 40-63: Implement method `run` for `Simulator` / 为 `Simulator` 实现方法 `run`
```python
    def run(self, requests: List[SimRequest]) -> SimulationResult:
        self.gpu_states = [
            GPUState(gpu_id=i, max_total_tokens=self.max_total_tokens)
            for i in range(self.num_gpus_per_engine)
        ]
        self.step = 0
        step_records: List[StepRecord] = []
        incoming_requests = list(requests)

        while True:
            self._route_requests(incoming_requests)
            incoming_requests.clear()
            self._schedule_all_gpus()
            if self._should_stop():
                break
            self._execute_step()
            step_records.extend(
                gpu.get_step_record(self.step) for gpu in self.gpu_states
            )
            self._log_step()
            self._record_metrics()
            self.step += 1

        return SimulationResult(step_records=step_records, summary=self._get_summary())
```
**EN:** Method `run` implements behavior on `Simulator`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `run` 为 `Simulator` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 65-74: Implement method `_should_stop` for `Simulator` / 为 `Simulator` 实现方法 `_should_stop`
```python
    def _should_stop(self) -> bool:
        if self.max_steps is not None and self.step >= self.max_steps:
            return True
        if self.stop_criteria == "exist_no_pending":
            return any(not gpu.pending_requests for gpu in self.gpu_states)
        if self.stop_criteria == "all_done":
            return not any(
                gpu.pending_requests or gpu.running_requests for gpu in self.gpu_states
            )
        raise ValueError(f"Unknown stop criteria: {self.stop_criteria}")
```
**EN:** Method `_should_stop` implements behavior on `Simulator`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_should_stop` 为 `Simulator` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 76-80: Implement method `_route_requests` for `Simulator` / 为 `Simulator` 实现方法 `_route_requests`
```python
    def _route_requests(self, incoming_requests: List[SimRequest]) -> None:
        for req in incoming_requests:
            gpu_id = self.router.route(req)
            if gpu_id < self.num_gpus_per_engine:
                self.gpu_states[gpu_id].pending_requests.append(req)
```
**EN:** Method `_route_requests` implements behavior on `Simulator`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_route_requests` 为 `Simulator` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 82-88: Implement method `_schedule_all_gpus` for `Simulator` / 为 `Simulator` 实现方法 `_schedule_all_gpus`
```python
    def _schedule_all_gpus(self) -> None:
        for gpu in self.gpu_states:
            self.scheduler.schedule(gpu)
            assert gpu.is_valid(), (
                f"GPU{gpu.gpu_id} invalid after scheduling "
                f"({gpu.total_seq_len()=}, {gpu.max_total_tokens=})"
            )
```
**EN:** Method `_schedule_all_gpus` implements behavior on `Simulator`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_schedule_all_gpus` 为 `Simulator` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 90-92: Implement method `_execute_step` for `Simulator` / 为 `Simulator` 实现方法 `_execute_step`
```python
    def _execute_step(self) -> None:
        for gpu in self.gpu_states:
            gpu.execute_step()
```
**EN:** Method `_execute_step` implements behavior on `Simulator`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_execute_step` 为 `Simulator` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 94-106: Implement method `_log_step` for `Simulator` / 为 `Simulator` 实现方法 `_log_step`
```python
    def _log_step(self) -> None:
        if self.log_level == 0 and self.step % 100 != 0:
            return
        parts = [f"step={self.step:<4}"]
        for gpu in self.gpu_states:
            r, q = len(gpu.running_requests), len(gpu.pending_requests)
            if self.log_level <= 1:
                parts.append(f"GPU{gpu.gpu_id}[R={r:<3} Q={q:<3}]")
            else:
                run_ids = _format_ids(gpu.running_requests)
                queue_ids = _format_ids(gpu.pending_requests)
                parts.append(f"GPU{gpu.gpu_id}[R={r}:{run_ids} Q={q}:{queue_ids}]")
        print(" | ".join(parts))
```
**EN:** Method `_log_step` implements behavior on `Simulator`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_log_step` 为 `Simulator` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 108-110: Implement method `_record_metrics` for `Simulator` / 为 `Simulator` 实现方法 `_record_metrics`
```python
    def _record_metrics(self) -> None:
        for recorder in self.recorders:
            recorder.on_step_end(self.step, self.gpu_states)
```
**EN:** Method `_record_metrics` implements behavior on `Simulator`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_record_metrics` 为 `Simulator` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 112-113: Implement method `_get_summary` for `Simulator` / 为 `Simulator` 实现方法 `_get_summary`
```python
    def _get_summary(self) -> Dict[str, Any]:
        return {k: v for r in self.recorders for k, v in r.get_summary().items()}
```
**EN:** Method `_get_summary` implements behavior on `Simulator`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `_get_summary` 为 `Simulator` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 116-122: Implement helper `_format_ids` / 实现辅助函数 `_format_ids`
```python
def _format_ids(requests: List[SimRequest], limit: int = 5) -> str:
    if not requests:
        return "-"
    ids = ",".join(r.request_id for r in requests[:limit])
    if len(requests) > limit:
        ids += f"...+{len(requests) - limit}"
    return ids
```
**EN:** Function `_format_ids` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_format_ids` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `SimulationResult`, `Simulator`, `_format_ids`
- **Module role / 模块角色**: Request scheduling simulation / 请求调度模拟
- **Implementation focus / 实现重点**: Models scheduler behavior so engineers can reason about queueing and routing decisions offline / 对调度器行为建模，使工程师可以离线分析排队与路由决策

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `sglang.srt.debug_utils.schedule_simulator.gpu_state`, `sglang.srt.debug_utils.schedule_simulator.metrics`, `sglang.srt.debug_utils.schedule_simulator.request`, `sglang.srt.debug_utils.schedule_simulator.routers.base`, `sglang.srt.debug_utils.schedule_simulator.schedulers.base`
