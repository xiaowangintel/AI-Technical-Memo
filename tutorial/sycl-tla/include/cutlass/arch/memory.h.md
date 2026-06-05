# memory.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/arch/memory.h`

- **EN:** Architecture-specific operators on memory

- **CN:** 该头文件主要实现 CUTLASS 的架构相关原语与指令封装。文件级摘要：Architecture-specific operators on memory

## Line-by-Line Analysis / 逐行分析

### Lines 1-30

```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
```

**EN:** This block records the file copyright, SPDX identifier, and redistribution disclaimer.

**CN:** 该代码块记录了文件的版权信息、SPDX 标识以及再分发免责声明。

### Lines 31-33

```cpp
/*! \file
    \brief Architecture-specific operators on memory
*/
```

**EN:** This file-level comment explains the purpose of the header and introduces the abstractions defined below.

**CN:** 这个文件级注释说明了头文件的用途，并引出了后续定义的抽象。

### Lines 35-35

```cpp
#pragma once
```

**EN:** This directive uses `#pragma once` to avoid repeated inclusion of the same header.

**CN:** 这里使用 `#pragma once` 来避免同一头文件被重复包含。

### Lines 37-40

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/arch/cache_operation.h"
#include "cutlass/platform/platform.h"
#include "cute/config.hpp"
```

**EN:** This block imports dependencies such as `cutlass/cutlass.h`, `cutlass/arch/cache_operation.h`, `cutlass/platform/platform.h`, `cute/config.hpp`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cutlass/cutlass.h`, `cutlass/arch/cache_operation.h`, `cutlass/platform/platform.h`, `cute/config.hpp` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 42-43

```cpp
namespace cutlass {
namespace arch {
```

**EN:** This block opens the namespace scope `cutlass::arch` for the declarations that follow.

**CN:** 该代码块打开了 `cutlass::arch` 命名空间作用域，以容纳后续声明。

### Lines 45-54

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
template <
    /// Fragment type to store loaded data
    typename AccessType,
    /// The bytes of loading
    int LoadBytes,
    /// Cache operation
    CacheOperation::Kind cache_op = CacheOperation::Always
    >
struct global_load;
```

**EN:** The preceding comment documents this block. This block begins the definition of `global_load`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `global_load` 这个 `struct`，其成员会在后续代码中展开。

### Lines 57-61

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
//
// Specializations
//
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 65-67

```cpp
#if (((__CUDACC_VER_MAJOR__ == 11) && (__CUDACC_VER_MINOR__ >= 4)) || \
     (__CUDACC_VER_MAJOR__ > 11)) &&                                  \
    defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 750)
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#if (((__CUDACC_VER_MAJOR__ == 11) && (__CUDACC_VER_MINOR__ >= 4)) || \`.

**CN:** 这个预处理代码块围绕 `#if (((__CUDACC_VER_MAJOR__ == 11) && (__CUDACC_VER_MINOR__ >= 4)) || \` 选择编译路径或功能开关。

### Lines 68-68

```cpp
  #define CUTLASS_ENABLE_L2_PREFETCH 1
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#define CUTLASS_ENABLE_L2_PREFETCH 1`.

**CN:** 这个预处理代码块围绕 `#define CUTLASS_ENABLE_L2_PREFETCH 1` 选择编译路径或功能开关。

### Lines 69-69

```cpp
#else
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#else`.

**CN:** 这个预处理代码块围绕 `#else` 选择编译路径或功能开关。

### Lines 70-70

```cpp
  #define CUTLASS_ENABLE_L2_PREFETCH 0
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#define CUTLASS_ENABLE_L2_PREFETCH 0`.

**CN:** 这个预处理代码块围绕 `#define CUTLASS_ENABLE_L2_PREFETCH 0` 选择编译路径或功能开关。

### Lines 71-71

```cpp
#endif
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#endif`.

**CN:** 这个预处理代码块围绕 `#endif` 选择编译路径或功能开关。

### Lines 75-81

