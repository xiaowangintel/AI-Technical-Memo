# multi_layer_eagle_worker_v2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/multi_layer_eagle_worker_v2.py`
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

import contextlib
import logging
from typing import TYPE_CHECKING, List, Optional, Tuple

import torch

from sglang.srt.environ import envs
from sglang.srt.layers.moe.utils import speculative_moe_backend_context
from sglang.srt.layers.utils.logprob import compute_spec_v2_logprobs
from sglang.srt.managers.io_struct import (
    UpdateWeightFromDiskReqInput,
    UpdateWeightsFromIPCReqInput,
)
from sglang.srt.managers.schedule_batch import ScheduleBatch
from sglang.srt.managers.scheduler import GenerationBatchResult
from sglang.srt.managers.tp_worker import TpModelWorker
from sglang.srt.model_executor.forward_batch_info import (
    CaptureHiddenMode,
    ForwardBatch,
)
from sglang.srt.server_args import ServerArgs
from sglang.srt.speculative.base_spec_worker import BaseDraftWorker, BaseSpecWorker
from sglang.srt.speculative.draft_utils import DraftBackendFactory
from sglang.srt.speculative.eagle_info import EagleDraftInput, EagleVerifyInput
from sglang.srt.speculative.eagle_info_v2 import fill_bonus_tokens
from sglang.srt.speculative.eagle_utils import TreeMaskMode, build_tree_kernel_efficient
from sglang.srt.speculative.multi_layer_eagle_draft_extend_cuda_graph_runner import (
    MultiLayerEagleMultiStepDraftExtendCudaGraphRunner,
)
from sglang.srt.speculative.multi_layer_eagle_utils import (
    assign_hidden_states_pool_triton,
    rotate_input_ids_triton,
)
from sglang.srt.speculative.spec_info import SpeculativeAlgorithm
from sglang.srt.speculative.spec_utils import (
    draft_tp_context,
    maybe_detect_nan,
    maybe_detect_oob,
    record_stream_each,
    record_stream_for_v2_verify,
    select_top_k_tokens,
)
from sglang.srt.utils.common import empty_context, fast_topk

if TYPE_CHECKING:
    from sglang.srt.model_executor.model_runner import ModelRunner, ModelRunnerOutput
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on. It is one chunk of a larger setup region.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。 这是较大初始化区域中的一部分。

### Lines 61-65: Module header, imports, and shared constants (part 2/2)
```python


logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on. It is one chunk of a larger setup region.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。 这是较大初始化区域中的一部分。

### Lines 66-74: get plan stream function
```python
def _get_plan_stream(
    device: str,
) -> Tuple[any, contextlib.AbstractContextManager]:
    if envs.SGLANG_ENABLE_OVERLAP_PLAN_STREAM.get():
        plan_stream = torch.get_device_module(device).Stream()
        plan_stream_ctx = torch.get_device_module(device).stream(plan_stream)
        return plan_stream, plan_stream_ctx
    else:
        return None, contextlib.nullcontext()
