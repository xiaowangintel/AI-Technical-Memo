# logprob_result_processor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/scheduler_components/logprob_result_processor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements logprob result processor logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 logprob result 处理器 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-7: Provide supporting module logic / 提供辅助模块逻辑
```python
from dataclasses import dataclass
from typing import (
    List,
    Tuple,
)
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 9-9: Import runtime dependencies / 导入运行时依赖
```python
import torch
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 11-23: Provide supporting module logic / 提供辅助模块逻辑
```python
from sglang.srt.configs.model_config import ModelConfig
from sglang.srt.layers.logits_processor import LogitsProcessorOutput
from sglang.srt.managers.schedule_batch import Req
from sglang.srt.server_args import (
    MIS_DELIMITER_TOKEN_ID,
    ServerArgs,
)


@dataclass(kw_only=True, slots=True, frozen=True)
class SchedulerLogprobResultProcessor:
    server_args: ServerArgs
    model_config: ModelConfig
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 25-55: Implement process input token logprobs / 实现process input Token logprobs
```python
    def _process_input_token_logprobs(
        self, req: Req, input_token_logprobs: List
    ) -> None:
        """Process input token logprobs values and indices."""
        is_multi_item_scoring = self._is_multi_item_scoring(req)

        # Process logprob values - handle multi-item scoring vs regular requests
        if is_multi_item_scoring:
            # Multi-item scoring: use all logprobs as-is
            req.input_token_logprobs_val = input_token_logprobs
        else:
            # Regular request: add None at start, remove last (sampling token)
            req.input_token_logprobs_val = [None] + input_token_logprobs[:-1]

        # Process logprob indices based on scoring type
        if is_multi_item_scoring:
            # MIS scores come from input_token_ids_logprobs, not input_token_logprobs.
            # But the shared pipeline requires input_token_logprobs_idx to be the same
            # length as input_token_logprobs_val (validated at line 816). We fill with
            # MIS_DELIMITER_TOKEN_ID as a dummy — score_request() ignores this field.
            delimiter_count = len(req.multi_item_delimiter_indices)
            input_token_logprobs_idx = [MIS_DELIMITER_TOKEN_ID] * delimiter_count
        else:
            # Regular request: include all tokens from logprob_start_len onwards
            input_token_logprobs_idx = req.origin_input_ids[req.logprob_start_len :]

        # Clip padded hash values from image tokens to prevent detokenization errors
        req.input_token_logprobs_idx = [
            x if x < self.model_config.vocab_size - 1 else 0
            for x in input_token_logprobs_idx
        ]
```
**EN:** This block implements the method `_process_input_token_logprobs(req, input_token_logprobs)` on `SchedulerLogprobResultProcessor`. It focuses on Process input token logprobs values and indices., so the class can advance the logprob result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerLogprobResultProcessor` 上的方法 `_process_input_token_logprobs(req, input_token_logprobs)`。它围绕 `_process_input_token_logprobs` 所承担的 logprob result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 57-84: Implement process input top logprobs / 实现process input top logprobs
```python
    def _process_input_top_logprobs(self, req: Req) -> None:
        """Process input top logprobs."""
        if req.top_logprobs_num <= 0:
            return

        is_multi_item_scoring = self._is_multi_item_scoring(req)

        # Initialize arrays - multi-item scoring starts empty, others start with None
        req.input_top_logprobs_val = [] if is_multi_item_scoring else [None]
        req.input_top_logprobs_idx = [] if is_multi_item_scoring else [None]

        # Extend arrays with temp values
        for val, idx in zip(
            req.temp_input_top_logprobs_val,
            req.temp_input_top_logprobs_idx,
            strict=True,
        ):
            req.input_top_logprobs_val.extend(val)
            req.input_top_logprobs_idx.extend(idx)

        # Remove last token (sampling token) for non multi-item scoring requests
        if not is_multi_item_scoring:
            req.input_top_logprobs_val.pop()
            req.input_top_logprobs_idx.pop()

        # Clean up temp storage
        req.temp_input_top_logprobs_idx = None
        req.temp_input_top_logprobs_val = None
```
**EN:** This block implements the method `_process_input_top_logprobs(req)` on `SchedulerLogprobResultProcessor`. It focuses on Process input top logprobs., so the class can advance the logprob result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerLogprobResultProcessor` 上的方法 `_process_input_top_logprobs(req)`。它围绕 `_process_input_top_logprobs` 所承担的 logprob result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 86-116: Implement process input token ids logprobs / 实现process input Token ids logprobs
```python
    def _process_input_token_ids_logprobs(self, req: Req) -> None:
        """Process input token IDs logprobs."""
        if req.token_ids_logprob is None:
            return

        is_multi_item_scoring = self._is_multi_item_scoring(req)

        # Initialize arrays - multi-item scoring starts empty, others start with None
        req.input_token_ids_logprobs_val = [] if is_multi_item_scoring else [None]
        req.input_token_ids_logprobs_idx = [] if is_multi_item_scoring else [None]

        # Process temp values - convert tensors to lists and extend arrays
        for val, idx in zip(
            req.temp_input_token_ids_logprobs_val,
            req.temp_input_token_ids_logprobs_idx,
            strict=True,
        ):
            val_list = val.tolist() if isinstance(val, torch.Tensor) else val
            req.input_token_ids_logprobs_val.extend(
                val_list if isinstance(val_list, list) else [val_list]
            )
            req.input_token_ids_logprobs_idx.extend(idx)

        # Remove last token (sampling token) for non multi-item scoring requests
        if not is_multi_item_scoring:
            req.input_token_ids_logprobs_val.pop()
            req.input_token_ids_logprobs_idx.pop()

        # Clean up temp storage
        req.temp_input_token_ids_logprobs_idx = None
        req.temp_input_token_ids_logprobs_val = None
