# fused_two_gemms_s8_sm80_shmem.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/13_two_tensor_op_fusion/fused_two_gemms_s8_sm80_shmem.cu`  
**Purpose / 用途**: Compares non-fused and fused back-to-back GEMM on SM80 with INT8 inputs and shared-memory accumulator residency. / 对比在 SM80 上使用 INT8 输入、采用共享内存累加器驻留的非融合与融合双 GEMM。

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

### Lines 31-31 / 第31-31行

```cpp
#include <iostream>
```

**EN**: These headers pull in the building blocks required by this file. Console stream utilities for logging progress and results.  
**CN**: 这些头文件引入了当前文件所需的构建模块。用于记录进度和结果的控制台流工具。

### Lines 33-34 / 第33-34行

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/gemm/device/gemm.h"
```

**EN**: These headers pull in the building blocks required by this file. Core CUTLASS definitions, architecture tags, and status types. Device-level GEMM wrapper that hides kernel selection and launch details.  
**CN**: 这些头文件引入了当前文件所需的构建模块。CUTLASS 的核心定义、架构标签与状态类型。设备级 GEMM 封装，隐藏内核选择与启动细节。

### Lines 36-41 / 第36-41行

```cpp
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/gemm.h"
```

**EN**: These headers pull in the building blocks required by this file. Host/device tensor wrapper used to allocate storage and transfer data. Tensor printing helpers for debugging layouts and values. Host-side reference implementation used for correctness checking. Host-side reference implementation used for correctness checking.  
**CN**: 这些头文件引入了当前文件所需的构建模块。主机/设备张量封装，用于分配存储并传输数据。用于调试布局和值的张量打印辅助工具。用于正确性检查的主机端参考实现。用于正确性检查的主机端参考实现。

### Lines 43-45 / 第43-45行

```cpp
#include "device/b2b_gemm.h"
#include "b2b_interleaved_gemm_run.h"
#include "test_run.h"
```

**EN**: These headers pull in the building blocks required by this file. Example-local device wrapper that exposes a custom fused kernel as a callable object. Host-side runner that allocates tensors, launches kernels, and checks outputs. Host-side runner that allocates tensors, launches kernels, and checks outputs.  
**CN**: 这些头文件引入了当前文件所需的构建模块。示例本地设备封装，把自定义融合内核暴露为可调用对象。主机侧运行器，负责分配张量、启动内核并检查输出。主机侧运行器，负责分配张量、启动内核并检查输出。

### Lines 47-47 / 第47-47行

```cpp
////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 49-49 / 第49-49行

```cpp
cutlass::gemm::GemmCoord gemm_s8_sm80_problem_size_0(128*640, 64, 576);
```

**EN**: This block defines concrete problem sizes. For GEMM-like code, `GemmCoord(M, N, K)` specifies the row, column, and reduction dimensions that the kernel will process.  
**CN**: 这段代码定义了具体的问题规模。对于 GEMM 类代码，`GemmCoord(M, N, K)` 分别表示内核要处理的行、列与归约维度。

### Lines 50-50 / 第50-50行

```cpp
cutlass::gemm::GemmCoord gemm_s8_sm80_problem_size_1(128*640, 256, 64);
```

**EN**: This block defines concrete problem sizes. For GEMM-like code, `GemmCoord(M, N, K)` specifies the row, column, and reduction dimensions that the kernel will process.  
**CN**: 这段代码定义了具体的问题规模。对于 GEMM 类代码，`GemmCoord(M, N, K)` 分别表示内核要处理的行、列与归约维度。

### Lines 52-52 / 第52-52行

```cpp
bool run_nonfused_gemm_s8_sm80() {
```

**EN**: This line starts a function or callable scope. The body that follows typically configures a kernel, launches it, or checks its result.  
**CN**: 这一行开始定义一个函数或可调用作用域。随后的函数体通常会配置内核、启动执行或检查结果。

### Lines 54-56 / 第54-56行

```cpp
  using ElementOutput = int8_t;
  using ElementAccumulator = int32_t;
  using ElementCompute = float;
```

**EN**: These aliases pin down the scalar types used for inputs, accumulators, and outputs. Choosing them explicitly matters because CUTLASS can compute in a different precision from storage.  
**CN**: 这些别名固定了输入、累加器和输出使用的标量类型。显式指定它们很重要，因为 CUTLASS 可以让计算精度与存储精度不同。

### Lines 58-61 / 第58-61行

```cpp
  ElementCompute alpha0 = ElementCompute(1);
  ElementCompute beta0 = ElementCompute(1); //beta=1 for bias
  ElementCompute alpha1 = ElementCompute(1);
  ElementCompute beta1 = ElementCompute(1); //beta=1 for bias
```

