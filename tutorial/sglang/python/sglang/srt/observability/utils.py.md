# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/observability/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the metrics, tracing, and monitoring part of the SRT runtime and implements logic centered on `utils`. The module docstring frames it as: "Utilities for Prometheus Metrics." / 该模块属于 SRT 运行时的指标、追踪与监控部分，主要实现围绕 `utils` 的逻辑。 它对外提供的主要入口包括 `two_sides_exponential_buckets`, `generate_buckets`, `exponential_buckets`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Module imports, constants, and setup
```python
# Copyright 2023-2025 SGLang Team
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
"""Utilities for Prometheus Metrics."""

import math
from typing import List


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; records metrics or tracing signals.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；记录指标或追踪信号。

### Lines 20-31: Function two_sides_exponential_buckets
```python
def two_sides_exponential_buckets(
    middle: float, base: float, count: int
) -> List[float]:
    buckets = []
    half_count = math.ceil(count / 2)
    distance = 1
    buckets.append(middle)
    for i in range(half_count):
        distance *= base
        buckets.append(middle + distance)
        buckets.append(max(0, middle - distance))
    return sorted(set(buckets))
```
**EN:** This callable implements `two_sides_exponential_buckets`. It takes `middle`, `base`, `count` and mainly implements two sides exponential buckets.
**CN:** 这一可调用对象实现了 `two_sides_exponential_buckets`。它接收 `middle`, `base`, `count`，主要用于实现 two sides exponential buckets 相关逻辑。

### Lines 34-49: Function generate_buckets
```python
def generate_buckets(
    buckets_rule: List[str], default_buckets: List[float]
) -> List[float]:
    if not buckets_rule:
        buckets_rule = ["default"]

    assert len(buckets_rule) > 0
    rule = buckets_rule[0]
    if rule == "tse":
        middle, base, count = buckets_rule[1:]
        assert float(base) > 1.0, "Base must be greater than 1.0"
        return two_sides_exponential_buckets(float(middle), float(base), int(count))
    if rule == "default":
        return sorted(set(default_buckets))
    assert rule == "custom"
    return sorted(set([float(x) for x in buckets_rule[1:]]))
```
**EN:** This callable implements `generate_buckets`. It takes `buckets_rule`, `default_buckets` and mainly implements generate buckets. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `generate_buckets`。它接收 `buckets_rule`, `default_buckets`，主要用于实现 generate buckets 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 52-56: Function exponential_buckets
```python
def exponential_buckets(start: float, width: float, length: int) -> List[float]:
    buckets = []
    for i in range(length):
        buckets.append(start * (width**i))
    return buckets
```
**EN:** This callable implements `exponential_buckets`. It takes `start`, `width`, `length` and mainly implements exponential buckets.
**CN:** 这一可调用对象实现了 `exponential_buckets`。它接收 `start`, `width`, `length`，主要用于实现 exponential buckets 相关逻辑。

## Key Concepts / 关键概念
- `two_sides_exponential_buckets`: implements two sides exponential buckets / 实现 two sides exponential buckets 相关逻辑
- `generate_buckets`: implements generate buckets / 实现 generate buckets 相关逻辑
- `exponential_buckets`: implements exponential buckets / 实现 exponential buckets 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `math`, `typing`
