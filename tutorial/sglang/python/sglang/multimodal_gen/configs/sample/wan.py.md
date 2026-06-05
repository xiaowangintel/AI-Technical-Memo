# wan.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/sample/wan.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `WanT2V_1_3B_SamplingParams`, `WanT2V_14B_SamplingParams`, and `WanI2V_14B_480P_SamplingParam`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `WanT2V_1_3B_SamplingParams`、`WanT2V_14B_SamplingParams` 和 `WanI2V_14B_480P_SamplingParam` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-7: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field

from sglang.multimodal_gen.configs.sample.sampling_params import SamplingParams
from sglang.multimodal_gen.configs.sample.teacache import TeaCacheParams
```
**EN:** This block establishes the module context and imports `dataclasses`, `sglang.multimodal_gen.configs.sample.sampling_params`, and `sglang.multimodal_gen.configs.sample.teacache`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`sglang.multimodal_gen.configs.sample.sampling_params` 和 `sglang.multimodal_gen.configs.sample.teacache`。这些依赖为后续实现提供所需符号。

### Lines 10-27: `_wan_1_3b_coefficients` implementation / `_wan_1_3b_coefficients` 实现
```python
def _wan_1_3b_coefficients(p: TeaCacheParams) -> list[float]:
    if p.use_ret_steps:
        # from https://github.com/ali-vilab/TeaCache/blob/7c10efc4702c6b619f47805f7abe4a7a08085aa0/TeaCache4Wan2.1/teacache_generate.py#L883
        return [
            -5.21862437e04,
            9.23041404e03,
            -5.28275948e02,
            1.36987616e01,
            -4.99875664e-02,
        ]
    # from https://github.com/ali-vilab/TeaCache/blob/7c10efc4702c6b619f47805f7abe4a7a08085aa0/TeaCache4Wan2.1/teacache_generate.py#L890
    return [
        2.39676752e03,
        -1.31110545e03,
        2.01331979e02,
        -8.29855975e00,
        1.37887774e-01,
    ]
```
**EN:** This block defines function `_wan_1_3b_coefficients`. It handles wan 1 3b coefficients logic. The implementation branches on conditions. Parameters such as `p` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_wan_1_3b_coefficients`。 它用于处理 wan 1 3b coefficients 相关逻辑。 实现中包含条件分支。 本段逻辑主要由 `p` 等参数驱动。

### Lines 30-41: `_wan_14b_coefficients` implementation / `_wan_14b_coefficients` 实现
```python
def _wan_14b_coefficients(p: TeaCacheParams) -> list[float]:
    if p.use_ret_steps:
        # from https://github.com/ali-vilab/TeaCache/blob/7c10efc4702c6b619f47805f7abe4a7a08085aa0/TeaCache4Wan2.1/teacache_generate.py#L885
        return [
            -3.03318725e05,
            4.90537029e04,
            -2.65530556e03,
            5.87365115e01,
            -3.15583525e-01,
        ]
    # from https://github.com/ali-vilab/TeaCache/blob/7c10efc4702c6b619f47805f7abe4a7a08085aa0/TeaCache4Wan2.1/teacache_generate.py#L892
    return [-5784.54975374, 5449.50911966, -1811.16591783, 256.27178429, -13.02252404]
```
**EN:** This block defines function `_wan_14b_coefficients`. It handles wan 14b coefficients logic. The implementation branches on conditions. Parameters such as `p` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_wan_14b_coefficients`。 它用于处理 wan 14b coefficients 相关逻辑。 实现中包含条件分支。 本段逻辑主要由 `p` 等参数驱动。

### Lines 45-46: `WanT2V_1_3B_SamplingParams` class overview / `WanT2V_1_3B_SamplingParams` 类概览
```python
class WanT2V_1_3B_SamplingParams(SamplingParams):
    # Video parameters
