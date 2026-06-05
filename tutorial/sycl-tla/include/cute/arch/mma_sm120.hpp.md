# mma_sm120.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/mma_sm120.hpp`
- **EN:** Defines matrix-multiply-accumulate instruction wrappers for NVIDIA SM120.
- **CN:** 为 NVIDIA SM120 定义矩阵乘加指令封装。

## Line-by-Line Analysis / 逐行分析

### Lines 1-33
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

### Lines 35-38
```cpp
#include <cute/arch/config.hpp>
#include <cute/arch/mma.hpp>
#include <cute/numeric/numeric_types.hpp> // cute::float_e4m3_t, etc
#include <cutlass/detail/dependent_false.hpp>
```
- **EN:** Imports `cute/arch/config.hpp` (related definitions from `cute/arch/config.hpp`); `cute/arch/mma.hpp` (generic low-level MMA operation tags and wrappers); `cute/numeric/numeric_types.hpp` (numeric helper types such as fixed-width bit wrappers); `cutlass/detail/dependent_false.hpp` (related definitions from `cutlass/detail/dependent_false.hpp`).
- **CN:** 引入 `cute/arch/config.hpp`（来自 `cute/arch/config.hpp` 的相关定义）；`cute/arch/mma.hpp`（通用底层 MMA 操作标签与封装）；`cute/numeric/numeric_types.hpp`（数值辅助类型，例如固定位宽包装类型）；`cutlass/detail/dependent_false.hpp`（来自 `cutlass/detail/dependent_false.hpp` 的相关定义）。

