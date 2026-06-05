# lora_pipeline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/lora_pipeline.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for lora pipeline, connecting stages, tensors, and runtime metadata. Key symbols include `LoRAPipeline`. / 该模块负责 lora pipeline 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `LoRAPipeline`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-35: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
import os
from collections import defaultdict
from collections.abc import Hashable
from contextlib import contextmanager
from typing import Any

import torch
import torch.distributed as dist
from safetensors.torch import load_file
from torch.distributed.tensor import DTensor

# ...
# to avoid deadlocks when forking
os.environ["TOKENIZERS_PARALLELISM"] = "false"

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 38-1060: Class `LoRAPipeline` / 类 `LoRAPipeline`
```python
class LoRAPipeline(ComposedPipelineBase):
    """
    Pipeline that supports injecting LoRA adapters into the diffusion transformer.
    """

    # Type annotations for instance attributes (initialized in __init__)
    # [lora_nickname][target_LoRA_weight_name_in_SGLang_dit] = weight
    # e.g., [jinx][transformer_blocks.0.attn.to_v.lora_A]
    lora_adapters: dict[str, dict[str, torch.Tensor]]
    loaded_adapter_paths: dict[str, str]  # nickname -> lora_path
    # Track current adapter per module: {"transformer": "high_lora", "transformer_2": "low_lora"}
    cur_adapter_name: dict[str, str]
    cur_adapter_path: dict[str, str]
    cur_adapter_strength: dict[str, float]  # Track current strength per module
# ...
        return {
            "loaded_adapters": loaded_adapters,
            "active": active,
        }
```
**EN:** This class models `LoRAPipeline` as a specialization of `ComposedPipelineBase`. Pipeline that supports injecting LoRA adapters into the diffusion transformer. Important methods include `__init__`, `is_target_layer`, `_get_target_lora_layers`, `_temporarily_disable_offload`.
**CN:** 该类实现 `LoRAPipeline`，并继承/扩展 `ComposedPipelineBase`。 文档字符串指出：Pipeline that supports injecting LoRA adapters into the diffusion transformer. 其中较重要的方法包括 `__init__`, `is_target_layer`, `_get_target_lora_layers`, `_temporarily_disable_offload`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Model/component loading / 模型/组件加载
- Pipeline orchestration / 流水线编排
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.layers.lora.linear`, `sglang.multimodal_gen.runtime.loader.utils`, `sglang.multimodal_gen.runtime.pipelines_core.composed_pipeline_base`, `sglang.multimodal_gen.runtime.pipelines_core.lora_format_adapter`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.hf_diffusers_utils`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `torch`, `torch.distributed`, `safetensors.torch`, `torch.distributed.tensor`
- **Stdlib / 标准库**: `os`, `collections`, `collections.abc`, `contextlib`, `typing`
