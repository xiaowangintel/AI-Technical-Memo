# default_b2b_gemm_smem_accumulator.h — Code Analysis / 代码分析

**Source / 源文件**: `examples/13_two_tensor_op_fusion/kernel/default_b2b_gemm_smem_accumulator.h`  
**Purpose / 用途**: Provides the default kernel builder for fused GEMM variants that spill intermediate accumulators to shared memory. / 提供融合 GEMM 变体的默认内核构建器，支持把中间累加器暂存到共享内存。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第1-30行

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
```

**EN**: This is the standard BSD-3-Clause license header. It documents redistribution terms before any executable code appears.  
**CN**: 这里是标准的 BSD-3-Clause 许可证头，在任何可执行代码之前先说明再分发条款。

### Lines 32-32 / 第32-32行

```cpp
/*! \file
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 33-35 / 第33-35行

```cpp
    \brief
      Default kernel-level GEMM definitions combine threadblock-scoped matrix multiply-add with
      the appropriate threadblock-scoped epilogue.
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 37-40 / 第37-40行

```cpp
      Note, CUTLASS epilogues universally target row-major outputs. Column-major outputs are
      accommodated by exchanging A and B operands and assuming transposed layouts. Partial
      specializations here choose 'device::GemmTransposed' to implement this functionality.
*/
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 42-42 / 第42-42行

```cpp
#pragma once
```

**EN**: This pragma makes the header idempotent so repeated inclusion does not create duplicate definitions.  
**CN**: 这个 pragma 保证头文件只会被处理一次，避免重复包含产生重复定义。

### Lines 44-44 / 第44-44行

```cpp
#include "cutlass/cutlass.h"
```

**EN**: These headers pull in the building blocks required by this file. Core CUTLASS definitions, architecture tags, and status types.  
**CN**: 这些头文件引入了当前文件所需的构建模块。CUTLASS 的核心定义、架构标签与状态类型。

### Lines 46-47 / 第46-47行

```cpp
#include "cutlass/layout/matrix.h"
#include "cutlass/numeric_types.h"
```

**EN**: These headers pull in the building blocks required by this file. Layout descriptors that define how tensors are mapped in memory. CUTLASS numeric scalar and packed numeric type definitions.  
**CN**: 这些头文件引入了当前文件所需的构建模块。描述张量内存映射方式的布局描述符。CUTLASS 数值标量与打包数值类型定义。

### Lines 49-50 / 第49-50行

```cpp
#include "cutlass/epilogue/threadblock/epilogue.h"
#include "cutlass/epilogue/thread/linear_combination.h"
```

**EN**: These headers pull in the building blocks required by this file. Epilogue components that convert accumulators to outputs and optionally fuse extra work. Epilogue components that convert accumulators to outputs and optionally fuse extra work.  
**CN**: 这些头文件引入了当前文件所需的构建模块。epilogue 组件，用于把累加器转换为输出并可选地融合额外计算。epilogue 组件，用于把累加器转换为输出并可选地融合额外计算。

### Lines 52-61 / 第52-61行

```cpp
#include "cutlass/gemm/gemm.h"
#include "cutlass/gemm/kernel/gemm_pipelined.h"
#include "cutlass/gemm/threadblock/default_mma_core_sm75.h"
#include "cutlass/gemm/threadblock/default_mma_core_sm70.h"
#include "cutlass/gemm/threadblock/default_mma_core_sm80.h"
#include "cutlass/gemm/threadblock/default_mma_core_simt.h"
#include "cutlass/gemm/threadblock/threadblock_swizzle.h"
#include "cutlass/epilogue/threadblock/default_epilogue_tensor_op.h"
#include "cutlass/epilogue/threadblock/default_epilogue_volta_tensor_op.h"
#include "cutlass/epilogue/threadblock/default_epilogue_simt.h"
```

