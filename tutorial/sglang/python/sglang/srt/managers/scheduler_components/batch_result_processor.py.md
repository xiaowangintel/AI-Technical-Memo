# batch_result_processor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/scheduler_components/batch_result_processor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements batch result processor logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 批处理 result 处理器 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-11: Provide supporting module logic / 提供辅助模块逻辑
```python
import logging
from dataclasses import dataclass
from typing import (
    TYPE_CHECKING,
    Callable,
    List,
    Optional,
    Union,
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 13-13: Import runtime dependencies / 导入运行时依赖
```python
import torch
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 15-29: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.disaggregation.utils import DisaggregationMode
from sglang.srt.environ import envs
from sglang.srt.layers.logits_processor import LogitsProcessorOutput
from sglang.srt.managers.io_struct import AbortReq
from sglang.srt.managers.schedule_batch import (
    Req,
    ScheduleBatch,
)
from sglang.srt.mem_cache.common import (
    maybe_cache_unfinished_req,
    release_kv_cache,
)
from sglang.srt.server_args import get_global_server_args
from sglang.srt.state_capturer.indexer_topk import get_global_indexer_capturer
from sglang.srt.state_capturer.routed_experts import get_global_experts_capturer
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 31-55: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.configs.model_config import ModelConfig
    from sglang.srt.disaggregation.decode_kvcache_offload_manager import (
        DecodeKVCacheOffloadManager,
    )
    from sglang.srt.managers.hisparse_coordinator import HiSparseCoordinator
    from sglang.srt.managers.scheduler_components.logprob_result_processor import (
        SchedulerLogprobResultProcessor,
    )
    from sglang.srt.managers.scheduler_components.metrics_reporter import (
        SchedulerMetricsReporter,
    )
    from sglang.srt.managers.scheduler_components.output_streamer import (
        SchedulerOutputStreamer,
    )
    from sglang.srt.managers.tp_worker import BaseTpWorker
    from sglang.srt.managers.utils import (
        EmbeddingBatchResult,
        GenerationBatchResult,
    )
    from sglang.srt.mem_cache.allocator import BaseTokenToKVPoolAllocator
    from sglang.srt.mem_cache.base_prefix_cache import BasePrefixCache
    from sglang.srt.mem_cache.memory_pool import ReqToTokenPool
    from sglang.srt.observability.metrics_collector import SchedulerMetricsCollector
    from sglang.srt.server_args import ServerArgs
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 57-79: Provide supporting module logic / 提供辅助模块逻辑
```python
logger = logging.getLogger(__name__)


@dataclass(kw_only=True, slots=True, frozen=True)
class SchedulerBatchResultProcessor:
    is_generation: bool
    disaggregation_mode: "DisaggregationMode"
    enable_overlap: bool
    enable_overlap_mlx: bool
    server_args: "ServerArgs"
    model_config: "ModelConfig"
    token_to_kv_pool_allocator: "BaseTokenToKVPoolAllocator"
    tree_cache: "BasePrefixCache"
    hisparse_coordinator: Optional["HiSparseCoordinator"]
    req_to_token_pool: "ReqToTokenPool"
    decode_offload_manager: Optional["DecodeKVCacheOffloadManager"]
    metrics_collector: "SchedulerMetricsCollector"
    metrics_reporter: "SchedulerMetricsReporter"
    draft_worker: "BaseTpWorker"
    model_worker: "BaseTpWorker"
    logprob_result_processor: "SchedulerLogprobResultProcessor"
    output_streamer: "SchedulerOutputStreamer"
    abort_request: Callable
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 81-98: Implement process batch result prebuilt / 实现process 批处理 result prebuilt
```python
    def process_batch_result_prebuilt(self, batch: ScheduleBatch):
        assert self.disaggregation_mode == DisaggregationMode.DECODE
        use_free_group = self.server_args.disaggregation_decode_enable_radix_cache
        if use_free_group:
            self.token_to_kv_pool_allocator.free_group_begin()
        for req in batch.reqs:
            req.time_stats.set_decode_prebuilt_finish_time()
            req.check_finished()
            if req.finished():
                req.time_stats.set_quick_finish_time()
                if self.server_args.enable_hisparse:
                    self.hisparse_coordinator.request_finished(req)
                release_kv_cache(req, self.tree_cache)

        # Note: Logprobs should be handled on the prefill engine.
        self.output_streamer.stream_output(batch.reqs, batch.return_logprob)
        if use_free_group:
            self.token_to_kv_pool_allocator.free_group_end()
