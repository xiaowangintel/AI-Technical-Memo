# RangeFactories.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/RangeFactories.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on range factories; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 range factories；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS
#include <ATen/native/RangeFactories.h>
#include <ATen/native/RangeUtils.h>
#include <ATen/AccumulateType.h>
#include <ATen/Dispatch.h>
#include <ATen/Parallel.h>
#include <ATen/TensorIterator.h>
#include <c10/util/irange.h>
#include <cmath>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/linspace.h>
#include <ATen/ops/logspace.h>
#include <ATen/ops/arange_native.h>
#include <ATen/ops/linspace_native.h>
#include <ATen/ops/logspace_native.h>
#include <ATen/ops/range_native.h>
#endif

namespace at::native {

Tensor& linspace_out(const Tensor& start, const Tensor& end, int64_t steps, Tensor& result) {
  TORCH_CHECK(start.dim() == 0 && end.dim() == 0, "linspace only supports 0-dimensional start and end tensors, "
    "but got start with ", start.dim(), " dimension(s) and end with ", end.dim()," dimension(s).");
  return at::linspace_out(result, start.item(), end.item(), steps);
}
```
- EN: Lines 1-30 pull in 16 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are linspace_out, concentrating a specific part of the operator behavior.
- CN: 第 1-30 行引入了 16 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 linspace_out，它们承载了某一部分算子行为的核心逻辑。

### Lines 31-60
```cpp
Tensor& linspace_out(const Tensor& start, const Scalar& end, int64_t steps, Tensor& result) {
  TORCH_CHECK(start.dim() == 0, "linspace only supports 0-dimensional start and end tensors, "
    "but got start with ", start.dim(), " dimension(s).");
  return at::linspace_out(result, start.item(), end, steps);
}

Tensor& linspace_out(const Scalar& start, const Tensor& end, int64_t steps, Tensor& result) {
  TORCH_CHECK(end.dim() == 0, "linspace only supports 0-dimensional start and end tensors, "
    "but got end with ", end.dim()," dimension(s).");
  return at::linspace_out(result, start, end.item(), steps);
}

Tensor& linspace_out(const Scalar& start, const Scalar& end, int64_t steps, Tensor& result) {
  TORCH_CHECK(steps >= 0, "number of steps must be non-negative");
  if (result.numel() != steps) {
    result.resize_({steps});
  }

  if (result.device() == kMeta) {
    return result;
  }

  if (steps == 0) {
    // skip
  } else if (steps == 1) {
    result.fill_(start);
  } else {
    Tensor r = result.is_contiguous() ? result : result.contiguous();
    auto iter = TensorIterator::borrowing_nullary_op(r);
    linspace_stub(iter.device_type(), iter, start, end, steps);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are linspace_out, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 linspace_out，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 61-90
```cpp
    if (!result.is_contiguous()) {
      result.copy_(r);
    }
  }

  return result;
}

Tensor& logspace_out(const Tensor& start, const Tensor& end, int64_t steps, double base, Tensor& result) {
  TORCH_CHECK(start.dim() == 0 && end.dim() == 0, "logspace only supports 0-dimensional start and end tensors, "
    "but got start with ", start.dim(), " dimension(s) and end with ", end.dim()," dimension(s).");
  return at::logspace_out(result, start.item(), end.item(), steps, base);
}

Tensor& logspace_out(const Tensor& start, const Scalar& end, int64_t steps, double base, Tensor& result) {
  TORCH_CHECK(start.dim() == 0, "logspace only supports 0-dimensional start and end tensors, "
    "but got start with ", start.dim(), " dimension(s).");
  return at::logspace_out(result, start.item(), end, steps, base);
}

Tensor& logspace_out(const Scalar& start, const Tensor& end, int64_t steps, double base, Tensor& result) {
  TORCH_CHECK(end.dim() == 0, "logspace only supports 0-dimensional start and end tensors, "
    "but got end with ", end.dim()," dimension(s).");
  return at::logspace_out(result, start, end.item(), steps, base);
}

Tensor& logspace_out(const Scalar& start, const Scalar& end, int64_t steps, double base, Tensor& result) {
  TORCH_CHECK(steps >= 0, "number of steps must be non-negative");

  if (result.numel() != steps) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are logspace_out, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 logspace_out，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 91-120
```cpp
    result.resize_({steps});
  }

  if (result.device() == kMeta) {
    return result;
  }

  Tensor r = result.is_contiguous() ? result : result.contiguous();

  if (steps == 0) {
    // skip
  } else if (steps == 1) {
    if (isComplexType(r.scalar_type())){
      r.fill_(std::pow(base, start.to<c10::complex<double>>()));
    } else {
      r.fill_(std::pow(base, start.to<double>()));
    }
  } else if (isComplexType(r.scalar_type())) {
    AT_DISPATCH_COMPLEX_TYPES(r.scalar_type(), "logspace_cpu", [&]() {
      scalar_t scalar_base = static_cast<scalar_t>(base);
      scalar_t scalar_start = start.to<scalar_t>();
      scalar_t scalar_end = end.to<scalar_t>();
      scalar_t *data_ptr = r.data_ptr<scalar_t>();
      scalar_t step = (scalar_end - scalar_start) / static_cast<scalar_t>(steps - 1);
      const int64_t halfway = steps / 2;
      at::parallel_for(0, steps, internal::GRAIN_SIZE, [&](int64_t p_begin, int64_t p_end) {
        scalar_t is = static_cast<scalar_t>(p_begin);
        for (int64_t i = p_begin; i < p_end; ++i, is+=1) { //std::complex does not support ++operator
          if (i < halfway) {
            data_ptr[i] = std::pow(scalar_base, scalar_start + step*is);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are AT_DISPATCH_COMPLEX_TYPES, parallel_for, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 AT_DISPATCH_COMPLEX_TYPES, parallel_for，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 121-150
```cpp
          } else {
            data_ptr[i] = std::pow(scalar_base, scalar_end - (step * static_cast<scalar_t>(steps - i - 1)));
          }
        }
      });
    });
  } else {
    AT_DISPATCH_ALL_TYPES_AND2(kBFloat16, kHalf, r.scalar_type(), "logspace_cpu", [&]() {
      double scalar_base = static_cast<double>(base); // will be autopromoted anyway
      scalar_t scalar_start = start.to<scalar_t>();
      scalar_t scalar_end = end.to<scalar_t>();
      scalar_t *data_ptr = r.data_ptr<scalar_t>();
      double step = static_cast<double>(scalar_end - scalar_start) / (steps - 1);
      const int64_t halfway = steps / 2;
      at::parallel_for(0, steps, internal::GRAIN_SIZE, [&](int64_t p_begin, int64_t p_end) {
        for (const auto i : c10::irange(p_begin, p_end)) {
          if (i < halfway) {
            data_ptr[i] = std::pow(scalar_base, scalar_start + step*i);
          } else {
            data_ptr[i] = std::pow(scalar_base, scalar_end - step * (steps - i - 1));
          }
        }
      });
    });
  }

  if (!result.is_contiguous()) {
    result.copy_(r);
  }
  return result;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are AT_DISPATCH_ALL_TYPES_AND2, parallel_for, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 AT_DISPATCH_ALL_TYPES_AND2, parallel_for，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 151-180
```cpp
}

Tensor& range_out(const Scalar& start, const Scalar& end, const Scalar& step, Tensor& result) {
  AT_DISPATCH_ALL_TYPES_AND2(kBFloat16, kHalf, result.scalar_type(), "range_cpu", [&]() {
    using accscalar_t = at::acc_type<scalar_t, false>;
    auto xstart = start.to<accscalar_t>();
    auto xend = end.to<accscalar_t>();
    auto xstep = step.to<accscalar_t>();

    arange_check_bounds(start, end, step);

    int64_t size = static_cast<int64_t>(((xend - xstart) / xstep) + 1);
    if (result.numel() != size) {
      result.resize_({size});
    }

    if (result.device() == kMeta) {
      return;
    }

    Tensor r = result.is_contiguous() ? result : result.contiguous();
    scalar_t *data_ptr = r.data_ptr<scalar_t>();

    at::parallel_for(0, size, internal::GRAIN_SIZE, [&](int64_t p_begin, int64_t p_end) {
      accscalar_t is = p_begin;
      for (int64_t i = p_begin; i < p_end; ++i, ++is) {
        data_ptr[i] = xstart + is * xstep;
      }
    });
    if (!result.is_contiguous()) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are range_out, AT_DISPATCH_ALL_TYPES_AND2, parallel_for, concentrating a specific part of the operator behavior.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 range_out, AT_DISPATCH_ALL_TYPES_AND2, parallel_for，它们承载了某一部分算子行为的核心逻辑。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 181-210
```cpp
      result.copy_(r);
    }
  });

  return result;
}

Tensor& range_out_no_step(const Scalar& start, const Scalar& end, Tensor& result) {
  return range_out(start, end, /*step = */ 1, result);
}

Tensor& arange_out(const Scalar& start, const Scalar& end, const Scalar& step, Tensor& result) {
  AT_DISPATCH_ALL_TYPES_AND2(kHalf, kBFloat16, result.scalar_type(), "arange_cpu", [&]() {
    int64_t size = compute_arange_size<scalar_t>(start, end, step);
    int64_t numel = result.numel();

    if (numel != size) {
      if(numel > 0){
        TORCH_WARN("The number of elements in the out tensor of shape ", result.sizes(),
                    " is ", numel, " which does not match the computed number of elements ", size,
                    ". Note that this may occur as a result of rounding error. "
                    "The out tensor will be resized to a tensor of shape (", size, ",).");
      }
      result.resize_({size});
    }

    if (result.device() == kMeta) {
      return;
    }
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are range_out_no_step, arange_out, AT_DISPATCH_ALL_TYPES_AND2, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 range_out_no_step, arange_out, AT_DISPATCH_ALL_TYPES_AND2，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 211-225
```cpp
    Tensor r = result.is_contiguous() ? result : result.contiguous();
    auto iter = TensorIterator::borrowing_nullary_op(r);
    arange_stub(iter.device_type(), iter, start, size, step);
    if (!result.is_contiguous()) {
      result.copy_(r);
    }
  });

  return result;
}

DEFINE_DISPATCH(arange_stub);
DEFINE_DISPATCH(linspace_stub);

} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Registration or dispatch macros appear here, wiring the implementation into PyTorch operator lookup and backend dispatch.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这里出现了注册或分发宏，用于把实现接入 PyTorch 的算子查找与后端分发机制。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

## Key Concepts / 关键概念

- EN: Registration/dispatch logic is part of the file structure.
- CN: 文件结构中包含注册/分发逻辑。
- EN: TensorIterator is used to describe elementwise or reduction-style iteration.
- CN: 使用 TensorIterator 描述逐元素或归约式遍历。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: linspace_out, logspace_out, AT_DISPATCH_COMPLEX_TYPES, parallel_for, AT_DISPATCH_ALL_TYPES_AND2, range_out, range_out_no_step, arange_out.
- CN: 重要符号：linspace_out, logspace_out, AT_DISPATCH_COMPLEX_TYPES, parallel_for, AT_DISPATCH_ALL_TYPES_AND2, range_out, range_out_no_step, arange_out。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/native/RangeFactories.h, ATen/native/RangeUtils.h, ATen/AccumulateType.h, ATen/Dispatch.h, ATen/Parallel.h, ATen/TensorIterator.h, c10/util/irange.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/linspace.h, ATen/ops/logspace.h, ATen/ops/arange_native.h`.
- CN: 主要内部头文件：`ATen/native/RangeFactories.h, ATen/native/RangeUtils.h, ATen/AccumulateType.h, ATen/Dispatch.h, ATen/Parallel.h, ATen/TensorIterator.h, c10/util/irange.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/linspace.h, ATen/ops/logspace.h, ATen/ops/arange_native.h`。
- EN: External/system headers: `cmath`.
- CN: 外部/系统头文件：`cmath`。
- EN: The implementation revolves around symbols such as `linspace_out, logspace_out, AT_DISPATCH_COMPLEX_TYPES, parallel_for, AT_DISPATCH_ALL_TYPES_AND2, range_out, range_out_no_step, arange_out`.
- CN: 实现围绕 `linspace_out, logspace_out, AT_DISPATCH_COMPLEX_TYPES, parallel_for, AT_DISPATCH_ALL_TYPES_AND2, range_out, range_out_no_step, arange_out` 等符号展开。
