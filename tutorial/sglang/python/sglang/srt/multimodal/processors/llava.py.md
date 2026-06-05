# llava.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/llava.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for llava models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 llava 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31: Imports dependencies
```python
import asyncio
import os
from typing import Dict, List, Optional, Union

import numpy as np
from transformers.models.auto.processing_auto import (
    PROCESSOR_MAPPING_NAMES as HF_MAPPING_NAMES,
)

import sglang.srt.managers.multimodal_processor as sgl_mm_processor_utils
from sglang.srt.managers.schedule_batch import (
    Modality,
    MultimodalDataItem,
    MultimodalProcessorOutput,
)
from sglang.srt.models.llava import (
    LlavaForConditionalGeneration,
    LlavaLlamaForCausalLM,
    LlavaMistralForCausalLM,
    LlavaQwenForCausalLM,
)
from sglang.srt.models.llavavid import LlavaVidForCausalLM
from sglang.srt.models.mistral import Mistral3ForConditionalGeneration
from sglang.srt.multimodal.mm_utils import (
    ensure_numpy,
    expand2square,
    process_anyres_image,
)
from sglang.srt.multimodal.processors.base_processor import BaseMultimodalProcessor
from sglang.srt.utils import ImageData, load_image, logger
from sglang.utils import get_exception_traceback
```
**EN:** This block groups related imports for the module, including asyncio, os, typing.Dict, typing.List, typing.Optional, and 21 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 asyncio, os, typing.Dict, typing.List, typing.Optional 等 21 项，为后续代码准备所需名称。

### Lines 32-33: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 34-34: Declares class LlavaImageProcessor
```python
class LlavaImageProcessor(BaseMultimodalProcessor):
```
**EN:** This block introduces class `LlavaImageProcessor` as a reusable abstraction inside the module. It inherits from BaseMultimodalProcessor.
**CN:** 该代码块声明类 `LlavaImageProcessor`，作为模块中的可复用抽象。 它继承自 BaseMultimodalProcessor。

### Lines 35-41: Declares models, gpu_image_decode
```python
    models = [
        LlavaLlamaForCausalLM,
        LlavaVidForCausalLM,
        LlavaQwenForCausalLM,
        LlavaMistralForCausalLM,
    ]
    gpu_image_decode = False  # Llava processes loaded image as PIL image explicitly
```
**EN:** This block initializes a related set of values in the LlavaImageProcessor, including models, gpu_image_decode. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 LlavaImageProcessor 中初始化一组相关值，包括 models, gpu_image_decode。将这些赋值集中在一起有助于理解周边配置。

### Lines 42-42: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the LlavaImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 LlavaImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 43-44: Defines function LlavaImageProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)
```
**EN:** This block defines function `LlavaImageProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `LlavaImageProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 45-45: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the LlavaImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 LlavaImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 46-94: Defines function LlavaImageProcessor._process_single_image_task
```python
    @staticmethod
    def _process_single_image_task(
        image_data: Union[str, bytes, ImageData],
        image_aspect_ratio: Optional[str] = None,
        image_grid_pinpoints: Optional[str] = None,
        processor=None,
    ):

        image_processor = processor.image_processor

        try:
            url = image_data.url if isinstance(image_data, ImageData) else image_data
            image, image_size = load_image(url, False)
            if image_size is not None:
                # It is a video with multiple images
                image_hash = hash(url)
                pixel_values = image_processor(image)["pixel_values"]
                for i in range(len(pixel_values)):
                    pixel_values[i] = ensure_numpy(pixel_values[i]).astype(np.float16)
                pixel_values = np.stack(pixel_values, axis=0)
                return pixel_values, image_hash, image_size
            else:
                # It is an image
                image_hash = hash(url)
                if image_aspect_ratio == "pad":
                    image = expand2square(
                        image,
                        tuple(int(x * 255) for x in image_processor.image_mean),
                    )
                    pixel_values = image_processor(image.convert("RGB"))[
                        "pixel_values"
                    ][0]
                elif image_aspect_ratio == "anyres" or (
                    image_aspect_ratio is not None
                    and "anyres_max" in image_aspect_ratio
                ):
                    pixel_values = process_anyres_image(
                        image, image_processor, image_grid_pinpoints
                    )
                else:
                    pixel_values = image_processor(image)["pixel_values"][0]

                pixel_values = ensure_numpy(pixel_values)
                if isinstance(pixel_values, np.ndarray):
                    pixel_values = pixel_values.astype(np.float16)

                return pixel_values, image_hash, image.size
        except Exception:
            logger.error("Exception in TokenizerManager:\n" + get_exception_traceback())
