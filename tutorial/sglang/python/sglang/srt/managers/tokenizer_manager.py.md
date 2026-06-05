# tokenizer_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/tokenizer_manager.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements tokenizer manager logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 分词器 管理器 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Provide supporting module logic / 提供辅助模块逻辑
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
"""TokenizerManager is a process that tokenizes the text."""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 16-32: Import runtime dependencies / 导入运行时依赖
```python
import asyncio
import copy
import dataclasses
import json
import logging
import os
import pickle
import signal
import socket
import sys
import threading
from collections import deque
from contextlib import nullcontext
from datetime import datetime
from enum import Enum
from http import HTTPStatus
from typing import Any, Awaitable, Dict, List, Optional, Tuple, Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 34-40: Import runtime dependencies / 导入运行时依赖
```python
import fastapi
import pybase64
import torch
import uvloop
import zmq
import zmq.asyncio
from fastapi import BackgroundTasks
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 42-101: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.configs.model_config import ModelConfig
from sglang.srt.constants import HEALTH_CHECK_RID_PREFIX
from sglang.srt.disaggregation.encode_receiver import create_mm_receiver
from sglang.srt.disaggregation.utils import DisaggregationMode
from sglang.srt.environ import envs
from sglang.srt.lora.lora_registry import LoRARef, LoRARegistry
from sglang.srt.managers.async_dynamic_batch_tokenizer import AsyncDynamicbatchTokenizer
from sglang.srt.managers.disagg_service import start_disagg_service
from sglang.srt.managers.embed_types import PositionalEmbeds
from sglang.srt.managers.io_struct import (
    AbortReq,
    ActiveRanksOutput,
    BatchEmbeddingOutput,
    BatchStrOutput,
    BatchTokenIDOutput,
    BatchTokenizedEmbeddingReqInput,
    BatchTokenizedGenerateReqInput,
    ConfigureLoggingReq,
    ContinueGenerationReqInput,
    EmbeddingReqInput,
    FreezeGCReq,
    GenerateReqInput,
    HealthCheckOutput,
    LoadLoRAAdapterReqInput,
    OpenSessionReqOutput,
    PauseGenerationReqInput,
    SessionParams,
    TokenizedEmbeddingReqInput,
    TokenizedGenerateReqInput,
    UpdateWeightFromDiskReqInput,
    UpdateWeightFromDiskReqOutput,
    WatchLoadUpdateReq,
)
from sglang.srt.managers.mm_utils import TensorTransportMode, wrap_shm_features
from sglang.srt.managers.multimodal_processor import get_mm_processor, import_processors
from sglang.srt.managers.schedule_batch import MultimodalDataItem
from sglang.srt.managers.scheduler_input_blocker import input_blocker_guard_region
from sglang.srt.managers.tokenizer_control_mixin import TokenizerControlMixin
from sglang.srt.managers.tokenizer_manager_score_mixin import (
    TokenizerManagerScoreMixin,
)
from sglang.srt.managers.utils import is_health_check_generate_req
from sglang.srt.observability.cpu_monitor import start_cpu_monitor_thread
from sglang.srt.observability.metrics_collector import TokenizerMetricsCollector
from sglang.srt.observability.req_time_stats import (
    APIServerReqTimeStats,
    convert_time_to_realtime,
    real_time,
    set_time_batch,
)
from sglang.srt.observability.request_metrics_exporter import (
    RequestMetricsExporterManager,
)
from sglang.srt.observability.trace import SpanAttributes, extract_trace_headers
from sglang.srt.sampling.sampling_params import SamplingParams
from sglang.srt.server_args import (
    PortArgs,
    ServerArgs,
    set_global_server_args_for_tokenizer,
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 102-119: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.speculative.spec_info import SpeculativeAlgorithm
from sglang.srt.utils import (
    configure_gc_warning,
    freeze_gc,
    get_bool_env_var,
    get_or_create_event_loop,
    kill_process_tree,
)
from sglang.srt.utils.aio_rwlock import RWLock
from sglang.srt.utils.hf_transformers_utils import (
    get_processor,
    get_tokenizer,
    get_tokenizer_from_processor,
)
from sglang.srt.utils.network import get_zmq_socket
from sglang.srt.utils.request_logger import RequestLogger
from sglang.srt.utils.watchdog import Watchdog
from sglang.utils import TypeBasedDispatcher, get_exception_traceback
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 121-121: Provide supporting module logic / 提供辅助模块逻辑
```python
asyncio.set_event_loop_policy(uvloop.EventLoopPolicy())
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 123-123: Provide supporting module logic / 提供辅助模块逻辑
```python
_REQUEST_STATE_WAIT_TIMEOUT = envs.SGLANG_REQUEST_STATE_WAIT_TIMEOUT.get()
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 125-125: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 127-134: Provide supporting module logic / 提供辅助模块逻辑
```python
_INCREMENTAL_STREAMING_META_INFO_KEYS = (
    "output_token_logprobs",
    "output_top_logprobs",
    "output_token_ids_logprobs",
)


@dataclasses.dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 135-196: Define class ReqState / 定义类 ReqState
```python
class ReqState:
    """Store the state a request."""

    out_list: List[Dict[Any, Any]]
    finished: bool
    event: asyncio.Event
    obj: Union[GenerateReqInput, EmbeddingReqInput]

    # For performance metrics
    time_stats: APIServerReqTimeStats
    last_completion_tokens: int = 1
    ttft_observed: bool = False

    # For streaming output
    last_output_offset: int = 0

    # Accumulate text lazily so incremental streaming can emit the incoming
    # delta directly without rebuilding the full output prefix.
    text: str = ""
    text_chunks: List[str] = dataclasses.field(default_factory=list)

    def append_text(self, chunk: str):
        if chunk:
            self.text_chunks.append(chunk)

    def get_text(self) -> str:
        if self.text_chunks:
            self.text += "".join(self.text_chunks)
            self.text_chunks.clear()
        return self.text

    def get_crash_dump_output(self) -> Dict[Any, Any]:
        out = {}
        if self.text or self.text_chunks:
            out["text"] = self.get_text()
        if self.output_ids:
            out["output_ids"] = self.output_ids.copy()
        return out

    # For incremental state update.
    # TODO(lianmin): do not initialize some lists if not needed.
    output_ids: List[int] = dataclasses.field(default_factory=list)
    input_token_logprobs_val: List[float] = dataclasses.field(default_factory=list)
    input_token_logprobs_idx: List[int] = dataclasses.field(default_factory=list)
    output_token_logprobs_val: List[float] = dataclasses.field(default_factory=list)
    output_token_logprobs_idx: List[int] = dataclasses.field(default_factory=list)
    input_top_logprobs_val: List[List[float]] = dataclasses.field(default_factory=list)
    input_top_logprobs_idx: List[List[int]] = dataclasses.field(default_factory=list)
    output_top_logprobs_val: List[List[float]] = dataclasses.field(default_factory=list)
    output_top_logprobs_idx: List[List[int]] = dataclasses.field(default_factory=list)
    input_token_ids_logprobs_val: List = dataclasses.field(default_factory=list)
    input_token_ids_logprobs_idx: List = dataclasses.field(default_factory=list)
    output_token_ids_logprobs_val: List = dataclasses.field(default_factory=list)
    output_token_ids_logprobs_idx: List = dataclasses.field(default_factory=list)

    # For detokenized logprobs
    input_token_logprobs: List[Any] = dataclasses.field(default_factory=list)
    output_token_logprobs: List[Any] = dataclasses.field(default_factory=list)
    input_top_logprobs: List[Any] = dataclasses.field(default_factory=list)
    output_top_logprobs: List[Any] = dataclasses.field(default_factory=list)
    input_token_ids_logprobs: List[Any] = dataclasses.field(default_factory=list)
    output_token_ids_logprobs: List[Any] = dataclasses.field(default_factory=list)
```
**EN:** This block declares the class `ReqState`. It centers on Store the state a request., with methods such as append_text, get_text, get_crash_dump_output.
**CN:** 该代码块声明类 `ReqState`。它负责承载与 分词器 管理器 相关的核心状态与行为，并通过 append_text, get_text, get_crash_dump_output 等方法组织实现。

### Lines 199-205: Implement slice streaming output meta info / 实现slice streaming output meta info
```python
def _slice_streaming_output_meta_info(
    meta_info: Dict[Any, Any],
    last_output_offset: int,
) -> None:
    """Align output-side metadata with the current incremental streaming chunk."""
    for key in meta_info.keys() & set(_INCREMENTAL_STREAMING_META_INFO_KEYS):
        meta_info[key] = meta_info[key][last_output_offset:]
```
**EN:** This block implements the function `_slice_streaming_output_meta_info(meta_info, last_output_offset)`. It focuses on Align output-side metadata with the current incremental streaming chunk., providing reusable behavior for the tokenizer manager pipeline.
**CN:** 该代码块实现函数 `_slice_streaming_output_meta_info(meta_info, last_output_offset)`。它围绕 `_slice_streaming_output_meta_info` 所承担的 分词器 管理器 相关职责展开，为对应处理链路提供可复用能力。

### Lines 208-213: Define class InputFormat / 定义类 InputFormat
```python
class InputFormat(Enum):
    """Input format types for tokenization handling."""

    SINGLE_STRING = 1  # Regular single text like "Hello world"
    BATCH_STRINGS = 2  # Regular batch like ["Hello", "World"]
    CROSS_ENCODER_PAIRS = 3  # Cross-encoder pairs like [["query", "document"]]
```
**EN:** This block declares the class `InputFormat`. It centers on Input format types for tokenization handling..
**CN:** 该代码块声明类 `InputFormat`。它负责承载与 分词器 管理器 相关的核心状态与行为。

### Lines 216-217: Provide supporting module logic / 提供辅助模块逻辑
```python
class TokenizerManager(TokenizerControlMixin, TokenizerManagerScoreMixin):
    """TokenizerManager is a process that tokenizes the text."""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 219-259: Initialize TokenizerManager / 初始化 TokenizerManager
```python
    def __init__(
        self,
        server_args: ServerArgs,
        port_args: PortArgs,
    ):
        # Parse args
        self.server_args = server_args
        self.enable_metrics = server_args.enable_metrics
        self.preferred_sampling_params = server_args.preferred_sampling_params
        self.crash_dump_folder = server_args.crash_dump_folder
        set_global_server_args_for_tokenizer(server_args)

        # Init model config
        self.init_model_config()

        # Initialize tokenizer and multimodalprocessor
        self.init_tokenizer_and_processor()

        # Init inter-process communication
        self.init_ipc_channels(port_args)

        # Init running status
        self.init_running_status()

        # Init logging and dumping
        self.init_request_logging_and_dumping()

        # Init weight update
        self.init_weight_update()

        # Init LoRA status
        self.init_lora()

        # Init PD disaggregation and encoder disaggregation
        self.init_disaggregation()

        # Init metric collector and watchdog
        self.init_metric_collector_watchdog()

        # Init request dispatcher
        self.init_request_dispatcher()
```
**EN:** This block implements the initializer `__init__(server_args, port_args)` for `TokenizerManager`. It prepares the object state and connects the instance to the surrounding tokenizer manager workflow.
**CN:** 该代码块实现 `TokenizerManager` 的初始化方法 `__init__(server_args, port_args)`。它负责准备对象状态，并把实例接入 分词器 管理器 相关的运行流程。

### Lines 261-287: Implement init model config / 实现init 模型 配置
```python
    def init_model_config(self):
        server_args = self.server_args
        model_config_class = getattr(self, "model_config_class", ModelConfig)

        # Read model args
        self.model_path = server_args.model_path
        self.served_model_name = server_args.served_model_name
        self.model_config = model_config_class.from_server_args(server_args)
        self.is_generation = self.model_config.is_generation
        self.context_len = self.model_config.context_len
        self.image_token_id = self.model_config.image_token_id
        self.max_req_input_len = None  # Will be set later in engine.py
        self.enable_priority_scheduling = server_args.enable_priority_scheduling
        self.default_priority_value = server_args.default_priority_value
        speculative_algorithm = SpeculativeAlgorithm.from_string(
            server_args.speculative_algorithm
        )
        if speculative_algorithm.is_eagle():
            # In the current eagle implementation, we store the draft tokens in the output token slots,
            # so we need to reserve the space for the draft tokens.
            self.num_reserved_tokens = max(
                server_args.speculative_eagle_topk * server_args.speculative_num_steps,
                server_args.speculative_num_draft_tokens,
            )
        else:
            self.num_reserved_tokens = 0
        self.validate_total_tokens = True
```
**EN:** This block implements the method `init_model_config()` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `init_model_config`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `init_model_config()`。它围绕 `init_model_config` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 289-324: Implement init tokenizer and processor / 实现init 分词器 and 处理器
```python
    def init_tokenizer_and_processor(self):
        server_args = self.server_args

        # Initialize tokenizer and processor
        if self.model_config.is_multimodal:
            import_processors("sglang.srt.multimodal.processors")
            if mm_process_pkg := envs.SGLANG_EXTERNAL_MM_PROCESSOR_PACKAGE.get():
                import_processors(mm_process_pkg, overwrite=True)
            _processor = _get_processor_wrapper(server_args)
            transport_mode = _determine_tensor_transport_mode(self.server_args)

            # We want to parallelize the image pre-processing so we create an executor for it
            # We create mm_processor for any skip_tokenizer_init to make sure we still encode
            # images even with skip_tokenizer_init=False.
            self.mm_processor = get_mm_processor(
                self.model_config.hf_config,
                server_args,
                _processor,
                transport_mode,
                model_config=self.model_config,
            )

            if server_args.skip_tokenizer_init:
                self.tokenizer = self.processor = None
            else:
                self.processor = _processor
                self.tokenizer = get_tokenizer_from_processor(self.processor)
                os.environ["TOKENIZERS_PARALLELISM"] = "false"
        else:
            self.mm_processor = self.processor = None

            if server_args.skip_tokenizer_init:
                self.tokenizer = None
            else:
                self.tokenizer = get_tokenizer(
                    server_args.tokenizer_path,
```
**EN:** This block implements the method `init_tokenizer_and_processor()` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `init_tokenizer_and_processor`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `init_tokenizer_and_processor()`。它围绕 `init_tokenizer_and_processor` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 325-342: Continue init tokenizer and processor / 继续说明init 分词器 and 处理器
```python
                    tokenizer_mode=server_args.tokenizer_mode,
                    trust_remote_code=server_args.trust_remote_code,
                    revision=server_args.revision,
                    tokenizer_backend=server_args.tokenizer_backend,
                )

        # Initialize async dynamic batch tokenizer if enabled (common for both multimodal and non-multimodal)
        if (
            server_args.enable_dynamic_batch_tokenizer
            and not server_args.skip_tokenizer_init
        ):
            self.async_dynamic_batch_tokenizer = AsyncDynamicbatchTokenizer(
                self.tokenizer,
                max_batch_size=server_args.dynamic_batch_tokenizer_batch_size,
                batch_wait_timeout_s=server_args.dynamic_batch_tokenizer_batch_timeout,
            )
        else:
            self.async_dynamic_batch_tokenizer = None
```
**EN:** This block implements the method `init_tokenizer_and_processor()` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `init_tokenizer_and_processor`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `init_tokenizer_and_processor()`。它围绕 `init_tokenizer_and_processor` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 344-362: Implement init ipc channels / 实现init ipc channels
```python
    def init_ipc_channels(self, port_args: PortArgs):
        context = zmq.asyncio.Context(2)
        self.recv_from_detokenizer = get_zmq_socket(
            context, zmq.PULL, port_args.tokenizer_ipc_name, True
        )
        if self.server_args.tokenizer_worker_num == 1:
            self.send_to_scheduler = get_zmq_socket(
                context, zmq.PUSH, port_args.scheduler_input_ipc_name, True
            )
        else:
            from sglang.srt.managers.multi_tokenizer_mixin import SenderWrapper

            # Use tokenizer_worker_ipc_name in multi-tokenizer mode
            send_to_scheduler = get_zmq_socket(
                context, zmq.PUSH, port_args.tokenizer_worker_ipc_name, False
            )

            # Make sure that each request carries the tokenizer_ipc_name for response routing
            self.send_to_scheduler = SenderWrapper(port_args, send_to_scheduler)
```
**EN:** This block implements the method `init_ipc_channels(port_args)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `init_ipc_channels`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `init_ipc_channels(port_args)`。它围绕 `init_ipc_channels` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 364-379: Implement init running status / 实现init running status
```python
    def init_running_status(self):
        # Request states
        self.rid_to_state: Dict[str, ReqState] = {}
        self.event_loop = None
        self.asyncio_tasks = set()

        # Health check
        self.server_status = ServerStatus.Starting
        self.gracefully_exit = False
        self.last_receive_tstamp = real_time()

        # Session
        self.session_futures = {}  # session_id -> asyncio event

        # Subprocess liveness watchdog — set by Engine or http_server after construction
        self._subprocess_watchdog = None
```
**EN:** This block implements the method `init_running_status()` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `init_running_status`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `init_running_status()`。它围绕 `init_running_status` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 381-406: Implement init request logging and dumping / 实现init 请求 logging and dumping
```python
    def init_request_logging_and_dumping(self):
        # TODO: Refactor and organize the log export code.
        # Request logging
        self.request_logger = RequestLogger(
            log_requests=self.server_args.log_requests,
            log_requests_level=self.server_args.log_requests_level,
            log_requests_format=self.server_args.log_requests_format,
            log_requests_target=self.server_args.log_requests_target,
        )

        # Dumping
        self.dump_requests_folder = ""  # By default do not dump
        self.dump_requests_threshold = 1000
        self.dump_requests_exclude_meta_keys: List[str] = [
            "routed_experts",
            "hidden_states",
        ]
        self.dump_request_list: List[Tuple] = []
        self.crash_dump_request_list: deque[Tuple] = deque()
        self.crash_dump_performed = False  # Flag to ensure dump is only called once

        # Initialize performance metrics loggers with proper skip names
        _, obj_skip_names, out_skip_names = self.request_logger.metadata
        self.request_metrics_exporter_manager = RequestMetricsExporterManager(
            self.server_args, obj_skip_names, out_skip_names
        )
```
**EN:** This block implements the method `init_request_logging_and_dumping()` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `init_request_logging_and_dumping`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `init_request_logging_and_dumping()`。它围绕 `init_request_logging_and_dumping` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 408-421: Implement init weight update / 实现init weight update
```python
    def init_weight_update(self):
        # Initial weights status
        self.initial_weights_loaded = True
        if self.server_args.checkpoint_engine_wait_weights_before_ready:
            self.initial_weights_loaded = False

        # Weight updates
        # The event to notify the weight sync is finished.
        self.model_update_lock = RWLock()
        self.model_update_result: Optional[Awaitable[UpdateWeightFromDiskReqOutput]] = (
            None
        )
        self.is_pause = False
        self.is_pause_cond = asyncio.Condition()
```
**EN:** This block implements the method `init_weight_update()` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `init_weight_update`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `init_weight_update()`。它围绕 `init_weight_update` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 423-440: Implement init lora / 实现init lora
```python
    def init_lora(self):
        # LoRA
        # Initialize the `LoRARegistry` with initial LoRA adapter paths provided in `server_args`.
        # The registry dynamically updates as adapters are loaded / unloaded during runtime. It
        # serves as the source of truth for available adapters and maps user-friendly LoRA names
        # to internally used unique LoRA IDs.
        self.lora_registry = LoRARegistry(self.server_args.lora_paths)
        # Lock to serialize LoRA update operations.
        # Please note that, unlike `model_update_lock`, this does not block inference, allowing
        # LoRA updates and inference to overlap.
        self.lora_update_lock = asyncio.Lock()
        # A cache for mapping the lora_name for LoRA adapters that have been loaded at any
        # point to their latest LoRARef objects, so that they can be
        # dynamically loaded if needed for inference
        self.lora_ref_cache: Dict[str, LoRARef] = {}
        if self.server_args.lora_paths is not None:
            for lora_ref in self.server_args.lora_paths:
                self.lora_ref_cache[lora_ref.lora_name] = lora_ref
```
**EN:** This block implements the method `init_lora()` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `init_lora`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `init_lora()`。它围绕 `init_lora` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 442-456: Implement init disaggregation / 实现init disaggregation
```python
    def init_disaggregation(self):
        # PD Disaggregation
        self.disaggregation_mode = DisaggregationMode(
            self.server_args.disaggregation_mode
        )
        start_disagg_service(self.server_args)
        # Single-source counter for auto-assigning fake bootstrap_room.
        self.fake_bootstrap_room_counter = 0

        # Encoder Disaggregation
        if self.server_args.language_only:
            self.mm_receiver = create_mm_receiver(
                self.server_args,
                dtype=self.model_config.dtype,
            )
```
**EN:** This block implements the method `init_disaggregation()` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `init_disaggregation`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `init_disaggregation()`。它围绕 `init_disaggregation` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 458-493: Implement init metric collector watchdog / 实现init 指标 collector watchdog
```python
    def init_metric_collector_watchdog(self):
        # Metrics
        if self.enable_metrics:
            engine_type = DisaggregationMode.to_engine_type(
                self.server_args.disaggregation_mode
            )

            labels = {
                "model_name": self.server_args.served_model_name,
                "engine_type": engine_type,
            }
            if self.enable_priority_scheduling:
                labels["priority"] = ""
            if self.server_args.tokenizer_metrics_allowed_custom_labels:
                for label in self.server_args.tokenizer_metrics_allowed_custom_labels:
                    labels[label] = ""
            if self.server_args.extra_metric_labels:
                labels.update(self.server_args.extra_metric_labels)
            self.metrics_collector = TokenizerMetricsCollector(
                server_args=self.server_args,
                labels=labels,
                bucket_time_to_first_token=self.server_args.bucket_time_to_first_token,
                bucket_e2e_request_latency=self.server_args.bucket_e2e_request_latency,
                bucket_inter_token_latency=self.server_args.bucket_inter_token_latency,
            )

            start_cpu_monitor_thread("tokenizer")

        if self.server_args.gc_warning_threshold_secs > 0.0:
            configure_gc_warning(self.server_args.gc_warning_threshold_secs)
        self.soft_watchdog = Watchdog.create(
            debug_name="TokenizerManager",
            watchdog_timeout=self.server_args.soft_watchdog_timeout,
            soft=True,
            test_stuck_time=envs.SGLANG_TEST_STUCK_TOKENIZER.get(),
        )
```
**EN:** This block implements the method `init_metric_collector_watchdog()` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `init_metric_collector_watchdog`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `init_metric_collector_watchdog()`。它围绕 `init_metric_collector_watchdog` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 495-513: Implement init request dispatcher / 实现init 请求 dispatcher
```python
    def init_request_dispatcher(self):
        self._result_dispatcher = TypeBasedDispatcher(
            [
                (AbortReq, self._handle_abort_req),
                (OpenSessionReqOutput, self._handle_open_session_req_output),
                (
                    UpdateWeightFromDiskReqOutput,
                    self._handle_update_weights_from_disk_req_output,
                ),
                (FreezeGCReq, lambda x: None),
                # For handling case when scheduler skips detokenizer and forwards back to the tokenizer manager, we ignore it.
                (HealthCheckOutput, lambda x: None),
                (ActiveRanksOutput, self.update_active_ranks),
            ]
        )
        self.init_communicators(self.server_args)

        self.sampling_params_class = SamplingParams
        self.signal_handler_class = SignalHandler
```
**EN:** This block implements the method `init_request_dispatcher()` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `init_request_dispatcher`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `init_request_dispatcher()`。它围绕 `init_request_dispatcher` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 515-560: Implement async generate request / 实现异步generate 请求
```python
    async def generate_request(
        self,
        obj: Union[GenerateReqInput, EmbeddingReqInput],
        request: Optional[fastapi.Request] = None,
    ):
        self.auto_create_handle_loop()

        # Normalize the request
        obj.normalize_batch_and_arguments()
        self._set_default_priority(obj)

        if isinstance(obj, GenerateReqInput) and obj.routed_dp_rank is not None:
            dp_size = self.server_args.dp_size
            if dp_size <= 1 and obj.routed_dp_rank == 0:
                logger.warning(
                    f"routed_dp_rank={obj.routed_dp_rank} is ignored because dp_size={dp_size}"
                )
            elif obj.routed_dp_rank < 0 or obj.routed_dp_rank >= dp_size:
                raise ValueError(
                    f"routed_dp_rank={obj.routed_dp_rank} out of range [0, {dp_size})"
                )

        self._init_req_state(obj, request)
        if self.server_args.language_only:
            self._handle_epd_disaggregation_encode_request(obj)
        if self.server_args.tokenizer_worker_num > 1:
            self._attach_multi_http_worker_info(obj)

        # Log the request
        self.request_logger.log_received_request(obj, self.tokenizer, request)

        async with self.is_pause_cond:
            await self.is_pause_cond.wait_for(lambda: not self.is_pause)

        async with self.model_update_lock.reader_lock:
            await self._validate_and_resolve_lora(obj)

            # Tokenize the request and send it to the scheduler
            if obj.is_single:
                tokenized_obj = await self._tokenize_one_request(obj)
                self._send_one_request(tokenized_obj)
                async for response in self._wait_one_response(obj, request):
                    yield response
            else:
                async for response in self._handle_batch_request(obj, request):
                    yield response
```
**EN:** This block implements the async method `generate_request(obj, request)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `generate_request`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `generate_request(obj, request)`。它围绕 `generate_request` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 562-583: Implement detect input format / 实现detect input format
```python
    def _detect_input_format(
        self, texts: Union[str, List[str]], is_cross_encoder: bool
    ) -> InputFormat:
        """Detect the format of input texts for proper tokenization handling.

        Returns:
            - InputFormat.SINGLE_STRING: Regular single text like "Hello world"
            - InputFormat.BATCH_STRINGS: Regular batch like ["Hello", "World"]
            - InputFormat.CROSS_ENCODER_PAIRS: Cross-encoder pairs like [["query", "document"]]
        """
        if isinstance(texts, str):
            return InputFormat.SINGLE_STRING

        if (
            is_cross_encoder
            and len(texts) > 0
            and isinstance(texts[0], list)
            and len(texts[0]) == 2
        ):
            return InputFormat.CROSS_ENCODER_PAIRS

        return InputFormat.BATCH_STRINGS
```
**EN:** This block implements the method `_detect_input_format(texts, is_cross_encoder)` on `TokenizerManager`. It focuses on Detect the format of input texts for proper tokenization handling., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_detect_input_format(texts, is_cross_encoder)`。它围绕 `_detect_input_format` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 585-594: Implement prepare tokenizer input / 实现prepare 分词器 input
```python
    def _prepare_tokenizer_input(
        self, texts: Union[str, List[str]], input_format: InputFormat
    ) -> Union[List[str], List[List[str]]]:
        """Prepare input for the tokenizer based on detected format."""
        if input_format == InputFormat.SINGLE_STRING:
            return [texts]  # Wrap single string for batch processing
        elif input_format == InputFormat.CROSS_ENCODER_PAIRS:
            return texts  # Already in correct format: [["query", "doc"]]
        else:  # BATCH_STRINGS
            return texts  # Already in correct format: ["text1", "text2"]
```
**EN:** This block implements the method `_prepare_tokenizer_input(texts, input_format)` on `TokenizerManager`. It focuses on Prepare input for the tokenizer based on detected format., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_prepare_tokenizer_input(texts, input_format)`。它围绕 `_prepare_tokenizer_input` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 596-618: Implement extract tokenizer results / 实现extract 分词器 results
```python
    def _extract_tokenizer_results(
        self,
        input_ids: List[List[int]],
        token_type_ids: Optional[List[List[int]]],
        input_format: InputFormat,
        original_batch_size: int,
    ) -> Union[
        Tuple[List[int], Optional[List[int]]],
        Tuple[List[List[int]], Optional[List[List[int]]]],
    ]:
        """Extract results from tokenizer output based on input format."""

        # For single inputs (string or single cross-encoder pair), extract first element
        if (
            input_format in [InputFormat.SINGLE_STRING, InputFormat.CROSS_ENCODER_PAIRS]
            and original_batch_size == 1
        ):
            single_input_ids = input_ids[0] if input_ids else []
            single_token_type_ids = token_type_ids[0] if token_type_ids else None
            return single_input_ids, single_token_type_ids

        # For true batches, return as-is
        return input_ids, token_type_ids
```
**EN:** This block implements the method `_extract_tokenizer_results(input_ids, token_type_ids, input_format, original_batch_size)` on `TokenizerManager`. It focuses on Extract results from tokenizer output based on input format., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_extract_tokenizer_results(input_ids, token_type_ids, input_format, original_batch_size)`。它围绕 `_extract_tokenizer_results` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 620-655: Implement async tokenize texts / 实现异步tokenize texts
```python
    async def _tokenize_texts(
        self, texts: Union[str, List[str]], is_cross_encoder: bool = False
    ) -> Union[
        Tuple[List[int], Optional[List[int]]],
        Tuple[List[List[int]], Optional[List[List[int]]]],
    ]:
        """
        Tokenize text(s) using the appropriate tokenizer strategy.

        This method handles multiple input formats and chooses between async dynamic
        batch tokenizer (for single texts only) and regular tokenizer.

        Args:
            texts: Text input in various formats:

                   Regular cases:
                   - Single string: "How are you?"
                   - Batch of strings: ["Hello", "World", "How are you?"]

                   Cross-encoder cases (sentence pairs for similarity/ranking):
                   - Single pair: [["query text", "document text"]]
                   - Multiple pairs: [["q1", "d1"], ["q2", "d2"], ["q3", "d3"]]

            is_cross_encoder: Whether to return token_type_ids for cross-encoder models.
                             Enables proper handling of sentence pairs with segment IDs.

        Returns:
            Single input cases:
                Tuple[List[int], Optional[List[int]]]: (input_ids, token_type_ids)
                Example: ([101, 2129, 102], [0, 0, 0]) for single text
                Example: ([101, 2129, 102, 4068, 102], [0, 0, 0, 1, 1]) for cross-encoder pair

            Batch input cases:
                Tuple[List[List[int]], Optional[List[List[int]]]]: (batch_input_ids, batch_token_type_ids)
                Example: ([[101, 2129, 102], [101, 4068, 102]], None) for regular batch
```
**EN:** This block implements the async method `_tokenize_texts(texts, is_cross_encoder)` on `TokenizerManager`. It focuses on Tokenize text(s) using the appropriate tokenizer strategy., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_tokenize_texts(texts, is_cross_encoder)`。它围绕 `_tokenize_texts` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 656-691: Continue async tokenize texts / 继续说明异步tokenize texts
```python
            Note: token_type_ids is None unless is_cross_encoder=True.
        """
        if not texts or self.tokenizer is None:
            raise ValueError("texts cannot be empty and tokenizer must be initialized")

        # Step 1: Detect input format and prepare for tokenization
        input_format = self._detect_input_format(texts, is_cross_encoder)
        tokenizer_input = self._prepare_tokenizer_input(texts, input_format)
        original_batch_size = len(texts) if not isinstance(texts, str) else 1

        # Step 2: Set up tokenizer arguments
        tokenizer_kwargs = (
            {"return_token_type_ids": is_cross_encoder} if is_cross_encoder else {}
        )

        # Step 3: Choose tokenization strategy
        use_async_tokenizer = (
            self.async_dynamic_batch_tokenizer is not None
            and input_format == InputFormat.SINGLE_STRING
        )

        if use_async_tokenizer:
            logger.debug("Using async dynamic batch tokenizer for single text")
            result = await self.async_dynamic_batch_tokenizer.encode(
                tokenizer_input[0], **tokenizer_kwargs
            )
            # Convert to batch format for consistency
            input_ids = [result["input_ids"]]
            token_type_ids = (
                [result["token_type_ids"]]
                if is_cross_encoder and result.get("token_type_ids")
                else None
            )
        else:
            logger.debug(f"Using regular tokenizer for {len(tokenizer_input)} inputs")
```
**EN:** This block implements the async method `_tokenize_texts(texts, is_cross_encoder)` on `TokenizerManager`. It focuses on Tokenize text(s) using the appropriate tokenizer strategy., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_tokenize_texts(texts, is_cross_encoder)`。它围绕 `_tokenize_texts` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 692-705: Continue async tokenize texts / 继续说明异步tokenize texts
```python
            if not is_cross_encoder and (not getattr(self.tokenizer, "is_fast", False)):
                input_ids = [self.tokenizer.encode(t) for t in tokenizer_input]
                token_type_ids = None
            else:
                encoded = self.tokenizer(tokenizer_input, **tokenizer_kwargs)
                input_ids = encoded["input_ids"]
                token_type_ids = (
                    encoded.get("token_type_ids") if is_cross_encoder else None
                )

        # Step 4: Extract results based on input format
        return self._extract_tokenizer_results(
            input_ids, token_type_ids, input_format, original_batch_size
        )
```
**EN:** This block implements the async method `_tokenize_texts(texts, is_cross_encoder)` on `TokenizerManager`. It focuses on Tokenize text(s) using the appropriate tokenizer strategy., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_tokenize_texts(texts, is_cross_encoder)`。它围绕 `_tokenize_texts` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 707-742: Implement async tokenize one request / 实现异步tokenize one 请求
```python
    async def _tokenize_one_request(
        self,
        obj: Union[GenerateReqInput, EmbeddingReqInput],
    ):
        """Tokenize one request."""
        # Tokenize
        input_embeds = None
        input_text = obj.text
        token_type_ids = None
        is_cross_encoder_request = (
            isinstance(obj, EmbeddingReqInput) and obj.is_cross_encoder_request
        )
        if obj.input_embeds is not None:
            if not self.server_args.disable_radix_cache:
                raise ValueError(
                    "input_embeds is provided while disable_radix_cache is False. "
                    "Please add `--disable-radix-cache` when you launch the server "
                    "if you want to use input_embeds as inputs."
                )
            input_embeds = obj.input_embeds
            input_ids = obj.input_ids
        elif obj.input_ids is not None:
            input_ids = obj.input_ids
        else:
            if self.tokenizer is None:
                raise ValueError(
                    "The engine initialized with skip_tokenizer_init=True cannot "
                    "accept text prompts. Please provide input_ids or re-initialize "
                    "the engine with skip_tokenizer_init=False."
                )

            # For audio-only requests (e.g., Whisper), text may be empty.
            # The multimodal processor will provide input_ids later.
            if not input_text and self.mm_processor and obj.contains_mm_input():
                # Use empty placeholder - multimodal processor will override
                input_ids = []
```
**EN:** This block implements the async method `_tokenize_one_request(obj)` on `TokenizerManager`. It focuses on Tokenize one request., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_tokenize_one_request(obj)`。它围绕 `_tokenize_one_request` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 743-778: Continue async tokenize one request / 继续说明异步tokenize one 请求
```python
            else:
                input_ids, token_type_ids = await self._tokenize_texts(
                    input_text, is_cross_encoder_request
                )

        contains_mm_input = obj.contains_mm_input()
        is_mossvl = (
            "MossVLForConditionalGeneration"
            in self.model_config.hf_config.architectures
        )
        should_run_mm_processor = self.mm_processor is not None and (
            contains_mm_input or is_mossvl
        )

        if should_run_mm_processor:
            if obj.image_data is not None and not isinstance(obj.image_data, list):
                obj.image_data = [obj.image_data]
            if obj.video_data is not None and not isinstance(obj.video_data, list):
                obj.video_data = [obj.video_data]
            if obj.audio_data is not None and not isinstance(obj.audio_data, list):
                obj.audio_data = [obj.audio_data]
            if contains_mm_input:
                self._validate_mm_limits(obj)

            mm_inputs = None

            if (
                not self.server_args.language_only
                or self.server_args.encoder_transfer_backend
                in ["zmq_to_tokenizer", "mooncake"]
            ):
                if self.server_args.language_only:
                    mm_inputs = await self.mm_receiver.recv_mm_data(
                        request_obj=obj,
                        mm_processor=self.mm_processor,
                        prompt=(input_text or input_ids),
```
**EN:** This block implements the async method `_tokenize_one_request(obj)` on `TokenizerManager`. It focuses on Tokenize one request., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_tokenize_one_request(obj)`。它围绕 `_tokenize_one_request` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 779-814: Continue async tokenize one request / 继续说明异步tokenize one 请求
```python
                        need_wait_for_mm_inputs=obj.need_wait_for_mm_inputs,
                    )
                if mm_inputs is None:
                    if self.server_args.language_only:
                        logger.warning(
                            "Encoder embedding not available, "
                            "falling back to local mm processing"
                        )
                    mm_inputs = await self.mm_processor.process_mm_data_async(
                        image_data=obj.image_data,
                        audio_data=obj.audio_data,
                        input_text=(input_text or input_ids),
                        request_obj=obj,
                        max_req_input_len=self.max_req_input_len,
                    )
            elif (
                self.server_args.language_only
                and self.server_args.encoder_transfer_backend == "zmq_to_scheduler"
                and not obj.need_wait_for_mm_inputs
            ):
                # In language_only mode with zmq_to_scheduler, if we didn't dispatch
                # to encoder (e.g., only one image), process locally like non-language_only mode
                mm_inputs = await self.mm_processor.process_mm_data_async(
                    image_data=obj.image_data,
                    audio_data=obj.audio_data,
                    input_text=(input_text or input_ids),
                    request_obj=obj,
                    max_req_input_len=self.max_req_input_len,
                )

            if mm_inputs and mm_inputs.input_ids is not None:
                input_ids = mm_inputs.input_ids
            if mm_inputs and mm_inputs.token_type_ids is not None:
                token_type_ids = mm_inputs.token_type_ids
                if not isinstance(token_type_ids, list):
                    token_type_ids = token_type_ids.flatten().tolist()
