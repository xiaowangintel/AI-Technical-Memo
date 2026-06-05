# deepseekvl2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/deepseekvl2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for deepseekvl2 so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 deepseekvl2 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12: Imports dependencies
```python
import math
from dataclasses import dataclass
from typing import Dict, List, Optional, Tuple

import torch
from PIL import Image, ImageOps
from transformers import (
    AutoProcessor,
    LlamaTokenizerFast,
    PretrainedConfig,
    ProcessorMixin,
)
```
**EN:** This block groups related imports for the module, including math, dataclasses.dataclass, typing.Dict, typing.List, typing.Optional, and 8 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 math, dataclasses.dataclass, typing.Dict, typing.List, typing.Optional 等 8 项，为后续代码准备所需名称。

### Lines 13-14: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 15-40: Defines function select_best_resolution
```python
def select_best_resolution(image_size, candidate_resolutions):
    # used for cropping
    original_width, original_height = image_size
    best_fit = None
    max_effective_resolution = 0
    min_wasted_resolution = float("inf")

    for width, height in candidate_resolutions:
        scale = min(width / original_width, height / original_height)
        downscaled_width, downscaled_height = int(original_width * scale), int(
            original_height * scale
        )
        effective_resolution = min(
            downscaled_width * downscaled_height, original_width * original_height
        )
        wasted_resolution = (width * height) - effective_resolution

        if effective_resolution > max_effective_resolution or (
            effective_resolution == max_effective_resolution
            and wasted_resolution < min_wasted_resolution
        ):
            max_effective_resolution = effective_resolution
            min_wasted_resolution = wasted_resolution
            best_fit = (width, height)

    return best_fit
```
**EN:** This block defines function `select_best_resolution`. Parameters: image_size, candidate_resolutions.
**CN:** 该代码块定义函数 `select_best_resolution`。 参数包括 image_size、candidate_resolutions。

### Lines 41-42: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 43-43: Declares class DictOutput
```python
class DictOutput(object):
```
**EN:** This block introduces class `DictOutput` as a reusable abstraction inside the module. It inherits from object.
**CN:** 该代码块声明类 `DictOutput`，作为模块中的可复用抽象。 它继承自 object。

### Lines 44-45: Defines function DictOutput.items
```python
    def items(self):
        return self.__dict__.items()
```
**EN:** This block defines function `DictOutput.items`. Parameters: self.
**CN:** 该代码块定义函数 `DictOutput.items`。 参数包括 self。

### Lines 46-46: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DictOutput, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DictOutput 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 47-48: Defines function DictOutput.keys
```python
    def keys(self):
        return self.__dict__.keys()
```
**EN:** This block defines function `DictOutput.keys`. Parameters: self.
**CN:** 该代码块定义函数 `DictOutput.keys`。 参数包括 self。

### Lines 49-49: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DictOutput, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DictOutput 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 50-51: Defines function DictOutput.__getitem__
```python
    def __getitem__(self, item):
        return self.__dict__[item]
```
**EN:** This block defines function `DictOutput.__getitem__`. Parameters: self, item.
**CN:** 该代码块定义函数 `DictOutput.__getitem__`。 参数包括 self、item。

### Lines 52-52: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DictOutput, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DictOutput 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 53-54: Defines function DictOutput.__contains__
```python
    def __contains__(self, key):
        return key in self.__dict__
```
**EN:** This block defines function `DictOutput.__contains__`. Parameters: self, key.
**CN:** 该代码块定义函数 `DictOutput.__contains__`。 参数包括 self、key。

### Lines 55-55: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DictOutput, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DictOutput 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 56-57: Defines function DictOutput.__setitem__
```python
    def __setitem__(self, key, value):
        self.__dict__[key] = value
```
**EN:** This block defines function `DictOutput.__setitem__`. Parameters: self, key, value.
**CN:** 该代码块定义函数 `DictOutput.__setitem__`。 参数包括 self、key、value。

### Lines 58-59: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 60-61: Declares class VLChatProcessorOutput
```python
@dataclass
class VLChatProcessorOutput(DictOutput):
```
**EN:** This block introduces class `VLChatProcessorOutput` as a reusable abstraction inside the module. It inherits from DictOutput.
**CN:** 该代码块声明类 `VLChatProcessorOutput`，作为模块中的可复用抽象。 它继承自 DictOutput。

