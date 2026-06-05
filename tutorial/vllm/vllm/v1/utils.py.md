# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `ConstantList`, `CpuGpuBuffer`, `get_engine_client_zmq_addr` for the V1 `v1` subsystem. / 为 V1 的 `v1` 子系统实现 `ConstantList`, `CpuGpuBuffer`, `get_engine_client_zmq_addr`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import argparse
import contextlib
import multiprocessing
import threading
import time
import weakref
from collections.abc import Callable, Sequence
from contextlib import AbstractContextManager
from dataclasses import dataclass
from multiprocessing import connection
from multiprocessing.process import BaseProcess
from multiprocessing.queues import Queue
from typing import (
    TYPE_CHECKING,
    Any,
    Generic,
    TypeVar,
    Union,
    overload,
)

import torch
import uvloop
from torch.autograd.profiler import record_function

import vllm.envs as envs
from vllm.logger import init_logger
from vllm.usage.usage_lib import UsageContext, is_usage_stats_enabled, usage_message
from vllm.utils.network_utils import get_open_port, get_open_zmq_ipc_path, get_tcp_uri
from vllm.utils.system_utils import decorate_logs, kill_process_tree, set_process_title
from vllm.v1.core.sched.output import SchedulerOutput

if TYPE_CHECKING:
    import numpy as np

    from vllm.v1.engine.coordinator import DPCoordinator
    from vllm.v1.engine.utils import CoreEngineActorManager, CoreEngineProcManager

logger = init_logger(__name__)

T = TypeVar("T")
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `T`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `T`。

### `ConstantList` class / `ConstantList` 类
```python
class ConstantList(Generic[T], Sequence):
```
**EN:** Introduces the `ConstantList` class on top of `Generic[T]`, `Sequence`. Core methods include `__init__`, `append`, `extend`, `insert`, `pop`, `remove`.
**CN:** 这里定义 `ConstantList` 类，其基类包括 `Generic[T]`, `Sequence`。核心方法包括 `__init__`, `append`, `extend`, `insert`, `pop`, `remove`。

### `ConstantList.__init__` method / `ConstantList.__init__` 方法
```python
    def __init__(self, x: list[T]) -> None:
        self._x = x
```
**EN:** This method initializes the object state within `ConstantList`. It touches state such as `_x`.
**CN:** 该方法会初始化对象状态，其作用域位于`ConstantList`。 它会读写 `_x` 等状态。

### `ConstantList.append` method / `ConstantList.append` 方法
```python
    def append(self, item):
        raise TypeError("Cannot append to a constant list")
```
**EN:** This method implements `append` within `ConstantList`. Key calls include `TypeError`.
**CN:** 该方法会实现 `append`，其作用域位于`ConstantList`。 关键调用包括 `TypeError`。

### `ConstantList.extend` method / `ConstantList.extend` 方法
```python
    def extend(self, item):
        raise TypeError("Cannot extend a constant list")
```
**EN:** This method implements `extend` within `ConstantList`. Key calls include `TypeError`.
**CN:** 该方法会实现 `extend`，其作用域位于`ConstantList`。 关键调用包括 `TypeError`。

### `CpuGpuBuffer` class / `CpuGpuBuffer` 类
```python
class CpuGpuBuffer:
    """Buffer to easily copy tensors between CPU and GPU."""
```
**EN:** Introduces the `CpuGpuBuffer` class. Core methods include `__init__`, `copy_to_gpu`, `copy_to_cpu`. Docstring signal: Buffer to easily copy tensors between CPU and GPU.
**CN:** 这里定义 `CpuGpuBuffer` 类。核心方法包括 `__init__`, `copy_to_gpu`, `copy_to_cpu`。

