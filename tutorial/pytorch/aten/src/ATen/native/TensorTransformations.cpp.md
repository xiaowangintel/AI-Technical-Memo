# TensorTransformations.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/TensorTransformations.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on tensor transformations; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 tensor transformations；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS
#include <ATen/native/IndexKernel.h> // for flip_stub
#include <ATen/native/TensorTransformations.h>

#include <ATen/Parallel.h>
#include <ATen/TensorIterator.h>
#include <ATen/WrapDimUtilsMulti.h>
#include <ATen/core/DimVector.h>
#include <c10/util/Exception.h>
#include <c10/util/irange.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/atleast_1d_native.h>
#include <ATen/ops/atleast_2d_native.h>
#include <ATen/ops/atleast_3d_native.h>
#include <ATen/ops/cat.h>
#include <ATen/ops/chalf_native.h>
#include <ATen/ops/empty_like.h>
#include <ATen/ops/flip_native.h>
#include <ATen/ops/fliplr_native.h>
#include <ATen/ops/flipud_native.h>
#include <ATen/ops/roll_native.h>
#include <ATen/ops/rot90_native.h>
#include <ATen/ops/zeros_like_ops.h>
#endif

#include <algorithm>
```
- EN: Lines 1-30 pull in 23 header dependencies, defining the compilation surface for this segment.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 第 1-30 行引入了 23 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 31-60
```cpp
#include <utility>
#include <vector>

