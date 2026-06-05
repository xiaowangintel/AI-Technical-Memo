# mkldnn_ir.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/mkldnn_ir.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `ConvolutionUnary`, `ConvolutionBinary`, `ConvolutionBinaryInplace`, `ConvolutionTransposeUnary`, `QConvPointWisePT2E`, `QConvPointWiseBinaryPT2E`, and `...+7`. It exposes functions such as `_prepare_convolution_fusion_create`, `_prepare_linear_fusion_create`, and `_create_output_node`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `ConvolutionUnary`、`ConvolutionBinary`、`ConvolutionBinaryInplace`、`ConvolutionTransposeUnary`、`QConvPointWisePT2E`、`QConvPointWiseBinaryPT2E`、`另有7项` 等类。同时提供 `_prepare_convolution_fusion_create`、`_prepare_linear_fusion_create`、`_create_output_node` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
# mypy: allow-untyped-defs
from collections.abc import Sequence
from typing import Any, Optional

import sympy

import torch
from torch._prims_common import make_channels_last_strides_for, StrideType
from torch.utils._ordered_set import OrderedSet

from .ir import (
    ExternKernelAlloc,
    FixedLayout,
    FlexibleLayout,
    get_device_type,
    ir_node_to_tensor,
    IRNode,
    is_contiguous_storage_and_layout,
    Layout,
    may_convert_to_optional,
    MultiOutput,
    MultiOutputLayout,
    MutationOutput,
    NoneLayout,
    TensorBox,
)
from .utils import convert_shape_to_inductor, pad_listlike, SUPPORTED_MKLDNN_DEVICES
from .virtualized import V
````
- **EN**: Imports dependencies such as `collections.abc`, `typing`, `sympy`, `torch`, `torch._prims_common`, `torch.utils._ordered_set`, and `...+3` for the logic in this range. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里导入了 `collections.abc`、`typing`、`sympy`、`torch`、`torch._prims_common`、`torch.utils._ordered_set`、`另有3项` 等依赖，为后续逻辑提供基础能力。借助符号表达式来推理索引、形状或代数性质。

### Lines 29-56 / 第 29-56 行
````python


def _prepare_convolution_fusion_create(
    cls,
    x: "TensorBox",
    weight: "TensorBox",
    bias: "TensorBox",
    padding: Sequence[int],
    stride: Sequence[int],
    dilation: Sequence[int],
    groups: int,
    transposed: bool = False,
    output_padding: Sequence[int] | None = None,
    quantize_args: list["TensorBox"] | None = None,
    other: Optional["TensorBox"] = None,
):
    """
    This function is a helper function to prepare inputs, layout and constant args
    for convolution post-op fusion's create function, including deciding the output
    layout (channels first or channels last), realizing inputs and make them etc. The
    function only supports the CPU/XPU device since conv post-op fusion kernel is only
    supported on CPU/XPU right now.
    """

    # Port from aten/src/ATen/native/ConvUtils.h: _conv_input_size
    def _conv_input_size(
        output_size, weight_size, padding, output_padding, stride, dilation, groups
    ):
````
- **EN**: Introduces function `_prepare_convolution_fusion_create`, function `_conv_input_size`. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `x`, `weight`, `bias`, `padding`, `stride`, `dilation`, and `...+5`.
- **CN**: 这里定义了函数`_prepare_convolution_fusion_create`、函数`_conv_input_size`。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `x`、`weight`、`bias`、`padding`、`stride`、`dilation`、`另有5项` 等值。

### Lines 57-84 / 第 57-84 行
````python
        assert len(output_size) == len(weight_size), "Expect input dim == weight dim"
        dim = len(output_size)
        assert dim > 2, "Expect input dim > 2"

        BATCH_DIM = 0
        WEIGHT_INPUT_CHANNELS_DIM = 1
        input_size = []
        input_size.append(output_size[BATCH_DIM])
        input_size.append(weight_size[WEIGHT_INPUT_CHANNELS_DIM] * groups)
        for d in range(2, dim):
            kernel = (weight_size[d] - 1) * dilation[d - 2] + 1
            input_size_d = (
                (output_size[d] - 1) * stride[d - 2]
                - (padding[d - 2] * 2)
                + kernel
                + output_padding[d - 2]
            )
            input_size.append(input_size_d)
        return list(map(int, input_size))

    # Port from aten/src/ATen/native/ConvUtils.h: _conv_output_size
    def _conv_output_size(input_size, weight_size, padding, stride, dilation=None):
        has_dilation = dilation is not None
        dim = len(input_size)
        output_size = []
        output_size.append(input_size[0])
        output_size.append(weight_size[0])
        for d in range(2, dim):
````
- **EN**: Introduces function `_conv_output_size`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dim`, `BATCH_DIM`, `WEIGHT_INPUT_CHANNELS_DIM`, `input_size`, `kernel`, `input_size_d`, and `...+2`.
- **CN**: 这里定义了函数`_conv_output_size`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `dim`、`BATCH_DIM`、`WEIGHT_INPUT_CHANNELS_DIM`、`input_size`、`kernel`、`input_size_d`、`另有2项` 等值。

### Lines 85-112 / 第 85-112 行
````python
            # pyrefly: ignore [unsupported-operation]
            dilation_ = dilation[d - 2] if has_dilation else 1
            kernel = dilation_ * (weight_size[d] - 1) + 1
            output_size_d = (input_size[d] + (2 * padding[d - 2]) - kernel) // stride[
                d - 2
            ] + 1
            output_size.append(output_size_d)
        return output_size

    # The size of prepacked_weight is the prepacked weight size of deconv:
    #   Groups > 1:  [g*o, i/g, ...]
    #   Groups == 1: [o, i, ...]
    # Returns original weight size in [i, o, ...]
    def _original_deconv_weight_size(
        prepacked_weight,
        groups,
    ):
        prepacked_weight_size = prepacked_weight.size()
        dim = len(prepacked_weight_size)
        assert dim > 2, "Expect weight dim > 2"
        if groups > 1:
            weight_size = []
            weight_size.append(prepacked_weight_size[1] * groups)
            weight_size.append(prepacked_weight_size[0] / groups)
            weight_size.extend(prepacked_weight_size[d] for d in range(2, dim))
        else:
            weight_size = prepacked_weight.transpose(0, 1).size()
        return weight_size
````
- **EN**: Introduces function `_original_deconv_weight_size`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dilation_`, `kernel`, `output_size_d`, `prepacked_weight_size`, `dim`, `weight_size`, and `...+1`.
- **CN**: 这里定义了函数`_original_deconv_weight_size`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `dilation_`、`kernel`、`output_size_d`、`prepacked_weight_size`、`dim`、`weight_size`、`另有1项` 等值。

### Lines 113-140 / 第 113-140 行
````python

    x.realize()
    weight.realize()
    if bias is not None:
        bias.realize()
    with V.graph.fake_mode:
        # TODO <Leslie> cleaned up the fake_tensor trace as Linear implementation
        x_fake = ir_node_to_tensor(x)
        weight_fake = ir_node_to_tensor(weight)
        dims = len(x_fake.size()) - 2
        assert 0 < len(padding) <= dims
        assert 0 < len(dilation) <= dims
        assert 0 < len(stride) <= dims
        padding = pad_listlike(padding, dims)
        dilation = pad_listlike(dilation, dims)
        stride = pad_listlike(stride, dims)
        if output_padding is None:
            output_padding = pad_listlike([0], dims)
        else:
            assert 0 < len(output_padding) <= dims
            output_padding = pad_listlike(output_padding, dims)
        assert isinstance(groups, (int, sympy.core.numbers.Integer))
        if transposed:
            # When transposed, the size of the prepacked oneDNN weight is different
            # from the PyTorch weight. We're not able to run aten conv with such
            # size. We infer the output size from the input params here:
            weight_size = _original_deconv_weight_size(weight_fake, groups)
            input_size = x_fake.size()
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `x_fake`, `weight_fake`, `dims`, `padding`, `dilation`, `stride`, and `...+4`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `x_fake`、`weight_fake`、`dims`、`padding`、`dilation`、`stride`、`另有4项` 等值。

