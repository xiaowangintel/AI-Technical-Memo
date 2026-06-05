# activations.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/pooler/activations.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `PoolerActivation`, `PoolerIdentity`, `PoolerNormalize` for pooling heads and methods. / 实现 `PoolerActivation`, `PoolerIdentity`, `PoolerNormalize`，用于池化头与池化方法。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 3-14)
```python
from abc import ABC, abstractmethod
from collections.abc import Callable
from typing import TypeVar

import torch
import torch.nn as nn
import torch.nn.functional as F
from transformers import PretrainedConfig

from vllm.config import ModelConfig
from vllm.logger import init_logger
from vllm.utils.import_utils import resolve_obj_by_qualname
```
**EN:** This opening block pulls in external dependencies such as `abc`, `collections`, `typing`, `torch`, `transformers` and internal modules such as `vllm.config`, `vllm.logger`, `vllm.utils.import_utils`. That import mix shows the file is part of the pooling heads and methods stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `abc`, `collections`, `typing`, `torch`, `transformers`）以及内部模块（如 `vllm.config`, `vllm.logger`, `vllm.utils.import_utils`）。这些导入关系表明该文件属于池化头与池化方法栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 16-74)
```python
logger = init_logger(__name__)


def get_act_fn(
    config: PretrainedConfig,
    static_num_labels: bool = True,
) -> "PoolerActivation":
    # get classification act_fn
    # Implement alignment with transformers ForSequenceClassificationLoss
    # https://github.com/huggingface/transformers/blob/57bb6db6ee4cfaccc45b8d474dfad5a17811ca60/src/transformers/loss/loss_utils.py#L92
    num_labels: int | None = None
    if static_num_labels:
        num_labels = getattr(config, "num_labels", 0)

    problem_type = getattr(config, "problem_type", "")
    if problem_type == "regression":
        return PoolerIdentity()
    if problem_type == "single_label_classification":
        return PoolerClassify(num_labels=num_labels)
    if problem_type == "multi_label_classification":
        return PoolerMultiLabelClassify()

    # get cross_encoder act_fn
    function_name: str | None = None
    if (
        hasattr(config, "sentence_transformers")
        and "activation_fn" in config.sentence_transformers
    ):
        function_name = config.sentence_transformers["activation_fn"]
    elif (
        hasattr(config, "sbert_ce_default_activation_function")
        and config.sbert_ce_default_activation_function is not None
    ):
        function_name = config.sbert_ce_default_activation_function

    if function_name is not None:
        assert function_name.startswith("torch.nn.modules."), (
            "Loading of activation functions is restricted to "
            "torch.nn.modules for security reasons"
        )
        fn = resolve_obj_by_qualname(function_name)()
        return PoolerActivation.wraps(fn)

    return PoolerClassify(num_labels=num_labels)


def resolve_classifier_act_fn(
    model_config: ModelConfig,
    static_num_labels: bool = True,
    act_fn: "PoolerActivation | None" = None,
):
    if act_fn is None:
        return get_act_fn(model_config.hf_config, static_num_labels)

    assert callable(act_fn)
    return act_fn


_T = TypeVar("_T", torch.Tensor, list[torch.Tensor])
```
**EN:** This block defines module-level metadata or constants such as `logger`, `_T`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the pooling heads and methods pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`, `_T`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在池化头与池化方法流程中复用。

### Function `get_act_fn` (lines 19-59)
```python
def get_act_fn(
    config: PretrainedConfig,
    static_num_labels: bool = True,
) -> "PoolerActivation":
    # get classification act_fn
    # Implement alignment with transformers ForSequenceClassificationLoss
    # https://github.com/huggingface/transformers/blob/57bb6db6ee4cfaccc45b8d474dfad5a17811ca60/src/transformers/loss/loss_utils.py#L92
    num_labels: int | None = None
    if static_num_labels:
        num_labels = getattr(config, "num_labels", 0)

    problem_type = getattr(config, "problem_type", "")
    if problem_type == "regression":
        return PoolerIdentity()
    if problem_type == "single_label_classification":
        return PoolerClassify(num_labels=num_labels)
    if problem_type == "multi_label_classification":
        return PoolerMultiLabelClassify()

    # get cross_encoder act_fn
    function_name: str | None = None
    if (
        hasattr(config, "sentence_transformers")
        and "activation_fn" in config.sentence_transformers
    ):
        function_name = config.sentence_transformers["activation_fn"]
    elif (
        hasattr(config, "sbert_ce_default_activation_function")
        and config.sbert_ce_default_activation_function is not None
    ):
        function_name = config.sbert_ce_default_activation_function

    if function_name is not None:
        assert function_name.startswith("torch.nn.modules."), (
            "Loading of activation functions is restricted to "
            "torch.nn.modules for security reasons"
        )
        fn = resolve_obj_by_qualname(function_name)()
        return PoolerActivation.wraps(fn)

    return PoolerClassify(num_labels=num_labels)
