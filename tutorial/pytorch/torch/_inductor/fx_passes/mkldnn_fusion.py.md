# mkldnn_fusion.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/mkldnn_fusion.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes.
- **用途（中文）**: 该模块实现 FX 图变换 pass。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
# mypy: allow-untyped-defs
import functools
import operator
from functools import reduce
from typing import Any, TYPE_CHECKING

import torch
from torch._dynamo.utils import counters
from torch.fx.experimental.symbolic_shapes import has_free_symbols, optimization_hint
from torch.utils._ordered_set import OrderedSet

from .. import ir, mkldnn_ir
from ..lowering import lowerings as L
from ..pattern_matcher import (
    Arg,
    CallFunction,
    filter_nodes,
    get_arg_value,
    KeywordArg,
    MULTIPLE,
)
from ..utils import (
    is_mkldnn_bf16_supported,
    is_mkldnn_fp16_supported,
    SUPPORTED_MKLDNN_DEVICES,
)
from ..virtualized import ops, V
from .freezing_patterns import register_freezing_graph_pattern
````
- **EN**: Imports dependencies such as `functools`, `operator`, `typing`, `torch`, `torch._dynamo.utils`, `torch.fx.experimental.symbolic_shapes`, and `...+7` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里导入了 `functools`、`operator`、`typing`、`torch`、`torch._dynamo.utils`、`torch.fx.experimental.symbolic_shapes`、`另有7项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 29-56 / 第 29-56 行
````python
from .post_grad import register_lowering_pattern
from .quantization import (
    _register_int8_woq_concat_linear_pattern,
    _register_quantization_lowerings,
    _register_quantization_weight_pack_pass,
    _register_woq_lowerings,
)


if TYPE_CHECKING:
    from collections.abc import Callable


if torch._C._has_mkldnn:
    aten = torch.ops.aten
    mkldnn = torch.ops.mkldnn
    prims = torch.ops.prims

    _conv_args = [Arg() for _ in range(10)]
    _linear_args = [Arg() for _ in range(6)]
    _conv_transpose_args = [Arg() for _ in range(11)]

    class MkldnnDeviceOpBase:
        def get_linear_transpose_weight(self, weight_node):
            raise NotImplementedError

        def pack_conv_weight(
            self,
````
- **EN**: Imports dependencies such as `.post_grad`, `.quantization`, and `collections.abc` for the logic in this range. Introduces class `MkldnnDeviceOpBase`, function `get_linear_transpose_weight`, function `pack_conv_weight`. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里导入了 `.post_grad`、`.quantization`、`collections.abc` 等依赖，为后续逻辑提供基础能力。这里定义了类`MkldnnDeviceOpBase`、函数`get_linear_transpose_weight`、函数`pack_conv_weight`。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 57-84 / 第 57-84 行
````python
            graph,
            is_transposed,
            weight,
            constant_args,
            input_size,
        ):
            raise NotImplementedError

        def pack_linear_weight(
            self, graph, is_lp_weight, transpose_weight_node, batch_size
        ):
            raise NotImplementedError

        def pack_linear(
            self, graph, is_lp_weight, batch_size, input, packed_weight_node, bias
        ):
            raise NotImplementedError

    class CpuMkldnnDeviceOp(MkldnnDeviceOpBase):
        def get_linear_transpose_weight(self, weight_node):
            packed_weight_node = weight_node
            assert packed_weight_node.target == mkldnn._reorder_linear_weight
            transpose_weight_node = packed_weight_node.args[0]
            assert transpose_weight_node.target is aten.permute.default
            return transpose_weight_node

        def pack_conv_weight(
            self,
````
- **EN**: Introduces function `pack_linear_weight`, function `pack_linear`, class `CpuMkldnnDeviceOp`, function `get_linear_transpose_weight`, function `pack_conv_weight`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `packed_weight_node`, and `transpose_weight_node`.
- **CN**: 这里定义了函数`pack_linear_weight`、函数`pack_linear`、类`CpuMkldnnDeviceOp`、函数`get_linear_transpose_weight`、函数`pack_conv_weight`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `packed_weight_node`、`transpose_weight_node` 等值。

### Lines 85-112 / 第 85-112 行
````python
            graph,
            is_transposed,
            weight,
            constant_args,
            input_size,
        ):
            packed_weight_op = mkldnn._reorder_convolution_weight
            if is_transposed:
                packed_weight_op = mkldnn._reorder_convolution_transpose_weight

            # mkldnn_reorder_conv_weight(self, padding, stride, dilation, groups, input_size)
            packed_weight_inputs = (weight,) + tuple(constant_args) + (input_size,)
            return graph.create_node(
                "call_function", packed_weight_op, args=packed_weight_inputs
            )

        def pack_linear_weight(
            self, graph, is_lp_weight, transpose_weight_node, batch_size
        ):
            # For bfloat16 dynamic shape path, using input size hint to pack weight for a better performance.
            packed_weight_inputs = (
                transpose_weight_node,
                optimization_hint(batch_size)
                if has_free_symbols(batch_size)
                else batch_size,
            )

            # MKL packed matrix can't be copied to a different address because the internal implementation
````
- **EN**: Introduces function `pack_linear_weight`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`pack_linear_weight`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 113-140 / 第 113-140 行
````python
            # depends on the alignment of internally-stored metadata.
            # In aot mode, we need to firstly save the packed weight, when loading it,
            # it will be in a different address which doesn't work.
            # Disable MKL prepack linear in AOT mode.
            # Disable MKL prepack linear when batch_size has free symbols.
            packed_weight_op = (
                mkldnn._reorder_linear_weight
                if (
                    is_lp_weight
                    or mkldnn._is_mkldnn_acl_supported()
                    or V.aot_compilation
                    or has_free_symbols(batch_size)
                )
                else torch.ops.mkl._mkl_reorder_linear_weight
            )
            return graph.create_node(
                "call_function", packed_weight_op, args=packed_weight_inputs
            )

        def pack_linear(
            self, graph, is_lp_weight, batch_size, input, packed_weight_node, bias
        ):
            packed_linear_inputs: tuple[Any, ...] = (input, packed_weight_node)
            transpose_weight_node = packed_weight_node.args[0]
            if (
                is_lp_weight
                or mkldnn._is_mkldnn_acl_supported()
                or V.aot_compilation
````
- **EN**: Introduces function `pack_linear`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `packed_weight_op`, `packed_linear_inputs`, and `transpose_weight_node`.
- **CN**: 这里定义了函数`pack_linear`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `packed_weight_op`、`packed_linear_inputs`、`transpose_weight_node` 等值。

### Lines 141-168 / 第 141-168 行
````python
                or has_free_symbols(batch_size)
            ):
                packed_linear_inputs += (bias, "none", [], "")
                packed_linear_op: Callable[..., Any] = mkldnn._linear_pointwise.default
            else:
                packed_linear_inputs += (transpose_weight_node, bias, batch_size)
                packed_linear_op = torch.ops.mkl._mkl_linear

            return graph.create_node(
                "call_function", packed_linear_op, packed_linear_inputs
            )

    class XpuMkldnnDeviceOp(MkldnnDeviceOpBase):
        def pack_conv_weight(
            self,
            graph,
            is_transposed,
            weight,
            constant_args,
            input_size,
        ):
            assert not is_transposed, (
                "'mkldnn::_convolution_transpose_pointwise' is not currently implemented for the XPU device."
            )
            return weight

    def _get_mkldnn_device_op(device_type: str) -> MkldnnDeviceOpBase:
        """
````
- **EN**: Introduces class `XpuMkldnnDeviceOp`, function `pack_conv_weight`, function `_get_mkldnn_device_op`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`XpuMkldnnDeviceOp`、函数`pack_conv_weight`、函数`_get_mkldnn_device_op`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 169-196 / 第 169-196 行
````python
        Returns the MKLDNN device operation class based on the current device type.
        """
        if device_type == "cpu":
            return CpuMkldnnDeviceOp()
        elif device_type == "xpu":
            return XpuMkldnnDeviceOp()
        else:
            raise RuntimeError(f"MKLDNN is not supported on {device_type} device.")

    def _is_valid_grouped_gemm_fusion(computation_nodes):
        """
        Here we check:
        1. More than 1 GEMM nodes has been found.
        2. All the GEMM nodes share the same activation.
        3. All the GEMM nodes have same weight size but different wgt node.
        """
        computation_op = mkldnn._linear_pointwise.default
        act = computation_nodes[0].args[0]
        wgt = computation_nodes[0].args[1]
        wgt_size = wgt.meta.get("val").size()  # type: ignore[union-attr]
        return len(computation_nodes) >= 2 and all(
            (
                node.target == computation_op
                and node.args[0] == act
                and (node.args[1].meta.get("val").size() == wgt_size)
                and (node.args[1] != wgt or gemm_idx == 0)
            )
            for gemm_idx, node in enumerate(computation_nodes)
````
- **EN**: Introduces function `_is_valid_grouped_gemm_fusion`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_is_valid_grouped_gemm_fusion`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 197-224 / 第 197-224 行
````python
        )

    def grouped_gemm_pass(graph: torch.fx.Graph):
        """
        Group GEMM has multi output nodes which is complicated to define a Pattern.
        Use below way to connect the pattern to the lowering.
        TODO: Use MultiOutputPattern, current limitation is the pattern requires
        fixed number of output nodes. Extend to support Group GEMM for pattern matcher.
        """
        computation_op = mkldnn._linear_pointwise.default
        from ..mkldnn_lowerings import grouped_gemm_lowering

        for node in graph.find_nodes(op="call_function", target=computation_op):
            if (
                not node._erased
                and isinstance(node.meta.get("val"), torch.Tensor)
                and node.meta["val"].device.type == "cpu"
            ):
                act = node.args[0]
                users = list(act.users)
                if _is_valid_grouped_gemm_fusion(users):
                    with graph.inserting_before(node):
                        grouped_gemm_node = graph.create_node(
                            "call_function",
                            grouped_gemm_lowering,
                            (
                                act,
                                [user.args[1] for user in users],
````
- **EN**: Imports dependencies such as `..mkldnn_lowerings` for the logic in this range. Introduces function `grouped_gemm_pass`. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `..mkldnn_lowerings` 等依赖，为后续逻辑提供基础能力。这里定义了函数`grouped_gemm_pass`。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。

### Lines 225-252 / 第 225-252 行
````python
                                [user.args[2] for user in users],
                            ),
                        )
                        grouped_gemm_node.meta["val"] = [
                            user.meta["val"] for user in users
                        ]
                        with graph.inserting_after(grouped_gemm_node):
                            for gemm_idx, user in enumerate(users):
                                assert user.target == computation_op
                                get_item = graph.create_node(
                                    "call_function",
                                    operator.getitem,
                                    (
                                        grouped_gemm_node,
                                        gemm_idx,
                                    ),
                                )
                                user.replace_all_uses_with(get_item)
                                graph.erase_node(user)
        return

    def _conv_call(users=1):
        return CallFunction(
            mkldnn._convolution_pointwise.default, *_conv_args, _users=users
        )

    def _linear_call(users=1):
        return CallFunction(
````
- **EN**: Introduces function `_conv_call`, function `_linear_call`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `get_item`.
- **CN**: 这里定义了函数`_conv_call`、函数`_linear_call`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `get_item` 等值。

### Lines 253-280 / 第 253-280 行
````python
            mkldnn._linear_pointwise.default, *_linear_args, _users=users
        )

    def _conv_transpose_call(users=1):
        return CallFunction(
            mkldnn._convolution_transpose_pointwise.default,
            *_conv_transpose_args,
            _users=users,
        )

    def _to_float(input_call, users=1):
        return CallFunction(
            prims.convert_element_type.default,
            input_call,
            KeywordArg("to_float"),
            _users=users,
        )

    def _to_bf16(input_call):
        return CallFunction(
            prims.convert_element_type.default,
            input_call,
            KeywordArg("to_bf16"),
            _users=1,
        )

    def _to_fp16(input_call):
        return CallFunction(
````
- **EN**: Introduces function `_conv_transpose_call`, function `_to_float`, function `_to_bf16`, function `_to_fp16`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `_users`.
- **CN**: 这里定义了函数`_conv_transpose_call`、函数`_to_float`、函数`_to_bf16`、函数`_to_fp16`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `_users` 等值。

### Lines 281-308 / 第 281-308 行
````python
            prims.convert_element_type.default,
            input_call,
            KeywordArg("to_fp16"),
            _users=1,
        )

    def _unary_fusion_pattern(unary_fusion, call_fn, users, lowp_dtype):
        # only insert to_dtype if lowp_dtype is True
        computation_call = (
            _to_float(call_fn(), users=users) if lowp_dtype else call_fn(users=users)
        )
        out = unary_fusion(computation_call)
        if lowp_dtype == torch.bfloat16:
            return _to_bf16(out)
        elif lowp_dtype == torch.float16:
            return _to_fp16(out)
        else:
            return out

    def _gelu_fusion_1(computation_call):
        return CallFunction(
            aten.mul,
            CallFunction(aten.mul, computation_call, 0.5),
            CallFunction(
                aten.add,
                CallFunction(
                    aten.erf,
                    CallFunction(aten.mul, computation_call, 0.7071067811865476),
````
- **EN**: Introduces function `_unary_fusion_pattern`, function `_gelu_fusion_1`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_unary_fusion_pattern`、函数`_gelu_fusion_1`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 309-336 / 第 309-336 行
````python
                ),
                1,
            ),
        )

    def _gelu_fusion_2(computation_call):
        return CallFunction(
            aten.mul,
            CallFunction(aten.mul, computation_call, 0.5),
            CallFunction(
                aten.add,
                CallFunction(
                    aten.tanh,
                    CallFunction(
                        aten.mul,
                        CallFunction(
                            aten.add,
                            computation_call,
                            CallFunction(
                                aten.mul,
                                CallFunction(
                                    aten.mul,
                                    CallFunction(
                                        aten.mul, computation_call, computation_call
                                    ),
                                    computation_call,
                                ),
                                0.044715,
````
- **EN**: Introduces function `_gelu_fusion_2`. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_gelu_fusion_2`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 337-364 / 第 337-364 行
````python
                            ),
                        ),
                        0.7978845608028654,
                    ),
                ),
                1,
            ),
        )

    def _hardswish_fusion(computation_call):
        return CallFunction(
            aten.div,
            CallFunction(
                aten.mul,
                computation_call,
                CallFunction(
                    aten.clamp_max,
                    CallFunction(
                        aten.clamp_min, CallFunction(aten.add, computation_call, 3), 0
                    ),
                    6,
                ),
            ),
            6,
        )

    def _silu_fusion(computation_call):
        # Match: x / (exp(-x) + 1) pattern used by inductor's silu decomposition
