# core.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/engine/core.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `EngineCore`, `EngineShutdownState`, `EngineCoreProc` for the V1 `engine` subsystem. / 为 V1 的 `engine` 子系统实现 `EngineCore`, `EngineShutdownState`, `EngineCoreProc`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import gc
import os
import queue
import signal
import threading
import time
from collections import defaultdict, deque
from collections.abc import Callable, Generator
from concurrent.futures import Future
from contextlib import ExitStack, contextmanager
from enum import IntEnum
from functools import partial
from inspect import isclass, signature
from logging import DEBUG
from multiprocessing.queues import Queue
from typing import Any, TypeVar, cast

import msgspec
import zmq

import vllm.envs as envs
from vllm.config import ParallelConfig, VllmConfig
from vllm.distributed import stateless_destroy_torch_distributed_process_group
from vllm.envs import enable_envs_cache
from vllm.logger import init_logger
from vllm.logging_utils.dump_input import dump_engine_exception
from vllm.lora.request import LoRARequest
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.tasks import POOLING_TASKS, SupportedTask
from vllm.tracing import instrument, maybe_init_worker_tracer
from vllm.transformers_utils.config import maybe_register_config_serialize_by_value
from vllm.utils import numa_utils
from vllm.utils.gc_utils import (
    freeze_gc_heap,
    maybe_attach_gc_debug_callback,
)
from vllm.utils.hashing import get_hash_fn_by_name
from vllm.utils.network_utils import make_zmq_socket
from vllm.utils.system_utils import decorate_logs, set_process_title
from vllm.v1.core.kv_cache_utils import (
    BlockHash,
    generate_scheduler_kv_cache_config,
    get_kv_cache_configs,
    get_request_block_hasher,
    init_none_hash,
    resolve_kv_cache_block_sizes,
)
from vllm.v1.core.sched.interface import PauseState, SchedulerInterface
from vllm.v1.core.sched.output import SchedulerOutput
from vllm.v1.engine import (
# ... omitted for brevity ...
    UtilityOutput,
    UtilityResult,
)
from vllm.v1.engine.tensor_ipc import TensorIpcReceiver
from vllm.v1.engine.utils import (
    EngineHandshakeMetadata,
    EngineZmqAddresses,
    SignalCallback,
    get_device_indices,
)
from vllm.v1.executor import Executor
from vllm.v1.kv_cache_interface import KVCacheConfig, get_kv_cache_spec_kind
from vllm.v1.metrics.stats import SchedulerStats
from vllm.v1.outputs import ModelRunnerOutput
from vllm.v1.request import Request, RequestStatus
from vllm.v1.serial_utils import MsgpackDecoder, MsgpackEncoder
from vllm.v1.structured_output import StructuredOutputManager
from vllm.v1.utils import compute_iteration_details
from vllm.version import __version__ as VLLM_VERSION

logger = init_logger(__name__)

HANDSHAKE_TIMEOUT_MINS = 5

_R = TypeVar("_R")  # Return type for collective_rpc
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `HANDSHAKE_TIMEOUT_MINS`, `_R`. This excerpt omits repetitive declarations.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `HANDSHAKE_TIMEOUT_MINS`, `_R`。 该片段省略了重复性声明。

### `EngineCore` class / `EngineCore` 类
```python
class EngineCore:
    """Inner loop of vLLM's Engine."""
```
**EN:** Introduces the `EngineCore` class. Core methods include `__init__`, `_initialize_kv_caches`, `get_supported_tasks`, `get_kv_cache_group_metadata`, `add_request`, `abort_requests`. Docstring signal: Inner loop of vLLM's Engine.
**CN:** 这里定义 `EngineCore` 类。核心方法包括 `__init__`, `_initialize_kv_caches`, `get_supported_tasks`, `get_kv_cache_group_metadata`, `add_request`, `abort_requests`。

### `EngineCore.__init__` method / `EngineCore.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        executor_class: type[Executor],
        log_stats: bool,
        executor_fail_callback: Callable | None = None,
        include_finished_set: bool = False,
    ):
        # plugins need to be loaded at the engine/scheduler level too
        from vllm.plugins import load_general_plugins

        load_general_plugins()

        self.vllm_config = vllm_config
        if not vllm_config.parallel_config.data_parallel_rank_local:
            logger.info(
                "Initializing a V1 LLM engine (v%s) with config: %s",
                VLLM_VERSION,
                vllm_config,
            )

        self.log_stats = log_stats

        # Setup Model.
        self.model_executor = executor_class(vllm_config)
        if executor_fail_callback is not None:
            self.model_executor.register_failure_callback(executor_fail_callback)

        self.available_gpu_memory_for_kv_cache = -1

        if envs.VLLM_ELASTIC_EP_SCALE_UP_LAUNCH:
            self._eep_scale_up_before_kv_init()

        # Setup KV Caches and update CacheConfig after profiling.
        kv_cache_config = self._initialize_kv_caches(vllm_config)
        self.structured_output_manager = StructuredOutputManager(vllm_config)

        # Setup scheduler.
        Scheduler = vllm_config.scheduler_config.get_scheduler_cls()

        if len(kv_cache_config.kv_cache_groups) == 0:  # noqa: SIM102
            # Encoder models without KV cache don't support
            # chunked prefill. But do SSM models?
            if vllm_config.scheduler_config.enable_chunked_prefill:
                logger.warning("Disabling chunked prefill for model without KVCache")
                vllm_config.scheduler_config.enable_chunked_prefill = False

        scheduler_block_size, hash_block_size = resolve_kv_cache_block_sizes(
            kv_cache_config, vllm_config
        )
    # ... omitted for brevity ...
                vllm_config.cache_config.prefix_caching_hash_algo
            )
            init_none_hash(caching_hash_fn)

            self.request_block_hasher = get_request_block_hasher(
                hash_block_size, caching_hash_fn
            )

        self.step_fn = (
            self.step if self.batch_queue is None else self.step_with_batch_queue
        )
        self.async_scheduling = vllm_config.scheduler_config.async_scheduling

        self.aborts_queue = queue.Queue[list[str]]()

        self._idle_state_callbacks: list[Callable] = []

        # Mark the startup heap as static so that it's ignored by GC.
        # Reduces pause times of oldest generation collections.
        freeze_gc_heap()
        # If enable, attach GC debugger after static variable freeze.
        maybe_attach_gc_debug_callback()
        # Enable environment variable cache (e.g. assume no more
        # environment variable overrides after this point)
        enable_envs_cache()
