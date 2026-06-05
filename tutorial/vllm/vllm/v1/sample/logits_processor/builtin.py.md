# builtin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/sample/logits_processor/builtin.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `MinPLogitsProcessor`, `LogitBiasLogitsProcessor`, `MinTokensLogitsProcessor` for the V1 `sample/logits_processor` subsystem. / 为 V1 的 `sample/logits_processor` 子系统实现 `MinPLogitsProcessor`, `LogitBiasLogitsProcessor`, `MinTokensLogitsProcessor`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from collections.abc import Callable, Sequence
from typing import TYPE_CHECKING, TypeVar

import numpy as np
import torch

from vllm import SamplingParams
from vllm.v1.sample.logits_processor.interface import (
    BatchUpdate,
    LogitsProcessor,
    MoveDirectionality,
)

if TYPE_CHECKING:
    from vllm.config import VllmConfig

T = TypeVar("T")
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `T`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `T`。

### `MinPLogitsProcessor` class / `MinPLogitsProcessor` 类
```python
class MinPLogitsProcessor(LogitsProcessor):
```
**EN:** Introduces the `MinPLogitsProcessor` class on top of `LogitsProcessor`. Core methods include `__init__`, `is_argmax_invariant`, `get_min_p_by_index`, `update_state`, `apply`.
**CN:** 这里定义 `MinPLogitsProcessor` 类，其基类包括 `LogitsProcessor`。核心方法包括 `__init__`, `is_argmax_invariant`, `get_min_p_by_index`, `update_state`, `apply`。

### `MinPLogitsProcessor.__init__` method / `MinPLogitsProcessor.__init__` 方法
```python
    def __init__(
        self, vllm_config: "VllmConfig", device: torch.device, is_pin_memory: bool
    ):
        max_num_reqs = vllm_config.scheduler_config.max_num_seqs
        self.min_p_count: int = 0

        self.min_p_cpu_tensor = torch.zeros(
            (max_num_reqs,), dtype=torch.float32, device="cpu", pin_memory=is_pin_memory
        )
        self.min_p_cpu = self.min_p_cpu_tensor.numpy()

        self.use_double_tensor = torch.device(device).type != "cpu"

        if self.use_double_tensor:
            # Pre-allocated device tensor
            self.min_p_device: torch.Tensor = torch.empty(
                (max_num_reqs,), dtype=torch.float32, device=device
            )
        else:
            self.min_p_device = self.min_p_cpu_tensor
        # Current slice of the device tensor
        self.min_p: torch.Tensor = self.min_p_device[:0]
```
**EN:** This method initializes the object state within `MinPLogitsProcessor`. Key calls include `zeros`, `numpy`, `empty`, `device`. It touches state such as `min_p_count`, `min_p_cpu_tensor`, `min_p_cpu`, `use_double_tensor`, `min_p`, `min_p_device`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`MinPLogitsProcessor`。 关键调用包括 `zeros`, `numpy`, `empty`, `device`。 它会读写 `min_p_count`, `min_p_cpu_tensor`, `min_p_cpu`, `use_double_tensor`, `min_p`, `min_p_device` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `MinPLogitsProcessor.is_argmax_invariant` method / `MinPLogitsProcessor.is_argmax_invariant` 方法
```python
    def is_argmax_invariant(self) -> bool:
        """Min-p never impacts greedy sampling"""
        return True
```
**EN:** This method answers a boolean capability check within `MinPLogitsProcessor`. The docstring frames it as: Min-p never impacts greedy sampling
**CN:** 该方法会回答布尔能力判断，其作用域位于`MinPLogitsProcessor`。

### `MinPLogitsProcessor.get_min_p_by_index` method / `MinPLogitsProcessor.get_min_p_by_index` 方法
```python
    def get_min_p_by_index(self, index: int) -> float:
        return float(self.min_p_cpu[index])
```
**EN:** This method returns or derives a value within `MinPLogitsProcessor`. Key calls include `float`.
**CN:** 该方法会返回或推导一个值，其作用域位于`MinPLogitsProcessor`。 关键调用包括 `float`。

