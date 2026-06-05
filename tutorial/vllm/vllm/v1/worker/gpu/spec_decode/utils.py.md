# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/spec_decode/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `DraftTokensHandler` for the V1 `worker/gpu/spec_decode` subsystem. / 为 V1 的 `worker/gpu/spec_decode` 子系统实现 `DraftTokensHandler`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import numpy as np
import torch

from vllm.v1.outputs import DraftTokenIds
from vllm.v1.worker.gpu.async_utils import async_copy_to_np
from vllm.v1.worker.gpu.input_batch import InputBatch
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`, and internal vLLM modules such as `vllm.v1.outputs`, `vllm.v1.worker.gpu.async_utils`, `vllm.v1.worker.gpu.input_batch`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch` 等外部依赖，以及 `vllm.v1.outputs`, `vllm.v1.worker.gpu.async_utils`, `vllm.v1.worker.gpu.input_batch` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `DraftTokensHandler` class / `DraftTokensHandler` 类
```python
class DraftTokensHandler:
```
**EN:** Introduces the `DraftTokensHandler` class. Core methods include `__init__`, `set_draft_tokens`, `get_draft_tokens`.
**CN:** 这里定义 `DraftTokensHandler` 类。核心方法包括 `__init__`, `set_draft_tokens`, `get_draft_tokens`。

### `DraftTokensHandler.__init__` method / `DraftTokensHandler.__init__` 方法
```python
    def __init__(self, device: torch.device | None = None):
        self.device = device
        self.copy_stream = torch.cuda.Stream(device)
        self.copy_event = torch.cuda.Event()

        self.req_ids: list[str] = []
        self.draft_tokens_np: np.ndarray | None = None
        self.num_draft_tokens: int = 0
```
**EN:** This method initializes the object state within `DraftTokensHandler`. Key calls include `Stream`, `Event`. It touches state such as `device`, `copy_stream`, `copy_event`, `req_ids`, `draft_tokens_np`, `num_draft_tokens`.
**CN:** 该方法会初始化对象状态，其作用域位于`DraftTokensHandler`。 关键调用包括 `Stream`, `Event`。 它会读写 `device`, `copy_stream`, `copy_event`, `req_ids`, `draft_tokens_np`, `num_draft_tokens` 等状态。

### `DraftTokensHandler.set_draft_tokens` method / `DraftTokensHandler.set_draft_tokens` 方法
```python
    def set_draft_tokens(
        self, input_batch: InputBatch, draft_tokens: torch.Tensor
    ) -> None:
        self.req_ids = input_batch.req_ids
        self.num_draft_tokens = draft_tokens.shape[1]
        if not input_batch.has_structured_output_reqs:
            # No draft token validation needs to be performed by
            # the scheduler for this batch.
            self.draft_tokens_np = None
            return

        # For spec decoding + structured outputs, we must transfer the
        # draft tokens back to the scheduler for grammar validation.
        current_stream = torch.cuda.current_stream(self.device)
        self.copy_stream.wait_stream(current_stream)
        with torch.cuda.stream(self.copy_stream):
            self.draft_tokens_np = async_copy_to_np(draft_tokens)
            self.copy_event.record()
```
**EN:** This method stores a value into object state within `DraftTokensHandler`. Key calls include `current_stream`, `wait_stream`, `stream`, `async_copy_to_np`, `record`. It touches state such as `req_ids`, `num_draft_tokens`, `draft_tokens_np`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会将值写入对象状态，其作用域位于`DraftTokensHandler`。 关键调用包括 `current_stream`, `wait_stream`, `stream`, `async_copy_to_np`, `record`。 它会读写 `req_ids`, `num_draft_tokens`, `draft_tokens_np` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DraftTokensHandler.get_draft_tokens` method / `DraftTokensHandler.get_draft_tokens` 方法
```python
    def get_draft_tokens(self) -> DraftTokenIds | None:
        if self.draft_tokens_np is not None:
            self.copy_event.synchronize()
            draft_token_ids = self.draft_tokens_np.tolist()
        else:
            # This case only happens when async scheduling is disabled.
            draft_token_ids = [[-1] * self.num_draft_tokens for _ in self.req_ids]
        return DraftTokenIds(self.req_ids, draft_token_ids)
```
**EN:** This method returns or derives a value within `DraftTokensHandler`. Key calls include `DraftTokenIds`, `synchronize`, `tolist`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`DraftTokensHandler`。 关键调用包括 `DraftTokenIds`, `synchronize`, `tolist`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `DraftTokensHandler`: central class or interface in this module. / `DraftTokensHandler`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.v1.outputs`, `vllm.v1.worker.gpu.async_utils`, `vllm.v1.worker.gpu.input_batch`
