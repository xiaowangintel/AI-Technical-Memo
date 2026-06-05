# MaxPooling.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/xnnpack/MaxPooling.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the XNNPACK integration in PyTorch ATen native code and focuses on max pooling; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的XNNPACK 集成，主题聚焦于 max pooling；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#ifdef USE_XNNPACK

#include <ATen/native/Pool.h>
#include <ATen/native/utils/Factory.h>
#include <ATen/native/xnnpack/Common.h>
#include <ATen/native/xnnpack/Engine.h>
#include <ATen/native/xnnpack/Pooling.h>

namespace at::native::xnnpack {

// Supports NHWC and NCHW FP32 max pooling with any
//  - kernel size
//  - padding
//  - stride
//  - dilation

bool use_max_pool2d(
    const Tensor& input,
    const IntArrayRef kernel_,
    const IntArrayRef padding_,
    IntArrayRef stride_,
    const IntArrayRef dilation_,
    const bool ceil_mode,
    const float output_min,
    const float output_max) {
  using namespace internal;

  // Make sure we are not dealing with an unorthodox configuration.
  if (kernel_.empty() || padding_.empty() || dilation_.empty()) {
    return false;
```
- EN: Lines 1-30 pull in 5 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 第 1-30 行引入了 5 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 31-60
```cpp
  }

  // Stride can be legitimately empty, in which case it is to be defaulted to kernel size.
  if (stride_.empty()) {
    stride_ = kernel_;
  }

  // Normalize the parameters.
  const internal::pooling::Parameters parameters{
    kernel_,
    padding_,
    stride_,
    dilation_,
  };

  // Here are the list of conditions required for this code path to be taken:
  // * Input must be 4D CPU float tensor with no gradients.
  // * Kernel must be a 2D IntArrayRef containing two positive numbers.
  //   Furthermore, 1x1 kernels are not valid as XNNPACK prohibits their use.
  // * Padding must be a 2D IntArrayRef containing two non-negative numbers.
  // * Stride must be a 2D IntArrayRef containing two positive numbers.
  // * Dilation must be a 2D IntArrayRef containing two positive numbers.
  // * Ceil mode is not supported and must be disabled.
  // * output_max must be greater than output_min.
  //   Namely, setting both output_min and output_max to 0 is not valid usage.
  // * Finally, application of this operator to the input tensor with the given
  //   max pool 2d parameters must result in an output tensor with a valid shape.
  const int64_t pt_outputHeight = pooling_output_shape(
      input.size(Layout::Activation4D::height),
      parameters.kernel[Layout::Parameter::height],
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 61-90
```cpp
      parameters.padding[Layout::Parameter::height],
      parameters.stride[Layout::Parameter::height],
      parameters.dilation[Layout::Parameter::height],
      ceil_mode);
  const int64_t pt_outputWidth = pooling_output_shape(
      input.size(Layout::Activation4D::width),
      parameters.kernel[Layout::Parameter::width],
      parameters.padding[Layout::Parameter::width],
      parameters.stride[Layout::Parameter::width],
      parameters.dilation[Layout::Parameter::width],
      ceil_mode);
  const int64_t xnnpack_outputHeight = pooling_output_shape(
      input.size(Layout::Activation4D::height),
      parameters.kernel[Layout::Parameter::height],
      parameters.padding[Layout::Parameter::height],
      parameters.stride[Layout::Parameter::height],
      parameters.dilation[Layout::Parameter::height],
      false);
  const int64_t xnnpack_outputWidth = pooling_output_shape(
      input.size(Layout::Activation4D::width),
      parameters.kernel[Layout::Parameter::width],
      parameters.padding[Layout::Parameter::width],
      parameters.stride[Layout::Parameter::width],
      parameters.dilation[Layout::Parameter::width],
      false);

  const bool output_size_eq = (pt_outputHeight == xnnpack_outputHeight) &&
    (pt_outputWidth == xnnpack_outputWidth);

  return xnnpack::available() &&
```
- EN: The segment closes with direct return paths, indicating a compact helper or forwarding routine.
- CN: 这一段以直接返回路径收尾，说明它更像是紧凑的辅助函数或转发例程。

### Lines 91-120
```cpp
      // Input
      (4 == input.dim()) &&
      (input.device().is_cpu()) &&
      (kFloat == input.scalar_type()) &&
      !input.requires_grad() &&
      // Kernel
      (2 == parameters.kernel.size()) &&
      (parameters.kernel[Layout::Parameter::height] > 0) &&
      (parameters.kernel[Layout::Parameter::width] > 0) &&
      ((parameters.kernel[Layout::Parameter::height] *
        parameters.kernel[Layout::Parameter::width]) > 1) &&
      // Padding
      (2 == parameters.padding.size()) &&
      (parameters.padding[Layout::Parameter::height] >= 0) &&
      (parameters.padding[Layout::Parameter::width] >= 0) &&
      // Stride
      (2 == parameters.stride.size()) &&
      (parameters.stride[Layout::Parameter::height] > 0) &&
      (parameters.stride[Layout::Parameter::width] > 0) &&
      // Dilation
      (2 == parameters.dilation.size()) &&
      (parameters.dilation[Layout::Parameter::height] > 0) &&
      (parameters.dilation[Layout::Parameter::width] > 0) &&
      // Ceil Mode
      (!ceil_mode || output_size_eq) &&
      // Output Min / Max
      (output_max > output_min) &&
      // Output
      (pooling_output_shape(
        input.size(Layout::Activation4D::height),
```
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 121-150
```cpp
        parameters.kernel[Layout::Parameter::height],
        parameters.padding[Layout::Parameter::height],
        parameters.stride[Layout::Parameter::height],
        parameters.dilation[Layout::Parameter::height],
        ceil_mode) > 0) &&
      (pooling_output_shape(
        input.size(Layout::Activation4D::width),
        parameters.kernel[Layout::Parameter::width],
        parameters.padding[Layout::Parameter::width],
        parameters.stride[Layout::Parameter::width],
        parameters.dilation[Layout::Parameter::width],
        ceil_mode) > 0) &&
      true;
}

Tensor max_pool2d(
    const Tensor& input,
    const IntArrayRef kernel_,
    const IntArrayRef padding_,
    IntArrayRef stride_,
    const IntArrayRef dilation_,
    const bool ceil_mode,
    const float output_min,
    const float output_max) {
  using namespace internal;

  // A call to max_pool2d must have been gated by a call to use_maxpool2d, so
  // the parameters are guaranteed to be valid at this point.  Still, stride can
  // be empty, and the parameters not normalized.
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 151-180
```cpp
  if (stride_.empty()) {
    stride_ = kernel_;
  }

  const internal::pooling::Parameters parameters{
    kernel_,
    padding_,
    stride_,
    dilation_,
  };

  const Tensor input_padded_contig_nhwc =
      mobile::allocate_padded_contiguous_if_needed(
          input,
          MemoryFormat::ChannelsLast);

  Tensor output_padded_contig_nhwc = mobile::empty_with_tail_padding(
      {
        input_padded_contig_nhwc.size(Layout::Activation4D::batch),
        input_padded_contig_nhwc.size(Layout::Activation4D::channels),
        pooling_output_shape(
            input_padded_contig_nhwc.size(Layout::Activation4D::height),
            parameters.kernel[Layout::Parameter::height],
            parameters.padding[Layout::Parameter::height],
            parameters.stride[Layout::Parameter::height],
            parameters.dilation[Layout::Parameter::height],
            ceil_mode),
        pooling_output_shape(
            input_padded_contig_nhwc.size(Layout::Activation4D::width),
            parameters.kernel[Layout::Parameter::width],
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 181-210
```cpp
            parameters.padding[Layout::Parameter::width],
            parameters.stride[Layout::Parameter::width],
            parameters.dilation[Layout::Parameter::width],
            ceil_mode),
      },
      input_padded_contig_nhwc.options().dtype(),
      MemoryFormat::ChannelsLast,
      input_padded_contig_nhwc.opt_names());

  xnn_operator_t max_pool_op{};

  const xnn_status create_status = xnn_create_max_pooling2d_nhwc_f32(
      parameters.padding[Layout::Parameter::height],                  // input_padding_top
      parameters.padding[Layout::Parameter::width],                   // input_padding_right
      parameters.padding[Layout::Parameter::height],                  // input_padding_bottom
      parameters.padding[Layout::Parameter::width],                   // input_padding_left
      parameters.kernel[Layout::Parameter::height],                   // kernel_height
      parameters.kernel[Layout::Parameter::width],                    // kernel_width
      parameters.stride[Layout::Parameter::height],                   // subsampling_height
      parameters.stride[Layout::Parameter::width],                    // subsampling_width
      parameters.dilation[Layout::Parameter::height],                 // dilation_height
      parameters.dilation[Layout::Parameter::width],                  // dilation_width
      output_min,                                                     // output_min
      output_max,                                                     // output_max
      0u,                                                             // flags
      &max_pool_op);                                                  // operator

  Operator max_pool_scoped_op(max_pool_op);

  TORCH_CHECK(
```
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 211-240
```cpp
      xnn_status_success == create_status,
      "xnn_create_max_pooling2d_nhwc_f32 failed!");

  const xnn_status reshape_status = xnn_reshape_max_pooling2d_nhwc_f32(
      max_pool_op,                                                    // operator
      input_padded_contig_nhwc.size(Layout::Activation4D::batch),     // batch_size
      input_padded_contig_nhwc.size(Layout::Activation4D::height),    // input_height
      input_padded_contig_nhwc.size(Layout::Activation4D::width),     // input_width
      input_padded_contig_nhwc.size(Layout::Activation4D::channels),  // channels
      input_padded_contig_nhwc.size(Layout::Activation4D::channels),  // input_pixel_stride - NHWC Contiguous
      output_padded_contig_nhwc.size(Layout::Activation4D::channels), // output_pixel_stride - NHWC Contiguous
      nullptr,                                                        // output_height_out
      nullptr,                                                        // output_width_out
      caffe2::pthreadpool_());                                        // threadpool

  TORCH_CHECK(
    xnn_status_success == reshape_status,
    "xnn_reshape_max_pooling2d_nhwc_f32 failed!");

  const xnn_status setup_status = xnn_setup_max_pooling2d_nhwc_f32(
      max_pool_op,                                                  // operator
      input_padded_contig_nhwc.data_ptr<float>(),                   // input
      output_padded_contig_nhwc.data_ptr<float>());                 // output

  TORCH_CHECK(
      xnn_status_success == setup_status,
      "xnn_setup_max_pooling2d_nhwc_f32 failed!");

  const xnn_status run_status = xnn_run_operator(
      max_pool_op,              // operator
```
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 241-252
```cpp
      caffe2::pthreadpool_());  // threadpool

  TORCH_INTERNAL_ASSERT(
      xnn_status_success == run_status,
      "xnn_run_operator failed!");

  return output_padded_contig_nhwc.contiguous(input.suggest_memory_format());
}

} // namespace at::native::xnnpack

#endif /* USE_XNNPACK */
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

## Key Concepts / 关键概念

- EN: Backend focus: XNNPACK integration.
- CN: 后端重点：XNNPACK 集成。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: use_max_pool2d, max_pool2d.
- CN: 重要符号：use_max_pool2d, max_pool2d。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/native/Pool.h, ATen/native/utils/Factory.h, ATen/native/xnnpack/Common.h, ATen/native/xnnpack/Engine.h, ATen/native/xnnpack/Pooling.h`.
- CN: 主要内部头文件：`ATen/native/Pool.h, ATen/native/utils/Factory.h, ATen/native/xnnpack/Common.h, ATen/native/xnnpack/Engine.h, ATen/native/xnnpack/Pooling.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `use_max_pool2d, max_pool2d`.
- CN: 实现围绕 `use_max_pool2d, max_pool2d` 等符号展开。
