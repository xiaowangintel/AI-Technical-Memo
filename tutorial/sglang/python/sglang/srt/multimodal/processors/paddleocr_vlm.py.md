# paddleocr_vlm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/paddleocr_vlm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for paddleocr vlm models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 paddleocr vlm 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Comments and module notes
```python
# Reference: ccr-2vdh3abv-pub.cnc.bj.baidubce.com/paddlepaddle/paddleocr-genai-vllm-server:latest
# Copyright (c) 2025 PaddlePaddle Authors. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 15-17: Imports dependencies
```python
from sglang.srt.models.paddleocr_vl import PaddleOCRVLForConditionalGeneration
from sglang.srt.multimodal.processors.base_processor import MultimodalSpecialTokens
from sglang.srt.multimodal.processors.qwen_vl import QwenVLImageProcessor
```
**EN:** This block groups related imports for the module, including sglang.srt.models.paddleocr_vl.PaddleOCRVLForConditionalGeneration, sglang.srt.multimodal.processors.base_processor.MultimodalSpecialTokens, sglang.srt.multimodal.processors.qwen_vl.QwenVLImageProcessor. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 sglang.srt.models.paddleocr_vl.PaddleOCRVLForConditionalGeneration, sglang.srt.multimodal.processors.base_processor.MultimodalSpecialTokens, sglang.srt.multimodal.processors.qwen_vl.QwenVLImageProcessor，为后续代码准备所需名称。

### Lines 18-19: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 20-20: Declares class PaddleOCRVLImageProcessor
```python
class PaddleOCRVLImageProcessor(QwenVLImageProcessor):
```
**EN:** This block introduces class `PaddleOCRVLImageProcessor` as a reusable abstraction inside the module. It inherits from QwenVLImageProcessor.
**CN:** 该代码块声明类 `PaddleOCRVLImageProcessor`，作为模块中的可复用抽象。 它继承自 QwenVLImageProcessor。

### Lines 21-21: Declares models
```python
    models = [PaddleOCRVLForConditionalGeneration]
```
**EN:** This statement initializes models in the PaddleOCRVLImageProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 PaddleOCRVLImageProcessor 中初始化 models。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 22-22: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the PaddleOCRVLImageProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 PaddleOCRVLImageProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 23-30: Defines function PaddleOCRVLImageProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)

        self.mm_tokens = MultimodalSpecialTokens(
            image_token="<|IMAGE_START|><|IMAGE_PLACEHOLDER|><|IMAGE_END|>",
            image_token_id=hf_config.image_token_id,
            video_token_id=hf_config.video_token_id,
        ).build(_processor)
```
**EN:** This block defines function `PaddleOCRVLImageProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `PaddleOCRVLImageProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

## Key Concepts / 关键概念
- **Classes / 类**: `PaddleOCRVLImageProcessor`

## Dependencies / 依赖关系
- **Local Modules / 本地模块**: `sglang.srt.models.paddleocr_vl`, `sglang.srt.multimodal.processors.base_processor`, `sglang.srt.multimodal.processors.qwen_vl`
