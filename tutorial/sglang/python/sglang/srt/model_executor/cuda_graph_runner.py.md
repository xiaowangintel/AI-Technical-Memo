# cuda_graph_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/model_executor/cuda_graph_runner.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the model execution and graph runners part of the SRT runtime and implements logic centered on `cuda_graph_runner`. The module docstring frames it as: "Run the model with cuda graph and torch.compile." / 该模块属于 SRT 运行时的模型执行与图运行器部分，主要实现围绕 `cuda_graph_runner` 的逻辑。 它对外提供的主要入口包括 `_grouped_foreach_copy_`, `DecodeInputBuffers`, `get_is_capture_mode`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60: Module imports, constants, and setup (part 1/2)
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
"""Run the model with cuda graph and torch.compile."""

from __future__ import annotations

import bisect
import contextlib
import gc
import inspect
import logging
import os
from contextlib import contextmanager
from dataclasses import dataclass
from functools import partial
from typing import TYPE_CHECKING, Callable, Dict, List, Optional, Tuple, Union

import torch
import tqdm
from torch.profiler import ProfilerActivity, profile

from sglang.srt.batch_overlap.two_batch_overlap import TboCudaGraphRunnerPlugin
from sglang.srt.constants import GPU_MEMORY_TYPE_CUDA_GRAPH
from sglang.srt.distributed import get_tensor_model_parallel_rank
from sglang.srt.distributed.device_communicators.pynccl_allocator import (
    set_graph_pool_id,
)
from sglang.srt.distributed.parallel_state import (
    GroupCoordinator,
    graph_capture,
    set_pdmux_status,
)
from sglang.srt.dllm.config import DllmConfig
from sglang.srt.environ import envs
from sglang.srt.layers.attention.nsa.utils import is_nsa_enable_prefill_cp
from sglang.srt.layers.dp_attention import (
    DpPaddingMode,
    get_attention_cp_size,
    get_attention_tp_rank,
    get_attention_tp_size,
    set_dp_buffer_len,
    set_is_extend_in_batch,
)
from sglang.srt.layers.logits_processor import LogitsProcessorOutput
from sglang.srt.layers.moe.token_dispatcher.deepep import DeepEPBuffer
from sglang.srt.layers.moe.utils import get_deepep_mode, get_moe_a2a_backend
from sglang.srt.layers.utils import MultiPlatformOp
from sglang.srt.model_executor.forward_batch_info import (
    CaptureHiddenMode,
```
**EN:** This range organizes module-level state and shared setup. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; reads environment-driven configuration; prepares compilation-related behavior.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；读取环境变量驱动的配置；处理与编译相关的行为。

### Lines 61-107: Module imports, constants, and setup (part 2/2)
```python
    ForwardBatch,
    ForwardMode,
    NgramEmbeddingInfo,
    PPProxyTensors,
    compute_local_num_token_non_padded,
    enable_num_token_non_padded,
)
from sglang.srt.model_executor.input_buffers import ForwardInputBuffers
from sglang.srt.multiplex.pdmux_context import get_current_stream_idx, get_stream_groups
from sglang.srt.utils import (
    empty_context,
    get_available_gpu_memory,
    get_bool_env_var,
    is_hip,
    log_info_on_rank0,
    require_attn_tp_gather,
    require_gathered_buffer,
    require_mlp_sync,
    require_mlp_tp_gather,
)
from sglang.srt.utils.patch_torch import monkey_patch_torch_compile
from sglang.srt.utils.torch_memory_saver_adapter import TorchMemorySaverAdapter

try:
    from kt_kernel import KTMoEWrapper

    KTRANSFORMERS_AVAILABLE = True
except ImportError:
    KTRANSFORMERS_AVAILABLE = False

_is_hip = is_hip()

if not _is_hip:
    from sglang.srt.model_executor.breakable_cuda_graph.breakable_cuda_graph import (
        BreakableCUDAGraph,
        BreakableCUDAGraphCapture,
        eager_on_graph,
    )

logger = logging.getLogger(__name__)

if TYPE_CHECKING:
    from sglang.srt.model_executor.model_runner import ModelRunner

_has_foreach_copy = hasattr(torch, "_foreach_copy_")


```
**EN:** This range organizes module-level state and shared setup. This chunk is part 2 of 2 for the same logical block. In this range it sets up imports and shared symbols; emits logs for diagnostics; prepares compilation-related behavior.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；处理与编译相关的行为。

### Lines 108-126: Function _grouped_foreach_copy_
```python
def _grouped_foreach_copy_(dsts: List[torch.Tensor], srcs: List[torch.Tensor]) -> None:
    """Call torch._foreach_copy_ grouped by (dst_dtype, src_dtype) pairs."""

    def foreach_copy(dsts: List[torch.Tensor], srcs: List[torch.Tensor]) -> None:
        if _has_foreach_copy:
            torch._foreach_copy_(dsts, srcs)
        else:
            for dst, src in zip(dsts, srcs):
                dst.copy_(src)

    groups: Dict[Tuple[torch.dtype, torch.dtype], Tuple[List, List]] = {}
    for dst, src in zip(dsts, srcs):
        key = (dst.dtype, src.dtype)
        if key not in groups:
            groups[key] = ([], [])
        groups[key][0].append(dst)
        groups[key][1].append(src)
    for group_dsts, group_srcs in groups.values():
        foreach_copy(group_dsts, group_srcs)
```
**EN:** This callable implements `_grouped_foreach_copy_`. It takes `dsts`, `srcs` and mainly implements grouped foreach copy. The docstring states: "Call torch._foreach_copy_ grouped by (dst_dtype, src_dtype) pairs."
**CN:** 这一可调用对象实现了 `_grouped_foreach_copy_`。它接收 `dsts`, `srcs`，主要用于实现 grouped foreach copy 相关逻辑。

### Lines 129-151: Class DecodeInputBuffers
```python
@dataclass
class DecodeInputBuffers(ForwardInputBuffers):

    input_ids: torch.Tensor
    input_embeds: torch.Tensor
    req_pool_indices: torch.Tensor
    seq_lens: torch.Tensor
    seq_lens_cpu: torch.Tensor
    out_cache_loc: torch.Tensor
    out_cache_loc_swa: Optional[torch.Tensor]
    positions: torch.Tensor
    mrope_positions: torch.Tensor
    num_token_non_padded: torch.Tensor
    custom_mask: torch.Tensor
    next_token_logits_buffer: torch.Tensor
    mamba_track_indices: Optional[torch.Tensor]
    mamba_track_mask: Optional[torch.Tensor]
    global_num_tokens_gpu: torch.Tensor
    global_num_tokens_for_logprob_gpu: torch.Tensor
    encoder_lens: Optional[torch.Tensor]
    pp_proxy_tensors: Optional[Dict[str, torch.Tensor]]
    ngram_embedding_info: Optional["NgramEmbeddingInfo"]

