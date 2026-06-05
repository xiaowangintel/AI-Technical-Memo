# nano_nemotron_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/nano_nemotron_vl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for nano nemotron vl so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 nano nemotron vl 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15: Comments and module notes
```python
# Copyright 2025 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
# Adapted from https://huggingface.co/nvidia/NVIDIA-Nemotron-Nano-12B-v2-VL-BF16/blob/cb5a65ff10232128389d882d805fa609427544f1/configuration.py

```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 16-22: Imports dependencies
```python
from typing import Any

from transformers.configuration_utils import PretrainedConfig

from sglang.srt.configs.nemotron_h import NemotronHConfig
from sglang.srt.configs.radio import RadioConfig
from sglang.srt.multimodal.internvl_utils import IMAGENET_MEAN, IMAGENET_STD
```
**EN:** This block groups related imports for the module, including typing.Any, transformers.configuration_utils.PretrainedConfig, sglang.srt.configs.nemotron_h.NemotronHConfig, sglang.srt.configs.radio.RadioConfig, sglang.srt.multimodal.internvl_utils.IMAGENET_MEAN, and 1 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 typing.Any, transformers.configuration_utils.PretrainedConfig, sglang.srt.configs.nemotron_h.NemotronHConfig, sglang.srt.configs.radio.RadioConfig, sglang.srt.multimodal.internvl_utils.IMAGENET_MEAN 等 1 项，为后续代码准备所需名称。

### Lines 23-24: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 25-30: Defines function float_triplet
```python
def float_triplet(seq: Any):
    a, b, c = tuple(seq)
    assert (
        isinstance(a, float) and isinstance(b, float) and isinstance(c, float)
    ), "expected three floats"
    return a, b, c
```
**EN:** This block defines function `float_triplet`. Parameters: seq.
**CN:** 该代码块定义函数 `float_triplet`。 参数包括 seq。

### Lines 31-32: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 33-33: Declares class NemotronH_Nano_VL_V2_Config
```python
class NemotronH_Nano_VL_V2_Config(PretrainedConfig):
```
**EN:** This block introduces class `NemotronH_Nano_VL_V2_Config` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `NemotronH_Nano_VL_V2_Config`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 34-35: Declares model_type, is_composition
```python
    model_type = "NemotronH_Nano_VL_V2"
    is_composition = True
```
**EN:** This block initializes a related set of values in the NemotronH_Nano_VL_V2_Config, including model_type, is_composition. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 NemotronH_Nano_VL_V2_Config 中初始化一组相关值，包括 model_type, is_composition。将这些赋值集中在一起有助于理解周边配置。

