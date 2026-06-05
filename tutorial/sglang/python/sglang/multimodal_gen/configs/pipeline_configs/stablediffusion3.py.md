# stablediffusion3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/pipeline_configs/stablediffusion3.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `SD3CLIPTextArchConfig`, `SD3CLIPTextConfig`, and `SD3T5ArchConfig`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Stable Diffusion 3 pipeline configuration. / 该文件属于配置层。它围绕 `SD3CLIPTextArchConfig`、`SD3CLIPTextConfig` 和 `SD3T5ArchConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-28: module setup and imports / 模块初始化与导入
```python
"""Stable Diffusion 3 pipeline configuration."""

import os
from dataclasses import dataclass, field
from typing import Callable

import torch

from sglang.multimodal_gen.configs.models import DiTConfig, EncoderConfig, VAEConfig
from sglang.multimodal_gen.configs.models.dits import StableDiffusion3TransformerConfig
from sglang.multimodal_gen.configs.models.encoders import BaseEncoderOutput
from sglang.multimodal_gen.configs.models.encoders.base import TextEncoderArchConfig
from sglang.multimodal_gen.configs.models.encoders.clip import (
    CLIPTextArchConfig,
    CLIPTextConfig,
)
from sglang.multimodal_gen.configs.models.encoders.t5 import (
    T5ArchConfig,
    T5Config,
)
from sglang.multimodal_gen.configs.models.vaes.stablediffusion3 import (
    StableDiffusion3VAEConfig,
)
from sglang.multimodal_gen.configs.pipeline_configs.base import (
    ModelTaskType,
    SpatialImagePipelineConfig,
)
```
**EN:** This block establishes the module context and imports `os`, `dataclasses`, `typing`, `torch`, `sglang.multimodal_gen.configs.models`, and `sglang.multimodal_gen.configs.models.dits`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `os`、`dataclasses`、`typing`、`torch`、`sglang.multimodal_gen.configs.models` 和 `sglang.multimodal_gen.configs.models.dits`。这些依赖为后续实现提供所需符号。

### Lines 31-37: `sd3_clip_postprocess_text` implementation / `sd3_clip_postprocess_text` 实现
```python
def sd3_clip_postprocess_text(outputs: BaseEncoderOutput, _text_inputs) -> torch.Tensor:
    """Extract pre-final hidden state for SD3 CLIP encoders."""
    if outputs.hidden_states is None:
        raise ValueError(
            "SD3 CLIP postprocessing requires hidden_states from encoder output."
        )
    return outputs.hidden_states[-2]
```
**EN:** This block defines function `sd3_clip_postprocess_text`. Extract pre-final hidden state for SD3 CLIP encoders. Key calls include `ValueError`. The implementation branches on conditions. Parameters such as `outputs`, and `_text_inputs` drive the behavior in this section.
**CN:** 该代码块定义了函数 `sd3_clip_postprocess_text`。 它用于处理 sd3 clip postprocess text 相关逻辑。 关键调用包括 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `outputs` 和 `_text_inputs` 等参数驱动。

### Lines 40-41: `t5_postprocess_text` implementation / `t5_postprocess_text` 实现
```python
def t5_postprocess_text(outputs: BaseEncoderOutput, _text_inputs) -> torch.Tensor:
    return outputs.last_hidden_state
```
**EN:** This block defines function `t5_postprocess_text`. It handles t5 postprocess text logic. Parameters such as `outputs`, and `_text_inputs` drive the behavior in this section.
**CN:** 该代码块定义了函数 `t5_postprocess_text`。 它用于处理 t5 postprocess text 相关逻辑。 本段逻辑主要由 `outputs` 和 `_text_inputs` 等参数驱动。

