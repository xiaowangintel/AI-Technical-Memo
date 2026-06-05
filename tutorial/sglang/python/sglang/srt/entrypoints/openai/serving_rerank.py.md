# serving_rerank.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/openai/serving_rerank.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements serving rerank logic for the OpenAI-compatible serving layer. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 serving rerank 相关的逻辑，并服务于 OpenAI 兼容服务层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-4: Import runtime dependencies / 导入运行时依赖
```python
import heapq
import logging
import math
from typing import Any, Dict, List, Optional, Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 6-7: Import runtime dependencies / 导入运行时依赖
```python
from fastapi import Request
from fastapi.responses import ORJSONResponse
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 9-19: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.entrypoints.openai.protocol import (
    ChatCompletionMessageContentImagePart,
    ChatCompletionMessageContentTextPart,
    ChatCompletionMessageContentVideoPart,
    ErrorResponse,
    RerankContent,
    RerankResponse,
    V1RerankReqInput,
)
from sglang.srt.entrypoints.openai.serving_base import OpenAIServingBase
from sglang.srt.managers.io_struct import EmbeddingReqInput, GenerateReqInput
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 21-21: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 24-49: Implement get yes no token ids / 实现get yes no Token ids
```python
def _get_yes_no_token_ids(tokenizer) -> tuple[int, int]:
    """Get token IDs for 'yes' and 'no' from the tokenizer.

    Different model sizes may have different token IDs, so we look them up dynamically.
    """
    # Try to encode 'yes' and 'no' to get their token IDs
    # The tokenizer should return a single token for these common words
    try:
        yes_tokens = tokenizer.encode("yes", add_special_tokens=False)
        no_tokens = tokenizer.encode("no", add_special_tokens=False)

        if len(yes_tokens) == 1 and len(no_tokens) == 1:
            return yes_tokens[0], no_tokens[0]

        # Fallback: try convert_tokens_to_ids
        yes_id = tokenizer.convert_tokens_to_ids("yes")
        no_id = tokenizer.convert_tokens_to_ids("no")
        if yes_id is not None and no_id is not None:
            return yes_id, no_id

    except Exception as e:
        logger.warning(f"Failed to get yes/no token IDs dynamically: {e}")

    # Fallback to known Qwen3 token IDs (may not work for all model sizes)
    logger.warning("Using fallback token IDs for yes/no (9693/2152)")
    return 9693, 2152
```
**EN:** This block implements the function `_get_yes_no_token_ids(tokenizer)`. It focuses on Get token IDs for 'yes' and 'no' from the tokenizer., providing reusable behavior for the serving rerank pipeline.
**CN:** 该代码块实现函数 `_get_yes_no_token_ids(tokenizer)`。它围绕 `_get_yes_no_token_ids` 所承担的 serving rerank 相关职责展开，为对应处理链路提供可复用能力。

### Lines 52-59: Implement is qwen3 reranker template / 实现is qwen3 reranker template
```python
def _is_qwen3_reranker_template(chat_template: str) -> bool:
    """Detect if the chat template is for Qwen3 text-only reranker."""
    if not chat_template:
        return False
    t = chat_template.lower()
    return ('answer can only be "yes" or "no"' in t) or (
        "answer can only be" in t and '"yes"' in t and '"no"' in t
    )
```
**EN:** This block implements the function `_is_qwen3_reranker_template(chat_template)`. It focuses on Detect if the chat template is for Qwen3 text-only reranker., providing reusable behavior for the serving rerank pipeline.
**CN:** 该代码块实现函数 `_is_qwen3_reranker_template(chat_template)`。它围绕 `_is_qwen3_reranker_template` 所承担的 serving rerank 相关职责展开，为对应处理链路提供可复用能力。

