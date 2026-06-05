# TensorProperties.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/TensorProperties.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on tensor properties; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 tensor properties；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#define TORCH_ASSERT_ONLY_METHOD_OPERATORS
#include <ATen/Context.h>
#include <ATen/NamedTensorUtils.h>
#include <ATen/core/Tensor.h>
#include <ATen/detail/CUDAHooksInterface.h>
#include <ATen/native/TensorProperties.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#else
#include <ATen/ops/_nested_tensor_size_native.h>
#include <ATen/ops/contiguous_native.h>
#include <ATen/ops/cudnn_is_acceptable_native.h>
#include <ATen/ops/detach_native.h>
#include <ATen/ops/dim_native.h>
#include <ATen/ops/equal.h>
#include <ATen/ops/get_device_native.h>
#include <ATen/ops/is_contiguous_native.h>
#include <ATen/ops/is_same_size_native.h>
#include <ATen/ops/is_set_to_native.h>
#include <ATen/ops/numel_native.h>
#include <ATen/ops/size_native.h>
#include <ATen/ops/storage_offset_native.h>
#include <ATen/ops/stride_native.h>
#include <ATen/ops/sym_is_contiguous_native.h>
#include <ATen/ops/sym_numel_native.h>
#include <ATen/ops/sym_size_native.h>
#include <ATen/ops/sym_storage_offset_native.h>
#include <ATen/ops/sym_stride_native.h>
```
- EN: Lines 1-30 pull in 26 header dependencies, defining the compilation surface for this segment.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 第 1-30 行引入了 26 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 31-60
```cpp
#endif

#include <c10/util/irange.h>