**EN**: These headers pull in the building blocks required by this file. Provides `cutlass/gemm/gemm.h` so this file can use the related API or helper utilities. Kernel-level GEMM building blocks and default kernel assemblers. Threadblock-scoped GEMM machinery such as MMA cores and swizzles. Threadblock-scoped GEMM machinery such as MMA cores and swizzles.  
**CN**: 这些头文件引入了当前文件所需的构建模块。提供 `cutlass/gemm/gemm.h`，使本文件能够使用相关 API 或辅助工具。内核级 GEMM 构建模块与默认内核拼装器。threadblock 级 GEMM 组件，例如 MMA core 与 swizzle。threadblock 级 GEMM 组件，例如 MMA core 与 swizzle。

### Lines 63-65 / 第63-65行

```cpp
#include "cutlass/transform/threadblock/predicated_tile_iterator.h"
#include "cutlass/transform/threadblock/vector_iterator.h"
#include "cutlass/transform/threadblock/predicated_vector_access_iterator.h"
```

**EN**: These headers pull in the building blocks required by this file. Iterator and transform utilities that move tensor tiles efficiently. Iterator and transform utilities that move tensor tiles efficiently. Iterator and transform utilities that move tensor tiles efficiently.  
**CN**: 这些头文件引入了当前文件所需的构建模块。用于高效搬运张量分块的迭代器与变换工具。用于高效搬运张量分块的迭代器与变换工具。用于高效搬运张量分块的迭代器与变换工具。

### Lines 67-69 / 第67-69行

```cpp
#include "kernel/b2b_gemm.h"
#include "threadblock/default_b2b_mma.h"
#include "threadblock/default_b2b_mma_smem_accumulator.h"
```

**EN**: These headers pull in the building blocks required by this file. Example-local kernel definition used to compose specialized fused execution paths. Example-local threadblock policy or pipeline component. Example-local threadblock policy or pipeline component.  
**CN**: 这些头文件引入了当前文件所需的构建模块。示例本地内核定义，用于组合专门化的融合执行路径。示例本地 threadblock 策略或流水线组件。示例本地 threadblock 策略或流水线组件。

### Lines 71-71 / 第71-71行

