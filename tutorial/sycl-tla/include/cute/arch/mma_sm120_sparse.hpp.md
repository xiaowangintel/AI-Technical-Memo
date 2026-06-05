# mma_sm120_sparse.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/mma_sm120_sparse.hpp`
- **EN:** Defines matrix-multiply-accumulate instruction wrappers for NVIDIA SM120.
- **CN:** 为 NVIDIA SM120 定义矩阵乘加指令封装。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31
```cpp
/***************************************************************************************************
 * Copyright (c) 2025 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

### Lines 33-37
```cpp
#include <cute/arch/config.hpp>
#include <cute/arch/mma.hpp>
#include <cute/numeric/numeric_types.hpp>  // cute::float_e4m3_t, etc
#include <cute/arch/mma_sm120.hpp>
#include <cutlass/detail/dependent_false.hpp>
```
- **EN:** Imports `cute/arch/config.hpp` (related definitions from `cute/arch/config.hpp`); `cute/arch/mma.hpp` (generic low-level MMA operation tags and wrappers); `cute/numeric/numeric_types.hpp` (numeric helper types such as fixed-width bit wrappers); `cute/arch/mma_sm120.hpp` (related definitions from `cute/arch/mma_sm120.hpp`); `cutlass/detail/dependent_false.hpp` (related definitions from `cutlass/detail/dependent_false.hpp`).
- **CN:** 引入 `cute/arch/config.hpp`（来自 `cute/arch/config.hpp` 的相关定义）；`cute/arch/mma.hpp`（通用底层 MMA 操作标签与封装）；`cute/numeric/numeric_types.hpp`（数值辅助类型，例如固定位宽包装类型）；`cute/arch/mma_sm120.hpp`（来自 `cute/arch/mma_sm120.hpp` 的相关定义）；`cutlass/detail/dependent_false.hpp`（来自 `cutlass/detail/dependent_false.hpp` 的相关定义）。

### Lines 39-39
```cpp
namespace cute {
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 41-41
```cpp
namespace SM120::SPARSE {
```
- **EN:** Enters or leaves namespace scope `SM120::SPARSE` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `SM120::SPARSE`，以便把相关符号组织在一起。

### Lines 43-49
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

template <class a_type, class b_type, class c_type>
struct SM120_SPARSE_16x8x64_TN
{
  static_assert(cutlass::detail::dependent_false<a_type>, "No MMA matches SM120_SPARSE_16x8x64_TN for given data types.");
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 51-87
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E2M1 x E2M1
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m1_t, float_e2m1_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e2m1.e2m1.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 89-125
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E2M1 x E3M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m1_t, float_e3m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e2m1.e3m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 127-163
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E2M1 x E2M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m1_t, float_e2m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e2m1.e2m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 165-201
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E2M1 x E4M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m1_t, float_e4m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e2m1.e4m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 203-239
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E2M1 x E5M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m1_t, float_e5m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e2m1.e5m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 241-277
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E3M2 x E2M1
template <>
struct SM120_SPARSE_16x8x64_TN<float_e3m2_t, float_e2m1_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e3m2.e2m1.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 279-315
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E3M2 x E3M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e3m2_t, float_e3m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e3m2.e3m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 317-353
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E3M2 x E2M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e3m2_t, float_e2m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e3m2.e2m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 355-391
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E3M2 x E4M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e3m2_t, float_e4m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e3m2.e4m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 393-429
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E3M2 x E5M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e3m2_t, float_e5m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e3m2.e5m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 431-467
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E2M3 x E2M1
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m3_t, float_e2m1_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e2m3.e2m1.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 469-505
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E2M3 x E3M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m3_t, float_e3m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e2m3.e3m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 507-543
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E2M3 x E2M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m3_t, float_e2m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e2m3.e2m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 545-581
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E2M3 x E4M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m3_t, float_e4m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e2m3.e4m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 583-619
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E2M3 x E5M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m3_t, float_e5m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e2m3.e5m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 622-658
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E4M3 x E2M1
template <>
struct SM120_SPARSE_16x8x64_TN<float_e4m3_t, float_e2m1_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e4m3.e2m1.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 660-696
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E4M3 x E3M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e4m3_t, float_e3m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e4m3.e3m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 698-734
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E4M3 x E2M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e4m3_t, float_e2m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e4m3.e2m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 736-772
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E4M3 x E4M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e4m3_t, float_e4m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e4m3.e4m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 774-810
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E4M3 x E5M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e4m3_t, float_e5m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e4m3.e5m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 812-848
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E5M2 x E2M1
template <>
struct SM120_SPARSE_16x8x64_TN<float_e5m2_t, float_e2m1_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e5m2.e2m1.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 850-886
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E5M2 x E3M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e5m2_t, float_e3m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e5m2.e3m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 888-924
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E5M2 x E2M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e5m2_t, float_e2m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e5m2.e2m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 926-962
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E5M2 x E4M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e5m2_t, float_e4m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e5m2.e4m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 964-1000
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP32 ACC and inputs E5M2 x E5M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e5m2_t, float_e5m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f32.e5m2.e5m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "%16, 0x0;\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1003-1038
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////
// MMA 16x8x64 TN with FP16 ACC and inputs E2M1 x E2M1
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m1_t, float_e2m1_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e2m1.e2m1.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1040-1076
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E2M1 x E3M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m1_t, float_e3m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e2m1.e3m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1078-1114
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E2M1 x E2M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m1_t, float_e2m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e2m1.e2m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1116-1152
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E2M1 x E4M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m1_t, float_e4m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e2m1.e4m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1154-1190
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E2M1 x E5M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m1_t, float_e5m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e2m1.e5m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1192-1228
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E3M2 x E2M1
template <>
struct SM120_SPARSE_16x8x64_TN<float_e3m2_t, float_e2m1_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e3m2.e2m1.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1230-1266
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E3M2 x E3M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e3m2_t, float_e3m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e3m2.e3m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1268-1304
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E3M2 x E2M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e3m2_t, float_e2m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e3m2.e2m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1306-1342
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E3M2 x E4M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e3m2_t, float_e4m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e3m2.e4m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1344-1380
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E3M2 x E5M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e3m2_t, float_e5m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e3m2.e5m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1383-1419
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E2M3 x E2M1
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m3_t, float_e2m1_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e2m3.e2m1.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1421-1457
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E2M3 x E3M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m3_t, float_e3m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e2m3.e3m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1459-1495
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E2M3 x E2M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m3_t, float_e2m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e2m3.e2m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1497-1533
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E2M3 x E4M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m3_t, float_e4m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e2m3.e4m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1535-1571
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E2M3 x E5M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e2m3_t, float_e5m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e2m3.e5m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1573-1609
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E4M3 x E2M1
template <>
struct SM120_SPARSE_16x8x64_TN<float_e4m3_t, float_e2m1_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e4m3.e2m1.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1611-1647
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E4M3 x E3M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e4m3_t, float_e3m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e4m3.e3m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1649-1685
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E4M3 x E2M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e4m3_t, float_e2m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e4m3.e2m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1687-1723
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E4M3 x E4M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e4m3_t, float_e4m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e4m3.e4m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1725-1761
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E4M3 x E5M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e4m3_t, float_e5m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e4m3.e5m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1763-1799
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E5M2 x E2M1
template <>
struct SM120_SPARSE_16x8x64_TN<float_e5m2_t, float_e2m1_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e5m2.e2m1.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1801-1837
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E5M2 x E3M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e5m2_t, float_e3m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e5m2.e3m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1839-1875
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E5M2 x E2M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e5m2_t, float_e2m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e5m2.e2m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1877-1913
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E5M2 x E4M3
template <>
struct SM120_SPARSE_16x8x64_TN<float_e5m2_t, float_e4m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e5m2.e4m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1915-1951
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN with FP16 ACC and inputs E5M2 x E5M2
template <>
struct SM120_SPARSE_16x8x64_TN<float_e5m2_t, float_e5m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = uint32_t[2];
  using ERegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      uint32_t const& c0, uint32_t const& c1,
      uint32_t const& e)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.sp::ordered_metadata.m16n8k64.row.col.f16.e5m2.e5m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4, %5},"
      "{%6,  %7,  %8, %9},"
      "{%10, %11},"
      "%12, 0x0;\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),  "r"(b2),  "r"(b3),
         "r"(c0),  "r"(c1),
         "r"(e));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1953-1953
