# joy_image.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/pipeline_configs/joy_image.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `JoyImageEditPipelineConfig`, and `joy_image_postprocess_text`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `JoyImageEditPipelineConfig` 和 `joy_image_postprocess_text` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: module setup and imports / 模块初始化与导入
```python
import math
from dataclasses import dataclass, field
from typing import Callable, Tuple

import torch
import torchvision.transforms.functional as TF
from einops import rearrange
from PIL import Image

from sglang.multimodal_gen.configs.models import DiTConfig, EncoderConfig, VAEConfig
from sglang.multimodal_gen.configs.models.dits.joy_image import JoyImageDiTConfig
from sglang.multimodal_gen.configs.models.encoders.qwen3vl import Qwen3VLConfig
from sglang.multimodal_gen.configs.models.vaes import WanVAEConfig
from sglang.multimodal_gen.configs.pipeline_configs.base import (
    ImagePipelineConfig,
    ModelTaskType,
)
```
**EN:** This block establishes the module context and imports `math`, `dataclasses`, `typing`, `torch`, `torchvision.transforms.functional`, and `einops`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `math`、`dataclasses`、`typing`、`torch`、`torchvision.transforms.functional` 和 `einops`。这些依赖为后续实现提供所需符号。

### Lines 20-30: `joy_image_postprocess_text` implementation / `joy_image_postprocess_text` 实现
```python
def joy_image_postprocess_text(
    outputs,
    _text_inputs,
    drop_idx=34,
    max_sequence_length=4096,
):
    last_hidden_states = outputs.hidden_states[-1]
    prompt_embeds = last_hidden_states[:, drop_idx:]
    if max_sequence_length is not None and prompt_embeds.shape[1] > max_sequence_length:
        prompt_embeds = prompt_embeds[:, -max_sequence_length:, :]
    return prompt_embeds
```
**EN:** This block defines function `joy_image_postprocess_text`. It handles joy image postprocess text logic. The implementation branches on conditions. Parameters such as `outputs`, `_text_inputs`, `drop_idx`, and `max_sequence_length` drive the behavior in this section.
**CN:** 该代码块定义了函数 `joy_image_postprocess_text`。 它用于处理 joy image postprocess text 相关逻辑。 实现中包含条件分支。 本段逻辑主要由 `outputs`、`_text_inputs`、`drop_idx` 和 `max_sequence_length` 等参数驱动。

### Lines 34-34: `JoyImageEditPipelineConfig` class overview / `JoyImageEditPipelineConfig` 类概览
```python
class JoyImageEditPipelineConfig(ImagePipelineConfig):
```
**EN:** This block defines class `JoyImageEditPipelineConfig`. It encapsulates joy image edit pipeline config behavior. It inherits from `ImagePipelineConfig`.
**CN:** 该代码块定义了类 `JoyImageEditPipelineConfig`。 它用于封装 joy image edit pipeline config 相关行为。 它继承自 `ImagePipelineConfig`。

### Lines 35-60: supporting statements / 辅助语句
```python
    task_type: ModelTaskType = ModelTaskType.I2I

    dit_config: DiTConfig = field(default_factory=JoyImageDiTConfig)

    vae_config: VAEConfig = field(default_factory=WanVAEConfig)
    vae_tiling: bool = False
    vae_sp: bool = False

    flow_shift: float = 1.5

    # Text encoding stage (Qwen3-VL for both text and image understanding)
    text_encoder_configs: tuple[EncoderConfig, ...] = field(
        default_factory=lambda: (Qwen3VLConfig(),)
    )

    enable_torch_compile: bool = False

    # Precision for each component
    precision: str = "bf16"
    vae_precision: str = "bf16"
    text_encoder_precisions: tuple[str, ...] = field(default_factory=lambda: ("bf16",))
    postprocess_text_funcs: tuple[Callable, ...] = field(
        default_factory=lambda: (joy_image_postprocess_text,)
    )
    prioritize_frame_matching: bool = True
    bucket_configs: list[tuple[int, int, int, int, int]] = field(init=False)
```
**EN:** This block gathers supporting statements inside `JoyImageEditPipelineConfig`. It updates names such as `task_type`, `dit_config`, `vae_config`, `vae_tiling`, `vae_sp`, and `flow_shift`. The code collaborates with `field`, and `Qwen3VLConfig`.
**CN:** 该代码块汇集了位于 `JoyImageEditPipelineConfig` 内部的辅助语句。 它会更新 `task_type`、`dit_config`、`vae_config`、`vae_tiling`、`vae_sp` 和 `flow_shift` 等名称。 代码会与 `field` 和 `Qwen3VLConfig` 协同工作。

