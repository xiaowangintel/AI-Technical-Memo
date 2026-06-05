# test_mm_processor_kwargs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/llm/test_mm_processor_kwargs.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers LLM entrypoint behavior and process management. The file defines 9 test(s), 0 fixture(s), and 2 helper/class block(s) to validate this area. / [CN] 该文件覆盖LLM 入口行为与进程管理。它定义了 9 个测试、0 个 fixture，以及 2 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L9)
```python
from types import SimpleNamespace
from unittest.mock import Mock

import pytest

from vllm import LLM, SamplingParams
```
**EN:** Imports standard-library modules such as `types.SimpleNamespace`, `unittest.mock.Mock`, third-party packages like `pytest`, project helpers such as `vllm.LLM`, `vllm.SamplingParams`.
**CN:** 导入标准库模块（如 `types.SimpleNamespace`、`unittest.mock.Mock`）、第三方包（如 `pytest`）、项目内辅助模块（如 `vllm.LLM`、`vllm.SamplingParams`）。

### Helper / 辅助函数: _make_mock_llm (L12-L17)
```python
def _make_mock_llm() -> LLM:
    llm = object.__new__(LLM)
    llm.model_config = SimpleNamespace(
        runner_type="generate", enable_prompt_embeds=False
    )
    return llm
```
**EN:** This helper encapsulates reusable logic in `_make_mock_llm`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_make_mock_llm` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_generate_forwards_mm_processor_kwargs (L20-L36)
```python
def test_generate_forwards_mm_processor_kwargs() -> None:
    llm = _make_mock_llm()
    mm_processor_kwargs = {"num_crops": 4}
    sampling_params = SamplingParams(max_tokens=1)

    llm._run_completion = Mock(return_value=["ok"])

    outputs = llm.generate(
        "prompt",
        sampling_params=sampling_params,
        mm_processor_kwargs=mm_processor_kwargs,
    )

    assert outputs == ["ok"]
    assert llm._run_completion.call_args.kwargs["mm_processor_kwargs"] == (
        mm_processor_kwargs
    )
```
**EN:** This test validates `test_generate_forwards_mm_processor_kwargs`. The main assertion is `outputs == ['ok']` and `llm._run_completion.call_args.kwargs['mm_processor_kwargs'] == mm_processor_kwargs`.
**CN:** 这个测试验证 `test_generate_forwards_mm_processor_kwargs`。 核心断言是 `outputs == ['ok']` and `llm._run_completion.call_args.kwargs['mm_processor_kwargs'] == mm_processor_kwargs`。

### Test / 测试: test_enqueue_forwards_mm_processor_kwargs (L39-L56)
```python
def test_enqueue_forwards_mm_processor_kwargs() -> None:
    llm = _make_mock_llm()
    mm_processor_kwargs = {"do_resize": False}
    sampling_params = SamplingParams(max_tokens=1)

    llm._add_completion_requests = Mock(return_value=["req-0"])

    request_ids = llm.enqueue(
        "prompt",
        sampling_params=sampling_params,
        use_tqdm=False,
        mm_processor_kwargs=mm_processor_kwargs,
    )

    assert request_ids == ["req-0"]
    assert llm._add_completion_requests.call_args.kwargs["mm_processor_kwargs"] == (
        mm_processor_kwargs
    )
```
**EN:** This test validates `test_enqueue_forwards_mm_processor_kwargs`. The main assertion is `request_ids == ['req-0']` and `llm._add_completion_requests.call_args.kwargs['mm_processor_kwargs'] == mm_processor_kwargs`.
**CN:** 这个测试验证 `test_enqueue_forwards_mm_processor_kwargs`。 核心断言是 `request_ids == ['req-0']` and `llm._add_completion_requests.call_args.kwargs['mm_processor_kwargs'] == mm_processor_kwargs`。

### Test / 测试: test_chat_forwards_mm_processor_kwargs (L59-L76)
```python
def test_chat_forwards_mm_processor_kwargs() -> None:
    llm = _make_mock_llm()
    mm_processor_kwargs = {"do_pan_and_scan": True}
    sampling_params = SamplingParams(max_tokens=1)
    messages = [{"role": "user", "content": "hello"}]

    llm._run_chat = Mock(return_value=["ok"])

    outputs = llm.chat(
        messages,
        sampling_params=sampling_params,
        mm_processor_kwargs=mm_processor_kwargs,
    )

    assert outputs == ["ok"]
    assert llm._run_chat.call_args.kwargs["mm_processor_kwargs"] == (
        mm_processor_kwargs
    )
