# copy_xe_2d.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cute/arch/copy_xe_2d.hpp`
- **EN:** Defines low-level copy instructions and wrappers for Intel Xe or generic backends.
- **CN:** 为 Intel Xe 或通用后端定义底层拷贝指令与封装。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
/***************************************************************************************************
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

### Lines 34-34
```cpp
#include "cute/numeric/int.hpp"
```
- **EN:** Imports `cute/numeric/int.hpp` (related definitions from `cute/numeric/int.hpp`).
- **CN:** 引入 `cute/numeric/int.hpp`（来自 `cute/numeric/int.hpp` 的相关定义）。

### Lines 36-38
```cpp
#if defined(__SYCL_DEVICE_ONLY__) && defined(SYCL_INTEL_TARGET)
#define CUTE_ARCH_COPY_XE_ENABLED
#endif
```
- **EN:** Uses preprocessor checks/macros (__SYCL_DEVICE_ONLY__, CUTE_ARCH_COPY_XE_ENABLED) to enable only the build paths valid for this target.
- **CN:** 使用预处理条件/宏（__SYCL_DEVICE_ONLY__, CUTE_ARCH_COPY_XE_ENABLED）只启用当前目标有效的构建路径。

### Lines 40-40
```cpp
namespace cute {
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

### Lines 42-68
```cpp
// Xe 2D copy atoms.
//       Bits: bits per element in underlying memory operation.
//     Height: number of elements in the gmem-strided matrix dimension
//      Width: number of elements in the gmem-contiguous matrix dimension
// BlockWidth: blocking factor (in registers) for the width dimension.
//
// For a row-major-in-memory matrix:
//    height = #rows, width = #columns.
//
// For a column-major-in-memory matrix:
//    height = #columns, width = #rows.

/* Base class for 2D copy ops, to support common queries */
template <int Bits, int Height, int Width, int Count = 1, bool Transpose = false>
struct XE_Copy_Op_2D_Base
{
  static_assert(Height <= 32, "Height exceeds hardware limits");
  static_assert(Bits * Width <= 8 * 64, "Total width exceeds hardware limits");
  static_assert(Bits * Count <= 64 && Count <= 4 && Count != 3, "Unsupported block count");
  static_assert(Bits == 8 || Bits == 16 || Bits == 32 || Bits == 64, "Unsupported data size");

  static constexpr int CopyBits = Bits;
  static constexpr int AtomWidth = Width;
  static constexpr int AtomHeight = Height;
  static constexpr int BlockCount = Count;
  static constexpr bool Transposing = Transpose;
};
```
- **EN:** Defines `XE_Copy_Op_2D_Base`, a tag or wrapper that describes how data movement should be performed on this backend.
- **CN:** 定义 `XE_Copy_Op_2D_Base`，它是一个标签或封装，用于描述该后端上的数据搬运方式。

### Lines 70-71
```cpp
template <int Bits, int Height, int Width, int BlockWidth = 0 /* unused */>
struct XE_PREFETCH_2D;
```
- **EN:** Defines `XE_PREFETCH_2D` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_PREFETCH_2D`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 74-93
```cpp
template <int Bits, int Height, int Width, int BlockWidth = Width>
struct XE_LOAD_2D : XE_Copy_Op_2D_Base<Bits, Height, Width, Width/BlockWidth>
{
  template <typename T>
  CUTE_HOST_DEVICE static void copy(const int *payload, T *dst) {
#ifdef CUTE_ARCH_COPY_XE_ENABLED
    using namespace intel;
    auto &dv = *reinterpret_cast<storage_vector_t<T, Width * Height * Bits / sg_size>*>(dst);
    asm (
      "lsc_load_block2d.ugm (M1, 1)  %0:d%2.%3x%4x%5nn flat[%1+(0,0)]"
        : "=rw"(dv)
        : "rw.u"(payload), "P"(Bits), "P"(Width/BlockWidth), "P"(BlockWidth), "P"(Height)
    );
#else
    CUTE_INVALID_CONTROL_PATH("Cannot use Xe block 2D copy atom on non-Xe hardware");
#endif
  }

  using PREFETCH = XE_PREFETCH_2D<Bits, Height, Width>;
};
```
- **EN:** Defines `XE_LOAD_2D` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_LOAD_2D`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 95-116
```cpp
template <int Bits, int Height, int Width, int BlockWidth = Width>
struct XE_LOAD_2D_VNNI : XE_Copy_Op_2D_Base<Bits, Height, Width, Width/BlockWidth>
{
  static_assert(Bits == 8 || Bits == 16, "Unsupported data size");

  template <typename T>
  CUTE_HOST_DEVICE static void copy(const int *payload, T *dst) {
#ifdef CUTE_ARCH_COPY_XE_ENABLED
    using namespace intel;
    auto &dv = *reinterpret_cast<storage_vector_t<T, Width * Height * Bits / sg_size>*>(dst);
    asm (
      "lsc_load_block2d.ugm (M1, 1)  %0:d%2.%3x%4x%5nt flat[%1+(0,0)]"
        : "=rw"(dv)
        : "rw.u"(payload), "P"(Bits), "P"(Width/BlockWidth), "P"(BlockWidth), "P"(Height)
    );
#else
    CUTE_INVALID_CONTROL_PATH("Cannot use Xe block 2D copy atom on non-Xe hardware");
#endif
  }