### Lines 62-72: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        self.bucket_configs = self.generate_video_image_bucket(
            basesize=1024,
            min_temporal=1,
            max_temporal=1,
            bs_img=8,
            bs_vid=4,
            bs_mimg=8,
            min_items=1,
            max_items=6,
        )
```
**EN:** This block defines method `__post_init__` on `JoyImageEditPipelineConfig`. It post-processes init. Key calls include `self.generate_video_image_bucket`.
**CN:** 该代码块定义了 `JoyImageEditPipelineConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `self.generate_video_image_bucket`。

### Lines 74-77: `slice_noise_pred` implementation / `slice_noise_pred` 实现
```python
    def slice_noise_pred(self, noise, latents):
        # remove noise over input image
        noise = noise[:, : latents.size(1)]
        return noise
```
**EN:** This block defines method `slice_noise_pred` on `JoyImageEditPipelineConfig`. It handles slice noise pred logic. Key calls include `latents.size`. Parameters such as `noise`, and `latents` drive the behavior in this section.
**CN:** 该代码块定义了 `JoyImageEditPipelineConfig` 的方法 `slice_noise_pred`。 它用于处理 slice noise pred 相关逻辑。 关键调用包括 `latents.size`。 本段逻辑主要由 `noise` 和 `latents` 等参数驱动。

### Lines 79-104: `_generate_hw_buckets` implementation / `_generate_hw_buckets` 实现
```python
    def _generate_hw_buckets(
        self,
        base_height=256,
        base_width=256,
        step_width=16,
        step_height=16,
        max_ratio=4.0,
    ) -> list[tuple[int, int, int, int, int]]:
        """Generate dimension buckets based on aspect ratios"""
        buckets = []
        target_pixels = base_height * base_width

        height = target_pixels // step_width
        width = step_width

        while height >= step_height:
            if max(height, width) / min(height, width) <= max_ratio:
                ratio = height / width
                buckets.append((1, 1, 1, height, width))
            # Try to increase width or decrease height
            if height * (width + step_width) <= target_pixels:
                width += step_width
            else:
                height -= step_height

        return buckets
```
**EN:** This block defines method `_generate_hw_buckets` on `JoyImageEditPipelineConfig`. Generate dimension buckets based on aspect ratios Key calls include `buckets.append`, `max`, and `min`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `base_height`, `base_width`, `step_width`, `step_height`, and `max_ratio` drive the behavior in this section.
**CN:** 该代码块定义了 `JoyImageEditPipelineConfig` 的方法 `_generate_hw_buckets`。 它用于生成hw buckets。 关键调用包括 `buckets.append`、`max` 和 `min`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `base_height`、`base_width`、`step_width`、`step_height` 和 `max_ratio` 等参数驱动。