```cpp
// The redundant mov PTX instruction is used to enforce the compiler to
// keep the initializing code before ld.global
template <typename AccessType>
struct global_load<AccessType,
                   32,
                   CacheOperation::Always
                  > {
```

**EN:** The preceding comment documents this block. This block begins the definition of `global_load`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `global_load` 这个 `struct`，其成员会在后续代码中展开。

### Lines 82-116

```cpp
  CUTLASS_DEVICE
  global_load(AccessType &D, void const *ptr, bool pred_guard) {
  uint4 *data = reinterpret_cast<uint4 *>(&D);

#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
    asm volatile(
        "{\n"
        "  .reg .pred p;\n"
        "  setp.ne.b32 p, %9, 0;\n"
        "  mov.b32 %0, %10;\n"
        "  mov.b32 %1, %11;\n"
        "  mov.b32 %2, %12;\n"
        "  mov.b32 %3, %13;\n"
        "  mov.b32 %4, %14;\n"
        "  mov.b32 %5, %15;\n"
        "  mov.b32 %6, %16;\n"
        "  mov.b32 %7, %17;\n"
#if CUTLASS_ENABLE_L2_PREFETCH
        "  @p ld.global.L2::128B.v4.u32 {%0, %1, %2, %3}, [%8];\n"
        "  @p ld.global.L2::128B.v4.u32 {%4, %5, %6, %7}, [%18];\n"
#else
        "  @p ld.global.v4.u32 {%0, %1, %2, %3}, [%8];\n"
        "  @p ld.global.v4.u32 {%4, %5, %6, %7}, [%18];\n"
#endif
        "}\n"
        : "=r"(data[0].x), "=r"(data[0].y), "=r"(data[0].z), "=r"(data[0].w),
          "=r"(data[1].x), "=r"(data[1].y), "=r"(data[1].z), "=r"(data[1].w)
        : "l"(ptr), "r"((int)pred_guard), "r"(data[0].x), "r"(data[0].y),
          "r"(data[0].z), "r"(data[0].w), "r"(data[1].x), "r"(data[1].y),
          "r"(data[1].z), "r"(data[1].w), "l"(((uint8_t *)ptr) + 16));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
  }
```

**EN:** The function `data` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** `data` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 119-123

```cpp
template <typename AccessType>
struct global_load<AccessType,
                   32,
                   CacheOperation::LastUse
                  > {
```

**EN:** This block begins the definition of `global_load`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `global_load` 这个 `struct`，其成员会在后续代码中展开。

### Lines 124-152

```cpp
  CUTLASS_DEVICE
  global_load(AccessType &D, void const *ptr, bool pred_guard) {
  uint4 *data = reinterpret_cast<uint4 *>(&D);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
    asm volatile(
        "{\n"
        "  .reg .pred p;\n"
        "  setp.ne.b32 p, %9, 0;\n"
        "  mov.b32 %0, %10;\n"
        "  mov.b32 %1, %11;\n"
        "  mov.b32 %2, %12;\n"
        "  mov.b32 %3, %13;\n"
        "  mov.b32 %4, %14;\n"
        "  mov.b32 %5, %15;\n"
        "  mov.b32 %6, %16;\n"
        "  mov.b32 %7, %17;\n"
        "  @p ld.global.lu.v4.u32 {%0, %1, %2, %3}, [%8];\n"
        "  @p ld.global.lu.v4.u32 {%4, %5, %6, %7}, [%18];\n"
        "}\n"
        : "=r"(data[0].x), "=r"(data[0].y), "=r"(data[0].z), "=r"(data[0].w),
          "=r"(data[1].x), "=r"(data[1].y), "=r"(data[1].z), "=r"(data[1].w)
        : "l"(ptr), "r"((int)pred_guard), "r"(data[0].x), "r"(data[0].y),
          "r"(data[0].z), "r"(data[0].w), "r"(data[1].x), "r"(data[1].y),
          "r"(data[1].z), "r"(data[1].w), "l"(((uint8_t *)ptr) + 16));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
  }
```

**EN:** The function `data` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** `data` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 155-159

