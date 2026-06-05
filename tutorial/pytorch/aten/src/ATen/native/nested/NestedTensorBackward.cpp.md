# NestedTensorBackward.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/nested/NestedTensorBackward.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the nested tensor support in PyTorch ATen native code and focuses on nested tensor backward; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的嵌套张量支持，主题聚焦于 nested tensor backward；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#include <ATen/native/nested/NestedTensorMath.h>

#include <ATen/ATen.h>
#include <ATen/AccumulateType.h>
#include <ATen/NamedTensorUtils.h>
#include <ATen/WrapDimUtils.h>
#include <ATen/core/op_registration/op_registration.h>
#include <ATen/native/layer_norm.h>
#include <ATen/NestedTensorImpl.h>
#include <ATen/native/nested/NestedTensorUtils.h>
#include <c10/core/DeviceType.h>

#include <utility>

namespace at::native {

// See Note [nested tensor matmul] in NestedTensorMath.cpp
std::tuple<Tensor, Tensor> matmul_backward_nested(
    const Tensor& grad,
    const Tensor& self,
    const Tensor& other,
    std::array<bool, 2> grad_input_mask) {
  if (!grad.defined()) {
    return std::make_tuple(Tensor(), Tensor());
  }
  Tensor grad_self, grad_other;
  if (grad_input_mask[0]) {
    grad_self = at::matmul(grad, other.transpose(-1, -2));
  }
  if (grad_input_mask[1]) {
```
- EN: Lines 1-30 pull in 11 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 第 1-30 行引入了 11 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 31-60
```cpp
    grad_other = at::matmul(self.transpose(-1, -2), grad);
  }
  return std::make_tuple(std::move(grad_self), std::move(grad_other));
}

std::tuple<Tensor, Tensor, Tensor> nested_linear_backward(
    const Tensor& input,
    const Tensor& grad_output,
    const Tensor& weight,
    std::array<bool, 3> output_mask) {
  if (!grad_output.defined()) {
    return std::tuple<Tensor, Tensor, Tensor>{Tensor(), Tensor(), Tensor()};
  }
  Tensor grad_input, grad_weight, grad_bias;
  auto grad_output_contiguous = grad_output.contiguous();
  auto* nt_grad_output = get_nested_tensor_impl(grad_output_contiguous);
  auto* nt_input = get_nested_tensor_impl(input);
  TORCH_INTERNAL_ASSERT(nt_grad_output != nullptr);
  TORCH_INTERNAL_ASSERT(nt_input != nullptr);
  TORCH_INTERNAL_ASSERT(nested_tensor_impl_is_contiguous(nt_grad_output));
  auto grad_output_buffer = nt_grad_output->get_buffer();
  auto input_buffer = nt_input->get_buffer();

  auto reshaped_grad = grad_output_buffer.reshape({-1, weight.size(0)});

  if (output_mask[0]) {
    auto grad_input_buffer = at::mm(reshaped_grad, weight).view({-1});
    auto grad_input_nt_size = nt_input->get_nested_sizes().clone();
    grad_input = wrap_buffer(grad_input_buffer, grad_input_nt_size);
  }
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 61-90
```cpp
  if (output_mask[1]) {
    grad_weight =
        at::mm(reshaped_grad.t(), input_buffer.reshape({-1, weight.size(1)}));
  }
  if (output_mask[2]) {
    grad_bias = reshaped_grad.sum(0);
  }
  return std::tuple<Tensor, Tensor, Tensor>{grad_input, grad_weight, grad_bias};
}

Tensor nested_softmax_backward(
    const Tensor& grad,
    const Tensor& output,
    int64_t dim,
    ScalarType input_dtype) {
  TORCH_INTERNAL_ASSERT(grad.is_nested(), "Should be nested grad")
  TORCH_INTERNAL_ASSERT(output.is_nested(), "Should be nested output")

  auto output_ptr = get_nested_tensor_impl(output);
  auto grad_ptr = get_nested_tensor_impl(grad);
  int64_t ntensors = output_ptr->size(0);
  if (ntensors == 0) {
    return grad.clone();
  }
  int64_t positive_dim = at::maybe_wrap_dim(dim, output_ptr->dim());

  //  Get the info about the output
  const Tensor &output_buffer = output_ptr->get_buffer(),
               &output_sizemat = output_ptr->get_nested_sizes();
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are TORCH_INTERNAL_ASSERT, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 TORCH_INTERNAL_ASSERT，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 91-120
```cpp
  //  Get the info about the grad
  const Tensor &grad_sizemat = grad_ptr->get_nested_sizes();

  TORCH_INTERNAL_ASSERT(output_sizemat.equal(grad_sizemat));
  Tensor grad_output =
      wrap_buffer(at::empty_like(output_buffer), output_sizemat.clone());

  // Unbind nt into individual tensor slices for calculating the derivative
  std::vector<Tensor> grad_output_unbind{grad_output.unbind()},
      grad_unbind{grad.unbind()}, output_unbind{output.unbind()};

  for(const auto i: c10::irange(ntensors)) {
    at::_softmax_backward_data_out(
        grad_output_unbind[i],
        grad_unbind[i],
        output_unbind[i],
        positive_dim - 1,
        input_dtype);
  }
  return grad_output;

}

// Rudimentary sum backward assuming the conditions in #82387
Tensor _nested_sum_backward_cpu(
  const Tensor& grad,
  const Tensor& nested_self,
  OptionalIntArrayRef opt_dims,
  bool keepdim) {
  auto nt_self = get_nested_tensor_impl(nested_self);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 121-150
```cpp
  auto nt_grad = get_nested_tensor_impl(grad);
  const Tensor& grad_buffer = nt_grad->get_buffer();
  const Tensor& self_buffer = nt_self->get_buffer();
  auto grad_sizes = nt_grad->get_nested_sizes();
  auto self_sizes = nt_self->get_nested_sizes();
  int64_t ntensors = nt_self->size(0);
  const Tensor& self_grad_buffer = self_buffer.new_empty(self_buffer.sizes());

  auto num_segments = at::prod(grad_sizes, -1);
  auto segment_lengths = self_sizes.select(1, -1);

  // This logic assumes for now that
  // (1) all the gradient nested tensors are contiguous
  // (2) the gradient nested tensors are stored contiguously in the buffer
  AT_DISPATCH_ALL_TYPES_AND2(
    ScalarType::Half, ScalarType::BFloat16, self_grad_buffer.scalar_type(), "nested_sum_dim_cpu", [&]() {
    auto* self_grad_data = self_grad_buffer.data_ptr<scalar_t>();
    const auto* output_grad_data = grad_buffer.const_data_ptr<scalar_t>();
    int64_t out_idx = 0, in_idx = 0;
    for (const auto i : c10::irange(ntensors)) {
      int64_t segments = num_segments[i].item<int64_t>();
      int64_t segment_length = segment_lengths[i].item<int64_t>();
      for (int64_t j = 0; j < segments; j++) {
        scalar_t output_grad = output_grad_data[out_idx];
        for (int64_t k = 0; k < segment_length; k++) {
          self_grad_data[in_idx] = output_grad;
          in_idx += 1;
        }
        out_idx += 1;
      }
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are scalar_type, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 scalar_type，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 151-180
```cpp
    }
  });

  return wrap_buffer(self_grad_buffer, self_sizes);

}


