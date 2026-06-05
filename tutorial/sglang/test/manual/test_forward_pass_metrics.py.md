# test_forward_pass_metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_forward_pass_metrics.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `forward pass metrics` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual` 中的 `forward pass metrics` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Scenario logic / 场景逻辑
```python
"""
Manual test for Forward Pass Metrics (FPM) ZMQ PUB/SUB path.

Tests:
1. Schema encode/decode roundtrip
2. _FpmPublisherThread ZMQ PUB -> ZMQ SUB end-to-end
3. Heartbeat emission on idle
"""

import sys
import time

import zmq
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `Metrics`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 16-40: Test routines around test_schema_roundtrip / 测试例程
```python
def test_schema_roundtrip():
    from sglang.srt.observability.forward_pass_metrics import (
        ForwardPassMetrics,
        QueuedRequestMetrics,
        ScheduledRequestMetrics,
        WelfordAccumulator,
        decode,
        encode,
    )

    # WelfordAccumulator
    acc = WelfordAccumulator()
    for v in [10, 20, 30]:
        acc.add(v)
    assert acc.count == 3
    assert acc.total == 60
    var = acc.variance()
    assert abs(var - 66.667) < 0.01, f"Expected ~66.667, got {var}"

    # Encode/decode roundtrip
    fpm = ForwardPassMetrics(
        worker_id="test-worker",
        dp_rank=1,
        wall_time=0.042,
        scheduled_requests=ScheduledRequestMetrics(
```
**EN:** This range defines concrete test routine(s) `test_schema_roundtrip`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `WelfordAccumulator`, `add`, `variance` and `abs`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 41-65: Assertions and result checks / 断言与结果检查
```python
            num_prefill_requests=5,
            sum_prefill_tokens=1024,
            var_prefill_length=33.3,
            sum_prefill_kv_tokens=512,
            num_decode_requests=32,
            sum_decode_kv_tokens=8192,
            var_decode_kv_tokens=100.0,
        ),
        queued_requests=QueuedRequestMetrics(
            num_prefill_requests=3,
            sum_prefill_tokens=768,
            var_prefill_length=25.0,
            num_decode_requests=1,
            sum_decode_kv_tokens=128,
            var_decode_kv_tokens=0.0,
        ),
    )

    data = encode(fpm)
    fpm2 = decode(data)

    assert fpm2.worker_id == "test-worker"
    assert fpm2.dp_rank == 1
    assert fpm2.wall_time == 0.042
    assert fpm2.scheduled_requests.num_prefill_requests == 5
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `QueuedRequestMetrics`, `encode` and `decode`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 66-70: Assertions and result checks / 断言与结果检查
```python
    assert fpm2.scheduled_requests.sum_prefill_tokens == 1024
    assert fpm2.scheduled_requests.num_decode_requests == 32
    assert fpm2.queued_requests.num_prefill_requests == 3

    print("PASS: schema roundtrip")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。

### Lines 73-97: Test routines around test_zmq_pub_sub / 测试例程
```python
def test_zmq_pub_sub():
    """Test _FpmPublisherThread -> ZMQ SUB end-to-end."""
    from sglang.srt.observability.forward_pass_metrics import (
        ForwardPassMetrics,
        ScheduledRequestMetrics,
        _FpmPublisherThread,
        decode,
    )

    port = 29999
    endpoint = f"tcp://127.0.0.1:{port}"

    # Start publisher
    pub = _FpmPublisherThread(
        f"tcp://*:{port}",
        worker_id="test-pub",
        dp_rank=0,
    )

    # Connect subscriber
    ctx = zmq.Context()
    sub = ctx.socket(zmq.SUB)
    sub.connect(endpoint)
    sub.setsockopt(zmq.SUBSCRIBE, b"")
    sub.setsockopt(zmq.RCVTIMEO, 5000)  # 5s timeout
