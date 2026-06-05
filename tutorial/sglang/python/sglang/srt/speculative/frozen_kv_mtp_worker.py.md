# frozen_kv_mtp_worker.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/frozen_kv_mtp_worker.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Frozen-KV MTP draft worker. / 该文件实现了推测解码基础设施，例如 worker、元数据对象、CUDA Graph 运行器或共享工具函数。它通过协调草稿模型执行与目标模型校验来提升吞吐。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60: Module header, imports, and shared constants (part 1/2)
```python
# Copyright 2026 SGLang Team
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
"""Frozen-KV MTP draft worker.

The assistant reads target KV only. It reuses EAGLE's verify input/output
contract, but owns the seed and recurrent draft loop because there is no
assistant-side KV extension.
"""

from __future__ import annotations

import logging
from typing import List, Optional, Tuple

import torch

from sglang.srt.layers.logits_processor import LogitsProcessorOutput
from sglang.srt.layers.moe.utils import (
    speculative_moe_a2a_backend_context,
    speculative_moe_backend_context,
)
from sglang.srt.layers.utils.logprob import add_output_logprobs_for_spec_v1
from sglang.srt.managers.schedule_batch import ScheduleBatch
from sglang.srt.managers.scheduler import GenerationBatchResult
from sglang.srt.managers.tp_worker import TpModelWorker
from sglang.srt.model_executor.forward_batch_info import (
    CaptureHiddenMode,
    ForwardBatch,
    ForwardMode,
)
from sglang.srt.model_executor.pool_configurator import MemoryPoolConfig
from sglang.srt.observability.req_time_stats import set_time_batch
from sglang.srt.observability.trace import get_global_tracing_enabled
from sglang.srt.server_args import ServerArgs
from sglang.srt.speculative.eagle_utils import (
    build_tree_kernel_efficient,
    organize_draft_results,
)
from sglang.srt.speculative.frozen_kv_mtp_info import (
    FrozenKVMTPContext,
    FrozenKVMTPDraftExtendInput,
    FrozenKVMTPDraftInput,
    FrozenKVMTPVerifyInput,
    FrozenKVMTPVerifyOutput,
)
from sglang.srt.speculative.frozen_kv_mtp_utils import (
    capture_for_decode,
    expand_for_topk_draft,
    frozen_kv_target_view,
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on. It is one chunk of a larger setup region.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。 这是较大初始化区域中的一部分。

### Lines 61-80: Module header, imports, and shared constants (part 2/2)
```python
    position_for_batch,
    select_last_extend_hidden,
    select_last_verified_seed,
    set_frozen_kv_positions,
    target_kv_pool_view,
)
from sglang.srt.speculative.spec_info import SpeculativeAlgorithm
from sglang.srt.speculative.spec_utils import (
    draft_tp_context,
    fast_topk,
    generate_token_bitmask,
    maybe_detect_nan,
    maybe_detect_oob,
    select_top_k_tokens,
)
from sglang.srt.utils import empty_context

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on. It is one chunk of a larger setup region.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。 这是较大初始化区域中的一部分。

