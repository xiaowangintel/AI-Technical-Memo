# engine.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/entrypoints/engine.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements engine logic for runtime entrypoints and service adapters. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 引擎 相关的逻辑，并服务于 运行时入口与服务适配层。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18: Provide supporting module logic / 提供辅助模块逻辑
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
"""
The entry point of inference server. (SRT = SGLang Runtime)

This file implements python APIs for the inference engine.
"""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 20-20: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 22-46: Register helpers or handlers / 注册辅助逻辑或处理器
```python
import asyncio
import atexit
import dataclasses
import logging
import multiprocessing as mp
import os
import random
import signal
import tempfile
import threading
import time
from typing import (
    Any,
    AsyncIterator,
    Callable,
    Dict,
    Iterator,
    List,
    Optional,
    Tuple,
    Union,
)

# Fix a bug of Python threading
setattr(threading, "_register_atexit", lambda *args, **kwargs: None)
```
**EN:** This block wires callbacks, registries, or handlers into the surrounding runtime.
**CN:** 该代码块把回调、注册表或处理器接入周边运行时流程。

### Lines 48-50: Import runtime dependencies / 导入运行时依赖
```python
import torch
import uvloop
import zmq
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 52-111: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.elastic_ep.expert_backup_manager import run_expert_backup_manager
from sglang.srt.entrypoints.engine_info_bootstrap_server import (
    EngineInfoBootstrapServer,
)
from sglang.srt.entrypoints.engine_score_mixin import EngineScoreMixin
from sglang.srt.entrypoints.EngineBase import EngineBase
from sglang.srt.managers.data_parallel_controller import (
    SCHEDULER_PIDS_ARG,
    run_data_parallel_controller_process,
)
from sglang.srt.managers.detokenizer_manager import run_detokenizer_process
from sglang.srt.managers.io_struct import (
    CloseSessionReqInput,
    DestroyWeightsUpdateGroupReqInput,
    EmbeddingReqInput,
    GenerateReqInput,
    GetWeightsByNameReqInput,
    InitWeightsUpdateGroupReqInput,
    LoadLoRAAdapterFromTensorsReqInput,
    LoadLoRAAdapterReqInput,
    MultimodalDataInputFormat,
    OpenSessionReqInput,
    ReleaseMemoryOccupationReqInput,
    ResumeMemoryOccupationReqInput,
    RpcReqInput,
    RpcReqOutput,
    UnloadLoRAAdapterReqInput,
    UpdateWeightFromDiskReqInput,
    UpdateWeightsFromDistributedReqInput,
    UpdateWeightsFromIPCReqInput,
    UpdateWeightsFromTensorReqInput,
)
from sglang.srt.managers.multi_tokenizer_mixin import (
    MultiTokenizerRouter,
    run_multi_detokenizer_router_process,
)
from sglang.srt.managers.scheduler import run_scheduler_process
from sglang.srt.managers.template_detection import resolve_auto_parsers
from sglang.srt.managers.template_manager import TemplateManager
from sglang.srt.managers.tokenizer_manager import TokenizerManager
from sglang.srt.observability.trace import process_tracing_init, trace_set_thread_info
from sglang.srt.plugins import load_plugins
from sglang.srt.server_args import PortArgs, ServerArgs
from sglang.srt.utils import (
    MultiprocessingSerializer,
    assert_pkg_version,
    configure_logger,
    get_bool_env_var,
    is_cuda,
    kill_process_tree,
    launch_dummy_health_check_server,
    maybe_reindex_device_id,
    numa_utils,
    set_prometheus_multiproc_dir,
    set_ulimit,
)
from sglang.srt.utils.network import get_zmq_socket, is_port_available
from sglang.srt.utils.torch_memory_saver_adapter import TorchMemorySaverAdapter
from sglang.srt.utils.watchdog import SubprocessWatchdog
from sglang.version import __version__
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 113-114: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)
asyncio.set_event_loop_policy(uvloop.EventLoopPolicy())
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 116-119: Provide supporting module logic / 提供辅助模块逻辑
```python
_is_cuda = is_cuda()


@dataclasses.dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 120-127: Define class SchedulerInitResult / 定义类 SchedulerInitResult
```python
class SchedulerInitResult:
    """Result from launching schedulers."""

    scheduler_infos: List[Dict[str, Any]]
    all_child_pids: List[int] = dataclasses.field(default_factory=list)
    wait_for_ready: Callable[[], None] = lambda: None
    wait_for_completion: Callable[[], None] = lambda: None
    engine_info_bootstrap_server: Optional[Any] = None
```
**EN:** This block declares the class `SchedulerInitResult`. It centers on Result from launching schedulers..
**CN:** 该代码块声明类 `SchedulerInitResult`。它负责承载与 引擎 相关的核心状态与行为。

### Lines 130-175: Implement init tokenizer manager / 实现init 分词器 管理器
```python
def init_tokenizer_manager(
    server_args: ServerArgs,
    port_args: PortArgs,
    TokenizerManagerClass: Optional[TokenizerManager] = None,
) -> Tuple[TokenizerManager, TemplateManager]:
    # Launch tokenizer process
    TokenizerManagerClass = TokenizerManagerClass or TokenizerManager
    tokenizer_manager = TokenizerManagerClass(server_args, port_args)

    # Initialize templates
    template_manager = TemplateManager()
    template_manager.initialize_templates(
        tokenizer_manager=tokenizer_manager,
        model_path=server_args.model_path,
        chat_template=server_args.chat_template,
        completion_template=server_args.completion_template,
    )

    # Resolve any remaining auto parsers using template manager's detection results
    for attr, suggested, label in (
        (
            "reasoning_parser",
            template_manager.suggested_reasoning_parser,
            "reasoning parser",
        ),
        (
            "tool_call_parser",
            template_manager.suggested_tool_call_parser,
            "tool-call parser",
        ),
    ):
        if getattr(server_args, attr) != "auto":
            continue
        if suggested is not None:
            setattr(server_args, attr, suggested)
            logger.info(
                f"Auto-detected --{attr.replace('_', '-')} as '{suggested}' from chat template"
            )
        else:
            logger.warning(
                f"--{attr.replace('_', '-')}=auto specified but could not detect "
                f"{label} from chat template. Disabling {label}."
            )
            setattr(server_args, attr, None)

    return tokenizer_manager, template_manager
```
**EN:** This block implements the function `init_tokenizer_manager(server_args, port_args, TokenizerManagerClass)`. It focuses on handling the engine responsibilities represented by `init_tokenizer_manager`, providing reusable behavior for the engine pipeline.
**CN:** 该代码块实现函数 `init_tokenizer_manager(server_args, port_args, TokenizerManagerClass)`。它围绕 `init_tokenizer_manager` 所承担的 引擎 相关职责展开，为对应处理链路提供可复用能力。

### Lines 178-197: Provide supporting module logic / 提供辅助模块逻辑
```python
class Engine(EngineScoreMixin, EngineBase):
    """
    The entry point to the inference engine.

    - The engine consists of three components:
        1. TokenizerManager: Tokenizes the requests and sends them to the scheduler.
        2. Scheduler (subprocess): Receives requests from the Tokenizer Manager, schedules batches, forwards them, and sends the output tokens to the Detokenizer Manager.
        3. DetokenizerManager (subprocess): Detokenizes the output tokens and sends the result back to the Tokenizer Manager.

    Note:
    1. The HTTP server, Engine, and TokenizerManager all run in the main process.
    2. Inter-process communication is done through IPC (each process uses a different port) via the ZMQ library.
    """

    # Some fields to allow people to override the server args
    # and launch processes for their private forks.
    server_args_class: ServerArgs = ServerArgs
    init_tokenizer_manager_func: Callable = staticmethod(init_tokenizer_manager)
    run_scheduler_process_func: Callable = staticmethod(run_scheduler_process)
    run_detokenizer_process_func: Callable = staticmethod(run_detokenizer_process)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 199-234: Initialize Engine / 初始化 Engine
```python
    def __init__(self, **kwargs):
        """
        The arguments of this function is the same as `sglang/srt/server_args.py::ServerArgs`.
        Please refer to `ServerArgs` for the documentation.
        """

        # Ensure plugins are loaded before ServerArgs construction,
        # so hooks on ServerArgs.__post_init__ fire correctly.
        load_plugins()

        # Parse server_args
        if "server_args" in kwargs:
            # Directly load server_args
            server_args = kwargs["server_args"]
        else:
            # Construct server_args from kwargs
            if "log_level" not in kwargs:
                # Do not print logs by default
                kwargs["log_level"] = "error"
            server_args = self.server_args_class(**kwargs)
        self.server_args = server_args
        logger.info(f"{server_args=}")

        # Pre-initialize tokenizer_manager so the atexit handler in
        # shutdown() won't hit AttributeError.
        self.tokenizer_manager = None

        # Shutdown the subprocesses automatically when the program exits
        atexit.register(self.shutdown)

        # Launch subprocesses
        (
            tokenizer_manager,
            template_manager,
            port_args,
            scheduler_init_result,
```
**EN:** This block implements the initializer `__init__(**kwargs)` for `Engine`. It prepares the object state and connects the instance to the surrounding engine workflow.
**CN:** 该代码块实现 `Engine` 的初始化方法 `__init__(**kwargs)`。它负责准备对象状态，并把实例接入 引擎 相关的运行流程。

### Lines 235-270: Initialize Engine (continued) / 初始化 Engine（续）
```python
            subprocess_watchdog,
        ) = self._launch_subprocesses(
            server_args=server_args,
            init_tokenizer_manager_func=self.init_tokenizer_manager_func,
            run_scheduler_process_func=self.run_scheduler_process_func,
            run_detokenizer_process_func=self.run_detokenizer_process_func,
        )
        self.tokenizer_manager = tokenizer_manager
        self.template_manager = template_manager
        self._scheduler_init_result = scheduler_init_result
        if tokenizer_manager is not None:
            tokenizer_manager._subprocess_watchdog = subprocess_watchdog
        self.port_args = port_args

        # Initialize ZMQ sockets
        context = zmq.Context(2)
        if self.server_args.node_rank == 0:
            self.send_to_rpc = get_zmq_socket(
                context, zmq.DEALER, self.port_args.rpc_ipc_name, True
            )
        else:
            self.send_to_rpc = None

        # Enable tracing
        if server_args.enable_trace:
            process_tracing_init(server_args.otlp_traces_endpoint, "sglang")
            thread_label = "Tokenizer"
            if server_args.disaggregation_mode == "prefill":
                thread_label = "Prefill Tokenizer"
            elif server_args.disaggregation_mode == "decode":
                thread_label = "Decode Tokenizer"
            trace_set_thread_info(thread_label)

        try:
            self.loop = asyncio.get_running_loop()
        except RuntimeError:
