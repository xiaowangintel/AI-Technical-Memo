# test_qwen2_5_omni_embed.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/processing/test_qwen2_5_omni_embed.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 11 test(s), 0 fixture(s), and 10 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 11 个测试、0 个 fixture，以及 10 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L13-L21)
```python
from unittest.mock import Mock

import pytest
import torch

from vllm.model_executor.models.qwen2_5_omni_thinker import (
    check_interleaved_audio_video,
    merge_interleaved_embeddings,
)
```
**EN:** Imports standard-library modules such as `unittest.mock.Mock`, third-party packages like `pytest`, `torch`, project helpers such as `vllm.model_executor.models.interfaces.SupportsMultiModal`, `vllm.model_executor.models.qwen2_5_omni_thinker.Qwen2_5OmniThinkerForConditionalGeneration`, `vllm.model_executor.models.qwen2_5_omni_thinker.check_interleaved_audio_video`.
**CN:** 导入标准库模块（如 `unittest.mock.Mock`）、第三方包（如 `pytest`、`torch`）、项目内辅助模块（如 `vllm.model_executor.models.interfaces.SupportsMultiModal`、`vllm.model_executor.models.qwen2_5_omni_thinker.Qwen2_5OmniThinkerForConditionalGeneration`、`vllm.model_executor.models.qwen2_5_omni_thinker.check_interleaved_audio_video`）。

### Module setup / 模块级配置: AUDIO_TOKEN_ID, IMAGE_TOKEN_ID, VIDEO_TOKEN_ID (L24-L27)
```python
AUDIO_TOKEN_ID = 1001
IMAGE_TOKEN_ID = 1002
VIDEO_TOKEN_ID = 1003
TEXT_TOKEN_ID = 0
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `AUDIO_TOKEN_ID`, `IMAGE_TOKEN_ID`, `VIDEO_TOKEN_ID`, `TEXT_TOKEN_ID`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `AUDIO_TOKEN_ID`、`IMAGE_TOKEN_ID`、`VIDEO_TOKEN_ID`、`TEXT_TOKEN_ID`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: make_token_seq (L35-L59)
```python
def make_token_seq(
    audio_n: int, image_n: int, video_n: int, text_prefix: int = 3, text_sep: int = 2
):
    """
    Build a flat token sequence:
      [text_prefix] [AUDIO * audio_n] [text_sep] [IMAGE * image_n]
      [text_sep] [VIDEO * video_n] [text_sep]
    Returns (input_ids tensor, is_multimodal mask, positions dict).
    """
    tokens = (
        [TEXT_TOKEN_ID] * text_prefix
        + [AUDIO_TOKEN_ID] * audio_n
        + [TEXT_TOKEN_ID] * text_sep
        + [IMAGE_TOKEN_ID] * image_n
        + [TEXT_TOKEN_ID] * text_sep
        + [VIDEO_TOKEN_ID] * video_n
        + [TEXT_TOKEN_ID] * text_sep
    )
    input_ids = torch.tensor(tokens)
    is_multimodal = (
        (input_ids == AUDIO_TOKEN_ID)
        | (input_ids == IMAGE_TOKEN_ID)
        | (input_ids == VIDEO_TOKEN_ID)
    )
    return input_ids, is_multimodal
```
**EN:** This helper encapsulates reusable logic in `make_token_seq`. Key inputs are `audio_n`, `image_n`, `video_n`, `text_prefix`, `text_sep`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `make_token_seq` 中。 关键输入包括 `audio_n`、`image_n`、`video_n`、`text_prefix`、`text_sep`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: make_interleaved_seq (L62-L74)
```python
def make_interleaved_seq(
    video_chunks: list[int], audio_chunks: list[int], text_prefix: int = 2
):
    """
    Build an interleaved sequence like use_audio_in_video:
      [text] [V*v0] [A*a0] [V*v1] [A*a1] ...
    """
    tokens = [TEXT_TOKEN_ID] * text_prefix
    for v, a in zip(video_chunks, audio_chunks):
        tokens += [VIDEO_TOKEN_ID] * v + [AUDIO_TOKEN_ID] * a
    input_ids = torch.tensor(tokens)
    is_multimodal = (input_ids == VIDEO_TOKEN_ID) | (input_ids == AUDIO_TOKEN_ID)
    return input_ids, is_multimodal
