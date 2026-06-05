# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements utils logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 工具 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-6: Import runtime dependencies / 导入运行时依赖
```python
import dataclasses
import logging
from dataclasses import dataclass
from typing import TYPE_CHECKING, Any, List, Optional, Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 8-8: Import runtime dependencies / 导入运行时依赖
```python
import torch
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 10-17: Import runtime dependencies / 导入运行时依赖
```python
from sglang.srt.constants import HEALTH_CHECK_RID_PREFIX
from sglang.srt.eplb.expert_distribution import ExpertDistributionMetrics
from sglang.srt.layers.logits_processor import LogitsProcessorOutput
from sglang.srt.managers.overlap_utils import FutureIndices
from sglang.srt.managers.schedule_batch import Req
from sglang.srt.model_executor.forward_batch_info import PPProxyTensors
from sglang.srt.server_args import ServerArgs
from sglang.srt.state_capturer.base import TopkCaptureOutput
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 19-21: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.managers.scheduler import GenerationBatchResult
    from sglang.srt.speculative.eagle_info import EagleDraftInput
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 24-27: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)


@dataclasses.dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 28-133: Define class GenerationBatchResult / 定义类 GenerationBatchResult
```python
class GenerationBatchResult:
    logits_output: Optional[LogitsProcessorOutput] = None
    pp_hidden_states_proxy_tensors: Optional[PPProxyTensors] = None
    next_token_ids: Optional[Union[torch.Tensor, List[torch.Tensor]]] = None
    num_correct_drafts: int = 0  # no bonus included
    num_correct_drafts_per_req_cpu: Optional[List[int]] = None
    can_run_cuda_graph: bool = False

    # For output processing
    extend_input_len_per_req: Optional[List[int]] = None
    extend_logprob_start_len_per_req: Optional[List[int]] = None

    # For overlap scheduling
    copy_done: Optional[torch.cuda.Event] = None
    delay_sample_func: Optional[callable] = None
    future_indices: Optional[FutureIndices] = None
    speculative_num_draft_tokens: Optional[int] = None

    # FIXME(lsyin): maybe move to a better place?
    # sync path: forward stream -> output processor
    accept_lens: Optional[torch.Tensor] = None

    # relay path: forward stream -> next step forward
    next_draft_input: Optional[EagleDraftInput] = None

    # Refs the worker wants scheduler to keep alive for the same 2-iter window
    # as batch_record_buf. Used for cross-stream tensor lifetime (e.g. a spec
    # V2 verify ForwardBatch whose tensors must outlive mid-iter SB rebinds).
    extra_keep_alive_refs: Optional[List[Any]] = None

    # Routed experts: pending async D2H for overlap scheduling
    routed_experts_output: Optional[TopkCaptureOutput] = None
    indexer_topk_output: Optional[TopkCaptureOutput] = None

    # metrics
    expert_distribution_metrics: Optional[ExpertDistributionMetrics] = None

    # Forward pass metrics (FPM) — GPU-accurate timing via CUDA events
    fpm_start_event: Optional[torch.cuda.Event] = None
    fpm_end_event: Optional[torch.cuda.Event] = None

    def copy_to_cpu(self, return_logprob: bool, return_hidden_states: bool = True):
        """Copy tensors to CPU in overlap scheduling.
        Only the tensors which are needed for processing results are copied,
        e.g., next_token_ids, logits outputs
        """
        if return_logprob:
            if self.logits_output.next_token_logprobs is not None:
                self.logits_output.next_token_logprobs = (
                    self.logits_output.next_token_logprobs.to("cpu", non_blocking=True)
                )
            if self.logits_output.input_token_logprobs is not None:
                self.logits_output.input_token_logprobs = (
                    self.logits_output.input_token_logprobs.to("cpu", non_blocking=True)
                )
            if self.logits_output.next_token_top_logprobs_val is not None:
                self.logits_output.next_token_top_logprobs_val = [
                    v.to("cpu", non_blocking=True) if torch.is_tensor(v) else v
                    for v in self.logits_output.next_token_top_logprobs_val
                ]
            if self.logits_output.next_token_top_logprobs_idx is not None:
                self.logits_output.next_token_top_logprobs_idx = [
                    x.to("cpu", non_blocking=True) if torch.is_tensor(x) else x
                    for x in self.logits_output.next_token_top_logprobs_idx
                ]
            if self.logits_output.next_token_token_ids_logprobs_val is not None:
                self.logits_output.next_token_token_ids_logprobs_val = [
                    v.to("cpu", non_blocking=True) if torch.is_tensor(v) else v
                    for v in self.logits_output.next_token_token_ids_logprobs_val
                ]
        if return_hidden_states and self.logits_output.hidden_states is not None:
            self.logits_output.hidden_states = self.logits_output.hidden_states.to(
                "cpu", non_blocking=True
            )
        self.next_token_ids = self.next_token_ids.to("cpu", non_blocking=True)

        if self.accept_lens is not None:
            self.accept_lens = self.accept_lens.to("cpu", non_blocking=True)

        if self.routed_experts_output is not None:
            self.routed_experts_output.copy_to_cpu()

        if self.indexer_topk_output is not None:
            self.indexer_topk_output.copy_to_cpu()

        if (x := self.expert_distribution_metrics) is not None:
            x.copy_to_cpu()

        self.copy_done.record()

    @classmethod
    def from_pp_proxy(
        cls, logits_output, next_pp_outputs: PPProxyTensors, can_run_cuda_graph
    ):
        # TODO(lsyin): refactor PP and avoid using dict
        proxy_dict = next_pp_outputs.tensors
        return cls(
            logits_output=logits_output,
            pp_hidden_states_proxy_tensors=None,
            next_token_ids=next_pp_outputs["next_token_ids"],
            extend_input_len_per_req=proxy_dict.get("extend_input_len_per_req", None),
            extend_logprob_start_len_per_req=proxy_dict.get(
                "extend_logprob_start_len_per_req", None
            ),
            can_run_cuda_graph=can_run_cuda_graph,
        )