```
**EN:** This block implements the async method `_tokenize_one_request(obj)` on `TokenizerManager`. It focuses on Tokenize one request., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_tokenize_one_request(obj)`。它围绕 `_tokenize_one_request` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 815-829: Continue async tokenize one request / 继续说明异步tokenize one 请求
```python
            if (
                envs.SGLANG_MM_PRECOMPUTE_HASH.get()
                and mm_inputs
                and mm_inputs.mm_items
            ):
                for item in mm_inputs.mm_items:
                    if isinstance(item, MultimodalDataItem):
                        item.set_pad_value()
        else:
            mm_inputs = None

        self._validate_one_request(obj, input_ids)
        return self._create_tokenized_object(
            obj, input_text, input_ids, input_embeds, mm_inputs, token_type_ids
        )
```
**EN:** This block implements the async method `_tokenize_one_request(obj)` on `TokenizerManager`. It focuses on Tokenize one request., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_tokenize_one_request(obj)`。它围绕 `_tokenize_one_request` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 831-866: Implement validate one request / 实现validate one 请求
```python
    def _validate_one_request(
        self, obj: Union[GenerateReqInput, EmbeddingReqInput], input_ids: List[int]
    ) -> None:
        """Validates that the input token count and the requested token count doesn't exceed the model's context length."""
        # FIXME: unify the length validation logic with the one in the scheduler.
        _max_req_len = self.context_len
        input_token_num = len(input_ids) if input_ids is not None else 0
        input_token_num += self.num_reserved_tokens

        # Validate input length
        if input_token_num >= self.context_len:
            if self.server_args.allow_auto_truncate:
                logger.warning(
                    f"The input ({input_token_num} tokens) is longer than the "
                    f"model's context length ({self.context_len} tokens). "
                    "Truncating the input."
                )
                del input_ids[_max_req_len:]
                input_token_num = len(input_ids)
            else:
                raise ValueError(
                    f"The input ({input_token_num} tokens) is longer than the "
                    f"model's context length ({self.context_len} tokens)."
                )

        # Validate total tokens (input + max_new_tokens)
        max_new_tokens = obj.sampling_params.get("max_new_tokens")
        if (
            self.validate_total_tokens
            and max_new_tokens is not None
            and (max_new_tokens + input_token_num) > _max_req_len
        ):
            if self.server_args.allow_auto_truncate:
                logger.warning(
                    f"Requested token count ({input_token_num} input + {max_new_tokens} new) "
                    f"exceeds the model's context length ({self.context_len} tokens). "
```
**EN:** This block implements the method `_validate_one_request(obj, input_ids)` on `TokenizerManager`. It focuses on Validates that the input token count and the requested token count doesn't exceed the model's context length., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_validate_one_request(obj, input_ids)`。它围绕 `_validate_one_request` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 867-902: Continue validate one request / 继续说明validate one 请求
```python
                    "Truncating max_new_tokens."
                )
                obj.sampling_params["max_new_tokens"] = max(
                    0, _max_req_len - input_token_num
                )
            else:
                total_tokens = max_new_tokens + input_token_num
                error_msg = (
                    f"Requested token count exceeds the model's maximum context length "
                    f"of {self.context_len} tokens. You requested a total of {total_tokens} "
                    f"tokens: {input_token_num} tokens from the input messages and "
                    f"{max_new_tokens} tokens for the completion. Please reduce the number "
                    f"of tokens in the input messages or the completion to fit within the limit."
                )
                raise ValueError(error_msg)

        # Validate embedding requests
        if isinstance(obj, EmbeddingReqInput) and self.is_generation:
            raise ValueError(
                "This model does not appear to be an embedding model by default. "
                "Please add `--is-embedding` when launching the server or try another model."
            )

        # Validate Matryoshka embeddings
        if isinstance(obj, EmbeddingReqInput):
            self._validate_for_matryoshka_dim(obj)

        # Validate custom logit processor
        if isinstance(obj, GenerateReqInput):
            if (
                obj.return_hidden_states
                and not self.server_args.enable_return_hidden_states
            ):
                raise ValueError(
                    "The server is not configured to return the hidden states. "
                    "Please set `--enable-return-hidden-states` to enable this feature."
```
**EN:** This block implements the method `_validate_one_request(obj, input_ids)` on `TokenizerManager`. It focuses on Validates that the input token count and the requested token count doesn't exceed the model's context length., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_validate_one_request(obj, input_ids)`。它围绕 `_validate_one_request` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 903-911: Continue validate one request / 继续说明validate one 请求
```python
                )
            if (
                obj.custom_logit_processor
                and not self.server_args.enable_custom_logit_processor
            ):
                raise ValueError(
                    "The server is not configured to enable custom logit processor. "
                    "Please set `--enable-custom-logit-processor` to enable this feature."
                )