### Lines 141-168 / 第 141-168 行
````python
            output_size = _conv_input_size(
                input_size,
                weight_size,
                padding,
                output_padding,
                stride,
                dilation,
                groups,
            )
        else:
            x_shape = list(x_fake.shape)
            weight_shape = list(weight_fake.shape)
            if len(x_shape) != len(weight_shape):
                assert len(x_shape) == 3 and len(weight_shape) == 4
                weight_shape.pop(2)
            output_size = _conv_output_size(
                x_shape,
                weight_shape,
                padding,
                stride,
                dilation,
            )

        req_stride_order = [0] + list(reversed(range(1, len(stride) + 1)))
        req_stride_order = [len(req_stride_order)] + req_stride_order

    x = cls.require_stride_order(x, req_stride_order)

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `output_size`, `else`, `x_shape`, `weight_shape`, `req_stride_order`, and `x`. This range continues the implementation of function `_prepare_convolution_fusion_create`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `output_size`、`else`、`x_shape`、`weight_shape`、`req_stride_order`、`x` 等值。这一段延续了函数`_prepare_convolution_fusion_create` 的具体实现。

### Lines 169-196 / 第 169-196 行
````python
    # We won't do weight prepack for Conv if dynamic_shapes or if is xpu.
    # In static shape cases, since weight is prepacked, we'll always force output to be channels last in the Conv kernel.
    # In dynamic shape cases, for input with channels = 1, like tensor of size (s0, 1, 28, 28) and stride (784, 784, 28, 1),
    # x = cls.require_stride_order(x, req_stride_order) where req_stride_order is in the channels last order
    # won't change the stride of this tensor since stride for dimensions of size 1 is ignored. While in Conv kernel,
    # this tensor is considered as channels first and the output will be in contiguous format.
    # To align the behavior of the Conv kernel, we set the output_stride in such case to be contiguous instead of channels last.
    dynamic_shapes = not all(isinstance(i, int) for i in (output_size))
    if (
        dynamic_shapes or get_device_type(x) == "xpu"
    ) and is_contiguous_storage_and_layout(x):
        output_stride: StrideType = FlexibleLayout.contiguous_strides(output_size)
    # Currently we don't support channel last for the situation that stride of input's batch dim is 0,
    # eg. input_size = (1, 1280, 64, 64), but input_stride=(0, 1, 81920, 1280).
    # So we use NCHW hear instead.
    # Different with cpu, cpu conv always use channels_last for convolution when weight is prepacked,
    # but xpu does not do the prepack, so the problem exposed here is only for xpu.
    # TODO support channels_last for such zero stride input.
    elif get_device_type(x) == "xpu" and x.get_stride()[0] == 0:
        output_stride = FlexibleLayout.contiguous_strides(output_size)
    else:
        output_stride = make_channels_last_strides_for(output_size)

    assert get_device_type(x) == get_device_type(weight)
    assert get_device_type(x) in SUPPORTED_MKLDNN_DEVICES
    inputs = [x]

    if quantize_args is not None:
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dynamic_shapes`, `output_stride`, `else`, and `inputs`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `dynamic_shapes`、`output_stride`、`else`、`inputs` 等值。

### Lines 197-224 / 第 197-224 行
````python
        x_scale, x_zero_point, w_scale, w_zero_point = quantize_args
        x_scale.realize()
        x_zero_point.realize()
        w_scale.realize()
        w_zero_point.realize()
        inputs = inputs + [x_scale, x_zero_point] + [weight] + [w_scale, w_zero_point]
    else:
        inputs += [weight]

    if other is not None:
        other = cls.require_stride_order(other, req_stride_order)
        assert isinstance(other, TensorBox)
        inputs += [other]

    kernel_layout = FixedLayout(
        x.get_device_or_error(),
        x.get_dtype(),
        convert_shape_to_inductor(output_size),
        convert_shape_to_inductor(output_stride),
    )
    constant_args = [padding, stride, dilation, groups]
    if transposed:
        constant_args.insert(1, output_padding)

    if bias is not None:
        inputs.append(bias)
    else:
        constant_args.insert(0, bias)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `inputs`, `else`, `other`, `kernel_layout`, and `constant_args`. This range continues the implementation of function `_prepare_convolution_fusion_create`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `inputs`、`else`、`other`、`kernel_layout`、`constant_args` 等值。这一段延续了函数`_prepare_convolution_fusion_create` 的具体实现。

### Lines 225-252 / 第 225-252 行
````python
    return inputs, constant_args, kernel_layout, req_stride_order, other


def _prepare_linear_fusion_create(
    cls,
    x: "TensorBox",
    weight: "TensorBox",
    bias: "TensorBox",
    quantize_args: list["TensorBox"] | None = None,
    other: Optional["TensorBox"] = None,
    binary_sum: bool = False,
):
    """
    This function is a helper function to prepare inputs, layout and constant args
    for linear post-op fusion's create function. The function only supports the CPU device
    since linear post-op fusion kernel is only supported on CPU right now.
    """
    x.realize()
    weight.realize()
    if bias is not None:
        bias.realize()

    *m, _ = x.get_size()
    # The weight has been transposed during the qlinear weight prepack process.
    # https://github.com/pytorch/pytorch/blob/4979f9c0d72490970e2019bb1d2284f83d93f76b/
    # aten/src/ATen/native/quantized/cpu/qlinear_prepack.cpp#L291
    _, oc = weight.get_size()
    output_size = list(m) + [oc]
````
- **EN**: Introduces function `_prepare_linear_fusion_create`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_prepare_linear_fusion_create`。保留了指向设计说明、规范或厂商数据手册的注释引用。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 253-280 / 第 253-280 行
````python
    req_stride_order = list(reversed(range(len(x.get_size()))))

    x = cls.require_stride_order(x, req_stride_order)
    assert get_device_type(x) == get_device_type(weight)
    assert get_device_type(x) in SUPPORTED_MKLDNN_DEVICES
    inputs = [x]

    if quantize_args is not None:
        x_scale, x_zero_point, w_scale, w_zero_point = quantize_args
        x_scale.realize()
        x_zero_point.realize()
        w_scale.realize()
        w_zero_point.realize()
        inputs = inputs + [x_scale, x_zero_point] + [weight] + [w_scale, w_zero_point]
    else:
        inputs += [weight]

    if other is not None:
        if binary_sum:
            other = cls.require_stride_order(other, req_stride_order)
        inputs = inputs + [other]

    output_stride = FlexibleLayout.contiguous_strides(output_size)
    kernel_layout = FixedLayout(
        x.get_device(),
        x.get_dtype(),
        output_size,
        output_stride,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `req_stride_order`, `x`, `inputs`, `else`, `other`, `output_stride`, and `...+1`. This range continues the implementation of function `_prepare_linear_fusion_create`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `req_stride_order`、`x`、`inputs`、`else`、`other`、`output_stride`、`另有1项` 等值。这一段延续了函数`_prepare_linear_fusion_create` 的具体实现。

### Lines 281-308 / 第 281-308 行
````python
    )
    constant_args: list[Any] = []

    if bias is not None:
        inputs.append(bias)
    else:
        constant_args.insert(0, bias)
    return inputs, constant_args, kernel_layout, req_stride_order, other


