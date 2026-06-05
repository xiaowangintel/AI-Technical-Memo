# detokenizer_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/detokenizer_manager.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements detokenizer manager logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 反分词器 管理器 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

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
"""DetokenizerManager is a process that detokenizes the token ids."""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 16-21: Import runtime dependencies / 导入运行时依赖
```python
import dataclasses
import logging
import os
import signal
from collections import OrderedDict, defaultdict
from typing import Dict, List, Optional, Tuple, Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 23-27: Import runtime dependencies / 导入运行时依赖
```python
import psutil
import pybase64
import setproctitle
import torch
import zmq
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 29-53: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.constants import HEALTH_CHECK_RID_PREFIX
from sglang.srt.environ import envs
from sglang.srt.managers.io_struct import (
    BatchEmbeddingOutput,
    BatchStrOutput,
    BatchTokenIDOutput,
    FreezeGCReq,
)
from sglang.srt.managers.multi_tokenizer_mixin import MultiHttpWorkerDetokenizerMixin
from sglang.srt.observability.cpu_monitor import start_cpu_monitor_thread
from sglang.srt.server_args import PortArgs, ServerArgs
from sglang.srt.utils import (
    configure_logger,
    freeze_gc,
    kill_itself_when_parent_died,
)
from sglang.srt.utils.hf_transformers_utils import get_tokenizer
from sglang.srt.utils.network import get_zmq_socket
from sglang.srt.utils.patch_tokenizer import decode_without_hf_kwargs
from sglang.srt.utils.watchdog import Watchdog
from sglang.utils import (
    TypeBasedDispatcher,
    find_printable_text,
    get_exception_traceback,
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 55-64: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)

# Maximum number of request states that detokenizer can hold. When exceeded,
# oldest request states will be evicted. Default: 65536 (1<<16).
# For more details, see: https://github.com/sgl-project/sglang/issues/2812
# Use power of 2 values for better memory allocation.
DETOKENIZER_MAX_STATES = int(os.environ.get("SGLANG_DETOKENIZER_MAX_STATES", 1 << 16))


@dataclasses.dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 65-73: Define class DecodeStatus / 定义类 DecodeStatus
```python
class DecodeStatus:
    """Store the status of incremental decoding."""

    decoded_text: str
    decode_ids: List[int]
    surr_offset: int
    read_offset: int
    # Offset that's sent to tokenizer for incremental update.
    sent_offset: int = 0
```
**EN:** This block declares the class `DecodeStatus`. It centers on Store the status of incremental decoding..
**CN:** 该代码块声明类 `DecodeStatus`。它负责承载与 反分词器 管理器 相关的核心状态与行为。

### Lines 76-77: Provide supporting module logic / 提供辅助模块逻辑
```python
class DetokenizerManager(MultiHttpWorkerDetokenizerMixin):
    """DetokenizerManager is a process that detokenizes the token ids."""
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 79-94: Initialize DetokenizerManager / 初始化 DetokenizerManager
```python
    def __init__(
        self,
        server_args: ServerArgs,
        port_args: PortArgs,
    ):
        # Init inter-process communication
        self.init_ipc_channels(port_args, server_args)

        # Init tokenizer
        self.init_tokenizer(server_args)

        # Init running status
        self.init_running_status(server_args)

        # Init dispatcher
        self.init_request_dispatcher()
```
**EN:** This block implements the initializer `__init__(server_args, port_args)` for `DetokenizerManager`. It prepares the object state and connects the instance to the surrounding detokenizer manager workflow.
**CN:** 该代码块实现 `DetokenizerManager` 的初始化方法 `__init__(server_args, port_args)`。它负责准备对象状态，并把实例接入 反分词器 管理器 相关的运行流程。

### Lines 96-107: Implement init ipc channels / 实现init ipc channels
```python
    def init_ipc_channels(self, port_args: PortArgs, server_args: ServerArgs):
        context = zmq.Context(2)
        self.recv_from_scheduler = get_zmq_socket(
            context, zmq.PULL, port_args.detokenizer_ipc_name, True
        )
        # In multi-tokenizer mode, results are pushed back to each TokenizerWorker
        # directly via SocketMapping inside multi_http_worker_event_loop, so the
        # single send_to_tokenizer socket is unused.
        if server_args.tokenizer_worker_num == 1:
            self.send_to_tokenizer = get_zmq_socket(
                context, zmq.PUSH, port_args.tokenizer_ipc_name, False
            )
