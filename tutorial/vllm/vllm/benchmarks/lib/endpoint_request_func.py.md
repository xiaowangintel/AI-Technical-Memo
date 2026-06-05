# endpoint_request_func.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/benchmarks/lib/endpoint_request_func.py`
- **Repository**: vllm-project/vllm
- **Purpose**: The request function for API endpoints. / 该文件的核心目的为：The request function for API endpoints.

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-3)
```python
"""The request function for API endpoints."""
```
**EN:** This opening docstring states the module's intent, expected usage, and any important assumptions before the implementation details begin.
**CN:** 开头的文档字符串先说明模块意图、预期用法以及实现展开前需要了解的重要假设。

### Imports (lines 5-18)
```python
import codecs

import io

import json

import os

import sys

import time

import traceback

from collections.abc import Awaitable

from dataclasses import dataclass, field

from typing import Any, Literal, Protocol

import aiohttp

import regex as re

from tqdm.asyncio import tqdm
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 20-819)
```python
AIOHTTP_TIMEOUT = aiohttp.ClientTimeout(total=6 * 60 * 60)

ASYNC_REQUEST_FUNCS: dict[str, RequestFunc] = {
    "vllm": async_request_openai_completions,
    "openai": async_request_openai_completions,
    "openai-chat": async_request_openai_chat_completions,
    "openai-audio": async_request_openai_audio,
    "openai-embeddings": async_request_openai_embeddings,
    "openai-embeddings-chat": async_request_openai_embeddings_chat,
    "openai-embeddings-clip": async_request_openai_embeddings_clip,
    "openai-embeddings-vlm2vec": async_request_openai_embeddings_vlm2vec,
    # Infinity embedding server: https://github.com/michaelfeil/infinity
    "infinity-embeddings": async_request_infinity_embeddings,
    "infinity-embeddings-clip": async_request_infinity_embeddings_clip,
    # (Infinity embedding server does not support vlm2vec)
    "vllm-pooling": async_request_vllm_pooling,
    "vllm-rerank": async_request_vllm_rerank,
}

