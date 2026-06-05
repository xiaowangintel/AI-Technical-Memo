# special.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/pooler/special.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `DispatchPooler`, `IdentityPooler`, `BOSEOSFilter` for pooling heads and methods. / 实现 `DispatchPooler`, `IdentityPooler`, `BOSEOSFilter`，用于池化头与池化方法。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 3-22)
```python
import dataclasses
from collections.abc import Mapping, Set
from itertools import groupby

import torch

from vllm.config import PoolerConfig
from vllm.model_executor.layers.pooler import PoolingParamsUpdate
from vllm.tasks import PoolingTask
from vllm.v1.pool.metadata import PoolingMetadata

from .abstract import Pooler, PoolerOutput
from .common import ClassifierFn
from .seqwise import (
    SequencePoolingFn,
    SequencePoolingMethod,
    pooler_for_classify,
    pooler_for_embed,
)
from .tokwise import AllPool, pooler_for_token_classify, pooler_for_token_embed
```
**EN:** This opening block pulls in external dependencies such as `dataclasses`, `collections`, `itertools`, `torch` and internal modules such as `vllm.config`, `vllm.model_executor.layers.pooler`, `vllm.tasks`, `vllm.v1.pool.metadata`, `.abstract`, `.common`. That import mix shows the file is part of the pooling heads and methods stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `dataclasses`, `collections`, `itertools`, `torch`）以及内部模块（如 `vllm.config`, `vllm.model_executor.layers.pooler`, `vllm.tasks`, `vllm.v1.pool.metadata`, `.abstract`, `.common`）。这些导入关系表明该文件属于池化头与池化方法栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 236-236)
```python
__all__ = ["BOSEOSFilter", "DispatchPooler", "IdentityPooler", "BgeM3Pooler"]
```
**EN:** This block defines module-level metadata or constants such as `__all__`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the pooling heads and methods pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `__all__`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在池化头与池化方法流程中复用。

### Class `DispatchPooler` overview (lines 25-136)
```python
class DispatchPooler(Pooler):
    """Dispatches calls to a sub-pooler based on the pooling task."""

    @classmethod
    def for_embedding(cls, pooler_config: PoolerConfig):
        return cls(
            {
                "token_embed": pooler_for_token_embed(pooler_config),
                "embed": pooler_for_embed(pooler_config),
            },
        )

    @classmethod
    def for_seq_cls(
        cls,
        pooler_config: PoolerConfig,
        *,
        pooling: SequencePoolingMethod | SequencePoolingFn | None = None,
        classifier: ClassifierFn | None = None,
    ):
        return cls(
            {
                "token_classify": pooler_for_token_classify(
                    pooler_config,
                    pooling=AllPool(),
```
**EN:** Defines class `DispatchPooler` with base classes `Pooler` and decorators none. It acts as a pooling implementation and exposes 7 direct methods, with notable entries `for_embedding`, `for_seq_cls`, `__init__`, `get_supported_tasks`, `get_pooling_updates`, `forward`. Its docstring says: Dispatches calls to a sub-pooler based on the pooling task.
**CN:** 定义类 `DispatchPooler`，其基类为 `Pooler`，装饰器为 无。它在整体实现中充当池化实现，并直接暴露 7 个方法，较重要的包括 `for_embedding`, `for_seq_cls`, `__init__`, `get_supported_tasks`, `get_pooling_updates`, `forward`。 文档字符串进一步说明了该类的定位。

### Method `DispatchPooler.for_embedding` (lines 29-35)
```python
    def for_embedding(cls, pooler_config: PoolerConfig):
        return cls(
            {
                "token_embed": pooler_for_token_embed(pooler_config),
                "embed": pooler_for_embed(pooler_config),
            },
        )
```
**EN:** Defines function `DispatchPooler.for_embedding` with signature `for_embedding(cls, pooler_config: PoolerConfig)`. It mainly works with `pooler_config`; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include `cls`, `pooler_for_token_embed`, `pooler_for_embed`.
**CN:** 定义函数 `DispatchPooler.for_embedding`，其签名为 `for_embedding(cls, pooler_config: PoolerConfig)`。它主要围绕 `pooler_config` 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `cls`, `pooler_for_token_embed`, `pooler_for_embed`。

