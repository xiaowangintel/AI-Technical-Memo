# EmbeddingBag.h — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/mps/kernels/EmbeddingBag.h`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the MPS backend in PyTorch ATen native code and focuses on embedding bag; it mainly declares interfaces, helper types, and inline utilities.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的MPS 后端，主题聚焦于 embedding bag；其主要作用是声明接口、辅助类型以及内联工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#pragma once
#include <c10/metal/common.h>

#ifdef __METAL__
enum class EmbeddingBagMode { SUM = 0, MEAN, MAX };
#else
#include <ATen/native/EmbeddingBag.h>
using at::native::EmbeddingBagMode;
#endif

template <typename idx_type_t = uint32_t>
struct EmbeddingBagParams {
  ::c10::metal::array<idx_type_t, 2> weight_strides;
  ::c10::metal::array<idx_type_t, 2> output_strides;
  ::c10::metal::array<idx_type_t, 2> max_indices_strides;

  bool use_per_sample_weights;
  idx_type_t per_sample_weights_stride;

  idx_type_t num_indices;
  idx_type_t num_bags;
  idx_type_t feature_size;
  idx_type_t num_weights;

  EmbeddingBagMode mode;
  int64_t padding_idx;
};

template <typename idx_type_t = uint32_t>
struct EmbeddingBagBackwardParams {
```
- EN: Lines 1-30 pull in 2 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- CN: 第 1-30 行引入了 2 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。

### Lines 31-48
```cpp
  ::c10::metal::array<idx_type_t, 2> weight_grad_strides;
  ::c10::metal::array<idx_type_t, 2> output_grad_strides;
  ::c10::metal::array<idx_type_t, 2> max_indices_strides;
  bool use_per_sample_weights;
  idx_type_t per_sample_weights_stride;
  idx_type_t feature_size;
  EmbeddingBagMode mode;
  int64_t padding_idx;
};

template <typename idx_type_t = uint32_t>
struct EmbeddingBagPerSampleWeightsBackwardParams {
  ::c10::metal::array<idx_type_t, 2> output_grad_strides;
  ::c10::metal::array<idx_type_t, 2> weight_strides;
  idx_type_t per_sample_weights_grad_stride;
  idx_type_t feature_size;
  int64_t padding_idx;
};
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。

## Key Concepts / 关键概念

- EN: Backend focus: MPS backend.
- CN: 后端重点：MPS 后端。
- EN: Template-based reuse helps share logic across scalar types, layouts, or backends.
- CN: 基于模板的复用帮助在不同标量类型、布局或后端之间共享逻辑。

## Dependencies / 依赖关系

- EN: Primary internal headers: `c10/metal/common.h, ATen/native/EmbeddingBag.h`.
- CN: 主要内部头文件：`c10/metal/common.h, ATen/native/EmbeddingBag.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
