# penalties.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/sample/penalties.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `PenaltiesState`, `_penalties_kernel`, `apply_penalties` for the V1 `worker/gpu/sample` subsystem. / 为 V1 的 `worker/gpu/sample` 子系统实现 `PenaltiesState`, `_penalties_kernel`, `apply_penalties`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import numpy as np
import torch

from vllm.sampling_params import SamplingParams
from vllm.triton_utils import tl, triton
from vllm.utils.math_utils import cdiv
from vllm.utils.torch_utils import async_tensor_h2d
from vllm.v1.worker.gpu.buffer_utils import UvaBackedTensor
from vllm.v1.worker.gpu.states import RequestState
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`, and internal vLLM modules such as `vllm.sampling_params`, `vllm.triton_utils`, `vllm.utils.math_utils`, `vllm.utils.torch_utils`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch` 等外部依赖，以及 `vllm.sampling_params`, `vllm.triton_utils`, `vllm.utils.math_utils`, `vllm.utils.torch_utils` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `PenaltiesState` class / `PenaltiesState` 类
```python
class PenaltiesState:
```
**EN:** Introduces the `PenaltiesState` class. Core methods include `__init__`, `add_request`, `apply_staged_writes`, `apply_penalties`.
**CN:** 这里定义 `PenaltiesState` 类。核心方法包括 `__init__`, `add_request`, `apply_staged_writes`, `apply_penalties`。

### `PenaltiesState.__init__` method / `PenaltiesState.__init__` 方法
```python
    def __init__(self, req_states: RequestState):
        self.req_states = req_states

        max_num_reqs = req_states.max_num_reqs
        self.vocab_size = req_states.vocab_size
        self.device = req_states.device

        self.repetition_penalty = UvaBackedTensor(max_num_reqs, dtype=torch.float32)
        self.frequency_penalty = UvaBackedTensor(max_num_reqs, dtype=torch.float32)
        self.presence_penalty = UvaBackedTensor(max_num_reqs, dtype=torch.float32)
        self.use_penalty = np.zeros(max_num_reqs, dtype=bool)

        # Initialize repetition penalty manually because 0 is an invalid value for it.
        self.repetition_penalty.np.fill(1.0)
        self.repetition_penalty.copy_to_uva()

        # Statistics for penalties.
        self.prompt_bin_mask = torch.zeros(
            max_num_reqs,
            cdiv(self.vocab_size, 32),
            dtype=torch.int32,
            device=self.device,
        )
        # TODO(woosuk): This tensor is rarely used but can be very large, taking up
        # GBs of GPU memory. Optimize the memory usage.
        self.output_bin_counts = torch.zeros(
            max_num_reqs, self.vocab_size, dtype=torch.int32, device=self.device
        )

        self._new_penalties_reqs: list[int] = []
```
**EN:** This method initializes the object state within `PenaltiesState`. Key calls include `UvaBackedTensor`, `zeros`, `fill`, `copy_to_uva`, `cdiv`. It touches state such as `req_states`, `vocab_size`, `device`, `repetition_penalty`, `frequency_penalty`, `presence_penalty`, `use_penalty`, `prompt_bin_mask`.
**CN:** 该方法会初始化对象状态，其作用域位于`PenaltiesState`。 关键调用包括 `UvaBackedTensor`, `zeros`, `fill`, `copy_to_uva`, `cdiv`。 它会读写 `req_states`, `vocab_size`, `device`, `repetition_penalty`, `frequency_penalty`, `presence_penalty`, `use_penalty`, `prompt_bin_mask` 等状态。

