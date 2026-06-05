# label_transform.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/observability/label_transform.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the metrics, tracing, and monitoring part of the SRT runtime and implements logic centered on `label_transform`. It exposes primary entry points such as `transform_priority`. / 该模块属于 SRT 运行时的指标、追踪与监控部分，主要实现围绕 `label_transform` 的逻辑。 它对外提供的主要入口包括 `transform_priority`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Module imports, constants, and setup
```python
from typing import Optional

_PRIORITY_MIN = 0
_PRIORITY_MAX = 31
_LOW_PRIORITY_VALUE = "LOW"
_HIGH_PRIORITY_VALUE = "HIGH"

UNKNOWN_PRIORITY_VALUE = "UNKNOWN"


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 11-27: Function transform_priority
```python
def transform_priority(priority: Optional[int]) -> str:
    """Transform the priority to a string for metrics reporting.
    Limit the range to prevent high cardinality issues.

    Args:
        priority: The priority to transform.
    Returns:
        The transformed priority.
    """
    if priority is None:
        return UNKNOWN_PRIORITY_VALUE
    elif priority < _PRIORITY_MIN:
        return _LOW_PRIORITY_VALUE
    elif priority >= _PRIORITY_MAX:
        return _HIGH_PRIORITY_VALUE
    else:
        return str(priority)
```
**EN:** This callable implements `transform_priority`. It takes `priority` and mainly implements transform priority. The docstring states: "Transform the priority to a string for metrics reporting." In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `transform_priority`。它接收 `priority`，主要用于实现 transform priority 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

## Key Concepts / 关键概念
- `_PRIORITY_MIN`: module constant or capability flag / 模块常量或能力标记
- `_PRIORITY_MAX`: module constant or capability flag / 模块常量或能力标记
- `_LOW_PRIORITY_VALUE`: module constant or capability flag / 模块常量或能力标记
- `_HIGH_PRIORITY_VALUE`: module constant or capability flag / 模块常量或能力标记
- `UNKNOWN_PRIORITY_VALUE`: module constant or capability flag / 模块常量或能力标记
- `transform_priority`: implements transform priority / 实现 transform priority 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
