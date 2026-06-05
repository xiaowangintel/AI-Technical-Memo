# janus_pro.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/janus_pro.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for janus pro so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 janus pro 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-3: Comments and module notes
```python
# Adapted from:
# https://github.com/deepseek-ai/Janus/tree/main/janus/models

```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 4-22: Imports dependencies
```python
from dataclasses import dataclass
from typing import Dict, List, Tuple, Union

import numpy as np
import PIL
import torch
from PIL.Image import Image
from transformers import (
    BaseImageProcessor,
    BatchFeature,
    LlamaConfig,
    LlamaTokenizerFast,
    PretrainedConfig,
    ProcessorMixin,
)
from transformers.image_utils import to_numpy_array

from sglang.srt.configs.utils import register_image_processor, register_processor
from sglang.srt.multimodal.mm_utils import expand2square
```
**EN:** This block groups related imports for the module, including dataclasses.dataclass, typing.Dict, typing.List, typing.Tuple, typing.Union, and 14 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 dataclasses.dataclass, typing.Dict, typing.List, typing.Tuple, typing.Union 等 14 项，为后续代码准备所需名称。

### Lines 23-24: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 25-25: Declares class DictToObject
```python
class DictToObject(dict):
```
**EN:** This block introduces class `DictToObject` as a reusable abstraction inside the module. It inherits from dict.
**CN:** 该代码块声明类 `DictToObject`，作为模块中的可复用抽象。 它继承自 dict。

### Lines 26-32: Defines function DictToObject.__init__
```python
    def __init__(self, dictionary):
        super(self).__init__(dictionary)

        for key, value in dictionary.items():
            if isinstance(value, dict):
                value = DictToObject(value)
            setattr(self, key, value)
```
**EN:** This block defines function `DictToObject.__init__`. Parameters: self, dictionary.
**CN:** 该代码块定义函数 `DictToObject.__init__`。 参数包括 self、dictionary。

### Lines 33-34: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 35-35: Declares class VisionConfig
```python
class VisionConfig(PretrainedConfig):
```
**EN:** This block introduces class `VisionConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `VisionConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 36-38: Declares model_type, cls, params
```python
    model_type = "vision"
    cls: str = ""
    params = {}
```
**EN:** This block initializes a related set of values in the VisionConfig, including model_type, cls, params. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 VisionConfig 中初始化一组相关值，包括 model_type, cls, params。将这些赋值集中在一起有助于理解周边配置。

### Lines 39-39: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VisionConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VisionConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 40-47: Defines function VisionConfig.__init__
```python
    def __init__(self, **kwargs):
        super().__init__(**kwargs)

        self.cls = kwargs.get("cls", "")
        if not isinstance(self.cls, str):
            self.cls = self.cls.__name__

        self.params = kwargs.get("params", {})
```
**EN:** This block defines function `VisionConfig.__init__`. Parameters: self.
**CN:** 该代码块定义函数 `VisionConfig.__init__`。 参数包括 self。

### Lines 48-49: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 50-50: Declares class GenAlignerConfig
```python
class GenAlignerConfig(PretrainedConfig):
```
**EN:** This block introduces class `GenAlignerConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `GenAlignerConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 51-53: Declares model_type, cls, params
```python
    model_type = "gen_aligner"
    cls: str = ""
    params = {}
```
**EN:** This block initializes a related set of values in the GenAlignerConfig, including model_type, cls, params. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 GenAlignerConfig 中初始化一组相关值，包括 model_type, cls, params。将这些赋值集中在一起有助于理解周边配置。

### Lines 54-54: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the GenAlignerConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 GenAlignerConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 55-62: Defines function GenAlignerConfig.__init__
```python
    def __init__(self, **kwargs):
        super().__init__(**kwargs)

        self.cls = kwargs.get("cls", "")
        if not isinstance(self.cls, str):
            self.cls = self.cls.__name__

        self.params = kwargs.get("params", {})
```
**EN:** This block defines function `GenAlignerConfig.__init__`. Parameters: self.
**CN:** 该代码块定义函数 `GenAlignerConfig.__init__`。 参数包括 self。

### Lines 63-64: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 65-65: Declares class GenHeadConfig
```python
class GenHeadConfig(PretrainedConfig):
```
**EN:** This block introduces class `GenHeadConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `GenHeadConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 66-68: Declares model_type, cls, params
```python
    model_type = "gen_head"
    cls: str = ""
    params = {}
```
**EN:** This block initializes a related set of values in the GenHeadConfig, including model_type, cls, params. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 GenHeadConfig 中初始化一组相关值，包括 model_type, cls, params。将这些赋值集中在一起有助于理解周边配置。

### Lines 69-69: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the GenHeadConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 GenHeadConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 70-77: Defines function GenHeadConfig.__init__
```python
    def __init__(self, **kwargs):
        super().__init__(**kwargs)

        self.cls = kwargs.get("cls", "")
        if not isinstance(self.cls, str):
            self.cls = self.cls.__name__

        self.params = kwargs.get("params", {})
```
**EN:** This block defines function `GenHeadConfig.__init__`. Parameters: self.
**CN:** 该代码块定义函数 `GenHeadConfig.__init__`。 参数包括 self。

