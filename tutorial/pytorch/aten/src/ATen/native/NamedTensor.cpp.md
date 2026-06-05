# NamedTensor.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/NamedTensor.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on named tensor; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 named tensor；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60
```cpp
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS
#include <ATen/core/Tensor.h>
#include <ATen/NamedTensorUtils.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/align_as_native.h>
#include <ATen/ops/align_tensors_native.h>
#include <ATen/ops/align_to_native.h>
#include <ATen/ops/gather_native.h>
#include <ATen/ops/index_add_native.h>
#include <ATen/ops/index_copy_native.h>
#include <ATen/ops/index_fill.h>
#include <ATen/ops/index_fill_native.h>
#include <ATen/ops/index_select_native.h>
#include <ATen/ops/refine_names_native.h>
#include <ATen/ops/rename_native.h>
#include <ATen/ops/scatter_add_native.h>
#include <ATen/ops/scatter_native.h>
#include <ATen/ops/sort_native.h>
#include <ATen/ops/squeeze.h>
#include <ATen/ops/squeeze_native.h>
#include <ATen/ops/zeros_like_ops.h>
#endif

#include <c10/util/irange.h>

#include <bitset>

namespace at::native {

Tensor& rename_(Tensor& self, std::optional<DimnameList> names) {
  at::internal_set_names_inplace(self, names);
  return self;
}

Tensor rename(const Tensor& self, std::optional<DimnameList> names) {
  auto result = self.alias();
  at::internal_set_names_inplace(result, names);
  return result;
}

static void report_moving_unnamed_dim_error(
    DimnameList names, DimnameList other, bool is_aligning_two_tensors) {
  if (is_aligning_two_tensors) {
    TORCH_CHECK(false,
        "Aligning Tensor", names, " and Tensor", other,
        " would change the absolute position from the right of an unnamed dimension. ",
        "Please name unnamed dimensions to avoid ambiguity.");
  } else {
    TORCH_CHECK(false,
        "Aligning Tensor", names, " to `names` ", other,
        " would change the absolute position from the right of an unnamed dimension. ",
        "Please name unnamed dimensions to avoid ambiguity.");
  }
}

static void report_not_a_subsequence_error(
```
- EN: Lines 1-60 pull in 23 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are rename_, rename, concentrating a specific part of the operator behavior.
- CN: 第 1-60 行引入了 23 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 rename_, rename，它们承载了某一部分算子行为的核心逻辑。

