# deepseek_ocr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/deepseek_ocr.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for deepseek ocr so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 deepseek ocr 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19: Imports dependencies
```python
import math
from dataclasses import dataclass
from typing import Any, Dict, List, Optional, Tuple

import torch
from PIL import Image, ImageOps
from transformers import (
    AutoProcessor,
    LlamaTokenizerFast,
    PretrainedConfig,
    ProcessorMixin,
)

from sglang.srt.multimodal.customized_mm_processor_utils import (
    register_customized_processor,
)
from sglang.srt.sampling.custom_logit_processor import (
    DeepseekOCRNoRepeatNGramLogitProcessor,
)
```
**EN:** This block groups related imports for the module, including math, dataclasses.dataclass, typing.Any, typing.Dict, typing.List, and 11 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 math, dataclasses.dataclass, typing.Any, typing.Dict, typing.List 等 11 项，为后续代码准备所需名称。

### Lines 20-20: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 21-30: Declares BASE_SIZE, IMAGE_SIZE, CROP_MODE, MIN_CROPS, MAX_CROPS, and 5 more
```python
BASE_SIZE = 1024
IMAGE_SIZE = 640
CROP_MODE = True
MIN_CROPS = 2
MAX_CROPS = 6  # max:9; If your GPU memory is small, it is recommended to set it to 6.
MAX_CONCURRENCY = 100  # If you have limited GPU memory, lower the concurrency count.
NUM_WORKERS = 64  # image pre-process (resize/padding) workers
PRINT_NUM_VIS_TOKENS = False
SKIP_REPEAT = True
MODEL_PATH = "deepseek-ai/DeepSeek-OCR"  # change to your model path
```
**EN:** This block initializes a related set of values in the module, including BASE_SIZE, IMAGE_SIZE, CROP_MODE, MIN_CROPS, MAX_CROPS, and 5 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 模块 中初始化一组相关值，包括 BASE_SIZE, IMAGE_SIZE, CROP_MODE, MIN_CROPS, MAX_CROPS 等 5 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 31-31: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 32-33: Declares NGRAM_NO_REPEAT_SIZE, NGRAM_NO_REPEAT_WINDOW
```python
NGRAM_NO_REPEAT_SIZE = 30
NGRAM_NO_REPEAT_WINDOW = 90
```
**EN:** This block initializes a related set of values in the module, including NGRAM_NO_REPEAT_SIZE, NGRAM_NO_REPEAT_WINDOW. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 模块 中初始化一组相关值，包括 NGRAM_NO_REPEAT_SIZE, NGRAM_NO_REPEAT_WINDOW。将这些赋值集中在一起有助于理解周边配置。

### Lines 34-34: Comments and module notes
```python
# Whitelist `<td>` and `</td>` token ids to allow table structures.
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 35-35: Declares NGRAM_NO_REPEAT_WHITELIST
```python
NGRAM_NO_REPEAT_WHITELIST = (128821, 128822)
```
**EN:** This statement initializes NGRAM_NO_REPEAT_WHITELIST in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 NGRAM_NO_REPEAT_WHITELIST。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 36-36: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 37-37: Declares DEFAULT_CUSTOM_LOGIT_PROCESSOR
```python
DEFAULT_CUSTOM_LOGIT_PROCESSOR = DeepseekOCRNoRepeatNGramLogitProcessor.to_str()
```
**EN:** This statement initializes DEFAULT_CUSTOM_LOGIT_PROCESSOR in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 DEFAULT_CUSTOM_LOGIT_PROCESSOR。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 38-39: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 40-47: Defines function get_default_ngram_custom_params
```python
def get_default_ngram_custom_params() -> Dict[str, Any]:
    """Return default custom params for the DeepSeek-OCR n-gram no repeat processor."""

    return {
        "ngram_size": NGRAM_NO_REPEAT_SIZE,
        "window_size": NGRAM_NO_REPEAT_WINDOW,
        "whitelist_token_ids": list(NGRAM_NO_REPEAT_WHITELIST),
    }
```
**EN:** This block defines function `get_default_ngram_custom_params`. It takes no explicit parameters. Return default custom params for the DeepSeek-OCR n-gram no repeat processor.
**CN:** 该代码块定义函数 `get_default_ngram_custom_params`。 它没有显式参数。 文档字符串摘要：Return default custom params for the DeepSeek-OCR n-gram no repeat processor.

### Lines 48-49: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 50-50: Declares PROMPT
```python
PROMPT = "<image>\n<|grounding|>Convert the document to markdown."
```
**EN:** This statement initializes PROMPT in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 PROMPT。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 51-52: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 53-53: Declares class DictOutput
```python
class DictOutput(object):
```
**EN:** This block introduces class `DictOutput` as a reusable abstraction inside the module. It inherits from object.
**CN:** 该代码块声明类 `DictOutput`，作为模块中的可复用抽象。 它继承自 object。

### Lines 54-55: Defines function DictOutput.items
```python
    def items(self):
        return self.__dict__.items()
```
**EN:** This block defines function `DictOutput.items`. Parameters: self.
**CN:** 该代码块定义函数 `DictOutput.items`。 参数包括 self。

### Lines 56-56: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DictOutput, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DictOutput 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 57-58: Defines function DictOutput.keys
```python
    def keys(self):
        return self.__dict__.keys()
```
**EN:** This block defines function `DictOutput.keys`. Parameters: self.
**CN:** 该代码块定义函数 `DictOutput.keys`。 参数包括 self。

### Lines 59-59: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DictOutput, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DictOutput 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 60-61: Defines function DictOutput.__getitem__
```python
    def __getitem__(self, item):
        return self.__dict__[item]
```
**EN:** This block defines function `DictOutput.__getitem__`. Parameters: self, item.
**CN:** 该代码块定义函数 `DictOutput.__getitem__`。 参数包括 self、item。

### Lines 62-62: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DictOutput, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DictOutput 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 63-64: Defines function DictOutput.__contains__
```python
    def __contains__(self, key):
        return key in self.__dict__
```
**EN:** This block defines function `DictOutput.__contains__`. Parameters: self, key.
**CN:** 该代码块定义函数 `DictOutput.__contains__`。 参数包括 self、key。

### Lines 65-65: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DictOutput, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DictOutput 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 66-67: Defines function DictOutput.__setitem__
```python
    def __setitem__(self, key, value):
        self.__dict__[key] = value
