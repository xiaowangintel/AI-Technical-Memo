# test_kv_cache_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/core/test_kv_cache_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `KV cache utils` behavior and regressions in the v1 stack. / 验证 v1 栈中 `KV 缓存 utils` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-58)
```python
import hashlib
import importlib
from collections.abc import Callable
from typing import Any

import pytest
import torch
import vllm.v1.core.kv_cache_utils as kv_cache_utils
from vllm.config import ModelConfig, SchedulerConfig, VllmConfig
from vllm.config.kv_events import KVEventsConfig
from vllm.lora.request import LoRARequest
from vllm.multimodal.inputs import (
    MultiModalFeatureSpec,
    MultiModalKwargsItem,
    PlaceholderRange,
)
from vllm.sampling_params import SamplingParams
# ... excerpt omitted for brevity ...
    UniformTypeKVCacheSpecs,
    get_kv_cache_spec_kind,
    get_kv_cache_spec_sliding_window,
from vllm.v1.metrics.stats import CachingMetrics, PrefixCacheStats
from vllm.v1.request import Request
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm.v1.core.kv_cache_utils, vllm.config, vllm.config.kv_events, vllm.lora.request, vllm.multimodal.inputs, ...`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm.v1.core.kv_cache_utils, vllm.config, vllm.config.kv_events, vllm.lora.request, vllm.multimodal.inputs, ...`。

### Module state / 模块级状态 (line 60)
```python
pytestmark = pytest.mark.cpu_test
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `pytestmark`. It also sets pytest marks that scope the whole file.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`pytestmark`。 该块还设置了作用于整个文件的 pytest 标记。

### _auto_init_hash_fn (lines 64-70)
```python
def _auto_init_hash_fn(request):
    hash_fn: Callable
    if "hash_fn" in request.fixturenames:
        hash_fn = request.getfixturevalue("hash_fn")
    else:
        hash_fn = sha256
    init_none_hash(hash_fn)
```
**EN:** Fixture/helper `_auto_init_hash_fn` prepares reusable state for downstream tests. Inputs: `request`. Key calls include `pytest.fixture, init_none_hash, request.getfixturevalue`.
**CN:** `_auto_init_hash_fn` 是为后续测试准备可复用状态的 fixture/辅助函数。 输入参数：`request`。 关键调用包括 `pytest.fixture, init_none_hash, request.getfixturevalue`。

### make_request (lines 73-108)
```python
def make_request(
    request_id: str,
    prompt_token_ids: list[int] | None,
    block_size: int = 3,
    hash_fn: Callable = hash,
    mm_positions: list[PlaceholderRange] | None = None,
    mm_hashes: list[str] | None = None,
    cache_salt: str | None = None,
    prompt_embeds: torch.Tensor | None = None,
):
    mm_features = []
    if mm_positions is not None:
        for j, position in enumerate(mm_positions):
            identifier = mm_hashes[j] if mm_hashes else f"hash_{j}"
            mm_feature = MultiModalFeatureSpec(
                data=MultiModalKwargsItem.dummy(),
                mm_position=position,
                identifier=identifier,
    # ... excerpt omitted for brevity ...
    return Request(
        pooling_params=None,
        lora_request=None,
        cache_salt=cache_salt,
        block_hasher=get_request_block_hasher(block_size, hash_fn),
        prompt_embeds=prompt_embeds,
    )
```
**EN:** Helper function `make_request` encapsulates reusable logic for `request`. Inputs: `request_id, prompt_token_ids, block_size, hash_fn, mm_positions, mm_hashes, cache_salt, prompt_embeds`. Key calls include `SamplingParams, sampling_params.update_from_generation_config, Request, enumerate, MultiModalFeatureSpec, mm_features.append`.
**CN:** 辅助函数 `make_request` 封装了与 `request` 相关的可复用逻辑。 输入参数：`request_id, prompt_token_ids, block_size, hash_fn, mm_positions, mm_hashes, cache_salt, prompt_embeds`。 关键调用包括 `SamplingParams, sampling_params.update_from_generation_config, Request, enumerate, MultiModalFeatureSpec, mm_features.append`。

### new_kv_cache_spec (lines 111-128)
```python
def new_kv_cache_spec(
    block_size=16,
    num_kv_heads=2,
    head_size=64,
    dtype=torch.float32,
    page_size_padded=None,
    sliding_window=None,
    attention_chunk_size=None,
):
    return FullAttentionSpec(
        block_size=block_size,
        num_kv_heads=num_kv_heads,
        head_size=head_size,
        dtype=dtype,
        page_size_padded=page_size_padded,
        sliding_window=sliding_window,
        attention_chunk_size=attention_chunk_size,
    )
```
**EN:** Helper function `new_kv_cache_spec` encapsulates reusable logic for `new KV cache spec`. Inputs: `block_size, num_kv_heads, head_size, dtype, page_size_padded, sliding_window, attention_chunk_size`. Key calls include `FullAttentionSpec`.
**CN:** 辅助函数 `new_kv_cache_spec` 封装了与 `new KV 缓存 spec` 相关的可复用逻辑。 输入参数：`block_size, num_kv_heads, head_size, dtype, page_size_padded, sliding_window, attention_chunk_size`。 关键调用包括 `FullAttentionSpec`。

### new_sliding_window_spec (lines 131-146)
```python
def new_sliding_window_spec(
    block_size=16,
    num_kv_heads=2,
    head_size=64,
    dtype=torch.float32,
    page_size_padded=None,
    sliding_window=1,
):
    return SlidingWindowSpec(
        block_size=block_size,
        num_kv_heads=num_kv_heads,
        head_size=head_size,
        dtype=dtype,
        page_size_padded=page_size_padded,
        sliding_window=sliding_window,
    )
```
**EN:** Helper function `new_sliding_window_spec` encapsulates reusable logic for `new sliding window spec`. Inputs: `block_size, num_kv_heads, head_size, dtype, page_size_padded, sliding_window`. Key calls include `SlidingWindowSpec`.
**CN:** 辅助函数 `new_sliding_window_spec` 封装了与 `new sliding window spec` 相关的可复用逻辑。 输入参数：`block_size, num_kv_heads, head_size, dtype, page_size_padded, sliding_window`。 关键调用包括 `SlidingWindowSpec`。

### new_chunked_local_attention_spec (lines 149-164)
```python
def new_chunked_local_attention_spec(
    block_size=16,
    num_kv_heads=2,
    head_size=64,
    dtype=torch.float32,
    page_size_padded=None,
    attention_chunk_size=4,
):
    return ChunkedLocalAttentionSpec(
        block_size=block_size,
        num_kv_heads=num_kv_heads,
        head_size=head_size,
        dtype=dtype,
        page_size_padded=page_size_padded,
        attention_chunk_size=attention_chunk_size,
    )
```
**EN:** Helper function `new_chunked_local_attention_spec` encapsulates reusable logic for `new chunked local attention spec`. Inputs: `block_size, num_kv_heads, head_size, dtype, page_size_padded, attention_chunk_size`. Key calls include `ChunkedLocalAttentionSpec`.
**CN:** 辅助函数 `new_chunked_local_attention_spec` 封装了与 `new chunked local 注意力 spec` 相关的可复用逻辑。 输入参数：`block_size, num_kv_heads, head_size, dtype, page_size_padded, attention_chunk_size`。 关键调用包括 `ChunkedLocalAttentionSpec`。

### new_mamba_spec (lines 167-182)
```python
def new_mamba_spec(
    block_size=16,
    shapes=((2, 512), (3, 32, 32)),
    dtypes=(torch.float32, torch.float32),
    num_speculative_blocks=2,
    mamba_cache_mode="none",
    page_size_padded=None,
):
    return MambaSpec(
        block_size=block_size,
        shapes=shapes,
        dtypes=dtypes,
        page_size_padded=page_size_padded,
        mamba_cache_mode=mamba_cache_mode,
        num_speculative_blocks=num_speculative_blocks,
    )
```
**EN:** Helper function `new_mamba_spec` encapsulates reusable logic for `newMamba spec`. Inputs: `block_size, shapes, dtypes, num_speculative_blocks, mamba_cache_mode, page_size_padded`. Key calls include `MambaSpec`.
**CN:** 辅助函数 `new_mamba_spec` 封装了与 `newmamba spec` 相关的可复用逻辑。 输入参数：`block_size, shapes, dtypes, num_speculative_blocks, mamba_cache_mode, page_size_padded`。 关键调用包括 `MambaSpec`。

### test_none_hash (lines 186-205)
```python
def test_none_hash(monkeypatch, hash_fn):
    import vllm.v1.core.kv_cache_utils

    # case 1: PYTHONHASHSEED is not set, use random
    with monkeypatch.context() as m:
        m.delenv("PYTHONHASHSEED", raising=False)
        reloaded_kv_cache_utils = importlib.reload(vllm.v1.core.kv_cache_utils)
        reloaded_kv_cache_utils.init_none_hash(hash_fn)
        assert reloaded_kv_cache_utils.NONE_HASH is not None
        assert isinstance(reloaded_kv_cache_utils.NONE_HASH, bytes)
        assert reloaded_kv_cache_utils.NONE_HASH != b""

    # case 2: PYTHONHASHSEED is set, use the seed and hash_fn
    with monkeypatch.context() as m:
        m.setenv("PYTHONHASHSEED", "python hash seed")
        reloaded_kv_cache_utils = importlib.reload(vllm.v1.core.kv_cache_utils)
        reloaded_kv_cache_utils.init_none_hash(hash_fn)
        assert reloaded_kv_cache_utils.NONE_HASH is not None
        assert isinstance(reloaded_kv_cache_utils.NONE_HASH, bytes)
        assert hash_fn("python hash seed") == reloaded_kv_cache_utils.NONE_HASH
```
**EN:** Parameterized test covering `none hash`. Parameter axes: `hash_fn`. Inputs/fixtures: `monkeypatch, hash_fn`. It exercises `mark.parametrize, monkeypatch.context, m.delenv, importlib.reload, reloaded_kv_cache_utils.init_none_hash, isinstance`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `none hash` 的测试用例。 参数维度：`hash_fn`。 输入或 fixture：`monkeypatch, hash_fn`。 该测试会调用 `mark.parametrize, monkeypatch.context, m.delenv, importlib.reload, reloaded_kv_cache_utils.init_none_hash, isinstance`。 代码主体包含 6 个显式断言。

