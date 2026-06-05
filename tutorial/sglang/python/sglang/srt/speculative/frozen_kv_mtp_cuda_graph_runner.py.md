# frozen_kv_mtp_cuda_graph_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/frozen_kv_mtp_cuda_graph_runner.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements speculative decoding infrastructure, such as workers, metadata objects, CUDA-graph runners, or shared utilities. It coordinates draft-model execution with target-model verification to improve throughput. / 该文件实现了推测解码基础设施，例如 worker、元数据对象、CUDA Graph 运行器或共享工具函数。它通过协调草稿模型执行与目标模型校验来提升吞吐。

## Line-by-Line Analysis / 逐行分析

### Lines 1-38: Module header, imports, and shared constants
```python
from __future__ import annotations

import bisect
from dataclasses import dataclass
from typing import TYPE_CHECKING, Callable, Optional

import torch

from sglang.srt.layers.dp_attention import DpPaddingMode, set_dp_buffer_len
from sglang.srt.model_executor.cuda_graph_runner import (
    CUDA_GRAPH_CAPTURE_FAILED_MSG,
    CudaGraphRunner,
    DeepEPCudaGraphRunnerAdapter,
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
from sglang.srt.speculative.frozen_kv_mtp_info import FrozenKVMTPDraftInput
from sglang.srt.utils import (
    require_attn_tp_gather,
    require_gathered_buffer,
    require_mlp_sync,
    require_mlp_tp_gather,
)

if TYPE_CHECKING:
    from sglang.srt.speculative.frozen_kv_mtp_worker import FrozenKVMTPWorker
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 40-50: FrozenKVMTPInputBuffers class declaration
```python
class FrozenKVMTPInputBuffers(ForwardInputBuffers):
    req_pool_indices: torch.Tensor
    positions: torch.Tensor
    mrope_positions: torch.Tensor
    seq_lens: torch.Tensor
    seq_lens_cpu: torch.Tensor
    topk_p: torch.Tensor
    topk_index: torch.Tensor
    hidden_states: torch.Tensor
    global_num_tokens_gpu: Optional[torch.Tensor]
    global_num_tokens_for_logprob_gpu: Optional[torch.Tensor]
```
**EN:** This block declares the `FrozenKVMTPInputBuffers` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `FrozenKVMTPInputBuffers` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 53-55: FrozenKVMTPCudaGraphRunner class declaration
```python
class FrozenKVMTPCudaGraphRunner:
    """CUDA graph runner for the Frozen-KV MTP recurrent draft-loop step."""
