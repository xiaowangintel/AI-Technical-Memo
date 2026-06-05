# test_ec_example_connector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/ec_connector/unit/test_ec_example_connector.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Unit tests for ECExampleConnector. / 该文件的文档字符串表明其用途：`unit tests for ecexampleconnector`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 3-5)
```python
"""
Unit tests for ECExampleConnector.
"""
```
**EN:** Module docstring that declares the scope of the file: Unit tests for ECExampleConnector.
**CN:** 模块文档字符串直接说明了文件范围：`unit tests for ecexampleconnector`。

### Imports and setup / 导入与设置 (lines 7-22)
```python
import os
from unittest.mock import Mock, patch

import pytest
import safetensors
import torch

from vllm.config import VllmConfig
from vllm.distributed.ec_transfer.ec_connector.base import ECConnectorRole
from vllm.distributed.ec_transfer.ec_connector.example_connector import (
    ECExampleConnector,
    ECExampleConnectorMetadata,
    MMMeta,
)
from vllm.multimodal.inputs import MultiModalFeatureSpec, PlaceholderRange
from vllm.v1.core.sched.output import SchedulerOutput
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, safetensors, torch`. vLLM modules under test include `vllm.config, vllm.distributed.ec_transfer.ec_connector.base, vllm.distributed.ec_transfer.ec_connector.example_connector, vllm.multimodal.inputs, vllm.v1.core.sched.output`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, safetensors, torch`。 被测试的 vLLM 模块包括 `vllm.config, vllm.distributed.ec_transfer.ec_connector.base, vllm.distributed.ec_transfer.ec_connector.example_connector, vllm.multimodal.inputs, vllm.v1.core.sched.output`。

### MockRequest (lines 26-43)
```python
class MockRequest:
    def __init__(self, request_id, mm_hashes: list[str], token_counts: list[int]):
        assert len(mm_hashes) == len(token_counts)
        self.request_id = request_id
        self._token_counts = token_counts
        self.mm_features = []
        for i, mm_hash in enumerate(mm_hashes):
            feature = MultiModalFeatureSpec(
                data=None,
                modality="image",
                identifier=mm_hash,
                mm_position=PlaceholderRange(offset=0, length=self._token_counts[i]),
            )
            self.mm_features.append(feature)

    def get_num_encoder_embeds(self, input_id: int) -> int:
        assert input_id < len(self._token_counts)
        return self._token_counts[input_id]
```
**EN:** Class `MockRequest` groups 0 test method(s) and 2 helper/fixture method(s).
**CN:** 类 `MockRequest` 组织了 0 个测试方法，以及 2 个辅助或 fixture 方法。

### temp_storage (lines 47-49)
```python
def temp_storage(tmp_path):
    """Fixture providing temporary storage path."""
    return str(tmp_path)
```
**EN:** Fixture/helper `temp_storage` prepares reusable state for downstream tests. Inputs: `tmp_path`. Key calls include `str`.
**CN:** `temp_storage` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`tmp_path`。 关键调用包括 `str`。

### mock_vllm_config_producer (lines 53-59)
```python
def mock_vllm_config_producer(temp_storage):
    """Fixture providing mock VllmConfig for producer role."""
    config = Mock(spec=VllmConfig)
    config.ec_transfer_config = Mock()
    config.ec_transfer_config.get_from_extra_config = Mock(return_value=temp_storage)
    config.ec_transfer_config.is_ec_producer = True
    return config
```
**EN:** Fixture/helper `mock_vllm_config_producer` prepares reusable state for downstream tests. Inputs: `temp_storage`. Key calls include `Mock`.
**CN:** `mock_vllm_config_producer` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`temp_storage`。 关键调用包括 `Mock`。

### mock_vllm_config_consumer (lines 63-69)
```python
def mock_vllm_config_consumer(temp_storage):
    """Fixture providing mock VllmConfig for consumer role."""
    config = Mock(spec=VllmConfig)
    config.ec_transfer_config = Mock()
    config.ec_transfer_config.get_from_extra_config = Mock(return_value=temp_storage)
    config.ec_transfer_config.is_ec_producer = False
    return config
```
**EN:** Fixture/helper `mock_vllm_config_consumer` prepares reusable state for downstream tests. Inputs: `temp_storage`. Key calls include `Mock`.
**CN:** `mock_vllm_config_consumer` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`temp_storage`。 关键调用包括 `Mock`。

### mock_request_with_3_mm (lines 73-80)
```python
def mock_request_with_3_mm():
    """Fixture providing mock Request with 3 multimodal items."""
    request_id = "test_req_123"
    mm_hashes = ["img_hash_1", "img_hash_2", "img_hash_3"]
    token_counts = [100, 150, 200]

    request = MockRequest(request_id, mm_hashes, token_counts)
    return request
