# mma_sparse_sm80.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/arch/mma_sparse_sm80.h`

- **EN:** Sparse matrix multiply accumulate for SM80

- **CN:** 该头文件主要实现 CUTLASS 的架构相关原语与指令封装。文件级摘要：Sparse matrix multiply accumulate for SM80

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

### Lines 32-34

```cpp
/*! \file
    \brief Sparse matrix multiply accumulate for SM80
*/
```

**EN:** This file-level comment explains the purpose of the header and introduces the abstractions defined below.

**CN:** 这个文件级注释说明了头文件的用途，并引出了后续定义的抽象。

### Lines 36-36

```cpp
#pragma once
```

**EN:** This directive uses `#pragma once` to avoid repeated inclusion of the same header.

**CN:** 这里使用 `#pragma once` 来避免同一头文件被重复包含。

### Lines 37-37

```cpp
#include "cutlass/cutlass.h"
```

**EN:** This block imports dependencies such as `cutlass/cutlass.h`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cutlass/cutlass.h` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 38-38

```cpp
#if !defined(CUTLASS_ENABLE_SYCL)
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#if !defined(CUTLASS_ENABLE_SYCL)`.

**CN:** 这个预处理代码块围绕 `#if !defined(CUTLASS_ENABLE_SYCL)` 选择编译路径或功能开关。

### Lines 39-39

```cpp
#include CUDA_STD_HEADER(cassert)
```

**EN:** This block imports dependencies such as `CUDA_STD_HEADER(cassert)`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `CUDA_STD_HEADER(cassert)` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 40-40

```cpp
#endif
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#endif`.

**CN:** 这个预处理代码块围绕 `#endif` 选择编译路径或功能开关。

### Lines 42-44

```cpp
#include "mma.h"
#include "cutlass/layout/matrix.h"
#include "cutlass/numeric_types.h"
```

**EN:** This block imports dependencies such as `mma.h`, `cutlass/layout/matrix.h`, `cutlass/numeric_types.h`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `mma.h`, `cutlass/layout/matrix.h`, `cutlass/numeric_types.h` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 48-48

```cpp
#if ((__CUDACC_VER_MAJOR__ > 11) || (__CUDACC_VER_MAJOR__ == 11 && __CUDACC_VER_MINOR__ >= 1))
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#if ((__CUDACC_VER_MAJOR__ > 11) || (__CUDACC_VER_MAJOR__ == 11 && __CUDACC_VER_MINOR__ >= 1))`.

**CN:** 这个预处理代码块围绕 `#if ((__CUDACC_VER_MAJOR__ > 11) || (__CUDACC_VER_MAJOR__ == 11 && __CUDACC_VER_MINOR__ >= 1))` 选择编译路径或功能开关。

### Lines 50-50

```cpp
#define CUTLASS_ARCH_SPARSE_MMA_SM80_SUPPORTED 1
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#define CUTLASS_ARCH_SPARSE_MMA_SM80_SUPPORTED 1`.

**CN:** 这个预处理代码块围绕 `#define CUTLASS_ARCH_SPARSE_MMA_SM80_SUPPORTED 1` 选择编译路径或功能开关。

### Lines 52-52

```cpp
#if (defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 800))
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#if (defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 800))`.

**CN:** 这个预处理代码块围绕 `#if (defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 800))` 选择编译路径或功能开关。

### Lines 53-53

```cpp
#define CUTLASS_ARCH_SPARSE_MMA_SM80_ENABLED
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#define CUTLASS_ARCH_SPARSE_MMA_SM80_ENABLED`.

**CN:** 这个预处理代码块围绕 `#define CUTLASS_ARCH_SPARSE_MMA_SM80_ENABLED` 选择编译路径或功能开关。

### Lines 54-54

```cpp
#endif
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#endif`.

**CN:** 这个预处理代码块围绕 `#endif` 选择编译路径或功能开关。

### Lines 56-56

```cpp
#endif
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#endif`.

**CN:** 这个预处理代码块围绕 `#endif` 选择编译路径或功能开关。

