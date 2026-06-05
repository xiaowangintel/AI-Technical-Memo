# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/engine/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `CoreEngineState`, `CoreEngine`, `EngineZmqAddresses` for the V1 `engine` subsystem. / 为 V1 的 `engine` 子系统实现 `CoreEngineState`, `CoreEngine`, `EngineZmqAddresses`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import contextlib
import os
import threading
import weakref
from collections.abc import Callable, Iterator
from dataclasses import dataclass
from enum import Enum, auto
from multiprocessing import Process, connection
from multiprocessing.process import BaseProcess
from multiprocessing.queues import Queue
from typing import TYPE_CHECKING, cast
from unittest.mock import patch

import msgspec
import zmq

from vllm import envs
from vllm.config import CacheConfig, ParallelConfig, VllmConfig
from vllm.logger import init_logger
from vllm.platforms import current_platform
from vllm.ray.ray_env import get_env_vars_to_copy
from vllm.utils import numa_utils
from vllm.utils.network_utils import get_open_zmq_ipc_path, zmq_socket_ctx
from vllm.utils.system_utils import get_mp_context
from vllm.v1.engine.coordinator import DPCoordinator
from vllm.v1.executor import Executor
from vllm.v1.executor.ray_utils import WORKER_SPECIFIC_ENV_VARS
from vllm.v1.utils import get_engine_client_zmq_addr, shutdown

if TYPE_CHECKING:
    from ray.util.placement_group import PlacementGroup

logger = init_logger(__name__)

STARTUP_POLL_PERIOD_MS = 10000
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `STARTUP_POLL_PERIOD_MS`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `STARTUP_POLL_PERIOD_MS`。

### `CoreEngineState` class / `CoreEngineState` 类
```python
class CoreEngineState(Enum):
    NEW = auto()
    CONNECTED = auto()
    READY = auto()
```
**EN:** Defines the `CoreEngineState` enum used to normalize modes or options across the subsystem. Representative members: `NEW`, `CONNECTED`, `READY`.
**CN:** `CoreEngineState` 定义了一个枚举，用于统一子系统中的模式或选项。代表性成员包括：`NEW`, `CONNECTED`, `READY`。

### `CoreEngine` class / `CoreEngine` 类
```python
class CoreEngine:
    """One per data parallel rank, used to track state during handshaking."""
```
**EN:** Introduces the `CoreEngine` class. Core methods include `__init__`. Docstring signal: One per data parallel rank, used to track state during handshaking.
**CN:** 这里定义 `CoreEngine` 类。核心方法包括 `__init__`。

### `CoreEngine.__init__` method / `CoreEngine.__init__` 方法
```python
    def __init__(self, index: int = 0, local: bool = True):
        self.local = local
        self.identity = index.to_bytes(2, "little")

        self.state = CoreEngineState.NEW
```
**EN:** This method initializes the object state within `CoreEngine`. Key calls include `to_bytes`. It touches state such as `local`, `identity`, `state`.
**CN:** 该方法会初始化对象状态，其作用域位于`CoreEngine`。 关键调用包括 `to_bytes`。 它会读写 `local`, `identity`, `state` 等状态。

### `EngineZmqAddresses` class / `EngineZmqAddresses` 类
```python
@dataclass
class EngineZmqAddresses:
    # ZMQ input socket addresses for each front-end client (requests)
    inputs: list[str]
    # ZMQ output socket addresses for each front-end client (responses)
    outputs: list[str]
    # ZMQ input socket address of DP coordinator if applicable
    coordinator_input: str | None = None
    # ZMQ output socket address of DP coordinator if applicable
    coordinator_output: str | None = None
    # ZMQ socket for front-end to connect to DP coordinator.
    # Not used by engine, just relayed to front-end in handshake response.
    # Only required for external DP LB case.
    frontend_stats_publish_address: str | None = None
```
**EN:** Uses `@dataclass` to package related state for `EngineZmqAddresses`. Typical fields include `inputs`, `outputs`, `coordinator_input`, `coordinator_output`, `frontend_stats_publish_address`.
**CN:** `EngineZmqAddresses` 使用 `@dataclass` 打包相关状态。典型字段包括 `inputs`, `outputs`, `coordinator_input`, `coordinator_output`, `frontend_stats_publish_address`。

### `EngineHandshakeMetadata` class / `EngineHandshakeMetadata` 类
```python
@dataclass
class EngineHandshakeMetadata:
    """Metadata sent to each engine process during startup handshake,
    including addresses of the front-end ZMQ queues that they should
    connect to.
    """

    addresses: EngineZmqAddresses
    parallel_config: dict[str, int | str | list[int]]
```
**EN:** Uses `@dataclass` to package related state for `EngineHandshakeMetadata`. Typical fields include `addresses`, `parallel_config`.
**CN:** `EngineHandshakeMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `addresses`, `parallel_config`。

### `_make_control_bundle` function / `_make_control_bundle` 函数
```python
def _make_control_bundle(node_ip: str) -> dict[str, float]:
    # The engine actor is scheduled on the final CPU-only bundle. Keep that
    # bundle colocated with the group's first GPU bundle so the actor does not
    # float to an unrelated node and reorder worker ranks away from the
    # advertised DP bootstrap host.
    return {"CPU": 1.0, "node:" + node_ip: 0.001}
