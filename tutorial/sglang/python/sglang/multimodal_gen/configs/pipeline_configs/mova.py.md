# mova.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/pipeline_configs/mova.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `MOVAPipelineConfig`, `MOVA360PConfig`, and `MOVA720PConfig`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: MOVA pipeline configuration. / 该文件属于配置层。它围绕 `MOVAPipelineConfig`、`MOVA360PConfig` 和 `MOVA720PConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-24: module setup and imports / 模块初始化与导入
```python
"""
MOVA pipeline configuration.
"""

from dataclasses import dataclass, field

import numpy as np
import torch
import torch.nn.functional as F
from PIL import Image

from sglang.multimodal_gen.configs.models.dits import MOVAAudioConfig, MOVAVideoConfig
from sglang.multimodal_gen.configs.models.encoders import T5Config
from sglang.multimodal_gen.configs.models.vaes import DacVAEConfig, WanVAEConfig
from sglang.multimodal_gen.configs.pipeline_configs.base import (
    ModelTaskType,
    PipelineConfig,
)
from sglang.multimodal_gen.configs.pipeline_configs.model_deployment_config import (
    ModelDeploymentConfig,
)
from sglang.multimodal_gen.configs.pipeline_configs.wan import t5_postprocess_text
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `dataclasses`, `numpy`, `torch`, `torch.nn.functional`, `PIL`, and `sglang.multimodal_gen.configs.models.dits`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`numpy`、`torch`、`torch.nn.functional`、`PIL` 和 `sglang.multimodal_gen.configs.models.dits`。这些依赖为后续实现提供所需符号。

### Lines 26-26: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 30-32: `MOVAPipelineConfig` class overview / `MOVAPipelineConfig` 类概览
```python
class MOVAPipelineConfig(PipelineConfig):
    """Configuration for MOVA (text+image -> video+audio) pipelines."""
```
**EN:** This block defines class `MOVAPipelineConfig`. Configuration for MOVA (text+image -> video+audio) pipelines. It inherits from `PipelineConfig`.
**CN:** 该代码块定义了类 `MOVAPipelineConfig`。 它用于封装 movapipeline config 相关行为。 它继承自 `PipelineConfig`。

### Lines 33-57: supporting statements / 辅助语句
```python
    task_type: ModelTaskType = ModelTaskType.I2V

    # Model configs
    dit_config: MOVAVideoConfig = field(default_factory=MOVAVideoConfig)
    audio_dit_config: MOVAAudioConfig = field(default_factory=MOVAAudioConfig)

    # Video VAE (Wan) + Audio VAE (DAC)
    vae_config: WanVAEConfig = field(default_factory=WanVAEConfig)
    vae_precision: str = "bf16"
    audio_vae_config: DacVAEConfig = field(default_factory=DacVAEConfig)
    audio_vae_precision: str = "bf16"

    # Text encoder (UMT5 compatible)
    text_encoder_configs: tuple = field(default_factory=lambda: (T5Config(),))
    postprocess_text_funcs: tuple = field(
        default_factory=lambda: (t5_postprocess_text,)
    )

    # MOVA specific
    audio_vae_type: str = "dac"
    boundary_ratio: float | None = 0.9

    # temporal alignment: MOVA expects (num_frames - 1) % 4 == 0
    time_division_factor: int = 4
    time_division_remainder: int = 1
```
**EN:** This block gathers supporting statements inside `MOVAPipelineConfig`. It updates names such as `task_type`, `dit_config`, `audio_dit_config`, `vae_config`, `vae_precision`, and `audio_vae_config`. The code collaborates with `field`, and `T5Config`.
**CN:** 该代码块汇集了位于 `MOVAPipelineConfig` 内部的辅助语句。 它会更新 `task_type`、`dit_config`、`audio_dit_config`、`vae_config`、`vae_precision` 和 `audio_vae_config` 等名称。 代码会与 `field` 和 `T5Config` 协同工作。

### Lines 59-63: `get_model_deployment_config` implementation / `get_model_deployment_config` 实现
```python
    def get_model_deployment_config(self) -> ModelDeploymentConfig:
        return ModelDeploymentConfig(
            auto_dit_layerwise_offload=True,
            auto_dit_layerwise_offload_high_memory_disable_gb=130,
        )
