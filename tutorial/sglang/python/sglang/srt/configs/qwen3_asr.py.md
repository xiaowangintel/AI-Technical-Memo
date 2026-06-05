# qwen3_asr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/qwen3_asr.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for qwen3 asr so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 qwen3 asr 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Imports dependencies
```python
import torch
from transformers import (
    AutoConfig,
    AutoFeatureExtractor,
    AutoTokenizer,
    PretrainedConfig,
    ProcessorMixin,
)

from sglang.srt.configs.qwen3_omni import Qwen3OmniMoeAudioEncoderConfig
from sglang.srt.multimodal.customized_mm_processor_utils import (
    register_customized_processor,
)
from sglang.utils import logger
```
**EN:** This block groups related imports for the module, including torch, transformers.AutoConfig, transformers.AutoFeatureExtractor, transformers.AutoTokenizer, transformers.PretrainedConfig, and 4 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 torch, transformers.AutoConfig, transformers.AutoFeatureExtractor, transformers.AutoTokenizer, transformers.PretrainedConfig 等 4 项，为后续代码准备所需名称。

### Lines 15-16: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 17-17: Declares class Qwen3ASRProcessor
```python
class Qwen3ASRProcessor(ProcessorMixin):
```
**EN:** This block introduces class `Qwen3ASRProcessor` as a reusable abstraction inside the module. It inherits from ProcessorMixin. Minimal composite processor: WhisperFeatureExtractor + Qwen2Tokenizer. AutoProcessor.from_pretrained() for Qwen3-ASR returns just a tokenizer, but SGLang's multimodal pipeline needs a processor that handles audio.
**CN:** 该代码块声明类 `Qwen3ASRProcessor`，作为模块中的可复用抽象。 它继承自 ProcessorMixin。 文档字符串摘要：Minimal composite processor: WhisperFeatureExtractor + Qwen2Tokenizer. AutoProcessor.from_pretrained() for Qwen3-ASR returns just a tokenizer, but SGLang's multimodal pipeline needs a processor that handles audio.

### Lines 18-22: Documents the scope
```python
    """Minimal composite processor: WhisperFeatureExtractor + Qwen2Tokenizer.

    AutoProcessor.from_pretrained() for Qwen3-ASR returns just a tokenizer,
    but SGLang's multimodal pipeline needs a processor that handles audio.
    """
```
**EN:** This string literal serves as documentation for the Qwen3ASRProcessor, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 Qwen3ASRProcessor 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 23-23: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3ASRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3ASRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 24-26: Declares attributes, feature_extractor_class, tokenizer_class
```python
    attributes = ["feature_extractor", "tokenizer"]
    feature_extractor_class = "WhisperFeatureExtractor"
    tokenizer_class = "AutoTokenizer"
```
**EN:** This block initializes a related set of values in the Qwen3ASRProcessor, including attributes, feature_extractor_class, tokenizer_class. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3ASRProcessor 中初始化一组相关值，包括 attributes, feature_extractor_class, tokenizer_class。将这些赋值集中在一起有助于理解周边配置。

### Lines 27-27: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3ASRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3ASRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 28-29: Defines function Qwen3ASRProcessor.__init__
```python
    def __init__(self, feature_extractor=None, tokenizer=None, **kwargs):
        super().__init__(feature_extractor=feature_extractor, tokenizer=tokenizer)
```
**EN:** This block defines function `Qwen3ASRProcessor.__init__`. Parameters: self, feature_extractor, tokenizer.
**CN:** 该代码块定义函数 `Qwen3ASRProcessor.__init__`。 参数包括 self、feature_extractor、tokenizer。

### Lines 30-30: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3ASRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3ASRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 31-42: Defines function Qwen3ASRProcessor.from_pretrained
```python
    @classmethod
    def from_pretrained(cls, pretrained_model_name_or_path, **kwargs):
        trust_remote_code = kwargs.pop("trust_remote_code", True)
        feature_extractor = AutoFeatureExtractor.from_pretrained(
            pretrained_model_name_or_path,
            trust_remote_code=trust_remote_code,
        )
        tokenizer = AutoTokenizer.from_pretrained(
            pretrained_model_name_or_path,
            trust_remote_code=trust_remote_code,
        )
        return cls(feature_extractor=feature_extractor, tokenizer=tokenizer)
```
**EN:** This block defines function `Qwen3ASRProcessor.from_pretrained`. Parameters: cls, pretrained_model_name_or_path. Decorators: classmethod.
**CN:** 该代码块定义函数 `Qwen3ASRProcessor.from_pretrained`。 参数包括 cls、pretrained_model_name_or_path。 装饰器包括 classmethod。

