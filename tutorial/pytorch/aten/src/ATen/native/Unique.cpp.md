# Unique.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/Unique.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on unique; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 unique；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-60
```cpp
// Returns unique elements of input tensor.
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS

#include <ATen/core/Tensor.h>
#include <ATen/Dispatch_v2.h>
#include <ATen/Parallel.h>
#include <ATen/native/TensorIterator.h>
#include <c10/util/irange.h>
#include <c10/util/Load.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/_unique2_native.h>
#include <ATen/ops/_unique_native.h>
#include <ATen/ops/empty.h>
#include <ATen/ops/equal.h>
#include <ATen/ops/narrow.h>
#include <ATen/ops/stack.h>
#include <ATen/ops/unbind.h>
#include <ATen/ops/unique_consecutive_native.h>
#include <ATen/ops/unique_dim_consecutive_native.h>
#include <ATen/ops/unique_dim_native.h>
#include <ATen/ops/zeros.h>
#endif

namespace at::native {

namespace {

// This unique implementation when dtype is bool is mapped
// from UniqueCub.cu which uses a reduction to find the number of
// true values.
std::tuple<Tensor, Tensor, Tensor> unique_cpu_bool_template(
    const Tensor& self,
    const bool return_inverse,
    const bool return_counts) {
  const Tensor& input = self.contiguous();
  const bool* input_data = input.const_data_ptr<bool>();

  int64_t numel = input.numel();
  Tensor output = at::empty({0}, self.options());
  Tensor inverse_indices = at::empty({0}, self.options().dtype(kLong));
  Tensor counts = at::empty({0}, self.options().dtype(kLong));

  if (numel == 0) {
    if (return_inverse) {
      inverse_indices.resize_(input.sizes());
    }
    return std::make_tuple(output, inverse_indices, counts);
  }

  int num_threads = at::get_num_threads();
  std::vector<int64_t> num_true_thread(num_threads, 0);

  const int64_t grain_size = at::internal::GRAIN_SIZE;
  at::parallel_for(0, numel, grain_size, [&](int64_t begin, int64_t end) {
    int tid = at::get_thread_num();
    for (const auto i : c10::irange(begin, end)) {
```
- EN: Lines 1-60 pull in 19 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are parallel_for, concentrating a specific part of the operator behavior.
- CN: 第 1-60 行引入了 19 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 parallel_for，它们承载了某一部分算子行为的核心逻辑。

### Lines 61-120
```cpp
      const bool value = c10::load(&input_data[i]);
      if (value) {
        num_true_thread[tid]++;
      }
    }
  });

  int64_t num_true = std::accumulate(num_true_thread.begin(), num_true_thread.end(), 0);
  int64_t num_false = numel - num_true;
  int num_out = ((num_true > 0) + (num_false > 0));

  constexpr int false_idx = 0;
  const int true_idx = num_false > 0;

  output.resize_({num_out});
  if (return_counts) {
    counts.resize_({num_out});
  }
  bool* output_data = output.data_ptr<bool>();
  int64_t* counts_data = return_counts ? counts.data_ptr<int64_t>() : nullptr;

  // write output and counts
  if (num_false > 0) {
    output_data[false_idx] = false;
    if (return_counts) {
      counts_data[false_idx] = num_false;
    }
  }
  if (num_true > 0) {
    output_data[true_idx] = true;
    if (return_counts) {
      counts_data[true_idx] = num_true;
    }
  }

  if (return_inverse) {
    inverse_indices.resize_(input.sizes());
    int64_t* inverse_indices_data = inverse_indices.data_ptr<int64_t>();
    at::parallel_for(0, numel, grain_size, [&](int64_t begin, int64_t end) {
      for (const auto i : c10::irange(begin, end)) {
        const bool value = c10::load(&input_data[i]);
        inverse_indices_data[i] = value ? true_idx : false_idx;
      }
    });
  }
  return std::make_tuple(output, inverse_indices, counts);
}

// check whether the element on index i is `unique`,
// in the sorted sequence, the 1st element is always true.
//
// NaN is propagated to the rear in a sorted sequence,
// consider a sorted sequence of
//   {1.0, 1.0, 2.0, 2.0, NaN, NaN, NaN}
//
// a. `equal_nan` == true will give:
//   {T,   F,   T,   F,   T,   F,   F  }
//
// b. `equal_nan` == false will give:
//   {T,   F,   T,   F,   T,   T,   T  }
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are parallel_for, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 parallel_for，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 121-180
```cpp
//
template <typename scalar_t, bool equal_nan>
struct IsUnique {};