### Lines 44-63: `select_sd3_vae_weight_files` implementation / `select_sd3_vae_weight_files` 实现
```python
def select_sd3_vae_weight_files(
    safetensors_list: list[str],
    component_model_path: str,
    component_name: str,
    vae_precision: str,
) -> list[str]:
    """Select SD3 VAE checkpoint file candidates with minimal policy."""
    if component_name not in ("vae", "video_vae"):
        return safetensors_list

    base_name = "diffusion_pytorch_model"
    if vae_precision == "fp16":
        fp16_path = os.path.join(component_model_path, f"{base_name}.fp16.safetensors")
        if os.path.exists(fp16_path):
            return [fp16_path]

    full_path = os.path.join(component_model_path, f"{base_name}.safetensors")
    if os.path.exists(full_path):
        return [full_path]
    return safetensors_list
```
**EN:** This block defines function `select_sd3_vae_weight_files`. Select SD3 VAE checkpoint file candidates with minimal policy. Key calls include `os.path.join`, and `os.path.exists`. The implementation branches on conditions. Parameters such as `safetensors_list`, `component_model_path`, `component_name`, and `vae_precision` drive the behavior in this section.
**CN:** 该代码块定义了函数 `select_sd3_vae_weight_files`。 它用于选择sd3 vae weight files。 关键调用包括 `os.path.join` 和 `os.path.exists`。 实现中包含条件分支。 本段逻辑主要由 `safetensors_list`、`component_model_path`、`component_name` 和 `vae_precision` 等参数驱动。

### Lines 67-67: `SD3CLIPTextArchConfig` class overview / `SD3CLIPTextArchConfig` 类概览
```python
class SD3CLIPTextArchConfig(CLIPTextArchConfig):
```
**EN:** This block defines class `SD3CLIPTextArchConfig`. It encapsulates sd3 cliptext arch config behavior. It inherits from `CLIPTextArchConfig`.
**CN:** 该代码块定义了类 `SD3CLIPTextArchConfig`。 它用于封装 sd3 cliptext arch config 相关行为。 它继承自 `CLIPTextArchConfig`。

### Lines 68-75: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self) -> None:
        super().__post_init__()
        self.tokenizer_kwargs.update(
            {
                "max_length": self.text_len,
                "padding": "max_length",
            }
        )
```
**EN:** This block defines method `__post_init__` on `SD3CLIPTextArchConfig`. It post-processes init. Key calls include `super.__post_init__`, `self.tokenizer_kwargs.update`, and `super`.
**CN:** 该代码块定义了 `SD3CLIPTextArchConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `super.__post_init__`、`self.tokenizer_kwargs.update` 和 `super`。

### Lines 79-79: `SD3CLIPTextConfig` class overview / `SD3CLIPTextConfig` 类概览
```python
class SD3CLIPTextConfig(CLIPTextConfig):
```
**EN:** This block defines class `SD3CLIPTextConfig`. It encapsulates sd3 cliptext config behavior. It inherits from `CLIPTextConfig`.
**CN:** 该代码块定义了类 `SD3CLIPTextConfig`。 它用于封装 sd3 cliptext config 相关行为。 它继承自 `CLIPTextConfig`。

### Lines 80-80: supporting statements / 辅助语句
```python
    arch_config: TextEncoderArchConfig = field(default_factory=SD3CLIPTextArchConfig)
```
**EN:** This block gathers supporting statements inside `SD3CLIPTextConfig`. It updates names such as `arch_config`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `SD3CLIPTextConfig` 内部的辅助语句。 它会更新 `arch_config` 等名称。 代码会与 `field` 协同工作。

### Lines 84-84: `SD3T5ArchConfig` class overview / `SD3T5ArchConfig` 类概览
```python
class SD3T5ArchConfig(T5ArchConfig):
```
**EN:** This block defines class `SD3T5ArchConfig`. It encapsulates sd3 t5 arch config behavior. It inherits from `T5ArchConfig`.
**CN:** 该代码块定义了类 `SD3T5ArchConfig`。 它用于封装 sd3 t5 arch config 相关行为。 它继承自 `T5ArchConfig`。

### Lines 85-87: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self) -> None:
        super().__post_init__()
        self.tokenizer_kwargs.update({"max_length": 256})
```
**EN:** This block defines method `__post_init__` on `SD3T5ArchConfig`. It post-processes init. Key calls include `super.__post_init__`, `self.tokenizer_kwargs.update`, and `super`.
**CN:** 该代码块定义了 `SD3T5ArchConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `super.__post_init__`、`self.tokenizer_kwargs.update` 和 `super`。