### Lines 81-85: FrozenKVMTPWorker class declaration
```python
class FrozenKVMTPWorker(TpModelWorker):
    """Frozen-KV MTP worker; same constructor shape as EAGLEWorker. Entry:
    :meth:`forward_batch_generation` (stubs for now).
    """
```
**EN:** This block declares the `FrozenKVMTPWorker` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `FrozenKVMTPWorker` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 86-145: FrozenKVMTPWorker initializer (part 1/2)
```python
    def __init__(
        self,
        server_args: ServerArgs,
        gpu_id: int,
        tp_rank: int,
        dp_rank: Optional[int],
        moe_ep_rank: int,
        attn_cp_rank: int,
        moe_dp_rank: int,
        nccl_port: int,
        target_worker: TpModelWorker,
    ):
        self.server_args = server_args
        self.topk = server_args.speculative_eagle_topk
        self.speculative_num_steps = server_args.speculative_num_steps
        self.speculative_num_draft_tokens = server_args.speculative_num_draft_tokens
        self.gpu_id = gpu_id
        self.device = server_args.device
        self.target_worker = target_worker
        self.page_size = server_args.page_size
        self.speculative_algorithm = SpeculativeAlgorithm.from_string(
            server_args.speculative_algorithm
        )
        assert self.speculative_algorithm.is_frozen_kv_mtp(), (
            "FrozenKVMTPWorker should only be instantiated for "
            "SpeculativeAlgorithm.FROZEN_KV_MTP, got "
            f"{self.speculative_algorithm.name}. The dispatch happens in "
            "arg_groups.speculative_hook.handle_speculative_decoding -> "
            "_resolve_speculative_algorithm_alias."
        )

        # Assistant reads target KV directly, so its context length must match the target.
        server_args.context_length = target_worker.model_runner.model_config.context_len

        # Defer cuda graph capture; we do it ourselves below.
        backup_disable_cuda_graph = server_args.disable_cuda_graph
        server_args.disable_cuda_graph = True

        # Draft attention uses target req_to_token + KV allocator (read-only).
        self.req_to_token_pool, self.token_to_kv_pool_allocator = (
            target_worker.get_memory_pool()
        )

        target_cfg = target_worker.model_runner.memory_pool_config
        draft_pool_config = MemoryPoolConfig(
            max_total_num_tokens=64,  # Dummy value
            max_running_requests=target_cfg.max_running_requests,
        )

        self.hot_token_id = None

        with (
            empty_context()
        ), speculative_moe_backend_context(), speculative_moe_a2a_backend_context():
            super().__init__(
                server_args=server_args,
                gpu_id=gpu_id,
                tp_rank=tp_rank,
                pp_rank=0,
                dp_rank=dp_rank,
```
**EN:** This block initializes the `FrozenKVMTPWorker` object, setting up the state, buffers, and references that later methods rely on. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块初始化 `FrozenKVMTPWorker` 对象，建立后续方法依赖的状态、缓冲区与引用。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 146-187: FrozenKVMTPWorker initializer (part 2/2)
```python
                moe_ep_rank=moe_ep_rank,
                attn_cp_rank=attn_cp_rank,
                moe_dp_rank=moe_dp_rank,
                nccl_port=nccl_port,
                is_draft_worker=True,
                req_to_token_pool=self.req_to_token_pool,
                token_to_kv_pool_allocator=self.token_to_kv_pool_allocator,
                memory_pool_config=draft_pool_config,
            )

        embed, head = self.target_worker.model_runner.model.get_embed_and_head()
        if hasattr(self.draft_model_runner.model, "set_embed_and_head"):
            self.draft_model_runner.model.set_embed_and_head(embed, head)
        else:
            logger.debug(
                "Draft model %s does not implement set_embed_and_head; "
                "skipping target-embedding bind in Frozen-KV MTP skeleton.",
                type(self.draft_model_runner.model).__name__,
            )

        self.kv_context: Optional["FrozenKVMTPContext"] = None
        if hasattr(self.draft_model_runner.model, "bind_frozen_kv_context"):
            self._bind_kv_context()

        self.draft_model_runner.server_args.disable_cuda_graph = (
            backup_disable_cuda_graph
        )

        self.draft_tp_context = (
            draft_tp_context if server_args.enable_dp_attention else empty_context
        )

        self.draft_attn_backend = self._init_draft_attn_backend()
        self.draft_model_runner.draft_attn_backend = self.draft_attn_backend
        self.cuda_graph_runner = None

        with (
            self.draft_tp_context(self.draft_model_runner.tp_group),
            speculative_moe_backend_context(),
            speculative_moe_a2a_backend_context(),
        ):
            self.init_cuda_graphs()
```
**EN:** This block initializes the `FrozenKVMTPWorker` object, setting up the state, buffers, and references that later methods rely on. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块初始化 `FrozenKVMTPWorker` 对象，建立后续方法依赖的状态、缓冲区与引用。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 189-191: FrozenKVMTPWorker.draft_model_runner method
```python
    @property
    def draft_model_runner(self):
        return self.model_runner
```
**EN:** This block uses `FrozenKVMTPWorker.draft_model_runner` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker.draft_model_runner` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 193-194: FrozenKVMTPWorker.get_attn_backend method
```python
    def get_attn_backend(self):  # pragma: no cover - exposed for adaptive
        return self.draft_attn_backend
```
**EN:** This block uses `FrozenKVMTPWorker.get_attn_backend` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker.get_attn_backend` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 196-197: FrozenKVMTPWorker.clear_cache_pool method
```python
    def clear_cache_pool(self):
        pass
```
**EN:** This block uses `FrozenKVMTPWorker.clear_cache_pool` to clear cached state or buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker.clear_cache_pool` 来清理缓存状态或缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 199-204: FrozenKVMTPWorker._resolve_draft_backend_type method
```python
    def _resolve_draft_backend_type(self) -> str:
        return (
            self.server_args.speculative_draft_attention_backend
            or self.server_args.decode_attention_backend
            or self.server_args.attention_backend
        )
```
**EN:** This block uses `FrozenKVMTPWorker._resolve_draft_backend_type` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker._resolve_draft_backend_type` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 206-216: FrozenKVMTPWorker._init_draft_attn_backend method
```python
    def _init_draft_attn_backend(self):
        if self.topk == 1:
            return self.draft_model_runner.attn_backend

        backend_type = self._resolve_draft_backend_type()
        if backend_type != "triton":
            raise ValueError(
                "Frozen-KV MTP topk > 1 currently supports only the triton "
                f"attention backend, got {backend_type}."
            )
        return self._init_triton_draft_attn_backend()
```
**EN:** This block uses `FrozenKVMTPWorker._init_draft_attn_backend` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker._init_draft_attn_backend` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 218-229: FrozenKVMTPWorker._init_triton_draft_attn_backend method
```python
    def _init_triton_draft_attn_backend(self):
        from sglang.srt.layers.attention.triton_backend import TritonAttnBackend

        max_bs = self.req_to_token_pool.size * self.topk
        kv_indptr_buf = torch.zeros(
            (max_bs + 1,), dtype=torch.int32, device=self.draft_model_runner.device
        )
        return TritonAttnBackend(
            self.draft_model_runner,
            skip_prefill=True,
            kv_indptr_buf=kv_indptr_buf,
        )
