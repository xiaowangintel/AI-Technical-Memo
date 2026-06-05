# methods.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/pooler/tokwise/methods.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `TokenPoolingMethod`, `AllPool`, `StepPool` for pooling heads and methods. / 实现 `TokenPoolingMethod`, `AllPool`, `StepPool`，用于池化头与池化方法。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 3-14)
```python
from abc import ABC, abstractmethod
from collections.abc import Set
from typing import TypeAlias

import torch
import torch.nn as nn

from vllm.config import get_current_vllm_config
from vllm.config.pooler import TokenPoolingType
from vllm.model_executor.layers.pooler import PoolingParamsUpdate
from vllm.tasks import PoolingTask
from vllm.v1.pool.metadata import PoolingMetadata
```
**EN:** This opening block pulls in external dependencies such as `abc`, `collections`, `typing`, `torch` and internal modules such as `vllm.config`, `vllm.config.pooler`, `vllm.model_executor.layers.pooler`, `vllm.tasks`, `vllm.v1.pool.metadata`. That import mix shows the file is part of the pooling heads and methods stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `abc`, `collections`, `typing`, `torch`）以及内部模块（如 `vllm.config`, `vllm.config.pooler`, `vllm.model_executor.layers.pooler`, `vllm.tasks`, `vllm.v1.pool.metadata`）。这些导入关系表明该文件属于池化头与池化方法栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 16-16)
```python
TokenPoolingMethodOutputItem: TypeAlias = torch.Tensor | None
```
**EN:** This block defines module-level metadata or constants such as `TokenPoolingMethodOutputItem`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the pooling heads and methods pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `TokenPoolingMethodOutputItem`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在池化头与池化方法流程中复用。

### Function `get_tok_pooling_method` (lines 122-128)
```python
def get_tok_pooling_method(pooling_type: TokenPoolingType | str):
    if pooling_type == "ALL":
        return AllPool()
    if pooling_type == "STEP":
        return StepPool()

    raise NotImplementedError(f"Unknown tokenwise pooling type: {pooling_type!r}")
```
**EN:** Defines function `get_tok_pooling_method` with signature `get_tok_pooling_method(pooling_type: TokenPoolingType | str)`. It mainly works with `pooling_type`; returns a derived property or capability check. The body uses branching, validation/error handling. Key calls include `NotImplementedError`, `AllPool`, `StepPool`.
**CN:** 定义函数 `get_tok_pooling_method`，其签名为 `get_tok_pooling_method(pooling_type: TokenPoolingType | str)`。它主要围绕 `pooling_type` 展开；返回派生属性或能力判断结果。函数体包含分支判断、校验或报错逻辑。关键调用包括 `NotImplementedError`, `AllPool`, `StepPool`。

### Class `TokenPoolingMethod` overview (lines 19-32)
```python
class TokenPoolingMethod(nn.Module, ABC):
    def get_supported_tasks(self) -> Set[PoolingTask]:
        return {"token_embed", "token_classify"}

    def get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate:
        return PoolingParamsUpdate()

    @abstractmethod
    def forward(
        self,
        hidden_states: torch.Tensor,
        pooling_metadata: PoolingMetadata,
    ) -> list[TokenPoolingMethodOutputItem]:
        raise NotImplementedError
```
**EN:** Defines class `TokenPoolingMethod` with base classes `nn.Module`, `ABC` and decorators none. It acts as a backend or execution-method adapter and exposes 3 direct methods, with notable entries `get_supported_tasks`, `get_pooling_updates`, `forward`.
**CN:** 定义类 `TokenPoolingMethod`，其基类为 `nn.Module`, `ABC`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 3 个方法，较重要的包括 `get_supported_tasks`, `get_pooling_updates`, `forward`。

### Method `TokenPoolingMethod.get_supported_tasks` (lines 20-21)
```python
    def get_supported_tasks(self) -> Set[PoolingTask]:
        return {"token_embed", "token_classify"}
```
**EN:** Defines function `TokenPoolingMethod.get_supported_tasks` with signature `get_supported_tasks(self) -> Set[PoolingTask]`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `TokenPoolingMethod.get_supported_tasks`，其签名为 `get_supported_tasks(self) -> Set[PoolingTask]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `TokenPoolingMethod.get_pooling_updates` (lines 23-24)
```python
    def get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate:
        return PoolingParamsUpdate()
```
**EN:** Defines function `TokenPoolingMethod.get_pooling_updates` with signature `get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate`. It mainly works with `task`; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include `PoolingParamsUpdate`.
**CN:** 定义函数 `TokenPoolingMethod.get_pooling_updates`，其签名为 `get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate`。它主要围绕 `task` 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `PoolingParamsUpdate`。

### Method `TokenPoolingMethod.forward` (lines 27-32)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        pooling_metadata: PoolingMetadata,
    ) -> list[TokenPoolingMethodOutputItem]:
        raise NotImplementedError
```
**EN:** Defines function `TokenPoolingMethod.forward` with signature `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> list[TokenPoolingMethodOutputItem]`. It mainly works with `hidden_states`, `pooling_metadata`; runs the main forward-path computation. The body uses validation/error handling. Key calls include no notable helper calls.
**CN:** 定义函数 `TokenPoolingMethod.forward`，其签名为 `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> list[TokenPoolingMethodOutputItem]`。它主要围绕 `hidden_states`, `pooling_metadata` 展开；执行主要的前向计算路径。函数体包含校验或报错逻辑。关键调用包括 无显著辅助调用。