### Lines 106-157: `generate_video_image_bucket` implementation / `generate_video_image_bucket` 实现
```python
    def generate_video_image_bucket(
        self,
        basesize=256,
        min_temporal=65,
        max_temporal=129,
        bs_img=8,
        bs_vid=1,
        bs_mimg=4,
        min_items=1,
        max_items=1,
    ):
        # (batch_size, num_items, num_frames, height, width)
        assert basesize in [
            256,
            512,
            768,
            1024,
        ], f"[generate_video_image_bucket] wrong basesize {basesize}"
        bucket_list = []

        base_bucket_list = self._generate_hw_buckets()
        # image
        for _bucket in base_bucket_list:
            bucket = list(_bucket)
            bucket[0] = bs_img
            bucket_list.append(bucket)
        # video
        for temporal in range(min_temporal, max_temporal + 1, 8):
            for _bucket in base_bucket_list:
                bucket = list(_bucket)
                bs = (max_temporal + 1) // temporal * bs_vid
                bucket[0] = bs
                bucket[2] = temporal
                bucket_list.append(bucket)
        # multiple images
        for num_items in range(min_items, max_items + 1):
            for _bucket in base_bucket_list:
                bucket = list(_bucket)
                bucket[0] = bs_mimg
                bucket[1] = num_items
                bucket_list.append(bucket)
        # spatial resize
        if basesize > 256:
            ratio = basesize // 256

            def resize(bucket, r):
                bucket[-2] *= r
                bucket[-1] *= r
                return bucket

            bucket_list = [resize(bucket, ratio) for bucket in bucket_list]
        return bucket_list
```
**EN:** This block defines method `generate_video_image_bucket` on `JoyImageEditPipelineConfig`. It generates video image bucket. Key calls include `self._generate_hw_buckets`, `range`, `list`, `bucket_list.append`, and `resize`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `basesize`, `min_temporal`, `max_temporal`, `bs_img`, and `bs_vid` drive the behavior in this section.
**CN:** 该代码块定义了 `JoyImageEditPipelineConfig` 的方法 `generate_video_image_bucket`。 它用于生成video image bucket。 关键调用包括 `self._generate_hw_buckets`、`range`、`list`、`bucket_list.append` 和 `resize`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `basesize`、`min_temporal`、`max_temporal`、`bs_img` 和 `bs_vid` 等参数驱动。

### Lines 159-220: `find_best_bucket` implementation / `find_best_bucket` 实现
```python
    def find_best_bucket(
        self, media_shape: tuple[int, int, int, int]
    ) -> tuple[int, int, int, int, int]:
        """
        Find the best matching bucket for given media dimensions.

        Args:
            media_shape: (num_items, num_frames, height, width) of input media

        Returns:
            Best matching bucket as (batch_size, num_items, num_frames, height, width)
        """
        num_items, num_frames, height, width = media_shape
        target_aspect_ratio = height / width

        if num_frames == 1:
            valid_buckets = []
            for bucket in self.bucket_configs:
                if bucket[1] == num_items and bucket[2] == 1:
                    valid_buckets.append(bucket)

            if len(valid_buckets) == 0:
                raise ValueError(f"No image buckets found for shape {media_shape}")

            return min(
                valid_buckets,
                key=lambda bucket: abs((bucket[3] / bucket[4]) - target_aspect_ratio),
            )
        else:
            valid_buckets = []
            for bucket in self.bucket_configs:
                if bucket[1] == num_items and bucket[2] > 1 and bucket[2] <= num_frames:
                    valid_buckets.append(bucket)

            if len(valid_buckets) == 0:
                raise ValueError(f"No video buckets found for shape {media_shape}")

            if self.prioritize_frame_matching:
                max_frame_count = max(bucket[2] for bucket in valid_buckets)
                max_frame_buckets = [
                    bucket for bucket in valid_buckets if bucket[2] == max_frame_count
                ]

                return min(
                    max_frame_buckets,
                    key=lambda bucket: abs(
                        (bucket[3] / bucket[4]) - target_aspect_ratio
                    ),
                )
            else:
                min_ratio_difference = min(
                    abs((bucket[3] / bucket[4]) - target_aspect_ratio)
                    for bucket in valid_buckets
                )
                best_ratio_buckets = [
                    bucket
                    for bucket in valid_buckets
                    if abs((bucket[3] / bucket[4]) - target_aspect_ratio)
                    == min_ratio_difference
                ]

                return max(best_ratio_buckets, key=lambda bucket: bucket[2])
```
**EN:** This block defines method `find_best_bucket` on `JoyImageEditPipelineConfig`. Find the best matching bucket for given media dimensions. Args: media_shape: (num_items, num_frames, height, width) of input media Returns: Best matching bucket as (batch_size, num_items, num_frames, height, width) Key calls include `min`, `len`, `ValueError`, `max`, and `valid_buckets.append`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `media_shape` drive the behavior in this section.
**CN:** 该代码块定义了 `JoyImageEditPipelineConfig` 的方法 `find_best_bucket`。 它用于处理 find best bucket 相关逻辑。 关键调用包括 `min`、`len`、`ValueError`、`max` 和 `valid_buckets.append`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `media_shape` 等参数驱动。

