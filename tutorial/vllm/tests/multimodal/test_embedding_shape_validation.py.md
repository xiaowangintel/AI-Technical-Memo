# test_embedding_shape_validation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/multimodal/test_embedding_shape_validation.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Embedding shape validation in multimodal APIs. / 该文件主要围绕 Embedding Shape Validation 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-21)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""
Embedding shape validation in multimodal APIs.

Tests verify that embeddings with correct ndim but incorrect hidden_size
are rejected before they can cause crashes during model inference.

Validation is performed by the parser (MultiModalDataParser) and EmbeddingItems
classes, not by MediaIO classes.
"""

import pytest
import torch

from vllm.multimodal.parse import (
    AudioEmbeddingItems,
    ImageEmbeddingItems,
    MultiModalDataParser,
    VideoEmbeddingItems,
)
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `pytest`, `torch`, `vllm.multimodal.parse`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Class: TestMultiModalParserShapeValidation (lines 24-113)
```python
class TestMultiModalParserShapeValidation:
    """Test hidden_size validation in MultiModalDataParser."""

    def test_image_embeddings_correct_hidden_size_accepted(self):
        """Baseline: Image embeddings with correct hidden_size should work."""
        expected_hidden_size = 768
        parser = MultiModalDataParser(expected_hidden_size=expected_hidden_size)

        valid_embeds = torch.randn(2, 100, expected_hidden_size)

        result = parser.parse_mm_data({"image": valid_embeds})

        assert "image" in result
        assert isinstance(result["image"], ImageEmbeddingItems)
        assert result["image"].get_count() == 2

    def test_image_embeddings_wrong_hidden_size_rejected(self):
        """Security: Image embeddings with wrong hidden_size should be rejected."""
        expected_hidden_size = 768
# ... omitted for brevity ...
        # Should identify which embedding failed
        assert "[1]" in str(exc_info.value)

    def test_validation_disabled_allows_any_size(self):
        """When validation disabled (legacy), any hidden_size allowed."""
        parser = MultiModalDataParser(expected_hidden_size=None)

        any_hidden_size = 12345
        embeds = torch.randn(2, 100, any_hidden_size)

        # Should not raise
        result = parser.parse_mm_data({"image": embeds})
        assert "image" in result
        assert isinstance(result["image"], ImageEmbeddingItems)
```
**EN:** Groups related scenarios for Testmultimodalparsershapevalidation. The class contains 6 test method(s).
**CN:** 该类把与 Testmultimodalparsershapevalidation 相关的场景组织在一起。 其中包含 6 个测试方法。

### Class: TestEmbeddingItemsDirectValidation (lines 116-174)
```python
class TestEmbeddingItemsDirectValidation:
    """Direct tests for EmbeddingItems hidden_size validation."""

    def test_image_embedding_items_validates_batched_tensor(self):
        """Test validation for batched (3D) image embeddings."""
        expected = 768
        wrong = 1024

        # Valid
        valid = torch.randn(2, 100, expected)
        items = ImageEmbeddingItems(valid, expected_hidden_size=expected)
        assert items.get_count() == 2

        # Invalid
        invalid = torch.randn(2, 100, wrong)
        with pytest.raises(ValueError) as exc_info:
            ImageEmbeddingItems(invalid, expected_hidden_size=expected)

        assert str(wrong) in str(exc_info.value)
# ... omitted for brevity ...
            AudioEmbeddingItems(invalid, expected_hidden_size=expected)

        assert "audio" in str(exc_info.value).lower()

    def test_video_embedding_items_validates(self):
        """Test validation for video embeddings."""
        expected = 768
        wrong = 384

        invalid = torch.randn(2, 100, wrong)
        with pytest.raises(ValueError) as exc_info:
            VideoEmbeddingItems(invalid, expected_hidden_size=expected)

        assert "video" in str(exc_info.value).lower()
```
**EN:** Groups related scenarios for Testembeddingitemsdirectvalidation. The class contains 4 test method(s).
**CN:** 该类把与 Testembeddingitemsdirectvalidation 相关的场景组织在一起。 其中包含 4 个测试方法。

### Class: TestShapeValidationIntegration (lines 177-223)
```python
class TestShapeValidationIntegration:
    """Integration tests verifying attack scenarios are blocked."""

    def test_attack_scenario_multimodal_image(self):
        """
        Simulate attack through Chat API with image embeddings.

        Verifies validation occurs in multimodal parser path.
        """
        expected_hidden_size = 768
        wrong_hidden_size = 4096
        parser = MultiModalDataParser(expected_hidden_size=expected_hidden_size)

        attack_tensor = torch.randn(1, 100, wrong_hidden_size)

        with pytest.raises(ValueError):
            parser.parse_mm_data({"image": attack_tensor})

    def test_attack_scenario_multimodal_audio(self):
# ... omitted for brevity ...
    def test_attack_scenario_multimodal_video(self):
        """
        Simulate attack through Chat API with video embeddings.

        Verifies validation occurs in multimodal parser path.
        """
        expected_hidden_size = 768
        wrong_hidden_size = 1024
        parser = MultiModalDataParser(expected_hidden_size=expected_hidden_size)

        attack_tensor = torch.randn(1, 100, wrong_hidden_size)

        with pytest.raises(ValueError):
            parser.parse_mm_data({"video": attack_tensor})
```
**EN:** Groups related scenarios for Testshapevalidationintegration. The class contains 3 test method(s).
**CN:** 该类把与 Testshapevalidationintegration 相关的场景组织在一起。 其中包含 3 个测试方法。

## Key Concepts / 关键概念
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Multimodal inputs / 多模态输入**
  - **EN:** The tests combine text with image, audio, or video data and verify the modality-specific pipeline.
  - **CN:** 这些测试把文本与图像、音频或视频数据结合起来，验证模态专用处理流程。
- **Assertions / 断言**
  - **EN:** The file relies on explicit assertions to pin down expected state transitions and outputs.
  - **CN:** 该文件通过显式断言固定预期的状态变化与输出结果。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `pytest`, `torch`
- **vLLM internal / vLLM 内部依赖**: `vllm.multimodal.parse`