```
**EN:** This block uses `FrozenKVMTPWorker._init_triton_draft_attn_backend` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker._init_triton_draft_attn_backend` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 231-248: FrozenKVMTPWorker._bind_kv_context method
```python
    def _bind_kv_context(self) -> None:
        draft_model = self.draft_model_runner.model
        if not hasattr(draft_model, "build_frozen_kv_mtp_context") or not hasattr(
            draft_model, "bind_frozen_kv_context"
        ):
            logger.debug(
                "Draft model %s does not implement Frozen-KV MTP context hooks; "
                "skipping frozen-kv bind.",
                type(draft_model).__name__,
            )
            return

        ctx = draft_model.build_frozen_kv_mtp_context(
            target_model=self.target_worker.model_runner.model,
            target_token_to_kv_pool=self.target_worker.model_runner.token_to_kv_pool,
        )
        draft_model.bind_frozen_kv_context(ctx)
        self.kv_context = ctx
```
**EN:** This block uses `FrozenKVMTPWorker._bind_kv_context` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker._bind_kv_context` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 250-251: FrozenKVMTPWorker._frozen_kv_target_view method
```python
    def _frozen_kv_target_view(self, forward_batch: ForwardBatch):
        return frozen_kv_target_view(forward_batch, self.kv_context)
```
**EN:** This block uses `FrozenKVMTPWorker._frozen_kv_target_view` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker._frozen_kv_target_view` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 253-254: FrozenKVMTPWorker._target_kv_pool_view method
```python
    def _target_kv_pool_view(self, forward_batch: ForwardBatch):
        return target_kv_pool_view(forward_batch, self.kv_context)
```
**EN:** This block uses `FrozenKVMTPWorker._target_kv_pool_view` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker._target_kv_pool_view` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 256-257: FrozenKVMTPWorker._set_positions method
```python
    def _set_positions(self, forward_batch: ForwardBatch) -> None:
        set_frozen_kv_positions(forward_batch, self.topk)
```
**EN:** This block uses `FrozenKVMTPWorker._set_positions` to update stored state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker._set_positions` 来更新已保存的状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 259-260: FrozenKVMTPWorker._expand_for_topk_draft method
```python
    def _expand_for_topk_draft(self, forward_batch: ForwardBatch) -> None:
        expand_for_topk_draft(forward_batch, self.topk)
```
**EN:** This block uses `FrozenKVMTPWorker._expand_for_topk_draft` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker._expand_for_topk_draft` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 262-263: FrozenKVMTPWorker._position_for_batch method
```python
    def _position_for_batch(self, batch: ScheduleBatch) -> torch.Tensor:
        return position_for_batch(batch)
```
**EN:** This block uses `FrozenKVMTPWorker._position_for_batch` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker._position_for_batch` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 265-267: FrozenKVMTPWorker._recurrent_hidden_size method
```python
    @property
    def _recurrent_hidden_size(self) -> int:
        return int(self.draft_model_runner.model.backbone_hidden_size)
```
**EN:** This block uses `FrozenKVMTPWorker._recurrent_hidden_size` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker._recurrent_hidden_size` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 269-278: FrozenKVMTPWorker._init_frozen_kv_metadata method
```python
    def _init_frozen_kv_metadata(self, forward_batch: ForwardBatch) -> None:
        if forward_batch.forward_mode.is_idle():
            return
        if forward_batch.seq_lens_cpu is not None:
            forward_batch.seq_lens_sum = forward_batch.seq_lens_cpu.sum().item()
        else:
            forward_batch.seq_lens_sum = torch.sum(forward_batch.seq_lens).item()
        with self._frozen_kv_target_view(forward_batch):
            self.draft_attn_backend.init_forward_metadata(forward_batch)
        forward_batch.attn_backend = self.draft_attn_backend