### `MinPLogitsProcessor.update_state` method / `MinPLogitsProcessor.update_state` 方法
```python
    def update_state(self, batch_update: BatchUpdate | None):
        if not batch_update:
            return

        needs_update = False
        # Process added requests.
        for index, params, _, _ in batch_update.added:
            min_p = params.min_p
            min_p_before = self.min_p_cpu[index]
            if min_p_before != min_p:
                needs_update = True
                self.min_p_cpu[index] = min_p
                if min_p and not min_p_before:
                    self.min_p_count += 1
                elif not min_p and min_p_before:
                    self.min_p_count -= 1

        if self.min_p_count:
            # Process removed requests.
            if batch_update.removed:
                needs_update = True
                for index in batch_update.removed:
                    if self.min_p_cpu[index]:
                        self.min_p_cpu[index] = 0
                        self.min_p_count -= 1

            # Process moved requests, unidirectional (a->b) and swap (a<->b).
            for adx, bdx, direct in batch_update.moved:
                min_p_a, min_p_b = self.min_p_cpu[adx], self.min_p_cpu[bdx]
                if min_p_a != min_p_b:
                    needs_update = True
                    self.min_p_cpu[bdx] = min_p_a
                    if direct == MoveDirectionality.SWAP:
                        self.min_p_cpu[adx] = min_p_b
                if direct == MoveDirectionality.UNIDIRECTIONAL:
                    if min_p_a:
                        self.min_p_cpu[adx] = 0
                    if min_p_b:
                        self.min_p_count -= 1

        # Update tensors if needed.
        size = batch_update.batch_size
        if self.min_p_count and (needs_update or self.min_p.shape[0] != size):
            self.min_p = self.min_p_device[:size]
            if self.use_double_tensor:
                self.min_p.copy_(self.min_p_cpu_tensor[:size], non_blocking=True)
            self.min_p.unsqueeze_(1)
```
**EN:** This method updates existing state within `MinPLogitsProcessor`. Key calls include `unsqueeze_`, `copy_`. It touches state such as `min_p`, `min_p_cpu`, `min_p_count`. The control flow contains 14 branch(es) and 3 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会更新现有状态，其作用域位于`MinPLogitsProcessor`。 关键调用包括 `unsqueeze_`, `copy_`。 它会读写 `min_p`, `min_p_cpu`, `min_p_count` 等状态。 控制流包含 14 个分支和 3 个循环，说明这里承担了较强的协调逻辑。

### `MinPLogitsProcessor.apply` method / `MinPLogitsProcessor.apply` 方法
```python
    def apply(self, logits: torch.Tensor) -> torch.Tensor:
        if not self.min_p_count:
            return logits

        # Convert logits to probability distribution
        probability_values = torch.nn.functional.softmax(logits, dim=-1)
        # Calculate maximum probabilities per sequence
        max_probabilities = torch.amax(probability_values, dim=-1, keepdim=True)
        # Adjust min_p
        adjusted_min_p = max_probabilities.mul_(self.min_p)
        # Identify valid tokens using threshold comparison
        invalid_token_mask = probability_values < adjusted_min_p
        # Apply mask using boolean indexing
        logits.masked_fill_(invalid_token_mask, -float("inf"))
        return logits
```
**EN:** This method implements `apply` within `MinPLogitsProcessor`. Key calls include `softmax`, `amax`, `mul_`, `masked_fill_`, `float`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `apply`，其作用域位于`MinPLogitsProcessor`。 关键调用包括 `softmax`, `amax`, `mul_`, `masked_fill_`, `float`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `LogitBiasLogitsProcessor` class / `LogitBiasLogitsProcessor` 类
```python
class LogitBiasLogitsProcessor(LogitsProcessor):
```
**EN:** Introduces the `LogitBiasLogitsProcessor` class on top of `LogitsProcessor`. Core methods include `__init__`, `is_argmax_invariant`, `update_state`, `_device_tensor`, `apply`.
**CN:** 这里定义 `LogitBiasLogitsProcessor` 类，其基类包括 `LogitsProcessor`。核心方法包括 `__init__`, `is_argmax_invariant`, `update_state`, `_device_tensor`, `apply`。

