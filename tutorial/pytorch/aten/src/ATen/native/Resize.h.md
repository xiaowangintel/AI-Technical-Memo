# Resize.h — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/Resize.h`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the ATen native layer in PyTorch ATen native code and focuses on resize; it mainly declares interfaces, helper types, and inline utilities.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的ATen 原生层，主题聚焦于 resize；其主要作用是声明接口、辅助类型以及内联工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
#pragma once

#include <ATen/core/Tensor.h>
#include <ATen/native/ResizeCommon.h>
#include <ATen/EmptyTensor.h>
#include <ATen/TensorUtils.h>

#include <c10/core/CPUAllocator.h>
#include <c10/core/SymBool.h>

#include <utility>


namespace at::native {

// TODO: make all operations that resize given outputs use this function
//   for consistency and maintainability.
//   Some operations like `cat` might not be able to make the use of
//   resize_output directly. For more details to understand how it works in `cat`,
//   see https://github.com/pytorch/pytorch/pull/62560#discussion_r687363362
// Resizes outputs
// Functions accepting output tensors, like with the "out" kwarg, should
//   call this function to handle resizing their output tensor.
// Issues a warning if the output tensor has one or more elements and
//   needs resizing
// NOTE: In the future the warning will become an error
// Returns a bool saying whether or not the resize actually happened or not
TORCH_API bool resize_output(const Tensor& output, IntArrayRef shape);
// WARNING: Do NOT call this directly. If you are resizing an output and want
// to support dynamic shapes call at::resize__symint and resize_output_check_symint.
```
- EN: Lines 1-30 pull in 7 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 第 1-30 行引入了 7 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 31-60
```cpp
// For more details, see: https://github.com/pytorch/pytorch/pull/111530/files#r1365845272
TORCH_API bool resize_output_symint(const Tensor& output, SymIntArrayRef shape);

// Utility for resize_output
//  Returns a bool saying resize should happen or not and
//  raises a warning if resizing for one or more elements
TORCH_API bool resize_output_check(const Tensor& output, IntArrayRef shape);
TORCH_API bool resize_output_check_symint(const Tensor& output, SymIntArrayRef shape);

TORCH_API void resize_bytes_cpu(StorageImpl* storage, size_t size_bytes);
TORCH_API void resize_bytes_meta(StorageImpl* storage, c10::SymInt size_bytes);
TORCH_API void resize_bytes_nocuda(const Storage& storage, const c10::SymInt& size_bytes);

inline void maybe_resize_storage_cpu(TensorImpl* self, size_t new_size_bytes) {
  // It does not make sense to try to resize a storage
  // to hold 0 elements, and this can break
  // if storage_offset is positive but
  // new_size is 0, so just bail in that case
  // (same comment is in cuda/Resize.h)
  if (self->numel() == 0) {
    return;
  }

  const Storage& storage = self->unsafe_storage();
  if (!storage) {
    auto new_storage = c10::make_intrusive<StorageImpl>(
        StorageImpl::use_byte_size_t(),
        new_size_bytes,
        c10::GetCPUAllocator(),
        true);
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are maybe_resize_storage_cpu, concentrating a specific part of the operator behavior.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 maybe_resize_storage_cpu，它们承载了某一部分算子行为的核心逻辑。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 61-90
```cpp
    self->set_storage_keep_dtype(std::move(new_storage));
  } else if (new_size_bytes > storage.nbytes()) {
    resize_bytes_cpu(storage.unsafeGetStorageImpl(), new_size_bytes);
  }
}

TORCH_API TensorImpl* resize_impl_cpu_(
    TensorImpl* self,
    IntArrayRef size,
    at::OptionalIntArrayRef stride,
    bool resize_storage = true);

template <typename T>
T maybe_convert_symint(c10::SymInt) = delete;

template <>
inline c10::SymInt maybe_convert_symint(c10::SymInt x) { return x; }

template <>
inline int64_t maybe_convert_symint(c10::SymInt x) { return x.guard_int(__FILE__, __LINE__); }

template <typename T>
inline void checkInBoundsForStorage(
    ArrayRef<T> size,
    ArrayRef<T> stride,
    T storage_offset,
    const caffe2::TypeMeta& data_type,
    const Storage& new_storage) {
  T storage_size_bytes, storage_size_plus_offset_bytes;
  if (stride.data()) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 91-120
```cpp
    storage_size_bytes =
        at::detail::computeStorageNbytes(size, stride, data_type.itemsize());
    storage_size_plus_offset_bytes = at::detail::computeStorageNbytes(
        size, stride, data_type.itemsize(), storage_offset);
  } else {
    storage_size_bytes =
        at::detail::computeStorageNbytesContiguous(size, data_type.itemsize());
    storage_size_plus_offset_bytes = at::detail::computeStorageNbytesContiguous(
        size, data_type.itemsize(), storage_offset);
  }
  // It's ok to always evaluate to False for this early return for SymInts because
  // (1) maybe_convert_symint below only installs guard for int64_t case
  // (2) we check for this condition in the TORCH_MAYBE_SYM_CHECK below
  if (TORCH_GUARD_OR_FALSE(sym_eq(storage_size_bytes, 0))) {
    // NB: (a tensor with arbitrary 0 dims)'s storage can have any numel.
    return;
  }
  T new_storage_size_bytes = maybe_convert_symint<T>(new_storage.sym_nbytes());
  TORCH_MAYBE_SYM_CHECK(
      sym_eq(storage_size_bytes, 0) || sym_le(storage_size_plus_offset_bytes, new_storage_size_bytes),
      "setStorage: sizes ",
      size,
      ", strides ",
      stride,
      ","
      " storage offset ",
      storage_offset,
      ", and itemsize ",
      data_type.itemsize(),
      " requiring a storage size of ",
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 121-150
```cpp
      storage_size_plus_offset_bytes,
      " are out of bounds for storage of size ",
      new_storage_size_bytes);
}

template <typename T>
inline void checkSetStorage(Tensor& result, Storage storage, T storage_offset,
                                   ArrayRef<T> size, ArrayRef<T> stride, bool check_offset_in_bounds = true) {
  // FIXME: stride should be optional
  if (stride.data()) {
    TORCH_CHECK(size.size() == stride.size(), "unequal size length (", size.size(),
                                              ") and stride length (", stride.size(), ")");
  }

#ifdef DEBUG
  TORCH_CHECK(size.size() <= INT_MAX, "size length (", size.size(), ") greater than INT_MAX");
#endif

  // storageOffset
  TORCH_CHECK(
    TORCH_GUARD_OR_TRUE(sym_ge(storage_offset, 0)), "Tensor: invalid storage offset ", storage_offset);

  // set_storage_{device} (except set_storage_meta__symint)
  // will (unsafely) set the storage offset and then call resize_impl that
  // handles resizing the storage However, resize_impl will only resize the
  // storage if the sizes/strides changed. For the case that the sizes/strides
  // remain unchanged, the storage offset is not properly validated, so we do
  // that here.
  if (check_offset_in_bounds) {
    auto result_tensor_impl = result.unsafeGetTensorImpl();
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 151-180
```cpp
    bool size_unchanged = result_tensor_impl->generic_sizes<T>() == size;
    bool stride_unchanged = stride.data()
        ? result_tensor_impl->generic_strides<T>() == stride
        : true;
    if (size_unchanged && stride_unchanged) {
      checkInBoundsForStorage(
          size, stride, storage_offset, result.dtype(), storage);
    }
  }

  // storage: note this can't be replaced with result.set_(storage) as the semantics of that
  // function is to set the tensor size to be equal to the size of the storage.
  if (!result.storage().is_alias_of(storage)) {
    // Caffe2 might have tensors whose storages are null, but we
    // don't allow it in PyTorch.
    TORCH_INTERNAL_ASSERT(storage);
    TORCH_INTERNAL_ASSERT(result.storage());

    // We used to allow this, but this breaks device caching.
    // Let's put an actual error message for this one.
    TORCH_CHECK(result.storage().device() == storage.device(),
                "Attempted to set the storage of a tensor on device \"", result.storage().device(),
                "\" to a storage on different device \"", storage.device(),
                "\".  This is no longer allowed; the devices must match.");
    result.unsafeGetTensorImpl()->set_storage_keep_dtype(std::move(storage));
  }
}

/**
 * Set self's sizes, strides, and storage_offset.
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are data, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 data，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 181-210
```cpp
 * (size, stride, storage_offset) must be in bounds for self's storage.
 */
template <typename T>
void checkAsStridedArgs(
    ArrayRef<T> size,
    ArrayRef<T> stride,
    T storage_offset) {
  TORCH_CHECK(
      size.size() == stride.size(), "mismatch in length of strides and shape");
  for (const auto& val : stride) {
    TORCH_CHECK(
        val >= 0,
        "as_strided: Negative strides are not supported at the moment, "
        "got strides: ",
        stride);
  }
  TORCH_CHECK(storage_offset >= 0, "Tensor: invalid storage offset ", storage_offset);
}

template <typename T>
void checkAsStridedArgsAllowUnbackedSymInts(
    ArrayRef<T> size,
    ArrayRef<T> stride,
    T storage_offset) {
  TORCH_CHECK(
      size.size() == stride.size(), "mismatch in length of strides and shape");
  if constexpr (std::is_same_v<T, c10::SymInt>) {
    // FakeTensor/Meta view replay can pass ephemeral symbolic metadata here,
    // so only validate values once the SymInts become concrete.
    for (const auto& val : stride) {
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 211-240
```cpp
      if (auto maybe_val = val.maybe_as_int()) {
        TORCH_CHECK(
            *maybe_val >= 0,
            "as_strided: Negative strides are not supported at the moment, "
            "got strides: ",
            stride);
      }
    }

    if (auto maybe_storage_offset = storage_offset.maybe_as_int()) {
      TORCH_CHECK(
          *maybe_storage_offset >= 0,
          "Tensor: invalid storage offset ",
          storage_offset);
    }
  } else {
    for (const auto& val : stride) {
      TORCH_CHECK(
          val >= 0,
          "as_strided: Negative strides are not supported at the moment, "
          "got strides: ",
          stride);
    }

    TORCH_CHECK(
        storage_offset >= 0,
        "Tensor: invalid storage offset ",
        storage_offset);
  }
}
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Loop constructs iterate over tensor dimensions, descriptors, lookup tables, or grouped work items to build results or metadata.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 循环结构会遍历张量维度、描述符、查找表或分组工作项，以构造结果或元数据。

### Lines 241-257
```cpp

template <typename T>
inline void setStrided(
    const Tensor& self,
    ArrayRef<T> size,
    ArrayRef<T> stride,
    T storage_offset) {
  checkAsStridedArgs(size, stride, storage_offset);

  auto* self_ = self.unsafeGetTensorImpl();
  checkInBoundsForStorage(
      size, stride, storage_offset, self_->dtype(), self_->storage());

  self_->set_sizes_and_strides(size, stride, storage_offset);
}

} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

## Key Concepts / 关键概念

- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Template-based reuse helps share logic across scalar types, layouts, or backends.
- CN: 基于模板的复用帮助在不同标量类型、布局或后端之间共享逻辑。
- EN: Notable symbols: maybe_resize_storage_cpu, maybe_convert_symint, checkInBoundsForStorage, checkSetStorage, checkAsStridedArgs, checkAsStridedArgsAllowUnbackedSymInts, constexpr, setStrided.
- CN: 重要符号：maybe_resize_storage_cpu, maybe_convert_symint, checkInBoundsForStorage, checkSetStorage, checkAsStridedArgs, checkAsStridedArgsAllowUnbackedSymInts, constexpr, setStrided。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/core/Tensor.h, ATen/native/ResizeCommon.h, ATen/EmptyTensor.h, ATen/TensorUtils.h, c10/core/CPUAllocator.h, c10/core/SymBool.h`.
- CN: 主要内部头文件：`ATen/core/Tensor.h, ATen/native/ResizeCommon.h, ATen/EmptyTensor.h, ATen/TensorUtils.h, c10/core/CPUAllocator.h, c10/core/SymBool.h`。
- EN: External/system headers: `utility`.
- CN: 外部/系统头文件：`utility`。
- EN: The implementation revolves around symbols such as `maybe_resize_storage_cpu, maybe_convert_symint, checkInBoundsForStorage, checkSetStorage, checkAsStridedArgs, checkAsStridedArgsAllowUnbackedSymInts, constexpr, setStrided`.
- CN: 实现围绕 `maybe_resize_storage_cpu, maybe_convert_symint, checkInBoundsForStorage, checkSetStorage, checkAsStridedArgs, checkAsStridedArgsAllowUnbackedSymInts, constexpr, setStrided` 等符号展开。