def _create_output_node(packed):
    output_ir = MultiOutput(
        packed.get_layout(),
        packed,
        [],
    )
    packed.layout = MultiOutputLayout(device=packed.get_device())
    packed.outputs = [output_ir]
    return output_ir


class ConvolutionUnary(ExternKernelAlloc):
    def __init__(
        self,
        layout,
        inputs,
        constant_args=(),
    ) -> None:
````
- **EN**: Introduces function `_create_output_node`, class `ConvolutionUnary`, function `__init__`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_create_output_node`、类`ConvolutionUnary`、函数`__init__`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 309-336 / 第 309-336 行
````python
        self.device_type = get_device_type(inputs[0])
        super().__init__(
            layout,
            inputs,
            constant_args,
            None,
            op_overload=torch.ops.mkldnn._convolution_pointwise.default,
            cpp_kernel_name=f"aoti_torch_{self.device_type}_mkldnn__convolution_pointwise",
        )

    def codegen(self, wrapper):
        wrapper.include_extra_header(
            f"torch/csrc/inductor/aoti_torch/c/shim_{self.device_type}.h"
        )
        super().codegen(wrapper)

    @classmethod
    def create(
        cls,
        x: "TensorBox",
        weight: "TensorBox",
        bias: "TensorBox",
        padding_: list[int],
        stride_: list[int],
        dilation_: list[int],
        groups: int,
        attr,
        scalars: list[Any] | None,
````
- **EN**: Introduces function `codegen`, function `create`. Applies decorators to register behavior or alter how the following definition is constructed. Initializes or updates values such as `op_overload`, `cpp_kernel_name`, `x`, `weight`, `bias`, `padding_`, and `...+4`.
- **CN**: 这里定义了函数`codegen`、函数`create`。使用装饰器来注册行为，或改变后续定义的构造方式。初始化或更新了 `op_overload`、`cpp_kernel_name`、`x`、`weight`、`bias`、`padding_`、`另有4项` 等值。

### Lines 337-364 / 第 337-364 行
````python
        algorithm,
    ):
        (
            inputs,
            constant_args,
            kernel_layout,
            _,
            _,
        ) = _prepare_convolution_fusion_create(
            cls, x, weight, bias, padding_, stride_, dilation_, groups
        )
        constant_args = constant_args + [
            attr,
            may_convert_to_optional(scalars),
            algorithm,
        ]
        packed = ConvolutionUnary(
            layout=kernel_layout,
            inputs=inputs,
            constant_args=constant_args,
        )
        return _create_output_node(packed)


class ConvolutionBinary(ExternKernelAlloc):
    def __init__(
        self,
        layout,
````
- **EN**: Introduces class `ConvolutionBinary`, function `__init__`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `constant_args`, `packed`, `layout`, and `inputs`.
- **CN**: 这里定义了类`ConvolutionBinary`、函数`__init__`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `constant_args`、`packed`、`layout`、`inputs` 等值。

### Lines 365-392 / 第 365-392 行
````python
        inputs,
        constant_args=(),
        cpp_constant_args=(),
    ) -> None:
        self.device_type = get_device_type(inputs[0])
        super().__init__(
            layout,
            inputs,
            constant_args,
            None,
            op_overload=torch.ops.mkldnn._convolution_pointwise.binary,
            cpp_kernel_name=f"aoti_torch_{self.device_type}_mkldnn__convolution_pointwise_binary",
        )
        self.cpp_constant_args = cpp_constant_args

    def codegen(self, wrapper):
        wrapper.include_extra_header(
            f"torch/csrc/inductor/aoti_torch/c/shim_{self.device_type}.h"
        )
        super().codegen(wrapper)

    @classmethod
    def create(
        cls,
        x: "TensorBox",
        other: "TensorBox",
        weight: "TensorBox",
        bias: "TensorBox",
````
- **EN**: Introduces function `codegen`, function `create`. Applies decorators to register behavior or alter how the following definition is constructed. Initializes or updates values such as `constant_args`, `cpp_constant_args`, `op_overload`, `cpp_kernel_name`, `x`, `other`, and `...+2`.
- **CN**: 这里定义了函数`codegen`、函数`create`。使用装饰器来注册行为，或改变后续定义的构造方式。初始化或更新了 `constant_args`、`cpp_constant_args`、`op_overload`、`cpp_kernel_name`、`x`、`other`、`另有2项` 等值。

### Lines 393-420 / 第 393-420 行
````python
        padding_: list[int],
        stride_: list[int],
        dilation_: list[int],
        groups: int,
        binary_attr: str,
        binary_alpha: float | None,
        unary_attr: str | None,
        unary_scalars: list[Any] | None,
        unary_algorithm: str | None,
    ):
        (
            inputs,
            constant_args,
            kernel_layout,
            req_stride_order,
            _,
        ) = _prepare_convolution_fusion_create(
            cls, x, weight, bias, padding_, stride_, dilation_, groups
        )
        # pyrefly: ignore [bad-assignment]
        other = cls.require_stride_order(other, req_stride_order)
        inputs.insert(1, other)
        constant_args = constant_args + [
            binary_attr,
            binary_alpha,
            unary_attr,
            may_convert_to_optional(unary_scalars),
            unary_algorithm,
````
- **EN**: Initializes or updates values such as `padding_`, `stride_`, `dilation_`, `groups`, `binary_attr`, `binary_alpha`, and `...+5`. This range continues the implementation of function `ConvolutionBinary.create`.
- **CN**: 初始化或更新了 `padding_`、`stride_`、`dilation_`、`groups`、`binary_attr`、`binary_alpha`、`另有5项` 等值。这一段延续了函数`ConvolutionBinary.create` 的具体实现。

### Lines 421-448 / 第 421-448 行
````python
        ]
        packed = ConvolutionBinary(
            layout=kernel_layout,
            inputs=inputs,
            constant_args=constant_args,
        )
        return _create_output_node(packed)


class ConvolutionBinaryInplace(ExternKernelAlloc):
    def __init__(
        self,
        kernel_layout,
        inputs,
        constant_args=(),
    ) -> None:
        # Due to constrain of op.call, other (Tensor&) should be at input[0]
        self.device_type = get_device_type(inputs[0])
        reordered_inputs = [inputs[1], inputs[0]] + inputs[2:]

        super().__init__(
            kernel_layout,
            reordered_inputs,
            constant_args,
            None,
            op_overload=torch.ops.mkldnn._convolution_pointwise_.binary,
            cpp_kernel_name=f"aoti_torch_{self.device_type}_mkldnn__convolution_pointwise_binary_",
        )
````
- **EN**: Introduces class `ConvolutionBinaryInplace`, function `__init__`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `packed`, `layout`, `inputs`, `constant_args`, `reordered_inputs`, `op_overload`, and `...+1`.
- **CN**: 这里定义了类`ConvolutionBinaryInplace`、函数`__init__`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `packed`、`layout`、`inputs`、`constant_args`、`reordered_inputs`、`op_overload`、`另有1项` 等值。

### Lines 449-476 / 第 449-476 行
````python

        self.mutation_outputs = [
            MutationOutput(NoneLayout(device=inputs[0].get_device()), inputs[0], self),
            MutationOutput(NoneLayout(device=inputs[1].get_device()), inputs[1], self),
        ]

    def codegen(self, wrapper):
        wrapper.include_extra_header(
            f"torch/csrc/inductor/aoti_torch/c/shim_{self.device_type}.h"
        )
        super().codegen(wrapper)

    def get_unbacked_symbol_defs(self) -> OrderedSet[sympy.Symbol]:
        return OrderedSet()

    @classmethod
    def create(
        cls,
        x: "TensorBox",
        other: "TensorBox",
        weight: "TensorBox",
        bias: "TensorBox",
        padding_: list[int],
        stride_: list[int],
        dilation_: list[int],
        groups: int,
        binary_attr: str,
        binary_alpha: float | None,
````
- **EN**: Introduces function `codegen`, function `get_unbacked_symbol_defs`, function `create`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`codegen`、函数`get_unbacked_symbol_defs`、函数`create`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。处理与设备相关的行为、能力信息或派发决策。

### Lines 477-504 / 第 477-504 行
````python
        unary_attr: str | None,
        unary_scalars: list[Any] | None,
        unary_algorithm: str | None,
    ):
        (
            inputs,
            constant_args,
            _,
            req_stride_order,
            _,
        ) = _prepare_convolution_fusion_create(
            cls, x, weight, bias, padding_, stride_, dilation_, groups
        )
        # pyrefly: ignore [bad-assignment]
        other = cls.require_stride_order(other, req_stride_order)
        inputs.insert(1, other)
        constant_args = constant_args + [
            binary_attr,
            binary_alpha,
            unary_attr,
            may_convert_to_optional(unary_scalars),
            unary_algorithm,
        ]
        packed = ConvolutionBinaryInplace(
            kernel_layout=NoneLayout(device=inputs[1].get_device()),  # type: ignore[arg-type]
            inputs=inputs,
            constant_args=constant_args,
        )
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Initializes or updates values such as `unary_attr`, `unary_scalars`, `unary_algorithm`, `other`, `constant_args`, `packed`, and `...+2`. This range continues the implementation of function `ConvolutionBinaryInplace.create`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。初始化或更新了 `unary_attr`、`unary_scalars`、`unary_algorithm`、`other`、`constant_args`、`packed`、`另有2项` 等值。这一段延续了函数`ConvolutionBinaryInplace.create` 的具体实现。

### Lines 505-532 / 第 505-532 行
````python
        # This op mutates in place which means that the result is not the
        # target but rather the input that is being mutated
        # init reorders the inputs, so inputs[1] becomes packed.inputs[0]
        return packed.inputs[0]


class ConvolutionTransposeUnary(ExternKernelAlloc):
    def __init__(
        self,
        layout,
        inputs,
        constant_args=(),
    ) -> None:
        self.device_type = get_device_type(inputs[0])
        super().__init__(
            layout,
            inputs,
            constant_args,
            None,
            op_overload=torch.ops.mkldnn._convolution_transpose_pointwise.default,
            cpp_kernel_name=f"aoti_torch_{self.device_type}_mkldnn__convolution_transpose_pointwise",
        )

    def codegen(self, wrapper):
        wrapper.include_extra_header(
            f"torch/csrc/inductor/aoti_torch/c/shim_{self.device_type}.h"
        )
        super().codegen(wrapper)
````
- **EN**: Introduces class `ConvolutionTransposeUnary`, function `__init__`, function `codegen`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `constant_args`, `op_overload`, and `cpp_kernel_name`.
- **CN**: 这里定义了类`ConvolutionTransposeUnary`、函数`__init__`、函数`codegen`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `constant_args`、`op_overload`、`cpp_kernel_name` 等值。

### Lines 533-560 / 第 533-560 行
````python

    @classmethod
    def create(
        cls,
        x: "TensorBox",
        weight: "TensorBox",
        bias: "TensorBox",
        padding_: list[int],
        output_padding_: list[int],
        stride_: list[int],
        dilation_: list[int],
        groups_: int,
        attr,
        scalars: list[Any] | None,
        algorithm,
    ):
        transposed = True
        (
            inputs,
            constant_args,
            kernel_layout,
            _,
            _,
        ) = _prepare_convolution_fusion_create(
            cls,
            x,
            weight,
            bias,
````
- **EN**: Introduces function `create`. Applies decorators to register behavior or alter how the following definition is constructed. Initializes or updates values such as `x`, `weight`, `bias`, `padding_`, `output_padding_`, `stride_`, and `...+4`.
- **CN**: 这里定义了函数`create`。使用装饰器来注册行为，或改变后续定义的构造方式。初始化或更新了 `x`、`weight`、`bias`、`padding_`、`output_padding_`、`stride_`、`另有4项` 等值。

### Lines 561-588 / 第 561-588 行
````python
            padding_,
            stride_,
            dilation_,
            groups_,
            transposed,
            output_padding_,
        )
        constant_args = constant_args + [
            attr,
            may_convert_to_optional(scalars),
            algorithm,
        ]
        packed = ConvolutionTransposeUnary(
            layout=kernel_layout,
            inputs=inputs,
            constant_args=constant_args,
        )
        return _create_output_node(packed)


class QConvPointWisePT2E(ExternKernelAlloc):
    def __init__(
        self,
        layout,
        inputs,
        constant_args=(),
    ) -> None:
        """
````
- **EN**: Introduces class `QConvPointWisePT2E`, function `__init__`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `constant_args`, `packed`, `layout`, and `inputs`.
- **CN**: 这里定义了类`QConvPointWisePT2E`、函数`__init__`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `constant_args`、`packed`、`layout`、`inputs` 等值。

### Lines 589-616 / 第 589-616 行
````python
        if bias is not None
            - inputs = [x, w, b, weight_scale, weight_zp]
            - const_args is: [stride, padding, dilation, groups, x_scale, x_zp, o_scale, o_zp,
              fp32_output, unary_attr, unary_scalars, unary_algorithm]
        else
            - inputs = [x, w, weight_scale, weight_zp]
            - const_args is: [bias, stride, padding, dilation, groups, x_scale, x_zp, o_scale, o_zp,
              fp32_output, unary_attr, unary_scalars, unary_algorithm]
        """
        self.device_type = get_device_type(inputs[0])
        self.has_bias = len(inputs) == 5
        super().__init__(
            layout,
            inputs,
            constant_args,
            None,
            op_overload=torch.ops.onednn.qconv_pointwise.tensor,
            cpp_kernel_name=f"aoti_torch_{self.device_type}__qconv_pointwise_tensor",
        )

    def codegen(self, wrapper):
        wrapper.include_extra_header(
            f"torch/csrc/inductor/aoti_torch/c/shim_{self.device_type}.h"
        )
        super().codegen(wrapper)
        if isinstance(self.layout, Layout):
            self.codegen_size_asserts(wrapper)

````
- **EN**: Introduces function `codegen`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `op_overload`, and `cpp_kernel_name`.
- **CN**: 这里定义了函数`codegen`。包含分支、循环或上下文管理等控制流。初始化或更新了 `op_overload`、`cpp_kernel_name` 等值。

### Lines 617-644 / 第 617-644 行
````python
    @classmethod
    def create(
        cls,
        qx: "TensorBox",
        x_scale: "TensorBox",
        x_zero_point: "TensorBox",
        qw: "TensorBox",  # qw
        w_scale: "TensorBox",
        w_zero_point,
        bias: "TensorBox",
        stride: list[int],
        padding: list[int],
        dilation: list[int],
        groups: int,
        output_scale: float,
        output_zero_point: int,
        output_dtype,
        attr,
        scalars,
        algorithm,
    ):
        transposed = False
        output_padding = None
        (
            inputs,
            constant_args,
            kernel_layout,
            _,
````
- **EN**: Introduces function `create`. Applies decorators to register behavior or alter how the following definition is constructed. Initializes or updates values such as `qx`, `x_scale`, `x_zero_point`, `qw`, `w_scale`, `bias`, and `...+8`.
- **CN**: 这里定义了函数`create`。使用装饰器来注册行为，或改变后续定义的构造方式。初始化或更新了 `qx`、`x_scale`、`x_zero_point`、`qw`、`w_scale`、`bias`、`另有8项` 等值。

### Lines 645-672 / 第 645-672 行
````python
            _,
        ) = _prepare_convolution_fusion_create(
            cls,
            qx,
            qw,
            bias,
            padding,
            stride,
            dilation,
            groups,
            transposed,
            output_padding,
            [x_scale, x_zero_point, w_scale, w_zero_point],  # type: ignore[list-item]
        )
        # swap padding and stride to align with functional conv arg order
        if bias is None:
            constant_args[1], constant_args[2] = constant_args[2], constant_args[1]
        else:
            constant_args[0], constant_args[1] = constant_args[1], constant_args[0]

        constant_args = constant_args + [
            output_scale,
            output_zero_point,
            output_dtype,
            attr,
            may_convert_to_optional(scalars),
            algorithm,
        ]
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, and `constant_args`. This range continues the implementation of function `QConvPointWisePT2E.create`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`constant_args` 等值。这一段延续了函数`QConvPointWisePT2E.create` 的具体实现。

### Lines 673-700 / 第 673-700 行
````python

        assert output_dtype is not None
        if output_dtype in [torch.float32, torch.bfloat16]:
            # in _prepare_convolution_fusion_create, we use x.dtype (uint8) to create kernel_layout
            # if we set output_dtype is not None, the output buf should be output_dtype instead of uint8.
            kernel_layout.dtype = output_dtype

        return QConvPointWisePT2E(
            layout=kernel_layout,
            inputs=inputs,
            constant_args=constant_args,
        )


class QConvPointWiseBinaryPT2E(ExternKernelAlloc):
    def __init__(
        self,
        layout,
        inputs,
        constant_args=(),
    ) -> None:
        """
        Needs input/weight/output qparams
        if bias is not None
            - inputs = [x, x_scale, x_zp, w,  w_scale, w_zp, accum, b]
            - const_args = [stride, padding, dilation, groups, o_scale, o_zp,
            output_dtype, accum_scale, accum_zp, binary_attr, alpha, unary_attr, unary_scalars, unary_algorithm]
        else
````
- **EN**: Introduces class `QConvPointWiseBinaryPT2E`, function `__init__`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`QConvPointWiseBinaryPT2E`、函数`__init__`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 701-728 / 第 701-728 行
````python
            - inputs = [x, x_scale, x_zp, w,  w_scale, w_zp, accum]
            - const_args [b, stride, padding, dilation, groups, o_scale, o_zp,
             output_dtype, accum_scale, accum_zp, binary_attr, alpha, unary_attr, unary_scalars, unary_algorithm]
        """
        self.device_type = get_device_type(inputs[0])
        self.has_bias = len(inputs) == 8
        self.idx_for_inplace_sum = 6
        super().__init__(
            layout,
            inputs,
            constant_args,
            None,
            op_overload=torch.ops.onednn.qconv2d_pointwise.binary_tensor,
            cpp_kernel_name=(
                f"aoti_torch_{self.device_type}__qconv2d_pointwise_binary_tensor"
            ),
        )

    def codegen(self, wrapper):
        wrapper.include_extra_header(
            f"torch/csrc/inductor/aoti_torch/c/shim_{self.device_type}.h"
        )
        super().codegen(wrapper)
        if isinstance(self.layout, Layout):
            self.codegen_size_asserts(wrapper)

    def get_mutation_names(self) -> Sequence[str]:
        return [self.input_name(self.idx_for_inplace_sum)]
````
- **EN**: Introduces function `codegen`, function `get_mutation_names`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `op_overload`, and `cpp_kernel_name`.
- **CN**: 这里定义了函数`codegen`、函数`get_mutation_names`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `op_overload`、`cpp_kernel_name` 等值。

### Lines 729-756 / 第 729-756 行
````python

    def get_unbacked_symbol_defs(self) -> OrderedSet[sympy.Symbol]:
        return OrderedSet()

    @classmethod
    def create(
        cls,
        qx: "TensorBox",
        x_scale: "TensorBox",
        x_zero_point: "TensorBox",
        qw: "TensorBox",  # packed_weight
        w_scale,
        w_zero_point,
        qaccum: "TensorBox",
        bias: "TensorBox",
        stride: list[int],
        padding: list[int],
        dilation: list[int],
        groups: int,
        output_scale: "TensorBox",
        output_zero_point: "TensorBox",
        output_dtype,
        accum_scale,
        accum_zero_point,
        binary_attr,
        alpha,
        unary_attr,
        unary_scalars,
````
- **EN**: Introduces function `get_unbacked_symbol_defs`, function `create`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_unbacked_symbol_defs`、函数`create`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 757-784 / 第 757-784 行
````python
        unary_algorithm,
    ):
        transposed = False
        output_padding = None
        (
            inputs,
            constant_args,
            _kernel_layout,
            req_stride_order,
            qaccum,
        ) = _prepare_convolution_fusion_create(
            cls,
            qx,
            qw,
            bias,
            padding,
            stride,
            dilation,
            groups,
            transposed,
            output_padding,
            [x_scale, x_zero_point, w_scale, w_zero_point],
            qaccum,
        )

        # swap padding and stride to align with functional conv arg order
        if bias is None:
            constant_args[1], constant_args[2] = constant_args[2], constant_args[1]
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `transposed`, and `output_padding`. This range continues the implementation of function `QConvPointWiseBinaryPT2E.create`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `transposed`、`output_padding` 等值。这一段延续了函数`QConvPointWiseBinaryPT2E.create` 的具体实现。

