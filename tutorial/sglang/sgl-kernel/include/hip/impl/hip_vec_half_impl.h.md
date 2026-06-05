# hip_vec_half_impl.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/include/hip/impl/hip_vec_half_impl.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Types and data layout
```cpp
#pragma once

#if USE_ROCM

#include <hip/hip_common.h>
#include <hip/hip_fp16.h>

// Adapted from flashinfer-rocm [PR#491](https://github.com/flashinfer-ai/flashinfer/pull/491)

using half = __half;
using half2 = __half2;

namespace sgl_hip {

// half x 1
template <>
struct vec_t<half, 1> {
  half data;

  SGL_HIP_INLINE half& operator[](size_t i) {
    return ((half*)(&data))[i];
  }
```
**EN:** This section defines `vec_t`, `half`, `half2`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`vec_t`、`half`、`half2`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 23-44: Templates, aliases, and constants
```cpp
  SGL_HIP_INLINE const half& operator[](size_t i) const {
    return ((const half*)(&data))[i];
  }
  SGL_HIP_INLINE half* ptr() {
    return reinterpret_cast<half*>(&data);
  }
  SGL_HIP_INLINE void load(const half* ptr);
  SGL_HIP_INLINE void store(half* ptr) const;
  template <typename T>
  SGL_HIP_INLINE void cast_from(const vec_t<T, 1>& src) {
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

### Lines 45-66: Types and data layout
```cpp
SGL_HIP_INLINE void vec_t<half, 1>::load(const half* ptr) {
  data = *ptr;
}

SGL_HIP_INLINE void vec_t<half, 1>::store(half* ptr) const {
  *ptr = data;
}

// half x 2
template <>
struct vec_t<half, 2> {
  half2 data;

  SGL_HIP_INLINE half& operator[](size_t i) {
    return ((half*)(&data))[i];
  }
  SGL_HIP_INLINE const half& operator[](size_t i) const {
    return ((const half*)(&data))[i];
  }
  SGL_HIP_INLINE half* ptr() {
    return reinterpret_cast<half*>(&data);
  }
```
**EN:** This section defines `vec_t`, `load`, `store`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`vec_t`、`load`、`store`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 67-89: Templates, aliases, and constants
```cpp
  SGL_HIP_INLINE void load(const half* ptr);
  SGL_HIP_INLINE void store(half* ptr) const;
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

SGL_HIP_INLINE void vec_t<half, 2>::load(const half* ptr) {
  data = *((half2*)ptr);
}

SGL_HIP_INLINE void vec_t<half, 2>::store(half* ptr) const {
  *((half2*)ptr) = data;
}
```
**EN:** This section defines `cast_from`, `cast_load`, `cast_store`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`cast_from`、`cast_load`、`cast_store`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 90-111: Types and data layout
```cpp

// half x 4

template <>
struct vec_t<half, 4> {
  uint2 data;

  SGL_HIP_INLINE half& operator[](size_t i) {
    return ((half*)(&data))[i];
  }
  SGL_HIP_INLINE const half& operator[](size_t i) const {
    return ((const half*)(&data))[i];
  }
  SGL_HIP_INLINE half* ptr() {
    return reinterpret_cast<half*>(&data);
  }
  SGL_HIP_INLINE void load(const half* ptr);
  SGL_HIP_INLINE void store(half* ptr) const;
  template <typename T>
  SGL_HIP_INLINE void cast_from(const vec_t<T, 4>& src) {
    cast_from_impl(*this, src);
  }
```
**EN:** This section defines `vec_t`, `ptr`, `cast_from`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`vec_t`、`ptr`、`cast_from`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 112-135: Types and data layout
```cpp
  template <typename T>
  SGL_HIP_INLINE void cast_load(const T* ptr) {
    cast_load_impl(*this, ptr);
  }
  template <typename T>
  SGL_HIP_INLINE void cast_store(T* ptr) const {
    cast_store_impl(ptr, *this);
  }
};

SGL_HIP_INLINE void vec_t<half, 4>::load(const half* ptr) {
  data = *((uint2*)ptr);
}

SGL_HIP_INLINE void vec_t<half, 4>::store(half* ptr) const {
  *((uint2*)ptr) = data;
}

// half x 8 or more

template <size_t vec_size>
struct vec_t<half, vec_size> {
  uint4 data[vec_size / 8];
```
**EN:** This section defines `vec_t`, `cast_load`, `cast_store`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`vec_t`、`cast_load`、`cast_store`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 136-156: Control flow and branching
```cpp
  SGL_HIP_INLINE half& operator[](size_t i) {
    return ((half*)data)[i];
  }
  SGL_HIP_INLINE const half& operator[](size_t i) const {
    return ((const half*)data)[i];
  }
  SGL_HIP_INLINE half* ptr() {
    return reinterpret_cast<half*>(&data);
  }
  SGL_HIP_INLINE void load(const half* ptr) {
#pragma unroll
    for (size_t i = 0; i < vec_size / 8; ++i) {
      data[i] = ((uint4*)ptr)[i];
    }
  }
  SGL_HIP_INLINE void store(half* ptr) const {
#pragma unroll
    for (size_t i = 0; i < vec_size / 8; ++i) {
      ((uint4*)ptr)[i] = data[i];
    }
  }
```
**EN:** This section drives `ptr`, `load`, `store` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`ptr`、`load`、`store`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 157-170: Templates, aliases, and constants
```cpp
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
```
**EN:** This section defines `cast_from`, `cast_load`, `cast_store`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`cast_from`、`cast_load`、`cast_store`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 171-172: Local implementation details
```cpp
}  // namespace sgl_hip
#endif
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **HIP runtime / HIP 运行时**: Uses HIP APIs or AMD-specific intrinsics for portability. / 使用 HIP API 或 AMD 特定 intrinsic 实现可移植性。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: None explicitly listed / 未显式列出
- **External headers / 外部头文件**: `hip/hip_common.h`, `hip/hip_fp16.h`
- **Path context / 路径上下文**: include / hip / impl / hip_vec_half_impl.h