### Lines 61-120
```cpp
    DimnameList names, DimnameList other, bool is_aligning_two_tensors) {
  if (is_aligning_two_tensors) {
#ifndef STRIP_ERROR_MESSAGES
    auto shorter = names.size() > other.size() ? other : names;
    auto longer = names.size() > other.size() ? names : other;
#endif
    TORCH_CHECK(false,
        "Could not align Tensor", shorter, " and Tensor", longer,
        " because ", shorter, " is not a subsequence of ", longer, ". ");
  } else {
    TORCH_CHECK(false,
        "Could not align Tensor", names, " to `names` ", other,
        " because ", names, " is not a subsequence of `names`.");
  }
}


// Let tensor `t` have size `tensor_sizes` and `tensor_names`.
// This helper function computes the resulting size of `t` after aligning it
// to `aligned_names`. Enforces the alignment rules in Note [Alignment rules].
static std::vector<int64_t> aligned_size(
    IntArrayRef tensor_sizes,
    DimnameList tensor_names,
    DimnameList aligned_names,
    bool is_aligning_two_tensors) {
  std::vector<int64_t> expanded_sizes(aligned_names.size(), 1);
  ptrdiff_t dim = static_cast<ptrdiff_t>(tensor_sizes.size()) - 1;
  ptrdiff_t idx = static_cast<ptrdiff_t>(aligned_names.size()) - 1;
  for (; idx >= 0 && dim >= 0; --idx) {
    if (tensor_names[dim] != aligned_names[idx]) {
      continue;
    }
    // We've found a None name in `shorter` and `longer`. If their absolute positions
    // from the right are not equal, then aligning the two names would require
    // changing the absolute position from right of one of the None names,
    // violating condition 2 of our [Alignment rules].
    //
    // For example:
    // *, c, a, b
    //       *, a
    // [*, a] is a subsequence of [*, c, a, b], but in order to align them,
    // we'd have to move the * to create [*, c: 1, a, b: 1]
    if (tensor_names[dim].isWildcard() &&
        tensor_sizes.size() - dim != aligned_names.size() - idx) {
      report_moving_unnamed_dim_error(
          tensor_names, aligned_names, /*is_aligning_two_tensors=*/false);
    }
    expanded_sizes[idx] = tensor_sizes[dim];
    --dim;
  }
  if (dim != -1) {
    report_not_a_subsequence_error(
        tensor_names, aligned_names, /*is_aligning_two_tensors=*/false);
  }

  return expanded_sizes;
}

Tensor refine_names(const Tensor& self, DimnameList names) {
  const auto self_names = self.names();
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are size, refine_names, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 size, refine_names，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 121-180
```cpp
  TORCH_CHECK(self_names.size() == names.size(),
      "refine_names: cannot coerce Tensor", self_names, " to Tensor", names,
      " because they have a different number of dims (",
      self_names.size(), " and ", names.size(), " respectively).");
  check_names_valid_for(self, names);

  for (const auto idx : c10::irange(self_names.size())) {
    const auto& self_name = self_names[idx];
    const auto& out_name = names[idx];
    if (self_name == out_name || self_name.isWildcard()) {
      continue;
    }
    if (out_name.isWildcard()) {
      TORCH_CHECK(false,
          "refine_names: cannot coerce Tensor", self_names, " to Tensor", names,
          " because ", self_name, " is more specific than ", out_name, " at index ",
          idx);
    }
    TORCH_CHECK(false,
        "refine_names: cannot coerce Tensor", self_names, " to Tensor", names,
        " because ", self_name, " is different from ", out_name, " at index ",
        idx);
    TORCH_INTERNAL_ASSERT(false); // done handling errors
  }

  auto result = self.alias();
  internal_set_names_inplace(result, names);
  return result;
}

// [Alignment rules]
// Aligns `tensor` to names with the following rules:
// 1) Check that tensor.names is a subsequence (not necessarily contiguous) of `names`.
// 2) Aligning tensor.names to names must not change the absolute position from the
//    right of any unnamed dimension.
//
// is_aligning_two_tensors tunes the error message to better match the following cases:
// 1) tensor.align_to(names)  (is_aligning_two_tensors=false)
// 2) torch.align_tensors([tensor, other])  (is_aligning_two_tensors=true)
static Tensor align(const Tensor& tensor, DimnameList names, bool is_aligning_two_tensors) {
  std::vector<int64_t> expanded_sizes = aligned_size(
        tensor.sizes(),
        tensor.names(),
        names,
        is_aligning_two_tensors);
  auto result = tensor.rename(std::nullopt).view(expanded_sizes);
  at::internal_set_names_inplace(result, names);
  return result;
}

static int64_t countUnset(std::bitset<kMaxNamedTensorDim> set, int64_t up_to_idx) {
  int64_t result = 0;
  for (const auto i : c10::irange(up_to_idx)) {
    if (!set.test(i)) result++;
  }
  return result;
}

