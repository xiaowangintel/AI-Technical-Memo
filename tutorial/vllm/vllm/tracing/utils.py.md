# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tracing/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `utils`-related logic centered around `SpanAttributes`, `LoadingSpanAttributes`, `contains_trace_headers`, `extract_trace_headers`. / 实现与 `utils` 相关的逻辑，核心符号包括 `SpanAttributes`, `LoadingSpanAttributes`, `contains_trace_headers`, `extract_trace_headers`。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-12)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from collections.abc import Mapping

from vllm.logger import init_logger
from vllm.utils.func_utils import run_once

logger = init_logger(__name__)

# Standard W3C headers used for context propagation
TRACE_HEADERS = ["traceparent", "tracestate"]
```
**EN:** Sets up the module with standard-library support such as `collections.abc`, vLLM modules such as `vllm.logger`, `vllm.utils.func_utils`. It prepares the symbols later used by `SpanAttributes`, `LoadingSpanAttributes`, `contains_trace_headers`, `extract_trace_headers`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.logger`, `vllm.utils.func_utils` 等 vLLM 内部依赖。 这些准备工作为后续的 `SpanAttributes`, `LoadingSpanAttributes`, `contains_trace_headers`, `extract_trace_headers` 提供上下文。

### SpanAttributes (lines 15-45)
```python
class SpanAttributes:
    """
    Standard attributes for spans.

    These are largely based on OpenTelemetry Semantic Conventions but are defined
    here as constants so they can be used by any backend or logger.
    """

    # Attribute names copied from OTel semantic conventions to avoid version conflicts
    GEN_AI_USAGE_COMPLETION_TOKENS = "gen_ai.usage.completion_tokens"
    GEN_AI_USAGE_PROMPT_TOKENS = "gen_ai.usage.prompt_tokens"
    GEN_AI_REQUEST_MAX_TOKENS = "gen_ai.request.max_tokens"
    GEN_AI_REQUEST_TOP_P = "gen_ai.request.top_p"
    GEN_AI_REQUEST_TEMPERATURE = "gen_ai.request.temperature"
    GEN_AI_RESPONSE_MODEL = "gen_ai.response.model"

    # Custom attributes added until they are standardized
    GEN_AI_REQUEST_ID = "gen_ai.request.id"
    GEN_AI_REQUEST_N = "gen_ai.request.n"
    GEN_AI_USAGE_NUM_SEQUENCES = "gen_ai.usage.num_sequences"
    GEN_AI_LATENCY_TIME_IN_QUEUE = "gen_ai.latency.time_in_queue"
    GEN_AI_LATENCY_TIME_TO_FIRST_TOKEN = "gen_ai.latency.time_to_first_token"
    GEN_AI_LATENCY_E2E = "gen_ai.latency.e2e"
    GEN_AI_LATENCY_TIME_IN_SCHEDULER = "gen_ai.latency.time_in_scheduler"

    # Latency breakdowns
    GEN_AI_LATENCY_TIME_IN_MODEL_FORWARD = "gen_ai.latency.time_in_model_forward"
    GEN_AI_LATENCY_TIME_IN_MODEL_EXECUTE = "gen_ai.latency.time_in_model_execute"
    GEN_AI_LATENCY_TIME_IN_MODEL_PREFILL = "gen_ai.latency.time_in_model_prefill"
    GEN_AI_LATENCY_TIME_IN_MODEL_DECODE = "gen_ai.latency.time_in_model_decode"
    GEN_AI_LATENCY_TIME_IN_MODEL_INFERENCE = "gen_ai.latency.time_in_model_inference"
```
**EN:** `SpanAttributes`: Standard attributes for spans.
**CN:** `SpanAttributes` 是该文件中的核心类，用于封装与 `SpanAttributes` 相关的状态和行为。

### LoadingSpanAttributes (lines 48-54)
```python
class LoadingSpanAttributes:
    """Custom attributes for code-level tracing (file, line number)."""

    CODE_NAMESPACE = "code.namespace"
    CODE_FUNCTION = "code.function"
    CODE_FILEPATH = "code.filepath"
    CODE_LINENO = "code.lineno"
```
**EN:** `LoadingSpanAttributes`: Custom attributes for code-level tracing (file, line number).
**CN:** `LoadingSpanAttributes` 是该文件中的核心类，用于封装与 `LoadingSpanAttributes` 相关的状态和行为。

### contains_trace_headers (lines 57-59)
```python
def contains_trace_headers(headers: Mapping[str, str]) -> bool:
    """Check if the provided headers dictionary contains trace context."""
    return any(h in headers for h in TRACE_HEADERS)
```
**EN:** `contains_trace_headers`: Check if the provided headers dictionary contains trace context. It mainly works with `headers`. Inside the body, it relies on `any` to complete the main steps.
**CN:** `contains_trace_headers` 负责实现本模块使用的辅助逻辑。 它主要处理 `headers` 等参数。 实现过程中会调用 `any` 等函数完成关键步骤。

### extract_trace_headers (lines 62-67)
```python
def extract_trace_headers(headers: Mapping[str, str]) -> Mapping[str, str]:
    """
    Extract only trace-related headers from a larger header dictionary.
    Useful for logging or passing context to a non-OTel client.
    """
    return {h: headers[h] for h in TRACE_HEADERS if h in headers}
```
**EN:** `extract_trace_headers`: Extract only trace-related headers from a larger header dictionary. It mainly works with `headers`.
**CN:** `extract_trace_headers` 负责实现本模块使用的辅助逻辑。 它主要处理 `headers` 等参数。

### log_tracing_disabled_warning (lines 71-72)
```python
def log_tracing_disabled_warning() -> None:
    logger.warning("Received a request with trace context but tracing is disabled")
```
**EN:** `log_tracing_disabled_warning` implements helper logic used by this module. Inside the body, it relies on `logger.warning` to complete the main steps.
**CN:** `log_tracing_disabled_warning` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `logger.warning` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`SpanAttributes`**: Core class that organizes module behavior. / **`SpanAttributes`**：组织模块行为的核心类。
- **`LoadingSpanAttributes`**: Core class that organizes module behavior. / **`LoadingSpanAttributes`**：组织模块行为的核心类。
- **`contains_trace_headers`**: Key helper or entry point in this file. / **`contains_trace_headers`**：本文件中的关键辅助函数或入口。
- **`extract_trace_headers`**: Key helper or entry point in this file. / **`extract_trace_headers`**：本文件中的关键辅助函数或入口。
- **`log_tracing_disabled_warning`**: Key helper or entry point in this file. / **`log_tracing_disabled_warning`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: collections.abc
- **Third-party / 第三方**: None / 无
- **Internal vLLM / vLLM 内部依赖**: vllm.logger, vllm.utils.func_utils
