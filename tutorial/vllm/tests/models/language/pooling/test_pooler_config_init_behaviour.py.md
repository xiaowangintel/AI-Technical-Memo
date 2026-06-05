# test_pooler_config_init_behaviour.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/pooling/test_pooler_config_init_behaviour.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and language-model behavior. The file defines 4 test(s), 0 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与语言模型行为。它定义了 4 个测试、0 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L8)
```python
import pytest
import torch
import torch.nn.functional as F

from tests.models.utils import softmax
from vllm.config import PoolerConfig
```
**EN:** Imports third-party packages like `pytest`, `torch`, `torch.nn.functional`, project helpers such as `tests.models.utils.softmax`, `vllm.config.PoolerConfig`.
**CN:** 导入第三方包（如 `pytest`、`torch`、`torch.nn.functional`）、项目内辅助模块（如 `tests.models.utils.softmax`、`vllm.config.PoolerConfig`）。

### Test / 测试: test_classify_models_using_activation (L11-L48)
```python
@pytest.mark.parametrize(
    "model",
    ["jason9693/Qwen2.5-1.5B-apeach", "papluca/xlm-roberta-base-language-detection"],
)
@pytest.mark.parametrize("dtype", ["half"])
def test_classify_models_using_activation(
    hf_runner,
    vllm_runner,
    example_prompts,
    model: str,
    dtype: str,
) -> None:
    with vllm_runner(
        model,
        max_model_len=512,
        dtype=dtype,
        pooler_config=PoolerConfig(use_activation=False),
    ) as vllm_model:
# ... 12 lines omitted for brevity ...
        w_activation = torch.tensor(w_activation)

        assert not torch.allclose(wo_activation, w_activation, atol=1e-2), (
            "pooler_config is not working"
        )
        assert torch.allclose(
            softmax(wo_activation), w_activation, 1e-3 if dtype == "float" else 1e-2
        )
```
**EN:** This test validates `test_classify_models_using_activation`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `model`, `dtype`. The main assertion is `not torch.allclose(wo_activation, w_activation, atol=0.01)` and `torch.allclose(softmax(wo_activation), w_activation, 0.001 if dtype == 'float' else 0.01)`.
**CN:** 这个测试验证 `test_classify_models_using_activation`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`model`、`dtype`。 核心断言是 `not torch.allclose(wo_activation, w_activation, atol=0.01)` and `torch.allclose(softmax(wo_activation), w_activation, 0.001 if dtype == 'float' else 0.01)`。

### Test / 测试: test_embed_models_using_normalize (L51-L86)
```python
@pytest.mark.parametrize(
    "model",
    [
        "intfloat/multilingual-e5-small",
    ],
)
@pytest.mark.parametrize("dtype", ["half"])
def test_embed_models_using_normalize(
    hf_runner,
    vllm_runner,
    example_prompts,
    model: str,
    dtype: str,
) -> None:
    with vllm_runner(
        model,
        max_model_len=512,
        dtype=dtype,
# ... 10 lines omitted for brevity ...
        w_normalize = torch.tensor(vllm_model.embed(example_prompts))

    assert not torch.allclose(wo_normalize, w_normalize, atol=1e-2), (
        "pooler_config normalize is not working"
    )
    assert torch.allclose(
        F.normalize(wo_normalize, p=2, dim=-1), w_normalize, atol=1e-2
    ), "w_normal should be close to normal(wo_normal)."
```
**EN:** This test validates `test_embed_models_using_normalize`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `model`, `dtype`. The main assertion is `not torch.allclose(wo_normalize, w_normalize, atol=0.01)` and `torch.allclose(F.normalize(wo_normalize, p=2, dim=-1), w_normalize, atol=0.01)`.
**CN:** 这个测试验证 `test_embed_models_using_normalize`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`model`、`dtype`。 核心断言是 `not torch.allclose(wo_normalize, w_normalize, atol=0.01)` and `torch.allclose(F.normalize(wo_normalize, p=2, dim=-1), w_normalize, atol=0.01)`。

### Test / 测试: test_reward_models_using_activation (L89-L128)
```python
@pytest.mark.parametrize(
    "model",
    [
        "internlm/internlm2-1_8b-reward",
    ],
)
@pytest.mark.parametrize("dtype", ["half"])
def test_reward_models_using_activation(
    hf_runner,
    vllm_runner,
    example_prompts,
    model: str,
    dtype: str,
) -> None:
    with vllm_runner(
        model,
        max_model_len=1024,
        dtype=dtype,
# ... 14 lines omitted for brevity ...
        w = torch.tensor(w)

        assert not torch.allclose(wo, w, atol=1e-2), (
            "pooler_config activation is not working"
        )
        assert torch.allclose(softmax(wo), w, atol=1e-2), (
            "w_activation should be close to activation(wo_activation)."
        )
```
**EN:** This test validates `test_reward_models_using_activation`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `model`, `dtype`. The main assertion is `not torch.allclose(wo, w, atol=0.01)` and `torch.allclose(softmax(wo), w, atol=0.01)`.
**CN:** 这个测试验证 `test_reward_models_using_activation`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`model`、`dtype`。 核心断言是 `not torch.allclose(wo, w, atol=0.01)` and `torch.allclose(softmax(wo), w, atol=0.01)`。

### Test / 测试: test_multi_vector_retrieval_models_using_normalize (L131-L167)
```python
@pytest.mark.parametrize(
    "model",
    [
        "intfloat/multilingual-e5-small",
    ],
)
@pytest.mark.parametrize("dtype", ["half"])
def test_multi_vector_retrieval_models_using_normalize(
    hf_runner,
    vllm_runner,
    example_prompts,
    model: str,
    dtype: str,
) -> None:
    with vllm_runner(
        model,
        max_model_len=512,
        dtype=dtype,
# ... 11 lines omitted for brevity ...

    for wo, w in zip(wo_normalize, w_normalize):
        assert not torch.allclose(wo, w, atol=1e-2), (
            "pooler_config normalize is not working"
        )
        assert torch.allclose(F.normalize(wo, p=2, dim=-1), w, atol=1e-2), (
            "w_normal should be close to normal(wo_normal)."
        )
```
**EN:** This test validates `test_multi_vector_retrieval_models_using_normalize`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `model`, `dtype`. The main assertion is `not torch.allclose(wo, w, atol=0.01)` and `torch.allclose(F.normalize(wo, p=2, dim=-1), w, atol=0.01)`.
**CN:** 这个测试验证 `test_multi_vector_retrieval_models_using_normalize`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`model`、`dtype`。 核心断言是 `not torch.allclose(wo, w, atol=0.01)` and `torch.allclose(F.normalize(wo, p=2, dim=-1), w, atol=0.01)`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `pytest`, `torch`, `torch.nn.functional`
- **Project / 项目内**: `tests.models.utils.softmax`, `vllm.config.PoolerConfig`