```
**EN:** This helper encapsulates reusable logic in `make_interleaved_seq`. Key inputs are `video_chunks`, `audio_chunks`, `text_prefix`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `make_interleaved_seq` 中。 关键输入包括 `video_chunks`、`audio_chunks`、`text_prefix`。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: TestCheckInterleavedAudioVideo (L82-L143)
```python
class TestCheckInterleavedAudioVideo:
    def test_non_interleaved_audio_then_video(self):
        """Audio entirely before video → not interleaved."""
        input_ids, is_multimodal = make_token_seq(5, 0, 4)
        is_video = is_multimodal & (input_ids == VIDEO_TOKEN_ID)
        is_audio = is_multimodal & (input_ids == AUDIO_TOKEN_ID)
        assert not check_interleaved_audio_video(
            is_video, is_audio, is_video.sum().item(), is_audio.sum().item()
        )

    def test_non_interleaved_with_image(self):
        """Audio + image + video (the mixed_modalities case) → not interleaved."""
        input_ids, is_multimodal = make_token_seq(5, 4, 6)
        is_video = is_multimodal & (input_ids == VIDEO_TOKEN_ID)
# ... 40 lines omitted for brevity ...
            | (batched_ids == IMAGE_TOKEN_ID)
            | (batched_ids == VIDEO_TOKEN_ID)
        )
        is_video = is_multimodal & (batched_ids == VIDEO_TOKEN_ID)
        is_audio = is_multimodal & (batched_ids == AUDIO_TOKEN_ID)
        assert not check_interleaved_audio_video(
            is_video, is_audio, is_video.sum().item(), is_audio.sum().item()
        ), "Batched non-interleaved requests should not be detected as interleaved"
```
**EN:** This class groups related scenarios in `TestCheckInterleavedAudioVideo`. It contains 5 test method(s) and 0 supporting method(s). Representative methods include `test_non_interleaved_audio_then_video`, `test_non_interleaved_with_image`, `test_no_audio`.
**CN:** 该类将与 `TestCheckInterleavedAudioVideo` 相关的场景组织在一起。 它包含 5 个测试方法和 0 个辅助方法。 代表性方法包括 `test_non_interleaved_audio_then_video`、`test_non_interleaved_with_image`、`test_no_audio`。

### Test method / 测试方法: TestCheckInterleavedAudioVideo.test_non_interleaved_audio_then_video (L83-L90)
```python
    def test_non_interleaved_audio_then_video(self):
        """Audio entirely before video → not interleaved."""
        input_ids, is_multimodal = make_token_seq(5, 0, 4)
        is_video = is_multimodal & (input_ids == VIDEO_TOKEN_ID)
        is_audio = is_multimodal & (input_ids == AUDIO_TOKEN_ID)
        assert not check_interleaved_audio_video(
            is_video, is_audio, is_video.sum().item(), is_audio.sum().item()
        )
```
**EN:** This test validates `TestCheckInterleavedAudioVideo.test_non_interleaved_audio_then_video`. The main assertion is `not check_interleaved_audio_video(is_video, is_audio, is_video.sum().item(), is_audio.sum().item())`.
**CN:** 这个测试验证 `TestCheckInterleavedAudioVideo.test_non_interleaved_audio_then_video`。 核心断言是 `not check_interleaved_audio_video(is_video, is_audio, is_video.sum().item(), is_audio.sum().item())`。

### Test method / 测试方法: TestCheckInterleavedAudioVideo.test_non_interleaved_with_image (L92-L99)
```python
    def test_non_interleaved_with_image(self):
        """Audio + image + video (the mixed_modalities case) → not interleaved."""
        input_ids, is_multimodal = make_token_seq(5, 4, 6)
        is_video = is_multimodal & (input_ids == VIDEO_TOKEN_ID)
        is_audio = is_multimodal & (input_ids == AUDIO_TOKEN_ID)
        assert not check_interleaved_audio_video(
            is_video, is_audio, is_video.sum().item(), is_audio.sum().item()
        )
