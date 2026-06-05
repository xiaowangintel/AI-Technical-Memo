# test_cross_encoder_correctness_mteb.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/scoring/test_cross_encoder_correctness_mteb.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and scoring or reranking behavior. The file defines 2 test(s), 1 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与打分或重排行为。它定义了 2 个测试、1 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L16)
```python
import os

import pytest

from tests.models.language.pooling_mteb_test.mteb_score_utils import (
    MTEB_RERANK_LANGS,
    MTEB_RERANK_TASKS,
    MTEB_RERANK_TOL,
    RerankClientMtebEncoder,
    ScoreClientMtebEncoder,
    run_mteb_rerank,
)
from tests.utils import RemoteOpenAIServer
from vllm.platforms import current_platform
```
**EN:** Imports standard-library modules such as `os`, third-party packages like `pytest`, project helpers such as `tests.models.language.pooling_mteb_test.mteb_score_utils.MTEB_RERANK_LANGS`, `tests.models.language.pooling_mteb_test.mteb_score_utils.MTEB_RERANK_TASKS`, `tests.models.language.pooling_mteb_test.mteb_score_utils.MTEB_RERANK_TOL`.
**CN:** 导入标准库模块（如 `os`）、第三方包（如 `pytest`）、项目内辅助模块（如 `tests.models.language.pooling_mteb_test.mteb_score_utils.MTEB_RERANK_LANGS`、`tests.models.language.pooling_mteb_test.mteb_score_utils.MTEB_RERANK_TASKS`、`tests.models.language.pooling_mteb_test.mteb_score_utils.MTEB_RERANK_TOL`）。

### Module setup / 模块级配置: MODEL_NAME, st_main_score (L18-L21)
```python
os.environ["VLLM_LOGGING_LEVEL"] = "WARNING"

MODEL_NAME = "cross-encoder/ms-marco-MiniLM-L-6-v2"
st_main_score = 0.33457
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `st_main_score`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`st_main_score`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L24-L33)
```python
@pytest.fixture(scope="module")
def server():
    args = ["--runner", "pooling", "--enforce-eager", "--disable-uvicorn-access-log"]

    # ROCm: Use Flex Attention to support encoder-only self-attention.
    if current_platform.is_rocm():
        args.extend(["--attention-backend", "FLEX_ATTENTION"])

    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_mteb_score (L36-L47)
```python
def test_mteb_score(server):
    url = server.url_for("score")
    encoder = ScoreClientMtebEncoder(MODEL_NAME, url)
    vllm_main_score = run_mteb_rerank(encoder, MTEB_RERANK_TASKS, MTEB_RERANK_LANGS)

    print("VLLM main score: ", vllm_main_score)
    print("SentenceTransformer main score: ", st_main_score)
    print("Difference: ", st_main_score - vllm_main_score)

    # We are not concerned that the vllm mteb results are better
    # than SentenceTransformers, so we only perform one-sided testing.
    assert st_main_score - vllm_main_score < MTEB_RERANK_TOL
```
**EN:** This test validates `test_mteb_score`. Key inputs are `server`. The main assertion is `st_main_score - vllm_main_score < MTEB_RERANK_TOL`.
**CN:** 这个测试验证 `test_mteb_score`。 关键输入包括 `server`。 核心断言是 `st_main_score - vllm_main_score < MTEB_RERANK_TOL`。

### Test / 测试: test_mteb_rerank (L50-L61)
```python
def test_mteb_rerank(server):
    url = server.url_for("rerank")
    encoder = RerankClientMtebEncoder(MODEL_NAME, url)
    vllm_main_score = run_mteb_rerank(encoder, MTEB_RERANK_TASKS, MTEB_RERANK_LANGS)

    print("VLLM main score: ", vllm_main_score)
    print("SentenceTransformer main score: ", st_main_score)
    print("Difference: ", st_main_score - vllm_main_score)

    # We are not concerned that the vllm mteb results are better
    # than SentenceTransformers, so we only perform one-sided testing.
    assert st_main_score - vllm_main_score < MTEB_RERANK_TOL
```
**EN:** This test validates `test_mteb_rerank`. Key inputs are `server`. The main assertion is `st_main_score - vllm_main_score < MTEB_RERANK_TOL`.
**CN:** 这个测试验证 `test_mteb_rerank`。 关键输入包括 `server`。 核心断言是 `st_main_score - vllm_main_score < MTEB_RERANK_TOL`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `os`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `tests.models.language.pooling_mteb_test.mteb_score_utils.MTEB_RERANK_LANGS`, `tests.models.language.pooling_mteb_test.mteb_score_utils.MTEB_RERANK_TASKS`, `tests.models.language.pooling_mteb_test.mteb_score_utils.MTEB_RERANK_TOL`, `tests.models.language.pooling_mteb_test.mteb_score_utils.RerankClientMtebEncoder`, `tests.models.language.pooling_mteb_test.mteb_score_utils.ScoreClientMtebEncoder`, `tests.models.language.pooling_mteb_test.mteb_score_utils.run_mteb_rerank`, `tests.utils.RemoteOpenAIServer`, `vllm.platforms.current_platform`
