# flux.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/sample/flux.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `FluxSamplingParams`, `Flux2SamplingParams`, and `Flux2KleinSamplingParams`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `FluxSamplingParams`、`Flux2SamplingParams` 和 `Flux2KleinSamplingParams` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-7: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass
from typing import ClassVar

from sglang.multimodal_gen.configs.sample.sampling_params import SamplingParams
```
**EN:** This block establishes the module context and imports `dataclasses`, `typing`, and `sglang.multimodal_gen.configs.sample.sampling_params`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`typing` 和 `sglang.multimodal_gen.configs.sample.sampling_params`。这些依赖为后续实现提供所需符号。

### Lines 11-11: `FluxSamplingParams` class overview / `FluxSamplingParams` 类概览
```python
class FluxSamplingParams(SamplingParams):
```
**EN:** This block defines class `FluxSamplingParams`. It encapsulates flux sampling params behavior. It inherits from `SamplingParams`.
**CN:** 该代码块定义了类 `FluxSamplingParams`。 它用于封装 flux sampling params 相关行为。 它继承自 `SamplingParams`。

### Lines 12-19: supporting statements / 辅助语句
```python
    _default_height: ClassVar[int] = 128 * 8  # default_sample_size * vae_scale_factor
    _default_width: ClassVar[int] = 128 * 8

    num_frames: int = 1
    # Denoising stage
    guidance_scale: float = 3.5
    negative_prompt: str = None
    num_inference_steps: int = 50
```
**EN:** This block gathers supporting statements inside `FluxSamplingParams`. It updates names such as `_default_height`, `_default_width`, `num_frames`, `guidance_scale`, `negative_prompt`, and `num_inference_steps`.
**CN:** 该代码块汇集了位于 `FluxSamplingParams` 内部的辅助语句。 它会更新 `_default_height`、`_default_width`、`num_frames`、`guidance_scale`、`negative_prompt` 和 `num_inference_steps` 等名称。

### Lines 23-23: `Flux2SamplingParams` class overview / `Flux2SamplingParams` 类概览
```python
class Flux2SamplingParams(FluxSamplingParams):
```
**EN:** This block defines class `Flux2SamplingParams`. It encapsulates flux2 sampling params behavior. It inherits from `FluxSamplingParams`.
**CN:** 该代码块定义了类 `Flux2SamplingParams`。 它用于封装 flux2 sampling params 相关行为。 它继承自 `FluxSamplingParams`。

### Lines 24-24: supporting statements / 辅助语句
```python
    guidance_scale: float = 4.0
```
**EN:** This block gathers supporting statements inside `Flux2SamplingParams`. It updates names such as `guidance_scale`.
**CN:** 该代码块汇集了位于 `Flux2SamplingParams` 内部的辅助语句。 它会更新 `guidance_scale` 等名称。

### Lines 28-29: `Flux2KleinSamplingParams` class overview / `Flux2KleinSamplingParams` 类概览
```python
class Flux2KleinSamplingParams(Flux2SamplingParams):
    # Klein is step-distilled, so default to 4 steps
```
**EN:** This block defines class `Flux2KleinSamplingParams`. It encapsulates flux2 klein sampling params behavior. It inherits from `Flux2SamplingParams`.
**CN:** 该代码块定义了类 `Flux2KleinSamplingParams`。 它用于封装 flux2 klein sampling params 相关行为。 它继承自 `Flux2SamplingParams`。

### Lines 30-31: supporting statements / 辅助语句
```python
    guidance_scale: float = 1.0
    num_inference_steps: int = 4
```
**EN:** This block gathers supporting statements inside `Flux2KleinSamplingParams`. It updates names such as `guidance_scale`, and `num_inference_steps`.
**CN:** 该代码块汇集了位于 `Flux2KleinSamplingParams` 内部的辅助语句。 它会更新 `guidance_scale` 和 `num_inference_steps` 等名称。

## Key Concepts / 关键概念
- `FluxSamplingParams`: Primary class that encapsulates flux sampling params behavior. / 核心类，用于封装 flux sampling params 相关行为。
- `Flux2SamplingParams`: Primary class that encapsulates flux2 sampling params behavior. / 核心类，用于封装 flux2 sampling params 相关行为。
- `Flux2KleinSamplingParams`: Primary class that encapsulates flux2 klein sampling params behavior. / 核心类，用于封装 flux2 klein sampling params 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`

- **Total lines / 总行数**: 31