```
**EN:** This range introduces `DecodeInputBuffers` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `DecodeInputBuffers`，并定义其后续方法依赖的结构或元数据。

### Lines 152-211: Method DecodeInputBuffers.create (part 1/3)
```python
    @classmethod
    def create(
        cls,
        *,
        device: torch.device,
        max_bs: int,
        max_num_token: int,
        hidden_size: int,
        vocab_size: int,
        dtype: torch.dtype,
        dp_size: int,
        pp_size: int,
        is_encoder_decoder: bool,
        require_mlp_tp_gather: bool,
        seq_len_fill_value: int,
        encoder_len_fill_value: int,
        num_tokens_per_bs: int,
        cache_loc_dtype: torch.dtype,
        enable_mamba_track: bool,
        ne_token_table: Optional[torch.Tensor] = None,
        is_hybrid_swa: bool = False,
        hc_hidden_size: Optional[int] = None,
    ) -> "DecodeInputBuffers":
        with torch.device(device):
            input_ids = torch.zeros((max_num_token,), dtype=torch.int64)
            input_embeds = torch.zeros((max_num_token, hidden_size), dtype=dtype)
            req_pool_indices = torch.zeros((max_bs,), dtype=torch.int64)
            seq_lens = torch.full((max_bs,), seq_len_fill_value, dtype=torch.int32)
            out_cache_loc = torch.zeros((max_num_token,), dtype=cache_loc_dtype)
            out_cache_loc_swa = (
                torch.zeros((max_num_token,), dtype=torch.int32)
                if is_hybrid_swa
                else None
            )
            positions = torch.zeros((max_num_token,), dtype=torch.int64)
            mrope_positions = torch.zeros((3, max_num_token), dtype=torch.int64)
            num_token_non_padded = torch.zeros((1,), dtype=torch.int32)
            custom_mask = torch.ones(
                (max_bs * seq_len_fill_value + max_num_token) * num_tokens_per_bs,
                dtype=torch.bool,
            )
            next_token_logits_buffer = torch.zeros(
                (max_num_token, vocab_size),
                dtype=torch.float,
            )
            mamba_track_indices = (
                torch.zeros((max_bs,), dtype=torch.int64)
                if enable_mamba_track
                else None
            )
            mamba_track_mask = (
                torch.zeros((max_bs,), dtype=torch.bool) if enable_mamba_track else None
            )

            if pp_size > 1:
                # mHC (e.g. DSV4) flattens residual into hidden_states (size = hc_hidden_size).
                is_mhc = hc_hidden_size is not None
                hs = hc_hidden_size if is_mhc else hidden_size
                pp_proxy_tensors = {
                    "hidden_states": torch.zeros((max_bs, hs), dtype=dtype),
```
**EN:** This callable implements `DecodeInputBuffers.create`. It takes `cls` and mainly constructs new objects or contexts. This chunk is part 1 of 3 for the same logical block.
**CN:** 这一可调用对象实现了 `DecodeInputBuffers.create`。它接收 `cls`，主要用于构造新的对象或上下文。 该片段是同一逻辑块的第 1/3 部分。

### Lines 212-271: Method DecodeInputBuffers.create (part 2/3)
```python
                }
                if not is_mhc:
                    pp_proxy_tensors["residual"] = torch.zeros(
                        (max_bs, hidden_size), dtype=dtype
                    )
            else:
                pp_proxy_tensors = None

            if is_encoder_decoder:
                encoder_lens = torch.full(
                    (max_bs,), encoder_len_fill_value, dtype=torch.int32
                )
            else:
                encoder_lens = None

            if require_mlp_tp_gather:
                global_num_tokens_gpu = torch.zeros((dp_size,), dtype=torch.int32)
                global_num_tokens_for_logprob_gpu = torch.zeros(
                    (dp_size,), dtype=torch.int32
                )
            else:
                global_num_tokens_gpu = torch.zeros((1,), dtype=torch.int32)
                global_num_tokens_for_logprob_gpu = torch.zeros((1,), dtype=torch.int32)

            ngram_embedding_info = (
                NgramEmbeddingInfo(
                    token_table=ne_token_table,
                    column_starts=torch.zeros([max_bs], dtype=torch.int32),
                    req_lens=torch.ones([max_bs], dtype=torch.int32),
                    out_column_starts=torch.zeros([max_bs], dtype=torch.int32),
                    out_req_lens=torch.ones([max_bs], dtype=torch.int32),
                )
                if ne_token_table is not None
                else None
            )

        # Keep seq_lens_cpu as a true CPU tensor, like the old implementation.
        seq_lens_cpu = torch.full(
            (max_bs,),
            seq_len_fill_value,
            dtype=torch.int32,
            device="cpu",
        )

        return cls(
            input_ids=input_ids,
            input_embeds=input_embeds,
            req_pool_indices=req_pool_indices,
            seq_lens=seq_lens,
            seq_lens_cpu=seq_lens_cpu,
            out_cache_loc=out_cache_loc,
            out_cache_loc_swa=out_cache_loc_swa,
            positions=positions,
            mrope_positions=mrope_positions,
            num_token_non_padded=num_token_non_padded,
            custom_mask=custom_mask,
            next_token_logits_buffer=next_token_logits_buffer,
            mamba_track_indices=mamba_track_indices,
            mamba_track_mask=mamba_track_mask,
            encoder_lens=encoder_lens,
```
**EN:** This callable implements `DecodeInputBuffers.create`. It takes `cls` and mainly constructs new objects or contexts. This chunk is part 2 of 3 for the same logical block.
**CN:** 这一可调用对象实现了 `DecodeInputBuffers.create`。它接收 `cls`，主要用于构造新的对象或上下文。 该片段是同一逻辑块的第 2/3 部分。

### Lines 272-276: Method DecodeInputBuffers.create (part 3/3)
```python
            global_num_tokens_gpu=global_num_tokens_gpu,
            global_num_tokens_for_logprob_gpu=global_num_tokens_for_logprob_gpu,
            pp_proxy_tensors=pp_proxy_tensors,
            ngram_embedding_info=ngram_embedding_info,
        )
```
**EN:** This callable implements `DecodeInputBuffers.create`. It takes `cls` and mainly constructs new objects or contexts. This chunk is part 3 of 3 for the same logical block.
**CN:** 这一可调用对象实现了 `DecodeInputBuffers.create`。它接收 `cls`，主要用于构造新的对象或上下文。 该片段是同一逻辑块的第 3/3 部分。

### Lines 278-337: Method DecodeInputBuffers.populate_from_forward_batch (part 1/2)
```python
    def populate_from_forward_batch(
        self,
        *,
        forward_batch: ForwardBatch,
        raw_bs: int,
        raw_num_token: int,
        bs: int,
        seq_len_fill_value: int,
        require_gathered_buffer: bool,
        num_tokens_per_bs: int,
        nsa_enable_prefill_cp: bool,
        enable_num_token_non_padded_flag: bool,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
    ):
        if bs != raw_bs:
            self.seq_lens.fill_(seq_len_fill_value)
            self.out_cache_loc.zero_()
            # Padded SWA indices left over from a previous replay would point
            # into real SWA slots, so set_kv_buffer on padded tokens would
            # corrupt active requests' KV. Zero the whole buffer so padded
            # positions map to the sentinel slot (matches piecewise runner).
            if self.out_cache_loc_swa is not None:
                self.out_cache_loc_swa.zero_()
            if self.mamba_track_indices is not None:
                self.mamba_track_indices.zero_()
            if self.mamba_track_mask is not None:
                self.mamba_track_mask.fill_(False)

        # Build batched copy lists for all GPU tensors.
        dsts = [
            self.input_ids[:raw_num_token],
            self.req_pool_indices[:raw_bs],
            self.seq_lens[:raw_bs],
            self.out_cache_loc[:raw_num_token],
            self.positions[:raw_num_token],
        ]
        srcs = [
            forward_batch.input_ids,
            forward_batch.req_pool_indices,
            forward_batch.seq_lens,
            forward_batch.out_cache_loc,
            forward_batch.positions,
        ]

        if self.ngram_embedding_info is not None:
            ngram_embedding_info = forward_batch.ngram_embedding_info
            self.ngram_embedding_info.column_starts[:raw_bs].copy_(
                ngram_embedding_info.column_starts
            )
            self.ngram_embedding_info.req_lens[:raw_bs].copy_(
                ngram_embedding_info.req_lens
            )

        if (
            self.mamba_track_indices is not None
            and forward_batch.mamba_track_indices is not None
        ):
            dsts.append(self.mamba_track_indices[:raw_bs])
            srcs.append(forward_batch.mamba_track_indices)
        if (
```
**EN:** This callable implements `DecodeInputBuffers.populate_from_forward_batch` and mainly constructs data from an external representation. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `DecodeInputBuffers.populate_from_forward_batch`，主要用于从外部表示构造数据。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 338-392: Method DecodeInputBuffers.populate_from_forward_batch (part 2/2)
```python
            self.mamba_track_mask is not None
            and forward_batch.mamba_track_mask is not None
        ):
            dsts.append(self.mamba_track_mask[:raw_bs])
            srcs.append(forward_batch.mamba_track_mask)

        if self.encoder_lens is not None and forward_batch.encoder_lens is not None:
            dsts.append(self.encoder_lens[:raw_bs])
            srcs.append(forward_batch.encoder_lens)

        if forward_batch.mrope_positions is not None:
            dsts.append(self.mrope_positions[:, :raw_num_token])
            srcs.append(forward_batch.mrope_positions)

        if require_gathered_buffer:
            self.global_num_tokens_gpu.fill_(bs * num_tokens_per_bs)
            self.global_num_tokens_for_logprob_gpu.fill_(bs * num_tokens_per_bs)

        if enable_num_token_non_padded_flag:
            if require_gathered_buffer and not nsa_enable_prefill_cp:
                num_tokens_per_dp = bs * num_tokens_per_bs
                local = compute_local_num_token_non_padded(
                    global_num_token_non_padded=forward_batch.num_token_non_padded,
                    num_tokens_per_dp=num_tokens_per_dp,
                )
                dsts.append(self.num_token_non_padded)
                srcs.append(local)
            else:
                dsts.append(self.num_token_non_padded)
                srcs.append(forward_batch.num_token_non_padded)

        # Pipeline-parallel proxy tensors.
        if pp_proxy_tensors is not None and self.pp_proxy_tensors is not None:
            for key, buf in self.pp_proxy_tensors.items():
                src = pp_proxy_tensors.tensors[key]
                dim = src.shape[0]
                dsts.append(buf[:dim])
                srcs.append(src)

        # SWA cache location (int32, separate from the int64 batch above).
        if (
            self.out_cache_loc_swa is not None
            and forward_batch.out_cache_loc_swa is not None
        ):
            dsts.append(self.out_cache_loc_swa[:raw_num_token])
            srcs.append(forward_batch.out_cache_loc_swa[:raw_num_token])

        # Batch all GPU copies, grouped by dtype pair.
        _grouped_foreach_copy_(dsts, srcs)

        # CPU tensor copy (cannot be batched with GPU tensors).
        if forward_batch.seq_lens_cpu is not None:
            if bs != raw_bs:
                self.seq_lens_cpu.fill_(seq_len_fill_value)
            self.seq_lens_cpu[:raw_bs].copy_(forward_batch.seq_lens_cpu)
```
**EN:** This callable implements `DecodeInputBuffers.populate_from_forward_batch` and mainly constructs data from an external representation. This chunk is part 2 of 2 for the same logical block. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `DecodeInputBuffers.populate_from_forward_batch`，主要用于从外部表示构造数据。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 393-398: Module-level constants and helpers
```python


# Detect whether the current forward pass is in capture mode
is_capture_mode = False