```
**EN:** This block implements the method `init_ipc_channels(port_args, server_args)` on `DetokenizerManager`. It focuses on handling the detokenizer manager responsibilities represented by `init_ipc_channels`, so the class can advance the detokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `DetokenizerManager` 上的方法 `init_ipc_channels(port_args, server_args)`。它围绕 `init_ipc_channels` 所承担的 反分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 109-119: Implement init tokenizer / 实现init 分词器
```python
    def init_tokenizer(self, server_args: ServerArgs):
        if server_args.skip_tokenizer_init:
            self.tokenizer = None
        else:
            self.tokenizer = get_tokenizer(
                server_args.tokenizer_path,
                tokenizer_mode=server_args.tokenizer_mode,
                trust_remote_code=server_args.trust_remote_code,
                revision=server_args.revision,
                tokenizer_backend=server_args.tokenizer_backend,
            )
```
**EN:** This block implements the method `init_tokenizer(server_args)` on `DetokenizerManager`. It focuses on handling the detokenizer manager responsibilities represented by `init_tokenizer`, so the class can advance the detokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `DetokenizerManager` 上的方法 `init_tokenizer(server_args)`。它围绕 `init_tokenizer` 所承担的 反分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 121-134: Implement init running status / 实现init running status
```python
    def init_running_status(self, server_args: ServerArgs):
        self.decode_status = LimitedCapacityDict(capacity=DETOKENIZER_MAX_STATES)
        self.disable_tokenizer_batch_decode = server_args.disable_tokenizer_batch_decode
        self.is_tool_call_parser_gpt_oss = server_args.tool_call_parser == "gpt-oss"

        self.soft_watchdog = Watchdog.create(
            debug_name="DetokenizerManager",
            watchdog_timeout=server_args.soft_watchdog_timeout,
            soft=True,
            test_stuck_time=envs.SGLANG_TEST_STUCK_DETOKENIZER.get(),
        )

        if server_args.enable_metrics:
            start_cpu_monitor_thread("detokenizer")
```
**EN:** This block implements the method `init_running_status(server_args)` on `DetokenizerManager`. It focuses on handling the detokenizer manager responsibilities represented by `init_running_status`, so the class can advance the detokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `DetokenizerManager` 上的方法 `init_running_status(server_args)`。它围绕 `init_running_status` 所承担的 反分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 136-143: Implement init request dispatcher / 实现init 请求 dispatcher
```python
    def init_request_dispatcher(self):
        self._request_dispatcher = TypeBasedDispatcher(
            [
                (BatchEmbeddingOutput, self.handle_batch_embedding_out),
                (BatchTokenIDOutput, self.handle_batch_token_id_out),
                (FreezeGCReq, self.handle_freeze_gc_req),
            ]
        )
```
**EN:** This block implements the method `init_request_dispatcher()` on `DetokenizerManager`. It focuses on handling the detokenizer manager responsibilities represented by `init_request_dispatcher`, so the class can advance the detokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `DetokenizerManager` 上的方法 `init_request_dispatcher()`。它围绕 `init_request_dispatcher` 所承担的 反分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 145-153: Implement event loop / 实现event loop
```python
    def event_loop(self):
        """The event loop that handles requests"""
        while True:
            with self.soft_watchdog.disable():
                recv_obj = self.recv_from_scheduler.recv_pyobj()
            output = self._request_dispatcher(recv_obj)
            if output is not None:
                self.send_to_tokenizer.send_pyobj(output)
            self.soft_watchdog.feed()
```
**EN:** This block implements the method `event_loop()` on `DetokenizerManager`. It focuses on The event loop that handles requests, so the class can advance the detokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `DetokenizerManager` 上的方法 `event_loop()`。它围绕 `event_loop` 所承担的 反分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 155-180: Implement trim matched stop / 实现trim matched stop
```python
    def trim_matched_stop(
        self, output: Union[str, List[int]], finished_reason: Dict, no_stop_trim: bool
    ):
        if no_stop_trim or not finished_reason:
            return output

        matched = finished_reason.get("matched", None)
        if not matched:
            return output

        # TODO(lmzheng): handle the case where multiple stop strs are hit

        # Trim stop str.
        if isinstance(matched, str) and isinstance(output, str):
            pos = output.find(matched)
            return output[:pos] if pos != -1 else output

        # Trim stop token.
        if isinstance(matched, int) and isinstance(output, list):
            # 200012 <|call|> is the tool call token and one of eos tokens for gpt-oss model
            if output[-1] == 200012 and self.is_tool_call_parser_gpt_oss:
                return output
            assert len(output) > 0
            # NOTE: We can always assume the last token is the matched stop token
            return output[:-1]
        return output
