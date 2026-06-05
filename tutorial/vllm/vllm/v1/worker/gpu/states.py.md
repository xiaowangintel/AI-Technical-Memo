# states.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/states.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `RequestState` for the V1 `worker/gpu` subsystem. / 为 V1 的 `worker/gpu` 子系统实现 `RequestState`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import numpy as np
import torch

from vllm.v1.worker.gpu.buffer_utils import StagedWriteTensor, UvaBackedTensor
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`, and internal vLLM modules such as `vllm.v1.worker.gpu.buffer_utils`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch` 等外部依赖，以及 `vllm.v1.worker.gpu.buffer_utils` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `RequestState` class / `RequestState` 类
```python
class RequestState:
```
**EN:** Introduces the `RequestState` class. Core methods include `__init__`, `num_reqs`, `add_request`, `apply_staged_writes`, `remove_request`, `is_prefilling`.
**CN:** 这里定义 `RequestState` 类。核心方法包括 `__init__`, `num_reqs`, `add_request`, `apply_staged_writes`, `remove_request`, `is_prefilling`。

### `RequestState.__init__` method / `RequestState.__init__` 方法
```python
    def __init__(
        self,
        max_num_reqs: int,
        max_model_len: int,
        max_num_batched_tokens: int,
        num_speculative_steps: int,
        vocab_size: int,
        device: torch.device,
    ):
        self.max_num_reqs = max_num_reqs
        self.max_model_len = max_model_len
        self.max_num_batched_tokens = max_num_batched_tokens
        self.num_speculative_steps = num_speculative_steps
        self.vocab_size = vocab_size
        self.device = device

        self.req_id_to_index: dict[str, int] = {}
        self.index_to_req_id: dict[int, str] = {}
        self.free_indices = list(range(max_num_reqs))

        # NOTE(woosuk): This tensor can be extremely large (e.g., several GBs)
        # depending on the configured max_num_reqs and max_model_len.
        # To save GPU memory, we use UVA instead of GPU for this tensor.
        self.all_token_ids = StagedWriteTensor(
            (self.max_num_reqs, self.max_model_len),
            dtype=torch.int32,
            device=device,
            uva_instead_of_gpu=True,
        )
        # NOTE(woosuk): Distinguish clearly between prompt_len and prefill_len:
        # - prompt_len: Number of tokens in the user-provided prompt.
        # - prefill_len: Number of tokens passed into the model runner.
        #   This can include the prompt and additional partial output tokens,
        #   so prefill_len >= prompt_len.
        # Usually, prefill_len equals prompt_len, but in cases such as resumption after
        # preemption, prefill_len may be greater. Differentiating between these values
        # is crucial, as certain features such as prompt logprobs or frequency penalties
        # must treat prompt and output tokens separately.
        self.prompt_len = UvaBackedTensor(self.max_num_reqs, dtype=torch.int32)
        self.prefill_len = UvaBackedTensor(self.max_num_reqs, dtype=torch.int32)
        # total_len = prompt_len + output_len. It grows as the request progresses.
        self.total_len = StagedWriteTensor(
            self.max_num_reqs, dtype=torch.int32, device=device
        )

        # Number of computed tokens.
        self.num_computed_prefill_tokens = np.zeros(self.max_num_reqs, dtype=np.int32)
        self.num_computed_tokens = StagedWriteTensor(
            self.max_num_reqs, dtype=torch.int32, device=device
        )
        # Optimistic CPU mirror of num_computed_tokens (upper bound on GPU value).
        self.num_computed_tokens_np = np.zeros(self.max_num_reqs, dtype=np.int32)

        # Last sampled tokens.
        self.last_sampled_tokens = torch.zeros(
            self.max_num_reqs, 1, dtype=torch.int64, device=device
        )

        # Draft tokens.
        self.draft_tokens = torch.zeros(
            self.max_num_reqs,
            self.num_speculative_steps,
            dtype=torch.int64,
            device=device,
        )

        self.next_prefill_tokens = torch.zeros(
            self.max_num_reqs, dtype=torch.int32, device=device
        )
```
**EN:** This method initializes the object state within `RequestState`. Key calls include `list`, `StagedWriteTensor`, `UvaBackedTensor`, `zeros`, `range`. It touches state such as `max_num_reqs`, `max_model_len`, `max_num_batched_tokens`, `num_speculative_steps`, `vocab_size`, `device`, `req_id_to_index`, `index_to_req_id`.
**CN:** 该方法会初始化对象状态，其作用域位于`RequestState`。 关键调用包括 `list`, `StagedWriteTensor`, `UvaBackedTensor`, `zeros`, `range`。 它会读写 `max_num_reqs`, `max_model_len`, `max_num_batched_tokens`, `num_speculative_steps`, `vocab_size`, `device`, `req_id_to_index`, `index_to_req_id` 等状态。

