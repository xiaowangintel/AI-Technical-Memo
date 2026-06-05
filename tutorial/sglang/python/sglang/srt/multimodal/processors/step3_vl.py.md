# step3_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/step3_vl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for step3 vl models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 step3 vl 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21: Imports dependencies
```python
import math
import re
from itertools import product
from typing import List, Optional, Union

import numpy as np
import torch
from PIL import Image
from torchvision import transforms
from torchvision.transforms import InterpolationMode
from transformers import BatchFeature, ProcessorMixin, TensorType

from sglang.srt.managers.schedule_batch import MultimodalProcessorOutput
from sglang.srt.models.step3_vl import Step3VLForConditionalGeneration
from sglang.srt.models.step3_vl_10b import StepVLForConditionalGeneration
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor as SGLangBaseProcessor,
)
from sglang.srt.multimodal.processors.base_processor import (
    MultimodalSpecialTokens,
)
```
**EN:** This block groups related imports for the module, including math, re, itertools.product, typing.List, typing.Optional, and 14 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 math, re, itertools.product, typing.List, typing.Optional 等 14 项，为后续代码准备所需名称。

### Lines 22-22: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 23-23: Declares ImageWithPatches
```python
ImageWithPatches = tuple[Image.Image, list[Image.Image], list[int] | None]
```
**EN:** This statement initializes ImageWithPatches in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 ImageWithPatches。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 24-25: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 26-27: Declares class GPUToTensor
```python
class GPUToTensor(torch.nn.Module):

```
**EN:** This block introduces class `GPUToTensor` as a reusable abstraction inside the module. It inherits from torch.nn.Module.
**CN:** 该代码块声明类 `GPUToTensor`，作为模块中的可复用抽象。 它继承自 torch.nn.Module。

### Lines 28-41: Defines function GPUToTensor.forward
```python
    def forward(self, raw_image: Union[np.ndarray, Image.Image]) -> torch.Tensor:
        if isinstance(raw_image, Image.Image):
            return transforms.ToTensor()(raw_image)
        if raw_image.ndim == 2:
            raw_image = raw_image[:, :, None].repeat(3, -1)
        if torch.cuda.is_available():
            device = torch.device("cuda")
        else:
            device = torch.device("cpu")
        image_tensor = torch.from_numpy(raw_image).to(device)
        image_tensor = torch.permute(image_tensor, (2, 0, 1)).contiguous()
        if image_tensor.dtype == torch.uint8:
            image_tensor = image_tensor.to(torch.float32).div(255)
        return image_tensor
```
**EN:** This block defines function `GPUToTensor.forward`. Parameters: self, raw_image.
**CN:** 该代码块定义函数 `GPUToTensor.forward`。 参数包括 self、raw_image。

### Lines 42-43: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 44-44: Declares class Step3VisionProcessor
```python
class Step3VisionProcessor:
```
**EN:** This block introduces class `Step3VisionProcessor` as a reusable abstraction inside the module.
**CN:** 该代码块声明类 `Step3VisionProcessor`，作为模块中的可复用抽象。

### Lines 45-84: Defines function Step3VisionProcessor.__init__
```python
    def __init__(self, size, interpolation_mode="bicubic", patch_size=None):
        mean = [0.48145466, 0.4578275, 0.40821073]
        std = [0.26862954, 0.26130258, 0.27577711]
        patch_size = patch_size if patch_size is not None else size

        self.transform = transforms.Compose(
            [
                GPUToTensor(),
                transforms.Normalize(mean, std),
                transforms.Resize(
                    (size, size),
                    interpolation=(
                        InterpolationMode.BICUBIC
                        if interpolation_mode == "bicubic"
                        else InterpolationMode.BILINEAR
                    ),
                    antialias=True,
                ),
            ]
        )

        self.patch_transform = (
            transforms.Compose(
                [
                    GPUToTensor(),
                    transforms.Normalize(mean, std),
                    transforms.Resize(
                        (patch_size, patch_size),
                        interpolation=(
                            InterpolationMode.BICUBIC
                            if interpolation_mode == "bicubic"
                            else InterpolationMode.BILINEAR
                        ),
                        antialias=True,
                    ),
                ]
            )
            if patch_size is not None
            else None
        )
```
**EN:** This block defines function `Step3VisionProcessor.__init__`. Parameters: self, size, interpolation_mode, patch_size.
**CN:** 该代码块定义函数 `Step3VisionProcessor.__init__`。 参数包括 self、size、interpolation_mode、patch_size。