```
**EN:** This block defines function `DictOutput.__setitem__`. Parameters: self, key, value.
**CN:** 该代码块定义函数 `DictOutput.__setitem__`。 参数包括 self、key、value。

### Lines 68-69: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 70-71: Declares class VLChatProcessorOutput
```python
@dataclass
class VLChatProcessorOutput(DictOutput):
```
**EN:** This block introduces class `VLChatProcessorOutput` as a reusable abstraction inside the module. It inherits from DictOutput.
**CN:** 该代码块声明类 `VLChatProcessorOutput`，作为模块中的可复用抽象。 它继承自 DictOutput。

### Lines 72-79: Declares input_ids, target_ids, images_crop, pixel_values, images_seq_mask, and 1 more
```python
    input_ids: torch.LongTensor
    target_ids: torch.LongTensor
    images_crop: torch.LongTensor
    pixel_values: (
        torch.Tensor
    )  # rename from "images" to "pixel_values" for compatibility
    images_seq_mask: torch.BoolTensor
    images_spatial_crop: torch.LongTensor
```
**EN:** This block initializes a related set of values in the VLChatProcessorOutput, including input_ids, target_ids, images_crop, pixel_values, images_seq_mask, and 1 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 VLChatProcessorOutput 中初始化一组相关值，包括 input_ids, target_ids, images_crop, pixel_values, images_seq_mask 等 1 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 80-80: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLChatProcessorOutput, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLChatProcessorOutput 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 81-82: Defines function VLChatProcessorOutput.__len__
```python
    def __len__(self):
        return len(self.input_ids)
```
**EN:** This block defines function `VLChatProcessorOutput.__len__`. Parameters: self.
**CN:** 该代码块定义函数 `VLChatProcessorOutput.__len__`。 参数包括 self。

### Lines 83-84: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 85-85: Declares class ImageTransform
```python
class ImageTransform(object):
```
**EN:** This block introduces class `ImageTransform` as a reusable abstraction inside the module. It inherits from object.
**CN:** 该代码块声明类 `ImageTransform`，作为模块中的可复用抽象。 它继承自 object。

### Lines 86-111: Defines function ImageTransform.__init__
```python
    def __init__(
        self,
        mean: Optional[Tuple[float, float, float]] = (0.5, 0.5, 0.5),
        std: Optional[Tuple[float, float, float]] = (0.5, 0.5, 0.5),
        normalize: bool = True,
    ):
        self.mean = mean
        self.std = std
        self.normalize = normalize

        # only load torchvision.transforms when needed
        try:
            import torchvision.transforms as T

            # FIXME: add version check for gguf
        except ImportError as err:
            raise ImportError(
                "Please install torchvision via `pip install torchvision` to use Deepseek-VL2."
            ) from err

        transform_pipelines = [T.ToTensor()]

        if normalize:
            transform_pipelines.append(T.Normalize(mean, std))

        self.transform = T.Compose(transform_pipelines)
```
**EN:** This block defines function `ImageTransform.__init__`. Parameters: self, mean, std, normalize.
**CN:** 该代码块定义函数 `ImageTransform.__init__`。 参数包括 self、mean、std、normalize。

### Lines 112-112: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the ImageTransform, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 ImageTransform 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 113-115: Defines function ImageTransform.__call__
```python
    def __call__(self, pil_img: Image.Image):
        x = self.transform(pil_img)
        return x
```
**EN:** This block defines function `ImageTransform.__call__`. Parameters: self, pil_img.
**CN:** 该代码块定义函数 `ImageTransform.__call__`。 参数包括 self、pil_img。

### Lines 116-117: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 118-131: Defines function find_closest_aspect_ratio
```python
def find_closest_aspect_ratio(aspect_ratio, target_ratios, width, height, image_size):
    best_ratio_diff = float("inf")
    best_ratio = (1, 1)
    area = width * height
    for ratio in target_ratios:
        target_aspect_ratio = ratio[0] / ratio[1]
        ratio_diff = abs(aspect_ratio - target_aspect_ratio)
        if ratio_diff < best_ratio_diff:
            best_ratio_diff = ratio_diff
            best_ratio = ratio
        elif ratio_diff == best_ratio_diff:
            if area > 0.5 * image_size * image_size * ratio[0] * ratio[1]:
                best_ratio = ratio
    return best_ratio
