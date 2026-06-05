# gemma4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/gemma4.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for gemma4 models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 gemma4 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Comments and module notes
```python
# Copyright 2025 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================

```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 15-27: Imports dependencies
```python
from typing import Dict, List, Optional, Union

import numpy as np
import torch

from sglang.srt.managers.multimodal_processor import (
    BaseMultimodalProcessor as SGLangBaseProcessor,
)
from sglang.srt.managers.schedule_batch import Modality, MultimodalProcessorOutput
from sglang.srt.models.gemma4_audio import _SSCP_CONV_STRIDE_SIZES
from sglang.srt.models.gemma4_mm import Gemma4ForConditionalGeneration
from sglang.srt.multimodal.processors.base_processor import MultimodalSpecialTokens
from sglang.srt.utils.video_decoder import VideoDecoderWrapper
```
**EN:** This block groups related imports for the module, including typing.Dict, typing.List, typing.Optional, typing.Union, numpy, and 8 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 typing.Dict, typing.List, typing.Optional, typing.Union, numpy 等 8 项，为后续代码准备所需名称。

### Lines 28-29: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 30-30: Declares class Gemma4SGLangProcessor
```python
class Gemma4SGLangProcessor(SGLangBaseProcessor):
```
**EN:** This block introduces class `Gemma4SGLangProcessor` as a reusable abstraction inside the module. It inherits from SGLangBaseProcessor. Multimodal processor for Gemma4 supporting image, video, and audio inputs.
**CN:** 该代码块声明类 `Gemma4SGLangProcessor`，作为模块中的可复用抽象。 它继承自 SGLangBaseProcessor。 文档字符串摘要：Multimodal processor for Gemma4 supporting image, video, and audio inputs.

### Lines 31-31: Documents the scope
```python
    """Multimodal processor for Gemma4 supporting image, video, and audio inputs."""
```
**EN:** This string literal serves as documentation for the Gemma4SGLangProcessor, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 Gemma4SGLangProcessor 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 32-32: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Gemma4SGLangProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Gemma4SGLangProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 33-33: Declares models
```python
    models = [Gemma4ForConditionalGeneration]
```
**EN:** This statement initializes models in the Gemma4SGLangProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 Gemma4SGLangProcessor 中初始化 models。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 34-34: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Gemma4SGLangProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Gemma4SGLangProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 35-52: Defines function Gemma4SGLangProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)

        self.IM_START_TOKEN_ID = hf_config.boi_token_id
        self.IM_END_TOKEN_ID = hf_config.eoi_token_id

        self.AUDIO_START_TOKEN_ID = hf_config.boa_token_id
        self.AUDIO_END_TOKEN_ID = hf_config.eoa_token_id
        self.mm_tokens = MultimodalSpecialTokens(
            image_token_id=hf_config.image_token_id,
            video_token_id=hf_config.video_token_id,
            audio_token_id=hf_config.audio_token_id,
        ).build(_processor)

        # Register image-processor and video-processor outputs so they are stored on
        # MultimodalDataItem via collect_mm_items_from_processor_output.
        self.ATTR_NAME_TO_MODALITY["image_position_ids"] = Modality.IMAGE
        self.ATTR_NAME_TO_MODALITY["video_position_ids"] = Modality.VIDEO
```
**EN:** This block defines function `Gemma4SGLangProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `Gemma4SGLangProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 53-53: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Gemma4SGLangProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Gemma4SGLangProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 54-66: Defines function Gemma4SGLangProcessor._get_audio_pad_multiple
```python
    def _get_audio_pad_multiple(self) -> int:
        """Derive the waveform padding alignment from processor config.

        The HF processor's ceil(duration_ms / audio_ms_per_token) formula can
        overshoot by 1 token relative to what the SSCP convolutions produce.
        Padding waveforms to a multiple of (hop_length * first_conv_stride)
        aligns the two calculations.
        See: gemma-4-eap-extras/examples/gemma-4-audio-examples.ipynb
        """
        fe = getattr(self._processor, "feature_extractor", None)
        hop = getattr(fe, "hop_length", 160)
        first_stride = _SSCP_CONV_STRIDE_SIZES[0][0]
        return hop * first_stride
