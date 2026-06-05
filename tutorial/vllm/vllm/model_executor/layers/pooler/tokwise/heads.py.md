# heads.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/pooler/tokwise/heads.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `TokenPoolerHead`, `TokenEmbeddingPoolerHead`, `TokenClassifierPoolerHead` for pooling heads and methods. / 实现 `TokenPoolerHead`, `TokenEmbeddingPoolerHead`, `TokenClassifierPoolerHead`，用于池化头与池化方法。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 3-15)
```python
from abc import ABC, abstractmethod
from collections.abc import Set
from typing import TypeAlias

import torch
import torch.nn as nn

from vllm.model_executor.layers.pooler import ActivationFn, ClassifierFn, ProjectorFn
from vllm.pooling_params import PoolingParams
from vllm.tasks import PoolingTask
from vllm.v1.pool.metadata import PoolingMetadata

from .methods import TokenPoolingMethodOutputItem
```
**EN:** This opening block pulls in external dependencies such as `abc`, `collections`, `typing`, `torch` and internal modules such as `vllm.model_executor.layers.pooler`, `vllm.pooling_params`, `vllm.tasks`, `vllm.v1.pool.metadata`, `.methods`. That import mix shows the file is part of the pooling heads and methods stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `abc`, `collections`, `typing`, `torch`）以及内部模块（如 `vllm.model_executor.layers.pooler`, `vllm.pooling_params`, `vllm.tasks`, `vllm.v1.pool.metadata`, `.methods`）。这些导入关系表明该文件属于池化头与池化方法栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 17-17)
```python
TokenPoolerHeadOutputItem: TypeAlias = torch.Tensor | None
```
**EN:** This block defines module-level metadata or constants such as `TokenPoolerHeadOutputItem`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the pooling heads and methods pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `TokenPoolerHeadOutputItem`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在池化头与池化方法流程中复用。

### Class `TokenPoolerHead` overview (lines 20-41)
```python
class TokenPoolerHead(nn.Module, ABC):
    @abstractmethod
    def get_supported_tasks(self) -> Set[PoolingTask]:
        raise NotImplementedError

    @abstractmethod
    def forward_chunk(
        self,
        pooled_data: TokenPoolingMethodOutputItem,
        pooling_param: PoolingParams,
    ) -> TokenPoolerHeadOutputItem:
        raise NotImplementedError

    def forward(
        self,
        pooled_data: list[TokenPoolingMethodOutputItem],
        pooling_metadata: PoolingMetadata,
    ) -> list[TokenPoolerHeadOutputItem]:
        pooling_params = pooling_metadata.pooling_params
        assert len(pooled_data) == len(pooling_params)

        return [self.forward_chunk(d, p) for d, p in zip(pooled_data, pooling_params)]
```
**EN:** Defines class `TokenPoolerHead` with base classes `nn.Module`, `ABC` and decorators none. It acts as a head that projects hidden states into task outputs and exposes 3 direct methods, with notable entries `get_supported_tasks`, `forward_chunk`, `forward`.
**CN:** 定义类 `TokenPoolerHead`，其基类为 `nn.Module`, `ABC`，装饰器为 无。它在整体实现中充当把隐藏状态映射到任务输出的头部模块，并直接暴露 3 个方法，较重要的包括 `get_supported_tasks`, `forward_chunk`, `forward`。

### Method `TokenPoolerHead.get_supported_tasks` (lines 22-23)
```python
    def get_supported_tasks(self) -> Set[PoolingTask]:
        raise NotImplementedError
```
**EN:** Defines function `TokenPoolerHead.get_supported_tasks` with signature `get_supported_tasks(self) -> Set[PoolingTask]`. It mainly works with object context only; returns a derived property or capability check. The body uses validation/error handling. Key calls include no notable helper calls.
**CN:** 定义函数 `TokenPoolerHead.get_supported_tasks`，其签名为 `get_supported_tasks(self) -> Set[PoolingTask]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含校验或报错逻辑。关键调用包括 无显著辅助调用。

### Method `TokenPoolerHead.forward_chunk` (lines 26-31)
```python
    def forward_chunk(
        self,
        pooled_data: TokenPoolingMethodOutputItem,
        pooling_param: PoolingParams,
    ) -> TokenPoolerHeadOutputItem:
        raise NotImplementedError
