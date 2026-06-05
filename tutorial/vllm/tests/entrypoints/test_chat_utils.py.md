# test_chat_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/test_chat_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers `test_chat_utils` scenarios. The file defines 61 test(s), 12 fixture(s), and 4 helper/class block(s) to validate this area. / [CN] 该文件覆盖`test_chat_utils` 场景。它定义了 61 个测试、12 个 fixture，以及 4 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L25)
```python
import warnings
from collections.abc import Mapping
from typing import Literal

import pytest
import torch

from vllm.assets.audio import AudioAsset
from vllm.assets.image import ImageAsset
from vllm.assets.video import VideoAsset
from vllm.config import ModelConfig
from vllm.entrypoints.chat_utils import (
    parse_chat_messages,
    parse_chat_messages_async,
)
from vllm.inputs import MultiModalDataDict, MultiModalUUIDDict
from vllm.multimodal.utils import (
    encode_audio_url,
    encode_image_url,
    encode_video_url,
)
from vllm.utils.serial_utils import tensor2base64
```
**EN:** Imports standard-library modules such as `collections.abc.Mapping`, `typing.Literal`, `warnings`, third-party packages like `pytest`, `torch`, project helpers such as `vllm.assets.audio.AudioAsset`, `vllm.assets.image.ImageAsset`, `vllm.assets.video.VideoAsset`.
**CN:** 导入标准库模块（如 `collections.abc.Mapping`、`typing.Literal`、`warnings`）、第三方包（如 `pytest`、`torch`）、项目内辅助模块（如 `vllm.assets.audio.AudioAsset`、`vllm.assets.image.ImageAsset`、`vllm.assets.video.VideoAsset`）。

### Module setup / 模块级配置: KIMI_K2_5_MODEL_ID, PHI3V_MODEL_ID, QWEN2AUDIO_MODEL_ID (L27-L31)
```python
KIMI_K2_5_MODEL_ID = "moonshotai/Kimi-K2.5"
PHI3V_MODEL_ID = "microsoft/Phi-3.5-vision-instruct"
QWEN2AUDIO_MODEL_ID = "Qwen/Qwen2-Audio-7B-Instruct"
QWEN25OMNI_MODEL_ID = "Qwen/Qwen2.5-Omni-7B"
MISTRAL_MODEL_ID = "mistralai/Mistral-Small-3.1-24B-Instruct-2503"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `KIMI_K2_5_MODEL_ID`, `PHI3V_MODEL_ID`, `QWEN2AUDIO_MODEL_ID`, `QWEN25OMNI_MODEL_ID`, `MISTRAL_MODEL_ID`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `KIMI_K2_5_MODEL_ID`、`PHI3V_MODEL_ID`、`QWEN2AUDIO_MODEL_ID`、`QWEN25OMNI_MODEL_ID`、`MISTRAL_MODEL_ID`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: kimi_k2_5_model_config (L34-L43)
```python
@pytest.fixture(scope="function")
def kimi_k2_5_model_config():
    return ModelConfig(
        KIMI_K2_5_MODEL_ID,
        runner="generate",
        trust_remote_code=True,
        limit_mm_per_prompt={
            "image": 2,
        },
    )
```
**EN:** This fixture prepares `kimi_k2_5_model_config` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `kimi_k2_5_model_config`。

### Fixture / 夹具: phi3v_model_config (L46-L55)
```python
@pytest.fixture(scope="function")
def phi3v_model_config():
    return ModelConfig(
        PHI3V_MODEL_ID,
        runner="generate",
        trust_remote_code=True,
        limit_mm_per_prompt={
            "image": 2,
        },
    )
```
**EN:** This fixture prepares `phi3v_model_config` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `phi3v_model_config`。

### Fixture / 夹具: phi3v_model_config_mm_interleaved (L58-L68)
```python
@pytest.fixture(scope="function")
def phi3v_model_config_mm_interleaved():
    return ModelConfig(
        PHI3V_MODEL_ID,
        runner="generate",
        trust_remote_code=True,
        interleave_mm_strings=True,
        limit_mm_per_prompt={
            "image": 2,
        },
    )
```
**EN:** This fixture prepares `phi3v_model_config_mm_interleaved` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `phi3v_model_config_mm_interleaved`。

### Fixture / 夹具: phi3v_model_config_image_embeds (L71-L81)
```python
@pytest.fixture(scope="function")
def phi3v_model_config_image_embeds():
    return ModelConfig(
        PHI3V_MODEL_ID,
        runner="generate",
        trust_remote_code=True,
        limit_mm_per_prompt={
            "image": 2,
        },
        enable_mm_embeds=True,
    )
```
**EN:** This fixture prepares `phi3v_model_config_image_embeds` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `phi3v_model_config_image_embeds`。

### Fixture / 夹具: qwen25omni_model_config_image_embeds (L84-L91)
```python
@pytest.fixture(scope="function")
def qwen25omni_model_config_image_embeds():
    return ModelConfig(
        QWEN25OMNI_MODEL_ID,
        runner="generate",
        limit_mm_per_prompt={"image": 2},
        enable_mm_embeds=True,
    )
```
**EN:** This fixture prepares `qwen25omni_model_config_image_embeds` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `qwen25omni_model_config_image_embeds`。

### Fixture / 夹具: qwen2_audio_model_config (L94-L103)
```python
@pytest.fixture(scope="function")
def qwen2_audio_model_config():
    return ModelConfig(
        QWEN2AUDIO_MODEL_ID,
        runner="generate",
        trust_remote_code=True,
        limit_mm_per_prompt={
            "audio": 1,
        },
    )
```
**EN:** This fixture prepares `qwen2_audio_model_config` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `qwen2_audio_model_config`。

### Fixture / 夹具: audio_embeds_model_config (L106-L116)
```python
@pytest.fixture(scope="function")
def audio_embeds_model_config():
    return ModelConfig(
        QWEN2AUDIO_MODEL_ID,
        runner="generate",
        trust_remote_code=True,
        limit_mm_per_prompt={
            "audio": 2,
        },
        enable_mm_embeds=True,
    )
```
**EN:** This fixture prepares `audio_embeds_model_config` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `audio_embeds_model_config`。

### Fixture / 夹具: qwen25omni_model_config_mm_interleaved (L119-L130)
```python
@pytest.fixture(scope="function")
def qwen25omni_model_config_mm_interleaved():
    return ModelConfig(
        QWEN25OMNI_MODEL_ID,
        runner="generate",
        interleave_mm_strings=True,
        limit_mm_per_prompt={
            "image": 2,
            "audio": 1,
            "video": 1,
        },
    )
```
**EN:** This fixture prepares `qwen25omni_model_config_mm_interleaved` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `qwen25omni_model_config_mm_interleaved`。

### Fixture / 夹具: mistral_model_config (L133-L141)
```python
@pytest.fixture(scope="function")
def mistral_model_config():
    return ModelConfig(
        MISTRAL_MODEL_ID,
        runner="generate",
        limit_mm_per_prompt={
            "image": 2,
        },
    )
```
**EN:** This fixture prepares `mistral_model_config` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `mistral_model_config`。

### Fixture / 夹具: image_url (L144-L147)
```python
@pytest.fixture(scope="module")
def image_url():
    image = ImageAsset("cherry_blossom")
    return encode_image_url(image.pil_image)
```
**EN:** This fixture prepares `image_url` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `image_url`。

### Fixture / 夹具: video_url (L150-L153)
```python
@pytest.fixture(scope="module")
def video_url():
    video = VideoAsset("baby_reading", 1)
    return encode_video_url(video.np_ndarrays)
```
**EN:** This fixture prepares `video_url` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `video_url`。

### Fixture / 夹具: audio_url (L156-L159)
```python
@pytest.fixture(scope="module")
def audio_url():
    audio = AudioAsset("mary_had_lamb")
    return encode_audio_url(*audio.audio_and_sample_rate)
```
**EN:** This fixture prepares `audio_url` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `audio_url`。

### Helper / 辅助函数: _assert_mm_data_is_image_input (L162-L176)
```python
def _assert_mm_data_is_image_input(
    mm_data: MultiModalDataDict | None,
    image_count: int,
    skipped_image_indices: list | None = None,
) -> None:
    assert mm_data is not None
    assert set(mm_data.keys()) == {"image"}

    image_data = mm_data.get("image")
    assert image_data is not None

    assert isinstance(image_data, list) and len(image_data) == image_count
    if skipped_image_indices is not None:
        for i in skipped_image_indices:
            assert image_data[i] is None
```
**EN:** This helper encapsulates reusable logic in `_assert_mm_data_is_image_input`. Key inputs are `mm_data`, `image_count`, `skipped_image_indices`. The main assertion is `mm_data is not None` and `set(mm_data.keys()) == {'image'}`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_assert_mm_data_is_image_input` 中。 关键输入包括 `mm_data`、`image_count`、`skipped_image_indices`。 核心断言是 `mm_data is not None` and `set(mm_data.keys()) == {'image'}`。

### Helper / 辅助函数: _assert_mm_data_is_vision_chunk_input (L179-L192)
```python
def _assert_mm_data_is_vision_chunk_input(
    mm_data: MultiModalDataDict | None,
    vision_chunk_count: int,
) -> None:
    assert mm_data is not None
    assert set(mm_data.keys()) == {"vision_chunk"}

    vision_chunk_data = mm_data.get("vision_chunk")
    assert vision_chunk_data is not None

    assert (
        isinstance(vision_chunk_data, list)
        and len(vision_chunk_data) == vision_chunk_count
    )
```
**EN:** This helper encapsulates reusable logic in `_assert_mm_data_is_vision_chunk_input`. Key inputs are `mm_data`, `vision_chunk_count`. The main assertion is `mm_data is not None` and `set(mm_data.keys()) == {'vision_chunk'}`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_assert_mm_data_is_vision_chunk_input` 中。 关键输入包括 `mm_data`、`vision_chunk_count`。 核心断言是 `mm_data is not None` and `set(mm_data.keys()) == {'vision_chunk'}`。

### Helper / 辅助函数: _assert_mm_uuids (L195-L212)
```python
def _assert_mm_uuids(
    mm_uuids: MultiModalUUIDDict | None,
    media_count: int,
    expected_uuids: list[str | None],
    modality: str = "image",
) -> None:
    if len(expected_uuids) > 0:
        assert mm_uuids is not None
        assert modality in mm_uuids

        image_uuids = mm_uuids.get(modality)
        assert image_uuids is not None

        assert isinstance(image_uuids, list) and len(image_uuids) == media_count

        assert image_uuids == expected_uuids
    else:
        assert mm_uuids is None
```
**EN:** This helper encapsulates reusable logic in `_assert_mm_uuids`. Key inputs are `mm_uuids`, `media_count`, `expected_uuids`, `modality`. The main assertion is `mm_uuids is not None` and `modality in mm_uuids`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_assert_mm_uuids` 中。 关键输入包括 `mm_uuids`、`media_count`、`expected_uuids`、`modality`。 核心断言是 `mm_uuids is not None` and `modality in mm_uuids`。

