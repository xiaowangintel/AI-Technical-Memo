# qlinear.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/mkldnn/xpu/qlinear.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the oneDNN/MKLDNN integration in PyTorch ATen native code and focuses on qlinear; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的oneDNN/MKLDNN 集成，主题聚焦于 qlinear；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#include <torch/library.h>

#include <ATen/native/mkldnn/xpu/detail/oneDNN.h>
#include <ATen/native/mkldnn/xpu/qlinear.h>
#include <c10/core/ScalarType.h>

using namespace at::native::onednn;

namespace at::native::xpu {

inline c10::ScalarType QLinearOnednnXPU::qlinear_decide_out_dtype(
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

Tensor QLinearOnednnXPU::q_linear_pointwise(
    Tensor act,
    double act_scale,
    int64_t act_zero_point,
    Tensor weight,
    Tensor weight_scales,
    Tensor weight_zero_points,
    std::optional<Tensor> bias,
```
- EN: Lines 1-30 pull in 4 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 第 1-30 行引入了 4 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 31-60
```cpp
    double output_scale,
    int64_t output_zero_point,
    std::optional<c10::ScalarType> output_dtype,
    std::string_view post_op_name,
    torch::List<std::optional<at::Scalar>> post_op_args,
    std::string_view post_op_algorithm) {
  Tensor b_raw = bias.has_value() ? bias.value() : at::Tensor();

  const int64_t dim = act.dim();
  TORCH_CHECK(dim == 2, "qliner XPU: input dim should be 2, but got", dim);
  TORCH_CHECK(
      act.device() == weight.device() &&
          act.device() == weight_scales.device() &&
          act.device() == weight_zero_points.device(),
      "qlinear xpu: input tensors(act, weight, weight scale, weight zero-points) should be on the same device");
  int64_t K = act.size(dim - 1);
  int64_t M = act.numel() / K;
  // [M, K] x [K, N]
  int64_t N = weight.size(1);

  std::vector<int64_t> src_dims = {M, K};
  std::vector<int64_t> dst_dims = {M, N};

  auto dst_dtype = qlinear_decide_out_dtype(act, output_dtype);
  Tensor qout = at::empty(dst_dims, act.options().dtype(dst_dtype));

  quantized_matmul(
      act.contiguous(),
      act_scale,
      act_zero_point,
```
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 61-90
```cpp
      weight.contiguous(),
      weight_scales,
      weight_zero_points,
      b_raw,
      qout,
      output_scale,
      output_zero_point,
      output_dtype,
      /*other*/ std::nullopt,
      /*other scale*/ 1.0,
      /*other zp*/ 0,
      /*binary post op*/ "none",
      /*binary alpha*/ 1.0,
      post_op_name,
      post_op_args,
      post_op_algorithm,
      /*m2_trans*/ true);

  return qout;
}

Tensor QLinearOnednnXPU::q_linear_pointwise_tensor(
    Tensor act,
    Tensor act_scale,
    Tensor act_zero_point,
    Tensor weight,
    Tensor weight_scales,
    Tensor weight_zero_points,
    std::optional<Tensor> bias,
    double output_scale,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 91-120
```cpp
    int64_t output_zero_point,
    std::optional<c10::ScalarType> output_dtype,
    std::string_view post_op_name,
    torch::List<std::optional<at::Scalar>> post_op_args,
    std::string_view post_op_algorithm) {
  Tensor b_raw = bias.has_value() ? bias.value() : at::Tensor();

  const int64_t dim = act.dim();
  TORCH_CHECK(dim == 2, "qliner XPU: input dim should be 2, but got", dim);
  TORCH_CHECK(
      act.device() == weight.device() &&
          act.device() == weight_scales.device() &&
          act.device() == weight_zero_points.device(),
      "qlinear xpu: input tensors(act, weight, weight scale, weight zero-points) should be on the same device");
  int64_t K = act.size(dim - 1);
  int64_t M = act.numel() / K;
  // [M, K] x [K, N]
  int64_t N = weight.size(1);

  std::vector<int64_t> src_dims = {M, K};
  std::vector<int64_t> dst_dims = {M, N};

  auto dst_dtype = qlinear_decide_out_dtype(act, output_dtype);
  Tensor qout = at::empty(dst_dims, act.options().dtype(dst_dtype));

  quantized_matmul(
      act.contiguous(),
      act_scale.item().toDouble(),
      act_zero_point.item().toLong(),
      weight.contiguous(),
```
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 121-150
```cpp
      weight_scales,
      weight_zero_points,
      b_raw,
      qout,
      output_scale,
      output_zero_point,
      output_dtype,
      /*other*/ std::nullopt,
      /*other scale*/ 1.0,
      /*other zp*/ 0,
      /*binary post op*/ "none",
      /*binary alpha*/ 1.0,
      post_op_name,
      post_op_args,
      post_op_algorithm,
      /*m2_trans*/ true);

  return qout;
}

Tensor QLinearOnednnXPU::q_linear_pointwise_binary(
    Tensor act,
    double act_scale,
    int64_t act_zero_point,
    Tensor weight,
    Tensor weight_scales,
    Tensor weight_zero_points,
    std::optional<at::Tensor> other,
    std::optional<Tensor> bias,
    double output_scale,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 151-180
```cpp
    int64_t output_zero_point,
    std::optional<c10::ScalarType> output_dtype,
    double other_scale,
    int64_t other_zero_point,
    std::string_view binary_post_op,
    double binary_alpha,
    std::string_view unary_post_op,
    torch::List<std::optional<at::Scalar>> unary_post_op_args,
    std::string_view unary_post_op_algorithm) {
  TORCH_CHECK(
      act.device() == weight.device() &&
          act.device() == weight_scales.device() &&
          act.device() == weight_zero_points.device(),
      "qlinear xpu: input tensors(act, weight, weight scale, weight zero-points) should be on the same device");
  Tensor b_raw = bias.has_value() ? bias.value() : at::Tensor();

  const int64_t dim = act.dim();
  TORCH_CHECK(
      dim == 2 || dim == 3,
      "qliner_pointwise_binary XPU: input dim should be 2 or 3, but got",
      dim);
  int64_t K = act.size(dim - 1);
  int64_t M = act.numel() / K;
  // [M, K] x [K, N]
  int64_t N = weight.size(1);
  Tensor input = dim == 3 ? act.reshape({-1, K}) : act;
  std::vector<int64_t> src_dims = {M, K};
  std::vector<int64_t> dst_dims = {M, N};
  auto dst_dtype = qlinear_decide_out_dtype(act, output_dtype);
  bool has_accum_postop_sum = (binary_post_op == "sum");
```
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 181-210
```cpp
  if (dim == 3) {
    other = other.has_value() ? other.value().reshape({-1, N}) : other;
  }
  Tensor qout = has_accum_postop_sum
      ? other.value()
      : at::empty(dst_dims, act.options().dtype(dst_dtype));
  quantized_matmul(
      input.contiguous(),
      act_scale,
      act_zero_point,
      weight.contiguous(),
      weight_scales,
      weight_zero_points,
      b_raw,
      qout,
      output_scale,
      output_zero_point,
      output_dtype,
      /*other*/ other,
      /*other scale*/ other_scale,
      /*other zp*/ other_zero_point,
      /*binary post op*/ binary_post_op,
      /*binary alpha*/ binary_alpha,
      unary_post_op,
      unary_post_op_args,
      unary_post_op_algorithm,
      /*m2_trans*/ true);

  return dim == 3 ? qout.reshape({act.size(0), -1, N}) : qout;
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are value, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 value，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 211-240
```cpp

Tensor QLinearOnednnXPU::q_linear_pointwise_binary_tensor(
    Tensor act,
    Tensor act_scale,
    Tensor act_zero_point,
    Tensor weight,
    Tensor weight_scales,
    Tensor weight_zero_points,
    std::optional<at::Tensor> other,
    std::optional<Tensor> bias,
    double output_scale,
    int64_t output_zero_point,
    std::optional<c10::ScalarType> output_dtype,
    double other_scale,
    int64_t other_zero_point,
    std::string_view binary_post_op,
    double binary_alpha,
    std::string_view unary_post_op,
    torch::List<std::optional<at::Scalar>> unary_post_op_args,
    std::string_view unary_post_op_algorithm) {
  return q_linear_pointwise_binary(
      act,
      act_scale.item().toDouble(),
      act_zero_point.item().toLong(),
      weight,
      weight_scales,
      weight_zero_points,
      other,
      bias,
      output_scale,
```
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- EN: The segment closes with direct return paths, indicating a compact helper or forwarding routine.
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。
- CN: 这一段以直接返回路径收尾，说明它更像是紧凑的辅助函数或转发例程。

### Lines 241-270
```cpp
      output_zero_point,
      output_dtype,
      other_scale,
      other_zero_point,
      binary_post_op,
      binary_alpha,
      unary_post_op,
      unary_post_op_args,
      unary_post_op_algorithm);
}

Tensor QLinearOnednnXPU::q_linear_prepack_onednn(
    at::Tensor weight,
    std::optional<torch::List<int64_t>> input_shape) {
  at::Tensor weight_transposed = weight.transpose(0, 1);
  return weight_transposed;
}

TORCH_LIBRARY_IMPL(onednn, XPU, m) {
  m.impl(
      TORCH_SELECTIVE_NAME("onednn::qlinear_pointwise"),
      TORCH_FN(QLinearOnednnXPU::q_linear_pointwise));
  m.impl(
      TORCH_SELECTIVE_NAME("onednn::qlinear_pointwise.tensor"),
      TORCH_FN(QLinearOnednnXPU::q_linear_pointwise_tensor));
  m.impl(
      TORCH_SELECTIVE_NAME("onednn::qlinear_prepack"),
      TORCH_FN(QLinearOnednnXPU::q_linear_prepack_onednn));
  m.impl(
      TORCH_SELECTIVE_NAME("onednn::qlinear_pointwise.binary"),
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are TORCH_LIBRARY_IMPL, concentrating a specific part of the operator behavior.
- EN: Registration or dispatch macros appear here, wiring the implementation into PyTorch operator lookup and backend dispatch.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 TORCH_LIBRARY_IMPL，它们承载了某一部分算子行为的核心逻辑。
- CN: 这里出现了注册或分发宏，用于把实现接入 PyTorch 的算子查找与后端分发机制。

### Lines 271-277
```cpp
      TORCH_FN(QLinearOnednnXPU::q_linear_pointwise_binary));
  m.impl(
      TORCH_SELECTIVE_NAME("onednn::qlinear_pointwise.binary_tensor"),
      TORCH_FN(QLinearOnednnXPU::q_linear_pointwise_binary_tensor));
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
- EN: Notable symbols: qlinear_decide_out_dtype, q_linear_pointwise, q_linear_pointwise_tensor, q_linear_pointwise_binary, q_linear_pointwise_binary_tensor, q_linear_prepack_onednn, TORCH_LIBRARY_IMPL.
- CN: 重要符号：qlinear_decide_out_dtype, q_linear_pointwise, q_linear_pointwise_tensor, q_linear_pointwise_binary, q_linear_pointwise_binary_tensor, q_linear_prepack_onednn, TORCH_LIBRARY_IMPL。

## Dependencies / 依赖关系

- EN: Primary internal headers: `torch/library.h, ATen/native/mkldnn/xpu/detail/oneDNN.h, ATen/native/mkldnn/xpu/qlinear.h, c10/core/ScalarType.h`.
- CN: 主要内部头文件：`torch/library.h, ATen/native/mkldnn/xpu/detail/oneDNN.h, ATen/native/mkldnn/xpu/qlinear.h, c10/core/ScalarType.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `qlinear_decide_out_dtype, q_linear_pointwise, q_linear_pointwise_tensor, q_linear_pointwise_binary, q_linear_pointwise_binary_tensor, q_linear_prepack_onednn, TORCH_LIBRARY_IMPL`.
- CN: 实现围绕 `qlinear_decide_out_dtype, q_linear_pointwise, q_linear_pointwise_tensor, q_linear_pointwise_binary, q_linear_pointwise_binary_tensor, q_linear_prepack_onednn, TORCH_LIBRARY_IMPL` 等符号展开。