```
**EN:** This block implements the method `_validate_one_request(obj, input_ids)` on `TokenizerManager`. It focuses on Validates that the input token count and the requested token count doesn't exceed the model's context length., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_validate_one_request(obj, input_ids)`。它围绕 `_validate_one_request` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 913-926: Implement validate mm limits / 实现validate mm limits
```python
    def _validate_mm_limits(
        self, obj: Union[GenerateReqInput, EmbeddingReqInput]
    ) -> None:
        if not self.server_args.limit_mm_data_per_request:
            return

        for modality, limit in self.server_args.limit_mm_data_per_request.items():
            data = getattr(obj, f"{modality}_data", None)
            if data:
                count = len(data) if isinstance(data, list) else 1
                if count > limit:
                    raise ValueError(
                        f"{modality.capitalize()} count {count} exceeds limit {limit} per request."
                    )
```
**EN:** This block implements the method `_validate_mm_limits(obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_validate_mm_limits`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_validate_mm_limits(obj)`。它围绕 `_validate_mm_limits` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 928-954: Implement validate for matryoshka dim / 实现validate for matryoshka dim
```python
    def _validate_for_matryoshka_dim(self, obj: EmbeddingReqInput) -> None:
        """Validate the request for Matryoshka dim if it has the field set."""
        if obj.dimensions is None:
            return

        if not self.model_config.is_matryoshka:
            raise ValueError(
                f"Model '{self.model_config.model_path}' does not support matryoshka representation, "
                f"changing output dimensions will lead to poor results."
            )

        if obj.dimensions < 1:
            raise ValueError("Requested dimensions must be greater than 0")

        if (
            self.model_config.matryoshka_dimensions
            and obj.dimensions not in self.model_config.matryoshka_dimensions
        ):
            raise ValueError(
                f"Model '{self.model_config.model_path}' only supports {self.model_config.matryoshka_dimensions} matryoshka dimensions, "
                f"using other output dimensions will lead to poor results."
            )

        if obj.dimensions > self.model_config.hidden_size:
            raise ValueError(
                f"Provided dimensions are greater than max embedding dimension: {self.model_config.hidden_size}"
            )