### Lines 43-43: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3ASRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3ASRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 44-49: Defines function Qwen3ASRProcessor._get_feat_extract_output_lengths
```python
    def _get_feat_extract_output_lengths(self, input_lengths):
        if not isinstance(input_lengths, torch.Tensor):
            input_lengths = torch.tensor(input_lengths)
        input_lengths_leave = input_lengths % 100
        feat_lengths = (input_lengths_leave - 1) // 2 + 1
        return ((feat_lengths - 1) // 2 + 1 - 1) // 2 + 1 + (input_lengths // 100) * 13
```
**EN:** This block defines function `Qwen3ASRProcessor._get_feat_extract_output_lengths`. Parameters: self, input_lengths.
**CN:** 该代码块定义函数 `Qwen3ASRProcessor._get_feat_extract_output_lengths`。 参数包括 self、input_lengths。

### Lines 50-50: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3ASRProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3ASRProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 51-101: Defines function Qwen3ASRProcessor.__call__
```python
    def __call__(self, text=None, audio=None, audio_kwargs=None, **kwargs):
        inputs = {}
        if audio is not None:
            audio_kwargs = audio_kwargs or {}
            audio_inputs = self.feature_extractor(
                audio,
                sampling_rate=self.feature_extractor.sampling_rate,
                return_attention_mask=True,
                return_tensors=kwargs.get("return_tensors"),
                **audio_kwargs,
            )
            inputs["input_features"] = audio_inputs["input_features"]
            if "attention_mask" in audio_inputs:
                inputs["feature_attention_mask"] = audio_inputs["attention_mask"]

        if text is not None:
            text_inputs = self.tokenizer(
                text,
                return_tensors=kwargs.get("return_tensors"),
                padding=kwargs.get("padding", False),
            )
            input_ids = text_inputs["input_ids"]

            # Expand the single <|audio_pad|> placeholder in the prompt to N
            # copies, where N is the audio encoder's output length for this clip.
            # Without this, the model only sees 1 audio token for hundreds of
            # feature frames and can't align audio embeddings with token positions.
            if audio is not None and "feature_attention_mask" in inputs:
                audio_pad_id = self.tokenizer.convert_tokens_to_ids("<|audio_pad|>")
                feat_lengths = inputs["feature_attention_mask"].sum(dim=-1)
                audio_token_counts = self._get_feat_extract_output_lengths(feat_lengths)
                expanded = []
                for seq_idx in range(input_ids.shape[0]):
                    ids = input_ids[seq_idx].tolist()
                    audio_idx = 0
                    new_ids = []
                    for tid in ids:
                        if tid == audio_pad_id and audio_idx < len(audio_token_counts):
                            n = int(audio_token_counts[audio_idx].item())
                            new_ids.extend([audio_pad_id] * n)
                            audio_idx += 1
                        else:
                            new_ids.append(tid)
                    expanded.append(new_ids)
                max_len = max(len(s) for s in expanded)
                pad_id = self.tokenizer.pad_token_id or 0
                padded = [s + [pad_id] * (max_len - len(s)) for s in expanded]
                input_ids = torch.tensor(padded, dtype=torch.long)

            inputs["input_ids"] = input_ids
        return inputs
```
**EN:** This block defines function `Qwen3ASRProcessor.__call__`. Parameters: self, text, audio, audio_kwargs.
**CN:** 该代码块定义函数 `Qwen3ASRProcessor.__call__`。 参数包括 self、text、audio、audio_kwargs。

### Lines 102-103: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 104-104: Declares class Qwen3ASRThinkerConfig
```python
class Qwen3ASRThinkerConfig(PretrainedConfig):
```
**EN:** This block introduces class `Qwen3ASRThinkerConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `Qwen3ASRThinkerConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 105-108: Declares model_type, sub_configs
```python
    model_type = "qwen3_asr_thinker"
    sub_configs = {
        "audio_config": Qwen3OmniMoeAudioEncoderConfig,
    }
```
**EN:** This block initializes a related set of values in the Qwen3ASRThinkerConfig, including model_type, sub_configs. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3ASRThinkerConfig 中初始化一组相关值，包括 model_type, sub_configs。将这些赋值集中在一起有助于理解周边配置。