```
**EN:** This range defines concrete test routine(s) `test_zmq_pub_sub`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `_FpmPublisherThread`, `Context`, `socket` and `connect`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 98-122: Assertions and result checks / 断言与结果检查
```python

    # ZMQ PUB/SUB needs time to connect
    time.sleep(0.5)

    # Publish a metric
    fpm = ForwardPassMetrics(
        worker_id="test-pub",
        dp_rank=0,
        wall_time=0.05,
        scheduled_requests=ScheduledRequestMetrics(
            num_prefill_requests=10,
            sum_prefill_tokens=2048,
            num_decode_requests=64,
            sum_decode_kv_tokens=16384,
        ),
    )
    pub.publish(fpm)

    # Receive
    frames = sub.recv_multipart()
    assert len(frames) == 3, f"Expected 3 frames, got {len(frames)}"

    topic, seq_bytes, payload = frames
    assert topic == b""
    seq = int.from_bytes(seq_bytes, "big")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `sleep`, `ForwardPassMetrics`, `ScheduledRequestMetrics` and `publish`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 123-141: Assertions and result checks / 断言与结果检查
```python
    assert seq == 0

    received = decode(payload)
    assert received.worker_id == "test-pub"
    assert received.scheduled_requests.num_prefill_requests == 10
    assert received.scheduled_requests.sum_decode_kv_tokens == 16384
    print(f"PASS: ZMQ PUB/SUB (seq={seq}, {len(payload)} bytes)")

    # Publish a second message -- seq should increment
    pub.publish(fpm)
    frames2 = sub.recv_multipart()
    seq2 = int.from_bytes(frames2[1], "big")
    assert seq2 == 1, f"Expected seq=1, got {seq2}"
    print(f"PASS: sequence incremented (seq={seq2})")

    # Cleanup
    pub.shutdown()
    sub.close()
    ctx.term()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `decode`, `SUB`, `publish` and `recv_multipart`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 144-168: Test routines around test_heartbeat / 测试例程
```python
def test_heartbeat():
    """Test that heartbeat messages are emitted when idle."""
    from sglang.srt.observability.forward_pass_metrics import (
        _FpmPublisherThread,
        decode,
    )

    port = 29998
    endpoint = f"tcp://127.0.0.1:{port}"

    pub = _FpmPublisherThread(
        f"tcp://*:{port}",
        worker_id="heartbeat-test",
        dp_rank=0,
    )
    # Override heartbeat interval for faster test
    pub.HEARTBEAT_INTERVAL = 0.3

    ctx = zmq.Context()
    sub = ctx.socket(zmq.SUB)
    sub.connect(endpoint)
    sub.setsockopt(zmq.SUBSCRIBE, b"")
    sub.setsockopt(zmq.RCVTIMEO, 3000)

    time.sleep(0.5)
```
**EN:** This range defines concrete test routine(s) `test_heartbeat`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `_FpmPublisherThread`, `Context`, `socket` and `connect`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 169-184: Assertions and result checks / 断言与结果检查
```python

    # Don't publish anything -- wait for heartbeat
    try:
        frames = sub.recv_multipart()
        heartbeat = decode(frames[2])
        assert heartbeat.worker_id == "heartbeat-test"
        assert heartbeat.wall_time == 0.0  # idle heartbeat
        assert heartbeat.scheduled_requests.num_prefill_requests == 0
        print("PASS: heartbeat received")
    except zmq.Again:
        print("FAIL: no heartbeat received within timeout")
        sys.exit(1)

    pub.shutdown()
    sub.close()
    ctx.term()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `recv_multipart`, `decode`, `exit` and `shutdown`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 185-191: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    test_schema_roundtrip()
    test_zmq_pub_sub()
    test_heartbeat()
    print("\nAll tests passed!")
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `test_schema_roundtrip`, `test_zmq_pub_sub` and `test_heartbeat`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- HTTP/API interaction / HTTP/API 交互
- Token-level inspection / Token 级分析
- Disaggregated serving / 解耦式服务
- Structured outputs / 结构化输出

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `sys`, `time`
- **Third-party / 第三方库**: `zmq`
- **Project Modules / 项目模块**: `sglang.srt.observability.forward_pass_metrics`