```
**EN:** This test validates `test_chat_forwards_mm_processor_kwargs`. The main assertion is `outputs == ['ok']` and `llm._run_chat.call_args.kwargs['mm_processor_kwargs'] == mm_processor_kwargs`.
**CN:** 这个测试验证 `test_chat_forwards_mm_processor_kwargs`。 核心断言是 `outputs == ['ok']` and `llm._run_chat.call_args.kwargs['mm_processor_kwargs'] == mm_processor_kwargs`。

### Test / 测试: test_run_completion_forwards_mm_processor_kwargs (L79-L99)
```python
def test_run_completion_forwards_mm_processor_kwargs() -> None:
    llm = _make_mock_llm()
    mm_processor_kwargs = {"min_pixels": 4 * 28 * 28}
    sampling_params = SamplingParams(max_tokens=1)
    sentinel_output = ["done"]

    llm._add_completion_requests = Mock()
    llm._run_engine = Mock(return_value=sentinel_output)

    outputs = llm._run_completion(
        prompts=["prompt"],
        params=sampling_params,
        output_type=object,
        use_tqdm=False,
        mm_processor_kwargs=mm_processor_kwargs,
    )

    assert outputs == sentinel_output
    assert llm._add_completion_requests.call_args.kwargs["mm_processor_kwargs"] == (
        mm_processor_kwargs
    )
```
**EN:** This test validates `test_run_completion_forwards_mm_processor_kwargs`. The main assertion is `outputs == sentinel_output` and `llm._add_completion_requests.call_args.kwargs['mm_processor_kwargs'] == mm_processor_kwargs`.
**CN:** 这个测试验证 `test_run_completion_forwards_mm_processor_kwargs`。 核心断言是 `outputs == sentinel_output` and `llm._add_completion_requests.call_args.kwargs['mm_processor_kwargs'] == mm_processor_kwargs`。

### Test / 测试: test_add_completion_requests_forwards_mm_processor_kwargs (L102-L133)
```python
def test_add_completion_requests_forwards_mm_processor_kwargs() -> None:
    llm = _make_mock_llm()
    mm_processor_kwargs = {"max_dynamic_patch": 4}
    sampling_params = SamplingParams(max_tokens=1)

    llm._params_to_seq = Mock(return_value=[sampling_params])
    llm._lora_request_to_seq = Mock(return_value=[None])
    llm._priority_to_seq = Mock(return_value=[0])
    llm._preprocess_cmpl_one = Mock(return_value={"prompt_token_ids": [1]})

    captured_prompts = []

    def fake_render_and_add_requests(*, prompts, **_kwargs):
        captured_prompts.extend(prompts)
        return ["req-0"]

    llm._render_and_add_requests = Mock(side_effect=fake_render_and_add_requests)

# ... 6 lines omitted for brevity ...

    assert request_ids == ["req-0"]
    llm._preprocess_cmpl_one.assert_called_once_with(
        "prompt",
        None,
        mm_processor_kwargs=mm_processor_kwargs,
    )
    assert captured_prompts == [{"prompt_token_ids": [1]}]
```
**EN:** This test validates `test_add_completion_requests_forwards_mm_processor_kwargs`. The main assertion is `request_ids == ['req-0']` and `captured_prompts == [{'prompt_token_ids': [1]}]`.
**CN:** 这个测试验证 `test_add_completion_requests_forwards_mm_processor_kwargs`。 核心断言是 `request_ids == ['req-0']` and `captured_prompts == [{'prompt_token_ids': [1]}]`。

### Test / 测试: test_preprocess_cmpl_applies_mm_processor_kwargs_to_renderer (L136-L164)
```python
def test_preprocess_cmpl_applies_mm_processor_kwargs_to_renderer(
    monkeypatch: pytest.MonkeyPatch,
) -> None:
    llm = _make_mock_llm()
    mm_processor_kwargs = {"num_crops": 8}
    prompt = {"prompt": "<image>", "multi_modal_data": {"image": object()}}

    renderer = Mock()
    renderer.default_cmpl_tok_params = Mock()
    renderer.default_cmpl_tok_params.with_kwargs.return_value = "tok-params"
    renderer.render_cmpl.return_value = ["engine-input"]
    llm.renderer = renderer

    monkeypatch.setattr(
        "vllm.entrypoints.llm.parse_model_prompt",
        lambda _model_config, parsed_prompt: parsed_prompt,
    )

    outputs = llm._preprocess_cmpl(
        [prompt],
        mm_processor_kwargs=mm_processor_kwargs,
    )

    assert outputs == ["engine-input"]
    renderer.render_cmpl.assert_called_once_with(
        [prompt],
        "tok-params",
        prompt_extras={"mm_processor_kwargs": mm_processor_kwargs},
    )
