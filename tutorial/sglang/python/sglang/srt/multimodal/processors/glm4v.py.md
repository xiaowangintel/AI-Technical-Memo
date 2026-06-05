# glm4v.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/glm4v.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for glm4v models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 glm4v 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12: Imports dependencies
```python
from typing import List, Union

from sglang.srt.layers.rotary_embedding import MRotaryEmbedding
from sglang.srt.managers.schedule_batch import MultimodalProcessorOutput
from sglang.srt.models.glm4v import Glm4vForConditionalGeneration
from sglang.srt.models.glm4v_moe import Glm4vMoeForConditionalGeneration
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor as SGLangBaseProcessor,
)
from sglang.srt.multimodal.processors.base_processor import (
    MultimodalSpecialTokens,
)
```
**EN:** This block groups related imports for the module, including typing.List, typing.Union, sglang.srt.layers.rotary_embedding.MRotaryEmbedding, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput, sglang.srt.models.glm4v.Glm4vForConditionalGeneration, and 3 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 typing.List, typing.Union, sglang.srt.layers.rotary_embedding.MRotaryEmbedding, sglang.srt.managers.schedule_batch.MultimodalProcessorOutput, sglang.srt.models.glm4v.Glm4vForConditionalGeneration 等 3 项，为后续代码准备所需名称。

### Lines 13-13: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 14-17: Handles exceptional control flow
```python
try:
    from sglang.srt.models.glm_ocr import GlmOcrForConditionalGeneration
except ImportError:
    GlmOcrForConditionalGeneration = None
```
**EN:** This block protects a section of the module with exception handling and optional cleanup logic.
**CN:** 该代码块为 模块 中的一段逻辑提供异常处理与可选清理流程。

### Lines 18-19: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 20-20: Declares class Glm4vImageProcessor
```python
class Glm4vImageProcessor(SGLangBaseProcessor):
```
**EN:** This block introduces class `Glm4vImageProcessor` as a reusable abstraction inside the module. It inherits from SGLangBaseProcessor.
**CN:** 该代码块声明类 `Glm4vImageProcessor`，作为模块中的可复用抽象。 它继承自 SGLangBaseProcessor。

### Lines 21-29: Declares models
```python
    models = [
        m
        for m in [
            Glm4vForConditionalGeneration,
            Glm4vMoeForConditionalGeneration,
            GlmOcrForConditionalGeneration,
        ]
        if m is not None
    ]
```
**EN:** This statement initializes models in the Glm4vImageProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 Glm4vImageProcessor 中初始化 models。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 30-30: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Glm4vImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Glm4vImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 31-61: Defines function Glm4vImageProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)

        # GLM-V specific tokens
        self.IMAGE_TOKEN = "<|image|>"
        self.VIDEO_TOKEN = "<|video|>"
        self.IMAGE_START_TOKEN = "<|begin_of_image|>"
        self.IMAGE_END_TOKEN = "<|end_of_image|>"
        self.VIDEO_START_TOKEN = "<|begin_of_video|>"
        self.VIDEO_END_TOKEN = "<|end_of_video|>"

        # Token IDs
        self.IM_TOKEN_ID = hf_config.image_token_id
        self.VIDEO_TOKEN_ID = hf_config.video_token_id
        self.IMAGE_START_TOKEN_ID = hf_config.image_start_token_id
        self.IMAGE_END_TOKEN_ID = hf_config.image_end_token_id
        self.VIDEO_START_TOKEN_ID = hf_config.video_start_token_id
        self.VIDEO_END_TOKEN_ID = hf_config.video_end_token_id

        # Vision config
        self.IMAGE_FACTOR = 28
        self.MIN_PIXELS = 112 * 112
        self.MAX_PIXELS = 30000 * 28 * 28 * 2

        self.mm_tokens = MultimodalSpecialTokens(
            image_token=self.IMAGE_TOKEN,
            image_token_id=self.IM_TOKEN_ID,
            video_token=self.VIDEO_TOKEN,
            # Note: For GLM4v videos, it uses the video token before tokenization but uses image token after tokenization
            video_token_id=self.IM_TOKEN_ID,
        ).build(_processor)
```
**EN:** This block defines function `Glm4vImageProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `Glm4vImageProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 62-62: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Glm4vImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Glm4vImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 63-83: Defines function Glm4vImageProcessor.compute_mrope_positions
```python
    def compute_mrope_positions(self, input_ids, mm_items):
        image_grid_thw = None
        video_grid_thw = None
        for item in mm_items:
            if "image_grid_thw" in item.model_specific_data:
                image_grid_thw = item.model_specific_data["image_grid_thw"]
            if "video_grid_thw" in item.model_specific_data:
                video_grid_thw = item.model_specific_data["video_grid_thw"]

        import torch

        input_ids_tensor = torch.tensor(input_ids, dtype=torch.long).unsqueeze(0)
        attention_mask = torch.ones_like(input_ids_tensor)
        mrope_positions, mrope_position_delta = MRotaryEmbedding.get_rope_index_glm4v(
            input_ids=input_ids_tensor,
            hf_config=self.hf_config,
            image_grid_thw=image_grid_thw,
            video_grid_thw=video_grid_thw,
            attention_mask=attention_mask,
        )
        return mrope_positions.squeeze(1), mrope_position_delta
```
**EN:** This block defines function `Glm4vImageProcessor.compute_mrope_positions`. Parameters: self, input_ids, mm_items.
**CN:** 该代码块定义函数 `Glm4vImageProcessor.compute_mrope_positions`。 参数包括 self、input_ids、mm_items。

### Lines 84-84: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Glm4vImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Glm4vImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 85-123: Defines async function Glm4vImageProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self,
        image_data: List[Union[str, bytes]],
        input_text,
        request_obj,
        *args,
        **kwargs,
    ):
        base_output = self.load_mm_data(
            prompt=input_text,
            image_data=image_data,
            video_data=request_obj.video_data,
            multimodal_tokens=self.mm_tokens,
        )

        if base_output.videos:
            base_output.videos = request_obj.video_data
        mm_items, input_ids, ret = self.process_and_combine_mm_data(
            base_output, self.mm_tokens
        )

        input_ids = input_ids.flatten()
        mrope_positions, mrope_position_delta = MRotaryEmbedding.get_rope_index_glm4v(
            input_ids=input_ids.unsqueeze(0),
            hf_config=self.hf_config,
            image_grid_thw=getattr(ret, "image_grid_thw", None),
            video_grid_thw=getattr(ret, "video_grid_thw", None),
            attention_mask=getattr(ret, "attention_mask", None),
        )
        mrope_positions = mrope_positions.squeeze(1)

        return MultimodalProcessorOutput(
            input_ids=input_ids.tolist(),
            mm_items=mm_items,
            im_token_id=self.mm_tokens.image_token_id,
            video_token_id=self.mm_tokens.video_token_id,
            mrope_positions=mrope_positions,
            mrope_position_delta=mrope_position_delta,
        )
```
**EN:** This block defines async function `Glm4vImageProcessor.process_mm_data_async`. Parameters: self, image_data, input_text, request_obj.
**CN:** 该代码块定义异步函数 `Glm4vImageProcessor.process_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。

## Key Concepts / 关键概念
- **Classes / 类**: `Glm4vImageProcessor`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `typing`
- **Third-Party / 第三方**: `torch`
- **Local Modules / 本地模块**: `sglang.srt.layers.rotary_embedding`, `sglang.srt.managers.schedule_batch`, `sglang.srt.models.glm4v`, `sglang.srt.models.glm4v_moe`, `sglang.srt.models.glm_ocr`, `sglang.srt.multimodal.processors.base_processor`