```
**EN:** Defines function `get_act_fn` with signature `get_act_fn(config: PretrainedConfig, static_num_labels: bool=True) -> 'PoolerActivation'`. It mainly works with `config`, `static_num_labels`; returns a derived property or capability check. The body uses branching, validation/error handling. Key calls include `getattr`, `PoolerClassify`, `PoolerIdentity`, `PoolerMultiLabelClassify`, `hasattr`, `function_name.startswith`.
**CN:** 定义函数 `get_act_fn`，其签名为 `get_act_fn(config: PretrainedConfig, static_num_labels: bool=True) -> 'PoolerActivation'`。它主要围绕 `config`, `static_num_labels` 展开；返回派生属性或能力判断结果。函数体包含分支判断、校验或报错逻辑。关键调用包括 `getattr`, `PoolerClassify`, `PoolerIdentity`, `PoolerMultiLabelClassify`, `hasattr`, `function_name.startswith`。

### Function `resolve_classifier_act_fn` (lines 62-71)
```python
def resolve_classifier_act_fn(
    model_config: ModelConfig,
    static_num_labels: bool = True,
    act_fn: "PoolerActivation | None" = None,
):
    if act_fn is None:
        return get_act_fn(model_config.hf_config, static_num_labels)

    assert callable(act_fn)
    return act_fn
```
**EN:** Defines function `resolve_classifier_act_fn` with signature `resolve_classifier_act_fn(model_config: ModelConfig, static_num_labels: bool=True, act_fn: 'PoolerActivation | None'=None)`. It mainly works with `model_config`, `static_num_labels`, `act_fn`; connects the implementation to a registry or backend lookup path. The body uses branching, validation/error handling. Key calls include `callable`, `get_act_fn`.
**CN:** 定义函数 `resolve_classifier_act_fn`，其签名为 `resolve_classifier_act_fn(model_config: ModelConfig, static_num_labels: bool=True, act_fn: 'PoolerActivation | None'=None)`。它主要围绕 `model_config`, `static_num_labels`, `act_fn` 展开；把实现接入注册表或后端查找路径。函数体包含分支判断、校验或报错逻辑。关键调用包括 `callable`, `get_act_fn`。

### Class `PoolerActivation` overview (lines 77-98)
```python
class PoolerActivation(nn.Module, ABC):
    @staticmethod
    def wraps(module: nn.Module):
        if isinstance(module, nn.Identity):
            return PoolerIdentity()
        if isinstance(module, (nn.Sigmoid, nn.Softmax)):
            return PoolerClassify()

        return LambdaPoolerActivation(module)

    @abstractmethod
    def forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor:
        raise NotImplementedError

    def forward(self, pooled_data: _T) -> _T:
        # shape:
        # classify -> (batch_size, num_classes)
        # embed -> (batch_size, embedding_size) or list(embedding_size)
        if isinstance(pooled_data, list):
            return [self.forward_chunk(data) for data in pooled_data]

        return self.forward_chunk(pooled_data)
