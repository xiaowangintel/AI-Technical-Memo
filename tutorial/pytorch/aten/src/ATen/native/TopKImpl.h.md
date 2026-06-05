# TopKImpl.h — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/TopKImpl.h`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on top k impl; it mainly declares interfaces, helper types, and inline utilities.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 top k impl；其主要作用是声明接口、辅助类型以及内联工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#pragma once
#include <ATen/core/TensorAccessor.h>
#include <ATen/NumericUtils.h>

namespace at::native {

#ifdef CPU_CAPABILITY
inline namespace CPU_CAPABILITY {
#else
inline namespace DEFAULT {
#endif

// Core topk loop, shared between CPU and QuantizedCPU
template <typename scalar_t, typename accscalar_t>
void topk_impl_loop(
    const int64_t mode_values_stride,
    const int64_t mode_indices_stride,
    const int64_t tmp_values_stride,
    const int64_t k,
    const int64_t dim_size,
    const bool largest,
    const bool sorted,
    char** data, const int64_t* strides, const int64_t n) {

  // If k is zero, then output values and indices are empty tensors
  // So iterating over other dims is pointless
  if (k == 0) {
    return;
  }
  using elem_t = std::pair<accscalar_t, int64_t>;
```
- EN: Lines 1-30 pull in 2 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 第 1-30 行引入了 2 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 31-60
```cpp
  std::vector<elem_t> queue(dim_size);
  for (const auto i : c10::irange(n)) {
    TensorAccessor<scalar_t, 1> mode_values(
        reinterpret_cast<scalar_t*>(data[0] + i * strides[0]),
        &k, &mode_values_stride);
    TensorAccessor<int64_t, 1> mode_indices(
        reinterpret_cast<int64_t*>(data[1] + i * strides[1]),
        &k, &mode_indices_stride);
    TensorAccessor<const scalar_t, 1> tmp_values(
        reinterpret_cast<scalar_t*>(data[2] + i * strides[2]),
        &dim_size, &tmp_values_stride);

    auto n_2 = dim_size;
    auto use_partial_sort = k * 64 <= n_2;

    for (const auto j : c10::irange(n_2)) {
      queue[j].first = tmp_values[j];
      queue[j].second = j;
    }

    // we want nan to be sorted as top for numpy compatibility
    if (use_partial_sort) {
      if (largest) {
        std::partial_sort(queue.begin(), queue.begin() + k, queue.end(),
          [](const elem_t& x, const elem_t& y) -> bool {
            return ((_isnan<accscalar_t>(x.first) && !_isnan<accscalar_t>(y.first)) || (x.first > y.first));
          });
      } else {
        std::partial_sort(queue.begin(), queue.begin() + k, queue.end(),
          [](const elem_t& x, const elem_t& y) -> bool {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 61-90
```cpp
            return ((!_isnan<accscalar_t>(x.first) && _isnan<accscalar_t>(y.first)) || (x.first < y.first));
          });
      }
    } else {
      if (largest) {
        std::nth_element(queue.begin(), queue.begin() + k - 1, queue.end(),
          [](const elem_t& x, const elem_t& y) -> bool {
            return ((_isnan<accscalar_t>(x.first) && !_isnan<accscalar_t>(y.first)) || (x.first > y.first));
          });
        if (sorted) {
          std::sort(queue.begin(), queue.begin() + k - 1,
            [](const elem_t& x, const elem_t& y) -> bool {
              return ((_isnan<accscalar_t>(x.first) && !_isnan<accscalar_t>(y.first)) || (x.first > y.first));
            });
        }
      } else {
        std::nth_element(queue.begin(), queue.begin() + k -1, queue.end(),
          [](const elem_t& x, const elem_t& y) -> bool {
            return ((!_isnan<accscalar_t>(x.first) && _isnan<accscalar_t>(y.first)) || (x.first < y.first));
          });
        if (sorted) {
          std::sort(queue.begin(), queue.begin() + k -1,
            [](const elem_t& x, const elem_t& y) -> bool {
              return ((!_isnan<accscalar_t>(x.first) && _isnan<accscalar_t>(y.first)) || (x.first < y.first));
            });
        }
      }
    }

    for (const auto j : c10::irange(k)) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 91-98
```cpp
      mode_values[j] = queue[j].first;
      mode_indices[j] = queue[j].second;
    }
  }
}

} // namespace CPU_CAPABILITY
} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。

## Key Concepts / 关键概念

- EN: Template-based reuse helps share logic across scalar types, layouts, or backends.
- CN: 基于模板的复用帮助在不同标量类型、布局或后端之间共享逻辑。
- EN: Notable symbols: topk_impl_loop.
- CN: 重要符号：topk_impl_loop。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/core/TensorAccessor.h, ATen/NumericUtils.h`.
- CN: 主要内部头文件：`ATen/core/TensorAccessor.h, ATen/NumericUtils.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `topk_impl_loop`.
- CN: 实现围绕 `topk_impl_loop` 等符号展开。