```
**EN:** This block uses `FrozenKVMTPWorker._init_frozen_kv_metadata` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker._init_frozen_kv_metadata` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 280-293: FrozenKVMTPWorker._init_frozen_kv_metadata_capture_cuda_graph method
```python
    def _init_frozen_kv_metadata_capture_cuda_graph(
        self, forward_batch: ForwardBatch
    ) -> None:
        with self._frozen_kv_target_view(forward_batch):
            self.draft_attn_backend.init_forward_metadata_capture_cuda_graph(
                forward_batch.batch_size,
                forward_batch.positions.numel(),
                forward_batch.req_pool_indices,
                forward_batch.seq_lens,
                encoder_lens=None,
                forward_mode=ForwardMode.DECODE,
                spec_info=None,
            )
        forward_batch.attn_backend = self.draft_attn_backend
```
**EN:** This block uses `FrozenKVMTPWorker._init_frozen_kv_metadata_capture_cuda_graph` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker._init_frozen_kv_metadata_capture_cuda_graph` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 295-313: FrozenKVMTPWorker._init_frozen_kv_metadata_replay_cuda_graph method
```python
    def _init_frozen_kv_metadata_replay_cuda_graph(
        self, forward_batch: ForwardBatch, bs: int, seq_lens_sum: int
    ) -> None:
        with self._frozen_kv_target_view(forward_batch):
            self.draft_attn_backend.init_forward_metadata_replay_cuda_graph(
                bs,
                forward_batch.req_pool_indices[:bs],
                forward_batch.seq_lens[:bs],
                seq_lens_sum,
                encoder_lens=None,
                forward_mode=ForwardMode.DECODE,
                spec_info=None,
                seq_lens_cpu=(
                    forward_batch.seq_lens_cpu[:bs]
                    if forward_batch.seq_lens_cpu is not None
                    else None
                ),
            )
        forward_batch.attn_backend = self.draft_attn_backend
```
**EN:** This block uses `FrozenKVMTPWorker._init_frozen_kv_metadata_replay_cuda_graph` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker._init_frozen_kv_metadata_replay_cuda_graph` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 315-332: FrozenKVMTPWorker.init_cuda_graphs method
```python
    def init_cuda_graphs(self) -> None:
        if self.server_args.disable_cuda_graph or self.speculative_num_steps <= 1:
            return
        if self.target_worker.device != "cuda":
            logger.info(
                "Frozen-KV MTP draft CUDA graph is only supported on CUDA; "
                "running the draft loop eagerly on %s.",
                self.target_worker.device,
            )
            return

        from sglang.srt.speculative.frozen_kv_mtp_cuda_graph_runner import (
            FrozenKVMTPCudaGraphRunner,
        )

        logger.info("Capture Frozen-KV MTP draft cuda graph begin.")
        self.cuda_graph_runner = FrozenKVMTPCudaGraphRunner(self)
        logger.info("Capture Frozen-KV MTP draft cuda graph end.")
```
**EN:** This block uses `FrozenKVMTPWorker.init_cuda_graphs` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker.init_cuda_graphs` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 334-337: FrozenKVMTPWorker._select_last_extend_hidden method
```python
    def _select_last_extend_hidden(
        self, batch: ScheduleBatch, hidden_states: torch.Tensor
    ) -> torch.Tensor:
        return select_last_extend_hidden(batch, hidden_states)
```
**EN:** This block uses `FrozenKVMTPWorker._select_last_extend_hidden` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker._select_last_extend_hidden` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 339-342: FrozenKVMTPWorker._select_last_verified_seed method
```python
    def _select_last_verified_seed(
        self, draft_input: FrozenKVMTPDraftExtendInput
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        return select_last_verified_seed(draft_input)
```
**EN:** This block uses `FrozenKVMTPWorker._select_last_verified_seed` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker._select_last_verified_seed` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 344-347: FrozenKVMTPWorker._capture_for_decode method
```python
    def _capture_for_decode(
        self, logits_output: LogitsProcessorOutput, draft_input: FrozenKVMTPDraftInput
    ) -> None:
        capture_for_decode(logits_output, draft_input, self.topk)
