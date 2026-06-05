# test_custom_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/logits_processors/test_custom_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `custom offline` behavior and regressions in the v1 stack. / 验证 v1 栈中 `custom offline` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-26)
```python
from typing import Any

import pytest

from tests.utils import create_new_process_for_each_test, set_random_seed
from tests.v1.logits_processors.utils import (
    DUMMY_LOGITPROC_ARG,
    DUMMY_LOGITPROC_FQCN,
    MAX_TOKENS,
    MODEL_NAME,
    POOLING_MODEL_NAME,
    TEMP_GREEDY,
    CustomLogitprocSource,
    DummyLogitsProcessor,
    WrappedPerReqLogitsProcessor,
    prompts,
    setup_fake_entrypoint,
)
from vllm import LLM, SamplingParams
from vllm.v1.sample.logits_processor import (
    STR_POOLING_REJECTS_LOGITSPROCS,
    STR_SPEC_DEC_REJECTS_LOGITSPROCS,
    LogitsProcessor,
)
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest`. vLLM modules under test include `vllm, vllm.v1.sample.logits_processor`. Local helpers come from `tests.utils, tests.v1.logits_processors.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest`。 被测试的 vLLM 模块包括 `vllm, vllm.v1.sample.logits_processor`。 本地测试辅助逻辑来自 `tests.utils, tests.v1.logits_processors.utils`。

### Module state / 模块级状态 (lines 29-42)
```python
sampling_params_list = [
    SamplingParams(
        temperature=TEMP_GREEDY,
        max_tokens=MAX_TOKENS,
        extra_args={DUMMY_LOGITPROC_ARG: 128},
    ),
    SamplingParams(temperature=TEMP_GREEDY, max_tokens=MAX_TOKENS),
    SamplingParams(
        temperature=TEMP_GREEDY,
        max_tokens=MAX_TOKENS,
        extra_args={DUMMY_LOGITPROC_ARG: 67},
    ),
    SamplingParams(temperature=TEMP_GREEDY, max_tokens=MAX_TOKENS),
]
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `sampling_params_list`. Shared setup calls include `SamplingParams`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`sampling_params_list`。 共享初始化调用包括 `SamplingParams`。

### _run_test (lines 45-100)
```python
def _run_test(kwargs: dict, logitproc_loaded: bool) -> None:
    """Compare `LLM` instance initialized with specified `kwargs` against
    reference `LLM` instance.

    Two scenarios:
    1. Server has loaded dummy logitproc; test that requests which specify
       dummy logitproc arg value behave as if logitproc is operating (output
       token value should repeat), while requests that don't specify dummy
       logitproc arg value should match reference `LLM` output.
    2. Server has *not* loaded dummy logitproc; test that all requests
       behave as if logitproc is *not* operating (output matches reference
       `LLM` output.)
    Args:
      kwargs: `LLM` constructor kwargs
      logitproc_loaded: server has loaded dummy logitproc if True
    """
    # ... excerpt omitted for brevity ...
            # reference result
            ref_toks = out_ref.outputs[0].token_ids
            if lp_toks != ref_toks:
                raise AssertionError(
                    f"Request {bdx} generated {lp_toks}, should match {ref_toks}"
                )
