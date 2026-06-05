# parallel_executor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/executors/parallel_executor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for parallel executor, connecting stages, tensors, and runtime metadata. Key symbols include `ParallelExecutor`. / 该模块负责 parallel executor 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `ParallelExecutor`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

from typing import Any, Callable, List

import torch

from sglang.multimodal_gen.runtime.distributed import get_sp_group
from sglang.multimodal_gen.runtime.distributed.parallel_state import (
    get_cfg_group,
    get_classifier_free_guidance_rank,
    get_world_rank,
)
from sglang.multimodal_gen.runtime.pipelines_core import Req
from sglang.multimodal_gen.runtime.pipelines_core.executors.pipeline_executor import (
# ...
from sglang.multimodal_gen.runtime.utils.distributed import broadcast_pyobj
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 29-132: Class `ParallelExecutor` / 类 `ParallelExecutor`
```python
class ParallelExecutor(PipelineExecutor):
    """
    The correctness of the execution relies on the parallelism_type declared by stages

    """

    def collect_from_main(self, batches: list[Req]):

        # TODO: fix this condition
        if self.server_args.sp_degree != 1:
            sp_group = get_sp_group()
            batches = broadcast_pyobj(
                batches,
                sp_group.rank,
# ...
            batches,
            server_args,
            lambda stage, current: stage.run_grouped_requests(current, server_args),
        )
```
**EN:** This class models `ParallelExecutor` as a specialization of `PipelineExecutor`. The correctness of the execution relies on the parallelism_type declared by stages Important methods include `collect_from_main`, `_execute_stages`, `execute`, `execute_group`.
**CN:** 该类实现 `ParallelExecutor`，并继承/扩展 `PipelineExecutor`。 文档字符串指出：The correctness of the execution relies on the parallelism_type declared by stages 其中较重要的方法包括 `collect_from_main`, `_execute_stages`, `execute`, `execute_group`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Distributed execution / 分布式执行
- Video generation flow / 视频生成流程
- Symbol `ParallelExecutor` anchors the module API / 符号 `ParallelExecutor` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.pipelines_core`, `sglang.multimodal_gen.runtime.pipelines_core.executors.pipeline_executor`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.base`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.distributed`
- **External / 外部**: `torch`
- **Stdlib / 标准库**: `typing`
