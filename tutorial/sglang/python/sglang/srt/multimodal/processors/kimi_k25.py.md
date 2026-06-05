# kimi_k25.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/kimi_k25.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for kimi k25 models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 kimi k25 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21: Imports dependencies
```python
import math
import re
from collections import defaultdict
from typing import Dict, List, Union

import numpy as np
import torch
import torch.nn.functional as F
from PIL import Image

from sglang.srt.managers.schedule_batch import (
    MultimodalProcessorOutput,
)
from sglang.srt.models.kimi_k25 import KimiK25ForConditionalGeneration
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor as SGLangBaseProcessor,
)
from sglang.srt.multimodal.processors.base_processor import (
    MultimodalSpecialTokens,
)
from sglang.srt.multimodal.processors.kimi_common import KimiGridMMDataMixin
```
**EN:** This block groups related imports for the module, including math, re, collections.defaultdict, typing.Dict, typing.List, and 10 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 math, re, collections.defaultdict, typing.Dict, typing.List 等 10 项，为后续代码准备所需名称。

### Lines 22-27: Comments and module notes
```python

# ---------------------------------------------------------------------------
# GPU image preprocessing utilities (resize, pad, normalize, patchify on CUDA)
# ---------------------------------------------------------------------------


```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 28-68: Defines function navit_resize_config
```python
def navit_resize_config(
    width: int,
    height: int,
    patch_size: int,
    merge_kernel_size: int,
    in_patch_limit: int,
    patch_limit_on_one_side: int,
    fixed_output_tokens: int | None = None,
) -> dict:
    """Compute NaViT resize target dimensions and token count.

    Pure math -- no image data needed, only (width, height).
    """
    s1 = math.sqrt(
        in_patch_limit
        / (max(1.0, width // patch_size) * max(1.0, height // patch_size))
    )
    s2 = patch_limit_on_one_side * patch_size / width
    s3 = patch_limit_on_one_side * patch_size / height
    scale = min(1.0, s1, s2, s3)
    new_w = min(max(1, int(width * scale)), patch_limit_on_one_side * patch_size)
    new_h = min(max(1, int(height * scale)), patch_limit_on_one_side * patch_size)

    factor = merge_kernel_size * patch_size
    pad_height = (factor - new_h % factor) % factor
    pad_width = (factor - new_w % factor) % factor

    if fixed_output_tokens is not None:
        num_tokens = fixed_output_tokens
    else:
        token_height = (new_h + pad_height) // factor
        token_width = (new_w + pad_width) // factor
        num_tokens = token_height * token_width

    return {
        "num_tokens": num_tokens,
        "new_width": new_w,
        "new_height": new_h,
        "pad_width": pad_width,
        "pad_height": pad_height,
    }
```
**EN:** This block defines function `navit_resize_config`. Parameters: width, height, patch_size, merge_kernel_size, in_patch_limit, patch_limit_on_one_side, fixed_output_tokens. Compute NaViT resize target dimensions and token count. Pure math -- no image data needed, only (width, height).
**CN:** 该代码块定义函数 `navit_resize_config`。 参数包括 width、height、patch_size、merge_kernel_size、in_patch_limit、patch_limit_on_one_side、fixed_output_tokens。 文档字符串摘要：Compute NaViT resize target dimensions and token count. Pure math -- no image data needed, only (width, height).

### Lines 69-70: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 71-76: Defines function _get_image_dimensions
```python
def _get_image_dimensions(image: Union[torch.Tensor, Image.Image]) -> tuple[int, int]:
    """Get (width, height) from a CUDA tensor or PIL Image."""
    if isinstance(image, torch.Tensor):
        # nvJPEG returns (C, H, W) uint8
        return image.shape[2], image.shape[1]
    return image.size  # PIL returns (width, height)
```
**EN:** This block defines function `_get_image_dimensions`. Parameters: image. Get (width, height) from a CUDA tensor or PIL Image.
**CN:** 该代码块定义函数 `_get_image_dimensions`。 参数包括 image。 文档字符串摘要：Get (width, height) from a CUDA tensor or PIL Image.

