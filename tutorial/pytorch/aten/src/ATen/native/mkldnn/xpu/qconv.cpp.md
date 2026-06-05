# qconv.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/mkldnn/xpu/qconv.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the oneDNN/MKLDNN integration in PyTorch ATen native code and focuses on qconv; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的oneDNN/MKLDNN 集成，主题聚焦于 qconv；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60
```cpp
#include <ATen/core/op_registration/op_registration.h>
#include <ATen/native/mkldnn/xpu/detail/oneDNN.h>
#include <ATen/native/mkldnn/xpu/qconv.h>

#include <c10/core/MemoryFormat.h>
#include <c10/core/ScalarType.h>
#include <torch/library.h>

using namespace at::native::onednn;
namespace at::native::xpu {

inline c10::ScalarType QConvoneDNNXPU::qconv_decide_out_dtype(
    const at::Tensor& act,
    const std::optional<c10::ScalarType> output_dtype) {
  bool fp32_output = output_dtype.has_value() && (output_dtype == c10::kFloat);
  bool bfloat16_output =
      output_dtype.has_value() && (output_dtype == c10::kBFloat16);
  auto dst_dtype = fp32_output
      ? c10::kFloat
      : (bfloat16_output ? c10::kBFloat16 : act.scalar_type());
  return dst_dtype;
}

at::Tensor QConvoneDNNXPU::qconv_prepack_xpu(
    at::Tensor weight,
    at::Tensor weight_scales,
    double input_scale,
    int64_t input_zero_point,
    torch::List<int64_t> stride,
    torch::List<int64_t> padding,
    torch::List<int64_t> dilation,
    int64_t groups,
    std::optional<torch::List<int64_t>> input_shape) {
  // XPU has no prepack at present
  return weight;
}

at::Tensor QConvoneDNNXPU::run_pointwise(
    at::Tensor act,
    double act_scale,
    int64_t act_zero_point,
    at::Tensor weight,
    at::Tensor weight_scales,
    at::Tensor weight_zero_points,
    std::optional<at::Tensor> bias,
    torch::List<int64_t> stride,
    torch::List<int64_t> padding,
    torch::List<int64_t> dilation,
    int64_t groups,
    double inv_output_scale,
    int64_t output_zero_point,
    std::optional<c10::ScalarType> output_dtype,
    std::string_view attr,
    torch::List<std::optional<at::Scalar>> scalars,
    std::optional<std::string_view> algorithm) {
  if (act.dim() == 3 || act.dim() == 5) {
    TORCH_CHECK(
        attr == "none",
        "quantized pointwise conv",
        act.dim() - 2,
```
- EN: Lines 1-60 pull in 6 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 第 1-60 行引入了 6 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 61-120
```cpp
        "d doesn't support unary_post_op fusion. Got unary_post_op:",
        attr,
        ".");
  } else {
    TORCH_CHECK(
        attr == "none" || attr == "relu" || attr == "hardtanh" ||
            attr == "hardswish" || attr == "swish",
        "We support quantized convolution without any post-ops or combinations for Quantized Conv + ReLU, Hardtanh, GELU, Swish, and Hardswish are supported. However, encountered unsupported post operation:",
        attr,
        ".");
  }

  bool is_channels_last_suggested = use_channels_last_for_conv(act, weight);
  auto mfmt = is_channels_last_suggested ? get_cl_tag_by_ndim(act.ndimension())
                                         : at::MemoryFormat::Contiguous;
  Tensor input_ = act.contiguous(mfmt);
  Tensor weight_ = weight.contiguous(mfmt);

  auto dst_tz = conv_dst_size(
      input_.ndimension(),
      input_.sizes(),
      weight_.sizes(),
      padding.vec(),
      padding.vec(),
      stride.vec(),
      dilation.vec());

  auto dst_dtype = qconv_decide_out_dtype(act, output_dtype);
  Tensor output =
      at::empty(dst_tz, act.options().dtype(dst_dtype).memory_format(mfmt));

  return quantized_convolution(
      act,
      act_scale,
      act_zero_point,
      weight,
      weight_scales,
      weight_zero_points,
      bias,
      stride,
      padding,
      dilation,
      /*transposed*/ false,
      groups,
      output,
      inv_output_scale,
      output_zero_point,
      /*accum*/ std::nullopt,
      /*accum_scale*/ 0.0,
      /*accum_zero_point*/ 0,
      /*output_dtype*/ output_dtype,
      /*binary_attr*/ std::nullopt,
      /*binary_alpha*/ std::nullopt,
      /*unary_attr*/ attr,
      /*unary_scalars*/ scalars,
      /*unary_algorithm*/ algorithm);
}

at::Tensor QConvoneDNNXPU::run_pointwise_tensor(
    at::Tensor act,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are get_cl_tag_by_ndim, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 get_cl_tag_by_ndim，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 121-180
```cpp
    at::Tensor act_scale,
    at::Tensor act_zero_point,
    at::Tensor weight,
    at::Tensor weight_scales,
    at::Tensor weight_zero_points,
    std::optional<at::Tensor> bias,
    torch::List<int64_t> stride,
    torch::List<int64_t> padding,
    torch::List<int64_t> dilation,
    int64_t groups,
    double output_scale,
    int64_t output_zero_point,
    std::optional<c10::ScalarType> output_dtype,
    std::string_view attr,
    torch::List<std::optional<at::Scalar>> scalars,
    std::optional<std::string_view> algorithm) {
  return run_pointwise(
      act,
      act_scale.item().toDouble(),
      act_zero_point.item().toLong(),
      weight,
      weight_scales,
      weight_zero_points,
      bias,
      stride,
      padding,
      dilation,
      groups,
      output_scale,
      output_zero_point,
      output_dtype,
      /*unary_attr*/ attr,
      /*unary_scalars*/ scalars,
      /*unary_algorithm*/ algorithm);
}

