# b2b_implicit_gemm_pipelined.h — Code Analysis / 代码分析

**Source / 源文件**: `examples/13_two_tensor_op_fusion/threadblock/b2b_implicit_gemm_pipelined.h`  
**Purpose / 用途**: Defines a pipelined threadblock operator for fused implicit-GEMM convolution. / 定义融合隐式 GEMM 卷积的流水化 threadblock 算子。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-31 / 第1-31行

```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
```

**EN**: This is the standard BSD-3-Clause license header. It documents redistribution terms before any executable code appears.  
**CN**: 这里是标准的 BSD-3-Clause 许可证头，在任何可执行代码之前先说明再分发条款。

### Lines 32-33 / 第32-33行

```cpp
    \brief Template for a double-buffered threadblock-scoped GEMM kernel.
*/
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 35-35 / 第35-35行

```cpp
#pragma once
```

**EN**: This pragma makes the header idempotent so repeated inclusion does not create duplicate definitions.  
**CN**: 这个 pragma 保证头文件只会被处理一次，避免重复包含产生重复定义。

### Lines 37-40 / 第37-40行

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/array.h"
#include "cutlass/aligned_buffer.h"
#include "cutlass/numeric_conversion.h"
```

**EN**: These headers pull in the building blocks required by this file. Core CUTLASS definitions, architecture tags, and status types. Provides `cutlass/array.h` so this file can use the related API or helper utilities. Provides `cutlass/aligned_buffer.h` so this file can use the related API or helper utilities. Provides `cutlass/numeric_conversion.h` so this file can use the related API or helper utilities.  
**CN**: 这些头文件引入了当前文件所需的构建模块。CUTLASS 的核心定义、架构标签与状态类型。提供 `cutlass/array.h`，使本文件能够使用相关 API 或辅助工具。提供 `cutlass/aligned_buffer.h`，使本文件能够使用相关 API 或辅助工具。提供 `cutlass/numeric_conversion.h`，使本文件能够使用相关 API 或辅助工具。

### Lines 42-43 / 第42-43行

```cpp
#include "cutlass/numeric_types.h"
#include "cutlass/matrix_shape.h"
```

**EN**: These headers pull in the building blocks required by this file. CUTLASS numeric scalar and packed numeric type definitions. Provides `cutlass/matrix_shape.h` so this file can use the related API or helper utilities.  
**CN**: 这些头文件引入了当前文件所需的构建模块。CUTLASS 数值标量与打包数值类型定义。提供 `cutlass/matrix_shape.h`，使本文件能够使用相关 API 或辅助工具。

### Lines 45-46 / 第45-46行

```cpp
#include "cutlass/gemm/gemm.h"
#include "cutlass/gemm/warp/mma_tensor_op_fragment_iterator.h"
```

**EN**: These headers pull in the building blocks required by this file. Provides `cutlass/gemm/gemm.h` so this file can use the related API or helper utilities. Provides `cutlass/gemm/warp/mma_tensor_op_fragment_iterator.h` so this file can use the related API or helper utilities.  
**CN**: 这些头文件引入了当前文件所需的构建模块。提供 `cutlass/gemm/gemm.h`，使本文件能够使用相关 API 或辅助工具。提供 `cutlass/gemm/warp/mma_tensor_op_fragment_iterator.h`，使本文件能够使用相关 API 或辅助工具。

### Lines 48-48 / 第48-48行

```cpp
#include "threadblock/b2b_mma_base.h"
```

**EN**: These headers pull in the building blocks required by this file. Example-local threadblock policy or pipeline component.  
**CN**: 这些头文件引入了当前文件所需的构建模块。示例本地 threadblock 策略或流水线组件。

### Lines 50-50 / 第50-50行

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 52-52 / 第52-52行

