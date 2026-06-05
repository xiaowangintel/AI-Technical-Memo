# utils.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/elementwise/utils.cuh`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Device helpers and synchronization
```cpp
// Adapted from https://github.com/deepseek-ai/DeepEP/blob/main/csrc/kernels/utils.cuh

#pragma once

#include <cuda_bf16.h>
#include <cuda_runtime.h>

#include <cstdint>

#ifndef USE_MUSA
__forceinline__ __device__ int get_lane_id() {
  int lane_id;
  asm("mov.s32 %0, %laneid;" : "=r"(lane_id));
  return lane_id;
}
#else
constexpr int WarpSize = 32;
__forceinline__ __device__ int get_lane_id() {
  return threadIdx.x % WarpSize;
}
```
**EN:** This section implements `get_lane_id`, `WarpSize`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`get_lane_id`、`WarpSize`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 21-39: Device helpers and synchronization
```cpp
#endif

int ceil_div(int a, int b) {
  return (a + b - 1) / b;
}

__device__ __forceinline__ void st_na_global_v1(const int* ptr, int v) {
  asm volatile("st.global.L1::no_allocate.s32 [%0], %1;" ::"l"(ptr), "r"(v) : "memory");
}

__device__ __forceinline__ void st_na_global_v2(const int2* ptr, const int2& v) {
  asm volatile("st.global.L1::no_allocate.v2.s32 [%0], {%1, %2};" ::"l"(ptr), "r"(v.x), "r"(v.y) : "memory");
}

__device__ __forceinline__ void st_na_global_v4(const int4* ptr, const int4& v) {
  asm volatile(
      "st.global.L1::no_allocate.v4.s32 [%0], {%1, %2, %3, %4};" ::"l"(ptr), "r"(v.x), "r"(v.y), "r"(v.z), "r"(v.w)
      : "memory");
}
```
**EN:** This section implements `ceil_div`, `st_na_global_v1`, `st_na_global_v2`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`ceil_div`、`st_na_global_v1`、`st_na_global_v2`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 40-59: Device helpers and synchronization
```cpp

__device__ __forceinline__ int ld_na_global_v1(const int* ptr) {
  int r;
#ifdef USE_L2_HINT
  asm volatile("ld.global.nc.L1::no_allocate.L2::128B.s32 %0, [%1];" : "=r"(r) : "l"(ptr));
#else
  asm volatile("ld.global.nc.L1::no_allocate.s32 %0, [%1];" : "=r"(r) : "l"(ptr));
#endif
  return r;
}

__device__ __forceinline__ int2 ld_na_global_v2(const int2* ptr) {
  int2 r;
#ifdef USE_L2_HINT
  asm volatile("ld.global.nc.L1::no_allocate.L2::128B.v2.s32 {%0, %1}, [%2];" : "=r"(r.x), "=r"(r.y) : "l"(ptr));
#else
  asm volatile("ld.global.nc.L1::no_allocate.v2.s32 {%0, %1}, [%2];" : "=r"(r.x), "=r"(r.y) : "l"(ptr));
#endif
  return r;
}
```
**EN:** This section implements `ld_na_global_v1`, `ld_na_global_v2`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`ld_na_global_v1`、`ld_na_global_v2`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 60-79: Device helpers and synchronization
```cpp

__device__ __forceinline__ int4 ld_na_global_v4(const int4* ptr) {
  int4 r;
#ifdef USE_L2_HINT
  asm volatile("ld.global.nc.L1::no_allocate.L2::128B.v4.s32 {%0, %1, %2, %3}, [%4];"
               : "=r"(r.x), "=r"(r.y), "=r"(r.z), "=r"(r.w)
               : "l"(ptr));
#else
  asm volatile("ld.global.nc.L1::no_allocate.v4.s32 {%0, %1, %2, %3}, [%4];"
               : "=r"(r.x), "=r"(r.y), "=r"(r.z), "=r"(r.w)
               : "l"(ptr));
#endif
  return r;
}

__device__ __forceinline__ void prefetch_L2(const void* p) {
#if defined(ENABLE_L2_PREFETCH)
  asm volatile("prefetch.global.L2 [%0];" ::"l"(p));
#endif
}
```
**EN:** This section implements `ld_na_global_v4`, `prefetch_L2`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`ld_na_global_v4`、`prefetch_L2`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

## Key Concepts / 关键概念
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **MUSA portability / MUSA 可移植性**: Contains conditional logic for the MUSA toolchain or runtime. / 包含面向 MUSA 工具链或运行时的条件分支。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: None explicitly listed / 未显式列出
- **External headers / 外部头文件**: `cuda_bf16.h`, `cuda_runtime.h`, `cstdint`
- **Path context / 路径上下文**: elementwise / utils.cuh