```
**EN:** This function implements `_make_control_bundle` within the module.
**CN:** 该函数会实现 `_make_control_bundle`，其作用域位于the module。

### `_get_bundle_node_ip` function / `_get_bundle_node_ip` 函数
```python
def _get_bundle_node_ip(bundle: dict[str, float]) -> str:
    for key in bundle:
        if key.startswith("node:"):
            return key.split(":", 1)[1]
    raise ValueError(f"Missing node affinity in placement bundle: {bundle}")
```
**EN:** This function implements `_get_bundle_node_ip` within the module. Key calls include `ValueError`, `startswith`, `split`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_get_bundle_node_ip`，其作用域位于the module。 关键调用包括 `ValueError`, `startswith`, `split`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `CoreEngineProcManager` class / `CoreEngineProcManager` 类
```python
class CoreEngineProcManager:
    """
    Utility class to handle creation, readiness, and shutdown
    of background processes used by the AsyncLLM and LLMEngine.
    """
```
**EN:** Introduces the `CoreEngineProcManager` class. Core methods include `__init__`, `shutdown`, `monitor_engine_liveness`, `sentinels`, `finished_procs`. Docstring signal: Utility class to handle creation, readiness, and shutdown of background processes used by the AsyncLLM and LLMEngine.
**CN:** 这里定义 `CoreEngineProcManager` 类。核心方法包括 `__init__`, `shutdown`, `monitor_engine_liveness`, `sentinels`, `finished_procs`。

### `CoreEngineProcManager.__init__` method / `CoreEngineProcManager.__init__` 方法
```python
    def __init__(
        self,
        local_engine_count: int,
        start_index: int,
        local_start_index: int,
        vllm_config: VllmConfig,
        local_client: bool,
        handshake_address: str,
        executor_class: type[Executor],
        log_stats: bool,
        client_handshake_address: str | None = None,
        tensor_queue: Queue | None = None,
    ):
        context = get_mp_context()
        common_kwargs = {
            "vllm_config": vllm_config,
            "local_client": local_client,
            "handshake_address": handshake_address,
            "executor_class": executor_class,
            "log_stats": log_stats,
            "tensor_queue": tensor_queue,
        }

        if client_handshake_address:
            common_kwargs["client_handshake_address"] = client_handshake_address

        is_dp = vllm_config.parallel_config.data_parallel_size > 1

        from vllm.v1.engine.core import EngineCoreProc

        self.processes: list[BaseProcess] = []
        local_dp_ranks = []
        for index in range(local_engine_count):
            local_index = local_start_index + index
            global_index = start_index + index

            # Start EngineCore in background process.
            local_dp_ranks.append(local_index)
            self.processes.append(
                context.Process(
                    target=EngineCoreProc.run_engine_core,
                    name=f"EngineCore_DP{global_index}" if is_dp else "EngineCore",
                    kwargs=common_kwargs
                    | {"dp_rank": global_index, "local_dp_rank": local_index},
                )
            )

        self._finalizer = weakref.finalize(self, shutdown, self.processes)
        self.manager_stopped = threading.Event()
        self.failed_proc_name: str | None = None

        try:
            for proc, local_dp_rank in zip(self.processes, local_dp_ranks):
                # Adjust device control in DP for platforms that cannot rely
                # on torch.accelerator.set_device_index(), and for Ray launchers.
                device_control_context: contextlib.AbstractContextManager[None] = (
                    contextlib.nullcontext()
                )
                needs_device_env_isolation = not (
                    current_platform.is_cuda_alike() or current_platform.is_xpu()
                )
                if is_dp and (
                    needs_device_env_isolation or vllm_config.parallel_config.use_ray
                ):
                    device_control_context = set_device_control_env_var(
                        vllm_config, local_dp_rank
                    )

                with (
                    device_control_context,
                    numa_utils.configure_subprocess(
                        # EngineCore itself does not have a TP/PP-local rank.
                        # When DP is enabled, set_device_control_env_var()
                        # narrows visible devices to this DP shard first, so
                        # local_rank=0 means "the first local GPU in this
                        # shard". The actual TP/PP worker processes spawned by
                        # the executor are bound separately with their own
                        # local_rank values.
                        vllm_config,
                        local_rank=0,
                        dp_local_rank=local_dp_rank,
                        process_kind="EngineCore",
                    ),
                ):
                    proc.start()
        finally:
            # Kill other procs if not all are running.
            if self.finished_procs():
                self.shutdown()
```
**EN:** This method initializes the object state within `CoreEngineProcManager`. Key calls include `get_mp_context`, `range`, `finalize`, `Event`, `append`, `zip`. It touches state such as `processes`, `_finalizer`, `manager_stopped`, `failed_proc_name`. The control flow contains 5 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`CoreEngineProcManager`。 关键调用包括 `get_mp_context`, `range`, `finalize`, `Event`, `append`, `zip`。 它会读写 `processes`, `_finalizer`, `manager_stopped`, `failed_proc_name` 等状态。 控制流包含 5 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `CoreEngineProcManager.shutdown` method / `CoreEngineProcManager.shutdown` 方法
```python
    def shutdown(self, timeout: float | None = None) -> None:
        """Shutdown engine core processes with configurable timeout."""
        self.manager_stopped.set()
        if self._finalizer.detach() is not None:
            shutdown(self.processes, timeout=timeout)
```
**EN:** This method implements `shutdown` within `CoreEngineProcManager`. The docstring frames it as: Shutdown engine core processes with configurable timeout. Key calls include `set`, `detach`, `shutdown`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `shutdown`，其作用域位于`CoreEngineProcManager`。 关键调用包括 `set`, `detach`, `shutdown`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `CoreEngineProcManager.monitor_engine_liveness` method / `CoreEngineProcManager.monitor_engine_liveness` 方法
```python
    def monitor_engine_liveness(self) -> None:
        """Monitor engine core process liveness."""

        sentinel_to_proc = {proc.sentinel: proc for proc in self.processes}
        sentinels = set(sentinel_to_proc.keys())

        while sentinels and not self.manager_stopped.is_set():
            died_sentinels = connection.wait(sentinels, timeout=1)

            for sentinel in died_sentinels:
                proc = sentinel_to_proc.pop(cast(int, sentinel))
                exitcode = proc.exitcode
                if exitcode != 0 and not self.manager_stopped.is_set():
                    self.failed_proc_name = proc.name
            if died_sentinels:
                # Any engine exit currently triggers a shutdown. Future
                # work (e.g., Elastic and fault-tolerant EP) will add finer-grained
                # handling for different exit scenarios.
                break

        self.shutdown()
```
**EN:** This method implements `monitor_engine_liveness` within `CoreEngineProcManager`. The docstring frames it as: Monitor engine core process liveness. Key calls include `set`, `shutdown`, `keys`, `wait`, `is_set`, `pop`. It touches state such as `failed_proc_name`. The control flow contains 2 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `monitor_engine_liveness`，其作用域位于`CoreEngineProcManager`。 关键调用包括 `set`, `shutdown`, `keys`, `wait`, `is_set`, `pop`。 它会读写 `failed_proc_name` 等状态。 控制流包含 2 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `CoreEngineProcManager.finished_procs` method / `CoreEngineProcManager.finished_procs` 方法
```python
    def finished_procs(self) -> dict[str, int]:
        """Returns dict of proc name -> exit code for any finished procs."""
        return {
            proc.name: proc.exitcode
            for proc in self.processes
            if proc.exitcode is not None
        }
```
**EN:** This method implements `finished_procs` within `CoreEngineProcManager`. The docstring frames it as: Returns dict of proc name -> exit code for any finished procs.
**CN:** 该方法会实现 `finished_procs`，其作用域位于`CoreEngineProcManager`。