```
**EN:** This block implements the initializer `__init__(**kwargs)` for `Engine`. It prepares the object state and connects the instance to the surrounding engine workflow.
**CN:** 该代码块实现 `Engine` 的初始化方法 `__init__(**kwargs)`。它负责准备对象状态，并把实例接入 引擎 相关的运行流程。

### Lines 271-272: Initialize Engine (continued) / 初始化 Engine（续）
```python
            self.loop = asyncio.new_event_loop()
            asyncio.set_event_loop(self.loop)
```
**EN:** This block implements the initializer `__init__(**kwargs)` for `Engine`. It prepares the object state and connects the instance to the surrounding engine workflow.
**CN:** 该代码块实现 `Engine` 的初始化方法 `__init__(**kwargs)`。它负责准备对象状态，并把实例接入 引擎 相关的运行流程。

### Lines 274-276: Implement get all child pids / 实现get all child pids
```python
    def get_all_child_pids(self) -> List[int]:
        """Returns a list of all child process PIDs."""
        return self._scheduler_init_result.all_child_pids
```
**EN:** This block implements the method `get_all_child_pids()` on `Engine`. It focuses on Returns a list of all child process PIDs., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `get_all_child_pids()`。它围绕 `get_all_child_pids` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 278-307: Implement resolve routed dp rank / 实现resolve routed dp rank
```python
    def _resolve_routed_dp_rank(
        self,
        routed_dp_rank: Optional[int],
        data_parallel_rank: Optional[int],
    ) -> Optional[int]:
        if data_parallel_rank is not None:
            import warnings

            warnings.warn(
                "'data_parallel_rank' is deprecated, use 'routed_dp_rank' instead.",
                DeprecationWarning,
                stacklevel=3,
            )
            if routed_dp_rank is None:
                routed_dp_rank = data_parallel_rank

        if routed_dp_rank is not None:
            dp_size = self.server_args.dp_size
            if dp_size <= 1 and routed_dp_rank == 0:
                logger.warning(
                    f"routed_dp_rank={routed_dp_rank} is ignored because dp_size={dp_size}"
                )
                return None
            if routed_dp_rank < 0 or routed_dp_rank >= dp_size:
                raise ValueError(
                    f"routed_dp_rank={routed_dp_rank} out of range [0, {dp_size})"
                )

        logger.debug(f"routed_dp_rank: {routed_dp_rank}")
        return routed_dp_rank
```
**EN:** This block implements the method `_resolve_routed_dp_rank(routed_dp_rank, data_parallel_rank)` on `Engine`. It focuses on handling the engine responsibilities represented by `_resolve_routed_dp_rank`, so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `_resolve_routed_dp_rank(routed_dp_rank, data_parallel_rank)`。它围绕 `_resolve_routed_dp_rank` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 309-344: Implement generate / 实现generate
```python
    def generate(
        self,
        # The input prompt. It can be a single prompt or a batch of prompts.
        prompt: Optional[Union[List[str], str]] = None,
        sampling_params: Optional[Union[List[Dict], Dict]] = None,
        # The token ids for text; one can either specify text or input_ids.
        input_ids: Optional[Union[List[List[int]], List[int]]] = None,
        # The image input. It can be an image instance, file name, URL, or base64 encoded string.
        # Can be formatted as:
        # - Single image for a single request
        # - List of images (one per request in a batch)
        # - List of lists of images (multiple images per request)
        # - List of preprocessed outputs from a Huggingface processor, each as a dict containing `format`: 'processor_output' and other data
        # - List of precomputed image embeddings, each as a dict containing field `format`: 'precomputed_embedding' and `feature`: the precomputed embedding
        # See also python/sglang/srt/utils.py:load_image for more details.
        image_data: Optional[MultimodalDataInputFormat] = None,
        audio_data: Optional[MultimodalDataInputFormat] = None,
        video_data: Optional[MultimodalDataInputFormat] = None,
        return_logprob: Optional[Union[List[bool], bool]] = False,
        logprob_start_len: Optional[Union[List[int], int]] = None,
        top_logprobs_num: Optional[Union[List[int], int]] = None,
        token_ids_logprob: Optional[Union[List[List[int]], List[int]]] = None,
        lora_path: Optional[List[Optional[str]]] = None,
        custom_logit_processor: Optional[Union[List[str], str]] = None,
        return_hidden_states: bool = False,
        return_routed_experts: bool = False,
        routed_experts_start_len: int = 0,
        stream: bool = False,
        bootstrap_host: Optional[Union[List[str], str]] = None,
        bootstrap_port: Optional[Union[List[int], int]] = None,
        bootstrap_room: Optional[Union[List[int], int]] = None,
        routed_dp_rank: Optional[int] = None,
        disagg_prefill_dp_rank: Optional[int] = None,
        # Deprecated: use routed_dp_rank instead
        data_parallel_rank: Optional[int] = None,
        external_trace_header: Optional[Dict] = None,
```
**EN:** This block implements the method `generate(prompt, sampling_params, input_ids, image_data, audio_data, ...)` on `Engine`. It focuses on The arguments of this function is the same as `sglang/srt/managers/io_struct.py::GenerateReqInput`., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `generate(prompt, sampling_params, input_ids, image_data, audio_data, ...)`。它围绕 `generate` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 345-380: Continue generate / 继续说明generate
```python
        rid: Optional[Union[List[str], str]] = None,
        session_params: Optional[Dict] = None,
        priority: Optional[int] = None,
    ) -> Union[Dict, Iterator[Dict]]:
        """
        The arguments of this function is the same as `sglang/srt/managers/io_struct.py::GenerateReqInput`.
        Please refer to `GenerateReqInput` for the documentation.
        """
        routed_dp_rank = self._resolve_routed_dp_rank(
            routed_dp_rank, data_parallel_rank
        )

        obj = GenerateReqInput(
            text=prompt,
            input_ids=input_ids,
            sampling_params=sampling_params,
            image_data=image_data,
            audio_data=audio_data,
            video_data=video_data,
            return_logprob=return_logprob,
            logprob_start_len=logprob_start_len,
            top_logprobs_num=top_logprobs_num,
            token_ids_logprob=token_ids_logprob,
            lora_path=lora_path,
            custom_logit_processor=custom_logit_processor,
            return_hidden_states=return_hidden_states,
            return_routed_experts=return_routed_experts,
            routed_experts_start_len=routed_experts_start_len,
            stream=stream,
            bootstrap_host=bootstrap_host,
            bootstrap_port=bootstrap_port,
            bootstrap_room=bootstrap_room,
            routed_dp_rank=routed_dp_rank,
            disagg_prefill_dp_rank=disagg_prefill_dp_rank,
            external_trace_header=external_trace_header,
            rid=rid,
```
**EN:** This block implements the method `generate(prompt, sampling_params, input_ids, image_data, audio_data, ...)` on `Engine`. It focuses on The arguments of this function is the same as `sglang/srt/managers/io_struct.py::GenerateReqInput`., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `generate(prompt, sampling_params, input_ids, image_data, audio_data, ...)`。它围绕 `generate` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 381-399: Continue generate / 继续说明generate
```python
            session_params=session_params,
            priority=priority,
        )
        generator = self.tokenizer_manager.generate_request(obj, None)

        if stream:

            def generator_wrapper():
                while True:
                    try:
                        chunk = self.loop.run_until_complete(generator.__anext__())
                        yield chunk
                    except StopAsyncIteration:
                        break

            return generator_wrapper()
        else:
            ret = self.loop.run_until_complete(generator.__anext__())
            return ret