```cpp
namespace cutlass {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 53-53 / 第53-53行

```cpp
namespace conv {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 54-54 / 第54-54行

```cpp
namespace threadblock {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 56-56 / 第56-56行

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 58-58 / 第58-58行

```cpp
/// Structure to compute the matrix product targeting CUDA cores and SIMT math instructions.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 59-122 / 第59-122行

```cpp
template <
  /// Size of the Gemm problem - concept: gemm::GemmShape<>
  typename Shape0_,
  /// Iterates over tiles of A operand in global memory 
  //  (concept: ReadableTileIterator | ForwardTileIterator | MaskedTileIterator)
  typename IteratorA0_,
  /// Iterates over tiles of A operand in shared memory
  /// (concept: WriteableTileIterator | RandomAccessTileIterator)
  typename SmemIteratorA0_,
  /// Iterates over tiles of B operand in global memory
  //  (concept: ReadableTileIterator | ForwardTileIterator | MaskedTileIterator)
  typename IteratorB0_,
  /// Iterates over tiles of B operand in shared memory
  /// (concept: WriteableTileIterator | RandomAccessTileIterator)
  typename SmemIteratorB0_,
  /// Size of the Gemm problem - concept: gemm::GemmShape<>
  typename Shape1_,
  /// Iterates over the intermediate accumulator tile
  //  (concept::MmaTensorOpFragmentIterator) 
  typename FragmentIteratorA1_,
  /// Iterates over vectors of scale and bias vector in global memory
  //  (concept: VectorIterator)
  typename IteratorAccumulatorScaleBias_,
  /// FragmentIterator to load Scale or Bias vector from threadblock fragment
  typename FragmentIteratorA1ScaleBias_,
  //  (concept: VectorFragmentIterator) 
  /// Iterates over tiles of B operand in global memory
  //  (concept: ReadableTileIterator | ForwardTileIterator | MaskedTileIterator)
  typename IteratorB1_,
  /// Iterates over tiles of B operand in shared memory
  /// (concept: WriteableTileIterator | RandomAccessTileIterator)
  typename SmemIteratorB1_,
  /// Data type of accumulator matrix
  typename ElementC_,
  /// Data type of accumulator matrix
  typename LayoutC_,
  /// Output operator for 1st Gemm(concept: epilogue::thread::LinearCombinationClamp, etc...) 
  typename OutputOp_,
  /// Policy describing tuning details (concept: MmaPolicy)
  typename Policy0_,
  /// Policy describing tuning details (concept: MmaPolicy)
  typename Policy1_,
  /// Transformation applied to A operand
  typename TransformA0_ = NumericArrayConverter<
    typename SmemIteratorA0_::Element, 
    typename IteratorA0_::Element, 
    IteratorA0_::Fragment::kElements>,
  ///
  /// Transformation applied to B operand
  typename TransformB0_ = NumericArrayConverter<
    typename SmemIteratorB0_::Element, 
    typename IteratorB0_::Element, 
    IteratorB0_::Fragment::kElements>,
  ///
  /// Transformation applied to B operand
  typename TransformB1_ = NumericArrayConverter<
    typename SmemIteratorB1_::Element, 
    typename IteratorB1_::Element, 
    IteratorB1_::Fragment::kElements>,
  /// Used for partial specialization
  typename Enable = bool
>
class B2bImplicitGemmPipelined : 
  public gemm::threadblock::B2bMmaBase<Shape0_, Shape1_, Policy0_, Policy1_, 2> {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates. `GemmShape<M, N, K>` encodes tile extents along the GEMM M/N/K axes. `LinearCombination` is CUTLASS's standard epilogue functor for scaling accumulators and combining them with existing outputs. Implicit GEMM converts convolution into a GEMM-like tiled traversal without materializing the lowered matrix explicitly. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。 `GemmShape<M, N, K>` 用于描述 GEMM 在 M/N/K 三个维度上的分块大小。 `LinearCombination` 是 CUTLASS 标准 epilogue functor，用于缩放累加器并与已有输出组合。 隐式 GEMM 会把卷积转换为类似 GEMM 的分块遍历，而不显式生成展开后的矩阵。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 123-123 / 第123-123行

```cpp
public:
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 125-125 / 第125-125行

```cpp
  ///< Base class
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 126-126 / 第126-126行

```cpp
  using Base = gemm::threadblock::B2bMmaBase<Shape0_, Shape1_, Policy0_, Policy1_, 2>;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。

### Lines 128-131 / 第128-131行

```cpp
  using Shape0 = Shape0_;             ///< Size of the Gemm problem - concept: gemm::GemmShape<>
  using IteratorA0 = IteratorA0_;     ///< Iterates over tiles of A operand in global memory
  using IteratorB0 = IteratorB0_;     ///< Iterates over tiles of B operand in global memory
  using Policy0 = Policy0_;           ///< Policy0 describing tuning details
```

**EN**: These aliases describe CUTLASS's tiling hierarchy. The code separately chooses CTA, warp, and instruction shapes so the kernel matches the hardware execution pattern it targets. `GemmShape<M, N, K>` encodes tile extents along the GEMM M/N/K axes.  
**CN**: 这些别名描述了 CUTLASS 的分块层次。代码分别选择 CTA、warp 与指令级分块，以匹配目标硬件的执行模式。 `GemmShape<M, N, K>` 用于描述 GEMM 在 M/N/K 三个维度上的分块大小。

### Lines 133-134 / 第133-134行

```cpp
  using SmemIteratorA0 = SmemIteratorA0_;
  using SmemIteratorB0 = SmemIteratorB0_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 136-142 / 第136-142行

```cpp
  using Shape1 = Shape1_;             ///< Size of the Gemm problem - concept: gemm::GemmShape<>
  using FragmentIteratorA1 = FragmentIteratorA1_;     ///< Iterates over tiles of A1 operand from accumulator tile
  using IteratorAccumulatorScaleBias = IteratorAccumulatorScaleBias_;   ///< Iterates over tiles of the scale and bias vectors in global memory
  using FragmentIteratorA1ScaleBias = 
    FragmentIteratorA1ScaleBias_;     ///< WarpIterator to load Scale or Bias vector from the threadblock fragment
  using IteratorB1 = IteratorB1_;     ///< Iterates over tiles of B operand in global memory
  using Policy1 = Policy1_;           ///< Policy1 describing tuning details
```

**EN**: These aliases describe CUTLASS's tiling hierarchy. The code separately chooses CTA, warp, and instruction shapes so the kernel matches the hardware execution pattern it targets. `GemmShape<M, N, K>` encodes tile extents along the GEMM M/N/K axes.  
**CN**: 这些别名描述了 CUTLASS 的分块层次。代码分别选择 CTA、warp 与指令级分块，以匹配目标硬件的执行模式。 `GemmShape<M, N, K>` 用于描述 GEMM 在 M/N/K 三个维度上的分块大小。

### Lines 144-144 / 第144-144行

```cpp
  using SmemIteratorB1 = SmemIteratorB1_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 147-148 / 第147-148行

```cpp
  using ElementC = ElementC_;       ///< Data type of accumulator matrix
  using LayoutC = LayoutC_;         ///< Layout of accumulator matrix
```

**EN**: These aliases pin down the scalar types used for inputs, accumulators, and outputs. Choosing them explicitly matters because CUTLASS can compute in a different precision from storage.  
**CN**: 这些别名固定了输入、累加器和输出使用的标量类型。显式指定它们很重要，因为 CUTLASS 可以让计算精度与存储精度不同。

### Lines 150-150 / 第150-150行

```cpp
  using OutputOp = OutputOp_;       ///< Epilogue after 1st Gemm
```

**EN**: This block defines epilogue functors. In CUTLASS, the epilogue converts accumulators into the output type and can fuse extra work such as scaling, bias handling, clamping, or activation.  
**CN**: 这段代码定义了 epilogue functor。在 CUTLASS 中，epilogue 负责把累加器转换为输出类型，并可融合缩放、偏置处理、截断或激活等附加工作。

### Lines 152-153 / 第152-153行

```cpp
  static const bool PerChannelScale = (OutputOp::kScale ==
      epilogue::thread::ScaleType::OnlyAlphaPerChannelScaling);
```

**EN**: This block defines compile-time constants or static values that parameterize the kernel or the benchmark.  
**CN**: 这段代码定义编译期常量或静态值，用于参数化内核或基准测试。

### Lines 155-157 / 第155-157行

```cpp
  using TransformA0 = TransformA0_;
  using TransformB0 = TransformB0_;
  using TransformB1 = TransformB1_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 159-161 / 第159-161行

```cpp
  //
  // Dependent types
  //
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 163-163 / 第163-163行

```cpp
  /// Fragment of operand A loaded from global memory
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 164-164 / 第164-164行

```cpp
  using FragmentA0 = typename IteratorA0::Fragment;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 166-166 / 第166-166行

```cpp
  /// Fragment of operand B loaded from global memory
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 167-167 / 第167-167行

```cpp
  using FragmentB0 = typename IteratorB0::Fragment;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 169-169 / 第169-169行

```cpp
  /// Fragment of accumulator tile
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 170-170 / 第170-170行

```cpp
  using FragmentC0 = typename Policy0::Operator::FragmentC;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 172-172 / 第172-172行

```cpp
  /// Warp-level Mma
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 173-173 / 第173-173行

```cpp
  using Operator0 = typename Policy0::Operator;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 175-175 / 第175-175行

```cpp
  /// Fragment of Scale and Bias loaded from global memory
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 176-176 / 第176-176行

```cpp
  using FragmentA1ScaleBias = typename IteratorAccumulatorScaleBias::Fragment;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 178-178 / 第178-178行

```cpp
  /// Fragment of operand B loaded from global memory
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 179-179 / 第179-179行

```cpp
  using FragmentB1 = typename IteratorB1::Fragment;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 181-181 / 第181-181行

```cpp
  /// Fragment of accumulator tile
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 182-182 / 第182-182行

```cpp
  using FragmentC1 = typename Policy1::Operator::FragmentC;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 184-184 / 第184-184行

```cpp
  /// Warp-level Mma
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 185-185 / 第185-185行

```cpp
  using Operator1 = typename Policy1::Operator;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 187-187 / 第187-187行

```cpp
  /// Obtain the arch tag from the warp-level operator
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 188-188 / 第188-188行

```cpp
  using ArchTag = typename Policy0::Operator::ArchTag;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 190-190 / 第190-190行

```cpp
  /// Complex transform on A0 operand
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 191-191 / 第191-191行

```cpp
  static ComplexTransform const kTransformA0 = Operator0::kTransformA;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 193-193 / 第193-193行

```cpp
  /// Complex transform on B0 operand
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 194-194 / 第194-194行

```cpp
  static ComplexTransform const kTransformB0 = Operator0::kTransformB;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 196-196 / 第196-196行

```cpp
  /// Complex transform on B1 operand
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 197-197 / 第197-197行

```cpp
  static ComplexTransform const kTransformB1 = Operator1::kTransformB;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 199-199 / 第199-199行

```cpp
  // staticaly assert kStages for MmaPipelined is two (Double-buffered pipeline)
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 200-200 / 第200-200行

```cpp
  static_assert((Base::kStages==2), "MmaPipelined requires kStages set to value 2");
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 202-202 / 第202-202行

```cpp
private:
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 204-205 / 第204-205行

```cpp
  using WarpFragmentA0 = typename Operator0::FragmentA;
  using WarpFragmentB0 = typename Operator0::FragmentB;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 206-206 / 第206-206行

```cpp
  /// Warp Fragment of operand A1 loaded from accmulator tile
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 207-207 / 第207-207行

```cpp
  using WarpFragmentA1 = typename FragmentIteratorA1::Fragment;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 208-208 / 第208-208行

```cpp
  /// Warp Fragment of operand A1 scale and bias loaded from threadblock fragment
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 209-211 / 第209-211行

```cpp
  using WarpFragmentA1ScaleBias =
      typename FragmentIteratorA1ScaleBias::Fragment;
  using WarpFragmentB1 = typename Operator1::FragmentB;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 213-213 / 第213-213行

```cpp
protected:
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 215-215 / 第215-215行

```cpp
  /// Iterator to write threadblock-scoped tile of A operand to shared memory
```

**EN**: This comment separates sections and documents the purpose of the code that follows. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 216-216 / 第216-216行

```cpp
  SmemIteratorA0 smem_iterator_A_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 218-218 / 第218-218行

```cpp
  /// Iterator to write threadblock-scoped tile of B0 operand to shared memory
```

**EN**: This comment separates sections and documents the purpose of the code that follows. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 219-219 / 第219-219行

```cpp
  SmemIteratorB0 smem_iterator_B0_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 221-221 / 第221-221行

```cpp
  /// Iterator to write threadblock-scoped tile of B1 operand to shared memory
```

**EN**: This comment separates sections and documents the purpose of the code that follows. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 222-222 / 第222-222行

```cpp
  SmemIteratorB1 smem_iterator_B1_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 224-224 / 第224-224行

```cpp
public:
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 226-226 / 第226-226行

```cpp
  /// Construct from tensor references
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 227-254 / 第227-254行

```cpp
  CUTLASS_DEVICE
  B2bImplicitGemmPipelined(
    typename Base::B2bMmaSharedStorage &shared_storage, ///< Shared storage needed for internal use by threadblock-scoped GEMM
    int thread_idx,                                     ///< ID within the threadblock
    int warp_idx,                                       ///< ID of warp
    int lane_idx                                        ///< ID of each thread within a warp
  ):
    Base(shared_storage, thread_idx, warp_idx, lane_idx),
    smem_iterator_A_(shared_storage.shared_storage0.operand_A_ref(), thread_idx),
    smem_iterator_B0_(shared_storage.shared_storage0.operand_B_ref(), thread_idx),
    smem_iterator_B1_(shared_storage.shared_storage1.operand_B_ref(), thread_idx) {
    // Compute warp location within threadblock tile by mapping the warp_id to
    // three coordinates:
    //   _m: the warp's position within the threadblock along the M dimension
    //   _n: the warp's position within the threadblock along the N dimension
    //   _k: the warp's position within the threadblock along the K dimension
    int warp_idx_mn = warp_idx % (Base::WarpCount0::kM * Base::WarpCount0::kN);
    int warp_idx_k = warp_idx / (Base::WarpCount0::kM * Base::WarpCount0::kN);
    int warp_idx_m = warp_idx_mn % Base::WarpCount0::kM;
    int warp_idx_n = warp_idx_mn / Base::WarpCount0::kM;
    //These may change across different GEMM layers
    int tile_offset_k_0 = Base::kWarpGemmIterations0 * warp_idx_k;
    int tile_offset_k_1 = Base::kWarpGemmIterations1 * warp_idx_k;
    // Add per-warp offsets in units of warp-level tiles
    this->warp_tile_iterator_A0_.add_tile_offset({warp_idx_m, tile_offset_k_0});
    this->warp_tile_iterator_B0_.add_tile_offset({tile_offset_k_0, warp_idx_n});
    this->warp_tile_iterator_B1_.add_tile_offset({tile_offset_k_1, warp_idx_n});
  }
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps. Implicit GEMM converts convolution into a GEMM-like tiled traversal without materializing the lowered matrix explicitly.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。 隐式 GEMM 会把卷积转换为类似 GEMM 的分块遍历，而不显式生成展开后的矩阵。

### Lines 262-262 / 第262-262行

```cpp
  /// Perform a threadblock-scoped matrix multiply-accumulate
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 263-467 / 第263-467行

```cpp
  CUTLASS_DEVICE
  void operator()(
    int gemm_k_iterations_0,                           ///< number of iterations of the mainloop
    FragmentC1 &accum,                                 ///< destination accumulator tile
    IteratorA0 iterator_A,                             ///< iterator over A operand in global memory
    IteratorB0 iterator_B0,                            ///< iterator over B0 operand in global memory
    IteratorAccumulatorScaleBias iterator_A1_scale,    ///< iterator over A1 operand scale vectors in global memory
    IteratorAccumulatorScaleBias iterator_A1_bias,     ///< iterator over A1 operand bias vectors in global memory
    IteratorB1 iterator_B1,                            ///< iterator over B1 operand in global memory
    FragmentC0 const &src_accum,                       ///< source accumulator tile
    OutputOp output_op_0,                              ///< epilogue operation after 1st Gemm
    TransformA0 transform_A0 = TransformA0(),            ///< transformation applied to A0 fragment
    TransformB0 transform_B0 = TransformB0(),           ///< transformation applied to B0 fragment
    TransformB1 transform_B1 = TransformB1()) {         ///< transformation applied to B1 fragment
    //
    // Prologue
    //
    // Perform accumulation in the 'd' output operand
    FragmentC0 accum0 = src_accum;
    FragmentA0 tb_frag_A;
    FragmentB0 tb_frag_B0;
    tb_frag_A.clear();
    tb_frag_B0.clear();
    // The last kblock is loaded in the prolog
    iterator_A.load(tb_frag_A);
    iterator_B0.load(tb_frag_B0);
    ++iterator_A;
    ++iterator_B0;
    this->smem_iterator_A_.store(transform_A0(tb_frag_A));
    this->smem_iterator_B0_.store(transform_B0(tb_frag_B0));
    ++this->smem_iterator_A_;
    ++this->smem_iterator_B0_;
    __syncthreads();
    // Pair of fragments used to overlap shared memory loads and math instructions
    WarpFragmentA0 warp_frag_A0[2];
    WarpFragmentB0 warp_frag_B0[2];
    this->warp_tile_iterator_A0_.set_kgroup_index(0);
    this->warp_tile_iterator_B0_.set_kgroup_index(0);
    this->warp_tile_iterator_A0_.load(warp_frag_A0[0]);
    this->warp_tile_iterator_B0_.load(warp_frag_B0[0]);
    ++this->warp_tile_iterator_A0_;
    ++this->warp_tile_iterator_B0_;
    Operator0 warp_mma0;
    int smem_write_stage_idx = 1;
    // Issue loads during the first warp-level matrix multiply-add *AFTER* issuing 
    // shared memory loads (which have the tightest latency requirement).
    //
    // Mainloop
    //
    // Note: The main loop does not support Base::kWarpGemmIterations == 2.
    CUTLASS_GEMM_LOOP
    for (; gemm_k_iterations_0 > 0; --gemm_k_iterations_0) {
      //
      // Loop over GEMM K dimension
      //
      CUTLASS_PRAGMA_UNROLL
      for (int warp_mma_k = 0; warp_mma_k < Base::kWarpGemmIterations0; ++warp_mma_k) {
        // Load warp-level tiles from shared memory, wrapping to k offset if this is the last group
        // as the case may be.
        if (warp_mma_k == Base::kWarpGemmIterations0 - 1) {
          // Write fragments to shared memory
          this->smem_iterator_A_.store(transform_A0(tb_frag_A));
          this->smem_iterator_B0_.store(transform_B0(tb_frag_B0));
          __syncthreads();
          ++this->smem_iterator_A_;
          ++this->smem_iterator_B0_;
          // Add negative offsets to return iterators to the 'start' of the circular buffer in shared memory
          if (smem_write_stage_idx == 1) {
            this->smem_iterator_A_.add_tile_offset({0, -Base::kStages});
            this->smem_iterator_B0_.add_tile_offset({-Base::kStages, 0});
          }
          else {
            this->warp_tile_iterator_A0_.add_tile_offset(
                {0, -Base::kStages * Policy0::kPartitionsK * Base::kWarpGemmIterations0});
            this->warp_tile_iterator_B0_.add_tile_offset(
                {-Base::kStages * Policy0::kPartitionsK * Base::kWarpGemmIterations0,
                 0});
          }
          smem_write_stage_idx ^= 1;
        }
        this->warp_tile_iterator_A0_.set_kgroup_index((warp_mma_k + 1) % Base::kWarpGemmIterations0);
        this->warp_tile_iterator_B0_.set_kgroup_index((warp_mma_k + 1) % Base::kWarpGemmIterations0);
        this->warp_tile_iterator_A0_.load(warp_frag_A0[(warp_mma_k + 1) % 2]);
        this->warp_tile_iterator_B0_.load(warp_frag_B0[(warp_mma_k + 1) % 2]);
        ++this->warp_tile_iterator_A0_;
        ++this->warp_tile_iterator_B0_;
        if (warp_mma_k == 0) {
          iterator_A.load(tb_frag_A);
          iterator_B0.load(tb_frag_B0);
          ++iterator_A;
          ++iterator_B0;
        }
        warp_mma0(accum0, warp_frag_A0[warp_mma_k % 2],
                 warp_frag_B0[warp_mma_k % 2], accum0);
      }
    }
    //2nd Implicit Gemm
    /// Iterator to load a warp-scoped tile of A1 operand from intermediate accumulator tile
    FragmentIteratorA1 warp_tile_iterator_A1_(accum0);
    //
    // Prologue
    //
    FragmentA1ScaleBias tb_frag_A1_scale;
    FragmentA1ScaleBias tb_frag_A1_bias;
    FragmentIteratorA1ScaleBias warp_tile_iterator_A1_scale_(tb_frag_A1_scale);
    FragmentIteratorA1ScaleBias warp_tile_iterator_A1_bias_(tb_frag_A1_bias);
    FragmentB1 tb_frag_B1;
    if(PerChannelScale)
        tb_frag_A1_scale.clear();
    tb_frag_A1_bias.clear();
    tb_frag_B1.clear();
    // The last kblock is loaded in the prolog
    if(PerChannelScale)
        iterator_A1_scale.load(tb_frag_A1_scale);
    iterator_A1_bias.load(tb_frag_A1_bias);
    iterator_B1.load(tb_frag_B1);
    if(PerChannelScale)
        ++iterator_A1_scale;
    ++iterator_A1_bias;
    ++iterator_B1;
    this->smem_iterator_B1_.store(transform_B1(tb_frag_B1));
    ++this->smem_iterator_B1_;
    __syncthreads();
    // Pair of fragments used to overlap shared memory loads and math instructions
    WarpFragmentA1ScaleBias warp_frag_A1_scale[2];
    WarpFragmentA1ScaleBias warp_frag_A1_bias[2];
    WarpFragmentA1 warp_frag_A1[2];
    WarpFragmentB1 warp_frag_B1[2];
    this->warp_tile_iterator_B1_.set_kgroup_index(0);
    if(PerChannelScale)
        warp_tile_iterator_A1_scale_.load(warp_frag_A1_scale[0]);
    warp_tile_iterator_A1_bias_.load(warp_frag_A1_bias[0]);
    warp_tile_iterator_A1_.load(warp_frag_A1[0], warp_frag_A1_scale[0], 
        warp_frag_A1_bias[0], output_op_0);
    this->warp_tile_iterator_B1_.load(warp_frag_B1[0]);
    ++warp_tile_iterator_A1_;
    if(PerChannelScale)
        ++warp_tile_iterator_A1_scale_;
    ++warp_tile_iterator_A1_bias_;
    ++this->warp_tile_iterator_B1_;
    Operator1 warp_mma1;
    smem_write_stage_idx = 1;
    int gemm_k_iterations_1 = FragmentIteratorA1::Policy::kIterations / Base::kWarpGemmIterations1;
    // Issue loads during the first warp-level matrix multiply-add *AFTER* issuing 
    // shared memory loads (which have the tightest latency requirement).
    //
    // Mainloop
    //
    // Note: The main loop does not support Base::kWarpGemmIterations == 2.
    CUTLASS_PRAGMA_UNROLL
    for (; gemm_k_iterations_1 > 0; --gemm_k_iterations_1) {
      //
      // Loop over GEMM K dimension
      //
      CUTLASS_PRAGMA_UNROLL
      for (int warp_mma_k = 0; warp_mma_k < Base::kWarpGemmIterations1; ++warp_mma_k) {
        // Load warp-level tiles from shared memory, wrapping to k offset if this is the last group
        // as the case may be.
        if (warp_mma_k == Base::kWarpGemmIterations1 - 1) {
          this->smem_iterator_B1_.store(transform_B1(tb_frag_B1));
          __syncthreads();
          ++this->smem_iterator_B1_;
          // Add negative offsets to return iterators to the 'start' of the circular buffer in shared memory
          if (smem_write_stage_idx == 1) {
            this->smem_iterator_B1_.add_tile_offset({-Base::kStages, 0});
          }
          else {
            this->warp_tile_iterator_B1_.add_tile_offset(
                {-Base::kStages * Policy1::kPartitionsK * Base::kWarpGemmIterations1,
                 0});
          }
          smem_write_stage_idx ^= 1;
          if(PerChannelScale) {
            tb_frag_A1_scale.clear();
            iterator_A1_scale.load(tb_frag_A1_scale);
            ++iterator_A1_scale;
          }
          tb_frag_A1_bias.clear();
          iterator_A1_bias.load(tb_frag_A1_bias);
          ++iterator_A1_bias;
        }
        this->warp_tile_iterator_B1_.set_kgroup_index((warp_mma_k + 1) % Base::kWarpGemmIterations1);
        if(PerChannelScale)
          warp_tile_iterator_A1_scale_.load(warp_frag_A1_scale[(warp_mma_k + 1) % 2]);
        warp_tile_iterator_A1_bias_.load(warp_frag_A1_bias[(warp_mma_k + 1) % 2]);
        warp_tile_iterator_A1_.load(warp_frag_A1[(warp_mma_k + 1) % 2], 
            warp_frag_A1_scale[(warp_mma_k + 1) % 2], 
            warp_frag_A1_bias[(warp_mma_k + 1) % 2], 
            output_op_0);
        this->warp_tile_iterator_B1_.load(warp_frag_B1[(warp_mma_k + 1) % 2]);
        if(PerChannelScale)
          ++warp_tile_iterator_A1_scale_;
        ++warp_tile_iterator_A1_bias_;
        ++warp_tile_iterator_A1_;
        ++this->warp_tile_iterator_B1_;
        if (warp_mma_k == 0) {
          iterator_B1.load(tb_frag_B1);
          ++iterator_B1;
        }
        warp_mma1(accum, warp_frag_A1[warp_mma_k % 2],
                 warp_frag_B1[warp_mma_k % 2], accum);
      }
    }
  }
};
```

**EN**: This block exercises the configured CUTLASS object at runtime. Typical calls include initialization, capability checks, workspace queries, kernel launches, or reference validation. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这段代码在运行时驱动已配置好的 CUTLASS 对象。常见调用包括初始化、能力检查、工作区查询、内核启动以及参考结果验证。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 547-547 / 第547-547行

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 549-551 / 第549-551行

```cpp
} // namespace threadblock
} // namespace gemm
} // namespace cutlass
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 553-553 / 第553-553行

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

---

## Key Concepts / 关键概念

- Epilogue fusion and output operators / Epilogue 融合与输出算子
- Benchmark and validation harness / 基准测试与验证框架
- Template-driven kernel specialization / 模板驱动的内核特化

## Dependencies / 依赖项

- `"cutlass/cutlass.h"` — Core CUTLASS definitions, architecture tags, and status types. / CUTLASS 的核心定义、架构标签与状态类型。
- `"cutlass/array.h"` — Provides `cutlass/array.h` so this file can use the related API or helper utilities. / 提供 `cutlass/array.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/aligned_buffer.h"` — Provides `cutlass/aligned_buffer.h` so this file can use the related API or helper utilities. / 提供 `cutlass/aligned_buffer.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/numeric_conversion.h"` — Provides `cutlass/numeric_conversion.h` so this file can use the related API or helper utilities. / 提供 `cutlass/numeric_conversion.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/numeric_types.h"` — CUTLASS numeric scalar and packed numeric type definitions. / CUTLASS 数值标量与打包数值类型定义。
- `"cutlass/matrix_shape.h"` — Provides `cutlass/matrix_shape.h` so this file can use the related API or helper utilities. / 提供 `cutlass/matrix_shape.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/gemm/gemm.h"` — Provides `cutlass/gemm/gemm.h` so this file can use the related API or helper utilities. / 提供 `cutlass/gemm/gemm.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/gemm/warp/mma_tensor_op_fragment_iterator.h"` — Provides `cutlass/gemm/warp/mma_tensor_op_fragment_iterator.h` so this file can use the related API or helper utilities. / 提供 `cutlass/gemm/warp/mma_tensor_op_fragment_iterator.h`，使本文件能够使用相关 API 或辅助工具。
- `"threadblock/b2b_mma_base.h"` — Example-local threadblock policy or pipeline component. / 示例本地 threadblock 策略或流水线组件。