### Method `DispatchPooler.for_seq_cls` (lines 38-58)
```python
    def for_seq_cls(
        cls,
        pooler_config: PoolerConfig,
        *,
        pooling: SequencePoolingMethod | SequencePoolingFn | None = None,
        classifier: ClassifierFn | None = None,
    ):
        return cls(
            {
                "token_classify": pooler_for_token_classify(
                    pooler_config,
                    pooling=AllPool(),
                    classifier=classifier,
                ),
                "classify": pooler_for_classify(
                    pooler_config,
                    pooling=pooling,
                    classifier=classifier,
                ),
            }
        )
```
**EN:** Defines function `DispatchPooler.for_seq_cls` with signature `for_seq_cls(cls, pooler_config: PoolerConfig, *, pooling: SequencePoolingMethod | SequencePoolingFn | None=None, classifier: ClassifierFn | None=None)`. It mainly works with `pooler_config`, `pooling`, `classifier`; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include `cls`, `pooler_for_token_classify`, `pooler_for_classify`, `AllPool`.
**CN:** 定义函数 `DispatchPooler.for_seq_cls`，其签名为 `for_seq_cls(cls, pooler_config: PoolerConfig, *, pooling: SequencePoolingMethod | SequencePoolingFn | None=None, classifier: ClassifierFn | None=None)`。它主要围绕 `pooler_config`, `pooling`, `classifier` 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `cls`, `pooler_for_token_classify`, `pooler_for_classify`, `AllPool`。

### Method `DispatchPooler.__init__` (lines 60-70)
```python
    def __init__(self, poolers_by_task: Mapping[PoolingTask, Pooler]) -> None:
        super().__init__()

        for task, pooler in poolers_by_task.items():
            if task not in pooler.get_supported_tasks():
                raise ValueError(
                    f"{pooler=} does not support {task=}. "
                    f"Supported tasks: {pooler.get_supported_tasks()}"
                )

        self.poolers_by_task = poolers_by_task
```
**EN:** Defines function `DispatchPooler.__init__` with signature `__init__(self, poolers_by_task: Mapping[PoolingTask, Pooler]) -> None`. It mainly works with `poolers_by_task`; initializes the object state and cached resources. The body uses branching, iteration, validation/error handling. Key calls include `super.__init__`, `poolers_by_task.items`, `super`, `pooler.get_supported_tasks`, `ValueError`.
**CN:** 定义函数 `DispatchPooler.__init__`，其签名为 `__init__(self, poolers_by_task: Mapping[PoolingTask, Pooler]) -> None`。它主要围绕 `poolers_by_task` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、循环处理、校验或报错逻辑。关键调用包括 `super.__init__`, `poolers_by_task.items`, `super`, `pooler.get_supported_tasks`, `ValueError`。

### Method `DispatchPooler.get_supported_tasks` (lines 72-73)
```python
    def get_supported_tasks(self) -> Set[PoolingTask]:
        return set(self.poolers_by_task)
```
**EN:** Defines function `DispatchPooler.get_supported_tasks` with signature `get_supported_tasks(self) -> Set[PoolingTask]`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include `set`.
**CN:** 定义函数 `DispatchPooler.get_supported_tasks`，其签名为 `get_supported_tasks(self) -> Set[PoolingTask]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `set`。

### Method `DispatchPooler.get_pooling_updates` (lines 75-76)
```python
    def get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate:
        return self.poolers_by_task[task].get_pooling_updates(task)
```
**EN:** Defines function `DispatchPooler.get_pooling_updates` with signature `get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate`. It mainly works with `task`; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include `self.poolers_by_task.get_pooling_updates`.
**CN:** 定义函数 `DispatchPooler.get_pooling_updates`，其签名为 `get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate`。它主要围绕 `task` 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.poolers_by_task.get_pooling_updates`。

