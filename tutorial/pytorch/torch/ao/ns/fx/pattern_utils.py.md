# pattern_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/ns/fx/pattern_utils.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements numeric-suite tooling used to compare floating-point and quantized model behavior. This specific file centers on `pattern_utils.py`. The file is especially concerned with quantization metadata, calibration, or fake-quant behavior. It also participates in graph/module transformation before or after quantized execution. Runtime configuration, environment overrides, or feature gating are important in this module.
- **Purpose (CN) / 用途（中文）**: 实现数值套件工具，用于比较浮点模型与量化模型的行为。 该文件具体围绕 `pattern_utils.py` 展开。 该文件尤其关注量化元数据、校准流程或 fake-quant 行为。 它也参与量化执行前后的图/模块转换。 运行时配置、环境变量覆盖或特性开关是此模块的重要主题。
## Line-by-Line Analysis / 逐行分析

### Lines 1-17 / 第 1-17 行
```python
from collections.abc import Callable
from typing import Any

import torch
import torch.nn as nn
import torch.nn.functional as F
from torch.ao.quantization import FakeQuantizeBase, ObserverBase
from torch.ao.quantization.backend_config import get_native_backend_config
from torch.ao.quantization.fx.quantize_handler import _get_pattern_to_quantize_handlers
from torch.ao.quantization.utils import getattr_from_fqn
from torch.fx import GraphModule
from torch.fx.graph import Node

from .ns_types import NSNodeTargetType


toq = torch.ops.quantized
```
- **EN**: This block establishes the module dependencies, pulling in internal torch modules such as torch, torch.nn, torch.nn.functional, torch.ao.quantization:FakeQuantizeBase; standard-library helpers such as collections.abc:Callable, typing:Any. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段建立模块依赖，引入了内部 torch 模块，如 torch, torch.nn, torch.nn.functional, torch.ao.quantization:FakeQuantizeBase；标准库辅助模块，如 collections.abc:Callable, typing:Any。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 20-36 / 第 20-36 行
```python
def get_type_a_related_to_b(
    base_name_to_sets_of_related_ops: dict[str, set[NSNodeTargetType]],
) -> set[tuple[NSNodeTargetType, NSNodeTargetType]]:
    # TODO(future PR): allow customizations
    # TODO(future PR): reuse existing quantization mappings
    # TODO(future PR): add the rest of modules and ops here
    type_a_related_to_b: set[tuple[NSNodeTargetType, NSNodeTargetType]] = set()

    for s in base_name_to_sets_of_related_ops.values():
        s_list = list(s)
        # add every bidirectional pair
        for idx_0 in range(len(s_list)):
            for idx_1 in range(idx_0, len(s_list)):
                type_a_related_to_b.add((s_list[idx_0], s_list[idx_1]))
                type_a_related_to_b.add((s_list[idx_1], s_list[idx_0]))

    return type_a_related_to_b
```
- **EN**: Key callable entry points in this range include `get_type_a_related_to_b`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_type_a_related_to_b`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 39-58 / 第 39-58 行
```python
NSFusionElType = (
    Callable  # call_function or call_module type, example: F.linear or nn.Conv2d
    | str  # call_method name, example: "dequantize"
    | tuple[
        str, Any
    ]  # call_method name and first argument, example: ("to", torch.float16)
)
NSFusionType = (
    tuple[NSFusionElType, NSFusionElType]
    | tuple[NSFusionElType, NSFusionElType, NSFusionElType, NSFusionElType]
)


