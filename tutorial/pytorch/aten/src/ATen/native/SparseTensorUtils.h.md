# SparseTensorUtils.h — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/SparseTensorUtils.h`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on sparse tensor utils; it mainly declares interfaces, helper types, and inline utilities.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 sparse tensor utils；其主要作用是声明接口、辅助类型以及内联工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#pragma once

#include <ATen/Parallel.h>
#include <ATen/SparseTensorImpl.h>
#include <ATen/core/Tensor.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#else
#include <ATen/ops/empty.h>
#include <ATen/ops/tensor.h>
#endif

namespace at::sparse {

// Just for documentary purposes
using SparseTensor = Tensor;
using SparseType = Type;

// This is an internal utility function for getting at the SparseTensorImpl,
// so that we can write sparse tensor specific accessors for special fields
// in SparseTensor.  You should only use this for writing low level
// setters/getters for SparseTensorImpl fields; otherwise, you should use
// the low level setters/getters that were implemented using this.
//
// This may be called repeatedly, so make sure it's pretty cheap.
inline SparseTensorImpl* get_sparse_impl(const SparseTensor& self) {
  TORCH_INTERNAL_ASSERT(
      self.is_sparse(), "_internal_get_SparseTensorImpl: not a sparse tensor");
  return static_cast<SparseTensorImpl*>(self.unsafeGetTensorImpl());
```
- EN: Lines 1-30 pull in 6 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are get_sparse_impl, concentrating a specific part of the operator behavior.
- CN: 第 1-30 行引入了 6 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 get_sparse_impl，它们承载了某一部分算子行为的核心逻辑。

### Lines 31-60
```cpp
}

// Takes indices and values and directly puts them into the sparse tensor, no
// copy.  This used to be called THSTensor_(_move)
inline void alias_into_sparse(
    const SparseTensor& self,
    const Tensor& indices,
    const Tensor& values) {
  get_sparse_impl(self)->set_indices_and_values_unsafe(indices, values);
}

// Take indices and values and makes a (data) copy of them to put into the
// sparse indices/values.  This used to be called THSTensor_(_set)
inline void copy_into_sparse(
    const SparseTensor& self,
    const Tensor& indices,
    const Tensor& values,
    bool non_blocking) {
  alias_into_sparse(
      self,
      indices.to(self._indices().options(), non_blocking, /*copy=*/true),
      values.to(self._values().options(), non_blocking, /*copy=*/true));
}

// TODO: put this into the public API
inline bool is_same_tensor(const Tensor& lhs, const Tensor& rhs) {
  return lhs.unsafeGetTensorImpl() == rhs.unsafeGetTensorImpl();
}

inline bool is_same_density(const SparseTensor& self, const SparseTensor& src) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are THSTensor_, is_same_tensor, is_same_density, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 THSTensor_, is_same_tensor, is_same_density，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 61-90
```cpp
  return self.sparse_dim() == src.sparse_dim() &&
      self.dense_dim() == src.dense_dim();
}

// Give us a new values tensor, with the same dimensionality
// as 'values' but with a new number of non-zero elements.
// TODO: Expose this for real in ATen, some day?
// NB: Doesn't preserve data.
inline Tensor new_values_with_size_of(const Tensor& values, int64_t nnz) {
  std::vector<int64_t> size = values.sizes().vec();
  size[0] = nnz;
  return at::empty(size, values.options());
}

// NOTE [ Flatten Sparse Indices ]
// This helper function flattens a sparse indices tensor (a Tensor) into a 1D
// indices tensor. E.g.,
//   input = [[2, 4, 0],
//            [3, 1, 10]]
//   full_size = [2, 12]
//   output = [ 2 * 12 + 3, 4 * 12 + 1, 0 * 12 + 10 ] = [27, 49, 10]
//
// In other words, assuming that each `indices[i, :]` is a valid index to a
// tensor `t` of shape `full_size`. This returns the corresponding indices to
// the flattened tensor `t.reshape( prod(full_size[:indices.size(0)]), -1 )`.
// if forceClone is true, the result will forced to be a clone of self.
// if force_clone is true, the result will forced to be a clone of self.
TORCH_API Tensor flatten_indices(
    const Tensor& indices,
    IntArrayRef full_size,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are new_values_with_size_of, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 new_values_with_size_of，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 91-120
```cpp
    bool force_clone = false);

// Flatten sparse tensor's indices from nD to 1D, similar to NOTE [ Flatten
// Sparse Indices ], except this one allows partial flatten: only flatten on
// specified dims. Note that the flatten indices might be uncoalesced if
// dims_to_flatten.size() < sparse_dim. Also if input indices is already
// coalesced, the flattened indices will also be sorted.
//
// args:
//    indices: sparse tensor indices
//    sizes: sparse tensor sizes
//    dims_to_flatten: a list of dim index to flatten
//
// Ex1:
//   indices = [[2, 4, 0],
//             [3, 1, 3]]
//   sizes = [2, 12]
//   dims_to_flatten = [0, 1]
//   new_indices = [ 2 * 12 + 3, 4 * 12 + 1, 0 * 12 + 3 ] = [27, 49, 3]
//
// Ex2:
//   dims_to_flatten = [1]
//   new_indices = [ 3, 1, 3 ]  # uncoalesced
TORCH_API Tensor flatten_indices_by_dims(
    const Tensor& indices,
    const IntArrayRef& sizes,
    const IntArrayRef& dims_to_flatten);

