# ampere_sparse_tensorop_gemm.cu — Code Analysis / 代码分析

**Source / 源文件**: `examples/15_ampere_sparse_tensorop_gemm/ampere_sparse_tensorop_gemm.cu`  
**Purpose / 用途**: Shows Ampere structured-sparse Tensor Core GEMM with the standard device-level interface. / 展示通过标准设备接口调用的 Ampere 结构化稀疏 Tensor Core GEMM。

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
/**
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 33-34 / 第33-34行

```cpp
Please check example 07, 08 and 17 for the basics of dense tensor op gemm kernels.  NVIDIA Ampere
architecture also supports structured sparse tensor op for tf32, fp16, int8 and int4.
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow. TF32 keeps FP32 range while using Tensor Core-friendly mantissa precision for higher throughput. Ampere sparse Tensor Core kernels exploit structured 2:4 sparsity to reduce arithmetic and data movement.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。 TF32 保留 FP32 的指数范围，同时用更适合 Tensor Core 的尾数精度来提升吞吐。 Ampere 稀疏 Tensor Core 内核利用结构化 2:4 稀疏性来降低算术量与数据搬运。

### Lines 36-38 / 第36-38行

```cpp
Sparse GEMM kernels needs to takes an additional E matrix which stores the meta data.  The format of
meta data is different for every data types.   CUTLASS templates can automatically infer it based on
input A and B.  Check code below.
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow. Ampere sparse Tensor Core kernels exploit structured 2:4 sparsity to reduce arithmetic and data movement.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。 Ampere 稀疏 Tensor Core 内核利用结构化 2:4 稀疏性来降低算术量与数据搬运。

### Lines 40-42 / 第40-42行

```cpp
Moreover, matrix E needs to be preprocessed so that it can use ldmatrix to load into the registers
efficiently.
*/
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 44-44 / 第44-44行

```cpp
#include <iostream>
```

**EN**: These headers pull in the building blocks required by this file. Console stream utilities for logging progress and results.  
**CN**: 这些头文件引入了当前文件所需的构建模块。用于记录进度和结果的控制台流工具。

### Lines 46-56 / 第46-56行

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/gemm/device/gemm_sparse.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/reference/host/gemm.h"
#include "cutlass/util/host_reorder.h"
#include "cutlass/util/host_uncompress.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/tensor_view_io.h"
#include "helper.h"
```

**EN**: These headers pull in the building blocks required by this file. Core CUTLASS definitions, architecture tags, and status types. Device-level GEMM wrapper that hides kernel selection and launch details. Host/device tensor wrapper used to allocate storage and transfer data. Host-side reference implementation used for correctness checking. Ampere sparse Tensor Core kernels exploit structured 2:4 sparsity to reduce arithmetic and data movement.  
**CN**: 这些头文件引入了当前文件所需的构建模块。CUTLASS 的核心定义、架构标签与状态类型。设备级 GEMM 封装，隐藏内核选择与启动细节。主机/设备张量封装，用于分配存储并传输数据。用于正确性检查的主机端参考实现。 Ampere 稀疏 Tensor Core 内核利用结构化 2:4 稀疏性来降低算术量与数据搬运。

### Lines 58-59 / 第58-59行

```cpp
// The code section below describes datatype for input, output matrices and computation between
// elements in input matrices.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 60-64 / 第60-64行

```cpp
using ElementAccumulator = int32_t;                 // <- data type of accumulator
using ElementComputeEpilogue = ElementAccumulator;  // <- data type of epilogue operations
using ElementInputA = cutlass::int4b_t;             // <- data type of elements in input matrix A
using ElementInputB = cutlass::int4b_t;             // <- data type of elements in input matrix B
using ElementOutput = int32_t;                      // <- data type of elements in output matrix D
```

**EN**: These aliases pin down the scalar types used for inputs, accumulators, and outputs. Choosing them explicitly matters because CUTLASS can compute in a different precision from storage.  
**CN**: 这些别名固定了输入、累加器和输出使用的标量类型。显式指定它们很重要，因为 CUTLASS 可以让计算精度与存储精度不同。