### Lines 77-78: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 79-82: Defines function _pil_to_cuda_chw
```python
def _pil_to_cuda_chw(image: Image.Image) -> torch.Tensor:
    """Convert PIL Image to (C, H, W) uint8 CUDA tensor."""
    arr = np.asarray(image.convert("RGB"))
    return torch.from_numpy(arr).permute(2, 0, 1).cuda()
```
**EN:** This block defines function `_pil_to_cuda_chw`. Parameters: image. Convert PIL Image to (C, H, W) uint8 CUDA tensor.
**CN:** 该代码块定义函数 `_pil_to_cuda_chw`。 参数包括 image。 文档字符串摘要：Convert PIL Image to (C, H, W) uint8 CUDA tensor.

### Lines 83-84: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 85-115: Defines function _process_single_image
```python
def _process_single_image(
    image: Union[torch.Tensor, Image.Image],
    config: dict,
    image_mean: torch.Tensor,
    image_std_inv: torch.Tensor,
    patch_size: int,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Process a single image on GPU: resize -> pad -> normalize -> patchify."""
    if isinstance(image, Image.Image):
        image = _pil_to_cuda_chw(image)

    new_h, new_w = config["new_height"], config["new_width"]
    pad_h, pad_w = config["pad_height"], config["pad_width"]

    x = image.unsqueeze(0).float()
    x = F.interpolate(x, size=(new_h, new_w), mode="bicubic", align_corners=False)

    if pad_h > 0 or pad_w > 0:
        x = F.pad(x, (0, pad_w, 0, pad_h), value=0.0)

    x = x / 255.0
    x = (x - image_mean) * image_std_inv

    _, C, H, W = x.shape
    T = 1
    gh, gw = H // patch_size, W // patch_size
    x = x.view(T, C, gh, patch_size, gw, patch_size)
    x = x.permute(0, 2, 4, 1, 3, 5).reshape(-1, C, patch_size, patch_size)

    grid_thw = torch.tensor([T, gh, gw], dtype=torch.int64, device=x.device)
    return x, grid_thw
```
**EN:** This block defines function `_process_single_image`. Parameters: image, config, image_mean, image_std_inv, patch_size. Process a single image on GPU: resize -> pad -> normalize -> patchify.
**CN:** 该代码块定义函数 `_process_single_image`。 参数包括 image、config、image_mean、image_std_inv、patch_size。 文档字符串摘要：Process a single image on GPU: resize -> pad -> normalize -> patchify.