### Lines 36-36: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronH_Nano_VL_V2_Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronH_Nano_VL_V2_Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 37-112: Defines function NemotronH_Nano_VL_V2_Config.__init__ (part 1)
```python
    def __init__(
        self,
        vision_config=None,
        llm_config=None,
        sound_config=None,
        force_image_size: int = 512,
        patch_size: int = 16,
        downsample_ratio=0.5,
        template=None,
        ps_version="v2",
        image_tag_type="internvl",
        projector_hidden_size=4096,
        vit_hidden_size=1280,
        video_pruning_rate: float = 0.0,
        video_context_token: str = "<video>",
        img_context_token: str = "<image>",
        img_start_token: str = "<img>",
        img_end_token: str = "</img>",
        audio_context_token: str = "<so_embedding>",
        audio_start_token: str = "<so_start>",
        audio_end_token: str = "<so_end>",
        norm_mean: tuple[float, float, float] | list[float] = IMAGENET_MEAN,
        norm_std: tuple[float, float, float] | list[float] = IMAGENET_STD,
        use_thumbnail: bool = True,
        **kwargs,
    ):
        super().__init__(**kwargs)

        # Handle both cases: when loading from JSON (llm_config is dict) and when called internally by transformers (llm_config; vision_config are None)
        if llm_config is not None:
            self.llm_config = NemotronHConfig(**llm_config)
            assert isinstance(vision_config, dict), "vision_config must be a dictionary"
            self.raw_vision_config = vision_config
        else:
            assert vision_config is None
            self.llm_config = NemotronHConfig()
            self.raw_vision_config = {}

        # Audio (Parakeet) config: stored as a PretrainedConfig sub-object
        if sound_config is not None and isinstance(sound_config, dict):
            self.sound_config = PretrainedConfig.from_dict(sound_config)
        else:
            self.sound_config = sound_config

        # Assign configuration values
        vision_image_size = self.raw_vision_config.get("image_size", force_image_size)
        vision_patch_size = self.raw_vision_config.get("patch_size", patch_size)
        self.image_size = int(
            vision_image_size[0]
            if isinstance(vision_image_size, list)
            else vision_image_size
        )
        self.patch_size = int(
            vision_patch_size[0]
            if isinstance(vision_patch_size, list)
            else vision_patch_size
        )

        self.downsample_ratio = downsample_ratio
        self.video_context_token = video_context_token
        self.img_context_token = img_context_token
        self.template = template  # TODO move out of here and into the tokenizer
        self.ps_version = ps_version  # Pixel shuffle version
        self.image_tag_type = image_tag_type  # TODO: into the tokenizer too?
        self.projector_hidden_size = projector_hidden_size
        self.vit_hidden_size = vit_hidden_size
        self.video_pruning_rate = video_pruning_rate

        self.norm_mean = float_triplet(norm_mean)
        self.norm_std = float_triplet(norm_std)
        self.use_thumbnail = use_thumbnail
        self.img_start_token = img_start_token
        self.img_end_token = img_end_token
        self.audio_context_token = audio_context_token
        self.audio_start_token = audio_start_token
        self.audio_end_token = audio_end_token
```
**EN:** This block defines function `NemotronH_Nano_VL_V2_Config.__init__`. Parameters: self, vision_config, llm_config, sound_config, force_image_size, patch_size, downsample_ratio, template, ps_version, image_tag_type, projector_hidden_size, vit_hidden_size, video_pruning_rate, video_context_token, img_context_token, img_start_token, img_end_token, audio_context_token, audio_start_token, audio_end_token, norm_mean, norm_std, use_thumbnail. This subsection covers lines 37-112 of the same logical block.
**CN:** 该代码块定义函数 `NemotronH_Nano_VL_V2_Config.__init__`。 参数包括 self、vision_config、llm_config、sound_config、force_image_size、patch_size、downsample_ratio、template、ps_version、image_tag_type、projector_hidden_size、vit_hidden_size、video_pruning_rate、video_context_token、img_context_token、img_start_token、img_end_token、audio_context_token、audio_start_token、audio_end_token、norm_mean、norm_std、use_thumbnail。 本小节覆盖同一逻辑块中的第 37-112 行。

### Lines 113-113: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 114-117: Defines function NemotronH_Nano_VL_V2_Config.__init__ (part 2)
```python
        # Dynamic resolution: from vision_config top-level
        self.min_num_patches = self.raw_vision_config.get("min_num_patches", 0)
        self.max_num_patches = self.raw_vision_config.get("max_num_patches", 0)
        self.dynamic_resolution = self.min_num_patches > 0
```
**EN:** This block defines function `NemotronH_Nano_VL_V2_Config.__init__`. Parameters: self, vision_config, llm_config, sound_config, force_image_size, patch_size, downsample_ratio, template, ps_version, image_tag_type, projector_hidden_size, vit_hidden_size, video_pruning_rate, video_context_token, img_context_token, img_start_token, img_end_token, audio_context_token, audio_start_token, audio_end_token, norm_mean, norm_std, use_thumbnail. This subsection covers lines 114-117 of the same logical block.
**CN:** 该代码块定义函数 `NemotronH_Nano_VL_V2_Config.__init__`。 参数包括 self、vision_config、llm_config、sound_config、force_image_size、patch_size、downsample_ratio、template、ps_version、image_tag_type、projector_hidden_size、vit_hidden_size、video_pruning_rate、video_context_token、img_context_token、img_start_token、img_end_token、audio_context_token、audio_start_token、audio_end_token、norm_mean、norm_std、use_thumbnail。 本小节覆盖同一逻辑块中的第 114-117 行。

