# sync_executor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/executors/sync_executor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for sync executor, connecting stages, tensors, and runtime metadata. Key symbols include `SyncExecutor`. / 该模块负责 sync executor 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `SyncExecutor`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
"""
Synchronous pipeline executor implementation.
"""

from typing import Any, Callable, List

from sglang.multimodal_gen.runtime.pipelines_core.executors.pipeline_executor import (
    PipelineExecutor,
    SGLDiffusionProfiler,
)
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import OutputBatch, Req
from sglang.multimodal_gen.runtime.pipelines_core.stages import PipelineStage
from sglang.multimodal_gen.runtime.server_args import ServerArgs
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 19-83: Class `SyncExecutor` / 类 `SyncExecutor`
```python
class SyncExecutor(PipelineExecutor):
    """
    A simple synchronous executor that runs stages sequentially.
    """

    def _run_profile_all_stages(
        self,
        stages: List[PipelineStage],
        payload: Any,
        server_args: ServerArgs,
        run_stage: Callable[[PipelineStage, Any], Any],
    ) -> Any:
        """Execute all pipeline stages sequentially and step the profiler."""
        self.begin_component_residency_request(stages, payload, server_args)
# ...
            batches,
            server_args,
            lambda stage, current: stage.run_grouped_requests(current, server_args),
        )
```
**EN:** This class models `SyncExecutor` as a specialization of `PipelineExecutor`. A simple synchronous executor that runs stages sequentially. Important methods include `_run_profile_all_stages`, `run_profile_all_stages`, `execute`, `execute_group`.
**CN:** 该类实现 `SyncExecutor`，并继承/扩展 `PipelineExecutor`。 文档字符串指出：A simple synchronous executor that runs stages sequentially. 其中较重要的方法包括 `_run_profile_all_stages`, `run_profile_all_stages`, `execute`, `execute_group`。

## Key Concepts / 关键概念
- Pipeline orchestration / 流水线编排
- Video generation flow / 视频生成流程
- Symbol `SyncExecutor` anchors the module API / 符号 `SyncExecutor` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.pipelines_core.executors.pipeline_executor`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages`, `sglang.multimodal_gen.runtime.server_args`
- **Stdlib / 标准库**: `typing`
