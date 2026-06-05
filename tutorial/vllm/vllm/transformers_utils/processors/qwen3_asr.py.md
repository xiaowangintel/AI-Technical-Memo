# qwen3_asr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/processors/qwen3_asr.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements model-specific processor integration for Qwen3 Asr inputs. / [CN] 为 Qwen3 Asr 输入实现模型专用的处理器集成。

## Line-by-Line Analysis / 逐行分析

### Lines 20-28: Imports
```python
import regex as re

import numpy as np

from transformers import AutoProcessor
from transformers.audio_utils import AudioInput
from transformers.feature_extraction_utils import BatchFeature
from transformers.processing_utils import ProcessingKwargs, ProcessorMixin
from transformers.tokenization_utils_base import TextInput
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as none, external APIs such as `regex`, `numpy`, `transformers`, `transformers.audio_utils`, `transformers.feature_extraction_utils`, `transformers.processing_utils`, `transformers.tokenization_utils_base`, and internal vLLM modules such as none. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 无，外部 API 如 `regex`, `numpy`, `transformers`, `transformers.audio_utils`, `transformers.feature_extraction_utils`, `transformers.processing_utils`, `transformers.tokenization_utils_base`，以及 vLLM 内部模块如 无。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 31-42: Class `Qwen3ASRProcessorKwargs`
```python
class Qwen3ASRProcessorKwargs(ProcessingKwargs, total=False):
    _defaults = {
        "text_kwargs": {
            "padding": False,
            "padding_side": "left",
        },
        "audio_kwargs": {
            "sampling_rate": 16000,
            "padding": True,
            "return_attention_mask": True,
        },
    }
```
**EN:** Defines `Qwen3ASRProcessorKwargs`, a processor-related class derived from `ProcessingKwargs`. Class attributes such as `_defaults` encode defaults or metadata.
**CN:** 定义 `Qwen3ASRProcessorKwargs`，这是一个处理器相关类，继承自 `ProcessingKwargs`。 类属性如 `_defaults` 编码了默认值或元数据。

### Lines 45-55: Function `_get_feat_extract_output_lengths`
```python
def _get_feat_extract_output_lengths(input_lengths):
    """
    Computes the output length of the convolutional layers and the output length of the audio encoder
    """

    input_lengths_leave = input_lengths % 100
    feat_lengths = (input_lengths_leave - 1) // 2 + 1
    output_lengths = (
        ((feat_lengths - 1) // 2 + 1 - 1) // 2 + 1 + (input_lengths // 100) * 13
    )
    return output_lengths
```
**EN:** This private function retrieves feat extract output lengths. The docstring states that Computes the output length of the convolutional layers and the output length of the audio encoder Main inputs include `input_lengths`.
**CN:** 该私有函数负责完成 `_get_feat_extract_output_lengths` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `input_lengths`。

### Lines 58-229: Class `Qwen3ASRProcessor`
```python
class Qwen3ASRProcessor(ProcessorMixin):
    r"""
    Constructs a Qwen3ASR processor.
    [`Qwen3ASRProcessor`] offers all the functionalities of [`WhisperFeatureExtractor`], and [`Qwen2TokenizerFast`]. See the
    [`~Qwen3ASRProcessor.__call__`] and [`~Qwen3ASRProcessor.decode`] for more information.

    Args:
        feature_extractor ([`WhisperFeatureExtractor`], *optional*):
            The audio feature extractor.
        tokenizer ([`Qwen2TokenizerFast`], *optional*):
            The text tokenizer.
        chat_template (`Optional[str]`, *optional*):
            The Jinja template to use for formatting the conversation. If not provided, the default chat template is used.
    """

    attributes = ["feature_extractor", "tokenizer"]
    feature_extractor_class = "WhisperFeatureExtractor"
    tokenizer_class = ("Qwen2Tokenizer", "Qwen2TokenizerFast")

    def __init__(self, feature_extractor=None, tokenizer=None, chat_template=None):
        super().__init__(feature_extractor, tokenizer, chat_template=chat_template)
        self.audio_token = self.tokenizer.audio_token
        self.audio_bos_token = self.tokenizer.audio_bos_token
        self.audio_eos_token = self.tokenizer.audio_eos_token

    def __call__(
        self,
        text: TextInput = None,
        audio: AudioInput = None,
        **kwargs,
    ) -> BatchFeature:
        """
        Main method to prepare for the model one or several sequences(s) and audio(s). This method forwards the `text`
        and `kwargs` arguments to Qwen2TokenizerFast's [`~Qwen2TokenizerFast.__call__`] if `text` is not `None` to encode
        the text. To prepare the audio(s), this method forwards the `audio` and `kwargs` arguments to
        WhisperFeatureExtractor's [`~WhisperFeatureExtractor.__call__`] if `audio` is not `None`. Please refer to the doctsring
        of the above two methods for more information.

        Args:
            text (`str`, `List[str]`, `List[List[str]]`):
# ... omitted for brevity ...

    @property
    def model_input_names(self):
        tokenizer_input_names = self.tokenizer.model_input_names
        feature_extractor_input_names = self.feature_extractor.model_input_names
        return list(
            dict.fromkeys(
                tokenizer_input_names
                + feature_extractor_input_names
                + ["feature_attention_mask"]
            )
        )
```
**EN:** Defines `Qwen3ASRProcessor`, a processor-related class derived from `ProcessorMixin`. The class docstring highlights that Constructs a Qwen3ASR processor. Key methods include `__init__`, `__call__`, `replace_multimodal_special_tokens`, `get_chunked_index`, `apply_chat_template`, ... (+1 more).
**CN:** 定义 `Qwen3ASRProcessor`，这是一个处理器相关类，继承自 `ProcessorMixin`。 类文档字符串进一步说明了它的职责。 关键方法包括 `__init__`, `__call__`, `replace_multimodal_special_tokens`, `get_chunked_index`, `apply_chat_template`, ... (+1 more)。

## Key Concepts / 关键概念
- **EN:** Processor wrappers unify tokenizers and modality-specific preprocessors behind a single serving-oriented interface.
- **CN:** 处理器封装把 tokenizer 与模态专用预处理器统一到面向服务的单一接口之下。
- **EN:** The module is tightly coupled to Hugging Face Transformers APIs, so version compatibility is often an explicit concern.
- **CN:** 该模块与 Hugging Face Transformers API 紧密耦合，因此版本兼容性通常是显式关注点。

## Dependencies / 依赖关系
- **EN:** Standard library modules: none.
- **CN:** 标准库模块：无。
- **EN:** External packages: `regex`, `numpy`, `transformers`, `transformers.audio_utils`, `transformers.feature_extraction_utils`, `transformers.processing_utils`, `transformers.tokenization_utils_base`.
- **CN:** 外部依赖包：`regex`, `numpy`, `transformers`, `transformers.audio_utils`, `transformers.feature_extraction_utils`, `transformers.processing_utils`, `transformers.tokenization_utils_base`。
- **EN:** Internal modules: none.
- **CN:** 内部模块：无。