### Lines 116-117: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 118-194: Defines function _gpu_preprocess_images
```python
def _gpu_preprocess_images(
    images: list[Union[torch.Tensor, Image.Image]],
    resize_configs: list[dict],
    image_mean: torch.Tensor,
    image_std_inv: torch.Tensor,
    patch_size: int,
) -> tuple[torch.Tensor, torch.Tensor]:
    """GPU preprocessing pipeline for a batch of images.

    Groups images with the same target padded size for batch processing.
    """
    n = len(images)
    if n == 0:
        device = image_mean.device
        return (
            torch.empty(0, 3, patch_size, patch_size, device=device),
            torch.empty(0, 3, dtype=torch.int64, device=device),
        )

    groups = defaultdict(list)
    for idx, (image, config) in enumerate(zip(images, resize_configs)):
        padded_h = config["new_height"] + config["pad_height"]
        padded_w = config["new_width"] + config["pad_width"]
        target_h = config["new_height"]
        target_w = config["new_width"]
        groups[(target_h, target_w, padded_h, padded_w)].append((idx, image, config))

    all_patches = [None] * n
    all_grids = [None] * n

    for (target_h, target_w, padded_h, padded_w), group in groups.items():
        if len(group) == 1:
            idx, image, config = group[0]
            patches, grid = _process_single_image(
                image, config, image_mean, image_std_inv, patch_size
            )
            all_patches[idx] = patches
            all_grids[idx] = grid
        else:
            tensors = []
            for _, image, _ in group:
                if isinstance(image, Image.Image):
                    image = _pil_to_cuda_chw(image)
                tensors.append(image.unsqueeze(0).float())

            resized = []
            for t in tensors:
                r = F.interpolate(
                    t, size=(target_h, target_w), mode="bicubic", align_corners=False
                )
                resized.append(r)
            batch = torch.cat(resized, dim=0)

            pad_h = padded_h - target_h
            pad_w = padded_w - target_w
            if pad_h > 0 or pad_w > 0:
                batch = F.pad(batch, (0, pad_w, 0, pad_h), value=0.0)

            batch = batch / 255.0
            batch = (batch - image_mean) * image_std_inv

            B, C, H, W = batch.shape
            T = 1
            gh, gw = H // patch_size, W // patch_size
            batch = batch.view(B, C, gh, patch_size, gw, patch_size)
            batch = batch.permute(0, 2, 4, 1, 3, 5).reshape(
                B, -1, C, patch_size, patch_size
            )

            grid = torch.tensor([T, gh, gw], dtype=torch.int64, device=batch.device)
            for i, (idx, _, _) in enumerate(group):
                all_patches[idx] = batch[i]
                all_grids[idx] = grid

    pixel_values = torch.cat(all_patches, dim=0)
    grid_thws = torch.stack(all_grids, dim=0)
    return pixel_values, grid_thws
```
**EN:** This block defines function `_gpu_preprocess_images`. Parameters: images, resize_configs, image_mean, image_std_inv, patch_size. GPU preprocessing pipeline for a batch of images. Groups images with the same target padded size for batch processing.
**CN:** 该代码块定义函数 `_gpu_preprocess_images`。 参数包括 images、resize_configs、image_mean、image_std_inv、patch_size。 文档字符串摘要：GPU preprocessing pipeline for a batch of images. Groups images with the same target padded size for batch processing.

### Lines 195-201: Comments and module notes
```python


# ---------------------------------------------------------------------------
# Kimi K2.5 GPU processor wrapper
# ---------------------------------------------------------------------------


```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 202-202: Declares class KimiGPUProcessorWrapper
```python
class KimiGPUProcessorWrapper:
```
**EN:** This block introduces class `KimiGPUProcessorWrapper` as a reusable abstraction inside the module. Wraps Kimi's HF processor to do GPU image preprocessing. GPU path: nvJPEG CUDA tensor / PIL -> _gpu_preprocess_images() CPU fallback: PIL -> medias kwarg -> original HF KimiK25Processor.__call__ Exposes attributes that base class's process_mm_data needs so it
**CN:** 该代码块声明类 `KimiGPUProcessorWrapper`，作为模块中的可复用抽象。 文档字符串摘要：Wraps Kimi's HF processor to do GPU image preprocessing. GPU path: nvJPEG CUDA tensor / PIL -> _gpu_preprocess_images() CPU fallback: PIL -> medias kwarg -> original HF KimiK25Processor.__call__ Exposes attributes that base class's process_mm_data needs so it

