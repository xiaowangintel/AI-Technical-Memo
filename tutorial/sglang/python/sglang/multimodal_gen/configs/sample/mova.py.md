# mova.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/sample/mova.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `MOVASamplingParams`, `MOVA_360P_SamplingParams`, and `MOVA_720P_SamplingParams`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `MOVASamplingParams`、`MOVA_360P_SamplingParams` 和 `MOVA_720P_SamplingParams` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 2-4: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field

from sglang.multimodal_gen.configs.sample.sampling_params import SamplingParams
```
**EN:** This block establishes the module context and imports `dataclasses`, and `sglang.multimodal_gen.configs.sample.sampling_params`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses` 和 `sglang.multimodal_gen.configs.sample.sampling_params`。这些依赖为后续实现提供所需符号。

### Lines 8-9: `MOVASamplingParams` class overview / `MOVASamplingParams` 类概览
```python
class MOVASamplingParams(SamplingParams):
    # Video parameters (MOVA defaults)
```
**EN:** This block defines class `MOVASamplingParams`. It encapsulates movasampling params behavior. It inherits from `SamplingParams`.
**CN:** 该代码块定义了类 `MOVASamplingParams`。 它用于封装 movasampling params 相关行为。 它继承自 `SamplingParams`。

### Lines 10-29: supporting statements / 辅助语句
```python
    height: int = 352
    width: int = 640
    num_frames: int = 193
    fps: int = 24

    # Denoising stage
    guidance_scale: float = 5.0
    num_inference_steps: int = 50
    sigma_shift: float = 5.0
    visual_shift: float = 5.0
    audio_shift: float = 5.0

    adjust_frames: bool = False

    negative_prompt: str = (
        "色调艳丽，过曝，静态，细节模糊不清，字幕，风格，作品，画作，画面，静止，"
        "整体发灰，最差质量，低质量，JPEG压缩残留，丑陋的，残缺的，多余的手指，"
        "画得不好的手部，画得不好的脸部，畸形的，毁容的，形态畸形的肢体，手指融合，"
        "静止不动的画面，杂乱的背景，三条腿，背景人很多，倒着走"
    )
```
**EN:** This block gathers supporting statements inside `MOVASamplingParams`. It updates names such as `height`, `width`, `num_frames`, `fps`, `guidance_scale`, and `num_inference_steps`.
**CN:** 该代码块汇集了位于 `MOVASamplingParams` 内部的辅助语句。 它会更新 `height`、`width`、`num_frames`、`fps`、`guidance_scale` 和 `num_inference_steps` 等名称。

### Lines 33-34: `MOVA_360P_SamplingParams` class overview / `MOVA_360P_SamplingParams` 类概览
```python
class MOVA_360P_SamplingParams(MOVASamplingParams):
    # Video parameters (MOVA 360P)
```
**EN:** This block defines class `MOVA_360P_SamplingParams`. It encapsulates mova 360 p sampling params behavior. It inherits from `MOVASamplingParams`.
**CN:** 该代码块定义了类 `MOVA_360P_SamplingParams`。 它用于封装 mova 360 p sampling params 相关行为。 它继承自 `MOVASamplingParams`。

### Lines 35-44: supporting statements / 辅助语句
```python
    height: int = 352
    width: int = 640

    # MOVA 360P supported resolutions
    supported_resolutions: list[tuple[int, int]] = field(
        default_factory=lambda: [
            (352, 640),
            (640, 352),
        ]
    )
```
**EN:** This block gathers supporting statements inside `MOVA_360P_SamplingParams`. It updates names such as `height`, `width`, and `supported_resolutions`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `MOVA_360P_SamplingParams` 内部的辅助语句。 它会更新 `height`、`width` 和 `supported_resolutions` 等名称。 代码会与 `field` 协同工作。

### Lines 48-49: `MOVA_720P_SamplingParams` class overview / `MOVA_720P_SamplingParams` 类概览
```python
class MOVA_720P_SamplingParams(MOVASamplingParams):
    # Video parameters (MOVA 720P)
```
**EN:** This block defines class `MOVA_720P_SamplingParams`. It encapsulates mova 720 p sampling params behavior. It inherits from `MOVASamplingParams`.
**CN:** 该代码块定义了类 `MOVA_720P_SamplingParams`。 它用于封装 mova 720 p sampling params 相关行为。 它继承自 `MOVASamplingParams`。

### Lines 50-59: supporting statements / 辅助语句
```python
    height: int = 720
    width: int = 1280

    # MOVA 720P supported resolutions
    supported_resolutions: list[tuple[int, int]] = field(
        default_factory=lambda: [
            (720, 1280),
            (1280, 720),
        ]
    )
```
**EN:** This block gathers supporting statements inside `MOVA_720P_SamplingParams`. It updates names such as `height`, `width`, and `supported_resolutions`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `MOVA_720P_SamplingParams` 内部的辅助语句。 它会更新 `height`、`width` 和 `supported_resolutions` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `MOVASamplingParams`: Primary class that encapsulates movasampling params behavior. / 核心类，用于封装 movasampling params 相关行为。
- `MOVA_360P_SamplingParams`: Primary class that encapsulates mova 360 p sampling params behavior. / 核心类，用于封装 mova 360 p sampling params 相关行为。
- `MOVA_720P_SamplingParams`: Primary class that encapsulates mova 720 p sampling params behavior. / 核心类，用于封装 mova 720 p sampling params 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`

- **Total lines / 总行数**: 59
