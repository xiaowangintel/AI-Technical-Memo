# util.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/util.hpp`
- **EN:** Provides low-level utility macros, traits, and helpers shared by architecture backends.
- **CN:** 提供各体系结构后端共享的底层工具宏、traits 与辅助函数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
/***************************************************************************************************
 * Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * Copyright (C) 2025 Intel Corporation, All rights reserved.
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

### Lines 34-35
```cpp
#include <cute/config.hpp>
#include <cute/numeric/integer_sequence.hpp>
```
- **EN:** Imports `cute/config.hpp` (core CuTe configuration macros and portability annotations); `cute/numeric/integer_sequence.hpp` (related definitions from `cute/numeric/integer_sequence.hpp`).
- **CN:** 引入 `cute/config.hpp`（CuTe 核心配置宏与可移植性标注）；`cute/numeric/integer_sequence.hpp`（来自 `cute/numeric/integer_sequence.hpp` 的相关定义）。

### Lines 37-41
```cpp
#if defined(__clang__) && defined(__CUDA__)
  //  __cvta_generic_to_shared was added in Clang 14: https://reviews.llvm.org/D111665
  #if __clang_major__ >= 14
    #define CUTE_CLANG_SUPPORTS_CVTA_GENERIC_TO_SHARED 1
  #endif
```
- **EN:** Uses preprocessor conditions to select architecture-specific or build-specific behavior.
- **CN:** 使用预处理条件选择特定体系结构或构建配置下的行为。

### Lines 43-48
```cpp
  // __nvvm_get_smem_pointer added in Clang 14: https://reviews.llvm.org/D111665
  // ... but will not work on Windows until Clang 15: https://reviews.llvm.org/D122897
  #if (!defined(_WIN32) && __clang_major__ >= 14) || __clang_major__ >= 15
    #define CUTE_CLANG_SUPPORTS_NVVM_GET_SMEM_POINTER 1
  #endif
#endif
```
- **EN:** Uses preprocessor conditions to select architecture-specific or build-specific behavior.
- **CN:** 使用预处理条件选择特定体系结构或构建配置下的行为。

### Lines 50-54
```cpp
#if defined(__NVCC__) || defined(__CUDACC_RTC__)
  // __cvta_generic_to_shared added in CUDA 11+
  #if __CUDACC_VER_MAJOR__ >= 11
    #define CUTE_NVCC_SUPPORTS_CVTA_GENERIC_TO_SHARED 1
  #endif
```
- **EN:** Uses preprocessor conditions to select architecture-specific or build-specific behavior.
- **CN:** 使用预处理条件选择特定体系结构或构建配置下的行为。

### Lines 56-60
```cpp
  // __nvvm_get_smem_pointer added in CUDA 10.2
  #if __CUDACC_VER_MAJOR__ == 10 && __CUDACC_VER_MINOR__ >= 2
    #define CUTE_NVCC_SUPPORTS_NVVM_GET_SMEM_POINTER 1
  #endif
#endif
```
- **EN:** Uses preprocessor conditions to select architecture-specific or build-specific behavior.
- **CN:** 使用预处理条件选择特定体系结构或构建配置下的行为。

### Lines 62-64
```cpp
#if CUTE_NVCC_SUPPORTS_CVTA_GENERIC_TO_SHARED || CUTE_CLANG_SUPPORTS_CVTA_GENERIC_TO_SHARED
  #define CUTE_CVTA_GENERIC_TO_SHARED_SUPPORTED 1
#endif
```
- **EN:** Uses preprocessor conditions to select architecture-specific or build-specific behavior.
- **CN:** 使用预处理条件选择特定体系结构或构建配置下的行为。

### Lines 66-68
```cpp
#if !defined(CUTE_CVTA_GENERIC_TO_SHARED_ACTIVATED) && CUTE_CVTA_GENERIC_TO_SHARED_SUPPORTED && defined(__CUDA_ARCH__)
  #define CUTE_CVTA_GENERIC_TO_SHARED_ACTIVATED 1
#endif
```
- **EN:** Uses preprocessor checks/macros (__CUDA_ARCH__) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（__CUDA_ARCH__）只启用当前目标有效的构建路径。

### Lines 70-72
```cpp
#if CUTE_NVCC_SUPPORTS_NVVM_GET_SMEM_POINTER || CUTE_CLANG_SUPPORTS_NVVM_GET_SMEM_POINTER
  #define CUTE_NVVM_GET_SMEM_POINTER_SUPPORTED 1
#endif
```
- **EN:** Uses preprocessor conditions to select architecture-specific or build-specific behavior.
- **CN:** 使用预处理条件选择特定体系结构或构建配置下的行为。

### Lines 74-76
```cpp
#if !defined(CUTE_NVVM_GET_SMEM_POINTER_ACTIVATED) && CUTE_NVVM_GET_SMEM_POINTER_SUPPORTED && defined(__CUDA_ARCH__)
  #define CUTE_NVVM_GET_SMEM_POINTER_ACTIVATED 1
#endif
```
- **EN:** Uses preprocessor checks/macros (__CUDA_ARCH__) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（__CUDA_ARCH__）只启用当前目标有效的构建路径。

### Lines 78-86
```cpp
// Clang 14+ provides a declaration of __nvvm_get_smem_pointer, so we only need
// to provide one for NVCC
#if CUTE_NVCC_SUPPORTS_NVVM_GET_SMEM_POINTER
  extern "C" {
  // This NVVM intrinsic is subject to change in future versions of CUDA.
  // Clients should not call it directly.
  CUTE_DEVICE uint32_t __nvvm_get_smem_pointer(void*);
  }
#endif
```
- **EN:** Defines or forwards `__nvvm_get_smem_pointer` as part of this header's executable interface.
- **CN:** 定义或转发 `__nvvm_get_smem_pointer`，作为该头文件可执行接口的一部分。

### Lines 88-89
```cpp
namespace cute
{
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 91-137
```cpp
/// CUTE helper to cast SMEM pointer to unsigned
CUTE_HOST_DEVICE
uint32_t
cast_smem_ptr_to_uint(void const* const ptr)
{
// We prefer to use the new CVTA intrinsics if they are available, otherwise we will fall back to
// the previous internal intrinsics if they are available.
#if CUTE_CVTA_GENERIC_TO_SHARED_ACTIVATED
  //
  // This NVVM intrinsic converts an address in shared memory to a plain
  // unsigned integer. This is necessary to pass to shared memory instructions
  // in inline PTX.
  //
  // In CUDA 11 and beyond, this replaces __nvvm_get_smem_pointer()  [only available in 10.2].
  //
  //__device__ size_t __cvta_generic_to_shared(void* ptr);

  /// CUTE helper to get SMEM pointer
  return static_cast<uint32_t>(__cvta_generic_to_shared(ptr));

#elif CUTE_NVVM_GET_SMEM_POINTER_ACTIVATED

  return __nvvm_get_smem_pointer(ptr);

#elif defined(CUTLASS_ENABLE_SYCL)

  return (intptr_t)(sycl::decorated_local_ptr<const void>::pointer)ptr;

#elif defined(__CUDA_ARCH__)

  uint32_t smem_ptr;

  asm(
  "{ .reg .u64 smem_ptr; cvta.to.shared.u64 smem_ptr, %1; cvt.u32.u64 %0, smem_ptr; }\n"
    : "=r"(smem_ptr) : "l"(ptr));

  return smem_ptr;

#else

  (void) ptr;
  printf("ERROR: cast_smem_ptr_to_uint not supported but used.\n");
  return 0;

#endif
}
```
- **EN:** Defines or forwards `cast_smem_ptr_to_uint` as part of this header's executable interface.
- **CN:** 定义或转发 `cast_smem_ptr_to_uint`，作为该头文件可执行接口的一部分。

### Lines 139-139
```cpp
namespace detail {
```
- **EN:** Enters or leaves namespace scope `detail` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `detail`，以便把相关符号组织在一起。

### Lines 141-152
```cpp
//
// Wrapper for MMAOp::fma
//

template <class MmaOp>
struct CallFMA {
  template <class... Args>
  CUTE_HOST_DEVICE constexpr void
  operator()(Args&&... args) const {
    return MmaOp::fma(static_cast<Args&&>(args)...);
  }
};
```
- **EN:** Defines `CallFMA` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `CallFMA`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 154-165
```cpp
//
// Wrapper for CopyOp::copy
//

template <class CopyOp>
struct CallCOPY {
  template <class... Args>
  CUTE_HOST_DEVICE constexpr void
  operator()(Args&&... args) const {
    return CopyOp::copy(static_cast<Args&&>(args)...);
  }
};
```
- **EN:** Defines `CallCOPY` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `CallCOPY`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 167-178
```cpp
//
// Wrapper for ReorderOp::reorder
//

template <class ReorderOp>
struct CallReorder {
  template <class... Args>
  CUTE_HOST_DEVICE constexpr void
  operator()(Args&&... args) const {
    return ReorderOp::reorder(static_cast<Args&&>(args)...);
  }
};
```
- **EN:** Defines `CallReorder` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `CallReorder`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 180-192
```cpp
//
// Utility for exploding pointers/arrays/tensors into functions
//

template <class Fn,
          class PtrA, int... I>
CUTE_HOST_DEVICE constexpr
void
explode(Fn fn,
        PtrA&& a, int_sequence<I...>)
{
  return fn(a[I]...);
}
```
- **EN:** Defines or forwards `explode` as part of this header's executable interface.
- **CN:** 定义或转发 `explode`，作为该头文件可执行接口的一部分。

### Lines 194-204
```cpp
template <class Fn,
          class PtrS, int... Is,
          class PtrD, int... Id>
CUTE_HOST_DEVICE constexpr
void
explode(Fn fn,
        PtrS&& s, int_sequence<Is...>,
        PtrD&& d, int_sequence<Id...>)
{
  return fn(s[Is]..., d[Id]...);
}
```
- **EN:** Defines or forwards `explode` as part of this header's executable interface.
- **CN:** 定义或转发 `explode`，作为该头文件可执行接口的一部分。

### Lines 206-218
```cpp
template <class Fn,
          class PtrA, int... Ia,
          class PtrB, int... Ib,
          class PtrC, int... Ic>
CUTE_HOST_DEVICE constexpr
void
explode(Fn fn,
        PtrA&& a, int_sequence<Ia...>,
        PtrB&& b, int_sequence<Ib...>,
        PtrC&& c, int_sequence<Ic...>)
{
  return fn(a[Ia]..., b[Ib]..., c[Ic]...);
}
```
- **EN:** Defines or forwards `explode` as part of this header's executable interface.
- **CN:** 定义或转发 `explode`，作为该头文件可执行接口的一部分。

### Lines 220-234
```cpp
template <class Fn,
          class PtrD, int... Id,
          class PtrA, int... Ia,
          class PtrB, int... Ib,
          class PtrC, int... Ic>
CUTE_HOST_DEVICE constexpr
void
explode(Fn fn,
        PtrD&& d, int_sequence<Id...>,
        PtrA&& a, int_sequence<Ia...>,
        PtrB&& b, int_sequence<Ib...>,
        PtrC&& c, int_sequence<Ic...>)
{
  return fn(d[Id]..., a[Ia]..., b[Ib]..., c[Ic]...);
}
```
- **EN:** Defines or forwards `explode` as part of this header's executable interface.
- **CN:** 定义或转发 `explode`，作为该头文件可执行接口的一部分。

### Lines 236-252
```cpp
template <class Fn,
          class PtrD, int... Id,
          class PtrA, int... Ia,
          class PtrB, int... Ib,
          class PtrC, int... Ic,
          class PtrE, int... Ie>
CUTE_HOST_DEVICE constexpr
void
explode(Fn fn,
        PtrD&& d, int_sequence<Id...>,
        PtrA&& a, int_sequence<Ia...>,
        PtrB&& b, int_sequence<Ib...>,
        PtrC&& c, int_sequence<Ic...>,
        PtrE&& e, int_sequence<Ie...>)
{
  return fn(d[Id]..., a[Ia]..., b[Ib]..., c[Ic]..., e[Ie]...);
}
```
- **EN:** Defines or forwards `explode` as part of this header's executable interface.
- **CN:** 定义或转发 `explode`，作为该头文件可执行接口的一部分。

### Lines 254-272
```cpp
template <class Fn,
          class PtrD, int... Id,
          class PtrA, int... Ia,
          class PtrB, int... Ib,
          class PtrC, int... Ic,
          class PtrE, int... Ie,
          class PtrF, int... If>
CUTE_HOST_DEVICE constexpr
void
explode(Fn fn,
        PtrD&& d, int_sequence<Id...>,
        PtrA&& a, int_sequence<Ia...>,
        PtrB&& b, int_sequence<Ib...>,
        PtrC&& c, int_sequence<Ic...>,
        PtrE&& e, int_sequence<Ie...>,
        PtrF&& f, int_sequence<If...>)
{
  return fn(d[Id]..., a[Ia]..., b[Ib]..., c[Ic]..., e[Ie]..., f[If]...);
}
```
- **EN:** Defines or forwards `explode` as part of this header's executable interface.
- **CN:** 定义或转发 `explode`，作为该头文件可执行接口的一部分。

### Lines 274-294
```cpp
template <class Fn,
          class PtrD, int... Id,
          class PtrA, int... Ia,
          class PtrB, int... Ib,
          class PtrC, int... Ic,
          class PtrE, int... Ie,
          class PtrF, int... If,
          class PtrG, int... Ig>
CUTE_HOST_DEVICE constexpr
void
explode(Fn fn,
        PtrD&& d, int_sequence<Id...>,
        PtrA&& a, int_sequence<Ia...>,
        PtrB&& b, int_sequence<Ib...>,
        PtrC&& c, int_sequence<Ic...>,
        PtrE&& e, int_sequence<Ie...>,
        PtrF&& f, int_sequence<If...>,
        PtrG&& g, int_sequence<Ig...>)
{
  return fn(d[Id]..., a[Ia]..., b[Ib]..., c[Ic]..., e[Ie]..., f[If]..., g[Ig]...);
}
```
- **EN:** Defines or forwards `explode` as part of this header's executable interface.
- **CN:** 定义或转发 `explode`，作为该头文件可执行接口的一部分。

### Lines 296-311
```cpp
#if defined(CUTLASS_ENABLE_SYCL)
template <class MMA_Op,
          class PtrD, int... Id,
          class PtrA, int... Ia,
          class PtrB, int... Ib,
          class PtrC, int... Ic>
CUTE_HOST_DEVICE constexpr
void
explode_mma(PtrD&& d, int_sequence<Id...>,
        PtrA&& a, int_sequence<Ia...>,
        PtrB&& b, int_sequence<Ib...>,
        PtrC&& c, int_sequence<Ic...>)
{
  return MMA_Op::fma(d[Id]..., a[Ia]..., b[Ib]..., c[Ic]...);
}
#endif
```
- **EN:** Defines or forwards `explode_mma` as part of this header's executable interface.
- **CN:** 定义或转发 `explode_mma`，作为该头文件可执行接口的一部分。

### Lines 313-325
```cpp
//
// Utility for exploding tuples into functions
//

template <class Fn,
          class TupleA, int... I>
CUTE_HOST_DEVICE constexpr
void
explode_tuple(Fn fn,
              TupleA&& a, int_sequence<I...>)
{
  return fn(get<I>(a)...);
}
```
- **EN:** Defines or forwards `explode_tuple` as part of this header's executable interface.
- **CN:** 定义或转发 `explode_tuple`，作为该头文件可执行接口的一部分。

### Lines 327-337
```cpp
template <class Fn,
          class TupleA, int... Ia,
          class TupleB, int... Ib>
CUTE_HOST_DEVICE constexpr
void
explode_tuple(Fn fn,
              TupleA&& a, int_sequence<Ia...>,
              TupleB&& b, int_sequence<Ib...>)
{
  return fn(get<Ia>(a)..., get<Ib>(b)...);
}
```
- **EN:** Defines or forwards `explode_tuple` as part of this header's executable interface.
- **CN:** 定义或转发 `explode_tuple`，作为该头文件可执行接口的一部分。

### Lines 339-351
```cpp
template <class Fn,
          class TupleA, int... Ia,
          class TupleB, int... Ib,
          class TupleC, int... Ic>
CUTE_HOST_DEVICE constexpr
void
explode_tuple(Fn fn,
              TupleA&& a, int_sequence<Ia...>,
              TupleB&& b, int_sequence<Ib...>,
              TupleC&& c, int_sequence<Ic...>)
{
  return fn(get<Ia>(a)..., get<Ib>(b)..., get<Ic>(c)...);
}
```
- **EN:** Defines or forwards `explode_tuple` as part of this header's executable interface.
- **CN:** 定义或转发 `explode_tuple`，作为该头文件可执行接口的一部分。

### Lines 353-353
```cpp
} // end namespace detail
```
- **EN:** Enters or leaves namespace scope `detail` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `detail`，以便把相关符号组织在一起。

### Lines 355-355
```cpp
} // end namespace cute
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

## Key Concepts / 关键概念

- **EN:** MMA/GMMA/UMMA wrappers expose tensor-core style matrix instructions as typed C++ interfaces.
  **CN:** MMA/GMMA/UMMA 封装把张量核心式矩阵指令暴露为带类型的 C++ 接口。

## Dependencies / 依赖关系

- **EN:** `cute/config.hpp` supplies core CuTe configuration macros and portability annotations.
  **CN:** `cute/config.hpp` 提供了CuTe 核心配置宏与可移植性标注。
- **EN:** `cute/numeric/integer_sequence.hpp` supplies related definitions from `cute/numeric/integer_sequence.hpp`.
  **CN:** `cute/numeric/integer_sequence.hpp` 提供了来自 `cute/numeric/integer_sequence.hpp` 的相关定义。
- **EN:** CUDA architecture macros gate device-only fast paths and inline instructions.
  **CN:** CUDA 架构宏用于控制仅设备端可用的快速路径与内联指令。
- **EN:** SYCL/SPIR-V feature macros select alternate code paths for Intel/Xe-style backends.
  **CN:** SYCL/SPIR-V 特性宏会为 Intel/Xe 风格后端选择替代代码路径。
