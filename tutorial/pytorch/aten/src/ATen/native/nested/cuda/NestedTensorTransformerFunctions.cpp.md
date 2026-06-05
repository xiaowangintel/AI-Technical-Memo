# NestedTensorTransformerFunctions.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/nested/cuda/NestedTensorTransformerFunctions.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the nested tensor support in PyTorch ATen native code and focuses on nested tensor transformer functions; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的嵌套张量支持，主题聚焦于 nested tensor transformer functions；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60
```cpp
#include <c10/util/Exception.h>

#include <ATen/ATen.h>
#include <ATen/NestedTensorImpl.h>
#include <ATen/native/NonSymbolicBC.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/_nested_from_padded_native.h>
#include <ATen/ops/narrow_native.h>
#endif

#include <ATen/native/nested/NestedTensorTransformerFunctions.h>
#include <ATen/native/nested/NestedTensorTransformerUtils.h>
#include <ATen/native/nested/NestedTensorMath.h>
#include <ATen/native/nested/NestedTensorUtils.h>
#include <ATen/native/transformers/cuda/sdp_utils.h>

#include <ATen/cuda/CUDAContext.h>
#include <ATen/cuda/CUDAGeneratorImpl.h>
#include <ATen/cuda/CUDAGraphsUtils.cuh>

namespace at::native {
namespace {
int64_t padded_tensor_numel(const Tensor& sizes) {
  const auto sizes_num_rows = sizes.sizes()[0];
  const auto sizes_row_length = sizes.sizes()[1];
  const auto* sizes_data = sizes.const_data_ptr<int64_t>();
  int64_t numel = 0;
  for (const auto row_num : c10::irange(sizes_num_rows)) {
    const auto* row_ptr = sizes_data + row_num * sizes_row_length;
    int64_t prod = 1;
    for (const auto idx : c10::irange(sizes_row_length)) {
      prod *= row_ptr[idx];
    }
    numel += prod;
  }
  return numel;
}
} // namespace
Tensor nested_from_padded_cuda(
    const Tensor& padded,
    const Tensor& sizes,
    bool do_transform_0213) {
  if (padded.dim() > 1 && padded.dim() < 5) {
    // Instead of erroring call the generic version
    if(!(padded.dim() == 4 && do_transform_0213) && !(padded.dim() == 3 && !do_transform_0213)){
      return at::native::nested_from_padded_generic(padded, sizes, do_transform_0213);
    }
    if (padded.dtype() != kFloat && padded.dtype() != kHalf) {
      TORCH_WARN_ONCE(
          "nested_from_padded CUDA kernels only support fp32/fp16; falling "
          "back to slower generic kernel");
      return at::native::nested_from_padded_generic(padded, sizes, do_transform_0213);
    }
    Tensor target_offsets =
        NestedTensor_batch_offsets_from_size_tensor(sizes, 0);
    Tensor padded_sizes_tensor = at::tensor(padded.sizes());
    Tensor output = at::empty({padded_tensor_numel(sizes)}, padded.options());
```
- EN: Lines 1-60 pull in 15 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are padded_tensor_numel, concentrating a specific part of the operator behavior.
- CN: 第 1-60 行引入了 15 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 padded_tensor_numel，它们承载了某一部分算子行为的核心逻辑。

