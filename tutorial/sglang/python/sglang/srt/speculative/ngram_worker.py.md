# ngram_worker.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/ngram_worker.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements speculative decoding infrastructure, such as workers, metadata objects, CUDA-graph runners, or shared utilities. It coordinates draft-model execution with target-model verification to improve throughput. / 该文件实现了推测解码基础设施，例如 worker、元数据对象、CUDA Graph 运行器或共享工具函数。它通过协调草稿模型执行与目标模型校验来提升吞吐。

## Line-by-Line Analysis / 逐行分析

### Lines 1-26: Module header, imports, and shared constants
```python
import logging
from typing import List, Optional

import numpy as np
import torch
from sgl_kernel.speculative import reconstruct_indices_from_tree_mask

from sglang.srt.layers.utils.logprob import add_output_logprobs_for_spec_v1
from sglang.srt.managers.schedule_batch import ScheduleBatch
from sglang.srt.managers.scheduler import GenerationBatchResult
from sglang.srt.managers.tp_worker import TpModelWorker
from sglang.srt.model_executor.forward_batch_info import ForwardMode
from sglang.srt.observability.req_time_stats import set_time_batch
from sglang.srt.observability.trace import get_global_tracing_enabled
from sglang.srt.server_args import ServerArgs
from sglang.srt.speculative.cpp_ngram.ngram_corpus import NgramCorpus
from sglang.srt.speculative.ngram_info import NgramVerifyInput
from sglang.srt.speculative.spec_info import SpeculativeAlgorithm
from sglang.srt.speculative.spec_utils import generate_token_bitmask

logger = logging.getLogger(__name__)


USE_FULL_MASK = True
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 27-27: NGRAMWorker class declaration
```python
class NGRAMWorker:
```
**EN:** This block declares the `NGRAMWorker` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `NGRAMWorker` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 28-81: NGRAMWorker initializer
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
        self.target_worker = target_worker
        self.model_runner = target_worker.model_runner
        self.tp_rank = tp_rank
        self.page_size = server_args.page_size
        self.draft_token_num: int = server_args.speculative_num_draft_tokens
        self.max_trie_depth: int = server_args.speculative_ngram_max_trie_depth

        self.max_batch_size = target_worker.max_running_requests
        self.device = f"cuda:{gpu_id}" if gpu_id >= 0 else "cuda"

        self._init_preallocated_tensors()

        self.ngram_corpus = NgramCorpus(
            min_bfs_breadth=server_args.speculative_ngram_min_bfs_breadth,
            max_bfs_breadth=server_args.speculative_ngram_max_bfs_breadth,
            match_type=server_args.speculative_ngram_match_type,
            capacity=server_args.speculative_ngram_capacity,
            max_trie_depth=server_args.speculative_ngram_max_trie_depth,
            draft_token_num=server_args.speculative_num_draft_tokens,
            external_sam_budget=server_args.speculative_ngram_external_sam_budget,
            external_corpus_max_tokens=server_args.speculative_ngram_external_corpus_max_tokens,
        )
        if server_args.speculative_ngram_external_corpus_path is not None:
            from sglang.srt.speculative.cpp_ngram.external_corpus import (
                iter_external_corpus_chunks,
            )

            corpus_path = server_args.speculative_ngram_external_corpus_path
            chunks = list(
                iter_external_corpus_chunks(
                    corpus_path,
                    target_worker.tokenizer,
                    server_args.speculative_ngram_external_corpus_max_tokens,
                )
            )
            loaded = self.add_external_corpus(corpus_path, chunks)
            self.commit_corpus_load(corpus_path, loaded)
            logger.info(
                "Loaded external ngram corpus '%s' (%d tokens).",
                corpus_path,
                loaded,
            )
```
**EN:** This block initializes the `NGRAMWorker` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `NGRAMWorker` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 83-84: NGRAMWorker.clear_cache_pool method
```python
    def clear_cache_pool(self):
        self.ngram_corpus.reset()
```
**EN:** This block uses `NGRAMWorker.clear_cache_pool` to clear cached state or buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NGRAMWorker.clear_cache_pool` 来清理缓存状态或缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 86-93: NGRAMWorker.update_weights_from_tensor method
```python
    def update_weights_from_tensor(self, recv_req):
        # NGRAM has no draft weights of its own — the n-gram corpus is a CPU
        # lookup structure built from request token streams — and its
        # `model_runner` is shared with the target worker. The scheduler
        # mixin dispatches via `self.draft_worker or self.tp_worker`, so
        # without this method any caller of `update_weights_from_tensor`
        # under `--speculative-algorithm NGRAM` raises AttributeError.
        return self.target_worker.update_weights_from_tensor(recv_req)