### Class `AllPool` overview (lines 35-80)
```python
class AllPool(TokenPoolingMethod):
    def __init__(self):
        super().__init__()

        vllm_config = get_current_vllm_config()
        scheduler_config = vllm_config.scheduler_config

        self.enable_chunked_prefill = scheduler_config.enable_chunked_prefill

    def forward(
        self,
        hidden_states: torch.Tensor,
        pooling_metadata: PoolingMetadata,
    ) -> list[TokenPoolingMethodOutputItem]:
        pooling_cursor = pooling_metadata.get_pooling_cursor()
        # Use the already-CPU num_scheduled_tokens tensor so `.tolist()`
        # doesn't trigger a GPU->CPU sync. torch.split produces the same
        # consecutive slices as indexing with first/last per-sequence indices.
        hidden_states_lst = list(
            torch.split(hidden_states, pooling_cursor.num_scheduled_tokens_cpu.tolist())
        )

        if not self.enable_chunked_prefill:
            return hidden_states_lst
```
**EN:** Defines class `AllPool` with base classes `TokenPoolingMethod` and decorators none. It acts as a reusable module building block and exposes 2 direct methods, with notable entries `__init__`, `forward`.
**CN:** 定义类 `AllPool`，其基类为 `TokenPoolingMethod`，装饰器为 无。它在整体实现中充当可复用的模块构件，并直接暴露 2 个方法，较重要的包括 `__init__`, `forward`。

### Method `AllPool.__init__` (lines 36-42)
```python
    def __init__(self):
        super().__init__()

        vllm_config = get_current_vllm_config()
        scheduler_config = vllm_config.scheduler_config

        self.enable_chunked_prefill = scheduler_config.enable_chunked_prefill
```
**EN:** Defines function `AllPool.__init__` with signature `__init__(self)`. It mainly works with object context only; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `get_current_vllm_config`, `super`.
**CN:** 定义函数 `AllPool.__init__`，其签名为 `__init__(self)`。它主要围绕 仅依赖对象上下文 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `get_current_vllm_config`, `super`。

### Method `AllPool.forward` (lines 44-80)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        pooling_metadata: PoolingMetadata,
    ) -> list[TokenPoolingMethodOutputItem]:
        pooling_cursor = pooling_metadata.get_pooling_cursor()
        # Use the already-CPU num_scheduled_tokens tensor so `.tolist()`
        # doesn't trigger a GPU->CPU sync. torch.split produces the same
        # consecutive slices as indexing with first/last per-sequence indices.
        hidden_states_lst = list(
            torch.split(hidden_states, pooling_cursor.num_scheduled_tokens_cpu.tolist())
        )

        if not self.enable_chunked_prefill:
            return hidden_states_lst

        pooling_states = pooling_metadata.pooling_states

        # If chunked_prefill is enabled
        # 1. first store the chunked hidden_states in pooling_states.hidden_states_cache
        for p, hs_chunk in zip(pooling_states, hidden_states_lst):
            p.hidden_states_cache.append(hs_chunk)

        # 2. Once prefill is finished, send hidden_states_cache to PoolerHead
        output_list = list[TokenPoolingMethodOutputItem]()
        for p, finished in zip(pooling_states, pooling_cursor.is_finished()):
            if finished:
                hidden_states_cache = p.hidden_states_cache
                if len(hidden_states_cache) == 1:
                    output_list.append(hidden_states_cache[0])
                else:
                    output_list.append(torch.concat(hidden_states_cache, dim=0))
                p.clean()
            else:
                output_list.append(None)

        return output_list