template <typename scalar_t>
struct IsUnique<scalar_t, false> {
  bool operator() (const scalar_t* data_ptr, int64_t i) {
    if (i == 0) { return true; }
    return c10::load(&data_ptr[i]) != c10::load(&data_ptr[i - 1]);
  }
};

template <typename scalar_t>
struct IsUnique<scalar_t, true> {
  bool operator() (const scalar_t* data_ptr, int64_t i) {
    if (i == 0) { return true; }
    return (c10::load(&data_ptr[i]) != c10::load(&data_ptr[i - 1]))
        && !(_isnan(data_ptr[i]) && _isnan(data_ptr[i - 1]));
  }
};

// NB: Unique implementation using sort
//
// The whole algo is taken from NumPy at numpy/lib/arraysetops.py
// which firstly do sort on the input sequence and then convert
// it to consecutive unique.
//
// Also improvement has been made upon the NumPy version: parallel
// `inverse_indices` and `counts` computation in a fused loop,
// which made this part almost a free launch.
//
// This kernel also implements a `equal_nan` flag which has same
// function as NumPy's unique. Currently this is always disabled.
//
// TODO: add `bool` specialization, use similar approach as UniqueCub
//
template <typename scalar_t, typename CompareOp>
std::tuple<Tensor, Tensor, Tensor> unique_cpu_sorted_template(
    const Tensor& self,
    const bool return_inverse,
    const bool return_counts,
    CompareOp is_unique) {
  const Tensor& input = self.contiguous();

  int64_t numel = input.numel();
  Tensor output = at::empty({0}, self.options());
  Tensor inverse_indices = at::empty({0}, self.options().dtype(kLong));
  Tensor counts = at::empty({0}, self.options().dtype(kLong));

  if (numel == 0) {
    if (return_inverse) {
      inverse_indices.resize_(input.sizes());
    }
    return std::make_tuple(output, inverse_indices, counts);
  }

  // index of first unique in each consecutive section
  // this is used to compute counts for parallelization purpose
  Tensor unique_index = at::empty({0}, self.options().dtype(kLong));
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are operator, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 operator，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 181-240
```cpp
  // original behavior with unique on scalar tensor
  // is to return a output size of ([1]), `flatten` here will do the job
  auto input_flattened = input.flatten();

  auto [input_sorted, indices] = input_flattened.sort();

  const scalar_t* input_sorted_data = input_sorted.const_data_ptr<scalar_t>();
  const int64_t* indices_data = indices.const_data_ptr<int64_t>();

  int num_threads = at::get_num_threads();
  std::vector<int64_t> unique_count_thread(num_threads, 0);
  std::vector<int64_t> offset_thread(num_threads, 0);

  const int64_t grain_size = at::internal::GRAIN_SIZE;

  // calculate unique count from each thread
  at::parallel_for(0, numel, grain_size, [&](int64_t begin, int64_t end) {
    int tid = at::get_thread_num();
    for (const auto i : c10::irange(begin, end)) {
      if (is_unique(input_sorted_data, i)) {
        unique_count_thread[tid]++;
      }
    }
  });

  // calculate thread offset in output and
  // `unique_count` records total count of uniques at last
  int64_t unique_count = 0;
  for (const auto t : c10::irange(num_threads)) {
    offset_thread[t] = unique_count;
    unique_count += unique_count_thread[t];
  }

  output.resize_({unique_count});
  scalar_t* output_data = output.data_ptr<scalar_t>();

  int64_t* inverse_indices_data = nullptr;
  if (return_inverse) {
    inverse_indices.resize_(input.sizes());
    inverse_indices_data = inverse_indices.data_ptr<int64_t>();
  }

  int64_t* counts_data = nullptr;
  int64_t* unique_index_data = nullptr;
  if (return_counts) {
    counts.resize_({unique_count});
    counts_data = counts.data_ptr<int64_t>();

    unique_index.resize_({unique_count + 1});
    unique_index_data = unique_index.data_ptr<int64_t>();
    unique_index_data[unique_count] = numel;
  }

  at::parallel_for(0, numel, grain_size, [&](int64_t begin, int64_t end) {
    int tid = at::get_thread_num();
    int64_t offset = offset_thread[tid];

    for (const auto i : c10::irange(begin, end)) {
      if (is_unique(input_sorted_data, i)) {
        output_data[offset] = c10::load(&input_sorted_data[i]);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are parallel_for, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 parallel_for，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 241-300
```cpp
        if (return_counts) {
          unique_index_data[offset] = i;
        }
        offset++;
      }

      if (return_inverse) {
        int64_t inverse_index = offset - 1;
        int64_t perm = indices_data[i];
        inverse_indices_data[perm] = inverse_index;
      }
    }
  });

  if (return_counts) {
    // do diff to get count
    at::parallel_for(0, unique_count, grain_size, [&](int64_t begin, int64_t end) {
      for (const auto i : c10::irange(begin, end)) {
        counts_data[i] = unique_index_data[i + 1] - unique_index_data[i];
      }
    });
  }
  return std::make_tuple(output, inverse_indices, counts);
}

