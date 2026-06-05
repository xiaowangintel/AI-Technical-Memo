# serving_tokenize.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/openai/serving_tokenize.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements serving tokenize logic for the OpenAI-compatible serving layer. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 serving tokenize 相关的逻辑，并服务于 OpenAI 兼容服务层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-3: Import runtime dependencies / 导入运行时依赖
```python
import logging
from http import HTTPStatus
from typing import List, Optional, Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 5-5: Import runtime dependencies / 导入运行时依赖
```python
from fastapi import Request
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 7-15: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.entrypoints.openai.protocol import (
    DetokenizeRequest,
    DetokenizeResponse,
    ErrorResponse,
    TokenizeRequest,
    TokenizeResponse,
)
from sglang.srt.entrypoints.openai.serving_base import OpenAIServingBase
from sglang.srt.entrypoints.openai.serving_chat import OpenAIServingChat
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 17-17: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 20-21: Provide supporting module logic / 提供辅助模块逻辑
```python
class OpenAIServingTokenize(OpenAIServingBase):
    """Handler for /v1/tokenize requests"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 23-29: Initialize OpenAIServingTokenize / 初始化 OpenAIServingTokenize
```python
    def __init__(self, tokenizer_manager, template_manager=None):
        super().__init__(tokenizer_manager)
        self.chat_serving: Optional[OpenAIServingChat] = (
            OpenAIServingChat(tokenizer_manager, template_manager)
            if template_manager is not None
            else None
        )
```
**EN:** This block implements the initializer `__init__(tokenizer_manager, template_manager)` for `OpenAIServingTokenize`. It prepares the object state and connects the instance to the surrounding serving tokenize workflow.
**CN:** 该代码块实现 `OpenAIServingTokenize` 的初始化方法 `__init__(tokenizer_manager, template_manager)`。它负责准备对象状态，并把实例接入 serving tokenize 相关的运行流程。

### Lines 31-32: Implement request id prefix / 实现请求 id prefix
```python
    def _request_id_prefix(self) -> str:
        return "tok-"
```
**EN:** This block implements the method `_request_id_prefix()` on `OpenAIServingTokenize`. It focuses on handling the serving tokenize responsibilities represented by `_request_id_prefix`, so the class can advance the serving tokenize workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingTokenize` 上的方法 `_request_id_prefix()`。它围绕 `_request_id_prefix` 所承担的 serving tokenize 相关职责展开，使该类能够独立推进相应流程。

### Lines 34-37: Implement convert to internal request / 实现convert to internal 请求
```python
    def _convert_to_internal_request(
        self, request: TokenizeRequest, raw_request: Request
    ) -> tuple[TokenizeRequest, TokenizeRequest]:
        return request, request
```
**EN:** This block implements the method `_convert_to_internal_request(request, raw_request)` on `OpenAIServingTokenize`. It focuses on handling the serving tokenize responsibilities represented by `_convert_to_internal_request`, so the class can advance the serving tokenize workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingTokenize` 上的方法 `_convert_to_internal_request(request, raw_request)`。它围绕 `_convert_to_internal_request` 所承担的 serving tokenize 相关职责展开，使该类能够独立推进相应流程。

### Lines 39-85: Implement async handle non streaming request / 实现异步handle non streaming 请求
```python
    async def _handle_non_streaming_request(
        self,
        adapted_request: TokenizeRequest,
        request: TokenizeRequest,
        raw_request: Request,
    ) -> Union[TokenizeResponse, ErrorResponse]:
        try:
            tokenizer = self.tokenizer_manager.tokenizer
            max_model_len = getattr(tokenizer, "model_max_length", -1)

            if request.messages is not None:
                token_ids = self._tokenize_chat_request(request)
                tokens = token_ids
                count = len(token_ids)
            elif isinstance(request.prompt, str):
                token_ids = tokenizer.encode(
                    request.prompt,
                    add_special_tokens=request.add_special_tokens,
                )
                tokens = token_ids
                count = len(token_ids)
            elif isinstance(request.prompt, list):
                token_ids_list = [
                    tokenizer.encode(
                        text, add_special_tokens=request.add_special_tokens
                    )
                    for text in request.prompt
                ]
                tokens = token_ids_list
                count = [len(ids) for ids in token_ids_list]
            else:
                return self.create_error_response(
                    f"Invalid prompt type: {type(request.prompt)}. Expected str or List[str]."
                )

            return TokenizeResponse(
                tokens=tokens, count=count, max_model_len=max_model_len
            )
        except ValueError as e:
            return self.create_error_response(str(e))
        except Exception as e:
            logger.error("Error during tokenization", exc_info=True)
            return self.create_error_response(
                f"Internal server error during tokenization: {e}",
                err_type="InternalServerError",
                status_code=HTTPStatus.INTERNAL_SERVER_ERROR,
            )