**EN**: This block initializes runtime scalars such as `alpha` and `beta`. They control how the epilogue scales accumulated results and mixes in source or bias terms.  
**CN**: 这段代码初始化 `alpha`、`beta` 等运行时标量。它们控制 epilogue 如何缩放累加结果，并混入源项或偏置项。

### Lines 63-67 / 第63-67行

```cpp
  using ThreadblockShape0 = cutlass::gemm::GemmShape<128, 64, 64>;
  using WarpShape0 = cutlass::gemm::GemmShape<64, 64, 64>;
  using ThreadblockShape1 = cutlass::gemm::GemmShape<128, 128, 64>;
  using WarpShape1 = cutlass::gemm::GemmShape<64, 64, 64>;
  using InstructionShape = cutlass::gemm::GemmShape<16, 8, 32>;
```

**EN**: These aliases describe CUTLASS's tiling hierarchy. The code separately chooses CTA, warp, and instruction shapes so the kernel matches the hardware execution pattern it targets. `GemmShape<M, N, K>` encodes tile extents along the GEMM M/N/K axes. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps. `InstructionShape` mirrors the hardware MMA instruction tile used by Tensor Cores.  
**CN**: 这些别名描述了 CUTLASS 的分块层次。代码分别选择 CTA、warp 与指令级分块，以匹配目标硬件的执行模式。 `GemmShape<M, N, K>` 用于描述 GEMM 在 M/N/K 三个维度上的分块大小。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。 `InstructionShape` 对应 Tensor Core 使用的硬件 MMA 指令级分块。

### Lines 69-95 / 第69-95行

```cpp
  using Gemm0 = cutlass::gemm::device::Gemm<
    int8_t,
    cutlass::layout::ColumnMajorInterleaved<32>,
    int8_t,
    cutlass::layout::RowMajorInterleaved<32>,
    ElementOutput,
    cutlass::layout::ColumnMajorInterleaved<32>,
    ElementAccumulator,
    cutlass::arch::OpClassTensorOp,
    cutlass::arch::Sm80,
    ThreadblockShape0,
    WarpShape0,
    InstructionShape,
    cutlass::epilogue::thread::LinearCombinationRelu<
      ElementOutput,
      64 / cutlass::sizeof_bits<ElementOutput>::value,
      ElementAccumulator,
      ElementCompute,
      cutlass::epilogue::thread::ScaleType::NoBetaScaling
    >,
    cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>,
    3,
    16,
    16,
    false,
    cutlass::arch::OpMultiplyAddSaturate
  >;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps. `InstructionShape` mirrors the hardware MMA instruction tile used by Tensor Cores. `arch::Sm80` targets Ampere-generation GPUs. `OpClassTensorOp` selects Tensor Core math instead of plain SIMT arithmetic.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。 `InstructionShape` 对应 Tensor Core 使用的硬件 MMA 指令级分块。 `arch::Sm80` 表示面向 Ampere 架构 GPU。 `OpClassTensorOp` 选择 Tensor Core 计算，而不是普通的 SIMT 算术。

### Lines 96-122 / 第96-122行

```cpp
  using Gemm1 = cutlass::gemm::device::Gemm<
    int8_t,
    cutlass::layout::ColumnMajorInterleaved<32>,
    int8_t,
    cutlass::layout::RowMajorInterleaved<32>,
    ElementOutput,
    cutlass::layout::ColumnMajorInterleaved<32>,
    ElementAccumulator,
    cutlass::arch::OpClassTensorOp,
    cutlass::arch::Sm80,
    ThreadblockShape1,
    WarpShape1,
    InstructionShape,
    cutlass::epilogue::thread::LinearCombinationRelu<
      ElementOutput,
      64 / cutlass::sizeof_bits<ElementOutput>::value,
      ElementAccumulator,
      ElementCompute,
      cutlass::epilogue::thread::ScaleType::NoBetaScaling
    >,
    cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>,
    3,
    16,
    16,
    false,
    cutlass::arch::OpMultiplyAddSaturate
  >;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps. `InstructionShape` mirrors the hardware MMA instruction tile used by Tensor Cores. `arch::Sm80` targets Ampere-generation GPUs. `OpClassTensorOp` selects Tensor Core math instead of plain SIMT arithmetic.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。 `InstructionShape` 对应 Tensor Core 使用的硬件 MMA 指令级分块。 `arch::Sm80` 表示面向 Ampere 架构 GPU。 `OpClassTensorOp` 选择 Tensor Core 计算，而不是普通的 SIMT 算术。

