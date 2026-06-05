# tmem_allocator_sm100.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/tmem_allocator_sm100.hpp`
- **EN:** Implements SM100 tensor-memory allocation helpers and bookkeeping utilities.
- **CN:** 实现 SM100 tensor memory 分配辅助与簿记工具。

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

### Lines 34-37
```cpp
#include <cute/arch/config.hpp>
#include <cute/arch/util.hpp>
#include <cute/numeric/integral_constant.hpp>
#include <cute/pointer.hpp>
```
- **EN:** Imports `cute/arch/config.hpp` (related definitions from `cute/arch/config.hpp`); `cute/arch/util.hpp` (architecture utility helpers and low-level support macros); `cute/numeric/integral_constant.hpp` (related definitions from `cute/numeric/integral_constant.hpp`); `cute/pointer.hpp` (related definitions from `cute/pointer.hpp`).
- **CN:** 引入 `cute/arch/config.hpp`（来自 `cute/arch/config.hpp` 的相关定义）；`cute/arch/util.hpp`（体系结构工具辅助与底层支持宏）；`cute/numeric/integral_constant.hpp`（来自 `cute/numeric/integral_constant.hpp` 的相关定义）；`cute/pointer.hpp`（来自 `cute/pointer.hpp` 的相关定义）。

### Lines 39-39
```cpp
namespace cute::TMEM {
```
- **EN:** Enters or leaves namespace scope `cute::TMEM` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute::TMEM`，以便把相关符号组织在一起。

### Lines 41-46
```cpp
//
// TMEM Addressing Constants
//

// 128 DP x 512 COL x uint32_t-addressing
using MAX_CAPACITY_BITS = Int<128*512*32>;
```
- **EN:** Introduces the alias `MAX_CAPACITY_BITS` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `MAX_CAPACITY_BITS`，便于复用周边的寄存器、布局或策略类型。

### Lines 48-49
```cpp
// TMEM DP stride in bit-addressing (shift by 5 for conversion from uint32_t)
using DP_b = cute::constant<int32_t, (1 << 21)>;
```
- **EN:** Introduces the alias `DP_b` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `DP_b`，便于复用周边的寄存器、布局或策略类型。

### Lines 51-53
```cpp
// TMEM DP stride in type-T addressing
template <class T = uint32_t>
using DP = cute::constant<int32_t, shiftl((1 << 16), tmem_ptr<T>::OffsetShift)>;
```
- **EN:** Introduces the alias `DP` to make the surrounding register, layout, or policy type easier to reuse.
- **CN:** 引入别名 `DP`，便于复用周边的寄存器、布局或策略类型。

### Lines 55-111
```cpp
//
// TMEM Allocators
//

// All operations of this class require that only a single warp uniformly participates
class Allocator1Sm {
public:
  static constexpr int ColumnsPerAllocationSlice = 32;
  static constexpr int Sm100TmemCapacityColumns = 512;

  CUTLASS_DEVICE Allocator1Sm() { }

  /**
   * Performs a non-blocking allocation of TMEM.
   * @param num_columns Number of columns being freed. Must be 32 <= num_columns <= 512 and power of 2.
   * @param dst_ptr Pointer to shared memory to which to write the result tmem pointer to.
   * @pre Must be issued by a single fully active warp of the CTA.
   * @pre Must never be issued by more than one warp at the same time.
   * @pre For repeated allocations, the same warp must be used to issue all allocations.
  **/
  CUTE_HOST_DEVICE void
  allocate(int num_columns, uint32_t* dst_ptr) {
  #if defined(CUTE_ARCH_TCGEN05_TMEM_ENABLED)
    uint32_t dst_intptr = cute::cast_smem_ptr_to_uint(dst_ptr);
    asm volatile(
      "tcgen05.alloc.cta_group::1.sync.aligned.shared::cta.b32 [%0], %1;"
      :
      : "r"(dst_intptr), "r"(num_columns));
  #else
    CUTE_INVALID_CONTROL_PATH("Attempting to use TMEM allocation PTX without CUTE_ARCH_TCGEN05_TMEM_ENABLED");
  #endif
  }

  CUTLASS_DEVICE
  void
  free(uint32_t tmem_ptr, int num_columns) {
  #if defined(CUTE_ARCH_TCGEN05_TMEM_ENABLED)
    asm volatile(
      "{\n\t"
      "tcgen05.dealloc.cta_group::1.sync.aligned.b32  %0, %1; \n\t"
      "}"
      :
      : "r"(tmem_ptr), "r"(num_columns));
  #else
    CUTE_INVALID_CONTROL_PATH("Attempting to use TMEM allocation PTX without CUTE_ARCH_TCGEN05_TMEM_ENABLED");
  #endif
  }

