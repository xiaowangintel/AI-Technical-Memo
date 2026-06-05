# test_openai_schema.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/test_openai_schema.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers OpenAI-compatible serving. The file defines 1 test(s), 2 fixture(s), and 2 helper/class block(s) to validate this area. / [CN] 该文件覆盖OpenAI 兼容服务。它定义了 1 个测试、2 个 fixture，以及 2 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L14)
```python
import json
from typing import Final

import pytest
import schemathesis
from hypothesis import HealthCheck, settings
from schemathesis import GenerationConfig
from schemathesis.models import Case

from vllm.platforms import current_platform

from ...utils import RemoteOpenAIServer
```
**EN:** Imports standard-library modules such as `json`, `typing.Final`, third-party packages like `hypothesis.HealthCheck`, `hypothesis.settings`, `pytest`, project helpers such as `vllm.platforms.current_platform`, `...utils.RemoteOpenAIServer`.
**CN:** 导入标准库模块（如 `json`、`typing.Final`）、第三方包（如 `hypothesis.HealthCheck`、`hypothesis.settings`、`pytest`）、项目内辅助模块（如 `vllm.platforms.current_platform`、`...utils.RemoteOpenAIServer`）。

### Module setup / 模块级配置: MODEL_NAME, MAXIMUM_IMAGES, _ROCM_TIMEOUT_MULTIPLIER (L16-L22)
```python
schemathesis.experimental.OPEN_API_3_1.enable()

MODEL_NAME = "HuggingFaceTB/SmolVLM-256M-Instruct"
MAXIMUM_IMAGES = 2
_ROCM_TIMEOUT_MULTIPLIER = 3 if current_platform.is_rocm() else 1
DEFAULT_TIMEOUT_SECONDS: Final[int] = 10 * _ROCM_TIMEOUT_MULTIPLIER
LONG_TIMEOUT_SECONDS: Final[int] = 60 * _ROCM_TIMEOUT_MULTIPLIER
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `MAXIMUM_IMAGES`, `_ROCM_TIMEOUT_MULTIPLIER`, `DEFAULT_TIMEOUT_SECONDS`, `LONG_TIMEOUT_SECONDS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`MAXIMUM_IMAGES`、`_ROCM_TIMEOUT_MULTIPLIER`、`DEFAULT_TIMEOUT_SECONDS`、`LONG_TIMEOUT_SECONDS`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L25-L41)
```python
@pytest.fixture(scope="module")
def server():
    args = [
        "--runner",
        "generate",
        "--max-model-len",
        "2048",
        "--max-num-seqs",
        "5",
        "--enforce-eager",
        "--trust-remote-code",
        "--limit-mm-per-prompt",
        json.dumps({"image": MAXIMUM_IMAGES}),
    ]

    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: get_schema (L44-L50)
```python
@pytest.fixture(scope="module")
def get_schema(server):
    # avoid generating null (\x00) bytes in strings during test case generation
    return schemathesis.openapi.from_uri(
        f"{server.url_root}/openapi.json",
        generation_config=GenerationConfig(allow_x00=False),
    )
```
**EN:** This fixture prepares `get_schema` for dependent tests. Key inputs are `server`.
**CN:** 这个 fixture 为依赖它的测试准备 `get_schema`。 关键输入包括 `server`。

### Module setup / 模块级配置: schema (L53-L53)
```python
schema = schemathesis.from_pytest_fixture("get_schema")
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `schema`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `schema`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: before_generate_case (L56-L107)
```python
@schemathesis.hook
def before_generate_case(context: schemathesis.hooks.HookContext, strategy):
    op = context.operation
    assert op is not None

    def no_invalid_types(case: schemathesis.models.Case):
        """
        Skips tool_calls with `"type": "custom"` which schemathesis incorrectly
        generates instead of the valid `"type": "function"`.

        Example test case that is skipped:
        curl -X POST -H 'Content-Type: application/json' \
            -d '{"messages": [{"role": "assistant", "tool_calls": [{"custom": {"input": "", "name": ""}, "id": "", "type": "custom"}]}]}' \
            http://localhost:8000/v1/chat/completions
        """  # noqa: E501
        if hasattr(case, "body") and isinstance(case.body, dict):
            if (
                "messages" in case.body
# ... 26 lines omitted for brevity ...
            if grammar == "":
                # Allow None (will be handled as no grammar)
                # But skip empty strings
                return False

        return True

    return strategy.filter(no_invalid_types)
```
**EN:** This helper encapsulates reusable logic in `before_generate_case`. Key inputs are `context`, `strategy`. It returns computed state or helper objects back to the caller. The main assertion is `op is not None`.
**CN:** 这个辅助函数将可复用逻辑封装在 `before_generate_case` 中。 关键输入包括 `context`、`strategy`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `op is not None`。

### Test / 测试: test_openapi_stateless (L110-L152)
```python
@schema.parametrize()
@schema.override(headers={"Content-Type": "application/json"})
@settings(
    deadline=LONG_TIMEOUT_SECONDS * 1000,
    max_examples=50,
    # Under CI's derandomized hypothesis seed, the schemathesis strategy
    # for /v1/chat/completions/batch's nested-message body, combined with
    # the no_invalid_types filter (notably the grammar=="" rule), exceeds
    # the default filtered-vs-good ratio. The filter is intentional, so
    # suppress the health check rather than drop the filter — dropping it
    # exposes pre-existing server bugs out of scope here.
    suppress_health_check=[HealthCheck.filter_too_much],
)
def test_openapi_stateless(case: Case):
    key = (
        case.operation.method.upper(),
        case.operation.path,
    )
# ... 17 lines omitted for brevity ...
        ("POST", "/v1/chat/completions/batch"): LONG_TIMEOUT_SECONDS,
        ("POST", "/v1/completions"): LONG_TIMEOUT_SECONDS,
        ("POST", "/v1/messages"): LONG_TIMEOUT_SECONDS,
        ("POST", "/inference/v1/generate"): LONG_TIMEOUT_SECONDS,
    }.get(key, DEFAULT_TIMEOUT_SECONDS)

    # No need to verify SSL certificate for localhost
    case.call_and_validate(verify=False, timeout=timeout)
```
**EN:** This test validates `test_openapi_stateless`. Key inputs are `case`.
**CN:** 这个测试验证 `test_openapi_stateless`。 关键输入包括 `case`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `typing.Final`
- **Third-party / 第三方**: `hypothesis.HealthCheck`, `hypothesis.settings`, `pytest`, `schemathesis`, `schemathesis.GenerationConfig`, `schemathesis.models.Case`
- **Project / 项目内**: `vllm.platforms.current_platform`
- **Local relative imports / 本地相对导入**: `...utils.RemoteOpenAIServer`
