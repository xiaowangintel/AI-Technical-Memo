# ltx_2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/sample/ltx_2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `LTX2SamplingParams`, `LTX23SamplingParams`, and `LTX23HQSamplingParams`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `LTX2SamplingParams`、`LTX23SamplingParams` 和 `LTX23HQSamplingParams` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: module setup and imports / 模块初始化与导入
```python
import dataclasses
from dataclasses import field
from typing import Any

from sglang.multimodal_gen.configs.sample.sampling_params import SamplingParams
```
**EN:** This block establishes the module context and imports `dataclasses`, `typing`, and `sglang.multimodal_gen.configs.sample.sampling_params`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`typing` 和 `sglang.multimodal_gen.configs.sample.sampling_params`。这些依赖为后续实现提供所需符号。

### Lines 9-13: `LTX2SamplingParams` class overview / `LTX2SamplingParams` 类概览
```python
class LTX2SamplingParams(SamplingParams):
    """Sampling parameters for LTX-2."""

    # Match the reference defaults used by ltx-pipelines (one-stage).
    # See: LTX-2/packages/ltx-pipelines/src/ltx_pipelines/utils/constants.py
```
**EN:** This block defines class `LTX2SamplingParams`. Sampling parameters for LTX-2. It inherits from `SamplingParams`.
**CN:** 该代码块定义了类 `LTX2SamplingParams`。 它用于封装 ltx2 sampling params 相关行为。 它继承自 `SamplingParams`。

### Lines 14-43: supporting statements / 辅助语句
```python
    seed: int = 10
    generator_device: str = "cpu"

    # Video parameters
    height: int = 512
    width: int = 768
    num_frames: int = 121
    fps: int = 24

    # Audio specific
    generate_audio: bool = True

    # Denoising parameters
    guidance_scale: float = 4.0
    num_inference_steps: int = 40

    # Match ltx-pipelines default negative prompt (covers video + audio artifacts).
    negative_prompt: str = (
        "blurry, out of focus, overexposed, underexposed, low contrast, washed out colors, excessive noise, "
        "grainy texture, poor lighting, flickering, motion blur, distorted proportions, unnatural skin tones, "
        "deformed facial features, asymmetrical face, missing facial features, extra limbs, disfigured hands, "
        "wrong hand count, artifacts around text, inconsistent perspective, camera shake, incorrect depth of "
        "field, background too sharp, background clutter, distracting reflections, harsh shadows, inconsistent "
        "lighting direction, color banding, cartoonish rendering, 3D CGI look, unrealistic materials, uncanny "
        "valley effect, incorrect ethnicity, wrong gender, exaggerated expressions, wrong gaze direction, "
        "mismatched lip sync, silent or muted audio, distorted voice, robotic voice, echo, background noise, "
        "off-sync audio, incorrect dialogue, added dialogue, repetitive speech, jittery movement, awkward "
        "pauses, incorrect timing, unnatural transitions, inconsistent framing, tilted camera, flat lighting, "
        "inconsistent tone, cinematic oversaturation, stylized filters, or AI artifacts."
    )
```
**EN:** This block gathers supporting statements inside `LTX2SamplingParams`. It updates names such as `seed`, `generator_device`, `height`, `width`, `num_frames`, and `fps`.
**CN:** 该代码块汇集了位于 `LTX2SamplingParams` 内部的辅助语句。 它会更新 `seed`、`generator_device`、`height`、`width`、`num_frames` 和 `fps` 等名称。

### Lines 47-49: `LTX23SamplingParams` class overview / `LTX23SamplingParams` 类概览
```python
class LTX23SamplingParams(LTX2SamplingParams):
    """Sampling parameters matching official LTX-2.3 one-stage defaults."""
```
**EN:** This block defines class `LTX23SamplingParams`. Sampling parameters matching official LTX-2.3 one-stage defaults. It inherits from `LTX2SamplingParams`.
**CN:** 该代码块定义了类 `LTX23SamplingParams`。 它用于封装 ltx23 sampling params 相关行为。 它继承自 `LTX2SamplingParams`。

### Lines 50-68: supporting statements / 辅助语句
```python
    seed: int = 42
    generator_device: str = "cuda"
    guidance_scale: float = 3.0
    num_inference_steps: int = 30

    video_cfg_scale: float = 3.0
    video_stg_scale: float = 1.0
    video_rescale_scale: float = 0.7
    video_modality_scale: float = 3.0
    video_skip_step: int = 0
    video_stg_blocks: list[int] = field(default_factory=lambda: [28])

    audio_cfg_scale: float = 7.0
    audio_stg_scale: float = 1.0
    audio_rescale_scale: float = 0.7
    audio_modality_scale: float = 3.0
    audio_skip_step: int = 0
    audio_stg_blocks: list[int] = field(default_factory=lambda: [28])
    skip_v2a_cross_attn_for_video_gt: bool = False
```
**EN:** This block gathers supporting statements inside `LTX23SamplingParams`. It updates names such as `seed`, `generator_device`, `guidance_scale`, `num_inference_steps`, `video_cfg_scale`, and `video_stg_scale`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `LTX23SamplingParams` 内部的辅助语句。 它会更新 `seed`、`generator_device`、`guidance_scale`、`num_inference_steps`、`video_cfg_scale` 和 `video_stg_scale` 等名称。 代码会与 `field` 协同工作。

