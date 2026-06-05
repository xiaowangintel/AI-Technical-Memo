# b2b_gemm.h — Code Analysis / 代码分析

**Source / 源文件**: `examples/13_two_tensor_op_fusion/device/b2b_gemm.h`  
**Purpose / 用途**: Implements the device-level wrapper that packages, launches, and validates fused back-to-back GEMM kernels. / 实现设备级封装，用于打包、启动并校验融合双 GEMM 内核。

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
    \brief Template for a pipelined GEMM kernel. Does not compute batching or support split-K.
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
#include "cutlass/numeric_types.h"
#include "cutlass/arch/arch.h"
#include "cutlass/device_kernel.h"
```

**EN**: These headers pull in the building blocks required by this file. Core CUTLASS definitions, architecture tags, and status types. CUTLASS numeric scalar and packed numeric type definitions. Provides `cutlass/arch/arch.h` so this file can use the related API or helper utilities. Provides `cutlass/device_kernel.h` so this file can use the related API or helper utilities.  
**CN**: 这些头文件引入了当前文件所需的构建模块。CUTLASS 的核心定义、架构标签与状态类型。CUTLASS 数值标量与打包数值类型定义。提供 `cutlass/arch/arch.h`，使本文件能够使用相关 API 或辅助工具。提供 `cutlass/device_kernel.h`，使本文件能够使用相关 API 或辅助工具。

### Lines 42-42 / 第42-42行

```cpp
#include "cutlass/gemm/threadblock/threadblock_swizzle.h"
```

**EN**: These headers pull in the building blocks required by this file. Threadblock-scoped GEMM machinery such as MMA cores and swizzles.  
**CN**: 这些头文件引入了当前文件所需的构建模块。threadblock 级 GEMM 组件，例如 MMA core 与 swizzle。

### Lines 44-45 / 第44-45行

```cpp
#include "cutlass/gemm/device/default_gemm_configuration.h"
#include "cutlass/epilogue/thread/linear_combination_relu.h"
```

**EN**: These headers pull in the building blocks required by this file. Device-level GEMM wrapper that hides kernel selection and launch details. Epilogue components that convert accumulators to outputs and optionally fuse extra work.  
**CN**: 这些头文件引入了当前文件所需的构建模块。设备级 GEMM 封装，隐藏内核选择与启动细节。epilogue 组件，用于把累加器转换为输出并可选地融合额外计算。

### Lines 47-49 / 第47-49行

```cpp
#include "kernel/b2b_gemm.h"
#include "kernel/default_b2b_gemm.h"
#include "kernel/default_b2b_gemm_smem_accumulator.h"
```

**EN**: These headers pull in the building blocks required by this file. Example-local kernel definition used to compose specialized fused execution paths. Example-local kernel definition used to compose specialized fused execution paths. Example-local kernel definition used to compose specialized fused execution paths.  
**CN**: 这些头文件引入了当前文件所需的构建模块。示例本地内核定义，用于组合专门化的融合执行路径。示例本地内核定义，用于组合专门化的融合执行路径。示例本地内核定义，用于组合专门化的融合执行路径。

### Lines 51-51 / 第51-51行

```cpp
////////////////////////////////////////////////////////////////////////////////
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
namespace gemm {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 55-55 / 第55-55行

```cpp
namespace device {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 57-57 / 第57-57行

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 59-126 / 第59-126行

```cpp
template <
    /// Element type for A matrix operand
    typename ElementA_,
    /// Layout type for A matrix operand
    typename LayoutA_,
    /// Element type for B matrix operand
    typename ElementB_,
    /// Layout type for B matrix operand
    typename LayoutB_,
    /// Element type for C and D matrix operands
    typename ElementC_,
    /// Layout type for C and D matrix operands
    typename LayoutC_,
    /// Element type for internal accumulation
    typename ElementAccumulator_ = ElementC_,
    /// Operator class tag
    typename OperatorClass_ = arch::OpClassSimt,
    /// Tag indicating architecture to tune for
    typename ArchTag_ = arch::Sm70,
    /// Threadblock-level tile size (concept: GemmShape)
    typename ThreadblockShape0_ = typename DefaultGemmConfiguration<
        OperatorClass_, ArchTag_, ElementA_, ElementB_, ElementC_,
        ElementAccumulator_>::ThreadblockShape,
    /// Threadblock-level tile size (concept: GemmShape)
    typename ThreadblockShape1_ = typename DefaultGemmConfiguration<
        OperatorClass_, ArchTag_, ElementA_, ElementB_, ElementC_,
        ElementAccumulator_>::ThreadblockShape,
    /// Warp-level tile size (concept: GemmShape)
    typename WarpShape0_ = typename DefaultGemmConfiguration<
        OperatorClass_, ArchTag_, ElementA_, ElementB_, ElementC_,
        ElementAccumulator_>::WarpShape,
    /// Warp-level tile size (concept: GemmShape)
    typename WarpShape1_ = typename DefaultGemmConfiguration<
        OperatorClass_, ArchTag_, ElementA_, ElementB_, ElementC_,
        ElementAccumulator_>::WarpShape,
    /// Instruction-level tile size (concept: GemmShape)
    typename InstructionShape_ = typename DefaultGemmConfiguration<
        OperatorClass_, ArchTag_, ElementA_, ElementB_, ElementC_,
        ElementAccumulator_>::InstructionShape,
    /// Epilogue output operator
    typename EpilogueOutputOp0_ = typename DefaultGemmConfiguration<
        OperatorClass_, ArchTag_, ElementA_, ElementB_, ElementC_,
        ElementAccumulator_>::EpilogueOutputOp,
    /// Epilogue output operator
    typename EpilogueOutputOp1_ = typename DefaultGemmConfiguration<
        OperatorClass_, ArchTag_, ElementA_, ElementB_, ElementC_,
        ElementAccumulator_>::EpilogueOutputOp,
    /// Threadblock-level swizzling operator
    typename ThreadblockSwizzle_ = threadblock::GemmIdentityThreadblockSwizzle<>,
    /// Number of stages used in the pipelined mainloop
    int Stages =
        DefaultGemmConfiguration<OperatorClass_, ArchTag_, ElementA_, ElementB_,
                                 ElementC_, ElementAccumulator_>::kStages,
    /// Stage accumulator in shared memory
    bool SmemAccumulator = false,
    /// Access granularity of A matrix in units of elements
    int AlignmentA =
        DefaultGemmConfiguration<OperatorClass_, ArchTag_, ElementA_, ElementB_,
                                 ElementC_, ElementAccumulator_>::kAlignmentA,
    /// Access granularity of B matrix in units of elements
    int AlignmentB =
        DefaultGemmConfiguration<OperatorClass_, ArchTag_, ElementA_, ElementB_,
                                 ElementC_, ElementAccumulator_>::kAlignmentB,
    /// Operation performed by GEMM
    typename Operator_ = typename DefaultGemmConfiguration<
        OperatorClass_, ArchTag_, ElementA_, ElementB_, ElementC_,
        ElementAccumulator_>::Operator>
class B2bGemm {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps. `InstructionShape` mirrors the hardware MMA instruction tile used by Tensor Cores. `OpClassSimt` keeps the kernel on standard CUDA cores instead of Tensor Cores. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。 `InstructionShape` 对应 Tensor Core 使用的硬件 MMA 指令级分块。 `OpClassSimt` 表示该内核使用标准 CUDA Core，而不是 Tensor Core。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 127-127 / 第127-127行

```cpp
 public:
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 129-129 / 第129-129行

```cpp
  using ElementA = ElementA_;
```

**EN**: These aliases pin down the scalar types used for inputs, accumulators, and outputs. Choosing them explicitly matters because CUTLASS can compute in a different precision from storage.  
**CN**: 这些别名固定了输入、累加器和输出使用的标量类型。显式指定它们很重要，因为 CUTLASS 可以让计算精度与存储精度不同。

### Lines 130-130 / 第130-130行

```cpp
  using LayoutA = LayoutA_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 131-132 / 第131-132行

```cpp
  using TensorRefA = TensorRef<ElementA const, LayoutA>;
  using ElementB = ElementB_;
```

**EN**: These aliases pin down the scalar types used for inputs, accumulators, and outputs. Choosing them explicitly matters because CUTLASS can compute in a different precision from storage.  
**CN**: 这些别名固定了输入、累加器和输出使用的标量类型。显式指定它们很重要，因为 CUTLASS 可以让计算精度与存储精度不同。

### Lines 133-133 / 第133-133行

```cpp
  using LayoutB = LayoutB_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 134-135 / 第134-135行

```cpp
  using TensorRefB = TensorRef<ElementB const, LayoutB>;
  using ElementC = ElementC_;
```

**EN**: These aliases pin down the scalar types used for inputs, accumulators, and outputs. Choosing them explicitly matters because CUTLASS can compute in a different precision from storage.  
**CN**: 这些别名固定了输入、累加器和输出使用的标量类型。显式指定它们很重要，因为 CUTLASS 可以让计算精度与存储精度不同。

### Lines 136-136 / 第136-136行

```cpp
  using LayoutC = LayoutC_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 137-139 / 第137-139行

```cpp
  using TensorRefC = TensorRef<ElementC const, LayoutC>;
  using TensorRefD = TensorRef<ElementC, LayoutC>;
  using ElementAccumulator = ElementAccumulator_;
```

**EN**: These aliases pin down the scalar types used for inputs, accumulators, and outputs. Choosing them explicitly matters because CUTLASS can compute in a different precision from storage.  
**CN**: 这些别名固定了输入、累加器和输出使用的标量类型。显式指定它们很重要，因为 CUTLASS 可以让计算精度与存储精度不同。

### Lines 140-146 / 第140-146行

```cpp
  using OperatorClass = OperatorClass_;
  using ArchTag = ArchTag_;
  using ThreadblockShape0 = ThreadblockShape0_;
  using ThreadblockShape1 = ThreadblockShape1_;
  using WarpShape0 = WarpShape0_;
  using WarpShape1 = WarpShape1_;
  using InstructionShape = InstructionShape_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps. `InstructionShape` mirrors the hardware MMA instruction tile used by Tensor Cores.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。 `InstructionShape` 对应 Tensor Core 使用的硬件 MMA 指令级分块。

### Lines 147-147 / 第147-147行

```cpp
  using EpilogueOutputOp0 = EpilogueOutputOp0_;
```

**EN**: This block defines epilogue functors. In CUTLASS, the epilogue converts accumulators into the output type and can fuse extra work such as scaling, bias handling, clamping, or activation.  
**CN**: 这段代码定义了 epilogue functor。在 CUTLASS 中，epilogue 负责把累加器转换为输出类型，并可融合缩放、偏置处理、截断或激活等附加工作。

### Lines 148-148 / 第148-148行

```cpp
  using EpilogueOutputOp1 = EpilogueOutputOp1_;
```

**EN**: This block defines epilogue functors. In CUTLASS, the epilogue converts accumulators into the output type and can fuse extra work such as scaling, bias handling, clamping, or activation.  
**CN**: 这段代码定义了 epilogue functor。在 CUTLASS 中，epilogue 负责把累加器转换为输出类型，并可融合缩放、偏置处理、截断或激活等附加工作。

### Lines 149-150 / 第149-150行

```cpp
  using ThreadblockSwizzle = ThreadblockSwizzle_;
  using Operator = Operator_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 151-156 / 第151-156行

```cpp
  static int const kStages = Stages;
  static int const kAlignmentA = AlignmentA;
  static int const kAlignmentB = AlignmentB;
  static int const kAlignmentC = EpilogueOutputOp1::kCount;
  static ComplexTransform const kTransformA = ComplexTransform::kNone;
  static ComplexTransform const kTransformB = ComplexTransform::kNone;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 158-158 / 第158-158行

```cpp
  /// Derived types
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 159-159 / 第159-159行

```cpp
  using ElementScaleBias = typename EpilogueOutputOp0::ElementCompute;
```

**EN**: This block defines epilogue functors. In CUTLASS, the epilogue converts accumulators into the output type and can fuse extra work such as scaling, bias handling, clamping, or activation.  
**CN**: 这段代码定义了 epilogue functor。在 CUTLASS 中，epilogue 负责把累加器转换为输出类型，并可融合缩放、偏置处理、截断或激活等附加工作。

### Lines 160-160 / 第160-160行

```cpp
  using LayoutScaleBias = layout::RowMajor;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 162-162 / 第162-162行

```cpp
  /// Define the kernel
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 163-186 / 第163-186行

```cpp
  using B2bGemmKernel = typename kernel::DefaultB2bGemm<
    ElementA,
    LayoutA,
    kAlignmentA,
    ElementB,
    LayoutB,
    kAlignmentB,
    ElementC,
    LayoutC,
    ElementAccumulator,
    OperatorClass,
    ArchTag,
    ThreadblockShape0,
    ThreadblockShape1,
    WarpShape0,
    WarpShape1,
    InstructionShape,
    EpilogueOutputOp0,
    EpilogueOutputOp1,
    ThreadblockSwizzle,
    kStages,
    Operator,
    SmemAccumulator
  >::B2bGemmKernel;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps. `InstructionShape` mirrors the hardware MMA instruction tile used by Tensor Cores. The `Default*` meta-program chooses architecture-specific iterators, MMA cores, and epilogues for the requested tile policy. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。 `InstructionShape` 对应 Tensor Core 使用的硬件 MMA 指令级分块。 `Default*` 元编程构件会根据所请求的分块策略选择架构相关的迭代器、MMA core 与 epilogue。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 188-188 / 第188-188行

```cpp
  using Arguments = typename B2bGemmKernel::Arguments;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。

### Lines 190-190 / 第190-190行

```cpp
private:
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 192-192 / 第192-192行

```cpp
  /// Kernel parameters object
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 193-193 / 第193-193行

```cpp
  typename B2bGemmKernel::Params params_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 195-195 / 第195-195行

```cpp
public:
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 197-197 / 第197-197行

```cpp
  /// Constructs the GEMM.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 198-198 / 第198-198行

```cpp
  B2bGemm() { }
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 200-200 / 第200-200行

```cpp
  /// Determines whether the GEMM can execute the given problem.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 201-201 / 第201-201行

```cpp
  static Status can_implement(Arguments const &args) {
```

**EN**: This line starts a function or callable scope. The body that follows typically configures a kernel, launches it, or checks its result.  
**CN**: 这一行开始定义一个函数或可调用作用域。随后的函数体通常会配置内核、启动执行或检查结果。

### Lines 203-212 / 第203-212行

```cpp
    Status status = B2bGemmKernel::can_implement(
      args.problem_size_0,
      args.problem_size_1,
      args.ref_A0.non_const_ref(),
      args.ref_B0.non_const_ref(),
      args.ref_C0.non_const_ref(),
      args.ref_B1.non_const_ref(),
      args.ref_C1.non_const_ref(),
      args.ref_D1
    );
```

**EN**: This block exercises the configured CUTLASS object at runtime. Typical calls include initialization, capability checks, workspace queries, kernel launches, or reference validation.  
**CN**: 这段代码在运行时驱动已配置好的 CUTLASS 对象。常见调用包括初始化、能力检查、工作区查询、内核启动以及参考结果验证。

### Lines 214-215 / 第214-215行

```cpp
    if (status != Status::kSuccess) {
      return status;
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 216-216 / 第216-216行

```cpp
    }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 218-218 / 第218-218行

```cpp
    return Status::kSuccess;
```

**EN**: This return statement propagates the current result back to the caller so the surrounding harness can continue its control flow.  
**CN**: 这个返回语句会把当前结果传回调用者，使外围测试框架能够继续执行后续控制流程。

### Lines 219-219 / 第219-219行

```cpp
  }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 221-221 / 第221-221行

```cpp
  /// Gets the workspace size
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 222-222 / 第222-222行

```cpp
  static size_t get_workspace_size(Arguments const &args) {
```

**EN**: This line starts a function or callable scope. The body that follows typically configures a kernel, launches it, or checks its result.  
**CN**: 这一行开始定义一个函数或可调用作用域。随后的函数体通常会配置内核、启动执行或检查结果。

### Lines 224-224 / 第224-224行

```cpp
    size_t bytes = 0;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 226-226 / 第226-226行

```cpp
    // Determine grid shape
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 227-227 / 第227-227行

```cpp
    ThreadblockSwizzle threadblock_swizzle;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 229-232 / 第229-232行

```cpp
    cutlass::gemm::GemmCoord tiled_shape = threadblock_swizzle.get_tiled_shape(
      args.problem_size_0,
      {ThreadblockShape0::kM, ThreadblockShape0::kN, ThreadblockShape0::kK},
      args.batch_count);
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps. `ThreadblockShape` selects the CTA-sized tile computed by one thread block.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。

### Lines 234-234 / 第234-234行

```cpp
    return bytes;
```

**EN**: This return statement propagates the current result back to the caller so the surrounding harness can continue its control flow.  
**CN**: 这个返回语句会把当前结果传回调用者，使外围测试框架能够继续执行后续控制流程。

### Lines 235-235 / 第235-235行

```cpp
  }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 237-237 / 第237-237行

```cpp
  /// Initializes GEMM state from arguments.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 238-240 / 第238-240行

```cpp
  Status initialize(Arguments const &args, void *workspace = nullptr, cudaStream_t stream = nullptr) {
    // Determine grid shape
    ThreadblockSwizzle threadblock_swizzle;
```

**EN**: This block exercises the configured CUTLASS object at runtime. Typical calls include initialization, capability checks, workspace queries, kernel launches, or reference validation.  
**CN**: 这段代码在运行时驱动已配置好的 CUTLASS 对象。常见调用包括初始化、能力检查、工作区查询、内核启动以及参考结果验证。

### Lines 243-246 / 第243-246行

```cpp
    cutlass::gemm::GemmCoord grid_shape = threadblock_swizzle.get_tiled_shape(
      args.problem_size_0,
      {ThreadblockShape0::kM, ThreadblockShape0::kN, ThreadblockShape0::kK},
      args.batch_count);
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps. `ThreadblockShape` selects the CTA-sized tile computed by one thread block.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。

### Lines 247-250 / 第247-250行

```cpp
//    cutlass::gemm::GemmCoord grid_shape_1 = threadblock_swizzle.get_tiled_shape(
//      args.problem_size_1,
//      {ThreadblockShape1::kM, ThreadblockShape1::kN, ThreadblockShape1::kK},
//      args.batch_count);
```

**EN**: This comment separates sections and documents the purpose of the code that follows. `ThreadblockShape` selects the CTA-sized tile computed by one thread block.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。

### Lines 252-252 / 第252-252行

```cpp
    // Initialize the Params structure
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 253-276 / 第253-276行

```cpp
    params_ = typename B2bGemmKernel::Params{
      args.mode,
      args.problem_size_0,
      args.problem_size_1,
      grid_shape,
      args.ref_A0.non_const_ref(),
      args.ref_B0.non_const_ref(),
      args.ref_C0.non_const_ref(),
      args.ref_Scale0.non_const_ref(),
      args.ref_Bias0.non_const_ref(),
      args.ref_B1.non_const_ref(),
      args.ref_C1.non_const_ref(),
      args.ref_D1,
      args.batch_stride_A0,
      args.batch_stride_B0,
      args.batch_stride_B1,
      args.batch_stride_C1,
      args.batch_stride_D1,
      args.batch_stride_Bias0,
      args.batch_stride_Scale0,
      args.epilogue0,
      args.epilogue1,
      static_cast<int *>(workspace),
    };
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 278-278 / 第278-278行

```cpp
    return Status::kSuccess;
```

**EN**: This return statement propagates the current result back to the caller so the surrounding harness can continue its control flow.  
**CN**: 这个返回语句会把当前结果传回调用者，使外围测试框架能够继续执行后续控制流程。

### Lines 279-279 / 第279-279行

```cpp
  }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 281-281 / 第281-281行

```cpp
  /// Lightweight update given a subset of arguments
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 282-283 / 第282-283行

```cpp
  Status update(Arguments const &args, void *workspace = nullptr) {
    params_.ref_A0.reset(args.ref_A0.non_const_ref().data());
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 285-285 / 第285-285行

```cpp
    params_.ref_B0.reset(args.ref_B0.non_const_ref().data());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 286-286 / 第286-286行

```cpp
    params_.ref_C0.reset(args.ref_C0.non_const_ref().data());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 287-287 / 第287-287行

```cpp
    params_.ref_Scale0.reset(args.ref_Scale0.non_const_ref().data());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 288-288 / 第288-288行

```cpp
    params_.ref_Bias0.reset(args.ref_Bias0.non_const_ref().data());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 289-289 / 第289-289行

```cpp
    params_.ref_B1.reset(args.ref_B1.non_const_ref().data());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 290-290 / 第290-290行

```cpp
    params_.ref_C1.reset(args.ref_C1.non_const_ref().data());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 291-291 / 第291-291行

```cpp
    params_.ref_D1.reset(args.ref_D1.data());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 292-294 / 第292-294行

```cpp
    params_.output_op_0 = args.epilogue0;
    params_.output_op_1 = args.epilogue1;
    params_.semaphore = static_cast<int *>(workspace);
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 296-296 / 第296-296行

```cpp
    return Status::kSuccess;
```

**EN**: This return statement propagates the current result back to the caller so the surrounding harness can continue its control flow.  
**CN**: 这个返回语句会把当前结果传回调用者，使外围测试框架能够继续执行后续控制流程。

### Lines 297-297 / 第297-297行

```cpp
  }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 299-299 / 第299-299行

```cpp
  /// Runs the kernel using initialized state.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 300-301 / 第300-301行

```cpp
  Status run(cudaStream_t stream = nullptr) {
    ThreadblockSwizzle threadblock_swizzle;
```

**EN**: This block exercises the configured CUTLASS object at runtime. Typical calls include initialization, capability checks, workspace queries, kernel launches, or reference validation.  
**CN**: 这段代码在运行时驱动已配置好的 CUTLASS 对象。常见调用包括初始化、能力检查、工作区查询、内核启动以及参考结果验证。

### Lines 304-304 / 第304-304行

```cpp
    dim3 grid = threadblock_swizzle.get_grid_shape(params_.grid_tiled_shape);
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 305-305 / 第305-305行

```cpp
    dim3 block(B2bGemmKernel::kThreadCount, 1, 1);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 307-307 / 第307-307行

```cpp
    cudaError_t result;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 309-309 / 第309-309行

```cpp
    int smem_size = int(sizeof(typename B2bGemmKernel::SharedStorage));
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 310-313 / 第310-313行

```cpp
    if (smem_size >= (48 << 10)) {
      result = cudaFuncSetAttribute(Kernel<B2bGemmKernel>,
                                    cudaFuncAttributeMaxDynamicSharedMemorySize,
                                    smem_size);
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 315-316 / 第315-316行

```cpp
      if (result != cudaSuccess) {
        return Status::kErrorInternal;
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 317-317 / 第317-317行

```cpp
      }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 318-318 / 第318-318行

```cpp
    }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 320-320 / 第320-320行

```cpp
    cutlass::Kernel<B2bGemmKernel><<<grid, block, smem_size, stream>>>(params_);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 322-322 / 第322-322行

```cpp
    result = cudaGetLastError();
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 324-324 / 第324-324行

```cpp
    return result == cudaSuccess ? Status::kSuccess : Status::kErrorInternal;
```

**EN**: This return statement propagates the current result back to the caller so the surrounding harness can continue its control flow.  
**CN**: 这个返回语句会把当前结果传回调用者，使外围测试框架能够继续执行后续控制流程。

### Lines 325-325 / 第325-325行

```cpp
  }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 327-327 / 第327-327行

```cpp
  /// Runs the kernel using initialized state.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 328-329 / 第328-329行

```cpp
  Status operator()(cudaStream_t stream = nullptr) {
    return run(stream);
```

**EN**: This block exercises the configured CUTLASS object at runtime. Typical calls include initialization, capability checks, workspace queries, kernel launches, or reference validation.  
**CN**: 这段代码在运行时驱动已配置好的 CUTLASS 对象。常见调用包括初始化、能力检查、工作区查询、内核启动以及参考结果验证。

### Lines 330-330 / 第330-330行

```cpp
  }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 332-332 / 第332-332行

```cpp
  /// Runs the kernel using initialized state.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 333-337 / 第333-337行

```cpp
  Status operator()(
    Arguments const &args,
    void *workspace = nullptr,
    cudaStream_t stream = nullptr) {
    Status status = initialize(args, workspace, stream);
```

**EN**: This block exercises the configured CUTLASS object at runtime. Typical calls include initialization, capability checks, workspace queries, kernel launches, or reference validation.  
**CN**: 这段代码在运行时驱动已配置好的 CUTLASS 对象。常见调用包括初始化、能力检查、工作区查询、内核启动以及参考结果验证。

### Lines 340-341 / 第340-341行

```cpp
    if (status == Status::kSuccess) {
      status = run(stream);
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 342-342 / 第342-342行

```cpp
    }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 344-344 / 第344-344行

```cpp
    return status;
```

**EN**: This return statement propagates the current result back to the caller so the surrounding harness can continue its control flow.  
**CN**: 这个返回语句会把当前结果传回调用者，使外围测试框架能够继续执行后续控制流程。

### Lines 345-345 / 第345-345行

```cpp
  }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 346-346 / 第346-346行

```cpp
};
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 348-350 / 第348-350行

```cpp
} // namespace device
} // namespace gemm
} // namespace cutlass
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 352-352 / 第352-352行

```cpp
////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

---

## Key Concepts / 关键概念

- Back-to-back GEMM fusion / 前后相接 GEMM 融合
- Hierarchical tiling: threadblock / warp / instruction / 分层分块：threadblock / warp / instruction
- Epilogue fusion and output operators / Epilogue 融合与输出算子
- SIMT kernel construction / SIMT 内核构建
- Shared-memory accumulator staging / 共享内存累加器暂存

## Dependencies / 依赖项

- `"cutlass/cutlass.h"` — Core CUTLASS definitions, architecture tags, and status types. / CUTLASS 的核心定义、架构标签与状态类型。
- `"cutlass/numeric_types.h"` — CUTLASS numeric scalar and packed numeric type definitions. / CUTLASS 数值标量与打包数值类型定义。
- `"cutlass/arch/arch.h"` — Provides `cutlass/arch/arch.h` so this file can use the related API or helper utilities. / 提供 `cutlass/arch/arch.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/device_kernel.h"` — Provides `cutlass/device_kernel.h` so this file can use the related API or helper utilities. / 提供 `cutlass/device_kernel.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/gemm/threadblock/threadblock_swizzle.h"` — Threadblock-scoped GEMM machinery such as MMA cores and swizzles. / threadblock 级 GEMM 组件，例如 MMA core 与 swizzle。
- `"cutlass/gemm/device/default_gemm_configuration.h"` — Device-level GEMM wrapper that hides kernel selection and launch details. / 设备级 GEMM 封装，隐藏内核选择与启动细节。
- `"cutlass/epilogue/thread/linear_combination_relu.h"` — Epilogue components that convert accumulators to outputs and optionally fuse extra work. / epilogue 组件，用于把累加器转换为输出并可选地融合额外计算。
- `"kernel/b2b_gemm.h"` — Example-local kernel definition used to compose specialized fused execution paths. / 示例本地内核定义，用于组合专门化的融合执行路径。
- `"kernel/default_b2b_gemm.h"` — Example-local kernel definition used to compose specialized fused execution paths. / 示例本地内核定义，用于组合专门化的融合执行路径。
- `"kernel/default_b2b_gemm_smem_accumulator.h"` — Example-local kernel definition used to compose specialized fused execution paths. / 示例本地内核定义，用于组合专门化的融合执行路径。
