# test_image.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/responses/test_image.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers Responses API behavior and OpenAI-compatible serving. The file defines 3 test(s), 4 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖Responses API 行为与OpenAI 兼容服务。它定义了 3 个测试、4 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L11)
```python
import json

import openai
import pytest
import pytest_asyncio

from tests.utils import RemoteOpenAIServer
from vllm.multimodal.utils import encode_image_url
```
**EN:** Imports standard-library modules such as `json`, third-party packages like `openai`, `pytest`, `pytest_asyncio`, project helpers such as `tests.utils.RemoteOpenAIServer`, `vllm.multimodal.utils.encode_image_url`.
**CN:** 导入标准库模块（如 `json`）、第三方包（如 `openai`、`pytest`、`pytest_asyncio`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`vllm.multimodal.utils.encode_image_url`）。

### Module setup / 模块级配置: MODEL_NAME, MAXIMUM_IMAGES, TEST_IMAGE_ASSETS (L14-L22)
```python
MODEL_NAME = "Qwen/Qwen2.5-VL-3B-Instruct"
MAXIMUM_IMAGES = 2
# Test different image extensions (JPG/PNG) and formats (gray/RGB/RGBA)
TEST_IMAGE_ASSETS = [
    "2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg",  # "https://vllm-public-assets.s3.us-west-2.amazonaws.com/vision_model_images/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg"
    "Grayscale_8bits_palette_sample_image.png",  # "https://vllm-public-assets.s3.us-west-2.amazonaws.com/vision_model_images/Grayscale_8bits_palette_sample_image.png",
    "1280px-Venn_diagram_rgb.svg.png",  # "https://vllm-public-assets.s3.us-west-2.amazonaws.com/vision_model_images/1280px-Venn_diagram_rgb.svg.png",
    "RGBA_comp.png",  # "https://vllm-public-assets.s3.us-west-2.amazonaws.com/vision_model_images/RGBA_comp.png",
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `MAXIMUM_IMAGES`, `TEST_IMAGE_ASSETS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`MAXIMUM_IMAGES`、`TEST_IMAGE_ASSETS`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: default_image_server_args (L25-L35)
```python
@pytest.fixture(scope="module")
def default_image_server_args():
    return [
        "--enforce-eager",
        "--max-model-len",
        "6000",
        "--max-num-seqs",
        "128",
        "--limit-mm-per-prompt",
        json.dumps({"image": MAXIMUM_IMAGES}),
    ]
```
**EN:** This fixture prepares `default_image_server_args` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `default_image_server_args`。

### Fixture / 夹具: image_server (L38-L45)
```python
@pytest.fixture(scope="module")
def image_server(default_image_server_args):
    with RemoteOpenAIServer(
        MODEL_NAME,
        default_image_server_args,
        env_dict={"VLLM_ENABLE_RESPONSES_API_STORE": "1"},
    ) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `image_server` for dependent tests. Key inputs are `default_image_server_args`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `image_server`。 关键输入包括 `default_image_server_args`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L48-L51)
```python
@pytest_asyncio.fixture
async def client(image_server):
    async with image_server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `image_server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `image_server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: url_encoded_image (L54-L59)
```python
@pytest.fixture(scope="session")
def url_encoded_image(local_asset_server) -> dict[str, str]:
    return {
        image_url: encode_image_url(local_asset_server.get_image_asset(image_url))
        for image_url in TEST_IMAGE_ASSETS
    }
```
**EN:** This fixture prepares `url_encoded_image` for dependent tests. Key inputs are `local_asset_server`.
**CN:** 这个 fixture 为依赖它的测试准备 `url_encoded_image`。 关键输入包括 `local_asset_server`。

### Test / 测试: test_single_chat_session_image (L62-L88)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("image_url", TEST_IMAGE_ASSETS, indirect=True)
async def test_single_chat_session_image(
    client: openai.AsyncOpenAI, model_name: str, image_url: str
):
    content_text = "What's in this image?"
    messages = [
        {
            "role": "user",
            "content": [
                {
                    "type": "input_image",
                    "image_url": image_url,
                    "detail": "auto",
                },
                {"type": "input_text", "text": content_text},
            ],
        }
    ]

    # test image url
    response = await client.responses.create(
        model=model_name,
        input=messages,
    )
    assert len(response.output_text) > 0
```
**EN:** This async test validates `test_single_chat_session_image`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `image_url`. It drives client-facing request creation through the API surface under test. The main assertion is `len(response.output_text) > 0`.
**CN:** 这个异步测试验证 `test_single_chat_session_image`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`image_url`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(response.output_text) > 0`。

### Test / 测试: test_single_chat_session_image_base64encoded (L91-L119)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("raw_image_url", TEST_IMAGE_ASSETS)
async def test_single_chat_session_image_base64encoded(
    client: openai.AsyncOpenAI,
    model_name: str,
    raw_image_url: str,
    url_encoded_image: dict[str, str],
):
    content_text = "What's in this image?"
    messages = [
        {
            "role": "user",
            "content": [
                {
                    "type": "input_image",
                    "image_url": url_encoded_image[raw_image_url],
                    "detail": "auto",
                },
                {"type": "input_text", "text": content_text},
            ],
        }
    ]
    # test image base64
    response = await client.responses.create(
        model=model_name,
        input=messages,
    )
    assert len(response.output_text) > 0
```
**EN:** This async test validates `test_single_chat_session_image_base64encoded`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `raw_image_url`, `url_encoded_image`. It drives client-facing request creation through the API surface under test. The main assertion is `len(response.output_text) > 0`.
**CN:** 这个异步测试验证 `test_single_chat_session_image_base64encoded`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`raw_image_url`、`url_encoded_image`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(response.output_text) > 0`。

### Test / 测试: test_multi_image_input (L122-L171)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize(
    "image_urls",
    [TEST_IMAGE_ASSETS[:i] for i in range(2, len(TEST_IMAGE_ASSETS))],
    indirect=True,
)
async def test_multi_image_input(
    client: openai.AsyncOpenAI, model_name: str, image_urls: list[str]
):
    messages = [
        {
            "role": "user",
            "content": [
                *(
                    {
                        "type": "input_image",
                        "image_url": image_url,
# ... 24 lines omitted for brevity ...
        )
        assert len(response.output_text) > 0
    else:
        response = await client.responses.create(
            model=model_name,
            input=messages,
        )
        assert len(response.output_text) > 0
```
**EN:** This async test validates `test_multi_image_input`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `image_urls`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `len(response.output_text) > 0` and `len(response.output_text) > 0`.
**CN:** 这个异步测试验证 `test_multi_image_input`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`image_urls`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(response.output_text) > 0` and `len(response.output_text) > 0`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`
- **Third-party / 第三方**: `openai`, `pytest`, `pytest_asyncio`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `vllm.multimodal.utils.encode_image_url`