```
**EN:** Defines class `PoolerActivation` with base classes `nn.Module`, `ABC` and decorators none. It acts as a reusable module building block and exposes 3 direct methods, with notable entries `wraps`, `forward_chunk`, `forward`.
**CN:** 定义类 `PoolerActivation`，其基类为 `nn.Module`, `ABC`，装饰器为 无。它在整体实现中充当可复用的模块构件，并直接暴露 3 个方法，较重要的包括 `wraps`, `forward_chunk`, `forward`。

### Method `PoolerActivation.wraps` (lines 79-85)
```python
    def wraps(module: nn.Module):
        if isinstance(module, nn.Identity):
            return PoolerIdentity()
        if isinstance(module, (nn.Sigmoid, nn.Softmax)):
            return PoolerClassify()

        return LambdaPoolerActivation(module)
```
**EN:** Defines function `PoolerActivation.wraps` with signature `wraps(module: nn.Module)`. It mainly works with `module`; implements one step of the module control flow. The body uses branching. Key calls include `isinstance`, `LambdaPoolerActivation`, `PoolerIdentity`, `PoolerClassify`.
**CN:** 定义函数 `PoolerActivation.wraps`，其签名为 `wraps(module: nn.Module)`。它主要围绕 `module` 展开；实现模块控制流中的一个步骤。函数体包含分支判断。关键调用包括 `isinstance`, `LambdaPoolerActivation`, `PoolerIdentity`, `PoolerClassify`。

### Method `PoolerActivation.forward_chunk` (lines 88-89)
```python
    def forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor:
        raise NotImplementedError
```
**EN:** Defines function `PoolerActivation.forward_chunk` with signature `forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor`. It mainly works with `pooled_data`; implements one step of the module control flow. The body uses validation/error handling. Key calls include no notable helper calls.
**CN:** 定义函数 `PoolerActivation.forward_chunk`，其签名为 `forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor`。它主要围绕 `pooled_data` 展开；实现模块控制流中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 无显著辅助调用。

### Method `PoolerActivation.forward` (lines 91-98)
```python
    def forward(self, pooled_data: _T) -> _T:
        # shape:
        # classify -> (batch_size, num_classes)
        # embed -> (batch_size, embedding_size) or list(embedding_size)
        if isinstance(pooled_data, list):
            return [self.forward_chunk(data) for data in pooled_data]

        return self.forward_chunk(pooled_data)
```
**EN:** Defines function `PoolerActivation.forward` with signature `forward(self, pooled_data: _T) -> _T`. It mainly works with `pooled_data`; runs the main forward-path computation. The body uses branching, comprehensions. Key calls include `isinstance`, `self.forward_chunk`.
**CN:** 定义函数 `PoolerActivation.forward`，其签名为 `forward(self, pooled_data: _T) -> _T`。它主要围绕 `pooled_data` 展开；执行主要的前向计算路径。函数体包含分支判断、推导式。关键调用包括 `isinstance`, `self.forward_chunk`。

### Class `PoolerIdentity` overview (lines 101-103)
```python
class PoolerIdentity(PoolerActivation):
    def forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor:
        return pooled_data
```
**EN:** Defines class `PoolerIdentity` with base classes `PoolerActivation` and decorators none. It acts as a reusable module building block and exposes 1 direct methods, with notable entries `forward_chunk`.
**CN:** 定义类 `PoolerIdentity`，其基类为 `PoolerActivation`，装饰器为 无。它在整体实现中充当可复用的模块构件，并直接暴露 1 个方法，较重要的包括 `forward_chunk`。

### Method `PoolerIdentity.forward_chunk` (lines 102-103)
```python
    def forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor:
        return pooled_data
```
**EN:** Defines function `PoolerIdentity.forward_chunk` with signature `forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor`. It mainly works with `pooled_data`; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `PoolerIdentity.forward_chunk`，其签名为 `forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor`。它主要围绕 `pooled_data` 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Class `PoolerNormalize` overview (lines 106-108)
```python
class PoolerNormalize(PoolerActivation):
    def forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor:
        return F.normalize(pooled_data, p=2, dim=-1)
