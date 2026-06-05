# counter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/utils/counter.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides reusable utility helpers shared across vLLM components. / 提供 vLLM 各组件复用的通用工具函数。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import threading
```
**EN:** Sets up the module with standard-library support such as `threading`. It prepares the symbols later used by `Counter`, `AtomicCounter`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义。 这些准备工作为后续的 `Counter`, `AtomicCounter` 提供上下文。

### Counter (lines 6-18)
```python
class Counter:
    def __init__(self, start: int = 0) -> None:
        super().__init__()

        self.counter = start

    def __next__(self) -> int:
        i = self.counter
        self.counter += 1
        return i

    def reset(self) -> None:
        self.counter = 0
```
**EN:** Defines the `Counter` class used by this module. Key methods include `__init__`, `reset`.
**CN:** `Counter` 是该文件中的核心类，用于封装与 `Counter` 相关的状态和行为。 关键方法包括 `__init__`, `reset`。

### AtomicCounter (lines 21-45)
```python
class AtomicCounter:
    """An atomic, thread-safe counter"""

    def __init__(self, initial: int = 0) -> None:
        """Initialize a new atomic counter to given initial value"""
        super().__init__()

        self._value = initial
        self._lock = threading.Lock()

    @property
    def value(self) -> int:
        return self._value

    def inc(self, num: int = 1) -> int:
        """Atomically increment the counter by num and return the new value"""
        with self._lock:
            self._value += num
            return self._value

    def dec(self, num: int = 1) -> int:
        """Atomically decrement the counter by num and return the new value"""
        with self._lock:
            self._value -= num
            return self._value
```
**EN:** `AtomicCounter`: An atomic, thread-safe counter. Key methods include `__init__`, `value`, `inc`, `dec`.
**CN:** `AtomicCounter` 是该文件中的核心类，用于封装与 `AtomicCounter` 相关的状态和行为。 关键方法包括 `__init__`, `value`, `inc`, `dec`。

## Key Concepts / 关键概念
- **`Counter`**: Core class that organizes module behavior. / **`Counter`**：组织模块行为的核心类。
- **`AtomicCounter`**: Core class that organizes module behavior. / **`AtomicCounter`**：组织模块行为的核心类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: threading
- **Third-party / 第三方**: None / 无
- **Internal vLLM / vLLM 内部依赖**: None / 无
