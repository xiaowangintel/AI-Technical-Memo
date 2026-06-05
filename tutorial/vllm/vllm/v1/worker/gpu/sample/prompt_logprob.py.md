# prompt_logprob.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/sample/prompt_logprob.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `PromptLogprobsWorker`, `_prompt_logprobs_token_ids_kernel`, `get_prompt_logprobs_token_ids` for the V1 `worker/gpu/sample` subsystem. / 为 V1 的 `worker/gpu/sample` 子系统实现 `PromptLogprobsWorker`, `_prompt_logprobs_token_ids_kernel`, `get_prompt_logprobs_token_ids`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from collections.abc import Callable

import numpy as np
import torch

from vllm.sampling_params import SamplingParams
from vllm.triton_utils import tl, triton
from vllm.v1.outputs import LogprobsTensors
from vllm.v1.worker.gpu.input_batch import InputBatch
from vllm.v1.worker.gpu.sample.logprob import compute_topk_logprobs
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`, and internal vLLM modules such as `vllm.sampling_params`, `vllm.triton_utils`, `vllm.v1.outputs`, `vllm.v1.worker.gpu.input_batch`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch` 等外部依赖，以及 `vllm.sampling_params`, `vllm.triton_utils`, `vllm.v1.outputs`, `vllm.v1.worker.gpu.input_batch` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `PromptLogprobsWorker` class / `PromptLogprobsWorker` 类
```python
class PromptLogprobsWorker:
```
**EN:** Introduces the `PromptLogprobsWorker` class. Core methods include `__init__`, `add_request`, `remove_request`, `compute_prompt_logprobs`.
**CN:** 这里定义 `PromptLogprobsWorker` 类。核心方法包括 `__init__`, `add_request`, `remove_request`, `compute_prompt_logprobs`。

### `PromptLogprobsWorker.__init__` method / `PromptLogprobsWorker.__init__` 方法
```python
    def __init__(self, max_num_reqs: int):
        self.max_num_reqs = max_num_reqs

        self.uses_prompt_logprobs = np.zeros(self.max_num_reqs, dtype=bool)
        self.num_prompt_logprobs = np.zeros(self.max_num_reqs, dtype=np.int32)
        # req_idx -> list of in-progress LogprobsTensors
        self.in_progress_prompt_logprobs: dict[str, list[LogprobsTensors]] = {}
```
**EN:** This method initializes the object state within `PromptLogprobsWorker`. Key calls include `zeros`. It touches state such as `max_num_reqs`, `uses_prompt_logprobs`, `num_prompt_logprobs`, `in_progress_prompt_logprobs`.
**CN:** 该方法会初始化对象状态，其作用域位于`PromptLogprobsWorker`。 关键调用包括 `zeros`。 它会读写 `max_num_reqs`, `uses_prompt_logprobs`, `num_prompt_logprobs`, `in_progress_prompt_logprobs` 等状态。

### `PromptLogprobsWorker.add_request` method / `PromptLogprobsWorker.add_request` 方法
```python
    def add_request(self, req_id: str, req_idx: int, sampling_params: SamplingParams):
        uses_prompt_logprobs = sampling_params.prompt_logprobs is not None
        self.uses_prompt_logprobs[req_idx] = uses_prompt_logprobs
        self.num_prompt_logprobs[req_idx] = sampling_params.prompt_logprobs or 0
        if uses_prompt_logprobs:
            self.in_progress_prompt_logprobs[req_id] = []
```
**EN:** This method implements `add_request` within `PromptLogprobsWorker`. It touches state such as `uses_prompt_logprobs`, `num_prompt_logprobs`, `in_progress_prompt_logprobs`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `add_request`，其作用域位于`PromptLogprobsWorker`。 它会读写 `uses_prompt_logprobs`, `num_prompt_logprobs`, `in_progress_prompt_logprobs` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `PromptLogprobsWorker.remove_request` method / `PromptLogprobsWorker.remove_request` 方法
```python
    def remove_request(self, req_id: str) -> None:
        self.in_progress_prompt_logprobs.pop(req_id, None)
```
**EN:** This method implements `remove_request` within `PromptLogprobsWorker`. Key calls include `pop`.
**CN:** 该方法会实现 `remove_request`，其作用域位于`PromptLogprobsWorker`。 关键调用包括 `pop`。

