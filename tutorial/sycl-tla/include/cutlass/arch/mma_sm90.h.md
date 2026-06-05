# mma_sm90.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/arch/mma_sm90.h`

- **EN:** Matrix multiply

- **CN:** 该头文件主要实现 CUTLASS 的架构相关原语与指令封装。文件级摘要：Matrix multiply

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
    \brief Matrix multiply
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

### Lines 36-36

```cpp
#include "cutlass/cutlass.h"
```

**EN:** This block imports dependencies such as `cutlass/cutlass.h`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cutlass/cutlass.h` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 37-37

```cpp
#if !defined(CUTLASS_ENABLE_SYCL)
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#if !defined(CUTLASS_ENABLE_SYCL)`.

**CN:** 这个预处理代码块围绕 `#if !defined(CUTLASS_ENABLE_SYCL)` 选择编译路径或功能开关。

### Lines 38-38

```cpp
#include CUDA_STD_HEADER(cassert)
```

**EN:** This block imports dependencies such as `CUDA_STD_HEADER(cassert)`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `CUDA_STD_HEADER(cassert)` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 39-39

```cpp
#endif
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#endif`.

**CN:** 这个预处理代码块围绕 `#endif` 选择编译路径或功能开关。

### Lines 41-44

```cpp
#include "mma.h"
#include "cutlass/layout/matrix.h"
#include "cutlass/numeric_types.h"
#include "cutlass/arch/config.h"
```

**EN:** This block imports dependencies such as `mma.h`, `cutlass/layout/matrix.h`, `cutlass/numeric_types.h`, `cutlass/arch/config.h`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `mma.h`, `cutlass/layout/matrix.h`, `cutlass/numeric_types.h`, `cutlass/arch/config.h` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 46-47

```cpp
////////////////////////////////////////////////////////////////////////////////
namespace cutlass {
```

**EN:** The preceding comment documents this block. This block opens the namespace scope `cutlass` for the declarations that follow.

**CN:** 前面的注释说明了这个代码块。该代码块打开了 `cutlass` 命名空间作用域，以容纳后续声明。

### Lines 49-49

```cpp
namespace arch {
```

**EN:** This block opens the namespace scope `arch` for the declarations that follow.

**CN:** 该代码块打开了 `arch` 命名空间作用域，以容纳后续声明。

### Lines 51-53

```cpp
////////////////////////////////////////////////////////////////////////////////
/// Matrix Multiply-Add 16x8x4 fp64
////////////////////////////////////////////////////////////////////////////////
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 55-66

```cpp
/// Matrix multiply-add operation: F64 = F64 * F64 + F64
template <>
struct Mma<
  gemm::GemmShape<16,8,4>,
  32,
  double,
  layout::RowMajor,
  double,
  layout::ColumnMajor,
  double,
  layout::RowMajor,
  OpMultiplyAdd> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `Mma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `Mma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 68-68

```cpp
  using Shape = gemm::GemmShape<16,8,4>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<16,8,4>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<16,8,4>` 的别名，以简化后续模板或成员声明。

### Lines 70-70

```cpp
  using ElementA = double;
```

**EN:** This alias defines `ElementA` as `double`, shortening later template or member declarations.

**CN:** 这里把 `ElementA` 定义为 `double` 的别名，以简化后续模板或成员声明。

### Lines 71-71

```cpp
  using LayoutA = layout::RowMajor;
```

**EN:** This alias defines `LayoutA` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutA` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 72-72

```cpp
  using FragmentA = Array<double, 2>;
```

**EN:** This alias defines `FragmentA` as `Array<double, 2>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentA` 定义为 `Array<double, 2>` 的别名，以简化后续模板或成员声明。

### Lines 74-74

```cpp
  using ElementB = double;
```

**EN:** This alias defines `ElementB` as `double`, shortening later template or member declarations.

**CN:** 这里把 `ElementB` 定义为 `double` 的别名，以简化后续模板或成员声明。

### Lines 75-75

```cpp
  using LayoutB = layout::ColumnMajor;
```

**EN:** This alias defines `LayoutB` as `layout::ColumnMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutB` 定义为 `layout::ColumnMajor` 的别名，以简化后续模板或成员声明。

### Lines 76-76

```cpp
  using FragmentB = Array<double, 1>;
```

**EN:** This alias defines `FragmentB` as `Array<double, 1>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentB` 定义为 `Array<double, 1>` 的别名，以简化后续模板或成员声明。

### Lines 78-78

```cpp
  using ElementC = double;
```

**EN:** This alias defines `ElementC` as `double`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `double` 的别名，以简化后续模板或成员声明。

### Lines 79-79

```cpp
  using LayoutC = layout::RowMajor;