### Lines 66-67 / 第66-67行

```cpp
// The code section below describes matrix layout of input and output matrices. Row Major for
// Matrix A, Column Major for Matrix B and Row Major for Matrix C
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 68-70 / 第68-70行

```cpp
using LayoutInputA = cutlass::layout::RowMajor;
using LayoutInputB = cutlass::layout::ColumnMajor;
using LayoutOutput = cutlass::layout::RowMajor;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 72-72 / 第72-72行

```cpp
// This code section describes whether you want to use tensor cores or regular SIMT cores on GPU SM
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 73-73 / 第73-73行

```cpp
using MMAOp = cutlass::arch::OpClassTensorOp;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow. `OpClassTensorOp` selects Tensor Core math instead of plain SIMT arithmetic.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。 `OpClassTensorOp` 选择 Tensor Core 计算，而不是普通的 SIMT 算术。

### Lines 75-75 / 第75-75行

```cpp
// This code section describes CUDA SM architecture number
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 76-76 / 第76-76行

```cpp
using SmArch = cutlass::arch::Sm80;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow. `arch::Sm80` targets Ampere-generation GPUs.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。 `arch::Sm80` 表示面向 Ampere 架构 GPU。

### Lines 78-78 / 第78-78行

```cpp
// This code section describes the tile size a thread block will compute
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 79-84 / 第79-84行

```cpp
using ShapeMMAThreadBlock =
    cutlass::gemm::GemmShape<128, 128, 256>;  // <- threadblock tile M = 128, N = 128, K = 256
// This code section describes tile size a warp will compute
using ShapeMMAWarp = cutlass::gemm::GemmShape<64, 64, 256>;  // <- warp tile M = 64, N = 64, K = 256
// This code section describes the size of MMA op
using ShapeMMAOp = cutlass::gemm::GemmShape<16, 8, 128>;  // <- MMA Op tile M = 16, N = 8, K = 128
```

**EN**: These aliases describe CUTLASS's tiling hierarchy. The code separately chooses CTA, warp, and instruction shapes so the kernel matches the hardware execution pattern it targets. `GemmShape<M, N, K>` encodes tile extents along the GEMM M/N/K axes.  
**CN**: 这些别名描述了 CUTLASS 的分块层次。代码分别选择 CTA、warp 与指令级分块，以匹配目标硬件的执行模式。 `GemmShape<M, N, K>` 用于描述 GEMM 在 M/N/K 三个维度上的分块大小。

### Lines 86-86 / 第86-86行

```cpp
// This code section describes how threadblocks are scheduled on GPU
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 87-87 / 第87-87行

```cpp
using SwizzleThreadBlock = cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 89-89 / 第89-89行

```cpp
// This code section describes the epilogue part of the kernel
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 90-97 / 第90-97行

```cpp
using EpilogueOp = cutlass::epilogue::thread::LinearCombination<
    ElementOutput,                                     // <- data type of output matrix
    128 / cutlass::sizeof_bits<ElementOutput>::value,  // <- the number of elements per vectorized
                                                       // memory access. For a byte, it's 16
                                                       // elements. This becomes the vector width of
                                                       // math instructions in the epilogue too
    ElementAccumulator,                                // <- data type of accumulator
    ElementComputeEpilogue>;  // <- data type for alpha/beta in linear combination function
```

**EN**: This block defines epilogue functors. In CUTLASS, the epilogue converts accumulators into the output type and can fuse extra work such as scaling, bias handling, clamping, or activation. `LinearCombination` is CUTLASS's standard epilogue functor for scaling accumulators and combining them with existing outputs.  
**CN**: 这段代码定义了 epilogue functor。在 CUTLASS 中，epilogue 负责把累加器转换为输出类型，并可融合缩放、偏置处理、截断或激活等附加工作。 `LinearCombination` 是 CUTLASS 标准 epilogue functor，用于缩放累加器并与已有输出组合。

