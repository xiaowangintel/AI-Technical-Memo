# qwenimage.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/sample/qwenimage.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `QwenImageSamplingParams`, `QwenImage2512SamplingParams`, and `QwenImageEditPlusSamplingParams`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `QwenImageSamplingParams`、`QwenImage2512SamplingParams` 和 `QwenImageEditPlusSamplingParams` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-6: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass

from sglang.multimodal_gen.configs.sample.sampling_params import SamplingParams
```
**EN:** This block establishes the module context and imports `dataclasses`, and `sglang.multimodal_gen.configs.sample.sampling_params`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses` 和 `sglang.multimodal_gen.configs.sample.sampling_params`。这些依赖为后续实现提供所需符号。

### Lines 10-10: `QwenImageSamplingParams` class overview / `QwenImageSamplingParams` 类概览
```python
class QwenImageSamplingParams(SamplingParams):
```
**EN:** This block defines class `QwenImageSamplingParams`. It encapsulates qwen image sampling params behavior. It inherits from `SamplingParams`.
**CN:** 该代码块定义了类 `QwenImageSamplingParams`。 它用于封装 qwen image sampling params 相关行为。 它继承自 `SamplingParams`。

### Lines 11-15: supporting statements / 辅助语句
```python
    negative_prompt: str = " "
    num_frames: int = 1
    # Denoising stage
    guidance_scale: float = 4.0
    num_inference_steps: int = 50
```
**EN:** This block gathers supporting statements inside `QwenImageSamplingParams`. It updates names such as `negative_prompt`, `num_frames`, `guidance_scale`, and `num_inference_steps`.
**CN:** 该代码块汇集了位于 `QwenImageSamplingParams` 内部的辅助语句。 它会更新 `negative_prompt`、`num_frames`、`guidance_scale` 和 `num_inference_steps` 等名称。

### Lines 19-19: `QwenImage2512SamplingParams` class overview / `QwenImage2512SamplingParams` 类概览
```python
class QwenImage2512SamplingParams(QwenImageSamplingParams):
```
**EN:** This block defines class `QwenImage2512SamplingParams`. It encapsulates qwen image2512 sampling params behavior. It inherits from `QwenImageSamplingParams`.
**CN:** 该代码块定义了类 `QwenImage2512SamplingParams`。 它用于封装 qwen image2512 sampling params 相关行为。 它继承自 `QwenImageSamplingParams`。

### Lines 20-22: supporting statements / 辅助语句
```python
    negative_prompt: str = (
        "低分辨率，低画质，肢体畸形，手指畸形，画面过饱和，蜡像感，人脸无细节，过度光滑，画面具有AI感。构图混乱。文字模糊，扭曲。"
    )
```
**EN:** This block gathers supporting statements inside `QwenImage2512SamplingParams`. It updates names such as `negative_prompt`.
**CN:** 该代码块汇集了位于 `QwenImage2512SamplingParams` 内部的辅助语句。 它会更新 `negative_prompt` 等名称。

### Lines 26-27: `QwenImageEditPlusSamplingParams` class overview / `QwenImageEditPlusSamplingParams` 类概览
```python
class QwenImageEditPlusSamplingParams(QwenImageSamplingParams):
    # Denoising stage
```
**EN:** This block defines class `QwenImageEditPlusSamplingParams`. It encapsulates qwen image edit plus sampling params behavior. It inherits from `QwenImageSamplingParams`.
**CN:** 该代码块定义了类 `QwenImageEditPlusSamplingParams`。 它用于封装 qwen image edit plus sampling params 相关行为。 它继承自 `QwenImageSamplingParams`。

### Lines 28-30: supporting statements / 辅助语句
```python
    guidance_scale: float = 4.0
    # true_cfg_scale: float = 4.0
    num_inference_steps: int = 40
```
**EN:** This block gathers supporting statements inside `QwenImageEditPlusSamplingParams`. It updates names such as `guidance_scale`, and `num_inference_steps`.
**CN:** 该代码块汇集了位于 `QwenImageEditPlusSamplingParams` 内部的辅助语句。 它会更新 `guidance_scale` 和 `num_inference_steps` 等名称。

### Lines 34-35: `QwenImageLayeredSamplingParams` class overview / `QwenImageLayeredSamplingParams` 类概览
```python
class QwenImageLayeredSamplingParams(QwenImageSamplingParams):
    # num_frames: int = 4
```
**EN:** This block defines class `QwenImageLayeredSamplingParams`. It encapsulates qwen image layered sampling params behavior. It inherits from `QwenImageSamplingParams`.
**CN:** 该代码块定义了类 `QwenImageLayeredSamplingParams`。 它用于封装 qwen image layered sampling params 相关行为。 它继承自 `QwenImageSamplingParams`。

### Lines 36-44: supporting statements / 辅助语句
```python
    height: int = 640
    width: int = 640
    prompt: str = " "
    negative_prompt: str = " "

    guidance_scale: float = 4.0
    num_inference_steps: int = 50
    cfg_normalize: bool = True
    use_en_prompt: bool = True
```
**EN:** This block gathers supporting statements inside `QwenImageLayeredSamplingParams`. It updates names such as `height`, `width`, `prompt`, `negative_prompt`, `guidance_scale`, and `num_inference_steps`.
**CN:** 该代码块汇集了位于 `QwenImageLayeredSamplingParams` 内部的辅助语句。 它会更新 `height`、`width`、`prompt`、`negative_prompt`、`guidance_scale` 和 `num_inference_steps` 等名称。

## Key Concepts / 关键概念
- `QwenImageSamplingParams`: Primary class that encapsulates qwen image sampling params behavior. / 核心类，用于封装 qwen image sampling params 相关行为。
- `QwenImage2512SamplingParams`: Primary class that encapsulates qwen image2512 sampling params behavior. / 核心类，用于封装 qwen image2512 sampling params 相关行为。
- `QwenImageEditPlusSamplingParams`: Primary class that encapsulates qwen image edit plus sampling params behavior. / 核心类，用于封装 qwen image edit plus sampling params 相关行为。
- `QwenImageLayeredSamplingParams`: Primary class that encapsulates qwen image layered sampling params behavior. / 核心类，用于封装 qwen image layered sampling params 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`

- **Total lines / 总行数**: 44
