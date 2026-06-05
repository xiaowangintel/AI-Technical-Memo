# async_llm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/engine/async_llm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `InputStreamError`, `AsyncLLM` for the V1 `engine` subsystem. / 为 V1 的 `engine` 子系统实现 `InputStreamError`, `AsyncLLM`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import asyncio
import os
import socket
import time
import warnings
from collections.abc import AsyncGenerator, Iterable, Mapping
from copy import copy
from typing import Any

import torch

import vllm.envs as envs
from vllm import TokensPrompt
from vllm.config import VllmConfig
from vllm.distributed.weight_transfer.base import (
    WeightTransferInitRequest,
    WeightTransferUpdateRequest,
)
from vllm.engine.arg_utils import AsyncEngineArgs
from vllm.engine.protocol import EngineClient, StreamingInput
from vllm.entrypoints.serve.elastic_ep.middleware import set_scaling_elastic_ep
from vllm.inputs import EngineInput, PromptType
from vllm.logger import init_logger
from vllm.lora.request import LoRARequest
from vllm.multimodal import MULTIMODAL_REGISTRY, MultiModalRegistry
from vllm.outputs import STREAM_FINISHED, PoolingRequestOutput, RequestOutput
from vllm.pooling_params import PoolingParams
from vllm.renderers import renderer_from_config
from vllm.renderers.inputs.preprocess import extract_prompt_components
from vllm.sampling_params import RequestOutputKind, SamplingParams
from vllm.tasks import SupportedTask
from vllm.tokenizers import TokenizerLike
from vllm.tracing import init_tracer
from vllm.transformers_utils.config import maybe_register_config_serialize_by_value
from vllm.usage.usage_lib import UsageContext
from vllm.utils.async_utils import cancel_task_threadsafe
from vllm.utils.collection_utils import as_list
from vllm.v1.engine import EngineCoreRequest, PauseMode
from vllm.v1.engine.core_client import EngineCoreClient
from vllm.v1.engine.exceptions import EngineDeadError, EngineGenerateError
from vllm.v1.engine.input_processor import InputProcessor
from vllm.v1.engine.output_processor import OutputProcessor, RequestOutputCollector
from vllm.v1.engine.parallel_sampling import ParentRequest
from vllm.v1.executor import Executor
from vllm.v1.metrics.loggers import (
    StatLoggerFactory,
    StatLoggerManager,
    load_stat_logger_plugin_factories,
)
from vllm.v1.metrics.prometheus import shutdown_prometheus
from vllm.v1.metrics.stats import IterationStats

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `InputStreamError` class / `InputStreamError` 类
```python
class InputStreamError(Exception):
    """Wrapper for errors from the input stream generator.

    This is used to propagate errors from the user's input generator
    without wrapping them in EngineGenerateError.
    """
```
**EN:** Introduces the `InputStreamError` class on top of `Exception`. Core methods include `__init__`. Docstring signal: Wrapper for errors from the input stream generator.
**CN:** 这里定义 `InputStreamError` 类，其基类包括 `Exception`。核心方法包括 `__init__`。

### `InputStreamError.__init__` method / `InputStreamError.__init__` 方法
```python
    def __init__(self, cause: Exception):
        self.cause = cause
        super().__init__(str(cause))
```
**EN:** This method initializes the object state within `InputStreamError`. Key calls include `__init__`, `str`, `super`. It touches state such as `cause`.
**CN:** 该方法会初始化对象状态，其作用域位于`InputStreamError`。 关键调用包括 `__init__`, `str`, `super`。 它会读写 `cause` 等状态。

### `AsyncLLM` class / `AsyncLLM` 类
```python
class AsyncLLM(EngineClient):
    """An asynchronous wrapper for the vLLM engine."""
```
**EN:** Introduces the `AsyncLLM` class on top of `EngineClient`. Core methods include `__init__`, `from_vllm_config`, `from_engine_args`, `__del__`, `shutdown`, `get_supported_tasks`. Docstring signal: An asynchronous wrapper for the vLLM engine.
**CN:** 这里定义 `AsyncLLM` 类，其基类包括 `EngineClient`。核心方法包括 `__init__`, `from_vllm_config`, `from_engine_args`, `__del__`, `shutdown`, `get_supported_tasks`。