```
**EN:** This block uses `_get_plan_stream` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_get_plan_stream` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 77-77: MultiLayerEagleDraftWorker class declaration
```python
class MultiLayerEagleDraftWorker(BaseDraftWorker):
```
**EN:** This block declares the `MultiLayerEagleDraftWorker` class, which exists to produce draft tokens or draft-side state. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `MultiLayerEagleDraftWorker` 类，其职责是生成草稿 token 或草稿侧状态。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 78-137: MultiLayerEagleDraftWorker initializer (part 1/2)
```python
    def __init__(
        self,
        server_args: ServerArgs,
        gpu_id: int,
        tp_rank: int,
        dp_rank: int,
        moe_ep_rank: int,
        attn_cp_rank: int,
        moe_dp_rank: int,
        nccl_port: int,
        target_worker: TpModelWorker,
    ):
        # copy args
        self.server_args = server_args
        self.gpu_id = gpu_id
        self.tp_rank = tp_rank
        self.dp_rank = dp_rank
        self.moe_ep_rank = moe_ep_rank
        self.nccl_port = nccl_port
        self.target_worker = target_worker
        self.draft_extend_attn_backend_list = []
        self.model_config = target_worker.model_config

        # Args for easy access
        self.device = server_args.device
        self.topk = server_args.speculative_eagle_topk
        self.speculative_num_steps = server_args.speculative_num_steps
        self.speculative_num_draft_tokens = server_args.speculative_num_draft_tokens
        self.speculative_algorithm = SpeculativeAlgorithm.from_string(
            server_args.speculative_algorithm
        )

        # Set constant
        EagleDraftInput.ALLOC_LEN_PER_DECODE = max(
            self.speculative_num_steps * self.topk, self.speculative_num_draft_tokens
        )

        # Do not capture cuda graph in `TpModelWorker` init,
        # will capture later with init_cuda_graphs()
        backup_disable_cuda_graph = server_args.disable_cuda_graph
        server_args.disable_cuda_graph = True

        # Share the allocator with a target worker.
        # Draft and target worker own their own KV cache pools.
        self.req_to_token_pool, self.token_to_kv_pool_allocator = (
            target_worker.get_memory_pool()
        )
        with empty_context(), speculative_moe_backend_context():
            # Init draft worker
            self.draft_worker = TpModelWorker(
                server_args=server_args,
                gpu_id=gpu_id,
                tp_rank=tp_rank,
                pp_rank=0,  # FIXME
                dp_rank=dp_rank,
                moe_ep_rank=moe_ep_rank,
                attn_cp_rank=attn_cp_rank,
                moe_dp_rank=moe_dp_rank,
                nccl_port=nccl_port,
                is_draft_worker=True,
```
**EN:** This block initializes the `MultiLayerEagleDraftWorker` object, setting up the state, buffers, and references that later methods rely on. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块初始化 `MultiLayerEagleDraftWorker` 对象，建立后续方法依赖的状态、缓冲区与引用。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 138-186: MultiLayerEagleDraftWorker initializer (part 2/2)
```python
                req_to_token_pool=self.req_to_token_pool,
                token_to_kv_pool_allocator=self.token_to_kv_pool_allocator,
                memory_pool_config=target_worker.model_runner.memory_pool_config,
                is_multi_layer_eagle=True,
            )

        # Alias for better readability
        self.draft_runner_list: List[ModelRunner] = self.draft_worker.model_runner_list
        # Match `EagleDraftWorker.draft_runner` so `_draft_runner_of(self)` works
        # for the EagleDraftInput shape classmethods.
        self.draft_runner: ModelRunner = self.draft_runner_list[0]

        # Chain-style MTP: each step propagates its own output hidden states to the
        # next step.  Non-chain: each step uses the target model's hidden states.
        draft_arch = self.draft_worker.model_config.hf_config.architectures[0]
        self.chain_mtp_hidden_states = draft_arch in ["Step3p5MTP"]

        self.init_lm_head()

        # KV cache reversion buffer; sized to mirror req_to_token (indexed by
        # req_pool_idx).
        self.req_to_hidden_states_pool = torch.empty(
            (
                self.req_to_token_pool.req_to_token.shape[0],
                self.speculative_num_steps - 1,
                self.model_config.hidden_size,
            ),
            dtype=self.model_config.dtype,
            device=self.device,
        )

        # Init attention backend and cuda graphs
        for i in range(self.speculative_num_steps):
            self.draft_runner_list[i].server_args.disable_cuda_graph = (
                backup_disable_cuda_graph
            )
        self.draft_tp_context = (
            draft_tp_context if server_args.enable_dp_attention else empty_context
        )
        with (
            self.draft_tp_context(self.draft_runner_list[0].tp_group),
            speculative_moe_backend_context(),
        ):
            self.init_attention_backend()
            self.init_cuda_graphs()

        self.tree_mask_mode = TreeMaskMode.FULL_MASK

        self.plan_stream, self.plan_stream_ctx = _get_plan_stream(self.device)
```
**EN:** This block initializes the `MultiLayerEagleDraftWorker` object, setting up the state, buffers, and references that later methods rely on. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块初始化 `MultiLayerEagleDraftWorker` 对象，建立后续方法依赖的状态、缓冲区与引用。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 188-189: MultiLayerEagleDraftWorker.mtp_model_runner method
```python
    def mtp_model_runner(self, step: int):
        return self.draft_runner_list[step]
```
**EN:** This block uses `MultiLayerEagleDraftWorker.mtp_model_runner` to coordinate staged runtime execution. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleDraftWorker.mtp_model_runner` 来协调分阶段运行时执行。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 191-195: MultiLayerEagleDraftWorker.init_lm_head method
```python
    def init_lm_head(self):
        embed, head = self.target_worker.model_runner.model.get_embed_and_head()
        # Share the embedding and lm_head
        for i in range(self.speculative_num_steps):
            self.draft_runner_list[i].model.set_embed_and_head(embed, head)
```
**EN:** This block uses `MultiLayerEagleDraftWorker.init_lm_head` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleDraftWorker.init_lm_head` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 197-212: MultiLayerEagleDraftWorker.init_attention_backend method
```python
    def init_attention_backend(self):
        # Create attn backends
        self.draft_extend_attn_backend_list = []
        for step in range(self.speculative_num_steps):
            draft_backend_factory = DraftBackendFactory(
                self.server_args,
                self.draft_runner_list[step],
                self.topk,
                self.speculative_num_steps,
            )
            self.draft_extend_attn_backend_list.append(
                draft_backend_factory.create_draft_extend_backend()
            )
            self.draft_runner_list[step].attn_backend = (
                self.draft_extend_attn_backend_list[-1]
            )
```
**EN:** This block uses `MultiLayerEagleDraftWorker.init_attention_backend` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleDraftWorker.init_attention_backend` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 214-224: MultiLayerEagleDraftWorker.init_cuda_graphs method
```python
    def init_cuda_graphs(self):
        """Capture cuda graphs."""
        self.cuda_graph_runner = None
        self.cuda_graph_runner_for_draft_extend = None

        if self.server_args.disable_cuda_graph:
            return

        self.cuda_graph_runner_for_draft_extend = (
            MultiLayerEagleMultiStepDraftExtendCudaGraphRunner(self)
        )
