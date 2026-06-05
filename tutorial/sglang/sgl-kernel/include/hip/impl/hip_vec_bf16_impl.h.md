# hip_vec_bf16_impl.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/include/hip/impl/hip_vec_bf16_impl.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Templates, aliases, and constants
```cpp
#pragma once

#if USE_ROCM

#include <hip/hip_bf16.h>
#include <hip/hip_common.h>

// Adapted from flashinfer-rocm [PR#491](https://github.com/flashinfer-ai/flashinfer/pull/491)

using nv_bfloat16 = __hip_bfloat16;
using nv_bfloat162 = __hip_bfloat162;

__BF16_HOST_DEVICE_STATIC__ __hip_bfloat162 make_bfloat162(const __hip_bfloat16 x, const __hip_bfloat16 y) {
  __hip_bfloat162 t;
  t.x = x;
  t.y = y;
  return t;
}

namespace sgl_hip {
```
**EN:** This section defines `make_bfloat162`, `nv_bfloat16`, `nv_bfloat162`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`make_bfloat162`、`nv_bfloat16`、`nv_bfloat162`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 22-44: Types and data layout
```cpp
// nv_bfloat16 x 1
template <>
struct vec_t<nv_bfloat16, 1> {
  nv_bfloat16 data;
  SGL_HIP_INLINE nv_bfloat16& operator[](size_t i) {
    return ((nv_bfloat16*)(&data))[i];
  }
  SGL_HIP_INLINE const nv_bfloat16& operator[](size_t i) const {
    return ((const nv_bfloat16*)(&data))[i];
  }
  SGL_HIP_INLINE nv_bfloat16* ptr() {
    return reinterpret_cast<nv_bfloat16*>(&data);
  }
  SGL_HIP_INLINE void load(const nv_bfloat16* ptr);
  SGL_HIP_INLINE void store(nv_bfloat16* ptr) const;
  template <typename T>
  SGL_HIP_INLINE void cast_from(const vec_t<T, 1>& src) {
    cast_from_impl(*this, src);
  }
  template <typename T>
  SGL_HIP_INLINE void cast_load(const T* ptr) {
    cast_load_impl(*this, ptr);
  }
```
**EN:** This section defines `vec_t`, `ptr`, `cast_from`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`vec_t`、`ptr`、`cast_from`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 45-66: Types and data layout
```cpp
  template <typename T>
  SGL_HIP_INLINE void cast_store(T* ptr) const {
    cast_store_impl(ptr, *this);
  }
};

SGL_HIP_INLINE void vec_t<nv_bfloat16, 1>::load(const nv_bfloat16* ptr) {
  data = *ptr;
}

SGL_HIP_INLINE void vec_t<nv_bfloat16, 1>::store(nv_bfloat16* ptr) const {
  *ptr = data;
}

// nv_bfloat16 x 2
template <>
struct vec_t<nv_bfloat16, 2> {
  nv_bfloat162 data;

  SGL_HIP_INLINE nv_bfloat16& operator[](size_t i) {
    return ((nv_bfloat16*)(&data))[i];
  }
```
**EN:** This section defines `vec_t`, `cast_store`, `load`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`vec_t`、`cast_store`、`load`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 67-88: Templates, aliases, and constants
```cpp
  SGL_HIP_INLINE const nv_bfloat16& operator[](size_t i) const {
    return ((const nv_bfloat16*)(&data))[i];
  }
  SGL_HIP_INLINE nv_bfloat16* ptr() {
    return reinterpret_cast<nv_bfloat16*>(&data);
  }
  SGL_HIP_INLINE void load(const nv_bfloat16* ptr);
  SGL_HIP_INLINE void store(nv_bfloat16* ptr) const;
  template <typename T>
  SGL_HIP_INLINE void cast_from(const vec_t<T, 2>& src) {
    cast_from_impl(*this, src);
  }
  template <typename T>
  SGL_HIP_INLINE void cast_load(const T* ptr) {
    cast_load_impl(*this, ptr);
  }
  template <typename T>
  SGL_HIP_INLINE void cast_store(T* ptr) const {
    cast_store_impl(ptr, *this);
  }
};
```
**EN:** This section defines `ptr`, `cast_from`, `cast_load`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`ptr`、`cast_from`、`cast_load`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 89-109: Types and data layout
```cpp
SGL_HIP_INLINE void vec_t<nv_bfloat16, 2>::load(const nv_bfloat16* ptr) {
  data = *((nv_bfloat162*)ptr);
}

SGL_HIP_INLINE void vec_t<nv_bfloat16, 2>::store(nv_bfloat16* ptr) const {
  *((nv_bfloat162*)ptr) = data;
}

template <>
struct vec_t<nv_bfloat16, 4> {
  uint2 data;

  SGL_HIP_INLINE nv_bfloat16& operator[](size_t i) {
    return ((nv_bfloat16*)(&data))[i];
  }
  SGL_HIP_INLINE const nv_bfloat16& operator[](size_t i) const {
    return ((const nv_bfloat16*)(&data))[i];
  }
  SGL_HIP_INLINE nv_bfloat16* ptr() {
    return reinterpret_cast<nv_bfloat16*>(&data);
  }
```
**EN:** This section defines `vec_t`, `load`, `store`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`vec_t`、`load`、`store`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 110-132: Templates, aliases, and constants
```cpp
  SGL_HIP_INLINE void load(const nv_bfloat16* ptr);
  SGL_HIP_INLINE void store(nv_bfloat16* ptr) const;
  template <typename T>
  SGL_HIP_INLINE void cast_from(const vec_t<T, 4>& src) {
    cast_from_impl(*this, src);
  }
  template <typename T>
  SGL_HIP_INLINE void cast_load(const T* ptr) {
    cast_load_impl(*this, ptr);
  }
  template <typename T>
  SGL_HIP_INLINE void cast_store(T* ptr) const {
    cast_store_impl(ptr, *this);
  }
};

SGL_HIP_INLINE void vec_t<nv_bfloat16, 4>::load(const nv_bfloat16* ptr) {
  data = *((uint2*)ptr);
}

SGL_HIP_INLINE void vec_t<nv_bfloat16, 4>::store(nv_bfloat16* ptr) const {
  *((uint2*)ptr) = data;
}
```
**EN:** This section defines `cast_from`, `cast_load`, `cast_store`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`cast_from`、`cast_load`、`cast_store`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 133-154: Types and data layout
```cpp

// nv_bfloat16 x 8 or more

template <size_t vec_size>
struct vec_t<nv_bfloat16, vec_size> {
  uint4 data[vec_size / 8];

  SGL_HIP_INLINE nv_bfloat16& operator[](size_t i) {
    return ((nv_bfloat16*)data)[i];
  }
  SGL_HIP_INLINE const nv_bfloat16& operator[](size_t i) const {
    return ((const nv_bfloat16*)data)[i];
  }
  SGL_HIP_INLINE nv_bfloat16* ptr() {
    return reinterpret_cast<nv_bfloat16*>(&data);
  }
  SGL_HIP_INLINE void load(const nv_bfloat16* ptr) {
#pragma unoll
    for (size_t i = 0; i < vec_size / 8; ++i) {
      data[i] = ((uint4*)ptr)[i];
    }
  }
```
**EN:** This section defines `vec_t`, `ptr`, `load`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`vec_t`、`ptr`、`load`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 155-176: Templates, aliases, and constants
```cpp
  SGL_HIP_INLINE void store(nv_bfloat16* ptr) const {
#pragma unoll
    for (size_t i = 0; i < vec_size / 8; ++i) {
      ((uint4*)ptr)[i] = data[i];
    }
  }
  template <typename T>
  SGL_HIP_INLINE void cast_from(const vec_t<T, vec_size>& src) {
    cast_from_impl(*this, src);
  }
  template <typename T>
  SGL_HIP_INLINE void cast_load(const T* ptr) {
    cast_load_impl(*this, ptr);
  }
  template <typename T>
  SGL_HIP_INLINE void cast_store(T* ptr) const {
    cast_store_impl(ptr, *this);
  }
};

}  // namespace sgl_hip
```
**EN:** This section defines `store`, `cast_from`, `cast_load`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`store`、`cast_from`、`cast_load`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 177-177: Local implementation details
```cpp
#endif
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **HIP runtime / HIP 运行时**: Uses HIP APIs or AMD-specific intrinsics for portability. / 使用 HIP API 或 AMD 特定 intrinsic 实现可移植性。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: None explicitly listed / 未显式列出
- **External headers / 外部头文件**: `hip/hip_bf16.h`, `hip/hip_common.h`
- **Path context / 路径上下文**: include / hip / impl / hip_vec_bf16_impl.h