```
**EN:** This block implements the method `generate(prompt, sampling_params, input_ids, image_data, audio_data, ...)` on `Engine`. It focuses on The arguments of this function is the same as `sglang/srt/managers/io_struct.py::GenerateReqInput`., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `generate(prompt, sampling_params, input_ids, image_data, audio_data, ...)`。它围绕 `generate` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 401-436: Implement async async generate / 实现异步异步 generate
```python
    async def async_generate(
        self,
        # The input prompt. It can be a single prompt or a batch of prompts.
        prompt: Optional[Union[List[str], str]] = None,
        sampling_params: Optional[Union[List[Dict], Dict]] = None,
        # The token ids for text; one can either specify text or input_ids.
        input_ids: Optional[Union[List[List[int]], List[int]]] = None,
        # The image input. It can be an image instance, file name, URL, or base64 encoded string.
        # Can be formatted as:
        # - Single image for a single request
        # - List of images (one per request in a batch)
        # - List of lists of images (multiple images per request)
        # - List of preprocessed outputs from a Huggingface processor, each as a dict containing `format`: 'processor_output' and other data
        # - List of precomputed image embeddings, each as a dict containing field `format`: 'precomputed_embedding' and `feature`: the precomputed embedding
        # See also python/sglang/srt/utils.py:load_image for more details.
        image_data: Optional[MultimodalDataInputFormat] = None,
        audio_data: Optional[MultimodalDataInputFormat] = None,
        video_data: Optional[MultimodalDataInputFormat] = None,
        return_logprob: Optional[Union[List[bool], bool]] = False,
        logprob_start_len: Optional[Union[List[int], int]] = None,
        top_logprobs_num: Optional[Union[List[int], int]] = None,
        token_ids_logprob: Optional[Union[List[List[int]], List[int]]] = None,
        lora_path: Optional[List[Optional[str]]] = None,
        custom_logit_processor: Optional[Union[List[str], str]] = None,
        return_hidden_states: bool = False,
        return_routed_experts: bool = False,
        routed_experts_start_len: int = 0,
        stream: bool = False,
        bootstrap_host: Optional[Union[List[str], str]] = None,
        bootstrap_port: Optional[Union[List[int], int]] = None,
        bootstrap_room: Optional[Union[List[int], int]] = None,
        routed_dp_rank: Optional[int] = None,
        disagg_prefill_dp_rank: Optional[int] = None,
        # Deprecated: use routed_dp_rank instead
        data_parallel_rank: Optional[int] = None,
        external_trace_header: Optional[Dict] = None,
```
**EN:** This block implements the async method `async_generate(prompt, sampling_params, input_ids, image_data, audio_data, ...)` on `Engine`. It focuses on The arguments of this function is the same as `sglang/srt/managers/io_struct.py::GenerateReqInput`., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的异步方法 `async_generate(prompt, sampling_params, input_ids, image_data, audio_data, ...)`。它围绕 `async_generate` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 437-472: Continue async async generate / 继续说明异步异步 generate
```python
        rid: Optional[Union[List[str], str]] = None,
        session_params: Optional[Dict] = None,
        priority: Optional[int] = None,
    ) -> Union[Dict, AsyncIterator[Dict]]:
        """
        The arguments of this function is the same as `sglang/srt/managers/io_struct.py::GenerateReqInput`.
        Please refer to `GenerateReqInput` for the documentation.
        """
        routed_dp_rank = self._resolve_routed_dp_rank(
            routed_dp_rank, data_parallel_rank
        )

        obj = GenerateReqInput(
            text=prompt,
            input_ids=input_ids,
            sampling_params=sampling_params,
            image_data=image_data,
            audio_data=audio_data,
            video_data=video_data,
            return_logprob=return_logprob,
            logprob_start_len=logprob_start_len,
            top_logprobs_num=top_logprobs_num,
            token_ids_logprob=token_ids_logprob,
            lora_path=lora_path,
            return_hidden_states=return_hidden_states,
            return_routed_experts=return_routed_experts,
            routed_experts_start_len=routed_experts_start_len,
            stream=stream,
            custom_logit_processor=custom_logit_processor,
            bootstrap_host=bootstrap_host,
            bootstrap_port=bootstrap_port,
            bootstrap_room=bootstrap_room,
            routed_dp_rank=routed_dp_rank,
            disagg_prefill_dp_rank=disagg_prefill_dp_rank,
            external_trace_header=external_trace_header,
            rid=rid,
```
**EN:** This block implements the async method `async_generate(prompt, sampling_params, input_ids, image_data, audio_data, ...)` on `Engine`. It focuses on The arguments of this function is the same as `sglang/srt/managers/io_struct.py::GenerateReqInput`., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的异步方法 `async_generate(prompt, sampling_params, input_ids, image_data, audio_data, ...)`。它围绕 `async_generate` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 473-481: Continue async async generate / 继续说明异步异步 generate
```python
            session_params=session_params,
            priority=priority,
        )
        generator = self.tokenizer_manager.generate_request(obj, None)

        if stream is True:
            return generator
        else:
            return await generator.__anext__()
```
**EN:** This block implements the async method `async_generate(prompt, sampling_params, input_ids, image_data, audio_data, ...)` on `Engine`. It focuses on The arguments of this function is the same as `sglang/srt/managers/io_struct.py::GenerateReqInput`., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的异步方法 `async_generate(prompt, sampling_params, input_ids, image_data, audio_data, ...)`。它围绕 `async_generate` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 483-514: Implement encode / 实现encode
```python
    def encode(
        self,
        prompt: Union[str, List[str], List[Dict], List[List[Dict]]],
        image_data: Optional[MultimodalDataInputFormat] = None,
        audio_data: Optional[MultimodalDataInputFormat] = None,
        video_data: Optional[MultimodalDataInputFormat] = None,
        dimensions: Optional[int] = None,
        lora_path: Optional[Union[List[Optional[str]], Optional[str]]] = None,
        embed_override_token_id: Optional[int] = None,
        embed_overrides: Optional[List[List[torch.Tensor]]] = None,
        external_trace_header: Optional[Dict] = None,
        rid: Optional[Union[List[str], str]] = None,
    ) -> Dict:
        """
        The arguments of this function is the same as `sglang/srt/managers/io_struct.py::EmbeddingReqInput`.
        Please refer to `EmbeddingReqInput` for the documentation.
        """
        obj = EmbeddingReqInput(
            text=prompt,
            image_data=image_data,
            audio_data=audio_data,
            video_data=video_data,
            dimensions=dimensions,
            lora_path=lora_path,
            embed_override_token_id=embed_override_token_id,
            embed_overrides=embed_overrides,
            external_trace_header=external_trace_header,
            rid=rid,
        )
        generator = self.tokenizer_manager.generate_request(obj, None)
        ret = self.loop.run_until_complete(generator.__anext__())
        return ret
```
**EN:** This block implements the method `encode(prompt, image_data, audio_data, video_data, dimensions, ...)` on `Engine`. It focuses on The arguments of this function is the same as `sglang/srt/managers/io_struct.py::EmbeddingReqInput`., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `encode(prompt, image_data, audio_data, video_data, dimensions, ...)`。它围绕 `encode` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 516-548: Implement async async encode / 实现异步异步 encode
```python
    async def async_encode(
        self,
        prompt: Union[str, List[str], List[Dict], List[List[Dict]]],
        image_data: Optional[MultimodalDataInputFormat] = None,
        audio_data: Optional[MultimodalDataInputFormat] = None,
        video_data: Optional[MultimodalDataInputFormat] = None,
        dimensions: Optional[int] = None,
        lora_path: Optional[Union[List[Optional[str]], Optional[str]]] = None,
        embed_override_token_id: Optional[int] = None,
        embed_overrides: Optional[List[List[torch.Tensor]]] = None,
        external_trace_header: Optional[Dict] = None,
        rid: Optional[Union[List[str], str]] = None,
    ) -> Dict:
        """
        Asynchronous version of encode method.

        The arguments of this function is the same as `sglang/srt/managers/io_struct.py::EmbeddingReqInput`.
        Please refer to `EmbeddingReqInput` for the documentation.
        """
        obj = EmbeddingReqInput(
            text=prompt,
            image_data=image_data,
            audio_data=audio_data,
            video_data=video_data,
            dimensions=dimensions,
            lora_path=lora_path,
            embed_override_token_id=embed_override_token_id,
            embed_overrides=embed_overrides,
            external_trace_header=external_trace_header,
            rid=rid,
        )
        generator = self.tokenizer_manager.generate_request(obj, None)
        return await generator.__anext__()
```
**EN:** This block implements the async method `async_encode(prompt, image_data, audio_data, video_data, dimensions, ...)` on `Engine`. It focuses on Asynchronous version of encode method., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的异步方法 `async_encode(prompt, image_data, audio_data, video_data, dimensions, ...)`。它围绕 `async_encode` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 550-561: Implement rerank / 实现rerank
```python
    def rerank(
        self,
        prompt: Union[List[List[str]]],
    ) -> Dict:
        """
        The arguments of this function is the same as `sglang/srt/managers/io_struct.py::EmbeddingReqInput`.
        Please refer to `EmbeddingReqInput` for the documentation.
        """
        obj = EmbeddingReqInput(text=prompt, is_cross_encoder_request=True)
        generator = self.tokenizer_manager.generate_request(obj, None)
        ret = self.loop.run_until_complete(generator.__anext__())
        return ret
```
**EN:** This block implements the method `rerank(prompt)` on `Engine`. It focuses on The arguments of this function is the same as `sglang/srt/managers/io_struct.py::EmbeddingReqInput`., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `rerank(prompt)`。它围绕 `rerank` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 562-563: Provide supporting module logic / 提供辅助模块逻辑
```python

    @classmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 564-599: Implement launch scheduler processes / 实现launch 调度器 processes