template <typename scalar_t>
std::tuple<Tensor, Tensor, Tensor> unique_consecutive_cpu_template(
    const Tensor& self,
    const bool return_inverse,
    const bool return_counts) {
  const Tensor& input = self.contiguous();
  const scalar_t* input_data = input.const_data_ptr<scalar_t>();
  int64_t numel = input.numel();
  Tensor output = at::empty({numel}, input.options());
  Tensor inverse_indices = at::empty({0}, self.options().dtype(kLong));
  Tensor counts = at::empty({0}, self.options().dtype(kLong));

  if (return_inverse) {
    inverse_indices.resize_(input.sizes());
  }

  if (numel > 0) {
    scalar_t *output_data = output.data_ptr<scalar_t>();
    int64_t *inverse_data = inverse_indices.data_ptr<int64_t>();;
    int64_t *counts_data = nullptr;
    scalar_t last_value = c10::load(input_data);
    *output_data = last_value;

    if (return_counts) {
      counts.resize_({numel});
      counts_data = counts.data_ptr<int64_t>();
    }
    scalar_t *p = output_data;
    int64_t *q = counts_data;
    int64_t last = 0;
    if (return_inverse) {
      inverse_data[0] = 0;
    }
    for (const auto i : c10::irange(1, numel)) {
      const auto value = c10::load(&input_data[i]);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are parallel_for, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 parallel_for，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 301-360
```cpp
      if (value != last_value) {
        *(++p) = value;
        last_value = value;
        if (return_counts) {
          *(q++) = i - last;
          last = i;
        }
      }
      if (return_inverse) {
        inverse_data[i] = p - output_data;
      }
    }
    int64_t output_size = p - output_data + 1;
    if (return_counts) {
      *q = numel - last;
      counts.resize_({output_size});
    }
    output.resize_({output_size});
  }

  return std::make_tuple(output, inverse_indices, counts);
}

template<class ForwardIt>
ForwardIt _unique_dim_cpu_impl(ForwardIt first, ForwardIt last,
  std::vector<int64_t>& indices, Tensor inverse_indices_vec, Tensor counts) {
    if (first == last) {
      return last;
    }

    TORCH_INTERNAL_ASSERT(inverse_indices_vec.is_contiguous(),
        "_unique_dim_cpu_impl only support contiguous inverse_indices_vec");
    TORCH_INTERNAL_ASSERT(counts.is_contiguous(),
        "_unique_dim_cpu_impl only support contiguous counts");

    int64_t *indices_data = indices.data();
    int64_t *inverse_data = inverse_indices_vec.data_ptr<int64_t>();
    int64_t *counts_data = counts.data_ptr<int64_t>();

    ForwardIt result = first;
    ForwardIt previous = first;
    int64_t *current_counts = counts_data;
    inverse_data[*(indices_data++)] = 0;
    for (ForwardIt current = std::next(first); current != last; current++) {
      if (!at::equal(*current, *result)) {
        *(++result) = std::move(*current);
        *(current_counts++) = std::distance(previous, current);
        previous = current;
      }
      inverse_data[*(indices_data++)] = std::distance(first, result);
    }
    *current_counts = std::distance(previous, last);
    return ++result;
  }

template <typename scalar_t>
std::tuple<Tensor, Tensor, Tensor> _unique_dim_cpu_template(
    const Tensor& self,
    const int64_t dim,
    const bool consecutive,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 361-420
```cpp
    const bool return_inverse,
    const bool return_counts) {

    auto sizes = self.sizes().vec();
    // check how many zero dimensions exist
    auto num_zero_dims = std::count(sizes.begin(), sizes.end(), 0);

    // tensor is not well formed as it has 0 sized dimensions
    if (self.size(dim) == 0){
      TORCH_CHECK(
          num_zero_dims == 1,
          "Number of zero sized dimensions is more than one, so unique cannot be applied ")
      Tensor output = at::empty(sizes, self.options());
      Tensor inverse_indices =
          at::empty({0}, self.options().dtype(kLong));
      Tensor counts = at::empty({0}, self.options().dtype(kLong));

      return std::make_tuple(output, inverse_indices, counts);
    }

    TORCH_CHECK(num_zero_dims == 0,
    "There are 0 sized dimensions, and they aren't selected, so unique cannot be applied");

  // reshape tensor as [dim, -1]
  Tensor input_flat = self.moveaxis(dim, 0);
  auto orig_sizes = input_flat.sizes().vec();
  input_flat = input_flat.contiguous().view({input_flat.size(0), -1});

  std::vector<int64_t> indices(input_flat.size(0));
  std::iota(indices.begin(), indices.end(), 0);
  int64_t numel = input_flat.size(1);
  const scalar_t* input_flat_ptr = ((const scalar_t*)input_flat.const_data_ptr());

  // sort indices using data
  if (!consecutive) {
    std::sort(indices.begin(), indices.end(),
      [&](int64_t a, int64_t b) -> bool {
        for (const auto i : c10::irange(numel)) {
          scalar_t lhs = c10::load(&input_flat_ptr[i + a * numel]);
          scalar_t rhs = c10::load(&input_flat_ptr[i + b * numel]);
          if (lhs < rhs) {
            return true;
          } else if (lhs > rhs) {
            return false;
          }
        }
        return false;
      });
  }

  Tensor input_sorted;
  if (!consecutive) {
    input_sorted = at::empty(input_flat.sizes(), input_flat.options());
    for (const auto i : c10::irange(indices.size())) {
      input_sorted[i] = input_flat[indices[i]];
    }
  } else {
    input_sorted = input_flat;
  }
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 421-480
```cpp
  Tensor inverse_indices = at::empty(indices.size(), self.options().dtype(kLong));
  Tensor counts = at::zeros(indices.size(), self.options().dtype(kLong));
  std::vector<Tensor> input_unbind = at::unbind(input_sorted, 0);
  auto last = _unique_dim_cpu_impl(
    input_unbind.begin(), input_unbind.end(), indices, inverse_indices, counts);
  input_unbind.erase(last, input_unbind.end());
  counts = at::narrow(counts, 0, 0, input_unbind.size());

  // reshape back
  auto output = at::stack(input_unbind, 0);
  auto new_sizes = std::vector<int64_t>(std::move(orig_sizes));
  new_sizes[0] = -1;
  output = output.view(new_sizes);
  output = output.moveaxis(0, dim);

  return std::make_tuple(std::move(output), std::move(inverse_indices), std::move(counts));
}

} // namespace

std::tuple<Tensor, Tensor>
_unique_cpu(const Tensor& self, const bool sorted, const bool return_inverse) {
  if (self.scalar_type() == kBool) {
    auto [output, inverse, _] = unique_cpu_bool_template(
        self, return_inverse, /* return_counts */false);
    return std::make_tuple(output, inverse);
  }
  return AT_DISPATCH_V2(self.scalar_type(), "unique", [&] AT_WRAP({
    // The current CPU implementation of unique always sort due to
    // this is faster than hash table
    auto [output, inverse, _] = unique_cpu_sorted_template<scalar_t>(
        self, return_inverse, /* return_counts */false, IsUnique<scalar_t, /* equal_nan */false>());
    return std::make_tuple(output, inverse);
  }), AT_EXPAND(AT_ALL_TYPES), kBFloat16, kHalf, AT_EXPAND(AT_BAREBONES_UNSIGNED_TYPES));
}

std::tuple<Tensor, Tensor, Tensor>
_unique2_cpu(const Tensor& self, const bool sorted, const bool return_inverse, const bool return_counts) {
  if (self.scalar_type() == kBool) {
    return unique_cpu_bool_template(self, return_inverse, return_counts);
  }
  return AT_DISPATCH_V2(self.scalar_type(), "unique", AT_WRAP([&] {
    // The current CPU implementation of unique always sort due to
    // this is faster than hash table
    return unique_cpu_sorted_template<scalar_t>(
        self, return_inverse, return_counts, IsUnique<scalar_t, /* equal_nan */ false>());
  }), AT_EXPAND(AT_ALL_TYPES), kBFloat16, kHalf, AT_EXPAND(AT_BAREBONES_UNSIGNED_TYPES));
}

std::tuple<Tensor, Tensor, Tensor>
unique_dim_cpu(const Tensor& self, const int64_t dim, const bool sorted, const bool return_inverse, const bool return_counts) {
  return AT_DISPATCH_V2(self.scalar_type(), "unique_dim", AT_WRAP([&] {
    // The current implementation using `dim` always sorts due to unhashable tensors
    return _unique_dim_cpu_template<scalar_t>(self, dim, false, return_inverse, return_counts);
  }), AT_EXPAND(AT_ALL_TYPES), kBFloat16, kBool, kHalf, AT_EXPAND(AT_BAREBONES_UNSIGNED_TYPES));
}

std::tuple<Tensor, Tensor, Tensor>
unique_dim_consecutive_cpu(const Tensor& self, const int64_t dim, const bool return_inverse, const bool return_counts) {
  return AT_DISPATCH_V2(self.scalar_type(), "unique_dim", AT_WRAP([&] {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are _unique_cpu, _unique2_cpu, unique_dim_cpu, unique_dim_consecutive_cpu, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 _unique_cpu, _unique2_cpu, unique_dim_cpu, unique_dim_consecutive_cpu，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 481-495
```cpp
    return _unique_dim_cpu_template<scalar_t>(self, dim, true, return_inverse, return_counts);
  }), AT_EXPAND(AT_ALL_TYPES), kBFloat16, kBool, kHalf, AT_EXPAND(AT_BAREBONES_UNSIGNED_TYPES));
}

