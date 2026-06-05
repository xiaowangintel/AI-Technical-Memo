# test_mistral.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/generation/test_mistral.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers language-model behavior and model-facing behavior. The file defines 5 test(s), 0 fixture(s), and 3 helper/class block(s) to validate this area. / [CN] 该文件覆盖语言模型行为与面向模型的行为。它定义了 5 个测试、0 个 fixture，以及 3 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L15)
```python
import copy
import json

import pytest

from vllm.sampling_params import SamplingParams
from vllm.tokenizers.mistral import MistralTokenizer
from vllm.tool_parsers.mistral_tool_parser import (
    MistralToolCall,
    MistralToolParser,
)

from ...utils import check_logprobs_close
```
**EN:** Imports standard-library modules such as `copy`, `json`, third-party packages like `pytest`, project helpers such as `vllm.sampling_params.SamplingParams`, `vllm.tokenizers.mistral.MistralTokenizer`, `vllm.tool_parsers.mistral_tool_parser.MistralToolCall`.
**CN:** 导入标准库模块（如 `copy`、`json`）、第三方包（如 `pytest`）、项目内辅助模块（如 `vllm.sampling_params.SamplingParams`、`vllm.tokenizers.mistral.MistralTokenizer`、`vllm.tool_parsers.mistral_tool_parser.MistralToolCall`）。

### Module setup / 模块级配置: MODELS, MISTRAL_FORMAT_MODELS, SAMPLING_PARAMS (L17-L150)
```python
MODELS = [
    "mistralai/Mistral-7B-Instruct-v0.3",
]

MISTRAL_FORMAT_MODELS = [
    "mistralai/Mistral-7B-Instruct-v0.3",
    # uses the v3-Tekken tokenizer
    "mistralai/Ministral-8B-Instruct-2410",
    # Mistral-Nemo is too big for CI, but passes locally
    # "mistralai/Mistral-Nemo-Instruct-2407"
]

SAMPLING_PARAMS = SamplingParams(max_tokens=512, temperature=0.0, logprobs=5)
SYMBOLIC_LANG_PROMPTS = [
    "勇敢な船乗りについての詩を書く",  # japanese
    "寫一首關於勇敢的水手的詩",  # chinese
# ... 112 lines omitted for brevity ...
                "required": ["company", "position"],
            },
        },
    },
    "required": ["name", "age", "skills", "work_history"],
}
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODELS`, `MISTRAL_FORMAT_MODELS`, `SAMPLING_PARAMS`, `SYMBOLIC_LANG_PROMPTS`, `TOOLS`, `MSGS`, `SAMPLE_JSON_SCHEMA`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODELS`、`MISTRAL_FORMAT_MODELS`、`SAMPLING_PARAMS`、`SYMBOLIC_LANG_PROMPTS`、`TOOLS`、`MSGS`、`SAMPLE_JSON_SCHEMA`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_models (L153-L182)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["bfloat16"])
@pytest.mark.parametrize("max_tokens", [64])
@pytest.mark.parametrize("num_logprobs", [5])
def test_models(
    hf_runner,
    vllm_runner,
    example_prompts,
    model: str,
    dtype: str,
    max_tokens: int,
    num_logprobs: int,
) -> None:
    # TODO(sang): Sliding window should be tested separately.
    with hf_runner(model, dtype=dtype) as hf_model:
        hf_outputs = hf_model.generate_greedy_logprobs_limit(
            example_prompts, max_tokens, num_logprobs
        )

    with vllm_runner(model, dtype=dtype, tokenizer_mode="mistral") as vllm_model:
        vllm_outputs = vllm_model.generate_greedy_logprobs(
            example_prompts, max_tokens, num_logprobs
        )

    check_logprobs_close(
        outputs_0_lst=hf_outputs,
        outputs_1_lst=vllm_outputs,
        name_0="hf",
        name_1="vllm",
    )
```
**EN:** This test validates `test_models`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `model`, `dtype`, `max_tokens`.
**CN:** 这个测试验证 `test_models`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`model`、`dtype`、`max_tokens`。

### Test / 测试: test_mistral_format (L185-L224)
```python
@pytest.mark.parametrize("model", MISTRAL_FORMAT_MODELS)
@pytest.mark.parametrize("dtype", ["bfloat16"])
@pytest.mark.parametrize("max_tokens", [64])
@pytest.mark.parametrize("num_logprobs", [5])
def test_mistral_format(
    vllm_runner,
    example_prompts,
    model: str,
    dtype: str,
    max_tokens: int,
    num_logprobs: int,
) -> None:
    with vllm_runner(
        model,
        dtype=dtype,
        tokenizer_mode="mistral",
        load_format="mistral",
        config_format="mistral",
# ... 14 lines omitted for brevity ...
        )

    check_logprobs_close(
        outputs_0_lst=hf_format_outputs,
        outputs_1_lst=mistral_format_outputs,
        name_0="hf",
        name_1="mistral",
    )
```
**EN:** This test validates `test_mistral_format`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `example_prompts`, `model`, `dtype`, `max_tokens`, `num_logprobs`.
**CN:** 这个测试验证 `test_mistral_format`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`example_prompts`、`model`、`dtype`、`max_tokens`、`num_logprobs`。

