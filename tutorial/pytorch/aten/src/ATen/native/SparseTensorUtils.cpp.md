# SparseTensorUtils.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/SparseTensorUtils.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on sparse tensor utils; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 sparse tensor utils；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#include <ATen/native/SparseTensorUtils.h>

#include <ATen/ATen.h>
#include <ATen/SparseTensorImpl.h>
#include <ATen/native/sparse/SparseStubs.h>
#include <ATen/Parallel.h>
#include <c10/util/irange.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#else
#include <ATen/ops/_sparse_coo_tensor_with_dims_and_tensors.h>
#include <ATen/ops/zeros.h>
#endif

namespace at::native {

DEFINE_DISPATCH(flatten_indices_stub);

} // namespace at::native

namespace at::sparse {

// NOTE [ Flatten Sparse Indices ]
// This helper function flattens a sparse indices tensor (a Tensor) into a 1D
// indices tensor. E.g.,
//   input = [[2, 4, 0],
//            [3, 1, 10]]
//   full_size = [2, 12]
//   output = [ 2 * 12 + 3, 4 * 12 + 1, 0 * 12 + 10 ] = [27, 49, 10]
```
- EN: Lines 1-30 pull in 9 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Registration or dispatch macros appear here, wiring the implementation into PyTorch operator lookup and backend dispatch.
- CN: 第 1-30 行引入了 9 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这里出现了注册或分发宏，用于把实现接入 PyTorch 的算子查找与后端分发机制。

### Lines 31-60
```cpp
//
// In other words, assuming that each `indices[i, :]` is a valid index to a
// tensor `t` of shape `full_size`. This returns the corresponding indices to
// the flattened tensor `t.reshape( prod(full_size[:indices.size(0)]), -1 )`.
// if forceClone is true, the result will forced to be a clone of self.
// if force_clone is true, the result will forced to be a clone of self.
Tensor flatten_indices(const Tensor& indices, IntArrayRef full_size, bool force_clone /*= false*/) {
  int64_t sparse_dim = indices.size(0);
  if (sparse_dim == 1) {
    if (force_clone) {
      return indices.squeeze(0).clone(at::MemoryFormat::Contiguous);
    } else {
      return indices.squeeze(0);
    }
  } else {
    if (!indices.numel()) {
      return at::zeros({indices.size(1)}, indices.options().dtype(kLong));
    }
    return at::native::flatten_indices_stub(indices.device().type(), indices, full_size.slice(0, sparse_dim));
  }
}

// Flatten sparse tensor's indices from nD to 1D, similar to NOTE [ Flatten Sparse Indices ],
// except this one allows partial flatten: only flatten on specified dims. Note that
// the flatten indices might be uncoalesced if dims_to_flatten.size() < sparse_dim.
// Also if input indices is already coalesced, the flattened indices will also be sorted.
//
// args:
//    indices: sparse tensor indices
//    sizes: sparse tensor sizes
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are flatten_indices, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 flatten_indices，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 61-90
```cpp
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
Tensor flatten_indices_by_dims(const Tensor& indices, const IntArrayRef& sizes, const IntArrayRef& dims_to_flatten){
  Tensor new_indices = at::zeros({indices.size(1)}, indices.options());
  for (auto d : dims_to_flatten) {
    new_indices.mul_(sizes[d]);
    new_indices.add_(indices.select(0, d));
  }
  return new_indices;
}

Tensor coo_to_csr(const int64_t* indices, int64_t dim, int64_t nnz) {
  /*
    Find the CSR representation for a row `indices` from the COO format
    Inputs:
      `indices` is the row pointer from COO indices
      `dim` is the row dimensionality
      `nnz` is the number of non-zeros

    Output:
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are flatten_indices_by_dims, coo_to_csr, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 flatten_indices_by_dims, coo_to_csr，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 91-120
```cpp
      `csr` is a compressed row array in a CSR format
  */
  Tensor csr = at::zeros({dim + 1}, kLong);

  // TODO: eliminate this conditional when zero-size dims supported correctly
  if (nnz > 0) {
    auto csr_accessor = csr.accessor<int64_t, 1>();
    // Convert the sparse matrix to CSR format
    at::parallel_for(0, nnz, 10000, [&](int64_t start, int64_t end) {
      for (const auto i : c10::irange(start, end)) {
        auto hp0 = indices[i];
        auto hp1 = (i+1 == nnz) ?  dim : indices[i+1];
        if (hp0 != hp1) {
          for (int64_t h = hp0; h < hp1; h++) {
            csr_accessor[h+1] = i+1;
          }
        }
      }
    });
  }
  return csr;
}

Tensor zeros_like_with_indices(const Tensor& t) {
  TORCH_INTERNAL_ASSERT(t.is_sparse());
  return at::_sparse_coo_tensor_with_dims_and_tensors(
      t.sparse_dim(),
      t.dense_dim(),
      t.sizes(),
      t._indices().clone(),
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are parallel_for, zeros_like_with_indices, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 parallel_for, zeros_like_with_indices，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 121-143
```cpp
      at::zeros({1}, t._values().options()).expand_as(t._values()),
      t.options(),
      t.is_coalesced());
}

Tensor full_coo_indices(IntArrayRef sizes, TensorOptions options) {
  const auto max_size = *std::max_element(sizes.begin(), sizes.end());
  const auto max_size_arange = at::arange(max_size, options);
  std::vector<Tensor> stack;
  stack.reserve(sizes.size());
  for (size_t i=0; i < sizes.size(); i++) {
    Tensor a = max_size_arange.narrow(-1, 0, sizes[i]);
    for (size_t j=0; j < sizes.size(); j++) {
      if (i != j) {
        a.unsqueeze_(j);
      }
    }
    stack.push_back(a.expand(sizes));
  }
  return at::stack(stack).flatten(1, -1);
}

} // namespace at::sparse
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are full_coo_indices, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 full_coo_indices，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

## Key Concepts / 关键概念

- EN: Backend focus: utility layer.
- CN: 后端重点：工具层。
- EN: Backend focus: sparse tensor support.
- CN: 后端重点：稀疏张量支持。
- EN: Registration/dispatch logic is part of the file structure.
- CN: 文件结构中包含注册/分发逻辑。
- EN: Notable symbols: flatten_indices, flatten_indices_by_dims, coo_to_csr, parallel_for, zeros_like_with_indices, full_coo_indices.
- CN: 重要符号：flatten_indices, flatten_indices_by_dims, coo_to_csr, parallel_for, zeros_like_with_indices, full_coo_indices。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/native/SparseTensorUtils.h, ATen/ATen.h, ATen/SparseTensorImpl.h, ATen/native/sparse/SparseStubs.h, ATen/Parallel.h, c10/util/irange.h, ATen/Functions.h, ATen/ops/_sparse_coo_tensor_with_dims_and_tensors.h, ATen/ops/zeros.h`.
- CN: 主要内部头文件：`ATen/native/SparseTensorUtils.h, ATen/ATen.h, ATen/SparseTensorImpl.h, ATen/native/sparse/SparseStubs.h, ATen/Parallel.h, c10/util/irange.h, ATen/Functions.h, ATen/ops/_sparse_coo_tensor_with_dims_and_tensors.h, ATen/ops/zeros.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `flatten_indices, flatten_indices_by_dims, coo_to_csr, parallel_for, zeros_like_with_indices, full_coo_indices`.
- CN: 实现围绕 `flatten_indices, flatten_indices_by_dims, coo_to_csr, parallel_for, zeros_like_with_indices, full_coo_indices` 等符号展开。