```
**EN:** This block implements the method `process_batch_result_prebuilt(batch)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `process_batch_result_prebuilt`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `process_batch_result_prebuilt(batch)`。它围绕 `process_batch_result_prebuilt` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 100-141: Implement maybe collect routed experts / 实现maybe collect routed experts
```python
    def _maybe_collect_routed_experts(self, req: Req):
        """Collect routed experts for a finished request.

        Returns immediately if `return_routed_experts` was not set on the
        request, so non-opted-in reqs don't pay the host-gather cost.

        Honors the caller's absolute start so the response covers
        `[start_len, seqlen - 1)`. The default start_len is 0, which returns
        the full sequence.

        Logs a soft warning if the resulting tensor's row count differs from
        the expected `seqlen - 1 - start_len`, to catch silent regressions.
        """
        if not req.return_routed_experts:
            return
        capturer = get_global_experts_capturer()
        if capturer is None:
            return
        start_len = req.routed_experts_start_len
        req.routed_experts = capturer.get_topk(
            req_pool_idx=req.req_pool_idx,
            seqlen=req.seqlen,
            req_to_token_pool=self.req_to_token_pool,
            start_len=start_len,
        )

        expected_rows = max(0, req.seqlen - 1 - start_len)
        if (
            req.routed_experts is not None
            and req.routed_experts.shape[0] != expected_rows
        ):
            logger.warning(
                "routed_experts row-count mismatch for req %s: got %d, "
                "expected %d (seqlen=%d, cached_tokens=%d, start_len=%s). "
                "This indicates a silent bug.",
                req.rid,
                req.routed_experts.shape[0],
                expected_rows,
                req.seqlen,
                req.cached_tokens,
                req.routed_experts_start_len,
            )
```
**EN:** This block implements the method `_maybe_collect_routed_experts(req)` on `SchedulerBatchResultProcessor`. It focuses on Collect routed experts for a finished request., so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `_maybe_collect_routed_experts(req)`。它围绕 `_maybe_collect_routed_experts` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 143-151: Implement maybe collect indexer topk / 实现maybe collect indexer topk
```python
    def _maybe_collect_indexer_topk(self, req: Req):
        capturer = get_global_indexer_capturer()
        if capturer is None:
            return
        req.indexer_topk = capturer.get_topk(
            req_pool_idx=req.req_pool_idx,
            seqlen=req.seqlen,
            req_to_token_pool=self.req_to_token_pool,
        )
```
**EN:** This block implements the method `_maybe_collect_indexer_topk(req)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `_maybe_collect_indexer_topk`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `_maybe_collect_indexer_topk(req)`。它围绕 `_maybe_collect_indexer_topk` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 153-172: Implement maybe collect customized info / 实现maybe collect customized info
```python
    def _maybe_collect_customized_info(
        self,
        i: int,
        req: Req,
        logits_output: LogitsProcessorOutput,
    ):
        if logits_output is not None and logits_output.customized_info is not None:
            if req.customized_info is None:
                req.customized_info = {}
            for k, v in logits_output.customized_info.items():
                if k not in req.customized_info:
                    req.customized_info[k] = []
                # Copy the element so it doesn't retain the entire batch
                # tensor/array via a view reference.
                elem = v[i]
                if isinstance(elem, torch.Tensor):
                    elem = elem.clone()
                elif hasattr(elem, "copy") and callable(elem.copy):
                    elem = elem.copy()
                req.customized_info[k].append(elem)
