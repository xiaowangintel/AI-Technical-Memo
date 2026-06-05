# base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/base.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for base, connecting stages, tensors, and runtime metadata. Key symbols include `StageParallelismType`, `StageVerificationError`, `PipelineStage`. / 该模块负责 base 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `StageParallelismType`, `StageVerificationError`, `PipelineStage`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
"""
Base classes for pipeline stages.

This module defines the abstract base classes for pipeline stages that can be
composed to create complete diffusion pipelines.
"""

from abc import ABC, abstractmethod
from collections.abc import Iterator
from contextlib import contextmanager
from dataclasses import replace
# ...
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.runtime.utils.perf_logger import StageProfiler

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 36-42: Class `StageParallelismType` / 类 `StageParallelismType`
```python
class StageParallelismType(Enum):
    # execute on all gpus
    REPLICATED = auto()
    # executed on main rank only
    MAIN_RANK_ONLY = auto()
    # this stage requires a cfg-parallel
    CFG_PARALLEL = auto()
```
**EN:** This class models `StageParallelismType` as a specialization of `Enum`.
**CN:** 该类实现 `StageParallelismType`，并继承/扩展 `Enum`。

### Lines 45-48: Class `StageVerificationError` / 类 `StageVerificationError`
```python
class StageVerificationError(Exception):
    """Exception raised when stage verification fails."""

    pass
```
**EN:** This class models `StageVerificationError` as a specialization of `Exception`. Exception raised when stage verification fails.
**CN:** 该类实现 `StageVerificationError`，并继承/扩展 `Exception`。 文档字符串指出：Exception raised when stage verification fails.

### Lines 51-326: Class `PipelineStage` / 类 `PipelineStage`
```python
class PipelineStage(StageDedupMixin, ABC):
    """
    Abstract base class for all pipeline stages.

    A pipeline stage represents a discrete step in the diffusion process that can be
    composed with other stages to create a complete pipeline. Each stage is responsible
    for a specific part of the process, such as prompt encoding, latent preparation, etc.
    """

    def __init__(self):
        self.server_args = get_global_server_args()
        self._component_residency_manager = None

    def log_info(self, msg, *args):
# ...
        batch: Req,
        server_args: ServerArgs,
    ) -> Req:
        raise NotImplementedError
```
**EN:** This class models `PipelineStage` as a specialization of `StageDedupMixin`, `ABC`. Abstract base class for all pipeline stages. Important methods include `__init__`, `log_info`, `log_warning`, `log_error`.
**CN:** 该类实现 `PipelineStage`，并继承/扩展 `StageDedupMixin`, `ABC`。 文档字符串指出：Abstract base class for all pipeline stages. 其中较重要的方法包括 `__init__`, `log_info`, `log_warning`, `log_error`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Request disaggregation / 请求解耦
- Image generation flow / 图像生成流程
- Video generation flow / 视频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.disaggregation.roles`, `sglang.multimodal_gen.runtime.managers.memory_managers.component_manager`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.dedup`, `sglang.multimodal_gen.runtime.pipelines_core.stages.validators`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `torch`
- **Stdlib / 标准库**: `abc`, `collections.abc`, `contextlib`, `dataclasses`, `enum`
