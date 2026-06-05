# logit_bias.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/sample/logit_bias.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `LogitBiasState`, `_bias_kernel`, `apply_logit_bias` for the V1 `worker/gpu/sample` subsystem. / 为 V1 的 `worker/gpu/sample` 子系统实现 `LogitBiasState`, `_bias_kernel`, `apply_logit_bias`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import numpy as np
import torch

from vllm.sampling_params import SamplingParams
from vllm.triton_utils import tl, triton
from vllm.v1.worker.gpu.buffer_utils import StagedWriteTensor, UvaBackedTensor

MAX_NUM_ALLOWED_TOKEN_IDS = 1024
MAX_NUM_LOGIT_BIAS_TOKENS = 1024
MAX_NUM_STOP_TOKEN_IDS = 128
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `MAX_NUM_ALLOWED_TOKEN_IDS`, `MAX_NUM_LOGIT_BIAS_TOKENS`, `MAX_NUM_STOP_TOKEN_IDS`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `MAX_NUM_ALLOWED_TOKEN_IDS`, `MAX_NUM_LOGIT_BIAS_TOKENS`, `MAX_NUM_STOP_TOKEN_IDS`。

### `LogitBiasState` class / `LogitBiasState` 类
```python
class LogitBiasState:
```
**EN:** Introduces the `LogitBiasState` class. Core methods include `__init__`, `add_request`, `apply_staged_writes`, `apply_logit_bias`.
**CN:** 这里定义 `LogitBiasState` 类。核心方法包括 `__init__`, `add_request`, `apply_staged_writes`, `apply_logit_bias`。

### `LogitBiasState.__init__` method / `LogitBiasState.__init__` 方法
```python
    def __init__(self, max_num_reqs: int, device: torch.device):
        self.max_num_reqs = max_num_reqs

        # Allowed token IDs.
        self.num_allowed_token_ids = UvaBackedTensor(
            self.max_num_reqs, dtype=torch.int32
        )
        self.allowed_token_ids = StagedWriteTensor(
            (self.max_num_reqs, MAX_NUM_ALLOWED_TOKEN_IDS),
            dtype=torch.int32,
            device=device,
        )
        # Logit bias.
        self.num_logit_bias = UvaBackedTensor(self.max_num_reqs, dtype=torch.int32)
        self.logit_bias_token_ids = StagedWriteTensor(
            (self.max_num_reqs, MAX_NUM_LOGIT_BIAS_TOKENS),
            dtype=torch.int32,
            device=device,
        )
        self.logit_bias = StagedWriteTensor(
            (self.max_num_reqs, MAX_NUM_LOGIT_BIAS_TOKENS),
            dtype=torch.float32,
            device=device,
        )
        # Min tokens.
        self.min_lens = UvaBackedTensor(self.max_num_reqs, dtype=torch.int32)
        self.num_stop_token_ids = UvaBackedTensor(self.max_num_reqs, dtype=torch.int32)
        self.stop_token_ids = StagedWriteTensor(
            (self.max_num_reqs, MAX_NUM_STOP_TOKEN_IDS),
            dtype=torch.int32,
            device=device,
        )

        # Using any of the above.
        self.use_logit_bias = np.zeros(max_num_reqs, dtype=bool)
```
**EN:** This method initializes the object state within `LogitBiasState`. Key calls include `UvaBackedTensor`, `StagedWriteTensor`, `zeros`. It touches state such as `max_num_reqs`, `num_allowed_token_ids`, `allowed_token_ids`, `num_logit_bias`, `logit_bias_token_ids`, `logit_bias`, `min_lens`, `num_stop_token_ids`.
**CN:** 该方法会初始化对象状态，其作用域位于`LogitBiasState`。 关键调用包括 `UvaBackedTensor`, `StagedWriteTensor`, `zeros`。 它会读写 `max_num_reqs`, `num_allowed_token_ids`, `allowed_token_ids`, `num_logit_bias`, `logit_bias_token_ids`, `logit_bias`, `min_lens`, `num_stop_token_ids` 等状态。

