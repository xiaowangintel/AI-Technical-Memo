# simple_eval_common.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/infra/simple_eval_common.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises simple eval common behavior in the end-to-end infrastructure layer. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 端到端测试基础设施 中与 simple eval common 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Module header and comments
```python
# Adapted from https://github.com/openai/simple-evals/
```
**EN:** These leading lines typically contain a shebang, encoding note, or comments that frame the rest of the file.
**CN:** 这些起始行通常包含 shebang、编码说明或注释，用于为后续代码提供背景。

### Lines 2-2: Module docstring
```python
"""Common utilities for simple evaluations."""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 4-23: Imports and dependencies
```python
from __future__ import annotations

import logging
import os
import resource
import time
from collections import defaultdict
from dataclasses import dataclass, field
from multiprocessing.pool import ThreadPool
from typing import Any

import httpx
import jinja2
import numpy as np
import openai
import requests
from openai import OpenAI
from tqdm import tqdm

from .constants import MAX_RETRY_ATTEMPTS
```
**EN:** This block imports `__future__`, `logging`, `os`, `resource`, and 13 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 25-35: Module constants and configuration
```python
logger = logging.getLogger(__name__)

OPENAI_SYSTEM_MESSAGE_API = "You are a helpful assistant."
OPENAI_SYSTEM_MESSAGE_CHATGPT = (
    "You are ChatGPT, a large language model trained by OpenAI, based on the GPT-4 architecture."
    + "\nKnowledge cutoff: 2023-12\nCurrent date: 2024-04-01"
)


Message = dict[str, Any]  # keys role, content
MessageList = list[Message]
```
**EN:** This section defines module-level names such as `logger`, `OPENAI_SYSTEM_MESSAGE_API`, `OPENAI_SYSTEM_MESSAGE_CHATGPT`, `Message`, and 1 more, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 38-45: Class `SamplerBase`
```python
class SamplerBase:
    """
    Base class for defining a sampling model, which can be evaluated,
    or used as part of the grading process.
    """

    def __call__(self, message_list: MessageList) -> str:
        raise NotImplementedError()
```
**EN:** Class `SamplerBase` groups related state and behavior. It exposes 1 method(s) that implement the module's primary abstraction.
**CN:** 类 `SamplerBase` 将相关状态与行为封装在一起，提供 1 个方法来实现本模块的核心抽象。

### Lines 46-55: Class `EvalResult`
```python


@dataclass
class EvalResult:
    """Result of running an evaluation (usually consisting of many samples)."""

    score: float | None  # top-line metric
    metrics: dict[str, float] | None  # other metrics
    htmls: list[str]  # strings of valid HTML
    convos: list[MessageList]  # sampled conversations
```
**EN:** Class `EvalResult` groups related state and behavior. It exposes 0 method(s) that implement the module's primary abstraction.
**CN:** 类 `EvalResult` 将相关状态与行为封装在一起，提供 0 个方法来实现本模块的核心抽象。

### Lines 56-65: Class `SingleEvalResult`
```python


@dataclass
class SingleEvalResult:
    """Result of evaluating a single sample."""

    score: float | None
    metrics: dict[str, float] = field(default_factory=dict)
    html: str | None = None
    convo: MessageList | None = None  # sampled conversation
```
**EN:** Class `SingleEvalResult` groups related state and behavior. It exposes 0 method(s) that implement the module's primary abstraction.
**CN:** 类 `SingleEvalResult` 将相关状态与行为封装在一起，提供 0 个方法来实现本模块的核心抽象。

### Lines 68-74: Class `Eval`
```python
class Eval:
    """
    Base class for defining an evaluation.
    """

    def __call__(self, sampler: SamplerBase) -> EvalResult:
        raise NotImplementedError()
```
**EN:** Class `Eval` groups related state and behavior. It exposes 1 method(s) that implement the module's primary abstraction.
**CN:** 类 `Eval` 将相关状态与行为封装在一起，提供 1 个方法来实现本模块的核心抽象。

### Lines 77-84: Class `LargerHttpxClient`
```python
class LargerHttpxClient(httpx.Client):
    def __init__(self):
        timeout_config = httpx.Timeout(3600)
        limits = httpx.Limits(
            max_keepalive_connections=3600,
            max_connections=3600,
        )
        super().__init__(timeout=timeout_config, limits=limits)
