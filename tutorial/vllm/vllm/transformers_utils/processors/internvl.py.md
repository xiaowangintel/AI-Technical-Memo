# internvl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/processors/internvl.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements model-specific processor integration for Internvl inputs. / [CN] 为 Internvl 输入实现模型专用的处理器集成。

## Line-by-Line Analysis / 逐行分析

### Lines 11-20: Imports
```python
import numpy.typing as npt
import torch
import torchvision.transforms as T
from PIL import Image
from transformers import BatchFeature, TensorType
from transformers.processing_utils import ProcessorMixin

from vllm.multimodal.image import convert_image_mode
from vllm.multimodal.processing import PromptUpdateDetails
from vllm.tokenizers.hf import HfTokenizer
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as none, external APIs such as `numpy.typing`, `torch`, `torchvision.transforms`, `PIL`, `transformers`, `transformers.processing_utils`, and internal vLLM modules such as `vllm.multimodal.image`, `vllm.multimodal.processing`, `vllm.tokenizers.hf`. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 无，外部 API 如 `numpy.typing`, `torch`, `torchvision.transforms`, `PIL`, `transformers`, `transformers.processing_utils`，以及 vLLM 内部模块如 `vllm.multimodal.image`, `vllm.multimodal.processing`, `vllm.tokenizers.hf`。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 22-23: Module state and constants
```python
IMAGENET_MEAN = (0.485, 0.456, 0.406)
IMAGENET_STD = (0.229, 0.224, 0.225)
```
**EN:** This block defines module-level constants/defaults such as `IMAGENET_MEAN`, `IMAGENET_STD`. They centralize shared parameters for later helpers and classes.
**CN:** 该代码块定义了模块级常量或默认值，例如 `IMAGENET_MEAN`, `IMAGENET_STD`。它们把后续辅助函数和类会复用的参数集中到一起。

### Lines 27-38: Function `build_transform`
```python
def build_transform(input_size: int):
    MEAN, STD = IMAGENET_MEAN, IMAGENET_STD
    return T.Compose(
        [
            T.Lambda(lambda img: convert_image_mode(img, "RGB")),
            T.Resize(
                (input_size, input_size), interpolation=T.InterpolationMode.BICUBIC
            ),
            T.ToTensor(),
            T.Normalize(mean=MEAN, std=STD),
        ]
    )
```
**EN:** This function builds transform. Main inputs include `input_size`.
**CN:** 该函数负责完成 `build_transform` 对应的核心步骤。 主要输入参数包括 `input_size`。

### Lines 42-62: Function `find_closest_aspect_ratio`
```python
def find_closest_aspect_ratio(
    aspect_ratio: float,
    target_ratios: list[tuple[int, int]],
    *,
    width: int,
    height: int,
    image_size: int,
) -> tuple[int, int]:
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

### Lines 65-78: Function `resolve_internvl_min_max_num`
```python
def resolve_internvl_min_max_num(
    *,
    min_dynamic_patch: int,
    max_dynamic_patch: int,
    dynamic_image_size: bool,
    use_thumbnail: bool,
) -> tuple[int, int]:
    min_dynamic_patch = min_dynamic_patch if dynamic_image_size else 1
    max_dynamic_patch = max_dynamic_patch if dynamic_image_size else 1

    if use_thumbnail and max_dynamic_patch != 1:
        max_dynamic_patch += 1

    return min_dynamic_patch, max_dynamic_patch
```
**EN:** This function resolves internvl min max num. Main inputs include `min_dynamic_patch`, `max_dynamic_patch`, `dynamic_image_size`, `use_thumbnail`.
**CN:** 该函数负责完成 `resolve_internvl_min_max_num` 对应的核心步骤。 主要输入参数包括 `min_dynamic_patch`, `max_dynamic_patch`, `dynamic_image_size`, `use_thumbnail`。

