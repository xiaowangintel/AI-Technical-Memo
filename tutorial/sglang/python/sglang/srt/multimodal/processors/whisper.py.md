# whisper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/whisper.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for whisper models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 whisper 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Imports dependencies
```python
import logging
from typing import Any, Dict, Optional

from sglang.srt.entrypoints.openai.transcription_adapters.whisper import (
    FUSED_AUTODETECT_FLAG,
)
from sglang.srt.managers.schedule_batch import (
    Modality,
    MultimodalDataItem,
    MultimodalProcessorOutput,
)
from sglang.srt.models.whisper import WhisperForConditionalGeneration
from sglang.srt.multimodal.processors.base_processor import BaseMultimodalProcessor
from sglang.srt.utils import load_audio
```
**EN:** This block groups related imports for the module, including logging, typing.Any, typing.Dict, typing.Optional, sglang.srt.entrypoints.openai.transcription_adapters.whisper.FUSED_AUTODETECT_FLAG, and 6 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 logging, typing.Any, typing.Dict, typing.Optional, sglang.srt.entrypoints.openai.transcription_adapters.whisper.FUSED_AUTODETECT_FLAG 等 6 项，为后续代码准备所需名称。

### Lines 15-15: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 16-16: Declares logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This statement initializes logger in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 logger。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 17-20: Comments and module notes
```python

# ISO 639-1 supported languages for Whisper
# From https://platform.openai.com/docs/guides/speech-to-text/supported-languages
# Maps ISO 639-1 code -> Full language name
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 21-79: Declares ISO639_1_SUPPORTED_LANGS
```python
ISO639_1_SUPPORTED_LANGS = {
    "af": "Afrikaans",
    "ar": "Arabic",
    "hy": "Armenian",
    "az": "Azerbaijani",
    "be": "Belarusian",
    "bs": "Bosnian",
    "bg": "Bulgarian",
    "ca": "Catalan",
    "zh": "Chinese",
    "hr": "Croatian",
    "cs": "Czech",
    "da": "Danish",
    "nl": "Dutch",
    "en": "English",
    "et": "Estonian",
    "fi": "Finnish",
    "fr": "French",
    "gl": "Galician",
    "de": "German",
    "el": "Greek",
    "he": "Hebrew",
    "hi": "Hindi",
    "hu": "Hungarian",
    "is": "Icelandic",
    "id": "Indonesian",
    "it": "Italian",
    "ja": "Japanese",
    "kn": "Kannada",
    "kk": "Kazakh",
    "ko": "Korean",
    "lv": "Latvian",
    "lt": "Lithuanian",
    "mk": "Macedonian",
    "ms": "Malay",
    "mr": "Marathi",
    "mi": "Maori",
    "ne": "Nepali",
    "no": "Norwegian",
    "fa": "Persian",
    "pl": "Polish",
    "pt": "Portuguese",
    "ro": "Romanian",
    "ru": "Russian",
    "sr": "Serbian",
    "sk": "Slovak",
    "sl": "Slovenian",
    "es": "Spanish",
    "sw": "Swahili",
    "sv": "Swedish",
    "tl": "Tagalog",
    "ta": "Tamil",
    "th": "Thai",
    "tr": "Turkish",
    "uk": "Ukrainian",
    "ur": "Urdu",
    "vi": "Vietnamese",
    "cy": "Welsh",
}
```
**EN:** This statement initializes ISO639_1_SUPPORTED_LANGS in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 ISO639_1_SUPPORTED_LANGS。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 80-81: Comments and module notes
```python