### Lines 58-59

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
namespace cutlass {
```

**EN:** The preceding comment documents this block. This block opens the namespace scope `cutlass` for the declarations that follow.

**CN:** 前面的注释说明了这个代码块。该代码块打开了 `cutlass` 命名空间作用域，以容纳后续声明。

### Lines 61-61

```cpp
namespace arch {
```

**EN:** This block opens the namespace scope `arch` for the declarations that follow.

**CN:** 该代码块打开了 `arch` 命名空间作用域，以容纳后续声明。

### Lines 65-69

```cpp
////////////////////////////////////////////////////////////////////////////////
//
// Sparse Matrix Multiply 16832
//
////////////////////////////////////////////////////////////////////////////////
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 71-84

```cpp
/// Matrix multiply-add operation: F16 = F16 * F16 + F16
template <>
struct SparseMma<
  gemm::GemmShape<16, 8, 32>,
  32,
  half_t,
  layout::RowMajor,
  half_t,
  layout::ColumnMajor,
  half_t,
  layout::RowMajor,
  OpMultiplyAdd,
  SPFormatType::Thread
> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `SparseMma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `SparseMma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 86-86

```cpp
  using Shape = gemm::GemmShape<16, 8, 32>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<16, 8, 32>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<16, 8, 32>` 的别名，以简化后续模板或成员声明。

### Lines 88-88

```cpp
  using ElementA = half_t;
```

**EN:** This alias defines `ElementA` as `half_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementA` 定义为 `half_t` 的别名，以简化后续模板或成员声明。

### Lines 89-89

```cpp
  using LayoutA = layout::RowMajor;
```

**EN:** This alias defines `LayoutA` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutA` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 90-90

```cpp
  using FragmentA = Array<half_t, 8>;
```

**EN:** This alias defines `FragmentA` as `Array<half_t, 8>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentA` 定义为 `Array<half_t, 8>` 的别名，以简化后续模板或成员声明。

### Lines 92-92

```cpp
  using ElementB = half_t;
```

**EN:** This alias defines `ElementB` as `half_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementB` 定义为 `half_t` 的别名，以简化后续模板或成员声明。

### Lines 93-93

```cpp
  using LayoutB = layout::ColumnMajor;
```

**EN:** This alias defines `LayoutB` as `layout::ColumnMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutB` 定义为 `layout::ColumnMajor` 的别名，以简化后续模板或成员声明。

### Lines 94-94

```cpp
  using FragmentB = Array<half_t, 8>;
```

**EN:** This alias defines `FragmentB` as `Array<half_t, 8>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentB` 定义为 `Array<half_t, 8>` 的别名，以简化后续模板或成员声明。

### Lines 96-96

```cpp
  using ElementC = half_t;
```

**EN:** This alias defines `ElementC` as `half_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `half_t` 的别名，以简化后续模板或成员声明。

### Lines 97-97

```cpp
  using LayoutC = layout::RowMajor;
```

**EN:** This alias defines `LayoutC` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutC` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 98-98

```cpp
  using FragmentC = Array<half_t, 4>;
```

**EN:** This alias defines `FragmentC` as `Array<half_t, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentC` 定义为 `Array<half_t, 4>` 的别名，以简化后续模板或成员声明。

### Lines 100-100

```cpp
  using FragmentE = uint32_t;
```

**EN:** This alias defines `FragmentE` as `uint32_t`, shortening later template or member declarations.

**CN:** 这里把 `FragmentE` 定义为 `uint32_t` 的别名，以简化后续模板或成员声明。

### Lines 102-102

```cpp
  using Operator = OpMultiplyAdd;
```

**EN:** This alias defines `Operator` as `OpMultiplyAdd`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAdd` 的别名，以简化后续模板或成员声明。

### Lines 103-103

```cpp
  using ArchTag = arch::Sm80;
```

**EN:** This alias defines `ArchTag` as `arch::Sm80`, shortening later template or member declarations.

**CN:** 这里把 `ArchTag` 定义为 `arch::Sm80` 的别名，以简化后续模板或成员声明。

### Lines 105-105

```cpp
  static int const kSparse = 2;
```

**EN:** This declaration defines `kSparse` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kSparse`，并把它设为编译期表达式 `2`。

### Lines 107-107

```cpp
  static int const kMetaSizeInBits = 2;
```

**EN:** This declaration defines `kMetaSizeInBits` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kMetaSizeInBits`，并把它设为编译期表达式 `2`。

### Lines 109-109

```cpp
  static int const kMaxID2 = 2;
```

**EN:** This declaration defines `kMaxID2` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kMaxID2`，并把它设为编译期表达式 `2`。

### Lines 111-172

```cpp
  /// Computes multiply-add
  CUTLASS_HOST_DEVICE
  void operator()(FragmentC &d, FragmentA const &a, FragmentB const &b,
                  FragmentC const &c, uint32_t const &E, int const id2) const {

#if defined(CUTLASS_ARCH_SPARSE_MMA_SM80_ENABLED)

  uint32_t const *A = reinterpret_cast<uint32_t const *>(&a);
  uint32_t const *B = reinterpret_cast<uint32_t const *>(&b);
  uint32_t const *C = reinterpret_cast<uint32_t const *>(&c);
  uint32_t *D = reinterpret_cast<uint32_t *>(&d);

#if ((__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 5))
  if (id2 == 0) {
    asm volatile(
        "mma.sp::ordered_metadata.sync.aligned.m16n8k32.row.col.f16.f16.f16.f16 {%0,%1}, "
        "{%2,%3,%4,%5}, {%6,%7,%8,%9}, {%10,%11}, %12, 0x0;\n"
        : "=r"(D[0]), "=r"(D[1])
        : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]),
          "r"(B[2]), "r"(B[3]), "r"(C[0]), "r"(C[1]), "r"(E));
  }
  else if (id2 == 1) {
    asm volatile(
        "mma.sp::ordered_metadata.sync.aligned.m16n8k32.row.col.f16.f16.f16.f16 {%0,%1}, "
        "{%2,%3,%4,%5}, {%6,%7,%8,%9}, {%10,%11}, %12, 0x1;\n"
        : "=r"(D[0]), "=r"(D[1])
        : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]),
          "r"(B[2]), "r"(B[3]), "r"(C[0]), "r"(C[1]), "r"(E));
  }
  else {
    assert(0);
  }
#else
  if (id2 == 0) {
    asm volatile(
        "mma.sp.sync.aligned.m16n8k32.row.col.f16.f16.f16.f16 {%0,%1}, "
        "{%2,%3,%4,%5}, {%6,%7,%8,%9}, {%10,%11}, %12, 0x0;\n"
        : "=r"(D[0]), "=r"(D[1])
        : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]),
          "r"(B[2]), "r"(B[3]), "r"(C[0]), "r"(C[1]), "r"(E));
  }
  else if (id2 == 1) {
    asm volatile(
        "mma.sp.sync.aligned.m16n8k32.row.col.f16.f16.f16.f16 {%0,%1}, "
        "{%2,%3,%4,%5}, {%6,%7,%8,%9}, {%10,%11}, %12, 0x1;\n"
        : "=r"(D[0]), "=r"(D[1])
        : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]),
          "r"(B[2]), "r"(B[3]), "r"(C[0]), "r"(C[1]), "r"(E));
  }
  else {
    assert(0);
  }
#endif

#else
    CUTLASS_UNUSED(a);
    CUTLASS_UNUSED(b);
    CUTLASS_UNUSED(c);
    CUTLASS_UNUSED(d);
    assert(0);
#endif
  }
```

**EN:** The preceding comment documents this block. The function `A` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view. Debug/runtime checks guard invalid inputs or extents.

**CN:** 前面的注释说明了这个代码块。`A` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。其中的调试/运行时检查用于防止无效输入或越界范围。

### Lines 177-190

```cpp
/// Matrix multiply-add operation: F32 = F16 * F16 + F32
template <>
struct SparseMma<
  gemm::GemmShape<16, 8, 32>,
  32,
  half_t,
  layout::RowMajor,
  half_t,
  layout::ColumnMajor,
  float,
  layout::RowMajor,
  OpMultiplyAdd,
  SPFormatType::Thread
  > {
```

**EN:** The preceding comment documents this block. This block begins the definition of `SparseMma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `SparseMma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 192-192

```cpp
  using Shape = gemm::GemmShape<16, 8, 32>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<16, 8, 32>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<16, 8, 32>` 的别名，以简化后续模板或成员声明。

### Lines 194-194

```cpp
  using ElementA = half_t;
```

**EN:** This alias defines `ElementA` as `half_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementA` 定义为 `half_t` 的别名，以简化后续模板或成员声明。

### Lines 195-195

```cpp
  using LayoutA = layout::RowMajor;
```

**EN:** This alias defines `LayoutA` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutA` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 196-196

```cpp
  using FragmentA = Array<half_t, 8>;
```

**EN:** This alias defines `FragmentA` as `Array<half_t, 8>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentA` 定义为 `Array<half_t, 8>` 的别名，以简化后续模板或成员声明。

### Lines 198-198

```cpp
  using ElementB = half_t;
```

**EN:** This alias defines `ElementB` as `half_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementB` 定义为 `half_t` 的别名，以简化后续模板或成员声明。

### Lines 199-199

```cpp
  using LayoutB = layout::ColumnMajor;
```

**EN:** This alias defines `LayoutB` as `layout::ColumnMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutB` 定义为 `layout::ColumnMajor` 的别名，以简化后续模板或成员声明。

### Lines 200-200

```cpp
  using FragmentB = Array<half_t, 8>;
```

**EN:** This alias defines `FragmentB` as `Array<half_t, 8>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentB` 定义为 `Array<half_t, 8>` 的别名，以简化后续模板或成员声明。

### Lines 202-202

```cpp
  using ElementC = float;
```

**EN:** This alias defines `ElementC` as `float`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `float` 的别名，以简化后续模板或成员声明。

### Lines 203-203

```cpp
  using LayoutC = layout::RowMajor;
```

**EN:** This alias defines `LayoutC` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutC` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 204-204

```cpp
  using FragmentC = Array<float, 4>;
```

**EN:** This alias defines `FragmentC` as `Array<float, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentC` 定义为 `Array<float, 4>` 的别名，以简化后续模板或成员声明。

### Lines 206-206

```cpp
  using FragmentE = uint32_t;
```

**EN:** This alias defines `FragmentE` as `uint32_t`, shortening later template or member declarations.

**CN:** 这里把 `FragmentE` 定义为 `uint32_t` 的别名，以简化后续模板或成员声明。

### Lines 208-208

```cpp
  using Operator = OpMultiplyAdd;
```

**EN:** This alias defines `Operator` as `OpMultiplyAdd`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAdd` 的别名，以简化后续模板或成员声明。

### Lines 209-209

```cpp
  using ArchTag = arch::Sm80;
```

**EN:** This alias defines `ArchTag` as `arch::Sm80`, shortening later template or member declarations.

**CN:** 这里把 `ArchTag` 定义为 `arch::Sm80` 的别名，以简化后续模板或成员声明。

### Lines 211-211

```cpp
  static int const kSparse = 2;
```

**EN:** This declaration defines `kSparse` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kSparse`，并把它设为编译期表达式 `2`。

### Lines 213-213

```cpp
  static int const kMetaSizeInBits = 2;
```

**EN:** This declaration defines `kMetaSizeInBits` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kMetaSizeInBits`，并把它设为编译期表达式 `2`。

### Lines 215-215

```cpp
  static int const kMaxID2 = 2;
```

**EN:** This declaration defines `kMaxID2` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kMaxID2`，并把它设为编译期表达式 `2`。

### Lines 217-283

```cpp
  /// Computes multiply-add
  CUTLASS_HOST_DEVICE
  void operator()(FragmentC &d, FragmentA const &a, FragmentB const &b,
                  FragmentC const &c, uint32_t const &E, int const id2) const {

#if defined(CUTLASS_ARCH_SPARSE_MMA_SM80_ENABLED)

  uint32_t const *A = reinterpret_cast<uint32_t const *>(&a);
  uint32_t const *B = reinterpret_cast<uint32_t const *>(&b);
  float const *C = reinterpret_cast<float const *>(&c);
  float *D = reinterpret_cast<float *>(&d);

#if ((__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 5))
  if (id2 == 0) {
    asm volatile(
        "mma.sp::ordered_metadata.sync.aligned.m16n8k32.row.col.f32.f16.f16.f32 {%0,%1,%2,%3}, "
        "{%4,%5,%6,%7}, {%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
        : "=f"(D[0]), "=f"(D[1]), "=f"(D[2]), "=f"(D[3])
        : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]),
          "r"(B[2]), "r"(B[3]), "f"(C[0]), "f"(C[1]), "f"(C[2]), "f"(C[3]),
          "r"(E));
  }
  else if (id2 == 1) {
    asm volatile(
        "mma.sp::ordered_metadata.sync.aligned.m16n8k32.row.col.f32.f16.f16.f32 {%0,%1,%2,%3}, "
        "{%4,%5,%6,%7}, {%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x1;\n"
        : "=f"(D[0]), "=f"(D[1]), "=f"(D[2]), "=f"(D[3])
        : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]),
          "r"(B[2]), "r"(B[3]), "f"(C[0]), "f"(C[1]), "f"(C[2]), "f"(C[3]),
          "r"(E));
  }
  else {
    assert(0);
  }
#else
  if (id2 == 0) {
    asm volatile(
        "mma.sp.sync.aligned.m16n8k32.row.col.f32.f16.f16.f32 {%0,%1,%2,%3}, "
        "{%4,%5,%6,%7}, {%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
        : "=f"(D[0]), "=f"(D[1]), "=f"(D[2]), "=f"(D[3])
        : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]),
          "r"(B[2]), "r"(B[3]), "f"(C[0]), "f"(C[1]), "f"(C[2]), "f"(C[3]),
          "r"(E));
  }
  else if (id2 == 1) {
    asm volatile(
        "mma.sp.sync.aligned.m16n8k32.row.col.f32.f16.f16.f32 {%0,%1,%2,%3}, "
        "{%4,%5,%6,%7}, {%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x1;\n"
        : "=f"(D[0]), "=f"(D[1]), "=f"(D[2]), "=f"(D[3])
        : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]),
          "r"(B[2]), "r"(B[3]), "f"(C[0]), "f"(C[1]), "f"(C[2]), "f"(C[3]),
          "r"(E));
  }
  else {
    assert(0);
  }

#endif

#else
    CUTLASS_UNUSED(a);
    CUTLASS_UNUSED(b);
    CUTLASS_UNUSED(c);
    CUTLASS_UNUSED(d);
    assert(0);
#endif
  }
```

**EN:** The preceding comment documents this block. The function `A` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view. Debug/runtime checks guard invalid inputs or extents.

**CN:** 前面的注释说明了这个代码块。`A` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。其中的调试/运行时检查用于防止无效输入或越界范围。

### Lines 286-290

```cpp
////////////////////////////////////////////////////////////////////////////////
//
// Sparse Matrix Multiply 16832 - Float BF16, FP32 accumulation 
//
////////////////////////////////////////////////////////////////////////////////
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 292-296

```cpp
/// Matrix multiply-add operation: F32 = bf16 * bf16 + F32
template <>
struct SparseMma<gemm::GemmShape<16, 8, 32>, 32, bfloat16_t, layout::RowMajor,
           bfloat16_t, layout::ColumnMajor, float, layout::RowMajor,
           OpMultiplyAdd, SPFormatType::Thread> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `SparseMma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `SparseMma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 297-297

```cpp
  using Shape = gemm::GemmShape<16, 8, 32>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<16, 8, 32>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<16, 8, 32>` 的别名，以简化后续模板或成员声明。

### Lines 299-299

```cpp
  using ElementA = bfloat16_t;
```

**EN:** This alias defines `ElementA` as `bfloat16_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementA` 定义为 `bfloat16_t` 的别名，以简化后续模板或成员声明。

### Lines 300-300

```cpp
  using LayoutA = layout::RowMajor;
```

**EN:** This alias defines `LayoutA` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutA` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 301-301

```cpp
  using FragmentA = Array<bfloat16_t, 8>;
```

**EN:** This alias defines `FragmentA` as `Array<bfloat16_t, 8>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentA` 定义为 `Array<bfloat16_t, 8>` 的别名，以简化后续模板或成员声明。

### Lines 303-303

```cpp
  using ElementB = bfloat16_t;
```

**EN:** This alias defines `ElementB` as `bfloat16_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementB` 定义为 `bfloat16_t` 的别名，以简化后续模板或成员声明。

### Lines 304-304

```cpp
  using LayoutB = layout::ColumnMajor;
```

**EN:** This alias defines `LayoutB` as `layout::ColumnMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutB` 定义为 `layout::ColumnMajor` 的别名，以简化后续模板或成员声明。

### Lines 305-305

```cpp
  using FragmentB = Array<bfloat16_t, 8>;
```

**EN:** This alias defines `FragmentB` as `Array<bfloat16_t, 8>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentB` 定义为 `Array<bfloat16_t, 8>` 的别名，以简化后续模板或成员声明。

### Lines 307-307

```cpp
  using ElementC = float;
```

**EN:** This alias defines `ElementC` as `float`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `float` 的别名，以简化后续模板或成员声明。

### Lines 308-308

```cpp
  using LayoutC = layout::RowMajor;
```

**EN:** This alias defines `LayoutC` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutC` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 309-309

```cpp
  using FragmentC = Array<float, 4>;
```

**EN:** This alias defines `FragmentC` as `Array<float, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentC` 定义为 `Array<float, 4>` 的别名，以简化后续模板或成员声明。

### Lines 311-311

```cpp
  using FragmentE = uint32_t;
```

**EN:** This alias defines `FragmentE` as `uint32_t`, shortening later template or member declarations.

**CN:** 这里把 `FragmentE` 定义为 `uint32_t` 的别名，以简化后续模板或成员声明。

### Lines 313-313

```cpp
  using Operator = OpMultiplyAdd;
```

**EN:** This alias defines `Operator` as `OpMultiplyAdd`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAdd` 的别名，以简化后续模板或成员声明。

### Lines 314-314

```cpp
  using ArchTag = arch::Sm80;
```

**EN:** This alias defines `ArchTag` as `arch::Sm80`, shortening later template or member declarations.

**CN:** 这里把 `ArchTag` 定义为 `arch::Sm80` 的别名，以简化后续模板或成员声明。

### Lines 316-316

```cpp
  static int const kSparse = 2;
```

**EN:** This declaration defines `kSparse` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kSparse`，并把它设为编译期表达式 `2`。

### Lines 318-318

```cpp
  static int const kMetaSizeInBits = 2;
```

**EN:** This declaration defines `kMetaSizeInBits` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kMetaSizeInBits`，并把它设为编译期表达式 `2`。

### Lines 320-320

```cpp
  static int const kMaxID2 = 2;
```

**EN:** This declaration defines `kMaxID2` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kMaxID2`，并把它设为编译期表达式 `2`。

### Lines 322-379

```cpp
  CUTLASS_HOST_DEVICE
  void operator()(FragmentC &d, FragmentA const &a, FragmentB const &b,
                  FragmentC const &c, uint32_t const &E, int const id2) const {

#if defined(CUTLASS_ARCH_SPARSE_MMA_SM80_ENABLED)

    uint32_t const *A = reinterpret_cast<uint32_t const *>(&a);
    uint32_t const *B = reinterpret_cast<uint32_t const *>(&b);
    float const *C = reinterpret_cast<float const *>(&c);
    float *D = reinterpret_cast<float *>(&d);

#if ((__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 5))
    if (id2 == 0) {
      asm volatile(
          "mma.sp::ordered_metadata.sync.aligned.m16n8k32.row.col.f32.bf16.bf16.f32 "
          "{%0,%1,%2,%3}, {%4,%5,%6,%7}, {%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=f"(D[0]), "=f"(D[1]), "=f"(D[2]), "=f"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]), 
            "f"(C[0]), "f"(C[1]), "f"(C[2]), "f"(C[3]), "r"(E));
    } else if (id2 == 1) {
      asm volatile(
          "mma.sp::ordered_metadata.sync.aligned.m16n8k32.row.col.f32.bf16.bf16.f32 "
          "{%0,%1,%2,%3}, {%4,%5,%6,%7}, {%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x1;\n"
          : "=f"(D[0]), "=f"(D[1]), "=f"(D[2]), "=f"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]), 
            "f"(C[0]), "f"(C[1]), "f"(C[2]), "f"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#else
    if (id2 == 0) {
      asm volatile(
          "mma.sp.sync.aligned.m16n8k32.row.col.f32.bf16.bf16.f32 "
          "{%0,%1,%2,%3}, {%4,%5,%6,%7}, {%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=f"(D[0]), "=f"(D[1]), "=f"(D[2]), "=f"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]), 
            "f"(C[0]), "f"(C[1]), "f"(C[2]), "f"(C[3]), "r"(E));
    } else if (id2 == 1) {
      asm volatile(
          "mma.sp.sync.aligned.m16n8k32.row.col.f32.bf16.bf16.f32 "
          "{%0,%1,%2,%3}, {%4,%5,%6,%7}, {%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x1;\n"
          : "=f"(D[0]), "=f"(D[1]), "=f"(D[2]), "=f"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]), 
            "f"(C[0]), "f"(C[1]), "f"(C[2]), "f"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#endif

#else

    CUTLASS_UNUSED(a);
    CUTLASS_UNUSED(b);
    CUTLASS_UNUSED(c);
    CUTLASS_UNUSED(d);
    assert(0);
#endif
  }
```

**EN:** The function `A` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view. Debug/runtime checks guard invalid inputs or extents.

**CN:** `A` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。其中的调试/运行时检查用于防止无效输入或越界范围。

### Lines 382-386

```cpp
////////////////////////////////////////////////////////////////////////////////
//
// Sparse Matrix Multiply 16816 - Float TF32
//
////////////////////////////////////////////////////////////////////////////////
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 388-392

```cpp
/// Matrix multiply-add operation: F32 = tf32 * tf32 + F32
template <>
struct SparseMma<gemm::GemmShape<16, 8, 16>, 32, tfloat32_t, layout::RowMajor,
           tfloat32_t, layout::ColumnMajor, float, layout::RowMajor,
           OpMultiplyAdd, SPFormatType::Thread> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `SparseMma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `SparseMma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 393-393

```cpp
  using Shape = gemm::GemmShape<16, 8, 16>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<16, 8, 16>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<16, 8, 16>` 的别名，以简化后续模板或成员声明。

### Lines 395-395

```cpp
  using ElementA = tfloat32_t;
```

**EN:** This alias defines `ElementA` as `tfloat32_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementA` 定义为 `tfloat32_t` 的别名，以简化后续模板或成员声明。

### Lines 396-396

```cpp
  using LayoutA = layout::RowMajor;
```

**EN:** This alias defines `LayoutA` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutA` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 397-397

```cpp
  using FragmentA = Array<tfloat32_t, 4>;
```

**EN:** This alias defines `FragmentA` as `Array<tfloat32_t, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentA` 定义为 `Array<tfloat32_t, 4>` 的别名，以简化后续模板或成员声明。

### Lines 399-399

```cpp
  using ElementB = tfloat32_t;
```

**EN:** This alias defines `ElementB` as `tfloat32_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementB` 定义为 `tfloat32_t` 的别名，以简化后续模板或成员声明。

### Lines 400-400

```cpp
  using LayoutB = layout::ColumnMajor;
```

**EN:** This alias defines `LayoutB` as `layout::ColumnMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutB` 定义为 `layout::ColumnMajor` 的别名，以简化后续模板或成员声明。

### Lines 401-401

```cpp
  using FragmentB = Array<tfloat32_t, 4>;
```

**EN:** This alias defines `FragmentB` as `Array<tfloat32_t, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentB` 定义为 `Array<tfloat32_t, 4>` 的别名，以简化后续模板或成员声明。

### Lines 403-403

```cpp
  using ElementC = float;
```

**EN:** This alias defines `ElementC` as `float`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `float` 的别名，以简化后续模板或成员声明。

### Lines 404-404

```cpp
  using LayoutC = layout::RowMajor;
```

**EN:** This alias defines `LayoutC` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutC` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 405-405

```cpp
  using FragmentC = Array<float, 4>;
```

**EN:** This alias defines `FragmentC` as `Array<float, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentC` 定义为 `Array<float, 4>` 的别名，以简化后续模板或成员声明。

### Lines 407-407

```cpp
  using FragmentE = uint32_t;
```

**EN:** This alias defines `FragmentE` as `uint32_t`, shortening later template or member declarations.

**CN:** 这里把 `FragmentE` 定义为 `uint32_t` 的别名，以简化后续模板或成员声明。

### Lines 409-409

```cpp
  using Operator = OpMultiplyAdd;
```

**EN:** This alias defines `Operator` as `OpMultiplyAdd`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAdd` 的别名，以简化后续模板或成员声明。

### Lines 410-410

```cpp
  using ArchTag = arch::Sm80;
```

**EN:** This alias defines `ArchTag` as `arch::Sm80`, shortening later template or member declarations.

**CN:** 这里把 `ArchTag` 定义为 `arch::Sm80` 的别名，以简化后续模板或成员声明。

### Lines 412-412

```cpp
  static int const kSparse = 2;
```

**EN:** This declaration defines `kSparse` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kSparse`，并把它设为编译期表达式 `2`。

### Lines 414-414

```cpp
  static int const kMetaSizeInBits = 4;
```

**EN:** This declaration defines `kMetaSizeInBits` and assigns it the compile-time expression `4`.

**CN:** 这个声明定义了 `kMetaSizeInBits`，并把它设为编译期表达式 `4`。

### Lines 416-416

```cpp
  static int const kMaxID2 = 2;
```

**EN:** This declaration defines `kMaxID2` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kMaxID2`，并把它设为编译期表达式 `2`。

### Lines 418-475

```cpp
  CUTLASS_HOST_DEVICE
  void operator()(FragmentC &d, FragmentA const &a, FragmentB const &b,
                  FragmentC const &c, uint32_t const &E, int const id2) const {

#if defined(CUTLASS_ARCH_SPARSE_MMA_SM80_ENABLED)

    uint32_t const *A = reinterpret_cast<uint32_t const *>(&a);
    uint32_t const *B = reinterpret_cast<uint32_t const *>(&b);
    float const *C = reinterpret_cast<float const *>(&c);
    float *D = reinterpret_cast<float *>(&d);

#if ((__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 5))
    if (id2 == 0) {
      asm volatile(
          "mma.sp::ordered_metadata.sync.aligned.m16n8k16.row.col.f32.tf32.tf32.f32 "
          "{%0,%1,%2,%3}, {%4,%5,%6,%7}, {%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=f"(D[0]), "=f"(D[1]), "=f"(D[2]), "=f"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]), 
            "f"(C[0]), "f"(C[1]), "f"(C[2]), "f"(C[3]), "r"(E));
    } else if (id2 == 1) {
      asm volatile(
          "mma.sp::ordered_metadata.sync.aligned.m16n8k16.row.col.f32.tf32.tf32.f32 "
          "{%0,%1,%2,%3}, {%4,%5,%6,%7}, {%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x1;\n"
          : "=f"(D[0]), "=f"(D[1]), "=f"(D[2]), "=f"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]), 
            "f"(C[0]), "f"(C[1]), "f"(C[2]), "f"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#else
    if (id2 == 0) {
      asm volatile(
          "mma.sp.sync.aligned.m16n8k16.row.col.f32.tf32.tf32.f32 "
          "{%0,%1,%2,%3}, {%4,%5,%6,%7}, {%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=f"(D[0]), "=f"(D[1]), "=f"(D[2]), "=f"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]), 
            "f"(C[0]), "f"(C[1]), "f"(C[2]), "f"(C[3]), "r"(E));
    } else if (id2 == 1) {
      asm volatile(
          "mma.sp.sync.aligned.m16n8k16.row.col.f32.tf32.tf32.f32 "
          "{%0,%1,%2,%3}, {%4,%5,%6,%7}, {%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x1;\n"
          : "=f"(D[0]), "=f"(D[1]), "=f"(D[2]), "=f"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]), 
            "f"(C[0]), "f"(C[1]), "f"(C[2]), "f"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#endif

