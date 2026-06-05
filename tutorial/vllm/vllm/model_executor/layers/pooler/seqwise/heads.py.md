# heads.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/pooler/seqwise/heads.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `SequencePoolerHead`, `EmbeddingPoolerHead`, `ClassifierPoolerHead` for pooling heads and methods. / 实现 `SequencePoolerHead`, `EmbeddingPoolerHead`, `ClassifierPoolerHead`，用于池化头与池化方法。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 3-14)
```python
from abc import ABC, abstractmethod
from collections.abc import Set
from typing import TypeAlias

import torch
import torch.nn as nn

from vllm.model_executor.layers.pooler import ActivationFn, ClassifierFn, ProjectorFn
from vllm.tasks import PoolingTask
from vllm.v1.pool.metadata import PoolingMetadata

from .methods import SequencePoolingMethodOutput
```
**EN:** This opening block pulls in external dependencies such as `abc`, `collections`, `typing`, `torch` and internal modules such as `vllm.model_executor.layers.pooler`, `vllm.tasks`, `vllm.v1.pool.metadata`, `.methods`. That import mix shows the file is part of the pooling heads and methods stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `abc`, `collections`, `typing`, `torch`）以及内部模块（如 `vllm.model_executor.layers.pooler`, `vllm.tasks`, `vllm.v1.pool.metadata`, `.methods`）。这些导入关系表明该文件属于池化头与池化方法栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 16-16)
```python
SequencePoolerHeadOutput: TypeAlias = torch.Tensor | list[torch.Tensor]
```
**EN:** This block defines module-level metadata or constants such as `SequencePoolerHeadOutput`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the pooling heads and methods pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `SequencePoolerHeadOutput`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在池化头与池化方法流程中复用。

### Class `SequencePoolerHead` overview (lines 19-30)
```python
class SequencePoolerHead(nn.Module, ABC):
    @abstractmethod
    def get_supported_tasks(self) -> Set[PoolingTask]:
        raise NotImplementedError

    @abstractmethod
    def forward(
        self,
        pooled_data: SequencePoolingMethodOutput,
        pooling_metadata: PoolingMetadata,
    ) -> SequencePoolerHeadOutput:
        raise NotImplementedError
```
**EN:** Defines class `SequencePoolerHead` with base classes `nn.Module`, `ABC` and decorators none. It acts as a head that projects hidden states into task outputs and exposes 2 direct methods, with notable entries `get_supported_tasks`, `forward`.
**CN:** 定义类 `SequencePoolerHead`，其基类为 `nn.Module`, `ABC`，装饰器为 无。它在整体实现中充当把隐藏状态映射到任务输出的头部模块，并直接暴露 2 个方法，较重要的包括 `get_supported_tasks`, `forward`。

### Method `SequencePoolerHead.get_supported_tasks` (lines 21-22)
```python
    def get_supported_tasks(self) -> Set[PoolingTask]:
        raise NotImplementedError
```
**EN:** Defines function `SequencePoolerHead.get_supported_tasks` with signature `get_supported_tasks(self) -> Set[PoolingTask]`. It mainly works with object context only; returns a derived property or capability check. The body uses validation/error handling. Key calls include no notable helper calls.
**CN:** 定义函数 `SequencePoolerHead.get_supported_tasks`，其签名为 `get_supported_tasks(self) -> Set[PoolingTask]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含校验或报错逻辑。关键调用包括 无显著辅助调用。

### Method `SequencePoolerHead.forward` (lines 25-30)
```python
    def forward(
        self,
        pooled_data: SequencePoolingMethodOutput,
        pooling_metadata: PoolingMetadata,
    ) -> SequencePoolerHeadOutput:
        raise NotImplementedError