```
**EN:** This block uses `MultiLayerEagleDraftWorker.init_cuda_graphs` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleDraftWorker.init_cuda_graphs` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 226-230: MultiLayerEagleDraftWorker.reset_cuda_graph_buffers method
```python
    def reset_cuda_graph_buffers(self, forward_batch, batch_result):
        if self.cuda_graph_runner_for_draft_extend:
            self.cuda_graph_runner_for_draft_extend.reset_buffers(
                forward_batch, batch_result
            )
```
**EN:** This block uses `MultiLayerEagleDraftWorker.reset_cuda_graph_buffers` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleDraftWorker.reset_cuda_graph_buffers` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 232-291: MultiLayerEagleDraftWorker.draft method (part 1/2)
```python
    def draft(self, batch: ScheduleBatch):
        draft_input: EagleDraftInput = batch.spec_info
        forward_batch, can_cuda_graph = draft_input.prepare_for_v2_draft(
            self.req_to_token_pool,
            batch,
            self.cuda_graph_runner,
            self.draft_runner_list[0],
            self.topk,
            self.speculative_num_steps,
        )

        # Run draft
        parent_list, top_scores_index, draft_tokens = self.draft_forward(forward_batch)

        if batch.forward_mode.is_idle():
            return EagleVerifyInput.create_idle_input(
                self.topk,
                self.speculative_num_steps,
                self.speculative_num_draft_tokens,
            )

        # Build tree mask
        # Directly write to cuda graph buffers for verify attn
        tree_mask_buf, position_buf = (
            self.target_worker.model_runner.attn_backend.get_verify_buffers_to_fill_after_draft()
        )
        (
            tree_mask,
            position,
            retrieve_index,
            retrieve_next_token,
            retrieve_next_sibling,
            draft_tokens,
        ) = build_tree_kernel_efficient(
            draft_input.bonus_tokens,
            parent_list,
            top_scores_index,
            draft_tokens,
            batch.seq_lens,
            batch.seq_lens_sum,
            self.topk,
            self.speculative_num_steps,
            self.speculative_num_draft_tokens,
            self.tree_mask_mode,
            tree_mask_buf,
            position_buf,
        )

        return EagleVerifyInput(
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
            capture_hidden_mode=None,
```
**EN:** This block uses `MultiLayerEagleDraftWorker.draft` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `MultiLayerEagleDraftWorker.draft` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 292-294: MultiLayerEagleDraftWorker.draft method (part 2/2)
```python
            seq_lens_sum=None,
            seq_lens_cpu=None,
        )
```
**EN:** This block uses `MultiLayerEagleDraftWorker.draft` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `MultiLayerEagleDraftWorker.draft` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 296-355: MultiLayerEagleDraftWorker.draft_forward method (part 1/2)
```python
    def draft_forward(self, forward_batch: ForwardBatch):
        # Parse args
        spec_info: EagleDraftInput = forward_batch.spec_info
        topk_p, topk_index, hidden_states = (
            spec_info.topk_p,
            spec_info.topk_index,
            spec_info.hidden_states,
        )

        maybe_detect_nan(topk_p, "draft_forward: NaN in initial topk_p from spec_info")

        # Return values
        score_list: List[torch.Tensor] = []
        token_list: List[torch.Tensor] = []
        parents_list: List[torch.Tensor] = []

        # Forward multiple steps
        scores = None
        _, hidden_states, scores, tree_info = select_top_k_tokens(
            0, topk_p, topk_index, hidden_states, scores, self.topk
        )
        if self.speculative_num_steps == 1:
            score_list.append(tree_info[0])
            token_list.append(tree_info[1])
            parents_list.append(tree_info[2])
        else:
            for i in range(self.speculative_num_steps):
                score_list.append(tree_info[0][:, :, i].unsqueeze(-1))
                token_index = tree_info[1][:, i].unsqueeze(-1)
                token_list.append(token_index)
                if i == 0:
                    parents_list.append(tree_info[2])
                else:
                    parents_list.append(
                        torch.full(
                            (tree_info[2].size(0), 1),
                            i,
                            dtype=torch.long,
                            device="cuda",
                        )
                    )

        # Organize the results
        score_list = torch.cat(score_list, dim=1).flatten(
            1
        )  # b, n, topk; n= 1 + (num_steps-1) * self.topk
        ss_token_list = torch.cat(
            token_list, dim=1
        )  # b, (self.topk + (num_steps-1) * self.topk)
        top_scores = torch.topk(
            score_list, self.speculative_num_draft_tokens - 1, dim=-1
        )
        top_scores_index = top_scores.indices
        top_scores_index = torch.sort(top_scores_index).values
        maybe_detect_oob(
            top_scores_index,
            0,
            ss_token_list.shape[1],
            "draft_forward: top_scores_index OOB for gather on ss_token_list",
        )