### Lines 61-120
```cpp
    Tensor target_size_sizes = sizes.reshape(-1);

    Tensor metadata =
        at::cat({target_size_sizes, padded_sizes_tensor, target_offsets});
    metadata = metadata.to(at::Device(kCUDA), kInt, true, true);

    auto output_size_ptr = metadata.const_data_ptr<int>();
    auto input_size_ptr = output_size_ptr + target_size_sizes.numel();
    auto offsets_ptr = input_size_ptr + padded_sizes_tensor.numel();

    Tensor padded_contiguous = padded.contiguous();
    if (padded.dtype() == kFloat) {
      if (do_transform_0213) {
        remove_padding_transform0213_kernelLauncher(
            padded_contiguous.const_data_ptr<float>(),
            output.data_ptr<float>(),
            offsets_ptr,
            input_size_ptr,
            output_size_ptr,
            padded_contiguous.dim() - 2,
            padded_contiguous.sizes()[0]);
      } else {
        remove_padding_kernelLauncher(
            padded_contiguous.const_data_ptr<float>(),
            output.data_ptr<float>(),
            offsets_ptr,
            input_size_ptr,
            output_size_ptr,
            padded_contiguous.dim() - 1,
            padded_contiguous.sizes()[0]);
      }
    } else if (padded.dtype() == kHalf) {
      if (do_transform_0213) {
        remove_padding_transform0213_kernelLauncher(
            padded_contiguous.const_data_ptr<c10::Half>(),
            output.data_ptr<c10::Half>(),
            offsets_ptr,
            input_size_ptr,
            output_size_ptr,
            padded_contiguous.dim() - 2,
            padded_contiguous.sizes()[0]);
      } else {
        remove_padding_kernelLauncher(
            padded_contiguous.const_data_ptr<c10::Half>(),
            output.data_ptr<c10::Half>(),
            offsets_ptr,
            input_size_ptr,
            output_size_ptr,
            padded_contiguous.dim() - 1,
            padded_contiguous.sizes()[0]);
      }
    } else {
      TORCH_CHECK(false, "Only support fp32/fp16 for padded input");
    }
    return at::detail::make_tensor<NestedTensorImpl>(std::move(output), sizes);
  } else {
    return at::native::nested_from_padded_generic(padded, sizes);
  }
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 121-180
```cpp
static Tensor batch_offsets_from_efficient_size(const Tensor& ef_sizes) {
  const int64_t* nt_sizes_ptr = ef_sizes.const_data_ptr<int64_t>();
  int64_t ef_sizes_size_0 = ef_sizes.sizes()[0];
  Tensor offsets = at::empty({1 + ef_sizes_size_0}, at::kLong);
  int64_t* offsets_ptr = offsets.mutable_data_ptr<int64_t>();
  offsets_ptr[0] = 0;
  int64_t ef_sizes_size_1 = ef_sizes.sizes()[1];
  for (const auto i : c10::irange(ef_sizes_size_0)) {
    int64_t prod = 1;
    for (const auto j : c10::irange(ef_sizes_size_1)) {
      prod = prod * nt_sizes_ptr[i * ef_sizes_size_1 + j];
    }
    offsets_ptr[i + 1] = offsets_ptr[i] + prod;
  }
  return offsets;
}