#else

    CUTLASS_UNUSED(a);
    CUTLASS_UNUSED(b);
    CUTLASS_UNUSED(c);
    CUTLASS_UNUSED(d);
    assert(0);
#endif
  }
```

**EN:** The function `A` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view. Debug/runtime checks guard invalid inputs or extents.

**CN:** `A` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。其中的调试/运行时检查用于防止无效输入或越界范围。

### Lines 478-482

```cpp
////////////////////////////////////////////////////////////////////////////////
//
// Sparse Matrix Multiply 16864 - S8 input, S32 accumulation - SATURATE
//
////////////////////////////////////////////////////////////////////////////////
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 484-496

```cpp
/// Matrix multiply-add operation: S32 = S8 * S8 + S32
template <>
struct SparseMma<
  gemm::GemmShape<16,8,64>,
  32,
  int8_t,
  layout::RowMajor,
  int8_t,
  layout::ColumnMajor,
  int,
  layout::RowMajor,
  OpMultiplyAddSaturate,
  SPFormatType::Thread> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `SparseMma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `SparseMma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 498-498

```cpp
  using Shape = gemm::GemmShape<16,8,64>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<16,8,64>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<16,8,64>` 的别名，以简化后续模板或成员声明。

### Lines 500-500

```cpp
  using ElementA = int8_t;