```
**EN:** This block uses `MultiLayerEagleDraftWorker.draft_forward` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `MultiLayerEagleDraftWorker.draft_forward` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 356-364: MultiLayerEagleDraftWorker.draft_forward method (part 2/2)
```python
        draft_tokens = torch.gather(ss_token_list, index=top_scores_index, dim=1)

        if len(parents_list) > 1:
            parent_list = torch.cat(parents_list[:-1], dim=1)
        else:
            batch_size = parents_list[0].shape[0]
            parent_list = torch.empty(batch_size, 0, device=parents_list[0].device)

        return parent_list, top_scores_index, draft_tokens
```
**EN:** This block uses `MultiLayerEagleDraftWorker.draft_forward` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `MultiLayerEagleDraftWorker.draft_forward` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 366-367: MultiLayerEagleDraftWorker.draft_extend method
```python
    def draft_extend(self):
        pass
```
**EN:** This block uses `MultiLayerEagleDraftWorker.draft_extend` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleDraftWorker.draft_extend` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 369-428: MultiLayerEagleDraftWorker._draft_extend_for_prefill method (part 1/2)
```python
    def _draft_extend_for_prefill(
        self,
        batch: ScheduleBatch,
        target_hidden_states: torch.Tensor,
        next_token_ids: torch.Tensor,
    ):
        """
        Run draft model extend to correctly fill the KV cache.

        Args:
            batch: The batch to run.
            target_hidden_states: Hidden states from the target model forward
            next_token_ids: Next token ids generated from the target forward.
        """
        # Construct spec_info
        next_draft_input = EagleDraftInput(
            hidden_states=target_hidden_states,
            bonus_tokens=next_token_ids,
            new_seq_lens=batch.seq_lens,
            # draft mode is same with decode mode, only 1 token per req
            num_tokens_per_req=1,
            num_tokens_for_logprob_per_req=1,
        )

        batch.spec_info = next_draft_input

        # Chain-style MTP needs FULL to get all-token hidden states;
        # non-chain only needs LAST (the target model's hidden states).
        # STANDALONE skips hidden states end-to-end.
        if self.speculative_algorithm.is_standalone():
            draft_capture_hidden_mode = CaptureHiddenMode.NULL
        elif self.chain_mtp_hidden_states:
            draft_capture_hidden_mode = CaptureHiddenMode.FULL
        else:
            draft_capture_hidden_mode = CaptureHiddenMode.LAST

        # Run forward
        batch.capture_hidden_mode = draft_capture_hidden_mode
        batch.return_hidden_states_before_norm = True
        forward_batch = ForwardBatch.init_new(batch, self.draft_runner_list[0])

        # Construct input_ids
        if not batch.forward_mode.is_idle():
            rotate_input_ids_triton(
                forward_batch.input_ids,
                forward_batch.extend_start_loc,
                forward_batch.extend_seq_lens,
                next_token_ids,
            )

        topk_p_list = []
        topk_index_list = []
        for step in range(self.speculative_num_steps):
            forward_batch.req_to_token_pool = self.draft_runner_list[
                step
            ].req_to_token_pool
            output: ModelRunnerOutput = self.draft_runner_list[step].forward(
                forward_batch
            )
            maybe_detect_nan(
```
**EN:** This block uses `MultiLayerEagleDraftWorker._draft_extend_for_prefill` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `MultiLayerEagleDraftWorker._draft_extend_for_prefill` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 429-466: MultiLayerEagleDraftWorker._draft_extend_for_prefill method (part 2/2)
```python
                output.logits_output.next_token_logits,
                f"draft_extend_for_prefill step {step}",
            )
            probs = torch.softmax(output.logits_output.next_token_logits, dim=-1)
            topk_p, topk_index = fast_topk(probs, self.topk, dim=-1)
            topk_p_list.append(topk_p)
            topk_index_list.append(topk_index)
            # Chain-style: use this step's output hidden_states as next step's input
            if (
                self.chain_mtp_hidden_states
                and step < self.speculative_num_steps - 1
                and output.logits_output.hidden_states is not None
            ):
                forward_batch.spec_info.hidden_states = (
                    output.logits_output.hidden_states
                )
            if forward_batch.extend_seq_lens is not None:
                rotate_input_ids_triton(
                    forward_batch.input_ids,
                    forward_batch.extend_start_loc,
                    forward_batch.extend_seq_lens,
                    topk_index,
                )
        next_draft_input.topk_p = torch.cat(topk_p_list, dim=1)
        next_draft_input.topk_index = torch.cat(topk_index_list, dim=1)

        # Update req_to_hidden_states_pool for KV Cache reversion
        if forward_batch.extend_seq_lens is not None:
            assign_hidden_states_pool_triton(
                target_hidden_states,
                forward_batch.req_pool_indices,
                self.req_to_hidden_states_pool,
                self.speculative_num_steps - 1,
                forward_batch.batch_size,
                forward_batch.extend_seq_lens,
                forward_batch.extend_start_loc,
            )
        return next_draft_input
```
**EN:** This block uses `MultiLayerEagleDraftWorker._draft_extend_for_prefill` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `MultiLayerEagleDraftWorker._draft_extend_for_prefill` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 468-527: MultiLayerEagleDraftWorker._draft_extend_for_decode method (part 1/3)
```python
    def _draft_extend_for_decode(
        self, batch: ScheduleBatch, batch_result: GenerationBatchResult
    ):
        # Batch 2: Draft extend
        draft_input = EagleDraftInput(
            hidden_states=batch_result.logits_output.hidden_states,
            num_tokens_per_req=self.speculative_num_steps + 1,
            num_tokens_for_logprob_per_req=1,
        )

        # Prepare for draft extend in a separate stream
        # Notice that here we use batch_result.next_token_ids as the input ids
        with self.plan_stream_ctx:
            forward_batch = draft_input.prepare_for_extend_to_fill_draft_kvcache(
                batch,
                batch_result.next_token_ids,
                self.speculative_num_draft_tokens,
                self.draft_runner_list[0],
                self.cuda_graph_runner_for_draft_extend,
            )
            forward_batch.return_hidden_states_before_norm = True

        if self.plan_stream:
            torch.get_device_module(self.device).current_stream().wait_stream(
                self.plan_stream
            )
        # Run draft extend batch in the main compute stream
        can_cuda_graph = (
            self.cuda_graph_runner_for_draft_extend
            and self.cuda_graph_runner_for_draft_extend.can_run(forward_batch)
        )
        ret_topk_p_list = []
        ret_topk_index_list = []
        next_token_ids_backup = batch_result.next_token_ids.clone()

        if can_cuda_graph:
            self.reset_cuda_graph_buffers(forward_batch, batch_result)
        else:
            logger.warning_once(
                f"can't use cuda graph for draft extend! may have correctness issue!"
            )
            select_index = (
                torch.arange(len(batch.seq_lens), device=self.device)
                * self.speculative_num_draft_tokens
                + batch_result.accept_lens
                - 1
            )

        for step in range(self.speculative_num_steps):
            # log_info_on_rank0(logger, f"step: {step}, forward_batch.input_ids: {forward_batch.input_ids}")
            if can_cuda_graph:
                draft_logits_output = (
                    self.cuda_graph_runner_for_draft_extend.get_runner(step).replay(
                        forward_batch, init_state=(step == 0)
                    )
                )
                ret_topk_p, ret_topk_index = (
                    draft_logits_output.topk_p,
                    draft_logits_output.topk_index,
                )
```
**EN:** This block uses `MultiLayerEagleDraftWorker._draft_extend_for_decode` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `MultiLayerEagleDraftWorker._draft_extend_for_decode` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 528-587: MultiLayerEagleDraftWorker._draft_extend_for_decode method (part 2/3)
```python
            else:
                forward_batch.req_to_token_pool = self.draft_runner_list[
                    step
                ].req_to_token_pool
                draft_logits_output = self.draft_runner_list[step].forward(
                    forward_batch, skip_attn_backend_init=True
                )
                probs = torch.softmax(
                    draft_logits_output.logits_output.next_token_logits[select_index],
                    dim=-1,
                )
                ret_topk_p, ret_topk_index = fast_topk(probs, self.topk, dim=-1)
                # Chain-style: use this step's output hidden_states as next step's input
                if (
                    self.chain_mtp_hidden_states
                    and step < self.speculative_num_steps - 1
                    and draft_logits_output.logits_output.hidden_states is not None
                ):
                    forward_batch.spec_info.hidden_states = (
                        draft_logits_output.logits_output.hidden_states
                    )
                if forward_batch.extend_seq_lens is not None:
                    rotate_input_ids_triton(
                        forward_batch.input_ids,
                        forward_batch.extend_start_loc,
                        forward_batch.extend_seq_lens,
                        ret_topk_index,
                        select_index,
                    )
            ret_topk_p_list.append(ret_topk_p)
            ret_topk_index_list.append(ret_topk_index)

        # Update req_to_hidden_states_pool for KV Cache reversion
        if (
            forward_batch.extend_seq_lens is not None
            and self.cuda_graph_runner_for_draft_extend is not None
        ):
            if can_cuda_graph:
                last_runner = self.cuda_graph_runner_for_draft_extend.get_last_runner()
                hidden_states = last_runner.buffers.hidden_states
                req_pool_indices = last_runner.buffers.req_pool_indices
                extend_seq_lens = last_runner.buffers.extend_seq_lens
                extend_start_loc = last_runner.buffers.extend_start_loc
            else:
                hidden_states = draft_logits_output.logits_output.hidden_states
                req_pool_indices = forward_batch.req_pool_indices
                extend_seq_lens = forward_batch.extend_seq_lens
                extend_start_loc = forward_batch.extend_start_loc
            assign_hidden_states_pool_triton(
                hidden_states,
                req_pool_indices,
                self.req_to_hidden_states_pool,
                self.speculative_num_steps - 1,
                forward_batch.batch_size,
                extend_seq_lens,
                extend_start_loc,
            )

        # Reorganize the spec info for the next batch
        # draft_logits_output.next_token_logits = draft_logits_output.next_token_logits[
```
**EN:** This block uses `MultiLayerEagleDraftWorker._draft_extend_for_decode` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk continues the routine's middle-stage logic and data movement.
**CN:** 该代码块通过 `MultiLayerEagleDraftWorker._draft_extend_for_decode` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分继续展示例程中段的逻辑与数据流动。