```
**EN:** This method initializes the object state within `EngineCore`. Key calls include `load_general_plugins`, `executor_class`, `_initialize_kv_caches`, `StructuredOutputManager`, `get_scheduler_cls`, `resolve_kv_cache_block_sizes`. It touches state such as `vllm_config`, `log_stats`, `model_executor`, `available_gpu_memory_for_kv_cache`, `structured_output_manager`, `scheduler`, `use_spec_decode`, `mm_receiver_cache`. The control flow contains 12 branch(es) and 1 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会初始化对象状态，其作用域位于`EngineCore`。 关键调用包括 `load_general_plugins`, `executor_class`, `_initialize_kv_caches`, `StructuredOutputManager`, `get_scheduler_cls`, `resolve_kv_cache_block_sizes`。 它会读写 `vllm_config`, `log_stats`, `model_executor`, `available_gpu_memory_for_kv_cache`, `structured_output_manager`, `scheduler`, `use_spec_decode`, `mm_receiver_cache` 等状态。 控制流包含 12 个分支和 1 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `EngineCore.step` method / `EngineCore.step` 方法
```python
    def step(self) -> tuple[dict[int, EngineCoreOutputs], bool]:
        """Schedule, execute, and make output.

        Returns tuple of outputs and a flag indicating whether the model
        was executed.
        """

        # Check for any requests remaining in the scheduler - unfinished,
        # or finished and not yet removed from the batch.
        if not self.scheduler.has_requests():
            return {}, False
        scheduler_output = self.scheduler.schedule()
        future = self.model_executor.execute_model(scheduler_output, non_block=True)
        grammar_output = self.scheduler.get_grammar_bitmask(scheduler_output)
        with (
            self.log_error_detail(scheduler_output),
            self.log_iteration_details(scheduler_output),
        ):
            model_output = future.result()
            if model_output is None:
                model_output = self.model_executor.sample_tokens(grammar_output)

        # Before processing the model output, process any aborts that happened
        # during the model execution.
        self._process_aborts_queue()
        engine_core_outputs = self.scheduler.update_from_output(
            scheduler_output, model_output
        )

        return engine_core_outputs, scheduler_output.total_num_scheduled_tokens > 0
```
**EN:** This method advances one execution step within `EngineCore`. The docstring frames it as: Schedule, execute, and make output. Key calls include `schedule`, `execute_model`, `get_grammar_bitmask`, `_process_aborts_queue`, `update_from_output`, `has_requests`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会推进一次执行步骤，其作用域位于`EngineCore`。 关键调用包括 `schedule`, `execute_model`, `get_grammar_bitmask`, `_process_aborts_queue`, `update_from_output`, `has_requests`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EngineCore.post_step` method / `EngineCore.post_step` 方法
```python
    def post_step(self, model_executed: bool) -> None:
        # When using async scheduling we can't get draft token ids in advance,
        # so we update draft token ids in the worker process and don't
        # need to update draft token ids here.
        if not self.async_scheduling and self.use_spec_decode and model_executed:
            # Take the draft token ids.
            draft_token_ids = self.model_executor.take_draft_token_ids()
            if draft_token_ids is not None:
                self.scheduler.update_draft_token_ids(draft_token_ids)
```
**EN:** This method implements `post_step` within `EngineCore`. Key calls include `take_draft_token_ids`, `update_draft_token_ids`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `post_step`，其作用域位于`EngineCore`。 关键调用包括 `take_draft_token_ids`, `update_draft_token_ids`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EngineCore.step_with_batch_queue` method / `EngineCore.step_with_batch_queue` 方法
```python
    def step_with_batch_queue(
        self,
    ) -> tuple[dict[int, EngineCoreOutputs] | None, bool]:
        """Schedule and execute batches with the batch queue.
        Note that if nothing to output in this step, None is returned.

        The execution flow is as follows:
        1. Try to schedule a new batch if the batch queue is not full.
        If a new batch is scheduled, directly return an empty engine core
        output. In other words, fulfilling the batch queue has a higher priority
        than getting model outputs.
        2. If there is no new scheduled batch, meaning that the batch queue
        is full or no other requests can be scheduled, we block until the first
        batch in the job queue is finished.
        3. Update the scheduler from the output.
        """

        batch_queue = self.batch_queue
        assert batch_queue is not None

        # Try to schedule a new batch if the batch queue is not full, but
        # the scheduler may return an empty batch if all requests are scheduled.
        # Note that this is not blocking.
        assert len(batch_queue) < self.batch_queue_size

        model_executed = False
        deferred_scheduler_output = None
        if self.scheduler.has_requests():
            scheduler_output = self.scheduler.schedule()
            with self.log_error_detail(scheduler_output):
                exec_future = self.model_executor.execute_model(
                    scheduler_output, non_block=True
                )
            if self.is_ec_consumer:
                model_executed = scheduler_output.total_num_scheduled_tokens > 0

            if self.is_pooling_model or not model_executed:
                # No sampling required (no requests scheduled).
                future = cast(Future[ModelRunnerOutput], exec_future)
            else:
                if not scheduler_output.pending_structured_output_tokens:
                    # We aren't waiting for any tokens, get any grammar output
                    # and sample immediately.
                    grammar_output = self.scheduler.get_grammar_bitmask(
                        scheduler_output
                    )
                    future = self.model_executor.sample_tokens(
                        grammar_output, non_block=True
                    )
                else:
    # ... omitted for brevity ...
        # NOTE(nick): We can either handle the deferred tasks here or save
        # in a field and do it immediately once step_with_batch_queue is
        # re-called. The latter slightly favors TTFT over TPOT/throughput.
        if deferred_scheduler_output:
            # If we are doing speculative decoding with structured output,
            # we need to get the draft token ids from the prior step before
            # we can compute the grammar bitmask for the deferred request.
            if self.use_spec_decode:
                draft_token_ids = self.model_executor.take_draft_token_ids()
                assert draft_token_ids is not None
                # Update the draft token ids in the scheduler output to
                # filter out the invalid spec tokens, which will be padded
                # with -1 and skipped by the grammar bitmask computation.
                self.scheduler.update_draft_token_ids_in_output(
                    draft_token_ids, deferred_scheduler_output
                )
            # We now have the tokens needed to compute the bitmask for the
            # deferred request. Get the bitmask and call sample tokens.
            grammar_output = self.scheduler.get_grammar_bitmask(
                deferred_scheduler_output
            )
            future = self.model_executor.sample_tokens(grammar_output, non_block=True)
            batch_queue.appendleft((future, deferred_scheduler_output, exec_future))

        return engine_core_outputs, model_executed