### test_kv_cache_block (lines 208-227)
```python
def test_kv_cache_block():
    # Test KVCacheBlock initialization
    block = KVCacheBlock(block_id=0)
    assert block.block_id == 0
    assert block.ref_cnt == 0
    assert block.block_hash is None

    # Test reference count manipulation
    block.ref_cnt += 1
    assert block.ref_cnt == 1
    block.ref_cnt -= 1
    assert block.ref_cnt == 0

    # Test block hash setting and resetting
    block_hash = make_block_hash_with_group_id(BlockHash(b"abc"), 0)
    block.block_hash = block_hash
    assert block.block_hash == block_hash

    block.reset_hash()
    assert block.block_hash is None
```
**EN:** Test case covering `KV cache block`. It exercises `KVCacheBlock, make_block_hash_with_group_id, block.reset_hash, BlockHash`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `KV 缓存 block` 的测试用例。 该测试会调用 `KVCacheBlock, make_block_hash_with_group_id, block.reset_hash, BlockHash`。 代码主体包含 7 个显式断言。

### test_kv_cache_block_uses_slots (lines 230-239)
```python
def test_kv_cache_block_uses_slots():
    block = KVCacheBlock(block_id=0)

    # Slots eliminate per-instance __dict__, saving ~264 bytes per block.
    # At 100K+ blocks this avoids tens of MB of overhead and GC pressure.
    assert not hasattr(block, "__dict__")

    # Verify that slots actually prevent dynamic attribute assignment.
    with pytest.raises(AttributeError):
        block.unexpected_field = True
```
**EN:** Test case covering `KV cache block uses slots`. It exercises `KVCacheBlock, hasattr, pytest.raises`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `KV 缓存 block uses slots` 的测试用例。 该测试会调用 `KVCacheBlock, hasattr, pytest.raises`。 代码主体包含 1 个显式断言。

### test_free_kv_cache_block_queue_initialization (lines 242-248)
```python
def test_free_kv_cache_block_queue_initialization():
    # Test with a single block
    block = KVCacheBlock(block_id=0)
    queue = FreeKVCacheBlockQueue([block])
    assert queue.num_free_blocks == 1
    assert queue.fake_free_list_head.next_free_block is block
    assert queue.fake_free_list_tail.prev_free_block is block
```
**EN:** Test case covering `free KV cache block queue initialization`. It exercises `KVCacheBlock, FreeKVCacheBlockQueue`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `free KV 缓存 block queue initialization` 的测试用例。 该测试会调用 `KVCacheBlock, FreeKVCacheBlockQueue`。 代码主体包含 3 个显式断言。

### test_free_kv_cache_block_queue_operations (lines 251-294)
```python
def test_free_kv_cache_block_queue_operations():
    # Create a list of KVCacheBlock objects
    blocks = [KVCacheBlock(block_id=i) for i in range(5)]

    # Create a FreeKVCacheBlockQueue with these blocks
    queue = FreeKVCacheBlockQueue(blocks)
    # Check initial state
    assert queue.num_free_blocks == 5
    assert queue.fake_free_list_head.next_free_block is blocks[0]
    assert queue.fake_free_list_tail.prev_free_block is blocks[4]
    # Pop the first block
    block1 = queue.popleft()
    assert block1 == blocks[0]
    assert queue.num_free_blocks == 4
    assert queue.fake_free_list_head.next_free_block is blocks[1]
    # ... excerpt omitted for brevity ...
    assert queue.num_free_blocks == 3
    assert blocks[1].next_free_block is blocks[3]
    assert blocks[3].prev_free_block is blocks[1]
    assert queue.fake_free_list_tail.prev_free_block is block_to_remove
    assert block_to_remove.prev_free_block is blocks[4]
    assert queue.fake_free_list_tail.prev_free_block is queue.fake_free_list_head
    # Attempt to pop from an empty queue
    with pytest.raises(ValueError) as e:
        queue.popleft()
    assert str(e.value) == "No free blocks available"
```
**EN:** Test case covering `free KV cache block queue operations`. It exercises `FreeKVCacheBlockQueue, queue.popleft, queue.remove, queue.append, range, KVCacheBlock`. The body contains 18 explicit assertion(s).
**CN:** 该代码块是覆盖 `free KV 缓存 block queue operations` 的测试用例。 该测试会调用 `FreeKVCacheBlockQueue, queue.popleft, queue.remove, queue.append, range, KVCacheBlock`。 代码主体包含 18 个显式断言。

### test_free_kv_cache_block_queue_append_n (lines 297-358)
```python
def test_free_kv_cache_block_queue_append_n():
    # Create an empty FreeKVCacheBlockQueue with these blocks
    queue = FreeKVCacheBlockQueue([])
    blocks = [KVCacheBlock(block_id=i) for i in range(6)]
    # Append 0 block
    # fake_head->fake_tail
    queue.append_n([])
    assert queue.num_free_blocks == 0
    assert queue.fake_free_list_head.next_free_block is queue.fake_free_list_tail
    assert queue.fake_free_list_tail.prev_free_block is queue.fake_free_list_head
    # Append 1 block
    # fake_head->b0->fake_tail
    queue.append_n(blocks[0:1])
    assert queue.num_free_blocks == 1
    assert queue.fake_free_list_head.next_free_block is blocks[0]
    assert blocks[0].prev_free_block is queue.fake_free_list_head
    assert blocks[0].next_free_block is queue.fake_free_list_tail
    assert queue.fake_free_list_tail.prev_free_block is blocks[0]
    # ... excerpt omitted for brevity ...
    assert queue.num_free_blocks == 3
    assert blocks[0].next_free_block is blocks[4]
    assert blocks[4].prev_free_block is blocks[0]
    assert blocks[4].next_free_block is blocks[5]
        invalid_queue.append_n(blocks[0:1])
    assert invalid_queue.num_free_blocks == 0
    assert (
        invalid_queue.fake_free_list_head.next_free_block
        == invalid_queue.fake_free_list_tail
    )
```
**EN:** Test case covering `free KV cache block queue append n`. It exercises `FreeKVCacheBlockQueue, queue.append_n, KVCacheBlock, pytest.raises, invalid_queue.append_n, range`. The body contains 34 explicit assertion(s).
**CN:** 该代码块是覆盖 `free KV 缓存 block queue append n` 的测试用例。 该测试会调用 `FreeKVCacheBlockQueue, queue.append_n, KVCacheBlock, pytest.raises, invalid_queue.append_n, range`。 代码主体包含 34 个显式断言。

### test_free_kv_cache_block_queue_popleft_n (lines 361-416)
```python
def test_free_kv_cache_block_queue_popleft_n():
    blocks = [KVCacheBlock(block_id=i) for i in range(6)]
    # Create an empty FreeKVCacheBlockQueue with these blocks
    queue = FreeKVCacheBlockQueue(
        [blocks[1], blocks[3], blocks[5], blocks[4], blocks[0], blocks[2]]
    )
    assert queue.num_free_blocks == 6
    assert queue.fake_free_list_head.next_free_block is blocks[1]
    assert blocks[1].prev_free_block is queue.fake_free_list_head
    assert blocks[1].next_free_block is blocks[3]
    assert blocks[3].prev_free_block is blocks[1]
    assert blocks[3].next_free_block is blocks[5]
    assert blocks[5].prev_free_block is blocks[3]
    assert blocks[5].next_free_block is blocks[4]
    assert blocks[4].prev_free_block is blocks[5]
    assert blocks[4].next_free_block is blocks[0]
    assert blocks[0].prev_free_block is blocks[4]
    assert blocks[0].next_free_block is blocks[2]
    # ... excerpt omitted for brevity ...
    assert blocks[2].prev_free_block is blocks[0]
    assert blocks[2].next_free_block is queue.fake_free_list_tail
    assert queue.fake_free_list_tail.prev_free_block is blocks[2]
    assert len(queue.popleft_n(0)) == 0
    assert queue.num_free_blocks == 5
    assert result_blocks[0] is blocks[4]
    assert result_blocks[1] is blocks[0]
    assert result_blocks[2] is blocks[2]
    for block in result_blocks:
        assert block.prev_free_block is None
        assert block.next_free_block is None
```
**EN:** Test case covering `free KV cache block queue popleft n`. It exercises `FreeKVCacheBlockQueue, queue.popleft_n, KVCacheBlock, len, range`. The body contains 35 explicit assertion(s).
**CN:** 该代码块是覆盖 `free KV 缓存 block queue popleft n` 的测试用例。 该测试会调用 `FreeKVCacheBlockQueue, queue.popleft_n, KVCacheBlock, len, range`。 代码主体包含 35 个显式断言。

### test_free_kv_cache_block_queue_get_all_free_blocks (lines 419-440)
```python
def test_free_kv_cache_block_queue_get_all_free_blocks():
    # Create a list of KVCacheBlock objects
    blocks = [KVCacheBlock(block_id=i) for i in range(5)]

    # Create a FreeKVCacheBlockQueue with these blocks
    queue = FreeKVCacheBlockQueue(blocks)

    # Check all blocks are correctly retrieved
    assert queue.get_all_free_blocks() == blocks

    # Pop a block and check again
    queue.popleft()
    assert queue.get_all_free_blocks() == blocks[1:]

    # Remove a block and check again
    block_to_remove = blocks[2]
    queue.remove(block_to_remove)
    assert queue.get_all_free_blocks() == blocks[1:2] + blocks[3:]

    # Append a block back and check again
    queue.append(block_to_remove)
    assert queue.get_all_free_blocks() == blocks[1:2] + blocks[3:] + [block_to_remove]
```
**EN:** Test case covering `free KV cache block queue get all free blocks`. It exercises `FreeKVCacheBlockQueue, queue.popleft, queue.remove, queue.append, KVCacheBlock, queue.get_all_free_blocks`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `free KV 缓存 block queue get all free blocks` 的测试用例。 该测试会调用 `FreeKVCacheBlockQueue, queue.popleft, queue.remove, queue.append, KVCacheBlock, queue.get_all_free_blocks`。 代码主体包含 4 个显式断言。