### Lines 78-79: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 80-80: Declares class AlignerConfig
```python
class AlignerConfig(PretrainedConfig):
```
**EN:** This block introduces class `AlignerConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `AlignerConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 81-83: Declares model_type, cls, params
```python
    model_type = "aligner"
    cls: str = ""
    params = {}
```
**EN:** This block initializes a related set of values in the AlignerConfig, including model_type, cls, params. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 AlignerConfig 中初始化一组相关值，包括 model_type, cls, params。将这些赋值集中在一起有助于理解周边配置。

### Lines 84-84: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the AlignerConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 AlignerConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 85-92: Defines function AlignerConfig.__init__
```python
    def __init__(self, **kwargs):
        super().__init__(**kwargs)

        self.cls = kwargs.get("cls", "")
        if not isinstance(self.cls, str):
            self.cls = self.cls.__name__

        self.params = kwargs.get("params", {})
```
**EN:** This block defines function `AlignerConfig.__init__`. Parameters: self.
**CN:** 该代码块定义函数 `AlignerConfig.__init__`。 参数包括 self。

### Lines 93-94: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 95-95: Declares class GenVisionConfig
```python
class GenVisionConfig(PretrainedConfig):
```
**EN:** This block introduces class `GenVisionConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `GenVisionConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 96-98: Declares model_type, cls, params
```python
    model_type = "gen_vision"
    cls: str = ""
    params = {}
```
**EN:** This block initializes a related set of values in the GenVisionConfig, including model_type, cls, params. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 GenVisionConfig 中初始化一组相关值，包括 model_type, cls, params。将这些赋值集中在一起有助于理解周边配置。

### Lines 99-99: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the GenVisionConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 GenVisionConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 100-107: Defines function GenVisionConfig.__init__
```python
    def __init__(self, **kwargs):
        super().__init__(**kwargs)

        self.cls = kwargs.get("cls", "")
        if not isinstance(self.cls, str):
            self.cls = self.cls.__name__

        self.params = kwargs.get("params", {})
```
**EN:** This block defines function `GenVisionConfig.__init__`. Parameters: self.
**CN:** 该代码块定义函数 `GenVisionConfig.__init__`。 参数包括 self。

### Lines 108-109: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 110-111: Declares class SigLIPVisionCfg
```python
@dataclass
class SigLIPVisionCfg:
```
**EN:** This block introduces class `SigLIPVisionCfg` as a reusable abstraction inside the module.
**CN:** 该代码块声明类 `SigLIPVisionCfg`，作为模块中的可复用抽象。

### Lines 112-121: Declares width, layers, heads, patch_size, image_size, and 5 more
```python
    width: int = 1152
    layers: Union[Tuple[int, int, int, int], int] = 27
    heads: int = 16
    patch_size: int = 14
    image_size: Union[Tuple[int, int], int] = 336
    global_pool: str = "map"
    mlp_ratio: float = 3.7362
    class_token: bool = False
    num_classes: int = 0
    use_checkpoint: bool = False
```
**EN:** This block initializes a related set of values in the SigLIPVisionCfg, including width, layers, heads, patch_size, image_size, and 5 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 SigLIPVisionCfg 中初始化一组相关值，包括 width, layers, heads, patch_size, image_size 等 5 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 122-123: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 124-124: Declares class MultiModalityConfig
```python
class MultiModalityConfig(PretrainedConfig):
```
**EN:** This block introduces class `MultiModalityConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `MultiModalityConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 125-127: Declares model_type, vision_config, aligner_config
```python
    model_type = "multi_modality"
    vision_config: VisionConfig = None
    aligner_config: AlignerConfig = None
```
**EN:** This block initializes a related set of values in the MultiModalityConfig, including model_type, vision_config, aligner_config. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 MultiModalityConfig 中初始化一组相关值，包括 model_type, vision_config, aligner_config。将这些赋值集中在一起有助于理解周边配置。

### Lines 128-128: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MultiModalityConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MultiModalityConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 129-131: Declares gen_vision_config, gen_aligner_config, gen_head_config
```python
    gen_vision_config: GenVisionConfig = None
    gen_aligner_config: GenAlignerConfig = None
    gen_head_config: GenHeadConfig = None
```
**EN:** This block initializes a related set of values in the MultiModalityConfig, including gen_vision_config, gen_aligner_config, gen_head_config. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 MultiModalityConfig 中初始化一组相关值，包括 gen_vision_config, gen_aligner_config, gen_head_config。将这些赋值集中在一起有助于理解周边配置。

