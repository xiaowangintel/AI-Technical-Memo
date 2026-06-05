# mma_sm100_umma.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/mma_sm100_umma.hpp`
- **EN:** Defines UMMA instruction wrappers and descriptors for NVIDIA SM100.
- **CN:** 为 NVIDIA SM100 定义 UMMA 指令封装与描述符。

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

### Lines 34-38
```cpp
#include <cute/arch/config.hpp>
#include <cute/arch/mma.hpp>
#include <cute/arch/mma_sm100.hpp>
#include <cute/arch/mma_sm100_desc.hpp>
#include <cute/arch/cluster_sm90.hpp>
```
- **EN:** Imports `cute/arch/config.hpp` (related definitions from `cute/arch/config.hpp`); `cute/arch/mma.hpp` (generic low-level MMA operation tags and wrappers); `cute/arch/mma_sm100.hpp` (related definitions from `cute/arch/mma_sm100.hpp`); `cute/arch/mma_sm100_desc.hpp` (related definitions from `cute/arch/mma_sm100_desc.hpp`); `cute/arch/cluster_sm90.hpp` (related definitions from `cute/arch/cluster_sm90.hpp`).
- **CN:** 引入 `cute/arch/config.hpp`（来自 `cute/arch/config.hpp` 的相关定义）；`cute/arch/mma.hpp`（通用底层 MMA 操作标签与封装）；`cute/arch/mma_sm100.hpp`（来自 `cute/arch/mma_sm100.hpp` 的相关定义）；`cute/arch/mma_sm100_desc.hpp`（来自 `cute/arch/mma_sm100_desc.hpp` 的相关定义）；`cute/arch/cluster_sm90.hpp`（来自 `cute/arch/cluster_sm90.hpp` 的相关定义）。

