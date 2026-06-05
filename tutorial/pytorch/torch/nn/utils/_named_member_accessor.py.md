# _named_member_accessor.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/utils/_named_member_accessor.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects helper routines that reshape, inspect, or adapt nn.Module behavior.
- **Purpose (CN)**: 汇集用于重塑、检查或适配 nn.Module 行为的辅助函数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-27
```python
# This source code is licensed under the BSD-style license found in the
# LICENSE file in the root directory of this source tree.

from collections.abc import Iterable

import torch


_MISSING: torch.Tensor = object()  # type: ignore[assignment]


def set_tensor(module: "torch.nn.Module", name: str, tensor: torch.Tensor) -> None:
    if not isinstance(module, torch.nn.Module):
        raise TypeError(f"{module} is not an instance of torch.nn.Module")
    if not isinstance(tensor, torch.Tensor) and tensor is not None:
        raise TypeError(f"{tensor} is not an instance of torch.Tensor")
    if "." in name:
        raise KeyError('tensor name can\'t contain "."')
    if name == "":
        raise KeyError('tensor name can\'t be empty string ""')
    if name in module._parameters:
        module._parameters[name] = tensor  # type: ignore[assignment]
    elif name in module._buffers:
        module._buffers[name] = tensor
    else:
        setattr(module, name, tensor)
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 28-47
```python

def swap_tensor(
    module: "torch.nn.Module",
    name: str,
    tensor: torch.Tensor,
    allow_missing: bool = False,
) -> torch.Tensor:
    if not isinstance(module, torch.nn.Module):
        raise TypeError(f"{module} is not an instance of torch.nn.Module")
    if (
        tensor is not _MISSING
        and not isinstance(tensor, torch.Tensor)
        and tensor is not None
    ):
        raise TypeError(f"{tensor} is not an instance of torch.Tensor")
    if "." in name:
        raise KeyError('tensor name can\'t contain "."')
    if name == "":
        raise KeyError('tensor name can\'t be empty string ""')
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 48-65
```python
    orig_tensor: torch.Tensor
    if name in module._parameters:
        orig_tensor = module._parameters[name]  # type: ignore[assignment]
        if tensor is not _MISSING:
            module._parameters[name] = tensor  # type: ignore[assignment]
        else:
            del module._parameters[name]
    elif name in module._buffers:
        orig_tensor = module._buffers[name]  # type: ignore[assignment]
        if tensor is not _MISSING:
            module._buffers[name] = tensor
        else:
            del module._buffers[name]
    else:
        if hasattr(module, name):
            orig_tensor = getattr(module, name)
        else:
            if not allow_missing:
```
- **EN**: This block continues `swap_tensor` and works to organize reusable module behavior and state. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `swap_tensor`，用于组织可复用的模块行为与状态。 同时它还会根据运行时条件分支处理。

### Lines 66-83
```python
                raise AttributeError(f"{module._get_name()} has no attribute `{name}`")
            orig_tensor = _MISSING
        if (
            orig_tensor is not _MISSING
            and not isinstance(orig_tensor, torch.Tensor)
            and orig_tensor is not None
        ):
            raise TypeError(
                f"attribute `{name}`: {orig_tensor} is not an instance of torch.Tensor"
            )
        if tensor is not _MISSING:
            setattr(module, name, tensor)
        elif hasattr(module, name):
            delattr(module, name)
    # pyrefly: ignore [bad-return]
    return orig_tensor
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 84-110
```python
def swap_submodule(
    module: "torch.nn.Module",
    name: str,
    submodule: "torch.nn.Module",
) -> "torch.nn.Module":
    if not isinstance(module, torch.nn.Module):
        raise TypeError(f"{module} is not an instance of torch.nn.Module")
    if not isinstance(submodule, torch.nn.Module):
        raise TypeError(f"{submodule} is not an instance of torch.nn.Module")
    if "." in name:
        raise KeyError('submodule name can\'t contain "."')
    if name == "":
        raise KeyError('submodule name can\'t be empty string ""')
    if name not in module._modules:
        raise KeyError(f"submodule {name} does not exist")

    orig_submodule = module._modules[name]
    if not isinstance(orig_submodule, torch.nn.Module):
        raise TypeError(f"{name} attribute is not an instance of torch.nn.Module")
    module._modules[name] = submodule
    return orig_submodule


