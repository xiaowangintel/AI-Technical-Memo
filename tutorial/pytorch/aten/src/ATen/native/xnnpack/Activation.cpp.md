# Activation.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/xnnpack/Activation.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the XNNPACK integration in PyTorch ATen native code and focuses on activation; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的XNNPACK 集成，主题聚焦于 activation；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#ifdef USE_XNNPACK

#include <ATen/native/xnnpack/Common.h>
#include <ATen/native/xnnpack/Engine.h>
#include <ATen/native/utils/Factory.h>

namespace at::native::xnnpack {


bool use_hardswish(
  const Tensor& input) {
  return xnnpack::available() &&
          (1 <= input.ndimension()) &&
          (input.device().is_cpu()) &&
          (kFloat == input.scalar_type()) &&
          !input.requires_grad() &&
           true;
}

static Tensor& hardswish_impl(Tensor& input, Tensor& output) {
  using namespace internal;
  // Create XNNPACK Subgraph
  xnn_subgraph_t subgraph_ptr = nullptr;
  xnn_status status = xnn_create_subgraph(
    /*external_value_ids=*/2,
    /*flags=*/0,
    &subgraph_ptr);
  TORCH_CHECK(
      status == xnn_status_success,
      "xnn create subgraph failed(", status,")!");
```
- EN: Lines 1-30 pull in 3 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are hardswish_impl, concentrating a specific part of the operator behavior.
- CN: 第 1-30 行引入了 3 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 hardswish_impl，它们承载了某一部分算子行为的核心逻辑。

### Lines 31-60
```cpp
  std::unique_ptr<xnn_subgraph, decltype(&xnn_delete_subgraph)> subgraph(
      subgraph_ptr, &xnn_delete_subgraph);
  uint32_t input_id = XNN_INVALID_VALUE_ID, output_id = XNN_INVALID_VALUE_ID;
  std::vector<size_t> input_output_shape(input.sizes().begin(), input.sizes().end());

  status = xnn_define_tensor_value(
    subgraph_ptr,
    xnn_datatype_fp32,
    input_output_shape.size(),
    input_output_shape.data(),
    nullptr,
    0,
    XNN_VALUE_FLAG_EXTERNAL_INPUT,
    &input_id
  );
  TORCH_CHECK(
      status == xnn_status_success,
      "defining xnn input failed(", status,")!");

  status = xnn_define_tensor_value(
    subgraph_ptr,
    xnn_datatype_fp32,
    input_output_shape.size(),
    input_output_shape.data(),
    nullptr,
    1,
    XNN_VALUE_FLAG_EXTERNAL_OUTPUT,
    &output_id
  );
  TORCH_CHECK(
```
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 61-90
```cpp
      status == xnn_status_success,
      "defining xnn output failed(", status,")!");

  status = xnn_define_unary(
    subgraph_ptr,
    xnn_unary_hardswish,
    nullptr,
    input_id,
    output_id,
    0
  );

  // create runtime
  xnn_runtime_t runtime_ptr = nullptr;
  status = xnn_create_runtime_v2(subgraph_ptr, caffe2::pthreadpool_(), 0, &runtime_ptr);
  TORCH_CHECK(
      status == xnn_status_success,
      "xnn create runtime failed(", status,")!");
  TORCH_CHECK(
      runtime_ptr != nullptr,
      "xnn create runtime failed because runtime_ptr is null");
  std::unique_ptr<xnn_runtime, decltype(&xnn_delete_runtime)> auto_runtime(
      runtime_ptr, &xnn_delete_runtime);

  std::array<xnn_external_value, 2> external = {
    xnn_external_value{input_id, input.data_ptr<float>()},
    xnn_external_value{output_id, output.data_ptr<float>()}};

  status = xnn_setup_runtime(
    runtime_ptr,
```
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 91-120
```cpp
    external.size(),
    external.data());
  TORCH_CHECK(
      status == xnn_status_success,
      "xnn setup runtime failed(", status,")!");
  status = xnn_invoke_runtime(runtime_ptr);
  TORCH_CHECK(
      status == xnn_status_success,
      "xnn invoke runtime failed(", status,")!");

  return output;
}

Tensor hardswish(const Tensor& input) {
  Tensor padded_input = mobile::allocate_padded_contiguous_if_needed(
    input, input.suggest_memory_format());

  Tensor output = mobile::empty_with_tail_padding(
    padded_input.sizes(),
    padded_input.options().dtype(),
    input.suggest_memory_format(),
    padded_input.opt_names());

  hardswish_impl(padded_input, output);
  return output.contiguous(input.suggest_memory_format());
}

Tensor& hardswish_(Tensor& input) {
  Tensor padded_input = mobile::allocate_padded_contiguous_if_needed(
    input, input.suggest_memory_format());
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are hardswish, hardswish_, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 hardswish, hardswish_，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 121-139
```cpp

  // Don't need to allocate output if input is contiguous & already padded
  if (input.data_ptr() == padded_input.data_ptr()) {
    hardswish_impl(input, input);
    return input;
  } else {
    Tensor output = mobile::empty_with_tail_padding(
      padded_input.sizes(),
      padded_input.options().dtype(),
      input.suggest_memory_format(),
      padded_input.opt_names());
    hardswish_impl(padded_input, output);
    return input.copy_(output);
  }
}

} // namespace at::native::xnnpack

#endif /* USE_XNNPACK */
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

## Key Concepts / 关键概念

- EN: Backend focus: XNNPACK integration.
- CN: 后端重点：XNNPACK 集成。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: use_hardswish, hardswish_impl, hardswish, hardswish_.
- CN: 重要符号：use_hardswish, hardswish_impl, hardswish, hardswish_。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/native/xnnpack/Common.h, ATen/native/xnnpack/Engine.h, ATen/native/utils/Factory.h`.
- CN: 主要内部头文件：`ATen/native/xnnpack/Common.h, ATen/native/xnnpack/Engine.h, ATen/native/utils/Factory.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `use_hardswish, hardswish_impl, hardswish, hardswish_`.
- CN: 实现围绕 `use_hardswish, hardswish_impl, hardswish, hardswish_` 等符号展开。
