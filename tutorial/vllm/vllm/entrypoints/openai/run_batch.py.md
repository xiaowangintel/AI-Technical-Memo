# run_batch.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `./vllm/entrypoints/openai/run_batch.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Implements batch workflow orchestration around the serving stack. Scope: OpenAI-compatible. / 实现围绕服务栈的批处理工作流编排。 作用域：OpenAI 兼容。

## Line-by-Line Analysis / 逐行分析
### Lines 4-70 — Imports and shared dependencies
```python
import asyncio
import contextlib
import json
import sys
import tempfile
from argparse import Namespace
from collections.abc import Awaitable, Callable
from http import HTTPStatus
from io import BytesIO, StringIO
from typing import Any, TypeAlias
from urllib.parse import urlparse

import aiohttp
import pybase64 as base64
import pydantic
import torch
from fastapi import UploadFile
from prometheus_client import start_http_server
from pydantic import Field, TypeAdapter, field_validator, model_validator
from pydantic_core.core_schema import ValidationInfo
from starlette.datastructures import State
from starlette.responses import JSONResponse
from tqdm import tqdm
from urllib3.util import parse_url

import vllm.envs as envs
...
)
from vllm.entrypoints.utils import create_error_response
from vllm.exceptions import VLLMValidationError
from vllm.logger import init_logger
from vllm.reasoning import ReasoningParserManager
from vllm.utils import random_uuid
from vllm.utils.argparse_utils import FlexibleArgumentParser
from vllm.version import __version__ as VLLM_VERSION
```
**EN:** This import block pulls in standard-library modules such as `asyncio`, `contextlib`, `json`, `sys`, `tempfile`, `argparse`, uses third-party packages like `aiohttp`, `pybase64`, `pydantic`, `torch`, `fastapi`, `prometheus_client`, depends on internal helpers such as `vllm.envs`, `vllm.config`, `vllm.engine.arg_utils`, `vllm.engine.protocol`, `vllm.entrypoints.openai.api_server`, `vllm.entrypoints.openai.chat_completion.protocol`.
**CN:** 该导入块引入 `asyncio`, `contextlib`, `json`, `sys`, `tempfile`, `argparse` 等标准库模块，使用 `aiohttp`, `pybase64`, `pydantic`, `torch`, `fastapi`, `prometheus_client` 等第三方库，依赖 `vllm.envs`, `vllm.config`, `vllm.engine.arg_utils`, `vllm.engine.protocol`, `vllm.entrypoints.openai.api_server`, `vllm.entrypoints.openai.chat_completion.protocol` 等 vLLM 内部模块。

### Lines 72-72 — Shared module state
```python
logger = init_logger(__name__)
```
**EN:** This block initializes `logger`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `logger`，为后续逻辑准备模块级常量或共享状态。

### Lines 75-103 — Class `BatchTranscriptionRequest`
```python
class BatchTranscriptionRequest(TranscriptionRequest):
    """
    Batch transcription request that uses file_url instead of file.

    This class extends TranscriptionRequest but replaces the file field
    with file_url to support batch processing from audio files written in JSON format.
    """

    file_url: str = Field(
        ...,
        description=(
            "Either a URL of the audio or a data URL with base64 encoded audio data. "
        ),
    )

    # Override file to be optional and unused for batch processing
    file: UploadFile | None = Field(default=None, exclude=True)  # type: ignore[assignment]

    @model_validator(mode="before")
    @classmethod
    def validate_no_file(cls, data: Any):
        """Ensure file field is not provided in batch requests."""
        if isinstance(data, dict) and "file" in data:
            raise VLLMValidationError(
                "The 'file' field is not supported in batch requests. "
                "Use 'file_url' instead.",
                parameter="file",
            )
        return data
```
**EN:** Class `BatchTranscriptionRequest` is introduced here. Its docstring describes the intent as: Batch transcription request that uses file_url instead of file.
**CN:** 这里定义类 `BatchTranscriptionRequest`。其文档字符串说明了该类的职责与使用方式。

