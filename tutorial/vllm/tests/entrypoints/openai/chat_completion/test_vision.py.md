# test_vision.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/chat_completion/test_vision.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers chat completion behavior and text completion behavior. The file defines 11 test(s), 3 fixture(s), and 6 helper/class block(s) to validate this area. / [CN] 该文件覆盖聊天补全行为与文本补全行为。它定义了 11 个测试、3 个 fixture，以及 6 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L14)
```python
import json

import openai
import pytest
import pytest_asyncio
from transformers import AutoProcessor

from tests.utils import ROCM_ENV_OVERRIDES, ROCM_EXTRA_ARGS, RemoteOpenAIServer
from vllm.multimodal.media import MediaWithBytes
from vllm.multimodal.utils import encode_image_url, fetch_image
from vllm.platforms import current_platform
```
**EN:** Imports standard-library modules such as `json`, third-party packages like `openai`, `pytest`, `pytest_asyncio`, project helpers such as `tests.utils.ROCM_ENV_OVERRIDES`, `tests.utils.ROCM_EXTRA_ARGS`, `tests.utils.RemoteOpenAIServer`.
**CN:** 导入标准库模块（如 `json`）、第三方包（如 `openai`、`pytest`、`pytest_asyncio`）、项目内辅助模块（如 `tests.utils.ROCM_ENV_OVERRIDES`、`tests.utils.ROCM_EXTRA_ARGS`、`tests.utils.RemoteOpenAIServer`）。

### Module setup / 模块级配置: MODEL_NAME, MAXIMUM_IMAGES, TEST_IMAGE_ASSETS (L16-L40)
```python
MODEL_NAME = "microsoft/Phi-3.5-vision-instruct"
MAXIMUM_IMAGES = 2

# Test different image extensions (JPG/PNG) and formats (gray/RGB/RGBA)
TEST_IMAGE_ASSETS = [
    "2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg",  # "https://vllm-public-assets.s3.us-west-2.amazonaws.com/vision_model_images/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg"
    "Grayscale_8bits_palette_sample_image.png",  # "https://vllm-public-assets.s3.us-west-2.amazonaws.com/vision_model_images/Grayscale_8bits_palette_sample_image.png",
    "1280px-Venn_diagram_rgb.svg.png",  # "https://vllm-public-assets.s3.us-west-2.amazonaws.com/vision_model_images/1280px-Venn_diagram_rgb.svg.png",
    "RGBA_comp.png",  # "https://vllm-public-assets.s3.us-west-2.amazonaws.com/vision_model_images/RGBA_comp.png",
]

# Required terms for beam search validation
# Each entry is a list of term groups - ALL groups must match
# Each group is a list of alternatives - at least ONE term in the group must appear
# This provides semantic validation while allowing wording variation
REQUIRED_BEAM_SEARCH_TERMS = [
    # Boardwalk image: must have "boardwalk" AND ("wooden" or "wood")
    [["boardwalk"], ["wooden", "wood"]],
    # Parrots image: must have ("parrot" or "bird") AND "two"
    [["parrot", "bird"], ["two"]],
    # Venn diagram: must have "venn" AND "diagram"
    [["venn"], ["diagram"]],
    # Gradient image: must have "gradient" AND ("color" or "spectrum")
    [["gradient"], ["color", "spectrum"]],
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `MAXIMUM_IMAGES`, `TEST_IMAGE_ASSETS`, `REQUIRED_BEAM_SEARCH_TERMS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`MAXIMUM_IMAGES`、`TEST_IMAGE_ASSETS`、`REQUIRED_BEAM_SEARCH_TERMS`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: check_output_matches_terms (L43-L52)
```python
def check_output_matches_terms(content: str, term_groups: list[list[str]]) -> bool:
    """
    Check if content matches all required term groups.
    Each term group requires at least one of its terms to be present.
    All term groups must be satisfied.
    """
    content_lower = content.lower()
    return all(
        any(term.lower() in content_lower for term in group) for group in term_groups
    )
```
**EN:** This helper encapsulates reusable logic in `check_output_matches_terms`. Key inputs are `content`, `term_groups`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `check_output_matches_terms` 中。 关键输入包括 `content`、`term_groups`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: assert_non_empty_content (L55-L80)
```python
def assert_non_empty_content(chat_completion, *, context: str = "") -> str:
    """Assert the first choice has non-empty string content; return it.

    Provides a detailed failure message including the full ChatCompletion
    response so flaky / model-quality issues are easy to diagnose.
    """
    prefix = f"[{context}] " if context else ""
    choice = chat_completion.choices[0]
    content = choice.message.content

    assert content is not None, (
        f"{prefix}Expected non-None content but got None. "
        f"finish_reason={choice.finish_reason!r}, "
        f"full message={choice.message!r}, "
        f"usage={chat_completion.usage!r}"
    )
    assert isinstance(content, str), (
        f"{prefix}Expected str content, got {type(content).__name__}: {content!r}"
    )
    assert len(content) > 0, (
        f"{prefix}Expected non-empty content but got empty string. "
        f"finish_reason={choice.finish_reason!r}, "
        f"full message={choice.message!r}, "
        f"usage={chat_completion.usage!r}"
    )
    return content
```
**EN:** This helper encapsulates reusable logic in `assert_non_empty_content`. Key inputs are `chat_completion`. It returns computed state or helper objects back to the caller. The main assertion is `content is not None` and `isinstance(content, str)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `assert_non_empty_content` 中。 关键输入包括 `chat_completion`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `content is not None` and `isinstance(content, str)`。

### Fixture / 夹具: server (L83-L114)
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
        *ROCM_EXTRA_ARGS,
    ]

    # ROCm: Increase timeouts to handle potential network delays and slower
    # video processing when downloading multiple videos from external sources
# ... 6 lines omitted for brevity ...
            }
            if current_platform.is_rocm()
            else {}
        ),
    }

    with RemoteOpenAIServer(MODEL_NAME, args, env_dict=env_overrides) as remote_server:
        yield remote_server
```
**EN:** This fixture prepares `server` for dependent tests. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `server`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: client (L117-L120)
```python
@pytest_asyncio.fixture
async def client(server):
    async with server.get_async_client() as async_client:
        yield async_client
```
**EN:** This async fixture prepares `client` for dependent tests. Key inputs are `server`. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个异步 fixture 为依赖它的测试准备 `client`。 关键输入包括 `server`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: url_encoded_image (L123-L128)
```python
@pytest.fixture(scope="session")
def url_encoded_image(local_asset_server) -> dict[str, str]:
    return {
        image_asset: encode_image_url(local_asset_server.get_image_asset(image_asset))
        for image_asset in TEST_IMAGE_ASSETS
    }
```
**EN:** This fixture prepares `url_encoded_image` for dependent tests. Key inputs are `local_asset_server`.
**CN:** 这个 fixture 为依赖它的测试准备 `url_encoded_image`。 关键输入包括 `local_asset_server`。