```
**EN:** This test validates `TestCheckInterleavedAudioVideo.test_non_interleaved_with_image`. The main assertion is `not check_interleaved_audio_video(is_video, is_audio, is_video.sum().item(), is_audio.sum().item())`.
**CN:** 这个测试验证 `TestCheckInterleavedAudioVideo.test_non_interleaved_with_image`。 核心断言是 `not check_interleaved_audio_video(is_video, is_audio, is_video.sum().item(), is_audio.sum().item())`。

### Test method / 测试方法: TestCheckInterleavedAudioVideo.test_no_audio (L101-L108)
```python
    def test_no_audio(self):
        """Video only → not interleaved."""
        input_ids, is_multimodal = make_token_seq(0, 0, 6)
        is_video = is_multimodal & (input_ids == VIDEO_TOKEN_ID)
        is_audio = is_multimodal & (input_ids == AUDIO_TOKEN_ID)
        assert not check_interleaved_audio_video(
            is_video, is_audio, is_video.sum().item(), is_audio.sum().item()
        )
```
**EN:** This test validates `TestCheckInterleavedAudioVideo.test_no_audio`. The main assertion is `not check_interleaved_audio_video(is_video, is_audio, is_video.sum().item(), is_audio.sum().item())`.
**CN:** 这个测试验证 `TestCheckInterleavedAudioVideo.test_no_audio`。 核心断言是 `not check_interleaved_audio_video(is_video, is_audio, is_video.sum().item(), is_audio.sum().item())`。

### Test method / 测试方法: TestCheckInterleavedAudioVideo.test_interleaved (L110-L117)
```python
    def test_interleaved(self):
        """V A V A interleaved → True."""
        input_ids, is_multimodal = make_interleaved_seq([4, 4], [3, 3])
        is_video = is_multimodal & (input_ids == VIDEO_TOKEN_ID)
        is_audio = is_multimodal & (input_ids == AUDIO_TOKEN_ID)
        assert check_interleaved_audio_video(
            is_video, is_audio, is_video.sum().item(), is_audio.sum().item()
        )
```
**EN:** This test validates `TestCheckInterleavedAudioVideo.test_interleaved`. The main assertion is `check_interleaved_audio_video(is_video, is_audio, is_video.sum().item(), is_audio.sum().item())`.
**CN:** 这个测试验证 `TestCheckInterleavedAudioVideo.test_interleaved`。 核心断言是 `check_interleaved_audio_video(is_video, is_audio, is_video.sum().item(), is_audio.sum().item())`。

### Test method / 测试方法: TestCheckInterleavedAudioVideo.test_batched_non_interleaved_no_false_positive (L119-L143)
```python
    def test_batched_non_interleaved_no_false_positive(self):
        """
        Regression test for https://github.com/vllm-project/vllm/issues/35394.

        5 identical non-interleaved mixed-modality requests batched together:
        each has [audio][image][video] in separate blocks with text between them.
        Across the batch, audio from request N falls between video blocks of
        request N and request N+1, causing the global ranges to overlap.
        check_interleaved_audio_video must return False (not a false positive).
        """
        # Build one request: [text][audio*5][text][image*4][text][video*6][text]
        single_ids, _ = make_token_seq(5, 4, 6)
        # Batch 5 identical requests (separated by text tokens to simulate padding)
        sep = torch.tensor([TEXT_TOKEN_ID] * 3)
        batched_ids = torch.cat([single_ids, sep] * 5)
        is_multimodal = (
            (batched_ids == AUDIO_TOKEN_ID)
            | (batched_ids == IMAGE_TOKEN_ID)
            | (batched_ids == VIDEO_TOKEN_ID)
        )
        is_video = is_multimodal & (batched_ids == VIDEO_TOKEN_ID)
        is_audio = is_multimodal & (batched_ids == AUDIO_TOKEN_ID)
        assert not check_interleaved_audio_video(
            is_video, is_audio, is_video.sum().item(), is_audio.sum().item()
        ), "Batched non-interleaved requests should not be detected as interleaved"