### Lines 118-118: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 119-131: Defines function NemotronH_Nano_VL_V2_Config.__init__ (part 3)
```python
        # Video temporal compression: from vision_config top-level
        self.video_temporal_patch_size = self.raw_vision_config.get(
            "video_temporal_patch_size", 1
        )
        self.separate_video_embedder = self.raw_vision_config.get(
            "separate_video_embedder", True
        )
        self.video_target_num_patches = self.raw_vision_config.get(
            "video_target_num_patches", 0
        )
        self.video_maintain_aspect_ratio = self.raw_vision_config.get(
            "video_maintain_aspect_ratio", True
        )
```
**EN:** This block defines function `NemotronH_Nano_VL_V2_Config.__init__`. Parameters: self, vision_config, llm_config, sound_config, force_image_size, patch_size, downsample_ratio, template, ps_version, image_tag_type, projector_hidden_size, vit_hidden_size, video_pruning_rate, video_context_token, img_context_token, img_start_token, img_end_token, audio_context_token, audio_start_token, audio_end_token, norm_mean, norm_std, use_thumbnail. This subsection covers lines 119-131 of the same logical block.
**CN:** 该代码块定义函数 `NemotronH_Nano_VL_V2_Config.__init__`。 参数包括 self、vision_config、llm_config、sound_config、force_image_size、patch_size、downsample_ratio、template、ps_version、image_tag_type、projector_hidden_size、vit_hidden_size、video_pruning_rate、video_context_token、img_context_token、img_start_token、img_end_token、audio_context_token、audio_start_token、audio_end_token、norm_mean、norm_std、use_thumbnail。 本小节覆盖同一逻辑块中的第 119-131 行。

### Lines 132-132: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronH_Nano_VL_V2_Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronH_Nano_VL_V2_Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 133-152: Defines function NemotronH_Nano_VL_V2_Config.create_radio_config
```python
    def create_radio_config(self):
        config = self.raw_vision_config
        model_name = config["args"]["model"]
        reg_tokens = config["args"].get("register_multiple")
        image_size = config.get("preferred_resolution", [224])[0]
        radio_config = RadioConfig(
            patch_size=self.patch_size,
            norm_mean=self.norm_mean,
            norm_std=self.norm_std,
            model_name=model_name,
            reg_tokens=reg_tokens,
            image_size=image_size,
            min_num_patches=self.min_num_patches,
            max_num_patches=self.max_num_patches,
            video_temporal_patch_size=self.video_temporal_patch_size,
            separate_video_embedder=self.separate_video_embedder,
            video_target_num_patches=self.video_target_num_patches,
            video_maintain_aspect_ratio=self.video_maintain_aspect_ratio,
        )
        return radio_config
```
**EN:** This block defines function `NemotronH_Nano_VL_V2_Config.create_radio_config`. Parameters: self.
**CN:** 该代码块定义函数 `NemotronH_Nano_VL_V2_Config.create_radio_config`。 参数包括 self。

### Lines 153-154: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 155-155: Declares class NemotronH_Nano_Omni_Reasoning_V3_Config
```python
class NemotronH_Nano_Omni_Reasoning_V3_Config(NemotronH_Nano_VL_V2_Config):
```
**EN:** This block introduces class `NemotronH_Nano_Omni_Reasoning_V3_Config` as a reusable abstraction inside the module. It inherits from NemotronH_Nano_VL_V2_Config.
**CN:** 该代码块声明类 `NemotronH_Nano_Omni_Reasoning_V3_Config`，作为模块中的可复用抽象。 它继承自 NemotronH_Nano_VL_V2_Config。

### Lines 156-156: Declares model_type
```python
    model_type = "NemotronH_Nano_Omni_Reasoning_V3"
```
**EN:** This statement initializes model_type in the NemotronH_Nano_Omni_Reasoning_V3_Config. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 NemotronH_Nano_Omni_Reasoning_V3_Config 中初始化 model_type。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 157-157: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the NemotronH_Nano_Omni_Reasoning_V3_Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 NemotronH_Nano_Omni_Reasoning_V3_Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 158-161: Defines function NemotronH_Nano_Omni_Reasoning_V3_Config.__init__
```python
    def __init__(self, *args, **kwargs):
        # Explicit __init__ prevents PretrainedConfig.__init_subclass__ from
        # replacing the parent's custom __init__ with a dataclass-generated one.
        super().__init__(*args, **kwargs)
```
**EN:** This block defines function `NemotronH_Nano_Omni_Reasoning_V3_Config.__init__`. Parameters: self.
**CN:** 该代码块定义函数 `NemotronH_Nano_Omni_Reasoning_V3_Config.__init__`。 参数包括 self。

## Key Concepts / 关键概念
- **Classes / 类**: `NemotronH_Nano_VL_V2_Config`, `NemotronH_Nano_Omni_Reasoning_V3_Config`
- **Functions / 函数**: `float_triplet`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `typing`
- **Third-Party / 第三方**: `transformers.configuration_utils`
- **Local Modules / 本地模块**: `sglang.srt.configs.nemotron_h`, `sglang.srt.configs.radio`, `sglang.srt.multimodal.internvl_utils`
