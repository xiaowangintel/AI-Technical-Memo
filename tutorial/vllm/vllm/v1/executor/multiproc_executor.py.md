# multiproc_executor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/executor/multiproc_executor.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `FutureWrapper`, `MultiprocExecutor`, `UnreadyWorkerProcHandle` for the V1 `executor` subsystem. / 为 V1 的 `executor` 子系统实现 `FutureWrapper`, `MultiprocExecutor`, `UnreadyWorkerProcHandle`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import multiprocessing
import os
import pickle
import queue
import signal
import threading
import time
import traceback
import weakref
from collections import deque
from collections.abc import Callable, Sequence
from concurrent.futures import Future, InvalidStateError
from contextlib import suppress
from dataclasses import dataclass
from enum import Enum, auto
from functools import cached_property, partial
from multiprocessing.connection import Connection
from multiprocessing.process import BaseProcess
from multiprocessing.synchronize import Lock as LockType
from threading import Thread
from typing import Any, cast

import cloudpickle
import torch

import vllm.envs as envs
from vllm.config import VllmConfig
from vllm.distributed import destroy_distributed_environment, destroy_model_parallel
from vllm.distributed.device_communicators.shm_broadcast import Handle, MessageQueue
from vllm.distributed.kv_transfer.kv_connector.utils import KVOutputAggregator
from vllm.distributed.parallel_state import (
    get_dcp_group,
    get_dp_group,
    get_ep_group,
    get_inner_dp_world_group,
    get_pcp_group,
    get_pp_group,
    get_tp_group,
    model_parallel_is_initialized,
)
from vllm.envs import enable_envs_cache
from vllm.logger import init_logger
from vllm.platforms import current_platform
from vllm.tracing import instrument, maybe_init_worker_tracer
from vllm.utils import numa_utils
from vllm.utils.network_utils import (
    get_distributed_init_method,
    get_ip,
    get_loopback_ip,
    get_open_port,
)
from vllm.utils.ompmultiprocessing import OMPProcessManager
from vllm.utils.system_utils import (
    _maybe_force_spawn,
    decorate_logs,
    get_mp_context,
    set_process_title,
)
from vllm.v1.core.sched.output import GrammarOutput, SchedulerOutput
from vllm.v1.executor.abstract import Executor, FailureCallback
from vllm.v1.outputs import AsyncModelRunnerOutput, DraftTokenIds, ModelRunnerOutput
from vllm.v1.worker.worker_base import WorkerWrapperBase

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `FutureWrapper` class / `FutureWrapper` 类
```python
class FutureWrapper(Future):
```
**EN:** Introduces the `FutureWrapper` class on top of `Future`. Core methods include `__init__`, `result`, `_wait_for_response`.
**CN:** 这里定义 `FutureWrapper` 类，其基类包括 `Future`。核心方法包括 `__init__`, `result`, `_wait_for_response`。

### `FutureWrapper.__init__` method / `FutureWrapper.__init__` 方法
```python
    def __init__(
        self,
        futures_queue: deque["FutureWrapper"],
        get_response: Callable[[], Any],
        aggregate: Callable = lambda x: x,
    ):
        self.futures_queue = futures_queue
        self.get_response = get_response
        self.aggregate = aggregate
        super().__init__()
        self.futures_queue.appendleft(self)
```
**EN:** This method initializes the object state within `FutureWrapper`. Key calls include `__init__`, `appendleft`, `super`. It touches state such as `futures_queue`, `get_response`, `aggregate`.
**CN:** 该方法会初始化对象状态，其作用域位于`FutureWrapper`。 关键调用包括 `__init__`, `appendleft`, `super`。 它会读写 `futures_queue`, `get_response`, `aggregate` 等状态。