### Lines 106-134 — Class `BatchTranslationRequest`
```python
class BatchTranslationRequest(TranslationRequest):
    """
    Batch translation request that uses file_url instead of file.

    This class extends TranslationRequest but replaces the file field
    with file_url to support batch processing from audio files written in JSON format.
    """

    file_url: str = Field(
        ...,
        description=(
            "Either a URL of the audio or a data URL with base64 encoded audio data. "
        ),
    )

    # Override file to be optional and unused for batch processing
    file: UploadFile | None = Field(default=None, exclude=True)  # type: ignore[assignment]

    @model_validator(mode="before")
    @classmethod
    def validate_no_file(cls, data: Any):
        """Ensure file field is not provided in batch requests."""
        if isinstance(data, dict) and "file" in data:
            raise VLLMValidationError(
                "The 'file' field is not supported in batch requests. "
                "Use 'file_url' instead.",
                parameter="file",
            )
        return data
```
**EN:** Class `BatchTranslationRequest` is introduced here. Its docstring describes the intent as: Batch translation request that uses file_url instead of file.
**CN:** 这里定义类 `BatchTranslationRequest`。其文档字符串说明了该类的职责与使用方式。

### Lines 137-144 — Module constants and state
```python
BatchRequestInputBody: TypeAlias = (
    ChatCompletionRequest
    | EmbeddingRequest
    | ScoreRequest
    | RerankRequest
    | BatchTranscriptionRequest
    | BatchTranslationRequest
)
```
**EN:** This block initializes `BatchRequestInputBody`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `BatchRequestInputBody`，为后续逻辑准备模块级常量或共享状态。

### Lines 147-186 — Class `BatchRequestInput`
```python
class BatchRequestInput(OpenAIBaseModel):
    """
    The per-line object of the batch input file.

    NOTE: Currently only the `/v1/chat/completions` endpoint is supported.
    """

    # A developer-provided per-request id that will be used to match outputs to
    # inputs. Must be unique for each request in a batch.
    custom_id: str

    # The HTTP method to be used for the request. Currently only POST is
    # supported.
    method: str

    # The OpenAI API relative URL to be used for the request. Currently
    # /v1/chat/completions is supported.
    url: str

    # The parameters of the request.
    body: BatchRequestInputBody

    @field_validator("body", mode="plain")
    @classmethod
    def check_type_for_url(cls, value: Any, info: ValidationInfo):
        # Use url to disambiguate models
        url: str = info.data["url"]
        if url == "/v1/chat/completions":
            return ChatCompletionRequest.model_validate(value)
        if url == "/v1/embeddings":
            return TypeAdapter(EmbeddingRequest).validate_python(value)
        if url.endswith("/score"):
            return TypeAdapter(ScoreRequest).validate_python(value)
        if url.endswith("/rerank"):
            return RerankRequest.model_validate(value)
        if url == "/v1/audio/transcriptions":
            return BatchTranscriptionRequest.model_validate(value)
        if url == "/v1/audio/translations":
            return BatchTranslationRequest.model_validate(value)
        return TypeAdapter(BatchRequestInputBody).validate_python(value)
```
**EN:** Class `BatchRequestInput` is introduced here. Its docstring describes the intent as: The per-line object of the batch input file.
**CN:** 这里定义类 `BatchRequestInput`。其文档字符串说明了该类的职责与使用方式。

### Lines 189-198 — Module constants and state
```python
AllResponse: TypeAlias = (
    ChatCompletionResponse
    | EmbeddingResponse
    | ScoreResponse
    | RerankResponse
    | TranscriptionResponse
    | TranscriptionResponseVerbose
    | TranslationResponse
    | TranslationResponseVerbose
)
```
**EN:** This block initializes `AllResponse`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `AllResponse`，为后续逻辑准备模块级常量或共享状态。

