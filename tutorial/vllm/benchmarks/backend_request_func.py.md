# backend_request_func.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/backend_request_func.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements backend request helpers, benchmark orchestration, centered around `AIOHTTP_TIMEOUT`, `RequestFuncInput`, `RequestFuncOutput`, `async_request_tgi`. / 实现与后端请求辅助逻辑、基准测试编排相关的逻辑，核心符号包括 `AIOHTTP_TIMEOUT`, `RequestFuncInput`, `RequestFuncOutput`, `async_request_tgi`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-15)
```python
import io
import json
import os
import sys
import time
import traceback
from dataclasses import dataclass, field

import aiohttp
import huggingface_hub.constants
from tqdm.asyncio import tqdm
from transformers import AutoTokenizer, PreTrainedTokenizer, PreTrainedTokenizerFast
```
**EN:** This block gathers standard-library helpers such as `io`, `json`, `os`, `sys`, `time`, `traceback`; third-party packages such as `aiohttp`, `huggingface_hub.constants`, `tqdm.asyncio`, `transformers`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `io`, `json`, `os`, `sys`, `time`, `traceback`；第三方依赖，如 `aiohttp`, `huggingface_hub.constants`, `tqdm.asyncio`, `transformers`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 20-20)
```python
AIOHTTP_TIMEOUT = aiohttp.ClientTimeout(total=6 * 60 * 60)
```
**EN:** This top-level block prepares shared state such as `AIOHTTP_TIMEOUT`. It uses `aiohttp.ClientTimeout` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `AIOHTTP_TIMEOUT`。它借助 `aiohttp.ClientTimeout` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Class `RequestFuncInput` (lines 24-36)
```python
class RequestFuncInput:
    prompt: str
    api_url: str
    prompt_len: int
    output_len: int
    model: str
    model_name: str | None = None
    logprobs: int | None = None
    extra_body: dict | None = None
    multi_modal_content: dict | list[dict] | None = None
    ignore_eos: bool = False
    language: str | None = None
    request_id: str | None = None
```
**EN:** Class `RequestFuncInput` packages shared state and related operations for this benchmark module. It extends `object` and exposes methods such as its methods.
**CN:** 类 `RequestFuncInput` 用于封装该基准模块的共享状态与相关操作。它继承自 `object`，并提供 its methods 等方法。

### Class `RequestFuncOutput` (lines 40-49)
```python
class RequestFuncOutput:
    generated_text: str = ""
    success: bool = False
    latency: float = 0.0
    output_tokens: int = 0
    ttft: float = 0.0  # Time to first token
    itl: list[float] = field(default_factory=list)  # list of inter-token latencies
    tpot: float = 0.0  # avg next-token latencies
    prompt_len: int = 0
    error: str = ""
```
**EN:** Class `RequestFuncOutput` packages shared state and related operations for this benchmark module. It extends `object` and exposes methods such as its methods.
**CN:** 类 `RequestFuncOutput` 用于封装该基准模块的共享状态与相关操作。它继承自 `object`，并提供 its methods 等方法。

### Async function `async_request_tgi` (lines 52-130)
```python
async def async_request_tgi(
    request_func_input: RequestFuncInput,
    pbar: tqdm | None = None,
) -> RequestFuncOutput:
    api_url = request_func_input.api_url
    assert api_url.endswith("generate_stream")

    async with aiohttp.ClientSession(
        trust_env=True, timeout=AIOHTTP_TIMEOUT
    ) as session:
        params = {
            "max_new_tokens": request_func_input.output_len,
            "do_sample": True,
            "temperature": 0.01,  # TGI does not accept 0.0 temperature.
            "top_p": 0.99,  # TGI does not accept 1.0 top_p.
            "truncate": request_func_input.prompt_len,
            "ignore_eos_token": request_func_input.ignore_eos,
        }
        payload = {
            "inputs": request_func_input.prompt,
            "parameters": params,
        }
        headers = None
        if request_func_input.request_id:
            headers = {"x-request-id": request_func_input.request_id}
        output = RequestFuncOutput()
        output.prompt_len = request_func_input.prompt_len
        if request_func_input.ignore_eos:
            output.output_tokens = request_func_input.output_len
        else:
            output.output_tokens = None

        ttft = 0.0
        st = time.perf_counter()
    # ... omitted for brevity ...
        except Exception:
            output.success = False
            exc_info = sys.exc_info()
            output.error = "".join(traceback.format_exception(*exc_info))

        if pbar:
            pbar.update(1)
        return output
```
**EN:** `async_request_tgi` implements backend request helpers. It mainly works with `request_func_input`, `pbar` and relies on `api_url.endswith`, `aiohttp.ClientSession`, `RequestFuncOutput`, `time.perf_counter`, `session.post`, `chunk_bytes.strip` plus iteration, branching, error handling and context management to move data through this part of the benchmark pipeline.
**CN:** `async_request_tgi` 负责实现后端请求辅助逻辑。 它主要处理 `request_func_input`, `pbar`，并结合 `api_url.endswith`, `aiohttp.ClientSession`, `RequestFuncOutput`, `time.perf_counter`, `session.post`, `chunk_bytes.strip` 以及 循环迭代、条件分支、异常处理、上下文管理 来完成这一段基准测试流程。