### test_generate_block_hash_extra_keys (lines 443-472)
```python
def test_generate_block_hash_extra_keys():
    request = make_request(
        request_id="0",
        prompt_token_ids=[_ for _ in range(20)],
        mm_positions=[
            PlaceholderRange(offset=0, length=5),
            PlaceholderRange(offset=10, length=5),
        ],
        mm_hashes=["hash1", "hash2"],
    )

    # Test with no extra keys
    extra_keys, next_mm_idx = generate_block_hash_extra_keys(request, 0, 5, 0)
    assert extra_keys == (("hash1", 0),)
    assert next_mm_idx == 1

    # Test with partial overlap
    extra_keys, next_mm_idx = generate_block_hash_extra_keys(request, 3, 8, 0)
    assert extra_keys == (("hash1", -3),)
    assert next_mm_idx == 1

    # Test with no overlap
    extra_keys, next_mm_idx = generate_block_hash_extra_keys(request, 6, 10, 0)
    assert extra_keys is None
    assert next_mm_idx == 1

    # Test with multiple extra keys
    extra_keys, next_mm_idx = generate_block_hash_extra_keys(request, 0, 15, 0)
    assert extra_keys == (("hash1", 0), ("hash2", 10))
    assert next_mm_idx == 2
```
**EN:** Test case covering `generate block hash extra keys`. It exercises `make_request, generate_block_hash_extra_keys, PlaceholderRange, range`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `generate block hash extra keys` 的测试用例。 该测试会调用 `make_request, generate_block_hash_extra_keys, PlaceholderRange, range`。 代码主体包含 8 个显式断言。

### test_generate_block_hash_extra_keys_no_mm_inputs (lines 475-485)
```python
def test_generate_block_hash_extra_keys_no_mm_inputs():
    request = make_request(
        request_id="0",
        prompt_token_ids=[_ for _ in range(6)],
        mm_positions=None,
        mm_hashes=None,
    )

    extra_keys, next_mm_idx = generate_block_hash_extra_keys(request, 0, 5, 0)
    assert extra_keys is None
    assert next_mm_idx == 0
```
**EN:** Test case covering `generate block hash extra keys no mm inputs`. It exercises `make_request, generate_block_hash_extra_keys, range`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `generate block hash extra keys no mm inputs` 的测试用例。 该测试会调用 `make_request, generate_block_hash_extra_keys, range`。 代码主体包含 2 个显式断言。

### test_generate_block_hash_extra_keys_cache_salt (lines 488-523)
```python
def test_generate_block_hash_extra_keys_cache_salt():
    request = make_request(
        request_id="0",
        prompt_token_ids=[_ for _ in range(6)],
        mm_positions=None,
        mm_hashes=None,
        cache_salt="salt",
    )

    # salt is added for the first token
    extra_keys, _ = generate_block_hash_extra_keys(request, 0, 1, 0)
    assert extra_keys == ("salt",)
    extra_keys, _ = generate_block_hash_extra_keys(request, 0, 10, 0)
    # no salt added for other tokens
    extra_keys, _ = generate_block_hash_extra_keys(request, 1, 2, 0)
    assert extra_keys is None
    # ... excerpt omitted for brevity ...
    # Test with no extra keys
    extra_keys, next_mm_idx = generate_block_hash_extra_keys(request_mm, 0, 5, 0)
    assert extra_keys == (("hash1", 0), "salt")
    assert next_mm_idx == 1
```
**EN:** Test case covering `generate block hash extra keys cache salt`. It exercises `make_request, generate_block_hash_extra_keys, PlaceholderRange, range`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `generate block hash extra keys 缓存 salt` 的测试用例。 该测试会调用 `make_request, generate_block_hash_extra_keys, PlaceholderRange, range`。 代码主体包含 6 个显式断言。

### test_generate_block_hash_extra_keys_prompt_embeds (lines 526-546)
```python
def test_generate_block_hash_extra_keys_prompt_embeds():
    prompt_embeds = torch.randn(10, 3)
    request = make_request(
        request_id="0",
        prompt_token_ids=None,
        mm_positions=None,
        mm_hashes=None,
        prompt_embeds=prompt_embeds,
    )

    # Test with prompt embeds for the first block
    extra_keys, _ = generate_block_hash_extra_keys(request, 0, 5, 0)
    expected_embeds = prompt_embeds[0:5]
    expected_hash = hashlib.sha256(kv_cache_utils.tensor_data(expected_embeds)).digest()
    assert extra_keys == (expected_hash,)

    # Test with prompt embeds for the second block
    extra_keys, _ = generate_block_hash_extra_keys(request, 5, 10, 0)
    expected_embeds = prompt_embeds[5:10]
    expected_hash = hashlib.sha256(kv_cache_utils.tensor_data(expected_embeds)).digest()
    assert extra_keys == (expected_hash,)
```
**EN:** Test case covering `generate block hash extra keys prompt embeds`. It exercises `torch.randn, make_request, generate_block_hash_extra_keys, sha256.digest, hashlib.sha256, kv_cache_utils.tensor_data`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `generate block hash extra keys prompt embeds` 的测试用例。 该测试会调用 `torch.randn, make_request, generate_block_hash_extra_keys, sha256.digest, hashlib.sha256, kv_cache_utils.tensor_data`。 代码主体包含 2 个显式断言。

### test_generate_block_hash_extra_keys_prompt_embeds_cached (lines 549-573)
```python
def test_generate_block_hash_extra_keys_prompt_embeds_cached(monkeypatch):
    prompt_embeds = torch.randn(10, 3)
    request = make_request(
        request_id="0",
        prompt_token_ids=None,
        mm_positions=None,
        mm_hashes=None,
        prompt_embeds=prompt_embeds,
        block_size=20,
    )

    num_tensor_data_calls = 0
    original_tensor_data = kv_cache_utils.tensor_data

    def counting_tensor_data(tensor: torch.Tensor):
        nonlocal num_tensor_data_calls
        num_tensor_data_calls += 1
        return original_tensor_data(tensor)

    monkeypatch.setattr(kv_cache_utils, "tensor_data", counting_tensor_data)

    extra_keys_1, _ = generate_block_hash_extra_keys(request, 0, 5, 0)
    extra_keys_2, _ = generate_block_hash_extra_keys(request, 0, 5, 0)
    assert extra_keys_1 == extra_keys_2
    assert num_tensor_data_calls == 1
```
**EN:** Test case covering `generate block hash extra keys prompt embeds cached`. Inputs/fixtures: `monkeypatch`. It exercises `torch.randn, make_request, monkeypatch.setattr, generate_block_hash_extra_keys, original_tensor_data`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `generate block hash extra keys prompt embeds cached` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `torch.randn, make_request, monkeypatch.setattr, generate_block_hash_extra_keys, original_tensor_data`。 代码主体包含 2 个显式断言。

### test_generate_block_hash_extra_keys_different_prompt_embeds (lines 576-596)
```python
def test_generate_block_hash_extra_keys_different_prompt_embeds():
    prompt_embeds1 = torch.randn(10, 3)
    prompt_embeds2 = torch.randn(10, 3)
    request1 = make_request(
        request_id="0",
        prompt_token_ids=None,
        mm_positions=None,
        mm_hashes=None,
        prompt_embeds=prompt_embeds1,
    )
    request2 = make_request(
        request_id="1",
        prompt_token_ids=None,
        mm_positions=None,
        mm_hashes=None,
        prompt_embeds=prompt_embeds2,
    )

    extra_keys1, _ = generate_block_hash_extra_keys(request1, 0, 5, 0)
    extra_keys2, _ = generate_block_hash_extra_keys(request2, 0, 5, 0)
    assert extra_keys1 != extra_keys2
```
**EN:** Test case covering `generate block hash extra keys different prompt embeds`. It exercises `torch.randn, make_request, generate_block_hash_extra_keys`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `generate block hash extra keys different prompt embeds` 的测试用例。 该测试会调用 `torch.randn, make_request, generate_block_hash_extra_keys`。 代码主体包含 1 个显式断言。

### test_generate_block_hash_extra_keys_lora (lines 599-614)
```python
def test_generate_block_hash_extra_keys_lora():
    request = make_request(
        request_id="0",
        prompt_token_ids=[_ for _ in range(6)],
    )

    request.lora_request = LoRARequest(
        lora_name="test_lora_adapter", lora_int_id=1, lora_path="/path/to/lora"
    )

    extra_keys, _ = generate_block_hash_extra_keys(request, 0, 3, 0)
    assert extra_keys == ("test_lora_adapter",)

    request.lora_request = None
    extra_keys, _ = generate_block_hash_extra_keys(request, 0, 3, 0)
    assert extra_keys is None
```
**EN:** Test case covering `generate block hash extra keys lora`. It exercises `make_request, LoRARequest, generate_block_hash_extra_keys, range`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `generate block hash extra keys lora` 的测试用例。 该测试会调用 `make_request, LoRARequest, generate_block_hash_extra_keys, range`。 代码主体包含 2 个显式断言。