```
**EN:** This block implements the method `_maybe_collect_customized_info(i, req, logits_output)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `_maybe_collect_customized_info`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `_maybe_collect_customized_info(i, req, logits_output)`。它围绕 `_maybe_collect_customized_info` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 174-209: Implement process batch result prefill / 实现process 批处理 result prefill
```python
    def process_batch_result_prefill(
        self,
        batch: ScheduleBatch,
        result: Union[GenerationBatchResult, EmbeddingBatchResult],
    ):
        skip_stream_req = None

        if self.is_generation:
            if result.copy_done is not None:
                result.copy_done.synchronize()
            if result.routed_experts_output is not None:
                result.routed_experts_output.finalize()
                result.routed_experts_output = None
            if result.indexer_topk_output is not None:
                result.indexer_topk_output.finalize()
                result.indexer_topk_output = None

            (
                logits_output,
                next_token_ids,
                extend_input_len_per_req,
                extend_logprob_start_len_per_req,
            ) = (
                result.logits_output,
                result.next_token_ids,
                result.extend_input_len_per_req,
                result.extend_logprob_start_len_per_req,
            )

            # Move next_token_ids and logprobs to cpu
            next_token_ids = next_token_ids.tolist()
            if batch.return_logprob:
                if logits_output.next_token_logprobs is not None:
                    logits_output.next_token_logprobs = (
                        logits_output.next_token_logprobs.tolist()
                    )
```
**EN:** This block implements the method `process_batch_result_prefill(batch, result)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `process_batch_result_prefill`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `process_batch_result_prefill(batch, result)`。它围绕 `process_batch_result_prefill` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 210-245: Continue process batch result prefill / 继续说明process 批处理 result prefill
```python
                if logits_output.input_token_logprobs is not None:
                    logits_output.input_token_logprobs = tuple(
                        logits_output.input_token_logprobs.tolist()
                    )
                if logits_output.next_token_top_logprobs_val:
                    logits_output.next_token_top_logprobs_val = [
                        v.tolist() for v in logits_output.next_token_top_logprobs_val
                    ]
                    logits_output.next_token_top_logprobs_idx = [
                        x.tolist() for x in logits_output.next_token_top_logprobs_idx
                    ]
                if logits_output.next_token_token_ids_logprobs_val:
                    logits_output.next_token_token_ids_logprobs_val = [
                        v.tolist()
                        for v in logits_output.next_token_token_ids_logprobs_val
                    ]

            hidden_state_offset = 0

            # Check finish conditions
            logprob_pt = 0

            for i, (req, next_token_id) in enumerate(zip(batch.reqs, next_token_ids)):
                if req.finished() or req.is_retracted:
                    # decode req in mixed batch or retracted req
                    continue

                if req.is_chunked <= 0:
                    req.time_stats.set_prefill_finished_time()

                    # req output_ids are set here
                    req.output_ids.append(next_token_id)

                    self._maybe_update_reasoning_tokens(req, next_token_id)

                    req.check_finished()
```
**EN:** This block implements the method `process_batch_result_prefill(batch, result)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `process_batch_result_prefill`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `process_batch_result_prefill(batch, result)`。它围绕 `process_batch_result_prefill` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 246-281: Continue process batch result prefill / 继续说明process 批处理 result prefill
```python
                    if req.finished():
                        self._maybe_collect_routed_experts(req)
                        self._maybe_collect_indexer_topk(req)
                        release_kv_cache(req, self.tree_cache)
                        req.time_stats.set_completion_time()
                    elif not batch.decoding_reqs or req not in batch.decoding_reqs:
                        maybe_cache_unfinished_req(req, self.tree_cache)
                        if self.server_args.enable_hisparse:
                            self.hisparse_coordinator.admit_request_into_staging(req)

                    self._maybe_collect_customized_info(i, req, logits_output)

                    if batch.return_logprob:
                        assert extend_logprob_start_len_per_req is not None
                        assert extend_input_len_per_req is not None
                        extend_logprob_start_len = extend_logprob_start_len_per_req[i]
                        extend_input_len = extend_input_len_per_req[i]

                        num_input_logprobs = (
                            self.logprob_result_processor.calculate_num_input_logprobs(
                                req,
                                extend_input_len,
                                extend_logprob_start_len,
                            )
                        )

                        if req.return_logprob:
                            self.logprob_result_processor.add_logprob_return_values(
                                i,
                                req,
                                logprob_pt,
                                next_token_ids,
                                num_input_logprobs,
                                logits_output,
                            )
                        logprob_pt += num_input_logprobs
```
**EN:** This block implements the method `process_batch_result_prefill(batch, result)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `process_batch_result_prefill`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `process_batch_result_prefill(batch, result)`。它围绕 `process_batch_result_prefill` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 282-317: Continue process batch result prefill / 继续说明process 批处理 result prefill
```python

                    if (
                        req.return_hidden_states
                        and logits_output.hidden_states is not None
                    ):
                        req.hidden_states.append(
                            logits_output.hidden_states[
                                hidden_state_offset : (
                                    hidden_state_offset := hidden_state_offset
                                    + len(req.origin_input_ids)
                                )
                            ]
                            .cpu()
                            .clone()
                            .tolist()
                        )

                    if req.grammar is not None:
                        # FIXME: this try-except block is for handling unexpected xgrammar issue.
                        try:
                            req.grammar.accept_token(next_token_id)
                        except ValueError as e:
                            # Grammar accept_token can raise ValueError if the token is not in the grammar.
                            # This can happen if the grammar is not set correctly or the token is invalid.
                            logger.error(
                                f"Grammar accept_token failed for req {req.rid} with token {next_token_id}: {e}"
                            )
                            self.abort_request(AbortReq(rid=req.rid))
                        req.grammar.finished = req.finished()

                else:
                    # being chunked reqs' prefill is not finished
                    req.is_chunked -= 1
                    # There is only at most one request being currently chunked.
                    # Because this request does not finish prefill,
                    # we don't want to stream the request currently being chunked.