### Async function `async_request_trt_llm` (lines 133-201)
```python
async def async_request_trt_llm(
    request_func_input: RequestFuncInput,
    pbar: tqdm | None = None,
) -> RequestFuncOutput:
    api_url = request_func_input.api_url
    assert api_url.endswith("generate_stream")

    async with aiohttp.ClientSession(
        trust_env=True, timeout=AIOHTTP_TIMEOUT
    ) as session:
        payload = {
            "accumulate_tokens": True,
            "text_input": request_func_input.prompt,
            "temperature": 0.0,
            "top_p": 1.0,
            "max_tokens": request_func_input.output_len,
            "stream": True,
        }
        if request_func_input.ignore_eos:
            payload["min_length"] = request_func_input.output_len
        headers = None
        if request_func_input.request_id:
            headers = {"x-request-id": request_func_input.request_id}
        output = RequestFuncOutput()
        output.prompt_len = request_func_input.prompt_len

        ttft = 0.0
        st = time.perf_counter()
        most_recent_timestamp = st
        try:
            async with session.post(
                url=api_url, json=payload, headers=headers
            ) as response:
                if response.status == 200:
    # ... omitted for brevity ...
        except Exception:
            output.success = False
            exc_info = sys.exc_info()
            output.error = "".join(traceback.format_exception(*exc_info))

        if pbar:
            pbar.update(1)
        return output
```
**EN:** `async_request_trt_llm` implements backend request helpers. It mainly works with `request_func_input`, `pbar` and relies on `api_url.endswith`, `aiohttp.ClientSession`, `RequestFuncOutput`, `time.perf_counter`, `session.post`, `chunk_bytes.strip` plus iteration, branching, error handling and context management to move data through this part of the benchmark pipeline.
**CN:** `async_request_trt_llm` 负责实现后端请求辅助逻辑。 它主要处理 `request_func_input`, `pbar`，并结合 `api_url.endswith`, `aiohttp.ClientSession`, `RequestFuncOutput`, `time.perf_counter`, `session.post`, `chunk_bytes.strip` 以及 循环迭代、条件分支、异常处理、上下文管理 来完成这一段基准测试流程。

### Async function `async_request_deepspeed_mii` (lines 204-264)
```python
async def async_request_deepspeed_mii(
    request_func_input: RequestFuncInput,
    pbar: tqdm | None = None,
) -> RequestFuncOutput:
    api_url = request_func_input.api_url
    assert api_url.endswith(("completions", "profile")), (
        "OpenAI Completions API URL must end with 'completions' or 'profile'."
    )

    async with aiohttp.ClientSession(
        trust_env=True, timeout=AIOHTTP_TIMEOUT
    ) as session:
        payload = {
            "model": request_func_input.model,
            "prompt": request_func_input.prompt,
            "max_tokens": request_func_input.output_len,
            "temperature": 0.01,  # deepspeed-mii does not accept 0.0 temp.
            "top_p": 1.0,
        }
        headers = {"Authorization": f"Bearer {os.environ.get('OPENAI_API_KEY')}"}
        if request_func_input.request_id:
            headers["x-request-id"] = request_func_input.request_id

        output = RequestFuncOutput()
        output.prompt_len = request_func_input.prompt_len

        # NOTE: DeepSpeed-MII doesn't support streaming as of Jan 28 2024,
        # will use 0 as placeholder.
        # See https://github.com/microsoft/DeepSpeed-MII/pull/311
        output.ttft = 0

        st = time.perf_counter()
        try:
            async with session.post(
    # ... omitted for brevity ...
        except Exception:
            output.success = False
            exc_info = sys.exc_info()
            output.error = "".join(traceback.format_exception(*exc_info))

        if pbar:
            pbar.update(1)
        return output
```
**EN:** `async_request_deepspeed_mii` implements backend request helpers. It mainly works with `request_func_input`, `pbar` and relies on `api_url.endswith`, `aiohttp.ClientSession`, `os.environ.get`, `RequestFuncOutput`, `time.perf_counter`, `session.post` plus branching, error handling and context management to move data through this part of the benchmark pipeline.
**CN:** `async_request_deepspeed_mii` 负责实现后端请求辅助逻辑。 它主要处理 `request_func_input`, `pbar`，并结合 `api_url.endswith`, `aiohttp.ClientSession`, `os.environ.get`, `RequestFuncOutput`, `time.perf_counter`, `session.post` 以及 条件分支、异常处理、上下文管理 来完成这一段基准测试流程。