### Lines 85-85: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Step3VisionProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Step3VisionProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 86-90: Defines function Step3VisionProcessor.__call__
```python
    def __call__(self, image, is_patch=False):
        if is_patch:
            return {"pixel_values": self.patch_transform(image).unsqueeze(0)}
        else:
            return {"pixel_values": self.transform(image).unsqueeze(0)}
```
**EN:** This block defines function `Step3VisionProcessor.__call__`. Parameters: self, image, is_patch.
**CN:** 该代码块定义函数 `Step3VisionProcessor.__call__`。 参数包括 self、image、is_patch。

### Lines 91-92: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 93-94: Declares class ImagePatcher
```python
class ImagePatcher:

```
**EN:** This block introduces class `ImagePatcher` as a reusable abstraction inside the module.
**CN:** 该代码块声明类 `ImagePatcher`，作为模块中的可复用抽象。

### Lines 95-98: Defines function ImagePatcher.determine_window_size
```python
    def determine_window_size(self, long: int, short: int) -> int:
        if long <= 728:
            return short if long / short > 1.5 else 0
        return min(short, 504) if long / short > 4 else 504
```
**EN:** This block defines function `ImagePatcher.determine_window_size`. Parameters: self, long, short.
**CN:** 该代码块定义函数 `ImagePatcher.determine_window_size`。 参数包括 self、long、short。

### Lines 99-99: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the ImagePatcher, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 ImagePatcher 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 100-133: Defines function ImagePatcher.slide_window
```python
    def slide_window(
        self,
        width: int,
        height: int,
        sizes: list[tuple[int, int]],
        steps: list[tuple[int, int]],
        img_rate_thr: float = 0.6,
    ) -> tuple[list[tuple[int, int, int, int]], tuple[int, int]]:
        assert 1 >= img_rate_thr >= 0, "The `img_rate_thr` should lie in 0~1"
        windows = []
        # Sliding windows.
        for size, step in zip(sizes, steps):
            size_w, size_h = size
            step_w, step_h = step

            x_num = 1 if width <= size_w else math.ceil((width - size_w) / step_w + 1)
            x_start = [step_w * i for i in range(x_num)]
            if len(x_start) > 1 and x_start[-1] + size_w > width:
                x_start[-1] = width - size_w

            y_num = 1 if height <= size_h else math.ceil((height - size_h) / step_h + 1)
            y_start = [step_h * i for i in range(y_num)]
            if len(y_start) > 1 and y_start[-1] + size_h > height:
                y_start[-1] = height - size_h

            start = np.array(list(product(y_start, x_start)), dtype=int)
            start[:, [0, 1]] = start[:, [1, 0]]
            windows.append(np.concatenate([start, start + size], axis=1))
        windows = np.concatenate(windows, axis=0)

        return [
            (int(box[0]), int(box[1]), int(box[2] - box[0]), int(box[3] - box[1]))
            for box in windows
        ], (x_num, y_num)
```
**EN:** This block defines function `ImagePatcher.slide_window`. Parameters: self, width, height, sizes, steps, img_rate_thr.
**CN:** 该代码块定义函数 `ImagePatcher.slide_window`。 参数包括 self、width、height、sizes、steps、img_rate_thr。

### Lines 134-134: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the ImagePatcher, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 ImagePatcher 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 135-142: Defines function ImagePatcher.square_pad
```python
    def square_pad(self, img: Image.Image) -> Image.Image:
        w, h = img.size
        if w == h:
            return img
        size = max(w, h)
        padded = Image.new(img.mode, (size, size), 0)
        padded.paste(img, (0, 0))
        return padded
```
**EN:** This block defines function `ImagePatcher.square_pad`. Parameters: self, img.
**CN:** 该代码块定义函数 `ImagePatcher.square_pad`。 参数包括 self、img。