### Lines 62-68: Declares input_ids, target_ids, pixel_values, images_seq_mask, images_spatial_crop
```python
    input_ids: torch.LongTensor
    target_ids: torch.LongTensor
    pixel_values: (
        torch.Tensor
    )  # rename from "images" to "pixel_values" for compatibility
    images_seq_mask: torch.BoolTensor
    images_spatial_crop: torch.LongTensor
```
**EN:** This block initializes a related set of values in the VLChatProcessorOutput, including input_ids, target_ids, pixel_values, images_seq_mask, images_spatial_crop. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 VLChatProcessorOutput 中初始化一组相关值，包括 input_ids, target_ids, pixel_values, images_seq_mask, images_spatial_crop。将这些赋值集中在一起有助于理解周边配置。

### Lines 69-69: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLChatProcessorOutput, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLChatProcessorOutput 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 70-71: Defines function VLChatProcessorOutput.__len__
```python
    def __len__(self):
        return len(self.input_ids)
```
**EN:** This block defines function `VLChatProcessorOutput.__len__`. Parameters: self.
**CN:** 该代码块定义函数 `VLChatProcessorOutput.__len__`。 参数包括 self。

### Lines 72-73: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 74-74: Declares class ImageTransform
```python
class ImageTransform(object):
```
**EN:** This block introduces class `ImageTransform` as a reusable abstraction inside the module. It inherits from object.
**CN:** 该代码块声明类 `ImageTransform`，作为模块中的可复用抽象。 它继承自 object。

### Lines 75-100: Defines function ImageTransform.__init__
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

### Lines 101-101: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the ImageTransform, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 ImageTransform 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 102-104: Defines function ImageTransform.__call__
```python
    def __call__(self, pil_img: Image.Image):
        x = self.transform(pil_img)
        return x
```
**EN:** This block defines function `ImageTransform.__call__`. Parameters: self, pil_img.
**CN:** 该代码块定义函数 `ImageTransform.__call__`。 参数包括 self、pil_img。

### Lines 105-106: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 107-107: Declares class DeepseekVLV2Processor
```python
class DeepseekVLV2Processor(ProcessorMixin):
```
**EN:** This block introduces class `DeepseekVLV2Processor` as a reusable abstraction inside the module. It inherits from ProcessorMixin.
**CN:** 该代码块声明类 `DeepseekVLV2Processor`，作为模块中的可复用抽象。 它继承自 ProcessorMixin。

### Lines 108-109: Declares tokenizer_class, attributes
```python
    tokenizer_class = ("LlamaTokenizer", "LlamaTokenizerFast")
    attributes = ["tokenizer"]
```
**EN:** This block initializes a related set of values in the DeepseekVLV2Processor, including tokenizer_class, attributes. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepseekVLV2Processor 中初始化一组相关值，包括 tokenizer_class, attributes。将这些赋值集中在一起有助于理解周边配置。

### Lines 110-110: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekVLV2Processor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekVLV2Processor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 111-177: Defines function DeepseekVLV2Processor.__init__
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
        **kwargs,
    ):

        self.candidate_resolutions = candidate_resolutions
        self.image_size = candidate_resolutions[0][0]
        self.patch_size = patch_size
        self.image_mean = image_mean
        self.image_std = image_std
        self.normalize = normalize
        self.downsample_ratio = downsample_ratio

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

        super().__init__(
            tokenizer,
            **kwargs,
        )
```
**EN:** This block defines function `DeepseekVLV2Processor.__init__`. Parameters: self, tokenizer, candidate_resolutions, patch_size, downsample_ratio, image_mean, image_std, normalize, image_token, pad_token, add_special_token, sft_format, mask_prompt, ignore_id.
**CN:** 该代码块定义函数 `DeepseekVLV2Processor.__init__`。 参数包括 self、tokenizer、candidate_resolutions、patch_size、downsample_ratio、image_mean、image_std、normalize、image_token、pad_token、add_special_token、sft_format、mask_prompt、ignore_id。

### Lines 178-178: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekVLV2Processor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekVLV2Processor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 179-218: Defines function DeepseekVLV2Processor.format_messages_v2
```python
    def format_messages_v2(self, messages, pil_images, max_req_input_len=-1):
        """play the role of format_messages_v2 and get_images_info in the last version"""
        tokenized_data = []
        masked_tokenized_data = []  # labels
        images_list = []
        images_seq_mask = []
        images_spatial_crop = []

        image_index = 0
        image_token_cnt = messages.count(self.image_token)
        tokenized_str, images, seq_mask, spatial_crop = self.tokenize_with_images(
            messages,
            pil_images[image_index : image_index + image_token_cnt],
            bos=True,
            eos=True,
            cropping=len(pil_images) <= 2,
            max_req_input_len=max_req_input_len,
        )

        image_index = image_token_cnt
        tokenized_data += tokenized_str
        if self.mask_prompt:
            masked_tokenized_data += [self.ignore_id] * len(tokenized_str)
        else:
            masked_tokenized_data += tokenized_str
        images_list += images
        images_seq_mask += seq_mask
        images_spatial_crop += spatial_crop

        assert len(tokenized_data) == len(
            images_seq_mask
        ), f"format_messages_v2: tokenized_str's length {len(tokenized_str)} is not equal to imags_seq_mask's length {len(images_seq_mask)}"

        return (
            tokenized_data,
            masked_tokenized_data,
            images_list,
            images_seq_mask,
            images_spatial_crop,
        )