```
**EN:** This block defines method `get_model_deployment_config` on `MOVAPipelineConfig`. It retrieves model deployment config. Key calls include `ModelDeploymentConfig`.
**CN:** 该代码块定义了 `MOVAPipelineConfig` 的方法 `get_model_deployment_config`。 它用于获取model deployment config。 关键调用包括 `ModelDeploymentConfig`。

### Lines 65-112: `_center_crop_and_resize` implementation / `_center_crop_and_resize` 实现
```python
    def _center_crop_and_resize(
        self, image: torch.Tensor | Image.Image, target_height: int, target_width: int
    ) -> torch.Tensor | Image.Image:
        if not isinstance(image, (Image.Image, torch.Tensor)):
            raise TypeError(f"Unsupported image type: {type(image)}")
        if isinstance(image, Image.Image):
            image = torch.from_numpy(np.array(image))

        if image.ndim == 2:
            image = image[..., None]

        if not image.dtype.is_floating_point:
            image = image.to(torch.float32).div(255.0)

        if image.ndim == 3:
            if image.shape[0] in (1, 3, 4) and image.shape[-1] not in (1, 3, 4):
                image = image.unsqueeze(0)
            else:
                image = image.permute(2, 0, 1).unsqueeze(0)
        elif image.ndim == 4:
            if image.shape[1] not in (1, 3, 4) and image.shape[-1] in (1, 3, 4):
                image = image.permute(0, 3, 1, 2)

        image_height, image_width = image.shape[-2], image.shape[-1]
        if image_height == target_height and image_width == target_width:
            return image

        logger.info(
            "Center cropping and resizing image to %dx%d", target_width, target_height
        )

        if image_height * target_width < image_width * target_height:
            cropped_width = (image_height * target_width) // target_height
            left = (image_width - cropped_width) // 2
            image = image[..., :, left : left + cropped_width]
        else:
            cropped_height = (image_width * target_height) // target_width
            top = (image_height - cropped_height) // 2
            image = image[..., top : top + cropped_height, :]

        image = F.interpolate(
            image,
            size=(target_height, target_width),
            mode="bilinear",
            align_corners=False,
            antialias=True,
        )
        return image
```
**EN:** This block defines method `_center_crop_and_resize` on `MOVAPipelineConfig`. It handles center crop and resize logic. Key calls include `isinstance`, `logger.info`, `F.interpolate`, `TypeError`, and `torch.from_numpy`. The implementation branches on conditions. Parameters such as `image`, `target_height`, and `target_width` drive the behavior in this section.
**CN:** 该代码块定义了 `MOVAPipelineConfig` 的方法 `_center_crop_and_resize`。 它用于处理 center crop and resize 相关逻辑。 关键调用包括 `isinstance`、`logger.info`、`F.interpolate`、`TypeError` 和 `torch.from_numpy`。 实现中包含条件分支。 本段逻辑主要由 `image`、`target_height` 和 `target_width` 等参数驱动。

### Lines 114-131: `adjust_num_frames` implementation / `adjust_num_frames` 实现
```python
    def adjust_num_frames(self, num_frames: int) -> int:
        if num_frames is None:
            return num_frames
        if num_frames % self.time_division_factor != self.time_division_remainder:
            adjusted = (
                (num_frames + self.time_division_factor - 1)
                // self.time_division_factor
                * self.time_division_factor
                + self.time_division_remainder
            )
            logger.warning(
                "`num_frames` (%s) is not compatible with MOVA temporal constraints. "
                "Rounding to %s.",
                num_frames,
                adjusted,
            )
            return adjusted
        return num_frames
```
**EN:** This block defines method `adjust_num_frames` on `MOVAPipelineConfig`. It handles adjust num frames logic. Key calls include `logger.warning`. The implementation branches on conditions. Parameters such as `num_frames` drive the behavior in this section.
**CN:** 该代码块定义了 `MOVAPipelineConfig` 的方法 `adjust_num_frames`。 它用于处理 adjust num frames 相关逻辑。 关键调用包括 `logger.warning`。 实现中包含条件分支。 本段逻辑主要由 `num_frames` 等参数驱动。

### Lines 133-137: `preprocess_condition_image` implementation / `preprocess_condition_image` 实现
```python
    def preprocess_condition_image(
        self, image, target_width, target_height, _vae_image_processor
    ):
        image = self._center_crop_and_resize(image, target_height, target_width)
        return image, (target_width, target_height)
