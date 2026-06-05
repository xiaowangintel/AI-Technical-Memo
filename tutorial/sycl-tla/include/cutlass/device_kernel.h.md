# device_kernel.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/device_kernel.h`

- **EN:** Template for generic CUTLASS kernel.

- **CN:** 该头文件属于 CUTLASS 的核心类型或工具定义。文件级摘要：Template for generic CUTLASS kernel.

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
    \brief Template for generic CUTLASS kernel.
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

### Lines 37-39

```cpp
#include <cutlass/detail/helper_macros.hpp> // CUTLASS_HOST_DEVICE
#include <cutlass/arch/synclog.hpp>  // cutlass::arch::synclog_*
#include <cutlass/platform/platform.h> // uint64_t
```

**EN:** This block imports dependencies such as `cutlass/detail/helper_macros.hpp`, `cutlass/arch/synclog.hpp`, `cutlass/platform/platform.h`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cutlass/detail/helper_macros.hpp`, `cutlass/arch/synclog.hpp`, `cutlass/platform/platform.h` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 41-41

```cpp
// __grid_constant__ was introduced in CUDA 11.7.
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 42-42

```cpp
#if ((__CUDACC_VER_MAJOR__ >= 12) || ((__CUDACC_VER_MAJOR__ == 11) && (__CUDACC_VER_MINOR__ >= 7)))
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#if ((__CUDACC_VER_MAJOR__ >= 12) || ((__CUDACC_VER_MAJOR__ == 11) && (__CUDACC_VER_MINOR__ >= 7)))`.

**CN:** 这个预处理代码块围绕 `#if ((__CUDACC_VER_MAJOR__ >= 12) || ((__CUDACC_VER_MAJOR__ == 11) && (__CUDACC_VER_MINOR__ >= 7)))` 选择编译路径或功能开关。

### Lines 43-43

```cpp
#  define CUTLASS_GRID_CONSTANT_SUPPORTED
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#  define CUTLASS_GRID_CONSTANT_SUPPORTED`.

**CN:** 这个预处理代码块围绕 `#  define CUTLASS_GRID_CONSTANT_SUPPORTED` 选择编译路径或功能开关。

### Lines 44-44

```cpp
#endif
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#endif`.

**CN:** 这个预处理代码块围绕 `#endif` 选择编译路径或功能开关。

### Lines 46-46

```cpp
// __grid_constant__ can be enabled only on SM70+
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 47-47

```cpp
#if defined(CUTLASS_GRID_CONSTANT_SUPPORTED) && defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 700)
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#if defined(CUTLASS_GRID_CONSTANT_SUPPORTED) && defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 700)`.

**CN:** 这个预处理代码块围绕 `#if defined(CUTLASS_GRID_CONSTANT_SUPPORTED) && defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 700)` 选择编译路径或功能开关。

### Lines 48-48

```cpp
#  define CUTLASS_GRID_CONSTANT_ENABLED
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#  define CUTLASS_GRID_CONSTANT_ENABLED`.

**CN:** 这个预处理代码块围绕 `#  define CUTLASS_GRID_CONSTANT_ENABLED` 选择编译路径或功能开关。

### Lines 49-49

```cpp
#endif
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#endif`.

**CN:** 这个预处理代码块围绕 `#endif` 选择编译路径或功能开关。

### Lines 51-51

```cpp
#if ! defined(CUTLASS_GRID_CONSTANT)
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#if ! defined(CUTLASS_GRID_CONSTANT)`.

**CN:** 这个预处理代码块围绕 `#if ! defined(CUTLASS_GRID_CONSTANT)` 选择编译路径或功能开关。

### Lines 52-52

```cpp
#  if defined(CUTLASS_GRID_CONSTANT_ENABLED)
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#  if defined(CUTLASS_GRID_CONSTANT_ENABLED)`.

**CN:** 这个预处理代码块围绕 `#  if defined(CUTLASS_GRID_CONSTANT_ENABLED)` 选择编译路径或功能开关。

### Lines 53-53

```cpp
#    define CUTLASS_GRID_CONSTANT __grid_constant__
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#    define CUTLASS_GRID_CONSTANT __grid_constant__`.

**CN:** 这个预处理代码块围绕 `#    define CUTLASS_GRID_CONSTANT __grid_constant__` 选择编译路径或功能开关。