### `RequestState.num_reqs` method / `RequestState.num_reqs` 方法
```python
    @property
    def num_reqs(self) -> int:
        return len(self.req_id_to_index)
```
**EN:** This method implements `num_reqs` within `RequestState`. Key calls include `len`.
**CN:** 该方法会实现 `num_reqs`，其作用域位于`RequestState`。 关键调用包括 `len`。

### `RequestState.add_request` method / `RequestState.add_request` 方法
```python
    def add_request(
        self,
        req_id: str,
        prompt_len: int,
        all_token_ids: list[int],
        num_computed_tokens: int,
    ) -> None:
        assert len(self.free_indices) > 0, "No free indices"
        req_idx = self.free_indices.pop()
        self.req_id_to_index[req_id] = req_idx
        self.index_to_req_id[req_idx] = req_id

        self.prompt_len.np[req_idx] = prompt_len
        prefill_len = len(all_token_ids)
        assert prefill_len >= prompt_len, (
            f"prefill_len {prefill_len} < prompt_len {prompt_len}"
        )
        self.prefill_len.np[req_idx] = prefill_len
        self.total_len.stage_write_elem(req_idx, prefill_len)
        self.all_token_ids.stage_write(req_idx, 0, all_token_ids)
        self.num_computed_prefill_tokens[req_idx] = num_computed_tokens
        self.num_computed_tokens_np[req_idx] = num_computed_tokens
        self.num_computed_tokens.stage_write_elem(req_idx, num_computed_tokens)

        if 0 < num_computed_tokens <= prefill_len:
            # For PD disagg or resumed requests: set last_sampled to the last
            # computed token so the first decode step gets the right input_id.
            # For fresh prefill requests (num_computed_tokens == 0) the tensor
            # is not read by combine_sampled_and_draft_tokens so we skip the
            # write. Use a slice assignment rather than scalar indexing so the
            # write is dispatched through fill_ without a host/device sync.
            self.last_sampled_tokens[req_idx : req_idx + 1] = all_token_ids[
                num_computed_tokens - 1
            ]
        self.draft_tokens[req_idx].zero_()
```
**EN:** This method implements `add_request` within `RequestState`. Key calls include `pop`, `len`, `stage_write_elem`, `stage_write`, `zero_`. It touches state such as `req_id_to_index`, `index_to_req_id`, `prompt_len`, `prefill_len`, `num_computed_prefill_tokens`, `num_computed_tokens_np`, `last_sampled_tokens`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `add_request`，其作用域位于`RequestState`。 关键调用包括 `pop`, `len`, `stage_write_elem`, `stage_write`, `zero_`。 它会读写 `req_id_to_index`, `index_to_req_id`, `prompt_len`, `prefill_len`, `num_computed_prefill_tokens`, `num_computed_tokens_np`, `last_sampled_tokens` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RequestState.apply_staged_writes` method / `RequestState.apply_staged_writes` 方法
```python
    def apply_staged_writes(self) -> None:
        self.prompt_len.copy_to_uva()
        self.prefill_len.copy_to_uva()
        self.total_len.apply_write()
        self.all_token_ids.apply_write()
        self.num_computed_tokens.apply_write()
```
**EN:** This method implements `apply_staged_writes` within `RequestState`. Key calls include `copy_to_uva`, `apply_write`.
**CN:** 该方法会实现 `apply_staged_writes`，其作用域位于`RequestState`。 关键调用包括 `copy_to_uva`, `apply_write`。

### `RequestState.remove_request` method / `RequestState.remove_request` 方法
```python
    def remove_request(self, req_id: str) -> bool:
        req_idx = self.req_id_to_index.pop(req_id, None)
        if req_idx is None:
            # Request not found.
            return False
        self.index_to_req_id.pop(req_idx, None)
        self.free_indices.append(req_idx)
        return True
```
**EN:** This method implements `remove_request` within `RequestState`. Key calls include `pop`, `append`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `remove_request`，其作用域位于`RequestState`。 关键调用包括 `pop`, `append`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `RequestState.is_prefilling` method / `RequestState.is_prefilling` 方法
```python
    def is_prefilling(self, idx_mapping_np: np.ndarray) -> np.ndarray:
        return (
            self.num_computed_prefill_tokens[idx_mapping_np]
            < self.prefill_len.np[idx_mapping_np]
        )
```
**EN:** This method answers a boolean capability check within `RequestState`.
**CN:** 该方法会回答布尔能力判断，其作用域位于`RequestState`。

## Key Concepts / 关键概念
- `RequestState`: central class or interface in this module. / `RequestState`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.v1.worker.gpu.buffer_utils`
