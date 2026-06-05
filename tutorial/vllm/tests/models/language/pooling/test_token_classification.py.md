# test_token_classification.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/pooling/test_token_classification.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and language-model behavior. The file defines 3 test(s), 1 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与语言模型行为。它定义了 3 个测试、1 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L10)
```python
import pytest
import torch
from transformers import AutoModelForTokenClassification

from tests.models.utils import softmax
from vllm.platforms import current_platform
from vllm.utils.torch_utils import set_random_seed
```
**EN:** Imports third-party packages like `pytest`, `torch`, `transformers.AutoModelForTokenClassification`, project helpers such as `tests.models.utils.softmax`, `vllm.platforms.current_platform`, `vllm.utils.torch_utils.set_random_seed`.
**CN:** 导入第三方包（如 `pytest`、`torch`、`transformers.AutoModelForTokenClassification`）、项目内辅助模块（如 `tests.models.utils.softmax`、`vllm.platforms.current_platform`、`vllm.utils.torch_utils.set_random_seed`）。

### Fixture / 夹具: seed_everything (L13-L20)
```python
@pytest.fixture(autouse=True)
def seed_everything():
    """Seed all random number generators for reproducibility."""
    seed = 0
    set_random_seed(seed)
    torch.backends.cudnn.deterministic = True
    torch.backends.cudnn.benchmark = False
    yield
```
**EN:** This fixture prepares `seed_everything` for dependent tests. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `seed_everything`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_bert_like_models (L23-L67)
```python
@pytest.mark.parametrize(
    "model",
    [
        "boltuix/NeuroBERT-NER",
        "gyr66/Ernie-3.0-base-chinese-finetuned-ner",
    ],
)
# The float32 is required for this tiny model to pass the test.
@pytest.mark.parametrize("dtype", ["float"])
@torch.inference_mode
def test_bert_like_models(
    hf_runner,
    vllm_runner,
    example_prompts,
    model: str,
    dtype: str,
) -> None:
    with vllm_runner(model, max_model_len=None, dtype=dtype) as vllm_model:
# ... 19 lines omitted for brevity ...
            output = hf_model.model(**inputs)
            hf_outputs.append(softmax(output.logits[0]))

    # check logits difference
    for hf_output, vllm_output in zip(hf_outputs, vllm_outputs):
        hf_output = hf_output.detach().clone().cpu().float()
        vllm_output = vllm_output.detach().clone().cpu().float()
        torch.testing.assert_close(hf_output, vllm_output, atol=3.2e-2, rtol=1e-3)
```
**EN:** This test validates `test_bert_like_models`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `model`, `dtype`.
**CN:** 这个测试验证 `test_bert_like_models`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`model`、`dtype`。

### Test / 测试: test_modernbert_models (L70-L117)
```python
@pytest.mark.parametrize("model", ["disham993/electrical-ner-ModernBERT-base"])
@pytest.mark.parametrize("dtype", ["float"])
@pytest.mark.flaky(reruns=3)
@torch.inference_mode
def test_modernbert_models(
    hf_runner,
    vllm_runner,
    example_prompts,
    model: str,
    dtype: str,
) -> None:
    # NOTE: https://github.com/vllm-project/vllm/pull/32403
    # `disham993/electrical-ner-ModernBERT-base` is a randomly initialized
    # model, which can cause numerical precision variance and edge cases.
    # We use @flaky(reruns=3) to mitigate intermittent failures.
    print(
        f"\n[NOTE] Testing {model} (randomly initialized weights) - "
        "flaky tolerance enabled due to numerical precision variance."
# ... 22 lines omitted for brevity ...
            output = hf_model.model(**inputs)
            hf_outputs.append(softmax(output.logits[0]))

    # check logits difference
    for hf_output, vllm_output in zip(hf_outputs, vllm_outputs):
        hf_output = hf_output.detach().clone().cpu().float()
        vllm_output = vllm_output.detach().clone().cpu().float()
        torch.testing.assert_close(hf_output, vllm_output, atol=3.2e-2, rtol=1e-3)
```
**EN:** This test validates `test_modernbert_models`. It uses parameterization over `model`. Relevant pytest markers include `flaky`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `model`, `dtype`.
**CN:** 这个测试验证 `test_modernbert_models`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `flaky`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`model`、`dtype`。

### Test / 测试: test_auto_conversion (L120-L148)
```python
@pytest.mark.parametrize("model", ["bd2lcco/Qwen3-0.6B-finetuned"])
@pytest.mark.parametrize("dtype", ["float"])
@torch.inference_mode
def test_auto_conversion(
    hf_runner,
    vllm_runner,
    example_prompts,
    model: str,
    dtype: str,
) -> None:
    with vllm_runner(model, max_model_len=1024, dtype=dtype) as vllm_model:
        vllm_outputs = vllm_model.token_classify(example_prompts)

    with hf_runner(
        model, dtype=dtype, auto_cls=AutoModelForTokenClassification
    ) as hf_model:
        tokenizer = hf_model.tokenizer
        hf_outputs = []
        for prompt in example_prompts:
            inputs = tokenizer([prompt], return_tensors="pt")
            inputs = hf_model.wrap_device(inputs)
            output = hf_model.model(**inputs)
            hf_outputs.append(softmax(output.logits[0]))

    # check logits difference
    for hf_output, vllm_output in zip(hf_outputs, vllm_outputs):
        hf_output = hf_output.detach().clone().cpu().float()
        vllm_output = vllm_output.detach().clone().cpu().float()
        assert torch.allclose(hf_output, vllm_output, atol=1e-2)
```
**EN:** This test validates `test_auto_conversion`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `model`, `dtype`. The main assertion is `torch.allclose(hf_output, vllm_output, atol=0.01)`.
**CN:** 这个测试验证 `test_auto_conversion`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`model`、`dtype`。 核心断言是 `torch.allclose(hf_output, vllm_output, atol=0.01)`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `pytest`, `torch`, `transformers.AutoModelForTokenClassification`
- **Project / 项目内**: `tests.models.utils.softmax`, `vllm.platforms.current_platform`, `vllm.utils.torch_utils.set_random_seed`