### Lines 62-77: Implement is qwen3 vl reranker template / 实现is qwen3 vl reranker template
```python
def _is_qwen3_vl_reranker_template(chat_template: str) -> bool:
    """Detect if the chat template is for Qwen3-VL multimodal reranker.

    VL reranker templates use `query` and `document` as jinja variables
    and include vision token placeholders for image/video support.
    """
    if not chat_template:
        return False
    t = chat_template.lower()
    # Check for reranker phrase (yes/no judgment)
    has_reranker_phrase = ('answer can only be "yes" or "no"' in t) or (
        "answer can only be" in t and '"yes"' in t and '"no"' in t
    )
    # Check for vision token placeholders (unique to VL templates)
    has_vision_tokens = "<|vision_start|>" in t or "<|image_pad|>" in t
    return has_reranker_phrase and has_vision_tokens
```
**EN:** This block implements the function `_is_qwen3_vl_reranker_template(chat_template)`. It focuses on Detect if the chat template is for Qwen3-VL multimodal reranker., providing reusable behavior for the serving rerank pipeline.
**CN:** 该代码块实现函数 `_is_qwen3_vl_reranker_template(chat_template)`。它围绕 `_is_qwen3_vl_reranker_template` 所承担的 serving rerank 相关职责展开，为对应处理链路提供可复用能力。

### Lines 80-85: Implement is qwen3 vl model / 实现is qwen3 vl 模型
```python
def _is_qwen3_vl_model(model_path: str) -> bool:
    """Check if the model is a Qwen3-VL model based on model path."""
    if not model_path:
        return False
    model_lower = model_path.lower()
    return "qwen3-vl" in model_lower or "qwen3vl" in model_lower
```
**EN:** This block implements the function `_is_qwen3_vl_model(model_path)`. It focuses on Check if the model is a Qwen3-VL model based on model path., providing reusable behavior for the serving rerank pipeline.
**CN:** 该代码块实现函数 `_is_qwen3_vl_model(model_path)`。它围绕 `_is_qwen3_vl_model` 所承担的 serving rerank 相关职责展开，为对应处理链路提供可复用能力。

### Lines 88-111: Implement detect rerank backend / 实现detect rerank backend
```python
def _detect_rerank_backend(
    *,
    request: V1RerankReqInput,
    chat_template: Optional[str],
    model_path: str,
) -> str:
    """
    Unify rerank routing decisions used by both `_convert_to_internal_request` and
    `_handle_non_streaming_request`.

    Returns:
        "vl_decoder" | "text_decoder" | "cross_encoder"
    """
    is_multimodal = request.is_multimodal()
    is_vl_model = _is_qwen3_vl_model(model_path)
    is_vl_template = _is_qwen3_vl_reranker_template(chat_template)
    is_text_template = _is_qwen3_reranker_template(chat_template)

    # Prefer VL when template/model indicates VL, or request is multimodal with reranker template.
    if is_vl_template or is_vl_model or (is_multimodal and is_text_template):
        return "vl_decoder"
    if is_text_template:
        return "text_decoder"
    return "cross_encoder"
```
**EN:** This block implements the function `_detect_rerank_backend(request, chat_template, model_path)`. It focuses on Unify rerank routing decisions used by both `_convert_to_internal_request` and `_handle_non_streaming_request`., providing reusable behavior for the serving rerank pipeline.
**CN:** 该代码块实现函数 `_detect_rerank_backend(request, chat_template, model_path)`。它围绕 `_detect_rerank_backend` 所承担的 serving rerank 相关职责展开，为对应处理链路提供可复用能力。

### Lines 114-118: Implement qwen3 rerank score / 实现qwen3 rerank score
```python
def _qwen3_rerank_score(p_yes: float, p_no: float) -> float:
    denom = p_yes + p_no
    if denom <= 0.0:
        return 0.0
    return p_yes / denom
```
**EN:** This block implements the function `_qwen3_rerank_score(p_yes, p_no)`. It focuses on handling the serving rerank responsibilities represented by `_qwen3_rerank_score`, providing reusable behavior for the serving rerank pipeline.
**CN:** 该代码块实现函数 `_qwen3_rerank_score(p_yes, p_no)`。它围绕 `_qwen3_rerank_score` 所承担的 serving rerank 相关职责展开，为对应处理链路提供可复用能力。