```
**EN:** This test validates `TestCheckInterleavedAudioVideo.test_batched_non_interleaved_no_false_positive`. The main assertion is `not check_interleaved_audio_video(is_video, is_audio, is_video.sum().item(), is_audio.sum().item())`.
**CN:** 这个测试验证 `TestCheckInterleavedAudioVideo.test_batched_non_interleaved_no_false_positive`。 核心断言是 `not check_interleaved_audio_video(is_video, is_audio, is_video.sum().item(), is_audio.sum().item())`。

### Helper / 辅助函数: make_mock_model (L151-L210)
```python
def make_mock_model(hidden: int = 8):
    """
    Return a minimal mock of Qwen2_5OmniThinkerForConditionalGeneration
    that has enough structure to run embed_input_ids.
    """
    from vllm.model_executor.models.qwen2_5_omni_thinker import (
        Qwen2_5OmniThinkerForConditionalGeneration,
    )

    model = Mock(spec=Qwen2_5OmniThinkerForConditionalGeneration)

    # Config with token IDs
    cfg = Mock()
    cfg.video_token_index = VIDEO_TOKEN_ID
    cfg.audio_token_index = AUDIO_TOKEN_ID
    model.config = cfg

    # embed_input_ids: simply embed each token as a one-hot-like vector
# ... 34 lines omitted for brevity ...
            model, *a, **kw
        )
    )

    # Store super-embed for use inside the method
    model._super_embed_input_ids = fake_super_embed

    return model, hidden
```
**EN:** This helper encapsulates reusable logic in `make_mock_model`. Key inputs are `hidden`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `make_mock_model` 中。 关键输入包括 `hidden`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: build_mm_embeds (L213-L227)
```python
def build_mm_embeds(
    audio_n, image_n, video_n, hidden, audio_val=10.0, image_val=20.0, video_val=30.0
):
    """
    Build multimodal_embeddings list in position order (audio, image, video).
    Each embedding is filled with a distinct constant so we can verify placement.
    """
    embs = []
    if audio_n:
        embs.append(torch.full((audio_n, hidden), audio_val))
    if image_n:
        embs.append(torch.full((image_n, hidden), image_val))
    if video_n:
        embs.append(torch.full((video_n, hidden), video_val))
    return embs
```
**EN:** This helper encapsulates reusable logic in `build_mm_embeds`. Key inputs are `audio_n`, `image_n`, `video_n`, `hidden`, `audio_val`, `image_val`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `build_mm_embeds` 中。 关键输入包括 `audio_n`、`image_n`、`video_n`、`hidden`、`audio_val`、`image_val`。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: TestEmbedInputIds (L230-L344)
```python
class TestEmbedInputIds:
    def _run(self, audio_n, image_n, video_n, hidden=8):
        """
        Run embed_input_ids for a non-interleaved mixed-modality sequence.
        Returns (result_embeds, input_ids, is_multimodal).
        """
        input_ids, is_multimodal = make_token_seq(audio_n, image_n, video_n)
        mm_embeds = build_mm_embeds(audio_n, image_n, video_n, hidden)

        model, _ = make_mock_model(hidden)
        result = model.embed_input_ids(
            input_ids, mm_embeds, is_multimodal=is_multimodal
        )
        return result, input_ids, is_multimodal
# ... 93 lines omitted for brevity ...

        assert result[video_pos].allclose(torch.full((video_n, hidden), video_val)), (
            "Interleaved: video positions should get video embeddings"
        )

        assert result[audio_pos].allclose(torch.full((audio_n, hidden), audio_val)), (
            "Interleaved: audio positions should get audio embeddings"
        )