class NamedMemberAccessor:
    """
    A class that provides a way to access the submodules and parameters/buffers of a module.
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 111-133
```python
    It provides caching mechanism to speed up submodule lookups.
    This is useful for functional programming to manipulate the module state.
    """

    def __init__(self, module: "torch.nn.Module") -> None:
        self.module = module
        self.memo: dict[str, torch.nn.Module] = {}

    # Nested attribute access

    def get_submodule(self, name: str) -> "torch.nn.Module":
        """
        Return the submodule specified by the given path.

        For example, to get the submodule mod.layer1.conv1,
        use accessor.get_submodule("layer1.conv1")

        Compare to mod.get_submodule("layer1.conv1"), this method will cache the
        intermediate submodule access to speed up future lookups.
        """
        if not name:
            return self.module
```
- **EN**: Declares `NamedMemberAccessor`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `NamedMemberAccessor`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

### Lines 134-158
```python
        if name in self.memo:
            return self.memo[name]
        else:
            prefix, dot, attr = name.rpartition(".")
            if dot:
                module = self.get_submodule(prefix)
            else:
                module = self.module
            try:
                submodule = getattr(module, attr)
            except AttributeError as ex:
                raise AttributeError(
                    f"{module._get_name()} has no attribute `{attr}`"
                ) from ex
            if not isinstance(submodule, torch.nn.Module):
                raise TypeError(
                    f"submodule `{name}`: {submodule} is not an instance of torch.nn.Module"
                )
            self.memo[name] = submodule
            return submodule

    def swap_submodule(self, path: str, value: "torch.nn.Module") -> "torch.nn.Module":
        """
        Swap the submodule specified by the given ``path`` to ``value``.
```
- **EN**: Declares `NamedMemberAccessor`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `NamedMemberAccessor`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

### Lines 159-176
```python
        For example, to swap the attribute mod.layer1.conv1 use
        ``accessor.swap_submodule("layer1.conv1", conv2)``.
        """
        prefix, _, attr = path.rpartition(".")
        return swap_submodule(self.get_submodule(prefix), attr, value)

    def get_tensor(self, name: str) -> torch.Tensor:
        """
        Get the tensor specified by the given path to value.

        For example, to get the attribute mod.layer1.conv1.weight,
        use accessor.get_tensor('layer1.conv1.weight')

        Compare to mod.get_parameter("layer1.conv1.weight"), this method will
        cache the intermediate submodule access to speed up future lookups.
        """
        prefix, _, attr = name.rpartition(".")
        submodule = self.get_submodule(prefix)
```
- **EN**: Declares `NamedMemberAccessor`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `NamedMemberAccessor`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

### Lines 177-200
```python
        try:
            tensor = getattr(submodule, attr)
        except AttributeError as ex:
            raise AttributeError(
                f"{submodule._get_name()} has no attribute `{name}`"
            ) from ex
        if not isinstance(tensor, torch.Tensor) and tensor is not None:
            raise TypeError(f"{tensor} is not an instance of torch.Tensor")
        return tensor  # type: ignore[return-value]

    def set_tensor(self, name: str, value: torch.Tensor) -> None:
        """
        Set the attribute specified by the given path to value.

        For example, to set the attribute mod.layer1.conv1.weight,
        use accessor.set_tensor("layer1.conv1.weight", value)
        """
        prefix, _, attr = name.rpartition(".")
        set_tensor(self.get_submodule(prefix), attr, value)

    def del_tensor(self, name: str) -> None:
        """
        Delete the attribute specified by the given path.
```
- **EN**: Declares `NamedMemberAccessor`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `NamedMemberAccessor`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

### Lines 201-226
```python
        For example, to delete the attribute mod.layer1.conv1.weight,
        use accessor.del_tensor("layer1.conv1.weight")
        """
        prefix, _, attr = name.rpartition(".")
        submodule = self.get_submodule(prefix)
        try:
            delattr(submodule, attr)
        except AttributeError as ex:
            raise AttributeError(
                f"{submodule._get_name()} has no attribute `{name}`"
            ) from ex

    def swap_tensor(
        self, name: str, value: torch.Tensor, allow_missing: bool = False
    ) -> torch.Tensor:
        """
        Swap the attribute specified by the given path to value.

        For example, to swap the attribute mod.layer1.conv1.weight,
        use accessor.swap_tensor("layer1.conv1.weight", value)
        """
        prefix, _, attr = name.rpartition(".")
        return swap_tensor(
            self.get_submodule(prefix), attr, value, allow_missing=allow_missing
        )
