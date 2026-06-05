# copy_sm75.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/copy_sm75.hpp`
- **EN:** Defines low-level copy instructions and wrappers for NVIDIA SM75 or generic backends.
- **CN:** 为 NVIDIA SM75 或通用后端定义底层拷贝指令与封装。

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

### Lines 33-33
```cpp
#include <cute/config.hpp>
```
- **EN:** Imports `cute/config.hpp` (core CuTe configuration macros and portability annotations).
- **CN:** 引入 `cute/config.hpp`（CuTe 核心配置宏与可移植性标注）。

### Lines 35-35
```cpp
#include <cute/arch/copy.hpp>
```
- **EN:** Imports `cute/arch/copy.hpp` (generic low-level copy primitives and policies).
- **CN:** 引入 `cute/arch/copy.hpp`（通用底层拷贝原语与策略）。

### Lines 37-45
```cpp
// Config
#if defined(__clang__) && defined(__CUDA__)
  // ldmatrix PTX instructions added in Clang 14: https://reviews.llvm.org/D107046
  // ... but will not work until Clang 15:
  //   * https://reviews.llvm.org/D121666
  //   * https://reviews.llvm.org/D126846
  #define CUTE_ARCH_CLANG_SUPPORTS_LDSM_SM75 (__clang_major__ >= 15)
  #define CUTE_ARCH_CLANG_SUPPORTS_MOVM_SM75 (__clang_major__ >= 15)
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTE_ARCH_CLANG_SUPPORTS_LDSM_SM75, CUTE_ARCH_CLANG_SUPPORTS_MOVM_SM75) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTE_ARCH_CLANG_SUPPORTS_LDSM_SM75, CUTE_ARCH_CLANG_SUPPORTS_MOVM_SM75）只启用当前目标有效的构建路径。

### Lines 47-51
```cpp
#if defined(__NVCC__) || defined(__CUDACC_RTC__)
  // ldmatrix PTX instruction added in CUDA 10.2+
  #define CUTE_ARCH_NVCC_SUPPORTS_LDSM_SM75 ((__CUDACC_VER_MAJOR__  == 10 && __CUDACC_VER_MINOR__ >= 2) || __CUDACC_VER_MAJOR__ >= 11)
  #define CUTE_ARCH_NVCC_SUPPORTS_MOVM_SM75 ((__CUDACC_VER_MAJOR__  == 10 && __CUDACC_VER_MINOR__ >= 2) || __CUDACC_VER_MAJOR__ >= 11)
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTE_ARCH_NVCC_SUPPORTS_LDSM_SM75, CUTE_ARCH_NVCC_SUPPORTS_MOVM_SM75) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTE_ARCH_NVCC_SUPPORTS_LDSM_SM75, CUTE_ARCH_NVCC_SUPPORTS_MOVM_SM75）只启用当前目标有效的构建路径。

### Lines 53-56
```cpp
#if defined(SYCL_NVIDIA_TARGET)
  #define CUTE_ARCH_NVCC_SUPPORTS_LDSM_SM75 1
  #define CUTE_ARCH_NVCC_SUPPORTS_MOVM_SM75 1
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTE_ARCH_NVCC_SUPPORTS_LDSM_SM75, CUTE_ARCH_NVCC_SUPPORTS_MOVM_SM75) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTE_ARCH_NVCC_SUPPORTS_LDSM_SM75, CUTE_ARCH_NVCC_SUPPORTS_MOVM_SM75）只启用当前目标有效的构建路径。

### Lines 58-60
```cpp
#if ! defined(CUTE_ARCH_LDSM_SM75_SUPPORTED)
  #define CUTE_ARCH_LDSM_SM75_SUPPORTED (CUTE_ARCH_NVCC_SUPPORTS_LDSM_SM75 || CUTE_ARCH_CLANG_SUPPORTS_LDSM_SM75)
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTE_ARCH_LDSM_SM75_SUPPORTED, CUTE_ARCH_NVCC_SUPPORTS_LDSM_SM75, CUTE_ARCH_CLANG_SUPPORTS_LDSM_SM75) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTE_ARCH_LDSM_SM75_SUPPORTED, CUTE_ARCH_NVCC_SUPPORTS_LDSM_SM75, CUTE_ARCH_CLANG_SUPPORTS_LDSM_SM75）只启用当前目标有效的构建路径。

