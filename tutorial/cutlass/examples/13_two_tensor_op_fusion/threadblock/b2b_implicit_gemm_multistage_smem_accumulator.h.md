# b2b_implicit_gemm_multistage_smem_accumulator.h — Code Analysis / 代码分析

**Source / 源文件**: `examples/13_two_tensor_op_fusion/threadblock/b2b_implicit_gemm_multistage_smem_accumulator.h`  
**Purpose / 用途**: Defines a multistage fused implicit-GEMM pipeline that uses shared memory for intermediate accumulators. / 定义融合隐式 GEMM 的多级流水线，并使用共享内存保存中间累加器。

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
    \brief Template for a multistage threadblock-scoped Implicit GEMM Convolution kernel.
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

### Lines 37-46 / 第37-46行

```cpp
#include "cutlass/aligned_buffer.h"
#include "cutlass/arch/memory.h"
#include "cutlass/array.h"
#include "cutlass/cutlass.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/matrix_shape.h"
#include "cutlass/numeric_types.h"
#include "cutlass/arch/cache_operation.h"
#include "cutlass/gemm/threadblock/mma_base.h"
#include "cutlass/gemm/warp/mma_tensor_op_fragment_iterator.h"
```

**EN**: These headers pull in the building blocks required by this file. Provides `cutlass/aligned_buffer.h` so this file can use the related API or helper utilities. Provides `cutlass/arch/memory.h` so this file can use the related API or helper utilities. Provides `cutlass/array.h` so this file can use the related API or helper utilities. Core CUTLASS definitions, architecture tags, and status types.  
**CN**: 这些头文件引入了当前文件所需的构建模块。提供 `cutlass/aligned_buffer.h`，使本文件能够使用相关 API 或辅助工具。提供 `cutlass/arch/memory.h`，使本文件能够使用相关 API 或辅助工具。提供 `cutlass/array.h`，使本文件能够使用相关 API 或辅助工具。CUTLASS 的核心定义、架构标签与状态类型。

### Lines 48-49 / 第48-49行

```cpp
#include "threadblock/b2b_mma_base_smem_accumulator.h"
#include "cutlass/epilogue/threadblock/epilogue_smem_accumulator.h"
```

**EN**: These headers pull in the building blocks required by this file. Example-local threadblock policy or pipeline component. Epilogue components that convert accumulators to outputs and optionally fuse extra work.  
**CN**: 这些头文件引入了当前文件所需的构建模块。示例本地 threadblock 策略或流水线组件。epilogue 组件，用于把累加器转换为输出并可选地融合额外计算。

### Lines 51-51 / 第51-51行

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 53-53 / 第53-53行

