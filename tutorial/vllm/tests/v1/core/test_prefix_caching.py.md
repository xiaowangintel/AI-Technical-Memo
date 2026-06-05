# test_prefix_caching.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/core/test_prefix_caching.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Compare the with and without prefix caching. / 该文件的文档字符串表明其用途：`compare the with and without prefix caching`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (line 3)
```python
"""Compare the with and without prefix caching."""
```
**EN:** Module docstring that declares the scope of the file: Compare the with and without prefix caching.
**CN:** 模块文档字符串直接说明了文件范围：`compare the with and without prefix caching`。

### Imports and setup / 导入与设置 (lines 5-42)
```python
import copy
from collections.abc import Callable

import pytest
import torch
import vllm.v1.core.kv_cache_manager as kv_cache_manager
import vllm.v1.core.kv_cache_utils as kv_cache_utils
from vllm.distributed.kv_events import AllBlocksCleared, BlockRemoved, BlockStored
from vllm.lora.request import LoRARequest
from vllm.multimodal.inputs import (
    MultiModalFeatureSpec,
    MultiModalKwargsItem,
    PlaceholderRange,
)
from vllm.sampling_params import SamplingParams
from vllm.utils.hashing import sha256, sha256_cbor
from vllm.v1.core.block_pool import BlockHashToBlockMap, BlockPool
# ... excerpt omitted for brevity ...
    KVCacheConfig,
    KVCacheGroupSpec,
    KVCacheSpecKind,
    MambaSpec,
    SlidingWindowSpec,
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm.v1.core.kv_cache_manager, vllm.v1.core.kv_cache_utils, vllm.distributed.kv_events, vllm.lora.request, vllm.multimodal.inputs, ...`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm.v1.core.kv_cache_manager, vllm.v1.core.kv_cache_utils, vllm.distributed.kv_events, vllm.lora.request, vllm.multimodal.inputs, ...`。

### Module state / 模块级状态 (line 44)
```python
pytestmark = pytest.mark.cpu_test
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `pytestmark`. It also sets pytest marks that scope the whole file.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`pytestmark`。 该块还设置了作用于整个文件的 pytest 标记。

### _auto_init_hash_fn (lines 48-54)
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

### make_request (lines 57-92)
```python
def make_request(
    request_id: str,
    prompt_token_ids: list[int],
    block_size: int,
    hash_fn: Callable,
    mm_positions: list[PlaceholderRange] | None = None,
    mm_hashes: list[str] | None = None,
    prompt_logprobs: int | None = None,
    cache_salt: str | None = None,
    lora_request: LoRARequest | None = None,
):
    mm_features = []
    if mm_positions is not None:
        for j, position in enumerate(mm_positions):
            identifier = mm_hashes[j] if mm_hashes else f"hash_{j}"
            mm_feature = MultiModalFeatureSpec(
                data=MultiModalKwargsItem.dummy(),
                mm_position=position,
    # ... excerpt omitted for brevity ...
    return Request(
        sampling_params=sampling_params,
        pooling_params=None,
        lora_request=lora_request,
        cache_salt=cache_salt,
        block_hasher=get_request_block_hasher(block_size, hash_fn),
    )
```
**EN:** Helper function `make_request` encapsulates reusable logic for `request`. Inputs: `request_id, prompt_token_ids, block_size, hash_fn, mm_positions, mm_hashes, prompt_logprobs, cache_salt, ...`. Key calls include `SamplingParams, sampling_params.update_from_generation_config, Request, enumerate, MultiModalFeatureSpec, mm_features.append`.
**CN:** 辅助函数 `make_request` 封装了与 `request` 相关的可复用逻辑。 输入参数：`request_id, prompt_token_ids, block_size, hash_fn, mm_positions, mm_hashes, prompt_logprobs, cache_salt, ...`。 关键调用包括 `SamplingParams, sampling_params.update_from_generation_config, Request, enumerate, MultiModalFeatureSpec, mm_features.append`。

### make_kv_cache_config (lines 95-110)
```python
def make_kv_cache_config(block_size: int, num_blocks: int) -> KVCacheConfig:
    return KVCacheConfig(
        num_blocks=num_blocks,
        kv_cache_tensors=[],
        kv_cache_groups=[
            KVCacheGroupSpec(
                ["layer"],
                FullAttentionSpec(
                    block_size=block_size,
                    num_kv_heads=1,
                    head_size=1,
                    dtype=torch.float32,
                ),
            )
        ],
    )
```
**EN:** Helper function `make_kv_cache_config` encapsulates reusable logic for `KV cache config`. Inputs: `block_size, num_blocks`. Key calls include `KVCacheConfig, KVCacheGroupSpec, FullAttentionSpec`.
**CN:** 辅助函数 `make_kv_cache_config` 封装了与 `KV 缓存 config` 相关的可复用逻辑。 输入参数：`block_size, num_blocks`。 关键调用包括 `KVCacheConfig, KVCacheGroupSpec, FullAttentionSpec`。

### make_kv_cache_config_hybrid_model (lines 113-156)
```python
def make_kv_cache_config_hybrid_model(
    block_size: int,
    num_blocks: int,
    sliding_window_blocks: int,
    second_spec_type: str = "sliding_window",
) -> KVCacheConfig:
    if second_spec_type == "sliding_window":
        second_spec = SlidingWindowSpec(
            block_size=block_size,
            num_kv_heads=1,
            head_size=1,
            dtype=torch.float32,
            sliding_window=sliding_window_blocks * block_size,
        )
    elif second_spec_type == "mamba":
        second_spec = MambaSpec(
            shapes=(1, 1),
    # ... excerpt omitted for brevity ...
    return KVCacheConfig(
            KVCacheGroupSpec(
                ["layer3"],
                second_spec,
            ),
        ],
    )
```
**EN:** Helper function `make_kv_cache_config_hybrid_model` encapsulates reusable logic for `KV cache config hybrid model`. Inputs: `block_size, num_blocks, sliding_window_blocks, second_spec_type`. Key calls include `KVCacheConfig, SlidingWindowSpec, MambaSpec, KVCacheGroupSpec, FullAttentionSpec`.
**CN:** 辅助函数 `make_kv_cache_config_hybrid_model` 封装了与 `KV 缓存 config hybrid model` 相关的可复用逻辑。 输入参数：`block_size, num_blocks, sliding_window_blocks, second_spec_type`。 关键调用包括 `KVCacheConfig, SlidingWindowSpec, MambaSpec, KVCacheGroupSpec, FullAttentionSpec`。

### make_kv_cache_config_three_types (lines 159-205)
```python
def make_kv_cache_config_three_types(
    block_size: int, num_blocks: int, third_spec_type: str = "mamba"
) -> KVCacheConfig:
    if third_spec_type == "mamba":
        third_spec = MambaSpec(
            block_size=block_size,
            shapes=(1, 1),
            dtypes=(torch.float32,),
        )
    elif third_spec_type == "sliding_window":
        third_spec = SlidingWindowSpec(
            num_kv_heads=1,
            head_size=1,
            dtype=torch.float32,
            sliding_window=4 * block_size,

    # ... excerpt omitted for brevity ...
    return KVCacheConfig(
            KVCacheGroupSpec(
                ["layer3"],
                third_spec,
            ),
        ],
    )
```
**EN:** Helper function `make_kv_cache_config_three_types` encapsulates reusable logic for `KV cache config three types`. Inputs: `block_size, num_blocks, third_spec_type`. Key calls include `KVCacheConfig, MambaSpec, SlidingWindowSpec, KVCacheGroupSpec, FullAttentionSpec`.
**CN:** 辅助函数 `make_kv_cache_config_three_types` 封装了与 `KV 缓存 config three types` 相关的可复用逻辑。 输入参数：`block_size, num_blocks, third_spec_type`。 关键调用包括 `KVCacheConfig, MambaSpec, SlidingWindowSpec, KVCacheGroupSpec, FullAttentionSpec`。

### test_prefill (lines 209-329)
```python
def test_prefill(hash_fn):
    block_size = 16
    manager = KVCacheManager(
        make_kv_cache_config(block_size, 11),
        max_model_len=8192,
        enable_caching=True,
        hash_block_size=block_size,
    )

    # Complete 3 blocks (48 tokens)
    common_token_ids = [i for i in range(3) for _ in range(16)]
    # Fully cache miss
    # Incomplete 1 block (7 tokens)
    unique_token_ids = [3] * 7
    all_token_ids = common_token_ids + unique_token_ids
    req0 = make_request("0", all_token_ids, block_size, hash_fn)
    computed_blocks, num_computed_tokens = manager.get_computed_blocks(req0)
    # ... excerpt omitted for brevity ...
    assert len(req0.block_hashes) == 3
    assert not computed_blocks.blocks[0]
    assert num_computed_tokens == 0
    assert blocks is not None and blocks.get_block_ids() == ([1, 2, 3, 4],)
        assert blk_hash is not None
        assert get_block_hash(blk_hash) == block_hash
        is free_block_queue.fake_free_list_tail
    assert (
        free_block_queue.fake_free_list_tail.prev_free_block
        is free_block_queue.fake_free_list_head
```
**EN:** Parameterized test covering `prefill`. Parameter axes: `hash_fn`. Inputs/fixtures: `hash_fn`. It exercises `mark.parametrize, KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free`. The body contains 31 explicit assertion(s).
**CN:** 该代码块是覆盖 `prefill` 的测试用例。 参数维度：`hash_fn`。 输入或 fixture：`hash_fn`。 该测试会调用 `mark.parametrize, KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free`。 代码主体包含 31 个显式断言。