```
**EN:** This range organizes module-level state and shared setup. In this range it manages graph capture or replay logic.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 399-400: Function get_is_capture_mode
```python
def get_is_capture_mode():
    return is_capture_mode
```
**EN:** This callable implements `get_is_capture_mode` and mainly retrieves a value or derived view. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `get_is_capture_mode`，主要用于获取某个值或派生视图。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 403-406: Function compile_in_capture_mode
```python
def compile_in_capture_mode(func):
    if get_is_capture_mode():
        return torch.compile(func)
    return func
```
**EN:** This callable implements `compile_in_capture_mode`. It takes `func` and mainly implements compile in capture mode. In this range it prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `compile_in_capture_mode`。它接收 `func`，主要用于实现 compile in capture mode 相关逻辑。 在这一范围内，它会处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 409-416: Function model_capture_mode
```python
@contextmanager
def model_capture_mode():
    global is_capture_mode
    is_capture_mode = True

    yield

    is_capture_mode = False
```
**EN:** This callable implements `model_capture_mode` and mainly implements model capture mode. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `model_capture_mode`，主要用于实现 model capture mode 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 419-435: Function freeze_gc
```python
@contextmanager
def freeze_gc(enable_cudagraph_gc: bool):
    """
    Optimize garbage collection during CUDA graph capture.
    Clean up, then freeze all remaining objects from being included
    in future collections if GC is disabled during capture.
    """
    gc.collect()
    should_freeze = not enable_cudagraph_gc
    if should_freeze:
        gc.freeze()
    try:
        yield
    finally:
        if should_freeze:
            gc.unfreeze()
            gc.collect()
```
**EN:** This callable implements `freeze_gc`. It takes `enable_cudagraph_gc` and mainly implements freeze gc. The docstring states: "Optimize garbage collection during CUDA graph capture." In this range it sets up imports and shared symbols; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `freeze_gc`。它接收 `enable_cudagraph_gc`，主要用于实现 freeze gc 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；管理图捕获或回放逻辑。

### Lines 438-446: Function _to_torch
```python
def _to_torch(model: torch.nn.Module, reverse: bool, num_tokens: int):
    for sub in model._modules.values():
        if isinstance(sub, MultiPlatformOp):
            if reverse:
                sub.leave_torch_compile()
            else:
                sub.enter_torch_compile(num_tokens=num_tokens)
        if isinstance(sub, torch.nn.Module):
            _to_torch(sub, reverse, num_tokens)
```
**EN:** This callable implements `_to_torch`. It takes `model`, `reverse`, `num_tokens` and mainly converts data into another representation. In this range it prepares compilation-related behavior.
**CN:** 这一可调用对象实现了 `_to_torch`。它接收 `model`, `reverse`, `num_tokens`，主要用于将数据转换为另一种表示。 在这一范围内，它会处理与编译相关的行为。

### Lines 449-479: Function patch_model
```python
@contextmanager
def patch_model(
    model: torch.nn.Module,
    enable_compile: bool,
    num_tokens: int,
    tp_group: GroupCoordinator,
):
    """Patch the model to make it compatible with with torch.compile"""
    backup_ca_comm = None

    try:
        if enable_compile:
            _to_torch(model, reverse=False, num_tokens=num_tokens)
            backup_ca_comm = tp_group.ca_comm
            # Use custom-allreduce here.
            # We found the custom allreduce is much faster than the built-in allreduce in torch,
            # even with ENABLE_INTRA_NODE_COMM=1.
            # tp_group.ca_comm = None
            yield torch.compile(
                torch.no_grad()(model.forward),
                mode=os.environ.get(
                    "SGLANG_TORCH_COMPILE_MODE", "max-autotune-no-cudagraphs"
                ),
                dynamic=_is_hip and get_bool_env_var("SGLANG_TORCH_DYNAMIC_SHAPE"),
            )
        else:
            yield model.forward
    finally:
        if enable_compile:
            _to_torch(model, reverse=True, num_tokens=num_tokens)
            tp_group.ca_comm = backup_ca_comm
```
**EN:** This callable implements `patch_model`. It takes `model`, `enable_compile`, `num_tokens`, `tp_group` and mainly implements patch model. The docstring states: "Patch the model to make it compatible with with torch.compile" In this range it reads environment-driven configuration; prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `patch_model`。它接收 `model`, `enable_compile`, `num_tokens`, `tp_group`，主要用于实现 patch model 相关逻辑。 在这一范围内，它会读取环境变量驱动的配置；处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 482-495: Function set_torch_compile_config
```python
def set_torch_compile_config():
    import torch._dynamo.config
    import torch._inductor.config

    torch._inductor.config.coordinate_descent_tuning = True
    torch._inductor.config.triton.unique_kernel_names = True
    torch._inductor.config.fx_graph_cache = True  # Experimental feature to reduce compilation times, will be on by default in future

    # FIXME: tmp workaround
    torch._dynamo.config.accumulated_cache_size_limit = 1024
    if hasattr(torch._dynamo.config, "cache_size_limit"):
        torch._dynamo.config.cache_size_limit = 1024

    monkey_patch_torch_compile()
```
**EN:** This callable implements `set_torch_compile_config` and mainly converts data into another representation. In this range it sets up imports and shared symbols; prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `set_torch_compile_config`，主要用于将数据转换为另一种表示。 在这一范围内，它会建立导入关系并准备共享符号；处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 498-532: Function get_batch_sizes_to_capture
```python
def get_batch_sizes_to_capture(model_runner: ModelRunner, num_tokens_per_bs=1):
    server_args = model_runner.server_args
    capture_bs = server_args.cuda_graph_bs
    num_max_requests = model_runner.req_to_token_pool.size

    mul_base = 1
    if server_args.enable_two_batch_overlap:
        mul_base *= 2
        num_tokens_per_bs = 1  # tbo not test, set num_tokens_per_bs to 1

    if require_gathered_buffer(server_args):
        mul_base *= get_attention_tp_size()

    if mul_base % get_attention_cp_size() != 0:
        mul_base *= get_attention_cp_size()

    # pad `num_max_requests` to avoid being filtered out
    num_max_requests = (num_max_requests + mul_base - 1) // mul_base * mul_base
    if max(capture_bs) > num_max_requests:
        # In some cases (e.g., with a small GPU or --max-running-requests), the #max-running-requests
        # is very small. We add more values here to make sure we capture the maximum bs.
        capture_bs += [num_max_requests]

    # Model input token count = bs * num_tokens_per_bs; must be a multiple of attn_tp_size.
    capture_bs = [bs for bs in capture_bs if bs * num_tokens_per_bs % mul_base == 0]
    capture_bs = [bs for bs in capture_bs if bs <= num_max_requests]
    capture_bs = list(sorted(set(capture_bs)))

    assert len(capture_bs) > 0 and capture_bs[0] > 0, f"{capture_bs=}"
    compile_bs = (
        [bs for bs in capture_bs if bs <= server_args.torch_compile_max_bs]
        if server_args.enable_torch_compile
        else []
    )
    return capture_bs, compile_bs
```
**EN:** This callable implements `get_batch_sizes_to_capture`. It takes `model_runner`, `num_tokens_per_bs` and mainly converts data into another representation. In this range it performs defensive checks on invalid state; prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `get_batch_sizes_to_capture`。它接收 `model_runner`, `num_tokens_per_bs`，主要用于将数据转换为另一种表示。 在这一范围内，它会对非法状态执行防御性检查；处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 533-538: Module-level constants and helpers
```python


# Reuse this memory pool across all cuda graph runners.
global_graph_memory_pool = None


```
**EN:** This range organizes module-level state and shared setup. In this range it manages graph capture or replay logic.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 539-540: Function get_global_graph_memory_pool
```python
def get_global_graph_memory_pool():
    return global_graph_memory_pool
```
**EN:** This callable implements `get_global_graph_memory_pool` and mainly retrieves a value or derived view. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `get_global_graph_memory_pool`，主要用于获取某个值或派生视图。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 543-545: Function set_global_graph_memory_pool
```python
def set_global_graph_memory_pool(val):
    global global_graph_memory_pool
    global_graph_memory_pool = val
```
**EN:** This callable implements `set_global_graph_memory_pool`. It takes `val` and mainly applies configuration to mutable state. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `set_global_graph_memory_pool`。它接收 `val`，主要用于将配置写入可变状态。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 548-550: Class CudaGraphRunner
```python
class CudaGraphRunner:
    """A CudaGraphRunner runs the forward pass of a model with cuda graph and torch.compile."""