### Helper / 辅助函数: dummy_messages_from_image_url (L131-L149)
```python
def dummy_messages_from_image_url(
    image_urls: str | list[str],
    content_text: str = "What's in this image?",
):
    if isinstance(image_urls, str):
        image_urls = [image_urls]

    return [
        {
            "role": "user",
            "content": [
                *(
                    {"type": "image_url", "image_url": {"url": image_url}}
                    for image_url in image_urls
                ),
                {"type": "text", "text": content_text},
            ],
        }
    ]
```
**EN:** This helper encapsulates reusable logic in `dummy_messages_from_image_url`. Key inputs are `image_urls`, `content_text`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `dummy_messages_from_image_url` 中。 关键输入包括 `image_urls`、`content_text`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: describe_image_messages (L152-L174)
```python
def describe_image_messages(
    image_url: str, *, extra_image_fields: dict | None = None
) -> list[dict]:
    """Build the system + user messages used by the completions-with-image
    family of tests. *extra_image_fields* is merged into the top-level
    image content block (for uuid / bad-key tests)."""
    image_block: dict = {
        "type": "image_url",
        "image_url": {"url": image_url},
    }
    if extra_image_fields:
        image_block.update(extra_image_fields)

    return [
        {"role": "system", "content": "You are a helpful assistant."},
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Describe this image."},
                image_block,
            ],
        },
    ]
```
**EN:** This helper encapsulates reusable logic in `describe_image_messages`. Key inputs are `image_url`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `describe_image_messages` 中。 关键输入包括 `image_url`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: complete_and_check (L177-L194)
```python
async def complete_and_check(
    client: openai.AsyncOpenAI,
    model_name: str,
    messages: list[dict],
    *,
    context: str,
    max_completion_tokens: int = 50,
    temperature: float = 0.0,
) -> str:
    """Run a chat completion and assert the output is non-empty.
    Returns the content string."""
    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=max_completion_tokens,
        temperature=temperature,
    )
    return assert_non_empty_content(chat_completion, context=context)
```
**EN:** This async helper encapsulates reusable logic in `complete_and_check`. Key inputs are `client`, `model_name`, `messages`. It drives client-facing request creation through the API surface under test. It returns computed state or helper objects back to the caller.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `complete_and_check` 中。 关键输入包括 `client`、`model_name`、`messages`。 它通过被测 API 表面触发面向客户端的请求创建流程。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: get_hf_prompt_tokens (L197-L220)
```python
def get_hf_prompt_tokens(model_name, content, image_url):
    processor = AutoProcessor.from_pretrained(
        model_name, trust_remote_code=True, num_crops=4
    )

    placeholder = "<|image_1|>\n"
    messages = [
        {
            "role": "user",
            "content": f"{placeholder}{content}",
        }
    ]
    image = fetch_image(image_url)
    # Unwrap MediaWithBytes if present
    if isinstance(image, MediaWithBytes):
        image = image.media
    images = [image]

    prompt = processor.tokenizer.apply_chat_template(
        messages, tokenize=False, add_generation_prompt=True
    )
    inputs = processor(prompt, images, return_tensors="pt")

    return inputs.input_ids.shape[1]
```
**EN:** This helper encapsulates reusable logic in `get_hf_prompt_tokens`. Key inputs are `model_name`, `content`, `image_url`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_hf_prompt_tokens` 中。 关键输入包括 `model_name`、`content`、`image_url`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_single_chat_session_image (L223-L280)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("image_url", TEST_IMAGE_ASSETS, indirect=True)
async def test_single_chat_session_image(
    client: openai.AsyncOpenAI, model_name: str, image_url: str
):
    content_text = "What's in this image?"
    messages = dummy_messages_from_image_url(image_url, content_text)

    max_completion_tokens = 10
    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=max_completion_tokens,
        logprobs=True,
        temperature=0.0,
        top_logprobs=5,
    )
# ... 32 lines omitted for brevity ...
    messages.append({"role": "user", "content": "express your result in json"})
    await complete_and_check(
        client,
        model_name,
        messages,
        context=f"multi-turn follow-up for {image_url}",
        max_completion_tokens=10,
    )
```
**EN:** This async test validates `test_single_chat_session_image`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `image_url`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chat_completion.choices) == 1` and `choice.finish_reason == 'length'`.
**CN:** 这个异步测试验证 `test_single_chat_session_image`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`image_url`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chat_completion.choices) == 1` and `choice.finish_reason == 'length'`。