### `FutureWrapper.result` method / `FutureWrapper.result` 方法
```python
    def result(self, timeout=None):
        if timeout is not None:
            raise RuntimeError("timeout not implemented")

        # Drain any futures ahead of us in the queue.
        while not self.done():
            future = self.futures_queue.pop()
            future._wait_for_response()
        return super().result()
```
**EN:** This method implements `result` within `FutureWrapper`. Key calls include `result`, `RuntimeError`, `done`, `pop`, `_wait_for_response`, `super`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `result`，其作用域位于`FutureWrapper`。 关键调用包括 `result`, `RuntimeError`, `done`, `pop`, `_wait_for_response`, `super`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `FutureWrapper._wait_for_response` method / `FutureWrapper._wait_for_response` 方法
```python
    def _wait_for_response(self):
        try:
            response = self.aggregate(self.get_response())
            with suppress(InvalidStateError):
                self.set_result(response)
        except Exception as e:
            with suppress(InvalidStateError):
                self.set_exception(e)
```
**EN:** This method implements `_wait_for_response` within `FutureWrapper`. Key calls include `aggregate`, `get_response`, `suppress`, `set_result`, `set_exception`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_wait_for_response`，其作用域位于`FutureWrapper`。 关键调用包括 `aggregate`, `get_response`, `suppress`, `set_result`, `set_exception`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `MultiprocExecutor` class / `MultiprocExecutor` 类
```python
class MultiprocExecutor(Executor):
    supports_pp: bool = True
```
**EN:** Introduces the `MultiprocExecutor` class on top of `Executor`. Core methods include `__init__`, `_init_executor`, `_get_parallel_sizes`, `_post_init_executor`, `_is_driver_worker`, `start_worker_monitor`.
**CN:** 这里定义 `MultiprocExecutor` 类，其基类包括 `Executor`。核心方法包括 `__init__`, `_init_executor`, `_get_parallel_sizes`, `_post_init_executor`, `_is_driver_worker`, `start_worker_monitor`。

### `MultiprocExecutor._init_executor` method / `MultiprocExecutor._init_executor` 方法
```python
    def _init_executor(self) -> None:
        # Call self.shutdown at exit to clean up
        # and ensure workers will be terminated.
        self._finalizer = weakref.finalize(self, self.shutdown)
        self.is_failed = False
        self.failure_callback: FailureCallback | None = None

        tp_size, pp_size, pcp_size = self._get_parallel_sizes()
        assert self.world_size == tp_size * pp_size * pcp_size, (
            f"world_size ({self.world_size}) must be equal to the "
            f"tensor_parallel_size ({tp_size}) x pipeline"
            f"_parallel_size ({pp_size}) x prefill_context"
            f"_parallel_size ({pcp_size}). "
        )

        set_multiprocessing_worker_envs()

        # use the loopback address get_loopback_ip() for communication.
        distributed_init_method = get_distributed_init_method(
            get_loopback_ip(), get_open_port()
        )
        self.rpc_broadcast_mq: MessageQueue | None = None
        scheduler_output_handle: Handle | None = None
        # Initialize worker and set up message queues for SchedulerOutputs
        # and ModelRunnerOutputs
        if self.parallel_config.node_rank_within_dp == 0:
            # For leader node within each dp rank,
            # each dp will have its own leader multiproc executor.
            max_chunk_bytes = envs.VLLM_MQ_MAX_CHUNK_BYTES_MB * 1024 * 1024
            mq_connect_ip = get_ip()
            logger.info(
                "DP group leader: node_rank=%d, node_rank_within_dp=%d, "
                "master_addr=%s, mq_connect_ip=%s (local), "
                "world_size=%d, local_world_size=%d",
                self.parallel_config.node_rank,
                self.parallel_config.node_rank_within_dp,
                self.parallel_config.master_addr,
                mq_connect_ip,
                self.world_size,
                self.local_world_size,
            )
            self.rpc_broadcast_mq = MessageQueue(
                self.world_size,
                self.local_world_size,
                max_chunk_bytes=max_chunk_bytes,
                connect_ip=mq_connect_ip,
            )
            scheduler_output_handle = self.rpc_broadcast_mq.export_handle()
        # Create workers
        context = get_mp_context()
    # ... omitted for brevity ...
            # Must be kept consistent with the WorkerProc.

            # Wait for all input mqs to be ready.
            if self.rpc_broadcast_mq is not None:
                self.rpc_broadcast_mq.wait_until_ready()
            # Wait for all remote response mqs to be ready.
            for response_mq in self.response_mqs:
                response_mq.wait_until_ready()

            self.futures_queue = deque[FutureWrapper]()

            self._post_init_executor()

            success = True
        finally:
            if not success:
                # Clean up the worker procs if there was a failure.
                # Close death_writers first to signal workers to exit
                for uw in unready_workers:
                    if uw.death_writer is not None:
                        uw.death_writer.close()
                        uw.death_writer = None
                self._ensure_worker_termination([uw.proc for uw in unready_workers])

        self.output_rank = self._get_output_rank()
