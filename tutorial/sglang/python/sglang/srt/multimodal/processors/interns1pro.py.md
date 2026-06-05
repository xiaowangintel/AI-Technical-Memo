# interns1pro.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/interns1pro.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for interns1pro models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 interns1pro 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Imports dependencies
```python
import time
from typing import List, Union

from sglang.srt.managers.schedule_batch import (
    Modality,
    MultimodalDataItem,
    MultimodalProcessorOutput,
)
from sglang.srt.models.interns1pro import InternS1ProForConditionalGeneration
from sglang.srt.multimodal.processors.qwen_vl import (
    QwenVLImageProcessor,
    preprocess_video,
)
from sglang.utils import logger
```
**EN:** This block groups related imports for the module, including time, typing.List, typing.Union, sglang.srt.managers.schedule_batch.Modality, sglang.srt.managers.schedule_batch.MultimodalDataItem, and 5 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 time, typing.List, typing.Union, sglang.srt.managers.schedule_batch.Modality, sglang.srt.managers.schedule_batch.MultimodalDataItem 等 5 项，为后续代码准备所需名称。

### Lines 15-16: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 17-17: Declares class InternS1_1ImageProcessor
```python
class InternS1_1ImageProcessor(QwenVLImageProcessor):
```
**EN:** This block introduces class `InternS1_1ImageProcessor` as a reusable abstraction inside the module. It inherits from QwenVLImageProcessor.
**CN:** 该代码块声明类 `InternS1_1ImageProcessor`，作为模块中的可复用抽象。 它继承自 QwenVLImageProcessor。

### Lines 18-20: Declares models
```python
    models = [
        InternS1ProForConditionalGeneration,
    ]
```
**EN:** This statement initializes models in the InternS1_1ImageProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 InternS1_1ImageProcessor 中初始化 models。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 21-21: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternS1_1ImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternS1_1ImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 22-41: Defines function InternS1_1ImageProcessor.get_mm_data
```python
    def get_mm_data(self, prompt, embeddings, img_grid_thw):
        input_ids, offsets = self.build_input_ids(prompt, img_grid_thw)

        mm_items = [
            MultimodalDataItem(
                modality=Modality.IMAGE,
                offsets=offsets,
                precomputed_embeddings=embeddings,
            )
        ]

        return MultimodalProcessorOutput(
            input_ids=input_ids,
            mm_items=mm_items,
            im_start_id=self.IM_START_TOKEN_ID,
            im_end_id=self.IM_END_TOKEN_ID,
            im_token_id=self.mm_tokens.image_token_id,
            video_token_id=self.mm_tokens.video_token_id,
            audio_token_id=self.mm_tokens.audio_token_id,
        )
```
**EN:** This block defines function `InternS1_1ImageProcessor.get_mm_data`. Parameters: self, prompt, embeddings, img_grid_thw.
**CN:** 该代码块定义函数 `InternS1_1ImageProcessor.get_mm_data`。 参数包括 self、prompt、embeddings、img_grid_thw。

### Lines 42-42: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternS1_1ImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternS1_1ImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 43-122: Defines async function InternS1_1ImageProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self,
        image_data: List[Union[str, bytes]],
        input_text,
        request_obj,
        *args,
        **kwargs,
    ):
        entry_time = time.perf_counter()
        base_output = self.load_mm_data(
            prompt=input_text,
            image_data=image_data,
            video_data=request_obj.video_data,
            audio_data=request_obj.audio_data,
            multimodal_tokens=self.mm_tokens,
        )
        load_time = time.perf_counter()
        rid = getattr(request_obj, "rid", "anonymous_rid")

        video_metadata = None
        if base_output.videos:
            videos_processed = [
                await preprocess_video(video, video_config=self.video_config)
                for video in base_output.videos
            ]
            base_output.videos, video_metadata = map(list, zip(*videos_processed))

        preprocess_time = time.perf_counter()

        mm_items, input_ids, ret = self.process_and_combine_mm_data(
            base_output,
            self.mm_tokens,
            video_metadata=video_metadata,
            do_sample_frames=False,
        )

        second_per_grid_ts = getattr(ret, "second_per_grid_ts", None)
        if second_per_grid_ts is None:
            second_per_grid_ts = getattr(ret, "video_second_per_grid", None)

        process_time = time.perf_counter()

        input_ids = input_ids.flatten()

        image_grid_thw = None
        if hasattr(ret, "image_grid_thw"):
            image_grid_thw = ret.image_grid_thw

        if image_grid_thw is None and image_data and isinstance(image_data[0], dict):
            image_grid_thw = image_data[0].get("image_grid_thw")

        video_grid_thw = None
        if hasattr(ret, "video_grid_thw"):
            video_grid_thw = ret.video_grid_thw

        if video_grid_thw is None and request_obj.video_data:
            first_video = request_obj.video_data[0]
            if isinstance(first_video, dict):
                video_grid_thw = first_video.get("video_grid_thw")

        get_rope_index_time = time.perf_counter()

        logger.debug(
            f"[QwenVLProcessor Perf] {rid=}, "
            f"load_time: {(load_time - entry_time) * 1000:.2f} ms, "
            f"preprocess_time: {(preprocess_time - load_time) * 1000:.2f} ms, "
            f"process_time: {(process_time - preprocess_time) * 1000:.2f} ms, "
            f"get_rope_index_time: {(get_rope_index_time - process_time) * 1000:.2f} ms, "
            f"total_time: {(get_rope_index_time - entry_time) * 1000:.2f} ms"
        )

        return MultimodalProcessorOutput(
            input_ids=input_ids.tolist(),
            mm_items=mm_items,
            im_start_id=self.vision_start_token_id,
            im_end_id=self.vision_end_token_id,
            im_token_id=self.mm_tokens.image_token_id,
            video_token_id=self.mm_tokens.video_token_id,
            audio_token_id=self.mm_tokens.audio_token_id,
        )
```
**EN:** This block defines async function `InternS1_1ImageProcessor.process_mm_data_async`. Parameters: self, image_data, input_text, request_obj.
**CN:** 该代码块定义异步函数 `InternS1_1ImageProcessor.process_mm_data_async`。 参数包括 self、image_data、input_text、request_obj。

## Key Concepts / 关键概念
- **Classes / 类**: `InternS1_1ImageProcessor`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `time`, `typing`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.models.interns1pro`, `sglang.srt.multimodal.processors.qwen_vl`, `sglang.utils`