# Reverse mapping: Full language name (lowercase) -> ISO 639-1 code
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 82-84: Declares LANG_NAME_TO_CODE
```python
LANG_NAME_TO_CODE = {
    name.lower(): code for code, name in ISO639_1_SUPPORTED_LANGS.items()
}
```
**EN:** This statement initializes LANG_NAME_TO_CODE in the module. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 模块 中初始化 LANG_NAME_TO_CODE。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 85-86: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 87-130: Defines function normalize_language_to_code
```python
def normalize_language_to_code(language: Optional[str]) -> Optional[str]:
    """Convert a language input (full name or code) to ISO 639-1 code.

    Args:
        language: Language as full name (e.g., 'English', 'Spanish') or
                  ISO 639-1 code (e.g., 'en', 'es'). Three-letter Whisper
                  codes the model supports but that aren't in
                  ISO639_1_SUPPORTED_LANGS (e.g., 'yue', 'haw', 'jw') are
                  also accepted so that a code returned by fused autodetect
                  round-trips cleanly when reused as ``language=`` later.

    Returns:
        Whisper language code or None if input is None
    """
    if language is None:
        return None

    language_lower = language.lower().strip()

    # Check if it's already a valid ISO code
    if language_lower in ISO639_1_SUPPORTED_LANGS:
        return language_lower

    # Check if it's a full language name
    if language_lower in LANG_NAME_TO_CODE:
        return LANG_NAME_TO_CODE[language_lower]

    # Fused autodetect's FSM regex covers the full Whisper language-token
    # vocab (see WHISPER_LANG_TOKEN_CODES), which is wider than the
    # English-name-keyed ISO639_1_SUPPORTED_LANGS dict. Accept any code in
    # that wider set too so that detection -> reuse-as-input round-trips.
    # Lazy import to avoid top-level cycle with the openai entrypoint.
    from sglang.srt.entrypoints.openai.transcription_adapters.whisper import (
        WHISPER_LANG_TOKEN_CODES,
    )

    if language_lower in WHISPER_LANG_TOKEN_CODES:
        return language_lower

    # Not recognized
    raise ValueError(
        f"Language '{language}' not recognized. "
        f"Use full name (e.g., 'English') or ISO 639-1 code (e.g., 'en')."
    )
```
**EN:** This block defines function `normalize_language_to_code`. Parameters: language. Convert a language input (full name or code) to ISO 639-1 code. Args: language: Language as full name (e.g., 'English', 'Spanish') or ISO 639-1 code (e.g., 'en', 'es').
**CN:** 该代码块定义函数 `normalize_language_to_code`。 参数包括 language。 文档字符串摘要：Convert a language input (full name or code) to ISO 639-1 code. Args: language: Language as full name (e.g., 'English', 'Spanish') or ISO 639-1 code (e.g., 'en', 'es').

### Lines 131-132: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 133-133: Declares class WhisperProcessor
```python
class WhisperProcessor(BaseMultimodalProcessor):
```
**EN:** This block introduces class `WhisperProcessor` as a reusable abstraction inside the module. It inherits from BaseMultimodalProcessor.
**CN:** 该代码块声明类 `WhisperProcessor`，作为模块中的可复用抽象。 它继承自 BaseMultimodalProcessor。

### Lines 134-134: Declares models
```python
    models = [WhisperForConditionalGeneration]
```
**EN:** This statement initializes models in the WhisperProcessor. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 WhisperProcessor 中初始化 models。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 135-135: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the WhisperProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 WhisperProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 136-139: Defines function WhisperProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)
        # Cache tokenizer for language token lookup
        self._tokenizer = getattr(self._processor, "tokenizer", None)
```
**EN:** This block defines function `WhisperProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `WhisperProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 140-140: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the WhisperProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 WhisperProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 141-143: Defines function WhisperProcessor._pop_sampling_param
```python
    def _pop_sampling_param(self, request_obj, key: str):
        sampling_params = getattr(request_obj, "sampling_params", None) or {}
        return sampling_params.pop(key, None)
```
**EN:** This block defines function `WhisperProcessor._pop_sampling_param`. Parameters: self, request_obj, key.
**CN:** 该代码块定义函数 `WhisperProcessor._pop_sampling_param`。 参数包括 self、request_obj、key。

### Lines 144-144: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the WhisperProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 WhisperProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 145-164: Defines function WhisperProcessor._get_language_token_id
```python
    def _get_language_token_id(self, language: Optional[str]) -> int:
        # Default to English if not specified
        if language is None:
            language = "en"  # Default to English
        language_token = f"<|{language}|>"
        token_id = self._tokenizer.convert_tokens_to_ids(language_token)
        # normalize_language_to_code accepts the full Whisper language-token
        # vocab (including yue/haw/jw) so fused autodetect output round-trips.
        # Older checkpoints (v1/v2) don't have every newer token in their
        # vocab, in which case convert_tokens_to_ids returns the unk id.
        # Raise a clean error here instead of silently feeding unk into the
        # decoder and producing garbage.
        unk_id = getattr(self._tokenizer, "unk_token_id", None)
        if token_id is None or (unk_id is not None and token_id == unk_id):
            raise ValueError(
                f"Language '{language}' is not in this Whisper model's vocabulary. "
                f"The '{language_token}' token may have been added in a later "
                f"Whisper version than the loaded checkpoint."
            )
        return token_id