### `SignalCallback` class / `SignalCallback` 类
```python
class SignalCallback:
    """Safely trigger a callback from signal handler context via a dedicated thread."""
```
**EN:** Introduces the `SignalCallback` class. Core methods include `__init__`, `_run`, `trigger`, `stop`. Docstring signal: Safely trigger a callback from signal handler context via a dedicated thread.
**CN:** 这里定义 `SignalCallback` 类。核心方法包括 `__init__`, `_run`, `trigger`, `stop`。

### `SignalCallback.__init__` method / `SignalCallback.__init__` 方法
```python
    def __init__(self, callback: Callable[[], None]):
        self._callback = callback
        self._event = threading.Event()
        self._stopped = False
        self._thread = threading.Thread(
            target=self._run,
            daemon=True,
            name="signal-callback",
        )
        self._thread.start()
```
**EN:** This method initializes the object state within `SignalCallback`. Key calls include `Event`, `Thread`, `start`. It touches state such as `_callback`, `_event`, `_stopped`, `_thread`.
**CN:** 该方法会初始化对象状态，其作用域位于`SignalCallback`。 关键调用包括 `Event`, `Thread`, `start`。 它会读写 `_callback`, `_event`, `_stopped`, `_thread` 等状态。

### `SignalCallback._run` method / `SignalCallback._run` 方法
```python
    def _run(self):
        self._event.wait()
        if not self._stopped:
            self._callback()
```
**EN:** This method implements `_run` within `SignalCallback`. Key calls include `wait`, `_callback`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_run`，其作用域位于`SignalCallback`。 关键调用包括 `wait`, `_callback`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `SignalCallback.trigger` method / `SignalCallback.trigger` 方法
```python
    def trigger(self):
        self._event.set()
```
**EN:** This method implements `trigger` within `SignalCallback`. Key calls include `set`.
**CN:** 该方法会实现 `trigger`，其作用域位于`SignalCallback`。 关键调用包括 `set`。

### `SignalCallback.stop` method / `SignalCallback.stop` 方法
```python
    def stop(self):
        self._stopped = True
        self._event.set()