### Lines 143-143: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the ImagePatcher, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 ImagePatcher 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 144-151: Defines function ImagePatcher.get_image_size_for_padding
```python
    def get_image_size_for_padding(
        self, img_width: int, img_height: int
    ) -> tuple[int, int]:
        ratio = img_width / img_height
        if min(img_height, img_width) < 32 and (ratio > 4 or ratio < 1 / 4):
            new_size = max(img_height, img_width)
            return new_size, new_size
        return img_width, img_height
```
**EN:** This block defines function `ImagePatcher.get_image_size_for_padding`. Parameters: self, img_width, img_height.
**CN:** 该代码块定义函数 `ImagePatcher.get_image_size_for_padding`。 参数包括 self、img_width、img_height。

### Lines 152-152: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the ImagePatcher, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 ImagePatcher 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 153-163: Defines function ImagePatcher.get_image_size_for_preprocess
```python
    def get_image_size_for_preprocess(
        self, img_width: int, img_height: int
    ) -> tuple[int, int]:

        if max(img_height, img_width) > 3024:
            scale_factor = 3024 / max(img_height, img_width)
            img_width = int(img_width * scale_factor)
            img_height = int(img_height * scale_factor)
            return img_width, img_height
        else:
            return img_width, img_height
```
**EN:** This block defines function `ImagePatcher.get_image_size_for_preprocess`. Parameters: self, img_width, img_height.
**CN:** 该代码块定义函数 `ImagePatcher.get_image_size_for_preprocess`。 参数包括 self、img_width、img_height。

### Lines 164-164: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the ImagePatcher, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 ImagePatcher 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 165-183: Defines function ImagePatcher.get_image_size_for_crop
```python
    def get_image_size_for_crop(
        self, img_width: int, img_height: int, window_size: int
    ):
        w_ratio = img_width / window_size
        h_ratio = img_height / window_size

        if w_ratio < 1:
            width_new = img_width
        else:
            decimal_w = w_ratio - img_width // window_size
            w_ratio = int(w_ratio) + 1 if decimal_w > 0.2 else int(w_ratio)
            width_new = window_size * w_ratio
        if h_ratio < 1:
            height_new = img_height
        else:
            decimal_h = h_ratio - img_height // window_size
            h_ratio = int(h_ratio) + 1 if decimal_h > 0.2 else int(h_ratio)
            height_new = window_size * h_ratio
        return int(width_new), int(height_new)
```
**EN:** This block defines function `ImagePatcher.get_image_size_for_crop`. Parameters: self, img_width, img_height, window_size.
**CN:** 该代码块定义函数 `ImagePatcher.get_image_size_for_crop`。 参数包括 self、img_width、img_height、window_size。

### Lines 184-184: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the ImagePatcher, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 ImagePatcher 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 185-187: Defines function ImagePatcher.patch_crop
```python
    def patch_crop(self, img: Image.Image, i: int, j: int, th: int, tw: int):
        target = img.crop((j, i, j + tw, i + th))
        return target
```
**EN:** This block defines function `ImagePatcher.patch_crop`. Parameters: self, img, i, j, th, tw.
**CN:** 该代码块定义函数 `ImagePatcher.patch_crop`。 参数包括 self、img、i、j、th、tw。

### Lines 188-188: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the ImagePatcher, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 ImagePatcher 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 189-212: Defines function ImagePatcher.get_num_patches
```python
    def get_num_patches(self, img_width: int, img_height: int) -> tuple[int, int]:
        img_width, img_height = self.get_image_size_for_padding(img_width, img_height)
        img_width, img_height = self.get_image_size_for_preprocess(
            img_width, img_height
        )
        window_size = self.determine_window_size(
            max(img_height, img_width), min(img_height, img_width)
        )
        if window_size == 0:
            return 0, 0
        else:
            img_width, img_height = self.get_image_size_for_crop(
                img_width, img_height, window_size
            )
            center_list, (x_num, y_num) = self.slide_window(
                img_width,
                img_height,
                [(window_size, window_size)],
                [(window_size, window_size)],
            )
            full_rows = (len(center_list) - 1) // x_num + 1
            if len(center_list) > 0 and len(center_list) % x_num == 0:
                full_rows -= 1
            return len(center_list), full_rows
```
**EN:** This block defines function `ImagePatcher.get_num_patches`. Parameters: self, img_width, img_height.
**CN:** 该代码块定义函数 `ImagePatcher.get_num_patches`。 参数包括 self、img_width、img_height。