### Test / 测试: test_mistral_symbolic_languages (L227-L241)
```python
@pytest.mark.parametrize("model", MISTRAL_FORMAT_MODELS)
@pytest.mark.parametrize("dtype", ["bfloat16"])
def test_mistral_symbolic_languages(vllm_runner, model: str, dtype: str) -> None:
    with vllm_runner(
        model,
        dtype=dtype,
        max_model_len=8192,
        tokenizer_mode="mistral",
        config_format="mistral",
        load_format="mistral",
    ) as vllm_model:
        for prompt in SYMBOLIC_LANG_PROMPTS:
            msg = {"role": "user", "content": prompt}
            outputs = vllm_model.llm.chat([msg], sampling_params=SAMPLING_PARAMS)
            assert "�" not in outputs[0].outputs[0].text.strip()
```
**EN:** This test validates `test_mistral_symbolic_languages`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `model`, `dtype`. The main assertion is `'�' not in outputs[0].outputs[0].text.strip()`.
**CN:** 这个测试验证 `test_mistral_symbolic_languages`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`model`、`dtype`。 核心断言是 `'�' not in outputs[0].outputs[0].text.strip()`。

### Test / 测试: test_mistral_function_calling (L244-L274)
```python
@pytest.mark.parametrize("model", MISTRAL_FORMAT_MODELS)
@pytest.mark.parametrize("dtype", ["bfloat16"])
def test_mistral_function_calling(vllm_runner, model: str, dtype: str) -> None:
    with vllm_runner(
        model,
        dtype=dtype,
        tokenizer_mode="mistral",
        config_format="mistral",
        load_format="mistral",
    ) as vllm_model:
        msgs = copy.deepcopy(MSGS)
        outputs = vllm_model.llm.chat(
            msgs, tools=TOOLS, sampling_params=SAMPLING_PARAMS
        )

        tokenizer = vllm_model.llm.get_tokenizer()
        tool_parser = MistralToolParser(tokenizer)

# ... 5 lines omitted for brevity ...

        assert MistralToolCall.is_valid_id(parsed_message.tool_calls[0].id)
        assert parsed_message.tool_calls[0].function.name == "get_current_weather"
        assert (
            parsed_message.tool_calls[0].function.arguments
            == '{"city": "Dallas", "state": "TX", "unit": "fahrenheit"}'
        )  # noqa
        assert parsed_message.content is None
```
**EN:** This test validates `test_mistral_function_calling`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `model`, `dtype`. The main assertion is `model_output.startswith(tool_parser.bot_token)` and `parsed_message.tools_called`.
**CN:** 这个测试验证 `test_mistral_function_calling`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`model`、`dtype`。 核心断言是 `model_output.startswith(tool_parser.bot_token)` and `parsed_message.tools_called`。

### Test / 测试: test_mistral_function_call_nested_json (L277-L352)
```python
def test_mistral_function_call_nested_json():
    """Ensure that the function-name regex captures the entire outermost
    JSON block, including nested braces."""

    # Create a minimal stub tokenizer that provides the few attributes the
    # parser accesses (`version` and `get_vocab`).
    class _StubMistralTokenizer(MistralTokenizer):
        version = 11  # Satisfy the version check

        def __init__(self):
            pass

        @staticmethod
        def get_vocab():
            # Provide the special TOOL_CALLS token expected by the parser.
            return {"[TOOL_CALLS]": 0}

    tokenizer = _StubMistralTokenizer()
# ... 50 lines omitted for brevity ...
    assert len(parsed.tool_calls) == len(multiple_args_dict)

    for i, tool_call in enumerate(parsed.tool_calls):
        assert MistralToolCall.is_valid_id(tool_call.id)
        assert tool_call.function.name == names[i]
        assert json.loads(tool_call.function.arguments) == multiple_args_dict[i]
        # No additional content outside the tool call should be returned.
        assert parsed.content is None
```
**EN:** This test validates `test_mistral_function_call_nested_json`. The main assertion is `parsed.tools_called` and `MistralToolCall.is_valid_id(parsed.tool_calls[0].id)`.
**CN:** 这个测试验证 `test_mistral_function_call_nested_json`。 核心断言是 `parsed.tools_called` and `MistralToolCall.is_valid_id(parsed.tool_calls[0].id)`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `copy`, `json`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `vllm.sampling_params.SamplingParams`, `vllm.tokenizers.mistral.MistralTokenizer`, `vllm.tool_parsers.mistral_tool_parser.MistralToolCall`, `vllm.tool_parsers.mistral_tool_parser.MistralToolParser`
- **Local relative imports / 本地相对导入**: `...utils.check_logprobs_close`