```
**EN:** This method implements `stop` within `SignalCallback`. Key calls include `set`. It touches state such as `_stopped`.
**CN:** 该方法会实现 `stop`，其作用域位于`SignalCallback`。 关键调用包括 `set`。 它会读写 `_stopped` 等状态。

### `set_device_control_env_var` function / `set_device_control_env_var` 函数
```python
@contextlib.contextmanager
def set_device_control_env_var(
    vllm_config: VllmConfig, local_dp_rank: int
) -> Iterator[None]:
    """
    Temporarily set CUDA_VISIBLE_DEVICES or equivalent
    for engine subprocess.
    """
    world_size = vllm_config.parallel_config.world_size
    local_world_size = vllm_config.parallel_config.local_world_size
    evar = current_platform.device_control_env_var

    value = get_device_indices(evar, local_dp_rank, world_size, local_world_size)
    with patch.dict(os.environ, values=((evar, value),)):
        yield
```
**EN:** This function stores a value into object state within the module. The docstring frames it as: Temporarily set CUDA_VISIBLE_DEVICES or equivalent for engine subprocess. Key calls include `get_device_indices`, `dict`.
**CN:** 该函数会将值写入对象状态，其作用域位于the module。 关键调用包括 `get_device_indices`, `dict`。

### `get_device_indices` function / `get_device_indices` 函数
```python
def get_device_indices(
    device_control_env_var: str,
    local_dp_rank: int,
    world_size: int,
    local_world_size: int | None = None,
):
    """
    Returns a comma-separated string of device indices for the specified
    data parallel rank.

    For example, if world_size=2 and local_dp_rank=1, and there are 4 devices,
    this will select devices 2 and 3 for local_dp_rank=1.
    """
    if local_world_size is None:
        local_world_size = world_size
    try:
        value = ",".join(
            str(current_platform.device_id_to_physical_device_id(i))
            for i in range(
                local_dp_rank * world_size,
                local_dp_rank * world_size + local_world_size,
            )
        )
    except IndexError as e:
        raise Exception(
            f"Error setting {device_control_env_var}: "
            f"local range: [{local_dp_rank * world_size}, "
            f"{(local_dp_rank + 1) * world_size}) "
            "base value: "
            f'"{os.getenv(device_control_env_var)}"'
        ) from e
    return value
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Returns a comma-separated string of device indices for the specified data parallel rank. Key calls include `join`, `Exception`, `str`, `device_id_to_physical_device_id`, `range`, `getenv`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `join`, `Exception`, `str`, `device_id_to_physical_device_id`, `range`, `getenv`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_apply_dp_identity_suffix` function / `_apply_dp_identity_suffix` 函数
```python
def _apply_dp_identity_suffix(dp_vllm_config, dp_rank: int) -> None:
    # Ray actor names (RayExecutorV2) and KV-connector engine_ids must
    # be unique across sibling DP engines or registration collides.
    # Use the global DP rank, not a node-local rank, since sibling DP
    # engines can span multiple nodes.
    dp_vllm_config.instance_id = f"{dp_vllm_config.instance_id}_dp{dp_rank}"
    if dp_vllm_config.kv_transfer_config is not None:
        dp_vllm_config.kv_transfer_config.engine_id = (
            f"{dp_vllm_config.kv_transfer_config.engine_id}_dp{dp_rank}"
        )
```
**EN:** This function implements `_apply_dp_identity_suffix` within the module. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_apply_dp_identity_suffix`，其作用域位于the module。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `CoreEngineActorManager` class / `CoreEngineActorManager` 类
```python
class CoreEngineActorManager:
    """
    Utility class to handle creation, readiness, and shutdown
    of core engine Ray actors used by the AsyncLLM and LLMEngine.

    Different from CoreEngineProcManager, this class manages
    core engines for both local and remote nodes.
    """
```
**EN:** Introduces the `CoreEngineActorManager` class. Core methods include `__init__`, `create_dp_placement_groups`, `add_dp_placement_groups`, `scale_up_elastic_ep`, `scale_down_elastic_ep`, `remove_run_refs_for_scale_down`. Docstring signal: Utility class to handle creation, readiness, and shutdown of core engine Ray actors used by the AsyncLLM and LLMEngine.
**CN:** 这里定义 `CoreEngineActorManager` 类。核心方法包括 `__init__`, `create_dp_placement_groups`, `add_dp_placement_groups`, `scale_up_elastic_ep`, `scale_down_elastic_ep`, `remove_run_refs_for_scale_down`。