````
- **EN**: Introduces function `_hardswish_fusion`, function `_silu_fusion`. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_hardswish_fusion`、函数`_silu_fusion`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 365-392 / 第 365-392 行
````python
        return CallFunction(
            aten.div,
            computation_call,
            CallFunction(
                aten.add,
                CallFunction(aten.exp, CallFunction(aten.neg, computation_call)),
                1,
            ),
        )

    def _hardsigmoid_fusion(computation_call):
        return CallFunction(
            aten.div,
            CallFunction(
                aten.clamp_max,
                CallFunction(
                    aten.clamp_min, CallFunction(aten.add, computation_call, 3), 0
                ),
                6,
            ),
            6,
        )

    def _leaky_relu_fusion(computation_call):
        return CallFunction(
            aten.where,
            CallFunction(aten.gt, computation_call, 0),
            computation_call,
````
- **EN**: Introduces function `_hardsigmoid_fusion`, function `_leaky_relu_fusion`. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_hardsigmoid_fusion`、函数`_leaky_relu_fusion`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 393-420 / 第 393-420 行
````python
            CallFunction(aten.mul, computation_call, KeywordArg("negative_slope")),
        )

    def _hardtanh_fusion(computation_call):
        return CallFunction(
            aten.clamp_max,
            CallFunction(aten.clamp_min, computation_call, KeywordArg("min_value")),
            KeywordArg("max_value"),
        )

    def _combined_fusion(computation_call, elementwise_op):
        return CallFunction(elementwise_op, computation_call)

    # binary_op(other, computation_op)
    def _binary_fusion_v1(computation_call, binary_fn):
        return CallFunction(binary_fn, KeywordArg("other"), computation_call)

    # binary_op(computation_op, other)
    def _binary_fusion_v2(computation_call, binary_fn):
        return CallFunction(binary_fn, computation_call, KeywordArg("other"))

    def _is_single_computation_op(computation_op, lowp_dtype=None):
        def fn(match):
            computation_nodes = filter_nodes(match.nodes, computation_op)

            if lowp_dtype:
                output_node_meta = match.output_node().meta.get("val")
                if output_node_meta.dtype != lowp_dtype:
````
- **EN**: Introduces function `_hardtanh_fusion`, function `_combined_fusion`, function `_binary_fusion_v1`, function `_binary_fusion_v2`, function `_is_single_computation_op`, function `fn`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_hardtanh_fusion`、函数`_combined_fusion`、函数`_binary_fusion_v1`、函数`_binary_fusion_v2`、函数`_is_single_computation_op`、函数`fn`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 421-448 / 第 421-448 行
````python
                    return False

            if len(computation_nodes) < 1:
                return False
            if any(n.args[-3] != "none" for n in computation_nodes):
                return False
            return True

        return fn

    def _is_valid_computation_unary_fusion(computation_op, lowp_dtype=None):
        def fn(match):
            matched = _is_single_computation_op(computation_op, lowp_dtype)(match)
            computation_node = filter_nodes(match.nodes, computation_op)[0]
            if lowp_dtype:
                conversion_dtype_nodes = filter_nodes(
                    match.nodes, prims.convert_element_type.default
                )
                if len(conversion_dtype_nodes) != 2:
                    return False
                # fusion pattern is always in the form of computation_op + to_float32 + unary_op + to_bfloat16
                if computation_node == conversion_dtype_nodes[0].args[0]:
                    to_float = conversion_dtype_nodes[0].args[1]
                    to_lp = conversion_dtype_nodes[1].args[1]
                else:
                    to_float = conversion_dtype_nodes[1].args[1]
                    to_lp = conversion_dtype_nodes[0].args[1]
                matched = matched and to_float == torch.float and to_lp == lowp_dtype
````
- **EN**: Introduces function `_is_valid_computation_unary_fusion`, function `fn`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `matched`, `computation_node`, `conversion_dtype_nodes`, `to_float`, `to_lp`, and `else`.
- **CN**: 这里定义了函数`_is_valid_computation_unary_fusion`、函数`fn`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `matched`、`computation_node`、`conversion_dtype_nodes`、`to_float`、`to_lp`、`else` 等值。

### Lines 449-476 / 第 449-476 行
````python
            return matched

        return fn

    def _register_unary_fusion_lowering(
        pattern, unary_attr, computation_op, lowp_dtype=None
    ):
        @register_lowering_pattern(
            pattern,
            extra_check=_is_valid_computation_unary_fusion(computation_op, lowp_dtype),
        )
        def fn(match, *args, **kwargs):
            computation_args = list(args)[:-3] + [
                unary_attr.op_name,
                unary_attr.scalars_attr,
                unary_attr.algorithm_attr,
            ]
            counters["inductor"]["mkldnn_unary_fusion_matcher_count"] += 1
            counters["inductor"]["mkldnn_unary_fusion_matcher_nodes"] += len(
                match.nodes
            )
            return L[computation_op](*computation_args)

        return fn

    def _register_leaky_relu_fusion_lowering(pattern, computation_op, lowp_dtype=None):
        @register_lowering_pattern(
            pattern, extra_check=_is_single_computation_op(computation_op, lowp_dtype)
````
- **EN**: Introduces function `_register_unary_fusion_lowering`, function `fn`, function `_register_leaky_relu_fusion_lowering`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_register_unary_fusion_lowering`、函数`fn`、函数`_register_leaky_relu_fusion_lowering`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 477-504 / 第 477-504 行
````python
        )
        def fn(match, *args, **kwargs):
            negative_slope = kwargs.get("negative_slope")
            if isinstance(negative_slope, ir.TensorBox):
                matched = False
            else:  # inp is a Number
                matched = True
            if lowp_dtype:
                dtype1 = kwargs.get("to_float")
                dtype2 = (
                    kwargs.get("to_bf16")
                    if lowp_dtype == torch.bfloat16
                    else kwargs.get("to_fp16")
                )
                matched = matched and dtype1 == torch.float and dtype2 == lowp_dtype
            computation_args = list(args)
            counters["inductor"]["mkldnn_unary_fusion_matcher_count"] += 1
            counters["inductor"]["mkldnn_unary_fusion_matcher_nodes"] += len(
                match.nodes
            )
            if matched:
                computation_args = computation_args[:-3] + [
                    "leaky_relu",
                    [negative_slope],
                    "",
                ]
                return L[computation_op](*computation_args)
            else:
````
- **EN**: Introduces function `fn`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`fn`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 505-532 / 第 505-532 行
````python
                # computation_args += ["none", [], ""]
                out = L[computation_op](*computation_args)
                if lowp_dtype:
                    out = L[prims.convert_element_type.default](out, dtype=torch.float)
                out = L[aten.where](
                    L[aten.gt](out, 0),
                    out,
                    L[aten.mul](out, negative_slope),
                )
                if lowp_dtype:
                    out = L[prims.convert_element_type.default](out, dtype=dtype2)  # type: ignore[possibly-undefined]
                return out

        return fn

    def _register_hardtanh_fusion_lowering(pattern, computation_op, lowp_dtype=None):
        @register_lowering_pattern(
            pattern, extra_check=_is_single_computation_op(computation_op, lowp_dtype)
        )
        def fn(match, *args, **kwargs):
            min_value = kwargs.get("min_value")
            max_value = kwargs.get("max_value")
            if isinstance(min_value, ir.TensorBox) or isinstance(
                max_value, ir.TensorBox
            ):
                matched = False
            else:  # inp is a Number
                assert max_value is not None
````
- **EN**: Introduces function `_register_hardtanh_fusion_lowering`, function `fn`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`_register_hardtanh_fusion_lowering`、函数`fn`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 533-560 / 第 533-560 行
````python
                matched = min_value <= max_value
            if lowp_dtype:
                dtype1 = kwargs.get("to_float")
                dtype2 = (
                    kwargs.get("to_bf16")
                    if lowp_dtype == torch.bfloat16
                    else kwargs.get("to_fp16")
                )
                matched = matched and dtype1 == torch.float and dtype2 == lowp_dtype
            computation_args = list(args)
            counters["inductor"]["mkldnn_unary_fusion_matcher_count"] += 1
            counters["inductor"]["mkldnn_unary_fusion_matcher_nodes"] += len(
                match.nodes
            )
            if matched:
                computation_args = computation_args[:-3] + [
                    "hardtanh",
                    [min_value, max_value],
                    "",
                ]
                return L[computation_op](*computation_args)
            else:
                out = L[computation_op](*computation_args)
                if lowp_dtype:
                    out = L[prims.convert_element_type.default](out, dtype=torch.float)
                out = L[aten.clamp_max](L[aten.clamp_min](out, min_value), max_value)
                if lowp_dtype:
                    out = L[prims.convert_element_type.default](out, dtype=dtype2)  # type: ignore[possibly-undefined]
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `matched`, `dtype1`, `dtype2`, `computation_args`, `else`, and `out`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `matched`、`dtype1`、`dtype2`、`computation_args`、`else`、`out` 等值。

### Lines 561-588 / 第 561-588 行
````python
                return out

        return fn

    _binary_attr = {
        aten.add: "add",
        ops.add: "add",
        aten.sub: "sub",
        ops.sub: "sub",
    }

    def _is_valid_binary(match, computation_op, binary_op):
        binary_nodes = filter_nodes(match.nodes, binary_op)
        if len(binary_nodes) < 1:
            return False

        def get_meta_value(argument: torch.fx.node.Argument):
            # Only torch.fx.Node is expected to have meta.
            if isinstance(argument, torch.fx.Node):
                return argument.meta.get("val", None)
            return None

        if any(
            not isinstance(get_meta_value(n.args[0]), torch.Tensor)
            or not isinstance(get_meta_value(n.args[1]), torch.Tensor)
            for n in binary_nodes
        ):
            return False
````
- **EN**: Introduces function `_is_valid_binary`, function `get_meta_value`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `_binary_attr`, and `binary_nodes`.
- **CN**: 这里定义了函数`_is_valid_binary`、函数`get_meta_value`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `_binary_attr`、`binary_nodes` 等值。

### Lines 589-616 / 第 589-616 行
````python
        # check alpha is one.
        if any(
            get_arg_value(n, 2, kwarg_name="alpha") != 1.0
            and get_arg_value(n, 2, kwarg_name="alpha") is not None
            for n in binary_nodes
        ):
            return False

        def _check_input_sizes(n, computation_op):
            # Check if the tensor shape of the 'other' node is the same as or
            # can be broadcasted to the tensor shape of the computation node.
            computation_node = (
                n.args[0] if n.args[1] is match.kwargs["other"] else n.args[1]
            )
            assert computation_node.target == computation_op
            computation_node_size = get_meta_value(computation_node).size()
            if computation_op is mkldnn._linear_pointwise.default:
                broadcast_sizes = []
                if len(computation_node_size) >= 2:
                    broadcast_sizes = [
                        torch.Size(
                            [1 for _ in range(len(computation_node_size) - 1)]
                            + [computation_node_size[-1]]
                        ),
                    ]
            else:
                assert len(computation_node_size) > 2
                broadcast_sizes = [
````
- **EN**: Introduces function `_check_input_sizes`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `computation_node`, `computation_node_size`, `broadcast_sizes`, and `else`.
- **CN**: 这里定义了函数`_check_input_sizes`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `computation_node`、`computation_node_size`、`broadcast_sizes`、`else` 等值。

### Lines 617-644 / 第 617-644 行
````python
                    torch.Size(
                        [computation_node_size[0], computation_node_size[1]]
                        + [1 for _ in range(len(computation_node_size) - 2)]
                    ),
                    torch.Size(
                        [1, computation_node_size[1]]
                        + [1 for _ in range(len(computation_node_size) - 2)]
                    ),
                    torch.Size([1 for _ in range(len(computation_node_size))]),
                ]
            return (
                get_meta_value(match.kwargs["other"]).size()
                in [
                    computation_node_size,
                ]
                + broadcast_sizes
            )

        if any(
            not _check_input_sizes(n, computation_op)
            or get_meta_value(n.args[0]).device != get_meta_value(n.args[1]).device
            or get_meta_value(n.args[0]).dtype != get_meta_value(n.args[1]).dtype
            for n in binary_nodes
        ):
            return False
        # check args[0] and args[1] is not same
        if any(n.args[0] == n.args[1] for n in binary_nodes):
            return False
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 645-672 / 第 645-672 行
````python
        return True

    def _is_valid_computation_binary(computation_op, binary_op, other_index=None):
        def fn(match):
            if not _is_single_computation_op(computation_op)(match):
                return False
            if not _is_valid_binary(match, computation_op, binary_op):
                return False
            return True

        return fn

    def _get_remaining_users(extra_input_node, compute_node):
        # Think about this pattern:
        #      ReLU
        #     /   \
        #  Conv1
        #   /      \
        # Conv2
        #   \      /
        #      Add
        # Although, the extra input node (ReLU) has more than 1 users: Conv1 and Add.
        # The Conv1 is the ancestor node of the current compute node (Conv2).
        # This indicates that the buffer of ReLU has completed all its usage,
        # So we can safely make changes to it now by doing Conv2->Add inplace fusion.
        # Take above case as example:
        # * extra_input_node: ReLU
        # * compute_node: Conv2