// Handles `tensor.align_to(*order)` in the case where there is an ellipsis.
//
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are align_to, align_tensors, align, countUnset, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 align_to, align_tensors, align, countUnset，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 181-240
```cpp
// Let tensor: Tensor[N, C, H, W]. Consider `tensor.align_to('W', ..., 'N')`
// We expand the `...` to "all unmentioned dimensions, in the order which they
// appear in the original tensor."
//
// `order` is passed in **without** the ellipsis name. This is because ellipsis
// is not a valid name in cpp right now. Future work should be done on making
// ellipsis a valid name.
//
// `ellipsis_idx` is where the ellipsis occurs in the Python call.
// In our example, `tensor.align_to('W', ..., 'N')`, order = ['W', 'N'] and
// ellipsis_idx = 1.
Tensor align_to(const Tensor& tensor, DimnameList order, int64_t ellipsis_idx) {
  const auto tensor_names = tensor.names();
  const auto tensor_sizes = tensor.sizes();
  const auto tensor_strides = tensor.strides();
  const auto tensor_dim = tensor.sizes().size();
  constexpr int64_t not_found = -1;

  // General strategy.
  //
  // Step 1: We compute the following 3 things:
  // 1. How many names the ellipsis should expand to
  // 2. Which names in `tensor.names` are not mentioned in `order`.
  // 3. Where names in `order` occur in tensor, if at all.
  //
  // Step 2: Compute the new sizes/strides/names.
  // First, determine the ndim of the output tensor (this is not obvious)
  // by counting the number of names in `tensor` that are not in `order`.
  // Next, fill in output sizes/strides/names by using `order` and knowledge
  // of which dimensions in `tensor` are unmentioned in `order`.

  std::bitset<kMaxNamedTensorDim> order_has_tensor_name;

  // tensor_idx_for[i] = j means that the ith name in `order`
  // appears in the jth element of tensor.
  std::vector<int64_t> tensor_idx_for(order.size(), not_found);

  for (const auto order_idx : c10::irange(order.size())) {
    const auto name = order[order_idx];
    TORCH_CHECK(name.isBasic(),
        "align_to: the desired order of dimensions cannot contain a None name, got ",
        order);
    auto it = std::find(tensor_names.begin(), tensor_names.end(), name);
    if (it == tensor_names.end()) {
      continue;
    }
    auto idx_in_tensor = std::distance(tensor_names.begin(), it);
    tensor_idx_for[order_idx] = idx_in_tensor;
    order_has_tensor_name.set(idx_in_tensor);
  }

  const auto num_ellipsis_names = countUnset(order_has_tensor_name, tensor_dim);
  const auto out_dim = num_ellipsis_names + order.size();

  // Step 2: Now that we know the size of the output tensor, we can use the
  // metadata obtained from Step 1 to fill in the new sizes/strides/names
  std::vector<int64_t> new_sizes(out_dim, 1);
  std::vector<int64_t> new_strides(out_dim, 0);
  std::vector<Dimname> new_names(out_dim, Dimname::wildcard());
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are align_to, tensor, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 align_to, tensor，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 241-300
```cpp
  auto setNewSizesStridesNamesFor = [&](int64_t out_dim, int64_t tensor_dim) {
    new_sizes[out_dim] = tensor_sizes[tensor_dim];
    new_strides[out_dim] = tensor_strides[tensor_dim];
    new_names[out_dim] = tensor_names[tensor_dim];
  };

  // Fill in the non-ellipsis dimensions
  for (const auto order_idx : c10::irange(static_cast<int64_t>(order.size()))) {
    auto out_idx = order_idx;
    if (order_idx >= ellipsis_idx) {
      out_idx = order_idx + num_ellipsis_names;
    }
    const auto tensor_idx = tensor_idx_for[order_idx];
    if (tensor_idx == not_found) {
      // We are adding a new size-one dimension
      new_names[out_idx] = order[order_idx];
      continue;
    }
    setNewSizesStridesNamesFor(out_idx, tensor_idx);
  }

  // Fill in the ellipsis dimensions
  for (const auto tensor_idx : c10::irange(tensor_dim)) {
    if (order_has_tensor_name.test(tensor_idx)) {
      continue;
    }
    setNewSizesStridesNamesFor(ellipsis_idx, tensor_idx);
    ellipsis_idx++;
  }

  check_names_valid_for(out_dim, new_names);

  Tensor result;
  {
    NoNamesGuard guard;
    result = tensor.as_strided(new_sizes, new_strides);
  }
  internal_set_names_inplace(result, std::move(new_names), /*validate_names=*/false);
  return result;
}