### `LogitBiasLogitsProcessor.__init__` method / `LogitBiasLogitsProcessor.__init__` 方法
```python
    def __init__(self, _, device: torch.device, is_pin_memory: bool):
        self.device = device
        self.pin_memory = is_pin_memory
        self.biases: dict[int, dict[int, float]] = {}

        self.bias_tensor: torch.Tensor = torch.tensor(())
        self.logits_slice = (
            self._device_tensor([], torch.int32),
            self._device_tensor([], torch.int32),
        )
```
**EN:** This method initializes the object state within `LogitBiasLogitsProcessor`. Key calls include `tensor`, `_device_tensor`. It touches state such as `device`, `pin_memory`, `biases`, `bias_tensor`, `logits_slice`.
**CN:** 该方法会初始化对象状态，其作用域位于`LogitBiasLogitsProcessor`。 关键调用包括 `tensor`, `_device_tensor`。 它会读写 `device`, `pin_memory`, `biases`, `bias_tensor`, `logits_slice` 等状态。

### `LogitBiasLogitsProcessor.is_argmax_invariant` method / `LogitBiasLogitsProcessor.is_argmax_invariant` 方法
```python
    def is_argmax_invariant(self) -> bool:
        """Logit bias can rebalance token probabilities and change the
        outcome of argmax in greedy sampling."""
        return False
```
**EN:** This method answers a boolean capability check within `LogitBiasLogitsProcessor`. The docstring frames it as: Logit bias can rebalance token probabilities and change the outcome of argmax in greedy sampling.
**CN:** 该方法会回答布尔能力判断，其作用域位于`LogitBiasLogitsProcessor`。

### `LogitBiasLogitsProcessor.update_state` method / `LogitBiasLogitsProcessor.update_state` 方法
```python
    def update_state(self, batch_update: BatchUpdate | None):
        needs_update = process_dict_updates(
            self.biases, batch_update, lambda params, _, __: params.logit_bias or None
        )

        # Update tensors if needed.
        if needs_update:
            reqs: list[int] = []
            tok_ids: list[int] = []
            biases: list[float] = []
            for req, lb in self.biases.items():
                reqs.extend([req] * len(lb))
                tok_ids.extend(lb.keys())
                biases.extend(lb.values())

            self.bias_tensor = self._device_tensor(biases, torch.float32)
            self.logits_slice = (
                self._device_tensor(reqs, torch.int32),
                self._device_tensor(tok_ids, torch.int32),
            )
```
**EN:** This method updates existing state within `LogitBiasLogitsProcessor`. Key calls include `process_dict_updates`, `items`, `_device_tensor`, `extend`, `keys`, `values`. It touches state such as `bias_tensor`, `logits_slice`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会更新现有状态，其作用域位于`LogitBiasLogitsProcessor`。 关键调用包括 `process_dict_updates`, `items`, `_device_tensor`, `extend`, `keys`, `values`。 它会读写 `bias_tensor`, `logits_slice` 等状态。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `LogitBiasLogitsProcessor._device_tensor` method / `LogitBiasLogitsProcessor._device_tensor` 方法
```python
    def _device_tensor(self, data: list, dtype: torch.dtype) -> torch.Tensor:
        return torch.tensor(
            data, device="cpu", dtype=dtype, pin_memory=self.pin_memory
        ).to(device=self.device, non_blocking=True)
```
**EN:** This method implements `_device_tensor` within `LogitBiasLogitsProcessor`. Key calls include `to`, `tensor`.
**CN:** 该方法会实现 `_device_tensor`，其作用域位于`LogitBiasLogitsProcessor`。 关键调用包括 `to`, `tensor`。