````
- **EN**: Introduces function `_is_valid_computation_binary`, function `fn`, function `_get_remaining_users`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_is_valid_computation_binary`、函数`fn`、函数`_get_remaining_users`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 673-700 / 第 673-700 行
````python
        # _get_remaining_users will return the users of extra_input_node which are not
        # ancestor node of compute_node.
        def _is_ancestor_node(_current_node, _ancestor_node):
            # Check whether _ancestor_node is the ancestor node of _current_node
            _node_list = [_current_node]
            _visited_nodes = OrderedSet[torch.fx.Node]()
            while len(_node_list) != 0:
                _current_node = _node_list.pop(0)
                if _current_node not in _visited_nodes:
                    _visited_nodes.add(_current_node)
                    if _current_node == _ancestor_node:
                        return True
                    elif isinstance(
                        _current_node, torch.fx.Node
                    ) and _current_node.op not in ["placeholder", "output", "get_attr"]:
                        for input in _current_node.all_input_nodes:
                            _node_list.append(input)  # noqa: PERF402
            return False

        return [
            user
            for user in list(extra_input_node.users)
            if not _is_ancestor_node(compute_node, user)
        ]

    def _is_valid_computation_binary_inplace(computation_op, binary_op, other_index):
        def fn(match):
            if not _is_valid_computation_binary(computation_op, binary_op)(match):
````
- **EN**: Introduces function `_is_ancestor_node`, function `_is_valid_computation_binary_inplace`, function `fn`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `_node_list`, `_visited_nodes`, and `_current_node`.
- **CN**: 这里定义了函数`_is_ancestor_node`、函数`_is_valid_computation_binary_inplace`、函数`fn`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `_node_list`、`_visited_nodes`、`_current_node` 等值。

### Lines 701-728 / 第 701-728 行
````python
                return False
            binary_nodes = filter_nodes(match.nodes, binary_op)

            def _get_compute_node(_binary_node, _other_index):
                assert len(_binary_node.all_input_nodes) == 2, (
                    "Binary node should have 2 input nodes."
                )
                _compute_index = 1 if (_other_index == 0) else 0
                return _binary_node.args[_compute_index]

            def _other_input_not_inplaceable(_binary_node, _other_index):
                _compute_node = _get_compute_node(_binary_node, _other_index)
                return (
                    len(
                        _get_remaining_users(
                            _binary_node.args[_other_index], _compute_node
                        )
                    )
                    > 1
                    or _binary_node.args[_other_index] == _compute_node.args[0]
                )

            if any(_other_input_not_inplaceable(n, other_index) for n in binary_nodes):
                return False
            if any(
                n.args[other_index].op in ["placeholder", "output"]
                for n in binary_nodes
            ):
````
- **EN**: Introduces function `_get_compute_node`, function `_other_input_not_inplaceable`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `binary_nodes`, `_compute_index`, and `_compute_node`.
- **CN**: 这里定义了函数`_get_compute_node`、函数`_other_input_not_inplaceable`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `binary_nodes`、`_compute_index`、`_compute_node` 等值。

### Lines 729-756 / 第 729-756 行
````python
                return False
            return True

        return fn

    def _register_binary_unary_fusion_lowering(
        pattern,
        computation_op,
        binary_op,
        fusion_op,
        unary_attr=None,
    ):
        @register_lowering_pattern(
            pattern, extra_check=_is_valid_computation_binary(computation_op, binary_op)
        )
        def fn(match, *args, **kwargs):
            other = kwargs.get("other")
            assert isinstance(other, ir.TensorBox)
            binary_attr = _binary_attr[binary_op]
            args_list = list(args)
            computation_args = [args_list[0], other] + args_list[1:-3] + [binary_attr]
            if len(args_list) > 6:
                if unary_attr is not None:
                    computation_args += [
                        1.0,
                        unary_attr.op_name,
                        unary_attr.scalars_attr,
                        unary_attr.algorithm_attr,
````
- **EN**: Introduces function `_register_binary_unary_fusion_lowering`, function `fn`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_register_binary_unary_fusion_lowering`、函数`fn`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 757-784 / 第 757-784 行
````python
                    ]
                else:
                    computation_args += [1.0, None, [], None]
            counters["inductor"]["mkldnn_conv_binary_unary_fusion_matcher_count"] += 1
            counters["inductor"]["mkldnn_conv_binary_unary_fusion_matcher_nodes"] += (
                len(match.nodes)
            )
            return L[fusion_op](*computation_args)

        return fn

    def _can_be_inplace(_other):
        return not (
            isinstance(_other.data, ir.BaseView)
            or len(_other.get_inputs_that_alias_output()) > 0
        )

    def _qlinear_binary_can_be_inplace(_other):
        if isinstance(_other.data, ir.BaseView):

            def unwrap_buffer(data):
                if isinstance(data, ir.StorageBox):
                    return data.data
                return data

            data = _other.data.unwrap_view()
            if isinstance(unwrap_buffer(data), ir.CppTemplateBuffer):
                # It can be inplaced when _other is the 2D to 3D view of
````
- **EN**: Introduces function `_can_be_inplace`, function `_qlinear_binary_can_be_inplace`, function `unwrap_buffer`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, and `data`.
- **CN**: 这里定义了函数`_can_be_inplace`、函数`_qlinear_binary_can_be_inplace`、函数`unwrap_buffer`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`data` 等值。