```
**EN:** Defines function `SequencePoolerHead.forward` with signature `forward(self, pooled_data: SequencePoolingMethodOutput, pooling_metadata: PoolingMetadata) -> SequencePoolerHeadOutput`. It mainly works with `pooled_data`, `pooling_metadata`; runs the main forward-path computation. The body uses validation/error handling. Key calls include no notable helper calls.
**CN:** 定义函数 `SequencePoolerHead.forward`，其签名为 `forward(self, pooled_data: SequencePoolingMethodOutput, pooling_metadata: PoolingMetadata) -> SequencePoolerHeadOutput`。它主要围绕 `pooled_data`, `pooling_metadata` 展开；执行主要的前向计算路径。函数体包含校验或报错逻辑。关键调用包括 无显著辅助调用。

### Class `EmbeddingPoolerHead` overview (lines 33-99)
```python
class EmbeddingPoolerHead(SequencePoolerHead):
    def __init__(
        self,
        projector: ProjectorFn | None = None,
        head_dtype: torch.dtype | str | None = None,
        activation: ActivationFn | None = None,
    ) -> None:
        super().__init__()

        self.projector = projector
        self.head_dtype = head_dtype
        self.activation = activation

    def get_supported_tasks(self) -> Set[PoolingTask]:
        return {"embed"}

    def forward(
        self,
        pooled_data: SequencePoolingMethodOutput,
        pooling_metadata: PoolingMetadata,
    ) -> SequencePoolerHeadOutput:
        pooling_params = pooling_metadata.pooling_params
        assert len(pooled_data) == len(pooling_params)

        if isinstance(pooled_data, list):
```
**EN:** Defines class `EmbeddingPoolerHead` with base classes `SequencePoolerHead` and decorators none. It acts as a head that projects hidden states into task outputs and exposes 3 direct methods, with notable entries `__init__`, `get_supported_tasks`, `forward`.
**CN:** 定义类 `EmbeddingPoolerHead`，其基类为 `SequencePoolerHead`，装饰器为 无。它在整体实现中充当把隐藏状态映射到任务输出的头部模块，并直接暴露 3 个方法，较重要的包括 `__init__`, `get_supported_tasks`, `forward`。

### Method `EmbeddingPoolerHead.__init__` (lines 34-44)
```python
    def __init__(
        self,
        projector: ProjectorFn | None = None,
        head_dtype: torch.dtype | str | None = None,
        activation: ActivationFn | None = None,
    ) -> None:
        super().__init__()

        self.projector = projector
        self.head_dtype = head_dtype
        self.activation = activation
```
**EN:** Defines function `EmbeddingPoolerHead.__init__` with signature `__init__(self, projector: ProjectorFn | None=None, head_dtype: torch.dtype | str | None=None, activation: ActivationFn | None=None) -> None`. It mainly works with `projector`, `head_dtype`, `activation`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `EmbeddingPoolerHead.__init__`，其签名为 `__init__(self, projector: ProjectorFn | None=None, head_dtype: torch.dtype | str | None=None, activation: ActivationFn | None=None) -> None`。它主要围绕 `projector`, `head_dtype`, `activation` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `EmbeddingPoolerHead.get_supported_tasks` (lines 46-47)
```python
    def get_supported_tasks(self) -> Set[PoolingTask]:
        return {"embed"}