```
**EN:** This block implements the method `process_batch_result_prefill(batch, result)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `process_batch_result_prefill`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `process_batch_result_prefill(batch, result)`。它围绕 `process_batch_result_prefill` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 318-353: Continue process batch result prefill / 继续说明process 批处理 result prefill
```python
                    skip_stream_req = req

                    # Incrementally update input logprobs.
                    if batch.return_logprob:
                        extend_logprob_start_len = extend_logprob_start_len_per_req[i]
                        extend_input_len = extend_input_len_per_req[i]
                        if extend_logprob_start_len < extend_input_len:
                            # Update input logprobs.
                            num_input_logprobs = self.logprob_result_processor.calculate_num_input_logprobs(
                                req,
                                extend_input_len,
                                extend_logprob_start_len,
                            )
                            if req.return_logprob:
                                self.logprob_result_processor.add_input_logprob_return_values(
                                    i,
                                    req,
                                    logits_output,
                                    logprob_pt,
                                    num_input_logprobs,
                                    last_prefill_chunk=False,
                                )
                            logprob_pt += num_input_logprobs

                    req.time_stats.set_last_chunked_prefill_finish_time()

        else:  # embedding or reward model
            if result.copy_done is not None:
                result.copy_done.synchronize()

            is_sparse = envs.SGLANG_EMBEDDINGS_SPARSE_HEAD.is_set()

            embeddings = result.embeddings
            phs = result.pooled_hidden_states

            if is_sparse:
```
**EN:** This block implements the method `process_batch_result_prefill(batch, result)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `process_batch_result_prefill`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `process_batch_result_prefill(batch, result)`。它围绕 `process_batch_result_prefill` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 354-389: Continue process batch result prefill / 继续说明process 批处理 result prefill
```python
                batch_ids, token_ids = embeddings.indices()
                values = embeddings.values()

                embeddings = [{} for _ in range(embeddings.size(0))]
                for i in range(batch_ids.shape[0]):
                    embeddings[batch_ids[i].item()][token_ids[i].item()] = values[
                        i
                    ].item()
            else:
                if isinstance(embeddings, torch.Tensor):
                    embeddings = embeddings.tolist()
                else:
                    embeddings = [tensor.tolist() for tensor in embeddings]

            if phs is not None:
                if isinstance(phs, list):
                    phs = [t.cpu().detach() for t in phs]
                else:
                    phs = phs.cpu().detach()

            # Check finish conditions
            for i, req in enumerate(batch.reqs):
                if req.is_retracted:
                    continue

                req.embedding = embeddings[i]
                if req.return_pooled_hidden_states and phs is not None:
                    req.pooled_hidden_state = phs[i]
                if req.is_chunked <= 0:
                    req.time_stats.set_prefill_finished_time()
                    # Dummy output token for embedding models
                    req.output_ids.append(0)
                    req.check_finished()

                    if req.finished():
                        release_kv_cache(req, self.tree_cache)
