# states.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/sample/states.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `SamplingStates` for the V1 `worker/gpu/sample` subsystem. / 为 V1 的 `worker/gpu/sample` 子系统实现 `SamplingStates`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import numpy as np
import torch

from vllm.sampling_params import SamplingParams
from vllm.v1.sample.ops.topk_topp_sampler import apply_top_k_top_p
from vllm.v1.worker.gpu.buffer_utils import UvaBackedTensor
from vllm.v1.worker.gpu.sample.gumbel import apply_temperature
from vllm.v1.worker.gpu.sample.min_p import apply_min_p

NO_LOGPROBS = -1
_NP_INT64_MIN = np.iinfo(np.int64).min
_NP_INT64_MAX = np.iinfo(np.int64).max
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `NO_LOGPROBS`, `_NP_INT64_MIN`, `_NP_INT64_MAX`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `NO_LOGPROBS`, `_NP_INT64_MIN`, `_NP_INT64_MAX`。

### `SamplingStates` class / `SamplingStates` 类
```python
class SamplingStates:
```
**EN:** Introduces the `SamplingStates` class. Core methods include `__init__`, `add_request`, `apply_staged_writes`, `apply_temperature`, `apply_min_p`, `apply_top_k_top_p`.
**CN:** 这里定义 `SamplingStates` 类。核心方法包括 `__init__`, `add_request`, `apply_staged_writes`, `apply_temperature`, `apply_min_p`, `apply_top_k_top_p`。

### `SamplingStates.__init__` method / `SamplingStates.__init__` 方法
```python
    def __init__(self, max_num_reqs: int, vocab_size: int):
        self.max_num_reqs = max_num_reqs
        self.vocab_size = vocab_size

        self.temperature = UvaBackedTensor(max_num_reqs, dtype=torch.float32)
        self.top_k = UvaBackedTensor(max_num_reqs, dtype=torch.int32)
        self.top_p = UvaBackedTensor(max_num_reqs, dtype=torch.float32)
        self.min_p = UvaBackedTensor(max_num_reqs, dtype=torch.float32)
        self.seeds = UvaBackedTensor(max_num_reqs, dtype=torch.int64)

        # Initialize top_k and top_p manually because 0 is an invalid value for them.
        self.top_k.np.fill(self.vocab_size)
        self.top_k.copy_to_uva()
        self.top_p.np.fill(1.0)
        self.top_p.copy_to_uva()

        self.num_logprobs = np.empty(self.max_num_reqs, dtype=np.int32)
        # -1 means no logprobs are requested.
        self.num_logprobs.fill(NO_LOGPROBS)
```
**EN:** This method initializes the object state within `SamplingStates`. Key calls include `UvaBackedTensor`, `fill`, `copy_to_uva`, `empty`. It touches state such as `max_num_reqs`, `vocab_size`, `temperature`, `top_k`, `top_p`, `min_p`, `seeds`, `num_logprobs`.
**CN:** 该方法会初始化对象状态，其作用域位于`SamplingStates`。 关键调用包括 `UvaBackedTensor`, `fill`, `copy_to_uva`, `empty`。 它会读写 `max_num_reqs`, `vocab_size`, `temperature`, `top_k`, `top_p`, `min_p`, `seeds`, `num_logprobs` 等状态。

### `SamplingStates.add_request` method / `SamplingStates.add_request` 方法
```python
    def add_request(self, req_idx: int, sampling_params: SamplingParams) -> None:
        self.temperature.np[req_idx] = sampling_params.temperature
        self.top_p.np[req_idx] = sampling_params.top_p
        top_k = sampling_params.top_k
        if top_k <= 0 or top_k > self.vocab_size:
            top_k = self.vocab_size
        self.top_k.np[req_idx] = top_k
        self.min_p.np[req_idx] = sampling_params.min_p

        seed = sampling_params.seed
        if seed is None:
            seed = np.random.randint(_NP_INT64_MIN, _NP_INT64_MAX)
        self.seeds.np[req_idx] = seed

        num_logprobs = sampling_params.logprobs
        if num_logprobs is None:
            num_logprobs = NO_LOGPROBS
        self.num_logprobs[req_idx] = num_logprobs
```
**EN:** This method implements `add_request` within `SamplingStates`. Key calls include `randint`. It touches state such as `temperature`, `top_p`, `top_k`, `min_p`, `seeds`, `num_logprobs`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `add_request`，其作用域位于`SamplingStates`。 关键调用包括 `randint`。 它会读写 `temperature`, `top_p`, `top_k`, `min_p`, `seeds`, `num_logprobs` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `SamplingStates.apply_staged_writes` method / `SamplingStates.apply_staged_writes` 方法
```python
    def apply_staged_writes(self) -> None:
        self.temperature.copy_to_uva()
        self.top_p.copy_to_uva()
        self.top_k.copy_to_uva()
        self.min_p.copy_to_uva()
        self.seeds.copy_to_uva()
```
**EN:** This method implements `apply_staged_writes` within `SamplingStates`. Key calls include `copy_to_uva`.
**CN:** 该方法会实现 `apply_staged_writes`，其作用域位于`SamplingStates`。 关键调用包括 `copy_to_uva`。