```
**EN:** This block uses `FrozenKVMTPWorker._capture_for_decode` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker._capture_for_decode` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 349-408: FrozenKVMTPWorker._run_assistant_seed_step method (part 1/2)
```python
    def _run_assistant_seed_step(
        self,
        batch: ScheduleBatch,
        last_token_ids: torch.Tensor,
        last_hidden_states: torch.Tensor,
        seq_lens_cpu: Optional[torch.Tensor] = None,
        mm_input_embeds: Optional[torch.Tensor] = None,
        draft_input: Optional[FrozenKVMTPDraftInput] = None,
    ) -> None:
        """Run the one-token assistant seed step against frozen target KV."""
        if batch.forward_mode.is_idle() or last_token_ids.numel() == 0:
            batch.spec_info = FrozenKVMTPDraftInput.create_idle_input(
                device=batch.device,
                hidden_size=self._recurrent_hidden_size,
                dtype=self.model_config.dtype,
                topk=self.topk,
                capture_hidden_mode=CaptureHiddenMode.LAST,
            )
            return

        if draft_input is None:
            draft_input = FrozenKVMTPDraftInput()

        draft_input.bonus_tokens = last_token_ids.to(torch.int64)
        draft_input.hidden_states = last_hidden_states
        draft_input.capture_hidden_mode = CaptureHiddenMode.LAST
        draft_input.num_tokens_per_req = 1
        draft_input.num_tokens_for_logprob_per_req = 1
        draft_input.positions = self._position_for_batch(batch)

        forward_mode_backup = batch.forward_mode
        input_ids_backup = batch.input_ids
        return_hidden_states_backup = batch.return_hidden_states
        return_logprob_backup = batch.return_logprob
        spec_info_backup = batch.spec_info

        batch.forward_mode = ForwardMode.DECODE
        batch.input_ids = draft_input.bonus_tokens
        batch.return_hidden_states = False
        batch.return_logprob = False
        batch.spec_info = draft_input

        try:
            batch.seq_lens_cpu_cache = seq_lens_cpu
            forward_batch = ForwardBatch.init_new(batch, self.draft_model_runner)
            forward_batch.return_logprob = False
            if mm_input_embeds is not None:
                forward_batch.mm_input_embeds = mm_input_embeds
            self._set_positions(forward_batch)
            self._init_frozen_kv_metadata(forward_batch)
            with self._target_kv_pool_view(forward_batch):
                logits_output = self.draft_model_runner.forward(
                    forward_batch, skip_attn_backend_init=True
                ).logits_output
            maybe_detect_nan(logits_output.next_token_logits, "frozen_kv_mtp_seed")
            self._capture_for_decode(logits_output, draft_input)
        finally:
            batch.forward_mode = forward_mode_backup
            batch.input_ids = input_ids_backup
            batch.return_hidden_states = return_hidden_states_backup
```
**EN:** This block uses `FrozenKVMTPWorker._run_assistant_seed_step` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `FrozenKVMTPWorker._run_assistant_seed_step` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 409-413: FrozenKVMTPWorker._run_assistant_seed_step method (part 2/2)
```python
            batch.return_logprob = return_logprob_backup
            # Keep the seeded draft state; only restore the old object on error paths
            # before the assignment above could have happened.
            if batch.spec_info is not draft_input:
                batch.spec_info = spec_info_backup
```
**EN:** This block uses `FrozenKVMTPWorker._run_assistant_seed_step` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `FrozenKVMTPWorker._run_assistant_seed_step` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 415-474: FrozenKVMTPWorker.forward_batch_generation method (part 1/2)
```python
    def forward_batch_generation(self, batch: ScheduleBatch) -> GenerationBatchResult:
        if batch.forward_mode.is_extend() or batch.is_extend_in_batch:
            (
                logits_output,
                next_token_ids,
                seq_lens_cpu,
                can_run_cuda_graph,
            ) = self.forward_target_extend(batch)
            with (
                self.draft_tp_context(self.draft_model_runner.tp_group),
                speculative_moe_backend_context(),
                speculative_moe_a2a_backend_context(),
            ):
                self.forward_draft_extend(
                    batch,
                    logits_output.hidden_states,
                    next_token_ids,
                    seq_lens_cpu,
                    logits_output.mm_input_embeds,
                )
            return GenerationBatchResult(
                logits_output=logits_output,
                next_token_ids=next_token_ids,
                num_correct_drafts=0,
                can_run_cuda_graph=can_run_cuda_graph,
            )

        set_time_batch(batch.reqs, "set_spec_draft_start_time", trace_only=True)
        with (
            self.draft_tp_context(self.draft_model_runner.tp_group),
            speculative_moe_backend_context(),
            speculative_moe_a2a_backend_context(),
        ):
            verify_input = self.draft(batch)
        set_time_batch(batch.reqs, "set_spec_draft_end_time", trace_only=True)
        set_time_batch(batch.reqs, "set_spec_verify_start_time", trace_only=True)

        batch.spec_info = verify_input
        logits_output, verify_output, can_run_cuda_graph = self.verify(batch)

        if get_global_tracing_enabled():
            for idx, req in enumerate(batch.reqs):
                num_correct_drafts = verify_output.num_correct_drafts_per_req_cpu[idx]
                req.time_stats.set_spec_verify_end_time(
                    num_correct_drafts=num_correct_drafts
                )

        set_time_batch(batch.reqs, "set_spec_draft_extend_start_time", trace_only=True)
        with (
            self.draft_tp_context(self.draft_model_runner.tp_group),
            speculative_moe_backend_context(),
            speculative_moe_a2a_backend_context(),
        ):
            draft_extend_input = verify_output.draft_extend_input
            if (
                self.server_args.enable_dp_attention
                or draft_extend_input.input_ids.shape[0] > 0
            ):
                # Stash for the seed step; _run_assistant_seed_step swaps in
                # a fresh FrozenKVMTPDraftInput for next iter.
```
**EN:** This block uses `FrozenKVMTPWorker.forward_batch_generation` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `FrozenKVMTPWorker.forward_batch_generation` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 475-485: FrozenKVMTPWorker.forward_batch_generation method (part 2/2)
```python
                batch.spec_info = draft_extend_input
                self.forward_draft_extend_after_decode(batch)
        set_time_batch(batch.reqs, "set_spec_draft_extend_end_time", trace_only=True)

        return GenerationBatchResult(
            logits_output=logits_output,
            next_token_ids=verify_output.accept_tokens,
            num_correct_drafts=sum(verify_output.num_correct_drafts_per_req_cpu),
            num_correct_drafts_per_req_cpu=verify_output.num_correct_drafts_per_req_cpu,
            can_run_cuda_graph=can_run_cuda_graph,
        )
```
**EN:** This block uses `FrozenKVMTPWorker.forward_batch_generation` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `FrozenKVMTPWorker.forward_batch_generation` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 487-497: FrozenKVMTPWorker.forward_target_extend method
```python
    def forward_target_extend(
        self, batch: ScheduleBatch
    ) -> Tuple[LogitsProcessorOutput, torch.Tensor, Optional[torch.Tensor], bool]:
        batch.capture_hidden_mode = CaptureHiddenMode.FULL
        batch_result = self.target_worker.forward_batch_generation(batch)
        return (
            batch_result.logits_output,
            batch_result.next_token_ids,
            batch.seq_lens_cpu,
            batch_result.can_run_cuda_graph,
        )
```
**EN:** This block uses `FrozenKVMTPWorker.forward_target_extend` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker.forward_target_extend` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 499-514: FrozenKVMTPWorker.forward_draft_extend method
```python
    def forward_draft_extend(
        self,
        batch: ScheduleBatch,
        hidden_states: torch.Tensor,
        next_token_ids: torch.Tensor,
        seq_lens_cpu: Optional[torch.Tensor],
        mm_input_embeds: Optional[torch.Tensor] = None,
    ) -> None:
        last_hidden = self._select_last_extend_hidden(batch, hidden_states)
        self._run_assistant_seed_step(
            batch,
            next_token_ids,
            last_hidden,
            seq_lens_cpu=seq_lens_cpu,
            mm_input_embeds=mm_input_embeds,
        )