### Module setup / 模块级配置: ModalityType, MultiModalDataCounts (L215-L216)
```python
ModalityType = Literal["image", "video", "audio"]
MultiModalDataCounts = Mapping[ModalityType, int]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `ModalityType`, `MultiModalDataCounts`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `ModalityType`、`MultiModalDataCounts`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _assert_mm_data_inputs (L219-L236)
```python
def _assert_mm_data_inputs(
    mm_data: MultiModalDataDict | None,
    data_count: MultiModalDataCounts,
    skipped_media_indices: dict[str, list] | None = None,  # modality -> list[int]
) -> None:
    assert mm_data is not None
    assert set(data_count.keys()) == (set(mm_data.keys()))

    for modality, n in data_count.items():
        modality_data = mm_data.get(modality)
        assert modality_data is not None
        assert isinstance(modality_data, list) and len(modality_data) == n

        if skipped_media_indices is not None:
            skipped_media_indices_for_modality = skipped_media_indices.get(modality)
            assert skipped_media_indices_for_modality is not None
            for i in skipped_media_indices_for_modality:
                assert modality_data[i] is None
```
**EN:** This helper encapsulates reusable logic in `_assert_mm_data_inputs`. Key inputs are `mm_data`, `data_count`, `skipped_media_indices`. The main assertion is `mm_data is not None` and `set(data_count.keys()) == set(mm_data.keys())`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_assert_mm_data_inputs` 中。 关键输入包括 `mm_data`、`data_count`、`skipped_media_indices`。 核心断言是 `mm_data is not None` and `set(data_count.keys()) == set(mm_data.keys())`。

### Test / 测试: test_parse_chat_messages_single_image (L239-L261)
```python
def test_parse_chat_messages_single_image(
    phi3v_model_config,
    image_url,
):
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {"type": "image_url", "image_url": {"url": image_url}},
                    {"type": "text", "text": "What's in the image?"},
                ],
            }
        ],
        phi3v_model_config,
        content_format="string",
    )

    assert conversation == [
        {"role": "user", "content": "<|image_1|>\nWhat's in the image?"}
    ]
    _assert_mm_data_is_image_input(mm_data, 1)
    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[None])
```
**EN:** This test validates `test_parse_chat_messages_single_image`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in the image?"}]`.
**CN:** 这个测试验证 `test_parse_chat_messages_single_image`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in the image?"}]`。

### Test / 测试: test_parse_chat_messages_single_image_with_uuid (L264-L293)
```python
def test_parse_chat_messages_single_image_with_uuid(
    phi3v_model_config,
    image_url,
):
    image_uuid = str(hash(image_url))
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": image_url,
                        },
                        "uuid": image_uuid,
                    },
                    {"type": "text", "text": "What's in the image?"},
                ],
            }
        ],
        phi3v_model_config,
        content_format="string",
    )

    assert conversation == [
        {"role": "user", "content": "<|image_1|>\nWhat's in the image?"}
    ]
    _assert_mm_data_is_image_input(mm_data, 1)
    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[image_uuid])
```
**EN:** This test validates `test_parse_chat_messages_single_image_with_uuid`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in the image?"}]`.
**CN:** 这个测试验证 `test_parse_chat_messages_single_image_with_uuid`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in the image?"}]`。

### Test / 测试: test_parse_chat_messages_single_empty_image_with_uuid (L296-L323)
```python
def test_parse_chat_messages_single_empty_image_with_uuid(
    phi3v_model_config,
    image_url,
):
    image_uuid = str(hash(image_url))
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": None,
                        "uuid": image_uuid,
                    },
                    {"type": "text", "text": "What's in the image?"},
                ],
            }
        ],
        phi3v_model_config,
        content_format="string",
    )

    assert conversation == [
        {"role": "user", "content": "<|image_1|>\nWhat's in the image?"}
    ]
    _assert_mm_data_is_image_input(mm_data, 1, skipped_image_indices=[0])
    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[image_uuid])
```
**EN:** This test validates `test_parse_chat_messages_single_empty_image_with_uuid`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in the image?"}]`.
**CN:** 这个测试验证 `test_parse_chat_messages_single_empty_image_with_uuid`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in the image?"}]`。

### Test / 测试: test_parse_chat_messages_single_image_with_bad_uuid_format (L326-L356)
```python
def test_parse_chat_messages_single_image_with_bad_uuid_format(
    phi3v_model_config,
    image_url,
):
    image_uuid = str(hash(image_url))
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": image_url,
                            "uuid": image_uuid,
                        },
                        "bad_uuid_key": image_uuid,
                    },
# ... 5 lines omitted for brevity ...
        content_format="string",
    )

    assert conversation == [
        {"role": "user", "content": "<|image_1|>\nWhat's in the image?"}
    ]
    _assert_mm_data_is_image_input(mm_data, 1)
    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[None])
```
**EN:** This test validates `test_parse_chat_messages_single_image_with_bad_uuid_format`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in the image?"}]`.
**CN:** 这个测试验证 `test_parse_chat_messages_single_image_with_bad_uuid_format`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in the image?"}]`。

### Test / 测试: test_parse_chat_messages_multiple_images_with_uuids (L359-L400)
```python
def test_parse_chat_messages_multiple_images_with_uuids(
    phi3v_model_config,
    image_url,
):
    image_uuid1 = "my_uuid_1"
    image_uuid2 = "my_uuid_2"

    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": image_url,
                        },
                        "uuid": image_uuid1,
# ... 16 lines omitted for brevity ...
    assert conversation == [
        {
            "role": "user",
            "content": "<|image_1|>\n<|image_2|>\nWhat's in the image?",
        }
    ]
    _assert_mm_data_is_image_input(mm_data, 2)
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[image_uuid1, image_uuid2])
```
**EN:** This test validates `test_parse_chat_messages_multiple_images_with_uuids`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\n<|image_2|>\nWhat's in the image?"}]`.
**CN:** 这个测试验证 `test_parse_chat_messages_multiple_images_with_uuids`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\n<|image_2|>\nWhat's in the image?"}]`。

### Test / 测试: test_parse_chat_messages_multiple_empty_images_with_uuids (L403-L440)
```python
def test_parse_chat_messages_multiple_empty_images_with_uuids(
    phi3v_model_config,
    image_url,
):
    image_uuid1 = "my_uuid_1"
    image_uuid2 = "my_uuid_2"

    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": None,
                        "uuid": image_uuid1,
                    },
                    {
# ... 12 lines omitted for brevity ...
    assert conversation == [
        {
            "role": "user",
            "content": "<|image_1|>\n<|image_2|>\nWhat's in the image?",
        }
    ]
    _assert_mm_data_is_image_input(mm_data, 2, skipped_image_indices=[0, 1])
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[image_uuid1, image_uuid2])
```
**EN:** This test validates `test_parse_chat_messages_multiple_empty_images_with_uuids`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\n<|image_2|>\nWhat's in the image?"}]`.
**CN:** 这个测试验证 `test_parse_chat_messages_multiple_empty_images_with_uuids`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\n<|image_2|>\nWhat's in the image?"}]`。

### Test / 测试: test_parse_chat_messages_mixed_empty_images_with_uuids (L443-L482)
```python
def test_parse_chat_messages_mixed_empty_images_with_uuids(
    phi3v_model_config,
    image_url,
):
    image_uuid1 = "my_uuid_1"
    image_uuid2 = "my_uuid_2"

    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": image_url,
                        },
                        "uuid": image_uuid1,
# ... 14 lines omitted for brevity ...
    assert conversation == [
        {
            "role": "user",
            "content": "<|image_1|>\n<|image_2|>\nWhat's in the image?",
        }
    ]
    _assert_mm_data_is_image_input(mm_data, 2, skipped_image_indices=[1])
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[image_uuid1, image_uuid2])
```
**EN:** This test validates `test_parse_chat_messages_mixed_empty_images_with_uuids`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\n<|image_2|>\nWhat's in the image?"}]`.
**CN:** 这个测试验证 `test_parse_chat_messages_mixed_empty_images_with_uuids`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\n<|image_2|>\nWhat's in the image?"}]`。

### Test / 测试: test_parse_chat_messages_single_image_with_uuid_async (L485-L513)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_single_image_with_uuid_async(
    phi3v_model_config,
    image_url,
):
    image_uuid = str(hash(image_url))
    conversation, mm_data, mm_uuids = await parse_chat_messages_async(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": {"url": image_url},
                        "uuid": image_uuid,
                    },
                    {"type": "text", "text": "What's in the image?"},
                ],
            }
        ],
        phi3v_model_config,
        content_format="string",
    )

    assert conversation == [
        {"role": "user", "content": "<|image_1|>\nWhat's in the image?"}
    ]
    _assert_mm_data_is_image_input(mm_data, 1)
    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[image_uuid])
```
**EN:** This async test validates `test_parse_chat_messages_single_image_with_uuid_async`. Relevant pytest markers include `asyncio`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in the image?"}]`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_single_image_with_uuid_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in the image?"}]`。

### Test / 测试: test_parse_chat_messages_empty_image_with_uuid_async (L516-L544)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_empty_image_with_uuid_async(
    phi3v_model_config,
    image_url,
):
    image_uuid = str(hash(image_url))
    conversation, mm_data, mm_uuids = await parse_chat_messages_async(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": None,
                        "uuid": image_uuid,
                    },
                    {"type": "text", "text": "What's in the image?"},
                ],
            }
        ],
        phi3v_model_config,
        content_format="string",
    )

    assert conversation == [
        {"role": "user", "content": "<|image_1|>\nWhat's in the image?"}
    ]
    _assert_mm_data_is_image_input(mm_data, 1, skipped_image_indices=[0])
    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[image_uuid])
```
**EN:** This async test validates `test_parse_chat_messages_empty_image_with_uuid_async`. Relevant pytest markers include `asyncio`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in the image?"}]`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_empty_image_with_uuid_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in the image?"}]`。

### Test / 测试: test_parse_chat_messages_multiple_images_with_uuids_async (L547-L585)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_multiple_images_with_uuids_async(
    phi3v_model_config,
    image_url,
):
    image_uuid1 = "my_uuid_1"
    image_uuid2 = "my_uuid_2"

    conversation, mm_data, mm_uuids = await parse_chat_messages_async(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": {"url": image_url},
                        "uuid": image_uuid1,
                    },
# ... 13 lines omitted for brevity ...
    assert conversation == [
        {
            "role": "user",
            "content": "<|image_1|>\n<|image_2|>\nWhat's in these images?",
        }
    ]
    _assert_mm_data_is_image_input(mm_data, 2)
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[image_uuid1, image_uuid2])
```
**EN:** This async test validates `test_parse_chat_messages_multiple_images_with_uuids_async`. Relevant pytest markers include `asyncio`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\n<|image_2|>\nWhat's in these images?"}]`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_multiple_images_with_uuids_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\n<|image_2|>\nWhat's in these images?"}]`。

### Test / 测试: test_parse_chat_messages_multiple_empty_images_with_uuids_async (L588-L626)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_multiple_empty_images_with_uuids_async(
    phi3v_model_config,
    image_url,
):
    image_uuid1 = "my_uuid_1"
    image_uuid2 = "my_uuid_2"

    conversation, mm_data, mm_uuids = await parse_chat_messages_async(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": None,
                        "uuid": image_uuid1,
                    },