```
**EN:** This method implements `_init_executor` within `MultiprocExecutor`. Key calls include `finalize`, `_get_parallel_sizes`, `set_multiprocessing_worker_envs`, `get_distributed_init_method`, `get_mp_context`, `Lock`. It touches state such as `_finalizer`, `is_failed`, `failure_callback`, `rpc_broadcast_mq`, `output_rank`, `workers`, `response_mqs`, `futures_queue`. The control flow contains 10 branch(es) and 4 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `_init_executor`，其作用域位于`MultiprocExecutor`。 关键调用包括 `finalize`, `_get_parallel_sizes`, `set_multiprocessing_worker_envs`, `get_distributed_init_method`, `get_mp_context`, `Lock`。 它会读写 `_finalizer`, `is_failed`, `failure_callback`, `rpc_broadcast_mq`, `output_rank`, `workers`, `response_mqs`, `futures_queue` 等状态。 控制流包含 10 个分支和 4 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `MultiprocExecutor.execute_model` method / `MultiprocExecutor.execute_model` 方法
```python
    def execute_model(  # type: ignore[override]
        self, scheduler_output: SchedulerOutput, non_block: bool = False
    ) -> ModelRunnerOutput | None | Future[ModelRunnerOutput | None]:
        return self.collective_rpc(
            "execute_model",
            args=(scheduler_output,),
            unique_reply_rank=self.output_rank,
            non_block=non_block,
            timeout=envs.VLLM_EXECUTE_MODEL_TIMEOUT_SECONDS,
            kv_output_aggregator=self.kv_output_aggregator,
        )
```
**EN:** This method executes the main operation within `MultiprocExecutor`. Key calls include `collective_rpc`.
**CN:** 该方法会执行主要操作，其作用域位于`MultiprocExecutor`。 关键调用包括 `collective_rpc`。

### `MultiprocExecutor.execute_dummy_batch` method / `MultiprocExecutor.execute_dummy_batch` 方法
```python
    def execute_dummy_batch(self) -> None:
        self.collective_rpc("execute_dummy_batch", unique_reply_rank=self.output_rank)
```
**EN:** This method executes the main operation within `MultiprocExecutor`. Key calls include `collective_rpc`.
**CN:** 该方法会执行主要操作，其作用域位于`MultiprocExecutor`。 关键调用包括 `collective_rpc`。

### `MultiprocExecutor.collective_rpc` method / `MultiprocExecutor.collective_rpc` 方法
```python
    def collective_rpc(  # type: ignore[override]
        self,
        method: str | Callable,
        timeout: float | None = None,
        args: tuple = (),
        kwargs: dict | None = None,
        non_block: bool = False,
        unique_reply_rank: int | None = None,
        kv_output_aggregator: KVOutputAggregator | None = None,
    ) -> Any:
        """Returns single result if unique_reply_rank and/or kv_output_aggregator
        is provided, otherwise list."""
        assert self.rpc_broadcast_mq is not None, (
            "collective_rpc should not be called on follower node"
        )
        if self.is_failed:
            raise RuntimeError("Executor failed.")

        deadline = None if timeout is None else time.monotonic() + timeout
        kwargs = kwargs or {}

        if kv_output_aggregator is not None:
            output_rank = None
            aggregate: Callable[[Any], Any] = partial(
                kv_output_aggregator.aggregate, output_rank=unique_reply_rank or 0
            )
        else:
            output_rank = unique_reply_rank
            aggregate = lambda x: x

        if isinstance(method, str):
            send_method = method
        else:
            send_method = cloudpickle.dumps(method, protocol=pickle.HIGHEST_PROTOCOL)
        self.rpc_broadcast_mq.enqueue((send_method, args, kwargs, output_rank))

        response_mqs: Sequence[MessageQueue] = self.response_mqs
        if output_rank is not None:
            response_mqs = (response_mqs[output_rank],)

        def get_response():
            responses = []
            for mq in response_mqs:
                dequeue_timeout = (
                    None if deadline is None else (deadline - time.monotonic())
                )
                try:
                    status, result = mq.dequeue(timeout=dequeue_timeout)
                except TimeoutError as e:
                    raise TimeoutError(f"RPC call to {method} timed out.") from e
                if status != WorkerProc.ResponseStatus.SUCCESS:
                    raise RuntimeError(
                        f"Worker failed with error '{result}', please check the"
                        " stack trace above for the root cause"
                    )
                responses.append(result)
            return responses[0] if output_rank is not None else responses

        future = FutureWrapper(
            self.futures_queue,
            get_response=get_response,
            aggregate=aggregate,
        )

        return future if non_block else future.result()