### Lines 40-40
```cpp
namespace cute {
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 42-48
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

template <class a_type, class b_type, class c_type>
struct SM120_16x8x32_TN
{
  static_assert(cutlass::detail::dependent_false<a_type>, "No MMA matches SM120_16x8x32_TN for given data types.");
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 51-83
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E2M1 x E2M1
template <>
struct SM120_16x8x32_TN<float_e2m1_t, float_e2m1_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e2m1.e2m1.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 85-117
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E2M1 x E3M2
template <>
struct SM120_16x8x32_TN<float_e2m1_t, float_e3m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e2m1.e3m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 119-151
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E2M1 x E2M3
template <>
struct SM120_16x8x32_TN<float_e2m1_t, float_e2m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e2m1.e2m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 153-185
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E2M1 x E4M3
template <>
struct SM120_16x8x32_TN<float_e2m1_t, float_e4m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e2m1.e4m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 187-219
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E2M1 x E5M2
template <>
struct SM120_16x8x32_TN<float_e2m1_t, float_e5m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e2m1.e5m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 221-252
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////
// MMA 16x8x32 TN E3M2 x E2M1
template <>
struct SM120_16x8x32_TN<float_e3m2_t, float_e2m1_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e3m2.e2m1.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 254-286
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E3M2 x E3M2
template <>
struct SM120_16x8x32_TN<float_e3m2_t, float_e3m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e3m2.e3m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 288-320
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E3M2 x E2M3
template <>
struct SM120_16x8x32_TN<float_e3m2_t, float_e2m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e3m2.e2m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 322-354
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E3M2 x E4M3
template <>
struct SM120_16x8x32_TN<float_e3m2_t, float_e4m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e3m2.e4m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 356-388
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E3M2 x E5M2
template <>
struct SM120_16x8x32_TN<float_e3m2_t, float_e5m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e3m2.e5m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 390-424
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E2M3 x E2M1
template <>
struct SM120_16x8x32_TN<float_e2m3_t, float_e2m1_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e2m3.e2m1.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 426-458
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E2M3 x E3M2
template <>
struct SM120_16x8x32_TN<float_e2m3_t, float_e3m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e2m3.e3m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 460-492
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E2M3 x E2M3
template <>
struct SM120_16x8x32_TN<float_e2m3_t, float_e2m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e2m3.e2m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 494-526
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E2M3 x E4M3
template <>
struct SM120_16x8x32_TN<float_e2m3_t, float_e4m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e2m3.e4m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 528-560
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E2M3 x E5M2
template <>
struct SM120_16x8x32_TN<float_e2m3_t, float_e5m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e2m3.e5m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 562-596
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E4M3 x E2M1
template <>
struct SM120_16x8x32_TN<float_e4m3_t, float_e2m1_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e4m3.e2m1.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 598-630
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E4M3 x E3M2
template <>
struct SM120_16x8x32_TN<float_e4m3_t, float_e3m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e4m3.e3m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 632-664
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E4M3 x E2M3
template <>
struct SM120_16x8x32_TN<float_e4m3_t, float_e2m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e4m3.e2m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 666-698
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E4M3 x E4M3
template <>
struct SM120_16x8x32_TN<float_e4m3_t, float_e4m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e4m3.e4m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 700-732
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E4M3 x E5M2
template <>
struct SM120_16x8x32_TN<float_e4m3_t, float_e5m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e4m3.e5m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 734-768
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E5M2 x E2M1
template <>
struct SM120_16x8x32_TN<float_e5m2_t, float_e2m1_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e5m2.e2m1.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 770-802
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E5M2 x E3M2
template <>
struct SM120_16x8x32_TN<float_e5m2_t, float_e3m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e5m2.e3m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 804-836
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E5M2 x E2M3
template <>
struct SM120_16x8x32_TN<float_e5m2_t, float_e2m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e5m2.e2m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 838-870
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E5M2 x E4M3
template <>
struct SM120_16x8x32_TN<float_e5m2_t, float_e4m3_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e5m2.e4m3.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 872-904
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN E5M2 x E5M2
template <>
struct SM120_16x8x32_TN<float_e5m2_t, float_e5m2_t, float>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f32.e5m2.e5m2.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 906-939
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E2M1 x E2M1
template <>
struct SM120_16x8x32_TN<float_e2m1_t, float_e2m1_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e2m1.e2m1.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 941-973
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E2M1 x E3M2
template <>
struct SM120_16x8x32_TN<float_e2m1_t, float_e3m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e2m1.e3m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 975-1007
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E2M1 x E2M3
template <>
struct SM120_16x8x32_TN<float_e2m1_t, float_e2m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e2m1.e2m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1009-1041
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E2M1 x E4M3
template <>
struct SM120_16x8x32_TN<float_e2m1_t, float_e4m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e2m1.e4m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1043-1075
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E2M1 x E5M2
template <>
struct SM120_16x8x32_TN<float_e2m1_t, float_e5m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e2m1.e5m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1077-1109
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E3M2 x E2M1
template <>
struct SM120_16x8x32_TN<float_e3m2_t, float_e2m1_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e3m2.e2m1.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1111-1143
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E3M2 x E3M2
template <>
struct SM120_16x8x32_TN<float_e3m2_t, float_e3m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e3m2.e3m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1145-1177
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E3M2 x E2M3
template <>
struct SM120_16x8x32_TN<float_e3m2_t, float_e2m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e3m2.e2m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1179-1211
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E3M2 x E4M3
template <>
struct SM120_16x8x32_TN<float_e3m2_t, float_e4m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e3m2.e4m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1213-1245
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E3M2 x E5M2
template <>
struct SM120_16x8x32_TN<float_e3m2_t, float_e5m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e3m2.e5m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1247-1279
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E2M3 x E2M1
template <>
struct SM120_16x8x32_TN<float_e2m3_t, float_e2m1_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e2m3.e2m1.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1281-1313
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E2M3 x E3M2
template <>
struct SM120_16x8x32_TN<float_e2m3_t, float_e3m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e2m3.e3m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1315-1347
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E2M3 x E2M3
template <>
struct SM120_16x8x32_TN<float_e2m3_t, float_e2m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e2m3.e2m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1349-1381
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E2M3 x E4M3
template <>
struct SM120_16x8x32_TN<float_e2m3_t, float_e4m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e2m3.e4m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1383-1415
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E2M3 x E5M2
template <>
struct SM120_16x8x32_TN<float_e2m3_t, float_e5m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e2m3.e5m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1417-1450
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E4M3 x E2M1
template <>
struct SM120_16x8x32_TN<float_e4m3_t, float_e2m1_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e4m3.e2m1.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1452-1484
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E4M3 x E3M2
template <>
struct SM120_16x8x32_TN<float_e4m3_t, float_e3m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e4m3.e3m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1486-1518
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E4M3 x E2M3
template <>
struct SM120_16x8x32_TN<float_e4m3_t, float_e2m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e4m3.e2m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1520-1552
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E4M3 x E4M3
template <>
struct SM120_16x8x32_TN<float_e4m3_t, float_e4m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e4m3.e4m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1554-1586
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E4M3 x E5M2
template <>
struct SM120_16x8x32_TN<float_e4m3_t, float_e5m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e4m3.e5m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1588-1620
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E5M2 x E2M1
template <>
struct SM120_16x8x32_TN<float_e5m2_t, float_e2m1_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e5m2.e2m1.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1622-1654
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E5M2 x E3M2
template <>
struct SM120_16x8x32_TN<float_e5m2_t, float_e3m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e5m2.e3m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1656-1688
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E5M2 x E2M3
template <>
struct SM120_16x8x32_TN<float_e5m2_t, float_e2m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e5m2.e2m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1690-1722
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E5M2 x E4M3
template <>
struct SM120_16x8x32_TN<float_e5m2_t, float_e4m3_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e5m2.e4m3.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1724-1756
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.F16 16x8x32 TN E5M2 x E5M2
template <>
struct SM120_16x8x32_TN<float_e5m2_t, float_e5m2_t, half_t>
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_F8F6F4_MMA_ENABLED)
    asm volatile(
      "mma.sync.aligned.kind::f8f6f4.m16n8k32.row.col.f16.e5m2.e5m2.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120_16x8x32_TN without CUTE_ARCH_F8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM120_16x8x32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM120_16x8x32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1758-3217
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////////////////////////

namespace SM120::BLOCKSCALED {
////////////////////////////////////////////////////////////////////////////////////////////////////

template <class a_type, class b_type, class c_type, class sf_type, int VS>
struct SM120_16x8x32_TN_VS
{
  static_assert(cutlass::detail::dependent_false<a_type>, "No MMA matches SM120_16x8x32_TN_VS for given data types.");
};

////////////////////////////////////////////////////////////////////////////////////////////////////

template <class a_type, class b_type, class c_type, class sf_type, int VS>
struct SM120_16x8x64_TN_VS
{
  static_assert(cutlass::detail::dependent_false<a_type>, "No MMA matches SM120_16x8x64_TN_VS for given data types.");
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E2M1 x E2M1 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e2m1_t, float_e2m1_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e2m1.e2m1.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)), "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)), "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E2M1 x E3M2 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e2m1_t, float_e3m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e2m1.e3m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E2M1 x E2M3 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e2m1_t, float_e2m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e2m1.e2m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E2M1 x E4M3 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e2m1_t, float_e4m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e2m1.e4m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E2M1 x E5M2 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e2m1_t, float_e5m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e2m1.e5m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E3M2 x E2M1 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e3m2_t, float_e2m1_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e3m2.e2m1.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E3M2 x E3M2 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e3m2_t, float_e3m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e3m2.e3m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E3M2 x E2M3 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e3m2_t, float_e2m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e3m2.e2m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E3M2 x E4M3 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e3m2_t, float_e4m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e3m2.e4m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E3M2 x E5M2 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e3m2_t, float_e5m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e3m2.e5m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E2M3 x E2M1 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e2m3_t, float_e2m1_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e2m3.e2m1.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E2M3 x E3M2 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e2m3_t, float_e3m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e2m3.e3m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E2M3 x E2M3 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e2m3_t, float_e2m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e2m3.e2m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E2M3 x E4M3 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e2m3_t, float_e4m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e2m3.e4m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E2M3 x E5M2 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e2m3_t, float_e5m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e2m3.e5m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E4M3 x E2M1 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e4m3_t, float_e2m1_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e4m3.e2m1.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E4M3 x E3M2 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e4m3_t, float_e3m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e4m3.e3m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E4M3 x E2M3 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e4m3_t, float_e2m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e4m3.e2m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E4M3 x E4M3 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e4m3_t, float_e4m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e4m3.e4m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E4M3 x E5M2 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e4m3_t, float_e5m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e4m3.e5m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E5M2 x E2M1 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e5m2_t, float_e2m1_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e5m2.e2m1.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E5M2 x E3M2 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e5m2_t, float_e3m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e5m2.e3m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E5M2 x E2M3 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e5m2_t, float_e2m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e5m2.e2m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E5M2 x E4M3 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e5m2_t, float_e4m3_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e5m2.e4m3.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x32 TN E5M2 x E5M2 with SF UE8M0
template <int VS>
struct SM120_16x8x32_TN_VS<float_e5m2_t, float_e5m2_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  using SFARegisters = uint8_t[1];
  using SFBRegisters = uint8_t[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      uint8_t const& sfa0,
      uint8_t const& sfb0)
  {
#if defined(CUTE_ARCH_MXF8F6F4_MMA_ENABLED)
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

    CUTE_STATIC_ASSERT(VS == 32, "Scaling factor vector size has to be 32 for MXF8F6F4 MMA.");

    asm volatile(
    "mma.sync.aligned.kind::mxf8f6f4.block_scale.scale_vec::1X.m16n8k32.row.col.f32.e5m2.e5m2.f32.ue8m0 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));

#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x32_TN_VS without CUTE_ARCH_MXF8F6F4_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x64 TN E2M1 x E2M1 with SF UE8M0
template <int VS>
struct SM120_16x8x64_TN_VS<float_e2m1_t, float_e2m1_t, float, float_ue8m0_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  static constexpr int SFBits = (VS == 32) ? 16 : 32;
  using RegTypeSF = uint_bit_t<SFBits>;

  using SFARegisters = RegTypeSF[1];
  using SFBRegisters = RegTypeSF[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      RegTypeSF const& sfa0,
      RegTypeSF const& sfb0)
  {
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;
    CUTE_STATIC_ASSERT(VS == 16 || VS == 32, "Scaling factor vector size has to be 16 or 32 for MXF4NVF4 MMA.");

#if defined(CUTE_ARCH_MXF4NVF4_2X_UE8M0_MMA_ENABLED)
      asm volatile(
      "mma.sync.aligned.kind::mxf4nvf4.block_scale.scale_vec::2X.m16n8k64.row.col.f32.e2m1.e2m1.f32.ue8m0 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13},"
      "{%14},"
      "{%15, %16},"
      "{%17},"
      "{%18, %19};\n"
      :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
      :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
          "r"(b0),   "r"(b1),
          "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
          "r"(uint32_t(sfa0)),  "h"(bidA), "h"(tidA),
          "r"(uint32_t(sfb0)),  "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x64_TN_VS without CUTE_ARCH_MXF4NVF4_2X_UE8M0_MMA_ENABLED");
#endif
  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA.SF 16x8x64 TN E2M1 x E2M1 with SF E4M3
template <int VS>
struct SM120_16x8x64_TN_VS<float_e2m1_t, float_e2m1_t, float, float_ue4m3_t, VS>
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  static constexpr int SFBits = (VS == 32) ? 16 : 32;
  using RegTypeSF = uint_bit_t<SFBits>;

  using SFARegisters = RegTypeSF[1];
  using SFBRegisters = RegTypeSF[1];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3,
      RegTypeSF const& sfa0,
      RegTypeSF const& sfb0)
  {
    static constexpr uint16_t tidA = 0;
    static constexpr uint16_t bidA = 0;
    static constexpr uint16_t tidB = 0;
    static constexpr uint16_t bidB = 0;

 CUTE_STATIC_ASSERT(VS == 16 || VS == 32, "Scaling factor vector size has to be 16 or 32 for MXF4NVF4 MMA.");
#if defined(CUTE_ARCH_MXF4NVF4_4X_UE4M3_MMA_ENABLED)
    asm volatile(
    "mma.sync.aligned.kind::mxf4nvf4.block_scale.scale_vec::4X.m16n8k64.row.col.f32.e2m1.e2m1.f32.ue4m3 "
    "{%0,  %1,  %2,  %3},"
    "{%4,  %5,  %6,  %7},"
    "{%8,  %9},"
    "{%10, %11, %12, %13},"
    "{%14},"
    "{%15, %16},"
    "{%17},"
    "{%18, %19};\n"
    :  "=f"(d0),  "=f"(d1),  "=f"(d2),  "=f"(d3)
    :   "r"(a0),   "r"(a1),   "r"(a2),   "r"(a3),
        "r"(b0),   "r"(b1),
        "f"(c0),   "f"(c1),   "f"(c2),   "f"(c3),
        "r"(uint32_t(sfa0)) , "h"(bidA), "h"(tidA),
        "r"(uint32_t(sfb0)) , "h"(bidB), "h"(tidB));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM120::BLOCKSCALED::SM120_16x8x64_TN_VS without CUTE_ARCH_MXF4NVF4_4X_UE4M3_MMA_ENABLED");
#endif

  }
};

////////////////////////////////////////////////////////////////////////////////////////////////////

} // namespace SM120::BLOCKSCALED
```
- **EN:** Enters or leaves namespace scope `SM120::BLOCKSCALED, SM120::BLOCKSCALED` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `SM120::BLOCKSCALED, SM120::BLOCKSCALED`，以便把相关符号组织在一起。