```
**EN:** This block uses `FrozenKVMTPWorker.forward_draft_extend` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker.forward_draft_extend` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 516-562: FrozenKVMTPWorker.forward_draft_extend_after_decode method
```python
    def forward_draft_extend_after_decode(self, batch: ScheduleBatch) -> None:
        draft_extend_input: FrozenKVMTPDraftExtendInput = batch.spec_info
        input_is_idle = batch.forward_mode.is_idle()

        if not input_is_idle and draft_extend_input.input_ids.shape[0] == 0:
            # All reqs finished; stash an idle FrozenKVMTPDraftInput so the
            # next-iter draft sees a valid spec_info.
            batch = batch.copy()
            batch.prepare_for_idle()
            batch.spec_info = FrozenKVMTPDraftInput.create_idle_input(
                device=self.device,
                hidden_size=self._recurrent_hidden_size,
                dtype=self.model_config.dtype,
                topk=self.topk,
                capture_hidden_mode=CaptureHiddenMode.LAST,
            )
            return

        if batch.forward_mode.is_idle():
            return

        seq_lens_backup = batch.seq_lens.clone()
        seq_lens_cpu_backup = batch.seq_lens_cpu.clone()
        req_pool_indices_backup = batch.req_pool_indices

        try:
            # Verify may leave finished requests in ScheduleBatch; seed only
            # the unfinished reqs carried by `draft_extend_input`.
            batch.seq_lens = draft_extend_input.seq_lens
            batch.seq_lens_cpu = draft_extend_input.seq_lens_cpu
            batch.req_pool_indices = draft_extend_input.req_pool_indices

            last_token_ids, last_hidden = self._select_last_verified_seed(
                draft_extend_input
            )
            # `_run_assistant_seed_step` constructs a fresh `FrozenKVMTPDraftInput`
            # and installs it on `batch.spec_info` for next iter.
            self._run_assistant_seed_step(
                batch,
                last_token_ids,
                last_hidden,
                seq_lens_cpu=draft_extend_input.seq_lens_cpu,
            )
        finally:
            batch.seq_lens = seq_lens_backup
            batch.seq_lens_cpu = seq_lens_cpu_backup
            batch.req_pool_indices = req_pool_indices_backup
```
**EN:** This block uses `FrozenKVMTPWorker.forward_draft_extend_after_decode` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker.forward_draft_extend_after_decode` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 564-623: FrozenKVMTPWorker.draft method (part 1/2)
```python
    def draft(self, batch: ScheduleBatch):
        if batch.forward_mode.is_idle():
            return FrozenKVMTPVerifyInput.create_idle_input(
                self.topk,
                self.speculative_num_steps,
                self.speculative_num_draft_tokens,
            )

        batch.maybe_evict_swa()
        for req in batch.reqs:
            req.decode_batch_idx += 1

        spec_info = batch.spec_info
        assert isinstance(spec_info, FrozenKVMTPDraftInput)

        if batch.sampling_info.penalizer_orchestrator.is_required:
            batch.sampling_info.penalizer_orchestrator.cumulate_output_tokens(
                spec_info.bonus_tokens.to(torch.int64)
            )

        spec_info.capture_hidden_mode = CaptureHiddenMode.LAST
        spec_info.num_tokens_per_req = self.topk
        spec_info.num_tokens_for_logprob_per_req = self.topk
        spec_info.positions = self._position_for_batch(batch)
        batch.seq_lens_sum = torch.sum(batch.seq_lens).item()
        batch.return_hidden_states = False

        forward_batch = ForwardBatch.init_new(batch, self.draft_model_runner)
        assert forward_batch.capture_hidden_mode == CaptureHiddenMode.LAST
        self._set_positions(forward_batch)
        self._expand_for_topk_draft(forward_batch)

        can_run_cuda_graph = self.cuda_graph_runner and self.cuda_graph_runner.can_run(
            forward_batch
        )
        if can_run_cuda_graph:
            parent_list, top_scores_index, draft_tokens = self.cuda_graph_runner.replay(
                forward_batch
            )
        else:
            forward_batch.can_run_dp_cuda_graph = False
            parent_list, top_scores_index, draft_tokens = self.draft_forward(
                forward_batch
            )

        (
            tree_mask,
            position,
            retrieve_index,
            retrieve_next_token,
            retrieve_next_sibling,
            draft_tokens,
        ) = build_tree_kernel_efficient(
            spec_info.bonus_tokens,
            parent_list,
            top_scores_index,
            draft_tokens,
            batch.seq_lens,
            batch.seq_lens_sum,
            self.topk,
```
**EN:** This block uses `FrozenKVMTPWorker.draft` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `FrozenKVMTPWorker.draft` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 624-642: FrozenKVMTPWorker.draft method (part 2/2)
```python
            self.speculative_num_steps,
            self.speculative_num_draft_tokens,
        )

        return FrozenKVMTPVerifyInput(
            draft_token=draft_tokens,
            custom_mask=tree_mask,
            positions=position,
            retrieve_index=retrieve_index,
            retrieve_next_token=retrieve_next_token,
            retrieve_next_sibling=retrieve_next_sibling,
            retrieve_cum_len=None,
            spec_steps=self.speculative_num_steps,
            topk=self.topk,
            draft_token_num=self.speculative_num_draft_tokens,
            capture_hidden_mode=CaptureHiddenMode.FULL,
            seq_lens_sum=batch.seq_lens_sum,
            seq_lens_cpu=batch.seq_lens_cpu,
        )
