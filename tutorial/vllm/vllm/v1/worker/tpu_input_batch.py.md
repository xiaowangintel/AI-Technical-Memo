# tpu_input_batch.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/tpu_input_batch.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `InputBatch` for the V1 `worker` subsystem. / 为 V1 的 `worker` 子系统实现 `InputBatch`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from typing import cast

import numpy as np
import torch

from vllm.lora.request import LoRARequest
from vllm.sampling_params import SamplingType
from vllm.utils import length_from_prompt_token_ids_or_embeds
from vllm.utils.collection_utils import swap_dict_values
from vllm.v1.outputs import LogprobsTensors
from vllm.v1.worker.block_table import MultiGroupBlockTable
from vllm.v1.worker.gpu_input_batch import CachedRequestState

_SAMPLING_EPS = 1e-5
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `_SAMPLING_EPS`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `_SAMPLING_EPS`。

### `InputBatch` class / `InputBatch` 类
```python
class InputBatch:
```
**EN:** Introduces the `InputBatch` class. Core methods include `__init__`, `req_ids`, `add_request`, `remove_request`, `swap_states`, `condense`.
**CN:** 这里定义 `InputBatch` 类。核心方法包括 `__init__`, `req_ids`, `add_request`, `remove_request`, `swap_states`, `condense`。

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
    ):
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
        self.num_tokens_no_spec = np.zeros(max_num_reqs, dtype=np.int32)
        self.num_prompt_tokens = np.zeros(max_num_reqs, dtype=np.int32)
        self.num_computed_tokens_cpu_tensor = torch.zeros(
            (max_num_reqs,),
            device="cpu",
            dtype=torch.int32,
            pin_memory=pin_memory,
        )
        self.num_computed_tokens_cpu = self.num_computed_tokens_cpu_tensor.numpy()

        # Block table.
        self.block_table = MultiGroupBlockTable(
            max_num_reqs=max_num_reqs,
            max_model_len=max_model_len,
            max_num_batched_tokens=max_num_batched_tokens,
            pin_memory=pin_memory,
            device=device,
            block_sizes=block_sizes,
    # ... omitted for brevity ...
        self.request_lora_mapping = np.zeros((self.max_num_reqs,), dtype=np.int64)
        self.lora_id_to_request_ids: dict[int, set[str]] = {}
        self.lora_id_to_lora_request: dict[int, LoRARequest] = {}

        # req_index -> generator
        # NOTE(woosuk): The indices of the requests that do not have their own
        # generator should not be included in the dictionary.
        self.generators: dict[int, torch.Generator] = {}

        self.num_logprobs: dict[str, int] = {}

        # To accumulate prompt logprobs tensor chunks across prefill steps.
        self.in_progress_prompt_logprobs_cpu: dict[str, LogprobsTensors] = {}

        self.logit_bias: list[dict[int, float] | None] = [None] * max_num_reqs
        self.has_allowed_token_ids: set[str] = set()
        # NOTE(lufang): In the mask tensor, if the corresponding token allowed,
        # the value is False. Since we use masked_fill_ to set -inf.
        self.allowed_token_ids_mask: torch.Tensor | None = None
        self.allowed_token_ids_mask_cpu_tensor: torch.Tensor | None = None

        # req_index -> bad_words_token_ids
        self.bad_words_token_ids: dict[int, list[list[int]]] = {}

        self.req_output_token_ids: list[list[int] | None] = []
```
**EN:** This method initializes the object state within `InputBatch`. Key calls include `zeros`, `numpy`, `MultiGroupBlockTable`, `empty`, `set`. It touches state such as `max_num_reqs`, `max_model_len`, `max_num_batched_tokens`, `device`, `pin_memory`, `vocab_size`, `_req_ids`, `req_id_to_index`. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会初始化对象状态，其作用域位于`InputBatch`。 关键调用包括 `zeros`, `numpy`, `MultiGroupBlockTable`, `empty`, `set`。 它会读写 `max_num_reqs`, `max_model_len`, `max_num_batched_tokens`, `device`, `pin_memory`, `vocab_size`, `_req_ids`, `req_id_to_index` 等状态。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `InputBatch.add_request` method / `InputBatch.add_request` 方法
```python
    def add_request(
        self,
        request: "CachedRequestState",
        req_index: int | None = None,
    ) -> None:
        if req_index is None:
            req_index = self.num_reqs
        assert req_index < self.max_num_reqs

        req_id = request.req_id
        if req_index == len(self._req_ids):
            self._req_ids.append(req_id)
            self.req_output_token_ids.append(request.output_token_ids)
        else:
            self._req_ids[req_index] = req_id
            self.req_output_token_ids[req_index] = request.output_token_ids

        self.req_id_to_index[req_id] = req_index

        # Copy the prompt token ids and output token ids.
        num_prompt_tokens = length_from_prompt_token_ids_or_embeds(
            request.prompt_token_ids, request.prompt_embeds
        )
        # TODO: copy prompt_embeds
        self.num_prompt_tokens[req_index] = num_prompt_tokens
        self.token_ids_cpu[req_index, :num_prompt_tokens] = request.prompt_token_ids
        start_idx = num_prompt_tokens
        end_idx = start_idx + len(request.output_token_ids)
        self.token_ids_cpu[req_index, start_idx:end_idx] = request.output_token_ids
        # Number of tokens without spec decode tokens.
        self.num_tokens_no_spec[req_index] = request.num_tokens

        self.num_computed_tokens_cpu[req_index] = request.num_computed_tokens
        self.block_table.add_row(request.block_ids, req_index)

        sampling_params = request.sampling_params
        assert sampling_params is not None, "pooling requests not supported yet"
        if sampling_params.sampling_type == SamplingType.GREEDY:
            # Should avoid division by zero later when apply_temperature.
            self.temperature_cpu[req_index] = 0.0
            self.greedy_reqs.add(req_id)
        else:
            self.temperature_cpu[req_index] = sampling_params.temperature
            self.random_reqs.add(req_id)

        self.top_p_cpu[req_index] = sampling_params.top_p
        if sampling_params.top_p < 1:
            self.top_p_reqs.add(req_id)
        top_k = sampling_params.top_k
        if 0 < top_k < self.vocab_size:
    # ... omitted for brevity ...
                )
                self.allowed_token_ids_mask_cpu_tensor = torch.zeros(
                    self.max_num_reqs, self.vocab_size, dtype=torch.bool, device="cpu"
                )
            self.allowed_token_ids_mask_cpu_tensor[req_index] = True
            # False means we don't fill with -inf.
            self.allowed_token_ids_mask_cpu_tensor[req_index][
                sampling_params.allowed_token_ids
            ] = False

        if sampling_params.bad_words_token_ids:
            self.bad_words_token_ids[req_index] = sampling_params.bad_words_token_ids

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
```
**EN:** This method implements `add_request` within `InputBatch`. Key calls include `length_from_prompt_token_ids_or_embeds`, `add_row`, `len`, `append`, `add`, `zeros`. It touches state such as `req_id_to_index`, `num_prompt_tokens`, `token_ids_cpu`, `num_tokens_no_spec`, `num_computed_tokens_cpu`, `top_p_cpu`, `top_k_cpu`, `min_p_cpu`. The control flow contains 18 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `add_request`，其作用域位于`InputBatch`。 关键调用包括 `length_from_prompt_token_ids_or_embeds`, `add_row`, `len`, `append`, `add`, `zeros`。 它会读写 `req_id_to_index`, `num_prompt_tokens`, `token_ids_cpu`, `num_tokens_no_spec`, `num_computed_tokens_cpu`, `top_p_cpu`, `top_k_cpu`, `min_p_cpu` 等状态。 控制流包含 18 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `InputBatch.condense` method / `InputBatch.condense` 方法
```python
    def condense(self, empty_req_indices: list[int]) -> None:
        """Move non-empty requests down into lower, empty indices.

        Args:
          empty_req_indices: empty batch indices, sorted descending.
        """
        num_reqs = self.num_reqs
        if num_reqs == 0:
            # The batched states are empty.
            self._req_ids.clear()
            self.req_output_token_ids.clear()
            return

        # NOTE(woosuk): This function assumes that the empty_req_indices
        # is sorted in descending order.
        last_req_index = num_reqs + len(empty_req_indices) - 1
        while empty_req_indices:
            # Find the largest non-empty index.
            while last_req_index in empty_req_indices:
                last_req_index -= 1

            # Find the smallest empty index.
            empty_index = empty_req_indices.pop()
            if empty_index >= last_req_index:
                break

            # Swap the states.
            req_id = self._req_ids[last_req_index]
            output_token_ids = self.req_output_token_ids[last_req_index]
            assert req_id is not None
            self._req_ids[empty_index] = req_id
            self._req_ids[last_req_index] = None
            self.req_output_token_ids[empty_index] = output_token_ids
            self.req_output_token_ids[last_req_index] = None
            self.req_id_to_index[req_id] = empty_index

            num_tokens = self.num_tokens_no_spec[last_req_index]
            self.token_ids_cpu[empty_index, :num_tokens] = self.token_ids_cpu[
                last_req_index, :num_tokens
            ]
            self.num_tokens_no_spec[empty_index] = self.num_tokens_no_spec[
                last_req_index
            ]
            self.num_prompt_tokens[empty_index] = self.num_prompt_tokens[last_req_index]
            self.num_computed_tokens_cpu[empty_index] = self.num_computed_tokens_cpu[
                last_req_index
            ]
            self.block_table.move_row(last_req_index, empty_index)
            self.temperature_cpu[empty_index] = self.temperature_cpu[last_req_index]
            self.top_p_cpu[empty_index] = self.top_p_cpu[last_req_index]
            self.top_k_cpu[empty_index] = self.top_k_cpu[last_req_index]
            self.frequency_penalties_cpu[empty_index] = self.frequency_penalties_cpu[
                last_req_index
            ]
            self.presence_penalties_cpu[empty_index] = self.presence_penalties_cpu[
                last_req_index
            ]
            self.repetition_penalties_cpu[empty_index] = self.repetition_penalties_cpu[
                last_req_index
            ]
            self.min_p_cpu[empty_index] = self.min_p_cpu[last_req_index]
            generator = self.generators.pop(last_req_index, None)
            if generator is not None:
                self.generators[empty_index] = generator

            min_token = self.min_tokens.pop(last_req_index, None)
            if min_token is not None:
                self.min_tokens[empty_index] = min_token

            self.request_lora_mapping[empty_index] = self.request_lora_mapping[
                last_req_index
            ]

            self.logit_bias[empty_index] = self.logit_bias[last_req_index]

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
        del self._req_ids[self.num_reqs :]
        del self.req_output_token_ids[self.num_reqs :]
```
**EN:** This method implements `condense` within `InputBatch`. The docstring frames it as: Move non-empty requests down into lower, empty indices. Key calls include `clear`, `pop`, `move_row`, `len`. It touches state such as `_req_ids`, `req_output_token_ids`, `req_id_to_index`, `token_ids_cpu`, `num_tokens_no_spec`, `num_prompt_tokens`, `num_computed_tokens_cpu`, `temperature_cpu`. The control flow contains 6 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `condense`，其作用域位于`InputBatch`。 关键调用包括 `clear`, `pop`, `move_row`, `len`。 它会读写 `_req_ids`, `req_output_token_ids`, `req_id_to_index`, `token_ids_cpu`, `num_tokens_no_spec`, `num_prompt_tokens`, `num_computed_tokens_cpu`, `temperature_cpu` 等状态。 控制流包含 6 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `InputBatch`: central class or interface in this module. / `InputBatch`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `typing`
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.lora.request`, `vllm.sampling_params`, `vllm.utils`, `vllm.utils.collection_utils`, `vllm.v1.outputs`, `vllm.v1.worker.block_table`, `vllm.v1.worker.gpu_input_batch`