### Lines 124-124 / 第124-124行

```cpp
  B2bInterleavedNonFusedGemmRun<Gemm0, Gemm1, 32> nonFusedGemm;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 126-126 / 第126-126行

```cpp
  std::cout << "Running Non-fused back-to-back INT8 NT interleaved GEMMs...\n";
```

**EN**: This block prints human-readable status so it is clear which configuration is running and whether validation passed.  
**CN**: 这段代码打印可读的状态信息，让用户清楚当前运行的是哪种配置，以及验证是否通过。

### Lines 127-127 / 第127-127行

```cpp
  bool pass = nonFusedGemm.run(gemm_s8_sm80_problem_size_0, gemm_s8_sm80_problem_size_1, alpha0, beta0, alpha1, beta1);
```

**EN**: This block exercises the configured CUTLASS object at runtime. Typical calls include initialization, capability checks, workspace queries, kernel launches, or reference validation.  
**CN**: 这段代码在运行时驱动已配置好的 CUTLASS 对象。常见调用包括初始化、能力检查、工作区查询、内核启动以及参考结果验证。

### Lines 128-129 / 第128-129行

```cpp
  if(pass)
    std::cout << "Pass\n";
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 130-131 / 第130-131行

```cpp
  else
    std::cout << "Fail\n";
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 133-133 / 第133-133行

```cpp
  return pass;
```

**EN**: This return statement propagates the current result back to the caller so the surrounding harness can continue its control flow.  
**CN**: 这个返回语句会把当前结果传回调用者，使外围测试框架能够继续执行后续控制流程。

### Lines 134-134 / 第134-134行

```cpp
}
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 136-136 / 第136-136行

```cpp
bool run_fused_gemm_s8_sm80_shmem() {
```

**EN**: This line starts a function or callable scope. The body that follows typically configures a kernel, launches it, or checks its result.  
**CN**: 这一行开始定义一个函数或可调用作用域。随后的函数体通常会配置内核、启动执行或检查结果。

### Lines 138-140 / 第138-140行

```cpp
  using ElementOutput = int8_t;
  using ElementAccumulator = int32_t;
  using ElementCompute = float;
```

**EN**: These aliases pin down the scalar types used for inputs, accumulators, and outputs. Choosing them explicitly matters because CUTLASS can compute in a different precision from storage.  
**CN**: 这些别名固定了输入、累加器和输出使用的标量类型。显式指定它们很重要，因为 CUTLASS 可以让计算精度与存储精度不同。

### Lines 142-142 / 第142-142行

```cpp
  ElementCompute alpha0 = ElementCompute(1);
```

**EN**: This block initializes runtime scalars such as `alpha` and `beta`. They control how the epilogue scales accumulated results and mixes in source or bias terms.  
**CN**: 这段代码初始化 `alpha`、`beta` 等运行时标量。它们控制 epilogue 如何缩放累加结果，并混入源项或偏置项。

### Lines 143-143 / 第143-143行

```cpp
  //Fused kernel has built-in bias, setting beta=0
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 144-146 / 第144-146行

```cpp
  ElementCompute beta0 = ElementCompute(0);
  ElementCompute alpha1 = ElementCompute(1);
  ElementCompute beta1 = ElementCompute(1); //beta=1 for bias
```

**EN**: This block initializes runtime scalars such as `alpha` and `beta`. They control how the epilogue scales accumulated results and mixes in source or bias terms.  
**CN**: 这段代码初始化 `alpha`、`beta` 等运行时标量。它们控制 epilogue 如何缩放累加结果，并混入源项或偏置项。

### Lines 148-152 / 第148-152行

```cpp
  using ThreadblockShape0 = cutlass::gemm::GemmShape<64, 64, 64>;
  using WarpShape0 = cutlass::gemm::GemmShape<32, 32, 64>;
  using ThreadblockShape1 = cutlass::gemm::GemmShape<64, 256, 64>;
  using WarpShape1 = cutlass::gemm::GemmShape<64, 64, 64>;
  using InstructionShape = cutlass::gemm::GemmShape<16, 8, 32>;