### Lines 785-812 / 第 785-812 行
````python
        else:
            constant_args[0], constant_args[1] = constant_args[1], constant_args[0]

        constant_args = constant_args + [
            output_scale,
            output_zero_point,
            output_dtype,
            accum_scale,
            accum_zero_point,
            binary_attr,
            alpha,
            unary_attr,
            may_convert_to_optional(unary_scalars),
            unary_algorithm,
        ]

        assert binary_attr == "sum", (
            "For now, only post op sum is supported in QConvPointWiseBinaryPT2E."
        )

        V.graph.mark_buffer_mutated(qaccum.get_name())
        packed = QConvPointWiseBinaryPT2E(
            layout=NoneLayout(device=qaccum.get_device()),
            inputs=inputs,
            constant_args=constant_args,
        )

        # Return accum since it has been inplace changed.
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `constant_args`, `packed`, `layout`, and `inputs`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`constant_args`、`packed`、`layout`、`inputs` 等值。

### Lines 813-840 / 第 813-840 行
````python
        return packed.inputs[packed.idx_for_inplace_sum]


class MKLPackedLinear(ExternKernelAlloc):
    def __init__(
        self,
        layout,
        inputs,
        constant_args=(),
    ) -> None:
        super().__init__(
            layout,
            inputs,
            constant_args,
            None,
            op_overload=torch.ops.mkl._mkl_linear.default,
        )

    def codegen(self, wrapper):
        wrapper.include_extra_header("torch/csrc/inductor/aoti_torch/c/shim_cpu.h")
        super().codegen(wrapper)

    @classmethod
    def create(cls, x, packed_w, orig_w, B, batch_size):
        x = cls.require_stride1(cls.realize_input(x))
        orig_w = cls.require_stride1(cls.realize_input(orig_w))
        *m, _ = x.get_size()
        oc, _ = orig_w.get_size()