```cpp
////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 73-73 / 第73-73行

```cpp
namespace cutlass {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 74-74 / 第74-74行

```cpp
namespace gemm {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 75-75 / 第75-75行

```cpp
namespace kernel {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 77-77 / 第77-77行

```cpp
////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 79-79 / 第79-79行

```cpp
/// Partial specialization for Ampere Architecture
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 80-122 / 第80-122行

```cpp
template <
    /// Element type for A matrix operand
    typename ElementA,
    /// Layout type for A matrix operand
    typename LayoutA,
    /// Access granularity of A matrix in units of elements
    int kAlignmentA,
    /// Element type for B matrix operand
    typename ElementB,
    /// Layout type for B matrix operand
    typename LayoutB,
    /// Access granularity of A matrix in units of elements
    int kAlignmentB,
    /// Element type for C and D matrix operands
    typename ElementC,
    /// Element type for internal accumulation
    typename ElementAccumulator,
    /// Threadblock-level tile size (concept: GemmShape)
    typename ThreadblockShape0,
    /// Threadblock-level tile size (concept: GemmShape)
    typename ThreadblockShape1,
    /// Warp-level tile size (concept: GemmShape)
    typename WarpShape0,
    /// Warp-level tile size (concept: GemmShape)
    typename WarpShape1,
    /// Warp-level tile size (concept: GemmShape)
    typename InstructionShape,
    /// Epilogue output operator
    typename EpilogueOutputOp0,
    /// Epilogue output operator
    typename EpilogueOutputOp1,
    /// Threadblock-level swizzling operator
    typename ThreadblockSwizzle,
    /// Number of stages used in the pipelined mainloop
    int Stages,
    /// Operation performed by GEMM
    typename Operator>
struct DefaultB2bGemm<ElementA, LayoutA, kAlignmentA, ElementB, LayoutB, kAlignmentB, ElementC,
                   layout::RowMajor, ElementAccumulator, arch::OpClassTensorOp,
                   arch::Sm80, ThreadblockShape0, ThreadblockShape1,
                   WarpShape0, WarpShape1, InstructionShape,
                   EpilogueOutputOp0, EpilogueOutputOp1, ThreadblockSwizzle, Stages,
                   Operator, true> {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps. `InstructionShape` mirrors the hardware MMA instruction tile used by Tensor Cores. `arch::Sm80` targets Ampere-generation GPUs. `OpClassTensorOp` selects Tensor Core math instead of plain SIMT arithmetic.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。 `InstructionShape` 对应 Tensor Core 使用的硬件 MMA 指令级分块。 `arch::Sm80` 表示面向 Ampere 架构 GPU。 `OpClassTensorOp` 选择 Tensor Core 计算，而不是普通的 SIMT 算术。

### Lines 123-123 / 第123-123行

```cpp
  /// Define the threadblock-scoped matrix multiply-accumulate
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 124-128 / 第124-128行

```cpp
  using B2bMma = typename cutlass::gemm::threadblock::DefaultB2bMma<
      ElementA, LayoutA, kAlignmentA, ElementB, LayoutB, kAlignmentB,
      ElementAccumulator, layout::RowMajor, arch::OpClassTensorOp, arch::Sm80,
      ThreadblockShape0, ThreadblockShape1, WarpShape0, WarpShape1,
      InstructionShape, Stages, Operator, EpilogueOutputOp0, false, true>::ThreadblockB2bMma;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps. `InstructionShape` mirrors the hardware MMA instruction tile used by Tensor Cores. `arch::Sm80` targets Ampere-generation GPUs. `OpClassTensorOp` selects Tensor Core math instead of plain SIMT arithmetic.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。 `InstructionShape` 对应 Tensor Core 使用的硬件 MMA 指令级分块。 `arch::Sm80` 表示面向 Ampere 架构 GPU。 `OpClassTensorOp` 选择 Tensor Core 计算，而不是普通的 SIMT 算术。

### Lines 130-130 / 第130-130行

```cpp
  static const int kPartitionsK1 = ThreadblockShape1::kK / WarpShape1::kK;
```

**EN**: This block defines compile-time constants or static values that parameterize the kernel or the benchmark. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps.  
**CN**: 这段代码定义编译期常量或静态值，用于参数化内核或基准测试。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。

### Lines 132-132 / 第132-132行

```cpp
  /// Define the epilogue
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 133-136 / 第133-136行

```cpp
  using Epilogue =
      typename cutlass::epilogue::threadblock::DefaultEpilogueTensorOp<
          ThreadblockShape1, typename B2bMma::Operator1, kPartitionsK1, EpilogueOutputOp1,
          EpilogueOutputOp1::kCount>::Epilogue;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth. `ThreadblockShape` selects the CTA-sized tile computed by one thread block.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。

### Lines 138-138 / 第138-138行

```cpp
  /// Define the kernel-level GEMM operator.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 139-139 / 第139-139行

```cpp
  using B2bGemmKernel = kernel::B2bGemm<B2bMma, Epilogue, ThreadblockSwizzle>;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。

### Lines 140-140 / 第140-140行

```cpp
};
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 142-142 / 第142-142行

```cpp
////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 144-144 / 第144-144行

```cpp
/// Partial specialization for Turing Architecture
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 145-199 / 第145-199行

```cpp
template <
  /// Element type for A matrix operand
  typename ElementA,
  /// Layout type for A matrix operand
  typename LayoutA,
  /// Access granularity of A matrix in units of elements
  int kAlignmentA,
  /// Element type for B matrix operand
  typename ElementB,
  /// Layout type for B matrix operand
  typename LayoutB,
  /// Access granularity of B matrix in units of elements
  int kAlignmentB,
  /// Element type for C and D matrix operands
  typename ElementC,
  /// Element type for internal accumulation
  typename ElementAccumulator,
  /// Threadblock-level tile size (concept: GemmShape)
  typename ThreadblockShape0,
  /// Threadblock-level tile size (concept: GemmShape)
  typename ThreadblockShape1,
  /// Warp-level tile size (concept: GemmShape)
  typename WarpShape0,
  /// Warp-level tile size (concept: GemmShape)
  typename WarpShape1,
  /// Warp-level tile size (concept: GemmShape)
  typename InstructionShape,
  /// Epilogue output operator
  typename EpilogueOutputOp0,
  /// Epilogue output operator
  typename EpilogueOutputOp1,
  /// Threadblock-level swizzling operator
  typename ThreadblockSwizzle,
  /// Operation performed by GEMM
  typename Operator
>
struct DefaultB2bGemm<
  ElementA, LayoutA, kAlignmentA,
  ElementB, LayoutB, kAlignmentB,
  ElementC, layout::RowMajor,
  ElementAccumulator,
  arch::OpClassTensorOp,
  arch::Sm75,
  ThreadblockShape0,
  ThreadblockShape1,
  WarpShape0,
  WarpShape1,
  InstructionShape,
  EpilogueOutputOp0,
  EpilogueOutputOp1,
  ThreadblockSwizzle,
  2,
  Operator,
  true
> {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps. `InstructionShape` mirrors the hardware MMA instruction tile used by Tensor Cores. `arch::Sm75` targets Turing-generation GPUs. `OpClassTensorOp` selects Tensor Core math instead of plain SIMT arithmetic.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。 `InstructionShape` 对应 Tensor Core 使用的硬件 MMA 指令级分块。 `arch::Sm75` 表示面向 Turing 架构 GPU。 `OpClassTensorOp` 选择 Tensor Core 计算，而不是普通的 SIMT 算术。

### Lines 201-201 / 第201-201行

```cpp
  /// Define the threadblock-scoped matrix multiply-accumulate
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 202-223 / 第202-223行

```cpp
  using B2bMma = typename cutlass::gemm::threadblock::DefaultB2bMma<
    ElementA,
    LayoutA,
    kAlignmentA,
    ElementB,
    LayoutB,
    kAlignmentB,
    ElementAccumulator,
    layout::RowMajor,
    arch::OpClassTensorOp,
    arch::Sm75,
    ThreadblockShape0,
    ThreadblockShape1,
    WarpShape0,
    WarpShape1,
    InstructionShape,
    2,
    Operator,
    EpilogueOutputOp0,
    false,
    true
  >::ThreadblockB2bMma;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps. `InstructionShape` mirrors the hardware MMA instruction tile used by Tensor Cores. `arch::Sm75` targets Turing-generation GPUs. `OpClassTensorOp` selects Tensor Core math instead of plain SIMT arithmetic.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。 `InstructionShape` 对应 Tensor Core 使用的硬件 MMA 指令级分块。 `arch::Sm75` 表示面向 Turing 架构 GPU。 `OpClassTensorOp` 选择 Tensor Core 计算，而不是普通的 SIMT 算术。

### Lines 225-225 / 第225-225行

```cpp
  static const int kPartitionsK1 = ThreadblockShape1::kK / WarpShape1::kK;
```

**EN**: This block defines compile-time constants or static values that parameterize the kernel or the benchmark. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps.  
**CN**: 这段代码定义编译期常量或静态值，用于参数化内核或基准测试。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。

### Lines 227-227 / 第227-227行

```cpp
  /// Define the epilogue
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 228-234 / 第228-234行

```cpp
  using Epilogue = typename cutlass::epilogue::threadblock::DefaultEpilogueTensorOp<
    ThreadblockShape1,
    typename B2bMma::Operator1,
    kPartitionsK1,
    EpilogueOutputOp1,
    EpilogueOutputOp1::kCount
  >::Epilogue;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth. `ThreadblockShape` selects the CTA-sized tile computed by one thread block.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。

### Lines 236-236 / 第236-236行

```cpp
  /// Define the kernel-level GEMM operator.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 237-237 / 第237-237行

```cpp
  using B2bGemmKernel = kernel::B2bGemm<B2bMma, Epilogue, ThreadblockSwizzle>;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。

### Lines 238-238 / 第238-238行

```cpp
};
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 241-241 / 第241-241行

