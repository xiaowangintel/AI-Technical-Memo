# mma_sm89.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/mma_sm89.hpp`
- **EN:** Defines matrix-multiply-accumulate instruction wrappers for NVIDIA SM89.
- **CN:** 为 NVIDIA SM89 定义矩阵乘加指令封装。

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

//
```
- **EN:** Carries the BSD-3-Clause license and redistribution notice for this header.
- **CN:** 给出该头文件的 BSD-3-Clause 许可证与再分发声明。

### Lines 34-35
```cpp
//
#pragma once
```
- **EN:** Uses `#pragma once` to prevent multiple inclusion of the header.
- **CN:** 使用 `#pragma once` 防止头文件被重复包含。

### Lines 37-38
```cpp
#include <cute/config.hpp>
#include <cute/arch/mma.hpp>
```
- **EN:** Imports `cute/config.hpp` (core CuTe configuration macros and portability annotations); `cute/arch/mma.hpp` (generic low-level MMA operation tags and wrappers).
- **CN:** 引入 `cute/config.hpp`（CuTe 核心配置宏与可移植性标注）；`cute/arch/mma.hpp`（通用底层 MMA 操作标签与封装）。

### Lines 40-44
```cpp
////////////////////////////////////////////////////////////////////////////////

#if (__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 4)
#  define CUTE_ARCH_MMA_F32_SM89_SUPPORTED
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTE_ARCH_MMA_F32_SM89_SUPPORTED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTE_ARCH_MMA_F32_SM89_SUPPORTED）只启用当前目标有效的构建路径。

### Lines 46-48
```cpp
#if (__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 8)
#  define CUTE_ARCH_MMA_F16_SM89_SUPPORTED
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTE_ARCH_MMA_F16_SM89_SUPPORTED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTE_ARCH_MMA_F16_SM89_SUPPORTED）只启用当前目标有效的构建路径。

### Lines 50-53
```cpp
#if defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 890)
#  if defined(CUTE_ARCH_MMA_F32_SM89_SUPPORTED)
#    define CUTE_ARCH_MMA_F32_SM89_ENABLED
#  endif
```
- **EN:** Uses preprocessor checks/macros (__CUDA_ARCH__, CUTE_ARCH_MMA_F32_SM89_SUPPORTED, CUTE_ARCH_MMA_F32_SM89_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（__CUDA_ARCH__, CUTE_ARCH_MMA_F32_SM89_SUPPORTED, CUTE_ARCH_MMA_F32_SM89_ENABLED）只启用当前目标有效的构建路径。

### Lines 55-58
```cpp
#  if defined(CUTE_ARCH_MMA_F16_SM89_SUPPORTED)
#    define CUTE_ARCH_MMA_F16_SM89_ENABLED
#  endif
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTE_ARCH_MMA_F16_SM89_SUPPORTED, CUTE_ARCH_MMA_F16_SM89_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTE_ARCH_MMA_F16_SM89_SUPPORTED, CUTE_ARCH_MMA_F16_SM89_ENABLED）只启用当前目标有效的构建路径。

### Lines 60-296
```cpp
////////////////////////////////////////////////////////////////////////////////

namespace cute {
// MMA 16x8x32 TN
struct SM89_16x8x32_F32E4M3E4M3F32_TN
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float      & d0, float      & d1, float      & d2, float      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const& c0, float const& c1, float const& c2, float const& c3)
  {
#if defined(CUTE_ARCH_MMA_F32_SM89_ENABLED)
    asm(
      "mma.sync.aligned.m16n8k32.row.col.f32.e4m3.e4m3.f32 "
      "{%0,%1,%2,%3}, {%4,%5,%6,%7}, {%8,%9}, {%10,%11,%12,%13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :
        "r"(a0), "r"(a1), "r"(a2), "r"(a3),
        "r"(b0), "r"(b1),
        "f"(c0), "f"(c1), "f"(c2), "f"(c3)
  );
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM89_16x8x32_F32E4M3E4M3F32_TN without CUTE_ARCH_MMA_F32_SM89_ENABLED");
#endif
  }
};

struct SM89_16x8x32_F32E4M3E5M2F32_TN
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float      & d0, float      & d1, float      & d2, float      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const& c0, float const& c1, float const& c2, float const& c3)
  {
#if defined(CUTE_ARCH_MMA_F32_SM89_ENABLED)
    asm(
      "mma.sync.aligned.m16n8k32.row.col.f32.e4m3.e5m2.f32 "
      "{%0,%1,%2,%3}, {%4,%5,%6,%7}, {%8,%9}, {%10,%11,%12,%13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :
        "r"(a0), "r"(a1), "r"(a2), "r"(a3),
        "r"(b0), "r"(b1),
        "f"(c0), "f"(c1), "f"(c2), "f"(c3)
  );
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM89_16x8x32_F32E4M3E5M2F32_TN without CUTE_ARCH_MMA_F32_SM89_ENABLED");
#endif
  }
};

struct SM89_16x8x32_F32E5M2E5M2F32_TN
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float      & d0, float      & d1, float      & d2, float      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const& c0, float const& c1, float const& c2, float const& c3)
  {
#if defined(CUTE_ARCH_MMA_F32_SM89_ENABLED)
    asm(
      "mma.sync.aligned.m16n8k32.row.col.f32.e5m2.e5m2.f32 "
      "{%0,%1,%2,%3}, {%4,%5,%6,%7}, {%8,%9}, {%10,%11,%12,%13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :
        "r"(a0), "r"(a1), "r"(a2), "r"(a3),
        "r"(b0), "r"(b1),
        "f"(c0), "f"(c1), "f"(c2), "f"(c3)
  );
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM89_16x8x32_F32E5M2E5M2F32_TN without CUTE_ARCH_MMA_F32_SM89_ENABLED");
#endif
  }
};

