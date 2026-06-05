# gpu_state.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/schedule_simulator/gpu_state.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on request scheduling simulation. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于请求调度模拟。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Import dependencies and shared types / 导入依赖与共享类型
```python
from dataclasses import dataclass, field
from typing import List, Optional

from sglang.srt.debug_utils.schedule_simulator.request import SimRequest
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 8-8: Define class `StepRecord` and class context / 定义类 `StepRecord`及类上下文
```python
class StepRecord:
```
**EN:** This section introduces `StepRecord`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `StepRecord`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 9-15: Declare fields for `StepRecord` such as `step`, `gpu_id`, `running_count`, `pending_count`, `total_seq_len` / 为 `StepRecord` 声明字段，例如 `step`, `gpu_id`, `running_count`, `pending_count`, `total_seq_len`
```python
    step: int
    gpu_id: int
    running_count: int
    pending_count: int
    total_seq_len: int
    running_req_ids: List[str] = field(default_factory=list)
    pending_req_ids: List[str] = field(default_factory=list)
```
**EN:** These lines declare the state carried by `StepRecord`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `StepRecord` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 19-19: Define class `GPUState` and class context / 定义类 `GPUState`及类上下文
```python
class GPUState:
```
**EN:** This section introduces `GPUState`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `GPUState`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 20-23: Declare fields for `GPUState` such as `gpu_id`, `max_total_tokens`, `pending_requests`, `running_requests` / 为 `GPUState` 声明字段，例如 `gpu_id`, `max_total_tokens`, `pending_requests`, `running_requests`
```python
    gpu_id: int
    max_total_tokens: int
    pending_requests: List[SimRequest] = field(default_factory=list)
    running_requests: List[SimRequest] = field(default_factory=list)
```
**EN:** These lines declare the state carried by `GPUState`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `GPUState` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 25-26: Implement method `batch_size` for `GPUState` / 为 `GPUState` 实现方法 `batch_size`
```python
    def batch_size(self) -> int:
        return len(self.running_requests)
```
**EN:** Method `batch_size` implements behavior on `GPUState`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `batch_size` 为 `GPUState` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 28-29: Implement method `total_attention_compute` for `GPUState` / 为 `GPUState` 实现方法 `total_attention_compute`
```python
    def total_attention_compute(self) -> int:
        return sum(req.seq_len() for req in self.running_requests)
```
**EN:** Method `total_attention_compute` implements behavior on `GPUState`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `total_attention_compute` 为 `GPUState` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 31-39: Implement method `total_seq_len` for `GPUState` / 为 `GPUState` 实现方法 `total_seq_len`
```python
    def total_seq_len(self, extra_reqs: Optional[List[SimRequest]] = None) -> int:
        seen_groups = set()
        total = 0
        for req in self.running_requests + (extra_reqs or []):
            is_shared = req.group_id is not None and req.group_id in seen_groups
            total += req.seq_len() - (req.prefix_len if is_shared else 0)
            if req.group_id is not None:
                seen_groups.add(req.group_id)
        return total
```
**EN:** Method `total_seq_len` implements behavior on `GPUState`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `total_seq_len` 为 `GPUState` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 41-42: Implement method `is_valid` for `GPUState` / 为 `GPUState` 实现方法 `is_valid`
```python
    def is_valid(self) -> bool:
        return self.total_seq_len() <= self.max_total_tokens
```
**EN:** Method `is_valid` implements behavior on `GPUState`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `is_valid` 为 `GPUState` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 44-47: Implement method `start_request` for `GPUState` / 为 `GPUState` 实现方法 `start_request`
```python
    def start_request(self, req: SimRequest) -> None:
        assert req in self.pending_requests
        self.pending_requests.remove(req)
        self.running_requests.append(req)
```
**EN:** Method `start_request` implements behavior on `GPUState`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `start_request` 为 `GPUState` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 49-52: Implement method `evict_request` for `GPUState` / 为 `GPUState` 实现方法 `evict_request`
```python
    def evict_request(self, req: SimRequest) -> None:
        assert req in self.running_requests
        self.running_requests.remove(req)
        self.pending_requests.insert(0, req)
```
**EN:** Method `evict_request` implements behavior on `GPUState`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `evict_request` 为 `GPUState` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 54-59: Implement method `execute_step` for `GPUState` / 为 `GPUState` 实现方法 `execute_step`
```python
    def execute_step(self) -> None:
        for req in self.running_requests:
            req.decoded_tokens += 1
        self.running_requests = [
            r for r in self.running_requests if not r.is_finished()
        ]
```
**EN:** Method `execute_step` implements behavior on `GPUState`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `execute_step` 为 `GPUState` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 61-70: Implement method `get_step_record` for `GPUState` / 为 `GPUState` 实现方法 `get_step_record`
```python
    def get_step_record(self, step: int) -> StepRecord:
        return StepRecord(
            step=step,
            gpu_id=self.gpu_id,
            running_count=len(self.running_requests),
            pending_count=len(self.pending_requests),
            total_seq_len=self.total_seq_len(),
            running_req_ids=[r.request_id for r in self.running_requests],
            pending_req_ids=[r.request_id for r in self.pending_requests],
        )
```
**EN:** Method `get_step_record` implements behavior on `GPUState`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `get_step_record` 为 `GPUState` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `StepRecord`, `GPUState`
- **Module role / 模块角色**: Request scheduling simulation / 请求调度模拟
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `sglang.srt.debug_utils.schedule_simulator.request`
