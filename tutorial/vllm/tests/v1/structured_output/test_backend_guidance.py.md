# test_backend_guidance.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/structured_output/test_backend_guidance.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `backend guidance` behavior and regressions in the v1 stack. / 验证 v1 栈中 `后端 guidance` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-18)
```python
import time
from concurrent.futures import Future

import pytest
from transformers import AutoTokenizer

from vllm.config import StructuredOutputsConfig, VllmConfig
from vllm.config.model import ModelConfig
from vllm.config.parallel import ParallelConfig
from vllm.config.speculative import SpeculativeConfig
from vllm.sampling_params import SamplingParams, StructuredOutputsParams
from vllm.tokenizers import get_tokenizer
from vllm.v1.request import Request
from vllm.v1.structured_output import StructuredOutputManager
from vllm.v1.structured_output.backend_guidance import GuidanceBackend
from vllm.v1.structured_output.backend_types import StructuredOutputOptions
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, transformers`. vLLM modules under test include `vllm.config, vllm.config.model, vllm.config.parallel, vllm.config.speculative, vllm.sampling_params, ...`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, transformers`。 被测试的 vLLM 模块包括 `vllm.config, vllm.config.model, vllm.config.parallel, vllm.config.speculative, vllm.sampling_params, ...`。

### Module state / 模块级状态 (line 20)
```python
TOKENIZER = "gpt2"
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `TOKENIZER`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`TOKENIZER`。

### mistral_tokenizer (lines 24-28)
```python
def mistral_tokenizer():
    return get_tokenizer(
        tokenizer_name="mistralai/Mistral-Small-3.2-24B-Instruct-2506",
        tokenizer_mode="mistral",
    )
```
**EN:** Fixture/helper `mistral_tokenizer` prepares reusable state for downstream tests. Key calls include `pytest.fixture, get_tokenizer`.
**CN:** `mistral_tokenizer` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `pytest.fixture, get_tokenizer`。

### test_backend_guidance_rollback_terminated (lines 31-75)
```python
def test_backend_guidance_rollback_terminated():
    # Test that the backend guidance successfully rollbacks from a
    # terminated state. This can happen with speculative decoding,
    # where the draft model proposes EOS and it is verified by the
    # guidance backend. In that case we are in a stopped state, but
    # it should be reverted in case EOS is not accepted by the target
    # model.
    structured_outputs_config = StructuredOutputsConfig(backend="guidance")
    vllm_config = VllmConfig(structured_outputs_config=structured_outputs_config)
    tokenizer = AutoTokenizer.from_pretrained(TOKENIZER)

    backend = GuidanceBackend(
        vllm_config,
        tokenizer=tokenizer,
        vocab_size=50257,
    )
    grammar = backend.compile_grammar(
    # ... excerpt omitted for brevity ...
    assert len(prompt) > 1
        assert grammar.accept_tokens("", [token])
    assert not grammar.is_terminated()
    assert grammar.accept_tokens("", [tokenizer.eos_token_id])
    assert grammar.is_terminated()
    assert grammar.accept_tokens("", dummy_wrong)
    # Rollback of <= 0 should not change the terminated state
    grammar.rollback(0)
    grammar.rollback(-1)
```
**EN:** Test case covering `backend guidance rollback terminated`. It exercises `StructuredOutputsConfig, VllmConfig, AutoTokenizer.from_pretrained, GuidanceBackend, backend.compile_grammar, tokenizer.encode`. The body contains 15 explicit assertion(s).
**CN:** 该代码块是覆盖 `后端 guidance rollback terminated` 的测试用例。 该测试会调用 `StructuredOutputsConfig, VllmConfig, AutoTokenizer.from_pretrained, GuidanceBackend, backend.compile_grammar, tokenizer.encode`。 代码主体包含 15 个显式断言。

### test_grammar_bitmask_with_specdec (lines 78-129)
```python
def test_grammar_bitmask_with_specdec():
    tokenizer = AutoTokenizer.from_pretrained(TOKENIZER)
    prompt = tokenizer.encode('{"a": "b"}')
    vllm_config = VllmConfig(
        model_config=ModelConfig(tokenizer=TOKENIZER),
        structured_outputs_config=StructuredOutputsConfig(backend="guidance"),
        speculative_config=SpeculativeConfig(model="[ngram]", num_speculative_tokens=3),
    )
    structured_output_manager = StructuredOutputManager(vllm_config)

    for i in range(1, 2):
        sampling_params = SamplingParams(
            structured_outputs=StructuredOutputsParams(
                json='{"type": "object"}',
            ),
        )
        sampling_params.structured_outputs._backend = "guidance"
        sampling_params.update_from_generation_config({}, tokenizer.eos_token_id)
    # ... excerpt omitted for brevity ...
            assert not req.structured_output_request.grammar.is_terminated()
        assert request.structured_output_request.grammar.accept_tokens(
        grammar_bitmask(request, prompt[i:] + [tokenizer.eos_token_id])
        grammar_bitmask(
            request, prompt[i:] + [tokenizer.eos_token_id] + prompt
        )  # EOS not the final token
        grammar_bitmask(request, prompt[i:])  # EOS not present
```
**EN:** Test case covering `grammar bitmask with specdec`. It exercises `AutoTokenizer.from_pretrained, tokenizer.encode, VllmConfig, StructuredOutputManager, range, SamplingParams`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `grammar bitmask with specdec` 的测试用例。 该测试会调用 `AutoTokenizer.from_pretrained, tokenizer.encode, VllmConfig, StructuredOutputManager, range, SamplingParams`。 代码主体包含 2 个显式断言。

