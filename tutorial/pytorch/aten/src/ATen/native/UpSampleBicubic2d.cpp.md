# UpSampleBicubic2d.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/UpSampleBicubic2d.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on up sample bicubic 2 d; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 up sample bicubic 2 d；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60
```cpp
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS
#include <ATen/core/Tensor.h>
#include <ATen/Dispatch.h>
#include <ATen/TensorMeta.h>
#include <ATen/native/UpSample.h>
#include <c10/util/irange.h>
#include <ATen/Parallel.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/_upsample_bicubic2d_aa.h>
#include <ATen/ops/_upsample_bicubic2d_aa_backward.h>
#include <ATen/ops/_upsample_bicubic2d_aa_backward_native.h>
#include <ATen/ops/_upsample_bicubic2d_aa_native.h>
#include <ATen/ops/upsample_bicubic2d.h>
#include <ATen/ops/upsample_bicubic2d_backward.h>
#include <ATen/ops/upsample_bicubic2d_backward_native.h>
#include <ATen/ops/upsample_bicubic2d_native.h>
#endif

namespace at::meta {

TORCH_META_FUNC(upsample_bicubic2d) (
  const Tensor& input, IntArrayRef output_size, bool align_corners, std::optional<double> scales_h, std::optional<double> scales_w
) {
  auto full_output_size = native::upsample_2d_common_check(input.sizes(), output_size);

  // Allow for empty batch size but not other dimensions
  TORCH_CHECK(
      input.numel() != 0 || c10::multiply_integers(input.sizes().begin() + 1, input.sizes().end()),
      "Non-empty 4D data tensor expected but got a tensor with sizes ",
      input.sizes());

  set_output_raw_strided(0, full_output_size, {}, input.options().memory_format(input.suggest_memory_format()));
}

TORCH_META_FUNC(upsample_bicubic2d_backward) (
  const Tensor& grad_output,
  IntArrayRef output_size,
  IntArrayRef input_size,
  bool align_corners,
  std::optional<double> scales_h,
  std::optional<double> scales_w
) {
  auto full_output_size = native::upsample_2d_common_check(input_size, output_size);

  TORCH_CHECK(
      grad_output.dim() == 4,
      "Expected grad_output to be a tensor of dimension 4 but got: dimension ", grad_output.dim());

  for (const auto i : c10::irange(4)) {
    TORCH_CHECK(
        grad_output.size(i) == full_output_size[i],
        "Expected grad_output to have the same shape as output;",
        " output.size(", i, ") = ", full_output_size[i],
        " but got grad_output.size(", i, ") = ", grad_output.size(i));
  }
```
- EN: Lines 1-60 pull in 16 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 第 1-60 行引入了 16 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 61-120
```cpp
  set_output_raw_strided(0, input_size, {}, grad_output.options());
}

TORCH_META_FUNC(_upsample_bicubic2d_aa) (
  const Tensor& input, IntArrayRef output_size, bool align_corners, std::optional<double> scales_h, std::optional<double> scales_w
) {
  auto full_output_size = native::upsample_2d_common_check(input.sizes(), output_size);

  // Allow for empty batch size but not other dimensions
  TORCH_CHECK(
      input.numel() != 0 || c10::multiply_integers(input.sizes().begin() + 1, input.sizes().end()),
      "Non-empty 4D data tensor expected but got a tensor with sizes ",
      input.sizes());

  set_output_raw_strided(0, full_output_size, {}, input.options().memory_format(input.suggest_memory_format()));
}

TORCH_META_FUNC(_upsample_bicubic2d_aa_backward) (
  const Tensor& grad_output,
  IntArrayRef output_size,
  IntArrayRef input_size,
  bool align_corners,
  std::optional<double> scales_h,
  std::optional<double> scales_w
) {
  auto full_output_size = native::upsample_2d_common_check(input_size, output_size);

  TORCH_CHECK(
      grad_output.dim() == 4,
      "Expected grad_output to be a tensor of dimension 4 but got: dimension ", grad_output.dim());

  for (const auto i : c10::irange(4)) {
    TORCH_CHECK(
        grad_output.size(i) == full_output_size[i],
        "Expected grad_output to have the same shape as output;",
        " output.size(", i, ") = ", full_output_size[i],
        " but got grad_output.size(", i, ") = ", grad_output.size(i));
  }

  set_output_raw_strided(0, input_size, {}, grad_output.options());
}

} // namespace at::meta
namespace at::native {
namespace {

template <typename scalar_t>
void upsample_bicubic2d_backward_out_frame(
    const scalar_t* odata,
    scalar_t* idata,
    int64_t input_height,
    int64_t input_width,
    int64_t output_height,
    int64_t output_width,
    int64_t nbatch,
    int64_t channels,
    bool align_corners,
    std::optional<double> scales_h,
    std::optional<double> scales_w) {
  channels = channels * nbatch;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 121-180
```cpp
  auto input_slice_size = input_height * input_width;
  auto output_slice_size = output_height * output_width;

  using opmath_t = at::opmath_type<scalar_t>;
  const opmath_t height_scale = area_pixel_compute_scale<opmath_t>(
      input_height, output_height, align_corners, scales_h);
  const opmath_t width_scale = area_pixel_compute_scale<opmath_t>(
      input_width, output_width, align_corners, scales_w);
  at::parallel_for(0, channels, at::internal::GRAIN_SIZE / output_slice_size / 4, [&](int64_t start, int64_t end) {
    opmath_t* acc_data_ptr = nullptr;
    std::unique_ptr<opmath_t[]> buffer_data;
    if constexpr (!std::is_same_v<scalar_t, opmath_t>) {
      buffer_data = std::make_unique<opmath_t[]>(input_slice_size);
      acc_data_ptr = buffer_data.get();
      memset(acc_data_ptr, 0, sizeof(opmath_t) * input_slice_size);
    }
    for (const auto i : c10::irange(start, end)) {
      scalar_t* in = idata + i * input_slice_size;
      const scalar_t* out = odata + i * output_slice_size;
      for (const auto output_y : c10::irange(output_height)) {
        for (const auto output_x : c10::irange(output_width)) {

          const opmath_t real_x = area_pixel_compute_source_index(width_scale, output_x, align_corners, /*cubic=*/true);
          int64_t input_x;
          opmath_t t_x;
          guard_index_and_lambda(real_x, input_width, input_x, t_x);

          const opmath_t real_y = area_pixel_compute_source_index(height_scale, output_y, align_corners, /*cubic=*/true);
          int64_t input_y;
          opmath_t t_y;
          guard_index_and_lambda(real_y, input_height, input_y, t_y);

          std::array<opmath_t, 4> x_coeffs;
          std::array<opmath_t, 4> y_coeffs;

          get_cubic_upsample_coefficients<opmath_t>(x_coeffs.data(), t_x);
          get_cubic_upsample_coefficients<opmath_t>(y_coeffs.data(), t_y);

          opmath_t out_value = out[output_y * output_width + output_x];
          for (const auto ii : c10::irange(4)) {
            for (const auto jj : c10::irange(4)) {
              upsample_increment_value_bounded<opmath_t>(
                  acc_data_ptr == nullptr ? reinterpret_cast<opmath_t*>(in) : acc_data_ptr,
                  input_width,
                  input_height,
                  input_x - 1 + ii,
                  input_y - 1 + jj,
                  out_value * y_coeffs[jj] * x_coeffs[ii]);
            }
          }
        }
      }
      if (acc_data_ptr != nullptr) {
        apply_grad_input(acc_data_ptr, in, input_slice_size);
      }
    }
  });
}

