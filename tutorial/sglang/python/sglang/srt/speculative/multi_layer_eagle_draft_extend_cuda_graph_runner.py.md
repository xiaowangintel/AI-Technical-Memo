# multi_layer_eagle_draft_extend_cuda_graph_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/multi_layer_eagle_draft_extend_cuda_graph_runner.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements speculative decoding infrastructure, such as workers, metadata objects, CUDA-graph runners, or shared utilities. It coordinates draft-model execution with target-model verification to improve throughput. / 该文件实现了推测解码基础设施，例如 worker、元数据对象、CUDA Graph 运行器或共享工具函数。它通过协调草稿模型执行与目标模型校验来提升吞吐。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60: Module header, imports, and shared constants (part 1/2)
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

from __future__ import annotations

import bisect
import logging
import time
from dataclasses import dataclass
from typing import TYPE_CHECKING, Callable, List, Optional

import torch

from sglang.srt.layers.dp_attention import DpPaddingMode, set_dp_buffer_len
from sglang.srt.model_executor.cuda_graph_runner import (
    CUDA_GRAPH_CAPTURE_FAILED_MSG,
    CudaGraphRunner,
    DeepEPCudaGraphRunnerAdapter,
    LogitsProcessorOutput,
    get_batch_sizes_to_capture,
    get_global_graph_memory_pool,
    model_capture_mode,
    set_global_graph_memory_pool,
    set_is_extend_in_batch,
    set_torch_compile_config,
)
from sglang.srt.model_executor.forward_batch_info import (
    CaptureHiddenMode,
    ForwardBatch,
    ForwardMode,
)
from sglang.srt.model_executor.input_buffers import ForwardInputBuffers
from sglang.srt.speculative.eagle_info import EagleDraftExtendInput
from sglang.srt.speculative.multi_layer_eagle_utils import assign_new_state_triton
from sglang.srt.speculative.spec_utils import fast_topk
from sglang.srt.utils import (
    get_available_gpu_memory,
    require_attn_tp_gather,
    require_gathered_buffer,
    require_mlp_sync,
    require_mlp_tp_gather,
)

if TYPE_CHECKING:
    from sglang.srt.speculative.multi_layer_eagle_worker_v2 import (
        MultiLayerEagleDraftWorker,
    )
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on. It is one chunk of a larger setup region.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。 这是较大初始化区域中的一部分。

### Lines 61-63: Module header, imports, and shared constants (part 2/2)
```python
logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on. It is one chunk of a larger setup region.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。 这是较大初始化区域中的一部分。

### Lines 65-84: MultiLayerEagleDraftExtendInputBuffers class declaration
```python
class MultiLayerEagleDraftExtendInputBuffers(ForwardInputBuffers):
    # Sliced from shared parent buffers
    input_ids: torch.Tensor
    out_cache_loc: torch.Tensor
    swa_out_cache_loc: torch.Tensor
    positions: torch.Tensor
    # Shared from parent
    seq_lens: torch.Tensor
    seq_lens_cpu: torch.Tensor
    req_pool_indices: torch.Tensor
    num_correct_drafts: torch.Tensor
    num_accept_tokens: torch.Tensor
    # Per-step buffers
    extend_seq_lens: torch.Tensor
    extend_start_loc: torch.Tensor
    mrope_positions: torch.Tensor
    hidden_states: torch.Tensor
    next_token_logits_buffer: torch.Tensor
    global_num_tokens_gpu: Optional[torch.Tensor]
    global_num_tokens_for_logprob_gpu: Optional[torch.Tensor]
