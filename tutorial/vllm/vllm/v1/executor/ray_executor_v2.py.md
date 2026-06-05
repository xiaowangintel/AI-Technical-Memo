# ray_executor_v2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/executor/ray_executor_v2.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `RayWorkerHandle`, `RayWorkerProc`, `RayExecutorV2` for the V1 `executor` subsystem. / 为 V1 的 `executor` 子系统实现 `RayWorkerHandle`, `RayWorkerProc`, `RayExecutorV2`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import copy
import os
import threading
import weakref
from collections import defaultdict, deque
from dataclasses import dataclass
from typing import Any

import vllm.envs as envs
from vllm.config import VllmConfig
from vllm.distributed.device_communicators.shm_broadcast import (
    Handle,
    MessageQueue,
)
from vllm.logger import init_logger
from vllm.platforms import current_platform
from vllm.utils.network_utils import (
    get_distributed_init_method,
    get_open_port,
)
from vllm.v1.executor.multiproc_executor import (
    FutureWrapper,
    MultiprocExecutor,
    WorkerProc,
)
from vllm.v1.executor.ray_env_utils import get_driver_env_vars
from vllm.v1.executor.ray_utils import (
    WORKER_SPECIFIC_ENV_VARS,
    build_actor_name,
    get_bundles_for_indices,
    get_bundles_sorted_by_node,
    initialize_ray_cluster,
    ray,
)

if ray is not None:
    from ray.actor import ActorHandle
    from ray.types import ObjectRef
    from ray.util.scheduling_strategies import PlacementGroupSchedulingStrategy
else:
    ActorHandle = None

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `RayWorkerHandle` class / `RayWorkerHandle` 类
```python
@dataclass
class RayWorkerHandle:
    """Handle for a Ray worker actor, compatible with MultiprocExecutor."""

    actor: ActorHandle
    """Ray worker actor"""

    rank: int
    """Rank of the worker"""

    local_rank: int
    """Local rank of the worker"""

    node_id: str
    """Node ID of the worker"""

    bundle_id_idx: int = -1
    """Placement group bundle index for the worker"""

    run_ref: ObjectRef | None = None
    """run() ObjectRef used as a sentinel for health monitoring"""
```
**EN:** Uses `@dataclass` to package related state for `RayWorkerHandle`. Typical fields include `actor`, `rank`, `local_rank`, `node_id`, `bundle_id_idx`, `run_ref`.
**CN:** `RayWorkerHandle` 使用 `@dataclass` 打包相关状态。典型字段包括 `actor`, `rank`, `local_rank`, `node_id`, `bundle_id_idx`, `run_ref`。

### `RayWorkerHandle.run` method / `RayWorkerHandle.run` 方法
```python
    def run(self):
        """Start the worker's busy loop"""
        self.run_ref = self.actor.run.remote()
```
**EN:** This method runs the main execution path within `RayWorkerHandle`. The docstring frames it as: Start the worker's busy loop Key calls include `remote`. It touches state such as `run_ref`.
**CN:** 该方法会执行主要运行路径，其作用域位于`RayWorkerHandle`。 关键调用包括 `remote`。 它会读写 `run_ref` 等状态。