```
**EN:** Class `LargerHttpxClient` groups related state and behavior. It exposes 1 method(s) that implement the module's primary abstraction.
**CN:** 类 `LargerHttpxClient` 将相关状态与行为封装在一起，提供 1 个方法来实现本模块的核心抽象。

### Lines 87-176: Class `ChatCompletionSampler`
```python
class ChatCompletionSampler(SamplerBase):
    """Sample from OpenAI's chat completion API."""

    def __init__(
        self,
        base_url: str | None = None,
        model: str | None = None,
        system_message: str | None = None,
        temperature: float = 0.0,
        reasoning_effort: str | None = None,
        max_tokens: int = 2048,
        extra_body: dict[str, Any] | None = None,
    ):
        self.client = OpenAI(base_url=base_url, http_client=LargerHttpxClient())

        if model is None:
            model = self.client.models.list().data[0].id

        self.model = model
        self.system_message = system_message
        self.temperature = temperature
        self.max_tokens = max_tokens
        self.reasoning_effort = reasoning_effort
        self.extra_body = extra_body
        self.image_format = "url"
        logger.debug(
            "ChatCompletionSampler: model=%s, temp=%.2f, max_tokens=%d",
            self.model,
            self.temperature,
            self.max_tokens,
        )

    def _handle_image(
        self,
        image: str,
        encoding: str = "base64",
        format: str = "png",
    ):
        new_image = {
            "type": "image_url",
            "image_url": {
                "url": f"data:image/{format};{encoding},{image}",
            },
        }
        return new_image

    def _handle_text(self, text: str):
        return {"type": "text", "text": text}

    def _pack_message(self, role: str, content: Any):
        return {"role": str(role), "content": content}

    def __call__(self, message_list: MessageList) -> str:
        if self.system_message:
            message_list = [
                self._pack_message("system", self.system_message)
            ] + message_list
        trial = 0
        while trial < MAX_RETRY_ATTEMPTS:
            try:
                response = self.client.chat.completions.create(
                    model=self.model,
                    messages=message_list,
                    temperature=self.temperature,
                    max_tokens=self.max_tokens,
                    reasoning_effort=self.reasoning_effort,
                    extra_body=self.extra_body,
                )
                return response.choices[0].message.content or ""
            except openai.BadRequestError as e:
                logger.warning("Bad request error: %s", e)
                return ""
            except Exception as e:
                exception_backoff = 2**trial  # exponential back off
                # Log first few retries at debug, later ones at warning
                log_fn = logger.warning if trial >= 3 else logger.debug
                log_fn(
                    "Request failed (retry %d/%d, backoff %ds): %s",
                    trial + 1,
                    MAX_RETRY_ATTEMPTS,
                    exception_backoff,
                    e,
                )
                time.sleep(exception_backoff)
                trial += 1
        logger.warning(
            "All retry attempts exhausted after %d retries, returning empty response",
            MAX_RETRY_ATTEMPTS,
        )
        return ""
```
**EN:** Class `ChatCompletionSampler` groups related state and behavior. It exposes 5 method(s) that implement the module's primary abstraction.
**CN:** 类 `ChatCompletionSampler` 将相关状态与行为封装在一起，提供 5 个方法来实现本模块的核心抽象。

### Lines 179-266: Module constants and configuration
```python
QUERY_TEMPLATE_MULTICHOICE = """
Answer the following multiple choice question. The last line of your response should be of the following format: 'Answer: $LETTER' (without quotes) where LETTER is one of ABCD. Think step by step before answering.

{Question}

A) {A}
B) {B}
C) {C}
D) {D}
""".strip()

ANSWER_PATTERN_MULTICHOICE = r"(?i)Answer\s*:\s*([A-D])"
ANSWER_PATTERN = r"(?i)Answer\s*:\s*([^\n]+)"


EQUALITY_TEMPLATE = r"""
Look at the following two expressions (answers to a math problem) and judge whether they are equivalent. Only perform trivial simplifications

Examples:

    Expression 1: $2x+3$
    Expression 2: $3+2x$

Yes

    Expression 1: 3/2
    Expression 2: 1.5

Yes

    Expression 1: $x^2+2x+1$
    Expression 2: $y^2+2y+1$

No

    Expression 1: $x^2+2x+1$
    Expression 2: $(x+1)^2$

Yes

    Expression 1: 3245/5
    Expression 2: 649

No
(these are actually equal, don't mark them equivalent if you need to do nontrivial simplifications)

    Expression 1: 2/(-3)
    Expression 2: -2/3

Yes
(trivial simplifications are allowed)

    Expression 1: 72 degrees
    Expression 2: 72

Yes
(give benefit of the doubt to units)

    Expression 1: 64
    Expression 2: 64 square feet

Yes
(give benefit of the doubt to units)

---

YOUR TASK


Respond with only "Yes" or "No" (without quotes). Do not include a rationale.

    Expression 1: %(expression1)s
    Expression 2: %(expression2)s
""".strip()