```

**EN**: These aliases describe CUTLASS's tiling hierarchy. The code separately chooses CTA, warp, and instruction shapes so the kernel matches the hardware execution pattern it targets. `GemmShape<M, N, K>` encodes tile extents along the GEMM M/N/K axes. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps. `InstructionShape` mirrors the hardware MMA instruction tile used by Tensor Cores.  
**CN**: 这些别名描述了 CUTLASS 的分块层次。代码分别选择 CTA、warp 与指令级分块，以匹配目标硬件的执行模式。 `GemmShape<M, N, K>` 用于描述 GEMM 在 M/N/K 三个维度上的分块大小。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。 `InstructionShape` 对应 Tensor Core 使用的硬件 MMA 指令级分块。

### Lines 154-161 / 第154-161行

```cpp
  using EpilogueOutputOp0 =
    cutlass::epilogue::thread::LinearCombinationRelu<
      ElementOutput,
      8 * InstructionShape::kN / 32,
      ElementAccumulator,
      ElementCompute,
      cutlass::epilogue::thread::ScaleType::OnlyAlphaScaling
    >;
```

**EN**: This block defines epilogue functors. In CUTLASS, the epilogue converts accumulators into the output type and can fuse extra work such as scaling, bias handling, clamping, or activation. `InstructionShape` mirrors the hardware MMA instruction tile used by Tensor Cores. `LinearCombinationRelu` fuses scaling, bias-like accumulation, and ReLU activation in the epilogue. `OnlyAlphaScaling` means the epilogue only multiplies by alpha and does not read a beta-scaled source tensor.  
**CN**: 这段代码定义了 epilogue functor。在 CUTLASS 中，epilogue 负责把累加器转换为输出类型，并可融合缩放、偏置处理、截断或激活等附加工作。 `InstructionShape` 对应 Tensor Core 使用的硬件 MMA 指令级分块。 `LinearCombinationRelu` 在 epilogue 中融合了缩放、偏置式累加与 ReLU 激活。 `OnlyAlphaScaling` 表示 epilogue 只做 alpha 缩放，不读取 beta 缩放的源张量。

### Lines 163-170 / 第163-170行

```cpp
  using EpilogueOutputOp1 =
    cutlass::epilogue::thread::LinearCombinationRelu<
      ElementOutput,
      64 / cutlass::sizeof_bits<ElementOutput>::value,
      ElementAccumulator,
      ElementCompute,
      cutlass::epilogue::thread::ScaleType::NoBetaScaling
    >;
```

**EN**: This block defines epilogue functors. In CUTLASS, the epilogue converts accumulators into the output type and can fuse extra work such as scaling, bias handling, clamping, or activation. `LinearCombinationRelu` fuses scaling, bias-like accumulation, and ReLU activation in the epilogue. `NoBetaScaling` is typically used when the source contribution behaves like bias rather than a conventional beta-scaled matrix.  
**CN**: 这段代码定义了 epilogue functor。在 CUTLASS 中，epilogue 负责把累加器转换为输出类型，并可融合缩放、偏置处理、截断或激活等附加工作。 `LinearCombinationRelu` 在 epilogue 中融合了缩放、偏置式累加与 ReLU 激活。 `NoBetaScaling` 常用于源项更像偏置而不是常规 beta 缩放矩阵的场景。

### Lines 172-172 / 第172-172行

```cpp
  const bool SmemAccumulator = true;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 174-197 / 第174-197行

```cpp
  using B2bGemm = cutlass::gemm::device::B2bGemm<
    int8_t,
    cutlass::layout::ColumnMajorInterleaved<32>,
    int8_t,
    cutlass::layout::RowMajorInterleaved<32>,
    ElementOutput,
    cutlass::layout::ColumnMajorInterleaved<32>,
    ElementAccumulator,
    cutlass::arch::OpClassTensorOp,
    cutlass::arch::Sm80,
    ThreadblockShape0,
    ThreadblockShape1,
    WarpShape0,
    WarpShape1,
    InstructionShape,
    EpilogueOutputOp0,
    EpilogueOutputOp1,
    cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>,
    3,
    SmemAccumulator,
    16,
    16,
    cutlass::arch::OpMultiplyAddSaturate
  >;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. `WarpShape` controls how the CTA tile is partitioned across warps. `InstructionShape` mirrors the hardware MMA instruction tile used by Tensor Cores. `arch::Sm80` targets Ampere-generation GPUs. `OpClassTensorOp` selects Tensor Core math instead of plain SIMT arithmetic.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 `WarpShape` 控制 CTA 分块如何在多个 warp 之间划分。 `InstructionShape` 对应 Tensor Core 使用的硬件 MMA 指令级分块。 `arch::Sm80` 表示面向 Ampere 架构 GPU。 `OpClassTensorOp` 选择 Tensor Core 计算，而不是普通的 SIMT 算术。

### Lines 199-199 / 第199-199行

```cpp
  B2bInterleavedFusedGemmRun<B2bGemm, 32> fusedGemm;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 201-201 / 第201-201行

```cpp
  std::cout << "Running Fused back-to-back INT8 NT interleaved GEMMs with shared memory staging...\n";
```