```
**EN:** This block implements the method `_process_input_token_ids_logprobs(req)` on `SchedulerLogprobResultProcessor`. It focuses on Process input token IDs logprobs., so the class can advance the logprob result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerLogprobResultProcessor` 上的方法 `_process_input_token_ids_logprobs(req)`。它围绕 `_process_input_token_ids_logprobs` 所承担的 logprob result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 118-129: Implement calculate relevant tokens len / 实现calculate relevant tokens len
```python
    def _calculate_relevant_tokens_len(self, req: Req) -> int:
        """Calculate the expected length of logprob arrays based on whether multi-item scoring is enabled.

        For multi-item scoring, only delimiter positions have logprobs.
        For regular requests, all positions from logprob_start_len onwards have logprobs.
        """
        is_multi_item_scoring = self._is_multi_item_scoring(req)

        if is_multi_item_scoring:
            return len(req.multi_item_delimiter_indices)
        else:
            return len(req.origin_input_ids[req.logprob_start_len :])
```
**EN:** This block implements the method `_calculate_relevant_tokens_len(req)` on `SchedulerLogprobResultProcessor`. It focuses on Calculate the expected length of logprob arrays based on whether multi-item scoring is enabled., so the class can advance the logprob result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerLogprobResultProcessor` 上的方法 `_calculate_relevant_tokens_len(req)`。它围绕 `_calculate_relevant_tokens_len` 所承担的 logprob result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 131-153: Implement calculate num input logprobs / 实现calculate num input logprobs
```python
    def calculate_num_input_logprobs(
        self,
        req: Req,
        extend_input_len: int,
        extend_logprob_start_len: int,
    ) -> int:
        """Calculate the number of input logprobs based on whether multi-item scoring is enabled.

        For multi-item scoring, only delimiter positions have logprobs.
        For regular requests, all positions in the range have logprobs.
        """
        is_multi_item_scoring = self._is_multi_item_scoring(req)

        if is_multi_item_scoring:
            # Count pre-computed delimiter indices within the extend range
            return sum(
                1
                for idx in req.multi_item_delimiter_indices
                if extend_logprob_start_len <= idx < extend_input_len
            )
        else:
            # Regular request: all tokens in the range
            return extend_input_len - extend_logprob_start_len