### Async function `async_request_openai_completions` (lines 267-364)
```python
async def async_request_openai_completions(
    request_func_input: RequestFuncInput,
    pbar: tqdm | None = None,
) -> RequestFuncOutput:
    api_url = request_func_input.api_url
    assert api_url.endswith(("completions", "profile")), (
        "OpenAI Completions API URL must end with 'completions' or 'profile'."
    )

    async with aiohttp.ClientSession(
        trust_env=True, timeout=AIOHTTP_TIMEOUT
    ) as session:
        payload = {
            "model": request_func_input.model_name
            if request_func_input.model_name
            else request_func_input.model,
            "prompt": request_func_input.prompt,
            "temperature": 0.0,
            "repetition_penalty": 1.0,
            "max_tokens": request_func_input.output_len,
            "logprobs": request_func_input.logprobs,
            "stream": True,
            "stream_options": {
                "include_usage": True,
            },
        }
        if request_func_input.ignore_eos:
            payload["ignore_eos"] = request_func_input.ignore_eos
        if request_func_input.extra_body:
            payload.update(request_func_input.extra_body)
        headers = {"Authorization": f"Bearer {os.environ.get('OPENAI_API_KEY')}"}
        if request_func_input.request_id:
            headers["x-request-id"] = request_func_input.request_id

    # ... omitted for brevity ...
        except Exception:
            output.success = False
            exc_info = sys.exc_info()
            output.error = "".join(traceback.format_exception(*exc_info))

    if pbar:
        pbar.update(1)
    return output
```
**EN:** `async_request_openai_completions` implements backend request helpers. It mainly works with `request_func_input`, `pbar` and relies on `api_url.endswith`, `aiohttp.ClientSession`, `payload.update`, `os.environ.get`, `RequestFuncOutput`, `time.perf_counter` plus iteration, branching, error handling and context management to move data through this part of the benchmark pipeline.
**CN:** `async_request_openai_completions` 负责实现后端请求辅助逻辑。 它主要处理 `request_func_input`, `pbar`，并结合 `api_url.endswith`, `aiohttp.ClientSession`, `payload.update`, `os.environ.get`, `RequestFuncOutput`, `time.perf_counter` 以及 循环迭代、条件分支、异常处理、上下文管理 来完成这一段基准测试流程。

### Async function `async_request_openai_chat_completions` (lines 367-473)
```python
async def async_request_openai_chat_completions(
    request_func_input: RequestFuncInput,
    pbar: tqdm | None = None,
) -> RequestFuncOutput:
    api_url = request_func_input.api_url
    assert api_url.endswith(("chat/completions", "profile")), (
        "OpenAI Chat Completions API URL must end with 'chat/completions'."
    )

    async with aiohttp.ClientSession(
        trust_env=True, timeout=AIOHTTP_TIMEOUT
    ) as session:
        content = [{"type": "text", "text": request_func_input.prompt}]
        if request_func_input.multi_modal_content:
            mm_content = request_func_input.multi_modal_content
            if isinstance(mm_content, list):
                content.extend(mm_content)
            elif isinstance(mm_content, dict):
                content.append(mm_content)
            else:
                raise TypeError(
                    "multi_modal_content must be a dict or list[dict] for openai-chat"
                )
        payload = {
            "model": request_func_input.model_name
            if request_func_input.model_name
            else request_func_input.model,
            "messages": [
                {"role": "user", "content": content},
            ],
            "temperature": 0.0,
            "max_completion_tokens": request_func_input.output_len,
            "stream": True,
            "stream_options": {
    # ... omitted for brevity ...
        except Exception:
            output.success = False
            exc_info = sys.exc_info()
            output.error = "".join(traceback.format_exception(*exc_info))

    if pbar:
        pbar.update(1)
    return output
```
**EN:** `async_request_openai_chat_completions` implements backend request helpers. It mainly works with `request_func_input`, `pbar` and relies on `api_url.endswith`, `aiohttp.ClientSession`, `isinstance`, `content.extend`, `content.append`, `TypeError` plus iteration, branching, error handling and context management to move data through this part of the benchmark pipeline.
**CN:** `async_request_openai_chat_completions` 负责实现后端请求辅助逻辑。 它主要处理 `request_func_input`, `pbar`，并结合 `api_url.endswith`, `aiohttp.ClientSession`, `isinstance`, `content.extend`, `content.append`, `TypeError` 以及 循环迭代、条件分支、异常处理、上下文管理 来完成这一段基准测试流程。

