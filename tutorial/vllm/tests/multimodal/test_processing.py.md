# test_processing.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/multimodal/test_processing.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Processing behavior in the Multimodal test area through focused pytest scenarios. It focuses on scenarios such as Iter Token Matches, Replace Token Matches, Find Token Matches. / 该文件在 Multimodal 测试域中，通过有针对性的 pytest 场景验证 Processing 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-28)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import time
from contextlib import nullcontext

import numpy as np
import pytest

from vllm.config import ModelConfig
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.processing.context import InputProcessingContext
from vllm.multimodal.processing.processor import (
    PlaceholderFeaturesInfo,
    PromptIndexTargets,
    PromptInsertion,
    PromptReplacement,
    _apply_matches,
    apply_text_matches,
    apply_token_matches,
    find_mm_placeholders,
    iter_token_matches,
    replace_token_matches,
)

from .utils import random_image

pytestmark = pytest.mark.cpu_test
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `time`, `contextlib`, `numpy`, `pytest`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Test: test_iter_token_matches (lines 31-89)
```python
@pytest.mark.parametrize(
    ("token_ids", "match_ids", "expected"),
    [
        ([], [], []),
        ([], [32000], []),
        (
            [32000, 32000, 32000],
            [32000],
            [
                {"start_idx": 0, "end_idx": 1},
                {"start_idx": 1, "end_idx": 2},
                {"start_idx": 2, "end_idx": 3},
            ],
        ),
        (
            [32000, 32000, 32000],
            [32000, 32000],
            [{"start_idx": 0, "end_idx": 2}],
        ),
# ... omitted for brevity ...
)
@pytest.mark.parametrize("start_idx", [0, 4, 8])
def test_iter_token_matches(token_ids, match_ids, expected, start_idx):
    result = list(iter_token_matches(token_ids, match_ids, start_idx=start_idx))

    # Manually constructed results
    assert [item._asdict() for item in result] == [
        item for item in expected if item["start_idx"] >= start_idx
    ]

    # Invariants
    match_lens = [end - start for start, end in result]
    print("match_lens:", match_lens)  # Only displayed on error
    assert all(match_len == len(match_ids) for match_len in match_lens)
```
**EN:** Checks Iter Token Matches under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `list`, `print` before asserting the expected outcome.
**CN:** 该测试用例验证 Iter Token Matches 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `list`, `print` 驱动目标逻辑，再断言预期结果。

### Test: test_replace_token_matches (lines 92-139)
```python
@pytest.mark.parametrize(
    ("token_ids", "match_ids", "new_ids", "expected"),
    [
        ([], [], [-1], []),
        ([], [32000], [-1], []),
        (
            [32000, 32000, 32000],
            [32000],
            [-1],
            [-1, -1, -1],
        ),
        (
            [32000, 32000, 32000],
            [32000, 32000],
            [-1],
            [-1, 32000],
        ),
        (
            [32000, 32000, 32000],
# ... omitted for brevity ...
        ),
        (
            [9833, 28747, 32000, 32000, 32000, 9833, 28747, 32000, 32000, 918],
            [28747, 0, 32000],
            [-1],
            [9833, 28747, 32000, 32000, 32000, 9833, 28747, 32000, 32000, 918],
        ),
    ],
)
def test_replace_token_matches(token_ids, match_ids, new_ids, expected):
    result = replace_token_matches(token_ids, match_ids, new_ids)

    # Manually constructed results
    assert result == expected
```
**EN:** Checks Replace Token Matches under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `replace_token_matches` before asserting the expected outcome.
**CN:** 该测试用例验证 Replace Token Matches 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `replace_token_matches` 驱动目标逻辑，再断言预期结果。

### Test: test_find_token_matches (lines 142-257)
```python
@pytest.mark.parametrize(
    ("prompt", "target_by_key", "expected_by_key"),
    [
        (
            [],
            {
                "pattern_1": [],
                "pattern_2": [32000],
                "pattern_3": PromptIndexTargets.start(),
                "pattern_4": PromptIndexTargets.prefix([32000]),
                "pattern_5": PromptIndexTargets.end(),
            },
            {
                "pattern_1": [],
                "pattern_2": [],
                "pattern_3": [
                    {"start_idx": 0, "end_idx": 0},
                ],
                "pattern_4": [],
# ... omitted for brevity ...
        for key, update in prompt_updates.items()
    }

    # Only displayed on error
    print("result:", result)

    # Manually constructed results
    assert {
        key: [
            dict(start_idx=item.start_idx, end_idx=item.end_idx)
            for item in result.get(key, [])
        ]
        for key in expected_by_key
    } == expected_by_key
```
**EN:** Checks Find Token Matches under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `print`, `update_type(key, target, []).resolve` before asserting the expected outcome.
**CN:** 该测试用例验证 Find Token Matches 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `print`, `update_type(key, target, []).resolve` 驱动目标逻辑，再断言预期结果。