```
**EN:** Defines function `TokenPoolerHead.forward_chunk` with signature `forward_chunk(self, pooled_data: TokenPoolingMethodOutputItem, pooling_param: PoolingParams) -> TokenPoolerHeadOutputItem`. It mainly works with `pooled_data`, `pooling_param`; implements one step of the module control flow. The body uses validation/error handling. Key calls include no notable helper calls.
**CN:** 定义函数 `TokenPoolerHead.forward_chunk`，其签名为 `forward_chunk(self, pooled_data: TokenPoolingMethodOutputItem, pooling_param: PoolingParams) -> TokenPoolerHeadOutputItem`。它主要围绕 `pooled_data`, `pooling_param` 展开；实现模块控制流中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 无显著辅助调用。

### Method `TokenPoolerHead.forward` (lines 33-41)
```python
    def forward(
        self,
        pooled_data: list[TokenPoolingMethodOutputItem],
        pooling_metadata: PoolingMetadata,
    ) -> list[TokenPoolerHeadOutputItem]:
        pooling_params = pooling_metadata.pooling_params
        assert len(pooled_data) == len(pooling_params)

        return [self.forward_chunk(d, p) for d, p in zip(pooled_data, pooling_params)]
```
**EN:** Defines function `TokenPoolerHead.forward` with signature `forward(self, pooled_data: list[TokenPoolingMethodOutputItem], pooling_metadata: PoolingMetadata) -> list[TokenPoolerHeadOutputItem]`. It mainly works with `pooled_data`, `pooling_metadata`; runs the main forward-path computation. The body uses comprehensions, validation/error handling. Key calls include `len`, `self.forward_chunk`, `zip`.
**CN:** 定义函数 `TokenPoolerHead.forward`，其签名为 `forward(self, pooled_data: list[TokenPoolingMethodOutputItem], pooling_metadata: PoolingMetadata) -> list[TokenPoolerHeadOutputItem]`。它主要围绕 `pooled_data`, `pooling_metadata` 展开；执行主要的前向计算路径。函数体包含推导式、校验或报错逻辑。关键调用包括 `len`, `self.forward_chunk`, `zip`。

### Class `TokenEmbeddingPoolerHead` overview (lines 44-88)
```python
class TokenEmbeddingPoolerHead(TokenPoolerHead):
    def __init__(
        self,
        head_dtype: torch.dtype | str | None = None,
        projector: ProjectorFn | None = None,
        activation: ActivationFn | None = None,
    ) -> None:
        super().__init__()

        self.head_dtype = head_dtype
        self.projector = projector
        self.activation = activation

    def get_supported_tasks(self) -> Set[PoolingTask]:
        return {"token_embed"}

    def forward_chunk(
        self,
        pooled_data: TokenPoolingMethodOutputItem,
        pooling_param: PoolingParams,
    ) -> TokenPoolerHeadOutputItem:
        # for unfinished chunked prefill
        if pooled_data is None:
            return None
```
**EN:** Defines class `TokenEmbeddingPoolerHead` with base classes `TokenPoolerHead` and decorators none. It acts as a head that projects hidden states into task outputs and exposes 3 direct methods, with notable entries `__init__`, `get_supported_tasks`, `forward_chunk`.
**CN:** 定义类 `TokenEmbeddingPoolerHead`，其基类为 `TokenPoolerHead`，装饰器为 无。它在整体实现中充当把隐藏状态映射到任务输出的头部模块，并直接暴露 3 个方法，较重要的包括 `__init__`, `get_supported_tasks`, `forward_chunk`。

### Method `TokenEmbeddingPoolerHead.__init__` (lines 45-55)
```python
    def __init__(
        self,
        head_dtype: torch.dtype | str | None = None,
        projector: ProjectorFn | None = None,
        activation: ActivationFn | None = None,
    ) -> None:
        super().__init__()

        self.head_dtype = head_dtype
        self.projector = projector
        self.activation = activation