### Method `DispatchPooler.forward` (lines 78-132)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        pooling_metadata: PoolingMetadata,
    ) -> PoolerOutput:
        poolers_by_task = self.poolers_by_task
        cursor = pooling_metadata.pooling_cursor

        outputs = list[torch.Tensor | None]()
        offset = 0
        token_offset = 0
        for task, group in groupby(pooling_metadata.tasks):
            if not (pooler := poolers_by_task.get(task)):
                raise ValueError(
                    f"Unsupported task: {task!r} "
                    f"Supported tasks: {self.get_supported_tasks()}"
                )

            num_items = len(list(group))
            group_metadata = pooling_metadata[offset : offset + num_items]
            if cursor is None:
                group_hidden_states = hidden_states
            else:
                # Slice out this group's tokens so sub-poolers see only their
                # portion of the batch. Token offset is computed from the CPU
                # `num_scheduled_tokens_cpu` to avoid a GPU->CPU sync.
                group_cursor = group_metadata.pooling_cursor
                num_group_tokens = int(group_cursor.num_scheduled_tokens_cpu.sum())
                group_hidden_states = hidden_states[
                    token_offset : token_offset + num_group_tokens
                ]
                if token_offset:
                    # Shift first/last indices to be relative to the slice
                    # so seqwise poolers (which index `hidden_states` directly)
                    # remain correct.
                    pooling_cursor = dataclasses.replace(
                        group_cursor,
                        first_token_indices_gpu=(
                            group_cursor.first_token_indices_gpu - token_offset
                        ),
                        last_token_indices_gpu=(
                            group_cursor.last_token_indices_gpu - token_offset
                        ),
                    )
                    group_metadata = dataclasses.replace(
                        group_metadata, pooling_cursor=pooling_cursor
                    )
                token_offset += num_group_tokens

            group_output: PoolerOutput = pooler(group_hidden_states, group_metadata)

            outputs.extend(group_output)
            offset += num_items

        return outputs
```
**EN:** Defines function `DispatchPooler.forward` with signature `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> PoolerOutput`. It mainly works with `hidden_states`, `pooling_metadata`; runs the main forward-path computation. The body uses branching, iteration, validation/error handling. Key calls include `list`, `groupby`, `len`, `pooler`, `outputs.extend`, `ValueError`.
**CN:** 定义函数 `DispatchPooler.forward`，其签名为 `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> PoolerOutput`。它主要围绕 `hidden_states`, `pooling_metadata` 展开；执行主要的前向计算路径。函数体包含分支判断、循环处理、校验或报错逻辑。关键调用包括 `list`, `groupby`, `len`, `pooler`, `outputs.extend`, `ValueError`。

### Class `IdentityPooler` overview (lines 139-148)
```python
class IdentityPooler(Pooler):
    def get_supported_tasks(self) -> Set[PoolingTask]:
        return {"plugin"}

    def forward(
        self,
        hidden_states: torch.Tensor,
        pooling_metadata: PoolingMetadata,
    ) -> PoolerOutput:
        return hidden_states
```
**EN:** Defines class `IdentityPooler` with base classes `Pooler` and decorators none. It acts as a pooling implementation and exposes 2 direct methods, with notable entries `get_supported_tasks`, `forward`.
**CN:** 定义类 `IdentityPooler`，其基类为 `Pooler`，装饰器为 无。它在整体实现中充当池化实现，并直接暴露 2 个方法，较重要的包括 `get_supported_tasks`, `forward`。

### Method `IdentityPooler.get_supported_tasks` (lines 140-141)
```python
    def get_supported_tasks(self) -> Set[PoolingTask]:
        return {"plugin"}
```
**EN:** Defines function `IdentityPooler.get_supported_tasks` with signature `get_supported_tasks(self) -> Set[PoolingTask]`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `IdentityPooler.get_supported_tasks`，其签名为 `get_supported_tasks(self) -> Set[PoolingTask]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `IdentityPooler.forward` (lines 143-148)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        pooling_metadata: PoolingMetadata,
    ) -> PoolerOutput:
        return hidden_states
