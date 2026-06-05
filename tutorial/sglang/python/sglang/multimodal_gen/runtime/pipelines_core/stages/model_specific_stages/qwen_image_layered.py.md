# qwen_image_layered.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/model_specific_stages/qwen_image_layered.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for qwen image layered, connecting stages, tensors, and runtime metadata. Key symbols include `_seq_lens_from_optional_mask`, `calculate_dimensions`, `retrieve_latents`. / 该模块负责 qwen image layered 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `_seq_lens_from_optional_mask`, `calculate_dimensions`, `retrieve_latents`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Imports and module setup / 导入与模块初始化
```python
import inspect
import math
from typing import List, Optional, Union

import numpy as np
import torch
from diffusers.image_processor import VaeImageProcessor
from diffusers.utils.torch_utils import randn_tensor

from sglang.multimodal_gen.runtime.distributed import get_local_torch_device
from sglang.multimodal_gen.runtime.managers.forward_context import set_forward_context
from sglang.multimodal_gen.runtime.managers.memory_managers.component_manager import (
    ComponentUse,
)
# ...
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 24-30: Function `_seq_lens_from_optional_mask` / 函数 `_seq_lens_from_optional_mask`
```python
def _seq_lens_from_optional_mask(
    prompt_embeds: torch.Tensor, prompt_embeds_mask: torch.Tensor | None
) -> list[int]:
    """Return real text lengths, treating a missing mask as all tokens valid."""
    if prompt_embeds_mask is None:
        return [int(prompt_embeds.shape[1])] * int(prompt_embeds.shape[0])
    return [int(x) for x in prompt_embeds_mask.sum(dim=1).tolist()]
```
**EN:** This function drives `_seq_lens_from_optional_mask` with inputs such as `prompt_embeds`, `prompt_embeds_mask`. Return real text lengths, treating a missing mask as all tokens valid.
**CN:** 这个函数负责 `_seq_lens_from_optional_mask`，主要处理 `prompt_embeds`, `prompt_embeds_mask` 等输入。 文档字符串说明：Return real text lengths, treating a missing mask as all tokens valid.

### Lines 34-41: Function `calculate_dimensions` / 函数 `calculate_dimensions`
```python
def calculate_dimensions(target_area, ratio):
    width = math.sqrt(target_area * ratio)
    height = width / ratio

    width = round(width / 32) * 32
    height = round(height / 32) * 32

    return width, height
```
**EN:** This function drives `calculate_dimensions` with inputs such as `target_area`, `ratio`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `calculate_dimensions`，主要处理 `target_area`, `ratio` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 45-55: Function `retrieve_latents` / 函数 `retrieve_latents`
```python
def retrieve_latents(
    encoder_output: torch.Tensor,
    generator: Optional[torch.Generator] = None,
    sample_mode: str = "sample",
):
    if sample_mode == "sample":
        return encoder_output.sample(generator)
    elif sample_mode == "argmax":
        return encoder_output.mode()
    else:
        return encoder_output
```
**EN:** This function drives `retrieve_latents` with inputs such as `encoder_output`, `generator`, `sample_mode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `retrieve_latents`，主要处理 `encoder_output`, `generator`, `sample_mode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 59-121: Function `retrieve_timesteps` / 函数 `retrieve_timesteps`
```python
def retrieve_timesteps(
    scheduler,
    num_inference_steps: Optional[int] = None,
    device: Optional[Union[str, torch.device]] = None,
    timesteps: Optional[List[int]] = None,
    sigmas: Optional[List[float]] = None,
    **kwargs,
):
    r"""
    Calls the scheduler's `set_timesteps` method and retrieves timesteps from the scheduler after the call. Handles
    custom timesteps. Any kwargs will be supplied to `scheduler.set_timesteps`.

    Args:
        scheduler (`SchedulerMixin`):
# ...
    else:
        scheduler.set_timesteps(num_inference_steps, device=device, **kwargs)
        timesteps = scheduler.timesteps
    return timesteps, num_inference_steps
```
**EN:** This function drives `retrieve_timesteps` with inputs such as `scheduler`, `num_inference_steps`, `device`, `timesteps`. Calls the scheduler's `set_timesteps` method and retrieves timesteps from the scheduler after the call.
**CN:** 这个函数负责 `retrieve_timesteps`，主要处理 `scheduler`, `num_inference_steps`, `device`, `timesteps` 等输入。 文档字符串说明：Calls the scheduler's `set_timesteps` method and retrieves timesteps from the scheduler after the call.

### Lines 124-559: Class `QwenImageLayeredBeforeDenoisingStage` / 类 `QwenImageLayeredBeforeDenoisingStage`
```python
class QwenImageLayeredBeforeDenoisingStage(PipelineStage):
    def __init__(
        self, vae, tokenizer, processor, transformer, scheduler, model_path
    ) -> None:
        super().__init__()
        self.vae = vae.to(torch.bfloat16)
        from transformers import Qwen2_5_VLForConditionalGeneration

        self.text_encoder = Qwen2_5_VLForConditionalGeneration.from_pretrained(
            model_path, subfolder="text_encoder"
        ).to(torch.bfloat16)
        self.tokenizer = tokenizer
        self.processor = processor
        self.transformer = transformer
# ...
        batch.txt_seq_lens = txt_seq_lens
        batch.img_shapes = img_shapes

        return batch
```
**EN:** This class models `QwenImageLayeredBeforeDenoisingStage` as a specialization of `PipelineStage`. Important methods include `__init__`, `component_uses`, `_extract_masked_hidden`, `get_image_caption`.
**CN:** 该类实现 `QwenImageLayeredBeforeDenoisingStage`，并继承/扩展 `PipelineStage`。 其中较重要的方法包括 `__init__`, `component_uses`, `_extract_masked_hidden`, `get_image_caption`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.managers.forward_context`, `sglang.multimodal_gen.runtime.managers.memory_managers.component_manager`, `sglang.multimodal_gen.runtime.models.vision_utils`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.base`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `numpy`, `torch`, `diffusers.image_processor`, `diffusers.utils.torch_utils`, `transformers`
- **Stdlib / 标准库**: `inspect`, `math`, `typing`
