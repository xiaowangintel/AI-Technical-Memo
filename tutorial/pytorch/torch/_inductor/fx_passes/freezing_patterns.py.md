# freezing_patterns.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/freezing_patterns.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It exposes functions such as `freezing_passes`, `lazy_init`, `register_freezing_graph_pattern`, `register_binary_folding_pattern`, `addmm_patterns_init`, `same_dtype`, and `...+1`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。同时提供 `freezing_passes`、`lazy_init`、`register_freezing_graph_pattern`、`register_binary_folding_pattern`、`addmm_patterns_init`、`same_dtype`、`另有1项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import functools

import torch
from torch._inductor.compile_fx import fake_tensor_prop
from torch._inductor.utils import GPU_TYPES

from ..._dynamo.utils import counters
from .. import config
from ..pattern_matcher import (
    _return_true,
    CallFunction,
    fwd_only,
    Ignored,
    init_once_fakemode,
    KeywordArg,
    Match,
    PatternMatcherPass,
    register_graph_pattern,
    register_replacement,
````
- **EN**: Imports dependencies such as `functools`, `torch`, `torch._inductor.compile_fx`, `torch._inductor.utils`, `..._dynamo.utils`, `..`, and `...+1` for the logic in this range. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里导入了 `functools`、`torch`、`torch._inductor.compile_fx`、`torch._inductor.utils`、`..._dynamo.utils`、`..`、`另有1项` 等依赖，为后续逻辑提供基础能力。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 21-40 / 第 21-40 行
````python
    stable_topological_sort,
)


aten = torch.ops.aten

# First pass_patterns[0] are applied, then [1], then [2]
pass_patterns = [
    PatternMatcherPass(),
    PatternMatcherPass(),
    PatternMatcherPass(),
]

binary_folding_pass = PatternMatcherPass()


def freezing_passes(gm: torch.fx.GraphModule, aot_example_inputs):
    """
    Passes that are applied to the graph to freeze pass.
    """
````
- **EN**: Introduces function `freezing_passes`. Initializes or updates values such as `aten`, `pass_patterns`, and `binary_folding_pass`.
- **CN**: 这里定义了函数`freezing_passes`。初始化或更新了 `aten`、`pass_patterns`、`binary_folding_pass` 等值。

### Lines 41-60 / 第 41-60 行
````python

    from ..freezing import constant_fold

    lazy_init()
    # We need a few rounds of binary folding to get rid of all the
    # unnecessary nodes, but may need a good method to chose the rounds number.
    # works like: conv+binary+binary.
    binary_folding = counters["inductor"]["binary_folding"]
    fake_tensor_prop(gm, aot_example_inputs, True)

    torch._inductor.fx_passes.binary_folding.mark_mixed_dtype_allowed_computation_ops(
        gm
    )
    for _ in range(4):
        constant_fold(gm)
        # Make sure meta['val'] is properly set for all nodes
        fake_tensor_prop(gm, aot_example_inputs, True)
        binary_folding_pass.apply(gm.graph)  # type: ignore[arg-type]
        # If we don't have binary folding, we don't need to run the pass again.
        # TODO: remove the need to run fake_tensor_prop on the whole model.
````
- **EN**: Imports dependencies such as `..freezing` for the logic in this range. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `binary_folding`. This range continues the implementation of function `freezing_passes`.
- **CN**: 这里导入了 `..freezing` 等依赖，为后续逻辑提供基础能力。包含分支、循环或上下文管理等控制流。初始化或更新了 `binary_folding` 等值。这一段延续了函数`freezing_passes` 的具体实现。

### Lines 61-80 / 第 61-80 行
````python
        if counters["inductor"]["binary_folding"] == binary_folding:
            break
        binary_folding = counters["inductor"]["binary_folding"]

    torch._inductor.fx_passes.binary_folding.recover_original_precision_folded_computation_ops(
        gm
    )

    constant_fold(gm)
    fake_tensor_prop(gm, aot_example_inputs, True)

    for pattern in pass_patterns:
        pattern.apply(gm.graph)  # type: ignore[arg-type]

    # The CPU weight packing always assume the conv's weight is channels last,
    # So make sure the layout_optimization is on when doing it.
    if (
        torch._C._has_mkldnn
        and config.cpp.weight_prepack
        and config.layout_optimization
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `binary_folding`. This range continues the implementation of function `freezing_passes`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `binary_folding` 等值。这一段延续了函数`freezing_passes` 的具体实现。

### Lines 81-100 / 第 81-100 行
````python
    ):
        from .mkldnn_fusion import _eliminate_duplicate_packed_nodes

        _eliminate_duplicate_packed_nodes(gm)

    stable_topological_sort(gm.graph)
    gm.recompile()
    gm.graph.lint()


