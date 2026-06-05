# bert.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/encoders/bert.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for bert within the multimodal runtime. Key symbols include `HunyuanClip`. / 该模块实现多模态运行时中与 bert 相关的模型构件。 关键符号包括 `HunyuanClip`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
# type: ignore
import os

import torch
import torch.nn as nn
from transformers import BertModel, BertTokenizer
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 12-46: Class `HunyuanClip` / 类 `HunyuanClip`
```python
class HunyuanClip(nn.Module):
    """
    Hunyuan clip code copied from https://github.com/huggingface/diffusers/blob/main/src/diffusers/pipelines/hunyuandit/pipeline_hunyuandit.py
    hunyuan's clip used BertModel and BertTokenizer, so we copy it.
    """

    def __init__(self, model_dir, max_length=77):
        super().__init__()

        self.max_length = max_length
        self.tokenizer = BertTokenizer.from_pretrained(
            os.path.join(model_dir, "tokenizer")
        )
        self.text_encoder = BertModel.from_pretrained(
# ...
                text_inputs.attention_mask.to(self.device) if with_mask else None
            ),
        )
        return prompt_embeds.last_hidden_state, prompt_embeds.pooler_output
```
**EN:** This class models `HunyuanClip` as a specialization of `nn.Module`. Hunyuan clip code copied from https://github.com/huggingface/diffusers/blob/main/src/diffusers/pipelines/hunyuandit/pipeline_hunyuandit.py Important methods include `__init__`, `forward`.
**CN:** 该类实现 `HunyuanClip`，并继承/扩展 `nn.Module`。 文档字符串指出：Hunyuan clip code copied from https://github.com/huggingface/diffusers/blob/main/src/diffusers/pipelines/hunyuandit/pipeline_hunyuandit.py 其中较重要的方法包括 `__init__`, `forward`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Pipeline orchestration / 流水线编排
- Command-line interface / 命令行接口
- Video generation flow / 视频生成流程

## Dependencies / 依赖关系
- **External / 外部**: `torch`, `torch.nn`, `transformers`
- **Stdlib / 标准库**: `os`