```
**EN:** This method advances one execution step within `EngineCore`. The docstring frames it as: Schedule and execute batches with the batch queue. Key calls include `has_requests`, `pop`, `_process_aborts_queue`, `update_from_output`, `len`, `schedule`. The control flow contains 10 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会推进一次执行步骤，其作用域位于`EngineCore`。 关键调用包括 `has_requests`, `pop`, `_process_aborts_queue`, `update_from_output`, `len`, `schedule`。 控制流包含 10 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `EngineCore.pause_scheduler` method / `EngineCore.pause_scheduler` 方法
```python
    def pause_scheduler(
        self, mode: PauseMode = "abort", clear_cache: bool = True
    ) -> Future | None:
        """Pause generation; behavior depends on mode.

        All pause modes queue new adds -- "abort" and "keep" skip step();
        "wait" allows step() so in-flight requests can drain.

        - ``abort``: Set PAUSED_NEW, abort all requests, wait for abort
          outputs to be sent (when running with output_queue), optionally
          clear caches, then complete the returned Future.
        - ``wait``: Set PAUSED_NEW (queue adds, keep stepping); when drained,
          optionally clear caches, then complete the returned Future.
        - ``keep``: Set PAUSED_ALL; return a Future that completes when the
          output queue is empty.
        """
        if mode not in ("keep", "abort", "wait"):
            raise ValueError(f"Invalid pause mode: {mode}")
        if mode == "wait":
            raise ValueError("'wait' mode can't be used in inproc-engine mode")

        if mode == "abort":
            self.scheduler.finish_requests(None, RequestStatus.FINISHED_ABORTED)

        pause_state = PauseState.PAUSED_ALL if mode == "keep" else PauseState.PAUSED_NEW
        self.scheduler.set_pause_state(pause_state)
        if clear_cache:
            self._reset_caches()

        return None
```
**EN:** This method implements `pause_scheduler` within `EngineCore`. The docstring frames it as: Pause generation; behavior depends on mode. Key calls include `set_pause_state`, `ValueError`, `finish_requests`, `_reset_caches`. The control flow contains 5 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `pause_scheduler`，其作用域位于`EngineCore`。 关键调用包括 `set_pause_state`, `ValueError`, `finish_requests`, `_reset_caches`。 控制流包含 5 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EngineShutdownState` class / `EngineShutdownState` 类
```python
class EngineShutdownState(IntEnum):
    RUNNING = 0
    REQUESTED = 1
    SHUTTING_DOWN = 2
```
**EN:** Defines the `EngineShutdownState` enum used to normalize modes or options across the subsystem. Representative members: `RUNNING`, `REQUESTED`, `SHUTTING_DOWN`.
**CN:** `EngineShutdownState` 定义了一个枚举，用于统一子系统中的模式或选项。代表性成员包括：`RUNNING`, `REQUESTED`, `SHUTTING_DOWN`。

### `EngineCoreProc` class / `EngineCoreProc` 类
```python
class EngineCoreProc(EngineCore):
    """ZMQ-wrapper for running EngineCore in background process."""

    ENGINE_CORE_DEAD = b"ENGINE_CORE_DEAD"
    addresses: EngineZmqAddresses
```
**EN:** Introduces the `EngineCoreProc` class on top of `EngineCore`. Core methods include `__init__`, `_perform_handshakes`, `_perform_handshake`, `startup_handshake`, `run_engine_core`, `_init_data_parallel`. Docstring signal: ZMQ-wrapper for running EngineCore in background process.
**CN:** 这里定义 `EngineCoreProc` 类，其基类包括 `EngineCore`。核心方法包括 `__init__`, `_perform_handshakes`, `_perform_handshake`, `startup_handshake`, `run_engine_core`, `_init_data_parallel`。