### Lines 40-41
```cpp
namespace cute
{
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 43-81
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_TF32_SS
{
  static_assert(M == 64 || M == 128, "SM100_MMA_TF32 M-mode size should be 64 or 128 for 1 CTA cluster MMA.");
  static_assert((N % 8 == 0) && (8 <= N) && (N <= 256),
                "SM100_MMA_TF32 N-mode size should be a multiple of 8 between 8 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_TF32_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[4] = {0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::1.kind::tf32 [%0], %1, %2, %3, {%5, %6, %7, %8}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_TF32_SS without CUTE_ARCH_TCGEN05_TF32_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_TF32_SS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_TF32_SS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 83-121
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_F16BF16_SS
{
  static_assert(M == 64 || M == 128, "SM100_MMA_F16BF16 M-mode size should be 64 or 128 for 1 CTA cluster MMA.");
  static_assert((N % 8 == 0)  && (8 <= N)  && (N <= 256),
                "SM100_MMA_F16BF16 N-mode size should be a multiple of 8 between 8 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_F16F32_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[4] = {0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::1.kind::f16 [%0], %1, %2, %3, {%5, %6, %7, %8}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_F16BF16_SS without CUTE_ARCH_MMA_SM100A_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_F16BF16_SS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_F16BF16_SS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 123-165
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One,
          UMMA::Saturate c_sat = UMMA::Saturate::False>
struct SM100_MMA_TF32_TS
{
  static_assert(M == 64 || M == 128, "SM100_MMA_TF32 M-mode size should be 64 or 128 for 1 CTA cluster MMA.");
  static_assert((M == 64  && (N % 8 == 0)  && (8 <= N)  && (N <= 256)) ||
                (M == 128 && (N % 16 == 0) && (16 <= N) && (N <= 256)),
                "SM100_MMA_TF32 N-mode size should be a multiple of 8 between 8 and 256 for M=64,\
                 or a multiple of 16 between 16 and 256 for M=128.");
  static_assert(a_major == UMMA::Major::K, "SM100_MMA_TF32 A from TMEM can't be transposed");

  using DRegisters = void;
  using ARegisters = uint32_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t const& tmem_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_TF32_MMA_ENABLED)
    uint32_t mask[4] = {0, 0, 0, 0};
    if (cute::elect_one_sync()) {
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::1.kind::tf32 [%0], [%1], %2, %3, {%5, %6, %7, %8}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "r"(tmem_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_TF32_TS without CUTE_ARCH_TCGEN05_TF32_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_TF32_TS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_TF32_TS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 167-209
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One,
          UMMA::Saturate c_sat = UMMA::Saturate::False>
struct SM100_MMA_F16BF16_TS
{
  static_assert(M == 64 || M == 128, "SM100_MMA_F16BF16 M-mode size should be 64 or 128 for 1 CTA cluster MMA.");
  static_assert((M == 64  && (N % 8 == 0)  && (8 <= N)  && (N <= 256)) ||
                (M == 128 && (N % 16 == 0) && (16 <= N) && (N <= 256)),
                "SM100_MMA_F16BF16 N-mode size should be a multiple of 8 between 8 and 256 for M=64,\
                 or a multiple of 16 between 16 and 256 for M=128.");
  static_assert(a_major == UMMA::Major::K, "SM100_MMA_F16BF16 A from TMEM can't be transposed");

  using DRegisters = void;
  using ARegisters = uint32_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t const& tmem_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_F16F32_MMA_ENABLED)
    uint32_t mask[4] = {0, 0, 0, 0};
    if (cute::elect_one_sync()) {
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::1.kind::f16 [%0], [%1], %2, %3, {%5, %6, %7, %8}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "r"(tmem_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_F16BF16_TS without CUTE_ARCH_MMA_SM100A_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_F16BF16_TS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_F16BF16_TS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 211-250
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major, uint32_t ScaleC,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_F16BF16_SS_SCALED
{
  static_assert(M == 64 || M == 128, "SM100_MMA_F16BF16_SS_SCALED M-mode size should be 64 or 128 for 1 CTA cluster MMA.");
  static_assert((N % 8 == 0) && (8 <= N) && (N <= 256),
                "SM100_MMA_F16BF16_SS_SCALED N-mode size should be a multiple of 8 between 8 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& accumulate,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_F16BF16_MMA_SCALED_ENABLED)
    if (cute::elect_one_sync()) {
      // ScaleC input should be a literal or compile time constant
      uint32_t mask[4] = {0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::1.kind::f16 [%0], %1, %2, %3, {%5, %6, %7, %8}, p, %9; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(accumulate),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]), "n"(ScaleC));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_F16BF16_SS without CUTE_ARCH_TCGEN05_F16BF16_MMA_SCALED_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_F16BF16_SS_SCALED` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_F16BF16_SS_SCALED`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 252-295
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major, uint32_t ScaleC,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One,
          UMMA::Saturate c_sat = UMMA::Saturate::False>
struct SM100_MMA_F16BF16_TS_SCALED
{
  static_assert(M == 64 || M == 128, "SM100_MMA_F16BF16_TS_SCALED M-mode size should be 64 or 128 for 1 CTA cluster MMA.");
  static_assert((M == 64  && (N % 8 == 0)  && (8 <= N)  && (N <= 256)) ||
                (M == 128 && (N % 16 == 0) && (16 <= N) && (N <= 256)),
                "SM100_MMA_F16BF16_TS_SCALED N-mode size should be a multiple of 8 between 8 and 256 for M=64,\
                 or a multiple of 16 between 16 and 256 for M=128.");
  static_assert(a_major == UMMA::Major::K, "SM100_MMA_F16BF16_TS_SCALED A from TMEM can't be transposed");

  using DRegisters = void;
  using ARegisters = uint32_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t const& tmem_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& accumulate,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_F16BF16_MMA_SCALED_ENABLED)
    if (cute::elect_one_sync()) {
      // ScaleC input should be a literal or compile time constant
      uint32_t mask[4] = {0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::1.kind::f16 [%0], [%1], %2, %3, {%5, %6, %7, %8}, p, %9; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "r"(tmem_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(accumulate),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]), "n"(ScaleC));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_F16BF16_TS_SCALED without CUTE_ARCH_TCGEN05_F16BF16_MMA_SCALED_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_F16BF16_TS_SCALED` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_F16BF16_TS_SCALED`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 297-336
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_TF32_SS_SPARSE
{
  static_assert(M == 64 || M == 128, "SM100_MMA_TF32_SS_SPARSE M-mode size should be 64 or 128 for 1 CTA cluster MMA.");
  static_assert((N % 8 == 0) && (8 <= N) && (N <= 256),
                "SM100_MMA_TF32_SS_SPARSE N-mode size should be a multiple of 8 between 8 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE,
      uint32_t const& tmem_e)
  {
#if defined(CUTE_ARCH_TCGEN05_TF32_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[4] = {0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.sp.cta_group::1.kind::tf32 [%0], %1, %2, [%9], %3, {%5, %6, %7, %8}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]), "r"(tmem_e));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_TF32_SS_SPARSE without CUTE_ARCH_TCGEN05_TF32_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_TF32_SS_SPARSE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_TF32_SS_SPARSE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 338-377
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_F16BF16_SS_SPARSE
{
  static_assert(M == 64 || M == 128, "SM100_MMA_F16BF16_SS_SPARSE M-mode size should be 64 or 128 for 1 CTA cluster MMA.");
  static_assert((N % 8 == 0) && (8 <= N) && (N <= 256),
                "SM100_MMA_F16BF16_SS_SPARSE N-mode size should be a multiple of 8 between 8 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE,
      uint32_t const& tmem_e)
  {
#if defined(CUTE_ARCH_TCGEN05_F16F32_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[4] = {0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.sp.cta_group::1.kind::f16 [%0], %1, %2, [%9], %3, {%5, %6, %7, %8}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]), "r"(tmem_e));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_F16BF16_SS_SPARSE without CUTE_ARCH_TCGEN05_F16F32_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_F16BF16_SS_SPARSE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_F16BF16_SS_SPARSE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 379-417
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_TF32_2x1SM_SS
{
  static_assert(M == 128 || M == 256, "SM100_MMA_TF32_2x1SM_SS M-mode size should be 128 or 256 for 2 CTA cluster MMA.");
  static_assert((N % 16 == 0) && (16 <= N) && (N <= 256), "SM100_MMA_TF32_2x1SM_SS N-mode size should be a multiple of 16 between 16 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_TF32_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[8] = {0, 0, 0, 0, 0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::2.kind::tf32 [%0], %1, %2, %3, {%5, %6, %7, %8, %9, %10, %11, %12}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]),
          "r"(mask[4]), "r"(mask[5]), "r"(mask[6]), "r"(mask[7]));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_TF32_2x1SM_SS without CUTE_ARCH_TCGEN05_TF32_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_TF32_2x1SM_SS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_TF32_2x1SM_SS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 419-457
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_F16BF16_2x1SM_SS
{
  static_assert(M == 128 || M == 256, "SM100_MMA_F16BF16_2x1SM_SS M-mode size should be 128 or 256 for 2 CTA cluster MMA.");
  static_assert((N % 16 == 0) && (16 <= N) && (N <= 256), "SM100_MMA_F16BF16_2x1SM_SS N-mode size should be a multiple of 16 between 16 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_F16F32_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[8] = {0, 0, 0, 0, 0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::2.kind::f16 [%0], %1, %2, %3, {%5, %6, %7, %8, %9, %10, %11, %12}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]),
          "r"(mask[4]), "r"(mask[5]), "r"(mask[6]), "r"(mask[7]));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_F16BF16_2x1SM_SS without CUTE_ARCH_TCGEN05_F16F32_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_F16BF16_2x1SM_SS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_F16BF16_2x1SM_SS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 459-499
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One,
          UMMA::Saturate c_sat = UMMA::Saturate::False>
struct SM100_MMA_TF32_2x1SM_TS
{
  static_assert(M == 128 || M == 256, "SM100_MMA_TF32_2x1SM_TS M-mode size should be 128 or 256 for 2 CTA cluster MMA.");
  static_assert((N % 32 == 0) && (32 <= N) && (N <= 256), "SM100_MMA_TF32_2x1SM_TS N-mode size should be a multiple of 32 between 32 and 256.");
  static_assert(a_major == UMMA::Major::K, "SM100_MMA_TF32_2x1SM_TS A from TMEM can't be transposed");

  using DRegisters = void;
  using ARegisters = uint32_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t const& tmem_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_TF32_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[8] = {0, 0, 0, 0, 0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::2.kind::tf32 [%0], [%1], %2, %3, {%5, %6, %7, %8, %9, %10, %11, %12}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "r"(tmem_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]),
          "r"(mask[4]), "r"(mask[5]), "r"(mask[6]), "r"(mask[7]));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_TF32_2x1SM_TS without CUTE_ARCH_TCGEN05_TF32_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_TF32_2x1SM_TS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_TF32_2x1SM_TS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 501-541
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One,
          UMMA::Saturate c_sat = UMMA::Saturate::False>
struct SM100_MMA_F16BF16_2x1SM_TS
{
  static_assert(M == 128 || M == 256, "SM100_MMA_F16BF16_2x1SM_TS M-mode size should be 128 or 256 for 2 CTA cluster MMA.");
  static_assert((N % 32 == 0) && (32 <= N) && (N <= 256), "SM100_MMA_F16BF16_2x1SM_TS N-mode size should be a multiple of 32 between 32 and 256.");
  static_assert(a_major == UMMA::Major::K, "SM100_MMA_F16BF16_2x1SM_TS A from TMEM can't be transposed");

  using DRegisters = void;
  using ARegisters = uint32_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t const& tmem_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_F16F32_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[8] = {0, 0, 0, 0, 0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::2.kind::f16 [%0], [%1], %2, %3, {%5, %6, %7, %8, %9, %10, %11, %12}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "r"(tmem_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]),
          "r"(mask[4]), "r"(mask[5]), "r"(mask[6]), "r"(mask[7]));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_F16BF16_2x1SM_TS without CUTE_ARCH_TCGEN05_F16F32_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_F16BF16_2x1SM_TS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_F16BF16_2x1SM_TS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 543-582
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major, uint32_t ScaleC,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_F16BF16_2x1SM_SS_SCALED
{
  static_assert(M == 128 || M == 256, "SM100_MMA_F16BF16_2x1SM_SS_SCALED M-mode size should be 128 or 256 for 2 CTA cluster MMA.");
  static_assert((N % 16 == 0) && (16 <= N) && (N <= 256), "SM100_MMA_F16BF16_2x1SM_SS_SCALED N-mode size should be a multiple of 16 between 16 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& accumulate,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_F16BF16_MMA_SCALED_ENABLED)
    if (cute::elect_one_sync()) {
      // ScaleC input should be a literal or compile time constant
      uint32_t mask[8] = {0, 0, 0, 0, 0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::2.kind::f16 [%0], %1, %2, %3, {%5, %6, %7, %8, %9, %10, %11, %12}, p, %13; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(accumulate),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]),
          "r"(mask[4]), "r"(mask[5]), "r"(mask[6]), "r"(mask[7]), "n"(ScaleC));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_F16BF16_2x1SM_SS_SCALED without CUTE_ARCH_TCGEN05_F16BF16_MMA_SCALED_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_F16BF16_2x1SM_SS_SCALED` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_F16BF16_2x1SM_SS_SCALED`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 584-625
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major, uint32_t ScaleC,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One,
          UMMA::Saturate c_sat = UMMA::Saturate::False>
struct SM100_MMA_F16BF16_2x1SM_TS_SCALED
{
  static_assert(M == 128 || M == 256, "SM100_MMA_F16BF16_2x1SM_TS_SCALED M-mode size should be 128 or 256 for 2 CTA cluster MMA.");
  static_assert((N % 32 == 0) && (32 <= N) && (N <= 256), "SM100_MMA_F16BF16_2x1SM_TS_SCALED N-mode size should be a multiple of 32 between 32 and 256.");
  static_assert(a_major == UMMA::Major::K, "SM100_MMA_F16BF16_2x1SM_TS_SCALED A from TMEM can't be transposed");

  using DRegisters = void;
  using ARegisters = uint32_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t const& tmem_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& accumulate,
      uint64_t idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_F16BF16_MMA_SCALED_ENABLED)
    if (cute::elect_one_sync()) {
      // ScaleC input should be a literal or compile time constant
      uint32_t mask[8] = {0, 0, 0, 0, 0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::2.kind::f16 [%0], [%1], %2, %3, {%5, %6, %7, %8, %9, %10, %11, %12}, p, %13; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "r"(tmem_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(accumulate),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]),
          "r"(mask[4]), "r"(mask[5]), "r"(mask[6]), "r"(mask[7]), "n"(ScaleC));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_F16BF16_2x1SM_TS_SCALED without CUTE_ARCH_TCGEN05_F16BF16_MMA_SCALED_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_F16BF16_2x1SM_TS_SCALED` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_F16BF16_2x1SM_TS_SCALED`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 627-666
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_TF32_2x1SM_SS_SPARSE
{
  static_assert(M == 128 || M == 256, "SM100_MMA_TF32_2x1SM_SS_SPARSE M-mode size should be 128 or 256 for 2 CTA cluster MMA.");
  static_assert((N % 16 == 0) && (16 <= N) && (N <= 256), "SM100_MMA_TF32_2x1SM_SS_SPARSE N-mode size should be a multiple of 16 between 16 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE,
      uint32_t const& tmem_e)
  {
#if defined(CUTE_ARCH_TCGEN05_TF32_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[8] = {0, 0, 0, 0, 0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.sp.cta_group::2.kind::tf32 [%0], %1, %2, [%13], %3, {%5, %6, %7, %8, %9, %10, %11, %12}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]),
          "r"(mask[4]), "r"(mask[5]), "r"(mask[6]), "r"(mask[7]), "r"(tmem_e));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_TF32_2x1SM_SS_SPARSE without CUTE_ARCH_TCGEN05_TF32_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_TF32_2x1SM_SS_SPARSE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_TF32_2x1SM_SS_SPARSE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 668-707
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_F16BF16_2x1SM_SS_SPARSE
{
  static_assert(M == 128 || M == 256, "SM100_MMA_F16BF16_2x1SM_SS_SPARSE M-mode size should be 128 or 256 for 2 CTA cluster MMA.");
  static_assert((N % 16 == 0) && (16 <= N) && (N <= 256), "SM100_MMA_F16BF16_2x1SM_SS_SPARSE N-mode size should be a multiple of 16 between 16 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE,
      uint32_t const& tmem_e)
  {
#if defined(CUTE_ARCH_TCGEN05_F16F32_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[8] = {0, 0, 0, 0, 0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.sp.cta_group::2.kind::f16 [%0], %1, %2, [%13], %3, {%5, %6, %7, %8, %9, %10, %11, %12}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]),
          "r"(mask[4]), "r"(mask[5]), "r"(mask[6]), "r"(mask[7]), "r"(tmem_e));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_F16BF16_2x1SM_SS_SPARSE without CUTE_ARCH_TCGEN05_F16F32_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_F16BF16_2x1SM_SS_SPARSE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_F16BF16_2x1SM_SS_SPARSE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 709-747
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::Saturate c_sat = UMMA::Saturate::False>
struct SM100_MMA_S8_SS
{
  static_assert(is_same_v<c_type, int32_t>, "SM100_MMA_S8_SS result type can only be int32_t.");
  static_assert(M == 64 || M == 128, "SM100_MMA_S8_SS M-mode size should be 64 or 128 for 1 CTA cluster MMA.");
  static_assert(N == 8 || ((N % 16 == 0) && (16 <= N) && (N <= 256)), "SM100_MMA_S8_SS N-mode size should be 8 or a multiple of 16 between 16 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_S8_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[4] = {0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::1.kind::i8 [%0], %1, %2, %3, {%5, %6, %7, %8}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_S8_SS without CUTE_ARCH_TCGEN05_S8_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_S8_SS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_S8_SS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 749-788
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One,
          UMMA::Saturate c_sat = UMMA::Saturate::False>
struct SM100_MMA_S8_TS
{
  static_assert(M == 64 || M == 128, "SM100_MMA_S8_TS M-mode size should be 64 or 128 for 1 CTA cluster MMA.");
  static_assert(N == 8 || ((N % 16 == 0) && (16 <= N) && (N <= 256)), "SM100_MMA_S8_TS N-mode size should be 8 or a multiple of 16 between 16 and 256.");
  static_assert(a_major == UMMA::Major::K, "SM100_MMA_S8_TS A from TMEM can't be transposed");

  using DRegisters = void;
  using ARegisters = uint32_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t const& tmem_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_S8_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[4] = {0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::1.kind::i8 [%0], [%1], %2, %3, {%5, %6, %7, %8}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "r"(tmem_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_S8_TS without CUTE_ARCH_TCGEN05_S8_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_S8_TS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_S8_TS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 790-829
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::Saturate c_sat = UMMA::Saturate::False>
struct SM100_MMA_S8_SS_SPARSE
{
  static_assert(is_same_v<c_type, int32_t>, "SM100_MMA_S8_SS_SPARSE result type can only be int32_t.");
  static_assert(M == 64 || M == 128, "SM100_MMA_S8_SS_SPARSE M-mode size should be 64 or 128 for 1 CTA cluster MMA.");
  static_assert(N == 8 || ((N % 16 == 0) && (16 <= N) && (N <= 256)), "SM100_MMA_S8_SS_SPARSE N-mode size should be 8 or a multiple of 16 between 16 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE,
      uint32_t const& tmem_e)
  {
#if defined(CUTE_ARCH_TCGEN05_S8_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[4] = {0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.sp.cta_group::1.kind::i8 [%0], %1, %2, [%9], %3, {%5, %6, %7, %8}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]), "r"(tmem_e));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_S8_SS_SPARSE without CUTE_ARCH_TCGEN05_S8_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_S8_SS_SPARSE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_S8_SS_SPARSE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 831-869
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::Saturate c_sat = UMMA::Saturate::False>
struct SM100_MMA_S8_2x1SM_SS
{
  static_assert(M == 128 || M == 256, "SM100_MMA_S8_2x1SM_SS M-mode size should be 128 or 256 for 2 CTA cluster MMA.");
  static_assert((N % 32 == 0) && (32 <= N) && (N <= 256), "SM100_MMA_S8_2x1SM_SS N-mode size should be a multiple of 32 between 32 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_S8_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[8] = {0, 0, 0, 0, 0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::2.kind::i8 [%0], %1, %2, %3, {%5, %6, %7, %8, %9, %10, %11, %12}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]),
          "r"(mask[4]), "r"(mask[5]), "r"(mask[6]), "r"(mask[7]));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_S8_2x1SM_SS without CUTE_ARCH_TCGEN05_S8_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_S8_2x1SM_SS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_S8_2x1SM_SS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 871-911
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One,
          UMMA::Saturate c_sat = UMMA::Saturate::False>
struct SM100_MMA_S8_2x1SM_TS
{
  static_assert(M == 128 || M == 256, "SM100_MMA_S8_2x1SM_TS M-mode size should be 128 or 256 for 2 CTA cluster MMA.");
  static_assert((N % 32 == 0) && (32 <= N) && (N <= 256), "SM100_MMA_S8_2x1SM_TS N-mode size should be a multiple of 32 between 32 and 256.");
  static_assert(a_major == UMMA::Major::K, "SM100_MMA_S8_2x1SM_TS A from TMEM can't be transposed");

  using DRegisters = void;
  using ARegisters = uint32_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t const& tmem_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_S8_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[8] = {0, 0, 0, 0, 0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::2.kind::i8 [%0], [%1], %2, %3, {%5, %6, %7, %8, %9, %10, %11, %12}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "r"(tmem_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]),
          "r"(mask[4]), "r"(mask[5]), "r"(mask[6]), "r"(mask[7]));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_S8_2x1SM_TS without CUTE_ARCH_TCGEN05_S8_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_S8_2x1SM_TS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_S8_2x1SM_TS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 913-952
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::Saturate c_sat = UMMA::Saturate::False>
struct SM100_MMA_S8_2x1SM_SS_SPARSE
{
  static_assert(M == 128 || M == 256, "SM100_MMA_S8 M-mode size should be 128 or 256 for 2 CTA cluster MMA.");
  static_assert((N % 32 == 0) && (32 <= N) && (N <= 256), "SM100_MMA_S8 N-mode size should be a multiple of 32 between 32 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE,
      uint32_t const& tmem_e)
  {
#if defined(CUTE_ARCH_TCGEN05_S8_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[8] = {0, 0, 0, 0, 0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.sp.cta_group::2.kind::i8 [%0], %1, %2, [%13], %3, {%5, %6, %7, %8, %9, %10, %11, %12}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]),
          "r"(mask[4]), "r"(mask[5]), "r"(mask[6]), "r"(mask[7]), "r"(tmem_e));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_S8_2x1SM_SS_SPARSE without CUTE_ARCH_TCGEN05_S8_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_S8_2x1SM_SS_SPARSE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_S8_2x1SM_SS_SPARSE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 954-985
```cpp
struct SM100_MMA_F8F6F4_SS
{
  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[4] = {0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::1.kind::f8f6f4 [%0], %1, %2, %3, {%5, %6, %7, %8}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_F8F6F4_SS without CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_F8F6F4_SS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_F8F6F4_SS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 987-1026
```cpp
template <class a_type, class b_type, class c_type, class sf_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_MXF8F6F4_SS
{
  static_assert(M == 128, "SM100_MMA_MXF8F6F4_SS M-mode size should be 64 or 128 for 1 CTA cluster MMA.");
  static_assert((N % 8 == 0) && (8 <= N) && (N <= 256), "SM100_MMA_MXF8F6F4_SS N-mode size should be a multiple of 8 between 8 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];
  using SFARegisters = uint32_t[1];
  using SFBRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE,
      uint32_t const& tsfa_addr,
      uint32_t const& tsfb_addr)
  {
#if defined(CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::1.kind::mxf8f6f4.block_scale [%0], %1, %2, %3, [%5], [%6], p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC), "r"(tsfa_addr), "r"(tsfb_addr));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_F8F6F4_SS without CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_MXF8F6F4_SS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_MXF8F6F4_SS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1028-1070
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One,
          UMMA::Saturate c_sat = UMMA::Saturate::False>
struct SM100_MMA_F8F6F4_TS
{
  static_assert(M == 64 || M == 128, "SM100_MMA_F8F6F4_TS M-mode size should be 64 or 128 for 1 CTA cluster MMA.");
  static_assert((M == 64  && (N % 8 == 0)  && (8 <= N)  && (N <= 256)) ||
                (M == 128 && (N % 16 == 0) && (16 <= N) && (N <= 256)),
                "SM100_MMA_F8F6F4_TS N-mode size should be a multiple of 8 between 8 and 256 for M=64,\
                 or a multiple of 16 between 16 and 256 for M=128.");
  static_assert(a_major == UMMA::Major::K, "SM100_MMA_F8F6F4_TS A from TMEM can't be transposed");

  using DRegisters = void;
  using ARegisters = uint32_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t const& tmem_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[4] = {0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::1.kind::f8f6f4 [%0], [%1], %2, %3, {%5, %6, %7, %8}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "r"(tmem_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_F8F6F4_TS without CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_F8F6F4_TS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_F8F6F4_TS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1072-1112
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One,
          UMMA::Saturate c_sat = UMMA::Saturate::False>
struct SM100_MMA_F8F6F4_2x1SM_TS
{
  static_assert(M == 128 || M == 256, "SM100_MMA_F8F6F4_2x1SM_TS M-mode size should be 64 or 128 for 1 CTA cluster MMA.");
  static_assert((N % 32 == 0) && (32 <= N) && (N <= 256), "SM100_MMA_F8F6F4_2x1SM_TS N-mode size should be a multiple of 32 between 32 and 256.");
  static_assert(a_major == UMMA::Major::K, "SM100_MMA_F8F6F4_2x1SM_TS A from TMEM can't be transposed");

  using DRegisters = void;
  using ARegisters = uint32_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint32_t const& tmem_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[8] = {0, 0, 0, 0, 0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::2.kind::f8f6f4 [%0], [%1], %2, %3, {%5, %6, %7, %8, %9, %10, %11, %12}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "r"(tmem_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]),
          "r"(mask[4]), "r"(mask[5]), "r"(mask[6]), "r"(mask[7]));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_F8F6F4_TS without CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_F8F6F4_2x1SM_TS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_F8F6F4_2x1SM_TS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1114-1155
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_F8F6F4_SS_SPARSE
{
  static_assert(M == 64 || M == 128, "SM100_MMA_F8F6F4_SS_SPARSE M-mode size should be 64 or 128 for 1 CTA cluster MMA.");
  static_assert((M == 64  && (N % 8 == 0)  && (8 <= N)  && (N <= 256)) ||
                (M == 128 && (N % 16 == 0) && (16 <= N) && (N <= 256)),
                "SM100_MMA_F8F6F4_SS_SPARSE N-mode size should be a multiple of 8 between 8 and 256 for M=64,\
                 or a multiple of 16 between 16 and 256 for M=128.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE,
      uint32_t const& tmem_e)
  {
#if defined(CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[4] = {0, 0, 0, 0}; // %5, %6, %7, %8
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.sp.cta_group::1.kind::f8f6f4 [%0], %1, %2, [%9], %3, {%5, %6, %7, %8}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]), "r"(tmem_e));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_F8F6F4_SS_SPARSE without CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_F8F6F4_SS_SPARSE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_F8F6F4_SS_SPARSE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1157-1197
```cpp
template <class a_type, class b_type, class c_type, class sf_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_MXF8F6F4_SS_SPARSE
{
  static_assert(M == 128, "SM100_MMA_MXF8F6F4_SS_SPARSE M-mode size should be 128 for 1 CTA cluster MMA.");
  static_assert((N % 8 == 0) && (8 <= N) && (N <= 256), "SM100_MMA_MXF8F6F4_SS_SPARSE N-mode size should be a multiple of 8 between 8 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];
  using SFARegisters = uint32_t[1];
  using SFBRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE,
      uint32_t const& tsfa_addr,
      uint32_t const& tsfb_addr,
      uint32_t const& tmem_e)
  {
#if defined(CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.sp.cta_group::1.kind::mxf8f6f4.block_scale [%0], %1, %2, [%7], %3, [%5], [%6], p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC), "r"(tsfa_addr), "r"(tsfb_addr), "r"(tmem_e));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_MXF8F6F4_SS_SPARSE without CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_MXF8F6F4_SS_SPARSE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_MXF8F6F4_SS_SPARSE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1199-1231
```cpp
struct SM100_MMA_F8F6F4_2x1SM_SS
{  
  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE)
  {
#if defined(CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[8] = {0, 0, 0, 0, 0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::2.kind::f8f6f4 [%0], %1, %2, %3, {%5, %6, %7, %8, %9, %10, %11, %12}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]),
          "r"(mask[4]), "r"(mask[5]), "r"(mask[6]), "r"(mask[7]));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_F8F6F4_2x1SM_SS without CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_F8F6F4_2x1SM_SS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_F8F6F4_2x1SM_SS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1233-1272
```cpp
template <class a_type, class b_type, class c_type, class sf_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_MXF8F6F4_2x1SM_SS_SPARSE
{
  static_assert(M == 256, "SM100_MMA_MXF8F6F4_2x1SM_SS_SPARSE M-mode size should be 256 for 2 CTA cluster MMA.");
  static_assert((N % 16 == 0) && (16 <= N) && (N <= 256), "SM100_MMA_MXF8F6F4_2x1SM_SS_SPARSE N-mode size should be a multiple of 16 between 16 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE,
      uint32_t const& tsfa_addr,
      uint32_t const& tsfb_addr,
      uint32_t const& tmem_e)
  {
#if defined(CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.sp.cta_group::2.kind::mxf8f6f4.block_scale [%0], %1, %2, [%7], %3, [%5], [%6], p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(tsfa_addr), "r"(tsfb_addr), "r"(tmem_e));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_MXF8F6F4_2x1SM_SS_SPARSE without CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_MXF8F6F4_2x1SM_SS_SPARSE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_MXF8F6F4_2x1SM_SS_SPARSE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1274-1312
```cpp
template <class a_type, class b_type, class c_type, class sf_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_MXF8F6F4_2x1SM_SS
{
  static_assert(M == 256, "SM100_MMA_MXF8F6F4_2x1SM_SS M-mode size should be 128 or 256 for 2 CTA cluster MMA.");
  static_assert((N % 16 == 0) && (16 <= N) && (N <= 256), "SM100_MMA_MXF8F6F4_2x1SM_SS N-mode size should be a multiple of 16 between 16 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE,
      uint32_t const& tsfa_addr,
      uint32_t const& tsfb_addr)
  {
#if defined(CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.cta_group::2.kind::mxf8f6f4.block_scale [%0], %1, %2, %3, [%5], [%6], p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(tsfa_addr), "r"(tsfb_addr));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_MXF8F6F4_2x1SM_SS without CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_MXF8F6F4_2x1SM_SS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_MXF8F6F4_2x1SM_SS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1314-1353
```cpp
template <class a_type, class b_type, class c_type,
          int M, int N, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_F8F6F4_2x1SM_SS_SPARSE
{
  static_assert(M == 128 || M == 256, "SM100_MMA_F8F6F4_2x1SM_SS_SPARSE M-mode size should be 128 or 256 for 2 CTA cluster MMA.");
  static_assert((N % 32 == 0) && (32 <= N) && (N <= 256), "SM100_MMA_F8F6F4_2x1SM_SS_SPARSE N-mode size should be a multiple of 32 between 32 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE,
      uint32_t const& tmem_e)
  {
#if defined(CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED)
    if (cute::elect_one_sync()) {
      uint32_t mask[8] = {0, 0, 0, 0, 0, 0, 0, 0};
      asm volatile(
        "{\n\t"
        ".reg .pred p;\n\t"
        "setp.ne.b32 p, %4, 0;\n\t"
        "tcgen05.mma.sp.cta_group::2.kind::f8f6f4 [%0], %1, %2, [%13], %3, {%5, %6, %7, %8, %9, %10, %11, %12}, p; \n\t"
        "}\n"
        :
        : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
          "r"(mask[0]), "r"(mask[1]), "r"(mask[2]), "r"(mask[3]),
          "r"(mask[4]), "r"(mask[5]), "r"(mask[6]), "r"(mask[7]), "r"(tmem_e));
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_F8F6F4_2x1SM_SS_SPARSE without CUTE_ARCH_TCGEN05_MXF8F6F4_MMA_ENABLED");
#endif
  }
};
```
- **EN:** Defines `SM100_MMA_F8F6F4_2x1SM_SS_SPARSE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_F8F6F4_2x1SM_SS_SPARSE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1355-1424
```cpp
template <class a_type, class b_type, class c_type, class sf_type,
          int M, int N, int VS, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_MXF4_SS
{
  static_assert(M == 128, "SM100_MMA_MXF4_SS M-mode size should be 128 for 1 CTA cluster MMA.");
  static_assert((N % 8 == 0) && (8 <= N) && (N <= 256), "SM100_MMA_MXF4_SS N-mode size should be a multiple of 8 between 8 and 256.");
  static_assert((VS == 16) || (VS == 32), "SM100_MMA_MXF4_SS Vector size can only be 16 or 32.");

  using DRegisters   = void;
  using ARegisters   = uint64_t[1];
  using BRegisters   = uint64_t[1];
  using CRegisters   = uint32_t[1];
  using SFARegisters = uint32_t[1];
  using SFBRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE,
      uint32_t const& tsfa_addr,
      uint32_t const& tsfb_addr)
  {
    if constexpr (VS == 16) {
#if defined(CUTE_ARCH_TCGEN05_MXF4NVF4_MMA_ENABLED)
      if (cute::elect_one_sync()) {
        asm volatile(
          "{\n\t"
          ".reg .pred p;\n\t"
          "setp.ne.b32 p, %4, 0;\n\t"
#if (__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9)
          "tcgen05.mma.cta_group::1.kind::mxf4nvf4.block_scale.block16 [%0], %1, %2, %3, [%5], [%6], p; \n\t"
#else
          "tcgen05.mma.cta_group::1.kind::mxf4nvf4.block_scale.scale_vec::4X [%0], %1, %2, %3, [%5], [%6], p; \n\t"
#endif
          "}\n"
          :
          : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
            "r"(tsfa_addr), "r"(tsfb_addr));
      }
#else
      CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_MXF4_SS (VS = 16) without CUTE_ARCH_TCGEN05_MXF4NVF4_MMA_ENABLED");
#endif
    }
    if constexpr (VS == 32) {
#if defined(CUTE_ARCH_TCGEN05_MXF4_MMA_ENABLED)
      if (cute::elect_one_sync()) {
        asm volatile(
          "{\n\t"
          ".reg .pred p;\n\t"
          "setp.ne.b32 p, %4, 0;\n\t"
#if (__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9)
          "tcgen05.mma.cta_group::1.kind::mxf4.block_scale.block32 [%0], %1, %2, %3, [%5], [%6], p; \n\t"
#else
          "tcgen05.mma.cta_group::1.kind::mxf4.block_scale.scale_vec::2X [%0], %1, %2, %3, [%5], [%6], p; \n\t"
#endif
          "}\n"
          :
          : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
            "r"(tsfa_addr), "r"(tsfb_addr));
      }
#else
      CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_MXF4_SS (VS = 32) without CUTE_ARCH_TCGEN05_MXF4_MMA_ENABLED");
#endif
    }
  }

};
```
- **EN:** Defines `SM100_MMA_MXF4_SS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_MXF4_SS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1426-1495
```cpp
template <class a_type, class b_type, class c_type, class sf_type,
          int M, int N, int VS, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_MXF4NVF4_SS_SPARSE
{
  static_assert(M == 128, "SM100_MMA_MXF4NVF4_SS_SPARSE M-mode size should be 128 for 1 CTA cluster MMA.");
  static_assert((N % 8 == 0) && (8 <= N) && (N <= 256), "SM100_MMA_MXF4NVF4_SS_SPARSE N-mode size should be a multiple of 8 between 8 and 256.");

  using DRegisters = void;
  using ARegisters = uint64_t[1];
  using BRegisters = uint64_t[1];
  using CRegisters = uint32_t[1];
  using SFARegisters = uint32_t[1];
  using SFBRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE,
      uint32_t const& tsfa_addr,
      uint32_t const& tsfb_addr,
      uint32_t const& tmem_e)
  {
    if constexpr (VS == 32) {
#if defined(CUTE_ARCH_TCGEN05_MXF4NVF4_MMA_ENABLED)
      if (cute::elect_one_sync()) {
        asm volatile(
          "{\n\t"
          ".reg .pred p;\n\t"
          "setp.ne.b32 p, %4, 0;\n\t"
#if (__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9)
          "tcgen05.mma.sp.cta_group::1.kind::mxf4nvf4.block_scale.block16 [%0], %1, %2, [%7], %3, [%5], [%6], p; \n\t"
#else
          "tcgen05.mma.sp.cta_group::1.kind::mxf4nvf4.block_scale.scale_vec::4X [%0], %1, %2, [%7], %3, [%5], [%6], p; \n\t"
#endif
          "}\n"
          :
          : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
            "r"(tsfa_addr), "r"(tsfb_addr), "r"(tmem_e));
      }
#else
      CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_MXF4NVF4_SS_SPARSE (VS = 32) without CUTE_ARCH_TCGEN05_MXF4NVF4_MMA_ENABLED");
#endif
    }

    if constexpr (VS == 64) {
#if defined(CUTE_ARCH_TCGEN05_MXF4_MMA_ENABLED)
      if (cute::elect_one_sync()) {
        asm volatile(
          "{\n\t"
          ".reg .pred p;\n\t"
          "setp.ne.b32 p, %4, 0;\n\t"
#if (__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9)
          "tcgen05.mma.sp.cta_group::1.kind::mxf4.block_scale.block32 [%0], %1, %2, [%7], %3, [%5], [%6], p; \n\t"
#else
          "tcgen05.mma.sp.cta_group::1.kind::mxf4.block_scale.scale_vec::2X [%0], %1, %2, [%7], %3, [%5], [%6], p; \n\t"
#endif
          "}\n"
          :
          : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
            "r"(tsfa_addr), "r"(tsfb_addr), "r"(tmem_e));
      }
#else
      CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_MXF4NVF4_SS_SPARSE (VS = 64) without CUTE_ARCH_TCGEN05_MXF4_MMA_ENABLED");
#endif
    }
  }
};
```
- **EN:** Defines `SM100_MMA_MXF4NVF4_SS_SPARSE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_MXF4NVF4_SS_SPARSE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1497-1565
```cpp
template <class a_type, class b_type, class c_type, class sf_type,
          int M, int N, int VS, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_MXF4_2x1SM_SS
{
  static_assert(M == 128 || M == 256, "SM100_MMA_MXF4_2x1SM_SS M-mode size should be 128 or 256 for 2 CTA cluster MMA.");
  static_assert((N % 16 == 0) && (16 <= N) && (N <= 256), "SM100_MMA_MXF4_2x1SM_SS N-mode size should be a multiple of 16 between 16 and 256.");
  static_assert((VS == 16) || (VS == 32), "SM100_MMA_MXF4_2x1SM_SS Vector size can only be 16 or 32.");

  using DRegisters   = void;
  using ARegisters   = uint64_t[1];
  using BRegisters   = uint64_t[1];
  using CRegisters   = uint32_t[1];
  using SFARegisters = uint32_t[1];
  using SFBRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE,
      uint32_t const& tsfa_addr,
      uint32_t const& tsfb_addr)
  {
    if constexpr (VS == 16) {
#if defined(CUTE_ARCH_TCGEN05_MXF4NVF4_MMA_ENABLED)
      if (cute::elect_one_sync()) {
        asm volatile(
          "{\n\t"
          ".reg .pred p;\n\t"
          "setp.ne.b32 p, %4, 0;\n\t"
#if (__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9)
          "tcgen05.mma.cta_group::2.kind::mxf4nvf4.block_scale.block16 [%0], %1, %2, %3, [%5], [%6], p; \n\t"
#else
          "tcgen05.mma.cta_group::2.kind::mxf4nvf4.block_scale.scale_vec::4X [%0], %1, %2, %3, [%5], [%6], p; \n\t"
#endif
          "}\n"
          :
          : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
            "r"(tsfa_addr), "r"(tsfb_addr));
      }
#else
      CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_MXF4_2x1SM_SS (VS = 16) without CUTE_ARCH_TCGEN05_MXF4NVF4_MMA_ENABLED");
#endif
    }
    if constexpr (VS == 32) {
#if defined(CUTE_ARCH_TCGEN05_MXF4_MMA_ENABLED)
      if (cute::elect_one_sync()) {
        asm volatile(
          "{\n\t"
          ".reg .pred p;\n\t"
          "setp.ne.b32 p, %4, 0;\n\t"
#if (__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9)
          "tcgen05.mma.cta_group::2.kind::mxf4.block_scale.block32 [%0], %1, %2, %3, [%5], [%6], p; \n\t"
#else
          "tcgen05.mma.cta_group::2.kind::mxf4.block_scale.scale_vec::2X [%0], %1, %2, %3, [%5], [%6], p; \n\t"
#endif
          "}\n"
          :
          : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
            "r"(tsfa_addr), "r"(tsfb_addr));
      }
#else
      CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_MXF4_2x1SM_SS (VS = 32) without CUTE_ARCH_TCGEN05_MXF4_MMA_ENABLED");
#endif
    }
  }
};
```
- **EN:** Defines `SM100_MMA_MXF4_2x1SM_SS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_MXF4_2x1SM_SS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1567-1636
```cpp
template <class a_type, class b_type, class c_type, class sf_type,
          int M, int N, int VS, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM100_MMA_MXF4NVF4_2x1SM_SS_SPARSE
{
  static_assert((N % 16 == 0) && (16 <= N) && (N <= 256), "SM100_MMA_MXF4NVF4_2x1SM_SS_SPARSE N-mode size should be a multiple of 16 between 16 and 256.");
  static_assert((VS == 32) || (VS == 64), "SM100_MMA_MXF4NVF4_2x1SM_SS_SPARSE Vector size can only be 32 or 64.");

  using DRegisters   = void;
  using ARegisters   = uint64_t[1];
  using BRegisters   = uint64_t[1];
  using CRegisters   = uint32_t[1];
  using SFARegisters = uint32_t[1];
  using SFBRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE,
      uint32_t const& tsfa_addr,
      uint32_t const& tsfb_addr,
      uint32_t const& tmem_e)
  {
    if constexpr (VS == 32) {
#if defined(CUTE_ARCH_TCGEN05_MXF4NVF4_MMA_ENABLED)
      if (cute::elect_one_sync()) {
        asm volatile(
          "{\n\t"
          ".reg .pred p;\n\t"
          "setp.ne.b32 p, %4, 0;\n\t"
#if (__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9)
          "tcgen05.mma.sp.cta_group::2.kind::mxf4nvf4.block_scale.block16 [%0], %1, %2, [%7], %3, [%5], [%6], p; \n\t"
#else
          "tcgen05.mma.sp.cta_group::2.kind::mxf4nvf4.block_scale.scale_vec::4X [%0], %1, %2, [%7], %3, [%5], [%6], p; \n\t"
#endif
          "}\n"
          :
          : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
            "r"(tsfa_addr), "r"(tsfb_addr), "r"(tmem_e));
      }
#else
      CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_MXF4NVF4_2x1SM_SS_SPARSE (VS = 32) without CUTE_ARCH_TCGEN05_MXF4NVF4_MMA_ENABLED");
#endif
    }

    if constexpr (VS == 64) {
#if defined(CUTE_ARCH_TCGEN05_MXF4_MMA_ENABLED)
      if (cute::elect_one_sync()) {
        asm volatile(
          "{\n\t"
          ".reg .pred p;\n\t"
          "setp.ne.b32 p, %4, 0;\n\t"
#if (__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9)
          "tcgen05.mma.sp.cta_group::2.kind::mxf4.block_scale.block32 [%0], %1, %2, [%7], %3, [%5], [%6], p; \n\t"
#else
          "tcgen05.mma.sp.cta_group::2.kind::mxf4.block_scale.scale_vec::2X [%0], %1, %2, [%7], %3, [%5], [%6], p; \n\t"
#endif
          "}\n"
          :
          : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
            "r"(tsfa_addr), "r"(tsfb_addr), "r"(tmem_e));
      }
#else
      CUTE_INVALID_CONTROL_PATH("Attempting to use SM100_MMA_MXF4NVF4_2x1SM_SS_SPARSE (VS = 64) without CUTE_ARCH_TCGEN05_MXF4_MMA_ENABLED");
#endif
    }
  }
};
```
- **EN:** Defines `SM100_MMA_MXF4NVF4_2x1SM_SS_SPARSE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM100_MMA_MXF4NVF4_2x1SM_SS_SPARSE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1638-1647
```cpp
namespace SM103 {
template <class a_type, class b_type, class c_type, class sf_type,
          int M, int N, int VS, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM103_MXF4_ULTRA_SS_VS
{
  static_assert(M == 128, "MMA M-mode size should be 128 for 1 CTA cluster MMA.");
  static_assert((N % 16 == 0) && (16 <= N) && (N <= 256), "MMA N-mode size should be a multiple of 16 between 16 and 256.");
  static_assert(((VS == 32) & (is_same_v<a_type, cutlass::float_e2m1_t> && is_same_v<sf_type, cutlass::float_ue8m0_t>)) || (VS == 16),
    "Vector size can only be 4x mode (VS=16) or 2x mode (VS=32) for MMA. 2x mode only supports float_e2m1_t for a/b types and ue8m0_t for sf type");
```
- **EN:** Enters or leaves namespace scope `SM103` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `SM103`，以便把相关符号组织在一起。

### Lines 1649-1654
```cpp
  using DRegisters   = void;
  using ARegisters   = uint64_t[1];
  using BRegisters   = uint64_t[1];
  using CRegisters   = uint32_t[1];
  using SFARegisters = uint32_t[1];
  using SFBRegisters = uint32_t[1];
```
- **EN:** Introduces the alias `DRegisters` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `DRegisters`，便于复用周边的寄存器、布局或策略类型。

### Lines 1656-1703
```cpp
  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE,
      uint32_t const& tsfa_addr,
      uint32_t const& tsfb_addr)
  {
#if defined(CUTE_ARCH_TCGEN05_MXF4NVF4_MMA_ULTRA_ENABLED)
    if constexpr (VS == 16) {
      if (cute::elect_one_sync()) {
        asm volatile(
          "{\n\t"
          ".reg .pred p;\n\t"
          "setp.ne.b32 p, %4, 0;\n\t"
#if (__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9)
          "tcgen05.mma.cta_group::1.kind::mxf4nvf4.block_scale.block16 [%0], %1, %2, %3, [%5], [%6], p; \n\t"
#else
          "tcgen05.mma.cta_group::1.kind::mxf4nvf4.block_scale.scale_vec::4X [%0], %1, %2, %3, [%5], [%6], p; \n\t"
#endif
          "}\n"
          :
          : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
            "r"(tsfa_addr), "r"(tsfb_addr));
      }
    }
    else if constexpr (VS == 32) {
      if (cute::elect_one_sync()) {
        asm volatile(
          "{\n\t"
          ".reg .pred p;\n\t"
          "setp.ne.b32 p, %4, 0;\n\t"
#if (__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9)
          "tcgen05.mma.cta_group::1.kind::mxf4nvf4.block_scale.block32 [%0], %1, %2, %3, [%5], [%6], p; \n\t"
#else
          "tcgen05.mma.cta_group::1.kind::mxf4nvf4.block_scale.scale_vec::2X [%0], %1, %2, %3, [%5], [%6], p; \n\t"
#endif
          "}\n"
          :
          : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
            "r"(tsfa_addr), "r"(tsfb_addr));
      }
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM103_MXF4_ULTRA_SS_VS without CUTE_ARCH_MMA_SM103A_ENABLED");
#endif
  }
```
- **EN:** Implements `fma` by issuing one NVIDIA SM100 MMA instruction and mapping the accumulator/source fragments onto hardware operands.
- **CN:** 通过发出一条 NVIDIA SM100 MMA 指令实现 `fma`，并把累加器/源片段映射到硬件操作数。

### Lines 1705-1705
```cpp
};
```
- **EN:** Implements a supporting fragment of the surrounding algorithm, trait, or architecture wrapper.
- **CN:** 实现周边算法、trait 或体系结构封装所需的辅助片段。

### Lines 1708-1775
```cpp
template <class a_type, class b_type, class c_type, class sf_type,
          int M, int N, int VS, UMMA::Major a_major, UMMA::Major b_major,
          UMMA::ScaleIn a_neg = UMMA::ScaleIn::One, UMMA::ScaleIn b_neg = UMMA::ScaleIn::One>
struct SM103_MXF4_ULTRA_2x1SM_SS_VS
{
  static_assert(M == 128 || M == 256, "MMA M-mode size should be 128 or 256 for 2 CTA cluster MMA.");
  static_assert((N % 16 == 0) && (16 <= N) && (N <= 256), "MMA N-mode size should be a multiple of 16 between 16 and 256.");
  static_assert(((VS == 32) & (is_same_v<a_type, cutlass::float_e2m1_t> && is_same_v<sf_type, cutlass::float_ue8m0_t>)) || (VS == 16),
    "Vector size can only be 4x mode (VS=16) or 2x mode (VS=32) for MMA. 2x mode only supports float_e2m1_t for a/b types and ue8m0_t for sf type");

  using DRegisters   = void;
  using ARegisters   = uint64_t[1];
  using BRegisters   = uint64_t[1];
  using CRegisters   = uint32_t[1];
  using SFARegisters = uint32_t[1];
  using SFBRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  fma(uint64_t const& desc_a,
      uint64_t const& desc_b,
      uint32_t const& tmem_c,
      uint32_t const& scaleC,
      uint64_t const& idescE,
      uint32_t const& tsfa_addr,
      uint32_t const& tsfb_addr)
  {
#if defined(CUTE_ARCH_TCGEN05_MXF4NVF4_MMA_ULTRA_ENABLED)
    if constexpr (VS == 16) {
      if (cute::elect_one_sync()) {
        asm volatile(
          "{\n\t"
          ".reg .pred p;\n\t"
          "setp.ne.b32 p, %4, 0;\n\t"
#if (__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9)
          "tcgen05.mma.cta_group::2.kind::mxf4nvf4.block_scale.block16 [%0], %1, %2, %3, [%5], [%6], p; \n\t"
#else
          "tcgen05.mma.cta_group::2.kind::mxf4nvf4.block_scale.scale_vec::4X [%0], %1, %2, %3, [%5], [%6], p; \n\t"
#endif
          "}\n"
          :
          : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
            "r"(tsfa_addr), "r"(tsfb_addr));
      }
    }
    else if constexpr (VS == 32) {
      if (cute::elect_one_sync()) {
        asm volatile(
          "{\n\t"
          ".reg .pred p;\n\t"
          "setp.ne.b32 p, %4, 0;\n\t"
#if (__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 9)
          "tcgen05.mma.cta_group::2.kind::mxf4nvf4.block_scale.block32 [%0], %1, %2, %3, [%5], [%6], p; \n\t"
#else
          "tcgen05.mma.cta_group::2.kind::mxf4nvf4.block_scale.scale_vec::2X [%0], %1, %2, %3, [%5], [%6], p; \n\t"
#endif
          "}\n"
          :
          : "r"(tmem_c), "l"(desc_a), "l"(desc_b), "r"(uint32_t(idescE>>32)), "r"(scaleC),
            "r"(tsfa_addr), "r"(tsfb_addr));
      }
    }
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM103_MXF4_ULTRA_2x1SM_SS_VS without CUTE_ARCH_MMA_SM103A_ENABLED");
#endif
  }

};
} // namespace SM103
```
- **EN:** Defines `SM103_MXF4_ULTRA_2x1SM_SS_VS` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM103_MXF4_ULTRA_2x1SM_SS_VS`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 1777-1777
```cpp
} // end namespace cute
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
- **EN:** `cute/arch/mma_sm100.hpp` supplies related definitions from `cute/arch/mma_sm100.hpp`.
  **CN:** `cute/arch/mma_sm100.hpp` 提供了来自 `cute/arch/mma_sm100.hpp` 的相关定义。
- **EN:** `cute/arch/mma_sm100_desc.hpp` supplies related definitions from `cute/arch/mma_sm100_desc.hpp`.
  **CN:** `cute/arch/mma_sm100_desc.hpp` 提供了来自 `cute/arch/mma_sm100_desc.hpp` 的相关定义。
- **EN:** `cute/arch/cluster_sm90.hpp` supplies related definitions from `cute/arch/cluster_sm90.hpp`.
  **CN:** `cute/arch/cluster_sm90.hpp` 提供了来自 `cute/arch/cluster_sm90.hpp` 的相关定义。