### Lines 91-91: `SD3T5Config` class overview / `SD3T5Config` 类概览
```python
class SD3T5Config(T5Config):
```
**EN:** This block defines class `SD3T5Config`. It encapsulates sd3 t5 config behavior. It inherits from `T5Config`.
**CN:** 该代码块定义了类 `SD3T5Config`。 它用于封装 sd3 t5 config 相关行为。 它继承自 `T5Config`。

### Lines 92-92: supporting statements / 辅助语句
```python
    arch_config: TextEncoderArchConfig = field(default_factory=SD3T5ArchConfig)
```
**EN:** This block gathers supporting statements inside `SD3T5Config`. It updates names such as `arch_config`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `SD3T5Config` 内部的辅助语句。 它会更新 `arch_config` 等名称。 代码会与 `field` 协同工作。

### Lines 96-102: `StableDiffusion3PipelineConfig` class overview / `StableDiffusion3PipelineConfig` 类概览
```python
class StableDiffusion3PipelineConfig(SpatialImagePipelineConfig):
    """Configuration for SD3 image generation pipeline.

    This config intentionally relies on SD3-specific encoder configs to provide
    tokenizer kwargs, instead of stage-level tokenizer overrides.
    """
```
**EN:** This block defines class `StableDiffusion3PipelineConfig`. Configuration for SD3 image generation pipeline. This config intentionally relies on SD3-specific encoder configs to provide tokenizer kwargs, instead of stage-level tokenizer overrides. It inherits from `SpatialImagePipelineConfig`.
**CN:** 该代码块定义了类 `StableDiffusion3PipelineConfig`。 它用于封装 stable diffusion3 pipeline config 相关行为。 它继承自 `SpatialImagePipelineConfig`。

### Lines 103-139: supporting statements / 辅助语句
```python
    task_type: ModelTaskType = ModelTaskType.T2I

    dit_config: DiTConfig = field(default_factory=StableDiffusion3TransformerConfig)
    vae_config: VAEConfig = field(default_factory=StableDiffusion3VAEConfig)

    text_encoder_configs: tuple[EncoderConfig, ...] = field(
        default_factory=lambda: (
            SD3CLIPTextConfig(),
            SD3CLIPTextConfig(),
            SD3T5Config(),
        )
    )

    text_encoder_precisions: tuple[str, ...] = field(
        default_factory=lambda: ("fp16", "fp16", "fp32")
    )

    preprocess_text_funcs: tuple[Callable[[str], str] | None, ...] = field(
        default_factory=lambda: (
            None,
            None,
            None,
        )
    )

    postprocess_text_funcs: tuple[
        Callable[[BaseEncoderOutput, dict], torch.Tensor], ...
    ] = field(
        default_factory=lambda: (
            sd3_clip_postprocess_text,
            sd3_clip_postprocess_text,
            t5_postprocess_text,
        )
    )

    should_use_guidance: bool = False
    guidance_scale: float = 7.0
```
**EN:** This block gathers supporting statements inside `StableDiffusion3PipelineConfig`. It updates names such as `task_type`, `dit_config`, `vae_config`, `text_encoder_configs`, `text_encoder_precisions`, and `preprocess_text_funcs`. The code collaborates with `field`, `SD3CLIPTextConfig`, and `SD3T5Config`.
**CN:** 该代码块汇集了位于 `StableDiffusion3PipelineConfig` 内部的辅助语句。 它会更新 `task_type`、`dit_config`、`vae_config`、`text_encoder_configs`、`text_encoder_precisions` 和 `preprocess_text_funcs` 等名称。 代码会与 `field`、`SD3CLIPTextConfig` 和 `SD3T5Config` 协同工作。

### Lines 141-146: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self) -> None:
        configs = list(self.text_encoder_configs)
        configs[0].update_model_arch({"_class_name": "CLIPTextModelWithProjection"})
        configs[1].update_model_arch({"_class_name": "CLIPTextModelWithProjection"})
        configs[2].update_model_arch({"_class_name": "T5EncoderModel"})
        self.text_encoder_configs = tuple(configs)
```
**EN:** This block defines method `__post_init__` on `StableDiffusion3PipelineConfig`. It post-processes init. Key calls include `list`, `configs.update_model_arch`, and `tuple`.
**CN:** 该代码块定义了 `StableDiffusion3PipelineConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `list`、`configs.update_model_arch` 和 `tuple`。

