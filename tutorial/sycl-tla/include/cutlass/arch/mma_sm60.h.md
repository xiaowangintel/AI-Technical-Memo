# mma_sm60.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/arch/mma_sm60.h`

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

### Lines 37-37

```cpp
#if !defined(CUTLASS_ENABLE_SYCL)
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#if !defined(CUTLASS_ENABLE_SYCL)`.

**CN:** 这个预处理代码块围绕 `#if !defined(CUTLASS_ENABLE_SYCL)` 选择编译路径或功能开关。

### Lines 38-38

```cpp
#include <cuda_fp16.h>
```

**EN:** This block imports dependencies such as `cuda_fp16.h`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cuda_fp16.h` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 39-39

```cpp
#endif
```

**EN:** This preprocessor block selects a compilation path or feature flag around `#endif`.

**CN:** 这个预处理代码块围绕 `#endif` 选择编译路径或功能开关。

### Lines 41-41

```cpp
#include "cutlass/arch/mma.h"
```

**EN:** This block imports dependencies such as `cutlass/arch/mma.h`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cutlass/arch/mma.h` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 43-43

```cpp
#include "cutlass/layout/matrix.h"
```

**EN:** This block imports dependencies such as `cutlass/layout/matrix.h`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cutlass/layout/matrix.h` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 45-46

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
namespace cutlass {
```

**EN:** The preceding comment documents this block. This block opens the namespace scope `cutlass` for the declarations that follow.

**CN:** 前面的注释说明了这个代码块。该代码块打开了 `cutlass` 命名空间作用域，以容纳后续声明。

### Lines 48-48

```cpp
namespace arch {
```

**EN:** This block opens the namespace scope `arch` for the declarations that follow.

**CN:** 该代码块打开了 `arch` 命名空间作用域，以容纳后续声明。

### Lines 52-63

```cpp
/// Matrix multiply-add operation
template <typename LayoutA, typename LayoutB, typename LayoutC>
struct Mma<
  gemm::GemmShape<2,1,1>,
  1,
  half_t,
  LayoutA,
  half_t,
  LayoutB,
  half_t,
  LayoutC,
  OpMultiplyAdd> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `Mma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `Mma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 65-65

```cpp
  using Shape = gemm::GemmShape<2, 1, 1>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<2, 1, 1>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<2, 1, 1>` 的别名，以简化后续模板或成员声明。

### Lines 66-66

```cpp
  using Operator = OpMultiplyAdd;
```

**EN:** This alias defines `Operator` as `OpMultiplyAdd`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAdd` 的别名，以简化后续模板或成员声明。

### Lines 67-67

```cpp
  using ElementC = half_t;
```

**EN:** This alias defines `ElementC` as `half_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `half_t` 的别名，以简化后续模板或成员声明。

### Lines 69-93

```cpp
  CUTLASS_HOST_DEVICE
  void operator()(
    Array<half_t, 2> &d,
    Array<half_t, 2> const &a,
    Array<half_t, 1> const &b,
    Array<half_t, 2> const &c
  ) {

#if (defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 600))

    __half2 const & A = reinterpret_cast<__half2 const &>(a);
    __half2 B = __half2half2(reinterpret_cast<__half const &>(b));
    __half2 const & C = reinterpret_cast<__half2 const &>(c);

    __half2 D = __hfma2(A, B, C);

    d = reinterpret_cast<Array<half_t, 2> &>(D);

#else
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < 2; ++i) {
      d[i] = a[i] * b[0] + c[i];
    }
#endif
  }
```

**EN:** The function `A` implements a concrete operation in this abstraction. It reinterprets raw storage to expose a typed view.

**CN:** `A` 函数实现了该抽象中的一个具体操作。它通过重新解释原始存储来暴露带类型的视图。

### Lines 98-109

```cpp
/// Matrix multiply-add operation
template <typename LayoutA, typename LayoutB>
struct Mma<
  gemm::GemmShape<1,2,1>,
  1,
  half_t,
  LayoutA,
  half_t,
  LayoutB,
  half_t,
  layout::RowMajor,
  OpMultiplyAdd> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `Mma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `Mma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 111-111

```cpp
  using Shape = gemm::GemmShape<1, 2, 1>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<1, 2, 1>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<1, 2, 1>` 的别名，以简化后续模板或成员声明。

### Lines 112-112

```cpp
  using Operator = OpMultiplyAdd;
```

**EN:** This alias defines `Operator` as `OpMultiplyAdd`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAdd` 的别名，以简化后续模板或成员声明。

### Lines 113-113

```cpp
  using ElementC = half_t;
```

**EN:** This alias defines `ElementC` as `half_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `half_t` 的别名，以简化后续模板或成员声明。

### Lines 115-139

```cpp
  CUTLASS_HOST_DEVICE
  void operator()(
    Array<half_t, 2> &d,
    Array<half_t, 1> const &a,
    Array<half_t, 2> const &b,
    Array<half_t, 2> const &c
  ) {

#if (defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 600))

    __half2 const & A = __half2half2(reinterpret_cast<__half const &>(a));
    __half2 B = reinterpret_cast<__half2 const &>(b);
    __half2 const & C = reinterpret_cast<__half2 const &>(c);

    __half2 D = __hfma2(A, B, C);

    d = reinterpret_cast<Array<half_t, 2> &>(D);

#else
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < 2; ++i) {
      d[i] = a[0] * b[i] + c[i];
    }
#endif
  }
```

**EN:** The function `A` implements a concrete operation in this abstraction. It reinterprets raw storage to expose a typed view.

**CN:** `A` 函数实现了该抽象中的一个具体操作。它通过重新解释原始存储来暴露带类型的视图。

### Lines 144-155

```cpp
/// Matrix multiply-add operation
template <>
struct Mma <
  gemm::GemmShape<2, 2, 1>,
  1,
  half_t,
  layout::ColumnMajor,
  half_t,
  layout::RowMajor,
  half_t,
  layout::ColumnMajor,
  OpMultiplyAdd> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `Mma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `Mma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 157-157

```cpp
  using Shape = gemm::GemmShape<2, 2, 1>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<2, 2, 1>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<2, 2, 1>` 的别名，以简化后续模板或成员声明。

### Lines 158-158

```cpp
  using Operator = OpMultiplyAdd;
```

**EN:** This alias defines `Operator` as `OpMultiplyAdd`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAdd` 的别名，以简化后续模板或成员声明。

### Lines 159-159

```cpp
  using ElementC = half_t;
```

**EN:** This alias defines `ElementC` as `half_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `half_t` 的别名，以简化后续模板或成员声明。

### Lines 161-194

```cpp
  CUTLASS_HOST_DEVICE
  void operator()(
    Array<half_t, 4> &d,
    Array<half_t, 2> const &a,
    Array<half_t, 2> const &b,
    Array<half_t, 4> const &c
  ) {

#if (defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 600))

    __half2 const & A = reinterpret_cast<__half2 const &>(a);
    __half2 Blo = __low2half2(reinterpret_cast<__half2 const &>(b));
    __half2 Bhi = __high2half2(reinterpret_cast<__half2 const &>(b));

    __half2 const *C = reinterpret_cast<__half2 const *>(&c);

    __half2 Dlo = __hfma2(A, Blo, C[0]);
    __half2 Dhi = __hfma2(A, Bhi, C[1]);

    Array<half_t, 2> * D = reinterpret_cast<Array<half_t, 2> *>(&d);

    D[0] = reinterpret_cast<Array<half_t, 2> const &>(Dlo);
    D[1] = reinterpret_cast<Array<half_t, 2> const &>(Dhi);

#else
    CUTLASS_PRAGMA_UNROLL
    for (int j = 0; j < 2; ++j) {
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < 2; ++i) {
        d[i + 2 * j] = a[i] * b[j] + c[i + 2 * j];
      }
    }
#endif
  }
```

**EN:** The function `A` implements a concrete operation in this abstraction. It reinterprets raw storage to expose a typed view.

**CN:** `A` 函数实现了该抽象中的一个具体操作。它通过重新解释原始存储来暴露带类型的视图。

### Lines 199-210

```cpp
/// Matrix multiply-add operation
template <>
struct Mma<
  gemm::GemmShape<2, 2, 1>,
  1,
  half_t,
  layout::ColumnMajor,
  half_t,
  layout::RowMajor,
  half_t,
  layout::RowMajor,
  OpMultiplyAdd> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `Mma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `Mma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 212-212

```cpp
  using Shape = gemm::GemmShape<2, 2, 1>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<2, 2, 1>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<2, 2, 1>` 的别名，以简化后续模板或成员声明。

### Lines 213-213

```cpp
  using Operator = OpMultiplyAdd;
```

**EN:** This alias defines `Operator` as `OpMultiplyAdd`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `OpMultiplyAdd` 的别名，以简化后续模板或成员声明。

### Lines 214-214

```cpp
  using ElementC = half_t;
```

**EN:** This alias defines `ElementC` as `half_t`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `half_t` 的别名，以简化后续模板或成员声明。

### Lines 216-248

```cpp
  CUTLASS_HOST_DEVICE
  void operator()(
    Array<half_t, 4> &d,
    Array<half_t, 2> const &a,
    Array<half_t, 2> const &b,
    Array<half_t, 4> const &c
  ) {

#if (defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 600))

    __half2 Alo = __low2half2(reinterpret_cast<__half2 const &>(a));
    __half2 Ahi = __high2half2(reinterpret_cast<__half2 const &>(a));
    __half2 const & B = reinterpret_cast<__half2 const &>(b);

    __half2 const *C = reinterpret_cast<__half2 const *>(&c);

    __half2 Dlo = __hfma2(Alo, B, C[0]);
    __half2 Dhi = __hfma2(Ahi, B, C[1]);

    Array<half_t, 2> * D = reinterpret_cast<Array<half_t, 2> *>(&d);

    D[0] = reinterpret_cast<Array<half_t, 2> &>(Dlo);
    D[1] = reinterpret_cast<Array<half_t, 2> &>(Dhi);
#else
    CUTLASS_PRAGMA_UNROLL
    for (int i = 0; i < 2; ++i) {
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < 2; ++j) {
        d[i * 2 + j] = a[i] * b[j] + c[i * 2 + j];
      }
    }
#endif
  }
```

**EN:** The function `Alo` implements a concrete operation in this abstraction. It reinterprets raw storage to expose a typed view.

**CN:** `Alo` 函数实现了该抽象中的一个具体操作。它通过重新解释原始存储来暴露带类型的视图。

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

- **EN:** Direct includes: `cuda_fp16.h`, `cutlass/arch/mma.h`, `cutlass/layout/matrix.h`.
  **CN:** 直接包含：`cuda_fp16.h`, `cutlass/arch/mma.h`, `cutlass/layout/matrix.h`。

- **EN:** Primary namespaces: `cutlass`, `arch`.
  **CN:** 主要命名空间：`cutlass`, `arch`。

- **EN:** Important macros or compile flags: `CUTLASS_ENABLE_SYCL`, `CUTLASS_HOST_DEVICE`, `CUTLASS_PRAGMA_UNROLL`.
  **CN:** 重要宏或编译开关：`CUTLASS_ENABLE_SYCL`, `CUTLASS_HOST_DEVICE`, `CUTLASS_PRAGMA_UNROLL`。