```
**EN:** This block defines class `WanT2V_1_3B_SamplingParams`. It encapsulates wan t2 v 1 3 b sampling params behavior. It inherits from `SamplingParams`.
**CN:** 该代码块定义了类 `WanT2V_1_3B_SamplingParams`。 它用于封装 wan t2 v 1 3 b sampling params 相关行为。 它继承自 `SamplingParams`。

### Lines 47-75: supporting statements / 辅助语句
```python
    height: int = 480
    width: int = 832
    num_frames: int = 81
    fps: int = 16

    # Denoising stage
    guidance_scale: float = 3.0
    negative_prompt: str = (
        "Bright tones, overexposed, static, blurred details, subtitles, style, works, paintings, images, static, overall gray, worst quality, low quality, JPEG compression residue, ugly, incomplete, extra fingers, poorly drawn hands, poorly drawn faces, deformed, disfigured, misshapen limbs, fused fingers, still picture, messy background, three legs, many people in the background, walking backwards"
    )
    num_inference_steps: int = 50

    # Wan T2V 1.3B supported resolutions
    supported_resolutions: list[tuple[int, int]] | None = field(
        default_factory=lambda: [
            (832, 480),  # 16:9
            (480, 832),  # 9:16
        ]
    )

    teacache_params: TeaCacheParams = field(
        default_factory=lambda: TeaCacheParams(
            teacache_thresh=0.08,
            use_ret_steps=True,
            coefficients_callback=_wan_1_3b_coefficients,
            start_skipping=5,
            end_skipping=1.0,
        )
    )
```
**EN:** This block gathers supporting statements inside `WanT2V_1_3B_SamplingParams`. It updates names such as `height`, `width`, `num_frames`, `fps`, `guidance_scale`, and `negative_prompt`. The code collaborates with `field`, and `TeaCacheParams`.
**CN:** 该代码块汇集了位于 `WanT2V_1_3B_SamplingParams` 内部的辅助语句。 它会更新 `height`、`width`、`num_frames`、`fps`、`guidance_scale` 和 `negative_prompt` 等名称。 代码会与 `field` 和 `TeaCacheParams` 协同工作。

### Lines 79-80: `WanT2V_14B_SamplingParams` class overview / `WanT2V_14B_SamplingParams` 类概览
```python
class WanT2V_14B_SamplingParams(SamplingParams):
    # Video parameters
```
**EN:** This block defines class `WanT2V_14B_SamplingParams`. It encapsulates wan t2 v 14 b sampling params behavior. It inherits from `SamplingParams`.
**CN:** 该代码块定义了类 `WanT2V_14B_SamplingParams`。 它用于封装 wan t2 v 14 b sampling params 相关行为。 它继承自 `SamplingParams`。

### Lines 81-111: supporting statements / 辅助语句
```python
    height: int = 720
    width: int = 1280
    num_frames: int = 81
    fps: int = 16

    # Denoising stage
    guidance_scale: float = 5.0
    negative_prompt: str = (
        "Bright tones, overexposed, static, blurred details, subtitles, style, works, paintings, images, static, overall gray, worst quality, low quality, JPEG compression residue, ugly, incomplete, extra fingers, poorly drawn hands, poorly drawn faces, deformed, disfigured, misshapen limbs, fused fingers, still picture, messy background, three legs, many people in the background, walking backwards"
    )
    num_inference_steps: int = 50

    # Wan T2V 14B supported resolutions
    supported_resolutions: list[tuple[int, int]] | None = field(
        default_factory=lambda: [
            (1280, 720),  # 16:9
            (720, 1280),  # 9:16
            (832, 480),  # 16:9
            (480, 832),  # 9:16
        ]
    )

    teacache_params: TeaCacheParams = field(
        default_factory=lambda: TeaCacheParams(
            teacache_thresh=0.20,
            use_ret_steps=False,
            coefficients_callback=_wan_14b_coefficients,
            start_skipping=1,
            end_skipping=-1,
        )
    )
```
**EN:** This block gathers supporting statements inside `WanT2V_14B_SamplingParams`. It updates names such as `height`, `width`, `num_frames`, `fps`, `guidance_scale`, and `negative_prompt`. The code collaborates with `field`, and `TeaCacheParams`.
**CN:** 该代码块汇集了位于 `WanT2V_14B_SamplingParams` 内部的辅助语句。 它会更新 `height`、`width`、`num_frames`、`fps`、`guidance_scale` 和 `negative_prompt` 等名称。 代码会与 `field` 和 `TeaCacheParams` 协同工作。

### Lines 115-116: `WanI2V_14B_480P_SamplingParam` class overview / `WanI2V_14B_480P_SamplingParam` 类概览
```python
class WanI2V_14B_480P_SamplingParam(WanT2V_1_3B_SamplingParams):
    # Denoising stage