### Lines 203-210: Documents the scope
```python
    """Wraps Kimi's HF processor to do GPU image preprocessing.

    GPU path: nvJPEG CUDA tensor / PIL -> _gpu_preprocess_images()
    CPU fallback: PIL -> medias kwarg -> original HF KimiK25Processor.__call__

    Exposes attributes that base class's process_mm_data needs so it behaves
    like a normal HF processor from the outside.
    """
```
**EN:** This string literal serves as documentation for the KimiGPUProcessorWrapper, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 KimiGPUProcessorWrapper 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 211-211: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiGPUProcessorWrapper, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiGPUProcessorWrapper 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 212-241: Defines function KimiGPUProcessorWrapper.__init__
```python
    def __init__(
        self,
        hf_processor,
        image_token,
        patch_size,
        merge_kernel_size,
        in_patch_limit,
        patch_limit_on_one_side,
        fixed_output_tokens,
        image_mean,
        image_std,
    ):
        self._hf_processor = hf_processor
        self._image_token = image_token
        self._patch_size = patch_size
        self._merge_kernel_size = merge_kernel_size
        self._in_patch_limit = in_patch_limit
        self._patch_limit_on_one_side = patch_limit_on_one_side
        self._fixed_output_tokens = fixed_output_tokens
        self._image_mean = image_mean
        self._image_std = image_std
        self._gpu_norm_tensors = None

        # Explicitly expose attributes that base class process_mm_data needs:
        # - image_processor: checked via isinstance(..., BaseImageProcessor)
        # - tokenizer: used for tokenization
        # - media_processor: used by CPU fallback path
        self.image_processor = hf_processor.image_processor
        self.tokenizer = hf_processor.tokenizer
        self.media_processor = hf_processor.media_processor
```
**EN:** This block defines function `KimiGPUProcessorWrapper.__init__`. Parameters: self, hf_processor, image_token, patch_size, merge_kernel_size, in_patch_limit, patch_limit_on_one_side, fixed_output_tokens, image_mean, image_std.
**CN:** 该代码块定义函数 `KimiGPUProcessorWrapper.__init__`。 参数包括 self、hf_processor、image_token、patch_size、merge_kernel_size、in_patch_limit、patch_limit_on_one_side、fixed_output_tokens、image_mean、image_std。

### Lines 242-242: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiGPUProcessorWrapper, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiGPUProcessorWrapper 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 243-249: Defines function KimiGPUProcessorWrapper.__call__
```python
    def __call__(self, text=None, images=None, **kwargs):
        # process_mm_data passes images via kwargs["images"]
        images = images or kwargs.pop("images", None)

        if images and torch.cuda.is_available():
            return self._gpu_call(text, images)
        return self._cpu_call(text, images, **kwargs)
```
**EN:** This block defines function `KimiGPUProcessorWrapper.__call__`. Parameters: self, text, images.
**CN:** 该代码块定义函数 `KimiGPUProcessorWrapper.__call__`。 参数包括 self、text、images。

### Lines 250-250: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiGPUProcessorWrapper, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiGPUProcessorWrapper 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 251-295: Defines function KimiGPUProcessorWrapper._gpu_call
```python
    def _gpu_call(self, text, images):
        """Bypass HF KimiK25VisionProcessor.preprocess entirely -- use GPU ops."""
        input_text = text[0] if isinstance(text, list) else text

        # 1. Compute resize configs (CPU math)
        resize_configs = []
        for image in images:
            w, h = _get_image_dimensions(image)
            resize_configs.append(
                navit_resize_config(
                    w,
                    h,
                    self._patch_size,
                    self._merge_kernel_size,
                    self._in_patch_limit,
                    self._patch_limit_on_one_side,
                    self._fixed_output_tokens,
                )
            )

        # 2. Expand image tokens
        parts = input_text.split(self._image_token)
        result = [parts[0]]
        for config, part in zip(resize_configs, parts[1:]):
            result.append(self._image_token * config["num_tokens"] + part)
        input_text = "".join(result)

        # 3. Tokenize
        text_inputs = self._hf_processor.tokenizer(input_text, return_tensors="pt")

        # 4. GPU image preprocessing
        image_mean, image_std_inv = self._get_gpu_norm_tensors()
        pixel_values, grid_thws = _gpu_preprocess_images(
            images, resize_configs, image_mean, image_std_inv, self._patch_size
        )

        grid_thws = grid_thws.cpu()

        return {
            "input_ids": text_inputs["input_ids"],
            "pixel_values": pixel_values,
            # Use SGL-standard key so get_new_expanded_mm_items() can split
            # per-image for cache granularity (it looks up 'image_grid_thw').
            "image_grid_thw": grid_thws,
        }
```
**EN:** This block defines function `KimiGPUProcessorWrapper._gpu_call`. Parameters: self, text, images. Bypass HF KimiK25VisionProcessor.preprocess entirely -- use GPU ops.
**CN:** 该代码块定义函数 `KimiGPUProcessorWrapper._gpu_call`。 参数包括 self、text、images。 文档字符串摘要：Bypass HF KimiK25VisionProcessor.preprocess entirely -- use GPU ops.

