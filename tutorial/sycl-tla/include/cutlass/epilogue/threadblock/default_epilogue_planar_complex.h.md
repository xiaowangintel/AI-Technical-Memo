# default_epilogue_planar_complex.h — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/epilogue/threadblock/default_epilogue_planar_complex.h`

- **Purpose (EN):** Constructs a default epilogue for planar complex outputs. This template reuses components for real-valued epilogues and applies them to planar complex output matrices.

- **作用 (CN):** 为特定 epilogue 后端提供默认的 `default epilogue planar complex` 组合规则。


## Line-by-Line Analysis / 逐行分析

### Lines 1-32

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
/*! \file
  \brief Constructs a default epilogue for planar complex outputs.
```

**EN:** Carries the BSD-3-Clause license notice and the file-level documentation that tells readers what this header contributes.

**CN:** 这一段包含 BSD-3-Clause 许可声明以及文件级说明，帮助读者先理解该头文件的职责。


### Lines 34-35

```cpp
  This template reuses components for real-valued epilogues and applies them to planar complex
  output matrices.
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 37

```cpp
*/
```

**EN:** This block contributes supporting declarations or implementation details needed by the surrounding epilogue component.

**CN:** 该代码块提供外围 epilogue 组件所需的辅助声明或实现细节。


### Line 39

```cpp
#pragma once
```

**EN:** Uses `#pragma once` so the header can be included multiple times safely in large template instantiations.

**CN:** 使用 `#pragma once` 让该头文件在大型模板实例化过程中被重复包含时仍然安全。


### Lines 41-44

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/numeric_types.h"
#include "cutlass/array.h"
#include "cutlass/array_planar_complex.h"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/cutlass.h`, `cutlass/numeric_types.h`, `cutlass/array.h`, `cutlass/array_planar_complex.h`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/cutlass.h`，`cutlass/numeric_types.h`，`cutlass/array.h`，`cutlass/array_planar_complex.h`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Line 46

```cpp
#include "cutlass/arch/arch.h"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/arch/arch.h`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/arch/arch.h`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Lines 48-51

```cpp
#include "cutlass/epilogue/thread/linear_combination_planar_complex.h"
#include "cutlass/epilogue/threadblock/default_epilogue_simt.h"
#include "cutlass/epilogue/threadblock/default_epilogue_volta_tensor_op.h"
#include "cutlass/epilogue/threadblock/default_epilogue_tensor_op.h"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/epilogue/thread/linear_combination_planar_complex.h`, `cutlass/epilogue/threadblock/default_epilogue_simt.h`, `cutlass/epilogue/threadblock/default_epilogue_volta_tensor_op.h`, `cutlass/epilogue/threadblock/default_epilogue_tensor_op.h`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/epilogue/thread/linear_combination_planar_complex.h`，`cutlass/epilogue/threadblock/default_epilogue_simt.h`，`cutlass/epilogue/threadblock/default_epilogue_volta_tensor_op.h`，`cutlass/epilogue/threadblock/default_epilogue_tensor_op.h`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Line 53

```cpp
#include "cutlass/epilogue/threadblock/epilogue_planar_complex.h"
```

**EN:** This include block pulls in the direct dependencies for the file, such as `cutlass/epilogue/threadblock/epilogue_planar_complex.h`. These headers supply the core CUTLASS types, iterator utilities, math helpers, or fusion abstractions used below.

**CN:** 这一组 `#include` 引入了文件的直接依赖，例如 `cutlass/epilogue/threadblock/epilogue_planar_complex.h`。这些头文件为下方实现提供核心 CUTLASS 类型、迭代器工具、数学辅助函数或融合抽象。


### Lines 57-59

```cpp
namespace cutlass {
namespace epilogue {
namespace threadblock {
```

**EN:** Opens the namespace hierarchy so the following declarations live in the intended CUTLASS epilogue scope.

**CN:** 打开命名空间层级，使后续声明落在预期的 CUTLASS epilogue 作用域中。


### Lines 63-73