### Lines 588-604: MultiLayerEagleDraftWorker._draft_extend_for_decode method (part 3/3)
```python
        #     select_index
        # ]
        # draft_logits_output.hidden_states = draft_logits_output.hidden_states[
        #     select_index
        # ]
        batch_result.next_token_ids = next_token_ids_backup
        # Construct the return values
        next_draft_input = batch_result.next_draft_input
        (
            next_draft_input.topk_p,
            next_draft_input.topk_index,
            next_draft_input.hidden_states,
        ) = (
            torch.cat(ret_topk_p_list, dim=1).clone(),
            torch.cat(ret_topk_index_list, dim=1).clone(),
            None,
        )
```
**EN:** This block uses `MultiLayerEagleDraftWorker._draft_extend_for_decode` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `MultiLayerEagleDraftWorker._draft_extend_for_decode` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 607-607: MultiLayerEagleWorkerV2 class declaration
```python
class MultiLayerEagleWorkerV2(BaseSpecWorker):
```
**EN:** This block declares the `MultiLayerEagleWorkerV2` class, which exists to coordinate staged runtime execution. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `MultiLayerEagleWorkerV2` 类，其职责是协调分阶段运行时执行。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 608-658: MultiLayerEagleWorkerV2 initializer
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
        # Parse arguments
        self.server_args = server_args
        self.topk = server_args.speculative_eagle_topk
        self.speculative_num_steps = server_args.speculative_num_steps
        self.speculative_num_draft_tokens = server_args.speculative_num_draft_tokens
        self.gpu_id = gpu_id
        self.device = server_args.device
        self._target_worker = target_worker
        self.page_size = server_args.page_size
        self.speculative_algorithm = SpeculativeAlgorithm.from_string(
            server_args.speculative_algorithm
        )

        self.req_to_token_pool, self.token_to_kv_pool_allocator = (
            target_worker.get_memory_pool()
        )

        # Override the context length of the draft model to be the same as the target model.
        server_args.context_length = target_worker.model_runner.model_config.context_len

        self._draft_worker = MultiLayerEagleDraftWorker(
            server_args,
            gpu_id,
            tp_rank,
            dp_rank,
            moe_ep_rank,
            attn_cp_rank,
            moe_dp_rank,
            nccl_port,
            target_worker,
        )

        # Some dummy tensors
        self.num_new_pages_per_topk = torch.empty(
            (), dtype=torch.int64, device=self.device
        )
        self.extend_lens = torch.empty((), dtype=torch.int64, device=self.device)

        self.plan_stream, self.plan_stream_ctx = _get_plan_stream(self.device)
