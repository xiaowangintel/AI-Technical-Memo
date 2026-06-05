# ray_executor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/executor/ray_executor.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `RayWorkerMetaData`, `RayDistributedExecutor` for the V1 `executor` subsystem. / 为 V1 的 `executor` 子系统实现 `RayWorkerMetaData`, `RayDistributedExecutor`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import os
from collections import defaultdict
from collections.abc import Callable
from concurrent.futures import Future
from dataclasses import dataclass
from typing import TYPE_CHECKING, Any

import cloudpickle

import vllm.envs as envs
from vllm.logger import init_logger
from vllm.platforms import current_platform
from vllm.ray.ray_env import get_env_vars_to_copy
from vllm.utils.network_utils import (
    get_distributed_init_method,
    get_ip,
    get_open_port,
)
from vllm.v1.core.sched.output import GrammarOutput, SchedulerOutput
from vllm.v1.engine import ReconfigureDistributedRequest, ReconfigureRankType
from vllm.v1.executor.abstract import Executor
from vllm.v1.executor.ray_utils import (
    WORKER_SPECIFIC_ENV_VARS,
    FutureWrapper,
    RayWorkerWrapper,
    detach_zero_copy_from_model_runner_output,
    initialize_ray_cluster,
    ray,
)
from vllm.v1.outputs import ModelRunnerOutput

if ray is not None:
    from ray.actor import ActorHandle
    from ray.util.scheduling_strategies import PlacementGroupSchedulingStrategy
else:
    ActorHandle = None

if TYPE_CHECKING:
    from ray.util.placement_group import PlacementGroup

logger = init_logger(__name__)

COMPLETED_NONE_FUTURE: Future[ModelRunnerOutput | None] = Future()
COMPLETED_NONE_FUTURE.set_result(None)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `COMPLETED_NONE_FUTURE`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `COMPLETED_NONE_FUTURE`。

### `RayWorkerMetaData` class / `RayWorkerMetaData` 类
```python
@dataclass
class RayWorkerMetaData:
    """
    Metadata for a Ray worker.
    The order of ray worker creation can be random,
    and we need to reset the rank after creating all workers.
    """

    worker: ActorHandle
    created_rank: int
    adjusted_rank: int = -1
    ip: str = ""
```
**EN:** Uses `@dataclass` to package related state for `RayWorkerMetaData`. Typical fields include `worker`, `created_rank`, `adjusted_rank`, `ip`.
**CN:** `RayWorkerMetaData` 使用 `@dataclass` 打包相关状态。典型字段包括 `worker`, `created_rank`, `adjusted_rank`, `ip`。

### `RayDistributedExecutor` class / `RayDistributedExecutor` 类
```python
class RayDistributedExecutor(Executor):
    """Ray-based distributed executor"""

    uses_ray: bool = True
    supports_pp: bool = True
```
**EN:** Introduces the `RayDistributedExecutor` class on top of `Executor`. Core methods include `_init_executor`, `max_concurrent_batches`, `shutdown`, `_configure_ray_workers_use_nsight`, `_update_noset_device_env_vars`, `_get_env_vars_to_be_updated`. Docstring signal: Ray-based distributed executor
**CN:** 这里定义 `RayDistributedExecutor` 类，其基类包括 `Executor`。核心方法包括 `_init_executor`, `max_concurrent_batches`, `shutdown`, `_configure_ray_workers_use_nsight`, `_update_noset_device_env_vars`, `_get_env_vars_to_be_updated`。

