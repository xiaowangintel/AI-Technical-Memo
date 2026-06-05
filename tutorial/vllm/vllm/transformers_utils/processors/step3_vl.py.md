# step3_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/processors/step3_vl.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements model-specific processor integration for Step3 VL inputs. / [CN] 为 Step3 VL 输入实现模型专用的处理器集成。

## Line-by-Line Analysis / 逐行分析

### Lines 3-13: Imports
```python
from itertools import product
from math import ceil

import numpy as np
import torch
from PIL import Image
from torchvision import transforms
from torchvision.transforms.functional import InterpolationMode
from transformers import BatchFeature, ProcessorMixin, TensorType

from vllm.tokenizers import TokenizerLike
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as `itertools`, `math`, external APIs such as `numpy`, `torch`, `PIL`, `torchvision`, `torchvision.transforms.functional`, `transformers`, and internal vLLM modules such as `vllm.tokenizers`. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 `itertools`, `math`，外部 API 如 `numpy`, `torch`, `PIL`, `torchvision`, `torchvision.transforms.functional`, `transformers`，以及 vLLM 内部模块如 `vllm.tokenizers`。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 15-17: Module state and constants
```python
MAX_IMAGE_SIZE: int = 3024

ImageWithPatches = tuple[Image.Image, list[Image.Image], list[bool]]
```
**EN:** This block defines module-level constants/defaults such as `MAX_IMAGE_SIZE`, `ImageWithPatches`. They centralize shared parameters for later helpers and classes.
**CN:** 该代码块定义了模块级常量或默认值，例如 `MAX_IMAGE_SIZE`, `ImageWithPatches`。它们把后续辅助函数和类会复用的参数集中到一起。

### Lines 20-63: Class `Step3VisionProcessor`
```python
class Step3VisionProcessor:
    def __init__(self, size, interpolation_mode="bicubic", patch_size=None):
        mean = [0.48145466, 0.4578275, 0.40821073]
        std = [0.26862954, 0.26130258, 0.27577711]
        patch_size = patch_size if patch_size is not None else size

        self.transform = transforms.Compose(
            [
                transforms.ToTensor(),
                transforms.Normalize(mean, std),
                transforms.Resize(
                    (size, size),
                    interpolation=InterpolationMode.BICUBIC
                    if interpolation_mode == "bicubic"
                    else InterpolationMode.BILINEAR,
                    antialias=True,
                ),
            ]
        )

        self.patch_transform = (
            transforms.Compose(
                [
                    transforms.ToTensor(),
                    transforms.Normalize(mean, std),
                    transforms.Resize(
                        (patch_size, patch_size),
                        interpolation=InterpolationMode.BICUBIC
                        if interpolation_mode == "bicubic"
                        else InterpolationMode.BILINEAR,
                        antialias=True,
                    ),
                ]
            )
            if patch_size is not None
            else None
        )

    def __call__(self, image, is_patch=False):
        if is_patch:
            assert self.patch_transform is not None
            return {"pixel_values": self.patch_transform(image).unsqueeze(0)}

        return {"pixel_values": self.transform(image).unsqueeze(0)}