```
**EN:** This block declares the class `GenerationBatchResult`. It centers on coordinating utils behavior, with methods such as copy_to_cpu, from_pp_proxy.
**CN:** 该代码块声明类 `GenerationBatchResult`。它负责承载与 工具 相关的核心状态与行为，并通过 copy_to_cpu, from_pp_proxy 等方法组织实现。

### Lines 136-166: Implement validate input length / 实现validate input length
```python
def validate_input_length(
    req: Req, max_req_input_len: int, allow_auto_truncate: bool
) -> Optional[str]:
    """Validate and potentially truncate input length.

    Args:
        req: The request containing input_ids to validate
        max_req_input_len: Maximum allowed input length
        allow_auto_truncate: Whether to truncate long inputs

    Returns:
        Error message if validation fails, None if successful
    """
    if len(req.origin_input_ids) >= max_req_input_len:
        if allow_auto_truncate:
            logger.warning(
                "Request length is longer than the KV cache pool size or "
                "the max context length. Truncated. "
                f"{len(req.origin_input_ids)=}, {max_req_input_len=}."
            )
            req.origin_input_ids = req.origin_input_ids[:max_req_input_len]
            return None
        else:
            error_msg = (
                f"Input length ({len(req.origin_input_ids)} tokens) exceeds "
                f"the maximum allowed length ({max_req_input_len} tokens). "
                f"Use a shorter input or enable --allow-auto-truncate."
            )
            return error_msg

    return None
```
**EN:** This block implements the function `validate_input_length(req, max_req_input_len, allow_auto_truncate)`. It focuses on Validate and potentially truncate input length., providing reusable behavior for the utils pipeline.
**CN:** 该代码块实现函数 `validate_input_length(req, max_req_input_len, allow_auto_truncate)`。它围绕 `validate_input_length` 所承担的 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 169-187: Implement get logprob dict from result / 实现get logprob dict from result
```python
def get_logprob_dict_from_result(result: GenerationBatchResult) -> dict:

    logits_output = result.logits_output
    assert logits_output is not None

    return {
        "extend_input_len_per_req": result.extend_input_len_per_req,
        "extend_logprob_start_len_per_req": result.extend_logprob_start_len_per_req,
        "next_token_logprobs": result.logits_output.next_token_logprobs,
        "next_token_top_logprobs_val": result.logits_output.next_token_top_logprobs_val,
        "next_token_top_logprobs_idx": result.logits_output.next_token_top_logprobs_idx,
        "next_token_token_ids_logprobs_val": result.logits_output.next_token_token_ids_logprobs_val,
        "next_token_token_ids_logprobs_idx": result.logits_output.next_token_token_ids_logprobs_idx,
        "input_token_logprobs": result.logits_output.input_token_logprobs,
        "input_top_logprobs_val": result.logits_output.input_top_logprobs_val,
        "input_top_logprobs_idx": result.logits_output.input_top_logprobs_idx,
        "input_token_ids_logprobs_val": result.logits_output.input_token_ids_logprobs_val,
        "input_token_ids_logprobs_idx": result.logits_output.input_token_ids_logprobs_idx,
    }
