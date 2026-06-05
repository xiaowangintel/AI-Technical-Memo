# test_late_interaction_offline_vision.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/scoring/test_late_interaction_offline_vision.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and scoring or reranking behavior. The file defines 3 test(s), 1 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与打分或重排行为。它定义了 3 个测试、1 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L12)
```python
import weakref

import pytest

from vllm import LLM
from vllm.distributed import cleanup_dist_env_and_memory
from vllm.platforms import current_platform

from .util import make_base64_image, make_image_mm_param
```
**EN:** Imports standard-library modules such as `weakref`, third-party packages like `pytest`, project helpers such as `vllm.LLM`, `vllm.distributed.cleanup_dist_env_and_memory`, `vllm.platforms.current_platform`.
**CN:** 导入标准库模块（如 `weakref`）、第三方包（如 `pytest`）、项目内辅助模块（如 `vllm.LLM`、`vllm.distributed.cleanup_dist_env_and_memory`、`vllm.platforms.current_platform`）。

### Module setup / 模块级配置: MODEL_NAME (L14-L14)
```python
MODEL_NAME = "vidore/colpali-v1.3-hf"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: llm (L17-L41)
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

### Test / 测试: test_query_text_vs_docs_image (L44-L59)
```python
@pytest.mark.skip_global_cleanup
def test_query_text_vs_docs_image(llm):
    """Score a text query against image documents via the multimodal path."""
    red_image = make_base64_image(64, 64, color=(255, 0, 0))
    blue_image = make_base64_image(64, 64, color=(0, 0, 255))

    query = "Describe the red object"
    image_docs = [
        make_image_mm_param(red_image),
        make_image_mm_param(blue_image),
    ]

    scores = llm.score(query, image_docs)

    assert len(scores) == 2
    assert scores[0].outputs.score > scores[1].outputs.score
```
**EN:** This test validates `test_query_text_vs_docs_image`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. The main assertion is `len(scores) == 2` and `scores[0].outputs.score > scores[1].outputs.score`.
**CN:** 这个测试验证 `test_query_text_vs_docs_image`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 核心断言是 `len(scores) == 2` and `scores[0].outputs.score > scores[1].outputs.score`。

### Test / 测试: test_query_text_vs_docs_mix (L62-L76)
```python
@pytest.mark.skip_global_cleanup
def test_query_text_vs_docs_mix(llm) -> None:
    """Score a text query against a mix of text and image documents."""
    red_image = make_base64_image(64, 64, color=(255, 0, 0))

    query = "What is the capital of France?"
    documents: list = [
        "The capital of France is Paris.",
        make_image_mm_param(red_image),
    ]

    scores = llm.score(query, documents)

    assert len(scores) == 2
    assert scores[0].outputs.score > scores[1].outputs.score
```
**EN:** This test validates `test_query_text_vs_docs_mix`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. The main assertion is `len(scores) == 2` and `scores[0].outputs.score > scores[1].outputs.score`.
**CN:** 这个测试验证 `test_query_text_vs_docs_mix`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 核心断言是 `len(scores) == 2` and `scores[0].outputs.score > scores[1].outputs.score`。

### Test / 测试: test_query_image_vs_docs_text (L79-L93)
```python
@pytest.mark.skip_global_cleanup
def test_query_image_vs_docs_text(llm) -> None:
    """Score an image query against text documents."""
    red_image = make_base64_image(64, 64, color=(255, 0, 0))
    image_query = make_image_mm_param(red_image, text="red color")

    documents = [
        "Describe the red object.",
        "The capital of France is Paris.",
    ]

    scores = llm.score(image_query, documents)

    assert len(scores) == 2
    assert scores[0].outputs.score > scores[1].outputs.score
```
**EN:** This test validates `test_query_image_vs_docs_text`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. The main assertion is `len(scores) == 2` and `scores[0].outputs.score > scores[1].outputs.score`.
**CN:** 这个测试验证 `test_query_image_vs_docs_text`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 核心断言是 `len(scores) == 2` and `scores[0].outputs.score > scores[1].outputs.score`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `weakref`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `vllm.LLM`, `vllm.distributed.cleanup_dist_env_and_memory`, `vllm.platforms.current_platform`
- **Local relative imports / 本地相对导入**: `.util.make_base64_image`, `.util.make_image_mm_param`