```
**EN:** This block defines method `preprocess_condition_image` on `MOVAPipelineConfig`. It handles preprocess condition image logic. Key calls include `self._center_crop_and_resize`. Parameters such as `image`, `target_width`, `target_height`, and `_vae_image_processor` drive the behavior in this section.
**CN:** 该代码块定义了 `MOVAPipelineConfig` 的方法 `preprocess_condition_image`。 它用于处理 preprocess condition image 相关逻辑。 关键调用包括 `self._center_crop_and_resize`。 本段逻辑主要由 `image`、`target_width`、`target_height` 和 `_vae_image_processor` 等参数驱动。

### Lines 139-149: `prepare_latent_shape` implementation / `prepare_latent_shape` 实现
```python
    def prepare_latent_shape(self, batch, batch_size, num_frames):
        spatial = self.vae_config.arch_config.spatial_compression_ratio
        length = (num_frames - 1) // self.time_division_factor + 1
        shape = (
            batch_size,
            self.dit_config.arch_config.out_dim,
            length,
            batch.height // spatial,
            batch.width // spatial,
        )
        return shape
```
**EN:** This block defines method `prepare_latent_shape` on `MOVAPipelineConfig`. It prepares latent shape. Parameters such as `batch`, `batch_size`, and `num_frames` drive the behavior in this section.
**CN:** 该代码块定义了 `MOVAPipelineConfig` 的方法 `prepare_latent_shape`。 它用于准备latent shape。 本段逻辑主要由 `batch`、`batch_size` 和 `num_frames` 等参数驱动。

### Lines 151-153: `prepare_audio_latent_shape` implementation / `prepare_audio_latent_shape` 实现
```python
    def prepare_audio_latent_shape(self, batch_size, num_samples, audio_vae):
        latent_T = (num_samples + audio_vae.hop_length - 1) // audio_vae.hop_length
        return (batch_size, audio_vae.latent_dim, latent_T)
```
**EN:** This block defines method `prepare_audio_latent_shape` on `MOVAPipelineConfig`. It prepares audio latent shape. Parameters such as `batch_size`, `num_samples`, and `audio_vae` drive the behavior in this section.
**CN:** 该代码块定义了 `MOVAPipelineConfig` 的方法 `prepare_audio_latent_shape`。 它用于准备audio latent shape。 本段逻辑主要由 `batch_size`、`num_samples` 和 `audio_vae` 等参数驱动。

### Lines 155-166: `normalize_video_latents` implementation / `normalize_video_latents` 实现
```python
    def normalize_video_latents(self, latents: torch.Tensor, video_vae) -> torch.Tensor:
        latents_mean = getattr(video_vae.config, "latents_mean", None)
        latents_std = getattr(video_vae.config, "latents_std", None)
        if latents_mean is None or latents_std is None:
            return latents
        mean = torch.tensor(
            latents_mean, device=latents.device, dtype=latents.dtype
        ).view(1, video_vae.config.z_dim, 1, 1, 1)
        inv_std = (
            1.0 / torch.tensor(latents_std, device=latents.device, dtype=latents.dtype)
        ).view(1, video_vae.config.z_dim, 1, 1, 1)
        return (latents - mean) * inv_std
```
**EN:** This block defines method `normalize_video_latents` on `MOVAPipelineConfig`. It handles normalize video latents logic. Key calls include `getattr`, `torch.tensor.view`, `view`, and `torch.tensor`. The implementation branches on conditions. Parameters such as `latents`, and `video_vae` drive the behavior in this section.
**CN:** 该代码块定义了 `MOVAPipelineConfig` 的方法 `normalize_video_latents`。 它用于处理 normalize video latents 相关逻辑。 关键调用包括 `getattr`、`torch.tensor.view`、`view` 和 `torch.tensor`。 实现中包含条件分支。 本段逻辑主要由 `latents` 和 `video_vae` 等参数驱动。

### Lines 168-181: `denormalize_video_latents` implementation / `denormalize_video_latents` 实现
```python
    def denormalize_video_latents(
        self, latents: torch.Tensor, video_vae
    ) -> torch.Tensor:
        latents_mean = getattr(video_vae.config, "latents_mean", None)
        latents_std = getattr(video_vae.config, "latents_std", None)
        if latents_mean is None or latents_std is None:
            return latents
        mean = torch.tensor(
            latents_mean, device=latents.device, dtype=latents.dtype
        ).view(1, video_vae.config.z_dim, 1, 1, 1)
        std = torch.tensor(
            latents_std, device=latents.device, dtype=latents.dtype
        ).view(1, video_vae.config.z_dim, 1, 1, 1)
        return latents * std + mean