```
**EN:** This range introduces `CudaGraphRunner` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "A CudaGraphRunner runs the forward pass of a model with cuda graph and torch.compile." In this range it prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一段引入 `CudaGraphRunner`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 551-610: Method CudaGraphRunner.__init__ (part 1/3)
```python
    def __init__(
        self,
        model_runner: ModelRunner,
        *,
        attn_backend=None,
        speculative_num_steps: Optional[int] = None,
        speculative_num_draft_tokens: Optional[int] = None,
    ):
        # Parse args
        self.model_runner = model_runner
        self.device = model_runner.device
        self.device_module = torch.get_device_module(self.device)
        self.graphs = {}
        self.output_buffers = {}
        self.enable_torch_compile = model_runner.server_args.enable_torch_compile
        self.disable_padding = model_runner.server_args.disable_cuda_graph_padding
        self.is_encoder_decoder = model_runner.model_config.is_encoder_decoder
        self.require_gathered_buffer = require_gathered_buffer(model_runner.server_args)
        self.require_mlp_tp_gather = require_mlp_tp_gather(model_runner.server_args)
        self.require_mlp_sync = require_mlp_sync(model_runner.server_args)
        self.require_attn_tp_gather = require_attn_tp_gather(model_runner.server_args)
        self.enable_two_batch_overlap = (
            model_runner.server_args.enable_two_batch_overlap
        )
        self.use_ngram_embedding = model_runner.use_ngram_embedding
        if self.use_ngram_embedding:
            hf_config = model_runner.model_config.hf_config
            self.ngram_embedding_n = hf_config.ngram_embedding_n
            self.ngram_embedding_k = hf_config.ngram_embedding_k
        self.speculative_algorithm = model_runner.server_args.speculative_algorithm
        self.enable_profile_cuda_graph = (
            model_runner.server_args.enable_profile_cuda_graph
        )
        self.tp_size = model_runner.server_args.tp_size
        self.dp_size = model_runner.server_args.dp_size
        self.pp_size = model_runner.server_args.pp_size
        self.enable_pdmux = model_runner.server_args.enable_pdmux

        self.attn_tp_size = get_attention_tp_size()
        self.attn_tp_rank = get_attention_tp_rank()
        self.nsa_enable_prefill_cp = is_nsa_enable_prefill_cp()

        self.deepep_adapter = DeepEPCudaGraphRunnerAdapter()

        self.dllm_config = DllmConfig.from_server_args(model_runner.server_args)
        self.is_dllm = self.dllm_config is not None
        self.attn_backend = attn_backend or model_runner.attn_backend
        self.speculative_num_steps = (
            model_runner.server_args.speculative_num_steps
            if speculative_num_steps is None
            else speculative_num_steps
        )
        self.speculative_num_draft_tokens = (
            model_runner.server_args.speculative_num_draft_tokens
            if speculative_num_draft_tokens is None
            else speculative_num_draft_tokens
        )

        self.capture_forward_mode = ForwardMode.DECODE
        self.capture_hidden_mode = CaptureHiddenMode.NULL
```
**EN:** This callable implements `CudaGraphRunner.__init__`. It takes `model_runner` and mainly initializes instance state and defaults. This chunk is part 1 of 3 for the same logical block. In this range it prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.__init__`。它接收 `model_runner`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 1/3 部分。 在这一范围内，它会处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 611-670: Method CudaGraphRunner.__init__ (part 2/3)
```python
        self.num_tokens_per_bs = 1
        if model_runner.spec_algorithm.is_speculative():
            if self.model_runner.is_draft_worker:
                # Draft workers can use TARGET_VERIFY mode.
                if (
                    not self.model_runner.spec_algorithm.supports_target_verify_for_draft()
                ):
                    raise RuntimeError("This should not happen")
            self.capture_forward_mode = ForwardMode.TARGET_VERIFY
            self.num_tokens_per_bs = self.speculative_num_draft_tokens
        elif self.is_dllm:
            self.capture_forward_mode = ForwardMode.DLLM_EXTEND
            self.num_tokens_per_bs = self.dllm_config.block_size

        # Batch sizes to capture
        self.capture_bs, self.compile_bs = get_batch_sizes_to_capture(
            model_runner, self.num_tokens_per_bs
        )
        log_info_on_rank0(logger, f"Capture cuda graph bs {self.capture_bs}")
        if KTRANSFORMERS_AVAILABLE:
            KTMoEWrapper.set_capture_batch_sizes(self.capture_bs)

        # If returning hidden states is enabled, set initial capture hidden mode to full to avoid double-capture on startup
        if model_runner.server_args.enable_return_hidden_states:
            self.capture_hidden_mode = CaptureHiddenMode.FULL

        # Attention backend
        self.max_bs = max(self.capture_bs)
        self.max_num_token = self.max_bs * self.num_tokens_per_bs
        self.attn_backend.init_cuda_graph_state(self.max_bs, self.max_num_token)

        # Init PDMux if needed
        self.maybe_init_pdmux()
        self.seq_len_fill_value = (
            self.attn_backend.get_cuda_graph_seq_len_fill_value()
            if self.dllm_config is None
            else self.dllm_config.block_size
        )

        # Non-zero encoder length ensures cross-attention kernels are captured in the graph.
        self.encoder_len_fill_value = (
            getattr(model_runner.model_config.hf_config, "max_source_positions", 0)
            if self.is_encoder_decoder
            else 0
        )

        if self.enable_torch_compile:
            set_torch_compile_config()

        if self.model_runner.server_args.enable_lora:
            # Phase 2 of LoRA CUDA graph init: dense LoRA batch metadata.
            # Phase 1 (MoE buffers) was handled earlier in ModelRunner via
            # lora_manager.init_cuda_graph_moe_buffers().
            self.model_runner.lora_manager.init_cuda_graph_batch_info(
                max_bs_in_cuda_graph=self.max_bs,
                num_tokens_per_bs=self.num_tokens_per_bs,
            )

        enable_mamba_track = (
            self.model_runner.server_args.enable_mamba_extra_buffer()
```
**EN:** This callable implements `CudaGraphRunner.__init__`. It takes `model_runner` and mainly initializes instance state and defaults. This chunk is part 2 of 3 for the same logical block. In this range it performs defensive checks on invalid state; prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.__init__`。它接收 `model_runner`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 2/3 部分。 在这一范围内，它会对非法状态执行防御性检查；处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 671-711: Method CudaGraphRunner.__init__ (part 3/3)
```python
            and self.model_runner.spec_algorithm.is_none()
        )

        if self.require_gathered_buffer:
            assert self.require_mlp_tp_gather or self.require_attn_tp_gather
        self.buffers: DecodeInputBuffers = DecodeInputBuffers.create(
            device=self.device,
            max_bs=self.max_bs,
            max_num_token=self.max_num_token,
            hidden_size=self.model_runner.model_config.hidden_size,
            vocab_size=self.model_runner.model_config.vocab_size,
            dtype=self.model_runner.model_config.dtype,
            dp_size=self.dp_size,
            pp_size=self.pp_size,
            is_encoder_decoder=self.is_encoder_decoder,
            require_mlp_tp_gather=self.require_mlp_tp_gather,
            seq_len_fill_value=self.seq_len_fill_value,
            encoder_len_fill_value=self.encoder_len_fill_value,
            num_tokens_per_bs=self.num_tokens_per_bs,
            cache_loc_dtype=self._cache_loc_dtype(),
            enable_mamba_track=enable_mamba_track,
            ne_token_table=(
                model_runner.token_table if self.use_ngram_embedding else None
            ),
            is_hybrid_swa=model_runner.is_hybrid_swa,
            hc_hidden_size=getattr(
                self.model_runner.model_config, "hc_hidden_size", None
            ),
        )
        self.buffers.share_buffers()

        self.tbo_plugin = TboCudaGraphRunnerPlugin()

        # Capture
        try:
            with model_capture_mode():
                self.capture()
        except RuntimeError as e:
            raise Exception(
                f"Capture cuda graph failed: {e}\n{CUDA_GRAPH_CAPTURE_FAILED_MSG}"
            )
```
**EN:** This callable implements `CudaGraphRunner.__init__`. It takes `model_runner` and mainly initializes instance state and defaults. This chunk is part 3 of 3 for the same logical block. In this range it performs defensive checks on invalid state; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.__init__`。它接收 `model_runner`，主要用于初始化实例状态与默认值。 该片段是同一逻辑块的第 3/3 部分。 在这一范围内，它会对非法状态执行防御性检查；管理图捕获或回放逻辑。

### Lines 713-717: Method CudaGraphRunner.maybe_init_pdmux
```python
    def maybe_init_pdmux(self):
        if self.enable_pdmux:
            self.stream_groups = get_stream_groups()
            for attn_backend in self.model_runner.decode_attn_backend_group:
                attn_backend.init_cuda_graph_state(self.max_bs, self.max_num_token)
```
**EN:** This callable implements `CudaGraphRunner.maybe_init_pdmux` and mainly implements maybe init pdmux. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.maybe_init_pdmux`，主要用于实现 maybe init pdmux 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 719-720: Method CudaGraphRunner._cache_loc_dtype
```python
    def _cache_loc_dtype(self):
        return torch.int64