def get_reversed_fusions() -> list[tuple[NSFusionType, int]]:
    """
    Set of potential fusions, in reverse order.  The order is reversed
    to match how fusion patterns are defined in quantization code.

    Fusion format:
    ((fusion_op_0, fusion_op_1), base_op_idx)
```
- **EN**: Key callable entry points in this range include `get_reversed_fusions`. They package a focused unit of behavior behind named helpers or APIs. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_reversed_fusions`，它们把聚焦的行为封装成具名辅助函数或 API。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 60-83 / 第 60-83 行
```python
    Where base_op_idx is the idx of the op we should use to match other related
    ops. Note: base_op_idx is specified in non-reverse order, i.e. a base_op_idx
    of 0 represents the first op in regular (non-reverse) order, 1 represents the
    second op, etc.
    """
    results: list[tuple[NSFusionType, int]] = []

    # Possible syntaxes:
    # * single op: torch.nn.Conv2d
    # * multiple ops: (torch.nn.ReLU, torch.nn.Conv2d)
    # For fusions, we only care about patterns composed of multiple ops.
    # TODO(future PR): allow customizations from default patterns.
    all_quant_patterns = _get_pattern_to_quantize_handlers(get_native_backend_config())

    default_base_op_idx = 0
    for quant_pattern in all_quant_patterns:
        # TODO: this is a temporary hack to flatten the patterns from quantization so
        # that it works with the ns matcher function, maybe we should use `_is_match`
        # in torch.ao.quantization.fx.match_utils to match the patterns
        if (
            isinstance(quant_pattern, tuple)
            and len(quant_pattern) == 2
            and isinstance(quant_pattern[1], tuple)
            and len(quant_pattern[1]) == 2
```
- **EN**: Key callable entry points in this range include `get_reversed_fusions`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_reversed_fusions`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 84-104 / 第 84-104 行
```python
        ):
            # flatten the pattern with form (nn.ReLU, (nn.BatchNorm2d, nn.Conv2d))
            quant_pattern = (quant_pattern[0], quant_pattern[1][0], quant_pattern[1][1])

        # Only patterns of multiple ops are fusions, ignore
        # patterns which contain a single ops (they get matched
        # without caring about fusions).
        if isinstance(quant_pattern, tuple):
            results.append((quant_pattern, default_base_op_idx))  # type: ignore[arg-type]

        # For each pattern, add additional patterns with observers and
        # fake quants at the end.
        # TODO(future PR): if needed, implement matching for a node
        #   having multiple output observers.
        for cls in (ObserverBase, FakeQuantizeBase):
            if isinstance(quant_pattern, tuple):
                # pyrefly: ignore [not-iterable]
                new_pattern = (cls, *quant_pattern)
            else:
                new_pattern = (cls, quant_pattern)
            results.append((new_pattern, default_base_op_idx))  # type: ignore[arg-type]
```
- **EN**: Key callable entry points in this range include `get_reversed_fusions`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_reversed_fusions`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 106-129 / 第 106-129 行
```python
    # After this point, results contains values such as
    # [..., ((torch.nn.Relu, torch.nn.Conv2d), 0), ...]

    # Patterns for matching fp16 emulation are not specified in the quantization
    # fusion mappings.  For now, define them here.
    fp16_em_base_op_idx = 1
    patterns_to_add = [
        # linear-relu fp16 emulation:
        # fp16_to_fp32 -> linear -> relu -> fp32_to_fp16
        (
            (("to", torch.float16), F.relu, F.linear, "dequantize"),
            fp16_em_base_op_idx,
        ),
        # Conv-BN fusion (this happens outside of quantization patterns,
        # which is why it is defined separately here).
        ((nn.BatchNorm1d, nn.Conv1d), default_base_op_idx),
        ((nn.BatchNorm2d, nn.Conv2d), default_base_op_idx),
        ((nn.BatchNorm3d, nn.Conv3d), default_base_op_idx),
        ((nn.ReLU, nn.BatchNorm1d, nn.Conv1d), default_base_op_idx),
        ((nn.ReLU, nn.BatchNorm2d, nn.Conv2d), default_base_op_idx),
        ((nn.ReLU, nn.BatchNorm3d, nn.Conv3d), default_base_op_idx),
    ]
    for p in patterns_to_add:
        results.append(p)  # type: ignore[arg-type]
```
- **EN**: Key callable entry points in this range include `get_reversed_fusions`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_reversed_fusions`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 130-152 / 第 130-152 行
```python
        results.append(((ObserverBase, *p[0]), p[1]))  # type: ignore[arg-type]
        results.append(((FakeQuantizeBase, *p[0]), p[1]))  # type: ignore[arg-type]

    return results


def end_node_matches_reversed_fusion(
    end_node: Node,
    reversed_fusion: NSFusionType,
    gm: GraphModule,
    seen_nodes: set[Node],
) -> bool:
    """
    Returns true if a pattern ending with `end_node` matches
    the fusion pattern.
    """
    cur_node = end_node
    for fusion_idx in range(len(reversed_fusion)):
        # each node can only belong to one matched pattern
        if cur_node in seen_nodes:
            return False

        cur_fusion_el = reversed_fusion[fusion_idx]
```
- **EN**: Key callable entry points in this range include `get_reversed_fusions`, `end_node_matches_reversed_fusion`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. The surrounding logic is quantization-centric, so scale/zero-point choices or transformed execution paths matter.
- **CN**: 这一段的重要可调用入口包括 `get_reversed_fusions`, `end_node_matches_reversed_fusion`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 周边逻辑以量化为中心，因此 scale/zero-point 选择或变换后的执行路径很重要。

### Lines 154-177 / 第 154-177 行
```python
        if cur_node.op == "call_function":
            fusion_el_is_fun = (not isinstance(cur_fusion_el, str)) and (
                not isinstance(cur_fusion_el, type)
            )
            if fusion_el_is_fun:
                if cur_node.target != cur_fusion_el:
                    return False
                if len(cur_node.args) > 0 and isinstance(cur_node.args[0], Node):
                    cur_node = cur_node.args[0]
                else:
                    return False
            else:
                return False

        elif cur_node.op == "call_module":
            fusion_el_is_mod = isinstance(cur_fusion_el, type)
            if fusion_el_is_mod:
                if not isinstance(cur_node.target, str):
                    raise AssertionError(f"Expected str, got {type(cur_node.target)}")
                target_mod = getattr_from_fqn(gm, cur_node.target)
                if not isinstance(cur_fusion_el, type):
                    return False
                if not isinstance(target_mod, cur_fusion_el):
                    return False
```
- **EN**: Key callable entry points in this range include `end_node_matches_reversed_fusion`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `end_node_matches_reversed_fusion`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 178-201 / 第 178-201 行
```python
                if len(cur_node.args) > 0 and isinstance(cur_node.args[0], Node):
                    cur_node = cur_node.args[0]
                else:
                    return False
            else:
                return False

        elif cur_node.op == "call_method":
            fusion_el_is_meth_with_second_arg = (
                isinstance(cur_fusion_el, tuple) and len(cur_fusion_el) == 2
            )
            fusion_el_is_meth_without_args = isinstance(cur_fusion_el, str)
            if fusion_el_is_meth_without_args or fusion_el_is_meth_with_second_arg:
                if fusion_el_is_meth_without_args:
                    if cur_node.target != cur_fusion_el:
                        return False
                else:
                    if not isinstance(cur_fusion_el, tuple):
                        raise AssertionError(
                            f"Expected tuple, got {type(cur_fusion_el)}"
                        )
                    if cur_node.target != cur_fusion_el[0]:
                        return False
                    elif len(cur_node.args) < 2:
```
- **EN**: Key callable entry points in this range include `end_node_matches_reversed_fusion`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `end_node_matches_reversed_fusion`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 202-215 / 第 202-215 行
```python
                        return False
                    elif cur_node.args[1] != cur_fusion_el[1]:
                        return False

                if len(cur_node.args) > 0 and isinstance(cur_node.args[0], Node):
                    cur_node = cur_node.args[0]
                else:
                    return False
            else:
                return False
        else:
            return False

    return True
```
- **EN**: Key callable entry points in this range include `end_node_matches_reversed_fusion`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `end_node_matches_reversed_fusion`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements numeric-suite tooling used to compare floating-point and quantized model behavior.
  - CN: 实现数值套件工具，用于比较浮点模型与量化模型的行为。
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
- **Data loading**
  - EN: Coordinates dataset traversal, batching, multiprocessing workers, and sample collation.
  - CN: 协调数据集遍历、批处理、多进程 worker 以及样本拼装。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `torch.nn`, `torch.nn.functional`, `torch.ao.quantization:FakeQuantizeBase`, `torch.ao.quantization:ObserverBase`, `torch.ao.quantization.backend_config:get_native_backend_config`, `torch.ao.quantization.fx.quantize_handler:_get_pattern_to_quantize_handlers`, `torch.ao.quantization.utils:getattr_from_fqn`, `torch.fx:GraphModule`, `torch.fx.graph:Node`, `.ns_types:NSNodeTargetType`
- **Python standard library / Python 标准库**: `collections.abc:Callable`, `typing:Any`
- **Primary symbols / 核心符号**: `get_type_a_related_to_b`, `get_reversed_fusions`, `end_node_matches_reversed_fusion`
