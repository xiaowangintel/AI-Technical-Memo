# replace_random.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/replace_random.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It exposes functions such as `_shape_to_offset`, `replace_random_passes`, `fuse_offset_creation_pass`, `fuse_seed_creation_pass`, `default_kwargs`, `get_device`, and `...+2`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。同时提供 `_shape_to_offset`、`replace_random_passes`、`fuse_offset_creation_pass`、`fuse_seed_creation_pass`、`default_kwargs`、`get_device`、`另有2项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import collections
import logging

import torch
from torch.fx.experimental.symbolic_shapes import guard_or_false, statically_known_true
from torch.fx.passes.graph_transform_observer import GraphTransformObserver
from torch.fx.passes.shape_prop import _extract_tensor_metadata

from .. import config, inductor_prims
from ..pattern_matcher import (
    CallFunctionVarArgs,
    Match,
    PatternMatcherPass,
    register_graph_pattern,
)
from ..virtualized import V


log = logging.getLogger(__name__)
````
- **EN**: Imports dependencies such as `collections`, `logging`, `torch`, `torch.fx.experimental.symbolic_shapes`, `torch.fx.passes.graph_transform_observer`, `torch.fx.passes.shape_prop`, and `...+3` for the logic in this range. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `log`.
- **CN**: 这里导入了 `collections`、`logging`、`torch`、`torch.fx.experimental.symbolic_shapes`、`torch.fx.passes.graph_transform_observer`、`torch.fx.passes.shape_prop`、`另有3项` 等依赖，为后续逻辑提供基础能力。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `log` 等值。

### Lines 21-40 / 第 21-40 行
````python
patterns = PatternMatcherPass(subsystem="joint_graph_passes")
aten = torch.ops.aten


def _shape_to_offset(shape, device: torch.device):
    # Modified from torch/_prims/rng_prims.py:philox_rand_offset
    nelem = 1
    for s in shape:
        nelem *= s

    # Empty tensor: no random numbers are generated/consumed.
    is_empty = nelem == 0
    if statically_known_true(is_empty) or guard_or_false(is_empty):
        return 0

    if device is None:
        device = torch.device("cpu")
    elif isinstance(device, str):
        device = torch.device(device)

