# uniproc_executor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/executor/uniproc_executor.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `AsyncOutputFuture`, `UniProcExecutor`, `ExecutorWithExternalLauncher` for the V1 `executor` subsystem. / 为 V1 的 `executor` 子系统实现 `AsyncOutputFuture`, `UniProcExecutor`, `ExecutorWithExternalLauncher`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import os
from collections.abc import Callable
from concurrent.futures import Future
from functools import cached_property
from multiprocessing import Lock
from typing import Any

import torch
import torch.distributed as dist

import vllm.envs as envs
from vllm.logger import init_logger
from vllm.platforms import current_platform
from vllm.utils.network_utils import get_distributed_init_method, get_ip, get_open_port
from vllm.v1.core.sched.output import GrammarOutput, SchedulerOutput
from vllm.v1.executor.abstract import Executor
from vllm.v1.outputs import AsyncModelRunnerOutput, DraftTokenIds, ModelRunnerOutput
from vllm.v1.serial_utils import run_method
from vllm.v1.worker.worker_base import WorkerWrapperBase

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `AsyncOutputFuture` class / `AsyncOutputFuture` 类
```python
class AsyncOutputFuture(Future):
```
**EN:** Introduces the `AsyncOutputFuture` class on top of `Future`. Core methods include `__init__`, `result`.
**CN:** 这里定义 `AsyncOutputFuture` 类，其基类包括 `Future`。核心方法包括 `__init__`, `result`。

### `AsyncOutputFuture.__init__` method / `AsyncOutputFuture.__init__` 方法
```python
    def __init__(self, async_output: AsyncModelRunnerOutput, single_value: bool):
        self.async_output = async_output
        self.single_value = single_value
        super().__init__()
```
**EN:** This method initializes the object state within `AsyncOutputFuture`. Key calls include `__init__`, `super`. It touches state such as `async_output`, `single_value`.
**CN:** 该方法会初始化对象状态，其作用域位于`AsyncOutputFuture`。 关键调用包括 `__init__`, `super`。 它会读写 `async_output`, `single_value` 等状态。

### `AsyncOutputFuture.result` method / `AsyncOutputFuture.result` 方法
```python
    def result(self, timeout=None):
        if timeout is not None:
            raise RuntimeError("timeout not implemented")

        if not super().done():
            try:
                output = self.async_output.get_output()
                self.set_result(output if self.single_value else [output])
            except Exception as e:
                self.set_exception(e)
        return super().result()
```
**EN:** This method implements `result` within `AsyncOutputFuture`. Key calls include `result`, `RuntimeError`, `done`, `get_output`, `set_result`, `super`. The control flow contains 4 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `result`，其作用域位于`AsyncOutputFuture`。 关键调用包括 `result`, `RuntimeError`, `done`, `get_output`, `set_result`, `super`。 控制流包含 4 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `UniProcExecutor` class / `UniProcExecutor` 类
```python
class UniProcExecutor(Executor):
```
**EN:** Introduces the `UniProcExecutor` class on top of `Executor`. Core methods include `_init_executor`, `_distributed_args`, `max_concurrent_batches`, `collective_rpc`, `execute_model`, `sample_tokens`.
**CN:** 这里定义 `UniProcExecutor` 类，其基类包括 `Executor`。核心方法包括 `_init_executor`, `_distributed_args`, `max_concurrent_batches`, `collective_rpc`, `execute_model`, `sample_tokens`。