```
**EN:** This block implements the method `process_batch_result_prefill(batch, result)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `process_batch_result_prefill`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `process_batch_result_prefill(batch, result)`。它围绕 `process_batch_result_prefill` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 390-408: Continue process batch result prefill / 继续说明process 批处理 result prefill
```python
                        req.time_stats.set_completion_time()
                    else:
                        maybe_cache_unfinished_req(req, self.tree_cache)
                else:
                    # being chunked reqs' prefill is not finished
                    req.is_chunked -= 1
                    req.time_stats.set_last_chunked_prefill_finish_time()

        self.output_streamer.stream_output(
            batch.reqs, batch.return_logprob, skip_stream_req
        )

        can_run_cuda_graph = getattr(result, "can_run_cuda_graph", False)
        self.metrics_reporter.report_prefill_stats(
            batch=batch,
            prefill_stats=batch.prefill_stats,
            can_run_cuda_graph=can_run_cuda_graph,
            dp_cooperation_info=batch.dp_cooperation_info,
        )
```
**EN:** This block implements the method `process_batch_result_prefill(batch, result)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `process_batch_result_prefill`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `process_batch_result_prefill(batch, result)`。它围绕 `process_batch_result_prefill` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 410-457: Implement resolve spec overlap tokens / 实现resolve spec overlap tokens
```python
    def _resolve_spec_overlap_tokens(
        self,
        result: GenerationBatchResult,
        batch: ScheduleBatch,
    ) -> List[List[int]]:
        """Resolve the padding next token ids for speculative decoding with overlap."""
        assert result.next_token_ids.is_cpu
        assert result.accept_lens.is_cpu

        next_token_ids = result.next_token_ids.tolist()
        accept_lens = result.accept_lens.tolist()
        result.num_correct_drafts = sum(accept_lens) - len(batch.reqs)
        result.num_correct_drafts_per_req_cpu = [x - 1 for x in accept_lens]

        # Feed the adaptive controller now that accept_lens is on CPU,
        # instead of doing a synchronous GPU→CPU copy in the worker hot path.
        # BaseSpecWorker provides a no-op default for non-adaptive workers.
        self.model_worker.on_verify_complete_cpu(result.num_correct_drafts_per_req_cpu)

        predict_tokens = []
        # In adaptive spec-v2, the worker state may already have switched when this
        # delayed result is processed. Use the draft token count recorded on result.
        stride = result.speculative_num_draft_tokens
        assert stride is not None, "spec-v2 result missing speculative_num_draft_tokens"

        for i, req in enumerate(batch.reqs):
            predict_tokens.append(
                next_token_ids[i * stride : i * stride + accept_lens[i]]
            )

            if req.is_retracted:
                # reset_for_retract() already zeroes committed/allocated KV.
                continue

            if req.finished():
                # -1 because prepare_for_decode pre-claimed the bonus slot.
                req.kv_committed_len -= 1
                continue

            # -1 because prepare_for_decode pre-claimed the bonus slot.
            req.kv_committed_len += accept_lens[i] - 1
            req.spec_verify_ct += 1

            num_correct_drafts = result.num_correct_drafts_per_req_cpu[i]
            req.spec_num_correct_drafts += num_correct_drafts
            req.update_spec_correct_drafts_histogram(num_correct_drafts)

        return predict_tokens
```
**EN:** This block implements the method `_resolve_spec_overlap_tokens(result, batch)` on `SchedulerBatchResultProcessor`. It focuses on Resolve the padding next token ids for speculative decoding with overlap., so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `_resolve_spec_overlap_tokens(result, batch)`。它围绕 `_resolve_spec_overlap_tokens` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 459-469: Implement process batch result idle / 实现process 批处理 result idle
```python
    def process_batch_result_idle(
        self,
        batch: ScheduleBatch,
        result: GenerationBatchResult,
    ):
        if result.copy_done is not None:
            result.copy_done.synchronize()

        self.output_streamer._stream_output_generation(
            batch.reqs, batch.return_logprob, is_idle_batch=True
        )