### Lines 222-242: `resize_center_crop` implementation / `resize_center_crop` 实现
```python
    def resize_center_crop(
        self, img: Image.Image, target_size: Tuple[int, int]
    ) -> Image.Image:
        if isinstance(img, list):
            img = img[0]
        w, h = img.size  # PIL (width, height)
        bh, bw = target_size
        if w == bw and h == bh:
            return img

        scale = max(bh / h, bw / w)
        resize_h, resize_w = math.ceil(h * scale), math.ceil(w * scale)

        img = TF.resize(
            img,
            (resize_h, resize_w),
            interpolation=TF.InterpolationMode.BILINEAR,
            antialias=True,
        )
        img = TF.center_crop(img, target_size)
        return img
```
**EN:** This block defines method `resize_center_crop` on `JoyImageEditPipelineConfig`. It handles resize center crop logic. Key calls include `isinstance`, `max`, `TF.resize`, `TF.center_crop`, and `math.ceil`. The implementation branches on conditions. Parameters such as `img`, and `target_size` drive the behavior in this section.
**CN:** 该代码块定义了 `JoyImageEditPipelineConfig` 的方法 `resize_center_crop`。 它用于处理 resize center crop 相关逻辑。 关键调用包括 `isinstance`、`max`、`TF.resize`、`TF.center_crop` 和 `math.ceil`。 实现中包含条件分支。 本段逻辑主要由 `img` 和 `target_size` 等参数驱动。

### Lines 244-248: `preprocess_condition_image` implementation / `preprocess_condition_image` 实现
```python
    def preprocess_condition_image(
        self, img, width, height, _vae_image_processor
    ) -> None:
        target_w, target_h = self.prepare_calculated_size(img)
        return self.resize_center_crop(img, (target_h, target_w)), (target_w, target_h)
```
**EN:** This block defines method `preprocess_condition_image` on `JoyImageEditPipelineConfig`. It handles preprocess condition image logic. Key calls include `self.prepare_calculated_size`, and `self.resize_center_crop`. Parameters such as `img`, `width`, `height`, and `_vae_image_processor` drive the behavior in this section.
**CN:** 该代码块定义了 `JoyImageEditPipelineConfig` 的方法 `preprocess_condition_image`。 它用于处理 preprocess condition image 相关逻辑。 关键调用包括 `self.prepare_calculated_size` 和 `self.resize_center_crop`。 本段逻辑主要由 `img`、`width`、`height` 和 `_vae_image_processor` 等参数驱动。

### Lines 250-277: `get_decode_scale_and_shift` implementation / `get_decode_scale_and_shift` 实现
```python
    def get_decode_scale_and_shift(
        self, device, dtype, vae
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        """Get VAE denormalization scale and shift.

        Args:
            device: Target device
            dtype: Target dtype
            vae: VAE model

        Returns:
            Tuple of (scaling_factor, shift_factor)
        """
        vae_arch_config = self.vae_config.arch_config

        # Create scale factor: 1.0 / std
        scaling_factor = 1.0 / torch.tensor(
            vae_arch_config.latents_std, device=device
        ).view(1, vae_arch_config.z_dim, 1, 1, 1).to(device, dtype)

        # Create shift factor: mean
        shift_factor = (
            torch.tensor(vae_arch_config.latents_mean)
            .view(1, vae_arch_config.z_dim, 1, 1, 1)
            .to(device, dtype)
        )

        return scaling_factor, shift_factor
```
**EN:** This block defines method `get_decode_scale_and_shift` on `JoyImageEditPipelineConfig`. Get VAE denormalization scale and shift. Args: device: Target device dtype: Target dtype vae: VAE model Returns: Tuple of (scaling_factor, shift_factor) Key calls include `torch.tensor.view.to`, `torch.tensor.view`, and `torch.tensor`. Parameters such as `device`, `dtype`, and `vae` drive the behavior in this section.
**CN:** 该代码块定义了 `JoyImageEditPipelineConfig` 的方法 `get_decode_scale_and_shift`。 它用于获取decode scale and shift。 关键调用包括 `torch.tensor.view.to`、`torch.tensor.view` 和 `torch.tensor`。 本段逻辑主要由 `device`、`dtype` 和 `vae` 等参数驱动。