```

**EN:** This alias defines `LayoutC` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutC` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 80-80

```cpp
  using FragmentC = Array<double, 4>;
```

**EN:** This alias defines `FragmentC` as `Array<double, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentC` 定义为 `Array<double, 4>` 的别名，以简化后续模板或成员声明。

### Lines 82-82

```cpp
  using Operator = OpMultiplyAdd;
```

**EN:** This alias defines `Operator` as `OpMultiplyAdd`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAdd` 的别名，以简化后续模板或成员声明。

### Lines 84-84

```cpp
  using ArchTag = arch::Sm90;
```

**EN:** This alias defines `ArchTag` as `arch::Sm90`, shortening later template or member declarations.

**CN:** 这里把 `ArchTag` 定义为 `arch::Sm90` 的别名，以简化后续模板或成员声明。

### Lines 86-111

```cpp
  CUTLASS_HOST_DEVICE
  void operator()(FragmentC &d, FragmentA const &a, FragmentB const &b,
                  FragmentC const &c) const {

#if defined(CUTLASS_ARCH_MMA_SM90_F64_MMA_ENABLED)

  double const *A = reinterpret_cast<double const *>(&a);
  double const *B = reinterpret_cast<double const *>(&b);

  double const *C = reinterpret_cast<double const *>(&c);
  double *D = reinterpret_cast<double *>(&d);

  asm volatile("mma.sync.aligned.m16n8k4.row.col.f64.f64.f64.f64.rn {%0, %1, %2, %3}, {%4, %5}, {%6}, {%7, %8, %9, %10};\n"
      : "=d"(D[0]), "=d"(D[1]), "=d"(D[2]), "=d"(D[3])
      : "d"(A[0]), "d"(A[1]),
        "d"(B[0]),
        "d"(C[0]), "d"(C[1]), "d"(C[2]), "d"(C[3]));

#else
    CUTLASS_UNUSED(d);
    CUTLASS_UNUSED(a);
    CUTLASS_UNUSED(b);
    CUTLASS_UNUSED(c);
    CUTLASS_NOT_IMPLEMENTED();
#endif
  }
```

**EN:** The function `A` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view. An unsupported target path falls back to a not-implemented marker.

**CN:** `A` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。对于不受支持的目标，它会回退到未实现标记。

### Lines 114-116

```cpp
////////////////////////////////////////////////////////////////////////////////
/// Matrix Multiply-Add 16x8x8 fp64
////////////////////////////////////////////////////////////////////////////////
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 118-129

```cpp
/// Matrix multiply-add operation: F64 = F64 * F64 + F64
template <>
struct Mma<
  gemm::GemmShape<16,8,8>,
  32,
  double,
  layout::RowMajor,
  double,
  layout::ColumnMajor,
  double,
  layout::RowMajor,
  OpMultiplyAdd> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `Mma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `Mma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 131-131

```cpp
  using Shape = gemm::GemmShape<16,8,8>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<16,8,8>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<16,8,8>` 的别名，以简化后续模板或成员声明。

### Lines 133-133

```cpp
  using ElementA = double;
```

**EN:** This alias defines `ElementA` as `double`, shortening later template or member declarations.

**CN:** 这里把 `ElementA` 定义为 `double` 的别名，以简化后续模板或成员声明。

### Lines 134-134

```cpp
  using LayoutA = layout::RowMajor;
```

**EN:** This alias defines `LayoutA` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutA` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 135-135

```cpp
  using FragmentA = Array<double, 4>;
```

**EN:** This alias defines `FragmentA` as `Array<double, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentA` 定义为 `Array<double, 4>` 的别名，以简化后续模板或成员声明。

### Lines 137-137

```cpp
  using ElementB = double;
```

**EN:** This alias defines `ElementB` as `double`, shortening later template or member declarations.

**CN:** 这里把 `ElementB` 定义为 `double` 的别名，以简化后续模板或成员声明。

### Lines 138-138

```cpp
  using LayoutB = layout::ColumnMajor;
```

**EN:** This alias defines `LayoutB` as `layout::ColumnMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutB` 定义为 `layout::ColumnMajor` 的别名，以简化后续模板或成员声明。

### Lines 139-139

```cpp
  using FragmentB = Array<double, 2>;
```

**EN:** This alias defines `FragmentB` as `Array<double, 2>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentB` 定义为 `Array<double, 2>` 的别名，以简化后续模板或成员声明。

### Lines 141-141

```cpp
  using ElementC = double;
```

**EN:** This alias defines `ElementC` as `double`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `double` 的别名，以简化后续模板或成员声明。

### Lines 142-142

```cpp
  using LayoutC = layout::RowMajor;
```