### Lines 121-135: Implement get jinja env / 实现get jinja env
```python
def _get_jinja_env():
    try:
        import jinja2  # Lazy import: server env should provide this dependency.
        from jinja2.sandbox import ImmutableSandboxedEnvironment
    except ModuleNotFoundError as e:
        raise ValueError(
            "Rendering Qwen3 reranker prompts requires `jinja2`. "
            "Please install it in your runtime environment (e.g., `pip install jinja2`)."
        ) from e
    # Using a sandboxed environment to stop malicious execution during model loading.
    return ImmutableSandboxedEnvironment(
        loader=jinja2.BaseLoader(),
        autoescape=False,
        undefined=jinja2.Undefined,
    )
```
**EN:** This block implements the function `_get_jinja_env()`. It focuses on handling the serving rerank responsibilities represented by `_get_jinja_env`, providing reusable behavior for the serving rerank pipeline.
**CN:** 该代码块实现函数 `_get_jinja_env()`。它围绕 `_get_jinja_env` 所承担的 serving rerank 相关职责展开，为对应处理链路提供可复用能力。

### Lines 138-165: Implement render jinja chat template / 实现render jinja 聊天 template
```python
def _render_jinja_chat_template(
    chat_template: str,
    *,
    query: RerankContent,
    document: RerankContent,
    instruct: Optional[str],
) -> str:
    """Render a loaded Jinja chat template for Qwen3 reranker prompts (text-only)."""
    env = _get_jinja_env()
    template = env.from_string(chat_template)

    # For text-only template, extract text content
    query_text = query if isinstance(query, str) else _extract_text_from_content(query)
    doc_text = (
        document if isinstance(document, str) else _extract_text_from_content(document)
    )

    render_kwargs = {
        "messages": [
            {"role": "user", "content": query_text},
            {"role": "user", "content": doc_text},
        ]
    }
    # Only pass instruct when explicitly provided; template uses `default(...)`
    # which works only when the variable is undefined (not None).
    if instruct:
        render_kwargs["instruct"] = instruct
    return template.render(**render_kwargs)
```
**EN:** This block implements the function `_render_jinja_chat_template(chat_template, query, document, instruct)`. It focuses on Render a loaded Jinja chat template for Qwen3 reranker prompts (text-only)., providing reusable behavior for the serving rerank pipeline.
**CN:** 该代码块实现函数 `_render_jinja_chat_template(chat_template, query, document, instruct)`。它围绕 `_render_jinja_chat_template` 所承担的 serving rerank 相关职责展开，为对应处理链路提供可复用能力。

### Lines 168-189: Implement render vl jinja template / 实现render vl jinja template
```python
def _render_vl_jinja_template(
    chat_template: str,
    *,
    query: List[Dict[str, Any]],
    document: List[Dict[str, Any]],
    instruct: Optional[str],
) -> str:
    """Render a loaded Jinja chat template for Qwen3-VL reranker prompts (multimodal).

    The template expects `query` and `document` as lists of content parts,
    where each part has a `type` field (text, image, video) and corresponding data.
    """
    env = _get_jinja_env()
    template = env.from_string(chat_template)

    render_kwargs = {
        "query": query,
        "document": document,
    }
    if instruct:
        render_kwargs["instruct"] = instruct
    return template.render(**render_kwargs)
```
**EN:** This block implements the function `_render_vl_jinja_template(chat_template, query, document, instruct)`. It focuses on Render a loaded Jinja chat template for Qwen3-VL reranker prompts (multimodal)., providing reusable behavior for the serving rerank pipeline.
**CN:** 该代码块实现函数 `_render_vl_jinja_template(chat_template, query, document, instruct)`。它围绕 `_render_vl_jinja_template` 所承担的 serving rerank 相关职责展开，为对应处理链路提供可复用能力。

### Lines 192-202: Implement extract text from content / 实现extract text from content
```python
def _extract_text_from_content(content: RerankContent) -> str:
    """Extract text from multimodal content."""
    if isinstance(content, str):
        return content
    texts = []
    for part in content:
        if isinstance(part, ChatCompletionMessageContentTextPart):
            texts.append(part.text)
        elif isinstance(part, dict) and part.get("type") == "text":
            texts.append(part.get("text", ""))
    return " ".join(texts)
```
**EN:** This block implements the function `_extract_text_from_content(content)`. It focuses on Extract text from multimodal content., providing reusable behavior for the serving rerank pipeline.
**CN:** 该代码块实现函数 `_extract_text_from_content(content)`。它围绕 `_extract_text_from_content` 所承担的 serving rerank 相关职责展开，为对应处理链路提供可复用能力。

