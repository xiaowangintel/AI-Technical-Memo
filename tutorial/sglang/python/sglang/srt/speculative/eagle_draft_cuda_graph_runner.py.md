# eagle_draft_cuda_graph_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/eagle_draft_cuda_graph_runner.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements speculative decoding infrastructure, such as workers, metadata objects, CUDA-graph runners, or shared utilities. It coordinates draft-model execution with target-model verification to improve throughput. / 该文件实现了推测解码基础设施，例如 worker、元数据对象、CUDA Graph 运行器或共享工具函数。它通过协调草稿模型执行与目标模型校验来提升吞吐。

## Line-by-Line Analysis / 逐行分析

### Lines 1-43: Module header, imports, and shared constants
```python
from __future__ import annotations

import bisect
import contextlib
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
from sglang.srt.speculative.eagle_info import EagleDraftInput
from sglang.srt.speculative.spec_utils import (
    maybe_detect_nan,
    maybe_detect_oob,
)
from sglang.srt.utils import (
    require_attn_tp_gather,
    require_gathered_buffer,
    require_mlp_sync,
    require_mlp_tp_gather,
)

if TYPE_CHECKING:
    from sglang.srt.speculative.eagle_worker import EAGLEWorker
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 45-58: EagleDraftInputBuffers class declaration
```python
class EagleDraftInputBuffers(ForwardInputBuffers):
    input_ids: torch.Tensor
    req_pool_indices: torch.Tensor
    out_cache_loc: torch.Tensor
    positions: torch.Tensor
    mrope_positions: torch.Tensor
    seq_lens: torch.Tensor
    seq_lens_cpu: torch.Tensor
    extend_seq_lens: torch.Tensor
    topk_p: torch.Tensor
    topk_index: torch.Tensor
    hidden_states: Optional[torch.Tensor]
    global_num_tokens_gpu: Optional[torch.Tensor]
    global_num_tokens_for_logprob_gpu: Optional[torch.Tensor]
```
**EN:** This block declares the `EagleDraftInputBuffers` class, which exists to produce draft tokens or draft-side state. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `EagleDraftInputBuffers` 类，其职责是生成草稿 token 或草稿侧状态。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 61-61: EAGLEDraftCudaGraphRunner class declaration
```python
class EAGLEDraftCudaGraphRunner:
```
**EN:** This block declares the `EAGLEDraftCudaGraphRunner` class, which exists to coordinate staged runtime execution. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `EAGLEDraftCudaGraphRunner` 类，其职责是协调分阶段运行时执行。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 62-121: EAGLEDraftCudaGraphRunner initializer (part 1/3)
```python
    def __init__(
        self,
        eagle_worker: EAGLEWorker,
        *,
        draft_attn_backend=None,
        speculative_num_steps: Optional[int] = None,
    ):
        # Parse args
        self.eagle_worker = eagle_worker
        if not hasattr(eagle_worker, "model_runner"):
            # V2: EagleDraftWorker
            self.model_runner = model_runner = eagle_worker.draft_runner
        else:
            self.model_runner = model_runner = eagle_worker.model_runner
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
        self.speculative_num_steps = (
            model_runner.server_args.speculative_num_steps
            if speculative_num_steps is None
            else speculative_num_steps
        )
        self.topk = model_runner.server_args.speculative_eagle_topk
        self.draft_attn_backend = draft_attn_backend or model_runner.draft_attn_backend
        self.enable_profile_cuda_graph = (
            model_runner.server_args.enable_profile_cuda_graph
        )
        self.enable_pdmux = False
        self.deepep_adapter = DeepEPCudaGraphRunnerAdapter()

        # Batch sizes to capture
        self.capture_bs, self.compile_bs = get_batch_sizes_to_capture(model_runner)

        # Attention backend
        self.num_tokens_per_bs = self.topk
        self.max_bs = max(self.capture_bs)
        self.max_num_token = self.max_bs * self.num_tokens_per_bs

        self.draft_attn_backend.init_cuda_graph_state(self.max_bs, self.max_num_token)
        self.seq_len_fill_value = self.draft_attn_backend.attn_backends[
            0
        ].get_cuda_graph_seq_len_fill_value()
        seq_lens_cpu = torch.full(
            (self.max_bs,), self.seq_len_fill_value, dtype=torch.int32
        )
        self.extend_seq_lens_cpu = [self.seq_len_fill_value] * self.max_bs

        if self.enable_torch_compile:
            set_torch_compile_config()

        # Graph inputs
        with torch.device(model_runner.device):
            input_ids = torch.zeros((self.max_num_token,), dtype=torch.int64)