````
- **EN**: Introduces class `MKLPackedLinear`, function `__init__`, function `codegen`, function `create`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `constant_args`, `op_overload`, `x`, and `orig_w`.
- **CN**: 这里定义了类`MKLPackedLinear`、函数`__init__`、函数`codegen`、函数`create`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `constant_args`、`op_overload`、`x`、`orig_w` 等值。

### Lines 841-868 / 第 841-868 行
````python
        output_size = list(m) + [oc]
        output_stride = FlexibleLayout.contiguous_strides(output_size)
        inputs = [x, packed_w, orig_w]
        constant_args = [batch_size]
        if B is not None:
            inputs += [B]
        else:
            constant_args.insert(0, None)

        device = x.get_device()
        assert device is not None
        return MKLPackedLinear(
            layout=FixedLayout(device, x.get_dtype(), output_size, output_stride),
            inputs=inputs,
            constant_args=constant_args,
        )


class LinearUnary(ExternKernelAlloc):
    def __init__(
        self,
        layout,
        inputs,
        constant_args=(),
    ) -> None:
        self.device_type = get_device_type(inputs[0])
        super().__init__(
            layout,
````
- **EN**: Introduces class `LinearUnary`, function `__init__`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`LinearUnary`、函数`__init__`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 869-896 / 第 869-896 行
````python
            inputs,
            constant_args,
            None,
            op_overload=torch.ops.mkldnn._linear_pointwise.default,
            cpp_kernel_name=f"aoti_torch_{self.device_type}__linear_pointwise",
        )

    def codegen(self, wrapper):
        wrapper.include_extra_header(
            f"torch/csrc/inductor/aoti_torch/c/shim_{self.device_type}.h"
        )
        super().codegen(wrapper)

    @classmethod
    def create(cls, x, w, B, attr, scalars, algorithm):
        x = cls.require_contiguous(cls.realize_input(x))
        w = cls.require_contiguous(cls.realize_input(w))

        *m, _ic = x.get_size()
        oc, _ic = w.get_size()
        output_size = list(m) + [oc]
        inputs = [x, w]
        constant_args = [attr, scalars if scalars else [-1], algorithm]
        if B is not None:
            B = cls.require_contiguous(cls.realize_input(B))
            inputs.append(B)
        else:
            constant_args.insert(0, None)