### Lines 205-206: Provide supporting module logic / 提供辅助模块逻辑
```python
class OpenAIServingRerank(OpenAIServingBase):
    """Handler for /v1/rerank requests"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 208-217: Initialize OpenAIServingRerank / 初始化 OpenAIServingRerank
```python
    def __init__(self, tokenizer_manager, template_manager=None):
        super().__init__(tokenizer_manager)
        # TemplateManager is optional; rerank uses tokenizer.chat_template today.
        # Keeping this explicit makes the dependency clear and supports future extensions.
        self.template_manager = template_manager

        # Cache yes/no token IDs for Qwen3 reranker scoring
        self._yes_token_id, self._no_token_id = _get_yes_no_token_ids(
            tokenizer_manager.tokenizer
        )
```
**EN:** This block implements the initializer `__init__(tokenizer_manager, template_manager)` for `OpenAIServingRerank`. It prepares the object state and connects the instance to the surrounding serving rerank workflow.
**CN:** 该代码块实现 `OpenAIServingRerank` 的初始化方法 `__init__(tokenizer_manager, template_manager)`。它负责准备对象状态，并把实例接入 serving rerank 相关的运行流程。

### Lines 218-221: Import runtime dependencies / 导入运行时依赖
```python

    # NOTE: /v1/rerank is not an official OpenAI endpoint. This module may be moved
    # to another module in the future.
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 222-223: Implement request id prefix / 实现请求 id prefix
```python
    def _request_id_prefix(self) -> str:
        return "rerank-"
```
**EN:** This block implements the method `_request_id_prefix()` on `OpenAIServingRerank`. It focuses on handling the serving rerank responsibilities represented by `_request_id_prefix`, so the class can advance the serving rerank workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingRerank` 上的方法 `_request_id_prefix()`。它围绕 `_request_id_prefix` 所承担的 serving rerank 相关职责展开，使该类能够独立推进相应流程。

### Lines 225-243: Implement validate request / 实现validate 请求
```python
    def _validate_request(self, request: V1RerankReqInput) -> Optional[str]:
        """Validate rerank request format and content"""
        if not request.query:
            return "Query cannot be empty"

        if isinstance(request.query, str):
            if not request.query.strip():
                return "Query cannot be empty or whitespace only"

        if not request.documents:
            return "Documents cannot be empty"

        for doc in request.documents:
            if not doc:
                return "Each document must be a non-empty string"
            if isinstance(doc, str) and not doc.strip():
                return "Each document cannot be empty or whitespace only"

        return None
```
**EN:** This block implements the method `_validate_request(request)` on `OpenAIServingRerank`. It focuses on Validate rerank request format and content, so the class can advance the serving rerank workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingRerank` 上的方法 `_validate_request(request)`。它围绕 `_validate_request` 所承担的 serving rerank 相关职责展开，使该类能够独立推进相应流程。

