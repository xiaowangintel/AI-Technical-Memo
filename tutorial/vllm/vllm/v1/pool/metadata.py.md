# metadata.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/pool/metadata.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `PoolingCursor`, `PoolingStates`, `PoolingMetadata` for the V1 `pool` subsystem. / 为 V1 的 `pool` 子系统实现 `PoolingCursor`, `PoolingStates`, `PoolingMetadata`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from dataclasses import dataclass

import numpy as np
import torch

from vllm.pooling_params import PoolingParams
from vllm.tasks import PoolingTask
from vllm.utils.platform_utils import is_pin_memory_available

pin_memory = is_pin_memory_available()
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `pin_memory`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `pin_memory`。

### `PoolingCursor` class / `PoolingCursor` 类
```python
@dataclass
class PoolingCursor:
    first_token_indices_gpu: torch.Tensor
    last_token_indices_gpu: torch.Tensor
    prompt_lens_cpu: torch.Tensor
    seq_lens_cpu: torch.Tensor
    num_scheduled_tokens_cpu: torch.Tensor
```
**EN:** Uses `@dataclass` to package related state for `PoolingCursor`. Typical fields include `first_token_indices_gpu`, `last_token_indices_gpu`, `prompt_lens_cpu`, `seq_lens_cpu`, `num_scheduled_tokens_cpu`.
**CN:** `PoolingCursor` 使用 `@dataclass` 打包相关状态。典型字段包括 `first_token_indices_gpu`, `last_token_indices_gpu`, `prompt_lens_cpu`, `seq_lens_cpu`, `num_scheduled_tokens_cpu`。

### `PoolingCursor.__getitem__` method / `PoolingCursor.__getitem__` 方法
```python
    def __getitem__(self, indices: slice):
        return PoolingCursor(
            first_token_indices_gpu=self.first_token_indices_gpu[indices],
            last_token_indices_gpu=self.last_token_indices_gpu[indices],
            prompt_lens_cpu=self.prompt_lens_cpu[indices],
            seq_lens_cpu=self.seq_lens_cpu[indices],
            num_scheduled_tokens_cpu=self.num_scheduled_tokens_cpu[indices],
        )
```
**EN:** This method implements `__getitem__` within `PoolingCursor`. Key calls include `PoolingCursor`.
**CN:** 该方法会实现 `__getitem__`，其作用域位于`PoolingCursor`。 关键调用包括 `PoolingCursor`。

### `PoolingCursor.is_partial_prefill` method / `PoolingCursor.is_partial_prefill` 方法
```python
    def is_partial_prefill(self):
        return not torch.all(self.prompt_lens_cpu == self.num_scheduled_tokens_cpu)
```
**EN:** This method answers a boolean capability check within `PoolingCursor`. Key calls include `all`.
**CN:** 该方法会回答布尔能力判断，其作用域位于`PoolingCursor`。 关键调用包括 `all`。

### `PoolingCursor.is_finished` method / `PoolingCursor.is_finished` 方法
```python
    def is_finished(self):
        return self.prompt_lens_cpu == self.seq_lens_cpu
```
**EN:** This method answers a boolean capability check within `PoolingCursor`.
**CN:** 该方法会回答布尔能力判断，其作用域位于`PoolingCursor`。

### `PoolingStates` class / `PoolingStates` 类
```python
class PoolingStates:
```
**EN:** Introduces the `PoolingStates` class. Core methods include `__init__`, `clean`.
**CN:** 这里定义 `PoolingStates` 类。核心方法包括 `__init__`, `clean`。

### `PoolingStates.__init__` method / `PoolingStates.__init__` 方法
```python
    def __init__(self):
        # for chunked prefill with ALL pooling
        self.hidden_states_cache: list[torch.Tensor] = []
```
**EN:** This method initializes the object state within `PoolingStates`. It touches state such as `hidden_states_cache`.
**CN:** 该方法会初始化对象状态，其作用域位于`PoolingStates`。 它会读写 `hidden_states_cache` 等状态。