### Lines 201-209 — Class `BatchResponseData`
```python
class BatchResponseData(OpenAIBaseModel):
    # HTTP status code of the response.
    status_code: int = 200

    # An unique identifier for the API request.
    request_id: str

    # The body of the response.
    body: AllResponse | None = None
```
**EN:** Class `BatchResponseData` is defined here, extending `OpenAIBaseModel`, and groups behavior through methods like its internal helpers.
**CN:** 这里定义类 `BatchResponseData`，其职责是封装相关状态与行为，并通过 内部辅助方法 等方法组织逻辑。

### Lines 212-227 — Class `BatchRequestOutput`
```python
class BatchRequestOutput(OpenAIBaseModel):
    """
    The per-line object of the batch output and error files
    """

    id: str

    # A developer-provided per-request id that will be used to match outputs to
    # inputs.
    custom_id: str

    response: BatchResponseData | None

    # For requests that failed with a non-HTTP error, this will contain more
    # information on the cause of the failure.
    error: Any | None
```
**EN:** Class `BatchRequestOutput` is introduced here. Its docstring describes the intent as: The per-line object of the batch output and error files
**CN:** 这里定义类 `BatchRequestOutput`。其文档字符串说明了该类的职责与使用方式。

### Lines 231-272 — Class `BatchFrontendArgs`
```python
class BatchFrontendArgs(BaseFrontendArgs):
    """Arguments for the batch runner frontend."""

    input_file: str | None = None
    """The path or url to a single input file. Currently supports local file
    paths, or the http protocol (http or https). If a URL is specified,
    the file should be available via HTTP GET."""
    output_file: str | None = None
    """The path or url to a single output file. Currently supports
    local file paths, or web (http or https) urls. If a URL is specified,
    the file should be available via HTTP PUT."""
    output_tmp_dir: str | None = None
    """The directory to store the output file before uploading it
    to the output URL."""
    enable_metrics: bool = False
    """Enable Prometheus metrics"""
    host: str | None = None
    """Host name for the Prometheus metrics server
    (only needed if enable-metrics is set)."""
    port: int = 8000
    """Port number for the Prometheus metrics server
    (only needed if enable-metrics is set)."""
    url: str = "0.0.0.0"
    """[DEPRECATED] Host name for the Prometheus metrics server
    (only needed if enable-metrics is set). Use --host instead."""

    @classmethod
    def _customize_cli_kwargs(
        cls,
        frontend_kwargs: dict[str, Any],
    ) -> dict[str, Any]:
        frontend_kwargs = super()._customize_cli_kwargs(frontend_kwargs)

        frontend_kwargs["input_file"]["flags"] = ["-i"]
        frontend_kwargs["input_file"]["required"] = True
        frontend_kwargs["output_file"]["flags"] = ["-o"]
        frontend_kwargs["output_file"]["required"] = True

        frontend_kwargs["enable_metrics"]["action"] = "store_true"

        frontend_kwargs["url"]["deprecated"] = True
        return frontend_kwargs
```
**EN:** Class `BatchFrontendArgs` is introduced here. Its docstring describes the intent as: Arguments for the batch runner frontend.
**CN:** 这里定义类 `BatchFrontendArgs`。其文档字符串说明了该类的职责与使用方式。

### Lines 275-278 — Function `make_arg_parser`
```python
def make_arg_parser(parser: FlexibleArgumentParser):
    parser = BatchFrontendArgs.add_cli_args(parser)
    parser = AsyncEngineArgs.add_cli_args(parser)
    return parser
```
**EN:** This function `make_arg_parser` checks or normalizes inputs.
**CN:** 该函数 `make_arg_parser` 检查或规范化输入。

### Lines 281-296 — Function `parse_args`
```python
def parse_args():
    parser = FlexibleArgumentParser(description="vLLM OpenAI-Compatible batch runner.")
    args = make_arg_parser(parser).parse_args()

    # Backward compatibility: If --url is set, use it for host
    url_explicit = any(arg == "--url" or arg.startswith("--url=") for arg in sys.argv)
    host_explicit = any(
        arg == "--host" or arg.startswith("--host=") for arg in sys.argv
    )
    if url_explicit and hasattr(args, "url") and not host_explicit:
        args.host = args.url
        logger.warning_once(
            "Using --url for metrics is deprecated. Please use --host instead."
        )

    return args
```
**EN:** This function `parse_args` checks or normalizes inputs.
**CN:** 该函数 `parse_args` 检查或规范化输入。