### Lines 245-279: Implement convert to internal request / 实现convert to internal 请求
```python
    def _convert_to_internal_request(
        self,
        request: V1RerankReqInput,
        raw_request: Request = None,
    ) -> tuple[Union[EmbeddingReqInput, V1RerankReqInput], V1RerankReqInput]:
        """
        Convert OpenAI rerank request to internal format.

        - For Qwen3-VL reranker (multimodal decoder-only): keep the request.
        - For Qwen3 reranker (text-only decoder-only): keep the request and score via
          `tokenizer_manager.score_prompts(...)` in the handler.
        - For cross-encoder rerank models: adapt into `EmbeddingReqInput` pairs.
        """
        chat_template = self.tokenizer_manager.tokenizer.chat_template
        model_path = getattr(self.tokenizer_manager.model_config, "model_path", "")
        backend = _detect_rerank_backend(
            request=request,
            chat_template=chat_template if isinstance(chat_template, str) else None,
            model_path=model_path,
        )
        if backend in ("vl_decoder", "text_decoder"):
            return request, request

        # Cross-encoder rerank: Create pairs of [query, document] for each document.
        # Note: Cross-encoder only supports text-only content
        if request.is_multimodal():
            # Extract text for cross-encoder (multimodal not supported)
            query_text = _extract_text_from_content(request.query)
            doc_texts = [_extract_text_from_content(doc) for doc in request.documents]
            pairs = [[query_text, doc] for doc in doc_texts]
        else:
            pairs = [[request.query, doc] for doc in request.documents]

        adapted_request = EmbeddingReqInput(text=pairs, is_cross_encoder_request=True)
        return adapted_request, request
```
**EN:** This block implements the method `_convert_to_internal_request(request, raw_request)` on `OpenAIServingRerank`. It focuses on Convert OpenAI rerank request to internal format., so the class can advance the serving rerank workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingRerank` 上的方法 `_convert_to_internal_request(request, raw_request)`。它围绕 `_convert_to_internal_request` 所承担的 serving rerank 相关职责展开，使该类能够独立推进相应流程。

### Lines 281-317: Implement async handle non streaming request / 实现异步handle non streaming 请求
```python
    async def _handle_non_streaming_request(
        self,
        adapted_request: Union[EmbeddingReqInput, V1RerankReqInput],
        request: V1RerankReqInput,
        raw_request: Request,
    ) -> Union[List[RerankResponse], ErrorResponse, ORJSONResponse]:
        """Handle the rerank request"""
        chat_template = getattr(self.tokenizer_manager.tokenizer, "chat_template", None)
        model_path = getattr(self.tokenizer_manager.model_config, "model_path", "")
        rerank_ret = await self._handle_rerank_paths(
            request=request,
            raw_request=raw_request,
            chat_template=chat_template,
            model_path=model_path,
        )
        if rerank_ret is not None:
            return rerank_ret

        # Default cross-encoder rerank path (existing behavior).
        try:
            if not isinstance(adapted_request, EmbeddingReqInput):
                raise ValueError(
                    "Invalid rerank request adaptation. "
                    "If you are serving a decoder-only reranker (e.g., Qwen3-Reranker), "
                    "please provide the corresponding --chat-template and launch without --is-embedding."
                )
            ret = await self.tokenizer_manager.generate_request(
                adapted_request, raw_request
            ).__anext__()
        except ValueError as e:
            return self.create_error_response(str(e))

        if not isinstance(ret, list):
            ret = [ret]

        responses = self._build_rerank_response(ret, request)
        return responses
```
**EN:** This block implements the async method `_handle_non_streaming_request(adapted_request, request, raw_request)` on `OpenAIServingRerank`. It focuses on Handle the rerank request, so the class can advance the serving rerank workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingRerank` 上的异步方法 `_handle_non_streaming_request(adapted_request, request, raw_request)`。它围绕 `_handle_non_streaming_request` 所承担的 serving rerank 相关职责展开，使该类能够独立推进相应流程。

