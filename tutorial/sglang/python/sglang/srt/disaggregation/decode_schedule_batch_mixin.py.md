# decode_schedule_batch_mixin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/disaggregation/decode_schedule_batch_mixin.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file provides a mixin with decode scheduling helpers that understand disaggregated transfer state. It keeps the scheduler aware of requests that are still waiting for remote KV data. / 该文件提供了理解解耦传输状态的解码调度辅助混入类，使调度器能够感知仍在等待远端 KV 数据的请求。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Imports and module setup
```python
from __future__ import annotations

import logging
from http import HTTPStatus
from typing import TYPE_CHECKING

import torch

from sglang.srt.mem_cache.common import maybe_cache_unfinished_req
from sglang.srt.model_executor.forward_batch_info import CaptureHiddenMode, ForwardMode
from sglang.srt.sampling.sampling_batch_info import SamplingBatchInfo

logger = logging.getLogger(__name__)

if TYPE_CHECKING:
    from sglang.srt.managers.overlap_utils import FutureMap
    from sglang.srt.managers.schedule_batch import ScheduleBatch
    from sglang.srt.server_args import ServerArgs


```
**EN:** This block gathers the imports and module-level setup for decode batch scheduling helpers. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `getLogger`.
**CN:** 这一段汇集了与解码批次调度辅助逻辑相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `getLogger`。

### Lines 21-22: Class `ScheduleBatchDisaggregationDecodeMixin` declaration
```python
class ScheduleBatchDisaggregationDecodeMixin:

```
**EN:** This block declares the class `ScheduleBatchDisaggregationDecodeMixin` and establishes its responsibility inside decode batch scheduling helpers. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `ScheduleBatchDisaggregationDecodeMixin`.
**CN:** 这一段声明了类 `ScheduleBatchDisaggregationDecodeMixin`，并说明它在解码批次调度辅助逻辑中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `ScheduleBatchDisaggregationDecodeMixin`。

### Lines 23-27: Method `prepare_for_prebuilt` signature and setup
```python
    def prepare_for_prebuilt(self: ScheduleBatch):
        """
        Prepare a prebuilt extend by populate metadata
        Adapted from .prepare_for_extend().
        """
```
**EN:** This block defines the method `prepare_for_prebuilt` on `ScheduleBatchDisaggregationDecodeMixin`. It introduces the parameters, setup steps, and the main entry point for this piece of decode batch scheduling helpers. Definitions introduced here include `prepare_for_prebuilt`. Notable operations include `prepare_for_extend`.
**CN:** 这一段定义了method `prepare_for_prebuilt`（属于 `ScheduleBatchDisaggregationDecodeMixin`），介绍了参数、初始化步骤，以及这部分解码批次调度辅助逻辑逻辑的主要入口。 此处引入的定义包括 `prepare_for_prebuilt`。 值得注意的操作包括 `prepare_for_extend`。

### Lines 28-54: Method `prepare_for_prebuilt` logic (part 1)
```python

        self.forward_mode = ForwardMode.PREBUILT
        reqs = self.reqs
        input_ids = [r.fill_ids[len(r.prefix_indices) :] for r in reqs]
        extend_num_tokens = sum(len(ids) for ids in input_ids)
        seq_lens = []
        pre_lens = []
        req_pool_indices = []

        # Pre-calculate total size
        total_size = sum(req.extend_input_len for req in reqs)
        out_cache_loc = torch.empty(total_size, dtype=torch.int64, device=self.device)

        # Fill the tensor in one pass
        offset = 0
        for i, req in enumerate(reqs):
            req_pool_indices.append(req.req_pool_idx)
            pre_len = len(req.prefix_indices)

            chunk = self.req_to_token_pool.req_to_token[req.req_pool_idx][
                pre_len : pre_len + req.extend_input_len
            ]
            assert (
                offset + req.extend_input_len <= total_size
            ), f"Exceeds total size: offset={offset}, req.extend_input_len={req.extend_input_len}, total_size={total_size}"
            out_cache_loc[offset : offset + req.extend_input_len] = chunk
            offset += req.extend_input_len
```
**EN:** This block continues `prepare_for_prebuilt` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding decode batch scheduling helpers workflow. Notable operations include `empty`, `append`.
**CN:** 这一段延续了 `prepare_for_prebuilt` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的解码批次调度辅助逻辑工作流。 值得注意的操作包括 `empty`、`append`。

