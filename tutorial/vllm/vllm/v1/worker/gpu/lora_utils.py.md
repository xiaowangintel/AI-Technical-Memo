# lora_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/lora_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `LoraState` for the V1 `worker/gpu` subsystem. / 为 V1 的 `worker/gpu` 子系统实现 `LoraState`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import numpy as np

from vllm.lora.request import LoRARequest

NO_LORA_ID = 0
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `NO_LORA_ID`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `NO_LORA_ID`。

### `LoraState` class / `LoraState` 类
```python
class LoraState:
```
**EN:** Introduces the `LoraState` class. Core methods include `__init__`, `add_request`, `remove_request`, `make_lora_inputs`.
**CN:** 这里定义 `LoraState` 类。核心方法包括 `__init__`, `add_request`, `remove_request`, `make_lora_inputs`。

### `LoraState.__init__` method / `LoraState.__init__` 方法
```python
    def __init__(self, max_num_reqs: int):
        self.lora_ids = np.zeros(max_num_reqs, dtype=np.int32)
        self.lora_ids.fill(NO_LORA_ID)
        # req_id -> lora_request
        self.lora_requests: dict[str, LoRARequest] = {}
```
**EN:** This method initializes the object state within `LoraState`. Key calls include `zeros`, `fill`. It touches state such as `lora_ids`, `lora_requests`.
**CN:** 该方法会初始化对象状态，其作用域位于`LoraState`。 关键调用包括 `zeros`, `fill`。 它会读写 `lora_ids`, `lora_requests` 等状态。

### `LoraState.add_request` method / `LoraState.add_request` 方法
```python
    def add_request(
        self, req_id: str, req_index: int, lora_request: LoRARequest | None
    ) -> None:
        if lora_request is not None:
            self.lora_requests[req_id] = lora_request
            self.lora_ids[req_index] = lora_request.lora_int_id
        else:
            self.lora_ids[req_index] = NO_LORA_ID
```
**EN:** This method implements `add_request` within `LoraState`. It touches state such as `lora_requests`, `lora_ids`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `add_request`，其作用域位于`LoraState`。 它会读写 `lora_requests`, `lora_ids` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `LoraState.remove_request` method / `LoraState.remove_request` 方法
```python
    def remove_request(self, req_id: str) -> None:
        self.lora_requests.pop(req_id, None)
```
**EN:** This method implements `remove_request` within `LoraState`. Key calls include `pop`.
**CN:** 该方法会实现 `remove_request`，其作用域位于`LoraState`。 关键调用包括 `pop`。

### `LoraState.make_lora_inputs` method / `LoraState.make_lora_inputs` 方法
```python
    def make_lora_inputs(
        self,
        req_ids: list[str],
        idx_mapping: np.ndarray,
        num_scheduled_tokens: np.ndarray,
    ) -> tuple[tuple[int, ...], tuple[int, ...], set[LoRARequest]]:
        lora_ids = self.lora_ids[idx_mapping]
        prompt_lora_mapping = tuple(lora_ids)
        token_lora_mapping = tuple(lora_ids.repeat(num_scheduled_tokens))

        active_lora_requests: set[LoRARequest] = set()
        for req_id in req_ids:
            lora_request = self.lora_requests.get(req_id)
            if lora_request is not None:
                active_lora_requests.add(lora_request)
        return prompt_lora_mapping, token_lora_mapping, active_lora_requests
```
**EN:** This method implements `make_lora_inputs` within `LoraState`. Key calls include `tuple`, `set`, `repeat`, `get`, `add`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `make_lora_inputs`，其作用域位于`LoraState`。 关键调用包括 `tuple`, `set`, `repeat`, `get`, `add`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `LoraState`: central class or interface in this module. / `LoraState`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- External / 外部依赖: `numpy`
- Internal vLLM / 内部依赖: `vllm.lora.request`