```
**EN:** This block defines class `WanI2V_14B_480P_SamplingParam`. It encapsulates wan i2 v 14 b 480 p sampling param behavior. It inherits from `WanT2V_1_3B_SamplingParams`.
**CN:** 该代码块定义了类 `WanI2V_14B_480P_SamplingParam`。 它用于封装 wan i2 v 14 b 480 p sampling param 相关行为。 它继承自 `WanT2V_1_3B_SamplingParams`。

### Lines 117-137: supporting statements / 辅助语句
```python
    guidance_scale: float = 5.0
    num_inference_steps: int = 50
    # num_inference_steps: int = 40

    # Wan I2V 480P supported resolutions (override parent)
    supported_resolutions: list[tuple[int, int]] | None = field(
        default_factory=lambda: [
            (832, 480),  # 16:9
            (480, 832),  # 9:16
        ]
    )

    teacache_params: TeaCacheParams = field(
        default_factory=lambda: TeaCacheParams(
            teacache_thresh=0.26,
            use_ret_steps=True,
            coefficients_callback=_wan_14b_coefficients,
            start_skipping=5,
            end_skipping=1.0,
        )
    )
```
**EN:** This block gathers supporting statements inside `WanI2V_14B_480P_SamplingParam`. It updates names such as `guidance_scale`, `num_inference_steps`, `supported_resolutions`, and `teacache_params`. The code collaborates with `field`, and `TeaCacheParams`.
**CN:** 该代码块汇集了位于 `WanI2V_14B_480P_SamplingParam` 内部的辅助语句。 它会更新 `guidance_scale`、`num_inference_steps`、`supported_resolutions` 和 `teacache_params` 等名称。 代码会与 `field` 和 `TeaCacheParams` 协同工作。

### Lines 141-142: `WanI2V_14B_720P_SamplingParam` class overview / `WanI2V_14B_720P_SamplingParam` 类概览
```python
class WanI2V_14B_720P_SamplingParam(WanT2V_14B_SamplingParams):
    # Denoising stage
```
**EN:** This block defines class `WanI2V_14B_720P_SamplingParam`. It encapsulates wan i2 v 14 b 720 p sampling param behavior. It inherits from `WanT2V_14B_SamplingParams`.
**CN:** 该代码块定义了类 `WanI2V_14B_720P_SamplingParam`。 它用于封装 wan i2 v 14 b 720 p sampling param 相关行为。 它继承自 `WanT2V_14B_SamplingParams`。

### Lines 143-165: supporting statements / 辅助语句
```python
    guidance_scale: float = 5.0
    num_inference_steps: int = 50
    # num_inference_steps: int = 40

    # Wan I2V 720P supported resolutions (override parent)
    supported_resolutions: list[tuple[int, int]] | None = field(
        default_factory=lambda: [
            (1280, 720),  # 16:9
            (720, 1280),  # 9:16
            (832, 480),  # 16:9
            (480, 832),  # 9:16
        ]
    )

    teacache_params: TeaCacheParams = field(
        default_factory=lambda: TeaCacheParams(
            teacache_thresh=0.3,
            use_ret_steps=True,
            coefficients_callback=_wan_14b_coefficients,
            start_skipping=5,
            end_skipping=1.0,
        )
    )
```
**EN:** This block gathers supporting statements inside `WanI2V_14B_720P_SamplingParam`. It updates names such as `guidance_scale`, `num_inference_steps`, `supported_resolutions`, and `teacache_params`. The code collaborates with `field`, and `TeaCacheParams`.
**CN:** 该代码块汇集了位于 `WanI2V_14B_720P_SamplingParam` 内部的辅助语句。 它会更新 `guidance_scale`、`num_inference_steps`、`supported_resolutions` 和 `teacache_params` 等名称。 代码会与 `field` 和 `TeaCacheParams` 协同工作。

### Lines 169-171: `FastWanT2V480PConfig` class overview / `FastWanT2V480PConfig` 类概览
```python
class FastWanT2V480PConfig(WanT2V_1_3B_SamplingParams):
    # DMD parameters
    # dmd_denoising_steps: list[int] | None = field(default_factory=lambda: [1000, 757, 522])
```
**EN:** This block defines class `FastWanT2V480PConfig`. It encapsulates fast wan t2 v480 pconfig behavior. It inherits from `WanT2V_1_3B_SamplingParams`.
**CN:** 该代码块定义了类 `FastWanT2V480PConfig`。 它用于封装 fast wan t2 v480 pconfig 相关行为。 它继承自 `WanT2V_1_3B_SamplingParams`。

### Lines 172-176: supporting statements / 辅助语句
```python
    num_inference_steps: int = 3
    num_frames: int = 61
    height: int = 448
    width: int = 832
    fps: int = 16
