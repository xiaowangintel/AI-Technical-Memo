# poolers.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/pooler/seqwise/poolers.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `SequencePooler` for pooling heads and methods. / 实现 `SequencePooler`，用于池化头与池化方法。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 3-30)
```python
from collections.abc import Callable, Set
from typing import TypeAlias

import torch

from vllm.config import PoolerConfig, get_current_vllm_config
from vllm.model_executor.layers.pooler import ClassifierFn, PoolingParamsUpdate
from vllm.model_executor.layers.pooler.abstract import Pooler
from vllm.model_executor.layers.pooler.activations import (
    PoolerActivation,
    PoolerNormalize,
    resolve_classifier_act_fn,
)
from vllm.model_executor.models.adapters import _load_st_projector
from vllm.tasks import POOLING_TASKS, PoolingTask
from vllm.v1.pool.metadata import PoolingMetadata

from .heads import (
    ClassifierPoolerHead,
    EmbeddingPoolerHead,
    SequencePoolerHead,
    SequencePoolerHeadOutput,
)
from .methods import (
    SequencePoolingMethod,
    SequencePoolingMethodOutput,
    get_seq_pooling_method,
)
```
**EN:** This opening block pulls in external dependencies such as `collections`, `typing`, `torch` and internal modules such as `vllm.config`, `vllm.model_executor.layers.pooler`, `vllm.model_executor.layers.pooler.abstract`, `vllm.model_executor.layers.pooler.activations`, `vllm.model_executor.models.adapters`, `vllm.tasks`. That import mix shows the file is part of the pooling heads and methods stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `collections`, `typing`, `torch`）以及内部模块（如 `vllm.config`, `vllm.model_executor.layers.pooler`, `vllm.model_executor.layers.pooler.abstract`, `vllm.model_executor.layers.pooler.activations`, `vllm.model_executor.models.adapters`, `vllm.tasks`）。这些导入关系表明该文件属于池化头与池化方法栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 32-41)
```python
SequencePoolingFn: TypeAlias = Callable[
    [torch.Tensor, PoolingMetadata],
    SequencePoolingMethodOutput,
]
SequencePoolingHeadFn: TypeAlias = Callable[
    [SequencePoolingMethodOutput, PoolingMetadata],
    SequencePoolerHeadOutput,
]

SequencePoolerOutput: TypeAlias = torch.Tensor | list[torch.Tensor]
```
**EN:** This block defines module-level metadata or constants such as `SequencePoolingFn`, `SequencePoolingHeadFn`, `SequencePoolerOutput`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the pooling heads and methods pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `SequencePoolingFn`, `SequencePoolingHeadFn`, `SequencePoolerOutput`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在池化头与池化方法流程中复用。

### Function `pooler_for_embed` (lines 92-103)
```python
def pooler_for_embed(pooler_config: PoolerConfig):
    pooling = get_seq_pooling_method(pooler_config.get_seq_pooling_type())

    vllm_config = get_current_vllm_config()
    model_config = vllm_config.model_config
    head = EmbeddingPoolerHead(
        head_dtype=model_config.head_dtype,
        projector=_load_st_projector(model_config),
        activation=PoolerNormalize(),
    )

    return SequencePooler(pooling=pooling, head=head)
```
**EN:** Defines function `pooler_for_embed` with signature `pooler_for_embed(pooler_config: PoolerConfig)`. It mainly works with `pooler_config`; reduces token features into pooled outputs. The body uses mostly straightforward data movement and object wiring. Key calls include `get_seq_pooling_method`, `get_current_vllm_config`, `EmbeddingPoolerHead`, `SequencePooler`, `pooler_config.get_seq_pooling_type`, `_load_st_projector`.
**CN:** 定义函数 `pooler_for_embed`，其签名为 `pooler_for_embed(pooler_config: PoolerConfig)`。它主要围绕 `pooler_config` 展开；将 token 特征汇聚为池化输出。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `get_seq_pooling_method`, `get_current_vllm_config`, `EmbeddingPoolerHead`, `SequencePooler`, `pooler_config.get_seq_pooling_type`, `_load_st_projector`。

