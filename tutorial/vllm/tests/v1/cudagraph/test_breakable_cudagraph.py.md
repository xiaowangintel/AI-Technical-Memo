# test_breakable_cudagraph.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/cudagraph/test_breakable_cudagraph.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Unit tests for the breakable cudagraph primitives. / 该文件的文档字符串表明其用途：`unit tests for the breakable cudagraph primitives`。

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
Unit tests for the breakable cudagraph primitives.
"""
```
**EN:** Module docstring that declares the scope of the file: Unit tests for the breakable cudagraph primitives.
**CN:** 模块文档字符串直接说明了文件范围：`unit tests for the breakable cudagraph primitives`。

### Imports and setup / 导入与设置 (lines 7-13)
```python
from __future__ import annotations

import os
import threading

import pytest
import torch
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm.compilation.breakable_cudagraph`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm.compilation.breakable_cudagraph`。

### Module state / 模块级状态 (line 15)
```python
os.environ["VLLM_USE_BREAKABLE_CUDAGRAPH"] = "1"
```
**EN:** Defines module-level constants, feature gates, or shared state.
**CN:** 定义模块级常量、特性开关或共享状态。

### _reset_breakable_tls (lines 19-26)
```python
def _reset_breakable_tls():
    """Defensively clear thread-local capture state between tests so a
    failure in one test can't leak "nested capture" errors into the next."""
    from vllm.compilation.breakable_cudagraph import BreakableCUDAGraphCapture

    BreakableCUDAGraphCapture._tls.active = None
    yield
    BreakableCUDAGraphCapture._tls.active = None
```
**EN:** Fixture/helper `_reset_breakable_tls` prepares reusable state for downstream tests. Key calls include `pytest.fixture`.
**CN:** `_reset_breakable_tls` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `pytest.fixture`。

### cuda_capture_stream (lines 30-42)
```python
def cuda_capture_stream():
    """A non-default CUDA stream suitable for cudagraph capture.

    ``CUDAGraph.capture_begin`` refuses to capture from the default
    stream, so all capture-using tests need to run under
    ``torch.cuda.stream(...)`` for a separate stream.
    """
    if not torch.cuda.is_available():
        pytest.skip("CUDA required")
    stream = torch.cuda.Stream()
    with torch.cuda.stream(stream):
        yield stream
    torch.cuda.current_stream().wait_stream(stream)
```
**EN:** Fixture/helper `cuda_capture_stream` prepares reusable state for downstream tests. Key calls include `cuda.Stream, current_stream.wait_stream, cuda.is_available, pytest.skip, cuda.stream, cuda.current_stream`.
**CN:** `cuda_capture_stream` 是为后续测试准备可复用状态的 fixture/辅助函数。 关键调用包括 `cuda.Stream, current_stream.wait_stream, cuda.is_available, pytest.skip, cuda.stream, cuda.current_stream`。

### test_decorator_passthrough_outside_capture (lines 50-61)
```python
def test_decorator_passthrough_outside_capture():
    from vllm.compilation.breakable_cudagraph import eager_break_during_capture

    calls = []

    @eager_break_during_capture
    def f(x):
        calls.append(x)
        return x * 2

    assert f(3) == 6
    assert calls == [3]
```
**EN:** Test case covering `decorator passthrough outside capture`. It exercises `calls.append, f`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `decorator passthrough outside capture` 的测试用例。 该测试会调用 `calls.append, f`。 代码主体包含 2 个显式断言。

### test_current_is_none_when_inactive (lines 69-73)
```python
def test_current_is_none_when_inactive():
    from vllm.compilation.breakable_cudagraph import BreakableCUDAGraphCapture

    assert BreakableCUDAGraphCapture.current() is None
    assert BreakableCUDAGraphCapture.is_active() is False
```
**EN:** Test case covering `current is none when inactive`. It exercises `BreakableCUDAGraphCapture.current, BreakableCUDAGraphCapture.is_active`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `current is none when inactive` 的测试用例。 该测试会调用 `BreakableCUDAGraphCapture.current, BreakableCUDAGraphCapture.is_active`。 代码主体包含 2 个显式断言。

### test_thread_local_active_during_context (lines 76-83)
```python
def test_thread_local_active_during_context(cuda_capture_stream):
    from vllm.compilation.breakable_cudagraph import BreakableCUDAGraphCapture

    cap = BreakableCUDAGraphCapture()
    with cap:
        assert BreakableCUDAGraphCapture.current() is cap
        assert BreakableCUDAGraphCapture.is_active() is True
    assert BreakableCUDAGraphCapture.current() is None
