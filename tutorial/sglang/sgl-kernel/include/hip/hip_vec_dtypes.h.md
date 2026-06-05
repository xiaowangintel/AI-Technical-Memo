# hip_vec_dtypes.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/include/hip/hip_vec_dtypes.h`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Local implementation details
```cpp
/* Copyright 2025 SGLang Team. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
==============================================================================*/

#pragma once

#if USE_ROCM
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 20-38: Types and data layout
```cpp
#include <hip/hip_bf16.h>
#include <hip/hip_common.h>
#include <hip/hip_fp16.h>

// Adapted from flashinfer-rocm [PR#491](https://github.com/flashinfer-ai/flashinfer/pull/491)d

#define SGL_HIP_INLINE inline __attribute__((always_inline)) __device__

namespace sgl_hip {

template <typename float_t, size_t vec_size>
struct vec_t;

template <typename srcDtype, typename dstDtype, size_t vec_size>
SGL_HIP_INLINE void cast_load_impl(vec_t<dstDtype, vec_size>& dst, const srcDtype* src);

template <typename srcDtype, typename dstDtype, size_t vec_size>
SGL_HIP_INLINE void cast_store_impl(dstDtype* dst_ptr, const vec_t<srcDtype, vec_size>& src);
```
**EN:** This section defines `vec_t`, `cast_load_impl`, `cast_store_impl`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`vec_t`、`cast_load_impl`、`cast_store_impl`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 39-57: Types and data layout
```cpp
template <typename float_t, size_t vec_size>
struct vec_t {
  SGL_HIP_INLINE float_t& operator[](size_t i);
  SGL_HIP_INLINE const float_t& operator[](size_t i) const;
  SGL_HIP_INLINE float_t* ptr();

  SGL_HIP_INLINE void load(const float_t* ptr);
  SGL_HIP_INLINE void store(float_t* ptr) const;

  template <typename T>
  SGL_HIP_INLINE void cast_from(const vec_t<T, vec_size>& src);
  template <typename T>
  SGL_HIP_INLINE void cast_load(const T* ptr);
  template <typename T>
  SGL_HIP_INLINE void cast_store(T* ptr) const;
};

}  // namespace sgl_hip
```
**EN:** This section defines `vec_t`, `ptr`, `load`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`vec_t`、`ptr`、`load`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 58-72: Namespace and shared declarations
```cpp
// **** impl *****

namespace sgl_hip {

template <typename srcDtype, typename dstDtype, size_t vec_size>
SGL_HIP_INLINE void cast_load_impl(vec_t<dstDtype, vec_size>& dst, const srcDtype* src_ptr) {
  if constexpr (std::is_same<srcDtype, dstDtype>::value) {
    dst.load(src_ptr);
  } else {
    vec_t<srcDtype, vec_size> tmp;
    tmp.load(src_ptr);
    dst.cast_from(tmp);
  }
}
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 73-89: Templates, aliases, and constants
```cpp
template <typename srcDtype, typename dstDtype, size_t vec_size>
SGL_HIP_INLINE void cast_store_impl(dstDtype* dst_ptr, const vec_t<srcDtype, vec_size>& src) {
  if constexpr (std::is_same<srcDtype, dstDtype>::value) {
    src.store(dst_ptr);
  } else {
    vec_t<dstDtype, vec_size> tmp;
    tmp.cast_from(src);
    tmp.store(dst_ptr);
  }
}

template <typename float_t, size_t vec_size>
template <typename T>
SGL_HIP_INLINE void vec_t<float_t, vec_size>::cast_load(const T* ptr) {
  cast_load_impl(*this, ptr);
}
```
**EN:** This section defines `cast_store_impl`, `cast_load`, `store`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`cast_store_impl`、`cast_load`、`store`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 90-97: Templates, aliases, and constants
```cpp
template <typename float_t, size_t vec_size>
template <typename T>
SGL_HIP_INLINE void vec_t<float_t, vec_size>::cast_store(T* ptr) const {
  cast_store_impl(ptr, *this);
}

}  // namespace sgl_hip
```
**EN:** This section defines `cast_store`, `cast_store_impl`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`cast_store`、`cast_store_impl`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 98-101: Headers and compile-time setup
```cpp
#include "impl/hip_vec_bf16_impl.h"
#include "impl/hip_vec_fp32_impl.h"
#include "impl/hip_vec_half_impl.h"
#endif
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

## Key Concepts / 关键概念
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **HIP runtime / HIP 运行时**: Uses HIP APIs or AMD-specific intrinsics for portability. / 使用 HIP API 或 AMD 特定 intrinsic 实现可移植性。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `impl/hip_vec_bf16_impl.h`, `impl/hip_vec_fp32_impl.h`, `impl/hip_vec_half_impl.h`
- **External headers / 外部头文件**: `hip/hip_bf16.h`, `hip/hip_common.h`, `hip/hip_fp16.h`
- **Path context / 路径上下文**: include / hip / hip_vec_dtypes.h