```

**EN:** This alias defines `ElementA` as `int8_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementA` 定义为 `int8_t` 的别名，以简化后续模板或成员声明。

### Lines 501-501

```cpp
  using LayoutA = layout::RowMajor;
```

**EN:** This alias defines `LayoutA` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutA` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 502-502

```cpp
  using FragmentA = Array<int8_t, 16>;
```

**EN:** This alias defines `FragmentA` as `Array<int8_t, 16>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentA` 定义为 `Array<int8_t, 16>` 的别名，以简化后续模板或成员声明。

### Lines 504-504

```cpp
  using ElementB = int8_t;
```

**EN:** This alias defines `ElementB` as `int8_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementB` 定义为 `int8_t` 的别名，以简化后续模板或成员声明。

### Lines 505-505

```cpp
  using LayoutB = layout::ColumnMajor;
```

**EN:** This alias defines `LayoutB` as `layout::ColumnMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutB` 定义为 `layout::ColumnMajor` 的别名，以简化后续模板或成员声明。

### Lines 506-506

```cpp
  using FragmentB = Array<int8_t, 16>;
```

**EN:** This alias defines `FragmentB` as `Array<int8_t, 16>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentB` 定义为 `Array<int8_t, 16>` 的别名，以简化后续模板或成员声明。

### Lines 508-508

```cpp
  using ElementC = int;
```

**EN:** This alias defines `ElementC` as `int`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `int` 的别名，以简化后续模板或成员声明。

### Lines 509-509

```cpp
  using LayoutC = layout::RowMajor;
```

**EN:** This alias defines `LayoutC` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutC` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 510-510

```cpp
  using FragmentC = Array<int, 4>;
```

**EN:** This alias defines `FragmentC` as `Array<int, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentC` 定义为 `Array<int, 4>` 的别名，以简化后续模板或成员声明。

### Lines 512-512

```cpp
  using FragmentE = uint32_t;
```

**EN:** This alias defines `FragmentE` as `uint32_t`, shortening later template or member declarations.

**CN:** 这里把 `FragmentE` 定义为 `uint32_t` 的别名，以简化后续模板或成员声明。

### Lines 514-514

```cpp
  using Operator = OpMultiplyAddSaturate;
```

**EN:** This alias defines `Operator` as `OpMultiplyAddSaturate`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAddSaturate` 的别名，以简化后续模板或成员声明。

### Lines 515-515

```cpp
  using ArchTag = arch::Sm80;
```

**EN:** This alias defines `ArchTag` as `arch::Sm80`, shortening later template or member declarations.

**CN:** 这里把 `ArchTag` 定义为 `arch::Sm80` 的别名，以简化后续模板或成员声明。

### Lines 517-517

```cpp
  static int const kSparse = 2;
```

**EN:** This declaration defines `kSparse` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kSparse`，并把它设为编译期表达式 `2`。

### Lines 519-519

```cpp
  static int const kMetaSizeInBits = 2;
```

**EN:** This declaration defines `kMetaSizeInBits` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kMetaSizeInBits`，并把它设为编译期表达式 `2`。

### Lines 521-521

```cpp
  static int const kMaxID2 = 1;
```

**EN:** This declaration defines `kMaxID2` and assigns it the compile-time expression `1`.

**CN:** 这个声明定义了 `kMaxID2`，并把它设为编译期表达式 `1`。

### Lines 523-573

```cpp
  /// Computes multiply-add
  CUTLASS_HOST_DEVICE
  void operator()(
    FragmentC &d,
    FragmentA const &a,
    FragmentB const &b,
    FragmentC const &c,
    uint32_t const &E,
    int const id2
  ) const {

#if defined(CUTLASS_ARCH_SPARSE_MMA_SM80_ENABLED)

    uint32_t const *A = reinterpret_cast<uint32_t const *>(&a);
    uint32_t const *B = reinterpret_cast<uint32_t const *>(&b);

    int const *C = reinterpret_cast<int const *>(&c);
    int *D = reinterpret_cast<int *>(&d);

#if ((__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 5))
    if (id2 == 0) {
      asm volatile(
          "mma.sp::ordered_metadata.sync.aligned.m16n8k64.row.col.s32.s8.s8.s32.satfinite {%0,%1,%2,%3}, {%4,%5,%6,%7}, "
          "{%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=r"(D[0]), "=r"(D[1]), "=r"(D[2]), "=r"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]),
            "r"(C[0]), "r"(C[1]), "r"(C[2]), "r"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#else
    if (id2 == 0) {
      asm volatile(
          "mma.sp.sync.aligned.m16n8k64.row.col.s32.s8.s8.s32.satfinite {%0,%1,%2,%3}, {%4,%5,%6,%7}, "
          "{%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=r"(D[0]), "=r"(D[1]), "=r"(D[2]), "=r"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]),
            "r"(C[0]), "r"(C[1]), "r"(C[2]), "r"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#endif

#else
    CUTLASS_UNUSED(a);
    CUTLASS_UNUSED(b);
    CUTLASS_UNUSED(c);
    CUTLASS_UNUSED(d);
    assert(0);
#endif
  }
```

**EN:** The preceding comment documents this block. The function `A` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view. Debug/runtime checks guard invalid inputs or extents.

**CN:** 前面的注释说明了这个代码块。`A` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。其中的调试/运行时检查用于防止无效输入或越界范围。

### Lines 576-588

```cpp
/// Matrix multiply-add operation: S32 = S8 * U8 + S32
template <>
struct SparseMma<
  gemm::GemmShape<16,8,64>,
  32,
  int8_t,
  layout::RowMajor,
  uint8_t,
  layout::ColumnMajor,
  int,
  layout::RowMajor,
  OpMultiplyAddSaturate,
  SPFormatType::Thread> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `SparseMma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `SparseMma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 590-590

```cpp
  using Shape = gemm::GemmShape<16,8,64>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<16,8,64>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<16,8,64>` 的别名，以简化后续模板或成员声明。

### Lines 592-592

```cpp
  using ElementA = int8_t;
```

**EN:** This alias defines `ElementA` as `int8_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementA` 定义为 `int8_t` 的别名，以简化后续模板或成员声明。

### Lines 593-593

```cpp
  using LayoutA = layout::RowMajor;
```

**EN:** This alias defines `LayoutA` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutA` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 594-594

```cpp
  using FragmentA = Array<int8_t, 16>;
```

**EN:** This alias defines `FragmentA` as `Array<int8_t, 16>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentA` 定义为 `Array<int8_t, 16>` 的别名，以简化后续模板或成员声明。

### Lines 596-596

```cpp
  using ElementB = uint8_t;
```

**EN:** This alias defines `ElementB` as `uint8_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementB` 定义为 `uint8_t` 的别名，以简化后续模板或成员声明。

### Lines 597-597

```cpp
  using LayoutB = layout::ColumnMajor;
```

**EN:** This alias defines `LayoutB` as `layout::ColumnMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutB` 定义为 `layout::ColumnMajor` 的别名，以简化后续模板或成员声明。

### Lines 598-598

```cpp
  using FragmentB = Array<uint8_t, 16>;
```

**EN:** This alias defines `FragmentB` as `Array<uint8_t, 16>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentB` 定义为 `Array<uint8_t, 16>` 的别名，以简化后续模板或成员声明。

### Lines 600-600

```cpp
  using ElementC = int;
```

**EN:** This alias defines `ElementC` as `int`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `int` 的别名，以简化后续模板或成员声明。

### Lines 601-601

```cpp
  using LayoutC = layout::RowMajor;
```

**EN:** This alias defines `LayoutC` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutC` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 602-602

```cpp
  using FragmentC = Array<int, 4>;
```

**EN:** This alias defines `FragmentC` as `Array<int, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentC` 定义为 `Array<int, 4>` 的别名，以简化后续模板或成员声明。

### Lines 604-604

```cpp
  using FragmentE = uint32_t;
```

**EN:** This alias defines `FragmentE` as `uint32_t`, shortening later template or member declarations.

**CN:** 这里把 `FragmentE` 定义为 `uint32_t` 的别名，以简化后续模板或成员声明。

### Lines 606-606

```cpp
  using Operator = OpMultiplyAddSaturate;
```

**EN:** This alias defines `Operator` as `OpMultiplyAddSaturate`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAddSaturate` 的别名，以简化后续模板或成员声明。

### Lines 607-607

```cpp
  using ArchTag = arch::Sm80;
```

**EN:** This alias defines `ArchTag` as `arch::Sm80`, shortening later template or member declarations.

**CN:** 这里把 `ArchTag` 定义为 `arch::Sm80` 的别名，以简化后续模板或成员声明。

### Lines 609-609

```cpp
  static int const kSparse = 2;
```

**EN:** This declaration defines `kSparse` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kSparse`，并把它设为编译期表达式 `2`。

### Lines 611-611

```cpp
  static int const kMetaSizeInBits = 2;
```

**EN:** This declaration defines `kMetaSizeInBits` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kMetaSizeInBits`，并把它设为编译期表达式 `2`。