### `AsyncLLM.__init__` method / `AsyncLLM.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        executor_class: type[Executor],
        log_stats: bool,
        usage_context: UsageContext = UsageContext.ENGINE_CONTEXT,
        mm_registry: MultiModalRegistry = MULTIMODAL_REGISTRY,
        log_requests: bool = True,
        start_engine_loop: bool = True,
        stat_loggers: list[StatLoggerFactory] | None = None,
        aggregate_engine_logging: bool = False,
        client_addresses: dict[str, str] | None = None,
        client_count: int = 1,
        client_index: int = 0,
    ) -> None:
        """
        Create an AsyncLLM.

        Args:
            vllm_config: global configuration.
            executor_class: an Executor impl, e.g. MultiprocExecutor.
            log_stats: Whether to log stats.
            usage_context: Usage context of the LLM.
            mm_registry: Multi-modal registry.
            log_requests: Whether to log requests.
            start_engine_loop: Whether to start the engine loop.
            stat_loggers: customized stat loggers for the engine.
                If not provided, default stat loggers will be used.
                PLEASE BE AWARE THAT STAT LOGGER IS NOT STABLE
                IN V1, AND ITS BASE CLASS INTERFACE MIGHT CHANGE.

        Returns:
            None
        """
        # Ensure we can serialize custom transformer configs
        maybe_register_config_serialize_by_value()

        self.vllm_config = vllm_config
        self.model_config = vllm_config.model_config
        self.observability_config = vllm_config.observability_config

        tracing_endpoint = self.observability_config.otlp_traces_endpoint
        if tracing_endpoint is not None:
            init_tracer("vllm.llm_engine", tracing_endpoint)

        self.log_requests = log_requests

        custom_stat_loggers = list(stat_loggers or [])
        custom_stat_loggers.extend(load_stat_logger_plugin_factories())

    # ... omitted for brevity ...
            pass

        if (
            vllm_config.profiler_config.profiler == "torch"
            and not vllm_config.profiler_config.ignore_frontend
        ):
            profiler_dir = vllm_config.profiler_config.torch_profiler_dir
            logger.info(
                "Torch profiler enabled. AsyncLLM CPU traces will be collected under %s",  # noqa: E501
                profiler_dir,
            )
            worker_name = f"{socket.gethostname()}_{os.getpid()}.async_llm"
            self.profiler = torch.profiler.profile(
                activities=[
                    torch.profiler.ProfilerActivity.CPU,
                ],
                with_stack=vllm_config.profiler_config.torch_profiler_with_stack,
                on_trace_ready=torch.profiler.tensorboard_trace_handler(
                    profiler_dir,
                    worker_name=worker_name,
                    use_gzip=vllm_config.profiler_config.torch_profiler_use_gzip,
                ),
            )
        else:
            self.profiler = None
