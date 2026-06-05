# preprocess.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/inputs/preprocess.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements preprocess support for the `inputs` portion of vLLM. / 为 vLLM 的 `inputs` 子目录实现与 preprocess 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-43)
```python
from collections.abc import Mapping

from typing import Any, overload

from typing_extensions import assert_never

from vllm.config import VllmConfig

from vllm.inputs import build_enc_dec_input

from vllm.logger import init_logger

from vllm.multimodal import MULTIMODAL_REGISTRY, MultiModalRegistry

from vllm.renderers import BaseRenderer, renderer_from_config

from vllm.renderers.inputs import (
    DecoderDictPrompt,
    DecoderOnlyDictPrompt,
    EncoderDecoderDictPrompt,
    EncoderDictPrompt,
    SingletonDictPrompt,
)

from vllm.renderers.inputs.preprocess import parse_dec_only_prompt, parse_enc_dec_prompt

from vllm.tokenizers import TokenizerLike

from .engine import (
    DecoderEngineInput,
    DecoderOnlyEngineInput,
    EmbedsInput,
    EncoderDecoderInput,
    EncoderInput,
    EngineInput,
    MultiModalInput,
# ... omitted for brevity ...
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 45-45)
```python
logger = init_logger(__name__)
```
**EN:** This constant/configuration block defines `logger`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `logger`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Class `InputPreprocessor` (lines 48-291)
```python
class InputPreprocessor:
    def __init__(
        self,
        vllm_config: VllmConfig,
        renderer: BaseRenderer | None = None,
        mm_registry: MultiModalRegistry = MULTIMODAL_REGISTRY,
    ) -> None:
        super().__init__()

        self.model_config = vllm_config.model_config
        self.renderer = renderer or renderer_from_config(vllm_config)
        self.mm_registry = mm_registry

    @property
    def tokenizer(self) -> TokenizerLike | None:
        return self.renderer.tokenizer

    def get_tokenizer(self) -> TokenizerLike:
        return self.renderer.get_tokenizer()

    def _tokenize_prompt(
        self,
        prompt: str,
    # ... omitted for brevity ...
            tokenization_kwargs=tokenization_kwargs,
        )
```
**EN:** Class `InputPreprocessor` is a structured building block in this module. Key methods include `__init__`, `tokenizer`, `get_tokenizer`, `_tokenize_prompt`, `_process_multimodal`, `_process_embeds`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `InputPreprocessor` 是该模块中的结构化构件。 关键方法包括 `__init__`, `tokenizer`, `get_tokenizer`, `_tokenize_prompt`, `_process_multimodal`, `_process_embeds`，它们共同定义初始化、校验、变换或访问模式。

### Method `InputPreprocessor.__init__` (lines 49-59)
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        renderer: BaseRenderer | None = None,
        mm_registry: MultiModalRegistry = MULTIMODAL_REGISTRY,
    ) -> None:
        super().__init__()

        self.model_config = vllm_config.model_config
        self.renderer = renderer or renderer_from_config(vllm_config)
        self.mm_registry = mm_registry
```
**EN:** Method `InputPreprocessor.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `super().__init__`, `super`, `renderer_from_config` show the concrete execution path.
**CN:** Method `InputPreprocessor.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `super().__init__`, `super`, `renderer_from_config` 这样的关键调用展示了该代码块的具体执行路径。

### Method `InputPreprocessor.tokenizer` (lines 62-63)
```python
    def tokenizer(self) -> TokenizerLike | None:
        return self.renderer.tokenizer
```
**EN:** Method `InputPreprocessor.tokenizer` exposes a computed property so callers can access derived state without duplicating logic.
**CN:** Method `InputPreprocessor.tokenizer` 暴露计算属性，使调用方无需重复逻辑即可访问派生状态。

### Method `InputPreprocessor.get_tokenizer` (lines 65-66)
```python
    def get_tokenizer(self) -> TokenizerLike:
        return self.renderer.get_tokenizer()
```
**EN:** Method `InputPreprocessor.get_tokenizer` implements an encoding/decoding or token-transformation step. Key calls such as `self.renderer.get_tokenizer` show the concrete execution path.
**CN:** Method `InputPreprocessor.get_tokenizer` 实现编码/解码或 Token 变换步骤。 像 `self.renderer.get_tokenizer` 这样的关键调用展示了该代码块的具体执行路径。

### Method `InputPreprocessor._process_text` (lines 161-188)
```python
    def _process_text(
        self,
        parsed_content: TextPrompt,
        tokenization_kwargs: dict[str, Any] | None = None,
    ) -> TokensInput | MultiModalInput:
        prompt_text = parsed_content["prompt"]

        inputs: TokensInput | MultiModalInput
        if multi_modal_data := parsed_content.get("multi_modal_data"):
            inputs = self._process_multimodal(
                prompt_text,
                multi_modal_data,
                parsed_content.get("mm_processor_kwargs") or {},
                tokenization_kwargs=tokenization_kwargs,
            )
        else:
            prompt_token_ids = self._tokenize_prompt(
                prompt_text,
                tokenization_kwargs=tokenization_kwargs,
    # ... omitted for brevity ...

        return inputs
```
**EN:** Method `InputPreprocessor._process_text` provides a reusable helper around the module's main workflow. Key calls such as `parsed_content.get`, `self._process_multimodal`, `self._tokenize_prompt`, `tokens_input` show the concrete execution path.
**CN:** Method `InputPreprocessor._process_text` 为模块主流程提供可复用的辅助逻辑。 像 `parsed_content.get`, `self._process_multimodal`, `self._tokenize_prompt`, `tokens_input` 这样的关键调用展示了该代码块的具体执行路径。

