# deepseek_ocr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/processors/deepseek_ocr.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements model-specific processor integration for Deepseek Ocr inputs. / [CN] 为 Deepseek Ocr 输入实现模型专用的处理器集成。

## Line-by-Line Analysis / 逐行分析

### Lines 5-12: Imports
```python
import math
from typing import Literal

import torch
import torchvision.transforms as T
from PIL import Image, ImageOps
from transformers import BatchFeature, LlamaTokenizerFast
from transformers.processing_utils import ProcessorMixin
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as `math`, `typing`, external APIs such as `torch`, `torchvision.transforms`, `PIL`, `transformers`, `transformers.processing_utils`, and internal vLLM modules such as none. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 `math`, `typing`，外部 API 如 `torch`, `torchvision.transforms`, `PIL`, `transformers`, `transformers.processing_utils`，以及 vLLM 内部模块如 无。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 21-27: Module state and constants
```python
BASE_SIZE = 1024
IMAGE_SIZE = 640
CROP_MODE = True

# TODO(Isotr0py): Expose as mm_kwargs
MIN_CROPS = 2
MAX_CROPS = 6  # max:9; If your GPU memory is small, it is recommended to set it to 6.
```
**EN:** This block defines module-level constants/defaults such as `BASE_SIZE`, `IMAGE_SIZE`, `CROP_MODE`, `MIN_CROPS`, `MAX_CROPS`. They centralize shared parameters for later helpers and classes.
**CN:** 该代码块定义了模块级常量或默认值，例如 `BASE_SIZE`, `IMAGE_SIZE`, `CROP_MODE`, `MIN_CROPS`, `MAX_CROPS`。它们把后续辅助函数和类会复用的参数集中到一起。

### Lines 30-43: Function `find_closest_aspect_ratio`
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
**EN:** This function finds closest aspect ratio. Main inputs include `aspect_ratio`, `target_ratios`, `width`, `height`, `image_size`.
**CN:** 该函数负责完成 `find_closest_aspect_ratio` 对应的核心步骤。 主要输入参数包括 `aspect_ratio`, `target_ratios`, `width`, `height`, `image_size`。

### Lines 46-57: Function `calculate_aspect_ratios`
```python
def calculate_aspect_ratios(
    min_num: int = MIN_CROPS, max_num: int = MAX_CROPS
) -> list[tuple[int, int]]:
    target_ratios: set[tuple[int, int]] = set(
        (i, j)
        for n in range(min_num, max_num + 1)
        for i in range(1, n + 1)
        for j in range(1, n + 1)
        if i * j <= max_num and i * j >= min_num
    )
    sorted_target_ratios = sorted(target_ratios, key=lambda x: x[0] * x[1])
    return sorted_target_ratios
```
**EN:** This function implements `calculate_aspect_ratios`. Main inputs include `min_num`, `max_num`.
**CN:** 该函数实现 `calculate_aspect_ratios` 相关逻辑。 主要输入参数包括 `min_num`, `max_num`。

### Lines 60-78: Function `count_tiles`
```python
def count_tiles(
    orig_width,
    orig_height,
    min_num=MIN_CROPS,
    max_num=MAX_CROPS,
    image_size=640,
    use_thumbnail=False,
):
    aspect_ratio = orig_width / orig_height

    # calculate the existing image aspect ratio
    target_ratios = calculate_aspect_ratios(min_num, max_num)

    # find the closest aspect ratio to the target
    target_aspect_ratio = find_closest_aspect_ratio(
        aspect_ratio, target_ratios, orig_width, orig_height, image_size
    )

    return target_aspect_ratio
```
**EN:** This function implements `count_tiles`. Main inputs include `orig_width`, `orig_height`, `min_num`, `max_num`, `image_size`, ... (+1 more).
**CN:** 该函数实现 `count_tiles` 相关逻辑。 主要输入参数包括 `orig_width`, `orig_height`, `min_num`, `max_num`, `image_size`, ... (+1 more)。

### Lines 81-117: Function `dynamic_preprocess`
```python
def dynamic_preprocess(
    image, min_num=MIN_CROPS, max_num=MAX_CROPS, image_size=640, use_thumbnail=False
):
    orig_width, orig_height = image.size
    aspect_ratio = orig_width / orig_height

    # calculate the existing image aspect ratio
    target_ratios = calculate_aspect_ratios(min_num, max_num)

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
**EN:** This function implements `dynamic_preprocess`. Main inputs include `image`, `min_num`, `max_num`, `image_size`, `use_thumbnail`.
**CN:** 该函数实现 `dynamic_preprocess` 相关逻辑。 主要输入参数包括 `image`, `min_num`, `max_num`, `image_size`, `use_thumbnail`。