```python
    def _launch_scheduler_processes(
        cls,
        server_args: ServerArgs,
        port_args: PortArgs,
        run_scheduler_process_func: Callable,
    ) -> Tuple[SchedulerInitResult, Optional[List]]:
        """Launch scheduler processes using multiprocessing.
        Override in subclasses for different backends (e.g. Ray).

        Returns:
            Tuple of (SchedulerInitResult, scheduler_procs).
            scheduler_procs is None for RayEngine (uses Ray actors instead).
        """
        scheduler_procs = []

        if server_args.dp_size == 1:
            # Launch tensor parallel scheduler processes
            memory_saver_adapter = TorchMemorySaverAdapter.create(
                enable=server_args.enable_memory_saver
            )
            scheduler_pipe_readers = []

            pp_rank_range, tp_rank_range, pp_size_per_node, tp_size_per_node = (
                _calculate_rank_ranges(
                    server_args.nnodes,
                    server_args.pp_size,
                    server_args.tp_size,
                    server_args.node_rank,
                )
            )

            for pp_rank in pp_rank_range:
                for tp_rank in tp_rank_range:
                    reader, writer = mp.Pipe(duplex=False)
                    gpu_id = (
                        server_args.base_gpu_id
```
**EN:** This block implements the method `_launch_scheduler_processes(server_args, port_args, run_scheduler_process_func)` on `Engine`. It focuses on Launch scheduler processes using multiprocessing., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `_launch_scheduler_processes(server_args, port_args, run_scheduler_process_func)`。它围绕 `_launch_scheduler_processes` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 600-635: Continue launch scheduler processes / 继续说明launch 调度器 processes
```python
                        + ((pp_rank % pp_size_per_node) * tp_size_per_node)
                        + (tp_rank % tp_size_per_node) * server_args.gpu_id_step
                    )
                    attn_cp_rank, moe_dp_rank, moe_ep_rank = _compute_parallelism_ranks(
                        server_args, tp_rank
                    )

                    with maybe_reindex_device_id(gpu_id) as gpu_id:
                        proc = mp.Process(
                            target=run_scheduler_process_func,
                            args=(
                                server_args,
                                port_args,
                                gpu_id,
                                tp_rank,
                                attn_cp_rank,
                                moe_dp_rank,
                                moe_ep_rank,
                                pp_rank,
                                None,
                                writer,
                            ),
                        )
                        with (
                            memory_saver_adapter.configure_subprocess(),
                            numa_utils.configure_subprocess(server_args, gpu_id),
                        ):
                            proc.start()

                    scheduler_procs.append(proc)
                    scheduler_pipe_readers.append(reader)
        else:
            # Launch the data parallel controller
            reader, writer = mp.Pipe(duplex=False)
            scheduler_pipe_readers = [reader]
            proc = mp.Process(
```
**EN:** This block implements the method `_launch_scheduler_processes(server_args, port_args, run_scheduler_process_func)` on `Engine`. It focuses on Launch scheduler processes using multiprocessing., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `_launch_scheduler_processes(server_args, port_args, run_scheduler_process_func)`。它围绕 `_launch_scheduler_processes` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 636-671: Continue launch scheduler processes / 继续说明launch 调度器 processes
```python
                target=run_data_parallel_controller_process,
                kwargs=dict(
                    server_args=server_args,
                    port_args=port_args,
                    pipe_writer=writer,
                    run_scheduler_process_func=run_scheduler_process_func,
                ),
            )
            proc.start()
            scheduler_procs.append(proc)

        all_child_pids = [proc.pid for proc in scheduler_procs]
        scheduler_infos = []

        def wait_for_ready():
            infos = _wait_for_scheduler_ready(scheduler_pipe_readers, scheduler_procs)
            scheduler_infos.extend(infos)
            # For dp_size > 1, collect child scheduler PIDs from the DP controller
            if server_args.dp_size > 1:
                for info in infos:
                    if SCHEDULER_PIDS_ARG in info:
                        all_child_pids.extend(info[SCHEDULER_PIDS_ARG])

        def wait_for_completion():
            for proc in scheduler_procs:
                proc.join()
                logger.error(
                    f"Scheduler or DataParallelController {proc.pid} "
                    f"terminated with {proc.exitcode}"
                )

        return (
            SchedulerInitResult(
                scheduler_infos=scheduler_infos,
                all_child_pids=all_child_pids,
                wait_for_ready=wait_for_ready,
```
**EN:** This block implements the method `_launch_scheduler_processes(server_args, port_args, run_scheduler_process_func)` on `Engine`. It focuses on Launch scheduler processes using multiprocessing., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `_launch_scheduler_processes(server_args, port_args, run_scheduler_process_func)`。它围绕 `_launch_scheduler_processes` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 672-675: Continue launch scheduler processes / 继续说明launch 调度器 processes
```python
                wait_for_completion=wait_for_completion,
            ),
            scheduler_procs,
        )
```
**EN:** This block implements the method `_launch_scheduler_processes(server_args, port_args, run_scheduler_process_func)` on `Engine`. It focuses on Launch scheduler processes using multiprocessing., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `_launch_scheduler_processes(server_args, port_args, run_scheduler_process_func)`。它围绕 `_launch_scheduler_processes` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 676-677: Provide supporting module logic / 提供辅助模块逻辑
```python

    @classmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 678-713: Implement launch detokenizer subprocesses / 实现launch 反分词器 subprocesses
```python
    def _launch_detokenizer_subprocesses(
        cls,
        server_args: ServerArgs,
        port_args: PortArgs,
        run_detokenizer_process_func: Callable,
    ) -> Tuple[List[mp.Process], List[str]]:
        """Launch detokenizer worker(s).

        - When ``detokenizer_worker_num == 1``: a single detokenizer process listens on
          ``port_args.detokenizer_ipc_name`` (the original behavior).
        - When ``detokenizer_worker_num > 1``: each detokenizer worker gets its own
          private IPC socket, and a ``MultiDetokenizerRouter`` process owns the
          original ``port_args.detokenizer_ipc_name`` and fans out to them.

        Returns (processes, names) for SubprocessWatchdog.
        """
        processes: List[mp.Process] = []
        names: List[str] = []

        if server_args.detokenizer_worker_num <= 1:
            proc = mp.Process(
                target=run_detokenizer_process_func,
                args=(server_args, port_args),
            )
            proc.start()
            processes.append(proc)
            names.append("detokenizer")
            return processes, names

        router_ipc_name = port_args.detokenizer_ipc_name
        worker_ipc_names: List[str] = []
        try:
            for i in range(server_args.detokenizer_worker_num):
                worker_ipc = f"ipc://{tempfile.NamedTemporaryFile(delete=False).name}"
                port_args.detokenizer_ipc_name = worker_ipc
                proc = mp.Process(
```
**EN:** This block implements the method `_launch_detokenizer_subprocesses(server_args, port_args, run_detokenizer_process_func)` on `Engine`. It focuses on Launch detokenizer worker(s)., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `_launch_detokenizer_subprocesses(server_args, port_args, run_detokenizer_process_func)`。它围绕 `_launch_detokenizer_subprocesses` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 714-732: Continue launch detokenizer subprocesses / 继续说明launch 反分词器 subprocesses
```python
                    target=run_detokenizer_process_func,
                    args=(server_args, port_args),
                )
                proc.start()
                processes.append(proc)
                names.append(f"detokenizer_{i}")
                worker_ipc_names.append(worker_ipc)
        finally:
            port_args.detokenizer_ipc_name = router_ipc_name

        router_proc = mp.Process(
            target=run_multi_detokenizer_router_process,
            args=(worker_ipc_names, server_args, port_args),
        )
        router_proc.start()
        processes.append(router_proc)
        names.append("detokenizer_router")

        return processes, names
```
**EN:** This block implements the method `_launch_detokenizer_subprocesses(server_args, port_args, run_detokenizer_process_func)` on `Engine`. It focuses on Launch detokenizer worker(s)., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `_launch_detokenizer_subprocesses(server_args, port_args, run_detokenizer_process_func)`。它围绕 `_launch_detokenizer_subprocesses` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 733-734: Provide supporting module logic / 提供辅助模块逻辑
```python

    @classmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 735-770: Implement launch subprocesses / 实现launch subprocesses
