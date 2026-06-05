# glm_image.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/model_specific_stages/glm_image.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for glm image, connecting stages, tensors, and runtime metadata. Key symbols include `calculate_shift`, `retrieve_timesteps`, `retrieve_latents`. / 该模块负责 glm image 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `calculate_shift`, `retrieve_timesteps`, `retrieve_latents`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Imports and module setup / 导入与模块初始化
```python
import inspect
import re
import time
from math import sqrt
from typing import List, Optional, Tuple, Union

import numpy as np
import PIL
import torch
from diffusers.image_processor import VaeImageProcessor
from diffusers.utils.torch_utils import randn_tensor

from sglang.multimodal_gen.runtime.distributed import get_local_torch_device
from sglang.multimodal_gen.runtime.managers.forward_context import set_forward_context
# ...
from sglang.multimodal_gen.runtime.server_args import ServerArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 25-33: Function `calculate_shift` / 函数 `calculate_shift`
```python
def calculate_shift(
    image_seq_len,
    base_seq_len: int = 256,
    base_shift: float = 0.25,
    max_shift: float = 0.75,
) -> float:
    m = (image_seq_len / base_seq_len) ** 0.5
    mu = m * max_shift + base_shift
    return mu
```
**EN:** This function drives `calculate_shift` with inputs such as `image_seq_len`, `base_seq_len`, `base_shift`, `max_shift`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `calculate_shift`，主要处理 `image_seq_len`, `base_seq_len`, `base_shift`, `max_shift` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 36-87: Function `retrieve_timesteps` / 函数 `retrieve_timesteps`
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
    """
    accepts_timesteps = "timesteps" in set(
        inspect.signature(scheduler.set_timesteps).parameters.keys()
# ...
    else:
        scheduler.set_timesteps(num_inference_steps, device=device, **kwargs)
        timesteps = scheduler.timesteps
    return timesteps, num_inference_steps
```
**EN:** This function drives `retrieve_timesteps` with inputs such as `scheduler`, `num_inference_steps`, `device`, `timesteps`. Calls the scheduler's `set_timesteps` method and retrieves timesteps from the scheduler after the call.
**CN:** 这个函数负责 `retrieve_timesteps`，主要处理 `scheduler`, `num_inference_steps`, `device`, `timesteps` 等输入。 文档字符串说明：Calls the scheduler's `set_timesteps` method and retrieves timesteps from the scheduler after the call.

### Lines 91-103: Function `retrieve_latents` / 函数 `retrieve_latents`
```python
def retrieve_latents(
    encoder_output: torch.Tensor,
    generator: Optional[torch.Generator] = None,
    sample_mode: str = "sample",
):
    if hasattr(encoder_output, "latent_dist") and sample_mode == "sample":
        return encoder_output.latent_dist.sample(generator)
    elif hasattr(encoder_output, "latent_dist") and sample_mode == "argmax":
        return encoder_output.latent_dist.mode()
    elif hasattr(encoder_output, "latents"):
        return encoder_output.latents
    else:
        raise AttributeError("Could not access latents of provided encoder_output")
```
**EN:** This function drives `retrieve_latents` with inputs such as `encoder_output`, `generator`, `sample_mode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `retrieve_latents`，主要处理 `encoder_output`, `generator`, `sample_mode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 106-825: Class `GlmImageBeforeDenoisingStage` / 类 `GlmImageBeforeDenoisingStage`
```python
class GlmImageBeforeDenoisingStage(PipelineStage):
    r"""
    Pipeline for text-to-image generation using GLM-Image.

    This pipeline integrates both the AR (autoregressive) model for token generation and the DiT (diffusion
    transformer) model for image decoding.

    Args:
        vae ([`AutoencoderKL`]):
            Variational Auto-Encoder (VAE) Model to encode and decode images to and from latent representations.
        text_encoder ([`T5EncoderModel`]):
            Frozen text-encoder for glyph embeddings.
        tokenizer (`PreTrainedTokenizer`):
            Tokenizer for the text encoder.
# ...
        batch.height = height
        batch.width = width

        return batch
```
**EN:** This class models `GlmImageBeforeDenoisingStage` as a specialization of `PipelineStage`. Pipeline for text-to-image generation using GLM-Image. Important methods include `__init__`, `_parse_and_expand_shape_info`, `_build_image_grid_thw`, `_calculate_ar_generation_params`.
**CN:** 该类实现 `GlmImageBeforeDenoisingStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Pipeline for text-to-image generation using GLM-Image. 其中较重要的方法包括 `__init__`, `_parse_and_expand_shape_info`, `_build_image_grid_thw`, `_calculate_ar_generation_params`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Pipeline orchestration / 流水线编排
- Scheduling and batching / 调度与批处理
- Caching strategy / 缓存策略

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.managers.forward_context`, `sglang.multimodal_gen.runtime.models.dits.glm_image`, `sglang.multimodal_gen.runtime.models.vision_utils`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.base`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `numpy`, `PIL`, `torch`, `diffusers.image_processor`, `diffusers.utils.torch_utils`
- **Stdlib / 标准库**: `inspect`, `re`, `time`, `math`, `typing`