```
**EN:** This block defines function `DeepseekVLV2Processor.format_messages_v2`. Parameters: self, messages, pil_images, max_req_input_len. play the role of format_messages_v2 and get_images_info in the last version
**CN:** 该代码块定义函数 `DeepseekVLV2Processor.format_messages_v2`。 参数包括 self、messages、pil_images、max_req_input_len。 文档字符串摘要：play the role of format_messages_v2 and get_images_info in the last version

### Lines 219-219: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekVLV2Processor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekVLV2Processor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 220-222: Defines function DeepseekVLV2Processor.bos_id
```python
    @property
    def bos_id(self):
        return self.tokenizer.bos_token_id
```
**EN:** This block defines function `DeepseekVLV2Processor.bos_id`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `DeepseekVLV2Processor.bos_id`。 参数包括 self。 装饰器包括 property。

### Lines 223-223: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekVLV2Processor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekVLV2Processor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 224-226: Defines function DeepseekVLV2Processor.eos_id
```python
    @property
    def eos_id(self):
        return self.tokenizer.eos_token_id
```
**EN:** This block defines function `DeepseekVLV2Processor.eos_id`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `DeepseekVLV2Processor.eos_id`。 参数包括 self。 装饰器包括 property。

### Lines 227-227: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekVLV2Processor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekVLV2Processor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 228-230: Defines function DeepseekVLV2Processor.pad_id
```python
    @property
    def pad_id(self):
        return self.tokenizer.pad_token_id
```
**EN:** This block defines function `DeepseekVLV2Processor.pad_id`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `DeepseekVLV2Processor.pad_id`。 参数包括 self。 装饰器包括 property。

### Lines 231-231: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekVLV2Processor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekVLV2Processor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 232-240: Defines function DeepseekVLV2Processor.encode
```python
    def encode(self, text: str, bos: bool = True, eos: bool = False):
        t = self.tokenizer.encode(text, add_special_tokens=False)

        if bos:
            t = [self.bos_id] + t
        if eos:
            t = t + [self.eos_id]

        return t
```
**EN:** This block defines function `DeepseekVLV2Processor.encode`. Parameters: self, text, bos, eos.
**CN:** 该代码块定义函数 `DeepseekVLV2Processor.encode`。 参数包括 self、text、bos、eos。

### Lines 241-241: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekVLV2Processor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekVLV2Processor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 242-243: Defines function DeepseekVLV2Processor.decode
```python
    def decode(self, t: List[int], **kwargs) -> str:
        return self.tokenizer.decode(t, **kwargs)
```
**EN:** This block defines function `DeepseekVLV2Processor.decode`. Parameters: self, t.
**CN:** 该代码块定义函数 `DeepseekVLV2Processor.decode`。 参数包括 self、t。

### Lines 244-244: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekVLV2Processor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekVLV2Processor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 245-321: Defines function DeepseekVLV2Processor.process_one (part 1)
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

        assert (
            prompt is None or conversations is None
        ), "prompt and conversations cannot be used at the same time."

        (
            tokenized_str,
            masked_tokenized_str,
            images_list,
            images_seq_mask,
            images_spatial_crop,
        ) = self.format_messages_v2(conversations, images, max_req_input_len)

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

        if inference_mode:
            assert input_ids[-1] == self.eos_id
            input_ids = input_ids[:-1]
            target_ids = target_ids[:-1]
            images_seq_mask = images_seq_mask[:-1]

        if len(images_list) == 0:
            images = torch.zeros((1, 3, self.image_size, self.image_size))
            images_spatial_crop = torch.zeros((1, 2), dtype=torch.long)
        else:
            images = torch.stack(images_list, dim=0)
            images_spatial_crop = torch.tensor(images_spatial_crop, dtype=torch.long)

        images_spatial_crop = torch.stack(
            [images_spatial_crop], dim=0
        )  # stack the tensor to make it a batch of 1
```
**EN:** This block defines function `DeepseekVLV2Processor.process_one`. Parameters: self, prompt, conversations, images, apply_sft_format, inference_mode, system_prompt, max_req_input_len. Args: prompt (str): the formatted prompt; conversations (List[Dict]): conversations with a list of messages; images (List[ImageType]): the list of images; apply_sft_format (bool): if prompt is not None, then apply the SFT format to prompt; if conversations is This subsection covers lines 245-321 of the same logical block.
**CN:** 该代码块定义函数 `DeepseekVLV2Processor.process_one`。 参数包括 self、prompt、conversations、images、apply_sft_format、inference_mode、system_prompt、max_req_input_len。 文档字符串摘要：Args: prompt (str): the formatted prompt; conversations (List[Dict]): conversations with a list of messages; images (List[ImageType]): the list of images; apply_sft_format (bool): if prompt is not None, then apply the SFT format to prompt; if conversations is 本小节覆盖同一逻辑块中的第 245-321 行。