```python
    def _launch_subprocesses(
        cls,
        server_args: ServerArgs,
        init_tokenizer_manager_func: Callable,
        run_scheduler_process_func: Callable,
        run_detokenizer_process_func: Callable,
        port_args: Optional[PortArgs] = None,
    ) -> Tuple[
        TokenizerManager,
        TemplateManager,
        PortArgs,
        SchedulerInitResult,
        Optional[SubprocessWatchdog],
    ]:
        """Launch the TokenizerManager in the main process, the Scheduler in a subprocess, and the DetokenizerManager in another subprocess.

        Returns:
            Tuple of (tokenizer_manager, template_manager, port_args, scheduler_init_result, subprocess_watchdog).
        """
        # Configure global environment
        configure_logger(server_args)
        _set_envs_and_config(server_args)

        # Defensive: ensure plugins loaded (may already be loaded by
        # Engine.__init__ or CLI entry).
        load_plugins()

        server_args.check_server_args()
        _set_gc(server_args)

        # Allocate ports for inter-process communications
        if port_args is None:
            port_args = PortArgs.init_new(server_args)
        logger.info(f"{server_args=}")

        # Start the engine info bootstrap server if per-rank info is needed.
```
**EN:** This block implements the method `_launch_subprocesses(server_args, init_tokenizer_manager_func, run_scheduler_process_func, run_detokenizer_process_func, port_args)` on `Engine`. It focuses on Launch the TokenizerManager in the main process, the Scheduler in a subprocess, and the DetokenizerManager in another subprocess., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `_launch_subprocesses(server_args, init_tokenizer_manager_func, run_scheduler_process_func, run_detokenizer_process_func, port_args)`。它围绕 `_launch_subprocesses` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 771-806: Continue launch subprocesses / 继续说明launch subprocesses
```python
        engine_info_bootstrap_server = None
        if (
            server_args.remote_instance_weight_loader_start_seed_via_transfer_engine
            and server_args.node_rank == 0
        ):
            bootstrap_port = server_args.engine_info_bootstrap_port
            if not is_port_available(bootstrap_port):
                raise RuntimeError(
                    f"engine_info_bootstrap_port {bootstrap_port} is already in use. "
                    f"When running multiple instances on the same node, each instance must use a "
                    f"different --engine-info-bootstrap-port."
                )
            engine_info_bootstrap_server = EngineInfoBootstrapServer(
                host=server_args.host, port=bootstrap_port
            )

        if (
            server_args.reasoning_parser == "auto"
            or server_args.tool_call_parser == "auto"
        ):
            resolve_auto_parsers(server_args)

        # Launch scheduler processes
        scheduler_init_result, scheduler_procs = cls._launch_scheduler_processes(
            server_args, port_args, run_scheduler_process_func
        )
        scheduler_init_result.engine_info_bootstrap_server = (
            engine_info_bootstrap_server
        )

        if (
            server_args.enable_elastic_expert_backup
            and server_args.elastic_ep_backend is not None
        ):
            run_expert_backup_manager(server_args, port_args)
```
**EN:** This block implements the method `_launch_subprocesses(server_args, init_tokenizer_manager_func, run_scheduler_process_func, run_detokenizer_process_func, port_args)` on `Engine`. It focuses on Launch the TokenizerManager in the main process, the Scheduler in a subprocess, and the DetokenizerManager in another subprocess., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `_launch_subprocesses(server_args, init_tokenizer_manager_func, run_scheduler_process_func, run_detokenizer_process_func, port_args)`。它围绕 `_launch_subprocesses` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 807-842: Continue launch subprocesses / 继续说明launch subprocesses
```python
        if server_args.node_rank >= 1:
            # In multi-node cases, non-zero rank nodes do not need to run tokenizer or detokenizer,
            # so they can just wait here.
            scheduler_init_result.wait_for_ready()

            if os.getenv("SGLANG_BLOCK_NONZERO_RANK_CHILDREN") == "0":
                # When using `Engine` as a Python API, we don't want to block here.
                return (
                    None,
                    None,
                    port_args,
                    scheduler_init_result,
                    None,
                )

            launch_dummy_health_check_server(
                server_args.host, server_args.port, server_args.enable_metrics
            )

            scheduler_init_result.wait_for_completion()
            return (
                None,
                None,
                port_args,
                scheduler_init_result,
                None,
            )

        # Launch detokenizer process(es) — optionally fronted by a router when
        # detokenizer_worker_num > 1.
        detoken_procs, detoken_names = cls._launch_detokenizer_subprocesses(
            server_args=server_args,
            port_args=port_args,
            run_detokenizer_process_func=run_detokenizer_process_func,
        )
        for p in detoken_procs:
```
**EN:** This block implements the method `_launch_subprocesses(server_args, init_tokenizer_manager_func, run_scheduler_process_func, run_detokenizer_process_func, port_args)` on `Engine`. It focuses on Launch the TokenizerManager in the main process, the Scheduler in a subprocess, and the DetokenizerManager in another subprocess., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `_launch_subprocesses(server_args, init_tokenizer_manager_func, run_scheduler_process_func, run_detokenizer_process_func, port_args)`。它围绕 `_launch_subprocesses` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 843-878: Continue launch subprocesses / 继续说明launch subprocesses
```python
            scheduler_init_result.all_child_pids.append(p.pid)

        # Init tokenizer manager first, as the bootstrap server is initialized here
        if server_args.tokenizer_worker_num == 1:
            tokenizer_manager, template_manager = init_tokenizer_manager_func(
                server_args, port_args
            )
        else:
            # Launch multi-tokenizer router
            tokenizer_manager = MultiTokenizerRouter(server_args, port_args)
            template_manager = None

        # Wait for the model to finish loading
        scheduler_init_result.wait_for_ready()

        # Get back some info from scheduler to tokenizer_manager
        tokenizer_manager.max_req_input_len = scheduler_init_result.scheduler_infos[0][
            "max_req_input_len"
        ]

        # Set up subprocess liveness watchdog to detect crashes
        # Note: RayEngine returns scheduler_procs=None as it uses Ray actors instead of mp.Process
        processes = list(scheduler_procs or [])
        names = [f"scheduler_{i}" for i in range(len(processes))]
        processes.extend(detoken_procs)
        names.extend(detoken_names)
        subprocess_watchdog = SubprocessWatchdog(
            processes=processes, process_names=names
        )
        subprocess_watchdog.start()

        return (
            tokenizer_manager,
            template_manager,
            port_args,
            scheduler_init_result,
```
**EN:** This block implements the method `_launch_subprocesses(server_args, init_tokenizer_manager_func, run_scheduler_process_func, run_detokenizer_process_func, port_args)` on `Engine`. It focuses on Launch the TokenizerManager in the main process, the Scheduler in a subprocess, and the DetokenizerManager in another subprocess., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `_launch_subprocesses(server_args, init_tokenizer_manager_func, run_scheduler_process_func, run_detokenizer_process_func, port_args)`。它围绕 `_launch_subprocesses` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 879-880: Continue launch subprocesses / 继续说明launch subprocesses
```python
            subprocess_watchdog,
        )
```
**EN:** This block implements the method `_launch_subprocesses(server_args, init_tokenizer_manager_func, run_scheduler_process_func, run_detokenizer_process_func, port_args)` on `Engine`. It focuses on Launch the TokenizerManager in the main process, the Scheduler in a subprocess, and the DetokenizerManager in another subprocess., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `_launch_subprocesses(server_args, init_tokenizer_manager_func, run_scheduler_process_func, run_detokenizer_process_func, port_args)`。它围绕 `_launch_subprocesses` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 882-891: Implement shutdown / 实现shutdown
```python
    def shutdown(self):
        """Shutdown the engine; block until the scheduler subprocess releases
        its GPU context so the caller can immediately reallocate on the same
        device."""
        if (
            self.tokenizer_manager is not None
            and self.tokenizer_manager._subprocess_watchdog is not None
        ):
            self.tokenizer_manager._subprocess_watchdog.stop()
        kill_process_tree(os.getpid(), include_parent=False, wait_timeout=60)
```
**EN:** This block implements the method `shutdown()` on `Engine`. It focuses on Shutdown the engine; block until the scheduler subprocess releases its GPU context so the caller can immediately reallocate on the same device., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `shutdown()`。它围绕 `shutdown` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 893-894: Implement enter / 实现enter
```python
    def __enter__(self):
        return self
```
**EN:** This block implements the method `__enter__()` on `Engine`. It focuses on handling the engine responsibilities represented by `__enter__`, so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `__enter__()`。它围绕 `__enter__` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 896-898: Implement exit / 实现exit
```python
    def __exit__(self, exc_type, exc_value, traceback):
        self.shutdown()
        return False
```
**EN:** This block implements the method `__exit__(exc_type, exc_value, traceback)` on `Engine`. It focuses on handling the engine responsibilities represented by `__exit__`, so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `__exit__(exc_type, exc_value, traceback)`。它围绕 `__exit__` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 900-901: Implement flush cache / 实现flush 缓存
```python
    def flush_cache(self):
        return self.loop.run_until_complete(self.tokenizer_manager.flush_cache())
```
**EN:** This block implements the method `flush_cache()` on `Engine`. It focuses on handling the engine responsibilities represented by `flush_cache`, so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `flush_cache()`。它围绕 `flush_cache` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 903-931: Implement open session / 实现open 会话
```python
    def open_session(
        self,
        capacity_of_str_len: int,
        session_id: Optional[str] = None,
        streaming: bool = False,
        timeout: Optional[float] = None,
    ) -> str:
        """Open a session for multi-turn conversation with shared context.

        Args:
            capacity_of_str_len: Maximum string length capacity for the session.
            session_id: Optional session ID. If not provided, a UUID will be generated.
            streaming: Use low-overhead path for realtime streaming (append-only mode).
            timeout: If set, the session is automatically closed after being inactive
                for this many seconds. Inactivity is measured from session open or the
                most recent request submission.

        Returns:
            The session ID (either the provided one or a newly generated UUID).
        """
        obj = OpenSessionReqInput(
            capacity_of_str_len=capacity_of_str_len,
            session_id=session_id,
            streaming=streaming,
            timeout=timeout,
        )
        return self.loop.run_until_complete(
            self.tokenizer_manager.open_session(obj, None)
        )
```
**EN:** This block implements the method `open_session(capacity_of_str_len, session_id, streaming, timeout)` on `Engine`. It focuses on Open a session for multi-turn conversation with shared context., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `open_session(capacity_of_str_len, session_id, streaming, timeout)`。它围绕 `open_session` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 933-940: Implement close session / 实现close 会话
```python
    def close_session(self, session_id: str) -> None:
        """Close a session and release its resources.

        Args:
            session_id: The session ID to close.
        """
        obj = CloseSessionReqInput(session_id=session_id)
        self.loop.run_until_complete(self.tokenizer_manager.close_session(obj, None))
```
**EN:** This block implements the method `close_session(session_id)` on `Engine`. It focuses on Close a session and release its resources., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `close_session(session_id)`。它围绕 `close_session` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 942-943: Implement start profile / 实现start 剖析
```python
    def start_profile(self, **kwargs):
        self.loop.run_until_complete(self.tokenizer_manager.start_profile(**kwargs))
```
**EN:** This block implements the method `start_profile(**kwargs)` on `Engine`. It focuses on handling the engine responsibilities represented by `start_profile`, so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `start_profile(**kwargs)`。它围绕 `start_profile` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 945-946: Implement stop profile / 实现stop 剖析
```python
    def stop_profile(self):
        self.loop.run_until_complete(self.tokenizer_manager.stop_profile())
```
**EN:** This block implements the method `stop_profile()` on `Engine`. It focuses on handling the engine responsibilities represented by `stop_profile`, so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `stop_profile()`。它围绕 `stop_profile` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 948-951: Implement start expert distribution record / 实现start expert distribution record
```python
    def start_expert_distribution_record(self):
        self.loop.run_until_complete(
            self.tokenizer_manager.start_expert_distribution_record()
        )
```
**EN:** This block implements the method `start_expert_distribution_record()` on `Engine`. It focuses on handling the engine responsibilities represented by `start_expert_distribution_record`, so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `start_expert_distribution_record()`。它围绕 `start_expert_distribution_record` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 953-956: Implement stop expert distribution record / 实现stop expert distribution record
```python
    def stop_expert_distribution_record(self):
        self.loop.run_until_complete(
            self.tokenizer_manager.stop_expert_distribution_record()
        )
```
**EN:** This block implements the method `stop_expert_distribution_record()` on `Engine`. It focuses on handling the engine responsibilities represented by `stop_expert_distribution_record`, so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `stop_expert_distribution_record()`。它围绕 `stop_expert_distribution_record` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 958-961: Implement dump expert distribution record / 实现dump expert distribution record
```python
    def dump_expert_distribution_record(self):
        self.loop.run_until_complete(
            self.tokenizer_manager.dump_expert_distribution_record()
        )
```
**EN:** This block implements the method `dump_expert_distribution_record()` on `Engine`. It focuses on handling the engine responsibilities represented by `dump_expert_distribution_record`, so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `dump_expert_distribution_record()`。它围绕 `dump_expert_distribution_record` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 963-972: Implement get server info / 实现get 服务器 info
```python
    def get_server_info(self):
        internal_states = self.loop.run_until_complete(
            self.tokenizer_manager.get_internal_state()
        )
        return {
            **dataclasses.asdict(self.tokenizer_manager.server_args),
            **self._scheduler_init_result.scheduler_infos[0],
            "internal_states": internal_states,
            "version": __version__,
        }
```
**EN:** This block implements the method `get_server_info()` on `Engine`. It focuses on handling the engine responsibilities represented by `get_server_info`, so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `get_server_info()`。它围绕 `get_server_info` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 974-994: Implement init weights update group / 实现init weights update group
```python
    def init_weights_update_group(
        self,
        master_address: str,
        master_port: int,
        rank_offset: int,
        world_size: int,
        group_name: str,
        backend: str = "nccl",
    ):
        """Initialize parameter update group."""
        obj = InitWeightsUpdateGroupReqInput(
            master_address=master_address,
            master_port=master_port,
            rank_offset=rank_offset,
            world_size=world_size,
            group_name=group_name,
            backend=backend,
        )
        return self.loop.run_until_complete(
            self.tokenizer_manager.init_weights_update_group(obj, None)
        )
```
**EN:** This block implements the method `init_weights_update_group(master_address, master_port, rank_offset, world_size, group_name, ...)` on `Engine`. It focuses on Initialize parameter update group., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `init_weights_update_group(master_address, master_port, rank_offset, world_size, group_name, ...)`。它围绕 `init_weights_update_group` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 996-1006: Implement destroy weights update group / 实现destroy weights update group
```python
    def destroy_weights_update_group(
        self,
        group_name: str,
    ):
        """Destroy parameter update group."""
        obj = DestroyWeightsUpdateGroupReqInput(
            group_name=group_name,
        )
        return self.loop.run_until_complete(
            self.tokenizer_manager.destroy_weights_update_group(obj, None)
        )
```
**EN:** This block implements the method `destroy_weights_update_group(group_name)` on `Engine`. It focuses on Destroy parameter update group., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `destroy_weights_update_group(group_name)`。它围绕 `destroy_weights_update_group` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 1008-1028: Implement update weights from distributed / 实现update weights from distributed
```python
    def update_weights_from_distributed(
        self,
        names: list[str],
        dtypes: list[str],
        shapes: list[list[int]],
        group_name: str = "weight_update_group",
        flush_cache: bool = True,
        load_format: Optional[str] = None,
    ):
        """Update weights from distributed source."""
        obj = UpdateWeightsFromDistributedReqInput(
            names=names,
            dtypes=dtypes,
            shapes=shapes,
            group_name=group_name,
            flush_cache=flush_cache,
            load_format=load_format,
        )
        return self.loop.run_until_complete(
            self.tokenizer_manager.update_weights_from_distributed(obj, None)
        )
```
**EN:** This block implements the method `update_weights_from_distributed(names, dtypes, shapes, group_name, flush_cache, ...)` on `Engine`. It focuses on Update weights from distributed source., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `update_weights_from_distributed(names, dtypes, shapes, group_name, flush_cache, ...)`。它围绕 `update_weights_from_distributed` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 1030-1052: Implement update weights from tensor / 实现update weights from 张量
```python
    def update_weights_from_tensor(
        self,
        named_tensors: List[Tuple[str, torch.Tensor]],
        load_format: Optional[str] = None,
        flush_cache: bool = True,
    ):
        """Update weights from distributed source. If there are going to be more updates, set `flush_cache` to be false
        to avoid duplicated cache cleaning operation."""
        if load_format == "flattened_bucket":
            serialized_named_tensors = named_tensors
        else:
            serialized_named_tensors = [
                MultiprocessingSerializer.serialize(named_tensors)
                for _ in range(self.server_args.tp_size)
            ]
        obj = UpdateWeightsFromTensorReqInput(
            serialized_named_tensors=serialized_named_tensors,
            load_format=load_format,
            flush_cache=flush_cache,
        )
        return self.loop.run_until_complete(
            self.tokenizer_manager.update_weights_from_tensor(obj, None)
        )
```
**EN:** This block implements the method `update_weights_from_tensor(named_tensors, load_format, flush_cache)` on `Engine`. It focuses on Update weights from distributed source., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `update_weights_from_tensor(named_tensors, load_format, flush_cache)`。它围绕 `update_weights_from_tensor` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 1054-1072: Implement update weights from disk / 实现update weights from disk
```python
    def update_weights_from_disk(
        self,
        model_path: str,
        load_format: Optional[str] = None,
    ):
        """Update the weights from disk inplace without re-launching the engine.

        This method allows updating the model weights from disk without restarting
        the engine. It can be used to load a different model or update weights with
        new training.
        """
        obj = UpdateWeightFromDiskReqInput(
            model_path=model_path,
            load_format=load_format,
        )

        return self.loop.run_until_complete(
            self.tokenizer_manager.update_weights_from_disk(obj, None)
        )
```
**EN:** This block implements the method `update_weights_from_disk(model_path, load_format)` on `Engine`. It focuses on Update the weights from disk inplace without re-launching the engine., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `update_weights_from_disk(model_path, load_format)`。它围绕 `update_weights_from_disk` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 1074-1086: Implement update weights from ipc / 实现update weights from ipc
```python
    def update_weights_from_ipc(
        self,
        zmq_handles: Dict[str, str],
        flush_cache: bool = True,
    ):
        """Update weights from IPC for checkpoint-engine integration."""
        obj = UpdateWeightsFromIPCReqInput(
            zmq_handles=zmq_handles,
            flush_cache=flush_cache,
        )
        return self.loop.run_until_complete(
            self.tokenizer_manager.update_weights_from_ipc(obj, None)
        )
```
**EN:** This block implements the method `update_weights_from_ipc(zmq_handles, flush_cache)` on `Engine`. It focuses on Update weights from IPC for checkpoint-engine integration., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `update_weights_from_ipc(zmq_handles, flush_cache)`。它围绕 `update_weights_from_ipc` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 1088-1093: Implement get weights by name / 实现get weights by name
```python
    def get_weights_by_name(self, name: str, truncate_size: int = 100):
        """Get weights by parameter name."""
        obj = GetWeightsByNameReqInput(name=name, truncate_size=truncate_size)
        return self.loop.run_until_complete(
            self.tokenizer_manager.get_weights_by_name(obj, None)
        )
```
**EN:** This block implements the method `get_weights_by_name(name, truncate_size)` on `Engine`. It focuses on Get weights by parameter name., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `get_weights_by_name(name, truncate_size)`。它围绕 `get_weights_by_name` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 1095-1116: Implement load lora adapter from tensors / 实现load lora adapter from tensors
```python
    def load_lora_adapter_from_tensors(
        self,
        lora_name: str,
        tensors,
        config_dict: Dict,
        load_format: Optional[str] = None,
    ):
        if load_format == "flattened_bucket":
            serialized_tensors = tensors
        else:
            serialized_tensors = MultiprocessingSerializer.serialize(
                tensors, output_str=True
            )
        lora_req = LoadLoRAAdapterFromTensorsReqInput(
            lora_name=lora_name,
            config_dict=config_dict,
            serialized_tensors=serialized_tensors,
            load_format=load_format,
        )
        return self.loop.run_until_complete(
            self.tokenizer_manager.load_lora_adapter_from_tensors(lora_req, None)
        )
```
**EN:** This block implements the method `load_lora_adapter_from_tensors(lora_name, tensors, config_dict, load_format)` on `Engine`. It focuses on handling the engine responsibilities represented by `load_lora_adapter_from_tensors`, so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `load_lora_adapter_from_tensors(lora_name, tensors, config_dict, load_format)`。它围绕 `load_lora_adapter_from_tensors` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 1118-1129: Implement load lora adapter / 实现load lora adapter
```python
    def load_lora_adapter(self, lora_name: str, lora_path: str, pinned: bool = False):
        """Load a new LoRA adapter without re-launching the engine."""

        obj = LoadLoRAAdapterReqInput(
            lora_name=lora_name,
            lora_path=lora_path,
            pinned=pinned,
        )

        return self.loop.run_until_complete(
            self.tokenizer_manager.load_lora_adapter(obj, None)
        )
```
**EN:** This block implements the method `load_lora_adapter(lora_name, lora_path, pinned)` on `Engine`. It focuses on Load a new LoRA adapter without re-launching the engine., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `load_lora_adapter(lora_name, lora_path, pinned)`。它围绕 `load_lora_adapter` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 1131-1138: Implement unload lora adapter / 实现unload lora adapter
```python
    def unload_lora_adapter(self, lora_name: str):
        """Unload a LoRA adapter without re-launching the engine."""

        obj = UnloadLoRAAdapterReqInput(lora_name=lora_name)

        return self.loop.run_until_complete(
            self.tokenizer_manager.unload_lora_adapter(obj, None)
        )
```
**EN:** This block implements the method `unload_lora_adapter(lora_name)` on `Engine`. It focuses on Unload a LoRA adapter without re-launching the engine., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `unload_lora_adapter(lora_name)`。它围绕 `unload_lora_adapter` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 1140-1155: Implement async async load lora adapter / 实现异步异步 load lora adapter
```python
    async def async_load_lora_adapter(
        self, lora_name: str, lora_path: str, pinned: bool = False
    ):
        """
        Asynchronous version of load_lora_adapter.

        See load_lora_adapter() for detailed documentation.
        """

        obj = LoadLoRAAdapterReqInput(
            lora_name=lora_name,
            lora_path=lora_path,
            pinned=pinned,
        )

        return await self.tokenizer_manager.load_lora_adapter(obj, None)
```
**EN:** This block implements the async method `async_load_lora_adapter(lora_name, lora_path, pinned)` on `Engine`. It focuses on Asynchronous version of load_lora_adapter., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的异步方法 `async_load_lora_adapter(lora_name, lora_path, pinned)`。它围绕 `async_load_lora_adapter` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 1157-1166: Implement async async unload lora adapter / 实现异步异步 unload lora adapter
```python
    async def async_unload_lora_adapter(self, lora_name: str):
        """
        Asynchronous version of unload_lora_adapter.

        See unload_lora_adapter() for detailed documentation.
        """

        obj = UnloadLoRAAdapterReqInput(lora_name=lora_name)

        return await self.tokenizer_manager.unload_lora_adapter(obj, None)
```
**EN:** This block implements the async method `async_unload_lora_adapter(lora_name)` on `Engine`. It focuses on Asynchronous version of unload_lora_adapter., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的异步方法 `async_unload_lora_adapter(lora_name)`。它围绕 `async_unload_lora_adapter` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 1168-1172: Implement release memory occupation / 实现release memory occupation
```python
    def release_memory_occupation(self, tags: Optional[List[str]] = None):
        obj = ReleaseMemoryOccupationReqInput(tags=tags)
        return self.loop.run_until_complete(
            self.tokenizer_manager.release_memory_occupation(obj, None)
        )
```
**EN:** This block implements the method `release_memory_occupation(tags)` on `Engine`. It focuses on handling the engine responsibilities represented by `release_memory_occupation`, so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `release_memory_occupation(tags)`。它围绕 `release_memory_occupation` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 1174-1178: Implement resume memory occupation / 实现resume memory occupation
```python
    def resume_memory_occupation(self, tags: Optional[List[str]] = None):
        obj = ResumeMemoryOccupationReqInput(tags=tags)
        return self.loop.run_until_complete(
            self.tokenizer_manager.resume_memory_occupation(obj, None)
        )
```
**EN:** This block implements the method `resume_memory_occupation(tags)` on `Engine`. It focuses on handling the engine responsibilities represented by `resume_memory_occupation`, so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `resume_memory_occupation(tags)`。它围绕 `resume_memory_occupation` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 1180-1193: Implement freeze gc / 实现freeze gc
```python
    def freeze_gc(self):
        """
        To maintain a high performance server with low latency, we want to reduce the
        stalls caused by the garbage collector scanning through a large number of objects.

        It is usually helpful to start the server and warm it up with real requests to
        initialize many of the long-lived objects that do not need to be garbage collected.

        After sufficient warmup, we can call this function to freeze the garbage collector
        so that all objects created before this point are considered out of scope for garbage
        collection.
        """

        self.loop.run_until_complete(self.tokenizer_manager.freeze_gc())
```
**EN:** This block implements the method `freeze_gc()` on `Engine`. It focuses on To maintain a high performance server with low latency, we want to reduce the stalls caused by the garbage collector scanning through a large number of objects., so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `freeze_gc()`。它围绕 `freeze_gc` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 1195-1197: Provide supporting module logic / 提供辅助模块逻辑
```python
    """
    Execute an RPC call on all scheduler processes.
    """
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1199-1204: Implement collective rpc / 实现collective rpc
```python
    def collective_rpc(self, method: str, **kwargs):
        obj = RpcReqInput(method=method, parameters=kwargs)
        self.send_to_rpc.send_pyobj(obj)
        recv_req = self.send_to_rpc.recv_pyobj(zmq.BLOCKY)
        assert isinstance(recv_req, RpcReqOutput)
        assert recv_req.success, recv_req.message
