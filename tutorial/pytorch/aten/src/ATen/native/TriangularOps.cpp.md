# TriangularOps.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/TriangularOps.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on triangular ops; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 triangular ops；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS
#include <ATen/core/Tensor.h>
#include <ATen/Dispatch.h>
#include <ATen/Parallel.h>
#include <ATen/TensorMeta.h>
#include <ATen/native/TriangularOpsUtils.h>
#include <ATen/TensorSubclassLikeUtils.h>
#include <c10/util/irange.h>
#include <c10/util/Exception.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/arange.h>
#include <ATen/ops/empty_like.h>
#include <ATen/ops/trace_backward_native.h>
#include <ATen/ops/tril_native.h>
#include <ATen/ops/triu_native.h>
#include <ATen/ops/zeros.h>
#endif

namespace at::meta {

TORCH_META_FUNC(tril)(const Tensor& self, int64_t k) {
  TORCH_CHECK(self.dim() >= 2, "tril: input tensor must have at least 2 dimensions")
  set_output_raw_strided(0, self.sizes(), {}, self.options());
}

TORCH_META_FUNC(triu)(const Tensor& self, int64_t k) {
```
- EN: Lines 1-30 pull in 16 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are TORCH_META_FUNC, TORCH_CHECK, concentrating a specific part of the operator behavior.
- CN: 第 1-30 行引入了 16 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 TORCH_META_FUNC, TORCH_CHECK，它们承载了某一部分算子行为的核心逻辑。

### Lines 31-60
```cpp
  TORCH_CHECK(self.dim() >= 2, "triu: input tensor must have at least 2 dimensions")
  set_output_raw_strided(0, self.sizes(), {}, self.options());
}

}  // namespace at::meta