```cpp
/// Partial specialization for Ampere Integer Matrix Multiply Interleaved layout
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 242-283 / 第242-283行

```cpp
template <
    /// Element type for A matrix operand
    typename ElementA,
    /// Access granularity of A matrix in units of elements
    int kAlignmentA,
    /// Element type for B matrix operand
    typename ElementB,
    /// Access granularity of B matrix in units of elements
    int kAlignmentB,
    /// Element type for C and D matrix operands
    typename ElementC,
    /// Threadblock-level tile size (concept: GemmShape)
    typename ThreadblockShape0,
    /// Threadblock-level tile size (concept: GemmShape)
    typename ThreadblockShape1,
    /// Warp-level tile size (concept: GemmShape)
    typename WarpShape0,
    /// Warp-level tile size (concept: GemmShape)
    typename WarpShape1,
    /// Warp-level tile size (concept: GemmShape)
    typename InstructionShape,
    /// Epilogue output operator
    typename EpilogueOutputOp0,
    /// Epilogue output operator
    typename EpilogueOutputOp1,
    /// Threadblock-level swizzling operator
    typename ThreadblockSwizzle,
    /// Number of stages used in the pipelined mainloop
    int Stages,
    /// Number of Interleaved k
    int InterleavedK,
    /// Operation performed by GEMM
    typename Operator>
