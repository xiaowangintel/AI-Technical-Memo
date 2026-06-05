# gemm_splitk_parallel.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `include/cutlass/gemm/kernel/gemm_splitk_parallel.h`
- **Purpose / 用途 (EN):** Implements kernel-side support for GEMM splitk parallel. Briefly, the file comment says: Template for GEMM performing a reduction over K partitions in parallel.
- **Purpose / 用途 (CN):** 实现 GEMM splitk parallel 的内核侧支持逻辑。 文件注释的简要说明是：Template for GEMM performing a reduction over K partitions in parallel。
- **Line count / 行数:** 253

## Line-by-Line Analysis / 逐行分析
> The source is grouped into contiguous significant line ranges for readability. / 为了便于阅读，下面按连续的重要代码区间进行分析。

### Lines 1-20

```cpp
   1 | /***************************************************************************************************
   2 |  * Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
   3 |  * SPDX-License-Identifier: BSD-3-Clause
   4 |  *
   5 |  * Redistribution and use in source and binary forms, with or without
   6 |  * modification, are permitted provided that the following conditions are met:
   7 |  *
   8 |  * 1. Redistributions of source code must retain the above copyright notice, this
   9 |  * list of conditions and the following disclaimer.
  10 |  *
  11 |  * 2. Redistributions in binary form must reproduce the above copyright notice,
  12 |  * this list of conditions and the following disclaimer in the documentation
  13 |  * and/or other materials provided with the distribution.
  14 |  *
  15 |  * 3. Neither the name of the copyright holder nor the names of its
  16 |  * contributors may be used to endorse or promote products derived from
  17 |  * this software without specific prior written permission.
  18 |  *
  19 |  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
  20 |  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
```
**EN:** This opening comment records the license, copyright ownership, and redistribution conditions that apply to the header.
**CN:** 开头注释说明了该头文件适用的许可证、版权归属以及再分发条件。

### Lines 21-33

```cpp
  21 |  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
  22 |  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
  23 |  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
  24 |  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
  25 |  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
  26 |  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
  27 |  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
  28 |  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
  29 |  *
  30 |  **************************************************************************************************/
  31 | /*! \file
  32 |     \brief Template for GEMM performing a reduction over K partitions in parallel.
  33 | */
```
**EN:** This opening comment records the license, copyright ownership, and redistribution conditions that apply to the header.
**CN:** 开头注释说明了该头文件适用的许可证、版权归属以及再分发条件。

### Lines 35-35

```cpp
  35 | #pragma once
```
**EN:** This directive makes the header idempotent so repeated inclusion does not create duplicate definitions.
**CN:** 该指令让头文件只生效一次，避免重复包含导致的重复定义。

### Lines 37-37

```cpp
  37 | #include "cutlass/cutlass.h"
```
**EN:** This include block imports `cutlass/cutlass.h`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/cutlass.h`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 39-40

```cpp
  39 | #include "cutlass/gemm/gemm.h"
  40 | #include "cutlass/matrix_coord.h"