### Lines 70-88: `build_request_extra` implementation / `build_request_extra` 实现
```python
    def build_request_extra(self) -> dict[str, Any]:
        extra = super().build_request_extra()
        extra["ltx2_stage1_guider_params"] = {
            "video_cfg_scale": self.video_cfg_scale,
            "video_stg_scale": self.video_stg_scale,
            "video_rescale_scale": self.video_rescale_scale,
            "video_modality_scale": self.video_modality_scale,
            "video_skip_step": self.video_skip_step,
            "video_stg_blocks": self.video_stg_blocks,
            "audio_cfg_scale": self.audio_cfg_scale,
            "audio_stg_scale": self.audio_stg_scale,
            "audio_rescale_scale": self.audio_rescale_scale,
            "audio_modality_scale": self.audio_modality_scale,
            "audio_skip_step": self.audio_skip_step,
            "audio_stg_blocks": self.audio_stg_blocks,
        }
        if self.skip_v2a_cross_attn_for_video_gt:
            extra["ltx2_skip_v2a_cross_attn_for_video_gt"] = True
        return extra
```
**EN:** This block defines method `build_request_extra` on `LTX23SamplingParams`. It builds request extra. Key calls include `super.build_request_extra`, and `super`. The implementation branches on conditions.
**CN:** 该代码块定义了 `LTX23SamplingParams` 的方法 `build_request_extra`。 它用于构建request extra。 关键调用包括 `super.build_request_extra` 和 `super`。 实现中包含条件分支。

### Lines 92-94: `LTX23HQSamplingParams` class overview / `LTX23HQSamplingParams` 类概览
```python
class LTX23HQSamplingParams(LTX23SamplingParams):
    """Sampling parameters matching official LTX-2.3 HQ two-stage defaults."""
```
**EN:** This block defines class `LTX23HQSamplingParams`. Sampling parameters matching official LTX-2.3 HQ two-stage defaults. It inherits from `LTX23SamplingParams`.
**CN:** 该代码块定义了类 `LTX23HQSamplingParams`。 它用于封装 ltx23 hqsampling params 相关行为。 它继承自 `LTX23SamplingParams`。

### Lines 95-113: supporting statements / 辅助语句
```python
    height: int = 1088
    width: int = 1920
    num_inference_steps: int = 15
    distilled_lora_strength_stage_1: float = 0.25
    distilled_lora_strength_stage_2: float = 0.5

    video_cfg_scale: float = 3.0
    video_stg_scale: float = 0.0
    video_rescale_scale: float = 0.45
    video_modality_scale: float = 3.0
    video_skip_step: int = 0
    video_stg_blocks: list[int] = field(default_factory=list)

    audio_cfg_scale: float = 7.0
    audio_stg_scale: float = 0.0
    audio_rescale_scale: float = 1.0
    audio_modality_scale: float = 3.0
    audio_skip_step: int = 0
    audio_stg_blocks: list[int] = field(default_factory=list)
```
**EN:** This block gathers supporting statements inside `LTX23HQSamplingParams`. It updates names such as `height`, `width`, `num_inference_steps`, `distilled_lora_strength_stage_1`, `distilled_lora_strength_stage_2`, and `video_cfg_scale`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `LTX23HQSamplingParams` 内部的辅助语句。 它会更新 `height`、`width`、`num_inference_steps`、`distilled_lora_strength_stage_1`、`distilled_lora_strength_stage_2` 和 `video_cfg_scale` 等名称。 代码会与 `field` 协同工作。

### Lines 115-123: `build_request_extra` implementation / `build_request_extra` 实现
```python
    def build_request_extra(self) -> dict[str, Any]:
        extra = super().build_request_extra()
        extra["ltx2_distilled_lora_strength_stage_1"] = float(
            self.distilled_lora_strength_stage_1
        )
        extra["ltx2_distilled_lora_strength_stage_2"] = float(
            self.distilled_lora_strength_stage_2
        )
        return extra
```
**EN:** This block defines method `build_request_extra` on `LTX23HQSamplingParams`. It builds request extra. Key calls include `super.build_request_extra`, `float`, and `super`.
**CN:** 该代码块定义了 `LTX23HQSamplingParams` 的方法 `build_request_extra`。 它用于构建request extra。 关键调用包括 `super.build_request_extra`、`float` 和 `super`。

## Key Concepts / 关键概念
- `LTX2SamplingParams`: Sampling parameters for LTX-2. / 核心类，用于封装 ltx2 sampling params 相关行为。
- `LTX23SamplingParams`: Sampling parameters matching official LTX-2.3 one-stage defaults. / 核心类，用于封装 ltx23 sampling params 相关行为。
- `LTX23HQSamplingParams`: Sampling parameters matching official LTX-2.3 HQ two-stage defaults. / 核心类，用于封装 ltx23 hqsampling params 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`

- **Total lines / 总行数**: 123