### `UniProcExecutor.max_concurrent_batches` method / `UniProcExecutor.max_concurrent_batches` 方法
```python
    @cached_property
    def max_concurrent_batches(self) -> int:
        return 2 if self.scheduler_config.async_scheduling else 1
```
**EN:** This method implements `max_concurrent_batches` within `UniProcExecutor`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `max_concurrent_batches`，其作用域位于`UniProcExecutor`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `UniProcExecutor.collective_rpc` method / `UniProcExecutor.collective_rpc` 方法
```python
    def collective_rpc(  # type: ignore[override]
        self,
        method: str | Callable,
        timeout: float | None = None,
        args: tuple = (),
        kwargs: dict | None = None,
        non_block: bool = False,
        single_value: bool = False,
    ) -> Any:
        if kwargs is None:
            kwargs = {}

        if not non_block:
            result = run_method(self.driver_worker, method, args, kwargs)
            return result if single_value else [result]

        try:
            result = run_method(self.driver_worker, method, args, kwargs)
            if isinstance(result, AsyncModelRunnerOutput):
                return AsyncOutputFuture(result, single_value)
            future = Future[Any]()
            future.set_result(result if single_value else [result])
        except Exception as e:
            future = Future[Any]()
            future.set_exception(e)
        return future
```
**EN:** This method collects data from collaborators within `UniProcExecutor`. Key calls include `run_method`, `isinstance`, `set_result`, `AsyncOutputFuture`, `set_exception`. The control flow contains 6 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会从协作组件收集数据，其作用域位于`UniProcExecutor`。 关键调用包括 `run_method`, `isinstance`, `set_result`, `AsyncOutputFuture`, `set_exception`。 控制流包含 6 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `UniProcExecutor.execute_model` method / `UniProcExecutor.execute_model` 方法
```python
    def execute_model(  # type: ignore[override]
        self, scheduler_output: SchedulerOutput, non_block: bool = False
    ) -> ModelRunnerOutput | None | Future[ModelRunnerOutput | None]:
        output = self.collective_rpc(
            "execute_model",
            args=(scheduler_output,),
            non_block=non_block,
            single_value=True,
        )
        # In non-blocking mode, surface any exception as early as possible.
        if non_block and output.done():
            # Raise the exception in-line if the task failed.
            output.result()
        return output
```
**EN:** This method executes the main operation within `UniProcExecutor`. Key calls include `collective_rpc`, `done`, `result`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会执行主要操作，其作用域位于`UniProcExecutor`。 关键调用包括 `collective_rpc`, `done`, `result`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `UniProcExecutor.sample_tokens` method / `UniProcExecutor.sample_tokens` 方法
```python
    def sample_tokens(  # type: ignore[override]
        self, grammar_output: GrammarOutput | None, non_block: bool = False
    ) -> ModelRunnerOutput | None | Future[ModelRunnerOutput | None]:
        return self.collective_rpc(
            "sample_tokens",
            args=(grammar_output,),
            non_block=non_block,
            single_value=True,
        )
```
**EN:** This method samples outputs from model state within `UniProcExecutor`. Key calls include `collective_rpc`.
**CN:** 该方法会从模型状态中采样输出，其作用域位于`UniProcExecutor`。 关键调用包括 `collective_rpc`。

### `UniProcExecutor.take_draft_token_ids` method / `UniProcExecutor.take_draft_token_ids` 方法
```python
    def take_draft_token_ids(self) -> DraftTokenIds | None:
        return self.collective_rpc("take_draft_token_ids", single_value=True)
```
**EN:** This method implements `take_draft_token_ids` within `UniProcExecutor`. Key calls include `collective_rpc`.
**CN:** 该方法会实现 `take_draft_token_ids`，其作用域位于`UniProcExecutor`。 关键调用包括 `collective_rpc`。

### `UniProcExecutor.check_health` method / `UniProcExecutor.check_health` 方法
```python
    def check_health(self) -> None:
        # UniProcExecutor will always be healthy as long as
        # it's running.
        return
```
**EN:** This method validates assumptions or constraints within `UniProcExecutor`.
**CN:** 该方法会校验前提与约束，其作用域位于`UniProcExecutor`。

### `UniProcExecutor.shutdown` method / `UniProcExecutor.shutdown` 方法
```python
    def shutdown(self) -> None:
        if worker := self.driver_worker:
            worker.shutdown()
```
**EN:** This method implements `shutdown` within `UniProcExecutor`. Key calls include `shutdown`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `shutdown`，其作用域位于`UniProcExecutor`。 关键调用包括 `shutdown`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `UniProcExecutor.supports_async_scheduling` method / `UniProcExecutor.supports_async_scheduling` 方法
```python
    @classmethod
    def supports_async_scheduling(cls) -> bool:
        return True
```
**EN:** This method implements `supports_async_scheduling` within `UniProcExecutor`.
**CN:** 该方法会实现 `supports_async_scheduling`，其作用域位于`UniProcExecutor`。