```
**EN:** This test validates `test_preprocess_cmpl_applies_mm_processor_kwargs_to_renderer`. Key inputs are `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `outputs == ['engine-input']`.
**CN:** 这个测试验证 `test_preprocess_cmpl_applies_mm_processor_kwargs_to_renderer`。 关键输入包括 `monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `outputs == ['engine-input']`。

### Test / 测试: test_preprocess_cmpl_keeps_prompt_mm_processor_kwargs_when_no_override (L167-L195)
```python
def test_preprocess_cmpl_keeps_prompt_mm_processor_kwargs_when_no_override(
    monkeypatch: pytest.MonkeyPatch,
) -> None:
    llm = _make_mock_llm()
    prompt = {
        "prompt": "<image>",
        "multi_modal_data": {"image": object()},
        "mm_processor_kwargs": {"num_crops": 2},
    }

    renderer = Mock()
    renderer.default_cmpl_tok_params = Mock()
    renderer.default_cmpl_tok_params.with_kwargs.return_value = "tok-params"
    renderer.render_cmpl.return_value = ["engine-input"]
    llm.renderer = renderer

    monkeypatch.setattr(
        "vllm.entrypoints.llm.parse_model_prompt",
        lambda _model_config, parsed_prompt: parsed_prompt,
    )

    outputs = llm._preprocess_cmpl([prompt])

    assert outputs == ["engine-input"]
    renderer.render_cmpl.assert_called_once_with(
        [prompt],
        "tok-params",
        prompt_extras=None,
    )
```
**EN:** This test validates `test_preprocess_cmpl_keeps_prompt_mm_processor_kwargs_when_no_override`. Key inputs are `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `outputs == ['engine-input']`.
**CN:** 这个测试验证 `test_preprocess_cmpl_keeps_prompt_mm_processor_kwargs_when_no_override`。 关键输入包括 `monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `outputs == ['engine-input']`。

### Test / 测试: test_preprocess_chat_applies_mm_processor_kwargs_to_renderer (L198-L222)
```python
def test_preprocess_chat_applies_mm_processor_kwargs_to_renderer() -> None:
    llm = _make_mock_llm()
    mm_processor_kwargs = {"num_crops": 8}
    messages = [[{"role": "user", "content": "Describe this image."}]]

    renderer = Mock()
    renderer.tokenizer = object()
    renderer.default_chat_tok_params = Mock()
    renderer.default_chat_tok_params.with_kwargs.return_value = "tok-params"
    renderer.render_chat.return_value = (messages, ["engine-input"])
    llm.renderer = renderer

    outputs = llm._preprocess_chat(
        messages,
        mm_processor_kwargs=mm_processor_kwargs,
    )

    assert outputs == ["engine-input"]
    call_args = renderer.render_chat.call_args
    assert call_args.args[0] == messages
    assert call_args.args[1].mm_processor_kwargs == mm_processor_kwargs
    assert call_args.args[2] == "tok-params"
    assert call_args.kwargs["prompt_extras"] == {
        "mm_processor_kwargs": mm_processor_kwargs
    }
```
**EN:** This test validates `test_preprocess_chat_applies_mm_processor_kwargs_to_renderer`. The main assertion is `outputs == ['engine-input']` and `call_args.args[0] == messages`.
**CN:** 这个测试验证 `test_preprocess_chat_applies_mm_processor_kwargs_to_renderer`。 核心断言是 `outputs == ['engine-input']` and `call_args.args[0] == messages`。

### Test / 测试: test_preprocess_chat_omits_mm_processor_kwargs_when_no_override (L225-L243)
```python
def test_preprocess_chat_omits_mm_processor_kwargs_when_no_override() -> None:
    llm = _make_mock_llm()
    messages = [[{"role": "user", "content": "Describe this image."}]]

    renderer = Mock()
    renderer.tokenizer = object()
    renderer.default_chat_tok_params = Mock()
    renderer.default_chat_tok_params.with_kwargs.return_value = "tok-params"
    renderer.render_chat.return_value = (messages, ["engine-input"])
    llm.renderer = renderer

    outputs = llm._preprocess_chat(messages)

    assert outputs == ["engine-input"]
    call_args = renderer.render_chat.call_args
    assert call_args.args[0] == messages
    assert call_args.args[1].mm_processor_kwargs is None
    assert call_args.args[2] == "tok-params"
    assert call_args.kwargs["prompt_extras"] is None
```
**EN:** This test validates `test_preprocess_chat_omits_mm_processor_kwargs_when_no_override`. The main assertion is `outputs == ['engine-input']` and `call_args.args[0] == messages`.
**CN:** 这个测试验证 `test_preprocess_chat_omits_mm_processor_kwargs_when_no_override`。 核心断言是 `outputs == ['engine-input']` and `call_args.args[0] == messages`。

## Key Concepts / 关键概念
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `types.SimpleNamespace`, `unittest.mock.Mock`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `vllm.LLM`, `vllm.SamplingParams`
