# test_encoder_cache_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/core/test_encoder_cache_manager.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `encoder cache manager` behavior and regressions in the v1 stack. / 验证 v1 栈中 `encoder 缓存 manager` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-10)
```python
import pytest
import torch

from vllm.multimodal.inputs import MultiModalFeatureSpec, PlaceholderRange
from vllm.v1.core.encoder_cache_manager import (
    EncoderCacheManager,
    EncoderDecoderCacheManager,
)
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm.multimodal.inputs, vllm.v1.core.encoder_cache_manager`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm.multimodal.inputs, vllm.v1.core.encoder_cache_manager`。

### Module state / 模块级状态 (line 12)
```python
pytestmark = pytest.mark.cpu_test
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `pytestmark`. It also sets pytest marks that scope the whole file.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`pytestmark`。 该块还设置了作用于整个文件的 pytest 标记。

### MockRequest (lines 16-31)
```python
class MockRequest:
    def __init__(self, request_id, mm_hashes, token_counts):
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
        return self._token_counts[input_id]
```
**EN:** Class `MockRequest` groups 0 test method(s) and 2 helper/fixture method(s).
**CN:** 类 `MockRequest` 组织了 0 个测试方法，以及 2 个辅助或 fixture 方法。

### test_basic_allocate_and_reuse (lines 35-55)
```python
def test_basic_allocate_and_reuse():
    cache = EncoderCacheManager(cache_size=10)
    req = MockRequest("r1", ["imgA"], [4])

    assert not cache.check_and_update_cache(req, 0)
    assert cache.can_allocate(req, 0, int(1e9), 0)

    cache.allocate(req, 0)

    assert cache.check_and_update_cache(req, 0)
    assert "r1" in cache.cached["imgA"]
    assert cache.num_free_slots == 6

    # Free twice to bring refcount to 0.
    cache.free_encoder_input(req, 0)
    cache.free_encoder_input(req, 0)

    assert not cache.cached["imgA"]
    assert "imgA" in cache.freeable
    assert cache.num_freeable_slots == 10
    assert cache.num_free_slots == 6
```
**EN:** Test case covering `basic allocate and reuse`. It exercises `EncoderCacheManager, MockRequest, cache.can_allocate, cache.allocate, cache.check_and_update_cache, cache.free_encoder_input`. The body contains 9 explicit assertion(s).
**CN:** 该代码块是覆盖 `basic allocate and reuse` 的测试用例。 该测试会调用 `EncoderCacheManager, MockRequest, cache.can_allocate, cache.allocate, cache.check_and_update_cache, cache.free_encoder_input`。 代码主体包含 9 个显式断言。

### test_freeing_decreases_refcount_and_moves_to_freeable (lines 58-71)
```python
def test_freeing_decreases_refcount_and_moves_to_freeable():
    manager = EncoderCacheManager(cache_size=10)
    req = MockRequest("req2", ["img3"], [5])

    assert manager.can_allocate(req, 0, int(1e9), 0)
    manager.allocate(req, 0)

    assert len(manager.cached["img3"]) == 1

    manager.free_encoder_input(req, 0)

    assert not manager.cached["img3"]
    assert "img3" in manager.freeable
    assert manager.num_freeable_slots == 10
```
**EN:** Test case covering `freeing decreases refcount and moves to freeable`. It exercises `EncoderCacheManager, MockRequest, manager.can_allocate, manager.allocate, manager.free_encoder_input, int`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `freeing decreases refcount and moves to freeable` 的测试用例。 该测试会调用 `EncoderCacheManager, MockRequest, manager.can_allocate, manager.allocate, manager.free_encoder_input, int`。 代码主体包含 5 个显式断言。