```
**EN:** This block implements the method `_validate_for_matryoshka_dim(obj)` on `TokenizerManager`. It focuses on Validate the request for Matryoshka dim if it has the field set., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_validate_for_matryoshka_dim(obj)`。它围绕 `_validate_for_matryoshka_dim` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 956-972: Implement validate input ids in vocab / 实现validate input ids in vocab
```python
    def _validate_input_ids_in_vocab(
        self, input_ids: Union[List[int], List[List[int]]], vocab_size: int
    ) -> None:
        # Handle both single sequence and batch of sequences
        if isinstance(input_ids[0], list):
            # Batch of sequences
            for seq in input_ids:
                if any(id >= vocab_size for id in seq):
                    raise ValueError(
                        f"The input_ids {seq} contains values greater than the vocab size ({vocab_size})."
                    )
        else:
            # Single sequence
            if any(id >= vocab_size for id in input_ids):
                raise ValueError(
                    f"The input_ids {input_ids} contains values greater than the vocab size ({vocab_size})."
                )
```
**EN:** This block implements the method `_validate_input_ids_in_vocab(input_ids, vocab_size)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_validate_input_ids_in_vocab`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_validate_input_ids_in_vocab(input_ids, vocab_size)`。它围绕 `_validate_input_ids_in_vocab` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 974-1009: Implement create tokenized object / 实现create tokenized object
```python
    def _create_tokenized_object(
        self,
        obj: Union[GenerateReqInput, EmbeddingReqInput],
        input_text: str,
        input_ids: List[int],
        input_embeds: Optional[Union[List[float], None]] = None,
        mm_inputs=None,
        token_type_ids: Optional[List[int]] = None,
    ) -> Union[TokenizedGenerateReqInput, TokenizedEmbeddingReqInput]:
        """Create a tokenized request object from common parameters."""
        # Parse sampling parameters
        # Note: if there are preferred sampling params, we use them if they are not
        # explicitly passed in sampling_params
        if self.preferred_sampling_params:
            sampling_kwargs = {**self.preferred_sampling_params, **obj.sampling_params}
        else:
            sampling_kwargs = obj.sampling_params
        sampling_params = self.sampling_params_class(**sampling_kwargs)
        sampling_params.normalize(self.tokenizer)
        sampling_params.verify(self.model_config.vocab_size)

        # Build return object
        if isinstance(obj, GenerateReqInput):
            session_params = (
                SessionParams(**obj.session_params) if obj.session_params else None
            )

            bootstrap_room = obj.bootstrap_room
            if (
                bootstrap_room is None
                and self.server_args.disaggregation_transfer_backend == "fake"
            ):
                bootstrap_room = self.fake_bootstrap_room_counter
                self.fake_bootstrap_room_counter += 1

            tokenized_obj = TokenizedGenerateReqInput(
```
**EN:** This block implements the method `_create_tokenized_object(obj, input_text, input_ids, input_embeds, mm_inputs, ...)` on `TokenizerManager`. It focuses on Create a tokenized request object from common parameters., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_create_tokenized_object(obj, input_text, input_ids, input_embeds, mm_inputs, ...)`。它围绕 `_create_tokenized_object` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1010-1045: Continue create tokenized object / 继续说明create tokenized object
```python
                input_text,
                input_ids,
                mm_inputs,
                sampling_params,
                obj.return_logprob,
                obj.logprob_start_len,
                obj.top_logprobs_num,
                obj.token_ids_logprob,
                obj.stream,
                rid=obj.rid,
                http_worker_ipc=obj.http_worker_ipc,
                bootstrap_host=obj.bootstrap_host,
                bootstrap_port=obj.bootstrap_port,
                bootstrap_room=bootstrap_room,
                lora_id=obj.lora_id,
                input_embeds=input_embeds,
                positional_embed_overrides=obj.positional_embed_overrides,
                session_params=session_params,
                custom_logit_processor=obj.custom_logit_processor,
                require_reasoning=obj.require_reasoning,
                return_hidden_states=obj.return_hidden_states,
                return_routed_experts=obj.return_routed_experts,
                routed_experts_start_len=obj.routed_experts_start_len,
                return_indexer_topk=obj.return_indexer_topk,
                routed_dp_rank=obj.routed_dp_rank,
                disagg_prefill_dp_rank=obj.disagg_prefill_dp_rank,
                priority=obj.priority,
                extra_key=obj.extra_key,
                routing_key=obj.routing_key,
                token_type_ids=token_type_ids,
                need_wait_for_mm_inputs=obj.need_wait_for_mm_inputs,
                num_items_assigned=obj.num_items_assigned,
                multi_item_delimiter_indices=obj.multi_item_delimiter_indices,
            )
        elif isinstance(obj, EmbeddingReqInput):
            # Resolve unresolved embed overrides now that input_ids are available
```
**EN:** This block implements the method `_create_tokenized_object(obj, input_text, input_ids, input_embeds, mm_inputs, ...)` on `TokenizerManager`. It focuses on Create a tokenized request object from common parameters., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_create_tokenized_object(obj, input_text, input_ids, input_embeds, mm_inputs, ...)`。它围绕 `_create_tokenized_object` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1046-1075: Continue create tokenized object / 继续说明create tokenized object
```python
            positional_embed_overrides = obj.positional_embed_overrides
            if (
                positional_embed_overrides is None
                and obj.embed_overrides is not None
                and obj.embed_override_token_id is not None
            ):
                positional_embed_overrides = self._resolve_embed_overrides(
                    input_ids, obj.embed_override_token_id, obj.embed_overrides
                )

            tokenized_obj = TokenizedEmbeddingReqInput(
                input_text,
                input_ids,
                mm_inputs,
                token_type_ids,
                sampling_params,
                positional_embed_overrides=positional_embed_overrides,
                rid=obj.rid,
                priority=obj.priority,
                dimensions=obj.dimensions,
                lora_id=obj.lora_id,
                http_worker_ipc=obj.http_worker_ipc,
                return_pooled_hidden_states=obj.return_pooled_hidden_states,
                multi_item_delimiter_indices=obj.multi_item_delimiter_indices,
            )

        tokenized_obj.time_stats = self.rid_to_state[obj.rid].time_stats
        self.rid_to_state[obj.rid].time_stats.set_tokenize_finish_time()

        return tokenized_obj
```
**EN:** This block implements the method `_create_tokenized_object(obj, input_text, input_ids, input_embeds, mm_inputs, ...)` on `TokenizerManager`. It focuses on Create a tokenized request object from common parameters., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_create_tokenized_object(obj, input_text, input_ids, input_embeds, mm_inputs, ...)`。它围绕 `_create_tokenized_object` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1076-1077: Provide supporting module logic / 提供辅助模块逻辑
```python

    @staticmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 1078-1095: Implement resolve embed overrides / 实现resolve embed overrides
```python
    def _resolve_embed_overrides(
        input_ids: List[int],
        token_id: int,
        embeds: List[torch.Tensor],
    ) -> PositionalEmbeds:
        """Resolve placeholder positions in input_ids and create PositionalEmbeds.

        Scans input_ids for occurrences of token_id and pairs them with the
        provided embedding tensors.
        """
        positions = [idx for idx, tok in enumerate(input_ids) if tok == token_id]
        if len(positions) != len(embeds):
            raise ValueError(
                f"input contains {len(positions)} occurrences of "
                f"embed_override_token_id={token_id}, "
                f"but embed_overrides has {len(embeds)} entries."
            )
        return PositionalEmbeds(embeds=embeds, positions=positions)
```
**EN:** This block implements the method `_resolve_embed_overrides(input_ids, token_id, embeds)` on `TokenizerManager`. It focuses on Resolve placeholder positions in input_ids and create PositionalEmbeds., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_resolve_embed_overrides(input_ids, token_id, embeds)`。它围绕 `_resolve_embed_overrides` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1097-1139: Implement async batch tokenize and process / 实现异步批处理 tokenize and process
```python
    async def _batch_tokenize_and_process(
        self, batch_size: int, obj: Union[GenerateReqInput, EmbeddingReqInput]
    ) -> List[Union[TokenizedGenerateReqInput, TokenizedEmbeddingReqInput]]:
        """Handle batch tokenization for text inputs only."""
        logger.debug(f"Starting batch tokenization for {batch_size} text requests")

        # If batch does not have text nothing to tokenize
        # so lets construct the return object
        if not self._batch_has_text(batch_size, obj):
            # All requests already have input_ids, no need to tokenize
            return [await self._tokenize_one_request(obj[i]) for i in range(batch_size)]

        self._validate_batch_tokenization_constraints(batch_size, obj)

        # Collect requests and texts
        requests = [obj[i] for i in range(batch_size)]
        texts = [req.text for req in requests]

        # Check if any request is a cross-encoder request
        is_cross_encoder_request = any(
            isinstance(req, EmbeddingReqInput) and req.is_cross_encoder_request
            for req in requests
        )

        # Batch tokenize all texts using unified method
        input_ids_list, token_type_ids_list = await self._tokenize_texts(
            texts, is_cross_encoder_request
        )

        # Process all requests
        tokenized_objs = []
        for i, req in enumerate(requests):
            self._validate_one_request(obj[i], input_ids_list[i])
            token_type_ids = (
                token_type_ids_list[i] if token_type_ids_list is not None else None
            )
            tokenized_objs.append(
                self._create_tokenized_object(
                    req, req.text, input_ids_list[i], None, None, token_type_ids
                )
            )
        logger.debug(f"Completed batch processing for {batch_size} requests")
        return tokenized_objs
```
**EN:** This block implements the async method `_batch_tokenize_and_process(batch_size, obj)` on `TokenizerManager`. It focuses on Handle batch tokenization for text inputs only., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_batch_tokenize_and_process(batch_size, obj)`。它围绕 `_batch_tokenize_and_process` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1141-1157: Implement validate batch tokenization constraints / 实现validate 批处理 tokenization constraints
```python
    def _validate_batch_tokenization_constraints(
        self, batch_size: int, obj: Union[GenerateReqInput, EmbeddingReqInput]
    ) -> None:
        """Validate constraints for batch tokenization processing."""
        for i in range(batch_size):
            if self.is_generation and obj[i].contains_mm_input():
                raise ValueError(
                    "For multimodal input processing do not set `enable_tokenizer_batch_encode`."
                )
            if obj[i].input_ids is not None:
                raise ValueError(
                    "Batch tokenization is not needed for pre-tokenized input_ids. Do not set `enable_tokenizer_batch_encode`."
                )
            if obj[i].input_embeds is not None:
                raise ValueError(
                    "Batch tokenization is not needed for input_embeds. Do not set `enable_tokenizer_batch_encode`."
                )
```
**EN:** This block implements the method `_validate_batch_tokenization_constraints(batch_size, obj)` on `TokenizerManager`. It focuses on Validate constraints for batch tokenization processing., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_validate_batch_tokenization_constraints(batch_size, obj)`。它围绕 `_validate_batch_tokenization_constraints` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1159-1169: Implement batch has text / 实现批处理 has text
```python
    def _batch_has_text(
        self, batch_size: int, obj: Union[GenerateReqInput, EmbeddingReqInput]
    ) -> bool:
        """Check if any request in the batch contains text input."""
        for i in range(batch_size):
            if obj[i].text:
                return True
            elif self.is_generation and obj[i].contains_mm_input():
                return True

        return False
```
**EN:** This block implements the method `_batch_has_text(batch_size, obj)` on `TokenizerManager`. It focuses on Check if any request in the batch contains text input., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_batch_has_text(batch_size, obj)`。它围绕 `_batch_has_text` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1171-1185: Implement should use batch tokenization / 实现should use 批处理 tokenization
```python
    def _should_use_batch_tokenization(self, batch_size, requests) -> bool:
        """Return True if we should run the tokenizer in batch mode.

        Current policy:
        - Respect explicit server flag `enable_tokenizer_batch_encode`.
        - Or, if no request has text or multimodal input (all use pre-tokenized input_ids or input_embeds), batch the requests without tokenization.
        - Batch tokenization does not support DP attention yet, and it will make everything goes to the first rank currently
        """
        return batch_size > 0 and (
            self.server_args.enable_tokenizer_batch_encode
            or (
                (not self.server_args.enable_dp_attention)
                and (not self._batch_has_text(batch_size, requests))
            )
        )
```
**EN:** This block implements the method `_should_use_batch_tokenization(batch_size, requests)` on `TokenizerManager`. It focuses on Return True if we should run the tokenizer in batch mode., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_should_use_batch_tokenization(batch_size, requests)`。它围绕 `_should_use_batch_tokenization` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1187-1194: Implement send one request / 实现send one 请求
```python
    def _send_one_request(
        self,
        tokenized_obj: Union[TokenizedGenerateReqInput, TokenizedEmbeddingReqInput],
    ):
        tokenized_obj.time_stats.set_api_server_dispatch_time()
        tokenized_obj = wrap_shm_features(tokenized_obj)
        self.send_to_scheduler.send_pyobj(tokenized_obj)
        tokenized_obj.time_stats.set_api_server_dispatch_finish_time()
```
**EN:** This block implements the method `_send_one_request(tokenized_obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_send_one_request`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_send_one_request(tokenized_obj)`。它围绕 `_send_one_request` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1196-1210: Implement send batch request / 实现send 批处理 请求
```python
    def _send_batch_request(
        self,
        tokenized_objs: List[
            Union[TokenizedGenerateReqInput, TokenizedEmbeddingReqInput]
        ],
    ):
        """Send a batch of tokenized requests as a single batched request to the scheduler."""
        if isinstance(tokenized_objs[0], TokenizedGenerateReqInput):
            batch_req = BatchTokenizedGenerateReqInput(batch=tokenized_objs)
        else:
            batch_req = BatchTokenizedEmbeddingReqInput(batch=tokenized_objs)

        set_time_batch(tokenized_objs, "set_api_server_dispatch_time")
        self.send_to_scheduler.send_pyobj(batch_req)
        set_time_batch(tokenized_objs, "set_api_server_dispatch_finish_time")
```
**EN:** This block implements the method `_send_batch_request(tokenized_objs)` on `TokenizerManager`. It focuses on Send a batch of tokenized requests as a single batched request to the scheduler., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_send_batch_request(tokenized_objs)`。它围绕 `_send_batch_request` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1212-1243: Implement coalesce streaming chunks / 实现coalesce streaming chunks
```python
    def _coalesce_streaming_chunks(
        self,
        out_list: list,
        rid: str,
    ) -> dict:
        """Coalesce multiple incremental streaming chunks into one.

        Both text and output_ids are incremental deltas, so we concatenate them;
        all other fields (meta_info, etc.) are taken from the last chunk.
        """
        if len(out_list) >= 20:
            logger.warning(
                "Streaming backlog: rid=%s, coalescing %d queued chunks into one. "
                "This may inflate P99 ITL for affected requests.",
                rid,
                len(out_list),
            )
        out = dict(out_list[-1])
        if "output_ids" in out:
            out["output_ids"] = [id for chunk in out_list for id in chunk["output_ids"]]
        if "text" in out:
            out["text"] = "".join(chunk["text"] for chunk in out_list)
        if "meta_info" in out:
            meta_info_list = [chunk["meta_info"] for chunk in out_list]
            meta_info = dict(meta_info_list[-1])
            for key in _INCREMENTAL_STREAMING_META_INFO_KEYS:
                if any(key in m for m in meta_info_list):
                    meta_info[key] = [
                        item for m in meta_info_list for item in m.get(key, [])
                    ]
            out["meta_info"] = meta_info
        return out
```
**EN:** This block implements the method `_coalesce_streaming_chunks(out_list, rid)` on `TokenizerManager`. It focuses on Coalesce multiple incremental streaming chunks into one., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_coalesce_streaming_chunks(out_list, rid)`。它围绕 `_coalesce_streaming_chunks` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1245-1287: Implement async handle abort finish reason / 实现异步handle abort finish reason
```python
    async def _handle_abort_finish_reason(
        self,
        out: dict,
        state: ReqState,
        is_stream: bool,
    ) -> Optional[dict]:
        """Handle abort/error finish reasons from the scheduler.

        Returns the output dict if it should be yielded (stream abort), or None
        for normal flow. Raises ValueError or HTTPException for non-stream aborts.
        """
        finish_reason = out["meta_info"]["finish_reason"]

        if (
            finish_reason.get("type") == "abort"
            and finish_reason.get("status_code") == HTTPStatus.BAD_REQUEST
        ):
            if not is_stream:
                raise ValueError(finish_reason["message"])
            return out

        if finish_reason.get("type") == "abort" and finish_reason.get(
            "status_code"
        ) in (
            HTTPStatus.SERVICE_UNAVAILABLE,
            HTTPStatus.INTERNAL_SERVER_ERROR,
        ):
            # Delete the key to prevent resending abort request to the scheduler and
            # to ensure aborted request state is cleaned up.
            if state.obj.rid in self.rid_to_state:
                del self.rid_to_state[state.obj.rid]

            # Mark ongoing LoRA request as finished.
            if self.server_args.enable_lora and state.obj.lora_path:
                await self.lora_registry.release(state.obj.lora_id)
            if not is_stream:
                raise fastapi.HTTPException(
                    status_code=finish_reason["status_code"],
                    detail=finish_reason["message"],
                )
            return out

        return None
```
**EN:** This block implements the async method `_handle_abort_finish_reason(out, state, is_stream)` on `TokenizerManager`. It focuses on Handle abort/error finish reasons from the scheduler., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_handle_abort_finish_reason(out, state, is_stream)`。它围绕 `_handle_abort_finish_reason` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1289-1324: Implement async wait one response / 实现异步wait one 响应
```python
    async def _wait_one_response(
        self,
        obj: Union[GenerateReqInput, EmbeddingReqInput],
        request: Optional[fastapi.Request] = None,
    ):
        """Wait for the response of one request."""
        state = self.rid_to_state[obj.rid]
        # Not all request types have `stream` (e.g., EmbeddingReqInput). Default to non-streaming.
        is_stream = getattr(obj, "stream", False)
        while True:
            try:
                await asyncio.wait_for(
                    state.event.wait(), timeout=_REQUEST_STATE_WAIT_TIMEOUT
                )
            except asyncio.TimeoutError:
                if (
                    request is not None
                    and not obj.background
                    and await request.is_disconnected()
                ):
                    # Abort the request for disconnected requests (non-streaming, waiting queue)
                    self.abort_request(obj.rid)
                    # Use exception to kill the whole call stack and asyncio task
                    raise ValueError(
                        f"Request is disconnected from the client side (type 1). Abort request {obj.rid=}"
                    )
                continue

            # Drain all pending outputs atomically.
            out_list = state.out_list
            state.out_list = []
            finished = state.finished
            state.event.clear()

            # With incremental streaming, each chunk is a delta — coalesce
            # multiple queued chunks to avoid dropping token ids.
```
**EN:** This block implements the async method `_wait_one_response(obj, request)` on `TokenizerManager`. It focuses on Wait for the response of one request., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_wait_one_response(obj, request)`。它围绕 `_wait_one_response` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1325-1360: Continue async wait one response / 继续说明异步wait one 响应
```python
            incremental_stream = (
                is_stream and self.server_args.incremental_streaming_output
            )
            if incremental_stream and len(out_list) > 1:
                out = self._coalesce_streaming_chunks(out_list, obj.rid)
            else:
                out = out_list[-1]

            # Resolve deferred text for non-incremental streaming.
            # _handle_batch_output sets "text": None on intermediate chunks
            # to avoid O(n) string rebuild per step (O(n^2) total).
            if (
                is_stream
                and not incremental_stream
                and "text" in out
                and out["text"] is None
            ):
                out["text"] = state.get_text()

            if finished:
                # Record response sent time right before we log finished results and metrics.
                if not state.time_stats.response_sent_to_client_time:
                    state.time_stats.set_response_sent_to_client_time()
                    out["meta_info"][
                        "response_sent_to_client_ts"
                    ] = state.time_stats.get_response_sent_to_client_realtime()
                self.request_logger.log_finished_request(
                    obj,
                    out,
                    request=request,
                )

                if self.request_metrics_exporter_manager.exporter_enabled():
                    asyncio.create_task(
                        self.request_metrics_exporter_manager.write_record(obj, out)
                    )
```
**EN:** This block implements the async method `_wait_one_response(obj, request)` on `TokenizerManager`. It focuses on Wait for the response of one request., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_wait_one_response(obj, request)`。它围绕 `_wait_one_response` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1361-1393: Continue async wait one response / 继续说明异步wait one 响应
```python

                # Check if this was an abort/error created by scheduler
                if isinstance(out["meta_info"].get("finish_reason"), dict):
                    abort_out = await self._handle_abort_finish_reason(
                        out, state, is_stream
                    )
                    if abort_out is not None:
                        yield abort_out
                        break

                yield out
                break

            if is_stream:
                # Record response sent time right before we send response.
                if not state.time_stats.response_sent_to_client_time:
                    state.time_stats.set_response_sent_to_client_time()
                    out["meta_info"][
                        "response_sent_to_client_ts"
                    ] = state.time_stats.get_response_sent_to_client_realtime()
                yield out
            else:
                if (
                    request is not None
                    and not obj.background
                    and await request.is_disconnected()
                ):
                    # Abort the request for disconnected requests (non-streaming, running)
                    self.abort_request(obj.rid)
                    # Use exception to kill the whole call stack and asyncio task
                    raise ValueError(
                        f"Request is disconnected from the client side (type 3). Abort request {obj.rid=}"
                    )
```
**EN:** This block implements the async method `_wait_one_response(obj, request)` on `TokenizerManager`. It focuses on Wait for the response of one request., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_wait_one_response(obj, request)`。它围绕 `_wait_one_response` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1395-1430: Implement async handle batch request / 实现异步handle 批处理 请求
```python
    async def _handle_batch_request(
        self,
        obj: Union[GenerateReqInput, EmbeddingReqInput],
        request: Optional[fastapi.Request] = None,
    ):
        batch_size = obj.batch_size

        generators = []
        rids = []
        if getattr(obj, "parallel_sample_num", 1) == 1:
            if self._should_use_batch_tokenization(batch_size, obj):
                tokenized_objs = await self._batch_tokenize_and_process(batch_size, obj)
                self._send_batch_request(tokenized_objs)

                # Set up generators for each request in the batch
                for i in range(batch_size):
                    tmp_obj = obj[i]
                    generators.append(self._wait_one_response(tmp_obj, request))
                    rids.append(tmp_obj.rid)
            else:
                # Sequential tokenization and processing
                with (
                    input_blocker_guard_region(send_to_scheduler=self.send_to_scheduler)
                    if get_bool_env_var("SGLANG_ENABLE_COLOCATED_BATCH_GEN")
                    else nullcontext()
                ):
                    for i in range(batch_size):
                        tmp_obj = obj[i]
                        tokenized_obj = await self._tokenize_one_request(tmp_obj)
                        self._send_one_request(tokenized_obj)
                        generators.append(self._wait_one_response(tmp_obj, request))
                        rids.append(tmp_obj.rid)
        else:
            # FIXME: When using batch and parallel_sample_num together, the perf is not optimal.
            if batch_size > 128:
                logger.warning(
```
**EN:** This block implements the async method `_handle_batch_request(obj, request)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_handle_batch_request`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_handle_batch_request(obj, request)`。它围绕 `_handle_batch_request` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1431-1466: Continue async handle batch request / 继续说明异步handle 批处理 请求
```python
                    "Sending a single large batch with parallel sampling (n > 1) has not been well optimized. "
                    "The performance might be better if you just duplicate the requests n times or use "
                    "many threads to send them one by one with parallel sampling (n > 1)."
                )

            # Tokenize all requests
            objs = [obj[i] for i in range(batch_size)]
            tokenized_objs = await asyncio.gather(
                *(self._tokenize_one_request(obj) for obj in objs)
            )

            # Cache the common prefix for parallel sampling
            for i in range(batch_size):
                tmp_obj = copy.copy(objs[i])
                tokenized_obj = copy.copy(tokenized_objs[i])
                tokenized_obj.rid = tmp_obj.regenerate_rid()
                tokenized_obj.sampling_params = copy.copy(tokenized_obj.sampling_params)
                tokenized_obj.sampling_params.max_new_tokens = 0
                tokenized_obj.stream = False
                self._init_req_state(tmp_obj)
                self._send_one_request(tokenized_obj)
                await self._wait_one_response(tmp_obj, request).__anext__()

            # Expand requests, assign new rids for them, and send them
            for i in range(batch_size):
                for _ in range(obj.parallel_sample_num):
                    tmp_obj = copy.copy(objs[i])
                    tokenized_obj = copy.copy(tokenized_objs[i])
                    tokenized_obj.rid = tmp_obj.regenerate_rid()
                    self._init_req_state(tmp_obj)
                    tokenized_obj.time_stats = self.rid_to_state[tmp_obj.rid].time_stats
                    self._send_one_request(tokenized_obj)
                    generators.append(self._wait_one_response(tmp_obj, request))
                    rids.append(tmp_obj.rid)

                self.rid_to_state[objs[i].rid].time_stats.set_finished_time()
```
**EN:** This block implements the async method `_handle_batch_request(obj, request)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_handle_batch_request`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_handle_batch_request(obj, request)`。它围绕 `_handle_batch_request` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1467-1491: Continue async handle batch request / 继续说明异步handle 批处理 请求
```python
                del self.rid_to_state[objs[i].rid]

        # Wait for all requests
        is_stream = hasattr(obj, "stream") and obj.stream
        if not is_stream:
            outputs = await asyncio.gather(*(gen.__anext__() for gen in generators))
            yield outputs
        else:
            rid_to_index = {rid: i for i, rid in enumerate(rids)}
            task_map = {asyncio.create_task(gen.__anext__()): gen for gen in generators}
            while task_map:
                done, _ = await asyncio.wait(
                    task_map.keys(), return_when=asyncio.FIRST_COMPLETED
                )

                for task in done:
                    gen = task_map.pop(task)
                    try:
                        result = task.result()
                        result["index"] = rid_to_index[result["meta_info"]["id"]]
                        yield result
                        new_task = asyncio.create_task(gen.__anext__())
                        task_map[new_task] = gen
                    except StopAsyncIteration:
                        pass
```
**EN:** This block implements the async method `_handle_batch_request(obj, request)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_handle_batch_request`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_handle_batch_request(obj, request)`。它围绕 `_handle_batch_request` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1493-1502: Implement abort request / 实现abort 请求
```python
    def abort_request(self, rid: str = "", abort_all: bool = False):
        if not abort_all and rid not in self.rid_to_state:
            return
        req = AbortReq(rid=rid, abort_all=abort_all)
        self.send_to_scheduler.send_pyobj(req)
        if self.enable_metrics:
            # TODO: also use custom_labels from the request
            self.metrics_collector.observe_one_aborted_request(
                self.metrics_collector.labels
            )
```
**EN:** This block implements the method `abort_request(rid, abort_all)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `abort_request`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `abort_request(rid, abort_all)`。它围绕 `abort_request` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1504-1517: Implement async pause generation / 实现异步pause generation
```python
    async def pause_generation(self, obj: PauseGenerationReqInput):
        async with self.is_pause_cond:
            self.is_pause = True
            if obj.mode != "abort":
                await self.send_to_scheduler.send_pyobj(obj)
            else:
                # we are using the model_update_lock to check if there is still on-going requests.
                while True:
                    # TODO: maybe make it async instead of fire-and-forget
                    self.abort_request(abort_all=True)
                    is_locked = await self.model_update_lock.is_locked()
                    if not is_locked:
                        break
                    await asyncio.sleep(1.0)
```
**EN:** This block implements the async method `pause_generation(obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `pause_generation`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `pause_generation(obj)`。它围绕 `pause_generation` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1519-1523: Implement async continue generation / 实现异步continue generation
```python
    async def continue_generation(self, obj: ContinueGenerationReqInput):
        async with self.is_pause_cond:
            self.is_pause = False
            await self.send_to_scheduler.send_pyobj(obj)
            self.is_pause_cond.notify_all()
```
**EN:** This block implements the async method `continue_generation(obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `continue_generation`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `continue_generation(obj)`。它围绕 `continue_generation` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1525-1556: Implement async update weights from disk / 实现异步update weights from disk
```python
    async def update_weights_from_disk(
        self,
        obj: UpdateWeightFromDiskReqInput,
        request: Optional[fastapi.Request] = None,
    ) -> Tuple[bool, str]:
        self.auto_create_handle_loop()

        # default the load format to the server_args
        if obj.load_format is None:
            obj.load_format = self.server_args.load_format
        logger.info("Start update_weights. Load format=%s", obj.load_format)

        if obj.abort_all_requests:
            self.abort_request(abort_all=True)

        # Immediately update the weights if the engine is in paused state
        async with self.is_pause_cond:
            is_paused = self.is_pause

        lock_context = (
            self.model_update_lock.writer_lock if not is_paused else nullcontext()
        )
        async with lock_context:
            success, message, num_paused_requests = (
                await self._wait_for_model_update_from_disk(obj)
            )

        if success and obj.weight_version is not None:
            self._update_weight_version_if_provided(obj.weight_version)
            message += f" Weight version updated to {obj.weight_version}."

        return success, message, num_paused_requests
```
**EN:** This block implements the async method `update_weights_from_disk(obj, request)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `update_weights_from_disk`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `update_weights_from_disk(obj, request)`。它围绕 `update_weights_from_disk` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1558-1562: Implement update model path info / 实现update 模型 path info
```python
    def _update_model_path_info(self, model_path: str, load_format: str):
        self.served_model_name = model_path
        self.server_args.model_path = model_path
        self.server_args.load_format = load_format
        self.model_path = model_path
```
**EN:** This block implements the method `_update_model_path_info(model_path, load_format)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_update_model_path_info`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_update_model_path_info(model_path, load_format)`。它围绕 `_update_model_path_info` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1564-1584: Implement async wait for model update from disk / 实现异步wait for 模型 update from disk
```python
    async def _wait_for_model_update_from_disk(
        self, obj: UpdateWeightFromDiskReqInput
    ) -> Tuple[bool, str]:
        self.send_to_scheduler.send_pyobj(obj)
        self.model_update_result = asyncio.Future()
        if self.server_args.dp_size == 1:
            result = await self.model_update_result
            if result.success:
                self._update_model_path_info(obj.model_path, obj.load_format)
            return result.success, result.message, result.num_paused_requests
        else:  # self.server_args.dp_size > 1
            self.model_update_tmp = []
            result = await self.model_update_result

            all_success = all([r.success for r in result])
            if all_success is True:
                self._update_model_path_info(obj.model_path, obj.load_format)
            all_message = [r.message for r in result]
            all_message = " | ".join(all_message)
            all_paused_requests = [r.num_paused_requests for r in result]
            return all_success, all_message, all_paused_requests
```
**EN:** This block implements the async method `_wait_for_model_update_from_disk(obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_wait_for_model_update_from_disk`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_wait_for_model_update_from_disk(obj)`。它围绕 `_wait_for_model_update_from_disk` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1586-1602: Implement configure logging / 实现configure logging
```python
    def configure_logging(self, obj: ConfigureLoggingReq):
        self.request_logger.configure(
            log_requests=obj.log_requests,
            log_requests_level=obj.log_requests_level,
            log_requests_format=obj.log_requests_format,
        )
        if obj.dump_requests_folder is not None:
            self.dump_requests_folder = obj.dump_requests_folder
        if obj.dump_requests_threshold is not None:
            self.dump_requests_threshold = obj.dump_requests_threshold
        if obj.dump_requests_exclude_meta_keys is not None:
            self.dump_requests_exclude_meta_keys = list(
                obj.dump_requests_exclude_meta_keys
            )
        if obj.crash_dump_folder is not None:
            self.crash_dump_folder = obj.crash_dump_folder
        logging.info(f"Config logging: {obj=}")
```
**EN:** This block implements the method `configure_logging(obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `configure_logging`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `configure_logging(obj)`。它围绕 `configure_logging` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1604-1608: Implement async freeze gc / 实现异步freeze gc
```python
    async def freeze_gc(self):
        """Send a freeze_gc message to the scheduler first, then freeze locally."""
        self.send_to_scheduler.send_pyobj(FreezeGCReq())
        freeze_gc("Tokenizer Manager")
        return None
```
**EN:** This block implements the async method `freeze_gc()` on `TokenizerManager`. It focuses on Send a freeze_gc message to the scheduler first, then freeze locally., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `freeze_gc()`。它围绕 `freeze_gc` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1610-1622: Implement create abort task / 实现create abort task
```python
    def create_abort_task(self, obj: GenerateReqInput):
        # Abort the request if the client is disconnected.
        async def abort_request():
            await asyncio.sleep(2)
            if obj.is_single:
                self.abort_request(obj.rid)
            else:
                for rid in obj.rid:
                    self.abort_request(rid)

        background_tasks = BackgroundTasks()
        background_tasks.add_task(abort_request)
        return background_tasks
```
**EN:** This block implements the method `create_abort_task(obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `create_abort_task`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `create_abort_task(obj)`。它围绕 `create_abort_task` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1624-1647: Implement auto create handle loop / 实现auto create handle loop
```python
    def auto_create_handle_loop(self):
        if self.event_loop is not None:
            return

        # Create and start the handle_loop task
        loop = get_or_create_event_loop()
        self.asyncio_tasks.add(
            loop.create_task(print_exception_wrapper(self.handle_loop))
        )
        self.event_loop = loop

        # We only add signal handler when the tokenizer manager is in the main thread
        # due to the CPython limitation.
        if threading.current_thread() is threading.main_thread():
            signal_handler = self.signal_handler_class(self)
            loop.add_signal_handler(signal.SIGTERM, signal_handler.sigterm_handler)
            # Update the signal handler for the process. It overrides the sigquit handler in the launch phase.
            loop.add_signal_handler(
                signal.SIGQUIT, signal_handler.running_phase_sigquit_handler
            )

        self.asyncio_tasks.add(
            loop.create_task(print_exception_wrapper(self.sigterm_watchdog))
        )
```
**EN:** This block implements the method `auto_create_handle_loop()` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `auto_create_handle_loop`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `auto_create_handle_loop()`。它围绕 `auto_create_handle_loop` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1649-1662: Implement async handle loop / 实现异步handle loop
```python
    async def handle_loop(self):
        """The event loop that handles requests"""
        while True:
            with self.soft_watchdog.disable():
                recv_obj = await self.recv_from_detokenizer.recv_pyobj()
            if isinstance(
                recv_obj,
                (BatchStrOutput, BatchEmbeddingOutput, BatchTokenIDOutput),
            ):
                await self._handle_batch_output(recv_obj)
            else:
                self._result_dispatcher(recv_obj)
            self.last_receive_tstamp = real_time()
            self.soft_watchdog.feed()
```
**EN:** This block implements the async method `handle_loop()` on `TokenizerManager`. It focuses on The event loop that handles requests, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `handle_loop()`。它围绕 `handle_loop` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1664-1699: Implement async handle batch output / 实现异步handle 批处理 output
```python
    async def _handle_batch_output(
        self,
        recv_obj: Union[
            BatchStrOutput,
            BatchEmbeddingOutput,
            BatchTokenIDOutput,
        ],
    ):
        pending_notify: dict[str, ReqState] = {}
        batch_notify_size = self.server_args.batch_notify_size
        for i, rid in enumerate(recv_obj.rids):
            state = self.rid_to_state.get(rid, None)
            if state is None:
                # Known race: /health_generate pops its rid as soon as ANY message bumps last_receive_tstamp.
                if rid.startswith(HEALTH_CHECK_RID_PREFIX):
                    continue
                logger.error(
                    f"Received output for {rid=} but the state was deleted in TokenizerManager."
                )
                continue

            # Build meta_info and return value
            meta_info = {
                "id": rid,
                "finish_reason": recv_obj.finished_reasons[i],
                "prompt_tokens": recv_obj.prompt_tokens[i],
                "weight_version": self.server_args.weight_version,
                "num_retractions": recv_obj.retraction_counts[i],
            }

            if self.enable_metrics:
                if recv_obj.time_stats is not None:
                    scheduler_time_stats = recv_obj.time_stats[i]
                    meta_info.update(scheduler_time_stats.convert_to_output_meta_info())

            if getattr(state.obj, "return_logprob", False):
```
**EN:** This block implements the async method `_handle_batch_output(recv_obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_handle_batch_output`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_handle_batch_output(recv_obj)`。它围绕 `_handle_batch_output` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1700-1735: Continue async handle batch output / 继续说明异步handle 批处理 output
```python
                self.convert_logprob_style(
                    meta_info,
                    state,
                    state.obj.top_logprobs_num,
                    state.obj.token_ids_logprob,
                    state.obj.return_text_in_logprobs
                    and not self.server_args.skip_tokenizer_init,
                    recv_obj,
                    i,
                )

            if not isinstance(recv_obj, BatchEmbeddingOutput):
                meta_info.update(
                    {
                        "reasoning_tokens": recv_obj.reasoning_tokens[i],
                        "completion_tokens": recv_obj.completion_tokens[i],
                        "cached_tokens": recv_obj.cached_tokens[i],
                    }
                )
                # Add detailed cache breakdown if available
                if (
                    hasattr(recv_obj, "cached_tokens_details")
                    and recv_obj.cached_tokens_details
                ):
                    meta_info["cached_tokens_details"] = recv_obj.cached_tokens_details[
                        i
                    ]

            if getattr(recv_obj, "output_hidden_states", None):
                meta_info["hidden_states"] = recv_obj.output_hidden_states[i]
            if getattr(recv_obj, "routed_experts", None):
                val = recv_obj.routed_experts[i]
                if val is not None:
                    # BatchStrOutput is pre-encoded by the detokenizer;
                    # BatchTokenIDOutput (skip_tokenizer_init) bypasses it.
                    if isinstance(val, torch.Tensor):
```
**EN:** This block implements the async method `_handle_batch_output(recv_obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_handle_batch_output`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_handle_batch_output(recv_obj)`。它围绕 `_handle_batch_output` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1736-1771: Continue async handle batch output / 继续说明异步handle 批处理 output
```python
                        val = pybase64.b64encode(val.numpy().tobytes()).decode("utf-8")
                    meta_info["routed_experts"] = val
            if getattr(recv_obj, "indexer_topk", None):
                val = recv_obj.indexer_topk[i]
                if val is not None:
                    if isinstance(val, torch.Tensor):
                        val = pybase64.b64encode(val.numpy().tobytes()).decode("utf-8")
                    meta_info["indexer_topk"] = val
            if getattr(recv_obj, "customized_info", None):
                for k, v in recv_obj.customized_info.items():
                    meta_info[k] = v[i]
            if getattr(recv_obj, "dp_ranks", None):
                meta_info["dp_rank"] = recv_obj.dp_ranks[i]

            state.finished = recv_obj.finished_reasons[i] is not None
            if isinstance(recv_obj, BatchStrOutput):
                # Not all request types have `stream` (e.g., EmbeddingReqInput). Default to non-streaming.
                is_stream = getattr(state.obj, "stream", False)
                incremental = (
                    self.server_args.incremental_streaming_output and is_stream
                )
                delta_text = recv_obj.output_strs[i]
                delta_output_ids = recv_obj.output_ids[i]
                output_offset = state.last_output_offset
                state.append_text(delta_text)
                state.output_ids.extend(delta_output_ids)

                if is_stream:
                    if incremental:
                        output_token_ids = delta_output_ids
                        _slice_streaming_output_meta_info(meta_info, output_offset)
                        state.last_output_offset = len(state.output_ids)
                        out_dict = {
                            "text": delta_text,
                            "output_ids": output_token_ids,
                            "meta_info": meta_info,
```
**EN:** This block implements the async method `_handle_batch_output(recv_obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_handle_batch_output`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_handle_batch_output(recv_obj)`。它围绕 `_handle_batch_output` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1772-1807: Continue async handle batch output / 继续说明异步handle 批处理 output
```python
                        }
                    elif state.finished:
                        out_dict = {
                            "text": state.get_text(),
                            "output_ids": state.output_ids.copy(),
                            "meta_info": meta_info,
                        }
                    else:
                        # Non-incremental intermediate: pass reference (no
                        # copy) and defer text to _wait_one_response to avoid
                        # O(n) per-step cost that compounds to O(n^2).
                        out_dict = {
                            "text": None,
                            "output_ids": state.output_ids,
                            "meta_info": meta_info,
                        }
                elif state.finished:
                    out_dict = {
                        "text": state.get_text(),
                        "output_ids": state.output_ids.copy(),
                        "meta_info": meta_info,
                    }
                else:
                    out_dict = None
            elif isinstance(recv_obj, BatchTokenIDOutput):
                is_stream = getattr(state.obj, "stream", False)
                incremental = (
                    self.server_args.incremental_streaming_output and is_stream
                )
                delta_output_ids = recv_obj.output_ids[i]
                output_offset = state.last_output_offset
                state.output_ids.extend(delta_output_ids)

                if is_stream:
                    if incremental:
                        output_token_ids = delta_output_ids
```
**EN:** This block implements the async method `_handle_batch_output(recv_obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_handle_batch_output`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_handle_batch_output(recv_obj)`。它围绕 `_handle_batch_output` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1808-1843: Continue async handle batch output / 继续说明异步handle 批处理 output
```python
                        _slice_streaming_output_meta_info(meta_info, output_offset)
                        state.last_output_offset = len(state.output_ids)
                        out_dict = {
                            "output_ids": output_token_ids,
                            "meta_info": meta_info,
                        }
                    elif state.finished:
                        out_dict = {
                            "output_ids": state.output_ids.copy(),
                            "meta_info": meta_info,
                        }
                    else:
                        out_dict = {
                            "output_ids": state.output_ids,
                            "meta_info": meta_info,
                        }
                elif state.finished:
                    out_dict = {
                        "output_ids": state.output_ids.copy(),
                        "meta_info": meta_info,
                    }
                else:
                    out_dict = None
            else:
                assert isinstance(recv_obj, BatchEmbeddingOutput)
                out_dict = {
                    "embedding": recv_obj.embeddings[i],
                    "meta_info": meta_info,
                }
                if (
                    recv_obj.pooled_hidden_states is not None
                    and recv_obj.pooled_hidden_states[i] is not None
                ):
                    out_dict["pooled_hidden_state"] = recv_obj.pooled_hidden_states[i]

            # Set first_token_time on the first output batch.
```
**EN:** This block implements the async method `_handle_batch_output(recv_obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_handle_batch_output`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_handle_batch_output(recv_obj)`。它围绕 `_handle_batch_output` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1844-1879: Continue async handle batch output / 继续说明异步handle 批处理 output
```python
            # This is the single write point for first_token_time.
            if state.time_stats.first_token_time == 0.0:
                state.time_stats.set_first_token_time()

            if state.finished:
                if state.time_stats.trace_ctx.tracing_enable:
                    state.time_stats.trace_ctx.trace_set_root_attrs(
                        self.convert_to_span_attrs(state, recv_obj, i)
                    )
                state.time_stats.set_finished_time()
                meta_info["e2e_latency"] = state.time_stats.get_e2e_latency()

                if self.server_args.speculative_algorithm:
                    self._calculate_spec_decoding_metrics(meta_info, recv_obj, i)
                if self.enable_metrics:
                    scheduler_time_stats = (
                        recv_obj.time_stats[i]
                        if recv_obj.time_stats is not None
                        else None
                    )
                    completion_tokens = (
                        recv_obj.completion_tokens[i]
                        if not isinstance(recv_obj, BatchEmbeddingOutput)
                        else 0
                    )
                    meta_info.update(
                        state.time_stats.convert_to_output_meta_info(
                            scheduler_time_stats, completion_tokens
                        )
                    )

                del self.rid_to_state[rid]

                # Mark ongoing LoRA request as finished.
                if self.server_args.enable_lora and state.obj.lora_path:
                    asyncio.create_task(self.lora_registry.release(state.obj.lora_id))
```
**EN:** This block implements the async method `_handle_batch_output(recv_obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_handle_batch_output`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_handle_batch_output(recv_obj)`。它围绕 `_handle_batch_output` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1880-1910: Continue async handle batch output / 继续说明异步handle 批处理 output
```python

            if out_dict is not None:
                state.out_list.append(out_dict)
                pending_notify[rid] = state

                if len(pending_notify) >= batch_notify_size:
                    for s in pending_notify.values():
                        s.event.set()
                    pending_notify = {}
                    await asyncio.sleep(0)

            if self.enable_metrics and state.obj.log_metrics:
                self.collect_metrics(state, recv_obj, i)
            if self.dump_requests_folder and state.finished and state.obj.log_metrics:
                self.dump_requests(state, out_dict)
            if self.crash_dump_folder and state.finished and state.obj.log_metrics:
                self.record_request_for_crash_dump(state, out_dict)

        # handle_loop awaits next recv immediately
        for s in pending_notify.values():
            s.event.set()

        # When skip_tokenizer_init is enabled, tokensizer_manager receives
        # BatchTokenIDOutput.
        if (
            self.server_args.dp_size > 1
            and isinstance(recv_obj, (BatchStrOutput, BatchTokenIDOutput))
            and recv_obj.load is not None
        ):
            load_update_req = WatchLoadUpdateReq(loads=[recv_obj.load])
            self.send_to_scheduler.send_pyobj(load_update_req)
```
**EN:** This block implements the async method `_handle_batch_output(recv_obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_handle_batch_output`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_handle_batch_output(recv_obj)`。它围绕 `_handle_batch_output` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1912-1947: Implement add logprob to meta info / 实现add logprob to meta info
```python
    def add_logprob_to_meta_info(
        self,
        meta_info: dict,
        state: ReqState,
        top_logprobs_num: int,
        token_ids_logprob: List[int],
        return_text_in_logprobs: bool,
    ):
        # 1. Handle regular logprobs
        if len(state.input_token_logprobs_val) > len(state.input_token_logprobs):
            state.input_token_logprobs.extend(
                self.detokenize_logprob_tokens(
                    state.input_token_logprobs_val[len(state.input_token_logprobs) :],
                    state.input_token_logprobs_idx[len(state.input_token_logprobs) :],
                    return_text_in_logprobs,
                )
            )

        if len(state.output_token_logprobs_val) > len(state.output_token_logprobs):
            state.output_token_logprobs.extend(
                self.detokenize_logprob_tokens(
                    state.output_token_logprobs_val[len(state.output_token_logprobs) :],
                    state.output_token_logprobs_idx[len(state.output_token_logprobs) :],
                    return_text_in_logprobs,
                )
            )

        meta_info["input_token_logprobs"] = state.input_token_logprobs
        meta_info["output_token_logprobs"] = state.output_token_logprobs
        meta_info["output_token_logprobs_length"] = len(state.output_token_logprobs)

        # 2. Handle top logprobs
        if top_logprobs_num > 0:
            if len(state.input_top_logprobs_val) > len(state.input_top_logprobs):
                state.input_top_logprobs.extend(
                    self.detokenize_top_logprobs_tokens(
```
**EN:** This block implements the method `add_logprob_to_meta_info(meta_info, state, top_logprobs_num, token_ids_logprob, return_text_in_logprobs)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `add_logprob_to_meta_info`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `add_logprob_to_meta_info(meta_info, state, top_logprobs_num, token_ids_logprob, return_text_in_logprobs)`。它围绕 `add_logprob_to_meta_info` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1948-1983: Continue add logprob to meta info / 继续说明add logprob to meta info
```python
                        state.input_top_logprobs_val[len(state.input_top_logprobs) :],
                        state.input_top_logprobs_idx[len(state.input_top_logprobs) :],
                        return_text_in_logprobs,
                    )
                )
            if len(state.output_top_logprobs_val) > len(state.output_top_logprobs):
                state.output_top_logprobs.extend(
                    self.detokenize_top_logprobs_tokens(
                        state.output_top_logprobs_val[len(state.output_top_logprobs) :],
                        state.output_top_logprobs_idx[len(state.output_top_logprobs) :],
                        return_text_in_logprobs,
                    )
                )

            meta_info["input_top_logprobs"] = state.input_top_logprobs
            meta_info["output_top_logprobs"] = state.output_top_logprobs

        # 3. Handle token_ids_logprob
        if token_ids_logprob is not None:
            if len(state.input_token_ids_logprobs_val) > len(
                state.input_token_ids_logprobs
            ):
                state.input_token_ids_logprobs.extend(
                    self.detokenize_top_logprobs_tokens(
                        state.input_token_ids_logprobs_val[
                            len(state.input_token_ids_logprobs) :
                        ],
                        state.input_token_ids_logprobs_idx[
                            len(state.input_token_ids_logprobs) :
                        ],
                        return_text_in_logprobs,
                    )
                )
            if len(state.output_token_ids_logprobs_val) > len(
                state.output_token_ids_logprobs
            ):
```
**EN:** This block implements the method `add_logprob_to_meta_info(meta_info, state, top_logprobs_num, token_ids_logprob, return_text_in_logprobs)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `add_logprob_to_meta_info`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `add_logprob_to_meta_info(meta_info, state, top_logprobs_num, token_ids_logprob, return_text_in_logprobs)`。它围绕 `add_logprob_to_meta_info` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1984-1997: Continue add logprob to meta info / 继续说明add logprob to meta info
```python
                state.output_token_ids_logprobs.extend(
                    self.detokenize_top_logprobs_tokens(
                        state.output_token_ids_logprobs_val[
                            len(state.output_token_ids_logprobs) :
                        ],
                        state.output_token_ids_logprobs_idx[
                            len(state.output_token_ids_logprobs) :
                        ],
                        return_text_in_logprobs,
                    )
                )

            meta_info["input_token_ids_logprobs"] = state.input_token_ids_logprobs
            meta_info["output_token_ids_logprobs"] = state.output_token_ids_logprobs
```
**EN:** This block implements the method `add_logprob_to_meta_info(meta_info, state, top_logprobs_num, token_ids_logprob, return_text_in_logprobs)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `add_logprob_to_meta_info`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `add_logprob_to_meta_info(meta_info, state, top_logprobs_num, token_ids_logprob, return_text_in_logprobs)`。它围绕 `add_logprob_to_meta_info` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 1999-2034: Implement convert logprob style / 实现convert logprob style
```python
    def convert_logprob_style(
        self,
        meta_info: dict,
        state: ReqState,
        top_logprobs_num: int,
        token_ids_logprob: List[int],
        return_text_in_logprobs: bool,
        recv_obj: BatchStrOutput,
        recv_obj_index: int,
    ):
        if recv_obj.input_token_logprobs_val is None:
            return

        if (
            len(recv_obj.input_token_logprobs_val) > 0
            and recv_obj.input_token_logprobs_val[recv_obj_index] is not None
        ):
            state.input_token_logprobs_val.extend(
                recv_obj.input_token_logprobs_val[recv_obj_index]
            )
            state.input_token_logprobs_idx.extend(
                recv_obj.input_token_logprobs_idx[recv_obj_index]
            )
        state.output_token_logprobs_val.extend(
            recv_obj.output_token_logprobs_val[recv_obj_index]
        )
        state.output_token_logprobs_idx.extend(
            recv_obj.output_token_logprobs_idx[recv_obj_index]
        )

        if top_logprobs_num > 0:
            if len(recv_obj.input_top_logprobs_val) > 0:
                state.input_top_logprobs_val.extend(
                    recv_obj.input_top_logprobs_val[recv_obj_index]
                )
                state.input_top_logprobs_idx.extend(
```
**EN:** This block implements the method `convert_logprob_style(meta_info, state, top_logprobs_num, token_ids_logprob, return_text_in_logprobs, ...)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `convert_logprob_style`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `convert_logprob_style(meta_info, state, top_logprobs_num, token_ids_logprob, return_text_in_logprobs, ...)`。它围绕 `convert_logprob_style` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2035-2065: Continue convert logprob style / 继续说明convert logprob style
```python
                    recv_obj.input_top_logprobs_idx[recv_obj_index]
                )
            state.output_top_logprobs_val.extend(
                recv_obj.output_top_logprobs_val[recv_obj_index]
            )
            state.output_top_logprobs_idx.extend(
                recv_obj.output_top_logprobs_idx[recv_obj_index]
            )

        if token_ids_logprob is not None:
            if len(recv_obj.input_token_ids_logprobs_val) > 0:
                state.input_token_ids_logprobs_val.extend(
                    recv_obj.input_token_ids_logprobs_val[recv_obj_index]
                )
                state.input_token_ids_logprobs_idx.extend(
                    recv_obj.input_token_ids_logprobs_idx[recv_obj_index]
                )
            state.output_token_ids_logprobs_val.extend(
                recv_obj.output_token_ids_logprobs_val[recv_obj_index]
            )
            state.output_token_ids_logprobs_idx.extend(
                recv_obj.output_token_ids_logprobs_idx[recv_obj_index]
            )

        self.add_logprob_to_meta_info(
            meta_info,
            state,
            state.obj.top_logprobs_num,
            state.obj.token_ids_logprob,
            return_text_in_logprobs,
        )
```
**EN:** This block implements the method `convert_logprob_style(meta_info, state, top_logprobs_num, token_ids_logprob, return_text_in_logprobs, ...)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `convert_logprob_style`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `convert_logprob_style(meta_info, state, top_logprobs_num, token_ids_logprob, return_text_in_logprobs, ...)`。它围绕 `convert_logprob_style` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2067-2085: Implement detokenize logprob tokens / 实现detokenize logprob tokens
```python
    def detokenize_logprob_tokens(
        self,
        token_logprobs_val: List[float],
        token_logprobs_idx: List[int],
        decode_to_text: bool,
    ):
        if not decode_to_text:
            return [
                (logprob, token_id, None)
                for logprob, token_id in zip(token_logprobs_val, token_logprobs_idx)
            ]
        else:
            assert self.tokenizer is not None
            # In transformers v5, batch_decode([1, 2, 3]) concatenates all tokens
            # into one string. Wrap each ID in its own list so they decode separately.
            token_texts = self.tokenizer.batch_decode(
                [[idx] for idx in token_logprobs_idx]
            )
            return list(zip(token_logprobs_val, token_logprobs_idx, token_texts))
```
**EN:** This block implements the method `detokenize_logprob_tokens(token_logprobs_val, token_logprobs_idx, decode_to_text)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `detokenize_logprob_tokens`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `detokenize_logprob_tokens(token_logprobs_val, token_logprobs_idx, decode_to_text)`。它围绕 `detokenize_logprob_tokens` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2087-2105: Implement detokenize top logprobs tokens / 实现detokenize top logprobs tokens
```python
    def detokenize_top_logprobs_tokens(
        self,
        token_logprobs_val: List[float],
        token_logprobs_idx: List[int],
        decode_to_text: bool,
    ):
        # TODO: The current implementation only batches the detokenization for top-k tokens per single position.
        # We should batch all top-k tokens in all positions.
        ret = []
        for i in range(len(token_logprobs_val)):
            if token_logprobs_val[i]:
                ret.append(
                    self.detokenize_logprob_tokens(
                        token_logprobs_val[i], token_logprobs_idx[i], decode_to_text
                    )
                )
            else:
                ret.append(None)
        return ret
```
**EN:** This block implements the method `detokenize_top_logprobs_tokens(token_logprobs_val, token_logprobs_idx, decode_to_text)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `detokenize_top_logprobs_tokens`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `detokenize_top_logprobs_tokens(token_logprobs_val, token_logprobs_idx, decode_to_text)`。它围绕 `detokenize_top_logprobs_tokens` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2107-2142: Implement calculate spec decoding metrics / 实现calculate spec decoding 指标
```python
    def _calculate_spec_decoding_metrics(
        self,
        meta_info: Dict[str, Any],
        recv_obj: Union[
            BatchStrOutput,
            BatchEmbeddingOutput,
            BatchTokenIDOutput,
        ],
        i: int,
    ) -> None:
        """Calculate speculative decoding metrics, such as acceptance rate and acceptance length metrics."""
        if (
            hasattr(recv_obj, "spec_verify_ct")
            and recv_obj.spec_verify_ct[i] > 0
            and hasattr(recv_obj, "spec_num_correct_drafts")
            and len(recv_obj.spec_num_correct_drafts) > i
        ):
            # Total number of proposed draft tokens per request.
            num_proposed_drafts = recv_obj.spec_verify_ct[i] * (
                self.server_args.speculative_num_draft_tokens - 1
            )
            num_correct_drafts = recv_obj.spec_num_correct_drafts[i]

            # Calculate per-request acceptance rate and average acceptance length.
            if num_proposed_drafts > 0:
                # accept_rate: num_correct_drafts / num_proposed_drafts (strict count, no bonus).
                meta_info["spec_accept_rate"] = num_correct_drafts / num_proposed_drafts
                # accept_length: completion_tokens / verify_ct (includes bonus token).
                meta_info["spec_accept_length"] = (
                    recv_obj.completion_tokens[i] / recv_obj.spec_verify_ct[i]
                )

                meta_info["spec_num_correct_drafts"] = num_correct_drafts
                meta_info["spec_num_proposed_drafts"] = num_proposed_drafts
                meta_info["spec_verify_ct"] = recv_obj.spec_verify_ct[i]
```
**EN:** This block implements the method `_calculate_spec_decoding_metrics(meta_info, recv_obj, i)` on `TokenizerManager`. It focuses on Calculate speculative decoding metrics, such as acceptance rate and acceptance length metrics., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_calculate_spec_decoding_metrics(meta_info, recv_obj, i)`。它围绕 `_calculate_spec_decoding_metrics` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2143-2159: Continue calculate spec decoding metrics / 继续说明calculate spec decoding 指标
```python
                # FIXME: backward-compat aliases, remove in next release.
                meta_info["spec_accepted_drafts"] = num_correct_drafts
                meta_info["spec_proposed_drafts"] = num_proposed_drafts

            # Acceptance histogram: tracks how many decoding steps accepted a certain number of draft tokens.
            if (
                recv_obj.spec_correct_drafts_histogram
                and len(recv_obj.spec_correct_drafts_histogram) > i
                and recv_obj.spec_correct_drafts_histogram[i]
            ):
                meta_info["spec_correct_drafts_histogram"] = (
                    recv_obj.spec_correct_drafts_histogram[i]
                )
                # FIXME: backward-compat alias, remove in next release.
                meta_info["spec_accept_histogram"] = (
                    recv_obj.spec_correct_drafts_histogram[i]
                )
```
**EN:** This block implements the method `_calculate_spec_decoding_metrics(meta_info, recv_obj, i)` on `TokenizerManager`. It focuses on Calculate speculative decoding metrics, such as acceptance rate and acceptance length metrics., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_calculate_spec_decoding_metrics(meta_info, recv_obj, i)`。它围绕 `_calculate_spec_decoding_metrics` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2161-2167: Implement request has grammar / 实现请求 has grammar
```python
    def _request_has_grammar(self, obj: GenerateReqInput) -> bool:
        return (
            obj.sampling_params.get("json_schema", None)
            or obj.sampling_params.get("regex", None)
            or obj.sampling_params.get("ebnf", None)
            or obj.sampling_params.get("structural_tag", None)
        )
```
**EN:** This block implements the method `_request_has_grammar(obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_request_has_grammar`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_request_has_grammar(obj)`。它围绕 `_request_has_grammar` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2169-2204: Implement collect metrics / 实现collect 指标
```python
    def collect_metrics(self, state: ReqState, recv_obj: BatchStrOutput, i: int):
        completion_tokens = (
            recv_obj.completion_tokens[i]
            if getattr(recv_obj, "completion_tokens", None)
            else 0
        )

        custom_labels = getattr(state.obj, "custom_labels", None)
        labels = dict(self.metrics_collector.labels)
        if custom_labels:
            labels.update(custom_labels)
        if self.enable_priority_scheduling:
            priority = getattr(state.obj, "priority", None)
            if priority is not None:
                labels["priority"] = str(priority)
        if (
            not state.ttft_observed
            and self.disaggregation_mode != DisaggregationMode.PREFILL
        ):
            state.ttft_observed = True
            state.last_completion_tokens = completion_tokens
            self.metrics_collector.observe_time_to_first_token(
                labels, state.time_stats.get_first_token_latency()
            )
        else:
            num_new_tokens = completion_tokens - state.last_completion_tokens
            if num_new_tokens:
                self.metrics_collector.observe_inter_token_latency(
                    labels,
                    state.time_stats.get_interval(),
                    num_new_tokens,
                )
                state.time_stats.set_last_time()
                state.last_completion_tokens = completion_tokens

        if state.finished:
```
**EN:** This block implements the method `collect_metrics(state, recv_obj, i)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `collect_metrics`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `collect_metrics(state, recv_obj, i)`。它围绕 `collect_metrics` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2205-2221: Continue collect metrics / 继续说明collect 指标
```python
            # Get detailed cache breakdown if available
            cached_tokens_details = None
            if (
                hasattr(recv_obj, "cached_tokens_details")
                and recv_obj.cached_tokens_details
            ):
                cached_tokens_details = recv_obj.cached_tokens_details[i]

            self.metrics_collector.observe_one_finished_request(
                labels,
                recv_obj.prompt_tokens[i],
                completion_tokens,
                recv_obj.cached_tokens[i],
                state.time_stats.get_e2e_latency(),
                self._request_has_grammar(state.obj),
                cached_tokens_details,
            )
```
**EN:** This block implements the method `collect_metrics(state, recv_obj, i)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `collect_metrics`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `collect_metrics(state, recv_obj, i)`。它围绕 `collect_metrics` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2223-2253: Implement dump requests / 实现dump 请求
```python
    def dump_requests(self, state: ReqState, out_dict: dict):
        if self.dump_requests_exclude_meta_keys and isinstance(
            out_dict.get("meta_info"), dict
        ):
            exclude = self.dump_requests_exclude_meta_keys
            if any(k in out_dict["meta_info"] for k in exclude):
                filtered_meta = {
                    k: v for k, v in out_dict["meta_info"].items() if k not in exclude
                }
                out_dict = {**out_dict, "meta_info": filtered_meta}

        self.dump_request_list.append(
            (
                state.obj,
                out_dict,
                convert_time_to_realtime(state.time_stats.created_time),
                convert_time_to_realtime(state.time_stats.finished_time),
            )
        )

        if len(self.dump_request_list) >= self.dump_requests_threshold:
            filename = os.path.join(
                self.dump_requests_folder,
                datetime.now().strftime("%Y-%m-%d_%H-%M-%S") + ".pkl",
            )
            self._dump_data_to_file(
                data_list=self.dump_request_list,
                filename=filename,
                log_message=f"Dump {len(self.dump_request_list)} requests to {filename}",
            )
            self.dump_request_list = []
```
**EN:** This block implements the method `dump_requests(state, out_dict)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `dump_requests`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `dump_requests(state, out_dict)`。它围绕 `dump_requests` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2255-2270: Implement record request for crash dump / 实现record 请求 for crash dump
```python
    def record_request_for_crash_dump(self, state: ReqState, out_dict: dict):
        current_time = real_time()
        self.crash_dump_request_list.append(
            (
                state.obj,
                out_dict,
                convert_time_to_realtime(state.time_stats.created_time),
                current_time,
            )
        )
        # Remove requests older than 5 minutes based on finish time
        while (
            self.crash_dump_request_list
            and current_time - self.crash_dump_request_list[0][3] >= 300
        ):
            self.crash_dump_request_list.popleft()
```
**EN:** This block implements the method `record_request_for_crash_dump(state, out_dict)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `record_request_for_crash_dump`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `record_request_for_crash_dump(state, out_dict)`。它围绕 `record_request_for_crash_dump` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2272-2299: Implement dump data to file / 实现dump 数据 to file
```python
    def _dump_data_to_file(
        self, data_list: List[Tuple], filename: str, log_message: str
    ):
        logger.info(log_message)
        to_dump_with_server_args = {
            "server_args": self.server_args,
            "requests": data_list.copy(),
        }

        def background_task():
            os.makedirs(os.path.dirname(filename), exist_ok=True)
            with open(filename, "wb") as f:
                try:
                    pickle.dump(to_dump_with_server_args, f)
                except Exception as e:
                    # When the server is launched with --trust-remote-code,
                    # server_args sometimes fails to pickle. Retry without
                    # server_args so the request data still gets persisted.
                    logger.error(
                        f"Failed to pickle dump with server_args: {e!r}; "
                        "retrying without server_args"
                    )
                    f.seek(0)
                    f.truncate()
                    to_dump_with_server_args["server_args"] = None
                    pickle.dump(to_dump_with_server_args, f)

        asyncio.create_task(asyncio.to_thread(background_task))
```
**EN:** This block implements the method `_dump_data_to_file(data_list, filename, log_message)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_dump_data_to_file`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_dump_data_to_file(data_list, filename, log_message)`。它围绕 `_dump_data_to_file` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2301-2336: Implement dump requests before crash / 实现dump 请求 before crash
```python
    def dump_requests_before_crash(
        self, hostname: str = os.getenv("HOSTNAME", socket.gethostname())
    ):
        if not self.crash_dump_folder:
            return

        if self.crash_dump_performed:
            logger.info(
                "SIGTERM/SIGQUIT/Exception triggered, but crash dump already performed, skipping."
            )
            return
        else:
            self.crash_dump_performed = True

        logger.error(f"Dumping requests before crash. {self.crash_dump_folder=}")

        # Add finished requests from crash_dump_request_list
        data_to_dump = []
        if self.crash_dump_request_list:
            data_to_dump.extend(self.crash_dump_request_list)

        # Add unfinished requests from rid_to_state
        unfinished_requests = []
        for rid, state in self.rid_to_state.items():
            if not state.finished:
                state.time_stats.set_finished_time()
                unfinished_requests.append(
                    (
                        state.obj,
                        (
                            state.out_list[-1]
                            if state.out_list
                            else state.get_crash_dump_output()
                        ),
                        convert_time_to_realtime(state.time_stats.created_time),
                        convert_time_to_realtime(state.time_stats.finished_time),
```
**EN:** This block implements the method `dump_requests_before_crash(hostname)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `dump_requests_before_crash`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `dump_requests_before_crash(hostname)`。它围绕 `dump_requests_before_crash` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2337-2372: Continue dump requests before crash / 继续说明dump 请求 before crash
```python
                    )
                )
        if unfinished_requests:
            data_to_dump.extend(unfinished_requests)

        if not data_to_dump:
            return

        # Create a file
        filename = os.path.join(
            self.crash_dump_folder,
            hostname,
            f'crash_dump_{datetime.now().strftime("%Y-%m-%d_%H-%M-%S")}.pkl',
        )
        os.makedirs(os.path.dirname(filename), exist_ok=True)

        # Write the data to the file
        data_to_dump_with_server_args = {
            "server_args": self.server_args,  # Include server_args in the dump
            "requests": data_to_dump,
            "launch_command": " ".join(sys.argv),
        }
        with open(filename, "wb") as f:
            try:
                pickle.dump(data_to_dump_with_server_args, f)
            except Exception as e:
                # When the server is launched with --trust-remote-code,
                # server_args sometimes fails to pickle. Retry without
                # server_args so the request data still gets persisted.
                logger.error(
                    f"Failed to pickle dump with server_args: {e!r}; "
                    "retrying without server_args"
                )
                f.seek(0)
                f.truncate()
                data_to_dump_with_server_args["server_args"] = None
```
**EN:** This block implements the method `dump_requests_before_crash(hostname)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `dump_requests_before_crash`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `dump_requests_before_crash(hostname)`。它围绕 `dump_requests_before_crash` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2373-2377: Continue dump requests before crash / 继续说明dump 请求 before crash
```python
                pickle.dump(data_to_dump_with_server_args, f)
        logger.error(
            f"Dumped {len(self.crash_dump_request_list)} finished and {len(unfinished_requests)} unfinished requests before crash to {filename}"
        )
        return filename
```
**EN:** This block implements the method `dump_requests_before_crash(hostname)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `dump_requests_before_crash`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `dump_requests_before_crash(hostname)`。它围绕 `dump_requests_before_crash` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2379-2415: Implement async sigterm watchdog / 实现异步sigterm watchdog
```python
    async def sigterm_watchdog(self):
        while not self.gracefully_exit:
            await asyncio.sleep(5)

        # Drain requests
        while True:
            remain_num_req = len(self.rid_to_state)
            remaining_rids = list(self.rid_to_state.keys())

            if self.server_status == ServerStatus.UnHealthy:
                # if health check failed, we should exit immediately
                logger.error(
                    "Signal SIGTERM received while health check failed. Force exiting."
                )
                self.dump_requests_before_crash()
                self.force_exit_handler()
                break

            elif get_bool_env_var("SGL_FORCE_SHUTDOWN"):
                # if force shutdown flag set, exit immediately
                logger.error(
                    "Signal SIGTERM received while force shutdown flag set. Force exiting."
                )
                self.force_exit_handler()
                break

            logger.info(
                f"Gracefully exiting... Remaining number of requests {remain_num_req}. Remaining requests {remaining_rids=}."
            )
            if remain_num_req > 0:
                await asyncio.sleep(5)
            else:
                self.dump_requests_before_crash()
                break

        kill_process_tree(os.getpid(), include_parent=True)
        sys.exit(0)
```
**EN:** This block implements the async method `sigterm_watchdog()` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `sigterm_watchdog`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `sigterm_watchdog()`。它围绕 `sigterm_watchdog` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2417-2419: Implement force exit handler / 实现force exit handler
```python
    def force_exit_handler(self):
        """Put some custom force exit logic here."""
        pass
```
**EN:** This block implements the method `force_exit_handler()` on `TokenizerManager`. It focuses on Put some custom force exit logic here., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `force_exit_handler()`。它围绕 `force_exit_handler` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2421-2462: Implement handle abort req / 实现handle abort req
```python
    def _handle_abort_req(self, recv_obj: AbortReq):
        if is_health_check_generate_req(recv_obj):
            return
        state = self.rid_to_state[recv_obj.rid]
        state.finished = True
        state.time_stats.set_finished_time()

        abort_message = recv_obj.abort_message or "Abort in waiting queue"
        finish_reason = {
            "type": "abort",
            "message": abort_message,
        }
        if recv_obj.finished_reason:
            finish_reason = recv_obj.finished_reason
        meta_info = {
            "id": recv_obj.rid,
            "finish_reason": finish_reason,
            "weight_version": self.server_args.weight_version,
            "e2e_latency": state.time_stats.get_e2e_latency(),
        }
        is_stream = getattr(state.obj, "stream", False)
        if getattr(state.obj, "return_logprob", False):
            self.add_logprob_to_meta_info(
                meta_info,
                state,
                state.obj.top_logprobs_num,
                state.obj.token_ids_logprob,
                state.obj.return_text_in_logprobs
                and not self.server_args.skip_tokenizer_init,
            )

        output_ids = state.output_ids
        meta_info["completion_tokens"] = len(output_ids)
        if is_stream:
            output_ids = [output_ids[-1]] if len(output_ids) > 0 else []
        out = {
            "text": state.get_text(),
            "output_ids": output_ids,
            "meta_info": meta_info,
        }
        state.out_list.append(out)
        state.event.set()
```
**EN:** This block implements the method `_handle_abort_req(recv_obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_handle_abort_req`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_handle_abort_req(recv_obj)`。它围绕 `_handle_abort_req` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2464-2465: Implement update active ranks / 实现update active ranks
```python
    def update_active_ranks(self, ranks: ActiveRanksOutput):
        self.send_to_scheduler.send_pyobj(ranks)
```
**EN:** This block implements the method `update_active_ranks(ranks)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `update_active_ranks`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `update_active_ranks(ranks)`。它围绕 `update_active_ranks` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2467-2476: Implement handle open session req output / 实现handle open 会话 req output
```python
    def _handle_open_session_req_output(self, recv_obj):
        future = self.session_futures.get(recv_obj.session_id)
        if future is None:
            logger.warning(
                "Open session response arrived after waiter cleanup: %s",
                recv_obj.session_id,
            )
            return
        if not future.done():
            future.set_result(recv_obj.session_id if recv_obj.success else None)
```
**EN:** This block implements the method `_handle_open_session_req_output(recv_obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_handle_open_session_req_output`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_handle_open_session_req_output(recv_obj)`。它围绕 `_handle_open_session_req_output` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2478-2485: Implement handle update weights from disk req output / 实现handle update weights from disk req output
```python
    def _handle_update_weights_from_disk_req_output(self, recv_obj):
        if self.server_args.dp_size == 1:
            self.model_update_result.set_result(recv_obj)
        else:  # self.server_args.dp_size > 1
            self.model_update_tmp.append(recv_obj)
            # set future if the all results are received
            if len(self.model_update_tmp) == self.server_args.dp_size:
                self.model_update_result.set_result(self.model_update_tmp)
```
**EN:** This block implements the method `_handle_update_weights_from_disk_req_output(recv_obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_handle_update_weights_from_disk_req_output`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_handle_update_weights_from_disk_req_output(recv_obj)`。它围绕 `_handle_update_weights_from_disk_req_output` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2487-2506: Implement async validate and resolve lora / 实现异步validate and resolve lora
```python
    async def _validate_and_resolve_lora(
        self, obj: Union[GenerateReqInput, EmbeddingReqInput]
    ) -> None:
        if not obj.lora_path:
            return

        if not self.server_args.enable_lora:
            first_adapter = (
                obj.lora_path
                if isinstance(obj.lora_path, str)
                else next((a for a in obj.lora_path if a), None)
            )

            raise ValueError(
                f"LoRA adapter '{first_adapter}' was requested, but LoRA is not enabled. "
                "Please launch the server with --enable-lora flag and preload adapters "
                "using --lora-paths or /load_lora_adapter endpoint."
            )

        await self._resolve_lora_path(obj)
```
**EN:** This block implements the async method `_validate_and_resolve_lora(obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_validate_and_resolve_lora`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_validate_and_resolve_lora(obj)`。它围绕 `_validate_and_resolve_lora` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2508-2543: Implement async resolve lora path / 实现异步resolve lora path
```python
    async def _resolve_lora_path(self, obj: Union[GenerateReqInput, EmbeddingReqInput]):
        if isinstance(obj.lora_path, str):
            unique_lora_paths = set([obj.lora_path])
        else:
            unique_lora_paths = set(obj.lora_path)

        if (
            self.server_args.max_loaded_loras is not None
            and len(unique_lora_paths) > self.server_args.max_loaded_loras
        ):
            raise ValueError(
                f"Received request with {len(unique_lora_paths)} unique loras requested "
                f"but max loaded loras is {self.server_args.max_loaded_loras}"
            )

        # Reload all existing LoRA adapters that have been dynamically unloaded
        unregistered_loras = await self.lora_registry.get_unregistered_loras(
            unique_lora_paths
        )
        for lora_path in unregistered_loras:
            if lora_path is None:
                continue

            if lora_path not in self.lora_ref_cache:
                raise ValueError(
                    f"Got LoRA adapter that has never been loaded: {lora_path}\n"
                    f"All loaded adapters: {self.lora_ref_cache.keys()}."
                )

            logger.info(f"Reloading evicted adapter: {lora_path}")
            new_lora_ref = self.lora_ref_cache[lora_path]
            load_result = await self.load_lora_adapter(
                LoadLoRAAdapterReqInput(
                    lora_name=new_lora_ref.lora_name,
                    lora_path=new_lora_ref.lora_path,
                    pinned=new_lora_ref.pinned,
```
**EN:** This block implements the async method `_resolve_lora_path(obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_resolve_lora_path`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_resolve_lora_path(obj)`。它围绕 `_resolve_lora_path` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2544-2560: Continue async resolve lora path / 继续说明异步resolve lora path
```python
                )
            )
            if (
                not load_result.success
                and "already loaded" not in load_result.error_message
            ):
                raise ValueError(
                    f"Failed to implicitly load LoRA adapter {lora_path}: {load_result.error_message}"
                )

        # Look up the LoRA ID from the registry and start tracking ongoing LoRA requests.
        obj.lora_id = await self.lora_registry.acquire(obj.lora_path)
        # Propagate lora_id to any sub-objects already cached by __getitem__.
        for i, sub_obj in obj.__dict__.get("_sub_obj_cache", {}).items():
            sub_obj.lora_id = (
                obj.lora_id[i] if isinstance(obj.lora_id, list) else obj.lora_id
            )
```
**EN:** This block implements the async method `_resolve_lora_path(obj)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_resolve_lora_path`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的异步方法 `_resolve_lora_path(obj)`。它围绕 `_resolve_lora_path` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2562-2605: Implement init req state / 实现init req 状态
```python
    def _init_req_state(
        self,
        obj: Union[GenerateReqInput, EmbeddingReqInput],
        request: Optional[fastapi.Request] = None,
    ):
        created_time = obj.received_time

        external_trace_header = None
        if self.server_args.enable_trace:
            if obj.external_trace_header:
                # When the request comes from the rust grpc server or Engine there isn't a
                # real request object but we still need to propagate the trace context from
                # the trace context that is explicitly passed in
                external_trace_header = obj.external_trace_header
            elif request:
                external_trace_header = extract_trace_headers(request.headers)
                obj.external_trace_header = external_trace_header

        # Normalize single/batch into a uniform list of (rid, sub_obj, bootstrap_room)
        if not hasattr(obj, "is_single") or obj.is_single:
            items = [(obj.rid, obj, getattr(obj, "bootstrap_room", None))]
        else:
            items = [
                (
                    obj.rid[i],
                    obj[i],
                    (
                        obj.bootstrap_room[i]
                        if hasattr(obj, "bootstrap_room") and obj.bootstrap_room
                        else None
                    ),
                )
                for i in range(len(obj.rid))
            ]

        for rid, sub_obj, bootstrap_room in items:
            if rid in self.rid_to_state:
                raise ValueError(f"Duplicate request ID detected: {rid}")
            time_stats = APIServerReqTimeStats(disagg_mode=self.disaggregation_mode)
            state = ReqState([], False, asyncio.Event(), sub_obj, time_stats)
            self.rid_to_state[rid] = state
            if self.server_args.enable_trace:
                time_stats.init_trace_ctx(rid, bootstrap_room, external_trace_header)
            time_stats.set_created_time(created_time)
```
**EN:** This block implements the method `_init_req_state(obj, request)` on `TokenizerManager`. It focuses on handling the tokenizer manager responsibilities represented by `_init_req_state`, so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_init_req_state(obj, request)`。它围绕 `_init_req_state` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2607-2641: Implement should dispatch to encoder / 实现should dispatch to encoder
```python
    def _should_dispatch_to_encoder(
        self, obj: Union[GenerateReqInput, EmbeddingReqInput]
    ) -> bool:
        """Check if the request should be dispatched to encoder for processing.

        Returns True if the request should be dispatched to encoder (multiple multimodal items),
        False if it should be processed locally (single multimodal item or no multimodal items).

        Args:
            obj: The request input object

        Returns:
            bool: True if should dispatch to encoder, False otherwise
        """
        if obj.batch_size > 1:
            logger.warning(
                "Batch request (batch_size=%d) is not supported in EPD disaggregation mode; skipping encoder dispatch.",
                obj.batch_size,
            )
            return False
        if not isinstance(obj, GenerateReqInput) or not obj.contains_mm_input():
            return False

        # Count image / video / audio items for dispatch threshold
        def _count_mm_items(data):
            return (
                len(data) if isinstance(data, list) else (1 if data is not None else 0)
            )

        total_mm_items = (
            _count_mm_items(getattr(obj, "image_data", None))
            + _count_mm_items(getattr(obj, "video_data", None))
            + _count_mm_items(getattr(obj, "audio_data", None))
        )
        return total_mm_items >= envs.SGLANG_ENCODER_DISPATCH_MIN_ITEMS.get()
```
**EN:** This block implements the method `_should_dispatch_to_encoder(obj)` on `TokenizerManager`. It focuses on Check if the request should be dispatched to encoder for processing., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_should_dispatch_to_encoder(obj)`。它围绕 `_should_dispatch_to_encoder` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2643-2660: Implement handle epd disaggregation encode request / 实现handle epd disaggregation encode 请求
```python
    def _handle_epd_disaggregation_encode_request(
        self, obj: Union[GenerateReqInput, EmbeddingReqInput]
    ):
        """Handle EPD-disaggregation mode encoding request."""
        if isinstance(obj, GenerateReqInput) and obj.contains_mm_input():
            # dispatch to encoder by default
            should_dispatch = True
            if self.server_args.enable_adaptive_dispatch_to_encoder:
                should_dispatch = self._should_dispatch_to_encoder(obj)

            # Set need_wait_for_mm_inputs flag based on whether we dispatch to encoder
            # This flag will be used in _tokenize_one_request to determine processing path
            if should_dispatch:
                obj.need_wait_for_mm_inputs = True
                if self.server_args.encoder_transfer_backend == "zmq_to_scheduler":
                    self.mm_receiver.send_encode_request(obj)
            else:
                obj.need_wait_for_mm_inputs = False
```
**EN:** This block implements the method `_handle_epd_disaggregation_encode_request(obj)` on `TokenizerManager`. It focuses on Handle EPD-disaggregation mode encoding request., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_handle_epd_disaggregation_encode_request(obj)`。它围绕 `_handle_epd_disaggregation_encode_request` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2662-2697: Implement convert to span attrs / 实现convert to span attrs
```python
    def convert_to_span_attrs(
        self,
        state: ReqState,
        recv_obj: Union[
            BatchStrOutput,
            BatchEmbeddingOutput,
            BatchTokenIDOutput,
        ],
        i: int,
    ) -> Dict[str, Any]:
        """Convert attributes to span attributes."""
        span_attrs = {}

        if not self.server_args.enable_trace:
            return span_attrs

        # Token usage attributes
        if not isinstance(recv_obj, BatchEmbeddingOutput):
            span_attrs[SpanAttributes.GEN_AI_USAGE_COMPLETION_TOKENS] = (
                recv_obj.completion_tokens[i]
            )
        span_attrs[SpanAttributes.GEN_AI_USAGE_PROMPT_TOKENS] = recv_obj.prompt_tokens[
            i
        ]
        span_attrs[SpanAttributes.GEN_AI_USAGE_CACHED_TOKENS] = recv_obj.cached_tokens[
            i
        ]

        # Request identifiers
        span_attrs[SpanAttributes.GEN_AI_REQUEST_ID] = (
            str(state.obj.rid) if state.obj.rid else None
        )

        # Sampling parameters
        sampling_params = state.obj.sampling_params or {}
```
**EN:** This block implements the method `convert_to_span_attrs(state, recv_obj, i)` on `TokenizerManager`. It focuses on Convert attributes to span attributes., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `convert_to_span_attrs(state, recv_obj, i)`。它围绕 `convert_to_span_attrs` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2698-2729: Continue convert to span attrs / 继续说明convert to span attrs
```python
        if max_new_tokens := sampling_params.get("max_new_tokens"):
            span_attrs[SpanAttributes.GEN_AI_REQUEST_MAX_TOKENS] = max_new_tokens

        if top_p := sampling_params.get("top_p"):
            span_attrs[SpanAttributes.GEN_AI_REQUEST_TOP_P] = top_p

        if temperature := sampling_params.get("temperature"):
            span_attrs[SpanAttributes.GEN_AI_REQUEST_TEMPERATURE] = temperature

        if top_k := sampling_params.get("top_k"):
            span_attrs[SpanAttributes.GEN_AI_REQUEST_TOP_K] = top_k

        if n := sampling_params.get("n"):
            span_attrs[SpanAttributes.GEN_AI_REQUEST_N] = n

        # Response attributes
        span_attrs[SpanAttributes.GEN_AI_RESPONSE_MODEL] = self.served_model_name

        finish_reason = (
            recv_obj.finished_reasons[i].get("type")
            if recv_obj.finished_reasons[i]
            else None
        )
        if finish_reason:
            span_attrs[SpanAttributes.GEN_AI_RESPONSE_FINISH_REASONS] = json.dumps(
                [finish_reason]
            )

        # Latency attributes
        span_attrs.update(state.time_stats.convert_to_gen_ai_span_attrs())

        return span_attrs
```
**EN:** This block implements the method `convert_to_span_attrs(state, recv_obj, i)` on `TokenizerManager`. It focuses on Convert attributes to span attributes., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `convert_to_span_attrs(state, recv_obj, i)`。它围绕 `convert_to_span_attrs` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2731-2738: Implement set default priority / 实现set default priority
```python
    def _set_default_priority(self, obj: Union[GenerateReqInput, EmbeddingReqInput]):
        """Set the default priority value."""
        if (
            self.enable_priority_scheduling
            and obj.priority is None
            and self.default_priority_value is not None
        ):
            obj.priority = self.default_priority_value
```
**EN:** This block implements the method `_set_default_priority(obj)` on `TokenizerManager`. It focuses on Set the default priority value., so the class can advance the tokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `TokenizerManager` 上的方法 `_set_default_priority(obj)`。它围绕 `_set_default_priority` 所承担的 分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 2741-2744: Define class ServerStatus / 定义类 ServerStatus
```python
class ServerStatus(Enum):
    Up = "Up"
    Starting = "Starting"
    UnHealthy = "UnHealthy"
```
**EN:** This block declares the class `ServerStatus`. It centers on coordinating tokenizer manager behavior.
**CN:** 该代码块声明类 `ServerStatus`。它负责承载与 分词器 管理器 相关的核心状态与行为。

### Lines 2747-2760: Implement async print exception wrapper / 实现异步print exception wrapper
```python
async def print_exception_wrapper(func):
    """
    Sometimes an asyncio function does not print exception.
    We do another wrapper to handle the exception.
    """
    try:
        await func()
    except Exception:
        traceback = get_exception_traceback()
        logger.error(f"TokenizerManager hit an exception: {traceback}")
        if hasattr(func, "__self__") and isinstance(func.__self__, TokenizerManager):
            func.__self__.dump_requests_before_crash()
        kill_process_tree(os.getpid(), include_parent=True)
        sys.exit(1)
```
**EN:** This block implements the async function `print_exception_wrapper(func)`. It focuses on Sometimes an asyncio function does not print exception., providing reusable behavior for the tokenizer manager pipeline.
**CN:** 该代码块实现异步函数 `print_exception_wrapper(func)`。它围绕 `print_exception_wrapper` 所承担的 分词器 管理器 相关职责展开，为对应处理链路提供可复用能力。

### Lines 2763-2789: Implement get processor wrapper / 实现get 处理器 wrapper
```python
def _get_processor_wrapper(server_args):
    try:
        processor = get_processor(
            server_args.tokenizer_path,
            tokenizer_mode=server_args.tokenizer_mode,
            trust_remote_code=server_args.trust_remote_code,
            revision=server_args.revision,
            use_fast=not server_args.disable_fast_image_processor,
            tokenizer_backend=server_args.tokenizer_backend,
        )
    except ValueError as e:
        error_message = str(e)
        if "does not have a slow version" in error_message:
            logger.info(
                f"Processor {server_args.tokenizer_path} does not have a slow version. Automatically use fast version"
            )
            processor = get_processor(
                server_args.tokenizer_path,
                tokenizer_mode=server_args.tokenizer_mode,
                trust_remote_code=server_args.trust_remote_code,
                revision=server_args.revision,
                use_fast=True,
                tokenizer_backend=server_args.tokenizer_backend,
            )
        else:
            raise e
    return processor
```
**EN:** This block implements the function `_get_processor_wrapper(server_args)`. It focuses on handling the tokenizer manager responsibilities represented by `_get_processor_wrapper`, providing reusable behavior for the tokenizer manager pipeline.
**CN:** 该代码块实现函数 `_get_processor_wrapper(server_args)`。它围绕 `_get_processor_wrapper` 所承担的 分词器 管理器 相关职责展开，为对应处理链路提供可复用能力。

### Lines 2792-2799: Implement determine tensor transport mode / 实现determine 张量 transport mode
```python
def _determine_tensor_transport_mode(server_args: ServerArgs) -> TensorTransportMode:
    is_cross_node = server_args.dist_init_addr

    if is_cross_node:
        # Fallback to default CPU transport for multi-node
        return "default"
    else:
        return "cuda_ipc"
```
**EN:** This block implements the function `_determine_tensor_transport_mode(server_args)`. It focuses on handling the tokenizer manager responsibilities represented by `_determine_tensor_transport_mode`, providing reusable behavior for the tokenizer manager pipeline.
**CN:** 该代码块实现函数 `_determine_tensor_transport_mode(server_args)`。它围绕 `_determine_tensor_transport_mode` 所承担的 分词器 管理器 相关职责展开，为对应处理链路提供可复用能力。

### Lines 2802-2821: Define class SignalHandler / 定义类 SignalHandler
```python
class SignalHandler:
    def __init__(self, tokenizer_manager: TokenizerManager):
        self.tokenizer_manager = tokenizer_manager

    def sigterm_handler(self, signum=None, frame=None):
        logger.warning(
            f"SIGTERM received. {signum=} {frame=}. Draining requests and shutting down..."
        )
        self.tokenizer_manager.gracefully_exit = True

    def running_phase_sigquit_handler(self, signum=None, frame=None):
        logger.error(
            f"SIGQUIT received. {signum=}, {frame=}. It usually means one child failed."
        )
        # Stop subprocess watchdog before killing processes to prevent false-positive
        # crash detection during normal shutdown
        if self.tokenizer_manager._subprocess_watchdog is not None:
            self.tokenizer_manager._subprocess_watchdog.stop()
        self.tokenizer_manager.dump_requests_before_crash()
        kill_process_tree(os.getpid())
```
**EN:** This block declares the class `SignalHandler`. It centers on coordinating tokenizer manager behavior, with methods such as __init__, sigterm_handler, running_phase_sigquit_handler.
**CN:** 该代码块声明类 `SignalHandler`。它负责承载与 分词器 管理器 相关的核心状态与行为，并通过 __init__, sigterm_handler, running_phase_sigquit_handler 等方法组织实现。

### Lines 2822-2835: Import runtime dependencies / 导入运行时依赖
```python


# Note: request abort handling logic
# We should handle all of the following cases correctly.
#
# | entrypoint | is_streaming | status          | abort engine    | cancel asyncio task   | rid_to_state                |
# | ---------- | ------------ | --------------- | --------------- | --------------------- | --------------------------- |
# | http       | yes          | validation      | background task | fast api              | del in _handle_abort_req    |
# | http       | yes          | waiting queue   | background task | fast api              | del in _handle_abort_req    |
# | http       | yes          | running         | background task | fast api              | del in _handle_batch_output |
# | http       | no           | validation      | http exception  | http exception        | del in _handle_abort_req    |
# | http       | no           | waiting queue   | type 1          | type 1 exception      | del in _handle_abort_req    |
# | http       | no           | running         | type 3          | type 3 exception      | del in _handle_batch_output |
#
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

## Key Concepts / 关键概念
- **Core types / 核心类型**: ReqState, InputFormat, TokenizerManager, ServerStatus, SignalHandler
- **Main callables / 主要可调用对象**: _slice_streaming_output_meta_info, print_exception_wrapper, _get_processor_wrapper, _determine_tensor_transport_mode
- **Domain focus / 领域焦点**: tokenizer manager / 分词器 管理器
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: asyncio, collections, contextlib, copy, dataclasses, enum, json, logging, os, sys, threading, typing
- **Third-party / 第三方库**: datetime, fastapi, http, pickle, pybase64, signal, socket, torch, uvloop, zmq, zmq.asyncio
- **Local Modules / 本地模块**: sglang.srt.configs.model_config, sglang.srt.constants, sglang.srt.disaggregation.encode_receiver, sglang.srt.disaggregation.utils, sglang.srt.environ, sglang.srt.lora.lora_registry, sglang.srt.managers.async_dynamic_batch_tokenizer, sglang.srt.managers.disagg_service, sglang.srt.managers.embed_types, sglang.srt.managers.io_struct, sglang.srt.managers.mm_utils, sglang.srt.managers.multi_tokenizer_mixin