namespace at::native {

Tensor flip(const Tensor& self, IntArrayRef dims) {
  TORCH_CHECK(
      self.scalar_type() != at::kQUInt4x2 &&
          self.scalar_type() != at::kQUInt2x4,
      "flip is not supported for tensor with data type ",
      self.scalar_type());
  const int64_t total_dims = self.dim();
  // It wraps the dims and checks that there are no repeated dims
  auto flip_dims_b = at::dim_list_to_bitset(dims, total_dims);

  Tensor out_tensor = at::empty_like(self, MemoryFormat::Preserve);

  // Count dimensions in which we need to do work
  int n = 0;
  auto strides = DimVector(self.strides());
  for (const auto i : c10::irange(total_dims)) {
    if (flip_dims_b[i] && self.size(i) > 1 && self.stride(i) != 0) {
      n++;
      strides[i] = 0;
    }
  }

  // Nothing to do, we return fast
  if (n == 0 || self.numel() <= 1) {
    out_tensor.copy_(self);
```
- EN: Lines 31-60 pull in 2 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are flip, concentrating a specific part of the operator behavior.
- CN: 第 31-60 行引入了 2 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 flip，它们承载了某一部分算子行为的核心逻辑。

### Lines 61-90
```cpp
    return out_tensor;
  }

  // create dummy output with 0 strides at flipped dimension, to prevent
  // tensorIterator from coalescing flipped dims
  const auto restrided_self = self.as_strided(self.sizes(), strides);
  auto iter =
      TensorIteratorConfig()
          .set_check_mem_overlap(false)
          .check_all_same_dtype(false)
          .declare_static_dtype_and_device(self.scalar_type(), self.device())
          .add_output(out_tensor)
          .add_const_input(self)
          .add_const_input(restrided_self)
          .build();

  auto* data = reinterpret_cast<char*>(iter.data_ptr(0));
  const auto sizes = iter.shape();
  // This is a SmallVector of _signed_ ints
  auto strides_bytes = DimVector(iter.strides(0));
  const auto strides_self = iter.strides(1);
  const auto strides_dummy = iter.strides(2);

  // To understand this transformation, think of a 3D cube.
  //   - The data ptr points to the lower-left most vertex of the cube
  //   - The strides tell us how to move in each dimension,
  //     that is, data + stride[i] advances one element in the dimension i
  // To flip a dimension:
  //   - We move the pointer to the opposite vertex of the cube
  //   - We iterate in the opposite direction (invert the strides)
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are TensorIteratorConfig, set_check_mem_overlap, check_all_same_dtype, declare_static_dtype_and_device, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 TensorIteratorConfig, set_check_mem_overlap, check_all_same_dtype, declare_static_dtype_and_device，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 91-120
```cpp

  for (const auto i : c10::irange(iter.ndim())) {
    // We know that an dimension has a zero stride and self[i] does not, as we
    // defined above Note that it may be the case that strides_dummy[i] = 0 not
    // because we set it, but because strides_self[i] == 0. We do not want to do
    // anything there
    if (strides_dummy[i] == 0 && strides_self[i] != 0) {
      data += strides_bytes[i] * (sizes[i] - 1);
      strides_bytes[i] *= -1;
    }
  }
  iter._unsafe_set_arg_strides(0, strides_bytes);
  iter._unsafe_set_arg_data(0, reinterpret_cast<void*>(data));

  flip_stub(iter.device_type(), iter, self.is_quantized());

  return out_tensor;
}

Tensor roll(
    const Tensor& self,
    IntArrayRef shifts,
    IntArrayRef dims) { // Used by CPU and MPS dispatch.
  if (dims.size() != 1 || shifts.size() != 1) {
    return roll_common(self, shifts, dims);
  }
  // avoid a div zero error below.
  if (self.numel() == 0) {
    return self.clone(at::MemoryFormat::Preserve);
  }
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 121-150
```cpp
  int64_t dim = dims[0];
  int64_t size = self.size(dim);
  int64_t start = (size - shifts[0]) % size;
  // Behavior of % is different in C++ vs Python for negative numbers. This
  // corrects the difference.
  if (start < 0) {
    start = start + size;
  }
  auto t0 = self.narrow(dim, start, size - start);
  auto t1 = self.narrow(dim, 0, start);
  return at::cat({std::move(t0), std::move(t1)}, dim);
}

Tensor rot90(const Tensor& self, int64_t k, IntArrayRef dims) {
  const int64_t total_dims = self.dim(), total_rot_dims = dims.size();

  TORCH_CHECK(
      total_rot_dims == 2,
      "expected total rotation dims == 2, but got dims = ",
      total_rot_dims);

  TORCH_CHECK(
      total_dims >= 2,
      "expected total dims >= 2, but got total dims = ",
      total_dims);

  TORCH_CHECK(
      dims[0] != dims[1] && std::abs(dims[0] - dims[1]) != total_dims,
      "expected rotation dims to be different, but got dim0 = ",
      dims[0],
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are rot90, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 rot90，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 151-180
```cpp
      " and dim1 = ",
      dims[1]);

  // check range of dims
  TORCH_CHECK(
      dims[0] < total_dims && dims[0] >= -total_dims,
      "Rotation dim0 out of range, dim0 = ",
      dims[0]);

  TORCH_CHECK(
      dims[1] < total_dims && dims[1] >= -total_dims,
      "Rotation dim1 out of range, dim1 = ",
      dims[1]);

  // handle modulo with negative k
  k = (4 + (k % 4)) % 4;

  switch (k) {
    case 1:
      return self.flip({dims[1]}).transpose_(dims[0], dims[1]);
    case 2:
      return self.flip(dims);
    case 3:
      return self.flip({dims[0]}).transpose_(dims[0], dims[1]);
    default:
      return self.clone(at::MemoryFormat::Contiguous);
  }
}

Tensor fliplr(const Tensor& self) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are fliplr, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 fliplr，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 181-210
```cpp
  TORCH_CHECK(self.dim() >= 2, "Input must be >= 2-d.");

  return self.flip({1});
}

Tensor flipud(const Tensor& self) {
  TORCH_CHECK(self.dim() >= 1, "Input must be >= 1-d.");

  return self.flip({0});
}

Tensor atleast_1d(const Tensor& self) {
  switch (self.dim()) {
    case 0:
      return self.reshape({1});
    default:
      return self;
  }
}

std::vector<Tensor> atleast_1d(TensorList tensors) {
  std::vector<Tensor> result(tensors.size());
  auto transform_lambda = [](const Tensor& input) -> Tensor {
    return at::native::atleast_1d(input);
  };
  std::transform(
      tensors.cbegin(), tensors.cend(), result.begin(), transform_lambda);
  return result;
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are flipud, atleast_1d, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 flipud, atleast_1d，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 211-240
```cpp
Tensor atleast_2d(const Tensor& self) {
  switch (self.dim()) {
    case 0:
      return self.reshape({1, 1});
    case 1: {
      return self.unsqueeze(0);
    }
    default:
      return self;
  }
}

std::vector<Tensor> atleast_2d(TensorList tensors) {
  std::vector<Tensor> result(tensors.size());
  auto transform_lambda = [](const Tensor& input) -> Tensor {
    return at::native::atleast_2d(input);
  };
  std::transform(
      tensors.cbegin(), tensors.cend(), result.begin(), transform_lambda);
  return result;
}

Tensor atleast_3d(const Tensor& self) {
  switch (self.dim()) {
    case 0:
      return self.reshape({1, 1, 1});
    case 1: {
      return self.unsqueeze(0).unsqueeze(-1);
    }
    case 2: {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are atleast_2d, atleast_3d, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 atleast_2d, atleast_3d，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 241-264
```cpp
      return self.unsqueeze(-1);
    }
    default:
      return self;
  }
}