### Lines 613-613

```cpp
  static int const kMaxID2 = 1;
```

**EN:** This declaration defines `kMaxID2` and assigns it the compile-time expression `1`.

**CN:** 这个声明定义了 `kMaxID2`，并把它设为编译期表达式 `1`。

### Lines 615-666

```cpp
  /// Computes multiply-add
  CUTLASS_HOST_DEVICE
  void operator()(
    FragmentC &d,
    FragmentA const &a,
    FragmentB const &b,
    FragmentC const &c,
    uint32_t const &E,
    int const id2
  ) const {

#if defined(CUTLASS_ARCH_SPARSE_MMA_SM80_ENABLED)

    uint32_t const *A = reinterpret_cast<uint32_t const *>(&a);
    uint32_t const *B = reinterpret_cast<uint32_t const *>(&b);

    int const *C = reinterpret_cast<int const *>(&c);
    int *D = reinterpret_cast<int *>(&d);

#if ((__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 5))
    if (id2 == 0) {
      asm volatile(
          "mma.sp::ordered_metadata.sync.aligned.m16n8k64.row.col.s32.s8.u8.s32.satfinite {%0,%1,%2,%3}, {%4,%5,%6,%7}, "
          "{%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=r"(D[0]), "=r"(D[1]), "=r"(D[2]), "=r"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]),
            "r"(C[0]), "r"(C[1]), "r"(C[2]), "r"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#else
    if (id2 == 0) {
      asm volatile(
          "mma.sp.sync.aligned.m16n8k64.row.col.s32.s8.u8.s32.satfinite {%0,%1,%2,%3}, {%4,%5,%6,%7}, "
          "{%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=r"(D[0]), "=r"(D[1]), "=r"(D[2]), "=r"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]),
            "r"(C[0]), "r"(C[1]), "r"(C[2]), "r"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#endif

#else

    CUTLASS_UNUSED(a);
    CUTLASS_UNUSED(b);
    CUTLASS_UNUSED(c);
    CUTLASS_UNUSED(d);
    assert(0);
#endif
  }
```

**EN:** The preceding comment documents this block. The function `A` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view. Debug/runtime checks guard invalid inputs or extents.

**CN:** 前面的注释说明了这个代码块。`A` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。其中的调试/运行时检查用于防止无效输入或越界范围。

### Lines 669-681

```cpp
/// Matrix multiply-add operation: S32 = U8 * S8 + S32
template <>
struct SparseMma<
  gemm::GemmShape<16,8,64>,
  32,
  uint8_t,
  layout::RowMajor,
  int8_t,
  layout::ColumnMajor,
  int,
  layout::RowMajor,
  OpMultiplyAddSaturate,
  SPFormatType::Thread> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `SparseMma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `SparseMma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 683-683

```cpp
  using Shape = gemm::GemmShape<16,8,64>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<16,8,64>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<16,8,64>` 的别名，以简化后续模板或成员声明。

### Lines 685-685

```cpp
  using ElementA = uint8_t;
```

**EN:** This alias defines `ElementA` as `uint8_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementA` 定义为 `uint8_t` 的别名，以简化后续模板或成员声明。

### Lines 686-686

```cpp
  using LayoutA = layout::RowMajor;
```

**EN:** This alias defines `LayoutA` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutA` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 687-687

```cpp
  using FragmentA = Array<uint8_t, 16>;
```

**EN:** This alias defines `FragmentA` as `Array<uint8_t, 16>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentA` 定义为 `Array<uint8_t, 16>` 的别名，以简化后续模板或成员声明。

### Lines 689-689

```cpp
  using ElementB = int8_t;
```

**EN:** This alias defines `ElementB` as `int8_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementB` 定义为 `int8_t` 的别名，以简化后续模板或成员声明。

### Lines 690-690

```cpp
  using LayoutB = layout::ColumnMajor;
```

**EN:** This alias defines `LayoutB` as `layout::ColumnMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutB` 定义为 `layout::ColumnMajor` 的别名，以简化后续模板或成员声明。

### Lines 691-691

```cpp
  using FragmentB = Array<int8_t, 16>;
```

**EN:** This alias defines `FragmentB` as `Array<int8_t, 16>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentB` 定义为 `Array<int8_t, 16>` 的别名，以简化后续模板或成员声明。

### Lines 693-693

```cpp
  using ElementC = int;
```

**EN:** This alias defines `ElementC` as `int`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `int` 的别名，以简化后续模板或成员声明。

### Lines 694-694

```cpp
  using LayoutC = layout::RowMajor;
```

**EN:** This alias defines `LayoutC` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutC` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 695-695

```cpp
  using FragmentC = Array<int, 4>;
```

**EN:** This alias defines `FragmentC` as `Array<int, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentC` 定义为 `Array<int, 4>` 的别名，以简化后续模板或成员声明。

### Lines 697-697

```cpp
  using FragmentE = uint32_t;
```

**EN:** This alias defines `FragmentE` as `uint32_t`, shortening later template or member declarations.

**CN:** 这里把 `FragmentE` 定义为 `uint32_t` 的别名，以简化后续模板或成员声明。

### Lines 699-699

```cpp
  using Operator = OpMultiplyAddSaturate;
```

**EN:** This alias defines `Operator` as `OpMultiplyAddSaturate`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAddSaturate` 的别名，以简化后续模板或成员声明。

### Lines 700-700

```cpp
  using ArchTag = arch::Sm80;
```

**EN:** This alias defines `ArchTag` as `arch::Sm80`, shortening later template or member declarations.

**CN:** 这里把 `ArchTag` 定义为 `arch::Sm80` 的别名，以简化后续模板或成员声明。

### Lines 702-702

```cpp
  static int const kSparse = 2;
```

**EN:** This declaration defines `kSparse` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kSparse`，并把它设为编译期表达式 `2`。

### Lines 704-704

```cpp
  static int const kMetaSizeInBits = 2;
```

**EN:** This declaration defines `kMetaSizeInBits` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kMetaSizeInBits`，并把它设为编译期表达式 `2`。

### Lines 706-706

```cpp
  static int const kMaxID2 = 1;
```

**EN:** This declaration defines `kMaxID2` and assigns it the compile-time expression `1`.

**CN:** 这个声明定义了 `kMaxID2`，并把它设为编译期表达式 `1`。

### Lines 708-758

```cpp
  /// Computes multiply-add
  CUTLASS_HOST_DEVICE
  void operator()(
    FragmentC &d,
    FragmentA const &a,
    FragmentB const &b,
    FragmentC const &c,
    uint32_t const &E,
    int const id2
  ) const {

#if defined(CUTLASS_ARCH_SPARSE_MMA_SM80_ENABLED)

    uint32_t const *A = reinterpret_cast<uint32_t const *>(&a);
    uint32_t const *B = reinterpret_cast<uint32_t const *>(&b);

    int const *C = reinterpret_cast<int const *>(&c);
    int *D = reinterpret_cast<int *>(&d);

#if ((__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 5))
    if (id2 == 0) {
      asm volatile(
          "mma.sp::ordered_metadata.sync.aligned.m16n8k64.row.col.s32.u8.s8.s32.satfinite {%0,%1,%2,%3}, {%4,%5,%6,%7}, "
          "{%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=r"(D[0]), "=r"(D[1]), "=r"(D[2]), "=r"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]),
            "r"(C[0]), "r"(C[1]), "r"(C[2]), "r"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#else
    if (id2 == 0) {
      asm volatile(
          "mma.sp.sync.aligned.m16n8k64.row.col.s32.u8.s8.s32.satfinite {%0,%1,%2,%3}, {%4,%5,%6,%7}, "
          "{%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=r"(D[0]), "=r"(D[1]), "=r"(D[2]), "=r"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]),
            "r"(C[0]), "r"(C[1]), "r"(C[2]), "r"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#endif

#else
    CUTLASS_UNUSED(a);
    CUTLASS_UNUSED(b);
    CUTLASS_UNUSED(c);
    CUTLASS_UNUSED(d);
    assert(0);
#endif
  }
```

**EN:** The preceding comment documents this block. The function `A` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view. Debug/runtime checks guard invalid inputs or extents.

**CN:** 前面的注释说明了这个代码块。`A` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。其中的调试/运行时检查用于防止无效输入或越界范围。

### Lines 761-773

```cpp
/// Matrix multiply-add operation: S32 = U8 * U8 + S32
template <>
struct SparseMma<
  gemm::GemmShape<16,8,64>,
  32,
  uint8_t,
  layout::RowMajor,
  uint8_t,
  layout::ColumnMajor,
  int,
  layout::RowMajor,
  OpMultiplyAddSaturate,
  SPFormatType::Thread> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `SparseMma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `SparseMma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 775-775

```cpp
  using Shape = gemm::GemmShape<16,8,64>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<16,8,64>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<16,8,64>` 的别名，以简化后续模板或成员声明。

### Lines 777-777

```cpp
  using ElementA = uint8_t;
```

**EN:** This alias defines `ElementA` as `uint8_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementA` 定义为 `uint8_t` 的别名，以简化后续模板或成员声明。

### Lines 778-778

```cpp
  using LayoutA = layout::RowMajor;
```

**EN:** This alias defines `LayoutA` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutA` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 779-779

```cpp
  using FragmentA = Array<uint8_t, 16>;
```

**EN:** This alias defines `FragmentA` as `Array<uint8_t, 16>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentA` 定义为 `Array<uint8_t, 16>` 的别名，以简化后续模板或成员声明。

### Lines 781-781

```cpp
  using ElementB = uint8_t;
```

**EN:** This alias defines `ElementB` as `uint8_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementB` 定义为 `uint8_t` 的别名，以简化后续模板或成员声明。

### Lines 782-782

```cpp
  using LayoutB = layout::ColumnMajor;
```

**EN:** This alias defines `LayoutB` as `layout::ColumnMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutB` 定义为 `layout::ColumnMajor` 的别名，以简化后续模板或成员声明。

### Lines 783-783

```cpp
  using FragmentB = Array<uint8_t, 16>;
```

**EN:** This alias defines `FragmentB` as `Array<uint8_t, 16>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentB` 定义为 `Array<uint8_t, 16>` 的别名，以简化后续模板或成员声明。

### Lines 785-785

```cpp
  using ElementC = int;
```

**EN:** This alias defines `ElementC` as `int`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `int` 的别名，以简化后续模板或成员声明。

### Lines 786-786

```cpp
  using LayoutC = layout::RowMajor;
```

**EN:** This alias defines `LayoutC` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutC` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 787-787

```cpp
  using FragmentC = Array<int, 4>;
