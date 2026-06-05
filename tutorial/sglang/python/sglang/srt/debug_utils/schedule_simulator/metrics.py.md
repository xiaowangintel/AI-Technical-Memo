# metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/schedule_simulator/metrics.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on request scheduling simulation. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于请求调度模拟。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Import dependencies and shared types / 导入依赖与共享类型
```python
from abc import ABC, abstractmethod
from typing import Any, Callable, Dict, List

from sglang.srt.debug_utils.schedule_simulator.gpu_state import GPUState
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 7-8: Define class `MetricRecorder` and class context / 定义类 `MetricRecorder`及类上下文
```python
class MetricRecorder(ABC):
    @abstractmethod
```
**EN:** This section introduces `MetricRecorder`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `MetricRecorder`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 9-9: Implement method `on_step_end` for `MetricRecorder` / 为 `MetricRecorder` 实现方法 `on_step_end`
```python
    def on_step_end(self, step: int, gpu_states: List[GPUState]) -> None: ...
```
**EN:** Method `on_step_end` implements behavior on `MetricRecorder`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `on_step_end` 为 `MetricRecorder` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 12-12: Implement method `get_summary` for `MetricRecorder` / 为 `MetricRecorder` 实现方法 `get_summary`
```python
    def get_summary(self) -> Dict[str, Any]: ...
```
**EN:** Method `get_summary` implements behavior on `MetricRecorder`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `get_summary` 为 `MetricRecorder` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 15-15: Define class `BalancednessRecorder` and class context / 定义类 `BalancednessRecorder`及类上下文
```python
class BalancednessRecorder(MetricRecorder):
```
**EN:** This section introduces `BalancednessRecorder`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `BalancednessRecorder`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 16-19: Implement method `__init__` for `BalancednessRecorder` / 为 `BalancednessRecorder` 实现方法 `__init__`
```python
    def __init__(self, name: str, value_fn: Callable[[GPUState], float]):
        self._name = name
        self._value_fn = value_fn
        self._history: List[float] = []
```
**EN:** Method `__init__` implements behavior on `BalancednessRecorder`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__init__` 为 `BalancednessRecorder` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 21-26: Implement method `on_step_end` for `BalancednessRecorder` / 为 `BalancednessRecorder` 实现方法 `on_step_end`
```python
    def on_step_end(self, step: int, gpu_states: List[GPUState]) -> None:
        values = [self._value_fn(gpu) for gpu in gpu_states]
        max_val = max(values) if values else 0
        mean_val = sum(values) / len(values) if values else 0
        balancedness = mean_val / max_val if max_val > 0 else 1.0
        self._history.append(balancedness)
```
**EN:** Method `on_step_end` implements behavior on `BalancednessRecorder`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `on_step_end` 为 `BalancednessRecorder` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 28-35: Implement method `get_summary` for `BalancednessRecorder` / 为 `BalancednessRecorder` 实现方法 `get_summary`
```python
    def get_summary(self) -> Dict[str, Any]:
        if not self._history:
            return {f"{self._name}_mean": 0.0}
        return {
            f"{self._name}_mean": sum(self._history) / len(self._history),
            f"{self._name}_min": min(self._history),
            f"{self._name}_max": max(self._history),
        }
```
**EN:** Method `get_summary` implements behavior on `BalancednessRecorder`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `get_summary` 为 `BalancednessRecorder` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 38-39: Implement function `BatchSizeBalancednessRecorder` / 实现函数 `BatchSizeBalancednessRecorder`
```python
def BatchSizeBalancednessRecorder() -> BalancednessRecorder:
    return BalancednessRecorder("batch_size_balancedness", lambda gpu: gpu.batch_size())
```
**EN:** Function `BatchSizeBalancednessRecorder` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `BatchSizeBalancednessRecorder` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 42-45: Implement function `AttentionComputeBalancednessRecorder` / 实现函数 `AttentionComputeBalancednessRecorder`
```python
def AttentionComputeBalancednessRecorder() -> BalancednessRecorder:
    return BalancednessRecorder(
        "attention_compute_balancedness", lambda gpu: gpu.total_attention_compute()
    )
```
**EN:** Function `AttentionComputeBalancednessRecorder` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `AttentionComputeBalancednessRecorder` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 48-48: Define class `AvgBatchSizeRecorder` and class context / 定义类 `AvgBatchSizeRecorder`及类上下文
```python
class AvgBatchSizeRecorder(MetricRecorder):
```
**EN:** This section introduces `AvgBatchSizeRecorder`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `AvgBatchSizeRecorder`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 49-51: Implement method `__init__` for `AvgBatchSizeRecorder` / 为 `AvgBatchSizeRecorder` 实现方法 `__init__`
```python
    def __init__(self):
        self._total_running = 0
        self._num_records = 0
```
**EN:** Method `__init__` implements behavior on `AvgBatchSizeRecorder`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__init__` 为 `AvgBatchSizeRecorder` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 53-56: Implement method `on_step_end` for `AvgBatchSizeRecorder` / 为 `AvgBatchSizeRecorder` 实现方法 `on_step_end`
```python
    def on_step_end(self, step: int, gpu_states: List[GPUState]) -> None:
        for gpu in gpu_states:
            self._total_running += gpu.batch_size()
            self._num_records += 1
```
**EN:** Method `on_step_end` implements behavior on `AvgBatchSizeRecorder`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `on_step_end` 为 `AvgBatchSizeRecorder` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 58-60: Implement method `get_summary` for `AvgBatchSizeRecorder` / 为 `AvgBatchSizeRecorder` 实现方法 `get_summary`
```python
    def get_summary(self) -> Dict[str, Any]:
        avg = self._total_running / self._num_records if self._num_records else 0.0
        return {"avg_batch_size": avg}
```
**EN:** Method `get_summary` implements behavior on `AvgBatchSizeRecorder`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `get_summary` 为 `AvgBatchSizeRecorder` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `MetricRecorder`, `BalancednessRecorder`, `AvgBatchSizeRecorder`, `BatchSizeBalancednessRecorder`, `AttentionComputeBalancednessRecorder`
- **Module role / 模块角色**: Request scheduling simulation / 请求调度模拟
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `abc`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `sglang.srt.debug_utils.schedule_simulator.gpu_state`