```cpp
namespace cutlass {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 54-54 / 第54-54行

```cpp
namespace conv {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 55-55 / 第55-55行

```cpp
namespace threadblock {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 57-57 / 第57-57行

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 59-60 / 第59-60行

```cpp
/// Structure to compute the matrix product targeting CUDA cores and SIMT math
/// instructions.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 61-114 / 第61-114行

```cpp
template <
    /// Size of the Gemm problem - concept: gemm::GemmShape<>
    typename Shape0_,
    /// Iterates over tiles of A operand in global memory
    //  (concept: ReadableTileIterator | ForwardTileIterator |
    //  MaskedTileIterator)
    typename IteratorA0_,
    /// Iterates over tiles of A operand in shared memory
    /// (concept: WriteableTileIterator | RandomAccessTileIterator)
    typename SmemIteratorA0_,
    /// Cache operation for operand A
    cutlass::arch::CacheOperation::Kind CacheOpA0,
    /// Iterates over tiles of B operand in global memory
    //  (concept: ReadableTileIterator | ForwardTileIterator |
    //  MaskedTileIterator)
    typename IteratorB0_,
    /// Iterates over tiles of B operand in shared memory
    /// (concept: WriteableTileIterator | RandomAccessTileIterator)
    typename SmemIteratorB0_,
    /// Cache operation for operand B
    cutlass::arch::CacheOperation::Kind CacheOpB0,
    /// Iterates over vectors of scale and bias vector in global memory
    //  (concept: VectorIterator)
    typename IteratorAccumulatorScaleBias_,
    /// Iterates over accumulator tile
    typename FragmentIteratorAccumulator_,
    /// Iterates over accumulator tile in shared memory
    typename SmemIteratorD0_,
    /// Size of the Gemm problem - concept: gemm::GemmShape<>
    typename Shape1_,
    /// Iterates over the intermediate accumulator tile
    //  (concept::MmaTensorOpFragmentIterator) 
    typename WarpIteratorA1_,
    /// Iterates over tiles of B operand in global memory
    //  (concept: ReadableTileIterator | ForwardTileIterator |
    //  MaskedTileIterator)
    typename IteratorB1_,
    /// Iterates over tiles of B operand in shared memory
    /// (concept: WriteableTileIterator | RandomAccessTileIterator)
    typename SmemIteratorB1_,
    /// Cache operation for operand B
    cutlass::arch::CacheOperation::Kind CacheOpB1,
    /// Output operator for 1st Gemm(concept: epilogue::thread::LinearCombinationClamp, etc...) 
    typename OutputOp_,
    /// Policy describing tuning details (concept: MmaPolicy)
    typename Policy0_,
    /// Policy describing tuning details (concept: MmaPolicy)
    typename Policy1_,
    /// Number of stages,
    int Stages,
    /// Used for partial specialization
    typename Enable = bool>
class B2bImplicitGemmMultistageSmemAccumulator : 
  public gemm::threadblock::B2bMmaBaseSmemAccumulator<Shape0_, Shape1_, Policy0_, Policy1_, SmemIteratorD0_, Stages> {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates. `GemmShape<M, N, K>` encodes tile extents along the GEMM M/N/K axes. `LinearCombination` is CUTLASS's standard epilogue functor for scaling accumulators and combining them with existing outputs. Implicit GEMM converts convolution into a GEMM-like tiled traversal without materializing the lowered matrix explicitly. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。 `GemmShape<M, N, K>` 用于描述 GEMM 在 M/N/K 三个维度上的分块大小。 `LinearCombination` 是 CUTLASS 标准 epilogue functor，用于缩放累加器并与已有输出组合。 隐式 GEMM 会把卷积转换为类似 GEMM 的分块遍历，而不显式生成展开后的矩阵。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 115-127 / 第115-127行

```cpp
public:
  ///< Base class
  using Base = gemm::threadblock::B2bMmaBaseSmemAccumulator<Shape0_, Shape1_, Policy0_, Policy1_, SmemIteratorD0_, Stages>;
  ///< Size of the Gemm problem - concept: gemm::GemmShape<>
  using Shape0 = Shape0_;
  ///< Iterates over tiles of A operand in global memory
  using IteratorA0 = IteratorA0_;
  ///< Iterates over tiles of B operand in global memory
  using IteratorB0 = IteratorB0_;
  ///< Iterates over tiles of the scale and bias vectors in global memory
  using IteratorAccumulatorScaleBias = IteratorAccumulatorScaleBias_;
  ///< Policy describing tuning details
  using Policy0 = Policy0_;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps. `GemmShape<M, N, K>` encodes tile extents along the GEMM M/N/K axes. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。 `GemmShape<M, N, K>` 用于描述 GEMM 在 M/N/K 三个维度上的分块大小。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 129-131 / 第129-131行

```cpp
  using SmemIteratorA0 = SmemIteratorA0_;
  using SmemIteratorB0 = SmemIteratorB0_;
  using SmemIteratorD0 = SmemIteratorD0_; ///< Iterates over accumulator tile in shared memory
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 133-133 / 第133-133行

```cpp
  using FragmentIteratorAccumulator = FragmentIteratorAccumulator_;  ///< Iterates over accumulator tile
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 135-135 / 第135-135行

```cpp
  ///< Size of the Gemm problem - concept: gemm::GemmShape<>
```

**EN**: This comment separates sections and documents the purpose of the code that follows. `GemmShape<M, N, K>` encodes tile extents along the GEMM M/N/K axes.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。 `GemmShape<M, N, K>` 用于描述 GEMM 在 M/N/K 三个维度上的分块大小。

### Lines 136-136 / 第136-136行

```cpp
  using Shape1 = Shape1_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 137-137 / 第137-137行

```cpp
  ///< Iterates over tiles of B operand in global memory
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 138-138 / 第138-138行

```cpp
  using IteratorB1 = IteratorB1_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 139-139 / 第139-139行

```cpp
  ///< Policy describing tuning details
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 140-140 / 第140-140行

```cpp
  using Policy1 = Policy1_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 142-143 / 第142-143行

```cpp
  using SmemIteratorB1 = SmemIteratorB1_;
  using WarpIteratorA1 = WarpIteratorA1_;   ///< Iterates over the intermediate accumulator tile in shared memory
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 145-145 / 第145-145行

```cpp
  ///< Epilogue after 1st Gemm
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 146-146 / 第146-146行

```cpp
  using OutputOp = OutputOp_;
```

**EN**: This block defines epilogue functors. In CUTLASS, the epilogue converts accumulators into the output type and can fuse extra work such as scaling, bias handling, clamping, or activation.  
**CN**: 这段代码定义了 epilogue functor。在 CUTLASS 中，epilogue 负责把累加器转换为输出类型，并可融合缩放、偏置处理、截断或激活等附加工作。

### Lines 148-149 / 第148-149行

```cpp
  static const bool PerChannelScale = (OutputOp::kScale ==
      epilogue::thread::ScaleType::OnlyAlphaPerChannelScaling);
```

**EN**: This block defines compile-time constants or static values that parameterize the kernel or the benchmark.  
**CN**: 这段代码定义编译期常量或静态值，用于参数化内核或基准测试。

### Lines 151-153 / 第151-153行

```cpp
  static cutlass::arch::CacheOperation::Kind const kCacheOpA0 = CacheOpA0;
  static cutlass::arch::CacheOperation::Kind const kCacheOpB0 = CacheOpB0;
  static cutlass::arch::CacheOperation::Kind const kCacheOpB1 = CacheOpB1;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 155-157 / 第155-157行

```cpp
  //
  // Dependent types
  //
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 159-159 / 第159-159行

```cpp
  using ElementC = typename Policy0::Operator::ElementC;
```

**EN**: These aliases pin down the scalar types used for inputs, accumulators, and outputs. Choosing them explicitly matters because CUTLASS can compute in a different precision from storage.  
**CN**: 这些别名固定了输入、累加器和输出使用的标量类型。显式指定它们很重要，因为 CUTLASS 可以让计算精度与存储精度不同。

### Lines 161-161 / 第161-161行

```cpp
  /// Fragment of accumulator tile
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 162-162 / 第162-162行

```cpp
  using FragmentC0 = typename Policy0::Operator::FragmentC;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 164-164 / 第164-164行

```cpp
  /// Warp-level Mma
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 165-165 / 第165-165行

```cpp
  using Operator0 = typename Policy0::Operator;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 167-167 / 第167-167行

```cpp
  /// Fragment of Scale and Bias loaded from global memory
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 168-168 / 第168-168行

```cpp
  using FragmentA1ScaleBias = typename IteratorAccumulatorScaleBias::Fragment;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 170-170 / 第170-170行

```cpp
  /// Fragment of accumulator tile
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 171-171 / 第171-171行

```cpp
  using FragmentC1 = typename Policy1::Operator::FragmentC;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 173-173 / 第173-173行

```cpp
  /// Warp-level Mma
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 174-174 / 第174-174行

```cpp
  using Operator1 = typename Policy1::Operator;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 176-176 / 第176-176行

```cpp
  /// Epilog in shared memory
```

**EN**: This comment separates sections and documents the purpose of the code that follows. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 177-181 / 第177-181行

```cpp
  using Epilogue0 = epilogue::threadblock::EpilogueSmemAccumulator<
    SmemIteratorD0,                 ///< SmemTileIterator
    FragmentIteratorAccumulator,    ///< AccumulatorFragmentIterator
    IteratorAccumulatorScaleBias,   ///< ScaleBiasIterator
    OutputOp>;                      ///< Output operator
```

**EN**: This block defines epilogue functors. In CUTLASS, the epilogue converts accumulators into the output type and can fuse extra work such as scaling, bias handling, clamping, or activation. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这段代码定义了 epilogue functor。在 CUTLASS 中，epilogue 负责把累加器转换为输出类型，并可融合缩放、偏置处理、截断或激活等附加工作。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 183-183 / 第183-183行

```cpp
  /// Internal structure exposed for introspection.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 184-184 / 第184-184行

```cpp
  struct Detail {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。

### Lines 186-188 / 第186-188行

```cpp
    static_assert(Base::kWarpGemmIterations0 > 1,
                  "The pipelined structure requires at least two warp-level "
                  "GEMM operations.");
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 189-191 / 第189-191行

```cpp
    static_assert(Base::kWarpGemmIterations1 > 1,
                  "The pipelined structure requires at least two warp-level "
                  "GEMM operations.");
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 193-193 / 第193-193行

```cpp
    /// Number of cp.async instructions to load one stage of operand A
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 194-195 / 第194-195行

```cpp
    static int const AsyncCopyIterationsPerStageA0 =
        IteratorA0::ThreadMap::Iterations::kCount;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 197-197 / 第197-197行

```cpp
    /// Number of cp.async instructions to load one stage of operand B
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 198-199 / 第198-199行

```cpp
    static int const AsyncCopyIterationsPerStageB0 =
        IteratorB0::ThreadMap::Iterations::kCount;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 201-201 / 第201-201行

```cpp
    /// Number of cp.async instructions to load one stage of operand B
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 202-203 / 第202-203行

```cpp
    static int const AsyncCopyIterationsPerStageB1 =
        IteratorB1::ThreadMap::Iterations::kCount;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 205-205 / 第205-205行

```cpp
    /// Number of stages
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 206-206 / 第206-206行

```cpp
    static int const kStages = Stages;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 208-208 / 第208-208行

```cpp
    /// Number of cp.async instructions to load on group of operand A
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 209-210 / 第209-210行

```cpp
    static int const kAccessesPerGroupA0 =
        (AsyncCopyIterationsPerStageA0 + Base::kWarpGemmIterations0 - 1) / Base::kWarpGemmIterations0;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 212-212 / 第212-212行

```cpp
    /// Number of cp.async instructions to load on group of operand B
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 213-214 / 第213-214行

```cpp
    static int const kAccessesPerGroupB0 =
        (AsyncCopyIterationsPerStageB0 + Base::kWarpGemmIterations0 - 1) / Base::kWarpGemmIterations0;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 216-216 / 第216-216行

```cpp
    /// Number of cp.async instructions to load on group of operand B
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 217-218 / 第217-218行

```cpp
    static int const kAccessesPerGroupB1 =
        (AsyncCopyIterationsPerStageB1 + Base::kWarpGemmIterations1 - 1) / Base::kWarpGemmIterations1;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 219-219 / 第219-219行

```cpp
  };
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 221-221 / 第221-221行