```
**EN:** This block defines function `WhisperProcessor._get_language_token_id`. Parameters: self, language.
**CN:** 该代码块定义函数 `WhisperProcessor._get_language_token_id`。 参数包括 self、language。

### Lines 165-165: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the WhisperProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 WhisperProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 166-238: Defines async function WhisperProcessor.process_mm_data_async (part 1)
```python
    async def process_mm_data_async(
        self,
        image_data,
        audio_data,
        input_text,
        request_obj,
        **kwargs,
    ) -> Optional[Dict[str, Any]]:
        if not audio_data:
            return None

        if len(audio_data) != 1:
            raise ValueError(
                f"Whisper expects exactly 1 audio input, got {len(audio_data)}"
            )

        # Check if this is a fused auto-detect request (decoder prompt = [SOT] only,
        # structured generation handles the rest via regex constraint).
        detect_language = self._pop_sampling_param(request_obj, FUSED_AUTODETECT_FLAG)
        # timestamp_granularities is a transcription-level field; it must be
        # popped in both branches or it leaks into SamplingParams(**kwargs)
        # downstream and TypeErrors. In the fused branch the FSM regex was
        # already picked in build_fused_autodetect_params based on this value,
        # so we only need to keep it here to pick the timestamp_token_id for
        # the explicit-language branch.
        timestamp_granularities = self._pop_sampling_param(
            request_obj, "timestamp_granularities"
        )

        audios = [load_audio(audio) for audio in audio_data]

        # Whisper expects input features padded to max_length (3000 frames = 30 seconds)
        # This is the standard context length for Whisper
        input_features = self._processor.feature_extractor(
            audios[0],
            sampling_rate=16000,
            padding="max_length",  # Pad to 3000 frames
            return_tensors="pt",
        )["input_features"][0]

        # Whisper is a pure speech-to-text model; text prompts are ignored.
        # The full decoder sequence is:
        #   <|startoftranscript|> <|lang|> <|transcribe|> [<|notimestamps|> | <|0.00|>]
        #
        # When language is known, we build this prefix explicitly below.
        # When auto-detecting (_detect_language=True), we feed only <|startoftranscript|>
        # and let SGLang's structured generation (regex) constrain the model to produce
        # <|lang|><|transcribe|><|notimestamps|> as the first 3 decode tokens — this is
        # equivalent to HuggingFace's forced_decoder_ids but uses SGLang's native API.

        decoder_start_token_id = getattr(
            self.hf_config, "decoder_start_token_id", 50258
        )

        if detect_language:
            input_ids = [decoder_start_token_id]
        else:
            language = normalize_language_to_code(
                self._pop_sampling_param(request_obj, "language")
            )
            language_token_id = self._get_language_token_id(language)

            transcribe_token_id = self._tokenizer.convert_tokens_to_ids(
                "<|transcribe|>"
            )

            # Use <|0.00|> to enable timestamp generation, or <|notimestamps|> to disable
            if timestamp_granularities:
                timestamp_token_id = self._tokenizer.convert_tokens_to_ids("<|0.00|>")
            else:
                timestamp_token_id = self._tokenizer.convert_tokens_to_ids(
                    "<|notimestamps|>"
                )
```
**EN:** This block defines async function `WhisperProcessor.process_mm_data_async`. Parameters: self, image_data, audio_data, input_text, request_obj. This subsection covers lines 166-238 of the same logical block.
**CN:** 该代码块定义异步函数 `WhisperProcessor.process_mm_data_async`。 参数包括 self、image_data、audio_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 166-238 行。

### Lines 239-239: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 240-245: Defines async function WhisperProcessor.process_mm_data_async (part 2)
```python
            input_ids = [
                decoder_start_token_id,
                language_token_id,
                transcribe_token_id,
                timestamp_token_id,
            ]
```
**EN:** This block defines async function `WhisperProcessor.process_mm_data_async`. Parameters: self, image_data, audio_data, input_text, request_obj. This subsection covers lines 240-245 of the same logical block.
**CN:** 该代码块定义异步函数 `WhisperProcessor.process_mm_data_async`。 参数包括 self、image_data、audio_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 240-245 行。

### Lines 246-246: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 247-255: Defines async function WhisperProcessor.process_mm_data_async (part 3)
```python
        return MultimodalProcessorOutput(
            input_ids=input_ids,
            mm_items=[
                MultimodalDataItem(
                    feature=input_features,
                    modality=Modality.AUDIO,
                )
            ],
        )
```
**EN:** This block defines async function `WhisperProcessor.process_mm_data_async`. Parameters: self, image_data, audio_data, input_text, request_obj. This subsection covers lines 247-255 of the same logical block.
**CN:** 该代码块定义异步函数 `WhisperProcessor.process_mm_data_async`。 参数包括 self、image_data、audio_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 247-255 行。

## Key Concepts / 关键概念
- **Classes / 类**: `WhisperProcessor`
- **Functions / 函数**: `normalize_language_to_code`
- **Constants / 常量**: `ISO639_1_SUPPORTED_LANGS`, `LANG_NAME_TO_CODE`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `logging`, `typing`
- **Local Modules / 本地模块**: `sglang.srt.entrypoints.openai.transcription_adapters.whisper`, `sglang.srt.managers.schedule_batch`, `sglang.srt.models.whisper`, `sglang.srt.multimodal.processors.base_processor`, `sglang.srt.utils`