### Lines 296-296: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiGPUProcessorWrapper, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiGPUProcessorWrapper 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 297-319: Defines function KimiGPUProcessorWrapper._cpu_call
```python
    def _cpu_call(self, text, images, **kwargs):
        """Fallback: token expansion + medias kwarg -> original HF processor."""
        input_text = text[0] if isinstance(text, list) else text

        if images:
            # Token expansion via media_tokens_calculator
            parts = input_text.split(self._image_token)
            result = [parts[0]]
            for image, part in zip(images, parts[1:]):
                num_tokens = self._hf_processor.media_processor.media_tokens_calculator(
                    {"type": "image", "image": image}
                )
                result.append(self._image_token * num_tokens + part)
            input_text = "".join(result)

            # Convert to medias format for Kimi's HF processor
            kwargs["medias"] = [{"type": "image", "image": img} for img in images]

        out = self._hf_processor(text=[input_text], **kwargs)
        grid_thws = out.pop("grid_thws", None)
        if grid_thws is not None:
            out["image_grid_thw"] = grid_thws
        return out
```
**EN:** This block defines function `KimiGPUProcessorWrapper._cpu_call`. Parameters: self, text, images. Fallback: token expansion + medias kwarg -> original HF processor.
**CN:** 该代码块定义函数 `KimiGPUProcessorWrapper._cpu_call`。 参数包括 self、text、images。 文档字符串摘要：Fallback: token expansion + medias kwarg -> original HF processor.

### Lines 320-320: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiGPUProcessorWrapper, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiGPUProcessorWrapper 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 321-330: Defines function KimiGPUProcessorWrapper._get_gpu_norm_tensors
```python
    def _get_gpu_norm_tensors(self, device="cuda"):
        if self._gpu_norm_tensors is None:
            image_mean = torch.tensor(
                self._image_mean, device=device, dtype=torch.float32
            ).view(1, 3, 1, 1)
            image_std_inv = (
                1.0 / torch.tensor(self._image_std, device=device, dtype=torch.float32)
            ).view(1, 3, 1, 1)
            self._gpu_norm_tensors = (image_mean, image_std_inv)
        return self._gpu_norm_tensors
```
**EN:** This block defines function `KimiGPUProcessorWrapper._get_gpu_norm_tensors`. Parameters: self, device.
**CN:** 该代码块定义函数 `KimiGPUProcessorWrapper._get_gpu_norm_tensors`。 参数包括 self、device。

### Lines 331-338: Comments and module notes
```python


# ---------------------------------------------------------------------------
# Kimi K2.5 SGLang multimodal processor
# ---------------------------------------------------------------------------


# Compatible with KimiVLForConditionalGeneration
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 339-339: Declares class KimiK2_5VLImageProcessor
```python
class KimiK2_5VLImageProcessor(KimiGridMMDataMixin, SGLangBaseProcessor):
```
**EN:** This block introduces class `KimiK2_5VLImageProcessor` as a reusable abstraction inside the module. It inherits from KimiGridMMDataMixin, SGLangBaseProcessor.
**CN:** 该代码块声明类 `KimiK2_5VLImageProcessor`，作为模块中的可复用抽象。 它继承自 KimiGridMMDataMixin、SGLangBaseProcessor。

### Lines 340-341: Declares models, gpu_image_decode
```python
    models = [KimiK25ForConditionalGeneration]
    gpu_image_decode = True  # nvJPEG for JPEG, PIL fallback for others