```
**EN:** This method initializes the object state within `AsyncLLM`. The docstring frames it as: Create an AsyncLLM. Key calls include `maybe_register_config_serialize_by_value`, `list`, `extend`, `bool`, `renderer_from_config`, `InputProcessor`. It touches state such as `vllm_config`, `model_config`, `observability_config`, `log_requests`, `log_stats`, `renderer`, `input_processor`, `output_processor`. The control flow contains 5 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会初始化对象状态，其作用域位于`AsyncLLM`。 关键调用包括 `maybe_register_config_serialize_by_value`, `list`, `extend`, `bool`, `renderer_from_config`, `InputProcessor`。 它会读写 `vllm_config`, `model_config`, `observability_config`, `log_requests`, `log_stats`, `renderer`, `input_processor`, `output_processor` 等状态。 控制流包含 5 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `AsyncLLM.add_request` method / `AsyncLLM.add_request` 方法
```python
    async def add_request(
        self,
        request_id: str,
        prompt: EngineCoreRequest
        | PromptType
        | EngineInput
        | AsyncGenerator[StreamingInput, None],
        params: SamplingParams | PoolingParams,
        arrival_time: float | None = None,
        lora_request: LoRARequest | None = None,
        tokenization_kwargs: dict[str, Any] | None = None,
        trace_headers: Mapping[str, str] | None = None,
        priority: int = 0,
        data_parallel_rank: int | None = None,
        prompt_text: str | None = None,
        reasoning_ended: bool | None = None,
        reasoning_parser_kwargs: dict[str, Any] | None = None,
    ) -> RequestOutputCollector:
        """Add new request to the AsyncLLM."""

        if self.errored:
            raise EngineDeadError()

        is_pooling = isinstance(params, PoolingParams)

        if (
            self.vllm_config.cache_config.kv_sharing_fast_prefill
            and not is_pooling
            and params.prompt_logprobs
        ):
            raise ValueError(
                "--kv-sharing-fast-prefill produces incorrect logprobs for "
                "prompt tokens, please disable it when the requests need "
                "prompt logprobs"
            )

        if isinstance(prompt, AsyncGenerator):
            if reasoning_ended is not None or reasoning_parser_kwargs is not None:
                raise NotImplementedError

            # Streaming input case.
            return await self._add_streaming_input_request(
                request_id,
                prompt,
                params,
                arrival_time,
                lora_request,
                tokenization_kwargs,
                trace_headers,
                priority,
    # ... omitted for brevity ...

        # Create a new output collector for the request.
        queue = RequestOutputCollector(params.output_kind, request.request_id)

        # Use cloned params that may have been updated in process_inputs()
        params = request.params

        if is_pooling or params.n == 1:
            await self._add_request(request, prompt_text, None, 0, queue)
            return queue

        parent_params = params
        assert isinstance(parent_params, SamplingParams)

        # Fan out child requests (for n>1).
        parent_request = ParentRequest(request)
        for idx in range(parent_params.n):
            request_id, child_params = parent_request.get_child_info(idx)
            child_request = request if idx == parent_params.n - 1 else copy(request)
            child_request.request_id = request_id
            child_request.sampling_params = child_params
            await self._add_request(
                child_request, prompt_text, parent_request, idx, queue
            )
        return queue
```
**EN:** This method implements `add_request` within `AsyncLLM`. The docstring frames it as: Add new request to the AsyncLLM. Key calls include `isinstance`, `assign_request_id`, `_run_output_handler`, `RequestOutputCollector`, `ParentRequest`, `range`. The control flow contains 10 branch(es) and 1 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `add_request`，其作用域位于`AsyncLLM`。 关键调用包括 `isinstance`, `assign_request_id`, `_run_output_handler`, `RequestOutputCollector`, `ParentRequest`, `range`。 控制流包含 10 个分支和 1 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `AsyncLLM.generate` method / `AsyncLLM.generate` 方法
```python
    async def generate(
        self,
        prompt: EngineCoreRequest
        | PromptType
        | EngineInput
        | AsyncGenerator[StreamingInput, None],
        sampling_params: SamplingParams,
        request_id: str,
        *,
        prompt_text: str | None = None,
        lora_request: LoRARequest | None = None,
        tokenization_kwargs: dict[str, Any] | None = None,
        trace_headers: Mapping[str, str] | None = None,
        priority: int = 0,
        data_parallel_rank: int | None = None,
        reasoning_ended: bool | None = None,
        reasoning_parser_kwargs: dict[str, Any] | None = None,
    ) -> AsyncGenerator[RequestOutput, None]:
        """
        Main function called by the API server to kick off a request
            * 1) Making an AsyncStream corresponding to the Request.
            * 2) Processing the Input.
            * 3) Adding the Request to the Detokenizer.
            * 4) Adding the Request to the EngineCore (separate process).

        A separate output_handler loop runs in a background AsyncIO task,
        pulling outputs from EngineCore and putting them into the
        per-request AsyncStream.

        The caller of generate() iterates the returned AsyncGenerator,
        returning the RequestOutput back to the caller.
        """

        q: RequestOutputCollector | None = None
        try:
            q = await self.add_request(
                request_id,
                prompt,
                sampling_params,
                lora_request=lora_request,
                tokenization_kwargs=tokenization_kwargs,
                trace_headers=trace_headers,
                priority=priority,
                data_parallel_rank=data_parallel_rank,
                prompt_text=prompt_text,
                reasoning_ended=reasoning_ended,
                reasoning_parser_kwargs=reasoning_parser_kwargs,
            )

            # The output_handler task pushes items into the queue.
    # ... omitted for brevity ...
        except InputStreamError as e:
            if q is not None:
                await self.abort(q.request_id, internal=True)
            if self.log_requests:
                logger.info("Request %s failed (input error): %s.", request_id, e)
            raise e.cause from e

        # Unexpected error in the generate() task (possibly recoverable).
        except Exception as e:
            if q is not None:
                await self.abort(q.request_id, internal=True)
            if self.log_requests:
                try:
                    s = f"{e.__class__.__name__}: {e}"
                except Exception as e2:
                    s = (
                        f"{e.__class__.__name__}: "
                        "error during printing an exception of class"
                        + e2.__class__.__name__
                    )
                logger.info("Request %s failed due to %s.", request_id, s)
            raise EngineGenerateError() from e
        finally:
            if q is not None:
                q.close()
```
**EN:** This method implements `generate` within `AsyncLLM`. The docstring frames it as: Main function called by the API server to kick off a request * 1) Making an AsyncStream corresponding to the Request. Key calls include `add_request`, `isinstance`, `EngineGenerateError`, `close`, `get_nowait`, `info`. The control flow contains 12 branch(es) and 1 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `generate`，其作用域位于`AsyncLLM`。 关键调用包括 `add_request`, `isinstance`, `EngineGenerateError`, `close`, `get_nowait`, `info`。 控制流包含 12 个分支和 1 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `AsyncLLM._run_output_handler` method / `AsyncLLM._run_output_handler` 方法
```python
    def _run_output_handler(self):
        """Background loop: pulls from EngineCore and pushes to AsyncStreams."""

        if self.output_handler is not None:
            return

        # Ensure that the task doesn't have a circular ref back to the AsyncLLM
        # object, or else it won't be garbage collected and cleaned up properly.
        engine_core = self.engine_core
        output_processor = self.output_processor
        log_stats = self.log_stats
        # We use a mutable list for logger_manager so that it can be updated
        # during elastic EP scaling (see scale_elastic_ep) without creating
        # a circular reference via self.
        self._logger_ref = [self.logger_manager]
        logger_ref = self._logger_ref
        renderer = self.renderer
        chunk_size = envs.VLLM_V1_OUTPUT_PROC_CHUNK_SIZE

        async def output_handler():
            try:
                while True:
                    # 1) Pull EngineCoreOutputs from the EngineCore.
                    outputs = await engine_core.get_output_async()
                    num_outputs = len(outputs.outputs)

                    iteration_stats = (
                        IterationStats() if (log_stats and num_outputs) else None
                    )

                    # Split outputs into chunks of at most
                    # VLLM_V1_OUTPUT_PROC_CHUNK_SIZE, so that we don't block the
                    # event loop for too long.
                    engine_core_outputs = outputs.outputs
                    for start in range(0, num_outputs, chunk_size):
                        end = start + chunk_size
                        outputs_slice = engine_core_outputs[start:end]
                        # 2) Process EngineCoreOutputs.
                        processed_outputs = output_processor.process_outputs(
                            outputs_slice, outputs.timestamp, iteration_stats
                        )
                        # NOTE: RequestOutputs are pushed to their queues.
                        assert not processed_outputs.request_outputs

                        # Allow other asyncio tasks to run between chunks
                        if end < num_outputs:
                            await asyncio.sleep(0)

                        # 3) Abort any reqs that finished due to stop strings.
                        if processed_outputs.reqs_to_abort:
                            await engine_core.abort_requests_async(
                                processed_outputs.reqs_to_abort
                            )

                    output_processor.update_scheduler_stats(outputs.scheduler_stats)

                    # 4) Logging.
                    # TODO(rob): make into a coroutine and launch it in
                    # background thread once Prometheus overhead is non-trivial.
                    if logger_ref[0]:
                        logger_ref[0].record(
                            engine_idx=outputs.engine_index,
                            scheduler_stats=outputs.scheduler_stats,
                            iteration_stats=iteration_stats,
                            mm_cache_stats=renderer.stat_mm_cache(),
                        )
            except Exception as e:
                logger.exception("AsyncLLM output_handler failed.")
                output_processor.propagate_error(e)

        self.output_handler = asyncio.create_task(output_handler())
```
**EN:** This method implements `_run_output_handler` within `AsyncLLM`. The docstring frames it as: Background loop: pulls from EngineCore and pushes to AsyncStreams. Key calls include `create_task`, `output_handler`, `len`, `range`, `update_scheduler_stats`, `exception`. It touches state such as `_logger_ref`, `output_handler`. The control flow contains 6 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_run_output_handler`，其作用域位于`AsyncLLM`。 关键调用包括 `create_task`, `output_handler`, `len`, `range`, `update_scheduler_stats`, `exception`。 它会读写 `_logger_ref`, `output_handler` 等状态。 控制流包含 6 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `InputStreamError`: central class or interface in this module. / `InputStreamError`：本模块中的核心类或接口。
- `AsyncLLM`: central class or interface in this module. / `AsyncLLM`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `asyncio`, `os`, `socket`, `time`, `warnings`, `collections`, `copy`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm`, `vllm.config`, `vllm.distributed.weight_transfer.base`, `vllm.engine.arg_utils`, `vllm.engine.protocol`, `vllm.entrypoints.serve.elastic_ep.middleware`, `vllm.inputs`, `vllm.logger`, `vllm.lora.request`, `vllm.multimodal`, `vllm.outputs`