```
**EN:** This block defines function `LlavaImageProcessor._process_single_image_task`. Parameters: image_data, image_aspect_ratio, image_grid_pinpoints, processor. Decorators: staticmethod.
**CN:** 该代码块定义函数 `LlavaImageProcessor._process_single_image_task`。 参数包括 image_data、image_aspect_ratio、image_grid_pinpoints、processor。 装饰器包括 staticmethod。

### Lines 95-95: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the LlavaImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 LlavaImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 96-120: Defines async function LlavaImageProcessor._process_single_image
```python
    async def _process_single_image(
        self,
        image_data: Union[bytes, str, ImageData],
        aspect_ratio: str,
        grid_pinpoints: str,
    ):
        if self.cpu_executor is not None:
            loop = asyncio.get_running_loop()
            fut = loop.run_in_executor(
                self.cpu_executor,
                LlavaImageProcessor._process_single_image_task,
                image_data,
                aspect_ratio,
                grid_pinpoints,
                self._processor,
            )
            timeout = int(os.environ.get("REQUEST_TIMEOUT", "10"))
            return await asyncio.wait_for(fut, timeout=timeout)
        else:
            return self._process_single_image_task(
                image_data,
                aspect_ratio,
                grid_pinpoints,
                self._processor.image_processor,
            )
```
**EN:** This block defines async function `LlavaImageProcessor._process_single_image`. Parameters: self, image_data, aspect_ratio, grid_pinpoints.
**CN:** 该代码块定义异步函数 `LlavaImageProcessor._process_single_image`。 参数包括 self、image_data、aspect_ratio、grid_pinpoints。

### Lines 121-121: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the LlavaImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 LlavaImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 122-146: Defines function LlavaImageProcessor._process_precomputed_image_data
```python
    def _process_precomputed_image_data(self, image_data: List[Dict]) -> Dict:
        mm_items = []
        for item in image_data:
            # Infer size logic...
            if "image_sizes" not in item:
                if "pixel_values" in item:
                    pv = item["pixel_values"]
                    # Handle simplified if/else
                    h, w = (
                        (pv.shape[2], pv.shape[3])
                        if len(pv.shape) == 4
                        else (pv.shape[1], pv.shape[2])
                    )
                    item["image_sizes"] = [(w, h)]
                else:
                    item["image_sizes"] = [(336, 336)]

            mm_items.append(
                MultimodalDataItem(
                    feature=item["feature"],
                    modality=Modality.IMAGE,
                    model_specific_data=item,
                )
            )
        return MultimodalProcessorOutput(mm_items=mm_items)
```
**EN:** This block defines function `LlavaImageProcessor._process_precomputed_image_data`. Parameters: self, image_data.
**CN:** 该代码块定义函数 `LlavaImageProcessor._process_precomputed_image_data`。 参数包括 self、image_data。

### Lines 147-147: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the LlavaImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 LlavaImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 148-227: Defines async function LlavaImageProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self,
        image_data: List[Union[str, bytes, ImageData]],
        input_text,
        request_obj,
        *args,
        **kwargs,
    ):
        # FIX: Handle precomputed embeddings (dictionaries)
        # If the input is already a dictionary, we skip the CPU image processor.
        # We also need to infer 'image_sizes' from 'pixel_values' if missing,
        # because pad_input_ids requires it.
        if (
            isinstance(image_data, list)
            and len(image_data) > 0
            and isinstance(image_data[0], dict)
        ):
            return self._process_precomputed_image_data(image_data)

        modalities = request_obj.modalities or ["image"]
        aspect_ratio = getattr(self.hf_config, "image_aspect_ratio", None)
        grid_pinpoints = (
            self.hf_config.image_grid_pinpoints
            if hasattr(self.hf_config, "image_grid_pinpoints")
            and "anyres" in aspect_ratio
            else None
        )

        if isinstance(image_data, list) and len(image_data) > 0:
            if "multi-images" in modalities or "video" in modalities:
                # Multiple images
                aspect_ratio = "pad"  # LLaVA OneVision Handling: more than one image --> interleaved image mode or video mode. We do not use anyres
                pixel_values, data_hashes, image_sizes = [], [], []
                res = []
                for img_data in image_data:
                    res.append(
                        self._process_single_image(
                            img_data, aspect_ratio, grid_pinpoints
                        )
                    )

                res = await asyncio.gather(*res)
                for pixel_v, image_h, image_s in res:
                    pixel_values.append(pixel_v)
                    data_hashes.append(image_h)
                    image_sizes.append(image_s)
            else:
                # A single image
                pixel_values, image_hash, image_size = await self._process_single_image(
                    image_data[0], aspect_ratio, grid_pinpoints
                )
                pixel_values = [pixel_values]
                image_sizes = [image_size]
        else:
            raise ValueError(f"Invalid image data: {image_data}")
        modality = Modality.IMAGE
        if isinstance(request_obj.modalities, list):
            if request_obj.modalities[0] == "video":
                modality = Modality.VIDEO

        # Create one item per image for better cache granularity
        mm_items = []
        for pixel_v, image_s in zip(pixel_values, image_sizes):
            # Ensure ndim=4 so the model forward takes the correct encode branch
            if isinstance(pixel_v, np.ndarray) and pixel_v.ndim == 3:
                pixel_v = np.expand_dims(pixel_v, 0)
            mm_items.append(
                MultimodalDataItem(
                    feature=pixel_v,
                    model_specific_data={
                        "image_sizes": [image_s],
                        "image_aspect_ratio": aspect_ratio,
                    },
                    modality=modality,
                )
            )

        return MultimodalProcessorOutput(
            mm_items=mm_items,
        )
```
**EN:** This block defines async function `LlavaImageProcessor.process_mm_data_async`. Parameters: self, image_data, input_text, request_obj.
**CN:** 该代码块定义异步函数 `LlavaImageProcessor.process_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。

### Lines 228-229: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 230-230: Declares class LlavaMultimodalProcessor
```python
class LlavaMultimodalProcessor(BaseMultimodalProcessor):
```
**EN:** This block introduces class `LlavaMultimodalProcessor` as a reusable abstraction inside the module. It inherits from BaseMultimodalProcessor. This is a wrapper class used to identify the multimodal processor for Llava architectures' vision model.
**CN:** 该代码块声明类 `LlavaMultimodalProcessor`，作为模块中的可复用抽象。 它继承自 BaseMultimodalProcessor。 文档字符串摘要：This is a wrapper class used to identify the multimodal processor for Llava architectures' vision model.

### Lines 231-233: Documents the scope
```python
    """
    This is a wrapper class used to identify the multimodal processor for Llava architectures' vision model.
    """
