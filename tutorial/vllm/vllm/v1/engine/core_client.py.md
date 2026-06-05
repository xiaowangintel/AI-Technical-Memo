# core_client.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/engine/core_client.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `EngineCoreClient`, `InprocClient`, `BackgroundResources` for the V1 `engine` subsystem. / 为 V1 的 `engine` 子系统实现 `EngineCoreClient`, `InprocClient`, `BackgroundResources`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import asyncio
import contextlib
import queue
import sys
import uuid
import weakref
from abc import ABC, abstractmethod
from collections import defaultdict, deque
from collections.abc import Awaitable, Callable, Sequence
from concurrent.futures import Future
from dataclasses import dataclass
from multiprocessing.queues import Queue
from threading import Thread
from typing import Any, TypeAlias, TypeVar

import msgspec.msgpack
import zmq
import zmq.asyncio

from vllm.config import VllmConfig
from vllm.envs import VLLM_ENGINE_READY_TIMEOUT_S
from vllm.logger import init_logger
from vllm.lora.request import LoRARequest
from vllm.tasks import SupportedTask
from vllm.tracing import instrument
from vllm.utils.async_utils import in_loop
from vllm.utils.network_utils import (
    close_sockets,
    get_open_zmq_inproc_path,
    make_zmq_socket,
)
from vllm.v1.engine import (
    EEP_NOTIFICATION_CALL_ID,
    EEPNotificationType,
    EngineCoreOutputs,
    EngineCoreReadyResponse,
    EngineCoreRequest,
    EngineCoreRequestType,
    PauseMode,
    ReconfigureDistributedRequest,
    ReconfigureRankType,
    UtilityOutput,
)
from vllm.v1.engine.coordinator import DPCoordinator
from vllm.v1.engine.core import EngineCore, EngineCoreProc
from vllm.v1.engine.exceptions import EngineDeadError
from vllm.v1.engine.tensor_ipc import TensorIpcSender
from vllm.v1.engine.utils import (
    CoreEngineActorManager,
    CoreEngineProcManager,
    get_engine_zmq_addresses,
    launch_core_engines,
)
from vllm.v1.executor import Executor
from vllm.v1.pool.late_interaction import get_late_interaction_engine_index
from vllm.v1.serial_utils import MsgpackDecoder, MsgpackEncoder, bytestr

logger = init_logger(__name__)

AnyFuture: TypeAlias = asyncio.Future[Any] | Future[Any]

_R = TypeVar("_R")  # Return type for collective_rpc

EngineIdentity = bytes
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `AnyFuture`, `_R`, `EngineIdentity`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `AnyFuture`, `_R`, `EngineIdentity`。

### `EngineCoreClient` class / `EngineCoreClient` 类
```python
class EngineCoreClient(ABC):
    """
    EngineCoreClient: subclasses handle different methods for pushing
        and pulling from the EngineCore for asyncio / multiprocessing.

    Subclasses:
    * InprocClient: In process EngineCore (for V0-style LLMEngine use)
    * SyncMPClient: ZMQ + background proc EngineCore (for LLM)
    * AsyncMPClient: ZMQ + background proc EngineCore w/ asyncio (for AsyncLLM)
    """
```
**EN:** Declares the `EngineCoreClient` interface. Downstream implementations are expected to provide methods such as `make_client`, `make_async_mp_client`, `shutdown`, `get_output`, `get_supported_tasks`, `add_request`.
**CN:** `EngineCoreClient` 声明了一组接口约定。下游实现需要提供 `make_client`, `make_async_mp_client`, `shutdown`, `get_output`, `get_supported_tasks`, `add_request` 等方法。

### `EngineCoreClient.execute_dummy_batch` method / `EngineCoreClient.execute_dummy_batch` 方法
```python
    def execute_dummy_batch(self) -> None:
        raise NotImplementedError
```
**EN:** This method executes the main operation within `EngineCoreClient`.
**CN:** 该方法会执行主要操作，其作用域位于`EngineCoreClient`。

### `EngineCoreClient.execute_dummy_batch_async` method / `EngineCoreClient.execute_dummy_batch_async` 方法
```python
    async def execute_dummy_batch_async(self) -> None:
        raise NotImplementedError
```
**EN:** This method executes the main operation within `EngineCoreClient`.
**CN:** 该方法会执行主要操作，其作用域位于`EngineCoreClient`。

### `EngineCoreClient.collective_rpc` method / `EngineCoreClient.collective_rpc` 方法
```python
    def collective_rpc(
        self,
        method: str | Callable[..., _R],
        timeout: float | None = None,
        args: tuple = (),
        kwargs: dict[str, Any] | None = None,
    ) -> list[_R]:
        raise NotImplementedError
```
**EN:** This method collects data from collaborators within `EngineCoreClient`.
**CN:** 该方法会从协作组件收集数据，其作用域位于`EngineCoreClient`。

### `EngineCoreClient.dp_engines_running` method / `EngineCoreClient.dp_engines_running` 方法
```python
    def dp_engines_running(self) -> bool:
        """Returns True if data parallel engines are collectively in a
        running state."""
        raise NotImplementedError
```
**EN:** This method implements `dp_engines_running` within `EngineCoreClient`. The docstring frames it as: Returns True if data parallel engines are collectively in a running state.
**CN:** 该方法会实现 `dp_engines_running`，其作用域位于`EngineCoreClient`。

### `InprocClient` class / `InprocClient` 类
```python
class InprocClient(EngineCoreClient):
    """
    InprocClient: client for in-process EngineCore. Intended
    for use in LLMEngine for V0-style add_request() and step()
        EngineCore setup in this process (no busy loop).

        * pushes EngineCoreRequest directly into the EngineCore
        * pulls EngineCoreOutputs by stepping the EngineCore
    """
```
**EN:** Introduces the `InprocClient` class on top of `EngineCoreClient`. Core methods include `__init__`, `get_output`, `get_supported_tasks`, `add_request`, `abort_requests`, `shutdown`. Docstring signal: InprocClient: client for in-process EngineCore.
**CN:** 这里定义 `InprocClient` 类，其基类包括 `EngineCoreClient`。核心方法包括 `__init__`, `get_output`, `get_supported_tasks`, `add_request`, `abort_requests`, `shutdown`。

### `InprocClient.execute_dummy_batch` method / `InprocClient.execute_dummy_batch` 方法
```python
    def execute_dummy_batch(self) -> None:
        self.engine_core.execute_dummy_batch()
```
**EN:** This method executes the main operation within `InprocClient`. Key calls include `execute_dummy_batch`.
**CN:** 该方法会执行主要操作，其作用域位于`InprocClient`。 关键调用包括 `execute_dummy_batch`。