### Lines 148-152: `get_text_encoder_pooler_output` implementation / `get_text_encoder_pooler_output` 实现
```python
    def get_text_encoder_pooler_output(self, outputs, encoder_index):
        # SD3 uses pooled embeddings only from the two CLIP encoders (indices 0 and 1).
        if encoder_index <= 1:
            return outputs.pooler_output
        return None
```
**EN:** This block defines method `get_text_encoder_pooler_output` on `StableDiffusion3PipelineConfig`. It retrieves text encoder pooler output. The implementation branches on conditions. Parameters such as `outputs`, and `encoder_index` drive the behavior in this section.
**CN:** 该代码块定义了 `StableDiffusion3PipelineConfig` 的方法 `get_text_encoder_pooler_output`。 它用于获取text encoder pooler output。 实现中包含条件分支。 本段逻辑主要由 `outputs` 和 `encoder_index` 等参数驱动。

### Lines 154-166: `select_vae_weight_files` implementation / `select_vae_weight_files` 实现
```python
    def select_vae_weight_files(
        self,
        safetensors_list: list[str],
        component_model_path: str,
        component_name: str,
        vae_precision: str,
    ) -> list[str]:
        return select_sd3_vae_weight_files(
            safetensors_list=safetensors_list,
            component_model_path=component_model_path,
            component_name=component_name,
            vae_precision=vae_precision,
        )
```
**EN:** This block defines method `select_vae_weight_files` on `StableDiffusion3PipelineConfig`. It selects vae weight files. Key calls include `select_sd3_vae_weight_files`. Parameters such as `safetensors_list`, `component_model_path`, `component_name`, and `vae_precision` drive the behavior in this section.
**CN:** 该代码块定义了 `StableDiffusion3PipelineConfig` 的方法 `select_vae_weight_files`。 它用于选择vae weight files。 关键调用包括 `select_sd3_vae_weight_files`。 本段逻辑主要由 `safetensors_list`、`component_model_path`、`component_name` 和 `vae_precision` 等参数驱动。

### Lines 168-171: `tokenize_prompt` implementation / `tokenize_prompt` 实现
```python
    def tokenize_prompt(self, prompt: list[str], tokenizer, tok_kwargs) -> dict:
        text_inputs = tokenizer(prompt, **tok_kwargs)
        text_inputs["attention_mask"] = None
        return text_inputs
```
**EN:** This block defines method `tokenize_prompt` on `StableDiffusion3PipelineConfig`. It handles tokenize prompt logic. Key calls include `tokenizer`. Parameters such as `prompt`, `tokenizer`, and `tok_kwargs` drive the behavior in this section.
**CN:** 该代码块定义了 `StableDiffusion3PipelineConfig` 的方法 `tokenize_prompt`。 它用于处理 tokenize prompt 相关逻辑。 关键调用包括 `tokenizer`。 本段逻辑主要由 `prompt`、`tokenizer` 和 `tok_kwargs` 等参数驱动。

### Lines 173-174: `get_pos_prompt_embeds` implementation / `get_pos_prompt_embeds` 实现
```python
    def get_pos_prompt_embeds(self, batch):
        return batch.prompt_embeds[0]
```
**EN:** This block defines method `get_pos_prompt_embeds` on `StableDiffusion3PipelineConfig`. It retrieves pos prompt embeds. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `StableDiffusion3PipelineConfig` 的方法 `get_pos_prompt_embeds`。 它用于获取pos prompt embeds。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 176-177: `get_neg_prompt_embeds` implementation / `get_neg_prompt_embeds` 实现
```python
    def get_neg_prompt_embeds(self, batch):
        return batch.negative_prompt_embeds[0]
```
**EN:** This block defines method `get_neg_prompt_embeds` on `StableDiffusion3PipelineConfig`. It retrieves neg prompt embeds. Parameters such as `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `StableDiffusion3PipelineConfig` 的方法 `get_neg_prompt_embeds`。 它用于获取neg prompt embeds。 本段逻辑主要由 `batch` 等参数驱动。

### Lines 179-184: `prepare_pos_cond_kwargs` implementation / `prepare_pos_cond_kwargs` 实现
```python
    def prepare_pos_cond_kwargs(self, batch, device, rotary_emb, dtype):
        return {
            "pooled_projections": (
                batch.pooled_embeds[0] if batch.pooled_embeds else None
            )
        }