```
**EN:** This block defines method `denormalize_video_latents` on `MOVAPipelineConfig`. It handles denormalize video latents logic. Key calls include `getattr`, `torch.tensor.view`, and `torch.tensor`. The implementation branches on conditions. Parameters such as `latents`, and `video_vae` drive the behavior in this section.
**CN:** 该代码块定义了 `MOVAPipelineConfig` 的方法 `denormalize_video_latents`。 它用于处理 denormalize video latents 相关逻辑。 关键调用包括 `getattr`、`torch.tensor.view` 和 `torch.tensor`。 实现中包含条件分支。 本段逻辑主要由 `latents` 和 `video_vae` 等参数驱动。

### Lines 185-187: `MOVA360PConfig` class overview / `MOVA360PConfig` 类概览
```python
class MOVA360PConfig(MOVAPipelineConfig):
    """Configuration for MOVA 360P (text+image -> video+audio) pipelines."""
```
**EN:** This block defines class `MOVA360PConfig`. Configuration for MOVA 360P (text+image -> video+audio) pipelines. It inherits from `MOVAPipelineConfig`.
**CN:** 该代码块定义了类 `MOVA360PConfig`。 它用于封装 mova360 pconfig 相关行为。 它继承自 `MOVAPipelineConfig`。

### Lines 188-188: supporting statements / 辅助语句
```python
    max_area: int = 352 * 640
```
**EN:** This block gathers supporting statements inside `MOVA360PConfig`. It updates names such as `max_area`.
**CN:** 该代码块汇集了位于 `MOVA360PConfig` 内部的辅助语句。 它会更新 `max_area` 等名称。

### Lines 192-194: `MOVA720PConfig` class overview / `MOVA720PConfig` 类概览
```python
class MOVA720PConfig(MOVAPipelineConfig):
    """Configuration for MOVA 720P (text+image -> video+audio) pipelines."""
```
**EN:** This block defines class `MOVA720PConfig`. Configuration for MOVA 720P (text+image -> video+audio) pipelines. It inherits from `MOVAPipelineConfig`.
**CN:** 该代码块定义了类 `MOVA720PConfig`。 它用于封装 mova720 pconfig 相关行为。 它继承自 `MOVAPipelineConfig`。

### Lines 195-195: supporting statements / 辅助语句
```python
    max_area: int = 720 * 1280
```
**EN:** This block gathers supporting statements inside `MOVA720PConfig`. It updates names such as `max_area`.
**CN:** 该代码块汇集了位于 `MOVA720PConfig` 内部的辅助语句。 它会更新 `max_area` 等名称。

## Key Concepts / 关键概念
- `MOVAPipelineConfig`: Configuration for MOVA (text+image -> video+audio) pipelines. / 核心类，用于封装 movapipeline config 相关行为。
- `MOVA360PConfig`: Configuration for MOVA 360P (text+image -> video+audio) pipelines. / 核心类，用于封装 mova360 pconfig 相关行为。
- `MOVA720PConfig`: Configuration for MOVA 720P (text+image -> video+audio) pipelines. / 核心类，用于封装 mova720 pconfig 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Third-party / 第三方依赖**: `numpy`, `torch`, `torch.nn.functional`, `PIL`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.dits`, `sglang.multimodal_gen.configs.models.encoders`, `sglang.multimodal_gen.configs.models.vaes`, `sglang.multimodal_gen.configs.pipeline_configs.base`, `sglang.multimodal_gen.configs.pipeline_configs.model_deployment_config`, `sglang.multimodal_gen.configs.pipeline_configs.wan`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 195