```
**EN:** This block gathers supporting statements inside `FastWanT2V480PConfig`. It updates names such as `num_inference_steps`, `num_frames`, `height`, `width`, and `fps`.
**CN:** 该代码块汇集了位于 `FastWanT2V480PConfig` 内部的辅助语句。 它会更新 `num_inference_steps`、`num_frames`、`height`、`width` 和 `fps` 等名称。

### Lines 183-185: `Wan2_1_Fun_1_3B_InP_SamplingParams` class overview / `Wan2_1_Fun_1_3B_InP_SamplingParams` 类概览
```python
class Wan2_1_Fun_1_3B_InP_SamplingParams(SamplingParams):
    """Sampling parameters for Wan2.1 Fun 1.3B InP model."""
```
**EN:** This block defines class `Wan2_1_Fun_1_3B_InP_SamplingParams`. Sampling parameters for Wan2.1 Fun 1.3B InP model. It inherits from `SamplingParams`.
**CN:** 该代码块定义了类 `Wan2_1_Fun_1_3B_InP_SamplingParams`。 它用于封装 wan2 1 fun 1 3 b in p sampling params 相关行为。 它继承自 `SamplingParams`。

### Lines 186-194: supporting statements / 辅助语句
```python
    height: int = 480
    width: int = 832
    num_frames: int = 81
    fps: int = 16
    negative_prompt: str | None = (
        "色调艳丽，过曝，静态，细节模糊不清，字幕，风格，作品，画作，画面，静止，整体发灰，最差质量，低质量，JPEG压缩残留，丑陋的，残缺的，多余的手指，画得不好的手部，画得不好的脸部，畸形的，毁容的，形态畸形的肢体，手指融合，静止不动的画面，杂乱的背景，三条腿，背景人很多，倒着走"
    )
    guidance_scale: float = 6.0
    num_inference_steps: int = 50
```
**EN:** This block gathers supporting statements inside `Wan2_1_Fun_1_3B_InP_SamplingParams`. It updates names such as `height`, `width`, `num_frames`, `fps`, `negative_prompt`, and `guidance_scale`.
**CN:** 该代码块汇集了位于 `Wan2_1_Fun_1_3B_InP_SamplingParams` 内部的辅助语句。 它会更新 `height`、`width`、`num_frames`、`fps`、`negative_prompt` 和 `guidance_scale` 等名称。

### Lines 201-203: `Wan2_2_Base_SamplingParams` class overview / `Wan2_2_Base_SamplingParams` 类概览
```python
class Wan2_2_Base_SamplingParams(SamplingParams):
    """Sampling parameters for Wan2.2 TI2V 5B model."""
```
**EN:** This block defines class `Wan2_2_Base_SamplingParams`. Sampling parameters for Wan2.2 TI2V 5B model. It inherits from `SamplingParams`.
**CN:** 该代码块定义了类 `Wan2_2_Base_SamplingParams`。 它用于封装 wan2 2 base sampling params 相关行为。 它继承自 `SamplingParams`。

### Lines 204-206: supporting statements / 辅助语句
```python
    negative_prompt: str | None = (
        "色调艳丽，过曝，静态，细节模糊不清，字幕，风格，作品，画作，画面，静止，整体发灰，最差质量，低质量，JPEG压缩残留，丑陋的，残缺的，多余的手指，画得不好的手部，画得不好的脸部，畸形的，毁容的，形态畸形的肢体，手指融合，静止不动的画面，杂乱的背景，三条腿，背景人很多，倒着走"
    )
```
**EN:** This block gathers supporting statements inside `Wan2_2_Base_SamplingParams`. It updates names such as `negative_prompt`.
**CN:** 该代码块汇集了位于 `Wan2_2_Base_SamplingParams` 内部的辅助语句。 它会更新 `negative_prompt` 等名称。

### Lines 215-217: `Wan2_2_TI2V_5B_SamplingParam` class overview / `Wan2_2_TI2V_5B_SamplingParam` 类概览
```python
class Wan2_2_TI2V_5B_SamplingParam(Wan2_2_Base_SamplingParams):
    """Sampling parameters for Wan2.2 TI2V 5B model."""