### `InprocClient.save_sharded_state` method / `InprocClient.save_sharded_state` 方法
```python
    def save_sharded_state(
        self, path: str, pattern: str | None = None, max_size: int | None = None
    ) -> None:
        self.engine_core.save_sharded_state(path, pattern, max_size)
```
**EN:** This method persists computed state within `InprocClient`. Key calls include `save_sharded_state`.
**CN:** 该方法会持久化计算结果，其作用域位于`InprocClient`。 关键调用包括 `save_sharded_state`。

### `InprocClient.collective_rpc` method / `InprocClient.collective_rpc` 方法
```python
    def collective_rpc(
        self,
        method: str | Callable[..., _R],
        timeout: float | None = None,
        args: tuple = (),
        kwargs: dict[str, Any] | None = None,
    ) -> list[_R]:
        return self.engine_core.collective_rpc(method, timeout, args, kwargs)
```
**EN:** This method collects data from collaborators within `InprocClient`. Key calls include `collective_rpc`.
**CN:** 该方法会从协作组件收集数据，其作用域位于`InprocClient`。 关键调用包括 `collective_rpc`。

### `InprocClient.dp_engines_running` method / `InprocClient.dp_engines_running` 方法
```python
    def dp_engines_running(self) -> bool:
        return False
```
**EN:** This method implements `dp_engines_running` within `InprocClient`.
**CN:** 该方法会实现 `dp_engines_running`，其作用域位于`InprocClient`。

### `BackgroundResources` class / `BackgroundResources` 类
```python
@dataclass
class BackgroundResources:
    """Used as a finalizer for clean shutdown, avoiding
    circular reference back to the client object."""

    ctx: zmq.Context
    # If CoreEngineProcManager, it manages local engines;
    # if CoreEngineActorManager, it manages all engines.
    engine_manager: CoreEngineProcManager | CoreEngineActorManager | None = None
    coordinator: DPCoordinator | None = None
    output_socket: zmq.Socket | zmq.asyncio.Socket | None = None
    input_socket: zmq.Socket | zmq.asyncio.Socket | None = None
    first_req_send_socket: zmq.asyncio.Socket | None = None
    first_req_rcv_socket: zmq.asyncio.Socket | None = None
    stats_update_socket: zmq.asyncio.Socket | None = None
    output_queue_task: asyncio.Task | None = None
    stats_update_task: asyncio.Task | None = None
    shutdown_path: str | None = None

    # Set if any of the engines are dead. Here so that the output
    # processing threads can access it without holding a ref to the client.
    engine_dead: bool = False
```
**EN:** Uses `@dataclass` to package related state for `BackgroundResources`. Typical fields include `ctx`, `engine_manager`, `coordinator`, `output_socket`, `input_socket`, `first_req_send_socket`.
**CN:** `BackgroundResources` 使用 `@dataclass` 打包相关状态。典型字段包括 `ctx`, `engine_manager`, `coordinator`, `output_socket`, `input_socket`, `first_req_send_socket`。

### `BackgroundResources.__call__` method / `BackgroundResources.__call__` 方法
```python
    def __call__(self):
        """Clean up background resources."""

        self.engine_dead = True
        if self.engine_manager is not None:
            self.engine_manager.shutdown()
        if self.coordinator is not None:
            self.coordinator.shutdown()

        if isinstance(self.output_socket, zmq.asyncio.Socket):
            # Async case.
            loop = self.output_queue_task._loop if self.output_queue_task else None

            sockets = (
                self.output_socket,
                self.input_socket,
                self.first_req_send_socket,
                self.first_req_rcv_socket,
                self.stats_update_socket,
            )

            tasks = (self.output_queue_task, self.stats_update_task)

            def close_sockets_and_tasks():
                close_sockets(sockets)
                for task in tasks:
                    if task is not None and not task.done():
                        with contextlib.suppress(Exception):
                            task.cancel()

            if loop is not None:
                if in_loop(loop):
                    close_sockets_and_tasks()
                elif not loop.is_closed():
                    loop.call_soon_threadsafe(close_sockets_and_tasks)
            else:
                # Loop has been closed, try to clean up directly.
                del tasks
                del close_sockets_and_tasks
                close_sockets(sockets)
                del self.output_queue_task
                del self.stats_update_task
        else:
            # Sync case.

            # ZMQ context termination can hang if the sockets
            # aren't explicitly closed first.
            close_sockets((self.output_socket, self.input_socket))

            if self.shutdown_path is not None:
                # We must ensure that the sync output socket is
                # closed cleanly in its own thread.
                with self.ctx.socket(zmq.PAIR) as shutdown_sender:
                    shutdown_sender.connect(self.shutdown_path)
                    # Send shutdown signal.
                    shutdown_sender.send(b"")
```
**EN:** This method implements `__call__` within `BackgroundResources`. The docstring frames it as: Clean up background resources. Key calls include `isinstance`, `shutdown`, `close_sockets`, `in_loop`, `close_sockets_and_tasks`, `socket`. It touches state such as `engine_dead`. The control flow contains 9 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `__call__`，其作用域位于`BackgroundResources`。 关键调用包括 `isinstance`, `shutdown`, `close_sockets`, `in_loop`, `close_sockets_and_tasks`, `socket`。 它会读写 `engine_dead` 等状态。 控制流包含 9 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `BackgroundResources.validate_alive` method / `BackgroundResources.validate_alive` 方法
```python
    def validate_alive(self, frames: Sequence[zmq.Frame]):
        if len(frames) == 1 and (frames[0].buffer == EngineCoreProc.ENGINE_CORE_DEAD):
            self.engine_dead = True
            raise EngineDeadError()
```
**EN:** This method validates assumptions or constraints within `BackgroundResources`. Key calls include `EngineDeadError`, `len`. It touches state such as `engine_dead`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会校验前提与约束，其作用域位于`BackgroundResources`。 关键调用包括 `EngineDeadError`, `len`。 它会读写 `engine_dead` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `ElasticScalingCache` class / `ElasticScalingCache` 类
```python
@dataclass
class ElasticScalingCache:
    existing_core_engines: list[EngineIdentity]
    num_new_core_engines: int
    pending_notifications: dict[EEPNotificationType, set[int]]
```
**EN:** Uses `@dataclass` to package related state for `ElasticScalingCache`. Typical fields include `existing_core_engines`, `num_new_core_engines`, `pending_notifications`.
**CN:** `ElasticScalingCache` 使用 `@dataclass` 打包相关状态。典型字段包括 `existing_core_engines`, `num_new_core_engines`, `pending_notifications`。