```cpp
template <typename AccessType>
struct global_load<AccessType,
                   16,
                   CacheOperation::Always
                  > {
```

**EN:** This block begins the definition of `global_load`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `global_load` 这个 `struct`，其成员会在后续代码中展开。

### Lines 160-184

```cpp
  CUTLASS_DEVICE
  global_load(AccessType &D, void const *ptr, bool pred_guard) {
  uint4 &data = reinterpret_cast<uint4 &>(D);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
    asm volatile(
        "{\n"
        "  .reg .pred p;\n"
        "  setp.ne.b32 p, %5, 0;\n"
        "  mov.b32 %0, %6;\n"
        "  mov.b32 %1, %7;\n"
        "  mov.b32 %2, %8;\n"
        "  mov.b32 %3, %9;\n"
#if CUTLASS_ENABLE_L2_PREFETCH
        "  @p ld.global.L2::128B.v4.u32 {%0, %1, %2, %3}, [%4];\n"
#else
        "  @p ld.global.v4.u32 {%0, %1, %2, %3}, [%4];\n"
#endif
        "}\n"
        : "=r"(data.x), "=r"(data.y), "=r"(data.z), "=r"(data.w)
        : "l"(ptr), "r"((int)pred_guard), "r"(data.x), "r"(data.y), "r"(data.z), "r"(data.w));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
  }
```

**EN:** The function `data` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** `data` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 187-191

```cpp
template <typename AccessType>
struct global_load<AccessType,
                   16,
                   CacheOperation::LastUse
                  > {
```

**EN:** This block begins the definition of `global_load`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `global_load` 这个 `struct`，其成员会在后续代码中展开。

### Lines 192-212

```cpp
  CUTLASS_DEVICE
  global_load(AccessType &D, void const *ptr, bool pred_guard) {
  uint4 &data = reinterpret_cast<uint4 &>(D);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
    asm volatile(
        "{\n"
        "  .reg .pred p;\n"
        "  setp.ne.b32 p, %5, 0;\n"
        "  mov.b32 %0, %6;\n"
        "  mov.b32 %1, %7;\n"
        "  mov.b32 %2, %8;\n"
        "  mov.b32 %3, %9;\n"
        "  @p ld.global.lu.v4.u32 {%0, %1, %2, %3}, [%4];\n"
        "}\n"
        : "=r"(data.x), "=r"(data.y), "=r"(data.z), "=r"(data.w)
        : "l"(ptr), "r"((int)pred_guard), "r"(data.x), "r"(data.y), "r"(data.z), "r"(data.w));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
  }
```

**EN:** The function `data` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** `data` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 215-219

```cpp
template <typename AccessType>
struct global_load<AccessType,
                   8,
                   CacheOperation::Always
                  > {
```

**EN:** This block begins the definition of `global_load`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `global_load` 这个 `struct`，其成员会在后续代码中展开。

### Lines 220-242

```cpp
  CUTLASS_DEVICE
  global_load(AccessType &D, void const *ptr, bool pred_guard) {
  uint2 &data = reinterpret_cast<uint2 &>(D);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
    asm volatile(
        "{\n"
        "  .reg .pred p;\n"
        "  setp.ne.b32 p, %3, 0;\n"
        "  mov.b32 %0, %4;\n"
        "  mov.b32 %1, %5;\n"
#if CUTLASS_ENABLE_L2_PREFETCH
        "  @p ld.global.L2::128B.v2.u32 {%0, %1}, [%2];\n"
#else
        "  @p ld.global.v2.u32 {%0, %1}, [%2];\n"
#endif
        "}\n"
        : "=r"(data.x), "=r"(data.y)
        : "l"(ptr), "r"((int)pred_guard), "r"(data.x), "r"(data.y));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
  }
```

**EN:** The function `data` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** `data` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 245-249

```cpp
template <typename AccessType>
struct global_load<AccessType,
                   8,
                   CacheOperation::LastUse
                  > {
```

**EN:** This block begins the definition of `global_load`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `global_load` 这个 `struct`，其成员会在后续代码中展开。

