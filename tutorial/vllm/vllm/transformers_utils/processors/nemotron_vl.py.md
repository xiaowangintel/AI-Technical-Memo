# nemotron_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/processors/nemotron_vl.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements model-specific processor integration for Nemotron VL inputs. / [CN] 为 Nemotron VL 输入实现模型专用的处理器集成。

## Line-by-Line Analysis / 逐行分析

### Lines 4-11: Imports
```python
import torch
import torchvision.transforms as T
from PIL import Image

from vllm.multimodal.image import convert_image_mode
from vllm.tokenizers.hf import HfTokenizer

from .internvl import InternVLImageProcessor, InternVLProcessor
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as none, external APIs such as `torch`, `torchvision.transforms`, `PIL`, and internal vLLM modules such as `vllm.multimodal.image`, `vllm.tokenizers.hf`, `.internvl`. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 无，外部 API 如 `torch`, `torchvision.transforms`, `PIL`，以及 vLLM 内部模块如 `vllm.multimodal.image`, `vllm.tokenizers.hf`, `.internvl`。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 15-15: Module state and constants
```python
Image.MAX_IMAGE_PIXELS = None  # Disable the limit entirely
```
**EN:** This block initializes module-level state such as none. The assigned names act as shared context for the rest of the file.
**CN:** 该代码块初始化了模块级状态，例如 none。这些名称为文件其余部分提供共享上下文。

### Lines 20-29: Function `build_transform`
```python
def build_transform(input_size: int):
    return T.Compose(
        [
            T.Lambda(lambda img: convert_image_mode(img, "RGB")),
            T.Resize(
                (input_size, input_size), interpolation=T.InterpolationMode.BICUBIC
            ),
            T.ToTensor(),
        ]
    )
```
**EN:** This function builds transform. Main inputs include `input_size`.
**CN:** 该函数负责完成 `build_transform` 对应的核心步骤。 主要输入参数包括 `input_size`。

### Lines 33-57: Function `find_closest_aspect_ratio`
```python
def find_closest_aspect_ratio(
    aspect_ratio: float,
    target_ratios: list[tuple[int, int]],
    *,
    width: int,
    height: int,
    image_size: int,
) -> tuple[int, int]:
    best_factor = float("-inf")
    best_ratio = (1, 1)
    area = width * height

    for rw, rh in target_ratios:
        target_aspect_ratio = rw / rh
        size_factor = min((rw * rh * image_size * image_size) / area, 0.6)
        ratio_closeness = min(
            target_aspect_ratio / aspect_ratio, aspect_ratio / target_aspect_ratio
        )
        factor = size_factor * ratio_closeness

        if factor > best_factor:
            best_factor = factor
            best_ratio = (rw, rh)

    return best_ratio
```
**EN:** This function finds closest aspect ratio. Main inputs include `aspect_ratio`, `target_ratios`, `width`, `height`, `image_size`.
**CN:** 该函数负责完成 `find_closest_aspect_ratio` 对应的核心步骤。 主要输入参数包括 `aspect_ratio`, `target_ratios`, `width`, `height`, `image_size`。

### Lines 60-88: Function `calculate_nemotron_vl_targets`
```python
def calculate_nemotron_vl_targets(
    *,
    orig_width: int,
    orig_height: int,
    target_ratios: list[tuple[int, int]],
    image_size: int,
    use_thumbnail: bool,
) -> tuple[int, int, int]:
    aspect_ratio = orig_width / orig_height

    # find the closest aspect ratio to the target
    target_aspect_ratio = find_closest_aspect_ratio(
        aspect_ratio,
        target_ratios,
        width=orig_width,
        height=orig_height,
        image_size=image_size,
    )

    # calculate the target width and height
    target_width = image_size * target_aspect_ratio[0]
    target_height = image_size * target_aspect_ratio[1]
    blocks = target_aspect_ratio[0] * target_aspect_ratio[1]

    # add thumbnail image if num_blocks != 1
    if use_thumbnail and blocks != 1:
        blocks += 1

    return blocks, target_width, target_height