at::Tensor QConvoneDNNXPU::run_pointwise_binary(
    at::Tensor act,
    double act_scale,
    int64_t act_zero_point,
    at::Tensor weight,
    at::Tensor weight_scales,
    at::Tensor weight_zero_points,
    at::Tensor accum,
    std::optional<at::Tensor> bias,
    torch::List<int64_t> stride,
    torch::List<int64_t> padding,
    torch::List<int64_t> dilation,
    int64_t groups,
    double output_scale,
    int64_t output_zero_point,
    std::optional<c10::ScalarType> output_dtype,
    double accum_scale,
    int64_t accum_zero_point,
    std::string_view binary_attr,
    std::optional<at::Scalar> alpha,
    std::optional<std::string_view> unary_attr,
    torch::List<std::optional<at::Scalar>> unary_scalars,
    std::optional<std::string_view> unary_algorithm) {
  TORCH_CHECK(
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 181-240
```cpp
      act.dim() == 4 && binary_attr == "sum" &&
          (!unary_attr.has_value() ||
           (unary_attr.has_value() &&
            (unary_attr.value() == "none" || unary_attr.value() == "relu"))),
      "post_op sum or post_op sum_relu is supported for quantized pointwise conv2d. Got binary_post_op: ",
      binary_attr,
      " unary_post_op: ",
      unary_attr.has_value() ? unary_attr.value() : "none",
      ".")

  bool is_channels_last_suggested = use_channels_last_for_conv(act, weight);
  auto mfmt = is_channels_last_suggested ? get_cl_tag_by_ndim(act.ndimension())
                                         : at::MemoryFormat::Contiguous;
  Tensor input_ = act.contiguous(mfmt);
  Tensor weight_ = weight.contiguous(mfmt);

  auto dst_tz = conv_dst_size(
      input_.ndimension(),
      input_.sizes(),
      weight_.sizes(),
      padding.vec(),
      padding.vec(),
      stride.vec(),
      dilation.vec());

  auto dst_dtype = qconv_decide_out_dtype(act, output_dtype);
  bool has_accum_postop_sum = binary_attr == "sum";
  Tensor output = has_accum_postop_sum
      ? accum
      : at::empty(dst_tz, act.options().dtype(dst_dtype).memory_format(mfmt));

  output = quantized_convolution(
      act,
      act_scale,
      act_zero_point,
      weight,
      weight_scales,
      weight_zero_points,
      bias,
      stride,
      padding,
      dilation,
      /*transposed*/ false,
      groups,
      output,
      output_scale,
      output_zero_point,
      /*accum*/ accum,
      /*accum_scale*/ accum_scale,
      /*accum_zero_point*/ accum_zero_point,
      /*output_dtype*/ output_dtype,
      /*binary_attr*/ binary_attr,
      /*binary_alpha*/ alpha,
      /*unary_attr*/ unary_attr,
      /*unary_scalars*/ unary_scalars,
      /*unary_algorithm*/ unary_algorithm);

  if (!has_accum_postop_sum) {
    return output;
  } else {
```
- EN: The main callable definitions or declarations in this block are get_cl_tag_by_ndim, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段的主要可调用定义或声明包括 get_cl_tag_by_ndim，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 241-300
```cpp
    return accum;
  }
}

at::Tensor QConvoneDNNXPU::run_pointwise_binary_tensor(
    at::Tensor act, // contains quantized values but not QTensor
    at::Tensor act_scale,
    at::Tensor act_zero_point,
    at::Tensor weight, // contains quantized values but not QTensor
    at::Tensor weight_scales,
    at::Tensor weight_zero_points,
    at::Tensor accum, // contains quantized values but not QTensor
    std::optional<at::Tensor> bias,
    torch::List<int64_t> stride,
    torch::List<int64_t> padding,
    torch::List<int64_t> dilation,
    int64_t groups,
    double output_scale,
    int64_t output_zero_point,
    std::optional<c10::ScalarType> output_dtype,
    double accum_scale,
    int64_t accum_zero_point,
    std::string_view binary_attr,
    std::optional<at::Scalar> alpha,
    std::optional<std::string_view> unary_attr,
    torch::List<std::optional<at::Scalar>> unary_scalars,
    std::optional<std::string_view> unary_algorithm) {
  return run_pointwise_binary(
      act,
      act_scale.item().toDouble(),
      act_zero_point.item().toLong(),
      weight,
      weight_scales,
      weight_zero_points,
      accum,
      bias,
      stride,
      padding,
      dilation,
      groups,
      output_scale,
      output_zero_point,
      output_dtype,
      accum_scale,
      accum_zero_point,
      binary_attr,
      alpha,
      unary_attr,
      unary_scalars,
      unary_algorithm);
}

TORCH_LIBRARY_IMPL(onednn, XPU, m) {
  m.impl(
      TORCH_SELECTIVE_NAME("onednn::qconv_prepack"),
      TORCH_FN(QConvoneDNNXPU::qconv_prepack_xpu));
  m.impl(
      TORCH_SELECTIVE_NAME("onednn::qconv1d_pointwise"),
      QConvoneDNNXPU::run_pointwise);
  m.impl(
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are TORCH_LIBRARY_IMPL, concentrating a specific part of the operator behavior.
- EN: Registration or dispatch macros appear here, wiring the implementation into PyTorch operator lookup and backend dispatch.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 TORCH_LIBRARY_IMPL，它们承载了某一部分算子行为的核心逻辑。
- CN: 这里出现了注册或分发宏，用于把实现接入 PyTorch 的算子查找与后端分发机制。

### Lines 301-320
```cpp
      TORCH_SELECTIVE_NAME("onednn::qconv2d_pointwise"),
      QConvoneDNNXPU::run_pointwise);
  m.impl(
      TORCH_SELECTIVE_NAME("onednn::qconv3d_pointwise"),
      QConvoneDNNXPU::run_pointwise);
  m.impl(
      TORCH_SELECTIVE_NAME("onednn::qconv2d_pointwise.binary"),
      QConvoneDNNXPU::run_pointwise_binary);
  m.impl(
      TORCH_SELECTIVE_NAME("onednn::qconv_pointwise"),
      QConvoneDNNXPU::run_pointwise);
  m.impl(
      TORCH_SELECTIVE_NAME("onednn::qconv_pointwise.tensor"),
      QConvoneDNNXPU::run_pointwise_tensor);
  m.impl(
      TORCH_SELECTIVE_NAME("onednn::qconv2d_pointwise.binary_tensor"),
      QConvoneDNNXPU::run_pointwise_binary_tensor);
}

} // namespace at::native::xpu
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