@init_once_fakemode
def lazy_init(input_device: torch.device | None = None):
    if torch._C._has_mkldnn and config.cpp.weight_prepack:
        from .mkldnn_fusion import _mkldnn_weight_pack_init

        _mkldnn_weight_pack_init()

    from .binary_folding import binary_folding_init

    addmm_patterns_init()
````
- **EN**: Imports dependencies such as `.mkldnn_fusion`, and `.binary_folding` for the logic in this range. Introduces function `lazy_init`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里导入了 `.mkldnn_fusion`、`.binary_folding` 等依赖，为后续逻辑提供基础能力。这里定义了函数`lazy_init`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。

### Lines 101-120 / 第 101-120 行
````python
    binary_folding_init()


def register_freezing_graph_pattern(pattern, extra_check=_return_true, pass_number=0):
    while pass_number > len(pass_patterns) - 1:
        pass_patterns.append(PatternMatcherPass())
    return register_graph_pattern(
        pattern,
        extra_check=extra_check,
        # pyrefly: ignore [bad-argument-type]
        pass_dict=pass_patterns[pass_number],
    )


def register_binary_folding_pattern(pattern, extra_check=_return_true):
    return register_graph_pattern(
        pattern,
        extra_check=extra_check,
        # pyrefly: ignore [bad-argument-type]
        pass_dict=binary_folding_pass,
````
- **EN**: Introduces function `register_freezing_graph_pattern`, function `register_binary_folding_pattern`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`register_freezing_graph_pattern`、函数`register_binary_folding_pattern`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 121-140 / 第 121-140 行
````python
    )


@functools.cache
def addmm_patterns_init():
    """
    addmm related patterns.
    To avoid duplication, also includes int8 WoQ GEMM pattern without bias.
    """
    device = next(
        (gpu for gpu in GPU_TYPES if getattr(torch, gpu).is_available()), "cpu"
    )
    val = functools.partial(torch.empty, (10, 10), device=device, requires_grad=False)
    scale = functools.partial(torch.empty, (10,), device=device, requires_grad=False)

    def check_int8_woq_concat_linear_weights(match):
        is_cpu = match.kwargs["inp"].meta["val"].is_cpu
        if not is_cpu or not config.cpp.enable_concat_linear:
            # Currently, this pattern is only supported on CPU
            return False
````
- **EN**: Introduces function `addmm_patterns_init`, function `check_int8_woq_concat_linear_weights`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`addmm_patterns_init`、函数`check_int8_woq_concat_linear_weights`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。

### Lines 141-160 / 第 141-160 行
````python

        weight_inputs = ["w1", "w2"]
        if "w3" in match.kwargs:
            weight_inputs.append("w3")

        if not all(
            match.kwargs[wgt].target is torch.ops.prims.convert_element_type.default
            for wgt in weight_inputs
        ):
            return False

        if not all(
            next(iter(match.kwargs[wgt]._input_nodes.keys())).meta["val"].dtype
            is torch.int8
            for wgt in weight_inputs
        ):
            return False

        if not all(
            match.kwargs[wgt].meta["val"].dtype is torch.bfloat16
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `weight_inputs`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `weight_inputs` 等值。

### Lines 161-180 / 第 161-180 行
````python
            for wgt in weight_inputs
        ):
            return False

        return True

    def check_concat_weights(match):
        is_cpu = match.kwargs["inp"].meta["val"].is_cpu
        if is_cpu and not config.cpp.enable_concat_linear:
            return False

        weight_inputs = ["w1", "w2"]
        if "w3" in match.kwargs:
            weight_inputs.append("w3")

        equal_shape_inputs = [weight_inputs]

        if "b1" in match.kwargs:
            bias_inputs = ["b1", "b2"]
            if "b3" in match.kwargs:
