# TensorIteratorReduce.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/TensorIteratorReduce.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on tensor iterator reduce; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 tensor iterator reduce；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS
#include <ATen/Parallel.h>
#include <ATen/TensorIterator.h>
#include <ATen/TensorIteratorInternal.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#else
#include <ATen/ops/empty.h>
#endif

#include <c10/util/irange.h>

/// Contains the implementation of parallel reductions in TensorIterator.

namespace at {

using loop2d_t = TensorIteratorBase::loop2d_t;

static bool use_two_pass_reduction(TensorIteratorBase& iter);
static void two_pass_reduction(TensorIteratorBase& iter, loop2d_t loop);
static void parallel_dim_reduction(TensorIteratorBase& iter, loop2d_t loop);

void TensorIteratorBase::parallel_reduce(loop2d_t loop) {
  TORCH_CHECK(
      ntensors() == 2,
      "parallel_reduce only supports one input and one output");
  int64_t numel = this->numel();
  if (numel < at::internal::GRAIN_SIZE || at::get_num_threads() == 1 ||
      at::in_parallel_region()) {
```
- EN: Lines 1-30 pull in 6 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are parallel_reduce, in_parallel_region, concentrating a specific part of the operator behavior.
- CN: 第 1-30 行引入了 6 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 parallel_reduce, in_parallel_region，它们承载了某一部分算子行为的核心逻辑。

### Lines 31-60
```cpp
    serial_for_each(loop, {0, numel});
  } else if (use_two_pass_reduction(*this)) {
    two_pass_reduction(*this, loop);
  } else {
    parallel_dim_reduction(*this, loop);
  }
}

static bool use_two_pass_reduction(TensorIteratorBase& iter) {
  return iter.output(0).numel() == 1;
}

static void two_pass_reduction(TensorIteratorBase& iter, loop2d_t loop) {
  const int max_threads = at::get_num_threads();

  const auto& dst = iter.output(0);
  auto unsqueezed = dst.unsqueeze(0);
  auto buffer_shape = DimVector(unsqueezed.sizes());
  buffer_shape[0] = max_threads;
  auto buffer = at::empty(buffer_shape, dst.options());
  // Fill with the identity
  buffer.copy_(unsqueezed);

  auto buffer_stride = buffer.strides()[0] * buffer.element_size();
  auto buffer_0 = buffer[0];
  auto first_reduce = TensorIterator::reduce_op(buffer_0, iter.input(0));
  TORCH_INTERNAL_ASSERT(first_reduce.output(0).is_alias_of(buffer_0));

  at::parallel_for(
      0, iter.numel(), internal::GRAIN_SIZE, [&](int64_t begin, int64_t end) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are use_two_pass_reduction, two_pass_reduction, numel, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 use_two_pass_reduction, two_pass_reduction, numel，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 61-90
```cpp
        const auto thread_num = at::get_thread_num();
        auto shape = first_reduce.shape();
        auto strides = first_reduce.get_strides();

        // Bump output ptr so each thread has its own output slice
        auto base_ptrs = first_reduce.get_base_ptrs();
        base_ptrs[0] += buffer_stride * thread_num;

        at::internal::serial_for_each(
            shape,
            strides,
            base_ptrs.data(),
            base_ptrs.size(),
            loop,
            {begin, end});
      });

  auto final_reduce = TensorIterator::reduce_op(unsqueezed, buffer);
  final_reduce.for_each(loop);
}

/// Chooses a dimension over which to parallelize. Prefers the outer-most
/// dimension that's larger than the number of available threads.
static int find_split_dim(TensorIteratorBase& iter) {
  int num_threads = at::get_num_threads();
  auto shape = iter.shape();

  // start with the outer-most dimension
  int best_dim = iter.ndim() - 1;
  for (int dim = best_dim; dim >= 0 && !iter.is_dim_reduced(dim); dim--) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are find_split_dim, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 find_split_dim，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 91-120
```cpp
    if (shape[dim] >= num_threads) {
      return dim;
    } else if (shape[dim] > shape[best_dim]) {
      best_dim = dim;
    }
  }

  AT_ASSERT(!iter.is_dim_reduced(best_dim));
  return best_dim;
}

static std::tuple<int64_t, int64_t> round_columns(
    TensorIteratorBase& iter,
    int dim,
    int multiple,
    int64_t begin,
    int64_t end) {
  begin = begin - (begin % multiple);
  if (end != iter.shape()[dim]) {
    // only round the 'end' column down if it's not the final column
    end = end - (end % multiple);
  }
  return std::make_tuple(begin, end);
}

static void parallel_dim_reduction(TensorIteratorBase& iter, loop2d_t loop) {
  AT_ASSERT(iter.ndim() >= 1);
  int dim = find_split_dim(iter);
  int64_t cols = iter.shape()[dim];
  int element_size = iter.element_size(/*arg=*/1);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are parallel_dim_reduction, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 parallel_dim_reduction，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 121-150
```cpp

  bool should_round_columns = iter.strides(1)[dim] == element_size;
  at::parallel_for(0, cols, 1, [&](int64_t begin, int64_t end) {
    if (should_round_columns) {
      // round columns to multiples of 128 bytes if adjacent columns are
      // contiguous in memory.
      int64_t cols_per_128_bytes = 128 / element_size;
      std::tie(begin, end) =
          round_columns(iter, dim, cols_per_128_bytes, begin, end);
    }
    if (begin == end) {
      return;
    }
    auto sub_iter = TensorIterator(iter);
    sub_iter.narrow(dim, begin, end - begin);
    sub_iter.for_each(loop);
  });
}

void TensorIteratorBase::foreach_reduced_elt(
    loop_subiter_t loop,
    bool parallelize) {
  AT_ASSERT(ninputs() == 1);
  AT_ASSERT(noutputs() >= 1);

  auto shape = this->shape();
  if (output(0).numel() == 0) {
    return;
  }
  if (output(0).numel() == 1) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are parallel_for, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 parallel_for，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 151-180
```cpp
    loop(*this);
  } else if (
      numel() < at::internal::GRAIN_SIZE || at::get_num_threads() == 1 ||
      at::in_parallel_region() || !parallelize) {
    auto reduce_dims = num_reduce_dims();

    auto non_reduced_shape =
        shape.slice(reduce_dims, shape.size() - reduce_dims);

    int64_t non_reduced_numel = 1;
    for (const auto i : non_reduced_shape) {
      non_reduced_numel *= i;
    }
    DimCounter dims{non_reduced_shape, {0, non_reduced_numel}};
    while (!dims.is_done()) {
      TensorIterator reduced = *this;
      reduced.select_all_keeping_dim(reduce_dims, dims.values);
      loop(reduced);
      dims.increment({1, 1});
    }
  } else {
    int dim = find_split_dim(*this);
    int64_t cols = shape[dim];
    at::parallel_for(0, cols, 1, [&](int64_t begin, int64_t end) {
      if (begin == end) {
        return;
      }

      TensorIterator sub_iter(*this);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are in_parallel_region, parallel_for, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 in_parallel_region, parallel_for，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 181-195
```cpp
      sub_iter.narrow(dim, begin, end - begin);
      // On some broken setups, `#ifdef _OPENMP` is true,
      // and `get_num_threads` returns > 1, but
      // `#pragma omp parallel` is ignored.
      // There is no API to check for this, so we need to explicitly
      // stop trying to parallelize if we've already gotten here.
      //
      // (If we are on one of those broken setups, we will
      //  only have one thread here, and end - begin == cols.)
      sub_iter.foreach_reduced_elt(loop, false);
    });
  }
}

} // namespace at
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Comments in this range document invariants, backend limits, or rationale that clarify why the surrounding implementation is structured this way.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段中的注释记录了不变式、后端限制或设计动机，帮助解释周围实现为何采用当前结构。