```
**EN:** This block declares the `MultiLayerEagleDraftExtendInputBuffers` class, which exists to produce draft tokens or draft-side state. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `MultiLayerEagleDraftExtendInputBuffers` 类，其职责是生成草稿 token 或草稿侧状态。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 87-87: MultiLayerEagleDraftExtendCudaGraphRunner class declaration
```python
class MultiLayerEagleDraftExtendCudaGraphRunner:
```
**EN:** This block declares the `MultiLayerEagleDraftExtendCudaGraphRunner` class, which exists to produce draft tokens or draft-side state. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `MultiLayerEagleDraftExtendCudaGraphRunner` 类，其职责是生成草稿 token 或草稿侧状态。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 88-128: MultiLayerEagleDraftExtendCudaGraphRunner initializer
```python
    def __init__(self, eagle_worker: MultiLayerEagleDraftWorker, step: int):
        # Parse args
        self.step = step
        self.eagle_worker = eagle_worker
        self.model_runner = model_runner = eagle_worker.mtp_model_runner(self.step)
        self.forward_mode = ForwardMode.DRAFT_EXTEND_V2

        self.graphs = {}
        self.output_buffers = {}
        self.enable_torch_compile = model_runner.server_args.enable_torch_compile
        self.disable_padding = model_runner.server_args.disable_cuda_graph_padding
        self.require_gathered_buffer = require_gathered_buffer(model_runner.server_args)
        self.require_mlp_tp_gather = require_mlp_tp_gather(model_runner.server_args)
        self.require_mlp_sync = require_mlp_sync(model_runner.server_args)
        self.require_attn_tp_gather = require_attn_tp_gather(model_runner.server_args)
        self.tp_size = self.model_runner.tp_size
        self.dp_size = model_runner.server_args.dp_size
        self.enable_pdmux = model_runner.server_args.enable_pdmux
        self.speculative_num_steps = model_runner.server_args.speculative_num_steps
        self.speculative_num_draft_tokens = (
            model_runner.server_args.speculative_num_draft_tokens
        )
        self.topk = model_runner.server_args.speculative_eagle_topk
        self.enable_profile_cuda_graph = (
            model_runner.server_args.enable_profile_cuda_graph
        )
        self.capture_bs, self.compile_bs = get_batch_sizes_to_capture(model_runner)
        self.padded_static_len = -1
        self.deepep_adapter = DeepEPCudaGraphRunnerAdapter()

        # For Attention Backend
        self.num_tokens_per_bs = self.speculative_num_steps + 1 + step
        self.max_bs = max(self.capture_bs)
        self.max_num_token = self.max_bs * self.num_tokens_per_bs

        self.eagle_worker.draft_extend_attn_backend_list[
            self.step
        ].init_cuda_graph_state(self.max_bs, self.max_num_token)
        self.seq_len_fill_value = self.eagle_worker.draft_extend_attn_backend_list[
            self.step
        ].get_cuda_graph_seq_len_fill_value()
```
**EN:** This block initializes the `MultiLayerEagleDraftExtendCudaGraphRunner` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `MultiLayerEagleDraftExtendCudaGraphRunner` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 130-189: MultiLayerEagleDraftExtendCudaGraphRunner.init_buffers_and_capture method (part 1/3)
```python
    def init_buffers_and_capture(
        self,
        cuda_graph_buffers,
        offset,
        next_cuda_graph_runner,
    ):
        self.next_cuda_graph_runner = next_cuda_graph_runner
        seq_lens_cpu = cuda_graph_buffers["seq_lens_cpu"]
        self.extend_seq_lens_cpu = [self.num_tokens_per_bs] * self.max_bs

        if self.enable_torch_compile:
            set_torch_compile_config()

        # Graph inputs
        with torch.device(self.model_runner.device):
            # sliced buffers
            # slice according to max_num_token
            input_ids = cuda_graph_buffers["input_ids"][
                offset : offset + self.max_num_token
            ]
            out_cache_loc = cuda_graph_buffers["out_cache_loc"][
                offset : offset + self.max_num_token
            ]
            swa_out_cache_loc = cuda_graph_buffers["swa_out_cache_loc"][
                offset : offset + self.max_num_token
            ]
            positions = cuda_graph_buffers["positions"][
                offset : offset + self.max_num_token
            ]

            # shared states
            seq_lens = cuda_graph_buffers["seq_lens"]
            req_pool_indices = cuda_graph_buffers["req_pool_indices"]
            num_correct_drafts = cuda_graph_buffers["num_correct_drafts"]
            num_accept_tokens = cuda_graph_buffers["num_accept_tokens"]

            extend_seq_lens = torch.full(
                (self.max_bs,),
                self.num_tokens_per_bs,
                dtype=torch.int32,
            )
            extend_start_loc = torch.arange(
                0,
                self.max_bs * self.num_tokens_per_bs,
                step=self.num_tokens_per_bs,
                dtype=torch.int32,
            )

            mrope_positions = torch.zeros((3, self.max_num_token), dtype=torch.int64)

            hidden_states = torch.zeros(
                (
                    self.max_num_token,
                    EagleDraftExtendInput.hidden_size_for(self.eagle_worker),
                ),
                dtype=EagleDraftExtendInput.dtype_for(self.eagle_worker),
            )

            if self.require_gathered_buffer:
                if self.require_mlp_tp_gather:
```
**EN:** This block uses `MultiLayerEagleDraftExtendCudaGraphRunner.init_buffers_and_capture` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `MultiLayerEagleDraftExtendCudaGraphRunner.init_buffers_and_capture` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 190-249: MultiLayerEagleDraftExtendCudaGraphRunner.init_buffers_and_capture method (part 2/3)
```python
                    global_num_tokens_gpu = torch.zeros(
                        (self.dp_size,), dtype=torch.int32
                    )
                    global_num_tokens_for_logprob_gpu = torch.zeros(
                        (self.dp_size,), dtype=torch.int32
                    )
                else:
                    assert self.require_attn_tp_gather
                    global_num_tokens_gpu = torch.zeros((1,), dtype=torch.int32)
                    global_num_tokens_for_logprob_gpu = torch.zeros(
                        (1,), dtype=torch.int32
                    )
            else:
                global_num_tokens_gpu = None
                global_num_tokens_for_logprob_gpu = None

            if hasattr(
                self.model_runner.model_config.hf_config, "draft_vocab_size"
            ):  # llama_eagle
                vocab_size = self.model_runner.model_config.hf_config.draft_vocab_size
            elif hasattr(
                self.model_runner.model_config.hf_config, "hot_vocab_size"
            ):  # llama_eagle3
                vocab_size = self.model_runner.model_config.hf_config.hot_vocab_size
            else:
                vocab_size = self.model_runner.model_config.vocab_size

            next_token_logits_buffer = torch.zeros(
                (
                    (
                        self.max_bs * self.num_tokens_per_bs
                        if self.forward_mode == ForwardMode.DRAFT_EXTEND_V2
                        else self.max_bs
                    ),
                    vocab_size,
                ),
                dtype=torch.float,
            )

        self.buffers = MultiLayerEagleDraftExtendInputBuffers(
            input_ids=input_ids,
            out_cache_loc=out_cache_loc,
            swa_out_cache_loc=swa_out_cache_loc,
            positions=positions,
            seq_lens=seq_lens,
            seq_lens_cpu=seq_lens_cpu,
            req_pool_indices=req_pool_indices,
            num_correct_drafts=num_correct_drafts,
            num_accept_tokens=num_accept_tokens,
            extend_seq_lens=extend_seq_lens,
            extend_start_loc=extend_start_loc,
            mrope_positions=mrope_positions,
            hidden_states=hidden_states,
            next_token_logits_buffer=next_token_logits_buffer,
            global_num_tokens_gpu=global_num_tokens_gpu,
            global_num_tokens_for_logprob_gpu=global_num_tokens_for_logprob_gpu,
        )

        # Capture
        try:
```
**EN:** This block uses `MultiLayerEagleDraftExtendCudaGraphRunner.init_buffers_and_capture` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk continues the routine's middle-stage logic and data movement.
**CN:** 该代码块通过 `MultiLayerEagleDraftExtendCudaGraphRunner.init_buffers_and_capture` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分继续展示例程中段的逻辑与数据流动。

### Lines 250-255: MultiLayerEagleDraftExtendCudaGraphRunner.init_buffers_and_capture method (part 3/3)
```python
            with model_capture_mode():
                self.capture()
        except RuntimeError as e:
            raise Exception(
                f"Capture cuda graph failed: {e}\n{CUDA_GRAPH_CAPTURE_FAILED_MSG}"
            )
```
**EN:** This block uses `MultiLayerEagleDraftExtendCudaGraphRunner.init_buffers_and_capture` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `MultiLayerEagleDraftExtendCudaGraphRunner.init_buffers_and_capture` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 257-276: MultiLayerEagleDraftExtendCudaGraphRunner.can_run method
```python
    def can_run(self, forward_batch: ForwardBatch):
        if self.require_mlp_tp_gather:
            cuda_graph_bs = (
                max(forward_batch.global_num_tokens_cpu) // self.num_tokens_per_bs
                if self.model_runner.spec_algorithm.is_eagle()
                else max(forward_batch.global_num_tokens_cpu)
            )
        else:
            cuda_graph_bs = forward_batch.seq_lens.numel()

        is_bs_supported = (
            cuda_graph_bs in self.graphs
            if self.disable_padding
            else cuda_graph_bs <= self.max_bs
        )

        if self.require_mlp_sync:
            is_bs_supported = is_bs_supported and forward_batch.can_run_dp_cuda_graph

        return is_bs_supported
```
**EN:** This block uses `MultiLayerEagleDraftExtendCudaGraphRunner.can_run` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleDraftExtendCudaGraphRunner.can_run` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 278-279: MultiLayerEagleDraftExtendCudaGraphRunner._create_graph method
```python
    def _create_graph(self):
        return torch.cuda.CUDAGraph()
```
**EN:** This block uses `MultiLayerEagleDraftExtendCudaGraphRunner._create_graph` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleDraftExtendCudaGraphRunner._create_graph` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 281-285: MultiLayerEagleDraftExtendCudaGraphRunner._capture_init method
```python
    def _capture_init(self, run_once_fn):
        for _ in range(2):
            torch.cuda.synchronize()
            self.model_runner.tp_group.barrier()
            run_once_fn()
```
**EN:** This block uses `MultiLayerEagleDraftExtendCudaGraphRunner._capture_init` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleDraftExtendCudaGraphRunner._capture_init` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 287-290: MultiLayerEagleDraftExtendCudaGraphRunner._capture_graph method
```python
    def _capture_graph(self, graph, pool, stream, run_once_fn):
        with torch.cuda.graph(graph, pool=pool, stream=stream):
            out = run_once_fn()
        return out
```
**EN:** This block uses `MultiLayerEagleDraftExtendCudaGraphRunner._capture_graph` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleDraftExtendCudaGraphRunner._capture_graph` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 292-293: MultiLayerEagleDraftExtendCudaGraphRunner._replay method
```python
    def _replay(self, forward_batch: ForwardBatch):
        self.graphs[self.bs].replay()
