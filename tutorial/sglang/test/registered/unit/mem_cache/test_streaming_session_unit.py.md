# test_streaming_session_unit.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/mem_cache/test_streaming_session_unit.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates streaming session unit behavior in SGLang's unit / mem cache area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 内存缓存 领域中与 streaming session unit 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: module imports and dependencies / 模块导入与依赖
```python
from types import SimpleNamespace

import torch

from sglang.srt.managers.schedule_batch import FINISH_ABORT
from sglang.srt.mem_cache.base_prefix_cache import MatchResult
from sglang.srt.session.streaming_session import SessionSlot, StreamingSession
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `types`, `torch`, `sglang.srt.managers.schedule_batch`, `sglang.srt.mem_cache.base_prefix_cache`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `types`, `torch`, `sglang.srt.managers.schedule_batch`, `sglang.srt.mem_cache.base_prefix_cache`。

### Lines 10-10: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=12, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 13-13: class _FakeAllocator declaration / 类 _FakeAllocator 声明
```python
class _FakeAllocator:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 14-15: method init / 方法 init
```python
    def __init__(self):
        self.freed = []
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 17-18: method free / 方法 free
```python
    def free(self, free_index: torch.Tensor):
        self.freed.append(free_index.clone())
```
**EN:** This block implements `free` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `free`，承担模块行为中的一个聚焦逻辑片段。

### Lines 21-21: class _FakeInnerCache declaration / 类 _FakeInnerCache 声明
```python
class _FakeInnerCache:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 22-27: method init / 方法 init
```python
    def __init__(self, req_to_token_pool, allocator, page_size, match_results=None):
        self.req_to_token_pool = req_to_token_pool
        self.token_to_kv_pool_allocator = allocator
        self.page_size = page_size
        self.match_results = list(match_results or [])
        self.dec_lock_ref_calls = []
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 29-30: method cache finished req / 方法 cache finished req
```python
    def cache_finished_req(self, *args, **kwargs):
        raise AssertionError("Streaming requests should not delegate to inner cache")
```
**EN:** This block implements `cache_finished_req` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `cache_finished_req`，承担模块行为中的一个聚焦逻辑片段。

### Lines 32-35: method match prefix / 方法 match prefix
```python
    def match_prefix(self, *args, **kwargs):
        if not self.match_results:
            raise AssertionError("Unexpected match_prefix call")
        return self.match_results.pop(0)
```
**EN:** This block implements `match_prefix` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `match_prefix`，承担模块行为中的一个聚焦逻辑片段。

### Lines 37-38: method dec lock ref / 方法 dec lock ref
```python
    def dec_lock_ref(self, node, *args, **kwargs):
        self.dec_lock_ref_calls.append(node)
```
**EN:** This block implements `dec_lock_ref` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `dec_lock_ref`，承担模块行为中的一个聚焦逻辑片段。

### Lines 40-41: method supports mamba / 方法 supports mamba
```python
    def supports_mamba(self):
        return False
```
**EN:** This block implements `supports_mamba` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `supports_mamba`，承担模块行为中的一个聚焦逻辑片段。

### Lines 43-44: method sanity check / 方法 sanity check
```python
    def sanity_check(self):
        return None
```
**EN:** This block implements `sanity_check` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `sanity_check`，承担模块行为中的一个聚焦逻辑片段。

### Lines 47-47: class _FakeReq declaration / 类 _FakeReq 声明
```python
class _FakeReq:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 48-78: method init / 方法 init
```python
    def __init__(
        self, session_id: str, req_pool_idx: int, committed: int, allocated: int
    ):
        self.session = SimpleNamespace(
            session_id=session_id,
            streaming=True,
            finish_req=lambda req: None,
            abort_req=lambda: None,
            _inflight=False,
        )
        self.req_pool_idx = req_pool_idx
        self.kv_committed_len = committed
        self.kv_allocated_len = allocated
        self.kv_committed_freed = False
        self.kv_overallocated_freed = False
        self.origin_input_ids = list(range(committed))
        self.output_ids = []
        self.extra_key = None
        self.swa_evicted_seqlen = 0
        self.last_node = None
        self.cache_protected_len = 0
        self.swa_uuid_for_lock = None
        self.mamba_pool_idx = None
        self.mamba_ping_pong_track_buffer = None
        self.mamba_next_track_idx = None
        self.mamba_last_track_seqlen = None
        self.mamba_branching_seqlen = None
        self.pop_overallocated_calls = 0
        self.to_finish = None
        self.finished_reason = None
        self.finished_len = None
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 80-83: method pop committed kv cache / 方法 pop committed kv cache
```python
    def pop_committed_kv_cache(self):
        assert not self.kv_committed_freed
        self.kv_committed_freed = True
        return self.kv_committed_len
```
**EN:** This block implements `pop_committed_kv_cache` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `pop_committed_kv_cache`，承担模块行为中的一个聚焦逻辑片段。

### Lines 85-89: method pop overallocated kv cache / 方法 pop overallocated kv cache
```python
    def pop_overallocated_kv_cache(self):
        assert not self.kv_overallocated_freed
        self.pop_overallocated_calls += 1
        self.kv_overallocated_freed = True
        return self.kv_committed_len, self.kv_allocated_len