### Lines 132-132: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MultiModalityConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MultiModalityConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 133-133: Declares language_config
```python
    language_config: LlamaConfig = None
```
**EN:** This statement initializes language_config in the MultiModalityConfig. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 MultiModalityConfig 中初始化 language_config。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 134-134: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MultiModalityConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MultiModalityConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 135-156: Defines function MultiModalityConfig.__init__
```python
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        vision_config = kwargs.get("vision_config", {})
        self.vision_config = VisionConfig(**vision_config)

        aligner_config = kwargs.get("aligner_config", {})
        self.aligner_config = AlignerConfig(**aligner_config)

        gen_vision_config = kwargs.get("gen_vision_config", {})
        self.gen_vision_config = GenVisionConfig(**gen_vision_config)

        gen_aligner_config = kwargs.get("gen_aligner_config", {})
        self.gen_aligner_config = GenAlignerConfig(**gen_aligner_config)

        gen_head_config = kwargs.get("gen_head_config", {})
        self.gen_head_config = GenHeadConfig(**gen_head_config)

        language_config = kwargs.get("language_config", {})
        if isinstance(language_config, LlamaConfig):
            self.language_config = language_config
        else:
            self.language_config = LlamaConfig(**language_config)
```
**EN:** This block defines function `MultiModalityConfig.__init__`. Parameters: self.
**CN:** 该代码块定义函数 `MultiModalityConfig.__init__`。 参数包括 self。

### Lines 157-158: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 159-159: Declares class VLMImageProcessor
```python
class VLMImageProcessor(BaseImageProcessor):
```
**EN:** This block introduces class `VLMImageProcessor` as a reusable abstraction inside the module. It inherits from BaseImageProcessor.
**CN:** 该代码块声明类 `VLMImageProcessor`，作为模块中的可复用抽象。 它继承自 BaseImageProcessor。

### Lines 160-160: Declares model_input_names
```python
    model_input_names = ["pixel_values"]
```
**EN:** This statement initializes model_input_names in the VLMImageProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 VLMImageProcessor 中初始化 model_input_names。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 161-161: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLMImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLMImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 162-192: Defines function VLMImageProcessor.__init__
```python
    def __init__(
        self,
        image_size: int,
        min_size: int = 14,
        image_mean: Union[Tuple[float, float, float], List[float]] = (
            0.48145466,
            0.4578275,
            0.40821073,
        ),
        image_std: Union[Tuple[float, float, float], List[float]] = (
            0.26862954,
            0.26130258,
            0.27577711,
        ),
        rescale_factor: float = 1.0 / 255.0,
        do_normalize: bool = True,
        **kwargs,
    ):
        super().__init__(**kwargs)

        self.image_size = image_size
        self.rescale_factor = rescale_factor
        self.image_mean = image_mean
        self.image_std = image_std
        self.min_size = min_size
        self.do_normalize = do_normalize

        if image_mean is None:
            self.background_color = (127, 127, 127)
        else:
            self.background_color = tuple([int(x * 255) for x in image_mean])
```
**EN:** This block defines function `VLMImageProcessor.__init__`. Parameters: self, image_size, min_size, image_mean, image_std, rescale_factor, do_normalize.
**CN:** 该代码块定义函数 `VLMImageProcessor.__init__`。 参数包括 self、image_size、min_size、image_mean、image_std、rescale_factor、do_normalize。

### Lines 193-193: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLMImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLMImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 194-247: Defines function VLMImageProcessor.resize
```python
    def resize(self, pil_img: Image) -> np.ndarray:
        """

        Args:
            pil_img (PIL.Image): [H, W, 3] in PIL.Image in RGB

        Returns:
            x (np.ndarray): [3, self.image_size, self.image_size]
        """

        width, height = pil_img.size
        max_size = max(width, height)

        size = [
            max(int(height / max_size * self.image_size), self.min_size),
            max(int(width / max_size * self.image_size), self.min_size),
        ]

        if width <= 0 or height <= 0 or size[0] <= 0 or size[1] <= 0:
            # print(f"orig size = {pil_img.size}, new size = {size}")
            raise ValueError("Invalid size!")

        def resize(
            pil_img, size, interpolation=PIL.Image.Resampling.BICUBIC, antialias=True
        ):
            if isinstance(size, int):
                w, h = pil_img.size
                if (w <= h and w == size) or (h <= w and h == size):
                    return pil_img
                if w < h:
                    ow = size
                    oh = int(size * h / w)
                else:
                    oh = size
                    ow = int(size * w / h)
                size = (ow, oh)
            else:
                size = (size[1], size[0])

            return pil_img.resize(
                size, resample=interpolation, reducing_gap=None if antialias else 3.0
            )

        pil_img = resize(
            pil_img, size, interpolation=PIL.Image.Resampling.BICUBIC, antialias=True
        )

        pil_img = expand2square(pil_img, self.background_color)
        x = to_numpy_array(pil_img)

        # [H, W, 3] -> [3, H, W]
        x = np.transpose(x, (2, 0, 1))

        return x
```
**EN:** This block defines function `VLMImageProcessor.resize`. Parameters: self, pil_img. Args: pil_img (PIL.Image): [H, W, 3] in PIL.Image in RGB Returns: x (np.ndarray): [3, self.image_size, self.image_size]
**CN:** 该代码块定义函数 `VLMImageProcessor.resize`。 参数包括 self、pil_img。 文档字符串摘要：Args: pil_img (PIL.Image): [H, W, 3] in PIL.Image in RGB Returns: x (np.ndarray): [3, self.image_size, self.image_size]