  using PREFETCH = XE_PREFETCH_2D<Bits, Height, Width>;
};
```
- **EN:** Defines `XE_LOAD_2D_VNNI` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_LOAD_2D_VNNI`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 118-141
```cpp
template <int Bits, int Height, int Width>
struct XE_LOAD_2D_TRANSPOSE : XE_Copy_Op_2D_Base<Bits, Height, Width, 1, true>
{
  static_assert(Bits == 32 || Bits == 64, "Unsupported data size");
  static_assert(Width <= 8, "Width exceeds hardware limits");
  static_assert(Bits != 64 || (Height == 8 && Width < 4), "Unsupported D64 transpose block size");

  template <typename T>
  CUTE_HOST_DEVICE static void copy(const int *payload, T *dst) {
#ifdef CUTE_ARCH_COPY_XE_ENABLED
    using namespace intel;
    auto &dv = *reinterpret_cast<storage_vector_t<T, Width * Height * Bits / sg_size>*>(dst);
    asm (
      "lsc_load_block2d.ugm (M1, 1)  %0:d%2.%3x%4tn flat[%1+(0,0)]"
        : "=rw"(dv)
        : "rw.u"(payload), "P"(Bits), "P"(Width), "P"(Height)
    );
#else
    CUTE_INVALID_CONTROL_PATH("Cannot use Xe block 2D copy atom on non-Xe hardware");
#endif
  }

  using PREFETCH = XE_PREFETCH_2D<Bits, Height, Width>;
};
```
- **EN:** Defines `XE_LOAD_2D_TRANSPOSE` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_LOAD_2D_TRANSPOSE`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 143-158
```cpp
template <int Bits, int Height, int Width, int>
struct XE_PREFETCH_2D : XE_Copy_Op_2D_Base<Bits, Height, Width>
{
  CUTE_HOST_DEVICE static void copy(const int *payload) {
#ifdef CUTE_ARCH_COPY_XE_ENABLED
    asm (
      "lsc_load_block2d.ugm.ca.ca (M1, 1)  %%null:d%1.%2x%3nn flat[%0+(0,0)]"
        :: "rw.u"(payload), "P"(Bits), "P"(Width), "P"(Height)
    );
#else
    CUTE_INVALID_CONTROL_PATH("Cannot use Xe block 2D copy atom on non-Xe hardware");
#endif
  }

  using PREFETCH = XE_PREFETCH_2D;
};
```
- **EN:** Defines `XE_PREFETCH_2D` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_PREFETCH_2D`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 160-178
```cpp
template <int Bits, int Height, int Width>
struct XE_STORE_2D : XE_Copy_Op_2D_Base<Bits, Height, Width>
{
  static_assert(Height <= 8, "Height exceeds hardware limits");

  template <typename T>
  CUTE_HOST_DEVICE static void copy(const int *payload, const T *src) {
#ifdef CUTE_ARCH_COPY_XE_ENABLED
  using namespace intel;
    auto &sv = *reinterpret_cast<const storage_vector_t<T, Width * Height * Bits / sg_size>*>(src); \
    asm (
      "lsc_store_block2d.ugm (M1, 1) flat[%1+(0,0)] %0:d%2.%3x%4nn"
        :: "rw"(sv), "rw.u"(payload), "P"(Bits), "P"(Width), "P"(Height)
    );
#else
    CUTE_INVALID_CONTROL_PATH("Cannot use Xe block 2D copy atom on non-Xe hardware");
#endif
  }
};
```
- **EN:** Defines `XE_STORE_2D` and groups the types, constants, and behavior needed by this part of the header.
- **CN:** 定义 `XE_STORE_2D`，把该头文件这一部分所需的类型、常量与行为组织在一起。

### Lines 180-180
```cpp
} // end namespace cute
```
- **EN:** Enters or leaves namespace scope `cute` so related symbols stay grouped.
- **CN:** 进入或离开命名空间作用域 `cute`，以便把相关符号组织在一起。

## Key Concepts / 关键概念

- **EN:** Copy traits/policies separate abstract data movement from the concrete instruction selected underneath.
  **CN:** Copy trait/策略把抽象数据搬运与底层实际选用的指令解耦。
- **EN:** Compile-time assertions encode hardware and type constraints directly in the API surface.
  **CN:** 编译期断言把硬件与类型约束直接编码到 API 表面。
- **EN:** Prefetch paths try to reduce latency by staging data or metadata early.
  **CN:** 预取路径尝试通过提前准备数据或元数据来降低延迟。
- **EN:** The Xe/SYCL path bridges CuTe abstractions to Intel GPU builtins or SPIR-V operations.
  **CN:** Xe/SYCL 路径把 CuTe 抽象桥接到 Intel GPU 内建函数或 SPIR-V 操作。

## Dependencies / 依赖关系

- **EN:** `cute/numeric/int.hpp` supplies related definitions from `cute/numeric/int.hpp`.
  **CN:** `cute/numeric/int.hpp` 提供了来自 `cute/numeric/int.hpp` 的相关定义。
- **EN:** SYCL/SPIR-V feature macros select alternate code paths for Intel/Xe-style backends.
  **CN:** SYCL/SPIR-V 特性宏会为 Intel/Xe 风格后端选择替代代码路径。