```
**EN:** This function implements `calculate_nemotron_vl_targets`. Main inputs include `orig_width`, `orig_height`, `target_ratios`, `image_size`, `use_thumbnail`.
**CN:** 该函数实现 `calculate_nemotron_vl_targets` 相关逻辑。 主要输入参数包括 `orig_width`, `orig_height`, `target_ratios`, `image_size`, `use_thumbnail`。

### Lines 91-129: Function `dynamic_preprocess_nemotron_vl`
```python
def dynamic_preprocess_nemotron_vl(
    image: Image.Image,
    *,
    target_ratios: list[tuple[int, int]],
    image_size: int,
    use_thumbnail: bool,
) -> list[Image.Image]:
    orig_width, orig_height = image.size

    # calculate the number of blocks without thumbnail
    blocks, target_width, target_height = calculate_nemotron_vl_targets(
        orig_width=orig_width,
        orig_height=orig_height,
        target_ratios=target_ratios,
        image_size=image_size,
        use_thumbnail=False,
    )

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

    return processed_images
```
**EN:** This function implements `dynamic_preprocess_nemotron_vl`. Main inputs include `image`, `target_ratios`, `image_size`, `use_thumbnail`.
**CN:** 该函数实现 `dynamic_preprocess_nemotron_vl` 相关逻辑。 主要输入参数包括 `image`, `target_ratios`, `image_size`, `use_thumbnail`。

### Lines 132-143: Function `get_nemotron_vl_target_ratios`
```python
def get_nemotron_vl_target_ratios(
    min_num: int,
    max_num: int,
) -> list[tuple[int, int]]:
    target_ratios = {
        (i, j)
        for n in range(min_num, max_num + 1)
        for i in range(1, n + 1)
        for j in range(1, n + 1)
        if min_num <= i * j <= max_num
    }
    return sorted(target_ratios, key=lambda x: x[0] * x[1])
```
**EN:** This function retrieves nemotron vl target ratios. Main inputs include `min_num`, `max_num`.
**CN:** 该函数负责完成 `get_nemotron_vl_target_ratios` 对应的核心步骤。 主要输入参数包括 `min_num`, `max_num`。

### Lines 146-168: Function `image_to_pixel_values_nemotron_vl`
```python
def image_to_pixel_values_nemotron_vl(
    image: Image.Image,
    *,
    input_size: int,
    min_num: int,
    max_num: int,
    use_thumbnail: bool,
    transform: T.Compose | None = None,
) -> torch.Tensor:
    target_ratios = get_nemotron_vl_target_ratios(min_num, max_num)

    if transform is None:
        transform = build_transform(input_size=input_size)

    images = dynamic_preprocess_nemotron_vl(
        image,
        target_ratios=target_ratios,
        image_size=input_size,
        use_thumbnail=use_thumbnail,
    )

    pixel_values = torch.stack([transform(image) for image in images])
    return pixel_values
```
**EN:** This function implements `image_to_pixel_values_nemotron_vl`. Main inputs include `image`, `input_size`, `min_num`, `max_num`, `use_thumbnail`, ... (+1 more).
**CN:** 该函数实现 `image_to_pixel_values_nemotron_vl` 相关逻辑。 主要输入参数包括 `image`, `input_size`, `min_num`, `max_num`, `use_thumbnail`, ... (+1 more)。

### Lines 171-196: Class `LlamaNemotronNanoVLImageProcessor`
```python
class LlamaNemotronNanoVLImageProcessor(InternVLImageProcessor):
    def _images_to_pixel_values_lst(
        self,
        images: list[Image.Image],
        min_dynamic_patch: int | None = None,
        max_dynamic_patch: int | None = None,
        dynamic_image_size: bool | None = None,
    ) -> list[torch.Tensor]:
        min_num, max_num = self.resolve_min_max_num(
            min_dynamic_patch=min_dynamic_patch,
            max_dynamic_patch=max_dynamic_patch,
            dynamic_image_size=dynamic_image_size,
            use_thumbnail=False,  # Applied in image_to_pixel_values
        )

        return [
            image_to_pixel_values_nemotron_vl(
                image,
                input_size=self.image_size,
                min_num=min_num,
                max_num=max_num,
                use_thumbnail=self.use_thumbnail,
                transform=build_transform(self.image_size),
            )
            for image in images
        ]