```cpp
/// Defines sensible defaults for epilogues.
template <
  typename ThreadblockShape_,
  typename WarpMma_,
  typename OpcodeClass_,
  typename ArchTag_,
  int PartitionsK,
  typename OutputOp_,
  int ElementsPerAccess
>
struct DefaultEpiloguePlanarComplex;
```

**EN:** Declares the templated `DefaultEpiloguePlanarComplex` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: Defines sensible defaults for epilogues.

**CN:** 声明模板类型 `DefaultEpiloguePlanarComplex`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 77-92

```cpp
/// Defines sensible defaults for epilogues.
template <
  typename ThreadblockShape_,
  typename WarpMmaOperator_,
  int PartitionsK,
  typename OutputOp_,
  int ElementsPerAccess
>
struct DefaultEpiloguePlanarComplex<
  ThreadblockShape_, 
  WarpMmaOperator_, 
  arch::OpClassTensorOp, 
  arch::Sm70,
  PartitionsK, 
  OutputOp_, 
  ElementsPerAccess> {
```

**EN:** Declares the templated `DefaultEpiloguePlanarComplex` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: Defines sensible defaults for epilogues.

**CN:** 声明模板类型 `DefaultEpiloguePlanarComplex`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 94-100

```cpp
  using RealEpilogue = DefaultEpilogueVoltaTensorOp<
    ThreadblockShape_,
    WarpMmaOperator_,
    PartitionsK,
    OutputOp_,
    ElementsPerAccess
  >;
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 102-113

```cpp
  using Epilogue = EpiloguePlanarComplex<
    ThreadblockShape_,
    WarpMmaOperator_,
    PartitionsK,
    typename RealEpilogue::OutputTileIterator,
    typename RealEpilogue::AccumulatorFragmentIterator,
    typename RealEpilogue::WarpTileIterator,
    typename RealEpilogue::SharedLoadIterator,
    OutputOp_,
    typename RealEpilogue::Padding
  >;
};
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 117-132

```cpp
/// Defines sensible defaults for epilogues.
template <
  typename ThreadblockShape_,
  typename WarpMmaOperator_,
  int PartitionsK,
  typename OutputOp_,
  int ElementsPerAccess
>
struct DefaultEpiloguePlanarComplex<
  ThreadblockShape_, 
  WarpMmaOperator_, 
  arch::OpClassTensorOp, 
  arch::Sm75,
  PartitionsK, 
  OutputOp_, 
  ElementsPerAccess> {
```

**EN:** Declares the templated `DefaultEpiloguePlanarComplex` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: Defines sensible defaults for epilogues.

**CN:** 声明模板类型 `DefaultEpiloguePlanarComplex`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 134-140

```cpp
  using RealEpilogue = DefaultEpilogueTensorOp<
    ThreadblockShape_,
    WarpMmaOperator_,
    PartitionsK,
    OutputOp_,
    ElementsPerAccess
  >;
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 142-153

```cpp
  using Epilogue = EpiloguePlanarComplex<
    ThreadblockShape_,
    WarpMmaOperator_,
    PartitionsK,
    typename RealEpilogue::OutputTileIterator,
    typename RealEpilogue::AccumulatorFragmentIterator,
    typename RealEpilogue::WarpTileIterator,
    typename RealEpilogue::SharedLoadIterator,
    OutputOp_,
    typename RealEpilogue::Padding
  >;
};
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 157-172

```cpp
/// Defines sensible defaults for epilogues.
template <
  typename ThreadblockShape_,
  typename WarpMmaOperator_,
  int PartitionsK,
  typename OutputOp_,
  int ElementsPerAccess
>
struct DefaultEpiloguePlanarComplex<
  ThreadblockShape_, 
  WarpMmaOperator_, 
  arch::OpClassTensorOp, 
  arch::Sm80,
  PartitionsK, 
  OutputOp_, 
  ElementsPerAccess> {
```

**EN:** Declares the templated `DefaultEpiloguePlanarComplex` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: Defines sensible defaults for epilogues.

**CN:** 声明模板类型 `DefaultEpiloguePlanarComplex`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 174-180