std::vector<Tensor> atleast_3d(TensorList tensors) {
  std::vector<Tensor> result(tensors.size());
  auto transform_lambda = [](const Tensor& input) -> Tensor {
    return at::native::atleast_3d(input);
  };
  std::transform(
      tensors.cbegin(), tensors.cend(), result.begin(), transform_lambda);
  return result;
}

Tensor chalf(const Tensor& self, std::optional<MemoryFormat> memory_format) {
  return self.to(kComplexHalf, false, false, memory_format);
}

DEFINE_DISPATCH(flip_stub);

} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are atleast_3d, chalf, concentrating a specific part of the operator behavior.
- EN: Registration or dispatch macros appear here, wiring the implementation into PyTorch operator lookup and backend dispatch.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 atleast_3d, chalf，它们承载了某一部分算子行为的核心逻辑。
- CN: 这里出现了注册或分发宏，用于把实现接入 PyTorch 的算子查找与后端分发机制。

## Key Concepts / 关键概念

- EN: Registration/dispatch logic is part of the file structure.
- CN: 文件结构中包含注册/分发逻辑。
- EN: TensorIterator is used to describe elementwise or reduction-style iteration.
- CN: 使用 TensorIterator 描述逐元素或归约式遍历。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: flip, TensorIteratorConfig, roll, rot90, fliplr, flipud, atleast_1d, atleast_2d.
- CN: 重要符号：flip, TensorIteratorConfig, roll, rot90, fliplr, flipud, atleast_1d, atleast_2d。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/native/TensorTransformations.h, ATen/Parallel.h, ATen/TensorIterator.h, ATen/WrapDimUtilsMulti.h, ATen/core/DimVector.h, c10/util/Exception.h, c10/util/irange.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/atleast_1d_native.h, ATen/ops/atleast_2d_native.h, ATen/ops/atleast_3d_native.h`.
- CN: 主要内部头文件：`ATen/native/TensorTransformations.h, ATen/Parallel.h, ATen/TensorIterator.h, ATen/WrapDimUtilsMulti.h, ATen/core/DimVector.h, c10/util/Exception.h, c10/util/irange.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/atleast_1d_native.h, ATen/ops/atleast_2d_native.h, ATen/ops/atleast_3d_native.h`。
- EN: External/system headers: `#include <ATen/native/IndexKernel.h> // for flip_stub, algorithm, utility, vector`.
- CN: 外部/系统头文件：`#include <ATen/native/IndexKernel.h> // for flip_stub, algorithm, utility, vector`。
- EN: The implementation revolves around symbols such as `flip, TensorIteratorConfig, roll, rot90, fliplr, flipud, atleast_1d, atleast_2d, atleast_3d, chalf`.
- CN: 实现围绕 `flip, TensorIteratorConfig, roll, rot90, fliplr, flipud, atleast_1d, atleast_2d, atleast_3d, chalf` 等符号展开。