```
**EN:** This method collects data from collaborators within `MultiprocExecutor`. The docstring frames it as: Returns single result if unique_reply_rank and/or kv_output_aggregator is provided, otherwise list. Key calls include `isinstance`, `enqueue`, `FutureWrapper`, `RuntimeError`, `partial`, `dumps`. The control flow contains 10 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会从协作组件收集数据，其作用域位于`MultiprocExecutor`。 关键调用包括 `isinstance`, `enqueue`, `FutureWrapper`, `RuntimeError`, `partial`, `dumps`。 控制流包含 10 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `UnreadyWorkerProcHandle` class / `UnreadyWorkerProcHandle` 类
```python
@dataclass
class UnreadyWorkerProcHandle:
    """WorkerProcess handle before READY."""

    proc: BaseProcess
    rank: int
    ready_pipe: Connection
    death_writer: Connection | None = None
```
**EN:** Uses `@dataclass` to package related state for `UnreadyWorkerProcHandle`. Typical fields include `proc`, `rank`, `ready_pipe`, `death_writer`.
**CN:** `UnreadyWorkerProcHandle` 使用 `@dataclass` 打包相关状态。典型字段包括 `proc`, `rank`, `ready_pipe`, `death_writer`。

### `WorkerProcHandle` class / `WorkerProcHandle` 类
```python
@dataclass
class WorkerProcHandle:
    proc: BaseProcess
    rank: int
    # The worker process writes to this MQ in single-node mode
    worker_response_mq: MessageQueue | None
    # This is only non empty on driver node,
    # the peer worker process i writes to MQ
    # `peer_worker_response_mqs[i]`
    peer_worker_response_mqs: list[MessageQueue | None]
    death_writer: Connection | None = None
```
**EN:** Uses `@dataclass` to package related state for `WorkerProcHandle`. Typical fields include `proc`, `rank`, `worker_response_mq`, `peer_worker_response_mqs`, `death_writer`.
**CN:** `WorkerProcHandle` 使用 `@dataclass` 打包相关状态。典型字段包括 `proc`, `rank`, `worker_response_mq`, `peer_worker_response_mqs`, `death_writer`。

### `WorkerProcHandle.from_unready_handle` method / `WorkerProcHandle.from_unready_handle` 方法
```python
    @classmethod
    def from_unready_handle(
        cls,
        unready_handle: UnreadyWorkerProcHandle,
        worker_response_mq: MessageQueue | None,
        peer_worker_response_mqs: list[MessageQueue | None],
    ) -> "WorkerProcHandle":
        return cls(
            proc=unready_handle.proc,
            rank=unready_handle.rank,
            worker_response_mq=worker_response_mq,
            peer_worker_response_mqs=peer_worker_response_mqs,
            death_writer=unready_handle.death_writer,
        )
```
**EN:** This method reconstructs data from another representation within `WorkerProcHandle`. Key calls include `cls`.
**CN:** 该方法会从另一种表示恢复数据，其作用域位于`WorkerProcHandle`。 关键调用包括 `cls`。

### `WorkerProc` class / `WorkerProc` 类
```python
class WorkerProc:
    """Wrapper that runs one Worker in a separate process."""

    READY_STR = "READY"
    rpc_broadcast_mq: MessageQueue | None
    worker_response_mq: MessageQueue | None
