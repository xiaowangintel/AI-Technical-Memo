# test_stats.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/metrics/test_stats.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `stats` behavior and regressions in the v1 stack. / 验证 v1 栈中 `stats` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-9)
```python
from vllm.v1.engine import FinishReason
from vllm.v1.metrics.stats import (
    IterationStats,
    PrefillStats,
    PromptTokenStats,
    RequestStateStats,
)
```
**EN:** Imports the libraries needed to build the test harness. vLLM modules under test include `vllm.v1.engine, vllm.v1.metrics.stats`.
**CN:** 该代码块导入构建测试环境所需的库。 被测试的 vLLM 模块包括 `vllm.v1.engine, vllm.v1.metrics.stats`。

### test_iteration_stats_repr (lines 12-14)
```python
def test_iteration_stats_repr():
    iteration_stats = IterationStats()
    assert repr(iteration_stats).startswith("IterationStats(")
```
**EN:** Test case covering `iteration stats repr`. It exercises `IterationStats, repr.startswith, repr`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `iteration stats repr` 的测试用例。 该测试会调用 `IterationStats, repr.startswith, repr`。 代码主体包含 1 个显式断言。

### test_prefill_kv_computed_with_cache (lines 17-45)
```python
def test_prefill_kv_computed_with_cache():
    """Test that prefill KV compute correctly excludes cached tokens."""
    iteration_stats = IterationStats()
    req_stats = RequestStateStats(arrival_time=0.0)
    req_stats.scheduled_ts = 0.1
    req_stats.first_token_ts = 0.5
    req_stats.last_token_ts = 5.0
    req_stats.num_generation_tokens = 50

    # Case 1: With prefix cache (1200 tokens cached)
    iteration_stats.update_from_finished_request(
        finish_reason=FinishReason.STOP,
        request_id="test-req-001",
        num_prompt_tokens=10000,
        max_tokens_param=100,
        req_stats=req_stats,
        num_cached_tokens=1200,
    )

    finished_req = iteration_stats.finished_requests[0]
    assert finished_req.num_prompt_tokens == 10000
    assert finished_req.num_cached_tokens == 1200
    assert finished_req.request_id == "test-req-001"

    # Verify calculation: prefill KV = prompt tokens - cached tokens
    prefill_kv_computed = finished_req.num_prompt_tokens - max(
        finished_req.num_cached_tokens, 0
    )
    assert prefill_kv_computed == 8800  # 10000 - 1200
```
**EN:** Test case covering `prefill KV computed with cache`. It exercises `IterationStats, RequestStateStats, iteration_stats.update_from_finished_request, max`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `prefill kv computed with 缓存` 的测试用例。 该测试会调用 `IterationStats, RequestStateStats, iteration_stats.update_from_finished_request, max`。 代码主体包含 4 个显式断言。

### test_prefill_kv_computed_no_cache (lines 48-76)
```python
def test_prefill_kv_computed_no_cache():
    """Test prefill KV compute without prefix caching."""
    iteration_stats = IterationStats()
    req_stats = RequestStateStats(arrival_time=0.0)
    req_stats.scheduled_ts = 0.1
    req_stats.first_token_ts = 0.5
    req_stats.last_token_ts = 2.0
    req_stats.num_generation_tokens = 10

    # Case 2: No prefix cache
    iteration_stats.update_from_finished_request(
        finish_reason=FinishReason.STOP,
        request_id="test-req-002",
        num_prompt_tokens=2000,
        max_tokens_param=100,
        req_stats=req_stats,
        num_cached_tokens=0,
    )

    finished_req = iteration_stats.finished_requests[0]
    assert finished_req.num_prompt_tokens == 2000
    assert finished_req.num_cached_tokens == 0
    assert finished_req.request_id == "test-req-002"

    # Verify calculation: prefill KV = full prompt when no cache
    prefill_kv_computed = finished_req.num_prompt_tokens - max(
        finished_req.num_cached_tokens, 0
    )
    assert prefill_kv_computed == 2000
```
**EN:** Test case covering `prefill KV computed no cache`. It exercises `IterationStats, RequestStateStats, iteration_stats.update_from_finished_request, max`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `prefill kv computed no 缓存` 的测试用例。 该测试会调用 `IterationStats, RequestStateStats, iteration_stats.update_from_finished_request, max`。 代码主体包含 4 个显式断言。