### Lines 62-64
```cpp
#if ! defined(CUTE_ARCH_LDSM_SM75_ENABLED)
  #define CUTE_ARCH_LDSM_SM75_ENABLED (CUTE_ARCH_LDSM_SM75_SUPPORTED)
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTE_ARCH_LDSM_SM75_ENABLED, CUTE_ARCH_LDSM_SM75_SUPPORTED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTE_ARCH_LDSM_SM75_ENABLED, CUTE_ARCH_LDSM_SM75_SUPPORTED）只启用当前目标有效的构建路径。

### Lines 66-69
```cpp
#if (CUTE_ARCH_LDSM_SM75_ENABLED) && \
  ((defined(__CUDA_ARCH__) && __CUDA_ARCH__ >= 750) || (defined(__SYCL_CUDA_ARCH__) && __SYCL_CUDA_ARCH__ >= 750))
  #define CUTE_ARCH_LDSM_SM75_ACTIVATED 1
#endif
```
- **EN:** Implements supporting control flow for the surrounding type or function.
- **CN:** 实现周边类型或函数所需的辅助控制流。

### Lines 71-73
```cpp
#if ! defined(CUTE_ARCH_MOVM_SM75_SUPPORTED)
  #define CUTE_ARCH_MOVM_SM75_SUPPORTED (CUTE_ARCH_NVCC_SUPPORTS_MOVM_SM75 || CUTE_ARCH_CLANG_SUPPORTS_MOVM_SM75)
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTE_ARCH_MOVM_SM75_SUPPORTED, CUTE_ARCH_NVCC_SUPPORTS_MOVM_SM75, CUTE_ARCH_CLANG_SUPPORTS_MOVM_SM75) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTE_ARCH_MOVM_SM75_SUPPORTED, CUTE_ARCH_NVCC_SUPPORTS_MOVM_SM75, CUTE_ARCH_CLANG_SUPPORTS_MOVM_SM75）只启用当前目标有效的构建路径。

