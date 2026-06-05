# schedule_batch.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/schedule_batch.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for schedule batch, connecting stages, tensors, and runtime metadata. Key symbols include `BatchMetricsWindow`, `Req`, `OutputBatch`. / 该模块负责 schedule batch 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `BatchMetricsWindow`, `Req`, `OutputBatch`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
# Inspired by SGLang: https://github.com/sgl-project/sglang/blob/main/python/sglang/srt/model_executor/forward_batch_info.py
"""
Data structures for functional pipeline processing.

This module defines the dataclasses used to pass state between pipeline components
in a functional manner, reducing the need for explicit parameter passing.
"""

from __future__ import annotations

import logging
# ...

logger = init_logger(__name__)

SAMPLING_PARAMS_FIELDS = {f.name for f in fields(SamplingParams)}
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 43-57: Class `BatchMetricsWindow` / 类 `BatchMetricsWindow`
```python
@dataclass
class BatchMetricsWindow:
    """Counters accumulated between dynamic batching metric logs.

    `total_capacity` uses each dispatch's effective admission cap, so
    utilization reflects model/config limits instead of only the user max.
    """

    dispatches: int = 0
    total_requests: int = 0
    total_capacity: int = 0
    merged_dispatches: int = 0
    full_dispatches: int = 0
    wait_times_ms: list[float] = field(default_factory=list)
    reject_reasons: Counter[str] = field(default_factory=Counter)
```
**EN:** This class models `BatchMetricsWindow`. Counters accumulated between dynamic batching metric logs.
**CN:** 该类实现 `BatchMetricsWindow`。 文档字符串指出：Counters accumulated between dynamic batching metric logs.

### Lines 60-377: Class `Req` / 类 `Req`
```python
@dataclass(init=False)
class Req:
    """
    Complete state passed through the pipeline execution.

    This dataclass contains all information needed during the diffusion pipeline
    execution, allowing methods to update specific components without needing
    to manage numerous individual parameters.

    [IMPORTANT] Fields that overlap with SamplingParams are automatically delegated to the
    sampling_params member via __getattr__ and __setattr__.
    """

    sampling_params: SamplingParams | None = None
# ...
                 save_output: {self.save_output}
            output_file_path: {self.output_file_path()}
        """  # type: ignore[attr-defined]
        logger.info(debug_str)
```
**EN:** This class models `Req`. Complete state passed through the pipeline execution. Important methods include `__init__`, `__getattr__`, `__setattr__`, `batch_size`.
**CN:** 该类实现 `Req`。 文档字符串指出：Complete state passed through the pipeline execution. 其中较重要的方法包括 `__init__`, `__getattr__`, `__setattr__`, `batch_size`。

### Lines 380-402: Class `OutputBatch` / 类 `OutputBatch`
```python
@dataclass
class OutputBatch:
    """
    Final output (after pipeline completion)
    """

    output: Any | None = None
    audio: torch.Tensor | None = None
    audio_sample_rate: int | None = None
    trajectory_timesteps: torch.Tensor | None = None
    trajectory_latents: torch.Tensor | None = None
    rollout_trajectory_data: RolloutTrajectoryData | None = None
    trajectory_decoded: list[torch.Tensor] | None = None
    error: str | None = None
# ...

    # For ComfyUI integration: noise prediction from denoising stage
    noise_pred: torch.Tensor | None = None
    peak_memory_mb: float = 0.0
```
**EN:** This class models `OutputBatch`. Final output (after pipeline completion)
**CN:** 该类实现 `OutputBatch`。 文档字符串指出：Final output (after pipeline completion)

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Caching strategy / 缓存策略

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.runtime.post_training.rl_dataclasses`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.utils.perf_logger`, `sglang.multimodal_gen.utils`, `sglang.srt.observability.trace`
- **External / 外部**: `__future__`, `pprint`, `PIL.Image`, `torch`
- **Stdlib / 标准库**: `logging`, `os`, `collections`, `copy`, `dataclasses`, `typing`