```
**EN:** This block implements the method `process_batch_result_idle(batch, result)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `process_batch_result_idle`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `process_batch_result_idle(batch, result)`。它围绕 `process_batch_result_idle` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 471-506: Implement process batch result decode / 实现process 批处理 result 解码
```python
    def process_batch_result_decode(
        self,
        batch: ScheduleBatch,
        result: GenerationBatchResult,
    ):
        if result.copy_done is not None:
            result.copy_done.synchronize()
        if result.routed_experts_output is not None:
            result.routed_experts_output.finalize()
            result.routed_experts_output = None
        if result.indexer_topk_output is not None:
            result.indexer_topk_output.finalize()
            result.indexer_topk_output = None

        logits_output, next_token_ids, can_run_cuda_graph = (
            result.logits_output,
            result.next_token_ids,
            result.can_run_cuda_graph,
        )

        if batch.spec_algorithm.is_none() or batch.is_spec_v2:
            if batch.is_spec_v2:
                next_token_ids = self._resolve_spec_overlap_tokens(result, batch)
            elif isinstance(next_token_ids, list):
                pass  # MLX path: already a list[int], skip torch round-trip
            else:
                next_token_ids = next_token_ids.tolist()

            if batch.return_logprob:
                next_token_logprobs = logits_output.next_token_logprobs.tolist()
                if logits_output.next_token_top_logprobs_val:
                    logits_output.next_token_top_logprobs_val = [
                        v.tolist() for v in logits_output.next_token_top_logprobs_val
                    ]
                    logits_output.next_token_top_logprobs_idx = [
                        x.tolist() for x in logits_output.next_token_top_logprobs_idx
```
**EN:** This block implements the method `process_batch_result_decode(batch, result)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `process_batch_result_decode`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `process_batch_result_decode(batch, result)`。它围绕 `process_batch_result_decode` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 507-542: Continue process batch result decode / 继续说明process 批处理 result 解码
```python
                    ]

                if logits_output.next_token_token_ids_logprobs_val:
                    logits_output.next_token_token_ids_logprobs_val = [
                        v.tolist()
                        for v in logits_output.next_token_token_ids_logprobs_val
                    ]
        # else: Spec V1 — output_ids, check_finished, grammar, and reasoning tokens
        # are already handled in the verify phase (eagle_info.py / ngram_info.py).

        self.metrics_reporter.num_generated_tokens += len(batch.reqs)
        if not batch.spec_algorithm.is_none():
            self.metrics_reporter.update_spec_metrics(
                batch.batch_size(), result.num_correct_drafts
            )
        if self.server_args.enable_metrics:
            self.metrics_collector.increment_decode_cuda_graph_pass(
                value=can_run_cuda_graph
            )

        self.token_to_kv_pool_allocator.free_group_begin()

        # Spec V1 handles output_ids, check_finished, grammar, and reasoning tokens
        # in the verify phase. Non-spec and V2 handle them here in post-processing.
        is_spec_v1 = not batch.spec_algorithm.is_none() and not batch.is_spec_v2

        for i, req in enumerate(batch.reqs):
            req: Req

            if (self.enable_overlap or self.enable_overlap_mlx) and (
                req.finished() or req.is_retracted
            ):
                # NOTE: This (req.finished() or req.is_retracted) should only happen when overlap scheduling is enabled.
                # And all the over-allocated tokens will be freed in `release_kv_cache`.
                continue
