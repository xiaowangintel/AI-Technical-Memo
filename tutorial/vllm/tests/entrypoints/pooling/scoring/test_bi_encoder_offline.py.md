# test_bi_encoder_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/scoring/test_bi_encoder_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and scoring or reranking behavior. The file defines 4 test(s), 2 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与打分或重排行为。它定义了 4 个测试、2 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L11)
```python
import weakref

import pytest

from tests.entrypoints.pooling.scoring.util import EncoderScoringHfRunner
from vllm import LLM
from vllm.distributed import cleanup_dist_env_and_memory
from vllm.platforms import current_platform
```
**EN:** Imports standard-library modules such as `weakref`, third-party packages like `pytest`, project helpers such as `tests.entrypoints.pooling.scoring.util.EncoderScoringHfRunner`, `vllm.LLM`, `vllm.distributed.cleanup_dist_env_and_memory`.
**CN:** 导入标准库模块（如 `weakref`）、第三方包（如 `pytest`）、项目内辅助模块（如 `tests.entrypoints.pooling.scoring.util.EncoderScoringHfRunner`、`vllm.LLM`、`vllm.distributed.cleanup_dist_env_and_memory`）。

### Module setup / 模块级配置: MODEL_NAME, PROMPT, EMBEDDING_SIZE (L13-L27)
```python
MODEL_NAME = "intfloat/multilingual-e5-small"
PROMPT = "The chef prepared a delicious meal."
EMBEDDING_SIZE = 384

TEXTS_1 = [
    "What is the capital of France?",
    "What is the capital of Germany?",
]

TEXTS_2 = [
    "The capital of France is Paris.",
    "The capital of Germany is Berlin.",
]

DTYPE = "half"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `PROMPT`, `EMBEDDING_SIZE`, `TEXTS_1`, `TEXTS_2`, `DTYPE`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`PROMPT`、`EMBEDDING_SIZE`、`TEXTS_1`、`TEXTS_2`、`DTYPE`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: llm (L30-L54)
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

### Fixture / 夹具: hf_model (L57-L59)
```python
@pytest.fixture(scope="module")
def hf_model():
    return EncoderScoringHfRunner(MODEL_NAME)
```
**EN:** This fixture prepares `hf_model` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `hf_model`。

### Test / 测试: test_1_to_1 (L62-L74)
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

### Test / 测试: test_1_to_n (L77-L91)
```python
@pytest.mark.skip_global_cleanup
def test_1_to_n(llm, hf_model):
    text_pairs = [
        [TEXTS_1[0], TEXTS_2[0]],
        [TEXTS_1[0], TEXTS_2[1]],
    ]

    hf_outputs = hf_model.predict(text_pairs).tolist()
    vllm_outputs = [output.outputs.score for output in llm.score(TEXTS_1[0], TEXTS_2)]

    assert len(vllm_outputs) == 2
    assert len(hf_outputs) == 2

    assert hf_outputs[0] == pytest.approx(vllm_outputs[0], rel=0.01)
    assert hf_outputs[1] == pytest.approx(vllm_outputs[1], rel=0.01)
```
**EN:** This test validates `test_1_to_n`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`, `hf_model`. The main assertion is `len(vllm_outputs) == 2` and `len(hf_outputs) == 2`.
**CN:** 这个测试验证 `test_1_to_n`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`、`hf_model`。 核心断言是 `len(vllm_outputs) == 2` and `len(hf_outputs) == 2`。

### Test / 测试: test_n_to_n (L94-L108)
```python
@pytest.mark.skip_global_cleanup
def test_n_to_n(llm, hf_model):
    text_pairs = [
        [TEXTS_1[0], TEXTS_2[0]],
        [TEXTS_1[1], TEXTS_2[1]],
    ]

    hf_outputs = hf_model.predict(text_pairs).tolist()
    vllm_outputs = [output.outputs.score for output in llm.score(TEXTS_1, TEXTS_2)]

    assert len(vllm_outputs) == 2
    assert len(hf_outputs) == 2

    assert hf_outputs[0] == pytest.approx(vllm_outputs[0], rel=0.01)
    assert hf_outputs[1] == pytest.approx(vllm_outputs[1], rel=0.01)
```
**EN:** This test validates `test_n_to_n`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`, `hf_model`. The main assertion is `len(vllm_outputs) == 2` and `len(hf_outputs) == 2`.
**CN:** 这个测试验证 `test_n_to_n`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`、`hf_model`。 核心断言是 `len(vllm_outputs) == 2` and `len(hf_outputs) == 2`。

### Test / 测试: test_embed (L111-L114)
```python
def test_embed(llm):
    outputs = llm.encode(PROMPT, pooling_task="embed", use_tqdm=False)
    assert len(outputs) == 1
    assert len(outputs[0].outputs.data) == EMBEDDING_SIZE
```
**EN:** This test validates `test_embed`. Key inputs are `llm`. The main assertion is `len(outputs) == 1` and `len(outputs[0].outputs.data) == EMBEDDING_SIZE`.
**CN:** 这个测试验证 `test_embed`。 关键输入包括 `llm`。 核心断言是 `len(outputs) == 1` and `len(outputs[0].outputs.data) == EMBEDDING_SIZE`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `weakref`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `tests.entrypoints.pooling.scoring.util.EncoderScoringHfRunner`, `vllm.LLM`, `vllm.distributed.cleanup_dist_env_and_memory`, `vllm.platforms.current_platform`