### `LogitBiasState.add_request` method / `LogitBiasState.add_request` 方法
```python
    def add_request(
        self, req_idx: int, prompt_len: int, sampling_params: SamplingParams
    ) -> None:
        # Using any logit bias.
        use_logit_bias = False

        # Allowed token IDs.
        allowed_token_ids = sampling_params.allowed_token_ids
        if allowed_token_ids:
            num_allowed_token_ids = len(allowed_token_ids)
            if num_allowed_token_ids > MAX_NUM_ALLOWED_TOKEN_IDS:
                raise ValueError(
                    f"Too many allowed token IDs: {num_allowed_token_ids}. "
                    f"The max size is {MAX_NUM_ALLOWED_TOKEN_IDS}."
                )
            self.num_allowed_token_ids.np[req_idx] = num_allowed_token_ids
            self.allowed_token_ids.stage_write(req_idx, 0, allowed_token_ids)
            use_logit_bias = True
        else:
            self.num_allowed_token_ids.np[req_idx] = 0

        # Logit bias.
        logit_bias = sampling_params.logit_bias
        if logit_bias:
            num_logit_bias = len(logit_bias)
            if num_logit_bias > MAX_NUM_LOGIT_BIAS_TOKENS:
                raise ValueError(
                    f"Too many logit bias tokens: {num_logit_bias}. "
                    f"The max size is {MAX_NUM_LOGIT_BIAS_TOKENS}."
                )
            self.num_logit_bias.np[req_idx] = num_logit_bias
            self.logit_bias_token_ids.stage_write(req_idx, 0, logit_bias.keys())
            self.logit_bias.stage_write(req_idx, 0, logit_bias.values())
            use_logit_bias = True
        else:
            self.num_logit_bias.np[req_idx] = 0

        # Min tokens.
        min_tokens = sampling_params.min_tokens
        min_len = prompt_len + min_tokens
        self.min_lens.np[req_idx] = min_len
        stop_token_ids = sampling_params.all_stop_token_ids
        if min_tokens > 0 and stop_token_ids:
            num_stop_token_ids = len(stop_token_ids)
            if num_stop_token_ids > MAX_NUM_STOP_TOKEN_IDS:
                raise ValueError(
                    f"Too many stop tokens: {num_stop_token_ids}. "
                    f"The max size is {MAX_NUM_STOP_TOKEN_IDS}."
                )
            self.num_stop_token_ids.np[req_idx] = num_stop_token_ids
            self.stop_token_ids.stage_write(req_idx, 0, stop_token_ids)
            use_logit_bias = True
        else:
            self.num_stop_token_ids.np[req_idx] = 0

        self.use_logit_bias[req_idx] = use_logit_bias
```
**EN:** This method implements `add_request` within `LogitBiasState`. Key calls include `len`, `stage_write`, `ValueError`, `keys`, `values`. It touches state such as `min_lens`, `use_logit_bias`, `num_allowed_token_ids`, `num_logit_bias`, `num_stop_token_ids`. The control flow contains 6 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `add_request`，其作用域位于`LogitBiasState`。 关键调用包括 `len`, `stage_write`, `ValueError`, `keys`, `values`。 它会读写 `min_lens`, `use_logit_bias`, `num_allowed_token_ids`, `num_logit_bias`, `num_stop_token_ids` 等状态。 控制流包含 6 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `LogitBiasState.apply_staged_writes` method / `LogitBiasState.apply_staged_writes` 方法
```python
    def apply_staged_writes(self) -> None:
        self.num_allowed_token_ids.copy_to_uva()
        self.allowed_token_ids.apply_write()

        self.num_logit_bias.copy_to_uva()
        self.logit_bias_token_ids.apply_write()
        self.logit_bias.apply_write()

        self.min_lens.copy_to_uva()
        self.num_stop_token_ids.copy_to_uva()
        self.stop_token_ids.apply_write()
```
**EN:** This method implements `apply_staged_writes` within `LogitBiasState`. Key calls include `copy_to_uva`, `apply_write`.
**CN:** 该方法会实现 `apply_staged_writes`，其作用域位于`LogitBiasState`。 关键调用包括 `copy_to_uva`, `apply_write`。