### `PenaltiesState.add_request` method / `PenaltiesState.add_request` 方法
```python
    def add_request(self, req_idx: int, sampling_params: SamplingParams) -> None:
        self.repetition_penalty.np[req_idx] = sampling_params.repetition_penalty
        self.frequency_penalty.np[req_idx] = sampling_params.frequency_penalty
        self.presence_penalty.np[req_idx] = sampling_params.presence_penalty

        do_penalty = use_penalty(sampling_params)
        self.use_penalty[req_idx] = do_penalty
        if do_penalty:
            self._new_penalties_reqs.append(req_idx)
```
**EN:** This method implements `add_request` within `PenaltiesState`. Key calls include `use_penalty`, `append`. It touches state such as `repetition_penalty`, `frequency_penalty`, `presence_penalty`, `use_penalty`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `add_request`，其作用域位于`PenaltiesState`。 关键调用包括 `use_penalty`, `append`。 它会读写 `repetition_penalty`, `frequency_penalty`, `presence_penalty`, `use_penalty` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `PenaltiesState.apply_staged_writes` method / `PenaltiesState.apply_staged_writes` 方法
```python
    def apply_staged_writes(self) -> None:
        if self._new_penalties_reqs:
            idx_mapping = async_tensor_h2d(
                self._new_penalties_reqs,
                dtype=torch.int32,
                device=self.device,
            )

            prefill_lens = self.req_states.prefill_len.np[self._new_penalties_reqs]
            max_prefill_len = int(prefill_lens.max())
            bincount(
                idx_mapping,
                self.req_states.all_token_ids.gpu,
                self.req_states.prompt_len.gpu,
                self.req_states.prefill_len.gpu,
                self.prompt_bin_mask,
                self.output_bin_counts,
                max_prefill_len,
            )
            self._new_penalties_reqs.clear()

        self.repetition_penalty.copy_to_uva()
        self.frequency_penalty.copy_to_uva()
        self.presence_penalty.copy_to_uva()
```
**EN:** This method implements `apply_staged_writes` within `PenaltiesState`. Key calls include `copy_to_uva`, `async_tensor_h2d`, `int`, `bincount`, `clear`, `max`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `apply_staged_writes`，其作用域位于`PenaltiesState`。 关键调用包括 `copy_to_uva`, `async_tensor_h2d`, `int`, `bincount`, `clear`, `max`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `PenaltiesState.apply_penalties` method / `PenaltiesState.apply_penalties` 方法
```python
    def apply_penalties(
        self,
        logits: torch.Tensor,
        expanded_idx_mapping: torch.Tensor,
        idx_mapping_np: np.ndarray,
        input_ids: torch.Tensor,
        expanded_local_pos: torch.Tensor,
    ) -> None:
        if not np.any(self.use_penalty[idx_mapping_np]):
            # No request uses penalties. Skip the kernel launch.
            return

        apply_penalties(
            logits,
            expanded_idx_mapping,
            input_ids,
            expanded_local_pos,
            self.repetition_penalty.gpu,
            self.frequency_penalty.gpu,
            self.presence_penalty.gpu,
            self.prompt_bin_mask,
            self.output_bin_counts,
        )
```
**EN:** This method implements `apply_penalties` within `PenaltiesState`. Key calls include `apply_penalties`, `any`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `apply_penalties`，其作用域位于`PenaltiesState`。 关键调用包括 `apply_penalties`, `any`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_penalties_kernel` function / `_penalties_kernel` 函数
```python
@triton.jit
def _penalties_kernel(
    logits_ptr,
    logits_stride,
    expanded_idx_mapping_ptr,
    token_ids_ptr,
    expanded_local_pos_ptr,
    repetition_penalty_ptr,
    frequency_penalty_ptr,
    presence_penalty_ptr,
    prompt_bin_mask_ptr,
    prompt_bin_mask_stride,
    output_bin_counts_ptr,
    output_bin_counts_stride,
    vocab_size,
    BLOCK_SIZE: tl.constexpr,
):
    token_idx = tl.program_id(0)
    req_state_idx = tl.load(expanded_idx_mapping_ptr + token_idx)
    rep_penalty = tl.load(repetition_penalty_ptr + req_state_idx)
    freq_penalty = tl.load(frequency_penalty_ptr + req_state_idx)
    pres_penalty = tl.load(presence_penalty_ptr + req_state_idx)

    use_rep_penalty = rep_penalty != 1.0
    use_freq_penalty = freq_penalty != 0.0
    use_pres_penalty = pres_penalty != 0.0
    use_penalty = use_rep_penalty or use_freq_penalty or use_pres_penalty
    if not use_penalty:
        # Early return to avoid loading logits.
        return

    block_idx = tl.program_id(1)
    block = block_idx * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    mask = block < vocab_size
    logits = tl.load(logits_ptr + token_idx * logits_stride + block, mask=mask)
    logits = logits.to(tl.float32)

    base_output_counts = tl.load(
        output_bin_counts_ptr + req_state_idx * output_bin_counts_stride + block,
        mask=mask,
        other=0,
    )

    # Accumulate draft token counts from previous positions directly into
    # output_bin_counts (preserves its native tensor layout, avoiding an
    # expensive shared-memory layout conversion after the loop).
    pos = tl.load(expanded_local_pos_ptr + token_idx)
    start_idx = token_idx - pos
    output_bin_counts = base_output_counts
    for prev_pos in tl.range(pos):
        prev_token = tl.load(token_ids_ptr + start_idx + prev_pos + 1)
        token_match = block == prev_token
        output_bin_counts = output_bin_counts + token_match.to(tl.int32)
    output_bin_mask = output_bin_counts > 0

    # Apply repetition penalties.
    if use_rep_penalty:
        packed_block = block_idx * BLOCK_SIZE // 32 + tl.arange(0, BLOCK_SIZE // 32)
        packed_mask = tl.load(
            prompt_bin_mask_ptr + req_state_idx * prompt_bin_mask_stride + packed_block,
            mask=packed_block < tl.cdiv(vocab_size, 32),
            other=0,
        )
        prompt_bin_mask = (packed_mask[:, None] >> (tl.arange(0, 32)[None, :])) & 1
        prompt_bin_mask = prompt_bin_mask.to(tl.int1)
        prompt_bin_mask = prompt_bin_mask.reshape(BLOCK_SIZE)

        # If token appears in prompt or output, apply, otherwise use 1.0 for no-op.
        scale = tl.where(prompt_bin_mask | output_bin_mask, rep_penalty, 1.0)
        # If logits are positive, divide by penalty, otherwise multiply by penalty.
        logits *= tl.where(logits > 0, 1.0 / scale, scale)

    # Apply frequency penalties.
    logits -= freq_penalty * output_bin_counts
    # Apply presence penalties.
    logits -= pres_penalty * output_bin_mask
    # Store back to logits.
    tl.store(logits_ptr + token_idx * logits_stride + block, logits, mask=mask)