### Lines 248-248: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLMImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLMImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 249-279: Defines function VLMImageProcessor.preprocess
```python
    def preprocess(self, images, return_tensors: str = "pt", **kwargs) -> BatchFeature:
        # resize and pad to [self.image_size, self.image_size]
        # then convert from [H, W, 3] to [3, H, W]
        if not isinstance(images, list):
            images = [images]
        images: List[np.ndarray] = [self.resize(image) for image in images]
        images = [image[:3, ...] for image in images]

        # rescale from [0, 255] -> [0, 1]
        images = [
            self.rescale(
                image=image,
                scale=self.rescale_factor,
                input_data_format="channels_first",
            )
            for image in images
        ]

        # normalize
        if self.do_normalize:
            images = [
                self.normalize(
                    image=image,
                    mean=self.image_mean,
                    std=self.image_std,
                    input_data_format="channels_first",
                )
                for image in images
            ]
        data = {"pixel_values": images}
        return BatchFeature(data=data, tensor_type=return_tensors)
```
**EN:** This block defines function `VLMImageProcessor.preprocess`. Parameters: self, images, return_tensors.
**CN:** 该代码块定义函数 `VLMImageProcessor.preprocess`。 参数包括 self、images、return_tensors。

### Lines 280-280: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLMImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLMImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 281-283: Defines function VLMImageProcessor.default_shape
```python
    @property
    def default_shape(self):
        return [3, self.image_size, self.image_size]
```
**EN:** This block defines function `VLMImageProcessor.default_shape`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `VLMImageProcessor.default_shape`。 参数包括 self。 装饰器包括 property。

### Lines 284-285: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 286-286: Declares class DictOutput
```python
class DictOutput(object):
```
**EN:** This block introduces class `DictOutput` as a reusable abstraction inside the module. It inherits from object.
**CN:** 该代码块声明类 `DictOutput`，作为模块中的可复用抽象。 它继承自 object。

### Lines 287-288: Defines function DictOutput.items
```python
    def items(self):
        return self.__dict__.items()
```
**EN:** This block defines function `DictOutput.items`. Parameters: self.
**CN:** 该代码块定义函数 `DictOutput.items`。 参数包括 self。

### Lines 289-289: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DictOutput, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DictOutput 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 290-291: Defines function DictOutput.keys
```python
    def keys(self):
        return self.__dict__.keys()
```
**EN:** This block defines function `DictOutput.keys`. Parameters: self.
**CN:** 该代码块定义函数 `DictOutput.keys`。 参数包括 self。

### Lines 292-292: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DictOutput, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DictOutput 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 293-294: Defines function DictOutput.__getitem__
```python
    def __getitem__(self, item):
        return self.__dict__[item]
```
**EN:** This block defines function `DictOutput.__getitem__`. Parameters: self, item.
**CN:** 该代码块定义函数 `DictOutput.__getitem__`。 参数包括 self、item。

### Lines 295-295: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DictOutput, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DictOutput 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 296-297: Defines function DictOutput.__contains__
```python
    def __contains__(self, key):
        return key in self.__dict__
```
**EN:** This block defines function `DictOutput.__contains__`. Parameters: self, key.
**CN:** 该代码块定义函数 `DictOutput.__contains__`。 参数包括 self、key。

### Lines 298-298: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the DictOutput, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 DictOutput 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 299-300: Defines function DictOutput.__setitem__
```python
    def __setitem__(self, key, value):
        self.__dict__[key] = value
```
**EN:** This block defines function `DictOutput.__setitem__`. Parameters: self, key, value.
**CN:** 该代码块定义函数 `DictOutput.__setitem__`。 参数包括 self、key、value。

### Lines 301-302: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 303-304: Declares class VLChatProcessorOutput
```python
@dataclass
class VLChatProcessorOutput(DictOutput):
```
**EN:** This block introduces class `VLChatProcessorOutput` as a reusable abstraction inside the module. It inherits from DictOutput.
**CN:** 该代码块声明类 `VLChatProcessorOutput`，作为模块中的可复用抽象。 它继承自 DictOutput。

### Lines 305-308: Declares sft_format, input_ids, pixel_values, num_image_tokens
```python
    sft_format: str
    input_ids: torch.Tensor
    pixel_values: torch.Tensor
    num_image_tokens: torch.IntTensor
```
**EN:** This block initializes a related set of values in the VLChatProcessorOutput, including sft_format, input_ids, pixel_values, num_image_tokens. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 VLChatProcessorOutput 中初始化一组相关值，包括 sft_format, input_ids, pixel_values, num_image_tokens。将这些赋值集中在一起有助于理解周边配置。