### Lines 322-322: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 323-329: Defines function DeepseekVLV2Processor.process_one (part 2)
```python
        prepare = VLChatProcessorOutput(
            input_ids=input_ids,
            target_ids=target_ids,
            pixel_values=images,
            images_seq_mask=images_seq_mask,
            images_spatial_crop=images_spatial_crop,
        )
```
**EN:** This block defines function `DeepseekVLV2Processor.process_one`. Parameters: self, prompt, conversations, images, apply_sft_format, inference_mode, system_prompt, max_req_input_len. Args: prompt (str): the formatted prompt; conversations (List[Dict]): conversations with a list of messages; images (List[ImageType]): the list of images; apply_sft_format (bool): if prompt is not None, then apply the SFT format to prompt; if conversations is This subsection covers lines 323-329 of the same logical block.
**CN:** 该代码块定义函数 `DeepseekVLV2Processor.process_one`。 参数包括 self、prompt、conversations、images、apply_sft_format、inference_mode、system_prompt、max_req_input_len。 文档字符串摘要：Args: prompt (str): the formatted prompt; conversations (List[Dict]): conversations with a list of messages; images (List[ImageType]): the list of images; apply_sft_format (bool): if prompt is not None, then apply the SFT format to prompt; if conversations is 本小节覆盖同一逻辑块中的第 323-329 行。

### Lines 330-330: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 331-331: Defines function DeepseekVLV2Processor.process_one (part 3)
```python
        return prepare
```
**EN:** This block defines function `DeepseekVLV2Processor.process_one`. Parameters: self, prompt, conversations, images, apply_sft_format, inference_mode, system_prompt, max_req_input_len. Args: prompt (str): the formatted prompt; conversations (List[Dict]): conversations with a list of messages; images (List[ImageType]): the list of images; apply_sft_format (bool): if prompt is not None, then apply the SFT format to prompt; if conversations is This subsection covers lines 331-331 of the same logical block.
**CN:** 该代码块定义函数 `DeepseekVLV2Processor.process_one`。 参数包括 self、prompt、conversations、images、apply_sft_format、inference_mode、system_prompt、max_req_input_len。 文档字符串摘要：Args: prompt (str): the formatted prompt; conversations (List[Dict]): conversations with a list of messages; images (List[ImageType]): the list of images; apply_sft_format (bool): if prompt is not None, then apply the SFT format to prompt; if conversations is 本小节覆盖同一逻辑块中的第 331-331 行。

### Lines 332-332: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekVLV2Processor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekVLV2Processor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 333-355: Defines function DeepseekVLV2Processor.__call__
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
        **kwargs,
    ):
        prepare = self.process_one(
            prompt=prompt,
            conversations=conversations,
            images=images,
            apply_sft_format=apply_sft_format,
            inference_mode=inference_mode,
            system_prompt=system_prompt,
            max_req_input_len=max_req_input_len,
        )

        return prepare
```
**EN:** This block defines function `DeepseekVLV2Processor.__call__`. Parameters: self.
**CN:** 该代码块定义函数 `DeepseekVLV2Processor.__call__`。 参数包括 self。

### Lines 356-356: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekVLV2Processor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekVLV2Processor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 357-362: Defines function DeepseekVLV2Processor.find_all_indices
```python
    def find_all_indices(self, messages, target_value):
        indices = []
        for index, item in enumerate(messages):
            if item == target_value:
                indices.append(index)
        return indices