```
**EN:** This block defines class `Wan2_2_TI2V_5B_SamplingParam`. Sampling parameters for Wan2.2 TI2V 5B model. It inherits from `Wan2_2_Base_SamplingParams`.
**CN:** 该代码块定义了类 `Wan2_2_TI2V_5B_SamplingParam`。 它用于封装 wan2 2 ti2 v 5 b sampling param 相关行为。 它继承自 `Wan2_2_Base_SamplingParams`。

### Lines 218-231: supporting statements / 辅助语句
```python
    height: int = 704
    width: int = 1280
    num_frames: int = 121
    fps: int = 24
    guidance_scale: float = 5.0
    num_inference_steps: int = 50

    # Wan2.2 TI2V 5B supported resolutions
    supported_resolutions: list[tuple[int, int]] | None = field(
        default_factory=lambda: [
            (1280, 704),  # 16:9-ish
            (704, 1280),  # 9:16-ish
        ]
    )
```
**EN:** This block gathers supporting statements inside `Wan2_2_TI2V_5B_SamplingParam`. It updates names such as `height`, `width`, `num_frames`, `fps`, `guidance_scale`, and `num_inference_steps`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `Wan2_2_TI2V_5B_SamplingParam` 内部的辅助语句。 它会更新 `height`、`width`、`num_frames`、`fps`、`guidance_scale` 和 `num_inference_steps` 等名称。 代码会与 `field` 协同工作。

### Lines 235-235: `Wan2_2_T2V_A14B_SamplingParam` class overview / `Wan2_2_T2V_A14B_SamplingParam` 类概览
```python
class Wan2_2_T2V_A14B_SamplingParam(Wan2_2_Base_SamplingParams):
```
**EN:** This block defines class `Wan2_2_T2V_A14B_SamplingParam`. It encapsulates wan2 2 t2 v a14 b sampling param behavior. It inherits from `Wan2_2_Base_SamplingParams`.
**CN:** 该代码块定义了类 `Wan2_2_T2V_A14B_SamplingParam`。 它用于封装 wan2 2 t2 v a14 b sampling param 相关行为。 它继承自 `Wan2_2_Base_SamplingParams`。

### Lines 236-251: supporting statements / 辅助语句
```python
    guidance_scale: float = 4.0  # high_noise
    guidance_scale_2: float = 3.0  # low_noise
    num_inference_steps: int = 40
    fps: int = 16

    num_frames: int = 81

    # Wan2.2 T2V A14B supported resolutions
    supported_resolutions: list[tuple[int, int]] | None = field(
        default_factory=lambda: [
            (1280, 720),  # 16:9
            (720, 1280),  # 9:16
            (832, 480),  # 16:9
            (480, 832),  # 9:16
        ]
    )
```
**EN:** This block gathers supporting statements inside `Wan2_2_T2V_A14B_SamplingParam`. It updates names such as `guidance_scale`, `guidance_scale_2`, `num_inference_steps`, `fps`, `num_frames`, and `supported_resolutions`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `Wan2_2_T2V_A14B_SamplingParam` 内部的辅助语句。 它会更新 `guidance_scale`、`guidance_scale_2`、`num_inference_steps`、`fps`、`num_frames` 和 `supported_resolutions` 等名称。 代码会与 `field` 协同工作。

### Lines 255-255: `Wan2_2_I2V_A14B_SamplingParam` class overview / `Wan2_2_I2V_A14B_SamplingParam` 类概览
```python
class Wan2_2_I2V_A14B_SamplingParam(Wan2_2_Base_SamplingParams):
```
**EN:** This block defines class `Wan2_2_I2V_A14B_SamplingParam`. It encapsulates wan2 2 i2 v a14 b sampling param behavior. It inherits from `Wan2_2_Base_SamplingParams`.
**CN:** 该代码块定义了类 `Wan2_2_I2V_A14B_SamplingParam`。 它用于封装 wan2 2 i2 v a14 b sampling param 相关行为。 它继承自 `Wan2_2_Base_SamplingParams`。

### Lines 256-271: supporting statements / 辅助语句
```python
    guidance_scale: float = 3.5  # high_noise
    guidance_scale_2: float = 3.5  # low_noise
    num_inference_steps: int = 40
    fps: int = 16

    num_frames: int = 81

    # Wan2.2 I2V A14B supported resolutions
    supported_resolutions: list[tuple[int, int]] | None = field(
        default_factory=lambda: [
            (1280, 720),  # 16:9
            (720, 1280),  # 9:16
            (832, 480),  # 16:9
            (480, 832),  # 9:16
        ]
    )
