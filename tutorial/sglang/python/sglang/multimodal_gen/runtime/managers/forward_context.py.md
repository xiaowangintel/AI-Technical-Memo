# forward_context.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/managers/forward_context.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contains runtime support code for forward context in the multimodal generation stack. Key symbols include `ForwardContext`, `get_forward_context`, `set_forward_context`. / 该模块包含多模态生成体系中与 forward context 相关的运行时支持代码。 关键符号包括 `ForwardContext`, `get_forward_context`, `set_forward_context`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
# Adapted from vllm: https://github.com/vllm-project/vllm/blob/v0.7.3/vllm/forward_context.py
import time
from collections import defaultdict
from contextlib import contextmanager
from dataclasses import dataclass
from typing import TYPE_CHECKING, Optional, Type

import torch

from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

# ...
forward_start_time: float = 0
# batchsize_logging_interval: float = envs.SGLANG_DIFFUSION_LOG_BATCHSIZE_INTERVAL
batchsize_logging_interval: float = 1000
batchsize_forward_time: defaultdict = defaultdict(list)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 31-49: Class `ForwardContext` / 类 `ForwardContext`
```python
@dataclass
class ForwardContext:
    current_timestep: int
    # TODO(will): check this arg
    # copy from vllm_config.compilation_config.static_forward_context
    # attn_layers: Dict[str, Any]
    # TODO: extend to support per-layer dynamic forward context
    attn_metadata: "AttentionMetadata"  # set dynamically for each forward pass
    forward_batch: Optional["Req"] = None
    attention_backend_cls: Optional[Type] = None

    def set_attn_backend_cls(self, attention_backend_cls: Type):
        if self.attention_backend_cls:
            if self.attention_backend_cls != attention_backend_cls:
                raise RuntimeError(
                    f"Different types of attention backend in a same context detected, previous: {self.attention_backend_cls}, new: {attention_backend_cls}"
                )
        else:
            self.attention_backend_cls = attention_backend_cls
```
**EN:** This class models `ForwardContext`. Important methods include `set_attn_backend_cls`.
**CN:** 该类实现 `ForwardContext`。 其中较重要的方法包括 `set_attn_backend_cls`。

### Lines 50-52: Top-level configuration / 顶层配置
```python


_forward_context: Optional["ForwardContext"] = None
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 55-61: Function `get_forward_context` / 函数 `get_forward_context`
```python
def get_forward_context() -> "ForwardContext":
    """Get the current forward context."""
    assert _forward_context is not None, (
        "Forward context is not set. "
        "Please use `set_forward_context` to set the forward context."
    )
    return _forward_context
```
**EN:** This function drives `get_forward_context`. Get the current forward context.
**CN:** 这个函数负责 `get_forward_context`。 文档字符串说明：Get the current forward context.

### Lines 65-120: Function `set_forward_context` / 函数 `set_forward_context`
```python
@contextmanager
def set_forward_context(
    current_timestep, attn_metadata, forward_batch: Optional["Req"] = None
):
    """A context manager that stores the current forward context,
    can be attention metadata, etc.
    Here we can inject common logic for every model forward pass.
    """
    global forward_start_time
    need_to_track_batchsize = track_batchsize and attn_metadata is not None
    if need_to_track_batchsize:
        forward_start_time = time.perf_counter()
    global _forward_context
    prev_context = _forward_context
# ...
                        ),
                        forward_stats,
                    )
        _forward_context = prev_context
```
**EN:** This function drives `set_forward_context` with inputs such as `current_timestep`, `attn_metadata`, `forward_batch`. A context manager that stores the current forward context,
**CN:** 这个函数负责 `set_forward_context`，主要处理 `current_timestep`, `attn_metadata`, `forward_batch` 等输入。 文档字符串说明：A context manager that stores the current forward context,

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Pipeline orchestration / 流水线编排
- Video generation flow / 视频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.layers.attention`, `sglang.multimodal_gen.runtime.pipelines_core`
- **External / 外部**: `torch`
- **Stdlib / 标准库**: `time`, `collections`, `contextlib`, `dataclasses`, `typing`