### Lines 109-109: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3ASRThinkerConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3ASRThinkerConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 110-140: Defines function Qwen3ASRThinkerConfig.__init__
```python
    def __init__(
        self,
        audio_config=None,
        text_config=None,
        audio_token_id=151676,
        audio_start_token_id=151669,
        audio_end_token_id=151670,
        **kwargs,
    ):
        super().__init__(**kwargs)

        if isinstance(audio_config, dict):
            audio_config = Qwen3OmniMoeAudioEncoderConfig(**audio_config)
        elif audio_config is None:
            audio_config = Qwen3OmniMoeAudioEncoderConfig()
        self.audio_config = audio_config

        from transformers.models.qwen3.configuration_qwen3 import (
            Qwen3Config as HFQwen3Config,
        )

        if isinstance(text_config, dict):
            text_config = HFQwen3Config(**text_config)
        elif text_config is None:
            text_config = HFQwen3Config()

        self.text_config = text_config

        self.audio_token_id = audio_token_id
        self.audio_start_token_id = audio_start_token_id
        self.audio_end_token_id = audio_end_token_id
```
**EN:** This block defines function `Qwen3ASRThinkerConfig.__init__`. Parameters: self, audio_config, text_config, audio_token_id, audio_start_token_id, audio_end_token_id.
**CN:** 该代码块定义函数 `Qwen3ASRThinkerConfig.__init__`。 参数包括 self、audio_config、text_config、audio_token_id、audio_start_token_id、audio_end_token_id。

### Lines 141-142: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 143-144: Declares class Qwen3ASRConfig
```python
@register_customized_processor(Qwen3ASRProcessor)
class Qwen3ASRConfig(PretrainedConfig):
```
**EN:** This block introduces class `Qwen3ASRConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `Qwen3ASRConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 145-148: Declares model_type, sub_configs
```python
    model_type = "qwen3_asr"
    sub_configs = {
        "thinker_config": Qwen3ASRThinkerConfig,
    }
```
**EN:** This block initializes a related set of values in the Qwen3ASRConfig, including model_type, sub_configs. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3ASRConfig 中初始化一组相关值，包括 model_type, sub_configs。将这些赋值集中在一起有助于理解周边配置。

### Lines 149-149: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3ASRConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3ASRConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 150-161: Defines function Qwen3ASRConfig.__init__
```python
    def __init__(self, thinker_config=None, **kwargs):
        if thinker_config is None:
            thinker_config = {}
            logger.info(
                "thinker_config is None. "
                "Initializing Qwen3-ASR thinker with default values"
            )
        if isinstance(thinker_config, dict):
            self.thinker_config = Qwen3ASRThinkerConfig(**thinker_config)
        else:
            self.thinker_config = thinker_config
        super().__init__(**kwargs)
```
**EN:** This block defines function `Qwen3ASRConfig.__init__`. Parameters: self, thinker_config.
**CN:** 该代码块定义函数 `Qwen3ASRConfig.__init__`。 参数包括 self、thinker_config。

### Lines 162-162: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3ASRConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3ASRConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 163-164: Defines function Qwen3ASRConfig.get_text_config
```python
    def get_text_config(self, decoder=False) -> PretrainedConfig:
        return self.thinker_config.text_config
```
**EN:** This block defines function `Qwen3ASRConfig.get_text_config`. Parameters: self, decoder.
**CN:** 该代码块定义函数 `Qwen3ASRConfig.get_text_config`。 参数包括 self、decoder。

### Lines 165-166: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 167-167: Implements Expr
```python
AutoConfig.register("qwen3_asr", Qwen3ASRConfig)
```
**EN:** This block contributes general-purpose logic to the module using a `Expr` statement.
**CN:** 该代码块通过 `Expr` 语句为 模块 补充通用逻辑。

### Lines 168-168: Implements Expr
```python
AutoConfig.register("qwen3_asr_thinker", Qwen3ASRThinkerConfig)
```
**EN:** This block contributes general-purpose logic to the module using a `Expr` statement.
**CN:** 该代码块通过 `Expr` 语句为 模块 补充通用逻辑。

## Key Concepts / 关键概念
- **Classes / 类**: `Qwen3ASRProcessor`, `Qwen3ASRThinkerConfig`, `Qwen3ASRConfig`

## Dependencies / 依赖关系
- **Third-Party / 第三方**: `torch`, `transformers`, `transformers.models.qwen3.configuration_qwen3`
- **Local Modules / 本地模块**: `sglang.srt.configs.qwen3_omni`, `sglang.srt.multimodal.customized_mm_processor_utils`, `sglang.utils`
