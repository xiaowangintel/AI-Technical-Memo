# pooling_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/pool/pooling_runner.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `PoolingRunner` for the V1 `worker/gpu/pool` subsystem. / 为 V1 的 `worker/gpu/pool` 子系统实现 `PoolingRunner`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from typing import cast

import torch
import torch.nn as nn
import torch.nn.functional as F

from vllm.model_executor.models import VllmModelForPooling, is_pooling_model
from vllm.tasks import PoolingTask
from vllm.v1.worker.gpu.input_batch import InputBatch
from vllm.v1.worker.gpu.states import RequestState
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.model_executor.models`, `vllm.tasks`, `vllm.v1.worker.gpu.input_batch`, `vllm.v1.worker.gpu.states`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.model_executor.models`, `vllm.tasks`, `vllm.v1.worker.gpu.input_batch`, `vllm.v1.worker.gpu.states` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `PoolingRunner` class / `PoolingRunner` 类
```python
class PoolingRunner:
```
**EN:** Introduces the `PoolingRunner` class. Core methods include `__init__`, `get_supported_tasks`, `pool`, `dummy_pooler_run`.
**CN:** 这里定义 `PoolingRunner` 类。核心方法包括 `__init__`, `get_supported_tasks`, `pool`, `dummy_pooler_run`。

### `PoolingRunner.__init__` method / `PoolingRunner.__init__` 方法
```python
    def __init__(self, model: nn.Module):
        self.model = cast(VllmModelForPooling, model)
```
**EN:** This method initializes the object state within `PoolingRunner`. Key calls include `cast`. It touches state such as `model`.
**CN:** 该方法会初始化对象状态，其作用域位于`PoolingRunner`。 关键调用包括 `cast`。 它会读写 `model` 等状态。

### `PoolingRunner.get_supported_tasks` method / `PoolingRunner.get_supported_tasks` 方法
```python
    @staticmethod
    def get_supported_tasks(model: nn.Module) -> list[PoolingTask]:
        if not is_pooling_model(model):
            return []
        assert "embed" in model.pooler.get_supported_tasks()
        return ["embed"]
```
**EN:** This method returns or derives a value within `PoolingRunner`. Key calls include `is_pooling_model`, `get_supported_tasks`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`PoolingRunner`。 关键调用包括 `is_pooling_model`, `get_supported_tasks`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `PoolingRunner.pool` method / `PoolingRunner.pool` 方法
```python
    def pool(
        self,
        hidden_states: torch.Tensor,
        input_batch: InputBatch,
        req_states: RequestState,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        # TODO(woosuk): Support different types of pooling tasks.
        last_hidden_states = hidden_states[input_batch.logits_indices]
        # TODO(woosuk): Make normalization optional.
        last_hidden_states = F.normalize(last_hidden_states, p=2, dim=-1)

        prompt_len = req_states.prompt_len.gpu[input_batch.idx_mapping]
        is_valid = input_batch.seq_lens == prompt_len
        return last_hidden_states, is_valid
```
**EN:** This method implements `pool` within `PoolingRunner`. Key calls include `normalize`.
**CN:** 该方法会实现 `pool`，其作用域位于`PoolingRunner`。 关键调用包括 `normalize`。

### `PoolingRunner.dummy_pooler_run` method / `PoolingRunner.dummy_pooler_run` 方法
```python
    def dummy_pooler_run(self, hidden_states: torch.Tensor) -> None:
        F.normalize(hidden_states, p=2, dim=-1)
        return
```
**EN:** This method implements `dummy_pooler_run` within `PoolingRunner`. Key calls include `normalize`.
**CN:** 该方法会实现 `dummy_pooler_run`，其作用域位于`PoolingRunner`。 关键调用包括 `normalize`。

## Key Concepts / 关键概念
- `PoolingRunner`: central class or interface in this module. / `PoolingRunner`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.model_executor.models`, `vllm.tasks`, `vllm.v1.worker.gpu.input_batch`, `vllm.v1.worker.gpu.states`