```
**EN:** This block initializes a related set of values in the KimiK2_5VLImageProcessor, including models, gpu_image_decode. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 KimiK2_5VLImageProcessor 中初始化一组相关值，包括 models, gpu_image_decode。将这些赋值集中在一起有助于理解周边配置。

### Lines 342-342: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiK2_5VLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiK2_5VLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 343-366: Defines function KimiK2_5VLImageProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)
        self.mm_tokens = MultimodalSpecialTokens(
            image_token="<|media_pad|>",
            # TODO: could we convert in MultimodalSpecialTokens?
            image_token_id=hf_config.media_placeholder_token_id,
            image_token_regex=re.compile(r"(?:<\|media_pad\|>)+"),
        ).build(_processor)

        # Extract media processing config from HF processor
        media_proc_cfg = _processor.media_processor.media_proc_cfg

        # Replace with GPU-capable wrapper
        self._processor = KimiGPUProcessorWrapper(
            _processor,
            image_token=self.mm_tokens.image_token,
            patch_size=media_proc_cfg["patch_size"],
            merge_kernel_size=media_proc_cfg["merge_kernel_size"],
            in_patch_limit=media_proc_cfg["in_patch_limit"],
            patch_limit_on_one_side=media_proc_cfg["patch_limit_on_one_side"],
            fixed_output_tokens=media_proc_cfg.get("fixed_output_tokens"),
            image_mean=media_proc_cfg["image_mean"],
            image_std=media_proc_cfg["image_std"],
        )
```
**EN:** This block defines function `KimiK2_5VLImageProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `KimiK2_5VLImageProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 367-367: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiK2_5VLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiK2_5VLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 368-390: Defines async function KimiK2_5VLImageProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self,
        image_data: List[Union[str, bytes, Dict]],
        input_text,
        request_obj,
        *args,
        **kwargs,
    ):
        base_output = self.load_mm_data(
            prompt=input_text,
            image_data=image_data,
            multimodal_tokens=self.mm_tokens,
        )

        mm_items, input_ids, _ = self.process_and_combine_mm_data(
            base_output, self.mm_tokens
        )

        return MultimodalProcessorOutput(
            input_ids=input_ids.tolist(),
            mm_items=mm_items,
            im_token_id=self.mm_tokens.image_token_id,
        )
```
**EN:** This block defines async function `KimiK2_5VLImageProcessor.process_mm_data_async`. Parameters: self, image_data, input_text, request_obj.
**CN:** 该代码块定义异步函数 `KimiK2_5VLImageProcessor.process_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。

### Lines 391-391: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiK2_5VLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiK2_5VLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 392-399: Defines function KimiK2_5VLImageProcessor.get_mm_data
```python
    def get_mm_data(self, prompt, embeddings, **kwargs):
        img_grid_thw = kwargs.get("img_grid_thw", None)
        return self._build_kimi_mm_data_from_grids(
            prompt=prompt,
            embeddings=embeddings,
            image_token_id=self.mm_tokens.image_token_id,
            img_grid_thw=img_grid_thw,
        )
```
**EN:** This block defines function `KimiK2_5VLImageProcessor.get_mm_data`. Parameters: self, prompt, embeddings.
**CN:** 该代码块定义函数 `KimiK2_5VLImageProcessor.get_mm_data`。 参数包括 self、prompt、embeddings。

## Key Concepts / 关键概念
- **Classes / 类**: `KimiGPUProcessorWrapper`, `KimiK2_5VLImageProcessor`
- **Functions / 函数**: `navit_resize_config`, `_get_image_dimensions`, `_pil_to_cuda_chw`, `_process_single_image`, `_gpu_preprocess_images`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `collections`, `math`, `re`, `typing`
- **Third-Party / 第三方**: `PIL`, `numpy`, `torch`, `torch.nn.functional`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.models.kimi_k25`, `sglang.srt.multimodal.processors.base_processor`, `sglang.srt.multimodal.processors.kimi_common`
