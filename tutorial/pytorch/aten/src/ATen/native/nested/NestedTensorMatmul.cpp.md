# NestedTensorMatmul.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/nested/NestedTensorMatmul.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the nested tensor support in PyTorch ATen native code and focuses on nested tensor matmul; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的嵌套张量支持，主题聚焦于 nested tensor matmul；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60
```cpp
#include <ATen/native/nested/NestedTensorMath.h>
#include <ATen/native/nested/NestedTensorUtils.h>

#include <ATen/AccumulateType.h>
#include <ATen/Dispatch.h>
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#include <ATen/NestedTensorImpl.h>
#include <ATen/ScalarOps.h>
#include <ATen/TensorIndexing.h>
#include <ATen/TensorOperators.h>
#include <ATen/TensorUtils.h>
#include <ATen/core/Tensor.h>
#include <ATen/core/grad_mode.h>

namespace at::native {

Tensor bmm_nested(const Tensor& self, const Tensor& mat2) {
  TORCH_CHECK(self.dim() == 3, "batch1 must be a 3D tensor");
  TORCH_CHECK(mat2.dim() == 3, "batch2 must be a 3D tensor");

  int64_t ntensors = self.is_nested() ? get_nested_tensor_impl(self)->size(0) : self.size(0);
  int64_t ntensors2 = mat2.is_nested() ? get_nested_tensor_impl(mat2)->size(0) : mat2.size(0);

  TORCH_CHECK(ntensors == ntensors2,
      "Expected size for the 1st dimension of batch2 tensor to be: ", ntensors,
      " but got: ", ntensors2, ".");

  const Tensor& self_buffer = self.is_nested() ? get_nested_tensor_impl(self)->get_unsafe_storage_as_tensor() : self;
  const Tensor& mat2_buffer = mat2.is_nested() ? get_nested_tensor_impl(mat2)->get_unsafe_storage_as_tensor() : mat2;


  // create a contiguous output
  int64_t out_numel = 0;
  const Tensor& self_sizemat = self.is_nested() ?
      get_nested_tensor_impl(self)->get_nested_sizes() : get_nested_tensor_impl(mat2)->get_nested_sizes();

  Tensor out_sizemat = self_sizemat.new_empty(self_sizemat.sizes());
  int64_t* out_sizemat_ptr = out_sizemat.data_ptr<int64_t>();
  for (int64_t i = 0; i < ntensors; i++) {
    const IntArrayRef& self_shape = get_size_for_index(self, i);
    const IntArrayRef& mat2_shape = get_size_for_index(mat2, i);
    const int64_t& self_size0 = self_shape[0], & self_size1 = self_shape[1],
        & mat2_size0 = mat2_shape[0], & mat2_size1 = mat2_shape[1];
    TORCH_CHECK(self_size1 == mat2_size0,
        i, "-th nested matrices in batch cannot be multiplied (",
        self_size0, "x", self_size1, " and ",
        mat2_size0, "x", mat2_size1, ")");
    out_sizemat_ptr[0] = self_size0;
    out_sizemat_ptr[1] = mat2_size1;
    out_sizemat_ptr += 2;
    out_numel += self_size0 * mat2_size1;
  }
  Tensor out_buffer = self.is_nested() ? self_buffer.new_empty(out_numel) : mat2_buffer.new_empty(out_numel);
  Tensor output = wrap_buffer(out_buffer, out_sizemat);
  // call tensor mm
  // TODO: `padding nested tensor -> bmm -> remove padding` may be more efficient
  //       until we have specialized nested tensor bmm kernel
  //       useful resource: `aten/src/ATen/native/cpu/LinearAlgebra.cpp/bmm_out_or_baddbmm_`
  //                        `aten/src/ATen/native/cuda/Blas.cpp/baddbmm_out_cuda_impl`
```
- EN: Lines 1-60 pull in 13 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are bmm_nested, concentrating a specific part of the operator behavior.
- CN: 第 1-60 行引入了 13 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 bmm_nested，它们承载了某一部分算子行为的核心逻辑。