````
- **EN**: Introduces function `codegen`, function `create`. Applies decorators to register behavior or alter how the following definition is constructed. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `op_overload`, `cpp_kernel_name`, `x`, `w`, `output_size`, `inputs`, and `...+3`.
- **CN**: 这里定义了函数`codegen`、函数`create`。使用装饰器来注册行为，或改变后续定义的构造方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `op_overload`、`cpp_kernel_name`、`x`、`w`、`output_size`、`inputs`、`另有3项` 等值。

### Lines 897-924 / 第 897-924 行
````python

        device = x.get_device()
        assert device is not None

        packed = LinearUnary(
            layout=FixedLayout(
                device=device,
                dtype=x.get_dtype(),
                size=output_size,
            ),
            inputs=inputs,
            constant_args=constant_args,
        )
        return _create_output_node(packed)

    def apply_constraint(self):
        pass


class LinearBinary(ExternKernelAlloc):
    kernel = "torch.ops.mkldnn._linear_pointwise.binary"

    def __init__(
        self,
        layout,
        inputs,
        constant_args=(),
    ) -> None:
````
- **EN**: Introduces function `apply_constraint`, class `LinearBinary`, function `__init__`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`apply_constraint`、类`LinearBinary`、函数`__init__`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 925-952 / 第 925-952 行
````python
        self.device_type = get_device_type(inputs[0])
        super().__init__(
            layout,
            inputs,
            constant_args,
            None,
            op_overload=torch.ops.mkldnn._linear_pointwise.binary,
            cpp_kernel_name=f"aoti_torch_{self.device_type}__linear_pointwise_binary",
        )

    def codegen(self, wrapper):
        wrapper.include_extra_header(
            f"torch/csrc/inductor/aoti_torch/c/shim_{self.device_type}.h"
        )
        super().codegen(wrapper)

    @classmethod
    def create(cls, x, y, w, B, attr):
        x = cls.require_contiguous(cls.realize_input(x))
        y = cls.require_contiguous(cls.realize_input(y))
        w = cls.require_contiguous(cls.realize_input(w))

        *m, _ic = x.get_size()
        oc, _ic = w.get_size()
        output_size = list(m) + [oc]
        inputs = [x, y, w]
        constant_args = [attr]
        if B is not None:
````
- **EN**: Introduces function `codegen`, function `create`. Applies decorators to register behavior or alter how the following definition is constructed. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `op_overload`, `cpp_kernel_name`, `x`, `y`, `w`, `output_size`, and `...+2`.
- **CN**: 这里定义了函数`codegen`、函数`create`。使用装饰器来注册行为，或改变后续定义的构造方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `op_overload`、`cpp_kernel_name`、`x`、`y`、`w`、`output_size`、`另有2项` 等值。

### Lines 953-980 / 第 953-980 行
````python
            B = cls.require_contiguous(cls.realize_input(B))
            inputs.append(B)
        else:
            constant_args.insert(0, B)

        device = x.get_device()
        assert device is not None
        packed = LinearBinary(
            layout=FixedLayout(
                device=device,
                dtype=x.get_dtype(),
                size=output_size,
            ),
            inputs=inputs,
            constant_args=constant_args,
        )
        return _create_output_node(packed)

    def apply_constraint(self):
        pass


class QLinearPointwisePT2E(ExternKernelAlloc):
    def __init__(
        self,
        layout,
        inputs,
        constant_args=(),
````
- **EN**: Introduces function `apply_constraint`, class `QLinearPointwisePT2E`, function `__init__`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`apply_constraint`、类`QLinearPointwisePT2E`、函数`__init__`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 981-1008 / 第 981-1008 行
````python
        has_bias=True,
    ) -> None:
        """
        if bias is not None
            - inputs = [x, w, b, weight_scale, weight_zp]
            - const_args is: [x_scale, x_zp, o_scale, o_zp,
              fp32_output, unary_attr, unary_scalars, unary_algorithm]
        else
            - inputs = [x, w, weight_scale, weight_zp]
            - const_args is: [bias, x_scale, x_zp, o_scale, o_zp,
              fp32_output, unary_attr, unary_scalars, unary_algorithm]
        """
        self.device_type = get_device_type(inputs[0])
        self.has_bias = has_bias
        super().__init__(
            layout,
            inputs,
            constant_args,
            None,
            op_overload=(torch.ops.onednn.qlinear_pointwise.tensor),
            cpp_kernel_name=(
                f"aoti_torch_{self.device_type}__qlinear_pointwise_tensor"
            ),
        )

    def codegen(self, wrapper):
        wrapper.include_extra_header(
            f"torch/csrc/inductor/aoti_torch/c/shim_{self.device_type}.h"
````
- **EN**: Introduces function `codegen`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `has_bias`, `op_overload`, and `cpp_kernel_name`.
- **CN**: 这里定义了函数`codegen`。包含分支、循环或上下文管理等控制流。初始化或更新了 `has_bias`、`op_overload`、`cpp_kernel_name` 等值。

### Lines 1009-1036 / 第 1009-1036 行
````python
        )
        super().codegen(wrapper)

        if isinstance(self.layout, Layout):
            self.codegen_size_asserts(wrapper)

    @classmethod
    def create(
        cls,
        qx: "TensorBox",
        x_scale: "TensorBox",
        x_zero_point: "TensorBox",
        qw: "TensorBox",  # packed_weight
        w_scale: "TensorBox",
        w_zero_point: "TensorBox",
        bias: "TensorBox",
        output_scale: float,
        output_zero_point: int,
        output_dtype,
        post_op_name,
        post_op_args,
        post_op_algorithm,
    ):
        (inputs, constant_args, kernel_layout, _, _) = _prepare_linear_fusion_create(
            cls,
            qx,
            qw,
            bias,
````
- **EN**: Introduces function `create`. Applies decorators to register behavior or alter how the following definition is constructed. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `qx`, `x_scale`, `x_zero_point`, `qw`, `w_scale`, `w_zero_point`, and `...+3`.
- **CN**: 这里定义了函数`create`。使用装饰器来注册行为，或改变后续定义的构造方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `qx`、`x_scale`、`x_zero_point`、`qw`、`w_scale`、`w_zero_point`、`另有3项` 等值。

### Lines 1037-1064 / 第 1037-1064 行
````python
            [x_scale, x_zero_point, w_scale, w_zero_point],
        )

        constant_args = constant_args + [
            output_scale,
            output_zero_point,
            output_dtype,
            post_op_name,
            may_convert_to_optional(post_op_args),
            post_op_algorithm,
        ]

        assert output_dtype is not None
        if output_dtype in [torch.float32, torch.bfloat16]:
            # in _prepare_linear_fusion_create, we use x.dtype (uint8) to create kernel_layout
            # if we set fp32_output, the output buf should be dtype float32 instead of uint8.
            kernel_layout.dtype = output_dtype

        return QLinearPointwisePT2E(
            layout=kernel_layout,
            inputs=inputs,
            constant_args=constant_args,
            has_bias=(bias is not None),
        )