### Function `pooler_for_classify` (lines 106-129)
```python
def pooler_for_classify(
    pooler_config: PoolerConfig,
    *,
    pooling: SequencePoolingMethod | SequencePoolingFn | None = None,
    classifier: ClassifierFn | None = None,
    act_fn: PoolerActivation | None = None,
):
    if pooling is None:
        pooling = get_seq_pooling_method(pooler_config.get_seq_pooling_type())

    vllm_config = get_current_vllm_config()
    model_config = vllm_config.model_config
    assert model_config.pooler_config is not None
    head = ClassifierPoolerHead(
        head_dtype=model_config.head_dtype,
        classifier=classifier,
        logit_mean=model_config.pooler_config.logit_mean,
        logit_sigma=model_config.pooler_config.logit_sigma,
        activation=resolve_classifier_act_fn(
            model_config, static_num_labels=True, act_fn=act_fn
        ),
    )

    return SequencePooler(pooling=pooling, head=head)
```
**EN:** Defines function `pooler_for_classify` with signature `pooler_for_classify(pooler_config: PoolerConfig, *, pooling: SequencePoolingMethod | SequencePoolingFn | None=None, classifier: ClassifierFn | None=None, act_fn: PoolerActivation | None=None)`. It mainly works with `pooler_config`, `pooling`, `classifier`, `act_fn`; reduces token features into pooled outputs. The body uses branching, validation/error handling. Key calls include `get_current_vllm_config`, `ClassifierPoolerHead`, `SequencePooler`, `get_seq_pooling_method`, `pooler_config.get_seq_pooling_type`, `resolve_classifier_act_fn`.
**CN:** 定义函数 `pooler_for_classify`，其签名为 `pooler_for_classify(pooler_config: PoolerConfig, *, pooling: SequencePoolingMethod | SequencePoolingFn | None=None, classifier: ClassifierFn | None=None, act_fn: PoolerActivation | None=None)`。它主要围绕 `pooler_config`, `pooling`, `classifier`, `act_fn` 展开；将 token 特征汇聚为池化输出。函数体包含分支判断、校验或报错逻辑。关键调用包括 `get_current_vllm_config`, `ClassifierPoolerHead`, `SequencePooler`, `get_seq_pooling_method`, `pooler_config.get_seq_pooling_type`, `resolve_classifier_act_fn`。

### Class `SequencePooler` overview (lines 44-89)
```python
class SequencePooler(Pooler):
    """
    A layer that pools specific information from hidden states.

    This layer does the following:
    1. Extracts specific tokens or aggregates data based on pooling method.
    2. Postprocesses the output based on pooling head.
    3. Returns structured results as `PoolerOutput`.
    """

    def __init__(
        self,
        pooling: SequencePoolingMethod | SequencePoolingFn,
        head: SequencePoolerHead | SequencePoolingHeadFn,
    ) -> None:
        super().__init__()

        self.pooling = pooling
        self.head = head

    def get_supported_tasks(self) -> Set[PoolingTask]:
        tasks = set(POOLING_TASKS)

        if isinstance(self.pooling, SequencePoolingMethod):
            tasks &= self.pooling.get_supported_tasks()
```
**EN:** Defines class `SequencePooler` with base classes `Pooler` and decorators none. It acts as a pooling implementation and exposes 4 direct methods, with notable entries `__init__`, `get_supported_tasks`, `get_pooling_updates`, `forward`. Its docstring says: A layer that pools specific information from hidden states.
**CN:** 定义类 `SequencePooler`，其基类为 `Pooler`，装饰器为 无。它在整体实现中充当池化实现，并直接暴露 4 个方法，较重要的包括 `__init__`, `get_supported_tasks`, `get_pooling_updates`, `forward`。 文档字符串进一步说明了该类的定位。