```
**EN:** Defines function `TokenEmbeddingPoolerHead.__init__` with signature `__init__(self, head_dtype: torch.dtype | str | None=None, projector: ProjectorFn | None=None, activation: ActivationFn | None=None) -> None`. It mainly works with `head_dtype`, `projector`, `activation`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `TokenEmbeddingPoolerHead.__init__`，其签名为 `__init__(self, head_dtype: torch.dtype | str | None=None, projector: ProjectorFn | None=None, activation: ActivationFn | None=None) -> None`。它主要围绕 `head_dtype`, `projector`, `activation` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `TokenEmbeddingPoolerHead.get_supported_tasks` (lines 57-58)
```python
    def get_supported_tasks(self) -> Set[PoolingTask]:
        return {"token_embed"}
```
**EN:** Defines function `TokenEmbeddingPoolerHead.get_supported_tasks` with signature `get_supported_tasks(self) -> Set[PoolingTask]`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `TokenEmbeddingPoolerHead.get_supported_tasks`，其签名为 `get_supported_tasks(self) -> Set[PoolingTask]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `TokenEmbeddingPoolerHead.forward_chunk` (lines 60-88)
```python
    def forward_chunk(
        self,
        pooled_data: TokenPoolingMethodOutputItem,
        pooling_param: PoolingParams,
    ) -> TokenPoolerHeadOutputItem:
        # for unfinished chunked prefill
        if pooled_data is None:
            return None

        if self.head_dtype is not None:
            pooled_data = pooled_data.to(self.head_dtype)
        # pooled_data shape: [n_tokens, hidden_size]

        # Apply ST projector
        if self.projector is not None:
            embeddings = self.projector(pooled_data)
        else:
            embeddings = pooled_data
        # embeddings shape: [n_tokens, embedding_size]

        # for matryoshka representation
        embeddings = embeddings[..., : pooling_param.dimensions]

        # for normalize
        if self.activation is not None and pooling_param.use_activation:
            embeddings = self.activation(embeddings)

        # embeddings shape: [n_tokens, embedding_size]
        return embeddings
```
**EN:** Defines function `TokenEmbeddingPoolerHead.forward_chunk` with signature `forward_chunk(self, pooled_data: TokenPoolingMethodOutputItem, pooling_param: PoolingParams) -> TokenPoolerHeadOutputItem`. It mainly works with `pooled_data`, `pooling_param`; implements one step of the module control flow. The body uses branching. Key calls include `pooled_data.to`, `self.projector`, `self.activation`.
**CN:** 定义函数 `TokenEmbeddingPoolerHead.forward_chunk`，其签名为 `forward_chunk(self, pooled_data: TokenPoolingMethodOutputItem, pooling_param: PoolingParams) -> TokenPoolerHeadOutputItem`。它主要围绕 `pooled_data`, `pooling_param` 展开；实现模块控制流中的一个步骤。函数体包含分支判断。关键调用包括 `pooled_data.to`, `self.projector`, `self.activation`。

### Class `TokenClassifierPoolerHead` overview (lines 91-140)
```python
class TokenClassifierPoolerHead(TokenPoolerHead):
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
        return {"token_classify"}

    def forward_chunk(
        self,
        pooled_data: TokenPoolingMethodOutputItem,
        pooling_param: PoolingParams,
    ) -> TokenPoolerHeadOutputItem:
```
**EN:** Defines class `TokenClassifierPoolerHead` with base classes `TokenPoolerHead` and decorators none. It acts as a head that projects hidden states into task outputs and exposes 3 direct methods, with notable entries `__init__`, `get_supported_tasks`, `forward_chunk`.
**CN:** 定义类 `TokenClassifierPoolerHead`，其基类为 `TokenPoolerHead`，装饰器为 无。它在整体实现中充当把隐藏状态映射到任务输出的头部模块，并直接暴露 3 个方法，较重要的包括 `__init__`, `get_supported_tasks`, `forward_chunk`。