### test_hash_block_tokens (lines 618-627)
```python
def test_hash_block_tokens(hash_fn):
    parent_block_hash = BlockHash(b"123")
    curr_block_token_ids = (1, 2, 3)
    extra_keys = ("key1", "key2")

    block_hash = hash_block_tokens(
        hash_fn, parent_block_hash, curr_block_token_ids, extra_keys
    )
    expected = hash_fn((parent_block_hash, curr_block_token_ids, extra_keys))
    assert block_hash == expected
```
**EN:** Parameterized test covering `hash block tokens`. Parameter axes: `hash_fn`. Inputs/fixtures: `hash_fn`. It exercises `mark.parametrize, BlockHash, hash_block_tokens, hash_fn`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `hash block tokens` 的测试用例。 参数维度：`hash_fn`。 输入或 fixture：`hash_fn`。 该测试会调用 `mark.parametrize, BlockHash, hash_block_tokens, hash_fn`。 代码主体包含 1 个显式断言。

### test_request_block_hasher (lines 631-649)
```python
def test_request_block_hasher(hash_fn):
    request = make_request(
        request_id="0",
        prompt_token_ids=[_ for _ in range(6)],
        block_size=3,
        hash_fn=hash_fn,
        mm_positions=[
            PlaceholderRange(offset=0, length=3),
            PlaceholderRange(offset=3, length=3),
        ],
        mm_hashes=["hash1", "hash2"],
    )

    block_hashes = request.block_hashes
    assert len(block_hashes) == 2
    assert block_hashes[0] == hash_fn(
        (kv_cache_utils.NONE_HASH, (0, 1, 2), (("hash1", 0),))
    )
    assert block_hashes[1] == hash_fn((block_hashes[0], (3, 4, 5), (("hash2", 0),)))
```
**EN:** Parameterized test covering `request block hasher`. Parameter axes: `hash_fn`. Inputs/fixtures: `hash_fn`. It exercises `mark.parametrize, make_request, len, hash_fn, PlaceholderRange, range`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `request block hasher` 的测试用例。 参数维度：`hash_fn`。 输入或 fixture：`hash_fn`。 该测试会调用 `mark.parametrize, make_request, len, hash_fn, PlaceholderRange, range`。 代码主体包含 3 个显式断言。

### test_hash_tokens_different_mm_input (lines 653-677)
```python
def test_hash_tokens_different_mm_input(hash_fn):
    request1 = make_request(
        request_id="0",
        prompt_token_ids=[_ for _ in range(6)],
        block_size=3,
        hash_fn=hash_fn,
        mm_positions=[
            PlaceholderRange(offset=0, length=3),
            PlaceholderRange(offset=3, length=3),
        ],
        mm_hashes=["hash1", "hash2"],
    )
    request2 = make_request(
        request_id="1",
        prompt_token_ids=[_ for _ in range(6)],
        mm_positions=[
            PlaceholderRange(offset=0, length=3),
            PlaceholderRange(offset=3, length=3),
        ],
        mm_hashes=["hash3", "hash2"],
    )
    block_hashes1 = request1.block_hashes
    block_hashes2 = request2.block_hashes
    assert block_hashes1[0] != block_hashes2[0]
    assert block_hashes1[1] != block_hashes2[1]
```
**EN:** Parameterized test covering `hash tokens different mm input`. Parameter axes: `hash_fn`. Inputs/fixtures: `hash_fn`. It exercises `mark.parametrize, make_request, PlaceholderRange, range`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `hash tokens different mm input` 的测试用例。 参数维度：`hash_fn`。 输入或 fixture：`hash_fn`。 该测试会调用 `mark.parametrize, make_request, PlaceholderRange, range`。 代码主体包含 2 个显式断言。

### test_hash_request_tokens_no_mm_inputs (lines 681-695)
```python
def test_hash_request_tokens_no_mm_inputs(hash_fn):
    request = make_request(
        request_id="0",
        prompt_token_ids=[_ for _ in range(6)],
        block_size=3,
        hash_fn=hash_fn,
        mm_positions=None,
        mm_hashes=None,
    )

    block_hashes = request.block_hashes

    assert len(block_hashes) == 2
    assert block_hashes[0] == hash_fn((kv_cache_utils.NONE_HASH, (0, 1, 2), None))
    assert block_hashes[1] == hash_fn((block_hashes[0], (3, 4, 5), None))
```
**EN:** Parameterized test covering `hash request tokens no mm inputs`. Parameter axes: `hash_fn`. Inputs/fixtures: `hash_fn`. It exercises `mark.parametrize, make_request, len, hash_fn, range`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `hash request tokens no mm inputs` 的测试用例。 参数维度：`hash_fn`。 输入或 fixture：`hash_fn`。 该测试会调用 `mark.parametrize, make_request, len, hash_fn, range`。 代码主体包含 3 个显式断言。

### _stats (lines 698-699)
```python
def _stats(requests: int, queries: int, hits: int) -> PrefixCacheStats:
    return PrefixCacheStats(requests=requests, queries=queries, hits=hits)
```
**EN:** Helper function `_stats` encapsulates reusable logic for `stats`. Inputs: `requests, queries, hits`. Key calls include `PrefixCacheStats`.
**CN:** 辅助函数 `_stats` 封装了与 `stats` 相关的可复用逻辑。 输入参数：`requests, queries, hits`。 关键调用包括 `PrefixCacheStats`。

### test_metrics (lines 702-731)
```python
def test_metrics():
    """
    Test the prefix caching metrics.
    """
    metrics = CachingMetrics(max_recent_requests=5)
    assert metrics.hit_rate == 0.0

    metrics.observe(_stats(1, 20, 9))
    # 9 / 20 = 0.45
    assert metrics.hit_rate == 0.45

    metrics.observe(_stats(4, 80, 16))

    # 25 / 100 = 0.25
    assert metrics.hit_rate == 0.25

    metrics.observe(_stats(1, 10, 2))

    # Remove (20, 9) and add (10, 2): 18 / 90 = 0.2
    assert metrics.aggregated_requests == 5
    assert metrics.aggregated_query_total == 90
    assert metrics.aggregated_query_hit == 18
    assert metrics.hit_rate == 0.2

    metrics.reset()
    assert metrics.hit_rate == 0.0
    assert metrics.aggregated_requests == 0
    assert metrics.aggregated_query_total == 0
    assert metrics.aggregated_query_hit == 0
    assert not metrics.query_queue
```
**EN:** Test case covering `metrics`. It exercises `CachingMetrics, metrics.observe, metrics.reset, _stats`. The body contains 12 explicit assertion(s).
**CN:** 该代码块是覆盖 `metrics` 的测试用例。 该测试会调用 `CachingMetrics, metrics.observe, metrics.reset, _stats`。 代码主体包含 12 个显式断言。

### test_metrics_empty_stats (lines 734-763)
```python
def test_metrics_empty_stats():
    """
    Test the prefix caching metrics with empty stats.
    """
    metrics = CachingMetrics(max_recent_requests=5)
    metrics.observe(_stats(0, 0, 0))
    metrics.observe(_stats(1, 20, 9))
    metrics.observe(_stats(0, 0, 0))
    metrics.observe(_stats(4, 80, 16))
    metrics.observe(_stats(0, 0, 0))
    metrics.observe(_stats(1, 10, 2))
    # Remove (20, 9) and add (10, 2): 18 / 90 = 0.2
    assert metrics.aggregated_requests == 5
    assert metrics.aggregated_query_total == 90
    assert metrics.aggregated_query_hit == 18
    assert metrics.hit_rate == 0.2

    # Only the latest added stats preserved 10 / 20 = 0.5
    metrics.observe(_stats(11, 20, 10))
    assert metrics.aggregated_requests == 11
    assert metrics.aggregated_query_total == 20
    assert metrics.aggregated_query_hit == 10
    assert metrics.hit_rate == 0.5

    # Only the latest added stats preserved 30 / 40 = 0.75
    metrics.observe(_stats(22, 40, 30))
    assert metrics.aggregated_requests == 22
    assert metrics.aggregated_query_total == 40
    assert metrics.aggregated_query_hit == 30
    assert metrics.hit_rate == 0.75
```
**EN:** Test case covering `metrics empty stats`. It exercises `CachingMetrics, metrics.observe, _stats`. The body contains 12 explicit assertion(s).
**CN:** 该代码块是覆盖 `metrics empty stats` 的测试用例。 该测试会调用 `CachingMetrics, metrics.observe, _stats`。 代码主体包含 12 个显式断言。

### test_get_kv_cache_configs_multiple_workers (lines 766-1115)
```python
def test_get_kv_cache_configs_multiple_workers():
    model_config = ModelConfig(max_model_len=16)
    vllm_config = VllmConfig(model_config=model_config)

    ref_kv_cache_spec = new_kv_cache_spec()
    same_kv_cache_specs = [
        {
            "layer1": new_kv_cache_spec(),
            "layer2": new_kv_cache_spec(),
        },
    ]
    # Basic case. All things are the same.
    kv_cache_configs = get_kv_cache_configs(
    # ... excerpt omitted for brevity ...
    assert kv_cache_configs == [
            conflicting_layer_specs,
            [
                ref_kv_cache_spec.page_size_bytes * 2 * 10,
            ],
        )
```
**EN:** Test case covering `get KV cache configs multiple workers`. It exercises `ModelConfig, VllmConfig, new_kv_cache_spec, get_kv_cache_configs, pytest.raises, KVCacheConfig`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `get KV 缓存 configs multiple workers` 的测试用例。 该测试会调用 `ModelConfig, VllmConfig, new_kv_cache_spec, get_kv_cache_configs, pytest.raises, KVCacheConfig`。 代码主体包含 6 个显式断言。