```
**EN:** This include block imports `cutlass/gemm/gemm.h`, `cutlass/matrix_coord.h`, providing the types, architecture tags, tensor utilities, and helper kernels used later in the file.
**CN:** 这一组 include 引入了 `cutlass/gemm/gemm.h`, `cutlass/matrix_coord.h`，为后续代码提供类型、架构标签、张量工具以及辅助内核。

### Lines 44-46

```cpp
  44 | namespace cutlass {
  45 | namespace gemm {
  46 | namespace kernel {
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

### Lines 50-55

```cpp
  50 | template <
  51 |   typename Mma_,                  ///! Threadblock-scoped matrix multiply-accumulate 
  52 |   typename Epilogue_,             ///! Epilogue
  53 |   typename ThreadblockSwizzle_    ///! Threadblock swizzling function
  54 | >
  55 | struct GemmSplitKParallel {
```
**EN:** This block declares or specializes `GemmSplitKParallel`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `GemmSplitKParallel`，它是该头文件中承载某一层内核策略的核心结构体。

### Lines 57-60

```cpp
  57 |   using Mma = Mma_;
  58 |   using Epilogue = Epilogue_;
  59 |   using OutputOp = typename Epilogue::OutputOp;
  60 |   using ThreadblockSwizzle = ThreadblockSwizzle_;
```
**EN:** This alias block derives concise type names `Mma`, `Epilogue`, `OutputOp`, `ThreadblockSwizzle` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `Mma`, `Epilogue`, `OutputOp`, `ThreadblockSwizzle` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 62-64

```cpp
  62 |   /// Warp count (concept: GemmShape)
  63 |   using WarpCount = typename Mma::WarpCount;
  64 |   static int const kThreadCount = 32 * WarpCount::kCount;
```
**EN:** This alias block derives concise type names `WarpCount` and related compile-time constants, making the remaining kernel description easier to assemble and read.
**CN:** 这一段别名定义导出了简洁的类型名 `WarpCount` 及相关编译期常量，使后续内核描述更容易装配和阅读。

### Lines 66-66

```cpp
  66 |   static int const kAlignmentK = Mma::Operator::Shape::kK;
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 68-85

```cpp
  68 |   /// Parameters structure
  69 |   struct Params {
  70 |     cutlass::gemm::GemmCoord problem_size;
  71 |     cutlass::gemm::GemmCoord grid_tiled_shape;
  72 |     int swizzle_log_tile;
  73 |     typename Mma::IteratorA::Params params_A;
  74 |     typename Mma::IteratorA::TensorRef ref_A;
  75 |     typename Mma::IteratorB::Params params_B;
  76 |     typename Mma::IteratorB::TensorRef ref_B;
  77 |     typename Epilogue::OutputTileIterator::Params params_D;
  78 |     typename Epilogue::OutputTileIterator::TensorRef ref_D;
  79 |     typename OutputOp::Params output_op;
  80 |     int64_t splitk_slice_stride;
  81 |     int gemm_k_size;
  82 | 
  83 |     //
  84 |     // Methods
  85 |     //
```
**EN:** This `Params` structure stores the packed, device-ready launch parameters that the kernel reads after host-side preprocessing.
**CN:** 这个 `Params` 结构体保存经过主机侧预处理后的、可直接被设备端读取的启动参数。

### Lines 87-88

```cpp
  87 |     CUTLASS_HOST_DEVICE
  88 |     Params(): swizzle_log_tile(0) { }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 90-110

```cpp
  90 |     CUTLASS_HOST_DEVICE
  91 |     Params(
  92 |       cutlass::gemm::GemmCoord const & problem_size,
  93 |       cutlass::gemm::GemmCoord const & grid_tiled_shape,
  94 |       typename Mma::IteratorA::TensorRef ref_A,
  95 |       typename Mma::IteratorB::TensorRef ref_B,
  96 |       typename Epilogue::OutputTileIterator::TensorRef ref_D,
  97 |       typename OutputOp::Params output_op,
  98 |       int64_t splitk_slice_stride
  99 |     ):
 100 |       problem_size(problem_size),
 101 |       grid_tiled_shape(grid_tiled_shape),
 102 |       swizzle_log_tile(ThreadblockSwizzle().get_log_tile(grid_tiled_shape)),
 103 |       params_A(ref_A.layout()),
 104 |       ref_A(ref_A),
 105 |       params_B(ref_B.layout()),
 106 |       ref_B(ref_B),
 107 |       params_D(ref_D.layout()),
 108 |       ref_D(ref_D),
 109 |       output_op(output_op),
 110 |       splitk_slice_stride(splitk_slice_stride) {
```
**EN:** This block continues the matrix-multiply engine, epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎、epilogue 输出路径相关逻辑。

### Lines 112-113

```cpp
 112 |       int full_gemm_k_iterations = problem_size.k() / Mma::Shape::kK;
 113 |       int gemm_k_iterations = full_gemm_k_iterations / grid_tiled_shape.k();
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 115-117

```cpp
 115 |       gemm_k_size = gemm_k_iterations * Mma::Shape::kK;
 116 |     }
 117 |   };
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 119-127

```cpp
 119 |   /// Shared memory storage structure
 120 |   union SharedStorage {
 121 |     typename Mma::SharedStorage main_loop;
 122 |     typename Epilogue::SharedStorage epilogue;
 123 |   };
 124 | 
 125 |   //
 126 |   // Methods
 127 |   //
```
**EN:** This block continues the matrix-multiply engine, epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎、epilogue 输出路径相关逻辑。

### Lines 129-130

```cpp
 129 |   CUTLASS_HOST_DEVICE
 130 |   GemmSplitKParallel() { } 
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 132-134

```cpp
 132 |   /// Executes one GEMM
 133 |   CUTLASS_DEVICE
 134 |   void operator()(Params const &params, SharedStorage &shared_storage) {
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 136-137

```cpp
 136 |     // Compute threadblock location
 137 |     ThreadblockSwizzle threadblock_swizzle;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 139-140

```cpp
 139 |     cutlass::gemm::GemmCoord threadblock_tile_offset =
 140 |         threadblock_swizzle.get_tile_offset(params.swizzle_log_tile);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 142-144

```cpp
 142 |     // Early exit if CTA is out of range
 143 |     if (params.grid_tiled_shape.m() <= threadblock_tile_offset.m() ||
 144 |       params.grid_tiled_shape.n() <= threadblock_tile_offset.n()) {
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 146-147

```cpp
 146 |       return;
 147 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 149-153

```cpp
 149 |     // Compute initial location in logical coordinates
 150 |     cutlass::MatrixCoord tb_offset_A{
 151 |       threadblock_tile_offset.m() * Mma::Shape::kM,
 152 |       threadblock_tile_offset.k() * params.gemm_k_size,
 153 |     };
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 155-158

```cpp
 155 |     cutlass::MatrixCoord tb_offset_B{
 156 |       threadblock_tile_offset.k() * params.gemm_k_size,
 157 |       threadblock_tile_offset.n() * Mma::Shape::kN
 158 |     };
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 160-167

```cpp
 160 |     // Problem size is a function of threadblock index in the K dimension
 161 |     int problem_size_k;
 162 |     if (threadblock_tile_offset.k() + 1 == params.grid_tiled_shape.k()) {
 163 |       problem_size_k = params.problem_size.k();
 164 |     }
 165 |     else {
 166 |       problem_size_k = (threadblock_tile_offset.k() + 1) * params.gemm_k_size;
 167 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 169-170

```cpp
 169 |     // Compute threadblock-scoped matrix multiply-add
 170 |     int gemm_k_iterations = (problem_size_k - tb_offset_A.column() + Mma::Shape::kK - 1) / Mma::Shape::kK;
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 172-173

```cpp
 172 |     // Compute position within threadblock
 173 |     int thread_idx = threadIdx.x;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 175-181

```cpp
 175 |     // Construct iterators to A and B operands
 176 |     typename Mma::IteratorA iterator_A(
 177 |       params.params_A,
 178 |       params.ref_A.data(),
 179 |       {params.problem_size.m(), problem_size_k},
 180 |       thread_idx,
 181 |       tb_offset_A);
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 183-188

```cpp
 183 |     typename Mma::IteratorB iterator_B(
 184 |       params.params_B,
 185 |       params.ref_B.data(),
 186 |       {problem_size_k, params.problem_size.n()},
 187 |       thread_idx,
 188 |       tb_offset_B);
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 190-196

```cpp
 190 |     int warp_idx = threadIdx.x / 32;
 191 |     int lane_idx = threadIdx.x % 32;
 192 | 
 193 |     //
 194 |     // Main loop
 195 |     //
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 198-199

```cpp
 198 |     // Construct thread-scoped matrix multiply
 199 |     Mma mma(shared_storage.main_loop, thread_idx, warp_idx, lane_idx);
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 201-201

```cpp
 201 |     typename Mma::FragmentC accumulators;
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 203-203

```cpp
 203 |     accumulators.clear();
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 205-209

```cpp
 205 |     mma(gemm_k_iterations, accumulators, iterator_A, iterator_B, accumulators);
 206 | 
 207 |     //
 208 |     // Epilogue
 209 |     //
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 211-215

```cpp
 211 |     OutputOp output_op(params.output_op);
 212 | 
 213 |     //
 214 |     // Masked tile iterators constructed from members
 215 |     //
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 217-218

```cpp
 217 |     threadblock_tile_offset =
 218 |         threadblock_swizzle.get_tile_offset(params.swizzle_log_tile);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 220-224

```cpp
 220 |     //assume identity swizzle
 221 |     MatrixCoord threadblock_offset(
 222 |       threadblock_tile_offset.m() * Mma::Shape::kM,
 223 |       threadblock_tile_offset.n() * Mma::Shape::kN
 224 |     );
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 226-233

```cpp
 226 |     // Tile iterator writing to output tile
 227 |     typename Epilogue::OutputTileIterator iterator_D(
 228 |       params.params_D,
 229 |       params.ref_D.data(),
 230 |       params.problem_size.mn(),
 231 |       thread_idx,
 232 |       threadblock_offset
 233 |     );
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 235-235

```cpp
 235 |     iterator_D.add_pointer_offset(params.splitk_slice_stride * threadblock_tile_offset.k());
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 237-242

```cpp
 237 |     // Execute the epilogue
 238 |     Epilogue epilogue(
 239 |       shared_storage.epilogue, 
 240 |       thread_idx, 
 241 |       warp_idx, 
 242 |       lane_idx);
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 244-247

```cpp
 244 |     // Run efficient epilogue
 245 |     epilogue(output_op, iterator_D, accumulators, iterator_D);
 246 |   }
 247 | };
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 251-253

```cpp
 251 | } // namespace kernel
 252 | } // namespace gemm
 253 | } // namespace cutlass
```
**EN:** These lines open or close the nested CUTLASS namespaces so the declarations stay scoped to the GEMM kernel subsystem.
**CN:** 这些代码行用于打开或关闭 CUTLASS 的嵌套命名空间，使声明被限制在 GEMM 内核子系统内。

## Key Concepts / 关键概念
- Template metaprogramming / 模板元编程
- Packed launch parameters / 打包后的启动参数
- Shared memory layout / 共享内存布局

## Dependencies / 依赖关系
- **Included headers / 引入头文件:** `cutlass/cutlass.h`, `cutlass/gemm/gemm.h`, `cutlass/matrix_coord.h`
- **Subsystems / 子系统:** Epilogue/output pipeline / Epilogue 输出流水, Threadblock MMA building blocks / 线程块级 MMA 构件
- **Peer kernel headers / 同级内核头文件:** None directly detected / 未检测到直接依赖