### Lines 213-213: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the ImagePatcher, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 ImagePatcher 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 214-271: Defines function ImagePatcher.__call__
```python
    def __call__(
        self, img: Image.Image
    ) -> tuple[Image.Image, list[Image.Image], list[bool] | None]:
        img_width, img_height = img.size
        new_img_width, new_img_height = self.get_image_size_for_padding(
            img_width, img_height
        )
        if new_img_width != img_width or new_img_height != img_height:
            img = self.square_pad(img)
            img_width, img_height = img.size

        new_img_width, new_img_height = self.get_image_size_for_preprocess(
            img_width, img_height
        )
        img = img.resize((new_img_width, new_img_height), Image.Resampling.BILINEAR)
        window_size = self.determine_window_size(
            max(new_img_height, new_img_width), min(new_img_height, new_img_width)
        )
        if window_size == 0:
            return img, [], None
        else:
            new_img_width, new_img_height = self.get_image_size_for_crop(
                new_img_width, new_img_height, window_size
            )
            if (new_img_width, new_img_height) != (img_width, img_height):
                img_for_crop = img.resize(
                    (new_img_width, new_img_height), Image.Resampling.BILINEAR
                )
            else:
                img_for_crop = img

            patches = []
            newlines = []
            center_list, (x_num, y_num) = self.slide_window(
                new_img_width,
                new_img_height,
                [(window_size, window_size)],
                [(window_size, window_size)],
            )
            for patch_id, center_lf_point in enumerate(center_list):
                x, y, patch_w, patch_h = center_lf_point
                big_patch = self.patch_crop(img_for_crop, y, x, patch_h, patch_w)
                patches.append(big_patch)
                if (patch_id + 1) % x_num == 0:
                    newlines.append(patch_id)

            if newlines and newlines[-1] == len(patches) - 1:
                newlines.pop()

            return (
                img,
                patches,
                (
                    [i in newlines for i in range(len(patches))]
                    if len(patches) > 0
                    else None
                ),
            )
```
**EN:** This block defines function `ImagePatcher.__call__`. Parameters: self, img.
**CN:** 该代码块定义函数 `ImagePatcher.__call__`。 参数包括 self、img。

### Lines 272-273: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 274-274: Declares class Step3VLProcessor
```python
class Step3VLProcessor:
```
**EN:** This block introduces class `Step3VLProcessor` as a reusable abstraction inside the module.
**CN:** 该代码块声明类 `Step3VLProcessor`，作为模块中的可复用抽象。

### Lines 275-299: Defines function Step3VLProcessor.__init__
```python
    def __init__(
        self,
        config,
        tokenizer,
    ) -> None:
        super().__init__()

        self.config = config
        if isinstance(tokenizer, ProcessorMixin):
            tokenizer = tokenizer.tokenizer
        self.tokenizer = tokenizer

        self.image_size = 728
        self.patch_size = 504
        self.image_preprocessor = Step3VisionProcessor(
            self.image_size, "bilinear", self.patch_size
        )

        self.num_image_feature_size = 169
        self.num_patch_feature_size = 81
        self.image_token = "<im_patch>"
        self.image_feature_placeholder = self.image_token * self.num_image_feature_size
        self.patch_feature_placeholder = self.image_token * self.num_patch_feature_size

        self.patcher = ImagePatcher()
```
**EN:** This block defines function `Step3VLProcessor.__init__`. Parameters: self, config, tokenizer.
**CN:** 该代码块定义函数 `Step3VLProcessor.__init__`。 参数包括 self、config、tokenizer。

### Lines 300-300: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Step3VLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Step3VLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 301-303: Defines function Step3VLProcessor.image_token_id
```python
    @property
    def image_token_id(self) -> int:
        return self.tokenizer.get_vocab()[self.image_token]
```
**EN:** This block defines function `Step3VLProcessor.image_token_id`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `Step3VLProcessor.image_token_id`。 参数包括 self。 装饰器包括 property。