Tensor _nested_select_backward_symint(
  const Tensor& grad,
  const Tensor& nested_self,
  int64_t dim,
  c10::SymInt index) {
  auto nt_self = get_nested_tensor_impl(nested_self);
  const Tensor& self_buffer = nt_self->get_buffer();
  const auto self_sizes = nt_self->get_nested_sizes();
  const Tensor& self_grad_buffer = self_buffer.new_zeros(self_buffer.sizes());

  auto nt_grad = wrap_buffer(self_grad_buffer, self_sizes);
  nt_grad.select_symint(dim, std::move(index)).copy_(grad);

  return nt_grad;
}

Tensor gelu_backwards_nested(const Tensor& grad, const Tensor& self, std::string_view approximate){
    auto partial_gelu_backward = [approximate](auto && PH1, auto && PH2) { return at::gelu_backward(std::forward<decltype(PH1)>(PH1), std::forward<decltype(PH2)>(PH2), approximate); };
    return map_nt_binary(grad, self, partial_gelu_backward);
}

// Naming convention for relu
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are gelu_backwards_nested, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 gelu_backwards_nested，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 181-210
```cpp
Tensor threshold_backwards_nested(const Tensor& grad_output, const Tensor& input, const Scalar& threshold){
    auto partial_relu_backward = [threshold](auto && PH1, auto && PH2) { return at::threshold_backward(std::forward<decltype(PH1)>(PH1), std::forward<decltype(PH2)>(PH2), threshold); };
    return map_nt_binary(grad_output, input, partial_relu_backward);
}

// Tensor grad_output, Tensor self, *, Tensor(a!) grad_input) -> Tensor(a!)
Tensor silu_backward_nested(const Tensor& grad_output, const Tensor& self){
    auto partial_silu_backward = [](auto && PH1, auto && PH2) { return at::silu_backward(std::forward<decltype(PH1)>(PH1), std::forward<decltype(PH2)>(PH2)); };
    return map_nt_binary(grad_output, self, partial_silu_backward);
}

std::tuple<Tensor, Tensor, Tensor> layer_norm_backward_nested(
    const Tensor& grad,
    const Tensor& input,
    IntArrayRef normalized_shape,
    const Tensor& mean,
    const Tensor& rstd,
    const std::optional<Tensor>& weight_opt /* optional */,
    const std::optional<Tensor>& bias_opt /*{ optional */,
    std::array<bool, 3> grad_input_mask) {
  TORCH_CHECK_VALUE(weight_opt.has_value() && bias_opt.has_value(), "NestedTensor layer_norm requires weight and bias");
  // For NestedTensors weight and bias are non nested.
  auto* nt_impl_grad = get_nested_tensor_impl(grad);
  auto* nt_impl_input = get_nested_tensor_impl(input);
  const auto& weight = weight_opt.value();
  const auto& bias = bias_opt.value();
  const auto& sizes = nt_impl_input->get_nested_sizes();
  auto M_N = _check_nested_layer_norm_inputs(
      *nt_impl_input, normalized_shape, weight, bias);
  auto M = M_N.first;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are threshold_backwards_nested, Tensor, silu_backward_nested, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 threshold_backwards_nested, Tensor, silu_backward_nested，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 211-240
```cpp
  auto N = M_N.second;

  auto gamma = weight.expect_contiguous();
  auto beta = bias.expect_contiguous();

  Tensor dInput;
  Tensor dgamma;
  Tensor dbeta;
  auto input_buffer = nt_impl_input->get_buffer();
  auto grad_buffer = nt_impl_grad->get_buffer();
  if (grad_input_mask[0]) {
    dInput = at::native::empty_like(
        input_buffer,
        std::nullopt /* dtype */,
        std::nullopt /* layout */,
        std::nullopt /* device */,
        std::nullopt /* pin_memory */,
        at::MemoryFormat::Contiguous);
  } else {
    dInput = at::native::zeros_like(
        input_buffer,
        std::nullopt /* dtype */,
        std::nullopt /* layout */,
        std::nullopt /* device */,
        std::nullopt /* pin_memory */,
        at::MemoryFormat::Contiguous);
  }
  if (grad_input_mask[1]) {
    dgamma = M > 0 ? at::native::empty_like(
                         *gamma,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 241-270
```cpp
                         std::nullopt /* dtype */,
                         std::nullopt /* layout */,
                         std::nullopt /* device */,
                         std::nullopt /* pin_memory */,
                         at::MemoryFormat::Contiguous)
                   : at::native::zeros_like(
                         *gamma,
                         std::nullopt /* dtype */,
                         std::nullopt /* layout */,
                         std::nullopt /* device */,
                         std::nullopt /* pin_memory */,
                         at::MemoryFormat::Contiguous);
  }
  if (grad_input_mask[2]) {
    dbeta = M > 0 ? at::native::empty_like(
                        *beta,
                        std::nullopt /* dtype */,
                        std::nullopt /* layout */,
                        std::nullopt /* device */,
                        std::nullopt /* pin_memory */,
                        at::MemoryFormat::Contiguous)
                  : at::native::zeros_like(
                        *beta,
                        std::nullopt /* dtype */,
                        std::nullopt /* layout */,
                        std::nullopt /* device */,
                        std::nullopt /* pin_memory */,
                        at::MemoryFormat::Contiguous);
  }
  if (M > 0) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 271-288
```cpp
    LayerNormBackwardKernel(
        input_buffer.device().type(),
        grad_buffer,
        input_buffer,
        mean,
        rstd,
        *gamma,
        M,
        N,
        &dInput,
        &dgamma,
        &dbeta);
  }
  return std::make_tuple(
      wrap_buffer(dInput, sizes), std::move(dgamma), std::move(dbeta));
}

} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The segment closes with direct return paths, indicating a compact helper or forwarding routine.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段以直接返回路径收尾，说明它更像是紧凑的辅助函数或转发例程。

