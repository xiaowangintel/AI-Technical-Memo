# isaac.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/processors/isaac.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements model-specific processor integration for Isaac inputs. / [CN] 为 Isaac 输入实现模型专用的处理器集成。

## Line-by-Line Analysis / 逐行分析

### Lines 3-14: Imports
```python
import math
from typing import Any, TypedDict

import numpy as np
import torch
import torch.nn.functional as F
from PIL import Image
from transformers import BatchFeature, ProcessorMixin, TensorType
from transformers.processing_utils import ProcessingKwargs
from typing_extensions import Unpack

from vllm.tokenizers.hf import HfTokenizer
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as `math`, `typing`, external APIs such as `numpy`, `torch`, `torch.nn.functional`, `PIL`, `transformers`, `transformers.processing_utils`, `typing_extensions`, and internal vLLM modules such as `vllm.tokenizers.hf`. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 `math`, `typing`，外部 API 如 `numpy`, `torch`, `torch.nn.functional`, `PIL`, `transformers`, `transformers.processing_utils`, `typing_extensions`，以及 vLLM 内部模块如 `vllm.tokenizers.hf`。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 16-21: Module state and constants
```python
MAX_PIXELS = 60_000_000  # 60-megapixel ceiling ≈ 8200 × 7300 px

# Vision preprocessing constants
VISION_MEAN = (0.5, 0.5, 0.5)
VISION_STD = (0.5, 0.5, 0.5)
VISION_SCALE = 1 / 255
```
**EN:** This block defines module-level constants/defaults such as `MAX_PIXELS`, `VISION_MEAN`, `VISION_STD`, `VISION_SCALE`. They centralize shared parameters for later helpers and classes.
**CN:** 该代码块定义了模块级常量或默认值，例如 `MAX_PIXELS`, `VISION_MEAN`, `VISION_STD`, `VISION_SCALE`。它们把后续辅助函数和类会复用的参数集中到一起。

### Lines 43-51: Function `extract_image_pil`
```python
def extract_image_pil(image: Image.Image) -> torch.Tensor:
    if image.width * image.height > MAX_PIXELS:
        raise ValueError(
            f"Image (w={image.width}, h={image.height}) > MAX=`{MAX_PIXELS}`"
        )
    img = image if image.mode == "RGB" else image.convert("RGB")
    arr = np.asarray(img)
    arr = _make_writeable(arr)
    return torch.from_numpy(arr)
```
**EN:** This function implements `extract_image_pil`. Main inputs include `image`.
**CN:** 该函数实现 `extract_image_pil` 相关逻辑。 主要输入参数包括 `image`。

### Lines 54-156: Function `get_image_size_for_max_num_patches`
```python
def get_image_size_for_max_num_patches(
    image_height: int,
    image_width: int,
    patch_size: int,
    max_num_patches: int,
    min_num_patches: int | None = None,
    eps: float = 1e-5,
    pixel_shuffle_scale: int = 1,
) -> tuple[int, int]:
    r"""Compute a target resolution whose patch grid satisfies patching parametrization.

    Args:
        image_height (`int`):
            Height in pixels of the source image prior to any resizing.
        image_width (`int`):
            Width in pixels of the source image prior to any resizing.
        patch_size (`int`):
            Size of the square patch used by the vision encoder.
        max_num_patches (`int`):
            Upper bound on `(height / patch_size) * (width / patch_size)` after
            resizing.
        min_num_patches (`int`, *optional*):
            Lower bound on the number of patches. When provided the image will
            be scaled up if necessary.
        eps (`float`, *optional*, defaults to 1e-5):
            Convergence tolerance for the internal binary search to determine
            the target dimensions.
        pixel_shuffle_scale (`int`, *optional*, defaults to 1):
            Additional stride multiplier applied when pixel shuffle later
            reduces spatial resolution.

    Returns:
        `tuple[int, int]`: Height and width (in pixels) that are multiples of
        `patch_size * pixel_shuffle_scale` and respect both the maximum and
        optional minimum patch-count constraints.
    """

    def get_scaled_image_size(scale, original_size, patch_size, pixel_shuffle_scale):
        scaled_size = scale * original_size
        divisor = patch_size * pixel_shuffle_scale