### `PromptLogprobsWorker.compute_prompt_logprobs` method / `PromptLogprobsWorker.compute_prompt_logprobs` 方法
```python
    def compute_prompt_logprobs(
        self,
        logits_fn: Callable[[torch.Tensor], torch.Tensor],
        hidden_states: torch.Tensor,
        input_batch: InputBatch,
        # [max_num_reqs, max_model_len]
        all_token_ids: torch.Tensor,
        # [max_num_reqs]
        num_computed_tokens: torch.Tensor,
        # [max_num_reqs]
        prompt_lens: np.ndarray,
        # [max_num_reqs]
        prefill_lens: np.ndarray,
        # [max_num_reqs]
        num_computed_prefill_tokens: np.ndarray,
    ) -> dict[str, LogprobsTensors]:
        idx_mapping_np = input_batch.idx_mapping_np
        needs_prompt_logprobs = self.uses_prompt_logprobs[idx_mapping_np]
        if not np.any(needs_prompt_logprobs):
            # Common case: No request asks for prompt logprobs.
            return {}

        num_prompt_logprobs = self.num_prompt_logprobs[idx_mapping_np]
        prompt_lens = prompt_lens[idx_mapping_np]
        computed_prefill = num_computed_prefill_tokens[idx_mapping_np]
        includes_prompt = computed_prefill < prompt_lens
        # NOTE(woosuk): If the request was resumed after preemption, its prompt
        # logprobs must have been computed before preemption. Skip.
        resumed_after_prompt = prompt_lens < prefill_lens[idx_mapping_np]
        needs_prompt_logprobs &= includes_prompt & ~resumed_after_prompt
        if not np.any(needs_prompt_logprobs):
            return {}

        # get the maximum number in this batch
        requested_num_prompt_logprobs = num_prompt_logprobs[needs_prompt_logprobs]
        max_num_prompt_logprobs = (
            -1
            if np.any(requested_num_prompt_logprobs == -1)
            else int(requested_num_prompt_logprobs.max())
        )

        # Get the prompt logprobs token_ids.
        prompt_logprobs_token_ids = get_prompt_logprobs_token_ids(
            input_batch.num_tokens,
            input_batch.query_start_loc,
            input_batch.idx_mapping,
            num_computed_tokens,
            all_token_ids,
        )
        prompt_token_ids, prompt_logprobs, prompt_ranks = (
    # ... omitted for brevity ...
            prompt_logprobs_list = self.in_progress_prompt_logprobs[req_id]
            if logprobs is not None and (req_is_prompt_chunked or prompt_logprobs_list):
                prompt_logprobs_list.append(logprobs)
            if req_is_prompt_chunked:
                # Prompt is chunked. Do not return the logprobs yet.
                continue

            if prompt_logprobs_list:
                # Merge the in-progress logprobs.
                logprobs = LogprobsTensors(
                    logprob_token_ids=torch.cat(
                        [x.logprob_token_ids for x in prompt_logprobs_list]
                    ),
                    logprobs=torch.cat([x.logprobs for x in prompt_logprobs_list]),
                    selected_token_ranks=torch.cat(
                        [x.selected_token_ranks for x in prompt_logprobs_list]
                    ),
                )
                prompt_logprobs_list.clear()

            if logprobs is None:
                continue

            prompt_logprobs_dict[req_id] = logprobs
        return prompt_logprobs_dict
```
**EN:** This method computes derived values within `PromptLogprobsWorker`. Key calls include `get_prompt_logprobs_token_ids`, `compute_prompt_logprobs_with_chunking`, `enumerate`, `any`, `int`, `max`. The control flow contains 10 branch(es) and 1 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会计算派生值，其作用域位于`PromptLogprobsWorker`。 关键调用包括 `get_prompt_logprobs_token_ids`, `compute_prompt_logprobs_with_chunking`, `enumerate`, `any`, `int`, `max`。 控制流包含 10 个分支和 1 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `_prompt_logprobs_token_ids_kernel` function / `_prompt_logprobs_token_ids_kernel` 函数
```python
@triton.jit
def _prompt_logprobs_token_ids_kernel(
    prompt_logprobs_token_ids_ptr,
    query_start_loc_ptr,
    idx_mapping_ptr,
    num_computed_tokens_ptr,
    all_token_ids_ptr,
    all_token_ids_stride,
    BLOCK_SIZE: tl.constexpr,
):
    batch_idx = tl.program_id(0)
    req_state_idx = tl.load(idx_mapping_ptr + batch_idx)

    query_start = tl.load(query_start_loc_ptr + batch_idx)
    query_end = tl.load(query_start_loc_ptr + batch_idx + 1)
    query_len = query_end - query_start

    num_computed_tokens = tl.load(num_computed_tokens_ptr + req_state_idx)
    for i in range(0, query_len, BLOCK_SIZE):
        block = i + tl.arange(0, BLOCK_SIZE)
        mask = block < query_len
        # NOTE(woosuk): We should shift the pos by one
        # because the logprob is computed for the next token.
        target_pos = num_computed_tokens + 1 + block
        token_ids = tl.load(
            all_token_ids_ptr + req_state_idx * all_token_ids_stride + target_pos,
            mask=mask,
        )
        tl.store(
            prompt_logprobs_token_ids_ptr + query_start + block, token_ids, mask=mask
        )
```
**EN:** This function implements `_prompt_logprobs_token_ids_kernel` within the module. Key calls include `program_id`, `load`, `range`, `store`, `arange`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_prompt_logprobs_token_ids_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `range`, `store`, `arange`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `get_prompt_logprobs_token_ids` function / `get_prompt_logprobs_token_ids` 函数
```python
def get_prompt_logprobs_token_ids(
    num_tokens: int,
    query_start_loc: torch.Tensor,
    idx_mapping: torch.Tensor,
    num_computed_tokens: torch.Tensor,
    all_token_ids: torch.Tensor,
) -> torch.Tensor:
    token_ids = torch.empty(num_tokens, dtype=torch.int64, device=idx_mapping.device)
    num_reqs = idx_mapping.shape[0]
    _prompt_logprobs_token_ids_kernel[(num_reqs,)](
        token_ids,
        query_start_loc,
        idx_mapping,
        num_computed_tokens,
        all_token_ids,
        all_token_ids.stride(0),
        BLOCK_SIZE=1024,
    )
    return token_ids
```
**EN:** This function returns or derives a value within the module. Key calls include `empty`, `stride`.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `empty`, `stride`。