**EN:** This alias defines `LayoutC` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutC` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 143-143

```cpp
  using FragmentC = Array<double, 4>;
```

**EN:** This alias defines `FragmentC` as `Array<double, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentC` 定义为 `Array<double, 4>` 的别名，以简化后续模板或成员声明。

### Lines 145-145

```cpp
  using Operator = OpMultiplyAdd;
```

**EN:** This alias defines `Operator` as `OpMultiplyAdd`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAdd` 的别名，以简化后续模板或成员声明。

### Lines 147-147

```cpp
  using ArchTag = arch::Sm90;
```

**EN:** This alias defines `ArchTag` as `arch::Sm90`, shortening later template or member declarations.

**CN:** 这里把 `ArchTag` 定义为 `arch::Sm90` 的别名，以简化后续模板或成员声明。

### Lines 149-175

```cpp
  CUTLASS_HOST_DEVICE
  void operator()(FragmentC &d, FragmentA const &a, FragmentB const &b,
                  FragmentC const &c) const {

#if defined(CUTLASS_ARCH_MMA_SM90_F64_MMA_ENABLED)

  double const *A = reinterpret_cast<double const *>(&a);
  double const *B = reinterpret_cast<double const *>(&b);

  double const *C = reinterpret_cast<double const *>(&c);
  double *D = reinterpret_cast<double *>(&d);

  asm volatile("mma.sync.aligned.m16n8k8.row.col.f64.f64.f64.f64 {%0, %1, %2, %3}, {%4, %5, %6, %7}, {%8, %9}, {%10, %11, %12, %13};\n"
      : "=d"(D[0]), "=d"(d[1]), "=d"(d[2]), "=d"(d[3])
      : "d"(A[0]), "d"(A[1]), "d"(A[2]), "d"(A[3]),
        "d"(B[0]), "d"(B[1]),
        "d"(C[0]), "d"(C[1]), "d"(C[2]), "d"(C[3]));

#else

    CUTLASS_UNUSED(d);
    CUTLASS_UNUSED(a);
    CUTLASS_UNUSED(b);
    CUTLASS_UNUSED(c);
    CUTLASS_NOT_IMPLEMENTED();
#endif
  }
```

**EN:** The function `A` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view. An unsupported target path falls back to a not-implemented marker.

**CN:** `A` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。对于不受支持的目标，它会回退到未实现标记。

### Lines 178-180

```cpp
////////////////////////////////////////////////////////////////////////////////
/// Matrix Multiply-Add 16x8x16 fp64
////////////////////////////////////////////////////////////////////////////////
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 182-193

```cpp
/// Matrix multiply-add operation: F64 = F64 * F64 + F64
template <>
struct Mma<
  gemm::GemmShape<16,8,16>,
  32,
  double,
  layout::RowMajor,
  double,
  layout::ColumnMajor,
  double,
  layout::RowMajor,
  OpMultiplyAdd> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `Mma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `Mma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 195-195

```cpp
  using Shape = gemm::GemmShape<16,8,16>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<16,8,16>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<16,8,16>` 的别名，以简化后续模板或成员声明。

### Lines 197-197

```cpp
  using ElementA = double;
```

**EN:** This alias defines `ElementA` as `double`, shortening later template or member declarations.

**CN:** 这里把 `ElementA` 定义为 `double` 的别名，以简化后续模板或成员声明。

### Lines 198-198

```cpp
  using LayoutA = layout::RowMajor;
```

**EN:** This alias defines `LayoutA` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutA` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 199-199

```cpp
  using FragmentA = Array<double, 8>;
```

**EN:** This alias defines `FragmentA` as `Array<double, 8>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentA` 定义为 `Array<double, 8>` 的别名，以简化后续模板或成员声明。

### Lines 201-201

```cpp
  using ElementB = double;
```

**EN:** This alias defines `ElementB` as `double`, shortening later template or member declarations.

**CN:** 这里把 `ElementB` 定义为 `double` 的别名，以简化后续模板或成员声明。

### Lines 202-202

```cpp
  using LayoutB = layout::ColumnMajor;
```

**EN:** This alias defines `LayoutB` as `layout::ColumnMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutB` 定义为 `layout::ColumnMajor` 的别名，以简化后续模板或成员声明。

### Lines 203-203

```cpp
  using FragmentB = Array<double, 4>;
```

**EN:** This alias defines `FragmentB` as `Array<double, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentB` 定义为 `Array<double, 4>` 的别名，以简化后续模板或成员声明。

### Lines 205-205

```cpp
  using ElementC = double;
```

**EN:** This alias defines `ElementC` as `double`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `double` 的别名，以简化后续模板或成员声明。

### Lines 206-206

```cpp
  using LayoutC = layout::RowMajor;