```
**EN:** This block implements `pop_overallocated_kv_cache` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `pop_overallocated_kv_cache`，承担模块行为中的一个聚焦逻辑片段。

### Lines 92-136: test case preabort detaches session and preserves slot / 测试用例 preabort detaches session and preserves slot
```python
def test_preabort_detaches_session_and_preserves_slot():
    """Pre-aborted req (to_finish set before match_prefix) is detached from
    the session: session=None, abort_req() called. Slot stays intact."""
    req_to_token = torch.arange(256, dtype=torch.int32).reshape(2, 128)
    req_to_token_pool = SimpleNamespace(req_to_token=req_to_token, free_slots=[])
    allocator = _FakeAllocator()
    inner = _FakeInnerCache(
        req_to_token_pool,
        allocator,
        page_size=16,
        match_results=[
            MatchResult(
                device_indices=torch.tensor([], dtype=torch.int64),
                last_device_node=None,
                last_host_node=None,
                best_match_node=None,
            )
        ],
    )
    tree_cache = StreamingSession(inner)
    tree_cache.slots["session-a"] = SessionSlot(
        req_pool_idx=0,
        kv_committed_len=48,
        kv_allocated_len=48,
        cache_protected_len=16,
    )

    req = _FakeReq("session-a", req_pool_idx=1, committed=1, allocated=1)
    req.to_finish = FINISH_ABORT("too long")

    result = tree_cache.match_prefix(
        SimpleNamespace(
            req=req,
            key=SimpleNamespace(token_ids=list(range(64))),
        )
    )

    # Req detached from session.
    assert req.session is None
    # Slot untouched.
    slot = tree_cache.slots["session-a"]
    assert slot.req_pool_idx == 0
    assert slot.kv_committed_len == 48
    assert slot.kv_allocated_len == 48
    assert len(result.device_indices) == 0
```
**EN:** Pre-aborted req (to_finish set before match_prefix) is detached from the session: session=None, abort_req() called. This test exercises `test_preabort_detaches_session_and_preserves_slot` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Pre-aborted req (to_finish set before match_prefix) is detached from the session: session=None, abort_req() called. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_preabort_detaches_session_and_preserves_slot`。

### Lines 139-164: test case first mid abort nukes ephemeral slot / 测试用例 first mid abort nukes ephemeral slot
```python
def test_first_mid_abort_nukes_ephemeral_slot():
    """First-request mid-processing abort: no slot exists yet, ephemeral
    slot is created from req state and nuked via release_session."""
    page_size = 1
    req_to_token = torch.arange(128, dtype=torch.int32).reshape(1, 128)
    req_to_token_pool = SimpleNamespace(req_to_token=req_to_token, free_slots=[])
    allocator = _FakeAllocator()
    inner = _FakeInnerCache(req_to_token_pool, allocator, page_size)
    tree_cache = StreamingSession(inner)

    # No slot exists yet (first request).
    req = _FakeReq("session-a", req_pool_idx=0, committed=0, allocated=20)
    req.finished_reason = FINISH_ABORT("input too long")

    tree_cache.cache_finished_req(req)

    # Slot must NOT be created.
    assert "session-a" not in tree_cache.slots
    # Transient pool slot freed.
    assert req.req_pool_idx is None
    assert req_to_token_pool.free_slots == [0]
    assert len(allocator.freed) == 1
    assert allocator.freed[0].tolist() == list(range(20))
    # Bookkeeping flags set.
    assert req.kv_committed_freed is True
    assert req.kv_overallocated_freed is True
```
**EN:** First-request mid-processing abort: no slot exists yet, ephemeral slot is created from req state and nuked via release_session. This test exercises `test_first_mid_abort_nukes_ephemeral_slot` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** First-request mid-processing abort: no slot exists yet, ephemeral slot is created from req state and nuked via release_session. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_first_mid_abort_nukes_ephemeral_slot`。

### Lines 167-203: test case nth mid abort nukes session slot / 测试用例 nth mid abort nukes session slot
```python
def test_nth_mid_abort_nukes_session_slot():
    """Nth-request mid-processing abort: slot exists, restore_to_req ran.
    ALL KV is wiped (release_session). Slot is deleted. Token IDs stay
    in req_nodes for next turn's re-prefill."""
    page_size = 1
    req_to_token = torch.arange(256, dtype=torch.int32).reshape(2, 128)
    req_to_token_pool = SimpleNamespace(req_to_token=req_to_token, free_slots=[])
    allocator = _FakeAllocator()
    inner = _FakeInnerCache(req_to_token_pool, allocator, page_size)
    tree_cache = StreamingSession(inner)

    # Session already has a slot from a previous turn.
    tree_cache.slots["session-a"] = SessionSlot(
        req_pool_idx=0,
        kv_committed_len=50,
        kv_allocated_len=50,
        last_node=None,
        cache_protected_len=0,
    )

    # Mid-processing abort: req has the SESSION slot's pool_idx (restore_to_req ran).
    req = _FakeReq("session-a", req_pool_idx=0, committed=60, allocated=65)
    req.finished_reason = FINISH_ABORT("client disconnected")

    tree_cache.cache_finished_req(req)

    # Slot wiped — deleted from slots dict.
    assert "session-a" not in tree_cache.slots
    # All KV freed: [0, 65) from release_session (slot extended to req's allocated).
    assert len(allocator.freed) == 1
    assert allocator.freed[0].tolist() == list(range(65))
    # Pool slot returned.
    assert req_to_token_pool.free_slots == [0]
    assert req.req_pool_idx is None
    # Bookkeeping flags set.
    assert req.kv_committed_freed is True
    assert req.kv_overallocated_freed is True