### Lines 250-268

```cpp
  CUTLASS_DEVICE
  global_load(AccessType &D, void const *ptr, bool pred_guard) {
  uint2 &data = reinterpret_cast<uint2 &>(D);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
    asm volatile(
        "{\n"
        "  .reg .pred p;\n"
        "  setp.ne.b32 p, %3, 0;\n"
        "  mov.b32 %0, %4;\n"
        "  mov.b32 %1, %5;\n"
        "  @p ld.global.lu.v2.u32 {%0, %1}, [%2];\n"
        "}\n"
        : "=r"(data.x), "=r"(data.y)
        : "l"(ptr), "r"((int)pred_guard), "r"(data.x), "r"(data.y));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
  }
```

**EN:** The function `data` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** `data` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 271-275

```cpp
template <typename AccessType>
struct global_load<AccessType,
                   4,
                   CacheOperation::Always
                  > {
```

**EN:** This block begins the definition of `global_load`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `global_load` 这个 `struct`，其成员会在后续代码中展开。

### Lines 276-297

```cpp
  CUTLASS_DEVICE
  global_load(AccessType &D, void const *ptr, bool pred_guard) {
  unsigned &data = reinterpret_cast<unsigned &>(D);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
    asm volatile(
        "{\n"
        "  .reg .pred p;\n"
        "  setp.ne.b32 p, %2, 0;\n"
        "  mov.b32 %0, %3;\n"
#if CUTLASS_ENABLE_L2_PREFETCH
        "  @p ld.global.L2::128B.u32 %0, [%1];\n"
#else
        "  @p ld.global.u32 %0, [%1];\n"
#endif
        "}\n"
        : "=r"(data)
        : "l"(ptr), "r"((int)pred_guard), "r"(data));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
  }
```

**EN:** The function `data` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** `data` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 300-304

```cpp
template <typename AccessType>
struct global_load<AccessType,
                   4,
                   CacheOperation::LastUse
                  > {
```

**EN:** This block begins the definition of `global_load`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `global_load` 这个 `struct`，其成员会在后续代码中展开。

### Lines 305-322

```cpp
  CUTLASS_DEVICE
  global_load(AccessType &D, void const *ptr, bool pred_guard) {
  unsigned &data = reinterpret_cast<unsigned &>(D);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
    asm volatile(
        "{\n"
        "  .reg .pred p;\n"
        "  setp.ne.b32 p, %2, 0;\n"
        "  mov.b32 %0, %3;\n"
        "  @p ld.global.lu.u32 %0, [%1];\n"
        "}\n"
        : "=r"(data)
        : "l"(ptr), "r"((int)pred_guard), "r"(data));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
  }
```

**EN:** The function `data` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** `data` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 325-329

```cpp
template <typename AccessType>
struct global_load<AccessType,
                   2,
                   CacheOperation::Always
                  > {
```

**EN:** This block begins the definition of `global_load`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `global_load` 这个 `struct`，其成员会在后续代码中展开。

### Lines 330-351

```cpp
  CUTLASS_DEVICE
  global_load(AccessType &D, void const *ptr, bool pred_guard) {
  uint16_t &data = reinterpret_cast<uint16_t &>(D);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
    asm volatile(
        "{\n"
        "  .reg .pred p;\n"
        "  setp.ne.b32 p, %2, 0;\n"
        "  mov.b16 %0, %3;\n"
#if CUTLASS_ENABLE_L2_PREFETCH
        "  @p ld.global.L2::128B.u16 %0, [%1];\n"
#else
        "  @p ld.global.u16 %0, [%1];\n"
#endif
        "}\n"
        : "=h"(data)
        : "l"(ptr), "r"((int)pred_guard), "h"(data));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
  }
```

**EN:** The function `data` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** `data` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 354-358

```cpp
template <typename AccessType>
struct global_load<AccessType,
                   2,
                   CacheOperation::LastUse
                  > {
```

**EN:** This block begins the definition of `global_load`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `global_load` 这个 `struct`，其成员会在后续代码中展开。