### Test / 测试: test_error_on_invalid_image_url_type (L283-L307)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("image_url", TEST_IMAGE_ASSETS, indirect=True)
async def test_error_on_invalid_image_url_type(
    client: openai.AsyncOpenAI, model_name: str, image_url: str
):
    content_text = "What's in this image?"
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "image_url", "image_url": image_url},
                {"type": "text", "text": content_text},
            ],
        }
    ]

    # image_url should be a dict {"url": "some url"}, not directly a string
    with pytest.raises(openai.BadRequestError):
        await client.chat.completions.create(
            model=model_name,
            messages=messages,
            max_completion_tokens=10,
            temperature=0.0,
        )
```
**EN:** This async test validates `test_error_on_invalid_image_url_type`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `image_url`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_error_on_invalid_image_url_type`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`image_url`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_single_chat_session_image_beamsearch (L310-L337)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("image_url", TEST_IMAGE_ASSETS, indirect=True)
async def test_single_chat_session_image_beamsearch(
    client: openai.AsyncOpenAI, model_name: str, image_url: str
):
    content_text = "What's in this image?"
    messages = dummy_messages_from_image_url(image_url, content_text)

    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        n=2,
        max_completion_tokens=10,
        logprobs=True,
        top_logprobs=5,
        extra_body=dict(use_beam_search=True),
    )
    assert len(chat_completion.choices) == 2, (
        f"Expected 2 beam search choices, got {len(chat_completion.choices)}"
    )

    content_0 = chat_completion.choices[0].message.content
    content_1 = chat_completion.choices[1].message.content
    assert content_0 != content_1, (
        f"Beam search should produce different outputs for {image_url}, "
        f"but both returned: {content_0!r}"
    )