# ... 13 lines omitted for brevity ...
    assert conversation == [
        {
            "role": "user",
            "content": "<|image_1|>\n<|image_2|>\nWhat's in these images?",
        }
    ]
    _assert_mm_data_is_image_input(mm_data, 2, skipped_image_indices=[0, 1])
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[image_uuid1, image_uuid2])
```
**EN:** This async test validates `test_parse_chat_messages_multiple_empty_images_with_uuids_async`. Relevant pytest markers include `asyncio`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\n<|image_2|>\nWhat's in these images?"}]`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_multiple_empty_images_with_uuids_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\n<|image_2|>\nWhat's in these images?"}]`。

### Test / 测试: test_parse_chat_messages_multiple_images_with_partial_uuids_async (L629-L665)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_multiple_images_with_partial_uuids_async(
    phi3v_model_config,
    image_url,
):
    image_uuid2 = "my_uuid_2"

    conversation, mm_data, mm_uuids = await parse_chat_messages_async(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": {"url": image_url},
                    },
                    {
                        "type": "image_pil",
# ... 11 lines omitted for brevity ...
    assert conversation == [
        {
            "role": "user",
            "content": "<|image_1|>\n<|image_2|>\nWhat's in these images?",
        }
    ]
    _assert_mm_data_is_image_input(mm_data, 2)
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[None, image_uuid2])
```
**EN:** This async test validates `test_parse_chat_messages_multiple_images_with_partial_uuids_async`. Relevant pytest markers include `asyncio`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\n<|image_2|>\nWhat's in these images?"}]`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_multiple_images_with_partial_uuids_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\n<|image_2|>\nWhat's in these images?"}]`。

### Test / 测试: test_parse_chat_messages_empty_system (L668-L703)
```python
def test_parse_chat_messages_empty_system(
    mistral_model_config,
):
    # Test string format
    conversation, _, _ = parse_chat_messages(
        [
            {"role": "system", "content": ""},
            {
                "role": "user",
                "content": [{"type": "text", "text": "Who are you?"}],
            },
        ],
        mistral_model_config,
        content_format="string",
    )
    assert conversation == [
        {"role": "system", "content": ""},
        {"role": "user", "content": "Who are you?"},
# ... 10 lines omitted for brevity ...
        ],
        mistral_model_config,
        content_format="openai",
    )
    assert conversation == [
        {"role": "system", "content": [{"type": "text", "text": ""}]},
        {"role": "user", "content": [{"type": "text", "text": "Who are you?"}]},
    ]
```
**EN:** This test validates `test_parse_chat_messages_empty_system`. Key inputs are `mistral_model_config`. The main assertion is `conversation == [{'role': 'system', 'content': ''}, {'role': 'user', 'content': 'Who are you?'}]` and `conversation == [{'role': 'system', 'content': [{'type': 'text', 'text': ''}]}, {'role': 'user', 'content': [{'type': 'text', 'text': 'Wh...`.
**CN:** 这个测试验证 `test_parse_chat_messages_empty_system`。 关键输入包括 `mistral_model_config`。 核心断言是 `conversation == [{'role': 'system', 'content': ''}, {'role': 'user', 'content': 'Who are you?'}]` and `conversation == [{'role': 'system', 'content': [{'type': 'text', 'text': ''}]}, {'role': 'user', 'content': [{'type': 'text', 'text': 'Wh...`。

### Test / 测试: test_parse_chat_messages_single_image_async (L706-L729)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_single_image_async(
    phi3v_model_config,
    image_url,
):
    conversation, mm_data, mm_uuids = await parse_chat_messages_async(
        [
            {
                "role": "user",
                "content": [
                    {"type": "image_url", "image_url": {"url": image_url}},
                    {"type": "text", "text": "What's in the image?"},
                ],
            }
        ],
        phi3v_model_config,
        content_format="string",
    )

    assert conversation == [
        {"role": "user", "content": "<|image_1|>\nWhat's in the image?"}
    ]
    _assert_mm_data_is_image_input(mm_data, 1)
    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[None])
```
**EN:** This async test validates `test_parse_chat_messages_single_image_async`. Relevant pytest markers include `asyncio`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in the image?"}]`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_single_image_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in the image?"}]`。

### Test / 测试: test_parse_chat_messages_multiple_images (L732-L761)
```python
def test_parse_chat_messages_multiple_images(
    phi3v_model_config,
    image_url,
):
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {"type": "image_url", "image_url": {"url": image_url}},
                    {
                        "type": "image_pil",
                        "image_pil": ImageAsset("cherry_blossom").pil_image,
                    },
                    {"type": "text", "text": "What's in these images?"},
                ],
            }
        ],
        phi3v_model_config,
        content_format="string",
    )

    assert conversation == [
        {
            "role": "user",
            "content": "<|image_1|>\n<|image_2|>\nWhat's in these images?",
        }
    ]
    _assert_mm_data_is_image_input(mm_data, 2)
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[None, None])
```
**EN:** This test validates `test_parse_chat_messages_multiple_images`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\n<|image_2|>\nWhat's in these images?"}]`.
**CN:** 这个测试验证 `test_parse_chat_messages_multiple_images`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\n<|image_2|>\nWhat's in these images?"}]`。

### Test / 测试: test_parse_chat_messages_empty_pil_image_with_uuid (L764-L789)
```python
def test_parse_chat_messages_empty_pil_image_with_uuid(
    phi3v_model_config,
):
    uuid = "abcd"
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {"type": "image_pil", "image_pil": None, "uuid": uuid},
                    {"type": "text", "text": "What's in this image?"},
                ],
            }
        ],
        phi3v_model_config,
        content_format="string",
    )

    assert conversation == [
        {
            "role": "user",
            "content": "<|image_1|>\nWhat's in this image?",
        }
    ]
    _assert_mm_data_is_image_input(mm_data, 1, skipped_image_indices=[0])
    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[uuid])
```
**EN:** This test validates `test_parse_chat_messages_empty_pil_image_with_uuid`. Key inputs are `phi3v_model_config`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in this image?"}]`.
**CN:** 这个测试验证 `test_parse_chat_messages_empty_pil_image_with_uuid`。 关键输入包括 `phi3v_model_config`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in this image?"}]`。

### Test / 测试: test_parse_chat_messages_empty_image_embeds_with_uuid (L792-L823)
```python
def test_parse_chat_messages_empty_image_embeds_with_uuid(
    phi3v_model_config_image_embeds,
):
    uuid = "abcd"
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {"type": "image_embeds", "image_embeds": None, "uuid": uuid},
                    {"type": "text", "text": "What's in this image?"},
                ],
            }
        ],
        phi3v_model_config_image_embeds,
        content_format="string",
    )

# ... 6 lines omitted for brevity ...

    assert mm_data is not None
    assert "image" in mm_data
    assert isinstance(mm_data["image"], list)
    assert len(mm_data["image"]) == 1
    assert mm_data["image"][0] is None

    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[uuid])
```
**EN:** This test validates `test_parse_chat_messages_empty_image_embeds_with_uuid`. Key inputs are `phi3v_model_config_image_embeds`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in this image?"}]` and `mm_data is not None`.
**CN:** 这个测试验证 `test_parse_chat_messages_empty_image_embeds_with_uuid`。 关键输入包括 `phi3v_model_config_image_embeds`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in this image?"}]` and `mm_data is not None`。

### Test / 测试: test_parse_chat_messages_empty_audio_embeds_with_uuid (L826-L854)
```python
def test_parse_chat_messages_empty_audio_embeds_with_uuid(
    audio_embeds_model_config,
):
    """Test audio_embeds with UUID (no actual embeds data)."""
    uuid = "test-audio-uuid-123"

    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": "Describe this audio"},
                    {"type": "audio_embeds", "audio_embeds": None, "uuid": uuid},
                ],
            }
        ],
        audio_embeds_model_config,
        content_format="string",
    )

    # Should have audio in mm_data as None (UUID provided)
    assert mm_data is not None
    assert "audio" in mm_data
    assert isinstance(mm_data["audio"], list)
    assert len(mm_data["audio"]) == 1
    assert mm_data["audio"][0] is None

    # UUID should be recorded
    _assert_mm_uuids(mm_uuids, 1, modality="audio", expected_uuids=[uuid])
```
**EN:** This test validates `test_parse_chat_messages_empty_audio_embeds_with_uuid`. Key inputs are `audio_embeds_model_config`. The main assertion is `mm_data is not None` and `'audio' in mm_data`.
**CN:** 这个测试验证 `test_parse_chat_messages_empty_audio_embeds_with_uuid`。 关键输入包括 `audio_embeds_model_config`。 核心断言是 `mm_data is not None` and `'audio' in mm_data`。

### Test / 测试: test_parse_chat_messages_audio_embeds_with_string (L857-L896)
```python
def test_parse_chat_messages_audio_embeds_with_string(
    audio_embeds_model_config,
):
    """Test audio_embeds with base64 string embedding data."""

    import torch

    # Create a sample audio embedding tensor
    hidden_size = audio_embeds_model_config.get_inputs_embeds_size()
    audio_embedding = torch.randn(1, 128, hidden_size)

    # Encode it as base64
    base64_audio_embedding = tensor2base64(audio_embedding)

    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
# ... 14 lines omitted for brevity ...
    assert mm_data is not None
    assert "audio" in mm_data
    assert isinstance(mm_data["audio"], torch.Tensor)
    assert mm_data["audio"].shape == audio_embedding.shape
    # No UUID provided
    assert mm_uuids is not None
    assert "audio" in mm_uuids
    _assert_mm_uuids(mm_uuids, 1, modality="audio", expected_uuids=[None])
```
**EN:** This test validates `test_parse_chat_messages_audio_embeds_with_string`. Key inputs are `audio_embeds_model_config`. The main assertion is `mm_data is not None` and `'audio' in mm_data`.
**CN:** 这个测试验证 `test_parse_chat_messages_audio_embeds_with_string`。 关键输入包括 `audio_embeds_model_config`。 核心断言是 `mm_data is not None` and `'audio' in mm_data`。