### Lines 319-352: Implement async handle rerank paths / 实现异步handle rerank paths
```python
    async def _handle_rerank_paths(
        self,
        *,
        request: V1RerankReqInput,
        raw_request: Request,
        chat_template: Optional[str],
        model_path: str,
    ) -> Optional[Union[List[RerankResponse], ErrorResponse, ORJSONResponse]]:
        """
        Handle decoder-only rerank paths (VL/text) and return a response if matched.

        Returns None if the request should fall back to cross-encoder rerank.
        """
        backend = _detect_rerank_backend(
            request=request,
            chat_template=chat_template,
            model_path=model_path,
        )

        # Qwen3-VL reranker path (decoder-only scoring with query/document template format)
        if backend == "vl_decoder":
            return await self._handle_vl_reranker_request(
                request, raw_request, chat_template or ""
            )

        # Qwen3 text-only reranker path (decoder-only scoring).
        if backend == "text_decoder":
            return await self._handle_text_reranker_request(
                request=request,
                raw_request=raw_request,
                chat_template=chat_template or "",
            )

        return None
```
**EN:** This block implements the async method `_handle_rerank_paths(request, raw_request, chat_template, model_path)` on `OpenAIServingRerank`. It focuses on Handle decoder-only rerank paths (VL/text) and return a response if matched., so the class can advance the serving rerank workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingRerank` 上的异步方法 `_handle_rerank_paths(request, raw_request, chat_template, model_path)`。它围绕 `_handle_rerank_paths` 所承担的 serving rerank 相关职责展开，使该类能够独立推进相应流程。

### Lines 354-396: Implement async handle text reranker request / 实现异步handle text reranker 请求
```python
    async def _handle_text_reranker_request(
        self,
        *,
        request: V1RerankReqInput,
        raw_request: Request,
        chat_template: str,
    ) -> Union[List[RerankResponse], ErrorResponse]:
        """Handle text-only decoder reranker request via score_prompts()."""
        # Qwen3 reranker relies on decoder-only logprobs. If the server is launched
        # with --is-embedding, model_config.is_generation is typically False and
        # logprob scoring is not supported.
        if not self.tokenizer_manager.model_config.is_generation:
            return self.create_error_response(
                "Detected Qwen3 reranker chat template, but the server is not in generation mode. "
                "Please relaunch without --is-embedding for Qwen3-Reranker models."
            )

        try:
            prompts = [
                _render_jinja_chat_template(
                    chat_template,
                    query=request.query,
                    document=doc,
                    instruct=getattr(request, "instruct", None),
                )
                for doc in request.documents
            ]

            result = await self.tokenizer_manager.score_prompts(
                prompts,
                label_token_ids=[self._yes_token_id, self._no_token_id],
                apply_softmax=False,
                request=raw_request,
            )
            scores = [_qwen3_rerank_score(s[0], s[1]) for s in result.scores]
        except ValueError as e:
            return self.create_error_response(str(e))
        except Exception as e:
            # Includes template rendering errors from jinja2.
            return self.create_error_response(str(e))

        responses = self._build_rerank_response(scores, request)
        return responses
```
**EN:** This block implements the async method `_handle_text_reranker_request(request, raw_request, chat_template)` on `OpenAIServingRerank`. It focuses on Handle text-only decoder reranker request via score_prompts()., so the class can advance the serving rerank workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingRerank` 上的异步方法 `_handle_text_reranker_request(request, raw_request, chat_template)`。它围绕 `_handle_text_reranker_request` 所承担的 serving rerank 相关职责展开，使该类能够独立推进相应流程。

### Lines 398-433: Implement async handle vl reranker request / 实现异步handle vl reranker 请求
```python
    async def _handle_vl_reranker_request(
        self,
        request: V1RerankReqInput,
        raw_request: Request,
        _chat_template: str,
    ) -> Union[List[RerankResponse], ErrorResponse]:
        """Handle multimodal VL reranker request using chat completion with logprobs."""
        if not self.tokenizer_manager.model_config.is_generation:
            return self.create_error_response(
                "Detected Qwen3-VL reranker, but the server is not in generation mode. "
                "Please relaunch without --is-embedding for Qwen3-VL-Reranker models."
            )

        try:
            scores = []
            instruct = getattr(request, "instruct", None)

            for doc in request.documents:
                # Build multimodal content lists and render prompt using jinja template
                query_content, doc_content, image_data, video_data = (
                    self._build_vl_reranker_content(
                        query=request.query,
                        document=doc,
                    )
                )

                # Render the chat template directly with query/document variables
                prompt = _render_vl_jinja_template(
                    chat_template=_chat_template,
                    query=query_content,
                    document=doc_content,
                    instruct=instruct,
                )

                # Create generate request with logprobs
                gen_request = GenerateReqInput(
```
**EN:** This block implements the async method `_handle_vl_reranker_request(request, raw_request, _chat_template)` on `OpenAIServingRerank`. It focuses on Handle multimodal VL reranker request using chat completion with logprobs., so the class can advance the serving rerank workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingRerank` 上的异步方法 `_handle_vl_reranker_request(request, raw_request, _chat_template)`。它围绕 `_handle_vl_reranker_request` 所承担的 serving rerank 相关职责展开，使该类能够独立推进相应流程。

### Lines 434-462: Continue async handle vl reranker request / 继续说明异步handle vl reranker 请求
```python
                    text=prompt,
                    image_data=image_data if image_data else None,
                    video_data=video_data if video_data else None,
                    sampling_params={
                        "max_new_tokens": 1,
                        "temperature": 0,
                    },
                    return_logprob=True,
                    top_logprobs_num=50,  # Get enough logprobs to find yes/no tokens
                    logprob_start_len=0,
                )

                # Execute generation request
                ret = await self.tokenizer_manager.generate_request(
                    gen_request, raw_request
                ).__anext__()

                # Extract yes/no probabilities from logprobs
                score = self._extract_score_from_logprobs(ret)
                scores.append(score)

            responses = self._build_rerank_response(scores, request)
            return responses

        except ValueError as e:
            return self.create_error_response(str(e))
        except Exception as e:
            logger.exception("Error handling VL reranker request")
            return self.create_error_response(str(e))