```
**EN:** This block declares the `FrozenKVMTPCudaGraphRunner` class, which exists to coordinate staged runtime execution. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `FrozenKVMTPCudaGraphRunner` 类，其职责是协调分阶段运行时执行。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 56-115: FrozenKVMTPCudaGraphRunner initializer (part 1/2)
```python
    def __init__(self, frozen_kv_mtp_worker: FrozenKVMTPWorker):
        self.frozen_kv_mtp_worker = frozen_kv_mtp_worker
        self.model_runner = model_runner = frozen_kv_mtp_worker.draft_model_runner
        self.graphs = {}
        self.output_buffers = {}
        self.enable_torch_compile = model_runner.server_args.enable_torch_compile
        self.disable_padding = model_runner.server_args.disable_cuda_graph_padding
        self.require_gathered_buffer = require_gathered_buffer(model_runner.server_args)
        self.require_mlp_tp_gather = require_mlp_tp_gather(model_runner.server_args)
        self.require_mlp_sync = require_mlp_sync(model_runner.server_args)
        self.require_attn_tp_gather = require_attn_tp_gather(model_runner.server_args)
        self.tp_size = self.model_runner.tp_size
        self.dp_size = self.model_runner.dp_size
        self.speculative_num_steps = model_runner.server_args.speculative_num_steps
        self.topk = model_runner.server_args.speculative_eagle_topk
        self.draft_attn_backend = frozen_kv_mtp_worker.draft_attn_backend
        self.enable_profile_cuda_graph = (
            model_runner.server_args.enable_profile_cuda_graph
        )
        self.enable_pdmux = False
        self.deepep_adapter = DeepEPCudaGraphRunnerAdapter()

        self.num_tokens_per_bs = self.topk
        self.capture_bs, self.compile_bs = get_batch_sizes_to_capture(
            model_runner, self.num_tokens_per_bs
        )
        self.max_bs = max(self.capture_bs)
        self.max_num_token = self.max_bs * self.num_tokens_per_bs

        self.draft_attn_backend.init_cuda_graph_state(self.max_bs, self.max_num_token)
        self.seq_len_fill_value = (
            self.draft_attn_backend.get_cuda_graph_seq_len_fill_value()
        )
        seq_lens_cpu = torch.full(
            (self.max_num_token,), self.seq_len_fill_value, dtype=torch.int32
        )

        if self.enable_torch_compile:
            set_torch_compile_config()

        with torch.device(model_runner.device):
            req_pool_indices = torch.zeros((self.max_num_token,), dtype=torch.int64)
            positions = torch.zeros((self.max_num_token,), dtype=torch.int64)
            mrope_positions = torch.zeros((3, self.max_num_token), dtype=torch.int64)
            seq_lens = torch.full(
                (self.max_num_token,), self.seq_len_fill_value, dtype=torch.int32
            )
            topk_p = torch.zeros((self.max_bs, self.topk), dtype=torch.float32)
            topk_index = torch.zeros((self.max_bs, self.topk), dtype=torch.int64)
            hidden_states = torch.zeros(
                (self.max_bs, frozen_kv_mtp_worker._recurrent_hidden_size),
                dtype=self.model_runner.dtype,
            )

            if self.require_gathered_buffer:
                if self.require_mlp_tp_gather:
                    global_num_tokens_gpu = torch.zeros(
                        (self.dp_size,), dtype=torch.int32
                    )
                    global_num_tokens_for_logprob_gpu = torch.zeros(
```
**EN:** This block initializes the `FrozenKVMTPCudaGraphRunner` object, setting up the state, buffers, and references that later methods rely on. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块初始化 `FrozenKVMTPCudaGraphRunner` 对象，建立后续方法依赖的状态、缓冲区与引用。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 116-149: FrozenKVMTPCudaGraphRunner initializer (part 2/2)
```python
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

        self.buffers = FrozenKVMTPInputBuffers(
            req_pool_indices=req_pool_indices,
            positions=positions,
            mrope_positions=mrope_positions,
            seq_lens=seq_lens,
            seq_lens_cpu=seq_lens_cpu,
            topk_p=topk_p,
            topk_index=topk_index,
            hidden_states=hidden_states,
            global_num_tokens_gpu=global_num_tokens_gpu,
            global_num_tokens_for_logprob_gpu=global_num_tokens_for_logprob_gpu,
        )
        self.buffers.share_buffers()

        try:
            with model_capture_mode():
                self.capture()
        except RuntimeError as e:
            raise Exception(
                f"Capture frozen-KV MTP cuda graph failed: {e}\n"
                f"{CUDA_GRAPH_CAPTURE_FAILED_MSG}"
            )
```
**EN:** This block initializes the `FrozenKVMTPCudaGraphRunner` object, setting up the state, buffers, and references that later methods rely on. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块初始化 `FrozenKVMTPCudaGraphRunner` 对象，建立后续方法依赖的状态、缓冲区与引用。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 151-170: FrozenKVMTPCudaGraphRunner.can_run method
```python
    def can_run(self, forward_batch: ForwardBatch):
        if self.require_mlp_tp_gather:
            cuda_graph_bs = max(forward_batch.global_num_tokens_cpu) // (
                self.topk * self.topk
            )
        else:
            cuda_graph_bs = (
                forward_batch.batch_size // self.topk
                if self.topk > 1
                else forward_batch.batch_size
            )

        is_bs_supported = (
            cuda_graph_bs in self.graphs
            if self.disable_padding
            else cuda_graph_bs <= self.max_bs
        )
        if self.require_mlp_sync:
            is_bs_supported = is_bs_supported and forward_batch.can_run_dp_cuda_graph
        return is_bs_supported
```
**EN:** This block uses `FrozenKVMTPCudaGraphRunner.can_run` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPCudaGraphRunner.can_run` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 172-173: FrozenKVMTPCudaGraphRunner._create_graph method
```python
    def _create_graph(self):
        return torch.cuda.CUDAGraph()
```
**EN:** This block uses `FrozenKVMTPCudaGraphRunner._create_graph` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPCudaGraphRunner._create_graph` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 175-179: FrozenKVMTPCudaGraphRunner._capture_init method
```python
    def _capture_init(self, run_once_fn):
        for _ in range(2):
            torch.cuda.synchronize()
            self.model_runner.tp_group.barrier()
            run_once_fn()