```

**EN:** This alias defines `FragmentC` as `Array<int, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentC` 定义为 `Array<int, 4>` 的别名，以简化后续模板或成员声明。

### Lines 789-789

```cpp
  using FragmentE = uint32_t;
```

**EN:** This alias defines `FragmentE` as `uint32_t`, shortening later template or member declarations.

**CN:** 这里把 `FragmentE` 定义为 `uint32_t` 的别名，以简化后续模板或成员声明。

### Lines 791-791

```cpp
  using Operator = OpMultiplyAddSaturate;
```

**EN:** This alias defines `Operator` as `OpMultiplyAddSaturate`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAddSaturate` 的别名，以简化后续模板或成员声明。

### Lines 792-792

```cpp
  using ArchTag = arch::Sm80;
```

**EN:** This alias defines `ArchTag` as `arch::Sm80`, shortening later template or member declarations.

**CN:** 这里把 `ArchTag` 定义为 `arch::Sm80` 的别名，以简化后续模板或成员声明。

### Lines 794-794

```cpp
  static int const kSparse = 2;
```

**EN:** This declaration defines `kSparse` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kSparse`，并把它设为编译期表达式 `2`。

### Lines 796-796

```cpp
  static int const kMetaSizeInBits = 2;
```

**EN:** This declaration defines `kMetaSizeInBits` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kMetaSizeInBits`，并把它设为编译期表达式 `2`。

### Lines 798-798

```cpp
  static int const kMaxID2 = 1;
```

**EN:** This declaration defines `kMaxID2` and assigns it the compile-time expression `1`.

**CN:** 这个声明定义了 `kMaxID2`，并把它设为编译期表达式 `1`。

### Lines 800-850

```cpp
  /// Computes multiply-add
  CUTLASS_HOST_DEVICE
  void operator()(
    FragmentC &d,
    FragmentA const &a,
    FragmentB const &b,
    FragmentC const &c,
    uint32_t const &E,
    int const id2
  ) const {

#if defined(CUTLASS_ARCH_SPARSE_MMA_SM80_ENABLED)

    uint32_t const *A = reinterpret_cast<uint32_t const *>(&a);
    uint32_t const *B = reinterpret_cast<uint32_t const *>(&b);

    int const *C = reinterpret_cast<int const *>(&c);
    int *D = reinterpret_cast<int *>(&d);

#if ((__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 5))
    if (id2 == 0) {
      asm volatile(
          "mma.sp::ordered_metadata.sync.aligned.m16n8k64.row.col.s32.u8.u8.s32.satfinite {%0,%1,%2,%3}, {%4,%5,%6,%7}, "
          "{%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=r"(D[0]), "=r"(D[1]), "=r"(D[2]), "=r"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]),
            "r"(C[0]), "r"(C[1]), "r"(C[2]), "r"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#else
    if (id2 == 0) {
      asm volatile(
          "mma.sp.sync.aligned.m16n8k64.row.col.s32.u8.u8.s32.satfinite {%0,%1,%2,%3}, {%4,%5,%6,%7}, "
          "{%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=r"(D[0]), "=r"(D[1]), "=r"(D[2]), "=r"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]),
            "r"(C[0]), "r"(C[1]), "r"(C[2]), "r"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#endif

#else
    CUTLASS_UNUSED(a);
    CUTLASS_UNUSED(b);
    CUTLASS_UNUSED(c);
    CUTLASS_UNUSED(d);
    assert(0);
#endif
  }
```

**EN:** The preceding comment documents this block. The function `A` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view. Debug/runtime checks guard invalid inputs or extents.

**CN:** 前面的注释说明了这个代码块。`A` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。其中的调试/运行时检查用于防止无效输入或越界范围。

### Lines 853-857

```cpp
////////////////////////////////////////////////////////////////////////////////
//
// Sparse Matrix Multiply 168128 - S4 input, S32 accumulation - SATURATE
//
////////////////////////////////////////////////////////////////////////////////
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 859-871

```cpp
/// Matrix multiply-add operation: S32 = S4 * S4 + S32
template <>
struct SparseMma<
  gemm::GemmShape<16,8,128>,
  32,
  cutlass::int4b_t,
  layout::RowMajor,
  cutlass::int4b_t,
  layout::ColumnMajor,
  int,
  layout::RowMajor,
  OpMultiplyAddSaturate,
  SPFormatType::Thread> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `SparseMma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `SparseMma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 873-873

```cpp
  using Shape = gemm::GemmShape<16,8,128>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<16,8,128>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<16,8,128>` 的别名，以简化后续模板或成员声明。

### Lines 875-875

```cpp
  using ElementA = cutlass::int4b_t;
```

**EN:** This alias defines `ElementA` as `cutlass::int4b_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementA` 定义为 `cutlass::int4b_t` 的别名，以简化后续模板或成员声明。

### Lines 876-876

```cpp
  using LayoutA = layout::RowMajor;
```

**EN:** This alias defines `LayoutA` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutA` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 877-877

```cpp
  using FragmentA = Array<cutlass::int4b_t, 32>;
```

**EN:** This alias defines `FragmentA` as `Array<cutlass::int4b_t, 32>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentA` 定义为 `Array<cutlass::int4b_t, 32>` 的别名，以简化后续模板或成员声明。

### Lines 879-879

```cpp
  using ElementB = cutlass::int4b_t;
```

**EN:** This alias defines `ElementB` as `cutlass::int4b_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementB` 定义为 `cutlass::int4b_t` 的别名，以简化后续模板或成员声明。

### Lines 880-880

```cpp
  using LayoutB = layout::ColumnMajor;
```

**EN:** This alias defines `LayoutB` as `layout::ColumnMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutB` 定义为 `layout::ColumnMajor` 的别名，以简化后续模板或成员声明。

### Lines 881-881

```cpp
  using FragmentB = Array<cutlass::int4b_t, 32>;
```

**EN:** This alias defines `FragmentB` as `Array<cutlass::int4b_t, 32>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentB` 定义为 `Array<cutlass::int4b_t, 32>` 的别名，以简化后续模板或成员声明。

### Lines 883-883

```cpp
  using ElementC = int;
```

**EN:** This alias defines `ElementC` as `int`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `int` 的别名，以简化后续模板或成员声明。

### Lines 884-884

```cpp
  using LayoutC = layout::RowMajor;
```

**EN:** This alias defines `LayoutC` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutC` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 885-885

```cpp
  using FragmentC = Array<int, 4>;
```

**EN:** This alias defines `FragmentC` as `Array<int, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentC` 定义为 `Array<int, 4>` 的别名，以简化后续模板或成员声明。

### Lines 887-887

```cpp
  using FragmentE = uint32_t;
```

**EN:** This alias defines `FragmentE` as `uint32_t`, shortening later template or member declarations.

**CN:** 这里把 `FragmentE` 定义为 `uint32_t` 的别名，以简化后续模板或成员声明。

### Lines 889-889

```cpp
  using Operator = OpMultiplyAddSaturate;
```

**EN:** This alias defines `Operator` as `OpMultiplyAddSaturate`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAddSaturate` 的别名，以简化后续模板或成员声明。

### Lines 890-890

```cpp
  using ArchTag = arch::Sm80;
```

**EN:** This alias defines `ArchTag` as `arch::Sm80`, shortening later template or member declarations.

**CN:** 这里把 `ArchTag` 定义为 `arch::Sm80` 的别名，以简化后续模板或成员声明。

### Lines 892-892

```cpp
  static int const kSparse = 2;
```

**EN:** This declaration defines `kSparse` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kSparse`，并把它设为编译期表达式 `2`。

### Lines 894-894

```cpp
  static int const kMetaSizeInBits = 2;
```

**EN:** This declaration defines `kMetaSizeInBits` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kMetaSizeInBits`，并把它设为编译期表达式 `2`。

### Lines 896-896

```cpp
  static int const kMaxID2 = 1;
```

**EN:** This declaration defines `kMaxID2` and assigns it the compile-time expression `1`.

**CN:** 这个声明定义了 `kMaxID2`，并把它设为编译期表达式 `1`。

### Lines 898-949

```cpp
  /// Computes multiply-add
  CUTLASS_HOST_DEVICE
  void operator()(
    FragmentC &d,
    FragmentA const &a,
    FragmentB const &b,
    FragmentC const &c,
    uint32_t const &E,
    int const id2
  ) const {

#if defined(CUTLASS_ARCH_SPARSE_MMA_SM80_ENABLED)

    uint32_t const *A = reinterpret_cast<uint32_t const *>(&a);
    uint32_t const *B = reinterpret_cast<uint32_t const *>(&b);

    int const *C = reinterpret_cast<int const *>(&c);
    int *D = reinterpret_cast<int *>(&d);

#if ((__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 5))
    if (id2 == 0) {
      asm volatile(
          "mma.sp::ordered_metadata.sync.aligned.m16n8k128.row.col.s32.s4.s4.s32.satfinite {%0,%1,%2,%3}, {%4,%5,%6,%7}, "
          "{%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=r"(D[0]), "=r"(D[1]), "=r"(D[2]), "=r"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]),
            "r"(C[0]), "r"(C[1]), "r"(C[2]), "r"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#else
    if (id2 == 0) {
      asm volatile(
          "mma.sp.sync.aligned.m16n8k128.row.col.s32.s4.s4.s32.satfinite {%0,%1,%2,%3}, {%4,%5,%6,%7}, "
          "{%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=r"(D[0]), "=r"(D[1]), "=r"(D[2]), "=r"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]),
            "r"(C[0]), "r"(C[1]), "r"(C[2]), "r"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#endif

#else

    CUTLASS_UNUSED(a);
    CUTLASS_UNUSED(b);
    CUTLASS_UNUSED(c);
    CUTLASS_UNUSED(d);
    assert(0);
#endif
  }
```

**EN:** The preceding comment documents this block. The function `A` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view. Debug/runtime checks guard invalid inputs or extents.

**CN:** 前面的注释说明了这个代码块。`A` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。其中的调试/运行时检查用于防止无效输入或越界范围。

### Lines 952-964

```cpp
/// Matrix multiply-add operation: S32 = S4 * U4 + S32
template <>
struct SparseMma<
  gemm::GemmShape<16,8,128>,
  32,
  cutlass::int4b_t,
  layout::RowMajor,
  cutlass::uint4b_t,
  layout::ColumnMajor,
  int,
  layout::RowMajor,
  OpMultiplyAddSaturate,
  SPFormatType::Thread> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `SparseMma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `SparseMma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 966-966

```cpp
  using Shape = gemm::GemmShape<16,8,128>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<16,8,128>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<16,8,128>` 的别名，以简化后续模板或成员声明。

### Lines 968-968

```cpp
  using ElementA = cutlass::int4b_t;
```

