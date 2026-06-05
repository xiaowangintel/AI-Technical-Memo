# test_struct_output_generate.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/llm/test_struct_output_generate.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers LLM entrypoint behavior. The file defines 6 test(s), 0 fixture(s), and 3 helper/class block(s) to validate this area. / [CN] 该文件覆盖LLM 入口行为。它定义了 6 个测试、0 个 fixture，以及 3 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L25)
```python
import json
from enum import Enum
from typing import Any

import jsonschema
import pytest
import regex as re
import torch
from pydantic import BaseModel

from tests.reasoning.utils import run_reasoning_extraction
from vllm.config import StructuredOutputsConfig
from vllm.distributed import cleanup_dist_env_and_memory
from vllm.entrypoints.llm import LLM
from vllm.outputs import RequestOutput
from vllm.platforms import current_platform
from vllm.reasoning.abs_reasoning_parsers import ReasoningParserManager
from vllm.sampling_params import (
    SamplingParams,
    StructuredOutputsParams,
)
```
**EN:** Imports standard-library modules such as `enum.Enum`, `json`, `typing.Any`, third-party packages like `jsonschema`, `lark.Lark`, `pydantic.BaseModel`, project helpers such as `tests.reasoning.utils.run_reasoning_extraction`, `vllm.config.StructuredOutputsConfig`, `vllm.distributed.cleanup_dist_env_and_memory`.
**CN:** 导入标准库模块（如 `enum.Enum`、`json`、`typing.Any`）、第三方包（如 `jsonschema`、`lark.Lark`、`pydantic.BaseModel`）、项目内辅助模块（如 `tests.reasoning.utils.run_reasoning_extraction`、`vllm.config.StructuredOutputsConfig`、`vllm.distributed.cleanup_dist_env_and_memory`）。

### Module setup / 模块级配置: SAMPLE_REGEX, SAMPLE_JSON_SCHEMA, UNSUPPORTED_JSON_SCHEMA (L27-L194)
```python
SAMPLE_REGEX = (
    r"((25[0-5]|(2[0-4]|1\d|[1-9]|)\d)\.){3}"
    r"(25[0-5]|(2[0-4]|1\d|[1-9]|)\d)"
)

# Note: Ensure this only uses attributes compatible with xgrammar
SAMPLE_JSON_SCHEMA = {
    "type": "object",
    "properties": {
        "name": {"type": "string"},
        "age": {"type": "integer"},
        "skills": {
            "type": "array",
            "items": {
                "type": "string",
            },
# ... 146 lines omitted for brevity ...
    ("Qwen/Qwen2.5-1.5B-Instruct", "auto"),
]

platform_args = {}
if current_platform.is_rocm():
    platform_args["async_scheduling"] = False
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `SAMPLE_REGEX`, `SAMPLE_JSON_SCHEMA`, `UNSUPPORTED_JSON_SCHEMA`, `SAMPLE_STRUCTURED_OUTPUTS_CHOICES`, `SAMPLE_SQL_EBNF`, `SAMPLE_SQL_LARK`, `NGRAM_SPEC_CONFIG`, `EAGLE_SPEC_CONFIG`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `SAMPLE_REGEX`、`SAMPLE_JSON_SCHEMA`、`UNSUPPORTED_JSON_SCHEMA`、`SAMPLE_STRUCTURED_OUTPUTS_CHOICES`、`SAMPLE_SQL_EBNF`、`SAMPLE_SQL_LARK`、`NGRAM_SPEC_CONFIG`、`EAGLE_SPEC_CONFIG`。后续辅助函数和测试会复用这些值，以减少重复。

### Class / 类: CarType (L197-L201)
```python
class CarType(str, Enum):
    sedan = "sedan"
    suv = "SUV"
    truck = "Truck"
    coupe = "Coupe"
```
**EN:** This class groups related scenarios in `CarType`. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `CarType` 相关的场景组织在一起。 其主体主要是结构性定义，没有直接方法实现。

### Class / 类: CarDescription (L204-L207)
```python
class CarDescription(BaseModel):
    brand: str
    model: str
    car_type: CarType
```
**EN:** This class groups related scenarios in `CarDescription`. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `CarDescription` 相关的场景组织在一起。 其主体主要是结构性定义，没有直接方法实现。

### Test / 测试: test_structured_output (L210-L714)
```python
@pytest.mark.parametrize(
    "model_name, backend, tokenizer_mode, speculative_config",
    PARAMS_MODELS_BACKENDS_TOKENIZER_MODE,
)
def test_structured_output(
    backend: str,
    tokenizer_mode: str,
    model_name: str,
    speculative_config: dict[str, Any],
):
    sample_json_schema = SAMPLE_JSON_SCHEMA
    unsupported_json_schema = UNSUPPORTED_JSON_SCHEMA
    sample_sql_ebnf = SAMPLE_SQL_EBNF
    sample_sql_lark = SAMPLE_SQL_LARK
    sample_regex = SAMPLE_REGEX
    sample_structured_outputs_choices = SAMPLE_STRUCTURED_OUTPUTS_CHOICES
    if current_platform.is_tpu() and speculative_config:
        pytest.skip("TPU does not support speculative decoding")