```
**EN:** Introduces the `WorkerProc` class. Core methods include `_init_message_queues`, `__init__`, `make_worker_process`, `wait_for_response_handle_ready`, `wait_for_ready`, `shutdown`. Docstring signal: Wrapper that runs one Worker in a separate process.
**CN:** 这里定义 `WorkerProc` 类。核心方法包括 `_init_message_queues`, `__init__`, `make_worker_process`, `wait_for_response_handle_ready`, `wait_for_ready`, `shutdown`。

### `WorkerProc.__init__` method / `WorkerProc.__init__` 方法
```python
    @instrument(span_name="Worker init")
    def __init__(
        self,
        vllm_config: VllmConfig,
        local_rank: int,
        rank: int,
        distributed_init_method: str,
        input_shm_handle: Handle,
        shared_worker_lock: LockType,
        is_driver_worker: bool,
    ):
        self.rank = rank
        wrapper = WorkerWrapperBase(rpc_rank=local_rank, global_rank=rank)
        # TODO: move `init_worker` to executor level as a collective rpc call
        all_kwargs: list[dict] = [
            {} for _ in range(vllm_config.parallel_config.world_size)
        ]
        all_kwargs[local_rank] = {
            "vllm_config": vllm_config,
            "local_rank": local_rank,
            "rank": rank,
            "distributed_init_method": distributed_init_method,
            "is_driver_worker": is_driver_worker,
            "shared_worker_lock": shared_worker_lock,
        }
        wrapper.init_worker(all_kwargs)
        self.worker = wrapper

        self.setup_proc_title_and_log_prefix(
            enable_ep=vllm_config.parallel_config.enable_expert_parallel
        )

        # Load model
        self.worker.init_device()
        # Update process title now that parallel groups are initialized
        self.setup_proc_title_and_log_prefix(
            enable_ep=vllm_config.parallel_config.enable_expert_parallel
        )
        if envs.VLLM_ELASTIC_EP_SCALE_UP_LAUNCH:
            self.worker.elastic_ep_execute("load_model")
        else:
            self.worker.load_model()

        scheduler_config = vllm_config.scheduler_config
        self.use_async_scheduling = scheduler_config.async_scheduling
        if self.use_async_scheduling:
            self.async_output_queue: queue.Queue = queue.Queue()
            self.async_output_copy_thread = Thread(
                target=self.async_output_busy_loop,
                daemon=True,
                name="WorkerAsyncOutputCopy",
            )
            self.async_output_copy_thread.start()

        # Set block size based on the attention backends
        current_platform.update_block_size_for_backend(vllm_config)

        # Initialize message queues after init_device() since multi-node setups
        # (nnodes_within_dp > 1) require distributed groups to be initialized
        self._init_message_queues(input_shm_handle, vllm_config)

        # Enable environment variable cache (e.g. assume no more
        # environment variable overrides after this point)
        enable_envs_cache()
```
**EN:** This method initializes the object state within `WorkerProc`. Key calls include `instrument`, `WorkerWrapperBase`, `init_worker`, `setup_proc_title_and_log_prefix`, `init_device`, `update_block_size_for_backend`. It touches state such as `rank`, `worker`, `use_async_scheduling`, `async_output_queue`, `async_output_copy_thread`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`WorkerProc`。 关键调用包括 `instrument`, `WorkerWrapperBase`, `init_worker`, `setup_proc_title_and_log_prefix`, `init_device`, `update_block_size_for_backend`。 它会读写 `rank`, `worker`, `use_async_scheduling`, `async_output_queue`, `async_output_copy_thread` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `WorkerProc.make_worker_process` method / `WorkerProc.make_worker_process` 方法
```python
    @staticmethod
    def make_worker_process(
        vllm_config: VllmConfig,
        local_rank: int,
        rank: int,
        distributed_init_method: str,
        input_shm_handle,  # Receive SchedulerOutput
        shared_worker_lock: LockType,
        is_driver_worker: bool,
        inherited_fds: list[int] | None = None,
    ) -> UnreadyWorkerProcHandle:
        context = get_mp_context()
        # Ready pipe to communicate readiness from child to parent
        ready_reader, ready_writer = context.Pipe(duplex=False)
        # Death pipe to let child detect parent process exit
        death_reader, death_writer = context.Pipe(duplex=False)
        if inherited_fds is not None:
            inherited_fds = inherited_fds.copy()
            inherited_fds.extend((ready_reader.fileno(), death_writer.fileno()))
        process_kwargs = {
            "vllm_config": vllm_config,
            "local_rank": local_rank,
            "rank": rank,
            "distributed_init_method": distributed_init_method,
            "input_shm_handle": input_shm_handle,
            "ready_pipe": ready_writer,
            "death_pipe": death_reader,
            "shared_worker_lock": shared_worker_lock,
            "is_driver_worker": is_driver_worker,
            # Have the worker close parent end of this worker's pipes too
            "inherited_fds": inherited_fds if inherited_fds is not None else [],
        }
        # Run EngineCore busy loop in background process.
        proc = context.Process(
            target=WorkerProc.worker_main,
            kwargs=process_kwargs,
            name=f"VllmWorker-{rank}",
            daemon=True,
        )

        # Apply NUMA binding if configured
        with numa_utils.configure_subprocess(
            vllm_config, local_rank, process_kind="worker"
        ):
            proc.start()

        # Close child ends of pipes here in the parent
        ready_writer.close()
        death_reader.close()
        # Keep death_writer open in parent - when parent exits,
        # death_reader in child will get EOFError
        return UnreadyWorkerProcHandle(proc, rank, ready_reader, death_writer)