```
**EN:** This block gathers supporting statements inside `Wan2_2_I2V_A14B_SamplingParam`. It updates names such as `guidance_scale`, `guidance_scale_2`, `num_inference_steps`, `fps`, `num_frames`, and `supported_resolutions`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `Wan2_2_I2V_A14B_SamplingParam` 内部的辅助语句。 它会更新 `guidance_scale`、`guidance_scale_2`、`num_inference_steps`、`fps`、`num_frames` 和 `supported_resolutions` 等名称。 代码会与 `field` 协同工作。

### Lines 275-275: `Turbo_Wan2_2_I2V_A14B_SamplingParam` class overview / `Turbo_Wan2_2_I2V_A14B_SamplingParam` 类概览
```python
class Turbo_Wan2_2_I2V_A14B_SamplingParam(Wan2_2_Base_SamplingParams):
```
**EN:** This block defines class `Turbo_Wan2_2_I2V_A14B_SamplingParam`. It encapsulates turbo wan2 2 i2 v a14 b sampling param behavior. It inherits from `Wan2_2_Base_SamplingParams`.
**CN:** 该代码块定义了类 `Turbo_Wan2_2_I2V_A14B_SamplingParam`。 它用于封装 turbo wan2 2 i2 v a14 b sampling param 相关行为。 它继承自 `Wan2_2_Base_SamplingParams`。

### Lines 276-279: supporting statements / 辅助语句
```python
    guidance_scale: float = 3.5  # high_noise
    guidance_scale_2: float = 3.5  # low_noise
    num_inference_steps: int = 4
    fps: int = 16
```
**EN:** This block gathers supporting statements inside `Turbo_Wan2_2_I2V_A14B_SamplingParam`. It updates names such as `guidance_scale`, `guidance_scale_2`, `num_inference_steps`, and `fps`.
**CN:** 该代码块汇集了位于 `Turbo_Wan2_2_I2V_A14B_SamplingParam` 内部的辅助语句。 它会更新 `guidance_scale`、`guidance_scale_2`、`num_inference_steps` 和 `fps` 等名称。

### Lines 286-286: `SelfForcingWanT2V480PConfig` class overview / `SelfForcingWanT2V480PConfig` 类概览
```python
class SelfForcingWanT2V480PConfig(WanT2V_1_3B_SamplingParams):
```
**EN:** This block defines class `SelfForcingWanT2V480PConfig`. It encapsulates self forcing wan t2 v480 pconfig behavior. It inherits from `WanT2V_1_3B_SamplingParams`.
**CN:** 该代码块定义了类 `SelfForcingWanT2V480PConfig`。 它用于封装 self forcing wan t2 v480 pconfig 相关行为。 它继承自 `WanT2V_1_3B_SamplingParams`。

### Lines 287-287: supporting statements / 辅助语句
```python
    pass
```
**EN:** This block gathers supporting statements inside `SelfForcingWanT2V480PConfig`.
**CN:** 该代码块汇集了位于 `SelfForcingWanT2V480PConfig` 内部的辅助语句。

## Key Concepts / 关键概念
- `_wan_1_3b_coefficients`: Top-level function that handles wan 1 3b coefficients logic. / 顶层函数，用于处理 wan 1 3b coefficients 相关逻辑。
- `_wan_14b_coefficients`: Top-level function that handles wan 14b coefficients logic. / 顶层函数，用于处理 wan 14b coefficients 相关逻辑。
- `WanT2V_1_3B_SamplingParams`: Primary class that encapsulates wan t2 v 1 3 b sampling params behavior. / 核心类，用于封装 wan t2 v 1 3 b sampling params 相关行为。
- `WanT2V_14B_SamplingParams`: Primary class that encapsulates wan t2 v 14 b sampling params behavior. / 核心类，用于封装 wan t2 v 14 b sampling params 相关行为。
- `WanI2V_14B_480P_SamplingParam`: Primary class that encapsulates wan i2 v 14 b 480 p sampling param behavior. / 核心类，用于封装 wan i2 v 14 b 480 p sampling param 相关行为。
- `WanI2V_14B_720P_SamplingParam`: Primary class that encapsulates wan i2 v 14 b 720 p sampling param behavior. / 核心类，用于封装 wan i2 v 14 b 720 p sampling param 相关行为。
- `FastWanT2V480PConfig`: Primary class that encapsulates fast wan t2 v480 pconfig behavior. / 核心类，用于封装 fast wan t2 v480 pconfig 相关行为。
- `Wan2_1_Fun_1_3B_InP_SamplingParams`: Sampling parameters for Wan2.1 Fun 1.3B InP model. / 核心类，用于封装 wan2 1 fun 1 3 b in p sampling params 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.configs.sample.teacache`

- **Total lines / 总行数**: 287
