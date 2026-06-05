# hunyuan.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/pipeline_configs/hunyuan.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `PromptTemplate`, `HunyuanConfig`, and `FastHunyuanConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `PromptTemplate`、`HunyuanConfig` 和 `FastHunyuanConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-22: module setup and imports / 模块初始化与导入
```python
from collections.abc import Callable
from dataclasses import dataclass, field
from typing import TypedDict

import torch

from sglang.multimodal_gen.configs.models import DiTConfig, EncoderConfig, VAEConfig
from sglang.multimodal_gen.configs.models.dits import HunyuanVideoConfig
from sglang.multimodal_gen.configs.models.encoders import (
    BaseEncoderOutput,
    CLIPTextConfig,
    LlamaConfig,
)
from sglang.multimodal_gen.configs.models.vaes import HunyuanVAEConfig
from sglang.multimodal_gen.configs.pipeline_configs.base import (
    ModelTaskType,
    PipelineConfig,
    TextConditioningOutput,
)
```
**EN:** This block establishes the module context and imports `collections.abc`, `dataclasses`, `typing`, `torch`, `sglang.multimodal_gen.configs.models`, and `sglang.multimodal_gen.configs.models.dits`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `collections.abc`、`dataclasses`、`typing`、`torch`、`sglang.multimodal_gen.configs.models` 和 `sglang.multimodal_gen.configs.models.dits`。这些依赖为后续实现提供所需符号。

### Lines 24-32: supporting statements / 辅助语句
```python
PROMPT_TEMPLATE_ENCODE_VIDEO = (
    "<|start_header_id|>system<|end_header_id|>\n\nDescribe the video by detailing the following aspects: "
    "1. The main content and theme of the video."
    "2. The color, shape, size, texture, quantity, text, and spatial relationships of the objects."
    "3. Actions, events, behaviors temporal relationships, physical movement changes of the objects."
    "4. background environment, light, style and atmosphere."
    "5. camera angles, movements, and transitions used in the video:<|eot_id|>"
    "<|start_header_id|>user<|end_header_id|>\n\n{}<|eot_id|>"
)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `PROMPT_TEMPLATE_ENCODE_VIDEO`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `PROMPT_TEMPLATE_ENCODE_VIDEO` 等名称。

### Lines 35-35: `PromptTemplate` class overview / `PromptTemplate` 类概览
```python
class PromptTemplate(TypedDict):
```
**EN:** This block defines class `PromptTemplate`. It encapsulates prompt template behavior. It inherits from `TypedDict`.
**CN:** 该代码块定义了类 `PromptTemplate`。 它用于封装 prompt template 相关行为。 它继承自 `TypedDict`。

### Lines 36-37: supporting statements / 辅助语句
```python
    template: str
    crop_start: int
```
**EN:** This block gathers supporting statements inside `PromptTemplate`. It updates names such as `template`, and `crop_start`.
**CN:** 该代码块汇集了位于 `PromptTemplate` 内部的辅助语句。 它会更新 `template` 和 `crop_start` 等名称。

### Lines 40-43: supporting statements / 辅助语句
```python
prompt_template_video: PromptTemplate = {
    "template": PROMPT_TEMPLATE_ENCODE_VIDEO,
    "crop_start": 95,
}
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `prompt_template_video`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `prompt_template_video` 等名称。

### Lines 46-47: `llama_preprocess_text` implementation / `llama_preprocess_text` 实现
```python
def llama_preprocess_text(prompt: str) -> str:
    return prompt_template_video["template"].format(prompt)
```
**EN:** This block defines function `llama_preprocess_text`. It handles llama preprocess text logic. Key calls include `prompt_template_video.format`. Parameters such as `prompt` drive the behavior in this section.
**CN:** 该代码块定义了函数 `llama_preprocess_text`。 它用于处理 llama preprocess text 相关逻辑。 关键调用包括 `prompt_template_video.format`。 本段逻辑主要由 `prompt` 等参数驱动。