```
**EN:** Defines `LlamaNemotronNanoVLImageProcessor`, a processor-related class derived from `InternVLImageProcessor`. Key methods include `_images_to_pixel_values_lst`.
**CN:** 定义 `LlamaNemotronNanoVLImageProcessor`，这是一个处理器相关类，继承自 `InternVLImageProcessor`。 关键方法包括 `_images_to_pixel_values_lst`。

### Lines 199-246: Class `LlamaNemotronNanoVLProcessor`
```python
class LlamaNemotronNanoVLProcessor(InternVLProcessor):
    """
    This model doesn't define its own HF processor,
    so we implement our own one here.

    The image processor is given by:
    https://huggingface.co/nvidia/Llama-3.1-Nemotron-Nano-VL-8B-V1/blob/main/image_processing.py
    """

    def __init__(
        self,
        image_processor: LlamaNemotronNanoVLImageProcessor,
        tokenizer: HfTokenizer,
        *,
        image_seq_length: int,
        start_image_token: str = "<img>",
        end_image_token: str = "</img>",
        ctx_image_token: str = "<image>",
    ) -> None:
        super().__init__(
            image_processor=image_processor,
            tokenizer=tokenizer,
            image_seq_length=image_seq_length,
            start_image_token=start_image_token,
            end_image_token=end_image_token,
            ctx_image_token=ctx_image_token,
        )

    def get_num_image_tokens(
        self,
        *,
        image_width: int,
        image_height: int,
    ) -> int:
        image_processor = self.image_processor
        target_ratios = self.resolve_target_ratios(
            use_thumbnail=False,  # Applied in calculate_targets
        )

        num_patches, _, _ = calculate_nemotron_vl_targets(
            orig_width=image_width,
            orig_height=image_height,
            image_size=image_processor.image_size,
            target_ratios=target_ratios,
            use_thumbnail=image_processor.use_thumbnail,
        )

        return num_patches * self.image_seq_length
```
**EN:** Defines `LlamaNemotronNanoVLProcessor`, a processor-related class derived from `InternVLProcessor`. The class docstring highlights that This model doesn't define its own HF processor, Key methods include `__init__`, `get_num_image_tokens`.
**CN:** 定义 `LlamaNemotronNanoVLProcessor`，这是一个处理器相关类，继承自 `InternVLProcessor`。 类文档字符串进一步说明了它的职责。 关键方法包括 `__init__`, `get_num_image_tokens`。

### Lines 250-251: Module state and constants
```python
SIGLIP_MEAN = (0.5, 0.5, 0.5)
SIGLIP_STD = (0.5, 0.5, 0.5)
```
**EN:** This block defines module-level constants/defaults such as `SIGLIP_MEAN`, `SIGLIP_STD`. They centralize shared parameters for later helpers and classes.
**CN:** 该代码块定义了模块级常量或默认值，例如 `SIGLIP_MEAN`, `SIGLIP_STD`。它们把后续辅助函数和类会复用的参数集中到一起。

### Lines 254-264: Function `build_siglip_transform`
```python
def build_siglip_transform(input_size: int):
    """Build transform for SigLIP vision encoder with normalization.

    Extends the base transform from nemotron_vl with SigLIP-specific normalization.
    """
    return T.Compose(
        [
            build_transform(input_size=input_size),
            T.Normalize(mean=SIGLIP_MEAN, std=SIGLIP_STD),
        ]
    )
```
**EN:** This function builds siglip transform. The docstring states that Build transform for SigLIP vision encoder with normalization. Main inputs include `input_size`.
**CN:** 该函数负责完成 `build_siglip_transform` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `input_size`。

### Lines 267-292: Class `LlamaNemotronVLEmbedImageProcessor`
```python
class LlamaNemotronVLEmbedImageProcessor(InternVLImageProcessor):
    def _images_to_pixel_values_lst(
        self,
        images: list[Image.Image],
        min_dynamic_patch: int | None = None,
        max_dynamic_patch: int | None = None,
        dynamic_image_size: bool | None = None,
    ) -> list[torch.Tensor]:
        min_num, max_num = self.resolve_min_max_num(
            min_dynamic_patch=min_dynamic_patch,
            max_dynamic_patch=max_dynamic_patch,
            dynamic_image_size=dynamic_image_size,
            use_thumbnail=False,  # Applied in image_to_pixel_values
        )

        return [
            image_to_pixel_values_nemotron_vl(
                image,
                input_size=self.image_size,
                min_num=min_num,
                max_num=max_num,
                use_thumbnail=self.use_thumbnail,
                transform=build_siglip_transform(self.image_size),
            )
            for image in images
        ]