### Lines 61-120
```cpp
  std::vector<Tensor> output_unbind = output.unbind();
  for (int64_t i = 0; i < ntensors; i++) {
    at::mm_out(output_unbind[i],
              self_buffer.as_strided(get_size_for_index(self, i), get_stride_for_index(self, i), get_offset_for_index(self, i)),
              mat2_buffer.as_strided(get_size_for_index(mat2, i), get_stride_for_index(mat2, i), get_offset_for_index(mat2, i)));
  }
  return output;
}



static Tensor matmul_with_bmm_nested(const Tensor& self, const Tensor& mat2) {
  // Tensor self = self_.contiguous();
  // Tensor mat2 = mat2_.contiguous();
  // self [N, n_heads, *, head_dim]
  // mat2 [N, n_heads, head_dim, *]
  const auto self_ptr = get_nested_tensor_impl(self);
  const auto mat2_ptr = get_nested_tensor_impl(mat2);
  // metadata for self
  std::vector<IntArrayRef> self_sizes = NestedTensor_get_sizes(self_ptr);
  std::vector<IntArrayRef> self_strides = NestedTensor_get_strides(self_ptr);
  const int64_t* self_offsets_ptr =
      self_ptr->get_storage_offsets().const_data_ptr<int64_t>();
  auto opt = self_ptr->get_nested_sizes().options();

  // metadata for mat2
  std::vector<IntArrayRef> mat2_sizes = NestedTensor_get_sizes(mat2_ptr);
  std::vector<IntArrayRef> mat2_strides = NestedTensor_get_strides(mat2_ptr);
  const int64_t* mat2_offsets_ptr =
      mat2_ptr->get_storage_offsets().const_data_ptr<int64_t>();
  auto opt2 = mat2_ptr->get_nested_sizes().options();

  int64_t N = static_cast<int64_t>(self_sizes.size());
  int64_t n_heads = self_sizes[0][0];

  // viewed metadata for self
  auto self_new_sizes = at::empty({N * n_heads, 2}, opt);
  int64_t* self_new_sizes_ptr = self_new_sizes.mutable_data_ptr<int64_t>();

  auto self_new_strides = at::empty({N * n_heads, 2}, opt);
  int64_t* self_new_strides_ptr = self_new_strides.mutable_data_ptr<int64_t>();
  auto self_new_offsets = at::empty({N * n_heads}, opt);
  int64_t* self_new_offsets_ptr = self_new_offsets.mutable_data_ptr<int64_t>();

  // viewed metadata for mat2
  auto mat2_new_sizes = at::empty({N * n_heads, 2}, opt2);
  int64_t* mat2_new_sizes_ptr = mat2_new_sizes.mutable_data_ptr<int64_t>();

  auto mat2_new_strides = at::empty({N * n_heads, 2}, opt2);
  int64_t* mat2_new_strides_ptr = mat2_new_strides.mutable_data_ptr<int64_t>();
  auto mat2_new_offsets = at::empty({N * n_heads}, opt);
  int64_t* mat2_new_offsets_ptr = mat2_new_offsets.mutable_data_ptr<int64_t>();

  for (int64_t i = 0; i < N; i++) {
    const IntArrayRef& self_size_i = self_sizes[i];
    const IntArrayRef& self_stride_i = self_strides[i];
    int64_t self_offset = self_offsets_ptr[i];

    const IntArrayRef& mat2_size_i = mat2_sizes[i];
    const IntArrayRef& mat2_stride_i = mat2_strides[i];
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are matmul_with_bmm_nested, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 matmul_with_bmm_nested，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 121-180
```cpp
    int64_t mat2_offset = mat2_offsets_ptr[i];
    for (int64_t j = 0; j < n_heads; j++) {
      auto idx = (i * n_heads + j) * 2;
      self_new_sizes_ptr[idx] = self_size_i[1];
      self_new_sizes_ptr[idx + 1] = self_size_i[2];
      self_new_strides_ptr[idx] = self_stride_i[1];
      self_new_strides_ptr[idx + 1] = self_stride_i[2];
      auto offset_idx = i * n_heads + j;
      self_new_offsets_ptr[offset_idx] = self_offset;
      self_offset += self_stride_i[0];

      mat2_new_sizes_ptr[idx] = mat2_size_i[1];
      mat2_new_sizes_ptr[idx + 1] = mat2_size_i[2];
      mat2_new_strides_ptr[idx] = mat2_stride_i[1];
      mat2_new_strides_ptr[idx + 1] = mat2_stride_i[2];
      mat2_new_offsets_ptr[offset_idx] = mat2_offset;
      mat2_offset += mat2_stride_i[0];
    }
  }

  // view self as [N * n_heads, *, head_dim] (collapse first 2 dims)
  auto viewed_self = create_nested_view_tensor(
      self, self_new_sizes, self_new_strides, self_new_offsets);

  // view mat2 as [N * n_heads, head_dim, *] (collapse first 2_dims)
  auto viewed_mat2 = create_nested_view_tensor(
      mat2, mat2_new_sizes, mat2_new_strides, mat2_new_offsets);

  // output [N * n_heads, *, *]
  auto bmm_output = at::bmm(viewed_self, viewed_mat2);

  // generate metadata for viewing output as [N, n_heads, *, *]
  // output of bmm should be contiguous so stride calculations should hold
  auto out_new_sizes = at::empty({N, 3}, opt);
  auto out_new_strides = at::empty({N, 3}, opt);
  auto out_new_offsets = at::empty({N}, opt);
  int64_t* out_new_offsets_ptr = out_new_offsets.mutable_data_ptr<int64_t>();

  int64_t* out_new_sizes_ptr = out_new_sizes.data_ptr<int64_t>();
  int64_t* out_new_strides_ptr = out_new_strides.data_ptr<int64_t>();

  int64_t out_offset = 0;
  for (int64_t i = 0; i < N; i++) {
    out_new_offsets_ptr[i] = out_offset;
    const IntArrayRef& self_size_i = self_sizes[i];
    const IntArrayRef& mat2_size_i = mat2_sizes[i];
    auto idx = i * 3;
    out_new_sizes_ptr[idx] = n_heads;
    out_new_sizes_ptr[idx + 1] = self_size_i[1];
    out_new_sizes_ptr[idx + 2] = mat2_size_i[2];
    out_new_strides_ptr[idx] = self_size_i[1] * mat2_size_i[2];
    out_new_strides_ptr[idx + 1] = mat2_size_i[2];
    out_new_strides_ptr[idx + 2] = 1;
    out_offset += n_heads * (self_size_i[1] * mat2_size_i[2]);
  }

  auto viewed_out = create_nested_view_tensor(
      bmm_output, out_new_sizes, out_new_strides, out_new_offsets);

  return viewed_out;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 181-240