### Lines 54-54

```cpp
#  else
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#  else`.

**CN:** 这个预处理代码块围绕 `#  else` 选择编译路径或功能开关。

### Lines 55-55

```cpp
#    define CUTLASS_GRID_CONSTANT
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#    define CUTLASS_GRID_CONSTANT`.

**CN:** 这个预处理代码块围绕 `#    define CUTLASS_GRID_CONSTANT` 选择编译路径或功能开关。

### Lines 56-56

```cpp
#  endif
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#  endif`.

**CN:** 这个预处理代码块围绕 `#  endif` 选择编译路径或功能开关。

### Lines 57-57

```cpp
#endif
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#endif`.

**CN:** 这个预处理代码块围绕 `#endif` 选择编译路径或功能开关。

### Lines 59-60

```cpp
////////////////////////////////////////////////////////////////////////////////
namespace cutlass {
```

**EN:** The preceding comment documents this block. This block opens the namespace scope `cutlass` for the declarations that follow.

**CN:** 前面的注释说明了这个代码块。该代码块打开了 `cutlass` 命名空间作用域，以容纳后续声明。

### Lines 72-72

```cpp
/// Generic CUTLASS kernel template.
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 63-68

```cpp
template <typename T>   struct Type2Type  {  using type=T;                    };
// using the simple type to replace the complex type to reduce this symbol size
template <typename  T>                                                                        struct GetUnderlyingKernel                              : public Type2Type<T>               {};
template <uint64_t shader_guid, unsigned index, template <uint64_t, unsigned> class Wrapper > struct GetUnderlyingKernel<Wrapper<shader_guid,index>>  : public Wrapper<shader_guid,index> {};
template <typename  T>                                                                        using  GetUnderlyingKernel_t                            = typename GetUnderlyingKernel<T>::type;
#if defined(CUTLASS_ENABLE_SYCL) && !defined(SYCL_EXT_ONEAPI_WORK_GROUP_SCRATCH_MEMORY)
```

**EN:** The preceding comment documents this block. The function `defined` implements a concrete operation in this abstraction. It packages a focused unit of behavior for the surrounding class, specialization, or namespace.

**CN:** 前面的注释说明了这个代码块。`defined` 函数实现了该抽象中的一个具体操作。它为周围的类、特化或命名空间封装了一个聚焦的行为单元。

### Lines 74-86

```cpp
template <typename Operator>
void Kernel(typename Operator::Params params, char* smem) {
  // Dynamic shared memory base pointer
  int* SharedStorageBase = reinterpret_cast<int*>(smem);
  // Declare pointer to dynamic shared memory.
  typename Operator::SharedStorage *shared_storage =
      reinterpret_cast<typename Operator::SharedStorage *>(SharedStorageBase);

  Operator op;

  op(params, *shared_storage);
  cutlass::arch::synclog_print();
}
```

**EN:** The preceding comment documents this block. The function `SharedStorageBase` implements a concrete operation in this abstraction. It reinterprets raw storage to expose a typed view.

**CN:** 前面的注释说明了这个代码块。`SharedStorageBase` 函数实现了该抽象中的一个具体操作。它通过重新解释原始存储来暴露带类型的视图。

### Lines 87-87

```cpp
#else
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#else`.

**CN:** 这个预处理代码块围绕 `#else` 选择编译路径或功能开关。

### Lines 88-106

```cpp
template <typename Operator>
CUTLASS_GLOBAL
void Kernel(typename Operator::Params params) {
  // Dynamic shared memory base pointer
#if defined(CUTLASS_ENABLE_SYCL)
  int* SharedStorageBase = static_cast<int*>(
      sycl::ext::oneapi::experimental::get_work_group_scratch_memory());
#else
  extern __shared__ int SharedStorageBase[];
#endif
  // Declare pointer to dynamic shared memory.
  typename Operator::SharedStorage *shared_storage =
      reinterpret_cast<typename Operator::SharedStorage *>(SharedStorageBase);

  Operator op;

  op(params, *shared_storage);
  cutlass::arch::synclog_print();
}
```

**EN:** The preceding comment documents this block. The function `SharedStorageBase` implements a concrete operation in this abstraction. It reinterprets raw storage to expose a typed view. It performs explicit type conversion to keep the representation precise.