```
**EN:** This block implements the async method `_handle_non_streaming_request(adapted_request, request, raw_request)` on `OpenAIServingTokenize`. It focuses on handling the serving tokenize responsibilities represented by `_handle_non_streaming_request`, so the class can advance the serving tokenize workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingTokenize` 上的异步方法 `_handle_non_streaming_request(adapted_request, request, raw_request)`。它围绕 `_handle_non_streaming_request` 所承担的 serving tokenize 相关职责展开，使该类能够独立推进相应流程。

### Lines 87-115: Implement tokenize chat request / 实现tokenize 聊天 请求
```python
    def _tokenize_chat_request(self, request: TokenizeRequest) -> List[int]:
        if self.chat_serving is None:
            raise ValueError("Chat template tokenization requires a template manager.")

        chat_request = request.to_chat_completion_request()
        validation_error = self.chat_serving._validate_request(chat_request)
        if validation_error:
            raise ValueError(validation_error)

        is_multimodal = self.tokenizer_manager.model_config.is_multimodal
        processed_messages = self.chat_serving._process_messages(
            chat_request, is_multimodal
        )

        prompt_ids = processed_messages.prompt_ids
        if isinstance(prompt_ids, list) and (
            prompt_ids or not processed_messages.prompt
        ):
            return prompt_ids
        if isinstance(prompt_ids, str):
            return self.tokenizer_manager.tokenizer.encode(
                prompt_ids, add_special_tokens=False
            )
        if processed_messages.prompt:
            return self.tokenizer_manager.tokenizer.encode(
                processed_messages.prompt, add_special_tokens=False
            )

        raise ValueError("Failed to render chat messages into token ids.")
```
**EN:** This block implements the method `_tokenize_chat_request(request)` on `OpenAIServingTokenize`. It focuses on handling the serving tokenize responsibilities represented by `_tokenize_chat_request`, so the class can advance the serving tokenize workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingTokenize` 上的方法 `_tokenize_chat_request(request)`。它围绕 `_tokenize_chat_request` 所承担的 serving tokenize 相关职责展开，使该类能够独立推进相应流程。

### Lines 118-119: Provide supporting module logic / 提供辅助模块逻辑
```python
class OpenAIServingDetokenize(OpenAIServingBase):
    """Handler for /v1/detokenize requests"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 121-122: Implement request id prefix / 实现请求 id prefix
```python
    def _request_id_prefix(self) -> str:
        return "detok-"
```
**EN:** This block implements the method `_request_id_prefix()` on `OpenAIServingDetokenize`. It focuses on handling the serving tokenize responsibilities represented by `_request_id_prefix`, so the class can advance the serving tokenize workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingDetokenize` 上的方法 `_request_id_prefix()`。它围绕 `_request_id_prefix` 所承担的 serving tokenize 相关职责展开，使该类能够独立推进相应流程。

### Lines 124-127: Implement convert to internal request / 实现convert to internal 请求
```python
    def _convert_to_internal_request(
        self, request: DetokenizeRequest, raw_request: Request
    ) -> tuple[DetokenizeRequest, DetokenizeRequest]:
        return request, request
```
**EN:** This block implements the method `_convert_to_internal_request(request, raw_request)` on `OpenAIServingDetokenize`. It focuses on handling the serving tokenize responsibilities represented by `_convert_to_internal_request`, so the class can advance the serving tokenize workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingDetokenize` 上的方法 `_convert_to_internal_request(request, raw_request)`。它围绕 `_convert_to_internal_request` 所承担的 serving tokenize 相关职责展开，使该类能够独立推进相应流程。

### Lines 129-164: Implement async handle non streaming request / 实现异步handle non streaming 请求
```python
    async def _handle_non_streaming_request(
        self,
        adapted_request: DetokenizeRequest,
        request: DetokenizeRequest,
        raw_request: Request,
    ) -> Union[DetokenizeResponse, ErrorResponse]:
        try:
            tokenizer = self.tokenizer_manager.tokenizer

            if (
                isinstance(request.tokens, list)
                and request.tokens
                and isinstance(request.tokens[0], int)
            ):
                if not all(isinstance(t, int) for t in request.tokens):
                    return self.create_error_response(
                        "Invalid input: 'tokens' must be a list of integers."
                    )
                tokens_to_decode = [int(t) for t in request.tokens]
                text = tokenizer.decode(
                    tokens_to_decode, skip_special_tokens=request.skip_special_tokens
                )
                text_out: Union[str, List[str]] = text
            elif (
                isinstance(request.tokens, list)
                and request.tokens
                and isinstance(request.tokens[0], list)
            ):
                texts: List[str] = []
                for token_list in request.tokens:
                    if not all(isinstance(t, int) for t in token_list):
                        return self.create_error_response(
                            f"Invalid input: Sublist in 'tokens' must contain only integers. Found: {token_list}"
                        )
                    decoded_text = tokenizer.decode(
                        [int(t) for t in token_list],
```
**EN:** This block implements the async method `_handle_non_streaming_request(adapted_request, request, raw_request)` on `OpenAIServingDetokenize`. It focuses on handling the serving tokenize responsibilities represented by `_handle_non_streaming_request`, so the class can advance the serving tokenize workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingDetokenize` 上的异步方法 `_handle_non_streaming_request(adapted_request, request, raw_request)`。它围绕 `_handle_non_streaming_request` 所承担的 serving tokenize 相关职责展开，使该类能够独立推进相应流程。

### Lines 165-189: Continue async handle non streaming request / 继续说明异步handle non streaming 请求
```python
                        skip_special_tokens=request.skip_special_tokens,
                    )
                    texts.append(decoded_text)
                text_out = texts
            elif isinstance(request.tokens, list) and not request.tokens:
                text_out = ""
            else:
                return self.create_error_response(
                    f"Invalid tokens type: {type(request.tokens)}. Expected List[int] or List[List[int]]."
                )

            return DetokenizeResponse(text=text_out)
        except Exception as e:
            logger.error("Error during detokenization", exc_info=True)
            if "decode" in str(e).lower():
                return self.create_error_response(
                    f"Error decoding tokens: {e}. Input tokens might be invalid for the model.",
                    err_type="DecodeError",
                    status_code=HTTPStatus.BAD_REQUEST,
                )
            return self.create_error_response(
                f"Internal server error during detokenization: {e}",
                err_type="InternalServerError",
                status_code=HTTPStatus.INTERNAL_SERVER_ERROR,
            )
```
**EN:** This block implements the async method `_handle_non_streaming_request(adapted_request, request, raw_request)` on `OpenAIServingDetokenize`. It focuses on handling the serving tokenize responsibilities represented by `_handle_non_streaming_request`, so the class can advance the serving tokenize workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingDetokenize` 上的异步方法 `_handle_non_streaming_request(adapted_request, request, raw_request)`。它围绕 `_handle_non_streaming_request` 所承担的 serving tokenize 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: OpenAIServingTokenize, OpenAIServingDetokenize
- **Domain focus / 领域焦点**: serving tokenize / serving tokenize
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: logging, typing
- **Third-party / 第三方库**: fastapi, http
- **Local Modules / 本地模块**: sglang.srt.entrypoints.openai.protocol, sglang.srt.entrypoints.openai.serving_base, sglang.srt.entrypoints.openai.serving_chat
