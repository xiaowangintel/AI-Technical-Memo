# kv_events.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/disaggregation/kv_events.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file defines KV transfer event structures and related tracking helpers. It supports observability, diagnostics, and debugging for disaggregated data movement. / 该文件定义了 KV 传输事件结构及其追踪辅助逻辑，用于提升解耦数据移动过程中的可观测性、诊断与调试能力。

## Line-by-Line Analysis / 逐行分析
### Lines 1-38: Module-level constants and helper logic
```python
"""
Copyright 2025 SGLang Team
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
"""

"""
KV caching events
"""

import atexit
import enum
import logging
import queue
import threading
import time
from abc import ABC, abstractmethod
from collections import deque
from itertools import count
from queue import Queue
from typing import Any, Callable, Optional, Union

import msgspec
import zmq
from pydantic import BaseModel

logger = logging.getLogger(__name__)


```
**EN:** This block contains module-level constants, helpers, or documentation for KV-transfer event definitions and tracking. It prepares shared state that later classes and functions build on. Notable operations include `getLogger`.
**CN:** 这一段包含与KV 传输事件定义与追踪相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `getLogger`。

### Lines 39-44: Class `EventBatch` declaration
```python
class EventBatch(
    msgspec.Struct,
    array_like=True,  # type: ignore[call-arg]
    omit_defaults=True,  # type: ignore[call-arg]
    gc=False,  # type: ignore[call-arg]
):
```
**EN:** This block declares the class `EventBatch` and establishes its responsibility inside KV-transfer event definitions and tracking. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `EventBatch`.
**CN:** 这一段声明了类 `EventBatch`，并说明它在KV 传输事件定义与追踪中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `EventBatch`。

### Lines 45-47: Supporting state inside `EventBatch`
```python
    ts: float
    events: list[Any]
    attn_dp_rank: Optional[int] = None
```
**EN:** This block adds supporting state or helper logic inside `EventBatch`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `EventBatch` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 50-56: Class `KVCacheEvent` declaration
```python
class KVCacheEvent(
    msgspec.Struct,
    array_like=True,  # type: ignore[call-arg]
    omit_defaults=True,  # type: ignore[call-arg]
    gc=False,  # type: ignore[call-arg]
    tag=True,
):
```
**EN:** This block declares the class `KVCacheEvent` and establishes its responsibility inside KV-transfer event definitions and tracking. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `KVCacheEvent`.
**CN:** 这一段声明了类 `KVCacheEvent`，并说明它在KV 传输事件定义与追踪中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `KVCacheEvent`。

### Lines 57-57: Supporting state inside `KVCacheEvent`
```python
    """Base class for all KV cache-related events"""
```
**EN:** This block adds supporting state or helper logic inside `KVCacheEvent`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `KVCacheEvent` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 60-60: Class `StorageMedium` declaration
```python
class StorageMedium(str, enum.Enum):
```
**EN:** This block declares the class `StorageMedium` and establishes its responsibility inside KV-transfer event definitions and tracking. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `StorageMedium`.
**CN:** 这一段声明了类 `StorageMedium`，并说明它在KV 传输事件定义与追踪中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `StorageMedium`。

### Lines 61-66: Supporting state inside `StorageMedium`
```python
    """Storage tier for KV cache events."""

    GPU = "GPU"  # L1: device HBM
    CPU = "CPU_PINNED"  # L2: host pinned memory
    DISK = "DISK"  # L3: SSD / NVMe
    EXTERNAL = "EXTERNAL"  # L4: shared / remote pool (e.g. Mooncake)
```
**EN:** This block adds supporting state or helper logic inside `StorageMedium`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `pool`.
**CN:** 这一段为 `StorageMedium` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `pool`。

### Lines 69-69: Class `OffloadedState` declaration
```python
class OffloadedState:
```
**EN:** This block declares the class `OffloadedState` and establishes its responsibility inside KV-transfer event definitions and tracking. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `OffloadedState`.
**CN:** 这一段声明了类 `OffloadedState`，并说明它在KV 传输事件定义与追踪中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `OffloadedState`。