```
**EN:** Fixture/helper `mock_request_with_3_mm` prepares reusable state for downstream tests. Key calls include `MockRequest`.
**CN:** `mock_request_with_3_mm` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `MockRequest`。

### TestECExampleConnectorBasics (lines 84-122)
```python
class TestECExampleConnectorBasics:
    """Test basic EC connector functionality."""

    def test_initialization_producer(self, mock_vllm_config_producer, temp_storage):
        """Test connector initializes correctly as producer."""
        connector = ECExampleConnector(
            vllm_config=mock_vllm_config_producer,
            role=ECConnectorRole.SCHEDULER,
        )
        assert connector.role == ECConnectorRole.SCHEDULER
        assert connector.is_producer
        assert connector._storage_path == temp_storage
        assert connector._mm_datas_need_loads == {}
    def test_initialization_consumer(self, mock_vllm_config_consumer, temp_storage):
        """Test connector initializes correctly as consumer."""
    # ... excerpt omitted for brevity ...
        assert connector.role == ECConnectorRole.WORKER
        assert not connector.is_producer
            role=ECConnectorRole.WORKER,
        assert scheduler_connector.role == ECConnectorRole.SCHEDULER
        assert worker_connector.role == ECConnectorRole.WORKER
```
**EN:** Class `TestECExampleConnectorBasics` groups 3 test method(s). Representative scenarios: `test_initialization_producer, test_initialization_consumer, test_role_assignment`.
**CN:** 类 `TestECExampleConnectorBasics` 组织了 3 个测试方法。 代表性场景：`test_initialization_producer, test_initialization_consumer, test_role_assignment`。

### TestCacheExistence (lines 125-218)
```python
class TestCacheExistence:
    """Test cache existence checking using has_cache_item() API."""

    def test_has_cache_item_all_exist_3_items(
        self,
        mock_vllm_config_producer,
        mock_vllm_config_consumer,
        mock_request_with_3_mm,
    ):
        """Test has_cache_item returns True when all 3 caches exist."""
        # Test for producer first
        producer = ECExampleConnector(
            vllm_config=mock_vllm_config_producer,
            role=ECConnectorRole.SCHEDULER,
        )
        # Create cache files using save_caches (proper way)
        encoder_cache: dict[str, torch.Tensor] = {}
    # ... excerpt omitted for brevity ...
        assert len(producer_result) == 3
        assert all(producer_result), f"Expected all True, got {producer_result}"
        assert len(consumer_result) == 3
        assert all(consumer_result), f"Expected all True, got {consumer_result}"
        assert len(result) == 3
        assert not any(result), f"Expected all False, got {result}"
        # Assert
        assert not result[0]  # First doesn't exist
        assert result[1]  # Second exists
        assert not result[2]  # Third doesn't exist
```
**EN:** Class `TestCacheExistence` groups 3 test method(s). Representative scenarios: `test_has_cache_item_all_exist_3_items, test_has_cache_item_none_exist, test_has_cache_item_partial_exist`.
**CN:** 类 `TestCacheExistence` 组织了 3 个测试方法。 代表性场景：`test_has_cache_item_all_exist_3_items, test_has_cache_item_none_exist, test_has_cache_item_partial_exist`。

### TestStateManagement (lines 221-318)
```python
class TestStateManagement:
    """Test connector state management."""

    def test_update_state_after_alloc_3_items(
        self, mock_vllm_config_producer, mock_request_with_3_mm
    ):
        """Test state update after allocation for 3 MM items."""
        connector = ECExampleConnector(
            vllm_config=mock_vllm_config_producer,
            role=ECConnectorRole.SCHEDULER,
        )
        # Initial state should be empty
        assert len(connector._mm_datas_need_loads) == 0
        # Update state for all 3 items (mock cache existence)
        with patch.object(connector, "has_cache_item", return_value=True):
            for i in range(3):
    # ... excerpt omitted for brevity ...
        assert len(connector._mm_datas_need_loads) == 3
        assert "img_hash_1" in connector._mm_datas_need_loads
        assert "img_hash_2" in connector._mm_datas_need_loads
        assert "img_hash_3" in connector._mm_datas_need_loads
        assert connector._mm_datas_need_loads["img_hash_1"] == 100
        assert connector._mm_datas_need_loads["img_hash_2"] == 150
        # State should be empty
        # Build again should return empty metadata
        metadata2 = connector.build_connector_meta(scheduler_output)
        assert len(metadata2.mm_datas) == 0