### Lines 75-77
```cpp
#if ! defined(CUTE_ARCH_MOVM_SM75_ENABLED)
  #define CUTE_ARCH_MOVM_SM75_ENABLED (CUTE_ARCH_MOVM_SM75_SUPPORTED)
#endif
```
- **EN:** Uses preprocessor checks/macros (CUTE_ARCH_MOVM_SM75_ENABLED, CUTE_ARCH_MOVM_SM75_SUPPORTED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（CUTE_ARCH_MOVM_SM75_ENABLED, CUTE_ARCH_MOVM_SM75_SUPPORTED）只启用当前目标有效的构建路径。

### Lines 79-82
```cpp
#if (CUTE_ARCH_MOVM_SM75_ENABLED) && \
  ((defined(__CUDA_ARCH__) && __CUDA_ARCH__ >= 750) || (defined(__SYCL_CUDA_ARCH__) && __SYCL_CUDA_ARCH__ >= 750))
  #define CUTE_ARCH_MOVM_SM75_ACTIVATED 1
#endif
```
- **EN:** Implements supporting control flow for the surrounding type or function.
- **CN:** 实现周边类型或函数所需的辅助控制流。

### Lines 85-86
```cpp
namespace cute
{
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 88-106
```cpp
struct SM75_U32x1_LDSM_N
{
  using SRegisters = uint128_t[1];
  using DRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  copy(uint128_t const& smem_src,
       uint32_t& dst)
  {
#if defined(CUTE_ARCH_LDSM_SM75_ACTIVATED)
    uint32_t smem_int_ptr = cast_smem_ptr_to_uint(&smem_src);
    asm volatile ("ldmatrix.sync.aligned.x1.m8n8.shared.b16 {%0}, [%1];\n"
        : "=r"(dst)
        :  "r"(smem_int_ptr));
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use ldmatrix without CUTE_ARCH_LDSM_SM75_ACTIVATED.");
#endif
  }
};
```
- **EN:** Defines `SM75_U32x1_LDSM_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM75_U32x1_LDSM_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 108-126
```cpp
struct SM75_U32x2_LDSM_N
{
  using SRegisters = uint128_t[1];
  using DRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  copy(uint128_t const& smem_src,
       uint32_t& dst0, uint32_t& dst1)
  {
#if defined(CUTE_ARCH_LDSM_SM75_ACTIVATED)
    uint32_t smem_int_ptr = cast_smem_ptr_to_uint(&smem_src);
    asm volatile ("ldmatrix.sync.aligned.x2.m8n8.shared.b16 {%0, %1}, [%2];\n"
        : "=r"(dst0), "=r"(dst1)
        :  "r"(smem_int_ptr));
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use ldmatrix without CUTE_ARCH_LDSM_SM75_ACTIVATED.");
#endif
  }
};
```
- **EN:** Defines `SM75_U32x2_LDSM_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM75_U32x2_LDSM_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 128-146
```cpp
struct SM75_U32x4_LDSM_N
{
  using SRegisters = uint128_t[1];
  using DRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  copy(uint128_t const& smem_src,
       uint32_t& dst0, uint32_t& dst1, uint32_t& dst2, uint32_t& dst3)
  {
#if defined(CUTE_ARCH_LDSM_SM75_ACTIVATED)
    uint32_t smem_int_ptr = cast_smem_ptr_to_uint(&smem_src);
    asm volatile ("ldmatrix.sync.aligned.x4.m8n8.shared.b16 {%0, %1, %2, %3}, [%4];\n"
        : "=r"(dst0), "=r"(dst1), "=r"(dst2), "=r"(dst3)
        :  "r"(smem_int_ptr));
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use ldmatrix without CUTE_ARCH_LDSM_SM75_ACTIVATED.");
#endif
  }
};
```
- **EN:** Defines `SM75_U32x4_LDSM_N` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM75_U32x4_LDSM_N`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 148-166
```cpp
struct SM75_U16x2_LDSM_T
{
  using SRegisters = uint128_t[1];
  using DRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void
  copy(uint128_t const& smem_src,
       uint32_t& dst)
  {
#if defined(CUTE_ARCH_LDSM_SM75_ACTIVATED)
    uint32_t smem_int_ptr = cast_smem_ptr_to_uint(&smem_src);
    asm volatile ("ldmatrix.sync.aligned.x1.trans.m8n8.shared.b16 {%0}, [%1];\n"
        : "=r"(dst)
        :  "r"(smem_int_ptr));
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use ldmatrix without CUTE_ARCH_LDSM_SM75_ACTIVATED.");
#endif
  }
};
```
- **EN:** Defines `SM75_U16x2_LDSM_T` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM75_U16x2_LDSM_T`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 168-186
```cpp
struct SM75_U16x4_LDSM_T
{
  using SRegisters = uint128_t[1];
  using DRegisters = uint32_t[2];

  CUTE_HOST_DEVICE static void
  copy(uint128_t const& smem_src,
       uint32_t& dst0, uint32_t& dst1)
  {
#if defined(CUTE_ARCH_LDSM_SM75_ACTIVATED)
    uint32_t smem_int_ptr = cast_smem_ptr_to_uint(&smem_src);
    asm volatile ("ldmatrix.sync.aligned.x2.trans.m8n8.shared.b16 {%0, %1}, [%2];\n"
        : "=r"(dst0), "=r"(dst1)
        :  "r"(smem_int_ptr));
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use ldmatrix without CUTE_ARCH_LDSM_SM75_ACTIVATED.");
#endif
  }
};
```
- **EN:** Defines `SM75_U16x4_LDSM_T` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM75_U16x4_LDSM_T`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 188-206
```cpp
struct SM75_U16x8_LDSM_T
{
  using SRegisters = uint128_t[1];
  using DRegisters = uint32_t[4];

  CUTE_HOST_DEVICE static void
  copy(uint128_t const& smem_src,
       uint32_t& dst0, uint32_t& dst1, uint32_t& dst2, uint32_t& dst3)
  {
#if defined(CUTE_ARCH_LDSM_SM75_ACTIVATED)
    uint32_t smem_int_ptr = cast_smem_ptr_to_uint(&smem_src);
    asm volatile ("ldmatrix.sync.aligned.x4.trans.m8n8.shared.b16 {%0, %1, %2, %3}, [%4];\n"
        : "=r"(dst0), "=r"(dst1), "=r"(dst2), "=r"(dst3)
        :  "r"(smem_int_ptr));
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use ldmatrix without CUTE_ARCH_LDSM_SM75_ACTIVATED.");
#endif
  }
};
```
- **EN:** Defines `SM75_U16x8_LDSM_T` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM75_U16x8_LDSM_T`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 208-228
```cpp
struct SM75_U32x1_MOVM_T 
{
  using SRegisters = uint32_t[1];
  using DRegisters = uint32_t[1];

  CUTE_HOST_DEVICE static void 
  copy(uint32_t src, 
       uint32_t &dst) 
  {
#if CUTE_ARCH_MOVM_SM75_ACTIVATED
    asm volatile("movmatrix.sync.aligned.m8n8.trans.b16 %0, %1;\n"
        : "=r"(dst)
        :  "r"(src));
#else
    CUTE_INVALID_CONTROL_PATH("Trying to use movmatrix without CUTE_ARCH_MOVM_SM75_ACTIVATED.");
#endif
  }
};
//
// Legacy LDSM interfaces that aren't very useful
//
```
- **EN:** Defines `SM75_U32x1_MOVM_T` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `SM75_U32x1_MOVM_T`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 230-251
```cpp
template <class T>
CUTE_HOST_DEVICE
void
copy_ldsm(uint128_t const* const smem_ptr,
          T* rmem_ptr)
{
  uint32_t* reg_ptr = reinterpret_cast<uint32_t*>(rmem_ptr);

  // if constexpr
  if (sizeof(T) == 4) {
    SM75_U32x1_LDSM_N::copy(smem_ptr[0], reg_ptr[0]);
  }
  else if (sizeof(T) == 8) {
    SM75_U32x2_LDSM_N::copy(smem_ptr[0], reg_ptr[0], reg_ptr[1]);
  }
  else if (sizeof(T) == 16) {
    SM75_U32x4_LDSM_N::copy(smem_ptr[0], reg_ptr[0], reg_ptr[1], reg_ptr[2], reg_ptr[3]);
  }
  else {
    static_assert(sizeof(T) == 4 || sizeof(T) == 8 || sizeof(T) == 16, "sizeof(T) is not supported");
  }
}
```
- **EN:** Defines or forwards `copy_ldsm` as part of this header's executable interface.
- **CN:** 定义或转发 `copy_ldsm`，作为该头文件可执行接口的一部分。

### Lines 253-274
```cpp
template <class T>
CUTE_HOST_DEVICE
void
copy_ldsm_trans(uint128_t const* const smem_ptr,
                T* rmem_ptr)
{
  uint32_t* reg_ptr = reinterpret_cast<uint32_t*>(rmem_ptr);

  // if constexpr
  if (sizeof(T) == 4) {
    SM75_U16x2_LDSM_T::copy(smem_ptr[0], reg_ptr[0]);
  }
  else if (sizeof(T) == 8) {
    SM75_U16x4_LDSM_T::copy(smem_ptr[0], reg_ptr[0], reg_ptr[1]);
  }
  else if (sizeof(T) == 16) {
    SM75_U16x8_LDSM_T::copy(smem_ptr[0], reg_ptr[0], reg_ptr[1], reg_ptr[2], reg_ptr[3]);
  }
  else {
    static_assert(sizeof(T) == 4 || sizeof(T) == 8 || sizeof(T) == 16, "sizeof(T) is not supported");
  }
}
```
- **EN:** Defines or forwards `copy_ldsm_trans` as part of this header's executable interface.
- **CN:** 定义或转发 `copy_ldsm_trans`，作为该头文件可执行接口的一部分。

### Lines 276-276
```cpp
} // end namespace cute
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