### Lines 81-92: Function `get_internvl_target_ratios`
```python
def get_internvl_target_ratios(
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
**EN:** This function retrieves internvl target ratios. Main inputs include `min_num`, `max_num`.
**CN:** 该函数负责完成 `get_internvl_target_ratios` 对应的核心步骤。 主要输入参数包括 `min_num`, `max_num`。

### Lines 95-123: Function `calculate_internvl_targets`
```python
def calculate_internvl_targets(
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
**EN:** This function implements `calculate_internvl_targets`. Main inputs include `orig_width`, `orig_height`, `target_ratios`, `image_size`, `use_thumbnail`.
**CN:** 该函数实现 `calculate_internvl_targets` 相关逻辑。 主要输入参数包括 `orig_width`, `orig_height`, `target_ratios`, `image_size`, `use_thumbnail`。

### Lines 127-165: Function `dynamic_preprocess_internvl`
```python
def dynamic_preprocess_internvl(
    image: Image.Image,
    *,
    target_ratios: list[tuple[int, int]],
    image_size: int,
    use_thumbnail: bool,
) -> list[Image.Image]:
    orig_width, orig_height = image.size

    # calculate the number of blocks without thumbnail
    blocks, target_width, target_height = calculate_internvl_targets(
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
**EN:** This function implements `dynamic_preprocess_internvl`. Main inputs include `image`, `target_ratios`, `image_size`, `use_thumbnail`.
**CN:** 该函数实现 `dynamic_preprocess_internvl` 相关逻辑。 主要输入参数包括 `image`, `target_ratios`, `image_size`, `use_thumbnail`。

### Lines 169-188: Function `image_to_pixel_values_internvl`
```python
def image_to_pixel_values_internvl(
    image: Image.Image,
    *,
    input_size: int,
    min_num: int,
    max_num: int,
    use_thumbnail: bool,
) -> torch.Tensor:
    target_ratios = get_internvl_target_ratios(min_num, max_num)

    transform = build_transform(input_size=input_size)
    images = dynamic_preprocess_internvl(
        image,
        target_ratios=target_ratios,
        image_size=input_size,
        use_thumbnail=use_thumbnail,
    )

    pixel_values = torch.stack([transform(image) for image in images])
    return pixel_values
```
**EN:** This function implements `image_to_pixel_values_internvl`. Main inputs include `image`, `input_size`, `min_num`, `max_num`, `use_thumbnail`.
**CN:** 该函数实现 `image_to_pixel_values_internvl` 相关逻辑。 主要输入参数包括 `image`, `input_size`, `min_num`, `max_num`, `use_thumbnail`。

### Lines 192-215: Function `video_to_pixel_values_internvl`
```python
def video_to_pixel_values_internvl(
    video: npt.NDArray,
    *,
    input_size: int,
    min_num: int,
    max_num: int,
    use_thumbnail: bool,
) -> torch.Tensor:
    target_ratios = get_internvl_target_ratios(min_num, max_num)

    transform = build_transform(input_size=input_size)
    frames_list = list[Image.Image]()
    for frame in video:
        pil_frame = dynamic_preprocess_internvl(
            Image.fromarray(frame, mode="RGB"),
            target_ratios=target_ratios,
            image_size=input_size,
            use_thumbnail=use_thumbnail,
        )
        assert len(pil_frame) == 1
        frames_list.extend(pil_frame)

    pixel_values = torch.stack([transform(image) for image in frames_list])
    return pixel_values
```
**EN:** This function implements `video_to_pixel_values_internvl`. Main inputs include `video`, `input_size`, `min_num`, `max_num`, `use_thumbnail`.
**CN:** 该函数实现 `video_to_pixel_values_internvl` 相关逻辑。 主要输入参数包括 `video`, `input_size`, `min_num`, `max_num`, `use_thumbnail`。

### Lines 218-312: Class `InternVLImageProcessor`
```python
class InternVLImageProcessor:
    def __init__(
        self,
        image_size: int,
        min_dynamic_patch: int,
        max_dynamic_patch: int,
        dynamic_image_size: bool,
        use_thumbnail: bool,
    ) -> None:
        self.image_size = image_size
        self.min_dynamic_patch = min_dynamic_patch
        self.max_dynamic_patch = max_dynamic_patch
        self.dynamic_image_size = dynamic_image_size
        self.use_thumbnail = use_thumbnail

    def resolve_min_max_num(
        self,
        *,
        min_dynamic_patch: int | None = None,
        max_dynamic_patch: int | None = None,
        dynamic_image_size: bool | None = None,
        use_thumbnail: bool | None = None,
    ) -> tuple[int, int]:
        if min_dynamic_patch is None:
            min_dynamic_patch = self.min_dynamic_patch
        if max_dynamic_patch is None:
            max_dynamic_patch = self.max_dynamic_patch
        if dynamic_image_size is None:
            dynamic_image_size = self.dynamic_image_size
        if use_thumbnail is None:
            use_thumbnail = self.use_thumbnail

        return resolve_internvl_min_max_num(
            min_dynamic_patch=min_dynamic_patch,
            max_dynamic_patch=max_dynamic_patch,
            dynamic_image_size=dynamic_image_size,
            use_thumbnail=use_thumbnail,
        )

    def _images_to_pixel_values_lst(
# ... omitted for brevity ...
        pixel_values_lst = self._images_to_pixel_values_lst(
            images_lst,
            min_dynamic_patch=min_dynamic_patch,
            max_dynamic_patch=max_dynamic_patch,
            dynamic_image_size=dynamic_image_size,
        )

        image_inputs = {
            "pixel_values_flat": torch.cat(pixel_values_lst),
            "image_num_patches": torch.tensor([len(item) for item in pixel_values_lst]),
        }
        return BatchFeature(image_inputs, tensor_type=return_tensors)
```
**EN:** Defines `InternVLImageProcessor`, a processor-related class. Key methods include `__init__`, `resolve_min_max_num`, `_images_to_pixel_values_lst`, `__call__`.
**CN:** 定义 `InternVLImageProcessor`，这是一个处理器相关类。 关键方法包括 `__init__`, `resolve_min_max_num`, `_images_to_pixel_values_lst`, `__call__`。

### Lines 315-352: Class `InternVLVideoProcessor`
```python
class InternVLVideoProcessor:
    def __init__(
        self,
        image_size: int,
    ) -> None:
        self.image_size = image_size

    def _videos_to_pixel_values_lst(
        self,
        videos: list[npt.NDArray],
    ) -> list[torch.Tensor]:
        return [
            video_to_pixel_values_internvl(
                video,
                input_size=self.image_size,
                min_num=1,
                max_num=1,
                use_thumbnail=False,
            )
            for video in videos
        ]

    def __call__(
        self,
        videos: npt.NDArray | list[npt.NDArray],
        *,
        return_tensors: str | TensorType | None = None,
        **kwargs,
    ) -> BatchFeature:
        videos_lst = [videos] if not isinstance(videos, list) else videos

        pixel_values_lst = self._videos_to_pixel_values_lst(videos_lst)

        image_inputs = {
            "pixel_values_flat_video": torch.cat(pixel_values_lst),
            "video_num_patches": torch.tensor([len(item) for item in pixel_values_lst]),
        }
        return BatchFeature(image_inputs, tensor_type=return_tensors)
```
**EN:** Defines `InternVLVideoProcessor`, a processor-related class. Key methods include `__init__`, `_videos_to_pixel_values_lst`, `__call__`.
**CN:** 定义 `InternVLVideoProcessor`，这是一个处理器相关类。 关键方法包括 `__init__`, `_videos_to_pixel_values_lst`, `__call__`。

### Lines 355-564: Class `InternVLProcessor`
```python
class InternVLProcessor(ProcessorMixin):
    """
    This model doesn't define its own HF processor,
    so we implement our own one here.

    The code to insert image tokens is based on:
    https://huggingface.co/OpenGVLab/InternVL2-1B/blob/main/modeling_internvl_chat.py#L252

    Code for video processing is adapted from video example:
    https://huggingface.co/OpenGVLab/InternVL3-1B#inference-with-transformers
    """

    attributes = ["image_processor", "tokenizer", "video_processor"]

    def __init__(
        self,
        image_processor: InternVLImageProcessor,
        tokenizer: HfTokenizer,
        video_processor: InternVLVideoProcessor | None = None,
        *,
        image_seq_length: int,
        start_image_token: str = "<img>",
        end_image_token: str = "</img>",
        ctx_image_token: str = "<IMG_CONTEXT>",
        ctx_video_token: str | None = None,
    ) -> None:
        self.image_processor = image_processor
        self.tokenizer = tokenizer
        self.video_processor = video_processor

        self.image_seq_length = image_seq_length
        self.start_image_token = start_image_token
        self.end_image_token = end_image_token
        self.ctx_image_token = ctx_image_token
        self.ctx_video_token = ctx_video_token

        self.start_image_token_id = tokenizer.convert_tokens_to_ids(start_image_token)
        self.end_image_token_id = tokenizer.convert_tokens_to_ids(end_image_token)
        self.ctx_image_token_id = tokenizer.convert_tokens_to_ids(ctx_image_token)
        self.ctx_video_token_id = (
# ... omitted for brevity ...
                    processed_text.append(new_prompt)

                text = processed_text

            text_inputs = self.tokenizer(text, return_tensors=return_tensors)
        else:
            text_inputs = {}

        return BatchFeature(
            data={**text_inputs, **image_inputs, **video_inputs},
            tensor_type=return_tensors,
        )
```
**EN:** Defines `InternVLProcessor`, a processor-related class derived from `ProcessorMixin`. The class docstring highlights that This model doesn't define its own HF processor, Key methods include `__init__`, `resolve_target_ratios`, `get_num_image_tokens`, `get_image_repl`, `get_video_repl`, ... (+1 more).
**CN:** 定义 `InternVLProcessor`，这是一个处理器相关类，继承自 `ProcessorMixin`。 类文档字符串进一步说明了它的职责。 关键方法包括 `__init__`, `resolve_target_ratios`, `get_num_image_tokens`, `get_image_repl`, `get_video_repl`, ... (+1 more)。

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
- **EN:** Standard library modules: none.
- **CN:** 标准库模块：无。
- **EN:** External packages: `numpy.typing`, `torch`, `torchvision.transforms`, `PIL`, `transformers`, `transformers.processing_utils`.
- **CN:** 外部依赖包：`numpy.typing`, `torch`, `torchvision.transforms`, `PIL`, `transformers`, `transformers.processing_utils`。
- **EN:** Internal modules: `vllm.multimodal.image`, `vllm.multimodal.processing`, `vllm.tokenizers.hf`.
- **CN:** 内部模块：`vllm.multimodal.image`, `vllm.multimodal.processing`, `vllm.tokenizers.hf`。
