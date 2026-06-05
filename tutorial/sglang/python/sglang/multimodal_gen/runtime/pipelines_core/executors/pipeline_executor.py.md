# pipeline_executor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/executors/pipeline_executor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for pipeline executor, connecting stages, tensors, and runtime metadata. Key symbols include `Timer`, `PipelineExecutor`. / 该模块负责 pipeline executor 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `Timer`, `PipelineExecutor`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
"""
Base class for all pipeline executors.
"""

import contextlib
from abc import ABC, abstractmethod
from typing import TYPE_CHECKING, List

from sglang.multimodal_gen.runtime.distributed import get_world_rank
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import OutputBatch, Req
from sglang.multimodal_gen.runtime.server_args import ServerArgs
# ...
    # Only for type checkers; avoids runtime circular import
    from sglang.multimodal_gen.runtime.pipelines_core.stages.base import PipelineStage

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 26-35: Class `Timer` / 类 `Timer`
```python
class Timer(StageProfiler):
    """
    A wrapper around StageProfiler to maintain backward compatibility.
    It forces simple logging behavior (log start/end) regardless of env vars.
    """

    def __init__(self, name="Stage"):
        super().__init__(
            stage_name=name, logger=logger, metrics=None, log_stage_start_end=True
        )
```
**EN:** This class models `Timer` as a specialization of `StageProfiler`. A wrapper around StageProfiler to maintain backward compatibility. Important methods include `__init__`.
**CN:** 该类实现 `Timer`，并继承/扩展 `StageProfiler`。 文档字符串指出：A wrapper around StageProfiler to maintain backward compatibility. 其中较重要的方法包括 `__init__`。

### Lines 38-159: Class `PipelineExecutor` / 类 `PipelineExecutor`
```python
class PipelineExecutor(ABC):
    """
    Abstract base class for all pipeline executors.

    Executors orchestrate the execution of pipeline, with managing the parallel and communications required by stages

    """

    def __init__(self, server_args):
        self.server_args = server_args
        self.component_residency_manager = None

    def begin_component_residency_request(
        self,
# ...
        try:
            yield
        finally:
            profiler.stop(dump_rank=dump_rank)
```
**EN:** This class models `PipelineExecutor` as a specialization of `ABC`. Abstract base class for all pipeline executors. Important methods include `__init__`, `begin_component_residency_request`, `before_stage`, `after_stage`.
**CN:** 该类实现 `PipelineExecutor`，并继承/扩展 `ABC`。 文档字符串指出：Abstract base class for all pipeline executors. 其中较重要的方法包括 `__init__`, `begin_component_residency_request`, `before_stage`, `after_stage`。

## Key Concepts / 关键概念
- Pipeline orchestration / 流水线编排
- Distributed execution / 分布式执行
- Video generation flow / 视频生成流程
- Symbol `Timer` anchors the module API / 符号 `Timer` 构成该模块的核心 API
- Symbol `PipelineExecutor` anchors the module API / 符号 `PipelineExecutor` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.utils.perf_logger`, `sglang.multimodal_gen.runtime.utils.profiler`, `sglang.multimodal_gen.runtime.pipelines_core.stages.base`
- **Stdlib / 标准库**: `contextlib`, `abc`, `typing`