### `EngineCoreProc.__init__` method / `EngineCoreProc.__init__` 方法
```python
    @instrument(span_name="EngineCoreProc init")
    def __init__(
        self,
        vllm_config: VllmConfig,
        local_client: bool,
        handshake_address: str,
        executor_class: type[Executor],
        log_stats: bool,
        client_handshake_address: str | None = None,
        tensor_queue: Queue | None = None,
        *,
        engine_index: int = 0,
    ):
        self.input_queue = queue.Queue[tuple[EngineCoreRequestType, Any]]()
        self.output_queue = queue.Queue[tuple[int, EngineCoreOutputs] | bytes]()
        executor_fail_callback = lambda: self.input_queue.put_nowait(
            (EngineCoreRequestType.EXECUTOR_FAILED, b"")
        )

        self.engine_index = engine_index
        identity = self.engine_index.to_bytes(length=2, byteorder="little")
        self.engines_running = False
        self.shutdown_state = EngineShutdownState.RUNNING

        # Receiver for tensor IPC
        self.tensor_ipc_receiver: TensorIpcReceiver | None = None
        if tensor_queue is not None:
            self.tensor_ipc_receiver = TensorIpcReceiver(tensor_queue)
            logger.info("Using tensor IPC queue for multimodal tensor sharing")

        with self._perform_handshakes(
            handshake_address,
            identity,
            local_client,
            vllm_config,
            client_handshake_address,
        ) as addresses:
            # Set up data parallel environment.
            self.has_coordinator = addresses.coordinator_output is not None
            self.frontend_stats_publish_address = (
                addresses.frontend_stats_publish_address
            )
            logger.debug(
                "Has DP Coordinator: %s, stats publish address: %s",
                self.has_coordinator,
                self.frontend_stats_publish_address,
            )
            internal_dp_balancing = (
                self.has_coordinator
                and not vllm_config.parallel_config.data_parallel_external_lb
    # ... omitted for brevity ...
                    identity,
                    ready_event,
                ),
                daemon=True,
            )
            input_thread.start()

            self.output_thread = threading.Thread(
                target=self.process_output_sockets,
                args=(
                    addresses.outputs,
                    addresses.coordinator_output,
                    self.engine_index,
                ),
                daemon=True,
            )
            self.output_thread.start()

            # Don't complete handshake until DP coordinator ready message is
            # received.
            while not ready_event.wait(timeout=10):
                if not input_thread.is_alive():
                    raise RuntimeError("Input socket thread died during startup")
                assert addresses.coordinator_input is not None
                logger.info("Waiting for READY message from DP Coordinator...")
```
**EN:** This method initializes the object state within `EngineCoreProc`. Key calls include `instrument`, `to_bytes`, `put_nowait`, `TensorIpcReceiver`, `info`, `_perform_handshakes`. It touches state such as `input_queue`, `output_queue`, `engine_index`, `engines_running`, `shutdown_state`, `tensor_ipc_receiver`, `has_coordinator`, `frontend_stats_publish_address`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会初始化对象状态，其作用域位于`EngineCoreProc`。 关键调用包括 `instrument`, `to_bytes`, `put_nowait`, `TensorIpcReceiver`, `info`, `_perform_handshakes`。 它会读写 `input_queue`, `output_queue`, `engine_index`, `engines_running`, `shutdown_state`, `tensor_ipc_receiver`, `has_coordinator`, `frontend_stats_publish_address` 等状态。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `EngineCoreProc.run_engine_core` method / `EngineCoreProc.run_engine_core` 方法
```python
    @staticmethod
    def run_engine_core(*args, dp_rank: int = 0, local_dp_rank: int = 0, **kwargs):
        """Launch EngineCore busy loop in background process."""

        # Ensure we can serialize transformer config after spawning
        maybe_register_config_serialize_by_value()

        engine_core: EngineCoreProc | None = None
        signal_callback: SignalCallback | None = None
        try:
            vllm_config: VllmConfig = kwargs["vllm_config"]
            parallel_config: ParallelConfig = vllm_config.parallel_config
            data_parallel = parallel_config.data_parallel_size > 1 or dp_rank > 0
            if data_parallel:
                parallel_config.data_parallel_rank_local = local_dp_rank
                process_title = f"EngineCore_DP{dp_rank}"
            else:
                process_title = "EngineCore"
            set_process_title(process_title)
            maybe_init_worker_tracer("vllm.engine_core", "engine_core", process_title)
            decorate_logs()
            if parallel_config.numa_bind:
                numa_utils.log_current_affinity_state(process_title)

            if data_parallel and vllm_config.kv_transfer_config is not None:
                # modify the engine_id and append the local_dp_rank to it to ensure
                # that the kv_transfer_config is unique for each DP rank.
                vllm_config.kv_transfer_config.engine_id = (
                    f"{vllm_config.kv_transfer_config.engine_id}_dp{local_dp_rank}"
                )
                logger.debug(
                    "Setting kv_transfer_config.engine_id to %s",
                    vllm_config.kv_transfer_config.engine_id,
                )

            parallel_config.data_parallel_index = dp_rank
            if data_parallel and vllm_config.model_config.is_moe:
                # Set data parallel rank for this engine process.
                parallel_config.data_parallel_rank = dp_rank
                engine_core = DPEngineCoreProc(*args, **kwargs)
            else:
                # Non-MoE DP ranks are completely independent, so treat like DP=1.
                # Note that parallel_config.data_parallel_index will still reflect
                # the original DP rank.
                parallel_config.data_parallel_size = 1
                parallel_config.data_parallel_size_local = 1
                parallel_config.data_parallel_rank = 0
                engine_core = EngineCoreProc(*args, engine_index=dp_rank, **kwargs)

            assert engine_core is not None

            def wakeup_engine():
                # Wakes up idle engine via input_queue when shutdown is requested
                # Not safe in a signal handler - we may interrupt the main thread
                # while it is holding the non-reentrant input_queue.mutex
                engine_core.input_queue.put_nowait((EngineCoreRequestType.WAKEUP, None))

            signal_callback = SignalCallback(wakeup_engine)

            def signal_handler(signum, frame):
                engine_core.shutdown_state = EngineShutdownState.REQUESTED
                signal_callback.trigger()

            signal.signal(signal.SIGTERM, signal_handler)
            signal.signal(signal.SIGINT, signal_handler)

            engine_core.run_busy_loop()

        except SystemExit:
            logger.debug("EngineCore exiting.")
            raise
        except Exception as e:
            if engine_core is None:
                logger.exception("EngineCore failed to start.")
            else:
                logger.exception("EngineCore encountered a fatal error.")
                engine_core._send_engine_dead()
            raise e
        finally:
            signal.signal(signal.SIGTERM, signal.SIG_DFL)
            signal.signal(signal.SIGINT, signal.SIG_DFL)
            if signal_callback is not None:
                signal_callback.stop()
            if engine_core is not None:
                engine_core.shutdown()
```
**EN:** This method runs the main execution path within `EngineCoreProc`. The docstring frames it as: Launch EngineCore busy loop in background process. Key calls include `maybe_register_config_serialize_by_value`, `set_process_title`, `maybe_init_worker_tracer`, `decorate_logs`, `SignalCallback`, `signal`. The control flow contains 8 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会执行主要运行路径，其作用域位于`EngineCoreProc`。 关键调用包括 `maybe_register_config_serialize_by_value`, `set_process_title`, `maybe_init_worker_tracer`, `decorate_logs`, `SignalCallback`, `signal`。 控制流包含 8 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EngineCoreProc.is_running` method / `EngineCoreProc.is_running` 方法
```python
    def is_running(self) -> bool:
        """Returns true if shutdown has not been requested."""
        return self.shutdown_state == EngineShutdownState.RUNNING
```
**EN:** This method answers a boolean capability check within `EngineCoreProc`. The docstring frames it as: Returns true if shutdown has not been requested.
**CN:** 该方法会回答布尔能力判断，其作用域位于`EngineCoreProc`。

### `EngineCoreProc.run_busy_loop` method / `EngineCoreProc.run_busy_loop` 方法
```python
    def run_busy_loop(self):
        """Core busy loop of the EngineCore."""
        while self._handle_shutdown():
            # 1) Poll the input queue until there is work to do.
            self._process_input_queue()
            # 2) Step the engine core and return the outputs.
            self._process_engine_step()

        raise SystemExit