### Lines 70-77: Supporting state inside `OffloadedState`
```python
    """
    OffloadedState represents the state of a KV cache block offloaded to the hicache.

    - prefill_len (int): The length of the prefill part of the KV cache block.
    - inc_len (int): The length of the incremental part of the KV cache block.
    - last_hash (Optional[str]): The hash of the last token in the KV cache block.
    """

```
**EN:** This block adds supporting state or helper logic inside `OffloadedState`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `prefill_len`, `inc_len`, `last_hash`.
**CN:** 这一段为 `OffloadedState` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `prefill_len`、`inc_len`、`last_hash`。

### Lines 78-83: Method `__init__`
```python
    def __init__(
        self, prefill_len: int, inc_len: int = 0, last_hash: Optional[str] = None
    ):
        self.prefill_len = prefill_len
        self.inc_len = inc_len
        self.last_hash = last_hash
```
**EN:** This block defines the method `__init__` on `OffloadedState`. It introduces the parameters, setup steps, and the main entry point for this piece of KV-transfer event definitions and tracking. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `OffloadedState`），介绍了参数、初始化步骤，以及这部分KV 传输事件定义与追踪逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 86-86: Class `BlockStored` declaration
```python
class BlockStored(KVCacheEvent):
```
**EN:** This block declares the class `BlockStored` and establishes its responsibility inside KV-transfer event definitions and tracking. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `BlockStored`.
**CN:** 这一段声明了类 `BlockStored`，并说明它在KV 传输事件定义与追踪中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `BlockStored`。

### Lines 87-92: Supporting state inside `BlockStored`
```python
    block_hashes: list[int]
    parent_block_hash: Optional[int]
    token_ids: list[int]
    block_size: int
    lora_id: Optional[int]
    medium: Optional[str] = None
```
**EN:** This block adds supporting state or helper logic inside `BlockStored`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `BlockStored` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 95-95: Class `BlockRemoved` declaration
```python
class BlockRemoved(KVCacheEvent):
```
**EN:** This block declares the class `BlockRemoved` and establishes its responsibility inside KV-transfer event definitions and tracking. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `BlockRemoved`.
**CN:** 这一段声明了类 `BlockRemoved`，并说明它在KV 传输事件定义与追踪中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `BlockRemoved`。

### Lines 96-97: Supporting state inside `BlockRemoved`
```python
    block_hashes: list[int]
    medium: Optional[str] = None
```
**EN:** This block adds supporting state or helper logic inside `BlockRemoved`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `BlockRemoved` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 100-100: Class `AllBlocksCleared` declaration
```python
class AllBlocksCleared(KVCacheEvent):
```
**EN:** This block declares the class `AllBlocksCleared` and establishes its responsibility inside KV-transfer event definitions and tracking. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `AllBlocksCleared`.
**CN:** 这一段声明了类 `AllBlocksCleared`，并说明它在KV 传输事件定义与追踪中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `AllBlocksCleared`。

### Lines 101-101: Supporting state inside `AllBlocksCleared`
```python
    pass
```
**EN:** This block adds supporting state or helper logic inside `AllBlocksCleared`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `AllBlocksCleared` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 104-104: Class `KVEventBatch` declaration
```python
class KVEventBatch(EventBatch):
```
**EN:** This block declares the class `KVEventBatch` and establishes its responsibility inside KV-transfer event definitions and tracking. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `KVEventBatch`.
**CN:** 这一段声明了类 `KVEventBatch`，并说明它在KV 传输事件定义与追踪中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `KVEventBatch`。

### Lines 105-105: Supporting state inside `KVEventBatch`
```python
    events: list[Union[BlockStored, BlockRemoved, AllBlocksCleared]]
```
**EN:** This block adds supporting state or helper logic inside `KVEventBatch`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `KVEventBatch` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 108-108: Class `EventPublisher` declaration
```python
class EventPublisher(ABC):
```
**EN:** This block declares the class `EventPublisher` and establishes its responsibility inside KV-transfer event definitions and tracking. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `EventPublisher`.
**CN:** 这一段声明了类 `EventPublisher`，并说明它在KV 传输事件定义与追踪中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `EventPublisher`。