### `CoreEngineActorManager.__init__` method / `CoreEngineActorManager.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        addresses: EngineZmqAddresses,
        executor_class: type[Executor],
        log_stats: bool,
        placement_groups: list["PlacementGroup"] | None = None,
        local_dp_ranks: list[int] | None = None,
    ):
        import copy

        import ray
        from ray.runtime_env import RuntimeEnv
        from ray.util.scheduling_strategies import PlacementGroupSchedulingStrategy

        from vllm.v1.engine.core import DPMoEEngineCoreActor, EngineCoreActor

        dp_size = vllm_config.parallel_config.data_parallel_size
        actor_class = (
            DPMoEEngineCoreActor
            if dp_size > 1 and vllm_config.model_config.is_moe
            else EngineCoreActor
        )

        self.local_engine_actors: list[ray.ActorHandle] = []
        self.remote_engine_actors: list[ray.ActorHandle] = []

        env_vars_list = get_env_vars_to_copy(
            destination=actor_class.__name__,
            exclude_vars=WORKER_SPECIFIC_ENV_VARS,
        )
        self.env_vars_dict = {
            name: os.environ[name] for name in env_vars_list if name in os.environ
        }
        runtime_env = RuntimeEnv(env_vars=self.env_vars_dict)

        self.addresses = addresses
        self.executor_class = executor_class
        self.log_stats = log_stats
        local_engine_count = vllm_config.parallel_config.data_parallel_size_local
        world_size = vllm_config.parallel_config.world_size
        self.manager_stopped = threading.Event()
        self.failed_proc_name: str | None = None

        if ray.is_initialized():
            logger.info("Ray is already initialized. Skipping Ray initialization.")
        else:
            ray.init()

        parallel_config = vllm_config.parallel_config
    # ... omitted for brevity ...
                )
                .remote(
                    vllm_config=dp_vllm_config,
                    executor_class=executor_class,
                    log_stats=log_stats,
                    local_client=local_client,
                    addresses=addresses,
                    dp_rank=index,
                    local_dp_rank=local_index,
                )
            )
            if local_client:
                self.local_engine_actors.append(actor)
            else:
                self.remote_engine_actors.append(actor)
            self.placement_group_is_local.append(local_client)
            refs.append(actor.wait_for_init.remote())

        ray.get(refs)
        self.run_refs = []
        self.actor_run_ref_dict = dict()
        for actor in self.local_engine_actors + self.remote_engine_actors:
            ref = actor.run.remote()
            self.run_refs.append(ref)
            self.actor_run_ref_dict[actor] = ref
```
**EN:** This method initializes the object state within `CoreEngineActorManager`. Key calls include `get_env_vars_to_copy`, `RuntimeEnv`, `Event`, `is_initialized`, `zip`, `get`. It touches state such as `local_engine_actors`, `remote_engine_actors`, `env_vars_dict`, `addresses`, `executor_class`, `log_stats`, `manager_stopped`, `failed_proc_name`. The control flow contains 7 branch(es) and 2 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会初始化对象状态，其作用域位于`CoreEngineActorManager`。 关键调用包括 `get_env_vars_to_copy`, `RuntimeEnv`, `Event`, `is_initialized`, `zip`, `get`。 它会读写 `local_engine_actors`, `remote_engine_actors`, `env_vars_dict`, `addresses`, `executor_class`, `log_stats`, `manager_stopped`, `failed_proc_name` 等状态。 控制流包含 7 个分支和 2 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `CoreEngineActorManager.create_dp_placement_groups` method / `CoreEngineActorManager.create_dp_placement_groups` 方法
```python
    @staticmethod
    def create_dp_placement_groups(
        vllm_config: VllmConfig,
    ) -> tuple[list["PlacementGroup"], list[int]]:
        """
        Create placement groups for data parallel.
        """

        import ray
        from ray._private.state import available_resources_per_node

        logger.info("Creating placement groups for data parallel")
        dp_master_ip = vllm_config.parallel_config.data_parallel_master_ip
        dp_size = vllm_config.parallel_config.data_parallel_size
        dp_size_local = vllm_config.parallel_config.data_parallel_size_local

        available_resources = available_resources_per_node()
        world_size = vllm_config.parallel_config.world_size
        placement_groups: list[PlacementGroup] = []
        local_dp_ranks: list[int] = []

        dp_master_ip_key = f"node:{dp_master_ip}"
        nodes = sorted(
            available_resources.values(), key=lambda x: dp_master_ip_key not in x
        )
        assert len(nodes) > 0, "No nodes with resources found in Ray cluster."
        assert dp_master_ip_key in nodes[0], (
            f"The DP master node (ip: {dp_master_ip}) is missing or dead"
        )
        device_str = current_platform.ray_device_key
        n_node_devices: list[int] = [
            int(node_resources[device_str])
            for node_resources in nodes
            if device_str in node_resources
        ]
        assert n_node_devices, f"No {device_str} found in Ray cluster."
        max_device_per_node = max(n_node_devices)

        pack_strategy = envs.VLLM_RAY_DP_PACK_STRATEGY
        _supported_pack_strategies = ("strict", "fill", "span")
        if pack_strategy not in _supported_pack_strategies:
            raise ValueError(
                f"{envs.VLLM_RAY_DP_PACK_STRATEGY} is not supported. "
                "Make sure to set `VLLM_RAY_DP_PACK_STRATEGY` "
                f"to one of {_supported_pack_strategies}"
            )

        all2all_backend = vllm_config.parallel_config.all2all_backend
        if pack_strategy == "fill" and (
            all2all_backend == "deepep_high_throughput"
    # ... omitted for brevity ...
                    name=f"dp_rank_{len(placement_groups)}",
                    strategy=placement_strategy,
                    bundles=bundles,
                )
                placement_groups.append(pg)
                local_dp_ranks.append(i)
                if len(placement_groups) == dp_size:
                    break

        if len(placement_groups) < dp_size:
            raise ValueError(
                f"Not enough resources to allocate {dp_size} "
                "placement groups, only created "
                f"{len(placement_groups)} placement groups. "
                "Available resources: "
                f"{available_resources}"
            )
        assert len(placement_groups) == dp_size, (
            f"Created {len(placement_groups)} DP placement groups, expected {dp_size}"
        )
        assert len(local_dp_ranks) == dp_size, (
            f"local_dp_ranks length {len(local_dp_ranks)} does not match "
            f"expected {dp_size}"
        )
        return placement_groups, local_dp_ranks