### Test / 测试: test_parse_chat_messages_audio_embeds_async (L899-L939)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_audio_embeds_async(
    audio_embeds_model_config,
):
    """Test audio_embeds with async futures."""

    import torch

    # Create a sample audio embedding tensor
    hidden_size = audio_embeds_model_config.get_inputs_embeds_size()
    audio_embedding = torch.randn(1, 128, hidden_size)

    # Encode it as base64
    base64_audio_embedding = tensor2base64(audio_embedding)

    conversation, mm_data, mm_uuids = await parse_chat_messages_async(
        [
            {
# ... 15 lines omitted for brevity ...
    assert mm_data is not None
    assert "audio" in mm_data
    assert isinstance(mm_data["audio"], torch.Tensor)
    assert mm_data["audio"].shape == audio_embedding.shape
    # No UUID provided
    assert mm_uuids is not None
    assert "audio" in mm_uuids
    _assert_mm_uuids(mm_uuids, 1, modality="audio", expected_uuids=[None])
```
**EN:** This async test validates `test_parse_chat_messages_audio_embeds_async`. Relevant pytest markers include `asyncio`. Key inputs are `audio_embeds_model_config`. The main assertion is `mm_data is not None` and `'audio' in mm_data`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_audio_embeds_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `audio_embeds_model_config`。 核心断言是 `mm_data is not None` and `'audio' in mm_data`。

### Test / 测试: test_parse_chat_messages_multiple_image_embeds (L942-L1002)
```python
def test_parse_chat_messages_multiple_image_embeds(
    phi3v_model_config_image_embeds,
):
    """Test that multiple image_embeds in a single message are now supported.

    This test validates the fix for the limitation that previously only allowed
    one message with {'type': 'image_embeds'}. Now multiple image embeddings
    can be provided in a single request, similar to regular images.
    """
    # Create two sample image embedding tensors
    hidden_size = phi3v_model_config_image_embeds.get_inputs_embeds_size()
    image_embedding_1 = torch.randn(256, hidden_size)
    image_embedding_2 = torch.randn(128, hidden_size)

    # Encode them as base64 using the convenience function
    base64_image_embedding_1 = tensor2base64(image_embedding_1)
    base64_image_embedding_2 = tensor2base64(image_embedding_2)

# ... 35 lines omitted for brevity ...
    # Verify each embedding has the correct shape
    assert isinstance(mm_data["image"][0], torch.Tensor)
    assert mm_data["image"][0].shape == image_embedding_1.shape
    assert isinstance(mm_data["image"][1], torch.Tensor)
    assert mm_data["image"][1].shape == image_embedding_2.shape

    # Verify UUIDs (None since we didn't provide any)
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[None, None])
```
**EN:** This test validates `test_parse_chat_messages_multiple_image_embeds`. Key inputs are `phi3v_model_config_image_embeds`. The main assertion is `conversation == [{'role': 'user', 'content': '<|image_1|>\n<|image_2|>\nDescribe these two images.'}]` and `mm_data is not None`.
**CN:** 这个测试验证 `test_parse_chat_messages_multiple_image_embeds`。 关键输入包括 `phi3v_model_config_image_embeds`。 核心断言是 `conversation == [{'role': 'user', 'content': '<|image_1|>\n<|image_2|>\nDescribe these two images.'}]` and `mm_data is not None`。

### Test / 测试: test_parse_chat_messages_multiple_image_embeds_with_uuids (L1005-L1055)
```python
def test_parse_chat_messages_multiple_image_embeds_with_uuids(
    phi3v_model_config_image_embeds,
):
    """Test multiple image_embeds with UUIDs.

    This validates that UUIDs are properly tracked for multiple embeddings.
    """
    uuid1 = "image-uuid-1"
    uuid2 = "image-uuid-2"

    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_embeds",
                        "image_embeds": None,
# ... 25 lines omitted for brevity ...
    assert "image" in mm_data
    assert isinstance(mm_data["image"], list)
    assert len(mm_data["image"]) == 2
    assert mm_data["image"][0] is None
    assert mm_data["image"][1] is None

    # Verify UUIDs are correctly tracked
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[uuid1, uuid2])
```
**EN:** This test validates `test_parse_chat_messages_multiple_image_embeds_with_uuids`. Key inputs are `phi3v_model_config_image_embeds`. The main assertion is `conversation == [{'role': 'user', 'content': '<|image_1|>\n<|image_2|>\nCompare these images.'}]` and `mm_data is not None`.
**CN:** 这个测试验证 `test_parse_chat_messages_multiple_image_embeds_with_uuids`。 关键输入包括 `phi3v_model_config_image_embeds`。 核心断言是 `conversation == [{'role': 'user', 'content': '<|image_1|>\n<|image_2|>\nCompare these images.'}]` and `mm_data is not None`。

### Test / 测试: test_parse_chat_messages_multiple_image_embeds_async (L1058-L1117)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_multiple_image_embeds_async(
    phi3v_model_config_image_embeds,
):
    """Test multiple image_embeds with async parsing.

    This validates the AsyncMultiModalItemTracker also supports multiple embeddings.
    """
    # Create two sample image embedding tensors
    hidden_size = phi3v_model_config_image_embeds.get_inputs_embeds_size()
    image_embedding_1 = torch.randn(200, hidden_size)
    image_embedding_2 = torch.randn(150, hidden_size)

    # Encode them as base64 using the convenience function
    base64_image_embedding_1 = tensor2base64(image_embedding_1)
    base64_image_embedding_2 = tensor2base64(image_embedding_2)

    conversation, mm_data, mm_uuids = await parse_chat_messages_async(
# ... 34 lines omitted for brevity ...
    # Verify each embedding has the correct shape
    assert isinstance(mm_data["image"][0], torch.Tensor)
    assert mm_data["image"][0].shape == image_embedding_1.shape
    assert isinstance(mm_data["image"][1], torch.Tensor)
    assert mm_data["image"][1].shape == image_embedding_2.shape

    # Verify UUIDs
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[None, None])
```
**EN:** This async test validates `test_parse_chat_messages_multiple_image_embeds_async`. Relevant pytest markers include `asyncio`. Key inputs are `phi3v_model_config_image_embeds`. The main assertion is `conversation == [{'role': 'user', 'content': '<|image_1|>\n<|image_2|>\nWhat do these images show?'}]` and `mm_data is not None`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_multiple_image_embeds_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `phi3v_model_config_image_embeds`。 核心断言是 `conversation == [{'role': 'user', 'content': '<|image_1|>\n<|image_2|>\nWhat do these images show?'}]` and `mm_data is not None`。

### Test / 测试: test_parse_chat_messages_empty_image_embeds_with_uuid_async (L1120-L1151)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_empty_image_embeds_with_uuid_async(
    phi3v_model_config_image_embeds,
):
    uuid = "abcd"
    conversation, mm_data, mm_uuids = await parse_chat_messages_async(
        [
            {
                "role": "user",
                "content": [
                    {"type": "image_embeds", "image_embeds": None, "uuid": uuid},
                    {"type": "text", "text": "What's in this image?"},
                ],
            }
        ],
        phi3v_model_config_image_embeds,
        content_format="string",
    )
# ... 6 lines omitted for brevity ...
    ]
    assert mm_data is not None
    assert "image" in mm_data
    assert isinstance(mm_data["image"], list)
    assert len(mm_data["image"]) == 1
    assert mm_data["image"][0] is None

    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[uuid])
```
**EN:** This async test validates `test_parse_chat_messages_empty_image_embeds_with_uuid_async`. Relevant pytest markers include `asyncio`. Key inputs are `phi3v_model_config_image_embeds`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in this image?"}]` and `mm_data is not None`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_empty_image_embeds_with_uuid_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `phi3v_model_config_image_embeds`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in this image?"}]` and `mm_data is not None`。

### Test / 测试: test_parse_chat_messages_empty_dict_image_embeds (L1154-L1187)
```python
def test_parse_chat_messages_empty_dict_image_embeds(
    phi3v_model_config_image_embeds,
):
    """Test that empty dictionary for image_embeds is handled without errors."""
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {"type": "image_embeds", "image_embeds": {}},
                    {"type": "text", "text": "What's in this image?"},
                ],
            }
        ],
        phi3v_model_config_image_embeds,
        content_format="string",
    )

# ... 8 lines omitted for brevity ...
    # Verify mm_data contains an empty dictionary of embeddings
    assert mm_data is not None
    assert "image" in mm_data
    assert isinstance(mm_data["image"], dict)
    assert len(mm_data["image"]) == 0

    # Verify UUIDs (None since we didn't provide any)
    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[None])
```
**EN:** This test validates `test_parse_chat_messages_empty_dict_image_embeds`. Key inputs are `phi3v_model_config_image_embeds`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in this image?"}]` and `mm_data is not None`.
**CN:** 这个测试验证 `test_parse_chat_messages_empty_dict_image_embeds`。 关键输入包括 `phi3v_model_config_image_embeds`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in this image?"}]` and `mm_data is not None`。

### Test / 测试: test_parse_chat_messages_multiple_dict_image_embeds (L1190-L1247)
```python
def test_parse_chat_messages_multiple_dict_image_embeds(
    qwen25omni_model_config_image_embeds,
):
    """Test that multiple dictionaries for image_embeds is handled without errors."""
    # Create two sample image embedding tensors
    batch_size = 2
    hidden_size = qwen25omni_model_config_image_embeds.get_inputs_embeds_size()
    image_embeds = torch.randn(batch_size * 220, hidden_size)
    image_grid_thw = torch.tensor([[1, 22, 40] for _ in range(batch_size)])

    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_embeds",
                        "image_embeds": {
# ... 32 lines omitted for brevity ...
    # Verify each embedding has the correct shape
    assert isinstance(mm_data["image"]["image_embeds"], torch.Tensor)
    assert mm_data["image"]["image_embeds"].shape == image_embeds.shape
    assert isinstance(mm_data["image"]["image_grid_thw"], torch.Tensor)
    assert mm_data["image"]["image_grid_thw"].shape == image_grid_thw.shape

    # Verify UUIDs (None since we didn't provide any)
    _assert_mm_uuids(mm_uuids, batch_size, expected_uuids=[None, None])
```
**EN:** This test validates `test_parse_chat_messages_multiple_dict_image_embeds`. Key inputs are `qwen25omni_model_config_image_embeds`. The main assertion is `conversation == [{'role': 'user', 'content': '<|vision_start|><|IMAGE|><|vision_end|>\n<|vision_start|><|IMAGE|><|vision_end|>\nDescribe ...` and `mm_data is not None`.
**CN:** 这个测试验证 `test_parse_chat_messages_multiple_dict_image_embeds`。 关键输入包括 `qwen25omni_model_config_image_embeds`。 核心断言是 `conversation == [{'role': 'user', 'content': '<|vision_start|><|IMAGE|><|vision_end|>\n<|vision_start|><|IMAGE|><|vision_end|>\nDescribe ...` and `mm_data is not None`。

### Test / 测试: test_parse_chat_messages_multiple_images_async (L1250-L1280)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_multiple_images_async(
    phi3v_model_config,
    image_url,
):
    conversation, mm_data, mm_uuids = await parse_chat_messages_async(
        [
            {
                "role": "user",
                "content": [
                    {"type": "image_url", "image_url": {"url": image_url}},
                    {
                        "type": "image_pil",
                        "image_pil": ImageAsset("cherry_blossom").pil_image,
                    },
                    {"type": "text", "text": "What's in these images?"},
                ],
            }
# ... 5 lines omitted for brevity ...
    assert conversation == [
        {
            "role": "user",
            "content": "<|image_1|>\n<|image_2|>\nWhat's in these images?",
        }
    ]
    _assert_mm_data_is_image_input(mm_data, 2)
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[None, None])
```
**EN:** This async test validates `test_parse_chat_messages_multiple_images_async`. Relevant pytest markers include `asyncio`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\n<|image_2|>\nWhat's in these images?"}]`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_multiple_images_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\n<|image_2|>\nWhat's in these images?"}]`。

### Test / 测试: test_parse_chat_messages_placeholder_already_in_prompt (L1283-L1311)
```python
def test_parse_chat_messages_placeholder_already_in_prompt(
    phi3v_model_config,
    image_url,
):
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {"type": "image_url", "image_url": {"url": image_url}},
                    {"type": "image_url", "image_url": {"url": image_url}},
                    {
                        "type": "text",
                        "text": "What's in <|image_1|> and how does it compare to <|image_2|>?",  # noqa: E501
                    },
                ],
            }
        ],
        phi3v_model_config,
        content_format="string",
    )
    assert conversation == [
        {
            "role": "user",
            "content": "What's in <|image_1|> and how does it compare to <|image_2|>?",
        }
    ]
    _assert_mm_data_is_image_input(mm_data, 2)
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[None, None])
```
**EN:** This test validates `test_parse_chat_messages_placeholder_already_in_prompt`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "What's in <|image_1|> and how does it compare to <|image_2|>?"}]`.
**CN:** 这个测试验证 `test_parse_chat_messages_placeholder_already_in_prompt`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "What's in <|image_1|> and how does it compare to <|image_2|>?"}]`。