### `MPClient` class / `MPClient` 类
```python
class MPClient(EngineCoreClient):
    """
    MPClient: base client for multi-proc EngineCore.
        EngineCore runs in a background process busy loop, getting
        new EngineCoreRequests and returning EngineCoreOutputs

        * pushes EngineCoreRequests via input_socket
        * pulls EngineCoreOutputs via output_socket

        * AsyncMPClient subclass for AsyncLLM usage
        * SyncMPClient subclass for LLM usage
    """
```
**EN:** Introduces the `MPClient` class on top of `EngineCoreClient`. Core methods include `__init__`, `shutdown`, `_format_exception`, `ensure_alive`, `add_pending_message`, `free_pending_messages`. Docstring signal: MPClient: base client for multi-proc EngineCore.
**CN:** 这里定义 `MPClient` 类，其基类包括 `EngineCoreClient`。核心方法包括 `__init__`, `shutdown`, `_format_exception`, `ensure_alive`, `add_pending_message`, `free_pending_messages`。

### `MPClient.__init__` method / `MPClient.__init__` 方法
```python
    def __init__(
        self,
        asyncio_mode: bool,
        vllm_config: VllmConfig,
        executor_class: type[Executor],
        log_stats: bool,
        client_addresses: dict[str, str] | None = None,
    ):
        self.vllm_config = vllm_config

        # ZMQ setup.
        sync_ctx = zmq.Context(io_threads=2)
        self.ctx = zmq.asyncio.Context(sync_ctx) if asyncio_mode else sync_ctx

        # This will ensure resources created so far are closed
        # when the client is garbage collected, even if an
        # exception is raised mid-construction.
        self.resources = BackgroundResources(ctx=sync_ctx)
        self._finalizer = weakref.finalize(self, self.resources)
        success = False
        try:
            # State used for data parallel.
            self.engines_running = False
            parallel_config = vllm_config.parallel_config
            # Elastic EP can remove a rank and later add it back with the same
            # identity. The client input ROUTER needs handover to allow the new
            # engine to replace the dead connection.
            enable_input_socket_handover = parallel_config.enable_elastic_ep

            self.stats_update_address: str | None = None
            tensor_queue: Queue | None = None
            if client_addresses:
                # Engines are managed externally to this client.
                input_address = client_addresses["input_address"]
                output_address = client_addresses["output_address"]
                self.stats_update_address = client_addresses.get("stats_update_address")
                # Tensor queues passed via client_addresses for multi-API-server case
                tensor_queue = client_addresses.get("tensor_queue")  # type: ignore[assignment]
                self.input_socket = self.resources.input_socket = make_zmq_socket(
                    self.ctx,
                    input_address,
                    zmq.ROUTER,
                    bind=True,
                    router_handover=enable_input_socket_handover,
                )
                self.resources.output_socket = make_zmq_socket(
                    self.ctx, output_address, zmq.PULL
                )
            else:
                # Engines are managed by this client.
    # ... omitted for brevity ...
                        f"for large models. Waited "
                        f"{VLLM_ENGINE_READY_TIMEOUT_S}s (configured by "
                        f"VLLM_ENGINE_READY_TIMEOUT_S). To increase the "
                        f"timeout, set the environment variable: "
                        f"VLLM_ENGINE_READY_TIMEOUT_S=<seconds>"
                    )
                identity, payload = sync_input_socket.recv_multipart()
                identities.remove(identity)
                self._apply_ready_response(payload)

            self.core_engine: EngineIdentity = self.core_engines[0]
            self.utility_results: dict[int, AnyFuture] = {}

            # Request objects which may contain pytorch-allocated tensors
            # that we need to keep references to until zmq is done with the
            # underlying data.
            self.pending_messages = deque[tuple[zmq.MessageTracker, Any]]()

            # Start monitoring engine core processes for unexpected failures
            self.start_engine_core_monitor()

            success = True
        finally:
            if not success:
                self._finalizer()
```
**EN:** This method initializes the object state within `MPClient`. Key calls include `Context`, `BackgroundResources`, `finalize`, `getattr`, `MsgpackEncoder`, `MsgpackDecoder`. It touches state such as `vllm_config`, `ctx`, `resources`, `_finalizer`, `engines_running`, `stats_update_address`, `encoder`, `decoder`. The control flow contains 10 branch(es) and 1 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会初始化对象状态，其作用域位于`MPClient`。 关键调用包括 `Context`, `BackgroundResources`, `finalize`, `getattr`, `MsgpackEncoder`, `MsgpackDecoder`。 它会读写 `vllm_config`, `ctx`, `resources`, `_finalizer`, `engines_running`, `stats_update_address`, `encoder`, `decoder` 等状态。 控制流包含 10 个分支和 1 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `MPClient.free_pending_messages` method / `MPClient.free_pending_messages` 方法
```python
    def free_pending_messages(self):
        while self.pending_messages and self.pending_messages[-1][0].done:
            self.pending_messages.pop()
```
**EN:** This method implements `free_pending_messages` within `MPClient`. Key calls include `pop`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `free_pending_messages`，其作用域位于`MPClient`。 关键调用包括 `pop`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `MPClient.dp_engines_running` method / `MPClient.dp_engines_running` 方法
```python
    def dp_engines_running(self) -> bool:
        return self.engines_running
```
**EN:** This method implements `dp_engines_running` within `MPClient`.
**CN:** 该方法会实现 `dp_engines_running`，其作用域位于`MPClient`。

### `MPClient.start_engine_core_monitor` method / `MPClient.start_engine_core_monitor` 方法
```python
    def start_engine_core_monitor(self):
        """Start a monitor thread for engine core processes."""
        engine_manager = self.resources.engine_manager
        if engine_manager is None:
            # No engine processes to monitor
            return

        self_ref = weakref.ref(self)

        # Monitor engine core process liveness. If any die unexpectedly,
        # marks the engine as dead, and shuts down the client.
        def monitor_engine_cores():
            engine_manager.monitor_engine_liveness()
            _self = self_ref()
            if not _self or not _self._finalizer.alive or _self.resources.engine_dead:
                return
            _self.resources.engine_dead = True
            _self.shutdown()
            # Note: For MPClient, we don't have a failure callback mechanism
            # like MultiprocExecutor, but we set engine_dead flag which will
            # cause subsequent operations to raise EngineDeadError

        Thread(
            target=monitor_engine_cores, daemon=True, name="MPClientEngineMonitor"
        ).start()
```
**EN:** This method implements `start_engine_core_monitor` within `MPClient`. The docstring frames it as: Start a monitor thread for engine core processes. Key calls include `ref`, `start`, `monitor_engine_liveness`, `self_ref`, `shutdown`, `Thread`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `start_engine_core_monitor`，其作用域位于`MPClient`。 关键调用包括 `ref`, `start`, `monitor_engine_liveness`, `self_ref`, `shutdown`, `Thread`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_process_utility_output` function / `_process_utility_output` 函数
```python
def _process_utility_output(
    output: UtilityOutput, utility_results: dict[int, AnyFuture]
):
    """Set the result from a utility method in the waiting future."""
    future = utility_results.pop(output.call_id)
    failure_message = output.failure_message
    try:
        if failure_message is not None:
            future.set_exception(Exception(failure_message))
        else:
            assert output.result is not None
            future.set_result(output.result.result)
    except asyncio.InvalidStateError:
        # This can happen if the future is cancelled due to the
        # original calling task being cancelled.
        if failure_message is not None:
            logger.error(
                "Cancelled call to utility method failed with error: %s",
                failure_message,
            )