```
**EN:** This block initializes the `EAGLEDraftCudaGraphRunner` object, setting up the state, buffers, and references that later methods rely on. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块初始化 `EAGLEDraftCudaGraphRunner` 对象，建立后续方法依赖的状态、缓冲区与引用。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 122-181: EAGLEDraftCudaGraphRunner initializer (part 2/3)
```python
            req_pool_indices = torch.zeros((self.max_bs,), dtype=torch.int64)
            out_cache_loc = torch.zeros(
                (self.max_num_token * self.speculative_num_steps,),
                dtype=self._cache_loc_dtype(),
            )
            positions = torch.zeros((self.max_num_token,), dtype=torch.int64)
            mrope_positions = torch.zeros((3, self.max_num_token), dtype=torch.int64)
            seq_lens = torch.full(
                (self.max_bs,), self.seq_len_fill_value, dtype=torch.int32
            )
            extend_seq_lens = torch.ones((self.max_bs,), dtype=torch.int32)
            topk_p = torch.zeros((self.max_bs, self.topk), dtype=torch.float32)
            topk_index = torch.zeros((self.max_bs, self.topk), dtype=torch.int64)
            _hidden_size = EagleDraftInput.hidden_size_for(self.eagle_worker)
            hidden_states = (
                torch.zeros(
                    (self.max_bs, _hidden_size),
                    dtype=EagleDraftInput.dtype_for(self.eagle_worker),
                )
                if _hidden_size is not None
                else None
            )

            if self.require_gathered_buffer:
                if self.require_mlp_tp_gather:
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

        self.buffers = EagleDraftInputBuffers(
            input_ids=input_ids,
            req_pool_indices=req_pool_indices,
            out_cache_loc=out_cache_loc,
            positions=positions,
            mrope_positions=mrope_positions,
            seq_lens=seq_lens,
            seq_lens_cpu=seq_lens_cpu,
            extend_seq_lens=extend_seq_lens,
            topk_p=topk_p,
            topk_index=topk_index,
            hidden_states=hidden_states,
            global_num_tokens_gpu=global_num_tokens_gpu,
            global_num_tokens_for_logprob_gpu=global_num_tokens_for_logprob_gpu,
        )
        self.buffers.share_buffers()

        # Capture
        try:
```
**EN:** This block initializes the `EAGLEDraftCudaGraphRunner` object, setting up the state, buffers, and references that later methods rely on. This chunk continues the routine's middle-stage logic and data movement.
**CN:** 该代码块初始化 `EAGLEDraftCudaGraphRunner` 对象，建立后续方法依赖的状态、缓冲区与引用。 这一部分继续展示例程中段的逻辑与数据流动。

### Lines 182-187: EAGLEDraftCudaGraphRunner initializer (part 3/3)
```python
            with model_capture_mode():
                self.capture()
        except RuntimeError as e:
            raise Exception(
                f"Capture cuda graph failed: {e}\n{CUDA_GRAPH_CAPTURE_FAILED_MSG}"
            )
```
**EN:** This block initializes the `EAGLEDraftCudaGraphRunner` object, setting up the state, buffers, and references that later methods rely on. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块初始化 `EAGLEDraftCudaGraphRunner` 对象，建立后续方法依赖的状态、缓冲区与引用。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 189-190: EAGLEDraftCudaGraphRunner._cache_loc_dtype method
```python
    def _cache_loc_dtype(self):
        return torch.int64
```
**EN:** This block uses `EAGLEDraftCudaGraphRunner._cache_loc_dtype` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `EAGLEDraftCudaGraphRunner._cache_loc_dtype` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 192-212: EAGLEDraftCudaGraphRunner.can_run method
```python
    def can_run(self, forward_batch: ForwardBatch):
        if self.require_mlp_tp_gather:
            cuda_graph_bs = (
                max(forward_batch.global_num_tokens_cpu) // self.num_tokens_per_bs
                if self.model_runner.spec_algorithm.is_eagle()
                or self.model_runner.spec_algorithm.is_standalone()
                else max(forward_batch.global_num_tokens_cpu)
            )
        else:
            cuda_graph_bs = forward_batch.batch_size

        is_bs_supported = (
            cuda_graph_bs in self.graphs
            if self.disable_padding
            else cuda_graph_bs <= self.max_bs
        )

        if self.require_mlp_sync:
            is_bs_supported = is_bs_supported and forward_batch.can_run_dp_cuda_graph

        return is_bs_supported