### `RayWorkerProc` class / `RayWorkerProc` 类
```python
class RayWorkerProc(WorkerProc):
    """Worker process that runs inside a Ray actor.

    Initialization is split into two phases:
    1. __init__: lightweight setup, stores init args (no device/model init)
    2. initialize_worker: called after GPU IDs are discovered, completes
       the full WorkerProc initialization with the correct local_rank and
       CUDA_VISIBLE_DEVICES.

    CUDA_VISIBLE_DEVICES setup flow:

    1. RayExecutorV2 enables RAY_EXPERIMENTAL_NOSET_CUDA_VISIBLE_DEVICES so Ray does
       not set CUDA_VISIBLE_DEVICES on RayWorkerProc actors at creation time.
    2. Each actor is scheduled with a placement group and bundle index; Ray resolves
       the physical GPU ID for that bundle at placement time.
    3. After placement, the worker discovers that GPU ID and sets
       CUDA_VISIBLE_DEVICES before finishing WorkerProc initialization.

    There is no workaround for this unset-and-reset sequence when the placement group
    is externally managed: scheduling must complete before CUDA_VISIBLE_DEVICES can
    match the GPU tied to the worker's bundle.

    This sequence allows multiple vLLM instances to coexist on the same node:
    each instance is unaware which physical devices others hold, and the
    externally managed placement group avoids CUDA_VISIBLE_DEVICES conflicts
    by binding workers to specific placement group bundles.
    """
```
**EN:** Introduces the `RayWorkerProc` class on top of `WorkerProc`. Core methods include `__init__`, `get_node_and_gpu_ids`, `initialize_worker`, `_init_message_queues`, `wait_for_init`, `run`. Docstring signal: Worker process that runs inside a Ray actor.
**CN:** 这里定义 `RayWorkerProc` 类，其基类包括 `WorkerProc`。核心方法包括 `__init__`, `get_node_and_gpu_ids`, `initialize_worker`, `_init_message_queues`, `wait_for_init`, `run`。

### `RayWorkerProc.__init__` method / `RayWorkerProc.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        rank: int,
        distributed_init_method: str,
        input_shm_handle: Handle,
        is_driver_worker: bool,
        is_driver_node: bool = False,
    ):
        # Defer WorkerProc.__init__ until GPU IDs are known.
        self._is_driver_node = is_driver_node
        self._init_kwargs = dict(
            vllm_config=vllm_config,
            rank=rank,
            distributed_init_method=distributed_init_method,
            input_shm_handle=input_shm_handle,
            shared_worker_lock=None,
            is_driver_worker=is_driver_worker,
        )
```
**EN:** This method initializes the object state within `RayWorkerProc`. Key calls include `dict`. It touches state such as `_is_driver_node`, `_init_kwargs`.
**CN:** 该方法会初始化对象状态，其作用域位于`RayWorkerProc`。 关键调用包括 `dict`。 它会读写 `_is_driver_node`, `_init_kwargs` 等状态。

### `RayWorkerProc.initialize_worker` method / `RayWorkerProc.initialize_worker` 方法
```python
    def initialize_worker(
        self,
        local_rank: int,
        env_vars: dict[str, str],
        driver_env_vars: dict[str, str] | None = None,
    ) -> None:
        """Complete initialization after GPU assignment is known.

        *driver_env_vars* are applied with ``setdefault`` — they fill
        in missing vars but never overwrite node-local values.
        *env_vars* (e.g. CUDA_VISIBLE_DEVICES) always overwrite.
        """
        if driver_env_vars:
            for key, value in driver_env_vars.items():
                os.environ.setdefault(key, value)
        for key, value in env_vars.items():
            os.environ[key] = value

        self.local_rank = local_rank
        super().__init__(
            local_rank=local_rank,
            **self._init_kwargs,
        )
```
**EN:** This method implements `initialize_worker` within `RayWorkerProc`. The docstring frames it as: Complete initialization after GPU assignment is known. Key calls include `items`, `__init__`, `setdefault`, `super`. It touches state such as `local_rank`. The control flow contains 1 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `initialize_worker`，其作用域位于`RayWorkerProc`。 关键调用包括 `items`, `__init__`, `setdefault`, `super`。 它会读写 `local_rank` 等状态。 控制流包含 1 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `RayWorkerProc.run` method / `RayWorkerProc.run` 方法
```python
    def run(self) -> None:
        """Main entry point called via actor.run.remote()."""
        try:
            assert self.rpc_broadcast_mq is not None
            self.rpc_broadcast_mq.wait_until_ready()
            assert self.worker_response_mq is not None
            self.worker_response_mq.wait_until_ready()

            self.worker_busy_loop()
        except Exception as e:
            logger.exception("RayWorkerProc failed: %s", e)
            raise
        finally:
            self.shutdown()
