# RowwisePrune.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/RowwisePrune.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on rowwise prune; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 rowwise prune；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
// Copyright 2004-present Facebook. All Rights Reserved.
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS

#include <ATen/core/Tensor.h>
#include <ATen/Dispatch.h>
#include <c10/util/irange.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/_rowwise_prune_native.h>
#include <ATen/ops/empty.h>
#endif

namespace at::native {

namespace {

template <typename input_t>
std::tuple<Tensor, Tensor> _rowwise_prune_helper(
      const Tensor& weights, const Tensor& mask,
      ScalarType compressed_indices_dtype) {
  int num_non_masked_rows = 0;
  auto mask_contig = mask.contiguous();
  auto mask_data = mask_contig.const_data_ptr<bool>();
  for (const auto i : c10::irange(mask.numel())) {
    num_non_masked_rows += ((mask_data[i] == true) ? 1 : 0);
  }
  int num_cols = weights.size(1);
```
- EN: Lines 1-30 pull in 7 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 第 1-30 行引入了 7 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 31-60
```cpp
  auto pruned_2d_tensor = at::empty({num_non_masked_rows, num_cols},
      weights.options());
  auto compressed_indices_mapping = at::empty({mask.numel()},
      compressed_indices_dtype);
  AT_DISPATCH_ALL_TYPES_AND2(at::ScalarType::Half,
                             at::ScalarType::BFloat16,
                             weights.scalar_type(),
                            "rowwise_prune_helper", [&]() {
    auto* pruned_2d_tensor_data = pruned_2d_tensor.data_ptr<scalar_t>();
    auto compressed_indices_mapping_data =
        compressed_indices_mapping.data_ptr<input_t>();
    auto weights_data = weights.const_data_ptr<scalar_t>();
    int last_row_kept = 0;
    for (const auto i : c10::irange(mask.numel())) {
      if (mask_data[i]) {
        memcpy(pruned_2d_tensor_data + last_row_kept * num_cols,
              weights_data + i * num_cols,
              num_cols * sizeof (scalar_t));
        compressed_indices_mapping_data[i] = last_row_kept;
        last_row_kept++;
      } else {
        compressed_indices_mapping_data[i] = -1;
      }
    }
  });
  return std::tuple<Tensor, Tensor>(std::move(pruned_2d_tensor),
      std::move(compressed_indices_mapping));
}

} // namespace
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 61-90
```cpp


// This operator introduces sparsity to the 'weights' matrix with the help
// of the importance indicator 'mask'.
//
// A row is considered important and not pruned if the mask value for that
// particular row is 1(True) and not important otherwise.
//
// This operator doesn't zero out the pruned rows in-place. Instead, it
// returns a tuple that contains a pruned weights tensor as well as a map that
// can be used to look up the original row in the pruned weights tensor.
// We refer this map as 'compressed indices map' going forward.

// The 'compressed indices map' is an 1D tensor that contains one entry per
// original row in 'weights'. The array index is the index for the original
// non-pruned weight tensor and the value would be the re-mapped index in the
// pruned weights tensor. If the value for a index is -1, it means the
// corresponding row has been pruned from the original weight tensor.

// Arguments:
// 'weights' - two dimensional matrix that needs to be prune.
// 'mask' - 1D boolean tensor that represents whether a row is important or
//    not. A mask value of 1 means the row should be kept and 0 means the row
//    should be pruned.
//
// Returns:
// A tuple containing two tensors,
// 1. A pruned weight tensor that contains only the weights that are preserved
//    post pruning.
// 2. An 1D tensor that contains the mapping between original weight row and
```
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

### Lines 91-115
```cpp
//    the corresponding row in the pruned weights tensor.
std::tuple<Tensor, Tensor> _rowwise_prune(const Tensor& weights,
                                          const Tensor& mask,
                                          ScalarType compressed_indices_dtype) {
  TORCH_CHECK(weights.ndimension() == 2,
      "'weights' should have 2 dimensions.");
  TORCH_CHECK(
    mask.numel() == weights.size(0),
    "Number of elements in 'mask' should be equivalent to the "
    "number of rows in 'weights'."
  )
  TORCH_CHECK(
      compressed_indices_dtype == ScalarType::Int ||
      compressed_indices_dtype == ScalarType::Long,
      "compressed_indices_dtype should be either int(int32) or long(int64).");

  if (compressed_indices_dtype == at::ScalarType::Int) {
    return _rowwise_prune_helper<int32_t>(weights, mask,
                                          compressed_indices_dtype);
  }
  return _rowwise_prune_helper<int64_t>(weights, mask,
                                        compressed_indices_dtype);
}

} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

## Key Concepts / 关键概念

- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Template-based reuse helps share logic across scalar types, layouts, or backends.
- CN: 基于模板的复用帮助在不同标量类型、布局或后端之间共享逻辑。
- EN: Notable symbols: _rowwise_prune_helper, AT_DISPATCH_ALL_TYPES_AND2, _rowwise_prune, TORCH_CHECK.
- CN: 重要符号：_rowwise_prune_helper, AT_DISPATCH_ALL_TYPES_AND2, _rowwise_prune, TORCH_CHECK。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/core/Tensor.h, ATen/Dispatch.h, c10/util/irange.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/_rowwise_prune_native.h, ATen/ops/empty.h`.
- CN: 主要内部头文件：`ATen/core/Tensor.h, ATen/Dispatch.h, c10/util/irange.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/_rowwise_prune_native.h, ATen/ops/empty.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `_rowwise_prune_helper, AT_DISPATCH_ALL_TYPES_AND2, _rowwise_prune, TORCH_CHECK`.
- CN: 实现围绕 `_rowwise_prune_helper, AT_DISPATCH_ALL_TYPES_AND2, _rowwise_prune, TORCH_CHECK` 等符号展开。