# ... 479 lines omitted for brevity ...
                json_content = json.loads(json_str)
                assert "city" in json_content
                assert isinstance(json_content["city"], str)
                print(f"Found valid function call: {generated_text!r}")
            except (json.JSONDecodeError, AssertionError) as e:
                pytest.fail(
                    f"Invalid function call format: {generated_text!r}\nError: {str(e)}"
                )
```
**EN:** This test validates `test_structured_output`. It uses parameterization over `model_name`, `backend`, `tokenizer_mode`, `speculative_config`. Key inputs are `backend`, `tokenizer_mode`, `model_name`, `speculative_config`. It checks an expected failure path with `pytest.raises`. It touches the core vLLM initialization or engine path directly. The logic can skip unsupported environments when prerequisites are not satisfied. The main assertion is `outputs is not None` and `outputs is not None`.
**CN:** 这个测试验证 `test_structured_output`。 它通过参数化组合 `model_name`、`backend`、`tokenizer_mode`、`speculative_config`。 关键输入包括 `backend`、`tokenizer_mode`、`model_name`、`speculative_config`。 它使用 `pytest.raises` 检查预期失败路径。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 核心断言是 `outputs is not None` and `outputs is not None`。

### Test / 测试: test_structured_output_with_reasoning_matrices (L717-L802)
```python
@pytest.mark.parametrize(
    "model_name, backend, tokenizer_mode, reasoning_parser, speculative_config, async_scheduling",  # noqa: E501
    [
        (
            "deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B",
            "xgrammar",
            "auto",
            "deepseek_r1",
            NGRAM_SPEC_CONFIG,
            False,
        ),
        ("Qwen/Qwen3-1.7B", "xgrammar", "auto", "deepseek_r1", None, False),
        ("Qwen/Qwen3-1.7B", "xgrammar", "auto", "deepseek_r1", None, True),
    ],
)
def test_structured_output_with_reasoning_matrices(
    backend: str,
    tokenizer_mode: str,
# ... 60 lines omitted for brevity ...
    if "Qwen3" in model_name:
        assert content is not None

    assert reasoning is not None

    if content is not None:
        output_json = json.loads(content)
        jsonschema.validate(instance=output_json, schema=reasoning_schema)
```
**EN:** This test validates `test_structured_output_with_reasoning_matrices`. It uses parameterization over `model_name`, `backend`, `tokenizer_mode`, `reasoning_parser`, `speculative_config`, `async_scheduling`. Key inputs are `backend`, `tokenizer_mode`, `reasoning_parser`, `model_name`, `speculative_config`, `async_scheduling`. It touches the core vLLM initialization or engine path directly. The logic can skip unsupported environments when prerequisites are not satisfied. The main assertion is `outputs is not None` and `output is not None and isinstance(output, RequestOutput)`.
**CN:** 这个测试验证 `test_structured_output_with_reasoning_matrices`。 它通过参数化组合 `model_name`、`backend`、`tokenizer_mode`、`reasoning_parser`、`speculative_config`、`async_scheduling`。 关键输入包括 `backend`、`tokenizer_mode`、`reasoning_parser`、`model_name`、`speculative_config`、`async_scheduling`。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 核心断言是 `outputs is not None` and `output is not None and isinstance(output, RequestOutput)`。

### Test / 测试: test_structured_output_auto_mode (L805-L850)
```python
@pytest.mark.parametrize("model_name, tokenizer_mode", PARAMS_MODELS_TOKENIZER_MODE)
def test_structured_output_auto_mode(
    model_name: str,
    tokenizer_mode: str,
):
    unsupported_json_schema = UNSUPPORTED_JSON_SCHEMA
    llm = LLM(
        model=model_name,
        max_model_len=1024,
        structured_outputs_config=dict(backend="auto"),
        tokenizer_mode=tokenizer_mode,
        load_format="auto",
        config_format="auto",
    )

    sampling_params = SamplingParams(
        temperature=1.0,
        max_tokens=1000,
# ... 20 lines omitted for brevity ...
        assert isinstance(output, RequestOutput)
        generated_text = output.outputs[0].text
        assert generated_text is not None
        print(generated_text)

        # Parse to verify it is valid JSON
        parsed_json = json.loads(generated_text)
        assert isinstance(parsed_json, dict)
```
**EN:** This test validates `test_structured_output_auto_mode`. It uses parameterization over `model_name`, `tokenizer_mode`. Key inputs are `model_name`, `tokenizer_mode`. It touches the core vLLM initialization or engine path directly. The main assertion is `outputs is not None` and `output is not None`.
**CN:** 这个测试验证 `test_structured_output_auto_mode`。 它通过参数化组合 `model_name`、`tokenizer_mode`。 关键输入包括 `model_name`、`tokenizer_mode`。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 核心断言是 `outputs is not None` and `output is not None`。

### Test / 测试: test_guidance_no_additional_properties (L853-L908)
```python
def test_guidance_no_additional_properties():
    llm = LLM(
        model="Qwen/Qwen2.5-1.5B-Instruct",
        max_model_len=1024,
        structured_outputs_config=dict(
            backend="guidance",
            disable_any_whitespace=True,
            disable_additional_properties=True,
        ),
    )

    schema = {
        "type": "object",
        "properties": {
            "a1": {"type": "string"},
            "a2": {"type": "string"},
            "a3": {"type": "string"},
        },
# ... 30 lines omitted for brevity ...

    generated = generate_with_backend("guidance")
    assert "a1" in generated
    assert "a2" in generated
    assert "a3" in generated
    assert "a4" not in generated
    assert "a5" not in generated
    assert "a6" not in generated
```
**EN:** This test validates `test_guidance_no_additional_properties`. It touches the core vLLM initialization or engine path directly. The main assertion is `'a1' in generated` and `'a2' in generated`.
**CN:** 这个测试验证 `test_guidance_no_additional_properties`。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 核心断言是 `'a1' in generated` and `'a2' in generated`。

### Test / 测试: test_structured_output_batched_with_non_structured_outputs_requests (L911-L986)
```python
@pytest.mark.parametrize("backend", ["guidance", "xgrammar", "outlines"])
def test_structured_output_batched_with_non_structured_outputs_requests(
    backend: str,
):
    sample_json_schema = SAMPLE_JSON_SCHEMA
    # Don't use eager execution on TPUs because we want to test for no
    # recompilation at runtime
    enforce_eager = bool(not current_platform.is_tpu())

    llm = LLM(
        model="meta-llama/Meta-Llama-3.1-8B-Instruct",
        enforce_eager=enforce_eager,
        max_model_len=1024,
        structured_outputs_config=StructuredOutputsConfig(
            backend=backend,
            disable_any_whitespace=backend in {"xgrammar", "guidance"},
        ),
    )
# ... 50 lines omitted for brevity ...
        else:
            # Second prompt is not structured outputs, expect valid output
            # Cannot assert on exact output, but we can expect it to be factual
            assert "12,742" in generated_text

            # non-structured outputs requests should not return a valid JSON here
            with pytest.raises(ValueError):
                output_json = json.loads(generated_text)
```
**EN:** This test validates `test_structured_output_batched_with_non_structured_outputs_requests`. It uses parameterization over `backend`. Key inputs are `backend`. It checks an expected failure path with `pytest.raises`. It touches the core vLLM initialization or engine path directly. The main assertion is `outputs is not None` and `output is not None`.
**CN:** 这个测试验证 `test_structured_output_batched_with_non_structured_outputs_requests`。 它通过参数化组合 `backend`。 关键输入包括 `backend`。 它使用 `pytest.raises` 检查预期失败路径。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 核心断言是 `outputs is not None` and `output is not None`。

### Test / 测试: test_structured_output_with_structural_tag (L989-L1027)
```python
@pytest.mark.parametrize("backend", ["xgrammar"])
def test_structured_output_with_structural_tag(backend: str):
    llm = LLM(
        model="Qwen/Qwen2.5-1.5B-Instruct",
        structured_outputs_config=StructuredOutputsConfig(backend=backend),
    )

    structural_tag_config = {
        "type": "structural_tag",
        "format": {
            "type": "triggered_tags",
            "tags": [
                {"begin": "hello_flag", "content": {"type": "any_text"}, "end": "hello"}
            ],
            "triggers": ["hello"],
            "stop_after_first": False,
        },
    }
# ... 13 lines omitted for brevity ...
        assert output is not None
        assert isinstance(output, RequestOutput)
        prompt = output.prompt
        generated_text = output.outputs[0].text
        assert generated_text is not None
        assert "hello_flag" in generated_text, (
            f"Expected 'hello_flag' to be in generated text, but got: {generated_text}"
        )
```
**EN:** This test validates `test_structured_output_with_structural_tag`. It uses parameterization over `backend`. Key inputs are `backend`. It touches the core vLLM initialization or engine path directly. The main assertion is `outputs is not None` and `output is not None`.
**CN:** 这个测试验证 `test_structured_output_with_structural_tag`。 它通过参数化组合 `backend`。 关键输入包括 `backend`。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 核心断言是 `outputs is not None` and `output is not None`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `enum.Enum`, `json`, `typing.Any`
- **Third-party / 第三方**: `jsonschema`, `lark.Lark`, `pydantic.BaseModel`, `pytest`, `regex`, `torch`
- **Project / 项目内**: `tests.reasoning.utils.run_reasoning_extraction`, `vllm.config.StructuredOutputsConfig`, `vllm.distributed.cleanup_dist_env_and_memory`, `vllm.entrypoints.llm.LLM`, `vllm.outputs.RequestOutput`, `vllm.platforms.current_platform`, `vllm.reasoning.abs_reasoning_parsers.ReasoningParserManager`, `vllm.sampling_params.SamplingParams`, `vllm.sampling_params.StructuredOutputsParams`