```
**EN:** This callable implements `CudaGraphRunner._cache_loc_dtype` and mainly implements cache loc dtype.
**CN:** 这一可调用对象实现了 `CudaGraphRunner._cache_loc_dtype`，主要用于实现 cache loc dtype 相关逻辑。

### Lines 722-781: Method CudaGraphRunner.can_run (part 1/2)
```python
    def can_run(self, forward_batch: ForwardBatch):
        # Disable for token embedding overrides (dynamic per-request)
        if forward_batch.replace_embeds is not None:
            return False
        if self.require_mlp_tp_gather:
            cuda_graph_bs = (
                max(forward_batch.global_num_tokens_cpu) // self.num_tokens_per_bs
                if self.model_runner.spec_algorithm.is_eagle()
                or self.model_runner.spec_algorithm.is_standalone()
                or self.model_runner.spec_algorithm.is_dflash()
                else max(forward_batch.global_num_tokens_cpu)
            )
        else:
            cuda_graph_bs = forward_batch.batch_size

        graph_key = cuda_graph_bs
        if self.enable_pdmux:
            graph_key = f"{get_current_stream_idx()}_{cuda_graph_bs}"

        is_bs_supported = (
            graph_key in self.graphs
            if self.disable_padding
            else cuda_graph_bs <= self.max_bs
        )

        if self.require_mlp_sync:
            is_bs_supported = is_bs_supported and forward_batch.can_run_dp_cuda_graph

        # NOTE: cuda graph cannot handle mixed batch (encoder_len = 0)
        # If mixed batch cannot be supported, then encoder_lens can be removed in cuda graph
        # because the full_text_row_masked_out_mask tensor will always be ones
        is_encoder_lens_supported = (
            torch.all(forward_batch.encoder_lens > 0)
            if self.is_encoder_decoder
            else True
        )

        requested_capture_hidden_mode = max(
            forward_batch.capture_hidden_mode,
            (
                forward_batch.spec_info.capture_hidden_mode
                if getattr(forward_batch.spec_info, "capture_hidden_mode", None)
                is not None
                else CaptureHiddenMode.NULL
            ),
        )
        capture_hidden_mode_matches = (
            requested_capture_hidden_mode == CaptureHiddenMode.NULL
            or requested_capture_hidden_mode == self.capture_hidden_mode
        )
        is_tbo_supported = (
            forward_batch.can_run_tbo if self.enable_two_batch_overlap else True
        )

        is_ngram_supported = (
            (
                forward_batch.batch_size * self.num_tokens_per_bs
                == forward_batch.input_ids.numel()
            )
            if self.model_runner.spec_algorithm.is_ngram()
```
**EN:** This callable implements `CudaGraphRunner.can_run`. It takes `forward_batch` and mainly executes the main workflow. This chunk is part 1 of 2 for the same logical block. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.can_run`。它接收 `forward_batch`，主要用于执行主要流程。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 782-791: Method CudaGraphRunner.can_run (part 2/2)
```python
            else True
        )

        return (
            is_bs_supported
            and is_encoder_lens_supported
            and is_tbo_supported
            and capture_hidden_mode_matches
            and is_ngram_supported
        )
```
**EN:** This callable implements `CudaGraphRunner.can_run`. It takes `forward_batch` and mainly executes the main workflow. This chunk is part 2 of 2 for the same logical block. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.can_run`。它接收 `forward_batch`，主要用于执行主要流程。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 793-799: Method CudaGraphRunner._init_profile_context_and_memory_record
```python
    def _init_profile_context_and_memory_record(self):
        profile_context = profile(
            activities=[ProfilerActivity.CPU, ProfilerActivity.CUDA],
            record_shapes=True,
        )
        torch.cuda.memory._record_memory_history()
        return profile_context
```
**EN:** This callable implements `CudaGraphRunner._init_profile_context_and_memory_record` and mainly implements init profile context and memory record.
**CN:** 这一可调用对象实现了 `CudaGraphRunner._init_profile_context_and_memory_record`，主要用于实现 init profile context and memory record 相关逻辑。

### Lines 801-815: Method CudaGraphRunner._post_process_after_profile
```python
    def _post_process_after_profile(self, prof_context):
        torch.cuda.memory._dump_snapshot(f"cuda_graph_runner_memory_usage.pickle")
        torch.cuda.memory._record_memory_history(enabled=None)
        log_message = (
            "Sorted by CUDA Time:\n"
            + prof_context.key_averages(group_by_input_shape=True).table(
                sort_by="cuda_time_total", row_limit=10
            )
            + "\n\nSorted by CPU Time:\n"
            + prof_context.key_averages(group_by_input_shape=True).table(
                sort_by="cpu_time_total", row_limit=10
            )
            + "\n\nMemory Usage is saved to cuda_graph_runner_memory_usage.pickle\n"
        )
        logger.info(log_message)
```
**EN:** This callable implements `CudaGraphRunner._post_process_after_profile`. It takes `prof_context` and mainly implements post process after profile. In this range it emits logs for diagnostics; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner._post_process_after_profile`。它接收 `prof_context`，主要用于实现 post process after profile 相关逻辑。 在这一范围内，它会输出日志以便诊断；管理图捕获或回放逻辑。

### Lines 817-876: Method CudaGraphRunner.capture (part 1/2)
```python
    def capture(self) -> None:
        profile_context = empty_context()
        if self.enable_profile_cuda_graph:
            profile_context = self._init_profile_context_and_memory_record()

        def _capture_one_stream(stream_idx: Optional[int] = None):
            avail_mem = get_available_gpu_memory(
                self.model_runner.device,
                self.model_runner.gpu_id,
                empty_cache=False,
            )
            # Reverse the order to enable better memory sharing across cuda graphs.
            capture_range = (
                tqdm.tqdm(list(reversed(self.capture_bs)))
                if get_tensor_model_parallel_rank() == 0
                else reversed(self.capture_bs)
            )
            for i, bs in enumerate(capture_range):
                if get_tensor_model_parallel_rank() == 0:
                    avail_mem = get_available_gpu_memory(
                        self.model_runner.device,
                        self.model_runner.gpu_id,
                        empty_cache=False,
                    )
                    capture_range.set_description(
                        f"Capturing batches ({bs=} {avail_mem=:.2f} GB)"
                    )

                with patch_model(
                    self.model_runner.model,
                    bs in self.compile_bs,
                    num_tokens=bs * self.num_tokens_per_bs,
                    tp_group=self.model_runner.tp_group,
                ) as forward:
                    (
                        graph,
                        output_buffers,
                    ) = self.capture_one_batch_size(bs, forward, stream_idx)
                    # For pd_multiplexing, we need to save the graph and output buffers
                    key = bs if stream_idx is None else f"{stream_idx}_{bs}"
                    self.graphs[key] = graph
                    self.output_buffers[key] = output_buffers

        # Trigger CUDA graph capture for specific shapes.
        # Capture the large shapes first so that the smaller shapes
        # can reuse the memory pool allocated for the large shapes.
        with freeze_gc(self.model_runner.server_args.enable_cudagraph_gc):
            if not self.enable_pdmux:
                with graph_capture() as graph_capture_context, profile_context as prof:
                    self.stream = graph_capture_context.stream
                    _capture_one_stream()
            else:
                set_pdmux_status(False)
                for i, sg in enumerate(self.stream_groups):
                    with (
                        graph_capture(stream=sg[1]) as graph_capture_context,
                        profile_context as prof,
                    ):
                        self.stream = graph_capture_context.stream
                        _capture_one_stream(i)
```
**EN:** This callable implements `CudaGraphRunner.capture` and mainly implements capture. This chunk is part 1 of 2 for the same logical block. In this range it prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.capture`，主要用于实现 capture 相关逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 877-879: Method CudaGraphRunner.capture (part 2/2)
```python

        if self.enable_profile_cuda_graph:
            self._post_process_after_profile(prof)
```
**EN:** This callable implements `CudaGraphRunner.capture` and mainly implements capture. This chunk is part 2 of 2 for the same logical block. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.capture`，主要用于实现 capture 相关逻辑。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 881-912: Method CudaGraphRunner._capture_graph
```python
    def _capture_graph(self, graph, pool, stream, run_once_fn):
        if self.model_runner.server_args.debug_cuda_graph:
            assert (
                envs.SGLANG_USE_BREAKABLE_CUDA_GRAPH.get()
            ), "Breakable CUDA graph is not enabled in debug mode"

        memory_saver_adapter = TorchMemorySaverAdapter.create(
            enable=self.model_runner.server_args.enable_memory_saver
            and get_bool_env_var("SGLANG_MEMORY_SAVER_CUDA_GRAPH")
        )

        if envs.SGLANG_USE_BREAKABLE_CUDA_GRAPH.get():
            if memory_saver_adapter.enabled:
                raise NotImplementedError(
                    "Breakable CUDA graph is not compatible with memory saver mode"
                )
            graph_ctx = BreakableCUDAGraphCapture
        else:
            graph_ctx = (
                partial(memory_saver_adapter.cuda_graph, tag=GPU_MEMORY_TYPE_CUDA_GRAPH)
                if memory_saver_adapter.enabled
                else self.device_module.graph
            )

        if self.model_runner.server_args.debug_cuda_graph:
            captured_fn = eager_on_graph(True)(run_once_fn)
        else:
            captured_fn = run_once_fn

        with graph_ctx(cuda_graph=graph, pool=pool, stream=stream):
            out = captured_fn()
        return out
```
**EN:** This callable implements `CudaGraphRunner._capture_graph`. It takes `graph`, `pool`, `stream`, `run_once_fn` and mainly implements capture graph. In this range it performs defensive checks on invalid state; reads environment-driven configuration; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner._capture_graph`。它接收 `graph`, `pool`, `stream`, `run_once_fn`，主要用于实现 capture graph 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；读取环境变量驱动的配置；管理图捕获或回放逻辑。

### Lines 914-919: Method CudaGraphRunner._create_device_graph
```python
    def _create_device_graph(self):
        if envs.SGLANG_USE_BREAKABLE_CUDA_GRAPH.get():
            if _is_hip:
                raise RuntimeError("Breakable CUDA graph is not supported on ROCm/HIP")
            return BreakableCUDAGraph()
        return torch.cuda.CUDAGraph()
```
**EN:** This callable implements `CudaGraphRunner._create_device_graph` and mainly constructs new objects or contexts. In this range it performs defensive checks on invalid state; reads environment-driven configuration; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner._create_device_graph`，主要用于构造新的对象或上下文。 在这一范围内，它会对非法状态执行防御性检查；读取环境变量驱动的配置；管理图捕获或回放逻辑。

### Lines 921-980: Method CudaGraphRunner.capture_one_batch_size (part 1/4)
```python
    def capture_one_batch_size(
        self, bs: int, forward: Callable, stream_idx: Optional[int] = None
    ):
        buffers: DecodeInputBuffers = self.buffers
        graph = self._create_device_graph()
        stream = self.stream
        num_tokens = bs * self.num_tokens_per_bs

        # Graph inputs
        input_ids = buffers.input_ids[:num_tokens]
        req_pool_indices = buffers.req_pool_indices[:bs]
        seq_lens = buffers.seq_lens[:bs]
        seq_lens_cpu = buffers.seq_lens_cpu[:bs]
        out_cache_loc = buffers.out_cache_loc[:num_tokens]
        positions = buffers.positions[:num_tokens]
        if self.is_encoder_decoder:
            encoder_lens = buffers.encoder_lens[:bs]
        else:
            encoder_lens = None
        mrope_positions = buffers.mrope_positions[:, :num_tokens]
        next_token_logits_buffer = buffers.next_token_logits_buffer[:num_tokens]

        # Adjust for attention TP if needed (matching replay path in
        # populate_from_forward_batch).
        buffers.num_token_non_padded[...] = num_tokens
        if (
            enable_num_token_non_padded()
            and self.require_gathered_buffer
            and not self.nsa_enable_prefill_cp
        ):
            local = compute_local_num_token_non_padded(
                global_num_token_non_padded=buffers.num_token_non_padded,
                num_tokens_per_dp=num_tokens,
            )
            buffers.num_token_non_padded.copy_(local)

        # pipeline parallelism
        if self.pp_size > 1:
            pp_proxy_tensors = PPProxyTensors(
                {k: v[:num_tokens] for k, v in buffers.pp_proxy_tensors.items()}
            )

        if self.require_mlp_tp_gather:
            buffers.global_num_tokens_gpu.copy_(
                torch.tensor(
                    [num_tokens] * self.dp_size,
                    dtype=torch.int32,
                    device=input_ids.device,
                )
            )
            buffers.global_num_tokens_for_logprob_gpu.copy_(
                torch.tensor(
                    [num_tokens] * self.dp_size,
                    dtype=torch.int32,
                    device=input_ids.device,
                )
            )
            global_dp_buffer_len = num_tokens * self.dp_size
        elif self.require_attn_tp_gather:
            buffers.global_num_tokens_gpu.copy_(
```
**EN:** This callable implements `CudaGraphRunner.capture_one_batch_size`. It takes `bs`, `forward`, `stream_idx` and mainly implements capture one batch size. This chunk is part 1 of 4 for the same logical block. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.capture_one_batch_size`。它接收 `bs`, `forward`, `stream_idx`，主要用于实现 capture one batch size 相关逻辑。 该片段是同一逻辑块的第 1/4 部分。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 981-1040: Method CudaGraphRunner.capture_one_batch_size (part 2/4)
```python
                torch.tensor(
                    [num_tokens],
                    dtype=torch.int32,
                    device=input_ids.device,
                )
            )
            buffers.global_num_tokens_for_logprob_gpu.copy_(
                torch.tensor(
                    [num_tokens],
                    dtype=torch.int32,
                    device=input_ids.device,
                )
            )
            global_dp_buffer_len = num_tokens
        else:
            global_dp_buffer_len = None

        spec_info = self.get_spec_info(num_tokens)
        if self.capture_hidden_mode != CaptureHiddenMode.FULL:
            self.capture_hidden_mode = (
                spec_info.capture_hidden_mode if spec_info else CaptureHiddenMode.NULL
            )

        if self.model_runner.server_args.enable_lora:
            # It is safe to capture CUDA graph using empty LoRA id, as the LoRA kernels will always be launched whenever
            # `--enable-lora` is set to True (and return immediately if the LoRA id is empty for perf optimization).
            lora_ids = [None] * bs
        else:
            lora_ids = None

        # mamba state tracking
        mamba_track_indices = (
            buffers.mamba_track_indices[:bs]
            if buffers.mamba_track_indices is not None
            else None
        )
        mamba_track_mask = (
            buffers.mamba_track_mask[:bs]
            if buffers.mamba_track_mask is not None
            else None
        )

        if stream_idx is None:
            attn_backend = self.attn_backend
        else:
            assert self.enable_pdmux
            attn_backend = self.model_runner.decode_attn_backend_group[stream_idx]

        forward_batch = ForwardBatch(
            forward_mode=self.capture_forward_mode,
            batch_size=bs,
            input_ids=input_ids,
            req_pool_indices=req_pool_indices,
            seq_lens=seq_lens,
            seq_lens_cpu=seq_lens_cpu,
            next_token_logits_buffer=next_token_logits_buffer,
            orig_seq_lens=seq_lens,
            req_to_token_pool=self.model_runner.req_to_token_pool,
            token_to_kv_pool=self.model_runner.token_to_kv_pool,
            attn_backend=attn_backend,
```
**EN:** This callable implements `CudaGraphRunner.capture_one_batch_size`. It takes `bs`, `forward`, `stream_idx` and mainly implements capture one batch size. This chunk is part 2 of 4 for the same logical block. In this range it performs defensive checks on invalid state; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.capture_one_batch_size`。它接收 `bs`, `forward`, `stream_idx`，主要用于实现 capture one batch size 相关逻辑。 该片段是同一逻辑块的第 2/4 部分。 在这一范围内，它会对非法状态执行防御性检查；管理图捕获或回放逻辑。

### Lines 1041-1100: Method CudaGraphRunner.capture_one_batch_size (part 3/4)
```python
            out_cache_loc=out_cache_loc,
            seq_lens_sum=seq_lens.sum().item(),
            mamba_track_indices=mamba_track_indices,
            mamba_track_mask=mamba_track_mask,
            mamba_track_seqlens=None,  # Prefill only
            encoder_lens=encoder_lens,
            return_logprob=False,
            positions=positions,
            global_num_tokens_gpu=buffers.global_num_tokens_gpu,
            global_num_tokens_for_logprob_gpu=buffers.global_num_tokens_for_logprob_gpu,
            dp_padding_mode=DpPaddingMode.get_default_mode_in_cuda_graph(),
            global_dp_buffer_len=global_dp_buffer_len,
            mrope_positions=mrope_positions,
            spec_algorithm=self.model_runner.spec_algorithm,
            spec_info=spec_info,
            capture_hidden_mode=self.capture_hidden_mode,
            num_token_non_padded=buffers.num_token_non_padded,
            global_forward_mode=self.capture_forward_mode,
            lora_ids=lora_ids,
        )

        # HiSparse: set coordinator so the hisparse code path is captured into the graph
        forward_batch.hisparse_coordinator = self.model_runner.hisparse_coordinator
        if forward_batch.hisparse_coordinator is not None:
            forward_batch.hisparse_coordinator.num_real_reqs.fill_(bs)

        if buffers.ngram_embedding_info is not None:
            forward_batch.ngram_embedding_info = buffers.ngram_embedding_info.slice(bs)

        self.tbo_plugin.capture_one_batch_size(forward_batch, num_tokens=num_tokens)

        if lora_ids is not None:
            self.model_runner.lora_manager.prepare_lora_batch(forward_batch)

        # Attention backend
        attn_backend.init_forward_metadata_capture_cuda_graph(
            bs,
            num_tokens,
            req_pool_indices,
            seq_lens,
            encoder_lens,
            forward_batch.forward_mode,
            forward_batch.spec_info,
        )

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

            kwargs = {}
            if (
                self.pp_size > 1
                and "pp_proxy_tensors" in inspect.signature(forward).parameters
```
**EN:** This callable implements `CudaGraphRunner.capture_one_batch_size`. It takes `bs`, `forward`, `stream_idx` and mainly implements capture one batch size. This chunk is part 3 of 4 for the same logical block. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.capture_one_batch_size`。它接收 `bs`, `forward`, `stream_idx`，主要用于实现 capture one batch size 相关逻辑。 该片段是同一逻辑块的第 3/4 部分。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 1101-1146: Method CudaGraphRunner.capture_one_batch_size (part 4/4)
```python
            ):
                kwargs["pp_proxy_tensors"] = PPProxyTensors(
                    {k: v.clone() for k, v in pp_proxy_tensors.tensors.items()}
                )
            if (
                self.model_runner.spec_algorithm.is_dflash()
                and self.model_runner.is_draft_worker
                and "input_embeds" in inspect.signature(forward).parameters
            ):
                kwargs["input_embeds"] = buffers.input_embeds[:num_tokens]

            logits_output_or_pp_proxy_tensors = forward(
                input_ids,
                forward_batch.positions,
                forward_batch,
                **kwargs,
            )
            return logits_output_or_pp_proxy_tensors

        self.deepep_adapter.capture(is_extend_in_batch=False)

        # swa_loc must be set before capture so that set_kv_buffer's
        # Python branch (if self.swa_loc is not None) takes the fast path,
        # and the graph records GPU ops using this buffer instead of the
        # per-layer translate_loc_from_full_to_swa fallback.
        if self.buffers.out_cache_loc_swa is not None:
            self.model_runner.token_to_kv_pool.set_swa_loc(
                self.buffers.out_cache_loc_swa[:num_tokens]
            )

        for _ in range(2):
            self.device_module.synchronize()
            self.model_runner.tp_group.barrier()
            run_once()
            attn_backend.on_after_cuda_graph_warmup()

        if get_global_graph_memory_pool() is None:
            set_global_graph_memory_pool(self.device_module.graph_pool_handle())
        # Set graph pool id globally to be able to use symmetric memory
        set_graph_pool_id(get_global_graph_memory_pool())

        out = self._capture_graph(
            graph, get_global_graph_memory_pool(), stream, run_once
        )

        return graph, out