  CUTLASS_DEVICE void
  release_allocation_lock() {
  #if defined(CUTE_ARCH_TCGEN05_TMEM_ENABLED)
    asm volatile("tcgen05.relinquish_alloc_permit.cta_group::1.sync.aligned;" ::);
  #else
    CUTE_INVALID_CONTROL_PATH("Attempting to use TMEM allocation PTX without CUTE_ARCH_TCGEN05_TMEM_ENABLED");
  #endif
  }
};
```
- **EN:** Defines `Allocator1Sm` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Allocator1Sm`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 113-181
```cpp
///////////////////////////////////////////////////////////////////////////////
///////////////////////////////////////////////////////////////////////////////
///////////////////////////////////////////////////////////////////////////////

class Allocator2Sm {
public:
  static constexpr int ColumnsPerAllocationSlice = 32;
  static constexpr int Sm100TmemCapacityColumns = 512;

  CUTLASS_DEVICE Allocator2Sm() { }

  /**
   * Performs a non-blocking allocation of TMEM.
   * @param num_columns Number of columns being freed. Must be 32 <= num_columns <= 512 and power of 2.
   * @param dst_ptr Pointer to shared memory to which to write the result tmem pointer to.
   *   Both CTAs _must_ provide the exact same dst_ptr for correctness.
   * @pre Must be issued by a single fully active warp of the CTA.
   * @pre Must never be issued by more than one warp at the same time.
   * @pre For repeated allocations, the same warp must be used to issue all allocations.
   * @pre The 2 warps from participating CTAs have the same logical warp ID.
  **/
  CUTE_HOST_DEVICE void
  allocate(int num_columns, uint32_t* dst_ptr) {
  #if defined(CUTE_ARCH_TCGEN05_TMEM_ENABLED)
    uint32_t dst_intptr = cute::cast_smem_ptr_to_uint(dst_ptr);
    asm volatile(
      "tcgen05.alloc.cta_group::2.sync.aligned.shared::cta.b32 [%0], %1;"
      :
      : "r"(dst_intptr), "r"(num_columns));
  #else
    CUTE_INVALID_CONTROL_PATH("Attempting to use TMEM allocation PTX without CUTE_ARCH_TCGEN05_TMEM_ENABLED");
  #endif
  }

  /**
   * Frees the TMEM corresponding to the pointer and slice count provided.
   * Release the TMEM after checking that the CTA issuing the free does indeed own the corresponding slices.
   * @param tmem_ptr Base address of the TMEM address space being freed.
   * @param num_columns Number of columns being freed. Must be 32 <= num_columns <= 512 and power of 2.
   * @pre Must be issued by a single fully active warp of the CTA.
   * @pre Must never be issued by more than one warp at the same time.
   * @pre The 2 warps from participating CTAs have the same logical warp ID.
   * @returns true
  **/
  CUTLASS_DEVICE
  void
  free(uint32_t tmem_ptr, int num_columns) {
  #if defined(CUTE_ARCH_TCGEN05_TMEM_ENABLED)
    asm volatile(
      "{\n\t"
      "tcgen05.dealloc.cta_group::2.sync.aligned.b32  %0, %1; \n\t"
      "}"
      :
      : "r"(tmem_ptr), "r"(num_columns));
  #else
    CUTE_INVALID_CONTROL_PATH("Attempting to use TMEM allocation PTX without CUTE_ARCH_TCGEN05_TMEM_ENABLED");
  #endif
  }

  CUTLASS_DEVICE
  void
  release_allocation_lock() {
  #if defined(CUTE_ARCH_TCGEN05_TMEM_ENABLED)
    asm volatile("tcgen05.relinquish_alloc_permit.cta_group::2.sync.aligned;" ::);
  #else
    CUTE_INVALID_CONTROL_PATH("Attempting to use TMEM allocation PTX without CUTE_ARCH_TCGEN05_TMEM_ENABLED");
  #endif
  }
};
```
- **EN:** Defines `Allocator2Sm` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `Allocator2Sm`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 183-183
```cpp
} // namespace cute::TMEM
```
- **EN:** Enters or leaves namespace scope `cute::TMEM` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute::TMEM`，以便把相关符号组织在一起。

## Key Concepts / 关键概念

- **EN:** Inline assembly keeps the mapping between C++ fragments and hardware registers explicit.
  **CN:** 内联汇编使 C++ 片段与硬件寄存器之间的映射保持显式。

## Dependencies / 依赖关系

- **EN:** `cute/arch/config.hpp` supplies related definitions from `cute/arch/config.hpp`.
  **CN:** `cute/arch/config.hpp` 提供了来自 `cute/arch/config.hpp` 的相关定义。
- **EN:** `cute/arch/util.hpp` supplies architecture utility helpers and low-level support macros.
  **CN:** `cute/arch/util.hpp` 提供了体系结构工具辅助与底层支持宏。
- **EN:** `cute/numeric/integral_constant.hpp` supplies related definitions from `cute/numeric/integral_constant.hpp`.
  **CN:** `cute/numeric/integral_constant.hpp` 提供了来自 `cute/numeric/integral_constant.hpp` 的相关定义。
- **EN:** `cute/pointer.hpp` supplies related definitions from `cute/pointer.hpp`.
  **CN:** `cute/pointer.hpp` 提供了来自 `cute/pointer.hpp` 的相关定义。