```
**EN:** Defines class `PoolerNormalize` with base classes `PoolerActivation` and decorators none. It acts as a reusable module building block and exposes 1 direct methods, with notable entries `forward_chunk`.
**CN:** 定义类 `PoolerNormalize`，其基类为 `PoolerActivation`，装饰器为 无。它在整体实现中充当可复用的模块构件，并直接暴露 1 个方法，较重要的包括 `forward_chunk`。

### Method `PoolerNormalize.forward_chunk` (lines 107-108)
```python
    def forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor:
        return F.normalize(pooled_data, p=2, dim=-1)
```
**EN:** Defines function `PoolerNormalize.forward_chunk` with signature `forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor`. It mainly works with `pooled_data`; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include `F.normalize`.
**CN:** 定义函数 `PoolerNormalize.forward_chunk`，其签名为 `forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor`。它主要围绕 `pooled_data` 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `F.normalize`。

### Class `PoolerMultiLabelClassify` overview (lines 111-113)
```python
class PoolerMultiLabelClassify(PoolerActivation):
    def forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor:
        return F.sigmoid(pooled_data)
```
**EN:** Defines class `PoolerMultiLabelClassify` with base classes `PoolerActivation` and decorators none. It acts as a reusable module building block and exposes 1 direct methods, with notable entries `forward_chunk`.
**CN:** 定义类 `PoolerMultiLabelClassify`，其基类为 `PoolerActivation`，装饰器为 无。它在整体实现中充当可复用的模块构件，并直接暴露 1 个方法，较重要的包括 `forward_chunk`。

### Method `PoolerMultiLabelClassify.forward_chunk` (lines 112-113)
```python
    def forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor:
        return F.sigmoid(pooled_data)
```
**EN:** Defines function `PoolerMultiLabelClassify.forward_chunk` with signature `forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor`. It mainly works with `pooled_data`; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include `F.sigmoid`.
**CN:** 定义函数 `PoolerMultiLabelClassify.forward_chunk`，其签名为 `forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor`。它主要围绕 `pooled_data` 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `F.sigmoid`。

### Class `PoolerClassify` overview (lines 116-137)
```python
class PoolerClassify(PoolerActivation):
    def __init__(self, *, num_labels: int | None = None) -> None:
        super().__init__()

        if num_labels == 0:
            logger.warning(
                "num_labels should be > 0 for classification "
                "models, falling back to sigmoid. "
                "Please check if the configuration is correct."
            )

        self.num_labels = num_labels

    def forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor:
        num_labels = self.num_labels
        if num_labels is None:
            num_labels = pooled_data.shape[-1]

        if num_labels < 2:
            return F.sigmoid(pooled_data)

        return F.softmax(pooled_data, dim=-1)
```
**EN:** Defines class `PoolerClassify` with base classes `PoolerActivation` and decorators none. It acts as a reusable module building block and exposes 2 direct methods, with notable entries `__init__`, `forward_chunk`.
**CN:** 定义类 `PoolerClassify`，其基类为 `PoolerActivation`，装饰器为 无。它在整体实现中充当可复用的模块构件，并直接暴露 2 个方法，较重要的包括 `__init__`, `forward_chunk`。

### Method `PoolerClassify.__init__` (lines 117-127)
```python
    def __init__(self, *, num_labels: int | None = None) -> None:
        super().__init__()

        if num_labels == 0:
            logger.warning(
                "num_labels should be > 0 for classification "
                "models, falling back to sigmoid. "
                "Please check if the configuration is correct."
            )

        self.num_labels = num_labels
