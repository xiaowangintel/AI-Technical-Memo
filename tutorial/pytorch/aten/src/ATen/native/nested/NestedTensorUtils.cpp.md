# NestedTensorUtils.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/nested/NestedTensorUtils.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the nested tensor support in PyTorch ATen native code and focuses on nested tensor utils; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的嵌套张量支持，主题聚焦于 nested tensor utils；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#include <ATen/NestedTensorImpl.h>
#include <ATen/native/nested/NestedTensorUtils.h>
#include <optional>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/_nested_tensor_size_native.h>
#include <ATen/ops/_nested_tensor_storage_offsets_native.h>
#include <ATen/ops/_nested_tensor_strides_native.h>
#include <ATen/ops/chunk_native.h>
#include <ATen/ops/split_with_sizes_native.h>
#include <ATen/ops/value_selecting_reduction_backward_native.h>
#endif

namespace at::native {

/**
 * Thin wrapper around get_nested_sizes that is registered as a native function
 *
 * @return The nested tensors' size tensor.
 */
at::Tensor _nested_tensor_size(const at::Tensor& self) {
  return get_nested_sizes(self);
}

at::Tensor _nested_tensor_strides(const at::Tensor& self){
  return  get_nested_tensor_impl(self) -> get_nested_strides();
}
at::Tensor _nested_tensor_storage_offsets(const at::Tensor& self){
```
- EN: Lines 1-30 pull in 10 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are _nested_tensor_size, _nested_tensor_strides, _nested_tensor_storage_offsets, concentrating a specific part of the operator behavior.
- CN: 第 1-30 行引入了 10 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 _nested_tensor_size, _nested_tensor_strides, _nested_tensor_storage_offsets，它们承载了某一部分算子行为的核心逻辑。

### Lines 31-60
```cpp
  return get_nested_tensor_impl(self) -> get_storage_offsets();
}

// Helper functions for getting information about a nested tensor's shape.
std::vector<int64_t> NestedTensor_get_max_size_from_size_tensor(
    const Tensor& sizes) {
  if (sizes.dim() == 0) {
    return {};
  }
  const auto sizes_ptr = sizes.const_data_ptr<int64_t>();
  const auto sizes_size_0 = sizes.sizes()[0];
  const auto sizes_size_1 = sizes.sizes()[1];
  TORCH_INTERNAL_ASSERT(sizes_size_1 > 0);
  std::vector<int64_t> results(sizes_size_1, 0);
  for (const auto ii : c10::irange(sizes_size_0)) {
    for (const auto jj : c10::irange(sizes_size_1)) {
      auto val = sizes_ptr[ii * sizes_size_1 + jj];
      if (results[jj] < val) {
        results[jj] = val;
      }
    }
  }
  return results;
}

std::vector<int64_t> NestedTensor_get_max_size(const NestedTensorImpl& nt) {
  return NestedTensor_get_max_size_from_size_tensor(
      nt.get_nested_sizes());
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are NestedTensor_get_max_size, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 NestedTensor_get_max_size，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 61-90
```cpp
int64_t get_consistent_last_dim_of_nested_tensor(const NestedTensorImpl& nt) {
  std::optional<int64_t> last_dim = nt.opt_size(-1);
  TORCH_CHECK(
      last_dim.has_value(),
      "Expected all tensors in nested tensor to have the same trailing dimension, instead last dimension equals: ",
      nt.get_nested_sizes().select(1, -1));
  return *last_dim;
}

std::vector<Tensor> chunk_nested_tensor(const Tensor& self, int64_t chunks, int64_t dim) {
  int64_t ndim = self.dim();
  if (ndim == 0) {
    TORCH_CHECK_INDEX(false, "chunk() cannot be applied to a 0-dim tensor.");
  }
  dim = maybe_wrap_dim(dim, ndim);
  TORCH_CHECK(self.dim() - 1 == dim,
           "Chunk for nested tensors is currently only supported for the last dimension.");
  TORCH_CHECK(chunks > 0,"chunk expects `chunks` to be greater than 0, got: ", chunks);
  TORCH_CHECK(self.is_contiguous(), "chunk expects `self` to be contiguous.");
  auto self_impl = get_nested_tensor_impl(self);
  const int64_t last_dim_size = get_consistent_last_dim_of_nested_tensor(*self_impl);
    TORCH_CHECK(last_dim_size % chunks == 0,
           "Chunk for nested tensors is only supported for nested tensors with trailing dimension divisible by chunks, got: ",
           last_dim_size, " % ", chunks, " != 0");
  int64_t n_tensors = self.size(0);
  int64_t split_size = last_dim_size / chunks;
  std::vector<Tensor> splits(chunks);
  const auto& sizes = self_impl->get_nested_sizes();
  const auto& strides = self_impl->get_nested_strides();
  const auto offsets = self_impl->get_storage_offsets();
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are get_consistent_last_dim_of_nested_tensor, chunk_nested_tensor, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 get_consistent_last_dim_of_nested_tensor, chunk_nested_tensor，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 91-120
```cpp
  const int64_t *offsets_ptr = offsets.const_data_ptr<int64_t>();
  // Account for the implicit batch dim
  --dim;
  int64_t tensor_dim = sizes.size(1);
  for (const auto split_idx : c10::irange(chunks)) {
      auto new_sizes = sizes.clone();
      auto new_strides = strides.clone();
      // This copies offsets so we are safe to move
      auto new_offsets = offsets.clone();
      int64_t *size_ptr = new_sizes.data_ptr<int64_t>();
      int64_t *new_offsets_ptr = new_offsets.data_ptr<int64_t>();
      // Get start val for each split
      int64_t start_val = split_idx * split_size;
      for (int64_t i : c10::irange(n_tensors)) {
        const int64_t index = i * tensor_dim + dim;
        new_offsets_ptr[i] = offsets_ptr[i] + start_val;
        size_ptr[index] = split_size;
    }
    splits[split_idx] = create_nested_view_tensor(self, new_sizes, new_strides, new_offsets);
  }
  return splits;
}

std::vector<Tensor> split_with_sizes_nested(
    const Tensor& self,
    c10::IntArrayRef split_sizes,
    int64_t dim) {
  int64_t ndim = self.dim();
  if (ndim == 0) {
    TORCH_CHECK_INDEX(false, "split_with_sizes() cannot be applied to a 0-dim tensor.");
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 121-150
```cpp
  }
  dim = maybe_wrap_dim(dim, ndim);
  TORCH_CHECK(self.dim() - 1 == dim,
           "split_with_sizes for nested tensors is currently only supported for the last dimension.");
  auto num_splits = split_sizes.size();
  TORCH_CHECK(num_splits > 0,
           "split_with_sizes expects number of splits to be greater than 0, got: ", num_splits);
  TORCH_CHECK(self.is_contiguous(), "split_with_sizes expects `self` to be contiguous.");

  // Ensure entire dim is split.
  int64_t total_size = 0;
  for (const auto split_size : split_sizes) {
      total_size += split_size;
  }
  auto self_impl = get_nested_tensor_impl(self);
  auto self_size = get_consistent_last_dim_of_nested_tensor(*self_impl);
  TORCH_CHECK(total_size == self_size,
          "split_with_sizes expects split_sizes to sum exactly to ", self_size,
          " (input tensor's size at dimension ", dim, "), but got split_sizes=", split_sizes);

  int64_t n_tensors = self.size(0);
  std::vector<Tensor> splits(num_splits);
  const auto& sizes = self_impl->get_nested_sizes();
  const auto& strides = self_impl->get_nested_strides();
  const auto offsets = self_impl->get_storage_offsets();
  const int64_t *offsets_ptr = offsets.const_data_ptr<int64_t>();
  // Account for the implicit batch dim
  --dim;
  int64_t tensor_dim = sizes.size(1);
  int64_t start_val = 0;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 151-180
```cpp
  for (const auto split_idx : c10::irange(num_splits)) {
    auto split_size = split_sizes[split_idx];
    auto new_sizes = sizes.clone();
    auto new_strides = strides.clone();
    auto new_offsets = offsets.clone();
    int64_t *size_ptr = new_sizes.data_ptr<int64_t>();
    int64_t *new_offsets_ptr = new_offsets.data_ptr<int64_t>();
    // Get start val for each split
    for (int64_t i : c10::irange(n_tensors)) {
      const int64_t index = i * tensor_dim + dim;
      new_offsets_ptr[i] = offsets_ptr[i] + start_val;
      size_ptr[index] = split_size;
    }
    start_val += split_size;
    splits[split_idx] = create_nested_view_tensor(self, new_sizes, new_strides, new_offsets);
  }
  return splits;
}

Tensor value_selecting_reduction_backward_nested_symint(
    const Tensor& grad,
    int64_t dim,
    const Tensor& indices,
    c10::SymIntArrayRef sizes,
    bool keepdim) {
  TORCH_INTERNAL_ASSERT(
      false, "value_selecting_reduction_backward(): expected to be implemented in Python"
  );
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 181-181
```cpp
} // namespace at::native
```
- EN: This range provides supporting glue code: declarations, simple forwarding logic, or structural scaffolding reused by nearby operator code.
- CN: 这一段提供了辅助性胶水代码：声明、简单转发逻辑，或被邻近算子代码复用的结构骨架。

## Key Concepts / 关键概念

- EN: Backend focus: nested tensor support.
- CN: 后端重点：嵌套张量支持。
- EN: Backend focus: utility layer.
- CN: 后端重点：工具层。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: _nested_tensor_size, _nested_tensor_strides, _nested_tensor_storage_offsets, NestedTensor_get_max_size_from_size_tensor, NestedTensor_get_max_size, get_consistent_last_dim_of_nested_tensor, chunk_nested_tensor, split_with_sizes_nested.
- CN: 重要符号：_nested_tensor_size, _nested_tensor_strides, _nested_tensor_storage_offsets, NestedTensor_get_max_size_from_size_tensor, NestedTensor_get_max_size, get_consistent_last_dim_of_nested_tensor, chunk_nested_tensor, split_with_sizes_nested。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/NestedTensorImpl.h, ATen/native/nested/NestedTensorUtils.h, ATen/NativeFunctions.h, ATen/ops/_nested_tensor_size_native.h, ATen/ops/_nested_tensor_storage_offsets_native.h, ATen/ops/_nested_tensor_strides_native.h, ATen/ops/chunk_native.h, ATen/ops/split_with_sizes_native.h, ATen/ops/value_selecting_reduction_backward_native.h`.
- CN: 主要内部头文件：`ATen/NestedTensorImpl.h, ATen/native/nested/NestedTensorUtils.h, ATen/NativeFunctions.h, ATen/ops/_nested_tensor_size_native.h, ATen/ops/_nested_tensor_storage_offsets_native.h, ATen/ops/_nested_tensor_strides_native.h, ATen/ops/chunk_native.h, ATen/ops/split_with_sizes_native.h, ATen/ops/value_selecting_reduction_backward_native.h`。
- EN: External/system headers: `optional`.
- CN: 外部/系统头文件：`optional`。
- EN: The implementation revolves around symbols such as `_nested_tensor_size, _nested_tensor_strides, _nested_tensor_storage_offsets, NestedTensor_get_max_size_from_size_tensor, NestedTensor_get_max_size, get_consistent_last_dim_of_nested_tensor, chunk_nested_tensor, split_with_sizes_nested, value_selecting_reduction_backward_nested_symint`.
- CN: 实现围绕 `_nested_tensor_size, _nested_tensor_strides, _nested_tensor_storage_offsets, NestedTensor_get_max_size_from_size_tensor, NestedTensor_get_max_size, get_consistent_last_dim_of_nested_tensor, chunk_nested_tensor, split_with_sizes_nested, value_selecting_reduction_backward_nested_symint` 等符号展开。