# ... omitted for brevity ...
            if num_patches <= max_num_patches:
                scale_min = scale
            else:
                scale_max = scale
        scale = scale_min
        target_height = get_scaled_image_size(
            scale, image_height, patch_size, pixel_shuffle_scale
        )
        target_width = get_scaled_image_size(
            scale, image_width, patch_size, pixel_shuffle_scale
        )
        return target_height, target_width
```
**EN:** This function retrieves image size for max num patches. The docstring states that Compute a target resolution whose patch grid satisfies patching parametrization. Main inputs include `image_height`, `image_width`, `patch_size`, `max_num_patches`, `min_num_patches`, ... (+2 more).
**CN:** 该函数负责完成 `get_image_size_for_max_num_patches` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `image_height`, `image_width`, `patch_size`, `max_num_patches`, `min_num_patches`, ... (+2 more)。

### Lines 159-160: Module state and constants
```python
_MEAN_TENSOR = torch.tensor(VISION_MEAN, dtype=torch.float32).view(1, 1, 1, -1)
_STD_TENSOR = torch.tensor(VISION_STD, dtype=torch.float32).view(1, 1, 1, -1)
```
**EN:** This block defines module-level constants/defaults such as `_MEAN_TENSOR`, `_STD_TENSOR`. They centralize shared parameters for later helpers and classes.
**CN:** 该代码块定义了模块级常量或默认值，例如 `_MEAN_TENSOR`, `_STD_TENSOR`。它们把后续辅助函数和类会复用的参数集中到一起。

### Lines 163-188: Function `prepare_image_tensor`
```python
def prepare_image_tensor(
    image: torch.Tensor,
    scale: float = VISION_SCALE,
) -> torch.Tensor:
    r"""Standardize RGB images prior to patch extraction via rescaling and whitening.

    Args:
        image (`torch.Tensor`):
            Tensor with shape `(..., height, width, 3)` containing RGB values.
            The tensor is converted to floating point if needed.
        scale (`float`, *optional*, defaults to `VISION_SCALE`):
            Scalar multiplier applied before normalization.
    Returns:
        `torch.Tensor`: Normalized tensor with the same shape as the input and
        dtype `torch.float32`.
    """
    if not torch.is_floating_point(image):
        image = image.float()
    rescaled = image * scale

    # Use precomputed tensors and move to the correct device if needed
    mean_tensor = _MEAN_TENSOR.to(image.device)
    std_tensor = _STD_TENSOR.to(image.device)

    normalized = (rescaled - mean_tensor) / std_tensor
    return normalized
```
**EN:** This function implements `prepare_image_tensor`. The docstring states that Standardize RGB images prior to patch extraction via rescaling and whitening. Main inputs include `image`, `scale`.
**CN:** 该函数实现 `prepare_image_tensor` 相关逻辑。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `image`, `scale`。

### Lines 191-229: Function `patchify_vision`
```python
def patchify_vision(image: torch.Tensor, patch_size: int) -> torch.Tensor:
    r"""Convert normalized images into flattened ViT-style patches.

    Args:
        image (`torch.Tensor`):
            Tensor of shape `(num_images, height, width, channels)`.
        patch_size (`int`):
            Edge length of the square patches

    Returns:
        `torch.Tensor`:
            Patch tensor where each position stores the flattened pixels
            belonging to that patch.

    Raises:
        ValueError: If `height` or `width` is not divisible by `patch_size`.
    """
    num_images, height, width, channels = image.shape
    if height % patch_size or width % patch_size:
        raise ValueError(
            "Dimensions of images "
            f"{image.shape} are not divisible by patch_size={patch_size}."
        )
    patches = image.reshape(
        num_images,
        height // patch_size,
        patch_size,
        width // patch_size,
        patch_size,
        channels,
    )
    patches = patches.permute(0, 1, 3, 2, 4, 5)
    patches = patches.reshape(
        num_images,
        height // patch_size,
        width // patch_size,
        channels * patch_size * patch_size,
    )
    return patches