HTML_JINJA = """
<h3>Prompt conversation</h3>
{% for message in prompt_messages %}
{{ message_to_html(message) | safe }}
{% endfor %}
<h3>Sampled message</h3>
{{ message_to_html(next_message) | safe }}
<h3>Results</h3>
<p>Correct Answer: {{ correct_answer }}</p>
<p>Extracted Answer: {{ extracted_answer }}</p>
<p>Score: {{ score }}</p>
"""
```
**EN:** This section defines module-level names such as `QUERY_TEMPLATE_MULTICHOICE`, `ANSWER_PATTERN_MULTICHOICE`, `ANSWER_PATTERN`, `EQUALITY_TEMPLATE`, and 1 more, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 269-270: Helper function `format_multichoice_question`
```python
def format_multichoice_question(row):
    return QUERY_TEMPLATE_MULTICHOICE.format(**row)
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 273-276: Helper function `check_equality`
```python
def check_equality(sampler: SamplerBase, expr1: str, expr2: str):
    prompt = EQUALITY_TEMPLATE % {"expression1": expr1, "expression2": expr2}
    response = sampler([dict(content=prompt, role="user")])
    return (response or "").lower().strip() == "yes"
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 279-289: Helper function `_compute_stat`
```python
def _compute_stat(values: list, stat: str):
    if stat == "mean":
        return np.mean(values)
    elif stat == "std":
        return np.std(values)
    elif stat == "min":
        return np.min(values)
    elif stat == "max":
        return np.max(values)
    else:
        raise ValueError(f"Unknown {stat =}")
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 292-325: Helper function `aggregate_results`
```python
def aggregate_results(
    single_eval_results: list[SingleEvalResult],
    default_stats: tuple[str, ...] = ("mean", "std"),
    name2stats: dict[str, tuple[str, ...]] | None = None,
) -> EvalResult:
    """
    Aggregate results from multiple evaluations into a single EvalResult.
    """
    name2stats = name2stats or {}
    name2values = defaultdict(list)
    htmls = []
    convos = []
    for single_eval_result in single_eval_results:
        # Skip None results
        if single_eval_result is None:
            continue
        for name, value in single_eval_result.metrics.items():
            name2values[name].append(value)
        if single_eval_result.score is not None:
            name2values["score"].append(single_eval_result.score)
        htmls.append(single_eval_result.html)
        convos.append(single_eval_result.convo)
    final_metrics = {}
    for name, values in name2values.items():
        stats = name2stats.get(name, default_stats)
        for stat in stats:
            key = name if stat == "mean" else f"{name}:{stat}"
            final_metrics[key] = _compute_stat(values, stat)
    return EvalResult(
        score=final_metrics.pop("score", None),
        metrics=final_metrics,
        htmls=htmls,
        convos=convos,
    )
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 328-335: Helper function `map_with_progress`
```python
def map_with_progress(f: callable, xs: list[Any], num_threads: int) -> list[Any]:
    """Apply f to each element of xs, using a ThreadPool, and show progress."""
    # Use quiet progress bar that doesn't pollute logs
    if os.getenv("debug"):
        return list(map(f, tqdm(xs, total=len(xs), leave=False)))
    else:
        with ThreadPool(min(num_threads, len(xs))) as pool:
            return list(tqdm(pool.imap(f, xs), total=len(xs), leave=False))
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 338-353: Module constants and configuration
```python
jinja_env = jinja2.Environment(
    loader=jinja2.BaseLoader(),
    undefined=jinja2.StrictUndefined,
    autoescape=jinja2.select_autoescape(["html", "xml"]),
)
_message_template = """
<div class="message {{ role }}">
    <div class="role">
    {{ role }}
    {% if variant %}<span class="variant">({{ variant }})</span>{% endif %}
    </div>
    <div class="content">
    <pre>{{ content }}</pre>
    </div>
</div>
"""
```
**EN:** This section defines module-level names such as `jinja_env`, `_message_template`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 356-364: Helper function `message_to_html`
```python
def message_to_html(message: Message) -> str:
    """
    Generate HTML snippet (inside a <div>) for a message.
    """
    return jinja_env.from_string(_message_template).render(
        role=message["role"],
        content=message["content"],
        variant=message.get("variant", None),
    )
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 367-433: Module constants and configuration
```python
jinja_env.globals["message_to_html"] = message_to_html