```
**EN:** This block implements the method `calculate_num_input_logprobs(req, extend_input_len, extend_logprob_start_len)` on `SchedulerLogprobResultProcessor`. It focuses on Calculate the number of input logprobs based on whether multi-item scoring is enabled., so the class can advance the logprob result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerLogprobResultProcessor` 上的方法 `calculate_num_input_logprobs(req, extend_input_len, extend_logprob_start_len)`。它围绕 `calculate_num_input_logprobs` 所承担的 logprob result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 155-166: Implement is multi item scoring / 实现is multi item scoring
```python
    def _is_multi_item_scoring(self, req: Req) -> bool:
        """Check if request uses multi-item scoring.

        Multi-item scoring applies to prefill-only requests when a delimiter
        token is configured. In this mode, only positions containing the
        delimiter token receive logprobs.
        """
        return (
            self.server_args.enable_mis
            and req.is_prefill_only
            and req.multi_item_delimiter_indices is not None
        )
```
**EN:** This block implements the method `_is_multi_item_scoring(req)` on `SchedulerLogprobResultProcessor`. It focuses on Check if request uses multi-item scoring., so the class can advance the logprob result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerLogprobResultProcessor` 上的方法 `_is_multi_item_scoring(req)`。它围绕 `_is_multi_item_scoring` 所承担的 logprob result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 168-203: Implement add input logprob return values / 实现add input logprob return values
```python
    def add_input_logprob_return_values(
        self,
        i: int,
        req: Req,
        output: LogitsProcessorOutput,
        logprob_pt: int,
        num_input_logprobs: int,
        last_prefill_chunk: bool,  # If True, it means prefill is finished.
    ):
        """Incrementally add input logprobs to `req`.

        Args:
            i: The request index in a batch.
            req: The request. Input logprobs inside req are modified as a
                consequence of the API
            fill_ids: The prefill ids processed.
            output: Logit processor output that's used to compute input logprobs
            last_prefill_chunk: True if it is the last prefill (when chunked).
                Some of input logprob operation should only happen at the last
                prefill (e.g., computing input token logprobs).
        """
        assert output.input_token_logprobs is not None
        if req.input_token_logprobs is None:
            req.input_token_logprobs = []
        if req.temp_input_top_logprobs_val is None:
            req.temp_input_top_logprobs_val = []
        if req.temp_input_top_logprobs_idx is None:
            req.temp_input_top_logprobs_idx = []
        if req.temp_input_token_ids_logprobs_val is None:
            req.temp_input_token_ids_logprobs_val = []
        if req.temp_input_token_ids_logprobs_idx is None:
            req.temp_input_token_ids_logprobs_idx = []

        if req.input_token_logprobs_val is not None:
            # The input logprob has been already computed. It only happens
            # upon retract.
```
**EN:** This block implements the method `add_input_logprob_return_values(i, req, output, logprob_pt, num_input_logprobs, ...)` on `SchedulerLogprobResultProcessor`. It focuses on Incrementally add input logprobs to `req`., so the class can advance the logprob result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerLogprobResultProcessor` 上的方法 `add_input_logprob_return_values(i, req, output, logprob_pt, num_input_logprobs, ...)`。它围绕 `add_input_logprob_return_values` 所承担的 logprob result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 204-239: Continue add input logprob return values / 继续说明add input logprob return values
```python
            if req.top_logprobs_num > 0:
                assert req.input_token_logprobs_val is not None
            return

        # Important for the performance.
        assert isinstance(output.input_token_logprobs, tuple)
        input_token_logprobs: Tuple[int] = output.input_token_logprobs
        input_token_logprobs = input_token_logprobs[
            logprob_pt : logprob_pt + num_input_logprobs
        ]
        req.input_token_logprobs.extend(input_token_logprobs)

        if req.top_logprobs_num > 0:
            req.temp_input_top_logprobs_val.append(output.input_top_logprobs_val[i])
            req.temp_input_top_logprobs_idx.append(output.input_top_logprobs_idx[i])

        if req.token_ids_logprob is not None:
            req.temp_input_token_ids_logprobs_val.append(
                output.input_token_ids_logprobs_val[i]
            )
            req.temp_input_token_ids_logprobs_idx.append(
                output.input_token_ids_logprobs_idx[i]
            )

        if last_prefill_chunk:
            input_token_logprobs = req.input_token_logprobs
            req.input_token_logprobs = None
            assert req.input_token_logprobs_val is None
            assert req.input_token_logprobs_idx is None
            assert req.input_top_logprobs_val is None
            assert req.input_top_logprobs_idx is None

            # Process all input logprob types using helper functions
            self._process_input_token_logprobs(req, input_token_logprobs)
            self._process_input_top_logprobs(req)
```
**EN:** This block implements the method `add_input_logprob_return_values(i, req, output, logprob_pt, num_input_logprobs, ...)` on `SchedulerLogprobResultProcessor`. It focuses on Incrementally add input logprobs to `req`., so the class can advance the logprob result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerLogprobResultProcessor` 上的方法 `add_input_logprob_return_values(i, req, output, logprob_pt, num_input_logprobs, ...)`。它围绕 `add_input_logprob_return_values` 所承担的 logprob result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 240-251: Continue add input logprob return values / 继续说明add input logprob return values
```python
            self._process_input_token_ids_logprobs(req)

            if req.return_logprob:
                relevant_tokens_len = self._calculate_relevant_tokens_len(req)
                assert len(req.input_token_logprobs_val) == relevant_tokens_len
                assert len(req.input_token_logprobs_idx) == relevant_tokens_len
                if req.top_logprobs_num > 0:
                    assert len(req.input_top_logprobs_val) == relevant_tokens_len
                    assert len(req.input_top_logprobs_idx) == relevant_tokens_len
                if req.token_ids_logprob is not None:
                    assert len(req.input_token_ids_logprobs_val) == relevant_tokens_len
                    assert len(req.input_token_ids_logprobs_idx) == relevant_tokens_len
