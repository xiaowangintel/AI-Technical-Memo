# test_correctness_sliding_window.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/e2e/general/test_correctness_sliding_window.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises end-to-end `correctness sliding window` scenarios in the v1 test suite. / 在 v1 测试套件中覆盖端到端的 `correctness sliding window` 场景。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-10)
```python
from dataclasses import dataclass

import pytest

from vllm import LLM, SamplingParams
from vllm.platforms import current_platform

from ....utils import check_answers, prep_prompts
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest`. vLLM modules under test include `vllm, vllm.platforms`. Local helpers come from `tests.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest`。 被测试的 vLLM 模块包括 `vllm, vllm.platforms`。 本地测试辅助逻辑来自 `tests.utils`。

### TestConfig (lines 14-16)
```python
class TestConfig:
    sliding_window: int
    ln_range: tuple[int, int]
```
**EN:** Class `TestConfig` groups 0 test method(s).
**CN:** 类 `TestConfig` 组织了 0 个测试方法。

### Module state / 模块级状态 (lines 19-22)
```python
model_config = {
    "bigcode/starcoder2-3b": TestConfig(4096, (800, 1100)),
    "google/gemma-3-1b-it": TestConfig(4096, (400, 800)),
}
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `model_config`. Shared setup calls include `TestConfig`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`model_config`。 共享初始化调用包括 `TestConfig`。

### test_sliding_window_retrieval (lines 35-78)
```python
def test_sliding_window_retrieval(
    model, batch_size, seed, disable_hybrid_kv_cache_manager
):
    """
    The test does a bunch of assignments "x1 = 10\nx2 = 33\n..." and then
    asks for value of one of them (which is outside the sliding window).
    If we tell it upfront which we are going to be looking for, then
    it answers correctly (mostly).
    # NOTE: For ROCm, we have to enforce eager mode to use custom kernel
    # implementation of GELU with tanh approximation, as PyTorch's native
    # implementation is currently unstable with torch.compile and produces garbage.
    enforce_eager = current_platform.is_rocm()

    test_config = model_config[model]
    llm = LLM(
        model=model,
    # ... excerpt omitted for brevity ...
    check_answers(
        indices,
        answer,
        [response.outputs[0].text for response in responses],
        accept_rate=1.0,
    )
```
**EN:** Parameterized test covering `sliding window retrieval`. Parameter axes: `model, batch_size, seed, disable_hybrid_kv_cache_manager`. Inputs/fixtures: `model, batch_size, seed, disable_hybrid_kv_cache_manager`. It exercises `mark.parametrize, current_platform.is_rocm, LLM, SamplingParams, prep_prompts, check_length`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `sliding window retrieval` 的测试用例。 参数维度：`model, batch_size, seed, disable_hybrid_kv_cache_manager`。 输入或 fixture：`model, batch_size, seed, disable_hybrid_kv_cache_manager`。 该测试会调用 `mark.parametrize, current_platform.is_rocm, LLM, SamplingParams, prep_prompts, check_length`。 主要通过 mock、回调或输出检查来完成验证。

### check_length (lines 81-98)
```python
def check_length(prompts: list[str], llm: LLM, sliding_window: int):
    """
    Check if the prompt length is valid, i.e., longer than the sliding window
    size and shorter than the model's max length.

    Args:
        prompts: list of prompts
        llm: LLM object
        sliding_window: Sliding window size
    """
    tokenizer = llm.get_tokenizer()
    max_model_len = llm.llm_engine.model_config.max_model_len
    assert any(len(tokenizer.encode(prompt)) > sliding_window for prompt in prompts), (
        "Prompt is too short for test"
    )
    assert all(len(tokenizer.encode(prompt)) <= max_model_len for prompt in prompts), (
        "Prompt is too long for test"
    )
```
**EN:** Helper function `check_length` encapsulates reusable logic for `check length`. Inputs: `prompts, llm, sliding_window`. Key calls include `llm.get_tokenizer, any, all, len, tokenizer.encode`. It includes 2 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `check_length` 封装了与 `check length` 相关的可复用逻辑。 输入参数：`prompts, llm, sliding_window`。 关键调用包括 `llm.get_tokenizer, any, all, len, tokenizer.encode`。 其中包含 2 个内部断言，用于保护前置假设。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** End-to-end integration rather than isolated units
- **CN:** 端到端集成而非孤立单元

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest`.
- **CN:** 外部库：`pytest`。
- **EN:** vLLM modules under test: `vllm, vllm.platforms`.
- **CN:** 被测试的 vLLM 模块：`vllm, vllm.platforms`。
- **EN:** Local test helpers: `tests.utils`.
- **CN:** 本地测试辅助模块：`tests.utils`。
- **EN:** Standard-library support: `dataclasses`.
- **CN:** 标准库支持：`dataclasses`。