````
- **EN**: Introduces function `_shape_to_offset`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_shape_to_offset`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 41-60 / 第 41-60 行
````python
    if device.type != "cuda":
        return 0

    block_size = 256
    unroll = 4
    curand4_engine_calls = 4

    device_property = torch.cuda.get_device_properties(device)

    blocks_per_sm = device_property.max_threads_per_multi_processor // block_size
    max_grid = device_property.multi_processor_count * blocks_per_sm
    grid_size = (nelem + block_size - 1) // block_size
    grid_size = -torch.sym_min(-grid_size, -1)
    grid_size = torch.sym_min(grid_size, max_grid)

    return ((nelem - 1) // (block_size * grid_size * unroll) + 1) * curand4_engine_calls


def replace_random_passes(gm: torch.fx.GraphModule):
    """Modify the given FX graph to use backend-native random ops"""
````
- **EN**: Introduces function `replace_random_passes`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`replace_random_passes`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 61-80 / 第 61-80 行
````python
    if config.fallback_random:
        return 0

    count = patterns.apply(gm)
    with GraphTransformObserver(gm, "fuse_seed_creation_pass", "joint_graph_passes"):
        count += fuse_seed_creation_pass(gm.graph)
    if config.align_random_eager:
        with GraphTransformObserver(gm, "fuse_offset_creation_pass"):
            count += fuse_offset_creation_pass(gm.graph)

    return count


def fuse_offset_creation_pass(graph: torch.fx.Graph) -> int:
    """
    Here offset node means seed << 32 + offset, will unpacked in lowering.py:inductor_random()
    Horizontally fuse all the seed generation on each device
        a = inductor_prims.rand_eager_offset(offset, dev)
        b = inductor_prims.rand_eager_offset(offset, dev)
    Becomes:
````
- **EN**: Introduces function `fuse_offset_creation_pass`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`fuse_offset_creation_pass`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 81-100 / 第 81-100 行
````python
        offsets = inductor_prims.rand_eager_offsets([offset1, offset2...], dev)
        a = torch.ops.aten.select.int(offsets, 0, 0)
        b = torch.ops.aten.select.int(offsets, 0, 1)
    We do this because seed creation is entirely launch overhead bound.
    """
    device_offsets = collections.defaultdict(list)
    for node in graph.nodes:
        if CallFunctionVarArgs(inductor_prims.rand_eager_offset).match(node):
            device_offsets[node.args[1]].append(node)

    if not device_offsets:
        return 0

    for device, offsets in device_offsets.items():
        with graph.inserting_before(offsets[0]):
            offs = [n.args[0] for n in offsets]
            combined = graph.call_function(
                inductor_prims.rand_eager_offsets, (offs, device)
            )
            with V.fake_mode:
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `offsets`, `a`, `b`, `device_offsets`, `offs`, and `combined`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `offsets`、`a`、`b`、`device_offsets`、`offs`、`combined` 等值。

### Lines 101-120 / 第 101-120 行
````python
                combined.meta["val"] = torch.empty(
                    [len(offsets), 2], device=device, dtype=torch.int64
                )
                combined.meta["tensor_meta"] = _extract_tensor_metadata(
                    combined.meta["val"]
                )

        for idx, offset in enumerate(offsets):
            with graph.inserting_before(offset):
                new_state = graph.call_function(
                    torch.ops.aten.select.int, (combined, 0, idx)
                )
            offset.replace_all_uses_with(new_state)
            new_state.meta.update(offset.meta)
            graph.erase_node(offset)

    return len(device_offsets)


def fuse_seed_creation_pass(graph: torch.fx.Graph):
````
- **EN**: Introduces function `fuse_seed_creation_pass`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`fuse_seed_creation_pass`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 121-140 / 第 121-140 行
````python
    """
    Horizontally fuse all the seed generation on each device

        a = inductor_seed(dev)
        b = inductor_seed(dev)

    Becomes:
        seeds = inductor_seeds(2, dev)
        a = inductor_lookup_seed(seeds, 0)
        b = inductor_lookup_seed(seeds, 1)

    We do this because seed creation is entirely launch overhead bound.
    """
    device_seeds = collections.defaultdict(list)
    for node in graph.nodes:
        if CallFunctionVarArgs(inductor_prims.seed).match(node):
            device_seeds[node.args[0]].append(node)

    if not device_seeds:
        return 0
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `a`, `b`, `Becomes`, `seeds`, and `device_seeds`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `a`、`b`、`Becomes`、`seeds`、`device_seeds` 等值。

### Lines 141-160 / 第 141-160 行
````python

    for device, seeds in device_seeds.items():
        with graph.inserting_before(seeds[0]):
            combined = graph.call_function(inductor_prims.seeds, (len(seeds), device))
            with V.fake_mode:
                combined.meta["val"] = torch.empty(
                    [len(seeds)], device=device, dtype=torch.int64
                )
                combined.meta["tensor_meta"] = _extract_tensor_metadata(
                    combined.meta["val"]
                )

        for idx, seed in enumerate(seeds):
            with graph.inserting_before(seed):
                new_seed = graph.call_function(
                    inductor_prims.lookup_seed, (combined, idx)
                )
            seed.replace_all_uses_with(new_seed)
            new_seed.meta.update(seed.meta)
            graph.erase_node(seed)
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `combined`, and `new_seed`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `combined`、`new_seed` 等值。

### Lines 161-180 / 第 161-180 行
````python

    return len(device_seeds)


def default_kwargs(device):
    return {}


def get_device(device):
    if device is not None:
        return device
    return torch.empty([]).device  # default device


# pyrefly: ignore [bad-argument-type]
@register_graph_pattern(CallFunctionVarArgs(aten.rand.default), pass_dict=patterns)
# pyrefly: ignore [bad-argument-type]
@register_graph_pattern(CallFunctionVarArgs(aten.rand.generator), pass_dict=patterns)
# pyrefly: ignore [bad-argument-type]
@register_graph_pattern(CallFunctionVarArgs(aten.randn.default), pass_dict=patterns)
````
- **EN**: Introduces function `default_kwargs`, function `get_device`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`default_kwargs`、函数`get_device`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 181-200 / 第 181-200 行
````python
# pyrefly: ignore [bad-argument-type]
@register_graph_pattern(CallFunctionVarArgs(aten.randn.generator), pass_dict=patterns)
def replace_random(
    match: Match,
    size,
    *,
    generator=None,
    dtype=None,
    device=None,
    layout=None,
    pin_memory=None,
):
    if generator is not None:
        return

    def replacement(size):
        result = inductor_prims.random(
            size, inductor_prims.seed(device), mode, **default_kwargs(device)
        )
        if dtype is not None:
````
- **EN**: Introduces function `replace_random`, function `replacement`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`replace_random`、函数`replacement`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 201-220 / 第 201-220 行
````python
            result = result.to(dtype)
        return result

    mode = {
        aten.rand: "rand",
        aten.randn: "randn",
    }[
        match.output_node().target.overloadpacket  # type: ignore[union-attr]
    ]  # type: ignore[union-attr]
    device = get_device(device)
    replacement_fn = replacement

    if mode == "rand" and config.align_random_eager and device.type == "cuda":
        # Only enable when align_random_eager is on.
        def replacement_align(size):
            offset = _shape_to_offset(size, device)

            align_dtype = dtype
            if isinstance(align_dtype, (tuple, list)):
                align_dtype = align_dtype[0] if len(align_dtype) else None
````
- **EN**: Introduces function `replacement_align`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`replacement_align`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 221-240 / 第 221-240 行
````python

            result = inductor_prims.random(
                size,
                inductor_prims.rand_eager_offset(offset, device),
                mode,
                **default_kwargs(device),
                align_dtype=align_dtype,
            )
            if dtype is not None:
                result = result.to(dtype)
            return result

        replacement_fn = replacement_align

    # pyrefly: ignore [bad-argument-type]
    match.replace_by_example(replacement_fn, [size])


# pyrefly: ignore [bad-argument-type]
@register_graph_pattern(CallFunctionVarArgs(aten.randint.low), pass_dict=patterns)
````
- **EN**: Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 241-258 / 第 241-258 行
````python
def replace_randint(
    match: Match,
    low,
    high,
    size,
    *,
    dtype=torch.int64,
    device=None,
    layout=None,
    pin_memory=None,
):
    def replacement(low, high, size):
        result = inductor_prims.randint(low, high, size, inductor_prims.seed(device))
        return result.to(dtype)

    device = get_device(device)
    # pyrefly: ignore [bad-argument-type]
    match.replace_by_example(replacement, [low, high, size])
````
- **EN**: Introduces function `replace_randint`, function `replacement`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`replace_randint`、函数`replacement`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary functions: `_shape_to_offset`, `replace_random_passes`, `fuse_offset_creation_pass`, `fuse_seed_creation_pass`, `default_kwargs`, `get_device`, and `...+2`  
  **CN**: 主要函数：`_shape_to_offset`、`replace_random_passes`、`fuse_offset_creation_pass`、`fuse_seed_creation_pass`、`default_kwargs`、`get_device`、`另有2项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections`, `logging`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.fx.experimental.symbolic_shapes`, `torch.fx.passes.graph_transform_observer`, `torch.fx.passes.shape_prop`, `..`, `..pattern_matcher`, `..virtualized`
