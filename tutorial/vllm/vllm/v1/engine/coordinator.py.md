# coordinator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/engine/coordinator.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `DPCoordinator`, `EngineState`, `DPCoordinatorProc` for the V1 `engine` subsystem. / 为 V1 的 `engine` 子系统实现 `DPCoordinator`, `EngineState`, `DPCoordinatorProc`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import copy
import multiprocessing
import multiprocessing.connection
import time
import weakref

import msgspec.msgpack
import zmq

from vllm.config import ParallelConfig
from vllm.logger import init_logger
from vllm.utils.network_utils import get_tcp_uri, make_zmq_socket
from vllm.utils.system_utils import get_mp_context, set_process_title
from vllm.v1.engine import EngineCoreOutputs, EngineCoreRequestType
from vllm.v1.serial_utils import MsgpackDecoder
from vllm.v1.utils import get_engine_client_zmq_addr, shutdown

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `DPCoordinator` class / `DPCoordinator` 类
```python
class DPCoordinator:
    """Coordinator process used for data-parallel deployments (DP>1).

    Intermediates between multiple DP engine rank processes and one or more
    front-end API server processes.

    * Collects stats from each DP engine (currently just waiting and running
      queue lengths), and publishes these to all front-ends for use in
      load-balancing decisions.

    * Keeps track of the current DP "request wave" number and running state
      of the engines. This is received from the DP rank 0 engine and published
      to the front-end processes along with the current load stats.

      The engines alternate between a global running/paused state. The global
      "request wave" number is a count of the number of times that the workers
      collectively move from a running state to a paused state. This transition
      is synchronized via the all-reduce operation performed in the
      DPEngineCoreProc._has_global_unfinished_reqs method.

    * Broadcasts the START_DP_WAVE message to engines to move them from paused
      to running state when one engine receives a new request. This can happen
      in two cases:
      1) A front-end sending a new request while the engines are paused will
         concurrently notify the coordinator.
      2) An engine receiving a request for a stale request wave while in paused
         state will notify the coordinator.

    Engines will move into running state when receiving a new request or
    START_DP_WAVE message.

    Note that when deployed in External LB mode, no stats will be published by
    the engines and thus updates will only be sent to front-ends when the
    request wave / running state changes.
    """
```
**EN:** Introduces the `DPCoordinator` class. Core methods include `_wait_for_zmq_addrs`, `__init__`, `get_stats_publish_address`, `get_engine_socket_addresses`, `shutdown`. Docstring signal: Coordinator process used for data-parallel deployments (DP>1).
**CN:** 这里定义 `DPCoordinator` 类。核心方法包括 `_wait_for_zmq_addrs`, `__init__`, `get_stats_publish_address`, `get_engine_socket_addresses`, `shutdown`。

### `DPCoordinator.__init__` method / `DPCoordinator.__init__` 方法
```python
    def __init__(
        self, parallel_config: ParallelConfig, enable_wave_coordination: bool = True
    ):
        dp_size = parallel_config.data_parallel_size
        assert dp_size > 1, "Coordinator only used for data parallel"

        host = parallel_config.data_parallel_master_ip

        # Assume coordinator is colocated with front-end procs when not in
        # either external or hybrid DP LB mode.
        local_only = not parallel_config.local_engines_only
        local_only_eng = dp_size == parallel_config.data_parallel_size_local
        # NOTE(yongji): handling scaling from intra-node to inter-node
        if parallel_config.enable_elastic_ep:
            local_only_eng = False

        def bind_address(local_only: bool) -> str:
            return (
                get_engine_client_zmq_addr(local_only=True, host=host)
                if local_only
                else get_tcp_uri(host, 0)
            )

        front_publish_address = bind_address(local_only)
        back_publish_address = bind_address(local_only_eng)
        back_output_address = bind_address(local_only_eng)

        context = get_mp_context()
        parent_zmq_addr_pipe, child_zmq_addr_pipe = context.Pipe(duplex=False)
        self.proc: multiprocessing.Process = context.Process(
            target=DPCoordinatorProc.run_coordinator,
            name="VLLM_DP_Coordinator",
            kwargs={
                "engine_count": parallel_config.data_parallel_size,
                "front_publish_address": front_publish_address,
                "back_output_address": back_output_address,
                "back_publish_address": back_publish_address,
                "zmq_addr_pipe": child_zmq_addr_pipe,
                "enable_wave_coordination": enable_wave_coordination,
            },
            daemon=True,
        )
        self.proc.start()
        child_zmq_addr_pipe.close()
        (
            front_publish_address,
            back_output_address,
            back_publish_address,
        ) = self._wait_for_zmq_addrs(parent_zmq_addr_pipe)

        self.stats_publish_address = front_publish_address
        self.coord_in_address = back_publish_address
        self.coord_out_address = back_output_address
        self._finalizer = weakref.finalize(self, shutdown, [self.proc])
```
**EN:** This method initializes the object state within `DPCoordinator`. Key calls include `bind_address`, `get_mp_context`, `Pipe`, `Process`, `start`, `close`. It touches state such as `proc`, `stats_publish_address`, `coord_in_address`, `coord_out_address`, `_finalizer`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`DPCoordinator`。 关键调用包括 `bind_address`, `get_mp_context`, `Pipe`, `Process`, `start`, `close`。 它会读写 `proc`, `stats_publish_address`, `coord_in_address`, `coord_out_address`, `_finalizer` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DPCoordinator.get_engine_socket_addresses` method / `DPCoordinator.get_engine_socket_addresses` 方法
```python
    def get_engine_socket_addresses(self) -> tuple[str, str]:
        """Returns tuple of ZMQ input address, output address."""
        return self.coord_in_address, self.coord_out_address