```
**EN:** This block implements the method `process_batch_result_decode(batch, result)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `process_batch_result_decode`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `process_batch_result_decode(batch, result)`。它围绕 `process_batch_result_decode` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 543-578: Continue process batch result decode / 继续说明process 批处理 result 解码
```python
            if is_spec_v1:
                self._mamba_prefix_cache_update(req, batch, result, i)
                req.time_stats.set_last_decode_finish_time()
                self._handle_finished_req(req, i, logits_output)
                if req.return_hidden_states and logits_output.hidden_states is not None:
                    req.hidden_states.append(
                        logits_output.hidden_states[i].cpu().clone().tolist()
                    )
                if req.grammar is not None:
                    req.grammar.finished = req.finished()
                continue

            # Non-spec and V2: full post-processing
            next_token_id = next_token_ids[i]
            new_accepted_len = 1
            if batch.spec_algorithm.is_none():
                req.output_ids.append(next_token_id)
            else:
                req.output_ids.extend(next_token_id)
                new_accepted_len = len(next_token_id)

            self._maybe_update_reasoning_tokens(req, next_token_id)

            # Update Mamba last track seqlen
            self._mamba_prefix_cache_update(req, batch, result, i)
            req.time_stats.set_last_decode_finish_time()
            req.check_finished(new_accepted_len)

            self._handle_finished_req(req, i, logits_output)

            if req.return_logprob:
                # Spec v1 handles logprobs inside its own worker.
                # Normalize: non-spec has 1 token, spec v2 has multiple.
                if batch.is_spec_v2:
                    accepted_logprobs = next_token_logprobs[i]
                    accepted_ids = next_token_id
```
**EN:** This block implements the method `process_batch_result_decode(batch, result)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `process_batch_result_decode`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `process_batch_result_decode(batch, result)`。它围绕 `process_batch_result_decode` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 579-614: Continue process batch result decode / 继续说明process 批处理 result 解码
```python
                    max_accept = len(accepted_logprobs)
                else:
                    accepted_logprobs = [next_token_logprobs[i]]
                    accepted_ids = [next_token_id]
                    max_accept = 1

                for j, tok_id in enumerate(accepted_ids):
                    req.output_token_logprobs_val.append(accepted_logprobs[j])
                    req.output_token_logprobs_idx.append(tok_id)
                    if req.top_logprobs_num > 0:
                        flat_idx = i * max_accept + j
                        req.output_top_logprobs_val.append(
                            logits_output.next_token_top_logprobs_val[flat_idx]
                        )
                        req.output_top_logprobs_idx.append(
                            logits_output.next_token_top_logprobs_idx[flat_idx]
                        )
                    if req.token_ids_logprob is not None:
                        flat_idx = i * max_accept + j
                        req.output_token_ids_logprobs_val.append(
                            logits_output.next_token_token_ids_logprobs_val[flat_idx]
                        )
                        req.output_token_ids_logprobs_idx.append(
                            logits_output.next_token_token_ids_logprobs_idx[flat_idx]
                        )

            if req.return_hidden_states and logits_output.hidden_states is not None:
                req.hidden_states.append(
                    logits_output.hidden_states[i].cpu().clone().tolist()
                )

            if req.grammar is not None:
                # FIXME: this try-except block is for handling unexpected xgrammar issue.
                try:
                    if batch.spec_algorithm.is_none():
                        # Normal decode: single token
```
**EN:** This block implements the method `process_batch_result_decode(batch, result)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `process_batch_result_decode`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `process_batch_result_decode(batch, result)`。它围绕 `process_batch_result_decode` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 615-639: Continue process batch result decode / 继续说明process 批处理 result 解码
```python
                        req.grammar.accept_token(next_token_id)
                    elif batch.is_spec_v2:
                        # Speculative decode: next_token_id is a list of accepted tokens
                        for token_id in next_token_id:
                            req.grammar.accept_token(token_id)
                except ValueError as e:
                    # Grammar accept_token can raise ValueError if the token is not in the grammar.
                    # This can happen if the grammar is not set correctly or the token is invalid.
                    logger.error(
                        f"Grammar accept_token failed for req {req.rid} with token {next_token_id}: {e}"
                    )
                    self.abort_request(AbortReq(rid=req.rid))
                req.grammar.finished = req.finished()

        self.output_streamer.stream_output(batch.reqs, batch.return_logprob)
        self.token_to_kv_pool_allocator.free_group_end()

        self.metrics_reporter.forward_ct_decode = (
            self.metrics_reporter.forward_ct_decode + 1
        ) % (1 << 30)
        self.metrics_reporter.report_decode_stats(
            can_run_cuda_graph,
            running_batch=batch,
            num_correct_drafts=result.num_correct_drafts,
        )
```
**EN:** This block implements the method `process_batch_result_decode(batch, result)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `process_batch_result_decode`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `process_batch_result_decode(batch, result)`。它围绕 `process_batch_result_decode` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 641-671: Implement handle finished req / 实现handle finished req
```python
    def _handle_finished_req(
        self,
        req: Req,
        i: int,
        logits_output: LogitsProcessorOutput,
    ):
        if (
            self.server_args.disaggregation_decode_enable_offload_kvcache
            and not req.finished()
        ):
            self.decode_offload_manager.offload_kv_cache(req)

        if req.finished():
            # delete feature to save memory
            if req.multimodal_inputs is not None and req.session is None:
                req.multimodal_inputs.release_features()
            self._maybe_collect_routed_experts(req)
            self._maybe_collect_indexer_topk(req)

            if self.server_args.disaggregation_decode_enable_offload_kvcache:
                # Asynchronously offload KV cache; release_kv_cache will be called after Device->Host transfer completes
                if not self.decode_offload_manager.offload_kv_cache(req):
                    self.decode_offload_manager.finalize_release_on_finish(req)
            else:
                if self.server_args.enable_hisparse:
                    self.hisparse_coordinator.request_finished(req)
                release_kv_cache(req, self.tree_cache)

            req.time_stats.set_completion_time()

        self._maybe_collect_customized_info(i, req, logits_output)