```
**EN:** This block implements the method `trim_matched_stop(output, finished_reason, no_stop_trim)` on `DetokenizerManager`. It focuses on handling the detokenizer manager responsibilities represented by `trim_matched_stop`, so the class can advance the detokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `DetokenizerManager` 上的方法 `trim_matched_stop(output, finished_reason, no_stop_trim)`。它围绕 `trim_matched_stop` 所承担的 反分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 182-184: Implement handle batch embedding out / 实现handle 批处理 嵌入 out
```python
    def handle_batch_embedding_out(self, recv_obj: BatchEmbeddingOutput):
        # If it is embedding model, no detokenization is needed.
        return recv_obj
```
**EN:** This block implements the method `handle_batch_embedding_out(recv_obj)` on `DetokenizerManager`. It focuses on handling the detokenizer manager responsibilities represented by `handle_batch_embedding_out`, so the class can advance the detokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `DetokenizerManager` 上的方法 `handle_batch_embedding_out(recv_obj)`。它围绕 `handle_batch_embedding_out` 所承担的 反分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 186-229: Implement grouped batch decode / 实现grouped 批处理 解码
```python
    def _grouped_batch_decode(
        self,
        ids_list: List[List[int]],
        skip_list: List[bool],
        space_list: List[bool],
    ) -> List[str]:
        """Batch decode with grouping by (skip_special_tokens, spaces_between_special_tokens)."""

        if not getattr(self.tokenizer, "is_fast", False):
            return [
                decode_without_hf_kwargs(self.tokenizer, ids, skip)
                for ids, skip in zip(ids_list, skip_list)
            ]

        # fast path
        first_skip, first_space = skip_list[0], space_list[0]
        if all(
            s == first_skip and sp == first_space
            for s, sp in zip(skip_list, space_list)
        ):
            return self.tokenizer.batch_decode(
                ids_list,
                skip_special_tokens=first_skip,
                spaces_between_special_tokens=first_space,
            )

        # Group indices by (skip, space) tuple
        groups: Dict[Tuple[bool, bool], List[int]]
        groups = defaultdict(list)
        for idx, (skip, space) in enumerate(zip(skip_list, space_list)):
            groups[(skip, space)].append(idx)

        # Decode each group and collect results
        results: List[str] = [""] * len(ids_list)
        for (skip, space), indices in groups.items():
            decoded = self.tokenizer.batch_decode(
                [ids_list[idx] for idx in indices],
                skip_special_tokens=skip,
                spaces_between_special_tokens=space,
            )
            for idx, text in zip(indices, decoded):
                results[idx] = text

        return results