struct DefaultB2bGemm<
    ElementA, layout::ColumnMajorInterleaved<InterleavedK>, kAlignmentA,
    ElementB, layout::RowMajorInterleaved<InterleavedK>, kAlignmentB,
    ElementC, layout::ColumnMajorInterleaved<InterleavedK>, int32_t,
    arch::OpClassTensorOp, arch::Sm80,
    ThreadblockShape0, ThreadblockShape1, WarpShape0, WarpShape1,
    InstructionShape, EpilogueOutputOp0, EpilogueOutputOp1,
    ThreadblockSwizzle, Stages,
    Operator, true> {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps. `InstructionShape` mirrors the hardware MMA instruction tile used by Tensor Cores. `arch::Sm80` targets Ampere-generation GPUs. `OpClassTensorOp` selects Tensor Core math instead of plain SIMT arithmetic.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。 `InstructionShape` 对应 Tensor Core 使用的硬件 MMA 指令级分块。 `arch::Sm80` 表示面向 Ampere 架构 GPU。 `OpClassTensorOp` 选择 Tensor Core 计算，而不是普通的 SIMT 算术。

### Lines 284-286 / 第284-286行

```cpp
  using LayoutA = layout::ColumnMajorInterleaved<InterleavedK>;
  using LayoutB = layout::RowMajorInterleaved<InterleavedK>;
  using LayoutC = layout::ColumnMajorInterleaved<InterleavedK>;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 288-288 / 第288-288行

```cpp
  using ElementAccumulator = int32_t;
```

**EN**: These aliases pin down the scalar types used for inputs, accumulators, and outputs. Choosing them explicitly matters because CUTLASS can compute in a different precision from storage.  
**CN**: 这些别名固定了输入、累加器和输出使用的标量类型。显式指定它们很重要，因为 CUTLASS 可以让计算精度与存储精度不同。

### Lines 290-290 / 第290-290行

```cpp
  /// Define the threadblock-scoped matrix multiply-accumulate
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 291-296 / 第291-296行

```cpp
  using B2bMma = typename cutlass::gemm::threadblock::DefaultB2bMma<
      ElementA, LayoutA, kAlignmentA, ElementB, LayoutB, kAlignmentB,
      ElementAccumulator, LayoutC, arch::OpClassTensorOp, arch::Sm80,
      ThreadblockShape0, ThreadblockShape1, WarpShape0, WarpShape1,
      InstructionShape, Stages, Operator, EpilogueOutputOp0,
      true, true>::ThreadblockB2bMma;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps. `InstructionShape` mirrors the hardware MMA instruction tile used by Tensor Cores. `arch::Sm80` targets Ampere-generation GPUs. `OpClassTensorOp` selects Tensor Core math instead of plain SIMT arithmetic.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。 `InstructionShape` 对应 Tensor Core 使用的硬件 MMA 指令级分块。 `arch::Sm80` 表示面向 Ampere 架构 GPU。 `OpClassTensorOp` 选择 Tensor Core 计算，而不是普通的 SIMT 算术。

### Lines 298-298 / 第298-298行

```cpp
  static const int kPartitionsK1 = ThreadblockShape1::kK / WarpShape1::kK;
```

**EN**: This block defines compile-time constants or static values that parameterize the kernel or the benchmark. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps.  
**CN**: 这段代码定义编译期常量或静态值，用于参数化内核或基准测试。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。

### Lines 300-300 / 第300-300行

```cpp
  /// Define the epilogue
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 301-304 / 第301-304行

```cpp
  using Epilogue = typename cutlass::epilogue::threadblock::
      DefaultInterleavedEpilogueTensorOp<
          ThreadblockShape1, typename B2bMma::Operator1, kPartitionsK1, EpilogueOutputOp1,
          64 / sizeof_bits<ElementC>::value, InterleavedK>::Epilogue;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth. `ThreadblockShape` selects the CTA-sized tile computed by one thread block.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。

### Lines 306-306 / 第306-306行

```cpp
  /// Define the kernel-level GEMM operator.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 307-307 / 第307-307行

```cpp
  using B2bGemmKernel = kernel::B2bGemm<B2bMma, Epilogue, ThreadblockSwizzle>;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。

### Lines 308-308 / 第308-308行

```cpp
};
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 310-310 / 第310-310行

