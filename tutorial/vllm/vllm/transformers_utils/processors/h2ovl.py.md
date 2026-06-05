# h2ovl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/processors/h2ovl.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements model-specific processor integration for H2ovl inputs. / [CN] 为 H2ovl 输入实现模型专用的处理器集成。

## Line-by-Line Analysis / 逐行分析

### Lines 11-22: Imports
```python
import torch
from PIL import Image

from vllm.tokenizers.hf import HfTokenizer

from .internvl import (
    InternVLImageProcessor,
    InternVLProcessor,
    build_transform,
    find_closest_aspect_ratio,
    get_internvl_target_ratios,
)
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as none, external APIs such as `torch`, `PIL`, and internal vLLM modules such as `vllm.tokenizers.hf`, `.internvl`. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 无，外部 API 如 `torch`, `PIL`，以及 vLLM 内部模块如 `vllm.tokenizers.hf`, `.internvl`。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 25-38: Function `resolve_h2ovl_min_max_num`
```python
def resolve_h2ovl_min_max_num(
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
**EN:** This function resolves h2ovl min max num. Main inputs include `min_dynamic_patch`, `max_dynamic_patch`, `dynamic_image_size`, `use_thumbnail`.
**CN:** 该函数负责完成 `resolve_h2ovl_min_max_num` 对应的核心步骤。 主要输入参数包括 `min_dynamic_patch`, `max_dynamic_patch`, `dynamic_image_size`, `use_thumbnail`。

### Lines 41-58: Function `get_h2ovl_target_ratios`
```python
def get_h2ovl_target_ratios(
    min_num: int,
    max_num: int,
    *,
    prior_aspect_ratio: tuple[int, int] | None,
) -> list[tuple[int, int]]:
    target_ratios = get_internvl_target_ratios(min_num, max_num)

    # if prior_aspect_ratio is provided, filter the target ratios
    if prior_aspect_ratio is not None:
        target_ratios = [
            ratio
            for ratio in target_ratios
            if prior_aspect_ratio[0] % ratio[0] != 0
            and prior_aspect_ratio[1] % ratio[1] != 0
        ]

    return target_ratios
```
**EN:** This function retrieves h2ovl target ratios. Main inputs include `min_num`, `max_num`, `prior_aspect_ratio`.
**CN:** 该函数负责完成 `get_h2ovl_target_ratios` 对应的核心步骤。 主要输入参数包括 `min_num`, `max_num`, `prior_aspect_ratio`。

### Lines 62-90: Function `calculate_h2ovl_targets`
```python
def calculate_h2ovl_targets(
    *,
    orig_width: int,
    orig_height: int,
    target_ratios: list[tuple[int, int]],
    image_size: int,
    use_thumbnail: bool,
) -> tuple[int, int, int, tuple[int, int]]:
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

    return blocks, target_width, target_height, target_aspect_ratio
```
**EN:** This function implements `calculate_h2ovl_targets`. Main inputs include `orig_width`, `orig_height`, `target_ratios`, `image_size`, `use_thumbnail`.
**CN:** 该函数实现 `calculate_h2ovl_targets` 相关逻辑。 主要输入参数包括 `orig_width`, `orig_height`, `target_ratios`, `image_size`, `use_thumbnail`。

### Lines 95-138: Function `dynamic_preprocess_h2ovl`
```python
def dynamic_preprocess_h2ovl(
    image: Image.Image,
    *,
    target_ratios: list[tuple[int, int]],
    image_size: int,
    use_thumbnail: bool,
) -> tuple[list[Image.Image], tuple[int, int]]:
    orig_width, orig_height = image.size

    # calculate the number of blocks without thumbnail
    (
        blocks,
        target_width,
        target_height,
        target_aspect_ratio,
    ) = calculate_h2ovl_targets(
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

    return processed_images, target_aspect_ratio
```
**EN:** This function implements `dynamic_preprocess_h2ovl`. Main inputs include `image`, `target_ratios`, `image_size`, `use_thumbnail`.
**CN:** 该函数实现 `dynamic_preprocess_h2ovl` 相关逻辑。 主要输入参数包括 `image`, `target_ratios`, `image_size`, `use_thumbnail`。

### Lines 141-165: Function `_preprocess_image`
```python
def _preprocess_image(
    image: Image.Image,
    *,
    input_size: int,
    min_num: int,
    max_num: int,
    use_thumbnail: bool,
    prior_aspect_ratio: tuple[int, int] | None,
) -> tuple[torch.Tensor, tuple[int, int]]:
    target_ratios = get_h2ovl_target_ratios(
        min_num,
        max_num,
        prior_aspect_ratio=prior_aspect_ratio,
    )

    transform = build_transform(input_size=input_size)
    images, target_aspect_ratio = dynamic_preprocess_h2ovl(
        image,
        image_size=input_size,
        use_thumbnail=use_thumbnail,
        target_ratios=target_ratios,
    )

    pixel_values = torch.stack([transform(image) for image in images])
    return pixel_values, target_aspect_ratio
```
**EN:** This private function implements `_preprocess_image`. Main inputs include `image`, `input_size`, `min_num`, `max_num`, `use_thumbnail`, ... (+1 more).
**CN:** 该私有函数实现 `_preprocess_image` 相关逻辑。 主要输入参数包括 `image`, `input_size`, `min_num`, `max_num`, `use_thumbnail`, ... (+1 more)。

### Lines 169-213: Function `image_to_pixel_values_h2ovl`
```python
def image_to_pixel_values_h2ovl(
    image: Image.Image,
    *,
    input_size: int,
    min_num: int,
    max_num: int,
    use_thumbnail: bool,
    use_msac: bool,
) -> torch.Tensor:
    # when MSAC is turned on, we need to process the image twice
    if use_msac:
        # first pass
        pixel_values1, aspect_ratio1 = _preprocess_image(
            image,
            input_size=input_size,
            min_num=1,
            max_num=max_num,
            use_thumbnail=True,
            prior_aspect_ratio=None,
        )
        # second pass
        pixel_values2, _ = _preprocess_image(
            image,
            input_size=input_size,
            min_num=3,
            max_num=max_num,
            use_thumbnail=True,
            prior_aspect_ratio=aspect_ratio1,
        )
        # combine pixel values
        pixel_values = torch.cat(
            [pixel_values2[:-1], pixel_values1[:-1], pixel_values2[-1:]], 0
        )

    else:
        pixel_values, _ = _preprocess_image(
            image,
            input_size=input_size,
            min_num=min_num,
            max_num=max_num,
            use_thumbnail=use_thumbnail,
            prior_aspect_ratio=None,
        )

    return pixel_values
```
**EN:** This function implements `image_to_pixel_values_h2ovl`. Main inputs include `image`, `input_size`, `min_num`, `max_num`, `use_thumbnail`, ... (+1 more).
**CN:** 该函数实现 `image_to_pixel_values_h2ovl` 相关逻辑。 主要输入参数包括 `image`, `input_size`, `min_num`, `max_num`, `use_thumbnail`, ... (+1 more)。

### Lines 216-286: Class `H2OVLImageProcessor`
```python
class H2OVLImageProcessor(InternVLImageProcessor):
    def __init__(
        self,
        image_size: int,
        min_dynamic_patch: int,
        max_dynamic_patch: int,
        dynamic_image_size: bool,
        use_thumbnail: bool,
        use_msac: bool,
    ) -> None:
        super().__init__(
            image_size=image_size,
            min_dynamic_patch=min_dynamic_patch,
            max_dynamic_patch=max_dynamic_patch,
            dynamic_image_size=dynamic_image_size,
            use_thumbnail=use_thumbnail,
        )

        self.use_msac = use_msac

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

        return resolve_h2ovl_min_max_num(
            min_dynamic_patch=min_dynamic_patch,
            max_dynamic_patch=max_dynamic_patch,
# ... omitted for brevity ...

        return [
            image_to_pixel_values_h2ovl(
                image,
                input_size=self.image_size,
                min_num=min_num,
                max_num=max_num,
                use_thumbnail=self.use_thumbnail,
                use_msac=use_msac,
            )
            for image in images
        ]
```
**EN:** Defines `H2OVLImageProcessor`, a processor-related class derived from `InternVLImageProcessor`. Key methods include `__init__`, `resolve_min_max_num`, `_images_to_pixel_values_lst`.
**CN:** 定义 `H2OVLImageProcessor`，这是一个处理器相关类，继承自 `InternVLImageProcessor`。 关键方法包括 `__init__`, `resolve_min_max_num`, `_images_to_pixel_values_lst`。

### Lines 289-387: Class `H2OVLProcessor`
```python
class H2OVLProcessor(InternVLProcessor):
    def __init__(
        self,
        image_processor: H2OVLImageProcessor,
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

        self.image_processor: H2OVLImageProcessor

    def resolve_target_ratios(
        self,
        *,
        min_dynamic_patch: int | None = None,
        max_dynamic_patch: int | None = None,
        dynamic_image_size: bool | None = None,
        use_thumbnail: bool | None = None,
        prior_aspect_ratio: tuple[int, int] | None = None,
        override_min_num: int | None = None,
    ) -> list[tuple[int, int]]:
        min_num, max_num = self.image_processor.resolve_min_max_num(
            min_dynamic_patch=min_dynamic_patch,
            max_dynamic_patch=max_dynamic_patch,
            dynamic_image_size=dynamic_image_size,
            use_thumbnail=use_thumbnail,
        )
        if override_min_num is not None:
            min_num = override_min_num
# ... omitted for brevity ...
            target_ratios = self.resolve_target_ratios(
                use_thumbnail=False,  # Applied in calculate_targets
            )
            num_patches, _, _, _ = calculate_h2ovl_targets(
                orig_width=image_width,
                orig_height=image_height,
                image_size=image_processor.image_size,
                target_ratios=target_ratios,
                use_thumbnail=use_thumbnail,
            )

        return num_patches * self.image_seq_length
```
**EN:** Defines `H2OVLProcessor`, a processor-related class derived from `InternVLProcessor`. Key methods include `__init__`, `resolve_target_ratios`, `get_num_image_tokens`.
**CN:** 定义 `H2OVLProcessor`，这是一个处理器相关类，继承自 `InternVLProcessor`。 关键方法包括 `__init__`, `resolve_target_ratios`, `get_num_image_tokens`。

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
- **EN:** External packages: `torch`, `PIL`.
- **CN:** 外部依赖包：`torch`, `PIL`。
- **EN:** Internal modules: `vllm.tokenizers.hf`, `.internvl`.
- **CN:** 内部模块：`vllm.tokenizers.hf`, `.internvl`。