### Async function `async_request_openai_audio` (lines 476-585)
```python
async def async_request_openai_audio(
    request_func_input: RequestFuncInput,
    pbar: tqdm | None = None,
) -> RequestFuncOutput:
    # Lazy import without PlaceholderModule to avoid vllm dep.
    import soundfile

    api_url = request_func_input.api_url
    assert api_url.endswith(("transcriptions", "translations")), (
        "OpenAI Chat Completions API URL must end with 'transcriptions' "
    )
    "or `translations`."

    async with aiohttp.ClientSession(
        trust_env=True, timeout=AIOHTTP_TIMEOUT
    ) as session:
        content = [{"type": "text", "text": request_func_input.prompt}]
        payload = {
            "model": request_func_input.model_name
            if request_func_input.model_name
            else request_func_input.model,
            "temperature": 0.0,
            "max_completion_tokens": request_func_input.output_len,
            "stream": True,
            "language": "en",
            # Flattened due to multipart/form-data
            "stream_include_usage": True,
            "stream_continuous_usage_stats": True,
        }
        if request_func_input.extra_body:
            payload.update(request_func_input.extra_body)
        headers = {
            "Authorization": f"Bearer {os.environ.get('OPENAI_API_KEY')}",
        }
    # ... omitted for brevity ...
            except Exception:
                output.success = False
                exc_info = sys.exc_info()
                output.error = "".join(traceback.format_exception(*exc_info))

        if pbar:
            pbar.update(1)
        return output
```
**EN:** `async_request_openai_audio` implements backend request helpers. It mainly works with `request_func_input`, `pbar` and relies on `api_url.endswith`, `aiohttp.ClientSession`, `payload.update`, `os.environ.get`, `io.BytesIO`, `soundfile.write` plus iteration, branching, error handling and context management to move data through this part of the benchmark pipeline.
**CN:** `async_request_openai_audio` 负责实现后端请求辅助逻辑。 它主要处理 `request_func_input`, `pbar`，并结合 `api_url.endswith`, `aiohttp.ClientSession`, `payload.update`, `os.environ.get`, `io.BytesIO`, `soundfile.write` 以及 循环迭代、条件分支、异常处理、上下文管理 来完成这一段基准测试流程。

### Function `get_model` (lines 588-604)
```python
def get_model(pretrained_model_name_or_path: str) -> str:
    if os.getenv("VLLM_USE_MODELSCOPE", "False").lower() == "true":
        from modelscope import snapshot_download

        from vllm.model_executor.model_loader.weight_utils import get_lock

        # Use file lock to prevent multiple processes from
        # downloading the same model weights at the same time.
        with get_lock(pretrained_model_name_or_path):
            model_path = snapshot_download(
                model_id=pretrained_model_name_or_path,
                local_files_only=huggingface_hub.constants.HF_HUB_OFFLINE,
                ignore_file_pattern=[".*.pt", ".*.safetensors", ".*.bin"],
            )

            return model_path
    return pretrained_model_name_or_path
```
**EN:** `get_model` implements a helper used by `backend_request_func.py`. It mainly works with `pretrained_model_name_or_path` and relies on `os.getenv.lower`, `os.getenv`, `get_lock`, `snapshot_download` plus branching and context management to move data through this part of the benchmark pipeline.
**CN:** `get_model` 负责实现 `backend_request_func.py` 使用的辅助逻辑。 它主要处理 `pretrained_model_name_or_path`，并结合 `os.getenv.lower`, `os.getenv`, `get_lock`, `snapshot_download` 以及 条件分支、上下文管理 来完成这一段基准测试流程。