### Lines 109-121: Supporting state inside `EventPublisher`
```python
    """
    Lightweight publisher for EventBatch batches with
    support for DP attention.

    In DP attention - each rank has its own Scheduler and
    KV cache instance in order to avoid duplicate events
    and ensure proper event attribution. In our implementation

    - Each DP rank has its own EventPublisher
    - Publishers annotate events with the dp rank
    - This allows consumers to distinguish events from different DP ranks
    """

```
**EN:** This block adds supporting state or helper logic inside `EventPublisher`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `EventPublisher` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 122-128: Method `publish`
```python
    @abstractmethod
    def publish(self, events: EventBatch) -> None:
        """Emit events in order.

        Implementations should guarantee at-least-once delivery and
        monotonic ordering (e.g., via sequence numbers).
        """
```
**EN:** This block defines the method `publish` on `EventPublisher`. It introduces the parameters, setup steps, and the main entry point for this piece of KV-transfer event definitions and tracking. Definitions introduced here include `publish`. Notable operations include `ordering`.
**CN:** 这一段定义了method `publish`（属于 `EventPublisher`），介绍了参数、初始化步骤，以及这部分KV 传输事件定义与追踪逻辑的主要入口。 此处引入的定义包括 `publish`。 值得注意的操作包括 `ordering`。

### Lines 130-132: Method `shutdown`
```python
    @abstractmethod
    def shutdown(self) -> None:
        """Shutdown the publisher."""
```
**EN:** This block defines the method `shutdown` on `EventPublisher`. It introduces the parameters, setup steps, and the main entry point for this piece of KV-transfer event definitions and tracking. Definitions introduced here include `shutdown`.
**CN:** 这一段定义了method `shutdown`（属于 `EventPublisher`），介绍了参数、初始化步骤，以及这部分KV 传输事件定义与追踪逻辑的主要入口。 此处引入的定义包括 `shutdown`。

### Lines 135-135: Class `NullEventPublisher` declaration
```python
class NullEventPublisher(EventPublisher):
```
**EN:** This block declares the class `NullEventPublisher` and establishes its responsibility inside KV-transfer event definitions and tracking. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `NullEventPublisher`.
**CN:** 这一段声明了类 `NullEventPublisher`，并说明它在KV 传输事件定义与追踪中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `NullEventPublisher`。

### Lines 136-137: Supporting state inside `NullEventPublisher`
```python
    """No-op implementation (default when disabled)."""

```
**EN:** This block adds supporting state or helper logic inside `NullEventPublisher`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `implementation`.
**CN:** 这一段为 `NullEventPublisher` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `implementation`。

### Lines 138-139: Method `publish`
```python
    def publish(self, events) -> None:
        return
```
**EN:** This block defines the method `publish` on `NullEventPublisher`. It introduces the parameters, setup steps, and the main entry point for this piece of KV-transfer event definitions and tracking. Definitions introduced here include `publish`.
**CN:** 这一段定义了method `publish`（属于 `NullEventPublisher`），介绍了参数、初始化步骤，以及这部分KV 传输事件定义与追踪逻辑的主要入口。 此处引入的定义包括 `publish`。

### Lines 141-142: Method `shutdown`
```python
    def shutdown(self) -> None:
        return
```
**EN:** This block defines the method `shutdown` on `NullEventPublisher`. It introduces the parameters, setup steps, and the main entry point for this piece of KV-transfer event definitions and tracking. Definitions introduced here include `shutdown`.
**CN:** 这一段定义了method `shutdown`（属于 `NullEventPublisher`），介绍了参数、初始化步骤，以及这部分KV 传输事件定义与追踪逻辑的主要入口。 此处引入的定义包括 `shutdown`。

### Lines 145-145: Class `ZmqEventPublisher` declaration
```python
class ZmqEventPublisher(EventPublisher):
```
**EN:** This block declares the class `ZmqEventPublisher` and establishes its responsibility inside KV-transfer event definitions and tracking. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `ZmqEventPublisher`.
**CN:** 这一段声明了类 `ZmqEventPublisher`，并说明它在KV 传输事件定义与追踪中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `ZmqEventPublisher`。

### Lines 146-171: Supporting state inside `ZmqEventPublisher`
```python
    """Reliable PUB/ROUTER publisher with an in-memory replay buffer.

    Spawns a separate thread to handle publishing from a queue.

    Parameters
    ----------
    endpoint:
        PUB address. Use ``tcp://*:5557`` to bind or ``tcp://host:5557`` to
        connect.
    replay_endpoint:
        Optional ROUTER address for replay requests. When given, subscribers can
        request missed batches by sending the starting sequence number as an
        8-byte big-endian integer.
    buffer_steps:
        Number of past batches to keep for replay.
    hwm:
        ZeroMQ high-water-mark for PUB socket.
    max_queue_size:
        Maximum number of events to buffer in memory.
    topic:
        Topic to publish events to.
    """

    SHUTDOWN_TIMEOUT: float = 1.0
    END_SEQ = (-1).to_bytes(8, "big", signed=True)

```
**EN:** This block adds supporting state or helper logic inside `ZmqEventPublisher`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `to_bytes`.
**CN:** 这一段为 `ZmqEventPublisher` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `to_bytes`。