```
**EN:** This block implements the method `collective_rpc(method, **kwargs)` on `Engine`. It focuses on handling the engine responsibilities represented by `collective_rpc`, so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `collective_rpc(method, **kwargs)`。它围绕 `collective_rpc` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 1206-1207: Implement save remote model / 实现save remote 模型
```python
    def save_remote_model(self, **kwargs):
        self.collective_rpc("save_remote_model", **kwargs)
```
**EN:** This block implements the method `save_remote_model(**kwargs)` on `Engine`. It focuses on handling the engine responsibilities represented by `save_remote_model`, so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `save_remote_model(**kwargs)`。它围绕 `save_remote_model` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 1209-1210: Implement save sharded model / 实现save sharded 模型
```python
    def save_sharded_model(self, **kwargs):
        self.collective_rpc("save_sharded_model", **kwargs)
```
**EN:** This block implements the method `save_sharded_model(**kwargs)` on `Engine`. It focuses on handling the engine responsibilities represented by `save_sharded_model`, so the class can advance the engine workflow in a self-contained way.
**CN:** 该代码块实现 `Engine` 上的方法 `save_sharded_model(**kwargs)`。它围绕 `save_sharded_model` 所承担的 引擎 相关职责展开，使该类能够独立推进相应流程。

### Lines 1211-1214: Import runtime dependencies / 导入运行时依赖
```python

    # score() and async_score() are provided by EngineScoreMixin
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 1215-1250: Implement set envs and config / 实现set envs and 配置
```python
def _set_envs_and_config(server_args: ServerArgs):
    # Set global environments
    if "NCCL_CUMEM_ENABLE" not in os.environ or server_args.enable_symm_mem:
        os.environ["NCCL_CUMEM_ENABLE"] = str(int(server_args.enable_symm_mem))
    if (
        "NCCL_NVLS_ENABLE" not in os.environ
        or server_args.enable_nccl_nvls
        or server_args.enable_symm_mem
    ):
        os.environ["NCCL_NVLS_ENABLE"] = str(
            int(server_args.enable_nccl_nvls or server_args.enable_symm_mem)
        )
    os.environ["CUDA_DEVICE_MAX_CONNECTIONS"] = "8"
    os.environ["CUDA_MODULE_LOADING"] = "AUTO"

    if os.environ.get("TRTLLM_ENABLE_PDL", "1") != "0":
        # flashinfer uses this environment variable for various kernels from MoE to quant kernels
        os.environ["TRTLLM_ENABLE_PDL"] = "1"

    if os.environ.get("CUTE_DSL_LOG_LEVEL") is None:
        # Default to warning level, to avoid too many logs
        os.environ["CUTE_DSL_LOG_LEVEL"] = "30"

    if os.environ.get("CUTE_DSL_LOG_TO_CONSOLE") is None:
        # Need to set log to console, otherwise the log level won't take effect
        os.environ["CUTE_DSL_LOG_TO_CONSOLE"] = "1"

    # Can also be passed as argument
    os.environ["SGLANG_RUN_ID"] = (
        f"sglang-run-{time.time()}-{random.randint(0, 100000000)}"
    )

    # Set prometheus env vars
    if server_args.enable_metrics:
        set_prometheus_multiproc_dir()
```
**EN:** This block implements the function `_set_envs_and_config(server_args)`. It focuses on handling the engine responsibilities represented by `_set_envs_and_config`, providing reusable behavior for the engine pipeline.
**CN:** 该代码块实现函数 `_set_envs_and_config(server_args)`。它围绕 `_set_envs_and_config` 所承担的 引擎 相关职责展开，为对应处理链路提供可复用能力。