### Lines 50-69: `llama_postprocess_text` implementation / `llama_postprocess_text` 实现
```python
def llama_postprocess_text(
    outputs: BaseEncoderOutput, _text_inputs
) -> TextConditioningOutput:
    hidden_state_skip_layer = 2
    assert outputs.hidden_states is not None
    hidden_states: tuple[torch.Tensor, ...] = outputs.hidden_states
    last_hidden_state: torch.Tensor = hidden_states[-(hidden_state_skip_layer + 1)]
    crop_start = prompt_template_video.get("crop_start", -1)
    last_hidden_state = last_hidden_state[:, crop_start:]
    attention_mask = _text_inputs.attention_mask.to(
        device=last_hidden_state.device, dtype=torch.bool
    )
    if crop_start < 0:
        attention_mask = attention_mask[:, crop_start:]
    else:
        attention_mask = attention_mask[
            :, crop_start : crop_start + last_hidden_state.shape[1]
        ]
    seq_lens = [int(x) for x in attention_mask.to(torch.int64).sum(dim=1).tolist()]
    return TextConditioningOutput(last_hidden_state, attention_mask, seq_lens)
```
**EN:** This block defines function `llama_postprocess_text`. It handles llama postprocess text logic. Key calls include `prompt_template_video.get`, `_text_inputs.attention_mask.to`, `TextConditioningOutput`, `int`, and `attention_mask.to.sum.tolist`. The implementation branches on conditions. Parameters such as `outputs`, and `_text_inputs` drive the behavior in this section.
**CN:** 该代码块定义了函数 `llama_postprocess_text`。 它用于处理 llama postprocess text 相关逻辑。 关键调用包括 `prompt_template_video.get`、`_text_inputs.attention_mask.to`、`TextConditioningOutput`、`int` 和 `attention_mask.to.sum.tolist`。 实现中包含条件分支。 本段逻辑主要由 `outputs` 和 `_text_inputs` 等参数驱动。

### Lines 72-80: `clip_postprocess_text` implementation / `clip_postprocess_text` 实现
```python
def clip_postprocess_text(
    outputs: BaseEncoderOutput, _text_inputs
) -> TextConditioningOutput:
    pooler_output: torch.Tensor = outputs.pooler_output
    batch_size = int(pooler_output.shape[0])
    prompt_embeds_mask = torch.ones(
        (batch_size, 1), dtype=torch.bool, device=pooler_output.device
    )
    return TextConditioningOutput(pooler_output, prompt_embeds_mask, [1] * batch_size)
```
**EN:** This block defines function `clip_postprocess_text`. It handles clip postprocess text logic. Key calls include `int`, `torch.ones`, and `TextConditioningOutput`. Parameters such as `outputs`, and `_text_inputs` drive the behavior in this section.
**CN:** 该代码块定义了函数 `clip_postprocess_text`。 它用于处理 clip postprocess text 相关逻辑。 关键调用包括 `int`、`torch.ones` 和 `TextConditioningOutput`。 本段逻辑主要由 `outputs` 和 `_text_inputs` 等参数驱动。

### Lines 84-86: `HunyuanConfig` class overview / `HunyuanConfig` 类概览
```python
class HunyuanConfig(PipelineConfig):
    """Base configuration for HunYuan pipeline architecture."""
```
**EN:** This block defines class `HunyuanConfig`. Base configuration for HunYuan pipeline architecture. It inherits from `PipelineConfig`.
**CN:** 该代码块定义了类 `HunyuanConfig`。 它用于封装 hunyuan config 相关行为。 它继承自 `PipelineConfig`。