```
**EN:** This method runs the main execution path within `EngineCoreProc`. The docstring frames it as: Core busy loop of the EngineCore. Key calls include `_handle_shutdown`, `_process_input_queue`, `_process_engine_step`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会执行主要运行路径，其作用域位于`EngineCoreProc`。 关键调用包括 `_handle_shutdown`, `_process_input_queue`, `_process_engine_step`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `EngineCoreProc.pause_scheduler` method / `EngineCoreProc.pause_scheduler` 方法
```python
    def pause_scheduler(
        self, mode: PauseMode = "abort", clear_cache: bool = True
    ) -> Future | None:
        """Pause generation; behavior depends on mode.

        All pause modes queue new adds -- "abort" and "keep" skip step();
        "wait" allows step() so in-flight requests can drain.

        - ``abort``: Set PAUSED_NEW, abort all requests, wait for abort
          outputs to be sent (when running with output_queue), optionally
          clear caches, then complete the returned Future.
        - ``wait``: Set PAUSED_NEW (queue adds, keep stepping); when drained,
          optionally clear caches, then complete the returned Future.
        - ``keep``: Set PAUSED_ALL; return a Future that completes when the
          output queue is empty.
        """
        if mode not in ("keep", "abort", "wait"):
            raise ValueError(f"Invalid pause mode: {mode}")

        def engine_idle_callback(engine: "EngineCoreProc", future: Future[Any]) -> None:
            if clear_cache:
                engine._reset_caches()
            future.set_result(None)

        if mode == "abort":
            aborted_reqs = self.scheduler.finish_requests(
                None, RequestStatus.FINISHED_ABORTED
            )
            self._send_abort_outputs(aborted_reqs)

        pause_state = PauseState.PAUSED_ALL if mode == "keep" else PauseState.PAUSED_NEW
        self.scheduler.set_pause_state(pause_state)

        if self._pause_complete():
            if clear_cache:
                self._reset_caches()
            return None

        future = Future[Any]()
        self._idle_state_callbacks.append(partial(engine_idle_callback, future=future))
        return future
```
**EN:** This method implements `pause_scheduler` within `EngineCoreProc`. The docstring frames it as: Pause generation; behavior depends on mode. Key calls include `set_pause_state`, `_pause_complete`, `append`, `ValueError`, `set_result`, `finish_requests`. The control flow contains 6 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `pause_scheduler`，其作用域位于`EngineCoreProc`。 关键调用包括 `set_pause_state`, `_pause_complete`, `append`, `ValueError`, `set_result`, `finish_requests`。 控制流包含 6 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DPEngineCoreProc` class / `DPEngineCoreProc` 类
```python
class DPEngineCoreProc(EngineCoreProc):
    """ZMQ-wrapper for running EngineCore in background process
    in a data parallel context."""
```
**EN:** Introduces the `DPEngineCoreProc` class on top of `EngineCoreProc`. Core methods include `__init__`, `_init_data_parallel`, `shutdown`, `_pause_complete`, `add_request`, `resume_scheduler`. Docstring signal: ZMQ-wrapper for running EngineCore in background process in a data parallel context.
**CN:** 这里定义 `DPEngineCoreProc` 类，其基类包括 `EngineCoreProc`。核心方法包括 `__init__`, `_init_data_parallel`, `shutdown`, `_pause_complete`, `add_request`, `resume_scheduler`。

### `DPEngineCoreProc.__init__` method / `DPEngineCoreProc.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        local_client: bool,
        handshake_address: str,
        executor_class: type[Executor],
        log_stats: bool,
        client_handshake_address: str | None = None,
        tensor_queue: Queue | None = None,
    ):
        assert vllm_config.model_config.is_moe, (
            "DPEngineCoreProc should only be used for MoE models"
        )

        # Counts forward-passes of the model so that we can synchronize
        # finished with DP peers every N steps.
        self.step_counter = 0
        self.current_wave = 0
        self.last_counts = (0, 0)

        # Two-phase pause protocol state. When pending_pause is True, the
        # engine keeps stepping (dummy batches) while waiting for all DP
        # ranks to also set pending_pause. Once all ranks agree via
        # all-reduce, ignore_start_dp_wave is set so that stale
        # START_DP_WAVE messages cannot re-wake the engines.
        self.pending_pause = False
        self.ignore_start_dp_wave = False

        from vllm.distributed.elastic_ep.elastic_state import ElasticEPScalingState

        self.eep_scaling_state: ElasticEPScalingState | None = None

        # Initialize the engine.
        dp_rank = vllm_config.parallel_config.data_parallel_rank
        super().__init__(
            vllm_config,
            local_client,
            handshake_address,
            executor_class,
            log_stats,
            client_handshake_address,
            engine_index=dp_rank,
            tensor_queue=tensor_queue,
        )
```
**EN:** This method initializes the object state within `DPEngineCoreProc`. Key calls include `__init__`, `super`. It touches state such as `step_counter`, `current_wave`, `last_counts`, `pending_pause`, `ignore_start_dp_wave`, `eep_scaling_state`.
**CN:** 该方法会初始化对象状态，其作用域位于`DPEngineCoreProc`。 关键调用包括 `__init__`, `super`。 它会读写 `step_counter`, `current_wave`, `last_counts`, `pending_pause`, `ignore_start_dp_wave`, `eep_scaling_state` 等状态。

### `DPEngineCoreProc.resume_scheduler` method / `DPEngineCoreProc.resume_scheduler` 方法
```python
    def resume_scheduler(self):
        if self.pending_pause or (self.engines_running and self.ignore_start_dp_wave):
            raise RuntimeError(
                "resume_scheduler called while pause is still in "
                "flight. Wait for the pause future to resolve before "
                "resuming."
            )
        if self.engines_running:
            logger.debug("Resume called while engines are not paused, ignoring.")
            return

        super().resume_scheduler()
        self.ignore_start_dp_wave = False

        # Barrier: wait for all DP ranks to have resumed (and cleared
        # ignore_start_dp_wave) before any rank starts stepping. Uses
        # the existing all-reduce which is safe because engines are
        # stopped.
        has_global_unfinished = ParallelConfig.has_unfinished_dp(
            self.dp_group, self.scheduler.has_unfinished_requests()
        )

        if has_global_unfinished:
            self.engines_running = True