### Lines 1251-1286: Continue set envs and config / 继续说明set envs and 配置
```python
    # Set ulimit
    set_ulimit()

    # Check flashinfer version
    if not get_bool_env_var("SGLANG_SKIP_SGL_KERNEL_VERSION_CHECK"):
        if server_args.attention_backend == "flashinfer":
            assert_pkg_version(
                "flashinfer_python",
                "0.6.11.post1",
                "Please uninstall the old version and "
                "reinstall the latest version by following the instructions "
                "at https://docs.flashinfer.ai/installation.html.",
            )
        if _is_cuda:
            assert_pkg_version(
                "sglang-kernel",
                "0.4.2.post2",
                "Please reinstall the latest version with `pip install sglang-kernel --force-reinstall`",
            )

    # Signal handlers can only be registered from the main thread.
    if threading.current_thread() is threading.main_thread():
        if server_args.custom_sigquit_handler is None:
            # Register the signal handler.
            # The child processes will send SIGQUIT to this process when any error happens
            # This process then clean up the whole process tree
            # Note: This sigquit handler is used in the launch phase, and may be replaced by
            # the running_phase_sigquit_handler in the tokenizer manager after the grpc server is launched.
            def launch_phase_sigquit_handler(signum, frame):
                logger.error(
                    "Received sigquit from a child process. It usually means the child failed."
                )
                kill_process_tree(os.getpid())

            signal.signal(signal.SIGQUIT, launch_phase_sigquit_handler)
        else:
```
**EN:** This block implements the function `_set_envs_and_config(server_args)`. It focuses on handling the engine responsibilities represented by `_set_envs_and_config`, providing reusable behavior for the engine pipeline.
**CN:** 该代码块实现函数 `_set_envs_and_config(server_args)`。它围绕 `_set_envs_and_config` 所承担的 引擎 相关职责展开，为对应处理链路提供可复用能力。