```cpp
 private:
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 223-230 / 第223-230行

```cpp
  using WarpLoadedFragmentA0 = typename Operator0::FragmentA;
  using WarpLoadedFragmentB0 = typename Operator0::FragmentB;
  using WarpLoadedFragmentA1 = typename Operator1::FragmentA;
  using WarpLoadedFragmentB1 = typename Operator1::FragmentB;
  using WarpTransformedFragmentA0 = typename Operator0::TransformedFragmentA;
  using WarpTransformedFragmentB0 = typename Operator0::TransformedFragmentB;
  using WarpTransformedFragmentA1 = typename Operator1::TransformedFragmentA;
  using WarpTransformedFragmentB1 = typename Operator1::TransformedFragmentB;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 232-232 / 第232-232行

```cpp
 private:
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 234-236 / 第234-236行

```cpp
  //
  // Data members
  //
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 238-238 / 第238-238行

```cpp
  /// Iterator to write threadblock-scoped tile of A operand to shared memory
```

**EN**: This comment separates sections and documents the purpose of the code that follows. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 239-239 / 第239-239行

```cpp
  SmemIteratorA0 smem_iterator_A0_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 241-241 / 第241-241行

```cpp
  /// Iterator to write threadblock-scoped tile of B operand to shared memory
```

**EN**: This comment separates sections and documents the purpose of the code that follows. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 242-242 / 第242-242行

```cpp
  SmemIteratorB0 smem_iterator_B0_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 244-244 / 第244-244行