```
**EN:** This method creates a new object or plan within `CoreEngineActorManager`. The docstring frames it as: Create placement groups for data parallel. Key calls include `info`, `available_resources_per_node`, `sorted`, `max`, `values`, `len`. The control flow contains 12 branch(es) and 2 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会创建新的对象或计划，其作用域位于`CoreEngineActorManager`。 关键调用包括 `info`, `available_resources_per_node`, `sorted`, `max`, `values`, `len`。 控制流包含 12 个分支和 2 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `CoreEngineActorManager.scale_up_elastic_ep` method / `CoreEngineActorManager.scale_up_elastic_ep` 方法
```python
    def scale_up_elastic_ep(
        self, cur_vllm_config: VllmConfig, new_data_parallel_size: int
    ) -> None:
        import copy

        import ray
        from ray.runtime_env import RuntimeEnv
        from ray.util.scheduling_strategies import PlacementGroupSchedulingStrategy

        from vllm.v1.engine.core import DPMoEEngineCoreActor, EngineCoreActor

        actor_class = (
            DPMoEEngineCoreActor
            if cur_vllm_config.model_config.is_moe
            else EngineCoreActor
        )

        cur_data_parallel_size = len(self.local_engine_actors) + len(
            self.remote_engine_actors
        )

        assert new_data_parallel_size > cur_data_parallel_size, (
            f"New data parallel size {new_data_parallel_size} must be greater "
            f"than current data parallel size {cur_data_parallel_size} "
            "for scale up"
        )

        placement_groups, local_dp_ranks = self.add_dp_placement_groups(
            cur_vllm_config, new_data_parallel_size
        )

        world_size = cur_vllm_config.parallel_config.world_size
        dp_master_ip = cur_vllm_config.parallel_config.data_parallel_master_ip
        new_local_engines = 0

        runtime_env = RuntimeEnv(
            env_vars=self.env_vars_dict | {"VLLM_ELASTIC_EP_SCALE_UP_LAUNCH": "1"}
        )
        for i, (pg, local_rank) in enumerate(zip(placement_groups, local_dp_ranks)):
            rank = cur_data_parallel_size + i
            dp_vllm_config = copy.deepcopy(cur_vllm_config)
            if new_data_parallel_size > 1:
                _apply_dp_identity_suffix(dp_vllm_config, rank)
            dp_vllm_config.parallel_config.data_parallel_size = new_data_parallel_size
            dp_vllm_config.parallel_config.placement_group = pg

            # Check if this placement group is on the head node
            local_client = any(
                bundle.get("node:" + dp_master_ip, 0) > 0 for bundle in pg.bundle_specs
            )
    # ... omitted for brevity ...
                )
                + self.remote_engine_actors[
                    -(len(placement_groups) - new_local_engines) :
                ]
            ]
        )

        actors = (
            self.local_engine_actors[-new_local_engines:]
            if new_local_engines > 0
            else []
        ) + self.remote_engine_actors[-(len(placement_groups) - new_local_engines) :]

        for actor in actors:
            ref = actor.run.remote()
            self.run_refs.append(ref)
            self.actor_run_ref_dict[actor] = ref

        cur_vllm_config.parallel_config.data_parallel_size = new_data_parallel_size
        # Update old_vllm_config with new data_parallel_size_local if any new
        # local engines were added
        if new_local_engines > 0:
            cur_vllm_config.parallel_config.data_parallel_size_local += (
                new_local_engines
            )
```
**EN:** This method implements `scale_up_elastic_ep` within `CoreEngineActorManager`. Key calls include `add_dp_placement_groups`, `RuntimeEnv`, `enumerate`, `get`, `len`, `zip`. It touches state such as `actor_run_ref_dict`. The control flow contains 7 branch(es) and 2 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `scale_up_elastic_ep`，其作用域位于`CoreEngineActorManager`。 关键调用包括 `add_dp_placement_groups`, `RuntimeEnv`, `enumerate`, `get`, `len`, `zip`。 它会读写 `actor_run_ref_dict` 等状态。 控制流包含 7 个分支和 2 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `CoreEngineActorManager.remove_run_refs_for_scale_down` method / `CoreEngineActorManager.remove_run_refs_for_scale_down` 方法
```python
    def remove_run_refs_for_scale_down(self, removed_dp_size: int) -> None:
        if removed_dp_size <= 0:
            return
        flags = self.placement_group_is_local[-removed_dp_size:]
        li = len(self.local_engine_actors) - 1
        ri = len(self.remote_engine_actors) - 1
        for is_local in reversed(flags):
            if is_local:
                actor = self.local_engine_actors[li]
                li -= 1
            else:
                actor = self.remote_engine_actors[ri]
                ri -= 1
            ref = self.actor_run_ref_dict.pop(actor)
            self.run_refs.remove(ref)
