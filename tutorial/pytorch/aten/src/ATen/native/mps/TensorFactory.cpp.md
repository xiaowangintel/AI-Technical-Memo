# TensorFactory.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/mps/TensorFactory.cpp`
- Repository: `pytorch/pytorch` (`27a4844d7fb`)
- Purpose (EN): This file belongs to the MPS backend in PyTorch ATen native code and focuses on tensor factory; it mainly implements operator logic, runtime checks, and backend-specific execution paths.
- 目的（CN）: 该文件属于 PyTorch ATen 原生代码中的MPS 后端，主题聚焦于 tensor factory；其主要作用是实现算子逻辑、运行时检查以及后端专用执行路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
//  Copyright © 2022 Apple Inc.

#include <ATen/ATen.h>
#include <ATen/Tensor.h>
#include <ATen/Utils.h>
#include <torch/library.h>
#include <ATen/mps/EmptyTensor.h>
#include <ATen/mps/MPSDevice.h>
#include <ATen/native/Resize.h>
#include <ATen/native/ResizeCommon.h>
#include <ATen/native/mps/Copy.h>
#include <ATen/native/mps/TensorFactory.h>
#include <ATen/Dispatch.h>

#ifndef AT_PER_OPERATOR_HEADERS
#include <ATen/Functions.h>
#include <ATen/NativeFunctions.h>
#endif
#include <ATen/ops/_efficientzerotensor_native.h>

#include <utility>