### Lines 172-174: Method `__init__` signature and setup
```python
    def __init__(
        self,
        attn_dp_rank: int,
```
**EN:** This block defines the method `__init__` on `ZmqEventPublisher`. It introduces the parameters, setup steps, and the main entry point for this piece of KV-transfer event definitions and tracking. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `ZmqEventPublisher`），介绍了参数、初始化步骤，以及这部分KV 传输事件定义与追踪逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 175-196: Method `__init__` logic (part 1)
```python
        endpoint: str = "tcp://*:5557",
        replay_endpoint: Optional[str] = None,
        buffer_steps: int = 10_000,
        hwm: int = 100_000,
        max_queue_size: int = 100_000,
        topic: str = "",
    ) -> None:
        # Storage
        self._event_queue = Queue[Optional[EventBatch]](maxsize=max_queue_size)
        self._buffer = deque[tuple[int, bytes]](maxlen=buffer_steps)

        # ZMQ sockets
        self._ctx = zmq.Context.instance()
        self._pub: Optional[zmq.Socket] = None
        self._replay: Optional[zmq.Socket] = None
        self._dp_rank = attn_dp_rank
        self._endpoint = self.offset_endpoint_port(endpoint, self._dp_rank)
        self._replay_endpoint = self.offset_endpoint_port(
            replay_endpoint, self._dp_rank
        )
        self._hwm = hwm
        self._socket_setup()
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding KV-transfer event definitions and tracking workflow. Notable operations include `instance`, `offset_endpoint_port`, `_socket_setup`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的KV 传输事件定义与追踪工作流。 值得注意的操作包括 `instance`、`offset_endpoint_port`、`_socket_setup`。

### Lines 197-211: Method `__init__` logic (part 2)
```python

        # Payload
        self._seq_gen = count()
        self._topic_bytes = topic.encode("utf-8")

        # Thread
        self._running = True
        logger.info("Starting ZMQ publisher thread")

        self._thread = threading.Thread(
            target=self._publisher_thread, daemon=True, name="zmq-publisher"
        )
        self._thread.start()

        atexit.register(self.shutdown)
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding KV-transfer event definitions and tracking workflow. Notable operations include `count`, `encode`, `info`, `Thread`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的KV 传输事件定义与追踪工作流。 值得注意的操作包括 `count`、`encode`、`info`、`Thread`。

### Lines 213-218: Method `publish`
```python
    def publish(self, events: EventBatch) -> None:
        if not self._running:
            raise RuntimeError("Publisher is closed")
        if events.attn_dp_rank is None:
            events.attn_dp_rank = self._dp_rank
        self._event_queue.put(events)
```
**EN:** This block defines the method `publish` on `ZmqEventPublisher`. It introduces the parameters, setup steps, and the main entry point for this piece of KV-transfer event definitions and tracking. Definitions introduced here include `publish`. Notable operations include `RuntimeError`, `put`.
**CN:** 这一段定义了method `publish`（属于 `ZmqEventPublisher`），介绍了参数、初始化步骤，以及这部分KV 传输事件定义与追踪逻辑的主要入口。 此处引入的定义包括 `publish`。 值得注意的操作包括 `RuntimeError`、`put`。

### Lines 220-221: Method `shutdown` signature and setup
```python
    def shutdown(self) -> None:
        """Stop the publisher thread and clean up resources."""
```
**EN:** This block defines the method `shutdown` on `ZmqEventPublisher`. It introduces the parameters, setup steps, and the main entry point for this piece of KV-transfer event definitions and tracking. Definitions introduced here include `shutdown`.
**CN:** 这一段定义了method `shutdown`（属于 `ZmqEventPublisher`），介绍了参数、初始化步骤，以及这部分KV 传输事件定义与追踪逻辑的主要入口。 此处引入的定义包括 `shutdown`。