void upsample_bicubic2d_backward_kernel(
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are parallel_for, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 parallel_for，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 181-240
```cpp
    const Tensor& grad_input,
    const Tensor& grad_output_,
    IntArrayRef output_size,
    IntArrayRef input_size,
    bool align_corners,
    std::optional<double> scales_h,
    std::optional<double> scales_w) {

  int64_t output_height = output_size[0];
  int64_t output_width = output_size[1];

  int64_t nbatch = input_size[0];
  int64_t channels = input_size[1];
  int64_t input_height = input_size[2];
  int64_t input_width = input_size[3];

  auto grad_output = grad_output_.contiguous();
  // Special case: input/output same size, just copy
  if (input_height == output_height && input_width == output_width) {
    grad_input.copy_(grad_output);
    return;
  }
  AT_DISPATCH_FLOATING_TYPES_AND2(ScalarType::Half, ScalarType::BFloat16,
      grad_output.scalar_type(), "upsample_bicubic2d_backward", [&] {
        scalar_t* idata = grad_input.mutable_data_ptr<scalar_t>();
        const scalar_t* odata = grad_output.const_data_ptr<scalar_t>();

        upsample_bicubic2d_backward_out_frame<scalar_t>(
            odata,
            idata,
            input_height,
            input_width,
            output_height,
            output_width,
            nbatch,
            channels,
            align_corners,
            scales_h,
            scales_w);
      });
}
} // namespace

TORCH_IMPL_FUNC(upsample_bicubic2d_out_cpu) (
    const Tensor& input,
    IntArrayRef output_size,
    bool align_corners,
    std::optional<double> scales_h,
    std::optional<double> scales_w,
    const Tensor& output
) {
  upsample_bicubic2d_kernel(kCPU, output, input, align_corners, scales_h, scales_w);
}

TORCH_IMPL_FUNC(upsample_bicubic2d_backward_out_cpu) (
    const Tensor& grad_output,
    IntArrayRef output_size,
    IntArrayRef input_size,
    bool align_corners,
    std::optional<double> scales_h,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 241-300
```cpp
    std::optional<double> scales_w,
    const Tensor& grad_input
) {
  grad_input.zero_();
  upsample_bicubic2d_backward_kernel(grad_input, grad_output, output_size, input_size, align_corners, scales_h, scales_w);
}

