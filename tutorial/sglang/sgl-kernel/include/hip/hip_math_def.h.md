# hip_math_def.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/include/hip/hip_math_def.h`
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

#ifdef USE_ROCM
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 20-37: Namespace and shared declarations
```cpp
#include <hip/hip_bf16.h>
#include <hip/hip_common.h>
#include <hip/hip_fp16.h>

// Adapted from flashinfer-rocm [PR#491](https://github.com/flashinfer-ai/flashinfer/pull/491)

namespace amdgpu {

template <typename T>
__forceinline__ __device__ T shfl_xor_sync(unsigned mask, T var, int laneMask, int width = warpSize);

template <typename srcDtype, typename destDtype>
__forceinline__ __device__ destDtype cast(srcDtype val) {
  // Generic fallback used by most scalar types (int/float/double/etc).
  // Specific types like fp16/bf16 have explicit specializations below.
  return static_cast<destDtype>(val);
}
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 38-57: Device helpers and synchronization
```cpp
// specialization
template <>
__forceinline__ __device__ float shfl_xor_sync(unsigned mask, float var, int laneMask, int width) {
  return __shfl_xor(var, laneMask, width);
}

template <>
__forceinline__ __device__ int shfl_xor_sync(unsigned mask, int var, int laneMask, int width) {
  return __shfl_xor(var, laneMask, width);
}

template <>
__forceinline__ __device__ float cast<float, float>(float val) {
  return val;
}

template <>
__forceinline__ __device__ float cast<__half, float>(__half val) {
  return __half2float(val);
}
```
**EN:** This section implements `shfl_xor_sync`, `__shfl_xor`, `__half2float`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`shfl_xor_sync`、`__shfl_xor`、`__half2float`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 58-75: Device helpers and synchronization
```cpp

template <>
__forceinline__ __device__ float cast<__hip_bfloat16, float>(__hip_bfloat16 val) {
  return __bfloat162float(val);
}

template <>
__forceinline__ __device__ __half cast<float, __half>(float fval) {
  return __float2half(fval);
}

template <>
__forceinline__ __device__ __hip_bfloat16 cast<float, __hip_bfloat16>(float fval) {
  return __float2bfloat16(fval);
}

}  // namespace amdgpu
```
**EN:** This section implements `__bfloat162float`, `__float2half`, `__float2bfloat16`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__bfloat162float`、`__float2half`、`__float2bfloat16`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 76-96: Device helpers and synchronization
```cpp
template <typename T>
__forceinline__ __device__ T __shfl_xor_sync(unsigned mask, T var, int laneMask, int width = warpSize) {
  return amdgpu::shfl_xor_sync(mask, var, laneMask, width);
}

template <typename srcDtype>
__device__ __forceinline__ float castToFloat(srcDtype val) {
  return amdgpu::cast<srcDtype, float>(val);
}

template <typename dstDtype>
__device__ __forceinline__ dstDtype castFromFloat(float val) {
  return amdgpu::cast<float, dstDtype>(val);
}

// operator overload to support flashinfer
__host__ __device__ __forceinline__ __half operator*(const __half& x, const __half& y) {
  __half h_x = x;
  __half h_y = y;
  return __hmul(h_x, h_y);
}
```
**EN:** This section implements `__shfl_xor_sync`, `castToFloat`, `castFromFloat`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__shfl_xor_sync`、`castToFloat`、`castFromFloat`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 97-97: Local implementation details
```cpp

```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 98-98: Local implementation details
```cpp
#endif
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **HIP runtime / HIP 运行时**: Uses HIP APIs or AMD-specific intrinsics for portability. / 使用 HIP API 或 AMD 特定 intrinsic 实现可移植性。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: None explicitly listed / 未显式列出
- **External headers / 外部头文件**: `hip/hip_bf16.h`, `hip/hip_common.h`, `hip/hip_fp16.h`
- **Path context / 路径上下文**: include / hip / hip_math_def.h