### Lines 55-70: Method `prepare_for_prebuilt` logic (part 2)
```python

            seq_len = len(req.origin_input_ids) + max(0, len(req.output_ids) - 1)
            seq_lens.append(seq_len)
            if len(req.output_ids) == 0:
                assert (
                    seq_len - pre_len == req.extend_input_len
                ), f"seq_len={seq_len}, pre_len={pre_len}, req.extend_input_len={req.extend_input_len}"

            if not req.retracted_stain:
                req.cached_tokens += pre_len - req.already_computed
                req.already_computed = seq_len
            req.is_retracted = False
            pre_lens.append(pre_len)
            req.extend_logprob_start_len = 0

        extend_input_logprob_token_ids = None
```
**EN:** This block continues `prepare_for_prebuilt` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding decode batch scheduling helpers workflow. Notable operations include `append`.
**CN:** 这一段延续了 `prepare_for_prebuilt` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的解码批次调度辅助逻辑工作流。 值得注意的操作包括 `append`。

### Lines 71-96: Method `prepare_for_prebuilt` logic (part 3)
```python

        # Set fields
        self.input_ids = torch.tensor(
            sum(input_ids, []), dtype=torch.int32, device=self.device
        )
        self.req_pool_indices = torch.tensor(
            req_pool_indices, dtype=torch.int64, device=self.device
        )
        self.seq_lens = torch.tensor(seq_lens, dtype=torch.int64, device=self.device)
        self.seq_lens_cpu = torch.tensor(seq_lens, dtype=torch.int64)
        self.orig_seq_lens = torch.tensor(
            seq_lens, dtype=torch.int32, device=self.device
        )
        self.out_cache_loc = out_cache_loc
        self.seq_lens_sum = sum(seq_lens)

        if self.return_logprob:
            self.top_logprobs_nums = [r.top_logprobs_num for r in reqs]
            self.token_ids_logprobs = [r.token_ids_logprob for r in reqs]

        self.extend_num_tokens = extend_num_tokens
        self.prefix_lens = [len(r.prefix_indices) for r in reqs]
        self.extend_lens = [r.extend_input_len for r in reqs]
        self.extend_logprob_start_lens = [r.extend_logprob_start_len for r in reqs]
        self.extend_input_logprob_token_ids = extend_input_logprob_token_ids
        self.multimodal_inputs = [r.multimodal_inputs for r in reqs]
```
**EN:** This block continues `prepare_for_prebuilt` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding decode batch scheduling helpers workflow. Notable operations include `tensor`.
**CN:** 这一段延续了 `prepare_for_prebuilt` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的解码批次调度辅助逻辑工作流。 值得注意的操作包括 `tensor`。

### Lines 97-102: Method `prepare_for_prebuilt` logic (part 4)
```python

        # Build sampling info
        self.sampling_info = SamplingBatchInfo.from_schedule_batch(
            self,
            self.model_config.vocab_size,
        )
```
**EN:** This block continues `prepare_for_prebuilt` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding decode batch scheduling helpers workflow. Notable operations include `from_schedule_batch`.
**CN:** 这一段延续了 `prepare_for_prebuilt` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的解码批次调度辅助逻辑工作流。 值得注意的操作包括 `from_schedule_batch`。

### Lines 104-109: Method `process_prebuilt` signature and setup
```python
    def process_prebuilt(
        self: ScheduleBatch,
        server_args: ServerArgs,
        future_map: FutureMap,
    ):
        """Assign the buffered last input id to schedule batch"""
```
**EN:** This block defines the method `process_prebuilt` on `ScheduleBatchDisaggregationDecodeMixin`. It introduces the parameters, setup steps, and the main entry point for this piece of decode batch scheduling helpers. Definitions introduced here include `process_prebuilt`.
**CN:** 这一段定义了method `process_prebuilt`（属于 `ScheduleBatchDisaggregationDecodeMixin`），介绍了参数、初始化步骤，以及这部分解码批次调度辅助逻辑逻辑的主要入口。 此处引入的定义包括 `process_prebuilt`。