namespace at::native {
namespace {

// ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ triu/tril ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

template <typename scalar_t>
void apply_triu_tril_single(
    scalar_t* result,
    const scalar_t* self,
    bool inplace,
    int64_t k,
    int64_t n,
    int64_t m,
    int64_t res_row_stride,
    int64_t res_col_stride,
    int64_t self_row_stride,
    int64_t self_col_stride,
    bool upper) {
  constexpr int64_t zero = 0;
  k = std::clamp(k, -n, m); // Clamp k to [-n, m] to prevent i + k arithmetic overflow, especially if k approaches INT64_MAX/INT64_MIN.

  if (upper) {
    parallel_for(0, n, 0, [&](int64_t start, int64_t end) {
      for (int64_t i : c10::irange(start, end)) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are TORCH_CHECK, parallel_for, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 TORCH_CHECK, parallel_for，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 61-90
```cpp
        for (int64_t j = 0; j < std::min(m, i + k); j++) {
          result[i * res_row_stride + j * res_col_stride] = static_cast<scalar_t>(0);
        }
        if (!inplace) {  // copy the rest of the self if not inplace
          for (int64_t j = std::max(zero, i + k); j < m; j++) {
            result[i * res_row_stride + j * res_col_stride] = c10::load(&self[i * self_row_stride + j * self_col_stride]);
          }
        }
      }
    });
  } else {
    parallel_for(0, n, 0, [&](int64_t start, int64_t end) {
      for (int64_t i : c10::irange(start, end)) {
        for (int64_t j = std::max(zero, i + k + 1); j < m; j++) {
          result[i * res_row_stride + j * res_col_stride] = static_cast<scalar_t>(0);
        }
        if (!inplace) {  // copy the rest of the self if not inplace
          for (int64_t j = zero; j < std::min(m, i + k + 1); j++) {
            result[i * res_row_stride + j * res_col_stride] = c10::load(&self[i * self_row_stride + j * self_col_stride]);
          }
        }
      }
    });
  }
}

template <typename scalar_t>
void apply_triu_tril(const Tensor& result, const Tensor& self, bool inplace, int64_t k, bool upper) {
  auto n = self.size(-2);
  auto m = self.size(-1);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are parallel_for, apply_triu_tril, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 parallel_for, apply_triu_tril，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 91-120
```cpp
  auto self_data = self.const_data_ptr<scalar_t>();
  auto self_stride = (self.dim() > 2 && self.stride(-3) > 0) ? self.stride(-3) : 1;
  auto batchsize = batchCountTrilTriu(result);
  auto self_row_stride = self.stride(-2);
  auto self_col_stride = self.stride(-1);

  auto result_data = result.data_ptr<scalar_t>();
  int64_t result_stride = 0, result_row_stride = 0, result_col_stride = 0;
  if (result_data != self_data) {
    result_stride = (result.dim() > 2 && result.stride(-3) > 0) ? result.stride(-3) : 1;
    result_row_stride = result.stride(-2);
    result_col_stride = result.stride(-1);
  } else {
    result_stride = self_stride;
    result_row_stride = self_row_stride;
    result_col_stride = self_col_stride;
  }

  parallel_for(0, batchsize, 0, [&](int64_t start, int64_t end) {
    for (const auto b : c10::irange(start, end)) {
      const scalar_t* self_batch = &self_data[b * self_stride];
      scalar_t* result_batch = &result_data[b * result_stride];
      apply_triu_tril_single<scalar_t>(
          result_batch,
          self_batch,
          inplace,
          k,
          n,
          m,
          result_row_stride,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are parallel_for, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 parallel_for，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 121-150
```cpp
          result_col_stride,
          self_row_stride,
          self_col_stride,
          upper);
    }
  });
}

struct UpperTriangle {
  static constexpr const char* op_name = "triu";
  static constexpr bool upper = true;
};

struct LowerTriangle {
  static constexpr const char *op_name = "tril";
  static constexpr bool upper = false;
};

template <typename Triangle>
void compute_triu_tril(const Tensor& self, int64_t k, const Tensor &result) {
  if (self.numel() == 0) {
    return;
  }

  bool inplace_op = self.is_same(result);

  bool inplace_update = false;
  Tensor self_c;
  std::tie(inplace_update, self_c) = checkTrilTriuBatchContiguous(self, inplace_op);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are compute_triu_tril, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 compute_triu_tril，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 151-180
```cpp
  Tensor result_c;
  if (inplace_op && !inplace_update) {
    result_c = at::empty_like(result, LEGACY_CONTIGUOUS_MEMORY_FORMAT);
  } else {
    result_c = result;
  }

  AT_DISPATCH_ALL_TYPES_AND_COMPLEX_AND4(
      ScalarType::ComplexHalf,
      ScalarType::BFloat16,
      ScalarType::Half,
      ScalarType::Bool,
      self.scalar_type(),
      Triangle::op_name,
      [&]{
        apply_triu_tril<scalar_t>(result_c, self_c, inplace_op && inplace_update, k, Triangle::upper);
      });

  if (inplace_op && !inplace_update) {
    result.copy_(result_c);
  }
}

}  // namespace

TORCH_IMPL_FUNC(tril_cpu)(const Tensor& self, int64_t k, const Tensor &result) {
  compute_triu_tril<LowerTriangle>(self, k, result);
}

TORCH_IMPL_FUNC(triu_cpu)(const Tensor& self, int64_t k, const Tensor &result) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are TORCH_IMPL_FUNC, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 TORCH_IMPL_FUNC，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 181-202
```cpp
  compute_triu_tril<UpperTriangle>(self, k, result);
}

Tensor trace_backward_symint(const Tensor& grad, c10::SymIntArrayRef sizes) {
  TORCH_CHECK(sizes.size() == 2, "expected matrix input");

  auto grad_input = at::zeros_symint(sizes[0] * sizes[1], grad.options());
  // Only min(M, N) diagonal elements exist for an (M, N) matrix
  auto diag_size = std::min(sizes[0], sizes[1]);
  auto step = sizes[1] + 1;
  auto indices = at::arange(0, diag_size * step, step, grad.options().dtype(at::kLong));
  // for composite compliance, use out-of-place variant of
  // `index_fill` if grad tensor is a Tensor Subclass.
  if (isTensorSubclassLike(grad)) {
    grad_input = grad_input.index_fill(0, indices, grad);
  } else {
    grad_input.index_fill_(0, indices, grad);
  }
  return grad_input.view_symint(sizes);
}

}  // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are trace_backward_symint, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 trace_backward_symint，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

## Key Concepts / 关键概念

- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Template-based reuse helps share logic across scalar types, layouts, or backends.
- CN: 基于模板的复用帮助在不同标量类型、布局或后端之间共享逻辑。
- EN: Notable symbols: TORCH_META_FUNC, TORCH_CHECK, apply_triu_tril_single, parallel_for, apply_triu_tril, compute_triu_tril, TORCH_IMPL_FUNC, trace_backward_symint.
- CN: 重要符号：TORCH_META_FUNC, TORCH_CHECK, apply_triu_tril_single, parallel_for, apply_triu_tril, compute_triu_tril, TORCH_IMPL_FUNC, trace_backward_symint。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/core/Tensor.h, ATen/Dispatch.h, ATen/Parallel.h, ATen/TensorMeta.h, ATen/native/TriangularOpsUtils.h, ATen/TensorSubclassLikeUtils.h, c10/util/irange.h, c10/util/Exception.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/arange.h, ATen/ops/empty_like.h`.
- CN: 主要内部头文件：`ATen/core/Tensor.h, ATen/Dispatch.h, ATen/Parallel.h, ATen/TensorMeta.h, ATen/native/TriangularOpsUtils.h, ATen/TensorSubclassLikeUtils.h, c10/util/irange.h, c10/util/Exception.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/arange.h, ATen/ops/empty_like.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `TORCH_META_FUNC, TORCH_CHECK, apply_triu_tril_single, parallel_for, apply_triu_tril, compute_triu_tril, TORCH_IMPL_FUNC, trace_backward_symint`.
- CN: 实现围绕 `TORCH_META_FUNC, TORCH_CHECK, apply_triu_tril_single, parallel_for, apply_triu_tril, compute_triu_tril, TORCH_IMPL_FUNC, trace_backward_symint` 等符号展开。