```
**EN:** Defines function `AllPool.forward` with signature `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> list[TokenPoolingMethodOutputItem]`. It mainly works with `hidden_states`, `pooling_metadata`; runs the main forward-path computation. The body uses branching, iteration, tensor/kernel operations. Key calls include `pooling_metadata.get_pooling_cursor`, `list`, `zip`, `torch.split`, `p.hidden_states_cache.append`, `pooling_cursor.is_finished`.
**CN:** 定义函数 `AllPool.forward`，其签名为 `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> list[TokenPoolingMethodOutputItem]`。它主要围绕 `hidden_states`, `pooling_metadata` 展开；执行主要的前向计算路径。函数体包含分支判断、循环处理、张量或内核操作。关键调用包括 `pooling_metadata.get_pooling_cursor`, `list`, `zip`, `torch.split`, `p.hidden_states_cache.append`, `pooling_cursor.is_finished`。

### Class `StepPool` overview (lines 83-119)
```python
class StepPool(AllPool):
    def get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate:
        return PoolingParamsUpdate(requires_token_ids=True)

    def forward(
        self,
        hidden_states: torch.Tensor,
        pooling_metadata: PoolingMetadata,
    ) -> list[TokenPoolingMethodOutputItem]:
        pooled_data_lst = super().forward(hidden_states, pooling_metadata)
        # Use the CPU copy of prompt_token_ids so the step_tag_id mask can be
        # resolved to indices without a d2h sync from boolean indexing.
        prompt_token_ids_cpu = pooling_metadata.get_prompt_token_ids_cpu()
        pooling_params = pooling_metadata.pooling_params

        pooled_data = list[torch.Tensor | None]()
        for data, token_id_cpu, pooling_param in zip(
            pooled_data_lst, prompt_token_ids_cpu, pooling_params
        ):
            # for unfinished chunked prefill
            if data is None:
                pooled_data.append(None)
            else:
                step_tag_id = pooling_param.step_tag_id
                returned_token_ids = pooling_param.returned_token_ids
```
**EN:** Defines class `StepPool` with base classes `AllPool` and decorators none. It acts as a reusable module building block and exposes 2 direct methods, with notable entries `get_pooling_updates`, `forward`.
**CN:** 定义类 `StepPool`，其基类为 `AllPool`，装饰器为 无。它在整体实现中充当可复用的模块构件，并直接暴露 2 个方法，较重要的包括 `get_pooling_updates`, `forward`。

### Method `StepPool.get_pooling_updates` (lines 84-85)
```python
    def get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate:
        return PoolingParamsUpdate(requires_token_ids=True)
```
**EN:** Defines function `StepPool.get_pooling_updates` with signature `get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate`. It mainly works with `task`; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include `PoolingParamsUpdate`.
**CN:** 定义函数 `StepPool.get_pooling_updates`，其签名为 `get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate`。它主要围绕 `task` 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `PoolingParamsUpdate`。

### Method `StepPool.forward` (lines 87-119)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        pooling_metadata: PoolingMetadata,
    ) -> list[TokenPoolingMethodOutputItem]:
        pooled_data_lst = super().forward(hidden_states, pooling_metadata)
        # Use the CPU copy of prompt_token_ids so the step_tag_id mask can be
        # resolved to indices without a d2h sync from boolean indexing.
        prompt_token_ids_cpu = pooling_metadata.get_prompt_token_ids_cpu()
        pooling_params = pooling_metadata.pooling_params

        pooled_data = list[torch.Tensor | None]()
        for data, token_id_cpu, pooling_param in zip(
            pooled_data_lst, prompt_token_ids_cpu, pooling_params
        ):
            # for unfinished chunked prefill
            if data is None:
                pooled_data.append(None)
            else:
                step_tag_id = pooling_param.step_tag_id
                returned_token_ids = pooling_param.returned_token_ids

                if returned_token_ids is not None and len(returned_token_ids) > 0:
                    data = data[:, returned_token_ids]

                if step_tag_id is not None:
                    idx_cpu = (token_id_cpu == step_tag_id).nonzero(as_tuple=True)[0]
                    idx = idx_cpu.to(data.device, non_blocking=True)
                    data = data[idx]

                pooled_data.append(data)

        return pooled_data
```
**EN:** Defines function `StepPool.forward` with signature `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> list[TokenPoolingMethodOutputItem]`. It mainly works with `hidden_states`, `pooling_metadata`; runs the main forward-path computation. The body uses branching, iteration. Key calls include `super.forward`, `pooling_metadata.get_prompt_token_ids_cpu`, `list`, `zip`, `super`, `pooled_data.append`.
**CN:** 定义函数 `StepPool.forward`，其签名为 `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> list[TokenPoolingMethodOutputItem]`。它主要围绕 `hidden_states`, `pooling_metadata` 展开；执行主要的前向计算路径。函数体包含分支判断、循环处理。关键调用包括 `super.forward`, `pooling_metadata.get_prompt_token_ids_cpu`, `list`, `zip`, `super`, `pooled_data.append`。

## Key Concepts / 关键概念
- **EN:** The file is part of the pooling stack that converts token-level hidden states into sequence-level or token-wise outputs.
  **CN:** 该文件属于池化栈，用于把 token 级隐藏状态转换为序列级或逐 token 输出。
- **EN:** Top-level classes include `TokenPoolingMethod`, `AllPool`, `StepPool`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `TokenPoolingMethod`, `AllPool`, `StepPool`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `get_tok_pooling_method` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `get_tok_pooling_method` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `abc`, `collections`, `typing`, `torch`
- **Internal / 内部**: `vllm.config`, `vllm.config.pooler`, `vllm.model_executor.layers.pooler`, `vllm.tasks`, `vllm.v1.pool.metadata`
