# copy_sm90.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/copy_sm90.hpp`
- **EN:** Defines low-level copy instructions and wrappers for NVIDIA SM90 or generic backends.
- **CN:** 为 NVIDIA SM90 或通用后端定义底层拷贝指令与封装。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31
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

### Lines 33-35
```cpp
#include <cute/config.hpp>      // CUTE_HOST_DEVICE
#include <cute/arch/config.hpp> // CUTE_ARCH_TMA_SMxx_ENABLED
#include <cute/arch/copy.hpp>
```
- **EN:** Imports `cute/config.hpp` (core CuTe configuration macros and portability annotations); `cute/arch/config.hpp` (related definitions from `cute/arch/config.hpp`); `cute/arch/copy.hpp` (generic low-level copy primitives and policies).
- **CN:** 引入 `cute/config.hpp`（CuTe 核心配置宏与可移植性标注）；`cute/arch/config.hpp`（来自 `cute/arch/config.hpp` 的相关定义）；`cute/arch/copy.hpp`（通用底层拷贝原语与策略）。

### Lines 37-38
```cpp
namespace cute
{
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 40-58
```cpp
struct SM90_U32x1_STSM_N
{
  using SRegisters = uint32_t[1];
  using DRegisters = uint128_t[1];

  CUTE_HOST_DEVICE static void
  copy(uint32_t const& src,
       uint128_t     & smem_dst)
  {
#if defined(CUTE_ARCH_STSM_SM90_ENABLED)
    uint32_t smem_int_ptr = cast_smem_ptr_to_uint(&smem_dst);
    asm volatile ("stmatrix.sync.aligned.x1.m8n8.shared.b16 [%0], {%1};\n"
        :: "r"(smem_int_ptr),
           "r"(src));
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use stmatrix without CUTE_ARCH_STSM_SM90_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM90_U32x1_STSM_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM90_U32x1_STSM_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 60-78
```cpp
struct SM90_U32x2_STSM_N
{
  using SRegisters = uint32_t[2];
  using DRegisters = uint128_t[1];

  CUTE_HOST_DEVICE static void
  copy(uint32_t const& src0, uint32_t const& src1,
       uint128_t& smem_dst)
  {
#if defined(CUTE_ARCH_STSM_SM90_ENABLED)
    uint32_t smem_int_ptr = cast_smem_ptr_to_uint(&smem_dst);
    asm volatile ("stmatrix.sync.aligned.x2.m8n8.shared.b16 [%0], {%1, %2};\n"
        :: "r"(smem_int_ptr),
           "r"(src0), "r"(src1));
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use stmatrix without CUTE_ARCH_STSM_SM90_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM90_U32x2_STSM_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM90_U32x2_STSM_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 80-98
```cpp
struct SM90_U32x4_STSM_N
{
  using SRegisters = uint32_t[4];
  using DRegisters = uint128_t[1];

  CUTE_HOST_DEVICE static void
  copy(uint32_t const& src0, uint32_t const& src1, uint32_t const& src2, uint32_t const& src3,
       uint128_t& smem_dst)
  {
#if defined(CUTE_ARCH_STSM_SM90_ENABLED)
    uint32_t smem_int_ptr = cast_smem_ptr_to_uint(&smem_dst);
    asm volatile ("stmatrix.sync.aligned.x4.m8n8.shared.b16 [%0], {%1, %2, %3, %4};\n"
        :: "r"(smem_int_ptr),
          "r"(src0), "r"(src1), "r"(src2), "r"(src3));
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use stmatrix without CUTE_ARCH_STSM_SM90_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM90_U32x4_STSM_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM90_U32x4_STSM_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 100-118
```cpp
struct SM90_U16x2_STSM_T
{
  using SRegisters = uint32_t[1];
  using DRegisters = uint128_t[1];

  CUTE_HOST_DEVICE static void
  copy(uint32_t const& src,
       uint128_t& smem_dst)
  {
#if defined(CUTE_ARCH_STSM_SM90_ENABLED)
    uint32_t smem_int_ptr = cast_smem_ptr_to_uint(&smem_dst);
    asm volatile ("stmatrix.sync.aligned.x1.trans.m8n8.shared.b16 [%0], {%1};\n"
        :: "r"(smem_int_ptr),
           "r"(src));
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use stmatrix without CUTE_ARCH_STSM_SM90_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM90_U16x2_STSM_T` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM90_U16x2_STSM_T`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 120-138
```cpp
struct SM90_U16x4_STSM_T
{
  using SRegisters = uint32_t[2];
  using DRegisters = uint128_t[1];

  CUTE_HOST_DEVICE static void
  copy(uint32_t const& src0, uint32_t const& src1,
       uint128_t& smem_dst)
  {
#if defined(CUTE_ARCH_STSM_SM90_ENABLED)
    uint32_t smem_int_ptr = cast_smem_ptr_to_uint(&smem_dst);
    asm volatile ("stmatrix.sync.aligned.x2.trans.m8n8.shared.b16 [%0], {%1, %2};\n"
        :: "r"(smem_int_ptr),
           "r"(src0), "r"(src1));
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use stmatrix without CUTE_ARCH_STSM_SM90_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM90_U16x4_STSM_T` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM90_U16x4_STSM_T`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 140-158
```cpp
struct SM90_U16x8_STSM_T
{
  using SRegisters = uint32_t[4];
  using DRegisters = uint128_t[1];

  CUTE_HOST_DEVICE static void
  copy(uint32_t const& src0, uint32_t const& src1, uint32_t const& src2, uint32_t const& src3,
       uint128_t& smem_dst)
  {
#if defined(CUTE_ARCH_STSM_SM90_ENABLED)
    uint32_t smem_int_ptr = cast_smem_ptr_to_uint(&smem_dst);
    asm volatile ("stmatrix.sync.aligned.x4.trans.m8n8.shared.b16 [%0], {%1, %2, %3, %4};\n"
        :: "r"(smem_int_ptr),
          "r"(src0), "r"(src1), "r"(src2), "r"(src3));
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use stmatrix without CUTE_ARCH_STSM_SM90_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM90_U16x8_STSM_T` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM90_U16x8_STSM_T`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 160-185
```cpp
//
// Legacy STSM interfaces that aren't very useful
//

template <class T>
CUTE_HOST_DEVICE
void
copy_stsm(T const* const rmem_ptr,
          uint128_t* const smem_ptr)
{
  uint32_t const* reg_ptr = reinterpret_cast<uint32_t const*>(rmem_ptr);

  // if constexpr
  if (sizeof(T) == 4) {
    SM90_U32x1_STSM_N::copy(reg_ptr[0], smem_ptr[0]);
  }
  else if (sizeof(T) == 8) {
    SM90_U32x2_STSM_N::copy(reg_ptr[0], reg_ptr[1], smem_ptr[0]);
  }
  else if (sizeof(T) == 16) {
    SM90_U32x4_STSM_N::copy(reg_ptr[0], reg_ptr[1], reg_ptr[2], reg_ptr[3], smem_ptr[0]);
  }
  else {
    static_assert(sizeof(T) == 4 || sizeof(T) == 8 || sizeof(T) == 16, "sizeof(T) is not supported");
  }
}
```
- **EN:** Defines or forwards `copy_stsm` as part of this header's executable interface.
- **CN:** 定义或转发 `copy_stsm`，作为该头文件可执行接口的一部分。

### Lines 187-208
```cpp
template <class T>
CUTE_HOST_DEVICE
void
copy_stsm_trans(T const* const rmem_ptr,
                uint128_t* const smem_ptr)
{
  uint32_t const* reg_ptr = reinterpret_cast<uint32_t const*>(rmem_ptr);

  // if constexpr
  if (sizeof(T) == 4) {
    SM90_U16x2_STSM_T::copy(reg_ptr[0], smem_ptr[0]);
  }
  else if (sizeof(T) == 8) {
    SM90_U16x4_STSM_T::copy(reg_ptr[0], reg_ptr[1], smem_ptr[0]);
  }
  else if (sizeof(T) == 16) {
    SM90_U16x8_STSM_T::copy(reg_ptr[0], reg_ptr[1], reg_ptr[2], reg_ptr[3], smem_ptr[0]);
  }
  else {
    static_assert(sizeof(T) == 4 || sizeof(T) == 8 || sizeof(T) == 16, "sizeof(T) is not supported");
  }
}
```
- **EN:** Defines or forwards `copy_stsm_trans` as part of this header's executable interface.
- **CN:** 定义或转发 `copy_stsm_trans`，作为该头文件可执行接口的一部分。

### Lines 210-212
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

} // end namespace cute
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 214-217
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

#include <cute/arch/copy_sm90_desc.hpp>
#include <cute/arch/copy_sm90_tma.hpp>
```
- **EN:** Imports `cute/arch/copy_sm90_desc.hpp` (related definitions from `cute/arch/copy_sm90_desc.hpp`); `cute/arch/copy_sm90_tma.hpp` (related definitions from `cute/arch/copy_sm90_tma.hpp`).
- **CN:** 引入 `cute/arch/copy_sm90_desc.hpp`（来自 `cute/arch/copy_sm90_desc.hpp` 的相关定义）；`cute/arch/copy_sm90_tma.hpp`（来自 `cute/arch/copy_sm90_tma.hpp` 的相关定义）。

### Lines 219-219
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Section comment introducing the next logical part of the file: 
- **CN:** 分节注释，用来引出文件中的下一段逻辑：

## Key Concepts / 关键概念

- **EN:** Copy traits/policies separate abstract data movement from the concrete instruction selected underneath.
  **CN:** Copy trait/策略把抽象数据搬运与底层实际选用的指令解耦。
- **EN:** TMA-related code packages descriptors, barriers, swizzles, and coordinates for bulk memory movement.
  **CN:** TMA 相关代码会封装描述符、屏障、swizzle 与坐标，以支持批量内存搬运。
- **EN:** Inline assembly keeps the mapping between C++ fragments and hardware registers explicit.
  **CN:** 内联汇编使 C++ 片段与硬件寄存器之间的映射保持显式。
- **EN:** Compile-time assertions encode hardware and type constraints directly in the API surface.
  **CN:** 编译期断言把硬件与类型约束直接编码到 API 表面。

## Dependencies / 依赖关系

- **EN:** `cute/config.hpp` supplies core CuTe configuration macros and portability annotations.
  **CN:** `cute/config.hpp` 提供了CuTe 核心配置宏与可移植性标注。
- **EN:** `cute/arch/config.hpp` supplies related definitions from `cute/arch/config.hpp`.
  **CN:** `cute/arch/config.hpp` 提供了来自 `cute/arch/config.hpp` 的相关定义。
- **EN:** `cute/arch/copy.hpp` supplies generic low-level copy primitives and policies.
  **CN:** `cute/arch/copy.hpp` 提供了通用底层拷贝原语与策略。
- **EN:** `cute/arch/copy_sm90_desc.hpp` supplies related definitions from `cute/arch/copy_sm90_desc.hpp`.
  **CN:** `cute/arch/copy_sm90_desc.hpp` 提供了来自 `cute/arch/copy_sm90_desc.hpp` 的相关定义。
- **EN:** `cute/arch/copy_sm90_tma.hpp` supplies related definitions from `cute/arch/copy_sm90_tma.hpp`.
  **CN:** `cute/arch/copy_sm90_tma.hpp` 提供了来自 `cute/arch/copy_sm90_tma.hpp` 的相关定义。