```
**EN:** This function implements `_process_utility_output` within the module. The docstring frames it as: Set the result from a utility method in the waiting future. Key calls include `pop`, `set_exception`, `set_result`, `Exception`, `error`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_process_utility_output`，其作用域位于the module。 关键调用包括 `pop`, `set_exception`, `set_result`, `Exception`, `error`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `SyncMPClient` class / `SyncMPClient` 类
```python
class SyncMPClient(MPClient):
    """Synchronous client for multi-proc EngineCore."""
```
**EN:** Introduces the `SyncMPClient` class on top of `MPClient`. Core methods include `__init__`, `get_output`, `_send_input`, `call_utility`, `get_supported_tasks`, `add_request`. Docstring signal: Synchronous client for multi-proc EngineCore.
**CN:** 这里定义 `SyncMPClient` 类，其基类包括 `MPClient`。核心方法包括 `__init__`, `get_output`, `_send_input`, `call_utility`, `get_supported_tasks`, `add_request`。

### `SyncMPClient.__init__` method / `SyncMPClient.__init__` 方法
```python
    @instrument(span_name="SyncMPClient init")
    def __init__(
        self, vllm_config: VllmConfig, executor_class: type[Executor], log_stats: bool
    ):
        super().__init__(
            asyncio_mode=False,
            vllm_config=vllm_config,
            executor_class=executor_class,
            log_stats=log_stats,
        )

        self.is_dp = self.vllm_config.parallel_config.data_parallel_size > 1
        self.outputs_queue = queue.Queue[EngineCoreOutputs | Exception]()

        # Ensure that the outputs socket processing thread does not have
        # a ref to the client which prevents gc.
        ctx = self.ctx
        out_socket = self.resources.output_socket
        decoder = self.decoder
        utility_results = self.utility_results
        outputs_queue = self.outputs_queue

        shutdown_path = get_open_zmq_inproc_path()
        resources = self.resources
        resources.shutdown_path = shutdown_path

        def process_outputs_socket():
            assert isinstance(out_socket, zmq.Socket)
            shutdown_socket = ctx.socket(zmq.PAIR)
            try:
                shutdown_socket.bind(shutdown_path)
                poller = zmq.Poller()
                poller.register(shutdown_socket, zmq.POLLIN)
                poller.register(out_socket, zmq.POLLIN)
                while True:
                    socks = poller.poll()
                    if not socks:
                        continue
                    if len(socks) == 2 or socks[0][0] == shutdown_socket:
                        # shutdown signal, exit thread.
                        break

                    frames = out_socket.recv_multipart(copy=False)
                    resources.validate_alive(frames)
                    outputs: EngineCoreOutputs = decoder.decode(frames)
                    if outputs.utility_output:
                        _process_utility_output(outputs.utility_output, utility_results)
                    else:
                        outputs_queue.put_nowait(outputs)
            except Exception as e:
                outputs_queue.put_nowait(e)
            finally:
                # Close sockets.
                shutdown_socket.close(linger=0)
                out_socket.close(linger=0)

        # Process outputs from engine in separate thread.
        self.output_queue_thread = Thread(
            target=process_outputs_socket,
            name="EngineCoreOutputQueueThread",
            daemon=True,
        )
        self.output_queue_thread.start()

        # The thread takes on responsibility for closing the socket.
        self.resources.output_socket = None
```
**EN:** This method initializes the object state within `SyncMPClient`. Key calls include `instrument`, `__init__`, `get_open_zmq_inproc_path`, `Thread`, `start`, `isinstance`. It touches state such as `is_dp`, `outputs_queue`, `output_queue_thread`, `resources`. The control flow contains 4 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`SyncMPClient`。 关键调用包括 `instrument`, `__init__`, `get_open_zmq_inproc_path`, `Thread`, `start`, `isinstance`。 它会读写 `is_dp`, `outputs_queue`, `output_queue_thread`, `resources` 等状态。 控制流包含 4 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `SyncMPClient.execute_dummy_batch` method / `SyncMPClient.execute_dummy_batch` 方法
```python
    def execute_dummy_batch(self) -> None:
        self.call_utility("execute_dummy_batch")
```
**EN:** This method executes the main operation within `SyncMPClient`. Key calls include `call_utility`.
**CN:** 该方法会执行主要操作，其作用域位于`SyncMPClient`。 关键调用包括 `call_utility`。

### `SyncMPClient.collective_rpc` method / `SyncMPClient.collective_rpc` 方法
```python
    def collective_rpc(
        self,
        method: str | Callable[..., _R],
        timeout: float | None = None,
        args: tuple = (),
        kwargs: dict[str, Any] | None = None,
    ) -> list[_R]:
        return self.call_utility("collective_rpc", method, timeout, args, kwargs)
```
**EN:** This method collects data from collaborators within `SyncMPClient`. Key calls include `call_utility`.
**CN:** 该方法会从协作组件收集数据，其作用域位于`SyncMPClient`。 关键调用包括 `call_utility`。

### `SyncMPClient.save_sharded_state` method / `SyncMPClient.save_sharded_state` 方法
```python
    def save_sharded_state(
        self, path: str, pattern: str | None = None, max_size: int | None = None
    ) -> None:
        self.call_utility("save_sharded_state", path, pattern, max_size)
```
**EN:** This method persists computed state within `SyncMPClient`. Key calls include `call_utility`.
**CN:** 该方法会持久化计算结果，其作用域位于`SyncMPClient`。 关键调用包括 `call_utility`。