````
- **EN**: Introduces function `check_concat_weights`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `is_cpu`, `weight_inputs`, `equal_shape_inputs`, and `bias_inputs`.
- **CN**: 这里定义了函数`check_concat_weights`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `is_cpu`、`weight_inputs`、`equal_shape_inputs`、`bias_inputs` 等值。

### Lines 181-200 / 第 181-200 行
````python
                bias_inputs.append("b3")

            equal_shape_inputs.append(bias_inputs)

        for equal_shape_group in equal_shape_inputs:
            inps = [match.kwargs[name] for name in equal_shape_group]

            if not all(
                inp.op == "get_attr"
                and inp.meta["val"].shape[:-1] == inps[0].meta["val"].shape[:-1]
                for inp in inps
            ):
                return False
        return True

    def int8_woq_fusion_pattern(inp, w1, w2, w3, s1, s2, s3):
        return ((inp @ w1) * s1, (inp @ w2) * s2, (inp @ w3) * s3)

    def int8_woq_fusion_replacement(inp, w1, w2, w3, s1, s2, s3):
        cat_w = torch.cat((w1, w2, w3), dim=1)
````
- **EN**: Introduces function `int8_woq_fusion_pattern`, function `int8_woq_fusion_replacement`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `inps`, and `cat_w`.
- **CN**: 这里定义了函数`int8_woq_fusion_pattern`、函数`int8_woq_fusion_replacement`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `inps`、`cat_w` 等值。

### Lines 201-220 / 第 201-220 行
````python
        cat_s = torch.cat((s1, s2, s3), dim=0)
        mm = (inp @ cat_w).mul(cat_s)
        n1, n2 = w1.size(1), w2.size(1)
        return mm.tensor_split([n1, n1 + n2], dim=-1)

    register_replacement(
        # pyrefly: ignore [bad-argument-type]
        int8_woq_fusion_pattern,
        # pyrefly: ignore [bad-argument-type]
        int8_woq_fusion_replacement,
        [val(), val(), val(), val(), scale(), scale(), scale()],
        # pyrefly: ignore [bad-argument-type]
        fwd_only,
        # pyrefly: ignore [bad-argument-type]
        pass_patterns[0],
        extra_check=check_int8_woq_concat_linear_weights,
        exclusive_arg_names=("w1", "w2", "w3", "s1", "s2", "s3"),
    )

    def matmul_fuse_pattern(inp, w1, w2, w3):
````
- **EN**: Introduces function `matmul_fuse_pattern`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `cat_s`, `mm`, `extra_check`, and `exclusive_arg_names`.
- **CN**: 这里定义了函数`matmul_fuse_pattern`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `cat_s`、`mm`、`extra_check`、`exclusive_arg_names` 等值。