```
**EN:** This block defines function `find_closest_aspect_ratio`. Parameters: aspect_ratio, target_ratios, width, height, image_size.
**CN:** 该代码块定义函数 `find_closest_aspect_ratio`。 参数包括 aspect_ratio、target_ratios、width、height、image_size。

### Lines 132-133: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 134-177: Defines function dynamic_preprocess
```python
def dynamic_preprocess(
    image, min_num=MIN_CROPS, max_num=MAX_CROPS, image_size=640, use_thumbnail=False
):
    orig_width, orig_height = image.size
    aspect_ratio = orig_width / orig_height

    # calculate the existing image aspect ratio
    target_ratios = set(
        (i, j)
        for n in range(min_num, max_num + 1)
        for i in range(1, n + 1)
        for j in range(1, n + 1)
        if i * j <= max_num and i * j >= min_num
    )
    target_ratios = sorted(target_ratios, key=lambda x: x[0] * x[1])

    # find the closest aspect ratio to the target
    target_aspect_ratio = find_closest_aspect_ratio(
        aspect_ratio, target_ratios, orig_width, orig_height, image_size
    )

    # calculate the target width and height
    target_width = image_size * target_aspect_ratio[0]
    target_height = image_size * target_aspect_ratio[1]
    blocks = target_aspect_ratio[0] * target_aspect_ratio[1]

    # resize the image
    resized_img = image.resize((target_width, target_height))
    processed_images = []
    for i in range(blocks):
        box = (
            (i % (target_width // image_size)) * image_size,
            (i // (target_width // image_size)) * image_size,
            ((i % (target_width // image_size)) + 1) * image_size,
            ((i // (target_width // image_size)) + 1) * image_size,
        )
        # split the image
        split_img = resized_img.crop(box)
        processed_images.append(split_img)
    assert len(processed_images) == blocks
    if use_thumbnail and len(processed_images) != 1:
        thumbnail_img = image.resize((image_size, image_size))
        processed_images.append(thumbnail_img)
    return processed_images, target_aspect_ratio
```
**EN:** This block defines function `dynamic_preprocess`. Parameters: image, min_num, max_num, image_size, use_thumbnail.
**CN:** 该代码块定义函数 `dynamic_preprocess`。 参数包括 image、min_num、max_num、image_size、use_thumbnail。

### Lines 178-179: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 180-180: Declares class DeepseekOCRProcessor
```python
class DeepseekOCRProcessor(ProcessorMixin):
```
**EN:** This block introduces class `DeepseekOCRProcessor` as a reusable abstraction inside the module. It inherits from ProcessorMixin.
**CN:** 该代码块声明类 `DeepseekOCRProcessor`，作为模块中的可复用抽象。 它继承自 ProcessorMixin。

### Lines 181-182: Declares tokenizer_class, attributes
```python
    tokenizer_class = ("LlamaTokenizer", "LlamaTokenizerFast")
    attributes = ["tokenizer"]
```
**EN:** This block initializes a related set of values in the DeepseekOCRProcessor, including tokenizer_class, attributes. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepseekOCRProcessor 中初始化一组相关值，包括 tokenizer_class, attributes。将这些赋值集中在一起有助于理解周边配置。

### Lines 183-183: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekOCRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekOCRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 184-252: Defines function DeepseekOCRProcessor.__init__
```python
    def __init__(
        self,
        tokenizer: LlamaTokenizerFast,
        candidate_resolutions: Tuple[Tuple[int, int]],
        patch_size: int,
        downsample_ratio: int,
        image_mean: Tuple[float, float, float] = (0.5, 0.5, 0.5),
        image_std: Tuple[float, float, float] = (0.5, 0.5, 0.5),
        normalize: bool = True,
        image_token: str = "<image>",
        pad_token: str = "<｜▁pad▁｜>",
        add_special_token: bool = False,
        sft_format: str = "deepseek",
        mask_prompt: bool = True,
        ignore_id: int = -100,
        ocr2_mode: bool = False,
        **kwargs,
    ):

        self.candidate_resolutions = candidate_resolutions
        self.image_size = candidate_resolutions[0][0]
        self.patch_size = patch_size
        self.image_mean = image_mean
        self.image_std = image_std
        self.normalize = normalize
        self.downsample_ratio = downsample_ratio
        self.base_size = BASE_SIZE
        self.image_transform = ImageTransform(
            mean=image_mean, std=image_std, normalize=normalize
        )
        self.tokenizer = tokenizer
        # must set this，padding side with make a difference in batch inference
        self.tokenizer.padding_side = "left"

        # add the pad_token as special token to use 'tokenizer.pad_token' and 'tokenizer.pad_token_id'
        if tokenizer.pad_token is None:
            self.tokenizer.add_special_tokens({"pad_token": pad_token})

        # add image token
        image_token_id = self.tokenizer.vocab.get(image_token)
        if image_token_id is None:
            special_tokens = [image_token]
            special_tokens_dict = {"additional_special_tokens": special_tokens}
            self.tokenizer.add_special_tokens(special_tokens_dict)
        self.image_token_id = self.tokenizer.vocab.get(image_token)

        # add five special tokens for grounding-related tasks
        # <|ref|>, <|/ref|>, <|det|>, <|/det|>, <|grounding|>
        special_tokens = ["<|ref|>", "<|/ref|>", "<|det|>", "<|/det|>", "<|grounding|>"]
        special_tokens_dict = {"additional_special_tokens": special_tokens}
        self.tokenizer.add_special_tokens(special_tokens_dict)

        # add special tokens for SFT data
        special_tokens = ["<|User|>", "<|Assistant|>"]
        special_tokens_dict = {"additional_special_tokens": special_tokens}
        self.tokenizer.add_special_tokens(special_tokens_dict)

        self.image_token = image_token
        self.pad_token = pad_token
        self.add_special_token = add_special_token
        self.sft_format = sft_format
        self.mask_prompt = mask_prompt
        self.ignore_id = ignore_id
        self.ocr2_mode = ocr2_mode

        super().__init__(
            tokenizer,
            **kwargs,
        )
```
**EN:** This block defines function `DeepseekOCRProcessor.__init__`. Parameters: self, tokenizer, candidate_resolutions, patch_size, downsample_ratio, image_mean, image_std, normalize, image_token, pad_token, add_special_token, sft_format, mask_prompt, ignore_id, ocr2_mode.
**CN:** 该代码块定义函数 `DeepseekOCRProcessor.__init__`。 参数包括 self、tokenizer、candidate_resolutions、patch_size、downsample_ratio、image_mean、image_std、normalize、image_token、pad_token、add_special_token、sft_format、mask_prompt、ignore_id、ocr2_mode。

### Lines 253-253: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekOCRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekOCRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 254-292: Defines function DeepseekOCRProcessor.format_messages_v2
```python
    def format_messages_v2(self, messages: str, pil_images, max_req_input_len=-1):
        """play the role of format_messages_v2 and get_images_info in the last version"""
        tokenized_data = []
        masked_tokenized_data = []  # labels
        images_list = []
        images_seq_mask = []
        images_spatial_crop = []

        image_index = 0
        image_token_cnt = messages.count(self.image_token)
        (
            input_ids,
            images,
            images_crop,
            seq_mask,
            spatial_crop,
            num_image_tokens,
            image_shapes,
        ) = self.tokenize_with_images(
            messages,
            pil_images[image_index : image_index + image_token_cnt],
            bos=True,
            eos=True,
            cropping=len(pil_images) <= 2,
        )

        image_index = image_token_cnt
        images_list += images
        images_seq_mask += seq_mask
        images_spatial_crop = spatial_crop

        return (
            input_ids,
            masked_tokenized_data,
            images_list,
            images_seq_mask,
            images_spatial_crop,
            images_crop,
        )
```
**EN:** This block defines function `DeepseekOCRProcessor.format_messages_v2`. Parameters: self, messages, pil_images, max_req_input_len. play the role of format_messages_v2 and get_images_info in the last version
**CN:** 该代码块定义函数 `DeepseekOCRProcessor.format_messages_v2`。 参数包括 self、messages、pil_images、max_req_input_len。 文档字符串摘要：play the role of format_messages_v2 and get_images_info in the last version

### Lines 293-293: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekOCRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekOCRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 294-296: Defines function DeepseekOCRProcessor.bos_id
```python
    @property
    def bos_id(self):
        return self.tokenizer.bos_token_id
```
**EN:** This block defines function `DeepseekOCRProcessor.bos_id`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `DeepseekOCRProcessor.bos_id`。 参数包括 self。 装饰器包括 property。

### Lines 297-297: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekOCRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekOCRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 298-300: Defines function DeepseekOCRProcessor.eos_id
```python
    @property
    def eos_id(self):
        return self.tokenizer.eos_token_id
```
**EN:** This block defines function `DeepseekOCRProcessor.eos_id`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `DeepseekOCRProcessor.eos_id`。 参数包括 self。 装饰器包括 property。

### Lines 301-301: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekOCRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekOCRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 302-304: Defines function DeepseekOCRProcessor.pad_id
```python
    @property
    def pad_id(self):
        return self.tokenizer.pad_token_id
```
**EN:** This block defines function `DeepseekOCRProcessor.pad_id`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `DeepseekOCRProcessor.pad_id`。 参数包括 self。 装饰器包括 property。

### Lines 305-305: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekOCRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekOCRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 306-314: Defines function DeepseekOCRProcessor.encode
```python
    def encode(self, text: str, bos: bool = True, eos: bool = False):
        t = self.tokenizer.encode(text, add_special_tokens=False)

        if bos:
            t = [self.bos_id] + t
        if eos:
            t = t + [self.eos_id]

        return t
```
**EN:** This block defines function `DeepseekOCRProcessor.encode`. Parameters: self, text, bos, eos.
**CN:** 该代码块定义函数 `DeepseekOCRProcessor.encode`。 参数包括 self、text、bos、eos。

### Lines 315-315: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekOCRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekOCRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 316-317: Defines function DeepseekOCRProcessor.decode
```python
    def decode(self, t: List[int], **kwargs) -> str:
        return self.tokenizer.decode(t, **kwargs)
```
**EN:** This block defines function `DeepseekOCRProcessor.decode`. Parameters: self, t.
**CN:** 该代码块定义函数 `DeepseekOCRProcessor.decode`。 参数包括 self、t。

### Lines 318-318: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekOCRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekOCRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 319-391: Defines function DeepseekOCRProcessor.process_one
```python
    def process_one(
        self,
        prompt: str = None,
        conversations: List[Dict[str, str]] = None,
        images: List[Image.Image] = None,
        apply_sft_format: bool = False,
        inference_mode: bool = True,
        system_prompt: str = "",
        max_req_input_len: int = -1,
        cropping: bool = True,
        **kwargs,
    ):
        """

        Args:
            prompt (str): the formatted prompt;
            conversations (List[Dict]): conversations with a list of messages;
            images (List[ImageType]): the list of images;
            apply_sft_format (bool): if prompt is not None, then apply the SFT format to prompt;
                if conversations is not None, then it will always apply the SFT format to conversations;
            inference_mode (bool): if True, then remove the last eos token;
            system_prompt (str): the system prompt;
            **kwargs:

        Returns:
            outputs (BaseProcessorOutput): the output of the processor,
                - input_ids (torch.LongTensor): [N + image tokens]
                - target_ids (torch.LongTensor): [N + image tokens]
                - images (torch.FloatTensor): [n_images, 3, H, W]
                - image_id (int): the id of the image token
                - num_image_tokens (List[int]): the number of image tokens
        """

        prompt = conversations or prompt
        (
            input_ids,
            masked_tokenized_str,
            images_list,
            images_seq_mask,
            images_spatial_crop,
            images_crop,
        ) = self.format_messages_v2(prompt, images, max_req_input_len)

        target_ids = torch.LongTensor(masked_tokenized_str)

        has_images = len(images_list) > 0
        has_local_crops = False
        if len(images_spatial_crop) > 0:
            has_local_crops = any(
                crop[0] > 1 or crop[1] > 1 for crop in images_spatial_crop
            )

        if len(images_list) == 0:
            images = torch.zeros((1, 3, self.image_size, self.image_size))
        else:
            images = torch.stack(images_list, dim=0)

        images_spatial_crop = torch.stack(
            [images_spatial_crop], dim=0
        )  # stack the tensor to make it a batch of 1

        prepare = VLChatProcessorOutput(
            input_ids=input_ids,
            target_ids=target_ids,
            images_crop=images_crop,
            pixel_values=images,
            images_seq_mask=images_seq_mask,
            images_spatial_crop=images_spatial_crop,
        )
        prepare.has_images = has_images
        prepare.has_local_crops = has_local_crops

        return prepare
```
**EN:** This block defines function `DeepseekOCRProcessor.process_one`. Parameters: self, prompt, conversations, images, apply_sft_format, inference_mode, system_prompt, max_req_input_len, cropping. Args: prompt (str): the formatted prompt; conversations (List[Dict]): conversations with a list of messages; images (List[ImageType]): the list of images; apply_sft_format (bool): if prompt is not None, then apply the SFT format to prompt; if conversations is
**CN:** 该代码块定义函数 `DeepseekOCRProcessor.process_one`。 参数包括 self、prompt、conversations、images、apply_sft_format、inference_mode、system_prompt、max_req_input_len、cropping。 文档字符串摘要：Args: prompt (str): the formatted prompt; conversations (List[Dict]): conversations with a list of messages; images (List[ImageType]): the list of images; apply_sft_format (bool): if prompt is not None, then apply the SFT format to prompt; if conversations is

### Lines 392-392: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekOCRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekOCRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 393-419: Defines function DeepseekOCRProcessor.__call__
```python
    def __call__(
        self,
        *,
        prompt: str = None,
        conversations: List[Dict[str, str]] = None,
        images: List[Image.Image] = None,
        apply_sft_format: bool = False,
        inference_mode: bool = True,
        system_prompt: str = "",
        max_req_input_len: int = -1,
        text: list[str] = None,
        **kwargs,
    ):
        assert text is None or isinstance(text, list)
        if text is not None:
            text = text[0]
        prepare = self.process_one(
            prompt=prompt or text,
            conversations=conversations,
            images=images,
            apply_sft_format=apply_sft_format,
            inference_mode=inference_mode,
            system_prompt=system_prompt,
            max_req_input_len=max_req_input_len,
        )

        return prepare
```
**EN:** This block defines function `DeepseekOCRProcessor.__call__`. Parameters: self.
**CN:** 该代码块定义函数 `DeepseekOCRProcessor.__call__`。 参数包括 self。

### Lines 420-420: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekOCRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekOCRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 421-426: Defines function DeepseekOCRProcessor.find_all_indices
```python
    def find_all_indices(self, messages, target_value):
        indices = []
        for index, item in enumerate(messages):
            if item == target_value:
                indices.append(index)
        return indices
```
**EN:** This block defines function `DeepseekOCRProcessor.find_all_indices`. Parameters: self, messages, target_value.
**CN:** 该代码块定义函数 `DeepseekOCRProcessor.find_all_indices`。 参数包括 self、messages、target_value。

### Lines 427-427: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekOCRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekOCRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 428-493: Defines function DeepseekOCRProcessor.tokenize_with_images (part 1)
```python
    def tokenize_with_images(
        self,
        conversation: str,
        images: List[Image.Image],
        bos: bool = True,
        eos: bool = True,
        cropping: bool = True,
    ):
        """Tokenize text with <image> tags."""

        conversation = conversation
        assert conversation.count(self.image_token) == len(images)
        text_splits = conversation.split(self.image_token)
        images_list, images_crop_list, images_seq_mask, images_spatial_crop = (
            [],
            [],
            [],
            [],
        )
        image_shapes = []
        num_image_tokens = []
        tokenized_str = []
        for text_sep, image in zip(text_splits, images):
            """encode text_sep"""
            tokenized_sep = self.encode(text_sep, bos=False, eos=False)

            tokenized_str += tokenized_sep
            images_seq_mask += [False] * len(tokenized_sep)

            image_shapes.append(image.size)

            if image.size[0] <= 640 and image.size[1] <= 640:
                crop_ratio = [1, 1]
            else:
                if cropping:
                    images_crop_raw, crop_ratio = dynamic_preprocess(
                        image, image_size=IMAGE_SIZE
                    )
                else:
                    crop_ratio = [1, 1]

            """process the global view"""
            if self.image_size <= 640 and not cropping:
                image = image.resize((self.image_size, self.image_size))

            global_view = ImageOps.pad(
                image,
                (self.base_size, self.base_size),
                color=tuple(int(x * 255) for x in self.image_transform.mean),
            )
            images_list.append(self.image_transform(global_view))

            num_width_tiles, num_height_tiles = crop_ratio
            images_spatial_crop.append([num_width_tiles, num_height_tiles])

            if num_width_tiles > 1 or num_height_tiles > 1:
                for i in range(len(images_crop_raw)):
                    images_crop_list.append(self.image_transform(images_crop_raw[i]))

            """add image tokens"""
            num_queries = math.ceil(
                (self.image_size // self.patch_size) / self.downsample_ratio
            )
            num_queries_base = math.ceil(
                (self.base_size // self.patch_size) / self.downsample_ratio
            )
```
**EN:** This block defines function `DeepseekOCRProcessor.tokenize_with_images`. Parameters: self, conversation, images, bos, eos, cropping. Tokenize text with <image> tags. This subsection covers lines 428-493 of the same logical block.
**CN:** 该代码块定义函数 `DeepseekOCRProcessor.tokenize_with_images`。 参数包括 self、conversation、images、bos、eos、cropping。 文档字符串摘要：Tokenize text with <image> tags. 本小节覆盖同一逻辑块中的第 428-493 行。

### Lines 494-494: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 495-569: Defines function DeepseekOCRProcessor.tokenize_with_images (part 2)
```python
            if self.ocr2_mode:
                tokenized_image = []
                if num_width_tiles > 1 or num_height_tiles > 1:
                    tokenized_image += [self.image_token_id] * (
                        num_queries * num_width_tiles * num_queries * num_height_tiles
                    )
                tokenized_image += [self.image_token_id] * (
                    num_queries_base * num_queries_base
                )
                # One extra token for the view separator.
                tokenized_image += [self.image_token_id]
            else:
                tokenized_image = (
                    [self.image_token_id] * num_queries_base + [self.image_token_id]
                ) * num_queries_base
                tokenized_image += [self.image_token_id]
                if num_width_tiles > 1 or num_height_tiles > 1:
                    tokenized_image += (
                        [self.image_token_id] * (num_queries * num_width_tiles)
                        + [self.image_token_id]
                    ) * (num_queries * num_height_tiles)
            tokenized_str += tokenized_image

            images_seq_mask += [True] * len(tokenized_image)
            num_image_tokens.append(len(tokenized_image))

        """process the last text split"""
        tokenized_sep = self.encode(text_splits[-1], bos=False, eos=False)

        tokenized_str += tokenized_sep
        images_seq_mask += [False] * len(tokenized_sep)

        """add the bos and eos tokens"""
        if bos:
            tokenized_str = [self.bos_id] + tokenized_str
            images_seq_mask = [False] + images_seq_mask
        if eos:
            tokenized_str = tokenized_str + [self.eos_id]
            images_seq_mask = images_seq_mask + [False]

        assert len(tokenized_str) == len(
            images_seq_mask
        ), f"tokenize_with_images func: tokenized_str's length {len(tokenized_str)} is not equal to imags_seq_mask's length {len(images_seq_mask)}"

        masked_tokenized_str = []
        for token_index in tokenized_str:
            if token_index != self.image_token_id:
                masked_tokenized_str.append(token_index)
            else:
                masked_tokenized_str.append(self.ignore_id)

        assert (
            len(tokenized_str) == len(images_seq_mask) == len(masked_tokenized_str)
        ), (
            f"tokenized_str's length {len(tokenized_str)}, input_ids' length {len(masked_tokenized_str)}, "
            f"imags_seq_mask's length {len(images_seq_mask)}, are not equal"
        )
        input_ids = torch.LongTensor(tokenized_str)
        target_ids = torch.LongTensor(masked_tokenized_str)
        images_seq_mask = torch.tensor(images_seq_mask, dtype=torch.bool)

        # set input_ids < 0 | input_ids == self.image_token_id as ignore_id
        target_ids[(input_ids < 0) | (input_ids == self.image_token_id)] = (
            self.ignore_id
        )
        input_ids[input_ids < 0] = self.pad_id

        inference_mode = True

        if inference_mode:
            # Remove the ending eos token
            assert input_ids[-1] == self.eos_id
            input_ids = input_ids[:-1]
            target_ids = target_ids[:-1]
            images_seq_mask = images_seq_mask[:-1]
```
**EN:** This block defines function `DeepseekOCRProcessor.tokenize_with_images`. Parameters: self, conversation, images, bos, eos, cropping. Tokenize text with <image> tags. This subsection covers lines 495-569 of the same logical block.
**CN:** 该代码块定义函数 `DeepseekOCRProcessor.tokenize_with_images`。 参数包括 self、conversation、images、bos、eos、cropping。 文档字符串摘要：Tokenize text with <image> tags. 本小节覆盖同一逻辑块中的第 495-569 行。

### Lines 570-570: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 571-585: Defines function DeepseekOCRProcessor.tokenize_with_images (part 3)
```python
        if len(images_list) == 0:
            pixel_values = torch.zeros((1, 3, self.base_size, self.base_size))
            images_spatial_crop = torch.zeros((1, 1), dtype=torch.long)
            images_crop = torch.zeros(
                (1, 3, self.image_size, self.image_size)
            ).unsqueeze(0)
        else:
            pixel_values = torch.stack(images_list, dim=0)
            images_spatial_crop = torch.tensor(images_spatial_crop, dtype=torch.long)
            if images_crop_list:
                images_crop = torch.stack(images_crop_list, dim=0).unsqueeze(0)
            else:
                images_crop = torch.zeros(
                    (1, 3, self.image_size, self.image_size)
                ).unsqueeze(0)
```
**EN:** This block defines function `DeepseekOCRProcessor.tokenize_with_images`. Parameters: self, conversation, images, bos, eos, cropping. Tokenize text with <image> tags. This subsection covers lines 571-585 of the same logical block.
**CN:** 该代码块定义函数 `DeepseekOCRProcessor.tokenize_with_images`。 参数包括 self、conversation、images、bos、eos、cropping。 文档字符串摘要：Tokenize text with <image> tags. 本小节覆盖同一逻辑块中的第 571-585 行。

### Lines 586-586: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 587-596: Defines function DeepseekOCRProcessor.tokenize_with_images (part 4)
```python
        input_ids = input_ids.unsqueeze(0)
        return (
            input_ids,
            pixel_values,
            images_crop,
            images_seq_mask,
            images_spatial_crop,
            num_image_tokens,
            image_shapes,
        )
```
**EN:** This block defines function `DeepseekOCRProcessor.tokenize_with_images`. Parameters: self, conversation, images, bos, eos, cropping. Tokenize text with <image> tags. This subsection covers lines 587-596 of the same logical block.
**CN:** 该代码块定义函数 `DeepseekOCRProcessor.tokenize_with_images`。 参数包括 self、conversation、images、bos、eos、cropping。 文档字符串摘要：Tokenize text with <image> tags. 本小节覆盖同一逻辑块中的第 587-596 行。

### Lines 597-598: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 599-599: Declares class VisionEncoderConfig
```python
class VisionEncoderConfig(PretrainedConfig):
```
**EN:** This block introduces class `VisionEncoderConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `VisionEncoderConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 600-600: Declares model_type
```python
    model_type: str = "vision"
```
**EN:** This statement initializes model_type in the VisionEncoderConfig. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 VisionEncoderConfig 中初始化 model_type。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 601-601: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VisionEncoderConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VisionEncoderConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 602-616: Declares model_name, image_size, patch_size, width, layers, and 10 more
```python
    model_name: str = "vit_so400m_patch14_siglip_384.webli"
    image_size: int = 384
    patch_size: int = 16
    width: int = 1024
    layers: int = 24
    heads: int = 16
    mlp_ratio: int = 4
    global_pool: str = "map"
    ignore_head: bool = True
    class_token: bool = False
    num_classes: int = 0
    use_checkpoint: bool = False
    weight_init: str = "skip"
    deterministic: bool = False
    num_recomputing_layers: int = 0
```
**EN:** This block initializes a related set of values in the VisionEncoderConfig, including model_name, image_size, patch_size, width, layers, and 10 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 VisionEncoderConfig 中初始化一组相关值，包括 model_name, image_size, patch_size, width, layers 等 10 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 617-617: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VisionEncoderConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VisionEncoderConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 618-647: Defines function VisionEncoderConfig.__init__
```python
    def __init__(
        self,
        model_name: str = "vit_so400m_patch14_siglip_384.webli",
        image_size: int = 384,
        patch_size: int = 16,
        width: int = 1024,
        layers: int = 24,
        heads: int = 16,
        mlp_ratio: int = 4,
        global_pool: str = "map",
        ignore_head: bool = True,
        class_token: bool = False,
        num_classes: int = 0,
        use_checkpoint: bool = False,
        **kwargs,
    ):
        self.model_name = model_name
        self.image_size = image_size
        self.patch_size = patch_size
        self.width = width
        self.layers = layers
        self.heads = heads
        self.mlp_ratio = mlp_ratio
        self.global_pool = global_pool
        self.ignore_head = ignore_head
        self.class_token = class_token
        self.num_classes = num_classes
        self.use_checkpoint = use_checkpoint

        super().__init__(**kwargs)
```
**EN:** This block defines function `VisionEncoderConfig.__init__`. Parameters: self, model_name, image_size, patch_size, width, layers, heads, mlp_ratio, global_pool, ignore_head, class_token, num_classes, use_checkpoint.
**CN:** 该代码块定义函数 `VisionEncoderConfig.__init__`。 参数包括 self、model_name、image_size、patch_size、width、layers、heads、mlp_ratio、global_pool、ignore_head、class_token、num_classes、use_checkpoint。

### Lines 648-649: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 650-650: Declares class MlpProjectorConfig
```python
class MlpProjectorConfig(PretrainedConfig):
```
**EN:** This block introduces class `MlpProjectorConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `MlpProjectorConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 651-658: Declares model_type, projector_type, input_dim, n_embed, depth, and 3 more
```python
    model_type = "mlp_projector"
    projector_type: str = "downsample_mlp_gelu"
    input_dim: int = 1152
    n_embed: int = 2048
    depth: int = 2
    mlp_ratio: int = 1
    downsample_ratio: int = 2
    token_pooling: bool = False
```
**EN:** This block initializes a related set of values in the MlpProjectorConfig, including model_type, projector_type, input_dim, n_embed, depth, and 3 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 MlpProjectorConfig 中初始化一组相关值，包括 model_type, projector_type, input_dim, n_embed, depth 等 3 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 659-659: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MlpProjectorConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MlpProjectorConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 660-677: Defines function MlpProjectorConfig.__init__
```python
    def __init__(
        self,
        projector_type: str = "downsample_mlp_gelu",
        input_dim: int = 1152,
        n_embed: int = 2048,
        depth: int = 2,
        mlp_ratio: int = 1,
        downsample_ratio: int = 2,
        **kwargs,
    ):
        self.projector_type = projector_type
        self.input_dim = input_dim
        self.n_embed = n_embed
        self.depth = depth
        self.mlp_ratio = mlp_ratio
        self.downsample_ratio = downsample_ratio

        super().__init__(**kwargs)
```
**EN:** This block defines function `MlpProjectorConfig.__init__`. Parameters: self, projector_type, input_dim, n_embed, depth, mlp_ratio, downsample_ratio.
**CN:** 该代码块定义函数 `MlpProjectorConfig.__init__`。 参数包括 self、projector_type、input_dim、n_embed、depth、mlp_ratio、downsample_ratio。

### Lines 678-679: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 680-680: Declares class DeepseekV2Config
```python
class DeepseekV2Config(PretrainedConfig):
```
**EN:** This block introduces class `DeepseekV2Config` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `DeepseekV2Config`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 681-682: Declares model_type, keys_to_ignore_at_inference
```python
    model_type = "deepseek_v2"
    keys_to_ignore_at_inference = ["past_key_values"]
```
**EN:** This block initializes a related set of values in the DeepseekV2Config, including model_type, keys_to_ignore_at_inference. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepseekV2Config 中初始化一组相关值，包括 model_type, keys_to_ignore_at_inference。将这些赋值集中在一起有助于理解周边配置。

### Lines 683-683: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekV2Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekV2Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 684-757: Defines function DeepseekV2Config.__init__ (part 1)
```python
    def __init__(
        self,
        vocab_size=102400,
        hidden_size=4096,
        intermediate_size=11008,
        moe_intermediate_size=1407,
        num_hidden_layers=30,
        num_attention_heads=32,
        num_key_value_heads=32,
        n_shared_experts=None,
        n_routed_experts=None,
        ep_size=1,
        routed_scaling_factor=1.0,
        kv_lora_rank=512,
        q_lora_rank=1536,
        qk_rope_head_dim=64,
        v_head_dim=128,
        qk_nope_head_dim=128,
        topk_method="gready",
        n_group=None,
        topk_group=None,
        num_experts_per_tok=None,
        moe_layer_freq=1,
        first_k_dense_replace=0,
        norm_topk_prob=False,
        scoring_func="softmax",
        aux_loss_alpha=0.001,
        seq_aux=True,
        hidden_act="silu",
        max_position_embeddings=2048,
        initializer_range=0.02,
        rms_norm_eps=1e-6,
        use_cache=True,
        pad_token_id=None,
        bos_token_id=100000,
        eos_token_id=100001,
        pretraining_tp=1,
        tie_word_embeddings=False,
        rope_theta=10000.0,
        rope_scaling=None,
        attention_bias=False,
        attention_dropout=0.0,
        use_mla=True,
        **kwargs,
    ):
        self.vocab_size = vocab_size
        self.max_position_embeddings = max_position_embeddings
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size
        self.moe_intermediate_size = moe_intermediate_size
        self.num_hidden_layers = num_hidden_layers
        self.num_attention_heads = num_attention_heads
        self.n_shared_experts = n_shared_experts
        self.n_routed_experts = n_routed_experts
        self.ep_size = ep_size
        self.routed_scaling_factor = routed_scaling_factor
        self.kv_lora_rank = kv_lora_rank
        self.q_lora_rank = q_lora_rank
        self.qk_rope_head_dim = qk_rope_head_dim
        self.v_head_dim = v_head_dim
        self.qk_nope_head_dim = qk_nope_head_dim
        self.topk_method = topk_method
        self.n_group = n_group
        self.topk_group = topk_group
        self.num_experts_per_tok = num_experts_per_tok
        self.moe_layer_freq = moe_layer_freq
        self.first_k_dense_replace = first_k_dense_replace
        self.norm_topk_prob = norm_topk_prob
        self.scoring_func = scoring_func
        self.aux_loss_alpha = aux_loss_alpha
        self.seq_aux = seq_aux
        # for backward compatibility
        if num_key_value_heads is None:
            num_key_value_heads = num_attention_heads
```
**EN:** This block defines function `DeepseekV2Config.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, moe_intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, n_shared_experts, n_routed_experts, ep_size, routed_scaling_factor, kv_lora_rank, q_lora_rank, qk_rope_head_dim, v_head_dim, qk_nope_head_dim, topk_method, n_group, topk_group, num_experts_per_tok, moe_layer_freq, first_k_dense_replace, norm_topk_prob, scoring_func, aux_loss_alpha, seq_aux, hidden_act, max_position_embeddings, initializer_range, rms_norm_eps, use_cache, pad_token_id, bos_token_id, eos_token_id, pretraining_tp, tie_word_embeddings, rope_theta, rope_scaling, attention_bias, attention_dropout, use_mla. This subsection covers lines 684-757 of the same logical block.
**CN:** 该代码块定义函数 `DeepseekV2Config.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、moe_intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、n_shared_experts、n_routed_experts、ep_size、routed_scaling_factor、kv_lora_rank、q_lora_rank、qk_rope_head_dim、v_head_dim、qk_nope_head_dim、topk_method、n_group、topk_group、num_experts_per_tok、moe_layer_freq、first_k_dense_replace、norm_topk_prob、scoring_func、aux_loss_alpha、seq_aux、hidden_act、max_position_embeddings、initializer_range、rms_norm_eps、use_cache、pad_token_id、bos_token_id、eos_token_id、pretraining_tp、tie_word_embeddings、rope_theta、rope_scaling、attention_bias、attention_dropout、use_mla。 本小节覆盖同一逻辑块中的第 684-757 行。

### Lines 758-758: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 759-769: Defines function DeepseekV2Config.__init__ (part 2)
```python
        self.num_key_value_heads = num_key_value_heads
        self.hidden_act = hidden_act
        self.initializer_range = initializer_range
        self.rms_norm_eps = float(rms_norm_eps)
        self.pretraining_tp = pretraining_tp
        self.use_cache = use_cache
        self.rope_theta = rope_theta
        self.rope_scaling = rope_scaling
        self.attention_bias = attention_bias
        self.attention_dropout = attention_dropout
        self.use_mla = use_mla
```
**EN:** This block defines function `DeepseekV2Config.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, moe_intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, n_shared_experts, n_routed_experts, ep_size, routed_scaling_factor, kv_lora_rank, q_lora_rank, qk_rope_head_dim, v_head_dim, qk_nope_head_dim, topk_method, n_group, topk_group, num_experts_per_tok, moe_layer_freq, first_k_dense_replace, norm_topk_prob, scoring_func, aux_loss_alpha, seq_aux, hidden_act, max_position_embeddings, initializer_range, rms_norm_eps, use_cache, pad_token_id, bos_token_id, eos_token_id, pretraining_tp, tie_word_embeddings, rope_theta, rope_scaling, attention_bias, attention_dropout, use_mla. This subsection covers lines 759-769 of the same logical block.
**CN:** 该代码块定义函数 `DeepseekV2Config.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、moe_intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、n_shared_experts、n_routed_experts、ep_size、routed_scaling_factor、kv_lora_rank、q_lora_rank、qk_rope_head_dim、v_head_dim、qk_nope_head_dim、topk_method、n_group、topk_group、num_experts_per_tok、moe_layer_freq、first_k_dense_replace、norm_topk_prob、scoring_func、aux_loss_alpha、seq_aux、hidden_act、max_position_embeddings、initializer_range、rms_norm_eps、use_cache、pad_token_id、bos_token_id、eos_token_id、pretraining_tp、tie_word_embeddings、rope_theta、rope_scaling、attention_bias、attention_dropout、use_mla。 本小节覆盖同一逻辑块中的第 759-769 行。

### Lines 770-770: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 771-777: Defines function DeepseekV2Config.__init__ (part 3)
```python
        super().__init__(
            pad_token_id=pad_token_id,
            bos_token_id=bos_token_id,
            eos_token_id=eos_token_id,
            tie_word_embeddings=tie_word_embeddings,
            **kwargs,
        )
```
**EN:** This block defines function `DeepseekV2Config.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, moe_intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, n_shared_experts, n_routed_experts, ep_size, routed_scaling_factor, kv_lora_rank, q_lora_rank, qk_rope_head_dim, v_head_dim, qk_nope_head_dim, topk_method, n_group, topk_group, num_experts_per_tok, moe_layer_freq, first_k_dense_replace, norm_topk_prob, scoring_func, aux_loss_alpha, seq_aux, hidden_act, max_position_embeddings, initializer_range, rms_norm_eps, use_cache, pad_token_id, bos_token_id, eos_token_id, pretraining_tp, tie_word_embeddings, rope_theta, rope_scaling, attention_bias, attention_dropout, use_mla. This subsection covers lines 771-777 of the same logical block.
**CN:** 该代码块定义函数 `DeepseekV2Config.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、moe_intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、n_shared_experts、n_routed_experts、ep_size、routed_scaling_factor、kv_lora_rank、q_lora_rank、qk_rope_head_dim、v_head_dim、qk_nope_head_dim、topk_method、n_group、topk_group、num_experts_per_tok、moe_layer_freq、first_k_dense_replace、norm_topk_prob、scoring_func、aux_loss_alpha、seq_aux、hidden_act、max_position_embeddings、initializer_range、rms_norm_eps、use_cache、pad_token_id、bos_token_id、eos_token_id、pretraining_tp、tie_word_embeddings、rope_theta、rope_scaling、attention_bias、attention_dropout、use_mla。 本小节覆盖同一逻辑块中的第 771-777 行。

### Lines 778-779: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 780-782: Declares class DeepseekVLV2Config
```python
@register_customized_processor(processor_class=DeepseekOCRProcessor)
class DeepseekVLV2Config(PretrainedConfig):
    # model_type = "deepseek_vl_v2"
```
**EN:** This block introduces class `DeepseekVLV2Config` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `DeepseekVLV2Config`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 783-785: Declares model_type, vision_config, projector_config
```python
    model_type = "deepseek-ocr"
    vision_config: VisionEncoderConfig = None
    projector_config: MlpProjectorConfig = None
```
**EN:** This block initializes a related set of values in the DeepseekVLV2Config, including model_type, vision_config, projector_config. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepseekVLV2Config 中初始化一组相关值，包括 model_type, vision_config, projector_config。将这些赋值集中在一起有助于理解周边配置。

### Lines 786-786: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekVLV2Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekVLV2Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 787-790: Declares tile_tag, global_view_pos, candidate_resolutions, customized_processor_type
```python
    tile_tag: str = "2D"
    global_view_pos: str = "head"
    candidate_resolutions: tuple[tuple[int, int]] = ((384, 384),)
    customized_processor_type: type[Any] = DeepseekOCRProcessor
```
**EN:** This block initializes a related set of values in the DeepseekVLV2Config, including tile_tag, global_view_pos, candidate_resolutions, customized_processor_type. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepseekVLV2Config 中初始化一组相关值，包括 tile_tag, global_view_pos, candidate_resolutions, customized_processor_type。将这些赋值集中在一起有助于理解周边配置。

### Lines 791-791: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekVLV2Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekVLV2Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 792-814: Defines function DeepseekVLV2Config.__init__
```python
    def __init__(
        self,
        tile_tag: str = "tile_tag",
        global_view_pos: str = "head",
        candidate_resolutions: tuple[tuple[int, int]] = ((384, 384),),
        **kwargs,
    ):
        super().__init__(**kwargs)

        vision_config = kwargs.get("vision_config", {})
        self.vision_config = VisionEncoderConfig(**vision_config)

        projector_config = kwargs.get("projector_config", {})
        self.projector_config = MlpProjectorConfig(**projector_config)

        language_config = kwargs.get("language_config", {})
        self.text_config = DeepseekV2Config(**language_config)

        self.tile_tag = tile_tag
        self.global_view_pos = global_view_pos
        self.candidate_resolutions = candidate_resolutions
        self.vocab_size = self.text_config.vocab_size
        self.hidden_size = self.text_config.hidden_size
```
**EN:** This block defines function `DeepseekVLV2Config.__init__`. Parameters: self, tile_tag, global_view_pos, candidate_resolutions.
**CN:** 该代码块定义函数 `DeepseekVLV2Config.__init__`。 参数包括 self、tile_tag、global_view_pos、candidate_resolutions。

### Lines 815-816: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 817-817: Implements Expr
```python
AutoProcessor.register(DeepseekVLV2Config, DeepseekOCRProcessor)
```
**EN:** This block contributes general-purpose logic to the module using a `Expr` statement.
**CN:** 该代码块通过 `Expr` 语句为 模块 补充通用逻辑。

## Key Concepts / 关键概念
- **Classes / 类**: `DictOutput`, `VLChatProcessorOutput`, `ImageTransform`, `DeepseekOCRProcessor`, `VisionEncoderConfig`, `MlpProjectorConfig`, `DeepseekV2Config`, `DeepseekVLV2Config`
- **Functions / 函数**: `get_default_ngram_custom_params`, `find_closest_aspect_ratio`, `dynamic_preprocess`
- **Constants / 常量**: `BASE_SIZE`, `IMAGE_SIZE`, `CROP_MODE`, `MIN_CROPS`, `MAX_CROPS`, `MAX_CONCURRENCY`, `NUM_WORKERS`, `PRINT_NUM_VIS_TOKENS`, `SKIP_REPEAT`, `MODEL_PATH`, `NGRAM_NO_REPEAT_SIZE`, `NGRAM_NO_REPEAT_WINDOW`, `NGRAM_NO_REPEAT_WHITELIST`, `DEFAULT_CUSTOM_LOGIT_PROCESSOR`, `PROMPT`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `dataclasses`, `math`, `typing`
- **Third-Party / 第三方**: `PIL`, `torch`, `torchvision.transforms`, `transformers`
- **Local Modules / 本地模块**: `sglang.srt.multimodal.customized_mm_processor_utils`, `sglang.srt.sampling.custom_logit_processor`