```
**EN:** This block defines method `prepare_pos_cond_kwargs` on `StableDiffusion3PipelineConfig`. It prepares pos cond kwargs. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `StableDiffusion3PipelineConfig` 的方法 `prepare_pos_cond_kwargs`。 它用于准备pos cond kwargs。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 186-191: `prepare_neg_cond_kwargs` implementation / `prepare_neg_cond_kwargs` 实现
```python
    def prepare_neg_cond_kwargs(self, batch, device, rotary_emb, dtype):
        return {
            "pooled_projections": (
                batch.neg_pooled_embeds[0] if batch.neg_pooled_embeds else None
            )
        }
```
**EN:** This block defines method `prepare_neg_cond_kwargs` on `StableDiffusion3PipelineConfig`. It prepares neg cond kwargs. Parameters such as `batch`, `device`, `rotary_emb`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `StableDiffusion3PipelineConfig` 的方法 `prepare_neg_cond_kwargs`。 它用于准备neg cond kwargs。 本段逻辑主要由 `batch`、`device`、`rotary_emb` 和 `dtype` 等参数驱动。

### Lines 194-202: `prepare_latent_shape` implementation / `prepare_latent_shape` 实现
```python
    def prepare_latent_shape(self, batch, batch_size, num_frames):  # noqa: ARG002
        spatial_ratio = self.vae_config.arch_config.spatial_compression_ratio
        in_channels = self.dit_config.arch_config.in_channels
        return (
            batch_size,
            in_channels,
            batch.height // spatial_ratio,
            batch.width // spatial_ratio,
        )
```
**EN:** This block defines method `prepare_latent_shape` on `StableDiffusion3PipelineConfig`. It prepares latent shape. Parameters such as `batch`, `batch_size`, and `num_frames` drive the behavior in this section.
**CN:** 该代码块定义了 `StableDiffusion3PipelineConfig` 的方法 `prepare_latent_shape`。 它用于准备latent shape。 本段逻辑主要由 `batch`、`batch_size` 和 `num_frames` 等参数驱动。

## Key Concepts / 关键概念
- `sd3_clip_postprocess_text`: Extract pre-final hidden state for SD3 CLIP encoders. / 顶层函数，用于处理 sd3 clip postprocess text 相关逻辑。
- `t5_postprocess_text`: Top-level function that handles t5 postprocess text logic. / 顶层函数，用于处理 t5 postprocess text 相关逻辑。
- `select_sd3_vae_weight_files`: Select SD3 VAE checkpoint file candidates with minimal policy. / 顶层函数，用于选择sd3 vae weight files。
- `SD3CLIPTextArchConfig`: Primary class that encapsulates sd3 cliptext arch config behavior. / 核心类，用于封装 sd3 cliptext arch config 相关行为。
- `SD3CLIPTextConfig`: Primary class that encapsulates sd3 cliptext config behavior. / 核心类，用于封装 sd3 cliptext config 相关行为。
- `SD3T5ArchConfig`: Primary class that encapsulates sd3 t5 arch config behavior. / 核心类，用于封装 sd3 t5 arch config 相关行为。
- `SD3T5Config`: Primary class that encapsulates sd3 t5 config behavior. / 核心类，用于封装 sd3 t5 config 相关行为。
- `StableDiffusion3PipelineConfig`: Configuration for SD3 image generation pipeline. / 核心类，用于封装 stable diffusion3 pipeline config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits`, `sglang.multimodal_gen.configs.models.encoders`, `sglang.multimodal_gen.configs.models.encoders.base`, `sglang.multimodal_gen.configs.models.encoders.clip`, `sglang.multimodal_gen.configs.models.encoders.t5`, `sglang.multimodal_gen.configs.models.vaes.stablediffusion3`, `sglang.multimodal_gen.configs.pipeline_configs.base`

- **Total lines / 总行数**: 202