### `LogitBiasLogitsProcessor.apply` method / `LogitBiasLogitsProcessor.apply` 方法
```python
    def apply(self, logits: torch.Tensor) -> torch.Tensor:
        if self.biases:
            logits[self.logits_slice] += self.bias_tensor
        return logits
```
**EN:** This method implements `apply` within `LogitBiasLogitsProcessor`. It touches state such as `logits_slice`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `apply`，其作用域位于`LogitBiasLogitsProcessor`。 它会读写 `logits_slice` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `MinTokensLogitsProcessor` class / `MinTokensLogitsProcessor` 类
```python
class MinTokensLogitsProcessor(LogitsProcessor):
```
**EN:** Introduces the `MinTokensLogitsProcessor` class on top of `LogitsProcessor`. Core methods include `__init__`, `is_argmax_invariant`, `add_request`, `update_state`, `_device_tensor`, `apply`.
**CN:** 这里定义 `MinTokensLogitsProcessor` 类，其基类包括 `LogitsProcessor`。核心方法包括 `__init__`, `is_argmax_invariant`, `add_request`, `update_state`, `_device_tensor`, `apply`。

### `MinTokensLogitsProcessor.__init__` method / `MinTokensLogitsProcessor.__init__` 方法
```python
    def __init__(
        self, vllm_config: "VllmConfig", device: torch.device, is_pin_memory: bool
    ):
        # index -> (min_toks, output_token_ids, stop_token_ids)
        self.device = device
        self.pin_memory = is_pin_memory
        self.min_toks: dict[int, tuple[int, Sequence[int], set[int]]] = {}

        # (req_idx_tensor,eos_tok_id_tensor)
        self.logits_slice: tuple[torch.Tensor, torch.Tensor] = (
            self._device_tensor([], torch.int32),
            self._device_tensor([], torch.int32),
        )

        self.neg_inf_tensor = torch.tensor(
            -float("inf"), dtype=torch.float32, device=self.device
        )
```
**EN:** This method initializes the object state within `MinTokensLogitsProcessor`. Key calls include `tensor`, `_device_tensor`, `float`. It touches state such as `device`, `pin_memory`, `min_toks`, `logits_slice`, `neg_inf_tensor`.
**CN:** 该方法会初始化对象状态，其作用域位于`MinTokensLogitsProcessor`。 关键调用包括 `tensor`, `_device_tensor`, `float`。 它会读写 `device`, `pin_memory`, `min_toks`, `logits_slice`, `neg_inf_tensor` 等状态。

### `MinTokensLogitsProcessor.is_argmax_invariant` method / `MinTokensLogitsProcessor.is_argmax_invariant` 方法
```python
    def is_argmax_invariant(self) -> bool:
        """By censoring stop tokens, min-tokens can change the outcome
        of the argmax operation in greedy sampling."""
        return False
```
**EN:** This method answers a boolean capability check within `MinTokensLogitsProcessor`. The docstring frames it as: By censoring stop tokens, min-tokens can change the outcome of the argmax operation in greedy sampling.
**CN:** 该方法会回答布尔能力判断，其作用域位于`MinTokensLogitsProcessor`。