```
**EN:** This function implements `patchify_vision`. The docstring states that Convert normalized images into flattened ViT-style patches. Main inputs include `image`, `patch_size`.
**CN:** 该函数实现 `patchify_vision` 相关逻辑。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `image`, `patch_size`。

### Lines 232-307: Function `process_vision_for_patches`
```python
def process_vision_for_patches(
    images: torch.Tensor,
    patch_size: int,
    max_num_patches: int,
    min_num_patches: int | None = None,
    pixel_shuffle_scale: int = 1,
) -> tuple[torch.Tensor, list[int]]:
    r"""Resize, normalize, and patchify RGB images for the vision encoder.

    Args:
        images (`torch.Tensor`):
            Either `(height, width, channels)` for a single image or
            `(num_images, height, width, channels)` for a batch. Channels are
            expected to be RGB.
        patch_size (`int`):
            Edge length of square patches; implicitly controls resize grid granularity.
        max_num_patches (`int`):
            Maximum number of patches allowed after resizing.
        min_num_patches (`int`, *optional*):
            Minimum number of patches. If provided, the routine upsamples images
            as needed to satisfy the lower bound.
        pixel_shuffle_scale (`int`, *optional*, defaults to 1):
            Pixel shuffle scale factor; influences the target grid that the
            function produces.

    Returns:
        `tuple[torch.Tensor, list[int]]`: A pair `(patches, dims_virtual)`
        where `patches` has shape `(num_images, target_h / patch_size, target_w
        / patch_size, channels * patch_size**2)` and `dims_virtual` encodes
        effective `(images, height, width)` dimensions after optional pixel
        shuffling.
    """
    # Add batch dim if single image
    if images.dim() == 3:
        images = images.unsqueeze(0)

    # Permute to channel first for resize
    images = images.permute(0, 3, 1, 2)

    # Get target dimensions