### `CpuGpuBuffer.__init__` method / `CpuGpuBuffer.__init__` 方法
```python
    def __init__(
        self,
        *size: int | torch.SymInt,
        dtype: torch.dtype,
        device: torch.device,
        pin_memory: bool,
        with_numpy: bool = True,
    ) -> None:
        self.cpu = torch.zeros(*size, dtype=dtype, device="cpu", pin_memory=pin_memory)
        self.gpu = torch.zeros_like(self.cpu, device=device)
        self.np: np.ndarray
        # To keep type hints simple (avoiding generics and subclasses), we
        # only conditionally create the numpy array attribute. This can cause
        # AttributeError if `self.np` is accessed when `with_numpy=False`.
        if with_numpy:
            if dtype == torch.bfloat16:
                raise ValueError(
                    "Bfloat16 torch tensors cannot be directly cast to a "
                    "numpy array, so call CpuGpuBuffer with with_numpy=False"
                )
            self.np = self.cpu.numpy()
```
**EN:** This method initializes the object state within `CpuGpuBuffer`. Key calls include `zeros`, `zeros_like`, `numpy`, `ValueError`. It touches state such as `cpu`, `gpu`, `np`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`CpuGpuBuffer`。 关键调用包括 `zeros`, `zeros_like`, `numpy`, `ValueError`。 它会读写 `cpu`, `gpu`, `np` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `CpuGpuBuffer.copy_to_gpu` method / `CpuGpuBuffer.copy_to_gpu` 方法
```python
    def copy_to_gpu(self, n: int | None = None) -> torch.Tensor:
        if n is None:
            return self.gpu.copy_(self.cpu, non_blocking=True)
        return self.gpu[:n].copy_(self.cpu[:n], non_blocking=True)
```
**EN:** This method implements `copy_to_gpu` within `CpuGpuBuffer`. Key calls include `copy_`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `copy_to_gpu`，其作用域位于`CpuGpuBuffer`。 关键调用包括 `copy_`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `CpuGpuBuffer.copy_to_cpu` method / `CpuGpuBuffer.copy_to_cpu` 方法
```python
    def copy_to_cpu(self, n: int | None = None) -> torch.Tensor:
        """NOTE: Because this method is non-blocking, explicit synchronization
        is needed to ensure the data is copied to CPU."""
        if n is None:
            return self.cpu.copy_(self.gpu, non_blocking=True)
        return self.cpu[:n].copy_(self.gpu[:n], non_blocking=True)
```
**EN:** This method implements `copy_to_cpu` within `CpuGpuBuffer`. The docstring frames it as: NOTE: Because this method is non-blocking, explicit synchronization is needed to ensure the data is copied to CPU. Key calls include `copy_`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `copy_to_cpu`，其作用域位于`CpuGpuBuffer`。 关键调用包括 `copy_`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `get_engine_client_zmq_addr` function / `get_engine_client_zmq_addr` 函数
```python
def get_engine_client_zmq_addr(local_only: bool, host: str, port: int = 0) -> str:
    """Assign a new ZMQ socket address.

    If local_only is True, participants are colocated and so a unique IPC
    address will be returned.

    Otherwise, the provided host and port will be used to construct a TCP
    address (port == 0 means assign an available port)."""

    return (
        get_open_zmq_ipc_path()
        if local_only
        else (get_tcp_uri(host, port or get_open_port()))
    )
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Assign a new ZMQ socket address. Key calls include `get_open_zmq_ipc_path`, `get_tcp_uri`, `get_open_port`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `get_open_zmq_ipc_path`, `get_tcp_uri`, `get_open_port`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `APIServerProcessManager` class / `APIServerProcessManager` 类
```python
class APIServerProcessManager:
    """Manages a group of API server processes.

    Handles creation, monitoring, and termination of API server worker
    processes. Also monitors extra processes to check if they are healthy.
    """