### `ExecutorWithExternalLauncher` class / `ExecutorWithExternalLauncher` 类
```python
class ExecutorWithExternalLauncher(UniProcExecutor):
    """An executor that uses external launchers to launch engines,
    specially designed for torchrun-compatible launchers, for
    offline inference with tensor parallelism.

    see https://github.com/vllm-project/vllm/issues/11400 for
    the motivation, and examples/features/torchrun/torchrun_example_offline.py
    for the usage example.

    The key idea: although it is tensor-parallel inference, we only
    create one worker per executor, users will launch multiple
    engines with torchrun-compatible launchers, and all these engines
    work together to process the same prompts. When scheduling is
    deterministic, all the engines will generate the same outputs,
    and they don't need to synchronize the states with each other.
    """
```
**EN:** Introduces the `ExecutorWithExternalLauncher` class on top of `UniProcExecutor`. Core methods include `_init_executor`, `_distributed_args`, `determine_available_memory`. Docstring signal: An executor that uses external launchers to launch engines, specially designed for torchrun-compatible launchers, for offline inference with tensor parallelism.
**CN:** 这里定义 `ExecutorWithExternalLauncher` 类，其基类包括 `UniProcExecutor`。核心方法包括 `_init_executor`, `_distributed_args`, `determine_available_memory`。

### `ExecutorWithExternalLauncher._init_executor` method / `ExecutorWithExternalLauncher._init_executor` 方法
```python
    def _init_executor(self) -> None:
        """Initialize the worker and load the model."""
        assert not envs.VLLM_ENABLE_V1_MULTIPROCESSING, (
            "To get deterministic execution, "
            "please set VLLM_ENABLE_V1_MULTIPROCESSING=0"
        )
        super()._init_executor()
```
**EN:** This method implements `_init_executor` within `ExecutorWithExternalLauncher`. The docstring frames it as: Initialize the worker and load the model. Key calls include `_init_executor`, `super`.
**CN:** 该方法会实现 `_init_executor`，其作用域位于`ExecutorWithExternalLauncher`。 关键调用包括 `_init_executor`, `super`。

### `ExecutorWithExternalLauncher._distributed_args` method / `ExecutorWithExternalLauncher._distributed_args` 方法
```python
    def _distributed_args(self) -> tuple[str, int, int]:
        # engines are launched in torchrun-compatible launchers
        # so we can use the env:// method.
        # required env vars:
        # - RANK
        # - LOCAL_RANK
        # - MASTER_ADDR
        # - MASTER_PORT
        distributed_init_method = "env://"
        rank = int(os.environ["RANK"])
        local_rank = int(os.environ["LOCAL_RANK"])
        return distributed_init_method, rank, local_rank
```
**EN:** This method implements `_distributed_args` within `ExecutorWithExternalLauncher`. Key calls include `int`.
**CN:** 该方法会实现 `_distributed_args`，其作用域位于`ExecutorWithExternalLauncher`。 关键调用包括 `int`。

### `ExecutorWithExternalLauncher.determine_available_memory` method / `ExecutorWithExternalLauncher.determine_available_memory` 方法
```python
    def determine_available_memory(self) -> list[int]:  # in bytes
        # we need to get the min across all ranks.
        memory = super().determine_available_memory()
        from vllm.distributed.parallel_state import get_world_group

        cpu_group = get_world_group().cpu_group
        memory_tensor = torch.tensor([memory], device="cpu", dtype=torch.int64)
        dist.all_reduce(memory_tensor, group=cpu_group, op=dist.ReduceOp.MIN)
        return [memory_tensor.item()]
```
**EN:** This method implements `determine_available_memory` within `ExecutorWithExternalLauncher`. Key calls include `determine_available_memory`, `tensor`, `all_reduce`, `get_world_group`, `item`, `super`.
**CN:** 该方法会实现 `determine_available_memory`，其作用域位于`ExecutorWithExternalLauncher`。 关键调用包括 `determine_available_memory`, `tensor`, `all_reduce`, `get_world_group`, `item`, `super`。

## Key Concepts / 关键概念
- `AsyncOutputFuture`: central class or interface in this module. / `AsyncOutputFuture`：本模块中的核心类或接口。
- `UniProcExecutor`: central class or interface in this module. / `UniProcExecutor`：本模块中的核心类或接口。
- `ExecutorWithExternalLauncher`: central class or interface in this module. / `ExecutorWithExternalLauncher`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `os`, `collections`, `concurrent`, `functools`, `multiprocessing`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.logger`, `vllm.platforms`, `vllm.utils.network_utils`, `vllm.v1.core.sched.output`, `vllm.v1.executor.abstract`, `vllm.v1.outputs`, `vllm.v1.serial_utils`, `vllm.v1.worker.worker_base`, `vllm.distributed.parallel_state`