```
**EN:** This block implements the method `_grouped_batch_decode(ids_list, skip_list, space_list)` on `DetokenizerManager`. It focuses on Batch decode with grouping by (skip_special_tokens, spaces_between_special_tokens)., so the class can advance the detokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `DetokenizerManager` 上的方法 `_grouped_batch_decode(ids_list, skip_list, space_list)`。它围绕 `_grouped_batch_decode` 所承担的 反分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 231-266: Implement decode batch token id output / 实现解码 批处理 Token id output
```python
    def _decode_batch_token_id_output(self, recv_obj: BatchTokenIDOutput):
        bs = len(recv_obj.rids)

        # Initialize decode status
        read_ids, surr_ids = [], []
        for i in range(bs):
            rid = recv_obj.rids[i]
            if rid not in self.decode_status:
                s = DecodeStatus(
                    decoded_text=recv_obj.decoded_texts[i],
                    decode_ids=recv_obj.decode_ids[i],
                    surr_offset=0,
                    read_offset=recv_obj.read_offsets[i],
                )
                self.decode_status[rid] = s
            else:
                s = self.decode_status[rid]
                s.decode_ids.extend(recv_obj.decode_ids[i])

            read_ids.append(
                self.trim_matched_stop(
                    s.decode_ids[s.surr_offset :],
                    recv_obj.finished_reasons[i],
                    recv_obj.no_stop_trim[i],
                )
            )
            surr_ids.append(s.decode_ids[s.surr_offset : s.read_offset])

        # Decode token ids to strings
        if not self.disable_tokenizer_batch_decode:
            surr_texts = self._grouped_batch_decode(
                surr_ids,
                recv_obj.skip_special_tokens,
                recv_obj.spaces_between_special_tokens,
            )
            read_texts = self._grouped_batch_decode(
```
**EN:** This block implements the method `_decode_batch_token_id_output(recv_obj)` on `DetokenizerManager`. It focuses on handling the detokenizer manager responsibilities represented by `_decode_batch_token_id_output`, so the class can advance the detokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `DetokenizerManager` 上的方法 `_decode_batch_token_id_output(recv_obj)`。它围绕 `_decode_batch_token_id_output` 所承担的 反分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 267-302: Continue decode batch token id output / 继续说明解码 批处理 Token id output
```python
                read_ids,
                recv_obj.skip_special_tokens,
                recv_obj.spaces_between_special_tokens,
            )
        else:
            # Do not use batch decode to prevent some detokenization edge cases (e.g., gpt-oss).
            surr_texts = [
                self.tokenizer.decode(
                    surr, skip_special_tokens=skip, spaces_between_special_tokens=space
                )
                for surr, skip, space in zip(
                    surr_ids,
                    recv_obj.skip_special_tokens,
                    recv_obj.spaces_between_special_tokens,
                )
            ]
            read_texts = [
                self.tokenizer.decode(
                    read, skip_special_tokens=skip, spaces_between_special_tokens=space
                )
                for read, skip, space in zip(
                    read_ids,
                    recv_obj.skip_special_tokens,
                    recv_obj.spaces_between_special_tokens,
                )
            ]

        # Incremental decoding
        output_strs = []
        for i in range(bs):
            rid = recv_obj.rids[i]
            try:
                s = self.decode_status[rid]
            except KeyError:
                raise RuntimeError(
                    f"Decode status not found for request {rid}. "
```
**EN:** This block implements the method `_decode_batch_token_id_output(recv_obj)` on `DetokenizerManager`. It focuses on handling the detokenizer manager responsibilities represented by `_decode_batch_token_id_output`, so the class can advance the detokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `DetokenizerManager` 上的方法 `_decode_batch_token_id_output(recv_obj)`。它围绕 `_decode_batch_token_id_output` 所承担的 反分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 303-334: Continue decode batch token id output / 继续说明解码 批处理 Token id output
```python
                    "It may be due to the request being evicted from the decode status due to memory pressure. "
                    "Please increase the maximum number of requests by setting "
                    "the SGLANG_DETOKENIZER_MAX_STATES environment variable to a bigger value than the default value. "
                    f"The current value is {DETOKENIZER_MAX_STATES}. "
                    "For more details, see: https://github.com/sgl-project/sglang/issues/2812"
                )
            new_text = read_texts[i][len(surr_texts[i]) :]
            if recv_obj.finished_reasons[i] is None:
                # Streaming chunk: update the decode status
                if new_text and not new_text.endswith("�"):
                    s.decoded_text += new_text
                    s.surr_offset = s.read_offset
                    s.read_offset = len(s.decode_ids)
                    new_text = ""
                else:
                    new_text = find_printable_text(new_text)
            else:
                if rid in self.decode_status:
                    del self.decode_status[rid]

            output_str = self.trim_matched_stop(
                s.decoded_text + new_text,
                recv_obj.finished_reasons[i],
                recv_obj.no_stop_trim[i],
            )

            # Incrementally send text.
            incremental_output = output_str[s.sent_offset :]
            s.sent_offset = len(output_str)
            output_strs.append(incremental_output)

        return output_strs
```
**EN:** This block implements the method `_decode_batch_token_id_output(recv_obj)` on `DetokenizerManager`. It focuses on handling the detokenizer manager responsibilities represented by `_decode_batch_token_id_output`, so the class can advance the detokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `DetokenizerManager` 上的方法 `_decode_batch_token_id_output(recv_obj)`。它围绕 `_decode_batch_token_id_output` 所承担的 反分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 335-336: Provide supporting module logic / 提供辅助模块逻辑
```python

    @staticmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 337-353: Implement b64 encode per request / 实现b64 encode per 请求
