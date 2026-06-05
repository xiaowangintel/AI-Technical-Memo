# mma.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/arch/mma.h`

- **EN:** Templates exposing architecture support for multiply-add operations

- **CN:** 该头文件主要实现 CUTLASS 的架构相关原语与指令封装。文件级摘要：Templates exposing architecture support for multiply-add operations

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
    \brief Templates exposing architecture support for multiply-add operations
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
#include "cutlass/array.h"
#include "cutlass/numeric_types.h"
#include "cutlass/functional.h"
```

**EN:** This block imports dependencies such as `cutlass/array.h`, `cutlass/numeric_types.h`, `cutlass/functional.h`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cutlass/array.h`, `cutlass/numeric_types.h`, `cutlass/functional.h` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 41-42

```cpp
#include "cutlass/gemm/gemm.h"
#include "cutlass/arch/arch.h"
```

**EN:** This block imports dependencies such as `cutlass/gemm/gemm.h`, `cutlass/arch/arch.h`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cutlass/gemm/gemm.h`, `cutlass/arch/arch.h` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 44-45

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
namespace cutlass {
```

**EN:** The preceding comment documents this block. This block opens the namespace scope `cutlass` for the declarations that follow.

**CN:** 前面的注释说明了这个代码块。该代码块打开了 `cutlass` 命名空间作用域，以容纳后续声明。

### Lines 47-47

```cpp
namespace arch {
```

**EN:** This block opens the namespace scope `arch` for the declarations that follow.

**CN:** 该代码块打开了 `arch` 命名空间作用域，以容纳后续声明。

### Lines 51-52

```cpp
/// Tag indicating the operation implied by MMA.
struct OpMultiplyAdd {};
```

**EN:** The preceding comment documents this block. This block declares `OpMultiplyAdd` as a lightweight `struct`, often used as a tag or thin wrapper.

**CN:** 前面的注释说明了这个代码块。该代码块将 `OpMultiplyAdd` 声明为一个轻量级 `struct`，通常用作标签类型或薄封装。

### Lines 56-57

```cpp
/// Tag indicating the result is saturated to MAX_FLOAT|MIN_FLOAT or MAX_INT|MIN_INT
struct OpMultiplyAddSaturate {};
```

**EN:** The preceding comment documents this block. This block declares `OpMultiplyAddSaturate` as a lightweight `struct`, often used as a tag or thin wrapper.

**CN:** 前面的注释说明了这个代码块。该代码块将 `OpMultiplyAddSaturate` 声明为一个轻量级 `struct`，通常用作标签类型或薄封装。

### Lines 61-62

```cpp
/// Tag indicating the input is converted to a narrower type (BF16)
struct OpMultiplyAddFastBF16 {};
```

**EN:** The preceding comment documents this block. This block declares `OpMultiplyAddFastBF16` as a lightweight `struct`, often used as a tag or thin wrapper.

**CN:** 前面的注释说明了这个代码块。该代码块将 `OpMultiplyAddFastBF16` 声明为一个轻量级 `struct`，通常用作标签类型或薄封装。

### Lines 66-67

```cpp
/// Tag indicating the input is converted to a narrower type (F16)
struct OpMultiplyAddFastF16 {};
```

**EN:** The preceding comment documents this block. This block declares `OpMultiplyAddFastF16` as a lightweight `struct`, often used as a tag or thin wrapper.

**CN:** 前面的注释说明了这个代码块。该代码块将 `OpMultiplyAddFastF16` 声明为一个轻量级 `struct`，通常用作标签类型或薄封装。

### Lines 71-73

```cpp
/// Tag indicating the input data types are mixed and the narrower type is 
/// upcasted to the wider type
struct OpMultiplyAddMixedInputUpcast {};
```

**EN:** The preceding comment documents this block. This block declares `OpMultiplyAddMixedInputUpcast` as a lightweight `struct`, often used as a tag or thin wrapper.

**CN:** 前面的注释说明了这个代码块。该代码块将 `OpMultiplyAddMixedInputUpcast` 声明为一个轻量级 `struct`，通常用作标签类型或薄封装。