### test_prefill_kv_computed_edge_cases (lines 79-122)
```python
def test_prefill_kv_computed_edge_cases():
    """Test edge cases for prefill KV compute calculation."""
    iteration_stats = IterationStats()
    req_stats = RequestStateStats(arrival_time=0.0)
    req_stats.scheduled_ts = 0.1
    req_stats.first_token_ts = 0.5
    req_stats.last_token_ts = 1.0
    req_stats.num_generation_tokens = 1

    # Case 3: Negative num_cached_tokens (shouldn't happen, but handle gracefully)
    iteration_stats.update_from_finished_request(
        finish_reason=FinishReason.STOP,
        request_id="test-req-003",
        num_prompt_tokens=100,
        max_tokens_param=10,
        req_stats=req_stats,
        num_cached_tokens=-1,
    )
    # ... excerpt omitted for brevity ...
    assert prefill_kv_computed == 100  # Should treat negative as 0
    assert finished_req.request_id == "test-req-003"
    finished_req2 = iteration_stats2.finished_requests[0]
    prefill_kv_computed2 = finished_req2.num_prompt_tokens - max(
        finished_req2.num_cached_tokens, 0
    assert prefill_kv_computed2 == 0  # All cached, nothing computed
    assert finished_req2.request_id == "test-req-004"
```
**EN:** Test case covering `prefill KV computed edge cases`. It exercises `IterationStats, RequestStateStats, iteration_stats.update_from_finished_request, iteration_stats2.update_from_finished_request, max`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `prefill kv computed edge cases` 的测试用例。 该测试会调用 `IterationStats, RequestStateStats, iteration_stats.update_from_finished_request, iteration_stats2.update_from_finished_request, max`。 代码主体包含 4 个显式断言。

### test_prompt_token_stats_all_computed (lines 125-142)
```python
def test_prompt_token_stats_all_computed():
    """Test all tokens computed locally, no caching."""
    stats = PromptTokenStats()

    # Case 1: No caching (All tokens computed locally)
    prefill_stats = PrefillStats()
    prefill_stats.set(
        num_prompt_tokens=1000,
        num_local_cached_tokens=0,
        num_external_cached_tokens=0,
    )
    stats.update_from_output(prefill_stats)

    assert stats.computed == 1000
    assert stats.local_cache_hit == 0
    assert stats.external_kv_transfer == 0
    assert stats.cached_tokens == 0
    assert stats.total == 1000
```
**EN:** Test case covering `prompt token stats all computed`. It exercises `PromptTokenStats, PrefillStats, prefill_stats.set, stats.update_from_output`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `prompt token stats all computed` 的测试用例。 该测试会调用 `PromptTokenStats, PrefillStats, prefill_stats.set, stats.update_from_output`。 代码主体包含 5 个显式断言。

### test_prompt_token_stats_partial_local_cache (lines 145-162)
```python
def test_prompt_token_stats_partial_local_cache():
    """Test partial local prefix cache hit."""
    stats = PromptTokenStats()

    # Case 2: Partial local cache
    prefill_stats = PrefillStats()
    prefill_stats.set(
        num_prompt_tokens=1000,
        num_local_cached_tokens=300,
        num_external_cached_tokens=0,
    )
    stats.update_from_output(prefill_stats)

    assert stats.computed == 700
    assert stats.local_cache_hit == 300
    assert stats.external_kv_transfer == 0
    assert stats.cached_tokens == 300
    assert stats.total == 1000
```
**EN:** Test case covering `prompt token stats partial local cache`. It exercises `PromptTokenStats, PrefillStats, prefill_stats.set, stats.update_from_output`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `prompt token stats partial local 缓存` 的测试用例。 该测试会调用 `PromptTokenStats, PrefillStats, prefill_stats.set, stats.update_from_output`。 代码主体包含 5 个显式断言。

### test_prompt_token_stats_partial_external_transfer (lines 165-182)
```python
def test_prompt_token_stats_partial_external_transfer():
    """Test partial external KV transfer."""
    stats = PromptTokenStats()

    # Case 3: Partial external transfer
    prefill_stats = PrefillStats()
    prefill_stats.set(
        num_prompt_tokens=1000,
        num_local_cached_tokens=0,
        num_external_cached_tokens=500,
    )
    stats.update_from_output(prefill_stats)

    assert stats.computed == 500
    assert stats.local_cache_hit == 0
    assert stats.external_kv_transfer == 500
    assert stats.cached_tokens == 500
    assert stats.total == 1000
```
**EN:** Test case covering `prompt token stats partial external transfer`. It exercises `PromptTokenStats, PrefillStats, prefill_stats.set, stats.update_from_output`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `prompt token stats partial external transfer` 的测试用例。 该测试会调用 `PromptTokenStats, PrefillStats, prefill_stats.set, stats.update_from_output`。 代码主体包含 5 个显式断言。