### Lines 99-99 / 第99-99行

```cpp
// Number of pipelines you want to use
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 100-100 / 第100-100行

```cpp
constexpr int NumStages = 3;
```

**EN**: This block defines compile-time constants or static values that parameterize the kernel or the benchmark.  
**CN**: 这段代码定义编译期常量或静态值，用于参数化内核或基准测试。

### Lines 102-116 / 第102-116行

```cpp
using Gemm = cutlass::gemm::device::SparseGemm<ElementInputA,
                                               LayoutInputA,
                                               ElementInputB,
                                               LayoutInputB,
                                               ElementOutput,
                                               LayoutOutput,
                                               ElementAccumulator,
                                               MMAOp,
                                               SmArch,
                                               ShapeMMAThreadBlock,
                                               ShapeMMAWarp,
                                               ShapeMMAOp,
                                               EpilogueOp,
                                               SwizzleThreadBlock,
                                               NumStages>;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth. Ampere sparse Tensor Core kernels exploit structured 2:4 sparsity to reduce arithmetic and data movement.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。 Ampere 稀疏 Tensor Core 内核利用结构化 2:4 稀疏性来降低算术量与数据搬运。

### Lines 118-118 / 第118-118行

```cpp
// Data type and layout of meta data matrix E can be inferred from template Gemm.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 119-119 / 第119-119行

```cpp
using ElementInputE = typename Gemm::ElementE;
```

**EN**: These aliases pin down the scalar types used for inputs, accumulators, and outputs. Choosing them explicitly matters because CUTLASS can compute in a different precision from storage.  
**CN**: 这些别名固定了输入、累加器和输出使用的标量类型。显式指定它们很重要，因为 CUTLASS 可以让计算精度与存储精度不同。

### Lines 120-121 / 第120-121行

```cpp
using LayoutInputE = cutlass::layout::RowMajor;
using ReorderedLayoutInputE = typename Gemm::LayoutE;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 123-124 / 第123-124行

```cpp
// Blow property is defined in include/cutlass/arch/sp_mma_sm80.h
// 50% Sparsity on Ampere
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 125-125 / 第125-125行

```cpp
constexpr int kSparse = Gemm::kSparse;
```

**EN**: This block defines compile-time constants or static values that parameterize the kernel or the benchmark. Ampere sparse Tensor Core kernels exploit structured 2:4 sparsity to reduce arithmetic and data movement.  
**CN**: 这段代码定义编译期常量或静态值，用于参数化内核或基准测试。 Ampere 稀疏 Tensor Core 内核利用结构化 2:4 稀疏性来降低算术量与数据搬运。

### Lines 126-126 / 第126-126行

```cpp
// How many elements of A are covered per ElementE
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 127-127 / 第127-127行

```cpp
constexpr int kElementsPerElementE = Gemm::kElementsPerElementE;
```

**EN**: This block defines compile-time constants or static values that parameterize the kernel or the benchmark.  
**CN**: 这段代码定义编译期常量或静态值，用于参数化内核或基准测试。

### Lines 128-128 / 第128-128行