```cpp
////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 313-313 / 第313-313行

```cpp
/// Partial specialization for Turing Integer Tensor Core Interleaved layout
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 314-352 / 第314-352行

```cpp
template <
    /// Element type for A matrix operand
    typename ElementA,
    /// Access granularity of A matrix in units of elements
    int kAlignmentA,
    /// Element type for B matrix operand
    typename ElementB,
    /// Access granularity of B matrix in units of elements
    int kAlignmentB,
    /// Element type for C and D matrix operands
    typename ElementC,
    /// Threadblock-level tile size (concept: GemmShape)
    typename ThreadblockShape0,
    /// Threadblock-level tile size (concept: GemmShape)
    typename ThreadblockShape1,
    /// Warp-level tile size (concept: GemmShape)
    typename WarpShape0,
    /// Warp-level tile size (concept: GemmShape)
    typename WarpShape1,
    /// Warp-level tile size (concept: GemmShape)
    typename InstructionShape,
    /// Epilogue output operator
    typename EpilogueOutputOp0,
    /// Epilogue output operator
    typename EpilogueOutputOp1,
    /// Threadblock-level swizzling operator
    typename ThreadblockSwizzle,
    /// Number of Interleaved k
    int InterleavedK,
    /// Operation performed by GEMM
    typename Operator>
struct DefaultB2bGemm<ElementA, layout::ColumnMajorInterleaved<InterleavedK>,
                   kAlignmentA, ElementB,
                   layout::RowMajorInterleaved<InterleavedK>, kAlignmentB,
                   ElementC, layout::ColumnMajorInterleaved<InterleavedK>,
                   int32_t, arch::OpClassTensorOp, arch::Sm75,
                   ThreadblockShape0, ThreadblockShape1, WarpShape0, WarpShape1,
                   InstructionShape, EpilogueOutputOp0, EpilogueOutputOp1,
                   ThreadblockSwizzle, 2, Operator, true> {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps. `InstructionShape` mirrors the hardware MMA instruction tile used by Tensor Cores. `arch::Sm75` targets Turing-generation GPUs. `OpClassTensorOp` selects Tensor Core math instead of plain SIMT arithmetic.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。 `InstructionShape` 对应 Tensor Core 使用的硬件 MMA 指令级分块。 `arch::Sm75` 表示面向 Turing 架构 GPU。 `OpClassTensorOp` 选择 Tensor Core 计算，而不是普通的 SIMT 算术。

### Lines 353-355 / 第353-355行

```cpp
  using LayoutA = layout::ColumnMajorInterleaved<InterleavedK>;
  using LayoutB = layout::RowMajorInterleaved<InterleavedK>;
  using LayoutC = layout::ColumnMajorInterleaved<InterleavedK>;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 357-357 / 第357-357行

