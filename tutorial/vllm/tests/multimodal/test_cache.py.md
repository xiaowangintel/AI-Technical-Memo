# test_cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/multimodal/test_cache.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Cache behavior in the Multimodal test area through focused pytest scenarios. It focuses on scenarios such as Dummy Elem, Dummy Item, Dummy Items. / 该文件在 Multimodal 测试域中，通过有针对性的 pytest 场景验证 Cache 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-35)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import multiprocessing as mp

import numpy as np
import pytest
import torch

from vllm.config import ModelConfig, ParallelConfig, VllmConfig
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.cache import (
    BaseMultiModalProcessorCache,
    BaseMultiModalReceiverCache,
    MultiModalCache,
    MultiModalProcessorCacheInItem,
    MultiModalProcessorCacheItem,
    MultiModalProcessorCacheItemMetadata,
    MultiModalProcessorSenderCache,
    MultiModalReceiverCache,
# ... omitted for brevity ...
)
from vllm.multimodal.hasher import MultiModalHasher
from vllm.multimodal.inputs import (
    MultiModalFeatureSpec,
    MultiModalFieldElem,
    MultiModalKwargsItem,
    MultiModalKwargsItems,
    MultiModalSharedField,
    PlaceholderRange,
)
from vllm.multimodal.processing import PromptInsertion
from vllm.utils.mem_constants import GiB_bytes, MiB_bytes

pytestmark = pytest.mark.cpu_test
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `multiprocessing`, `numpy`, `pytest`, `vllm.config`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: _dummy_elem (lines 38-51)
```python
def _dummy_elem(
    size: int,
    *,
    rng: np.random.RandomState | None = None,
):
    if rng is None:
        data = torch.empty((size,), dtype=torch.int8)
    else:
        data = torch.from_numpy(rng.randint(4, size=(size,), dtype=np.int8))

    return MultiModalFieldElem(
        data=data,
        field=MultiModalSharedField(batch_size=1),
    )
```
**EN:** Implements a reusable helper for Dummy Elem, reducing duplication across related tests. It coordinates operations such as `MultiModalFieldElem`, `torch.empty`, `torch.from_numpy`.
**CN:** 该辅助函数为 Dummy Elem 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `MultiModalFieldElem`, `torch.empty`, `torch.from_numpy` 等操作。

### Helper: _dummy_item (lines 54-61)
```python
def _dummy_item(
    size_by_key: dict[str, int],
    *,
    rng: np.random.RandomState | None = None,
):
    return MultiModalKwargsItem(
        {key: _dummy_elem(size, rng=rng) for key, size in size_by_key.items()}
    )
```
**EN:** Implements a reusable helper for Dummy Item, reducing duplication across related tests. It coordinates operations such as `MultiModalKwargsItem`, `_dummy_elem`, `size_by_key.items`.
**CN:** 该辅助函数为 Dummy Item 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `MultiModalKwargsItem`, `_dummy_elem`, `size_by_key.items` 等操作。

### Test: test_cache_item_size (lines 77-100)
```python
@pytest.mark.parametrize(
    ("item", "expected_size"),
    [
        (_dummy_item({"a1": 100}), 100),
        (_dummy_item({"a1": 100, "a2": 110}), 210),
        (_dummy_items({"a": {"a1": 100, "a2": 110}, "b": {"b1": 120, "b2": 130}}), 460),  # noqa: E501
    ],
)
def test_cache_item_size(item, expected_size):
    cache = MultiModalCache.get_lru_cache(2048, type(item))

    cache[""] = item
    assert cache.currsize == expected_size

    prompt_update = PromptInsertion("dummy", "target", "insertion").resolve(0)

    cache[""] = MultiModalProcessorCacheItem(item, [prompt_update])
    assert cache.currsize == expected_size

    cache[""] = MultiModalProcessorCacheItemMetadata(item, [prompt_update])
    assert cache.currsize == expected_size

    cache[""] = item.get_data()
    assert cache.currsize == expected_size
```
**EN:** Checks Cache Item Size under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `MultiModalCache.get_lru_cache`, `PromptInsertion('dummy', 'target', 'insertion').resolve` before asserting the expected outcome.
**CN:** 该测试用例验证 Cache Item Size 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `MultiModalCache.get_lru_cache`, `PromptInsertion('dummy', 'target', 'insertion').resolve` 驱动目标逻辑，再断言预期结果。

### Test: test_ipc_enable_disable_consistency (lines 198-229)
```python
@pytest.mark.parametrize("is_cached_calls_per_iter", [1, 2, 3])
def test_ipc_enable_disable_consistency(is_cached_calls_per_iter):
    cache_size_gb = 1 / (1 << 20)

    vllm_config_ipc_enabled = _create_vllm_config(
        mm_processor_cache_gb=cache_size_gb,
        enable_ipc=True,
    )
    vllm_config_ipc_disabled = _create_vllm_config(
        mm_processor_cache_gb=0,
        enable_ipc=False,
    )
    vllm_config_cache_disabled = _create_vllm_config(
        mm_processor_cache_gb=cache_size_gb,
        enable_ipc=True,
    )

    _compare_caches(
        vllm_config_ipc_enabled,
        vllm_config_ipc_disabled,
        is_cached_calls_per_iter=is_cached_calls_per_iter,
    )
    _compare_caches(
        vllm_config_ipc_disabled,
        vllm_config_cache_disabled,
        is_cached_calls_per_iter=is_cached_calls_per_iter,
    )
    _compare_caches(
        vllm_config_cache_disabled,
        vllm_config_ipc_enabled,
        is_cached_calls_per_iter=is_cached_calls_per_iter,
    )
```
**EN:** Checks Ipc Enable Disable Consistency under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `_create_vllm_config`, `_compare_caches` before asserting the expected outcome.
**CN:** 该测试用例验证 Ipc Enable Disable Consistency 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `_create_vllm_config`, `_compare_caches` 驱动目标逻辑，再断言预期结果。