```
**EN:** This string literal serves as documentation for the LlavaMultimodalProcessor, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 LlavaMultimodalProcessor 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 234-234: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the LlavaMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 LlavaMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 235-235: Declares models
```python
    models = [LlavaForConditionalGeneration, Mistral3ForConditionalGeneration]
```
**EN:** This statement initializes models in the LlavaMultimodalProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 LlavaMultimodalProcessor 中初始化 models。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 236-236: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the LlavaMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 LlavaMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 237-249: Defines function LlavaMultimodalProcessor._get_sgl_processor_cls
```python
    def _get_sgl_processor_cls(self, model_type: str):
        if model_type == "clip_vision_model":
            return LlavaImageProcessor
        if hf_name := HF_MAPPING_NAMES.get(model_type):
            sgl_mm_processor_set = sgl_mm_processor_utils.PROCESSOR_MAPPING.values()
            sgl_processor_cls = list(
                filter(lambda p: p.__name__ == hf_name, sgl_mm_processor_set)
            )
            if sgl_processor_cls:
                return sgl_processor_cls[0]
        raise ValueError(
            f"Cannot find corresponding multimodal processor registered in sglang for model type `{model_type}`"
        )
```
**EN:** This block defines function `LlavaMultimodalProcessor._get_sgl_processor_cls`. Parameters: self, model_type.
**CN:** 该代码块定义函数 `LlavaMultimodalProcessor._get_sgl_processor_cls`。 参数包括 self、model_type。

### Lines 250-250: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the LlavaMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 LlavaMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 251-265: Defines function LlavaMultimodalProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        assert hasattr(hf_config, "vision_config")
        assert hasattr(hf_config, "text_config")
        self.vision_config = hf_config.vision_config
        self.text_config = hf_config.text_config
        self.hf_config = hf_config

        if vision_type := getattr(self.vision_config, "model_type"):
            self.inner = self._get_sgl_processor_cls(vision_type)(
                hf_config, server_args, _processor, *args, **kwargs
            )
        else:
            raise ValueError(
                f"Required `vision_config.model_type` is not found in hf_config: `{hf_config}`"
            )
```
**EN:** This block defines function `LlavaMultimodalProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `LlavaMultimodalProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 266-266: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the LlavaMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 LlavaMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 267-268: Defines async function LlavaMultimodalProcessor.process_mm_data_async
```python
    async def process_mm_data_async(self, *args, **kwargs):
        return await self.inner.process_mm_data_async(*args, **kwargs)
```
**EN:** This block defines async function `LlavaMultimodalProcessor.process_mm_data_async`. Parameters: self.
**CN:** 该代码块定义异步函数 `LlavaMultimodalProcessor.process_mm_data_async`。 参数包括 self。

## Key Concepts / 关键概念
- **Classes / 类**: `LlavaImageProcessor`, `LlavaMultimodalProcessor`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `asyncio`, `os`, `typing`
- **Third-Party / 第三方**: `numpy`, `transformers.models.auto.processing_auto`
- **Local Modules / 本地模块**: `sglang.srt.managers.multimodal_processor`, `sglang.srt.managers.schedule_batch`, `sglang.srt.models.llava`, `sglang.srt.models.llavavid`, `sglang.srt.models.mistral`, `sglang.srt.multimodal.mm_utils`, `sglang.srt.multimodal.processors.base_processor`, `sglang.srt.utils`, `sglang.utils`
