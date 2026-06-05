# test_kv_events.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_kv_events.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `kv events` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual` 中的 `kv events` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Constants and scenario settings / 常量与场景配置
```python
import time
import unittest

import requests
import zmq
from msgspec.msgpack import Decoder

from sglang.srt.disaggregation.kv_events import (
    AllBlocksCleared,
    BlockRemoved,
    BlockStored,
    KVEventBatch,
)
from sglang.srt.utils import kill_process_tree
from sglang.test.test_utils import (
    DEFAULT_MLA_MODEL_NAME_FOR_TEST,
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)

QWEN3_30B_MODEL_PATH = "Qwen/Qwen3-30B-A3B-FP8"
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Process management appears here to launch, monitor, or stop runtime components.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。

### Lines 27-27: Class definition for TestKvEvents / 类定义
```python
class TestKvEvents(CustomTestCase):
```
**EN:** This range declares `TestKvEvents`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 28-51: Test routines around test_kv_events_enabled / 测试例程
```python
    def test_kv_events_enabled(self):
        """Test that kv events are sent and received by subscriber data when enabled"""

        # Launch kv events subscriber
        decoder = Decoder(type=KVEventBatch)
        context = zmq.Context()
        sub = context.socket(zmq.SUB)
        sub.connect("tcp://localhost:5557")
        topic = "kv-events"
        sub.setsockopt_string(zmq.SUBSCRIBE, topic)

        # Launch sglang server
        process = popen_launch_server(
            DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
            DEFAULT_URL_FOR_TEST,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--kv-events-config",
                '{"publisher": "zmq", "topic": "kv-events"}',
                "--max-total-tokens",
                32,
                "--cuda-graph-max-bs",
                2,
                "--enable-dp-attention",
```
**EN:** This range defines concrete test routine(s) `test_kv_events_enabled`. The logic drives the target scenario and encodes the expected acceptance criteria. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `Decoder`, `Context`, `socket` and `connect`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 52-52: Scenario logic / 场景逻辑
```python
                "--dp-size",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 53-76: Assertions and result checks / 断言与结果检查
```python
                1,
            ],
        )

        try:
            # Make some requests to generate some metrics
            response = requests.get(f"{DEFAULT_URL_FOR_TEST}/health_generate")
            self.assertEqual(response.status_code, 200)

            response = requests.post(
                f"{DEFAULT_URL_FOR_TEST}/generate",
                json={
                    "text": "The capital of France is",
                    "sampling_params": {
                        "temperature": 0,
                        "max_new_tokens": 32,
                    },
                },
            )
            response = requests.post(
                f"{DEFAULT_URL_FOR_TEST}/generate",
                json={
                    "text": "The capital of Spain is",
                    "sampling_params": {
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get`, `assertEqual` and `post`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 77-77: Scenario logic / 场景逻辑
```python
                        "temperature": 0,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 78-101: Assertions and result checks / 断言与结果检查
```python
                        "max_new_tokens": 32,
                    },
                },
            )

            # Get events
            events = []
            start = time.time()
            max_wait_s = 5
            min_events_expected = 5  # Expect at least some events

            while (
                len(events) < min_events_expected and (time.time() - start) < max_wait_s
            ):
                if sub.poll(timeout=100):  # 100ms timeout
                    _, seq_bytes, payload = sub.recv_multipart()
                    event_batch = decoder.decode(payload)
                    for event in event_batch.events:
                        events.append(event)

            # Verify we received events
            self.assertGreater(
                len(events), 0, "Should have received at least one KV cache event"
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `time`, `and`, `poll` and `recv_multipart`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 103-126: Assertions and result checks / 断言与结果检查
```python
            # Track which blocks were stored and removed
            stored_blocks = {}  # hash -> BlockStored event
            removed_hashes = set()

            # Validate event structure and relationships
            for event in events:
                self.assertIsInstance(
                    event,
                    (BlockStored, BlockRemoved, AllBlocksCleared),
                    f"Event should be a KV cache event, got {type(event)}",
                )

                if isinstance(event, BlockStored):
                    # Validate BlockStored structure
                    self.assertIsInstance(event.block_hashes, list)
                    self.assertEqual(
                        len(event.block_hashes), 1, "Should have one hash per block"
                    )
                    self.assertIsInstance(event.token_ids, list)
                    self.assertEqual(
                        event.block_size,
                        len(event.token_ids),
                        "block_size should match token_ids length",
                    )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertIsInstance`, `type` and `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 127-127: Assertions and result checks / 断言与结果检查
```python
                    self.assertIsNone(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertIsNone`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 128-151: Assertions and result checks / 断言与结果检查
```python
                        event.lora_id, "lora_id should be None for basic test"
                    )

                    # Store this block for later validation
                    block_hash = event.block_hashes[0]
                    stored_blocks[block_hash] = event

                    # If parent_block_hash is set, verify it was stored earlier
                    if event.parent_block_hash is not None:
                        # Parent should either be in stored_blocks or could be from a previous request
                        pass  # Don't strictly enforce this as root blocks may have synthetic parents

                elif isinstance(event, BlockRemoved):
                    # Validate BlockRemoved structure
                    self.assertIsInstance(event.block_hashes, list)
                    self.assertEqual(
                        len(event.block_hashes), 1, "Should have one hash per block"
                    )
                    removed_hashes.add(event.block_hashes[0])

            # Verify we got both BlockStored and BlockRemoved events
            self.assertGreater(
                len(stored_blocks), 0, "Should have at least one BlockStored event"
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertIsInstance`, `assertEqual`, `add` and `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 152-152: Scenario logic / 场景逻辑
```python
            # BlockRemoved events may not always occur in this short test, so just check if they do occur
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 153-162: Process control logic / 进程控制逻辑
```python
            # that they reference previously stored blocks
            for removed_hash in removed_hashes:
                # It's OK if the removed block wasn't in our stored_blocks
                # (it could have been stored before we started listening)
                pass

        finally:
            sub.close()
            context.term()
            kill_process_tree(process.pid)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `close`, `term` and `kill_process_tree`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 164-176: Test routines around test_kv_events_attn_dp / 测试例程
```python
    def test_kv_events_attn_dp(self):
        """Test that kv events are properly tagged with DP rank in attention DP mode"""

        # Launch multiple subscribers for different DP ranks
        decoder = Decoder(type=KVEventBatch)
        context = zmq.Context()

        # Subscribe to both DP rank endpoints
        sub_dp0 = context.socket(zmq.SUB)
        sub_dp0.connect("tcp://localhost:5557")  # DP rank 0
        topic = "kv-events"
        sub_dp0.setsockopt_string(zmq.SUBSCRIBE, topic)
```
**EN:** This range defines concrete test routine(s) `test_kv_events_attn_dp`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `Decoder`, `Context`, `socket` and `connect`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 177-188: Process control logic / 进程控制逻辑
```python
        sub_dp1 = context.socket(zmq.SUB)
        sub_dp1.connect("tcp://localhost:5558")  # DP rank 1 (offset by rank)
        sub_dp1.setsockopt_string(zmq.SUBSCRIBE, topic)

        # Launch sglang server with DP attention enabled
        process = popen_launch_server(
            DEFAULT_MLA_MODEL_NAME_FOR_TEST,
            DEFAULT_URL_FOR_TEST,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--kv-events-config",
                '{"publisher": "zmq", "topic": "kv-events"}',
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `socket`, `connect`, `setsockopt_string` and `popen_launch_server`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 189-201: Scenario logic / 场景逻辑
```python
                "--max-total-tokens",
                64,
                "--cuda-graph-max-bs",
                4,
                "--enable-dp-attention",
                "--dp-size",
                2,
                "--tp-size",
                2,
            ],
        )

        try:
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 202-213: Assertions and result checks / 断言与结果检查
```python
            # Make requests to generate events
            response = requests.get(f"{DEFAULT_URL_FOR_TEST}/health_generate")
            self.assertEqual(response.status_code, 200)

            # Send multiple requests to trigger events from both DP ranks
            for i in range(4):
                response = requests.post(
                    f"{DEFAULT_URL_FOR_TEST}/generate",
                    json={
                        "text": f"Request {i}: The capital of country {i} is",
                        "sampling_params": {
                            "temperature": 0,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get`, `assertEqual` and `post`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 214-226: Scenario logic / 场景逻辑