### Lines 75-79

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
/// Tag indicating the input is converted to 2 (big and small) TF32 or FP16 components
//  Perform 3xTF32 or 4xTF32 for every F32 output element on Ampere
//  Perform 3xFP16 or 4xFP16 for every F32 output element on Hopper with axiswise quantization factor support
struct OpMultiplyAddFastF32 {};
```

**EN:** The preceding comment documents this block. This block declares `OpMultiplyAddFastF32` as a lightweight `struct`, often used as a tag or thin wrapper.

**CN:** 前面的注释说明了这个代码块。该代码块将 `OpMultiplyAddFastF32` 声明为一个轻量级 `struct`，通常用作标签类型或薄封装。

### Lines 81-85

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
/// Tag indicating the input is converted to 2 (big and small) TF32 or FP16 components
//  Perform 3xTF32 or 4xTF32 for every complex<F32> output element on Ampere
//  Perform 3xFP16 or 4xFP16 for every complex<F32> output element on Hopper with axiswise quantization factor support
struct OpMultiplyAddComplexFastF32 {};
```

**EN:** The preceding comment documents this block. This block declares `OpMultiplyAddComplexFastF32` as a lightweight `struct`, often used as a tag or thin wrapper.

**CN:** 前面的注释说明了这个代码块。该代码块将 `OpMultiplyAddComplexFastF32` 声明为一个轻量级 `struct`，通常用作标签类型或薄封装。

### Lines 89-91

```cpp
/// Tag indicating that staged accumulation is not to be used. This is valid only for SM89
/// FP8 kernels.
struct OpMultiplyAddFastAccum;
```

**EN:** The preceding comment documents this block. This block begins the definition of `OpMultiplyAddFastAccum`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `OpMultiplyAddFastAccum` 这个 `struct`，其成员会在后续代码中展开。

### Lines 95-96

```cpp
/// Tag indicating the complex multiply-add operation
struct OpMultiplyAddComplex {};
```

**EN:** The preceding comment documents this block. This block declares `OpMultiplyAddComplex` as a lightweight `struct`, often used as a tag or thin wrapper.

**CN:** 前面的注释说明了这个代码块。该代码块将 `OpMultiplyAddComplex` 声明为一个轻量级 `struct`，通常用作标签类型或薄封装。

### Lines 100-101

```cpp
/// Tag indicating the gaussian complex multiply-add operation
struct OpMultiplyAddGaussianComplex {};
```

**EN:** The preceding comment documents this block. This block declares `OpMultiplyAddGaussianComplex` as a lightweight `struct`, often used as a tag or thin wrapper.

**CN:** 前面的注释说明了这个代码块。该代码块将 `OpMultiplyAddGaussianComplex` 声明为一个轻量级 `struct`，通常用作标签类型或薄封装。

### Lines 105-106

```cpp
/// Tag indicating the inner product is defined by (XOR, POPC)
struct OpXorPopc {};
```

**EN:** The preceding comment documents this block. This block declares `OpXorPopc` as a lightweight `struct`, often used as a tag or thin wrapper.

**CN:** 前面的注释说明了这个代码块。该代码块将 `OpXorPopc` 声明为一个轻量级 `struct`，通常用作标签类型或薄封装。

### Lines 110-111

```cpp
/// Tag indicating the inner product is defined by (AND, POPC)
struct OpAndPopc {};
```

**EN:** The preceding comment documents this block. This block declares `OpAndPopc` as a lightweight `struct`, often used as a tag or thin wrapper.

**CN:** 前面的注释说明了这个代码块。该代码块将 `OpAndPopc` 声明为一个轻量级 `struct`，通常用作标签类型或薄封装。

### Lines 115-116

```cpp
/// Tag classifying math operators as thread-level operations.
struct OpClassSimt {};
```

**EN:** The preceding comment documents this block. This block declares `OpClassSimt` as a lightweight `struct`, often used as a tag or thin wrapper.

**CN:** 前面的注释说明了这个代码块。该代码块将 `OpClassSimt` 声明为一个轻量级 `struct`，通常用作标签类型或薄封装。

### Lines 120-121

```cpp
/// Tag classifying operators as Tensor Core operations.
struct OpClassTensorOp {};
```

**EN:** The preceding comment documents this block. This block declares `OpClassTensorOp` as a lightweight `struct`, often used as a tag or thin wrapper.

**CN:** 前面的注释说明了这个代码块。该代码块将 `OpClassTensorOp` 声明为一个轻量级 `struct`，通常用作标签类型或薄封装。