### Lines 304-304: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Step3VLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Step3VLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 305-313: Defines function Step3VLProcessor.get_num_image_tokens
```python
    def get_num_image_tokens(self, img_width: int, img_height: int) -> int:
        num_patches, num_newlines = self.patcher.get_num_patches(img_width, img_height)

        return (
            num_patches * (self.num_patch_feature_size + 2)
            + self.num_image_feature_size
            + 2
            + num_newlines
        )
```
**EN:** This block defines function `Step3VLProcessor.get_num_image_tokens`. Parameters: self, img_width, img_height.
**CN:** 该代码块定义函数 `Step3VLProcessor.get_num_image_tokens`。 参数包括 self、img_width、img_height。

### Lines 314-314: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Step3VLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Step3VLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 315-319: Defines function Step3VLProcessor._split_images
```python
    def _split_images(self, images: list[Image.Image]) -> list[ImageWithPatches]:
        result = []
        for img in images:
            result.append(self.patcher(img))
        return result
```
**EN:** This block defines function `Step3VLProcessor._split_images`. Parameters: self, images.
**CN:** 该代码块定义函数 `Step3VLProcessor._split_images`。 参数包括 self、images。

### Lines 320-320: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Step3VLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Step3VLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 321-329: Defines function Step3VLProcessor._convert_images_to_pixel_values
```python
    def _convert_images_to_pixel_values(
        self,
        images: list[Image.Image],
        is_patch: bool = False,
    ) -> list[torch.Tensor]:
        return [
            self.image_preprocessor(img, is_patch=is_patch)["pixel_values"]
            for img in images
        ]
```
**EN:** This block defines function `Step3VLProcessor._convert_images_to_pixel_values`. Parameters: self, images, is_patch.
**CN:** 该代码块定义函数 `Step3VLProcessor._convert_images_to_pixel_values`。 参数包括 self、images、is_patch。

### Lines 330-330: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Step3VLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Step3VLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 331-351: Defines function Step3VLProcessor._get_patch_repl
```python
    def _get_patch_repl(
        self,
        num_patches: int,
        patch_newline_mask: list[bool] | None,
    ) -> tuple[str, list[int]]:
        text = ""
        token_ids = []
        for i in range(num_patches):
            assert len(patch_newline_mask) == num_patches
            text += f"<patch_start>{self.patch_feature_placeholder}<patch_end>"
            token_ids.extend(
                [self.tokenizer.convert_tokens_to_ids("<patch_start>")]
                + [self.image_token_id] * self.num_patch_feature_size
                + [self.tokenizer.convert_tokens_to_ids("<patch_end>")]
            )
            if patch_newline_mask and patch_newline_mask[i]:
                text += "<patch_newline>"
                token_ids.append(
                    self.tokenizer.convert_tokens_to_ids("<patch_newline>")
                )
        return text, token_ids
```
**EN:** This block defines function `Step3VLProcessor._get_patch_repl`. Parameters: self, num_patches, patch_newline_mask.
**CN:** 该代码块定义函数 `Step3VLProcessor._get_patch_repl`。 参数包括 self、num_patches、patch_newline_mask。

### Lines 352-352: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Step3VLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Step3VLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 353-363: Defines function Step3VLProcessor._get_image_repl
```python
    def _get_image_repl(
        self,
        num_images: int,
    ) -> tuple[str, list[int]]:
        text = f"<im_start>{self.image_feature_placeholder}<im_end>"
        token_ids = (
            [self.tokenizer.convert_tokens_to_ids("<im_start>")]
            + [self.image_token_id] * self.num_image_feature_size
            + [self.tokenizer.convert_tokens_to_ids("<im_end>")]
        )
        return text * num_images, token_ids * num_images
```
**EN:** This block defines function `Step3VLProcessor._get_image_repl`. Parameters: self, num_images.
**CN:** 该代码块定义函数 `Step3VLProcessor._get_image_repl`。 参数包括 self、num_images。