```
**EN:** This block uses `MultiLayerEagleDraftExtendCudaGraphRunner._replay` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleDraftExtendCudaGraphRunner._replay` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 295-296: MultiLayerEagleDraftExtendCudaGraphRunner.capture method
```python
    def capture(self):
        CudaGraphRunner.capture(self)
```
**EN:** This block uses `MultiLayerEagleDraftExtendCudaGraphRunner.capture` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleDraftExtendCudaGraphRunner.capture` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 298-357: MultiLayerEagleDraftExtendCudaGraphRunner.get_forward_batch method (part 1/2)
```python
    def get_forward_batch(self, bs: int) -> ForwardBatch:
        buffers = self.buffers
        num_tokens = bs * self.num_tokens_per_bs

        # Graph inputs
        input_ids = buffers.input_ids[:num_tokens]
        req_pool_indices = buffers.req_pool_indices[:bs]
        seq_lens = buffers.seq_lens[:bs]
        seq_lens_cpu = buffers.seq_lens_cpu[:bs]
        extend_seq_lens = buffers.extend_seq_lens[:bs]
        extend_seq_lens_cpu = self.extend_seq_lens_cpu[:bs]
        extend_start_loc = buffers.extend_start_loc[:bs]
        num_correct_drafts = buffers.num_correct_drafts[:bs]
        num_accept_tokens = buffers.num_accept_tokens[:bs]
        out_cache_loc = buffers.out_cache_loc[:num_tokens]
        positions = buffers.positions[:num_tokens]
        mrope_positions = buffers.mrope_positions[:, :num_tokens]
        hidden_states = buffers.hidden_states[:num_tokens]
        next_token_logits_buffer = buffers.next_token_logits_buffer[
            : bs if self.forward_mode == ForwardMode.DRAFT_EXTEND else num_tokens
        ]

        if self.require_mlp_tp_gather:
            buffers.global_num_tokens_gpu.copy_(
                torch.tensor(
                    [num_tokens] * self.dp_size,
                    dtype=torch.int32,
                    device=buffers.input_ids.device,
                )
            )
            buffers.global_num_tokens_for_logprob_gpu.copy_(
                torch.tensor(
                    [num_tokens] * self.dp_size,
                    dtype=torch.int32,
                    device=buffers.input_ids.device,
                )
            )
            global_dp_buffer_len = num_tokens * self.dp_size
        elif self.require_attn_tp_gather:
            buffers.global_num_tokens_gpu.copy_(
                torch.tensor(
                    [num_tokens],
                    dtype=torch.int32,
                    device=buffers.input_ids.device,
                )
            )
            buffers.global_num_tokens_for_logprob_gpu.copy_(
                torch.tensor(
                    [bs],
                    dtype=torch.int32,
                    device=buffers.input_ids.device,
                )
            )
            global_dp_buffer_len = num_tokens
        else:
            global_dp_buffer_len = None

        spec_info = EagleDraftExtendInput(
            hidden_states=hidden_states,
            num_correct_drafts=num_correct_drafts,
```
**EN:** This block uses `MultiLayerEagleDraftExtendCudaGraphRunner.get_forward_batch` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `MultiLayerEagleDraftExtendCudaGraphRunner.get_forward_batch` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 358-401: MultiLayerEagleDraftExtendCudaGraphRunner.get_forward_batch method (part 2/2)
```python
            num_accept_tokens=num_accept_tokens,
        )
        spec_info.positions = None

        capture_mode = (
            CaptureHiddenMode.NULL
            if self.model_runner.spec_algorithm.is_standalone()
            else CaptureHiddenMode.FULL
        )

        # Forward batch
        forward_batch = ForwardBatch(
            forward_mode=self.forward_mode,
            batch_size=bs,
            input_ids=input_ids,
            req_pool_indices=req_pool_indices,
            seq_lens=seq_lens,
            seq_lens_cpu=seq_lens_cpu,
            next_token_logits_buffer=next_token_logits_buffer,
            req_to_token_pool=self.model_runner.req_to_token_pool,
            token_to_kv_pool=self.model_runner.token_to_kv_pool,
            out_cache_loc=out_cache_loc,
            seq_lens_sum=seq_lens.sum().item(),
            return_logprob=False,
            positions=positions,
            mrope_positions=mrope_positions,
            global_num_tokens_gpu=buffers.global_num_tokens_gpu,
            global_num_tokens_for_logprob_gpu=buffers.global_num_tokens_for_logprob_gpu,
            dp_padding_mode=DpPaddingMode.get_default_mode_in_cuda_graph(),
            global_dp_buffer_len=global_dp_buffer_len,
            spec_algorithm=self.model_runner.spec_algorithm,
            spec_info=spec_info,
            capture_hidden_mode=capture_mode,
            attn_backend=self.eagle_worker.draft_extend_attn_backend_list[self.step],
            extend_seq_lens=extend_seq_lens,
            extend_seq_lens_cpu=extend_seq_lens_cpu,
            padded_static_len=self.padded_static_len,
            # added args
            extend_start_loc=extend_start_loc,
            extend_num_tokens=self.num_tokens_per_bs * bs,
            num_token_non_padded_cpu=self.num_tokens_per_bs * bs,
            return_hidden_states_before_norm=True,
        )
        return forward_batch