### Lines 3220-3232
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

template <
  class ElementA,
  class ElementB,
  class ElementC
>
CUTE_HOST_DEVICE constexpr
auto
rr_op_selector_sm120()
{
  return SM120_16x8x32_TN<ElementA, ElementB, ElementC>{};
}
```
- **EN:** Defines or forwards `rr_op_selector_sm120` as part of this header's executable interface.
- **CN:** 定义或转发 `rr_op_selector_sm120`，作为该头文件可执行接口的一部分。

### Lines 3234-3252
```cpp
template <
  class ElementA,
  class ElementB,
  class ElementC,
  class ElementSF,
  int   SFVecSize,
  bool  UseF8F6F4
>
CUTE_HOST_DEVICE constexpr
auto
rr_blockscaled_op_selector_sm120()
{
  if constexpr (UseF8F6F4) {
    return SM120::BLOCKSCALED::SM120_16x8x32_TN_VS<ElementA, ElementB, ElementC, ElementSF, SFVecSize>{};
  }
  else{
    return SM120::BLOCKSCALED::SM120_16x8x64_TN_VS<ElementA, ElementB, ElementC, ElementSF, SFVecSize>{};
  }
}
```
- **EN:** Defines or forwards `rr_blockscaled_op_selector_sm120` as part of this header's executable interface.
- **CN:** 定义或转发 `rr_blockscaled_op_selector_sm120`，作为该头文件可执行接口的一部分。

### Lines 3254-3254
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
- **EN:** `cutlass/detail/dependent_false.hpp` supplies related definitions from `cutlass/detail/dependent_false.hpp`.
  **CN:** `cutlass/detail/dependent_false.hpp` 提供了来自 `cutlass/detail/dependent_false.hpp` 的相关定义。
- **EN:** CUTLASS headers provide adapter types, synchronization hooks, or shared low-level GPU infrastructure.
  **CN:** CUTLASS 头文件提供适配类型、同步钩子或共享的底层 GPU 基础设施。
