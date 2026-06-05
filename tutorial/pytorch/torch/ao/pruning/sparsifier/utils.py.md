# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/pruning/sparsifier/utils.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements pruning and sparsity utilities that remove or mask parameters while preserving model structure. This specific file centers on `utils.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior.
- **Purpose (CN) / 用途（中文）**: 实现剪枝与稀疏化工具，在保留模型结构的同时移除或屏蔽参数。 该文件具体围绕 `utils.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。
## Line-by-Line Analysis / 逐行分析

### Lines 1-16 / 第 1-16 行
```python
# mypy: allow-untyped-defs
from itertools import chain
from typing import Any

from torch import nn
from torch.nn.utils.parametrize import is_parametrized, type_before_parametrizations


__all__ = [
    "module_contains_param",
    "swap_module",
    "module_to_fqn",
    "fqn_to_module",
    "get_arg_info_from_tensor_fqn",
    "FakeSparsity",
]
```
- **EN**: The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export.
- **CN**: `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。

### Lines 19-36 / 第 19-36 行
```python
def module_contains_param(module: nn.Module, parametrization: type[nn.Module]) -> bool:
    if is_parametrized(module):
        # see if any of the module tensors have a parametriztion attached that matches the one passed in
        return any(
            any(isinstance(param, parametrization) for param in param_list)
            for key, param_list in module.parametrizations.items()  # type: ignore[union-attr,operator]
        )
    return False


def swap_module(
    mod: nn.Module, mapping: dict[type[nn.Module], type[nn.Module]]
) -> nn.Module:
    r"""Swaps the module using from_dense according to the mapping passed in.
    Args:
        mod: input module
        mapping: a dictionary that maps from nn module to sparse nn module
    Return:
```
- **EN**: Key callable entry points in this range include `module_contains_param`, `swap_module`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `module_contains_param`, `swap_module`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 37-51 / 第 37-51 行
```python
        The corresponding sparse module of `mod` according to mapping, created using from_dense
    """
    if type_before_parametrizations(mod) in mapping:
        sparse_mod = mapping[type_before_parametrizations(mod)]

        # TODO Fix this typing, as Type[Module] has no attribute "from_dense"
        new_mod = sparse_mod.from_dense(mod)  # type: ignore[attr-defined]

        # Preserve module's pre forward hooks. They'll be called on quantized input
        for pre_hook_fn in mod._forward_pre_hooks.values():
            new_mod.register_forward_pre_hook(pre_hook_fn)
        # Preserve module's post forward hooks except _observer_forward_hook
        # After convert they'll work with quantized output
        for hook_fn in mod._forward_hooks.values():
            new_mod.register_forward_hook(hook_fn)
```
- **EN**: Key callable entry points in this range include `swap_module`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `swap_module`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 53-67 / 第 53-67 行
```python
        # respect device affinity when swapping modules
        # pyrefly: ignore [bad-argument-type]
        devices = {p.device for p in chain(mod.parameters(), mod.buffers())}
        if len(devices) > 1:
            raise AssertionError(
                f"swap_module only works with cpu or single-device CUDA modules, but got devices {devices}"
            )
        device = next(iter(devices)) if len(devices) > 0 else None
        if device:
            new_mod.to(device)

        return new_mod

    else:
        return mod
```
- **EN**: Key callable entry points in this range include `swap_module`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `swap_module`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 70-80 / 第 70-80 行
```python
def module_to_fqn(model: nn.Module, module: nn.Module, prefix: str = "") -> str | None:
    """
    Returns the fqn for a module or None if module not a descendent of model.
    """
    if module is model:
        return ""
    for name, child in model.named_children():
        fqn = module_to_fqn(child, module, ".")
        if isinstance(fqn, str):
            return prefix + name + fqn
    return None
```
- **EN**: Key callable entry points in this range include `module_to_fqn`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `module_to_fqn`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 83-100 / 第 83-100 行
```python
def fqn_to_module(model: nn.Module | None, path: str) -> nn.Module | None:
    """
    Given an fqn, returns the corresponding module or tensor or None if the fqn given by `path`
    doesn't correspond to anything. Similar to model.get_submodule(path) but works for tensors.
    """
    if path != "":
        for name in path.split("."):
            model = getattr(model, name, None)
    return model


def get_arg_info_from_tensor_fqn(model: nn.Module, tensor_fqn: str) -> dict[str, Any]:
    """
    Uses tensor_fqn to obtain a dict containing module_fqn, module and tensor_name
    """
    # string manip to split tensor_fqn into module_fqn and tensor_name
    # if tensor_fqn is 'weight' then module_fqn and tensor_name are '' and 'weight'
    # if tensor_fqn is 'linear.weight' then module_fqn and tensor_name are 'linear' and 'weight'
```
- **EN**: Key callable entry points in this range include `fqn_to_module`, `get_arg_info_from_tensor_fqn`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `fqn_to_module`, `get_arg_info_from_tensor_fqn`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 101-117 / 第 101-117 行
```python
    tensor_name = tensor_fqn.rsplit(".", maxsplit=1)[-1]
    module_fqn = tensor_fqn[: -len(tensor_name) - ("." in tensor_fqn)]

    module = fqn_to_module(model, module_fqn)

    return {
        "module_fqn": module_fqn,
        "module": module,
        "tensor_name": tensor_name,
        "tensor_fqn": tensor_fqn,
    }


# Parametrizations
class FakeSparsity(nn.Module):
    r"""Parametrization for the weights. Should be attached to the 'weight' or
    any other parameter that requires a mask applied to it.
```
- **EN**: It introduces or extends class-level abstractions such as `FakeSparsity`, which organize state and behavior for this subsystem. Key callable entry points in this range include `get_arg_info_from_tensor_fqn`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `FakeSparsity` 等类级抽象，用于组织该子系统的状态与行为。 这一段的重要可调用入口包括 `get_arg_info_from_tensor_fqn`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 119-135 / 第 119-135 行
```python
    Note::

        Once the mask is passed, the variable should not change the id. The
        contents of the mask can change, but the mask reference itself should
        not.
    """

    def __init__(self, mask):
        super().__init__()
        self.register_buffer("mask", mask)

    def forward(self, x):
        if self.mask.shape != x.shape:
            raise AssertionError(
                f"mask shape ({self.mask.shape}) must match x shape ({x.shape})"
            )
        return self.mask * x
```
- **EN**: It introduces or extends class-level abstractions such as `FakeSparsity`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `FakeSparsity` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 137-141 / 第 137-141 行
```python
    def state_dict(self, *args, **kwargs):
        # We don't want to let the parametrizations to save the mask.
        # That way we make sure that the linear module doesn't store the masks
        # alongside their parametrizations.
        return {}
```
- **EN**: It introduces or extends class-level abstractions such as `FakeSparsity`, which organize state and behavior for this subsystem. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `FakeSparsity` 等类级抽象，用于组织该子系统的状态与行为。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements pruning and sparsity utilities that remove or mask parameters while preserving model structure.
  - CN: 实现剪枝与稀疏化工具，在保留模型结构的同时移除或屏蔽参数。