### Lines 123-125

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
/// Tag classifying operators as WMMA Tensor Core operations
struct OpClassWmmaTensorOp {};
```

**EN:** The preceding comment documents this block. This block declares `OpClassWmmaTensorOp` as a lightweight `struct`, often used as a tag or thin wrapper.

**CN:** 前面的注释说明了这个代码块。该代码块将 `OpClassWmmaTensorOp` 声明为一个轻量级 `struct`，通常用作标签类型或薄封装。

### Lines 129-130

```cpp
/// Tag classifying operators as Tensor Core with structure sparse operations.
struct OpClassSparseTensorOp {};
```

**EN:** The preceding comment documents this block. This block declares `OpClassSparseTensorOp` as a lightweight `struct`, often used as a tag or thin wrapper.

**CN:** 前面的注释说明了这个代码块。该代码块将 `OpClassSparseTensorOp` 声明为一个轻量级 `struct`，通常用作标签类型或薄封装。

### Lines 133-134

```cpp
/// Tag classifying operators as Tensor Core with blockScaled
struct OpClassBlockScaledTensorOp {};
```

**EN:** The preceding comment documents this block. This block declares `OpClassBlockScaledTensorOp` as a lightweight `struct`, often used as a tag or thin wrapper.

**CN:** 前面的注释说明了这个代码块。该代码块将 `OpClassBlockScaledTensorOp` 声明为一个轻量级 `struct`，通常用作标签类型或薄封装。

### Lines 136-137

```cpp
/// Tag classifying operators as Tensor Core with blockScaled structured sparse operations.
struct OpClassBlockScaledSparseTensorOp {};
```

**EN:** The preceding comment documents this block. This block declares `OpClassBlockScaledSparseTensorOp` as a lightweight `struct`, often used as a tag or thin wrapper.

**CN:** 前面的注释说明了这个代码块。该代码块将 `OpClassBlockScaledSparseTensorOp` 声明为一个轻量级 `struct`，通常用作标签类型或薄封装。

### Lines 141-162

```cpp
/// Matrix multiply-add operation
template <
  /// Size of the matrix product (concept: GemmShape)
  typename Shape_,
  /// Number of threads participating
  int kThreads_,
  /// Data type of A elements
  typename ElementA,
  /// Layout of A matrix (concept: MatrixLayout)
  typename LayoutA,
  /// Data type of B elements
  typename ElementB,
  /// Layout of B matrix (concept: MatrixLayout)
  typename LayoutB,
  /// Element type of C matrix
  typename ElementC,
  /// Layout of C matrix (concept: MatrixLayout)
  typename LayoutC,
  /// Inner product operator
  typename Operator
>
struct Mma;
```

**EN:** The preceding comment documents this block. This block begins the definition of `Mma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `Mma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 166-183

```cpp
/// Matrix multiply-add operation - specialized for 1x1x1x1 matrix multiply operation
template <
  /// Data type of A elements
  typename ElementA,
  /// Layout of A matrix (concept: MatrixLayout)
  typename LayoutA,
  /// Data type of B elements
  typename ElementB,
  /// Layout of B matrix (concept: MatrixLayout)
  typename LayoutB,
  /// Element type of C matrix
  typename ElementC_,
  /// Layout of C matrix (concept: MatrixLayout)
  typename LayoutC,
  /// Inner product operator
  typename Operator_