```python
    def _b64_encode_per_request(
        data_list: Optional[List[Optional[torch.Tensor]]],
    ) -> Optional[List[Optional[str]]]:
        """Encode a per-request list of tensors as base64 strings, off the
        tokenizer hot path. Returns None when the input is None; per-item None
        stays None.
        """
        if data_list is None:
            return None
        return [
            (
                pybase64.b64encode(item.numpy().tobytes()).decode("utf-8")
                if item is not None
                else None
            )
            for item in data_list
        ]
```
**EN:** This block implements the method `_b64_encode_per_request(data_list)` on `DetokenizerManager`. It focuses on Encode a per-request list of tensors as base64 strings, off the tokenizer hot path., so the class can advance the detokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `DetokenizerManager` 上的方法 `_b64_encode_per_request(data_list)`。它围绕 `_b64_encode_per_request` 所承担的 反分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 355-402: Implement handle batch token id out / 实现handle 批处理 Token id out
```python
    def handle_batch_token_id_out(self, recv_obj: BatchTokenIDOutput):
        # If handling idle batch, set output_strs to [].
        output_strs = (
            self._decode_batch_token_id_output(recv_obj)
            if len(recv_obj.rids) > 0
            else []
        )
        routed_experts = self._b64_encode_per_request(recv_obj.routed_experts)
        indexer_topk = self._b64_encode_per_request(recv_obj.indexer_topk)
        return BatchStrOutput(
            rids=recv_obj.rids,
            http_worker_ipcs=recv_obj.http_worker_ipcs,
            finished_reasons=recv_obj.finished_reasons,
            output_strs=output_strs,
            output_ids=recv_obj.output_ids,
            prompt_tokens=recv_obj.prompt_tokens,
            reasoning_tokens=recv_obj.reasoning_tokens,
            completion_tokens=recv_obj.completion_tokens,
            cached_tokens=recv_obj.cached_tokens,
            cached_tokens_details=recv_obj.cached_tokens_details,
            spec_verify_ct=recv_obj.spec_verify_ct,
            spec_num_correct_drafts=recv_obj.spec_num_correct_drafts,
            spec_correct_drafts_histogram=recv_obj.spec_correct_drafts_histogram,
            input_token_logprobs_val=recv_obj.input_token_logprobs_val,
            input_token_logprobs_idx=recv_obj.input_token_logprobs_idx,
            output_token_logprobs_val=recv_obj.output_token_logprobs_val,
            output_token_logprobs_idx=recv_obj.output_token_logprobs_idx,
            input_top_logprobs_val=recv_obj.input_top_logprobs_val,
            input_top_logprobs_idx=recv_obj.input_top_logprobs_idx,
            output_top_logprobs_val=recv_obj.output_top_logprobs_val,
            output_top_logprobs_idx=recv_obj.output_top_logprobs_idx,
            input_token_ids_logprobs_val=recv_obj.input_token_ids_logprobs_val,
            input_token_ids_logprobs_idx=recv_obj.input_token_ids_logprobs_idx,
            output_token_ids_logprobs_val=recv_obj.output_token_ids_logprobs_val,
            output_token_ids_logprobs_idx=recv_obj.output_token_ids_logprobs_idx,
            output_token_entropy_val=recv_obj.output_token_entropy_val,
            output_hidden_states=recv_obj.output_hidden_states,
            routed_experts=routed_experts,
            indexer_topk=indexer_topk,
            customized_info=recv_obj.customized_info,
            placeholder_tokens_idx=None,
            placeholder_tokens_val=None,
            retraction_counts=recv_obj.retraction_counts,
            token_steps=recv_obj.token_steps,
            load=recv_obj.load,
            dp_ranks=recv_obj.dp_ranks,
            time_stats=recv_obj.time_stats,
        )
```
**EN:** This block implements the method `handle_batch_token_id_out(recv_obj)` on `DetokenizerManager`. It focuses on handling the detokenizer manager responsibilities represented by `handle_batch_token_id_out`, so the class can advance the detokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `DetokenizerManager` 上的方法 `handle_batch_token_id_out(recv_obj)`。它围绕 `handle_batch_token_id_out` 所承担的 反分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 404-406: Implement handle freeze gc req / 实现handle freeze gc req
```python
    def handle_freeze_gc_req(self, recv_req: FreezeGCReq):
        freeze_gc("Detokenizer Manager")
        return None
```
**EN:** This block implements the method `handle_freeze_gc_req(recv_req)` on `DetokenizerManager`. It focuses on handling the detokenizer manager responsibilities represented by `handle_freeze_gc_req`, so the class can advance the detokenizer manager workflow in a self-contained way.
**CN:** 该代码块实现 `DetokenizerManager` 上的方法 `handle_freeze_gc_req(recv_req)`。它围绕 `handle_freeze_gc_req` 所承担的 反分词器 管理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 409-410: Implement is health check request / 实现is 健康 检查 请求
```python
def is_health_check_request(rid: Optional[str]) -> bool:
    return isinstance(rid, str) and rid.startswith(HEALTH_CHECK_RID_PREFIX)
```
**EN:** This block implements the function `is_health_check_request(rid)`. It focuses on handling the detokenizer manager responsibilities represented by `is_health_check_request`, providing reusable behavior for the detokenizer manager pipeline.
**CN:** 该代码块实现函数 `is_health_check_request(rid)`。它围绕 `is_health_check_request` 所承担的 反分词器 管理器 相关职责展开，为对应处理链路提供可复用能力。

