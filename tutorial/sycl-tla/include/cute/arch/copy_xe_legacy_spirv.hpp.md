# copy_xe_legacy_spirv.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/copy_xe_legacy_spirv.hpp`
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

### Lines 36-40
```cpp
// TODO(Codeplay): These builtins are not available on SPIRV
SYCL_EXTERNAL extern "C"
cute::intel::uchar __builtin_IB_subgroup_block_read_flat_u8_m1k16v1(
  long baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_flat_u8_m1k16v1` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_flat_u8_m1k16v1`，作为该头文件可执行接口的一部分。

### Lines 42-45
```cpp
SYCL_EXTERNAL extern "C"
cute::intel::uchar64 __builtin_IB_subgroup_block_read_flat_u8_m32k16v2(
  long baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_flat_u8_m32k16v2` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_flat_u8_m32k16v2`，作为该头文件可执行接口的一部分。

### Lines 47-50
```cpp
SYCL_EXTERNAL extern "C"
cute::intel::ushort64 __builtin_IB_subgroup_block_read_cacheopts_u16_m32k32v1(
  long baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, int cacheOpt = 0);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_cacheopts_u16_m32k32v1` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_cacheopts_u16_m32k32v1`，作为该头文件可执行接口的一部分。

### Lines 52-55
```cpp
SYCL_EXTERNAL extern "C"
cute::intel::uint2 __builtin_IB_subgroup_block_read_flat_transpose_u32_k2(
  intptr_t baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_flat_transpose_u32_k2` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_flat_transpose_u32_k2`，作为该头文件可执行接口的一部分。

### Lines 57-60
```cpp
SYCL_EXTERNAL extern "C"
cute::intel::uint4 __builtin_IB_subgroup_block_read_flat_transpose_u32_k4(
  intptr_t baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_flat_transpose_u32_k4` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_flat_transpose_u32_k4`，作为该头文件可执行接口的一部分。

### Lines 62-65
```cpp
SYCL_EXTERNAL extern "C"
cute::intel::ushort4  __builtin_IB_subgroup_block_read_cacheopts_transpose_u8_m32k4(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, int cacheOpt = 0);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_cacheopts_transpose_u8_m32k4` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_cacheopts_transpose_u8_m32k4`，作为该头文件可执行接口的一部分。

### Lines 67-70
```cpp
SYCL_EXTERNAL extern "C"
cute::intel::ushort8  __builtin_IB_subgroup_block_read_cacheopts_transpose_u8_m32k8(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, int cacheOpt = 0);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_cacheopts_transpose_u8_m32k8` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_cacheopts_transpose_u8_m32k8`，作为该头文件可执行接口的一部分。

### Lines 72-75
```cpp
SYCL_EXTERNAL extern "C"
    cute::intel::ushort __builtin_IB_subgroup_block_read_flat_u8_m1k16v2(
        intptr_t baseoffset, int width_minus_one, int height_minus_one,
        int pitch_minus_one, cute::intel::coord_t coord);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_flat_u8_m1k16v2` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_flat_u8_m1k16v2`，作为该头文件可执行接口的一部分。

### Lines 77-86
```cpp
enum class CacheControl {
  kDefault = 0,
  kL1UC_L3UC = 1, // Override to L1 uncached and L3 uncached
  kL1UC_L3C = 2, // Override to L1 uncached and L3 cached
  kL1C_L3UC = 3, // Override to L1 cached and L3 uncached
  kL1C_L3C = 4, // Override to L1 cached and L3 cached
  kL1S_L3UC = 5, // Override to L1 streaming load and L3 uncached
  kL1S_L3C = 6, // Override to L1 streaming load and L3 cached
  kL1IAR_L3C = 7, // Override to L1 invalidate-after-read, and L3 cached
};
```
- **EN:** Defines `CacheControl` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `CacheControl`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 88-91
```cpp
SYCL_EXTERNAL extern "C"
cute::intel::uchar16 __builtin_IB_subgroup_block_read_cacheopts_u8_m8k16v2(
    intptr_t baseoffset, int width_minus_one, int height_minus_one,
    int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control );
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_cacheopts_u8_m8k16v2` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_cacheopts_u8_m8k16v2`，作为该头文件可执行接口的一部分。

### Lines 93-97
```cpp
// U16 prefetch
SYCL_EXTERNAL extern "C"
void __builtin_IB_subgroup_block_read_prefetch_u16_m8k16v1(
  intptr_t baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_prefetch_u16_m8k16v1` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_prefetch_u16_m8k16v1`，作为该头文件可执行接口的一部分。

### Lines 99-102
```cpp
SYCL_EXTERNAL extern "C"
void __builtin_IB_subgroup_block_read_prefetch_u16_m8k16v2(
  intptr_t baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_prefetch_u16_m8k16v2` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_prefetch_u16_m8k16v2`，作为该头文件可执行接口的一部分。

### Lines 104-107
```cpp
SYCL_EXTERNAL extern "C"
void __builtin_IB_subgroup_block_read_prefetch_u16_m16k16v1(
  intptr_t baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_prefetch_u16_m16k16v1` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_prefetch_u16_m16k16v1`，作为该头文件可执行接口的一部分。

### Lines 109-112
```cpp
SYCL_EXTERNAL extern "C"
void __builtin_IB_subgroup_block_read_prefetch_u16_m32k16v1(
  intptr_t baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_prefetch_u16_m32k16v1` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_prefetch_u16_m32k16v1`，作为该头文件可执行接口的一部分。

### Lines 114-117
```cpp
SYCL_EXTERNAL extern "C"
void __builtin_IB_subgroup_block_read_prefetch_u16_m16k16v2(
  intptr_t baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_prefetch_u16_m16k16v2` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_prefetch_u16_m16k16v2`，作为该头文件可执行接口的一部分。

### Lines 119-122
```cpp
SYCL_EXTERNAL extern "C"
void __builtin_IB_subgroup_block_read_prefetch_u16_m32k16v2(
  intptr_t baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_prefetch_u16_m32k16v2` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_prefetch_u16_m32k16v2`，作为该头文件可执行接口的一部分。

### Lines 124-127
```cpp
SYCL_EXTERNAL extern "C"
void __builtin_IB_subgroup_block_read_prefetch_u8_m1k16v1(
  long baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_prefetch_u8_m1k16v1` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_prefetch_u8_m1k16v1`，作为该头文件可执行接口的一部分。

### Lines 129-132
```cpp
SYCL_EXTERNAL extern "C"
void __builtin_IB_subgroup_block_read_prefetch_u8_m1k32v1(
  long baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_prefetch_u8_m1k32v1` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_prefetch_u8_m1k32v1`，作为该头文件可执行接口的一部分。

### Lines 134-137
```cpp
SYCL_EXTERNAL extern "C"
void __builtin_IB_subgroup_block_read_prefetch_u8_m2k32v1(
  long baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_prefetch_u8_m2k32v1` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_prefetch_u8_m2k32v1`，作为该头文件可执行接口的一部分。

### Lines 139-142
```cpp
SYCL_EXTERNAL extern "C"
void __builtin_IB_subgroup_block_read_prefetch_u8_m4k32v1(
  long baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_prefetch_u8_m4k32v1` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_prefetch_u8_m4k32v1`，作为该头文件可执行接口的一部分。

### Lines 144-147
```cpp
SYCL_EXTERNAL extern "C"
void __builtin_IB_subgroup_block_read_prefetch_u8_m8k32v1(
  long baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_prefetch_u8_m8k32v1` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_prefetch_u8_m8k32v1`，作为该头文件可执行接口的一部分。

### Lines 149-152
```cpp
SYCL_EXTERNAL extern "C"
void __builtin_IB_subgroup_block_read_prefetch_u8_m32k32v1(
  long baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_prefetch_u8_m32k32v1` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_prefetch_u8_m32k32v1`，作为该头文件可执行接口的一部分。

### Lines 154-157
```cpp
SYCL_EXTERNAL
void intel_sub_group_2d_block_prefetch_8b_1r32x2c(
  __attribute__((opencl_global)) void *base_address, int width, int height, int pitch,
  cute::intel::coord_t coord);
```
- **EN:** Defines or forwards `intel_sub_group_2d_block_prefetch_8b_1r32x2c` as part of this header's executable interface.
- **CN:** 定义或转发 `intel_sub_group_2d_block_prefetch_8b_1r32x2c`，作为该头文件可执行接口的一部分。

### Lines 159-162
```cpp
SYCL_EXTERNAL
void intel_sub_group_2d_block_prefetch_8b_2r32x2c(
  __attribute__((opencl_global)) void *base_address, int width, int height, int pitch,
  cute::intel::coord_t coord);
```
- **EN:** Defines or forwards `intel_sub_group_2d_block_prefetch_8b_2r32x2c` as part of this header's executable interface.
- **CN:** 定义或转发 `intel_sub_group_2d_block_prefetch_8b_2r32x2c`，作为该头文件可执行接口的一部分。

### Lines 164-167
```cpp
SYCL_EXTERNAL
void intel_sub_group_2d_block_prefetch_8b_4r32x2c(
  __attribute__((opencl_global)) void *base_address, int width, int height, int pitch,
  cute::intel::coord_t coord);
```
- **EN:** Defines or forwards `intel_sub_group_2d_block_prefetch_8b_4r32x2c` as part of this header's executable interface.
- **CN:** 定义或转发 `intel_sub_group_2d_block_prefetch_8b_4r32x2c`，作为该头文件可执行接口的一部分。

### Lines 169-172
```cpp
SYCL_EXTERNAL
void intel_sub_group_2d_block_prefetch_8b_8r32x2c(
  __attribute__((opencl_global)) void *base_address, int width, int height, int pitch,
  cute::intel::coord_t coord);
```
- **EN:** Defines or forwards `intel_sub_group_2d_block_prefetch_8b_8r32x2c` as part of this header's executable interface.
- **CN:** 定义或转发 `intel_sub_group_2d_block_prefetch_8b_8r32x2c`，作为该头文件可执行接口的一部分。

### Lines 174-177
```cpp
SYCL_EXTERNAL
void intel_sub_group_2d_block_prefetch_8b_32r16x1c(
  __attribute__((opencl_global)) void *base_address, int width, int height, int pitch,
  cute::intel::coord_t coord);
```
- **EN:** Defines or forwards `intel_sub_group_2d_block_prefetch_8b_32r16x1c` as part of this header's executable interface.
- **CN:** 定义或转发 `intel_sub_group_2d_block_prefetch_8b_32r16x1c`，作为该头文件可执行接口的一部分。

### Lines 179-182
```cpp
SYCL_EXTERNAL
void __builtin_IB_subgroup_block_read_prefetch_u16_m8k16v1(
  intptr_t baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_prefetch_u16_m8k16v1` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_prefetch_u16_m8k16v1`，作为该头文件可执行接口的一部分。

### Lines 184-187
```cpp
SYCL_EXTERNAL
void __builtin_IB_subgroup_block_read_prefetch_u16_m8k16v2(
  intptr_t baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_prefetch_u16_m8k16v2` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_prefetch_u16_m8k16v2`，作为该头文件可执行接口的一部分。

### Lines 189-192
```cpp
SYCL_EXTERNAL extern "C"
void __builtin_IB_subgroup_block_read_prefetch_u16_m16k16v1(
  intptr_t baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_prefetch_u16_m16k16v1` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_prefetch_u16_m16k16v1`，作为该头文件可执行接口的一部分。

### Lines 194-197
```cpp
SYCL_EXTERNAL extern "C"
void __builtin_IB_subgroup_block_read_prefetch_u16_m32k16v1(
  intptr_t baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_prefetch_u16_m32k16v1` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_prefetch_u16_m32k16v1`，作为该头文件可执行接口的一部分。

### Lines 199-202
```cpp
SYCL_EXTERNAL extern "C"
void __builtin_IB_subgroup_block_read_prefetch_u16_m16k16v2(
  intptr_t baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_prefetch_u16_m16k16v2` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_prefetch_u16_m16k16v2`，作为该头文件可执行接口的一部分。

### Lines 204-207
```cpp
SYCL_EXTERNAL extern "C"
void __builtin_IB_subgroup_block_read_prefetch_u16_m32k16v2(
  intptr_t baseoffset, int width_minus_one, int height_minus_one,
  int pitch_minus_one, cute::intel::coord_t coord, enum CacheControl cache_control);
```
- **EN:** Defines or forwards `__builtin_IB_subgroup_block_read_prefetch_u16_m32k16v2` as part of this header's executable interface.
- **CN:** 定义或转发 `__builtin_IB_subgroup_block_read_prefetch_u16_m32k16v2`，作为该头文件可执行接口的一部分。

### Lines 209-212
```cpp
SYCL_EXTERNAL
void intel_sub_group_2d_block_prefetch_16b_1r16x2c(
  __attribute__((opencl_global)) void *base_address, int width, int height, int pitch,
  cute::intel::coord_t coord);
```
- **EN:** Defines or forwards `intel_sub_group_2d_block_prefetch_16b_1r16x2c` as part of this header's executable interface.
- **CN:** 定义或转发 `intel_sub_group_2d_block_prefetch_16b_1r16x2c`，作为该头文件可执行接口的一部分。

### Lines 214-217
```cpp
SYCL_EXTERNAL
void intel_sub_group_2d_block_prefetch_16b_2r16x2c(
  __attribute__((opencl_global)) void *base_address, int width, int height, int pitch,
  cute::intel::coord_t coord);
```
- **EN:** Defines or forwards `intel_sub_group_2d_block_prefetch_16b_2r16x2c` as part of this header's executable interface.
- **CN:** 定义或转发 `intel_sub_group_2d_block_prefetch_16b_2r16x2c`，作为该头文件可执行接口的一部分。

### Lines 219-222
```cpp
SYCL_EXTERNAL
void intel_sub_group_2d_block_prefetch_16b_4r16x2c(
  __attribute__((opencl_global)) void *base_address, int width, int height, int pitch,
  cute::intel::coord_t coord);
```
- **EN:** Defines or forwards `intel_sub_group_2d_block_prefetch_16b_4r16x2c` as part of this header's executable interface.
- **CN:** 定义或转发 `intel_sub_group_2d_block_prefetch_16b_4r16x2c`，作为该头文件可执行接口的一部分。

### Lines 224-227
```cpp
SYCL_EXTERNAL
void intel_sub_group_2d_block_prefetch_32b_16r8x1c(
  __attribute__((opencl_global)) void *base_address, int width, int height, int pitch,
  cute::intel::coord_t coord);
```
- **EN:** Defines or forwards `intel_sub_group_2d_block_prefetch_32b_16r8x1c` as part of this header's executable interface.
- **CN:** 定义或转发 `intel_sub_group_2d_block_prefetch_32b_16r8x1c`，作为该头文件可执行接口的一部分。

### Lines 229-233
```cpp
// SPIRV copy definitions
SYCL_EXTERNAL __attribute__((convergent)) void __spirv_Subgroup2DBlockLoadINTEL(
  int ElementSize, int BlockWidth, int BlockHeight, int BlockCount,
  const void *src_base_pointer, int memory_width, int memory_height,
  int memory_pitch, cute::intel::coord_t coordinate, void *dst_pointer);
```
- **EN:** Defines or forwards `__attribute__` as part of this header's executable interface.
- **CN:** 定义或转发 `__attribute__`，作为该头文件可执行接口的一部分。

### Lines 235-238
```cpp
SYCL_EXTERNAL __attribute__((convergent)) void __spirv_Subgroup2DBlockLoadTransformINTEL(
  int ElementSize, int BlockWidth, int BlockHeight, int BlockCount,
  const void *src_base_pointer, int memory_width, int memory_height,
  int memory_pitch, cute::intel::coord_t coordinate, void *dst_pointer);
```
- **EN:** Defines or forwards `__attribute__` as part of this header's executable interface.
- **CN:** 定义或转发 `__attribute__`，作为该头文件可执行接口的一部分。

### Lines 240-243
```cpp
SYCL_EXTERNAL __attribute__((convergent)) void __spirv_Subgroup2DBlockLoadTransposeINTEL(
  int ElementSize, int BlockWidth, int BlockHeight, int BlockCount,
  const void *src_base_pointer, int memory_width, int memory_height,
  int memory_pitch, cute::intel::coord_t coordinate, void *dst_pointer);
```
- **EN:** Defines or forwards `__attribute__` as part of this header's executable interface.
- **CN:** 定义或转发 `__attribute__`，作为该头文件可执行接口的一部分。

### Lines 245-248
```cpp
SYCL_EXTERNAL __attribute__((convergent)) void __spirv_Subgroup2DBlockStoreINTEL(
  int ElementSize, int BlockWidth, int BlockHeight, int BlockCount,
  void *src_pointer, const void *dst_base_pointer, int memory_width,
  int memory_height, int memory_pitch, cute::intel::coord_t coordinate);
```
- **EN:** Defines or forwards `__attribute__` as part of this header's executable interface.
- **CN:** 定义或转发 `__attribute__`，作为该头文件可执行接口的一部分。

### Lines 250-253
```cpp
SYCL_EXTERNAL __attribute__((convergent)) void __spirv_Subgroup2DBlockPrefetchINTEL(
  int ElementSize, int BlockWidth, int BlockHeight, int BlockCount,
  const void *src_base_pointer, int memory_width, int memory_height,
  int memory_pitch, cute::intel::coord_t coordinate);
```
- **EN:** Defines or forwards `__attribute__` as part of this header's executable interface.
- **CN:** 定义或转发 `__attribute__`，作为该头文件可执行接口的一部分。

### Lines 255-267
```cpp
namespace cute::detail {
template<int ElementSize, int BlockWidth, int BlockHeight, int BlockCount>
struct XeSubgroup2DBlockLoad {
  template<typename T>
  CUTE_HOST_DEVICE
  void operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
                  cute::intel::coord_t coordinate, T *dstPointer) {
    __spirv_Subgroup2DBlockLoadINTEL(ElementSize, BlockWidth, BlockHeight, BlockCount,
                                     srcBasePointer, memoryWidth,
                                     memoryHeight, memoryPitch, coordinate,
                                     static_cast<void *>(dstPointer));
  }
};
```
- **EN:** Enters or leaves namespace scope `cute::detail` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute::detail`，以便把相关符号组织在一起。

### Lines 269-280
```cpp
template<int ElementSize, int BlockWidth, int BlockHeight, int BlockCount>
struct XeSubgroup2DBlockLoadTransform {
  template<typename T>
  CUTE_HOST_DEVICE
  void operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
                  cute::intel::coord_t coordinate, T *dstPointer) {
    __spirv_Subgroup2DBlockLoadTransformINTEL(ElementSize, BlockWidth, BlockHeight, BlockCount,
                                              srcBasePointer, memoryWidth,
                                              memoryHeight, memoryPitch, coordinate,
                                              static_cast<void *>(dstPointer));
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTransform` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTransform`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 282-293
```cpp
template<int ElementSize, int BlockWidth, int BlockHeight, int BlockCount>
struct XeSubgroup2DBlockLoadTranspose {
  template<typename T>
  CUTE_HOST_DEVICE
  void operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
                  cute::intel::coord_t coordinate, T *dstPointer) {
    __spirv_Subgroup2DBlockLoadTransposeINTEL(ElementSize, BlockWidth, BlockHeight, BlockCount,
                                              srcBasePointer, memoryWidth,
                                              memoryHeight, memoryPitch, coordinate,
                                              static_cast<void *>(dstPointer));
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTranspose` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTranspose`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 295-304
```cpp
template<int ElementSize, int BlockWidth, int BlockHeight, int BlockCount>
struct XeSubgroup2DBlockPrefetch {
  CUTE_HOST_DEVICE
  void operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
                  cute::intel::coord_t coordinate) {
    __spirv_Subgroup2DBlockPrefetchINTEL(ElementSize, BlockWidth, BlockHeight, BlockCount,
                                         srcBasePointer, memoryWidth,
                                         memoryHeight, memoryPitch, coordinate);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 306-317
```cpp
template<int ElementSize, int BlockWidth, int BlockHeight, int BlockCount>
struct XeSubgroup2DBlockStore {
  template<typename T>
  CUTE_HOST_DEVICE
  void operator()(const void *dstBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
                  cute::intel::coord_t coordinate, T *srcPointer) {
    __spirv_Subgroup2DBlockStoreINTEL(ElementSize, BlockWidth, BlockHeight, BlockCount,
                                      (void *)(srcPointer), dstBasePointer,
                                      memoryWidth, memoryHeight,
                                      memoryPitch, coordinate);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockStore` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockStore`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 319-328
```cpp
template<>
struct XeSubgroup2DBlockLoad<1, 16, 1, 1> {
  template<typename T>
  CUTE_HOST_DEVICE void
  operator()(const void* srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
          cute::intel::coord_t coordinate, T* dstPointer) {
    *reinterpret_cast<intel::uchar *>(dstPointer) =  __builtin_IB_subgroup_block_read_flat_u8_m1k16v1(
       (intptr_t)(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoad` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoad`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 330-339
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

### Lines 341-350
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

### Lines 352-361
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

### Lines 363-372
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

### Lines 374-383
```cpp
template<>
struct XeSubgroup2DBlockLoadTranspose<4, 2, 16, 1> {
  template<typename T>
  CUTE_HOST_DEVICE
  void operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
                  cute::intel::coord_t coordinate, T *dstPointer) {
    *reinterpret_cast<intel::uint2 *>(dstPointer) = __builtin_IB_subgroup_block_read_flat_transpose_u32_k2(
      reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTranspose` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTranspose`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 385-394
```cpp
template<>
struct XeSubgroup2DBlockLoadTranspose<4, 4, 16, 1> {
  template<typename T>
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate, T *dstPointer) {
    *reinterpret_cast<intel::uint4 *>(dstPointer) = __builtin_IB_subgroup_block_read_flat_transpose_u32_k4(
      reinterpret_cast<long>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockLoadTranspose` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockLoadTranspose`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 397-406
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

### Lines 408-417
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

### Lines 419-429
```cpp
// TODO(Codeplay): Remove these Prefetch specializations once spirv prefetch performance bug is fixed.
template<>
struct XeSubgroup2DBlockPrefetch<1, 16, 1, 1> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    __builtin_IB_subgroup_block_read_prefetch_u8_m1k16v1(
      reinterpret_cast<intptr_t>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate,
      CacheControl::kL1C_L3C);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 431-440
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 32, 1, 1> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    __builtin_IB_subgroup_block_read_prefetch_u8_m1k32v1(
      reinterpret_cast<intptr_t>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate,
      CacheControl::kL1C_L3C);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 442-451
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 32, 2, 1> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    __builtin_IB_subgroup_block_read_prefetch_u8_m2k32v1(
      reinterpret_cast<intptr_t>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate,
      CacheControl::kL1C_L3C);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 453-462
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 32, 4, 1> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    __builtin_IB_subgroup_block_read_prefetch_u8_m4k32v1(
      reinterpret_cast<intptr_t>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate,
      CacheControl::kL1C_L3C);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 464-473
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 32, 8, 1> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    __builtin_IB_subgroup_block_read_prefetch_u8_m8k32v1(
      reinterpret_cast<intptr_t>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate,
      CacheControl::kL1C_L3C);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 475-484
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 32, 1, 2> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    intel_sub_group_2d_block_prefetch_8b_1r32x2c(
      (__attribute__((opencl_global)) void*)(srcBasePointer), memoryWidth, memoryHeight,
      memoryPitch, coordinate);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 486-495
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 32, 2, 2> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    intel_sub_group_2d_block_prefetch_8b_2r32x2c(
      (__attribute__((opencl_global)) void *)(srcBasePointer), memoryWidth, memoryHeight,
      memoryPitch, coordinate);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 497-506
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 32, 4, 2> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    intel_sub_group_2d_block_prefetch_8b_4r32x2c(
      (__attribute__((opencl_global)) void *)(srcBasePointer), memoryWidth, memoryHeight, memoryPitch,
      coordinate);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 508-517
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 32, 8, 2> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    intel_sub_group_2d_block_prefetch_8b_8r32x2c(
      (__attribute__((opencl_global)) void *)(srcBasePointer), memoryWidth, memoryHeight, memoryPitch,
      coordinate);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 519-528
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 16, 32, 1> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    intel_sub_group_2d_block_prefetch_8b_32r16x1c(
      (__attribute__((opencl_global)) void *)(srcBasePointer), memoryWidth, memoryHeight, memoryPitch,
      coordinate);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 530-539
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<1, 32, 32, 1> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    __builtin_IB_subgroup_block_read_prefetch_u8_m32k32v1(
      reinterpret_cast<intptr_t>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate,
      CacheControl::kL1C_L3C);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 541-550
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<2, 16, 8, 1> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    __builtin_IB_subgroup_block_read_prefetch_u16_m8k16v1(
      reinterpret_cast<intptr_t>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate,
      CacheControl::kL1C_L3C);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 552-561
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<2, 16, 16, 1> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    __builtin_IB_subgroup_block_read_prefetch_u16_m16k16v1(
      reinterpret_cast<intptr_t>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate,
      CacheControl::kL1C_L3C);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 563-572
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<2, 16, 32, 1> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    __builtin_IB_subgroup_block_read_prefetch_u16_m32k16v1(
      reinterpret_cast<intptr_t>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate,
      CacheControl::kL1C_L3C);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 574-583
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<2, 16, 1, 2> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    intel_sub_group_2d_block_prefetch_16b_1r16x2c(
      (__attribute__((opencl_global)) void *)(srcBasePointer), memoryWidth, memoryHeight, memoryPitch,
      coordinate);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 585-594
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<2, 16, 2, 2> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    intel_sub_group_2d_block_prefetch_16b_2r16x2c(
      (__attribute__((opencl_global)) void *)(srcBasePointer), memoryWidth, memoryHeight, memoryPitch,
      coordinate);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 596-605
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<2, 16, 4, 2> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    intel_sub_group_2d_block_prefetch_16b_4r16x2c(
      (__attribute__((opencl_global)) void *)(srcBasePointer), memoryWidth, memoryHeight, memoryPitch,
      coordinate);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 607-616
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<2, 16, 8, 2> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    __builtin_IB_subgroup_block_read_prefetch_u16_m8k16v2(
      reinterpret_cast<intptr_t>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate,
      CacheControl::kL1C_L3C);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 618-627
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<2, 16, 16, 2> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    __builtin_IB_subgroup_block_read_prefetch_u16_m16k16v2(
      reinterpret_cast<intptr_t>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate,
      CacheControl::kL1C_L3C);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 629-638
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<2, 16, 32, 2> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    __builtin_IB_subgroup_block_read_prefetch_u16_m32k16v2(
      reinterpret_cast<intptr_t>(srcBasePointer), memoryWidth - 1, memoryHeight - 1, memoryPitch - 1, coordinate,
      CacheControl::kL1C_L3C);
  }
};
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 640-650
```cpp
template<>
struct XeSubgroup2DBlockPrefetch<4, 8, 16, 1> {
  CUTE_HOST_DEVICE void
  operator()(const void *srcBasePointer, int memoryWidth, int memoryHeight, int memoryPitch,
             cute::intel::coord_t coordinate) {
    intel_sub_group_2d_block_prefetch_32b_16r8x1c(
      (__attribute__((opencl_global)) void *)(srcBasePointer), memoryWidth, memoryHeight, memoryPitch,
      coordinate);
  }
};
} // namespace cute::detail end
```
- **EN:** Defines `XeSubgroup2DBlockPrefetch` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XeSubgroup2DBlockPrefetch`，把该头文件这一部分所需的类型、常量与行为组织在一起。

## Key Concepts / 关键概念

- **EN:** Copy traits/policies separate abstract data movement from the concrete instruction selected underneath.
  **CN:** Copy trait/策略把抽象数据搬运与底层实际选用的指令解耦。
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
