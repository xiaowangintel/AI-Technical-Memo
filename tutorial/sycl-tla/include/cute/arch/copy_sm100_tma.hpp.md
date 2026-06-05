# copy_sm100_tma.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/copy_sm100_tma.hpp`
- **EN:** Defines NVIDIA SM100 TMA-oriented copy wrappers, descriptors, and launch helpers.
- **CN:** 定义面向 NVIDIA SM100 TMA 的拷贝封装、描述符与启动辅助工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-34
```cpp
/***************************************************************************************************
 * Copyright (c) 2020 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

### Lines 36-36
```cpp
#include <cute/arch/config.hpp>
```
- **EN:** Imports `cute/arch/config.hpp` (related definitions from `cute/arch/config.hpp`).
- **CN:** 引入 `cute/arch/config.hpp`（来自 `cute/arch/config.hpp` 的相关定义）。

### Lines 38-40
```cpp
#include <cute/arch/copy.hpp>
#include <cute/arch/copy_sm90.hpp>
#include "cutlass/arch/synclog.hpp"
```
- **EN:** Imports `cute/arch/copy.hpp` (generic low-level copy primitives and policies); `cute/arch/copy_sm90.hpp` (related definitions from `cute/arch/copy_sm90.hpp`); `cutlass/arch/synclog.hpp` (CUTLASS synchronization logging hooks).
- **CN:** 引入 `cute/arch/copy.hpp`（通用底层拷贝原语与策略）；`cute/arch/copy_sm90.hpp`（来自 `cute/arch/copy_sm90.hpp` 的相关定义）；`cutlass/arch/synclog.hpp`（CUTLASS 同步日志钩子）。

### Lines 42-43
```cpp
namespace cute
{
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 45-46
```cpp
constexpr uint32_t Sm100MmaPeerBitMask = 0xFEFFFFFF;
constexpr uint64_t Sm100MemDescDefault = uint64_t(0x1000000000000000);
```
- **EN:** Defines compile-time constant or variable template `Sm100MmaPeerBitMask`, which steers traits, specialization, or static policy decisions.
- **CN:** 定义编译期常量或变量模板 `Sm100MmaPeerBitMask`，用于驱动 traits、特化或静态策略决策。

### Lines 48-76
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////
/// UTMA_LOAD : Initiates a TMA copy from global memory to shared memory
////////////////////////////////////////////////////////////////////////////////////////////////////

struct SM100_TMA_2SM_LOAD_1D
{
  CUTE_HOST_DEVICE static void
  copy([[maybe_unused]] void const* desc_ptr, [[maybe_unused]] uint64_t* mbar_ptr, [[maybe_unused]] uint64_t cache_hint,
       [[maybe_unused]] void      * smem_ptr,
       [[maybe_unused]] int32_t const& crd0)
  {
#if defined(CUTE_ARCH_TMA_SM100_ENABLED)
    uint64_t gmem_int_desc = reinterpret_cast<uint64_t>(desc_ptr);
    // Executed by both CTAs. Set peer bit to 0 so that the
    // transaction bytes will update CTA0's barrier.
    uint32_t smem_int_mbar = cast_smem_ptr_to_uint(mbar_ptr) & Sm100MmaPeerBitMask;
    uint32_t smem_int_ptr  = cast_smem_ptr_to_uint(smem_ptr);
    asm volatile (
      "cp.async.bulk.tensor.1d.cta_group::2.shared::cluster.global.mbarrier::complete_tx::bytes.L2::cache_hint"
      " [%0], [%1, {%3}], [%2], %4;"
      :
      : "r"(smem_int_ptr), "l"(gmem_int_desc), "r"(smem_int_mbar),
        "r"(crd0), "l"(cache_hint)
      : "memory");
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use tma without CUTE_ARCH_TMA_SM100_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_1D`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_1D`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 78-102
```cpp
struct SM100_TMA_2SM_LOAD_2D
{
  CUTE_HOST_DEVICE static void
  copy([[maybe_unused]] void const* desc_ptr, [[maybe_unused]] uint64_t* mbar_ptr, [[maybe_unused]] uint64_t cache_hint,
       [[maybe_unused]] void      * smem_ptr,
       [[maybe_unused]] int32_t const& crd0, int32_t const& crd1)
  {
#if defined(CUTE_ARCH_TMA_SM100_ENABLED)
    uint64_t gmem_int_desc = reinterpret_cast<uint64_t>(desc_ptr);
    // Executed by both CTAs. Set peer bit to 0 so that the
    // transaction bytes will update CTA0's barrier.
    uint32_t smem_int_mbar = cast_smem_ptr_to_uint(mbar_ptr) & Sm100MmaPeerBitMask;
    uint32_t smem_int_ptr  = cast_smem_ptr_to_uint(smem_ptr);
    asm volatile (
      "cp.async.bulk.tensor.2d.cta_group::2.shared::cluster.global.mbarrier::complete_tx::bytes.L2::cache_hint"
      " [%0], [%1, {%3, %4}], [%2], %5;"
      :
      : "r"(smem_int_ptr), "l"(gmem_int_desc), "r"(smem_int_mbar),
        "r"(crd0), "r"(crd1), "l"(cache_hint)
      : "memory");
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use tma without CUTE_ARCH_TMA_SM100_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_2D`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_2D`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 104-128
```cpp
struct SM100_TMA_2SM_LOAD_3D
{
  CUTE_HOST_DEVICE static void
  copy([[maybe_unused]] void const* desc_ptr, [[maybe_unused]] uint64_t* mbar_ptr, [[maybe_unused]] uint64_t cache_hint,
       [[maybe_unused]] void      * smem_ptr,
       [[maybe_unused]] int32_t const& crd0, int32_t const& crd1, int32_t const& crd2)
  {
#if defined(CUTE_ARCH_TMA_SM100_ENABLED)
    uint64_t gmem_int_desc = reinterpret_cast<uint64_t>(desc_ptr);
    // Executed by both CTAs. Set peer bit to 0 so that the
    // transaction bytes will update CTA0's barrier.
    uint32_t smem_int_mbar = cast_smem_ptr_to_uint(mbar_ptr) & Sm100MmaPeerBitMask;
    uint32_t smem_int_ptr  = cast_smem_ptr_to_uint(smem_ptr);
    asm volatile (
      "cp.async.bulk.tensor.3d.cta_group::2.shared::cluster.global.mbarrier::complete_tx::bytes.L2::cache_hint"
      " [%0], [%1, {%3, %4, %5}], [%2], %6;"
      :
      : "r"(smem_int_ptr), "l"(gmem_int_desc), "r"(smem_int_mbar),
        "r"(crd0), "r"(crd1), "r"(crd2), "l"(cache_hint)
      : "memory");
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use tma without CUTE_ARCH_TMA_SM100_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_3D`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_3D`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 130-154
```cpp
struct SM100_TMA_2SM_LOAD_4D
{
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint64_t cache_hint,
       void      * smem_ptr,
       int32_t const& crd0, int32_t const& crd1, int32_t const& crd2, int32_t const& crd3)
  {
#if defined(CUTE_ARCH_TMA_SM100_ENABLED)
    uint64_t gmem_int_desc = reinterpret_cast<uint64_t>(desc_ptr);
    // Executed by both CTAs. Set peer bit to 0 so that the
    // transaction bytes will update CTA0's barrier.
    uint32_t smem_int_mbar = cast_smem_ptr_to_uint(mbar_ptr) & Sm100MmaPeerBitMask;
    uint32_t smem_int_ptr  = cast_smem_ptr_to_uint(smem_ptr);
    asm volatile (
      "cp.async.bulk.tensor.4d.cta_group::2.shared::cluster.global.mbarrier::complete_tx::bytes.L2::cache_hint"
      " [%0], [%1, {%3, %4, %5, %6}], [%2], %7;"
      :
      : "r"(smem_int_ptr), "l"(gmem_int_desc), "r"(smem_int_mbar),
        "r"(crd0), "r"(crd1), "r"(crd2), "r"(crd3), "l"(cache_hint)
      : "memory");
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use tma without CUTE_ARCH_TMA_SM100_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_4D`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_4D`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 156-180
```cpp
struct SM100_TMA_2SM_LOAD_5D
{
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint64_t cache_hint,
       void      * smem_ptr,
       int32_t const& crd0, int32_t const& crd1, int32_t const& crd2, int32_t const& crd3, int32_t const& crd4)
  {
#if defined(CUTE_ARCH_TMA_SM100_ENABLED)
    uint64_t gmem_int_desc = reinterpret_cast<uint64_t>(desc_ptr);
    // Executed by both CTAs. Set peer bit to 0 so that the
    // transaction bytes will update CTA0's barrier.
    uint32_t smem_int_mbar = cast_smem_ptr_to_uint(mbar_ptr) & Sm100MmaPeerBitMask;
    uint32_t smem_int_ptr  = cast_smem_ptr_to_uint(smem_ptr);
    asm volatile (
      "cp.async.bulk.tensor.5d.cta_group::2.shared::cluster.global.mbarrier::complete_tx::bytes.L2::cache_hint"
      " [%0], [%1, {%3, %4, %5, %6, %7}], [%2], %8;"
      :
      : "r"(smem_int_ptr), "l"(gmem_int_desc), "r"(smem_int_mbar),
        "r"(crd0), "r"(crd1), "r"(crd2), "r"(crd3), "r"(crd4), "l"(cache_hint)
      : "memory");
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use tma without CUTE_ARCH_TMA_SM100_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_5D`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_5D`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 182-221
```cpp
struct SM100_TMA_2SM_LOAD
{
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint64_t cache_hint,
       void      * smem_ptr,
       int32_t const& crd0)
  {
    return SM100_TMA_2SM_LOAD_1D::copy(desc_ptr, mbar_ptr, cache_hint, smem_ptr, crd0);
  }
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint64_t cache_hint,
       void      * smem_ptr,
       int32_t const& crd0, int32_t const& crd1)
  {
    return SM100_TMA_2SM_LOAD_2D::copy(desc_ptr, mbar_ptr, cache_hint, smem_ptr, crd0, crd1);
  }
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint64_t cache_hint,
       void      * smem_ptr,
       int32_t const& crd0, int32_t const& crd1, int32_t const& crd2)
  {
    return SM100_TMA_2SM_LOAD_3D::copy(desc_ptr, mbar_ptr, cache_hint, smem_ptr, crd0, crd1, crd2);
  }
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint64_t cache_hint,
       void      * smem_ptr,
       int32_t const& crd0, int32_t const& crd1, int32_t const& crd2, int32_t const& crd3)
  {
    return SM100_TMA_2SM_LOAD_4D::copy(desc_ptr, mbar_ptr, cache_hint, smem_ptr, crd0, crd1, crd2, crd3);
  }
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint64_t cache_hint,
       void      * smem_ptr,
       int32_t const& crd0, int32_t const& crd1, int32_t const& crd2, int32_t const& crd3, int32_t const& crd4)
  {
    return SM100_TMA_2SM_LOAD_5D::copy(desc_ptr, mbar_ptr, cache_hint, smem_ptr, crd0, crd1, crd2, crd3, crd4);
  }

  using PREFETCH = typename SM90_TMA_LOAD::PREFETCH;
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 225-253
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////
/// TMA_LOAD_MULTICAST: Initiates a TMA copy from global memory to shared memory
////////////////////////////////////////////////////////////////////////////////////////////////////

struct SM100_TMA_2SM_LOAD_MULTICAST_1D
{
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint16_t multicast_mask, uint64_t cache_hint,
       void      * smem_ptr,
       int32_t const& crd0)
  {
#if defined(CUTE_ARCH_TMA_SM100_ENABLED)
    uint64_t gmem_int_desc = reinterpret_cast<uint64_t>(desc_ptr);
    // Executed by both CTAs. Set peer bit to 0 so that the
    // transaction bytes will update CTA0's barrier.
    uint32_t smem_int_mbar = cast_smem_ptr_to_uint(mbar_ptr) & Sm100MmaPeerBitMask;
    uint32_t smem_int_ptr  = cast_smem_ptr_to_uint(smem_ptr);
    asm volatile (
      "cp.async.bulk.tensor.1d.cta_group::2.shared::cluster.global.mbarrier::complete_tx::bytes.multicast::cluster.L2::cache_hint"
      " [%0], [%1, {%4}], [%2], %3, %5;"
      :
      : "r"(smem_int_ptr), "l"(gmem_int_desc), "r"(smem_int_mbar), "h"(multicast_mask),
        "r"(crd0), "l"(cache_hint)
      : "memory");
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use tma without CUTE_ARCH_TMA_SM100_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_MULTICAST_1D`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_MULTICAST_1D`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 255-279
```cpp
struct SM100_TMA_2SM_LOAD_MULTICAST_2D
{
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint16_t multicast_mask, uint64_t cache_hint,
       void      * smem_ptr,
       int32_t const& crd0, int32_t const& crd1)
  {
#if defined(CUTE_ARCH_TMA_SM100_ENABLED)
    uint64_t gmem_int_desc = reinterpret_cast<uint64_t>(desc_ptr);
    // Executed by both CTAs. Set peer bit to 0 so that the
    // transaction bytes will update CTA0's barrier.
    uint32_t smem_int_mbar = cast_smem_ptr_to_uint(mbar_ptr) & Sm100MmaPeerBitMask;
    uint32_t smem_int_ptr  = cast_smem_ptr_to_uint(smem_ptr);
    asm volatile (
      "cp.async.bulk.tensor.2d.cta_group::2.shared::cluster.global.mbarrier::complete_tx::bytes.multicast::cluster.L2::cache_hint"
      " [%0], [%1, {%4, %5}], [%2], %3, %6;"
      :
      : "r"(smem_int_ptr), "l"(gmem_int_desc), "r"(smem_int_mbar), "h"(multicast_mask),
        "r"(crd0), "r"(crd1), "l"(cache_hint)
      : "memory");
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use tma without CUTE_ARCH_TMA_SM100_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_MULTICAST_2D`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_MULTICAST_2D`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 281-305
```cpp
struct SM100_TMA_2SM_LOAD_MULTICAST_3D
{
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint16_t multicast_mask, uint64_t cache_hint,
       void      * smem_ptr,
       int32_t const& crd0, int32_t const& crd1, int32_t const& crd2)
  {
#if defined(CUTE_ARCH_TMA_SM100_ENABLED)
uint64_t gmem_int_desc = reinterpret_cast<uint64_t>(desc_ptr);
    // Executed by both CTAs. Set peer bit to 0 so that the
    // transaction bytes will update CTA0's barrier.
    uint32_t smem_int_mbar = cast_smem_ptr_to_uint(mbar_ptr) & Sm100MmaPeerBitMask;
    uint32_t smem_int_ptr  = cast_smem_ptr_to_uint(smem_ptr);
    asm volatile (
      "cp.async.bulk.tensor.3d.cta_group::2.shared::cluster.global.mbarrier::complete_tx::bytes.multicast::cluster.L2::cache_hint"
      " [%0], [%1, {%4, %5, %6}], [%2], %3, %7;"
      :
      : "r"(smem_int_ptr), "l"(gmem_int_desc), "r"(smem_int_mbar), "h"(multicast_mask),
        "r"(crd0), "r"(crd1), "r"(crd2), "l"(cache_hint)
      : "memory");
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use tma without CUTE_ARCH_TMA_SM100_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_MULTICAST_3D`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_MULTICAST_3D`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 307-331
```cpp
struct SM100_TMA_2SM_LOAD_MULTICAST_4D
{
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint16_t multicast_mask, uint64_t cache_hint,
       void      * smem_ptr,
       int32_t const& crd0, int32_t const& crd1, int32_t const& crd2, int32_t const& crd3)
  {
#if defined(CUTE_ARCH_TMA_SM100_ENABLED)
    uint64_t gmem_int_desc = reinterpret_cast<uint64_t>(desc_ptr);
    // Executed by both CTAs. Set peer bit to 0 so that the
    // transaction bytes will update CTA0's barrier.
    uint32_t smem_int_mbar = cast_smem_ptr_to_uint(mbar_ptr) & Sm100MmaPeerBitMask;
    uint32_t smem_int_ptr  = cast_smem_ptr_to_uint(smem_ptr);
    asm volatile (
      "cp.async.bulk.tensor.4d.cta_group::2.shared::cluster.global.mbarrier::complete_tx::bytes.multicast::cluster.L2::cache_hint"
      " [%0], [%1, {%4, %5, %6, %7}], [%2], %3, %8;"
      :
      : "r"(smem_int_ptr), "l"(gmem_int_desc), "r"(smem_int_mbar), "h"(multicast_mask),
        "r"(crd0), "r"(crd1), "r"(crd2),  "r"(crd3), "l"(cache_hint)
      : "memory");
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use tma without CUTE_ARCH_TMA_SM100_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_MULTICAST_4D`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_MULTICAST_4D`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 333-357
```cpp
struct SM100_TMA_2SM_LOAD_MULTICAST_5D
{
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint16_t multicast_mask, uint64_t cache_hint,
       void      * smem_ptr,
       int32_t const& crd0, int32_t const& crd1, int32_t const& crd2, int32_t const& crd3, int32_t const& crd4)
  {
#if defined(CUTE_ARCH_TMA_SM100_ENABLED)
    uint64_t gmem_int_desc = reinterpret_cast<uint64_t>(desc_ptr);
    // Executed by both CTAs. Set peer bit to 0 so that the
    // transaction bytes will update CTA0's barrier.
    uint32_t smem_int_mbar = cast_smem_ptr_to_uint(mbar_ptr) & Sm100MmaPeerBitMask;
    uint32_t smem_int_ptr  = cast_smem_ptr_to_uint(smem_ptr);
    asm volatile (
      "cp.async.bulk.tensor.5d.cta_group::2.shared::cluster.global.mbarrier::complete_tx::bytes.multicast::cluster.L2::cache_hint"
      " [%0], [%1, {%4, %5, %6, %7, %8}], [%2], %3, %9;"
      :
      : "r"(smem_int_ptr), "l"(gmem_int_desc), "r"(smem_int_mbar), "h"(multicast_mask),
        "r"(crd0), "r"(crd1), "r"(crd2), "r"(crd3), "r"(crd4), "l"(cache_hint)
      : "memory");
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use tma without CUTE_ARCH_TMA_SM0_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_MULTICAST_5D`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_MULTICAST_5D`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 359-399
```cpp
struct SM100_TMA_2SM_LOAD_MULTICAST
{
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint16_t multicast_mask, uint64_t cache_hint,
       void      * smem_ptr,
       int32_t const& crd0)
  {
    return SM100_TMA_2SM_LOAD_MULTICAST_1D::copy(desc_ptr, mbar_ptr, multicast_mask, cache_hint, smem_ptr, crd0);
  }
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint16_t multicast_mask, uint64_t cache_hint,
       void      * smem_ptr,
       int32_t const& crd0, int32_t const& crd1)
  {
    return SM100_TMA_2SM_LOAD_MULTICAST_2D::copy(desc_ptr, mbar_ptr, multicast_mask, cache_hint, smem_ptr, crd0, crd1);
  }
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint16_t multicast_mask, uint64_t cache_hint,
       void      * smem_ptr,
       int32_t const& crd0, int32_t const& crd1, int32_t const& crd2)
  {
    return SM100_TMA_2SM_LOAD_MULTICAST_3D::copy(desc_ptr, mbar_ptr, multicast_mask, cache_hint, smem_ptr, crd0, crd1, crd2);
  }
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint16_t multicast_mask, uint64_t cache_hint,
       void      * smem_ptr,
       int32_t const& crd0, int32_t const& crd1, int32_t const& crd2, int32_t const& crd3)
  {
    return SM100_TMA_2SM_LOAD_MULTICAST_4D::copy(desc_ptr, mbar_ptr, multicast_mask, cache_hint, smem_ptr, crd0, crd1, crd2, crd3);
  }
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint16_t multicast_mask, uint64_t cache_hint,
       void      * smem_ptr,
       int32_t const& crd0, int32_t const& crd1, int32_t const& crd2, int32_t const& crd3, int32_t const& crd4)
  {
    return SM100_TMA_2SM_LOAD_MULTICAST_5D::copy(desc_ptr, mbar_ptr, multicast_mask, cache_hint, smem_ptr, crd0, crd1, crd2, crd3, crd4);
  }

  using PREFETCH = typename SM90_TMA_LOAD::PREFETCH;
};
////////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_MULTICAST`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_MULTICAST`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 402-428
```cpp
struct SM100_TMA_2SM_LOAD_IM2COL_3D
{
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr,
       void      * smem_ptr,
       int32_t const& coord_c, int32_t const& coord_w, int32_t const& coord_n,
       uint16_t const& offset_w)
  {
#if defined(CUTE_ARCH_TMA_SM100_ENABLED)
    uint64_t gmem_int_desc = reinterpret_cast<uint64_t>(desc_ptr);
    // Executed by both CTAs. Set peer bit to 0 so that the
    // transaction bytes will update CTA0's barrier.
    uint32_t smem_int_mbar = cast_smem_ptr_to_uint(mbar_ptr) & Sm100MmaPeerBitMask;
    uint32_t smem_int_ptr  = cast_smem_ptr_to_uint(smem_ptr);
    asm volatile (
      "cp.async.bulk.tensor.3d.im2col.cta_group::2.shared::cluster.global.mbarrier::complete_tx::bytes.L2::cache_hint"
      " [%0], [%1, {%3, %4, %5}], [%2], {%6}, %7;"
      :
      : "r"(smem_int_ptr), "l"(gmem_int_desc), "r"(smem_int_mbar),
        "r"(coord_c), "r"(coord_w), "r"(coord_n),
        "h"(offset_w), "l"(Sm100MemDescDefault)
      : "memory");
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use tma without CUTE_ARCH_TMA_SM100_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_IM2COL_3D`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_IM2COL_3D`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 430-457
```cpp
struct SM100_TMA_2SM_LOAD_IM2COL_4D
{
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr,
       void      * smem_ptr,
       int32_t const& coord_c, int32_t const& coord_w, int32_t const& coord_h, int32_t const& coord_n,
       uint16_t const& offset_w,
       uint16_t const& offset_h)
  {
#if defined(CUTE_ARCH_TMA_SM100_ENABLED)
    uint64_t gmem_int_desc = reinterpret_cast<uint64_t>(desc_ptr);
    // Executed by both CTAs. Set peer bit to 0 so that the
    // transaction bytes will update CTA0's barrier.
    uint32_t smem_int_mbar = cast_smem_ptr_to_uint(mbar_ptr) & Sm100MmaPeerBitMask;
    uint32_t smem_int_ptr  = cast_smem_ptr_to_uint(smem_ptr);
    asm volatile (
      "cp.async.bulk.tensor.4d.im2col.cta_group::2.shared::cluster.global.mbarrier::complete_tx::bytes.L2::cache_hint"
      " [%0], [%1, {%3, %4, %5, %6}], [%2], {%7, %8}, %9;"
      :
      : "r"(smem_int_ptr), "l"(gmem_int_desc), "r"(smem_int_mbar),
        "r"(coord_c), "r"(coord_w), "r"(coord_h), "r"(coord_n),
        "h"(offset_w), "h"(offset_h), "l"(Sm100MemDescDefault)
      : "memory");
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use tma without CUTE_ARCH_TMA_SM100_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_IM2COL_4D`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_IM2COL_4D`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 459-487
```cpp
struct SM100_TMA_2SM_LOAD_IM2COL_5D
{
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr,
       void      * smem_ptr,
       int32_t const& coord_c, int32_t const& coord_w, int32_t const& coord_h, int32_t const& coord_d, int32_t const& coord_n,
       uint16_t const& offset_w,
       uint16_t const& offset_h,
       uint16_t const& offset_d)
  {
#if defined(CUTE_ARCH_TMA_SM100_ENABLED)
    uint64_t gmem_int_desc = reinterpret_cast<uint64_t>(desc_ptr);
    // Executed by both CTAs. Set peer bit to 0 so that the
    // transaction bytes will update CTA0's barrier.
    uint32_t smem_int_mbar = cast_smem_ptr_to_uint(mbar_ptr) & Sm100MmaPeerBitMask;
    uint32_t smem_int_ptr  = cast_smem_ptr_to_uint(smem_ptr);
    asm volatile (
      "cp.async.bulk.tensor.5d.im2col.cta_group::2.shared::cluster.global.mbarrier::complete_tx::bytes.L2::cache_hint"
      " [%0], [%1, {%3, %4, %5, %6, %7}], [%2], {%8, %9, %10}, %11;"
      :
      : "r"(smem_int_ptr), "l"(gmem_int_desc), "r"(smem_int_mbar),
        "r"(coord_c), "r"(coord_w), "r"(coord_h), "r"(coord_d), "r"(coord_n),
        "h"(offset_w), "h"(offset_h), "h"(offset_d), "l"(Sm100MemDescDefault)
      : "memory");
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use tma without CUTE_ARCH_TMA_SM100_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_IM2COL_5D`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_IM2COL_5D`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 489-526
```cpp
struct SM100_TMA_2SM_LOAD_IM2COL
{
 CUTE_HOST_DEVICE static void
 copy(void const* desc_ptr, uint64_t* mbar_ptr,
      void      * smem_ptr,
      int32_t const& coord_c, int32_t const& coord_w, int32_t const& coord_n,
      uint16_t const& offset_w)
 {
   return SM100_TMA_2SM_LOAD_IM2COL_3D::copy(desc_ptr, mbar_ptr, smem_ptr,
                                             coord_c, coord_w, coord_n,
                                             offset_w);
 }
 CUTE_HOST_DEVICE static void
 copy(void const* desc_ptr, uint64_t* mbar_ptr,
      void      * smem_ptr,
      int32_t const& coord_c, int32_t const& coord_w, int32_t const& coord_h, int32_t const& coord_n,
      uint16_t const& offset_w,
      uint16_t const& offset_h)
 {
   return SM100_TMA_2SM_LOAD_IM2COL_4D::copy(desc_ptr, mbar_ptr, smem_ptr,
                                             coord_c, coord_w, coord_h, coord_n,
                                             offset_w, offset_h);
 }
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr,
       void      * smem_ptr,
       int32_t const& coord_c, int32_t const& coord_w, int32_t const& coord_h, int32_t const& coord_d, int32_t const& coord_n,
       uint16_t const& offset_w,
       uint16_t const& offset_h,
       uint16_t const& offset_d)
  {
    return SM100_TMA_2SM_LOAD_IM2COL_5D::copy(desc_ptr, mbar_ptr, smem_ptr,
                                              coord_c, coord_w, coord_h, coord_d, coord_n,
                                              offset_w, offset_h, offset_d);
  }

  using PREFETCH = typename SM90_TMA_LOAD_IM2COL::PREFETCH;
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_IM2COL`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_IM2COL`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 528-558
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

struct SM100_TMA_2SM_LOAD_IM2COL_MULTICAST_3D
{
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint16_t multicast_mask,
       void      * smem_ptr,
       int32_t const& coord_c, int32_t const& coord_w, int32_t const& coord_n,
       uint16_t const& offset_w)
  {
#if defined(CUTE_ARCH_TMA_SM100_ENABLED)
    uint64_t gmem_int_desc = reinterpret_cast<uint64_t>(desc_ptr);
    // Executed by both CTAs. Set peer bit to 0 so that the
    // transaction bytes will update CTA0's barrier.
    uint32_t smem_int_mbar = cast_smem_ptr_to_uint(mbar_ptr) & Sm100MmaPeerBitMask;
    uint32_t smem_int_ptr  = cast_smem_ptr_to_uint(smem_ptr);
    asm volatile (
      "cp.async.bulk.tensor.3d.im2col.cta_group::2.shared::cluster.global.mbarrier::complete_tx::bytes.multicast::cluster.L2::cache_hint"
      " [%0], [%1, {%3, %4, %5}], [%2], {%6}, %7, %8;"
      :
      : "r"(smem_int_ptr), "l"(gmem_int_desc), "r"(smem_int_mbar),
        "r"(coord_c), "r"(coord_w), "r"(coord_n),
        "h"(offset_w),
        "h"(multicast_mask),
        "l"(Sm100MemDescDefault)
      : "memory");
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use tma without CUTE_ARCH_TMA_SM100_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_IM2COL_MULTICAST_3D`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_IM2COL_MULTICAST_3D`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 560-589
```cpp
struct SM100_TMA_2SM_LOAD_IM2COL_MULTICAST_4D
{
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint16_t multicast_mask,
       void      * smem_ptr,
       int32_t const& coord_c, int32_t const& coord_w, int32_t const& coord_h, int32_t const& coord_n,
       uint16_t const& offset_w,
       uint16_t const& offset_h)
  {
#if defined(CUTE_ARCH_TMA_SM100_ENABLED)
    uint64_t gmem_int_desc = reinterpret_cast<uint64_t>(desc_ptr);
    // Executed by both CTAs. Set peer bit to 0 so that the
    // transaction bytes will update CTA0's barrier.
    uint32_t smem_int_mbar = cast_smem_ptr_to_uint(mbar_ptr) & Sm100MmaPeerBitMask;
    uint32_t smem_int_ptr  = cast_smem_ptr_to_uint(smem_ptr);
    asm volatile (
      "cp.async.bulk.tensor.4d.im2col.cta_group::2.shared::cluster.global.mbarrier::complete_tx::bytes.multicast::cluster.L2::cache_hint"
      " [%0], [%1, {%3, %4, %5, %6}], [%2], {%7, %8}, %9, %10;"
      :
      : "r"(smem_int_ptr), "l"(gmem_int_desc), "r"(smem_int_mbar),
        "r"(coord_c), "r"(coord_w), "r"(coord_h), "r"(coord_n),
        "h"(offset_w), "h"(offset_h),
        "h"(multicast_mask),
        "l"(Sm100MemDescDefault)
      : "memory");
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use tma without CUTE_ARCH_TMA_SM100_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_IM2COL_MULTICAST_4D`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_IM2COL_MULTICAST_4D`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 591-621
```cpp
struct SM100_TMA_2SM_LOAD_IM2COL_MULTICAST_5D
{
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint16_t multicast_mask,
       void      * smem_ptr,
       int32_t const& coord_c, int32_t const& coord_w, int32_t const& coord_h, int32_t const& coord_d, int32_t const& coord_n,
       uint16_t const& offset_w,
       uint16_t const& offset_h,
       uint16_t const& offset_d)
  {
#if defined(CUTE_ARCH_TMA_SM100_ENABLED)
    uint64_t gmem_int_desc = reinterpret_cast<uint64_t>(desc_ptr);
    // Executed by both CTAs. Set peer bit to 0 so that the
    // transaction bytes will update CTA0's barrier.
    uint32_t smem_int_mbar = cast_smem_ptr_to_uint(mbar_ptr) & Sm100MmaPeerBitMask;
    uint32_t smem_int_ptr  = cast_smem_ptr_to_uint(smem_ptr);
    asm volatile (
      "cp.async.bulk.tensor.5d.im2col.cta_group::2.shared::cluster.global.mbarrier::complete_tx::bytes.multicast::cluster.L2::cache_hint"
      " [%0], [%1, {%3, %4, %5, %6, %7}], [%2], {%8, %9, %10}, %11, %12;"
      :
      : "r"(smem_int_ptr), "l"(gmem_int_desc), "r"(smem_int_mbar),
        "r"(coord_c), "r"(coord_w), "r"(coord_h), "r"(coord_d), "r"(coord_n),
        "h"(offset_w), "h"(offset_h), "h"(offset_d),
        "h"(multicast_mask),
        "l"(Sm100MemDescDefault)
      : "memory");
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use tma without CUTE_ARCH_TMA_SM100_ENABLED.");
#endif
  }
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_IM2COL_MULTICAST_5D`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_IM2COL_MULTICAST_5D`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 623-660
```cpp
struct SM100_TMA_2SM_LOAD_IM2COL_MULTICAST
{
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint16_t multicast_mask,
       void      * smem_ptr,
       int32_t const& coord_c, int32_t const& coord_w, int32_t const& coord_n,
       uint16_t const& offset_w)
  {
    return SM100_TMA_2SM_LOAD_IM2COL_MULTICAST_3D::copy(desc_ptr, mbar_ptr, multicast_mask,
                                                        smem_ptr,
                                                        coord_c, coord_w, coord_n,
                                                        offset_w);
  }
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint16_t multicast_mask,
       void      * smem_ptr,
       int32_t const& coord_c, int32_t const& coord_w, int32_t const& coord_h, int32_t const& coord_n,
       uint16_t const& offset_w, uint16_t const& offset_h)
  {
    return SM100_TMA_2SM_LOAD_IM2COL_MULTICAST_4D::copy(desc_ptr, mbar_ptr, multicast_mask,
                                                        smem_ptr,
                                                        coord_c, coord_w, coord_h, coord_n,
                                                        offset_w, offset_h);
  }
  CUTE_HOST_DEVICE static void
  copy(void const* desc_ptr, uint64_t* mbar_ptr, uint16_t multicast_mask,
       void      * smem_ptr,
       int32_t const& coord_c, int32_t const& coord_w, int32_t const& coord_h, int32_t const& coord_d, int32_t const& coord_n,
       uint16_t const& offset_w, uint16_t const& offset_h, uint16_t const& offset_d)
  {
    return SM100_TMA_2SM_LOAD_IM2COL_MULTICAST_5D::copy(desc_ptr, mbar_ptr, multicast_mask,
                                                        smem_ptr,
                                                        coord_c, coord_w, coord_h, coord_d, coord_n,
                                                        offset_w, offset_h, offset_d);
  }

  using PREFETCH = typename SM90_TMA_LOAD_IM2COL::PREFETCH;
};
```
- **EN:** Defines `SM100_TMA_2SM_LOAD_IM2COL_MULTICAST`, storing tensor-memory-accelerator state, descriptors, or helper parameters.
- **CN:** 定义 `SM100_TMA_2SM_LOAD_IM2COL_MULTICAST`，用于保存 TMA（Tensor Memory Accelerator）状态、描述符或辅助参数。

### Lines 663-666
```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////

} // end namespace cute
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

## Key Concepts / 关键概念

- **EN:** Copy traits/policies separate abstract data movement from the concrete instruction selected underneath.
  **CN:** Copy trait/策略把抽象数据搬运与底层实际选用的指令解耦。
- **EN:** TMA-related code packages descriptors, barriers, swizzles, and coordinates for bulk memory movement.
  **CN:** TMA 相关代码会封装描述符、屏障、swizzle 与坐标，以支持批量内存搬运。
- **EN:** Inline assembly keeps the mapping between C++ fragments and hardware registers explicit.
  **CN:** 内联汇编使 C++ 片段与硬件寄存器之间的映射保持显式。
- **EN:** Prefetch paths try to reduce latency by staging data or metadata early.
  **CN:** 预取路径尝试通过提前准备数据或元数据来降低延迟。

## Dependencies / 依赖关系

- **EN:** `cute/arch/config.hpp` supplies related definitions from `cute/arch/config.hpp`.
  **CN:** `cute/arch/config.hpp` 提供了来自 `cute/arch/config.hpp` 的相关定义。
- **EN:** `cute/arch/copy.hpp` supplies generic low-level copy primitives and policies.
  **CN:** `cute/arch/copy.hpp` 提供了通用底层拷贝原语与策略。
- **EN:** `cute/arch/copy_sm90.hpp` supplies related definitions from `cute/arch/copy_sm90.hpp`.
  **CN:** `cute/arch/copy_sm90.hpp` 提供了来自 `cute/arch/copy_sm90.hpp` 的相关定义。
- **EN:** `cutlass/arch/synclog.hpp` supplies CUTLASS synchronization logging hooks.
  **CN:** `cutlass/arch/synclog.hpp` 提供了CUTLASS 同步日志钩子。
- **EN:** CUTLASS headers provide adapter types, synchronization hooks, or shared low-level GPU infrastructure.
  **CN:** CUTLASS 头文件提供适配类型、同步钩子或共享的底层 GPU 基础设施。