### Lines 222-240: Method `shutdown` logic (part 1)
```python
        self._running = False
        self._event_queue.put_nowait(None)

        start = time.time()
        pending_items = True
        while pending_items and (time.time() - start < self.SHUTDOWN_TIMEOUT):
            pending_items = not self._event_queue.empty()
            if pending_items:
                time.sleep(0.1)

        if pending_items:
            logger.warning(
                "Warning: Queue still has %s items after %s seconds timeout",
                self._event_queue.qsize(),
                self.SHUTDOWN_TIMEOUT,
            )

        if self._thread.is_alive():
            self._thread.join(timeout=self.SHUTDOWN_TIMEOUT)
```
**EN:** This block continues `shutdown` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding KV-transfer event definitions and tracking workflow. Notable operations include `put_nowait`, `time`, `and`, `empty`.
**CN:** 这一段延续了 `shutdown` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的KV 传输事件定义与追踪工作流。 值得注意的操作包括 `put_nowait`、`time`、`and`、`empty`。

### Lines 241-249: Method `shutdown` logic (part 2)
```python

        # Clean up ZMQ resources
        try:
            if self._pub is not None:
                self._pub.close(linger=0)
            if self._replay is not None:
                self._replay.close(linger=0)
        finally:
            pass  # Do not terminate context; other sockets may use it
```
**EN:** This block continues `shutdown` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding KV-transfer event definitions and tracking workflow. Notable operations include `close`.
**CN:** 这一段延续了 `shutdown` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的KV 传输事件定义与追踪工作流。 值得注意的操作包括 `close`。

### Lines 251-254: Method `_socket_setup` signature and setup
```python
    def _socket_setup(self) -> None:
        """Initialize sockets
        https://pyzmq.readthedocs.io/en/v19.0.0/morethanbindings.html#thread-safety
        """
```
**EN:** This block defines the method `_socket_setup` on `ZmqEventPublisher`. It introduces the parameters, setup steps, and the main entry point for this piece of KV-transfer event definitions and tracking. Definitions introduced here include `_socket_setup`.
**CN:** 这一段定义了method `_socket_setup`（属于 `ZmqEventPublisher`），介绍了参数、初始化步骤，以及这部分KV 传输事件定义与追踪逻辑的主要入口。 此处引入的定义包括 `_socket_setup`。

### Lines 255-271: Method `_socket_setup` logic (part 1)
```python
        if self._pub is None:
            self._pub = self._ctx.socket(zmq.PUB)
            self._pub.set_hwm(self._hwm)
            # Heuristic: bind if wildcard / * present, else connect.
            # bind stable, connect volatile convention
            if (
                "*" in self._endpoint
                or "::" in self._endpoint
                or self._endpoint.startswith("ipc://")
                or self._endpoint.startswith("inproc://")
            ):
                logger.debug(
                    f"ZmqEventPublisher socket publisher_endpoint bind to {self._endpoint}"
                )
                self._pub.bind(self._endpoint)
            else:
                self._pub.connect(self._endpoint)
```
**EN:** This block continues `_socket_setup` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding KV-transfer event definitions and tracking workflow. Notable operations include `socket`, `set_hwm`, `startswith`, `debug`.
**CN:** 这一段延续了 `_socket_setup` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的KV 传输事件定义与追踪工作流。 值得注意的操作包括 `socket`、`set_hwm`、`startswith`、`debug`。

### Lines 272-282: Method `_socket_setup` logic (part 2)
```python

        # Set up replay socket: use ROUTER
        # 1) handles multiple REQ clients (identities)
        # 2) lets us send back one request → many replies (streamed events)
        # 3) works in our non‑blocking poll loop alongside PUB
        if self._replay_endpoint is not None:
            self._replay = self._ctx.socket(zmq.ROUTER)
            logger.debug(
                f"ZmqEventPublisher socket replay_endpoint bind to {self._replay_endpoint}"
            )
            self._replay.bind(self._replay_endpoint)
```
**EN:** This block continues `_socket_setup` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding KV-transfer event definitions and tracking workflow. Notable operations include `clients`, `replies`, `socket`, `debug`.
**CN:** 这一段延续了 `_socket_setup` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的KV 传输事件定义与追踪工作流。 值得注意的操作包括 `clients`、`replies`、`socket`、`debug`。

