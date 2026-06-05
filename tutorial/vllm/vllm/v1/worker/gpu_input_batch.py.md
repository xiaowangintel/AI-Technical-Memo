# gpu_input_batch.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu_input_batch.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `CachedRequestState`, `InputBatch` for the V1 `worker` subsystem. / 为 V1 的 `worker` 子系统实现 `CachedRequestState`, `InputBatch`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from dataclasses import dataclass
from typing import cast

import numpy as np
import torch

from vllm.config.reasoning import ReasoningConfig
from vllm.lora.request import LoRARequest
from vllm.multimodal.inputs import MultiModalFeatureSpec
from vllm.pooling_params import PoolingParams
from vllm.sampling_params import SamplingParams, SamplingType
from vllm.utils import length_from_prompt_token_ids_or_embeds
from vllm.utils.collection_utils import swap_dict_values
from vllm.v1.outputs import LogprobsTensors
from vllm.v1.pool.metadata import PoolingMetadata, PoolingStates
from vllm.v1.sample.logits_processor import (
    BatchUpdateBuilder,
    LogitsProcessors,
    MoveDirectionality,
)
from vllm.v1.sample.metadata import SamplingMetadata
from vllm.v1.sample.thinking_budget_state import (
    maybe_create_thinking_budget_state_holder,
)
from vllm.v1.utils import copy_slice
from vllm.v1.worker.block_table import MultiGroupBlockTable
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`, and internal vLLM modules such as `vllm.config.reasoning`, `vllm.lora.request`, `vllm.multimodal.inputs`, `vllm.pooling_params`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch` 等外部依赖，以及 `vllm.config.reasoning`, `vllm.lora.request`, `vllm.multimodal.inputs`, `vllm.pooling_params` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `CachedRequestState` class / `CachedRequestState` 类
```python
@dataclass
class CachedRequestState:
    req_id: str
    prompt_token_ids: list[int] | None
    mm_features: list[MultiModalFeatureSpec]
    sampling_params: SamplingParams | None
    generator: torch.Generator | None

    block_ids: tuple[list[int], ...]
    num_computed_tokens: int
    output_token_ids: list[int]

    mrope_positions: torch.Tensor | None = None
    mrope_position_delta: int | None = None

    xdrope_positions: torch.Tensor | None = None

    lora_request: LoRARequest | None = None
    prompt_embeds: torch.Tensor | None = None
    # To accumulate prompt logprobs tensor chunks across prefill steps.
    in_progress_prompt_logprobs_cpu: LogprobsTensors | None = None

    # Per-position mask for mixed-mode inputs (e.g chat completion with
    # prompt_embeds content parts). See `Request.prompt_is_token_ids`.
    prompt_is_token_ids: list[bool] | None = None

    # Used when both async_scheduling and spec_decode are enabled.
    prev_num_draft_len: int = 0

    # for pooling models
    pooling_params: PoolingParams | None = None
    pooling_states: PoolingStates | None = None
```
**EN:** Uses `@dataclass` to package related state for `CachedRequestState`. Typical fields include `req_id`, `prompt_token_ids`, `mm_features`, `sampling_params`, `generator`, `block_ids`.
**CN:** `CachedRequestState` 使用 `@dataclass` 打包相关状态。典型字段包括 `req_id`, `prompt_token_ids`, `mm_features`, `sampling_params`, `generator`, `block_ids`。

### `CachedRequestState.__post_init__` method / `CachedRequestState.__post_init__` 方法
```python
    def __post_init__(self):
        self.num_prompt_tokens = length_from_prompt_token_ids_or_embeds(
            self.prompt_token_ids, self.prompt_embeds
        )

        if self.pooling_params is not None:
            self.pooling_states = PoolingStates()
```
**EN:** This method implements `__post_init__` within `CachedRequestState`. Key calls include `length_from_prompt_token_ids_or_embeds`, `PoolingStates`. It touches state such as `num_prompt_tokens`, `pooling_states`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `__post_init__`，其作用域位于`CachedRequestState`。 关键调用包括 `length_from_prompt_token_ids_or_embeds`, `PoolingStates`。 它会读写 `num_prompt_tokens`, `pooling_states` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `CachedRequestState.num_tokens` method / `CachedRequestState.num_tokens` 方法
```python
    @property
    def num_tokens(self) -> int:
        return self.num_prompt_tokens + len(self.output_token_ids)
```
**EN:** This method implements `num_tokens` within `CachedRequestState`. Key calls include `len`.
**CN:** 该方法会实现 `num_tokens`，其作用域位于`CachedRequestState`。 关键调用包括 `len`。

