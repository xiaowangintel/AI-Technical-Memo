# hunyuan.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/sample/hunyuan.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `HunyuanSamplingParams`, and `FastHunyuanSamplingParam`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `HunyuanSamplingParams` 和 `FastHunyuanSamplingParam` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-7: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field

from sglang.multimodal_gen.configs.sample.sampling_params import SamplingParams
from sglang.multimodal_gen.configs.sample.teacache import TeaCacheParams
```
**EN:** This block establishes the module context and imports `dataclasses`, `sglang.multimodal_gen.configs.sample.sampling_params`, and `sglang.multimodal_gen.configs.sample.teacache`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`sglang.multimodal_gen.configs.sample.sampling_params` 和 `sglang.multimodal_gen.configs.sample.teacache`。这些依赖为后续实现提供所需符号。

### Lines 11-11: `HunyuanSamplingParams` class overview / `HunyuanSamplingParams` 类概览
```python
class HunyuanSamplingParams(SamplingParams):
```
**EN:** This block defines class `HunyuanSamplingParams`. It encapsulates hunyuan sampling params behavior. It inherits from `SamplingParams`.
**CN:** 该代码块定义了类 `HunyuanSamplingParams`。 它用于封装 hunyuan sampling params 相关行为。 它继承自 `SamplingParams`。

### Lines 12-51: supporting statements / 辅助语句
```python
    num_inference_steps: int = 50

    num_frames: int = 125
    height: int = 720
    width: int = 1280
    fps: int = 24

    guidance_scale: float = 1.0

    # HunyuanVideo supported resolutions
    supported_resolutions: list[tuple[int, int]] | None = field(
        default_factory=lambda: [
            # 540p resolutions
            (960, 544),  # 9:16
            (544, 960),  # 16:9
            (832, 624),  # 4:3
            (624, 832),  # 3:4
            (720, 720),  # 1:1
            # 720p resolutions (recommended)
            (1280, 720),  # 9:16
            (720, 1280),  # 16:9
            (832, 1104),  # 4:3
            (1104, 832),  # 3:4
            (960, 960),  # 1:1
        ]
    )

    teacache_params: TeaCacheParams = field(
        default_factory=lambda: TeaCacheParams(
            teacache_thresh=0.15,
            # from https://github.com/ali-vilab/TeaCache/blob/7c10efc4702c6b619f47805f7abe4a7a08085aa0/TeaCache4HunyuanVideo/teacache_sample_video.py#L222
            coefficients=[
                7.33226126e02,
                -4.01131952e02,
                6.75869174e01,
                -3.14987800e00,
                9.61237896e-02,
            ],
        )
    )
```
**EN:** This block gathers supporting statements inside `HunyuanSamplingParams`. It updates names such as `num_inference_steps`, `num_frames`, `height`, `width`, `fps`, and `guidance_scale`. The code collaborates with `field`, and `TeaCacheParams`.
**CN:** 该代码块汇集了位于 `HunyuanSamplingParams` 内部的辅助语句。 它会更新 `num_inference_steps`、`num_frames`、`height`、`width`、`fps` 和 `guidance_scale` 等名称。 代码会与 `field` 和 `TeaCacheParams` 协同工作。

### Lines 55-55: `FastHunyuanSamplingParam` class overview / `FastHunyuanSamplingParam` 类概览
```python
class FastHunyuanSamplingParam(HunyuanSamplingParams):
```
**EN:** This block defines class `FastHunyuanSamplingParam`. It encapsulates fast hunyuan sampling param behavior. It inherits from `HunyuanSamplingParams`.
**CN:** 该代码块定义了类 `FastHunyuanSamplingParam`。 它用于封装 fast hunyuan sampling param 相关行为。 它继承自 `HunyuanSamplingParams`。

### Lines 56-56: supporting statements / 辅助语句
```python
    num_inference_steps: int = 6
```
**EN:** This block gathers supporting statements inside `FastHunyuanSamplingParam`. It updates names such as `num_inference_steps`.
**CN:** 该代码块汇集了位于 `FastHunyuanSamplingParam` 内部的辅助语句。 它会更新 `num_inference_steps` 等名称。

## Key Concepts / 关键概念
- `HunyuanSamplingParams`: Primary class that encapsulates hunyuan sampling params behavior. / 核心类，用于封装 hunyuan sampling params 相关行为。
- `FastHunyuanSamplingParam`: Primary class that encapsulates fast hunyuan sampling param behavior. / 核心类，用于封装 fast hunyuan sampling param 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.configs.sample.teacache`

- **Total lines / 总行数**: 56