### Lines 359-376

```cpp
  CUTLASS_DEVICE
  global_load(AccessType &D, void const *ptr, bool pred_guard) {
  uint16_t &data = reinterpret_cast<uint16_t &>(D);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
    asm volatile(
        "{\n"
        "  .reg .pred p;\n"
        "  setp.ne.b32 p, %2, 0;\n"
        "  mov.b16 %0, %3;\n"
        "  @p ld.global.lu.u16 %0, [%1];\n"
        "}\n"
        : "=h"(data)
        : "l"(ptr), "r"((int)pred_guard), "h"(data));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
  }
```

**EN:** The function `data` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** `data` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 379-383

```cpp
template <typename AccessType>
struct global_load<AccessType,
                   1,
                   CacheOperation::Always
                  > {
```

**EN:** This block begins the definition of `global_load`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `global_load` 这个 `struct`，其成员会在后续代码中展开。

### Lines 384-387

```cpp
  CUTLASS_DEVICE
  global_load(AccessType &D, void const *ptr, bool pred_guard) {
    if (pred_guard) D = *(reinterpret_cast<AccessType const *>(ptr));
  }
```

**EN:** The function `D` implements a concrete operation in this abstraction. It reinterprets raw storage to expose a typed view.

**CN:** `D` 函数实现了该抽象中的一个具体操作。它通过重新解释原始存储来暴露带类型的视图。

### Lines 390-397

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
template <
    /// Fragment type to store data
    typename AccessType,
    /// The bytes of storing
    int StoreBytes
    >
struct global_store;
```

**EN:** The preceding comment documents this block. This block begins the definition of `global_store`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `global_store` 这个 `struct`，其成员会在后续代码中展开。

### Lines 400-406

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
//
// Specializations
//
/////////////////////////////////////////////////////////////////////////////////////////////////
template <typename AccessType>
struct global_store<AccessType, 64> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `global_store`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `global_store` 这个 `struct`，其成员会在后续代码中展开。

### Lines 409-434

```cpp
  CUTLASS_DEVICE
  global_store(AccessType const &D, void *ptr, bool pred_guard) {
  uint4 const *data = reinterpret_cast<uint4 const *>(&D);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile(
      "{\n"
      "  .reg .pred p;\n"
      "  setp.ne.b32 p, %5, 0;\n"
      "  @p st.global.v4.u32 [%0], {%1, %2, %3, %4};\n"
      "  @p st.global.v4.u32 [%6], {%7, %8, %9, %10};\n"
      "  @p st.global.v4.u32 [%11], {%12, %13, %14, %15};\n"
      "  @p st.global.v4.u32 [%16], {%17, %18, %19, %20};\n"
      "}\n"
      :
      : "l"(ptr), "r"(data[0].x), "r"(data[0].y), "r"(data[0].z),
        "r"(data[0].w), "r"((int)pred_guard), "l"(((uint8_t *)ptr) + 16),
        "r"(data[1].x), "r"(data[1].y), "r"(data[1].z), "r"(data[1].w), 
        "l"(((uint8_t *)ptr) + 32),
        "r"(data[2].x), "r"(data[2].y), "r"(data[2].z), "r"(data[2].w),
        "l"(((uint8_t *)ptr) + 48),
        "r"(data[3].x), "r"(data[3].y), "r"(data[3].z), "r"(data[3].w));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
  }