```
**EN:** This block implements the function `get_logprob_dict_from_result(result)`. It focuses on handling the utils responsibilities represented by `get_logprob_dict_from_result`, providing reusable behavior for the utils pipeline.
**CN:** 该代码块实现函数 `get_logprob_dict_from_result(result)`。它围绕 `get_logprob_dict_from_result` 所承担的 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 190-217: Implement get logprob from pp outputs / 实现get logprob from pp outputs
```python
def get_logprob_from_pp_outputs(
    next_pp_outputs: PPProxyTensors,
) -> tuple[LogitsProcessorOutput, list[int], list[int]]:
    logits_output = LogitsProcessorOutput(
        # Do not send logits and hidden states because they are large
        next_token_logits=None,
        hidden_states=None,
        next_token_logprobs=next_pp_outputs["next_token_logprobs"],
        next_token_top_logprobs_val=next_pp_outputs["next_token_top_logprobs_val"],
        next_token_top_logprobs_idx=next_pp_outputs["next_token_top_logprobs_idx"],
        next_token_token_ids_logprobs_val=next_pp_outputs[
            "next_token_token_ids_logprobs_val"
        ],
        next_token_token_ids_logprobs_idx=next_pp_outputs[
            "next_token_token_ids_logprobs_idx"
        ],
        input_token_logprobs=next_pp_outputs["input_token_logprobs"],
        input_top_logprobs_val=next_pp_outputs["input_top_logprobs_val"],
        input_top_logprobs_idx=next_pp_outputs["input_top_logprobs_idx"],
        input_token_ids_logprobs_val=next_pp_outputs["input_token_ids_logprobs_val"],
        input_token_ids_logprobs_idx=next_pp_outputs["input_token_ids_logprobs_idx"],
    )
    extend_input_len_per_req = next_pp_outputs["extend_input_len_per_req"]
    extend_logprob_start_len_per_req = next_pp_outputs[
        "extend_logprob_start_len_per_req"
    ]

    return logits_output, extend_input_len_per_req, extend_logprob_start_len_per_req
```
**EN:** This block implements the function `get_logprob_from_pp_outputs(next_pp_outputs)`. It focuses on handling the utils responsibilities represented by `get_logprob_from_pp_outputs`, providing reusable behavior for the utils pipeline.
**CN:** 该代码块实现函数 `get_logprob_from_pp_outputs(next_pp_outputs)`。它围绕 `get_logprob_from_pp_outputs` 所承担的 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 220-244: Implement get alloc len per decode / 实现get alloc len per 解码
```python
def get_alloc_len_per_decode(server_args: Optional[ServerArgs] = None) -> int:
    if server_args is None:
        from sglang.srt.server_args import get_global_server_args

        server_args = get_global_server_args()

    if server_args.speculative_algorithm is None:
        return 1

    # Spec v1:
    # 1) alloc topk * num_steps when draft decoding and then restore the allocation
    # 2) alloc num_draft_tokens when verifying the drafts
    # Sepc v2: allocate max(topk * num_steps, num_draft_tokens)

    spec_steps = server_args.speculative_num_steps or 1
    spec_topk = server_args.speculative_eagle_topk or 1
    spec_tokens = server_args.speculative_num_draft_tokens
    page_size = server_args.page_size

    if page_size == 1 or spec_topk == 1:
        return max(spec_steps * spec_topk, spec_tokens)
    else:
        raise NotImplementedError(
            "get_alloc_len_per_decode not implemented for page_size > 1 and spec_topk > 1"
        )
