# _dynamism.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/experimental/_dynamism.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Holds experimental tracing, shape reasoning, and transformation infrastructure built on FX.
- **Purpose (CN)**: 存放基于 FX 的实验性跟踪、形状推理与变换基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
import re
from collections.abc import Callable
from typing import Any

import torch
from torch.utils._pytree import tree_flatten_with_path, tree_map


KeyPath = tuple[Any, ...]
NonTensorShapeFn = Callable[[int | float], tuple[Any, ...]]

__all__ = [
    "normalize_source_name",
    "module_to_nested_dict",
    "track_dynamism_across_examples",
    "clone_and_convert_to_meta",
]
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 19-33
```python

def normalize_source_name(name: str) -> str:
    # Match attribute access like .x and replace with ['x']
    return re.sub(r"\.([a-zA-Z_][a-zA-Z0-9_]*)", r"['\1']", name)


def module_to_nested_dict(module: torch.nn.Module) -> dict[str, Any]:
    """Recursively converts an nn.Module into a nested dictionary with explicit 'parameters' and 'modules' keys."""
    self_dict: dict[str, Any] = {}

    parameters: dict[str, torch.Tensor] = {}
    modules: dict[str, dict[str, Any]] = {}
    self_dict["_parameters"] = parameters
    self_dict["_modules"] = modules
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果。

### Lines 34-50
```python
    for attr_name in dir(module):
        try:
            if not attr_name.startswith("_") and not callable(
                getattr(module, attr_name)
            ):
                attr_value = getattr(module, attr_name)
                if (
                    not isinstance(attr_value, torch.nn.Module)
                    and isinstance(attr_value, (int, float, torch.Tensor))
                    and type(attr_value) is not bool
                ):
                    self_dict[attr_name] = attr_value
        except NotImplementedError:
            # Skip attributes that raise NotImplementedError since they won't
            # contain any dynamism anyways.
            continue
```
- **EN**: This block continues `module_to_nested_dict` and works to organize reusable module behavior and state. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `module_to_nested_dict`，用于组织可复用的模块行为与状态。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 51-62
```python
    for name, param in module.named_parameters(recurse=False):
        self_dict["_parameters"][name] = param
    for name, buffer in module.named_buffers(recurse=False):
        self_dict["_parameters"][name] = buffer

    for name, submodule in module.named_children():
        self_dict["_modules"][name] = module_to_nested_dict(submodule)

    return self_dict


def track_dynamism_across_examples(
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构。

### Lines 63-74
```python
    example_inputs: list[Any],
) -> dict[Any, Any]:
    """
    This function analyzes a list of example inputs to determine the dynamism of their shapes.
    It tracks whether the dimensions of tensors or non-tensor values change across
    different examples. The function returns a dictionary where each key represents
    a path to a value in the input examples, and the corresponding value is a tuple
    indicating which dimensions are dynamic (i.e., change across examples). This
    helps in understanding how the structure of data varies across different instances.
    """
    tracking: dict[KeyPath, tuple[list[set[Any]], bool]] = {}
```
- **EN**: This block continues `track_dynamism_across_examples` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `track_dynamism_across_examples`，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 75-86
```python
    for ex in example_inputs:
        if "self" in ex and isinstance(ex["self"], torch.nn.Module):
            ex["self"] = module_to_nested_dict(ex["self"])
        leaves_with_paths, _ = tree_flatten_with_path(ex)
        for key_path, value in leaves_with_paths:
            if not isinstance(value, (int, float, torch.Tensor)):
                continue
            if isinstance(value, torch.Tensor):
                shape: tuple[int | float, ...] = tuple(value.shape)
                is_tensor = True
            else:
                shape = (value,)
```
- **EN**: This block continues `track_dynamism_across_examples` and works to track symbolic shape constraints and shape-dependent reasoning. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `track_dynamism_across_examples`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 87-98
```python
                is_tensor = False
            if key_path not in tracking:
                tracking[key_path] = ([set() for _ in range(len(shape))], is_tensor)
            else:
                dim_sets, flag = tracking[key_path]
                if flag != is_tensor:
                    pass
                while len(dim_sets) < len(shape):
                    dim_sets.append(set())
            for i, dim in enumerate(shape):
                tracking[key_path][0][i].add(dim)
```
- **EN**: This block continues `track_dynamism_across_examples` and works to track symbolic shape constraints and shape-dependent reasoning. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `track_dynamism_across_examples`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 99-115
```python
    output: dict[Any, Any] = {}
    for key_path, (dim_sets, _is_tensor) in tracking.items():
        final_dyn = tuple(len(s) > 1 for s in dim_sets)
        key_str = "L" + "".join(f"{str(k)}" for k in key_path)
        key = key_path[0].key  # type: ignore[attr-defined]
        if key not in output:
            output[key] = {}
        output[key][key_str] = final_dyn
    return output


def clone_and_convert_to_meta(example_input: Any) -> Any:
    """
    This function takes a list of example inputs and for each tensor, clones it and converts it to device=meta.
    For non-tensor values, it keeps the reference. It uses pytree to handle nested structures recursively.
    """
```
- **EN**: This module-level block helps normalize dtype/device related arguments and behavior. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于规范化 dtype/device 相关参数与行为。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 116-121
```python
    def transform_fn(value: Any) -> Any:
        if isinstance(value, torch.Tensor):
            return value.clone().to(device="meta")
        return value

    return tree_map(transform_fn, example_input)
```
- **EN**: Defines the `clone_and_convert_to_meta` function; this block introduces logic that normalize dtype/device related arguments and behavior.
- **CN**: 定义`clone_and_convert_to_meta` 函数；该代码块引入了用于规范化 dtype/device 相关参数与行为的逻辑。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Program transforms / 程序变换**
  - EN: The implementation rewrites existing program structure into a more useful form.
  - CN: 该实现会把现有程序结构改写成更有用的形式。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.utils._pytree`
- **Standard library / 标准库**: `re`, `collections.abc`, `typing`
- **Primary symbols / 核心符号**: `__all__`, `normalize_source_name`, `module_to_nested_dict`, `track_dynamism_across_examples`, `clone_and_convert_to_meta`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
