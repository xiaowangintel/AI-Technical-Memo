# test_online_vision.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/embed/test_online_vision.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and vision or image inputs. The file defines 5 test(s), 1 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与视觉或图像输入。它定义了 5 个测试、1 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L13)
```python
import json

import pytest
import requests
from transformers import AutoProcessor

from tests.utils import VLLM_PATH, RemoteOpenAIServer
from vllm.entrypoints.pooling.embed.protocol import EmbeddingResponse
from vllm.multimodal.media import MediaWithBytes
from vllm.multimodal.utils import encode_image_url, fetch_image
```
**EN:** Imports standard-library modules such as `json`, third-party packages like `pytest`, `requests`, `transformers.AutoProcessor`, project helpers such as `tests.utils.RemoteOpenAIServer`, `tests.utils.VLLM_PATH`, `vllm.entrypoints.pooling.embed.protocol.EmbeddingResponse`.
**CN:** 导入标准库模块（如 `json`）、第三方包（如 `pytest`、`requests`、`transformers.AutoProcessor`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`、`tests.utils.VLLM_PATH`、`vllm.entrypoints.pooling.embed.protocol.EmbeddingResponse`）。

### Module setup / 模块级配置: MODEL_NAME, MAXIMUM_IMAGES, vlm2vec_jinja_path (L15-L31)
```python
MODEL_NAME = "TIGER-Lab/VLM2Vec-Full"
MAXIMUM_IMAGES = 2

vlm2vec_jinja_path = VLLM_PATH / "examples/pooling/embed/template/vlm2vec_phi3v.jinja"
assert vlm2vec_jinja_path.exists()

# Test different image extensions (JPG/PNG) and formats (gray/RGB/RGBA)
TEST_IMAGE_ASSETS = [
    "2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg",  # "https://vllm-public-assets.s3.us-west-2.amazonaws.com/vision_model_images/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg"
    "Grayscale_8bits_palette_sample_image.png",  # "https://vllm-public-assets.s3.us-west-2.amazonaws.com/vision_model_images/Grayscale_8bits_palette_sample_image.png",
    "1280px-Venn_diagram_rgb.svg.png",  # "https://vllm-public-assets.s3.us-west-2.amazonaws.com/vision_model_images/1280px-Venn_diagram_rgb.svg.png",
    "RGBA_comp.png",  # "https://vllm-public-assets.s3.us-west-2.amazonaws.com/vision_model_images/RGBA_comp.png",
]

input_text = "The best thing about vLLM is that it supports many different models"
image_url = "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/cat_snow.jpg"
image_base64 = {"url": encode_image_url(fetch_image(image_url))}
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `MAXIMUM_IMAGES`, `vlm2vec_jinja_path`, `TEST_IMAGE_ASSETS`, `input_text`, `image_url`, `image_base64`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`MAXIMUM_IMAGES`、`vlm2vec_jinja_path`、`TEST_IMAGE_ASSETS`、`input_text`、`image_url`、`image_base64`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: server (L34-L52)
```python
@pytest.fixture(scope="module")
def server():
    args = [
        "--runner",
        "pooling",
        "--max-model-len",
        "2048",
        "--max-num-seqs",
        "5",
        "--enforce-eager",
        "--trust-remote-code",
        "--limit-mm-per-prompt",
        json.dumps({"image": MAXIMUM_IMAGES}),
        "--chat-template",
        str(vlm2vec_jinja_path),
    ]

    with RemoteOpenAIServer(MODEL_NAME, args) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_chat_text_request (L55-L77)
```python
@pytest.mark.parametrize("model_name", [MODEL_NAME])
def test_chat_text_request(server: RemoteOpenAIServer, model_name: str):
    messages = [
        {
            "role": "user",
            "content": input_text,
        },
    ]

    # note: vlm2vec_phi3v.jinja
    # Embedding models should only embed one message at a time.

    response = requests.post(
        server.url_for("v1/embeddings"),
        json={"model": model_name, "messages": messages},
    )
    response.raise_for_status()

    output = EmbeddingResponse.model_validate(response.json())
    assert len(output.data) == 1
    assert output.model == MODEL_NAME
    assert len(output.data[0].embedding) == 3072
    assert output.usage.prompt_tokens == 14
```
**EN:** This test validates `test_chat_text_request`. It uses parameterization over `model_name`. Key inputs are `server`, `model_name`. The main assertion is `len(output.data) == 1` and `output.model == MODEL_NAME`.
**CN:** 这个测试验证 `test_chat_text_request`。 它通过参数化组合 `model_name`。 关键输入包括 `server`、`model_name`。 核心断言是 `len(output.data) == 1` and `output.model == MODEL_NAME`。

### Test / 测试: test_chat_image_url_request (L80-L102)
```python
@pytest.mark.parametrize("model_name", [MODEL_NAME])
def test_chat_image_url_request(server: RemoteOpenAIServer, model_name: str):
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Represent the user's input."},
                {"type": "image_url", "image_url": {"url": image_url}},
            ],
        }
    ]

    response = requests.post(
        server.url_for("v1/embeddings"),
        json={"model": model_name, "messages": messages},
    )
    response.raise_for_status()

    output = EmbeddingResponse.model_validate(response.json())
    assert len(output.data) == 1
    assert output.model == MODEL_NAME
    assert len(output.data[0].embedding) == 3072
    assert output.usage.prompt_tokens == 767
