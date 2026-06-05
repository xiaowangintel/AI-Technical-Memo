# tensor_scale_bias.h — Code Analysis / 代码分析

**Source / 源文件**: `examples/13_two_tensor_op_fusion/reference/device/tensor_scale_bias.h`  
**Purpose / 用途**: Implements a reference device utility that applies scaling and bias to tensors for validation. / 实现参考设备工具，用于对张量应用缩放和偏置，以便验证结果。

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
/* \file
```

**EN**: This is the standard BSD-3-Clause license header. It documents redistribution terms before any executable code appears.  
**CN**: 这里是标准的 BSD-3-Clause 许可证头，在任何可执行代码之前先说明再分发条款。

### Lines 32-36 / 第32-36行

```cpp
  \brief Defines device-side elementwise operations on TensorView. Note, the operations defined
    in this header are not specialized for any particular data layout and are therefore not
    intended to offer the best possible performance. Rather, they are intended to be generic
    reference implementations to support the CUTLASS unit tests.
*/
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 38-38 / 第38-38行

```cpp
#pragma once
```

**EN**: This pragma makes the header idempotent so repeated inclusion does not create duplicate definitions.  
**CN**: 这个 pragma 保证头文件只会被处理一次，避免重复包含产生重复定义。

### Lines 40-40 / 第40-40行

```cpp
// Cutlass includes
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 41-42 / 第41-42行

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/tensor_view.h"
```

**EN**: These headers pull in the building blocks required by this file. Core CUTLASS definitions, architecture tags, and status types. Provides `cutlass/tensor_view.h` so this file can use the related API or helper utilities.  
**CN**: 这些头文件引入了当前文件所需的构建模块。CUTLASS 的核心定义、架构标签与状态类型。提供 `cutlass/tensor_view.h`，使本文件能够使用相关 API 或辅助工具。

### Lines 44-44 / 第44-44行

```cpp
#include "cutlass/gemm/gemm.h"
```

**EN**: These headers pull in the building blocks required by this file. Provides `cutlass/gemm/gemm.h` so this file can use the related API or helper utilities.  
**CN**: 这些头文件引入了当前文件所需的构建模块。提供 `cutlass/gemm/gemm.h`，使本文件能够使用相关 API 或辅助工具。

### Lines 46-46 / 第46-46行