### `AsyncMPClient` class / `AsyncMPClient` 类
```python
class AsyncMPClient(MPClient):
    """Asyncio-compatible client for multi-proc EngineCore."""
```
**EN:** Introduces the `AsyncMPClient` class on top of `MPClient`. Core methods include `__init__`, `_ensure_output_queue_task`, `get_output_async`, `_send_input`, `_send_input_message`, `call_utility_async`. Docstring signal: Asyncio-compatible client for multi-proc EngineCore.
**CN:** 这里定义 `AsyncMPClient` 类，其基类包括 `MPClient`。核心方法包括 `__init__`, `_ensure_output_queue_task`, `get_output_async`, `_send_input`, `_send_input_message`, `call_utility_async`。

### `AsyncMPClient.__init__` method / `AsyncMPClient.__init__` 方法
```python
    @instrument(span_name="AsyncMPClient init")
    def __init__(
        self,
        vllm_config: VllmConfig,
        executor_class: type[Executor],
        log_stats: bool,
        client_addresses: dict[str, str] | None = None,
        client_count: int = 1,
        client_index: int = 0,
    ):
        super().__init__(
            asyncio_mode=True,
            vllm_config=vllm_config,
            executor_class=executor_class,
            log_stats=log_stats,
            client_addresses=client_addresses,
        )

        self.client_count = client_count
        self.client_index = client_index
        self.outputs_queue = asyncio.Queue[EngineCoreOutputs | Exception]()
        try:
            # If we are running in an asyncio event loop, start the queue task.
            # Otherwise, it will be started lazily. If it is not started here,
            # we could miss EXECUTOR_FAILED messages from engine core if they
            # occur prior to any requests being sent.
            asyncio.get_running_loop()
            self._ensure_output_queue_task()
        except RuntimeError:
            pass
```
**EN:** This method initializes the object state within `AsyncMPClient`. Key calls include `instrument`, `__init__`, `get_running_loop`, `_ensure_output_queue_task`, `super`. It touches state such as `client_count`, `client_index`, `outputs_queue`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`AsyncMPClient`。 关键调用包括 `instrument`, `__init__`, `get_running_loop`, `_ensure_output_queue_task`, `super`。 它会读写 `client_count`, `client_index`, `outputs_queue` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AsyncMPClient.pause_scheduler_async` method / `AsyncMPClient.pause_scheduler_async` 方法
```python
    async def pause_scheduler_async(
        self, mode: PauseMode = "abort", clear_cache: bool = True
    ) -> None:
        await self.call_utility_async("pause_scheduler", mode, clear_cache)
```
**EN:** This method implements `pause_scheduler_async` within `AsyncMPClient`. Key calls include `call_utility_async`.
**CN:** 该方法会实现 `pause_scheduler_async`，其作用域位于`AsyncMPClient`。 关键调用包括 `call_utility_async`。

### `AsyncMPClient.resume_scheduler_async` method / `AsyncMPClient.resume_scheduler_async` 方法
```python
    async def resume_scheduler_async(self) -> None:
        await self.call_utility_async("resume_scheduler")
```
**EN:** This method implements `resume_scheduler_async` within `AsyncMPClient`. Key calls include `call_utility_async`.
**CN:** 该方法会实现 `resume_scheduler_async`，其作用域位于`AsyncMPClient`。 关键调用包括 `call_utility_async`。

### `AsyncMPClient.is_scheduler_paused_async` method / `AsyncMPClient.is_scheduler_paused_async` 方法
```python
    async def is_scheduler_paused_async(self) -> bool:
        return await self.call_utility_async("is_scheduler_paused")
```
**EN:** This method answers a boolean capability check within `AsyncMPClient`. Key calls include `call_utility_async`.
**CN:** 该方法会回答布尔能力判断，其作用域位于`AsyncMPClient`。 关键调用包括 `call_utility_async`。

### `DPAsyncMPClient` class / `DPAsyncMPClient` 类
```python
class DPAsyncMPClient(AsyncMPClient):
    """Asyncio-compatible client for multi-proc, multi-engine (data parallel)
    EngineCore. Assumes external load-balancing by default."""
```
**EN:** Introduces the `DPAsyncMPClient` class on top of `AsyncMPClient`. Core methods include `__init__`, `_ensure_stats_update_task`, `add_request_async`, `get_core_engine_for_request`. Docstring signal: Asyncio-compatible client for multi-proc, multi-engine (data parallel) EngineCore.
**CN:** 这里定义 `DPAsyncMPClient` 类，其基类包括 `AsyncMPClient`。核心方法包括 `__init__`, `_ensure_stats_update_task`, `add_request_async`, `get_core_engine_for_request`。

### `DPAsyncMPClient.__init__` method / `DPAsyncMPClient.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        executor_class: type[Executor],
        log_stats: bool,
        client_addresses: dict[str, str] | None = None,
        client_count: int = 1,
        client_index: int = 0,
    ):
        self.current_wave = 0

        super().__init__(
            vllm_config,
            executor_class,
            log_stats,
            client_addresses,
            client_count,
            client_index,
        )

        # List of [waiting, running] pair per engine.
        # Used only by DPLBAsyncMPClient subclass.
        self.lb_engines: list[list[int]] = [[0, 0] for _ in self.core_engines]

        self.eep_scaling_cache: ElasticScalingCache | None = None

        self.first_req_sock_addr = get_open_zmq_inproc_path()
        self.first_req_send_socket = self.resources.first_req_send_socket = (
            make_zmq_socket(self.ctx, self.first_req_sock_addr, zmq.PAIR, bind=True)
        )
        try:
            # If we are running in an asyncio event loop, start the stats task.
            # Otherwise, it will be started lazily.
            asyncio.get_running_loop()
            self._ensure_stats_update_task()
        except RuntimeError:
            pass
