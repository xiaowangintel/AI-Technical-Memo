# copy_xe.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/copy_xe.hpp`
- **EN:** Defines low-level copy instructions and wrappers for Intel Xe or generic backends.
- **CN:** 为 Intel Xe 或通用后端定义底层拷贝指令与封装。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2024 Codeplay Software Ltd. All rights reserved.
 * Copyright (C) 2025 - 2026 Intel Corporation, All rights reserved.
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

### Lines 35-36
```cpp
namespace cute
{
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 38-57
```cpp
template<class S, class D = S>
struct XE_ATOMIC {
  using SRegisters = S[1];
  using DRegisters = D[1];

  CUTE_STATIC_ASSERT(is_same_v<S, float> || is_same_v<S, double> || is_same_v<S, int>);

  template<class S_, class D_>
  CUTE_HOST_DEVICE static void
  copy(S_ const& src, D_ & dst) {
    #if defined(SYCL_INTEL_TARGET)
      auto v = sycl::atomic_ref<D_, sycl::memory_order::relaxed,
                                  sycl::memory_scope::device,
                                  sycl::access::address_space::global_space>(*&dst);
      v += static_cast<D_>(*&src);
    #else
      CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
    #endif
  }
};
```
- **EN:** Defines `XE_ATOMIC` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_ATOMIC`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 59-116
```cpp
template <class S, class D = S>
struct XE_1D_LDSM {
  using SRegisters = S[1];
  using DRegisters = D[1];

  CUTE_STATIC_ASSERT(sizeof(D) >= sizeof(S), "Dst must be same size or larger than src for slm->reg copy");

  template<class S_, class D_>
  CUTE_HOST_DEVICE static void
  copy(const S_ &src, D_ &dst) {
    #if defined(__SYCL_DEVICE_ONLY__) && defined(SYCL_INTEL_TARGET)
      uint32_t smem_int_ptr = cast_smem_ptr_to_uint(&src);
      if constexpr (sizeof(D) >= sizeof(uint32_t)) {
        // 32-bit path: use d32 / d32xN
        static constexpr size_t N_d32 = sizeof(D) / sizeof(uint32_t);
        using StorageT = intel::vector_t<uint32_t, N_d32>;
        auto& data_vec = *reinterpret_cast<StorageT *>(&dst);
        if constexpr (N_d32 > 1) {
          asm volatile(
              "{\n"
              "lsc_load.slm (M1, 16) %0:d32x%2 flat[%1]:a32 \n"
              "}\n" : "=rw"(data_vec)
              : "rw"(smem_int_ptr), "P"(N_d32)
            );
        } else {
          asm volatile(
              "{\n"
              "lsc_load.slm (M1, 16) %0:d32 flat[%1]:a32 \n"
              "}\n" : "=rw"(data_vec)
              : "rw"(smem_int_ptr)
            );
        }
      } else if constexpr (sizeof(D) == sizeof(uint16_t)) {
        // 16-bit path: use d16u32 (load 16-bit, zero-extend to 32-bit in GRF)
        intel::vector_t<uint32_t, 1> data_vec32;
        asm volatile(
            "{\n"
            "lsc_load.slm (M1, 16) %0:d16u32 flat[%1]:a32 \n"
            "}\n" : "=rw"(data_vec32)
            : "rw"(smem_int_ptr)
          );
        *reinterpret_cast<uint16_t*>(&dst) = static_cast<uint16_t>(data_vec32[0]);
      } else {
        // 8-bit path: use d8u32 (load 8-bit, zero-extend to 32-bit in GRF)
        intel::vector_t<uint32_t, 1> data_vec32;
        asm volatile(
            "{\n"
            "lsc_load.slm (M1, 16) %0:d8u32 flat[%1]:a32 \n"
            "}\n" : "=rw"(data_vec32)
            : "rw"(smem_int_ptr)
          );
        *reinterpret_cast<uint8_t*>(&dst) = static_cast<uint8_t>(data_vec32[0]);
      }
    #else
      CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
    #endif
  }
};
```
- **EN:** Defines `XE_1D_LDSM` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_1D_LDSM`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 118-145
```cpp
template <class S, class D = S>
struct XE_1D_LOAD_GLOBAL {
  using SRegisters = S[1];
  using DRegisters = D[1];

  CUTE_STATIC_ASSERT(sizeof(D) % sizeof(S) == 0,
    "dst failed to vectorize into registers");
  static constexpr size_t N = sizeof(D) / sizeof(S);
  CUTE_STATIC_ASSERT(N == 1 || N == 2 || N == 4 || N == 8,
    "register vector only supports 1, 2, 4, 8");