### Lines 364-364: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Step3VLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Step3VLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 365-379: Defines function Step3VLProcessor._get_image_repl_features
```python
    def _get_image_repl_features(
        self,
        num_images: int,
        num_patches: int,
        patch_new_line_idx: Optional[list[bool]],
    ) -> tuple[str, list[int]]:
        if num_patches > 0:
            patch_repl, patch_repl_ids = self._get_patch_repl(
                num_patches, patch_new_line_idx
            )
        else:
            patch_repl = ""
            patch_repl_ids = []
        image_repl, image_repl_ids = self._get_image_repl(num_images)
        return patch_repl + image_repl, patch_repl_ids + image_repl_ids
```
**EN:** This block defines function `Step3VLProcessor._get_image_repl_features`. Parameters: self, num_images, num_patches, patch_new_line_idx.
**CN:** 该代码块定义函数 `Step3VLProcessor._get_image_repl_features`。 参数包括 self、num_images、num_patches、patch_new_line_idx。

### Lines 380-380: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Step3VLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Step3VLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 381-394: Defines function Step3VLProcessor.replace_placeholder
```python
    def replace_placeholder(self, text: str, placeholder: str, repls: list[str]) -> str:
        parts = text.split(placeholder)

        if len(parts) - 1 != len(repls):
            raise ValueError(
                "The number of placeholders does not match the number of replacements."  # noqa: E501
            )

        result = [parts[0]]
        for i, repl in enumerate(repls):
            result.append(repl)
            result.append(parts[i + 1])

        return "".join(result)
```
**EN:** This block defines function `Step3VLProcessor.replace_placeholder`. Parameters: self, text, placeholder, repls.
**CN:** 该代码块定义函数 `Step3VLProcessor.replace_placeholder`。 参数包括 self、text、placeholder、repls。

### Lines 395-395: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Step3VLProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Step3VLProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 396-469: Defines function Step3VLProcessor.__call__
```python
    def __call__(
        self,
        text: Optional[Union[str, list[str]]] = None,
        images: Optional[Union[Image.Image, list[Image.Image]]] = None,
        return_tensors: Optional[Union[str, TensorType]] = None,
        *args,
        **kwargs,
    ) -> BatchFeature:
        if text is None:
            text = []
        if not isinstance(text, list):
            text = [text]
        if images is None:
            images = []
        if not isinstance(images, list):
            images = [images]

        if len(images) == 0:
            image_inputs = {}
            text_inputs = self.tokenizer(text)
        else:
            splitted_images_data = self._split_images(images)
            pixel_values_lst = []
            patch_pixel_values_lst = []
            patch_newline_mask_lst = []
            image_repl_str_lst = []
            image_repl_ids_lst = []
            num_patches = []
            for (
                raw_img,
                img_patches,
                patch_newline_mask,
            ) in splitted_images_data:  # noqa: E501
                pixel_values_lst.extend(self._convert_images_to_pixel_values([raw_img]))

                if len(img_patches) > 0:
                    patch_pixel_values_lst.extend(
                        self._convert_images_to_pixel_values(img_patches, is_patch=True)
                    )
                num_patches.append(len(img_patches))

                image_repl_str, image_repl_ids = self._get_image_repl_features(
                    1, len(img_patches), patch_newline_mask
                )
                image_repl_str_lst.append(image_repl_str)
                image_repl_ids_lst.extend(image_repl_ids)

                if patch_newline_mask is not None:
                    patch_newline_mask_lst.extend(patch_newline_mask)

            image_inputs = {
                "pixel_values": torch.cat(pixel_values_lst),
                "num_patches": num_patches,
            }
            if patch_pixel_values_lst:
                image_inputs["patch_pixel_values"] = torch.cat(patch_pixel_values_lst)
            if patch_newline_mask_lst:
                image_inputs["patch_newline_mask"] = torch.tensor(
                    patch_newline_mask_lst, dtype=torch.bool
                )

            text = [
                self.replace_placeholder(t, self.image_token, image_repl_str_lst)
                for t in text
            ]
            text_inputs = self.tokenizer(text)

        return BatchFeature(
            {
                **text_inputs,
                **image_inputs,
            },
            tensor_type=return_tensors,
        )
```
**EN:** This block defines function `Step3VLProcessor.__call__`. Parameters: self, text, images, return_tensors.
**CN:** 该代码块定义函数 `Step3VLProcessor.__call__`。 参数包括 self、text、images、return_tensors。