### `SamplingStates.apply_temperature` method / `SamplingStates.apply_temperature` 方法
```python
    def apply_temperature(
        self,
        logits: torch.Tensor,
        expanded_idx_mapping: torch.Tensor,
        idx_mapping_np: np.ndarray,
    ) -> None:
        temp_np = self.temperature.np[idx_mapping_np]
        if np.all((temp_np == 0.0) | (temp_np == 1.0)):
            # No request requires temperature. Skip the kernel launch.
            return

        apply_temperature(logits, expanded_idx_mapping, self.temperature.gpu)
```
**EN:** This method implements `apply_temperature` within `SamplingStates`. Key calls include `all`, `apply_temperature`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `apply_temperature`，其作用域位于`SamplingStates`。 关键调用包括 `all`, `apply_temperature`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `SamplingStates.apply_min_p` method / `SamplingStates.apply_min_p` 方法
```python
    def apply_min_p(
        self,
        logits: torch.Tensor,
        expanded_idx_mapping: torch.Tensor,
        idx_mapping_np: np.ndarray,
    ) -> None:
        if np.all(self.min_p.np[idx_mapping_np] == 0.0):
            # No request uses min_p. Skip the kernel launch.
            return
        apply_min_p(logits, expanded_idx_mapping, self.min_p.gpu)
```
**EN:** This method implements `apply_min_p` within `SamplingStates`. Key calls include `all`, `apply_min_p`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `apply_min_p`，其作用域位于`SamplingStates`。 关键调用包括 `all`, `apply_min_p`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `SamplingStates.apply_top_k_top_p` method / `SamplingStates.apply_top_k_top_p` 方法
```python
    def apply_top_k_top_p(
        self,
        logits: torch.Tensor,
        expanded_idx_mapping: torch.Tensor,
        idx_mapping_np: np.ndarray,
    ) -> torch.Tensor:
        do_top_k = np.any(self.top_k.np[idx_mapping_np] != self.vocab_size)
        do_top_p = np.any(self.top_p.np[idx_mapping_np] != 1.0)
        if not (do_top_k or do_top_p):
            return logits

        top_k = self.top_k.gpu[expanded_idx_mapping] if do_top_k else None
        top_p = self.top_p.gpu[expanded_idx_mapping] if do_top_p else None
        return apply_top_k_top_p(logits, top_k, top_p)
```
**EN:** This method implements `apply_top_k_top_p` within `SamplingStates`. Key calls include `any`, `apply_top_k_top_p`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `apply_top_k_top_p`，其作用域位于`SamplingStates`。 关键调用包括 `any`, `apply_top_k_top_p`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `SamplingStates.max_num_logprobs` method / `SamplingStates.max_num_logprobs` 方法
```python
    def max_num_logprobs(self, idx_mapping_np: np.ndarray) -> int:
        return int(np.max(self.num_logprobs[idx_mapping_np]))
```
**EN:** This method implements `max_num_logprobs` within `SamplingStates`. Key calls include `int`, `max`.
**CN:** 该方法会实现 `max_num_logprobs`，其作用域位于`SamplingStates`。 关键调用包括 `int`, `max`。

## Key Concepts / 关键概念
- `SamplingStates`: central class or interface in this module. / `SamplingStates`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.sampling_params`, `vllm.v1.sample.ops.topk_topp_sampler`, `vllm.v1.worker.gpu.buffer_utils`, `vllm.v1.worker.gpu.sample.gumbel`, `vllm.v1.worker.gpu.sample.min_p`