### Lines 279-282: `prepare_calculated_size` implementation / `prepare_calculated_size` 实现
```python
    def prepare_calculated_size(self, img: Image.Image) -> Tuple[int, int]:
        img_h, img_w = img.size[1], img.size[0]  # PIL (w,h)
        bucket = self.find_best_bucket((1, 1, img_h, img_w))
        return bucket[-1], bucket[-2]  # (width, height)
```
**EN:** This block defines method `prepare_calculated_size` on `JoyImageEditPipelineConfig`. It prepares calculated size. Key calls include `self.find_best_bucket`. Parameters such as `img` drive the behavior in this section.
**CN:** 该代码块定义了 `JoyImageEditPipelineConfig` 的方法 `prepare_calculated_size`。 它用于准备calculated size。 关键调用包括 `self.find_best_bucket`。 本段逻辑主要由 `img` 等参数驱动。

### Lines 284-318: `prepare_image_processor_kwargs` implementation / `prepare_image_processor_kwargs` 实现
```python
    def prepare_image_processor_kwargs(self, batch, neg=False) -> dict:
        prompt = batch.prompt if not neg else batch.negative_prompt
        if prompt is None:
            return {}
        prompt_list = [prompt] if isinstance(prompt, str) else prompt
        image_list = batch.condition_image
        if image_list is None:
            image_list = []
        elif not isinstance(image_list, list):
            image_list = [image_list]

        if len(prompt_list) <= 1:
            per_prompt_images = [image_list]
        elif len(image_list) <= 1:
            per_prompt_images = [list(image_list) for _ in prompt_list]
        elif len(image_list) == len(prompt_list):
            per_prompt_images = [[image] for image in image_list]
        else:
            raise ValueError(
                "JoyImageEdit expects either one shared condition image or "
                "the same number of condition images and prompts."
            )

        prompt_template_encode = (
            "<|im_start|>system\n \\nDescribe the image by detailing the color, shape, size,"
            " texture, quantity, text, spatial relationships of the objects and background:<|im_end|>\n"
            "<|im_start|>user\n{}<|im_end|>\n"
            "<|im_start|>assistant\n"
        )
        img_prompt_template = "<|vision_start|><|image_pad|><|vision_end|>"
        txt = []
        for p, prompt_images in zip(prompt_list, per_prompt_images):
            base_img_prompt = img_prompt_template * len(prompt_images)
            txt.append(prompt_template_encode.format(base_img_prompt + p))
        return dict(text=txt, padding=True, per_prompt_images=per_prompt_images)
```
**EN:** This block defines method `prepare_image_processor_kwargs` on `JoyImageEditPipelineConfig`. It prepares image processor kwargs. Key calls include `zip`, `dict`, `isinstance`, `len`, and `txt.append`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `batch`, and `neg` drive the behavior in this section.
**CN:** 该代码块定义了 `JoyImageEditPipelineConfig` 的方法 `prepare_image_processor_kwargs`。 它用于准备image processor kwargs。 关键调用包括 `zip`、`dict`、`isinstance`、`len` 和 `txt.append`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `batch` 和 `neg` 等参数驱动。