Tensor align_to(const Tensor& tensor, DimnameList names) {
  auto tensor_names = tensor.names();
  auto tensor_sizes = tensor.sizes();
  auto tensor_strides = tensor.strides();
  std::vector<int64_t> new_sizes(names.size(), 1);
  std::vector<int64_t> new_strides(names.size(), 0);

  for (const auto idx : c10::irange(tensor_names.size())) {
    const auto& dim = tensor_names[idx];
    TORCH_CHECK(dim.isBasic(),
        "align_to: All input dims must be named. Found unnamed dim at index ",
        idx, " of Tensor", tensor_names);
    auto it = std::find(names.begin(), names.end(), dim);
    TORCH_CHECK(it != names.end(),
        "align_to: Cannot find dim ", dim, " from Tensor", names,
        " in desired alignment ", names, ".");
    int64_t new_idx = std::distance(names.begin(), it);
    new_sizes[new_idx] = tensor_sizes[idx];
    new_strides[new_idx] = tensor_strides[idx];
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are align_to, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 align_to，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 301-360
```cpp
  }
  Tensor result;
  {
    NoNamesGuard guard;
    result = tensor.as_strided(new_sizes, new_strides);
  }
  internal_set_names_inplace(result, names);
  return result;
}

Tensor align_as(const Tensor& tensor, const Tensor& other) {
  return native::align_to(tensor, other.names());
}

static std::vector<Tensor> align_tensors_to(TensorList tensors, DimnameList names) {
  std::vector<Tensor> result;
  result.reserve(tensors.size());
  for (const auto& tensor : tensors) {
    result.emplace_back(align(tensor, names, /*is_aligning_two_tensors=*/true));
  }
  return result;
}

std::vector<Tensor> align_tensors(TensorList tensors) {
  auto longest_dim = std::max_element(
      tensors.begin(), tensors.end(),
      [](const Tensor& a, const Tensor& b) {
        return a.dim() < b.dim();
      });
  return align_tensors_to(tensors, longest_dim->names());
}

// Misc. Dimname overloads that don't have homes. Maybe we should move
// all of them here or autogenerate them because they look so similar.
Tensor gather(const Tensor& self, Dimname dim, const Tensor& index, bool sparse_grad) {
  reportNYIDimnameOverload("gather");
}
Tensor& gather_out(const Tensor& self, Dimname dim, const Tensor& index, bool sparse_grad, Tensor& result) {
  reportNYIDimnameOverload("gather");
}
Tensor index_add(const Tensor& self, Dimname dim, const Tensor& index, const Tensor& source, const Scalar &alpha) {
  reportNYIDimnameOverload("index_add");
}
Tensor index_fill(const Tensor& self, Dimname dim, const Tensor& index, const Scalar& source) {
  return at::index_fill(self, dimname_to_position(self, dim), index, source);
}
Tensor& index_fill_(Tensor& self, Dimname dim, const Tensor& index, const Scalar& source) {
  return self.index_fill_(dimname_to_position(self, dim), index, source);
}
Tensor index_fill(const Tensor& self, Dimname dim, const Tensor& index, const Tensor& source) {
  return at::index_fill(self, dimname_to_position(self, dim), index, source);
}
Tensor& index_fill_(Tensor& self, Dimname dim, const Tensor& index, const Tensor& source) {
  return self.index_fill_(dimname_to_position(self, dim), index, source);
}
Tensor index_copy(const Tensor& self, Dimname dim, const Tensor& index, const Tensor& source) {
  reportNYIDimnameOverload("index_copy");
}
Tensor& index_copy_(Tensor& self, Dimname dim, const Tensor& index, const Tensor& source) {
  reportNYIDimnameOverload("index_copy");
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are align_as, align_tensors_to, align_tensors, gather, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 align_as, align_tensors_to, align_tensors, gather，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 361-397
```cpp
}
Tensor& index_select_out(const Tensor& self, Dimname dim, const Tensor& index, Tensor& out) {
  reportNYIDimnameOverload("index_select");
}
Tensor index_select(const Tensor& self, Dimname dim, const Tensor& index) {
  reportNYIDimnameOverload("index_select");
}
Tensor scatter(const Tensor& self, Dimname dim, const Tensor& index, const Tensor& source) {
  reportNYIDimnameOverload("scatter");
}
Tensor scatter(const Tensor& self, Dimname dim, const Tensor& index, const Scalar& source) {
  reportNYIDimnameOverload("scatter");
}
Tensor scatter_add(const Tensor& self, Dimname dim, const Tensor& index, const Tensor& source) {
  reportNYIDimnameOverload("scatter_add");
}
std::tuple<Tensor&, Tensor&> sort_out(const Tensor& self, std::optional<bool> stable, Dimname dim, bool keepdim, Tensor& values, Tensor& indices) {
  reportNYIDimnameOverload("sort");
}
std::tuple<Tensor&, Tensor&> sort_out(const Tensor& self, Dimname dim, bool keepdim, Tensor& values, Tensor& indices) {
  reportNYIDimnameOverload("sort");
}
std::tuple<Tensor, Tensor> sort(const Tensor& self, std::optional<bool> stable, Dimname dim, bool keepdim) {
  reportNYIDimnameOverload("sort");
}
std::tuple<Tensor, Tensor> sort(const Tensor& self, Dimname dim, bool keepdim) {
  reportNYIDimnameOverload("sort");
}
Tensor& squeeze_(Tensor& self, Dimname dim) {
  reportNYIDimnameOverload("squeeze");
}
Tensor squeeze(const Tensor& self, Dimname dim) {
  return at::squeeze(self, dimname_to_position(self, dim));
}


}  // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are index_select_out, index_select, scatter, scatter_add, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 index_select_out, index_select, scatter, scatter_add，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