### Test / 测试: test_parse_chat_messages_placeholder_one_already_in_prompt (L1314-L1345)
```python
def test_parse_chat_messages_placeholder_one_already_in_prompt(
    phi3v_model_config,
    image_url,
):
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {"type": "image_url", "image_url": {"url": image_url}},
                    {"type": "image_url", "image_url": {"url": image_url}},
                    {
                        "type": "text",
                        "text": "What's in <|image_1|> and how does it compare to "
                        "the other one?",
                    },
                ],
            }
# ... 6 lines omitted for brevity ...
        {
            "role": "user",
            "content": "<|image_2|>\nWhat's in <|image_1|> and how does it compare to "
            "the other one?",
        }
    ]
    _assert_mm_data_is_image_input(mm_data, 2)
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[None, None])
```
**EN:** This test validates `test_parse_chat_messages_placeholder_one_already_in_prompt`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_2|>\nWhat's in <|image_1|> and how does it compare to the other one?"}]`.
**CN:** 这个测试验证 `test_parse_chat_messages_placeholder_one_already_in_prompt`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_2|>\nWhat's in <|image_1|> and how does it compare to the other one?"}]`。

### Test / 测试: test_parse_chat_messages_multiple_images_across_messages (L1348-L1380)
```python
def test_parse_chat_messages_multiple_images_across_messages(
    phi3v_model_config,
    image_url,
):
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {"type": "image_url", "image_url": {"url": image_url}},
                    {"type": "text", "text": "What's in this image?"},
                ],
            },
            {"role": "assistant", "content": "Some stuff."},
            {
                "role": "user",
                "content": [
                    {"type": "image_url", "image_url": {"url": image_url}},
# ... 7 lines omitted for brevity ...

    assert conversation == [
        {"role": "user", "content": "<|image_1|>\nWhat's in this image?"},
        {"role": "assistant", "content": "Some stuff."},
        {"role": "user", "content": "<|image_2|>\nWhat about this one?"},
    ]
    _assert_mm_data_is_image_input(mm_data, 2)
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[None, None])
```
**EN:** This test validates `test_parse_chat_messages_multiple_images_across_messages`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in this image?"}, {'role': 'assistant', 'content': 'Some stuff.'}, {'ro...`.
**CN:** 这个测试验证 `test_parse_chat_messages_multiple_images_across_messages`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in this image?"}, {'role': 'assistant', 'content': 'Some stuff.'}, {'ro...`。

### Test / 测试: test_parse_chat_messages_multiple_images_with_uuids_across_messages (L1383-L1424)
```python
def test_parse_chat_messages_multiple_images_with_uuids_across_messages(
    phi3v_model_config,
    image_url,
):
    image_uuid = str(hash(image_url))
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": {"url": image_url},
                        "uuid": image_uuid,
                    },
                    {"type": "text", "text": "What's in this image?"},
                ],
            },
# ... 16 lines omitted for brevity ...

    assert conversation == [
        {"role": "user", "content": "<|image_1|>\nWhat's in this image?"},
        {"role": "assistant", "content": "Some stuff."},
        {"role": "user", "content": "<|image_2|>\nWhat about this one?"},
    ]
    _assert_mm_data_is_image_input(mm_data, 2)
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[image_uuid, image_uuid])
```
**EN:** This test validates `test_parse_chat_messages_multiple_images_with_uuids_across_messages`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in this image?"}, {'role': 'assistant', 'content': 'Some stuff.'}, {'ro...`.
**CN:** 这个测试验证 `test_parse_chat_messages_multiple_images_with_uuids_across_messages`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\nWhat's in this image?"}, {'role': 'assistant', 'content': 'Some stuff.'}, {'ro...`。

### Test / 测试: test_parse_chat_messages_context_text_format (L1427-L1458)
```python
def test_parse_chat_messages_context_text_format(
    phi3v_model_config,
):
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [{"type": "text", "text": "What's in this text?"}],
            },
            {"role": "assistant", "content": "Some stuff."},
            {"role": "user", "content": "What about this one?"},
        ],
        phi3v_model_config,
        content_format="openai",
    )

    assert conversation == [
        {
# ... 6 lines omitted for brevity ...
        },
        {
            "role": "user",
            "content": [{"type": "text", "text": "What about this one?"}],
        },
    ]
    assert mm_data is None
    assert mm_uuids is None
```
**EN:** This test validates `test_parse_chat_messages_context_text_format`. Key inputs are `phi3v_model_config`. The main assertion is `conversation == [{'role': 'user', 'content': [{'type': 'text', 'text': "What's in this text?"}]}, {'role': 'assistant', 'content': [{'typ...` and `mm_data is None`.
**CN:** 这个测试验证 `test_parse_chat_messages_context_text_format`。 关键输入包括 `phi3v_model_config`。 核心断言是 `conversation == [{'role': 'user', 'content': [{'type': 'text', 'text': "What's in this text?"}]}, {'role': 'assistant', 'content': [{'typ...` and `mm_data is None`。

### Test / 测试: test_parse_chat_messages_openai_format_image_url (L1461-L1490)
```python
def test_parse_chat_messages_openai_format_image_url(
    phi3v_model_config,
    image_url,
):
    content = [
        {"type": "image_url", "image_url": {"url": image_url}},
        {"type": "text", "text": "What's in the image?"},
    ]
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": content,
            }
        ],
        phi3v_model_config,
        content_format="openai",
    )

    assert conversation == [
        {
            "role": "user",
            "content": [
                {"type": "image"},
                {"type": "text", "text": "What's in the image?"},
            ],
        }
    ]
    _assert_mm_data_is_image_input(mm_data, 1)
    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[None])
```
**EN:** This test validates `test_parse_chat_messages_openai_format_image_url`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': [{'type': 'image'}, {'type': 'text', 'text': "What's in the image?"}]}]`.
**CN:** 这个测试验证 `test_parse_chat_messages_openai_format_image_url`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': [{'type': 'image'}, {'type': 'text', 'text': "What's in the image?"}]}]`。

### Test / 测试: test_parse_chat_messages_rejects_too_many_images_in_one_message (L1493-L1526)
```python
def test_parse_chat_messages_rejects_too_many_images_in_one_message(
    phi3v_model_config,
    image_url,
):
    with warnings.catch_warnings():
        warnings.filterwarnings(
            "ignore",
            message="coroutine 'async_get_and_parse_image' was never awaited",
        )
        with pytest.raises(ValueError, match="At most"):
            parse_chat_messages(
                [
                    {
                        "role": "user",
                        "content": [
                            {
                                "type": "image_url",
                                "image_url": {"url": image_url},
# ... 8 lines omitted for brevity ...
                            },
                            {"type": "text", "text": "What's in these images?"},
                        ],
                    }
                ],
                phi3v_model_config,
                content_format="string",
            )
```
**EN:** This test validates `test_parse_chat_messages_rejects_too_many_images_in_one_message`. Key inputs are `phi3v_model_config`, `image_url`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `test_parse_chat_messages_rejects_too_many_images_in_one_message`。 关键输入包括 `phi3v_model_config`、`image_url`。 它使用 `pytest.raises` 检查预期失败路径。

### Test / 测试: test_parse_chat_messages_rejects_too_many_images_across_messages (L1529-L1569)
```python
def test_parse_chat_messages_rejects_too_many_images_across_messages(
    phi3v_model_config,
    image_url,
):
    with warnings.catch_warnings():
        warnings.filterwarnings(
            "ignore",
            message="coroutine 'async_get_and_parse_image' was never awaited",
        )
        with pytest.raises(ValueError, match="At most"):
            parse_chat_messages(
                [
                    {
                        "role": "user",
                        "content": [
                            {
                                "type": "image_url",
                                "image_url": {"url": image_url},
# ... 15 lines omitted for brevity ...
                            },
                            {"type": "text", "text": "What about these two?"},
                        ],
                    },
                ],
                phi3v_model_config,
                content_format="string",
            )
```
**EN:** This test validates `test_parse_chat_messages_rejects_too_many_images_across_messages`. Key inputs are `phi3v_model_config`, `image_url`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `test_parse_chat_messages_rejects_too_many_images_across_messages`。 关键输入包括 `phi3v_model_config`、`image_url`。 它使用 `pytest.raises` 检查预期失败路径。

### Test / 测试: test_parse_chat_messages_multiple_images_uncommon_input (L1572-L1598)
```python
def test_parse_chat_messages_multiple_images_uncommon_input(
    phi3v_model_config,
    image_url,
):
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    "What's in these images?",
                    {"image_url": image_url},
                    {"image_url": image_url},
                ],
            }
        ],
        phi3v_model_config,
        content_format="string",
    )

    assert conversation == [
        {
            "role": "user",
            "content": "<|image_1|>\n<|image_2|>\nWhat's in these images?",
        }
    ]
    _assert_mm_data_is_image_input(mm_data, 2)
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[None, None])
```
**EN:** This test validates `test_parse_chat_messages_multiple_images_uncommon_input`. Key inputs are `phi3v_model_config`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "<|image_1|>\n<|image_2|>\nWhat's in these images?"}]`.
**CN:** 这个测试验证 `test_parse_chat_messages_multiple_images_uncommon_input`。 关键输入包括 `phi3v_model_config`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "<|image_1|>\n<|image_2|>\nWhat's in these images?"}]`。

### Test / 测试: test_parse_chat_messages_multiple_images_interleave (L1601-L1633)
```python
def test_parse_chat_messages_multiple_images_interleave(
    phi3v_model_config_mm_interleaved,
    image_url,
):
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "text",
                        "text": "I need you to compare this image",
                    },
                    {"type": "image_url", "image_url": {"url": image_url}},
                    {"type": "text", "text": "and this one"},
                    {"type": "image_url", "image_url": {"url": image_url}},
                    {"type": "text", "text": "Do they have differences?"},
                ],
# ... 7 lines omitted for brevity ...
        {
            "role": "user",
            "content": "I need you to compare this image\n<|image_1|>\nand this one\n<|image_2|>\n"  # noqa: E501
            "Do they have differences?",
        }
    ]
    _assert_mm_data_is_image_input(mm_data, 2)
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[None, None])
```
**EN:** This test validates `test_parse_chat_messages_multiple_images_interleave`. Key inputs are `phi3v_model_config_mm_interleaved`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': 'I need you to compare this image\n<|image_1|>\nand this one\n<|image_2|>\nDo they have diff...`.
**CN:** 这个测试验证 `test_parse_chat_messages_multiple_images_interleave`。 关键输入包括 `phi3v_model_config_mm_interleaved`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': 'I need you to compare this image\n<|image_1|>\nand this one\n<|image_2|>\nDo they have diff...`。

### Test / 测试: test_parse_chat_messages_multiple_images_interleave_async (L1636-L1669)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_multiple_images_interleave_async(
    phi3v_model_config_mm_interleaved,
    image_url,
):
    conversation, mm_data, mm_uuids = await parse_chat_messages_async(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "text",
                        "text": "I need you to compare this image",
                    },
                    {"type": "image_url", "image_url": {"url": image_url}},
                    {"type": "text", "text": "and this one"},
                    {"type": "image_url", "image_url": {"url": image_url}},
                    {"type": "text", "text": "Do they have differences?"},
# ... 8 lines omitted for brevity ...
        {
            "role": "user",
            "content": "I need you to compare this image\n<|image_1|>\nand this one\n<|image_2|>\n"  # noqa: E501
            "Do they have differences?",
        }
    ]
    _assert_mm_data_is_image_input(mm_data, 2)
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[None, None])
```
**EN:** This async test validates `test_parse_chat_messages_multiple_images_interleave_async`. Relevant pytest markers include `asyncio`. Key inputs are `phi3v_model_config_mm_interleaved`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': 'I need you to compare this image\n<|image_1|>\nand this one\n<|image_2|>\nDo they have diff...`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_multiple_images_interleave_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `phi3v_model_config_mm_interleaved`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': 'I need you to compare this image\n<|image_1|>\nand this one\n<|image_2|>\nDo they have diff...`。

### Test / 测试: test_parse_chat_messages_multiple_images_with_uuids_interleave_async (L1672-L1714)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_multiple_images_with_uuids_interleave_async(
    phi3v_model_config_mm_interleaved,
    image_url,
):
    image_uuid = str(hash(image_url))
    conversation, mm_data, mm_uuids = await parse_chat_messages_async(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "text",
                        "text": "I need you to compare this image",
                    },
                    {
                        "type": "image_url",
                        "image_url": {"url": image_url},
# ... 17 lines omitted for brevity ...
        {
            "role": "user",
            "content": "I need you to compare this image\n<|image_1|>\nand this one\n<|image_2|>\n"  # noqa: E501
            "Do they have differences?",
        }
    ]
    _assert_mm_data_is_image_input(mm_data, 2)
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[image_uuid, image_uuid])
```
**EN:** This async test validates `test_parse_chat_messages_multiple_images_with_uuids_interleave_async`. Relevant pytest markers include `asyncio`. Key inputs are `phi3v_model_config_mm_interleaved`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': 'I need you to compare this image\n<|image_1|>\nand this one\n<|image_2|>\nDo they have diff...`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_multiple_images_with_uuids_interleave_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `phi3v_model_config_mm_interleaved`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': 'I need you to compare this image\n<|image_1|>\nand this one\n<|image_2|>\nDo they have diff...`。