namespace at::native {

static inline void maybe_resize_storage_mps(TensorImpl* self, uint64_t new_size) {
  if (new_size == 0) {
    return;
  }

  auto storage = self->storage().unsafeGetStorageImpl();
```
- EN: Lines 1-30 pull in 15 header dependencies, defining the compilation surface for this segment.
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are maybe_resize_storage_mps, concentrating a specific part of the operator behavior.
- CN: 第 1-30 行引入了 15 个头文件依赖，为该代码段确定编译边界和可见接口。
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 maybe_resize_storage_mps，它们承载了某一部分算子行为的核心逻辑。

### Lines 31-60
```cpp
  if (!storage) {
    TORCH_CHECK(false, "Tensor: invalid null storage");
  }
  uint64_t new_size_bytes = (new_size + self->storage_offset()) * self->dtype().itemsize();
  if (new_size_bytes > self->storage().nbytes()) {
    if (new_size_bytes == 0) {
      storage->set_data_ptr_noswap(at::DataPtr(nullptr, at::Device(at::DeviceType::MPS, 0)));
      storage->set_nbytes(0);
    } else {
      at::DataPtr new_data = storage->allocator()->allocate(new_size_bytes);
      size_t copy_capacity = std::min<size_t>(new_size_bytes, storage->nbytes());
      if (storage->data() && copy_capacity > 0) {
        at::native::mps::copy_blit_mps(new_data.get(), storage->data(), copy_capacity);
      }
      // Destructively overwrite data_ptr
      storage->set_data_ptr_noswap(std::move(new_data));
      storage->set_nbytes(new_size_bytes);
    }
  }
}

inline TensorImpl* resize_impl_mps_(
    TensorImpl* self,
    IntArrayRef size,
    std::optional<IntArrayRef> stride,
    bool device_guard = true) {
  if (self->sizes() == size && (!stride || self->strides() == stride)) {
    return self;
  }
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 61-90
```cpp
  int64_t storage_size = 1;
  if (stride) {
    self->set_sizes_and_strides(size, *stride);
    // NB: storage size can be different from numel.
    storage_size = storage_size_for(size, *stride);
  } else {
    self->set_sizes_contiguous(size);
    storage_size = self->numel();
  }
  maybe_resize_storage_mps(self, storage_size);

  return self;
}

Tensor empty_mps(
    IntArrayRef size,
    std::optional<ScalarType> dtype_opt,
    std::optional<Layout> layout_opt,
    std::optional<Device> device_opt,
    std::optional<bool> pin_memory_opt,
    std::optional<c10::MemoryFormat> memory_format_opt) {

  return at::detail::empty_mps(size, dtype_opt, layout_opt, device_opt, pin_memory_opt, memory_format_opt);
}

Tensor empty_strided_mps(
    IntArrayRef size,
    IntArrayRef stride,
    std::optional<ScalarType> dtype_opt,
    std::optional<Layout> layout_opt,
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。

### Lines 91-120
```cpp
    std::optional<Device> device_opt,
    std::optional<bool> pin_memory_opt) {
  check_size_nonnegative(size);
  // empty memory formatempty
  auto t = at::native::empty_mps(
      {0},
      dtype_opt,
      layout_opt,
      device_opt,
      pin_memory_opt);
  resize_impl_mps_(t.unsafeGetTensorImpl(), size, stride);
  return t;
}

const Tensor& resize_mps_(
    const Tensor& self,
    IntArrayRef size,
    std::optional<MemoryFormat> optional_memory_format) {
  if (self.has_names()) {
    return resize_named_tensor_(self, size, optional_memory_format);
  }
  auto* self_ = self.unsafeGetTensorImpl();
  int64_t old_storage_nbytes = self_->unsafe_storage() ? self_->unsafe_storage().nbytes() : 0;
  resize_impl_mps_(self_, size, /*stride=*/std::nullopt);
  if (optional_memory_format.has_value()) {
    auto memory_format =
        optional_memory_format.value();
    TORCH_CHECK(
        memory_format != MemoryFormat::Preserve,
        "Unsupported memory format",
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- EN: Conditional control flow selects different paths according to shapes, flags, backend capabilities, or operator modes.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。
- CN: 条件控制流会依据形状、标志位、后端能力或算子模式选择不同执行路径。

### Lines 121-150
```cpp
        memory_format);
    self_->empty_tensor_restride(memory_format);
  }
  // See Note [Enabling Deterministic Operations]
  if (C10_UNLIKELY(at::globalContext().deterministicAlgorithms() && at::globalContext().deterministicFillUninitializedMemory())) {
    at::native::fill_resize_deterministic_(self, old_storage_nbytes);
  }
  return self;
}

Tensor& set_mps_(Tensor& result) {
  caffe2::TypeMeta dtype = result.dtype();
  Storage storage(
      Storage::use_byte_size_t(),
      0,
      at::mps::GetMPSAllocator(),
      true);
  result.set_(storage, 0, {0}, {});
  TORCH_INTERNAL_ASSERT(dtype == result.dtype());
  return result;
}

Tensor& set_storage_mps_(Tensor& result, Storage storage, int64_t storage_offset, IntArrayRef size, IntArrayRef stride) {
  checkSetStorage(result, std::move(storage), storage_offset, size, stride);
  //std::cout << "set storage_mps " << storage_offset << " stride " << stride << std::endl;
  result.unsafeGetTensorImpl()->set_storage_offset(storage_offset);
  std::optional<IntArrayRef> stride_opt = stride.data() != nullptr ?
                                          std::optional<IntArrayRef>(stride) : std::nullopt;
  at::native::resize_impl_mps_(result.unsafeGetTensorImpl(), size, stride_opt);
  return result;
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: The main callable definitions or declarations in this block are set_mps_, set_storage_mps_, concentrating a specific part of the operator behavior.
- EN: Runtime guards validate tensor shapes, dtypes, layout assumptions, or backend support before execution continues.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 这一段的主要可调用定义或声明包括 set_mps_, set_storage_mps_，它们承载了某一部分算子行为的核心逻辑。
- CN: 运行时保护逻辑会在继续执行前校验张量形状、数据类型、布局假设或后端支持情况。

### Lines 151-166
```cpp
}

Tensor _efficientzerotensor_mps(IntArrayRef size,
    std::optional<ScalarType> dtype,
    std::optional<Layout> layout,
    std::optional<Device> device,
    std::optional<bool> pin_memory) {
    auto device_ = device_or_default(device);
    auto allocator = at::native::ZeroTensorAllocator(device_);
    auto dtype_ = dtype_or_default(dtype);
    auto zero_ks = at::DispatchKeySet(c10::DispatchKey::MPS) | at::DispatchKeySet(c10::DispatchKey::ZeroTensor);
    auto out = at::detail::empty_generic(size, &allocator, zero_ks, dtype_, std::nullopt);
    return out;
}

} // namespace at::native
```
- EN: This range adjusts namespace or scope boundaries so the implementation remains inside the intended ATen/backend module.
- EN: Tensor-oriented expressions dominate this block, showing direct manipulation of ATen tensor metadata, storage, or values.
- EN: The segment closes with direct return paths, indicating a compact helper or forwarding routine.
- CN: 这一段调整了命名空间或作用域边界，使实现保持在预期的 ATen/后端模块内部。
- CN: 该代码块以 Tensor 相关表达式为主，说明它直接处理 ATen 张量的元数据、存储或数值。
- CN: 这一段以直接返回路径收尾，说明它更像是紧凑的辅助函数或转发例程。

## Key Concepts / 关键概念

- EN: Backend focus: MPS backend.
- CN: 后端重点：MPS 后端。
- EN: Shape, dtype, layout, or backend validation is a first-class concern.
- CN: 形状、数据类型、布局或后端校验是该文件的重要关注点。
- EN: Notable symbols: maybe_resize_storage_mps, resize_impl_mps_, empty_mps, empty_strided_mps, resize_mps_, set_mps_, set_storage_mps_, _efficientzerotensor_mps.
- CN: 重要符号：maybe_resize_storage_mps, resize_impl_mps_, empty_mps, empty_strided_mps, resize_mps_, set_mps_, set_storage_mps_, _efficientzerotensor_mps。

## Dependencies / 依赖关系

- EN: Primary internal headers: `ATen/ATen.h, ATen/Tensor.h, ATen/Utils.h, torch/library.h, ATen/mps/EmptyTensor.h, ATen/mps/MPSDevice.h, ATen/native/Resize.h, ATen/native/ResizeCommon.h, ATen/native/mps/Copy.h, ATen/native/mps/TensorFactory.h, ATen/Dispatch.h, ATen/Functions.h`.
- CN: 主要内部头文件：`ATen/ATen.h, ATen/Tensor.h, ATen/Utils.h, torch/library.h, ATen/mps/EmptyTensor.h, ATen/mps/MPSDevice.h, ATen/native/Resize.h, ATen/native/ResizeCommon.h, ATen/native/mps/Copy.h, ATen/native/mps/TensorFactory.h, ATen/Dispatch.h, ATen/Functions.h`。
- EN: External/system headers: `utility`.
- CN: 外部/系统头文件：`utility`。
- EN: The implementation revolves around symbols such as `maybe_resize_storage_mps, resize_impl_mps_, empty_mps, empty_strided_mps, resize_mps_, set_mps_, set_storage_mps_, _efficientzerotensor_mps`.
- CN: 实现围绕 `maybe_resize_storage_mps, resize_impl_mps_, empty_mps, empty_strided_mps, resize_mps_, set_mps_, set_storage_mps_, _efficientzerotensor_mps` 等符号展开。