```
**EN:** Introduces the `APIServerProcessManager` class. Core methods include `__init__`, `shutdown`. Docstring signal: Manages a group of API server processes.
**CN:** 这里定义 `APIServerProcessManager` 类。核心方法包括 `__init__`, `shutdown`。

### `APIServerProcessManager.__init__` method / `APIServerProcessManager.__init__` 方法
```python
    def __init__(
        self,
        listen_address: str,
        sock: Any,
        args: argparse.Namespace,
        num_servers: int,
        input_addresses: list[str],
        output_addresses: list[str],
        target_server_fn: Callable | None = None,
        stats_update_address: str | None = None,
        tensor_queue: Queue | None = None,
    ):
        """Initialize and start API server worker processes.

        Args:
            target_server_fn: Override function to call for each API server process
            listen_address: Address to listen for client connections
            sock: Socket for client connections
            args: Command line arguments
            num_servers: Number of API server processes to start
            input_addresses: Input addresses for each API server
            output_addresses: Output addresses for each API server
            stats_update_address: Optional stats update address
            tensor_queue: Optional tensor IPC queue for sharing MM tensors
        """
        self.listen_address = listen_address
        self.sock = sock
        self.args = args

        # Start API servers
        spawn_context = multiprocessing.get_context("spawn")
        self.processes: list[BaseProcess] = []

        for i, in_addr, out_addr in zip(
            range(num_servers), input_addresses, output_addresses
        ):
            client_config = {
                "input_address": in_addr,
                "output_address": out_addr,
                "client_count": num_servers,
                "client_index": i,
            }
            if stats_update_address is not None:
                client_config["stats_update_address"] = stats_update_address
            if tensor_queue is not None:
                client_config["tensor_queue"] = tensor_queue

            proc = spawn_context.Process(
                target=target_server_fn or run_api_server_worker_proc,
                name=f"ApiServer_{i}",
                args=(listen_address, sock, args, client_config),
            )
            self.processes.append(proc)
            proc.start()

        logger.info("Started %d API server processes", len(self.processes))

        # Shutdown only the API server processes on garbage collection
        # The extra processes are managed by their owners
        self._finalizer = weakref.finalize(self, shutdown, self.processes)
```
**EN:** This method initializes the object state within `APIServerProcessManager`. The docstring frames it as: Initialize and start API server worker processes. Key calls include `get_context`, `zip`, `info`, `finalize`, `range`, `Process`. It touches state such as `listen_address`, `sock`, `args`, `processes`, `_finalizer`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`APIServerProcessManager`。 关键调用包括 `get_context`, `zip`, `info`, `finalize`, `range`, `Process`。 它会读写 `listen_address`, `sock`, `args`, `processes`, `_finalizer` 等状态。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `APIServerProcessManager.shutdown` method / `APIServerProcessManager.shutdown` 方法
```python
    def shutdown(self, timeout: float | None = None) -> None:
        """Shutdown API server processes with configurable timeout"""
        if self._finalizer.detach() is not None:
            shutdown(self.processes, timeout=timeout)
```
**EN:** This method implements `shutdown` within `APIServerProcessManager`. The docstring frames it as: Shutdown API server processes with configurable timeout Key calls include `detach`, `shutdown`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `shutdown`，其作用域位于`APIServerProcessManager`。 关键调用包括 `detach`, `shutdown`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `run_api_server_worker_proc` function / `run_api_server_worker_proc` 函数
```python
def run_api_server_worker_proc(
    listen_address, sock, args, client_config=None, **uvicorn_kwargs
) -> None:
    """Entrypoint for individual API server worker processes."""

    from vllm.entrypoints.openai.api_server import run_server_worker

    client_config = client_config or {}
    server_index = client_config.get("client_index", 0)

    # Set process title and add process-specific prefix to stdout and stderr.
    set_process_title("APIServer", str(server_index))
    decorate_logs()

    uvloop.run(
        run_server_worker(listen_address, sock, args, client_config, **uvicorn_kwargs)
    )
