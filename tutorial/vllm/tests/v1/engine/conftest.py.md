# conftest.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/engine/conftest.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Contains support code for `conftest` within the v1 test suite. / 包含 v1 测试套件中与 `conftest` 相关的支持代码。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-20)
```python
import pytest
import torch
from transformers import AutoTokenizer

from tests.v1.engine.utils import (
    FULL_STRINGS,
    NUM_PROMPT_LOGPROBS_UNDER_TEST,
    NUM_SAMPLE_LOGPROBS_UNDER_TEST,
    PROMPT_LEN,
    TOKENIZER_NAME,
    DummyOutputProcessorTestVectors,
    generate_dummy_prompt_logprobs_tensors,
    generate_dummy_sample_logprobs,
)
from vllm.engine.arg_utils import EngineArgs

from ...distributed.conftest import publisher_config, random_port  # noqa: F401
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch, transformers`. vLLM modules under test include `vllm.engine.arg_utils`. Local helpers come from `tests.v1.engine.utils, tests.distributed.conftest`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch, transformers`。 被测试的 vLLM 模块包括 `vllm.engine.arg_utils`。 本地测试辅助逻辑来自 `tests.v1.engine.utils, tests.distributed.conftest`。

### Module state / 模块级状态 (lines 22-23)
```python
EngineCoreSampleLogprobsType = list[tuple[torch.Tensor, torch.Tensor]]
EngineCorePromptLogprobsType = tuple[torch.Tensor, torch.Tensor]
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `EngineCoreSampleLogprobsType, EngineCorePromptLogprobsType`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`EngineCoreSampleLogprobsType, EngineCorePromptLogprobsType`。

### _build_test_vectors_no_logprobs (lines 26-60)
```python
def _build_test_vectors_no_logprobs() -> DummyOutputProcessorTestVectors:
    """Generate output processor dummy test vectors, without logprobs

    Returns:
      DummyOutputProcessorTestVectors instance with no logprobs
    """
    tokenizer = AutoTokenizer.from_pretrained(TOKENIZER_NAME)
    vllm_config = EngineArgs(model=TOKENIZER_NAME).create_engine_config()
    # Tokenize prompts under test & create dummy generated tokens
    prompt_tokens = [tokenizer(text).input_ids[:PROMPT_LEN] for text in FULL_STRINGS]
    generation_tokens = [
        tokenizer(text).input_ids[PROMPT_LEN:] for text in FULL_STRINGS
    ]
    # Generate prompt strings
    prompt_strings = [
        tokenizer.decode(prompt_tokens, skip_special_tokens=True)
        for prompt_tokens in prompt_tokens
    # ... excerpt omitted for brevity ...
    return DummyOutputProcessorTestVectors(
            text[prompt_len:]
            for text, prompt_len in zip(FULL_STRINGS, prompt_strings_len)
        ],
        prompt_logprobs=[],
        generation_logprobs=[],
    )
```
**EN:** Helper function `_build_test_vectors_no_logprobs` encapsulates reusable logic for `test vectors no logprobs`. Key calls include `AutoTokenizer.from_pretrained, EngineArgs.create_engine_config, DummyOutputProcessorTestVectors, tokenizer.decode, len, EngineArgs`.
**CN:** 辅助函数 `_build_test_vectors_no_logprobs` 封装了与 `test vectors no 对数概率` 相关的可复用逻辑。 关键调用包括 `AutoTokenizer.from_pretrained, EngineArgs.create_engine_config, DummyOutputProcessorTestVectors, tokenizer.decode, len, EngineArgs`。

### dummy_test_vectors (lines 64-90)
```python
def dummy_test_vectors() -> DummyOutputProcessorTestVectors:
    """Generate output processor dummy test vectors, with logprobs

    Returns:
      DummyOutputProcessorTestVectors instance with logprobs
    """
    # Build dummy test vectors without logprobs
    dtv = _build_test_vectors_no_logprobs()
    # Inject logprobs into dummy test vectors
    # data structure
    dtv.generation_logprobs = [
        generate_dummy_sample_logprobs(
            sampled_tokens_list=tokens_list,
            num_logprobs=NUM_SAMPLE_LOGPROBS_UNDER_TEST,
            tokenizer=dtv.tokenizer,
        )
        for tokens_list in dtv.generation_tokens
    ]
    dtv.prompt_logprobs = [
        generate_dummy_prompt_logprobs_tensors(
            prompt_tokens_list=tokens_list,
            num_logprobs=NUM_PROMPT_LOGPROBS_UNDER_TEST,
            tokenizer=dtv.tokenizer,
        )
        for tokens_list in dtv.prompt_tokens
    ]
    return dtv
```
**EN:** Fixture/helper `dummy_test_vectors` prepares reusable state for downstream tests. Key calls include `_build_test_vectors_no_logprobs, generate_dummy_sample_logprobs, generate_dummy_prompt_logprobs_tensors`.
**CN:** `dummy_test_vectors` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `_build_test_vectors_no_logprobs, generate_dummy_sample_logprobs, generate_dummy_prompt_logprobs_tensors`。

## Key Concepts / 关键概念
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Engine-core request/response orchestration
- **CN:** 引擎核心的请求/响应编排
- **EN:** Token/logprob accounting correctness
- **CN:** token/logprob 统计正确性

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch, transformers`.
- **CN:** 外部库：`pytest, torch, transformers`。
- **EN:** vLLM modules under test: `vllm.engine.arg_utils`.
- **CN:** 被测试的 vLLM 模块：`vllm.engine.arg_utils`。
- **EN:** Local test helpers: `tests.v1.engine.utils, tests.distributed.conftest`.
- **CN:** 本地测试辅助模块：`tests.v1.engine.utils, tests.distributed.conftest`。
