# Linear.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/xnnpack/Linear.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the XNNPACK integration in PyTorch ATen native code and focuses on linear; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的XNNPACK 集成，主题聚焦于 linear；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#ifdef USE_XNNPACK

#include <ATen/native/xnnpack/Common.h>
#include <ATen/native/utils/Factory.h>
#include <ATen/native/xnnpack/Linear.h>

namespace at::native::xnnpack {
namespace internal::linear {

namespace {

// Supports NHWC and NCHW FP32 linear operators.

// TODO: Decouple and improve error handling and messages.
bool available(
    const Tensor& weight,
    const std::optional<Tensor>& bias,
    const float output_min,
    const float output_max) {
         // XNNPACK
  return xnnpack::available() &&
          // Weight
          (2 == weight.ndimension()) &&
          (weight.device().is_cpu()) &&
          (kFloat == weight.scalar_type()) &&
          !weight.requires_grad() &&
          // Bias
          ((bias && bias->defined()) ? ((1 == bias->ndimension()) &&
                                       (bias->device().is_cpu()) &&
                                       (kFloat == bias->scalar_type()) &&
```
- EN: Lines 1-30 pull in 3 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 第 1-30 行引入了 3 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 31-60
```cpp
                                       (weight.size(Layout::Filter::output)) == bias->size(0) &&
                                       !bias->requires_grad())
                                     : true) &&
          // Output Min / Max
          (output_max > output_min) &&
          true;
}

// TODO: Decouple and improve error handling and messages.
bool usable(const Tensor& input) {
         // Input
  return (1 <= input.ndimension()) &&
         (input.device().is_cpu()) &&
         (kFloat == input.scalar_type()) &&
         !input.requires_grad() &&
         true;
}

Tensor create_and_run(
    const Tensor& input,
    const Tensor& weight,
    const Tensor& bias,
    const float output_min,
    const float output_max) {
  return run(
      create(
          weight,
          bias,
          output_min,
          output_max),
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are requires_grad, usable, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 requires_grad, usable，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 61-90
```cpp
      input);
}

} // anonymous namespace

ContextLinear create(
    const Tensor& weight,
    const std::optional<Tensor>& bias,
    const float output_min,
    const float output_max) {
  const Tensor weight_contig = weight.contiguous();

  TORCH_CHECK(
        available(
          weight_contig,
          bias,
          output_min,
          output_max),
      "XNNPACK Linear not available! "
      "Reason: The provided (weight, bias, output_min, output_max) parameters are "
      "either invalid individually or their combination is not supported by XNNPACK.");

  xnn_operator_t linear_op{};

  const xnn_status create_status = xnn_create_fully_connected_nc_f32(
      weight_contig.size(Layout::Filter::input),                        // input_channels
      weight_contig.size(Layout::Filter::output),                       // output_channels
      weight_contig.size(Layout::Filter::input),                        // input_pixel_stride
      weight_contig.size(Layout::Filter::output),                       // output_pixel_stride
      weight_contig.data_ptr<float>(),                                  // kernel
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 91-120
```cpp
      (bias && bias->defined()) ?
          bias->contiguous().data_ptr<float>() :
          nullptr,                                                      // bias
      output_min,                                                     // output_min
      output_max,                                                     // output_max
      0u,                                                             // flags
      nullptr,                                                        // xnn_caches_t
      nullptr,                                                        // xnn_weights_cache_t
      &linear_op);                                                    // operator

  TORCH_CHECK(
      xnn_status_success == create_status,
      "xnn_create_fully_connected_nc_f32 failed!");

  return ContextLinear(
    Operator(linear_op),
    weight_contig.size(Layout::Filter::output)
  );
}

Tensor run(
    const ContextLinear& context,
    const Tensor& input) {
  using namespace internal;

  // For compatibility with aten::linear
  auto ip = input;
  if (input.ndimension() == 1) {
    ip = input.unsqueeze(0);
  }
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are size, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 size，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 121-150
```cpp

  const Tensor padded_input = mobile::allocate_padded_contiguous_if_needed(
      ip, ip.suggest_memory_format());

  TORCH_CHECK(
      usable(padded_input),
      "XNNPACK Linear not usable! "
      "Reason: The provided input tensor is either invalid or unsupported by XNNPACK.");

  const IntArrayRef input_size = padded_input.sizes();
  std::vector<int64_t> output_size(input_size.cbegin(), input_size.cend());
  // NOLINTNEXTLINE(facebook-hte-LocalUncheckedArrayBounds)
  output_size.back() = context.output_channels;

  Tensor output = mobile::empty_with_tail_padding(
      output_size,
      padded_input.options().dtype(),
      padded_input.suggest_memory_format(),
      padded_input.opt_names());

  const xnn_status reshape_status = xnn_reshape_fully_connected_nc_f32(
      context.op.get(),                                   // operator
      Layout::ActivationND::batch(padded_input.sizes()),  // Batch,
      caffe2::pthreadpool_());                            // threadpool

  TORCH_CHECK(
      xnn_status_success == reshape_status,
      "xnn_reshape_fully_connected_nc_f32 failed!");

  const xnn_status setup_status = xnn_setup_fully_connected_nc_f32(
```
- EN: The main callable definitions or declarations in this block are NOLINTNEXTLINE, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段的主要可调用定义或声明包括 NOLINTNEXTLINE，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 151-180
```cpp
      context.op.get(),                                   // operator
      padded_input.data_ptr<float>(),                     // input
      output.data_ptr<float>());                          // output

  TORCH_CHECK(
      xnn_status_success == setup_status,
      "xnn_setup_fully_connected_nc_f32 failed!");

  const xnn_status run_status = xnn_run_operator(
      context.op.get(),         // operator
      caffe2::pthreadpool_());  // threadpool

  TORCH_INTERNAL_ASSERT(
      xnn_status_success == run_status,
      "xnn_run_operator failed!");

  // For compatibility with aten::linear
  if (input.ndimension() == 1) {
      output.squeeze_(0);
  }

  return output;
}

c10::intrusive_ptr<xnnpack::LinearOpContext> createLinearClampPrePackOpContext(
    Tensor weight,
    std::optional<Tensor> bias,
    const std::optional<Scalar>& output_min,
    const std::optional<Scalar>& output_max) {
  return xnnpack::XNNPackLinearOpContext::create_context(
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 181-210
```cpp
      std::move(weight), std::move(bias), output_min, output_max);
}

Tensor linear_clamp_run(
    const Tensor& input,
    const c10::intrusive_ptr<xnnpack::LinearOpContext>& op_context) {
  return op_context->run(input);
}

IValue
unpack_prepacked_sizes_linear(const IValue& ivalue) {
  auto op_context = ivalue.toCustomClass<xnnpack::LinearOpContext>();
  const auto tuple = op_context->unpack();
  const auto& bias = std::get<1>(tuple);
  return IValue(std::make_tuple(
      std::get<0>(tuple).sizes(),
      (bias && bias->defined()) ? at::OptionalIntArrayRef(bias->sizes()) : std::nullopt));
}

} // namespace internal::linear

bool use_linear(
    const Tensor& input,
    const Tensor& weight,
    const Tensor& bias) {
  return internal::linear::available(
            weight,
            bias,
            ContextLinear::kMin,
            ContextLinear::kMax) &&
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are unpack_prepacked_sizes_linear, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 unpack_prepacked_sizes_linear，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 211-228
```cpp
         internal::linear::usable(input);
}

Tensor linear(
    const Tensor& input,
    const Tensor& weight,
    const Tensor& bias) {
  return internal::linear::create_and_run(
      input,
      weight,
      bias,
      ContextLinear::kMin,
      ContextLinear::kMax);
}

} // namespace at::native::xnnpack

#endif /* USE_XNNPACK */
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- EN: The segment closes with direct return paths, indicating a compact helper or forwarding routine.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。
- CN: 这一段以直接返回路径收尾，说明它更像是紧凑的辅助函数或转发例程。

## Key Concepts / 关键概念

- EN: Backend focus: XNNPACK integration.
- CN: 后端重点：XNNPACK 集成。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: available, usable, create_and_run, create, ContextLinear, run, createLinearClampPrePackOpContext, linear_clamp_run.
- CN: 重要符号：available, usable, create_and_run, create, ContextLinear, run, createLinearClampPrePackOpContext, linear_clamp_run。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/native/xnnpack/Common.h, ATen/native/utils/Factory.h, ATen/native/xnnpack/Linear.h`.
- CN: 主要内部头文件：`ATen/native/xnnpack/Common.h, ATen/native/utils/Factory.h, ATen/native/xnnpack/Linear.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `available, usable, create_and_run, create, ContextLinear, run, createLinearClampPrePackOpContext, linear_clamp_run, unpack_prepacked_sizes_linear, use_linear`.
- CN: 实现围绕 `available, usable, create_and_run, create, ContextLinear, run, createLinearClampPrePackOpContext, linear_clamp_run, unpack_prepacked_sizes_linear, use_linear` 等符号展开。