```
**EN:** This block uses `FrozenKVMTPCudaGraphRunner._capture_init` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPCudaGraphRunner._capture_init` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 181-184: FrozenKVMTPCudaGraphRunner._capture_graph method
```python
    def _capture_graph(self, graph, pool, stream, run_once_fn):
        with torch.cuda.graph(graph, pool=pool, stream=stream):
            out = run_once_fn()
        return out
```
**EN:** This block uses `FrozenKVMTPCudaGraphRunner._capture_graph` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPCudaGraphRunner._capture_graph` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 186-187: FrozenKVMTPCudaGraphRunner._replay method
```python
    def _replay(self):
        self.graphs[self.bs].replay()
```
**EN:** This block uses `FrozenKVMTPCudaGraphRunner._replay` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPCudaGraphRunner._replay` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 189-190: FrozenKVMTPCudaGraphRunner.capture method
```python
    def capture(self):
        CudaGraphRunner.capture(self)
```
**EN:** This block uses `FrozenKVMTPCudaGraphRunner.capture` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPCudaGraphRunner.capture` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 192-251: FrozenKVMTPCudaGraphRunner.capture_one_batch_size method (part 1/3)
```python
    def capture_one_batch_size(
        self, num_seqs: int, forward: Callable, stream_idx: int = 0
    ):
        del forward, stream_idx
        buffers = self.buffers
        graph = self._create_graph()
        stream = self.stream
        request_bs = num_seqs
        expanded_bs = request_bs * self.num_tokens_per_bs

        req_pool_indices = buffers.req_pool_indices[:expanded_bs]
        positions = buffers.positions[:expanded_bs]
        mrope_positions = buffers.mrope_positions[:, :expanded_bs]
        seq_lens = buffers.seq_lens[:expanded_bs]
        seq_lens_cpu = buffers.seq_lens_cpu[:expanded_bs]
        topk_p = buffers.topk_p[:request_bs]
        topk_index = buffers.topk_index[:request_bs]
        hidden_states = buffers.hidden_states[:request_bs]

        if self.require_mlp_tp_gather:
            buffers.global_num_tokens_gpu.copy_(
                torch.tensor(
                    [expanded_bs] * self.dp_size,
                    dtype=torch.int32,
                    device=buffers.positions.device,
                )
            )
            buffers.global_num_tokens_for_logprob_gpu.copy_(
                torch.tensor(
                    [expanded_bs] * self.dp_size,
                    dtype=torch.int32,
                    device=buffers.positions.device,
                )
            )
            global_num_tokens = buffers.global_num_tokens_gpu
            global_num_tokens_for_logprob = buffers.global_num_tokens_for_logprob_gpu
            global_dp_buffer_len = expanded_bs * self.dp_size
        elif self.require_attn_tp_gather:
            buffers.global_num_tokens_gpu.copy_(
                torch.tensor(
                    [expanded_bs],
                    dtype=torch.int32,
                    device=buffers.positions.device,
                )
            )
            buffers.global_num_tokens_for_logprob_gpu.copy_(
                torch.tensor(
                    [expanded_bs],
                    dtype=torch.int32,
                    device=buffers.positions.device,
                )
            )
            global_num_tokens = buffers.global_num_tokens_gpu
            global_num_tokens_for_logprob = buffers.global_num_tokens_for_logprob_gpu
            global_dp_buffer_len = expanded_bs
        else:
            global_num_tokens = None
            global_num_tokens_for_logprob = None
            global_dp_buffer_len = None