```
**EN:** This block implements the method `_handle_finished_req(req, i, logits_output)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `_handle_finished_req`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `_handle_finished_req(req, i, logits_output)`。它围绕 `_handle_finished_req` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 673-680: Implement maybe update reasoning tokens / 实现maybe update reasoning tokens
```python
    def _maybe_update_reasoning_tokens(
        self,
        req: Req,
        next_token_id: Union[int, List[int]],
    ):
        think_end_id = self.model_config.think_end_id
        if req.require_reasoning and think_end_id is not None:
            req.update_reasoning_tokens(next_token_id, think_end_id)
```
**EN:** This block implements the method `_maybe_update_reasoning_tokens(req, next_token_id)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `_maybe_update_reasoning_tokens`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `_maybe_update_reasoning_tokens(req, next_token_id)`。它围绕 `_maybe_update_reasoning_tokens` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 682-718: Implement mamba prefix cache update / 实现mamba prefix 缓存 update
```python
    def _mamba_prefix_cache_update(
        self,
        req: Req,
        batch: ScheduleBatch,
        result: GenerationBatchResult,
        i: int,
    ) -> None:
        seq_len = len(req.origin_input_ids) + len(req.output_ids) - 1
        if req.mamba_ping_pong_track_buffer is not None:
            mamba_track_interval = get_global_server_args().mamba_track_interval
            if batch.spec_algorithm.is_none() and seq_len % mamba_track_interval == 0:
                # for non-spec decode, we update mamba_last_track_seqlen at the end of each track interval
                req.mamba_next_track_idx = (
                    batch.req_to_token_pool.get_mamba_ping_pong_other_idx(
                        req.mamba_next_track_idx
                    )
                )
                req.mamba_last_track_seqlen = seq_len
            elif (
                not batch.spec_algorithm.is_none()
                and result.num_correct_drafts_per_req_cpu is not None
            ):
                # for spec decode, update mamba_last_track_seqlen if this iteration crosses a track interval
                actual_seq_len = req.seqlen - 1
                if (
                    actual_seq_len // mamba_track_interval
                    != (actual_seq_len - result.num_correct_drafts_per_req_cpu[i] - 1)
                    // mamba_track_interval
                ):
                    req.mamba_next_track_idx = (
                        batch.req_to_token_pool.get_mamba_ping_pong_other_idx(
                            req.mamba_next_track_idx
                        )
                    )
                    req.mamba_last_track_seqlen = (
                        actual_seq_len // mamba_track_interval * mamba_track_interval
                    )
```
**EN:** This block implements the method `_mamba_prefix_cache_update(req, batch, result, i)` on `SchedulerBatchResultProcessor`. It focuses on handling the batch result processor responsibilities represented by `_mamba_prefix_cache_update`, so the class can advance the batch result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerBatchResultProcessor` 上的方法 `_mamba_prefix_cache_update(req, batch, result, i)`。它围绕 `_mamba_prefix_cache_update` 所承担的 批处理 result 处理器 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: SchedulerBatchResultProcessor
- **Domain focus / 领域焦点**: batch result processor / 批处理 result 处理器
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: dataclasses, logging, typing
- **Third-party / 第三方库**: __future__, torch
- **Local Modules / 本地模块**: sglang.srt.configs.model_config, sglang.srt.disaggregation.decode_kvcache_offload_manager, sglang.srt.disaggregation.utils, sglang.srt.environ, sglang.srt.layers.logits_processor, sglang.srt.managers.hisparse_coordinator, sglang.srt.managers.io_struct, sglang.srt.managers.schedule_batch, sglang.srt.managers.scheduler_components.logprob_result_processor, sglang.srt.managers.scheduler_components.metrics_reporter, sglang.srt.managers.scheduler_components.output_streamer, sglang.srt.managers.tp_worker