```
**EN:** This block uses `EAGLEDraftCudaGraphRunner.can_run` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `EAGLEDraftCudaGraphRunner.can_run` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 214-215: EAGLEDraftCudaGraphRunner._create_graph method
```python
    def _create_graph(self):
        return torch.cuda.CUDAGraph()
```
**EN:** This block uses `EAGLEDraftCudaGraphRunner._create_graph` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `EAGLEDraftCudaGraphRunner._create_graph` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 217-228: EAGLEDraftCudaGraphRunner._capture_init method
```python
    def _capture_init(self, run_once_fn):
        for _ in range(2):
            torch.cuda.synchronize()
            self.model_runner.tp_group.barrier()
            run_once_fn()
            hook = getattr(
                self.model_runner.draft_attn_backend,
                "on_after_cuda_graph_warmup",
                None,
            )
            if hook is not None:
                hook()
```
**EN:** This block uses `EAGLEDraftCudaGraphRunner._capture_init` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `EAGLEDraftCudaGraphRunner._capture_init` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 230-233: EAGLEDraftCudaGraphRunner._capture_graph method
```python
    def _capture_graph(self, graph, pool, stream, run_once_fn):
        with torch.cuda.graph(graph, pool=pool, stream=stream):
            out = run_once_fn()
        return out
```
**EN:** This block uses `EAGLEDraftCudaGraphRunner._capture_graph` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `EAGLEDraftCudaGraphRunner._capture_graph` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 235-242: EAGLEDraftCudaGraphRunner._replay method
```python
    def _replay(self, forward_batch: ForwardBatch):
        ctx = (
            self.model_runner.device_timer.wrap(metadata={"category": "eagle_draft"})
            if self.model_runner.device_timer
            else contextlib.nullcontext()
        )
        with ctx:
            self.graphs[self.bs].replay()
```
**EN:** This block uses `EAGLEDraftCudaGraphRunner._replay` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `EAGLEDraftCudaGraphRunner._replay` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 244-245: EAGLEDraftCudaGraphRunner.capture method
```python
    def capture(self):
        CudaGraphRunner.capture(self)
```
**EN:** This block uses `EAGLEDraftCudaGraphRunner.capture` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `EAGLEDraftCudaGraphRunner.capture` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 247-306: EAGLEDraftCudaGraphRunner.capture_one_batch_size method (part 1/3)
```python
    def capture_one_batch_size(
        self, num_seqs: int, forward: Callable, stream_idx: int = 0
    ):
        buffers = self.buffers
        graph = self._create_graph()
        stream = self.stream
        num_tokens = num_seqs * self.num_tokens_per_bs

        # Graph inputs
        req_pool_indices = buffers.req_pool_indices[:num_seqs]
        seq_lens = buffers.seq_lens[:num_seqs]
        seq_lens_cpu = buffers.seq_lens_cpu[:num_seqs]
        extend_seq_lens = buffers.extend_seq_lens[:num_seqs]
        extend_seq_lens_cpu = self.extend_seq_lens_cpu[:num_seqs]
        out_cache_loc = buffers.out_cache_loc[: num_tokens * self.speculative_num_steps]
        positions = buffers.positions[:num_tokens]
        mrope_positions = buffers.mrope_positions[:, :num_tokens]
        hidden_states = (
            buffers.hidden_states[:num_seqs]
            if buffers.hidden_states is not None
            else None
        )
        topk_p = buffers.topk_p[:num_seqs]
        topk_index = buffers.topk_index[:num_seqs]

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
            global_num_tokens = buffers.global_num_tokens_gpu
            global_dp_buffer_len = num_tokens * self.dp_size
            global_num_tokens_for_logprob = buffers.global_num_tokens_for_logprob_gpu
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
                    [num_tokens],
                    dtype=torch.int32,
                    device=buffers.input_ids.device,
                )
            )
            global_num_tokens = buffers.global_num_tokens_gpu
            global_dp_buffer_len = num_tokens