# ... omitted for brevity ...
    # Patchify
    patches = patchify_vision(images, patch_size=patch_size)

    # Calculate dimensions for the patches
    n_images, h_patches, w_patches, _ = patches.shape
    dims_virtual = (
        [1, h_patches, w_patches]
        if pixel_shuffle_scale == 1
        else [1, h_patches // pixel_shuffle_scale, w_patches // pixel_shuffle_scale]
    )

    return patches, dims_virtual
```
**EN:** This function implements `process_vision_for_patches`. The docstring states that Resize, normalize, and patchify RGB images for the vision encoder. Main inputs include `images`, `patch_size`, `max_num_patches`, `min_num_patches`, `pixel_shuffle_scale`.
**CN:** 该函数实现 `process_vision_for_patches` 相关逻辑。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `images`, `patch_size`, `max_num_patches`, `min_num_patches`, `pixel_shuffle_scale`。

### Lines 310-314: Class `IsaacImagesKwargs`
```python
class IsaacImagesKwargs(TypedDict, total=False):
    patch_size: int
    max_num_patches: int
    min_num_patches: int
    pixel_shuffle_scale: int
```
**EN:** Defines `IsaacImagesKwargs`, a processor-related class derived from `TypedDict`. Class attributes such as `patch_size`, `max_num_patches`, `min_num_patches`, `pixel_shuffle_scale` encode defaults or metadata.
**CN:** 定义 `IsaacImagesKwargs`，这是一个处理器相关类，继承自 `TypedDict`。 类属性如 `patch_size`, `max_num_patches`, `min_num_patches`, `pixel_shuffle_scale` 编码了默认值或元数据。

### Lines 317-322: Class `IsaacProcessorKwargs`
```python
class IsaacProcessorKwargs(ProcessingKwargs, total=False):  # type: ignore[call-arg]
    images_kwargs: IsaacImagesKwargs
    _defaults = {
        "text_kwargs": {"padding": False},
        "images_kwargs": {},
    }
```
**EN:** Defines `IsaacProcessorKwargs`, a processor-related class derived from `ProcessingKwargs`. Class attributes such as `images_kwargs`, `_defaults` encode defaults or metadata.
**CN:** 定义 `IsaacProcessorKwargs`，这是一个处理器相关类，继承自 `ProcessingKwargs`。 类属性如 `images_kwargs`, `_defaults` 编码了默认值或元数据。

### Lines 325-398: Class `IsaacImageProcessor`
```python
class IsaacImageProcessor:
    model_input_names = ["pixel_values", "image_grid_thw"]

    def __init__(
        self,
        patch_size: int = 16,
        vision_max_num_patches: int = 6144,
        vision_min_num_patches: int = 256,
        pixel_shuffle_scale: int = 2,
    ) -> None:
        self.patch_size = patch_size
        self.vision_max_num_patches = vision_max_num_patches
        self.vision_min_num_patches = vision_min_num_patches
        self.pixel_shuffle_scale = pixel_shuffle_scale

    def __call__(
        self,
        images: Image.Image | list[Image.Image],
        return_tensors: str | TensorType | None = None,
        **kwargs: Unpack[IsaacImagesKwargs],
    ) -> BatchFeature:
        """Preprocess images into format compatible with vLLM input processing."""
        if not isinstance(images, list):
            images = [images]

        all_pixel_values: list[torch.Tensor] = []
        all_image_grids: list[torch.Tensor] = []

        for image in images:
            image_tensor = extract_image_pil(image)

            patches, dims_virtual = process_vision_for_patches(
                image_tensor,
                patch_size=kwargs.get("patch_size", self.patch_size),
                max_num_patches=kwargs.get(
                    "max_num_patches", self.vision_max_num_patches
                ),
                min_num_patches=kwargs.get(
                    "min_num_patches", self.vision_min_num_patches
                ),
# ... omitted for brevity ...
            final_image_grids = torch.cat(all_image_grids, dim=0)
        else:
            final_pixel_values = torch.empty(0, 0)
            final_image_grids = torch.empty(0, 3)

        return BatchFeature(
            data={
                "pixel_values": final_pixel_values,
                "image_grid_thw": final_image_grids,
            },
            tensor_type=return_tensors,
        )
```
**EN:** Defines `IsaacImageProcessor`, a processor-related class. Key methods include `__init__`, `__call__`.
**CN:** 定义 `IsaacImageProcessor`，这是一个处理器相关类。 关键方法包括 `__init__`, `__call__`。

### Lines 401-499: Class `IsaacProcessor`
```python
class IsaacProcessor(ProcessorMixin):
    attributes = ["image_processor", "tokenizer"]

    def __init__(
        self,
        image_processor: IsaacImageProcessor,
        tokenizer: HfTokenizer,
        image_token: str = "<image>",
    ):
        self.image_processor = image_processor
        self.tokenizer = tokenizer

        self.image_token = image_token

    def __call__(
        self,
        text: str | list[str] | None = None,
        images: Image.Image | list[Image.Image] | None = None,
        return_tensors: str | TensorType | None = None,
        **kwargs: Unpack[IsaacProcessorKwargs],  # type: ignore[misc]
    ) -> BatchFeature:
        output_kwargs = self._merge_kwargs(
            IsaacProcessorKwargs,
            tokenizer_init_kwargs=self.tokenizer.init_kwargs,
            **kwargs,
        )

        if images is not None:
            image_inputs = self.image_processor(
                images, **output_kwargs["images_kwargs"]
            )
            image_grid_thw = image_inputs["image_grid_thw"]
        else:
            image_inputs = {}
            image_grid_thw = []

        if text is not None:
            if not isinstance(text, list):
                text = [text]

# ... omitted for brevity ...
                processed_messages.append(processed_message)
            else:
                # Regular text message
                processed_messages.append(message)

        kwargs["return_dict"] = False
        return self.tokenizer.apply_chat_template(
            processed_messages,
            tokenize=tokenize,
            add_generation_prompt=add_generation_prompt,
            **kwargs,
        )
```
**EN:** Defines `IsaacProcessor`, a processor-related class derived from `ProcessorMixin`. Key methods include `__init__`, `__call__`, `apply_chat_template`.
**CN:** 定义 `IsaacProcessor`，这是一个处理器相关类，继承自 `ProcessorMixin`。 关键方法包括 `__init__`, `__call__`, `apply_chat_template`。

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
- **EN:** Standard library modules: `math`, `typing`.
- **CN:** 标准库模块：`math`, `typing`。
- **EN:** External packages: `numpy`, `torch`, `torch.nn.functional`, `PIL`, `transformers`, `transformers.processing_utils`, `typing_extensions`.
- **CN:** 外部依赖包：`numpy`, `torch`, `torch.nn.functional`, `PIL`, `transformers`, `transformers.processing_utils`, `typing_extensions`。
- **EN:** Internal modules: `vllm.tokenizers.hf`.
- **CN:** 内部模块：`vllm.tokenizers.hf`。