### `MinTokensLogitsProcessor.add_request` method / `MinTokensLogitsProcessor.add_request` 方法
```python
    @staticmethod
    def add_request(
        params: SamplingParams, _: list[int] | None, output_tok_ids: list[int]
    ) -> tuple[int, Sequence[int], set[int]] | None:
        min_tokens = params.min_tokens
        if not min_tokens or len(output_tok_ids) >= min_tokens:
            return None
        return min_tokens, output_tok_ids, params.all_stop_token_ids
```
**EN:** This method implements `add_request` within `MinTokensLogitsProcessor`. Key calls include `len`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `add_request`，其作用域位于`MinTokensLogitsProcessor`。 关键调用包括 `len`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `MinTokensLogitsProcessor.update_state` method / `MinTokensLogitsProcessor.update_state` 方法
```python
    def update_state(self, batch_update: BatchUpdate | None):
        needs_update = process_dict_updates(
            self.min_toks, batch_update, self.add_request
        )
        if self.min_toks:
            # Check for any requests that have attained their min tokens.
            to_remove = tuple(
                index
                for index, (min_toks, out_tok_ids, _) in self.min_toks.items()
                if len(out_tok_ids) >= min_toks
            )
            if to_remove:
                needs_update = True
                for index in to_remove:
                    del self.min_toks[index]

        # Update tensors if needed.
        if needs_update:
            reqs: list[int] = []
            tok_ids: list[int] = []
            for req, (_, _, stop_tok_ids) in self.min_toks.items():
                reqs.extend([req] * len(stop_tok_ids))
                tok_ids.extend(stop_tok_ids)

            self.logits_slice = (
                self._device_tensor(reqs, torch.int32),
                self._device_tensor(tok_ids, torch.int32),
            )
```
**EN:** This method updates existing state within `MinTokensLogitsProcessor`. Key calls include `process_dict_updates`, `tuple`, `items`, `extend`, `_device_tensor`, `len`. It touches state such as `logits_slice`. The control flow contains 3 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会更新现有状态，其作用域位于`MinTokensLogitsProcessor`。 关键调用包括 `process_dict_updates`, `tuple`, `items`, `extend`, `_device_tensor`, `len`。 它会读写 `logits_slice` 等状态。 控制流包含 3 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `MinTokensLogitsProcessor._device_tensor` method / `MinTokensLogitsProcessor._device_tensor` 方法
```python
    def _device_tensor(self, data: list, dtype: torch.dtype) -> torch.Tensor:
        return torch.tensor(
            data, device="cpu", dtype=dtype, pin_memory=self.pin_memory
        ).to(device=self.device, non_blocking=True)
```
**EN:** This method implements `_device_tensor` within `MinTokensLogitsProcessor`. Key calls include `to`, `tensor`.
**CN:** 该方法会实现 `_device_tensor`，其作用域位于`MinTokensLogitsProcessor`。 关键调用包括 `to`, `tensor`。