### `PoolingStates.clean` method / `PoolingStates.clean` 方法
```python
    def clean(self):
        self.hidden_states_cache.clear()
```
**EN:** This method implements `clean` within `PoolingStates`. Key calls include `clear`.
**CN:** 该方法会实现 `clean`，其作用域位于`PoolingStates`。 关键调用包括 `clear`。

### `PoolingMetadata` class / `PoolingMetadata` 类
```python
@dataclass
class PoolingMetadata:
    """Tensors for pooling."""

    prompt_lens: torch.Tensor  # CPU Tensor
    prompt_token_ids: torch.Tensor | None  # Model-device tensor
    prompt_token_ids_cpu: torch.Tensor | None  # CPU tensor
    pooling_params: list[PoolingParams]
    pooling_states: list[PoolingStates]
    pooling_cursor: PoolingCursor | None = None
```
**EN:** Uses `@dataclass` to package related state for `PoolingMetadata`. Typical fields include `prompt_lens`, `prompt_token_ids`, `prompt_token_ids_cpu`, `pooling_params`, `pooling_states`, `pooling_cursor`.
**CN:** `PoolingMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `prompt_lens`, `prompt_token_ids`, `prompt_token_ids_cpu`, `pooling_params`, `pooling_states`, `pooling_cursor`。

### `PoolingMetadata.__post_init__` method / `PoolingMetadata.__post_init__` 方法
```python
    def __post_init__(self) -> None:
        pooling_params = self.pooling_params

        tasks: list[PoolingTask] = [
            task
            for pooling_param in pooling_params
            if (task := pooling_param.task) is not None
        ]
        assert len(pooling_params) == len(tasks)

        self.tasks = tasks
```
**EN:** This method implements `__post_init__` within `PoolingMetadata`. Key calls include `len`. It touches state such as `tasks`.
**CN:** 该方法会实现 `__post_init__`，其作用域位于`PoolingMetadata`。 关键调用包括 `len`。 它会读写 `tasks` 等状态。

### `PoolingMetadata.__getitem__` method / `PoolingMetadata.__getitem__` 方法
```python
    def __getitem__(self, indices: slice):
        return PoolingMetadata(
            prompt_lens=self.prompt_lens[indices],
            prompt_token_ids=None
            if self.prompt_token_ids is None
            else self.prompt_token_ids[indices],
            prompt_token_ids_cpu=None
            if self.prompt_token_ids_cpu is None
            else self.prompt_token_ids_cpu[indices],
            pooling_params=self.pooling_params[indices],
            pooling_states=self.pooling_states[indices],
            pooling_cursor=None
            if self.pooling_cursor is None
            else self.pooling_cursor[indices],
        )
```
**EN:** This method implements `__getitem__` within `PoolingMetadata`. Key calls include `PoolingMetadata`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `__getitem__`，其作用域位于`PoolingMetadata`。 关键调用包括 `PoolingMetadata`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `PoolingMetadata._get_prompt_token_ids` method / `PoolingMetadata._get_prompt_token_ids` 方法
```python
    def _get_prompt_token_ids(
        self,
        prompt_token_ids: torch.Tensor | None,
    ) -> list[torch.Tensor]:
        assert prompt_token_ids is not None, (
            "Please set `requires_token_ids=True` in `get_pooling_updates`"
        )
        return [prompt_token_ids[i, :num] for i, num in enumerate(self.prompt_lens)]
```
**EN:** This method implements `_get_prompt_token_ids` within `PoolingMetadata`. Key calls include `enumerate`.
**CN:** 该方法会实现 `_get_prompt_token_ids`，其作用域位于`PoolingMetadata`。 关键调用包括 `enumerate`。

### `PoolingMetadata.get_prompt_token_ids` method / `PoolingMetadata.get_prompt_token_ids` 方法
```python
    def get_prompt_token_ids(self) -> list[torch.Tensor]:
        return self._get_prompt_token_ids(self.prompt_token_ids)