```
**EN:** This block uses `MultiLayerEagleDraftExtendCudaGraphRunner.get_forward_batch` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `MultiLayerEagleDraftExtendCudaGraphRunner.get_forward_batch` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 403-462: MultiLayerEagleDraftExtendCudaGraphRunner.capture_one_batch_size method (part 1/2)
```python
    def capture_one_batch_size(self, bs: int, forward: Callable, stream_idx: int = 0):
        buffers = self.buffers
        graph = self._create_graph()
        stream = self.stream

        self.deepep_adapter.capture(is_extend_in_batch=True)

        num_tokens = bs * self.num_tokens_per_bs
        forward_batch = self.get_forward_batch(bs)

        self.eagle_worker.draft_extend_attn_backend_list[
            self.step
        ].init_forward_metadata_capture_cuda_graph(
            bs=bs,
            num_tokens=num_tokens,
            req_pool_indices=forward_batch.req_pool_indices,
            seq_lens=forward_batch.seq_lens,
            encoder_lens=None,
            forward_mode=self.forward_mode,
            spec_info=forward_batch.spec_info,
        )

        # Run and capture
        def run_once():
            # Clean intermediate result cache for DP attention
            forward_batch.dp_local_start_pos = forward_batch.dp_local_num_tokens = None
            set_dp_buffer_len(
                forward_batch.global_dp_buffer_len,
                num_tokens,
                forward_batch.dp_padding_mode.is_max_len(),
            )
            set_is_extend_in_batch(False)

            # Backup two fields, which will be modified in-place in `draft_forward`.
            output_cache_loc_backup = forward_batch.out_cache_loc
            hidden_states_backup = forward_batch.spec_info.hidden_states

            ret = self.model_runner.model.forward(
                forward_batch.input_ids,
                forward_batch.positions,
                forward_batch,
            )

            # Chain-style MTP: overwrite buffers.hidden_states with the draft model's
            # output (hidden_states_before_norm) so that assign_new_state_triton
            # propagates each MTP layer's own output to the next MTP layer,
            # rather than always feeding the target model's hidden states.
            if (
                self.eagle_worker.chain_mtp_hidden_states
                and ret.hidden_states is not None
            ):
                buffers.hidden_states[:num_tokens].copy_(ret.hidden_states[:num_tokens])

            # num_correct_drafts is drafts-only; the last accepted draft sits at index
            # `num_correct_drafts` within the (current_token + drafts) slot range.
            select_index = (
                torch.arange(bs, device=self.model_runner.device)
                * (self.speculative_num_draft_tokens + self.step)
                + buffers.num_correct_drafts[:bs]
                + self.step
```
**EN:** This block uses `MultiLayerEagleDraftExtendCudaGraphRunner.capture_one_batch_size` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `MultiLayerEagleDraftExtendCudaGraphRunner.capture_one_batch_size` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 463-514: MultiLayerEagleDraftExtendCudaGraphRunner.capture_one_batch_size method (part 2/2)
```python
            )

            probs = torch.softmax(ret.next_token_logits[select_index], dim=-1)
            ret.topk_p, ret.topk_index = fast_topk(probs, self.topk, dim=-1)

            if self.next_cuda_graph_runner is not None:
                next_buffers = self.next_cuda_graph_runner.buffers
                # rejected drafts = proposed drafts - accepted drafts.
                # speculative_num_draft_tokens includes the current-token slot, so -1.
                padding_lens = (
                    self.speculative_num_draft_tokens - 1
                ) - buffers.num_correct_drafts[:bs]
                assign_new_state_triton(
                    ret.topk_index,
                    buffers.input_ids,
                    buffers.positions,
                    buffers.hidden_states,
                    buffers.out_cache_loc,
                    buffers.extend_seq_lens,
                    buffers.extend_start_loc,
                    next_buffers.input_ids,
                    next_buffers.positions,
                    next_buffers.hidden_states,
                    next_buffers.out_cache_loc,
                    next_buffers.extend_seq_lens,
                    next_buffers.extend_start_loc,
                    next_buffers.seq_lens,
                    padding_lens,
                    forward_batch.batch_size,
                    self.step,
                    forward_batch.req_pool_indices,
                    forward_batch.req_to_token_pool.req_to_token,
                    self.eagle_worker.req_to_hidden_states_pool,
                )
                next_buffers.swa_out_cache_loc.copy_(
                    self.model_runner.token_to_kv_pool.translate_loc_from_full_to_swa(
                        next_buffers.out_cache_loc
                    )
                )

            forward_batch.out_cache_loc = output_cache_loc_backup
            forward_batch.spec_info.hidden_states = hidden_states_backup
            return ret

        self._capture_init(run_once)

        out = self._capture_graph(
            graph, get_global_graph_memory_pool(), stream, run_once
        )

        set_global_graph_memory_pool(graph.pool())
        return graph, out