```
**EN:** Nth-request mid-processing abort: slot exists, restore_to_req ran. This test exercises `test_nth_mid_abort_nukes_session_slot` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Nth-request mid-processing abort: slot exists, restore_to_req ran. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nth_mid_abort_nukes_session_slot`。

### Lines 204-210: supporting source context / 辅助源码上下文
```python


# Shrink tests removed: streaming sessions are append-only after the
# rollback fix in session_controller (rollback_aborted_req).  The shrink
# code path in cache_finished_req no longer exists.


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 211-244: test case trim overshoot postcondition / 测试用例 trim overshoot postcondition
```python
def test_trim_overshoot_postcondition():
    """`_trim_overshoot` postcondition: every per-req KV field is capped at
    target = origin+finished_len, output_ids is truncated, and the tail
    KV slots are freed. Covers both non-SWA fields (kv_committed_len,
    kv_allocated_len, output_ids) and SWA bookkeeping (swa_evicted_seqlen)
    in one shot — same invariant `_free_tail` enforces on the match_prefix
    path.
    """
    page_size = 1
    req_to_token = torch.arange(128, dtype=torch.int32).reshape(1, 128)
    req_to_token_pool = SimpleNamespace(req_to_token=req_to_token, free_slots=[])
    allocator = _FakeAllocator()
    tree_cache = StreamingSession(
        _FakeInnerCache(req_to_token_pool, allocator, page_size)
    )

    # Overshoot scenario: origin=26, finished_len=12 -> target=38.
    # committed=40 (overshoot 2), allocated=44, swa_evicted=42 (> target),
    # output_ids extended to 14 by the overshoot round.
    req = _FakeReq("session-a", req_pool_idx=0, committed=40, allocated=44)
    req.origin_input_ids = list(range(26))
    req.output_ids = list(range(14))
    req.swa_evicted_seqlen = 42

    tree_cache._trim_overshoot(req, finished_len=12)

    target = 38
    assert req.kv_committed_len == target
    assert req.kv_allocated_len == target
    assert req.swa_evicted_seqlen == target
    assert len(req.output_ids) == 12
    # Tail [38, 44) freed by _free_kv_aligned.
    assert len(allocator.freed) == 1
    assert allocator.freed[0].tolist() == list(range(38, 44))
```
**EN:** `_trim_overshoot` postcondition: every per-req KV field is capped at target = origin+finished_len, output_ids is truncated, and the tail KV slots are freed. This test exercises `test_trim_overshoot_postcondition` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** `_trim_overshoot` postcondition: every per-req KV field is capped at target = origin+finished_len, output_ids is truncated, and the tail KV slots are freed. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trim_overshoot_postcondition`。

### Lines 247-252: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    import sys

    import pytest

    sys.exit(pytest.main([__file__, "-v"]))
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_FakeAllocator`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_FakeInnerCache`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_FakeReq`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `test_preabort_detaches_session_and_preserves_slot`: Pre-aborted req (to_finish set before match_prefix) is detached from the session: session=None, abort_req() called. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_preabort_detaches_session_and_preserves_slot`。
- `test_first_mid_abort_nukes_ephemeral_slot`: First-request mid-processing abort: no slot exists yet, ephemeral slot is created from req state and nuked via release_session. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_first_mid_abort_nukes_ephemeral_slot`。
- `test_nth_mid_abort_nukes_session_slot`: Nth-request mid-processing abort: slot exists, restore_to_req ran. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nth_mid_abort_nukes_session_slot`。
- `test_trim_overshoot_postcondition`: `_trim_overshoot` postcondition: every per-req KV field is capped at target = origin+finished_len, output_ids is truncated, and the tail KV slots are freed. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trim_overshoot_postcondition`。
- `_FakeAllocator.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。
- `_FakeAllocator.free`: This block implements `free` and captures one focused piece of the module's behavior. / 该代码块实现 `free`，承担模块行为中的一个聚焦逻辑片段。
- `_FakeInnerCache.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。
- `_FakeInnerCache.cache_finished_req`: This block implements `cache_finished_req` and captures one focused piece of the module's behavior. / 该代码块实现 `cache_finished_req`，承担模块行为中的一个聚焦逻辑片段。
- `_FakeInnerCache.match_prefix`: This block implements `match_prefix` and captures one focused piece of the module's behavior. / 该代码块实现 `match_prefix`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `types`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.session.streaming_session`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 252