```
**EN:** This method returns or derives a value within `PoolingMetadata`. Key calls include `_get_prompt_token_ids`.
**CN:** 该方法会返回或推导一个值，其作用域位于`PoolingMetadata`。 关键调用包括 `_get_prompt_token_ids`。

### `PoolingMetadata.get_prompt_token_ids_cpu` method / `PoolingMetadata.get_prompt_token_ids_cpu` 方法
```python
    def get_prompt_token_ids_cpu(self) -> list[torch.Tensor]:
        return self._get_prompt_token_ids(self.prompt_token_ids_cpu)
```
**EN:** This method returns or derives a value within `PoolingMetadata`. Key calls include `_get_prompt_token_ids`.
**CN:** 该方法会返回或推导一个值，其作用域位于`PoolingMetadata`。 关键调用包括 `_get_prompt_token_ids`。

### `PoolingMetadata.get_pooling_cursor` method / `PoolingMetadata.get_pooling_cursor` 方法
```python
    def get_pooling_cursor(self) -> PoolingCursor:
        pooling_cursor = self.pooling_cursor
        assert pooling_cursor is not None, "Should call `build_pooling_cursor` first"

        return pooling_cursor
```
**EN:** This method returns or derives a value within `PoolingMetadata`.
**CN:** 该方法会返回或推导一个值，其作用域位于`PoolingMetadata`。

### `PoolingMetadata.build_pooling_cursor` method / `PoolingMetadata.build_pooling_cursor` 方法
```python
    def build_pooling_cursor(
        self,
        num_scheduled_tokens_np: np.ndarray,
        seq_lens_cpu: torch.Tensor,
        device: torch.device,
        query_start_loc_gpu: torch.Tensor | None = None,
    ):
        n_seq = len(num_scheduled_tokens_np)
        prompt_lens = self.prompt_lens

        assert len(prompt_lens) == n_seq

        num_scheduled_tokens_cpu = torch.from_numpy(num_scheduled_tokens_np)
        if query_start_loc_gpu is None:
            cumsum = torch.zeros(
                n_seq + 1, dtype=torch.int64, pin_memory=pin_memory, device="cpu"
            )
            torch.cumsum(num_scheduled_tokens_cpu, dim=0, out=cumsum[1:])
            cumsum = cumsum.to(device, non_blocking=True)
        else:
            if query_start_loc_gpu.shape[0] != n_seq + 1:
                raise ValueError(
                    "query_start_loc_gpu length does not match "
                    f"the number of sequences: {query_start_loc_gpu.shape[0]} "
                    f"!= {n_seq + 1}."
                )
            if query_start_loc_gpu.device != device:
                raise ValueError(
                    "query_start_loc_gpu must be on the same device as the "
                    f"hidden states: {query_start_loc_gpu.device} != {device}."
                )
            cumsum = query_start_loc_gpu
        self.pooling_cursor = PoolingCursor(
            first_token_indices_gpu=cumsum[:n_seq],
            last_token_indices_gpu=cumsum[1:] - 1,
            prompt_lens_cpu=prompt_lens,
            seq_lens_cpu=seq_lens_cpu,
            num_scheduled_tokens_cpu=num_scheduled_tokens_cpu,
        )
```
**EN:** This method builds derived structures within `PoolingMetadata`. Key calls include `len`, `from_numpy`, `PoolingCursor`, `zeros`, `cumsum`, `to`. It touches state such as `pooling_cursor`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会构建派生结构，其作用域位于`PoolingMetadata`。 关键调用包括 `len`, `from_numpy`, `PoolingCursor`, `zeros`, `cumsum`, `to`。 它会读写 `pooling_cursor` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `PoolingCursor`: central class or interface in this module. / `PoolingCursor`：本模块中的核心类或接口。
- `PoolingStates`: central class or interface in this module. / `PoolingStates`：本模块中的核心类或接口。
- `PoolingMetadata`: central class or interface in this module. / `PoolingMetadata`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.pooling_params`, `vllm.tasks`, `vllm.utils.platform_utils`