```
**EN:** This block uses `MultiLayerEagleDraftExtendCudaGraphRunner.capture_one_batch_size` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `MultiLayerEagleDraftExtendCudaGraphRunner.capture_one_batch_size` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 516-550: MultiLayerEagleDraftExtendCudaGraphRunner.init_replay_state method
```python
    def init_replay_state(
        self, forward_batch: ForwardBatch, bs: int, raw_bs: int, num_tokens: int
    ):
        buffers = self.buffers
        # Common inputs
        buffers.input_ids[:num_tokens].copy_(forward_batch.input_ids)
        buffers.seq_lens[:raw_bs].copy_(forward_batch.seq_lens)
        if forward_batch.extend_seq_lens is not None:
            buffers.extend_seq_lens[:raw_bs].copy_(forward_batch.extend_seq_lens)
            buffers.extend_start_loc[:raw_bs].copy_(forward_batch.extend_start_loc)
        buffers.out_cache_loc[:num_tokens].copy_(forward_batch.out_cache_loc)
        buffers.positions[:num_tokens].copy_(forward_batch.positions)
        if (
            forward_batch.spec_info.hidden_states.shape[1]
            == buffers.hidden_states.shape[1]
        ):
            buffers.hidden_states[:num_tokens].copy_(
                forward_batch.spec_info.hidden_states
            )
        if forward_batch.spec_info.num_correct_drafts is not None:
            buffers.num_correct_drafts[:raw_bs].copy_(
                forward_batch.spec_info.num_correct_drafts
            )
            buffers.num_accept_tokens[:raw_bs].copy_(
                forward_batch.spec_info.num_accept_tokens
            )
        buffers.req_pool_indices[:raw_bs].copy_(forward_batch.req_pool_indices)

        if forward_batch.seq_lens_cpu is not None:
            if bs != raw_bs:
                buffers.seq_lens_cpu.fill_(self.seq_len_fill_value)
            buffers.seq_lens_cpu[:raw_bs].copy_(forward_batch.seq_lens_cpu)

        if forward_batch.extend_seq_lens_cpu is not None:
            self.extend_seq_lens_cpu[:raw_bs] = forward_batch.extend_seq_lens_cpu
