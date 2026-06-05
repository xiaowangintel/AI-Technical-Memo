# test_nomic_max_model_len.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/pooling/test_nomic_max_model_len.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and language-model behavior. The file defines 5 test(s), 0 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与语言模型行为。它定义了 5 个测试、0 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L8)
```python
from typing import Any

import pytest

from ...utils import EmbedModelInfo
```
**EN:** Imports standard-library modules such as `typing.Any`, third-party packages like `pytest`, project helpers such as `...utils.EmbedModelInfo`.
**CN:** 导入标准库模块（如 `typing.Any`）、第三方包（如 `pytest`）、项目内辅助模块（如 `...utils.EmbedModelInfo`）。

### Module setup / 模块级配置: MODELS, rope_theta, factor (L10-L27)
```python
MODELS = [
    EmbedModelInfo(
        "nomic-ai/nomic-embed-text-v1",
        # Fixme:
        #  Update nomic-embed code to support the latest
        #  HF version and remove revision set.
        revision="720244025c1a7e15661a174c63cce63c8218e52b",
    ),
    # EmbedModelInfo("nomic-ai/nomic-embed-text-v1.5"),
    # EmbedModelInfo("nomic-ai/CodeRankEmbed"),
    EmbedModelInfo("nomic-ai/nomic-embed-text-v2-moe"),
    # EmbedModelInfo("Snowflake/snowflake-arctic-embed-m-long"),
]

rope_theta = 1000
factor = 4.0
original_max_position_embeddings = 2048
max_model_len = int(original_max_position_embeddings * factor)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODELS`, `rope_theta`, `factor`, `original_max_position_embeddings`, `max_model_len`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODELS`、`rope_theta`、`factor`、`original_max_position_embeddings`、`max_model_len`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_default (L30-L44)
```python
@pytest.mark.parametrize("model_info", MODELS)
def test_default(model_info, vllm_runner):
    with vllm_runner(
        model_info.name,
        revision=model_info.revision,
        runner="pooling",
        max_model_len=None,
    ) as vllm_model:
        model_config = vllm_model.llm.llm_engine.model_config
        if model_info.name == "nomic-ai/nomic-embed-text-v2-moe":
            # For nomic-embed-text-v2-moe the length is set to 512
            # by sentence_bert_config.json.
            assert model_config.max_model_len == 512
        else:
            assert model_config.max_model_len == original_max_position_embeddings