```
**EN:** This method implements `remove_run_refs_for_scale_down` within `CoreEngineActorManager`. Key calls include `reversed`, `len`, `pop`, `remove`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `remove_run_refs_for_scale_down`，其作用域位于`CoreEngineActorManager`。 关键调用包括 `reversed`, `len`, `pop`, `remove`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `get_engine_zmq_addresses` function / `get_engine_zmq_addresses` 函数
```python
def get_engine_zmq_addresses(
    vllm_config: VllmConfig,
    num_api_servers: int = 1,
) -> EngineZmqAddresses:
    """Allocate ZMQ addresses for engine-client communication."""
    parallel_config = vllm_config.parallel_config
    local_engine_count = parallel_config.data_parallel_size_local
    local_start_index = parallel_config.data_parallel_rank_local
    dp_size = parallel_config.data_parallel_size
    host = parallel_config.data_parallel_master_ip
    local_engines_only = parallel_config.local_engines_only

    # In offline mode there is an LLM instance per DP rank and
    # one core engine per LLM, see
    # examples/features/data_parallel/data_parallel_offline.py.
    offline_mode = local_start_index is not None

    # client_local_only = True for cases where this front-end
    # sends requests only to colocated engines.
    client_local_only = (
        offline_mode or local_engines_only or (local_engine_count == dp_size)
    )
    # NOTE(yongji): handling scaling from intra-node to inter-node
    if parallel_config.enable_elastic_ep:
        client_local_only = False

    return EngineZmqAddresses(
        inputs=[
            get_engine_client_zmq_addr(client_local_only, host)
            for _ in range(num_api_servers)
        ],
        outputs=[
            get_engine_client_zmq_addr(client_local_only, host)
            for _ in range(num_api_servers)
        ],
    )
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Allocate ZMQ addresses for engine-client communication. Key calls include `EngineZmqAddresses`, `get_engine_client_zmq_addr`, `range`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `EngineZmqAddresses`, `get_engine_client_zmq_addr`, `range`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `launch_core_engines` function / `launch_core_engines` 函数
```python
@contextlib.contextmanager
def launch_core_engines(
    vllm_config: VllmConfig,
    executor_class: type[Executor],
    log_stats: bool,
    addresses: EngineZmqAddresses,
    num_api_servers: int = 1,
) -> Iterator[
    tuple[
        CoreEngineProcManager | CoreEngineActorManager | None,
        DPCoordinator | None,
        EngineZmqAddresses,
        Queue | None,
    ]
]:
    """Launch engine and DP coordinator processes as needed."""

    parallel_config = vllm_config.parallel_config
    dp_size = parallel_config.data_parallel_size
    local_engine_count = parallel_config.data_parallel_size_local
    local_start_index = parallel_config.data_parallel_rank_local
    dp_rank = parallel_config.data_parallel_rank
    host = parallel_config.data_parallel_master_ip
    local_engines_only = parallel_config.local_engines_only

    offline_mode = local_start_index is not None

    # Create a single tensor IPC queue for sharing multimodal tensors between
    # API servers and engine core. Returns a single queue since we only support
    # DP=1 for this data flow.
    tensor_queue: Queue | None = None
    multimodal_config = vllm_config.model_config.multimodal_config
    if multimodal_config is not None and multimodal_config.mm_tensor_ipc == "torch_shm":
        tensor_queue = get_mp_context().Queue()

    # Run the DP Coordinator process with rank 0 when in online DP mode.
    # The coordinator is needed for:
    # 1. Internal/hybrid LB: collecting and publishing queue stats for load balancing
    # 2. MoE models: wave coordination in addition to stats
    run_coordinator = (
        vllm_config.needs_dp_coordinator and not offline_mode and dp_rank == 0
    )

    if run_coordinator:
        coordinator = DPCoordinator(
            parallel_config,
            enable_wave_coordination=vllm_config.model_config.is_moe,
        )

        addresses.coordinator_input, addresses.coordinator_output = (
# ... omitted for brevity ...
                log_stats=log_stats,
                handshake_address=handshake_address,
                client_handshake_address=client_handshake_address,
                local_client=True,
                local_engine_count=local_engine_count,
                start_index=dp_rank,
                local_start_index=local_start_index or 0,
                tensor_queue=tensor_queue,
            )
        else:
            local_engine_manager = None

        yield local_engine_manager, coordinator, addresses, tensor_queue

        # Now wait for engines to start.
        wait_for_engine_startup(
            handshake_socket,
            addresses,
            engines_to_handshake,
            parallel_config,
            dp_size > 1 and vllm_config.model_config.is_moe,
            vllm_config.cache_config,
            local_engine_manager,
            coordinator.proc if coordinator else None,
        )
```
**EN:** This function implements `launch_core_engines` within the module. The docstring frames it as: Launch engine and DP coordinator processes as needed. Key calls include `get_engine_client_zmq_addr`, `Queue`, `DPCoordinator`, `get_engine_socket_addresses`, `get_stats_publish_address`, `info`. The control flow contains 9 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `launch_core_engines`，其作用域位于the module。 关键调用包括 `get_engine_client_zmq_addr`, `Queue`, `DPCoordinator`, `get_engine_socket_addresses`, `get_stats_publish_address`, `info`。 控制流包含 9 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `wait_for_engine_startup` function / `wait_for_engine_startup` 函数
```python
def wait_for_engine_startup(
    handshake_socket: zmq.Socket,
    addresses: EngineZmqAddresses,
    core_engines: list[CoreEngine],
    parallel_config: ParallelConfig,
    coordinated_dp: bool,
    cache_config: CacheConfig,
    proc_manager: CoreEngineProcManager | None,
    coord_process: Process | None,
):
    # Wait for engine core process(es) to send ready messages.
    local_count = parallel_config.data_parallel_size_local
    remote_count = len(core_engines) - local_count
    # [local, remote] counts
    conn_pending, start_pending = [local_count, remote_count], [0, 0]
    poller = zmq.Poller()
    poller.register(handshake_socket, zmq.POLLIN)

    remote_should_be_headless = (
        not parallel_config.data_parallel_hybrid_lb
        and not parallel_config.data_parallel_external_lb
    )

    if proc_manager is not None:
        for sentinel in proc_manager.sentinels():
            poller.register(sentinel, zmq.POLLIN)
    if coord_process is not None:
        poller.register(coord_process.sentinel, zmq.POLLIN)
    while any(conn_pending) or any(start_pending):
        events = poller.poll(STARTUP_POLL_PERIOD_MS)
        if not events:
            if any(conn_pending):
                logger.debug(
                    "Waiting for %d local, %d remote core engine proc(s) to connect.",
                    *conn_pending,
                )
            if any(start_pending):
                logger.debug(
                    "Waiting for %d local, %d remote core engine proc(s) to start.",
                    *start_pending,
                )
            continue
        if len(events) > 1 or events[0][0] != handshake_socket:
            # One of the local core processes exited.
            finished = proc_manager.finished_procs() if proc_manager else {}
            if coord_process is not None and coord_process.exitcode is not None:
                finished[coord_process.name] = coord_process.exitcode
            raise RuntimeError(
                "Engine core initialization failed. "
                "See root cause above. "
# ... omitted for brevity ...
                        f"{eng_index}. All DP workers must have identical "
                        f"configurations for parameters that affect collective "
                        f"communication (e.g., enable_eplb, "
                        f"eplb_config.log_balancedness). "
                        f"Worker hash: {worker_config_hash}, "
                        f"Expected hash: {expected_hash}. "
                        f"Please ensure all workers are started with the same "
                        f"command-line arguments."
                    )

            start_pending[0 if local else 1] -= 1
            engine.state = CoreEngineState.READY
        else:
            raise RuntimeError(
                f"Unexpected {status} message for "
                f"{'local' if local else 'remote'} engine "
                f"{eng_index} in {engine.state} state."
            )

        logger.debug(
            "%s from %s core engine process %s.",
            status,
            "local" if local else "remote",
            eng_index,
        )
```
**EN:** This function implements `wait_for_engine_startup` within the module. Key calls include `Poller`, `register`, `len`, `sentinels`, `any`, `poll`. The control flow contains 24 branch(es) and 2 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `wait_for_engine_startup`，其作用域位于the module。 关键调用包括 `Poller`, `register`, `len`, `sentinels`, `any`, `poll`。 控制流包含 24 个分支和 2 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

