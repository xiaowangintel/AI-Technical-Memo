# test_cross_encoder_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/scoring/test_cross_encoder_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and scoring or reranking behavior. The file defines 6 test(s), 2 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与打分或重排行为。它定义了 6 个测试、2 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L12)
```python
import weakref

import pytest
import torch

from tests.models.utils import softmax
from vllm import LLM, PoolingParams
from vllm.distributed import cleanup_dist_env_and_memory
from vllm.platforms import current_platform
```
**EN:** Imports standard-library modules such as `weakref`, third-party packages like `pytest`, `torch`, project helpers such as `tests.models.utils.softmax`, `vllm.LLM`, `vllm.PoolingParams`.
**CN:** 导入标准库模块（如 `weakref`）、第三方包（如 `pytest`、`torch`）、项目内辅助模块（如 `tests.models.utils.softmax`、`vllm.LLM`、`vllm.PoolingParams`）。

### Module setup / 模块级配置: MODEL_NAME, PROMPT, TEXTS_1 (L14-L24)
```python
MODEL_NAME = "tomaarsen/Qwen3-Reranker-0.6B-seq-cls"
PROMPT = "The chef prepared a delicious meal."
TEXTS_1 = [
    "What is the capital of France?",
    "What is the capital of Germany?",
]

TEXTS_2 = [
    "The capital of France is Paris.",
    "The capital of Germany is Berlin.",
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `PROMPT`, `TEXTS_1`, `TEXTS_2`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`PROMPT`、`TEXTS_1`、`TEXTS_2`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: llm (L27-L51)
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

    yield weakref.proxy(llm)

    del llm

    cleanup_dist_env_and_memory()
```
**EN:** This fixture prepares `llm` for dependent tests. It touches the core vLLM initialization or engine path directly. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `llm`。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: hf_model (L54-L56)
```python
@pytest.fixture(scope="module")
def hf_model(hf_runner):
    return hf_runner(MODEL_NAME, is_cross_encoder=True)
```
**EN:** This fixture prepares `hf_model` for dependent tests. Key inputs are `hf_runner`.
**CN:** 这个 fixture 为依赖它的测试准备 `hf_model`。 关键输入包括 `hf_runner`。

### Test / 测试: test_1_to_1 (L59-L71)
```python
@pytest.mark.skip_global_cleanup
def test_1_to_1(llm, hf_model):
    text_pair = [TEXTS_1[0], TEXTS_2[0]]

    hf_outputs = hf_model.predict([text_pair]).tolist()
    vllm_outputs = [
        output.outputs.score for output in llm.score(text_pair[0], text_pair[1])
    ]

    assert len(vllm_outputs) == 1
    assert len(hf_outputs) == 1

    assert hf_outputs[0] == pytest.approx(vllm_outputs[0], rel=0.01)
```
**EN:** This test validates `test_1_to_1`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`, `hf_model`. The main assertion is `len(vllm_outputs) == 1` and `len(hf_outputs) == 1`.
**CN:** 这个测试验证 `test_1_to_1`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`、`hf_model`。 核心断言是 `len(vllm_outputs) == 1` and `len(hf_outputs) == 1`。

### Test / 测试: test_1_to_n (L74-L88)
```python
@pytest.mark.skip_global_cleanup
def test_1_to_n(llm, hf_model):
    text_pairs = [
        [TEXTS_1[0], TEXTS_2[0]],
        [TEXTS_1[0], TEXTS_2[1]],
    ]

    vllm_outputs = [output.outputs.score for output in llm.score(TEXTS_1[0], TEXTS_2)]
    hf_outputs = hf_model.predict(text_pairs).tolist()

    assert len(vllm_outputs) == 2
    assert len(hf_outputs) == 2

    assert hf_outputs[0] == pytest.approx(vllm_outputs[0], rel=0.01)
    assert hf_outputs[1] == pytest.approx(vllm_outputs[1], rel=0.01)
```
**EN:** This test validates `test_1_to_n`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`, `hf_model`. The main assertion is `len(vllm_outputs) == 2` and `len(hf_outputs) == 2`.
**CN:** 这个测试验证 `test_1_to_n`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`、`hf_model`。 核心断言是 `len(vllm_outputs) == 2` and `len(hf_outputs) == 2`。