```
**EN:** This async test validates `test_single_chat_session_image_beamsearch`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `image_url`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chat_completion.choices) == 2` and `content_0 != content_1`.
**CN:** 这个异步测试验证 `test_single_chat_session_image_beamsearch`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`image_url`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chat_completion.choices) == 2` and `content_0 != content_1`。

### Test / 测试: test_single_chat_session_image_base64encoded (L340-L406)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("raw_image_url", TEST_IMAGE_ASSETS)
@pytest.mark.parametrize("image_url", TEST_IMAGE_ASSETS, indirect=True)
async def test_single_chat_session_image_base64encoded(
    client: openai.AsyncOpenAI,
    model_name: str,
    raw_image_url: str,
    image_url: str,
    url_encoded_image: dict[str, str],
):
    content_text = "What's in this image?"
    messages = dummy_messages_from_image_url(
        url_encoded_image[raw_image_url],
        content_text,
    )

    max_completion_tokens = 10
# ... 41 lines omitted for brevity ...
    await complete_and_check(
        client,
        model_name,
        messages,
        context=f"multi-turn base64 follow-up for {raw_image_url}",
        max_completion_tokens=10,
        temperature=0.0,
    )
```
**EN:** This async test validates `test_single_chat_session_image_base64encoded`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `raw_image_url`, `image_url`, `url_encoded_image`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chat_completion.choices) == 1` and `choice.finish_reason == 'length'`.
**CN:** 这个异步测试验证 `test_single_chat_session_image_base64encoded`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`raw_image_url`、`image_url`、`url_encoded_image`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chat_completion.choices) == 1` and `choice.finish_reason == 'length'`。

### Test / 测试: test_single_chat_session_image_base64encoded_beamsearch (L409-L468)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("image_idx", list(range(len(TEST_IMAGE_ASSETS))))
async def test_single_chat_session_image_base64encoded_beamsearch(
    client: openai.AsyncOpenAI,
    model_name: str,
    image_idx: int,
    url_encoded_image: dict[str, str],
):
    # NOTE: This test validates that we pass MM data through beam search
    raw_image_url = TEST_IMAGE_ASSETS[image_idx]
    required_terms = REQUIRED_BEAM_SEARCH_TERMS[image_idx]

    messages = dummy_messages_from_image_url(url_encoded_image[raw_image_url])

    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
# ... 34 lines omitted for brevity ...
    # Verify each output contains the required terms for this image
    for i, content in enumerate([content_0, content_1]):
        assert check_output_matches_terms(content, required_terms), (
            f"Beam output {i} for image {image_idx} ({raw_image_url}) "
            f"doesn't match required terms.\n"
            f"  content: {content!r}\n"
            f"  required (all groups, >=1 per group): {required_terms}"
        )
```
**EN:** This async test validates `test_single_chat_session_image_base64encoded_beamsearch`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `image_idx`, `url_encoded_image`. It drives client-facing request creation through the API surface under test. The main assertion is `len(chat_completion.choices) == 2` and `content_0`.
**CN:** 这个异步测试验证 `test_single_chat_session_image_base64encoded_beamsearch`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`image_idx`、`url_encoded_image`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(chat_completion.choices) == 2` and `content_0`。

### Test / 测试: test_chat_streaming_image (L471-L524)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize("image_url", TEST_IMAGE_ASSETS, indirect=True)
async def test_chat_streaming_image(
    client: openai.AsyncOpenAI, model_name: str, image_url: str
):
    messages = dummy_messages_from_image_url(image_url)

    # test single completion
    chat_completion = await client.chat.completions.create(
        model=model_name,
        messages=messages,
        max_completion_tokens=10,
        temperature=0.0,
    )
    output = chat_completion.choices[0].message.content
    stop_reason = chat_completion.choices[0].finish_reason

# ... 28 lines omitted for brevity ...
    )

    streamed_text = "".join(chunks)
    assert streamed_text == output, (
        f"Streamed output doesn't match non-streamed for {image_url}.\n"
        f"  streamed:     {streamed_text!r}\n"
        f"  non-streamed: {output!r}"
    )
```
**EN:** This async test validates `test_chat_streaming_image`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `image_url`. It drives client-facing request creation through the API surface under test. The main assertion is `finish_reason_count == 1` and `chunk.choices[0].finish_reason == stop_reason`.
**CN:** 这个异步测试验证 `test_chat_streaming_image`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`image_url`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `finish_reason_count == 1` and `chunk.choices[0].finish_reason == stop_reason`。

### Test / 测试: test_multi_image_input (L527-L567)
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
    messages = dummy_messages_from_image_url(image_urls)

    if len(image_urls) > MAXIMUM_IMAGES:
        with pytest.raises(openai.BadRequestError):  # test multi-image input
            await client.chat.completions.create(
                model=model_name,
                messages=messages,
                max_completion_tokens=10,
# ... 15 lines omitted for brevity ...
        await complete_and_check(
            client,
            model_name,
            messages,
            context=f"multi-image input ({len(image_urls)} images)",
            max_completion_tokens=10,
            temperature=0.0,
        )
```
**EN:** This async test validates `test_multi_image_input`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `image_urls`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test. The main assertion is `completion.choices[0].text is not None`.
**CN:** 这个异步测试验证 `test_multi_image_input`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`image_urls`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `completion.choices[0].text is not None`。