```

**EN:** This alias defines `LayoutC` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutC` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 207-207

```cpp
  using FragmentC = Array<double, 4>;
```

**EN:** This alias defines `FragmentC` as `Array<double, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentC` 定义为 `Array<double, 4>` 的别名，以简化后续模板或成员声明。

### Lines 209-209

```cpp
  using Operator = OpMultiplyAdd;
```

**EN:** This alias defines `Operator` as `OpMultiplyAdd`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAdd` 的别名，以简化后续模板或成员声明。

### Lines 211-211

```cpp
  using ArchTag = arch::Sm90;
```

**EN:** This alias defines `ArchTag` as `arch::Sm90`, shortening later template or member declarations.

**CN:** 这里把 `ArchTag` 定义为 `arch::Sm90` 的别名，以简化后续模板或成员声明。

### Lines 213-234

```cpp
  CUTLASS_HOST_DEVICE
  void operator()(FragmentC &d, FragmentA const &a, FragmentB const &b,
                  FragmentC const &c) const {
    
#if defined(CUTLASS_ARCH_MMA_SM90_F64_MMA_ENABLED)

  double const *A = reinterpret_cast<double const *>(&a);
  double const *B = reinterpret_cast<double const *>(&b);

  double const *C = reinterpret_cast<double const *>(&c);
  double *D = reinterpret_cast<double *>(&d);

  asm volatile("mma.sync.aligned.m16n8k16.row.col.f64.f64.f64.f64 {%0, %1, %2, %3}, {%4, %5, %6, %7, %8, %9, %10, %11}, {%12, %13, %14, %15}, {%16, %17, %18, %19};\n"
      : "=d"(D[0]), "=d"(D[1]), "=d"(D[2]), "=d"(D[3])
      : "d"(A[0]), "d"(A[2]), "d"(A[2]), "d"(A[3]), "d"(A[4]), "d"(A[5]), "d"(A[6]), "d"(A[7]),
        "d"(B[0]), "d"(B[1]), "d"(B[2]), "d"(B[3]), 
        "d"(C[0]), "d"(C[1]), "d"(C[2]), "d"(C[3]));

#else
    CUTLASS_NOT_IMPLEMENTED();
#endif
  }
```

**EN:** The function `A` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view. An unsupported target path falls back to a not-implemented marker.

**CN:** `A` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。对于不受支持的目标，它会回退到未实现标记。

## Key Concepts / 关键概念

- **EN:** Heavy use of templates enables compile-time specialization and zero-overhead abstractions.
  **CN:** 大量使用模板，使该文件能够进行编译期特化并保持零额外开销的抽象。

- **EN:** Architecture-specific paths map C++ wrappers onto GPU instructions or micro-architectural features.
  **CN:** 架构特定路径会把 C++ 封装映射到 GPU 指令或底层微架构特性。

- **EN:** Fixed-size containers and fragments are used to model register or shared-memory tiles.
  **CN:** 定长容器与片段类型用于建模寄存器块或共享内存块。

- **EN:** Conditional compilation keeps the header portable across host, device, CUDA, and SYCL builds.
  **CN:** 条件编译使该头文件能够在主机端、设备端、CUDA 与 SYCL 构建之间保持可移植性。

## Dependencies / 依赖关系

- **EN:** Direct includes: `cutlass/cutlass.h`, `CUDA_STD_HEADER(cassert)`, `mma.h`, `cutlass/layout/matrix.h`, `cutlass/numeric_types.h`, `cutlass/arch/config.h`.
  **CN:** 直接包含：`cutlass/cutlass.h`, `CUDA_STD_HEADER(cassert)`, `mma.h`, `cutlass/layout/matrix.h`, `cutlass/numeric_types.h`, `cutlass/arch/config.h`。

- **EN:** Primary namespaces: `cutlass`, `arch`.
  **CN:** 主要命名空间：`cutlass`, `arch`。

- **EN:** Important macros or compile flags: `CUDA_STD_HEADER`, `CUTLASS_ARCH_MMA_SM90_F64_MMA_ENABLED`, `CUTLASS_ENABLE_SYCL`, `CUTLASS_HOST_DEVICE`, `CUTLASS_NOT_IMPLEMENTED`, `CUTLASS_UNUSED`.
  **CN:** 重要宏或编译开关：`CUDA_STD_HEADER`, `CUTLASS_ARCH_MMA_SM90_F64_MMA_ENABLED`, `CUTLASS_ENABLE_SYCL`, `CUTLASS_HOST_DEVICE`, `CUTLASS_NOT_IMPLEMENTED`, `CUTLASS_UNUSED`。