**EN:** This alias defines `ElementA` as `cutlass::int4b_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementA` 定义为 `cutlass::int4b_t` 的别名，以简化后续模板或成员声明。

### Lines 969-969

```cpp
  using LayoutA = layout::RowMajor;
```

**EN:** This alias defines `LayoutA` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutA` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 970-970

```cpp
  using FragmentA = Array<cutlass::int4b_t, 32>;
```

**EN:** This alias defines `FragmentA` as `Array<cutlass::int4b_t, 32>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentA` 定义为 `Array<cutlass::int4b_t, 32>` 的别名，以简化后续模板或成员声明。

### Lines 972-972

```cpp
  using ElementB = cutlass::uint4b_t;
```

**EN:** This alias defines `ElementB` as `cutlass::uint4b_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementB` 定义为 `cutlass::uint4b_t` 的别名，以简化后续模板或成员声明。

### Lines 973-973

```cpp
  using LayoutB = layout::ColumnMajor;
```

**EN:** This alias defines `LayoutB` as `layout::ColumnMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutB` 定义为 `layout::ColumnMajor` 的别名，以简化后续模板或成员声明。

### Lines 974-974

```cpp
  using FragmentB = Array<cutlass::uint4b_t, 32>;
```

**EN:** This alias defines `FragmentB` as `Array<cutlass::uint4b_t, 32>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentB` 定义为 `Array<cutlass::uint4b_t, 32>` 的别名，以简化后续模板或成员声明。

### Lines 976-976

```cpp
  using ElementC = int;
```

**EN:** This alias defines `ElementC` as `int`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `int` 的别名，以简化后续模板或成员声明。

### Lines 977-977

```cpp
  using LayoutC = layout::RowMajor;
```

**EN:** This alias defines `LayoutC` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutC` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 978-978

```cpp
  using FragmentC = Array<int, 4>;
```

**EN:** This alias defines `FragmentC` as `Array<int, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentC` 定义为 `Array<int, 4>` 的别名，以简化后续模板或成员声明。

### Lines 980-980

```cpp
  using FragmentE = uint32_t;
```

**EN:** This alias defines `FragmentE` as `uint32_t`, shortening later template or member declarations.

**CN:** 这里把 `FragmentE` 定义为 `uint32_t` 的别名，以简化后续模板或成员声明。

### Lines 982-982

```cpp
  using Operator = OpMultiplyAddSaturate;
```

**EN:** This alias defines `Operator` as `OpMultiplyAddSaturate`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAddSaturate` 的别名，以简化后续模板或成员声明。

### Lines 983-983

```cpp
  using ArchTag = arch::Sm80;
```

**EN:** This alias defines `ArchTag` as `arch::Sm80`, shortening later template or member declarations.

**CN:** 这里把 `ArchTag` 定义为 `arch::Sm80` 的别名，以简化后续模板或成员声明。

### Lines 985-985

```cpp
  static int const kSparse = 2;
```

**EN:** This declaration defines `kSparse` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kSparse`，并把它设为编译期表达式 `2`。

### Lines 987-987

```cpp
  static int const kMetaSizeInBits = 2;
```

**EN:** This declaration defines `kMetaSizeInBits` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kMetaSizeInBits`，并把它设为编译期表达式 `2`。

### Lines 989-989

```cpp
  static int const kMaxID2 = 1;
```

**EN:** This declaration defines `kMaxID2` and assigns it the compile-time expression `1`.

**CN:** 这个声明定义了 `kMaxID2`，并把它设为编译期表达式 `1`。

### Lines 991-1042

```cpp
  /// Computes multiply-add
  CUTLASS_HOST_DEVICE
  void operator()(
    FragmentC &d,
    FragmentA const &a,
    FragmentB const &b,
    FragmentC const &c,
    uint32_t const &E,
    int const id2
  ) const {

#if defined(CUTLASS_ARCH_SPARSE_MMA_SM80_ENABLED)

    uint32_t const *A = reinterpret_cast<uint32_t const *>(&a);
    uint32_t const *B = reinterpret_cast<uint32_t const *>(&b);

    int const *C = reinterpret_cast<int const *>(&c);
    int *D = reinterpret_cast<int *>(&d);

#if ((__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 5))
    if (id2 == 0) {
      asm volatile(
          "mma.sp::ordered_metadata.sync.aligned.m16n8k128.row.col.s32.s4.u4.s32.satfinite {%0,%1,%2,%3}, {%4,%5,%6,%7}, "
          "{%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=r"(D[0]), "=r"(D[1]), "=r"(D[2]), "=r"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]),
            "r"(C[0]), "r"(C[1]), "r"(C[2]), "r"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#else
    if (id2 == 0) {
      asm volatile(
          "mma.sp.sync.aligned.m16n8k128.row.col.s32.s4.u4.s32.satfinite {%0,%1,%2,%3}, {%4,%5,%6,%7}, "
          "{%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=r"(D[0]), "=r"(D[1]), "=r"(D[2]), "=r"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]),
            "r"(C[0]), "r"(C[1]), "r"(C[2]), "r"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#endif

#else

    CUTLASS_UNUSED(a);
    CUTLASS_UNUSED(b);
    CUTLASS_UNUSED(c);
    CUTLASS_UNUSED(d);
    assert(0);
#endif
  }
```

**EN:** The preceding comment documents this block. The function `A` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view. Debug/runtime checks guard invalid inputs or extents.

**CN:** 前面的注释说明了这个代码块。`A` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。其中的调试/运行时检查用于防止无效输入或越界范围。

### Lines 1045-1057

```cpp
/// Matrix multiply-add operation: S32 = U4 * S4 + S32
template <>
struct SparseMma<
  gemm::GemmShape<16,8,128>,
  32,
  cutlass::uint4b_t,
  layout::RowMajor,
  cutlass::int4b_t,
  layout::ColumnMajor,
  int,
  layout::RowMajor,
  OpMultiplyAddSaturate,
  SPFormatType::Thread> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `SparseMma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `SparseMma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 1059-1059

```cpp
  using Shape = gemm::GemmShape<16,8,128>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<16,8,128>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<16,8,128>` 的别名，以简化后续模板或成员声明。

### Lines 1061-1061

```cpp
  using ElementA = cutlass::uint4b_t;
```

**EN:** This alias defines `ElementA` as `cutlass::uint4b_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementA` 定义为 `cutlass::uint4b_t` 的别名，以简化后续模板或成员声明。

### Lines 1062-1062

```cpp
  using LayoutA = layout::RowMajor;
```

**EN:** This alias defines `LayoutA` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutA` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 1063-1063

```cpp
  using FragmentA = Array<cutlass::uint4b_t, 32>;
```

**EN:** This alias defines `FragmentA` as `Array<cutlass::uint4b_t, 32>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentA` 定义为 `Array<cutlass::uint4b_t, 32>` 的别名，以简化后续模板或成员声明。

### Lines 1065-1065

```cpp
  using ElementB = cutlass::int4b_t;
```

**EN:** This alias defines `ElementB` as `cutlass::int4b_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementB` 定义为 `cutlass::int4b_t` 的别名，以简化后续模板或成员声明。

### Lines 1066-1066

```cpp
  using LayoutB = layout::ColumnMajor;
```

**EN:** This alias defines `LayoutB` as `layout::ColumnMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutB` 定义为 `layout::ColumnMajor` 的别名，以简化后续模板或成员声明。

### Lines 1067-1067

```cpp
  using FragmentB = Array<cutlass::int4b_t, 32>;
```

**EN:** This alias defines `FragmentB` as `Array<cutlass::int4b_t, 32>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentB` 定义为 `Array<cutlass::int4b_t, 32>` 的别名，以简化后续模板或成员声明。

### Lines 1069-1069

```cpp
  using ElementC = int;
```

**EN:** This alias defines `ElementC` as `int`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `int` 的别名，以简化后续模板或成员声明。

### Lines 1070-1070

```cpp
  using LayoutC = layout::RowMajor;
```

**EN:** This alias defines `LayoutC` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutC` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 1071-1071

```cpp
  using FragmentC = Array<int, 4>;
```

**EN:** This alias defines `FragmentC` as `Array<int, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentC` 定义为 `Array<int, 4>` 的别名，以简化后续模板或成员声明。

### Lines 1073-1073

```cpp
  using FragmentE = uint32_t;
```

**EN:** This alias defines `FragmentE` as `uint32_t`, shortening later template or member declarations.

**CN:** 这里把 `FragmentE` 定义为 `uint32_t` 的别名，以简化后续模板或成员声明。

### Lines 1075-1075

```cpp
  using Operator = OpMultiplyAddSaturate;
```

**EN:** This alias defines `Operator` as `OpMultiplyAddSaturate`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAddSaturate` 的别名，以简化后续模板或成员声明。

### Lines 1076-1076

```cpp
  using ArchTag = arch::Sm80;
```

**EN:** This alias defines `ArchTag` as `arch::Sm80`, shortening later template or member declarations.

**CN:** 这里把 `ArchTag` 定义为 `arch::Sm80` 的别名，以简化后续模板或成员声明。

### Lines 1078-1078

```cpp
  static int const kSparse = 2;
```

**EN:** This declaration defines `kSparse` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kSparse`，并把它设为编译期表达式 `2`。

### Lines 1080-1080

```cpp
  static int const kMetaSizeInBits = 2;
```

**EN:** This declaration defines `kMetaSizeInBits` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kMetaSizeInBits`，并把它设为编译期表达式 `2`。

### Lines 1082-1082

```cpp
  static int const kMaxID2 = 1;
```

**EN:** This declaration defines `kMaxID2` and assigns it the compile-time expression `1`.

**CN:** 这个声明定义了 `kMaxID2`，并把它设为编译期表达式 `1`。

### Lines 1084-1135

```cpp
  /// Computes multiply-add
  CUTLASS_HOST_DEVICE
  void operator()(
    FragmentC &d,
    FragmentA const &a,
    FragmentB const &b,
    FragmentC const &c,
    uint32_t const &E,
    int const id2
  ) const {

#if defined(CUTLASS_ARCH_SPARSE_MMA_SM80_ENABLED)

    uint32_t const *A = reinterpret_cast<uint32_t const *>(&a);
    uint32_t const *B = reinterpret_cast<uint32_t const *>(&b);

    int const *C = reinterpret_cast<int const *>(&c);
    int *D = reinterpret_cast<int *>(&d);

#if ((__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 5))
    if (id2 == 0) {
      asm volatile(
          "mma.sp::ordered_metadata.sync.aligned.m16n8k128.row.col.s32.u4.s4.s32.satfinite {%0,%1,%2,%3}, {%4,%5,%6,%7}, "
          "{%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=r"(D[0]), "=r"(D[1]), "=r"(D[2]), "=r"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]),
            "r"(C[0]), "r"(C[1]), "r"(C[2]), "r"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#else
    if (id2 == 0) {
      asm volatile(
          "mma.sp.sync.aligned.m16n8k128.row.col.s32.u4.s4.s32.satfinite {%0,%1,%2,%3}, {%4,%5,%6,%7}, "
          "{%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=r"(D[0]), "=r"(D[1]), "=r"(D[2]), "=r"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]),
            "r"(C[0]), "r"(C[1]), "r"(C[2]), "r"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#endif

#else

    CUTLASS_UNUSED(a);
    CUTLASS_UNUSED(b);
    CUTLASS_UNUSED(c);
    CUTLASS_UNUSED(d);
    assert(0);
#endif
  }