  template<class S_, class D_>
  CUTE_HOST_DEVICE static void
  copy(const S_ &src, D_ &dst) {
    #if defined(SYCL_INTEL_TARGET)
      CUTE_STATIC_ASSERT(sizeof(S_) == sizeof(S));
      CUTE_STATIC_ASSERT(sizeof(D_) == sizeof(D));
      auto sg = sycl::ext::oneapi::this_work_item::get_nd_item<3>().get_sub_group();
      auto props = sycl::ext::oneapi::experimental::properties{
          sycl::ext::oneapi::experimental::contiguous_memory,
          sycl::ext::oneapi::experimental::alignment<sizeof(D)>};
      sycl::ext::oneapi::experimental::group_load(
          sg, &src, *reinterpret_cast<sycl::vec<S_, N> *>(&dst), props);
    #else
      CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
    #endif
  }
};
```
- **EN:** Defines `XE_1D_LOAD_GLOBAL` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_1D_LOAD_GLOBAL`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 147-205
```cpp
template<class S, class D = S>
struct XE_1D_STSM {
  using SRegisters = S[1];
  using DRegisters = D[1];

  CUTE_STATIC_ASSERT(sizeof(S) >= sizeof(D), "Src must be same size or larger than dst for reg->slm copy");

  template<class S_, class D_>
  CUTE_HOST_DEVICE static void
  copy(S_ const& src, D_ & dst) {
    #if defined(__SYCL_DEVICE_ONLY__) && defined(SYCL_INTEL_TARGET)
          uint32_t smem_int_ptr = cast_smem_ptr_to_uint(&dst);
          if constexpr (sizeof(S) >= sizeof(uint32_t)) {
            // 32-bit path: use d32 / d32xN
            static constexpr size_t N_d32 = sizeof(S) / sizeof(uint32_t);
            using StorageT = intel::vector_t<uint32_t, N_d32>;
            auto& data_vec = *reinterpret_cast<StorageT const*>(&src);
            if constexpr (N_d32 > 1) {
              asm volatile(
                  "{\n"
                  "lsc_store.slm (M1, 16) flat[%1]:a32 %0:d32x%2 \n"
                  "}\n" ::
                      "rw"(data_vec),
                  "rw"(smem_int_ptr), "P"(N_d32)
                );
            } else {
              asm volatile(
                  "{\n"
                  "lsc_store.slm (M1, 16) flat[%1]:a32 %0:d32 \n"
                  "}\n" ::"rw"(data_vec),
                  "rw"(smem_int_ptr)
                );
            }
          } else if constexpr (sizeof(S) == sizeof(uint16_t)) {
            // 16-bit path: use d16u32 (store lower 16 bits of 32-bit GRF)
            intel::vector_t<uint32_t, 1> data_vec32;
            data_vec32[0] = static_cast<uint32_t>(*reinterpret_cast<const uint16_t*>(&src));
            asm volatile(
                "{\n"
                "lsc_store.slm (M1, 16) flat[%1]:a32 %0:d16u32 \n"
                "}\n" ::"rw"(data_vec32),
                "rw"(smem_int_ptr)
              );
          } else {
            // 8-bit path: use d8u32 (store lower 8 bits of 32-bit GRF)
            intel::vector_t<uint32_t, 1> data_vec32;
            data_vec32[0] = static_cast<uint32_t>(*reinterpret_cast<const uint8_t*>(&src));
            asm volatile(
                "{\n"
                "lsc_store.slm (M1, 16) flat[%1]:a32 %0:d8u32 \n"
                "}\n" ::"rw"(data_vec32),
                "rw"(smem_int_ptr)
              );
          }
    #else
      CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
    #endif
  }
};
```
- **EN:** Defines `XE_1D_STSM` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_1D_STSM`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 207-233
```cpp
template<class S, class D = S>
struct XE_1D_STORE_GLOBAL {
  using SRegisters = S[1];
  using DRegisters = D[1];

  CUTE_STATIC_ASSERT(sizeof(S) % sizeof(D) == 0,
      "src failed to vectorize into registers");
  static constexpr size_t N = sizeof(S) / sizeof(D);
  CUTE_STATIC_ASSERT(N == 1 || N == 2 || N == 4 || N == 8,
      "register vector only supports 1, 2, 4, 8");

  template<class S_, class D_>
  CUTE_HOST_DEVICE static void
  copy(S_ const& src, D_ &dst) {
    #if defined(SYCL_INTEL_TARGET)
      auto sg = sycl::ext::oneapi::this_work_item::get_nd_item<3>().get_sub_group();
      auto props = sycl::ext::oneapi::experimental::properties{
          sycl::ext::oneapi::experimental::contiguous_memory,
          sycl::ext::oneapi::experimental::alignment<sizeof(S)>};
      sycl::ext::oneapi::experimental::group_store(
          sg, *reinterpret_cast<sycl::vec<D_, N> const *>(&src), &dst, props);
    #else
      CUTE_INVALID_CONTROL_PATH("Trying to use block loads on non-Xe hardware");
    #endif
  }
};
} // end namespace cute
```
- **EN:** Defines `XE_1D_STORE_GLOBAL` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_1D_STORE_GLOBAL`，把该头文件这一部分所需的类型、常量与行为组织在一起。

## Key Concepts / 关键概念

- **EN:** Copy traits/policies separate abstract data movement from the concrete instruction selected underneath.
  **CN:** Copy trait/策略把抽象数据搬运与底层实际选用的指令解耦。
- **EN:** Inline assembly keeps the mapping between C++ fragments and hardware registers explicit.
  **CN:** 内联汇编使 C++ 片段与硬件寄存器之间的映射保持显式。
- **EN:** The Xe/SYCL path bridges CuTe abstractions to Intel GPU builtins or SPIR-V operations.
  **CN:** Xe/SYCL 路径把 CuTe 抽象桥接到 Intel GPU 内建函数或 SPIR-V 操作。

## Dependencies / 依赖关系

- **EN:** SYCL/SPIR-V feature macros select alternate code paths for Intel/Xe-style backends.
  **CN:** SYCL/SPIR-V 特性宏会为 Intel/Xe 风格后端选择替代代码路径。
