# mma_sm80.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/mma_sm80.hpp`
- **EN:** Defines matrix-multiply-accumulate instruction wrappers for NVIDIA SM80.
- **CN:** 为 NVIDIA SM80 定义矩阵乘加指令封装。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
/***************************************************************************************************
 * Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

### Lines 34-36
```cpp
#include <cute/config.hpp>
#include <cute/arch/mma.hpp>
#include <cute/numeric/complex.hpp>
```
- **EN:** Imports `cute/config.hpp` (core CuTe configuration macros and portability annotations); `cute/arch/mma.hpp` (generic low-level MMA operation tags and wrappers); `cute/numeric/complex.hpp` (related definitions from `cute/numeric/complex.hpp`).
- **CN:** 引入 `cute/config.hpp`（CuTe 核心配置宏与可移植性标注）；`cute/arch/mma.hpp`（通用底层 MMA 操作标签与封装）；`cute/numeric/complex.hpp`（来自 `cute/numeric/complex.hpp` 的相关定义）。

### Lines 38-41
```cpp
// Config
#if (defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 800)) || \
    (defined(__SYCL_CUDA_ARCH__) && (__SYCL_CUDA_ARCH__ >= 800))
#  define CUTE_ARCH_MMA_SM80_ENABLED
```
- **EN:** Implements supporting control flow for the surrounding type or function.
- **CN:** 实现周边类型或函数所需的辅助控制流。

### Lines 43-45
```cpp
#if (__CUDA_ARCH__ <= 900 || __SYCL_CUDA_ARCH__ <= 900)
#define CUTE_ARCH_MMA_B1_AND_SM80_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (__CUDA_ARCH__, __SYCL_CUDA_ARCH__, CUTE_ARCH_MMA_B1_AND_SM80_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（__CUDA_ARCH__, __SYCL_CUDA_ARCH__, CUTE_ARCH_MMA_B1_AND_SM80_ENABLED）只启用当前目标有效的构建路径。

### Lines 47-49
```cpp
#if (__CUDA_ARCH__ <= 890 || __SYCL_CUDA_ARCH__ <= 890)
#define CUTE_ARCH_MMA_B1_XOR_SM80_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (__CUDA_ARCH__, __SYCL_CUDA_ARCH__, CUTE_ARCH_MMA_B1_XOR_SM80_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（__CUDA_ARCH__, __SYCL_CUDA_ARCH__, CUTE_ARCH_MMA_B1_XOR_SM80_ENABLED）只启用当前目标有效的构建路径。

### Lines 51-51
```cpp
#endif
```
- **EN:** Uses preprocessor conditions to select architecture-specific or build-specific behavior.
- **CN:** 使用预处理条件选择特定体系结构或构建配置下的行为。

### Lines 55-55
```cpp
namespace cute {
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 57-88
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x8 TN
struct SM80_16x8x8_F16F16F16F16_TN
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k8.row.col.f16.f16.f16.f16 "
      "{%0, %1},"
      "{%2, %3},"
      "{%4},"
      "{%5, %6};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x8_F16F16F16F16_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x8_F16F16F16F16_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x8_F16F16F16F16_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 90-121
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x16 TN
struct SM80_16x8x16_F16F16F16F16_TN
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
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k16.row.col.f16.f16.f16.f16 "
      "{%0,  %1},"
      "{%2,  %3,  %4,  %5},"
      "{%6,  %7},"
      "{%8,  %9};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x16_F16F16F16F16_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x16_F16F16F16F16_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x16_F16F16F16F16_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 123-154
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x8 TN
struct SM80_16x8x8_F32F16F16F32_TN
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k8.row.col.f32.f16.f16.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x8_F32F16F16F32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x8_F32F16F16F32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x8_F32F16F16F32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 156-187
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x16 TN
struct SM80_16x8x16_F32F16F16F32_TN
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
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k16.row.col.f32.f16.f16.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x16_F32F16F16F32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x16_F32F16F16F32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x16_F32F16F16F32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 189-220
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x8 TN
struct SM80_16x8x8_F32BF16BF16F32_TN
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k8.row.col.f32.bf16.bf16.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x8_F32BF16BF16F32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x8_F32BF16BF16F32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x8_F32BF16BF16F32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 222-253
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x16 TN
struct SM80_16x8x16_F32BF16BF16F32_TN
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
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k16.row.col.f32.bf16.bf16.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x16_F32BF16BF16F32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x16_F32BF16BF16F32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x16_F32BF16BF16F32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 255-286
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x4 TN
struct SM80_16x8x4_F32TF32TF32F32_TN
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float         & d0, float         & d1, float         & d2, float         & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      float const   & c0, float const   & c1, float const   & c2, float const   & c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k4.row.col.f32.tf32.tf32.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x4_F32TF32TF32F32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x4_F32TF32TF32F32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x4_F32TF32TF32F32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 288-319
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x8 TN
struct SM80_16x8x8_F32TF32TF32F32_TN
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
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k8.row.col.f32.tf32.tf32.f32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "f"(c0),  "f"(c1),  "f"(c2),  "f"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x8_F32TF32TF32F32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x8_F32TF32TF32F32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x8_F32TF32TF32F32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 321-352
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x4 TN
struct SM80_8x8x4_F64F64F64F64_TN
{
  using DRegisters = double[2];
  using ARegisters = double[1];
  using BRegisters = double[1];
  using CRegisters = double[2];

  CUTE_HOST_DEVICE static void
  fma(double      & d0, double      & d1,
      double const& a0,
      double const& b0,
      double const& c0, double const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k4.row.col.f64.f64.f64.f64 "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=d"(d0), "=d"(d1)
      :  "d"(a0),
         "d"(b0),
         "d"(c0),  "d"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x4_F64F64F64F64_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x4_F64F64F64F64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x4_F64F64F64F64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 354-402
```cpp
// MMA 8x8x4 TN with Planar Complex multiplication
struct SM80_8x8x4_C64C64C64C64_TN
{
  using DRegisters = complex<double>[2];
  using ARegisters = complex<double>[1];
  using BRegisters = complex<double>[1];
  using CRegisters = complex<double>[2];

  CUTE_HOST_DEVICE static void
  fma(complex<double>      & d0, complex<double>      & d1,
      complex<double> const& a0,
      complex<double> const& b0,
      complex<double> const& c0, complex<double> const& c1)
  {
    // Because thrust::complex does not provide a mutable ref
    double& rd0 = reinterpret_cast<double(&)[2]>(d0)[0];
    double& id0 = reinterpret_cast<double(&)[2]>(d0)[1];
    double& rd1 = reinterpret_cast<double(&)[2]>(d1)[0];
    double& id1 = reinterpret_cast<double(&)[2]>(d1)[1];

    // d.real() =  a.real() * b.real() + c.real();
    SM80_8x8x4_F64F64F64F64_TN::fma(
      rd0, rd1,
      a0.real(),
      b0.real(),
      c0.real(), c1.real());

    // d.imag() =  a.imag() * b.real() + c.imag();
    SM80_8x8x4_F64F64F64F64_TN::fma(
      id0, id1,
      a0.imag(),
      b0.real(),
      c0.imag(), c1.imag());

    // d.real() = -a.imag() * b.imag() + d.real();
    SM80_8x8x4_F64F64F64F64_TN::fma(
      rd0, rd1,
      -a0.imag(),
      b0.imag(),
      d0.real(), d1.real());

    // d.imag() =  a.real() * b.imag() + d.imag();
    SM80_8x8x4_F64F64F64F64_TN::fma(
      id0, id1,
      a0.real(),
      b0.imag(),
      d0.imag(), d1.imag());
  }
};
```
- **EN:** Defines `SM80_8x8x4_C64C64C64C64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x4_C64C64C64C64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 404-456
```cpp
// MMA 8x8x4 TN with Gaussian Complex multiplication:
//    (a + bi)*(c + di)
//  yields
//    t0 += a*c
//    t1 += b*d
//    t2 += (a+b)*(c+d)
//  then
//    re = t0 - t1
//    im = t2 - t0 - t1
struct SM80_8x8x4_GC64C64C64GC64_TN
{
  struct GaussComplex {
    double t0, t1, t2;

    CUTE_HOST_DEVICE //constexpr
    operator complex<double>() const { return complex<double>(t0 - t1, t2 - t0 - t1); }

    CUTE_HOST_DEVICE friend //constexpr
    complex<double> operator*(GaussComplex const& a, complex<double> const& b) { return static_cast<complex<double>>(a) * b; }
    CUTE_HOST_DEVICE friend //constexpr
    complex<double> operator*(complex<double> const& a, GaussComplex const& b) { return b * a; }

    CUTE_HOST_DEVICE friend //constexpr
    complex<double> operator+(GaussComplex const& a, complex<double> const& b) { return static_cast<complex<double>>(a) + b; }
    CUTE_HOST_DEVICE friend //constexpr
    complex<double> operator+(complex<double> const& a, GaussComplex const& b) { return b + a; }
  };

  using DRegisters = GaussComplex[2];
  using ARegisters = complex<double>[1];
  using BRegisters = complex<double>[1];
  using CRegisters = GaussComplex[2];

  CUTE_HOST_DEVICE static void
  fma(GaussComplex         & d0, GaussComplex         & d1,
      complex<double> const& a0,
      complex<double> const& b0,
      GaussComplex    const& c0, GaussComplex    const& c1)
  {
    SM80_8x8x4_F64F64F64F64_TN::fma(d0.t0, d1.t0,
                                    a0.real(),
                                    b0.real(),
                                    c0.t0, c1.t0);
    SM80_8x8x4_F64F64F64F64_TN::fma(d0.t1, d1.t1,
                                    a0.imag(),
                                    b0.imag(),
                                    c0.t1, c1.t1);
    SM80_8x8x4_F64F64F64F64_TN::fma(d0.t2, d1.t2,
                                    a0.real() + a0.imag(),
                                    b0.real() + b0.imag(),
                                    c0.t2, c1.t2);
  }
};
```
- **EN:** Defines `SM80_8x8x4_GC64C64C64GC64_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x4_GC64C64C64GC64_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 458-489
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x16 TN
struct SM80_8x8x16_S32S8S8S32_TN
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[1];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k16.row.col.s32.s8.s8.s32 "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x16_S32S8S8S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x16_S32S8S8S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x16_S32S8S8S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 491-522
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x16 TN
struct SM80_8x8x16_S32S8S8S32_TN_SATURATE
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[1];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k16.row.col.s32.s8.s8.s32.satfinite "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x16_S32S8S8S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x16_S32S8S8S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x16_S32S8S8S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 524-555
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x16 TN
struct SM80_16x8x16_S32S8S8S32_TN
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k16.row.col.s32.s8.s8.s32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x16_S32S8S8S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x16_S32S8S8S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x16_S32S8S8S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 557-588
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x16 TN
struct SM80_16x8x16_S32S8S8S32_TN_SATURATE
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k16.row.col.s32.s8.s8.s32.satfinite "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x16_S32S8S8S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x16_S32S8S8S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x16_S32S8S8S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 590-621
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN
struct SM80_16x8x32_S32S8S8S32_TN
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k32.row.col.s32.s8.s8.s32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x32_S32S8S8S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x32_S32S8S8S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x32_S32S8S8S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 623-654
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN
struct SM80_16x8x32_S32S8S8S32_TN_SATURATE
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k32.row.col.s32.s8.s8.s32.satfinite "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x32_S32S8S8S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x32_S32S8S8S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x32_S32S8S8S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 656-687
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x16 TN
struct SM80_8x8x16_S32S8U8S32_TN
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[1];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k16.row.col.s32.s8.u8.s32 "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x16_S32S8U8S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x16_S32S8U8S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x16_S32S8U8S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 689-720
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x16 TN
struct SM80_8x8x16_S32S8U8S32_TN_SATURATE
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[1];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k16.row.col.s32.s8.u8.s32.satfinite "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x16_S32S8U8S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x16_S32S8U8S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x16_S32S8U8S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 722-753
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x16 TN
struct SM80_16x8x16_S32S8U8S32_TN
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k16.row.col.s32.s8.u8.s32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x16_S32S8U8S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x16_S32S8U8S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x16_S32S8U8S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 755-786
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x16 TN
struct SM80_16x8x16_S32S8U8S32_TN_SATURATE
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k16.row.col.s32.s8.u8.s32.satfinite "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x16_S32S8U8S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x16_S32S8U8S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x16_S32S8U8S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 788-819
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN
struct SM80_16x8x32_S32S8U8S32_TN
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k32.row.col.s32.s8.u8.s32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x32_S32S8U8S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x32_S32S8U8S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x32_S32S8U8S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 821-852
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN
struct SM80_16x8x32_S32S8U8S32_TN_SATURATE
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k32.row.col.s32.s8.u8.s32.satfinite "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x32_S32S8U8S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x32_S32S8U8S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x32_S32S8U8S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 854-885
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x16 TN
struct SM80_8x8x16_S32U8S8S32_TN
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[1];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k16.row.col.s32.u8.s8.s32 "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x16_S32U8S8S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x16_S32U8S8S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x16_S32U8S8S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 887-918
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x16 TN
struct SM80_8x8x16_S32U8S8S32_TN_SATURATE
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[1];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k16.row.col.s32.u8.s8.s32.satfinite "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x16_S32U8S8S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x16_S32U8S8S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x16_S32U8S8S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 920-951
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x16 TN
struct SM80_16x8x16_S32U8S8S32_TN
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k16.row.col.s32.u8.s8.s32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x16_S32U8S8S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x16_S32U8S8S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x16_S32U8S8S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 953-984
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x16 TN
struct SM80_16x8x16_S32U8S8S32_TN_SATURATE
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k16.row.col.s32.u8.s8.s32.satfinite "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x16_S32U8S8S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x16_S32U8S8S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x16_S32U8S8S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 986-1017
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN
struct SM80_16x8x32_S32U8S8S32_TN
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k32.row.col.s32.u8.s8.s32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x32_S32U8S8S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x32_S32U8S8S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x32_S32U8S8S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1019-1050
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN
struct SM80_16x8x32_S32U8S8S32_TN_SATURATE
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k32.row.col.s32.u8.s8.s32.satfinite "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x32_S32U8S8S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x32_S32U8S8S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x32_S32U8S8S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1052-1083
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x16 TN
struct SM80_8x8x16_S32U8U8S32_TN
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[1];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k16.row.col.s32.u8.u8.s32 "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x16_S32U8U8S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x16_S32U8U8S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x16_S32U8U8S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1085-1116
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x16 TN
struct SM80_8x8x16_S32U8U8S32_TN_SATURATE
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[1];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k16.row.col.s32.u8.u8.s32.satfinite "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x16_S32U8U8S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x16_S32U8U8S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x16_S32U8U8S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1118-1149
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x16 TN
struct SM80_16x8x16_S32U8U8S32_TN
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k16.row.col.s32.u8.u8.s32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x16_S32U8U8S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x16_S32U8U8S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x16_S32U8U8S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1151-1182
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x16 TN
struct SM80_16x8x16_S32U8U8S32_TN_SATURATE
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k16.row.col.s32.u8.u8.s32.satfinite "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x16_S32U8U8S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x16_S32U8U8S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x16_S32U8U8S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1184-1215
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN
struct SM80_16x8x32_S32U8U8S32_TN
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k32.row.col.s32.u8.u8.s32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x32_S32U8U8S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x32_S32U8U8S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x32_S32U8U8S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1217-1248
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN
struct SM80_16x8x32_S32U8U8S32_TN_SATURATE
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k32.row.col.s32.u8.u8.s32.satfinite "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x32_S32U8U8S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x32_S32U8U8S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x32_S32U8U8S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1250-1281
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x32 TN
struct SM80_8x8x32_S32S4S4S32_TN
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[1];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k32.row.col.s32.s4.s4.s32 "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x32_S32S4S4S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x32_S32S4S4S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x32_S32S4S4S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1283-1314
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x32 TN
struct SM80_8x8x32_S32S4S4S32_TN_SATURATE
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[1];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k32.row.col.s32.s4.s4.s32.satfinite "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x32_S32S4S4S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x32_S32S4S4S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x32_S32S4S4S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1316-1347
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN
struct SM80_16x8x32_S32S4S4S32_TN
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k32.row.col.s32.s4.s4.s32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x32_S32S4S4S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x32_S32S4S4S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x32_S32S4S4S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1349-1380
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN
struct SM80_16x8x32_S32S4S4S32_TN_SATURATE
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k32.row.col.s32.s4.s4.s32.satfinite "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x32_S32S4S4S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x32_S32S4S4S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x32_S32S4S4S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1382-1413
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN
struct SM80_16x8x64_S32S4S4S32_TN
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k64.row.col.s32.s4.s4.s32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x64_S32S4S4S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x64_S32S4S4S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x64_S32S4S4S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1415-1446
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN
struct SM80_16x8x64_S32S4S4S32_TN_SATURATE
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k64.row.col.s32.s4.s4.s32.satfinite "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x64_S32S4S4S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x64_S32S4S4S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x64_S32S4S4S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1448-1479
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x32 TN
struct SM80_8x8x32_S32S4U4S32_TN
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[1];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k32.row.col.s32.s4.u4.s32 "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x32_S32S4U4S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x32_S32S4U4S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x32_S32S4U4S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1481-1512
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x32 TN
struct SM80_8x8x32_S32S4U4S32_TN_SATURATE
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[1];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k32.row.col.s32.s4.u4.s32.satfinite "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x32_S32S4U4S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x32_S32S4U4S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x32_S32S4U4S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1514-1545
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN
struct SM80_16x8x32_S32S4U4S32_TN
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k32.row.col.s32.s4.u4.s32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x32_S32S4U4S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x32_S32S4U4S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x32_S32S4U4S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1547-1578
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN
struct SM80_16x8x32_S32S4U4S32_TN_SATURATE
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k32.row.col.s32.s4.u4.s32.satfinite "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x32_S32S4U4S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x32_S32S4U4S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x32_S32S4U4S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1580-1611
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN
struct SM80_16x8x64_S32S4U4S32_TN
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k64.row.col.s32.s4.u4.s32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x64_S32S4U4S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x64_S32S4U4S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x64_S32S4U4S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1613-1644
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN
struct SM80_16x8x64_S32S4U4S32_TN_SATURATE
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k64.row.col.s32.s4.u4.s32.satfinite "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x64_S32S4U4S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x64_S32S4U4S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x64_S32S4U4S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1646-1677
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x32 TN
struct SM80_8x8x32_S32U4S4S32_TN
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[1];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k32.row.col.s32.u4.s4.s32 "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x32_S32U4S4S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x32_S32U4S4S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x32_S32U4S4S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1679-1710
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x32 TN
struct SM80_8x8x32_S32U4S4S32_TN_SATURATE
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[1];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k32.row.col.s32.u4.s4.s32.satfinite "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x32_S32U4S4S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x32_S32U4S4S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x32_S32U4S4S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1712-1743
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN
struct SM80_16x8x32_S32U4S4S32_TN
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k32.row.col.s32.u4.s4.s32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x32_S32U4S4S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x32_S32U4S4S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x32_S32U4S4S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1745-1776
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN
struct SM80_16x8x32_S32U4S4S32_TN_SATURATE
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k32.row.col.s32.u4.s4.s32.satfinite "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x32_S32U4S4S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x32_S32U4S4S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x32_S32U4S4S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1778-1809
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN
struct SM80_16x8x64_S32U4S4S32_TN
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k64.row.col.s32.u4.s4.s32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x64_S32U4S4S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x64_S32U4S4S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x64_S32U4S4S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1811-1842
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN
struct SM80_16x8x64_S32U4S4S32_TN_SATURATE
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k64.row.col.s32.u4.s4.s32.satfinite "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x64_S32U4S4S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x64_S32U4S4S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x64_S32U4S4S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1844-1875
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x32 TN
struct SM80_8x8x32_S32U4U4S32_TN
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[1];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k32.row.col.s32.u4.u4.s32 "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x32_S32U4U4S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x32_S32U4U4S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x32_S32U4U4S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1877-1908
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x32 TN
struct SM80_8x8x32_S32U4U4S32_TN_SATURATE
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[1];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k32.row.col.s32.u4.u4.s32.satfinite "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x32_S32U4U4S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x32_S32U4U4S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x32_S32U4U4S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1910-1941
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN
struct SM80_16x8x32_S32U4U4S32_TN
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k32.row.col.s32.u4.u4.s32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x32_S32U4U4S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x32_S32U4U4S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x32_S32U4U4S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1943-1974
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x32 TN
struct SM80_16x8x32_S32U4U4S32_TN_SATURATE
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k32.row.col.s32.u4.u4.s32.satfinite "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x32_S32U4U4S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x32_S32U4U4S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x32_S32U4U4S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1976-2007
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN
struct SM80_16x8x64_S32U4U4S32_TN
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k64.row.col.s32.u4.u4.s32 "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x64_S32U4U4S32_TN without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x64_S32U4U4S32_TN` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x64_S32U4U4S32_TN`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2009-2040
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x64 TN
struct SM80_16x8x64_S32U4U4S32_TN_SATURATE
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k64.row.col.s32.u4.u4.s32.satfinite "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x64_S32U4U4S32_TN_SATURATE without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x64_S32U4U4S32_TN_SATURATE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x64_S32U4U4S32_TN_SATURATE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2042-2073
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x128 TN
struct SM80_8x8x128_S32U1U1S32_TN_ANDPOPC
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[1];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k128.row.col.s32.b1.b1.s32.and.popc "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x128_S32U1U1S32_TN_ANDPOPC without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x128_S32U1U1S32_TN_ANDPOPC` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x128_S32U1U1S32_TN_ANDPOPC`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2075-2106
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x128 TN
struct SM80_16x8x128_S32U1U1S32_TN_ANDPOPC
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k128.row.col.s32.b1.b1.s32.and.popc "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x128_S32U1U1S32_TN_ANDPOPC without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x128_S32U1U1S32_TN_ANDPOPC` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x128_S32U1U1S32_TN_ANDPOPC`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2108-2139
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x256 TN
struct SM80_16x8x256_S32U1U1S32_TN_ANDPOPC
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_B1_AND_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k256.row.col.s32.b1.b1.s32.and.popc "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x256_S32U1U1S32_TN_ANDPOPC without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x256_S32U1U1S32_TN_ANDPOPC` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x256_S32U1U1S32_TN_ANDPOPC`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2141-2172
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 8x8x128 TN
struct SM80_8x8x128_S32U1U1S32_TN_XORPOPC
{
  using DRegisters = uint32_t[2];
  using ARegisters = uint32_t[1];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1,
      uint32_t const& a0,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1)
  {
#if defined(CUTE_ARCH_MMA_B1_XOR_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m8n8k128.row.col.s32.b1.b1.s32.xor.popc "
      "{%0, %1},"
      "{%2},"
      "{%3},"
      "{%4, %5};\n"
      : "=r"(d0), "=r"(d1)
      :  "r"(a0),
         "r"(b0),
         "r"(c0),  "r"(c1));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_8x8x128_S32U1U1S32_TN_XORPOPC without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_8x8x128_S32U1U1S32_TN_XORPOPC` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_8x8x128_S32U1U1S32_TN_XORPOPC`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2174-2205
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x128 TN
struct SM80_16x8x128_S32U1U1S32_TN_XORPOPC
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[2];
  using BRegisters = uint32_t[1];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1,
      uint32_t const& b0,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_B1_XOR_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k128.row.col.s32.b1.b1.s32.xor.popc "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5},"
      "{%6},"
      "{%7,  %8,  %9,  %10};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),
         "r"(b0),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x128_S32U1U1S32_TN_XORPOPC without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x128_S32U1U1S32_TN_XORPOPC` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x128_S32U1U1S32_TN_XORPOPC`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2207-2238
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

// MMA 16x8x256 TN
struct SM80_16x8x256_S32U1U1S32_TN_XORPOPC
{
  using DRegisters = uint32_t[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  fma(uint32_t      & d0, uint32_t      & d1, uint32_t      & d2, uint32_t      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      uint32_t const& c0, uint32_t const& c1, uint32_t const& c2, uint32_t const& c3)
  {
#if defined(CUTE_ARCH_MMA_B1_XOR_SM80_ENABLED)
    asm volatile(
      "mma.sync.aligned.m16n8k256.row.col.s32.b1.b1.s32.xor.popc "
      "{%0,  %1,  %2,  %3},"
      "{%4,  %5,  %6,  %7},"
      "{%8,  %9},"
      "{%10, %11, %12, %13};\n"
      : "=r"(d0), "=r"(d1), "=r"(d2), "=r"(d3)
      :  "r"(a0),  "r"(a1),  "r"(a2),  "r"(a3),
         "r"(b0),  "r"(b1),
         "r"(c0),  "r"(c1),  "r"(c2),  "r"(c3));
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM80_16x8x256_S32U1U1S32_TN_XORPOPC without CUTE_ARCH_MMA_SM80_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM80_16x8x256_S32U1U1S32_TN_XORPOPC` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM80_16x8x256_S32U1U1S32_TN_XORPOPC`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 2240-2242
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

} // namespace cute
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

## Key Concepts / 关键概念

- **EN:** MMA/GMMA/UMMA wrappers expose tensor-core style matrix instructions as typed C++ interfaces.
  **CN:** MMA/GMMA/UMMA 封装把张量核心式矩阵指令暴露为带类型的 C++ 接口。
- **EN:** Inline assembly keeps the mapping between C++ fragments and hardware registers explicit.
  **CN:** 内联汇编使 C++ 片段与硬件寄存器之间的映射保持显式。
- **EN:** The Xe/SYCL path bridges CuTe abstractions to Intel GPU builtins or SPIR-V operations.
  **CN:** Xe/SYCL 路径把 CuTe 抽象桥接到 Intel GPU 内建函数或 SPIR-V 操作。

## Dependencies / 依赖关系

- **EN:** `cute/config.hpp` supplies core CuTe configuration macros and portability annotations.
  **CN:** `cute/config.hpp` 提供了CuTe 核心配置宏与可移植性标注。
- **EN:** `cute/arch/mma.hpp` supplies generic low-level MMA operation tags and wrappers.
  **CN:** `cute/arch/mma.hpp` 提供了通用底层 MMA 操作标签与封装。
- **EN:** `cute/numeric/complex.hpp` supplies related definitions from `cute/numeric/complex.hpp`.
  **CN:** `cute/numeric/complex.hpp` 提供了来自 `cute/numeric/complex.hpp` 的相关定义。
- **EN:** CUDA architecture macros gate device-only fast paths and inline instructions.
  **CN:** CUDA 架构宏用于控制仅设备端可用的快速路径与内联指令。