### Lines 303-303 — Module constants
```python
_BAR_FORMAT = "{desc}: {percentage:3.0f}% Completed | {n_fmt}/{total_fmt} [{elapsed}<{remaining}, {rate_fmt}]\n"  # noqa: E501
```
**EN:** This block initializes `_BAR_FORMAT`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `_BAR_FORMAT`，为后续逻辑准备模块级常量或共享状态。

### Lines 306-330 — Class `BatchProgressTracker`
```python
class BatchProgressTracker:
    def __init__(self):
        self._total = 0
        self._pbar: tqdm | None = None

    def submitted(self):
        self._total += 1

    def completed(self):
        if self._pbar:
            self._pbar.update()

    def pbar(self) -> tqdm:
        enable_tqdm = (
            not torch.distributed.is_initialized() or torch.distributed.get_rank() == 0
        )
        self._pbar = tqdm(
            total=self._total,
            unit="req",
            desc="Running batch",
            mininterval=5,
            disable=not enable_tqdm,
            bar_format=_BAR_FORMAT,
        )
        return self._pbar
```
**EN:** Class `BatchProgressTracker` is defined here, as a standalone type, and groups behavior through methods like `__init__`, `submitted`, `completed`, `pbar`.
**CN:** 这里定义类 `BatchProgressTracker`，其职责是封装相关状态与行为，并通过 `__init__`、`submitted`、`completed`、`pbar` 等方法组织逻辑。

### Lines 333-340 — Function `read_file`
```python
async def read_file(path_or_url: str) -> str:
    if path_or_url.startswith("http://") or path_or_url.startswith("https://"):
        async with aiohttp.ClientSession() as session, session.get(path_or_url) as resp:
            resp.raise_for_status()
            return await resp.text()
    else:
        with open(path_or_url, encoding="utf-8") as f:
            return f.read()
```
**EN:** This async function `read_file` implements the `read file` step within the module flow.
**CN:** 该异步函数 `read_file` 实现了模块流程中的“readfile”步骤。

### Lines 343-355 — Function `write_local_file`
```python
async def write_local_file(
    output_path: str, batch_outputs: list[BatchRequestOutput]
) -> None:
    """
    Write the responses to a local file.
    output_path: The path to write the responses to.
    batch_outputs: The list of batch outputs to write.
    """
    # We should make this async, but as long as run_batch runs as a
    # standalone program, blocking the event loop won't affect performance.
    with open(output_path, "w", encoding="utf-8") as f:
        for o in batch_outputs:
            print(o.model_dump_json(), file=f)
```
**EN:** This async function `write_local_file` is documented as: Write the responses to a local file.
**CN:** 这里定义异步函数 `write_local_file`，其文档字符串说明了主要职责与调用约定。

### Lines 358-408 — Function `upload_data`
```python
async def upload_data(output_url: str, data_or_file: str, from_file: bool) -> None:
    """
    Upload a local file to a URL.
    output_url: The URL to upload the file to.
    data_or_file: Either the data to upload or the path to the file to upload.
    from_file: If True, data_or_file is the path to the file to upload.
    """
    # Timeout is a common issue when uploading large files.
    # We retry max_retries times before giving up.
    max_retries = 5
    # Number of seconds to wait before retrying.
    delay = 5

    for attempt in range(1, max_retries + 1):
        try:
            # We increase the timeout to 1000 seconds to allow
            # for large files (default is 300).
            async with aiohttp.ClientSession(
                timeout=aiohttp.ClientTimeout(total=1000)
            ) as session:
                if from_file:
                    with open(data_or_file, "rb") as file:
                        async with session.put(output_url, data=file) as response:
                            if response.status != 200:
                                raise Exception(
                                    f"Failed to upload file.\n"
                                    f"Status: {response.status}\n"
                                    f"Response: {response.text()}"
...
                    e,
                    delay,
                )
                await asyncio.sleep(delay)
            else:
                raise Exception(
                    f"Failed to upload data (attempt {attempt}). Error message: {str(e)}."  # noqa: E501
                ) from e
```
**EN:** This async function `upload_data` is documented as: Upload a local file to a URL.
**CN:** 这里定义异步函数 `upload_data`，其文档字符串说明了主要职责与调用约定。