### test_free_request_frees_all_inputs (lines 74-93)
```python
def test_free_request_frees_all_inputs():
    manager = EncoderCacheManager(cache_size=10)
    req = MockRequest("req3", ["a", "b"], [2, 3])

    assert manager.can_allocate(req, 0, int(1e9), 0)
    manager.allocate(req, 0)

    assert manager.can_allocate(req, 1, int(1e9), 0)
    manager.allocate(req, 1)

    assert len(manager.cached["a"]) == 1
    assert len(manager.cached["b"]) == 1

    manager.free(req)

    assert not manager.cached["a"]
    assert not manager.cached["b"]
    assert "a" in manager.freeable
    assert "b" in manager.freeable
    assert manager.num_freeable_slots == 10
```
**EN:** Test case covering `free request frees all inputs`. It exercises `EncoderCacheManager, MockRequest, manager.can_allocate, manager.allocate, manager.free, int`. The body contains 9 explicit assertion(s).
**CN:** 该代码块是覆盖 `free request frees all inputs` 的测试用例。 该测试会调用 `EncoderCacheManager, MockRequest, manager.can_allocate, manager.allocate, manager.free, int`。 代码主体包含 9 个显式断言。

### test_eviction_when_cache_is_full (lines 96-111)
```python
def test_eviction_when_cache_is_full():
    manager = EncoderCacheManager(cache_size=10)

    req1 = MockRequest("req1", ["x"], [6])
    req2 = MockRequest("req2", ["y"], [5])

    assert manager.can_allocate(req1, 0, int(1e9), 0)
    manager.allocate(req1, 0)
    manager.free_encoder_input(req1, 0)

    assert manager.can_allocate(req2, 0, int(1e9), 0)
    manager.allocate(req2, 0)

    # 'x' should have been evicted.
    assert "x" not in manager.cached
    assert "x" in manager.get_freed_mm_hashes()
```
**EN:** Test case covering `eviction when cache is full`. It exercises `EncoderCacheManager, MockRequest, manager.can_allocate, manager.allocate, manager.free_encoder_input, int`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `eviction when 缓存 is full` 的测试用例。 该测试会调用 `EncoderCacheManager, MockRequest, manager.can_allocate, manager.allocate, manager.free_encoder_input, int`。 代码主体包含 4 个显式断言。

### test_get_cached_input_ids (lines 114-125)
```python
def test_get_cached_input_ids():
    manager = EncoderCacheManager(cache_size=10)
    req = MockRequest("reqX", ["m", "n", "o"], [2, 4, 3])

    assert manager.can_allocate(req, 0, int(1e9), 0)
    manager.allocate(req, 0)

    assert manager.can_allocate(req, 2, int(1e9), 0)
    manager.allocate(req, 2)

    cached_ids = manager.get_cached_input_ids(req)
    assert cached_ids == {0, 2}
```
**EN:** Test case covering `get cached input ids`. It exercises `EncoderCacheManager, MockRequest, manager.can_allocate, manager.allocate, manager.get_cached_input_ids, int`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `get cached input ids` 的测试用例。 该测试会调用 `EncoderCacheManager, MockRequest, manager.can_allocate, manager.allocate, manager.get_cached_input_ids, int`。 代码主体包含 3 个显式断言。

### test_has_cache_restores_from_freeable (lines 128-141)
```python
def test_has_cache_restores_from_freeable():
    manager = EncoderCacheManager(cache_size=10)
    req = MockRequest("reqY", ["imgZ"], [4])

    assert manager.can_allocate(req, 0, int(1e9), 0)
    manager.allocate(req, 0)

    manager.free_encoder_input(req, 0)

    # Should restore from freeable.
    assert manager.check_and_update_cache(req, 0)
    assert len(manager.cached["imgZ"]) == 1
    assert "imgZ" not in manager.freeable
    assert manager.num_freeable_slots == 6
```
**EN:** Test case covering `has cache restores from freeable`. It exercises `EncoderCacheManager, MockRequest, manager.can_allocate, manager.allocate, manager.free_encoder_input, manager.check_and_update_cache`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `has 缓存 restores from freeable` 的测试用例。 该测试会调用 `EncoderCacheManager, MockRequest, manager.can_allocate, manager.allocate, manager.free_encoder_input, manager.check_and_update_cache`。 代码主体包含 5 个显式断言。