### Lines 413-423: Define class LimitedCapacityDict / 定义类 LimitedCapacityDict
```python
class LimitedCapacityDict(OrderedDict):
    def __init__(self, capacity: int, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.capacity = capacity

    def __setitem__(self, key, value):
        if len(self) >= self.capacity:
            # Remove the oldest element (first item in the dict)
            self.popitem(last=False)
        # Set the new item
        super().__setitem__(key, value)
```
**EN:** This block declares the class `LimitedCapacityDict`. It centers on coordinating detokenizer manager behavior, with methods such as __init__, __setitem__.
**CN:** 该代码块声明类 `LimitedCapacityDict`。它负责承载与 反分词器 管理器 相关的核心状态与行为，并通过 __init__, __setitem__ 等方法组织实现。

### Lines 426-448: Implement run detokenizer process / 实现run 反分词器 process
```python
def run_detokenizer_process(
    server_args: ServerArgs,
    port_args: PortArgs,
    detokenizer_manager_class=DetokenizerManager,
):
    kill_itself_when_parent_died()
    setproctitle.setproctitle("sglang::detokenizer")
    configure_logger(server_args)
    parent_process = psutil.Process().parent()

    manager = None
    try:
        manager = detokenizer_manager_class(server_args, port_args)
        if server_args.tokenizer_worker_num == 1:
            manager.event_loop()
        else:
            manager.multi_http_worker_event_loop()
    except Exception:
        traceback = get_exception_traceback()
        logger.error(f"DetokenizerManager hit an exception: {traceback}")
        if manager is not None:
            manager.maybe_clear_socket_mapping()
        parent_process.send_signal(signal.SIGQUIT)
```
**EN:** This block implements the function `run_detokenizer_process(server_args, port_args, detokenizer_manager_class)`. It focuses on handling the detokenizer manager responsibilities represented by `run_detokenizer_process`, providing reusable behavior for the detokenizer manager pipeline.
**CN:** 该代码块实现函数 `run_detokenizer_process(server_args, port_args, detokenizer_manager_class)`。它围绕 `run_detokenizer_process` 所承担的 反分词器 管理器 相关职责展开，为对应处理链路提供可复用能力。

## Key Concepts / 关键概念
- **Core types / 核心类型**: DecodeStatus, DetokenizerManager, LimitedCapacityDict
- **Main callables / 主要可调用对象**: is_health_check_request, run_detokenizer_process
- **Domain focus / 领域焦点**: detokenizer manager / 反分词器 管理器
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: collections, dataclasses, logging, os, typing
- **Third-party / 第三方库**: psutil, pybase64, setproctitle, signal, torch, zmq
- **Local Modules / 本地模块**: sglang.srt.constants, sglang.srt.environ, sglang.srt.managers.io_struct, sglang.srt.managers.multi_tokenizer_mixin, sglang.srt.observability.cpu_monitor, sglang.srt.server_args, sglang.srt.utils, sglang.srt.utils.hf_transformers_utils, sglang.srt.utils.network, sglang.srt.utils.patch_tokenizer, sglang.srt.utils.watchdog, sglang.utils