### Lines 284-285: Method `_publisher_thread` signature and setup
```python
    def _publisher_thread(self) -> None:
        """Background thread that processes the event queue."""
```
**EN:** This block defines the method `_publisher_thread` on `ZmqEventPublisher`. It introduces the parameters, setup steps, and the main entry point for this piece of KV-transfer event definitions and tracking. Definitions introduced here include `_publisher_thread`.
**CN:** 这一段定义了method `_publisher_thread`（属于 `ZmqEventPublisher`），介绍了参数、初始化步骤，以及这部分KV 传输事件定义与追踪逻辑的主要入口。 此处引入的定义包括 `_publisher_thread`。

### Lines 286-307: Method `_publisher_thread` logic (part 1)
```python
        self._pack = msgspec.msgpack.Encoder()

        assert self._pub is not None  # narrows type for mypy

        while self._running or self._event_queue.qsize() > 0:
            # --- replay (non-critical) ---------------------------------
            if self._replay is not None and self._replay.poll(0):
                try:
                    self._service_replay()
                except Exception as e:
                    logger.exception("Error in replay: %s", e)

            # --- main queue (critical) ---------------------------------
            try:
                event = self._event_queue.get(timeout=0.1)
                if event is None:
                    break  # Sentinel received, exit thread
            except queue.Empty:
                continue

            try:
                seq = next(self._seq_gen)
```
**EN:** This block continues `_publisher_thread` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding KV-transfer event definitions and tracking workflow. Notable operations include `Encoder`, `qsize`, `replay`, `poll`.
**CN:** 这一段延续了 `_publisher_thread` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的KV 传输事件定义与追踪工作流。 值得注意的操作包括 `Encoder`、`qsize`、`replay`、`poll`。

### Lines 308-319: Method `_publisher_thread` logic (part 2)
```python

                payload = self._pack.encode(event)
                seq_bytes = seq.to_bytes(8, "big")
                self._pub.send_multipart((self._topic_bytes, seq_bytes, payload))

                self._buffer.append((seq, payload))
                self._event_queue.task_done()

            except Exception as e:
                # Publishing failed;  back-off a bit to avoid a tight error loop
                logger.exception("Error in publisher thread: %s", e)
                time.sleep(0.1)
```
**EN:** This block continues `_publisher_thread` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding KV-transfer event definitions and tracking workflow. Notable operations include `encode`, `to_bytes`, `send_multipart`, `append`.
**CN:** 这一段延续了 `_publisher_thread` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的KV 传输事件定义与追踪工作流。 值得注意的操作包括 `encode`、`to_bytes`、`send_multipart`、`append`。

### Lines 321-342: Method `_service_replay`
```python
    def _service_replay(self) -> None:
        """If a replay request is waiting, send buffered batches."""
        assert self._replay is not None  # narrows type for mypy

        frame = self._replay.recv_multipart()
        if len(frame) != 3:
            logger.warning("Invalid replay request: %s", frame)
            return
        client_id, _, start_seq_bytes = frame
        start_seq = int.from_bytes(start_seq_bytes, "big")

        for seq, buf in self._buffer:
            if seq >= start_seq:
                # [identity, empty_delim, seq_bytes, payload]
                # (identity, empty_delim) are stripped off by the router
                # receiving payload is (seq_bytes, payload)
                self._replay.send_multipart(
                    (client_id, b"", seq.to_bytes(8, "big"), buf)
                )
        # Send end of sequence marker
        # receiving payload is (-1, b""")
        self._replay.send_multipart((client_id, b"", self.END_SEQ, b""))
```
**EN:** This block defines the method `_service_replay` on `ZmqEventPublisher`. It introduces the parameters, setup steps, and the main entry point for this piece of KV-transfer event definitions and tracking. Definitions introduced here include `_service_replay`. Notable operations include `recv_multipart`, `warning`, `from_bytes`, `is`.
**CN:** 这一段定义了method `_service_replay`（属于 `ZmqEventPublisher`），介绍了参数、初始化步骤，以及这部分KV 传输事件定义与追踪逻辑的主要入口。 此处引入的定义包括 `_service_replay`。 值得注意的操作包括 `recv_multipart`、`warning`、`from_bytes`、`is`。