```
**EN:** This class groups related scenarios in `TestEmbedInputIds`. It contains 5 test method(s) and 1 supporting method(s). Representative methods include `test_audio_only`, `test_video_only`, `test_mixed_modalities_audio_goes_to_audio_pos`, `_run`.
**CN:** 该类将与 `TestEmbedInputIds` 相关的场景组织在一起。 它包含 5 个测试方法和 1 个辅助方法。 代表性方法包括 `test_audio_only`、`test_video_only`、`test_mixed_modalities_audio_goes_to_audio_pos`、`_run`。

### Helper method / 辅助方法: TestEmbedInputIds._run (L231-L243)
```python
    def _run(self, audio_n, image_n, video_n, hidden=8):
        """
        Run embed_input_ids for a non-interleaved mixed-modality sequence.
        Returns (result_embeds, input_ids, is_multimodal).
        """
        input_ids, is_multimodal = make_token_seq(audio_n, image_n, video_n)
        mm_embeds = build_mm_embeds(audio_n, image_n, video_n, hidden)

        model, _ = make_mock_model(hidden)
        result = model.embed_input_ids(
            input_ids, mm_embeds, is_multimodal=is_multimodal
        )
        return result, input_ids, is_multimodal
```
**EN:** This helper encapsulates reusable logic in `TestEmbedInputIds._run`. Key inputs are `audio_n`, `image_n`, `video_n`, `hidden`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `TestEmbedInputIds._run` 中。 关键输入包括 `audio_n`、`image_n`、`video_n`、`hidden`。 它把计算得到的状态或辅助对象返回给调用方。

### Test method / 测试方法: TestEmbedInputIds.test_audio_only (L245-L254)
```python
    def test_audio_only(self):
        """Audio-only: audio positions get audio embeddings."""
        audio_n, hidden = 5, 8
        audio_val = 10.0
        result, input_ids, is_multimodal = self._run(audio_n, 0, 0, hidden)

        audio_pos = (input_ids == AUDIO_TOKEN_ID).nonzero(as_tuple=True)[0]
        assert result[audio_pos].allclose(torch.full((audio_n, hidden), audio_val)), (
            "Audio positions should get audio embeddings"
        )
```
**EN:** This test validates `TestEmbedInputIds.test_audio_only`. The main assertion is `result[audio_pos].allclose(torch.full((audio_n, hidden), audio_val))`.
**CN:** 这个测试验证 `TestEmbedInputIds.test_audio_only`。 核心断言是 `result[audio_pos].allclose(torch.full((audio_n, hidden), audio_val))`。

### Test method / 测试方法: TestEmbedInputIds.test_video_only (L256-L265)
```python
    def test_video_only(self):
        """Video-only: video positions get video embeddings."""
        video_n, hidden = 6, 8
        video_val = 30.0
        result, input_ids, is_multimodal = self._run(0, 0, video_n, hidden)

        video_pos = (input_ids == VIDEO_TOKEN_ID).nonzero(as_tuple=True)[0]
        assert result[video_pos].allclose(torch.full((video_n, hidden), video_val)), (
            "Video positions should get video embeddings"
        )
```
**EN:** This test validates `TestEmbedInputIds.test_video_only`. The main assertion is `result[video_pos].allclose(torch.full((video_n, hidden), video_val))`.
**CN:** 这个测试验证 `TestEmbedInputIds.test_video_only`。 核心断言是 `result[video_pos].allclose(torch.full((video_n, hidden), video_val))`。

### Test method / 测试方法: TestEmbedInputIds.test_mixed_modalities_audio_goes_to_audio_pos (L267-L295)
```python
    def test_mixed_modalities_audio_goes_to_audio_pos(self):
        """
        Regression test for GitHub issue #34506:
        With audio + image + video (non-interleaved), audio positions must
        receive audio embeddings (not image or video embeddings).
        """
        audio_n, image_n, video_n, hidden = 5, 4, 6, 8
        audio_val, image_val, video_val = 10.0, 20.0, 30.0

        result, input_ids, is_multimodal = self._run(audio_n, image_n, video_n, hidden)

        audio_pos = (input_ids == AUDIO_TOKEN_ID).nonzero(as_tuple=True)[0]
        image_pos = (input_ids == IMAGE_TOKEN_ID).nonzero(as_tuple=True)[0]
        video_pos = (input_ids == VIDEO_TOKEN_ID).nonzero(as_tuple=True)[0]

        mean_a = result[audio_pos].mean().item()
        assert result[audio_pos].allclose(torch.full((audio_n, hidden), audio_val)), (
            f"Audio emb wrong: expected {audio_val}, got mean={mean_a:.1f}"
        )

        mean_i = result[image_pos].mean().item()
        assert result[image_pos].allclose(torch.full((image_n, hidden), image_val)), (
            f"Image emb wrong: expected {image_val}, got mean={mean_i:.1f}"
        )

        mean_v = result[video_pos].mean().item()
        assert result[video_pos].allclose(torch.full((video_n, hidden), video_val)), (
            f"Video emb wrong: expected {video_val}, got mean={mean_v:.1f}"
        )