```python
                            "max_new_tokens": 16,
                        },
                    },
                )

            # Collect events from both DP ranks
            events_dp0 = []
            events_dp1 = []
            start = time.time()
            max_wait_s = 10
            min_events_per_rank = 3  # Expect at least a few events from each rank

            while (time.time() - start) < max_wait_s and (
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `time` and `and`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 227-238: Assertions and result checks / 断言与结果检查
```python
                len(events_dp0) < min_events_per_rank
                or len(events_dp1) < min_events_per_rank
            ):
                # Check DP rank 0
                if sub_dp0.poll(timeout=100):  # 100ms timeout
                    _, seq_bytes, payload = sub_dp0.recv_multipart()
                    event_batch = decoder.decode(payload)
                    print(
                        f"DP Rank 0 - EventBatch: ts={event_batch.ts}, attn_dp_rank={event_batch.attn_dp_rank}"
                    )
                    self.assertEqual(
                        event_batch.attn_dp_rank,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `poll`, `recv_multipart`, `decode` and `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 239-251: Scenario logic / 场景逻辑
```python
                        0,
                        "DP rank 0 events should have attn_dp_rank=0",
                    )
                    for event in event_batch.events:
                        print(f"  DP0 - {event}")
                        events_dp0.append(event)

                # Check DP rank 1
                if sub_dp1.poll(timeout=100):  # 100ms timeout
                    _, seq_bytes, payload = sub_dp1.recv_multipart()
                    event_batch = decoder.decode(payload)
                    print(
                        f"DP Rank 1 - EventBatch: ts={event_batch.ts}, attn_dp_rank={event_batch.attn_dp_rank}"
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `append`, `poll`, `recv_multipart` and `decode`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 252-263: Assertions and result checks / 断言与结果检查
```python
                    )
                    self.assertEqual(
                        event_batch.attn_dp_rank,
                        1,
                        "DP rank 1 events should have attn_dp_rank=1",
                    )
                    for event in event_batch.events:
                        print(f"  DP1 - {event}")
                        events_dp1.append(event)

            # Verify we got events from both DP ranks
            print(f"Collected {len(events_dp0)} events from DP rank 0")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual` and `append`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 264-276: Assertions and result checks / 断言与结果检查
```python
            print(f"Collected {len(events_dp1)} events from DP rank 1")

            self.assertGreaterEqual(
                len(events_dp0),
                min_events_per_rank,
                f"Expected at least {min_events_per_rank} events from DP rank 0",
            )
            self.assertGreaterEqual(
                len(events_dp1),
                min_events_per_rank,
                f"Expected at least {min_events_per_rank} events from DP rank 1",
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertGreaterEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 277-288: Assertions and result checks / 断言与结果检查
```python
            # Verify event types are as expected
            for events in [events_dp0, events_dp1]:
                for event in events:
                    self.assertIsInstance(
                        event,
                        (BlockStored, BlockRemoved, AllBlocksCleared),
                        f"Event should be a KV cache event, got {type(event)}",
                    )

        finally:
            sub_dp0.close()
            sub_dp1.close()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertIsInstance`, `type` and `close`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 289-290: Process control logic / 进程控制逻辑
```python
            context.term()
            kill_process_tree(process.pid)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `term` and `kill_process_tree`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 292-301: Test routines around test_kv_events_attn_cp_single_stream_per_dp_rank / 测试例程
```python
    def test_kv_events_attn_cp_single_stream_per_dp_rank(self):
        """Test that CP replicas do not publish duplicate KV events for one DP rank."""

        decoder = Decoder(type=KVEventBatch)
        context = zmq.Context()

        sub_dp0 = context.socket(zmq.SUB)
        sub_dp0.connect("tcp://localhost:5557")
        topic = "kv-events"
        sub_dp0.setsockopt_string(zmq.SUBSCRIBE, topic)
```
**EN:** This range defines concrete test routine(s) `test_kv_events_attn_cp_single_stream_per_dp_rank`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `Decoder`, `Context`, `socket` and `connect`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 302-316: Process control logic / 进程控制逻辑
```python

        # There is only one DP rank in this test, so CP must not create another stream.
        sub_unexpected = context.socket(zmq.SUB)
        sub_unexpected.connect("tcp://localhost:5558")
        sub_unexpected.setsockopt_string(zmq.SUBSCRIBE, topic)

        process = popen_launch_server(
            QWEN3_30B_MODEL_PATH,
            DEFAULT_URL_FOR_TEST,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--kv-events-config",
                '{"publisher": "zmq", "topic": "kv-events"}',
                "--tp-size",
                2,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `socket`, `connect`, `setsockopt_string` and `popen_launch_server`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 317-326: Scenario logic / 场景逻辑
```python
                "--attn-cp-size",
                2,
                "--moe-dp-size",
                2,
                "--enable-prefill-context-parallel",
                "--trust-remote-code",
                "--max-total-tokens",
                4096,
                "--max-running-requests",
                4,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 327-341: Assertions and result checks / 断言与结果检查
```python
                "--disable-cuda-graph",
                "--cuda-graph-max-bs",
                4,
                "--model-loader-extra-config",
                '{"enable_multithread_load": true, "num_threads": 64}',
            ],
        )

        try:
            response = requests.get(f"{DEFAULT_URL_FOR_TEST}/health_generate")
            self.assertEqual(response.status_code, 200)

            for i in range(4):
                response = requests.post(
                    f"{DEFAULT_URL_FOR_TEST}/generate",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get`, `assertEqual` and `post`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 342-351: Scenario logic / 场景逻辑
```python
                    json={
                        "text": (
                            f"KV event context parallelism request {i}: "
                            "write a concise fact about distributed inference."
                        ),
                        "sampling_params": {
                            "temperature": 0,
                            "max_new_tokens": 16,
                        },
                    },
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 352-366: Assertions and result checks / 断言与结果检查
```python
                )
                self.assertEqual(response.status_code, 200)

            batches = []
            stored_hashes = set()
            duplicate_hashes = set()
            unexpected_batches = []
            start = time.time()
            max_wait_s = 15
            min_stored_blocks = 3

            while (time.time() - start) < max_wait_s and (
                len(stored_hashes) < min_stored_blocks
            ):
                if sub_dp0.poll(timeout=100):
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual`, `time`, `and` and `poll`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 367-376: Assertions and result checks / 断言与结果检查
```python
                    _, seq_bytes, payload = sub_dp0.recv_multipart()
                    event_batch = decoder.decode(payload)
                    print(
                        f"DP Rank 0 - EventBatch: ts={event_batch.ts}, "
                        f"attn_dp_rank={event_batch.attn_dp_rank}"
                    )
                    self.assertEqual(
                        event_batch.attn_dp_rank,
                        0,
                        "CP mode with one DP rank should publish events as attn_dp_rank=0",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `recv_multipart`, `decode` and `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 377-391: Assertions and result checks / 断言与结果检查
```python
                    )
                    batches.append(event_batch)

                    for event in event_batch.events:
                        print(f"  DP0 - {event}")
                        self.assertIsInstance(
                            event,
                            (BlockStored, BlockRemoved, AllBlocksCleared),
                            f"Event should be a KV cache event, got {type(event)}",
                        )
                        if isinstance(event, BlockStored):
                            for block_hash in event.block_hashes:
                                if block_hash in stored_hashes:
                                    duplicate_hashes.add(block_hash)
                                stored_hashes.add(block_hash)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `append`, `assertIsInstance`, `type` and `add`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 392-401: Assertions and result checks / 断言与结果检查