```
**EN:** This function implements `_penalties_kernel` within the module. Key calls include `program_id`, `load`, `to`, `range`, `store`, `arange`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_penalties_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `to`, `range`, `store`, `arange`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `apply_penalties` function / `apply_penalties` 函数
```python
def apply_penalties(
    logits: torch.Tensor,
    expanded_idx_mapping: torch.Tensor,
    token_ids: torch.Tensor,
    expanded_local_pos: torch.Tensor,
    repetition_penalty: torch.Tensor,
    frequency_penalty: torch.Tensor,
    presence_penalty: torch.Tensor,
    prompt_bin_mask: torch.Tensor,
    output_bin_counts: torch.Tensor,
) -> None:
    num_tokens, vocab_size = logits.shape
    BLOCK_SIZE = 8192
    num_blocks = triton.cdiv(vocab_size, BLOCK_SIZE)
    _penalties_kernel[(num_tokens, num_blocks)](
        logits,
        logits.stride(0),
        expanded_idx_mapping,
        token_ids,
        expanded_local_pos,
        repetition_penalty,
        frequency_penalty,
        presence_penalty,
        prompt_bin_mask,
        prompt_bin_mask.stride(0),
        output_bin_counts,
        output_bin_counts.stride(0),
        vocab_size,
        BLOCK_SIZE=BLOCK_SIZE,
    )