```
**EN:** This block uses `NGRAMWorker.update_weights_from_tensor` to refresh state using new information. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NGRAMWorker.update_weights_from_tensor` 来利用新信息刷新状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 95-96: NGRAMWorker.add_external_corpus method
```python
    def add_external_corpus(self, corpus_id: str, token_chunks: list[list[int]]) -> int:
        return self.ngram_corpus.load_external_corpus_named(corpus_id, token_chunks)
```
**EN:** This block uses `NGRAMWorker.add_external_corpus` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NGRAMWorker.add_external_corpus` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 98-99: NGRAMWorker.commit_corpus_load method
```python
    def commit_corpus_load(self, corpus_id: str, loaded_token_count: int) -> None:
        self.ngram_corpus.commit_external_corpus_load(corpus_id, loaded_token_count)
```
**EN:** This block uses `NGRAMWorker.commit_corpus_load` to load resources or weights. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NGRAMWorker.commit_corpus_load` 来加载资源或权重。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 101-102: NGRAMWorker.remove_external_corpus method
```python
    def remove_external_corpus(self, corpus_id: str) -> None:
        self.ngram_corpus.remove_external_corpus(corpus_id)
```
**EN:** This block uses `NGRAMWorker.remove_external_corpus` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NGRAMWorker.remove_external_corpus` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 104-105: NGRAMWorker.list_external_corpora method
```python
    def list_external_corpora(self) -> dict[str, int]:
        return self.ngram_corpus.list_external_corpora()
```
**EN:** This block uses `NGRAMWorker.list_external_corpora` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NGRAMWorker.list_external_corpora` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 107-113: NGRAMWorker._efficient_concat_last_n method
```python
    def _efficient_concat_last_n(self, seq1: List[int], seq2: List[int], n: int):
        seq2_len = len(seq2)
        if seq2_len >= n:
            return seq2[-n:]

        need_from_seq1 = n - seq2_len
        return seq1[-need_from_seq1:] + seq2
```
**EN:** This block uses `NGRAMWorker._efficient_concat_last_n` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NGRAMWorker._efficient_concat_last_n` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 115-163: NGRAMWorker._init_preallocated_tensors method
```python
    def _init_preallocated_tensors(self):
        max_total_drafts = self.max_batch_size * self.draft_token_num
        max_total_mask_size = (
            self.max_batch_size * self.draft_token_num * self.draft_token_num
        )

        self.draft_tokens = torch.empty(
            (max_total_drafts,), dtype=torch.int64, device=self.device
        )
        self.retrieve_indexes = torch.empty(
            (self.max_batch_size, self.draft_token_num),
            dtype=torch.int64,
            device=self.device,
        )
        self.retrieve_next_token = torch.empty(
            (self.max_batch_size, self.draft_token_num),
            dtype=torch.int64,
            device=self.device,
        )
        self.retrieve_next_sibling = torch.empty(
            (self.max_batch_size, self.draft_token_num),
            dtype=torch.int64,
            device=self.device,
        )
        self.positions = torch.empty(
            (max_total_drafts,), dtype=torch.int64, device=self.device
        )
        self.tree_mask = torch.empty(
            (max_total_mask_size,), dtype=torch.bool, device=self.device
        )

        self.draft_tokens_batch = []
        self.tree_mask_batch = []
        self.retrieve_indexes_batch = []
        self.retrieve_next_token_batch = []
        self.retrieve_next_sibling_batch = []
        self.positions_batch = []

        for bs in range(0, self.max_batch_size + 1):
            self.retrieve_indexes_batch.append(self.retrieve_indexes[:bs, :])
            self.retrieve_next_token_batch.append(self.retrieve_next_token[:bs, :])
            self.retrieve_next_sibling_batch.append(self.retrieve_next_sibling[:bs, :])
            self.positions_batch.append(self.positions[: bs * self.draft_token_num])
            self.draft_tokens_batch.append(
                self.draft_tokens[: bs * self.draft_token_num]
            )
            self.tree_mask_batch.append(
                self.tree_mask[: bs * self.draft_token_num * self.draft_token_num]
            )