### Lines 1287-1300: Continue set envs and config / 继续说明set envs and 配置
```python
            # Allow users to register a custom SIGQUIT handler for things like crash dump
            logger.error(
                f"Using custom SIGQUIT handler: {server_args.custom_sigquit_handler}"
            )
            signal.signal(signal.SIGQUIT, server_args.custom_sigquit_handler)
    else:
        logger.warning(
            "Signal handler is not added because the engine is not in the "
            "main thread. This disables the SIGQUIT handler for cleaning up "
            "the process tree when a child process fails."
        )

    # Set mp start method
    mp.set_start_method("spawn", force=True)
```
**EN:** This block implements the function `_set_envs_and_config(server_args)`. It focuses on handling the engine responsibilities represented by `_set_envs_and_config`, providing reusable behavior for the engine pipeline.
**CN:** 该代码块实现函数 `_set_envs_and_config(server_args)`。它围绕 `_set_envs_and_config` 所承担的 引擎 相关职责展开，为对应处理链路提供可复用能力。

### Lines 1303-1307: Implement set gc / 实现set gc
```python
def _set_gc(server_args: ServerArgs):
    if gc_threshold := server_args.gc_threshold:
        import gc

        gc.set_threshold(*gc_threshold)
```
**EN:** This block implements the function `_set_gc(server_args)`. It focuses on handling the engine responsibilities represented by `_set_gc`, providing reusable behavior for the engine pipeline.
**CN:** 该代码块实现函数 `_set_gc(server_args)`。它围绕 `_set_gc` 所承担的 引擎 相关职责展开，为对应处理链路提供可复用能力。

### Lines 1310-1318: Implement scheduler died error / 实现调度器 died error
```python
def _scheduler_died_error(rank: int, proc) -> RuntimeError:
    """Build a descriptive error for a scheduler process that died during init."""
    proc.join(timeout=10)
    return RuntimeError(
        f"Rank {rank} scheduler died during initialization "
        f"(exit code: {proc.exitcode}). "
        f"If exit code is -9 (SIGKILL), a common cause is the OS OOM killer. "
        f"Run `dmesg -T | grep -i oom` to check."
    )
```
**EN:** This block implements the function `_scheduler_died_error(rank, proc)`. It focuses on Build a descriptive error for a scheduler process that died during init., providing reusable behavior for the engine pipeline.
**CN:** 该代码块实现函数 `_scheduler_died_error(rank, proc)`。它围绕 `_scheduler_died_error` 所承担的 引擎 相关职责展开，为对应处理链路提供可复用能力。

### Lines 1321-1350: Implement wait for scheduler ready / 实现wait for 调度器 ready
```python
def _wait_for_scheduler_ready(
    scheduler_pipe_readers: List,
    scheduler_procs: List,
) -> List[Dict]:
    """Wait for the model to finish loading and return scheduler infos.

    Uses poll() with timeout instead of blocking recv(), so that child process
    death (e.g. OOM SIGKILL) is detected promptly instead of hanging forever.
    """
    scheduler_infos = []
    for i in range(len(scheduler_pipe_readers)):
        while True:
            if scheduler_pipe_readers[i].poll(timeout=5.0):
                try:
                    data = scheduler_pipe_readers[i].recv()
                except EOFError:
                    raise _scheduler_died_error(i, scheduler_procs[i])
                if data["status"] != "ready":
                    raise RuntimeError(
                        "Initialization failed. Please see the error messages above."
                    )
                scheduler_infos.append(data)
                break

            # Poll timed out — check all processes for early death
            for j in range(len(scheduler_procs)):
                if not scheduler_procs[j].is_alive():
                    raise _scheduler_died_error(j, scheduler_procs[j])

    return scheduler_infos
```
**EN:** This block implements the function `_wait_for_scheduler_ready(scheduler_pipe_readers, scheduler_procs)`. It focuses on Wait for the model to finish loading and return scheduler infos., providing reusable behavior for the engine pipeline.
**CN:** 该代码块实现函数 `_wait_for_scheduler_ready(scheduler_pipe_readers, scheduler_procs)`。它围绕 `_wait_for_scheduler_ready` 所承担的 引擎 相关职责展开，为对应处理链路提供可复用能力。

### Lines 1353-1385: Implement calculate rank ranges / 实现calculate rank ranges
```python
def _calculate_rank_ranges(
    nnodes: int, pp_size: int, tp_size: int, node_rank: int
) -> Tuple[range, range, int, int]:
    """Calculate pp_rank_range and tp_rank_range for a given node.

    Args:
        nnodes: Total number of nodes.
        pp_size: Pipeline parallel size.
        tp_size: Tensor parallel size.
        node_rank: The rank of the node to compute ranges for.

    Returns:
        A tuple of (pp_rank_range, tp_rank_range, pp_size_per_node, tp_size_per_node):
        - pp_rank_range: range of pipeline-parallel ranks assigned to this node.
        - tp_rank_range: range of tensor-parallel ranks assigned to this node.
        - pp_size_per_node: number of PP ranks per node.
        - tp_size_per_node: number of TP ranks per node.
    """
    pp_size_per_node = max(pp_size // nnodes, 1)
    nnodes_per_pp_rank = max(nnodes // pp_size, 1)
    pp_rank_range = range(
        pp_size_per_node * (node_rank // nnodes_per_pp_rank),
        pp_size_per_node * (node_rank // nnodes_per_pp_rank + 1),
    )

    nnodes_per_tp_group = nnodes_per_pp_rank
    tp_size_per_node = tp_size // nnodes_per_tp_group
    tp_rank_range = range(
        tp_size_per_node * (node_rank % nnodes_per_tp_group),
        tp_size_per_node * (node_rank % nnodes_per_tp_group + 1),
    )

    return pp_rank_range, tp_rank_range, pp_size_per_node, tp_size_per_node
```
**EN:** This block implements the function `_calculate_rank_ranges(nnodes, pp_size, tp_size, node_rank)`. It focuses on Calculate pp_rank_range and tp_rank_range for a given node., providing reusable behavior for the engine pipeline.
**CN:** 该代码块实现函数 `_calculate_rank_ranges(nnodes, pp_size, tp_size, node_rank)`。它围绕 `_calculate_rank_ranges` 所承担的 引擎 相关职责展开，为对应处理链路提供可复用能力。

### Lines 1388-1405: Implement compute parallelism ranks / 实现compute parallelism ranks
```python
def _compute_parallelism_ranks(
    server_args: ServerArgs, tp_rank: int
) -> Tuple[int, int, int]:
    """Compute attention-CP, MoE-DP, and MoE-EP ranks for a TP rank."""
    attn_dp_size = server_args.dp_size if server_args.enable_dp_attention else 1

    # Parallelism hierarchy (outermost to innermost):
    # - Attention: Global(TP) -> DP -> ATTN_CP -> ATTN_TP (innermost)
    # - MoE: Global(TP) -> MOE_DP -> EP -> MOE_TP (innermost)
    attn_tp_size = server_args.tp_size // attn_dp_size // server_args.attn_cp_size
    attn_cp_rank = (tp_rank // attn_tp_size) % server_args.attn_cp_size
    moe_dp_rank = tp_rank // (server_args.tp_size // server_args.moe_dp_size)
    moe_ep_rank = (
        tp_rank
        % (server_args.tp_size // server_args.moe_dp_size)
        // (server_args.tp_size // server_args.moe_dp_size // server_args.ep_size)
    )
    return attn_cp_rank, moe_dp_rank, moe_ep_rank
```
**EN:** This block implements the function `_compute_parallelism_ranks(server_args, tp_rank)`. It focuses on Compute attention-CP, MoE-DP, and MoE-EP ranks for a TP rank., providing reusable behavior for the engine pipeline.
**CN:** 该代码块实现函数 `_compute_parallelism_ranks(server_args, tp_rank)`。它围绕 `_compute_parallelism_ranks` 所承担的 引擎 相关职责展开，为对应处理链路提供可复用能力。

## Key Concepts / 关键概念
- **Core types / 核心类型**: SchedulerInitResult, Engine
- **Main callables / 主要可调用对象**: init_tokenizer_manager, _set_envs_and_config, _set_gc, _scheduler_died_error, _wait_for_scheduler_ready, _calculate_rank_ranges, _compute_parallelism_ranks
- **Domain focus / 领域焦点**: engine / 引擎
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: asyncio, dataclasses, gc, logging, os, tempfile, threading, time, typing, warnings
- **Third-party / 第三方库**: __future__, atexit, multiprocessing, random, signal, torch, uvloop, zmq
- **Local Modules / 本地模块**: sglang.srt.elastic_ep.expert_backup_manager, sglang.srt.entrypoints.EngineBase, sglang.srt.entrypoints.engine_info_bootstrap_server, sglang.srt.entrypoints.engine_score_mixin, sglang.srt.managers.data_parallel_controller, sglang.srt.managers.detokenizer_manager, sglang.srt.managers.io_struct, sglang.srt.managers.multi_tokenizer_mixin, sglang.srt.managers.scheduler, sglang.srt.managers.template_detection, sglang.srt.managers.template_manager, sglang.srt.managers.tokenizer_manager