### `MinTokensLogitsProcessor.apply` method / `MinTokensLogitsProcessor.apply` 方法
```python
    def apply(self, logits: torch.Tensor) -> torch.Tensor:
        if self.min_toks:
            # Inhibit EOS token for requests which have not reached min length
            logits.index_put_(self.logits_slice, self.neg_inf_tensor)
        return logits
```
**EN:** This method implements `apply` within `MinTokensLogitsProcessor`. Key calls include `index_put_`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `apply`，其作用域位于`MinTokensLogitsProcessor`。 关键调用包括 `index_put_`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `MinTokensLogitsProcessor.apply_with_spec_decode` method / `MinTokensLogitsProcessor.apply_with_spec_decode` 方法
```python
    def apply_with_spec_decode(
        self,
        logits: torch.Tensor,
        num_draft_tokens: list[int],
    ) -> torch.Tensor:
        """Spec-decode version of apply().
        Priority: ``min_tokens`` > ``stop_token_ids`` / EOS.
        Example: ``num_draft_tokens = [2, 3, 1]``
          → ``logits`` shape ``[6, V]``, ``cumsum = [0, 2, 5, 6]``
          → request 0 owns rows 0‑1, request 1 rows 2‑4, request 2 row 5.
        """
        if not self.min_toks:
            return logits

        num_draft_arr = np.array(num_draft_tokens, dtype=np.int64)
        cumsum = np.concatenate([[0], np.cumsum(num_draft_arr)])

        entries = [
            (req_idx, min_tok, len(out_tok_ids), list(stop_tok_ids))
            for req_idx, (min_tok, out_tok_ids, stop_tok_ids) in self.min_toks.items()
            if stop_tok_ids
        ]

        if not entries:
            return logits

        all_rows: list[np.ndarray] = []  # row indices to mask
        all_toks: list[np.ndarray] = []  # stop-token ids at those rows

        for req_idx, min_tok, current_len, stop_toks in entries:
            remaining = min_tok - current_len
            # How many leading draft positions still need stop-token masking.
            n_mask = int(min(max(remaining, 0), num_draft_arr[req_idx]))

            if n_mask > 0:
                offset = cumsum[req_idx]
                row_indices = np.arange(offset, offset + n_mask, dtype=np.int64)
                n_stop = len(stop_toks)
                all_rows.append(np.repeat(row_indices, n_stop))
                all_toks.append(np.tile(stop_toks, n_mask))

        if all_rows:
            rows_arr = np.concatenate(all_rows)
            toks_arr = np.concatenate(all_toks)
            # (row_indices, token_indices) for index_put_ to set -inf.
            logits_slice = (
                torch.from_numpy(rows_arr).to(self.device, non_blocking=True),
                torch.from_numpy(toks_arr).to(self.device, non_blocking=True),
            )
            logits.index_put_(logits_slice, self.neg_inf_tensor)

        return logits
```
**EN:** This method implements `apply_with_spec_decode` within `MinTokensLogitsProcessor`. The docstring frames it as: Spec-decode version of apply(). Key calls include `array`, `concatenate`, `int`, `index_put_`, `cumsum`, `len`. The control flow contains 4 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `apply_with_spec_decode`，其作用域位于`MinTokensLogitsProcessor`。 关键调用包括 `array`, `concatenate`, `int`, `index_put_`, `cumsum`, `len`。 控制流包含 4 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `process_dict_updates` function / `process_dict_updates` 函数
```python
def process_dict_updates(
    req_entries: dict[int, T],
    batch_update: BatchUpdate | None,
    new_state: Callable[[SamplingParams, list[int] | None, list[int]], T | None],
) -> bool:
    """Utility function to update dict state for sparse LogitsProcessors."""

    if not batch_update:
        # Nothing to do.
        return False

    updated = False
    for index, params, prompt_tok_ids, output_tok_ids in batch_update.added:
        if (state := new_state(params, prompt_tok_ids, output_tok_ids)) is not None:
            req_entries[index] = state
            updated = True
        elif req_entries.pop(index, None) is not None:
            updated = True

    if req_entries:
        # Process removed requests.
        for index in batch_update.removed:
            if req_entries.pop(index, None):
                updated = True

        # Process moved requests, unidirectional (a->b) and
        # swapped (a<->b)
        for a_index, b_index, direct in batch_update.moved:
            a_entry = req_entries.pop(a_index, None)
            b_entry = req_entries.pop(b_index, None)
            if a_entry is not None:
                req_entries[b_index] = a_entry
                updated = True
            if b_entry is not None:
                updated = True
                if direct == MoveDirectionality.SWAP:
                    req_entries[a_index] = b_entry

    return updated
```
**EN:** This function implements `process_dict_updates` within the module. The docstring frames it as: Utility function to update dict state for sparse LogitsProcessors. Key calls include `pop`, `new_state`. The control flow contains 8 branch(es) and 3 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `process_dict_updates`，其作用域位于the module。 关键调用包括 `pop`, `new_state`。 控制流包含 8 个分支和 3 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `MinPLogitsProcessor`: central class or interface in this module. / `MinPLogitsProcessor`：本模块中的核心类或接口。
- `LogitBiasLogitsProcessor`: central class or interface in this module. / `LogitBiasLogitsProcessor`：本模块中的核心类或接口。
- `MinTokensLogitsProcessor`: central class or interface in this module. / `MinTokensLogitsProcessor`：本模块中的核心类或接口。
- `process_dict_updates`: top-level helper or orchestration entry point. / `process_dict_updates`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `collections`, `typing`
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm`, `vllm.v1.sample.logits_processor.interface`, `vllm.config`