TORCH_IMPL_FUNC(_upsample_bicubic2d_aa_out_cpu) (
    const Tensor& input,
    IntArrayRef output_size,
    bool align_corners,
    std::optional<double> scales_h,
    std::optional<double> scales_w,
    const Tensor& output
) {
  _upsample_bicubic2d_aa_kernel(kCPU, output, input, align_corners, scales_h, scales_w);
}

TORCH_IMPL_FUNC(_upsample_bicubic2d_aa_backward_out_cpu) (
    const Tensor& grad_output,
    IntArrayRef output_size,
    IntArrayRef input_size,
    bool align_corners,
    std::optional<double> scales_h,
    std::optional<double> scales_w,
    const Tensor& grad_input
) {
  grad_input.zero_();
  _upsample_bicubic2d_aa_backward_kernel(kCPU, grad_input, grad_output, align_corners, scales_h, scales_w);
}

// vec variants

using at::native::upsample::compute_output_size;
using at::native::upsample::get_scale_value;

Tensor upsample_bicubic2d(
    const Tensor& input,
    at::OptionalIntArrayRef output_size,
    bool align_corners,
    std::optional<ArrayRef<double>> scale_factors) {
  auto osize = compute_output_size(input.sizes(), output_size, scale_factors);
  auto scale_h = get_scale_value(scale_factors, 0);
  auto scale_w = get_scale_value(scale_factors, 1);
  return at::upsample_bicubic2d(input, osize, align_corners, scale_h, scale_w);
}

Tensor _upsample_bicubic2d_aa(
    const Tensor& input,
    at::OptionalIntArrayRef output_size,
    bool align_corners,
    std::optional<ArrayRef<double>> scale_factors) {
  auto osize = compute_output_size(input.sizes(), output_size, scale_factors);
  auto scale_h = get_scale_value(scale_factors, 0);
  auto scale_w = get_scale_value(scale_factors, 1);
  return at::_upsample_bicubic2d_aa(input, osize, align_corners, scale_h, scale_w);
}

DEFINE_DISPATCH(upsample_bicubic2d_kernel);
DEFINE_DISPATCH(_upsample_bicubic2d_aa_kernel);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Registration or dispatch macros appear here, wiring the implementation into PyTorch operator lookup and backend dispatch.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这里出现了注册或分发宏，用于把实现接入 PyTorch 的算子查找与后端分发机制。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 301-303
```cpp
DEFINE_DISPATCH(_upsample_bicubic2d_aa_backward_kernel);

} // namespace at::native
```
- EN: Registration or dispatch macros appear here, wiring the implementation into PyTorch operator lookup and backend dispatch.
- CN: 这里出现了注册或分发宏，用于把实现接入 PyTorch 的算子查找与后端分发机制。

## Key Concepts / 关键概念

- EN: Registration/dispatch logic is part of the file structure.
- CN: 文件结构中包含注册/分发逻辑。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Template-based reuse helps share logic across scalar types, layouts, or backends.
- CN: 基于模板的复用帮助在不同标量类型、布局或后端之间共享逻辑。
- EN: Notable symbols: TORCH_META_FUNC, upsample_bicubic2d_backward_out_frame, parallel_for, constexpr, upsample_bicubic2d_backward_kernel, TORCH_IMPL_FUNC, upsample_bicubic2d, _upsample_bicubic2d_aa.
- CN: 重要符号：TORCH_META_FUNC, upsample_bicubic2d_backward_out_frame, parallel_for, constexpr, upsample_bicubic2d_backward_kernel, TORCH_IMPL_FUNC, upsample_bicubic2d, _upsample_bicubic2d_aa。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/core/Tensor.h, ATen/Dispatch.h, ATen/TensorMeta.h, ATen/native/UpSample.h, c10/util/irange.h, ATen/Parallel.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/_upsample_bicubic2d_aa.h, ATen/ops/_upsample_bicubic2d_aa_backward.h, ATen/ops/_upsample_bicubic2d_aa_backward_native.h, ATen/ops/_upsample_bicubic2d_aa_native.h`.
- CN: 主要内部头文件：`ATen/core/Tensor.h, ATen/Dispatch.h, ATen/TensorMeta.h, ATen/native/UpSample.h, c10/util/irange.h, ATen/Parallel.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/_upsample_bicubic2d_aa.h, ATen/ops/_upsample_bicubic2d_aa_backward.h, ATen/ops/_upsample_bicubic2d_aa_backward_native.h, ATen/ops/_upsample_bicubic2d_aa_native.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `TORCH_META_FUNC, upsample_bicubic2d_backward_out_frame, parallel_for, constexpr, upsample_bicubic2d_backward_kernel, TORCH_IMPL_FUNC, upsample_bicubic2d, _upsample_bicubic2d_aa`.
- CN: 实现围绕 `TORCH_META_FUNC, upsample_bicubic2d_backward_out_frame, parallel_for, constexpr, upsample_bicubic2d_backward_kernel, TORCH_IMPL_FUNC, upsample_bicubic2d, _upsample_bicubic2d_aa` 等符号展开。