```cpp
///////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 48-48 / 第48-48行

```cpp
namespace cutlass {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 49-49 / 第49-49行

```cpp
namespace reference {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 50-50 / 第50-50行

```cpp
namespace device {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 52-52 / 第52-52行

```cpp
///////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 54-54 / 第54-54行

```cpp
namespace kernel {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 56-71 / 第56-71行

```cpp
template <
  typename TensorRefIn,                   ///< Input TensorRef Type
  typename TensorRefOut,                  ///< Output TensorRef Type
  typename ScalarType,                    ///< alpha Type
  typename TensorRefScalar,               ///< Scale/Bias TensorRef Type
  typename OutputTile,
  typename ConvertOp = NumericConverter<typename TensorRefOut::Element, ScalarType>
>
__global__ void TensorScaleBiasGemm(
  gemm::GemmCoord problem_size,
  TensorRefIn tensor_in,                  ///< input tensor
  TensorRefOut tensor_out,                ///< output tensor
  ScalarType alpha,                       ///< alpha
  TensorRefScalar tensor_scale,           ///< scale tensor
  TensorRefScalar tensor_bias             ///< bias tensor
) {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。

### Lines 73-73 / 第73-73行

```cpp
  ConvertOp convert_op;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 75-78 / 第75-78行

```cpp
  MatrixCoord output_coord(
    MatrixCoord::Index((threadIdx.x + blockIdx.x * blockDim.x) * OutputTile::kRow),
    MatrixCoord::Index((threadIdx.y + blockIdx.y * blockDim.y) * OutputTile::kColumn)
  );
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 80-80 / 第80-80行

```cpp
  // Update the output tensor
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 81-96 / 第81-96行

```cpp
  for (int j = 0; j < OutputTile::kRow; ++j) {
    for (int i = 0; i < OutputTile::kColumn; ++i) {
      MatrixCoord coord = output_coord + MatrixCoord(i, j);
      if (coord.row() < problem_size.m() && coord.column() < problem_size.n()) {
        ScalarType scale = alpha;
        if(tensor_scale.good())
          scale = tensor_scale.at({0, coord.column()});
        ScalarType bias = ScalarType(0);
        if(tensor_bias.good())
          bias = tensor_bias.at({0, coord.column()});
        tensor_out.at(coord) = convert_op(
          scale * ScalarType(tensor_in.at(coord)) + bias);
      }
    }
  }
}
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 102-123 / 第102-123行

```cpp
template <
  typename TensorRefIn,                   ///< Input TensorRef Type
  typename TensorRefOut,                  ///< Output TensorRef Type
  typename ScalarType,                    ///< alpha Type
  typename TensorRefScalar,               ///< Scale/Bias TensorRef Type
  typename ConvertOp = NumericConverter<typename TensorRefOut::Element, ScalarType>,
  int kMblock = 4,
  int kNblock = 4
>
__global__ void TensorScaleBiasGemmBatched(
  gemm::GemmCoord problem_size,
  TensorRefIn tensor_in,                  ///< input tensor
  TensorRefOut tensor_out,                ///< output tensor
  ScalarType alpha,                       ///< alpha
  TensorRefScalar tensor_scale,           ///< scale tensor
  TensorRefScalar tensor_bias,             ///< bias tensor
  int batch_count = 1,
  int64_t batch_stride_tensor_in = 0,
  int64_t batch_stride_tensor_out = 0,
  int64_t batch_stride_tensor_scale = 0,
  int64_t batch_stride_tensor_bias = 0
) {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。

### Lines 125-125 / 第125-125行

```cpp
  ConvertOp convert_op;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 126-128 / 第126-128行

```cpp
  int row_block = (blockIdx.x * blockDim.x + threadIdx.x) * kMblock;
  int col_block = (blockIdx.y * blockDim.y + threadIdx.y) * kNblock;
  int batch_idx = blockIdx.z;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 130-130 / 第130-130行

```cpp
  tensor_in.add_pointer_offset(batch_idx * batch_stride_tensor_in);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 131-131 / 第131-131行

```cpp
  tensor_out.add_pointer_offset(batch_idx * batch_stride_tensor_out);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 132-132 / 第132-132行

```cpp
  tensor_scale.add_pointer_offset(batch_idx * batch_stride_tensor_scale);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 133-133 / 第133-133行

```cpp
  tensor_bias.add_pointer_offset(batch_idx * batch_stride_tensor_bias);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 135-160 / 第135-160行

```cpp
  for (; batch_idx < batch_count; batch_idx += gridDim.z) {
    CUTLASS_PRAGMA_UNROLL
    for (int j = 0; j < kNblock; j++) {
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < kMblock; i++) {
        int row = row_block + i;
        int col = col_block + j;
        MatrixCoord coord = MatrixCoord(row, col);
        if (coord.row() < problem_size.m() && coord.column() < problem_size.n()) {
          ScalarType scale = alpha;
          if(tensor_scale.good())
            scale = tensor_scale.at({0, coord.column()});
          ScalarType bias = ScalarType(0);
          if(tensor_bias.good())
            bias = tensor_bias.at({0, coord.column()});
          tensor_out.at(coord) = convert_op(
            scale * ScalarType(tensor_in.at(coord)) + bias);
        }
      }
    }
    tensor_in.add_pointer_offset(batch_stride_tensor_in * gridDim.z);
    tensor_out.add_pointer_offset(batch_stride_tensor_out * gridDim.z);
    tensor_scale.add_pointer_offset(batch_stride_tensor_scale * gridDim.z);
    tensor_bias.add_pointer_offset(batch_stride_tensor_bias * gridDim.z);
  }
}
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 166-184 / 第166-184行

```cpp
template <
  typename TensorRefIn,                   ///< Input TensorRef Type
  typename TensorRefOut,                  ///< Output TensorRef Type
  typename ScalarType,                    ///< alpha Type
  typename TensorRefScalar,               ///< Scale/Bias TensorRef Type
  typename ConvertOp = NumericConverter<typename TensorRefOut::Element, ScalarType>,
  int kThreadM = 4,       // shape of a thread's tile in the GEMM M dimension
  int kThreadN = 4,       // shape of a thread's tile in the GEMM N dimension
  int kCtaShapeM = 16,    // shape of a threadblock in units of threads
  int kCtaShapeN = 8      // shape of a threadblock in units of threads
>
__global__ void TensorScaleBiasConv2d(
  conv::Conv2dProblemSize problem_size,
  TensorRefIn tensor_in,                  ///< input tensor
  TensorRefOut tensor_out,                ///< output tensor
  ScalarType alpha,                       ///< alpha
  TensorRefScalar tensor_scale,           ///< scale tensor
  TensorRefScalar tensor_bias             ///< bias tensor
) {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。

### Lines 186-186 / 第186-186行

```cpp
  ConvertOp convert_op;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 188-189 / 第188-189行

```cpp
  int64_t npq_start = int64_t(blockIdx.x) * kCtaShapeM * kThreadM + threadIdx.x * kThreadM;
  int k_start = blockIdx.y * kCtaShapeN * kThreadN + threadIdx.y * kThreadN;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 191-191 / 第191-191行

```cpp
  int thread_n[kThreadM];
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 192-192 / 第192-192行

```cpp
  int thread_p[kThreadM];
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 193-193 / 第193-193行

```cpp
  int thread_q[kThreadM];
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 195-195 / 第195-195行

```cpp
  // Compute N, P, Q coordinates for each row of a thread's tile
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 196-196 / 第196-196行

```cpp
  int64_t PQ = int64_t(problem_size.P) * problem_size.Q;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 198-205 / 第198-205行

```cpp
  CUTLASS_PRAGMA_UNROLL
  for (int m = 0; m < kThreadM; ++m) {
    int64_t npq = npq_start + m;
    thread_n[m] = int(npq / PQ);
    int64_t residual = npq % PQ;
    thread_p[m] = int(residual / problem_size.Q);
    thread_q[m] = int(residual % problem_size.Q);
  }
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 210-210 / 第210-210行

```cpp
  // Write out the results
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 211-231 / 第211-231行

```cpp
  CUTLASS_PRAGMA_UNROLL
  for (int m = 0; m < kThreadM; ++m) {
    if (thread_n[m] < problem_size.N && thread_p[m] < problem_size.P && thread_q[m] < problem_size.Q) {
      CUTLASS_PRAGMA_UNROLL
      for (int n = 0; n < kThreadN; ++n) {
        int thread_k = k_start + n;
        if (thread_k < problem_size.K) {
          ScalarType scale = alpha;
          if(tensor_scale.good())
            scale = tensor_scale.at({0, thread_k});
          ScalarType bias = ScalarType(0);
          if(tensor_bias.good())
            bias = tensor_bias.at({0, thread_k});
          tensor_out.at({thread_n[m], thread_p[m], thread_q[m], thread_k}) = convert_op(
            scale * ScalarType(
              tensor_in.at({thread_n[m], thread_p[m], thread_q[m], thread_k})
            ) + bias);
        }
      }
    }
  }
```

**EN**: This block initializes runtime scalars such as `alpha` and `beta`. They control how the epilogue scales accumulated results and mixes in source or bias terms.  
**CN**: 这段代码初始化 `alpha`、`beta` 等运行时标量。它们控制 epilogue 如何缩放累加结果，并混入源项或偏置项。

### Lines 236-236 / 第236-236行

```cpp
}
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 238-238 / 第238-238行

```cpp
}
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 240-240 / 第240-240行

```cpp
/// Apply scale and bias on a tensor
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 241-256 / 第241-256行

```cpp
template <
  typename ElementIn,                   ///< Input Type
  typename ElementOut,                  ///< Output Type
  typename Layout,                      ///< Layout of input/output tensor
  typename ScalarType,                  ///< alpha Type
  typename LayoutScaleBias,             ///< Layout of scale and bias
  typename ConvertOp = NumericConverter<ElementOut, ScalarType>
>
void TensorScaleBiasGemm(
  gemm::GemmCoord problem_size,
  TensorRef<ElementIn, Layout> tensor_in,              ///< input tensor
  TensorRef<ElementOut, Layout> tensor_out,            ///< output tensor
  ScalarType alpha,                                    ///< alpha
  TensorRef<ScalarType, LayoutScaleBias> tensor_scale, ///< scale tensor
  TensorRef<ScalarType, LayoutScaleBias> tensor_bias    ///< bias tensor
) {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。

### Lines 258-258 / 第258-258行

```cpp
  using OutputTile = MatrixShape<4, 4>;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 260-260 / 第260-260行

```cpp
  dim3 block(16, 8);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 262-265 / 第262-265行

```cpp
  dim3 grid(
    (problem_size.m() + block.x * OutputTile::kRow - 1) / (block.x * OutputTile::kRow),
    (problem_size.n() + block.y * OutputTile::kColumn - 1) / (block.y * OutputTile::kColumn)
  );
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 267-281 / 第267-281行

```cpp
  kernel::TensorScaleBiasGemm<
    TensorRef<ElementIn, Layout>,
    TensorRef<ElementOut, Layout>,
    ScalarType,
    TensorRef<ScalarType, LayoutScaleBias>,
    OutputTile,
    ConvertOp
  ><<< grid, block >>> (
    problem_size,
    tensor_in,
    tensor_out,
    alpha,
    tensor_scale,
    tensor_bias
  );
```

**EN**: This block initializes runtime scalars such as `alpha` and `beta`. They control how the epilogue scales accumulated results and mixes in source or bias terms.  
**CN**: 这段代码初始化 `alpha`、`beta` 等运行时标量。它们控制 epilogue 如何缩放累加结果，并混入源项或偏置项。

### Lines 282-282 / 第282-282行

```cpp
}
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 284-284 / 第284-284行

```cpp
/// Apply scale and bias on a tensor
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 285-305 / 第285-305行

```cpp
template <
  typename ElementIn,                   ///< Input Type
  typename ElementOut,                  ///< Output Type
  typename Layout,                      ///< Layout of input/output tensor
  typename ScalarType,                  ///< alpha Type
  typename LayoutScaleBias,             ///< Layout of scale and bias
  typename ConvertOp = NumericConverter<ElementOut, ScalarType>
>
void TensorScaleBiasGemmBatched(
  gemm::GemmCoord problem_size,
  TensorRef<ElementIn, Layout> tensor_in,              ///< input tensor
  TensorRef<ElementOut, Layout> tensor_out,            ///< output tensor
  ScalarType alpha,                                    ///< alpha
  TensorRef<ScalarType, LayoutScaleBias> tensor_scale, ///< scale tensor
  TensorRef<ScalarType, LayoutScaleBias> tensor_bias,    ///< bias tensor
  int batch_count = 1,
  int64_t batch_stride_tensor_in = 0,
  int64_t batch_stride_tensor_out = 0,
  int64_t batch_stride_tensor_scale = 0,
  int64_t batch_stride_tensor_bias = 0
) {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。

### Lines 307-308 / 第307-308行

```cpp
  int const kMblock = 4;
  int const kNblock = 4;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 310-310 / 第310-310行

```cpp
  dim3 block(16, 8);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 311-315 / 第311-315行

```cpp
  dim3 grid(
    (problem_size.m() + block.x * kMblock - 1) / (block.x * kMblock),
    (problem_size.n() + block.y * kNblock - 1) / (block.y * kNblock),
    batch_count % std::numeric_limits<uint16_t>::max()
  );
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 317-337 / 第317-337行

```cpp
  kernel::TensorScaleBiasGemmBatched<
    TensorRef<ElementIn, Layout>,
    TensorRef<ElementOut, Layout>,
    ScalarType,
    TensorRef<ScalarType, LayoutScaleBias>,
    ConvertOp,
    kMblock,
    kNblock
  ><<< grid, block >>> (
    problem_size,
    tensor_in,
    tensor_out,
    alpha,
    tensor_scale,
    tensor_bias,
    batch_count,
    batch_stride_tensor_in,
    batch_stride_tensor_out,
    batch_stride_tensor_scale,
    batch_stride_tensor_bias
  );
```

**EN**: This block initializes runtime scalars such as `alpha` and `beta`. They control how the epilogue scales accumulated results and mixes in source or bias terms.  
**CN**: 这段代码初始化 `alpha`、`beta` 等运行时标量。它们控制 epilogue 如何缩放累加结果，并混入源项或偏置项。

### Lines 338-338 / 第338-338行

```cpp
}
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 340-340 / 第340-340行

```cpp
/// Apply scale and bias on a tensor
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 341-356 / 第341-356行

```cpp
template <
  typename ElementIn,                   ///< Input Type
  typename ElementOut,                  ///< Output Type
  typename Layout,                      ///< Layout of input/output tensor
  typename ScalarType,                  ///< alpha Type
  typename LayoutScaleBias,             ///< Layout of scale and bias
  typename ConvertOp = NumericConverter<ElementOut, ScalarType>
>
void TensorScaleBiasConv2d(
  conv::Conv2dProblemSize problem_size,
  TensorRef<ElementIn, Layout> tensor_in,              ///< input tensor
  TensorRef<ElementOut, Layout> tensor_out,            ///< output tensor
  ScalarType alpha,                                    ///< alpha
  TensorRef<ScalarType, LayoutScaleBias> tensor_scale, ///< scale tensor
  TensorRef<ScalarType, LayoutScaleBias> tensor_bias    ///< bias tensor
) {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。

### Lines 358-361 / 第358-361行

```cpp
  int const kThreadM = 4;       // shape of a thread's tile in the GEMM M dimension
  int const kThreadN = 4;       // shape of a thread's tile in the GEMM N dimension
  int const kCtaShapeM = 16;    // shape of a threadblock in units of threads
  int const kCtaShapeN = 8;     // shape of a threadblock in units of threads
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 363-364 / 第363-364行

```cpp
  int64_t npq = int64_t(problem_size.N) * problem_size.P * problem_size.Q;
  int64_t blocks_m = (npq + (kCtaShapeM * kThreadM) - 1) / (kCtaShapeM * kThreadM);
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 366-366 / 第366-366行

```cpp
  dim3 block(kCtaShapeM, kCtaShapeN);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 367-367 / 第367-367行

```cpp
  dim3 grid(uint32_t(blocks_m), (problem_size.K + (kCtaShapeN * kThreadN) - 1) / (kCtaShapeN * kThreadN));
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 370-387 / 第370-387行

```cpp
  kernel::TensorScaleBiasConv2d<
    TensorRef<ElementIn, Layout>,
    TensorRef<ElementOut, Layout>,
    ScalarType,
    TensorRef<ScalarType, LayoutScaleBias>,
    ConvertOp,
    kThreadM,
    kThreadN,
    kCtaShapeM,
    kCtaShapeN
  ><<< grid, block >>> (
    problem_size,
    tensor_in,
    tensor_out,
    alpha,
    tensor_scale,
    tensor_bias
  );
```

**EN**: This block initializes runtime scalars such as `alpha` and `beta`. They control how the epilogue scales accumulated results and mixes in source or bias terms.  
**CN**: 这段代码初始化 `alpha`、`beta` 等运行时标量。它们控制 epilogue 如何缩放累加结果，并混入源项或偏置项。

### Lines 388-388 / 第388-388行

```cpp
}
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 390-391 / 第390-391行

```cpp
///////////////////////////////////////////////////////////////////////////////////////////////////
///////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 393-395 / 第393-395行

```cpp
} // namespace device
} // namespace reference
} // namespace cutlass
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

---

## Key Concepts / 关键概念

- Implicit-GEMM convolution mapping / 隐式 GEMM 卷积映射
- Template-driven kernel specialization / 模板驱动的内核特化

## Dependencies / 依赖项

- `"cutlass/cutlass.h"` — Core CUTLASS definitions, architecture tags, and status types. / CUTLASS 的核心定义、架构标签与状态类型。
- `"cutlass/tensor_view.h"` — Provides `cutlass/tensor_view.h` so this file can use the related API or helper utilities. / 提供 `cutlass/tensor_view.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/gemm/gemm.h"` — Provides `cutlass/gemm/gemm.h` so this file can use the related API or helper utilities. / 提供 `cutlass/gemm/gemm.h`，使本文件能够使用相关 API 或辅助工具。