### test_get_kv_cache_configs_pp_sharding (lines 1123-1169)
```python
def test_get_kv_cache_configs_pp_sharding(asymmetric_memory):
    model_config = ModelConfig(max_model_len=512)
    vllm_config = VllmConfig(model_config=model_config)

    ref_kv_cache_spec = new_kv_cache_spec()
    pp_kv_cache_specs = [
        {"layer1": ref_kv_cache_spec},
        {"layer2": ref_kv_cache_spec},
    ]
    expected_num_blocks = model_config.max_model_len // ref_kv_cache_spec.block_size + 1
    avail_memory = ref_kv_cache_spec.page_size_bytes * expected_num_blocks
    # With per-worker validation, each worker only needs memory for its own
    # layers. Worker 2 having more memory shouldn't affect worker 1's config.
    available_memory = (
        [avail_memory, avail_memory * 2] if asymmetric_memory else [avail_memory] * 2
    )
    # ... excerpt omitted for brevity ...
    assert kv_cache_configs == [
            num_blocks=expected_num_blocks,
                    size=ref_kv_cache_spec.page_size_bytes * expected_num_blocks,
                    shared_by=["layer2"],
                ),
            ],
            kv_cache_groups=[KVCacheGroupSpec(["layer2"], ref_kv_cache_spec)],
        ),
```
**EN:** Parameterized test covering `get KV cache configs pp sharding`. Parameter axes: `asymmetric_memory`. Inputs/fixtures: `asymmetric_memory`. It exercises `mark.parametrize, ModelConfig, VllmConfig, new_kv_cache_spec, get_kv_cache_configs, KVCacheConfig`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `get KV 缓存 configs pp sharding` 的测试用例。 参数维度：`asymmetric_memory`。 输入或 fixture：`asymmetric_memory`。 该测试会调用 `mark.parametrize, ModelConfig, VllmConfig, new_kv_cache_spec, get_kv_cache_configs, KVCacheConfig`。 代码主体包含 1 个显式断言。

### test_project_kv_cache_groups_to_worker (lines 1172-1208)
```python
def test_project_kv_cache_groups_to_worker():
    spec_a = new_kv_cache_spec()
    spec_b = new_kv_cache_spec(num_kv_heads=4)

    global_groups = [
        KVCacheGroupSpec(["layer1", "layer2", "layer3"], spec_a),
    ]
    worker_spec = {"layer1": spec_a, "layer2": spec_a}
    projected = kv_cache_utils._project_kv_cache_groups_to_worker(
        global_groups, worker_spec
    )
    assert len(projected) == 1
    assert projected[0].layer_names == ["layer1", "layer2"]
    assert projected[0].kv_cache_spec is spec_a
        global_groups, {"layer4": spec_a}
    # ... excerpt omitted for brevity ...
    assert projected[0].layer_names == []
    assert projected[0].layer_names == ["layer1", "layer3"]
    proj_spec = projected[0].kv_cache_spec
    assert isinstance(proj_spec, UniformTypeKVCacheSpecs)
    assert set(proj_spec.kv_cache_specs.keys()) == {"layer1", "layer3"}
```
**EN:** Test case covering `project KV cache groups to worker`. It exercises `new_kv_cache_spec, kv_cache_utils._project_kv_cache_groups_to_worker, UniformTypeKVCacheSpecs, isinstance, KVCacheGroupSpec, len`. The body contains 10 explicit assertion(s).
**CN:** 该代码块是覆盖 `project KV 缓存 groups to worker` 的测试用例。 该测试会调用 `new_kv_cache_spec, kv_cache_utils._project_kv_cache_groups_to_worker, UniformTypeKVCacheSpecs, isinstance, KVCacheGroupSpec, len`。 代码主体包含 10 个显式断言。

### test_merge_kv_cache_spec (lines 1211-1272)
```python
def test_merge_kv_cache_spec():
    same_layer_specs = [
        new_kv_cache_spec(num_kv_heads=32),
    ]
    merged_layer_spec = same_layer_specs[0].merge(same_layer_specs)
    assert merged_layer_spec.block_size == 16
    assert merged_layer_spec.num_kv_heads == 32
    assert merged_layer_spec.head_size == 64
    assert merged_layer_spec.dtype == torch.float32
    assert merged_layer_spec.sliding_window is None

    different_layer_specs = [
        new_kv_cache_spec(num_kv_heads=16),
    with pytest.raises(AssertionError):
        different_layer_specs[0].merge(different_layer_specs)
    # ... excerpt omitted for brevity ...
    with pytest.raises(ValueError):
    assert merged_layer_spec.sliding_window == 1
        new_kv_cache_spec(num_kv_heads=32, sliding_window=None),
    merged_layer_spec = same_sliding_window_layer_spec_with_none[0].merge(
        same_sliding_window_layer_spec_with_none
    )
```
**EN:** Test case covering `merge KV cache spec`. It exercises `same_layer_specs.merge, new_kv_cache_spec, same_sliding_window_layer_specs.merge, same_sliding_window_layer_spec_with_none.merge, pytest.raises, different_layer_specs.merge`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `merge KV 缓存 spec` 的测试用例。 该测试会调用 `same_layer_specs.merge, new_kv_cache_spec, same_sliding_window_layer_specs.merge, same_sliding_window_layer_spec_with_none.merge, pytest.raises, different_layer_specs.merge`。 代码主体包含 7 个显式断言。

### test_is_kv_cache_spec_uniform (lines 1275-1304)
```python
def test_is_kv_cache_spec_uniform():
    kv_cache_spec = {
        "layer_1": new_kv_cache_spec(num_kv_heads=32),
        "layer_2": new_kv_cache_spec(num_kv_heads=32),
    }
    assert is_kv_cache_spec_uniform(kv_cache_spec)

    kv_cache_spec = {
        "layer_1": new_kv_cache_spec(num_kv_heads=32),
        "layer_2": new_kv_cache_spec(num_kv_heads=32, sliding_window=1),
    }
    assert is_kv_cache_spec_uniform(kv_cache_spec)

    kv_cache_spec = {
        "layer_1": new_kv_cache_spec(num_kv_heads=32),
        "layer_2": new_sliding_window_spec(num_kv_heads=32, sliding_window=1),
    }
    assert not is_kv_cache_spec_uniform(kv_cache_spec)

    kv_cache_spec = {
        "layer_1": new_sliding_window_spec(num_kv_heads=32, sliding_window=1),
        "layer_2": new_sliding_window_spec(num_kv_heads=32, sliding_window=1),
    }
    assert is_kv_cache_spec_uniform(kv_cache_spec)

    kv_cache_spec = {
        "layer_1": new_sliding_window_spec(num_kv_heads=32, sliding_window=1),
        "layer_2": new_sliding_window_spec(num_kv_heads=32, sliding_window=2),
    }
    assert not is_kv_cache_spec_uniform(kv_cache_spec)
```
**EN:** Test case covering `is KV cache spec uniform`. It exercises `is_kv_cache_spec_uniform, new_kv_cache_spec, new_sliding_window_spec`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `is KV 缓存 spec uniform` 的测试用例。 该测试会调用 `is_kv_cache_spec_uniform, new_kv_cache_spec, new_sliding_window_spec`。 代码主体包含 5 个显式断言。

### test_estimate_max_model_len (lines 1314-1347)
```python
def test_estimate_max_model_len(model_id, max_model_len, want_estimated_max_len):
    # Create a VllmConfig
    model_config = ModelConfig(
        model_id,
        runner="generate",
        dtype="float16",
        max_model_len=max_model_len,
    )
    scheduler_config = SchedulerConfig(
        max_num_batched_tokens=32768,
        max_model_len=model_config.max_model_len,
        is_encoder_decoder=model_config.is_encoder_decoder,

    vllm_config = VllmConfig(
        model_config=model_config,
        scheduler_config=scheduler_config,
    # ... excerpt omitted for brevity ...
        )
    # Estimate the maximum model length, 16384 model_len need 8GB
    estimated_max_len = estimate_max_model_len(
        vllm_config, kv_cache_spec, 8 * GiB_bytes
    assert estimated_max_len == want_estimated_max_len
```
**EN:** Test case covering `estimate max model len`. Inputs/fixtures: `model_id, max_model_len, want_estimated_max_len`. It exercises `mark.parametrize, ModelConfig, SchedulerConfig, VllmConfig, range, estimate_max_model_len`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `estimate max model len` 的测试用例。 输入或 fixture：`model_id, max_model_len, want_estimated_max_len`。 该测试会调用 `mark.parametrize, ModelConfig, SchedulerConfig, VllmConfig, range, estimate_max_model_len`。 代码主体包含 1 个显式断言。

### test_get_max_concurrency_for_kv_cache_config (lines 1350-1424)
```python
def test_get_max_concurrency_for_kv_cache_config():
    # Create a VllmConfig
    model_id = "Qwen/Qwen1.5-7B"
    max_model_len = 16384
    model_config = ModelConfig(
        model_id,
        runner="generate",
        dtype="float16",
        max_model_len=max_model_len,
    )
    scheduler_config = SchedulerConfig(
        max_num_batched_tokens=1024,
        enable_chunked_prefill=True,
        max_model_len=model_config.max_model_len,
        is_encoder_decoder=model_config.is_encoder_decoder,

    vllm_config = VllmConfig(
    # ... excerpt omitted for brevity ...
    assert max_concurrency_full_attention == 1.5
    assert max_concurrency_sliding_window == 3
        ],
    max_concurrency_hybrid_model = get_max_concurrency_for_kv_cache_config(
        vllm_config, kv_cache_config_hybrid_model
    assert max_concurrency_hybrid_model == 3
```
**EN:** Test case covering `get max concurrency for KV cache config`. It exercises `ModelConfig, SchedulerConfig, VllmConfig, FullAttentionSpec, SlidingWindowSpec, KVCacheConfig`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `get max concurrency for KV 缓存 config` 的测试用例。 该测试会调用 `ModelConfig, SchedulerConfig, VllmConfig, FullAttentionSpec, SlidingWindowSpec, KVCacheConfig`。 代码主体包含 3 个显式断言。