```
**EN:** Defines function `PoolerClassify.__init__` with signature `__init__(self, *, num_labels: int | None=None) -> None`. It mainly works with `num_labels`; initializes the object state and cached resources. The body uses branching. Key calls include `super.__init__`, `logger.warning`, `super`.
**CN:** 定义函数 `PoolerClassify.__init__`，其签名为 `__init__(self, *, num_labels: int | None=None) -> None`。它主要围绕 `num_labels` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断。关键调用包括 `super.__init__`, `logger.warning`, `super`。

### Method `PoolerClassify.forward_chunk` (lines 129-137)
```python
    def forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor:
        num_labels = self.num_labels
        if num_labels is None:
            num_labels = pooled_data.shape[-1]

        if num_labels < 2:
            return F.sigmoid(pooled_data)

        return F.softmax(pooled_data, dim=-1)
```
**EN:** Defines function `PoolerClassify.forward_chunk` with signature `forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor`. It mainly works with `pooled_data`; implements one step of the module control flow. The body uses branching. Key calls include `F.softmax`, `F.sigmoid`.
**CN:** 定义函数 `PoolerClassify.forward_chunk`，其签名为 `forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor`。它主要围绕 `pooled_data` 展开；实现模块控制流中的一个步骤。函数体包含分支判断。关键调用包括 `F.softmax`, `F.sigmoid`。

### Class `LambdaPoolerActivation` overview (lines 140-147)
```python
class LambdaPoolerActivation(PoolerActivation):
    def __init__(self, fn: Callable[[torch.Tensor], torch.Tensor]):
        super().__init__()

        self.fn = fn

    def forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor:
        return self.fn(pooled_data)
```
**EN:** Defines class `LambdaPoolerActivation` with base classes `PoolerActivation` and decorators none. It acts as a reusable module building block and exposes 2 direct methods, with notable entries `__init__`, `forward_chunk`.
**CN:** 定义类 `LambdaPoolerActivation`，其基类为 `PoolerActivation`，装饰器为 无。它在整体实现中充当可复用的模块构件，并直接暴露 2 个方法，较重要的包括 `__init__`, `forward_chunk`。

### Method `LambdaPoolerActivation.__init__` (lines 141-144)
```python
    def __init__(self, fn: Callable[[torch.Tensor], torch.Tensor]):
        super().__init__()

        self.fn = fn
```
**EN:** Defines function `LambdaPoolerActivation.__init__` with signature `__init__(self, fn: Callable[[torch.Tensor], torch.Tensor])`. It mainly works with `fn`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `LambdaPoolerActivation.__init__`，其签名为 `__init__(self, fn: Callable[[torch.Tensor], torch.Tensor])`。它主要围绕 `fn` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `LambdaPoolerActivation.forward_chunk` (lines 146-147)
```python
    def forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor:
        return self.fn(pooled_data)
```
**EN:** Defines function `LambdaPoolerActivation.forward_chunk` with signature `forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor`. It mainly works with `pooled_data`; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include `self.fn`.
**CN:** 定义函数 `LambdaPoolerActivation.forward_chunk`，其签名为 `forward_chunk(self, pooled_data: torch.Tensor) -> torch.Tensor`。它主要围绕 `pooled_data` 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.fn`。

## Key Concepts / 关键概念
- **EN:** The file is part of the pooling stack that converts token-level hidden states into sequence-level or token-wise outputs.
  **CN:** 该文件属于池化栈，用于把 token 级隐藏状态转换为序列级或逐 token 输出。
- **EN:** Top-level classes include `PoolerActivation`, `PoolerIdentity`, `PoolerNormalize`, `PoolerMultiLabelClassify`, `PoolerClassify`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `PoolerActivation`, `PoolerIdentity`, `PoolerNormalize`, `PoolerMultiLabelClassify`, `PoolerClassify`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `get_act_fn`, `resolve_classifier_act_fn` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `get_act_fn`, `resolve_classifier_act_fn` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `abc`, `collections`, `typing`, `torch`, `transformers`
- **Internal / 内部**: `vllm.config`, `vllm.logger`, `vllm.utils.import_utils`