### Lines 411-449 — Function `write_file`
```python
async def write_file(
    path_or_url: str, batch_outputs: list[BatchRequestOutput], output_tmp_dir: str
) -> None:
    """
    Write batch_outputs to a file or upload to a URL.
    path_or_url: The path or URL to write batch_outputs to.
    batch_outputs: The list of batch outputs to write.
    output_tmp_dir: The directory to store the output file before uploading it
    to the output URL.
    """
    if path_or_url.startswith("http://") or path_or_url.startswith("https://"):
        if output_tmp_dir is None:
            logger.info("Writing outputs to memory buffer")
            output_buffer = StringIO()
            for o in batch_outputs:
                print(o.model_dump_json(), file=output_buffer)
            output_buffer.seek(0)
            logger.info("Uploading outputs to %s", path_or_url)
            await upload_data(
                path_or_url,
                output_buffer.read().strip().encode("utf-8"),
                from_file=False,
            )
        else:
            # Write responses to a temporary file and then upload it to the URL.
            with tempfile.NamedTemporaryFile(
                mode="w",
                encoding="utf-8",
                dir=output_tmp_dir,
                prefix="tmp_batch_output_",
                suffix=".jsonl",
            ) as f:
                logger.info("Writing outputs to temporary local file %s", f.name)
                await write_local_file(f.name, batch_outputs)
                logger.info("Uploading outputs to %s", path_or_url)
                await upload_data(path_or_url, f.name, from_file=True)
    else:
        logger.info("Writing outputs to local file %s", path_or_url)
        await write_local_file(path_or_url, batch_outputs)
```
**EN:** This async function `write_file` is documented as: Write batch_outputs to a file or upload to a URL.
**CN:** 这里定义异步函数 `write_file`，其文档字符串说明了主要职责与调用约定。

### Lines 452-513 — Function `download_bytes_from_url`
```python
async def download_bytes_from_url(
    url: str,
    allowed_media_domains: list[str] | None = None,
) -> bytes:
    """
    Download data from a URL or decode from a data URL.

    Args:
        url: Either an HTTP/HTTPS URL or a data URL (data:...;base64,...)
        allowed_media_domains: If set, only HTTP/HTTPS URLs whose hostname
            is in this list are permitted. data: URLs are not subject to
            this restriction.

    Returns:
        Data as bytes
    """
    parsed = urlparse(url)

    # Handle data URLs (base64 encoded) - not subject to domain restrictions
    if parsed.scheme == "data":
        # Format: data:...;base64,<base64_data>
        if "," in url:
            header, data = url.split(",", 1)
            if "base64" in header:
                return base64.b64decode(data)
            else:
                raise ValueError(f"Unsupported data URL encoding: {header}")
        else:
...
                )
            return await resp.read()

    else:
        raise ValueError(
            f"Unsupported URL scheme: {parsed.scheme}. "
            "Supported schemes: http, https, data"
        )
```
**EN:** This async function `download_bytes_from_url` is documented as: Download data from a URL or decode from a data URL.
**CN:** 这里定义异步函数 `download_bytes_from_url`，其文档字符串说明了主要职责与调用约定。