```
**EN:** This function runs the main execution path within the module. The docstring frames it as: Entrypoint for individual API server worker processes. Key calls include `get`, `set_process_title`, `decorate_logs`, `run`, `str`, `run_server_worker`.
**CN:** 该函数会执行主要运行路径，其作用域位于the module。 关键调用包括 `get`, `set_process_title`, `decorate_logs`, `run`, `str`, `run_server_worker`。

### `wait_for_completion_or_failure` function / `wait_for_completion_or_failure` 函数
```python
def wait_for_completion_or_failure(
    api_server_manager: APIServerProcessManager,
    engine_manager: Union["CoreEngineProcManager", "CoreEngineActorManager"]
    | None = None,
    coordinator: "DPCoordinator | None" = None,
) -> None:
    """Wait for all processes to complete or detect if any fail.

    Raises an exception if any process exits with a non-zero status.

    Args:
        api_server_manager: The manager for API servers.
        engine_manager: The manager for engine processes.
            If CoreEngineProcManager, it manages local engines;
            if CoreEngineActorManager, it manages all engines.
        coordinator: The coordinator for data parallel.
    """

    try:
        logger.info("Waiting for API servers to complete ...")
        # Create a mapping of sentinels to their corresponding processes
        # for efficient lookup
        sentinel_to_proc: dict[Any, BaseProcess] = {
            proc.sentinel: proc for proc in api_server_manager.processes
        }

        if coordinator:
            sentinel_to_proc[coordinator.proc.sentinel] = coordinator.proc

        if engine_manager:
            core_shutdown_recv, core_shutdown_send = connection.Pipe(duplex=False)

            def monitor_engines():
                try:
                    engine_manager.monitor_engine_liveness()
                finally:
                    core_shutdown_send.close()
                    core_shutdown_recv.close()

            # start monitor for engine liveness
            threading.Thread(target=monitor_engines, daemon=True).start()
            sentinel_to_proc[core_shutdown_recv] = None  # type: ignore[assignment]

        # Check if any process terminates
        while sentinel_to_proc:
            # Wait for any process to terminate (or engine shutdown signal)
            ready_sentinels: list[Any] = connection.wait(sentinel_to_proc)

            # Process any terminated processes
            for sentinel in ready_sentinels:
                proc = sentinel_to_proc.pop(sentinel)

                # Check if process exited with error
                if proc is not None and proc.exitcode != 0:
                    raise RuntimeError(
                        f"Process {proc.name} (PID: {proc.pid}) "
                        f"died with exit code {proc.exitcode}"
                    )
                if engine_manager and engine_manager.failed_proc_name is not None:
                    raise RuntimeError(
                        f"Engine core process {engine_manager.failed_proc_name} "
                        "died unexpectedly."
                    )

    except KeyboardInterrupt:
        logger.info("Received KeyboardInterrupt, shutting down API servers...")
    except Exception as e:
        logger.exception("Exception occurred while running API servers: %s", str(e))
        raise
```
**EN:** This function implements `wait_for_completion_or_failure` within the module. The docstring frames it as: Wait for all processes to complete or detect if any fail. Key calls include `info`, `Pipe`, `start`, `wait`, `exception`, `pop`. The control flow contains 6 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `wait_for_completion_or_failure`，其作用域位于the module。 关键调用包括 `info`, `Pipe`, `start`, `wait`, `exception`, `pop`。 控制流包含 6 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `shutdown` function / `shutdown` 函数
```python
def shutdown(procs: list[BaseProcess], timeout: float | None = None) -> None:
    """Shutdown processes with timeout.

    Args:
        procs: List of processes to shutdown
        timeout: Maximum time in seconds to wait for graceful shutdown
    """
    if timeout is None:
        timeout = 0.0

    # Allow at least 5 seconds for remaining procs to terminate.
    timeout = max(timeout, 5.0)

    # Shutdown the process.
    for proc in procs:
        if proc.is_alive():
            proc.terminate()

    # Allow time for remaining procs to terminate.
    deadline = time.monotonic() + timeout
    for proc in procs:
        remaining = deadline - time.monotonic()
        if remaining <= 0:
            break
        if proc.is_alive():
            proc.join(remaining)

    for proc in procs:
        if proc.is_alive() and (pid := proc.pid) is not None:
            kill_process_tree(pid)
```
**EN:** This function implements `shutdown` within the module. The docstring frames it as: Shutdown processes with timeout. Key calls include `max`, `is_alive`, `monotonic`, `terminate`, `join`, `kill_process_tree`. The control flow contains 5 branch(es) and 3 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `shutdown`，其作用域位于the module。 关键调用包括 `max`, `is_alive`, `monotonic`, `terminate`, `join`, `kill_process_tree`。 控制流包含 5 个分支和 3 个循环，说明这里承担了较强的协调逻辑。

### `copy_slice` function / `copy_slice` 函数
```python
def copy_slice(
    from_tensor: torch.Tensor, to_tensor: torch.Tensor, length: int
) -> torch.Tensor:
    """
    Copy the first length elements of a tensor into another tensor in a
    non-blocking manner.

    Used to copy pinned CPU tensor data to pre-allocated GPU tensors.

    Returns the sliced target tensor.
    """
    return to_tensor[:length].copy_(from_tensor[:length], non_blocking=True)