```
**EN:** Test case covering `thread local active during context`. Inputs/fixtures: `cuda_capture_stream`. It exercises `BreakableCUDAGraphCapture, BreakableCUDAGraphCapture.current, BreakableCUDAGraphCapture.is_active`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `thread local active during context` 的测试用例。 输入或 fixture：`cuda_capture_stream`。 该测试会调用 `BreakableCUDAGraphCapture, BreakableCUDAGraphCapture.current, BreakableCUDAGraphCapture.is_active`。 代码主体包含 3 个显式断言。

### test_nested_capture_raises (lines 86-92)
```python
def test_nested_capture_raises(cuda_capture_stream):
    from vllm.compilation.breakable_cudagraph import BreakableCUDAGraphCapture

    outer = BreakableCUDAGraphCapture()
    inner = BreakableCUDAGraphCapture()
    with outer, pytest.raises(RuntimeError, match="Nested.*not supported"), inner:
        pass
```
**EN:** Test case covering `nested capture raises`. Inputs/fixtures: `cuda_capture_stream`. It exercises `BreakableCUDAGraphCapture, pytest.raises`. Validation is mainly performed through expected exception checks.
**CN:** 该代码块是覆盖 `nested capture raises` 的测试用例。 输入或 fixture：`cuda_capture_stream`。 该测试会调用 `BreakableCUDAGraphCapture, pytest.raises`。 主要通过预期异常检查来完成验证。

### test_active_state_isolated_across_threads (lines 95-121)
```python
def test_active_state_isolated_across_threads(cuda_capture_stream):
    """Verify the thread-local 'active capture' slot is per-thread.

    We don't run concurrent captures here -- CUDA only supports one
    in-flight capture per stream and we keep tests cheap. We just check
    that the worker thread sees its own slot as None while the main
    thread has a capture active.
    """
    from vllm.compilation.breakable_cudagraph import BreakableCUDAGraphCapture

    worker_view: dict[str, BreakableCUDAGraphCapture | None] = {}

    def worker():
        worker_view["state"] = BreakableCUDAGraphCapture.current()

    main_cap = BreakableCUDAGraphCapture()
    with main_cap:
        # Main thread has a live capture.
        assert BreakableCUDAGraphCapture.current() is main_cap
        t = threading.Thread(target=worker)
        t.start()
        t.join()

    # Worker thread saw None -- thread-local separation works.
    assert worker_view["state"] is None
    # Main thread's slot is cleared on exit.
    assert BreakableCUDAGraphCapture.current() is None
```
**EN:** Test case covering `active state isolated across threads`. Inputs/fixtures: `cuda_capture_stream`. It exercises `BreakableCUDAGraphCapture, BreakableCUDAGraphCapture.current, threading.Thread, t.start, t.join`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `active state isolated across threads` 的测试用例。 输入或 fixture：`cuda_capture_stream`。 该测试会调用 `BreakableCUDAGraphCapture, BreakableCUDAGraphCapture.current, threading.Thread, t.start, t.join`。 代码主体包含 3 个显式断言。

### test_capture_with_no_eager_break_records_one_graph (lines 129-138)
```python
def test_capture_with_no_eager_break_records_one_graph(cuda_capture_stream):
    from vllm.compilation.breakable_cudagraph import BreakableCUDAGraphCapture

    x = torch.zeros(4, device="cuda")
    cap = BreakableCUDAGraphCapture()
    with cap:
        x.add_(1.0)
    assert len(cap.segments) == 1
    assert cap.num_graphs == 1
    assert cap.num_eager_breaks == 0
```
**EN:** Test case covering `capture with no eager break records one graph`. Inputs/fixtures: `cuda_capture_stream`. It exercises `torch.zeros, BreakableCUDAGraphCapture, x.add_, len`. The body contains 3 explicit assertion(s).
**CN:** 该代码块是覆盖 `capture with no eager break records one graph` 的测试用例。 输入或 fixture：`cuda_capture_stream`。 该测试会调用 `torch.zeros, BreakableCUDAGraphCapture, x.add_, len`。 代码主体包含 3 个显式断言。

### test_add_eager_creates_alternating_graph_eager_graph (lines 141-166)
```python
def test_add_eager_creates_alternating_graph_eager_graph(cuda_capture_stream):
    from vllm.compilation.breakable_cudagraph import BreakableCUDAGraphCapture

    x = torch.zeros(4, device="cuda")
    counter = {"eager_calls": 0}

    def eager_step():
        counter["eager_calls"] += 1
        x.add_(10.0)

    cap = BreakableCUDAGraphCapture()
    with cap:
        x.add_(1.0)
        cap.add_eager(eager_step)
        x.add_(1.0)
        cap.add_eager(eager_step)
        x.add_(1.0)
    # 3 graph segments + 2 eager segments, interleaved as G E G E G.
    assert len(cap.segments) == 5
    assert cap.num_graphs == 3
    assert cap.num_eager_breaks == 2
    # Eager fn is stored as-is in the segment list, so we can confirm
    # the alternation pattern by identity check.
    assert cap.segments[1] is eager_step
    assert cap.segments[3] is eager_step
    assert counter["eager_calls"] == 2  # only the in-capture invocation