class QLinearPointwiseBinaryPT2E(ExternKernelAlloc):
    def __init__(
````
- **EN**: Introduces class `QLinearPointwiseBinaryPT2E`, function `__init__`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`QLinearPointwiseBinaryPT2E`、函数`__init__`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1065-1092 / 第 1065-1092 行
````python
        self,
        layout,
        inputs,
        constant_args=(),
        has_bias=True,
    ) -> None:
        """
        if bias is not None
            - inputs = [x, w, x_scale, x_zp, weight_scale, weight_zp, x2, bias]
            - const_args is: [o_scale, o_zp,
              fp32_output, binary_attr, alpha, unary_attr, unary_scalars, unary_algorithm]
        else
            - inputs = [x, w, x_scale, x_zp, weight_scale, weight_zp, x2]
            - const_args is: [bias, o_scale, o_zp,
              fp32_output, binary_attr, alpha, unary_attr, unary_scalars, unary_algorithm]
        """
        self.device_type = get_device_type(inputs[0])
        self.has_bias = has_bias
        self.idx_for_inplace_sum = 6
        super().__init__(
            layout,
            inputs,
            constant_args,
            None,
            op_overload=(torch.ops.onednn.qlinear_pointwise.binary_tensor),
            cpp_kernel_name=f"aoti_torch_{self.device_type}__qlinear_pointwise_binary_tensor",
        )

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `constant_args`, `has_bias`, `op_overload`, and `cpp_kernel_name`. This range continues the implementation of function `QLinearPointwiseBinaryPT2E.__init__`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `constant_args`、`has_bias`、`op_overload`、`cpp_kernel_name` 等值。这一段延续了函数`QLinearPointwiseBinaryPT2E.__init__` 的具体实现。

### Lines 1093-1120 / 第 1093-1120 行
````python
    def codegen(self, wrapper):
        wrapper.include_extra_header(
            f"torch/csrc/inductor/aoti_torch/c/shim_{self.device_type}.h"
        )
        super().codegen(wrapper)
        if isinstance(self.layout, Layout):
            self.codegen_size_asserts(wrapper)

    def get_mutation_names(self) -> Sequence[str]:
        binary_post_op = self.constant_args[-5]
        if binary_post_op == "sum":
            input = self.inputs[self.idx_for_inplace_sum]
            assert isinstance(input, IRNode)
            return [input.get_name()]
        else:
            return []

    @classmethod
    def create(
        cls,
        qx: "TensorBox",
        x_scale: "TensorBox",
        x_zero_point: "TensorBox",
        qw: "TensorBox",  # packed_weight
        w_scale: "TensorBox",
        w_zero_point: "TensorBox",
        other: "TensorBox",
        bias: "TensorBox",
````
- **EN**: Introduces function `codegen`, function `get_mutation_names`, function `create`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`codegen`、函数`get_mutation_names`、函数`create`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1121-1148 / 第 1121-1148 行
````python
        output_scale: float,
        output_zero_point: int,
        output_dtype,
        other_scale,
        other_zp,
        binary_post_op,
        binary_alpha,
        unary_post_op,
        unary_post_op_args,
        unary_post_op_algorithm,
    ):
        (
            inputs,
            constant_args,
            kernel_layout,
            req_stride_order,
            other,
        ) = _prepare_linear_fusion_create(
            cls,
            qx,
            qw,
            bias,
            [x_scale, x_zero_point, w_scale, w_zero_point],
            other,
            binary_post_op == "sum",
        )

        constant_args = constant_args + [
````
- **EN**: Initializes or updates values such as `output_scale`, `output_zero_point`, `binary_post_op`, and `constant_args`. This range continues the implementation of function `QLinearPointwiseBinaryPT2E.create`.
- **CN**: 初始化或更新了 `output_scale`、`output_zero_point`、`binary_post_op`、`constant_args` 等值。这一段延续了函数`QLinearPointwiseBinaryPT2E.create` 的具体实现。

### Lines 1149-1176 / 第 1149-1176 行
````python
            output_scale,
            output_zero_point,
            output_dtype,
            other_scale,
            other_zp,
            binary_post_op,
            binary_alpha,
            unary_post_op,
            may_convert_to_optional(unary_post_op_args),
            unary_post_op_algorithm,
        ]

        if binary_post_op == "sum":
            V.graph.mark_buffer_mutated(other.get_name())
            packed = QLinearPointwiseBinaryPT2E(
                layout=NoneLayout(device=other.get_device()),
                inputs=inputs,
                constant_args=constant_args,
                has_bias=(bias is not None),
            )
            # Return other since it has been inplace changed.
            return packed.inputs[packed.idx_for_inplace_sum]

        assert output_dtype is not None
        if output_dtype in [torch.float32, torch.bfloat16]:
            # in _prepare_linear_fusion_create, we use x.dtype (uint8) to create kernel_layout
            # if we set fp32_output, the output buf should be dtype float32 instead of uint8.
            kernel_layout.dtype = output_dtype
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1177-1204 / 第 1177-1204 行
````python

        return QLinearPointwiseBinaryPT2E(
            layout=kernel_layout,
            inputs=inputs,
            constant_args=constant_args,
            has_bias=(bias is not None),
        )


class MkldnnRnnLayer(ExternKernelAlloc):
    def __init__(
        self,
        layout,
        inputs,
        constant_args=(),
    ) -> None:
        super().__init__(
            layout,
            inputs,
            constant_args,
            None,
            op_overload=torch.ops.aten.mkldnn_rnn_layer.default,
        )

    @classmethod
    def create(
        cls,
        x: "TensorBox",
````
- **EN**: Introduces class `MkldnnRnnLayer`, function `__init__`, function `create`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `layout`, `inputs`, `constant_args`, `has_bias`, `op_overload`, and `x`.
- **CN**: 这里定义了类`MkldnnRnnLayer`、函数`__init__`、函数`create`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `layout`、`inputs`、`constant_args`、`has_bias`、`op_overload`、`x` 等值。

### Lines 1205-1232 / 第 1205-1232 行
````python
        w0: "TensorBox",
        w1: "TensorBox",
        w2: "TensorBox",
        w3: "TensorBox",
        hx: "TensorBox",
        cx: "TensorBox",
        reverse: bool,
        batch_sizes: list[int],
        mode: int,
        hidden_size: int,
        num_layers: int,
        has_biases: bool,
        bidirectional: bool,
        batch_first: bool,
        train: bool,
    ):
        # pyrefly: ignore [bad-assignment]
        x = cls.require_stride1(cls.realize_input(x))
        # If batch_first, x has been permuted in lstm before entering the mkldnn_rnn_layer.
        # Make sure x is contiguous in batch_first case.
        x.freeze_layout()
        # pyrefly: ignore [bad-assignment]
        w0 = cls.require_stride1(cls.realize_input(w0))
        # pyrefly: ignore [bad-assignment]
        w1 = cls.require_stride1(cls.realize_input(w1))
        # pyrefly: ignore [bad-assignment]
        w2 = cls.require_stride1(cls.realize_input(w2))
        # pyrefly: ignore [bad-assignment]