struct SM89_16x8x32_F32E5M2E4M3F32_TN
{
  using DRegisters = float[4];
  using ARegisters = uint32_t[4];
  using BRegisters = uint32_t[2];
  using CRegisters = float[4];

  CUTE_HOST_DEVICE static void
  fma(float      & d0, float      & d1, float      & d2, float      & d3,
      uint32_t const& a0, uint32_t const& a1, uint32_t const& a2, uint32_t const& a3,
      uint32_t const& b0, uint32_t const& b1,
      float const& c0, float const& c1, float const& c2, float const& c3)
  {
#if defined(CUTE_ARCH_MMA_F32_SM89_ENABLED)
    asm(
      "mma.sync.aligned.m16n8k32.row.col.f32.e5m2.e4m3.f32 "
      "{%0,%1,%2,%3}, {%4,%5,%6,%7}, {%8,%9}, {%10,%11,%12,%13};\n"
      : "=f"(d0), "=f"(d1), "=f"(d2), "=f"(d3)
      :
        "r"(a0), "r"(a1), "r"(a2), "r"(a3),
        "r"(b0), "r"(b1),
        "f"(c0), "f"(c1), "f"(c2), "f"(c3)
  );
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM89_16x8x32_F32E5M2E4M3F32_TN without CUTE_ARCH_MMA_F32_SM89_ENABLED");
#endif
  }
};

struct SM89_16x8x32_F16E4M3E4M3F16_TN
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
#if defined(CUTE_ARCH_MMA_F16_SM89_ENABLED)
    asm(
      "mma.sync.aligned.m16n8k32.row.col.f16.e4m3.e4m3.f16 "
      "{%0,%1}, {%2,%3,%4,%5}, {%6,%7}, {%8,%9};\n"
      : "=r"(d0), "=r"(d1)
      :
        "r"(a0), "r"(a1), "r"(a2), "r"(a3),
        "r"(b0), "r"(b1),
        "r"(c0), "r"(c1)
  );
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM89_16x8x32_F16E4M3E4M3F16_TN without CUTE_ARCH_MMA_F16_SM89_ENABLED");
#endif
  }
};

struct SM89_16x8x32_F16E4M3E5M2F16_TN
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
#if defined(CUTE_ARCH_MMA_F16_SM89_ENABLED)
    asm(
      "mma.sync.aligned.m16n8k32.row.col.f16.e4m3.e5m2.f16 "
      "{%0,%1}, {%2,%3,%4,%5}, {%6,%7}, {%8,%9};\n"
      : "=r"(d0), "=r"(d1)
      :
        "r"(a0), "r"(a1), "r"(a2), "r"(a3),
        "r"(b0), "r"(b1),
        "r"(c0), "r"(c1)
  );
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM89_16x8x32_F16E4M3E5M2F16_TN without CUTE_ARCH_MMA_F16_SM89_ENABLED");
#endif
  }
};

struct SM89_16x8x32_F16E5M2E4M3F16_TN
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
#if defined(CUTE_ARCH_MMA_F16_SM89_ENABLED)
    asm(
      "mma.sync.aligned.m16n8k32.row.col.f16.e5m2.e4m3.f16 "
      "{%0,%1}, {%2,%3,%4,%5}, {%6,%7}, {%8,%9};\n"
      : "=r"(d0), "=r"(d1)
      :
        "r"(a0), "r"(a1), "r"(a2), "r"(a3),
        "r"(b0), "r"(b1),
        "r"(c0), "r"(c1)
  );
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM89_16x8x32_F16E5M2E4M3F16_TN without CUTE_ARCH_MMA_F16_SM89_ENABLED");
#endif
  }
};

struct SM89_16x8x32_F16E5M2E5M2F16_TN
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
#if defined(CUTE_ARCH_MMA_F16_SM89_ENABLED)
    asm(
      "mma.sync.aligned.m16n8k32.row.col.f16.e5m2.e5m2.f16 "
      "{%0,%1}, {%2,%3,%4,%5}, {%6,%7}, {%8,%9};\n"
      : "=r"(d0), "=r"(d1)
      :
        "r"(a0), "r"(a1), "r"(a2), "r"(a3),
        "r"(b0), "r"(b1),
        "r"(c0), "r"(c1)
  );
#else
    CUTE_INVALID_CONTROL_PATH("Attempting to use SM89_16x8x32_F16E5M2E5M2F16_TN without CUTE_ARCH_MMA_F16_SM89_ENABLED");
#endif
  }
};

} // namespace cute
```
- **EN:** Enters or leaves namespace scope `cute, cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute, cute`，以便把相关符号组织在一起。

## Key Concepts / 关键概念

- **EN:** MMA/GMMA/UMMA wrappers expose tensor-core style matrix instructions as typed C++ interfaces.
  **CN:** MMA/GMMA/UMMA 封装把张量核心式矩阵指令暴露为带类型的 C++ 接口。

## Dependencies / 依赖关系

- **EN:** `cute/config.hpp` supplies core CuTe configuration macros and portability annotations.
  **CN:** `cute/config.hpp` 提供了CuTe 核心配置宏与可移植性标注。
- **EN:** `cute/arch/mma.hpp` supplies generic low-level MMA operation tags and wrappers.
  **CN:** `cute/arch/mma.hpp` 提供了通用底层 MMA 操作标签与封装。
- **EN:** CUDA architecture macros gate device-only fast paths and inline instructions.
  **CN:** CUDA 架构宏用于控制仅设备端可用的快速路径与内联指令。