```
**EN:** This method implements `resume_scheduler` within `DPEngineCoreProc`. Key calls include `resume_scheduler`, `has_unfinished_dp`, `RuntimeError`, `debug`, `has_unfinished_requests`, `super`. It touches state such as `ignore_start_dp_wave`, `engines_running`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `resume_scheduler`，其作用域位于`DPEngineCoreProc`。 关键调用包括 `resume_scheduler`, `has_unfinished_dp`, `RuntimeError`, `debug`, `has_unfinished_requests`, `super`。 它会读写 `ignore_start_dp_wave`, `engines_running` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DPEngineCoreProc.run_busy_loop` method / `DPEngineCoreProc.run_busy_loop` 方法
```python
    def run_busy_loop(self):
        """Core busy loop of the EngineCore for data parallel case."""

        # Loop until process is sent a SIGINT or SIGTERM
        while self._handle_shutdown():
            # 1) Poll the input queue until there is work to do.
            self._process_input_queue()
            # Publish request counts before and after GPU step to ensure freshness.
            self._maybe_publish_request_counts()

            if self.eep_scaling_state is not None:
                _ = self.eep_scaling_state.progress()
                if self.eep_scaling_state.is_complete():
                    if self.eep_scaling_state.worker_type == "removing":
                        raise SystemExit
                    self.process_input_queue_block = True
                    self.eep_scaling_state = None

            executed = self._process_engine_step()
            self._maybe_publish_request_counts()

            local_unfinished_reqs = self.scheduler.has_unfinished_requests()
            if not executed:
                if not local_unfinished_reqs and not self.engines_running:
                    # All engines are idle.
                    continue

                # We are in a running state and so must execute a dummy pass
                # if the model didn't execute any ready requests.
                self.execute_dummy_batch()

            # 3) All-reduce operation to determine global unfinished reqs.
            self.engines_running = self._has_global_unfinished_reqs(
                local_unfinished_reqs
            )

            if not self.engines_running:
                if self.dp_rank == 0 or not self.has_coordinator:
                    # Notify client that we are pausing the loop.
                    logger.debug(
                        "Wave %d finished, pausing engine loop.", self.current_wave
                    )
                    # In the coordinator case, dp rank 0 sends updates to the
                    # coordinator. Otherwise (offline spmd case), each rank
                    # sends the update to its colocated front-end process.
                    client_index = -1 if self.has_coordinator else 0
                    self.output_queue.put_nowait(
                        (
                            client_index,
                            EngineCoreOutputs(wave_complete=self.current_wave),
                        )
                    )
                # Increment wave count and reset step counter.
                self.current_wave += 1
                self.step_counter = 0

        raise SystemExit
```
**EN:** This method runs the main execution path within `DPEngineCoreProc`. The docstring frames it as: Core busy loop of the EngineCore for data parallel case. Key calls include `_handle_shutdown`, `_process_input_queue`, `_maybe_publish_request_counts`, `_process_engine_step`, `has_unfinished_requests`, `_has_global_unfinished_reqs`. It touches state such as `engines_running`, `current_wave`, `step_counter`, `process_input_queue_block`, `eep_scaling_state`. The control flow contains 8 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会执行主要运行路径，其作用域位于`DPEngineCoreProc`。 关键调用包括 `_handle_shutdown`, `_process_input_queue`, `_maybe_publish_request_counts`, `_process_engine_step`, `has_unfinished_requests`, `_has_global_unfinished_reqs`。 它会读写 `engines_running`, `current_wave`, `step_counter`, `process_input_queue_block`, `eep_scaling_state` 等状态。 控制流包含 8 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `DPEngineCoreProc.reinitialize_distributed` method / `DPEngineCoreProc.reinitialize_distributed` 方法
```python
    def reinitialize_distributed(
        self, reconfig_request: ReconfigureDistributedRequest
    ) -> None:
        from copy import deepcopy

        from vllm.distributed.elastic_ep.elastic_state import ElasticEPScalingState

        new_parallel_config = deepcopy(self.vllm_config.parallel_config)
        old_dp_size = new_parallel_config.data_parallel_size
        new_parallel_config.data_parallel_size = reconfig_request.new_data_parallel_size
        if (
            reconfig_request.new_data_parallel_rank
            != ReconfigureRankType.KEEP_CURRENT_RANK
        ):
            new_parallel_config.data_parallel_rank = (
                reconfig_request.new_data_parallel_rank
            )
        new_parallel_config.data_parallel_master_ip = (
            reconfig_request.new_data_parallel_master_ip
        )
        new_parallel_config.data_parallel_master_port = (
            reconfig_request.new_data_parallel_master_port
        )
        new_parallel_config._data_parallel_master_port_list = (
            reconfig_request.new_data_parallel_master_port_list
        )
        new_parallel_config._coord_store_port = reconfig_request.coord_store_port

        is_scale_down = reconfig_request.new_data_parallel_size < old_dp_size
        is_shutdown = (
            reconfig_request.new_data_parallel_rank
            == ReconfigureRankType.SHUTDOWN_CURRENT_RANK
        )

        self.eep_scaling_state = ElasticEPScalingState(
            model_executor=self.model_executor,
            engine_core=self,
            vllm_config=self.vllm_config,
            new_parallel_config=new_parallel_config,
            worker_type="removing" if is_shutdown else "existing",
            scale_type="scale_down" if is_scale_down else "scale_up",
            reconfig_request=reconfig_request,
        )
        self.process_input_queue_block = False
        logger.info(
            "[Elastic EP] Received reconfiguration request and starting scaling up/down"
        )
```
**EN:** This method implements `reinitialize_distributed` within `DPEngineCoreProc`. Key calls include `deepcopy`, `ElasticEPScalingState`, `info`. It touches state such as `eep_scaling_state`, `process_input_queue_block`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `reinitialize_distributed`，其作用域位于`DPEngineCoreProc`。 关键调用包括 `deepcopy`, `ElasticEPScalingState`, `info`。 它会读写 `eep_scaling_state`, `process_input_queue_block` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DPEngineCoreProc._eep_send_engine_core_notification` method / `DPEngineCoreProc._eep_send_engine_core_notification` 方法
```python
    def _eep_send_engine_core_notification(
        self,
        notification_type: EEPNotificationType,
        vllm_config: VllmConfig | None = None,
    ):
        """
        Send notifications to EngineCoreClient, which can then forward
        the notifications to other engine core processes. It is used for:
        1) In scale up: new core engines to notify existing core engines
           that they are ready;
        2) In scale down: removing core engines to notify EngineCoreClient
           so EngineCoreClient can release their ray placement groups;
        3) Both scale up/down: to notify EngineCoreClient that existing
           core engines have already switched to the new parallel setup.
        """
        if vllm_config is None:
            dp_rank = self.vllm_config.parallel_config.data_parallel_rank
        else:
            dp_rank = vllm_config.parallel_config.data_parallel_rank
        notification_data = (notification_type.value, dp_rank)
        outputs = EngineCoreOutputs(
            utility_output=UtilityOutput(
                call_id=EEP_NOTIFICATION_CALL_ID,
                result=UtilityResult(notification_data),
            )
        )
        outputs.engine_index = self.engine_index

        if hasattr(self, "output_thread") and self.output_thread.is_alive():
            self.output_queue.put_nowait((0, outputs))
        else:
            encoder = MsgpackEncoder()
            with (
                zmq.Context() as ctx,
                make_zmq_socket(
                    ctx, self.addresses.outputs[0], zmq.PUSH, linger=4000
                ) as socket,
            ):
                socket.send_multipart(encoder.encode(outputs))