```
**EN:** This callable implements `CudaGraphRunner.capture_one_batch_size`. It takes `bs`, `forward`, `stream_idx` and mainly implements capture one batch size. This chunk is part 4 of 4 for the same logical block. In this range it records metrics or tracing signals; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.capture_one_batch_size`。它接收 `bs`, `forward`, `stream_idx`，主要用于实现 capture one batch size 相关逻辑。 该片段是同一逻辑块的第 4/4 部分。 在这一范围内，它会记录指标或追踪信号；管理图捕获或回放逻辑。

### Lines 1148-1178: Method CudaGraphRunner.recapture_if_needed
```python
    def recapture_if_needed(self, forward_batch: ForwardBatch):

        # If the required capture_hidden_mode changes, we need to recapture the graph

        # These are the different factors that can influence the capture_hidden_mode
        capture_hidden_mode_required_by_forward_batch = (
            forward_batch.capture_hidden_mode
        )
        capture_hidden_mode_required_by_spec_info = (
            getattr(forward_batch.spec_info, "capture_hidden_mode", None)
            or CaptureHiddenMode.NULL
        )
        capture_hidden_mode_required_for_returning_hidden_states = (
            CaptureHiddenMode.FULL
            if self.model_runner.server_args.enable_return_hidden_states
            else CaptureHiddenMode.NULL
        )

        # Determine the highest capture_hidden_mode required
        # (If we have FULL, we can emulate LAST or NULL)
        # (If we have LAST, we can emulate NULL)
        required_capture_hidden_mode = max(
            capture_hidden_mode_required_by_forward_batch,
            capture_hidden_mode_required_by_spec_info,
            capture_hidden_mode_required_for_returning_hidden_states,
        )

        # If the current hidden mode is no longer aligned with the required hidden mode, we need to set it to what is required and re-capture
        if self.capture_hidden_mode != required_capture_hidden_mode:
            self.capture_hidden_mode = required_capture_hidden_mode
            self.capture()
```
**EN:** This callable implements `CudaGraphRunner.recapture_if_needed`. It takes `forward_batch` and mainly implements recapture if needed. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.recapture_if_needed`。它接收 `forward_batch`，主要用于实现 recapture if needed 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 1180-1239: Method CudaGraphRunner.replay_prepare (part 1/2)
```python
    def replay_prepare(
        self,
        forward_batch: ForwardBatch,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
    ):
        buffers = self.buffers
        self.recapture_if_needed(forward_batch)

        raw_bs = forward_batch.batch_size
        raw_num_token = raw_bs * self.num_tokens_per_bs

        # Pad
        if self.require_mlp_tp_gather:
            max_num_tokens = max(forward_batch.global_num_tokens_cpu)
            max_batch_size = (
                max_num_tokens / self.num_tokens_per_bs
                if self.model_runner.spec_algorithm.is_eagle()
                or self.model_runner.spec_algorithm.is_standalone()
                or self.model_runner.spec_algorithm.is_dflash()
                else max_num_tokens
            )
            index = bisect.bisect_left(self.capture_bs, max_batch_size)
        else:
            index = bisect.bisect_left(self.capture_bs, raw_bs)
        bs = self.capture_bs[index]

        buffers.populate_from_forward_batch(
            forward_batch=forward_batch,
            raw_bs=raw_bs,
            raw_num_token=raw_num_token,
            bs=bs,
            seq_len_fill_value=self.seq_len_fill_value,
            require_gathered_buffer=self.require_gathered_buffer,
            num_tokens_per_bs=self.num_tokens_per_bs,
            nsa_enable_prefill_cp=self.nsa_enable_prefill_cp,
            enable_num_token_non_padded_flag=enable_num_token_non_padded(),
            pp_proxy_tensors=pp_proxy_tensors,
        )

        if (
            self.model_runner.spec_algorithm.is_dflash()
            and self.model_runner.is_draft_worker
            and forward_batch.input_embeds is not None
        ):
            buffers.input_embeds[:raw_num_token].copy_(forward_batch.input_embeds)
            # Padded tokens aren't read, so skip zeroing them.
        if self.enable_two_batch_overlap:
            self.tbo_plugin.replay_prepare(
                forward_mode=self.capture_forward_mode,
                bs=bs,
                num_token_non_padded=len(forward_batch.input_ids),
                spec_info=forward_batch.spec_info,
            )
        if forward_batch.forward_mode.is_idle() and forward_batch.spec_info is not None:
            forward_batch.spec_info.custom_mask = buffers.custom_mask
        # Attention backend
        if self.enable_pdmux:
            stream_idx = get_current_stream_idx()
            attn_backend = self.model_runner.decode_attn_backend_group[stream_idx]
        else:
```
**EN:** This callable implements `CudaGraphRunner.replay_prepare`. It takes `forward_batch`, `pp_proxy_tensors` and mainly prepares runtime inputs. This chunk is part 1 of 2 for the same logical block. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.replay_prepare`。它接收 `forward_batch`, `pp_proxy_tensors`，主要用于准备运行时输入。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 1240-1262: Method CudaGraphRunner.replay_prepare (part 2/2)
```python
            attn_backend = self.attn_backend
        # FIXME: implicit channel for backends (dsv4) that need forward_batch
        # in replay metadata prep. Should become a real param on the interface.
        attn_backend._replay_forward_batch = forward_batch
        attn_backend.init_forward_metadata_replay_cuda_graph(
            bs,
            buffers.req_pool_indices[:bs],
            buffers.seq_lens[:bs],
            forward_batch.seq_lens_sum + (bs - raw_bs) * self.seq_len_fill_value,
            buffers.encoder_lens[:bs] if self.is_encoder_decoder else None,
            self.capture_forward_mode,
            forward_batch.spec_info,
            seq_lens_cpu=buffers.seq_lens_cpu[:bs],
        )
        attn_backend._replay_forward_batch = None

        # Store fields
        self.raw_bs = raw_bs
        self.raw_num_token = raw_num_token
        self.bs = bs

        if self.model_runner.hisparse_coordinator is not None:
            self.model_runner.hisparse_coordinator.num_real_reqs.fill_(raw_bs)
```
**EN:** This callable implements `CudaGraphRunner.replay_prepare`. It takes `forward_batch`, `pp_proxy_tensors` and mainly prepares runtime inputs. This chunk is part 2 of 2 for the same logical block. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.replay_prepare`。它接收 `forward_batch`, `pp_proxy_tensors`，主要用于准备运行时输入。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 1264-1323: Method CudaGraphRunner.replay (part 1/2)
```python
    def replay(
        self,
        forward_batch: ForwardBatch,
        skip_attn_backend_init: bool = False,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
    ) -> Union[LogitsProcessorOutput, PPProxyTensors]:
        self.deepep_adapter.replay()

        if not skip_attn_backend_init:
            self.replay_prepare(forward_batch, pp_proxy_tensors)
        else:
            # In speculative decoding, these two fields are still needed.
            self.buffers.input_ids[: self.raw_num_token].copy_(forward_batch.input_ids)
            self.buffers.positions[: self.raw_num_token].copy_(forward_batch.positions)
            if (
                self.model_runner.spec_algorithm.is_dflash()
                and self.model_runner.is_draft_worker
                and forward_batch.input_embeds is not None
            ):
                self.buffers.input_embeds[: self.raw_num_token].copy_(
                    forward_batch.input_embeds
                )

        # Replay
        if self.enable_pdmux:
            graph_key = f"{get_current_stream_idx()}_{self.bs}"
        else:
            graph_key = self.bs
        ctx = (
            self.model_runner.device_timer.wrap(
                metadata={
                    "category": forward_batch.forward_mode.name.lower(),
                }
            )
            if self.model_runner.device_timer
            else contextlib.nullcontext()
        )
        with ctx:
            self.graphs[graph_key].replay()

        output = self.output_buffers[graph_key]

        if isinstance(output, LogitsProcessorOutput):
            if self.is_dllm:
                next_token_logits = None
                full_logits = (
                    output.full_logits[: self.raw_num_token]
                    if output.full_logits is not None
                    else None
                )
            else:
                full_logits = None
                next_token_logits = (
                    output.next_token_logits[: self.raw_num_token]
                    if output.next_token_logits is not None
                    else None
                )

            return LogitsProcessorOutput(
                next_token_logits=next_token_logits,
```
**EN:** This callable implements `CudaGraphRunner.replay`. It takes `forward_batch`, `skip_attn_backend_init`, `pp_proxy_tensors` and mainly implements replay. This chunk is part 1 of 2 for the same logical block. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.replay`。它接收 `forward_batch`, `skip_attn_backend_init`, `pp_proxy_tensors`，主要用于实现 replay 相关逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 1324-1334: Method CudaGraphRunner.replay (part 2/2)
```python
                full_logits=full_logits,
                hidden_states=(
                    output.hidden_states[: self.raw_num_token]
                    if output.hidden_states is not None
                    else None
                ),
                customized_info=output.customized_info,
            )
        else:
            assert isinstance(output, PPProxyTensors)
            return PPProxyTensors({k: v[: self.bs] for k, v in output.tensors.items()})
```
**EN:** This callable implements `CudaGraphRunner.replay`. It takes `forward_batch`, `skip_attn_backend_init`, `pp_proxy_tensors` and mainly implements replay. This chunk is part 2 of 2 for the same logical block. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.replay`。它接收 `forward_batch`, `skip_attn_backend_init`, `pp_proxy_tensors`，主要用于实现 replay 相关逻辑。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 1336-1395: Method CudaGraphRunner.get_spec_info (part 1/2)
```python
    def get_spec_info(self, num_tokens: int):
        spec_info = None
        if (
            self.model_runner.spec_algorithm.is_eagle()
            or self.model_runner.spec_algorithm.is_standalone()
        ):
            from sglang.srt.speculative.eagle_info import EagleVerifyInput

            if self.model_runner.is_draft_worker:
                raise RuntimeError("This should not happen.")
            else:

                capture_mode = (
                    CaptureHiddenMode.NULL
                    if self.model_runner.spec_algorithm.is_standalone()
                    else CaptureHiddenMode.FULL
                )
                spec_info = EagleVerifyInput(
                    draft_token=None,
                    custom_mask=self.buffers.custom_mask,
                    positions=None,
                    retrieve_index=None,
                    retrieve_next_token=None,
                    retrieve_next_sibling=None,
                    retrieve_cum_len=None,
                    spec_steps=self.speculative_num_steps,
                    topk=self.model_runner.server_args.speculative_eagle_topk,
                    draft_token_num=self.speculative_num_draft_tokens,
                    capture_hidden_mode=capture_mode,
                    seq_lens_sum=None,
                    seq_lens_cpu=None,
                )
        elif self.model_runner.spec_algorithm.is_dflash():
            from sglang.srt.speculative.dflash_info import DFlashVerifyInput
            from sglang.srt.speculative.dflash_utils import (
                resolve_dflash_verify_mask_policy,
            )

            # Avoid enabling custom-mask modes during graph capture for backends that
            # can express DFLASH verify via their built-in causal path.
            _, build_custom_mask = resolve_dflash_verify_mask_policy(
                self.model_runner.attn_backend
            )
            spec_info = DFlashVerifyInput(
                draft_token=None,
                positions=None,
                draft_token_num=self.model_runner.server_args.speculative_num_draft_tokens,
                custom_mask=(
                    None
                    if (self.model_runner.is_draft_worker or not build_custom_mask)
                    else self.buffers.custom_mask
                ),
                capture_hidden_mode=(
                    CaptureHiddenMode.NULL
                    if self.model_runner.is_draft_worker
                    else CaptureHiddenMode.FULL
                ),
            )

        elif self.model_runner.spec_algorithm.is_ngram():
```
**EN:** This callable implements `CudaGraphRunner.get_spec_info`. It takes `num_tokens` and mainly retrieves a value or derived view. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.get_spec_info`。它接收 `num_tokens`，主要用于获取某个值或派生视图。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；管理图捕获或回放逻辑。

### Lines 1396-1409: Method CudaGraphRunner.get_spec_info (part 2/2)
```python
            from sglang.srt.speculative.ngram_info import NgramVerifyInput

            spec_info = NgramVerifyInput(
                draft_token=None,
                tree_mask=self.buffers.custom_mask,
                positions=None,
                retrieve_index=None,
                retrieve_next_token=None,
                retrieve_next_sibling=None,
                draft_token_num=self.num_tokens_per_bs,
            )
            spec_info.capture_hidden_mode = CaptureHiddenMode.NULL

        return spec_info
```
**EN:** This callable implements `CudaGraphRunner.get_spec_info`. It takes `num_tokens` and mainly retrieves a value or derived view. This chunk is part 2 of 2 for the same logical block. In this range it sets up imports and shared symbols; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CudaGraphRunner.get_spec_info`。它接收 `num_tokens`，主要用于获取某个值或派生视图。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；管理图捕获或回放逻辑。

### Lines 1410-1421: Module-level constants and helpers
```python


CUDA_GRAPH_CAPTURE_FAILED_MSG = (
    "Possible solutions:\n"
    "1. set --mem-fraction-static to a smaller value (e.g., 0.8 or 0.7)\n"
    "2. set --cuda-graph-max-bs to a smaller value (e.g., 16)\n"
    "3. disable torch compile by not using --enable-torch-compile\n"
    "4. disable CUDA graph by --disable-cuda-graph. (Not recommended. Huge performance loss)\n"
    "Open an issue on GitHub https://github.com/sgl-project/sglang/issues/new/choose \n"
)


```
**EN:** This range organizes module-level state and shared setup. In this range it prepares compilation-related behavior; manages graph capture or replay logic.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会处理与编译相关的行为；管理图捕获或回放逻辑。

### Lines 1422-1422: Class DeepEPCudaGraphRunnerAdapter
```python
class DeepEPCudaGraphRunnerAdapter:
```
**EN:** This range introduces `DeepEPCudaGraphRunnerAdapter` and defines the structure or metadata that its methods rely on. In this range it manages graph capture or replay logic.
**CN:** 这一段引入 `DeepEPCudaGraphRunnerAdapter`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 1423-1425: Method DeepEPCudaGraphRunnerAdapter.__init__
```python
    def __init__(self):
        # Record DeepEP mode used during capture to ensure replay consistency
        self._captured_deepep_mode = None
```
**EN:** This callable implements `DeepEPCudaGraphRunnerAdapter.__init__` and mainly initializes instance state and defaults. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `DeepEPCudaGraphRunnerAdapter.__init__`，主要用于初始化实例状态与默认值。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 1427-1433: Method DeepEPCudaGraphRunnerAdapter.capture
```python
    def capture(self, is_extend_in_batch: bool):
        if not get_moe_a2a_backend().is_deepep():
            return
        self._captured_deepep_mode = get_deepep_mode().resolve(
            is_extend_in_batch=is_extend_in_batch
        )
        DeepEPBuffer.set_dispatch_mode(self._captured_deepep_mode)
```
**EN:** This callable implements `DeepEPCudaGraphRunnerAdapter.capture`. It takes `is_extend_in_batch` and mainly implements capture. In this range it manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `DeepEPCudaGraphRunnerAdapter.capture`。它接收 `is_extend_in_batch`，主要用于实现 capture 相关逻辑。 在这一范围内，它会管理图捕获或回放逻辑。

### Lines 1435-1439: Method DeepEPCudaGraphRunnerAdapter.replay
```python
    def replay(self):
        if not get_moe_a2a_backend().is_deepep():
            return
        assert self._captured_deepep_mode is not None
        DeepEPBuffer.set_dispatch_mode(self._captured_deepep_mode)
```
**EN:** This callable implements `DeepEPCudaGraphRunnerAdapter.replay` and mainly implements replay. In this range it performs defensive checks on invalid state; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `DeepEPCudaGraphRunnerAdapter.replay`，主要用于实现 replay 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；管理图捕获或回放逻辑。

## Key Concepts / 关键概念
- `_grouped_foreach_copy_`: implements grouped foreach copy / 实现 grouped foreach copy 相关逻辑
- `DecodeInputBuffers`: core class or state container / 核心类或状态容器
- `get_is_capture_mode`: retrieves a value or derived view / 获取某个值或派生视图
- `compile_in_capture_mode`: implements compile in capture mode / 实现 compile in capture mode 相关逻辑
- `model_capture_mode`: implements model capture mode / 实现 model capture mode 相关逻辑
- `freeze_gc`: implements freeze gc / 实现 freeze gc 相关逻辑
- `_to_torch`: converts data into another representation / 将数据转换为另一种表示
- `patch_model`: implements patch model / 实现 patch model 相关逻辑
- `set_torch_compile_config`: converts data into another representation / 将数据转换为另一种表示
- `get_batch_sizes_to_capture`: converts data into another representation / 将数据转换为另一种表示

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `bisect`, `contextlib`, `gc`, `inspect`, `logging`, `os`, `dataclasses`, `functools`, `typing`
- **Third-party / 第三方**: `torch`, `tqdm`, `torch.profiler`, `kt_kernel`, `torch._dynamo.config`, `torch._inductor.config`
- **Internal modules / 内部模块**: `sglang.srt.batch_overlap.two_batch_overlap`, `sglang.srt.constants`, `sglang.srt.distributed`, `sglang.srt.distributed.device_communicators.pynccl_allocator`, `sglang.srt.distributed.parallel_state`, `sglang.srt.dllm.config`, `sglang.srt.environ`, `sglang.srt.layers.attention.nsa.utils`, `sglang.srt.layers.dp_attention`, `sglang.srt.layers.logits_processor`, `sglang.srt.layers.moe.token_dispatcher.deepep`, `sglang.srt.layers.moe.utils` + 13 more