```
**EN:** Defines `LlamaNemotronVLEmbedImageProcessor`, a processor-related class derived from `InternVLImageProcessor`. Key methods include `_images_to_pixel_values_lst`.
**CN:** 定义 `LlamaNemotronVLEmbedImageProcessor`，这是一个处理器相关类，继承自 `InternVLImageProcessor`。 关键方法包括 `_images_to_pixel_values_lst`。

### Lines 295-344: Class `LlamaNemotronVLEmbedProcessor`
```python
class LlamaNemotronVLEmbedProcessor(InternVLProcessor):
    """
    Processor for LlamaNemotronVL embedding model.

    Inherits from NemotronVLProcessor and specializes it for embedding tasks:
    - Uses SigLIP transform with normalization instead of base transform
    - Uses different image context token (<IMG_CONTEXT> vs <image>)
    """

    def __init__(
        self,
        image_processor: LlamaNemotronVLEmbedImageProcessor,
        tokenizer: HfTokenizer,
        *,
        image_seq_length: int,
        start_image_token: str = "<img>",
        end_image_token: str = "</img>",
        ctx_image_token: str = "<IMG_CONTEXT>",
    ) -> None:
        super().__init__(
            image_processor=image_processor,
            tokenizer=tokenizer,
            image_seq_length=image_seq_length,
            start_image_token=start_image_token,
            end_image_token=end_image_token,
            ctx_image_token=ctx_image_token,
        )

        self.image_processor: LlamaNemotronVLEmbedImageProcessor

    def get_num_image_tokens(
        self,
        *,
        image_width: int,
        image_height: int,
    ) -> int:
        image_processor = self.image_processor
        target_ratios = self.resolve_target_ratios(
            use_thumbnail=False,  # Applied in calculate_targets
        )

        num_patches, _, _ = calculate_nemotron_vl_targets(
            orig_width=image_width,
            orig_height=image_height,
            image_size=image_processor.image_size,
            target_ratios=target_ratios,
            use_thumbnail=image_processor.use_thumbnail,
        )

        return num_patches * self.image_seq_length
```
**EN:** Defines `LlamaNemotronVLEmbedProcessor`, a processor-related class derived from `InternVLProcessor`. The class docstring highlights that Processor for LlamaNemotronVL embedding model. Key methods include `__init__`, `get_num_image_tokens`.
**CN:** 定义 `LlamaNemotronVLEmbedProcessor`，这是一个处理器相关类，继承自 `InternVLProcessor`。 类文档字符串进一步说明了它的职责。 关键方法包括 `__init__`, `get_num_image_tokens`。

## Key Concepts / 关键概念
- **EN:** Processor wrappers unify tokenizers and modality-specific preprocessors behind a single serving-oriented interface.
- **CN:** 处理器封装把 tokenizer 与模态专用预处理器统一到面向服务的单一接口之下。
- **EN:** Internal vLLM imports connect the file to runtime registries, logging, artifact lookup, and model-execution utilities.
- **CN:** vLLM 内部导入会把该文件连接到运行时注册表、日志、模型文件查找以及模型执行工具。
- **EN:** Torch types and tensor conventions influence preprocessing, dtype selection, or runtime metadata in this module.
- **CN:** Torch 的类型与张量约定会影响本模块中的预处理、数据类型选择或运行时元数据。

## Dependencies / 依赖关系
- **EN:** Standard library modules: none.
- **CN:** 标准库模块：无。
- **EN:** External packages: `torch`, `torchvision.transforms`, `PIL`.
- **CN:** 外部依赖包：`torch`, `torchvision.transforms`, `PIL`。
- **EN:** Internal modules: `vllm.multimodal.image`, `vllm.tokenizers.hf`, `.internvl`.
- **CN:** 内部模块：`vllm.multimodal.image`, `vllm.tokenizers.hf`, `.internvl`。