```
**EN:** Helper function `_run_test` encapsulates reusable logic for `run test`. Inputs: `kwargs, logitproc_loaded`. Key calls include `LLM, llm_logitproc.generate, llm_ref.generate, enumerate, zip, all`.
**CN:** 辅助函数 `_run_test` 封装了与 `run test` 相关的可复用逻辑。 输入参数：`kwargs, logitproc_loaded`。 关键调用包括 `LLM, llm_logitproc.generate, llm_ref.generate, enumerate, zip, all`。

### test_custom_logitsprocs (lines 105-162)
```python
def test_custom_logitsprocs(monkeypatch, logitproc_source: CustomLogitprocSource):
    """Test offline Python interface for passing custom logitsprocs

    Construct an `LLM` instance which loads a custom logitproc that has a
    well-defined behavior (mask out all tokens except one `target_token`)
    Construct a reference `LLM` instance with no custom logitproc
    Pass in a batch of requests, 50% of which pass a `target_token` value
    in through `SamplingParams.extra_args`, 50% of which do not.
    Validate that
    * Requests which do not activate the custom logitproc, yield the same
      results for both `LLM` instances
    * Requests which activate the custom logitproc, only output `target_token`
    Test four scenarios, corresponding to `logitproc_source` value
    * No logitsprocs loaded - test that generated tokens match reference `LLM`
    # ... excerpt omitted for brevity ...
        kwargs["logits_processors"] = [DUMMY_LOGITPROC_FQCN]
    elif logitproc_source == CustomLogitprocSource.LOGITPROC_SOURCE_CLASS:
        # Scenario: load logitproc from provided class object
        kwargs["logits_processors"] = [DummyLogitsProcessor]
    _run_test(kwargs, logitproc_loaded=True)
```
**EN:** Parameterized test covering `custom logitsprocs`. Parameter axes: `logitproc_source`. Inputs/fixtures: `monkeypatch, logitproc_source`. It exercises `create_new_process_for_each_test, mark.parametrize, monkeypatch.setenv, set_random_seed, _run_test, list`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `custom logitsprocs` 的测试用例。 参数维度：`logitproc_source`。 输入或 fixture：`monkeypatch, logitproc_source`。 该测试会调用 `create_new_process_for_each_test, mark.parametrize, monkeypatch.setenv, set_random_seed, _run_test, list`。 主要通过 mock、回调或输出检查来完成验证。

### test_custom_logitsprocs_req (lines 166-195)
```python
def test_custom_logitsprocs_req(monkeypatch):
    """Test passing request-level logits processor to offline Python interface

    Wrap a request-level logits processor to create a batch level logits
    processor that has a well-defined behavior (mask out all tokens except one
    `target_token`)

    Construct an `LLM` instance which loads the wrapped logits processor. Pass
    the custom logitproc as a class object.

    Construct a reference `LLM` instance with no custom logitproc

    Pass in a batch of requests, 50% of which pass a `target_token` value
    in through `SamplingParams.extra_args`, 50% of which do not.

    Validate that
    * Requests which do not activate the custom logitproc, yield the same
      results for both `LLM` instances
    * Requests which activate the custom logitproc, only output `target_token`

    Args:
      monkeypatch: for setting env vars
    """

    # Test that logitproc info is passed to workers
    monkeypatch.setenv("VLLM_ENABLE_V1_MULTIPROCESSING", "1")
    set_random_seed(40)
    _run_test(
        {"logits_processors": [WrappedPerReqLogitsProcessor]}, logitproc_loaded=True
    )
```
**EN:** Test case covering `custom logitsprocs req`. Inputs/fixtures: `monkeypatch`. It exercises `create_new_process_for_each_test, monkeypatch.setenv, set_random_seed, _run_test`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `custom logitsprocs req` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `create_new_process_for_each_test, monkeypatch.setenv, set_random_seed, _run_test`。 主要通过 mock、回调或输出检查来完成验证。

### test_rejects_custom_logitsprocs (lines 208-288)
```python
def test_rejects_custom_logitsprocs(
    monkeypatch, model_scenario: str, logitproc_source: CustomLogitprocSource
):
    """Validate that vLLM engine initialization properly rejects custom
    logitsprocs when the model is a pooling model or speculative decoding
    enabled.

    Use `LLM` entrypoint. We expect `LLM` initialization to fail before the
    logitproc is actually loaded.
    Scenario 1:
    * Mock a logitproc entrypoint
    * Validate that `LLM` does not load the logitproc
    Scenario 2:
    * Pass custom logitproc to `LLM` constructor
      * Scenario 2a: via FQCN
      * Scenario 2b: via class object
    # ... excerpt omitted for brevity ...
            assert not isinstance(proc, DummyLogitsProcessor)
        llm_kwargs["logits_processors"] = [DummyLogitsProcessor]
    with pytest.raises(ValueError, match=config["error_message"]):
        # Require that loading a model alongside the logitproc raises
        # the appropriate exception.
        LLM(**llm_kwargs)
```
**EN:** Parameterized test covering `rejects custom logitsprocs`. Parameter axes: `model_scenario, logitproc_source`. Inputs/fixtures: `monkeypatch, model_scenario, logitproc_source`. It exercises `create_new_process_for_each_test, mark.parametrize, monkeypatch.setenv, set_random_seed, setup_fake_entrypoint, LLM`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `rejects custom logitsprocs` 的测试用例。 参数维度：`model_scenario, logitproc_source`。 输入或 fixture：`monkeypatch, model_scenario, logitproc_source`。 该测试会调用 `create_new_process_for_each_test, mark.parametrize, monkeypatch.setenv, set_random_seed, setup_fake_entrypoint, LLM`。 代码主体包含 1 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Logits post-processing behavior
- **CN:** logits 后处理行为

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest`.
- **CN:** 外部库：`pytest`。
- **EN:** vLLM modules under test: `vllm, vllm.v1.sample.logits_processor`.
- **CN:** 被测试的 vLLM 模块：`vllm, vllm.v1.sample.logits_processor`。
- **EN:** Local test helpers: `tests.utils, tests.v1.logits_processors.utils`.
- **CN:** 本地测试辅助模块：`tests.utils, tests.v1.logits_processors.utils`。
- **EN:** Standard-library support: `typing`.
- **CN:** 标准库支持：`typing`。