### Lines 309-309: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLChatProcessorOutput, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLChatProcessorOutput 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 310-311: Defines function VLChatProcessorOutput.__len__
```python
    def __len__(self):
        return len(self.input_ids)
```
**EN:** This block defines function `VLChatProcessorOutput.__len__`. Parameters: self.
**CN:** 该代码块定义函数 `VLChatProcessorOutput.__len__`。 参数包括 self。

### Lines 312-313: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 314-315: Declares class BatchedVLChatProcessorOutput
```python
@dataclass
class BatchedVLChatProcessorOutput(DictOutput):
```
**EN:** This block introduces class `BatchedVLChatProcessorOutput` as a reusable abstraction inside the module. It inherits from DictOutput.
**CN:** 该代码块声明类 `BatchedVLChatProcessorOutput`，作为模块中的可复用抽象。 它继承自 DictOutput。

### Lines 316-321: Declares sft_format, input_ids, pixel_values, attention_mask, images_seq_mask, and 1 more
```python
    sft_format: List[str]
    input_ids: torch.Tensor
    pixel_values: torch.Tensor
    attention_mask: torch.Tensor
    images_seq_mask: torch.BoolTensor
    images_emb_mask: torch.BoolTensor
```
**EN:** This block initializes a related set of values in the BatchedVLChatProcessorOutput, including sft_format, input_ids, pixel_values, attention_mask, images_seq_mask, and 1 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 BatchedVLChatProcessorOutput 中初始化一组相关值，包括 sft_format, input_ids, pixel_values, attention_mask, images_seq_mask 等 1 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 322-325: Comments and module notes
```python


# FIXME: had to place Official Processor here, since image_processor module would not be imported in all threads,
# hence AutoProcessor registration would not be affective in some cases
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 326-326: Declares class VLChatProcessor
```python
class VLChatProcessor(ProcessorMixin):
```
**EN:** This block introduces class `VLChatProcessor` as a reusable abstraction inside the module. It inherits from ProcessorMixin.
**CN:** 该代码块声明类 `VLChatProcessor`，作为模块中的可复用抽象。 它继承自 ProcessorMixin。

### Lines 327-328: Declares image_processor_class, tokenizer_class
```python
    image_processor_class = "AutoImageProcessor"
    tokenizer_class = ("LlamaTokenizer", "LlamaTokenizerFast")
```
**EN:** This block initializes a related set of values in the VLChatProcessor, including image_processor_class, tokenizer_class. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 VLChatProcessor 中初始化一组相关值，包括 image_processor_class, tokenizer_class。将这些赋值集中在一起有助于理解周边配置。

### Lines 329-329: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLChatProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLChatProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 330-330: Declares attributes
```python
    attributes = ["image_processor", "tokenizer"]
```
**EN:** This statement initializes attributes in the VLChatProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 VLChatProcessor 中初始化 attributes。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 331-331: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLChatProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLChatProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 332-371: Defines function VLChatProcessor.__init__
```python
    def __init__(
        self,
        image_processor: VLMImageProcessor,
        tokenizer: LlamaTokenizerFast,
        image_tag: str = "<image_placeholder>",
        image_start_tag: str = "<begin_of_image>",
        image_end_tag: str = "<end_of_image>",
        pad_tag: str = "<｜▁pad▁｜>",
        num_image_tokens: int = 576,
        add_special_token: bool = False,
        sft_format: str = "deepseek",
        mask_prompt: bool = True,
        ignore_id: int = -100,
        **kwargs,
    ):
        self.image_processor = image_processor
        self.tokenizer = tokenizer

        image_id = self.tokenizer.vocab.get(image_tag)
        if image_id is None:
            special_tokens = [image_tag]
            special_tokens_dict = {"additional_special_tokens": special_tokens}
            self.tokenizer.add_special_tokens(special_tokens_dict)
            # print(f"Add image tag = {image_tag} to the tokenizer")

        self.image_tag = image_tag
        self.image_start_tag = image_start_tag
        self.image_end_tag = image_end_tag
        self.pad_tag = pad_tag

        self.num_image_tokens = num_image_tokens
        self.add_special_token = add_special_token
        self.sft_format = sft_format
        self.ignore_id = ignore_id

        super().__init__(
            image_processor,
            tokenizer,
            **kwargs,
        )
```
**EN:** This block defines function `VLChatProcessor.__init__`. Parameters: self, image_processor, tokenizer, image_tag, image_start_tag, image_end_tag, pad_tag, num_image_tokens, add_special_token, sft_format, mask_prompt, ignore_id.
**CN:** 该代码块定义函数 `VLChatProcessor.__init__`。 参数包括 self、image_processor、tokenizer、image_tag、image_start_tag、image_end_tag、pad_tag、num_image_tokens、add_special_token、sft_format、mask_prompt、ignore_id。

### Lines 372-372: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLChatProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLChatProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 373-375: Defines function VLChatProcessor.image_token
```python
    @property
    def image_token(self):
        return self.image_tag