**CN:** 前面的注释说明了这个代码块。`SharedStorageBase` 函数实现了该抽象中的一个具体操作。它通过重新解释原始存储来暴露带类型的视图。它通过显式类型转换来保持表示精确。

### Lines 107-107

```cpp
#endif
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#endif`.

**CN:** 这个预处理代码块围绕 `#endif` 选择编译路径或功能开关。

### Lines 109-109

```cpp
/// Generic CUTLASS kernel template.
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 110-110

```cpp
#if defined(CUTLASS_ENABLE_SYCL) && !defined(SYCL_EXT_ONEAPI_WORK_GROUP_SCRATCH_MEMORY)
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#if defined(CUTLASS_ENABLE_SYCL) && !defined(SYCL_EXT_ONEAPI_WORK_GROUP_SCRATCH_MEMORY)`.

**CN:** 这个预处理代码块围绕 `#if defined(CUTLASS_ENABLE_SYCL) && !defined(SYCL_EXT_ONEAPI_WORK_GROUP_SCRATCH_MEMORY)` 选择编译路径或功能开关。

### Lines 111-122

```cpp
template <typename Operator>
void Kernel2(typename Operator::Params params, char* smem) {
  // Dynamic shared memory base pointer
  int* SharedStorageBase = reinterpret_cast<int*>(smem);
  // Declare pointer to dynamic shared memory.
  typename Operator::SharedStorage *shared_storage =
      reinterpret_cast<typename Operator::SharedStorage *>(SharedStorageBase);

  Operator::invoke(params, *shared_storage);
  cutlass::arch::synclog_print();

}
```

**EN:** The preceding comment documents this block. The function `SharedStorageBase` implements a concrete operation in this abstraction. It reinterprets raw storage to expose a typed view.

**CN:** 前面的注释说明了这个代码块。`SharedStorageBase` 函数实现了该抽象中的一个具体操作。它通过重新解释原始存储来暴露带类型的视图。

### Lines 123-123

```cpp
#else
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#else`.

**CN:** 这个预处理代码块围绕 `#else` 选择编译路径或功能开关。

### Lines 124-142

```cpp
template <typename Operator>
CUTLASS_GLOBAL
void Kernel2(typename Operator::Params params) {
  // Dynamic shared memory base pointer
#if defined(CUTLASS_ENABLE_SYCL)
  int* SharedStorageBase = static_cast<int*>(
      sycl::ext::oneapi::experimental::get_work_group_scratch_memory());

#else
  extern __shared__ int SharedStorageBase[];
#endif
  // Declare pointer to dynamic shared memory.
  typename Operator::SharedStorage *shared_storage =
      reinterpret_cast<typename Operator::SharedStorage *>(SharedStorageBase);

  Operator::invoke(params, *shared_storage);
  cutlass::arch::synclog_print();

}
```

**EN:** The preceding comment documents this block. The function `SharedStorageBase` implements a concrete operation in this abstraction. It reinterprets raw storage to expose a typed view. It performs explicit type conversion to keep the representation precise.

**CN:** 前面的注释说明了这个代码块。`SharedStorageBase` 函数实现了该抽象中的一个具体操作。它通过重新解释原始存储来暴露带类型的视图。它通过显式类型转换来保持表示精确。

### Lines 143-143

```cpp
#endif
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#endif`.

**CN:** 这个预处理代码块围绕 `#endif` 选择编译路径或功能开关。

### Lines 146-150

```cpp
////////////////////////////////////////////////////////////////////////////////
//
// 3.0 specific launch
//
////////////////////////////////////////////////////////////////////////////////
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 152-152

```cpp
/// Generic CUTLASS kernel template.
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 153-153

```cpp
#if defined(CUTLASS_ENABLE_SYCL) && !defined(SYCL_EXT_ONEAPI_WORK_GROUP_SCRATCH_MEMORY)
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#if defined(CUTLASS_ENABLE_SYCL) && !defined(SYCL_EXT_ONEAPI_WORK_GROUP_SCRATCH_MEMORY)`.

**CN:** 这个预处理代码块围绕 `#if defined(CUTLASS_ENABLE_SYCL) && !defined(SYCL_EXT_ONEAPI_WORK_GROUP_SCRATCH_MEMORY)` 选择编译路径或功能开关。

### Lines 154-160