## Key Concepts / 关键概念

- **EN:** Copy traits/policies separate abstract data movement from the concrete instruction selected underneath.
  **CN:** Copy trait/策略把抽象数据搬运与底层实际选用的指令解耦。
- **EN:** Inline assembly keeps the mapping between C++ fragments and hardware registers explicit.
  **CN:** 内联汇编使 C++ 片段与硬件寄存器之间的映射保持显式。
- **EN:** Compile-time assertions encode hardware and type constraints directly in the API surface.
  **CN:** 编译期断言把硬件与类型约束直接编码到 API 表面。
- **EN:** The Xe/SYCL path bridges CuTe abstractions to Intel GPU builtins or SPIR-V operations.
  **CN:** Xe/SYCL 路径把 CuTe 抽象桥接到 Intel GPU 内建函数或 SPIR-V 操作。

## Dependencies / 依赖关系

- **EN:** `cute/config.hpp` supplies core CuTe configuration macros and portability annotations.
  **CN:** `cute/config.hpp` 提供了CuTe 核心配置宏与可移植性标注。
- **EN:** `cute/arch/copy.hpp` supplies generic low-level copy primitives and policies.
  **CN:** `cute/arch/copy.hpp` 提供了通用底层拷贝原语与策略。
- **EN:** CUDA architecture macros gate device-only fast paths and inline instructions.
  **CN:** CUDA 架构宏用于控制仅设备端可用的快速路径与内联指令。