### Test: test_find_text_matches (lines 260-401)
```python
@pytest.mark.parametrize(
    ("prompt", "target_by_key", "expected_by_key"),
    [
        # Detokenized test cases of `test_find_token_matches`
        # using the vocab of llava-hf/llava-v1.6-mistral-7b-hf
        (
            "",
            {
                "pattern_1": "",
                "pattern_2": "<image>",
                "pattern_3": PromptIndexTargets.start(),
                "pattern_4": PromptIndexTargets.prefix("<image>"),
                "pattern_5": PromptIndexTargets.end(),
            },
            {
                "pattern_1": [{"start_idx": 0, "end_idx": 0}],
                "pattern_2": [],
                "pattern_3": [
                    {"start_idx": 0, "end_idx": 0},
# ... omitted for brevity ...
        for key, update in prompt_updates.items()
    }

    # Only displayed on error
    print("result:", result)

    # Manually constructed results
    assert {
        key: [
            dict(start_idx=item.start_idx, end_idx=item.end_idx)
            for item in result.get(key, [])
        ]
        for key in expected_by_key
    } == expected_by_key
```
**EN:** Checks Find Text Matches under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `print`, `update_type(key, target, []).resolve` before asserting the expected outcome.
**CN:** 该测试用例验证 Find Text Matches 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `print`, `update_type(key, target, []).resolve` 驱动目标逻辑，再断言预期结果。

### Test: test_find_update_text (lines 404-566)
```python
@pytest.mark.parametrize(
    ("prompt", "target_by_key", "repl_by_key", "expected_by_update_type_mm_count"),  # noqa: E501
    [
        (
            "Image:<image>Image:<image><image>!",
            {
                # We use `<image>` before `Image:` to test matches that
                # occur out of order
                "pattern_1": "<image>",
                "pattern_2": "Image:",
                "pattern_3": "!",
            },
            {
                # Test whether target is confused with replacement
                "pattern_1": "<image><image>",
                # Test empty replacement
                "pattern_2": "",
                # Test dynamic replacement (beyond the form of `unit * count`)
                "pattern_3": "?!?",
# ... omitted for brevity ...
                prompt,
                mm_prompt_updates,
                tokenizer=None,
            )

            # Only displayed on error
            print("update_type:", update_type)
            print("mm_count:", mm_count)
            print("mm_prompt_updates:", mm_prompt_updates)
            print("new_prompt:", new_prompt)
            print("result:", result)

            # Manually constructed results
            assert new_prompt == expected
```
**EN:** Checks Find Update Text under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `expected_by_update_type_mm_count.items`, `expected_by_mm_count.items` before asserting the expected outcome.
**CN:** 该测试用例验证 Find Update Text 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `expected_by_update_type_mm_count.items`, `expected_by_mm_count.items` 驱动目标逻辑，再断言预期结果。

### Test: test_find_update_tokens (lines 569-768)
```python
@pytest.mark.parametrize(
    ("prompt", "target_by_key", "repl_by_key", "expected_by_update_type_mm_count"),  # noqa: E501
    [
        # Tokenized test cases of `test_find_update_text`
        # using the vocab of llava-hf/llava-v1.6-mistral-7b-hf
        (
            [1, 9833, 28747, 32000, 9833, 28747, 32000, 32000, 918],
            {
                # We use `<image>` before `Image:` to test matches that
                # occur out of order
                "pattern_1": [32000],
                "pattern_2": [9833, 28747],
                "pattern_3": [918],
            },
            {
                # Test whether target is confused with replacement
                "pattern_1": [32000, 32000],
                # Test empty replacement
                "pattern_2": [],
# ... omitted for brevity ...
                prompt,
                mm_prompt_updates,
                tokenizer=None,
            )

            # Only displayed on error
            print("update_type:", update_type)
            print("mm_count:", mm_count)
            print("mm_prompt_updates:", mm_prompt_updates)
            print("new_prompt:", new_prompt)
            print("result:", result)

            # Manually constructed results
            assert new_prompt == expected
```
**EN:** Checks Find Update Tokens under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `expected_by_update_type_mm_count.items`, `expected_by_mm_count.items` before asserting the expected outcome.
**CN:** 该测试用例验证 Find Update Tokens 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `expected_by_update_type_mm_count.items`, `expected_by_mm_count.items` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
test_find_mm_placeholders
test_limit_mm_per_prompt_apply
test_budget_caps_prevent_dummy_input_validation_failure
DummyProcessor
test_hf_processor_init_kwargs
test_hf_processor_call_kwargs
test_apply_matches_no_match_exits_quickly
```
**EN:** The file contains more helper or test blocks than can be shown compactly here. This summary lists additional scenarios that continue the same validation strategy.
**CN:** 该文件包含的辅助逻辑或测试场景较多，无法在此完整展开；这里列出其余场景名称，便于快速把握覆盖范围。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **Tokenization / 分词与反分词**
  - **EN:** The logic checks how text and token IDs are converted, streamed, or reconstructed.
  - **CN:** 逻辑重点检查文本与 token ID 的转换、流式处理和重建过程。
- **Multimodal inputs / 多模态输入**
  - **EN:** The tests combine text with image, audio, or video data and verify the modality-specific pipeline.
  - **CN:** 这些测试把文本与图像、音频或视频数据结合起来，验证模态专用处理流程。
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `time`, `contextlib`
- **Third-party / 第三方依赖**: `numpy`, `pytest`
- **vLLM internal / vLLM 内部依赖**: `vllm.config`, `vllm.multimodal`, `vllm.multimodal.processing.context`, `vllm.multimodal.processing.processor`
- **Local test utilities / 本地测试辅助**: `.utils`