### `compute_prompt_logprobs_with_chunking` function / `compute_prompt_logprobs_with_chunking` 函数
```python
def compute_prompt_logprobs_with_chunking(
    prompt_token_ids: torch.Tensor,
    prompt_hidden_states: torch.Tensor,
    logits_fn: Callable[[torch.Tensor], torch.Tensor],
    num_prompt_logprobs: int,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    # Since materializing the full prompt logits can take too much memory,
    # we compute it in chunks.
    CHUNK_SIZE = 1024
    token_ids = []
    logprobs = []
    ranks = []
    prompt_token_ids = prompt_token_ids.to(torch.int64)
    for start_idx in range(0, prompt_token_ids.shape[0], CHUNK_SIZE):
        end_idx = start_idx + CHUNK_SIZE
        # NOTE(woosuk): logits_fn can be slow because it involves all-gather.
        prompt_logits = logits_fn(prompt_hidden_states[start_idx:end_idx])
        requested_num_prompt_logprobs = (
            prompt_logits.shape[-1]
            if num_prompt_logprobs == -1
            else num_prompt_logprobs
        )
        prompt_logprobs = compute_topk_logprobs(
            prompt_logits,
            requested_num_prompt_logprobs,
            prompt_token_ids[start_idx:end_idx],
        )
        token_ids.append(prompt_logprobs.logprob_token_ids)
        logprobs.append(prompt_logprobs.logprobs)
        ranks.append(prompt_logprobs.selected_token_ranks)

    token_ids = torch.cat(token_ids, dim=0) if len(token_ids) > 1 else token_ids[0]
    logprobs = torch.cat(logprobs, dim=0) if len(logprobs) > 1 else logprobs[0]
    ranks = torch.cat(ranks, dim=0) if len(ranks) > 1 else ranks[0]
    return token_ids, logprobs, ranks
```
**EN:** This function computes derived values within the module. Key calls include `to`, `range`, `logits_fn`, `compute_topk_logprobs`, `append`, `cat`. The control flow contains 4 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会计算派生值，其作用域位于the module。 关键调用包括 `to`, `range`, `logits_fn`, `compute_topk_logprobs`, `append`, `cat`。 控制流包含 4 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `PromptLogprobsWorker`: central class or interface in this module. / `PromptLogprobsWorker`：本模块中的核心类或接口。
- `_prompt_logprobs_token_ids_kernel`: top-level helper or orchestration entry point. / `_prompt_logprobs_token_ids_kernel`：顶层辅助函数或编排入口。
- `get_prompt_logprobs_token_ids`: top-level helper or orchestration entry point. / `get_prompt_logprobs_token_ids`：顶层辅助函数或编排入口。
- `compute_prompt_logprobs_with_chunking`: top-level helper or orchestration entry point. / `compute_prompt_logprobs_with_chunking`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `collections`
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.sampling_params`, `vllm.triton_utils`, `vllm.v1.outputs`, `vllm.v1.worker.gpu.input_batch`, `vllm.v1.worker.gpu.sample.logprob`