```
**EN:** This method runs the main execution path within `RayWorkerProc`. The docstring frames it as: Main entry point called via actor.run.remote(). Key calls include `wait_until_ready`, `worker_busy_loop`, `shutdown`, `exception`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会执行主要运行路径，其作用域位于`RayWorkerProc`。 关键调用包括 `wait_until_ready`, `worker_busy_loop`, `shutdown`, `exception`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RayExecutorV2` class / `RayExecutorV2` 类
```python
class RayExecutorV2(MultiprocExecutor):
    """Ray-based distributed executor using MessageQueue communication.

    Inherits from MultiprocExecutor to reuse the MQ-based control plane
    and NCCL data plane. Workers are Ray actors.

    Async scheduling is enabled, inherited from MultiprocExecutor.
    This is cricitcal for RayExecutorV2 to be performant.
    """

    uses_ray: bool = True
    supports_pp: bool = True
```
**EN:** Introduces the `RayExecutorV2` class on top of `MultiprocExecutor`. Core methods include `__init__`, `_build_runtime_env`, `_get_actor_resource_kwargs`, `_init_executor`, `start_worker_monitor`, `_join_monitor_thread`. Docstring signal: Ray-based distributed executor using MessageQueue communication.
**CN:** 这里定义 `RayExecutorV2` 类，其基类包括 `MultiprocExecutor`。核心方法包括 `__init__`, `_build_runtime_env`, `_get_actor_resource_kwargs`, `_init_executor`, `start_worker_monitor`, `_join_monitor_thread`。

### `RayExecutorV2.__init__` method / `RayExecutorV2.__init__` 方法
```python
    def __init__(self, vllm_config: VllmConfig):
        super().__init__(vllm_config)
```
**EN:** This method initializes the object state within `RayExecutorV2`. Key calls include `__init__`, `super`.
**CN:** 该方法会初始化对象状态，其作用域位于`RayExecutorV2`。 关键调用包括 `__init__`, `super`。