```
**EN:** This method implements `_eep_send_engine_core_notification` within `DPEngineCoreProc`. The docstring frames it as: Send notifications to EngineCoreClient, which can then forward the notifications to other engine core processes. Key calls include `EngineCoreOutputs`, `hasattr`, `is_alive`, `put_nowait`, `MsgpackEncoder`, `UtilityOutput`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_eep_send_engine_core_notification`，其作用域位于`DPEngineCoreProc`。 关键调用包括 `EngineCoreOutputs`, `hasattr`, `is_alive`, `put_nowait`, `MsgpackEncoder`, `UtilityOutput`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EngineCoreActorMixin` class / `EngineCoreActorMixin` 类
```python
class EngineCoreActorMixin:
    """
    Ray actor for running EngineCore in a data parallel context
    """
```
**EN:** Introduces the `EngineCoreActorMixin` class. Core methods include `__init__`, `_set_nixl_side_channel_host`, `_set_visible_devices`, `_set_cuda_visible_devices`, `_perform_handshakes`, `wait_for_init`. Docstring signal: Ray actor for running EngineCore in a data parallel context
**CN:** 这里定义 `EngineCoreActorMixin` 类。核心方法包括 `__init__`, `_set_nixl_side_channel_host`, `_set_visible_devices`, `_set_cuda_visible_devices`, `_perform_handshakes`, `wait_for_init`。

### `EngineCoreActorMixin.__init__` method / `EngineCoreActorMixin.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        addresses: EngineZmqAddresses,
        dp_rank: int = 0,
        local_dp_rank: int = 0,
    ):
        # Initialize tracer for distributed tracing if configured.
        maybe_init_worker_tracer(
            instrumenting_module_name="vllm.engine_core",
            process_kind="engine_core",
            process_name=f"DPEngineCoreActor_DP{dp_rank}",
        )

        self.addresses = addresses
        vllm_config.parallel_config.data_parallel_index = dp_rank
        vllm_config.parallel_config.data_parallel_rank_local = local_dp_rank

        self._set_nixl_side_channel_host()

        # Set CUDA_VISIBLE_DEVICES as early as possible in actor life cycle
        # NOTE: in MP we set CUDA_VISIBLE_DEVICES at process creation time,
        # and this cannot be done in the same way for Ray because:
        # 1) Ray manages life cycle of all ray workers (including
        # DPEngineCoreActor)
        # 2) Ray sets CUDA_VISIBLE_DEVICES based on num_gpus configuration
        # To bypass 2, we need to also set
        # RAY_EXPERIMENTAL_NOSET_CUDA_VISIBLE_DEVICES, but vLLM workers created
        # thereafter would have CUDA_VISIBLE_DEVICES set, which is sticky:
        # https://github.com/ray-project/ray/blob/e752fc319ddedd9779a0989b6d3613909bad75c9/python/ray/_private/worker.py#L456 # noqa: E501
        # This is problematic because when the vLLM worker (a Ray actor)
        # executes a task, it indexes into the sticky CUDA_VISIBLE_DEVICES
        # rather than directly using the GPU ID, potentially resulting in
        # index out of bounds error. See:
        # https://github.com/ray-project/ray/pull/40461/files#diff-31e8159767361e4bc259b6d9883d9c0d5e5db780fcea4a52ead4ee3ee4a59a78R1860 # noqa: E501
        # and get_accelerator_ids_for_accelerator_resource() in worker.py
        # of ray.
        self._set_visible_devices(vllm_config, local_dp_rank)
```
**EN:** This method initializes the object state within `EngineCoreActorMixin`. Key calls include `maybe_init_worker_tracer`, `_set_nixl_side_channel_host`, `_set_visible_devices`. It touches state such as `addresses`.
**CN:** 该方法会初始化对象状态，其作用域位于`EngineCoreActorMixin`。 关键调用包括 `maybe_init_worker_tracer`, `_set_nixl_side_channel_host`, `_set_visible_devices`。 它会读写 `addresses` 等状态。

### `EngineCoreActorMixin._set_cuda_visible_devices` method / `EngineCoreActorMixin._set_cuda_visible_devices` 方法
```python
    def _set_cuda_visible_devices(
        self, vllm_config: VllmConfig, local_dp_rank: int, device_control_env_var: str
    ):
        world_size = vllm_config.parallel_config.world_size
        # Set CUDA_VISIBLE_DEVICES or equivalent.
        try:
            value = get_device_indices(
                device_control_env_var, local_dp_rank, world_size
            )
            os.environ[device_control_env_var] = value
        except IndexError as e:
            raise Exception(
                f"Error setting {device_control_env_var}: "
                f"local range: [{local_dp_rank * world_size}, "
                f"{(local_dp_rank + 1) * world_size}) "
                f'base value: "{os.getenv(device_control_env_var)}"'
            ) from e
```
**EN:** This method implements `_set_cuda_visible_devices` within `EngineCoreActorMixin`. Key calls include `get_device_indices`, `Exception`, `getenv`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_set_cuda_visible_devices`，其作用域位于`EngineCoreActorMixin`。 关键调用包括 `get_device_indices`, `Exception`, `getenv`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EngineCoreActorMixin._perform_handshakes` method / `EngineCoreActorMixin._perform_handshakes` 方法
```python
    @contextmanager
    def _perform_handshakes(
        self,
        handshake_address: str,
        identity: bytes,
        local_client: bool,
        vllm_config: VllmConfig,
        client_handshake_address: str | None,
    ):
        """
        For Ray, we don't need to actually perform handshake.
        All addresses information is known before the actor creation.
        Therefore, we simply yield these addresses.
        """
        yield self.addresses