### test_prompt_token_stats_mixed_sources (lines 185-202)
```python
def test_prompt_token_stats_mixed_sources():
    """Test mix of local cache and external transfer."""
    stats = PromptTokenStats()

    # Case 4: Mixed sources
    prefill_stats = PrefillStats()
    prefill_stats.set(
        num_prompt_tokens=1000,
        num_local_cached_tokens=400,
        num_external_cached_tokens=200,
    )
    stats.update_from_output(prefill_stats)

    assert stats.computed == 400
    assert stats.local_cache_hit == 400
    assert stats.external_kv_transfer == 200
    assert stats.cached_tokens == 600
    assert stats.total == 1000
```
**EN:** Test case covering `prompt token stats mixed sources`. It exercises `PromptTokenStats, PrefillStats, prefill_stats.set, stats.update_from_output`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `prompt token stats mixed sources` 的测试用例。 该测试会调用 `PromptTokenStats, PrefillStats, prefill_stats.set, stats.update_from_output`。 代码主体包含 5 个显式断言。

### test_prompt_token_stats_full_local_cache_recompute (lines 205-226)
```python
def test_prompt_token_stats_full_local_cache_recompute():
    """Test full local cache triggers last token recomputation.

    When all tokens are cached, the scheduler forces the model to recompute
    the last token (num_computed_tokens=1), with the rest from cache.
    """
    stats = PromptTokenStats()

    # Case 5: Full local cache (999 cached, 1 recomputed)
    prefill_stats = PrefillStats()
    prefill_stats.set(
        num_prompt_tokens=1000,
        num_local_cached_tokens=999,
        num_external_cached_tokens=0,
    )
    stats.update_from_output(prefill_stats)

    assert stats.computed == 1
    assert stats.local_cache_hit == 999
    assert stats.external_kv_transfer == 0
    assert stats.cached_tokens == 999
    assert stats.total == 1000
```
**EN:** Test case covering `prompt token stats full local cache recompute`. It exercises `PromptTokenStats, PrefillStats, prefill_stats.set, stats.update_from_output`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `prompt token stats full local 缓存 recompute` 的测试用例。 该测试会调用 `PromptTokenStats, PrefillStats, prefill_stats.set, stats.update_from_output`。 代码主体包含 5 个显式断言。

### test_prompt_token_stats_full_external_transfer_recompute (lines 229-246)
```python
def test_prompt_token_stats_full_external_transfer_recompute():
    """Test full external transfer triggers last token recomputation."""
    stats = PromptTokenStats()

    # Case 6: Full external transfer (999 from external, 1 recomputed)
    prefill_stats = PrefillStats()
    prefill_stats.set(
        num_prompt_tokens=1000,
        num_local_cached_tokens=0,
        num_external_cached_tokens=999,
    )
    stats.update_from_output(prefill_stats)

    assert stats.computed == 1
    assert stats.local_cache_hit == 0
    assert stats.external_kv_transfer == 999
    assert stats.cached_tokens == 999
    assert stats.total == 1000
```
**EN:** Test case covering `prompt token stats full external transfer recompute`. It exercises `PromptTokenStats, PrefillStats, prefill_stats.set, stats.update_from_output`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `prompt token stats full external transfer recompute` 的测试用例。 该测试会调用 `PromptTokenStats, PrefillStats, prefill_stats.set, stats.update_from_output`。 代码主体包含 5 个显式断言。

## Key Concepts / 关键概念
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** Engine-core request/response orchestration
- **CN:** 引擎核心的请求/响应编排
- **EN:** Token-level boundary and decoding checks
- **CN:** token 级边界与解码检查

## Dependencies / 依赖关系
- **EN:** vLLM modules under test: `vllm.v1.engine, vllm.v1.metrics.stats`.
- **CN:** 被测试的 vLLM 模块：`vllm.v1.engine, vllm.v1.metrics.stats`。
