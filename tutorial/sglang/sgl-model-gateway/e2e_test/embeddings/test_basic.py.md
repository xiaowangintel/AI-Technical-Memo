# test_basic.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/embeddings/test_basic.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises basic behavior in the embeddings end-to-end suite. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 Embeddings 端到端测试 中与 basic 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Module docstring
```python
"""Basic embedding API tests.

Tests the embedding functionality through the router with both gRPC and HTTP backends.

Source: Migrated from e2e_grpc/basic/test_embedding_server.py

Usage:
    pytest e2e_test/embeddings/test_basic.py -v
    pytest e2e_test/embeddings/test_basic.py -v -k "grpc"
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 12-16: Imports and dependencies
```python
from __future__ import annotations

import logging

import pytest
```
**EN:** This block imports `__future__`, `logging`, `pytest`, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 18-32: Module constants and configuration
```python
logger = logging.getLogger(__name__)


_GRPC_EMBEDDING_SKIP_REASON = (
    "SMG router's vendored smg-grpc-client (pinned at =1.0.0 in "
    "sgl-model-gateway/Cargo.toml) uses the legacy oneof EmbedResponse "
    "proto. Python smg-grpc-servicer >= 0.5.2 (the only version compatible "
    "with current sglang utils + MultimodalInputs APIs) emits the new flat "
    "EmbedResponse layout. Wire-format mismatch -> Rust client decodes to "
    "all-None oneof variants -> 'embedding_no_response' 500. Re-enable once "
    "sgl-model-gateway is bumped to smg-grpc-client >= 1.4.0 (which has the "
    "flat proto); that bump cascades through openai-protocol 1.7.0 + several "
    "other crates and is its own coordinated effort. HTTP backend variant of "
    "these tests still runs and validates the embedding pipeline end-to-end."
)
```
**EN:** This section defines module-level names such as `logger`, `_GRPC_EMBEDDING_SKIP_REASON`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 33-166: Test class `TestEmbeddingBasic`
```python


@pytest.mark.e2e
@pytest.mark.model("embedding")
@pytest.mark.parametrize(
    "setup_backend",
    [
        pytest.param(
            "grpc", marks=pytest.mark.skip(reason=_GRPC_EMBEDDING_SKIP_REASON)
        ),
        "http",
    ],
    indirect=True,
)
class TestEmbeddingBasic:
    """Basic embedding API tests using local workers (HTTP — gRPC variant skipped)."""

    def test_embedding_single(self, setup_backend):
        """Test single text embedding.

        Verifies that:
        - Response object structure is correct
        - Embedding is a non-empty list of floats
        - Usage statistics are present
        """
        backend, model, client, gateway = setup_backend

        input_text = "Hello world"
        response = client.embeddings.create(
            model=model,
            input=input_text,
        )

        assert response.object == "list"
        assert len(response.data) == 1

        embedding = response.data[0]
        assert embedding.object == "embedding"
        assert embedding.index == 0
        assert len(embedding.embedding) > 0
        assert isinstance(embedding.embedding[0], float)

        # Verify usage statistics
        assert response.usage.prompt_tokens > 0
        assert response.usage.total_tokens == response.usage.prompt_tokens

        logger.info(
            "Single embedding: %d dimensions, %d tokens",
            len(embedding.embedding),
            response.usage.prompt_tokens,
        )

    def test_embedding_batch(self, setup_backend):
        """Test batch embedding with multiple texts.

        Note: The original test expected len(response.data) == 1 for batch,
        which seems incorrect. This might be model-specific behavior.
        """
        backend, model, client, gateway = setup_backend

        input_texts = ["Hello world", "SGLang is fast"]
        response = client.embeddings.create(
            model=model,
            input=input_texts,
        )

        # Note: Original test had len(response.data) == 1, which seems like
        # a bug or model-specific behavior. Standard behavior should return
        # one embedding per input text.
        assert len(response.data) >= 1
        assert response.data[0].index == 0
        assert len(response.data[0].embedding) > 0

        logger.info("Batch embedding: %d results", len(response.data))

    def test_embedding_dimensions_consistent(self, setup_backend):
        """Test that embedding dimensions are consistent across different inputs.

        Verifies that different length inputs produce embeddings with
        the same dimensionality.
        """
        backend, model, client, gateway = setup_backend

        response1 = client.embeddings.create(
            model=model,
            input="A short text",
        )
        dim1 = len(response1.data[0].embedding)

        response2 = client.embeddings.create(
            model=model,
            input="A much longer text to ensure dimensions match regardless of input length",
        )
        dim2 = len(response2.data[0].embedding)

        assert dim1 == dim2, f"Dimensions differ: {dim1} vs {dim2}"
        logger.info("Embedding dimensions: %d (consistent)", dim1)

    def test_embedding_empty_string(self, setup_backend):
        """Test embedding with empty string input.

        Some models may handle empty strings differently.
        This test verifies the API doesn't crash on empty input.
        """
        backend, model, client, gateway = setup_backend

        try:
            response = client.embeddings.create(
                model=model,
                input="",
            )
            # If it succeeds, verify structure
            assert len(response.data) >= 1
            logger.info("Empty string embedding succeeded")
        except Exception as e:
            # Some models may reject empty strings - that's acceptable
            logger.info("Empty string embedding rejected: %s", e)

    def test_embedding_unicode(self, setup_backend):
        """Test embedding with unicode characters.

        Verifies that the API handles non-ASCII characters correctly.
        """
        backend, model, client, gateway = setup_backend

        input_text = "Hello 世界! 🚀 Привет мир"
        response = client.embeddings.create(
            model=model,
            input=input_text,
        )

        assert len(response.data) == 1
        assert len(response.data[0].embedding) > 0
        logger.info("Unicode embedding: %d dimensions", len(response.data[0].embedding))
```
**EN:** This test class groups related scenarios around `TestEmbeddingBasic` and organizes shared assertions or helpers in one place.
**CN:** 这个测试类围绕 `TestEmbeddingBasic` 组织相关场景，并将共享断言或辅助逻辑集中在同一位置。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Embedding generation and correctness checks / 向量嵌入生成与正确性校验
- Primary classes: `TestEmbeddingBasic` / 主要类：`TestEmbeddingBasic`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`
- **Third-party / 第三方**: `pytest`
- **Internal / 内部模块**: None explicitly imported / 未显式导入