```
**EN:** This block uses `NGRAMWorker._init_preallocated_tensors` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NGRAMWorker._init_preallocated_tensors` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 165-190: NGRAMWorker._prepare_draft_tokens method
```python
    def _prepare_draft_tokens(
        self, batch: ScheduleBatch
    ) -> tuple[np.ndarray, np.ndarray]:
        bs = batch.batch_size()

        self.ngram_corpus.synchronize()
        req_ids = []
        batch_tokens = []
        total_lens = []
        for req in batch.reqs:
            check_token = self._efficient_concat_last_n(
                req.origin_input_ids, req.output_ids, self.max_trie_depth
            )
            req_ids.append(req.rid)
            batch_tokens.append(check_token)
            total_lens.append(len(req.origin_input_ids) + len(req.output_ids))
        req_drafts, mask = self.ngram_corpus.batch_get(
            req_ids, batch_tokens, total_lens
        )
        total_draft_token_num = len(req_drafts)

        # Check if speculative decoding is needed; here we always enforce it
        assert (
            total_draft_token_num == bs * self.draft_token_num
        ), f"{total_draft_token_num=}, {bs=}, {self.draft_token_num=}"
        return req_drafts, mask
```
**EN:** This block uses `NGRAMWorker._prepare_draft_tokens` to prepare inputs for later stages. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NGRAMWorker._prepare_draft_tokens` 来为后续阶段准备输入。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 192-247: NGRAMWorker._prepare_for_speculative_decoding method
```python
    def _prepare_for_speculative_decoding(self, batch: ScheduleBatch):
        if batch.forward_mode.is_extend():
            return

        bs = batch.batch_size()

        retrieve_index = self.retrieve_indexes_batch[bs]
        retrieve_next_token = self.retrieve_next_token_batch[bs]
        retrieve_next_sibling = self.retrieve_next_sibling_batch[bs]
        positions = self.positions_batch[bs]
        tree_mask = self.tree_mask_batch[bs]
        draft_tokens = self.draft_tokens_batch[bs]

        req_drafts, mask = self._prepare_draft_tokens(batch)
        tree_mask.copy_(torch.from_numpy(mask), non_blocking=True)
        draft_tokens.copy_(torch.from_numpy(req_drafts), non_blocking=True)

        reconstruct_indices_from_tree_mask(
            tree_mask,
            batch.seq_lens,
            positions,  # mutable
            retrieve_index,  # mutable
            retrieve_next_token,  # mutable
            retrieve_next_sibling,  # mutable
            bs,
            self.draft_token_num,
        )

        # NOTE: QLEN_MASK is faster than FULL_MASK, but requires corresponding changes in flashinfer.
        # Testing shows about 8% performance improvement (the effect is roughly proportional to batch size).
        if USE_FULL_MASK:
            tree_mask = []
            mask = mask.reshape(
                batch.batch_size(), self.draft_token_num, self.draft_token_num
            )
            for i, req in enumerate(batch.reqs):
                seq_len = len(req.origin_input_ids) + len(req.output_ids)
                req_mask = torch.ones((self.draft_token_num, seq_len - 1)).cuda()
                req_mask = torch.cat(
                    (req_mask, torch.from_numpy(mask[i]).cuda()), dim=1
                ).to(torch.bool)
                tree_mask.append(req_mask.flatten())
            tree_mask = torch.cat(tree_mask, dim=0)

        batch.spec_algorithm = SpeculativeAlgorithm.NGRAM
        batch.forward_mode = ForwardMode.TARGET_VERIFY
        batch.spec_info = NgramVerifyInput(
            draft_tokens,
            tree_mask,
            positions,
            retrieve_index,
            retrieve_next_token,
            retrieve_next_sibling,
            self.draft_token_num,
        )
        batch.spec_info.prepare_for_verify(batch, self.page_size)
```
**EN:** This block uses `NGRAMWorker._prepare_for_speculative_decoding` to prepare inputs for later stages. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NGRAMWorker._prepare_for_speculative_decoding` 来为后续阶段准备输入。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 249-261: NGRAMWorker._update_ngram_corpus method
```python
    def _update_ngram_corpus(self, batch: ScheduleBatch):
        batch_tokens = []
        for req in batch.reqs:
            # FIXME: Whether to insert 'extend' into the cache or not, after testing,
            # there is not much difference, so we will not insert it for now.
            # if batch.forward_mode.is_extend():
            #     put_ids = req.origin_input_ids + req.output_ids
            # else:
            put_ids = self._efficient_concat_last_n(
                req.origin_input_ids, req.output_ids, self.max_trie_depth
            )
            batch_tokens.append(put_ids)
        self.ngram_corpus.batch_put(batch_tokens)
```
**EN:** This block uses `NGRAMWorker._update_ngram_corpus` to refresh state using new information. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NGRAMWorker._update_ngram_corpus` 来利用新信息刷新状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 263-322: NGRAMWorker.forward_batch_generation method (part 1/2)
```python
    def forward_batch_generation(self, batch: ScheduleBatch) -> GenerationBatchResult:
        set_time_batch(batch.reqs, "set_spec_draft_start_time", trace_only=True)

        self._prepare_for_speculative_decoding(batch)

        set_time_batch(batch.reqs, "set_spec_draft_end_time", trace_only=True)

        spec_info = batch.spec_info
        num_correct_drafts = 0
        accept_lens = None
        num_correct_drafts_per_req_cpu = None

        if batch.forward_mode.is_target_verify():
            if batch.has_grammar:
                retrieve_next_token_cpu = spec_info.retrieve_next_token.cpu()
                retrieve_next_sibling_cpu = spec_info.retrieve_next_sibling.cpu()
                draft_tokens_cpu = spec_info.draft_token.view(
                    spec_info.retrieve_next_token.shape
                ).cpu()

            set_time_batch(batch.reqs, "set_spec_verify_start_time", trace_only=True)

            batch_result = self.target_worker.forward_batch_generation(
                batch, is_verify=True
            )
            logits_output, can_run_cuda_graph = (
                batch_result.logits_output,
                batch_result.can_run_cuda_graph,
            )

            verify_input: NgramVerifyInput = batch.spec_info
            vocab_mask = None
            if batch.has_grammar:
                # Generate the logit mask for structured output.
                # Overlap the CPU operations for bitmask generation with the forward pass.
                vocab_mask = generate_token_bitmask(
                    batch.reqs,
                    verify_input,
                    retrieve_next_token_cpu,
                    retrieve_next_sibling_cpu,
                    draft_tokens_cpu,
                    batch.sampling_info.vocab_size,
                )

                if vocab_mask is not None:
                    assert verify_input.grammar is not None
                    vocab_mask = vocab_mask.to(verify_input.retrieve_next_token.device)
                    # NOTE (sk): otherwise, this vocab mask will be the one from the previous extend stage
                    # and will be applied to produce wrong results
                    batch.sampling_info.vocab_mask = None

            logits_output, next_token_ids, num_correct_drafts = verify_input.verify(
                batch, logits_output, self.page_size, vocab_mask
            )
            num_correct_drafts_per_req_cpu = (
                verify_input.num_correct_drafts.cpu().tolist()
            )

            if get_global_tracing_enabled():
                for idx, req in enumerate(batch.reqs):