- **Quantization pipeline**
  - EN: Tracks calibration statistics, quantization parameters, or graph/module rewrites for lower-precision execution.
  - CN: 跟踪校准统计信息、量化参数，或面向低精度执行的图/模块改写。
- **Observer statistics**
  - EN: Collects runtime ranges or histograms so later code can derive quantization settings.
  - CN: 收集运行时范围或直方图，以便后续逻辑推导量化设置。
- **Fake quantization**
  - EN: Simulates quantized numerics during training while keeping tensors in differentiable forms.
  - CN: 在训练阶段模拟量化数值行为，同时保持张量处于可求导形式。
- **Graph rewriting**
  - EN: Rewrites module or FX graph structure so later execution follows a transformed pipeline.
  - CN: 改写模块或 FX 图结构，使后续执行遵循转换后的流水线。
- **Pruning and sparsity**
  - EN: Controls parameter masking, sparse layouts, or pruning schedules that reduce effective model size.
  - CN: 控制参数掩码、稀疏布局或剪枝计划，以降低模型的有效规模。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch:nn`, `torch.nn.utils.parametrize:is_parametrized`, `torch.nn.utils.parametrize:type_before_parametrizations`
- **Python standard library / Python 标准库**: `itertools:chain`, `typing:Any`
- **Explicit exports / 显式导出**: `module_contains_param`, `swap_module`, `module_to_fqn`, `fqn_to_module`, `get_arg_info_from_tensor_fqn`, `FakeSparsity`
- **Primary symbols / 核心符号**: `FakeSparsity`, `module_contains_param`, `swap_module`, `module_to_fqn`, `fqn_to_module`, `get_arg_info_from_tensor_fqn`