### Lines 785-812 / 第 785-812 行
````python
                # a CppTemplateBuffer because if there is a view of CppTemplateBuffer,
                # CppTemplateBuffer will not be used directly but the view.
                return True
            else:
                # The case of QLinearPointwiseBinaryPT2E(sum) -> QLinearPointwiseBinaryPT2E(sum)
                # is similar to CppTemplateBuffer above.
                # The output of previous QLinearPointwiseBinaryPT2E is
                # the input x2 of current QLinearPointwiseBinaryPT2E.
                # Use V.graph.operations to check if _other is a view of the output
                # of previous QLinearPointwiseBinaryPT2E (the inputs[6]).
                for op in V.graph.operations:
                    if (
                        isinstance(op, mkldnn_ir.QLinearPointwiseBinaryPT2E)
                        and unwrap_buffer(data) == op.inputs[6]  # type: ignore[attr-defined]
                    ):
                        return True
            return False
        elif len(_other.get_inputs_that_alias_output()) > 0:
            return False
        else:
            return True

    def _register_binary_unary_maybe_inplace_fusion_lowering(
        pattern,
        computation_op,
        binary_op,
        inplace_fusion_op,
        outplace_fusion_op,
````
- **EN**: Introduces function `_register_binary_unary_maybe_inplace_fusion_lowering`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_register_binary_unary_maybe_inplace_fusion_lowering`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 813-840 / 第 813-840 行
````python
        unary_attr=None,
        other_index=None,
    ):
        @register_lowering_pattern(
            pattern,
            extra_check=_is_valid_computation_binary_inplace(
                computation_op, binary_op, other_index
            ),
        )
        def fn(match, *args, **kwargs):
            other = kwargs.get("other")
            assert isinstance(other, ir.TensorBox)
            binary_attr = _binary_attr[binary_op]
            args_list = list(args)
            computation_args = [args_list[0], other] + args_list[1:-3] + [binary_attr]
            if len(args_list) > 6:
                if unary_attr is not None:
                    computation_args += [
                        1.0,
                        unary_attr.op_name,
                        unary_attr.scalars_attr,
                        unary_attr.algorithm_attr,
                    ]
                else:
                    computation_args += [1.0, None, [], None]
            counters["inductor"]["mkldnn_conv_binary_unary_fusion_matcher_count"] += 1
            counters["inductor"]["mkldnn_conv_binary_unary_fusion_matcher_nodes"] += (
                len(match.nodes)
````
- **EN**: Introduces function `fn`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`fn`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 841-868 / 第 841-868 行
````python
            )
            # Make sure the other is not an alias or mutation(fx side doesn't has such info).
            other.realize()
            if not _can_be_inplace(other) or other.data.shape != list(
                match.nodes[0].meta["val"].size()
            ):
                return L[outplace_fusion_op](*computation_args)
            return L[inplace_fusion_op](*computation_args)

        return fn

    computation_ops = [
        mkldnn._convolution_pointwise.default,
        mkldnn._linear_pointwise.default,
        mkldnn._convolution_transpose_pointwise.default,
    ]

    class UnaryAttr:
        def __init__(
            self, op_name: str, scalars_attr=None, algorithm_attr=None
        ) -> None:
            self.op_name = op_name
            self.scalars_attr = scalars_attr if scalars_attr else []
            self.algorithm_attr = algorithm_attr if algorithm_attr else ""

    def _register_unary_fusion():
        computation_call_fns = [_conv_call, _linear_call, _conv_transpose_call]

````
- **EN**: Introduces class `UnaryAttr`, function `__init__`, function `_register_unary_fusion`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`UnaryAttr`、函数`__init__`、函数`_register_unary_fusion`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 869-896 / 第 869-896 行
````python
        def _unary_fusion_patterns(lowp_dtype):
            replacement_unary_fusion_patterns = {
                UnaryAttr("gelu", algorithm_attr="tanh"): [
                    _unary_fusion_pattern(_gelu_fusion_2, call_fn, 4, lowp_dtype)
                    for call_fn in computation_call_fns
                ],
                UnaryAttr("gelu", algorithm_attr="none"): [
                    _unary_fusion_pattern(_gelu_fusion_1, call_fn, 2, lowp_dtype)
                    for call_fn in computation_call_fns
                ],
                UnaryAttr("hardswish"): [
                    _unary_fusion_pattern(_hardswish_fusion, call_fn, 2, lowp_dtype)
                    for call_fn in computation_call_fns
                ],
                UnaryAttr("hardsigmoid"): [
                    _unary_fusion_pattern(_hardsigmoid_fusion, call_fn, 1, lowp_dtype)
                    for call_fn in computation_call_fns
                ],
                UnaryAttr("swish"): [
                    _unary_fusion_pattern(_silu_fusion, call_fn, 2, lowp_dtype)
                    for call_fn in computation_call_fns
                ],
            }
            if not lowp_dtype:
                call_user1 = [call_fn(users=1) for call_fn in computation_call_fns]
                replacement_unary_fusion_patterns.update(
                    {
                        UnaryAttr("relu"): [
````
- **EN**: Introduces function `_unary_fusion_patterns`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `replacement_unary_fusion_patterns`, and `call_user1`.
- **CN**: 这里定义了函数`_unary_fusion_patterns`。包含分支、循环或上下文管理等控制流。初始化或更新了 `replacement_unary_fusion_patterns`、`call_user1` 等值。

### Lines 897-924 / 第 897-924 行
````python
                            _combined_fusion(u, aten.relu) for u in call_user1
                        ],
                        UnaryAttr("sigmoid"): [
                            _combined_fusion(u, aten.sigmoid) for u in call_user1
                        ],
                        UnaryAttr("tanh"): [
                            _combined_fusion(u, aten.tanh) for u in call_user1
                        ],
                    }
                )

            return replacement_unary_fusion_patterns

        for lowp_dtype in [torch.bfloat16, torch.float16, None]:
            replace_patterns = _unary_fusion_patterns(lowp_dtype)
            for unary_attr, patterns in replace_patterns.items():
                _register_unary_fusion_lowering(
                    patterns[0], unary_attr, computation_ops[0], lowp_dtype
                )
                _register_unary_fusion_lowering(
                    patterns[1], unary_attr, computation_ops[1], lowp_dtype
                )
                _register_unary_fusion_lowering(
                    patterns[2], unary_attr, computation_ops[2], lowp_dtype
                )
            _leaky_relu_patterns = [
                _unary_fusion_pattern(_leaky_relu_fusion, call_fn, 3, lowp_dtype)
                for call_fn in computation_call_fns
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 925-952 / 第 925-952 行
````python
            ]
            for pattern, computation_op in zip(_leaky_relu_patterns, computation_ops):
                _register_leaky_relu_fusion_lowering(
                    pattern, computation_op, lowp_dtype
                )
            hardtanh_patterns = [
                _unary_fusion_pattern(_hardtanh_fusion, call_fn, 1, lowp_dtype)
                for call_fn in computation_call_fns
            ]
            for pattern, computation_op in zip(hardtanh_patterns, computation_ops):
                _register_hardtanh_fusion_lowering(pattern, computation_op, lowp_dtype)

    def _register_inplace_fusion():
        binary_ops = [aten.add, ops.add]
        inplace_fusion_op = mkldnn._convolution_pointwise_.binary
        outplace_fusion_op = mkldnn._convolution_pointwise.binary
        conv_call = _conv_call(users=1)
        conv_op = computation_ops[0]
        for binary_op in binary_ops:
            binary_v1 = _binary_fusion_v1(conv_call, binary_op)
            binary_unary_v1 = _combined_fusion(binary_v1, aten.relu)
            _register_binary_unary_maybe_inplace_fusion_lowering(
                binary_unary_v1,
                conv_op,
                binary_op,
                inplace_fusion_op,
                outplace_fusion_op,
                other_index=0,
````
- **EN**: Introduces function `_register_inplace_fusion`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `hardtanh_patterns`, `binary_ops`, `inplace_fusion_op`, `outplace_fusion_op`, `conv_call`, `conv_op`, and `...+3`.
- **CN**: 这里定义了函数`_register_inplace_fusion`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。初始化或更新了 `hardtanh_patterns`、`binary_ops`、`inplace_fusion_op`、`outplace_fusion_op`、`conv_call`、`conv_op`、`另有3项` 等值。

### Lines 953-980 / 第 953-980 行
````python
                unary_attr=UnaryAttr("relu"),
            )
            _register_binary_unary_maybe_inplace_fusion_lowering(
                binary_v1,
                conv_op,
                binary_op,
                inplace_fusion_op,
                outplace_fusion_op,
                other_index=0,
            )
            binary_v2 = _binary_fusion_v2(conv_call, binary_op)
            binary_unary_v2 = _combined_fusion(binary_v2, aten.relu)
            _register_binary_unary_maybe_inplace_fusion_lowering(
                binary_unary_v2,
                conv_op,
                binary_op,
                inplace_fusion_op,
                outplace_fusion_op,
                other_index=1,
                unary_attr=UnaryAttr("relu"),
            )
            _register_binary_unary_maybe_inplace_fusion_lowering(
                binary_v2,
                conv_op,
                binary_op,
                inplace_fusion_op,
                outplace_fusion_op,
                other_index=1,
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `unary_attr`, `other_index`, `binary_v2`, and `binary_unary_v2`. This range continues the implementation of function `_register_inplace_fusion`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `unary_attr`、`other_index`、`binary_v2`、`binary_unary_v2` 等值。这一段延续了函数`_register_inplace_fusion` 的具体实现。

### Lines 981-1008 / 第 981-1008 行
````python
            )

    def _register_binary_fusion():
        binary_ops = [aten.add, ops.add, aten.sub, ops.sub]
        fusion_ops = [
            mkldnn._convolution_pointwise.binary,
            mkldnn._linear_pointwise.binary,
        ]
        _computation_user_1 = [_conv_call(users=1), _linear_call(users=1)]
        for computation_call, computation_op, fusion_op in zip(
            _computation_user_1, computation_ops[:-1], fusion_ops
        ):
            for binary_op in binary_ops:
                pattern = _binary_fusion_v2(computation_call, binary_op)
                _register_binary_unary_fusion_lowering(
                    pattern, computation_op, binary_op, fusion_op
                )

            for binary_op in [aten.add, ops.add]:
                pattern = _binary_fusion_v1(computation_call, binary_op)
                _register_binary_unary_fusion_lowering(
                    pattern, computation_op, binary_op, fusion_op
                )

    def _register_binary_unary_fusion():
        binary_ops = [aten.add, ops.add, aten.sub, ops.sub]
        fusion_ops = [mkldnn._convolution_pointwise.binary]
        _computation_user_1 = [_conv_call(users=1)]
````
- **EN**: Introduces function `_register_binary_fusion`, function `_register_binary_unary_fusion`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `binary_ops`, `fusion_ops`, `_computation_user_1`, and `pattern`.
- **CN**: 这里定义了函数`_register_binary_fusion`、函数`_register_binary_unary_fusion`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。初始化或更新了 `binary_ops`、`fusion_ops`、`_computation_user_1`、`pattern` 等值。

### Lines 1009-1036 / 第 1009-1036 行
````python
        for computation_call, computation_op, fusion_op in zip(
            _computation_user_1, computation_ops[:-1], fusion_ops
        ):
            for binary_op in binary_ops:
                pattern_v1 = _combined_fusion(
                    _binary_fusion_v2(computation_call, binary_op), aten.relu
                )
                _register_binary_unary_fusion_lowering(
                    pattern_v1,
                    computation_op,
                    binary_op,
                    fusion_op,
                    unary_attr=UnaryAttr("relu"),
                )
            for binary_op in [aten.add, ops.add]:
                pattern_v2 = _combined_fusion(
                    _binary_fusion_v1(computation_call, binary_op), aten.relu
                )
                _register_binary_unary_fusion_lowering(
                    pattern_v2,
                    computation_op,
                    binary_op,
                    fusion_op,
                    unary_attr=UnaryAttr("relu"),
                )

    def _recover_linear():
        # convert reshape+linear+reshape to a single linear for applying fusion path.
````
- **EN**: Introduces function `_recover_linear`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `pattern_v1`, `unary_attr`, and `pattern_v2`.
- **CN**: 这里定义了函数`_recover_linear`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。初始化或更新了 `pattern_v1`、`unary_attr`、`pattern_v2` 等值。

### Lines 1037-1064 / 第 1037-1064 行
````python
        # concat_linear (pass_number=0) -> mkldnn_linear_pack (pass_number=1) -> _recover_linear(pass_number=2)
        @register_freezing_graph_pattern(
            CallFunction(
                aten.reshape.default,
                CallFunction(
                    mkldnn._linear_pointwise.default,
                    CallFunction(
                        aten.reshape.default,
                        Arg(),
                        KeywordArg("reshape_1"),
                        _users=MULTIPLE,
                    ),
                    Arg(),
                    Arg(),
                    Arg(),
                    Arg(),
                    Arg(),
                ),
                KeywordArg("reshape_2"),
            ),
            pass_number=2,
        )
        def reshape_linear_reshape_pattern(match, *args, **kwargs):
            def get_val(val):
                return val if isinstance(val, int) else val.meta.get("val")

            reshape_1 = kwargs.get("reshape_1")
            reshape_2 = kwargs.get("reshape_2")
````
- **EN**: Introduces function `reshape_linear_reshape_pattern`, function `get_val`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`reshape_linear_reshape_pattern`、函数`get_val`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1065-1092 / 第 1065-1092 行
````python
            assert isinstance(reshape_1, list)
            assert isinstance(reshape_2, list)
            assert len(reshape_1) == 2

            graph = match.graph
            reshape_2_node = match.output_node()
            linear_input_node = reshape_2_node.args[0].args[0].args[0]
            # check linear's input's shape[:-1] == reshape_2[:-1]
            # and check product(reshape_2[:-1]) == reshape_1[0]
            can_remove_reshape = linear_input_node.meta.get("val").shape[
                :-1
            ] == torch.Size([get_val(val) for val in reshape_2[:-1]])
            can_remove_reshape = can_remove_reshape and (
                reduce(
                    operator.mul,
                    [get_val(val) for val in reshape_2[:-1]],
                )
                == get_val(reshape_1[0])
            )

            if can_remove_reshape:
                repl = graph.call_function(mkldnn._linear_pointwise.default, args)
                repl.meta.update(reshape_2_node.meta)
                reshape_2_node.replace_all_uses_with(repl)
                old_linear_node = reshape_2_node.args[0]
                reshape_1_node = old_linear_node.args[0]
                graph.erase_node(reshape_2_node)
                graph.erase_node(old_linear_node)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `graph`, `reshape_2_node`, `linear_input_node`, `can_remove_reshape`, `repl`, `old_linear_node`, and `...+1`. This range continues the implementation of function `_recover_linear.reshape_linear_reshape_pattern`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `graph`、`reshape_2_node`、`linear_input_node`、`can_remove_reshape`、`repl`、`old_linear_node`、`另有1项` 等值。这一段延续了函数`_recover_linear.reshape_linear_reshape_pattern` 的具体实现。

### Lines 1093-1120 / 第 1093-1120 行
````python
                if len(reshape_1_node.users) == 0:
                    graph.erase_node(reshape_1_node)
            counters["inductor"]["mkldnn_reshape_linear_reshape_matcher_count"] += 1
            counters["inductor"]["mkldnn_reshape_linear_reshape_matcher_nodes"] += len(
                match.nodes
            )

        def is_linear_add_bias(match):
            add_node = match.output_node()
            linear_node = add_node.args[0]
            device_type = add_node.meta.get("val").device.type
            mkldnn_device_op = _get_mkldnn_device_op(device_type)
            transpose_weight_node = mkldnn_device_op.get_linear_transpose_weight(
                linear_node.args[1]
            )
            weight_meta = transpose_weight_node.args[0].meta.get("val")
            bias_node = add_node.args[1]
            if isinstance(bias_node, int):
                # we only folding bias if it is a constant
                return False
            bias_meta = add_node.args[1].meta.get("val")
            if weight_meta is None or bias_meta is None:
                return False

            if bias_meta.dtype != weight_meta.dtype:
                return False
            return (
                linear_node.args[2] is None
````
- **EN**: Introduces function `is_linear_add_bias`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`is_linear_add_bias`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1121-1148 / 第 1121-1148 行
````python
                and bias_meta.dim() == 1
                and bias_meta.size(0) == weight_meta.size(1)
            )

        # convert linear+bias to a single linear for applying fusion path.
        @register_freezing_graph_pattern(
            CallFunction(
                aten.add.Tensor,
                CallFunction(mkldnn._linear_pointwise.default, *_linear_args),
                Arg(),
            ),
            pass_number=2,
            extra_check=is_linear_add_bias,
        )
        def linear_bias_pattern(match, *args):
            graph = match.graph
            add_node = match.output_node()
            linear_node = add_node.args[0]
            new_args = list(linear_node.args)
            new_args[2] = add_node.args[1]
            repl = graph.call_function(
                mkldnn._linear_pointwise.default, tuple(new_args)
            )
            repl.meta.update(add_node.meta)
            add_node.replace_all_uses_with(repl)
            match.erase_nodes()
            counters["inductor"]["mkldnn_linear_bias_matcher_count"] += 1
            counters["inductor"]["mkldnn_linear_bias_matcher_nodes"] += len(match.nodes)
````
- **EN**: Introduces function `linear_bias_pattern`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `pass_number`, `extra_check`, `graph`, `add_node`, `linear_node`, `new_args`, and `...+1`.
- **CN**: 这里定义了函数`linear_bias_pattern`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `pass_number`、`extra_check`、`graph`、`add_node`、`linear_node`、`new_args`、`另有1项` 等值。

### Lines 1149-1176 / 第 1149-1176 行
````python

    def _is_packable_mkldnn_rnn_layer(match):
        lstm_node = match.output_node()
        POS_WEIGHTS = [1, 2]
        POS_INPUTS = [0, 5, 6]
        POS_ARGS = POS_WEIGHTS + POS_INPUTS
        # Weights should be Constant
        if any(
            lstm_node.args[POS_WEIGHT].op != "get_attr" for POS_WEIGHT in POS_WEIGHTS
        ):
            return False

        # Meta info for weights and inputs should be available
        if any(lstm_node.args[POS_ARG].meta.get("val") is None for POS_ARG in POS_ARGS):
            return False

        # Check device
        if any(
            lstm_node.args[POS_ARG].meta.get("val").device.type != "cpu"
            for POS_ARG in POS_ARGS
        ):
            return False

        # Check dtype
        if any(
            lstm_node.args[POS_ARG].meta.get("val").dtype == torch.bfloat16
            and not is_mkldnn_bf16_supported("cpu")
            for POS_ARG in POS_ARGS
````
- **EN**: Introduces function `_is_packable_mkldnn_rnn_layer`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_is_packable_mkldnn_rnn_layer`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1177-1204 / 第 1177-1204 行
````python
        ):
            return False
        if any(
            lstm_node.args[POS_ARG].meta.get("val").dtype == torch.float16
            and not is_mkldnn_fp16_supported("cpu")
            for POS_ARG in POS_ARGS
        ):
            return False

        return True

    def _is_packable_convolution(match):
        """
        Check if the node is supported for MKLDNN convolution.
        """
        conv_node = match.output_node()
        device_type = conv_node.meta.get("val").device.type
        # The operator 'mkldnn::_convolution_transpose_pointwise' is not currently implemented for the XPU device.
        if match.kwargs["is_transposed"] and device_type == "xpu":
            return False

        input_meta_value = conv_node.args[0].meta.get("val")
        weight_meta_value = conv_node.args[1].meta.get("val")
        if input_meta_value is None or weight_meta_value is None:
            return False
        input_size = input_meta_value.shape
        if conv_node.args[1].op != "get_attr":
            return False
