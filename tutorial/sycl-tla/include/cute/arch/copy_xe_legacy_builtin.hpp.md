# copy_xe_legacy_builtin.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/copy_xe_legacy_builtin.hpp`
- **EN:** Provides legacy Intel Xe copy builtins, SPIR-V wrappers, or typed front-ends.
- **CN:** 提供传统 Intel Xe 拷贝内建、SPIR-V 封装或类型化前端。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31
```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2025 Codeplay Software Ltd. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this
 * list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/
#pragma once
```
- **EN:** Carries the BSD-3-Clause license banner and enables one-time inclusion with `#pragma once`.
- **CN:** 给出 BSD-3-Clause 许可证声明，并通过 `#pragma once` 启用一次性包含保护。

### Lines 33-34
```cpp
#include <cute/util/sycl_vec.hpp>
#include "cute/config.hpp"
```
- **EN:** Imports `cute/util/sycl_vec.hpp` (related definitions from `cute/util/sycl_vec.hpp`); `cute/config.hpp` (core CuTe configuration macros and portability annotations).
- **CN:** 引入 `cute/util/sycl_vec.hpp`（来自 `cute/util/sycl_vec.hpp` 的相关定义）；`cute/config.hpp`（CuTe 核心配置宏与可移植性标注）。

### Lines 36-44
```cpp
#ifdef __SYCL_DEVICE_ONLY__
#define SYCL_DEVICE_BUILTIN(x) SYCL_EXTERNAL extern "C" x
#else
#define SYCL_DEVICE_BUILTIN(x)                                                 \
  inline x {                                                                   \
    CUTE_INVALID_CONTROL_PATH(                                                 \
        "Attempting to use a device built-in in host code.");                  \
  }
#endif
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 46-54
```cpp
#ifdef __SYCL_DEVICE_ONLY__
#define SYCL_DEVICE_OCL(x) SYCL_EXTERNAL x
#else
#define SYCL_DEVICE_OCL(x)                                                     \
  inline x {                                                                   \
    CUTE_INVALID_CONTROL_PATH(                                                 \
        "Attempting to use a device built-in in host code.");                  \
  }
#endif
```
- **EN:** Defines or forwards `SYCL_DEVICE_OCL` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_OCL`，作为该头文件可执行接口的一部分。

### Lines 57-58
```cpp
#undef __global
#define __global __attribute__((opencl_global))
```
- **EN:** Uses preprocessor conditions to select architecture-specific or build-specific behavior.
- **CN:** 使用预处理条件选择特定体系结构或构建配置下的行为。

### Lines 61-63
```cpp
#if defined(__SYCL_DEVICE_ONLY__) && defined(SYCL_INTEL_TARGET)
#define CUTE_ARCH_XE_COPY_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (__SYCL_DEVICE_ONLY__, CUTE_ARCH_XE_COPY_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（__SYCL_DEVICE_ONLY__, CUTE_ARCH_XE_COPY_ENABLED）只启用当前目标有效的构建路径。

### Lines 65-87
```cpp
namespace cute::detail
{
template<int ElementSize, int BlockWidth, int BlockHeight, int BlockCount>
struct XeSubgroup2DBlockPrefetch {
  static_assert(dependent_false<>, "Unsupported 2D Block Load Configuration.");
};
template<int ElementSize, int BlockWidth, int BlockHeight, int BlockCount>
struct XeSubgroup2DBlockLoad {
  static_assert(dependent_false<>, "Unsupported 2D Block Load Configuration.");
};
template<int ElementSize, int BlockWidth, int BlockHeight, int BlockCount>
struct XeSubgroup2DBlockLoadTransform {
  static_assert(dependent_false<>, "Unsupported 2D Block Load Configuration.");
};
template<int ElementSize, int BlockWidth, int BlockHeight, int BlockCount>
struct XeSubgroup2DBlockLoadTranspose {
  static_assert(dependent_false<>, "Unsupported 2D Block Load Configuration.");
};
template<int ElementSize, int BlockWidth, int BlockHeight, int BlockCount>
struct XeSubgroup2DBlockStore {
  static_assert(dependent_false<>, "Unsupported 2D Block Load Configuration.");
};
}
```
- **EN:** Enters or leaves namespace scope `cute::detail` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute::detail`，以便把相关符号组织在一起。