### test_get_freed_mm_hashes_clears_freed_list (lines 144-159)
```python
def test_get_freed_mm_hashes_clears_freed_list():
    manager = EncoderCacheManager(cache_size=10)
    req1 = MockRequest("reqA", ["a"], [5])
    req2 = MockRequest("reqB", ["b"], [6])

    assert manager.can_allocate(req1, 0, int(1e9), 0)
    manager.allocate(req1, 0)
    manager.free_encoder_input(req1, 0)

    # Should trigger eviction of 'a'.
    assert manager.can_allocate(req2, 0, int(1e9), 0)
    manager.allocate(req2, 0)

    freed = manager.get_freed_mm_hashes()
    assert "a" in freed
    assert manager.get_freed_mm_hashes() == []
```
**EN:** Test case covering `get freed mm hashes clears freed list`. It exercises `EncoderCacheManager, MockRequest, manager.can_allocate, manager.allocate, manager.free_encoder_input, manager.get_freed_mm_hashes`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `get freed mm hashes clears freed list` 的测试用例。 该测试会调用 `EncoderCacheManager, MockRequest, manager.can_allocate, manager.allocate, manager.free_encoder_input, manager.get_freed_mm_hashes`。 代码主体包含 4 个显式断言。

### test_schedule_request_multi_images_respect_space_limit (lines 162-172)
```python
def test_schedule_request_multi_images_respect_space_limit():
    manager = EncoderCacheManager(cache_size=10)
    req = MockRequest("reqA", ["a", "b"], [5, 6])
    compute_budget = 100

    num_tokens_to_schedule = 0
    assert manager.can_allocate(req, 0, compute_budget, num_tokens_to_schedule)
    num_tokens_to_schedule += req.get_num_encoder_embeds(0)
    compute_budget -= req.get_num_encoder_embeds(0)

    assert not manager.can_allocate(req, 1, compute_budget, num_tokens_to_schedule)
```
**EN:** Test case covering `schedule request multi images respect space limit`. It exercises `EncoderCacheManager, MockRequest, manager.can_allocate, req.get_num_encoder_embeds`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `schedule request multi images respect space limit` 的测试用例。 该测试会调用 `EncoderCacheManager, MockRequest, manager.can_allocate, req.get_num_encoder_embeds`。 代码主体包含 2 个显式断言。

### test_schedule_request_multi_images_respect_compute_limit (lines 175-184)
```python
def test_schedule_request_multi_images_respect_compute_limit():
    manager = EncoderCacheManager(cache_size=100)
    req = MockRequest("reqA", ["a", "b"], [5, 6])
    compute_budget = 10
    num_tokens_to_schedule = 0
    assert manager.can_allocate(req, 0, compute_budget, num_tokens_to_schedule)
    num_tokens_to_schedule += req.get_num_encoder_embeds(0)
    compute_budget -= req.get_num_encoder_embeds(0)

    assert not manager.can_allocate(req, 1, compute_budget, num_tokens_to_schedule)
```
**EN:** Test case covering `schedule request multi images respect compute limit`. It exercises `EncoderCacheManager, MockRequest, manager.can_allocate, req.get_num_encoder_embeds`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `schedule request multi images respect compute limit` 的测试用例。 该测试会调用 `EncoderCacheManager, MockRequest, manager.can_allocate, req.get_num_encoder_embeds`。 代码主体包含 2 个显式断言。

