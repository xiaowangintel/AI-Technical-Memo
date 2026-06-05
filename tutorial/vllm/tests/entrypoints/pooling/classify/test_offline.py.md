# test_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/classify/test_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior. The file defines 6 test(s), 1 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为。它定义了 6 个测试、1 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L11)
```python
import weakref

import pytest
import torch

from tests.models.utils import softmax
from vllm import LLM, ClassificationRequestOutput, PoolingParams
from vllm.distributed import cleanup_dist_env_and_memory
from vllm.tasks import PoolingTask
```
**EN:** Imports standard-library modules such as `weakref`, third-party packages like `pytest`, `torch`, project helpers such as `tests.models.utils.softmax`, `vllm.ClassificationRequestOutput`, `vllm.LLM`.
**CN:** 导入标准库模块（如 `weakref`）、第三方包（如 `pytest`、`torch`）、项目内辅助模块（如 `tests.models.utils.softmax`、`vllm.ClassificationRequestOutput`、`vllm.LLM`）。

### Module setup / 模块级配置: MODEL_NAME, prompt, prompt_token_ids (L13-L17)
```python
MODEL_NAME = "jason9693/Qwen2.5-1.5B-apeach"

prompt = "The chef prepared a delicious meal."
prompt_token_ids = [785, 29706, 10030, 264, 17923, 15145, 13]
num_labels = 2
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `prompt`, `prompt_token_ids`, `num_labels`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`prompt`、`prompt_token_ids`、`num_labels`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: llm (L20-L37)
```python
@pytest.fixture(scope="module")
def llm():
    # pytest caches the fixture so we use weakref.proxy to
    # enable garbage collection
    llm = LLM(
        model=MODEL_NAME,
        max_num_batched_tokens=32768,
        tensor_parallel_size=1,
        gpu_memory_utilization=0.75,
        enforce_eager=True,
        seed=0,
    )

    yield weakref.proxy(llm)

    del llm

    cleanup_dist_env_and_memory()
```
**EN:** This fixture prepares `llm` for dependent tests. It touches the core vLLM initialization or engine path directly. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `llm`。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_str_prompts (L40-L46)
```python
@pytest.mark.skip_global_cleanup
def test_str_prompts(llm: LLM):
    outputs = llm.classify(prompt, use_tqdm=False)
    assert len(outputs) == 1
    assert isinstance(outputs[0], ClassificationRequestOutput)
    assert outputs[0].prompt_token_ids == prompt_token_ids
    assert len(outputs[0].outputs.probs) == num_labels
```
**EN:** This test validates `test_str_prompts`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. The main assertion is `len(outputs) == 1` and `isinstance(outputs[0], ClassificationRequestOutput)`.
**CN:** 这个测试验证 `test_str_prompts`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 核心断言是 `len(outputs) == 1` and `isinstance(outputs[0], ClassificationRequestOutput)`。

### Test / 测试: test_token_ids_prompts (L49-L55)
```python
@pytest.mark.skip_global_cleanup
def test_token_ids_prompts(llm: LLM):
    outputs = llm.classify([prompt_token_ids], use_tqdm=False)
    assert len(outputs) == 1
    assert isinstance(outputs[0], ClassificationRequestOutput)
    assert outputs[0].prompt_token_ids == prompt_token_ids
    assert len(outputs[0].outputs.probs) == num_labels
```
**EN:** This test validates `test_token_ids_prompts`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. The main assertion is `len(outputs) == 1` and `isinstance(outputs[0], ClassificationRequestOutput)`.
**CN:** 这个测试验证 `test_token_ids_prompts`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 核心断言是 `len(outputs) == 1` and `isinstance(outputs[0], ClassificationRequestOutput)`。

### Test / 测试: test_list_prompts (L58-L65)
```python
@pytest.mark.skip_global_cleanup
def test_list_prompts(llm: LLM):
    outputs = llm.classify([prompt, prompt_token_ids], use_tqdm=False)
    assert len(outputs) == 2
    for i in range(len(outputs)):
        assert isinstance(outputs[i], ClassificationRequestOutput)
        assert outputs[i].prompt_token_ids == prompt_token_ids
        assert len(outputs[i].outputs.probs) == num_labels
```
**EN:** This test validates `test_list_prompts`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. The main assertion is `len(outputs) == 2` and `isinstance(outputs[i], ClassificationRequestOutput)`.
**CN:** 这个测试验证 `test_list_prompts`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 核心断言是 `len(outputs) == 2` and `isinstance(outputs[i], ClassificationRequestOutput)`。

### Test / 测试: test_pooling_params (L68-L90)
```python
@pytest.mark.skip_global_cleanup
def test_pooling_params(llm: LLM):
    def get_outputs(use_activation):
        outputs = llm.classify(
            prompt,
            pooling_params=PoolingParams(use_activation=use_activation),
            use_tqdm=False,
        )
        return torch.tensor([x.outputs.probs for x in outputs])

    default = get_outputs(use_activation=None)
    w_activation = get_outputs(use_activation=True)
    wo_activation = get_outputs(use_activation=False)

    assert torch.allclose(default, w_activation, atol=1e-2), (
        "Default should use activation."
    )
    assert not torch.allclose(w_activation, wo_activation, atol=1e-2), (
        "wo_activation should not use activation."
    )
    assert torch.allclose(softmax(wo_activation), w_activation, atol=1e-2), (
        "w_activation should be close to activation(wo_activation)."
    )
```
**EN:** This test validates `test_pooling_params`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. The main assertion is `torch.allclose(default, w_activation, atol=0.01)` and `not torch.allclose(w_activation, wo_activation, atol=0.01)`.
**CN:** 这个测试验证 `test_pooling_params`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 核心断言是 `torch.allclose(default, w_activation, atol=0.01)` and `not torch.allclose(w_activation, wo_activation, atol=0.01)`。

### Test / 测试: test_score_api (L93-L97)
```python
@pytest.mark.skip_global_cleanup
def test_score_api(llm: LLM):
    err_msg = "Scoring API is only enabled for num_labels == 1."
    with pytest.raises(ValueError, match=err_msg):
        llm.score("ping", "pong", use_tqdm=False)
```
**EN:** This test validates `test_score_api`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `test_score_api`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 它使用 `pytest.raises` 检查预期失败路径。

### Test / 测试: test_unsupported_tasks (L100-L109)
```python
@pytest.mark.parametrize("task", ["embed", "token_embed", "token_classify", "plugin"])
def test_unsupported_tasks(llm: LLM, task: PoolingTask):
    if task == "plugin":
        err_msg = "No IOProcessor plugin installed."
    elif task == "token_classify":
        err_msg = "Try switching the model's pooling_task via.+"
    else:
        err_msg = "Embedding API is not supported by this model.+"
    with pytest.raises(ValueError, match=err_msg):
        llm.encode(prompt, pooling_task=task, use_tqdm=False)
```
**EN:** This test validates `test_unsupported_tasks`. It uses parameterization over `task`. Key inputs are `llm`, `task`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `test_unsupported_tasks`。 它通过参数化组合 `task`。 关键输入包括 `llm`、`task`。 它使用 `pytest.raises` 检查预期失败路径。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `weakref`
- **Third-party / 第三方**: `pytest`, `torch`
- **Project / 项目内**: `tests.models.utils.softmax`, `vllm.ClassificationRequestOutput`, `vllm.LLM`, `vllm.PoolingParams`, `vllm.distributed.cleanup_dist_env_and_memory`, `vllm.tasks.PoolingTask`