## Key Concepts / 关键概念

- EN: Backend focus: nested tensor support.
- CN: 后端重点：嵌套张量支持。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: matmul_backward_nested, nested_linear_backward, nested_softmax_backward, TORCH_INTERNAL_ASSERT, _nested_sum_backward_cpu, AT_DISPATCH_ALL_TYPES_AND2, _nested_select_backward_symint, gelu_backwards_nested.
- CN: 重要符号：matmul_backward_nested, nested_linear_backward, nested_softmax_backward, TORCH_INTERNAL_ASSERT, _nested_sum_backward_cpu, AT_DISPATCH_ALL_TYPES_AND2, _nested_select_backward_symint, gelu_backwards_nested。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/native/nested/NestedTensorMath.h, ATen/ATen.h, ATen/AccumulateType.h, ATen/NamedTensorUtils.h, ATen/WrapDimUtils.h, ATen/core/op_registration/op_registration.h, ATen/native/layer_norm.h, ATen/NestedTensorImpl.h, ATen/native/nested/NestedTensorUtils.h, c10/core/DeviceType.h`.
- CN: 主要内部头文件：`ATen/native/nested/NestedTensorMath.h, ATen/ATen.h, ATen/AccumulateType.h, ATen/NamedTensorUtils.h, ATen/WrapDimUtils.h, ATen/core/op_registration/op_registration.h, ATen/native/layer_norm.h, ATen/NestedTensorImpl.h, ATen/native/nested/NestedTensorUtils.h, c10/core/DeviceType.h`。
- EN: External/system headers: `utility`.
- CN: 外部/系统头文件：`utility`。
- EN: The implementation revolves around symbols such as `matmul_backward_nested, nested_linear_backward, nested_softmax_backward, TORCH_INTERNAL_ASSERT, _nested_sum_backward_cpu, AT_DISPATCH_ALL_TYPES_AND2, _nested_select_backward_symint, gelu_backwards_nested, threshold_backwards_nested, silu_backward_nested`.
- CN: 实现围绕 `matmul_backward_nested, nested_linear_backward, nested_softmax_backward, TORCH_INTERNAL_ASSERT, _nested_sum_backward_cpu, AT_DISPATCH_ALL_TYPES_AND2, _nested_select_backward_symint, gelu_backwards_nested, threshold_backwards_nested, silu_backward_nested` 等符号展开。