### Method `SequencePooler.__init__` (lines 54-62)
```python
    def __init__(
        self,
        pooling: SequencePoolingMethod | SequencePoolingFn,
        head: SequencePoolerHead | SequencePoolingHeadFn,
    ) -> None:
        super().__init__()

        self.pooling = pooling
        self.head = head
```
**EN:** Defines function `SequencePooler.__init__` with signature `__init__(self, pooling: SequencePoolingMethod | SequencePoolingFn, head: SequencePoolerHead | SequencePoolingHeadFn) -> None`. It mainly works with `pooling`, `head`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `SequencePooler.__init__`，其签名为 `__init__(self, pooling: SequencePoolingMethod | SequencePoolingFn, head: SequencePoolerHead | SequencePoolingHeadFn) -> None`。它主要围绕 `pooling`, `head` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `SequencePooler.get_supported_tasks` (lines 64-72)
```python
    def get_supported_tasks(self) -> Set[PoolingTask]:
        tasks = set(POOLING_TASKS)

        if isinstance(self.pooling, SequencePoolingMethod):
            tasks &= self.pooling.get_supported_tasks()
        if isinstance(self.head, SequencePoolerHead):
            tasks &= self.head.get_supported_tasks()

        return tasks
```
**EN:** Defines function `SequencePooler.get_supported_tasks` with signature `get_supported_tasks(self) -> Set[PoolingTask]`. It mainly works with object context only; returns a derived property or capability check. The body uses branching. Key calls include `set`, `isinstance`, `self.pooling.get_supported_tasks`, `self.head.get_supported_tasks`.
**CN:** 定义函数 `SequencePooler.get_supported_tasks`，其签名为 `get_supported_tasks(self) -> Set[PoolingTask]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含分支判断。关键调用包括 `set`, `isinstance`, `self.pooling.get_supported_tasks`, `self.head.get_supported_tasks`。

### Method `SequencePooler.get_pooling_updates` (lines 74-80)
```python
    def get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate:
        updates = PoolingParamsUpdate()

        if isinstance(self.pooling, SequencePoolingMethod):
            updates |= self.pooling.get_pooling_updates(task)

        return updates
```
**EN:** Defines function `SequencePooler.get_pooling_updates` with signature `get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate`. It mainly works with `task`; returns a derived property or capability check. The body uses branching. Key calls include `PoolingParamsUpdate`, `isinstance`, `self.pooling.get_pooling_updates`.
**CN:** 定义函数 `SequencePooler.get_pooling_updates`，其签名为 `get_pooling_updates(self, task: PoolingTask) -> PoolingParamsUpdate`。它主要围绕 `task` 展开；返回派生属性或能力判断结果。函数体包含分支判断。关键调用包括 `PoolingParamsUpdate`, `isinstance`, `self.pooling.get_pooling_updates`。

### Method `SequencePooler.forward` (lines 82-89)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        pooling_metadata: PoolingMetadata,
    ) -> SequencePoolerOutput:
        pooled_data = self.pooling(hidden_states, pooling_metadata)
        pooled_data = self.head(pooled_data, pooling_metadata)
        return pooled_data
```
**EN:** Defines function `SequencePooler.forward` with signature `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> SequencePoolerOutput`. It mainly works with `hidden_states`, `pooling_metadata`; runs the main forward-path computation. The body uses mostly straightforward data movement and object wiring. Key calls include `self.pooling`, `self.head`.
**CN:** 定义函数 `SequencePooler.forward`，其签名为 `forward(self, hidden_states: torch.Tensor, pooling_metadata: PoolingMetadata) -> SequencePoolerOutput`。它主要围绕 `hidden_states`, `pooling_metadata` 展开；执行主要的前向计算路径。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.pooling`, `self.head`。

## Key Concepts / 关键概念
- **EN:** The file is part of the pooling stack that converts token-level hidden states into sequence-level or token-wise outputs.
  **CN:** 该文件属于池化栈，用于把 token 级隐藏状态转换为序列级或逐 token 输出。
- **EN:** Top-level classes include `SequencePooler`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `SequencePooler`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `pooler_for_embed`, `pooler_for_classify` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `pooler_for_embed`, `pooler_for_classify` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `collections`, `typing`, `torch`
- **Internal / 内部**: `vllm.config`, `vllm.model_executor.layers.pooler`, `vllm.model_executor.layers.pooler.abstract`, `vllm.model_executor.layers.pooler.activations`, `vllm.model_executor.models.adapters`, `vllm.tasks`, `vllm.v1.pool.metadata`, `.heads`, `.methods`