### Lines 320-340: `prepare_latent_shape` implementation / `prepare_latent_shape` 实现
```python
    def prepare_latent_shape(self, batch, batch_size: int, num_frames: int) -> Tuple:
        """Prepare latent shape for I2I generation with multi-item support.

        Args:
            batch: The request batch
            batch_size: Batch size
            num_frames: Number of frames (1 for image)

        Returns:
            Tuple representing latent shape
        """

        shape = (
            batch_size,
            self.vae_config.arch_config.z_dim,  # 16 for WanxVAE
            1,
            int(batch.height) // self.vae_config.arch_config.scale_factor_spatial,
            int(batch.width) // self.vae_config.arch_config.scale_factor_spatial,
        )

        return shape
```
**EN:** This block defines method `prepare_latent_shape` on `JoyImageEditPipelineConfig`. Prepare latent shape for I2I generation with multi-item support. Args: batch: The request batch batch_size: Batch size num_frames: Number of frames (1 for image) Returns: Tuple representing latent shape Key calls include `int`. Parameters such as `batch`, `batch_size`, and `num_frames` drive the behavior in this section.
**CN:** 该代码块定义了 `JoyImageEditPipelineConfig` 的方法 `prepare_latent_shape`。 它用于准备latent shape。 关键调用包括 `int`。 本段逻辑主要由 `batch`、`batch_size` 和 `num_frames` 等参数驱动。

### Lines 342-382: `postprocess_image_latent` implementation / `postprocess_image_latent` 实现
```python
    def postprocess_image_latent(self, latent_condition, batch):
        if latent_condition.dim() == 4:
            latent_condition = latent_condition.unsqueeze(0)
        elif latent_condition.dim() != 5:
            raise ValueError(
                f"Expected 4D/5D condition latents, but got shape {latent_condition.shape}"
            )

        batch_size = int(batch.batch_size)
        cond_batch = int(latent_condition.shape[0])
        if batch_size > cond_batch:
            if batch_size % cond_batch != 0:
                raise ValueError(
                    f"Cannot duplicate condition image latents from batch size {cond_batch} "
                    f"to target batch size {batch_size}."
                )
            repeat_factor = batch_size // cond_batch
            latent_condition = latent_condition.repeat(repeat_factor, 1, 1, 1, 1)
        elif batch_size < cond_batch:
            raise ValueError(
                f"Condition image latents batch size {cond_batch} exceeds target batch size {batch_size}."
            )
        _, _, t, h, w = latent_condition.shape
        pt, ph, pw = self.dit_config.arch_config.patch_size
        condition_size = (t // pt, h // ph, w // pw)

        if batch.vae_image_sizes is None:
            batch.vae_image_sizes = [condition_size]
        else:
            # ImageVAEEncodingStage iterates condition images in input order.
            # Keep the same order in vae_image_sizes for RoPE range construction.
            batch.vae_image_sizes = batch.vae_image_sizes + [condition_size]

        latents = rearrange(
            latent_condition,
            "b c (t pt) (h ph) (w pw) -> b (t h w) c pt ph pw",
            pt=pt,
            ph=ph,
            pw=pw,
        )
        return latents
```
**EN:** This block defines method `postprocess_image_latent` on `JoyImageEditPipelineConfig`. It handles postprocess image latent logic. Key calls include `int`, `rearrange`, `latent_condition.dim`, `latent_condition.unsqueeze`, and `latent_condition.repeat`. The implementation branches on conditions. Parameters such as `latent_condition`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `JoyImageEditPipelineConfig` 的方法 `postprocess_image_latent`。 它用于处理 postprocess image latent 相关逻辑。 关键调用包括 `int`、`rearrange`、`latent_condition.dim`、`latent_condition.unsqueeze` 和 `latent_condition.repeat`。 实现中包含条件分支。 本段逻辑主要由 `latent_condition` 和 `batch` 等参数驱动。