```
**EN:** This method initializes the object state within `DPAsyncMPClient`. Key calls include `__init__`, `get_open_zmq_inproc_path`, `make_zmq_socket`, `get_running_loop`, `_ensure_stats_update_task`, `super`. It touches state such as `current_wave`, `lb_engines`, `eep_scaling_cache`, `first_req_sock_addr`, `first_req_send_socket`, `resources`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`DPAsyncMPClient`。 关键调用包括 `__init__`, `get_open_zmq_inproc_path`, `make_zmq_socket`, `get_running_loop`, `_ensure_stats_update_task`, `super`。 它会读写 `current_wave`, `lb_engines`, `eep_scaling_cache`, `first_req_sock_addr`, `first_req_send_socket`, `resources` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DPAsyncMPClient._ensure_stats_update_task` method / `DPAsyncMPClient._ensure_stats_update_task` 方法
```python
    def _ensure_stats_update_task(self):
        resources = self.resources
        if resources.stats_update_task is not None:
            return

        assert self.stats_update_address is not None
        stats_addr: str = self.stats_update_address
        assert len(self.engine_ranks_managed) > 0

        async def run_engine_stats_update_task():
            with (
                make_zmq_socket(self.ctx, stats_addr, zmq.XSUB, linger=0) as socket,
                make_zmq_socket(
                    self.ctx, self.first_req_sock_addr, zmq.PAIR, bind=False, linger=0
                ) as first_req_rcv_socket,
            ):
                assert isinstance(socket, zmq.asyncio.Socket)
                assert isinstance(first_req_rcv_socket, zmq.asyncio.Socket)
                self.resources.stats_update_socket = socket
                self.resources.first_req_rcv_socket = first_req_rcv_socket
                # Send subscription message.
                await socket.send(b"\x01")

                poller = zmq.asyncio.Poller()
                poller.register(socket, zmq.POLLIN)
                poller.register(first_req_rcv_socket, zmq.POLLIN)

                while True:
                    events = await poller.poll()
                    if (
                        not self.engines_running
                        and len(events) == 2
                        or (events[0][0] == first_req_rcv_socket)
                    ):
                        # Check if this is a regular request notification or
                        # scale up notification
                        buf = first_req_rcv_socket.recv(flags=zmq.NOBLOCK).result()

                        decoded = msgspec.msgpack.decode(buf)
                        if (
                            isinstance(decoded, (list, tuple))
                            and len(decoded) == 2
                            and decoded[0] == "SCALE_ELASTIC_EP"
                        ):
                            # Extract new engine count from the decoded message
                            new_engine_count = decoded[1]
                            # Update engine_ranks_managed and count_slice
                            parallel_config = self.vllm_config.parallel_config
                            dp_size = parallel_config.data_parallel_size
                            dp_rank = parallel_config.data_parallel_rank
    # ... omitted for brevity ...
                        if isinstance(future.exception(), zmq.Again):
                            break
                        buf = future.result()
                    if buf is None:
                        continue

                    # Update local load-balancing state.
                    counts, wave, running = msgspec.msgpack.decode(buf)
                    self.current_wave = wave
                    self.engines_running = running
                    if counts is not None:
                        # Running and waiting counts are global from the
                        # Coordinator including all EngineCores. Slice to get
                        # just the cores managed by this client.
                        ranks = self.engine_ranks_managed
                        count_slice = slice(ranks[0], ranks[-1] + 1)
                        sliced_counts = counts[count_slice]
                        self.lb_engines = sliced_counts
                        logger.debug(
                            "Received counts: %s (%s)", sliced_counts, count_slice
                        )

        resources.stats_update_task = asyncio.create_task(
            run_engine_stats_update_task()
        )
```
**EN:** This method implements `_ensure_stats_update_task` within `DPAsyncMPClient`. Key calls include `create_task`, `len`, `run_engine_stats_update_task`, `make_zmq_socket`, `isinstance`, `Poller`. It touches state such as `resources`, `current_wave`, `engines_running`, `lb_engines`, `engine_ranks_managed`. The control flow contains 7 branch(es) and 2 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `_ensure_stats_update_task`，其作用域位于`DPAsyncMPClient`。 关键调用包括 `create_task`, `len`, `run_engine_stats_update_task`, `make_zmq_socket`, `isinstance`, `Poller`。 它会读写 `resources`, `current_wave`, `engines_running`, `lb_engines`, `engine_ranks_managed` 等状态。 控制流包含 7 个分支和 2 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `DPAsyncMPClient.add_request_async` method / `DPAsyncMPClient.add_request_async` 方法
```python
    async def add_request_async(self, request: EngineCoreRequest) -> None:
        self._ensure_stats_update_task()

        request.current_wave = self.current_wave
        request.client_index = self.client_index

        chosen_engine = self.get_core_engine_for_request(request)
        to_await = self._send_input(EngineCoreRequestType.ADD, request, chosen_engine)
        if not self.engines_running:
            # Notify coordinator that we're sending a request
            req_msg = msgspec.msgpack.encode(("FIRST_REQ", chosen_engine))
            await self.first_req_send_socket.send(req_msg)

        await to_await

        self._ensure_output_queue_task()
```
**EN:** This method implements `add_request_async` within `DPAsyncMPClient`. Key calls include `_ensure_stats_update_task`, `get_core_engine_for_request`, `_send_input`, `_ensure_output_queue_task`, `encode`, `send`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `add_request_async`，其作用域位于`DPAsyncMPClient`。 关键调用包括 `_ensure_stats_update_task`, `get_core_engine_for_request`, `_send_input`, `_ensure_output_queue_task`, `encode`, `send`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DPAsyncMPClient.get_core_engine_for_request` method / `DPAsyncMPClient.get_core_engine_for_request` 方法
```python
    def get_core_engine_for_request(self, request: EngineCoreRequest):
        return self.core_engine
```
**EN:** This method returns or derives a value within `DPAsyncMPClient`.
**CN:** 该方法会返回或推导一个值，其作用域位于`DPAsyncMPClient`。

### `DPLBAsyncMPClient` class / `DPLBAsyncMPClient` 类
```python
class DPLBAsyncMPClient(DPAsyncMPClient):
    """Asyncio-compatible client for multi-proc, multi-engine (data parallel)
    EngineCore. Load-balances between multiple engine processes."""
```
**EN:** Introduces the `DPLBAsyncMPClient` class on top of `DPAsyncMPClient`. Core methods include `__init__`, `get_core_engine_for_request`, `call_utility_async`, `process_engine_outputs`, `eep_process_engine_core_notification`, `abort_requests_async`. Docstring signal: Asyncio-compatible client for multi-proc, multi-engine (data parallel) EngineCore.
**CN:** 这里定义 `DPLBAsyncMPClient` 类，其基类包括 `DPAsyncMPClient`。核心方法包括 `__init__`, `get_core_engine_for_request`, `call_utility_async`, `process_engine_outputs`, `eep_process_engine_core_notification`, `abort_requests_async`。

### `DPLBAsyncMPClient.__init__` method / `DPLBAsyncMPClient.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        executor_class: type[Executor],
        log_stats: bool,
        client_addresses: dict[str, str] | None = None,
        client_count: int = 1,
        client_index: int = 0,
    ):
        self.client_count = client_count

        # To route aborts to the correct engine.
        self.reqs_in_flight: dict[str, EngineIdentity] = {}

        super().__init__(
            vllm_config,
            executor_class,
            log_stats,
            client_addresses,
            client_count,
            client_index,
        )

        assert len(self.core_engines) > 1

        self.eng_start_index = (
            len(self.core_engines) * self.client_index
        ) // client_count
