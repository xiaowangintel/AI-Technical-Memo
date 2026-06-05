# methods.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/pooler/seqwise/methods.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `SequencePoolingMethod`, `CLSPool`, `LastPool` for pooling heads and methods. / 实现 `SequencePoolingMethod`, `CLSPool`, `LastPool`，用于池化头与池化方法。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 3-13)
```python
from abc import ABC, abstractmethod
from collections.abc import Set
from typing import TypeAlias

import torch
import torch.nn as nn

from vllm.config.pooler import SequencePoolingType
from vllm.model_executor.layers.pooler import PoolingParamsUpdate
from vllm.tasks import PoolingTask
from vllm.v1.pool.metadata import PoolingMetadata
```
**EN:** This opening block pulls in external dependencies such as `abc`, `collections`, `typing`, `torch` and internal modules such as `vllm.config.pooler`, `vllm.model_executor.layers.pooler`, `vllm.tasks`, `vllm.v1.pool.metadata`. That import mix shows the file is part of the pooling heads and methods stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `abc`, `collections`, `typing`, `torch`）以及内部模块（如 `vllm.config.pooler`, `vllm.model_executor.layers.pooler`, `vllm.tasks`, `vllm.v1.pool.metadata`）。这些导入关系表明该文件属于池化头与池化方法栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 15-17)
```python
SequencePoolingMethodOutput: TypeAlias = torch.Tensor | list[torch.Tensor]

_MEAN_POOL_ACCUMULATION_CHUNK_BYTES = 16 * 1024 * 1024  # 16MB
```
**EN:** This block defines module-level metadata or constants such as `SequencePoolingMethodOutput`, `_MEAN_POOL_ACCUMULATION_CHUNK_BYTES`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the pooling heads and methods pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `SequencePoolingMethodOutput`, `_MEAN_POOL_ACCUMULATION_CHUNK_BYTES`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在池化头与池化方法流程中复用。

### Function `get_seq_pooling_method` (lines 111-119)
```python
def get_seq_pooling_method(pooling_type: SequencePoolingType | str):
    if pooling_type == "CLS":
        return CLSPool()
    if pooling_type == "LAST":
        return LastPool()
    if pooling_type == "MEAN":
        return MeanPool()

    raise NotImplementedError(f"Unknown sequence pooling type: {pooling_type!r}")
```
**EN:** Defines function `get_seq_pooling_method` with signature `get_seq_pooling_method(pooling_type: SequencePoolingType | str)`. It mainly works with `pooling_type`; returns a derived property or capability check. The body uses branching, validation/error handling. Key calls include `NotImplementedError`, `CLSPool`, `LastPool`, `MeanPool`.
**CN:** 定义函数 `get_seq_pooling_method`，其签名为 `get_seq_pooling_method(pooling_type: SequencePoolingType | str)`。它主要围绕 `pooling_type` 展开；返回派生属性或能力判断结果。函数体包含分支判断、校验或报错逻辑。关键调用包括 `NotImplementedError`, `CLSPool`, `LastPool`, `MeanPool`。

### Class `SequencePoolingMethod` overview (lines 20-33)
```python
class SequencePoolingMethod(nn.Module, ABC):
    def get_supported_tasks(self) -> Set[PoolingTask]:
        return {"token_embed", "token_classify", "embed", "classify"}

    def get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate:
        return PoolingParamsUpdate()

    @abstractmethod
    def forward(
        self,
        hidden_states: torch.Tensor,
        pooling_metadata: PoolingMetadata,
    ) -> SequencePoolingMethodOutput:
        raise NotImplementedError
```
**EN:** Defines class `SequencePoolingMethod` with base classes `nn.Module`, `ABC` and decorators none. It acts as a backend or execution-method adapter and exposes 3 direct methods, with notable entries `get_supported_tasks`, `get_pooling_updates`, `forward`.
**CN:** 定义类 `SequencePoolingMethod`，其基类为 `nn.Module`, `ABC`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 3 个方法，较重要的包括 `get_supported_tasks`, `get_pooling_updates`, `forward`。

### Method `SequencePoolingMethod.get_supported_tasks` (lines 21-22)
```python
    def get_supported_tasks(self) -> Set[PoolingTask]:
        return {"token_embed", "token_classify", "embed", "classify"}
```
**EN:** Defines function `SequencePoolingMethod.get_supported_tasks` with signature `get_supported_tasks(self) -> Set[PoolingTask]`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `SequencePoolingMethod.get_supported_tasks`，其签名为 `get_supported_tasks(self) -> Set[PoolingTask]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `SequencePoolingMethod.get_pooling_updates` (lines 24-25)
```python
    def get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate:
        return PoolingParamsUpdate()