```cpp
}

// nt: NT of shape (B, *, C, D)
// other: dense tensor of shape (D, E)
// output: NT of shape (B, *, C, E)
static Tensor matmul_nested_with_broadcasted_dense(
    const Tensor& nt,
    const Tensor& other) {
  // View nt buffer as 3D jagged for matmul
  auto* nt_impl = get_nested_tensor_impl(nt);
  auto jagged = nt_impl->get_buffer().view({-1, nt.size(2), nt.size(3)});
  auto new_buffer = at::matmul(jagged, other);

  // Wrap result into nested tensor
  const auto E = other.size(-1);
  const auto component_dim = nt.dim() - 1;
  auto new_sizes = nt_impl->get_nested_sizes().clone();
  auto new_sizes_ptr = new_sizes.data_ptr<int64_t>();
  for (const auto i : c10::irange(nt.size(0))) {
    new_sizes_ptr[i * component_dim + 2] = E;
  }
  return at::detail::make_tensor<NestedTensorImpl>(
      new_buffer.view(-1), new_sizes);
}

// Note [nested tensor matmul]
// This is really a generalized batched matmul dedicated to nested tensors,
// where `self` and `mat2` have same number (>= 3) of dimensions.
// The last 2 dimensions will be considered as matrix dimensions,
// so they should be matrix-multiplicable.
// The leading dimensions are considered as batch dimensions,
// and since nested tensor does not support broadcasting for now,
// for each batch dimension `self` and `mat2` must have same size.
// TODO: Should make full matmul semantics support some day
Tensor matmul_nested(const Tensor& self, const Tensor& mat2) {
  // special case of NT (B, *, C, D) with broadcasted dense (D, E)
  if (self.is_nested() && self.is_contiguous() && !mat2.is_nested() &&
      self.dim() == 4 && mat2.dim() == 2 &&
      get_nested_tensor_impl(self)->opt_size(2).has_value() &&
      get_nested_tensor_impl(self)->opt_size(3).has_value() &&
      self.size(3) == mat2.size(0)) {
    return matmul_nested_with_broadcasted_dense(self, mat2);
  }
  if (self.is_nested() && !mat2.is_nested()) {
    TORCH_CHECK(false,
        "Expected both to be nested, but got a nested self and non-nested other");
  } else if (!self.is_nested() && mat2.is_nested()) {
    TORCH_CHECK(false,
        "Expected both to be nested, but got a non-nested self and nested other");
  }
  // to_padded_tensor only supports contiguous inputs
  auto self_contig = self.contiguous();
  auto mat2_contig = mat2.contiguous();
  // dispatcher should have guaranteed that at least one is nested
  const auto self_ptr = get_nested_tensor_impl(self_contig);
  const auto mat2_ptr = get_nested_tensor_impl(mat2_contig);
  int64_t self_dim = self_ptr->dim(), mat2_dim = mat2_ptr->dim();
  TORCH_CHECK(
      self_dim >= 3,
      "matmul: For nested tensors, only inputs with >= 3 dims are currently supported. 1st input has rank: ",
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are shape, matmul_nested, NT, size, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 shape, matmul_nested, NT, size，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 241-300
```cpp
      self_dim);
  TORCH_CHECK(
      mat2_dim >= 3,
      "matmul: For nested tensors, only inputs with >= 3 dims are currently supported. 2nd input has rank: ",
      mat2_dim);
  TORCH_CHECK(
      self_dim == mat2_dim, "matmul: both inputs must have the same rank");
  int64_t ntensors = self_ptr->size(0), ntensors2 = mat2_ptr->size(0);
  TORCH_CHECK(
      ntensors == ntensors2,
      "matmul: Expected size for the 1st dimension of 2nd input tensor to be: ",
      ntensors,
      " but got: ",
      ntensors2,
      ".");
  // Ensure batch dimensions have the same sizes (no broadcasting).
  const auto& self_sizes = self_ptr->get_nested_sizes();
  const auto& mat2_sizes = mat2_ptr->get_nested_sizes();
  const auto& self_batch_sizes = self_sizes.narrow(1, 0, self_dim - 3);
  const auto& mat2_batch_sizes = mat2_sizes.narrow(1, 0, mat2_dim - 3);
  TORCH_CHECK(
      at::equal(self_batch_sizes, mat2_batch_sizes),
      "matmul: For nested tensors, batch dimensions must have the same sizes, ",
      "no broadcasting is currently performed. Got batch shapes for self ",
      self_batch_sizes,
      " and batch shapes for mat2 ",
      mat2_batch_sizes);
  // Ensure last dim of self and second last dim of mat2 have the same size
  const auto& self_dim_size = self_sizes.select(1, -1);
  const auto& mat2_dim_size = mat2_sizes.select(1, -2);
  TORCH_CHECK(
      at::equal(self_dim_size, mat2_dim_size),
      "matmul: Nested tensors cannot be matrix multiplied, last dimension of self has sizes",
      self_dim_size,
      "second last dimension of mat2 has sizes",
      mat2_dim_size);

  // use bmm inference-only fast path for [N, n_heads, *, head_dim] [N, n_heads,
  // head_dim, *]
  if (self.is_cuda() && self_dim == 4 && self.is_contiguous() &&
      mat2_dim == 4 && mat2.is_contiguous() &&
      !(GradMode::is_enabled() &&
        (self.requires_grad() || mat2.requires_grad()))) {
    const auto& self_opt_head_dim = self_ptr->opt_size(1);
    const auto& mat2_opt_head_dim = mat2_ptr->opt_size(1);
    if (self_opt_head_dim.has_value() && mat2_opt_head_dim.has_value() &&
        self_opt_head_dim.value() == mat2_opt_head_dim.value()) {
      return matmul_with_bmm_nested(self, mat2);
    }
  }

  // Construct output size from input sizes
  Tensor output_sizes = self_sizes.clone();
  // The last entry in every row of output_sizes should be last column of
  // mat2_sizes
  output_sizes.index_put_(
      {at::indexing::Slice(), -1}, mat2_sizes.select(1, -1).clone());

  auto self_padded = self_contig.to_padded_tensor(0.);
  auto mat2_padded = mat2_contig.to_padded_tensor(0.);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are requires_grad, value, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 requires_grad, value，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 301-330