```
**EN:** This block defines function `VLChatProcessor.image_token`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `VLChatProcessor.image_token`。 参数包括 self。 装饰器包括 property。

### Lines 376-376: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLChatProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLChatProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 377-380: Defines function VLChatProcessor.image_id
```python
    @property
    def image_id(self) -> int:
        image_id = self.tokenizer.vocab.get(self.image_tag)
        return image_id
```
**EN:** This block defines function `VLChatProcessor.image_id`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `VLChatProcessor.image_id`。 参数包括 self。 装饰器包括 property。

### Lines 381-381: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLChatProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLChatProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 382-385: Defines function VLChatProcessor.image_start_id
```python
    @property
    def image_start_id(self):
        image_start_id = self.tokenizer.vocab.get(self.image_start_tag)
        return image_start_id
```
**EN:** This block defines function `VLChatProcessor.image_start_id`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `VLChatProcessor.image_start_id`。 参数包括 self。 装饰器包括 property。

### Lines 386-386: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLChatProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLChatProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 387-390: Defines function VLChatProcessor.image_end_id
```python
    @property
    def image_end_id(self):
        image_end_id = self.tokenizer.vocab.get(self.image_end_tag)
        return image_end_id
```
**EN:** This block defines function `VLChatProcessor.image_end_id`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `VLChatProcessor.image_end_id`。 参数包括 self。 装饰器包括 property。

### Lines 391-391: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLChatProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLChatProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 392-394: Defines function VLChatProcessor.image_start_token
```python
    @property
    def image_start_token(self):
        return self.image_start_tag
```
**EN:** This block defines function `VLChatProcessor.image_start_token`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `VLChatProcessor.image_start_token`。 参数包括 self。 装饰器包括 property。

### Lines 395-395: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLChatProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLChatProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 396-398: Defines function VLChatProcessor.image_end_token
```python
    @property
    def image_end_token(self):
        return self.image_end_tag
```
**EN:** This block defines function `VLChatProcessor.image_end_token`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `VLChatProcessor.image_end_token`。 参数包括 self。 装饰器包括 property。

### Lines 399-399: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLChatProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLChatProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 400-403: Defines function VLChatProcessor.pad_id
```python
    @property
    def pad_id(self):
        pad_id = self.tokenizer.vocab.get(self.pad_tag)
        return pad_id
```
**EN:** This block defines function `VLChatProcessor.pad_id`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `VLChatProcessor.pad_id`。 参数包括 self。 装饰器包括 property。

### Lines 404-404: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLChatProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLChatProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 405-448: Defines function VLChatProcessor.add_image_token
```python
    def add_image_token(
        self,
        image_indices: List[int],
        input_ids: torch.LongTensor,
    ):
        """

        Args:
            image_indices (List[int]): [index_0, index_1, ..., index_j]
            input_ids (torch.LongTensor): [N]

        Returns:
            input_ids (torch.LongTensor): [N + image tokens]
            num_image_tokens (torch.IntTensor): [n_images]
        """

        input_slices = []

        start = 0
        for index in image_indices:
            if self.add_special_token:
                end = index + 1
            else:
                end = index

            # original text tokens
            input_slices.append(input_ids[start:end])

            # add boi, image tokens, eoi and set the mask as False
            input_slices.append(self.image_start_id * torch.ones((1), dtype=torch.long))
            input_slices.append(
                self.image_id * torch.ones((self.num_image_tokens,), dtype=torch.long)
            )
            input_slices.append(self.image_end_id * torch.ones((1), dtype=torch.long))
            start = index + 1

        # the left part
        input_slices.append(input_ids[start:])

        # concat all slices
        input_ids = torch.cat(input_slices, dim=0)
        num_image_tokens = torch.IntTensor([self.num_image_tokens] * len(image_indices))

        return input_ids, num_image_tokens
```
**EN:** This block defines function `VLChatProcessor.add_image_token`. Parameters: self, image_indices, input_ids. Args: image_indices (List[int]): [index_0, index_1, ..., index_j] input_ids (torch.LongTensor): [N] Returns: input_ids (torch.LongTensor): [N + image tokens] num_image_tokens (torch.IntTensor): [n_images]
**CN:** 该代码块定义函数 `VLChatProcessor.add_image_token`。 参数包括 self、image_indices、input_ids。 文档字符串摘要：Args: image_indices (List[int]): [index_0, index_1, ..., index_j] input_ids (torch.LongTensor): [N] Returns: input_ids (torch.LongTensor): [N + image tokens] num_image_tokens (torch.IntTensor): [n_images]

### Lines 449-449: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLChatProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLChatProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 450-495: Defines function VLChatProcessor.process_one
```python
    def process_one(
        self,
        prompt: str = None,
        images: List[Image] = None,
        **kwargs,
    ):
        """

        Args:
            prompt (str): the formatted prompt;
            images (List[ImageType]): the list of images;
            **kwargs:

        Returns:
            outputs (BaseProcessorOutput): the output of the processor,
                - input_ids (torch.LongTensor): [N + image tokens]
                - target_ids (torch.LongTensor): [N + image tokens]
                - images (torch.FloatTensor): [n_images, 3, H, W]
                - image_id (int): the id of the image token
                - num_image_tokens (List[int]): the number of image tokens
        """

        sft_format = prompt
        # tokenize
        input_ids = self.tokenizer.encode(sft_format)
        input_ids = torch.LongTensor(input_ids)

        # add image tokens to the input_ids
        image_token_mask: torch.Tensor = (input_ids == self.image_id).to(torch.bool)
        image_indices = image_token_mask.nonzero()
        input_ids, num_image_tokens = self.add_image_token(
            image_indices=image_indices,
            input_ids=input_ids,
        )

        # load images
        images_outputs = self.image_processor(images, return_tensors="pt")

        prepare = VLChatProcessorOutput(
            sft_format=sft_format,
            input_ids=input_ids,
            pixel_values=images_outputs.pixel_values,
            num_image_tokens=num_image_tokens,
        )

        return prepare