### Lines 110-133: Method `process_prebuilt` logic (part 1)
```python
        self.output_ids = []
        for req in self.reqs:
            self.output_ids.append(req.output_ids[-1])
            maybe_cache_unfinished_req(req, self.tree_cache)
            if req.grammar is not None:
                # FIXME: this try-except block is for handling unexpected xgrammar issue.
                try:
                    # if it is not None, then the grammar is from a retracted request, and we should not
                    # accept the token as it's already accepted
                    if req.grammar.current_token is None:
                        req.grammar.accept_token(req.output_ids[-1])
                except ValueError as e:
                    from sglang.srt.managers.schedule_batch import FINISH_ABORT

                    # Grammar accept_token can raise ValueError if the token is not in the grammar.
                    # This can happen if the grammar is not set correctly or the token is invalid.
                    # Use to_finish (not finished_reason) so that process_batch_result_prebuilt
                    # handles the release via check_finished -> release_kv_cache in one place.
                    error_message = f"Grammar accept_token failed for req {req.rid} with token {req.output_ids[-1]}: {e}"
                    req.to_finish = FINISH_ABORT(
                        error_message, HTTPStatus.INTERNAL_SERVER_ERROR
                    )
                req.grammar.finished = req.finished()
        self.output_ids = torch.tensor(self.output_ids, device=self.device)
```
**EN:** This block continues `process_prebuilt` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding decode batch scheduling helpers workflow. Notable operations include `append`, `maybe_cache_unfinished_req`, `accept_token`, `to_finish`.
**CN:** 这一段延续了 `process_prebuilt` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的解码批次调度辅助逻辑工作流。 值得注意的操作包括 `append`、`maybe_cache_unfinished_req`、`accept_token`、`to_finish`。

### Lines 134-164: Method `process_prebuilt` logic (part 2)
```python

        # Simulate the eagle run.
        if self.spec_algorithm.is_eagle():
            num_states = server_args.speculative_eagle_topk
            if server_args.enable_multi_layer_eagle:
                num_states *= server_args.speculative_num_steps
            topk_p = torch.stack(
                [
                    torch.as_tensor(
                        req.output_topk_p[:num_states],
                        device=self.device,
                        dtype=torch.float32,
                    )
                    for req in self.reqs
                ],
                dim=0,
            )
            topk_index = torch.stack(
                [
                    torch.as_tensor(
                        req.output_topk_index[:num_states],
                        device=self.device,
                        dtype=torch.int64,
                    )
                    for req in self.reqs
                ],
                dim=0,
            )

            hidden_states_list = [req.hidden_states_tensor for req in self.reqs]
            hidden_states = torch.stack(hidden_states_list, dim=0).to(self.device)
```
**EN:** This block continues `process_prebuilt` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding decode batch scheduling helpers workflow. Notable operations include `is_eagle`, `stack`, `as_tensor`, `to`.
**CN:** 这一段延续了 `process_prebuilt` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的解码批次调度辅助逻辑工作流。 值得注意的操作包括 `is_eagle`、`stack`、`as_tensor`、`to`。

### Lines 165-185: Method `process_prebuilt` logic (part 3)
```python

            # local import to avoid circular import
            from sglang.srt.speculative.eagle_info import EagleDraftInput

            spec_info = EagleDraftInput(
                topk_p=topk_p,
                topk_index=topk_index,
                hidden_states=hidden_states,
                bonus_tokens=self.output_ids,
                new_seq_lens=self.seq_lens,
            )
            spec_info.prepare_for_extend(self)
            spec_info.capture_hidden_mode = CaptureHiddenMode.LAST
            if self.enable_overlap:
                spec_info.future_indices = future_map.alloc_future_indices(
                    len(self.seq_lens)
                )
                future_map.store_to_map_for_new_batch(
                    spec_info.future_indices, spec_info
                )
            self.spec_info = spec_info
```
**EN:** This block continues `process_prebuilt` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding decode batch scheduling helpers workflow. Notable operations include `EagleDraftInput`, `prepare_for_extend`, `alloc_future_indices`, `store_to_map_for_new_batch`.
**CN:** 这一段延续了 `process_prebuilt` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的解码批次调度辅助逻辑工作流。 值得注意的操作包括 `EagleDraftInput`、`prepare_for_extend`、`alloc_future_indices`、`store_to_map_for_new_batch`。

## Key Concepts / 关键概念
- `ScheduleBatchDisaggregationDecodeMixin`: Class that encapsulates schedule batch disaggregation decode mixin behavior in this module. / `ScheduleBatchDisaggregationDecodeMixin`：封装与“调度批次解耦部署解码mixin”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `http`, `typing`
- **External packages / 外部依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.mem_cache.common`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.sampling.sampling_batch_info`, `sglang.srt.managers.overlap_utils`, `sglang.srt.managers.schedule_batch`, `sglang.srt.server_args`, `sglang.srt.speculative.eagle_info`
