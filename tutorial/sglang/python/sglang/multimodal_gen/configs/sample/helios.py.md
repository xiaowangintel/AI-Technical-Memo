# helios.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/sample/helios.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `HeliosT2VSamplingParams`, `HeliosMidSamplingParams`, and `HeliosDistilledSamplingParams`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `HeliosT2VSamplingParams`、`HeliosMidSamplingParams` 和 `HeliosDistilledSamplingParams` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 2-4: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field

from sglang.multimodal_gen.configs.sample.sampling_params import SamplingParams
```
**EN:** This block establishes the module context and imports `dataclasses`, and `sglang.multimodal_gen.configs.sample.sampling_params`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses` 和 `sglang.multimodal_gen.configs.sample.sampling_params`。这些依赖为后续实现提供所需符号。

### Lines 8-9: `HeliosT2VSamplingParams` class overview / `HeliosT2VSamplingParams` 类概览
```python
class HeliosT2VSamplingParams(SamplingParams):
    # Video parameters
```
**EN:** This block defines class `HeliosT2VSamplingParams`. It encapsulates helios t2 vsampling params behavior. It inherits from `SamplingParams`.
**CN:** 该代码块定义了类 `HeliosT2VSamplingParams`。 它用于封装 helios t2 vsampling params 相关行为。 它继承自 `SamplingParams`。

### Lines 10-35: supporting statements / 辅助语句
```python
    height: int = 384
    width: int = 640
    num_frames: int = 99
    fps: int = 24

    # Denoising stage
    guidance_scale: float = 5.0
    negative_prompt: str = (
        "Bright tones, overexposed, static, blurred details, subtitles, style, "
        "works, paintings, images, static, overall gray, worst quality, low quality, "
        "JPEG compression residue, ugly, incomplete, extra fingers, poorly drawn hands, "
        "poorly drawn faces, deformed, disfigured, misshapen limbs, fused fingers, "
        "still picture, messy background, three legs, many people in the background, "
        "walking backwards"
    )
    num_inference_steps: int = 50

    # Helios T2V supported resolutions
    supported_resolutions: list[tuple[int, int]] | None = field(
        default_factory=lambda: [
            (640, 384),  # ~5:3
            (384, 640),  # ~3:5
            (832, 480),  # ~16:9-ish
            (480, 832),  # ~9:16-ish
        ]
    )
```
**EN:** This block gathers supporting statements inside `HeliosT2VSamplingParams`. It updates names such as `height`, `width`, `num_frames`, `fps`, `guidance_scale`, and `negative_prompt`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `HeliosT2VSamplingParams` 内部的辅助语句。 它会更新 `height`、`width`、`num_frames`、`fps`、`guidance_scale` 和 `negative_prompt` 等名称。 代码会与 `field` 协同工作。

### Lines 39-41: `HeliosMidSamplingParams` class overview / `HeliosMidSamplingParams` 类概览
```python
class HeliosMidSamplingParams(HeliosT2VSamplingParams):
    """Sampling params for Helios-Mid (Stage 2 pyramid SR)."""
```
**EN:** This block defines class `HeliosMidSamplingParams`. Sampling params for Helios-Mid (Stage 2 pyramid SR). It inherits from `HeliosT2VSamplingParams`.
**CN:** 该代码块定义了类 `HeliosMidSamplingParams`。 它用于封装 helios mid sampling params 相关行为。 它继承自 `HeliosT2VSamplingParams`。

### Lines 42-42: supporting statements / 辅助语句
```python
    num_inference_steps: int = 20
```
**EN:** This block gathers supporting statements inside `HeliosMidSamplingParams`. It updates names such as `num_inference_steps`.
**CN:** 该代码块汇集了位于 `HeliosMidSamplingParams` 内部的辅助语句。 它会更新 `num_inference_steps` 等名称。

### Lines 46-48: `HeliosDistilledSamplingParams` class overview / `HeliosDistilledSamplingParams` 类概览
```python
class HeliosDistilledSamplingParams(HeliosT2VSamplingParams):
    """Sampling params for Helios-Distilled (DMD, no CFG needed)."""
```
**EN:** This block defines class `HeliosDistilledSamplingParams`. Sampling params for Helios-Distilled (DMD, no CFG needed). It inherits from `HeliosT2VSamplingParams`.
**CN:** 该代码块定义了类 `HeliosDistilledSamplingParams`。 它用于封装 helios distilled sampling params 相关行为。 它继承自 `HeliosT2VSamplingParams`。

### Lines 49-50: supporting statements / 辅助语句
```python
    guidance_scale: float = 1.0
    num_inference_steps: int = 10
```
**EN:** This block gathers supporting statements inside `HeliosDistilledSamplingParams`. It updates names such as `guidance_scale`, and `num_inference_steps`.
**CN:** 该代码块汇集了位于 `HeliosDistilledSamplingParams` 内部的辅助语句。 它会更新 `guidance_scale` 和 `num_inference_steps` 等名称。

## Key Concepts / 关键概念
- `HeliosT2VSamplingParams`: Primary class that encapsulates helios t2 vsampling params behavior. / 核心类，用于封装 helios t2 vsampling params 相关行为。
- `HeliosMidSamplingParams`: Sampling params for Helios-Mid (Stage 2 pyramid SR). / 核心类，用于封装 helios mid sampling params 相关行为。
- `HeliosDistilledSamplingParams`: Sampling params for Helios-Distilled (DMD, no CFG needed). / 核心类，用于封装 helios distilled sampling params 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`

- **Total lines / 总行数**: 50
