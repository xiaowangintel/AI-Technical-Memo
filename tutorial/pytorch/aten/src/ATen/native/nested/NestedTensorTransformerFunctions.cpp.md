# NestedTensorTransformerFunctions.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/nested/NestedTensorTransformerFunctions.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the nested tensor support in PyTorch ATen native code and focuses on nested tensor transformer functions; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的嵌套张量支持，主题聚焦于 nested tensor transformer functions；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60
```cpp
#include <ATen/native/nested/NestedTensorTransformerFunctions.h>

#include <ATen/ATen.h>
#include <ATen/NativeFunctions.h>
#include <ATen/NestedTensorImpl.h>
#include <ATen/native/nested/NestedTensorUtils.h>

#include <c10/util/Exception.h>
#include <cmath>
#include <limits>
#include <optional>
#include <string_view>

namespace at::native {
namespace {

inline void check_nested_tensor_matrix_constraints(
    const Tensor& nested_tensor,
    const Tensor& dense_matrix,
    std::string_view caller) {
  auto* nt_input = get_nested_tensor_impl(nested_tensor);
  TORCH_INTERNAL_ASSERT(nt_input != nullptr);
  TORCH_CHECK(
      !dense_matrix.is_nested(),
      caller,
      " does not support nested weight when input is a nested tensor.")
  // TODO: support noncontiguous case
  // error out for now
  TORCH_CHECK(
      nested_tensor_impl_is_contiguous(nt_input),
      "for now linear only supports contiguous nested tensor");
  TORCH_CHECK(
      nested_tensor.dim() == 3 && dense_matrix.dim() == 2,
      caller,
      " requires nested_tensor.dim == 3 and dense_matrix.dim == 2."
      " Nested tensor dim: ",
      nested_tensor.dim(),
      ". Dense tensor dim: ",
      dense_matrix.dim());
  const auto last_dim = get_consistent_last_dim_of_nested_tensor(*nt_input);
  // We check check the second dimension for linear because it transposes before matrix multiply
  int64_t dim_constraint = (caller == "Linear") ? 1 : 0;
  auto dense_size = dense_matrix.size(dim_constraint);
  TORCH_CHECK(
      last_dim == dense_size,
      "Shape mismatch for NestedTensor ",
      caller,
      ": Expected input's (a nested tensor) 'last_dim' to equal 'weight.size(",
      dim_constraint,
      "),",
      " but got: last_dim = ",
      last_dim,
      ", and weight.size(",
      dim_constraint,
      ") = ",
      dense_size);
}
} // namespace

Tensor nested_linear(
```
- EN: Lines 1-60 pull in 10 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 第 1-60 行引入了 10 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 61-120
```cpp
    const Tensor& input,
    const Tensor& weight,
    const std::optional<Tensor>& bias_opt) {
  check_nested_tensor_matrix_constraints(input, weight, std::string_view{"Linear"});
  auto* nt_input = get_nested_tensor_impl(input);
  const Tensor& input_buffer = nt_input->get_buffer();
  Tensor result_buffer =
      at::linear(input_buffer.reshape({-1, weight.size(1)}), weight, bias_opt);
  result_buffer = result_buffer.reshape({-1});
  int64_t weight_size_1 = weight.size(0);
  Tensor new_sizes = nt_input->get_nested_sizes().clone();
  // Now the last entry in every row of new_sizes should be weight_size_1.
  new_sizes.index_put_({at::indexing::Slice(), -1}, weight_size_1);
  return wrap_buffer(result_buffer, new_sizes);
}

Tensor NestedTensor_matmul(const Tensor& self, const Tensor& other) {
  check_nested_tensor_matrix_constraints(self, other, std::string_view{"Matmul"});
  auto* nt_self = get_nested_tensor_impl_or_null(self);
  const Tensor& self_buffer = nt_self->get_buffer();
  Tensor result_buffer =
      at::mm(self_buffer.reshape({-1, other.sizes()[0]}), other);
  result_buffer = result_buffer.reshape({-1});
  int64_t other_size_1 = other.sizes()[1];
  Tensor new_sizes = nt_self->get_nested_sizes().clone();
  // Now the last entry in every row of new_sizes should be other_size_1.
  new_sizes.index_put_({at::indexing::Slice(), -1}, other_size_1);
  return wrap_buffer(result_buffer, new_sizes);
}

Tensor NestedTensor_times_Tensor_plus_Tensor_addmm(
    const Tensor& self,
    const Tensor& mat1,
    const Tensor& mat2,
    const c10::Scalar& beta,
    const c10::Scalar& alpha,
    std::optional<bool> use_gelu) {
  // Interesting case: alpha * NT * T + beta * T
  const auto* nt_mat1 = get_nested_tensor_impl_or_null(mat1);
  TORCH_INTERNAL_ASSERT(nt_mat1 != nullptr);
  TORCH_INTERNAL_ASSERT(!mat2.is_nested());
  TORCH_INTERNAL_ASSERT(!self.is_nested());
  TORCH_INTERNAL_ASSERT(nested_tensor_impl_is_contiguous(nt_mat1));
  TORCH_INTERNAL_ASSERT(mat1.dim() == 3 && mat2.dim() == 2);
  TORCH_INTERNAL_ASSERT(
      get_consistent_last_dim_of_nested_tensor(*nt_mat1) == mat2.sizes()[0]);
  const Tensor& mat1_buffer = nt_mat1->get_buffer();
  Tensor result_buffer = !use_gelu.has_value()
      ? at::addmm(
            self, mat1_buffer.reshape({-1, mat2.sizes()[0]}), mat2, beta, alpha)
      : at::_addmm_activation(
            self,
            mat1_buffer.reshape({-1, mat2.sizes()[0]}),
            mat2,
            beta,
            alpha,
            *use_gelu);
  result_buffer = result_buffer.reshape({-1});
  int64_t other_size_1 = mat2.sizes()[1];
  Tensor new_sizes = nt_mat1->get_nested_sizes().clone();
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are NestedTensor_matmul, has_value, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 NestedTensor_matmul, has_value，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 121-180
```cpp
  new_sizes.index_put_({at::indexing::Slice(), -1}, other_size_1);
  return at::detail::make_tensor<NestedTensorImpl>(
      std::move(result_buffer), std::move(new_sizes));
}