>
struct Mma<gemm::GemmShape<1, 1, 1>, 1, ElementA, LayoutA, ElementB, LayoutB, ElementC_, LayoutC, Operator_> {
```

**EN:** The preceding comment documents this block. This block begins the definition of `Mma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `Mma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 185-185

```cpp
  using Shape = gemm::GemmShape<1, 1, 1>;
```

**EN:** This alias defines `Shape` as `gemm::GemmShape<1, 1, 1>`, shortening later template or member declarations.

**CN:** 这里把 `Shape` 定义为 `gemm::GemmShape<1, 1, 1>` 的别名，以简化后续模板或成员声明。

### Lines 186-186

```cpp
  using Operator = Operator_;
```

**EN:** This alias defines `Operator` as `Operator_`, shortening later template or member declarations.

**CN:** 这里把 `Operator` 定义为 `Operator_` 的别名，以简化后续模板或成员声明。

### Lines 187-187

```cpp
  using ElementC = ElementC_;
```

**EN:** This alias defines `ElementC` as `ElementC_`, shortening later template or member declarations.

**CN:** 这里把 `ElementC` 定义为 `ElementC_` 的别名，以简化后续模板或成员声明。

### Lines 189-200

```cpp
  CUTLASS_HOST_DEVICE
  void operator()(
    Array<ElementC, 1> &d,
    Array<ElementA, 1> const &a,
    Array<ElementB, 1> const &b,
    Array<ElementC, 1> const &c
  ) {

    multiply_add<ElementA, ElementB, ElementC> op;

    d[0] = op(a[0], b[0], c[0]);
  }
```

**EN:** The function `op` implements a concrete operation in this abstraction. It packages a focused unit of behavior for the surrounding class, specialization, or namespace.

**CN:** `op` 函数实现了该抽象中的一个具体操作。它为周围的类、特化或命名空间封装了一个聚焦的行为单元。

### Lines 207-208

```cpp
/// Specifies internal data type for computation
struct SPFormatType {
```

**EN:** The preceding comment documents this block. This block begins the definition of `SPFormatType`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `SPFormatType` 这个 `struct`，其成员会在后续代码中展开。

### Lines 209-209

```cpp
  enum Kind {
```

**EN:** This block begins the definition of `Kind`, a `enum` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `Kind` 这个 `enum`，其成员会在后续代码中展开。

### Lines 210-211

```cpp
    Thread
  };
```

**EN:** This statement contributes an implementation detail to the surrounding abstraction.

**CN:** 这个语句为周围抽象补充了一个实现细节。

### Lines 216-239

```cpp
/// Matrix multiply-add operation
template <
  /// Size of the matrix product (concept: GemmShape)
  typename Shape_,
  /// Number of threads participating
  int kThreads_,
  /// Data type of A elements
  typename ElementA,
  /// Layout of A matrix (concept: MatrixLayout)
  typename LayoutA,
  /// Data type of B elements
  typename ElementB,
  /// Layout of B matrix (concept: MatrixLayout)
  typename LayoutB,
  /// Element type of C matrix
  typename ElementC,
  /// Layout of C matrix (concept: MatrixLayout)
  typename LayoutC,
  /// Inner product operator
  typename Operator,
  /// Specifies meta data format
  SPFormatType::Kind SPFormat = SPFormatType::Thread
>
struct SparseMma;
```

**EN:** The preceding comment documents this block. This block begins the definition of `SparseMma`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `SparseMma` 这个 `struct`，其成员会在后续代码中展开。

### Lines 246-248

```cpp
//
// Specializations for each compute capability
//
```

**EN:** This comment block separates sections or adds explanatory notes for the nearby code.

**CN:** 这个注释块用于划分章节，或为附近代码补充说明。

### Lines 250-259

```cpp
#include "cutlass/arch/mma_sm50.h"
#include "cutlass/arch/mma_sm60.h"
#include "cutlass/arch/mma_sm61.h"
#include "cutlass/arch/mma_sm70.h"
#include "cutlass/arch/mma_sm75.h"
#include "cutlass/arch/mma_sm80.h"
#include "cutlass/arch/mma_sparse_sm80.h"
#include "cutlass/arch/mma_sm89.h"
#include "cutlass/arch/mma_sparse_sm89.h"
#include "cutlass/arch/mma_sm90.h"
```

**EN:** This block imports dependencies such as `cutlass/arch/mma_sm50.h`, `cutlass/arch/mma_sm60.h`, `cutlass/arch/mma_sm61.h`, `cutlass/arch/mma_sm70.h`, `cutlass/arch/mma_sm75.h`, `cutlass/arch/mma_sm80.h`, `cutlass/arch/mma_sparse_sm80.h`, `cutlass/arch/mma_sm89.h`, and 2 more headers, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cutlass/arch/mma_sm50.h`, `cutlass/arch/mma_sm60.h`, `cutlass/arch/mma_sm61.h`, `cutlass/arch/mma_sm70.h`, `cutlass/arch/mma_sm75.h`, `cutlass/arch/mma_sm80.h`, `cutlass/arch/mma_sparse_sm80.h`, `cutlass/arch/mma_sm89.h`, and 2 more headers 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 260-261

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
namespace cutlass {
```

**EN:** The preceding comment documents this block. This block opens the namespace scope `cutlass` for the declarations that follow.

**CN:** 前面的注释说明了这个代码块。该代码块打开了 `cutlass` 命名空间作用域，以容纳后续声明。

### Lines 263-264

```cpp
namespace arch {
namespace detail {
```

**EN:** This block opens the namespace scope `arch::detail` for the declarations that follow.

**CN:** 该代码块打开了 `arch::detail` 命名空间作用域，以容纳后续声明。

### Lines 265-267

```cpp
/// Helper for determining whether staged accumulation should be used for a given operator
template <typename Operator>
struct UseStagedAccumulation {
```

**EN:** The preceding comment documents this block. This block begins the definition of `UseStagedAccumulation`, a `struct` whose members are laid out in the following lines.

**CN:** 前面的注释说明了这个代码块。该代码块开始定义 `UseStagedAccumulation` 这个 `struct`，其成员会在后续代码中展开。

### Lines 268-270

```cpp
  static bool const value = platform::is_same<typename Operator::MathOperator, OpMultiplyAddFastF32>::value ||
                            platform::is_same<typename Operator::MathOperator, OpMultiplyAddComplexFastF32>::value ||
                            is_sm89_staged_policy_v<Operator>;
```

**EN:** This declaration defines `value` and assigns it the compile-time expression `platform::is_same<typename Operator::MathOperator, OpMultiplyAddFastF32>::value || platform::is_same<typename Operator::MathOperator, OpMultiplyAddComplexFastF32>::value || is_sm89_staged_policy_v<Operator>`.

**CN:** 这个声明定义了 `value`，并把它设为编译期表达式 `platform::is_same<typename Operator::MathOperator, OpMultiplyAddFastF32>::value || platform::is_same<typename Operator::MathOperator, OpMultiplyAddComplexFastF32>::value || is_sm89_staged_policy_v<Operator>`。

## Key Concepts / 关键概念

- **EN:** Heavy use of templates enables compile-time specialization and zero-overhead abstractions.
  **CN:** 大量使用模板，使该文件能够进行编译期特化并保持零额外开销的抽象。

- **EN:** Architecture-specific paths map C++ wrappers onto GPU instructions or micro-architectural features.
  **CN:** 架构特定路径会把 C++ 封装映射到 GPU 指令或底层微架构特性。

- **EN:** Fixed-size containers and fragments are used to model register or shared-memory tiles.
  **CN:** 定长容器与片段类型用于建模寄存器块或共享内存块。

- **EN:** Type traits and compile-time checks constrain valid instantiations.
  **CN:** 类型萃取与编译期检查用于约束合法的模板实例化。

- **EN:** Conditional compilation keeps the header portable across host, device, CUDA, and SYCL builds.
  **CN:** 条件编译使该头文件能够在主机端、设备端、CUDA 与 SYCL 构建之间保持可移植性。

## Dependencies / 依赖关系

- **EN:** Direct includes: `cutlass/array.h`, `cutlass/numeric_types.h`, `cutlass/functional.h`, `cutlass/gemm/gemm.h`, `cutlass/arch/arch.h`, `cutlass/arch/mma_sm50.h`, `cutlass/arch/mma_sm60.h`, `cutlass/arch/mma_sm61.h`, `cutlass/arch/mma_sm70.h`, `cutlass/arch/mma_sm75.h` (+5 more).
  **CN:** 直接包含：`cutlass/array.h`, `cutlass/numeric_types.h`, `cutlass/functional.h`, `cutlass/gemm/gemm.h`, `cutlass/arch/arch.h`, `cutlass/arch/mma_sm50.h`, `cutlass/arch/mma_sm60.h`, `cutlass/arch/mma_sm61.h`, `cutlass/arch/mma_sm70.h`, `cutlass/arch/mma_sm75.h` (+5 more)。

- **EN:** Primary namespaces: `cutlass`, `arch`, `detail`.
  **CN:** 主要命名空间：`cutlass`, `arch`, `detail`。

- **EN:** Important macros or compile flags: `CUTLASS_HOST_DEVICE`.
  **CN:** 重要宏或编译开关：`CUTLASS_HOST_DEVICE`。