```
**EN:** This test validates `test_chat_image_url_request`. It uses parameterization over `model_name`. Key inputs are `server`, `model_name`. The main assertion is `len(output.data) == 1` and `output.model == MODEL_NAME`.
**CN:** 这个测试验证 `test_chat_image_url_request`。 它通过参数化组合 `model_name`。 关键输入包括 `server`、`model_name`。 核心断言是 `len(output.data) == 1` and `output.model == MODEL_NAME`。

### Test / 测试: test_chat_image_base64_request (L105-L127)
```python
@pytest.mark.parametrize("model_name", [MODEL_NAME])
def test_chat_image_base64_request(server: RemoteOpenAIServer, model_name: str):
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Represent the user's input."},
                {"type": "image_url", "image_url": image_base64},
            ],
        }
    ]

    response = requests.post(
        server.url_for("v1/embeddings"),
        json={"model": model_name, "messages": messages},
    )
    response.raise_for_status()

    output = EmbeddingResponse.model_validate(response.json())
    assert len(output.data) == 1
    assert output.model == MODEL_NAME
    assert len(output.data[0].embedding) == 3072
    assert output.usage.prompt_tokens == 767
```
**EN:** This test validates `test_chat_image_base64_request`. It uses parameterization over `model_name`. Key inputs are `server`, `model_name`. The main assertion is `len(output.data) == 1` and `output.model == MODEL_NAME`.
**CN:** 这个测试验证 `test_chat_image_base64_request`。 它通过参数化组合 `model_name`。 关键输入包括 `server`、`model_name`。 核心断言是 `len(output.data) == 1` and `output.model == MODEL_NAME`。

### Test / 测试: test_chat_image_with_media_io_kwargs (L130-L160)
```python
@pytest.mark.parametrize("model_name", [MODEL_NAME])
def test_chat_image_with_media_io_kwargs(server: RemoteOpenAIServer, model_name: str):
    rgba_image_url = (
        "https://vllm-public-assets.s3.us-west-2.amazonaws.com"
        "/vision_model_images/RGBA_comp.png"
    )
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Represent the user's input."},
                {"type": "image_url", "image_url": {"url": rgba_image_url}},
            ],
        }
    ]

    response = requests.post(
        server.url_for("v1/embeddings"),
# ... 5 lines omitted for brevity ...
            },
        },
    )
    response.raise_for_status()

    output = EmbeddingResponse.model_validate(response.json())
    assert len(output.data) == 1
    assert len(output.data[0].embedding) == 3072
```
**EN:** This test validates `test_chat_image_with_media_io_kwargs`. It uses parameterization over `model_name`. Key inputs are `server`, `model_name`. The main assertion is `len(output.data) == 1` and `len(output.data[0].embedding) == 3072`.
**CN:** 这个测试验证 `test_chat_image_with_media_io_kwargs`。 它通过参数化组合 `model_name`。 关键输入包括 `server`、`model_name`。 核心断言是 `len(output.data) == 1` and `len(output.data[0].embedding) == 3072`。

### Helper / 辅助函数: get_hf_prompt_tokens (L163-L176)
```python
def get_hf_prompt_tokens(model_name, content, image_url):
    processor = AutoProcessor.from_pretrained(
        model_name, trust_remote_code=True, num_crops=4
    )

    placeholder = "<|image_1|> "
    prompt = f"{placeholder}{content}"
    image = fetch_image(image_url)
    # Unwrap MediaWithBytes if present
    if isinstance(image, MediaWithBytes):
        image = image.media
    images = [image]
    inputs = processor(prompt, images, return_tensors="pt")
    return inputs.input_ids.shape[1]
```
**EN:** This helper encapsulates reusable logic in `get_hf_prompt_tokens`. Key inputs are `model_name`, `content`, `image_url`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_hf_prompt_tokens` 中。 关键输入包括 `model_name`、`content`、`image_url`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_image_embedding (L179-L210)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("image_url", TEST_IMAGE_ASSETS, indirect=True)
async def test_image_embedding(
    server: RemoteOpenAIServer, model_name: str, image_url: str
):
    content_text = "Represent the given image."
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "image_url", "image_url": {"url": image_url}},
                {"type": "text", "text": content_text},
            ],
        }
    ]

    response = requests.post(
# ... 6 lines omitted for brevity ...
    hf_prompt_tokens = get_hf_prompt_tokens(model_name, content_text, image_url)

    assert embeddings.id is not None
    assert len(embeddings.data) == 1
    assert len(embeddings.data[0].embedding) == 3072
    assert embeddings.usage.completion_tokens == 0
    assert embeddings.usage.prompt_tokens == hf_prompt_tokens
    assert embeddings.usage.total_tokens == hf_prompt_tokens
```
**EN:** This async test validates `test_image_embedding`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `server`, `model_name`, `image_url`. The main assertion is `embeddings.id is not None` and `len(embeddings.data) == 1`.
**CN:** 这个异步测试验证 `test_image_embedding`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `server`、`model_name`、`image_url`。 核心断言是 `embeddings.id is not None` and `len(embeddings.data) == 1`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`
- **Third-party / 第三方**: `pytest`, `requests`, `transformers.AutoProcessor`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`, `tests.utils.VLLM_PATH`, `vllm.entrypoints.pooling.embed.protocol.EmbeddingResponse`, `vllm.multimodal.media.MediaWithBytes`, `vllm.multimodal.utils.encode_image_url`, `vllm.multimodal.utils.fetch_image`