### test_encoder_cache_with_is_embed_mask (lines 187-213)
```python
def test_encoder_cache_with_is_embed_mask():
    class MockRequestWithMask(MockRequest):
        def get_num_encoder_embeds(self, input_id: int) -> int:
            return self.mm_features[input_id].mm_position.get_num_embeds()

    is_embed = torch.zeros(100, dtype=torch.bool)
    is_embed[torch.tensor([5, 15, 25, 35, 45, 55, 65, 75])] = True

    request = MockRequestWithMask("r1", ["img1"], [100])
    request.mm_features[0] = MultiModalFeatureSpec(
        data=None,
        modality="image",
        identifier="img1",
        mm_position=PlaceholderRange(offset=0, length=100, is_embed=is_embed),
    )

    manager = EncoderCacheManager(cache_size=100)
    manager.allocate(request, 0)

    assert manager.num_free_slots == 92
    assert "img1" in manager.cached

    old_size = 100
    new_size = request.mm_features[0].mm_position.get_num_embeds()
    assert new_size == 8
    savings_ratio = old_size / new_size
    assert savings_ratio == 12.5
```
**EN:** Test case covering `encoder cache with is embed mask`. It exercises `torch.zeros, MockRequestWithMask, MultiModalFeatureSpec, EncoderCacheManager, manager.allocate, mm_position.get_num_embeds`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `encoder 缓存 with is embed mask` 的测试用例。 该测试会调用 `torch.zeros, MockRequestWithMask, MultiModalFeatureSpec, EncoderCacheManager, manager.allocate, mm_position.get_num_embeds`。 代码主体包含 4 个显式断言。

### test_encoder_cache_mask_based_retrieval (lines 216-252)
```python
def test_encoder_cache_mask_based_retrieval():
    class MockRequestWithMask(MockRequest):
        def get_num_encoder_embeds(self, input_id: int) -> int:
            return self.mm_features[input_id].mm_position.get_num_embeds()

    is_embed = torch.tensor(
        [False, False, True, True, False, True, True, True, False, False]
    )
    request = MockRequestWithMask("r1", ["img1"], [10])
    request.mm_features[0] = MultiModalFeatureSpec(
        data=None,
        modality="image",
        identifier="img1",
        mm_position=PlaceholderRange(offset=0, length=10, is_embed=is_embed),
    manager = EncoderCacheManager(cache_size=50)
    # ... excerpt omitted for brevity ...
    assert request.mm_features[0].mm_position.get_num_embeds() == 5
    assert num_embeds_before == 0
    assert num_embeds_in_range == 5
    end_idx = 5
    num_embeds_before = is_embed[:start_idx].sum().item() if start_idx > 0 else 0
    num_embeds_in_range = is_embed[start_idx:end_idx].sum().item()
    assert num_embeds_in_range == 2
```
**EN:** Test case covering `encoder cache mask based retrieval`. It exercises `torch.tensor, MockRequestWithMask, MultiModalFeatureSpec, EncoderCacheManager, manager.allocate, sum.item`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `encoder 缓存 mask based retrieval` 的测试用例。 该测试会调用 `torch.tensor, MockRequestWithMask, MultiModalFeatureSpec, EncoderCacheManager, manager.allocate, sum.item`。 代码主体包含 5 个显式断言。

### test_reset_clears_all_state (lines 255-282)
```python
def test_reset_clears_all_state():
    """Test that reset() clears all cached entries and restores capacity."""
    manager = EncoderCacheManager(cache_size=20)

    req1 = MockRequest("req1", ["img1", "img2"], [5, 3])
    req2 = MockRequest("req2", ["img3"], [4])

    manager.allocate(req1, 0)
    manager.allocate(req1, 1)
    manager.allocate(req2, 0)
    manager.free_encoder_input(req1, 0)

    req3 = MockRequest("req3", ["img4"], [10])
    manager.free_encoder_input(req1, 1)
    manager.free_encoder_input(req2, 0)
    manager.can_allocate(req3, 0, int(1e9), 0)
    manager.allocate(req3, 0)

    assert len(manager.cached) > 0
    assert manager.num_free_slots < 20

    manager.reset()

    assert len(manager.cached) == 0
    assert len(manager.freeable) == 0
    assert len(manager.freed) == 0
    assert manager.num_free_slots == 20
    assert manager.num_freeable_slots == 20
```
**EN:** Test case covering `reset clears all state`. It exercises `EncoderCacheManager, MockRequest, manager.allocate, manager.free_encoder_input, manager.can_allocate, manager.reset`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `reset clears all state` 的测试用例。 该测试会调用 `EncoderCacheManager, MockRequest, manager.allocate, manager.free_encoder_input, manager.can_allocate, manager.reset`。 代码主体包含 7 个显式断言。