```cpp
  using ElementAccumulator = int32_t;
```

**EN**: These aliases pin down the scalar types used for inputs, accumulators, and outputs. Choosing them explicitly matters because CUTLASS can compute in a different precision from storage.  
**CN**: 这些别名固定了输入、累加器和输出使用的标量类型。显式指定它们很重要，因为 CUTLASS 可以让计算精度与存储精度不同。

### Lines 359-359 / 第359-359行

```cpp
  /// Define the threadblock-scoped matrix multiply-accumulate
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 360-364 / 第360-364行

```cpp
  using B2bMma = typename cutlass::gemm::threadblock::DefaultB2bMma<
      ElementA, LayoutA, kAlignmentA, ElementB, LayoutB, kAlignmentB,
      ElementAccumulator, LayoutC, arch::OpClassTensorOp, arch::Sm75,
      ThreadblockShape0, ThreadblockShape1, WarpShape0, WarpShape1,
      InstructionShape, 2, Operator, EpilogueOutputOp0, true, true>::ThreadblockB2bMma;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps. `InstructionShape` mirrors the hardware MMA instruction tile used by Tensor Cores. `arch::Sm75` targets Turing-generation GPUs. `OpClassTensorOp` selects Tensor Core math instead of plain SIMT arithmetic.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。 `InstructionShape` 对应 Tensor Core 使用的硬件 MMA 指令级分块。 `arch::Sm75` 表示面向 Turing 架构 GPU。 `OpClassTensorOp` 选择 Tensor Core 计算，而不是普通的 SIMT 算术。

### Lines 366-366 / 第366-366行

```cpp
  static const int kPartitionsK1 = ThreadblockShape1::kK / WarpShape1::kK;
```

**EN**: This block defines compile-time constants or static values that parameterize the kernel or the benchmark. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps.  
**CN**: 这段代码定义编译期常量或静态值，用于参数化内核或基准测试。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。

### Lines 368-368 / 第368-368行

```cpp
  /// Define the epilogue for the 2nd Gemm
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 369-372 / 第369-372行

```cpp
  using Epilogue = typename cutlass::epilogue::threadblock::
      DefaultInterleavedEpilogueTensorOp<
          ThreadblockShape1, typename B2bMma::Operator1, kPartitionsK1, EpilogueOutputOp1,
          64 / sizeof_bits<ElementC>::value, InterleavedK>::Epilogue;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth. `ThreadblockShape` selects the CTA-sized tile computed by one thread block.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。

### Lines 374-374 / 第374-374行

```cpp
  /// Define the kernel-level GEMM operator.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 375-375 / 第375-375行

```cpp
  using B2bGemmKernel = kernel::B2bGemm<B2bMma, Epilogue, ThreadblockSwizzle>;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。

### Lines 376-376 / 第376-376行

```cpp
};
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 378-378 / 第378-378行

```cpp
////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 380-380 / 第380-380行

```cpp
////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 382-384 / 第382-384行

```cpp
}  // namespace kernel
}  // namespace gemm
}  // namespace cutlass
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

---

## Key Concepts / 关键概念

- Back-to-back GEMM fusion / 前后相接 GEMM 融合
- Hierarchical tiling: threadblock / warp / instruction / 分层分块：threadblock / warp / instruction
- Epilogue fusion and output operators / Epilogue 融合与输出算子
- Tensor Core execution / Tensor Core 执行

## Dependencies / 依赖项