```
**EN:** This block uses `FrozenKVMTPCudaGraphRunner.capture_one_batch_size` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `FrozenKVMTPCudaGraphRunner.capture_one_batch_size` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 252-311: FrozenKVMTPCudaGraphRunner.capture_one_batch_size method (part 2/3)
```python
        spec_info = FrozenKVMTPDraftInput(
            topk_p=topk_p,
            topk_index=topk_index,
            hidden_states=hidden_states,
            capture_hidden_mode=CaptureHiddenMode.LAST,
        )
        spec_info.num_tokens_per_req = self.topk
        spec_info.num_tokens_for_logprob_per_req = self.topk
        spec_info.positions = positions

        forward_batch = ForwardBatch(
            forward_mode=ForwardMode.DECODE,
            batch_size=expanded_bs,
            input_ids=None,
            req_pool_indices=req_pool_indices,
            seq_lens=seq_lens,
            seq_lens_cpu=seq_lens_cpu,
            req_to_token_pool=self.model_runner.req_to_token_pool,
            token_to_kv_pool=self.frozen_kv_mtp_worker.kv_context.target_token_to_kv_pool,
            attn_backend=self.draft_attn_backend,
            out_cache_loc=None,
            seq_lens_sum=seq_lens.sum().item(),
            return_logprob=False,
            positions=positions,
            mrope_positions=mrope_positions,
            global_num_tokens_gpu=global_num_tokens,
            global_num_tokens_for_logprob_gpu=global_num_tokens_for_logprob,
            dp_padding_mode=DpPaddingMode.get_default_mode_in_cuda_graph(),
            global_dp_buffer_len=global_dp_buffer_len,
            spec_algorithm=self.model_runner.spec_algorithm,
            spec_info=spec_info,
            capture_hidden_mode=CaptureHiddenMode.LAST,
        )

        self.frozen_kv_mtp_worker._init_frozen_kv_metadata_capture_cuda_graph(
            forward_batch
        )

        def run_once():
            forward_batch.dp_local_start_pos = forward_batch.dp_local_num_tokens = None
            set_dp_buffer_len(
                global_dp_buffer_len,
                expanded_bs,
                forward_batch.dp_padding_mode.is_max_len(),
            )
            set_is_extend_in_batch(False)

            hidden_states_backup = forward_batch.spec_info.hidden_states
            ret = self.frozen_kv_mtp_worker.draft_forward(
                forward_batch, skip_attn_backend_init=True
            )
            forward_batch.spec_info.hidden_states = hidden_states_backup
            return ret

        self.deepep_adapter.capture(is_extend_in_batch=False)
        self._capture_init(run_once)
        out = self._capture_graph(
            graph, get_global_graph_memory_pool(), stream, run_once
        )
        set_global_graph_memory_pool(graph.pool())
```
**EN:** This block uses `FrozenKVMTPCudaGraphRunner.capture_one_batch_size` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk continues the routine's middle-stage logic and data movement.
**CN:** 该代码块通过 `FrozenKVMTPCudaGraphRunner.capture_one_batch_size` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分继续展示例程中段的逻辑与数据流动。

### Lines 312-312: FrozenKVMTPCudaGraphRunner.capture_one_batch_size method (part 3/3)
```python
        return graph, out
```
**EN:** This block uses `FrozenKVMTPCudaGraphRunner.capture_one_batch_size` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `FrozenKVMTPCudaGraphRunner.capture_one_batch_size` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 314-316: FrozenKVMTPCudaGraphRunner._postprocess_output_to_raw_bs method
```python
    def _postprocess_output_to_raw_bs(self, out, raw_bs):
        parent_list, top_scores_index, draft_tokens = (t[:raw_bs] for t in out)
        return parent_list, top_scores_index, draft_tokens
