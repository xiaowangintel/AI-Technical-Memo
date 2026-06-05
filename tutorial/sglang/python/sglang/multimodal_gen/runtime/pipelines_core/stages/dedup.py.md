# dedup.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/dedup.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for dedup, connecting stages, tensors, and runtime metadata. Key symbols include `StageDedupMixin`. / 该模块负责 dedup 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `StageDedupMixin`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Imports and module setup / 导入与模块初始化
```python
"""Stage-local grouped-request dedup helpers."""

# SPDX-License-Identifier: Apache-2.0
from __future__ import annotations

from copy import deepcopy
from typing import TYPE_CHECKING, Any, ClassVar

import torch

if TYPE_CHECKING:
    from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import Req
    from sglang.multimodal_gen.runtime.server_args import ServerArgs
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 16-186: Class `StageDedupMixin` / 类 `StageDedupMixin`
```python
class StageDedupMixin:
    """Mixin for stage-local grouped-request deduplication.

    The mixin handles only stage-local reuse. It is not a global cache and does
    not decide which requests are equivalent for a stage. A stage opts into the
    common full-stage path by declaring the ``Req`` fields it writes through the
    ``deduplicated_*`` class attributes and by overriding
    ``build_dedup_fingerprint``.

    Stages that can reuse only part of their work should override
    ``run_grouped_requests`` directly and may still use
    ``_group_requests_by_fingerprint`` for stable grouping.
    """

# ...
        for index, batch in enumerate(batches):
            fingerprint = fingerprint_fn(batch)
            groups.setdefault(fingerprint, []).append((index, batch))
        return list(groups.items())
```
**EN:** This class models `StageDedupMixin`. Mixin for stage-local grouped-request deduplication. Important methods include `run_grouped_requests`, `has_deduplicated_output_fields`, `build_dedup_fingerprint`, `run_deduplicated_group`.
**CN:** 该类实现 `StageDedupMixin`。 文档字符串指出：Mixin for stage-local grouped-request deduplication. 其中较重要的方法包括 `run_grouped_requests`, `has_deduplicated_output_fields`, `build_dedup_fingerprint`, `run_deduplicated_group`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Caching strategy / 缓存策略
- Symbol `StageDedupMixin` anchors the module API / 符号 `StageDedupMixin` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.server_args`
- **External / 外部**: `__future__`, `torch`
- **Stdlib / 标准库**: `copy`, `typing`