### Lines 516-528 — Function `make_error_request_output`
```python
def make_error_request_output(
    request: BatchRequestInput, error_msg: str
) -> BatchRequestOutput:
    batch_output = BatchRequestOutput(
        id=f"vllm-{random_uuid()}",
        custom_id=request.custom_id,
        response=BatchResponseData(
            status_code=HTTPStatus.BAD_REQUEST,
            request_id=f"vllm-batch-{random_uuid()}",
        ),
        error=error_msg,
    )
    return batch_output
```
**EN:** This function `make_error_request_output` implements the `makes error request output` step within the module flow.
**CN:** 该函数 `make_error_request_output` 实现了模块流程中的“生成error请求output”步骤。

### Lines 531-534 — Function `make_async_error_request_output`
```python
async def make_async_error_request_output(
    request: BatchRequestInput, error_msg: str
) -> BatchRequestOutput:
    return make_error_request_output(request, error_msg)
```
**EN:** This async function `make_async_error_request_output` implements the `makes async error request output` step within the module flow.
**CN:** 该异步函数 `make_async_error_request_output` 实现了模块流程中的“生成asyncerror请求output”步骤。

### Lines 537-578 — Function `run_request`
```python
async def run_request(
    serving_engine_func: Callable,
    request: BatchRequestInput,
    tracker: BatchProgressTracker,
) -> BatchRequestOutput:
    try:
        response = await serving_engine_func(request.body)
    except Exception as e:
        response = create_error_response(e)

    if isinstance(response, JSONResponse):
        with contextlib.suppress(pydantic.ValidationError):
            response = TypeAdapter(AllResponse | ErrorResponse).validate_python(
                json.loads(response.body)
            )

    if isinstance(response, AllResponse):
        batch_output = BatchRequestOutput(
            id=f"vllm-{random_uuid()}",
            custom_id=request.custom_id,
            response=BatchResponseData(
                body=response, request_id=f"vllm-batch-{random_uuid()}"
            ),
            error=None,
        )
    elif isinstance(response, ErrorResponse):
        batch_output = BatchRequestOutput(
            id=f"vllm-{random_uuid()}",
            custom_id=request.custom_id,
            response=BatchResponseData(
                status_code=response.error.code,
                request_id=f"vllm-batch-{random_uuid()}",
            ),
            error=response,
        )
    else:
        batch_output = make_error_request_output(
            request, error_msg="Request must not be sent in stream mode"
        )

    tracker.completed()
    return batch_output
```
**EN:** This async function `run_request` starts or coordinates serving.
**CN:** 该异步函数 `run_request` 启动或协调服务流程。

### Lines 581-581 — Module constants and state
```python
WrapperFn: TypeAlias = Callable[[Callable], Callable]
```
**EN:** This block initializes `WrapperFn`, establishing reusable constants or shared state for the rest of the module.
**CN:** 该代码块初始化 `WrapperFn`，为后续逻辑准备模块级常量或共享状态。

### Lines 584-618 — Function `handle_endpoint_request`
```python
def handle_endpoint_request(
    request: BatchRequestInput,
    tracker: BatchProgressTracker,
    url_matcher: Callable[[str], bool],
    handler_getter: Callable[[], Callable | None],
    wrapper_fn: WrapperFn | None = None,
) -> Awaitable[BatchRequestOutput] | None:
    """
    Generic handler for endpoint requests.

    Args:
        request: The batch request input
        tracker: Progress tracker for the batch
        url_matcher: Function that takes a URL and returns True if it matches
        handler_getter: Function that returns the handler function or None
        wrapper_fn: Optional function to wrap the handler (e.g., for transcriptions)

    Returns:
        Awaitable[BatchRequestOutput] if the request was handled,
        None if URL didn't match
    """
    if not url_matcher(request.url):
        return None

    handler_fn = handler_getter()
    if handler_fn is None:
        error_msg = f"Model does not support endpoint: {request.url}"
        return make_async_error_request_output(request, error_msg=error_msg)

    # Apply wrapper if provided (e.g., for transcriptions/translations)
    if wrapper_fn is not None:
        handler_fn = wrapper_fn(handler_fn)

    tracker.submitted()
    return run_request(handler_fn, request, tracker)
```
**EN:** This function `handle_endpoint_request` is documented as: Generic handler for endpoint requests.
**CN:** 这里定义函数 `handle_endpoint_request`，其文档字符串说明了主要职责与调用约定。