### Lines 87-114: supporting statements / 辅助语句
```python
    task_type: ModelTaskType = ModelTaskType.T2V

    # HunyuanConfig-specific parameters with defaults
    # DiT
    dit_config: DiTConfig = field(default_factory=HunyuanVideoConfig)
    # VAE
    vae_config: VAEConfig = field(default_factory=HunyuanVAEConfig)
    # Denoising stage
    embedded_cfg_scale: int = 6
    flow_shift: int = 7

    # Text encoding stage
    text_encoder_configs: tuple[EncoderConfig, ...] = field(
        default_factory=lambda: (LlamaConfig(), CLIPTextConfig())
    )
    preprocess_text_funcs: tuple[Callable[[str], str] | None, ...] = field(
        default_factory=lambda: (llama_preprocess_text, None)
    )
    postprocess_text_funcs: tuple[Callable[[BaseEncoderOutput], torch.tensor], ...] = (
        field(default_factory=lambda: (llama_postprocess_text, clip_postprocess_text))
    )

    # Precision for each component
    dit_precision: str = "bf16"
    vae_precision: str = "fp16"
    text_encoder_precisions: tuple[str, ...] = field(
        default_factory=lambda: ("fp16", "fp16")
    )
```
**EN:** This block gathers supporting statements inside `HunyuanConfig`. It updates names such as `task_type`, `dit_config`, `vae_config`, `embedded_cfg_scale`, `flow_shift`, and `text_encoder_configs`. The code collaborates with `field`, `LlamaConfig`, and `CLIPTextConfig`.
**CN:** 该代码块汇集了位于 `HunyuanConfig` 内部的辅助语句。 它会更新 `task_type`、`dit_config`、`vae_config`、`embedded_cfg_scale`、`flow_shift` 和 `text_encoder_configs` 等名称。 代码会与 `field`、`LlamaConfig` 和 `CLIPTextConfig` 协同工作。

### Lines 116-118: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        self.vae_config.load_encoder = False
        self.vae_config.load_decoder = True
```
**EN:** This block defines method `__post_init__` on `HunyuanConfig`. It post-processes init.
**CN:** 该代码块定义了 `HunyuanConfig` 的方法 `__post_init__`。 它用于后处理init。

### Lines 122-125: `FastHunyuanConfig` class overview / `FastHunyuanConfig` 类概览
```python
class FastHunyuanConfig(HunyuanConfig):
    """Configuration specifically optimized for FastHunyuan weights."""

    # Override HunyuanConfig defaults
```
**EN:** This block defines class `FastHunyuanConfig`. Configuration specifically optimized for FastHunyuan weights. It inherits from `HunyuanConfig`.
**CN:** 该代码块定义了类 `FastHunyuanConfig`。 它用于封装 fast hunyuan config 相关行为。 它继承自 `HunyuanConfig`。

### Lines 126-126: supporting statements / 辅助语句
```python
    flow_shift: int = 17
```
**EN:** This block gathers supporting statements inside `FastHunyuanConfig`. It updates names such as `flow_shift`.
**CN:** 该代码块汇集了位于 `FastHunyuanConfig` 内部的辅助语句。 它会更新 `flow_shift` 等名称。

## Key Concepts / 关键概念
- `PromptTemplate`: Primary class that encapsulates prompt template behavior. / 核心类，用于封装 prompt template 相关行为。
- `llama_preprocess_text`: Top-level function that handles llama preprocess text logic. / 顶层函数，用于处理 llama preprocess text 相关逻辑。
- `llama_postprocess_text`: Top-level function that handles llama postprocess text logic. / 顶层函数，用于处理 llama postprocess text 相关逻辑。
- `clip_postprocess_text`: Top-level function that handles clip postprocess text logic. / 顶层函数，用于处理 clip postprocess text 相关逻辑。
- `HunyuanConfig`: Base configuration for HunYuan pipeline architecture. / 核心类，用于封装 hunyuan config 相关行为。
- `FastHunyuanConfig`: Configuration specifically optimized for FastHunyuan weights. / 核心类，用于封装 fast hunyuan config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections.abc`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits`, `sglang.multimodal_gen.configs.models.encoders`, `sglang.multimodal_gen.configs.models.vaes`, `sglang.multimodal_gen.configs.pipeline_configs.base`

- **Total lines / 总行数**: 129
