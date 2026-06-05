# grouped_threadblock_swizzle.h — Code Analysis / 代码分析

**Source / 源文件**: `examples/13_two_tensor_op_fusion/threadblock/grouped_threadblock_swizzle.h`  
**Purpose / 用途**: Defines swizzle policies that map grouped fused GEMM problems onto thread blocks. / 定义 swizzle 策略，用于把分组融合 GEMM 问题映射到线程块。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-31 / 第1-31行

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
/*! \file
```

**EN**: This is the standard BSD-3-Clause license header. It documents redistribution terms before any executable code appears.  
**CN**: 这里是标准的 BSD-3-Clause 许可证头，在任何可执行代码之前先说明再分发条款。

### Lines 32-33 / 第32-33行

```cpp
    \brief Implements several threadblock-swizzling functions for grouped kernels
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
#include "cutlass/gemm/kernel/grouped_problem_visitor.h"
#include "cutlass/gemm/kernel/gemm_grouped_problem_visitor.h"
#include "kernel/b2b_gemm_grouped_problem_visitor.h"
```

**EN**: These headers pull in the building blocks required by this file. Core CUTLASS definitions, architecture tags, and status types. Kernel-level GEMM building blocks and default kernel assemblers. Kernel-level GEMM building blocks and default kernel assemblers. Example-local kernel definition used to compose specialized fused execution paths. Visitor-based epilogues traverse accumulator fragments and apply multiple fused operations in a programmable sequence.  
**CN**: 这些头文件引入了当前文件所需的构建模块。CUTLASS 的核心定义、架构标签与状态类型。内核级 GEMM 构建模块与默认内核拼装器。内核级 GEMM 构建模块与默认内核拼装器。示例本地内核定义，用于组合专门化的融合执行路径。 visitor 式 epilogue 会遍历累加器分片，并按可编程顺序执行多个融合操作。

### Lines 42-42 / 第42-42行

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 44-44 / 第44-44行

```cpp
namespace cutlass {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 45-45 / 第45-45行

```cpp
namespace gemm {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 46-46 / 第46-46行

```cpp
namespace threadblock {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 48-48 / 第48-48行

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 50-50 / 第50-50行

```cpp
namespace detail {
```

**EN**: The code opens the namespace scope that owns the following CUTLASS or example-specific types.  
**CN**: 这里打开命名空间作用域，使后续类型归属于对应的 CUTLASS 或示例命名空间。

### Lines 52-52 / 第52-52行

```cpp
struct GroupedThreadblockSwizzleBase {};
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates. Grouped execution lets one launch process many problems with different shapes and uses a scheduler to distribute them across thread blocks.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。 分组执行允许一次启动处理多个不同形状的问题，并通过调度器把它们分配给线程块。

### Lines 54-54 / 第54-54行

```cpp
/// Helper for determining if a swizzling function is specialized for grouped operation
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 55-56 / 第55-56行

```cpp
template <typename ThreadblockSwizzle>
struct IsGroupedSwizzle {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates. Grouped execution lets one launch process many problems with different shapes and uses a scheduler to distribute them across thread blocks.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。 分组执行允许一次启动处理多个不同形状的问题，并通过调度器把它们分配给线程块。

### Lines 57-57 / 第57-57行

```cpp
  static bool const value = cutlass::platform::is_base_of<GroupedThreadblockSwizzleBase, ThreadblockSwizzle>::value;
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps. Grouped execution lets one launch process many problems with different shapes and uses a scheduler to distribute them across thread blocks.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。 分组执行允许一次启动处理多个不同形状的问题，并通过调度器把它们分配给线程块。

### Lines 58-58 / 第58-58行

```cpp
};
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 60-60 / 第60-60行

```cpp
} // namespace detail
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 62-62 / 第62-62行

```cpp
/// Swizzling function for grouped kernels
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 63-64 / 第63-64行

```cpp
template <typename ProblemVisitor_>
struct GroupedThreadblockSwizzle : detail::GroupedThreadblockSwizzleBase {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates. Grouped execution lets one launch process many problems with different shapes and uses a scheduler to distribute them across thread blocks. Visitor-based epilogues traverse accumulator fragments and apply multiple fused operations in a programmable sequence.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。 分组执行允许一次启动处理多个不同形状的问题，并通过调度器把它们分配给线程块。 visitor 式 epilogue 会遍历累加器分片，并按可编程顺序执行多个融合操作。

### Lines 66-66 / 第66-66行

```cpp
  using ProblemVisitor = ProblemVisitor_;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow. Visitor-based epilogues traverse accumulator fragments and apply multiple fused operations in a programmable sequence.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。 visitor 式 epilogue 会遍历累加器分片，并按可编程顺序执行多个融合操作。

### Lines 67-67 / 第67-67行

```cpp
  ProblemVisitor problem_visitor;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow. Visitor-based epilogues traverse accumulator fragments and apply multiple fused operations in a programmable sequence.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。 visitor 式 epilogue 会遍历累加器分片，并按可编程顺序执行多个融合操作。

### Lines 69-72 / 第69-72行

```cpp
  CUTLASS_HOST_DEVICE
  GroupedThreadblockSwizzle(typename ProblemVisitor::Params& params,
                            typename ProblemVisitor::SharedStorage& shared_storage,
                            int block_idx) : problem_visitor(params, shared_storage, block_idx) {}
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow. Grouped execution lets one launch process many problems with different shapes and uses a scheduler to distribute them across thread blocks. Visitor-based epilogues traverse accumulator fragments and apply multiple fused operations in a programmable sequence.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。 分组执行允许一次启动处理多个不同形状的问题，并通过调度器把它们分配给线程块。 visitor 式 epilogue 会遍历累加器分片，并按可编程顺序执行多个融合操作。

### Lines 74-74 / 第74-74行

```cpp
  /// Obtains the threadblock offset (in units of threadblock-scoped tiles)
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 75-77 / 第75-77行

```cpp
  CUTLASS_DEVICE
  GemmCoord get_tile_offset(int /*log_tile*/) const {
    GemmCoord problem_size = problem_visitor.problem_size();
```

**EN**: This block defines concrete problem sizes. For GEMM-like code, `GemmCoord(M, N, K)` specifies the row, column, and reduction dimensions that the kernel will process. Visitor-based epilogues traverse accumulator fragments and apply multiple fused operations in a programmable sequence.  
**CN**: 这段代码定义了具体的问题规模。对于 GEMM 类代码，`GemmCoord(M, N, K)` 分别表示内核要处理的行、列与归约维度。 visitor 式 epilogue 会遍历累加器分片，并按可编程顺序执行多个融合操作。

### Lines 78-79 / 第78-79行

```cpp
    int32_t threadblock_idx = int32_t(problem_visitor.threadblock_idx());
    GemmCoord grid_shape = problem_visitor.grid_shape(problem_size);
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps. Visitor-based epilogues traverse accumulator fragments and apply multiple fused operations in a programmable sequence.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。 visitor 式 epilogue 会遍历累加器分片，并按可编程顺序执行多个融合操作。

### Lines 81-83 / 第81-83行

```cpp
    return GemmCoord(int(threadblock_idx / grid_shape.n()),
                     int(threadblock_idx % grid_shape.n()),
                     0);
```

**EN**: This return statement propagates the current result back to the caller so the surrounding harness can continue its control flow.  
**CN**: 这个返回语句会把当前结果传回调用者，使外围测试框架能够继续执行后续控制流程。

### Lines 84-84 / 第84-84行

```cpp
  }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 86-86 / 第86-86行

```cpp
  /// Dummy method to satisfy API for threadblock swizzling functions
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 87-89 / 第87-89行

```cpp
  CUTLASS_HOST_DEVICE
  static int get_log_tile(GemmCoord /*tiled_shape*/) {
    return 0;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 90-90 / 第90-90行

```cpp
  }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 91-91 / 第91-91行

