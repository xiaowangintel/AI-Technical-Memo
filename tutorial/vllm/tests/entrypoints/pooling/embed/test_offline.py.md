# test_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/embed/test_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior. The file defines 5 test(s), 1 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为。它定义了 5 个测试、1 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L12)
```python
import weakref

import pytest
import torch
import torch.nn.functional as F

from vllm import LLM, EmbeddingRequestOutput, PoolingParams
from vllm.distributed import cleanup_dist_env_and_memory
from vllm.platforms import current_platform
from vllm.tasks import PoolingTask
```
**EN:** Imports standard-library modules such as `weakref`, third-party packages like `pytest`, `torch`, `torch.nn.functional`, project helpers such as `vllm.EmbeddingRequestOutput`, `vllm.LLM`, `vllm.PoolingParams`.
**CN:** 导入标准库模块（如 `weakref`）、第三方包（如 `pytest`、`torch`、`torch.nn.functional`）、项目内辅助模块（如 `vllm.EmbeddingRequestOutput`、`vllm.LLM`、`vllm.PoolingParams`）。

### Module setup / 模块级配置: MODEL_NAME, prompt, prompt_token_ids (L14-L18)
```python
MODEL_NAME = "intfloat/multilingual-e5-small"

prompt = "The chef prepared a delicious meal."
prompt_token_ids = [0, 581, 21861, 133888, 10, 8, 150, 60744, 109911, 5, 2]
embedding_size = 384
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `prompt`, `prompt_token_ids`, `embedding_size`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`prompt`、`prompt_token_ids`、`embedding_size`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: llm (L21-L45)
```python
@pytest.fixture(scope="module")
def llm():
    # ROCm: Use FLEX_ATTENTION backend as it's the only attention backend
    # that supports encoder-only models on ROCm.
    attention_config = None
    if current_platform.is_rocm():
        attention_config = {"backend": "FLEX_ATTENTION"}

    # pytest caches the fixture so we use weakref.proxy to
    # enable garbage collection
    llm = LLM(
        model=MODEL_NAME,
        max_num_batched_tokens=32768,
        tensor_parallel_size=1,
        gpu_memory_utilization=0.75,
        enforce_eager=True,
        seed=0,
        attention_config=attention_config,
    )
    assert embedding_size == llm.model_config.embedding_size

    yield weakref.proxy(llm)

    del llm
    cleanup_dist_env_and_memory()
```
**EN:** This fixture prepares `llm` for dependent tests. It touches the core vLLM initialization or engine path directly. The fixture yields its resource so teardown can happen after the test completes. The main assertion is `embedding_size == llm.model_config.embedding_size`.
**CN:** 这个 fixture 为依赖它的测试准备 `llm`。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。 核心断言是 `embedding_size == llm.model_config.embedding_size`。

### Test / 测试: test_str_prompts (L48-L54)
```python
@pytest.mark.skip_global_cleanup
def test_str_prompts(llm: LLM):
    outputs = llm.embed(prompt, use_tqdm=False)
    assert len(outputs) == 1
    assert isinstance(outputs[0], EmbeddingRequestOutput)
    assert outputs[0].prompt_token_ids == prompt_token_ids
    assert len(outputs[0].outputs.embedding) == embedding_size
```
**EN:** This test validates `test_str_prompts`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. The main assertion is `len(outputs) == 1` and `isinstance(outputs[0], EmbeddingRequestOutput)`.
**CN:** 这个测试验证 `test_str_prompts`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 核心断言是 `len(outputs) == 1` and `isinstance(outputs[0], EmbeddingRequestOutput)`。

### Test / 测试: test_token_ids_prompts (L57-L63)
```python
@pytest.mark.skip_global_cleanup
def test_token_ids_prompts(llm: LLM):
    outputs = llm.embed([prompt_token_ids], use_tqdm=False)
    assert len(outputs) == 1
    assert isinstance(outputs[0], EmbeddingRequestOutput)
    assert outputs[0].prompt_token_ids == prompt_token_ids
    assert len(outputs[0].outputs.embedding) == embedding_size
```
**EN:** This test validates `test_token_ids_prompts`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. The main assertion is `len(outputs) == 1` and `isinstance(outputs[0], EmbeddingRequestOutput)`.
**CN:** 这个测试验证 `test_token_ids_prompts`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 核心断言是 `len(outputs) == 1` and `isinstance(outputs[0], EmbeddingRequestOutput)`。

### Test / 测试: test_list_prompts (L66-L73)
```python
@pytest.mark.skip_global_cleanup
def test_list_prompts(llm: LLM):
    outputs = llm.embed([prompt, prompt_token_ids], use_tqdm=False)
    assert len(outputs) == 2
    for i in range(len(outputs)):
        assert isinstance(outputs[i], EmbeddingRequestOutput)
        assert outputs[i].prompt_token_ids == prompt_token_ids
        assert len(outputs[i].outputs.embedding) == embedding_size
```
**EN:** This test validates `test_list_prompts`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. The main assertion is `len(outputs) == 2` and `isinstance(outputs[i], EmbeddingRequestOutput)`.
**CN:** 这个测试验证 `test_list_prompts`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 核心断言是 `len(outputs) == 2` and `isinstance(outputs[i], EmbeddingRequestOutput)`。

### Test / 测试: test_pooling_params (L76-L96)
```python
@pytest.mark.skip_global_cleanup
def test_pooling_params(llm: LLM):
    def get_outputs(normalize):
        outputs = llm.embed(
            [prompt],
            pooling_params=PoolingParams(use_activation=normalize),
            use_tqdm=False,
        )
        return torch.tensor([x.outputs.embedding for x in outputs])

    default = get_outputs(normalize=None)
    w_normal = get_outputs(normalize=True)
    wo_normal = get_outputs(normalize=False)

    assert torch.allclose(default, w_normal, atol=1e-2), "Default should use normal."
    assert not torch.allclose(w_normal, wo_normal, atol=1e-2), (
        "wo_normal should not use normal."
    )
    assert torch.allclose(w_normal, F.normalize(wo_normal, p=2, dim=-1), atol=1e-2), (
        "w_normal should be close to normal(wo_normal)."
    )
```
**EN:** This test validates `test_pooling_params`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. The main assertion is `torch.allclose(default, w_normal, atol=0.01)` and `not torch.allclose(w_normal, wo_normal, atol=0.01)`.
**CN:** 这个测试验证 `test_pooling_params`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 核心断言是 `torch.allclose(default, w_normal, atol=0.01)` and `not torch.allclose(w_normal, wo_normal, atol=0.01)`。

### Test / 测试: test_unsupported_tasks (L99-L110)
```python
@pytest.mark.parametrize(
    "task", ["token_classify", "classify", "token_embed", "plugin"]
)
def test_unsupported_tasks(llm: LLM, task: PoolingTask):
    if task == "plugin":
        err_msg = "No IOProcessor plugin installed."
    elif task == "token_embed":
        err_msg = "Try switching the model's pooling_task via.+"
    else:
        err_msg = "Classification API is not supported by this model.+"
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
- **Third-party / 第三方**: `pytest`, `torch`, `torch.nn.functional`
- **Project / 项目内**: `vllm.EmbeddingRequestOutput`, `vllm.LLM`, `vllm.PoolingParams`, `vllm.distributed.cleanup_dist_env_and_memory`, `vllm.platforms.current_platform`, `vllm.tasks.PoolingTask`