```
- **EN**: Declares `NamedMemberAccessor`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `NamedMemberAccessor`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

### Lines 227-244
```python
    # Batched operations

    def get_tensors(self, names: Iterable[str]) -> list[torch.Tensor]:
        """
        Get the tensors specified by the given paths.

        For example, to get the attributes mod.layer1.conv1.weight and
        mod.layer1.conv1.bias, use accessor.get_tensors(["layer1.conv1.weight",
        "layer1.conv1.bias"])
        """
        return [self.get_tensor(name) for name in names]

    def set_tensors(self, names: Iterable[str], values: Iterable[torch.Tensor]) -> None:
        """
        Set the attributes specified by the given paths to values.

        For example, to set the attributes mod.layer1.conv1.weight and
        mod.layer1.conv1.bias, use accessor.set_tensors(["layer1.conv1.weight",
```
- **EN**: Declares `NamedMemberAccessor`; this class packages state and methods that prepare neural-network operators or module behavior.
- **CN**: 声明 `NamedMemberAccessor`；该类封装了状态与方法，用于准备神经网络算子或模块行为。

### Lines 245-263
```python
        "layer1.conv1.bias"], [weight, bias])
        """
        if not isinstance(names, (list, tuple)):
            names = list(names)
        if not isinstance(values, (list, tuple)):
            values = list(values)
        if len(names) != len(values):
            raise AssertionError(
                f"names and values must have the same length, "
                f"got {len(names)} names and {len(values)} values"
            )

        for name, value in zip(names, values, strict=True):
            self.set_tensor(name, value)

    def set_tensors_dict(self, named_tensors: dict[str, torch.Tensor]) -> None:
        """
        Set the attributes specified by the given paths to values.
```
- **EN**: Declares `NamedMemberAccessor`; this class packages state and methods that validate invariants and surface meaningful failures.
- **CN**: 声明 `NamedMemberAccessor`；该类封装了状态与方法，用于校验不变量并给出有意义的失败信息。

### Lines 264-283
```python
        For example, to set the attributes mod.layer1.conv1.weight and
        mod.layer1.conv1.bias, use accessor.set_tensors_dict({
            "layer1.conv1.weight": weight,
            "layer1.conv1.bias": bias,
        })
        """
        for name, value in named_tensors.items():
            self.set_tensor(name, value)

    def del_tensors(self, names: Iterable[str]) -> None:
        """
        Delete the attributes specified by the given paths.

        For example, to delete the attributes mod.layer1.conv1.weight and
        mod.layer1.conv1.bias, use accessor.del_tensors(["layer1.conv1.weight",
        "layer1.conv1.bias"])
        """
        for name in names:
            self.del_tensor(name)
```
- **EN**: Declares `NamedMemberAccessor`; this class packages state and methods that prepare neural-network operators or module behavior.
- **CN**: 声明 `NamedMemberAccessor`；该类封装了状态与方法，用于准备神经网络算子或模块行为。

### Lines 284-306
```python
    def swap_tensors(
        self,
        names: Iterable[str],
        values: Iterable[torch.Tensor],
        allow_missing: bool = False,
    ) -> list[torch.Tensor]:
        """
        Swap the attributes specified by the given paths to values.

        For example, to swap the attributes mod.layer1.conv1.weight and
        mod.layer1.conv1.bias, use accessor.swap_tensors(["layer1.conv1.weight",
        "layer1.conv1.bias"], [weight, bias])
        """
        if not isinstance(names, (list, tuple)):
            names = list(names)
        if not isinstance(values, (list, tuple)):
            values = list(values)
        if len(names) != len(values):
            raise AssertionError(
                f"names and values must have the same length, "
                f"got {len(names)} names and {len(values)} values"
            )
```
- **EN**: Defines the `NamedMemberAccessor.swap_tensors` method; this block introduces logic that validate invariants and surface meaningful failures.
- **CN**: 定义`NamedMemberAccessor.swap_tensors` 方法；该代码块引入了用于校验不变量并给出有意义的失败信息的逻辑。

### Lines 307-324
```python
        return [
            self.swap_tensor(name, value, allow_missing=allow_missing)
            for name, value in zip(names, values, strict=True)
        ]

    def swap_tensors_dict(
        self, named_tensors: dict[str, torch.Tensor], allow_missing: bool = False
    ) -> tuple[dict[str, torch.Tensor], list[str]]:
        """
        Swap the attributes specified by the given paths to values.

        For example, to swap the attributes mod.layer1.conv1.weight and
        mod.layer1.conv1.bias, use accessor.swap_tensors_dict({
            "layer1.conv1.weight": weight,
            "layer1.conv1.bias": bias,
        })
        """
        orig_named_tensors = {}