- `"cutlass/cutlass.h"` — Core CUTLASS definitions, architecture tags, and status types. / CUTLASS 的核心定义、架构标签与状态类型。
- `"cutlass/layout/matrix.h"` — Layout descriptors that define how tensors are mapped in memory. / 描述张量内存映射方式的布局描述符。
- `"cutlass/numeric_types.h"` — CUTLASS numeric scalar and packed numeric type definitions. / CUTLASS 数值标量与打包数值类型定义。
- `"cutlass/epilogue/threadblock/epilogue.h"` — Epilogue components that convert accumulators to outputs and optionally fuse extra work. / epilogue 组件，用于把累加器转换为输出并可选地融合额外计算。
- `"cutlass/epilogue/thread/linear_combination.h"` — Epilogue components that convert accumulators to outputs and optionally fuse extra work. / epilogue 组件，用于把累加器转换为输出并可选地融合额外计算。
- `"cutlass/gemm/gemm.h"` — Provides `cutlass/gemm/gemm.h` so this file can use the related API or helper utilities. / 提供 `cutlass/gemm/gemm.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/gemm/kernel/gemm_pipelined.h"` — Kernel-level GEMM building blocks and default kernel assemblers. / 内核级 GEMM 构建模块与默认内核拼装器。
- `"cutlass/gemm/threadblock/default_mma_core_sm75.h"` — Threadblock-scoped GEMM machinery such as MMA cores and swizzles. / threadblock 级 GEMM 组件，例如 MMA core 与 swizzle。
- `"cutlass/gemm/threadblock/default_mma_core_sm70.h"` — Threadblock-scoped GEMM machinery such as MMA cores and swizzles. / threadblock 级 GEMM 组件，例如 MMA core 与 swizzle。
- `"cutlass/gemm/threadblock/default_mma_core_sm80.h"` — Threadblock-scoped GEMM machinery such as MMA cores and swizzles. / threadblock 级 GEMM 组件，例如 MMA core 与 swizzle。
- `"cutlass/gemm/threadblock/default_mma_core_simt.h"` — Threadblock-scoped GEMM machinery such as MMA cores and swizzles. / threadblock 级 GEMM 组件，例如 MMA core 与 swizzle。
- `"cutlass/gemm/threadblock/threadblock_swizzle.h"` — Threadblock-scoped GEMM machinery such as MMA cores and swizzles. / threadblock 级 GEMM 组件，例如 MMA core 与 swizzle。
- `"cutlass/epilogue/threadblock/default_epilogue_tensor_op.h"` — Epilogue components that convert accumulators to outputs and optionally fuse extra work. / epilogue 组件，用于把累加器转换为输出并可选地融合额外计算。
- `"cutlass/epilogue/threadblock/default_epilogue_volta_tensor_op.h"` — Epilogue components that convert accumulators to outputs and optionally fuse extra work. / epilogue 组件，用于把累加器转换为输出并可选地融合额外计算。
- `"cutlass/epilogue/threadblock/default_epilogue_simt.h"` — Epilogue components that convert accumulators to outputs and optionally fuse extra work. / epilogue 组件，用于把累加器转换为输出并可选地融合额外计算。
- `"cutlass/transform/threadblock/predicated_tile_iterator.h"` — Iterator and transform utilities that move tensor tiles efficiently. / 用于高效搬运张量分块的迭代器与变换工具。
- `"cutlass/transform/threadblock/vector_iterator.h"` — Iterator and transform utilities that move tensor tiles efficiently. / 用于高效搬运张量分块的迭代器与变换工具。
- `"cutlass/transform/threadblock/predicated_vector_access_iterator.h"` — Iterator and transform utilities that move tensor tiles efficiently. / 用于高效搬运张量分块的迭代器与变换工具。
- `"kernel/b2b_gemm.h"` — Example-local kernel definition used to compose specialized fused execution paths. / 示例本地内核定义，用于组合专门化的融合执行路径。
- `"threadblock/default_b2b_mma.h"` — Example-local threadblock policy or pipeline component. / 示例本地 threadblock 策略或流水线组件。
- `"threadblock/default_b2b_mma_smem_accumulator.h"` — Example-local threadblock policy or pipeline component. / 示例本地 threadblock 策略或流水线组件。