### Test / 测试: test_parse_chat_messages_multiple_images_multiple_messages_interleave (L1717-L1753)
```python
def test_parse_chat_messages_multiple_images_multiple_messages_interleave(
    phi3v_model_config_mm_interleaved,
    image_url,
):
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": "What's on this image?"},
                    {"type": "image_url", "image_url": {"url": image_url}},
                    {"type": "text", "text": "Be accurate."},
                ],
            },
            {"role": "assistant", "content": "Some stuff."},
            {
                "role": "user",
                "content": [
# ... 11 lines omitted for brevity ...
            "role": "user",
            "content": "What's on this image?\n<|image_1|>\nBe accurate.",
        },
        {"role": "assistant", "content": "Some stuff."},
        {"role": "user", "content": "What's on this image?\n<|image_2|>"},
    ]
    _assert_mm_data_is_image_input(mm_data, 2)
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[None, None])
```
**EN:** This test validates `test_parse_chat_messages_multiple_images_multiple_messages_interleave`. Key inputs are `phi3v_model_config_mm_interleaved`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "What's on this image?\n<|image_1|>\nBe accurate."}, {'role': 'assistant', 'content': 'Some ...`.
**CN:** 这个测试验证 `test_parse_chat_messages_multiple_images_multiple_messages_interleave`。 关键输入包括 `phi3v_model_config_mm_interleaved`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "What's on this image?\n<|image_1|>\nBe accurate."}, {'role': 'assistant', 'content': 'Some ...`。

### Test / 测试: test_parse_chat_messages_multiple_images_with_uuids_multiple_messages_interleave (L1756-L1801)
```python
def test_parse_chat_messages_multiple_images_with_uuids_multiple_messages_interleave(
    phi3v_model_config_mm_interleaved,
    image_url,
):
    image_uuid = str(hash(image_url))
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": "What's on this image?"},
                    {
                        "type": "image_url",
                        "image_url": {"url": image_url},
                        "uuid": image_uuid,
                    },
                    {"type": "text", "text": "Be accurate."},
                ],
# ... 20 lines omitted for brevity ...
            "role": "user",
            "content": "What's on this image?\n<|image_1|>\nBe accurate.",
        },
        {"role": "assistant", "content": "Some stuff."},
        {"role": "user", "content": "What's on this image?\n<|image_2|>"},
    ]
    _assert_mm_data_is_image_input(mm_data, 2)
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[image_uuid, image_uuid])
```
**EN:** This test validates `test_parse_chat_messages_multiple_images_with_uuids_multiple_messages_interleave`. Key inputs are `phi3v_model_config_mm_interleaved`, `image_url`. The main assertion is `conversation == [{'role': 'user', 'content': "What's on this image?\n<|image_1|>\nBe accurate."}, {'role': 'assistant', 'content': 'Some ...`.
**CN:** 这个测试验证 `test_parse_chat_messages_multiple_images_with_uuids_multiple_messages_interleave`。 关键输入包括 `phi3v_model_config_mm_interleaved`、`image_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "What's on this image?\n<|image_1|>\nBe accurate."}, {'role': 'assistant', 'content': 'Some ...`。

### Test / 测试: test_parse_chat_messages_multiple_modals_multiple_messages_interleave (L1804-L1853)
```python
def test_parse_chat_messages_multiple_modals_multiple_messages_interleave(
    qwen25omni_model_config_mm_interleaved,
    image_url,
    video_url,
    audio_url,
):
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": "What's on this image?"},
                    {"type": "image_url", "image_url": {"url": image_url}},
                    {"type": "text", "text": "Now listen to this audio"},
                    {"type": "audio_url", "audio_url": {"url": audio_url}},
                ],
            },
            {"role": "assistant", "content": "Some stuff."},
# ... 24 lines omitted for brevity ...
            "\nAnd what's in the video?\n<|vision_start|><|VIDEO|><|vision_end|>",
        },
    ]

    _assert_mm_data_inputs(mm_data, {"image": 2, "video": 1, "audio": 1})
    _assert_mm_uuids(mm_uuids, 2, modality="image", expected_uuids=[None, None])
    _assert_mm_uuids(mm_uuids, 1, modality="video", expected_uuids=[None])
    _assert_mm_uuids(mm_uuids, 1, modality="audio", expected_uuids=[None])
```
**EN:** This test validates `test_parse_chat_messages_multiple_modals_multiple_messages_interleave`. Key inputs are `qwen25omni_model_config_mm_interleaved`, `image_url`, `video_url`, `audio_url`. The main assertion is `conversation == [{'role': 'user', 'content': "What's on this image?\n<|vision_start|><|IMAGE|><|vision_end|>\nNow listen to this audio\nA...`.
**CN:** 这个测试验证 `test_parse_chat_messages_multiple_modals_multiple_messages_interleave`。 关键输入包括 `qwen25omni_model_config_mm_interleaved`、`image_url`、`video_url`、`audio_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "What's on this image?\n<|vision_start|><|IMAGE|><|vision_end|>\nNow listen to this audio\nA...`。

### Test / 测试: test_parse_chat_messages_multiple_modals_with_uuids_multiple_messages_interleave (L1856-L1923)
```python
def test_parse_chat_messages_multiple_modals_with_uuids_multiple_messages_interleave(
    qwen25omni_model_config_mm_interleaved,
    image_url,
    video_url,
    audio_url,
):
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": "What's on this image?"},
                    {
                        "type": "image_url",
                        "image_url": {"url": image_url},
                        "uuid": "image_123",
                    },
                    {"type": "text", "text": "Now listen to this audio"},
# ... 42 lines omitted for brevity ...
    ]

    _assert_mm_data_inputs(mm_data, {"image": 2, "video": 1, "audio": 1})
    _assert_mm_uuids(
        mm_uuids, 2, modality="image", expected_uuids=["image_123", "image_123"]
    )
    _assert_mm_uuids(mm_uuids, 1, modality="video", expected_uuids=["video_123"])
    _assert_mm_uuids(mm_uuids, 1, modality="audio", expected_uuids=["audio_123"])
```
**EN:** This test validates `test_parse_chat_messages_multiple_modals_with_uuids_multiple_messages_interleave`. Key inputs are `qwen25omni_model_config_mm_interleaved`, `image_url`, `video_url`, `audio_url`. The main assertion is `conversation == [{'role': 'user', 'content': "What's on this image?\n<|vision_start|><|IMAGE|><|vision_end|>\nNow listen to this audio\nA...`.
**CN:** 这个测试验证 `test_parse_chat_messages_multiple_modals_with_uuids_multiple_messages_interleave`。 关键输入包括 `qwen25omni_model_config_mm_interleaved`、`image_url`、`video_url`、`audio_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "What's on this image?\n<|vision_start|><|IMAGE|><|vision_end|>\nNow listen to this audio\nA...`。

### Test / 测试: test_parse_chat_messages_multiple_modals_with_uuids_multiple_empty_media_messages_interleave (L1926-L1997)
```python
def test_parse_chat_messages_multiple_modals_with_uuids_multiple_empty_media_messages_interleave(  # noqa: E501
    qwen25omni_model_config_mm_interleaved,
    image_url,
    video_url,
    audio_url,
):
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": "What's on this image?"},
                    {
                        "type": "image_url",
                        "image_url": None,
                        "uuid": "image_123",
                    },
                    {"type": "text", "text": "Now listen to this audio"},
# ... 46 lines omitted for brevity ...
        {"image": 2, "video": 1, "audio": 1},
        skipped_media_indices={"image": [0, 1], "video": [0], "audio": [0]},
    )
    _assert_mm_uuids(
        mm_uuids, 2, modality="image", expected_uuids=["image_123", "image_123"]
    )
    _assert_mm_uuids(mm_uuids, 1, modality="video", expected_uuids=["video_123"])
    _assert_mm_uuids(mm_uuids, 1, modality="audio", expected_uuids=["audio_123"])
```
**EN:** This test validates `test_parse_chat_messages_multiple_modals_with_uuids_multiple_empty_media_messages_interleave`. Key inputs are `qwen25omni_model_config_mm_interleaved`, `image_url`, `video_url`, `audio_url`. The main assertion is `conversation == [{'role': 'user', 'content': "What's on this image?\n<|vision_start|><|IMAGE|><|vision_end|>\nNow listen to this audio\nA...`.
**CN:** 这个测试验证 `test_parse_chat_messages_multiple_modals_with_uuids_multiple_empty_media_messages_interleave`。 关键输入包括 `qwen25omni_model_config_mm_interleaved`、`image_url`、`video_url`、`audio_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "What's on this image?\n<|vision_start|><|IMAGE|><|vision_end|>\nNow listen to this audio\nA...`。

### Test / 测试: test_parse_chat_messages_multiple_modals_with_partial_uuids_multiple_messages_interleave (L2000-L2057)
```python
def test_parse_chat_messages_multiple_modals_with_partial_uuids_multiple_messages_interleave(  # noqa: E501
    qwen25omni_model_config_mm_interleaved,
    image_url,
    video_url,
    audio_url,
):
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": "What's on this image?"},
                    {
                        "type": "image_url",
                        "image_url": {"url": image_url},
                        "uuid": "image_123",
                    },
                    {"type": "text", "text": "Now listen to this audio"},
# ... 32 lines omitted for brevity ...
            "\nAnd what's in the video?\n<|vision_start|><|VIDEO|><|vision_end|>",
        },
    ]

    _assert_mm_data_inputs(mm_data, {"image": 2, "video": 1, "audio": 1})
    _assert_mm_uuids(mm_uuids, 2, modality="image", expected_uuids=["image_123", None])
    _assert_mm_uuids(mm_uuids, 1, modality="video", expected_uuids=["video_123"])
    _assert_mm_uuids(mm_uuids, 1, modality="audio", expected_uuids=[None])
```
**EN:** This test validates `test_parse_chat_messages_multiple_modals_with_partial_uuids_multiple_messages_interleave`. Key inputs are `qwen25omni_model_config_mm_interleaved`, `image_url`, `video_url`, `audio_url`. The main assertion is `conversation == [{'role': 'user', 'content': "What's on this image?\n<|vision_start|><|IMAGE|><|vision_end|>\nNow listen to this audio\nA...`.
**CN:** 这个测试验证 `test_parse_chat_messages_multiple_modals_with_partial_uuids_multiple_messages_interleave`。 关键输入包括 `qwen25omni_model_config_mm_interleaved`、`image_url`、`video_url`、`audio_url`。 核心断言是 `conversation == [{'role': 'user', 'content': "What's on this image?\n<|vision_start|><|IMAGE|><|vision_end|>\nNow listen to this audio\nA...`。

### Test / 测试: test_parse_chat_messages_multiple_images_interleave_with_placeholders (L2060-L2086)
```python
def test_parse_chat_messages_multiple_images_interleave_with_placeholders(
    phi3v_model_config_mm_interleaved,
    image_url,
):
    with pytest.raises(
        ValueError,
        match=r"Found more '<|image_1|>' placeholders in input prompt "
        "than actual multimodal data items.",
    ):
        parse_chat_messages(
            [
                {
                    "role": "user",
                    "content": [
                        {"type": "image_url", "image_url": {"url": image_url}},
                        {"type": "image_url", "image_url": {"url": image_url}},
                        {
                            "type": "text",
                            "text": "I need you to compare this image\n<|image_1|>\nand this one\n<|image_2|>\n"  # noqa: E501
                            "Do they have differences?",
                        },
                    ],
                }
            ],
            phi3v_model_config_mm_interleaved,
            content_format="string",
        )
```
**EN:** This test validates `test_parse_chat_messages_multiple_images_interleave_with_placeholders`. Key inputs are `phi3v_model_config_mm_interleaved`, `image_url`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `test_parse_chat_messages_multiple_images_interleave_with_placeholders`。 关键输入包括 `phi3v_model_config_mm_interleaved`、`image_url`。 它使用 `pytest.raises` 检查预期失败路径。

### Test / 测试: test_parse_chat_messages_include_thinking_chunk (L2089-L2147)
```python
def test_parse_chat_messages_include_thinking_chunk(mistral_model_config):
    messages = [
        {
            "role": "system",
            "content": [
                {"type": "text", "text": "You are a helpful assistant."},
                {
                    "type": "thinking",
                    "closed": True,
                    "thinking": "Only return the answer when you are confident.",
                },
            ],
        },
        {"role": "user", "content": "What is 2+2?"},
        {
            "role": "assistant",
            "content": [
                {"type": "text", "text": "Let me think about it."},
# ... 33 lines omitted for brevity ...
                {"type": "text", "text": "Let me think about it."},
                {"type": "text", "text": "2+2 = 4"},
                {"type": "text", "text": "The answer is 4."},
            ],
        },
    ]

    assert conversation_with_thinking == expected_conversation
```
**EN:** This test validates `test_parse_chat_messages_include_thinking_chunk`. Key inputs are `mistral_model_config`. The main assertion is `conversation_with_thinking == expected_conversation`.
**CN:** 这个测试验证 `test_parse_chat_messages_include_thinking_chunk`。 关键输入包括 `mistral_model_config`。 核心断言是 `conversation_with_thinking == expected_conversation`。

### Test / 测试: test_parse_chat_messages_single_empty_audio_with_uuid (L2150-L2180)
```python
def test_parse_chat_messages_single_empty_audio_with_uuid(
    qwen2_audio_model_config,
):
    audio_uuid = "abcd"
    conversation, mm_data, mm_uuids = parse_chat_messages(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "input_audio",
                        "input_audio": {},
                        "uuid": audio_uuid,
                    },
                    {"type": "text", "text": "What does the audio say?"},
                ],
            }
        ],
# ... 5 lines omitted for brevity ...
        {
            "role": "user",
            "content": "Audio 1: <|audio_bos|><|AUDIO|><|audio_eos|>\nWhat does the "
            "audio say?",
        }
    ]
    _assert_mm_data_inputs(mm_data, {"audio": 1})
    _assert_mm_uuids(mm_uuids, 1, modality="audio", expected_uuids=[audio_uuid])
```
**EN:** This test validates `test_parse_chat_messages_single_empty_audio_with_uuid`. Key inputs are `qwen2_audio_model_config`. The main assertion is `conversation == [{'role': 'user', 'content': 'Audio 1: <|audio_bos|><|AUDIO|><|audio_eos|>\nWhat does the audio say?'}]`.
**CN:** 这个测试验证 `test_parse_chat_messages_single_empty_audio_with_uuid`。 关键输入包括 `qwen2_audio_model_config`。 核心断言是 `conversation == [{'role': 'user', 'content': 'Audio 1: <|audio_bos|><|AUDIO|><|audio_eos|>\nWhat does the audio say?'}]`。

### Test / 测试: test_parse_chat_messages_single_empty_audio_with_uuid_async (L2183-L2214)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_single_empty_audio_with_uuid_async(
    qwen2_audio_model_config,
):
    audio_uuid = "abcd"
    conversation, mm_data, mm_uuids = await parse_chat_messages_async(
        [
            {
                "role": "user",
                "content": [
                    {
                        "type": "input_audio",
                        "input_audio": {},
                        "uuid": audio_uuid,
                    },
                    {"type": "text", "text": "What does the audio say?"},
                ],
            }
# ... 6 lines omitted for brevity ...
        {
            "role": "user",
            "content": "Audio 1: <|audio_bos|><|AUDIO|><|audio_eos|>\nWhat does the "
            "audio say?",
        }
    ]
    _assert_mm_data_inputs(mm_data, {"audio": 1})
    _assert_mm_uuids(mm_uuids, 1, modality="audio", expected_uuids=[audio_uuid])
```
**EN:** This async test validates `test_parse_chat_messages_single_empty_audio_with_uuid_async`. Relevant pytest markers include `asyncio`. Key inputs are `qwen2_audio_model_config`. The main assertion is `conversation == [{'role': 'user', 'content': 'Audio 1: <|audio_bos|><|AUDIO|><|audio_eos|>\nWhat does the audio say?'}]`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_single_empty_audio_with_uuid_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `qwen2_audio_model_config`。 核心断言是 `conversation == [{'role': 'user', 'content': 'Audio 1: <|audio_bos|><|AUDIO|><|audio_eos|>\nWhat does the audio say?'}]`。

### Test / 测试: test_parse_chat_messages_image_vision_chunk (L2217-L2250)
```python
def test_parse_chat_messages_image_vision_chunk(
    kimi_k2_5_model_config,
    image_url,
):
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Analyze this image."},
                {
                    "type": "image_url",
                    "image_url": {"url": image_url},
                },
            ],
        }
    ]

    conversation, mm_data, mm_uuids = parse_chat_messages(
# ... 8 lines omitted for brevity ...
            "role": "user",
            "content": f"{placeholder}\nAnalyze this image.",
        }
    ]

    assert conversation == expected_conversation
    _assert_mm_data_is_vision_chunk_input(mm_data, 1)
    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[None], modality="vision_chunk")