### Test: test_cache_eviction_lru_cache (lines 325-333)
```python
def test_cache_eviction_lru_cache():
    model_config = ModelConfig(
        model="llava-hf/llava-onevision-qwen2-0.5b-ov-hf",
        mm_processor_cache_gb=6 / GiB_bytes,
    )
    sender_cache = MultiModalProcessorSenderCache(model_config)
    receiver_cache = MultiModalReceiverCache(model_config)

    _run_test_cache_eviction_lru(sender_cache, receiver_cache, base_item_size=1)
```
**EN:** Checks Cache Eviction Lru Cache under a focused test scenario. The body exercises logic via `ModelConfig`, `MultiModalProcessorSenderCache`, `MultiModalReceiverCache` before asserting the expected outcome.
**CN:** 该测试用例验证 Cache Eviction Lru Cache 在特定场景下的行为。 函数体会先通过 `ModelConfig`, `MultiModalProcessorSenderCache`, `MultiModalReceiverCache` 驱动目标逻辑，再断言预期结果。

### Test: test_cache_eviction_shm_cache (lines 502-514)
```python
def test_cache_eviction_shm_cache():
    vllm_config = VllmConfig(
        model_config=ModelConfig(
            model="llava-hf/llava-onevision-qwen2-0.5b-ov-hf",
            mm_processor_cache_type="shm",
            mm_shm_cache_max_object_size_mb=6,
            mm_processor_cache_gb=15.2 * MiB_bytes / GiB_bytes,
        ),
    )
    sender_cache = ShmObjectStoreSenderCache(vllm_config)
    receiver_cache = ShmObjectStoreReceiverCache(vllm_config, mp.Lock())

    _run_test_cache_eviction_shm(sender_cache, receiver_cache, base_item_size=MiB_bytes)
```
**EN:** Checks Cache Eviction Shm Cache under a focused test scenario. The body exercises logic via `VllmConfig`, `ShmObjectStoreSenderCache`, `ShmObjectStoreReceiverCache` before asserting the expected outcome.
**CN:** 该测试用例验证 Cache Eviction Shm Cache 在特定场景下的行为。 函数体会先通过 `VllmConfig`, `ShmObjectStoreSenderCache`, `ShmObjectStoreReceiverCache` 驱动目标逻辑，再断言预期结果。

### Test: test_processor_cache_shared_across_loras (lines 517-551)
```python
def test_processor_cache_shared_across_loras():
    """Test that processor cache uses mm_hash to share data across LoRAs."""
    model_config = ModelConfig(
        model="llava-hf/llava-onevision-qwen2-0.5b-ov-hf",
        mm_processor_cache_gb=1,
    )
    receiver_cache = MultiModalReceiverCache(model_config)

    base_mm_hash = "image_hash_abc123"
    lora_a_identifier = f"12345:{base_mm_hash}"
    lora_b_identifier = f"67890:{base_mm_hash}"

    item_data = MultiModalKwargsItem.dummy(1024)

    feature_lora_a = MultiModalFeatureSpec(
        data=item_data,
        modality="image",
        identifier=lora_a_identifier,
        mm_position=PlaceholderRange(offset=0, length=100),
# ... omitted for brevity ...

    receiver_cache.get_and_update_features([feature_lora_a])
    assert base_mm_hash in receiver_cache._cache

    feature_lora_b = MultiModalFeatureSpec(
        data=None,
        modality="image",
        identifier=lora_b_identifier,
        mm_position=PlaceholderRange(offset=0, length=100),
        mm_hash=base_mm_hash,
    )

    receiver_cache.get_and_update_features([feature_lora_b])
    assert feature_lora_b.data == item_data
```
**EN:** Test that processor cache uses mm_hash to share data across LoRAs. The body exercises logic via `ModelConfig`, `MultiModalReceiverCache`, `MultiModalKwargsItem.dummy` before asserting the expected outcome.
**CN:** 该测试用例验证 Processor Cache Shared Across Loras 在特定场景下的行为。 函数体会先通过 `ModelConfig`, `MultiModalReceiverCache`, `MultiModalKwargsItem.dummy` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
_dummy_items
_create_vllm_config
_compare_caches
_run_test_cache_eviction_lru
_run_test_cache_eviction_shm
```
**EN:** The file contains more helper or test blocks than can be shown compactly here. This summary lists additional scenarios that continue the same validation strategy.
**CN:** 该文件包含的辅助逻辑或测试场景较多，无法在此完整展开；这里列出其余场景名称，便于快速把握覆盖范围。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Multimodal inputs / 多模态输入**
  - **EN:** The tests combine text with image, audio, or video data and verify the modality-specific pipeline.
  - **CN:** 这些测试把文本与图像、音频或视频数据结合起来，验证模态专用处理流程。
- **Quantization / 量化**
  - **EN:** The file covers low-precision model paths, kernels, or configuration rules for quantized inference.
  - **CN:** 该文件覆盖低精度模型路径、相关内核或量化推理配置规则。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `multiprocessing`
- **Third-party / 第三方依赖**: `numpy`, `pytest`, `torch`
- **vLLM internal / vLLM 内部依赖**: `vllm.config`, `vllm.multimodal`, `vllm.multimodal.cache`, `vllm.multimodal.hasher`, `vllm.multimodal.inputs`, `vllm.multimodal.processing`, `vllm.utils.mem_constants`