```python

                if sub_unexpected.poll(timeout=0):
                    _, seq_bytes, payload = sub_unexpected.recv_multipart()
                    unexpected_batches.append(decoder.decode(payload))

            self.assertGreater(
                len(batches), 0, "Should have received KV cache event batches"
            )
            self.assertGreaterEqual(
                len(stored_hashes),
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `poll`, `recv_multipart`, `append` and `decode`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 402-416: Assertions and result checks / 断言与结果检查
```python
                min_stored_blocks,
                f"Expected at least {min_stored_blocks} stored KV blocks",
            )
            self.assertEqual(
                unexpected_batches,
                [],
                "CP ranks within one DP rank should not create a second KV event stream",
            )
            self.assertEqual(
                duplicate_hashes,
                set(),
                "CP ranks should not publish duplicate BlockStored events for replicated KV blocks",
            )

        finally:
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 417-420: Process control logic / 进程控制逻辑
```python
            sub_dp0.close()
            sub_unexpected.close()
            context.term()
            kill_process_tree(process.pid)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `close`, `term` and `kill_process_tree`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 421-424: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互
- Streaming responses / 流式响应

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `time`, `unittest`
- **Third-party / 第三方库**: `msgspec.msgpack`, `requests`, `zmq`
- **Project Modules / 项目模块**: `sglang.srt.disaggregation.kv_events`, `sglang.srt.utils`, `sglang.test.test_utils`
