# test_sparse_tensor_validation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/renderers/test_sparse_tensor_validation.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Tests verify that malicious sparse tensors are rejected before they can trigger out-of-bounds memory writes during to_dense() operations. / 该文件主要围绕 Sparse Tensor Validation 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-17)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""
Tests verify that malicious sparse tensors are rejected before they can trigger
out-of-bounds memory writes during to_dense() operations.
"""

import io

import numpy as np
import pybase64 as base64
import pytest
import torch

from vllm.exceptions import VLLMValidationError
from vllm.multimodal.media import AudioEmbeddingMediaIO, ImageEmbeddingMediaIO
from vllm.renderers.embed_utils import safe_load_prompt_embeds
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `io`, `numpy`, `pybase64`, `vllm.exceptions`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: model_config (lines 20-33)
```python
@pytest.fixture
def model_config():
    """Mock ModelConfig for testing."""
    from vllm.config import ModelConfig

    return ModelConfig(
        model="facebook/opt-125m",
        tokenizer="facebook/opt-125m",
        tokenizer_mode="auto",
        trust_remote_code=False,
        dtype="float32",
        seed=0,
        enable_prompt_embeds=True,  # Required for prompt embeds tests
    )
```
**EN:** Mock ModelConfig for testing. The fixture mainly builds or returns values through `ModelConfig`.
**CN:** 该代码块定义 pytest 夹具 `model_config`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `ModelConfig` 构造或返回测试所需的值。

### Helper: _encode_tensor (lines 36-41)
```python
def _encode_tensor(tensor: torch.Tensor) -> bytes:
    """Helper to encode a tensor as base64 bytes."""
    buffer = io.BytesIO()
    torch.save(tensor, buffer)
    buffer.seek(0)
    return base64.b64encode(buffer.read())
```
**EN:** Helper to encode a tensor as base64 bytes. It coordinates operations such as `io.BytesIO`, `torch.save`, `buffer.seek`.
**CN:** 该辅助函数为 Encode Tensor 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `io.BytesIO`, `torch.save`, `buffer.seek` 等操作。

### Helper: _create_malicious_sparse_tensor (lines 44-65)
```python
def _create_malicious_sparse_tensor() -> torch.Tensor:
    """
    Create a malicious sparse COO tensor with out-of-bounds indices.

    This tensor has indices that point beyond the declared shape, which would
    cause an out-of-bounds write when converted to dense format without
    validation.
    """
    # Create a 3x3 sparse tensor but with indices pointing to (10, 10)
    indices = torch.tensor([[10], [10]])  # Out of bounds for 3x3 shape
    values = torch.tensor([1.0])
    shape = (3, 3)

    # Create sparse tensor (this will be invalid). Pass `check_invariants=False`
    # explicitly so this fixture is robust to process-wide invariant-check state
    # left enabled by other tests (the global flag isn't thread-local, and
    # concurrent users of the `check_sparse_tensor_invariants` context manager
    # can leak the "enabled" state across tests).
    sparse_tensor = torch.sparse_coo_tensor(
        indices, values, shape, dtype=torch.float32, check_invariants=False
    )
    return sparse_tensor
```
**EN:** Create a malicious sparse COO tensor with out-of-bounds indices. It coordinates operations such as `torch.tensor`, `torch.sparse_coo_tensor`.
**CN:** 该辅助函数为 Create Malicious Sparse Tensor 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `torch.tensor`, `torch.sparse_coo_tensor` 等操作。

### Helper: _create_valid_sparse_tensor (lines 68-75)
```python
def _create_valid_sparse_tensor() -> torch.Tensor:
    """Create a valid sparse COO tensor for baseline testing."""
    indices = torch.tensor([[0, 1, 2], [0, 1, 2]])
    values = torch.tensor([1.0, 2.0, 3.0])
    shape = (3, 3)

    sparse_tensor = torch.sparse_coo_tensor(indices, values, shape, dtype=torch.float32)
    return sparse_tensor
```
**EN:** Create a valid sparse COO tensor for baseline testing. It coordinates operations such as `torch.tensor`, `torch.sparse_coo_tensor`.
**CN:** 该辅助函数为 Create Valid Sparse Tensor 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `torch.tensor`, `torch.sparse_coo_tensor` 等操作。

### Helper: _create_valid_dense_tensor (lines 78-80)
```python
def _create_valid_dense_tensor() -> torch.Tensor:
    """Create a valid dense tensor for baseline testing."""
    return torch.randn(10, 768, dtype=torch.float32)  # (seq_len, hidden_size)
```
**EN:** Create a valid dense tensor for baseline testing. It coordinates operations such as `torch.randn`.
**CN:** 该辅助函数为 Create Valid Dense Tensor 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `torch.randn` 等操作。

### Class: TestPromptEmbedsValidation (lines 83-203)
```python
class TestPromptEmbedsValidation:
    """Test sparse tensor validation in prompt embeddings (Completions API)."""

    def test_valid_dense_tensor_accepted(self, model_config):
        """Baseline: Valid dense tensors should work normally."""
        valid_tensor = _create_valid_dense_tensor()
        encoded = _encode_tensor(valid_tensor)

        # Should not raise any exception
        result = safe_load_prompt_embeds(model_config, encoded)
        assert result.shape == valid_tensor.shape

    def test_valid_sparse_tensor_accepted(self):
        """Baseline: Valid sparse tensors should load successfully."""
        io_handler = ImageEmbeddingMediaIO()

        valid_sparse = _create_valid_sparse_tensor()
        encoded = _encode_tensor(valid_sparse)

# ... omitted for brevity ...
            safe_load_prompt_embeds(model_config, encoded)

    def test_non_tensor_payload_rejected(self, model_config):
        """Deserializing to a non-Tensor object must raise a clear error
        instead of propagating an AssertionError."""
        # `torch.save` will serialize a plain dict; `weights_only=True` allows
        # loading built-in containers, so this exercises the isinstance check.
        buffer = io.BytesIO()
        torch.save({"not": "a tensor"}, buffer)
        buffer.seek(0)
        encoded = base64.b64encode(buffer.read())

        with pytest.raises(VLLMValidationError, match="torch.Tensor"):
            safe_load_prompt_embeds(model_config, encoded)
```
**EN:** Groups related scenarios for Testpromptembedsvalidation. The class contains 10 test method(s).
**CN:** 该类把与 Testpromptembedsvalidation 相关的场景组织在一起。 其中包含 10 个测试方法。

### Class: TestImageEmbedsValidation (lines 206-300)
```python
class TestImageEmbedsValidation:
    """Test sparse tensor validation in image embeddings (Chat API)."""

    def test_valid_dense_tensor_accepted(self):
        """Baseline: Valid dense tensors should work normally."""
        io_handler = ImageEmbeddingMediaIO()

        valid_tensor = _create_valid_dense_tensor()
        encoded = _encode_tensor(valid_tensor)

        # Should not raise any exception
        result = io_handler.load_base64("", encoded.decode("utf-8"))
        assert result.shape == valid_tensor.shape

    def test_valid_sparse_tensor_accepted(self):
        """Baseline: Valid sparse tensors should load successfully."""
        io_handler = AudioEmbeddingMediaIO()

        valid_sparse = _create_valid_sparse_tensor()
# ... omitted for brevity ...
    def test_load_file_numpy_tensor_accepted(self, tmp_path):
        """numpy .npy files should load correctly via load_file."""

        io_handler = ImageEmbeddingMediaIO()

        arr = np.array([[1.5, 2.5], [3.5, 4.5]], dtype=np.float32)
        npy_path = tmp_path / "image_embeds.npy"
        np.save(npy_path, arr)

        result = io_handler.load_file(npy_path)
        assert isinstance(result, torch.Tensor)
        assert result.shape == torch.Size([2, 2])
        assert result.dtype == torch.float32
        assert torch.allclose(result, torch.from_numpy(arr))
```
**EN:** Groups related scenarios for Testimageembedsvalidation. The class contains 7 test method(s).
**CN:** 该类把与 Testimageembedsvalidation 相关的场景组织在一起。 其中包含 7 个测试方法。

### Class: TestAudioEmbedsValidation (lines 303-352)
```python
class TestAudioEmbedsValidation:
    """Test sparse tensor validation in audio embeddings (Chat API)."""

    def test_valid_dense_tensor_accepted(self):
        """Baseline: Valid dense tensors should work normally."""
        io_handler = AudioEmbeddingMediaIO()

        valid_tensor = _create_valid_dense_tensor()
        encoded = _encode_tensor(valid_tensor)

        # Should not raise any exception
        result = io_handler.load_base64("", encoded.decode("utf-8"))
        assert result.shape == valid_tensor.shape

    def test_valid_sparse_tensor_accepted(self):
        """Baseline: Valid sparse tensors should be converted successfully."""
        io_handler = AudioEmbeddingMediaIO()

        valid_sparse = _create_valid_sparse_tensor()
# ... omitted for brevity ...
        error_msg = str(exc_info.value).lower()
        assert "sparse" in error_msg or "index" in error_msg or "bounds" in error_msg

    def test_load_bytes_validates(self):
        """Security: Validation should also work for load_bytes method."""
        io_handler = AudioEmbeddingMediaIO()

        malicious_tensor = _create_malicious_sparse_tensor()
        buffer = io.BytesIO()
        torch.save(malicious_tensor, buffer)
        buffer.seek(0)

        with pytest.raises((RuntimeError, ValueError)):
            io_handler.load_bytes(buffer.read())
```
**EN:** Groups related scenarios for Testaudioembedsvalidation. The class contains 4 test method(s).
**CN:** 该类把与 Testaudioembedsvalidation 相关的场景组织在一起。 其中包含 4 个测试方法。

### Class: TestSparseTensorValidationIntegration (lines 355-399)
```python
class TestSparseTensorValidationIntegration:
    """
    These tests verify the complete attack chain is blocked at all entry points.
    """

    def test_attack_scenario_completions_api(self, model_config):
        """
        Simulate a complete attack through the Completions API.

        Attack scenario:
        1. Attacker crafts malicious sparse tensor
        2. Encodes it as base64
        3. Sends to /v1/completions with prompt_embeds parameter
        4. Server should reject before memory corruption occurs
        """
        # Step 1-2: Attacker creates malicious payload
        attack_payload = _encode_tensor(_create_malicious_sparse_tensor())

        # Step 3-4: Server processes and should reject
# ... omitted for brevity ...
        with pytest.raises((RuntimeError, ValueError)):
            io_handler.load_base64("", attack_payload.decode("utf-8"))

    def test_attack_scenario_chat_api_audio(self):
        """
        Simulate attack through Chat API with audio_embeds.

        Verifies the audio embeddings path is protected.
        """
        io_handler = AudioEmbeddingMediaIO()
        attack_payload = _encode_tensor(_create_malicious_sparse_tensor())

        with pytest.raises((RuntimeError, ValueError)):
            io_handler.load_base64("", attack_payload.decode("utf-8"))
```
**EN:** Groups related scenarios for Testsparsetensorvalidationintegration. The class contains 3 test method(s).
**CN:** 该类把与 Testsparsetensorvalidationintegration 相关的场景组织在一起。 其中包含 3 个测试方法。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **Mocking and patching / 模拟与补丁**
  - **EN:** Several scenarios replace dependencies or environment state so the tests can isolate one behavior at a time.
  - **CN:** 多个场景会替换依赖或环境状态，从而把验证范围限定在单一行为上。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `io`
- **Third-party / 第三方依赖**: `numpy`, `pybase64`, `pytest`, `torch`
- **vLLM internal / vLLM 内部依赖**: `vllm.exceptions`, `vllm.multimodal.media`, `vllm.renderers.embed_utils`, `vllm.config`