```
**EN:** This method initializes the object state within `DPLBAsyncMPClient`. Key calls include `__init__`, `len`, `super`. It touches state such as `client_count`, `reqs_in_flight`, `eng_start_index`.
**CN:** 该方法会初始化对象状态，其作用域位于`DPLBAsyncMPClient`。 关键调用包括 `__init__`, `len`, `super`。 它会读写 `client_count`, `reqs_in_flight`, `eng_start_index` 等状态。

### `DPLBAsyncMPClient.eep_process_engine_core_notification` method / `DPLBAsyncMPClient.eep_process_engine_core_notification` 方法
```python
    @staticmethod
    async def eep_process_engine_core_notification(
        self: "DPLBAsyncMPClient", notification_data: tuple[str, int]
    ):
        cache = self.eep_scaling_cache
        notification_type_str, dp_rank = notification_data
        try:
            notification_type = EEPNotificationType(notification_type_str)
        except ValueError as e:
            raise ValueError(
                f"Unknown EEP notification type: {notification_type_str}"
            ) from e

        if notification_type == EEPNotificationType.RECONFIGURE_FINISHED:
            from vllm.v1.engine import UtilityResult

            # NOTE(yongji): process a dummy UtilityOutput to resolve the future
            # awaited in _eep_wait_for_setup_switch_complete(), signaling that
            # all engine cores have completed reconfiguration.
            dummy_output = UtilityOutput(
                call_id=EEP_NOTIFICATION_CALL_ID, result=UtilityResult(None)
            )
            _process_utility_output(dummy_output, self.utility_results)
            return
        assert cache is not None
        if notification_type not in cache.pending_notifications:
            cache.pending_notifications[notification_type] = set()
        if dp_rank in cache.pending_notifications[notification_type]:
            raise ValueError(
                f"Duplicate notification {notification_type} from dp_rank {dp_rank}"
            )
        cache.pending_notifications[notification_type].add(dp_rank)
        if len(cache.pending_notifications[notification_type]) >= abs(
            cache.num_new_core_engines
        ):
            if notification_type == EEPNotificationType.SHUTDOWN_COMPLETE:
                assert isinstance(self.resources.engine_manager, CoreEngineActorManager)
                assert cache.num_new_core_engines < 0
                old_dp_size = len(cache.existing_core_engines)
                new_dp_size = old_dp_size + cache.num_new_core_engines
                self.resources.engine_manager.scale_down_elastic_ep(
                    old_dp_size, new_dp_size
                )
            else:
                await asyncio.gather(
                    *[
                        self._call_utility_async(
                            "eep_handle_engine_core_notification",
                            notification_type,
                            engine=engine,
                        )
                        for engine in cache.existing_core_engines
                    ]
                )
            cache.pending_notifications[notification_type] = set()
            if notification_type in [
                EEPNotificationType.SHUTDOWN_COMPLETE,
                EEPNotificationType.NEW_CORE_ENGINES_WEIGHTS_INIT_READY,
            ]:
                self.eep_scaling_cache = None