````
- **EN**: Initializes or updates values such as `w0`, `w1`, `w2`, `w3`, `hx`, `cx`, and `...+10`. This range continues the implementation of function `MkldnnRnnLayer.create`.
- **CN**: 初始化或更新了 `w0`、`w1`、`w2`、`w3`、`hx`、`cx`、`另有10项` 等值。这一段延续了函数`MkldnnRnnLayer.create` 的具体实现。

### Lines 1233-1260 / 第 1233-1260 行
````python
        w3 = cls.require_stride1(cls.realize_input(w3))
        # pyrefly: ignore [bad-assignment]
        hx = cls.require_stride1(cls.realize_input(hx))
        hx.freeze_layout()
        # pyrefly: ignore [bad-assignment]
        cx = cls.require_stride1(cls.realize_input(cx))
        cx.freeze_layout()

        input_size = x.get_size()
        assert len(input_size) == 3, "Expect lstm input to be 3D"
        # batch_first is handled in the lstm OP. When entering
        # rnn_layer here, we'll always have batch_first = False
        seq_length, mini_batch, input_size = input_size
        output_shape = [seq_length, mini_batch, hidden_size]

        hy_shape = hx.get_size()
        cy_shape = cx.get_size()

        inputs = [x, w0, w1, w2, w3, hx, cx]
        constant_args = [
            reverse,
            batch_sizes,
            mode,
            hidden_size,
            num_layers,
            has_biases,
            bidirectional,
            batch_first,
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `w3`, `hx`, `cx`, `input_size`, `output_shape`, `hy_shape`, and `...+3`. This range continues the implementation of function `MkldnnRnnLayer.create`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `w3`、`hx`、`cx`、`input_size`、`output_shape`、`hy_shape`、`另有3项` 等值。这一段延续了函数`MkldnnRnnLayer.create` 的具体实现。

### Lines 1261-1288 / 第 1261-1288 行
````python
            train,
        ]

        device = x.get_device()
        assert device is not None
        packed = MkldnnRnnLayer(
            MultiOutputLayout(device=device),
            inputs=inputs,
            constant_args=constant_args,
        )

        def get_strides_of_lstm_output(output_shape, batch_first):
            assert len(output_shape) == 3, "Expect output_shape to be 3D"
            return FlexibleLayout.contiguous_strides(output_shape)

        # C shim call requires all the outputs to be passed in, and thus the last
        # dummy return value is added.
        output_sizes = [output_shape, hy_shape, cy_shape, [1]]
        output_strides = [
            get_strides_of_lstm_output(output_shape, batch_first),
            FlexibleLayout.contiguous_strides(hy_shape),
            FlexibleLayout.contiguous_strides(cy_shape),
            [1],
        ]
        output_ir = [
            MultiOutput(
                FixedLayout(
                    x.get_device(),  # type: ignore[arg-type]
````
- **EN**: Introduces function `get_strides_of_lstm_output`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `device`, `packed`, `inputs`, `constant_args`, `output_sizes`, `output_strides`, and `...+1`.
- **CN**: 这里定义了函数`get_strides_of_lstm_output`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `device`、`packed`、`inputs`、`constant_args`、`output_sizes`、`output_strides`、`另有1项` 等值。

### Lines 1289-1316 / 第 1289-1316 行
````python
                    x.get_dtype(),
                    output_size,
                    output_stride,
                ),
                packed,
                [(tuple, i)],
            )
            for i, (output_size, output_stride) in enumerate(
                zip(output_sizes, output_strides)
            )
        ]
        packed.outputs = output_ir

        return output_ir

    def codegen(self, wrapper):
        wrapper.include_extra_header("torch/csrc/inductor/aoti_torch/c/shim_cpu.h")
        return super().codegen(wrapper)


# Add this IR so that we can include shim_cpu.h for cpp_wrapper
class WeightInt4PackMatmul(ExternKernelAlloc):
    def __init__(
        self,
        layout,
        inputs,
        constant_args=(),
    ) -> None:
````
- **EN**: Introduces function `codegen`, class `WeightInt4PackMatmul`, function `__init__`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `constant_args`.
- **CN**: 这里定义了函数`codegen`、类`WeightInt4PackMatmul`、函数`__init__`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `constant_args` 等值。

### Lines 1317-1344 / 第 1317-1344 行
````python
        """
        inputs = [x, w, qGroupSize, qScalesAndZeros]
        constant_args = ()
        """
        assert len(inputs) == 4
        assert len(constant_args) == 0
        super().__init__(
            layout,
            inputs,
            constant_args,
            None,
            op_overload=(torch.ops.quantized.int4mm_packed_weight_cpu.default),
            cpp_kernel_name=("aoti_torch_cpu__weight_int4pack_mm_cpu_tensor"),
        )

    def codegen(self, wrapper):
        wrapper.include_extra_header("torch/csrc/inductor/aoti_torch/c/shim_cpu.h")
        super().codegen(wrapper)

        if isinstance(self.layout, Layout):
            self.codegen_size_asserts(wrapper)

    @classmethod
    def create(
        cls,
        x: "TensorBox",
        w: "TensorBox",
        qGroupSize: "TensorBox",
````
- **EN**: Introduces function `codegen`, function `create`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`codegen`、函数`create`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1345-1361 / 第 1345-1361 行
````python
        qScalesAndZeros: "TensorBox",
    ):
        inputs = [x, w, qGroupSize, qScalesAndZeros]
        *m, _ = x.get_size()
        n, _ = w.get_size()
        output_size = list(m) + [n]
        output_stride = FlexibleLayout.contiguous_strides(output_size)
        kernel_layout = FixedLayout(
            x.get_device(),  # type: ignore[arg-type]
            x.get_dtype(),
            output_size,
            output_stride,
        )
        return WeightInt4PackMatmul(
            layout=kernel_layout,
            inputs=inputs,
        )
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `qScalesAndZeros`, `inputs`, `output_size`, `output_stride`, `kernel_layout`, and `layout`. This range continues the implementation of function `WeightInt4PackMatmul.create`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `qScalesAndZeros`、`inputs`、`output_size`、`output_stride`、`kernel_layout`、`layout` 等值。这一段延续了函数`WeightInt4PackMatmul.create` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `ConvolutionUnary`, `ConvolutionBinary`, `ConvolutionBinaryInplace`, `ConvolutionTransposeUnary`, `QConvPointWisePT2E`, `QConvPointWiseBinaryPT2E`, and `...+7`  
  **CN**: 主要类：`ConvolutionUnary`、`ConvolutionBinary`、`ConvolutionBinaryInplace`、`ConvolutionTransposeUnary`、`QConvPointWisePT2E`、`QConvPointWiseBinaryPT2E`、`另有7项`
- **EN**: Primary functions: `_prepare_convolution_fusion_create`, `_prepare_linear_fusion_create`, and `_create_output_node`  
  **CN**: 主要函数：`_prepare_convolution_fusion_create`、`_prepare_linear_fusion_create`、`_create_output_node`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections.abc`, `typing`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._prims_common`, `torch.utils._ordered_set`, `.ir`, `.utils`, `.virtualized`