```
**EN:** This block uses `FrozenKVMTPCudaGraphRunner._postprocess_output_to_raw_bs` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPCudaGraphRunner._postprocess_output_to_raw_bs` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 318-377: FrozenKVMTPCudaGraphRunner.replay method (part 1/2)
```python
    def replay(self, forward_batch: ForwardBatch):
        self.deepep_adapter.replay()
        buffers = self.buffers

        raw_expanded_bs = forward_batch.batch_size
        raw_bs = (
            raw_expanded_bs // self.num_tokens_per_bs
            if self.topk > 1
            else raw_expanded_bs
        )
        raw_num_token = raw_expanded_bs

        if self.require_mlp_tp_gather:
            max_num_tokens = max(forward_batch.global_num_tokens_cpu)
            max_batch_size = max_num_tokens // (
                self.num_tokens_per_bs * self.num_tokens_per_bs
            )
            index = bisect.bisect_left(self.capture_bs, max_batch_size)
        else:
            index = bisect.bisect_left(self.capture_bs, raw_bs)

        bs = self.capture_bs[index]
        expanded_bs = bs * self.num_tokens_per_bs
        if bs != raw_bs:
            buffers.seq_lens.fill_(self.seq_len_fill_value)
            buffers.positions.zero_()

        num_tokens = expanded_bs
        buffers.seq_lens[:raw_expanded_bs].copy_(forward_batch.seq_lens)
        buffers.positions[:raw_num_token].copy_(forward_batch.positions)
        if forward_batch.mrope_positions is not None:
            buffers.mrope_positions[:, :raw_num_token].copy_(
                forward_batch.mrope_positions
            )
        buffers.topk_p[:raw_bs].copy_(forward_batch.spec_info.topk_p)
        buffers.topk_index[:raw_bs].copy_(forward_batch.spec_info.topk_index)
        buffers.hidden_states[:raw_bs].copy_(forward_batch.spec_info.hidden_states)
        buffers.req_pool_indices[:raw_expanded_bs].copy_(forward_batch.req_pool_indices)

        if self.require_gathered_buffer:
            buffers.global_num_tokens_gpu.fill_(expanded_bs)
            buffers.global_num_tokens_for_logprob_gpu.fill_(expanded_bs)

        if bs != raw_bs:
            forward_batch.batch_size = expanded_bs
            forward_batch.seq_lens = buffers.seq_lens[:expanded_bs]
            forward_batch.req_pool_indices = buffers.req_pool_indices[:expanded_bs]
            forward_batch.positions = buffers.positions[:num_tokens]
            if forward_batch.mrope_positions is not None:
                forward_batch.mrope_positions = buffers.mrope_positions[:, :num_tokens]

        if forward_batch.seq_lens_cpu is not None:
            if bs != raw_bs:
                buffers.seq_lens_cpu.fill_(self.seq_len_fill_value)
            buffers.seq_lens_cpu[:raw_expanded_bs].copy_(forward_batch.seq_lens_cpu)
            forward_batch.seq_lens_cpu = buffers.seq_lens_cpu[:expanded_bs]

        self.frozen_kv_mtp_worker._init_frozen_kv_metadata_replay_cuda_graph(
            forward_batch,
            expanded_bs,
```
**EN:** This block uses `FrozenKVMTPCudaGraphRunner.replay` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `FrozenKVMTPCudaGraphRunner.replay` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 378-400: FrozenKVMTPCudaGraphRunner.replay method (part 2/2)
```python
            forward_batch.seq_lens_sum
            + (expanded_bs - raw_expanded_bs) * self.seq_len_fill_value,
        )

        self.raw_bs = raw_bs
        self.bs = bs
        self._replay()
        out = self.output_buffers[bs]

        if bs != raw_bs:
            out = self._postprocess_output_to_raw_bs(out, raw_bs)
            forward_batch.batch_size = raw_expanded_bs
            forward_batch.positions = buffers.positions[:raw_num_token]
            forward_batch.seq_lens = buffers.seq_lens[:raw_expanded_bs]
            forward_batch.req_pool_indices = buffers.req_pool_indices[:raw_expanded_bs]
            if forward_batch.mrope_positions is not None:
                forward_batch.mrope_positions = buffers.mrope_positions[
                    :, :raw_num_token
                ]
            if forward_batch.seq_lens_cpu is not None:
                forward_batch.seq_lens_cpu = buffers.seq_lens_cpu[:raw_expanded_bs]

        return out
```
**EN:** This block uses `FrozenKVMTPCudaGraphRunner.replay` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `FrozenKVMTPCudaGraphRunner.replay` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

## Key Concepts / 关键概念
- Speculative decoding / 推测解码
- Frozen-KV MTP path / Frozen-KV MTP 路径
- CUDA Graph execution / CUDA Graph 执行
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算
- Graph-captured execution paths / 图捕获执行路径
- Integration with model workers / 与模型 worker 集成

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.layers.dp_attention`
- `sglang.srt.model_executor.cuda_graph_runner`
- `sglang.srt.model_executor.forward_batch_info`
- `sglang.srt.model_executor.input_buffers`
- `sglang.srt.speculative.frozen_kv_mtp_info`
- `sglang.srt.speculative.frozen_kv_mtp_worker`
- `sglang.srt.utils`
### External / 外部
- `__future__`
- `torch`
- `bisect` (stdlib)
- `dataclasses` (stdlib)
- `typing` (stdlib)