```
**EN:** This method returns or derives a value within `DPCoordinator`. The docstring frames it as: Returns tuple of ZMQ input address, output address.
**CN:** 该方法会返回或推导一个值，其作用域位于`DPCoordinator`。

### `DPCoordinator.shutdown` method / `DPCoordinator.shutdown` 方法
```python
    def shutdown(self, timeout: float | None = None) -> None:
        """Shutdown coordinator process with configurable timeout."""
        if self._finalizer.detach() is not None:
            shutdown([self.proc], timeout=timeout)
```
**EN:** This method implements `shutdown` within `DPCoordinator`. The docstring frames it as: Shutdown coordinator process with configurable timeout. Key calls include `detach`, `shutdown`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `shutdown`，其作用域位于`DPCoordinator`。 关键调用包括 `detach`, `shutdown`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EngineState` class / `EngineState` 类
```python
class EngineState:
```
**EN:** Introduces the `EngineState` class. Core methods include `__init__`.
**CN:** 这里定义 `EngineState` 类。核心方法包括 `__init__`。

### `EngineState.__init__` method / `EngineState.__init__` 方法
```python
    def __init__(self):
        self.request_counts = [0, 0]  # [waiting, running]
```
**EN:** This method initializes the object state within `EngineState`. It touches state such as `request_counts`.
**CN:** 该方法会初始化对象状态，其作用域位于`EngineState`。 它会读写 `request_counts` 等状态。

### `DPCoordinatorProc` class / `DPCoordinatorProc` 类
```python
class DPCoordinatorProc:
```
**EN:** Introduces the `DPCoordinatorProc` class. Core methods include `__init__`, `run_coordinator`, `process_input_socket`, `_send_start_wave`, `_get_engine_counts`.
**CN:** 这里定义 `DPCoordinatorProc` 类。核心方法包括 `__init__`, `run_coordinator`, `process_input_socket`, `_send_start_wave`, `_get_engine_counts`。

### `DPCoordinatorProc.__init__` method / `DPCoordinatorProc.__init__` 方法
```python
    def __init__(
        self,
        engine_count: int,
        min_stats_update_interval_ms: int = 100,
        enable_wave_coordination: bool = True,
    ):
        set_process_title("DPCoordinator")
        self.ctx = zmq.Context()

        self.engines = [EngineState() for _ in range(engine_count)]

        self.stats_update_interval_ms = min_stats_update_interval_ms
        self.enable_wave_coordination = enable_wave_coordination
```
**EN:** This method initializes the object state within `DPCoordinatorProc`. Key calls include `set_process_title`, `Context`, `EngineState`, `range`. It touches state such as `ctx`, `engines`, `stats_update_interval_ms`, `enable_wave_coordination`.
**CN:** 该方法会初始化对象状态，其作用域位于`DPCoordinatorProc`。 关键调用包括 `set_process_title`, `Context`, `EngineState`, `range`。 它会读写 `ctx`, `engines`, `stats_update_interval_ms`, `enable_wave_coordination` 等状态。