### Lines 621-692 — Function `make_transcription_wrapper`
```python
def make_transcription_wrapper(
    is_translation: bool,
    allowed_media_domains: list[str] | None = None,
) -> WrapperFn:
    """
    Factory function to create a wrapper for transcription/translation handlers.
    The wrapper converts BatchTranscriptionRequest or BatchTranslationRequest
    to TranscriptionRequest or TranslationRequest and calls the appropriate handler.

    Args:
        is_translation: If True, process as translation; otherwise process
            as transcription
        allowed_media_domains: If set, only URLs from these domains are
            permitted for HTTP/HTTPS fetches.

    Returns:
        A function that takes a handler and returns a wrapped handler
    """

    def wrapper(handler_fn: Callable):
        async def transcription_wrapper(
            batch_request_body: (BatchTranscriptionRequest | BatchTranslationRequest),
        ) -> (
            TranscriptionResponse
            | TranscriptionResponseVerbose
            | TranslationResponse
            | TranslationResponseVerbose
            | ErrorResponse
...
                        type="BadRequestError",
                        code=HTTPStatus.BAD_REQUEST.value,
                    )
                )

        return transcription_wrapper

    return wrapper
```
**EN:** This function `make_transcription_wrapper` is documented as: Factory function to create a wrapper for transcription/translation handlers.
**CN:** 这里定义函数 `make_transcription_wrapper`，其文档字符串说明了主要职责与调用约定。

### Lines 695-787 — Function `build_endpoint_registry`
```python
async def build_endpoint_registry(
    engine_client: EngineClient,
    args: Namespace,
) -> dict[str, dict[str, Any]]:
    """
    Build the endpoint registry with all serving objects and handler configurations.

    Args:
        engine_client: The engine client
        args: Command line arguments

    Returns:
        Dictionary mapping endpoint keys to their configurations
    """
    supported_tasks = await engine_client.get_supported_tasks()
    logger.info("Supported tasks: %s", supported_tasks)

    # Create a state object to hold serving objects
    state = State()

    # Initialize all serving objects using init_app_state
    # This provides full functionality including chat template processing,
    # LoRA support, tool servers, etc.
    await init_app_state(engine_client, state, args, supported_tasks)

    # Get serving objects from state (defaulting to None if not set)
    openai_serving_chat = getattr(state, "openai_serving_chat", None)
    openai_serving_transcription = getattr(state, "openai_serving_transcription", None)
...
            "wrapper_fn": make_transcription_wrapper(
                is_translation=True,
                allowed_media_domains=allowed_media_domains,
            ),
        },
    }

    return endpoint_registry
```
**EN:** This async function `build_endpoint_registry` is documented as: Build the endpoint registry with all serving objects and handler configurations.
**CN:** 这里定义异步函数 `build_endpoint_registry`，其文档字符串说明了主要职责与调用约定。

### Lines 790-798 — Function `validate_run_batch_args`
```python
def validate_run_batch_args(args):
    valid_reasoning_parsers = ReasoningParserManager.list_registered()
    if (
        reasoning_parser := args.structured_outputs_config.reasoning_parser
    ) and reasoning_parser not in valid_reasoning_parsers:
        raise KeyError(
            f"invalid reasoning parser: {reasoning_parser} "
            f"(chose from {{ {','.join(valid_reasoning_parsers)} }})"
        )
```
**EN:** This function `validate_run_batch_args` starts or coordinates serving, checks or normalizes inputs.
**CN:** 该函数 `validate_run_batch_args` 启动或协调服务流程，检查或规范化输入。