```
**EN:** This method implements `_perform_handshakes` within `EngineCoreActorMixin`. The docstring frames it as: For Ray, we don't need to actually perform handshake.
**CN:** 该方法会实现 `_perform_handshakes`，其作用域位于`EngineCoreActorMixin`。

### `EngineCoreActorMixin.wait_for_init` method / `EngineCoreActorMixin.wait_for_init` 方法
```python
    def wait_for_init(self):
        """
        Wait until the engine core is initialized.

        This is just an empty method. When ray.get() on this method
        (or any other method of the actor) returns, it is guaranteed
        that actor creation (i.e., __init__) is complete.
        """
        pass
```
**EN:** This method implements `wait_for_init` within `EngineCoreActorMixin`. The docstring frames it as: Wait until the engine core is initialized.
**CN:** 该方法会实现 `wait_for_init`，其作用域位于`EngineCoreActorMixin`。

### `EngineCoreActorMixin.run` method / `EngineCoreActorMixin.run` 方法
```python
    def run(self):
        """
        Run the engine core busy loop.
        """
        try:
            self.run_busy_loop()  # type: ignore[attr-defined]
        except SystemExit:
            logger.debug("EngineCore exiting.")
            raise
        except Exception:
            logger.exception("EngineCore encountered a fatal error.")
            raise
        finally:
            self.shutdown()  # type: ignore[attr-defined]
```
**EN:** This method runs the main execution path within `EngineCoreActorMixin`. The docstring frames it as: Run the engine core busy loop. Key calls include `run_busy_loop`, `shutdown`, `debug`, `exception`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会执行主要运行路径，其作用域位于`EngineCoreActorMixin`。 关键调用包括 `run_busy_loop`, `shutdown`, `debug`, `exception`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DPMoEEngineCoreActor` class / `DPMoEEngineCoreActor` 类
```python
class DPMoEEngineCoreActor(EngineCoreActorMixin, DPEngineCoreProc):
    """Used for MoE model data parallel cases."""
```
**EN:** Introduces the `DPMoEEngineCoreActor` class on top of `EngineCoreActorMixin`, `DPEngineCoreProc`. Core methods include `__init__`. Docstring signal: Used for MoE model data parallel cases.
**CN:** 这里定义 `DPMoEEngineCoreActor` 类，其基类包括 `EngineCoreActorMixin`, `DPEngineCoreProc`。核心方法包括 `__init__`。

### `DPMoEEngineCoreActor.__init__` method / `DPMoEEngineCoreActor.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        local_client: bool,
        addresses: EngineZmqAddresses,
        executor_class: type[Executor],
        log_stats: bool,
        dp_rank: int = 0,
        local_dp_rank: int = 0,
    ):
        vllm_config.parallel_config.data_parallel_rank = dp_rank

        EngineCoreActorMixin.__init__(
            self, vllm_config, addresses, dp_rank, local_dp_rank
        )
        DPEngineCoreProc.__init__(
            self, vllm_config, local_client, "", executor_class, log_stats
        )
```
**EN:** This method initializes the object state within `DPMoEEngineCoreActor`. Key calls include `__init__`.
**CN:** 该方法会初始化对象状态，其作用域位于`DPMoEEngineCoreActor`。 关键调用包括 `__init__`。

### `EngineCoreActor` class / `EngineCoreActor` 类
```python
class EngineCoreActor(EngineCoreActorMixin, EngineCoreProc):
    """Used for non-MoE and/or non-DP cases."""
```
**EN:** Introduces the `EngineCoreActor` class on top of `EngineCoreActorMixin`, `EngineCoreProc`. Core methods include `__init__`. Docstring signal: Used for non-MoE and/or non-DP cases.
**CN:** 这里定义 `EngineCoreActor` 类，其基类包括 `EngineCoreActorMixin`, `EngineCoreProc`。核心方法包括 `__init__`。

### `EngineCoreActor.__init__` method / `EngineCoreActor.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        local_client: bool,
        addresses: EngineZmqAddresses,
        executor_class: type[Executor],
        log_stats: bool,
        dp_rank: int = 0,
        local_dp_rank: int = 0,
    ):
        vllm_config.parallel_config.data_parallel_size = 1
        vllm_config.parallel_config.data_parallel_size_local = 1
        vllm_config.parallel_config.data_parallel_rank = 0

        EngineCoreActorMixin.__init__(
            self, vllm_config, addresses, dp_rank, local_dp_rank
        )
        EngineCoreProc.__init__(
            self,
            vllm_config,
            local_client,
            "",
            executor_class,
            log_stats,
            engine_index=dp_rank,
        )
```
**EN:** This method initializes the object state within `EngineCoreActor`. Key calls include `__init__`.
**CN:** 该方法会初始化对象状态，其作用域位于`EngineCoreActor`。 关键调用包括 `__init__`。

## Key Concepts / 关键概念
- `EngineCore`: central class or interface in this module. / `EngineCore`：本模块中的核心类或接口。
- `EngineShutdownState`: central class or interface in this module. / `EngineShutdownState`：本模块中的核心类或接口。
- `EngineCoreProc`: central class or interface in this module. / `EngineCoreProc`：本模块中的核心类或接口。
- `DPEngineCoreProc`: central class or interface in this module. / `DPEngineCoreProc`：本模块中的核心类或接口。
- `EngineCoreActorMixin`: central class or interface in this module. / `EngineCoreActorMixin`：本模块中的核心类或接口。
- `DPMoEEngineCoreActor`: central class or interface in this module. / `DPMoEEngineCoreActor`：本模块中的核心类或接口。
- `EngineCoreActor`: central class or interface in this module. / `EngineCoreActor`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `gc`, `os`, `queue`, `signal`, `threading`, `time`, `collections`, `concurrent`, `contextlib`, `enum`, `functools`, `inspect`
- External / 外部依赖: `msgspec`, `zmq`, `torch`, `ray`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.config`, `vllm.distributed`, `vllm.logger`, `vllm.logging_utils.dump_input`, `vllm.lora.request`, `vllm.multimodal`, `vllm.tasks`, `vllm.tracing`, `vllm.transformers_utils.config`, `vllm.utils`, `vllm.utils.gc_utils`