```
**EN:** This block uses `FrozenKVMTPWorker.draft` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `FrozenKVMTPWorker.draft` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 644-699: FrozenKVMTPWorker.draft_forward method
```python
    def draft_forward(
        self, forward_batch: ForwardBatch, skip_attn_backend_init: bool = False
    ):
        spec_info = forward_batch.spec_info
        assert isinstance(spec_info, FrozenKVMTPDraftInput)
        topk_p, topk_index, hidden_states = (
            spec_info.topk_p,
            spec_info.topk_index,
            spec_info.hidden_states,
        )
        maybe_detect_nan(topk_p, "frozen_kv_mtp_draft: initial topk_p")

        score_list: List[torch.Tensor] = []
        token_list: List[torch.Tensor] = []
        parents_list: List[torch.Tensor] = []

        if not skip_attn_backend_init and self.speculative_num_steps > 1:
            self._init_frozen_kv_metadata(forward_batch)

        scores = None
        for i in range(self.speculative_num_steps):
            input_ids, hidden_states, scores, tree_info = select_top_k_tokens(
                i, topk_p, topk_index, hidden_states, scores, self.topk
            )
            score_list.append(tree_info[0])
            token_list.append(tree_info[1])
            parents_list.append(tree_info[2])

            if i == self.speculative_num_steps - 1:
                break

            forward_batch.input_ids = input_ids
            forward_batch.spec_info.hidden_states = hidden_states
            self._set_positions(forward_batch)

            with self._target_kv_pool_view(forward_batch):
                logits_output = self.draft_model_runner.forward(
                    forward_batch, skip_attn_backend_init=True
                ).logits_output

            maybe_detect_nan(
                logits_output.next_token_logits, f"frozen_kv_mtp_draft step {i}"
            )
            probs = torch.softmax(logits_output.next_token_logits, dim=-1)
            topk_p, topk_index = fast_topk(probs, self.topk, dim=-1)
            maybe_detect_oob(
                topk_index,
                0,
                logits_output.next_token_logits.shape[-1],
                "frozen_kv_mtp_draft: topk_index OOB",
            )
            hidden_states = logits_output.hidden_states

        return organize_draft_results(
            score_list, token_list, parents_list, self.speculative_num_draft_tokens
        )