```
**EN:** This block implements the method `add_input_logprob_return_values(i, req, output, logprob_pt, num_input_logprobs, ...)` on `SchedulerLogprobResultProcessor`. It focuses on Incrementally add input logprobs to `req`., so the class can advance the logprob result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerLogprobResultProcessor` 上的方法 `add_input_logprob_return_values(i, req, output, logprob_pt, num_input_logprobs, ...)`。它围绕 `add_input_logprob_return_values` 所承担的 logprob result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 253-299: Implement add logprob return values / 实现add logprob return values
```python
    def add_logprob_return_values(
        self,
        i: int,
        req: Req,
        pt: int,
        next_token_ids: List[int],
        num_input_logprobs: int,
        output: LogitsProcessorOutput,
    ):
        """Attach logprobs to the return values."""
        if output.next_token_logprobs is not None:
            req.output_token_logprobs_val.append(output.next_token_logprobs[i])
            req.output_token_logprobs_idx.append(next_token_ids[i])

        # Only add input logprobs if there are input tokens to process
        # Note: For prefill-only requests with default logprob_start_len, this will be 0,
        # meaning we only compute output logprobs (which is the intended behavior)
        if num_input_logprobs > 0:
            self.add_input_logprob_return_values(
                i,
                req,
                output,
                pt,
                num_input_logprobs,
                last_prefill_chunk=True,
            )
        else:
            self._initialize_empty_logprob_containers(req)

        if req.top_logprobs_num > 0:
            req.output_top_logprobs_val.append(output.next_token_top_logprobs_val[i])
            req.output_top_logprobs_idx.append(output.next_token_top_logprobs_idx[i])

        if (
            req.token_ids_logprob is not None
            and output.next_token_token_ids_logprobs_val is not None
        ):
            # Convert GPU tensor to list if needed
            logprobs_val = output.next_token_token_ids_logprobs_val[i]
            if isinstance(logprobs_val, torch.Tensor):
                logprobs_val = logprobs_val.tolist()
            req.output_token_ids_logprobs_val.append(logprobs_val)
            req.output_token_ids_logprobs_idx.append(
                output.next_token_token_ids_logprobs_idx[i]
            )

        return num_input_logprobs
```
**EN:** This block implements the method `add_logprob_return_values(i, req, pt, next_token_ids, num_input_logprobs, ...)` on `SchedulerLogprobResultProcessor`. It focuses on Attach logprobs to the return values., so the class can advance the logprob result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerLogprobResultProcessor` 上的方法 `add_logprob_return_values(i, req, pt, next_token_ids, num_input_logprobs, ...)`。它围绕 `add_logprob_return_values` 所承担的 logprob result 处理器 相关职责展开，使该类能够独立推进相应流程。

### Lines 301-319: Implement initialize empty logprob containers / 实现initialize empty logprob containers
```python
    def _initialize_empty_logprob_containers(self, req: Req) -> None:
        """
        Initialize logprob fields to empty lists if unset.

        This is needed for prefill-only requests where the normal initialization
        flow might be bypassed, but downstream code expects these fields to be lists.
        """
        if req.input_token_logprobs_val is None:
            req.input_token_logprobs_val = []
        if req.input_token_logprobs_idx is None:
            req.input_token_logprobs_idx = []
        if req.input_top_logprobs_val is None:
            req.input_top_logprobs_val = []
        if req.input_top_logprobs_idx is None:
            req.input_top_logprobs_idx = []
        if req.input_token_ids_logprobs_val is None:
            req.input_token_ids_logprobs_val = []
        if req.input_token_ids_logprobs_idx is None:
            req.input_token_ids_logprobs_idx = []
```
**EN:** This block implements the method `_initialize_empty_logprob_containers(req)` on `SchedulerLogprobResultProcessor`. It focuses on Initialize logprob fields to empty lists if unset., so the class can advance the logprob result processor workflow in a self-contained way.
**CN:** 该代码块实现 `SchedulerLogprobResultProcessor` 上的方法 `_initialize_empty_logprob_containers(req)`。它围绕 `_initialize_empty_logprob_containers` 所承担的 logprob result 处理器 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: SchedulerLogprobResultProcessor
- **Domain focus / 领域焦点**: logprob result processor / logprob result 处理器
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: dataclasses, typing
- **Third-party / 第三方库**: __future__, torch
- **Local Modules / 本地模块**: sglang.srt.configs.model_config, sglang.srt.layers.logits_processor, sglang.srt.managers.schedule_batch, sglang.srt.server_args