```cpp
  auto output_padded = at::matmul(self_padded, mat2_padded);
  auto output_nested = nested_from_padded_generic(output_padded, output_sizes);
  return output_nested;
}

Tensor& matmul_out_nested(
    const Tensor& tensor1,
    const Tensor& tensor2,
    Tensor& result) {
  // TODO: this is a very quick and dirty implementation
  //       should improve it to avoid the intermediate memory usage
  Tensor function_result = at::matmul(tensor1, tensor2);
  auto function_result_ptr = get_nested_tensor_impl(function_result);
  // TODO: this is to reproduce function_result_ptr->opt_sizes_
  //       if an accessor is provided in the future, can replace this
  std::vector<int64_t> sizes;
  for (int64_t i = 0; i < function_result_ptr->dim(); i++) {
    std::optional<int64_t> opt_size = function_result_ptr->opt_size(i);
    if (opt_size.has_value()) {
      sizes.push_back(*opt_size);
    } else {
      sizes.push_back(-1);
    }
  }
  result.reshape(sizes);
  result.copy_(function_result);
  return result;
}

} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

## Key Concepts / 关键概念

- EN: Backend focus: nested tensor support.
- CN: 后端重点：嵌套张量支持。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: bmm_nested, matmul_with_bmm_nested, matmul_nested_with_broadcasted_dense, matmul_nested, matmul_out_nested.
- CN: 重要符号：bmm_nested, matmul_with_bmm_nested, matmul_nested_with_broadcasted_dense, matmul_nested, matmul_out_nested。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/native/nested/NestedTensorMath.h, ATen/native/nested/NestedTensorUtils.h, ATen/AccumulateType.h, ATen/Dispatch.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/NestedTensorImpl.h, ATen/ScalarOps.h, ATen/TensorIndexing.h, ATen/TensorOperators.h, ATen/TensorUtils.h, ATen/core/Tensor.h`.
- CN: 主要内部头文件：`ATen/native/nested/NestedTensorMath.h, ATen/native/nested/NestedTensorUtils.h, ATen/AccumulateType.h, ATen/Dispatch.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/NestedTensorImpl.h, ATen/ScalarOps.h, ATen/TensorIndexing.h, ATen/TensorOperators.h, ATen/TensorUtils.h, ATen/core/Tensor.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `bmm_nested, matmul_with_bmm_nested, matmul_nested_with_broadcasted_dense, matmul_nested, matmul_out_nested`.
- CN: 实现围绕 `bmm_nested, matmul_with_bmm_nested, matmul_nested_with_broadcasted_dense, matmul_nested, matmul_out_nested` 等符号展开。