```
**EN:** Test case covering `add eager creates alternating graph eager graph`. Inputs/fixtures: `cuda_capture_stream`. It exercises `torch.zeros, BreakableCUDAGraphCapture, x.add_, cap.add_eager, len`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `add eager creates alternating graph eager graph` 的测试用例。 输入或 fixture：`cuda_capture_stream`。 该测试会调用 `torch.zeros, BreakableCUDAGraphCapture, x.add_, cap.add_eager, len`。 代码主体包含 6 个显式断言。

### test_capture_replay_matches_eager_simple (lines 174-216)
```python
def test_capture_replay_matches_eager_simple(cuda_capture_stream):
    """Verify that replay reproduces the same end-state as a single eager
    forward, with an eager break in the middle.

    Note: during capture, the *captured* kernels are recorded but NOT
    executed (that's CUDA-graph semantics). Only the eager segments
    actually mutate state at capture time. So we check correctness after
    ``replay()``, not after ``with cap:`` exits.
    """
    from vllm.compilation.breakable_cudagraph import BreakableCUDAGraphCapture
    x = torch.zeros(8, device="cuda")
    log: list[str] = []
    def eager_break_op():
        x.mul_(2.0)
        log.append("eager")
    # ... excerpt omitted for brevity ...
    assert torch.equal(x, torch.zeros(8, device="cuda"))
    assert log == ["eager"]
    assert torch.equal(x, torch.full((8,), 27.0, device="cuda"))
    assert log == ["eager", "eager"]
    # Replay again with another input: 100 -> 101 -> 202 -> 207.
    x.fill_(100.0)
    cap.replay()
    torch.accelerator.synchronize()
    assert torch.equal(x, torch.full((8,), 207.0, device="cuda"))
    assert log == ["eager", "eager", "eager"]
```
**EN:** Test case covering `capture replay matches eager simple`. Inputs/fixtures: `cuda_capture_stream`. It exercises `torch.zeros, BreakableCUDAGraphCapture, accelerator.synchronize, torch.equal, x.fill_, cap.replay`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `capture replay matches eager simple` 的测试用例。 输入或 fixture：`cuda_capture_stream`。 该测试会调用 `torch.zeros, BreakableCUDAGraphCapture, accelerator.synchronize, torch.equal, x.fill_, cap.replay`。 代码主体包含 6 个显式断言。

### test_decorator_breaks_when_invoked_inside_capture (lines 219-261)
```python
def test_decorator_breaks_when_invoked_inside_capture(cuda_capture_stream):
    """Verify @eager_break_during_capture correctly routes through
    add_eager when inside a capture context, and runs straight through
    when there's no active capture."""
    from vllm.compilation.breakable_cudagraph import (
        BreakableCUDAGraphCapture,
        eager_break_during_capture,
    )

    @eager_break_during_capture
    def attention_like(t: torch.Tensor) -> None:
        # In-place double; stands in for "real" attention work.
        t.mul_(2.0)
    x = torch.zeros(4, device="cuda")
    # Outside capture: decorator should just call through.
    x.fill_(3.0)
    # ... excerpt omitted for brevity ...
    assert torch.equal(x, torch.full((4,), 6.0, device="cuda"))
    assert torch.equal(x, torch.zeros(4, device="cuda"))
    assert len(cap.segments) == 3
    assert cap.num_graphs == 2
    assert cap.num_eager_breaks == 1
    # Replay: 2 -> 7 -> 14 -> 15.
    x.fill_(2.0)
    cap.replay()
    torch.accelerator.synchronize()
    assert torch.equal(x, torch.full((4,), 15.0, device="cuda"))