```
**EN:** This block implements the function `get_alloc_len_per_decode(server_args)`. It focuses on handling the utils responsibilities represented by `get_alloc_len_per_decode`, providing reusable behavior for the utils pipeline.
**CN:** 该代码块实现函数 `get_alloc_len_per_decode(server_args)`。它围绕 `get_alloc_len_per_decode` 所承担的 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 245-247: Provide supporting module logic / 提供辅助模块逻辑
```python


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 248-288: Define class EmbeddingBatchResult / 定义类 EmbeddingBatchResult
```python
class EmbeddingBatchResult:
    """Result from an embedding/classification forward pass.

    Attributes:
        embeddings: Model output — pooled embeddings or classification logits.
        pooled_hidden_states: Raw hidden states before the task head.  Present
            only when the batch contained ``return_pooled_hidden_states=True``
            requests.  Tensor (uniform shapes) or list of tensors (MIS).
        copy_done: CUDA event recorded after the async CPU copy completes.
    """

    embeddings: torch.Tensor
    pooled_hidden_states: Optional[torch.Tensor] = None
    copy_done: Optional[torch.cuda.Event] = None

    def copy_to_cpu(self):
        """Copy embeddings and pooled hidden states to CPU for overlap scheduling."""
        if isinstance(self.embeddings, torch.Tensor):
            self.copy_done = torch.get_device_module(self.embeddings.device).Event()
            self.embeddings = self.embeddings.to("cpu", non_blocking=True)
        else:
            assert isinstance(self.embeddings, list)
            if len(self.embeddings) == 0:
                return

            self.copy_done = torch.get_device_module(self.embeddings[0].device).Event()
            self.embeddings = [
                emb.to("cpu", non_blocking=True) for emb in self.embeddings
            ]

        if self.pooled_hidden_states is not None:
            if isinstance(self.pooled_hidden_states, list):
                self.pooled_hidden_states = [
                    t.to("cpu", non_blocking=True) for t in self.pooled_hidden_states
                ]
            else:
                self.pooled_hidden_states = self.pooled_hidden_states.to(
                    "cpu", non_blocking=True
                )

        self.copy_done.record()
```
**EN:** This block declares the class `EmbeddingBatchResult`. It centers on Result from an embedding/classification forward pass., with methods such as copy_to_cpu.
**CN:** 该代码块声明类 `EmbeddingBatchResult`。它负责承载与 工具 相关的核心状态与行为，并通过 copy_to_cpu 等方法组织实现。

### Lines 291-293: Implement is health check generate req / 实现is 健康 检查 generate req
```python
def is_health_check_generate_req(recv_req):
    rid = getattr(recv_req, "rid", None)
    return rid is not None and rid.startswith(HEALTH_CHECK_RID_PREFIX)
```
**EN:** This block implements the function `is_health_check_generate_req(recv_req)`. It focuses on handling the utils responsibilities represented by `is_health_check_generate_req`, providing reusable behavior for the utils pipeline.
**CN:** 该代码块实现函数 `is_health_check_generate_req(recv_req)`。它围绕 `is_health_check_generate_req` 所承担的 工具 相关职责展开，为对应处理链路提供可复用能力。

## Key Concepts / 关键概念
- **Core types / 核心类型**: GenerationBatchResult, EmbeddingBatchResult
- **Main callables / 主要可调用对象**: validate_input_length, get_logprob_dict_from_result, get_logprob_from_pp_outputs, get_alloc_len_per_decode, is_health_check_generate_req
- **Domain focus / 领域焦点**: utils / 工具
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: dataclasses, logging, typing
- **Third-party / 第三方库**: __future__, torch
- **Local Modules / 本地模块**: sglang.srt.constants, sglang.srt.eplb.expert_distribution, sglang.srt.layers.logits_processor, sglang.srt.managers.overlap_utils, sglang.srt.managers.schedule_batch, sglang.srt.managers.scheduler, sglang.srt.model_executor.forward_batch_info, sglang.srt.server_args, sglang.srt.speculative.eagle_info, sglang.srt.state_capturer.base