```
**EN:** This block initializes the `MultiLayerEagleWorkerV2` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `MultiLayerEagleWorkerV2` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 660-662: MultiLayerEagleWorkerV2.target_worker method
```python
    @property
    def target_worker(self):
        return self._target_worker
```
**EN:** This block uses `MultiLayerEagleWorkerV2.target_worker` to coordinate staged runtime execution. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleWorkerV2.target_worker` 来协调分阶段运行时执行。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 664-666: MultiLayerEagleWorkerV2.draft_worker method
```python
    @property
    def draft_worker(self):
        return self._draft_worker
```
**EN:** This block uses `MultiLayerEagleWorkerV2.draft_worker` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleWorkerV2.draft_worker` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 668-670: MultiLayerEagleWorkerV2.clear_cache_pool method
```python
    def clear_cache_pool(self):
        # allocator and kv cache pool are shared with target worker, which are cleared in scheduler
        pass
```
**EN:** This block uses `MultiLayerEagleWorkerV2.clear_cache_pool` to clear cached state or buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleWorkerV2.clear_cache_pool` 来清理缓存状态或缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 672-714: MultiLayerEagleWorkerV2.forward_batch_generation method
```python
    def forward_batch_generation(self, batch: ScheduleBatch):
        if batch.forward_mode.is_extend() or batch.is_extend_in_batch:
            # Target prefill
            target_capture_mode = (
                CaptureHiddenMode.NULL
                if self.speculative_algorithm.is_standalone()
                else CaptureHiddenMode.FULL
            )
            batch.capture_hidden_mode = target_capture_mode
            batch_output = self.target_worker.forward_batch_generation(batch)

            # Chain-style MTP needs FULL to get all-token hidden states;
            # non-chain only needs LAST (the target model's hidden states).
            batch_output.next_draft_input = self.draft_worker._draft_extend_for_prefill(
                batch,
                batch_output.logits_output.hidden_states,
                batch_output.next_token_ids,
            )
            return batch_output
        else:
            if batch.spec_info is None:
                capture_mode = (
                    CaptureHiddenMode.NULL
                    if self.speculative_algorithm.is_standalone()
                    else CaptureHiddenMode.LAST
                )
                batch.spec_info = EagleDraftInput.create_idle_input(
                    device=self.device,
                    hidden_size=EagleDraftInput.hidden_size_for(self.draft_worker),
                    dtype=EagleDraftInput.dtype_for(self.draft_worker),
                    topk=self.topk * self.speculative_num_steps,
                    capture_hidden_mode=capture_mode,
                )
            verify_input: EagleVerifyInput = self.draft_worker.draft(batch)
            assert verify_input.is_verify_input()
            # Record a CUDA event after draft() GPU work is dispatched.
            if self.plan_stream:
                self._draft_done_event = torch.get_device_module(self.device).Event()
                self._draft_done_event.record()
            batch.spec_info = verify_input
            batch_output = self.verify(batch)
            self.draft_worker._draft_extend_for_decode(batch, batch_output)
            return batch_output
```
**EN:** This block uses `MultiLayerEagleWorkerV2.forward_batch_generation` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleWorkerV2.forward_batch_generation` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 716-775: MultiLayerEagleWorkerV2.verify method (part 1/2)
```python
    def verify(
        self,
        batch: ScheduleBatch,
    ):
        fwd_stream = torch.get_device_module(self.device).current_stream()
        verify_input: EagleVerifyInput = batch.spec_info
        record_stream_for_v2_verify(batch, verify_input, fwd_stream)

        bs = len(batch.seq_lens)

        # Batch 1: Target verify
        # Prepare for target verify in a separate stream
        with self.plan_stream_ctx:
            # Wait for the draft CUDA graph to finish before plan_stream
            # begins its work.
            if self.plan_stream and hasattr(self, "_draft_done_event"):
                self.plan_stream.wait_event(self._draft_done_event)
            verify_forward_batch, can_run_cuda_graph = (
                verify_input.prepare_for_v2_verify(
                    self.req_to_token_pool,
                    batch,
                    self.target_worker,
                )
            )

        # Cover post-prepare rebinds: draft_token, plan_stream-allocated out_cache_loc.
        record_stream_each((batch.input_ids, batch.out_cache_loc), fwd_stream)

        # Correct some buffers due to the overlap plan
        if self.plan_stream:
            torch.get_device_module(self.device).current_stream().wait_stream(
                self.plan_stream
            )

            # Some values such as custom_mask and position depend on the output of draft,
            # so the previous plan step used the wrong values. Here, we need to run the related
            # computation again to update them to the correct values.
            self.target_worker.model_runner.attn_backend.update_verify_buffers_to_fill_after_draft(
                verify_input,
                (
                    self.target_worker.model_runner.graph_runner.bs
                    if can_run_cuda_graph
                    else None
                ),
            )
        # Run target verify batch in the main compute stream
        forward_batch_output = self.target_worker.forward_batch_generation(
            batch=None,
            forward_batch=verify_forward_batch,
            is_verify=True,
            skip_attn_backend_init=True,
        )
        logits_output = forward_batch_output.logits_output

        # Sample
        maybe_detect_nan(logits_output.next_token_logits, "verify: target model logits")
        (
            predict,
            accept_lens,
            accept_index,
```
**EN:** This block uses `MultiLayerEagleWorkerV2.verify` to verify draft results against the target model. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `MultiLayerEagleWorkerV2.verify` 来根据目标模型校验草稿结果。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 776-816: MultiLayerEagleWorkerV2.verify method (part 2/2)
```python
        ) = verify_input.sample(batch, logits_output)
        new_seq_lens = batch.seq_lens + accept_lens
        verify_done = torch.get_device_module(self.device).Event()
        verify_done.record()

        if not batch.forward_mode.is_idle():
            accept_tokens = predict[accept_index]
            bonus_tokens = torch.empty_like(accept_lens, dtype=torch.int32)
            fill_bonus_tokens[(bs,)](
                accept_tokens,
                accept_lens,
                bonus_tokens,
                self.speculative_num_draft_tokens,
            )
        else:
            bonus_tokens = torch.empty((0,), device=self.device, dtype=torch.int32)

        if batch.return_logprob and not batch.forward_mode.is_idle():
            compute_spec_v2_logprobs(
                batch, logits_output, predict, accept_index, self.speculative_num_steps
            )

        next_draft_input = EagleDraftInput(
            bonus_tokens=bonus_tokens,
            new_seq_lens=new_seq_lens,
            verify_done=verify_done,
        )
        # verify_forward_batch transitively holds verify-time GPU tensors that
        # must outlive the imminent batch.input_ids rebind; scheduler pins it
        # in batch_record_buf via extra_keep_alive_refs. See EAGLEWorkerV2.verify.
        return GenerationBatchResult(
            logits_output=logits_output,
            next_token_ids=predict,
            can_run_cuda_graph=can_run_cuda_graph,
            speculative_num_draft_tokens=self.speculative_num_draft_tokens,
            next_draft_input=next_draft_input,
            accept_lens=accept_lens,
            routed_experts_output=forward_batch_output.routed_experts_output,
            indexer_topk_output=forward_batch_output.indexer_topk_output,
            extra_keep_alive_refs=[verify_forward_batch],
        )