```
**EN:** Test case covering `decorator breaks when invoked inside capture`. Inputs/fixtures: `cuda_capture_stream`. It exercises `torch.zeros, x.fill_, attention_like, accelerator.synchronize, torch.equal, BreakableCUDAGraphCapture`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `decorator breaks when invoked inside capture` 的测试用例。 输入或 fixture：`cuda_capture_stream`。 该测试会调用 `torch.zeros, x.fill_, attention_like, accelerator.synchronize, torch.equal, BreakableCUDAGraphCapture`。 代码主体包含 6 个显式断言。

### test_replay_invokes_eager_segments_in_order (lines 269-298)
```python
def test_replay_invokes_eager_segments_in_order(cuda_capture_stream):
    from vllm.compilation.breakable_cudagraph import BreakableCUDAGraphCapture

    log: list[str] = []
    x = torch.zeros(1, device="cuda")

    def make_eager(name):
        def step():
            log.append(name)
            x.add_(1.0)

        return step

    cap = BreakableCUDAGraphCapture()
    with cap:
        x.add_(1.0)
        cap.add_eager(make_eager("A"))
        x.add_(1.0)
        cap.add_eager(make_eager("B"))
        x.add_(1.0)
        cap.add_eager(make_eager("C"))
        x.add_(1.0)

    # Capture-time invocation order
    assert log == ["A", "B", "C"]

    log.clear()
    cap.replay()
    torch.accelerator.synchronize()
    assert log == ["A", "B", "C"]
```
**EN:** Test case covering `replay invokes eager segments in order`. Inputs/fixtures: `cuda_capture_stream`. It exercises `torch.zeros, BreakableCUDAGraphCapture, log.clear, cap.replay, accelerator.synchronize, x.add_`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `replay invokes eager segments in order` 的测试用例。 输入或 fixture：`cuda_capture_stream`。 该测试会调用 `torch.zeros, BreakableCUDAGraphCapture, log.clear, cap.replay, accelerator.synchronize, x.add_`。 代码主体包含 2 个显式断言。

### test_exception_in_body_clears_active (lines 306-314)
```python
def test_exception_in_body_clears_active(cuda_capture_stream):
    from vllm.compilation.breakable_cudagraph import BreakableCUDAGraphCapture

    cap = BreakableCUDAGraphCapture()
    with pytest.raises(RuntimeError, match="boom"), cap:
        raise RuntimeError("boom")

    # active must be reset even after an exception inside the body
    assert BreakableCUDAGraphCapture.current() is None
```
**EN:** Test case covering `exception in body clears active`. Inputs/fixtures: `cuda_capture_stream`. It exercises `BreakableCUDAGraphCapture, pytest.raises, RuntimeError, BreakableCUDAGraphCapture.current`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `exception in body clears active` 的测试用例。 输入或 fixture：`cuda_capture_stream`。 该测试会调用 `BreakableCUDAGraphCapture, pytest.raises, RuntimeError, BreakableCUDAGraphCapture.current`。 代码主体包含 1 个显式断言。

### test_nested_decorated_op_runs_inline (lines 322-367)
```python
def test_nested_decorated_op_runs_inline(cuda_capture_stream):
    """A decorated op invoked from inside another decorated op's eager
    body must execute inline -- starting a second eager break mid-flight
    corrupts the segment state and explodes ``_begin_segment``'s assert.

    This mirrors the deepseek_v4_attention case where the outer attention
    op's impl internally dispatches sparse_attn_indexer (also decorated).
    """
    from vllm.compilation.breakable_cudagraph import (
        BreakableCUDAGraphCapture,
        eager_break_during_capture,
    )
    x = torch.zeros(4, device="cuda")
    inner_calls = 0
    @eager_break_during_capture
    def inner_op(t: torch.Tensor) -> None:
    # ... excerpt omitted for brevity ...
    assert cap.num_graphs == 2
    assert cap.num_eager_breaks == 1
    assert inner_calls == 1  # only the capture-time invocation
    x.fill_(0.0)
    cap.replay()
    torch.accelerator.synchronize()
    # 0 -> +2 -> +1 (inner) -> +10 (outer) -> +100 = 113
    assert torch.equal(x, torch.full((4,), 113.0, device="cuda"))
    assert inner_calls == 2  # replay invokes the outer's lambda again
```
**EN:** Test case covering `nested decorated op runs inline`. Inputs/fixtures: `cuda_capture_stream`. It exercises `torch.zeros, BreakableCUDAGraphCapture, x.fill_, cap.replay, accelerator.synchronize, torch.equal`. The body contains 5 explicit assertion(s).
**CN:** 该代码块是覆盖 `nested decorated op runs inline` 的测试用例。 输入或 fixture：`cuda_capture_stream`。 该测试会调用 `torch.zeros, BreakableCUDAGraphCapture, x.fill_, cap.replay, accelerator.synchronize, torch.equal`。 代码主体包含 5 个显式断言。

## Key Concepts / 关键概念
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm.compilation.breakable_cudagraph`.
- **CN:** 被测试的 vLLM 模块：`vllm.compilation.breakable_cudagraph`。
- **EN:** Standard-library support: `__future__, os, threading`.
- **CN:** 标准库支持：`__future__, os, threading`。