```cpp
} // end namespace SM120::SPARSE
```
- **EN:** Enters or leaves namespace scope `SM120::SPARSE` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `SM120::SPARSE`，以便把相关符号组织在一起。

### Lines 1955-1955
```cpp
namespace SM120::BLOCKSCALED::SPARSE {
```
- **EN:** Enters or leaves namespace scope `SM120::BLOCKSCALED::SPARSE` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `SM120::BLOCKSCALED::SPARSE`，以便把相关符号组织在一起。

### Lines 1957-1963
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

template <class a_type, class b_type, class c_type, class sf_type, int VS>
struct SM120_SPARSE_16x8x64_TN_VS
{
  static_assert(cutlass::detail::dependent_false<a_type>, "No MMA matches SM120_SPARSE_16x8x64_TN_VS for given data types.");
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1965-1969
```cpp
template <class a_type, class b_type, class c_type, class sf_type, int VS>
struct SM120_SPARSE_16x8x128_TN_VS
{
  static_assert(cutlass::detail::dependent_false<a_type>, "No MMA matches SM120_SPARSE_16x8x128_TN_VS for given data types.");
};
```
- **EN:** Defines `SM120_SPARSE_16x8x128_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x128_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1971-2022
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E2M1 x E2M1, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e2m1_t, float_e2m1_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e2m1.e2m1.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2024-2075
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E2M1 x E3M2, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e2m1_t, float_e3m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e2m1.e3m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2077-2128
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E2M1 x E2M3, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e2m1_t, float_e2m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e2m1.e2m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2130-2181
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E2M1 x E4M3, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e2m1_t, float_e4m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e2m1.e4m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2183-2234
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E2M1 x E5M2, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e2m1_t, float_e5m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e2m1.e5m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2236-2287
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E3M2 x E2M1, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e3m2_t, float_e2m1_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e3m2.e2m1.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2289-2340
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E3M2 x E3M2, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e3m2_t, float_e3m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e3m2.e3m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2342-2393
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E3M2 x E2M3, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e3m2_t, float_e2m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e3m2.e2m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2395-2446
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E3M2 x E4M3, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e3m2_t, float_e4m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e3m2.e4m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2448-2499
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E3M2 x E5M2, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e3m2_t, float_e5m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e3m2.e5m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2501-2552
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E2M3 x E2M1, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e2m3_t, float_e2m1_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e2m3.e2m1.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2554-2605
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E2M3 x E3M2, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e2m3_t, float_e3m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e2m3.e3m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2607-2658
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E2M3 x E2M3, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e2m3_t, float_e2m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e2m3.e2m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2660-2711
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E2M3 x E4M3, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e2m3_t, float_e4m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e2m3.e4m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2713-2764
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E2M3 x E5M2, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e2m3_t, float_e5m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e2m3.e5m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2766-2817
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E4M3 x E2M1, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e4m3_t, float_e2m1_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e4m3.e2m1.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2819-2870
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E4M3 x E3M2, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e4m3_t, float_e3m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e4m3.e3m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2872-2923
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E4M3 x E2M3, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e4m3_t, float_e2m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e4m3.e2m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2925-2976
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E4M3 x E4M3, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e4m3_t, float_e4m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e4m3.e4m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2978-3029
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E4M3 x E5M2, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e4m3_t, float_e5m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e4m3.e5m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 3031-3082
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E5M2 x E2M1, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e5m2_t, float_e2m1_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e5m2.e2m1.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 3084-3135
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E5M2 x E3M2, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e5m2_t, float_e3m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e5m2.e3m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 3137-3188
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E5M2 x E2M3, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e5m2_t, float_e2m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e5m2.e2m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 3190-3241
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E5M2 x E4M3, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e5m2_t, float_e4m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e5m2.e4m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 3243-3294
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x64 TN with FP32 ACC and inputs E5M2 x E5M2, SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x64_TN_VS<float_e5m2_t, float_e5m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];
  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1, uint32_t const& b2, uint32_t const& b3,
      float    const& c0, float    const& c1, float    const& c2, float    const& c3,
      uint32_t const&  e, uint8_t const& sfa, uint8_t  const& sfb)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64, "Scaling factor vector size has to be 64 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.sp::ordered_metadata.block_scale.scale_vec::1X.m16n8k64.row.col.f32.e5m2.e5m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9,  %10, %11},"
    "{%12, %13, %14, %15},"
    "{%16}, 0x0,"
    "{%17},"
    "{%18, %19},"
    "{%20},"
    "{%21,  %22};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(e),
        "r"((uint32_t)sfa), "h"(bidA), "h"(tidA),
        "r"((uint32_t)sfb), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_SPARSE_16x8x64_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x64_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x64_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 3296-3348
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x128 TN E2M1 x E2M1 with SF UE8M0
template <int VS>
struct SM120_SPARSE_16x8x128_TN_VS<float_e2m1_t, float_e2m1_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  static constexpr int SFBits = (VS == 64) ? 16 : 32;
  using RegTypeSF = uint_bit_t<SFBits>;
  using SFARegisters = RegTypeSF[1];
  using SFBRegisters = RegTypeSF[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         &   d1, float          &  d2, float         & d3,
      uint32_t const& a0, uint32_t  const&  a1, uint32_t  const&  a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t  const&  b1, uint32_t  const&  b2, uint32_t const& b3,
      float    const& c0, float     const&  c1, float     const&  c2, float    const& c3,
      uint32_t const&  e, RegTypeSF const& sfa, RegTypeSF const& sfb)
  {

    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 64 || VS == 32, "Scaling factor vector size has to be 64 or 32 for MXF4NVF4.");
#if defined(CUTE_ARCH_MXF4NVF4_2X_UE8M0_MMA_ENABLED)
      asm volatile(
      "mma.sync.aligned.kind::mxf4nvf4.sp::ordered_metadata.block_scale.scale_vec::2X.m16n8k128.row.col.f32.e2m1.e2m1.f32.ue8m0 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "{%16}, 0x0,"
      "{%17}, {%18, %19},"
      "{%20}, {%21, %22};\n"
      :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
      :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
          "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
          "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
          "r"(e),
          "r"(uint32_t(sfa)), "h"(bidA), "h"(tidA),
          "r"(uint32_t(sfb)), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::SPARSE::SM120_SPARSE_16x8x128_TN_VS without CUTE_ARCH_MXF4NVF4_2X_UE8M0_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x128_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x128_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 3350-3401
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA SPARSE BLOCKSCALED 16x8x128 TN E2M1 x E2M1 with SF E4M3
template <int VS>
struct SM120_SPARSE_16x8x128_TN_VS<float_e2m1_t, float_e2m1_t, float, float_ue4m3_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[4];
  using CRegisters = float[4];
  using ERegisters = uint32_t[1];

  static constexpr int SFBits = (VS == 64) ? 16 : 32;
  using RegTypeSF = uint_bit_t<SFBits>;
  using SFARegisters = RegTypeSF[1];
  using SFBRegisters = RegTypeSF[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         &   d1, float          &  d2, float         & d3,
      uint32_t const& a0, uint32_t  const&  a1, uint32_t  const&  a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t  const&  b1, uint32_t  const&  b2, uint32_t const& b3,
      float    const& c0, float     const&  c1, float     const&  c2, float    const& c3,
      uint32_t const&  e, RegTypeSF const& sfa, RegTypeSF const& sfb)
  {
#if defined(CUTE_ARCH_MXF4NVF4_4X_UE4M3_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for NVF4 with e2m1 and scale factor e4m3.");
    asm volatile(
      "mma.sync.aligned.kind::mxf4nvf4.sp::ordered_metadata.block_scale.scale_vec::4X.m16n8k128.row.col.f32.e2m1.e2m1.f32.ue4m3 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9,  %10, %11},"
      "{%12, %13, %14, %15},"
      "{%16}, 0x0,"
      "{%17}, {%18, %19},"
      "{%20}, {%21, %22};\n"
      :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
      :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
          "r"(b0),   "r"(b1),   "r"(b2),   "r"(b3),
          "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
          "r"(e),
          "r"(uint32_t(sfa)), "h"(bidA), "h"(tidA),
          "r"(uint32_t(sfb)), "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::SPARSE::SM120_SPARSE_16x8x128_TN_VS without CUTE_ARCH_MXF4NVF4_4X_UE4M3_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_SPARSE_16x8x128_TN_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_SPARSE_16x8x128_TN_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 3403-3405
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

} // end namespace SM120::BLOCKSCALED::SPARSE
```
- **EN:** Enters or leaves namespace scope `SM120::BLOCKSCALED::SPARSE` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `SM120::BLOCKSCALED::SPARSE`，以便把相关符号组织在一起。

### Lines 3407-3409
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

////////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Section comment introducing the next logical part of the file: 
- **CN:** 分节注释，用来引出文件中的下一段逻辑：

### Lines 3411-3422
```cpp
template <
  class ElementA,
  class ElementB,
  class ElementC
>
CUTE_HOST_DEVICE constexpr
auto
rr_sparse_op_selector_sm120()
{
  // Get MMA SPARSE OP
  return SM120::SPARSE::SM120_SPARSE_16x8x64_TN<ElementA, ElementB, ElementC>{};
}
```
- **EN:** Defines or forwards `rr_sparse_op_selector_sm120` as part of this header's executable interface.
- **CN:** 定义或转发 `rr_sparse_op_selector_sm120`，作为该头文件可执行接口的一部分。

### Lines 3424-3442
```cpp
template <
  class  ElementA,
  class  ElementB,
  class  ElementC,
  class  ElementSF,
  int    SFVecSize,
  bool   UseF8F6F4
>
CUTE_HOST_DEVICE constexpr
auto
rr_blockscaled_sparse_op_selector_sm120()
{
  if constexpr (UseF8F6F4) {
    return SM120::BLOCKSCALED::SPARSE::SM120_SPARSE_16x8x64_TN_VS<ElementA, ElementB, ElementC, ElementSF, SFVecSize>{};
  }
  else {
    return SM120::BLOCKSCALED::SPARSE::SM120_SPARSE_16x8x128_TN_VS<ElementA, ElementB, ElementC, ElementSF, SFVecSize>{};
  }
}
```
- **EN:** Defines or forwards `rr_blockscaled_sparse_op_selector_sm120` as part of this header's executable interface.
- **CN:** 定义或转发 `rr_blockscaled_sparse_op_selector_sm120`，作为该头文件可执行接口的一部分。

### Lines 3444-3444
```cpp
} // namespace cute
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

## Key Concepts / 关键概念

- **EN:** MMA/GMMA/UMMA wrappers expose tensor-core style matrix instructions as typed C++ interfaces.
  **CN:** MMA/GMMA/UMMA 封装把张量核心式矩阵指令暴露为带类型的 C++ 接口。
- **EN:** Inline assembly keeps the mapping between C++ fragments and hardware registers explicit.
  **CN:** 内联汇编使 C++ 片段与硬件寄存器之间的映射保持显式。
- **EN:** Compile-time assertions encode hardware and type constraints directly in the API surface.
  **CN:** 编译期断言把硬件与类型约束直接编码到 API 表面。

## Dependencies / 依赖关系

- **EN:** `cute/arch/config.hpp` supplies related definitions from `cute/arch/config.hpp`.
  **CN:** `cute/arch/config.hpp` 提供了来自 `cute/arch/config.hpp` 的相关定义。
- **EN:** `cute/arch/mma.hpp` supplies generic low-level MMA operation tags and wrappers.
  **CN:** `cute/arch/mma.hpp` 提供了通用底层 MMA 操作标签与封装。
- **EN:** `cute/numeric/numeric_types.hpp` supplies numeric helper types such as fixed-width bit wrappers.
  **CN:** `cute/numeric/numeric_types.hpp` 提供了数值辅助类型，例如固定位宽包装类型。
- **EN:** `cute/arch/mma_sm120.hpp` supplies related definitions from `cute/arch/mma_sm120.hpp`.
  **CN:** `cute/arch/mma_sm120.hpp` 提供了来自 `cute/arch/mma_sm120.hpp` 的相关定义。
- **EN:** `cutlass/detail/dependent_false.hpp` supplies related definitions from `cutlass/detail/dependent_false.hpp`.
  **CN:** `cutlass/detail/dependent_false.hpp` 提供了来自 `cutlass/detail/dependent_false.hpp` 的相关定义。
- **EN:** CUTLASS headers provide adapter types, synchronization hooks, or shared low-level GPU infrastructure.
  **CN:** CUTLASS 头文件提供适配类型、同步钩子或共享的底层 GPU 基础设施。