### test_allocate_with_lookahead (lines 1427-1481)
```python
def test_allocate_with_lookahead():
    """Verify that lookahead tokens correctly affect block allocation"""
    block_size = 4
    config = KVCacheConfig(
        num_blocks=10,
        kv_cache_tensors=[
            KVCacheTensor(size=100, shared_by=["layer1"]),
        ],
        kv_cache_groups=[
            KVCacheGroupSpec(["layer1"], new_kv_cache_spec(block_size=block_size)),
    )

    request = make_request(
        request_id="0",
        prompt_token_ids=[],
        block_size=block_size,
        mm_positions=None,
    # ... excerpt omitted for brevity ...
    assert len(blocks.get_block_ids()[0]) == 2  # ceil(5/4)=2 blocks
    assert len(blocks.get_block_ids()[0]) == 2
    blocks = kv_cache_manager.allocate_slots(
        request,
        num_new_tokens=3,
        num_lookahead_tokens=4,
```
**EN:** Test case covering `allocate with lookahead`. It exercises `KVCacheConfig, make_request, KVCacheManager, kv_cache_manager.allocate_slots, len, KVCacheTensor`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `allocate with lookahead` 的测试用例。 该测试会调用 `KVCacheConfig, make_request, KVCacheManager, kv_cache_manager.allocate_slots, len, KVCacheTensor`。 代码主体包含 3 个显式断言。

### test_get_kv_cache_config_one_worker (lines 1484-1773)
```python
def test_get_kv_cache_config_one_worker():
    # pass max_model_len to pass check_enough_kv_cache_memory
    model_config = ModelConfig(max_model_len=16)
    vllm_config = VllmConfig(model_config=model_config)

    mem_per_block_per_layer = 16 * 2 * 64 * 4 * 2
    # all layers are full attention -> single group
    kv_cache_specs_full = {
        "layer_1": new_kv_cache_spec(),
        "layer_2": new_kv_cache_spec(),
    }
    kv_cache_config_full = get_kv_cache_configs(
        vllm_config, [kv_cache_specs_full], [mem_per_block_per_layer * 2 * 32]
    )[0]
    print(kv_cache_config_full)
    assert kv_cache_config_full == KVCacheConfig(
        num_blocks=32,
        kv_cache_tensors=[
    # ... excerpt omitted for brevity ...
    assert kv_cache_config_sliding == KVCacheConfig(
    assert kv_cache_config_hybrid == KVCacheConfig(
            KVCacheTensor(size=mem_per_block_per_layer * 16, shared_by=["layer_1"]),
            KVCacheTensor(size=mem_per_block_per_layer * 16, shared_by=["layer_2"]),
        ],
        kv_cache_groups=[KVCacheGroupSpec(["layer_1", "layer_2"], new_kv_cache_spec())],
    )
```
**EN:** Test case covering `get KV cache config one worker`. It exercises `ModelConfig, VllmConfig, print, new_kv_cache_spec, get_kv_cache_configs, KVCacheConfig`. The body contains 10 explicit assertion(s).
**CN:** 该代码块是覆盖 `get KV 缓存 config one worker` 的测试用例。 该测试会调用 `ModelConfig, VllmConfig, print, new_kv_cache_spec, get_kv_cache_configs, KVCacheConfig`。 代码主体包含 10 个显式断言。

### test_get_kv_cache_configs_attention_free (lines 1776-1786)
```python
def test_get_kv_cache_configs_attention_free():
    kv_cache_specs: dict[str, KVCacheSpec] = {}
    vllm_config = VllmConfig(model_config=ModelConfig(max_model_len=16))
    kv_cache_configs = get_kv_cache_configs(vllm_config, [kv_cache_specs], [0])
    assert kv_cache_configs == [
        KVCacheConfig(
            num_blocks=1,
            kv_cache_tensors=[],
            kv_cache_groups=[],
        )
    ]
```
**EN:** Test case covering `get KV cache configs attention free`. It exercises `VllmConfig, get_kv_cache_configs, ModelConfig, KVCacheConfig`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `get KV 缓存 configs 注意力 free` 的测试用例。 该测试会调用 `VllmConfig, get_kv_cache_configs, ModelConfig, KVCacheConfig`。 代码主体包含 1 个显式断言。

### test_generate_uniform_type_kv_cache_specs (lines 1789-1832)
```python
def test_generate_uniform_type_kv_cache_specs():
    # All layers are full attention, can be merged
    kv_cache_specs = {
        "layer_1": new_kv_cache_spec(),
        "layer_2": new_kv_cache_spec(head_size=128),
    }
    uniform_spec = UniformTypeKVCacheSpecs.from_specs(kv_cache_specs)
    assert uniform_spec == UniformTypeKVCacheSpecs(
        block_size=16, kv_cache_specs=kv_cache_specs
    )

    # Full attention + sliding window, cannot be merged
        "layer_2": new_sliding_window_spec(sliding_window=1),
    assert uniform_spec is None
    # ... excerpt omitted for brevity ...
        "layer_1": new_kv_cache_spec(block_size=16),
        "layer_2": new_kv_cache_spec(block_size=32),
```
**EN:** Test case covering `generate uniform type KV cache specs`. It exercises `UniformTypeKVCacheSpecs.from_specs, new_kv_cache_spec, UniformTypeKVCacheSpecs, new_sliding_window_spec`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `generate uniform type KV 缓存 specs` 的测试用例。 该测试会调用 `UniformTypeKVCacheSpecs.from_specs, new_kv_cache_spec, UniformTypeKVCacheSpecs, new_sliding_window_spec`。 代码主体包含 5 个显式断言。

### test_generate_scheduler_kv_cache_config (lines 1835-1859)
```python
def test_generate_scheduler_kv_cache_config():
    kv_cache_specs = {
        "layer_1": new_kv_cache_spec(),
        "layer_2": new_kv_cache_spec(head_size=128),
    }
    kv_cache_configs = [
        KVCacheConfig(
            num_blocks=10,
            kv_cache_tensors=[],
            kv_cache_groups=[
                KVCacheGroupSpec(
                    ["layer_1", "layer_2"],
                    UniformTypeKVCacheSpecs(
                        block_size=16, kv_cache_specs=kv_cache_specs
                    ),
                ),
            ],
        )
    ]
    scheduler_kv_cache_config = generate_scheduler_kv_cache_config(kv_cache_configs)
    assert scheduler_kv_cache_config == KVCacheConfig(
        num_blocks=10,
        kv_cache_tensors=[],
        kv_cache_groups=[KVCacheGroupSpec(["layer_1", "layer_2"], new_kv_cache_spec())],
    )
```
**EN:** Test case covering `generate scheduler KV cache config`. It exercises `generate_scheduler_kv_cache_config, new_kv_cache_spec, KVCacheConfig, KVCacheGroupSpec, UniformTypeKVCacheSpecs`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `generate 调度器 KV 缓存 config` 的测试用例。 该测试会调用 `generate_scheduler_kv_cache_config, new_kv_cache_spec, KVCacheConfig, KVCacheGroupSpec, UniformTypeKVCacheSpecs`。 代码主体包含 1 个显式断言。

### new_mla_spec (lines 1862-1870)
```python
def new_mla_spec(cache_dtype_str=None):
    # head_size = kv_lora_rank(512) + qk_rope_head_dim(64) = 576
    return MLAAttentionSpec(
        block_size=16,
        num_kv_heads=1,
        head_size=576,
        dtype=torch.float32,
        cache_dtype_str=cache_dtype_str,
    )
```
**EN:** Helper function `new_mla_spec` encapsulates reusable logic for `new MLA spec`. Inputs: `cache_dtype_str`. Key calls include `MLAAttentionSpec`.
**CN:** 辅助函数 `new_mla_spec` 封装了与 `new mla spec` 相关的可复用逻辑。 输入参数：`cache_dtype_str`。 关键调用包括 `MLAAttentionSpec`。

### test_get_kv_cache_spec_kind_prefers_specific_attention_subclasses (lines 1873-1898)
```python
def test_get_kv_cache_spec_kind_prefers_specific_attention_subclasses():
    assert get_kv_cache_spec_kind(new_mla_spec()) == KVCacheSpecKind.MLA_ATTENTION

    sliding_window_mla_spec = SlidingWindowMLASpec(
        block_size=16,
        num_kv_heads=1,
        head_size=576,
        dtype=torch.float32,
        sliding_window=128,
    )
    assert (
        get_kv_cache_spec_kind(sliding_window_mla_spec)
        == KVCacheSpecKind.SLIDING_WINDOW_MLA
    )

    sink_full_attention_spec = SinkFullAttentionSpec(
        block_size=16,
        num_kv_heads=1,
        head_size=64,
        dtype=torch.float32,
        sink_len=4,
    )
    assert (
        get_kv_cache_spec_kind(sink_full_attention_spec)
        == KVCacheSpecKind.SINK_FULL_ATTENTION
    )
```
**EN:** Test case covering `get KV cache spec kind prefers specific attention subclasses`. It exercises `SlidingWindowMLASpec, SinkFullAttentionSpec, get_kv_cache_spec_kind, new_mla_spec`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `get KV 缓存 spec kind prefers specific 注意力 subclasses` 的测试用例。 该测试会调用 `SlidingWindowMLASpec, SinkFullAttentionSpec, get_kv_cache_spec_kind, new_mla_spec`。 代码主体包含 3 个显式断言。

### test_get_kv_cache_spec_kind_unwraps_uniform_type_specs (lines 1901-1933)
```python
def test_get_kv_cache_spec_kind_unwraps_uniform_type_specs():
    uniform_mla_spec = UniformTypeKVCacheSpecs(
        block_size=16,
        kv_cache_specs={
            "layer_1": new_mla_spec(),
            "layer_2": new_mla_spec(cache_dtype_str="fp8"),
        },
    )
    assert get_kv_cache_spec_kind(uniform_mla_spec) == KVCacheSpecKind.MLA_ATTENTION

    uniform_swa_mla_spec = UniformTypeKVCacheSpecs(
            "layer_1": SlidingWindowMLASpec(
                block_size=16,
                num_kv_heads=1,
                head_size=576,
                dtype=torch.float32,
    # ... excerpt omitted for brevity ...
    assert (
        get_kv_cache_spec_kind(uniform_swa_mla_spec)
        == KVCacheSpecKind.SLIDING_WINDOW_MLA
```
**EN:** Test case covering `get KV cache spec kind unwraps uniform type specs`. It exercises `UniformTypeKVCacheSpecs, get_kv_cache_spec_kind, new_mla_spec, SlidingWindowMLASpec`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `get KV 缓存 spec kind unwraps uniform type specs` 的测试用例。 该测试会调用 `UniformTypeKVCacheSpecs, get_kv_cache_spec_kind, new_mla_spec, SlidingWindowMLASpec`。 代码主体包含 2 个显式断言。