```
**EN:** This block implements the async method `_handle_vl_reranker_request(request, raw_request, _chat_template)` on `OpenAIServingRerank`. It focuses on Handle multimodal VL reranker request using chat completion with logprobs., so the class can advance the serving rerank workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingRerank` 上的异步方法 `_handle_vl_reranker_request(request, raw_request, _chat_template)`。它围绕 `_handle_vl_reranker_request` 所承担的 serving rerank 相关职责展开，使该类能够独立推进相应流程。

### Lines 464-484: Implement build vl reranker content / 实现build vl reranker content
```python
    def _build_vl_reranker_content(
        self,
        query: RerankContent,
        document: RerankContent,
    ) -> tuple[List[Dict[str, Any]], List[Dict[str, Any]], List[str], List[str]]:
        """Build content lists for VL reranker request.

        Returns:
            Tuple of (query_content, document_content, image_data, video_data)
            where query_content and document_content are lists suitable for jinja template.
        """
        image_data = []
        video_data = []

        # Build query content list
        query_content = self._content_to_template_list(query, image_data, video_data)

        # Build document content list
        doc_content = self._content_to_template_list(document, image_data, video_data)

        return query_content, doc_content, image_data, video_data
```
**EN:** This block implements the method `_build_vl_reranker_content(query, document)` on `OpenAIServingRerank`. It focuses on Build content lists for VL reranker request., so the class can advance the serving rerank workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingRerank` 上的方法 `_build_vl_reranker_content(query, document)`。它围绕 `_build_vl_reranker_content` 所承担的 serving rerank 相关职责展开，使该类能够独立推进相应流程。

### Lines 486-533: Implement content to template list / 实现content to template list
```python
    def _content_to_template_list(
        self,
        content: RerankContent,
        image_data: List[str],
        video_data: List[str],
    ) -> List[Dict[str, Any]]:
        """Convert RerankContent to a list format suitable for jinja template."""
        result = []

        if isinstance(content, str):
            result.append({"type": "text", "text": content})
            return result

        for part in content:
            if isinstance(part, ChatCompletionMessageContentTextPart):
                result.append({"type": "text", "text": part.text})
            elif isinstance(part, ChatCompletionMessageContentImagePart):
                if part.image_url:
                    image_data.append(part.image_url.url)
                    result.append({"type": "image"})
            elif isinstance(part, ChatCompletionMessageContentVideoPart):
                if part.video_url:
                    video_data.append(part.video_url.url)
                    result.append({"type": "video"})
            elif isinstance(part, dict):
                part_type = part.get("type")
                if part_type == "text":
                    result.append({"type": "text", "text": part.get("text", "")})
                elif part_type == "image_url":
                    image_url = part.get("image_url", {})
                    if isinstance(image_url, dict):
                        url = image_url.get("url")
                    else:
                        url = image_url
                    if url:
                        image_data.append(url)
                        result.append({"type": "image"})
                elif part_type == "video_url":
                    video_url = part.get("video_url", {})
                    if isinstance(video_url, dict):
                        url = video_url.get("url")
                    else:
                        url = video_url
                    if url:
                        video_data.append(url)
                        result.append({"type": "video"})

        return result
```
**EN:** This block implements the method `_content_to_template_list(content, image_data, video_data)` on `OpenAIServingRerank`. It focuses on Convert RerankContent to a list format suitable for jinja template., so the class can advance the serving rerank workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingRerank` 上的方法 `_content_to_template_list(content, image_data, video_data)`。它围绕 `_content_to_template_list` 所承担的 serving rerank 相关职责展开，使该类能够独立推进相应流程。