```
**EN:** This block defines function `VLChatProcessor.process_one`. Parameters: self, prompt, images. Args: prompt (str): the formatted prompt; images (List[ImageType]): the list of images; **kwargs: Returns: outputs (BaseProcessorOutput): the output of the processor, - input_ids (torch.LongTensor): [N + image tokens] - target_ids (torch.LongTensor): [N + imag
**CN:** 该代码块定义函数 `VLChatProcessor.process_one`。 参数包括 self、prompt、images。 文档字符串摘要：Args: prompt (str): the formatted prompt; images (List[ImageType]): the list of images; **kwargs: Returns: outputs (BaseProcessorOutput): the output of the processor, - input_ids (torch.LongTensor): [N + image tokens] - target_ids (torch.LongTensor): [N + imag

### Lines 496-496: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLChatProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLChatProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 497-530: Defines function VLChatProcessor.__call__
```python
    def __call__(
        self,
        *,
        prompt: str = None,
        conversations: List[Dict[str, str]] = None,
        images: List[Image] = None,
        force_batchify: bool = True,
        **kwargs,
    ):
        """

        Args:
            prompt (str): the formatted prompt;
            conversations (List[Dict]): conversations with a list of messages;
            images (List[ImageType]): the list of images;
            force_batchify (bool): force batchify the inputs;
            **kwargs:

        Returns:
            outputs (BaseProcessorOutput): the output of the processor,
                - input_ids (torch.LongTensor): [N + image tokens]
                - images (torch.FloatTensor): [n_images, 3, H, W]
                - image_id (int): the id of the image token
                - num_image_tokens (List[int]): the number of image tokens
        """

        prepare = self.process_one(
            prompt=prompt, conversations=conversations, images=images
        )

        if force_batchify:
            prepare = self.batchify([prepare])

        return prepare
```
**EN:** This block defines function `VLChatProcessor.__call__`. Parameters: self. Args: prompt (str): the formatted prompt; conversations (List[Dict]): conversations with a list of messages; images (List[ImageType]): the list of images; force_batchify (bool): force batchify the inputs; **kwargs: Returns: outputs (BaseProcessorOutput): the o
**CN:** 该代码块定义函数 `VLChatProcessor.__call__`。 参数包括 self。 文档字符串摘要：Args: prompt (str): the formatted prompt; conversations (List[Dict]): conversations with a list of messages; images (List[ImageType]): the list of images; force_batchify (bool): force batchify the inputs; **kwargs: Returns: outputs (BaseProcessorOutput): the o

### Lines 531-531: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLChatProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLChatProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 532-593: Defines function VLChatProcessor.batchify
```python
    def batchify(
        self, prepare_list: List[VLChatProcessorOutput]
    ) -> BatchedVLChatProcessorOutput:
        """
        Preprocesses the inputs for multimodal inference.

        Args:
            prepare_list (List[VLChatProcessorOutput]): A list of VLChatProcessorOutput.

        Returns:
            BatchedVLChatProcessorOutput: A dictionary of the inputs to use for multimodal inference.
        """

        batch_size = len(prepare_list)
        sft_format = []
        n_images = []
        seq_lens = []
        for prepare in prepare_list:
            n_images.append(len(prepare.num_image_tokens))
            seq_lens.append(len(prepare))

        input_token_max_len = max(seq_lens)
        max_n_images = max(1, max(n_images))

        batched_input_ids = torch.full(
            (batch_size, input_token_max_len), self.pad_id
        ).long()  # FIXME
        batched_attention_mask = torch.zeros((batch_size, input_token_max_len)).long()
        batched_pixel_values = torch.zeros(
            (batch_size, max_n_images, *self.image_processor.default_shape)
        ).float()
        batched_images_seq_mask = torch.zeros((batch_size, input_token_max_len)).bool()
        batched_images_emb_mask = torch.zeros(
            (batch_size, max_n_images, self.num_image_tokens)
        ).bool()

        for i, prepare in enumerate(prepare_list):
            input_ids = prepare.input_ids
            seq_len = len(prepare)
            n_image = len(prepare.num_image_tokens)
            # left-padding
            batched_attention_mask[i, -seq_len:] = 1
            batched_input_ids[i, -seq_len:] = torch.LongTensor(input_ids)
            batched_images_seq_mask[i, -seq_len:] = input_ids == self.image_id

            if n_image > 0:
                batched_pixel_values[i, :n_image] = prepare.pixel_values
                for j, n_image_tokens in enumerate(prepare.num_image_tokens):
                    batched_images_emb_mask[i, j, :n_image_tokens] = True

            sft_format.append(prepare.sft_format)

        batched_prepares = BatchedVLChatProcessorOutput(
            input_ids=batched_input_ids,
            attention_mask=batched_attention_mask,
            pixel_values=batched_pixel_values,
            images_seq_mask=batched_images_seq_mask,
            images_emb_mask=batched_images_emb_mask,
            sft_format=sft_format,
        )

        return batched_prepares