```

**EN:** The function `data` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** `data` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 438-439

```cpp
template <typename AccessType>
struct global_store<AccessType, 32> {
```

**EN:** This block begins the definition of `global_store`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `global_store` 这个 `struct`，其成员会在后续代码中展开。

### Lines 440-459

```cpp
  CUTLASS_DEVICE
  global_store(AccessType const &D, void *ptr, bool pred_guard) {
  uint4 const *data = reinterpret_cast<uint4 const *>(&D);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile(
      "{\n"
      "  .reg .pred p;\n"
      "  setp.ne.b32 p, %5, 0;\n"
      "  @p st.global.v4.u32 [%0], {%1, %2, %3, %4};\n"
      "  @p st.global.v4.u32 [%6], {%7, %8, %9, %10};\n"
      "}\n"
      :
      : "l"(ptr), "r"(data[0].x), "r"(data[0].y), "r"(data[0].z),
        "r"(data[0].w), "r"((int)pred_guard), "l"(((uint8_t *)ptr) + 16),
        "r"(data[1].x), "r"(data[1].y), "r"(data[1].z), "r"(data[1].w));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
  }
```

**EN:** The function `data` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** `data` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 462-463

```cpp
template <typename AccessType>
struct global_store<AccessType, 16> {
```

**EN:** This block begins the definition of `global_store`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `global_store` 这个 `struct`，其成员会在后续代码中展开。

### Lines 464-480

```cpp
  CUTLASS_DEVICE
  global_store(AccessType const &D, void *ptr, bool pred_guard) {
  uint4 const &data = reinterpret_cast<uint4 const &>(D);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile(
      "{\n"
      "  .reg .pred p;\n"
      "  setp.ne.b32 p, %5, 0;\n"
      "  @p st.global.v4.u32 [%0], {%1, %2, %3, %4};\n"
      "}\n"
      :
      : "l"(ptr), "r"(data.x), "r"(data.y), "r"(data.z), "r"(data.w), "r"((int)pred_guard));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
  }
```

**EN:** The function `data` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** `data` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 483-484

```cpp
template <typename AccessType>
struct global_store<AccessType, 8> {
```

**EN:** This block begins the definition of `global_store`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `global_store` 这个 `struct`，其成员会在后续代码中展开。

### Lines 485-501

```cpp
  CUTLASS_DEVICE
  global_store(AccessType const &D, void *ptr, bool pred_guard) {
  uint2 const &data = reinterpret_cast<uint2 const &>(D);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile(
      "{\n"
      "  .reg .pred p;\n"
      "  setp.ne.b32 p, %3, 0;\n"
      "  @p st.global.v2.u32 [%0], {%1, %2};\n"
      "}\n"
      :
      : "l"(ptr), "r"(data.x), "r"(data.y), "r"((int)pred_guard));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
  }
```

**EN:** The function `data` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** `data` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 504-505

```cpp
template <typename AccessType>
struct global_store<AccessType, 4> {
```

**EN:** This block begins the definition of `global_store`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `global_store` 这个 `struct`，其成员会在后续代码中展开。

### Lines 506-522

```cpp
  CUTLASS_DEVICE
  global_store(AccessType const &D, void *ptr, bool pred_guard) {
  uint32_t const &data = reinterpret_cast<uint32_t const &>(D);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
    asm volatile(
      "{\n"
      "  .reg .pred p;\n"
      "  setp.ne.b32 p, %2, 0;\n"
      "  @p st.global.u32 [%0], %1;\n"
      "}\n"
      :
      : "l"(ptr), "r"(data), "r"((int)pred_guard));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
  }
```

**EN:** The function `data` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** `data` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 525-526

```cpp
template <typename AccessType>
struct global_store<AccessType, 2> {
```

**EN:** This block begins the definition of `global_store`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `global_store` 这个 `struct`，其成员会在后续代码中展开。

### Lines 527-543

```cpp
  CUTLASS_DEVICE
  global_store(AccessType const &D, void *ptr, bool pred_guard) {
  uint16_t const &data = reinterpret_cast<uint16_t const &>(D);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile(
      "{\n"
      "  .reg .pred p;\n"
      "  setp.ne.b32 p, %2, 0;\n"
      "  @p st.global.u16 [%0], %1;\n"
      "}\n"
      :
      : "l"(ptr), "h"(data), "r"((int)pred_guard));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
  }
```

**EN:** The function `data` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** `data` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 546-547

```cpp
template <typename AccessType>
struct global_store<AccessType, 1> {
```

**EN:** This block begins the definition of `global_store`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `global_store` 这个 `struct`，其成员会在后续代码中展开。

### Lines 548-551

```cpp
  CUTLASS_DEVICE
  global_store(AccessType const &D, void *ptr, bool pred_guard) {
    if (pred_guard) *(reinterpret_cast<AccessType *>(ptr)) = D;
  }
```

**EN:** The function `D` implements a concrete operation in this abstraction. It reinterprets raw storage to expose a typed view.

**CN:** `D` 函数实现了该抽象中的一个具体操作。它通过重新解释原始存储来暴露带类型的视图。

### Lines 557-560

```cpp
/// ld.shared
template <int Bytes>
CUTLASS_DEVICE
void shared_load(void *dst, uint32_t ptr);
```

**EN:** The preceding comment documents this block. The function `shared_load` implements a concrete operation in this abstraction. It packages a focused unit of behavior for the surrounding class, specialization, or namespace.

**CN:** 前面的注释说明了这个代码块。`shared_load` 函数实现了该抽象中的一个具体操作。它为周围的类、特化或命名空间封装了一个聚焦的行为单元。

### Lines 562-574

```cpp
/// ld.shared - 16b
template <>
CUTLASS_DEVICE
void shared_load<2>(void *dst, uint32_t ptr) {
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile("ld.shared.u16 %0, [%1];\n"
    : "=h"(*reinterpret_cast<uint16_t *>(dst))
    : "r"(ptr));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
}
```

**EN:** The preceding comment documents this block. The function `this entity` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** 前面的注释说明了这个代码块。`this entity` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 576-588

```cpp
/// ld.shared - 32b
template <>
CUTLASS_DEVICE
void shared_load<4>(void *dst, uint32_t ptr) {
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile("ld.shared.u32 %0, [%1];\n"
    : "=r"(*reinterpret_cast<uint32_t *>(dst))
    : "r"(ptr));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
}
```

**EN:** The preceding comment documents this block. The function `this entity` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** 前面的注释说明了这个代码块。`this entity` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 590-605

```cpp
/// ld.shared - 64b
template <>
CUTLASS_DEVICE
void shared_load<8>(void *dst, uint32_t ptr) {
  uint2 *dst_u64 = reinterpret_cast<uint2 *>(dst);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile("ld.shared.v2.u32 {%0, %1}, [%2];\n"
    :
      "=r"(dst_u64->x),
      "=r"(dst_u64->y)
    : "r"(ptr));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
}
```

**EN:** The preceding comment documents this block. The function `dst_u64` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** 前面的注释说明了这个代码块。`dst_u64` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 607-624

```cpp
/// ld.shared - 128b
template <>
CUTLASS_DEVICE
void shared_load<16>(void *dst, uint32_t ptr) {
  uint4 *dst_u128 = reinterpret_cast<uint4 *>(dst);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile("ld.shared.v4.u32 {%0, %1, %2, %3}, [%4];\n"
    :
      "=r"(dst_u128->x),
      "=r"(dst_u128->y),
      "=r"(dst_u128->z),
      "=r"(dst_u128->w)
    : "r"(ptr));
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
}
```

**EN:** The preceding comment documents this block. The function `dst_u128` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** 前面的注释说明了这个代码块。`dst_u128` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 629-632

```cpp
/// st.shared
template <int Bytes>
CUTLASS_DEVICE
void shared_store(uint32_t ptr, void const *src);
```

**EN:** The preceding comment documents this block. The function `shared_store` implements a concrete operation in this abstraction. It packages a focused unit of behavior for the surrounding class, specialization, or namespace.

**CN:** 前面的注释说明了这个代码块。`shared_store` 函数实现了该抽象中的一个具体操作。它为周围的类、特化或命名空间封装了一个聚焦的行为单元。

### Lines 634-648

```cpp
/// st.shared - 16b
template <>
CUTLASS_DEVICE
void shared_store<2>(uint32_t ptr, void const *src) {
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile("st.shared.u16 [%0], %1;\n"
    : :
    "r"(ptr),
    "h"(*reinterpret_cast<uint16_t const *>(src))
  );
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
}
```

**EN:** The preceding comment documents this block. The function `shared_store<2>` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** 前面的注释说明了这个代码块。`shared_store<2>` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 650-664

```cpp
/// st.shared - 32b
template <>
CUTLASS_DEVICE
void shared_store<4>(uint32_t ptr, void const *src) {
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile("st.shared.u32 [%0], %1;\n"
    : :
    "r"(ptr),
    "r"(*reinterpret_cast<uint32_t const  *>(src))
  );
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
}
```

**EN:** The preceding comment documents this block. The function `shared_store<4>` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** 前面的注释说明了这个代码块。`shared_store<4>` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 666-682

```cpp
/// st.shared - 64b
template <>
CUTLASS_DEVICE
void shared_store<8>(uint32_t ptr, void const *src) {
  uint2 const *dst_u64 = reinterpret_cast<uint2 const *>(src);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile("st.shared.v2.u32 [%0], {%1, %2};\n"
    : :
      "r"(ptr),
      "r"(dst_u64->x),
      "r"(dst_u64->y)
    );
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
}
```

**EN:** The preceding comment documents this block. The function `dst_u64` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** 前面的注释说明了这个代码块。`dst_u64` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 684-702

```cpp
/// st.shared - 128b
template <>
CUTLASS_DEVICE
void shared_store<16>(uint32_t ptr, void const *src) {
  uint4 const *dst_u128 = reinterpret_cast<uint4 const *>(src);
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile("st.shared.v4.u32 [%0], {%1, %2, %3, %4};\n"
    : :
      "r"(ptr),
      "r"(dst_u128->x),
      "r"(dst_u128->y),
      "r"(dst_u128->z),
      "r"(dst_u128->w)
    );
#else
  CUTE_INVALID_CONTROL_PATH(
      "Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
}
```

**EN:** The preceding comment documents this block. The function `dst_u128` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view.

**CN:** 前面的注释说明了这个代码块。`dst_u128` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。

### Lines 711-712

```cpp
#include "cutlass/arch/memory_sm75.h"
#include "cutlass/arch/memory_sm80.h"
```

**EN:** This block imports dependencies such as `cutlass/arch/memory_sm75.h`, `cutlass/arch/memory_sm80.h`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cutlass/arch/memory_sm75.h`, `cutlass/arch/memory_sm80.h` 等依赖，为后续代码提供类型、宏或辅助例程。

## Key Concepts / 关键概念

- **EN:** Heavy use of templates enables compile-time specialization and zero-overhead abstractions.
  **CN:** 大量使用模板，使该文件能够进行编译期特化并保持零额外开销的抽象。

- **EN:** Architecture-specific paths map C++ wrappers onto GPU instructions or micro-architectural features.
  **CN:** 架构特定路径会把 C++ 封装映射到 GPU 指令或底层微架构特性。

- **EN:** Conditional compilation keeps the header portable across host, device, CUDA, and SYCL builds.
  **CN:** 条件编译使该头文件能够在主机端、设备端、CUDA 与 SYCL 构建之间保持可移植性。

## Dependencies / 依赖关系

- **EN:** Direct includes: `cutlass/cutlass.h`, `cutlass/arch/cache_operation.h`, `cutlass/platform/platform.h`, `cute/config.hpp`, `cutlass/arch/memory_sm75.h`, `cutlass/arch/memory_sm80.h`.
  **CN:** 直接包含：`cutlass/cutlass.h`, `cutlass/arch/cache_operation.h`, `cutlass/platform/platform.h`, `cute/config.hpp`, `cutlass/arch/memory_sm75.h`, `cutlass/arch/memory_sm80.h`。

- **EN:** Primary namespaces: `cutlass`, `arch`.
  **CN:** 主要命名空间：`cutlass`, `arch`。

- **EN:** Important macros or compile flags: `CUTLASS_DEVICE`, `CUTLASS_ENABLE_L2_PREFETCH`.
  **CN:** 重要宏或编译开关：`CUTLASS_DEVICE`, `CUTLASS_ENABLE_L2_PREFETCH`。