```
**EN:** This block uses `MultiLayerEagleWorkerV2.verify` to verify draft results against the target model. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `MultiLayerEagleWorkerV2.verify` 来根据目标模型校验草稿结果。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 818-829: MultiLayerEagleWorkerV2.update_weights_from_disk method
```python
    def update_weights_from_disk(self, recv_req: UpdateWeightFromDiskReqInput):
        for i in range(self.speculative_num_steps):
            success, message = self._draft_worker.draft_runner_list[
                i
            ].update_weights_from_disk(
                recv_req.model_path,
                recv_req.load_format,
                recapture_cuda_graph=recv_req.recapture_cuda_graph,
            )
            if not success:
                return success, message
        return True, "Succeeded to update model weights."
```
**EN:** This block uses `MultiLayerEagleWorkerV2.update_weights_from_disk` to refresh state using new information. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleWorkerV2.update_weights_from_disk` 来利用新信息刷新状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 831-838: MultiLayerEagleWorkerV2.update_weights_from_ipc method
```python
    def update_weights_from_ipc(self, recv_req: UpdateWeightsFromIPCReqInput):
        for i in range(self.speculative_num_steps):
            success, message = self._draft_worker.draft_runner_list[
                i
            ].update_weights_from_ipc(recv_req)
            if not success:
                return success, message
        return True, "Succeeded to update model weights."
```
**EN:** This block uses `MultiLayerEagleWorkerV2.update_weights_from_ipc` to refresh state using new information. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MultiLayerEagleWorkerV2.update_weights_from_ipc` 来利用新信息刷新状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- Worker coordination / Worker 协调
- Speculative decoding / 推测解码
- EAGLE draft pipeline / EAGLE 草稿流水线
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算
- Graph-captured execution paths / 图捕获执行路径
- Integration with model workers / 与模型 worker 集成

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.environ`
- `sglang.srt.layers.moe.utils`
- `sglang.srt.layers.utils.logprob`
- `sglang.srt.managers.io_struct`
- `sglang.srt.managers.schedule_batch`
- `sglang.srt.managers.scheduler`
- `sglang.srt.managers.tp_worker`
- `sglang.srt.model_executor.forward_batch_info`
- `sglang.srt.model_executor.model_runner`
- `sglang.srt.server_args`
- `sglang.srt.speculative.base_spec_worker`
- `sglang.srt.speculative.draft_utils`
- `sglang.srt.speculative.eagle_info`
- `sglang.srt.speculative.eagle_info_v2`
- `sglang.srt.speculative.eagle_utils`
- `sglang.srt.speculative.multi_layer_eagle_draft_extend_cuda_graph_runner`
- `sglang.srt.speculative.multi_layer_eagle_utils`
- `sglang.srt.speculative.spec_info`
- `sglang.srt.speculative.spec_utils`
- `sglang.srt.utils.common`
### External / 外部
- `torch`
- `contextlib` (stdlib)
- `logging` (stdlib)
- `typing` (stdlib)