### Lines 221-240 / 第 221-240 行
````python
        return (inp @ w1, inp @ w2, inp @ w3)

    def matmul_replacement(inp, w1, w2, w3):
        weights = (w1, w2, w3)
        cat_t = torch.cat(weights, dim=1)
        mm = inp @ cat_t
        return mm.split([w.size(1) for w in weights], dim=1)

    register_replacement(
        # pyrefly: ignore [bad-argument-type]
        matmul_fuse_pattern,
        # pyrefly: ignore [bad-argument-type]
        matmul_replacement,
        [val(), val(), val(), val()],
        # pyrefly: ignore [bad-argument-type]
        fwd_only,
        # pyrefly: ignore [bad-argument-type]
        pass_patterns[0],
        extra_check=check_concat_weights,
        exclusive_arg_names=("w1", "w2", "w3"),
````
- **EN**: Introduces function `matmul_replacement`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `weights`, `cat_t`, `mm`, `extra_check`, and `exclusive_arg_names`.
- **CN**: 这里定义了函数`matmul_replacement`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `weights`、`cat_t`、`mm`、`extra_check`、`exclusive_arg_names` 等值。

### Lines 241-260 / 第 241-260 行
````python
    )

    def matmul_fuse_pattern_two(inp, w1, w2):
        return (inp @ w1, inp @ w2)

    def matmul_replacement_two(inp, w1, w2):
        weights = (w1, w2)
        cat_t = torch.cat(weights, dim=1)
        mm = inp @ cat_t
        return mm.split([w.size(1) for w in weights], dim=1)

    register_replacement(
        # pyrefly: ignore [bad-argument-type]
        matmul_fuse_pattern_two,
        # pyrefly: ignore [bad-argument-type]
        matmul_replacement_two,
        [val(), val(), val()],
        # pyrefly: ignore [bad-argument-type]
        fwd_only,
        # pyrefly: ignore [bad-argument-type]
````
- **EN**: Introduces function `matmul_fuse_pattern_two`, function `matmul_replacement_two`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `weights`, `cat_t`, and `mm`.
- **CN**: 这里定义了函数`matmul_fuse_pattern_two`、函数`matmul_replacement_two`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `weights`、`cat_t`、`mm` 等值。

### Lines 261-280 / 第 261-280 行
````python
        pass_patterns[0],
        extra_check=check_concat_weights,
        exclusive_arg_names=("w1", "w2"),
    )

    def addmm_fuse_pattern_second(inp, w1, w2, w3, b1, b2, b3):
        return (
            aten.addmm(b1, inp, w1),
            aten.addmm(b2, inp, w2),
            aten.addmm(b3, inp, w3),
        )

    def addmm_fuse_replacement_second(inp, w1, w2, w3, b1, b2, b3):
        weights = (w1, w2, w3)
        cat_w = torch.cat(weights, dim=1)
        cat_b = torch.cat((b1, b2, b3))
        return aten.addmm(cat_b, inp, cat_w).split([w.size(1) for w in weights], dim=1)

    register_replacement(
        # pyrefly: ignore [bad-argument-type]
````
- **EN**: Introduces function `addmm_fuse_pattern_second`, function `addmm_fuse_replacement_second`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `extra_check`, `exclusive_arg_names`, `weights`, `cat_w`, and `cat_b`.
- **CN**: 这里定义了函数`addmm_fuse_pattern_second`、函数`addmm_fuse_replacement_second`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `extra_check`、`exclusive_arg_names`、`weights`、`cat_w`、`cat_b` 等值。

### Lines 281-300 / 第 281-300 行
````python
        addmm_fuse_pattern_second,
        # pyrefly: ignore [bad-argument-type]
        addmm_fuse_replacement_second,
        [val() for _ in range(7)],
        # pyrefly: ignore [bad-argument-type]
        fwd_only,
        # pyrefly: ignore [bad-argument-type]
        pass_patterns[0],
        extra_check=check_concat_weights,
        exclusive_arg_names=("w1", "w2", "w3", "b1", "b2", "b3"),
    )


def same_dtype(match):
    return match.output_node().args[0].meta["val"].dtype == match.kwargs["dtype"]


@register_graph_pattern(
    CallFunction(
        torch.ops.prims.convert_element_type.default,
````
- **EN**: Introduces function `same_dtype`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`same_dtype`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 301-313 / 第 301-313 行
````python
        Ignored(),
        KeywordArg("dtype"),
    ),
    # pyrefly: ignore [bad-argument-type]
    pass_dict=pass_patterns[0],
    extra_check=same_dtype,
)
def unnecessary_dtype_convert(match: Match, **kwargs):
    """Remove unnecessary dtype conversion op, probably left as a result of Conv-Bn folding"""
    graph = match.graph
    node = match.output_node()
    node.replace_all_uses_with(node.args[0])  # type: ignore[arg-type]
    graph.erase_node(node)
````
- **EN**: Introduces function `unnecessary_dtype_convert`. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `pass_dict`, `extra_check`, `graph`, and `node`.
- **CN**: 这里定义了函数`unnecessary_dtype_convert`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `pass_dict`、`extra_check`、`graph`、`node` 等值。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary functions: `freezing_passes`, `lazy_init`, `register_freezing_graph_pattern`, `register_binary_folding_pattern`, `addmm_patterns_init`, `same_dtype`, and `...+1`  
  **CN**: 主要函数：`freezing_passes`、`lazy_init`、`register_freezing_graph_pattern`、`register_binary_folding_pattern`、`addmm_patterns_init`、`same_dtype`、`另有1项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor.compile_fx`, `torch._inductor.utils`, `..._dynamo.utils`, `..`, `..pattern_matcher`, `..freezing`, `.binary_folding`, `.mkldnn_fusion`