## Key Concepts / 关键概念

- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: rename_, rename, report_moving_unnamed_dim_error, report_not_a_subsequence_error, aligned_size, refine_names, align, countUnset.
- CN: 重要符号：rename_, rename, report_moving_unnamed_dim_error, report_not_a_subsequence_error, aligned_size, refine_names, align, countUnset。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/core/Tensor.h, ATen/NamedTensorUtils.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/align_as_native.h, ATen/ops/align_tensors_native.h, ATen/ops/align_to_native.h, ATen/ops/gather_native.h, ATen/ops/index_add_native.h, ATen/ops/index_copy_native.h, ATen/ops/index_fill.h, ATen/ops/index_fill_native.h`.
- CN: 主要内部头文件：`ATen/core/Tensor.h, ATen/NamedTensorUtils.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/align_as_native.h, ATen/ops/align_tensors_native.h, ATen/ops/align_to_native.h, ATen/ops/gather_native.h, ATen/ops/index_add_native.h, ATen/ops/index_copy_native.h, ATen/ops/index_fill.h, ATen/ops/index_fill_native.h`。
- EN: External/system headers: `bitset`.
- CN: 外部/系统头文件：`bitset`。
- EN: The implementation revolves around symbols such as `rename_, rename, report_moving_unnamed_dim_error, report_not_a_subsequence_error, aligned_size, refine_names, align, countUnset, align_to, align_as`.
- CN: 实现围绕 `rename_, rename, report_moving_unnamed_dim_error, report_not_a_subsequence_error, aligned_size, refine_names, align, countUnset, align_to, align_as` 等符号展开。