```
**EN:** This method implements `make_worker_process` within `WorkerProc`. Key calls include `get_mp_context`, `Pipe`, `Process`, `close`, `UnreadyWorkerProcHandle`, `copy`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `make_worker_process`，其作用域位于`WorkerProc`。 关键调用包括 `get_mp_context`, `Pipe`, `Process`, `close`, `UnreadyWorkerProcHandle`, `copy`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `WorkerProc.wait_for_ready` method / `WorkerProc.wait_for_ready` 方法
```python
    @staticmethod
    def wait_for_ready(
        unready_proc_handles: list[UnreadyWorkerProcHandle],
    ) -> list[WorkerProcHandle]:
        e = Exception(
            "WorkerProc initialization failed due to an exception in a "
            "background process. See stack trace for root cause."
        )

        pipes = {handle.ready_pipe: handle for handle in unready_proc_handles}
        ready_proc_handles: list[WorkerProcHandle | None] = [None] * len(
            unready_proc_handles
        )
        while pipes:
            ready = multiprocessing.connection.wait(pipes.keys())
            for pipe in ready:
                assert isinstance(pipe, Connection)
                try:
                    # Wait until the WorkerProc is ready.
                    unready_proc_handle = pipes.pop(pipe)
                    response: dict[str, Any] = pipe.recv()
                    if response["status"] != "READY":
                        raise e

                    idx = unready_proc_handle.rank % len(ready_proc_handles)
                    ready_proc_handles[idx] = WorkerProc.wait_for_response_handle_ready(
                        response, unready_proc_handle
                    )
                except EOFError:
                    e.__suppress_context__ = True
                    raise e from None

                finally:
                    # Close connection.
                    pipe.close()

        return cast(list[WorkerProcHandle], ready_proc_handles)
```
**EN:** This method implements `wait_for_ready` within `WorkerProc`. Key calls include `Exception`, `cast`, `len`, `wait`, `keys`, `isinstance`. The control flow contains 2 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `wait_for_ready`，其作用域位于`WorkerProc`。 关键调用包括 `Exception`, `cast`, `len`, `wait`, `keys`, `isinstance`。 控制流包含 2 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `WorkerProc.worker_main` method / `WorkerProc.worker_main` 方法
```python
    @staticmethod
    def worker_main(*args, **kwargs):
        """Worker initialization and execution loops.
        This runs a background process"""

        # Signal handler used for graceful termination.
        # SystemExit exception is only raised once to allow this and worker
        # processes to terminate without error
        shutdown_requested = threading.Event()

        def signal_handler(signum, frame):
            nonlocal shutdown_requested
            if not shutdown_requested.is_set():
                shutdown_requested.set()
                logger.debug(
                    "WorkerProc handling signal %d, raising SystemExit", signum
                )
                raise SystemExit()

        # Either SIGTERM or SIGINT will terminate the worker
        signal.signal(signal.SIGTERM, signal_handler)
        signal.signal(signal.SIGINT, signal_handler)

        worker = None
        ready_writer = kwargs.pop("ready_pipe")
        death_pipe = kwargs.pop("death_pipe", None)

        # Close inherited pipes from parent (incl. other worker pipes)
        # Explicitly passing in existing pipes and closing them makes the pipe
        # behave when using fork. Otherwise, a hidden reference to the pipes
        # exist in the child process and prevents EOF closure.
        for fd in kwargs.pop("inherited_fds", []):
            try:
                os.close(fd)
            except Exception as e:
                logger.warning("Error closing inherited connection: %s: %s", type(e), e)

        try:
            # Initialize tracer
            rank = kwargs.get("rank", 0)
            maybe_init_worker_tracer(
                instrumenting_module_name="vllm.worker",
                process_kind="worker",
                process_name=f"Worker_{rank}",
            )

            worker = WorkerProc(*args, **kwargs)
            assert worker.worker_response_mq is not None
            if kwargs["vllm_config"].parallel_config.numa_bind:
                numa_utils.log_current_affinity_state(f"Worker_{worker.rank}")
    # ... omitted for brevity ...
            elif shutdown_requested.is_set():
                logger.info("WorkerProc shutting down.")
            else:
                logger.exception("WorkerProc failed.")

            # The parent sends a SIGTERM to all worker processes if
            # any worker dies. Set this value so we don't re-throw
            # SystemExit() to avoid zmq exceptions in __del__.
            shutdown_requested.set()

        except SystemExit as e:
            # SystemExit is raised on SIGTERM or SIGKILL, which usually indicates that
            # the graceful shutdown process did not succeed
            logger.warning("WorkerProc was terminated")
            # SystemExit must never be ignored
            raise e

        finally:
            if ready_writer is not None:
                ready_writer.close()
            if death_pipe is not None:
                death_pipe.close()
            # Clean up once worker exits busy loop
            if worker is not None:
                worker.shutdown()