```
**EN:** This test validates `TestEmbedInputIds.test_mixed_modalities_audio_goes_to_audio_pos`. The main assertion is `result[audio_pos].allclose(torch.full((audio_n, hidden), audio_val))` and `result[image_pos].allclose(torch.full((image_n, hidden), image_val))`.
**CN:** 这个测试验证 `TestEmbedInputIds.test_mixed_modalities_audio_goes_to_audio_pos`。 核心断言是 `result[audio_pos].allclose(torch.full((audio_n, hidden), audio_val))` and `result[image_pos].allclose(torch.full((image_n, hidden), image_val))`。

### Test method / 测试方法: TestEmbedInputIds.test_text_positions_unchanged (L297-L306)
```python
    def test_text_positions_unchanged(self):
        """Text positions should keep their text embeddings."""
        audio_n, image_n, video_n, hidden = 3, 2, 4, 8
        result, input_ids, is_multimodal = self._run(audio_n, image_n, video_n, hidden)

        text_pos = (~is_multimodal).nonzero(as_tuple=True)[0]
        # Text tokens have value TEXT_TOKEN_ID=0, so embed → 0.0
        assert result[text_pos].allclose(torch.zeros(len(text_pos), hidden)), (
            "Text positions should keep text embeddings"
        )
```
**EN:** This test validates `TestEmbedInputIds.test_text_positions_unchanged`. The main assertion is `result[text_pos].allclose(torch.zeros(len(text_pos), hidden))`.
**CN:** 这个测试验证 `TestEmbedInputIds.test_text_positions_unchanged`。 核心断言是 `result[text_pos].allclose(torch.zeros(len(text_pos), hidden))`。

### Test method / 测试方法: TestEmbedInputIds.test_interleaved_use_audio_in_video (L308-L344)
```python
    def test_interleaved_use_audio_in_video(self):
        """
        Interleaved (use_audio_in_video): video chunks interleaved with audio.
        Video embeddings must go to video positions, audio to audio positions.
        """
        hidden = 8
        audio_val, video_val = 10.0, 30.0
        # Two video chunks of 4, two audio chunks of 3
        video_chunks = [4, 4]
        audio_chunks = [3, 3]
        input_ids, is_multimodal = make_interleaved_seq(video_chunks, audio_chunks)

        video_n = sum(video_chunks)  # 8
        audio_n = sum(audio_chunks)  # 6

        # mm_embeds come in [video, audio] order (video feature first in
        # mm_features when positions are the same for use_audio_in_video)
        mm_embeds = [
# ... 11 lines omitted for brevity ...

        assert result[video_pos].allclose(torch.full((video_n, hidden), video_val)), (
            "Interleaved: video positions should get video embeddings"
        )

        assert result[audio_pos].allclose(torch.full((audio_n, hidden), audio_val)), (
            "Interleaved: audio positions should get audio embeddings"
        )
```
**EN:** This test validates `TestEmbedInputIds.test_interleaved_use_audio_in_video`. The main assertion is `result[video_pos].allclose(torch.full((video_n, hidden), video_val))` and `result[audio_pos].allclose(torch.full((audio_n, hidden), audio_val))`.
**CN:** 这个测试验证 `TestEmbedInputIds.test_interleaved_use_audio_in_video`。 核心断言是 `result[video_pos].allclose(torch.full((video_n, hidden), video_val))` and `result[audio_pos].allclose(torch.full((audio_n, hidden), audio_val))`。

### Class / 类: TestMergeInterleavedEmbeddings (L352-L382)
```python
class TestMergeInterleavedEmbeddings:
    def test_basic_interleaved(self):
        """Video chunks + audio chunks scattered to correct positions."""
        hidden = 4
        input_ids, is_multimodal = make_interleaved_seq([3, 3], [2, 2])

        is_video = is_multimodal & (input_ids == VIDEO_TOKEN_ID)
        is_audio = is_multimodal & (input_ids == AUDIO_TOKEN_ID)
        num_video = is_video.sum().item()  # 6
        num_audio = is_audio.sum().item()  # 4

        inputs_embeds = torch.zeros(len(input_ids), hidden)
        mm_embeds = [
            torch.full((num_video, hidden), 30.0),
# ... 9 lines omitted for brevity ...
            num_video,
            num_audio,
        )

        video_pos = is_video.nonzero(as_tuple=True)[0]
        audio_pos = is_audio.nonzero(as_tuple=True)[0]
        assert result[video_pos].allclose(torch.full((num_video, hidden), 30.0))
        assert result[audio_pos].allclose(torch.full((num_audio, hidden), 10.0))
```
**EN:** This class groups related scenarios in `TestMergeInterleavedEmbeddings`. It contains 1 test method(s) and 0 supporting method(s). Representative methods include `test_basic_interleaved`.
**CN:** 该类将与 `TestMergeInterleavedEmbeddings` 相关的场景组织在一起。 它包含 1 个测试方法和 0 个辅助方法。 代表性方法包括 `test_basic_interleaved`。

### Test method / 测试方法: TestMergeInterleavedEmbeddings.test_basic_interleaved (L353-L382)
```python
    def test_basic_interleaved(self):
        """Video chunks + audio chunks scattered to correct positions."""
        hidden = 4
        input_ids, is_multimodal = make_interleaved_seq([3, 3], [2, 2])

        is_video = is_multimodal & (input_ids == VIDEO_TOKEN_ID)
        is_audio = is_multimodal & (input_ids == AUDIO_TOKEN_ID)
        num_video = is_video.sum().item()  # 6
        num_audio = is_audio.sum().item()  # 4

        inputs_embeds = torch.zeros(len(input_ids), hidden)
        mm_embeds = [
            torch.full((num_video, hidden), 30.0),
            torch.full((num_audio, hidden), 10.0),
        ]

        result = merge_interleaved_embeddings(
            inputs_embeds,
            mm_embeds,
            is_video,
            is_audio,
            is_multimodal,
            num_video,
            num_audio,
        )

        video_pos = is_video.nonzero(as_tuple=True)[0]
        audio_pos = is_audio.nonzero(as_tuple=True)[0]
        assert result[video_pos].allclose(torch.full((num_video, hidden), 30.0))
        assert result[audio_pos].allclose(torch.full((num_audio, hidden), 10.0))
```
**EN:** This test validates `TestMergeInterleavedEmbeddings.test_basic_interleaved`. The main assertion is `result[video_pos].allclose(torch.full((num_video, hidden), 30.0))` and `result[audio_pos].allclose(torch.full((num_audio, hidden), 10.0))`.
**CN:** 这个测试验证 `TestMergeInterleavedEmbeddings.test_basic_interleaved`。 核心断言是 `result[video_pos].allclose(torch.full((num_video, hidden), 30.0))` and `result[audio_pos].allclose(torch.full((num_audio, hidden), 10.0))`。

### Module setup / 模块级配置: module state (L385-L386)
```python
if __name__ == "__main__":
    pytest.main([__file__, "-v"])
```
**EN:** This block performs module-level configuration such as conditional imports, environment checks, or shared setup logic.
**CN:** 该代码块执行模块级配置，例如条件导入、环境检查或共享初始化逻辑。

## Key Concepts / 关键概念
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `unittest.mock.Mock`
- **Third-party / 第三方**: `pytest`, `torch`
- **Project / 项目内**: `vllm.model_executor.models.interfaces.SupportsMultiModal`, `vllm.model_executor.models.qwen2_5_omni_thinker.Qwen2_5OmniThinkerForConditionalGeneration`, `vllm.model_executor.models.qwen2_5_omni_thinker.check_interleaved_audio_video`, `vllm.model_executor.models.qwen2_5_omni_thinker.merge_interleaved_embeddings`
