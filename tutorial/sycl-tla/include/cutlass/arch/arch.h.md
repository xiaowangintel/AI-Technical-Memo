# arch.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/arch/arch.h`

- **EN:** Defines tags for architecture-specific configurations.

- **CN:** 该头文件主要实现 CUTLASS 的架构相关原语与指令封装。文件级摘要：Defines tags for architecture-specific configurations.

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
    \brief Defines tags for architecture-specific configurations.
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

### Lines 37-38

```cpp
#include "cutlass/cutlass.h"
#include <cute/config.hpp>
```

**EN:** This block imports dependencies such as `cutlass/cutlass.h`, `cute/config.hpp`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cutlass/cutlass.h`, `cute/config.hpp` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 40-41

```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////
namespace cutlass {
```

**EN:** The preceding comment documents this block. This block opens the namespace scope `cutlass` for the declarations that follow.

**CN:** 前面的注释说明了这个代码块。该代码块打开了 `cutlass` 命名空间作用域，以容纳后续声明。

### Lines 43-43

```cpp
namespace arch {
```

**EN:** This block opens the namespace scope `arch` for the declarations that follow.

**CN:** 该代码块打开了 `arch` 命名空间作用域，以容纳后续声明。

### Lines 45-45

```cpp
constexpr int sm100_smem_capacity_bytes = 232448;  
```

**EN:** This declaration defines `sm100_smem_capacity_bytes` and assigns it the compile-time expression `232448`.

**CN:** 这个声明定义了 `sm100_smem_capacity_bytes`，并把它设为编译期表达式 `232448`。

### Lines 46-46

```cpp
constexpr int sm120_smem_capacity_bytes = 101376;
```

**EN:** This declaration defines `sm120_smem_capacity_bytes` and assigns it the compile-time expression `101376`.

**CN:** 这个声明定义了 `sm120_smem_capacity_bytes`，并把它设为编译期表达式 `101376`。

### Lines 48-48

```cpp
#if defined(__NVCC__) || defined(__CUDACC_RTC__) || (defined(__clang__) && (defined(__CUDA__)) || defined(CUTLASS_ENABLE_SYCL))
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#if defined(__NVCC__) || defined(__CUDACC_RTC__) || (defined(__clang__) && (defined(__CUDA__)) || defined(CUTLASS_ENABLE_SYCL))`.

**CN:** 这个预处理代码块围绕 `#if defined(__NVCC__) || defined(__CUDACC_RTC__) || (defined(__clang__) && (defined(__CUDA__)) || defined(CUTLASS_ENABLE_SYCL))` 选择编译路径或功能开关。

### Lines 50-60

```cpp
/// Computes laneId within a warp
CUTLASS_DEVICE
int LaneId() {
  int ret;
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm ("mov.u32 %0, %%laneid;" : "=r"(ret) : );
  return ret;
#else
  CUTE_INVALID_CONTROL_PATH("Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
}
```

**EN:** The preceding comment documents this block. The function `ret` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction.

**CN:** 前面的注释说明了这个代码块。`ret` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。

### Lines 62-72

```cpp
/// Computes SM number the thread is running on
CUTLASS_DEVICE
int SmId() {
  int ret;
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm ("mov.u32 %0, %%smid;" : "=r"(ret) : );
  return ret;
#else
  CUTE_INVALID_CONTROL_PATH("Attempting to use Nvidia-specific code path on non-Nvidia hardware.");
#endif
}
```

**EN:** The preceding comment documents this block. The function `ret` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction.

**CN:** 前面的注释说明了这个代码块。`ret` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。

### Lines 74-74

```cpp
#endif
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#endif`.

**CN:** 这个预处理代码块围绕 `#endif` 选择编译路径或功能开关。

### Lines 76-77

```cpp
////////////////////////////////////////////////////////////////////////////////////////////////////
struct Sm50 {
```

**EN:** The preceding comment documents this block. This block begins the definition of `Sm50`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `Sm50` 这个 `struct`，其成员会在后续代码中展开。

### Lines 78-78

```cpp
  static int const kMinComputeCapability = 50;
```

**EN:** This declaration defines `kMinComputeCapability` and assigns it the compile-time expression `50`.

**CN:** 这个声明定义了 `kMinComputeCapability`，并把它设为编译期表达式 `50`。

### Lines 80-80

```cpp
struct Sm60 {
```

**EN:** This block begins the definition of `Sm60`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `Sm60` 这个 `struct`，其成员会在后续代码中展开。

### Lines 81-81

```cpp
  static int const kMinComputeCapability = 60;
```

**EN:** This declaration defines `kMinComputeCapability` and assigns it the compile-time expression `60`.

**CN:** 这个声明定义了 `kMinComputeCapability`，并把它设为编译期表达式 `60`。

### Lines 83-83

```cpp
struct Sm61 {
```

**EN:** This block begins the definition of `Sm61`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `Sm61` 这个 `struct`，其成员会在后续代码中展开。

### Lines 84-84

```cpp
  static int const kMinComputeCapability = 61;
```

**EN:** This declaration defines `kMinComputeCapability` and assigns it the compile-time expression `61`.

**CN:** 这个声明定义了 `kMinComputeCapability`，并把它设为编译期表达式 `61`。

### Lines 86-86

```cpp
struct Sm70 {
```

**EN:** This block begins the definition of `Sm70`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `Sm70` 这个 `struct`，其成员会在后续代码中展开。

### Lines 87-87

```cpp
  static int const kMinComputeCapability = 70;
```

**EN:** This declaration defines `kMinComputeCapability` and assigns it the compile-time expression `70`.

**CN:** 这个声明定义了 `kMinComputeCapability`，并把它设为编译期表达式 `70`。

### Lines 89-89

```cpp
struct Sm72 {
```

**EN:** This block begins the definition of `Sm72`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `Sm72` 这个 `struct`，其成员会在后续代码中展开。

### Lines 90-90

```cpp
  static int const kMinComputeCapability = 72;
```

**EN:** This declaration defines `kMinComputeCapability` and assigns it the compile-time expression `72`.

**CN:** 这个声明定义了 `kMinComputeCapability`，并把它设为编译期表达式 `72`。

### Lines 92-92

```cpp
struct Sm75 {
```

**EN:** This block begins the definition of `Sm75`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `Sm75` 这个 `struct`，其成员会在后续代码中展开。

### Lines 93-93

```cpp
  static int const kMinComputeCapability = 75;
```

**EN:** This declaration defines `kMinComputeCapability` and assigns it the compile-time expression `75`.

**CN:** 这个声明定义了 `kMinComputeCapability`，并把它设为编译期表达式 `75`。

### Lines 95-95

```cpp
struct Sm80 {
```

**EN:** This block begins the definition of `Sm80`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `Sm80` 这个 `struct`，其成员会在后续代码中展开。

### Lines 96-96

```cpp
  static int const kMinComputeCapability = 80; 
```

**EN:** This declaration defines `kMinComputeCapability` and assigns it the compile-time expression `80`.

**CN:** 这个声明定义了 `kMinComputeCapability`，并把它设为编译期表达式 `80`。

### Lines 98-98

```cpp
struct Sm86 {
```

**EN:** This block begins the definition of `Sm86`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `Sm86` 这个 `struct`，其成员会在后续代码中展开。

### Lines 99-99

```cpp
  static int const kMinComputeCapability = 86;
```

**EN:** This declaration defines `kMinComputeCapability` and assigns it the compile-time expression `86`.

**CN:** 这个声明定义了 `kMinComputeCapability`，并把它设为编译期表达式 `86`。

### Lines 101-101

```cpp
struct Sm89 {
```

**EN:** This block begins the definition of `Sm89`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `Sm89` 这个 `struct`，其成员会在后续代码中展开。

### Lines 102-102

```cpp
  static int const kMinComputeCapability = 89;
```

**EN:** This declaration defines `kMinComputeCapability` and assigns it the compile-time expression `89`.

**CN:** 这个声明定义了 `kMinComputeCapability`，并把它设为编译期表达式 `89`。

### Lines 104-104

```cpp
struct Sm90 {
```

**EN:** This block begins the definition of `Sm90`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `Sm90` 这个 `struct`，其成员会在后续代码中展开。

### Lines 105-105

```cpp
  static int const kMinComputeCapability = 90; 
```

**EN:** This declaration defines `kMinComputeCapability` and assigns it the compile-time expression `90`.

**CN:** 这个声明定义了 `kMinComputeCapability`，并把它设为编译期表达式 `90`。

### Lines 109-109

```cpp
struct Sm100 {
```

**EN:** This block begins the definition of `Sm100`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `Sm100` 这个 `struct`，其成员会在后续代码中展开。

### Lines 110-110

```cpp
  static int const kMinComputeCapability = 100; 
```

**EN:** This declaration defines `kMinComputeCapability` and assigns it the compile-time expression `100`.

**CN:** 这个声明定义了 `kMinComputeCapability`，并把它设为编译期表达式 `100`。

### Lines 113-113

```cpp
struct Sm101 {
```

**EN:** This block begins the definition of `Sm101`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `Sm101` 这个 `struct`，其成员会在后续代码中展开。

### Lines 114-114

```cpp
  static int const kMinComputeCapability = 101;
```

**EN:** This declaration defines `kMinComputeCapability` and assigns it the compile-time expression `101`.

**CN:** 这个声明定义了 `kMinComputeCapability`，并把它设为编译期表达式 `101`。

### Lines 117-117

```cpp
struct Sm120 {
```

**EN:** This block begins the definition of `Sm120`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `Sm120` 这个 `struct`，其成员会在后续代码中展开。

### Lines 118-118

```cpp
  static int const kMinComputeCapability = 120;
```

**EN:** This declaration defines `kMinComputeCapability` and assigns it the compile-time expression `120`.

**CN:** 这个声明定义了 `kMinComputeCapability`，并把它设为编译期表达式 `120`。

### Lines 121-121

```cpp
#if defined(CUTLASS_ENABLE_SYCL)
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#if defined(CUTLASS_ENABLE_SYCL)`.

**CN:** 这个预处理代码块围绕 `#if defined(CUTLASS_ENABLE_SYCL)` 选择编译路径或功能开关。

### Lines 122-122

```cpp
struct IntelXe {
```

**EN:** This block begins the definition of `IntelXe`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `IntelXe` 这个 `struct`，其成员会在后续代码中展开。

### Lines 123-123

```cpp
  static int const kMinComputeCapability = 0;
```

**EN:** This declaration defines `kMinComputeCapability` and assigns it the compile-time expression `0`.

**CN:** 这个声明定义了 `kMinComputeCapability`，并把它设为编译期表达式 `0`。

### Lines 126-128

```cpp
// Intel Xe architecture aliases for library generation compatibility
// Xe12 = PVC (Ponte Vecchio)
struct Xe12 : IntelXe {
```

**EN:** The preceding comment documents this block. This block begins the definition of `Xe12`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `Xe12` 这个 `struct`，其成员会在后续代码中展开。

### Lines 129-129

```cpp
  static int const kIntelXeArch = 12;
```

**EN:** This declaration defines `kIntelXeArch` and assigns it the compile-time expression `12`.

**CN:** 这个声明定义了 `kIntelXeArch`，并把它设为编译期表达式 `12`。

### Lines 132-133

```cpp
// Xe20 = BMG (Battlemage) 
struct Xe20 : IntelXe {
```

**EN:** The preceding comment documents this block. This block begins the definition of `Xe20`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `Xe20` 这个 `struct`，其成员会在后续代码中展开。

### Lines 134-134

```cpp
  static int const kIntelXeArch = 20;
```

**EN:** This declaration defines `kIntelXeArch` and assigns it the compile-time expression `20`.

**CN:** 这个声明定义了 `kIntelXeArch`，并把它设为编译期表达式 `20`。

### Lines 137-137

```cpp
struct Agnostic {
```

**EN:** This block begins the definition of `Agnostic`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `Agnostic` 这个 `struct`，其成员会在后续代码中展开。

### Lines 138-138

```cpp
  static int const kMinComputeCapability = 1;
```

**EN:** This declaration defines `kMinComputeCapability` and assigns it the compile-time expression `1`.

**CN:** 这个声明定义了 `kMinComputeCapability`，并把它设为编译期表达式 `1`。

### Lines 141-141

```cpp
#endif
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#endif`.

**CN:** 这个预处理代码块围绕 `#endif` 选择编译路径或功能开关。

### Lines 143-143

```cpp
struct Sm103 {
```

**EN:** This block begins the definition of `Sm103`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `Sm103` 这个 `struct`，其成员会在后续代码中展开。

### Lines 144-144

```cpp
  static int const kMinComputeCapability = 103; 
```

**EN:** This declaration defines `kMinComputeCapability` and assigns it the compile-time expression `103`.

**CN:** 这个声明定义了 `kMinComputeCapability`，并把它设为编译期表达式 `103`。

### Lines 147-153

```cpp
/// Triggers a breakpoint on the device
CUTLASS_DEVICE
void device_breakpoint() {
#if defined(__CUDA_ARCH__) || defined(__SYCL_CUDA_ARCH__)
  asm volatile ("  brkpt;\n");
#endif
}
```

**EN:** The preceding comment documents this block. The function `device_breakpoint` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction.

**CN:** 前面的注释说明了这个代码块。`device_breakpoint` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。

## Key Concepts / 关键概念

- **EN:** Architecture-specific paths map C++ wrappers onto GPU instructions or micro-architectural features.
  **CN:** 架构特定路径会把 C++ 封装映射到 GPU 指令或底层微架构特性。

- **EN:** Conditional compilation keeps the header portable across host, device, CUDA, and SYCL builds.
  **CN:** 条件编译使该头文件能够在主机端、设备端、CUDA 与 SYCL 构建之间保持可移植性。

## Dependencies / 依赖关系

- **EN:** Direct includes: `cutlass/cutlass.h`, `cute/config.hpp`.
  **CN:** 直接包含：`cutlass/cutlass.h`, `cute/config.hpp`。

- **EN:** Primary namespaces: `cutlass`, `arch`.
  **CN:** 主要命名空间：`cutlass`, `arch`。

- **EN:** Important macros or compile flags: `CUTLASS_DEVICE`, `CUTLASS_ENABLE_SYCL`.
  **CN:** 重要宏或编译开关：`CUTLASS_DEVICE`, `CUTLASS_ENABLE_SYCL`。