```
**EN:** This block uses `EAGLEDraftCudaGraphRunner.capture_one_batch_size` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `EAGLEDraftCudaGraphRunner.capture_one_batch_size` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 307-366: EAGLEDraftCudaGraphRunner.capture_one_batch_size method (part 2/3)
```python
            global_num_tokens_for_logprob = buffers.global_num_tokens_for_logprob_gpu
        else:
            global_num_tokens = None
            global_dp_buffer_len = None
            global_num_tokens_for_logprob = None

        capture_mode = (
            CaptureHiddenMode.NULL
            if self.model_runner.spec_algorithm.is_standalone()
            else CaptureHiddenMode.LAST
        )
        spec_info = EagleDraftInput(
            topk_p=topk_p,
            topk_index=topk_index,
            hidden_states=hidden_states,
            capture_hidden_mode=capture_mode,
        )

        # Forward batch
        forward_batch = ForwardBatch(
            forward_mode=ForwardMode.DECODE,
            batch_size=num_seqs,
            input_ids=None,
            req_pool_indices=req_pool_indices,
            seq_lens=seq_lens,
            seq_lens_cpu=seq_lens_cpu,
            extend_seq_lens=extend_seq_lens,
            extend_seq_lens_cpu=extend_seq_lens_cpu,
            req_to_token_pool=self.model_runner.req_to_token_pool,
            token_to_kv_pool=self.model_runner.token_to_kv_pool,
            out_cache_loc=out_cache_loc,
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
            capture_hidden_mode=(
                spec_info.capture_hidden_mode if spec_info else CaptureHiddenMode.NULL
            ),
        )

        # Attention backend
        self.draft_attn_backend.init_forward_metadata_capture_cuda_graph(forward_batch)

        # Run and capture
        def run_once():
            # Clean intermediate result cache for DP attention
            forward_batch.dp_local_start_pos = forward_batch.dp_local_num_tokens = None
            set_dp_buffer_len(
                global_dp_buffer_len,
                num_tokens,
                forward_batch.dp_padding_mode.is_max_len(),
            )
            set_is_extend_in_batch(False)
```
**EN:** This block uses `EAGLEDraftCudaGraphRunner.capture_one_batch_size` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk continues the routine's middle-stage logic and data movement.
**CN:** 该代码块通过 `EAGLEDraftCudaGraphRunner.capture_one_batch_size` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分继续展示例程中段的逻辑与数据流动。

### Lines 367-387: EAGLEDraftCudaGraphRunner.capture_one_batch_size method (part 3/3)
```python
            # Backup fields that are modified in-place in `draft_forward`.
            output_cache_loc_backup = forward_batch.out_cache_loc
            hidden_states_backup = forward_batch.spec_info.hidden_states

            ret = self.eagle_worker.draft_forward(forward_batch)

            forward_batch.out_cache_loc = output_cache_loc_backup
            forward_batch.spec_info.hidden_states = hidden_states_backup
            forward_batch.positions.sub_(self.eagle_worker.speculative_num_steps - 1)
            return ret

        self.deepep_adapter.capture(is_extend_in_batch=False)

        self._capture_init(run_once)

        out = self._capture_graph(
            graph, get_global_graph_memory_pool(), stream, run_once
        )

        set_global_graph_memory_pool(graph.pool())
        return graph, out
```
**EN:** This block uses `EAGLEDraftCudaGraphRunner.capture_one_batch_size` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `EAGLEDraftCudaGraphRunner.capture_one_batch_size` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 389-392: EAGLEDraftCudaGraphRunner._postprocess_output_to_raw_bs method
```python
    def _postprocess_output_to_raw_bs(self, out, raw_bs):
        # Keep the variables name for readability
        parent_list, top_scores_index, draft_tokens = (t[:raw_bs] for t in out)
        return parent_list, top_scores_index, draft_tokens