Tensor NestedTensor_to_padded_tensor_cuda(
    const Tensor& t,
    double padding,
    OptionalIntArrayRef output_size) {
  TORCH_CHECK(t.numel() > 0, "to_padded_tensor: at least one constituent tensor should have non-zero numel")
  int64_t t_dim = t.dim();
  if (t_dim >= 2 && t_dim <= 4 &&
      (t.dtype() == at::kFloat || t.dtype() == at::kDouble ||
       t.dtype() == at::kHalf)) {
    auto* nt_input = get_nested_tensor_impl(t);
    TORCH_CHECK(
        nested_tensor_impl_is_contiguous(nt_input),
        "for now to_padded_tensor only supports contiguous nested tensor");
    const auto& nt_buffer = nt_input->get_buffer();

    if (t_dim == 3 && nt_input->opt_size(2) && (*nt_input->opt_size(2) > 0) &&
        !(output_size.has_value())) {
      Tensor nt_sizes = nt_input->get_nested_sizes();
      Tensor sizes_dim1 = at::native::narrow_symint(nt_sizes, 1, 0, 1);
      Tensor sizes_dim2 = at::native::narrow_symint(nt_sizes, 1, 1, 1);
      Tensor result = at::detail::make_tensor<NestedTensorImpl>(
          nt_input->get_buffer(), sizes_dim1 * sizes_dim2[0]);
      TORCH_INTERNAL_ASSERT_DEBUG_ONLY(result.dim() == 2);
      result =
          NestedTensor_to_padded_tensor_cuda(result, padding, output_size);
      return result.reshape({result.sizes()[0], -1, *nt_input->opt_size(2)});
    }

    Tensor nt_sizes = nt_input->get_nested_sizes();
    Tensor offsets = batch_offsets_from_efficient_size(nt_sizes);
    auto new_size = NestedTensor_get_max_size(*nt_input);
    new_size.insert(new_size.begin(), nt_sizes.sizes()[0]);

    // Pad output tensor to output_size if provided
    if (output_size.has_value()) {
      auto output_size_ = output_size.value();
      TORCH_CHECK(
          output_size_.size() == new_size.size(),
          "Length of output_size does not match NestedTensor dims. Broadcasting is not supported.");
      for (uint64_t i = 0; i < new_size.size(); i++) {
        TORCH_CHECK(
            output_size_[i] >= new_size[i],
            "Value in output_size is less than NestedTensor padded size. Truncation is not supported.");
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are batch_offsets_from_efficient_size, TORCH_CHECK, dtype, has_value, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 batch_offsets_from_efficient_size, TORCH_CHECK, dtype, has_value，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 181-240
```cpp
        new_size[i] = output_size_[i];
      }
    }

    Tensor output = at::empty(IntArrayRef(new_size), nt_buffer.options());

    int64_t input_dim = nt_sizes.sizes()[1];
    int64_t batch_size = nt_sizes.sizes()[0];
    int64_t output_batch_size = new_size[0];
    // TODO: Remove need for cat here
    at::Tensor metadata = at::cat({offsets, nt_sizes.reshape(-1)});
    metadata = metadata.to(at::Device(kCUDA), at::kInt);

    std::vector<Tensor> split =
        at::split_with_sizes(metadata, {offsets.numel(), nt_sizes.numel()}, 0);

    offsets = split[0];
    nt_sizes = split[1];

    AT_DISPATCH_FLOATING_TYPES_AND_HALF(
        nt_buffer.scalar_type(), "NestedTensor_to_padded_tensor_cuda", [&]() {
          add_padding_kernelLauncher(
              nt_buffer.const_data_ptr<scalar_t>(),
              output.data_ptr<scalar_t>(),
              (scalar_t)(padding),
              offsets.const_data_ptr<int>(),
              nt_sizes.const_data_ptr<int>(),
              input_dim,
              new_size,
              batch_size,
              output_batch_size);
        });
    return output;
  }
  return NestedTensor_to_padded_tensor_generic(t, padding, output_size);
}

std::tuple<
    Tensor,
    Tensor,
    Tensor,
    Tensor,
    c10::SymInt,
    c10::SymInt,
    Tensor,
    Tensor,
    Tensor>
_scaled_dot_product_flash_attention_nestedtensor_cuda(
    const Tensor& query,
    const Tensor& key,
    const Tensor& value,
    double dropout_p,
    bool is_causal,
    bool return_debug_mask,
    std::optional<double> scale) {
  auto [
      query_buffer_reshaped,
      key_buffer_reshaped,
      value_buffer_reshaped,
      cumulative_sequence_length_q,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are scalar_type, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 scalar_type，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 241-300
```cpp
      cumulative_sequence_length_kv,
      max_seqlen_batch_q,
      max_seqlen_batch_kv,
      output_shape] = preprocessing::sdpa_nested_preprocessing(query, key, value);

  auto
      [attention,
       logsumexp,
       philox_seed,
       philox_offset,
       debug_attn_mask] =
      at::_flash_attention_forward(
          query_buffer_reshaped,
          key_buffer_reshaped,
          value_buffer_reshaped,
          cumulative_sequence_length_q,
          cumulative_sequence_length_kv,
          max_seqlen_batch_q,
          max_seqlen_batch_kv,
          dropout_p,
          is_causal,
          return_debug_mask,
          scale,
          std::nullopt,
          std::nullopt);
  // Reshape output to convert nnz to batch_size and seq_len
  attention = wrap_buffer(attention.view(-1), output_shape).transpose(1, 2);
  return std::make_tuple(
      attention,
      logsumexp,
      cumulative_sequence_length_q,
      cumulative_sequence_length_kv,
      max_seqlen_batch_q,
      max_seqlen_batch_kv,
      philox_seed,
      philox_offset,
      debug_attn_mask);
}

std::tuple<Tensor, Tensor, Tensor, Tensor>
_scaled_dot_product_efficient_attention_nestedtensor_cuda(
    const Tensor& query,
    const Tensor& key,
    const Tensor& value,
    const std::optional<at::Tensor>&  attn_bias,
    bool compute_log_sumexp,
    double dropout_p,
    bool is_causal,
    std::optional<double> scale) {
  auto [
      query_buffer_reshaped,
      key_buffer_reshaped,
      value_buffer_reshaped,
      cumulative_sequence_length_q,
      cumulative_sequence_length_kv,
      max_seqlen_batch_q,
      max_seqlen_batch_k,
      output_shape] = preprocessing::sdpa_nested_preprocessing(query, key, value);

  sdp::CustomMaskType custom_mask_type = is_causal
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- EN: The segment closes with direct return paths, indicating a compact helper or forwarding routine.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。
- CN: 这一段以直接返回路径收尾，说明它更像是紧凑的辅助函数或转发例程。

### Lines 301-360
```cpp
      ? sdp::CustomMaskType::CausalFromTopLeft
      : sdp::CustomMaskType::NoCustomMask;

  // See Note [Seed and Offset] for description of seed and offset
  // Although max_seqlen_q, and max_seqlen_batch_kv is returned we drop these values.
  auto [attention, log_sumexp, seed, offset, max_seqlen_q, max_seqlen_batch_kv] = at::_efficient_attention_forward(
      query_buffer_reshaped.unsqueeze(0),
      key_buffer_reshaped.unsqueeze(0),
      value_buffer_reshaped.unsqueeze(0),
      std::nullopt,
      cumulative_sequence_length_q,
      cumulative_sequence_length_kv,
      max_seqlen_batch_q,
      max_seqlen_batch_k,
      dropout_p,
      static_cast<int64_t>(custom_mask_type),
      compute_log_sumexp,
      scale);

  // Reshape output to convert nnz to batch_size and seq_len
  attention = wrap_buffer(attention.view(-1), output_shape).transpose(1, 2);
  return std::make_tuple(std::move(attention), std::move(log_sumexp), std::move(seed), std::move(offset));
}

std::tuple<Tensor, Tensor, Tensor, Tensor, c10::SymInt, c10::SymInt, Tensor, Tensor, Tensor>
_scaled_dot_product_cudnn_attention_nestedtensor_cuda(
    const Tensor& query,
    const Tensor& key,
    const Tensor& value,
    const std::optional<Tensor>& attn_bias,
    bool compute_logsumexp,
    double dropout_p,
    bool is_causal,
    bool return_debug_mask,
    std::optional<double> scale) {

  auto [
      query_buffer_reshaped,
      key_buffer_reshaped,
      value_buffer_reshaped,
      cumulative_sequence_length_q,
      cumulative_sequence_length_kv,
      max_seqlen_batch_q,
      max_seqlen_batch_kv,
      output_shape] = preprocessing::sdpa_nested_preprocessing(query, key, value);
  auto [attention, log_sumexp, ignore1, ignore2, ignore3, ignore4, cudnn_seed, cudnn_offset, ignore5] = at::_cudnn_attention_forward(query_buffer_reshaped, key_buffer_reshaped, value_buffer_reshaped, attn_bias, cumulative_sequence_length_q, cumulative_sequence_length_kv, max_seqlen_batch_q, max_seqlen_batch_kv, compute_logsumexp, dropout_p, is_causal, return_debug_mask, scale);

  attention = wrap_buffer(attention.view(-1), output_shape).transpose(1, 2);
  return std::make_tuple(std::move(attention), std::move(log_sumexp), cumulative_sequence_length_q, cumulative_sequence_length_kv, max_seqlen_batch_q, max_seqlen_batch_kv, std::move(cudnn_seed), std::move(cudnn_offset), Tensor());
}

std::tuple<Tensor, Tensor, Tensor> _scaled_dot_product_cudnn_attention_nestedtensor_backward_cuda(
    const Tensor& grad_out,
    const Tensor& query,
    const Tensor& key,
    const Tensor& value,
    const Tensor& out,
    const Tensor& logsumexp,
    const Tensor& philox_seed,
    const Tensor& philox_offset,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 361-420
```cpp
    const Tensor& attn_bias,
    const Tensor& cum_seq_q,
    const Tensor& cum_seq_k,
    const int64_t max_q,
    const int64_t max_k,
    double dropout_p,
    bool is_causal,
    std::optional<double> scale) {
  if (!grad_out.defined()) {
    return std::make_tuple(Tensor{}, Tensor{}, Tensor{});
  }
  auto [
      grad_out_buffer_reshaped,
      query_buffer_reshaped,
      key_buffer_reshaped,
      value_buffer_reshaped,
      output_buffer_reshaped] =
      preprocessing::sdpa_nested_preprocessing_backward(
          grad_out,
          query,
          key,
          value,
          out,
          cum_seq_q,
          cum_seq_k,
          max_q,
          max_k);

  auto [dq, dk, dv] = at::_cudnn_attention_backward(grad_out_buffer_reshaped,
                                                    query_buffer_reshaped,
                                                    key_buffer_reshaped,
                                                    value_buffer_reshaped,
                                                    output_buffer_reshaped,
                                                    logsumexp,
                                                    philox_seed,
                                                    philox_offset,
                                                    attn_bias,
                                                    cum_seq_q,
                                                    cum_seq_k,
                                                    max_q,
                                                    max_k,
                                                    dropout_p,
                                                    is_causal,
                                                    scale);
  return std::make_tuple(std::move(dq), std::move(dk), std::move(dv));
}


std::tuple<at::Tensor, at::Tensor, at::Tensor> _scaled_dot_product_flash_attention_backward_nested(
    const at::Tensor& grad_out_,
    const at::Tensor& query,
    const at::Tensor& key,
    const at::Tensor& value,
    const at::Tensor& out,
    const at::Tensor& logsumexp,
    const Tensor& cumulative_sequence_length_q,
    const Tensor& cumulative_sequence_length_k,
    const int64_t max_seqlen_batch_q,
    const int64_t max_seqlen_batch_k,
    double dropout_p,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 421-469
```cpp
    bool is_causal,
    const at::Tensor& philox_seed,
    const at::Tensor& philox_offset,
    std::optional<double> scale){
  if (!grad_out_.defined()) {
    return std::make_tuple(Tensor{}, Tensor{}, Tensor{});
  }
  auto [
      grad_out_buffer_reshaped,
      query_buffer_reshaped,
      key_buffer_reshaped,
      value_buffer_reshaped,
      output_buffer_reshaped] =
      preprocessing::sdpa_nested_preprocessing_backward(
          grad_out_,
          query,
          key,
          value,
          out,
          cumulative_sequence_length_q,
          cumulative_sequence_length_k,
          max_seqlen_batch_q,
          max_seqlen_batch_k);

  auto [grad_q, grad_k, grad_v] = at::_flash_attention_backward(
    grad_out_buffer_reshaped,
    query_buffer_reshaped,
    key_buffer_reshaped,
    value_buffer_reshaped,
    output_buffer_reshaped,
    logsumexp,
    cumulative_sequence_length_q,
    cumulative_sequence_length_k,
    max_seqlen_batch_q,
    max_seqlen_batch_k,
    dropout_p,
    is_causal,
    philox_seed,
    philox_offset,
    scale);

  grad_q = wrap_buffer(grad_q.view(-1), query.transpose(1,2)._nested_tensor_size()).transpose(1,2);
  grad_k = wrap_buffer(grad_k.view(-1), key.transpose(1,2)._nested_tensor_size()).transpose(1,2);
  grad_v = wrap_buffer(grad_v.view(-1), value.transpose(1,2)._nested_tensor_size()).transpose(1,2);

  return std::make_tuple(std::move(grad_q), std::move(grad_k), std::move(grad_v));
}

} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

## Key Concepts / 关键概念

- EN: Backend focus: nested tensor support.
- CN: 后端重点：嵌套张量支持。
- EN: Backend focus: CUDA backend.
- CN: 后端重点：CUDA 后端。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: padded_tensor_numel, nested_from_padded_cuda, batch_offsets_from_efficient_size, NestedTensor_to_padded_tensor_cuda, TORCH_CHECK, AT_DISPATCH_FLOATING_TYPES_AND_HALF, _scaled_dot_product_flash_attention_nestedtensor_cuda, _scaled_dot_product_efficient_attention_nestedtensor_cuda.
- CN: 重要符号：padded_tensor_numel, nested_from_padded_cuda, batch_offsets_from_efficient_size, NestedTensor_to_padded_tensor_cuda, TORCH_CHECK, AT_DISPATCH_FLOATING_TYPES_AND_HALF, _scaled_dot_product_flash_attention_nestedtensor_cuda, _scaled_dot_product_efficient_attention_nestedtensor_cuda。

## Dependencies / 依赖关系

- EN: Primary internal headers: `c10/util/Exception.h, ATen/ATen.h, ATen/NestedTensorImpl.h, ATen/native/NonSymbolicBC.h, ATen/NativeFunctions.h, ATen/ops/_nested_from_padded_native.h, ATen/ops/narrow_native.h, ATen/native/nested/NestedTensorTransformerFunctions.h, ATen/native/nested/NestedTensorTransformerUtils.h, ATen/native/nested/NestedTensorMath.h, ATen/native/nested/NestedTensorUtils.h, ATen/native/transformers/cuda/sdp_utils.h`.
- CN: 主要内部头文件：`c10/util/Exception.h, ATen/ATen.h, ATen/NestedTensorImpl.h, ATen/native/NonSymbolicBC.h, ATen/NativeFunctions.h, ATen/ops/_nested_from_padded_native.h, ATen/ops/narrow_native.h, ATen/native/nested/NestedTensorTransformerFunctions.h, ATen/native/nested/NestedTensorTransformerUtils.h, ATen/native/nested/NestedTensorMath.h, ATen/native/nested/NestedTensorUtils.h, ATen/native/transformers/cuda/sdp_utils.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `padded_tensor_numel, nested_from_padded_cuda, batch_offsets_from_efficient_size, NestedTensor_to_padded_tensor_cuda, TORCH_CHECK, AT_DISPATCH_FLOATING_TYPES_AND_HALF, _scaled_dot_product_flash_attention_nestedtensor_cuda, _scaled_dot_product_efficient_attention_nestedtensor_cuda, _scaled_dot_product_cudnn_attention_nestedtensor_cuda, _scaled_dot_product_cudnn_attention_nestedtensor_backward_cuda`.
- CN: 实现围绕 `padded_tensor_numel, nested_from_padded_cuda, batch_offsets_from_efficient_size, NestedTensor_to_padded_tensor_cuda, TORCH_CHECK, AT_DISPATCH_FLOATING_TYPES_AND_HALF, _scaled_dot_product_flash_attention_nestedtensor_cuda, _scaled_dot_product_efficient_attention_nestedtensor_cuda, _scaled_dot_product_cudnn_attention_nestedtensor_cuda, _scaled_dot_product_cudnn_attention_nestedtensor_backward_cuda` 等符号展开。