### Lines 470-474: Comments and module notes
```python


################################################


```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 475-475: Declares class Step3VLImageProcessor
```python
class Step3VLImageProcessor(SGLangBaseProcessor):
```
**EN:** This block introduces class `Step3VLImageProcessor` as a reusable abstraction inside the module. It inherits from SGLangBaseProcessor.
**CN:** 该代码块声明类 `Step3VLImageProcessor`，作为模块中的可复用抽象。 它继承自 SGLangBaseProcessor。

### Lines 476-476: Declares models
```python
    models = [Step3VLForConditionalGeneration, StepVLForConditionalGeneration]
```
**EN:** This statement initializes models in the Step3VLImageProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 Step3VLImageProcessor 中初始化 models。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 477-477: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Step3VLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Step3VLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 478-491: Defines function Step3VLImageProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        # TODO, check _processor is tokenizer or processor.
        processor = Step3VLProcessor(hf_config, _processor)
        super().__init__(hf_config, server_args, processor, *args, **kwargs)
        self.IM_TOKEN = "<im_patch>"
        self.IM_TOKEN_ID = self._processor.tokenizer.get_vocab()[self.IM_TOKEN]
        self.mm_tokens = MultimodalSpecialTokens(
            image_token=self.IM_TOKEN,
            image_token_id=self.IM_TOKEN_ID,
            image_token_regex=re.compile(r"(?:<im_patch>)"),
        ).build(_processor)

        mean = [0.48145466, 0.4578275, 0.40821073]
        std = [0.26862954, 0.26130258, 0.27577711]
```
**EN:** This block defines function `Step3VLImageProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `Step3VLImageProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 492-492: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Step3VLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Step3VLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 493-494: Defines function Step3VLImageProcessor.preprocess
```python
    def preprocess(self, image):
        return {"pixel_values": self.transform(image).unsqueeze(0)}
```
**EN:** This block defines function `Step3VLImageProcessor.preprocess`. Parameters: self, image.
**CN:** 该代码块定义函数 `Step3VLImageProcessor.preprocess`。 参数包括 self、image。

### Lines 495-495: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Step3VLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Step3VLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 496-497: Defines function Step3VLImageProcessor.__call__
```python
    def __call__(self, image):
        return self.preprocess(image)
```
**EN:** This block defines function `Step3VLImageProcessor.__call__`. Parameters: self, image.
**CN:** 该代码块定义函数 `Step3VLImageProcessor.__call__`。 参数包括 self、image。

### Lines 498-498: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Step3VLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Step3VLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 499-522: Defines async function Step3VLImageProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self,
        image_data: List[Union[str, bytes]],
        input_text: str | List[int],
        request_obj,
        *args,
        **kwargs,
    ):
        base_output = self.load_mm_data(
            prompt=input_text,
            image_data=image_data,
            video_data=request_obj.video_data,
            multimodal_tokens=self.mm_tokens,
        )

        mm_items, input_ids, ret = self.process_and_combine_mm_data(
            base_output, self.mm_tokens
        )

        return MultimodalProcessorOutput(
            input_ids=input_ids.tolist(),
            mm_items=mm_items,
            im_token_id=self.mm_tokens.image_token_id,
        )
```
**EN:** This block defines async function `Step3VLImageProcessor.process_mm_data_async`. Parameters: self, image_data, input_text, request_obj.
**CN:** 该代码块定义异步函数 `Step3VLImageProcessor.process_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。

## Key Concepts / 关键概念
- **Classes / 类**: `GPUToTensor`, `Step3VisionProcessor`, `ImagePatcher`, `Step3VLProcessor`, `Step3VLImageProcessor`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `itertools`, `math`, `re`, `typing`
- **Third-Party / 第三方**: `PIL`, `numpy`, `torch`, `torchvision`, `torchvision.transforms`, `transformers`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.models.step3_vl`, `sglang.srt.models.step3_vl_10b`, `sglang.srt.multimodal.processors.base_processor`
