# test_mm_cache_stats.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/llm/test_mm_cache_stats.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers LLM entrypoint behavior. The file defines 1 test(s), 0 fixture(s), and 4 helper/class block(s) to validate this area. / [CN] 该文件覆盖LLM 入口行为。它定义了 1 个测试、0 个 fixture，以及 4 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L13)
```python
import logging

import pytest
import regex as re

from tests.entrypoints.openai.chat_completion.test_vision import TEST_IMAGE_ASSETS
from vllm import LLM
from vllm.entrypoints.chat_utils import ChatCompletionMessageParam
from vllm.v1.metrics import loggers as stat_loggers
from vllm.v1.metrics.reader import Counter, Metric
```
**EN:** Imports standard-library modules such as `logging`, third-party packages like `pytest`, `regex`, project helpers such as `tests.entrypoints.openai.chat_completion.test_vision.TEST_IMAGE_ASSETS`, `vllm.LLM`, `vllm.entrypoints.chat_utils.ChatCompletionMessageParam`.
**CN:** 导入标准库模块（如 `logging`）、第三方包（如 `pytest`、`regex`）、项目内辅助模块（如 `tests.entrypoints.openai.chat_completion.test_vision.TEST_IMAGE_ASSETS`、`vllm.LLM`、`vllm.entrypoints.chat_utils.ChatCompletionMessageParam`）。

### Helper / 辅助函数: _make_messages (L16-L27)
```python
def _make_messages(image_url: str) -> list[ChatCompletionMessageParam]:
    return [
        {
            "role": "user",
            "content": [
                {
                    "type": "image_url",
                    "image_url": {"url": image_url},
                },
            ],
        }
    ]
```
**EN:** This helper encapsulates reusable logic in `_make_messages`. Key inputs are `image_url`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_make_messages` 中。 关键输入包括 `image_url`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _get_counter_value (L30-L33)
```python
def _get_counter_value(metrics: list[Metric], name: str):
    metric = next(m for m in metrics if m.name == name)
    assert isinstance(metric, Counter)
    return metric.value
```
**EN:** This helper encapsulates reusable logic in `_get_counter_value`. Key inputs are `metrics`, `name`. It returns computed state or helper objects back to the caller. The main assertion is `isinstance(metric, Counter)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_get_counter_value` 中。 关键输入包括 `metrics`、`name`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `isinstance(metric, Counter)`。

### Helper / 辅助函数: _get_mm_cache_stats (L36-L40)
```python
def _get_mm_cache_stats(metrics: list[Metric]):
    mm_cache_queries = _get_counter_value(metrics, "vllm:mm_cache_queries")
    mm_cache_hits = _get_counter_value(metrics, "vllm:mm_cache_hits")

    return mm_cache_queries, mm_cache_hits
```
**EN:** This helper encapsulates reusable logic in `_get_mm_cache_stats`. Key inputs are `metrics`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_get_mm_cache_stats` 中。 关键输入包括 `metrics`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _get_mm_cache_log (L43-L54)
```python
def _get_mm_cache_log(llm: LLM, caplog_vllm: pytest.LogCaptureFixture) -> float:
    caplog_vllm.clear()
    with caplog_vllm.at_level(logging.INFO, logger=stat_loggers.__name__):
        llm.llm_engine.do_log_stats()

    assert len(caplog_vllm.records) == 1
    msg = caplog_vllm.records[0].getMessage()

    assert "MM cache hit rate" in msg
    match = re.search(r"MM cache hit rate: ([0-9.]+)%", msg)
    assert match is not None
    return float(match.group(1))
```
**EN:** This helper encapsulates reusable logic in `_get_mm_cache_log`. Key inputs are `llm`, `caplog_vllm`. It returns computed state or helper objects back to the caller. The main assertion is `len(caplog_vllm.records) == 1` and `'MM cache hit rate' in msg`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_get_mm_cache_log` 中。 关键输入包括 `llm`、`caplog_vllm`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `len(caplog_vllm.records) == 1` and `'MM cache hit rate' in msg`。

### Test / 测试: test_mm_cache_stats (L57-L97)
```python
@pytest.mark.parametrize("image_urls", [TEST_IMAGE_ASSETS[:2]], indirect=True)
@pytest.mark.parametrize("mm_processor_cache_type", ["lru", "shm"])
def test_mm_cache_stats(
    num_gpus_available,
    image_urls,
    mm_processor_cache_type,
    caplog_vllm,
):
    llm = LLM(
        model="llava-hf/llava-1.5-7b-hf",
        max_model_len=4096,
        max_num_seqs=5,
        enforce_eager=True,
        mm_processor_cache_type=mm_processor_cache_type,
        disable_log_stats=False,
        limit_mm_per_prompt={"image": 2},
    )

# ... 15 lines omitted for brevity ...

    llm.chat(_make_messages(image_urls[0]))
    assert _get_mm_cache_stats(llm.get_metrics()) == (4, 1)
    assert _get_mm_cache_log(llm, caplog_vllm) == pytest.approx(0.0)

    llm.chat(_make_messages(image_urls[1]))
    assert _get_mm_cache_stats(llm.get_metrics()) == (5, 1)
    assert _get_mm_cache_log(llm, caplog_vllm) == pytest.approx(0.0)
```
**EN:** This test validates `test_mm_cache_stats`. It uses parameterization over `image_urls`. Key inputs are `num_gpus_available`, `image_urls`, `mm_processor_cache_type`, `caplog_vllm`. It touches the core vLLM initialization or engine path directly. The main assertion is `_get_mm_cache_stats(llm.get_metrics()) == (1, 0)` and `_get_mm_cache_log(llm, caplog_vllm) == pytest.approx(0.0)`.
**CN:** 这个测试验证 `test_mm_cache_stats`。 它通过参数化组合 `image_urls`。 关键输入包括 `num_gpus_available`、`image_urls`、`mm_processor_cache_type`、`caplog_vllm`。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 核心断言是 `_get_mm_cache_stats(llm.get_metrics()) == (1, 0)` and `_get_mm_cache_log(llm, caplog_vllm) == pytest.approx(0.0)`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `logging`
- **Third-party / 第三方**: `pytest`, `regex`
- **Project / 项目内**: `tests.entrypoints.openai.chat_completion.test_vision.TEST_IMAGE_ASSETS`, `vllm.LLM`, `vllm.entrypoints.chat_utils.ChatCompletionMessageParam`, `vllm.v1.metrics.loggers`, `vllm.v1.metrics.reader.Counter`, `vllm.v1.metrics.reader.Metric`