```cpp
// The size of individual meta data 
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 129-129 / 第129-129行

```cpp
constexpr int kMetaSizeInBits = Gemm::kMetaSizeInBits;
```

**EN**: This block defines compile-time constants or static values that parameterize the kernel or the benchmark.  
**CN**: 这段代码定义编译期常量或静态值，用于参数化内核或基准测试。

### Lines 131-131 / 第131-131行

```cpp
int run() {
```

**EN**: This line starts a function or callable scope. The body that follows typically configures a kernel, launches it, or checks its result.  
**CN**: 这一行开始定义一个函数或可调用作用域。随后的函数体通常会配置内核、启动执行或检查结果。

### Lines 133-135 / 第133-135行

```cpp
  const int length_m = 512;
  const int length_n = 512;
  const int length_k = 1024;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 137-137 / 第137-137行

```cpp
  // Create a tuple of problem size for matrix multiplication
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 138-138 / 第138-138行

```cpp
  cutlass::gemm::GemmCoord problem_size(length_m, length_n, length_k);
```

**EN**: This block defines concrete problem sizes. For GEMM-like code, `GemmCoord(M, N, K)` specifies the row, column, and reduction dimensions that the kernel will process.  
**CN**: 这段代码定义了具体的问题规模。对于 GEMM 类代码，`GemmCoord(M, N, K)` 分别表示内核要处理的行、列与归约维度。

### Lines 140-140 / 第140-140行

```cpp
  // Initialize tensors using CUTLASS helper functions
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 141-144 / 第141-144行

```cpp
  cutlass::HostTensor<ElementInputA, LayoutInputA> tensor_a(
      cutlass::make_Coord(problem_size.m(), problem_size.k() / kSparse));  // <- Create matrix A with dimensions M x (K / 2)
  cutlass::HostTensor<ElementInputA, LayoutInputA> tensor_a_uncompressed(
      problem_size.mk());  // <- Create uncompressed matrix A with dimensions M x K for reference computing
```

**EN**: This block initializes runtime scalars such as `alpha` and `beta`. They control how the epilogue scales accumulated results and mixes in source or bias terms. Ampere sparse Tensor Core kernels exploit structured 2:4 sparsity to reduce arithmetic and data movement.  
**CN**: 这段代码初始化 `alpha`、`beta` 等运行时标量。它们控制 epilogue 如何缩放累加结果，并混入源项或偏置项。 Ampere 稀疏 Tensor Core 内核利用结构化 2:4 稀疏性来降低算术量与数据搬运。

### Lines 146-155 / 第146-155行

```cpp
  cutlass::HostTensor<ElementInputB, LayoutInputB> tensor_b(
      problem_size.kn());  // <- Create matrix B with dimensions K x N
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_c(
      problem_size.mn());  // <- Create matrix C with dimensions M x N
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_d(
      problem_size.mn());  // <- Create matrix D with dimensions M x N used to store output from
                           // CUTLASS kernel
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_ref_d(
      problem_size.mn());  // <- Create matrix D with dimensions M x N used to store output from
                           // reference kernel
```

**EN**: This block initializes runtime scalars such as `alpha` and `beta`. They control how the epilogue scales accumulated results and mixes in source or bias terms.  
**CN**: 这段代码初始化 `alpha`、`beta` 等运行时标量。它们控制 epilogue 如何缩放累加结果，并混入源项或偏置项。

### Lines 157-157 / 第157-157行

```cpp
  // Create matrix E with dimensions M x (K / 2 / kElementsPerElementE). This one is used by reference computing.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 158-159 / 第158-159行

```cpp
  cutlass::HostTensor<ElementInputE, LayoutInputE> tensor_e(
      cutlass::make_Coord(problem_size.m(), problem_size.k() / kSparse / kElementsPerElementE));
```

**EN**: This block initializes runtime scalars such as `alpha` and `beta`. They control how the epilogue scales accumulated results and mixes in source or bias terms. Ampere sparse Tensor Core kernels exploit structured 2:4 sparsity to reduce arithmetic and data movement.  
**CN**: 这段代码初始化 `alpha`、`beta` 等运行时标量。它们控制 epilogue 如何缩放累加结果，并混入源项或偏置项。 Ampere 稀疏 Tensor Core 内核利用结构化 2:4 稀疏性来降低算术量与数据搬运。

### Lines 160-160 / 第160-160行

```cpp
  // Same size as the above.  The above one needs to be reordered and stored in this one.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 161-162 / 第161-162行

```cpp
  cutlass::HostTensor<ElementInputE, ReorderedLayoutInputE> tensor_e_reordered(
      cutlass::make_Coord(problem_size.m(), problem_size.k() / kSparse / kElementsPerElementE));
```

**EN**: This block initializes runtime scalars such as `alpha` and `beta`. They control how the epilogue scales accumulated results and mixes in source or bias terms. Ampere sparse Tensor Core kernels exploit structured 2:4 sparsity to reduce arithmetic and data movement.  
**CN**: 这段代码初始化 `alpha`、`beta` 等运行时标量。它们控制 epilogue 如何缩放累加结果，并混入源项或偏置项。 Ampere 稀疏 Tensor Core 内核利用结构化 2:4 稀疏性来降低算术量与数据搬运。

### Lines 164-164 / 第164-164行

```cpp
  // Fill input and output matrices on host using CUTLASS helper functions
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 165-190 / 第165-190行

```cpp
  cutlass::reference::host::TensorFillRandomUniform(
      tensor_a.host_view(),
      1,
      ElementInputA(2),
      ElementInputA(-2),
      0);  // <- Fill matrix A on host with uniform-distribution random data
  cutlass::reference::host::TensorFillRandomUniform(
      tensor_b.host_view(),
      1,
      ElementInputB(2),
      ElementInputB(-2),
      0);  // <- Fill matrix B on host with uniform-distribution random data
  cutlass::reference::host::TensorFillRandomUniform(
      tensor_c.host_view(),
      1,
      ElementOutput(2),
      ElementOutput(-2),
      0);  // <- Fill matrix C on host with uniform-distribution random data
  cutlass::reference::host::TensorFillRandomSparseMeta(                                           
      tensor_e.host_view(),
      1,
      kMetaSizeInBits);   // <- Fill matrix E on host with uniform-distribution random meta data
  cutlass::reference::host::TensorFill(
      tensor_d.host_view());  // <- fill matrix D on host with zeros
  cutlass::reference::host::TensorFill(
      tensor_ref_d.host_view());  // <- fill matrix D for reference on host with zeros
```

**EN**: This block initializes runtime scalars such as `alpha` and `beta`. They control how the epilogue scales accumulated results and mixes in source or bias terms. Ampere sparse Tensor Core kernels exploit structured 2:4 sparsity to reduce arithmetic and data movement.  
**CN**: 这段代码初始化 `alpha`、`beta` 等运行时标量。它们控制 epilogue 如何缩放累加结果，并混入源项或偏置项。 Ampere 稀疏 Tensor Core 内核利用结构化 2:4 稀疏性来降低算术量与数据搬运。

### Lines 192-193 / 第192-193行

```cpp
  // Reorder the meta data matrix so that we can use ldmatrix to load them to tensor core
  // instructions.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 194-196 / 第194-196行

```cpp
  cutlass::reorder_meta(tensor_e_reordered.host_ref(), tensor_e.host_ref(),                         
                        {problem_size.m(), problem_size.n(),                                        
                         problem_size.k() / kSparse / kElementsPerElementE});
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow. Ampere sparse Tensor Core kernels exploit structured 2:4 sparsity to reduce arithmetic and data movement.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。 Ampere 稀疏 Tensor Core 内核利用结构化 2:4 稀疏性来降低算术量与数据搬运。

### Lines 198-198 / 第198-198行

```cpp
  // Copy data from host to GPU
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 199-199 / 第199-199行

```cpp
  tensor_a.sync_device();
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 200-200 / 第200-200行

```cpp
  tensor_b.sync_device();
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 201-201 / 第201-201行

```cpp
  tensor_c.sync_device();
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 202-202 / 第202-202行

```cpp
  tensor_d.sync_device();
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 203-203 / 第203-203行

```cpp
  tensor_e_reordered.sync_device();
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 204-204 / 第204-204行

```cpp
  tensor_ref_d.sync_device();
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 206-206 / 第206-206行

```cpp
  // Initialize alpha and beta for dot product computation
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 207-208 / 第207-208行

```cpp
  ElementComputeEpilogue alpha = ElementComputeEpilogue(1);
  ElementComputeEpilogue beta = ElementComputeEpilogue(0);
```

**EN**: This block initializes runtime scalars such as `alpha` and `beta`. They control how the epilogue scales accumulated results and mixes in source or bias terms.  
**CN**: 这段代码初始化 `alpha`、`beta` 等运行时标量。它们控制 epilogue 如何缩放累加结果，并混入源项或偏置项。

### Lines 210-210 / 第210-210行

```cpp
  // Split K dimension into 1 partitions
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 211-211 / 第211-211行

```cpp
  int split_k_slices = 1;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 213-214 / 第213-214行

```cpp
  // Create a tuple of gemm kernel arguments. This is later passed as arguments to launch
  // instantiated CUTLASS kernel
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 215-222 / 第215-222行

```cpp
  typename Gemm::Arguments arguments{problem_size,  // <- problem size of matrix multiplication
                                     tensor_a.device_ref(),  // <- reference to matrix A on device
                                     tensor_b.device_ref(),  // <- reference to matrix B on device
                                     tensor_c.device_ref(),  // <- reference to matrix C on device
                                     tensor_d.device_ref(),  // <- reference to matrix D on device
                                     tensor_e_reordered.device_ref(),  // <- reference to matrix E on device
                                     {alpha, beta},          // <- tuple of alpha and beta
                                     split_k_slices};        // <- k-dimension split factor
```

**EN**: This block initializes runtime scalars such as `alpha` and `beta`. They control how the epilogue scales accumulated results and mixes in source or bias terms.  
**CN**: 这段代码初始化 `alpha`、`beta` 等运行时标量。它们控制 epilogue 如何缩放累加结果，并混入源项或偏置项。

### Lines 224-224 / 第224-224行

```cpp
  // Using the arguments, query for extra workspace required for matrix multiplication computation
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 225-225 / 第225-225行

```cpp
  size_t workspace_size = Gemm::get_workspace_size(arguments);
```

**EN**: This block exercises the configured CUTLASS object at runtime. Typical calls include initialization, capability checks, workspace queries, kernel launches, or reference validation.  
**CN**: 这段代码在运行时驱动已配置好的 CUTLASS 对象。常见调用包括初始化、能力检查、工作区查询、内核启动以及参考结果验证。

### Lines 227-227 / 第227-227行

```cpp
  // Allocate workspace memory
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 228-228 / 第228-228行

```cpp
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 230-230 / 第230-230行

```cpp
  // Instantiate CUTLASS kernel depending on templates
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 231-231 / 第231-231行

```cpp
  Gemm gemm_op;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 233-233 / 第233-233行

```cpp
  // Check the problem size is supported or not 
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 234-234 / 第234-234行

```cpp
  cutlass::Status status = gemm_op.can_implement(arguments);
```

**EN**: This block exercises the configured CUTLASS object at runtime. Typical calls include initialization, capability checks, workspace queries, kernel launches, or reference validation.  
**CN**: 这段代码在运行时驱动已配置好的 CUTLASS 对象。常见调用包括初始化、能力检查、工作区查询、内核启动以及参考结果验证。

### Lines 235-235 / 第235-235行

```cpp
  CUTLASS_CHECK(status);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 237-237 / 第237-237行

```cpp
  // Initialize CUTLASS kernel with arguments and workspace pointer
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 238-238 / 第238-238行

```cpp
  status = gemm_op.initialize(arguments, workspace.get());
```

**EN**: This block exercises the configured CUTLASS object at runtime. Typical calls include initialization, capability checks, workspace queries, kernel launches, or reference validation.  
**CN**: 这段代码在运行时驱动已配置好的 CUTLASS 对象。常见调用包括初始化、能力检查、工作区查询、内核启动以及参考结果验证。

### Lines 239-239 / 第239-239行

```cpp
  CUTLASS_CHECK(status);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 241-241 / 第241-241行

```cpp
  // Launch initialized CUTLASS kernel
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 242-242 / 第242-242行

```cpp
  status = gemm_op();
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 243-243 / 第243-243行

```cpp
  CUTLASS_CHECK(status);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 245-245 / 第245-245行

```cpp
  // uncompress tensor_a based on meta data tensor_e. We need it for reference computing.
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 246-247 / 第246-247行

```cpp
  cutlass::uncompress(tensor_a_uncompressed.host_ref(), tensor_a.host_ref(),
                      tensor_e.host_ref(), problem_size.m(), problem_size.k());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 249-249 / 第249-249行

```cpp
  // Create instantiation for host reference gemm kernel
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 250-259 / 第250-259行

```cpp
  cutlass::reference::host::Gemm<ElementInputA,
                                 LayoutInputA,
                                 ElementInputB,
                                 LayoutInputB,
                                 ElementOutput,
                                 LayoutOutput,
                                 ElementComputeEpilogue,
                                 ElementComputeEpilogue,
                                 typename Gemm::Operator>
      gemm_host;
```

**EN**: This block initializes runtime scalars such as `alpha` and `beta`. They control how the epilogue scales accumulated results and mixes in source or bias terms.  
**CN**: 这段代码初始化 `alpha`、`beta` 等运行时标量。它们控制 epilogue 如何缩放累加结果，并混入源项或偏置项。

### Lines 261-261 / 第261-261行

```cpp
  // Launch host reference gemm kernel
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 262-268 / 第262-268行

```cpp
  gemm_host(problem_size,
            alpha,
            tensor_a_uncompressed.host_ref(),
            tensor_b.host_ref(),
            beta,
            tensor_c.host_ref(),
            tensor_ref_d.host_ref());
```

**EN**: This block initializes runtime scalars such as `alpha` and `beta`. They control how the epilogue scales accumulated results and mixes in source or bias terms.  
**CN**: 这段代码初始化 `alpha`、`beta` 等运行时标量。它们控制 epilogue 如何缩放累加结果，并混入源项或偏置项。

### Lines 270-270 / 第270-270行

```cpp
  // Copy output data from CUTLASS host for comparison
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 271-271 / 第271-271行

```cpp
  tensor_d.sync_host();
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 273-273 / 第273-273行

```cpp
  // Check if output from CUTLASS kernel and reference kernel are equal or not
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 274-276 / 第274-276行

```cpp
  bool passed = cutlass::reference::host::TensorEquals(
    tensor_d.host_view(),
    tensor_ref_d.host_view());
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 278-278 / 第278-278行

```cpp
  std::cout << (passed ? "Passed" : "Failed") << std::endl;
```

**EN**: This block prints human-readable status so it is clear which configuration is running and whether validation passed.  
**CN**: 这段代码打印可读的状态信息，让用户清楚当前运行的是哪种配置，以及验证是否通过。

### Lines 280-280 / 第280-280行

```cpp
  return (passed ? 0  : -1);
```

**EN**: This return statement propagates the current result back to the caller so the surrounding harness can continue its control flow.  
**CN**: 这个返回语句会把当前结果传回调用者，使外围测试框架能够继续执行后续控制流程。

### Lines 281-281 / 第281-281行

```cpp
}
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 283-283 / 第283-283行

```cpp
int main() {
```

**EN**: This line starts a function or callable scope. The body that follows typically configures a kernel, launches it, or checks its result.  
**CN**: 这一行开始定义一个函数或可调用作用域。随后的函数体通常会配置内核、启动执行或检查结果。

### Lines 285-285 / 第285-285行

```cpp
  bool notSupported = false;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 287-290 / 第287-290行

```cpp
  // Ampere Sparse Tensor Core operations exposed with mma.sync and ldmatrix are first available
  // in CUDA 11.1. 
  //
  // CUTLASS must be compiled with CUDA 11.1 Toolkit to run these examples.
```

**EN**: This comment separates sections and documents the purpose of the code that follows. Ampere sparse Tensor Core kernels exploit structured 2:4 sparsity to reduce arithmetic and data movement.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。 Ampere 稀疏 Tensor Core 内核利用结构化 2:4 稀疏性来降低算术量与数据搬运。

### Lines 292-293 / 第292-293行

```cpp
  if (!(__CUDACC_VER_MAJOR__ > 11 || (__CUDACC_VER_MAJOR__ == 11 && __CUDACC_VER_MINOR__ >= 1))) {
    std::cerr << "Ampere Tensor Core operations must be compiled with CUDA 11.1 Toolkit or later." << std::endl;
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 294-294 / 第294-294行

```cpp
    notSupported = true;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 295-295 / 第295-295行

```cpp
  }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 297-297 / 第297-297行

```cpp
  cudaDeviceProp props;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 299-299 / 第299-299行

```cpp
  cudaError_t error = cudaGetDeviceProperties(&props, 0);
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 300-301 / 第300-301行

```cpp
  if (error != cudaSuccess) {
    std::cerr << "cudaGetDeviceProperties() returned an error: " << cudaGetErrorString(error) << std::endl;
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 302-302 / 第302-302行

```cpp
    return -1;
```

**EN**: This return statement propagates the current result back to the caller so the surrounding harness can continue its control flow.  
**CN**: 这个返回语句会把当前结果传回调用者，使外围测试框架能够继续执行后续控制流程。

### Lines 303-303 / 第303-303行

```cpp
  }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 305-309 / 第305-309行

```cpp
  if (props.major * 10 + props.minor < 80) {
    std::cerr << "Ampere Tensor Core operations must be run on a machine with compute capability at least 80."
              << std::endl;
    notSupported = true;
  }
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 311-313 / 第311-313行

```cpp
  if (notSupported) {
    // Returning zero so this test passes on older Toolkits. Its actions are no-op.
    return 0;
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 314-314 / 第314-314行

```cpp
  }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 316-316 / 第316-316行

```cpp
  return run();
```

**EN**: This return statement propagates the current result back to the caller so the surrounding harness can continue its control flow.  
**CN**: 这个返回语句会把当前结果传回调用者，使外围测试框架能够继续执行后续控制流程。

### Lines 317-317 / 第317-317行

```cpp
}
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

---

## Key Concepts / 关键概念

- Epilogue fusion and output operators / Epilogue 融合与输出算子
- Tensor Core execution / Tensor Core 执行
- Structured sparsity on Ampere / Ampere 上的结构化稀疏

## Dependencies / 依赖项

- `<iostream>` — Console stream utilities for logging progress and results. / 用于记录进度和结果的控制台流工具。
- `"cutlass/cutlass.h"` — Core CUTLASS definitions, architecture tags, and status types. / CUTLASS 的核心定义、架构标签与状态类型。
- `"cutlass/gemm/device/gemm_sparse.h"` — Device-level GEMM wrapper that hides kernel selection and launch details. / 设备级 GEMM 封装，隐藏内核选择与启动细节。
- `"cutlass/util/host_tensor.h"` — Host/device tensor wrapper used to allocate storage and transfer data. / 主机/设备张量封装，用于分配存储并传输数据。
- `"cutlass/util/reference/host/gemm.h"` — Host-side reference implementation used for correctness checking. / 用于正确性检查的主机端参考实现。
- `"cutlass/util/host_reorder.h"` — Provides `cutlass/util/host_reorder.h` so this file can use the related API or helper utilities. / 提供 `cutlass/util/host_reorder.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/util/host_uncompress.h"` — Provides `cutlass/util/host_uncompress.h` so this file can use the related API or helper utilities. / 提供 `cutlass/util/host_uncompress.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/util/reference/host/tensor_compare.h"` — Host-side reference implementation used for correctness checking. / 用于正确性检查的主机端参考实现。
- `"cutlass/util/reference/host/tensor_copy.h"` — Host-side reference implementation used for correctness checking. / 用于正确性检查的主机端参考实现。
- `"cutlass/util/reference/host/tensor_fill.h"` — Host-side reference implementation used for correctness checking. / 用于正确性检查的主机端参考实现。
- `"cutlass/util/tensor_view_io.h"` — Tensor printing helpers for debugging layouts and values. / 用于调试布局和值的张量打印辅助工具。
- `"helper.h"` — Provides `helper.h` so this file can use the related API or helper utilities. / 提供 `helper.h`，使本文件能够使用相关 API 或辅助工具。