```
**EN:** This block uses `EAGLEDraftCudaGraphRunner._postprocess_output_to_raw_bs` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `EAGLEDraftCudaGraphRunner._postprocess_output_to_raw_bs` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 394-453: EAGLEDraftCudaGraphRunner.replay method (part 1/2)
```python
    def replay(self, forward_batch: ForwardBatch):
        assert forward_batch.out_cache_loc is not None
        self.deepep_adapter.replay()
        buffers = self.buffers

        raw_bs = forward_batch.batch_size
        raw_num_token = raw_bs * self.num_tokens_per_bs

        # Pad
        if self.require_mlp_tp_gather:
            max_num_tokens = max(forward_batch.global_num_tokens_cpu)
            max_batch_size = (
                max_num_tokens // self.num_tokens_per_bs
                if self.model_runner.spec_algorithm.is_eagle()
                or self.model_runner.spec_algorithm.is_standalone()
                else max_num_tokens
            )
            index = bisect.bisect_left(self.capture_bs, max_batch_size)
        else:
            index = bisect.bisect_left(self.capture_bs, raw_bs)

        bs = self.capture_bs[index]
        if bs != raw_bs:
            buffers.seq_lens.fill_(self.seq_len_fill_value)
            buffers.out_cache_loc.zero_()
            buffers.positions.zero_()
            buffers.topk_p.zero_()
            buffers.topk_index.zero_()
            if buffers.hidden_states is not None:
                buffers.hidden_states.zero_()
            buffers.req_pool_indices.zero_()

        num_tokens = bs * self.num_tokens_per_bs

        # Common inputs
        buffers.seq_lens[:raw_bs].copy_(forward_batch.seq_lens)
        buffers.out_cache_loc[: raw_num_token * self.speculative_num_steps].copy_(
            forward_batch.out_cache_loc
        )
        buffers.positions[:raw_num_token].copy_(forward_batch.positions)
        maybe_detect_nan(
            forward_batch.spec_info.topk_p,
            "EagleDraftCudaGraphRunner.replay: topk_p",
        )
        maybe_detect_oob(
            forward_batch.spec_info.topk_index,
            0,
            self.model_runner.model_config.vocab_size,
            "EagleDraftCudaGraphRunner.replay: topk_index vs vocab_size="
            f"{self.model_runner.model_config.vocab_size}",
        )
        buffers.topk_p[:raw_bs].copy_(forward_batch.spec_info.topk_p)
        buffers.topk_index[:raw_bs].copy_(forward_batch.spec_info.topk_index)
        if (
            buffers.hidden_states is not None
            and forward_batch.spec_info.hidden_states is not None
        ):
            buffers.hidden_states[:raw_bs].copy_(forward_batch.spec_info.hidden_states)
        buffers.req_pool_indices[:raw_bs].copy_(forward_batch.req_pool_indices)
```
**EN:** This block uses `EAGLEDraftCudaGraphRunner.replay` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `EAGLEDraftCudaGraphRunner.replay` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 454-492: EAGLEDraftCudaGraphRunner.replay method (part 2/2)
```python
        # TODO(ch-wan): support num_token_non_padded
        if self.require_gathered_buffer:
            buffers.global_num_tokens_gpu.fill_(bs * self.num_tokens_per_bs)
            buffers.global_num_tokens_for_logprob_gpu.fill_(bs * self.num_tokens_per_bs)

        # Attention backend
        if bs != raw_bs:
            forward_batch.batch_size = bs
            forward_batch.seq_lens = buffers.seq_lens[:bs]
            forward_batch.req_pool_indices = buffers.req_pool_indices[:bs]
            forward_batch.positions = buffers.positions[:num_tokens]

        if forward_batch.seq_lens_cpu is not None:
            if bs != raw_bs:
                buffers.seq_lens_cpu.fill_(self.seq_len_fill_value)
            buffers.seq_lens_cpu[:raw_bs].copy_(forward_batch.seq_lens_cpu)
            forward_batch.seq_lens_cpu = buffers.seq_lens_cpu[:bs]

        self.draft_attn_backend.init_forward_metadata_replay_cuda_graph(
            forward_batch, bs
        )
        self.raw_bs = raw_bs
        self.bs = bs
        # TODO: The forward_batch.seq_len_sum might need to be updated to reflect the padding in the cuda graph

        # Replay
        self._replay(forward_batch)
        out = self.output_buffers[bs]

        if bs != raw_bs:
            out = self._postprocess_output_to_raw_bs(out, raw_bs)
            forward_batch.batch_size = raw_bs
            forward_batch.positions = buffers.positions[:raw_num_token]
            forward_batch.seq_lens = buffers.seq_lens[:raw_bs]
            forward_batch.req_pool_indices = buffers.req_pool_indices[:raw_bs]
            if forward_batch.seq_lens_cpu is not None:
                forward_batch.seq_lens_cpu = buffers.seq_lens_cpu[:raw_bs]

        return out
```
**EN:** This block uses `EAGLEDraftCudaGraphRunner.replay` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `EAGLEDraftCudaGraphRunner.replay` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

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
- `sglang.srt.speculative.eagle_worker`
- `sglang.srt.speculative.spec_utils`
- `sglang.srt.utils`
### External / 外部
- `__future__`
- `torch`
- `bisect` (stdlib)
- `contextlib` (stdlib)
- `dataclasses` (stdlib)
- `typing` (stdlib)