### Lines 535-562: Implement extract score from logprobs / 实现extract score from logprobs
```python
    def _extract_score_from_logprobs(self, ret: Dict[str, Any]) -> float:
        """Extract reranking score from generation response with logprobs."""
        # Get logprobs from the response
        meta_info = ret.get("meta_info", {})
        output_top_logprobs = meta_info.get("output_top_logprobs", [])

        # Use output_top_logprobs[0] - the model's prediction for the first generated token
        top_logprobs = output_top_logprobs[0] if output_top_logprobs else []

        # Find yes and no token probabilities
        # Format: list of tuples (logprob, token_id, token_text)
        p_yes = 0.0
        p_no = 0.0
        found_yes = False
        found_no = False

        for item in top_logprobs:
            logprob, token_id = item[0], item[1]
            if token_id == self._yes_token_id:
                p_yes = math.exp(logprob)
                found_yes = True
            elif token_id == self._no_token_id:
                p_no = math.exp(logprob)
                found_no = True
            if found_yes and found_no:
                break

        return _qwen3_rerank_score(p_yes, p_no)
```
**EN:** This block implements the method `_extract_score_from_logprobs(ret)` on `OpenAIServingRerank`. It focuses on Extract reranking score from generation response with logprobs., so the class can advance the serving rerank workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingRerank` 上的方法 `_extract_score_from_logprobs(ret)`。它围绕 `_extract_score_from_logprobs` 所承担的 serving rerank 相关职责展开，使该类能够独立推进相应流程。

### Lines 564-609: Implement build rerank response / 实现build rerank 响应
```python
    def _build_rerank_response(
        self, ret: Union[List[Dict[str, Any]], List[float]], request: V1RerankReqInput
    ) -> List[RerankResponse]:
        """Build the rerank response from generation results"""
        responses = []
        for idx, item in enumerate(ret):
            if isinstance(item, dict):
                score_val = item.get("embedding")
                # Some rerank/reward models return scalar score as embedding[0].
                if isinstance(score_val, list):
                    if len(score_val) == 0 or not isinstance(
                        score_val[0], (int, float)
                    ):
                        raise ValueError(
                            f"Invalid embedding score for rerank at index {idx}: {score_val!r}"
                        )
                    score_val = float(score_val[0])
                responses.append(
                    RerankResponse(
                        score=float(score_val),
                        document=(
                            request.documents[idx] if request.return_documents else None
                        ),
                        index=idx,
                        meta_info=item.get("meta_info"),
                    )
                )
            else:
                responses.append(
                    RerankResponse(
                        score=float(item),
                        document=(
                            request.documents[idx] if request.return_documents else None
                        ),
                        index=idx,
                    )
                )

        # When top_n is set, nlargest avoids fully sorting the candidate list
        # (O(N log top_n) vs O(N log N)) — meaningful for large rerank batches.
        # Validator (V1RerankReqInput.validate_top_n) guarantees top_n >= 1.
        if request.top_n is not None:
            return heapq.nlargest(request.top_n, responses, key=lambda x: x.score)

        responses.sort(key=lambda x: x.score, reverse=True)
        return responses
```
**EN:** This block implements the method `_build_rerank_response(ret, request)` on `OpenAIServingRerank`. It focuses on Build the rerank response from generation results, so the class can advance the serving rerank workflow in a self-contained way.
**CN:** 该代码块实现 `OpenAIServingRerank` 上的方法 `_build_rerank_response(ret, request)`。它围绕 `_build_rerank_response` 所承担的 serving rerank 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: OpenAIServingRerank
- **Main callables / 主要可调用对象**: _get_yes_no_token_ids, _is_qwen3_reranker_template, _is_qwen3_vl_reranker_template, _is_qwen3_vl_model, _detect_rerank_backend, _qwen3_rerank_score, _get_jinja_env, _render_jinja_chat_template, ...
- **Domain focus / 领域焦点**: serving rerank / serving rerank
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: logging, math, typing
- **Third-party / 第三方库**: fastapi, fastapi.responses, heapq, jinja2, jinja2.sandbox
- **Local Modules / 本地模块**: sglang.srt.entrypoints.openai.protocol, sglang.srt.entrypoints.openai.serving_base, sglang.srt.managers.io_struct