### Method `InputPreprocessor._process_encoder_decoder_prompt` (lines 230-262)
```python
    def _process_encoder_decoder_prompt(
        self,
        prompt: EncoderDecoderDictPrompt,
        tokenization_kwargs: dict[str, Any] | None = None,
    ) -> EncoderDecoderInput:
        encoder_prompt = prompt["encoder_prompt"]
        decoder_prompt = prompt["decoder_prompt"]

        skip_decoder_start_token = False
        if self.renderer.mm_processor is not None:
            from vllm.multimodal.processing import EncDecMultiModalProcessor

            if isinstance(self.renderer.mm_processor, EncDecMultiModalProcessor):
                skip_decoder_start_token = (
                    self.renderer.mm_processor.skip_decoder_start_token
                )

        return build_enc_dec_input(
            encoder_input=self._prompt_to_llm_inputs(
    # ... omitted for brevity ...
            skip_decoder_start_token=skip_decoder_start_token,
        )
```
**EN:** Method `InputPreprocessor._process_encoder_decoder_prompt` implements an encoding/decoding or token-transformation step. Key calls such as `isinstance`, `build_enc_dec_input`, `self._prompt_to_llm_inputs`, `self.renderer.get_dec_start_token_id` show the concrete execution path.
**CN:** Method `InputPreprocessor._process_encoder_decoder_prompt` 实现编码/解码或 Token 变换步骤。 像 `isinstance`, `build_enc_dec_input`, `self._prompt_to_llm_inputs`, `self.renderer.get_dec_start_token_id` 这样的关键调用展示了该代码块的具体执行路径。

### Method `InputPreprocessor.preprocess` (lines 274-291)
```python
    def preprocess(
        self,
        prompt: PromptType,
        tokenization_kwargs: dict[str, Any] | None = None,
    ) -> EngineInput:
        """Preprocess the input prompt."""
        if self.model_config.is_encoder_decoder:
            # Encoder-decoder model requires special mapping of
            # input prompts to encoder & decoder.
            return self._process_encoder_decoder_prompt(
                parse_enc_dec_prompt(prompt),
                tokenization_kwargs,
            )

        return self._process_decoder_only_prompt(
            parse_dec_only_prompt(prompt),
            tokenization_kwargs=tokenization_kwargs,
        )
```
**EN:** Method `InputPreprocessor.preprocess` provides a reusable helper around the module's main workflow. The docstring highlights: Preprocess the input prompt. Key calls such as `self._process_encoder_decoder_prompt`, `parse_enc_dec_prompt`, `self._process_decoder_only_prompt`, `parse_dec_only_prompt` show the concrete execution path.
**CN:** Method `InputPreprocessor.preprocess` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Preprocess the input prompt. 像 `self._process_encoder_decoder_prompt`, `parse_enc_dec_prompt`, `self._process_decoder_only_prompt`, `parse_dec_only_prompt` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Multimodality / 多模态**
  - **EN:** The code contains modality-specific paths for text, image, audio, or video data.
  - **CN:** 代码包含面向文本、图像、音频或视频数据的模态专用路径。
- **Caching / 缓存**
  - **EN:** Some definitions are designed to reuse computed state and avoid repeated work.
  - **CN:** 部分定义旨在复用已计算状态，避免重复工作。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from collections.abc import Mapping`, `from typing import Any, overload`
- **Third-party / 第三方**: `from typing_extensions import assert_never`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.inputs import build_enc_dec_input`, `from vllm.logger import init_logger`, `from vllm.multimodal import MULTIMODAL_REGISTRY, MultiModalRegistry`, `from vllm.renderers import BaseRenderer, renderer_from_config`, `from vllm.renderers.inputs import DecoderDictPrompt, DecoderOnlyDictPrompt, EncoderDecoderDictPrompt, EncoderDictPrompt, SingletonDictPrompt`, `from vllm.renderers.inputs.preprocess import parse_dec_only_prompt, parse_enc_dec_prompt`, `from vllm.tokenizers import TokenizerLike`, `from .engine import DecoderEngineInput, DecoderOnlyEngineInput, EmbedsInput, EncoderDecoderInput, EncoderInput, EngineInput, MultiModalInput, SingletonInput, TokensInput, tokens_input`, `from .llm import EmbedsPrompt, MultiModalDataDict, MultiModalUUIDDict, PromptType, TextPrompt, TokensPrompt`