```
**EN:** This block uses `MultiLayerEagleDraftExtendCudaGraphRunner.init_replay_state` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleDraftExtendCudaGraphRunner.init_replay_state` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 552-611: MultiLayerEagleDraftExtendCudaGraphRunner.replay method (part 1/2)
```python
    def replay(self, forward_batch: ForwardBatch, init_state: bool = True):
        assert forward_batch.out_cache_loc is not None
        self.deepep_adapter.replay()
        buffers = self.buffers

        # batch_size and num_seqs can be different in case there are finished examples
        # in the batch, which will not be counted as num_seqs
        raw_bs = forward_batch.batch_size
        num_tokens = raw_bs * self.num_tokens_per_bs
        # num_tokens = forward_batch.input_ids.shape[0]
        if self.require_mlp_tp_gather:
            max_batch_size = max(forward_batch.original_global_num_tokens_cpu)
            index = bisect.bisect_left(self.capture_bs, max_batch_size)
        else:
            index = bisect.bisect_left(self.capture_bs, raw_bs)

        bs = self.capture_bs[index]

        if init_state:
            self.init_replay_state(forward_batch, bs, raw_bs, num_tokens)

        if self.require_gathered_buffer:
            buffers.global_num_tokens_gpu.fill_(bs * self.num_tokens_per_bs)
            buffers.global_num_tokens_for_logprob_gpu.fill_(bs * self.num_tokens_per_bs)

        forward_batch.spec_info.hidden_states = buffers.hidden_states[:num_tokens]
        forward_batch.spec_info.num_correct_drafts = buffers.num_correct_drafts[:bs]
        forward_batch.spec_info.num_accept_tokens = buffers.num_accept_tokens[:bs]
        forward_batch.spec_info.num_tokens_per_req = self.num_tokens_per_bs
        forward_batch.spec_info.num_tokens_for_logprob_per_req = 1
        forward_batch.spec_info.positions = buffers.positions[:num_tokens]
        forward_batch.spec_info.extend_seq_lens_tensor = buffers.extend_seq_lens[:bs]

        self.eagle_worker.draft_extend_attn_backend_list[
            self.step
        ].init_forward_metadata_replay_cuda_graph(
            bs=bs,
            req_pool_indices=buffers.req_pool_indices,
            seq_lens=buffers.seq_lens,
            seq_lens_sum=forward_batch.seq_lens_sum
            + (bs - raw_bs) * self.seq_len_fill_value,
            encoder_lens=None,
            forward_mode=self.forward_mode,
            spec_info=forward_batch.spec_info,
            seq_lens_cpu=buffers.seq_lens_cpu,
        )

        # Replay
        self.raw_bs = raw_bs
        self.bs = bs
        self._replay(forward_batch)
        out = self.output_buffers[bs]

        if self.forward_mode == ForwardMode.DRAFT_EXTEND_V2:
            # DRAFT_EXTEND_V2: all tokens calculations whether accepted or not.
            unpadding_bs = num_tokens
        elif bs != raw_bs:
            forward_batch.spec_info.num_correct_drafts = buffers.num_correct_drafts[
                :raw_bs
            ]
```
**EN:** This block uses `MultiLayerEagleDraftExtendCudaGraphRunner.replay` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `MultiLayerEagleDraftExtendCudaGraphRunner.replay` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 612-627: MultiLayerEagleDraftExtendCudaGraphRunner.replay method (part 2/2)
```python
            forward_batch.spec_info.num_accept_tokens = buffers.num_accept_tokens[
                :raw_bs
            ]
            unpadding_bs = raw_bs
        else:
            unpadding_bs = None

        if unpadding_bs is not None:
            out_copy = out
            out = LogitsProcessorOutput(
                next_token_logits=out.next_token_logits[:unpadding_bs],
                hidden_states=out.hidden_states[:unpadding_bs],
            )
            out.topk_p = out_copy.topk_p[:raw_bs]
            out.topk_index = out_copy.topk_index[:raw_bs]
        return out
```
**EN:** This block uses `MultiLayerEagleDraftExtendCudaGraphRunner.replay` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `MultiLayerEagleDraftExtendCudaGraphRunner.replay` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 630-630: MultiLayerEagleMultiStepDraftExtendCudaGraphRunner class declaration
```python
class MultiLayerEagleMultiStepDraftExtendCudaGraphRunner:
```
**EN:** This block declares the `MultiLayerEagleMultiStepDraftExtendCudaGraphRunner` class, which exists to produce draft tokens or draft-side state. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `MultiLayerEagleMultiStepDraftExtendCudaGraphRunner` 类，其职责是生成草稿 token 或草稿侧状态。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 631-646: MultiLayerEagleMultiStepDraftExtendCudaGraphRunner initializer
```python
    def __init__(self, eagle_worker: MultiLayerEagleDraftWorker):
        self.eagle_worker = eagle_worker
        self.device = eagle_worker.device
        self.gpu_id = eagle_worker.gpu_id
        self.speculative_num_steps = eagle_worker.speculative_num_steps
        self.draft_extend_attn_backend_list = (
            eagle_worker.draft_extend_attn_backend_list
        )

        self.runners = []
        self.cuda_graph_buffers = {}
        self.seq_len_fill_value = 1
        self.max_bs = 1
        self.offsets = [0]

        self._init_and_capture()
```
**EN:** This block initializes the `MultiLayerEagleMultiStepDraftExtendCudaGraphRunner` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `MultiLayerEagleMultiStepDraftExtendCudaGraphRunner` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 648-707: MultiLayerEagleMultiStepDraftExtendCudaGraphRunner._init_and_capture method (part 1/2)
```python
    def _init_and_capture(self):
        if self.eagle_worker.server_args.disable_cuda_graph:
            self.runners = [None] * self.speculative_num_steps
            return

        self.runners: List[Optional[MultiLayerEagleDraftExtendCudaGraphRunner]] = []
        buffer_len_list: List[int] = []

        # 1. Capture loop
        for step in range(self.speculative_num_steps):
            if self.draft_extend_attn_backend_list[step]:
                runner = MultiLayerEagleDraftExtendCudaGraphRunner(
                    self.eagle_worker, step
                )
                self.runners.append(runner)

                self.seq_len_fill_value = runner.seq_len_fill_value
                self.max_bs = runner.max_bs
                buffer_len_list.append(runner.max_num_token)
                self.offsets.append(self.offsets[-1] + runner.max_num_token)
            else:
                self.runners.append(None)

        # 2. Allocate buffers
        self.cuda_graph_buffers["seq_lens_cpu"] = torch.full(
            (self.max_bs,),
            self.seq_len_fill_value,
            dtype=torch.int32,
        )

        with torch.device(self.device):
            # Sliced buffers
            self.cuda_graph_buffers["input_ids"] = torch.zeros(
                (self.offsets[-1],), dtype=torch.int64
            )
            self.cuda_graph_buffers["out_cache_loc"] = torch.ones(
                (self.offsets[-1],), dtype=torch.int64
            )
            self.cuda_graph_buffers["swa_out_cache_loc"] = torch.ones(
                (self.offsets[-1],), dtype=torch.int64
            )
            self.cuda_graph_buffers["positions"] = torch.zeros(
                (self.offsets[-1],), dtype=torch.int64
            )

            # Shared states
            self.cuda_graph_buffers["seq_lens"] = torch.full(
                (self.max_bs,),
                self.seq_len_fill_value,
                dtype=torch.int32,
            )
            self.cuda_graph_buffers["req_pool_indices"] = torch.zeros(
                (self.max_bs,), dtype=torch.int64
            )
            self.cuda_graph_buffers["num_correct_drafts"] = torch.full(
                (self.max_bs,), 1, dtype=torch.int32
            )
            self.cuda_graph_buffers["num_accept_tokens"] = torch.full(
                (self.max_bs,), 1, dtype=torch.int32
            )
```
**EN:** This block uses `MultiLayerEagleMultiStepDraftExtendCudaGraphRunner._init_and_capture` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `MultiLayerEagleMultiStepDraftExtendCudaGraphRunner._init_and_capture` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 708-730: MultiLayerEagleMultiStepDraftExtendCudaGraphRunner._init_and_capture method (part 2/2)
```python

        for step in range(self.speculative_num_steps - 1, -1, -1):
            if self.runners[step] is not None:
                tic = time.perf_counter()
                before_mem = get_available_gpu_memory(self.device, self.gpu_id)
                logger.info(
                    f"Capture draft extend cuda graph begin (step {step}). This can take up to several minutes. avail mem={before_mem:.2f} GB"
                )

                self.runners[step].init_buffers_and_capture(
                    self.cuda_graph_buffers,
                    self.offsets[step],
                    (
                        self.runners[step + 1]
                        if step + 1 < self.speculative_num_steps
                        else None
                    ),
                )

                after_mem = get_available_gpu_memory(self.device, self.gpu_id)
                logger.info(
                    f"Capture draft extend cuda graph end. Time elapsed: {time.perf_counter() - tic:.2f} s. mem usage={(before_mem - after_mem):.2f} GB. avail mem={after_mem:.2f} GB."
                )
```
**EN:** This block uses `MultiLayerEagleMultiStepDraftExtendCudaGraphRunner._init_and_capture` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `MultiLayerEagleMultiStepDraftExtendCudaGraphRunner._init_and_capture` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 732-745: MultiLayerEagleMultiStepDraftExtendCudaGraphRunner.reset_buffers method
```python
    def reset_buffers(self, forward_batch, batch_result):
        self.cuda_graph_buffers["input_ids"].zero_()
        self.cuda_graph_buffers["seq_lens"].fill_(self.seq_len_fill_value)
        self.cuda_graph_buffers["out_cache_loc"].zero_()
        self.cuda_graph_buffers["swa_out_cache_loc"].zero_()
        self.cuda_graph_buffers["positions"].zero_()
        # `batch_result.accept_lens` is drafts + bonus.
        bs = forward_batch.batch_size
        self.cuda_graph_buffers["num_correct_drafts"][:bs].copy_(
            batch_result.accept_lens - 1
        )
        self.cuda_graph_buffers["num_accept_tokens"][:bs].copy_(
            batch_result.accept_lens
        )
```
**EN:** This block uses `MultiLayerEagleMultiStepDraftExtendCudaGraphRunner.reset_buffers` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleMultiStepDraftExtendCudaGraphRunner.reset_buffers` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 747-748: MultiLayerEagleMultiStepDraftExtendCudaGraphRunner.get_runner method
```python
    def get_runner(self, step):
        return self.runners[step]