```
**EN:** This block defines function `VLChatProcessor.batchify`. Parameters: self, prepare_list. Preprocesses the inputs for multimodal inference. Args: prepare_list (List[VLChatProcessorOutput]): A list of VLChatProcessorOutput.
**CN:** 该代码块定义函数 `VLChatProcessor.batchify`。 参数包括 self、prepare_list。 文档字符串摘要：Preprocesses the inputs for multimodal inference. Args: prepare_list (List[VLChatProcessorOutput]): A list of VLChatProcessorOutput.

### Lines 594-595: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 596-596: Declares class VLMImageProcessorConfig
```python
class VLMImageProcessorConfig(PretrainedConfig):
```
**EN:** This block introduces class `VLMImageProcessorConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `VLMImageProcessorConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 597-603: Declares model_type, image_size, min_size, image_mean, image_std, and 2 more
```python
    model_type = "deepseek_vlm"
    image_size: int = None
    min_size: int = None
    image_mean: Union[Tuple[float, float, float], List[float]] = None
    image_std: Union[Tuple[float, float, float], List[float]] = None
    rescale_factor: float = None
    do_normalize: bool = None
```
**EN:** This block initializes a related set of values in the VLMImageProcessorConfig, including model_type, image_size, min_size, image_mean, image_std, and 2 more. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 VLMImageProcessorConfig 中初始化一组相关值，包括 model_type, image_size, min_size, image_mean, image_std 等 2 项。将这些赋值集中在一起有助于理解周边配置。

### Lines 604-604: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VLMImageProcessorConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VLMImageProcessorConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 605-630: Defines function VLMImageProcessorConfig.__init__
```python
    def __init__(
        self,
        image_size: int,
        min_size: int = 14,
        image_mean: Union[Tuple[float, float, float], List[float]] = (
            0.48145466,
            0.4578275,
            0.40821073,
        ),
        image_std: Union[Tuple[float, float, float], List[float]] = (
            0.26862954,
            0.26130258,
            0.27577711,
        ),
        rescale_factor: float = 1.0 / 255.0,
        do_normalize: bool = True,
        **kwargs,
    ):
        self.image_size = image_size
        self.min_size = min_size
        self.image_mean = image_mean
        self.image_std = image_std
        self.rescale_factor = rescale_factor
        self.do_normalize = do_normalize

        super().__init__(**kwargs)
```
**EN:** This block defines function `VLMImageProcessorConfig.__init__`. Parameters: self, image_size, min_size, image_mean, image_std, rescale_factor, do_normalize.
**CN:** 该代码块定义函数 `VLMImageProcessorConfig.__init__`。 参数包括 self、image_size、min_size、image_mean、image_std、rescale_factor、do_normalize。

### Lines 631-632: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 633-633: Implements Expr
```python
register_processor(MultiModalityConfig, VLChatProcessor)
```
**EN:** This block contributes general-purpose logic to the module using a `Expr` statement.
**CN:** 该代码块通过 `Expr` 语句为 模块 补充通用逻辑。

### Lines 634-634: Implements Expr
```python
register_image_processor(MultiModalityConfig, VLMImageProcessor)
```
**EN:** This block contributes general-purpose logic to the module using a `Expr` statement.
**CN:** 该代码块通过 `Expr` 语句为 模块 补充通用逻辑。

## Key Concepts / 关键概念
- **Classes / 类**: `DictToObject`, `VisionConfig`, `GenAlignerConfig`, `GenHeadConfig`, `AlignerConfig`, `GenVisionConfig`, `SigLIPVisionCfg`, `MultiModalityConfig`, `VLMImageProcessor`, `DictOutput`, `VLChatProcessorOutput`, `BatchedVLChatProcessorOutput`, `VLChatProcessor`, `VLMImageProcessorConfig`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `dataclasses`, `typing`
- **Third-Party / 第三方**: `PIL`, `PIL.Image`, `numpy`, `torch`, `transformers`, `transformers.image_utils`
- **Local Modules / 本地模块**: `sglang.srt.configs.utils`, `sglang.srt.multimodal.mm_utils`