### Lines 89-98
```cpp
enum class CacheControl {
  kDefault   = 0,
  kL1UC_L3UC = 1, // Override to L1 uncached and L3 uncached
  kL1UC_L3C  = 2, // Override to L1 uncached and L3 cached
  kL1C_L3UC  = 3, // Override to L1 cached and L3 uncached
  kL1C_L3C   = 4, // Override to L1 cached and L3 cached
  kL1S_L3UC  = 5, // Override to L1 streaming load and L3 uncached
  kL1S_L3C   = 6, // Override to L1 streaming load and L3 cached
  kL1IAR_L3C = 7, // Override to L1 invalidate-after-read, and L3 cached
};
```
- **EN:** Defines `CacheControl` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `CacheControl`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 100-128
```cpp
// 8bits No transform No transpose
SYCL_DEVICE_BUILTIN(
    cute::intel::uchar __builtin_IB_subgroup_block_read_flat_u8_m1k16v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort __builtin_IB_subgroup_block_read_flat_u8_m1k32v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort2 __builtin_IB_subgroup_block_read_flat_u8_m2k32v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort4 __builtin_IB_subgroup_block_read_flat_u8_m4k32v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort8 __builtin_IB_subgroup_block_read_flat_u8_m8k32v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort16 __builtin_IB_subgroup_block_read_flat_u8_m16k32v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort32 __builtin_IB_subgroup_block_read_flat_u8_m32k32v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 130-165
```cpp
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort __builtin_IB_subgroup_block_read_flat_u8_m1k16v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort2 __builtin_IB_subgroup_block_read_flat_u8_m1k32v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort4 __builtin_IB_subgroup_block_read_flat_u8_m2k32v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort8 __builtin_IB_subgroup_block_read_flat_u8_m4k32v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort16 __builtin_IB_subgroup_block_read_flat_u8_m8k32v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uchar16 __builtin_IB_subgroup_block_read_cacheopts_u8_m8k16v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control ));      
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort32 __builtin_IB_subgroup_block_read_flat_u8_m16k32v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort64 __builtin_IB_subgroup_block_read_flat_u8_m32k32v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uchar64 __builtin_IB_subgroup_block_read_flat_u8_m32k16v2(
        long baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 167-179
```cpp
// 8bits VNNI transform No transpose
SYCL_DEVICE_BUILTIN(
    cute::intel::uint8 __builtin_IB_subgroup_block_read_flat_transform_u8_k32(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint16 __builtin_IB_subgroup_block_read_flat_transform_u8_k32v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint32 __builtin_IB_subgroup_block_read_flat_transform_u8_k32v4(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 181-189
```cpp
// 8bits No transform Transpose
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort4  __builtin_IB_subgroup_block_read_cacheopts_transpose_u8_m32k4(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord, int cacheOpt = 0));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort8  __builtin_IB_subgroup_block_read_cacheopts_transpose_u8_m32k8(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord, int cacheOpt = 0));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 191-206
```cpp
// 8bits No transform No transpose
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_write_flat_u8_m1k16v1(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, cute::intel::uchar data));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_write_flat_u8_m2k16v1(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, cute::intel::uchar2 data));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_write_flat_u8_m4k16v1(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, cute::intel::uchar4));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_write_flat_u8_m8k16v1(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, cute::intel::uchar8));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_write_flat_u8_m8k16v2(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, cute::intel::uchar8));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 208-225
```cpp
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_read_prefetch_u8_m1k16v1(
    long baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_read_prefetch_u8_m1k32v1(
    long baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_read_prefetch_u8_m2k32v1(
    long baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_read_prefetch_u8_m4k32v1(
    long baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_read_prefetch_u8_m8k32v1(
    long baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_read_prefetch_u8_m32k32v1(
    long baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 227-242
```cpp
// // 2D prefetch
SYCL_DEVICE_OCL(void intel_sub_group_2d_block_prefetch_8b_1r32x2c(
    __global void* base_address, int width, int height, int pitch,
    cute::intel::coord_t coord));
SYCL_DEVICE_OCL(void intel_sub_group_2d_block_prefetch_8b_2r32x2c(
    __global void* base_address, int width, int height, int pitch,
    cute::intel::coord_t coord));
SYCL_DEVICE_OCL(void intel_sub_group_2d_block_prefetch_8b_4r32x2c(
    __global void* base_address, int width, int height, int pitch,
    cute::intel::coord_t coord));
SYCL_DEVICE_OCL(void intel_sub_group_2d_block_prefetch_8b_8r32x2c(
    __global void* base_address, int width, int height, int pitch,
    cute::intel::coord_t coord));
SYCL_DEVICE_OCL(void intel_sub_group_2d_block_prefetch_8b_32r16x1c(
    __global void* base_address, int width, int height, int pitch,
    cute::intel::coord_t coord));
```
- **EN:** Defines or forwards `SYCL_DEVICE_OCL` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_OCL`，作为该头文件可执行接口的一部分。

### Lines 244-246
```cpp
SYCL_DEVICE_BUILTIN(cute::intel::ushort16 intel_subgroup_block_read_u16_m8k16v2(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 248-250
```cpp
SYCL_DEVICE_BUILTIN(cute::intel::int8 intel_subgroup_block_read_transform_u16_k16(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 252-270
```cpp
// U16 prefetch
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_read_prefetch_u16_m8k16v1(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_read_prefetch_u16_m8k16v2(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_read_prefetch_u16_m16k16v1(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_read_prefetch_u16_m32k16v1(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_read_prefetch_u16_m16k16v2(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_read_prefetch_u16_m32k16v2(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 272-295
```cpp
// 16 bits No transform No transpose
SYCL_DEVICE_BUILTIN(cute::intel::ushort __builtin_IB_subgroup_block_read_flat_u16_m1k16v1(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort2 __builtin_IB_subgroup_block_read_flat_u16_m2k16v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort4 __builtin_IB_subgroup_block_read_flat_u16_m4k16v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort8 __builtin_IB_subgroup_block_read_flat_u16_m8k16v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort16 __builtin_IB_subgroup_block_read_flat_u16_m16k16v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort32 __builtin_IB_subgroup_block_read_flat_u16_m32k16v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 297-324
```cpp
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort2 __builtin_IB_subgroup_block_read_flat_u16_m1k16v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort4 __builtin_IB_subgroup_block_read_flat_u16_m2k16v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort8 __builtin_IB_subgroup_block_read_flat_u16_m4k16v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort16 __builtin_IB_subgroup_block_read_flat_u16_m8k16v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort32 __builtin_IB_subgroup_block_read_flat_u16_m16k16v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort64 __builtin_IB_subgroup_block_read_flat_u16_m32k16v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort64 __builtin_IB_subgroup_block_read_cacheopts_u16_m32k32v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord, int cacheOpt = 0));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 326-342
```cpp
// 16bits VNNI transform No transpose
SYCL_DEVICE_BUILTIN(
    cute::intel::uint8 __builtin_IB_subgroup_block_read_flat_transform_u16_k16(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint16 __builtin_IB_subgroup_block_read_flat_transform_u16_k32(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint16 __builtin_IB_subgroup_block_read_flat_transform_u16_k16v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint32 __builtin_IB_subgroup_block_read_flat_transform_u16_k32v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 344-356
```cpp
// 16bits
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_write_flat_u16_m1k16v1(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, cute::intel::ushort data));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_write_flat_u16_m2k16v1(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, cute::intel::ushort2 data));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_write_flat_u16_m4k16v1(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, cute::intel::ushort4 data));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_write_flat_u16_m8k16v1(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, cute::intel::ushort8 data));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 358-367
```cpp
// 2D prefetch
SYCL_DEVICE_OCL(void intel_sub_group_2d_block_prefetch_16b_1r16x2c(
    __global void* base_address, int width, int height, int pitch,
    cute::intel::coord_t coord));
SYCL_DEVICE_OCL(void intel_sub_group_2d_block_prefetch_16b_2r16x2c(
    __global void* base_address, int width, int height, int pitch,
    cute::intel::coord_t coord));
SYCL_DEVICE_OCL(void intel_sub_group_2d_block_prefetch_16b_4r16x2c(
    __global void* base_address, int width, int height, int pitch,
    cute::intel::coord_t coord));
```
- **EN:** Defines or forwards `SYCL_DEVICE_OCL` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_OCL`，作为该头文件可执行接口的一部分。

### Lines 370-394
```cpp
// 32bits specific for tf32 No transform No transpose
SYCL_DEVICE_BUILTIN(
    cute::intel::uint __builtin_IB_subgroup_block_read_flat_u32_m1k8v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint __builtin_IB_subgroup_block_read_flat_u32_m2k8v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint2 __builtin_IB_subgroup_block_read_flat_u32_m4k8v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint4 __builtin_IB_subgroup_block_read_flat_u32_m8k8v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint8 __builtin_IB_subgroup_block_read_flat_u32_m16k8v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint16 __builtin_IB_subgroup_block_read_flat_u32_m32k8v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 396-419
```cpp
SYCL_DEVICE_BUILTIN(
    cute::intel::uint2 __builtin_IB_subgroup_block_read_flat_u32_m1k8v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint2 __builtin_IB_subgroup_block_read_flat_u32_m2k8v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint4 __builtin_IB_subgroup_block_read_flat_u32_m4k8v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint8 __builtin_IB_subgroup_block_read_flat_u32_m8k8v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint16 __builtin_IB_subgroup_block_read_flat_u32_m16k8v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint32 __builtin_IB_subgroup_block_read_flat_u32_m32k8v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 421-444
```cpp
// 32bits No transform No transpose
SYCL_DEVICE_BUILTIN(cute::intel::uint __builtin_IB_subgroup_block_read_flat_u32_m1k16v1(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint2 __builtin_IB_subgroup_block_read_flat_u32_m2k16v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint4 __builtin_IB_subgroup_block_read_flat_u32_m4k16v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint8 __builtin_IB_subgroup_block_read_flat_u32_m8k16v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint16 __builtin_IB_subgroup_block_read_flat_u32_m16k16v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint32 __builtin_IB_subgroup_block_read_flat_u32_m32k16v1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 446-465
```cpp
// 32bits No transform Transpose
SYCL_DEVICE_BUILTIN(cute::intel::uint __builtin_IB_subgroup_block_read_flat_transpose_u32_k1(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint2 __builtin_IB_subgroup_block_read_flat_transpose_u32_k2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint4 __builtin_IB_subgroup_block_read_flat_transpose_u32_k4(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::uint8 __builtin_IB_subgroup_block_read_flat_transpose_u32_k8(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ushort8 __builtin_IB_subgroup_block_read_cacheopts_transpose_u32_m8k8(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord, int cacheOpt = 0));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 467-479
```cpp
// 32bits
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_write_flat_u32_m1k16v1(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, cute::intel::uint data));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_write_flat_u32_m2k16v1(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, cute::intel::uint2 data));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_write_flat_u32_m4k16v1(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, cute::intel::uint4 data));
SYCL_DEVICE_BUILTIN(void __builtin_IB_subgroup_block_write_flat_u32_m8k16v1(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, cute::intel::uint8 data));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 481-483
```cpp
SYCL_DEVICE_OCL(void intel_sub_group_2d_block_prefetch_32b_16r8x1c(
    __global void* base_address, int width, int height, int pitch,
    cute::intel::coord_t coord));
```
- **EN:** Defines or forwards `SYCL_DEVICE_OCL` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_OCL`，作为该头文件可执行接口的一部分。

### Lines 485-497
```cpp
// 64bits No transform Transpose
SYCL_DEVICE_BUILTIN(
    cute::intel::ulong __builtin_IB_subgroup_block_read_flat_transpose_u64_k1(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ulong2 __builtin_IB_subgroup_block_read_flat_transpose_u64_k2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
SYCL_DEVICE_BUILTIN(
    cute::intel::ulong4 __builtin_IB_subgroup_block_read_flat_transpose_u64_k4(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord));
```
- **EN:** Defines or forwards `SYCL_DEVICE_BUILTIN` as part of this header's executable interface.
- **CN:** 定义或转发 `SYCL_DEVICE_BUILTIN`，作为该头文件可执行接口的一部分。

### Lines 499-500
```cpp
namespace cute::detail
{
```
- **EN:** Enters or leaves namespace scope `cute::detail` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute::detail`，以便把相关符号组织在一起。

### Lines 502-511
```cpp
template<>
struct XeSubgroup2DBlockLoad<1, 16, 1, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<cute::intel::uchar *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u8_m1k16v1(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 513-522
```cpp
template<>
struct XeSubgroup2DBlockLoad<1, 32, 1, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<ushort *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u8_m1k32v1(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 524-533
```cpp
template<>
struct XeSubgroup2DBlockLoad<1, 16, 1, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<cute::intel::ushort *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u8_m1k16v2(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 535-544
```cpp
template<>
struct XeSubgroup2DBlockLoad<1, 32, 2, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort2 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u8_m2k32v1(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 546-555
```cpp
template<>
struct XeSubgroup2DBlockLoad<1, 32, 4, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort4 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u8_m4k32v1(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 557-566
```cpp
template<>
struct XeSubgroup2DBlockLoad<1, 32, 8, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort8 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u8_m8k32v1(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 568-577
```cpp
template<>
struct XeSubgroup2DBlockLoad<1, 32, 16, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort16 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u8_m16k32v1(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 579-588
```cpp
template<>
struct XeSubgroup2DBlockLoad<1, 32, 32, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort32 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u8_m32k32v1(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 590-599
```cpp
template<>
struct XeSubgroup2DBlockLoad<1, 16, 32, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uchar64 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u8_m32k16v2(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 601-610
```cpp
template<>
struct XeSubgroup2DBlockLoad<1, 32, 1, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort2 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u8_m1k32v2(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 612-621
```cpp
template<>
struct XeSubgroup2DBlockLoad<1, 32, 2, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort4 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u8_m2k32v2(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 623-632
```cpp
template<>
struct XeSubgroup2DBlockLoad<1, 32, 4, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort8 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u8_m4k32v2(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 634-643
```cpp
template<>
struct XeSubgroup2DBlockLoad<1, 32, 8, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort16 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u8_m8k32v2(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 646-655
```cpp
template<>
struct XeSubgroup2DBlockLoad<1, 16, 8, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uchar16 *>(dstPointer) =  __builtin_IB_subgroup_block_read_cacheopts_u8_m8k16v2(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate,  CacheControl::kL1C_L3C);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 657-666
```cpp
template<>
struct XeSubgroup2DBlockLoad<1, 32, 16, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort32 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u8_m16k32v2(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 668-677
```cpp
template<>
struct XeSubgroup2DBlockLoad<1, 32, 32, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort64 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u8_m32k32v2(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 679-688
```cpp
template<>
struct XeSubgroup2DBlockLoadTransform<1, 16, 32, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint8 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_transform_u8_k32(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTransform` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTransform`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 690-699
```cpp
template<>
struct XeSubgroup2DBlockLoadTransform<1, 16, 32, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint16 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_transform_u8_k32v2(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTransform` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTransform`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 701-710
```cpp
template<>
struct XeSubgroup2DBlockLoadTransform<1, 16, 32, 4> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint32 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_transform_u8_k32v4(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTransform` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTransform`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 712-721
```cpp
template<>
struct XeSubgroup2DBlockLoadTranspose<1, 4, 32, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort4 *>(dstPointer) =  __builtin_IB_subgroup_block_read_cacheopts_transpose_u8_m32k4(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTranspose` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTranspose`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 723-732
```cpp
template<>
struct XeSubgroup2DBlockLoadTranspose<1, 8, 32, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort8 *>(dstPointer) =  __builtin_IB_subgroup_block_read_cacheopts_transpose_u8_m32k8(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTranspose` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTranspose`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 734-743
```cpp
template<>
struct XeSubgroup2DBlockStore<1, 16, 1, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* dstBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* srcPointer) {
        __builtin_IB_subgroup_block_write_flat_u8_m1k16v1(
           (intptr_t)(dstBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, *(intel::uchar *)(srcPointer));
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockStore` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockStore`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 745-754
```cpp
template<>
struct XeSubgroup2DBlockStore<1, 16, 2, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* dstBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* srcPointer) {
        __builtin_IB_subgroup_block_write_flat_u8_m2k16v1(
           (intptr_t)(dstBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, *(intel::uchar2 *)(srcPointer));
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockStore` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockStore`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 756-765
```cpp
template<>
struct XeSubgroup2DBlockStore<1, 16, 4, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* dstBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* srcPointer) {
        __builtin_IB_subgroup_block_write_flat_u8_m4k16v1(
           (intptr_t)(dstBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, *(intel::uchar4 *)(srcPointer));
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockStore` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockStore`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 767-776
```cpp
template<>
struct XeSubgroup2DBlockStore<1, 16, 8, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* dstBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* srcPointer) {
        __builtin_IB_subgroup_block_write_flat_u8_m8k16v1(
           (intptr_t)(dstBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, *(intel::uchar8 *)(srcPointer));
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockStore` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockStore`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 778-787
```cpp
template<>
struct XeSubgroup2DBlockStore<1, 16, 8, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* dstBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* srcPointer) {
        __builtin_IB_subgroup_block_write_flat_u8_m8k16v2(
           (intptr_t)(dstBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, *(intel::uchar8 *)(srcPointer));
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockStore` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockStore`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 789-797
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 16, 1, 1> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        __builtin_IB_subgroup_block_read_prefetch_u8_m1k16v1(
            (intptr_t)srcBasePointer, memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, CacheControl::kL1C_L3C);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 799-807
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 32, 1, 1> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        __builtin_IB_subgroup_block_read_prefetch_u8_m1k32v1(
            (intptr_t)srcBasePointer, memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, CacheControl::kL1C_L3C);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 809-826
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 32, 2, 1> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        __builtin_IB_subgroup_block_read_prefetch_u8_m2k32v1(
            (intptr_t)srcBasePointer, memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, CacheControl::kL1C_L3C);
    }
};
template<>
struct XeSubgroup2DBlockPrefetch<1, 32, 4, 1> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        __builtin_IB_subgroup_block_read_prefetch_u8_m4k32v1(
            (intptr_t)srcBasePointer, memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, CacheControl::kL1C_L3C);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 828-836
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 32, 8, 1> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        __builtin_IB_subgroup_block_read_prefetch_u8_m8k32v1(
            (intptr_t)srcBasePointer, memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, CacheControl::kL1C_L3C);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 838-846
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 32, 32, 1> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        __builtin_IB_subgroup_block_read_prefetch_u8_m32k32v1(
            (intptr_t)srcBasePointer, memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, CacheControl::kL1C_L3C);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 848-856
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 32, 1, 2> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        intel_sub_group_2d_block_prefetch_8b_1r32x2c(
            (__global void*)(srcBasePointer), memoryWidth, memoryHeight, memoryPitch, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 858-866
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 32, 2, 2> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        intel_sub_group_2d_block_prefetch_8b_2r32x2c(
            (__global void*)(srcBasePointer), memoryWidth, memoryHeight, memoryPitch, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 868-876
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 32, 4, 2> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        intel_sub_group_2d_block_prefetch_8b_4r32x2c(
            (__global void*)(srcBasePointer), memoryWidth, memoryHeight, memoryPitch, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 878-886
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 32, 8, 2> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        intel_sub_group_2d_block_prefetch_8b_8r32x2c(
            (__global void*)(srcBasePointer), memoryWidth, memoryHeight, memoryPitch, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 888-896
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 16, 32, 1> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        intel_sub_group_2d_block_prefetch_8b_32r16x1c(
            (__global void*)(srcBasePointer), memoryWidth, memoryHeight, memoryPitch, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 898-907
```cpp
template<>
struct XeSubgroup2DBlockLoad<2, 16, 1, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<ushort *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u16_m1k16v1(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 909-918
```cpp
template<>
struct XeSubgroup2DBlockLoad<2, 16, 2, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort2 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u16_m2k16v1(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 921-930
```cpp
template<>
struct XeSubgroup2DBlockLoad<2, 16, 4, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort4 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u16_m4k16v1(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 932-941
```cpp
template<>
struct XeSubgroup2DBlockLoad<2, 16, 8, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort8 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u16_m8k16v1(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 943-952
```cpp
template<>
struct XeSubgroup2DBlockLoad<2, 16, 16, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort16 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u16_m16k16v1(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 954-963
```cpp
template<>
struct XeSubgroup2DBlockLoad<2, 16, 32, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort32 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u16_m32k16v1(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 965-974
```cpp
template<>
struct XeSubgroup2DBlockLoad<2, 16, 1, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort2 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u16_m1k16v2(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 976-985
```cpp
template<>
struct XeSubgroup2DBlockLoad<2, 16, 2, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort4 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u16_m2k16v2(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 987-996
```cpp
template<>
struct XeSubgroup2DBlockLoad<2, 16, 4, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort8 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u16_m4k16v2(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 998-1007
```cpp
template<>
struct XeSubgroup2DBlockLoad<2, 16, 8, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort16 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u16_m8k16v2(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1009-1018
```cpp
template<>
struct XeSubgroup2DBlockLoad<2, 16, 16, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort32 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u16_m16k16v2(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1020-1029
```cpp
template<>
struct XeSubgroup2DBlockLoad<2, 16, 32, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort64 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u16_m32k16v2(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1031-1040
```cpp
template<>
struct XeSubgroup2DBlockLoad<2, 32, 32, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort64 *>(dstPointer) =  __builtin_IB_subgroup_block_read_cacheopts_u16_m32k32v1(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1042-1051
```cpp
template<>
struct XeSubgroup2DBlockLoadTransform<2, 16, 16, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint8 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_transform_u16_k16(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTransform` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTransform`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1053-1062
```cpp
template<>
struct XeSubgroup2DBlockLoadTransform<2, 16, 32, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint16 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_transform_u16_k32(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTransform` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTransform`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1064-1073
```cpp
template<>
struct XeSubgroup2DBlockLoadTransform<2, 16, 16, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint16 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_transform_u16_k16v2(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTransform` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTransform`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1075-1084
```cpp
template<>
struct XeSubgroup2DBlockLoadTransform<2, 16, 32, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint32 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_transform_u16_k32v2(
           (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTransform` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTransform`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1086-1095
```cpp
template<>
struct XeSubgroup2DBlockStore<2, 16, 1, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* dstBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* srcPointer) {
        __builtin_IB_subgroup_block_write_flat_u16_m1k16v1(
           (intptr_t)(dstBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, *(ushort *)(srcPointer));
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockStore` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockStore`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1097-1106
```cpp
template<>
struct XeSubgroup2DBlockStore<2, 16, 2, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* dstBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* srcPointer) {
        __builtin_IB_subgroup_block_write_flat_u16_m2k16v1(
           (intptr_t)(dstBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, *(intel::ushort2 *)(srcPointer));
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockStore` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockStore`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1108-1117
```cpp
template<>
struct XeSubgroup2DBlockStore<2, 16, 4, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* dstBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* srcPointer) {
        __builtin_IB_subgroup_block_write_flat_u16_m4k16v1(
           (intptr_t)(dstBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, *(intel::ushort4 *)(srcPointer));
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockStore` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockStore`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1119-1128
```cpp
template<>
struct XeSubgroup2DBlockStore<2, 16, 8, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* dstBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* srcPointer) {
        __builtin_IB_subgroup_block_write_flat_u16_m8k16v1(
           (intptr_t)(dstBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, *(intel::ushort8 *)(srcPointer));
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockStore` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockStore`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1130-1138
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<2, 16, 8, 1> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        __builtin_IB_subgroup_block_read_prefetch_u16_m8k16v1(
            (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, CacheControl::kL1C_L3C);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1140-1148
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<2, 16, 16, 1> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        __builtin_IB_subgroup_block_read_prefetch_u16_m16k16v1(
            (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, CacheControl::kL1C_L3C);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1150-1158
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<2, 16, 32, 1> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        __builtin_IB_subgroup_block_read_prefetch_u16_m32k16v1(
            (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, CacheControl::kL1C_L3C);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1160-1168
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<2, 16, 1, 2> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        intel_sub_group_2d_block_prefetch_16b_1r16x2c(
            (__global void*)(srcBasePointer), memoryWidth, memoryHeight, memoryPitch, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1170-1178
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<2, 16, 2, 2> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        intel_sub_group_2d_block_prefetch_16b_2r16x2c(
            (__global void*)(srcBasePointer), memoryWidth, memoryHeight, memoryPitch, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1180-1188
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<2, 16, 4, 2> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        intel_sub_group_2d_block_prefetch_16b_4r16x2c(
            (__global void*)(srcBasePointer), memoryWidth, memoryHeight, memoryPitch, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1190-1198
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<2, 16, 8, 2> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        __builtin_IB_subgroup_block_read_prefetch_u16_m8k16v2(
            (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, CacheControl::kL1C_L3C);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1200-1208
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<2, 16, 16, 2> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        __builtin_IB_subgroup_block_read_prefetch_u16_m16k16v2(
            (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, CacheControl::kL1C_L3C);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1210-1218
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<2, 16, 32, 2> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        __builtin_IB_subgroup_block_read_prefetch_u16_m32k16v2(
            (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, CacheControl::kL1C_L3C);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1220-1229
```cpp
template<>
struct XeSubgroup2DBlockLoad<4, 16, 1, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<uint *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u32_m1k16v1(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1231-1240
```cpp
template<>
struct XeSubgroup2DBlockLoad<4, 16, 2, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint2 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u32_m2k16v1(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1242-1251
```cpp
template<>
struct XeSubgroup2DBlockLoad<4, 16, 4, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint4 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u32_m4k16v1(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1253-1262
```cpp
template<>
struct XeSubgroup2DBlockLoad<4, 16, 8, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint8 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u32_m8k16v1(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1264-1273
```cpp
template<>
struct XeSubgroup2DBlockLoad<4, 16, 16, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint16 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u32_m16k16v1(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1275-1284
```cpp
template<>
struct XeSubgroup2DBlockLoad<4, 16, 32, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint32 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u32_m32k16v1(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1286-1295
```cpp
template<>
struct XeSubgroup2DBlockLoad<4, 8, 1, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<uint *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u32_m1k8v1(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1297-1306
```cpp
template<>
struct XeSubgroup2DBlockLoad<4, 8, 2, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<uint *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u32_m2k8v1(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1308-1317
```cpp
template<>
struct XeSubgroup2DBlockLoad<4, 8, 4, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint2 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u32_m4k8v1(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1319-1328
```cpp
template<>
struct XeSubgroup2DBlockLoad<4, 8, 8, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint4 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u32_m8k8v1(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1330-1339
```cpp
template<>
struct XeSubgroup2DBlockLoad<4, 8, 16, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint8 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u32_m16k8v1(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1341-1350
```cpp
template<>
struct XeSubgroup2DBlockLoad<4, 8, 32, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint16 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u32_m32k8v1(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1352-1361
```cpp
template<>
struct XeSubgroup2DBlockLoad<4, 8, 1, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint2 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u32_m1k8v2(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1363-1372
```cpp
template<>
struct XeSubgroup2DBlockLoad<4, 8, 2, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint2 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u32_m2k8v2(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1374-1383
```cpp
template<>
struct XeSubgroup2DBlockLoad<4, 8, 4, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint4 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u32_m4k8v2(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1385-1394
```cpp
template<>
struct XeSubgroup2DBlockLoad<4, 8, 8, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint8 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u32_m8k8v2(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1396-1405
```cpp
template<>
struct XeSubgroup2DBlockLoad<4, 8, 16, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint16 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u32_m16k8v2(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1407-1416
```cpp
template<>
struct XeSubgroup2DBlockLoad<4, 8, 32, 2> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint32 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u32_m32k8v2(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1418-1427
```cpp
template<>
struct XeSubgroup2DBlockLoadTranspose<4, 1, 16, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<uint *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_transpose_u32_k1(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTranspose` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTranspose`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1429-1438
```cpp
template<>
struct XeSubgroup2DBlockLoadTranspose<4, 2, 16, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint2 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_transpose_u32_k2(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTranspose` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTranspose`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1440-1449
```cpp
template<>
struct XeSubgroup2DBlockLoadTranspose<4, 4, 16, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint4 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_transpose_u32_k4(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTranspose` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTranspose`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1451-1460
```cpp
template<>
struct XeSubgroup2DBlockLoadTranspose<4, 8, 16, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::uint8 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_transpose_u32_k8(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTranspose` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTranspose`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1462-1471
```cpp
template<>
struct XeSubgroup2DBlockLoadTranspose<4, 8, 8, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ushort8 *>(dstPointer) =  __builtin_IB_subgroup_block_read_cacheopts_transpose_u32_m8k8(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTranspose` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTranspose`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1473-1482
```cpp
template<>
struct XeSubgroup2DBlockStore<4, 16, 1, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* dstBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* srcPointer) {
        __builtin_IB_subgroup_block_write_flat_u32_m1k16v1(
           reinterpret_cast<long>(dstBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, *(uint *)(srcPointer));
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockStore` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockStore`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1484-1493
```cpp
template<>
struct XeSubgroup2DBlockStore<4, 16, 2, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* dstBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* srcPointer) {
        __builtin_IB_subgroup_block_write_flat_u32_m2k16v1(
           reinterpret_cast<long>(dstBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, *(intel::uint2 *)(srcPointer));
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockStore` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockStore`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1495-1504
```cpp
template<>
struct XeSubgroup2DBlockStore<4, 16, 4, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* dstBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* srcPointer) {
        __builtin_IB_subgroup_block_write_flat_u32_m4k16v1(
           reinterpret_cast<long>(dstBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, *(intel::uint4 *)(srcPointer));
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockStore` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockStore`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1506-1515
```cpp
template<>
struct XeSubgroup2DBlockStore<4, 16, 8, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* dstBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* srcPointer) {
        __builtin_IB_subgroup_block_write_flat_u32_m8k16v1(
           reinterpret_cast<long>(dstBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate, *(intel::uint8 *)(srcPointer));
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockStore` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockStore`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1517-1525
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<4, 8, 16, 1> {
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate) {
        intel_sub_group_2d_block_prefetch_32b_16r8x1c(
            (__global void*)(srcBasePointer), memoryWidth, memoryHeight, memoryPitch, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1527-1536
```cpp
template<>
struct XeSubgroup2DBlockLoadTranspose<8, 1, 8, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ulong *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_transpose_u64_k1(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTranspose` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTranspose`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1538-1547
```cpp
template<>
struct XeSubgroup2DBlockLoadTranspose<8, 2, 8, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ulong2 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_transpose_u64_k2(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTranspose` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTranspose`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1549-1558
```cpp
template<>
struct XeSubgroup2DBlockLoadTranspose<8, 4, 8, 1> {
    template<typename T>
    CUTE_HOST_DEVICE void 
    operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
            cute::intel::coord_t coordinate, T* dstPointer) {
        *reinterpret_cast<intel::ulong4 *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_transpose_u64_k4(
           reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
    }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTranspose` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTranspose`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1560-1560
```cpp
} // namespace cute::detail
```
- **EN:** Enters or leaves namespace scope `cute::detail` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute::detail`，以便把相关符号组织在一起。

## Key Concepts / 关键概念

- **EN:** Copy traits/policies separate abstract data movement from the concrete instruction selected underneath.
  **CN:** Copy trait/策略把抽象数据搬运与底层实际选用的指令解耦。
- **EN:** Compile-time assertions encode hardware and type constraints directly in the API surface.
  **CN:** 编译期断言把硬件与类型约束直接编码到 API 表面。
- **EN:** Prefetch paths try to reduce latency by staging data or metadata early.
  **CN:** 预取路径尝试通过提前准备数据或元数据来降低延迟。
- **EN:** The Xe/SYCL path bridges CuTe abstractions to Intel GPU builtins or SPIR-V operations.
  **CN:** Xe/SYCL 路径把 CuTe 抽象桥接到 Intel GPU 内建函数或 SPIR-V 操作。

## Dependencies / 依赖关系

- **EN:** `cute/util/sycl_vec.hpp` supplies related definitions from `cute/util/sycl_vec.hpp`.
  **CN:** `cute/util/sycl_vec.hpp` 提供了来自 `cute/util/sycl_vec.hpp` 的相关定义。
- **EN:** `cute/config.hpp` supplies core CuTe configuration macros and portability annotations.
  **CN:** `cute/config.hpp` 提供了CuTe 核心配置宏与可移植性标注。
- **EN:** SYCL/SPIR-V feature macros select alternate code paths for Intel/Xe-style backends.
  **CN:** SYCL/SPIR-V 特性宏会为 Intel/Xe 风格后端选择替代代码路径。