```
**EN:** This block uses `FrozenKVMTPWorker.draft_forward` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPWorker.draft_forward` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 701-760: FrozenKVMTPWorker.verify method (part 1/2)
```python
    def verify(self, batch: ScheduleBatch):
        spec_info: FrozenKVMTPVerifyInput = batch.spec_info
        seq_lens_pre_verify = batch.seq_lens.clone()
        spec_info.prepare_for_verify(batch, self.page_size)
        spec_info.num_tokens_per_req = self.speculative_num_steps + 1
        batch.return_hidden_states = False
        batch.forward_mode = (
            ForwardMode.TARGET_VERIFY
            if not batch.forward_mode.is_idle()
            else ForwardMode.IDLE
        )

        if batch.has_grammar:
            retrieve_next_token_cpu = spec_info.retrieve_next_token.cpu()
            retrieve_next_sibling_cpu = spec_info.retrieve_next_sibling.cpu()
            draft_tokens_cpu = spec_info.draft_token.view(
                spec_info.retrieve_next_token.shape
            ).cpu()

        batch.seq_lens_cpu_cache = spec_info.seq_lens_cpu
        batch_result = self.target_worker.forward_batch_generation(
            batch, is_verify=True
        )
        logits_output, can_run_cuda_graph = (
            batch_result.logits_output,
            batch_result.can_run_cuda_graph,
        )

        vocab_mask = None
        if batch.has_grammar:
            vocab_mask = generate_token_bitmask(
                batch.reqs,
                spec_info,
                retrieve_next_token_cpu,
                retrieve_next_sibling_cpu,
                draft_tokens_cpu,
                batch.sampling_info.vocab_size,
            )
            if vocab_mask is not None:
                assert spec_info.grammar is not None
                vocab_mask = vocab_mask.to(spec_info.retrieve_next_token.device)
                batch.sampling_info.vocab_mask = None

        maybe_detect_nan(logits_output.next_token_logits, "frozen_kv_mtp_verify")

        spec_info.hidden_states = logits_output.hidden_states
        res: FrozenKVMTPVerifyOutput = spec_info.verify(
            batch,
            logits_output,
            self.token_to_kv_pool_allocator,
            self.page_size,
            vocab_mask,
        )

        logits_output.next_token_logits = logits_output.next_token_logits[
            res.accept_indices
        ]
        logits_output.hidden_states = logits_output.hidden_states[res.accept_indices]

        if (
```
**EN:** This block uses `FrozenKVMTPWorker.verify` to verify draft results against the target model. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `FrozenKVMTPWorker.verify` 来根据目标模型校验草稿结果。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 761-778: FrozenKVMTPWorker.verify method (part 2/2)
```python
            self.target_worker.model_runner.hybrid_gdn_config is not None
            or self.target_worker.model_runner.mamba2_config is not None
            or self.target_worker.model_runner.hybrid_lightning_config is not None
        ):
            logger.warning(
                "Frozen-KV MTP does not implement mamba state updates; "
                "targets with recurrent state should not use this path."
            )

        if batch.return_logprob:
            add_output_logprobs_for_spec_v1(batch, res, logits_output)

        batch.forward_mode = (
            ForwardMode.DECODE if not batch.forward_mode.is_idle() else ForwardMode.IDLE
        )

        del seq_lens_pre_verify
        return logits_output, res, can_run_cuda_graph
```
**EN:** This block uses `FrozenKVMTPWorker.verify` to verify draft results against the target model. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `FrozenKVMTPWorker.verify` 来根据目标模型校验草稿结果。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

## Key Concepts / 关键概念
- Worker coordination / Worker 协调
- Speculative decoding / 推测解码
- Frozen-KV MTP path / Frozen-KV MTP 路径
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算
- Graph-captured execution paths / 图捕获执行路径
- Integration with model workers / 与模型 worker 集成

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.layers.attention.triton_backend`
- `sglang.srt.layers.logits_processor`
- `sglang.srt.layers.moe.utils`
- `sglang.srt.layers.utils.logprob`
- `sglang.srt.managers.schedule_batch`
- `sglang.srt.managers.scheduler`
- `sglang.srt.managers.tp_worker`
- `sglang.srt.model_executor.forward_batch_info`
- `sglang.srt.model_executor.pool_configurator`
- `sglang.srt.observability.req_time_stats`
- `sglang.srt.observability.trace`
- `sglang.srt.server_args`
- `sglang.srt.speculative.eagle_utils`
- `sglang.srt.speculative.frozen_kv_mtp_cuda_graph_runner`
- `sglang.srt.speculative.frozen_kv_mtp_info`
- `sglang.srt.speculative.frozen_kv_mtp_utils`
- `sglang.srt.speculative.spec_info`
- `sglang.srt.speculative.spec_utils`
- `sglang.srt.utils`
### External / 外部
- `__future__`
- `torch`
- `logging` (stdlib)
- `typing` (stdlib)