POOLING_BACKENDS = {
    "openai-embeddings",
    "openai-embeddings-chat",
    "openai-embeddings-clip",
    "openai-embeddings-vlm2vec",
    "infinity-embeddings",
    "infinity-embeddings-clip",
    "vllm-pooling",
    "vllm-rerank",
# ... omitted for brevity ...
```
**EN:** This constant/configuration block defines `AIOHTTP_TIMEOUT`, `ASYNC_REQUEST_FUNCS`, `POOLING_BACKENDS`, `OPENAI_COMPATIBLE_BACKENDS`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `AIOHTTP_TIMEOUT`, `ASYNC_REQUEST_FUNCS`, `POOLING_BACKENDS`, `OPENAI_COMPATIBLE_BACKENDS`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Large-file note
```python
# Focused on representative top-level definitions for a large module.
# The full file contains additional helpers following the same patterns.
```
**EN:** Because the file is large, the analysis below focuses on the most important top-level definitions and leaves repetitive helpers summarized at a higher level.
**CN:** 由于文件较大，下面的分析聚焦最重要的顶层定义；其余重复性辅助逻辑将以更高层次方式概括。

### Class `StreamedResponseHandler` (lines 23-62)
```python
class StreamedResponseHandler:
    """Handles streaming HTTP responses by accumulating chunks until complete
    messages are available."""

    def __init__(self):
        self.buffer = ""
        self._decoder = codecs.getincrementaldecoder("utf-8")()

    def add_chunk(self, chunk_bytes: bytes) -> list[str]:
        """Add a chunk of bytes to the buffer and return any complete
        messages."""
        chunk_str = self._decoder.decode(chunk_bytes)
        self.buffer += chunk_str

        messages = []

        # Split by double newlines (SSE message separator)
        while "\n\n" in self.buffer:
            message, self.buffer = self.buffer.split("\n\n", 1)
            message = message.strip()
            if message:
                messages.append(message)

    # ... omitted for brevity ...

        return messages
```
**EN:** Class `StreamedResponseHandler` is a structured building block in this module. Key methods include `__init__`, `add_chunk`, which define initialization, validation, transformation, or access patterns. The class docstring says: Handles streaming HTTP responses by accumulating chunks until complete messages are available.
**CN:** 类 `StreamedResponseHandler` 是该模块中的结构化构件。 关键方法包括 `__init__`, `add_chunk`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Handles streaming HTTP responses by accumulating chunks until complete messages are available.

### Method `StreamedResponseHandler.__init__` (lines 27-29)
```python
    def __init__(self):
        self.buffer = ""
        self._decoder = codecs.getincrementaldecoder("utf-8")()
```
**EN:** Method `StreamedResponseHandler.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `codecs.getincrementaldecoder` show the concrete execution path.
**CN:** Method `StreamedResponseHandler.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `codecs.getincrementaldecoder` 这样的关键调用展示了该代码块的具体执行路径。

### Method `StreamedResponseHandler.add_chunk` (lines 31-62)
```python
    def add_chunk(self, chunk_bytes: bytes) -> list[str]:
        """Add a chunk of bytes to the buffer and return any complete
        messages."""
        chunk_str = self._decoder.decode(chunk_bytes)
        self.buffer += chunk_str

        messages = []

        # Split by double newlines (SSE message separator)
        while "\n\n" in self.buffer:
            message, self.buffer = self.buffer.split("\n\n", 1)
            message = message.strip()
            if message:
                messages.append(message)

        # if self.buffer is not empty, check if it is a complete message
        # by removing data: prefix and check if it is a valid JSON
        if self.buffer.startswith("data: "):
            message_content = self.buffer.removeprefix("data: ").strip()
    # ... omitted for brevity ...

        return messages
```
**EN:** Method `StreamedResponseHandler.add_chunk` provides a reusable helper around the module's main workflow. The docstring highlights: Add a chunk of bytes to the buffer and return any complete messages. Key calls such as `self._decoder.decode`, `self.buffer.split`, `message.strip`, `messages.append`, `self.buffer.startswith` show the concrete execution path.
**CN:** Method `StreamedResponseHandler.add_chunk` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Add a chunk of bytes to the buffer and return any complete messages. 像 `self._decoder.decode`, `self.buffer.split`, `message.strip`, `messages.append`, `self.buffer.startswith` 这样的关键调用展示了该代码块的具体执行路径。

### Class `RequestFuncInput` (lines 66-81)
```python
class RequestFuncInput:
    """The input for the request function."""

    prompt: str | list[str]
    api_url: str
    prompt_len: int
    output_len: int
    model: str
    model_name: str | None = None
    logprobs: int | None = None
    extra_headers: dict | None = None
    extra_body: dict | None = None
    multi_modal_content: dict | list[dict] | None = None
    ignore_eos: bool = False
    language: str | None = None
    request_id: str | None = None
```
**EN:** Class `RequestFuncInput` is a structured building block in this module and uses a dataclass-style declaration to store explicit state. The class docstring says: The input for the request function.
**CN:** 类 `RequestFuncInput` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。 类文档说明：The input for the request function.

### Class `RequestFuncOutput` (lines 85-98)
```python
class RequestFuncOutput:
    """The output of the request function including metrics."""

    generated_text: str = ""
    success: bool = False
    latency: float = 0.0
    output_tokens: int = 0
    ttft: float = 0.0  # Time to first token
    itl: list[float] = field(default_factory=list)  # list of inter-token latencies
    tpot: float = 0.0  # avg next-token latencies
    prompt_len: int = 0
    error: str = ""
    start_time: float = 0.0
    input_audio_duration: float = 0.0  # in seconds
```
**EN:** Class `RequestFuncOutput` is a structured building block in this module and uses a dataclass-style declaration to store explicit state. The class docstring says: The output of the request function including metrics.
**CN:** 类 `RequestFuncOutput` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。 类文档说明：The output of the request function including metrics.

### Class `RequestFunc` (lines 101-107)
```python
class RequestFunc(Protocol):
    def __call__(
        self,
        request_func_input: RequestFuncInput,
        session: aiohttp.ClientSession,
        pbar: tqdm | None = None,
    ) -> Awaitable[RequestFuncOutput]: ...
```
**EN:** Class `RequestFunc` is a structured building block in this module. It inherits from `Protocol`. Key methods include `__call__`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `RequestFunc` 是该模块中的结构化构件，继承自 `Protocol`。 关键方法包括 `__call__`，它们共同定义初始化、校验、变换或访问模式。

### Method `RequestFunc.__call__` (lines 102-107)
```python
    def __call__(
        self,
        request_func_input: RequestFuncInput,
        session: aiohttp.ClientSession,
        pbar: tqdm | None = None,
    ) -> Awaitable[RequestFuncOutput]: ...
```
**EN:** Method `RequestFunc.__call__` provides a reusable helper around the module's main workflow.
**CN:** Method `RequestFunc.__call__` 为模块主流程提供可复用的辅助逻辑。

### Function `async_request_openai_completions` (lines 154-264)
```python
async def async_request_openai_completions(
    request_func_input: RequestFuncInput,
    session: aiohttp.ClientSession,
    pbar: tqdm | None = None,
) -> RequestFuncOutput:
    """The async request function for the OpenAI Completions API.

    Args:
        request_func_input: The input for the request function.
        pbar: The progress bar to display the progress.

    Returns:
        The output of the request function.
    """
    api_url = request_func_input.api_url
    _validate_api_url(api_url, "OpenAI Completions API", "completions")

    payload = {
        "model": request_func_input.model_name
        if request_func_input.model_name
        else request_func_input.model,
        "prompt": request_func_input.prompt,
        "repetition_penalty": 1.0,
        "max_tokens": request_func_input.output_len,
        "logprobs": request_func_input.logprobs,
    # ... omitted for brevity ...
        pbar.update(1)
    return output
```
**EN:** Function `async_request_openai_completions` handles loading or retrieval of external/internal data. The docstring highlights: The async request function for the OpenAI Completions API. Key calls such as `_validate_api_url`, `_update_payload_common`, `_get_headers`, `_update_headers_common`, `RequestFuncOutput` show the concrete execution path.
**CN:** Function `async_request_openai_completions` 负责加载或获取外部/内部数据。 文档字符串强调：The async request function for the OpenAI Completions API. 像 `_validate_api_url`, `_update_payload_common`, `_get_headers`, `_update_headers_common`, `RequestFuncOutput` 这样的关键调用展示了该代码块的具体执行路径。

### Function `async_request_openai_chat_completions` (lines 291-383)
```python
async def async_request_openai_chat_completions(
    request_func_input: RequestFuncInput,
    session: aiohttp.ClientSession,
    pbar: tqdm | None = None,
    mm_position: Literal["first", "last"] = "last",
) -> RequestFuncOutput:
    api_url = request_func_input.api_url
    _validate_api_url(api_url, "OpenAI Chat Completions API", "chat/completions")

    content = _get_chat_content(request_func_input, mm_position=mm_position)

    payload = {
        "model": request_func_input.model_name
        if request_func_input.model_name
        else request_func_input.model,
        "messages": [
            {"role": "user", "content": content},
        ],
        "max_completion_tokens": request_func_input.output_len,
        "stream": True,
        "stream_options": {
            "include_usage": True,
        },
    }
    _update_payload_common(payload, request_func_input)
    # ... omitted for brevity ...
        pbar.update(1)
    return output
```
**EN:** Function `async_request_openai_chat_completions` handles loading or retrieval of external/internal data. Key calls such as `_validate_api_url`, `_get_chat_content`, `_update_payload_common`, `_get_headers`, `_update_headers_common` show the concrete execution path.
**CN:** Function `async_request_openai_chat_completions` 负责加载或获取外部/内部数据。 像 `_validate_api_url`, `_get_chat_content`, `_update_payload_common`, `_get_headers`, `_update_headers_common` 这样的关键调用展示了该代码块的具体执行路径。

### Function `async_request_openai_audio` (lines 386-495)
```python
async def async_request_openai_audio(
    request_func_input: RequestFuncInput,
    session: aiohttp.ClientSession,
    pbar: tqdm | None = None,
) -> RequestFuncOutput:
    # Lazy import without PlaceholderModule to avoid vllm dep.
    import soundfile

    api_url = request_func_input.api_url
    _validate_api_url(api_url, "OpenAI Audio API", {"transcriptions", "translations"})

    content = [{"type": "text", "text": request_func_input.prompt}]
    payload = {
        "model": request_func_input.model_name
        if request_func_input.model_name
        else request_func_input.model,
        "max_completion_tokens": request_func_input.output_len,
        "stream": True,
        "language": "en",
        # Flattened due to multipart/form-data
        "stream_include_usage": True,
        "stream_continuous_usage_stats": True,
    }
    _update_payload_common(payload, request_func_input)

    # ... omitted for brevity ...
        pbar.update(1)
    return output
```
**EN:** Function `async_request_openai_audio` handles loading or retrieval of external/internal data. Key calls such as `_validate_api_url`, `_update_payload_common`, `_get_headers`, `_update_headers_common`, `io.BytesIO` show the concrete execution path.
**CN:** Function `async_request_openai_audio` 负责加载或获取外部/内部数据。 像 `_validate_api_url`, `_update_payload_common`, `_get_headers`, `_update_headers_common`, `io.BytesIO` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_run_pooling_request` (lines 498-532)
```python
async def _run_pooling_request(
    session: aiohttp.ClientSession,
    api_url: str,
    payload: dict[str, Any],
    headers: dict[str, Any],
    pbar: tqdm | None = None,
) -> RequestFuncOutput:
    output = RequestFuncOutput()
    st = time.perf_counter()
    output.start_time = st
    try:
        async with session.post(url=api_url, headers=headers, json=payload) as response:
            if response.status == 200:
                output.ttft = output.latency = time.perf_counter() - st

                if payload.get("encoding_format", "float") == "bytes":
                    metadata = json.loads(response.headers["metadata"])
                    usage = metadata.get("usage", {})
                else:
                    data = await response.json()
                    usage = data.get("usage", {})

                output.success = True
                output.generated_text = ""
                output.prompt_len = usage.get("prompt_tokens", 0)
    # ... omitted for brevity ...
        pbar.update(1)
    return output
```
**EN:** Function `_run_pooling_request` provides a reusable helper around the module's main workflow. Key calls such as `RequestFuncOutput`, `time.perf_counter`, `session.post`, `payload.get`, `json.loads` show the concrete execution path.
**CN:** Function `_run_pooling_request` 为模块主流程提供可复用的辅助逻辑。 像 `RequestFuncOutput`, `time.perf_counter`, `session.post`, `payload.get`, `json.loads` 这样的关键调用展示了该代码块的具体执行路径。

### Function `async_request_openai_embeddings` (lines 535-563)
```python
async def async_request_openai_embeddings(
    request_func_input: RequestFuncInput,
    session: aiohttp.ClientSession,
    pbar: tqdm | None = None,
) -> RequestFuncOutput:
    api_url = request_func_input.api_url
    _validate_api_url(api_url, "OpenAI Embeddings API", "embeddings")

    payload = {
        "model": request_func_input.model_name
        if request_func_input.model_name
        else request_func_input.model,
        "input": request_func_input.prompt,
        # Many embedding models have short context length,
        # this is to avoid dropping some of the requests.
        "truncate_prompt_tokens": -1,
    }
    _update_payload_common(payload, request_func_input)

    headers = _get_headers("application/json")
    _update_headers_common(headers, request_func_input)

    return await _run_pooling_request(
        session,
        api_url,
    # ... omitted for brevity ...
        pbar=pbar,
    )
```
**EN:** Function `async_request_openai_embeddings` handles loading or retrieval of external/internal data. Key calls such as `_validate_api_url`, `_update_payload_common`, `_get_headers`, `_update_headers_common`, `_run_pooling_request` show the concrete execution path.
**CN:** Function `async_request_openai_embeddings` 负责加载或获取外部/内部数据。 像 `_validate_api_url`, `_update_payload_common`, `_get_headers`, `_update_headers_common`, `_run_pooling_request` 这样的关键调用展示了该代码块的具体执行路径。

### Function `async_request_vllm_rerank` (lines 566-599)
```python
async def async_request_vllm_rerank(
    request_func_input: RequestFuncInput,
    session: aiohttp.ClientSession,
    pbar: tqdm | None = None,
) -> RequestFuncOutput:
    api_url = request_func_input.api_url
    _validate_api_url(api_url, "vLLM score API", "rerank")

    assert (
        isinstance(request_func_input.prompt, list)
        and len(request_func_input.prompt) > 1
    )

    payload = {
        "model": request_func_input.model_name
        if request_func_input.model_name
        else request_func_input.model,
        "query": request_func_input.prompt[0],
        "documents": request_func_input.prompt[1:],
        # Many reranker models have short context length,
        # this is to avoid dropping some of the requests.
        "truncate_prompt_tokens": -1,
    }

    headers = _get_headers("application/json")
    # ... omitted for brevity ...
        pbar=pbar,
    )
```
**EN:** Function `async_request_vllm_rerank` provides a reusable helper around the module's main workflow. Key calls such as `_validate_api_url`, `isinstance`, `len`, `_get_headers`, `_update_headers_common` show the concrete execution path.
**CN:** Function `async_request_vllm_rerank` 为模块主流程提供可复用的辅助逻辑。 像 `_validate_api_url`, `isinstance`, `len`, `_get_headers`, `_update_headers_common` 这样的关键调用展示了该代码块的具体执行路径。

### Function `async_request_openai_embeddings_chat` (lines 602-635)
```python
async def async_request_openai_embeddings_chat(
    request_func_input: RequestFuncInput,
    session: aiohttp.ClientSession,
    pbar: tqdm | None = None,
    mm_position: Literal["first", "last"] = "last",
) -> RequestFuncOutput:
    api_url = request_func_input.api_url
    _validate_api_url(api_url, "OpenAI Embeddings API", "embeddings")

    content = _get_chat_content(request_func_input, mm_position=mm_position)

    payload = {
        "model": request_func_input.model_name
        if request_func_input.model_name
        else request_func_input.model,
        "messages": [
            {"role": "user", "content": content},
        ],
        # Many embedding models have short context length,
        # this is to avoid dropping some of the requests.
        "truncate_prompt_tokens": -1,
    }
    _update_payload_common(payload, request_func_input)

    headers = _get_headers("application/json")
    # ... omitted for brevity ...
        pbar=pbar,
    )
```
**EN:** Function `async_request_openai_embeddings_chat` handles loading or retrieval of external/internal data. Key calls such as `_validate_api_url`, `_get_chat_content`, `_update_payload_common`, `_get_headers`, `_update_headers_common` show the concrete execution path.
**CN:** Function `async_request_openai_embeddings_chat` 负责加载或获取外部/内部数据。 像 `_validate_api_url`, `_get_chat_content`, `_update_payload_common`, `_get_headers`, `_update_headers_common` 这样的关键调用展示了该代码块的具体执行路径。

### Function `async_request_infinity_embeddings` (lines 703-738)
```python
async def async_request_infinity_embeddings(
    request_func_input: RequestFuncInput,
    session: aiohttp.ClientSession,
    pbar: tqdm | None = None,
) -> RequestFuncOutput:
    api_url = request_func_input.api_url
    _validate_api_url(api_url, "Infinity Embeddings API", "embeddings")

    payload = {
        "model": request_func_input.model_name
        if request_func_input.model_name
        else request_func_input.model,
    }

    if request_func_input.prompt:
        payload["input"] = request_func_input.prompt
    else:
        mm_content = request_func_input.multi_modal_content
        assert isinstance(mm_content, dict)

        mm_type = mm_content["type"]
        payload["input"] = mm_content[mm_type]["url"]
        payload["modality"] = mm_type.split("_", 1)[0]

    _update_payload_common(payload, request_func_input)
    # ... omitted for brevity ...
        pbar=pbar,
    )
```
**EN:** Function `async_request_infinity_embeddings` constructs derived objects, runtime state, or helper structures. Key calls such as `_validate_api_url`, `isinstance`, `mm_type.split`, `_update_payload_common`, `_get_headers` show the concrete execution path.
**CN:** Function `async_request_infinity_embeddings` 负责构造派生对象、运行时状态或辅助结构。 像 `_validate_api_url`, `isinstance`, `mm_type.split`, `_update_payload_common`, `_get_headers` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Multimodality / 多模态**
  - **EN:** The code contains modality-specific paths for text, image, audio, or video data.
  - **CN:** 代码包含面向文本、图像、音频或视频数据的模态专用路径。
- **Structured types / 结构化类型**
  - **EN:** Dataclasses, typed dictionaries, or aliases are used to make runtime contracts explicit.
  - **CN:** 通过 dataclass、类型化字典或别名来显式表达运行时契约。
- **Benchmarking/data prep / 基准测试/数据准备**
  - **EN:** The file includes utilities for preparing benchmark inputs or measuring runtime behavior.
  - **CN:** 该文件包含用于准备基准测试输入或测量运行时行为的工具。
- **Configuration / 配置**
  - **EN:** Configuration objects and validation rules centralize feature toggles and runtime settings.
  - **CN:** 配置对象与校验规则集中管理功能开关和运行时设置。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import codecs`, `import io`, `import json`, `import os`, `import sys`, `import time`, `import traceback`, `from collections.abc import Awaitable`, `from dataclasses import dataclass, field`, `from typing import Any, Literal, Protocol`
- **Third-party / 第三方**: `import aiohttp`, `import regex as re`, `from tqdm.asyncio import tqdm`