```cpp
  using RealEpilogue = DefaultEpilogueTensorOp<
    ThreadblockShape_,
    WarpMmaOperator_,
    PartitionsK,
    OutputOp_,
    ElementsPerAccess
  >;
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 182-193

```cpp
  using Epilogue = EpiloguePlanarComplex<
    ThreadblockShape_,
    WarpMmaOperator_,
    PartitionsK,
    typename RealEpilogue::OutputTileIterator,
    typename RealEpilogue::AccumulatorFragmentIterator,
    typename RealEpilogue::WarpTileIterator,
    typename RealEpilogue::SharedLoadIterator,
    OutputOp_,
    typename RealEpilogue::Padding
  >;
};
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 197-213

```cpp
/// Defines sensible defaults for epilogues.
template <
  typename ThreadblockShape_,
  typename WarpMmaOperator_,
  typename ArchTag_,
  int PartitionsK,
  typename OutputOp_,
  int ElementsPerAccess
>
struct DefaultEpiloguePlanarComplex<
  ThreadblockShape_, 
  WarpMmaOperator_, 
  arch::OpClassSimt, 
  ArchTag_,
  PartitionsK, 
  OutputOp_, 
  ElementsPerAccess> {
```

**EN:** Declares the templated `DefaultEpiloguePlanarComplex` type. The template parameters let this header specialize behavior for data types, tile shapes, layouts, and architecture tags at compile time. The nearby comment frames it as: Defines sensible defaults for epilogues.

**CN:** 声明模板类型 `DefaultEpiloguePlanarComplex`。这些模板参数让该头文件能够在编译期针对数据类型、tile 形状、布局和架构标签进行特化。


### Lines 215-220

```cpp
  using RealEpilogue = DefaultEpilogueSimt<
    ThreadblockShape_,
    WarpMmaOperator_,
    OutputOp_,
    ElementsPerAccess
  >;
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


### Lines 222-233

```cpp
  using Epilogue = EpiloguePlanarComplex<
    ThreadblockShape_,
    WarpMmaOperator_,
    PartitionsK,
    typename RealEpilogue::OutputTileIterator,
    typename RealEpilogue::AccumulatorFragmentIterator,
    typename RealEpilogue::WarpTileIterator,
    typename RealEpilogue::SharedLoadIterator,
    OutputOp_,
    typename RealEpilogue::Padding
  >;
};
```

**EN:** This alias block gives concise names to template-dependent shapes, layouts, fragments, and iterator types used throughout the implementation.

**CN:** 该别名代码块为实现中反复使用的形状、布局、片段和迭代器类型提供简洁名称。


## Key Concepts / 关键概念

- **Threadblock staging / 线程块级暂存:** Organizes shared-memory staging, tile iterators, and output traversal for threadblock-scope epilogues. / 为线程块范围的 epilogue 组织共享内存暂存、tile 迭代器和输出遍历。

- **Tensor-op path / Tensor-op 路径:** Matches the epilogue layout and iterator strategy to tensor-core style MMA pipelines when applicable. / 在适用时让 epilogue 的布局和迭代策略与 tensor core 风格的 MMA 流水线相匹配。


## Dependencies / 依赖关系

- **Direct includes / 直接包含:** `cutlass/cutlass.h`, `cutlass/numeric_types.h`, `cutlass/array.h`, `cutlass/array_planar_complex.h`, `cutlass/arch/arch.h`, `cutlass/epilogue/thread/linear_combination_planar_complex.h`, `cutlass/epilogue/threadblock/default_epilogue_simt.h`, `cutlass/epilogue/threadblock/default_epilogue_volta_tensor_op.h`, `cutlass/epilogue/threadblock/default_epilogue_tensor_op.h`, `cutlass/epilogue/threadblock/epilogue_planar_complex.h`

- **Primary namespace / 主要命名空间:** `cutlass::epilogue::threadblock`

- **Related epilogue headers / 相关 epilogue 头文件:** `cutlass/epilogue/thread/linear_combination_planar_complex.h`, `cutlass/epilogue/threadblock/default_epilogue_simt.h`, `cutlass/epilogue/threadblock/default_epilogue_volta_tensor_op.h`, `cutlass/epilogue/threadblock/default_epilogue_tensor_op.h`, `cutlass/epilogue/threadblock/epilogue_planar_complex.h`