### Lines 344-359: Method `offset_endpoint_port` signature and setup
```python
    @staticmethod
    def offset_endpoint_port(
        endpoint: Optional[str], data_parallel_rank: int
    ) -> Optional[str]:
        """Helper function to offset the port in an endpoint by
            the data parallel rank.

        Args:
            endpoint: The endpoint string
                (e.g., "tcp://*:5557" or "inproc://cache")
            data_parallel_rank: The data parallel rank to offset by

        Returns:
            The endpoint with the port offset by data_parallel_rank
                or suffix appended
        """
```
**EN:** This block defines the method `offset_endpoint_port` on `ZmqEventPublisher`. It introduces the parameters, setup steps, and the main entry point for this piece of KV-transfer event definitions and tracking. Definitions introduced here include `offset_endpoint_port`. Notable operations include `string`.
**CN:** 这一段定义了method `offset_endpoint_port`（属于 `ZmqEventPublisher`），介绍了参数、初始化步骤，以及这部分KV 传输事件定义与追踪逻辑的主要入口。 此处引入的定义包括 `offset_endpoint_port`。 值得注意的操作包括 `string`。

### Lines 360-375: Method `offset_endpoint_port` logic (part 1)
```python
        # Do nothing if input is None or data_parallel_rank is 0
        if not endpoint or data_parallel_rank == 0:
            return endpoint

        if "inproc" in endpoint:
            return f"{endpoint}_dp{data_parallel_rank}"
        if "tcp" in endpoint:
            if endpoint and ":" in endpoint:
                # Get everything after the last colon (the port)
                last_colon_idx = endpoint.rfind(":")
                base_addr = endpoint[:last_colon_idx]
                base_port = int(endpoint[last_colon_idx + 1 :])
                new_port = base_port + data_parallel_rank
                return f"{base_addr}:{new_port}"
            return endpoint
        raise ValueError("Invalid endpoint: must contain 'inproc' or 'tcp'")
```
**EN:** This block continues `offset_endpoint_port` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding KV-transfer event definitions and tracking workflow. Notable operations include `colon`, `rfind`, `ValueError`.
**CN:** 这一段延续了 `offset_endpoint_port` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的KV 传输事件定义与追踪工作流。 值得注意的操作包括 `colon`、`rfind`、`ValueError`。

### Lines 378-378: Class `KVEventsConfig` declaration
```python
class KVEventsConfig(BaseModel):
```
**EN:** This block declares the class `KVEventsConfig` and establishes its responsibility inside KV-transfer event definitions and tracking. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `KVEventsConfig`.
**CN:** 这一段声明了类 `KVEventsConfig`，并说明它在KV 传输事件定义与追踪中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `KVEventsConfig`。

### Lines 379-401: Supporting state inside `KVEventsConfig`
```python
    """Configuration for KV event publishing."""

    publisher: str = "null"
    """The publisher to use for publishing kv events. Can be "null", "zmq".
    """

    endpoint: str = "tcp://*:5557"
    """The zmq endpoint to use for publishing kv events.
    """

    replay_endpoint: Optional[str] = None
    """The zmq endpoint to use for replaying kv events.
    """

    buffer_steps: int = 10_000
    """The number of steps to cache for replay endpoint. Will only save
    events from the last N steps for the replay endpoint.
    """

    hwm: int = 100_000
    """The zmq high water mark for the event publisher. After queueing N events,
    events will start dropping if the consumer is not keeping up.
    """
```
**EN:** This block adds supporting state or helper logic inside `KVEventsConfig`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `KVEventsConfig` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 402-411: Supporting state inside `KVEventsConfig`
```python

    max_queue_size: int = 100_000
    """The maximum number of events to queue while waiting for publishing.
    """

    topic: str = ""
    """The topic to use for the event publisher. Consumers can subscribe to
    this topic to receive events.
    """

```
**EN:** This block adds supporting state or helper logic inside `KVEventsConfig`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `KVEventsConfig` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 412-415: Method `from_cli`
```python
    @classmethod
    def from_cli(cls, cli_value: str) -> "KVEventsConfig":
        """Parse the CLI value for the event publisher config."""
        return KVEventsConfig.model_validate_json(cli_value)
```
**EN:** This block defines the method `from_cli` on `KVEventsConfig`. It introduces the parameters, setup steps, and the main entry point for this piece of KV-transfer event definitions and tracking. Definitions introduced here include `from_cli`. Notable operations include `model_validate_json`.
**CN:** 这一段定义了method `from_cli`（属于 `KVEventsConfig`），介绍了参数、初始化步骤，以及这部分KV 传输事件定义与追踪逻辑的主要入口。 此处引入的定义包括 `from_cli`。 值得注意的操作包括 `model_validate_json`。