```
**EN:** This test validates `test_parse_chat_messages_image_vision_chunk`. Key inputs are `kimi_k2_5_model_config`, `image_url`. The main assertion is `conversation == expected_conversation`.
**CN:** 这个测试验证 `test_parse_chat_messages_image_vision_chunk`。 关键输入包括 `kimi_k2_5_model_config`、`image_url`。 核心断言是 `conversation == expected_conversation`。

### Test / 测试: test_parse_chat_messages_video_vision_chunk (L2253-L2286)
```python
def test_parse_chat_messages_video_vision_chunk(
    kimi_k2_5_model_config,
    video_url,
):
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Analyze this video."},
                {
                    "type": "video_url",
                    "video_url": {"url": video_url},
                },
            ],
        }
    ]

    conversation, mm_data, mm_uuids = parse_chat_messages(
# ... 8 lines omitted for brevity ...
            "role": "user",
            "content": f"{placeholder}\nAnalyze this video.",
        }
    ]

    assert conversation == expected_conversation
    _assert_mm_data_is_vision_chunk_input(mm_data, 1)
    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[None], modality="vision_chunk")
```
**EN:** This test validates `test_parse_chat_messages_video_vision_chunk`. Key inputs are `kimi_k2_5_model_config`, `video_url`. The main assertion is `conversation == expected_conversation`.
**CN:** 这个测试验证 `test_parse_chat_messages_video_vision_chunk`。 关键输入包括 `kimi_k2_5_model_config`、`video_url`。 核心断言是 `conversation == expected_conversation`。

### Test / 测试: test_parse_chat_messages_image_vision_chunk_with_uuid (L2289-L2324)
```python
def test_parse_chat_messages_image_vision_chunk_with_uuid(
    kimi_k2_5_model_config,
    image_url,
):
    image_uuid = "image_123"
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Analyze this image."},
                {
                    "type": "image_url",
                    "image_url": {"url": image_url},
                    "uuid": image_uuid,
                },
            ],
        }
    ]
# ... 10 lines omitted for brevity ...
            "role": "user",
            "content": f"{placeholder}\nAnalyze this image.",
        }
    ]

    assert conversation == expected_conversation
    _assert_mm_data_is_vision_chunk_input(mm_data, 1)
    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[image_uuid], modality="vision_chunk")
```
**EN:** This test validates `test_parse_chat_messages_image_vision_chunk_with_uuid`. Key inputs are `kimi_k2_5_model_config`, `image_url`. The main assertion is `conversation == expected_conversation`.
**CN:** 这个测试验证 `test_parse_chat_messages_image_vision_chunk_with_uuid`。 关键输入包括 `kimi_k2_5_model_config`、`image_url`。 核心断言是 `conversation == expected_conversation`。

### Test / 测试: test_parse_chat_messages_video_vision_chunk_with_uuid (L2327-L2362)
```python
def test_parse_chat_messages_video_vision_chunk_with_uuid(
    kimi_k2_5_model_config,
    video_url,
):
    video_uuid = "video_456"
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Analyze this video."},
                {
                    "type": "video_url",
                    "video_url": {"url": video_url},
                    "uuid": video_uuid,
                },
            ],
        }
    ]
