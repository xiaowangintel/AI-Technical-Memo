# b2b_gemm_grouped_problem_visitor.h — Code Analysis / 代码分析

**Source / 源文件**: `examples/13_two_tensor_op_fusion/kernel/b2b_gemm_grouped_problem_visitor.h`  
**Purpose / 用途**: Defines the grouped-problem visitor used to schedule fused GEMM work across heterogeneous problem sets. / 定义分组问题访问器，用于在异构问题集合上调度融合 GEMM 工作。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-30 / 第1-30行

```cpp
/***************************************************************************************************
 * Copyright (c) 2023 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

### Lines 33-34 / 第33-34行

```cpp
    \brief Scheduler for grouped B2b GEMMs
*/
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 36-36 / 第36-36行

```cpp
#pragma once
```

**EN**: This pragma makes the header idempotent so repeated inclusion does not create duplicate definitions.  
**CN**: 这个 pragma 保证头文件只会被处理一次，避免重复包含产生重复定义。

### Lines 38-42 / 第38-42行

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/matrix_coord.h"
#include "cutlass/gemm/kernel/grouped_problem_visitor.h"
#include "cutlass/gemm/kernel/gemm_grouped_problem_visitor.h"
```

**EN**: These headers pull in the building blocks required by this file. Core CUTLASS definitions, architecture tags, and status types. Provides `cutlass/gemm/gemm.h` so this file can use the related API or helper utilities. Provides `cutlass/matrix_coord.h` so this file can use the related API or helper utilities. Kernel-level GEMM building blocks and default kernel assemblers. Visitor-based epilogues traverse accumulator fragments and apply multiple fused operations in a programmable sequence.  
**CN**: 这些头文件引入了当前文件所需的构建模块。CUTLASS 的核心定义、架构标签与状态类型。提供 `cutlass/gemm/gemm.h`，使本文件能够使用相关 API 或辅助工具。提供 `cutlass/matrix_coord.h`，使本文件能够使用相关 API 或辅助工具。内核级 GEMM 构建模块与默认内核拼装器。 visitor 式 epilogue 会遍历累加器分片，并按可编程顺序执行多个融合操作。

### Lines 44-44 / 第44-44行

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 46-46 / 第46-46行

```cpp
namespace cutlass {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 47-47 / 第47-47行

```cpp
namespace gemm {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 48-48 / 第48-48行

```cpp
namespace kernel {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 50-50 / 第50-50行

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 52-52 / 第52-52行

```cpp
/// Visitor class to abstract away the algorithm for iterating over tiles
```

**EN**: This comment separates sections and documents the purpose of the code that follows. Visitor-based epilogues traverse accumulator fragments and apply multiple fused operations in a programmable sequence.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。 visitor 式 epilogue 会遍历累加器分片，并按可编程顺序执行多个融合操作。

### Lines 53-63 / 第53-63行

```cpp
template <typename ThreadblockShape,
          GroupScheduleMode GroupScheduleMode_,
          int PrefetchTileCount,
          int ThreadCount,
          bool Transposed = false>
struct B2bGemmGroupedProblemVisitor : public GroupedProblemVisitor<
                                            detail::GemmGroupedProblemSizeHelper<ThreadblockShape, Transposed>,
                                            ThreadblockShape,
                                            GroupScheduleMode_,
                                            PrefetchTileCount,
                                            ThreadCount> {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. Grouped execution lets one launch process many problems with different shapes and uses a scheduler to distribute them across thread blocks. Visitor-based epilogues traverse accumulator fragments and apply multiple fused operations in a programmable sequence.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 分组执行允许一次启动处理多个不同形状的问题，并通过调度器把它们分配给线程块。 visitor 式 epilogue 会遍历累加器分片，并按可编程顺序执行多个融合操作。

### Lines 65-68 / 第65-68行

```cpp
  using ProblemSizeHelper = detail::GemmGroupedProblemSizeHelper<ThreadblockShape, Transposed>;
  using Base = GroupedProblemVisitor<ProblemSizeHelper, ThreadblockShape, GroupScheduleMode_, PrefetchTileCount, ThreadCount>;
  using BaseParams = typename Base::Params;
  using SharedStorage = typename Base::SharedStorage;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. Grouped execution lets one launch process many problems with different shapes and uses a scheduler to distribute them across thread blocks. Visitor-based epilogues traverse accumulator fragments and apply multiple fused operations in a programmable sequence.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 分组执行允许一次启动处理多个不同形状的问题，并通过调度器把它们分配给线程块。 visitor 式 epilogue 会遍历累加器分片，并按可编程顺序执行多个融合操作。

### Lines 69-69 / 第69-69行

```cpp
  static bool const kTransposed = Transposed;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 71-71 / 第71-71行

```cpp
  cutlass::gemm::GemmCoord const *problem_sizes0;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 72-72 / 第72-72行

```cpp
  cutlass::gemm::GemmCoord const *problem_sizes1;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 74-74 / 第74-74行

```cpp
  struct Params {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。

### Lines 75-75 / 第75-75行

```cpp
    cutlass::gemm::GemmCoord const *problem_sizes0;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 76-76 / 第76-76行

```cpp
    cutlass::gemm::GemmCoord const *problem_sizes1;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 77-77 / 第77-77行

```cpp
    int32_t                         problem_count;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 78-78 / 第78-78行

```cpp
    void const                     *workspace;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 79-79 / 第79-79行

```cpp
    int32_t                         tile_count;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 81-83 / 第81-83行

```cpp
    //
    // Methods
    //
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 85-85 / 第85-85行

```cpp
    /// Ctor
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 86-88 / 第86-88行

```cpp
    CUTLASS_HOST_DEVICE
    Params(): problem_sizes0(nullptr), problem_sizes1(nullptr),
              problem_count(0), workspace(nullptr), tile_count(0) { }
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 90-90 / 第90-90行

```cpp
    /// Ctor
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 91-104 / 第91-104行

```cpp
    CUTLASS_HOST_DEVICE
    Params(
      cutlass::gemm::GemmCoord const *problem_sizes0,
      cutlass::gemm::GemmCoord const *problem_sizes1,
      int32_t                         problem_count,
      void const                     *workspace = nullptr,
      int32_t                         tile_count = 0
    ):
      problem_sizes0(problem_sizes0),
      problem_sizes1(problem_sizes1),
      problem_count(problem_count),
      workspace(workspace),
      tile_count(tile_count)
    {}
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 106-106 / 第106-106行

```cpp
    /// Convert the B2b-GEMM-specific parameters to those used by the base class
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 107-114 / 第107-114行

```cpp
    CUTLASS_HOST_DEVICE
    BaseParams to_base() const {
        return BaseParams(// Set problem_sizes as problem_sizes0 because these determine
                          // shape of the grid used in the non-grouped B2b GEMM
                          problem_sizes0,
                          problem_count,
                          workspace,
                          tile_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 115-115 / 第115-115行

```cpp
    }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 117-117 / 第117-117行

```cpp
  };
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 119-121 / 第119-121行

```cpp
  //
  // Methods
  //
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 122-132 / 第122-132行

```cpp
  CUTLASS_DEVICE
  B2bGemmGroupedProblemVisitor(
    Params const &params_,
    SharedStorage &shared_storage_, 
    int32_t block_idx
  ): Base (
        params_.to_base(),
        shared_storage_, block_idx),
     problem_sizes0(params_.problem_sizes0),
     problem_sizes1(params_.problem_sizes1)
  {}
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow. Grouped execution lets one launch process many problems with different shapes and uses a scheduler to distribute them across thread blocks. Visitor-based epilogues traverse accumulator fragments and apply multiple fused operations in a programmable sequence.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。 分组执行允许一次启动处理多个不同形状的问题，并通过调度器把它们分配给线程块。 visitor 式 epilogue 会遍历累加器分片，并按可编程顺序执行多个融合操作。

### Lines 134-134 / 第134-134行

```cpp
  /// Returns the problem size 0 for the current problem
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 135-137 / 第135-137行

```cpp
  CUTLASS_HOST_DEVICE
  cutlass::gemm::GemmCoord problem_size0() const {
    GemmCoord problem = problem_sizes0[this->problem_idx];
```

**EN**: This block defines concrete problem sizes. For GEMM-like code, `GemmCoord(M, N, K)` specifies the row, column, and reduction dimensions that the kernel will process.  
**CN**: 这段代码定义了具体的问题规模。对于 GEMM 类代码，`GemmCoord(M, N, K)` 分别表示内核要处理的行、列与归约维度。

### Lines 138-138 / 第138-138行

```cpp
    ProblemSizeHelper::possibly_transpose_problem(problem);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 139-139 / 第139-139行

```cpp
    return problem;
```

**EN**: This return statement propagates the current result back to the caller so the surrounding harness can continue its control flow.  
**CN**: 这个返回语句会把当前结果传回调用者，使外围测试框架能够继续执行后续控制流程。

### Lines 140-140 / 第140-140行

```cpp
  }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 142-142 / 第142-142行

```cpp
  /// Returns the problem size 1 for the current problem
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 143-145 / 第143-145行

```cpp
  CUTLASS_HOST_DEVICE
  cutlass::gemm::GemmCoord problem_size1() const {
    GemmCoord problem = problem_sizes1[this->problem_idx];
```

**EN**: This block defines concrete problem sizes. For GEMM-like code, `GemmCoord(M, N, K)` specifies the row, column, and reduction dimensions that the kernel will process.  
**CN**: 这段代码定义了具体的问题规模。对于 GEMM 类代码，`GemmCoord(M, N, K)` 分别表示内核要处理的行、列与归约维度。

### Lines 146-146 / 第146-146行

```cpp
    ProblemSizeHelper::possibly_transpose_problem(problem);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 147-147 / 第147-147行

```cpp
    return problem;
```

**EN**: This return statement propagates the current result back to the caller so the surrounding harness can continue its control flow.  
**CN**: 这个返回语句会把当前结果传回调用者，使外围测试框架能够继续执行后续控制流程。

### Lines 148-148 / 第148-148行

```cpp
  }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 149-149 / 第149-149行

```cpp
};
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 151-151 / 第151-151行

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 153-155 / 第153-155行

```cpp
} // namespace kernel
} // namespace gemm
} // namespace cutlass
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 157-157 / 第157-157行

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

---

## Key Concepts / 关键概念

- Back-to-back GEMM fusion / 前后相接 GEMM 融合
- Grouped scheduling for heterogeneous problems / 面向异构问题的分组调度
- Hierarchical tiling: threadblock / warp / instruction / 分层分块：threadblock / warp / instruction
- Visitor-based epilogue traversal / 基于 Visitor 的 epilogue 遍历

## Dependencies / 依赖项

- `"cutlass/cutlass.h"` — Core CUTLASS definitions, architecture tags, and status types. / CUTLASS 的核心定义、架构标签与状态类型。
- `"cutlass/gemm/gemm.h"` — Provides `cutlass/gemm/gemm.h` so this file can use the related API or helper utilities. / 提供 `cutlass/gemm/gemm.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/matrix_coord.h"` — Provides `cutlass/matrix_coord.h` so this file can use the related API or helper utilities. / 提供 `cutlass/matrix_coord.h`，使本文件能够使用相关 API 或辅助工具。
- `"cutlass/gemm/kernel/grouped_problem_visitor.h"` — Kernel-level GEMM building blocks and default kernel assemblers. / 内核级 GEMM 构建模块与默认内核拼装器。
- `"cutlass/gemm/kernel/gemm_grouped_problem_visitor.h"` — Kernel-level GEMM building blocks and default kernel assemblers. / 内核级 GEMM 构建模块与默认内核拼装器。