### Function `get_tokenizer` (lines 607-636)
```python
def get_tokenizer(
    pretrained_model_name_or_path: str,
    tokenizer_mode: str = "auto",
    trust_remote_code: bool = False,
    **kwargs,
) -> PreTrainedTokenizer | PreTrainedTokenizerFast:
    if pretrained_model_name_or_path is not None and not os.path.exists(
        pretrained_model_name_or_path
    ):
        pretrained_model_name_or_path = get_model(pretrained_model_name_or_path)
    if tokenizer_mode == "slow":
        if kwargs.get("use_fast", False):
            raise ValueError("Cannot use the fast tokenizer in slow tokenizer mode.")
        kwargs["use_fast"] = False
    if tokenizer_mode == "mistral":
        try:
            from vllm.tokenizers.mistral import MistralTokenizer
        except ImportError as e:
            raise ImportError(
                "MistralTokenizer requires vllm package.\n"
                "Please install it with `pip install vllm` "
                "to use mistral tokenizer mode."
            ) from e
        return MistralTokenizer.from_pretrained(str(pretrained_model_name_or_path))
    else:
        return AutoTokenizer.from_pretrained(
            pretrained_model_name_or_path,
            trust_remote_code=trust_remote_code,
            **kwargs,
        )
```
**EN:** `get_tokenizer` implements a helper used by `backend_request_func.py`. It mainly works with `pretrained_model_name_or_path`, `tokenizer_mode`, `trust_remote_code`, `**kwargs` and relies on `os.path.exists`, `get_model`, `kwargs.get`, `ValueError`, `ImportError`, `MistralTokenizer.from_pretrained` plus branching and error handling to move data through this part of the benchmark pipeline.
**CN:** `get_tokenizer` 负责实现 `backend_request_func.py` 使用的辅助逻辑。 它主要处理 `pretrained_model_name_or_path`, `tokenizer_mode`, `trust_remote_code`, `**kwargs`，并结合 `os.path.exists`, `get_model`, `kwargs.get`, `ValueError`, `ImportError`, `MistralTokenizer.from_pretrained` 以及 条件分支、异常处理 来完成这一段基准测试流程。

### Top-level setup (lines 639-651)
```python
ASYNC_REQUEST_FUNCS = {
    "tgi": async_request_tgi,
    "vllm": async_request_openai_completions,
    "lmdeploy": async_request_openai_completions,
    "deepspeed-mii": async_request_deepspeed_mii,
    "openai": async_request_openai_completions,
    "openai-chat": async_request_openai_chat_completions,
    "openai-audio": async_request_openai_audio,
    "tensorrt-llm": async_request_trt_llm,
    "scalellm": async_request_openai_completions,
    "sglang": async_request_openai_completions,
    "llama.cpp": async_request_openai_completions,
}
```
**EN:** This top-level block prepares shared state such as `ASYNC_REQUEST_FUNCS`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `ASYNC_REQUEST_FUNCS`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

## Key Concepts / 关键概念
- **EN:** Serving path benchmarking: the script interacts with HTTP services, servers, or request proxies.
- **CN:** 服务路径基准：脚本会与 HTTP 服务、服务器或请求代理交互。
- **EN:** Dataset handling: benchmark inputs are loaded, normalized, or synthesized before execution.
- **CN:** 数据集处理：基准输入会在执行前被加载、规范化或合成。
- **EN:** Performance metrics: the script emphasizes latency/throughput measurement rather than model quality.
- **CN:** 性能指标：该脚本更强调延迟/吞吐量测量，而不是模型质量。

## Dependencies / 依赖关系
- **EN:** Standard library: `io`, `json`, `os`, `sys`, `time`, `traceback`.
- **CN:** 标准库依赖：`io`, `json`, `os`, `sys`, `time`, `traceback`。
- **EN:** Third-party packages: `aiohttp`, `huggingface_hub.constants`, `tqdm.asyncio`, `transformers`.
- **CN:** 第三方依赖：`aiohttp`, `huggingface_hub.constants`, `tqdm.asyncio`, `transformers`。
- **EN:** Internal modules: none.
- **CN:** 内部模块：无。