# ... 10 lines omitted for brevity ...
            "role": "user",
            "content": f"{placeholder}\nAnalyze this video.",
        }
    ]

    assert conversation == expected_conversation
    _assert_mm_data_is_vision_chunk_input(mm_data, 1)
    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[video_uuid], modality="vision_chunk")
```
**EN:** This test validates `test_parse_chat_messages_video_vision_chunk_with_uuid`. Key inputs are `kimi_k2_5_model_config`, `video_url`. The main assertion is `conversation == expected_conversation`.
**CN:** 这个测试验证 `test_parse_chat_messages_video_vision_chunk_with_uuid`。 关键输入包括 `kimi_k2_5_model_config`、`video_url`。 核心断言是 `conversation == expected_conversation`。

### Test / 测试: test_parse_chat_messages_mixed_vision_chunk (L2365-L2409)
```python
def test_parse_chat_messages_mixed_vision_chunk(
    kimi_k2_5_model_config,
    image_url,
    video_url,
):
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Analyze this image and video."},
                {
                    "type": "image_url",
                    "image_url": {"url": image_url},
                },
                {
                    "type": "video_url",
                    "video_url": {"url": video_url},
                },
# ... 19 lines omitted for brevity ...
                "Analyze this image and video."
            ),
        }
    ]

    assert conversation == expected_conversation
    _assert_mm_data_is_vision_chunk_input(mm_data, 2)
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[None, None], modality="vision_chunk")
```
**EN:** This test validates `test_parse_chat_messages_mixed_vision_chunk`. Key inputs are `kimi_k2_5_model_config`, `image_url`, `video_url`. The main assertion is `conversation == expected_conversation`.
**CN:** 这个测试验证 `test_parse_chat_messages_mixed_vision_chunk`。 关键输入包括 `kimi_k2_5_model_config`、`image_url`、`video_url`。 核心断言是 `conversation == expected_conversation`。

### Test / 测试: test_parse_chat_messages_mixed_vision_chunk_with_uuid (L2412-L2462)
```python
def test_parse_chat_messages_mixed_vision_chunk_with_uuid(
    kimi_k2_5_model_config,
    image_url,
    video_url,
):
    image_uuid = "image_123"
    video_uuid = "video_456"
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Analyze this image and video."},
                {
                    "type": "image_url",
                    "image_url": {"url": image_url},
                    "uuid": image_uuid,
                },
                {
# ... 25 lines omitted for brevity ...
        }
    ]

    assert conversation == expected_conversation
    _assert_mm_data_is_vision_chunk_input(mm_data, 2)
    _assert_mm_uuids(
        mm_uuids, 2, expected_uuids=[image_uuid, video_uuid], modality="vision_chunk"
    )
```
**EN:** This test validates `test_parse_chat_messages_mixed_vision_chunk_with_uuid`. Key inputs are `kimi_k2_5_model_config`, `image_url`, `video_url`. The main assertion is `conversation == expected_conversation`.
**CN:** 这个测试验证 `test_parse_chat_messages_mixed_vision_chunk_with_uuid`。 关键输入包括 `kimi_k2_5_model_config`、`image_url`、`video_url`。 核心断言是 `conversation == expected_conversation`。

### Test / 测试: test_parse_chat_messages_mixed_vision_chunk_async (L2465-L2510)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_mixed_vision_chunk_async(
    kimi_k2_5_model_config,
    image_url,
    video_url,
):
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Analyze this image and video."},
                {
                    "type": "image_url",
                    "image_url": {"url": image_url},
                },
                {
                    "type": "video_url",
                    "video_url": {"url": video_url},
# ... 20 lines omitted for brevity ...
                "Analyze this image and video."
            ),
        }
    ]

    assert conversation == expected_conversation
    _assert_mm_data_is_vision_chunk_input(mm_data, 2)
    _assert_mm_uuids(mm_uuids, 2, expected_uuids=[None, None], modality="vision_chunk")
```
**EN:** This async test validates `test_parse_chat_messages_mixed_vision_chunk_async`. Relevant pytest markers include `asyncio`. Key inputs are `kimi_k2_5_model_config`, `image_url`, `video_url`. The main assertion is `conversation == expected_conversation`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_mixed_vision_chunk_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `kimi_k2_5_model_config`、`image_url`、`video_url`。 核心断言是 `conversation == expected_conversation`。

### Test / 测试: test_parse_chat_messages_mixed_vision_chunk_with_uuid_async (L2513-L2564)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_mixed_vision_chunk_with_uuid_async(
    kimi_k2_5_model_config,
    image_url,
    video_url,
):
    image_uuid = "image_123"
    video_uuid = "video_456"
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Analyze this image and video."},
                {
                    "type": "image_url",
                    "image_url": {"url": image_url},
                    "uuid": image_uuid,
                },
# ... 26 lines omitted for brevity ...
        }
    ]

    assert conversation == expected_conversation
    _assert_mm_data_is_vision_chunk_input(mm_data, 2)
    _assert_mm_uuids(
        mm_uuids, 2, expected_uuids=[image_uuid, video_uuid], modality="vision_chunk"
    )
```
**EN:** This async test validates `test_parse_chat_messages_mixed_vision_chunk_with_uuid_async`. Relevant pytest markers include `asyncio`. Key inputs are `kimi_k2_5_model_config`, `image_url`, `video_url`. The main assertion is `conversation == expected_conversation`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_mixed_vision_chunk_with_uuid_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `kimi_k2_5_model_config`、`image_url`、`video_url`。 核心断言是 `conversation == expected_conversation`。

### Test / 测试: test_parse_chat_messages_image_vision_chunk_async (L2567-L2601)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_image_vision_chunk_async(
    kimi_k2_5_model_config,
    image_url,
):
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Analyze this image."},
                {
                    "type": "image_url",
                    "image_url": {"url": image_url},
                },
            ],
        }
    ]

# ... 9 lines omitted for brevity ...
            "role": "user",
            "content": f"{placeholder}\nAnalyze this image.",
        }
    ]

    assert conversation == expected_conversation
    _assert_mm_data_is_vision_chunk_input(mm_data, 1)
    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[None], modality="vision_chunk")
```
**EN:** This async test validates `test_parse_chat_messages_image_vision_chunk_async`. Relevant pytest markers include `asyncio`. Key inputs are `kimi_k2_5_model_config`, `image_url`. The main assertion is `conversation == expected_conversation`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_image_vision_chunk_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `kimi_k2_5_model_config`、`image_url`。 核心断言是 `conversation == expected_conversation`。

### Test / 测试: test_parse_chat_messages_video_vision_chunk_async (L2604-L2638)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_video_vision_chunk_async(
    kimi_k2_5_model_config,
    video_url,
):
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Analyze this video."},
                {
                    "type": "video_url",
                    "video_url": {"url": video_url},
                },
            ],
        }
    ]

# ... 9 lines omitted for brevity ...
            "role": "user",
            "content": f"{placeholder}\nAnalyze this video.",
        }
    ]

    assert conversation == expected_conversation
    _assert_mm_data_is_vision_chunk_input(mm_data, 1)
    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[None], modality="vision_chunk")
```
**EN:** This async test validates `test_parse_chat_messages_video_vision_chunk_async`. Relevant pytest markers include `asyncio`. Key inputs are `kimi_k2_5_model_config`, `video_url`. The main assertion is `conversation == expected_conversation`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_video_vision_chunk_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `kimi_k2_5_model_config`、`video_url`。 核心断言是 `conversation == expected_conversation`。

### Test / 测试: test_parse_chat_messages_image_vision_chunk_with_uuid_async (L2641-L2677)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_image_vision_chunk_with_uuid_async(
    kimi_k2_5_model_config,
    image_url,
):
    image_uuid = "image_123"
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Analyze this image."},
                {
                    "type": "image_url",
                    "image_url": {"url": image_url},
                    "uuid": image_uuid,
                },
            ],
        }
# ... 11 lines omitted for brevity ...
            "role": "user",
            "content": f"{placeholder}\nAnalyze this image.",
        }
    ]

    assert conversation == expected_conversation
    _assert_mm_data_is_vision_chunk_input(mm_data, 1)
    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[image_uuid], modality="vision_chunk")
```
**EN:** This async test validates `test_parse_chat_messages_image_vision_chunk_with_uuid_async`. Relevant pytest markers include `asyncio`. Key inputs are `kimi_k2_5_model_config`, `image_url`. The main assertion is `conversation == expected_conversation`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_image_vision_chunk_with_uuid_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `kimi_k2_5_model_config`、`image_url`。 核心断言是 `conversation == expected_conversation`。

### Test / 测试: test_parse_chat_messages_video_vision_chunk_with_uuid_async (L2680-L2716)
```python
@pytest.mark.asyncio
async def test_parse_chat_messages_video_vision_chunk_with_uuid_async(
    kimi_k2_5_model_config,
    video_url,
):
    video_uuid = "video_456"
    messages = [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Analyze this video."},
                {
                    "type": "video_url",
                    "video_url": {"url": video_url},
                    "uuid": video_uuid,
                },
            ],
        }
# ... 11 lines omitted for brevity ...
            "role": "user",
            "content": f"{placeholder}\nAnalyze this video.",
        }
    ]

    assert conversation == expected_conversation
    _assert_mm_data_is_vision_chunk_input(mm_data, 1)
    _assert_mm_uuids(mm_uuids, 1, expected_uuids=[video_uuid], modality="vision_chunk")
```
**EN:** This async test validates `test_parse_chat_messages_video_vision_chunk_with_uuid_async`. Relevant pytest markers include `asyncio`. Key inputs are `kimi_k2_5_model_config`, `video_url`. The main assertion is `conversation == expected_conversation`.
**CN:** 这个异步测试验证 `test_parse_chat_messages_video_vision_chunk_with_uuid_async`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `kimi_k2_5_model_config`、`video_url`。 核心断言是 `conversation == expected_conversation`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。
- **EN:** Shared metadata and helper utilities keep nearby tests focused on behavior.
  **CN:** 共享元数据与辅助工具让相邻测试更专注于行为本身。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc.Mapping`, `typing.Literal`, `warnings`
- **Third-party / 第三方**: `pytest`, `torch`
- **Project / 项目内**: `vllm.assets.audio.AudioAsset`, `vllm.assets.image.ImageAsset`, `vllm.assets.video.VideoAsset`, `vllm.config.ModelConfig`, `vllm.entrypoints.chat_utils.parse_chat_messages`, `vllm.entrypoints.chat_utils.parse_chat_messages_async`, `vllm.inputs.MultiModalDataDict`, `vllm.inputs.MultiModalUUIDDict`, `vllm.multimodal.utils.encode_audio_url`, `vllm.multimodal.utils.encode_image_url`, `vllm.multimodal.utils.encode_video_url`, `vllm.utils.serial_utils.tensor2base64`