**EN**: This block prints human-readable status so it is clear which configuration is running and whether validation passed. Shared-memory accumulator residency trades more on-chip storage for the ability to decouple the two fused stages.  
**CN**: 这段代码打印可读的状态信息，让用户清楚当前运行的是哪种配置，以及验证是否通过。 共享内存累加器驻留通过使用更多片上存储，换取两个融合阶段之间更强的解耦能力。

### Lines 202-202 / 第202-202行

```cpp
  bool passed = fusedGemm.run(gemm_s8_sm80_problem_size_0, gemm_s8_sm80_problem_size_1, alpha0, beta0, alpha1, beta1);
```

**EN**: This block exercises the configured CUTLASS object at runtime. Typical calls include initialization, capability checks, workspace queries, kernel launches, or reference validation.  
**CN**: 这段代码在运行时驱动已配置好的 CUTLASS 对象。常见调用包括初始化、能力检查、工作区查询、内核启动以及参考结果验证。

### Lines 203-204 / 第203-204行

```cpp
  if(passed)
    std::cout << "Pass\n";
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 205-206 / 第205-206行

```cpp
  else
    std::cout << "Fail\n";
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 208-208 / 第208-208行

```cpp
  return passed;
```

**EN**: This return statement propagates the current result back to the caller so the surrounding harness can continue its control flow.  
**CN**: 这个返回语句会把当前结果传回调用者，使外围测试框架能够继续执行后续控制流程。

### Lines 209-209 / 第209-209行

```cpp
}
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 212-212 / 第212-212行

```cpp
int main() {
```

**EN**: This line starts a function or callable scope. The body that follows typically configures a kernel, launches it, or checks its result.  
**CN**: 这一行开始定义一个函数或可调用作用域。随后的函数体通常会配置内核、启动执行或检查结果。

### Lines 214-217 / 第214-217行

```cpp
  std::vector<bool (*)()>funcs = {
    &run_nonfused_gemm_s8_sm80,
    &run_fused_gemm_s8_sm80_shmem
  };
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 219-219 / 第219-219行

```cpp
  return testRun(80, funcs, "gemm int8 shmem staging");
```

**EN**: This return statement propagates the current result back to the caller so the surrounding harness can continue its control flow.  
**CN**: 这个返回语句会把当前结果传回调用者，使外围测试框架能够继续执行后续控制流程。

### Lines 222-222 / 第222-222行

```cpp
}
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 225-225 / 第225-225行

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
- Tensor Core execution / Tensor Core 执行
- Shared-memory accumulator staging / 共享内存累加器暂存

## Dependencies / 依赖项

- `<iostream>` — Console stream utilities for logging progress and results. / 用于记录进度和结果的控制台流工具。
- `"cutlass/cutlass.h"` — Core CUTLASS definitions, architecture tags, and status types. / CUTLASS 的核心定义、架构标签与状态类型。
- `"cutlass/gemm/device/gemm.h"` — Device-level GEMM wrapper that hides kernel selection and launch details. / 设备级 GEMM 封装，隐藏内核选择与启动细节。
- `"cutlass/util/host_tensor.h"` — Host/device tensor wrapper used to allocate storage and transfer data. / 主机/设备张量封装，用于分配存储并传输数据。
- `"cutlass/util/tensor_view_io.h"` — Tensor printing helpers for debugging layouts and values. / 用于调试布局和值的张量打印辅助工具。
- `"cutlass/util/reference/host/tensor_fill.h"` — Host-side reference implementation used for correctness checking. / 用于正确性检查的主机端参考实现。
- `"cutlass/util/reference/host/tensor_copy.h"` — Host-side reference implementation used for correctness checking. / 用于正确性检查的主机端参考实现。
- `"cutlass/util/reference/host/tensor_compare.h"` — Host-side reference implementation used for correctness checking. / 用于正确性检查的主机端参考实现。
- `"cutlass/util/reference/host/gemm.h"` — Host-side reference implementation used for correctness checking. / 用于正确性检查的主机端参考实现。
- `"device/b2b_gemm.h"` — Example-local device wrapper that exposes a custom fused kernel as a callable object. / 示例本地设备封装，把自定义融合内核暴露为可调用对象。
- `"b2b_interleaved_gemm_run.h"` — Host-side runner that allocates tensors, launches kernels, and checks outputs. / 主机侧运行器，负责分配张量、启动内核并检查输出。
- `"test_run.h"` — Host-side runner that allocates tensors, launches kernels, and checks outputs. / 主机侧运行器，负责分配张量、启动内核并检查输出。