```
**EN:** Defines function `IdentityPooler.forward` with signature `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> PoolerOutput`. It mainly works with `hidden_states`, `pooling_metadata`; runs the main forward-path computation. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `IdentityPooler.forward`，其签名为 `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> PoolerOutput`。它主要围绕 `hidden_states`, `pooling_metadata` 展开；执行主要的前向计算路径。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Class `BOSEOSFilter` overview (lines 151-195)
```python
class BOSEOSFilter(Pooler):
    """Filters the BOS and EOS token results from outputs."""

    def __init__(
        self,
        pooler: Pooler,
        bos_token_id: int = -1,  # -1 disables the filtering
        eos_token_id: int = -1,
    ) -> None:
        super().__init__()

        self.pooler = pooler
        self.bos_token_id = bos_token_id
        self.eos_token_id = eos_token_id

    def get_supported_tasks(self) -> Set[PoolingTask]:
        return self.pooler.get_supported_tasks()

    def get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate:
        return PoolingParamsUpdate(requires_token_ids=True)

    def forward(
        self,
        hidden_states: torch.Tensor | list[torch.Tensor],
        pooling_metadata: PoolingMetadata,
```
**EN:** Defines class `BOSEOSFilter` with base classes `Pooler` and decorators none. It acts as a reusable module building block and exposes 4 direct methods, with notable entries `__init__`, `get_supported_tasks`, `get_pooling_updates`, `forward`. Its docstring says: Filters the BOS and EOS token results from outputs.
**CN:** 定义类 `BOSEOSFilter`，其基类为 `Pooler`，装饰器为 无。它在整体实现中充当可复用的模块构件，并直接暴露 4 个方法，较重要的包括 `__init__`, `get_supported_tasks`, `get_pooling_updates`, `forward`。 文档字符串进一步说明了该类的定位。

### Method `BOSEOSFilter.__init__` (lines 154-164)
```python
    def __init__(
        self,
        pooler: Pooler,
        bos_token_id: int = -1,  # -1 disables the filtering
        eos_token_id: int = -1,
    ) -> None:
        super().__init__()

        self.pooler = pooler
        self.bos_token_id = bos_token_id
        self.eos_token_id = eos_token_id
```
**EN:** Defines function `BOSEOSFilter.__init__` with signature `__init__(self, pooler: Pooler, bos_token_id: int=-1, eos_token_id: int=-1) -> None`. It mainly works with `pooler`, `bos_token_id`, `eos_token_id`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `BOSEOSFilter.__init__`，其签名为 `__init__(self, pooler: Pooler, bos_token_id: int=-1, eos_token_id: int=-1) -> None`。它主要围绕 `pooler`, `bos_token_id`, `eos_token_id` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `BOSEOSFilter.get_supported_tasks` (lines 166-167)
```python
    def get_supported_tasks(self) -> Set[PoolingTask]:
        return self.pooler.get_supported_tasks()
```
**EN:** Defines function `BOSEOSFilter.get_supported_tasks` with signature `get_supported_tasks(self) -> Set[PoolingTask]`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include `self.pooler.get_supported_tasks`.
**CN:** 定义函数 `BOSEOSFilter.get_supported_tasks`，其签名为 `get_supported_tasks(self) -> Set[PoolingTask]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.pooler.get_supported_tasks`。

### Method `BOSEOSFilter.get_pooling_updates` (lines 169-170)
```python
    def get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate:
        return PoolingParamsUpdate(requires_token_ids=True)
```
**EN:** Defines function `BOSEOSFilter.get_pooling_updates` with signature `get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate`. It mainly works with `task`; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include `PoolingParamsUpdate`.
**CN:** 定义函数 `BOSEOSFilter.get_pooling_updates`，其签名为 `get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate`。它主要围绕 `task` 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `PoolingParamsUpdate`。

### Method `BOSEOSFilter.forward` (lines 172-195)
```python
    def forward(
        self,
        hidden_states: torch.Tensor | list[torch.Tensor],
        pooling_metadata: PoolingMetadata,
    ) -> PoolerOutput:
        pooled_outputs = self.pooler(hidden_states, pooling_metadata)
        assert isinstance(pooled_outputs, list)
        prompt_token_ids = pooling_metadata.get_prompt_token_ids_cpu()

        for i, (prompt_len, token_ids) in enumerate(
            zip(pooling_metadata.prompt_lens, prompt_token_ids)
        ):
            pooled_data = pooled_outputs[i]
            assert (
                isinstance(pooled_data, torch.Tensor)
                and pooled_data.shape[0] == prompt_len
            )
            if int(token_ids[0]) == self.bos_token_id:
                pooled_data = pooled_data[1:]
            if int(token_ids[-1]) == self.eos_token_id:
                pooled_data = pooled_data[:-1]
            pooled_outputs[i] = pooled_data.squeeze(-1)

        return pooled_outputs
```
**EN:** Defines function `BOSEOSFilter.forward` with signature `forward(self, hidden_states: torch.Tensor | list[torch.Tensor], pooling_metadata: PoolingMetadata) -> PoolerOutput`. It mainly works with `hidden_states`, `pooling_metadata`; runs the main forward-path computation. The body uses branching, iteration, validation/error handling. Key calls include `self.pooler`, `isinstance`, `pooling_metadata.get_prompt_token_ids_cpu`, `enumerate`, `zip`, `pooled_data.squeeze`.
**CN:** 定义函数 `BOSEOSFilter.forward`，其签名为 `forward(self, hidden_states: torch.Tensor | list[torch.Tensor], pooling_metadata: PoolingMetadata) -> PoolerOutput`。它主要围绕 `hidden_states`, `pooling_metadata` 展开；执行主要的前向计算路径。函数体包含分支判断、循环处理、校验或报错逻辑。关键调用包括 `self.pooler`, `isinstance`, `pooling_metadata.get_prompt_token_ids_cpu`, `enumerate`, `zip`, `pooled_data.squeeze`。

### Class `BgeM3Pooler` overview (lines 198-233)
```python
class BgeM3Pooler(Pooler):
    def __init__(self, token_classify_pooler: Pooler, embed_pooler: Pooler) -> None:
        super().__init__()
        self.token_classify_pooler = token_classify_pooler
        self.embed_pooler = embed_pooler

    def forward(
        self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata
    ) -> PoolerOutput:
        embed_outputs = self.embed_pooler(hidden_states, pooling_metadata)
        token_classify_outputs = self.token_classify_pooler(
            hidden_states, pooling_metadata
        )
        pooler_outputs: list[torch.Tensor] = []
        for embed_output, token_classify_output in zip(
            embed_outputs, token_classify_outputs
        ):
            pooler_outputs.append(
                torch.cat(
                    [embed_output.view(-1), token_classify_output.view(-1)], dim=-1
                )
            )

        return pooler_outputs
```
**EN:** Defines class `BgeM3Pooler` with base classes `Pooler` and decorators none. It acts as a pooling implementation and exposes 5 direct methods, with notable entries `__init__`, `forward`, `get_supported_tasks`, `get_pooling_updates`, `extra_repr`.
**CN:** 定义类 `BgeM3Pooler`，其基类为 `Pooler`，装饰器为 无。它在整体实现中充当池化实现，并直接暴露 5 个方法，较重要的包括 `__init__`, `forward`, `get_supported_tasks`, `get_pooling_updates`, `extra_repr`。

### Method `BgeM3Pooler.__init__` (lines 199-202)
```python
    def __init__(self, token_classify_pooler: Pooler, embed_pooler: Pooler) -> None:
        super().__init__()
        self.token_classify_pooler = token_classify_pooler
        self.embed_pooler = embed_pooler
```
**EN:** Defines function `BgeM3Pooler.__init__` with signature `__init__(self, token_classify_pooler: Pooler, embed_pooler: Pooler) -> None`. It mainly works with `token_classify_pooler`, `embed_pooler`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `BgeM3Pooler.__init__`，其签名为 `__init__(self, token_classify_pooler: Pooler, embed_pooler: Pooler) -> None`。它主要围绕 `token_classify_pooler`, `embed_pooler` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `BgeM3Pooler.forward` (lines 204-221)
```python
    def forward(
        self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata
    ) -> PoolerOutput:
        embed_outputs = self.embed_pooler(hidden_states, pooling_metadata)
        token_classify_outputs = self.token_classify_pooler(
            hidden_states, pooling_metadata
        )
        pooler_outputs: list[torch.Tensor] = []
        for embed_output, token_classify_output in zip(
            embed_outputs, token_classify_outputs
        ):
            pooler_outputs.append(
                torch.cat(
                    [embed_output.view(-1), token_classify_output.view(-1)], dim=-1
                )
            )

        return pooler_outputs
```
**EN:** Defines function `BgeM3Pooler.forward` with signature `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> PoolerOutput`. It mainly works with `hidden_states`, `pooling_metadata`; runs the main forward-path computation. The body uses iteration, tensor/kernel operations. Key calls include `self.embed_pooler`, `self.token_classify_pooler`, `zip`, `pooler_outputs.append`, `torch.cat`, `embed_output.view`.
**CN:** 定义函数 `BgeM3Pooler.forward`，其签名为 `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> PoolerOutput`。它主要围绕 `hidden_states`, `pooling_metadata` 展开；执行主要的前向计算路径。函数体包含循环处理、张量或内核操作。关键调用包括 `self.embed_pooler`, `self.token_classify_pooler`, `zip`, `pooler_outputs.append`, `torch.cat`, `embed_output.view`。

### Method `BgeM3Pooler.get_supported_tasks` (lines 223-224)
```python
    def get_supported_tasks(self) -> Set[PoolingTask]:
        return {"embed&token_classify"}
```
**EN:** Defines function `BgeM3Pooler.get_supported_tasks` with signature `get_supported_tasks(self) -> Set[PoolingTask]`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `BgeM3Pooler.get_supported_tasks`，其签名为 `get_supported_tasks(self) -> Set[PoolingTask]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `BgeM3Pooler.get_pooling_updates` (lines 226-229)
```python
    def get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate:
        return self.embed_pooler.get_pooling_updates(
            "embed"
        ) | self.token_classify_pooler.get_pooling_updates("token_classify")
```
**EN:** Defines function `BgeM3Pooler.get_pooling_updates` with signature `get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate`. It mainly works with `task`; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include `self.embed_pooler.get_pooling_updates`, `self.token_classify_pooler.get_pooling_updates`.
**CN:** 定义函数 `BgeM3Pooler.get_pooling_updates`，其签名为 `get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate`。它主要围绕 `task` 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.embed_pooler.get_pooling_updates`, `self.token_classify_pooler.get_pooling_updates`。

### Method `BgeM3Pooler.extra_repr` (lines 231-233)
```python
    def extra_repr(self) -> str:
        s = f"supported_task={self.get_supported_tasks()}"
        return s
```
**EN:** Defines function `BgeM3Pooler.extra_repr` with signature `extra_repr(self) -> str`. It mainly works with object context only; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include `self.get_supported_tasks`.
**CN:** 定义函数 `BgeM3Pooler.extra_repr`，其签名为 `extra_repr(self) -> str`。它主要围绕 仅依赖对象上下文 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.get_supported_tasks`。

## Key Concepts / 关键概念
- **EN:** The file is part of the pooling stack that converts token-level hidden states into sequence-level or token-wise outputs.
  **CN:** 该文件属于池化栈，用于把 token 级隐藏状态转换为序列级或逐 token 输出。
- **EN:** Top-level classes include `DispatchPooler`, `IdentityPooler`, `BOSEOSFilter`, `BgeM3Pooler`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `DispatchPooler`, `IdentityPooler`, `BOSEOSFilter`, `BgeM3Pooler`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `dataclasses`, `collections`, `itertools`, `torch`
- **Internal / 内部**: `vllm.config`, `vllm.model_executor.layers.pooler`, `vllm.tasks`, `vllm.v1.pool.metadata`, `.abstract`, `.common`, `.seqwise`, `.tokwise`