```
**EN:** This block defines function `DeepseekVLV2Processor.find_all_indices`. Parameters: self, messages, target_value.
**CN:** 该代码块定义函数 `DeepseekVLV2Processor.find_all_indices`。 参数包括 self、messages、target_value。

### Lines 363-363: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekVLV2Processor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekVLV2Processor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 364-440: Defines function DeepseekVLV2Processor.tokenize_with_images (part 1)
```python
    def tokenize_with_images(
        self,
        conversation: str,
        images: List[Image.Image],
        bos: bool = True,
        eos: bool = True,
        cropping: bool = True,
        max_req_input_len: int = -1,
    ):
        """Tokenize text with <image> tags."""
        images_list, images_seq_mask, images_spatial_crop = [], [], []
        text_splits = conversation.split(self.image_token)
        tokenized_str = []
        for text_sep, image in zip(text_splits, images):
            """encode text_sep"""
            tokenized_sep = self.encode(text_sep, bos=False, eos=False)
            tokenized_str += tokenized_sep
            images_seq_mask += [False] * len(tokenized_sep)

            """select best resolution for anyres"""
            if cropping:
                best_width, best_height = select_best_resolution(
                    image.size, self.candidate_resolutions
                )
            else:
                best_width, best_height = self.image_size, self.image_size
            # print(image.size, (best_width, best_height)) # check the select_best_resolutions func

            """process the global view"""
            global_view = ImageOps.pad(
                image,
                (self.image_size, self.image_size),
                color=tuple(int(x * 255) for x in self.image_transform.mean),
            )
            images_list.append(self.image_transform(global_view))

            """process the local views"""
            local_view = ImageOps.pad(
                image,
                (best_width, best_height),
                color=tuple(int(x * 255) for x in self.image_transform.mean),
            )
            for i in range(0, best_height, self.image_size):
                for j in range(0, best_width, self.image_size):
                    images_list.append(
                        self.image_transform(
                            local_view.crop(
                                (j, i, j + self.image_size, i + self.image_size)
                            )
                        )
                    )

            """record height / width crop num"""
            num_width_tiles, num_height_tiles = (
                best_width // self.image_size,
                best_height // self.image_size,
            )
            images_spatial_crop.append([num_width_tiles, num_height_tiles])

            """add image tokens"""
            h = w = math.ceil(
                (self.image_size // self.patch_size) / self.downsample_ratio
            )
            # global views tokens h * (w + 1), 1 is for line separator
            tokenized_image = [self.image_token_id] * h * (w + 1)
            # add a separator between global and local views
            tokenized_image += [self.image_token_id]
            # local views tokens, (num_height_tiles * h) * (num_width_tiles * w + 1)
            tokenized_image += (
                [self.image_token_id]
                * (num_height_tiles * h)
                * (num_width_tiles * w + 1)
            )

            tokenized_str += tokenized_image
            images_seq_mask += [True] * len(tokenized_image)
            # print(width_crop_num, height_crop_num, len(tokenized_image)) # test the correctness of the number of image-related tokens
```
**EN:** This block defines function `DeepseekVLV2Processor.tokenize_with_images`. Parameters: self, conversation, images, bos, eos, cropping, max_req_input_len. Tokenize text with <image> tags. This subsection covers lines 364-440 of the same logical block.
**CN:** 该代码块定义函数 `DeepseekVLV2Processor.tokenize_with_images`。 参数包括 self、conversation、images、bos、eos、cropping、max_req_input_len。 文档字符串摘要：Tokenize text with <image> tags. 本小节覆盖同一逻辑块中的第 364-440 行。

### Lines 441-441: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 442-463: Defines function DeepseekVLV2Processor.tokenize_with_images (part 2)
```python
        """process the last text split"""
        tokenized_sep = self.encode(text_splits[-1], bos=False, eos=False)
        # deal with video, limit with request len
        if max_req_input_len > -1:
            if max_req_input_len < len(tokenized_sep) + len(tokenized_str) - 1:
                rest = max_req_input_len - len(tokenized_sep) - 1 - 1024
                tokenized_str = tokenized_str[:rest]
                images_seq_mask = images_seq_mask[:rest]
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
```
**EN:** This block defines function `DeepseekVLV2Processor.tokenize_with_images`. Parameters: self, conversation, images, bos, eos, cropping, max_req_input_len. Tokenize text with <image> tags. This subsection covers lines 442-463 of the same logical block.
**CN:** 该代码块定义函数 `DeepseekVLV2Processor.tokenize_with_images`。 参数包括 self、conversation、images、bos、eos、cropping、max_req_input_len。 文档字符串摘要：Tokenize text with <image> tags. 本小节覆盖同一逻辑块中的第 442-463 行。

### Lines 464-464: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 465-465: Defines function DeepseekVLV2Processor.tokenize_with_images (part 3)
```python
        return tokenized_str, images_list, images_seq_mask, images_spatial_crop
```
**EN:** This block defines function `DeepseekVLV2Processor.tokenize_with_images`. Parameters: self, conversation, images, bos, eos, cropping, max_req_input_len. Tokenize text with <image> tags. This subsection covers lines 465-465 of the same logical block.
**CN:** 该代码块定义函数 `DeepseekVLV2Processor.tokenize_with_images`。 参数包括 self、conversation、images、bos、eos、cropping、max_req_input_len。 文档字符串摘要：Tokenize text with <image> tags. 本小节覆盖同一逻辑块中的第 465-465 行。

### Lines 466-467: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 468-468: Declares class DeepseekVL2VisionEncoderConfig
```python
class DeepseekVL2VisionEncoderConfig(PretrainedConfig):
```
**EN:** This block introduces class `DeepseekVL2VisionEncoderConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `DeepseekVL2VisionEncoderConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 469-469: Declares model_type
```python
    model_type: str = "vision"
```
**EN:** This statement initializes model_type in the DeepseekVL2VisionEncoderConfig. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 DeepseekVL2VisionEncoderConfig 中初始化 model_type。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 470-470: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekVL2VisionEncoderConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekVL2VisionEncoderConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 471-485: Declares model_name, image_size, patch_size, width, layers, and 10 more
```python
    model_name: str = "siglip_large_patch16_384"
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
**EN:** This block initializes a related set of values in the DeepseekVL2VisionEncoderConfig, including model_name, image_size, patch_size, width, layers, and 10 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepseekVL2VisionEncoderConfig 中初始化一组相关值，包括 model_name, image_size, patch_size, width, layers 等 10 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 486-486: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekVL2VisionEncoderConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekVL2VisionEncoderConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 487-516: Defines function DeepseekVL2VisionEncoderConfig.__init__
```python
    def __init__(
        self,
        model_name: str = "siglip_large_patch16_384",
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
**EN:** This block defines function `DeepseekVL2VisionEncoderConfig.__init__`. Parameters: self, model_name, image_size, patch_size, width, layers, heads, mlp_ratio, global_pool, ignore_head, class_token, num_classes, use_checkpoint.
**CN:** 该代码块定义函数 `DeepseekVL2VisionEncoderConfig.__init__`。 参数包括 self、model_name、image_size、patch_size、width、layers、heads、mlp_ratio、global_pool、ignore_head、class_token、num_classes、use_checkpoint。

### Lines 517-518: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 519-519: Declares class DeepseekVL2MlpProjectorConfig
```python
class DeepseekVL2MlpProjectorConfig(PretrainedConfig):
```
**EN:** This block introduces class `DeepseekVL2MlpProjectorConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `DeepseekVL2MlpProjectorConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 520-527: Declares model_type, projector_type, input_dim, n_embed, depth, and 3 more
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
**EN:** This block initializes a related set of values in the DeepseekVL2MlpProjectorConfig, including model_type, projector_type, input_dim, n_embed, depth, and 3 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepseekVL2MlpProjectorConfig 中初始化一组相关值，包括 model_type, projector_type, input_dim, n_embed, depth 等 3 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 528-528: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekVL2MlpProjectorConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekVL2MlpProjectorConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 529-546: Defines function DeepseekVL2MlpProjectorConfig.__init__
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
**EN:** This block defines function `DeepseekVL2MlpProjectorConfig.__init__`. Parameters: self, projector_type, input_dim, n_embed, depth, mlp_ratio, downsample_ratio.
**CN:** 该代码块定义函数 `DeepseekVL2MlpProjectorConfig.__init__`。 参数包括 self、projector_type、input_dim、n_embed、depth、mlp_ratio、downsample_ratio。

### Lines 547-548: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 549-550: Declares class DeepseekV2Config
```python
class DeepseekV2Config(PretrainedConfig):

```
**EN:** This block introduces class `DeepseekV2Config` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `DeepseekV2Config`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 551-552: Declares model_type, keys_to_ignore_at_inference
```python
    model_type = "deepseek_v2"
    keys_to_ignore_at_inference = ["past_key_values"]
```
**EN:** This block initializes a related set of values in the DeepseekV2Config, including model_type, keys_to_ignore_at_inference. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepseekV2Config 中初始化一组相关值，包括 model_type, keys_to_ignore_at_inference。将这些赋值集中在一起有助于理解周边配置。

### Lines 553-553: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekV2Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekV2Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 554-627: Defines function DeepseekV2Config.__init__ (part 1)
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
**EN:** This block defines function `DeepseekV2Config.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, moe_intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, n_shared_experts, n_routed_experts, ep_size, routed_scaling_factor, kv_lora_rank, q_lora_rank, qk_rope_head_dim, v_head_dim, qk_nope_head_dim, topk_method, n_group, topk_group, num_experts_per_tok, moe_layer_freq, first_k_dense_replace, norm_topk_prob, scoring_func, aux_loss_alpha, seq_aux, hidden_act, max_position_embeddings, initializer_range, rms_norm_eps, use_cache, pad_token_id, bos_token_id, eos_token_id, pretraining_tp, tie_word_embeddings, rope_theta, rope_scaling, attention_bias, attention_dropout, use_mla. This subsection covers lines 554-627 of the same logical block.
**CN:** 该代码块定义函数 `DeepseekV2Config.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、moe_intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、n_shared_experts、n_routed_experts、ep_size、routed_scaling_factor、kv_lora_rank、q_lora_rank、qk_rope_head_dim、v_head_dim、qk_nope_head_dim、topk_method、n_group、topk_group、num_experts_per_tok、moe_layer_freq、first_k_dense_replace、norm_topk_prob、scoring_func、aux_loss_alpha、seq_aux、hidden_act、max_position_embeddings、initializer_range、rms_norm_eps、use_cache、pad_token_id、bos_token_id、eos_token_id、pretraining_tp、tie_word_embeddings、rope_theta、rope_scaling、attention_bias、attention_dropout、use_mla。 本小节覆盖同一逻辑块中的第 554-627 行。

### Lines 628-628: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 629-639: Defines function DeepseekV2Config.__init__ (part 2)
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
**EN:** This block defines function `DeepseekV2Config.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, moe_intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, n_shared_experts, n_routed_experts, ep_size, routed_scaling_factor, kv_lora_rank, q_lora_rank, qk_rope_head_dim, v_head_dim, qk_nope_head_dim, topk_method, n_group, topk_group, num_experts_per_tok, moe_layer_freq, first_k_dense_replace, norm_topk_prob, scoring_func, aux_loss_alpha, seq_aux, hidden_act, max_position_embeddings, initializer_range, rms_norm_eps, use_cache, pad_token_id, bos_token_id, eos_token_id, pretraining_tp, tie_word_embeddings, rope_theta, rope_scaling, attention_bias, attention_dropout, use_mla. This subsection covers lines 629-639 of the same logical block.
**CN:** 该代码块定义函数 `DeepseekV2Config.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、moe_intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、n_shared_experts、n_routed_experts、ep_size、routed_scaling_factor、kv_lora_rank、q_lora_rank、qk_rope_head_dim、v_head_dim、qk_nope_head_dim、topk_method、n_group、topk_group、num_experts_per_tok、moe_layer_freq、first_k_dense_replace、norm_topk_prob、scoring_func、aux_loss_alpha、seq_aux、hidden_act、max_position_embeddings、initializer_range、rms_norm_eps、use_cache、pad_token_id、bos_token_id、eos_token_id、pretraining_tp、tie_word_embeddings、rope_theta、rope_scaling、attention_bias、attention_dropout、use_mla。 本小节覆盖同一逻辑块中的第 629-639 行。

### Lines 640-640: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 641-647: Defines function DeepseekV2Config.__init__ (part 3)
```python
        super().__init__(
            pad_token_id=pad_token_id,
            bos_token_id=bos_token_id,
            eos_token_id=eos_token_id,
            tie_word_embeddings=tie_word_embeddings,
            **kwargs,
        )
```
**EN:** This block defines function `DeepseekV2Config.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, moe_intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, n_shared_experts, n_routed_experts, ep_size, routed_scaling_factor, kv_lora_rank, q_lora_rank, qk_rope_head_dim, v_head_dim, qk_nope_head_dim, topk_method, n_group, topk_group, num_experts_per_tok, moe_layer_freq, first_k_dense_replace, norm_topk_prob, scoring_func, aux_loss_alpha, seq_aux, hidden_act, max_position_embeddings, initializer_range, rms_norm_eps, use_cache, pad_token_id, bos_token_id, eos_token_id, pretraining_tp, tie_word_embeddings, rope_theta, rope_scaling, attention_bias, attention_dropout, use_mla. This subsection covers lines 641-647 of the same logical block.
**CN:** 该代码块定义函数 `DeepseekV2Config.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、moe_intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、n_shared_experts、n_routed_experts、ep_size、routed_scaling_factor、kv_lora_rank、q_lora_rank、qk_rope_head_dim、v_head_dim、qk_nope_head_dim、topk_method、n_group、topk_group、num_experts_per_tok、moe_layer_freq、first_k_dense_replace、norm_topk_prob、scoring_func、aux_loss_alpha、seq_aux、hidden_act、max_position_embeddings、initializer_range、rms_norm_eps、use_cache、pad_token_id、bos_token_id、eos_token_id、pretraining_tp、tie_word_embeddings、rope_theta、rope_scaling、attention_bias、attention_dropout、use_mla。 本小节覆盖同一逻辑块中的第 641-647 行。

### Lines 648-649: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 650-650: Declares class DeepseekVL2Config
```python
class DeepseekVL2Config(PretrainedConfig):
```
**EN:** This block introduces class `DeepseekVL2Config` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `DeepseekVL2Config`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 651-654: Declares model_type, vision_config, projector_config, language_config
```python
    model_type = "deepseek_vl_v2"
    vision_config: DeepseekVL2VisionEncoderConfig = None
    projector_config: DeepseekVL2MlpProjectorConfig = None
    language_config: DeepseekV2Config = None
```
**EN:** This block initializes a related set of values in the DeepseekVL2Config, including model_type, vision_config, projector_config, language_config. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepseekVL2Config 中初始化一组相关值，包括 model_type, vision_config, projector_config, language_config。将这些赋值集中在一起有助于理解周边配置。

### Lines 655-655: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekVL2Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekVL2Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 656-658: Declares tile_tag, global_view_pos, candidate_resolutions
```python
    tile_tag: str = "2D"
    global_view_pos: str = "head"
    candidate_resolutions: Tuple[Tuple[int, int]] = ((384, 384),)
```
**EN:** This block initializes a related set of values in the DeepseekVL2Config, including tile_tag, global_view_pos, candidate_resolutions. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 DeepseekVL2Config 中初始化一组相关值，包括 tile_tag, global_view_pos, candidate_resolutions。将这些赋值集中在一起有助于理解周边配置。

### Lines 659-659: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DeepseekVL2Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DeepseekVL2Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 660-684: Defines function DeepseekVL2Config.__init__
```python
    def __init__(
        self,
        tile_tag: str = "tile_tag",
        global_view_pos: str = "head",
        candidate_resolutions: Tuple[Tuple[int, int]] = ((384, 384),),
        **kwargs,
    ):
        super().__init__(**kwargs)

        vision_config = kwargs.get("vision_config", {})
        self.vision_config = DeepseekVL2VisionEncoderConfig(**vision_config)

        projector_config = kwargs.get("projector_config", {})
        self.projector_config = DeepseekVL2MlpProjectorConfig(**projector_config)

        language_config = kwargs.get("language_config", {})
        if isinstance(language_config, DeepseekV2Config):
            self.language_config = language_config
        else:
            self.language_config = DeepseekV2Config(**language_config)

        self.tile_tag = tile_tag
        self.global_view_pos = global_view_pos
        self.candidate_resolutions = candidate_resolutions
        self.architectures = ["DeepseekVL2ForCausalLM"]
```
**EN:** This block defines function `DeepseekVL2Config.__init__`. Parameters: self, tile_tag, global_view_pos, candidate_resolutions.
**CN:** 该代码块定义函数 `DeepseekVL2Config.__init__`。 参数包括 self、tile_tag、global_view_pos、candidate_resolutions。

### Lines 685-686: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 687-687: Implements Expr
```python
AutoProcessor.register(DeepseekVL2Config, DeepseekVLV2Processor)
```
**EN:** This block contributes general-purpose logic to the module using a `Expr` statement.
**CN:** 该代码块通过 `Expr` 语句为 模块 补充通用逻辑。

## Key Concepts / 关键概念
- **Classes / 类**: `DictOutput`, `VLChatProcessorOutput`, `ImageTransform`, `DeepseekVLV2Processor`, `DeepseekVL2VisionEncoderConfig`, `DeepseekVL2MlpProjectorConfig`, `DeepseekV2Config`, `DeepseekVL2Config`
- **Functions / 函数**: `select_best_resolution`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `dataclasses`, `math`, `typing`
- **Third-Party / 第三方**: `PIL`, `torch`, `torchvision.transforms`, `transformers`