```
**EN:** Defines function `SequencePoolingMethod.get_pooling_updates` with signature `get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate`. It mainly works with `task`; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include `PoolingParamsUpdate`.
**CN:** 定义函数 `SequencePoolingMethod.get_pooling_updates`，其签名为 `get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate`。它主要围绕 `task` 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `PoolingParamsUpdate`。

### Method `SequencePoolingMethod.forward` (lines 28-33)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        pooling_metadata: PoolingMetadata,
    ) -> SequencePoolingMethodOutput:
        raise NotImplementedError
```
**EN:** Defines function `SequencePoolingMethod.forward` with signature `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> SequencePoolingMethodOutput`. It mainly works with `hidden_states`, `pooling_metadata`; runs the main forward-path computation. The body uses validation/error handling. Key calls include no notable helper calls.
**CN:** 定义函数 `SequencePoolingMethod.forward`，其签名为 `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> SequencePoolingMethodOutput`。它主要围绕 `hidden_states`, `pooling_metadata` 展开；执行主要的前向计算路径。函数体包含校验或报错逻辑。关键调用包括 无显著辅助调用。

### Class `CLSPool` overview (lines 36-47)
```python
class CLSPool(SequencePoolingMethod):
    def forward(
        self,
        hidden_states: torch.Tensor,
        pooling_metadata: PoolingMetadata,
    ) -> SequencePoolingMethodOutput:
        pooling_cursor = pooling_metadata.get_pooling_cursor()
        assert not pooling_cursor.is_partial_prefill(), (
            "partial prefill not supported with CLS pooling"
        )

        return hidden_states[pooling_cursor.first_token_indices_gpu]
```
**EN:** Defines class `CLSPool` with base classes `SequencePoolingMethod` and decorators none. It acts as a reusable module building block and exposes 1 direct methods, with notable entries `forward`.
**CN:** 定义类 `CLSPool`，其基类为 `SequencePoolingMethod`，装饰器为 无。它在整体实现中充当可复用的模块构件，并直接暴露 1 个方法，较重要的包括 `forward`。

### Method `CLSPool.forward` (lines 37-47)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        pooling_metadata: PoolingMetadata,
    ) -> SequencePoolingMethodOutput:
        pooling_cursor = pooling_metadata.get_pooling_cursor()
        assert not pooling_cursor.is_partial_prefill(), (
            "partial prefill not supported with CLS pooling"
        )

        return hidden_states[pooling_cursor.first_token_indices_gpu]
```
**EN:** Defines function `CLSPool.forward` with signature `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> SequencePoolingMethodOutput`. It mainly works with `hidden_states`, `pooling_metadata`; runs the main forward-path computation. The body uses validation/error handling. Key calls include `pooling_metadata.get_pooling_cursor`, `pooling_cursor.is_partial_prefill`.
**CN:** 定义函数 `CLSPool.forward`，其签名为 `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> SequencePoolingMethodOutput`。它主要围绕 `hidden_states`, `pooling_metadata` 展开；执行主要的前向计算路径。函数体包含校验或报错逻辑。关键调用包括 `pooling_metadata.get_pooling_cursor`, `pooling_cursor.is_partial_prefill`。

### Class `LastPool` overview (lines 50-57)
```python
class LastPool(SequencePoolingMethod):
    def forward(
        self,
        hidden_states: torch.Tensor,
        pooling_metadata: PoolingMetadata,
    ) -> SequencePoolingMethodOutput:
        pooling_cursor = pooling_metadata.get_pooling_cursor()
        return hidden_states[pooling_cursor.last_token_indices_gpu]
```
**EN:** Defines class `LastPool` with base classes `SequencePoolingMethod` and decorators none. It acts as a reusable module building block and exposes 1 direct methods, with notable entries `forward`.
**CN:** 定义类 `LastPool`，其基类为 `SequencePoolingMethod`，装饰器为 无。它在整体实现中充当可复用的模块构件，并直接暴露 1 个方法，较重要的包括 `forward`。

### Method `LastPool.forward` (lines 51-57)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        pooling_metadata: PoolingMetadata,
    ) -> SequencePoolingMethodOutput:
        pooling_cursor = pooling_metadata.get_pooling_cursor()
        return hidden_states[pooling_cursor.last_token_indices_gpu]
```
**EN:** Defines function `LastPool.forward` with signature `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> SequencePoolingMethodOutput`. It mainly works with `hidden_states`, `pooling_metadata`; runs the main forward-path computation. The body uses mostly straightforward data movement and object wiring. Key calls include `pooling_metadata.get_pooling_cursor`.
**CN:** 定义函数 `LastPool.forward`，其签名为 `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> SequencePoolingMethodOutput`。它主要围绕 `hidden_states`, `pooling_metadata` 展开；执行主要的前向计算路径。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `pooling_metadata.get_pooling_cursor`。