### `CachedRequestState.get_token_id` method / `CachedRequestState.get_token_id` 方法
```python
    def get_token_id(self, idx: int) -> int:
        if idx < self.num_prompt_tokens:
            if self.prompt_token_ids is None:
                raise ValueError(
                    f"Tried to access token index {idx}, but that token was "
                    "provided via prompt_embeds, and its ID is unknown."
                )
            return self.prompt_token_ids[idx]
        if idx - self.num_prompt_tokens < len(self.output_token_ids):
            return self.output_token_ids[idx - self.num_prompt_tokens]
        return -1
```
**EN:** This method returns or derives a value within `CachedRequestState`. Key calls include `len`, `ValueError`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`CachedRequestState`。 关键调用包括 `len`, `ValueError`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `InputBatch` class / `InputBatch` 类
```python
class InputBatch:
```
**EN:** Introduces the `InputBatch` class. Core methods include `__init__`, `req_ids`, `_register_add_request`, `add_request`, `update_req_spec_token_ids`, `remove_request`.
**CN:** 这里定义 `InputBatch` 类。核心方法包括 `__init__`, `req_ids`, `_register_add_request`, `add_request`, `update_req_spec_token_ids`, `remove_request`。

### `InputBatch.__init__` method / `InputBatch.__init__` 方法
```python
    def __init__(
        self,
        max_num_reqs: int,
        max_model_len: int,
        max_num_batched_tokens: int,
        device: torch.device,
        pin_memory: bool,
        vocab_size: int,
        block_sizes: list[int],  # The block_size of each kv cache group
        kernel_block_sizes: list[int],
        max_num_blocks_per_req: list[int] | None = None,
        logitsprocs: LogitsProcessors | None = None,
        logitsprocs_need_output_token_ids: bool = False,
        num_spec_tokens: int = 0,
        is_pooling_model: bool = False,
        cp_kv_cache_interleave_size: int = 1,
        reasoning_config: ReasoningConfig | None = None,
    ):
        self.thinking_budget_state_holder = maybe_create_thinking_budget_state_holder(
            reasoning_config,
            max_num_reqs,
            num_spec_tokens,
            device,
            pin_memory,
        )
        self.thinking_token_budget_reqs: set[str] = set()
        self.is_pooling_model = is_pooling_model
        self.max_num_reqs = max_num_reqs
        self.max_model_len = max_model_len
        self.max_num_batched_tokens = max_num_batched_tokens
        self.device = device
        self.pin_memory = pin_memory
        self.vocab_size = vocab_size

        self._req_ids: list[str | None] = []
        self.req_id_to_index: dict[str, int] = {}

        # TODO(woosuk): This buffer could be too large if max_model_len is big.
        # Find a way to reduce the CPU memory usage.
        # This buffer is not directly transferred to the GPU, so it does not
        # need to be pinned.
        self.token_ids_cpu_tensor = torch.zeros(
            (max_num_reqs, max_model_len),
            device="cpu",
            dtype=torch.int32,
            pin_memory=False,
        )
        self.token_ids_cpu = self.token_ids_cpu_tensor.numpy()
        self.is_token_ids_tensor = torch.zeros(
            (max_num_reqs, max_model_len), device="cpu", dtype=bool, pin_memory=False
    # ... omitted for brevity ...
        self.req_output_token_ids: list[list[int] | None] = []

        # Store provided logitsprocs. If none are provided, initialize empty
        # data structure
        self.logitsprocs = logitsprocs or LogitsProcessors()
        self.logitsprocs_need_output_token_ids = logitsprocs_need_output_token_ids

        # Store last speculative tokens for sampler.
        self.spec_token_ids: list[list[int]] = [[] for _ in range(max_num_reqs)]

        # This is updated each time the batch constituents change.
        self.sampling_metadata = self._make_sampling_metadata()

        # for pooling models
        self.pooling_params: dict[str, PoolingParams] = {}
        self.pooling_states: dict[str, PoolingStates] = {}

        # Cached reference to the GPU tensor of previously sampled tokens
        self.prev_sampled_token_ids: torch.Tensor | None = None
        self.prev_req_id_to_index: dict[str, int] | None = None
        # These are used to update output_token_ids with real sampled
        # ids from prior step, if required by current sampling params
        # (e.g. penalties).
        self.sampled_token_ids_cpu: torch.Tensor | None = None
        self.async_copy_ready_event: torch.Event | None = None
```
**EN:** This method initializes the object state within `InputBatch`. Key calls include `maybe_create_thinking_budget_state_holder`, `set`, `zeros`, `numpy`, `MultiGroupBlockTable`, `empty`. It touches state such as `thinking_budget_state_holder`, `thinking_token_budget_reqs`, `is_pooling_model`, `max_num_reqs`, `max_model_len`, `max_num_batched_tokens`, `device`, `pin_memory`. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会初始化对象状态，其作用域位于`InputBatch`。 关键调用包括 `maybe_create_thinking_budget_state_holder`, `set`, `zeros`, `numpy`, `MultiGroupBlockTable`, `empty`。 它会读写 `thinking_budget_state_holder`, `thinking_token_budget_reqs`, `is_pooling_model`, `max_num_reqs`, `max_model_len`, `max_num_batched_tokens`, `device`, `pin_memory` 等状态。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `InputBatch.add_request` method / `InputBatch.add_request` 方法
```python
    def add_request(
        self,
        request: "CachedRequestState",
    ) -> int:
        req_index = self._register_add_request(request)

        req_id = request.req_id
        if req_index == len(self._req_ids):
            self._req_ids.append(req_id)
            self.req_output_token_ids.append(request.output_token_ids)
            self.spec_token_ids.append([])
        else:
            self._req_ids[req_index] = req_id
            self.req_output_token_ids[req_index] = request.output_token_ids
            self.spec_token_ids[req_index].clear()

        self.req_id_to_index[req_id] = req_index

        # Copy the prompt token ids and output token ids.
        num_prompt_tokens = length_from_prompt_token_ids_or_embeds(
            request.prompt_token_ids, request.prompt_embeds
        )
        self.num_prompt_tokens[req_index] = num_prompt_tokens
        start_idx = num_prompt_tokens
        end_idx = start_idx + len(request.output_token_ids)
        if request.prompt_token_ids is not None:
            self.token_ids_cpu[req_index, :num_prompt_tokens] = request.prompt_token_ids
            if request.prompt_is_token_ids is not None:
                self.is_token_ids[req_index, :num_prompt_tokens] = (
                    request.prompt_is_token_ids
                )
            else:
                self.is_token_ids[req_index, :num_prompt_tokens] = True
        else:
            self.is_token_ids[req_index, :num_prompt_tokens] = False
        if request.prompt_embeds is not None:
            self.req_prompt_embeds[req_index] = request.prompt_embeds
        self.token_ids_cpu[req_index, start_idx:end_idx] = request.output_token_ids
        self.is_token_ids[req_index, start_idx:end_idx] = True
        # Number of tokens without spec decode tokens.
        self.num_tokens_no_spec[req_index] = request.num_tokens

        self.num_computed_tokens_cpu[req_index] = request.num_computed_tokens
        self.block_table.add_row(request.block_ids, req_index)

        if sampling_params := request.sampling_params:
            if sampling_params.sampling_type == SamplingType.GREEDY:
                # Should avoid division by zero later when apply_temperature.
                self.temperature_cpu[req_index] = 0.0
                self.greedy_reqs.add(req_id)
    # ... omitted for brevity ...
            self.pooling_params[req_id] = pooling_params
            self.pooling_states[req_id] = pooling_states
            self.logits_processing_needs_token_ids[req_index] = (
                pooling_params.requires_token_ids
            )
        else:
            raise NotImplementedError("Unrecognized request type")

        # Speculative decoding: by default 1 token is generated.
        self.num_accepted_tokens_cpu[req_index] = 1

        # Add request lora ID
        if request.lora_request:
            lora_id = request.lora_request.lora_int_id
            if lora_id not in self.lora_id_to_request_ids:
                self.lora_id_to_request_ids[lora_id] = set()

            self.request_lora_mapping[req_index] = lora_id
            self.lora_id_to_request_ids[lora_id].add(request.req_id)
            self.lora_id_to_lora_request[lora_id] = request.lora_request
        else:
            # No LoRA
            self.request_lora_mapping[req_index] = 0

        return req_index
```
**EN:** This method implements `add_request` within `InputBatch`. Key calls include `_register_add_request`, `length_from_prompt_token_ids_or_embeds`, `add_row`, `len`, `append`, `clear`. It touches state such as `req_id_to_index`, `num_prompt_tokens`, `token_ids_cpu`, `is_token_ids`, `num_tokens_no_spec`, `num_computed_tokens_cpu`, `num_accepted_tokens_cpu`, `_req_ids`. The control flow contains 21 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `add_request`，其作用域位于`InputBatch`。 关键调用包括 `_register_add_request`, `length_from_prompt_token_ids_or_embeds`, `add_row`, `len`, `append`, `clear`。 它会读写 `req_id_to_index`, `num_prompt_tokens`, `token_ids_cpu`, `is_token_ids`, `num_tokens_no_spec`, `num_computed_tokens_cpu`, `num_accepted_tokens_cpu`, `_req_ids` 等状态。 控制流包含 21 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `InputBatch.condense` method / `InputBatch.condense` 方法
```python
    def condense(self) -> None:
        """Slide non-empty requests down into lower, empty indices.

        Any consecutive empty indices at the very end of the list are not
        filled.

        Returns:
          swaps: list of (from,to) swap tuples for moved requests
          empty_req_indices: indices not filled by condensation
        """
        num_reqs = self.num_reqs

        if not (empty_req_indices := self.batch_update_builder.removed):
            # All removed requests were replaced by added requests, or else no
            # requests were removed at all. No condense() needed
            return
        if num_reqs == 0:
            # The batched states are empty.
            self._req_ids.clear()
            self.req_output_token_ids.clear()
            self.spec_token_ids.clear()
            return

        # NOTE(woosuk): This function assumes that the empty_req_indices
        # is sorted in descending order.
        last_req_index = num_reqs + len(empty_req_indices) - 1
        while empty_req_indices:
            # Find the largest non-empty index.
            while last_req_index in empty_req_indices:
                last_req_index -= 1

            # Find the smallest empty index.
            empty_index = self.batch_update_builder.peek_removed()
            assert empty_index is not None
            if empty_index >= last_req_index:
                break

            # Move active request down into empty request
            # index.
            self.batch_update_builder.pop_removed()
            req_id = self._req_ids[last_req_index]
            output_token_ids = self.req_output_token_ids[last_req_index]
            assert req_id is not None
            self._req_ids[empty_index] = req_id
            self._req_ids[last_req_index] = None
            self.req_output_token_ids[empty_index] = output_token_ids
            self.req_output_token_ids[last_req_index] = None
            self.req_id_to_index[req_id] = empty_index

            num_tokens = self._get_active_token_count(last_req_index)
    # ... omitted for brevity ...
            ]
            self.num_accepted_tokens_cpu[empty_index] = self.num_accepted_tokens_cpu[
                last_req_index
            ]
            generator = self.generators.pop(last_req_index, None)
            if generator is not None:
                self.generators[empty_index] = generator

            # TODO convert these to LogitsProcessors
            if self.allowed_token_ids_mask_cpu_tensor is not None:
                self.allowed_token_ids_mask_cpu_tensor[empty_index] = (
                    self.allowed_token_ids_mask_cpu_tensor[last_req_index]
                )

            bad_words_token_ids = self.bad_words_token_ids.pop(last_req_index, None)
            if bad_words_token_ids is not None:
                self.bad_words_token_ids[empty_index] = bad_words_token_ids

            # Decrement last_req_index since it is now empty.
            last_req_index -= 1

        # Trim lists to the batch size.
        del self._req_ids[num_reqs:]
        del self.req_output_token_ids[num_reqs:]
        del self.spec_token_ids[num_reqs:]
```
**EN:** This method implements `condense` within `InputBatch`. The docstring frames it as: Slide non-empty requests down into lower, empty indices. Key calls include `clear`, `peek_removed`, `pop_removed`, `_get_active_token_count`, `move_row`, `append`. It touches state such as `_req_ids`, `req_output_token_ids`, `req_id_to_index`, `spec_token_ids`, `token_ids_cpu`, `is_token_ids`, `num_tokens_no_spec`, `num_prompt_tokens`. The control flow contains 8 branch(es) and 2 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `condense`，其作用域位于`InputBatch`。 关键调用包括 `clear`, `peek_removed`, `pop_removed`, `_get_active_token_count`, `move_row`, `append`。 它会读写 `_req_ids`, `req_output_token_ids`, `req_id_to_index`, `spec_token_ids`, `token_ids_cpu`, `is_token_ids`, `num_tokens_no_spec`, `num_prompt_tokens` 等状态。 控制流包含 8 个分支和 2 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `InputBatch.update_async_output_token_ids` method / `InputBatch.update_async_output_token_ids` 方法
```python
    def update_async_output_token_ids(self) -> None:
        """
        In async scheduling case, update output_token_ids in sampling metadata
        from prior steps sampled token ids once they've finished copying to CPU.
        This is called right before they are needed by the logits processors.
        """
        output_token_ids = self.sampling_metadata.output_token_ids
        if self.sampled_token_ids_cpu is None or not output_token_ids:
            # Output token ids not needed or not async scheduling.
            return

        assert self.prev_req_id_to_index is not None
        sampled_token_ids = None
        for index, req_id in enumerate(self.req_ids):
            prev_index = self.prev_req_id_to_index.get(req_id)
            if prev_index is None:
                continue
            req_output_token_ids = output_token_ids[index]
            if not req_output_token_ids or req_output_token_ids[-1] != -1:
                # Final output id is not a placeholder, some tokens must have
                # been discarded after a kv-load failure.
                continue
            if sampled_token_ids is None:
                assert self.async_copy_ready_event is not None
                self.async_copy_ready_event.synchronize()
                sampled_token_ids = self.sampled_token_ids_cpu.tolist()
            # Replace placeholder token id(s) with actual sampled id(s).
            new_ids: list[int] = sampled_token_ids[prev_index]
            if not new_ids:
                continue
            num_sampled_ids = len(new_ids) if new_ids[-1] != -1 else new_ids.index(-1)
            # Also account for case where there may be a smaller number of
            # output placeholders (tokens can be discarded after kv-load
            # failure) or a larger number (async spec decode adds optimistic
            # placeholders that may exceed the actual acceptance count).
            first_placeholder = req_output_token_ids.index(-1)
            num_placeholders = len(req_output_token_ids) - first_placeholder
            num_to_replace = min(num_sampled_ids, num_placeholders)
            del new_ids[num_to_replace:]
            req_output_token_ids[first_placeholder:] = new_ids
```
**EN:** This method updates existing state within `InputBatch`. The docstring frames it as: In async scheduling case, update output_token_ids in sampling metadata from prior steps sampled token ids once they've finished copying to CPU. Key calls include `enumerate`, `get`, `index`, `min`, `synchronize`, `tolist`. The control flow contains 6 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会更新现有状态，其作用域位于`InputBatch`。 关键调用包括 `enumerate`, `get`, `index`, `min`, `synchronize`, `tolist`。 控制流包含 6 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `CachedRequestState`: central class or interface in this module. / `CachedRequestState`：本模块中的核心类或接口。
- `InputBatch`: central class or interface in this module. / `InputBatch`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`, `typing`
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.config.reasoning`, `vllm.lora.request`, `vllm.multimodal.inputs`, `vllm.pooling_params`, `vllm.sampling_params`, `vllm.utils`, `vllm.utils.collection_utils`, `vllm.v1.outputs`, `vllm.v1.pool.metadata`, `vllm.v1.sample.logits_processor`, `vllm.v1.sample.metadata`, `vllm.v1.sample.thinking_budget_state`