### Test / 测试: test_completions_with_image (L570-L589)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize(
    "image_urls",
    [TEST_IMAGE_ASSETS[:i] for i in range(2, len(TEST_IMAGE_ASSETS))],
    indirect=True,
)
async def test_completions_with_image(
    client: openai.AsyncOpenAI,
    model_name: str,
    image_urls: list[str],
):
    for image_url in image_urls:
        messages = describe_image_messages(image_url)
        await complete_and_check(
            client,
            model_name,
            messages,
            context=f"completions_with_image url={image_url}",
        )
```
**EN:** This async test validates `test_completions_with_image`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `image_urls`.
**CN:** 这个异步测试验证 `test_completions_with_image`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`image_urls`。

### Test / 测试: test_completions_with_image_with_uuid (L592-L631)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize(
    "image_urls",
    [TEST_IMAGE_ASSETS[:i] for i in range(2, len(TEST_IMAGE_ASSETS))],
    indirect=True,
)
async def test_completions_with_image_with_uuid(
    client: openai.AsyncOpenAI,
    model_name: str,
    image_urls: list[str],
):
    for image_url in image_urls:
        messages = describe_image_messages(
            image_url,
            extra_image_fields={"uuid": image_url},
        )
        await complete_and_check(
# ... 14 lines omitted for brevity ...
            },
        ]
        await complete_and_check(
            client,
            model_name,
            cached_messages,
            context=f"uuid cached (empty image) uuid={image_url}",
        )
```
**EN:** This async test validates `test_completions_with_image_with_uuid`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `image_urls`.
**CN:** 这个异步测试验证 `test_completions_with_image_with_uuid`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`image_urls`。

### Test / 测试: test_completions_with_empty_image_with_uuid_without_cache_hit (L634-L657)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
async def test_completions_with_empty_image_with_uuid_without_cache_hit(
    client: openai.AsyncOpenAI,
    model_name: str,
):
    with pytest.raises(openai.BadRequestError):
        await client.chat.completions.create(
            messages=[
                {"role": "system", "content": "You are a helpful assistant."},
                {
                    "role": "user",
                    "content": [
                        {"type": "text", "text": "Describe this image."},
                        {
                            "type": "image_url",
                            "image_url": {},
                            "uuid": "uuid_not_previously_seen",
                        },
                    ],
                },
            ],
            model=model_name,
        )
```
**EN:** This async test validates `test_completions_with_empty_image_with_uuid_without_cache_hit`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个异步测试验证 `test_completions_with_empty_image_with_uuid_without_cache_hit`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

### Test / 测试: test_completions_with_image_with_incorrect_uuid_format (L660-L687)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize("model_name", [MODEL_NAME])
@pytest.mark.parametrize(
    "image_urls",
    [TEST_IMAGE_ASSETS[:i] for i in range(2, len(TEST_IMAGE_ASSETS))],
    indirect=True,
)
async def test_completions_with_image_with_incorrect_uuid_format(
    client: openai.AsyncOpenAI,
    model_name: str,
    image_urls: list[str],
):
    for image_url in image_urls:
        messages = describe_image_messages(
            image_url,
            extra_image_fields={
                "also_incorrect_uuid_key": image_url,
            },
        )
        # Inject the bad key inside image_url dict too
        messages[1]["content"][1]["image_url"]["incorrect_uuid_key"] = image_url

        await complete_and_check(
            client,
            model_name,
            messages,
            context=f"incorrect uuid format url={image_url}",
        )
```
**EN:** This async test validates `test_completions_with_image_with_incorrect_uuid_format`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `client`, `model_name`, `image_urls`.
**CN:** 这个异步测试验证 `test_completions_with_image_with_incorrect_uuid_format`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `client`、`model_name`、`image_urls`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`
- **Third-party / 第三方**: `openai`, `pytest`, `pytest_asyncio`, `transformers.AutoProcessor`
- **Project / 项目内**: `tests.utils.ROCM_ENV_OVERRIDES`, `tests.utils.ROCM_EXTRA_ARGS`, `tests.utils.RemoteOpenAIServer`, `vllm.multimodal.media.MediaWithBytes`, `vllm.multimodal.utils.encode_image_url`, `vllm.multimodal.utils.fetch_image`, `vllm.platforms.current_platform`
