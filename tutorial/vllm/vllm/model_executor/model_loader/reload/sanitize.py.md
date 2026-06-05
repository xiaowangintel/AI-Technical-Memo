# sanitize.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/model_loader/reload/sanitize.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements sanitize support for the `reload` portion of vLLM. / 为 vLLM 的 `reload` 子目录实现与 sanitize 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-5)
```python
from types import MethodType

import torch
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 7-10)
```python
__all__ = ["sanitize_layer_refs", "restore_layer_refs"]

layer_ref_sentinel = object()
```
**EN:** This constant/configuration block defines `__all__`, `layer_ref_sentinel`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `__all__`, `layer_ref_sentinel`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Function `sanitize_layer_refs` (lines 13-31)
```python
def sanitize_layer_refs(tensor: torch.Tensor, layer: torch.nn.Module) -> torch.Tensor:
    """
    Removes references to layer held by tensor attributes. Specifically, removes the
    `__self__` attribute of weight loader methods attached to the tensor.

    Used by `capture_layer_to_meta` to avoid circular references to layers in
    `LAYERWISE_INFO`, leading to modules never being cleaned up. Without sanitation,
    tensors will reference layers, and the WeakKeyDictionary will never evict entries,
    even when the model is deleted.

    :param tensor: tensor to be sanitized
    :param layer: layer whose references should be removed
    :return: sanitized tensor
    """
    for key, value in tensor.__dict__.items():
        if isinstance(value, MethodType) and value.__self__ is layer:
            tensor.__dict__[key] = value.__func__.__get__(layer_ref_sentinel)

    return tensor
```
**EN:** Function `sanitize_layer_refs` provides a reusable helper around the module's main workflow. The docstring highlights: Removes references to layer held by tensor attributes. Key calls such as `tensor.__dict__.items`, `isinstance`, `value.__func__.__get__` show the concrete execution path.
**CN:** Function `sanitize_layer_refs` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Removes references to layer held by tensor attributes. 像 `tensor.__dict__.items`, `isinstance`, `value.__func__.__get__` 这样的关键调用展示了该代码块的具体执行路径。

### Function `restore_layer_refs` (lines 34-50)
```python
def restore_layer_refs(tensor: torch.Tensor, layer: torch.nn.Module) -> torch.Tensor:
    """
    Restores references to layer held by tensor attributes.

    Used by `restore_layer_on_meta` to add back layer references, allowing for proper
    weight loading.

    :param tensor: tensor to be sanitized
    :param layer: layer whose references should be removed
    :return: sanitized tensor

    """
    for key, value in tensor.__dict__.items():
        if isinstance(value, MethodType) and value.__self__ is layer_ref_sentinel:
            tensor.__dict__[key] = value.__func__.__get__(layer)

    return tensor
```
**EN:** Function `restore_layer_refs` provides a reusable helper around the module's main workflow. The docstring highlights: Restores references to layer held by tensor attributes. Key calls such as `tensor.__dict__.items`, `isinstance`, `value.__func__.__get__` show the concrete execution path.
**CN:** Function `restore_layer_refs` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Restores references to layer held by tensor attributes. 像 `tensor.__dict__.items`, `isinstance`, `value.__func__.__get__` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from types import MethodType`
- **Third-party / 第三方**: `import torch`