### test_get_kv_cache_spec_kind_unknown_for_mixed_uniform_type_specs (lines 1936-1950)
```python
def test_get_kv_cache_spec_kind_unknown_for_mixed_uniform_type_specs():
    uniform_mixed_spec = UniformTypeKVCacheSpecs(
        block_size=16,
        kv_cache_specs={
            "layer_1": new_mla_spec(),
            "layer_2": SlidingWindowMLASpec(
                block_size=16,
                num_kv_heads=1,
                head_size=576,
                dtype=torch.float32,
                sliding_window=128,
            ),
        },
    )
    assert get_kv_cache_spec_kind(uniform_mixed_spec) == KVCacheSpecKind.UNKNOWN
```
**EN:** Test case covering `get KV cache spec kind unknown for mixed uniform type specs`. It exercises `UniformTypeKVCacheSpecs, get_kv_cache_spec_kind, new_mla_spec, SlidingWindowMLASpec`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `get KV 缓存 spec kind unknown for mixed uniform type specs` 的测试用例。 该测试会调用 `UniformTypeKVCacheSpecs, get_kv_cache_spec_kind, new_mla_spec, SlidingWindowMLASpec`。 代码主体包含 1 个显式断言。

### test_get_kv_cache_spec_sliding_window_reads_windowed_specs (lines 1953-1969)
```python
def test_get_kv_cache_spec_sliding_window_reads_windowed_specs():
    full_attention_spec = FullAttentionSpec(
        block_size=16,
        num_kv_heads=1,
        head_size=64,
        dtype=torch.float32,
    )
    sliding_window_spec = SlidingWindowSpec(
        block_size=16,
        num_kv_heads=1,
        head_size=64,
        dtype=torch.float32,
        sliding_window=128,
    )

    assert get_kv_cache_spec_sliding_window(full_attention_spec) is None
    assert get_kv_cache_spec_sliding_window(sliding_window_spec) == 128
```
**EN:** Test case covering `get KV cache spec sliding window reads windowed specs`. It exercises `FullAttentionSpec, SlidingWindowSpec, get_kv_cache_spec_sliding_window`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `get KV 缓存 spec sliding window reads windowed specs` 的测试用例。 该测试会调用 `FullAttentionSpec, SlidingWindowSpec, get_kv_cache_spec_sliding_window`。 代码主体包含 2 个显式断言。

### test_get_kv_cache_spec_sliding_window_unwraps_uniform_type_specs (lines 1972-2013)
```python
def test_get_kv_cache_spec_sliding_window_unwraps_uniform_type_specs():
    uniform_window_spec = UniformTypeKVCacheSpecs(
        block_size=16,
        kv_cache_specs={
            "layer_1": SlidingWindowSpec(
                block_size=16,
                num_kv_heads=1,
                head_size=64,
                dtype=torch.float32,
                sliding_window=128,
            ),
            "layer_2": SlidingWindowSpec(
                num_kv_heads=2,
    # ... excerpt omitted for brevity ...
        },
    )

    assert get_kv_cache_spec_sliding_window(uniform_window_spec) == 128
    assert get_kv_cache_spec_sliding_window(mixed_window_spec) is None
```
**EN:** Test case covering `get KV cache spec sliding window unwraps uniform type specs`. It exercises `UniformTypeKVCacheSpecs, get_kv_cache_spec_sliding_window, SlidingWindowSpec`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `get KV 缓存 spec sliding window unwraps uniform type specs` 的测试用例。 该测试会调用 `UniformTypeKVCacheSpecs, get_kv_cache_spec_sliding_window, SlidingWindowSpec`。 代码主体包含 2 个显式断言。

### test_merge_mla_spec (lines 2016-2050)
```python
def test_merge_mla_spec():
    kv_cache_specs = [
        new_mla_spec(),
    ]
    mla_spec = kv_cache_specs[0].merge(kv_cache_specs)
    assert mla_spec == new_mla_spec()

        new_mla_spec(cache_dtype_str="fp8_ds_mla"),
    assert mla_spec == new_mla_spec(cache_dtype_str="fp8_ds_mla")
        new_mla_spec(cache_dtype_str=None),
    # ... excerpt omitted for brevity ...
    with pytest.raises(AssertionError):
        new_kv_cache_spec(),
        kv_cache_specs[0].merge(kv_cache_specs)
```
**EN:** Test case covering `merge MLA spec`. It exercises `kv_cache_specs.merge, new_mla_spec, pytest.raises, new_kv_cache_spec`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `merge mla spec` 的测试用例。 该测试会调用 `kv_cache_specs.merge, new_mla_spec, pytest.raises, new_kv_cache_spec`。 代码主体包含 2 个显式断言。

### test_request_block_hasher_with_prompt_embeds (lines 2054-2094)
```python
def test_request_block_hasher_with_prompt_embeds(hash_fn: Callable[[Any], bytes]):
    block_size = 3
    num_tokens = 2 * block_size
    prompt_token_ids = [_ for _ in range(num_tokens)]
    hidden_size = 5
    prompt_embeds = torch.randn((num_tokens, hidden_size))

    request = make_request(
        request_id="0",
        prompt_token_ids=prompt_token_ids,
        block_size=block_size,
        hash_fn=hash_fn,
        prompt_embeds=prompt_embeds,
    )
    block_hashes = request.block_hashes
    assert len(block_hashes) == 2
    # ... excerpt omitted for brevity ...
    expected_hash1 = hash_fn(
    assert block_hashes[0] == expected_hash1
    expected_hash2 = hash_fn(
            block_hashes[0],
            tuple(prompt_token_ids[block_size:num_tokens]),
            (block2_embeds_hash,),
        )
    assert block_hashes[1] == expected_hash2
```
**EN:** Parameterized test covering `request block hasher with prompt embeds`. Parameter axes: `hash_fn`. Inputs/fixtures: `hash_fn`. It exercises `mark.parametrize, torch.randn, make_request, sha256.digest, hash_fn, len`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `request block hasher with prompt embeds` 的测试用例。 参数维度：`hash_fn`。 输入或 fixture：`hash_fn`。 该测试会调用 `mark.parametrize, torch.randn, make_request, sha256.digest, hash_fn, len`。 代码主体包含 3 个显式断言。

### test_request_with_prompt_embeds_and_mm_inputs (lines 2098-2143)
```python
def test_request_with_prompt_embeds_and_mm_inputs(hash_fn: Callable[[Any], bytes]):
    block_size = 3
    num_tokens = 2 * block_size
    prompt_token_ids = [_ for _ in range(num_tokens)]
    hidden_size = 5
    prompt_embeds = torch.randn((num_tokens, hidden_size))

    request = make_request(
        request_id="0",
        prompt_token_ids=prompt_token_ids,
        block_size=block_size,
        hash_fn=hash_fn,
        mm_positions=[
            PlaceholderRange(offset=0, length=3),
            PlaceholderRange(offset=3, length=3),
        ],
        mm_hashes=["hash1", "hash2"],
        prompt_embeds=prompt_embeds,
    # ... excerpt omitted for brevity ...
    assert len(block_hashes) == 2
    expected_hash1 = hash_fn(
    assert block_hashes[0] == expected_hash1
    expected_hash2 = hash_fn(
            block_hashes[0],
            tuple(prompt_token_ids[block_size:num_tokens]),
            (("hash2", 0), block2_embeds_hash),
        )
    )
    assert block_hashes[1] == expected_hash2
```
**EN:** Parameterized test covering `request with prompt embeds and mm inputs`. Parameter axes: `hash_fn`. Inputs/fixtures: `hash_fn`. It exercises `mark.parametrize, torch.randn, make_request, sha256.digest, hash_fn, len`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `request with prompt embeds and mm inputs` 的测试用例。 参数维度：`hash_fn`。 输入或 fixture：`hash_fn`。 该测试会调用 `mark.parametrize, torch.randn, make_request, sha256.digest, hash_fn, len`。 代码主体包含 3 个显式断言。

### test_auto_fit_max_model_len (lines 2146-2181)
```python
def test_auto_fit_max_model_len():
    """Test that max_model_len=-1 auto-fits to available GPU memory."""
    # Create config with original_max_model_len=-1 to trigger auto-fit
    model_config = ModelConfig(max_model_len=1024)
    # Simulate the user passing -1 by setting original_max_model_len
    model_config.original_max_model_len = -1
    vllm_config = VllmConfig(model_config=model_config)

    mem_per_block_per_layer = 16 * 2 * 64 * 4 * 2  # 16KB per block per layer
    kv_cache_specs = {
        "layer_1": new_kv_cache_spec(),
        "layer_2": new_kv_cache_spec(),
    }
    # With enough memory, max_model_len stays at the derived max
    large_available_memory = mem_per_block_per_layer * 2 * 1024  # plenty of memory
    _kv_cache_configs = get_kv_cache_configs(
        vllm_config, [kv_cache_specs], [large_available_memory]
    # ... excerpt omitted for brevity ...
    assert vllm_config.model_config.max_model_len == 1024
        vllm_config, [kv_cache_specs], [limited_memory]
    )
    # Should be reduced to fit in memory
    assert vllm_config.model_config.max_model_len < 1024
    assert vllm_config.model_config.max_model_len > 0
```
**EN:** Test case covering `auto fit max model len`. It exercises `ModelConfig, VllmConfig, get_kv_cache_configs, new_kv_cache_spec`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `auto fit max model len` 的测试用例。 该测试会调用 `ModelConfig, VllmConfig, get_kv_cache_configs, new_kv_cache_spec`。 代码主体包含 3 个显式断言。

