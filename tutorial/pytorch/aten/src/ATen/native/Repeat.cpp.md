# Repeat.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/Repeat.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on repeat; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 repeat；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS
#include <ATen/core/Tensor.h>
#include <ATen/Dispatch.h>
#include <ATen/Parallel.h>
#include <ATen/native/Repeat.h>
#include <c10/util/irange.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/empty.h>
#include <ATen/ops/repeat_interleave.h>
#include <ATen/ops/repeat_interleave_native.h>
#endif

template <typename index_t>
static void compute_cpu(
    const index_t* repeat_ptr,
    const int64_t* cumsum_ptr,
    index_t* result_ptr,
    int64_t size,
    int64_t result_size) {
  TORCH_CHECK(
      (result_size == cumsum_ptr[size - 1]),
      "allocated size does not match required size");
  at::parallel_for(0, size, 1, [&](int64_t i_begin, int64_t i_end) {
    for (const auto i : c10::irange(i_begin, i_end)) {
      int64_t end = cumsum_ptr[i];
      index_t size = repeat_ptr[i];
```
- EN: Lines 1-30 pull in 10 header dependencies, defining the compilation surface for this segment.
- EN: The main callable definitions or declarations in this block are parallel_for, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 第 1-30 行引入了 10 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段的主要可调用定义或声明包括 parallel_for，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 31-60
```cpp
      TORCH_CHECK((size >= 0), "repeats can not be negative");
      int64_t start = end - size;
      for (const auto j : c10::irange(start, end)) {
        result_ptr[j] = i;
      }
    }
  });
}

namespace at::native {

Tensor repeat_interleave_cpu(
    const Tensor& repeat,
    std::optional<int64_t> output_size) {
  Tensor output;
  AT_DISPATCH_INDEX_TYPES(repeat.scalar_type(), "repeat_interleave_cpu", [&]() {
    output = repeat_interleave_common<index_t, compute_cpu<index_t>>(
        repeat, output_size);
  });

  return output;
}

Tensor repeat_interleave_symint(
    const Tensor& self,
    const Tensor& repeats,
    std::optional<int64_t> dim,
    std::optional<SymInt> output_size) {
  Tensor input = self;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are AT_DISPATCH_INDEX_TYPES, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 AT_DISPATCH_INDEX_TYPES，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 61-90
```cpp
  // Store conj and neg bits
  const auto conj = input.is_conj();
  if (conj) {
    input = input.conj();
  }
  const auto neg = input.is_neg();
  if (neg) {
    input = input._neg_view();
  }

  if (!dim) {
    input = input.flatten();
    dim = 0;
  }

  Tensor repeats_ = repeats;
  if (repeats.dim() == 0 || (repeats.dim() == 1 && TORCH_GUARD_OR_FALSE(repeats.sym_size(0).sym_eq(1)))) {
    repeats_ = repeats.reshape({1}).expand_symint({input.sym_size(dim.value())});
  } else if (repeats.dim() == 1) {
    TORCH_CHECK(
        repeats.sym_size(0) == input.sym_size(dim.value()),
        "repeats must have the same size as input along dim, but got repeats.size(0) = ",
        repeats.sym_size(0), " and input.size(", dim.value(), ") = ", input.sym_size(dim.value())
    );
  } else {
    TORCH_CHECK(false, "repeats must be 0-dim or 1-dim tensor");
  }

  auto ret = input.index_select(
      dim.value(), at::repeat_interleave_symint(repeats_, std::move(output_size)));
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are sym_size, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 sym_size，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 91-120
```cpp
  // Restore conj and neg bits
  if (conj) {
    ret = ret.conj();
  }
  if (neg) {
    ret = ret._neg_view();
  }
  return ret;
}

Tensor repeat_interleave_symint(
    const Tensor& self,
    c10::SymInt repeats,
    std::optional<int64_t> dim_opt,
    std::optional<SymInt> output_size) {
  Tensor input = dim_opt ? self : self.flatten();
  int64_t dim = c10::maybe_wrap_dim(dim_opt.value_or(0), self.dim());
  TORCH_SYM_CHECK(repeats.sym_ge(0), "Repeats must be non-negative");

  input = input.unsqueeze(dim + 1);
  auto expand_shape = input.sym_sizes().vec();
  expand_shape[dim + 1] = repeats;
  input = input.expand_symint(expand_shape);

  // This argument doesn't really make sense for the scalar overload, but exists
  // for consistency with the tensor overload
  if (output_size) {
    auto calculated_size = repeats * expand_shape[dim];
    TORCH_SYM_CHECK(
        output_size->sym_eq(calculated_size),
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 121-130
```cpp
        "repeat_interleave: Invalid output_size, expected ",
        calculated_size,
        " but got ",
        *output_size);
  }

  return input.clone(at::MemoryFormat::Contiguous).flatten(dim, dim + 1);
}

} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The segment closes with direct return paths, indicating a compact helper or forwarding routine.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段以直接返回路径收尾，说明它更像是紧凑的辅助函数或转发例程。

## Key Concepts / 关键概念

- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Template-based reuse helps share logic across scalar types, layouts, or backends.
- CN: 基于模板的复用帮助在不同标量类型、布局或后端之间共享逻辑。
- EN: Notable symbols: compute_cpu, parallel_for, repeat_interleave_cpu, AT_DISPATCH_INDEX_TYPES, repeat_interleave_symint, TORCH_CHECK.
- CN: 重要符号：compute_cpu, parallel_for, repeat_interleave_cpu, AT_DISPATCH_INDEX_TYPES, repeat_interleave_symint, TORCH_CHECK。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/core/Tensor.h, ATen/Dispatch.h, ATen/Parallel.h, ATen/native/Repeat.h, c10/util/irange.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/empty.h, ATen/ops/repeat_interleave.h, ATen/ops/repeat_interleave_native.h`.
- CN: 主要内部头文件：`ATen/core/Tensor.h, ATen/Dispatch.h, ATen/Parallel.h, ATen/native/Repeat.h, c10/util/irange.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/empty.h, ATen/ops/repeat_interleave.h, ATen/ops/repeat_interleave_native.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `compute_cpu, parallel_for, repeat_interleave_cpu, AT_DISPATCH_INDEX_TYPES, repeat_interleave_symint, TORCH_CHECK`.
- CN: 实现围绕 `compute_cpu, parallel_for, repeat_interleave_cpu, AT_DISPATCH_INDEX_TYPES, repeat_interleave_symint, TORCH_CHECK` 等符号展开。