```
**EN:** Defines function `EmbeddingPoolerHead.get_supported_tasks` with signature `get_supported_tasks(self) -> Set[PoolingTask]`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `EmbeddingPoolerHead.get_supported_tasks`，其签名为 `get_supported_tasks(self) -> Set[PoolingTask]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `EmbeddingPoolerHead.forward` (lines 49-99)
```python
    def forward(
        self,
        pooled_data: SequencePoolingMethodOutput,
        pooling_metadata: PoolingMetadata,
    ) -> SequencePoolerHeadOutput:
        pooling_params = pooling_metadata.pooling_params
        assert len(pooled_data) == len(pooling_params)

        if isinstance(pooled_data, list):
            pooled_data = torch.stack(pooled_data)
        # pooled_data shape: [batchsize, hidden_size]

        if self.head_dtype is not None:
            pooled_data = pooled_data.to(self.head_dtype)

        # Apply ST projector
        if self.projector is not None:
            embeddings = self.projector(pooled_data)
        else:
            embeddings = pooled_data
        # embeddings shape: [batchsize, embedding_size]

        # for matryoshka representation
        dimensions_list = [pooling_param.dimensions for pooling_param in pooling_params]
        if any(d is not None for d in dimensions_list):
            # change the output dimension
            assert len(embeddings) == len(dimensions_list)
            if len(set(dimensions_list)) == 1 and not isinstance(embeddings, list):
                # if all dimensions are the same
                d = dimensions_list[0]
                embeddings = embeddings[..., :d]
            else:
                embeddings = [
                    vecs if d is None else vecs[..., :d]
                    for vecs, d in zip(embeddings, dimensions_list)
                ]

        # for normalize
        if self.activation is not None:
            flags = [p.use_activation for p in pooling_params]
            if len(set(flags)) == 1:
                if flags[0]:
                    embeddings = self.activation(embeddings)
            else:
                embeddings = [
                    self.activation(vecs) if f else vecs
                    for vecs, f in zip(embeddings, flags)
                ]

        # embeddings shape: [batchsize, embedding_size]
        return embeddings
```
**EN:** Defines function `EmbeddingPoolerHead.forward` with signature `forward(self, pooled_data: SequencePoolingMethodOutput, pooling_metadata: PoolingMetadata) -> SequencePoolerHeadOutput`. It mainly works with `pooled_data`, `pooling_metadata`; runs the main forward-path computation. The body uses branching, comprehensions, validation/error handling, tensor/kernel operations. Key calls include `isinstance`, `any`, `len`, `torch.stack`, `pooled_data.to`, `self.projector`.
**CN:** 定义函数 `EmbeddingPoolerHead.forward`，其签名为 `forward(self, pooled_data: SequencePoolingMethodOutput, pooling_metadata: PoolingMetadata) -> SequencePoolerHeadOutput`。它主要围绕 `pooled_data`, `pooling_metadata` 展开；执行主要的前向计算路径。函数体包含分支判断、推导式、校验或报错逻辑、张量或内核操作。关键调用包括 `isinstance`, `any`, `len`, `torch.stack`, `pooled_data.to`, `self.projector`。

### Class `ClassifierPoolerHead` overview (lines 102-160)
```python
class ClassifierPoolerHead(SequencePoolerHead):
    def __init__(
        self,
        classifier: ClassifierFn | None = None,
        logit_mean: float | None = None,
        logit_sigma: float | None = None,
        head_dtype: torch.dtype | str | None = None,
        activation: ActivationFn | None = None,
    ) -> None:
        super().__init__()

        self.classifier = classifier
        self.logit_mean = logit_mean
        self.logit_sigma = logit_sigma
        self.head_dtype = head_dtype
        self.activation = activation

    def get_supported_tasks(self) -> Set[PoolingTask]:
        return {"classify"}

    def forward(
        self,
        pooled_data: SequencePoolingMethodOutput,
        pooling_metadata: PoolingMetadata,
    ) -> SequencePoolerHeadOutput:
```
**EN:** Defines class `ClassifierPoolerHead` with base classes `SequencePoolerHead` and decorators none. It acts as a head that projects hidden states into task outputs and exposes 3 direct methods, with notable entries `__init__`, `get_supported_tasks`, `forward`.
**CN:** 定义类 `ClassifierPoolerHead`，其基类为 `SequencePoolerHead`，装饰器为 无。它在整体实现中充当把隐藏状态映射到任务输出的头部模块，并直接暴露 3 个方法，较重要的包括 `__init__`, `get_supported_tasks`, `forward`。