### test_auto_fit_max_model_len_with_hybrid (lines 2184-2203)
```python
def test_auto_fit_max_model_len_with_hybrid():
    """Test that auto-fit works with hybrid KV cache specs."""
    # Create config with original_max_model_len=-1 to trigger auto-fit
    model_config = ModelConfig(max_model_len=8192)
    # Simulate the user passing -1 by setting original_max_model_len
    model_config.original_max_model_len = -1
    vllm_config = VllmConfig(model_config=model_config)

    mem_per_block_per_layer = 16 * 2 * 64 * 4 * 2  # 16KB per block per layer
    gamma = 2
    kv_cache_specs = {
        "layer_1": new_mamba_spec(num_speculative_blocks=gamma),
        "layer_2": new_kv_cache_spec(),
    }

    available_memory = mem_per_block_per_layer * (1024 // 16 + 1 + gamma)
    _kv_cache_configs = get_kv_cache_configs(
        vllm_config, [kv_cache_specs], [available_memory]
    )
    assert vllm_config.model_config.max_model_len == 1024
```
**EN:** Test case covering `auto fit max model len with hybrid`. It exercises `ModelConfig, VllmConfig, get_kv_cache_configs, new_mamba_spec, new_kv_cache_spec`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `auto fit max model len with hybrid` 的测试用例。 该测试会调用 `ModelConfig, VllmConfig, get_kv_cache_configs, new_mamba_spec, new_kv_cache_spec`。 代码主体包含 1 个显式断言。

### test_auto_fit_max_model_len_not_triggered (lines 2206-2222)
```python
def test_auto_fit_max_model_len_not_triggered():
    """Test that auto-fit is not triggered when original_max_model_len is not -1."""
    model_config = ModelConfig(max_model_len=16)
    # original_max_model_len should be None by default, not -1
    vllm_config = VllmConfig(model_config=model_config)

    mem_per_block_per_layer = 16 * 2 * 64 * 4 * 2
    kv_cache_specs = {
        "layer_1": new_kv_cache_spec(),
        "layer_2": new_kv_cache_spec(),
    }

    # This should work normally without auto-fit
    _kv_cache_configs = get_kv_cache_configs(
        vllm_config, [kv_cache_specs], [mem_per_block_per_layer * 2 * 32]
    )
    assert vllm_config.model_config.max_model_len == 16
```
**EN:** Test case covering `auto fit max model len not triggered`. It exercises `ModelConfig, VllmConfig, get_kv_cache_configs, new_kv_cache_spec`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `auto fit max model len not triggered` 的测试用例。 该测试会调用 `ModelConfig, VllmConfig, get_kv_cache_configs, new_kv_cache_spec`。 代码主体包含 1 个显式断言。

### test_auto_fit_max_model_len_respects_num_gpu_blocks_override (lines 2225-2248)
```python
def test_auto_fit_max_model_len_respects_num_gpu_blocks_override():
    """Auto-fit must size max_model_len against the override-clamped pool, not
    the raw `available_memory`. Without this, auto-fit could pick a
    max_model_len that no longer fits once `num_gpu_blocks_override` is applied.
    """
    model_config = ModelConfig(max_model_len=16384)
    model_config.original_max_model_len = -1  # request auto-fit
    vllm_config = VllmConfig(model_config=model_config)
    # Cap the cache to 32 blocks regardless of available memory.
    vllm_config.cache_config.num_gpu_blocks_override = 32

    mem_per_block_per_layer = 16 * 2 * 64 * 4 * 2
    kv_cache_specs = {
        "layer_1": new_kv_cache_spec(),  # block_size=16
        "layer_2": new_kv_cache_spec(),
    }
    # Plenty of raw memory (1024 blocks per layer would fit max_model_len=16384).
    large_available_memory = mem_per_block_per_layer * 2 * 1024

    get_kv_cache_configs(vllm_config, [kv_cache_specs], [large_available_memory])

    # 32 blocks * block_size 16 = 512 token slots, so max_model_len must
    # auto-fit at or below that.
    assert 0 < vllm_config.model_config.max_model_len <= 32 * 16
```
**EN:** Test case covering `auto fit max model len respects num GPU blocks override`. It exercises `ModelConfig, VllmConfig, get_kv_cache_configs, new_kv_cache_spec`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `auto fit max model len respects num gpu blocks override` 的测试用例。 该测试会调用 `ModelConfig, VllmConfig, get_kv_cache_configs, new_kv_cache_spec`。 代码主体包含 1 个显式断言。

### test_check_enough_kv_cache_memory_respects_num_gpu_blocks_override (lines 2251-2270)
```python
def test_check_enough_kv_cache_memory_respects_num_gpu_blocks_override():
    """Admission check must use the override-clamped pool size, not raw
    `available_memory`. Without this, startup could accept a max_model_len
    that does not actually fit in `num_gpu_blocks_override` blocks.
    """
    model_config = ModelConfig(max_model_len=16384)
    vllm_config = VllmConfig(model_config=model_config)
    # 32 blocks is far too small for max_model_len=16384 (would need 1024).
    vllm_config.cache_config.num_gpu_blocks_override = 32

    mem_per_block_per_layer = 16 * 2 * 64 * 4 * 2
    kv_cache_specs = {
        "layer_1": new_kv_cache_spec(),
        "layer_2": new_kv_cache_spec(),
    }
    # Plenty of raw memory: a bytes-only check against this would pass.
    large_available_memory = mem_per_block_per_layer * 2 * 1024

    with pytest.raises(ValueError, match="max seq len"):
        get_kv_cache_configs(vllm_config, [kv_cache_specs], [large_available_memory])
```
**EN:** Test case covering `check enough KV cache memory respects num GPU blocks override`. It exercises `ModelConfig, VllmConfig, new_kv_cache_spec, pytest.raises, get_kv_cache_configs`. Validation is mainly performed through expected exception checks.
**CN:** 该代码块是覆盖 `check enough KV 缓存 memory respects num gpu blocks override` 的测试用例。 该测试会调用 `ModelConfig, VllmConfig, new_kv_cache_spec, pytest.raises, get_kv_cache_configs`。 主要通过预期异常检查来完成验证。

### test_unify_hybrid_kv_cache_specs (lines 2273-2337)
```python
def test_unify_hybrid_kv_cache_specs():
    # 1. has_full_attention and has_sliding_window
    before_spec_1 = new_kv_cache_spec()
    before_spec_2 = new_sliding_window_spec(
        page_size_padded=32 * 1024, sliding_window=1024
    )
    kv_cache_spec = {
        "layer_1": before_spec_1,
        "layer_2": before_spec_2,
    }
    kv_cache_utils.unify_hybrid_kv_cache_specs(kv_cache_spec)
    expected_spec_1 = new_kv_cache_spec()
    expected_spec_2 = new_kv_cache_spec(page_size_padded=32 * 1024, sliding_window=1024)
    assert kv_cache_spec["layer_1"] == expected_spec_1
    assert kv_cache_spec["layer_2"] == expected_spec_2

    # 2. has_full_attention and has_chunked_local_attention
    # ... excerpt omitted for brevity ...
    expected_spec_2 = new_kv_cache_spec(
        "layer_1": new_sliding_window_spec(sliding_window=1024),
        "layer_2": new_chunked_local_attention_spec(attention_chunk_size=512),
    with pytest.raises(ValueError):
        kv_cache_utils.unify_hybrid_kv_cache_specs(kv_cache_spec)
```
**EN:** Test case covering `unify hybrid KV cache specs`. It exercises `new_kv_cache_spec, new_sliding_window_spec, kv_cache_utils.unify_hybrid_kv_cache_specs, new_chunked_local_attention_spec, pytest.raises`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `unify hybrid KV 缓存 specs` 的测试用例。 该测试会调用 `new_kv_cache_spec, new_sliding_window_spec, kv_cache_utils.unify_hybrid_kv_cache_specs, new_chunked_local_attention_spec, pytest.raises`。 代码主体包含 7 个显式断言。

### test_hma_not_disabled_when_kv_events_enabled (lines 2340-2364)
```python
def test_hma_not_disabled_when_kv_events_enabled():
    """
    Test enabling KV events must not force disable_hybrid_kv_cache_manager to True.

    This test guards against that regression by verifying that a VllmConfig
    with kv_events_config set still resolves disable_hybrid_kv_cache_manager
    to False (i.e. HMA remains enabled) when no other condition requires it
    to be disabled.
    """
    model_config = ModelConfig(max_model_len=16)
    kv_events_config = KVEventsConfig(
        enable_kv_cache_events=True,
        publisher="null",
    )

    # Leave disable_hybrid_kv_cache_manager as None (the default) so that
    # VllmConfig.__post_init__ resolves it automatically.
    vllm_config = VllmConfig(
        model_config=model_config,
        kv_events_config=kv_events_config,
    )

    assert vllm_config.scheduler_config.disable_hybrid_kv_cache_manager is False, (
        "kv_events_config must not force-disable the hybrid KV cache manager."
    )
```
**EN:** Test case covering `hma not disabled when KV events enabled`. It exercises `ModelConfig, KVEventsConfig, VllmConfig`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `hma not disabled when kv events enabled` 的测试用例。 该测试会调用 `ModelConfig, KVEventsConfig, VllmConfig`。 代码主体包含 1 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖
- **EN:** Scheduler state transitions and queue management
- **CN:** 调度器状态迁移与队列管理
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm.v1.core.kv_cache_utils, vllm.config, vllm.config.kv_events, vllm.lora.request, vllm.multimodal.inputs, vllm.sampling_params, vllm.utils.hashing, vllm.utils.mem_constants, ...`.
- **CN:** 被测试的 vLLM 模块：`vllm.v1.core.kv_cache_utils, vllm.config, vllm.config.kv_events, vllm.lora.request, vllm.multimodal.inputs, vllm.sampling_params, vllm.utils.hashing, vllm.utils.mem_constants, ...`。
- **EN:** Standard-library support: `hashlib, importlib, collections.abc, typing`.
- **CN:** 标准库支持：`hashlib, importlib, collections.abc, typing`。