### test_prefill_hybrid_model (lines 332-497)
```python
def test_prefill_hybrid_model():
    block_size = 16
    manager = KVCacheManager(
        make_kv_cache_config_hybrid_model(block_size, 21, 2),
        max_model_len=8192,
        enable_caching=True,
        hash_block_size=block_size,
    )

    hash_fn = sha256
    # Complete 3 blocks (48 tokens)
    num_full_blocks = 3
    common_token_ids = [i for i in range(num_full_blocks) for _ in range(block_size)]
    # Fully cache miss
    # Incomplete 1 block (7 tokens)
    unique_token_ids = [3] * 7
    # ... excerpt omitted for brevity ...
    assert len(req0.block_hashes) == 3
    assert not computed_blocks.blocks[0]
    assert num_computed_tokens == 0
    assert blocks is not None and blocks.get_block_ids() == (
            assert blk_hash is not None
            assert get_block_hash(blk_hash) == block_hash
            make_block_hash_with_group_id(block_hashes[2], 0),
            make_block_hash_with_group_id(block_hashes[0], 1),
            make_block_hash_with_group_id(block_hashes[0], 2),
        ],
        0,
```
**EN:** Test case covering `prefill hybrid model`. It exercises `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, zip, manager.free`. The body contains 15 explicit assertion(s).
**CN:** 该代码块是覆盖 `prefill hybrid model` 的测试用例。 该测试会调用 `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, zip, manager.free`。 代码主体包含 15 个显式断言。

### test_prefill_hybrid_model_eagle (lines 500-677)
```python
def test_prefill_hybrid_model_eagle():
    block_size = 16
    kv_cache_config = make_kv_cache_config_hybrid_model(block_size, 31, 3)
    manager = KVCacheManager(
        kv_cache_config,
        max_model_len=8192,
        enable_caching=True,
        hash_block_size=block_size,
        use_eagle=True,
    )

    hash_fn = sha256
    # Complete 6 blocks (96 tokens)
    num_full_blocks = 6
    common_token_ids = [i for i in range(num_full_blocks) for _ in range(block_size)]
    # Fully cache miss
    # ... excerpt omitted for brevity ...
    assert len(req0.block_hashes) == len(all_token_ids) // block_size
    assert not computed_blocks.blocks[0]
    assert num_computed_tokens == 0
    assert blocks is not None and blocks.get_block_ids() == block_ids
            assert blk_hash is not None
            assert get_block_hash(blk_hash) == block_hash
            make_block_hash_with_group_id(block_hashes[-1], 0),
            make_block_hash_with_group_id(block_hashes[0], 1),
            make_block_hash_with_group_id(block_hashes[0], 2),
        ],
        4,
```
**EN:** Test case covering `prefill hybrid model eagle`. It exercises `make_kv_cache_config_hybrid_model, KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, enumerate`. The body contains 15 explicit assertion(s).
**CN:** 该代码块是覆盖 `prefill hybrid model eagle` 的测试用例。 该测试会调用 `make_kv_cache_config_hybrid_model, KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, enumerate`。 代码主体包含 15 个显式断言。

### _test_partial_request_hit (lines 680-704)
```python
def _test_partial_request_hit(
    manager: KVCacheManager,
    block_size: int,
    num_full_blocks,
    request_id: str,
    prompt_token_ids: list[int],
    hash_to_evict: list[BlockHashWithGroupId],
    expect_hit_length: int,
):
    cached_block_hash_to_block_bak = copy.copy(
        manager.block_pool.cached_block_hash_to_block._cache
    )
    req = make_request(request_id, prompt_token_ids, block_size, sha256)
    for hash_with_group_id in hash_to_evict:
        manager.block_pool.cached_block_hash_to_block._cache.pop(hash_with_group_id)
    computed_blocks, num_computed_tokens = manager.get_computed_blocks(req)
    assert len(req.block_hashes) == num_full_blocks
    assert num_computed_tokens == expect_hit_length * block_size
    for block_per_group in computed_blocks.blocks:
        assert len(block_per_group) == num_computed_tokens // block_size
    for hash_with_group_id in hash_to_evict:
        manager.block_pool.cached_block_hash_to_block._cache[hash_with_group_id] = (
            cached_block_hash_to_block_bak[hash_with_group_id]
        )
    manager.free(req)
```
**EN:** Helper function `_test_partial_request_hit` encapsulates reusable logic for `partial request hit`. Inputs: `manager, block_size, num_full_blocks, request_id, prompt_token_ids, hash_to_evict, expect_hit_length`. Key calls include `copy.copy, make_request, manager.get_computed_blocks, manager.free, _cache.pop, len`. It includes 3 internal assertion(s) to guard assumptions.
**CN:** 辅助函数 `_test_partial_request_hit` 封装了与 `partial request hit` 相关的可复用逻辑。 输入参数：`manager, block_size, num_full_blocks, request_id, prompt_token_ids, hash_to_evict, expect_hit_length`。 关键调用包括 `copy.copy, make_request, manager.get_computed_blocks, manager.free, _cache.pop, len`。 其中包含 3 个内部断言，用于保护前置假设。

### _make_hybrid_kv_cache_config (lines 707-765)
```python
def _make_hybrid_kv_cache_config(
    block_size: int, num_blocks: int, spec_types: list[str]
) -> KVCacheConfig:
    """
    Create a KVCacheConfig with the specified spec types.

    Args:
        block_size: The block size for KV cache.
        num_blocks: The number of blocks in the KV cache.
        spec_types: List of spec type strings. Supported types:
            - "full": FullAttentionSpec
            - "sliding_window": SlidingWindowSpec with window=2*block_size
            - "sliding_window_large": SlidingWindowSpec with window=4*block_size
            - "mamba": MambaSpec
    spec_map = {
        "full": lambda: FullAttentionSpec(
            block_size=block_size,
    # ... excerpt omitted for brevity ...
    return KVCacheConfig(
        num_blocks=num_blocks,
        kv_cache_tensors=[],
        kv_cache_groups=kv_cache_groups,
    )
```
**EN:** Helper function `_make_hybrid_kv_cache_config` encapsulates reusable logic for `hybrid KV cache config`. Inputs: `block_size, num_blocks, spec_types`. Key calls include `KVCacheConfig, KVCacheGroupSpec, FullAttentionSpec, SlidingWindowSpec, MambaSpec, spec_map`.
**CN:** 辅助函数 `_make_hybrid_kv_cache_config` 封装了与 `hybrid KV 缓存 config` 相关的可复用逻辑。 输入参数：`block_size, num_blocks, spec_types`。 关键调用包括 `KVCacheConfig, KVCacheGroupSpec, FullAttentionSpec, SlidingWindowSpec, MambaSpec, spec_map`。