```
**EN:** This block defines function `Gemma4SGLangProcessor._get_audio_pad_multiple`. Parameters: self. Derive the waveform padding alignment from processor config. The HF processor's ceil(duration_ms / audio_ms_per_token) formula can overshoot by 1 token relative to what the SSCP convolutions produce.
**CN:** 该代码块定义函数 `Gemma4SGLangProcessor._get_audio_pad_multiple`。 参数包括 self。 文档字符串摘要：Derive the waveform padding alignment from processor config. The HF processor's ceil(duration_ms / audio_ms_per_token) formula can overshoot by 1 token relative to what the SSCP convolutions produce.

### Lines 67-67: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Gemma4SGLangProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Gemma4SGLangProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 68-88: Defines function Gemma4SGLangProcessor._video_decoder_to_tensor
```python
    def _video_decoder_to_tensor(self, vdw: VideoDecoderWrapper) -> torch.Tensor:
        """Convert a VideoDecoderWrapper to a (sampled_frames, C, H, W) uint8 tensor.

        SGLang's load_video returns VideoDecoderWrapper which the HF
        Gemma4VideoProcessor does not recognise (expects torch.Tensor or
        np.ndarray).  We replicate HF's uniform frame sampling here to
        avoid materialising the entire video in memory, then delegate the
        rest (resize, patchify, position IDs) to the HF video processor.
        """
        total = len(vdw)
        num_frames = getattr(
            getattr(self._processor, "video_processor", None),
            "num_frames",
            32,
        )
        if total <= num_frames:
            indices = list(range(total))
        else:
            indices = torch.arange(0, total, total / num_frames).int().tolist()
        frames_np = vdw.get_frames_at(indices)  # (N, H, W, C)
        return torch.from_numpy(frames_np).permute(0, 3, 1, 2).contiguous()
```
**EN:** This block defines function `Gemma4SGLangProcessor._video_decoder_to_tensor`. Parameters: self, vdw. Convert a VideoDecoderWrapper to a (sampled_frames, C, H, W) uint8 tensor. SGLang's load_video returns VideoDecoderWrapper which the HF Gemma4VideoProcessor does not recognise (expects torch.Tensor or np.ndarray).
**CN:** 该代码块定义函数 `Gemma4SGLangProcessor._video_decoder_to_tensor`。 参数包括 self、vdw。 文档字符串摘要：Convert a VideoDecoderWrapper to a (sampled_frames, C, H, W) uint8 tensor. SGLang's load_video returns VideoDecoderWrapper which the HF Gemma4VideoProcessor does not recognise (expects torch.Tensor or np.ndarray).

### Lines 89-89: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Gemma4SGLangProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Gemma4SGLangProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 90-115: Defines function Gemma4SGLangProcessor.process_mm_data
```python
    def process_mm_data(
        self, input_text, images=None, videos=None, audios=None, **kwargs
    ):
        if audios:
            pad_multiple = self._get_audio_pad_multiple()
            padded = []
            for a in audios:
                a = np.asarray(a)
                remainder = len(a) % pad_multiple
                if remainder != 0:
                    a = np.pad(a, (0, pad_multiple - remainder), mode="constant")
                padded.append(a)
            audios = padded
        if videos:
            videos = [
                (
                    self._video_decoder_to_tensor(v)
                    if isinstance(v, VideoDecoderWrapper)
                    else v
                )
                for v in videos
            ]
            kwargs.setdefault("do_sample_frames", False)
        return super().process_mm_data(
            input_text, images=images, videos=videos, audios=audios, **kwargs
        )
```
**EN:** This block defines function `Gemma4SGLangProcessor.process_mm_data`. Parameters: self, input_text, images, videos, audios.
**CN:** 该代码块定义函数 `Gemma4SGLangProcessor.process_mm_data`。 参数包括 self、input_text、images、videos、audios。

### Lines 116-116: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Gemma4SGLangProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Gemma4SGLangProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 117-145: Defines async function Gemma4SGLangProcessor.process_mm_data_async
```python
    async def process_mm_data_async(
        self,
        image_data: Optional[List[Union[str, bytes, Dict]]] = None,
        audio_data: Optional[List[Union[str, bytes, Dict]]] = None,
        input_text: str = "",
        request_obj=None,
        *args,
        **kwargs,
    ):
        """Process multimodal data including images, video, and audio."""
        base_output = self.load_mm_data(
            prompt=input_text,
            image_data=image_data,
            video_data=request_obj.video_data if request_obj else None,
            audio_data=audio_data,
            multimodal_tokens=self.mm_tokens,
        )

        mm_items, input_ids, _ = self.process_and_combine_mm_data(
            base_output, self.mm_tokens
        )

        return MultimodalProcessorOutput(
            input_ids=input_ids.tolist(),
            mm_items=mm_items,
            im_token_id=self.mm_tokens.image_token_id,
            video_token_id=self.mm_tokens.video_token_id,
            audio_token_id=self.mm_tokens.audio_token_id,
        )
```
**EN:** This block defines async function `Gemma4SGLangProcessor.process_mm_data_async`. Parameters: self, image_data, audio_data, input_text, request_obj. Process multimodal data including images, video, and audio.
**CN:** 该代码块定义异步函数 `Gemma4SGLangProcessor.process_mm_data_async`。 参数包括 self、image_data、audio_data、input_text、request_obj。 文档字符串摘要：Process multimodal data including images, video, and audio.

## Key Concepts / 关键概念
- **Classes / 类**: `Gemma4SGLangProcessor`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `typing`
- **Third-Party / 第三方**: `numpy`, `torch`
- **Local Modules / 本地模块**: `sglang.srt.managers.multimodal_processor`, `sglang.srt.managers.schedule_batch`, `sglang.srt.models.gemma4_audio`, `sglang.srt.models.gemma4_mm`, `sglang.srt.multimodal.processors.base_processor`, `sglang.srt.utils.video_decoder`