### Method `ClassifierPoolerHead.__init__` (lines 103-117)
```python
    def __init__(
        self,
        classifier: ClassifierFn | None = None,
        logit_mean: float | None = None,
        logit_sigma: float | None = None,
        head_dtype: torch.dtype | str | None = None,
        activation: ActivationFn | None = None,
    ) -> None:
        super().__init__()

        self.classifier = classifier
        self.logit_mean = logit_mean
        self.logit_sigma = logit_sigma
        self.head_dtype = head_dtype
        self.activation = activation
```
**EN:** Defines function `ClassifierPoolerHead.__init__` with signature `__init__(self, classifier: ClassifierFn | None=None, logit_mean: float | None=None, logit_sigma: float | None=None, head_dtype: torch.dtype | str | None=None, activation: ActivationFn | None=None) -> None`. It mainly works with `classifier`, `logit_mean`, `logit_sigma`, `head_dtype`, `activation`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `ClassifierPoolerHead.__init__`，其签名为 `__init__(self, classifier: ClassifierFn | None=None, logit_mean: float | None=None, logit_sigma: float | None=None, head_dtype: torch.dtype | str | None=None, activation: ActivationFn | None=None) -> None`。它主要围绕 `classifier`, `logit_mean`, `logit_sigma`, `head_dtype`, `activation` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `ClassifierPoolerHead.get_supported_tasks` (lines 119-120)
```python
    def get_supported_tasks(self) -> Set[PoolingTask]:
        return {"classify"}
```
**EN:** Defines function `ClassifierPoolerHead.get_supported_tasks` with signature `get_supported_tasks(self) -> Set[PoolingTask]`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `ClassifierPoolerHead.get_supported_tasks`，其签名为 `get_supported_tasks(self) -> Set[PoolingTask]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `ClassifierPoolerHead.forward` (lines 122-160)
```python
    def forward(
        self,
        pooled_data: SequencePoolingMethodOutput,
        pooling_metadata: PoolingMetadata,
    ) -> SequencePoolerHeadOutput:
        pooling_params = pooling_metadata.pooling_params
        assert len(pooled_data) == len(pooling_params)

        if isinstance(pooled_data, list):
            pooled_data = torch.stack(pooled_data)
        # pooled_data shape: [batchsize, hidden_size]

        if self.head_dtype is not None:
            pooled_data = pooled_data.to(self.head_dtype)

        if self.classifier is not None:
            logits = self.classifier(pooled_data)
        else:
            logits = pooled_data

        # logits shape: [batchsize, num_labels]
        # Affine score calibration: activation((logit - mean) / sigma)
        if self.logit_mean is not None:
            logits = logits - self.logit_mean
        if self.logit_sigma is not None:
            logits = logits / self.logit_sigma

        if self.activation is not None:
            flags = [p.use_activation for p in pooling_params]
            if len(set(flags)) == 1:
                logits = self.activation(logits) if flags[0] else logits
            else:
                logits = [
                    self.activation(vecs) if f else vecs
                    for vecs, f in zip(logits, flags)
                ]

        # logits shape: [batchsize, num_labels]
        return logits
```
**EN:** Defines function `ClassifierPoolerHead.forward` with signature `forward(self, pooled_data: SequencePoolingMethodOutput, pooling_metadata: PoolingMetadata) -> SequencePoolerHeadOutput`. It mainly works with `pooled_data`, `pooling_metadata`; runs the main forward-path computation. The body uses branching, comprehensions, validation/error handling, tensor/kernel operations. Key calls include `isinstance`, `len`, `torch.stack`, `pooled_data.to`, `self.classifier`, `set`.
**CN:** 定义函数 `ClassifierPoolerHead.forward`，其签名为 `forward(self, pooled_data: SequencePoolingMethodOutput, pooling_metadata: PoolingMetadata) -> SequencePoolerHeadOutput`。它主要围绕 `pooled_data`, `pooling_metadata` 展开；执行主要的前向计算路径。函数体包含分支判断、推导式、校验或报错逻辑、张量或内核操作。关键调用包括 `isinstance`, `len`, `torch.stack`, `pooled_data.to`, `self.classifier`, `set`。

## Key Concepts / 关键概念
- **EN:** The file is part of the pooling stack that converts token-level hidden states into sequence-level or token-wise outputs.
  **CN:** 该文件属于池化栈，用于把 token 级隐藏状态转换为序列级或逐 token 输出。
- **EN:** Top-level classes include `SequencePoolerHead`, `EmbeddingPoolerHead`, `ClassifierPoolerHead`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `SequencePoolerHead`, `EmbeddingPoolerHead`, `ClassifierPoolerHead`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `abc`, `collections`, `typing`, `torch`
- **Internal / 内部**: `vllm.model_executor.layers.pooler`, `vllm.tasks`, `vllm.v1.pool.metadata`, `.methods`