## Key Concepts / 关键概念
- `CoreEngineState`: central class or interface in this module. / `CoreEngineState`：本模块中的核心类或接口。
- `CoreEngine`: central class or interface in this module. / `CoreEngine`：本模块中的核心类或接口。
- `EngineZmqAddresses`: central class or interface in this module. / `EngineZmqAddresses`：本模块中的核心类或接口。
- `EngineHandshakeMetadata`: central class or interface in this module. / `EngineHandshakeMetadata`：本模块中的核心类或接口。
- `_make_control_bundle`: top-level helper or orchestration entry point. / `_make_control_bundle`：顶层辅助函数或编排入口。
- `_get_bundle_node_ip`: top-level helper or orchestration entry point. / `_get_bundle_node_ip`：顶层辅助函数或编排入口。
- `CoreEngineProcManager`: central class or interface in this module. / `CoreEngineProcManager`：本模块中的核心类或接口。
- `SignalCallback`: central class or interface in this module. / `SignalCallback`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `contextlib`, `os`, `threading`, `weakref`, `collections`, `dataclasses`, `enum`, `multiprocessing`, `typing`, `unittest`, `copy`
- External / 外部依赖: `msgspec`, `zmq`, `ray`
- Internal vLLM / 内部依赖: `vllm`, `vllm.config`, `vllm.logger`, `vllm.platforms`, `vllm.ray.ray_env`, `vllm.utils`, `vllm.utils.network_utils`, `vllm.utils.system_utils`, `vllm.v1.engine.coordinator`, `vllm.v1.executor`, `vllm.v1.executor.ray_utils`, `vllm.v1.utils`