### Lines 384-409: `maybe_pack_latents` implementation / `maybe_pack_latents` 实现
```python
    def maybe_pack_latents(self, latents, batch_size, batch):
        if latents.dim() == 4:
            latents = latents.unsqueeze(0)
        elif latents.dim() != 5:
            raise ValueError(f"Expected 4D/5D latents, but got shape {latents.shape}")

        _, _, t, h, w = latents.shape
        pt, ph, pw = self.dit_config.arch_config.patch_size
        if batch.vae_image_sizes is None:
            batch.vae_image_sizes = [(t // pt, h // ph, w // pw)]
        else:
            # LatentPreparationStage packs noisy latents after condition latents were packed
            # in ImageVAEEncodingStage. Denoising concatenates as [noisy, condition...],
            # so keep noisy size at index 0.
            batch.vae_image_sizes = [
                (t // pt, h // ph, w // pw)
            ] + batch.vae_image_sizes
        latents = rearrange(
            latents,
            "b c (t pt) (h ph) (w pw) -> b (t h w) c pt ph pw",
            pt=pt,
            ph=ph,
            pw=pw,
        )

        return latents
```
**EN:** This block defines method `maybe_pack_latents` on `JoyImageEditPipelineConfig`. It handles maybe pack latents logic. Key calls include `rearrange`, `latents.dim`, `latents.unsqueeze`, and `ValueError`. The implementation branches on conditions. Parameters such as `latents`, `batch_size`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `JoyImageEditPipelineConfig` 的方法 `maybe_pack_latents`。 它用于处理 maybe pack latents 相关逻辑。 关键调用包括 `rearrange`、`latents.dim`、`latents.unsqueeze` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `latents`、`batch_size` 和 `batch` 等参数驱动。

### Lines 411-421: `post_denoising_loop` implementation / `post_denoising_loop` 实现
```python
    def post_denoising_loop(self, latents, batch):
        lt, lh, lw = batch.vae_image_sizes[0]
        target_len = lt * lh * lw
        target_patches = latents[:, :target_len]
        return rearrange(
            target_patches,
            "b (t h w) c pt ph pw -> b c (t pt) (h ph) (w pw)",
            t=lt,
            h=lh,
            w=lw,
        )
```
**EN:** This block defines method `post_denoising_loop` on `JoyImageEditPipelineConfig`. It post-processes denoising loop. Key calls include `rearrange`. Parameters such as `latents`, and `batch` drive the behavior in this section.
**CN:** 该代码块定义了 `JoyImageEditPipelineConfig` 的方法 `post_denoising_loop`。 它用于后处理denoising loop。 关键调用包括 `rearrange`。 本段逻辑主要由 `latents` 和 `batch` 等参数驱动。

### Lines 423-431: `postprocess_cfg_noise` implementation / `postprocess_cfg_noise` 实现
```python
    def postprocess_cfg_noise(
        self,
        batch,
        noise_pred: torch.Tensor,
        noise_pred_cond: torch.Tensor,
    ) -> torch.Tensor:
        cond_norm = torch.norm(noise_pred_cond, dim=2, keepdim=True)
        noise_norm = torch.norm(noise_pred, dim=2, keepdim=True).clamp_min(1e-12)
        return noise_pred * (cond_norm / noise_norm)
```
**EN:** This block defines method `postprocess_cfg_noise` on `JoyImageEditPipelineConfig`. It handles postprocess cfg noise logic. Key calls include `torch.norm`, and `torch.norm.clamp_min`. Parameters such as `batch`, `noise_pred`, and `noise_pred_cond` drive the behavior in this section.
**CN:** 该代码块定义了 `JoyImageEditPipelineConfig` 的方法 `postprocess_cfg_noise`。 它用于处理 postprocess cfg noise 相关逻辑。 关键调用包括 `torch.norm` 和 `torch.norm.clamp_min`。 本段逻辑主要由 `batch`、`noise_pred` 和 `noise_pred_cond` 等参数驱动。

## Key Concepts / 关键概念
- `joy_image_postprocess_text`: Top-level function that handles joy image postprocess text logic. / 顶层函数，用于处理 joy image postprocess text 相关逻辑。
- `JoyImageEditPipelineConfig`: Primary class that encapsulates joy image edit pipeline config behavior. / 核心类，用于封装 joy image edit pipeline config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `math`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`, `torchvision.transforms.functional`, `einops`, `PIL`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models`, `sglang.multimodal_gen.configs.models.dits.joy_image`, `sglang.multimodal_gen.configs.models.encoders.qwen3vl`, `sglang.multimodal_gen.configs.models.vaes`, `sglang.multimodal_gen.configs.pipeline_configs.base`

- **Total lines / 总行数**: 431