### `RayExecutorV2._build_runtime_env` method / `RayExecutorV2._build_runtime_env` 方法
```python
    def _build_runtime_env(self) -> dict:
        """Build a runtime_env dict for RayWorkerProc actors.

        Driver env vars are applied separately via initialize_worker
        with setdefault semantics.
        """
        base = self.parallel_config.ray_runtime_env
        runtime_env: dict = copy.deepcopy(dict(base)) if base else {}

        env_vars = runtime_env.setdefault("env_vars", {})
        env_vars.update({v: "1" for v in current_platform.ray_noset_device_env_vars})
        if self.parallel_config.ray_workers_use_nsight:
            runtime_env["nsight"] = {
                "t": "cuda,cudnn,cublas",
                "o": "'worker_process_%p'",
                "cuda-graph-trace": "node",
            }
        return runtime_env
```
**EN:** This method implements `_build_runtime_env` within `RayExecutorV2`. The docstring frames it as: Build a runtime_env dict for RayWorkerProc actors. Key calls include `setdefault`, `update`, `deepcopy`, `dict`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_build_runtime_env`，其作用域位于`RayExecutorV2`。 关键调用包括 `setdefault`, `update`, `deepcopy`, `dict`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RayExecutorV2._init_executor` method / `RayExecutorV2._init_executor` 方法
```python
    def _init_executor(self) -> None:
        """Initialize the RayExecutorV2 executor."""
        self._finalizer = weakref.finalize(self, self.shutdown)
        self.is_failed = False
        self.failure_callback = None
        self.shutting_down = False
        self.shutdown_lock = threading.Lock()

        # Step 1: Initialize Ray cluster and retrieve placement group
        if ray is None:
            raise ImportError("Using Ray backend requires installation of ray.")
        initialize_ray_cluster(self.parallel_config, require_gpu_on_driver=False)
        placement_group = self.parallel_config.placement_group

        tp_size, pp_size, pcp_size = self._get_parallel_sizes()
        assert self.world_size == tp_size * pp_size * pcp_size, (
            f"world_size ({self.world_size}) must be equal to the "
            f"tensor_parallel_size ({tp_size}) x pipeline"
            f"_parallel_size ({pp_size}) x prefill_context"
            f"_parallel_size ({pcp_size}). "
        )

        # Step 2: Build bundle assignments for worker rank placement
        # while respecting VLLM_RAY_BUNDLE_INDICES.
        if envs.VLLM_RAY_BUNDLE_INDICES:
            bundle_to_node_id = get_bundles_for_indices(
                placement_group,
                list(map(int, envs.VLLM_RAY_BUNDLE_INDICES.split(","))),
                self.world_size,
            )
        else:
            bundle_to_node_id = get_bundles_sorted_by_node(placement_group)
        driver_node = ray.get_runtime_context().get_node_id()

        bundle_assignments: list[dict[str, Any]] = []
        for rank, (bundle_id_idx, node_id, node_ip) in enumerate(bundle_to_node_id):
            bundle_assignments.append(
                {
                    "rank": rank,
                    "bundle_id_idx": bundle_id_idx,
                    "node_id": node_id,
                    "node_ip": node_ip,
                }
            )

        # Step 3: Resolve the IP for torch.distributed TCPStore.
        # The TCPStore server runs on rank 0's node, so all workers
        # must be able to reach this address.
        dist_ip = bundle_assignments[0]["node_ip"]
        distributed_init_method = get_distributed_init_method(dist_ip, get_open_port())
    # ... omitted for brevity ...
        )

        self.response_mqs: list[MessageQueue] = []
        for i, result in enumerate(init_results):
            if result["status"] != RayWorkerProc.READY_STR:
                raise RuntimeError(f"Worker {i} failed to initialize: {result}")
            self.response_mqs.append(
                MessageQueue.create_from_handle(result["handle"], 0)
            )

        # Step 9: Start run() before wait_until_ready() to avoid
        # deadlock — workers send subscriptions inside run().
        for handle in self.ray_worker_handles:
            handle.run()

        # Step 10: wait_until_ready() barrier
        self.rpc_broadcast_mq.wait_until_ready()
        for response_mq in self.response_mqs:
            response_mq.wait_until_ready()

        self.futures_queue = deque[FutureWrapper]()
        self._post_init_executor()

        self.start_worker_monitor()
        self.output_rank = self._get_output_rank()
```
**EN:** This method implements `_init_executor` within `RayExecutorV2`. The docstring frames it as: Initialize the RayExecutorV2 executor. Key calls include `finalize`, `Lock`, `initialize_ray_cluster`, `_get_parallel_sizes`, `get_node_id`, `enumerate`. It touches state such as `_finalizer`, `is_failed`, `failure_callback`, `shutting_down`, `shutdown_lock`, `rpc_broadcast_mq`, `ray_worker_handles`, `driver_env_vars`. The control flow contains 3 branch(es) and 8 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `_init_executor`，其作用域位于`RayExecutorV2`。 关键调用包括 `finalize`, `Lock`, `initialize_ray_cluster`, `_get_parallel_sizes`, `get_node_id`, `enumerate`。 它会读写 `_finalizer`, `is_failed`, `failure_callback`, `shutting_down`, `shutdown_lock`, `rpc_broadcast_mq`, `ray_worker_handles`, `driver_env_vars` 等状态。 控制流包含 3 个分支和 8 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

## Key Concepts / 关键概念
- `RayWorkerHandle`: central class or interface in this module. / `RayWorkerHandle`：本模块中的核心类或接口。
- `RayWorkerProc`: central class or interface in this module. / `RayWorkerProc`：本模块中的核心类或接口。
- `RayExecutorV2`: central class or interface in this module. / `RayExecutorV2`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `copy`, `os`, `threading`, `weakref`, `collections`, `dataclasses`, `typing`
- External / 外部依赖: `ray`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.config`, `vllm.distributed.device_communicators.shm_broadcast`, `vllm.logger`, `vllm.platforms`, `vllm.utils.network_utils`, `vllm.v1.executor.multiproc_executor`, `vllm.v1.executor.ray_env_utils`, `vllm.v1.executor.ray_utils`