```cpp
  /// Shared Memory Iterator to store accumulator tile
```

**EN**: This comment separates sections and documents the purpose of the code that follows. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 245-245 / 第245-245行

```cpp
  SmemIteratorD0 smem_iterator_D0_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 247-247 / 第247-247行

```cpp
  /// Iterator to load a warp-scoped tile of A1 operand from intermediate accumulator tile
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 248-248 / 第248-248行

```cpp
  WarpIteratorA1 warp_tile_iterator_A1_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 250-250 / 第250-250行

```cpp
  /// Iterator to write threadblock-scoped tile of B operand to shared memory
```

**EN**: This comment separates sections and documents the purpose of the code that follows. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 251-251 / 第251-251行

```cpp
  SmemIteratorB1 smem_iterator_B1_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 253-253 / 第253-253行

```cpp
public:
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 255-255 / 第255-255行

```cpp
  /// Construct from tensor references
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 256-299 / 第256-299行

```cpp
  CUTLASS_DEVICE
  B2bImplicitGemmMultistageSmemAccumulator(
      ///< Shared storage needed for internal use by threadblock-scoped GEMM
      typename Base::B2bMmaSharedStorage &shared_storage,
      ///< ID within the threadblock
      int thread_idx,
      ///< ID of warp
      int warp_idx,
      ///< ID of each thread within a warp
      int lane_idx
    ):
      Base(shared_storage, thread_idx, warp_idx, lane_idx),
      smem_iterator_A0_(shared_storage.b2b_mma_shared_storage.shared_storage0.operand_A_ref(), thread_idx),
      smem_iterator_B0_(shared_storage.b2b_mma_shared_storage.shared_storage0.operand_B_ref(), thread_idx),
      smem_iterator_D0_(shared_storage.accumulator_shared_storage0.accum_ref(), lane_idx),
      warp_tile_iterator_A1_(shared_storage.accumulator_shared_storage0.accum_ref(), lane_idx),
      smem_iterator_B1_(shared_storage.b2b_mma_shared_storage.shared_storage1.operand_B_ref(), thread_idx)
  {
    // Compute warp location within threadblock tile by mapping the warp_id to
    // three coordinates:
    //   _m: the warp's position within the threadblock along the M dimension
    //   _n: the warp's position within the threadblock along the N dimension
    //   _k: the warp's position within the threadblock along the K dimension
    int warp_idx_mn_0 = warp_idx % (Base::WarpCount0::kM * Base::WarpCount0::kN);
    int warp_idx_k_0 = warp_idx / (Base::WarpCount0::kM * Base::WarpCount0::kN);
    int warp_idx_m_0 = warp_idx_mn_0 % Base::WarpCount0::kM;
    int warp_idx_n_0 = warp_idx_mn_0 / Base::WarpCount0::kM;
    int warp_idx_mn_1 = warp_idx % (Base::WarpCount1::kM * Base::WarpCount1::kN);
    int warp_idx_k_1 = warp_idx / (Base::WarpCount1::kM * Base::WarpCount1::kN);
    int warp_idx_m_1 = warp_idx_mn_1 % Base::WarpCount1::kM;
    int warp_idx_n_1 = warp_idx_mn_1 / Base::WarpCount1::kM;
    // Add per-warp offsets in units of warp-level tiles
    this->warp_tile_iterator_A0_.add_tile_offset(
        {warp_idx_m_0, Base::kWarpGemmIterations0 * warp_idx_k_0});
    this->warp_tile_iterator_B0_.add_tile_offset(
        {Base::kWarpGemmIterations0 * warp_idx_k_0, warp_idx_n_0});
    warp_tile_iterator_A1_.add_tile_offset(
        {warp_idx_m_1, Base::kWarpGemmIterations1 * warp_idx_k_1});
    this->warp_tile_iterator_B1_.add_tile_offset(
        {Base::kWarpGemmIterations1 * warp_idx_k_1, warp_idx_n_1});
    // Add smem accumulator iterator warp offset
    smem_iterator_D0_.add_tile_offset({ warp_idx_m_0 * SmemIteratorD0::TileIterations::kRow, 
                                        warp_idx_n_0 * SmemIteratorD0::TileIterations::kColumn});
  }
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps. Implicit GEMM converts convolution into a GEMM-like tiled traversal without materializing the lowered matrix explicitly. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。 隐式 GEMM 会把卷积转换为类似 GEMM 的分块遍历，而不显式生成展开后的矩阵。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 307-311 / 第307-311行

```cpp
  CUTLASS_DEVICE
  void copy_tiles_and_advance_0(
    IteratorA0 &iterator_A0, IteratorB0 &iterator_B0,
    int group_start_A0 = 0, int group_start_B0 = 0) {
    iterator_A0.set_iteration_index(group_start_A0);
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 313-313 / 第313-313行

```cpp
    this->smem_iterator_A0_.set_iteration_index(group_start_A0);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 315-315 / 第315-315行

```cpp
    // Async Copy for operand A
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 316-329 / 第316-329行

```cpp
    CUTLASS_PRAGMA_UNROLL
    for (int j = 0; j < Detail::kAccessesPerGroupA0; ++j) {
      if (group_start_A0 + j < Detail::AsyncCopyIterationsPerStageA0) {
        typename IteratorA0::AccessType *dst_ptr =
            reinterpret_cast<typename IteratorA0::AccessType *>(
                this->smem_iterator_A0_.get());
        int const kSrcBytes = sizeof_bits<typename IteratorA0::Element>::value *
                              IteratorA0::ThreadMap::kElementsPerAccess / 8;
        cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpA0>(
                dst_ptr, iterator_A0.get(), iterator_A0.valid());
        ++iterator_A0;
        ++this->smem_iterator_A0_;
      }
    }
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 336-336 / 第336-336行

```cpp
    iterator_B0.set_iteration_index(group_start_B0);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 338-338 / 第338-338行

```cpp
    this->smem_iterator_B0_.set_iteration_index(group_start_B0);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 340-340 / 第340-340行

```cpp
    // Async Copy for operand B
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 341-355 / 第341-355行

```cpp
    CUTLASS_PRAGMA_UNROLL
    for (int j = 0; j < Detail::kAccessesPerGroupB0; ++j) {
      if (group_start_B0 + j < Detail::AsyncCopyIterationsPerStageB0) {
        typename IteratorB0::AccessType *dst_ptr =
            reinterpret_cast<typename IteratorB0::AccessType *>(
                this->smem_iterator_B0_.get());
        int const kSrcBytes = sizeof_bits<typename IteratorB0::Element>::value *
                              IteratorB0::ThreadMap::kElementsPerAccess / 8;
        cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpB0>(
                dst_ptr, iterator_B0.get(), iterator_B0.valid());
        ++iterator_B0;
        ++this->smem_iterator_B0_;
      }
    }
  }
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 360-364 / 第360-364行

```cpp
  CUTLASS_DEVICE
  void copy_tiles_and_advance_1(
    IteratorB1 &iterator_B1,
    int group_start_B1 = 0) {
    iterator_B1.set_iteration_index(group_start_B1);
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 367-367 / 第367-367行

```cpp
    this->smem_iterator_B1_.set_iteration_index(group_start_B1);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 369-369 / 第369-369行

```cpp
    // Async Copy for operand B
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 370-384 / 第370-384行

```cpp
    CUTLASS_PRAGMA_UNROLL
    for (int j = 0; j < Detail::kAccessesPerGroupB1; ++j) {
      if (group_start_B1 + j < Detail::AsyncCopyIterationsPerStageB1) {
        typename IteratorB1::AccessType *dst_ptr =
            reinterpret_cast<typename IteratorB1::AccessType *>(
                this->smem_iterator_B1_.get());
        int const kSrcBytes = sizeof_bits<typename IteratorB1::Element>::value *
                              IteratorB1::ThreadMap::kElementsPerAccess / 8;
        cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpB1>(
                dst_ptr, iterator_B1.get(), iterator_B1.valid());
        ++iterator_B1;
        ++this->smem_iterator_B1_;
      }
    }
  }
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 389-389 / 第389-389行

```cpp
  /// Perform a threadblock-scoped matrix multiply-accumulate
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 390-715 / 第390-715行

```cpp
  CUTLASS_DEVICE
  void operator()(
      ///< problem size of GEMM
      int gemm_k_iterations_0,
      ///< destination accumulator tile
      FragmentC1 &accum,
      ///< iterator over A0 operand in global memory
      IteratorA0 iterator_A0,
      ///< iterator over B0 operand in global memory
      IteratorB0 iterator_B0,
      ///< iterator over A1 operand scale vector in global memory
      IteratorAccumulatorScaleBias iterator_accum0_scale,
      ///< iterator over A1 operand bias vector in global memory
      IteratorAccumulatorScaleBias iterator_accum0_bias,
      ///< iterator over B1 operand in global memory
      IteratorB1 iterator_B1,
      ///< initial value of accumulator
      FragmentC0 const &src_accum,
      ///< epilogue operation after 1st Gemm
      OutputOp output_op_0,
      ///< Imaginary strides used for planar-complex only - ignored here
      int64_t imag_stride_A = 0,
      int64_t imag_stride_B = 0) {
    //
    // Prologue
    //
    // Issue several complete stages
    CUTLASS_PRAGMA_UNROLL
    for (int stage = 0; stage < Base::kStages - 1;
         ++stage, --gemm_k_iterations_0) {
      iterator_A0.set_iteration_index(0);
      this->smem_iterator_A0_.set_iteration_index(0);
      // Async Copy for operand A
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < Detail::AsyncCopyIterationsPerStageA0; ++j) {
        typename IteratorA0::AccessType *dst_ptr =
          reinterpret_cast<typename IteratorA0::AccessType *>(
            this->smem_iterator_A0_.get());
        int const kSrcBytes =
            sizeof_bits<typename IteratorA0::Element>::value *
            IteratorA0::ThreadMap::kElementsPerAccess / 8;
        cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpA0>(
          dst_ptr, iterator_A0.get(), iterator_A0.valid());
        ++iterator_A0;
        ++this->smem_iterator_A0_;
      }
      iterator_B0.set_iteration_index(0);
      this->smem_iterator_B0_.set_iteration_index(0);
      // Async Copy for operand B
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < Detail::AsyncCopyIterationsPerStageB0; ++j) {
        typename IteratorB0::AccessType *dst_ptr =
          reinterpret_cast<typename IteratorB0::AccessType *>(
              this->smem_iterator_B0_.get());
        int const kSrcBytes =
            sizeof_bits<typename IteratorB0::Element>::value *
            IteratorB0::ThreadMap::kElementsPerAccess / 8;
        cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpB0>(
            dst_ptr, iterator_B0.get(), iterator_B0.valid());
        ++iterator_B0;
        ++this->smem_iterator_B0_;
      }
      // Move to the next stage
      iterator_A0.advance();
      iterator_B0.advance();
      this->smem_iterator_A0_.add_tile_offset({0, 1});
      this->smem_iterator_B0_.add_tile_offset({1, 0});
      // Inserts a fence to group cp.async instructions into stages.
      cutlass::arch::cp_async_fence();
    }
    // Perform accumulation in the 'd' output operand
    FragmentC0 accum0 = src_accum;
    // Waits until kStages-2 stages have committed. 
    cutlass::arch::cp_async_wait<Base::kStages - 2>();
    __syncthreads();
    // Pair of fragments used to overlap shared memory loads and math
    // instructions
    WarpLoadedFragmentA0 warp_loaded_frag_A0[2];
    WarpLoadedFragmentB0 warp_loaded_frag_B0[2];
    WarpTransformedFragmentA0 warp_transformed_frag_A0[2];
    WarpTransformedFragmentB0 warp_transformed_frag_B0[2];
    Operator0 warp_mma0;
    this->warp_tile_iterator_A0_.set_kgroup_index(0);
    this->warp_tile_iterator_B0_.set_kgroup_index(0);
    this->warp_tile_iterator_A0_.load(warp_loaded_frag_A0[0]);
    this->warp_tile_iterator_B0_.load(warp_loaded_frag_B0[0]);
    ++this->warp_tile_iterator_A0_;
    ++this->warp_tile_iterator_B0_;
    // Start issuing the first group of the next stage outside of the mainloop
    copy_tiles_and_advance_0(iterator_A0, iterator_B0);
    int smem_write_stage_idx = Base::kStages - 1;
    int smem_read_stage_idx = 0;
    warp_mma0.transform(warp_transformed_frag_A0[0], warp_transformed_frag_B0[0],
                       warp_loaded_frag_A0[0], warp_loaded_frag_B0[0]);
    //
    // Mainloop
    //
    CUTLASS_GEMM_LOOP
    for (; gemm_k_iterations_0 > (-Base::kStages + 1);) {
      //
      // Loop over GEMM K dimension
      //
      // Computes a warp-level GEMM on data held in shared memory
      // Each "warp_mma_k" refers to a warp-level matrix multiply-accumulate
      CUTLASS_PRAGMA_UNROLL
      for (int warp_mma_k = 0; warp_mma_k < Base::kWarpGemmIterations0;
           ++warp_mma_k) {
        // Load warp-level tiles from shared memory, wrapping to k offset if
        // this is the last group as the case may be.
        this->warp_tile_iterator_A0_.set_kgroup_index((warp_mma_k + 1) % Base::kWarpGemmIterations0);
        this->warp_tile_iterator_B0_.set_kgroup_index((warp_mma_k + 1) % Base::kWarpGemmIterations0);
        this->warp_tile_iterator_A0_.load(warp_loaded_frag_A0[(warp_mma_k + 1) % 2]);
        this->warp_tile_iterator_B0_.load(warp_loaded_frag_B0[(warp_mma_k + 1) % 2]);
        ++this->warp_tile_iterator_A0_;
        ++this->warp_tile_iterator_B0_;
        if (warp_mma_k > 0)
          warp_mma0.transform(warp_transformed_frag_A0[warp_mma_k % 2],
                             warp_transformed_frag_B0[warp_mma_k % 2],
                             warp_loaded_frag_A0[warp_mma_k % 2],
                             warp_loaded_frag_B0[warp_mma_k % 2]);
        // Issue global->shared copies for the next stage
        int group_start_iteration_A0, group_start_iteration_B0;
        if (warp_mma_k + 1 == Base::kWarpGemmIterations0) {
          group_start_iteration_A0 = 0;
          group_start_iteration_B0 = 0;
        } else {
          group_start_iteration_A0 =
              (warp_mma_k + 1) * Detail::kAccessesPerGroupA0;
          group_start_iteration_B0 =
              (warp_mma_k + 1) * Detail::kAccessesPerGroupB0;
        }
        copy_tiles_and_advance_0(iterator_A0, iterator_B0, group_start_iteration_A0,
                               group_start_iteration_B0);
        warp_mma0(
                accum0, 
                warp_transformed_frag_A0[warp_mma_k % 2],
                warp_transformed_frag_B0[warp_mma_k % 2], 
                accum0
                );
        if (warp_mma_k + 1 == Base::kWarpGemmIterations0)
          warp_mma0.transform(warp_transformed_frag_A0[(warp_mma_k + 1) % 2],
                             warp_transformed_frag_B0[(warp_mma_k + 1) % 2],
                             warp_loaded_frag_A0[(warp_mma_k + 1) % 2],
                             warp_loaded_frag_B0[(warp_mma_k + 1) % 2]);
        if (warp_mma_k + 2 == Base::kWarpGemmIterations0) {
          // Inserts a fence to group cp.async instructions into stages.
          cutlass::arch::cp_async_fence();
          // Waits until kStages-2 stages of cp.async have committed
          arch::cp_async_wait<Base::kStages - 2>();
          __syncthreads();
          // Move to the next stage
          iterator_A0.advance();
          iterator_B0.advance();
          this->smem_iterator_A0_.add_tile_offset({0, 1});
          this->smem_iterator_B0_.add_tile_offset({1, 0});
          // Add negative offsets to return iterators to the 'start' of the
          // circular buffer in shared memory
          if (smem_write_stage_idx == (Base::kStages - 1)) {
            this->smem_iterator_A0_.add_tile_offset({0, -Base::kStages});
            this->smem_iterator_B0_.add_tile_offset({-Base::kStages, 0});
            smem_write_stage_idx = 0;
          } else {
            ++smem_write_stage_idx;
          }
          if (smem_read_stage_idx == (Base::kStages - 1)) {
            this->warp_tile_iterator_A0_.add_tile_offset(
                {0, -Base::kStages * Policy0::kPartitionsK *
                        Base::kWarpGemmIterations0});
            this->warp_tile_iterator_B0_.add_tile_offset(
                {-Base::kStages * Policy0::kPartitionsK *
                     Base::kWarpGemmIterations0,
                 0});
            smem_read_stage_idx = 0;
          } else {
            ++smem_read_stage_idx;
          }
          --gemm_k_iterations_0;
        }
      }
    }
    // Insert fence and wait for all outstanding cp.async operations to commit.
    cutlass::arch::cp_async_fence();
    cutlass::arch::cp_async_wait<0>();
    __syncthreads();
    /// Epilogue for the first Implicit Gemm
    Epilogue0 epilogue0;
    epilogue0(output_op_0, smem_iterator_D0_, accum0, iterator_accum0_scale, iterator_accum0_bias);
    __syncthreads();
    // 2nd Implicit Gemm
    //
    // Prologue
    //
    int gemm_k_iterations_1 = Shape0::kN / Shape1::kK;
    // Issue several complete stages
    CUTLASS_PRAGMA_UNROLL
    for (int stage = 0; stage < Base::kStages - 1;
         ++stage, --gemm_k_iterations_1) {
      iterator_B1.set_iteration_index(0);
      this->smem_iterator_B1_.set_iteration_index(0);
      // Async Copy for operand B
      CUTLASS_PRAGMA_UNROLL
      for (int j = 0; j < Detail::AsyncCopyIterationsPerStageB1; ++j) {
        typename IteratorB1::AccessType *dst_ptr =
          reinterpret_cast<typename IteratorB1::AccessType *>(
              this->smem_iterator_B1_.get());
        int const kSrcBytes =
            sizeof_bits<typename IteratorB1::Element>::value *
            IteratorB1::ThreadMap::kElementsPerAccess / 8;
        cutlass::arch::cp_async_zfill<kSrcBytes, kCacheOpB1>(
            dst_ptr, iterator_B1.get(), iterator_B1.valid());
        ++iterator_B1;
        ++this->smem_iterator_B1_;
      }
      // Move to the next stage
      iterator_B1.advance();
      this->smem_iterator_B1_.add_tile_offset({1, 0});
      // Inserts a fence to group cp.async instructions into stages.
      cutlass::arch::cp_async_fence();
    }
    // Waits until kStages-2 stages have committed. 
    cutlass::arch::cp_async_wait<Base::kStages - 2>();
    __syncthreads();
    // Pair of fragments used to overlap shared memory loads and math
    // instructions
    WarpLoadedFragmentA1 warp_loaded_frag_A1[2];
    WarpLoadedFragmentB1 warp_loaded_frag_B1[2];
    WarpTransformedFragmentA1 warp_transformed_frag_A1[2];
    WarpTransformedFragmentB1 warp_transformed_frag_B1[2];
    Operator1 warp_mma1;
    warp_tile_iterator_A1_.load(warp_loaded_frag_A1[0]);
    ++warp_tile_iterator_A1_;
    this->warp_tile_iterator_B1_.set_kgroup_index(0);
    this->warp_tile_iterator_B1_.load(warp_loaded_frag_B1[0]);
    ++this->warp_tile_iterator_B1_;
    // Start issuing the first group of the next stage outside of the mainloop
    copy_tiles_and_advance_1(iterator_B1);
    smem_write_stage_idx = Base::kStages - 1;
    smem_read_stage_idx = 0;
    warp_mma1.transform(warp_transformed_frag_A1[0], warp_transformed_frag_B1[0],
                       warp_loaded_frag_A1[0], warp_loaded_frag_B1[0]);
    //
    // Mainloop
    //
    CUTLASS_PRAGMA_UNROLL
    for ( gemm_k_iterations_1 = Shape0::kN / Shape1::kK - (Base::kStages - 1); 
            gemm_k_iterations_1 > (-Base::kStages + 1); gemm_k_iterations_1--) {
      //
      // Loop over GEMM K dimension
      //
      // Computes a warp-level GEMM on data held in shared memory
      // Each "warp_mma_k" refers to a warp-level matrix multiply-accumulate
      CUTLASS_PRAGMA_UNROLL
      for (int warp_mma_k = 0; warp_mma_k < Base::kWarpGemmIterations1;
           ++warp_mma_k) {
        // Load warp-level tile from accumulator fragment
        // skip warp tile loading for the last kgroup
        if(gemm_k_iterations_1 > (-Base::kStages + 2) || warp_mma_k < Base::kWarpGemmIterations1 - 1) {
            warp_tile_iterator_A1_.load(warp_loaded_frag_A1[(warp_mma_k + 1) % 2]);
        }
        ++warp_tile_iterator_A1_;
        // Load warp-level tiles from shared memory, wrapping to k offset if
        // this is the last group as the case may be.
        this->warp_tile_iterator_B1_.set_kgroup_index((warp_mma_k + 1) % Base::kWarpGemmIterations1);
        this->warp_tile_iterator_B1_.load(warp_loaded_frag_B1[(warp_mma_k + 1) % 2]);
        ++this->warp_tile_iterator_B1_;
        if (warp_mma_k > 0)
          warp_mma1.transform(warp_transformed_frag_A1[warp_mma_k % 2],
                             warp_transformed_frag_B1[warp_mma_k % 2],
                             warp_loaded_frag_A1[warp_mma_k % 2],
                             warp_loaded_frag_B1[warp_mma_k % 2]);
        // Issue global->shared copies for the next stage
        int group_start_iteration_B1;
        if (warp_mma_k + 1 == Base::kWarpGemmIterations1) {
          group_start_iteration_B1 = 0;
        } else {
          group_start_iteration_B1 =
              (warp_mma_k + 1) * Detail::kAccessesPerGroupB1;
        }
        copy_tiles_and_advance_1(iterator_B1,
                               group_start_iteration_B1);
        warp_mma1(
                accum, 
                warp_transformed_frag_A1[warp_mma_k % 2],
                warp_transformed_frag_B1[warp_mma_k % 2], 
                accum
                );
        if (warp_mma_k + 1 == Base::kWarpGemmIterations1)
          warp_mma1.transform(warp_transformed_frag_A1[(warp_mma_k + 1) % 2],
                             warp_transformed_frag_B1[(warp_mma_k + 1) % 2],
                             warp_loaded_frag_A1[(warp_mma_k + 1) % 2],
                             warp_loaded_frag_B1[(warp_mma_k + 1) % 2]);
        if (warp_mma_k + 2 == Base::kWarpGemmIterations1) {
          // Inserts a fence to group cp.async instructions into stages.
          cutlass::arch::cp_async_fence();
          // Waits until kStages-2 stages of cp.async have committed
          arch::cp_async_wait<Base::kStages - 2>();
          __syncthreads();
          // Move to the next stage
          iterator_B1.advance();
          this->smem_iterator_B1_.add_tile_offset({1, 0});
          // Add negative offsets to return iterators to the 'start' of the
          // circular buffer in shared memory
          if (smem_write_stage_idx == (Base::kStages - 1)) {
            this->smem_iterator_B1_.add_tile_offset({-Base::kStages, 0});
            smem_write_stage_idx = 0;
          } else {
            ++smem_write_stage_idx;
          }
          if (smem_read_stage_idx == (Base::kStages - 1)) {
            this->warp_tile_iterator_B1_.add_tile_offset(
                {-Base::kStages * Policy1::kPartitionsK *
                     Base::kWarpGemmIterations1,
                 0});
            smem_read_stage_idx = 0;
          } else {
            ++smem_read_stage_idx;
          }
        }
      }
    }
    // Insert fence and wait for all outstanding cp.async operations to commit.
    cutlass::arch::cp_async_fence();
    cutlass::arch::cp_async_wait<0>();
    __syncthreads();
  }
};
```

**EN**: This block exercises the configured CUTLASS object at runtime. Typical calls include initialization, capability checks, workspace queries, kernel launches, or reference validation. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这段代码在运行时驱动已配置好的 CUTLASS 对象。常见调用包括初始化、能力检查、工作区查询、内核启动以及参考结果验证。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 810-810 / 第810-810行

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 812-814 / 第812-814行

```cpp
}  // namespace threadblock
}  // namespace gemm
}  // namespace cutlass
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 816-816 / 第816-816行

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

---

## Key Concepts / 关键概念

- Epilogue fusion and output operators / Epilogue 融合与输出算子
- Shared-memory accumulator staging / 共享内存累加器暂存
- Benchmark and validation harness / 基准测试与验证框架

## Dependencies / 依赖项

- `"cutlass/aligned_buffer.h"` — Provides `cutlass/aligned_buffer.h` so this file can use the related API or helper utilities. / 提供 `cutlass/aligned_buffer.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/arch/memory.h"` — Provides `cutlass/arch/memory.h` so this file can use the related API or helper utilities. / 提供 `cutlass/arch/memory.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/array.h"` — Provides `cutlass/array.h` so this file can use the related API or helper utilities. / 提供 `cutlass/array.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/cutlass.h"` — Core CUTLASS definitions, architecture tags, and status types. / CUTLASS 的核心定义、架构标签与状态类型。
- `"cutlass/gemm/gemm.h"` — Provides `cutlass/gemm/gemm.h` so this file can use the related API or helper utilities. / 提供 `cutlass/gemm/gemm.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/matrix_shape.h"` — Provides `cutlass/matrix_shape.h` so this file can use the related API or helper utilities. / 提供 `cutlass/matrix_shape.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/numeric_types.h"` — CUTLASS numeric scalar and packed numeric type definitions. / CUTLASS 数值标量与打包数值类型定义。
- `"cutlass/arch/cache_operation.h"` — Provides `cutlass/arch/cache_operation.h` so this file can use the related API or helper utilities. / 提供 `cutlass/arch/cache_operation.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/gemm/threadblock/mma_base.h"` — Threadblock-scoped GEMM machinery such as MMA cores and swizzles. / threadblock 级 GEMM 组件，例如 MMA core 与 swizzle。
- `"cutlass/gemm/warp/mma_tensor_op_fragment_iterator.h"` — Provides `cutlass/gemm/warp/mma_tensor_op_fragment_iterator.h` so this file can use the related API or helper utilities. / 提供 `cutlass/gemm/warp/mma_tensor_op_fragment_iterator.h`，使本文件能够使用相关 API 或辅助工具。
- `"threadblock/b2b_mma_base_smem_accumulator.h"` — Example-local threadblock policy or pipeline component. / 示例本地 threadblock 策略或流水线组件。
- `"cutlass/epilogue/threadblock/epilogue_smem_accumulator.h"` — Epilogue components that convert accumulators to outputs and optionally fuse extra work. / epilogue 组件，用于把累加器转换为输出并可选地融合额外计算。