### Class `MeanPool` overview (lines 60-108)
```python
class MeanPool(SequencePoolingMethod):
    def forward(
        self,
        hidden_states: torch.Tensor,
        pooling_metadata: PoolingMetadata,
    ) -> SequencePoolingMethodOutput:
        pooling_cursor = pooling_metadata.get_pooling_cursor()
        assert not pooling_cursor.is_partial_prefill(), (
            "partial prefill not supported with MEAN pooling"
        )

        prompt_lens_cpu = pooling_cursor.prompt_lens_cpu
        num_seqs = prompt_lens_cpu.numel()
        hidden_size = hidden_states.shape[-1]

        if num_seqs == 0:
            # early return for empty batch
            return hidden_states.new_empty((0, hidden_size), dtype=torch.float32)

        # Build segment_ids on CPU so repeat_interleave doesn't need to sync
        # GPU->CPU to learn its data-dependent output length, then upload
        # non-blocking. eg. [2, 1, 3] -> [0, 0, 1, 2, 2, 2]
        segment_ids = torch.repeat_interleave(
            torch.arange(num_seqs, dtype=torch.long),
            prompt_lens_cpu,
```
**EN:** Defines class `MeanPool` with base classes `SequencePoolingMethod` and decorators none. It acts as a reusable module building block and exposes 1 direct methods, with notable entries `forward`.
**CN:** 定义类 `MeanPool`，其基类为 `SequencePoolingMethod`，装饰器为 无。它在整体实现中充当可复用的模块构件，并直接暴露 1 个方法，较重要的包括 `forward`。

### Method `MeanPool.forward` (lines 61-108)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        pooling_metadata: PoolingMetadata,
    ) -> SequencePoolingMethodOutput:
        pooling_cursor = pooling_metadata.get_pooling_cursor()
        assert not pooling_cursor.is_partial_prefill(), (
            "partial prefill not supported with MEAN pooling"
        )

        prompt_lens_cpu = pooling_cursor.prompt_lens_cpu
        num_seqs = prompt_lens_cpu.numel()
        hidden_size = hidden_states.shape[-1]

        if num_seqs == 0:
            # early return for empty batch
            return hidden_states.new_empty((0, hidden_size), dtype=torch.float32)

        # Build segment_ids on CPU so repeat_interleave doesn't need to sync
        # GPU->CPU to learn its data-dependent output length, then upload
        # non-blocking. eg. [2, 1, 3] -> [0, 0, 1, 2, 2, 2]
        segment_ids = torch.repeat_interleave(
            torch.arange(num_seqs, dtype=torch.long),
            prompt_lens_cpu,
        ).to(hidden_states.device, non_blocking=True)
        prompt_lens = prompt_lens_cpu.to(
            hidden_states.device, dtype=torch.int64, non_blocking=True
        )
        segment_sums = torch.zeros(
            (num_seqs, hidden_size),
            dtype=torch.float32,
            device=hidden_states.device,
        )

        bytes_per_token = hidden_size * torch.finfo(torch.float32).bits // 8
        chunk_size = max(1, _MEAN_POOL_ACCUMULATION_CHUNK_BYTES // bytes_per_token)

        # iterate over the batch in chunks
        for start in range(0, hidden_states.shape[0], chunk_size):
            end = min(start + chunk_size, hidden_states.shape[0])
            # using index_add_ to accumulate for each segment
            segment_sums.index_add_(
                0,
                segment_ids[start:end],
                hidden_states[start:end].to(dtype=torch.float32),
            )

        return segment_sums / prompt_lens.unsqueeze(1)
```
**EN:** Defines function `MeanPool.forward` with signature `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> SequencePoolingMethodOutput`. It mainly works with `hidden_states`, `pooling_metadata`; runs the main forward-path computation. The body uses branching, iteration, validation/error handling, tensor/kernel operations. Key calls include `pooling_metadata.get_pooling_cursor`, `prompt_lens_cpu.numel`, `torch.repeat_interleave.to`, `prompt_lens_cpu.to`, `torch.zeros`, `max`.
**CN:** 定义函数 `MeanPool.forward`，其签名为 `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> SequencePoolingMethodOutput`。它主要围绕 `hidden_states`, `pooling_metadata` 展开；执行主要的前向计算路径。函数体包含分支判断、循环处理、校验或报错逻辑、张量或内核操作。关键调用包括 `pooling_metadata.get_pooling_cursor`, `prompt_lens_cpu.numel`, `torch.repeat_interleave.to`, `prompt_lens_cpu.to`, `torch.zeros`, `max`。

## Key Concepts / 关键概念
- **EN:** The file is part of the pooling stack that converts token-level hidden states into sequence-level or token-wise outputs.
  **CN:** 该文件属于池化栈，用于把 token 级隐藏状态转换为序列级或逐 token 输出。
- **EN:** Top-level classes include `SequencePoolingMethod`, `CLSPool`, `LastPool`, `MeanPool`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `SequencePoolingMethod`, `CLSPool`, `LastPool`, `MeanPool`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `get_seq_pooling_method` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `get_seq_pooling_method` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `abc`, `collections`, `typing`, `torch`
- **Internal / 内部**: `vllm.config.pooler`, `vllm.model_executor.layers.pooler`, `vllm.tasks`, `vllm.v1.pool.metadata`