```
- **EN**: Declares `NamedMemberAccessor`; this class packages state and methods that prepare neural-network operators or module behavior.
- **CN**: 声明 `NamedMemberAccessor`；该类封装了状态与方法，用于准备神经网络算子或模块行为。

### Lines 325-351
```python
        missing_keys = []
        try:
            for name, tensor in named_tensors.items():
                orig_tensor = self.swap_tensor(name, tensor, allow_missing=True)
                if orig_tensor is _MISSING:
                    missing_keys.append(name)
                orig_named_tensors[name] = orig_tensor
        except Exception:
            # Swap back if any exception occurs
            for name, orig_tensor in orig_named_tensors.items():
                self.swap_tensor(name, orig_tensor, allow_missing=True)
            raise
        if missing_keys and not allow_missing:
            # Swap back if any key is missing when allow_missing is False
            for name, orig_tensor in orig_named_tensors.items():
                self.swap_tensor(name, orig_tensor, allow_missing=True)
            raise RuntimeError(f"Missing key(s): {', '.join(map(repr, missing_keys))}.")
        return orig_named_tensors, missing_keys

    def check_keys(self, keys: Iterable[str]) -> tuple[list[str], list[str]]:
        """Check that the given keys are valid."""
        keys = set(keys)
        valid_keys = {name for name, _ in self.named_tensors(remove_duplicate=False)}
        missing_keys = valid_keys - keys
        unexpected_keys = keys - valid_keys
        return sorted(missing_keys), sorted(unexpected_keys)
```
- **EN**: Declares `NamedMemberAccessor`; this class packages state and methods that validate invariants and surface meaningful failures.
- **CN**: 声明 `NamedMemberAccessor`；该类封装了状态与方法，用于校验不变量并给出有意义的失败信息。

### Lines 352-375
```python
    # Shortcut methods

    def named_parameters(
        self,
        remove_duplicate: bool = True,
    ) -> Iterable[tuple[str, torch.Tensor]]:
        """Iterate over all the parameters in the module."""
        yield from self.module.named_parameters(remove_duplicate=remove_duplicate)

    def named_buffers(
        self,
        remove_duplicate: bool = True,
    ) -> Iterable[tuple[str, torch.Tensor]]:
        """Iterate over all the buffers in the module."""
        yield from self.module.named_buffers(remove_duplicate=remove_duplicate)

    def named_tensors(
        self,
        remove_duplicate: bool = True,
    ) -> Iterable[tuple[str, torch.Tensor]]:
        """Iterate over all the tensors in the module."""
        yield from self.module.named_parameters(remove_duplicate=remove_duplicate)
        yield from self.module.named_buffers(remove_duplicate=remove_duplicate)
```
- **EN**: Declares `NamedMemberAccessor`; this class packages state and methods that organize reusable module behavior and state.
- **CN**: 声明 `NamedMemberAccessor`；该类封装了状态与方法，用于组织可复用的模块行为与状态。

### Lines 376-381
```python
    def named_modules(
        self,
        remove_duplicate: bool = True,
    ) -> Iterable[tuple[str, "torch.nn.Module"]]:
        """Iterate over all the modules in the module."""
        yield from self.module.named_modules(remove_duplicate=remove_duplicate)
```
- **EN**: Defines the `NamedMemberAccessor.named_modules` method; this block introduces logic that organize reusable module behavior and state.
- **CN**: 定义`NamedMemberAccessor.named_modules` 方法；该代码块引入了用于组织可复用的模块行为与状态的逻辑。

## Key Concepts / 关键概念

- **Neural-network abstractions / 神经网络抽象**
  - EN: The file contributes to reusable neural-network layers, functionals, or support utilities.
  - CN: 该文件为可复用的神经网络层、函数式接口或辅助工具提供实现。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`
- **Standard library / 标准库**: `collections.abc`
- **Primary symbols / 核心符号**: `_MISSING`, `set_tensor`, `swap_tensor`, `swap_submodule`, `NamedMemberAccessor`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