```
**EN:** This block uses `NGRAMWorker.forward_batch_generation` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `NGRAMWorker.forward_batch_generation` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 323-365: NGRAMWorker.forward_batch_generation method (part 2/2)
```python
                    num_correct_drafts = (
                        verify_input.num_correct_drafts[idx].item()
                        if verify_input.num_correct_drafts is not None
                        else 0
                    )
                    req.time_stats.set_spec_verify_end_time(
                        num_correct_drafts=num_correct_drafts
                    )

            # Store accept_lens (with bonus) for per-request metrics; downstream
            # subtracts 1 to recover drafts-only counts.
            accept_lens = verify_input.num_accept_tokens
            if batch.return_logprob:
                add_output_logprobs_for_spec_v1(batch, verify_input, logits_output)
            self._update_ngram_corpus(batch)
            # Clean up per-request match state for finished/retracted requests.
            # State entries are created in _prepare_draft_tokens and cleaned here.
            # If a request is removed without passing through verify, the entry
            # persists until reset(); this is acceptable because MatchState is small.
            finished_req_ids = []
            for req in batch.reqs:
                if req.finished() or req.is_retracted:
                    finished_req_ids.append(req.rid)
            if finished_req_ids:
                self.ngram_corpus.erase_match_state(finished_req_ids)
            batch.forward_mode = ForwardMode.DECODE

        else:
            batch_result = self.target_worker.forward_batch_generation(batch)
            logits_output, next_token_ids, can_run_cuda_graph = (
                batch_result.logits_output,
                batch_result.next_token_ids,
                batch_result.can_run_cuda_graph,
            )

        return GenerationBatchResult(
            logits_output=logits_output,
            next_token_ids=next_token_ids,
            num_correct_drafts=num_correct_drafts,
            num_correct_drafts_per_req_cpu=num_correct_drafts_per_req_cpu,
            can_run_cuda_graph=can_run_cuda_graph,
            accept_lens=accept_lens,
        )
```
**EN:** This block uses `NGRAMWorker.forward_batch_generation` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `NGRAMWorker.forward_batch_generation` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

## Key Concepts / 关键概念
- Worker coordination / Worker 协调
- Speculative decoding / 推测解码
- N-gram drafting / N-gram 起草
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算
- Integration with model workers / 与模型 worker 集成

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.layers.utils.logprob`
- `sglang.srt.managers.schedule_batch`
- `sglang.srt.managers.scheduler`
- `sglang.srt.managers.tp_worker`
- `sglang.srt.model_executor.forward_batch_info`
- `sglang.srt.observability.req_time_stats`
- `sglang.srt.observability.trace`
- `sglang.srt.server_args`
- `sglang.srt.speculative.cpp_ngram.external_corpus`
- `sglang.srt.speculative.cpp_ngram.ngram_corpus`
- `sglang.srt.speculative.ngram_info`
- `sglang.srt.speculative.spec_info`
- `sglang.srt.speculative.spec_utils`
### External / 外部
- `numpy`
- `sgl_kernel`
- `torch`
- `logging` (stdlib)
- `typing` (stdlib)