```
**EN:** Defines `Step3VisionProcessor`, a processor-related class. Key methods include `__init__`, `__call__`.
**CN:** 定义 `Step3VisionProcessor`，这是一个处理器相关类。 关键方法包括 `__init__`, `__call__`。

### Lines 66-240: Class `ImagePatcher`
```python
class ImagePatcher:
    def __init__(self, enable_patch: bool = True) -> None:
        self.enable_patch = enable_patch

    def determine_window_size(self, long: int, short: int) -> int:
        if long < 728:
            return short if long / short > 1.5 else 0
        return min(short, 504) if long / short > 4 else 504

    def slide_window(
        self,
        width: int,
        height: int,
        sizes: list[tuple[int, int]],
        steps: list[tuple[int, int]],
        img_rate_thr: float = 0.6,
    ) -> tuple[list[tuple[int, int, int, int]], tuple[int, int]]:
        assert 1 >= img_rate_thr >= 0, "The `in_rate_thr` should lie in 0~1"
        windows = []
        # Sliding windows.
        for size, step in zip(sizes, steps):
            size_w, size_h = size
            step_w, step_h = step

            x_num = 1 if width <= size_w else ceil((width - size_w) / step_w + 1)
            x_start = [step_w * i for i in range(x_num)]
            if len(x_start) > 1 and x_start[-1] + size_w > width:
                x_start[-1] = width - size_w

            y_num = 1 if height <= size_h else ceil((height - size_h) / step_h + 1)
            y_start = [step_h * i for i in range(y_num)]
            if len(y_start) > 1 and y_start[-1] + size_h > height:
                y_start[-1] = height - size_h

            start = np.array(list(product(y_start, x_start)), dtype=int)
            start[:, [0, 1]] = start[:, [1, 0]]
            windows.append(np.concatenate([start, start + size], axis=1))
        windows = np.concatenate(windows, axis=0)

        return [
# ... omitted for brevity ...
                patches.append(big_patch)
                if (patch_id + 1) % x_num == 0:
                    newlines.append(patch_id)

            if newlines and newlines[-1] == len(patches) - 1:
                newlines.pop()

            return (
                img,
                patches,
                [i in newlines for i in range(len(patches))],
            )
```
**EN:** Defines `ImagePatcher`, a processor-related class. Key methods include `__init__`, `determine_window_size`, `slide_window`, `square_pad`, `get_image_size_for_padding`, ... (+5 more).
**CN:** 定义 `ImagePatcher`，这是一个处理器相关类。 关键方法包括 `__init__`, `determine_window_size`, `slide_window`, `square_pad`, `get_image_size_for_padding`, ... (+5 more)。

### Lines 243-322: Class `Step3VLImageProcessor`
```python
class Step3VLImageProcessor:
    def __init__(
        self,
        image_size: int = 728,
        patch_size: int = 504,
        num_image_feature_size: int = 169,
        num_patch_feature_size: int = 81,
        enable_patch: bool = True,
    ) -> None:
        self.image_size = image_size
        self.patch_size = patch_size
        self.num_image_feature_size = num_image_feature_size
        self.num_patch_feature_size = num_patch_feature_size
        self.image_preprocessor = Step3VisionProcessor(
            image_size, "bilinear", patch_size
        )
        self.patcher = ImagePatcher(enable_patch=enable_patch)

    def get_num_image_tokens(self, img_width: int, img_height: int) -> int:
        num_patches, num_newlines = self.patcher.get_num_patches(img_width, img_height)

        return (
            num_patches * (self.num_patch_feature_size + 2)
            + self.num_image_feature_size
            + 2
            + num_newlines
        )

    def _split_images(self, images: list[Image.Image]) -> list[ImageWithPatches]:
        result = []
        for img in images:
            result.append(self.patcher(img))
        return result

    def _convert_images_to_pixel_values(
        self,
        images: list[Image.Image],
        is_patch: bool = False,
    ) -> list[torch.Tensor]:
        return [
# ... omitted for brevity ...
            "pixel_values": pixel_values,
            "num_patches": num_patches,
            "patch_pixel_values": (
                torch.cat(patch_pixel_values_lst)
                if patch_pixel_values_lst
                else pixel_values.new_empty((0, 3, patch_size, patch_size))
            ),
            "patch_newline_mask": torch.tensor(
                patch_newline_mask_lst, dtype=torch.bool
            ),
        }
        return BatchFeature(image_inputs, tensor_type=return_tensors)
```
**EN:** Defines `Step3VLImageProcessor`, a processor-related class. Key methods include `__init__`, `get_num_image_tokens`, `_split_images`, `_convert_images_to_pixel_values`, `__call__`.
**CN:** 定义 `Step3VLImageProcessor`，这是一个处理器相关类。 关键方法包括 `__init__`, `get_num_image_tokens`, `_split_images`, `_convert_images_to_pixel_values`, `__call__`。

### Lines 325-509: Class `Step3VLProcessor`
```python
class Step3VLProcessor(ProcessorMixin):
    attributes = ["image_processor", "tokenizer"]

    def __init__(
        self,
        image_processor: Step3VLImageProcessor,
        tokenizer: TokenizerLike,
    ) -> None:
        self.image_processor = image_processor
        self.tokenizer = tokenizer

        self.image_start_token = image_start_token = "<im_start>"
        self.image_end_token = image_end_token = "<im_end>"
        self.patch_start_token = patch_start_token = "<patch_start>"
        self.patch_end_token = patch_end_token = "<patch_end>"
        self.patch_newline_token = patch_newline_token = "<patch_newline>"
        self.image_start_token_id = tokenizer.convert_tokens_to_ids(image_start_token)
        self.image_end_token_id = tokenizer.convert_tokens_to_ids(image_end_token)
        self.patch_start_token_id = tokenizer.convert_tokens_to_ids(patch_start_token)
        self.patch_end_token_id = tokenizer.convert_tokens_to_ids(patch_end_token)
        self.patch_newline_token_id = tokenizer.convert_tokens_to_ids(
            patch_newline_token
        )

        self.image_token = image_token = "<im_patch>"
        self.image_feature_tokens = image_token * image_processor.num_image_feature_size
        self.patch_feature_tokens = image_token * image_processor.num_patch_feature_size

        self.image_token_id = image_token_id = tokenizer.convert_tokens_to_ids(
            image_token
        )
        self.image_feature_token_ids = [
            image_token_id
        ] * image_processor.num_image_feature_size
        self.patch_feature_token_ids = [
            image_token_id
        ] * image_processor.num_patch_feature_size

    def _get_patch_repl_text(
        self,
# ... omitted for brevity ...
                    self.replace_placeholder(t, image_token, image_repl_str_lst)
                    for t in text
                ]

            text_inputs = self.tokenizer(text)
        else:
            text_inputs = {}

        return BatchFeature(
            data={**text_inputs, **image_inputs},
            tensor_type=return_tensors,
        )
```
**EN:** Defines `Step3VLProcessor`, a processor-related class derived from `ProcessorMixin`. Key methods include `__init__`, `_get_patch_repl_text`, `_get_patch_repl_ids`, `_get_image_repl_text`, `_get_image_repl_ids`, ... (+4 more).
**CN:** 定义 `Step3VLProcessor`，这是一个处理器相关类，继承自 `ProcessorMixin`。 关键方法包括 `__init__`, `_get_patch_repl_text`, `_get_patch_repl_ids`, `_get_image_repl_text`, `_get_image_repl_ids`, ... (+4 more)。

## Key Concepts / 关键概念
- **EN:** Processor wrappers unify tokenizers and modality-specific preprocessors behind a single serving-oriented interface.
- **CN:** 处理器封装把 tokenizer 与模态专用预处理器统一到面向服务的单一接口之下。
- **EN:** The module is tightly coupled to Hugging Face Transformers APIs, so version compatibility is often an explicit concern.
- **CN:** 该模块与 Hugging Face Transformers API 紧密耦合，因此版本兼容性通常是显式关注点。
- **EN:** Internal vLLM imports connect the file to runtime registries, logging, artifact lookup, and model-execution utilities.
- **CN:** vLLM 内部导入会把该文件连接到运行时注册表、日志、模型文件查找以及模型执行工具。
- **EN:** Torch types and tensor conventions influence preprocessing, dtype selection, or runtime metadata in this module.
- **CN:** Torch 的类型与张量约定会影响本模块中的预处理、数据类型选择或运行时元数据。

## Dependencies / 依赖关系
- **EN:** Standard library modules: `itertools`, `math`.
- **CN:** 标准库模块：`itertools`, `math`。
- **EN:** External packages: `numpy`, `torch`, `PIL`, `torchvision`, `torchvision.transforms.functional`, `transformers`.
- **CN:** 外部依赖包：`numpy`, `torch`, `PIL`, `torchvision`, `torchvision.transforms.functional`, `transformers`。
- **EN:** Internal modules: `vllm.tokenizers`.
- **CN:** 内部模块：`vllm.tokenizers`。