### `RayDistributedExecutor.execute_model` method / `RayDistributedExecutor.execute_model` 方法
```python
    def execute_model(  # type: ignore[override]
        self,
        scheduler_output: SchedulerOutput,
        non_block: bool = False,
    ) -> ModelRunnerOutput | None | Future[ModelRunnerOutput | None]:
        if self.scheduler_output is not None:
            raise RuntimeError(
                "State error: sample_tokens() must be called "
                "after execute_model() returns None."
            )

        if not self.uses_sampler or not scheduler_output.total_num_scheduled_tokens:
            # Model will not execute, call model runner immediately.
            return self._execute_dag(scheduler_output, None, non_block)

        # Model will execute, defer to sample_tokens() call.
        self.scheduler_output = scheduler_output
        return COMPLETED_NONE_FUTURE if non_block else None
```
**EN:** This method executes the main operation within `RayDistributedExecutor`. Key calls include `RuntimeError`, `_execute_dag`. It touches state such as `scheduler_output`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会执行主要操作，其作用域位于`RayDistributedExecutor`。 关键调用包括 `RuntimeError`, `_execute_dag`。 它会读写 `scheduler_output` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RayDistributedExecutor._execute_dag` method / `RayDistributedExecutor._execute_dag` 方法
```python
    def _execute_dag(
        self,
        scheduler_output: SchedulerOutput,
        grammar_output: "GrammarOutput | None",
        non_block: bool = False,
    ) -> ModelRunnerOutput | None | Future[ModelRunnerOutput | None]:
        # Build the compiled DAG for the first time.
        if self.forward_dag is None:  # type: ignore
            self.forward_dag = self._compiled_ray_dag(enable_asyncio=False)

        refs = self.forward_dag.execute((scheduler_output, grammar_output))  # type: ignore

        if not self.has_connector:
            # Get output only from a single worker (output_rank)
            # When PP is not used, we block here until the result is available.
            if not non_block:
                output = refs[0].get()
                detach_zero_copy_from_model_runner_output(output)
                return output

            # When PP is used, we return a FutureWrapper immediately so that
            # the scheduler can yield to the next batch.
            return FutureWrapper(refs[0])

        # Get output from all workers when connector is present
        assert self.kv_output_aggregator is not None
        if not non_block:
            # Block and get results from all workers
            outputs = ray.get(refs)
            for output in outputs:
                detach_zero_copy_from_model_runner_output(output)
            return self.kv_output_aggregator.aggregate(outputs)

        # Return a future that will aggregate outputs from all workers
        return FutureWrapper(refs, self.kv_output_aggregator)
```
**EN:** This method implements `_execute_dag` within `RayDistributedExecutor`. Key calls include `execute`, `FutureWrapper`, `_compiled_ray_dag`, `get`, `aggregate`, `detach_zero_copy_from_model_runner_output`. It touches state such as `forward_dag`. The control flow contains 4 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_execute_dag`，其作用域位于`RayDistributedExecutor`。 关键调用包括 `execute`, `FutureWrapper`, `_compiled_ray_dag`, `get`, `aggregate`, `detach_zero_copy_from_model_runner_output`。 它会读写 `forward_dag` 等状态。 控制流包含 4 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `RayDistributedExecutor.collective_rpc` method / `RayDistributedExecutor.collective_rpc` 方法
```python
    def collective_rpc(  # type: ignore[override]
        self,
        method: str | Callable,
        timeout: float | None = None,
        args: tuple = (),
        kwargs: dict[str, Any] | None = None,
        non_block: bool = False,
    ) -> list[Any] | Future[list[Any]]:
        """Runs the given method on all workers."""
        sent_method = method if isinstance(method, str) else cloudpickle.dumps(method)
        del method

        if kwargs is None:
            kwargs = {}
        ray_worker_outputs = [
            worker.execute_method.remote(  # type: ignore[attr-defined]
                sent_method, *args, **kwargs
            )
            for worker in self.workers
        ]

        # Get the results of the ray workers.
        if non_block:
            return FutureWrapper(ray_worker_outputs)

        return ray.get(ray_worker_outputs, timeout=timeout)
```
**EN:** This method collects data from collaborators within `RayDistributedExecutor`. The docstring frames it as: Runs the given method on all workers. Key calls include `get`, `isinstance`, `dumps`, `remote`, `FutureWrapper`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会从协作组件收集数据，其作用域位于`RayDistributedExecutor`。 关键调用包括 `get`, `isinstance`, `dumps`, `remote`, `FutureWrapper`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `RayWorkerMetaData`: central class or interface in this module. / `RayWorkerMetaData`：本模块中的核心类或接口。
- `RayDistributedExecutor`: central class or interface in this module. / `RayDistributedExecutor`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `os`, `collections`, `concurrent`, `dataclasses`, `typing`, `importlib`
- External / 外部依赖: `cloudpickle`, `ray`, `packaging`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.logger`, `vllm.platforms`, `vllm.ray.ray_env`, `vllm.utils.network_utils`, `vllm.v1.core.sched.output`, `vllm.v1.engine`, `vllm.v1.executor.abstract`, `vllm.v1.executor.ray_utils`, `vllm.v1.outputs`, `vllm.distributed.device_communicators.ray_communicator`