```

**EN:** The preceding comment documents this block. The function `A` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view. Debug/runtime checks guard invalid inputs or extents.

**CN:** 前面的注释说明了这个代码块。`A` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。其中的调试/运行时检查用于防止无效输入或越界范围。

### Lines 1138-1150

```cpp
/// Matrix multiply-add operation: S32 = U4 * U4 + S32
template <>
struct SparseMma<
  gemm::GemmShape<16,8,128>,
  32,
  cutlass::uint4b_t,
  layout::RowMajor,
  cutlass::uint4b_t,
  layout::ColumnMajor,
  int,
  layout::RowMajor,
  OpMultiplyAddSaturate,
  SPFormatType::Thread> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `SparseMma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `SparseMma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 1152-1152

```cpp
  using Shape = gemm::GemmShape<16,8,128>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<16,8,128>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<16,8,128>` 的别名，以简化后续模板或成员声明。

### Lines 1154-1154

```cpp
  using ElementA = cutlass::uint4b_t;
```

**EN:** This alias defines `ElementA` as `cutlass::uint4b_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementA` 定义为 `cutlass::uint4b_t` 的别名，以简化后续模板或成员声明。

### Lines 1155-1155

```cpp
  using LayoutA = layout::RowMajor;
```

**EN:** This alias defines `LayoutA` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutA` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 1156-1156

```cpp
  using FragmentA = Array<cutlass::uint4b_t, 32>;
```

**EN:** This alias defines `FragmentA` as `Array<cutlass::uint4b_t, 32>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentA` 定义为 `Array<cutlass::uint4b_t, 32>` 的别名，以简化后续模板或成员声明。

### Lines 1158-1158

```cpp
  using ElementB = cutlass::uint4b_t;
```

**EN:** This alias defines `ElementB` as `cutlass::uint4b_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementB` 定义为 `cutlass::uint4b_t` 的别名，以简化后续模板或成员声明。

### Lines 1159-1159

```cpp
  using LayoutB = layout::ColumnMajor;
```

**EN:** This alias defines `LayoutB` as `layout::ColumnMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutB` 定义为 `layout::ColumnMajor` 的别名，以简化后续模板或成员声明。

### Lines 1160-1160

```cpp
  using FragmentB = Array<cutlass::uint4b_t, 32>;
```

**EN:** This alias defines `FragmentB` as `Array<cutlass::uint4b_t, 32>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentB` 定义为 `Array<cutlass::uint4b_t, 32>` 的别名，以简化后续模板或成员声明。

### Lines 1162-1162

```cpp
  using ElementC = int;
```

**EN:** This alias defines `ElementC` as `int`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `int` 的别名，以简化后续模板或成员声明。

### Lines 1163-1163

```cpp
  using LayoutC = layout::RowMajor;
```

**EN:** This alias defines `LayoutC` as `layout::RowMajor`, shortening later template or member declarations.

**CN:** 这里把 `LayoutC` 定义为 `layout::RowMajor` 的别名，以简化后续模板或成员声明。

### Lines 1164-1164

```cpp
  using FragmentC = Array<int, 4>;
```

**EN:** This alias defines `FragmentC` as `Array<int, 4>`, shortening later template or member declarations.

**CN:** 这里把 `FragmentC` 定义为 `Array<int, 4>` 的别名，以简化后续模板或成员声明。

### Lines 1166-1166

```cpp
  using FragmentE = uint32_t;
```

**EN:** This alias defines `FragmentE` as `uint32_t`, shortening later template or member declarations.

**CN:** 这里把 `FragmentE` 定义为 `uint32_t` 的别名，以简化后续模板或成员声明。

### Lines 1168-1168

```cpp
  using Operator = OpMultiplyAddSaturate;
```

**EN:** This alias defines `Operator` as `OpMultiplyAddSaturate`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAddSaturate` 的别名，以简化后续模板或成员声明。

### Lines 1169-1169

```cpp
  using ArchTag = arch::Sm80;
```

**EN:** This alias defines `ArchTag` as `arch::Sm80`, shortening later template or member declarations.

**CN:** 这里把 `ArchTag` 定义为 `arch::Sm80` 的别名，以简化后续模板或成员声明。

### Lines 1171-1171

```cpp
  static int const kSparse = 2;
```

**EN:** This declaration defines `kSparse` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kSparse`，并把它设为编译期表达式 `2`。

### Lines 1173-1173

```cpp
  static int const kMetaSizeInBits = 2;
```

**EN:** This declaration defines `kMetaSizeInBits` and assigns it the compile-time expression `2`.

**CN:** 这个声明定义了 `kMetaSizeInBits`，并把它设为编译期表达式 `2`。

### Lines 1175-1175

```cpp
  static int const kMaxID2 = 1;
```

**EN:** This declaration defines `kMaxID2` and assigns it the compile-time expression `1`.

**CN:** 这个声明定义了 `kMaxID2`，并把它设为编译期表达式 `1`。

### Lines 1177-1228

```cpp
  /// Computes multiply-add
  CUTLASS_HOST_DEVICE
  void operator()(
    FragmentC &d,
    FragmentA const &a,
    FragmentB const &b,
    FragmentC const &c,
    uint32_t const &E,
    int const id2
  ) const {

#if defined(CUTLASS_ARCH_SPARSE_MMA_SM80_ENABLED)

    uint32_t const *A = reinterpret_cast<uint32_t const *>(&a);
    uint32_t const *B = reinterpret_cast<uint32_t const *>(&b);

    int const *C = reinterpret_cast<int const *>(&c);
    int *D = reinterpret_cast<int *>(&d);

#if ((__CUDACC_VER_MAJOR__ > 12) || (__CUDACC_VER_MAJOR__ == 12 && __CUDACC_VER_MINOR__ >= 5))
    if (id2 == 0) {
      asm volatile(
          "mma.sp::ordered_metadata.sync.aligned.m16n8k128.row.col.s32.u4.u4.s32.satfinite {%0,%1,%2,%3}, {%4,%5,%6,%7}, "
          "{%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=r"(D[0]), "=r"(D[1]), "=r"(D[2]), "=r"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]),
            "r"(C[0]), "r"(C[1]), "r"(C[2]), "r"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#else
    if (id2 == 0) {
      asm volatile(
          "mma.sp.sync.aligned.m16n8k128.row.col.s32.u4.u4.s32.satfinite {%0,%1,%2,%3}, {%4,%5,%6,%7}, "
          "{%8,%9,%10,%11}, {%12,%13,%14,%15}, %16, 0x0;\n"
          : "=r"(D[0]), "=r"(D[1]), "=r"(D[2]), "=r"(D[3])
          : "r"(A[0]), "r"(A[1]), "r"(A[2]), "r"(A[3]), "r"(B[0]), "r"(B[1]), "r"(B[2]), "r"(B[3]),
            "r"(C[0]), "r"(C[1]), "r"(C[2]), "r"(C[3]), "r"(E));
    } else {
      assert(0);
    }
#endif

#else

    CUTLASS_UNUSED(a);
    CUTLASS_UNUSED(b);
    CUTLASS_UNUSED(c);
    CUTLASS_UNUSED(d);
    assert(0);
#endif
  }
```

**EN:** The preceding comment documents this block. The function `A` implements a concrete operation in this abstraction. Inline assembly binds the interface directly to a hardware instruction. It reinterprets raw storage to expose a typed view. Debug/runtime checks guard invalid inputs or extents.

**CN:** 前面的注释说明了这个代码块。`A` 函数实现了该抽象中的一个具体操作。内联汇编把这个接口直接绑定到具体硬件指令上。它通过重新解释原始存储来暴露带类型的视图。其中的调试/运行时检查用于防止无效输入或越界范围。

## Key Concepts / 关键概念

- **EN:** Heavy use of templates enables compile-time specialization and zero-overhead abstractions.
  **CN:** 大量使用模板，使该文件能够进行编译期特化并保持零额外开销的抽象。

- **EN:** Architecture-specific paths map C++ wrappers onto GPU instructions or micro-architectural features.
  **CN:** 架构特定路径会把 C++ 封装映射到 GPU 指令或底层微架构特性。

- **EN:** The file handles specialized numeric formats or packed data representations used by accelerators.
  **CN:** 该文件处理加速器常用的特种数值格式或打包数据表示。

- **EN:** Fixed-size containers and fragments are used to model register or shared-memory tiles.
  **CN:** 定长容器与片段类型用于建模寄存器块或共享内存块。

- **EN:** Conditional compilation keeps the header portable across host, device, CUDA, and SYCL builds.
  **CN:** 条件编译使该头文件能够在主机端、设备端、CUDA 与 SYCL 构建之间保持可移植性。

## Dependencies / 依赖关系

- **EN:** Direct includes: `cutlass/cutlass.h`, `CUDA_STD_HEADER(cassert)`, `mma.h`, `cutlass/layout/matrix.h`, `cutlass/numeric_types.h`.
  **CN:** 直接包含：`cutlass/cutlass.h`, `CUDA_STD_HEADER(cassert)`, `mma.h`, `cutlass/layout/matrix.h`, `cutlass/numeric_types.h`。

- **EN:** Primary namespaces: `cutlass`, `arch`.
  **CN:** 主要命名空间：`cutlass`, `arch`。

- **EN:** Important macros or compile flags: `CUDA_STD_HEADER`, `CUTLASS_ARCH_SPARSE_MMA_SM80_ENABLED`, `CUTLASS_ARCH_SPARSE_MMA_SM80_SUPPORTED`, `CUTLASS_ENABLE_SYCL`, `CUTLASS_HOST_DEVICE`, `CUTLASS_UNUSED`.
  **CN:** 重要宏或编译开关：`CUDA_STD_HEADER`, `CUTLASS_ARCH_SPARSE_MMA_SM80_ENABLED`, `CUTLASS_ARCH_SPARSE_MMA_SM80_SUPPORTED`, `CUTLASS_ENABLE_SYCL`, `CUTLASS_HOST_DEVICE`, `CUTLASS_UNUSED`。