```cpp
template <typename Operator>
void device_kernel(typename Operator::Params const& params, sycl::local_ptr<char> smem)
{
  Operator op;
  op(params, smem);
  cutlass::arch::synclog_print();
}
```

**EN:** The function `device_kernel` implements a concrete operation in this abstraction. It packages a focused unit of behavior for the surrounding class, specialization, or namespace.

**CN:** `device_kernel` 函数实现了该抽象中的一个具体操作。它为周围的类、特化或命名空间封装了一个聚焦的行为单元。

### Lines 161-161

```cpp
#else
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#else`.

**CN:** 这个预处理代码块围绕 `#else` 选择编译路径或功能开关。

### Lines 162-183

```cpp
template <typename Operator>
CUTLASS_GLOBAL
#ifdef __CUDACC__
// Enclosing this in __CUDACC__ suppresses MSVC warnings.
__launch_bounds__(Operator::MaxThreadsPerBlock, Operator::MinBlocksPerMultiprocessor)
#endif // __CUDACC__
#if defined(CUTLASS_ENABLE_SYCL)
void device_kernel(typename Operator::Params const& params)
#else
void device_kernel(CUTLASS_GRID_CONSTANT typename Operator::Params const params)
#endif
{
  // Dynamic shared memory base pointer
#if defined(CUTLASS_ENABLE_SYCL)
  char* smem = static_cast<char*>(sycl::ext::oneapi::experimental::get_work_group_scratch_memory());
#else
  extern __shared__ char smem[];
#endif
  Operator op;
  op(params, smem);
  cutlass::arch::synclog_print();
}
```

**EN:** The preceding comment documents this block. The function `smem` implements a concrete operation in this abstraction. It performs explicit type conversion to keep the representation precise.

**CN:** 前面的注释说明了这个代码块。`smem` 函数实现了该抽象中的一个具体操作。它通过显式类型转换来保持表示精确。

### Lines 184-184

```cpp
#endif // defined(CUTLASS_ENABLE_SYCL) && !defined(SYCL_EXT_ONEAPI_WORK_GROUP_SCRATCH_MEMORY)
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#endif // defined(CUTLASS_ENABLE_SYCL) && !defined(SYCL_EXT_ONEAPI_WORK_GROUP_SCRATCH_MEMORY)`.

**CN:** 这个预处理代码块围绕 `#endif // defined(CUTLASS_ENABLE_SYCL) && !defined(SYCL_EXT_ONEAPI_WORK_GROUP_SCRATCH_MEMORY)` 选择编译路径或功能开关。

## Key Concepts / 关键概念

- **EN:** Heavy use of templates enables compile-time specialization and zero-overhead abstractions.
  **CN:** 大量使用模板，使该文件能够进行编译期特化并保持零额外开销的抽象。

- **EN:** Fixed-size containers and fragments are used to model register or shared-memory tiles.
  **CN:** 定长容器与片段类型用于建模寄存器块或共享内存块。

- **EN:** Conditional compilation keeps the header portable across host, device, CUDA, and SYCL builds.
  **CN:** 条件编译使该头文件能够在主机端、设备端、CUDA 与 SYCL 构建之间保持可移植性。

## Dependencies / 依赖关系

- **EN:** Direct includes: `cutlass/detail/helper_macros.hpp`, `cutlass/arch/synclog.hpp`, `cutlass/platform/platform.h`.
  **CN:** 直接包含：`cutlass/detail/helper_macros.hpp`, `cutlass/arch/synclog.hpp`, `cutlass/platform/platform.h`。

- **EN:** Primary namespaces: `cutlass`.
  **CN:** 主要命名空间：`cutlass`。

- **EN:** Important macros or compile flags: `CUTLASS_ENABLE_SYCL`, `CUTLASS_GLOBAL`, `CUTLASS_GRID_CONSTANT`, `CUTLASS_GRID_CONSTANT_ENABLED`, `CUTLASS_GRID_CONSTANT_SUPPORTED`, `CUTLASS_HOST_DEVICE`.
  **CN:** 重要宏或编译开关：`CUTLASS_ENABLE_SYCL`, `CUTLASS_GLOBAL`, `CUTLASS_GRID_CONSTANT`, `CUTLASS_GRID_CONSTANT_ENABLED`, `CUTLASS_GRID_CONSTANT_SUPPORTED`, `CUTLASS_HOST_DEVICE`。