std::tuple<Tensor, Tensor, Tensor>
unique_consecutive_cpu(const Tensor& self, const bool return_inverse, const bool return_counts, std::optional<int64_t> dim) {
  if (!dim.has_value() || (dim.value() == 0 && self.dim() == 1)) {
    return AT_DISPATCH_V2(self.scalar_type(), "unique", AT_WRAP([&] {
      return unique_consecutive_cpu_template<scalar_t>(self, return_inverse, return_counts);
    }), AT_EXPAND(AT_ALL_TYPES), kBFloat16, kBool, kHalf, AT_EXPAND(AT_BAREBONES_UNSIGNED_TYPES));
  }
  return unique_dim_consecutive_cpu(self, dim.value(), return_inverse, return_counts);
}

}  // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are unique_consecutive_cpu, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 unique_consecutive_cpu，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

## Key Concepts / 关键概念

- EN: TensorIterator is used to describe elementwise or reduction-style iteration.
- CN: 使用 TensorIterator 描述逐元素或归约式遍历。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Template-based reuse helps share logic across scalar types, layouts, or backends.
- CN: 基于模板的复用帮助在不同标量类型、布局或后端之间共享逻辑。
- EN: Notable symbols: unique_cpu_bool_template, parallel_for, operator, unique_cpu_sorted_template, unique_consecutive_cpu_template, _unique_dim_cpu_impl, _unique_dim_cpu_template, TORCH_CHECK.
- CN: 重要符号：unique_cpu_bool_template, parallel_for, operator, unique_cpu_sorted_template, unique_consecutive_cpu_template, _unique_dim_cpu_impl, _unique_dim_cpu_template, TORCH_CHECK。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/core/Tensor.h, ATen/Dispatch_v2.h, ATen/Parallel.h, ATen/native/TensorIterator.h, c10/util/irange.h, c10/util/Load.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/_unique2_native.h, ATen/ops/_unique_native.h, ATen/ops/empty.h, ATen/ops/equal.h`.
- CN: 主要内部头文件：`ATen/core/Tensor.h, ATen/Dispatch_v2.h, ATen/Parallel.h, ATen/native/TensorIterator.h, c10/util/irange.h, c10/util/Load.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/_unique2_native.h, ATen/ops/_unique_native.h, ATen/ops/empty.h, ATen/ops/equal.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `unique_cpu_bool_template, parallel_for, operator, unique_cpu_sorted_template, unique_consecutive_cpu_template, _unique_dim_cpu_impl, _unique_dim_cpu_template, TORCH_CHECK, _unique_cpu, _unique2_cpu`.
- CN: 实现围绕 `unique_cpu_bool_template, parallel_for, operator, unique_cpu_sorted_template, unique_consecutive_cpu_template, _unique_dim_cpu_impl, _unique_dim_cpu_template, TORCH_CHECK, _unique_cpu, _unique2_cpu` 等符号展开。