### test_grammar_init_async_and_sync (lines 133-198)
```python
def test_grammar_init_async_and_sync(async_grammar):
    """Test grammar initialization works correctly in both async and sync modes.

    This test validates that the distributed_executor_backend config option
    correctly controls whether grammar compilation happens asynchronously
    (via executor.submit) or synchronously. When set to "external_launcher",
    grammar compilation is synchronous to avoid deadlocks.
    """
    tokenizer = AutoTokenizer.from_pretrained(TOKENIZER)
    prompt = tokenizer.encode('{"a": "b"}')
    # Use "external_launcher" for sync mode, None for async mode
    executor_backend = None if async_grammar else "external_launcher"
    vllm_config = VllmConfig(
        model_config=ModelConfig(tokenizer=TOKENIZER),
        structured_outputs_config=StructuredOutputsConfig(backend="guidance"),
        parallel_config=ParallelConfig(distributed_executor_backend=executor_backend),
    )
    # ... excerpt omitted for brevity ...
        assert isinstance(raw_grammar, Future), (
        assert not isinstance(raw_grammar, Future), (
    assert not isinstance(request.structured_output_request._grammar, Future)
    grammar = request.structured_output_request.grammar
    assert grammar is not None
    assert not grammar.is_terminated()
    # Verify the grammar can accept valid tokens
    assert grammar.accept_tokens(request.request_id, prompt)
```
**EN:** Parameterized test covering `grammar init async and sync`. Parameter axes: `async_grammar`. Inputs/fixtures: `async_grammar`. It exercises `mark.parametrize, AutoTokenizer.from_pretrained, tokenizer.encode, VllmConfig, StructuredOutputManager, SamplingParams`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `grammar init async and sync` 的测试用例。 参数维度：`async_grammar`。 输入或 fixture：`async_grammar`。 该测试会调用 `mark.parametrize, AutoTokenizer.from_pretrained, tokenizer.encode, VllmConfig, StructuredOutputManager, SamplingParams`。 代码主体包含 6 个显式断言。

### test_mistral_tokenizer_compile_grammar (lines 216-233)
```python
def test_mistral_tokenizer_compile_grammar(
    mistral_tokenizer,
    request_type: StructuredOutputOptions,
    grammar_spec: str,
) -> None:
    vllm_config = VllmConfig(
        structured_outputs_config=StructuredOutputsConfig(backend="guidance"),
    )
    backend = GuidanceBackend(
        vllm_config,
        tokenizer=mistral_tokenizer,
        vocab_size=mistral_tokenizer.vocab_size,
    )
    assert backend.ll_tokenizer is mistral_tokenizer.llg_tokenizer

    grammar = backend.compile_grammar(request_type, grammar_spec)
    assert grammar is not None
    assert not grammar.is_terminated()
```
**EN:** Parameterized test covering `mistral tokenizer compile grammar`. Parameter axes: `request_type, grammar_spec`. Inputs/fixtures: `mistral_tokenizer, request_type, grammar_spec`. It exercises `mark.parametrize, VllmConfig, GuidanceBackend, backend.compile_grammar, grammar.is_terminated, pytest.param`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `mistral 分词器 compile grammar` 的测试用例。 参数维度：`request_type, grammar_spec`。 输入或 fixture：`mistral_tokenizer, request_type, grammar_spec`。 该测试会调用 `mark.parametrize, VllmConfig, GuidanceBackend, backend.compile_grammar, grammar.is_terminated, pytest.param`。 代码主体包含 3 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化
- **EN:** Token-level boundary and decoding checks
- **CN:** token 级边界与解码检查
- **EN:** Speculative execution or decoding paths
- **CN:** 推测执行或解码路径

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, transformers`.
- **CN:** 外部库：`pytest, transformers`。
- **EN:** vLLM modules under test: `vllm.config, vllm.config.model, vllm.config.parallel, vllm.config.speculative, vllm.sampling_params, vllm.tokenizers, vllm.v1.request, vllm.v1.structured_output, ...`.
- **CN:** 被测试的 vLLM 模块：`vllm.config, vllm.config.model, vllm.config.parallel, vllm.config.speculative, vllm.sampling_params, vllm.tokenizers, vllm.v1.request, vllm.v1.structured_output, ...`。
- **EN:** Standard-library support: `time, concurrent.futures`.
- **CN:** 标准库支持：`time, concurrent.futures`。