```cpp
};
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 93-107 / 第93-107行

```cpp
template <
  typename ThreadblockShape,
  typename LayoutC,
  cutlass::gemm::kernel::GroupScheduleMode GroupScheduleMode_ = cutlass::gemm::kernel::GroupScheduleMode::kDeviceOnly,
  int PrefetchTileCount = 128,
  int ThreadCount = PrefetchTileCount>
struct B2bGemmGroupedThreadblockSwizzle : GroupedThreadblockSwizzle<
                                            cutlass::gemm::kernel::B2bGemmGroupedProblemVisitor<
                                              ThreadblockShape,
                                              GroupScheduleMode_,
                                              PrefetchTileCount,
                                              ThreadCount,
                                              platform::is_same<LayoutC, cutlass::layout::ColumnMajor>::value
                                            >
                                          > {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. Grouped execution lets one launch process many problems with different shapes and uses a scheduler to distribute them across thread blocks. Visitor-based epilogues traverse accumulator fragments and apply multiple fused operations in a programmable sequence.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 分组执行允许一次启动处理多个不同形状的问题，并通过调度器把它们分配给线程块。 visitor 式 epilogue 会遍历累加器分片，并按可编程顺序执行多个融合操作。

### Lines 108-113 / 第108-113行

```cpp
  using Base = GroupedThreadblockSwizzle<cutlass::gemm::kernel::B2bGemmGroupedProblemVisitor<
                                          ThreadblockShape,
                                          GroupScheduleMode_,
                                          PrefetchTileCount,
                                          ThreadCount,
                                          platform::is_same<LayoutC, cutlass::layout::ColumnMajor>::value>>;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth. `ThreadblockShape` selects the CTA-sized tile computed by one thread block. Grouped execution lets one launch process many problems with different shapes and uses a scheduler to distribute them across thread blocks. Visitor-based epilogues traverse accumulator fragments and apply multiple fused operations in a programmable sequence.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。 分组执行允许一次启动处理多个不同形状的问题，并通过调度器把它们分配给线程块。 visitor 式 epilogue 会遍历累加器分片，并按可编程顺序执行多个融合操作。

### Lines 115-118 / 第115-118行

```cpp
  CUTLASS_HOST_DEVICE
  B2bGemmGroupedThreadblockSwizzle(typename Base::ProblemVisitor::Params& params,
                                   typename Base::ProblemVisitor::SharedStorage& shared_storage,
                                   int block_idx) : Base(params, shared_storage, block_idx) {}
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow. Grouped execution lets one launch process many problems with different shapes and uses a scheduler to distribute them across thread blocks. Visitor-based epilogues traverse accumulator fragments and apply multiple fused operations in a programmable sequence.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。 分组执行允许一次启动处理多个不同形状的问题，并通过调度器把它们分配给线程块。 visitor 式 epilogue 会遍历累加器分片，并按可编程顺序执行多个融合操作。

### Lines 119-119 / 第119-119行

```cpp
};
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 121-121 / 第121-121行

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 123-125 / 第123-125行

```cpp
} // namespace threadblock
} // namespace gemm
} // namespace cutlass
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

---

## Key Concepts / 关键概念

- Back-to-back GEMM fusion / 前后相接 GEMM 融合
- Grouped scheduling for heterogeneous problems / 面向异构问题的分组调度
- Hierarchical tiling: threadblock / warp / instruction / 分层分块：threadblock / warp / instruction
- Visitor-based epilogue traversal / 基于 Visitor 的 epilogue 遍历

## Dependencies / 依赖项

- `"cutlass/cutlass.h"` — Core CUTLASS definitions, architecture tags, and status types. / CUTLASS 的核心定义、架构标签与状态类型。
- `"cutlass/gemm/kernel/grouped_problem_visitor.h"` — Kernel-level GEMM building blocks and default kernel assemblers. / 内核级 GEMM 构建模块与默认内核拼装器。
- `"cutlass/gemm/kernel/gemm_grouped_problem_visitor.h"` — Kernel-level GEMM building blocks and default kernel assemblers. / 内核级 GEMM 构建模块与默认内核拼装器。
- `"kernel/b2b_gemm_grouped_problem_visitor.h"` — Example-local kernel definition used to compose specialized fused execution paths. / 示例本地内核定义，用于组合专门化的融合执行路径。