```
**EN:** Class `TestStateManagement` groups 4 test method(s). Representative scenarios: `test_update_state_after_alloc_3_items, test_build_connector_meta_3_items, test_build_connector_meta_empty, test_state_cleared_after_metadata_build`.
**CN:** 类 `TestStateManagement` 组织了 4 个测试方法。 代表性场景：`test_update_state_after_alloc_3_items, test_build_connector_meta_3_items, test_build_connector_meta_empty, test_state_cleared_after_metadata_build`。

### TestCacheSaving (lines 321-370)
```python
class TestCacheSaving:
    """Test encoder cache saving (producer only)."""

    def test_save_caches_producer_3_items(
        self, mock_vllm_config_producer, mock_request_with_3_mm, temp_storage
    ):
        """Test cache saving as producer for 3 different MM items."""
        connector = ECExampleConnector(
            vllm_config=mock_vllm_config_producer,
            role=ECConnectorRole.WORKER,
        )
        # Create and save 3 different caches
        mm_hashes = [f.identifier for f in mock_request_with_3_mm.mm_features]
        encoder_cache: dict[str, torch.Tensor] = {}
        for mm_hash in mm_hashes:
            encoder_cache[mm_hash] = torch.randn(10, 768)
    # ... excerpt omitted for brevity ...
        assert all(result), f"Not all caches were saved: {result}"
            assert "ec_cache" in loaded
            assert torch.allclose(loaded["ec_cache"], encoder_cache[mm_hash].cpu())
        # Save should not raise but also not create file
        connector.save_caches(encoder_cache, mm_hash)
        # Verify file doesn't exist using has_cache_item
        result = connector.has_cache_item(mm_hash)
        assert not result, "Consumer should not save caches"