````
- **EN**: Introduces function `_is_packable_convolution`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_is_packable_convolution`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1205-1232 / 第 1205-1232 行
````python
        for meta_value in [input_meta_value, weight_meta_value]:
            if (
                meta_value is None
                or meta_value.device.type not in SUPPORTED_MKLDNN_DEVICES
                or (meta_value.dim() != 4 and meta_value.dim() != 5)
            ):
                return False

        if (
            input_meta_value.dtype == torch.bfloat16
            or weight_meta_value.dtype == torch.bfloat16
        ):
            if not is_mkldnn_bf16_supported(device_type):
                return False
        if (
            input_meta_value.dtype == torch.float16
            or weight_meta_value.dtype == torch.float16
        ):
            if not is_mkldnn_fp16_supported(device_type):
                return False
        is_transposed = conv_node.args[-3]
        if is_transposed:
            # TODO: Support dynamic shape case for MKLDNN conv transpose.
            if has_free_symbols(input_size):
                return False
            groups = conv_node.args[-1]
            in_channels = weight_meta_value.size(0)
            # doesn't support group_depthwise_conv_transpose.
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1233-1260 / 第 1233-1260 行
````python
            if groups > 1 and groups == in_channels:
                return False
            # Port from: aten/src/ATen/native/Convolution.cpp:is_output_padding_big
            output_paddings = conv_node.args[-2]
            strides = conv_node.args[3]
            if any(
                output_padding >= stride
                for output_padding, stride in zip(output_paddings, strides)
            ):
                return False
        return True

    def _is_packable_linear(match):
        """
        Check if the node is supported for MKLDNN linear.
        """

        def is_const_or_cat_by_const(weight):
            if weight.op == "get_attr":
                return True
            if weight.target != aten.cat.default:
                return False
            return all(arg.op == "get_attr" for arg in weight.args[0])

        linear_node = match.output_node()
        # mkldnn linear only supports beta=1or0 and alpha=1
        if linear_node.target is aten.addmm.default:
            alpha = linear_node.kwargs.get("alpha", 1.0)
````
- **EN**: Introduces function `_is_packable_linear`, function `is_const_or_cat_by_const`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `output_paddings`, `strides`, `linear_node`, and `alpha`.
- **CN**: 这里定义了函数`_is_packable_linear`、函数`is_const_or_cat_by_const`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `output_paddings`、`strides`、`linear_node`、`alpha` 等值。

### Lines 1261-1288 / 第 1261-1288 行
````python
            beta = linear_node.kwargs.get("beta", 1.0)
            if (beta != 0.0 and beta != 1.0) or alpha != 1.0:
                return False
        # weight_idx is 1 for aten.mm and is 2 for aten.addmm
        weight_idx = 2 if linear_node.target is aten.addmm.default else 1
        if not is_const_or_cat_by_const(linear_node.args[weight_idx]):
            return False
        input_meta_value = linear_node.args[weight_idx - 1].meta.get("val")
        weight_meta_value = linear_node.args[weight_idx].meta.get("val")
        if input_meta_value is None or weight_meta_value is None:
            return False
        if (
            input_meta_value.dtype == torch.float64
            or weight_meta_value.dtype == torch.float64
        ):
            return False
        is_lp_weight = weight_meta_value.dtype in (
            torch.bfloat16,
            torch.float16,
        )
        reduced_f32_matmul_enabled = torch.backends.mkldnn.matmul.fp32_precision in [  # type: ignore[attr-defined]
            "bf16",
            "tf32",
        ]
        use_reduced_f32_for_fp32_weight = (
            reduced_f32_matmul_enabled and weight_meta_value.dtype == torch.float32
        )
        compute_with_lp = is_lp_weight or use_reduced_f32_for_fp32_weight
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `beta`, `weight_idx`, `input_meta_value`, `weight_meta_value`, `is_lp_weight`, `reduced_f32_matmul_enabled`, and `...+2`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `beta`、`weight_idx`、`input_meta_value`、`weight_meta_value`、`is_lp_weight`、`reduced_f32_matmul_enabled`、`另有2项` 等值。

### Lines 1289-1316 / 第 1289-1316 行
````python
        # on x86, for fp32, mkl should be enabled.
        # on aarch64, use mkldnn op for fp32 as well if acl is enabled
        if (
            not compute_with_lp
            and not mkldnn._is_mkldnn_acl_supported()
            and not torch._C.has_mkl
        ):
            return False
        for meta_value in [input_meta_value, weight_meta_value]:
            if (
                meta_value is None
                or meta_value.device.type != "cpu"
                or meta_value.dim() != 2
            ):
                return False
        if weight_idx == 2:
            bias_meta_value = linear_node.args[0].meta.get("val")
            if (
                bias_meta_value is None
                or meta_value.device.type != "cpu"
                or bias_meta_value.dim() != 1
                or bias_meta_value.size(0) != weight_meta_value.size(1)
            ):
                return False

        device_type = input_meta_value.device.type
        if (
            input_meta_value.dtype == torch.bfloat16
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1317-1344 / 第 1317-1344 行
````python
            or weight_meta_value.dtype == torch.bfloat16
        ):
            if not is_mkldnn_bf16_supported(device_type):
                return False
        if (
            input_meta_value.dtype == torch.float16
            or weight_meta_value.dtype == torch.float16
        ):
            if not is_mkldnn_fp16_supported(device_type):
                return False
        return True

    _aten_conv_args = (
        Arg(),
        Arg(),
        Arg(),
        Arg(),
        Arg(),
        Arg(),
        KeywordArg("is_transposed"),
        Arg(),
        Arg(),
    )

    _aten_mkldnn_rnn_layer_args = (
        Arg(),  # input
        Arg(),  # weight0
        Arg(),  # weight1
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `_aten_conv_args`, and `_aten_mkldnn_rnn_layer_args`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `_aten_conv_args`、`_aten_mkldnn_rnn_layer_args` 等值。

### Lines 1345-1372 / 第 1345-1372 行
````python
        Arg(),  # weight2
        Arg(),  # weight3
        Arg(),  # hx_
        Arg(),  # cx_
        KeywordArg("reverse"),  # reverse
        Arg(),  # batch_sizes
        Arg(),  # mode
        Arg(),  # hidden_size
        Arg(),  # num_layers
        Arg(),  # has_biases
        Arg(),  # bidirectional
        Arg(),  # batch_first
        Arg(),  # train
    )

    def _register_weight_pack_pass():
        @register_freezing_graph_pattern(
            CallFunction(aten.convolution.default, *_aten_conv_args),
            extra_check=_is_packable_convolution,
        )
        def convolution(match, *args, **kwargs):
            is_transposed = kwargs.get("is_transposed")
            assert isinstance(is_transposed, bool)
            graph = match.graph
            conv_node = match.output_node()
            device_type = conv_node.args[0].meta.get("val").device.type
            mkldnn_device_op = _get_mkldnn_device_op(device_type)
            input_size = conv_node.args[0].meta.get("val").shape
````
- **EN**: Introduces function `_register_weight_pack_pass`, function `convolution`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`_register_weight_pack_pass`、函数`convolution`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 1373-1400 / 第 1373-1400 行
````python
            with graph.inserting_before(conv_node):
                constant_args = [args[4], args[3], args[5], args[-1]]
                packed_conv_op = mkldnn._convolution_pointwise.default
                if is_transposed:
                    constant_args.insert(1, args[-2])  # output_padding
                    packed_conv_op = mkldnn._convolution_transpose_pointwise.default

                if not has_free_symbols(input_size):
                    packed_weight_node = mkldnn_device_op.pack_conv_weight(
                        graph,
                        is_transposed,
                        args[1],
                        constant_args,
                        input_size,
                    )
                else:
                    assert not is_transposed
                    # For dynamic shape case, we need to pack weight in runtime.
                    packed_weight_node = args[1]

                packed_conv_inputs = (
                    (args[0], packed_weight_node, args[2])
                    + tuple(constant_args)
                    + ("none", [], "")
                )
                packed_conv_node = graph.create_node(
                    "call_function", packed_conv_op, tuple(packed_conv_inputs)
                )
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `constant_args`, `packed_conv_op`, `packed_weight_node`, `else`, `packed_conv_inputs`, and `packed_conv_node`. This range continues the implementation of function `_register_weight_pack_pass.convolution`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `constant_args`、`packed_conv_op`、`packed_weight_node`、`else`、`packed_conv_inputs`、`packed_conv_node` 等值。这一段延续了函数`_register_weight_pack_pass.convolution` 的具体实现。

### Lines 1401-1428 / 第 1401-1428 行
````python
                conv_node.replace_all_uses_with(packed_conv_node)
                packed_conv_node.meta.update(conv_node.meta)
                graph.erase_node(conv_node)
            counters["inductor"]["mkldnn_conv_weight_pack_matcher_count"] += 1
            counters["inductor"]["mkldnn_conv_weight_pack_matcher_nodes"] += len(
                match.nodes
            )

        @register_freezing_graph_pattern(
            CallFunction(aten.mkldnn_rnn_layer.default, *_aten_mkldnn_rnn_layer_args),
            extra_check=_is_packable_mkldnn_rnn_layer,
        )
        def mkldnn_rnn_layer(match, *args, **kwargs):
            def get_item(graph, node, index):
                return graph.call_function(operator.getitem, (node, index))

            graph = match.graph
            lstm_node = match.output_node()
            weight0, weight1 = args[1:3]
            reverse = kwargs.get("reverse")
            packed_lstm_op = aten.mkldnn_rnn_layer.default
            hidden_size = args[9]
            has_biases = args[11]
            batch_first = args[13]
            with graph.inserting_before(lstm_node):
                packed_weight_op = mkldnn._reorder_mkldnn_rnn_layer_weight.default
                packed_weight_inputs = (
                    weight0,
````
- **EN**: Introduces function `mkldnn_rnn_layer`, function `get_item`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`mkldnn_rnn_layer`、函数`get_item`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1429-1456 / 第 1429-1456 行
````python
                    weight1,
                    hidden_size,
                    reverse,
                    has_biases,
                    batch_first,
                )
                packed_weight_node = graph.create_node(
                    "call_function", packed_weight_op, packed_weight_inputs, {}, "name"
                )
                packed_weight_items = [
                    get_item(graph, packed_weight_node, i) for i in range(2)
                ]
                pack_lstm_inputs = (
                    args[0],
                    *packed_weight_items,
                    args[3],
                    args[4],
                    args[5],
                    args[6],
                    reverse,
                    *args[7:],
                )

                packed_lstm_node = graph.create_node(
                    "call_function", packed_lstm_op, args=pack_lstm_inputs
                )
                lstm_node.replace_all_uses_with(packed_lstm_node)
                packed_lstm_node.meta.update(lstm_node.meta)
````
- **EN**: Initializes or updates values such as `packed_weight_node`, `packed_weight_items`, `pack_lstm_inputs`, and `packed_lstm_node`. This range continues the implementation of function `_register_weight_pack_pass.mkldnn_rnn_layer`.
- **CN**: 初始化或更新了 `packed_weight_node`、`packed_weight_items`、`pack_lstm_inputs`、`packed_lstm_node` 等值。这一段延续了函数`_register_weight_pack_pass.mkldnn_rnn_layer` 的具体实现。

### Lines 1457-1484 / 第 1457-1484 行
````python
                graph.erase_node(lstm_node)
            counters["inductor"]["mkldnn_rnn_weight_pack_matcher_count"] += 1
            counters["inductor"]["mkldnn_rnn_weight_pack_matcher_nodes"] += len(
                match.nodes
            )

        @register_freezing_graph_pattern(
            CallFunction(
                aten.addmm.default,
                Arg(),
                Arg(),
                Arg(),
                beta=KeywordArg("beta"),
                alpha=KeywordArg("alpha"),
            ),
            extra_check=_is_packable_linear,
            pass_number=1,
        )
        @register_freezing_graph_pattern(
            CallFunction(aten.mm.default, Arg(), Arg()),
            extra_check=_is_packable_linear,
            pass_number=1,
        )
        def linear(match, *args, **kwargs):
            graph = match.graph
            linear_node = match.output_node()
            input = args[0] if linear_node.target is aten.mm.default else args[1]
            bias = (
````
- **EN**: Introduces function `linear`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `beta`, `alpha`, `extra_check`, `pass_number`, `graph`, `linear_node`, and `...+2`.
- **CN**: 这里定义了函数`linear`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `beta`、`alpha`、`extra_check`、`pass_number`、`graph`、`linear_node`、`另有2项` 等值。

### Lines 1485-1512 / 第 1485-1512 行
````python
                None
                if linear_node.target is aten.mm.default
                or (
                    linear_node.target is aten.addmm.default
                    and linear_node.kwargs.get("beta", 1.0) == 0.0
                )
                else args[0]
            )
            weight = args[1] if linear_node.target is aten.mm.default else args[2]
            device_type = input.meta.get("val").device.type
            mkldnn_device_op = _get_mkldnn_device_op(device_type)
            with graph.inserting_before(linear_node):
                transpose_weight_node = graph.create_node(
                    "call_function", aten.permute.default, (weight, (1, 0))
                )
                weight_dtype = weight.meta.get("val").dtype
                is_lp_weight = weight_dtype in (
                    torch.bfloat16,
                    torch.float16,
                )
                reduced_f32_matmul_enabled = (
                    torch.backends.mkldnn.matmul.fp32_precision in ["bf16", "tf32"]  # type: ignore[attr-defined]
                )
                use_reduced_f32_for_fp32_weight = (
                    reduced_f32_matmul_enabled and weight_dtype == torch.float32
                )
                compute_with_lp = is_lp_weight or use_reduced_f32_for_fp32_weight
                batch_size = input.meta.get("val").shape[0]
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `weight`, `device_type`, `mkldnn_device_op`, `transpose_weight_node`, `weight_dtype`, `is_lp_weight`, and `...+4`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `weight`、`device_type`、`mkldnn_device_op`、`transpose_weight_node`、`weight_dtype`、`is_lp_weight`、`另有4项` 等值。

### Lines 1513-1540 / 第 1513-1540 行
````python
                packed_weight_node = mkldnn_device_op.pack_linear_weight(
                    graph, compute_with_lp, transpose_weight_node, batch_size
                )
                packed_linear_node = mkldnn_device_op.pack_linear(
                    graph, compute_with_lp, batch_size, input, packed_weight_node, bias
                )

                linear_node.replace_all_uses_with(packed_linear_node)
                packed_linear_node.meta.update(linear_node.meta)
                graph.erase_node(linear_node)
            counters["inductor"]["mkldnn_linear_weight_pack_matcher_count"] += 1
            counters["inductor"]["mkldnn_linear_weight_pack_matcher_nodes"] += len(
                match.nodes
            )

    def _eliminate_duplicate_packed_nodes(gm):
        """
        Combine packed weight nodes with the same inputs to reduce memory usage.
        for example:
        class Model(nn.Module):
            def __init__(self) -> None:
                super().__init__()
                self.linear = nn.Linear(32, 32, bias=True)

            def forward(self, x):
                return self.linear(self.linear(x))

        the above's packed weight nodes are duplicate if two linear calls have same input size.
````
- **EN**: Introduces function `_eliminate_duplicate_packed_nodes`, class `Model`, function `__init__`, function `forward`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `packed_weight_node`, and `packed_linear_node`.
- **CN**: 这里定义了函数`_eliminate_duplicate_packed_nodes`、类`Model`、函数`__init__`、函数`forward`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `packed_weight_node`、`packed_linear_node` 等值。

### Lines 1541-1568 / 第 1541-1568 行
````python
        """
        if not (torch.backends.mkldnn.enabled and torch.backends.mkldnn.is_available()):
            return gm

        packed_weight_ops = [
            torch._C._nn.mkldnn_reorder_conv2d_weight,
            torch._C._nn.mkldnn_reorder_conv3d_weight,
            mkldnn._reorder_convolution_transpose_weight,
            mkldnn._reorder_linear_weight,
            mkldnn._reorder_mkldnn_rnn_layer_weight,
        ]
        if torch._C.has_mkl:
            packed_weight_ops.append(torch.ops.mkl._mkl_reorder_linear_weight)

        for node in gm.graph.nodes:
            if node.target in packed_weight_ops and len(node.args[0].users) > 1:
                for user_node in list(node.args[0].users.keys()):
                    if (
                        user_node.target == node.target
                        and user_node != node
                        and user_node.args == node.args
                    ):
                        user_node.replace_all_uses_with(node)
                        gm.graph.erase_node(user_node)

    @functools.cache
    def _mkldnn_fusion_init():
        # TODO: aarch64: enable op fusion for acl once it supports fused operators. Disabling it for now.
````
- **EN**: Introduces function `_mkldnn_fusion_init`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_mkldnn_fusion_init`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1569-1591 / 第 1569-1591 行
````python
        # Otherwise even the matmul or innerproduct can not be accelerated with acl
        if (
            not torch.backends.mkldnn.enabled
            or not torch.backends.mkldnn.is_available()
        ):
            return

        if not torch.ops.mkldnn._is_mkldnn_acl_supported():
            _register_unary_fusion()
            _register_inplace_fusion()
            _register_binary_unary_fusion()
            _register_binary_fusion()
            _register_quantization_lowerings()

        _register_woq_lowerings()

    @functools.cache
    def _mkldnn_weight_pack_init():
        if torch.backends.mkldnn.enabled and torch.backends.mkldnn.is_available():
            _register_weight_pack_pass()
            _recover_linear()
            _register_quantization_weight_pack_pass()
            _register_int8_woq_concat_linear_pattern()
````
- **EN**: Introduces function `_mkldnn_weight_pack_init`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`_mkldnn_weight_pack_init`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`, `operator`, `typing`, `collections.abc`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._dynamo.utils`, `torch.fx.experimental.symbolic_shapes`, `torch.utils._ordered_set`, `..`, `..lowering`, `..pattern_matcher`, `..utils`, `..virtualized`, `.freezing_patterns`, `.post_grad`, `.quantization`, `..mkldnn_lowerings`