```
**EN:** This block uses `MultiLayerEagleMultiStepDraftExtendCudaGraphRunner.get_runner` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleMultiStepDraftExtendCudaGraphRunner.get_runner` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 750-751: MultiLayerEagleMultiStepDraftExtendCudaGraphRunner.get_last_runner method
```python
    def get_last_runner(self):
        return self.runners[-1] if self.runners else None
```
**EN:** This block uses `MultiLayerEagleMultiStepDraftExtendCudaGraphRunner.get_last_runner` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleMultiStepDraftExtendCudaGraphRunner.get_last_runner` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 753-754: MultiLayerEagleMultiStepDraftExtendCudaGraphRunner.can_run method
```python
    def can_run(self, forward_batch):
        return self.runners[0].can_run(forward_batch)
```
**EN:** This block uses `MultiLayerEagleMultiStepDraftExtendCudaGraphRunner.can_run` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleMultiStepDraftExtendCudaGraphRunner.can_run` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- Speculative decoding / 推测解码
- EAGLE draft pipeline / EAGLE 草稿流水线
- CUDA Graph execution / CUDA Graph 执行
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算
- Graph-captured execution paths / 图捕获执行路径

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.layers.dp_attention`
- `sglang.srt.model_executor.cuda_graph_runner`
- `sglang.srt.model_executor.forward_batch_info`
- `sglang.srt.model_executor.input_buffers`
- `sglang.srt.speculative.eagle_info`
- `sglang.srt.speculative.multi_layer_eagle_utils`
- `sglang.srt.speculative.multi_layer_eagle_worker_v2`
- `sglang.srt.speculative.spec_utils`
- `sglang.srt.utils`
### External / 外部
- `__future__`
- `torch`
- `bisect` (stdlib)
- `dataclasses` (stdlib)
- `logging` (stdlib)
- `time` (stdlib)
- `typing` (stdlib)