### Lines 801-855 — Function `run_batch`
```python
async def run_batch(
    engine_client: EngineClient,
    args: Namespace,
) -> None:
    endpoint_registry = await build_endpoint_registry(
        engine_client=engine_client,
        args=args,
    )

    tracker = BatchProgressTracker()
    logger.info("Reading batch from %s...", args.input_file)

    # Submit all requests in the file to the engine "concurrently".
    response_futures: list[Awaitable[BatchRequestOutput]] = []
    for request_json in (await read_file(args.input_file)).strip().split("\n"):
        # Skip empty lines.
        request_json = request_json.strip()
        if not request_json:
            continue

        request = BatchRequestInput.model_validate_json(request_json)

        # Use the last segment of the URL as the endpoint key.
        # More advanced URL matching is done in url_matcher of endpoint_registry.
        endpoint_key = request.url.split("/")[-1]

        result = None
        if endpoint_key in endpoint_registry:
...
                    "for supported score/rerank versions.",
                )
            )

    with tracker.pbar():
        responses = await asyncio.gather(*response_futures)

    await write_file(args.output_file, responses, args.output_tmp_dir)
```
**EN:** This async function `run_batch` starts or coordinates serving, interacts with the model engine.
**CN:** 该异步函数 `run_batch` 启动或协调服务流程，与模型引擎交互。

### Lines 858-868 — Function `main`
```python
async def main(args: Namespace):
    from vllm.entrypoints.openai.api_server import build_async_engine_client
    from vllm.usage.usage_lib import UsageContext

    validate_run_batch_args(args)

    async with build_async_engine_client(
        args,
        usage_context=UsageContext.OPENAI_BATCH_RUNNER,
    ) as engine_client:
        await run_batch(engine_client, args)
```
**EN:** This async function `main` interacts with the model engine.
**CN:** 该异步函数 `main` 与模型引擎交互。

### Lines 871-885 — Executable entrypoint guard
```python
if __name__ == "__main__":
    args = parse_args()

    logger.info("vLLM batch processing API version %s", VLLM_VERSION)
    logger.info("args: %s", args)

    # Start the Prometheus metrics server. LLMEngine uses the Prometheus client
    # to publish metrics at the /metrics endpoint.
    if args.enable_metrics:
        logger.info("Prometheus metrics enabled")
        start_http_server(port=args.port, addr=args.host)
    else:
        logger.info("Prometheus metrics disabled")

    asyncio.run(main(args))
```
**EN:** This guard makes the module runnable as a script by parsing arguments and invoking its main execution path.
**CN:** 该保护分支允许模块以脚本方式直接运行，负责解析参数并进入主执行路径。

## Key Concepts / 关键概念
- FastAPI routing and application assembly / FastAPI 路由与应用组装
- Async request handling / 异步请求处理
- Schema validation with Pydantic / 使用 Pydantic 进行模式校验
- Structured request/response models / 结构化请求/响应模型
- CLI parsing / 命令行参数解析
- Engine-backed serving orchestration / 基于引擎的服务编排
- Metrics and observability / 指标与可观测性
- LoRA adapter support / LoRA 适配器支持

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `asyncio`, `contextlib`, `json`, `sys`, `tempfile`, `argparse`, `collections`, `http`, `io`, `typing`
- **Third-party / 第三方**: `aiohttp`, `pybase64`, `pydantic`, `torch`, `fastapi`, `prometheus_client`, `pydantic_core`, `starlette`, `tqdm`, `urllib3`
- **vLLM Internal / vLLM 内部**: `vllm.envs`, `vllm.config`, `vllm.engine.arg_utils`, `vllm.engine.protocol`, `vllm.entrypoints.openai.api_server`, `vllm.entrypoints.openai.chat_completion.protocol`, `vllm.entrypoints.openai.cli_args`, `vllm.entrypoints.openai.engine.protocol`, `vllm.entrypoints.pooling.embed.protocol`, `vllm.entrypoints.pooling.scoring.protocol`, `vllm.entrypoints.speech_to_text.transcription.protocol`, `vllm.entrypoints.speech_to_text.translation.protocol`