## Key Concepts / 关键概念

- EN: Backend focus: XPU backend.
- CN: 后端重点：XPU 后端。
- EN: Backend focus: oneDNN/MKLDNN integration.
- CN: 后端重点：oneDNN/MKLDNN 集成。
- EN: Backend focus: MKL integration.
- CN: 后端重点：MKL 集成。
- EN: Registration/dispatch logic is part of the file structure.
- CN: 文件结构中包含注册/分发逻辑。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: qconv_decide_out_dtype, qconv_prepack_xpu, run_pointwise, run_pointwise_tensor, run_pointwise_binary, TORCH_CHECK, run_pointwise_binary_tensor, TORCH_LIBRARY_IMPL.
- CN: 重要符号：qconv_decide_out_dtype, qconv_prepack_xpu, run_pointwise, run_pointwise_tensor, run_pointwise_binary, TORCH_CHECK, run_pointwise_binary_tensor, TORCH_LIBRARY_IMPL。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/core/op_registration/op_registration.h, ATen/native/mkldnn/xpu/detail/oneDNN.h, ATen/native/mkldnn/xpu/qconv.h, c10/core/MemoryFormat.h, c10/core/ScalarType.h, torch/library.h`.
- CN: 主要内部头文件：`ATen/core/op_registration/op_registration.h, ATen/native/mkldnn/xpu/detail/oneDNN.h, ATen/native/mkldnn/xpu/qconv.h, c10/core/MemoryFormat.h, c10/core/ScalarType.h, torch/library.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `qconv_decide_out_dtype, qconv_prepack_xpu, run_pointwise, run_pointwise_tensor, run_pointwise_binary, TORCH_CHECK, run_pointwise_binary_tensor, TORCH_LIBRARY_IMPL`.
- CN: 实现围绕 `qconv_decide_out_dtype, qconv_prepack_xpu, run_pointwise, run_pointwise_tensor, run_pointwise_binary, TORCH_CHECK, run_pointwise_binary_tensor, TORCH_LIBRARY_IMPL` 等符号展开。