### Method `TokenClassifierPoolerHead.__init__` (lines 92-106)
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
**EN:** Defines function `TokenClassifierPoolerHead.__init__` with signature `__init__(self, classifier: ClassifierFn | None=None, logit_mean: float | None=None, logit_sigma: float | None=None, head_dtype: torch.dtype | str | None=None, activation: ActivationFn | None=None) -> None`. It mainly works with `classifier`, `logit_mean`, `logit_sigma`, `head_dtype`, `activation`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `TokenClassifierPoolerHead.__init__`，其签名为 `__init__(self, classifier: ClassifierFn | None=None, logit_mean: float | None=None, logit_sigma: float | None=None, head_dtype: torch.dtype | str | None=None, activation: ActivationFn | None=None) -> None`。它主要围绕 `classifier`, `logit_mean`, `logit_sigma`, `head_dtype`, `activation` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `TokenClassifierPoolerHead.get_supported_tasks` (lines 108-109)
```python
    def get_supported_tasks(self) -> Set[PoolingTask]:
        return {"token_classify"}
```
**EN:** Defines function `TokenClassifierPoolerHead.get_supported_tasks` with signature `get_supported_tasks(self) -> Set[PoolingTask]`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `TokenClassifierPoolerHead.get_supported_tasks`，其签名为 `get_supported_tasks(self) -> Set[PoolingTask]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `TokenClassifierPoolerHead.forward_chunk` (lines 111-140)
```python
    def forward_chunk(
        self,
        pooled_data: TokenPoolingMethodOutputItem,
        pooling_param: PoolingParams,
    ) -> TokenPoolerHeadOutputItem:
        # for unfinished chunked prefill
        if pooled_data is None:
            return None

        if self.head_dtype is not None:
            pooled_data = pooled_data.to(self.head_dtype)
        # hidden_states shape: [n_token, hidden_size]

        if self.classifier is not None:
            logits = self.classifier(pooled_data)
        else:
            logits = pooled_data
        # logits shape: [n_token, num_labels]

        # Affine score calibration: activation((logit - mean) / sigma)
        if self.logit_mean is not None:
            logits = logits - self.logit_mean
        if self.logit_sigma is not None:
            logits = logits / self.logit_sigma

        if self.activation is not None and pooling_param.use_activation:
            logits = self.activation(logits)

        # logits shape: [n_token, num_labels]
        return logits
```
**EN:** Defines function `TokenClassifierPoolerHead.forward_chunk` with signature `forward_chunk(self, pooled_data: TokenPoolingMethodOutputItem, pooling_param: PoolingParams) -> TokenPoolerHeadOutputItem`. It mainly works with `pooled_data`, `pooling_param`; implements one step of the module control flow. The body uses branching. Key calls include `pooled_data.to`, `self.classifier`, `self.activation`.
**CN:** 定义函数 `TokenClassifierPoolerHead.forward_chunk`，其签名为 `forward_chunk(self, pooled_data: TokenPoolingMethodOutputItem, pooling_param: PoolingParams) -> TokenPoolerHeadOutputItem`。它主要围绕 `pooled_data`, `pooling_param` 展开；实现模块控制流中的一个步骤。函数体包含分支判断。关键调用包括 `pooled_data.to`, `self.classifier`, `self.activation`。

## Key Concepts / 关键概念
- **EN:** The file is part of the pooling stack that converts token-level hidden states into sequence-level or token-wise outputs.
  **CN:** 该文件属于池化栈，用于把 token 级隐藏状态转换为序列级或逐 token 输出。
- **EN:** Top-level classes include `TokenPoolerHead`, `TokenEmbeddingPoolerHead`, `TokenClassifierPoolerHead`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `TokenPoolerHead`, `TokenEmbeddingPoolerHead`, `TokenClassifierPoolerHead`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `abc`, `collections`, `typing`, `torch`
- **Internal / 内部**: `vllm.model_executor.layers.pooler`, `vllm.pooling_params`, `vllm.tasks`, `vllm.v1.pool.metadata`, `.methods`