### Test / 测试: test_n_to_n (L91-L105)
```python
@pytest.mark.skip_global_cleanup
def test_n_to_n(llm, hf_model):
    text_pairs = [
        [TEXTS_1[0], TEXTS_2[0]],
        [TEXTS_1[1], TEXTS_2[1]],
    ]

    vllm_outputs = [output.outputs.score for output in llm.score(TEXTS_1, TEXTS_2)]
    hf_outputs = hf_model.predict(text_pairs).tolist()

    assert len(vllm_outputs) == 2
    assert len(hf_outputs) == 2

    assert hf_outputs[0] == pytest.approx(vllm_outputs[0], rel=0.01)
    assert hf_outputs[1] == pytest.approx(vllm_outputs[1], rel=0.01)
```
**EN:** This test validates `test_n_to_n`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`, `hf_model`. The main assertion is `len(vllm_outputs) == 2` and `len(hf_outputs) == 2`.
**CN:** 这个测试验证 `test_n_to_n`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`、`hf_model`。 核心断言是 `len(vllm_outputs) == 2` and `len(hf_outputs) == 2`。

### Test / 测试: test_classify (L108-L112)
```python
@pytest.mark.skip_global_cleanup
def test_classify(llm):
    outputs = llm.encode(PROMPT, pooling_task="classify", use_tqdm=False)
    assert len(outputs) == 1
    assert len(outputs[0].outputs.data) == 1
```
**EN:** This test validates `test_classify`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. The main assertion is `len(outputs) == 1` and `len(outputs[0].outputs.data) == 1`.
**CN:** 这个测试验证 `test_classify`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 核心断言是 `len(outputs) == 1` and `len(outputs[0].outputs.data) == 1`。

### Test / 测试: test_max_tokens_per_doc (L115-L141)
```python
@pytest.mark.skip_global_cleanup
def test_max_tokens_per_doc(llm: LLM):
    """Test max_tokens_per_doc via PoolingParams.extra_kwargs (offline)."""
    long_doc = "The capital of France is Paris. " * 20

    # Without truncation
    outputs_no_limit = llm.score(
        TEXTS_1[0],
        long_doc,
        use_tqdm=False,
    )

    # With truncation via extra_kwargs
    outputs_with_limit = llm.score(
        TEXTS_1[0],
        long_doc,
        pooling_params=PoolingParams(extra_kwargs={"max_tokens_per_doc": 10}),
        use_tqdm=False,
    )

    assert len(outputs_no_limit) == 1
    assert len(outputs_with_limit) == 1

    # Truncated version should have fewer prompt tokens
    no_limit_tokens = len(outputs_no_limit[0].prompt_token_ids)
    with_limit_tokens = len(outputs_with_limit[0].prompt_token_ids)
    assert with_limit_tokens < no_limit_tokens
```
**EN:** This test validates `test_max_tokens_per_doc`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. The main assertion is `len(outputs_no_limit) == 1` and `len(outputs_with_limit) == 1`.
**CN:** 这个测试验证 `test_max_tokens_per_doc`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 核心断言是 `len(outputs_no_limit) == 1` and `len(outputs_with_limit) == 1`。

### Test / 测试: test_pooling_params (L144-L166)
```python
def test_pooling_params(llm: LLM):
    def get_outputs(use_activation):
        outputs = llm.score(
            TEXTS_1[0],
            TEXTS_2[0],
            pooling_params=PoolingParams(use_activation=use_activation),
            use_tqdm=False,
        )
        return torch.tensor([x.outputs.score for x in outputs])

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
**EN:** This test validates `test_pooling_params`. Key inputs are `llm`. The main assertion is `torch.allclose(default, w_activation, atol=0.01)` and `not torch.allclose(w_activation, wo_activation, atol=0.01)`.
**CN:** 这个测试验证 `test_pooling_params`。 关键输入包括 `llm`。 核心断言是 `torch.allclose(default, w_activation, atol=0.01)` and `not torch.allclose(w_activation, wo_activation, atol=0.01)`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `weakref`
- **Third-party / 第三方**: `pytest`, `torch`
- **Project / 项目内**: `tests.models.utils.softmax`, `vllm.LLM`, `vllm.PoolingParams`, `vllm.distributed.cleanup_dist_env_and_memory`, `vllm.platforms.current_platform`