```
**EN:** This function implements `copy_slice` within the module. The docstring frames it as: Copy the first length elements of a tensor into another tensor in a non-blocking manner. Key calls include `copy_`.
**CN:** 该函数会实现 `copy_slice`，其作用域位于the module。 关键调用包括 `copy_`。

### `report_usage_stats` function / `report_usage_stats` 函数
```python
def report_usage_stats(
    vllm_config, usage_context: UsageContext = UsageContext.ENGINE_CONTEXT
) -> None:
    """Report usage statistics if enabled."""

    if not is_usage_stats_enabled():
        return

    from vllm.model_executor.model_loader import get_architecture_class_name

    parallel_config = vllm_config.parallel_config

    # Prepare KV connector string if applicable
    kv_connector = None
    if vllm_config.kv_transfer_config is not None:
        kv_connector = vllm_config.kv_transfer_config.kv_connector

    usage_message.report_usage(
        get_architecture_class_name(vllm_config.model_config),
        usage_context,
        extra_kvs={
            # Common configuration
            "dtype": str(vllm_config.model_config.dtype),
            "block_size": vllm_config.cache_config.block_size,
            "gpu_memory_utilization": vllm_config.cache_config.gpu_memory_utilization,
            "kv_cache_memory_bytes": vllm_config.cache_config.kv_cache_memory_bytes,
            # Quantization
            "quantization": vllm_config.model_config.quantization,
            "kv_cache_dtype": str(vllm_config.cache_config.cache_dtype),
            # Feature flags
            "enable_lora": bool(vllm_config.lora_config),
            "enable_prefix_caching": vllm_config.cache_config.enable_prefix_caching,
            "enforce_eager": vllm_config.model_config.enforce_eager,
            "disable_custom_all_reduce": parallel_config.disable_custom_all_reduce,
            # Distributed parallelism settings
            "tensor_parallel_size": parallel_config.tensor_parallel_size,
            "data_parallel_size": parallel_config.data_parallel_size,
            "pipeline_parallel_size": parallel_config.pipeline_parallel_size,
            "enable_expert_parallel": parallel_config.enable_expert_parallel,
            # All2All backend for MoE expert parallel
            "all2all_backend": parallel_config.all2all_backend,
            # KV connector used
            "kv_connector": kv_connector,
        },
    )
```
**EN:** This function implements `report_usage_stats` within the module. The docstring frames it as: Report usage statistics if enabled. Key calls include `report_usage`, `is_usage_stats_enabled`, `get_architecture_class_name`, `str`, `bool`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `report_usage_stats`，其作用域位于the module。 关键调用包括 `report_usage`, `is_usage_stats_enabled`, `get_architecture_class_name`, `str`, `bool`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### Module constants / 模块常量
```python
_PROFILER_FUNC = None
```
**EN:** Defines module-level constants or aliases such as `_PROFILER_FUNC`, which are reused by later definitions.
**CN:** 定义 `_PROFILER_FUNC` 等模块级常量或别名，供后续定义复用。

### `record_function_or_nullcontext` function / `record_function_or_nullcontext` 函数
```python
def record_function_or_nullcontext(name: str) -> AbstractContextManager:
    global _PROFILER_FUNC

    # fast path assume it is set
    if _PROFILER_FUNC is not None:
        return _PROFILER_FUNC(name)

    func = contextlib.nullcontext
    if envs.VLLM_CUSTOM_SCOPES_FOR_PROFILING:
        func = record_function
    elif envs.VLLM_NVTX_SCOPES_FOR_PROFILING:
        import nvtx

        func = nvtx.annotate

    _PROFILER_FUNC = func
    return func(name)
```
**EN:** This function implements `record_function_or_nullcontext` within the module. Key calls include `func`, `_PROFILER_FUNC`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `record_function_or_nullcontext`，其作用域位于the module。 关键调用包括 `func`, `_PROFILER_FUNC`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `tensor_data` function / `tensor_data` 函数
```python
def tensor_data(tensor: torch.Tensor) -> memoryview:
    """Get the raw data of a tensor as a uint8 memoryview, useful for
    serializing and hashing.

    Args:
        tensor: The input tensor.

    Returns:
        A memoryview of the tensor data as uint8.
    """
    return tensor.flatten().cpu().contiguous().view(torch.uint8).numpy().data
```
**EN:** This function implements `tensor_data` within the module. The docstring frames it as: Get the raw data of a tensor as a uint8 memoryview, useful for serializing and hashing. Key calls include `numpy`, `view`, `contiguous`, `cpu`, `flatten`.
**CN:** 该函数会实现 `tensor_data`，其作用域位于the module。 关键调用包括 `numpy`, `view`, `contiguous`, `cpu`, `flatten`。

### `IterationDetails` class / `IterationDetails` 类
```python
@dataclass
class IterationDetails:
    num_ctx_requests: int
    num_ctx_tokens: int
    num_generation_requests: int
    num_generation_tokens: int