### test_reset_allows_fresh_allocations (lines 285-300)
```python
def test_reset_allows_fresh_allocations():
    manager = EncoderCacheManager(cache_size=10)

    req1 = MockRequest("req1", ["img1"], [10])
    manager.allocate(req1, 0)
    assert manager.num_free_slots == 0

    manager.reset()

    req2 = MockRequest("req2", ["img2"], [8])
    assert manager.can_allocate(req2, 0, int(1e9), 0)
    manager.allocate(req2, 0)

    assert manager.num_free_slots == 2
    assert "img2" in manager.cached
    assert "img1" not in manager.cached
```
**EN:** Test case covering `reset allows fresh allocations`. It exercises `EncoderCacheManager, MockRequest, manager.allocate, manager.reset, manager.can_allocate, int`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `reset allows fresh allocations` 的测试用例。 该测试会调用 `EncoderCacheManager, MockRequest, manager.allocate, manager.reset, manager.can_allocate, int`。 代码主体包含 5 个显式断言。

### test_encoder_decoder_cache_manager_reset (lines 303-320)
```python
def test_encoder_decoder_cache_manager_reset():
    manager = EncoderDecoderCacheManager(cache_size=20)

    req1 = MockRequest("req1", ["img1"], [5])
    req2 = MockRequest("req2", ["img2"], [3])

    manager.allocate(req1, 0)
    manager.allocate(req2, 0)
    manager.free(req1)
    manager.get_freed_mm_hashes()

    assert manager.num_free_slots < 20

    manager.reset()

    assert len(manager.allocated) == 0
    assert len(manager.to_free) == 0
    assert manager.num_free_slots == 20
```
**EN:** Test case covering `encoder decoder cache manager reset`. It exercises `EncoderDecoderCacheManager, MockRequest, manager.allocate, manager.free, manager.get_freed_mm_hashes, manager.reset`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `encoder decoder 缓存 manager reset` 的测试用例。 该测试会调用 `EncoderDecoderCacheManager, MockRequest, manager.allocate, manager.free, manager.get_freed_mm_hashes, manager.reset`。 代码主体包含 4 个显式断言。

### test_encoder_decoder_cache_manager_reset_allows_fresh_allocations (lines 323-337)
```python
def test_encoder_decoder_cache_manager_reset_allows_fresh_allocations():
    manager = EncoderDecoderCacheManager(cache_size=10)

    req1 = MockRequest("req1", ["img1"], [10])
    manager.allocate(req1, 0)
    assert manager.num_free_slots == 0

    manager.reset()

    req2 = MockRequest("req2", ["img2"], [8])
    assert manager.can_allocate(req2, 0, int(1e9), 0)
    manager.allocate(req2, 0)

    assert manager.num_free_slots == 2
    assert "img2" in manager.allocated
```
**EN:** Test case covering `encoder decoder cache manager reset allows fresh allocations`. It exercises `EncoderDecoderCacheManager, MockRequest, manager.allocate, manager.reset, manager.can_allocate, int`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `encoder decoder 缓存 manager reset allows fresh allocations` 的测试用例。 该测试会调用 `EncoderDecoderCacheManager, MockRequest, manager.allocate, manager.reset, manager.can_allocate, int`。 代码主体包含 4 个显式断言。

## Key Concepts / 关键概念
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** Multimodal input bookkeeping
- **CN:** 多模态输入管理
- **EN:** Speculative execution or decoding paths
- **CN:** 推测执行或解码路径

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm.multimodal.inputs, vllm.v1.core.encoder_cache_manager`.
- **CN:** 被测试的 vLLM 模块：`vllm.multimodal.inputs, vllm.v1.core.encoder_cache_manager`。