### Lines 418-418: Class `EventPublisherFactory` declaration
```python
class EventPublisherFactory:
```
**EN:** This block declares the class `EventPublisherFactory` and establishes its responsibility inside KV-transfer event definitions and tracking. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `EventPublisherFactory`.
**CN:** 这一段声明了类 `EventPublisherFactory`，并说明它在KV 传输事件定义与追踪中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `EventPublisherFactory`。

### Lines 419-423: Supporting state inside `EventPublisherFactory`
```python
    _registry: dict[str, Callable[..., EventPublisher]] = {
        "null": NullEventPublisher,
        "zmq": ZmqEventPublisher,
    }

```
**EN:** This block adds supporting state or helper logic inside `EventPublisherFactory`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `EventPublisherFactory` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 424-428: Method `register_publisher`
```python
    @classmethod
    def register_publisher(cls, name: str, ctor: Callable[..., EventPublisher]) -> None:
        if name in cls._registry:
            raise KeyError(f"publisher '{name}' already registered")
        cls._registry[name] = ctor
```
**EN:** This block defines the method `register_publisher` on `EventPublisherFactory`. It introduces the parameters, setup steps, and the main entry point for this piece of KV-transfer event definitions and tracking. Definitions introduced here include `register_publisher`. Notable operations include `KeyError`.
**CN:** 这一段定义了method `register_publisher`（属于 `EventPublisherFactory`），介绍了参数、初始化步骤，以及这部分KV 传输事件定义与追踪逻辑的主要入口。 此处引入的定义包括 `register_publisher`。 值得注意的操作包括 `KeyError`。

### Lines 430-443: Method `create`
```python
    @classmethod
    def create(cls, config: Optional[str], attn_dp_rank: int = 0) -> EventPublisher:
        """Create publisher from a config mapping."""
        if not config:
            return NullEventPublisher()
        config = KVEventsConfig.from_cli(config)
        config_dict = config.model_dump()

        kind = config_dict.pop("publisher", "null")
        try:
            constructor = cls._registry[kind]
        except KeyError as exc:
            raise ValueError(f"Unknown event publisher '{kind}'") from exc
        return constructor(attn_dp_rank=attn_dp_rank, **config_dict)
```
**EN:** This block defines the method `create` on `EventPublisherFactory`. It introduces the parameters, setup steps, and the main entry point for this piece of KV-transfer event definitions and tracking. Definitions introduced here include `create`. Notable operations include `NullEventPublisher`, `from_cli`, `model_dump`, `pop`.
**CN:** 这一段定义了method `create`（属于 `EventPublisherFactory`），介绍了参数、初始化步骤，以及这部分KV 传输事件定义与追踪逻辑的主要入口。 此处引入的定义包括 `create`。 值得注意的操作包括 `NullEventPublisher`、`from_cli`、`model_dump`、`pop`。

## Key Concepts / 关键概念
- `EventBatch`: Class that encapsulates event batch behavior in this module. / `EventBatch`：封装与“事件批次”相关行为的类。
- `KVCacheEvent`: Class that encapsulates kvcache event behavior in this module. / `KVCacheEvent`：封装与“kvcache事件”相关行为的类。
- `StorageMedium`: Class that encapsulates storage medium behavior in this module. / `StorageMedium`：封装与“storagemedium”相关行为的类。
- `OffloadedState`: Class that encapsulates offloaded state behavior in this module. / `OffloadedState`：封装与“offloaded状态”相关行为的类。
- `BlockStored`: Class that encapsulates block stored behavior in this module. / `BlockStored`：封装与“blockstored”相关行为的类。
- `BlockRemoved`: Class that encapsulates block removed behavior in this module. / `BlockRemoved`：封装与“blockremoved”相关行为的类。
- `AllBlocksCleared`: Class that encapsulates all blocks cleared behavior in this module. / `AllBlocksCleared`：封装与“allblockscleared”相关行为的类。
- `KVEventBatch`: Class that encapsulates kvevent batch behavior in this module. / `KVEventBatch`：封装与“kvevent批次”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `atexit`, `enum`, `logging`, `queue`, `threading`, `time`, `abc`, `collections`, `itertools`, `typing`
- **External packages / 外部依赖**: `msgspec`, `zmq`, `pydantic`