namespace at::native {

bool is_same_size(const Tensor& self, const Tensor& other) {
  return self.sym_sizes().equals(other.sym_sizes());
}

bool nested_is_same_size(const Tensor& self, const Tensor& other) {
  TORCH_CHECK(
      self.is_nested() && other.is_nested(),
      "Expected both self and other to be nested tensors. ",
      "Self ",
      self.is_nested() ? "is " : "is not ",
      "nested. While Other ",
      other.is_nested() ? "is " : "is not ",
      "nested.")
  const auto self_nt_size = _nested_tensor_size(self);
  const auto other_nt_size = _nested_tensor_size(other);
  return at::equal(self_nt_size, other_nt_size);
}
int64_t size(const Tensor& self, int64_t dim) {
  return self.size(dim);
}

int64_t stride(const Tensor& self, int64_t dim) {
  return self.stride(dim);
}
```
- EN: Lines 31-60 pull in 1 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are is_same_size, nested_is_same_size, size, stride, concentrating a specific part of the operator behavior.
- CN: 第 31-60 行引入了 1 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 is_same_size, nested_is_same_size, size, stride，它们承载了某一部分算子行为的核心逻辑。

### Lines 61-90
```cpp

c10::SymInt sym_size(const Tensor& self, int64_t dim) {
  return self.sym_size(dim);
}

c10::SymBool sym_is_contiguous(
    const Tensor& self,
    c10::MemoryFormat memory_format) {
  return self.sym_is_contiguous(memory_format);
}

c10::SymInt sym_stride(const Tensor& self, int64_t dim) {
  return self.sym_stride(dim);
}

c10::SymInt sym_numel(const Tensor& self) {
  return self.sym_numel();
}

c10::SymInt sym_storage_offset(const Tensor& self) {
  return self.sym_storage_offset();
}

int64_t numel(const Tensor& self) {
  return self.numel();
}

int64_t dim(const Tensor& self) {
  return self.dim();
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are sym_size, sym_stride, sym_numel, sym_storage_offset, concentrating a specific part of the operator behavior.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 sym_size, sym_stride, sym_numel, sym_storage_offset，它们承载了某一部分算子行为的核心逻辑。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 91-120
```cpp

int64_t get_device(const Tensor& self) {
  return self.get_device();
}

int64_t storage_offset(const Tensor& self) {
  return self.storage_offset();
}

bool is_contiguous(const Tensor& self) {
  return self.is_contiguous();
}

bool is_contiguous(const Tensor& self, at::MemoryFormat memory_format) {
  return self.is_contiguous(memory_format);
}

int64_t size(const Tensor& self, Dimname dim) {
  size_t pos_dim = dimname_to_position(self, dim);
  return self.sizes()[pos_dim];
}

int64_t stride(const Tensor& self, Dimname dim) {
  size_t pos_dim = dimname_to_position(self, dim);
  return self.strides()[pos_dim];
}

bool cudnn_is_acceptable(const TensorBase& self) {
  if (!globalContext().userEnabledCuDNN())
    return false;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are get_device, storage_offset, is_contiguous, size, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 get_device, storage_offset, is_contiguous, size，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 121-150
```cpp
  if (!self.is_cuda())
    return false;
  if (!detail::getCUDAHooks().compiledWithCuDNN())
    return false;
  // cuDNN functions like grid_sampler returns CUDNN_STATUS_BAD_PARAM on empty
  // tensors. Maybe some cuDNN functions actually support empty tensors, but
  // native/THNN kernels shouldn't be much slower because the output is also
  // likely empty.
  if (self.sym_numel() == 0)
    return false;
  // NB: In the old Python code, there was also a test to see if the
  // cuDNN library was actually dynamically linked or not.  I'm not
  // sure if we can actually test this.
  return true;
}

bool cudnn_is_acceptable(const Tensor& self) {
  return cudnn_is_acceptable(static_cast<const TensorBase&>(self));
}

Tensor& detach_(Tensor& self) {
  // this just exists to give us a hook in VariableType and an entry in
  // Declarations.yaml
  // TORCH_CHECK(false, "detach_ is not implemented for Tensor");
  return self;
}

Tensor contiguous(const Tensor& self, MemoryFormat memory_format) {
  if (self.is_contiguous_or_false(memory_format)) {
    return self;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are cudnn_is_acceptable, detach_, contiguous, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 cudnn_is_acceptable, detach_, contiguous，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 151-174
```cpp
  }
  TORCH_CHECK(
      memory_format != MemoryFormat::Preserve,
      "preserve memory format is unsupported by the contiguous operator");

  return self.clone(memory_format);
}

bool is_set_to(const Tensor& self, const Tensor& src) {
  if (self.storage().unsafeGetStorageImpl() ==
          src.storage().unsafeGetStorageImpl() &&
      self.storage_offset() == src.storage_offset() &&
      self.dim() == src.dim()) {
    for (const auto d : c10::irange(self.dim())) {
      if (self.size(d) != src.size(d) || self.stride(d) != src.stride(d)) {
        return false;
      }
    }
    return true;
  }
  return false;
}

} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are is_set_to, dim, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 is_set_to, dim，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

## Key Concepts / 关键概念

- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: is_same_size, nested_is_same_size, TORCH_CHECK, size, stride, sym_size, sym_is_contiguous, sym_stride.
- CN: 重要符号：is_same_size, nested_is_same_size, TORCH_CHECK, size, stride, sym_size, sym_is_contiguous, sym_stride。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/Context.h, ATen/NamedTensorUtils.h, ATen/core/Tensor.h, ATen/detail/CUDAHooksInterface.h, ATen/native/TensorProperties.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/_nested_tensor_size_native.h, ATen/ops/contiguous_native.h, ATen/ops/cudnn_is_acceptable_native.h, ATen/ops/detach_native.h, ATen/ops/dim_native.h`.
- CN: 主要内部头文件：`ATen/Context.h, ATen/NamedTensorUtils.h, ATen/core/Tensor.h, ATen/detail/CUDAHooksInterface.h, ATen/native/TensorProperties.h, ATen/Functions.h, ATen/NativeFunctions.h, ATen/ops/_nested_tensor_size_native.h, ATen/ops/contiguous_native.h, ATen/ops/cudnn_is_acceptable_native.h, ATen/ops/detach_native.h, ATen/ops/dim_native.h`。
- EN: External/system headers are minimal in this file.
- CN: 该文件中的外部/系统头文件依赖较少。
- EN: The implementation revolves around symbols such as `is_same_size, nested_is_same_size, TORCH_CHECK, size, stride, sym_size, sym_is_contiguous, sym_stride, sym_numel, sym_storage_offset`.
- CN: 实现围绕 `is_same_size, nested_is_same_size, TORCH_CHECK, size, stride, sym_size, sym_is_contiguous, sym_stride, sym_numel, sym_storage_offset` 等符号展开。