```
**EN:** Uses `@dataclass` to package related state for `IterationDetails`. Typical fields include `num_ctx_requests`, `num_ctx_tokens`, `num_generation_requests`, `num_generation_tokens`.
**CN:** `IterationDetails` 使用 `@dataclass` 打包相关状态。典型字段包括 `num_ctx_requests`, `num_ctx_tokens`, `num_generation_requests`, `num_generation_tokens`。

### `IterationDetails.__repr__` method / `IterationDetails.__repr__` 方法
```python
    def __repr__(self) -> str:
        return f"IterationDetails(num_ctx_requests={self.num_ctx_requests},\
                 num_ctx_tokens={self.num_ctx_tokens}, \
                 num_generation_requests={self.num_generation_requests}, \
                 num_generation_tokens={self.num_generation_tokens})"
```
**EN:** This method implements `__repr__` within `IterationDetails`.
**CN:** 该方法会实现 `__repr__`，其作用域位于`IterationDetails`。

### `compute_iteration_details` function / `compute_iteration_details` 函数
```python
def compute_iteration_details(scheduler_output: SchedulerOutput) -> IterationDetails:
    """
    Compute the number of context/generation requests and tokens
    for the current iteration's scheduler output. A requests is regarded
    as a context request if its output tokens are still 0, an extended chunk
    of chunked prefill falls into this category.

    Args:
        scheduler_output: The scheduler output for the current iteration.

    Returns:
        An IterationDetails object containing the number of
        context/generation requests and tokens.
    """
    num_context_requests = 0
    num_context_tokens = 0
    num_generation_requests = 0
    num_generation_tokens = 0
    new_req_ids = {new_req.req_id for new_req in scheduler_output.scheduled_new_reqs}
    for req_id, num_tokens in scheduler_output.num_scheduled_tokens.items():
        if scheduler_output.scheduled_cached_reqs.is_context_phase(req_id) or (
            req_id in new_req_ids
        ):
            num_context_requests += 1
            num_context_tokens += num_tokens
        else:
            num_generation_requests += 1
            num_generation_tokens += num_tokens
    return IterationDetails(
        num_context_requests,
        num_context_tokens,
        num_generation_requests,
        num_generation_tokens,
    )
```
**EN:** This function computes derived values within the module. The docstring frames it as: Compute the number of context/generation requests and tokens for the current iteration's scheduler output. Key calls include `items`, `IterationDetails`, `is_context_phase`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会计算派生值，其作用域位于the module。 关键调用包括 `items`, `IterationDetails`, `is_context_phase`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `ConstantList`: central class or interface in this module. / `ConstantList`：本模块中的核心类或接口。
- `CpuGpuBuffer`: central class or interface in this module. / `CpuGpuBuffer`：本模块中的核心类或接口。
- `get_engine_client_zmq_addr`: top-level helper or orchestration entry point. / `get_engine_client_zmq_addr`：顶层辅助函数或编排入口。
- `APIServerProcessManager`: central class or interface in this module. / `APIServerProcessManager`：本模块中的核心类或接口。
- `run_api_server_worker_proc`: top-level helper or orchestration entry point. / `run_api_server_worker_proc`：顶层辅助函数或编排入口。
- `wait_for_completion_or_failure`: top-level helper or orchestration entry point. / `wait_for_completion_or_failure`：顶层辅助函数或编排入口。
- `shutdown`: top-level helper or orchestration entry point. / `shutdown`：顶层辅助函数或编排入口。
- `copy_slice`: top-level helper or orchestration entry point. / `copy_slice`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `argparse`, `contextlib`, `multiprocessing`, `threading`, `time`, `weakref`, `collections`, `dataclasses`, `typing`
- External / 外部依赖: `torch`, `uvloop`, `numpy`, `nvtx`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.logger`, `vllm.usage.usage_lib`, `vllm.utils.network_utils`, `vllm.utils.system_utils`, `vllm.v1.core.sched.output`, `vllm.v1.engine.coordinator`, `vllm.v1.engine.utils`, `vllm.entrypoints.openai.api_server`, `vllm.model_executor.model_loader`