### Module state / 模块级状态 (lines 774-817)
```python
_HYBRID_MODEL_TEST_CASES = [
    # 2 groups: 1 full + 1 other
    pytest.param(["full", "sliding_window"], id="2g-full+sw"),
    pytest.param(["full", "mamba"], id="2g-full+mamba"),
    # 2 groups: 0 full (all other types)
    pytest.param(["sliding_window", "mamba"], id="2g-sw+mamba"),
    pytest.param(["sliding_window", "sliding_window_large"], id="2g-sw+sw_large"),
    # 3 groups: 1 full + 2 others (same type)
    pytest.param(["full", "sliding_window", "sliding_window"], id="3g-full+2sw"),
    pytest.param(["full", "mamba", "mamba"], id="3g-full+2mamba"),
    # 3 groups: 1 full + 2 others (different types)
    pytest.param(["full", "sliding_window", "mamba"], id="3g-full+sw+mamba"),
    pytest.param(
        ["full", "sliding_window", "sliding_window_large"],
        id="3g-full+sw+sw_large",
    ),
    # 3 groups: 2 full + 1 other
    pytest.param(["full", "full", "sliding_window"], id="3g-2full+sw"),
    # ... excerpt omitted for brevity ...
    # 4 groups: 2 full + 2 others (grouped)
        ["full", "full", "sliding_window", "mamba"],
        id="4g-2full+sw+mamba",
]
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `_HYBRID_MODEL_TEST_CASES`. Shared setup calls include `pytest.param`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`_HYBRID_MODEL_TEST_CASES`。 共享初始化调用包括 `pytest.param`。

### test_prefill_hybrid_model_combinations (lines 821-890)
```python
def test_prefill_hybrid_model_combinations(spec_types: list[str]):
    """
    Test prefix caching with hybrid models containing various combinations of
    KV cache spec types.

    This unified test covers:
    - Various combinations (full attn + other attn types)
    - Varying number of groups (2, 3, or 4)
    - 0, 1, or 2 full attention groups in the combination
    - Two sliding_window attn groups with different window sizes
    - Interleaved group IDs (full attn and other types alternating)
    - Mamba spec with other attention types
    block_size = 16
    num_groups = len(spec_types)
    # Allocate enough blocks for all groups
    num_blocks = 10 * num_groups
    # ... excerpt omitted for brevity ...
    assert len(req0.block_hashes) == 3
    assert not computed_blocks.blocks[0]  # No cache hit initially
    assert num_computed_tokens == 0
    assert blocks is not None
    assert len(blocks.get_block_ids()) == num_groups
    assert num_computed_tokens == 3 * block_size
    )
    manager.free(req0)
    manager.free(req1)
```
**EN:** Parameterized test covering `prefill hybrid model combinations`. Parameter axes: `spec_types`. Inputs/fixtures: `spec_types`. It exercises `mark.parametrize, len, _make_hybrid_kv_cache_config, KVCacheManager, make_request, manager.get_computed_blocks`. The body contains 9 explicit assertion(s).
**CN:** 该代码块是覆盖 `prefill hybrid model combinations` 的测试用例。 参数维度：`spec_types`。 输入或 fixture：`spec_types`。 该测试会调用 `mark.parametrize, len, _make_hybrid_kv_cache_config, KVCacheManager, make_request, manager.get_computed_blocks`。 代码主体包含 9 个显式断言。

### Module state / 模块级状态 (lines 895-898)
```python
_EAGLE_HYBRID_MODEL_TEST_CASES = [
    # 2 groups: 1 full + 1 other
    pytest.param(["full", "sliding_window"], 3, id="2g-full+sw"),
]
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `_EAGLE_HYBRID_MODEL_TEST_CASES`. Shared setup calls include `pytest.param`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`_EAGLE_HYBRID_MODEL_TEST_CASES`。 共享初始化调用包括 `pytest.param`。

### test_prefill_hybrid_model_combinations_eagle (lines 902-969)
```python
def test_prefill_hybrid_model_combinations_eagle(
    spec_types: list[str], expect_hit_length: int
):
    """
    Test prefix caching with hybrid models (1 full attn + 1 other) with EAGLE.
    More complex hybrid models with EAGLE are not yet supported (see issue #32802).
    block_size = 16
    num_groups = len(spec_types)
    # Allocate enough blocks for all groups
    num_blocks = 10 * num_groups

    kv_cache_config = _make_hybrid_kv_cache_config(block_size, num_blocks, spec_types)
    manager = KVCacheManager(
        kv_cache_config,
        max_model_len=8192,
        enable_caching=True,
        hash_block_size=block_size,
    # ... excerpt omitted for brevity ...
    assert len(req0.block_hashes) == num_full_blocks
    assert not computed_blocks.blocks[0]  # No cache hit initially
    assert num_computed_tokens == 0
    assert blocks is not None
    assert len(blocks.get_block_ids()) == num_groups
    assert num_computed_tokens == expect_hit_length * block_size
    )
    manager.free(req0)
    manager.free(req1)
```
**EN:** Parameterized test covering `prefill hybrid model combinations eagle`. Parameter axes: `spec_types, expect_hit_length`. Inputs/fixtures: `spec_types, expect_hit_length`. It exercises `mark.parametrize, len, _make_hybrid_kv_cache_config, KVCacheManager, make_request, manager.get_computed_blocks`. The body contains 10 explicit assertion(s).
**CN:** 该代码块是覆盖 `prefill hybrid model combinations eagle` 的测试用例。 参数维度：`spec_types, expect_hit_length`。 输入或 fixture：`spec_types, expect_hit_length`。 该测试会调用 `mark.parametrize, len, _make_hybrid_kv_cache_config, KVCacheManager, make_request, manager.get_computed_blocks`。 代码主体包含 10 个显式断言。

### test_prefill_hybrid_model_mamba_align (lines 972-1009)
```python
def test_prefill_hybrid_model_mamba_align():
    """Test that MambaManager.cache_blocks() handles null blocks in align mode.

    Regression test for https://github.com/vllm-project/vllm/issues/34361.
    In mamba_cache_mode="align", allocate_new_blocks() pads req_to_blocks with
    null blocks. cache_full_blocks() correctly skips them, but
    MambaManager.cache_blocks() must also skip null blocks when tracking
    cached_blocks_this_step.
    """
    block_size = 16
    num_blocks = 30
    kv_cache_config = _make_hybrid_kv_cache_config(
        block_size, num_blocks, ["full", "mamba_align"]
    )
    manager = KVCacheManager(
        kv_cache_config,
        max_model_len=8192,
    # ... excerpt omitted for brevity ...
    assert num_computed_tokens == 0
    blocks = manager.allocate_slots(req0, 55, num_computed_tokens, computed_blocks)
    assert blocks is not None
    assert len(blocks.get_block_ids()) == 2  # full_attn + mamba groups
    manager.free(req0)
```
**EN:** Test case covering `prefill hybrid modelMamba align`. It exercises `_make_hybrid_kv_cache_config, KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `prefill hybrid modelmamba align` 的测试用例。 该测试会调用 `_make_hybrid_kv_cache_config, KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free`。 代码主体包含 3 个显式断言。

### test_prefill_plp (lines 1012-1123)
```python
def test_prefill_plp():
    """Test prefill with APC and some prompt logprobs (plp) requests.

    1. Schedule plp request and validate APC block allocation
    2. Schedule non-plp request and validate blocks
    3. Schedule plp request; no hit should occur; validate blocks
    """
    block_size = 16
    manager = KVCacheManager(
        make_kv_cache_config(block_size, 11),
        max_model_len=8192,
        enable_caching=True,
        hash_block_size=block_size,
    )
    # the default hash function is sha256
    hash_fn = sha256
    # Complete 3 blocks (48 tokens)
    # ... excerpt omitted for brevity ...
    assert len(req0.block_hashes) == 3
    assert not computed_blocks.blocks[0]
    assert num_computed_tokens == 0
    assert blocks is not None and blocks.get_block_ids() == ([1, 2, 3, 4],)
        assert blk_hash is not None
        assert get_block_hash(blk_hash) == block_hash
    # Request #2 block hashes are valid since request #0 hashes are.
    # Check block reference counts.
    for block_id in block_ids[0]:
        assert manager.block_pool.blocks[block_id].ref_cnt == 1
    manager.free(req2)
```
**EN:** Test case covering `prefill plp`. It exercises `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free, blocks.get_block_ids`. The body contains 25 explicit assertion(s).
**CN:** 该代码块是覆盖 `prefill plp` 的测试用例。 该测试会调用 `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free, blocks.get_block_ids`。 代码主体包含 25 个显式断言。

### test_decode (lines 1126-1186)
```python
def test_decode():
    block_size = 16
    manager = KVCacheManager(
        make_kv_cache_config(block_size, 11),
        max_model_len=8192,
        enable_caching=True,
        hash_block_size=block_size,
    )

    # Complete 3 blocks (48 tokens)
    common_token_ids = [i for i in range(3) for _ in range(16)]
    # Fully cache miss
    # Incomplete 1 block (7 tokens)
    unique_token_ids = [3] * 7
    req0 = make_request("0", common_token_ids + unique_token_ids, block_size, sha256)
    computed_blocks, num_computed_tokens = manager.get_computed_blocks(req0)
    assert not computed_blocks.blocks[0]
    # ... excerpt omitted for brevity ...
    assert num_computed_tokens == 0
    assert blocks is not None and blocks.get_block_ids() == ([1, 2, 3, 4],)
    assert new_blocks is not None and len(new_blocks.blocks[0]) == 0
    assert (
    assert new_blocks is not None and len(new_blocks.blocks[0]) == 1
        manager.coordinator.single_type_managers[0]
        .req_to_blocks[req0.request_id][-1]
        .block_hash
        is None
```
**EN:** Test case covering `decode`. It exercises `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, range, make_kv_cache_config`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `decode` 的测试用例。 该测试会调用 `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, range, make_kv_cache_config`。 代码主体包含 8 个显式断言。

### test_evict (lines 1189-1241)
```python
def test_evict():
    block_size = 16
    manager = KVCacheManager(
        make_kv_cache_config(block_size, 11),
        max_model_len=8192,
        enable_caching=True,
        hash_block_size=block_size,
    )

    last_token_id = 5 * 16 + 7
    req0 = make_request("0", list(range(last_token_id)), block_size, sha256)
    computed_blocks, num_computed_tokens = manager.get_computed_blocks(req0)
    assert not computed_blocks.blocks[0]
    assert num_computed_tokens == 0
    blocks = manager.allocate_slots(
        req0, 5 * 16 + 7, len(computed_blocks.blocks[0]) * 16, computed_blocks
    # 5 full + 1 partial
    # ... excerpt omitted for brevity ...
    assert blocks is not None and len(blocks.blocks[0]) == 6
    assert blocks is not None and len(blocks.blocks[0]) == 3  # 3 full blocks
    assert manager.block_pool.free_block_queue.num_free_blocks == 1
    assert manager.block_pool.free_block_queue.num_free_blocks == 10
    assert num_computed_tokens == 2 * 16
        req2, 3, len(computed_blocks.blocks[0]) * 16, computed_blocks
    assert blocks is not None and blocks.get_block_ids() == ([10],)
    assert manager.block_pool.free_block_queue.num_free_blocks == 7
```
**EN:** Test case covering `evict`. It exercises `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free, make_kv_cache_config`. The body contains 13 explicit assertion(s).
**CN:** 该代码块是覆盖 `evict` 的测试用例。 该测试会调用 `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free, make_kv_cache_config`。 代码主体包含 13 个显式断言。

### test_hash_block_correct_reuse (lines 1244-1282)
```python
def test_hash_block_correct_reuse():
    """
    This tests when a previously cached block is reused as a new block,
    its hash metadata should be correctly reset.
    block_size = 16
    manager = KVCacheManager(
        make_kv_cache_config(16, 2),
        max_model_len=8192,
        enable_caching=True,
        hash_block_size=block_size,
    )

    # Allocate 1 block and cache it.
    num_tokens = block_size * 1
    req = make_request("0", list(range(num_tokens)), block_size, sha256)
    computed_blocks, num_computed_tokens = manager.get_computed_blocks(req)
    assert not computed_blocks.blocks[0]
    # ... excerpt omitted for brevity ...
    assert num_computed_tokens == 0
    assert blocks is not None and len(blocks.blocks[0]) == 1
    blocks = manager.allocate_slots(
        req, num_tokens - 1, len(computed_blocks.blocks[0]) * 16, computed_blocks
    assert manager.block_pool.blocks[blocks.blocks[0][0].block_id].block_hash is None
```
**EN:** Test case covering `hash block correct reuse`. It exercises `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free, make_kv_cache_config`. The body contains 7 explicit assertion(s).
**CN:** 该代码块是覆盖 `hash block correct reuse` 的测试用例。 该测试会调用 `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free, make_kv_cache_config`。 代码主体包含 7 个显式断言。

### test_computed_blocks_not_evicted (lines 1285-1342)
```python
def test_computed_blocks_not_evicted():
    """
    Test that the computed blocks are not evicted when getting new blocks
    for a request if there are any other free blocks.
    block_size = 16
    manager = KVCacheManager(
        make_kv_cache_config(block_size, 3),
        max_model_len=8192,
        enable_caching=True,
        hash_block_size=block_size,
    )

    # Allocate a block and cache it.
    num_tokens = block_size * 1
    req0 = make_request("0", list(range(num_tokens)), block_size, sha256)
    computed_blocks, num_computed_tokens = manager.get_computed_blocks(req0)
    assert not computed_blocks.blocks[0]
    # ... excerpt omitted for brevity ...
    assert num_computed_tokens == 0
    assert blocks is not None and len(blocks.blocks[0]) == 1
    assert blocks.blocks[0][0].block_id == 1
        num_tokens * 2 - num_tokens,
        len(computed_blocks.blocks[0]) * 16,
        computed_blocks,
    assert blocks.blocks[0][0].block_id == 2
```
**EN:** Test case covering `computed blocks not evicted`. It exercises `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free, make_kv_cache_config`. The body contains 13 explicit assertion(s).
**CN:** 该代码块是覆盖 `computed blocks not evicted` 的测试用例。 该测试会调用 `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free, make_kv_cache_config`。 代码主体包含 13 个显式断言。

### test_basic_prefix_caching_disabled (lines 1345-1390)
```python
def test_basic_prefix_caching_disabled():
    """
    This tests that the prefix caching is disabled.
    block_size = 4
    manager = KVCacheManager(
        make_kv_cache_config(block_size, 5),
        max_model_len=8192,
        enable_caching=False,
        hash_block_size=block_size,
    )

    req1 = make_request(
        "1", list(range(10)), block_size, sha256
    )  # 2 blocks and some more
    computed_blocks, num_computed_tokens = manager.get_computed_blocks(req1)
    assert not computed_blocks.blocks[0]
    # ... excerpt omitted for brevity ...
    assert num_computed_tokens == 0
    assert blocks is not None and len(blocks.blocks[0]) == 3
    assert blocks is not None and len(blocks.blocks[0]) == 4
    blocks = manager.allocate_slots(
        req3, 4, len(computed_blocks.blocks[0]) * 16, computed_blocks
    assert not blocks
```
**EN:** Test case covering `basic prefix caching disabled`. It exercises `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free, make_kv_cache_config`. The body contains 9 explicit assertion(s).
**CN:** 该代码块是覆盖 `basic 前缀缓存 disabled` 的测试用例。 该测试会调用 `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free, make_kv_cache_config`。 代码主体包含 9 个显式断言。

### test_cache_blocks (lines 1394-1440)
```python
def test_cache_blocks(hash_fn):
    """
    This is a unit test that tests the correctness of the _cache_full_blocks
    function of KVCacheManager.

    block_size = 4
    block_pool = BlockPool(
        num_gpu_blocks=5,
        enable_caching=True,
        hash_block_size=block_size,
    )
    # Req:
    #  Block 0: [0, 1, 2, 3]
    #  Block 1: [4, 5, 6, 7]
    #  Block 2: [8, 9, 10, 11]
    #  Block 3: [12, 13]
    req = make_request("0", list(range(14)), block_size, hash_fn)
    # ... excerpt omitted for brevity ...
    assert len(block_pool.cached_block_hash_to_block) == 2
    assert all([block.block_hash is not None for block in blocks])
        num_full_blocks=3,
        block_size=block_size,
        kv_cache_group_id=0,
    assert len(block_pool.cached_block_hash_to_block) == 3
    assert blocks[0].block_hash is not None
```
**EN:** Parameterized test covering `cache blocks`. Parameter axes: `hash_fn`. Inputs/fixtures: `hash_fn`. It exercises `mark.parametrize, BlockPool, make_request, block_pool.cache_full_blocks, all, list`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `缓存 blocks` 的测试用例。 参数维度：`hash_fn`。 输入或 fixture：`hash_fn`。 该测试会调用 `mark.parametrize, BlockPool, make_request, block_pool.cache_full_blocks, all, list`。 代码主体包含 4 个显式断言。

### test_cache_blocks_multi_group (lines 1443-1525)
```python
def test_cache_blocks_multi_group():
    """
    This tests that blocks are cached correctly for different kv cache groups.
    block_size = 4
    block_pool = BlockPool(
        num_gpu_blocks=10, enable_caching=True, hash_block_size=block_size
    )

    # Req:
    #  Block 0/4: [0, 1, 2, 3]
    #  Block 1/5: [4, 5, 6, 7]
    #  Block 2/6: [8, 9, 10, 11]
    #  Block 3/7: [12, 13]
    req = make_request("0", list(range(14)), block_size, sha256)
    # Cache the blocks for group 0.
    blocks = [KVCacheBlock(block_id=i) for i in range(2)]
    # ... excerpt omitted for brevity ...
    assert len(block_pool.cached_block_hash_to_block) == 2
    assert len(req.block_hashes) == 3
    assert all([block.block_hash is not None for block in blocks])
    assert len(block_pool.cached_block_hash_to_block) == 5
        is not None
    assert (
        block_pool.get_cached_block(req.block_hashes[2], kv_cache_group_ids=[0, 1])
        is None
```
**EN:** Test case covering `cache blocks multi group`. It exercises `BlockPool, make_request, block_pool.cache_full_blocks, all, list, KVCacheBlock`. The body contains 15 explicit assertion(s).
**CN:** 该代码块是覆盖 `缓存 blocks multi group` 的测试用例。 该测试会调用 `BlockPool, make_request, block_pool.cache_full_blocks, all, list, KVCacheBlock`。 代码主体包含 15 个显式断言。

### test_mm_prefix_caching (lines 1528-1633)
```python
def test_mm_prefix_caching():
    """
    This tests that the multi-modal prefix caching is correct.

    block_size = 16
    manager = KVCacheManager(
        make_kv_cache_config(block_size, 11),
        max_model_len=8192,
        enable_caching=True,
        hash_block_size=block_size,
    )
    # Common prompt tokens (T is text tokens and P is image placeholder tokens)
    # [T,...,T, P0,...,P0], [P0,...,P0,T,...,T,P1,...,P1], [P1,...,P1]
    common_token_ids = list(range(10)) + [-1] * 6
    common_token_ids += [-1] * 4 + list(range(10, 20)) + [-1] * 2
    common_token_ids += [-1] * 16
    # ... excerpt omitted for brevity ...
    assert not computed_blocks.blocks[0]
    assert num_computed_tokens == 0
    assert len(block_hashes) == 3
    assert block_hashes[0] == sha256(
    assert block_hashes[1] == sha256(
    assert block_hashes[2] == sha256(
        mm_positions=mm_positions,
        mm_hashes=mm_hashes,
    computed_blocks, num_computed_tokens = manager.get_computed_blocks(req1)
    assert len(computed_blocks.blocks[0]) == 3
    assert num_computed_tokens == 3 * 16
```
**EN:** Test case covering `mm prefix caching`. It exercises `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, range, make_kv_cache_config`. The body contains 13 explicit assertion(s).
**CN:** 该代码块是覆盖 `mm 前缀缓存` 的测试用例。 该测试会调用 `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, range, make_kv_cache_config`。 代码主体包含 13 个显式断言。

### test_cache_key_salting (lines 1636-1713)
```python
def test_cache_key_salting():
    """
    This tests that cache salts are applied during hashing and the cache
    is separated cache as expected.
    block_size = 16
    manager = KVCacheManager(
        make_kv_cache_config(block_size, 11),
        max_model_len=8192,
        enable_caching=True,
        hash_block_size=block_size,
    )

    # 3 complete blocks and an incomplete block with 11 tokens.
    common_token_ids = [i for i in range(3) for _ in range(block_size)]
    token_ids = common_token_ids + [3] * 11
    req0 = make_request("0", token_ids, block_size, sha256, cache_salt="salt1")
    computed_blocks, num_computed_tokens = manager.get_computed_blocks(req0)
    # ... excerpt omitted for brevity ...
    assert not computed_blocks.blocks[0]
    assert num_computed_tokens == 0
    assert len(block_hashes) == 3
    assert block_hashes[0] == sha256(
    assert block_hashes[1] == sha256(
    assert block_hashes[2] == sha256(
        (block_hashes[0], tuple(token_ids[block_size : block_size * 2]), None)
        (block_hashes[1], tuple(token_ids[block_size * 2 : block_size * 3]), None)
```
**EN:** Test case covering `cache key salting`. It exercises `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, range, make_kv_cache_config`. The body contains 19 explicit assertion(s).
**CN:** 该代码块是覆盖 `缓存 key salting` 的测试用例。 该测试会调用 `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, range, make_kv_cache_config`。 代码主体包含 19 个显式断言。

### test_prefill_not_enough_free_blocks_with_computed_blocks (lines 1716-1792)
```python
def test_prefill_not_enough_free_blocks_with_computed_blocks():
    """
    This is a unit test that tests the correctness of the allocate_slots
    when there is not enough free blocks. Specifically, when a request
    has computed blocks but cannot be allocated due to not enough free blocks,
    the computed blocks should not be touched.
    block_size = 16
    manager = KVCacheManager(
        make_kv_cache_config(block_size, 11),
        max_model_len=8192,
        enable_caching=True,
        hash_block_size=block_size,
    )
    # Complete 3 blocks (48 tokens)
    # | Common-0 | Common-1 | Common-2 | ... |
    common_token_ids = [i for i in range(3) for _ in range(16)]
    req0 = make_request("0", common_token_ids, block_size, sha256)
    # ... excerpt omitted for brevity ...
    assert not computed_blocks.blocks[0]
    assert num_computed_tokens == 0
    assert computed_blocks.blocks[0] == block_part0
    assert num_computed_tokens == 3 * 16
    assert {block.ref_cnt for block in block_part1[:3]} == {1}
    assert {block.ref_cnt for block in block_part1[3:]} == {0}
        is None
    # Block 0-2 are used by Req 1.
    # Block 3-5 are free.
```
**EN:** Test case covering `prefill not enough free blocks with computed blocks`. It exercises `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free, make_kv_cache_config`. The body contains 14 explicit assertion(s).
**CN:** 该代码块是覆盖 `prefill not enough free blocks with computed blocks` 的测试用例。 该测试会调用 `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free, make_kv_cache_config`。 代码主体包含 14 个显式断言。

### test_reset_prefix_cache (lines 1795-1832)
```python
def test_reset_prefix_cache():
    block_size = 16
    manager = KVCacheManager(
        make_kv_cache_config(block_size, 11),
        max_model_len=8192,
        enable_caching=True,
        hash_block_size=block_size,
    )

    full_block_token_ids = [i for i in range(3) for _ in range(16)]
    unique_token_ids = [3] * 7
    all_token_ids = full_block_token_ids + unique_token_ids
    req0 = make_request("0", all_token_ids, block_size, sha256)
    blocks = manager.allocate_slots(req0, 55)
    assert blocks is not None and blocks.get_block_ids() == ([1, 2, 3, 4],)
    unique_token_ids = [4] * 7
    # ... excerpt omitted for brevity ...
    assert len(req1.block_hashes) == 3
    assert len(computed_blocks.blocks[0]) == 3
    assert blocks is not None and blocks.get_block_ids() == ([5],)
    assert not manager.reset_prefix_cache()
    assert manager.block_pool.cached_block_hash_to_block
    manager.free(req0)
    manager.free(req1)
    assert manager.reset_prefix_cache()
    assert not manager.block_pool.cached_block_hash_to_block
    assert all([blk.block_hash is None for blk in manager.block_pool.blocks])
```
**EN:** Test case covering `reset prefix cache`. It exercises `KVCacheManager, make_request, manager.allocate_slots, manager.get_computed_blocks, manager.free, manager.reset_prefix_cache`. The body contains 9 explicit assertion(s).
**CN:** 该代码块是覆盖 `reset prefix 缓存` 的测试用例。 该测试会调用 `KVCacheManager, make_request, manager.allocate_slots, manager.get_computed_blocks, manager.free, manager.reset_prefix_cache`。 代码主体包含 9 个显式断言。

### test_prefix_cache_stats_disabled (lines 1835-1858)
```python
def test_prefix_cache_stats_disabled():
    """Test that prefix_cache_stats is None when log_stats is False."""
    block_size = 16
    manager = KVCacheManager(
        make_kv_cache_config(block_size, 11),
        max_model_len=8192,
        enable_caching=True,
        hash_block_size=block_size,
        log_stats=False,  # Disable logging stats
    )
    assert manager.prefix_cache_stats is None

    # Call all functions that check whether log_stats is disabled.
    req = make_request("0", list(range(16)), block_size, sha256)
    computed_blocks, num_computed_tokens = manager.get_computed_blocks(req)
    assert not computed_blocks.blocks[0]
    assert num_computed_tokens == 0
    manager.allocate_slots(
        req, 16, len(computed_blocks.blocks[0]) * 16, computed_blocks
    )
    manager.reset_prefix_cache()

    # Ensure prefix_cache_stats remains None
    assert manager.prefix_cache_stats is None
```
**EN:** Test case covering `prefix cache stats disabled`. It exercises `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.reset_prefix_cache, make_kv_cache_config`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `prefix 缓存 stats disabled` 的测试用例。 该测试会调用 `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.reset_prefix_cache, make_kv_cache_config`。 代码主体包含 4 个显式断言。

### test_maybe_evict_cached_block (lines 1861-1906)
```python
def test_maybe_evict_cached_block():
    pool = BlockPool(num_gpu_blocks=4, enable_caching=True, hash_block_size=16)
    block_hash0 = make_block_hash_with_group_id(BlockHash(b"10"), 1000)
    block_hash1 = make_block_hash_with_group_id(BlockHash(b"20"), 2000)
    block_hash2 = make_block_hash_with_group_id(BlockHash(b"30"), 3000)
    block_hashes = [
        block_hash0,
        block_hash1,
        block_hash2,
        # block3 had the exact same block_hash as the first block
    ]
    assert len(pool.blocks) == len(block_hashes)
    # Manually add all blocks to cached_blocks
    for block, block_hash in zip(pool.blocks, block_hashes):
        block.block_hash = block_hash
        pool.cached_block_hash_to_block.insert(block_hash, block)

    # ... excerpt omitted for brevity ...
    assert pool.cached_block_hash_to_block._cache == {
    # Evict block2
    pool._maybe_evict_cached_block(block2)
    assert pool.cached_block_hash_to_block._cache == {block_hash0: {3: block3}}
    # Evict block3
    pool._maybe_evict_cached_block(block3)
    assert pool.cached_block_hash_to_block._cache == {}
```
**EN:** Test case covering `maybe evict cached block`. It exercises `BlockPool, make_block_hash_with_group_id, zip, pool._maybe_evict_cached_block, BlockHash, len`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `maybe evict cached block` 的测试用例。 该测试会调用 `BlockPool, make_block_hash_with_group_id, zip, pool._maybe_evict_cached_block, BlockHash, len`。 代码主体包含 6 个显式断言。

### test_kv_cache_events (lines 1910-1969)
```python
def test_kv_cache_events(blocks_to_cache: int):
    block_size = 16
    num_blocks = blocks_to_cache + 1

    # Allocate Blocks
    # Should see a single block stored event with a blocks_to_cache number of
    # block hashes
    # take_events should reset the kv_event_queue
    manager = KVCacheManager(
        make_kv_cache_config(block_size, num_blocks),
        max_model_len=8192,
        enable_caching=True,
        enable_kv_cache_events=True,
        hash_block_size=block_size,
    )
    num_tokens = block_size * blocks_to_cache
    req0 = make_request("0", list(range(num_tokens)), block_size, sha256)
    # ... excerpt omitted for brevity ...
    assert (
    assert len(block.token_ids) == block.block_size * len(block.block_hashes)
    assert block.kv_cache_spec_kind == KVCacheSpecKind.FULL_ATTENTION.value
    assert len(manager.block_pool.kv_event_queue) == 0
        assert isinstance(blocks, BlockRemoved)
        assert blocks.block_hashes[0] in stored_block_hash
    manager.free(req1)
    manager.reset_prefix_cache()
    events = manager.take_events()
    assert isinstance(events[-1], AllBlocksCleared)
    assert len(manager.block_pool.cached_block_hash_to_block) == 0
```
**EN:** Parameterized test covering `KV cache events`. Parameter axes: `blocks_to_cache`. Inputs/fixtures: `blocks_to_cache`. It exercises `mark.parametrize, KVCacheManager, make_request, manager.allocate_slots, manager.take_events, manager.free`. The body contains 11 explicit assertion(s).
**CN:** 该代码块是覆盖 `KV 缓存 events` 的测试用例。 参数维度：`blocks_to_cache`。 输入或 fixture：`blocks_to_cache`。 该测试会调用 `mark.parametrize, KVCacheManager, make_request, manager.allocate_slots, manager.take_events, manager.free`。 代码主体包含 11 个显式断言。

### test_null_parent_block_hash (lines 1972-2036)
```python
def test_null_parent_block_hash():
    block_size = 1
    num_cached_blocks = 2
    num_full_blocks = 4
    kv_cache_group_id = 0

    pool = BlockPool(
        num_gpu_blocks=8,
        enable_caching=True,
        hash_block_size=block_size,
        enable_kv_cache_events=True,
    )
    req = make_request(
        "req_null_parent",
        prompt_token_ids=[10, 11, 12, 13],
        block_size=block_size,
        hash_fn=sha256,
    # ... excerpt omitted for brevity ...
    assert len(req.block_hashes) == num_full_blocks
    assert pool.null_block.block_hash is None
    assert len(events) == 1
    assert isinstance(event, BlockStored)
    expected_parent = kv_cache_utils.maybe_convert_block_hash(
    assert event.parent_block_hash == expected_parent
    # Ensure we didn't accidentally assign a hash to the null block.
    # Sanity check: newly cached physical blocks should have hashes assigned.
    assert blocks[num_cached_blocks].block_hash is not None
    assert blocks[num_full_blocks - 1].block_hash is not None
```
**EN:** Test case covering `null parent block hash`. It exercises `BlockPool, make_request, pool.get_new_blocks, pool.cache_full_blocks, pool.take_events, isinstance`. The body contains 13 explicit assertion(s).
**CN:** 该代码块是覆盖 `null parent block hash` 的测试用例。 该测试会调用 `BlockPool, make_request, pool.get_new_blocks, pool.cache_full_blocks, pool.take_events, isinstance`。 代码主体包含 13 个显式断言。

### test_kv_cache_events_with_lora (lines 2040-2094)
```python
def test_kv_cache_events_with_lora(blocks_to_cache: int):
    """Test BlockStored events contain correct lora_id when using LoRA requests."""
    block_size = 16
    num_blocks = blocks_to_cache + 1

    # Create KVCacheManager with events enabled
    manager = KVCacheManager(
        make_kv_cache_config(block_size, num_blocks),
        max_model_len=8192,
        enable_caching=True,
        enable_kv_cache_events=True,
        hash_block_size=block_size,
    )
    # Test with LoRA request
    lora_request = LoRARequest(
        lora_name="test_lora", lora_int_id=42, lora_path="/test/path"
    # ... excerpt omitted for brevity ...
    assert isinstance(block_stored_event, BlockStored)
    assert block_stored_event.lora_id == 42  # Should match lora_request.adapter_id
    assert len(block_stored_event.block_hashes) == blocks_to_cache
    assert block_stored_event.block_size == block_size
    block_stored_event = events[-1]
    assert block_stored_event.lora_id is None  # Should be None when no LoRA request
```
**EN:** Parameterized test covering `KV cache events with lora`. Parameter axes: `blocks_to_cache`. Inputs/fixtures: `blocks_to_cache`. It exercises `mark.parametrize, KVCacheManager, LoRARequest, make_request, manager.allocate_slots, manager.take_events`. The body contains 8 explicit assertion(s).
**CN:** 该代码块是覆盖 `KV 缓存 events with lora` 的测试用例。 参数维度：`blocks_to_cache`。 输入或 fixture：`blocks_to_cache`。 该测试会调用 `mark.parametrize, KVCacheManager, LoRARequest, make_request, manager.allocate_slots, manager.take_events`。 代码主体包含 8 个显式断言。

### test_block_stored_event_group_idx (lines 2098-2149)
```python
def test_block_stored_event_group_idx(group_id: int):
    """Test BlockStored events emitted by cache_full_blocks carry the correct
    group_idx."""
    block_size = 4
    num_tokens = block_size * 2

    manager = KVCacheManager(
        make_kv_cache_config_three_types(block_size, num_blocks=5),
        max_model_len=8192,
        enable_caching=True,
        enable_kv_cache_events=True,
        hash_block_size=block_size,
    )
    pool = manager.block_pool
    req = make_request(
        "req_grp_idx",
        prompt_token_ids=list(range(num_tokens)),
    # ... excerpt omitted for brevity ...
    assert len(events) == 1
    assert isinstance(events[0], BlockStored)
    assert events[0].group_idx == group_id
    assert (
        == [
            None,
            2 * block_size,
        ][group_id]
```
**EN:** Parameterized test covering `block stored event group idx`. Parameter axes: `group_id`. Inputs/fixtures: `group_id`. It exercises `mark.parametrize, KVCacheManager, make_request, pool.get_new_blocks, pool.cache_full_blocks, manager.take_events`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `block stored event group idx` 的测试用例。 参数维度：`group_id`。 输入或 fixture：`group_id`。 该测试会调用 `mark.parametrize, KVCacheManager, make_request, pool.get_new_blocks, pool.cache_full_blocks, manager.take_events`。 代码主体包含 5 个显式断言。

### test_block_stored_event_group_idx_multiple_groups (lines 2152-2235)
```python
def test_block_stored_event_group_idx_multiple_groups():
    """
    Test BlockStored events for separate HMA groups that each carry the
    correct group_idx.

    Simulates the HMA scenario where full-attention blocks (group 0) and
    sliding-window blocks (group 1) are cached independently and must be
    distinguishable by consumers doing HMA-aware prefix-cache routing.
    block_size = 4
    num_tokens = block_size * 2
    manager = KVCacheManager(
        KVCacheConfig(
            num_blocks=5,
            kv_cache_tensors=[],
            kv_cache_groups=[
                KVCacheGroupSpec(
    # ... excerpt omitted for brevity ...
    assert len(events) == 2
    assert isinstance(events[0], BlockStored)
    assert events[0].group_idx == 0
    assert events[0].kv_cache_spec_kind == KVCacheSpecKind.FULL_ATTENTION.value
    assert events[0].kv_cache_spec_sliding_window is None
    assert isinstance(events[1], BlockStored)
    assert events[1].group_idx == 1
    assert events[1].kv_cache_spec_kind == KVCacheSpecKind.SLIDING_WINDOW.value
    assert events[1].kv_cache_spec_sliding_window == 128
```
**EN:** Test case covering `block stored event group idx multiple groups`. It exercises `KVCacheManager, make_request, pool.get_new_blocks, pool.cache_full_blocks, manager.take_events, isinstance`. The body contains 9 explicit assertion(s).
**CN:** 该代码块是覆盖 `block stored event group idx multiple groups` 的测试用例。 该测试会调用 `KVCacheManager, make_request, pool.get_new_blocks, pool.cache_full_blocks, manager.take_events, isinstance`。 代码主体包含 9 个显式断言。

### test_block_stored_event_group_idx_out_of_bounds (lines 2238-2271)
```python
def test_block_stored_event_group_idx_out_of_bounds(monkeypatch):
    """Out-of-range group_idx events are returned without metadata annotation."""
    block_size = 4
    manager = KVCacheManager(
        make_kv_cache_config(block_size, num_blocks=5),
        max_model_len=8192,
        enable_caching=True,
        enable_kv_cache_events=True,
        hash_block_size=block_size,
    )
    event = BlockStored(
        block_hashes=[1],
        parent_block_hash=None,
        token_ids=list(range(block_size)),
        block_size=block_size,
        lora_id=None,
        medium=None,
        lora_name=None,
    # ... excerpt omitted for brevity ...
    events = manager.take_events()

    assert events == [event]
    assert event.kv_cache_spec_kind is None
    assert event.kv_cache_spec_sliding_window is None
    assert warnings == ["Group index `1` not in KV cache metadata"]
```
**EN:** Test case covering `block stored event group idx out of bounds`. Inputs/fixtures: `monkeypatch`. It exercises `KVCacheManager, BlockStored, kv_event_queue.append, monkeypatch.setattr, manager.take_events, make_kv_cache_config`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `block stored event group idx out of bounds` 的测试用例。 输入或 fixture：`monkeypatch`。 该测试会调用 `KVCacheManager, BlockStored, kv_event_queue.append, monkeypatch.setattr, manager.take_events, make_kv_cache_config`。 代码主体包含 4 个显式断言。

### test_block_removed_event_group_idx (lines 2275-2324)
```python
def test_block_removed_event_group_idx(group_id: int):
    """
    Test BlockRemoved events emitted on eviction carry the group_idx extracted
    from the evicted block's BlockHashWithGroupId via get_group_id().
    block_size = 4
    num_tokens = block_size * 2

    # null block + 4 usable; allocate all 4, cache 2, free all, re-allocate
    # all 4 so the 2 cached blocks are forced through _maybe_evict_cached_block.
    pool = BlockPool(
        num_gpu_blocks=5,
        enable_caching=True,
        hash_block_size=block_size,
        enable_kv_cache_events=True,
    )
    req = make_request(
    # ... excerpt omitted for brevity ...
    events = pool.take_events()
    removed_events = [e for e in events if isinstance(e, BlockRemoved)]
    assert len(removed_events) == 2
    for event in removed_events:
        assert event.group_idx == group_id
```
**EN:** Parameterized test covering `block removed event group idx`. Parameter axes: `group_id`. Inputs/fixtures: `group_id`. It exercises `mark.parametrize, BlockPool, make_request, pool.get_new_blocks, pool.cache_full_blocks, pool.take_events`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `block removed event group idx` 的测试用例。 参数维度：`group_id`。 输入或 fixture：`group_id`。 该测试会调用 `mark.parametrize, BlockPool, make_request, pool.get_new_blocks, pool.cache_full_blocks, pool.take_events`。 代码主体包含 2 个显式断言。

### test_eagle_enabled_removes_last_block (lines 2327-2358)
```python
def test_eagle_enabled_removes_last_block():
    """Verify Eagle does NOT remove blocks when request
    length is divisible by block size."""
    block_size = 16
    manager = KVCacheManager(
        make_kv_cache_config(block_size, num_blocks=10),
        max_model_len=8192,
        enable_caching=True,
        use_eagle=True,
        hash_block_size=block_size,
    )

    # Request with 3 full blocks (48 tokens)
    token_ids = [0] * (3 * block_size)
    req = make_request("divisible_request", token_ids, block_size, sha256)

    # Prime the cache
    computed_blocks, _ = manager.get_computed_blocks(req)
    manager.allocate_slots(
        req, len(token_ids), len(computed_blocks.blocks[0]) * 16, computed_blocks
    )
    manager.free(req)

    # New request with same tokens + Eagle enabled
    req_eagle = make_request("eagle_divisible", token_ids, block_size, sha256)
    computed_blocks, num_tokens = manager.get_computed_blocks(req_eagle)

    # Should retain 1 block:
    # 1. Original 3 blocks → pop last hash → 2 matched blocks
    # 2. drop last matched block → 1 remaining block
    assert len(computed_blocks.blocks[0]) == 1
    assert num_tokens == 1 * block_size  # 16 tokens
```
**EN:** Test case covering `eagle enabled removes last block`. It exercises `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free, make_kv_cache_config`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `eagle enabled removes last block` 的测试用例。 该测试会调用 `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free, make_kv_cache_config`。 代码主体包含 2 个显式断言。

### test_eagle_with_partial_blocks (lines 2361-2387)
```python
def test_eagle_with_partial_blocks():
    """Test Eagle behavior with requests containing partial blocks."""
    block_size = 16
    manager = KVCacheManager(
        make_kv_cache_config(block_size, num_blocks=10),
        max_model_len=8192,
        enable_caching=True,
        use_eagle=True,
        hash_block_size=block_size,
    )
    # 2 full blocks + 5 tokens (non-divisible length)
    token_ids = [0] * (2 * block_size + 5)
    req = make_request("partial_block_test", token_ids, block_size, sha256)

    # Prime the cache
    computed_blocks, _ = manager.get_computed_blocks(req)
    manager.allocate_slots(
        req, len(token_ids), len(computed_blocks.blocks[0]) * 16, computed_blocks
    )
    manager.free(req)

    # New request with Eagle enabled
    req_eagle = make_request("partial_eagle", token_ids, block_size, sha256)
    computed_blocks, num_tokens = manager.get_computed_blocks(req_eagle)
    # Original match: 2 full blocks → Eagle removes 1 → 1 remaining
    assert len(computed_blocks.blocks[0]) == 1
    assert num_tokens == 1 * block_size
```
**EN:** Test case covering `eagle with partial blocks`. It exercises `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free, make_kv_cache_config`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `eagle with partial blocks` 的测试用例。 该测试会调用 `KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free, make_kv_cache_config`。 代码主体包含 2 个显式断言。

### test_eagle_with_sliding_window (lines 2390-2453)
```python
def test_eagle_with_sliding_window():
    """Test Eagle behavior with sliding window."""
    block_size = 16
    sliding_window_spec = SlidingWindowSpec(
        block_size=block_size,
        num_kv_heads=1,
        head_size=1,
        dtype=torch.float32,
        sliding_window=block_size,
    )
    manager = KVCacheManager(
        KVCacheConfig(
            num_blocks=10,
            kv_cache_tensors=[],
            kv_cache_groups=[KVCacheGroupSpec(["layer"], sliding_window_spec)],
        ),
        max_model_len=8192,
        enable_caching=True,
    # ... excerpt omitted for brevity ...
    assert block_hash_first_block is not None
    assert len(computed_blocks.blocks[0]) == 1
    assert num_tokens == 1 * block_size
    assert (
    computed_blocks, num_tokens = manager.get_computed_blocks(req_after_evict)
    # Cache miss. The only hit prefix is [NULL_BLOCK, BLOCK_2] if eagle is
    # not considered. But after dropping the last matched block due to eagle,
    # there will be no matched prefix.
    assert len(computed_blocks.blocks[0]) == 0
    assert num_tokens == 0
```
**EN:** Test case covering `eagle with sliding window`. It exercises `SlidingWindowSpec, KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `eagle with sliding window` 的测试用例。 该测试会调用 `SlidingWindowSpec, KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, manager.free`。 代码主体包含 6 个显式断言。

### test_different_block_size (lines 2456-2528)
```python
def test_different_block_size():
    block_size = 16
    # full attention and sliding window attention layers have the same page size:
    # (32 tokens/block * float16 token, vs. 16 tokens/block * float32 token)
    kv_cache_config = KVCacheConfig(
        num_blocks=100,
        kv_cache_tensors=[],
        kv_cache_groups=[
            KVCacheGroupSpec(
                ["layer1"],
                FullAttentionSpec(
                    block_size=block_size * 2,
                    num_kv_heads=1,
                    head_size=1,
                    dtype=torch.float16,
                ),
            ),
    # ... excerpt omitted for brevity ...
    assert not computed_blocks.blocks[0]
    assert not computed_blocks.blocks[1]
    assert num_computed_tokens == 0
    assert blocks.get_block_ids() == ([1, 2, 3, 4], [5, 6, 7, 8, 9, 10, 11])
    assert len(computed_blocks.blocks[0]) == 3
    assert len(computed_blocks.blocks[1]) == 6
        make_block_hash_with_group_id(req1.block_hashes[5], 1), 10
    )
    computed_blocks, num_computed_tokens = manager.get_computed_blocks(req1)
    assert len(computed_blocks.blocks[0]) == 2
    assert len(computed_blocks.blocks[1]) == 4
    assert num_computed_tokens == 4 * 16
```
**EN:** Test case covering `different block size`. It exercises `KVCacheConfig, KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, cached_block_hash_to_block.pop`. The body contains 13 explicit assertion(s).
**CN:** 该代码块是覆盖 `different block size` 的测试用例。 该测试会调用 `KVCacheConfig, KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, cached_block_hash_to_block.pop`。 代码主体包含 13 个显式断言。

### test_hybrid_cache_blocks_swa_tail_window_only (lines 2531-2600)
```python
def test_hybrid_cache_blocks_swa_tail_window_only():
    """Within each lcm-aligned segment, SWA's ``find_longest_cache_hit`` only
    returns the trailing ``ceil((sliding_window - 1) / block_size)`` blocks
    (its right-to-left scan stops once a contiguous match is found). Blocks
    earlier in the segment can never serve a hit, so
    ``HybridKVCacheCoordinator.cache_blocks`` should skip them rather than
    polluting the prefix-cache hash map."""
    block_size = 8
    # Full attn block_size=32, SWA block_size=8, sw=8 -> lcm=32.
    # tail = ceil(7/8) = 1; per_segment = 32/8 = 4.
    # Per-segment template = [F, F, F, T]; only the last SWA block in each
    # 32-token segment ends up in the prefix-cache hash map.
    kv_cache_config = KVCacheConfig(
        num_blocks=100,
        kv_cache_tensors=[],
        kv_cache_groups=[
            KVCacheGroupSpec(
                ["layer1"],
    # ... excerpt omitted for brevity ...
    assert blocks is not None
    assert len(req.block_hashes) == 8
    expected_cached = {3, 7}
        if i in expected_cached:
            assert cached is not None, f"SWA hash {i} should be cached"
        else:
            assert cached is None, (
                f"SWA hash {i} cannot serve any lcm-aligned hit; should not be cached"
            )
```
**EN:** Test case covering `hybrid cache blocks swa tail window only`. It exercises `KVCacheConfig, KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, range`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `hybrid 缓存 blocks swa tail window only` 的测试用例。 该测试会调用 `KVCacheConfig, KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, range`。 代码主体包含 4 个显式断言。

### test_hybrid_cache_blocks_clamped_to_lcm (lines 2603-2667)
```python
def test_hybrid_cache_blocks_clamped_to_lcm():
    """HybridKVCacheCoordinator.cache_blocks() clamps to lcm_block_size.
    Chunks past the last lcm-aligned boundary can never participate in a
    cache hit (find_longest_cache_hit always returns lcm-aligned hits), so
    caching them only pollutes the prefix-cache hash map and keeps blocks
    on the LRU list that could otherwise return to the free pool."""
    block_size = 16
    # Full attn block_size=32, SWA block_size=16 -> lcm=32.
    kv_cache_config = KVCacheConfig(
        num_blocks=100,
        kv_cache_tensors=[],
        kv_cache_groups=[
            KVCacheGroupSpec(
                ["layer1"],
                FullAttentionSpec(
                    block_size=block_size * 2,
                    num_kv_heads=1,
                    head_size=1,
    # ... excerpt omitted for brevity ...
    assert blocks is not None
    assert len(req.block_hashes) == 7
        assert (
            pool.get_cached_block(req.block_hashes[i], kv_cache_group_ids=[1])
            is not None
        ), f"SWA hash {i} should be cached"
    assert pool.get_cached_block(req.block_hashes[6], kv_cache_group_ids=[1]) is None, (
        "SWA hash 6 spans tokens past the lcm boundary; should not be cached"
    )
```
**EN:** Test case covering `hybrid cache blocks clamped to lcm`. It exercises `KVCacheConfig, KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, range`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `hybrid 缓存 blocks clamped to lcm` 的测试用例。 该测试会调用 `KVCacheConfig, KVCacheManager, make_request, manager.get_computed_blocks, manager.allocate_slots, range`。 代码主体包含 4 个显式断言。

### test_block_lookup_cache_single_block_per_key (lines 2670-2710)
```python
def test_block_lookup_cache_single_block_per_key():
    cache = BlockHashToBlockMap()
    key0 = BlockHashWithGroupId(b"hash0")
    key1 = BlockHashWithGroupId(b"hash1")
    key2 = BlockHashWithGroupId(b"hash2")
    block0 = KVCacheBlock(0)
    block1 = KVCacheBlock(1)

    assert cache.get_one_block(key0) is None
    assert cache.get_one_block(key1) is None
    assert cache.get_one_block(key2) is None
    # key0 inserted
    cache.insert(key0, block0)
    assert cache.get_one_block(key0) is block0
    # key1 inserted
    cache.insert(key1, block1)
    # ... excerpt omitted for brevity ...
    assert cache.get_one_block(key1) is block1
    assert cache.pop(key0, 100) is None
    # block popped with (key1, block ID 1)
    assert cache.pop(key1, 1) is block1
```
**EN:** Test case covering `block lookup cache single block per key`. It exercises `BlockHashToBlockMap, BlockHashWithGroupId, KVCacheBlock, cache.insert, cache.get_one_block, cache.pop`. The body contains 25 explicit assertion(s).
**CN:** 该代码块是覆盖 `block lookup 缓存 single block per key` 的测试用例。 该测试会调用 `BlockHashToBlockMap, BlockHashWithGroupId, KVCacheBlock, cache.insert, cache.get_one_block, cache.pop`。 代码主体包含 25 个显式断言。

### test_block_lookup_cache_multi_blocks_per_key (lines 2713-2742)
```python
def test_block_lookup_cache_multi_blocks_per_key():
    cache = BlockHashToBlockMap()
    key0 = BlockHashWithGroupId(b"hash0")
    key1 = BlockHashWithGroupId(b"hash1")
    block00 = KVCacheBlock(0)
    block01 = KVCacheBlock(1)
    block10 = KVCacheBlock(10)
    block11 = KVCacheBlock(11)

    assert cache.get_one_block(key0) is None
    assert cache.get_one_block(key1) is None

    cache.insert(key0, block00)
    cache.insert(key0, block01)
    cache.insert(key1, block10)
    cache.insert(key1, block11)

    assert cache.get_one_block(key0) is block00
    assert cache.pop(key0, 0) is block00
    assert cache.get_one_block(key0) is block01
    assert cache.pop(key0, 1) is block01
    assert cache.get_one_block(key0) is None
    assert cache.pop(key0, 2) is None

    assert cache.get_one_block(key1) is block10
    assert cache.pop(key1, 10) is block10
    assert cache.get_one_block(key1) is block11
    assert cache.pop(key1, 11) is block11
    assert cache.get_one_block(key1) is None
    assert cache.pop(key1, 12) is None
```
**EN:** Test case covering `block lookup cache multi blocks per key`. It exercises `BlockHashToBlockMap, BlockHashWithGroupId, KVCacheBlock, cache.insert, cache.get_one_block, cache.pop`. The body contains 14 explicit assertion(s).
**CN:** 该代码块是覆盖 `block lookup 缓存 multi blocks per key` 的测试用例。 该测试会调用 `BlockHashToBlockMap, BlockHashWithGroupId, KVCacheBlock, cache.insert, cache.get_one_block, cache.pop`。 代码主体包含 14 个显式断言。

### test_can_fit_full_sequence_swa_cap_admits_long_prompt (lines 2745-2801)
```python
def test_can_fit_full_sequence_swa_cap_admits_long_prompt():
    """Hybrid full+SWA model with a pool sized at the startup minimum should
    admit a prompt longer than the SWA cap, because SlidingWindowManager
    recycles blocks during chunked prefill (issue #39734)."""
    block_size = 16
    sliding_window = 4 * block_size  # 64 tokens
    max_num_batched_tokens = 8 * block_size  # 128 tokens
    max_model_len = 64 * block_size  # 1024 tokens — much larger than the SWA cap
    # Startup pool sizing: full demands cdiv(max_model_len, bs) = 64 blocks,
    # SWA demands cdiv(SW-1+max_batched, bs) + 1 = cdiv(191, 16) + 1 = 13.
    # Pool minimum = 64 + 13 = 77; +1 for the null block.
    num_blocks = 64 + 13 + 1

    config = KVCacheConfig(
        num_blocks=num_blocks,
        kv_cache_tensors=[],
        kv_cache_groups=[
            KVCacheGroupSpec(
    # ... excerpt omitted for brevity ...
    prompt_len = 32 * block_size
    req = make_request("long", list(range(prompt_len)), block_size, sha256)
    assert (
        manager.allocate_slots(req, block_size, full_sequence_must_fit=True) is not None
    )
```
**EN:** Test case covering `can fit full sequence swa cap admits long prompt`. It exercises `KVCacheConfig, KVCacheManager, make_request, list, manager.allocate_slots, range`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `can fit full sequence swa cap admits long prompt` 的测试用例。 该测试会调用 `KVCacheConfig, KVCacheManager, make_request, list, manager.allocate_slots, range`。 代码主体包含 1 个显式断言。

### test_can_fit_full_sequence_full_attention_still_gates_oversized (lines 2804-2852)
```python
def test_can_fit_full_sequence_full_attention_still_gates_oversized():
    """The cap only loosens the SWA group; a prompt that exceeds the
    full-attention pool capacity must still be rejected."""
    block_size = 16
    sliding_window = 4 * block_size
    max_num_batched_tokens = 8 * block_size
    max_model_len = 64 * block_size
    # Provide a tiny pool — even a small prompt should be rejected.
    num_blocks = 5

    config = KVCacheConfig(
        num_blocks=num_blocks,
        kv_cache_tensors=[],
        kv_cache_groups=[
            KVCacheGroupSpec(
                ["layer_full"],
                FullAttentionSpec(
                    block_size=block_size,
    # ... excerpt omitted for brevity ...
    # 16 blocks of full attention demand alone exceeds the 5-block pool.
    prompt_len = 16 * block_size
    req = make_request("oversized", list(range(prompt_len)), block_size, sha256)
    assert manager.allocate_slots(req, block_size, full_sequence_must_fit=True) is None
```
**EN:** Test case covering `can fit full sequence full attention still gates oversized`. It exercises `KVCacheConfig, KVCacheManager, make_request, list, manager.allocate_slots, range`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `can fit full sequence full 注意力 still gates oversized` 的测试用例。 该测试会调用 `KVCacheConfig, KVCacheManager, make_request, list, manager.allocate_slots, range`。 代码主体包含 1 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** Multimodal input bookkeeping
- **CN:** 多模态输入管理

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm.v1.core.kv_cache_manager, vllm.v1.core.kv_cache_utils, vllm.distributed.kv_events, vllm.lora.request, vllm.multimodal.inputs, vllm.sampling_params, vllm.utils.hashing, vllm.v1.core.block_pool, ...`.
- **CN:** 被测试的 vLLM 模块：`vllm.v1.core.kv_cache_manager, vllm.v1.core.kv_cache_utils, vllm.distributed.kv_events, vllm.lora.request, vllm.multimodal.inputs, vllm.sampling_params, vllm.utils.hashing, vllm.v1.core.block_pool, ...`。
- **EN:** Standard-library support: `copy, collections.abc`.
- **CN:** 标准库支持：`copy, collections.abc`。