```
**EN:** This test validates `test_default`. It uses parameterization over `model_info`. Key inputs are `model_info`, `vllm_runner`. The main assertion is `model_config.max_model_len == 512` and `model_config.max_model_len == original_max_position_embeddings`.
**CN:** 这个测试验证 `test_default`。 它通过参数化组合 `model_info`。 关键输入包括 `model_info`、`vllm_runner`。 核心断言是 `model_config.max_model_len == 512` and `model_config.max_model_len == original_max_position_embeddings`。

### Test / 测试: test_set_max_model_len_legal (L47-L79)
```python
@pytest.mark.parametrize("model_info", MODELS)
def test_set_max_model_len_legal(model_info, vllm_runner):
    # set max_model_len <= 512
    with vllm_runner(
        model_info.name,
        revision=model_info.revision,
        runner="pooling",
        max_model_len=256,
    ) as vllm_model:
        model_config = vllm_model.llm.llm_engine.model_config
        assert model_config.max_model_len == 256

    # set 512 < max_model_len <= 2048
    if model_info.name == "nomic-ai/nomic-embed-text-v2-moe":
        # For nomic-embed-text-v2-moe the length is set to 512
        # by sentence_bert_config.json.
        with pytest.raises(ValueError):
            with vllm_runner(
# ... 7 lines omitted for brevity ...
        with vllm_runner(
            model_info.name,
            revision=model_info.revision,
            runner="pooling",
            max_model_len=1024,
        ) as vllm_model:
            model_config = vllm_model.llm.llm_engine.model_config
            assert model_config.max_model_len == 1024
```
**EN:** This test validates `test_set_max_model_len_legal`. It uses parameterization over `model_info`. Key inputs are `model_info`, `vllm_runner`. It checks an expected failure path with `pytest.raises`. The main assertion is `model_config.max_model_len == 256` and `model_config.max_model_len == 1024`.
**CN:** 这个测试验证 `test_set_max_model_len_legal`。 它通过参数化组合 `model_info`。 关键输入包括 `model_info`、`vllm_runner`。 它使用 `pytest.raises` 检查预期失败路径。 核心断言是 `model_config.max_model_len == 256` and `model_config.max_model_len == 1024`。

### Test / 测试: test_set_max_model_len_illegal (L82-L104)
```python
@pytest.mark.parametrize("model_info", MODELS)
def test_set_max_model_len_illegal(model_info, vllm_runner):
    # set max_model_len > 2048
    with pytest.raises(ValueError):
        with vllm_runner(
            model_info.name,
            revision=model_info.revision,
            runner="pooling",
            max_model_len=4096,
        ):
            pass

    # set max_model_len > 2048 by hf_overrides
    hf_overrides = {"max_model_len": 4096}
    with pytest.raises(ValueError):
        with vllm_runner(
            model_info.name,
            revision=model_info.revision,
            runner="pooling",
            max_model_len=None,
            hf_overrides=hf_overrides,
        ):
            pass
```
**EN:** This test validates `test_set_max_model_len_illegal`. It uses parameterization over `model_info`. Key inputs are `model_info`, `vllm_runner`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `test_set_max_model_len_illegal`。 它通过参数化组合 `model_info`。 关键输入包括 `model_info`、`vllm_runner`。 它使用 `pytest.raises` 检查预期失败路径。

### Test / 测试: test_use_rope_scaling_legal (L107-L126)
```python
@pytest.mark.parametrize("model_info", MODELS)
def test_use_rope_scaling_legal(model_info, vllm_runner):
    hf_overrides = {
        "rope_parameters": {
            "rope_theta": rope_theta,
            "rope_type": "yarn",
            "factor": factor,
            "original_max_position_embeddings": original_max_position_embeddings,
        },
        "max_model_len": max_model_len,
    }

    with vllm_runner(
        model_info.name,
        revision=model_info.revision,
        runner="pooling",
        max_model_len=None,
        hf_overrides=hf_overrides,
    ):
        pass
```
**EN:** This test validates `test_use_rope_scaling_legal`. It uses parameterization over `model_info`. Key inputs are `model_info`, `vllm_runner`.
**CN:** 这个测试验证 `test_use_rope_scaling_legal`。 它通过参数化组合 `model_info`。 关键输入包括 `model_info`、`vllm_runner`。

### Test / 测试: test_use_rope_scaling_illegal (L129-L168)
```python
@pytest.mark.parametrize("model_info", MODELS)
def test_use_rope_scaling_illegal(model_info, vllm_runner):
    hf_overrides: dict[str, Any] = {
        "rope_parameters": {
            "rope_theta": rope_theta,
            "rope_type": "yarn",
            "factor": factor,
            "original_max_position_embeddings": original_max_position_embeddings,
        },
    }
    # illegal max_model_len
    with pytest.raises(ValueError):
        with vllm_runner(
            model_info.name,
            revision=model_info.revision,
            runner="pooling",
            max_model_len=max_model_len + 1,
            hf_overrides=hf_overrides,
# ... 14 lines omitted for brevity ...
        with vllm_runner(
            model_info.name,
            revision=model_info.revision,
            runner="pooling",
            max_model_len=None,
            hf_overrides=hf_overrides,
        ):
            pass
```
**EN:** This test validates `test_use_rope_scaling_illegal`. It uses parameterization over `model_info`. Key inputs are `model_info`, `vllm_runner`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `test_use_rope_scaling_illegal`。 它通过参数化组合 `model_info`。 关键输入包括 `model_info`、`vllm_runner`。 它使用 `pytest.raises` 检查预期失败路径。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `typing.Any`
- **Third-party / 第三方**: `pytest`
- **Local relative imports / 本地相对导入**: `...utils.EmbedModelInfo`