### `LogitBiasState.apply_logit_bias` method / `LogitBiasState.apply_logit_bias` 方法
```python
    def apply_logit_bias(
        self,
        logits: torch.Tensor,
        expanded_idx_mapping: torch.Tensor,
        idx_mapping_np: np.ndarray,
        pos: torch.Tensor,
    ) -> None:
        if not np.any(self.use_logit_bias[idx_mapping_np]):
            # No request uses logit bias. Skip the kernel launch.
            return

        apply_logit_bias(
            logits,
            expanded_idx_mapping,
            pos,
            self.num_allowed_token_ids.gpu,
            self.allowed_token_ids.gpu,
            self.num_logit_bias.gpu,
            self.logit_bias_token_ids.gpu,
            self.logit_bias.gpu,
            self.min_lens.gpu,
            self.num_stop_token_ids.gpu,
            self.stop_token_ids.gpu,
        )
```
**EN:** This method implements `apply_logit_bias` within `LogitBiasState`. Key calls include `apply_logit_bias`, `any`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `apply_logit_bias`，其作用域位于`LogitBiasState`。 关键调用包括 `apply_logit_bias`, `any`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_bias_kernel` function / `_bias_kernel` 函数
```python
@triton.jit
def _bias_kernel(
    logits_ptr,
    logits_stride,
    vocab_size,
    expanded_idx_mapping_ptr,
    # Allowed token IDs.
    num_allowed_token_ids_ptr,
    allowed_token_ids_ptr,
    allowed_token_ids_stride,
    # Logit bias.
    num_logit_bias_ptr,
    bias_token_ids_ptr,
    bias_token_ids_stride,
    bias_ptr,
    bias_stride,
    # Min tokens.
    pos_ptr,
    min_lens_ptr,
    num_stop_token_ids_ptr,
    stop_token_ids_ptr,
    stop_token_ids_stride,
    BLOCK_SIZE: tl.constexpr,
    LOGITS_BLOCK_SIZE: tl.constexpr,
):
    token_idx = tl.program_id(0)
    req_state_idx = tl.load(expanded_idx_mapping_ptr + token_idx)

    block = tl.arange(0, BLOCK_SIZE)

    # Allowed token IDs.
    num_allowed_token_ids = tl.load(num_allowed_token_ids_ptr + req_state_idx)
    if num_allowed_token_ids > 0:
        block = tl.arange(0, BLOCK_SIZE)
        mask = block < num_allowed_token_ids

        # Save logits for allowed token IDs.
        allowed_token_ids = tl.load(
            allowed_token_ids_ptr + req_state_idx * allowed_token_ids_stride + block,
            mask=mask,
        )
        logits = tl.load(
            logits_ptr + token_idx * logits_stride + allowed_token_ids, mask=mask
        )

        # Set logits to -inf for all tokens.
        for i in range(0, vocab_size, LOGITS_BLOCK_SIZE):
            offset = i + tl.arange(0, LOGITS_BLOCK_SIZE)
            tl.store(
                logits_ptr + token_idx * logits_stride + offset,
                -float("inf"),
                mask=offset < vocab_size,
            )

        # Restore logits for allowed token IDs.
        tl.store(
            logits_ptr + token_idx * logits_stride + allowed_token_ids,
            logits,
            mask=mask,
        )

    # Logit bias.
    num_logit_bias = tl.load(num_logit_bias_ptr + req_state_idx)
    if num_logit_bias > 0:
        mask = block < num_logit_bias
        token_ids = tl.load(
            bias_token_ids_ptr + req_state_idx * bias_token_ids_stride + block,
            mask=mask,
        )
        bias = tl.load(bias_ptr + req_state_idx * bias_stride + block, mask=mask)
        logits = tl.load(logits_ptr + token_idx * logits_stride + token_ids, mask=mask)
        logits += bias
        tl.store(logits_ptr + token_idx * logits_stride + token_ids, logits, mask=mask)

    # Apply min tokens.
    num_stop_token_ids = tl.load(num_stop_token_ids_ptr + req_state_idx)
    pos = tl.load(pos_ptr + token_idx)
    min_len = tl.load(min_lens_ptr + req_state_idx)
    if num_stop_token_ids > 0 and pos < min_len:
        mask = block < num_stop_token_ids
        stop_token_ids = tl.load(
            stop_token_ids_ptr + req_state_idx * stop_token_ids_stride + block,
            mask=mask,
        )
        tl.store(
            logits_ptr + token_idx * logits_stride + stop_token_ids,
            -float("inf"),
            mask=mask,
        )
```
**EN:** This function implements `_bias_kernel` within the module. Key calls include `program_id`, `load`, `arange`, `range`, `store`, `float`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_bias_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `arange`, `range`, `store`, `float`。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `apply_logit_bias` function / `apply_logit_bias` 函数
```python
def apply_logit_bias(
    logits: torch.Tensor,
    expanded_idx_mapping: torch.Tensor,
    pos: torch.Tensor,
    num_allowed_token_ids: torch.Tensor,
    allowed_token_ids: torch.Tensor,
    num_logit_bias: torch.Tensor,
    logit_bias_token_ids: torch.Tensor,
    logit_bias: torch.Tensor,
    min_lens: torch.Tensor,
    num_stop_token_ids: torch.Tensor,
    stop_token_ids: torch.Tensor,
) -> None:
    num_tokens, vocab_size = logits.shape
    BLOCK_SIZE = triton.next_power_of_2(
        max(
            allowed_token_ids.shape[-1],
            logit_bias_token_ids.shape[-1],
            stop_token_ids.shape[-1],
        )
    )
    LOGITS_BLOCK_SIZE = 8192
    _bias_kernel[(num_tokens,)](
        logits,
        logits.stride(0),
        vocab_size,
        expanded_idx_mapping,
        num_allowed_token_ids,
        allowed_token_ids,
        allowed_token_ids.stride(0),
        num_logit_bias,
        logit_bias_token_ids,
        logit_bias_token_ids.stride(0),
        logit_bias,
        logit_bias.stride(0),
        pos,
        min_lens,
        num_stop_token_ids,
        stop_token_ids,
        stop_token_ids.stride(0),
        BLOCK_SIZE=BLOCK_SIZE,
        LOGITS_BLOCK_SIZE=LOGITS_BLOCK_SIZE,
    )
```
**EN:** This function implements `apply_logit_bias` within the module. Key calls include `next_power_of_2`, `max`, `stride`.
**CN:** 该函数会实现 `apply_logit_bias`，其作用域位于the module。 关键调用包括 `next_power_of_2`, `max`, `stride`。

## Key Concepts / 关键概念
- `LogitBiasState`: central class or interface in this module. / `LogitBiasState`：本模块中的核心类或接口。
- `_bias_kernel`: top-level helper or orchestration entry point. / `_bias_kernel`：顶层辅助函数或编排入口。
- `apply_logit_bias`: top-level helper or orchestration entry point. / `apply_logit_bias`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.sampling_params`, `vllm.triton_utils`, `vllm.v1.worker.gpu.buffer_utils`