```
**EN:** This method implements `worker_main` within `WorkerProc`. The docstring frames it as: Worker initialization and execution loops. Key calls include `Event`, `signal`, `pop`, `get`, `maybe_init_worker_tracer`, `WorkerProc`. The control flow contains 10 branch(es) and 1 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `worker_main`，其作用域位于`WorkerProc`。 关键调用包括 `Event`, `signal`, `pop`, `get`, `maybe_init_worker_tracer`, `WorkerProc`。 控制流包含 10 个分支和 1 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `set_multiprocessing_worker_envs` function / `set_multiprocessing_worker_envs` 函数
```python
def set_multiprocessing_worker_envs():
    """Set up environment variables that should be used when there are workers
    in a multiprocessing environment. This should be called by the parent
    process before worker processes are created"""

    _maybe_force_spawn()

    if not current_platform.is_cpu():
        # Configure thread parallelism if OMP_NUM_THREADS isn't set
        #
        # Helps to avoid CPU contention. The default of spawning a thread per
        # core combined with multiprocessing for each GPU can have a negative
        # impact on performance. The contention is amplified when running in a
        # container where CPU limits can cause throttling.
        default_omp_num_threads = 1
        if (
            "OMP_NUM_THREADS" not in os.environ
            and (current_parallelism := torch.get_num_threads())
            > default_omp_num_threads
        ):
            logger.warning_once(
                "Reducing Torch parallelism from %d threads to %d to avoid "
                "unnecessary CPU contention. Set OMP_NUM_THREADS in the "
                "external environment to tune this value as needed.",
                current_parallelism,
                default_omp_num_threads,
            )
            os.environ["OMP_NUM_THREADS"] = str(default_omp_num_threads)
            torch.set_num_threads(default_omp_num_threads)
```
**EN:** This function stores a value into object state within the module. The docstring frames it as: Set up environment variables that should be used when there are workers in a multiprocessing environment. Key calls include `_maybe_force_spawn`, `is_cpu`, `warning_once`, `str`, `set_num_threads`, `get_num_threads`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会将值写入对象状态，其作用域位于the module。 关键调用包括 `_maybe_force_spawn`, `is_cpu`, `warning_once`, `str`, `set_num_threads`, `get_num_threads`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `FutureWrapper`: central class or interface in this module. / `FutureWrapper`：本模块中的核心类或接口。
- `MultiprocExecutor`: central class or interface in this module. / `MultiprocExecutor`：本模块中的核心类或接口。
- `UnreadyWorkerProcHandle`: central class or interface in this module. / `UnreadyWorkerProcHandle`：本模块中的核心类或接口。
- `WorkerProcHandle`: central class or interface in this module. / `WorkerProcHandle`：本模块中的核心类或接口。
- `WorkerProc`: central class or interface in this module. / `WorkerProc`：本模块中的核心类或接口。
- `set_multiprocessing_worker_envs`: top-level helper or orchestration entry point. / `set_multiprocessing_worker_envs`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `multiprocessing`, `os`, `pickle`, `queue`, `signal`, `threading`, `time`, `traceback`, `weakref`, `collections`, `concurrent`, `contextlib`
- External / 外部依赖: `cloudpickle`, `torch`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.config`, `vllm.distributed`, `vllm.distributed.device_communicators.shm_broadcast`, `vllm.distributed.kv_transfer.kv_connector.utils`, `vllm.distributed.parallel_state`, `vllm.logger`, `vllm.platforms`, `vllm.tracing`, `vllm.utils`, `vllm.utils.network_utils`, `vllm.utils.ompmultiprocessing`
