# hip_vec_fp32_impl.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/include/hip/impl/hip_vec_fp32_impl.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Namespace and shared declarations
```cpp
#pragma once

#if USE_ROCM

#include <hip/hip_common.h>

// Adapted from flashinfer-rocm [PR#491](https://github.com/flashinfer-ai/flashinfer/pull/491)

namespace sgl_hip {

template <>
struct vec_t<float, 1> {
  float data;

  SGL_HIP_INLINE float& operator[](size_t i) {
    return ((float*)(&data))[i];
  }
  SGL_HIP_INLINE const float& operator[](size_t i) const {
    return ((const float*)(&data))[i];
  }
  SGL_HIP_INLINE float* ptr() {
    return reinterpret_cast<float*>(&data);
  }
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 24-46: Templates, aliases, and constants
```cpp
  SGL_HIP_INLINE void load(const float* ptr);
  SGL_HIP_INLINE void store(float* ptr) const;
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

SGL_HIP_INLINE void vec_t<float, 1>::load(const float* ptr) {
  data = *ptr;
}

SGL_HIP_INLINE void vec_t<float, 1>::store(float* ptr) const {
  *ptr = data;
}
```
**EN:** This section defines `cast_from`, `cast_load`, `cast_store`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`cast_from`、`cast_load`、`cast_store`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 47-68: Types and data layout
```cpp

// float x 2

template <>
struct vec_t<float, 2> {
  float2 data;

  SGL_HIP_INLINE float& operator[](size_t i) {
    return ((float*)(&data))[i];
  }
  SGL_HIP_INLINE const float& operator[](size_t i) const {
    return ((const float*)(&data))[i];
  }
  SGL_HIP_INLINE float* ptr() {
    return reinterpret_cast<float*>(&data);
  }
  SGL_HIP_INLINE void load(const float* ptr);
  SGL_HIP_INLINE void store(float* ptr) const;
  template <typename T>
  SGL_HIP_INLINE void cast_from(const vec_t<T, 2>& src) {
    cast_from_impl(*this, src);
  }
```
**EN:** This section defines `vec_t`, `ptr`, `cast_from`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`vec_t`、`ptr`、`cast_from`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 69-91: Types and data layout
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

SGL_HIP_INLINE void vec_t<float, 2>::load(const float* ptr) {
  data = *((float2*)ptr);
}

SGL_HIP_INLINE void vec_t<float, 2>::store(float* ptr) const {
  *((float2*)ptr) = data;
}

// float x 4 or more
template <size_t vec_size>
struct vec_t<float, vec_size> {
  float4 data[vec_size / 4];
```
**EN:** This section defines `vec_t`, `cast_load`, `cast_store`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`vec_t`、`cast_load`、`cast_store`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 92-112: Control flow and branching
```cpp
  SGL_HIP_INLINE float& operator[](size_t i) {
    return ((float*)(data))[i];
  }
  SGL_HIP_INLINE const float& operator[](size_t i) const {
    return ((const float*)(data))[i];
  }
  SGL_HIP_INLINE float* ptr() {
    return reinterpret_cast<float*>(&data);
  }
  SGL_HIP_INLINE void load(const float* ptr) {
#pragma unroll
    for (size_t i = 0; i < vec_size / 4; ++i) {
      data[i] = ((float4*)ptr)[i];
    }
  }
  SGL_HIP_INLINE void store(float* ptr) const {
#pragma unroll
    for (size_t i = 0; i < vec_size / 4; ++i) {
      ((float4*)ptr)[i] = data[i];
    }
  }
```
**EN:** This section drives `ptr`, `load`, `store` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`ptr`、`load`、`store`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 113-128: Templates, aliases, and constants
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

}  // namespace sgl_hip
```
**EN:** This section defines `cast_from`, `cast_load`, `cast_store`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`cast_from`、`cast_load`、`cast_store`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 129-129: Local implementation details
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
- **External headers / 外部头文件**: `hip/hip_common.h`
- **Path context / 路径上下文**: include / hip / impl / hip_vec_fp32_impl.h