```
**EN:** This method implements `eep_process_engine_core_notification` within `DPLBAsyncMPClient`. Key calls include `add`, `EEPNotificationType`, `UtilityOutput`, `_process_utility_output`, `set`, `ValueError`. It touches state such as `eep_scaling_cache`. The control flow contains 7 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `eep_process_engine_core_notification`，其作用域位于`DPLBAsyncMPClient`。 关键调用包括 `add`, `EEPNotificationType`, `UtilityOutput`, `_process_utility_output`, `set`, `ValueError`。 它会读写 `eep_scaling_cache` 等状态。 控制流包含 7 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DPLBAsyncMPClient._scale_up_elastic_ep` method / `DPLBAsyncMPClient._scale_up_elastic_ep` 方法
```python
    async def _scale_up_elastic_ep(
        self, cur_data_parallel_size: int, new_data_parallel_size: int
    ) -> None:
        """Scale up the data parallel size by creating new engine cores
        and reconfiguring existing ones."""
        cur_data_parallel_size = len(self.core_engines)

        self.eep_scaling_cache = ElasticScalingCache(
            existing_core_engines=self.core_engines.copy(),
            num_new_core_engines=new_data_parallel_size - cur_data_parallel_size,
            pending_notifications=dict(),
        )

        parallel_config = self.vllm_config.parallel_config
        ip, coord_store_port = self._setup_elastic_ep_reconfig_bootstrap()

        # Phase 1: Send reconfig messages to existing engines
        reconfig_futures = []
        for engine in self.core_engines:
            reconfig_request = ReconfigureDistributedRequest(
                new_data_parallel_size=new_data_parallel_size,
                new_data_parallel_rank=ReconfigureRankType.KEEP_CURRENT_RANK,
                new_data_parallel_rank_local=ReconfigureRankType.KEEP_CURRENT_RANK,
                new_data_parallel_master_ip=ip,
                new_data_parallel_master_port=parallel_config.data_parallel_master_port,
                new_data_parallel_master_port_list=parallel_config._data_parallel_master_port_list,
                coord_store_port=coord_store_port,
            )
            coro = self._call_utility_async(
                "reinitialize_distributed", reconfig_request, engine=engine
            )
            reconfig_futures.append(asyncio.create_task(coro))

        # Phase 2: Create new engines
        assert isinstance(self.resources.engine_manager, CoreEngineActorManager)
        parallel_config.eplb_config.num_redundant_experts = 0
        start_new_worker_future = asyncio.to_thread(
            self.resources.engine_manager.scale_up_elastic_ep,
            self.vllm_config,
            new_data_parallel_size,
        )
        wait_future = self._eep_wait_for_setup_switch_complete()

        # Phase 3: Wait for new engines to be created
        # and reconfig messages to be received
        await asyncio.gather(start_new_worker_future, *reconfig_futures)
        logger.info("[Elastic EP] Successfully started new engines")

        # Create new CoreEngine objects for the new engines
        new_engine_identities = set()
    # ... omitted for brevity ...
                    f"VLLM_ENGINE_READY_TIMEOUT_S). To increase the "
                    f"timeout, set the environment variable: "
                    f"VLLM_ENGINE_READY_TIMEOUT_S=<seconds>"
                )
            identity, payload = sync_input_socket.recv_multipart()
            new_engine_identities.discard(identity)
            self._apply_ready_response(payload)

        # NOTE(yongji): Before we schedule any requests on the new workers,
        # we should wait for them to switch to the new setup.
        await wait_future
        # Update the parallel config
        self.vllm_config.parallel_config.data_parallel_size = new_data_parallel_size
        # Notify coordinator about scale up through existing
        # stats_update_task connection
        self._ensure_stats_update_task()
        scale_up_marker = msgspec.msgpack.encode(
            ("SCALE_ELASTIC_EP", new_data_parallel_size)
        )
        await self.first_req_send_socket.send(scale_up_marker)

        logger.info(
            "[Elastic EP] Scale up completed, new data parallel size: %s",
            new_data_parallel_size,
        )
```
**EN:** This method implements `_scale_up_elastic_ep` within `DPLBAsyncMPClient`. The docstring frames it as: Scale up the data parallel size by creating new engine cores and reconfiguring existing ones. Key calls include `len`, `ElasticScalingCache`, `_setup_elastic_ep_reconfig_bootstrap`, `isinstance`, `to_thread`, `_eep_wait_for_setup_switch_complete`. It touches state such as `eep_scaling_cache`, `vllm_config`. The control flow contains 1 branch(es) and 3 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `_scale_up_elastic_ep`，其作用域位于`DPLBAsyncMPClient`。 关键调用包括 `len`, `ElasticScalingCache`, `_setup_elastic_ep_reconfig_bootstrap`, `isinstance`, `to_thread`, `_eep_wait_for_setup_switch_complete`。 它会读写 `eep_scaling_cache`, `vllm_config` 等状态。 控制流包含 1 个分支和 3 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `DPLBAsyncMPClient._scale_down_elastic_ep` method / `DPLBAsyncMPClient._scale_down_elastic_ep` 方法
```python
    async def _scale_down_elastic_ep(
        self, cur_data_parallel_size: int, new_data_parallel_size: int
    ) -> None:
        """Scale down the data parallel size by shutting down and
        reconfiguring existing engine cores."""
        cur_data_parallel_size = len(self.core_engines)

        self.eep_scaling_cache = ElasticScalingCache(
            existing_core_engines=self.core_engines.copy(),
            num_new_core_engines=new_data_parallel_size - cur_data_parallel_size,
            pending_notifications=dict(),
        )

        parallel_config = self.vllm_config.parallel_config
        ip, coord_store_port = self._setup_elastic_ep_reconfig_bootstrap()

        removed_dp_size = cur_data_parallel_size - new_data_parallel_size
        assert isinstance(self.resources.engine_manager, CoreEngineActorManager)
        self.resources.engine_manager.remove_run_refs_for_scale_down(removed_dp_size)
        reconfig_futures = []
        for cur_dp_rank, engine in enumerate(self.core_engines):
            reconfig_request = ReconfigureDistributedRequest(
                new_data_parallel_size=new_data_parallel_size,
                new_data_parallel_rank=ReconfigureRankType.KEEP_CURRENT_RANK,
                new_data_parallel_rank_local=ReconfigureRankType.KEEP_CURRENT_RANK,
                new_data_parallel_master_ip=ip,
                new_data_parallel_master_port=parallel_config.data_parallel_master_port,
                new_data_parallel_master_port_list=parallel_config._data_parallel_master_port_list,
                coord_store_port=coord_store_port,
            )
            if cur_dp_rank >= new_data_parallel_size:
                reconfig_request.new_data_parallel_rank = (
                    ReconfigureRankType.SHUTDOWN_CURRENT_RANK
                )
            coro = self._call_utility_async(
                "reinitialize_distributed", reconfig_request, engine=engine
            )
            reconfig_futures.append(asyncio.create_task(coro))

        # NOTE(yongji): Immediately stop sending requests to the removing engines.
        self.core_engines = self.core_engines[:new_data_parallel_size]
        self.lb_engines = self.lb_engines[:new_data_parallel_size]
        wait_future = self._eep_wait_for_setup_switch_complete()

        await asyncio.gather(*reconfig_futures)

        self.vllm_config.parallel_config.data_parallel_size = new_data_parallel_size
        self._ensure_stats_update_task()
        scale_down_marker = msgspec.msgpack.encode(
            ("SCALE_ELASTIC_EP", new_data_parallel_size)
        )
        await self.first_req_send_socket.send(scale_down_marker)

        # NOTE(yongji): Unlike scaling up,
        # here we don't actually need to wait for the setup switch to complete.
        # We may want to remove it in the future.
        await wait_future
        logger.info(
            "[Elastic EP] Scale down completed, new data parallel size: %s",
            new_data_parallel_size,
        )
```
**EN:** This method implements `_scale_down_elastic_ep` within `DPLBAsyncMPClient`. The docstring frames it as: Scale down the data parallel size by shutting down and reconfiguring existing engine cores. Key calls include `len`, `ElasticScalingCache`, `_setup_elastic_ep_reconfig_bootstrap`, `isinstance`, `remove_run_refs_for_scale_down`, `enumerate`. It touches state such as `eep_scaling_cache`, `core_engines`, `lb_engines`, `vllm_config`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_scale_down_elastic_ep`，其作用域位于`DPLBAsyncMPClient`。 关键调用包括 `len`, `ElasticScalingCache`, `_setup_elastic_ep_reconfig_bootstrap`, `isinstance`, `remove_run_refs_for_scale_down`, `enumerate`。 它会读写 `eep_scaling_cache`, `core_engines`, `lb_engines`, `vllm_config` 等状态。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `EngineCoreClient`: central class or interface in this module. / `EngineCoreClient`：本模块中的核心类或接口。
- `InprocClient`: central class or interface in this module. / `InprocClient`：本模块中的核心类或接口。
- `BackgroundResources`: central class or interface in this module. / `BackgroundResources`：本模块中的核心类或接口。
- `ElasticScalingCache`: central class or interface in this module. / `ElasticScalingCache`：本模块中的核心类或接口。
- `MPClient`: central class or interface in this module. / `MPClient`：本模块中的核心类或接口。
- `_process_utility_output`: top-level helper or orchestration entry point. / `_process_utility_output`：顶层辅助函数或编排入口。
- `SyncMPClient`: central class or interface in this module. / `SyncMPClient`：本模块中的核心类或接口。
- `AsyncMPClient`: central class or interface in this module. / `AsyncMPClient`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `asyncio`, `contextlib`, `queue`, `sys`, `uuid`, `weakref`, `abc`, `collections`, `concurrent`, `dataclasses`, `multiprocessing`, `threading`
- External / 外部依赖: `msgspec`, `zmq`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.envs`, `vllm.logger`, `vllm.lora.request`, `vllm.tasks`, `vllm.tracing`, `vllm.utils.async_utils`, `vllm.utils.network_utils`, `vllm.v1.engine`, `vllm.v1.engine.coordinator`, `vllm.v1.engine.core`, `vllm.v1.engine.exceptions`