### `DPCoordinatorProc.run_coordinator` method / `DPCoordinatorProc.run_coordinator` 方法
```python
    @staticmethod
    def run_coordinator(
        engine_count: int,
        front_publish_address: str,
        back_output_address: str,
        back_publish_address: str,
        zmq_addr_pipe=None,
        min_stats_update_interval_ms: int = 100,
        enable_wave_coordination: bool = True,
    ):
        coordinator = DPCoordinatorProc(
            engine_count=engine_count,
            min_stats_update_interval_ms=min_stats_update_interval_ms,
            enable_wave_coordination=enable_wave_coordination,
        )
        try:
            coordinator.process_input_socket(
                front_publish_address,
                back_output_address,
                back_publish_address,
                zmq_addr_pipe,
            )
        except KeyboardInterrupt:
            logger.info("DP Coordinator process exiting")
        finally:
            if zmq_addr_pipe is not None:
                zmq_addr_pipe.close()
```
**EN:** This method runs the main execution path within `DPCoordinatorProc`. Key calls include `DPCoordinatorProc`, `process_input_socket`, `info`, `close`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会执行主要运行路径，其作用域位于`DPCoordinatorProc`。 关键调用包括 `DPCoordinatorProc`, `process_input_socket`, `info`, `close`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DPCoordinatorProc.process_input_socket` method / `DPCoordinatorProc.process_input_socket` 方法
```python
    def process_input_socket(
        self,
        front_publish_address: str,
        back_output_address: str,
        back_publish_address: str,
        zmq_addr_pipe=None,
    ):
        decoder = MsgpackDecoder(EngineCoreOutputs)

        # For tracking request wave progression.
        current_wave = 0
        engines_running = False

        # For tracking request counts for internal load-balancing.
        stats_changed = False
        last_stats_step = -1
        last_stats_wave = -1
        last_step_counts: list[list[int]] | None = None

        with (
            make_zmq_socket(
                path=front_publish_address,  # IPC
                ctx=self.ctx,
                socket_type=zmq.XPUB,
                bind=True,
            ) as publish_front,
            make_zmq_socket(
                path=back_output_address,  # IPC or TCP
                ctx=self.ctx,
                socket_type=zmq.PULL,
                bind=True,
            ) as output_back,
            make_zmq_socket(
                path=back_publish_address,  # IPC or TCP
                ctx=self.ctx,
                socket_type=zmq.XPUB,
                bind=True,
            ) as publish_back,
        ):
            if zmq_addr_pipe is not None:
                try:
                    zmq_addr_pipe.send(
                        (
                            publish_front.getsockopt(zmq.LAST_ENDPOINT).decode(),
                            output_back.getsockopt(zmq.LAST_ENDPOINT).decode(),
                            publish_back.getsockopt(zmq.LAST_ENDPOINT).decode(),
                        )
                    )
                finally:
                    zmq_addr_pipe.close()
    # ... omitted for brevity ...
                                    new_wave,
                                )
                                current_wave = new_wave
                                engines_running = False
                                wave_state_changed = True
                        elif (wave := outputs.start_wave) is not None and (
                            wave > current_wave
                            or (wave == current_wave and not engines_running)
                        ):
                            # 3. The engine received request for a non-current wave
                            # so we must ensure that other engines progress to the
                            # next wave (race condition handling).
                            logger.debug(
                                "Starting wave %d after notification of "
                                "stale wave request from engine.",
                                wave,
                            )
                            current_wave = wave
                            engines_running = True
                            wave_state_changed = True
                            self._send_start_wave(publish_back, wave, eng_index)

                if wave_state_changed:
                    message = (None, current_wave, engines_running)
                    publish_front.send(msgspec.msgpack.encode(message))
```
**EN:** This method implements `process_input_socket` within `DPCoordinatorProc`. Key calls include `MsgpackDecoder`, `make_zmq_socket`, `send`, `info`, `Poller`, `register`. It touches state such as `engines`. The control flow contains 27 branch(es) and 3 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `process_input_socket`，其作用域位于`DPCoordinatorProc`。 关键调用包括 `MsgpackDecoder`, `make_zmq_socket`, `send`, `info`, `Poller`, `register`。 它会读写 `engines` 等状态。 控制流包含 27 个分支和 3 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

## Key Concepts / 关键概念
- `DPCoordinator`: central class or interface in this module. / `DPCoordinator`：本模块中的核心类或接口。
- `EngineState`: central class or interface in this module. / `EngineState`：本模块中的核心类或接口。
- `DPCoordinatorProc`: central class or interface in this module. / `DPCoordinatorProc`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `copy`, `multiprocessing`, `time`, `weakref`
- External / 外部依赖: `msgspec`, `zmq`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.logger`, `vllm.utils.network_utils`, `vllm.utils.system_utils`, `vllm.v1.engine`, `vllm.v1.serial_utils`, `vllm.v1.utils`