## Key Concepts / 关键概念

- EN: TensorIterator is used to describe elementwise or reduction-style iteration.
- CN: 使用 TensorIterator 描述逐元素或归约式遍历。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: parallel_reduce, use_two_pass_reduction, two_pass_reduction, parallel_for, find_split_dim, round_columns, parallel_dim_reduction, foreach_reduced_elt.
- CN: 重要符号：parallel_reduce, use_two_pass_reduction, two_pass_reduction, parallel_for, find_split_dim, round_columns, parallel_dim_reduction, foreach_reduced_elt。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/Parallel.h, ATen/TensorIterator.h, ATen/TensorIteratorInternal.h, ATen/Functions.h, ATen/ops/empty.h, c10/util/irange.h`.
- CN: 主要内部头文件：`ATen/Parallel.h, ATen/TensorIterator.h, ATen/TensorIteratorInternal.h, ATen/Functions.h, ATen/ops/empty.h, c10/util/irange.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `parallel_reduce, use_two_pass_reduction, two_pass_reduction, parallel_for, find_split_dim, round_columns, parallel_dim_reduction, foreach_reduced_elt, numel`.
- CN: 实现围绕 `parallel_reduce, use_two_pass_reduction, two_pass_reduction, parallel_for, find_split_dim, round_columns, parallel_dim_reduction, foreach_reduced_elt, numel` 等符号展开。