```
**EN:** Class `TestCacheSaving` groups 2 test method(s). Representative scenarios: `test_save_caches_producer_3_items, test_save_caches_consumer_skips`.
**CN:** 类 `TestCacheSaving` 组织了 2 个测试方法。 代表性场景：`test_save_caches_producer_3_items, test_save_caches_consumer_skips`。

### TestCacheLoading (lines 373-478)
```python
class TestCacheLoading:
    """Test encoder cache loading (consumer)."""

    @pytest.mark.skipif(not torch.cuda.is_available(), reason="CUDA not available")
    def test_start_load_caches_consumer_3_items(
        self,
        mock_vllm_config_producer,
        mock_vllm_config_consumer,
        mock_request_with_3_mm,
        temp_storage,
    ):
        """Test consumer loads 3 caches from storage."""
        # First, create producer to save caches
        producer = ECExampleConnector(
            vllm_config=mock_vllm_config_producer,
            role=ECConnectorRole.WORKER,
        )
    # ... excerpt omitted for brevity ...
        assert len(encoder_cache) == 3
            assert mm_hash in encoder_cache, f"{mm_hash} missing in encoder_cache"
            assert encoder_cache[mm_hash].is_cuda, (
            assert torch.allclose(
        assert torch.equal(encoder_cache[mm_hash], existing_cache)
        # Load (should not raise)
        encoder_cache: dict[str, torch.Tensor] = {}
        consumer.start_load_caches(encoder_cache=encoder_cache)
        # Cache should remain empty
        assert len(encoder_cache) == 0
```
**EN:** Class `TestCacheLoading` groups 3 test method(s). Representative scenarios: `test_start_load_caches_consumer_3_items, test_start_load_caches_skip_existing, test_start_load_caches_empty_metadata`.
**CN:** 类 `TestCacheLoading` 组织了 3 个测试方法。 代表性场景：`test_start_load_caches_consumer_3_items, test_start_load_caches_skip_existing, test_start_load_caches_empty_metadata`。

### TestFilenameGeneration (lines 481-522)
```python
class TestFilenameGeneration:
    """Test filename and path generation."""

    def test_generate_foldername(self, mock_vllm_config_producer, temp_storage):
        """Test folder name generation."""
        connector = ECExampleConnector(
            vllm_config=mock_vllm_config_producer,
            role=ECConnectorRole.WORKER,
        )
        mm_hash = "test_folder_hash"
        folder = connector._generate_foldername_debug(mm_hash)
        assert folder == os.path.join(temp_storage, mm_hash)
        assert os.path.isdir(folder)  # Should be created
    def test_generate_filename(self, mock_vllm_config_producer, temp_storage):
        """Test filename generation."""
    # ... excerpt omitted for brevity ...
        expected = os.path.join(temp_storage, mm_hash, "encoder_cache.safetensors")
        assert filename == expected
        assert os.path.isdir(os.path.dirname(filename))  # Folder created
        mm_hash = "consistency_hash"
        filename1 = connector._generate_filename_debug(mm_hash)
        filename2 = connector._generate_filename_debug(mm_hash)
        assert filename1 == filename2
```
**EN:** Class `TestFilenameGeneration` groups 3 test method(s). Representative scenarios: `test_generate_foldername, test_generate_filename, test_generate_filename_consistency`.
**CN:** 类 `TestFilenameGeneration` 组织了 3 个测试方法。 代表性场景：`test_generate_foldername, test_generate_filename, test_generate_filename_consistency`。

### TestMetadataBindingLifecycle (lines 525-578)
```python
class TestMetadataBindingLifecycle:
    """Test metadata binding and clearing lifecycle."""

    def test_bind_connector_metadata(self, mock_vllm_config_consumer):
        """Test binding connector metadata."""
        connector = ECExampleConnector(
            vllm_config=mock_vllm_config_consumer,
            role=ECConnectorRole.WORKER,
        )
        metadata = ECExampleConnectorMetadata()
        metadata.add_mm_data(MMMeta.make_meta("hash_1", 100))
        connector.bind_connector_metadata(metadata)
        assert connector._connector_metadata is metadata
    def test_clear_connector_metadata(self, mock_vllm_config_consumer):
    # ... excerpt omitted for brevity ...
        assert connector._connector_metadata is None
        assert retrieved is metadata
        with pytest.raises(AssertionError):
            connector._get_connector_metadata()
```
**EN:** Class `TestMetadataBindingLifecycle` groups 4 test method(s). Representative scenarios: `test_bind_connector_metadata, test_clear_connector_metadata, test_get_connector_metadata, test_get_connector_metadata_not_set`.
**CN:** 类 `TestMetadataBindingLifecycle` 组织了 4 个测试方法。 代表性场景：`test_bind_connector_metadata, test_clear_connector_metadata, test_get_connector_metadata, test_get_connector_metadata_not_set`。

### TestEdgeCases (lines 581-623)
```python
class TestEdgeCases:
    """Test edge cases and error handling."""

    def test_save_empty_cache(self, mock_vllm_config_producer):
        """Test saving empty tensor."""
        connector = ECExampleConnector(
            vllm_config=mock_vllm_config_producer,
            role=ECConnectorRole.WORKER,
        )
        mm_hash = "empty_hash"
        encoder_cache = {mm_hash: torch.empty(0)}
        # Should not raise
        connector.save_caches(encoder_cache, mm_hash)
    def test_load_nonexistent_cache(self, mock_vllm_config_consumer):
        """Test loading cache that doesn't exist raises error."""
    # ... excerpt omitted for brevity ...
        with pytest.raises(FileNotFoundError):
            role=ECConnectorRole.SCHEDULER,
        result = connector.has_cache_item("nonexistent_hash")
        assert result is False
```
**EN:** Class `TestEdgeCases` groups 3 test method(s). Representative scenarios: `test_save_empty_cache, test_load_nonexistent_cache, test_has_cache_item_empty_request`.
**CN:** 类 `TestEdgeCases` 组织了 3 个测试方法。 代表性场景：`test_save_empty_cache, test_load_nonexistent_cache, test_has_cache_item_empty_request`。

## Key Concepts / 关键概念
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** Connector contracts and transfer paths
- **CN:** 连接器契约与传输路径
- **EN:** Output assembly and client-facing formatting
- **CN:** 输出拼装与面向客户端的格式化

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, safetensors, torch`.
- **CN:** 外部库：`pytest, safetensors, torch`。
- **EN:** vLLM modules under test: `vllm.config, vllm.distributed.ec_transfer.ec_connector.base, vllm.distributed.ec_transfer.ec_connector.example_connector, vllm.multimodal.inputs, vllm.v1.core.sched.output`.
- **CN:** 被测试的 vLLM 模块：`vllm.config, vllm.distributed.ec_transfer.ec_connector.base, vllm.distributed.ec_transfer.ec_connector.example_connector, vllm.multimodal.inputs, vllm.v1.core.sched.output`。
- **EN:** Standard-library support: `os, unittest.mock`.
- **CN:** 标准库支持：`os, unittest.mock`。