_report_template = """<!DOCTYPE html>
<html>
    <head>
        <style>
            .message {
                padding: 8px 16px;
                margin-bottom: 8px;
                border-radius: 4px;
            }
            .message.user {
                background-color: #B2DFDB;
                color: #00695C;
            }
            .message.assistant {
                background-color: #B39DDB;
                color: #4527A0;
            }
            .message.system {
                background-color: #EEEEEE;
                color: #212121;
            }
            .role {
                font-weight: bold;
                margin-bottom: 4px;
            }
            .variant {
                color: #795548;
            }
            table, th, td {
                border: 1px solid black;
            }
            pre {
                white-space: pre-wrap;
            }
        </style>
    </head>
    <body>
    {% if metrics %}
    <h1>Metrics</h1>
    <table>
    <tr>
        <th>Metric</th>
        <th>Value</th>
    </tr>
    <tr>
        <td><b>Score</b></td>
        <td>{{ score | float | round(3) }}</td>
    </tr>
    {% for name, value in metrics.items() %}
    <tr>
        <td>{{ name }}</td>
        <td>{{ value }}</td>
    </tr>
    {% endfor %}
    </table>
    {% endif %}
    <h1>Examples</h1>
    {% for html in htmls %}
    {{ html | safe }}
    <hr>
    {% endfor %}
    </body>
</html>
"""
```
**EN:** This section defines module-level names such as `_report_template`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 436-444: Helper function `make_report`
```python
def make_report(eval_result: EvalResult) -> str:
    """
    Create a standalone HTML report from an EvalResult.
    """
    return jinja_env.from_string(_report_template).render(
        score=eval_result.score,
        metrics=eval_result.metrics,
        htmls=eval_result.htmls,
    )
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 447-453: Helper function `make_report_from_example_htmls`
```python
def make_report_from_example_htmls(htmls: list[str]):
    """
    Create a standalone HTML report from a list of example htmls
    """
    return jinja_env.from_string(_report_template).render(
        score=None, metrics={}, htmls=htmls
    )
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 456-483: Helper function `download_dataset`
```python
def download_dataset(path: str, url: str) -> None:
    """Download a dataset from URL to path."""
    logger.info("Downloading dataset from %s", url)
    try:
        response = requests.get(url, stream=True, timeout=30)
        response.raise_for_status()

        total_size = int(response.headers.get("content-length", 0))
        block_size = 8192

        with (
            open(path, "wb") as f,
            tqdm(
                desc="Downloading",
                total=total_size,
                unit="iB",
                unit_scale=True,
                unit_divisor=1024,
                leave=False,
            ) as progress_bar,
        ):
            for data in response.iter_content(block_size):
                size = f.write(data)
                progress_bar.update(size)

        logger.debug("Dataset saved to %s", path)
    except requests.RequestException as e:
        raise RuntimeError(f"Failed to download dataset: {e}") from e
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 486-495: Helper function `set_ulimit`
```python
def set_ulimit(target_soft_limit: int = 65535) -> None:
    """Set the file descriptor limit for parallel requests."""
    resource_type = resource.RLIMIT_NOFILE
    current_soft, current_hard = resource.getrlimit(resource_type)

    if current_soft < target_soft_limit:
        try:
            resource.setrlimit(resource_type, (target_soft_limit, current_hard))
        except ValueError as e:
            logger.debug("Could not set RLIMIT_NOFILE: %s", e)
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Test infrastructure orchestration / 测试基础设施编排
- Primary classes: `SamplerBase`, `EvalResult`, `SingleEvalResult`, `Eval`, and 2 more / 主要类：`SamplerBase`, `EvalResult`, `SingleEvalResult`, `Eval`, and 2 more
- Reusable functions: `format_multichoice_question`, `check_equality`, `_compute_stat`, `aggregate_results`, and 6 more / 可复用函数：`format_multichoice_question`, `check_equality`, `_compute_stat`, `aggregate_results`, and 6 more

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `collections`, `dataclasses`, `logging`, `multiprocessing.pool`, `os`, `resource`, `time`, `typing`
- **Third-party / 第三方**: `httpx`, `jinja2`, `numpy`, `openai`, `requests`, `tqdm`
- **Internal / 内部模块**: `.constants`