```
**EN:** This function implements `apply_penalties` within the module. Key calls include `cdiv`, `stride`.
**CN:** 该函数会实现 `apply_penalties`，其作用域位于the module。 关键调用包括 `cdiv`, `stride`。

### `_bincount_kernel` function / `_bincount_kernel` 函数
```python
@triton.jit
def _bincount_kernel(
    expanded_idx_mapping_ptr,
    all_token_ids_ptr,
    all_token_ids_stride,
    prompt_len_ptr,
    prefill_len_ptr,
    prompt_bin_mask_ptr,
    prompt_bin_mask_stride,
    output_bin_counts_ptr,
    output_bin_counts_stride,
    BLOCK_SIZE: tl.constexpr,
):
    token_idx = tl.program_id(0)
    block_idx = tl.program_id(1)
    req_state_idx = tl.load(expanded_idx_mapping_ptr + token_idx)

    prefill_len = tl.load(prefill_len_ptr + req_state_idx)
    if block_idx * BLOCK_SIZE >= prefill_len:
        return

    prompt_len = tl.load(prompt_len_ptr + req_state_idx)
    block = block_idx * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    if block_idx * BLOCK_SIZE < prompt_len:
        mask = block < prompt_len
        prompt_tokens = tl.load(
            all_token_ids_ptr + req_state_idx * all_token_ids_stride + block, mask=mask
        )
        idx = prompt_tokens // 32
        bit_idx = prompt_tokens % 32
        bit = tl.full((BLOCK_SIZE,), 1, tl.int32) << bit_idx
        tl.atomic_or(
            prompt_bin_mask_ptr + req_state_idx * prompt_bin_mask_stride + idx,
            bit,
            mask=mask,
        )

    if (block_idx + 1) * BLOCK_SIZE >= prompt_len:
        mask = block < prefill_len
        mask &= block >= prompt_len
        output_tokens = tl.load(
            all_token_ids_ptr + req_state_idx * all_token_ids_stride + block, mask=mask
        )
        tl.atomic_add(
            output_bin_counts_ptr
            + req_state_idx * output_bin_counts_stride
            + output_tokens,
            1,
            mask=mask,
        )
```
**EN:** This function implements `_bincount_kernel` within the module. Key calls include `program_id`, `load`, `arange`, `atomic_or`, `atomic_add`, `full`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_bincount_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `arange`, `atomic_or`, `atomic_add`, `full`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `bincount` function / `bincount` 函数
```python
def bincount(
    expanded_idx_mapping: torch.Tensor,
    all_token_ids: torch.Tensor,
    prompt_len: torch.Tensor,
    prefill_len: torch.Tensor,
    prompt_bin_mask: torch.Tensor,
    output_bin_counts: torch.Tensor,
    max_prefill_len: int,
) -> None:
    # Use index_fill_ instead of `tensor[idx] = 0` to avoid sync.
    idx_long = expanded_idx_mapping.long()
    prompt_bin_mask.index_fill_(0, idx_long, 0)
    output_bin_counts.index_fill_(0, idx_long, 0)
    num_tokens = expanded_idx_mapping.shape[0]
    BLOCK_SIZE = 1024
    num_blocks = triton.cdiv(max_prefill_len, BLOCK_SIZE)
    _bincount_kernel[(num_tokens, num_blocks)](
        expanded_idx_mapping,
        all_token_ids,
        all_token_ids.stride(0),
        prompt_len,
        prefill_len,
        prompt_bin_mask,
        prompt_bin_mask.stride(0),
        output_bin_counts,
        output_bin_counts.stride(0),
        BLOCK_SIZE=BLOCK_SIZE,
    )
```
**EN:** This function implements `bincount` within the module. Key calls include `long`, `index_fill_`, `cdiv`, `stride`.
**CN:** 该函数会实现 `bincount`，其作用域位于the module。 关键调用包括 `long`, `index_fill_`, `cdiv`, `stride`。

### `use_penalty` function / `use_penalty` 函数
```python
def use_penalty(sampling_params: SamplingParams) -> bool:
    return (
        sampling_params.repetition_penalty != 1.0
        or sampling_params.frequency_penalty != 0.0
        or sampling_params.presence_penalty != 0.0
    )
```
**EN:** This function implements `use_penalty` within the module.
**CN:** 该函数会实现 `use_penalty`，其作用域位于the module。

## Key Concepts / 关键概念
- `PenaltiesState`: central class or interface in this module. / `PenaltiesState`：本模块中的核心类或接口。
- `_penalties_kernel`: top-level helper or orchestration entry point. / `_penalties_kernel`：顶层辅助函数或编排入口。
- `apply_penalties`: top-level helper or orchestration entry point. / `apply_penalties`：顶层辅助函数或编排入口。
- `_bincount_kernel`: top-level helper or orchestration entry point. / `_bincount_kernel`：顶层辅助函数或编排入口。
- `bincount`: top-level helper or orchestration entry point. / `bincount`：顶层辅助函数或编排入口。
- `use_penalty`: top-level helper or orchestration entry point. / `use_penalty`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.sampling_params`, `vllm.triton_utils`, `vllm.utils.math_utils`, `vllm.utils.torch_utils`, `vllm.v1.worker.gpu.buffer_utils`, `vllm.v1.worker.gpu.states`