// Find the CSR representation for a row `indices` from the COO format
TORCH_API Tensor coo_to_csr(const int64_t* indices, int64_t dim, int64_t nnz);
```
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 121-150
```cpp

TORCH_API Tensor zeros_like_with_indices(const Tensor& t);

template <size_t static_shape_max_len>
class TensorGeometryHolder {
  using geometry_holder_t = std::array<int64_t, static_shape_max_len>;

 public:
  explicit TensorGeometryHolder(
      IntArrayRef sizes,
      IntArrayRef strides,
      TensorOptions options = {}) {
    std::copy(sizes.begin(), sizes.end(), t_sizes.begin());
    std::copy(strides.begin(), strides.end(), t_strides.begin());
  }

  explicit TensorGeometryHolder(const Tensor& t)
      : TensorGeometryHolder(t.sizes(), t.strides()) {}

  auto operator*() const {
    return std::make_tuple(t_sizes, t_strides);
  }

 private:
  geometry_holder_t t_sizes;
  geometry_holder_t t_strides;
};

template <>
class TensorGeometryHolder<0> {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are TensorGeometryHolder, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 TensorGeometryHolder，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 151-180
```cpp
  using geometry_holder_t = Tensor;

 public:
  explicit TensorGeometryHolder(
      IntArrayRef sizes,
      IntArrayRef strides,
      TensorOptions options) {
    const int64_t t_ndims = sizes.size();
    const auto cpu_options = TensorOptions(options).dtype(kLong).device(kCPU);
    Tensor t_sizes_and_strides_cpu = at::empty({2, t_ndims}, cpu_options);
    t_sizes_and_strides_cpu.select(0, 0).copy_(at::tensor(sizes, cpu_options));
    t_sizes_and_strides_cpu.select(0, 1).copy_(
        at::tensor(strides, cpu_options));
    const Tensor t_sizes_and_strides =
        t_sizes_and_strides_cpu.to(options.device());
    t_sizes = t_sizes_and_strides.select(0, 0);
    t_strides = t_sizes_and_strides.select(0, 1);
  }

  explicit TensorGeometryHolder(const Tensor& t)
      : TensorGeometryHolder(t.sizes(), t.strides(), t.options()) {}

  auto operator*() const {
    return std::make_tuple(
        t_sizes.template data_ptr<int64_t>(),
        t_strides.template data_ptr<int64_t>());
  }

 private:
  geometry_holder_t t_sizes;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are TensorGeometryHolder, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 TensorGeometryHolder，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 181-190
```cpp
  geometry_holder_t t_strides;
};

// Return all indices of a tensor with the given shape.
//
// full_coo_indices(shape) is equivalent to
// torch.ones(shape).nonzero().transpose(-2, -1) but much faster.
TORCH_API Tensor full_coo_indices(IntArrayRef sizes, TensorOptions options);

} // namespace at::sparse
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

## Key Concepts / 关键概念

- EN: Backend focus: utility layer.
- CN: 后端重点：工具层。
- EN: Backend focus: sparse tensor support.
- CN: 后端重点：稀疏张量支持。
- EN: Template-based reuse helps share logic across scalar types, layouts, or backends.
- CN: 基于模板的复用帮助在不同标量类型、布局或后端之间共享逻辑。
- EN: Notable symbols: get_sparse_impl, alias_into_sparse, copy_into_sparse, is_same_tensor, is_same_density, new_values_with_size_of, TensorGeometryHolder.
- CN: 重要符号：get_sparse_impl, alias_into_sparse, copy_into_sparse, is_same_tensor, is_same_density, new_values_with_size_of, TensorGeometryHolder。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/Parallel.h, ATen/SparseTensorImpl.h, ATen/core/Tensor.h, ATen/Functions.h, ATen/ops/empty.h, ATen/ops/tensor.h`.
- CN: 主要内部头文件：`ATen/Parallel.h, ATen/SparseTensorImpl.h, ATen/core/Tensor.h, ATen/Functions.h, ATen/ops/empty.h, ATen/ops/tensor.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `get_sparse_impl, alias_into_sparse, copy_into_sparse, is_same_tensor, is_same_density, new_values_with_size_of, TensorGeometryHolder`.
- CN: 实现围绕 `get_sparse_impl, alias_into_sparse, copy_into_sparse, is_same_tensor, is_same_density, new_values_with_size_of, TensorGeometryHolder` 等符号展开。