### Lines 120-140: Class `ImageTransform`
```python
class ImageTransform:
    def __init__(
        self,
        mean: tuple[float, float, float] = (0.5, 0.5, 0.5),
        std: tuple[float, float, float] = (0.5, 0.5, 0.5),
        normalize: bool = True,
    ):
        self.mean = mean
        self.std = std
        self.normalize = normalize

        transform_pipelines = [T.ToTensor()]

        if normalize:
            transform_pipelines.append(T.Normalize(mean, std))

        self.transform = T.Compose(transform_pipelines)

    def __call__(self, pil_img: Image.Image):
        x = self.transform(pil_img)
        return x
```
**EN:** Defines `ImageTransform`, a processor-related class. Key methods include `__init__`, `__call__`.
**CN:** 定义 `ImageTransform`，这是一个处理器相关类。 关键方法包括 `__init__`, `__call__`。

### Lines 143-455: Class `DeepseekOCRProcessor`
```python
class DeepseekOCRProcessor(ProcessorMixin):
    tokenizer_class = ("LlamaTokenizer", "LlamaTokenizerFast")
    attributes = ["tokenizer"]

    def __init__(
        self,
        tokenizer: LlamaTokenizerFast,
        patch_size: int = 16,
        downsample_ratio: int = 4,
        image_mean: tuple[float, float, float] = (0.5, 0.5, 0.5),
        image_std: tuple[float, float, float] = (0.5, 0.5, 0.5),
        normalize: bool = True,
        image_token: str = "<image>",
        pad_token: str = "<｜▁pad▁｜>",
        add_special_token: bool = False,
        sft_format: str = "deepseek",
        mask_prompt: bool = True,
        ignore_id: int = -100,
        image_size: int = IMAGE_SIZE,
        base_size: int = BASE_SIZE,
        strategy: Literal["v1", "v2"] = "v1",
        **kwargs,
    ):
        self.image_size = image_size
        self.base_size = base_size

        # image token calculation strategy for
        # Deepseek-OCR and Deepseek-OCR-2
        self.strategy = strategy
        assert strategy in ["v1", "v2"], "Only 'v1' and 'v2' strategies are supported."

        self.patch_size = 16
        self.image_mean = image_mean
        self.image_std = image_std
        self.normalize = normalize
        self.downsample_ratio = 4

        self.image_transform = ImageTransform(
            mean=image_mean, std=image_std, normalize=normalize
        )
# ... omitted for brevity ...

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
**EN:** Defines `DeepseekOCRProcessor`, a processor-related class derived from `ProcessorMixin`. Key methods include `__init__`, `bos_id`, `eos_id`, `pad_id`, `encode`, ... (+4 more).
**CN:** 定义 `DeepseekOCRProcessor`，这是一个处理器相关类，继承自 `ProcessorMixin`。 关键方法包括 `__init__`, `bos_id`, `eos_id`, `pad_id`, `encode`, ... (+4 more)。

## Key Concepts / 关键概念
- **EN:** Processor wrappers unify tokenizers and modality-specific preprocessors behind a single serving-oriented interface.
- **CN:** 处理器封装把 tokenizer 与模态专用预处理器统一到面向服务的单一接口之下。
- **EN:** The module is tightly coupled to Hugging Face Transformers APIs, so version compatibility is often an explicit concern.
- **CN:** 该模块与 Hugging Face Transformers API 紧密耦合，因此版本兼容性通常是显式关注点。
- **EN:** Torch types and tensor conventions influence preprocessing, dtype selection, or runtime metadata in this module.
- **CN:** Torch 的类型与张量约定会影响本模块中的预处理、数据类型选择或运行时元数据。

## Dependencies / 依赖关系
- **EN:** Standard library modules: `math`, `typing`.
- **CN:** 标准库模块：`math`, `typing`。
- **EN:** External packages: `torch`, `torchvision.transforms`, `PIL`, `transformers`, `transformers.processing_utils`.
- **CN:** 外部依赖包：`torch`, `torchvision.transforms`, `PIL`, `transformers`, `transformers.processing_utils`。
- **EN:** Internal modules: none.
- **CN:** 内部模块：无。