Tensor NestedTensor_add_NestedTensor_in_place(
    const Tensor& self,
    const Tensor& other) {
  TORCH_INTERNAL_ASSERT(self.is_nested() && other.is_nested());
  const auto& nt_self = *get_nested_tensor_impl(self);
  const auto& nt_other = *get_nested_tensor_impl(other);

  const auto& self_sizes = nt_self.get_nested_sizes();
  const auto& other_sizes = nt_other.get_nested_sizes();

  TORCH_CHECK(at::equal(self_sizes, other_sizes));
  TORCH_INTERNAL_ASSERT(
      nested_tensor_impl_is_contiguous(&nt_self) &&
      nested_tensor_impl_is_contiguous(&nt_other));
  nt_self.get_buffer().view({-1}).add_(nt_other.get_buffer().view({-1}));
  return self;
}

Tensor NestedTensor_softmax_dropout(const Tensor& self, const Tensor& query) {
  const auto* query_nt = get_nested_tensor_impl_or_null(query);
  TORCH_INTERNAL_ASSERT(query_nt != nullptr);
  TORCH_INTERNAL_ASSERT(nested_tensor_impl_is_contiguous(query_nt));

  const Tensor& sizes = query_nt->get_nested_sizes();
  const auto num_tensors = sizes.sizes()[0];

  auto output = at::empty_like(self,{}, at::MemoryFormat::Contiguous);
  TORCH_INTERNAL_ASSERT(output.is_contiguous());

  const auto max_seq_len = self.sizes()[2];

  for (int64_t i = 0; i < num_tensors; i++) {
    auto seq_len = sizes.index({i, 0}).item<int64_t>();
    auto subseq = self.index(
        {i,
         indexing::Slice(),
         indexing::Slice(0, seq_len),
         indexing::Slice(0, seq_len)});
    auto subscores = at::softmax(subseq, subseq.dim() - 1);
    output.index_put_(
        {i,
         indexing::Slice(),
         indexing::Slice(0, seq_len),
         indexing::Slice(0, seq_len)},
        subscores);
    output.index_put_(
        {i,
         indexing::Slice(),
         indexing::Slice(0, seq_len),
         indexing::Slice(seq_len, max_seq_len)},
        0);
    output.index_put_(
        {i,
         indexing::Slice(),
         indexing::Slice(seq_len, max_seq_len),
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are NestedTensor_softmax_dropout, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 NestedTensor_softmax_dropout，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 181-240
```cpp
         indexing::Slice(0, max_seq_len)},
        0);
  }
  return output;
}

Tensor NestedTensor_softmax_dropout_cuda(const Tensor& self, const Tensor& query) {
  std::optional<Tensor> attn_mask;

  attn_mask = NestedTensor_to_mask(query, 2, self.size(2));
  attn_mask = attn_mask->to(query.device(), /*non-blocking=*/true);
  return _masked_softmax(self, *attn_mask, self.dim() - 1, /*mask type */ 1 );  // NestedTensor_to_mask produces a BxT mask
}

Tensor NestedTensor_batch_offsets_from_size_tensor(
    const Tensor& sizes,
    int64_t extra_elements) {
  const int64_t* const sizes_ptr = sizes.const_data_ptr<int64_t>();
  Tensor offsets = at::empty({1 + sizes.size(0) + extra_elements}, at::kInt);
  int32_t* const offsets_ptr = offsets.mutable_data_ptr<int32_t>();
  offsets_ptr[0] = 0;
  const auto sizes_size_1 = sizes.size(1);
  const auto sizes_size_0 = sizes.size(0);
  for (const auto i : c10::irange(sizes_size_0)) {
    int64_t prod = 1;
    for (const auto j : c10::irange(sizes_size_1)) {
      prod *= sizes_ptr[i * sizes_size_1 + j];
    }
    offsets_ptr[i + 1] = offsets_ptr[i] + static_cast<int32_t>(prod);
  }
  return offsets;
}


Tensor NestedTensor_to_mask(const Tensor& nt, std::optional<int64_t> mask_dim, std::optional<int64_t> mask_dim_length) {
  auto* nt_impl = get_nested_tensor_impl(nt);
  TORCH_CHECK(nested_tensor_impl_is_contiguous(nt_impl), "to_mask only works on contiguous NestedTensors.");
  TORCH_CHECK(
      !mask_dim || *mask_dim < nt.dim(),
      "Requested mask dimension ",
      *mask_dim,
      " is bigger than dimension ",
      nt.dim(),
      " of given NestedTensor.");

  // TODO: port optimization for 1x1 tensors from
  // pytorch/nestedtensor's version.

  TORCH_CHECK(
      mask_dim && *mask_dim == 2 && nt.dim() == 3,
      "Only the special case of mask_dim == 2 on a 3-D NestedTensor is supported right now.")
  const auto& sizes = nt_impl->get_nested_sizes();
  // Shape: # of tensors in our NestedTensor by max size along first dim
  // TODO: calculate this without allocating a std::vector.
  const auto result_size_1 = mask_dim_length ? *mask_dim_length : NestedTensor_get_max_size(*nt_impl)[0];
  auto result = at::ones({sizes.sizes()[0], result_size_1}, at::kBool);
  TORCH_INTERNAL_ASSERT_DEBUG_ONLY(sizes.dim() == 2);
  auto* result_data = result.data_ptr<bool>();
  const auto* sizes_ptr = sizes.const_data_ptr<int64_t>();
  const auto sizes_size_1 = sizes.sizes()[1];
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are NestedTensor_softmax_dropout_cuda, NestedTensor_to_mask, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 NestedTensor_softmax_dropout_cuda, NestedTensor_to_mask，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 241-300
```cpp
  for (const auto ii : c10::irange(sizes.sizes()[0])) {
    auto length = sizes_ptr[ii * sizes_size_1];
    for (const auto jj : c10::irange(length)) {
      result_data[ii * result_size_1 + jj] = false;
    }
  }
  return result;
}

Tensor _jagged_to_padded_dense_forward_cpu(
    const Tensor& values,
    TensorList offsets_list,
    c10::IntArrayRef max_lengths,
    const double padding_value) {
  // TODO: Make this kernel more efficient using TensorIterator or something.
  TORCH_INTERNAL_ASSERT(
      offsets_list.size() == 1 && max_lengths.size() == 1,
      "_jagged_to_padded_dense_forward(): only a single jagged dim is supported for now");

  // allocate appropriately-sized padded tensor
  const auto& offsets = offsets_list[0];
  TORCH_CHECK(
      offsets.dim() == 1,
      "_jagged_to_padded_dense_forward(): expected 1D offsets, but got offsets.dim() == ",
      offsets.dim());

  auto batch_size = offsets.size(0) - 1;
  auto max_length = max_lengths[0];
  auto values_shape = values.sizes().vec();
  std::vector<int64_t> padded_shape;
  padded_shape.reserve(values.dim() + 1);
  padded_shape.push_back(batch_size);
  padded_shape.push_back(max_length);
  padded_shape.insert(padded_shape.end(), values_shape.begin() + 1, values_shape.end());

  // We must clamp infinite sentinels to dtype min/max and use the typed value
  // directly to avoid double->int overflow (e.g., int64_max as double overflows).
  Tensor padded;
  AT_DISPATCH_V2(
      values.scalar_type(),
      "_jagged_to_padded_dense_forward_cpu",
      AT_WRAP([&] {
        scalar_t fill_value = _get_padding_value<scalar_t>(padding_value, values.is_floating_point());
        padded = values.new_full(padded_shape, fill_value);
      }),
      AT_EXPAND(AT_ALL_TYPES),
      kBool, kHalf, kBFloat16);

  // copy data to padded tensor
  for (auto i : c10::irange(batch_size)) {
    auto start_offset = offsets[i].item<int64_t>();
    auto end_offset = offsets[i + 1].item<int64_t>();
    auto length = end_offset - start_offset;
    // NB: truncate to max length to match CUDA kernel behavior.
    length = std::min(length, max_length);
    auto source = values.slice(0, start_offset, start_offset + length);
    auto dst = padded.select(0, i).slice(0, 0, length);
    dst.copy_(source);
  }
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 301-360
```cpp
  return padded;
}

Tensor _padded_dense_to_jagged_forward_cpu(
    const Tensor& padded,
    TensorList offsets_list,
    std::optional<int64_t> total_L) {
  // TODO: Make this kernel more efficient using TensorIterator or something.
  TORCH_INTERNAL_ASSERT(
      offsets_list.size() == 1,
      "_padded_dense_to_jagged_forward(): only a single jagged dim is supported for now");

  // allocate appropriately-sized values tensor
  const auto& offsets = offsets_list[0];
  TORCH_CHECK(
      offsets.dim() == 1,
      "_padded_dense_to_jagged_forward(): expected 1D offsets, but got offsets.dim() == ",
      offsets.dim());

  auto final_offset = offsets[-1].item<int64_t>();
  int64_t total_L_val = total_L.has_value() ? (*total_L) : final_offset;
  if (total_L.has_value()) {
    // error if the offsets try to index past the end of the packed dimension
    TORCH_CHECK(
        final_offset == total_L_val,
        "_padded_dense_to_jagged_forward(): final offset should match total_L value");
  }

  TORCH_CHECK(
      padded.dim() >= 2,
      "_padded_dense_to_jagged_forward(): expected padded dim >= 2, but padded.dim() == ",
      padded.dim());

  std::vector<int64_t> values_shape;
  values_shape.reserve(padded.dim() - 1);
  values_shape.push_back(total_L_val);
  auto padded_shape = padded.sizes();
  values_shape.insert(values_shape.end(), padded_shape.begin() + 2, padded_shape.end());
  Tensor values = padded.new_empty(values_shape);

  // copy data to values tensor
  auto batch_size = offsets.size(0) - 1;
  for (auto i : c10::irange(batch_size)) {
    auto start_offset = offsets[i].item<int64_t>();
    auto end_offset = offsets[i + 1].item<int64_t>();
    auto length = end_offset - start_offset;

    TORCH_CHECK(
        length <= padded_shape[1],
        "_padded_dense_to_jagged_forward(): found batch item of length ", length,
        " when max length specified by padded input is ", padded_shape[1]);

    auto dst = values.slice(0, start_offset, end_offset);
    auto source = padded.select(0, i).slice(0, 0, length);
    dst.copy_(source);
  }

  return values;
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 361-361
```cpp
} // namespace at::native
```
- EN: This range provides supporting glue code: declarations, simple forwarding logic, or structural scaffolding reused by nearby operator code.
- CN: 这一段提供了辅助性胶水代码：声明、简单转发逻辑，或被邻近算子代码复用的结构骨架。

## Key Concepts / 关键概念

- EN: Backend focus: nested tensor support.
- CN: 后端重点：嵌套张量支持。
- EN: TensorIterator is used to describe elementwise or reduction-style iteration.
- CN: 使用 TensorIterator 描述逐元素或归约式遍历。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: check_nested_tensor_matrix_constraints, TORCH_CHECK, nested_linear, NestedTensor_matmul, NestedTensor_times_Tensor_plus_Tensor_addmm, NestedTensor_add_NestedTensor_in_place, NestedTensor_softmax_dropout, NestedTensor_softmax_dropout_cuda.
- CN: 重要符号：check_nested_tensor_matrix_constraints, TORCH_CHECK, nested_linear, NestedTensor_matmul, NestedTensor_times_Tensor_plus_Tensor_addmm, NestedTensor_add_NestedTensor_in_place, NestedTensor_softmax_dropout, NestedTensor_softmax_dropout_cuda。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/native/nested/NestedTensorTransformerFunctions.h, ATen/ATen.h, ATen/NativeFunctions.h, ATen/NestedTensorImpl.h, ATen/native/nested/NestedTensorUtils.h, c10/util/Exception.h`.
- CN: 主要内部头文件：`ATen/native/nested/NestedTensorTransformerFunctions.h, ATen/ATen.h, ATen/NativeFunctions.h, ATen/NestedTensorImpl.h, ATen/native/nested/NestedTensorUtils.h, c10/util/Exception.h`。
- EN: External/system headers: `cmath, limits, optional, string_view`.
- CN: 外部/系统头文件：`cmath, limits, optional, string_view`。
- EN: The implementation revolves around symbols such as `check_nested_tensor_matrix_constraints, TORCH_CHECK, nested_linear, NestedTensor_matmul, NestedTensor_times_Tensor_plus_Tensor_addmm, NestedTensor_add_NestedTensor_in_place, NestedTensor_softmax_dropout, NestedTensor_softmax_dropout_cuda, NestedTensor_batch_offsets_from_size_tensor, NestedTensor_to_mask`.
- CN: 实现围绕 `check_nested_tensor_matrix_constraints, TORCH_CHECK, nested_linear, NestedTensor_matmul, NestedTensor_times_Tensor_plus_Tensor_addmm, NestedTensor_add_NestedTensor_in_place, NestedTensor_softmax_dropout, NestedTensor_softmax_dropout_cuda, NestedTensor_batch_offsets_from_size_tensor, NestedTensor_to_mask` 等符号展开。
