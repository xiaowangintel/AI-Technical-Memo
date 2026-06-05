# gemm_batched.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件:** `include/cutlass/gemm/kernel/gemm_batched.h`
- **Purpose / 用途 (EN):** Implements kernel-side support for GEMM batched. Briefly, the file comment says: Template for a pipelined GEMM kernel. Does not compute batching or support split-K.
- **Purpose / 用途 (CN):** 实现 GEMM batched 的内核侧支持逻辑。 文件注释的简要说明是：Template for a pipelined GEMM kernel. Does not compute batching or support split-K。
- **Line count / 行数:** 273

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
  32 |     \brief Template for a pipelined GEMM kernel. Does not compute batching or support split-K.
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
  55 | struct GemmBatched {
```
**EN:** This block declares or specializes `GemmBatched`, the main struct that packages one layer of the kernel policy defined in this header.
**CN:** 这一段声明或特化了 `GemmBatched`，它是该头文件中承载某一层内核策略的核心结构体。

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

### Lines 66-85

```cpp
  66 |   /// Parameters structure
  67 |   struct Params {
  68 |     cutlass::gemm::GemmCoord problem_size{};
  69 |     cutlass::gemm::GemmCoord grid_tiled_shape{};
  70 |     int swizzle_log_tile{0};
  71 |     typename Mma::IteratorA::Params params_A{};
  72 |     typename Mma::IteratorA::TensorRef ref_A{};
  73 |     int64_t stride_A{0};
  74 |     typename Mma::IteratorB::Params params_B{};
  75 |     typename Mma::IteratorB::TensorRef ref_B{};
  76 |     int64_t stride_B{0};
  77 |     typename Epilogue::OutputTileIterator::Params params_C{};
  78 |     typename Epilogue::OutputTileIterator::TensorRef ref_C{};
  79 |     int64_t stride_C{0};
  80 |     typename Epilogue::OutputTileIterator::Params params_D{};
  81 |     typename Epilogue::OutputTileIterator::TensorRef ref_D{};
  82 |     int64_t stride_D{0};
  83 |     typename OutputOp::Params epilogue{};
  84 |     int batch_count{1};
  85 |     int gemm_k_iterations{0};
```
**EN:** This `Params` structure stores the packed, device-ready launch parameters that the kernel reads after host-side preprocessing.
**CN:** 这个 `Params` 结构体保存经过主机侧预处理后的、可直接被设备端读取的启动参数。

### Lines 87-90

```cpp
  87 |     //
  88 |     // Methods
  89 |     //
  90 |     Params() = default;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 92-111

```cpp
  92 |     CUTLASS_HOST_DEVICE
  93 |     Params(
  94 |       cutlass::gemm::GemmCoord const & problem_size_,
  95 |       cutlass::gemm::GemmCoord const & grid_tiled_shape_,
  96 |       typename Mma::IteratorA::TensorRef ref_A_,
  97 |       int64_t stride_A_,
  98 |       typename Mma::IteratorB::TensorRef ref_B_,
  99 |       int64_t stride_B_,
 100 |       typename Epilogue::OutputTileIterator::TensorRef ref_C_,
 101 |       int64_t stride_C_,
 102 |       typename Epilogue::OutputTileIterator::TensorRef ref_D_,
 103 |       int64_t stride_D_,
 104 |       typename OutputOp::Params epilogue_,
 105 |       int batch_count_
 106 |     ):
 107 |       problem_size(problem_size_),
 108 |       grid_tiled_shape(grid_tiled_shape_),
 109 |       swizzle_log_tile(ThreadblockSwizzle().get_log_tile(grid_tiled_shape)),
 110 |       params_A(ref_A_.layout()),
 111 |       ref_A(ref_A_),
```
**EN:** This block continues the matrix-multiply engine, epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎、epilogue 输出路径相关逻辑。

### Lines 112-125

```cpp
 112 |       stride_A(stride_A_),
 113 |       params_B(ref_B_.layout()),
 114 |       ref_B(ref_B_),
 115 |       stride_B(stride_B_),
 116 |       params_C(ref_C_.layout()),
 117 |       ref_C(ref_C_),
 118 |       stride_C(stride_C_),
 119 |       params_D(ref_D_.layout()),
 120 |       ref_D(ref_D_),
 121 |       stride_D(stride_D_),
 122 |       epilogue(epilogue_),
 123 |       batch_count(batch_count_),
 124 |       gemm_k_iterations((problem_size.k() + Mma::Shape::kK - 1) / Mma::Shape::kK) {}
 125 |   };
```
**EN:** This block continues the matrix-multiply engine, epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎、epilogue 输出路径相关逻辑。

### Lines 127-131

```cpp
 127 |   /// Shared memory storage structure
 128 |   union SharedStorage {
 129 |     typename Mma::SharedStorage main_loop;
 130 |     typename Epilogue::SharedStorage epilogue;
 131 |   };
```
**EN:** This block continues the matrix-multiply engine, epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎、epilogue 输出路径相关逻辑。

### Lines 133-136

```cpp
 133 |   //
 134 |   // Methods
 135 |   //
 136 |   GemmBatched() = default;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 138-140

```cpp
 138 |   /// Executes one GEMM
 139 |   CUTLASS_DEVICE
 140 |   void operator()(Params const &params, SharedStorage &shared_storage) {
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 142-143

```cpp
 142 |     // Compute threadblock location
 143 |     ThreadblockSwizzle threadblock_swizzle;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 145-146

```cpp
 145 |     cutlass::gemm::GemmCoord threadblock_tile_offset =
 146 |         threadblock_swizzle.get_tile_offset(params.swizzle_log_tile);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 148-150

```cpp
 148 |     // Early exit if CTA is out of range
 149 |     if (params.grid_tiled_shape.m() <= threadblock_tile_offset.m() ||
 150 |       params.grid_tiled_shape.n() <= threadblock_tile_offset.n()) {
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 152-153

```cpp
 152 |       return;
 153 |     }
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 156-159

```cpp
 156 |     // Each CTA handles multiple batch indices to accommodate limited range of CUDA grid's Z dimension
 157 |     for (int batch_idx = threadblock_swizzle.get_batch_idx(); 
 158 |       batch_idx < params.batch_count; 
 159 |       batch_idx += gridDim.z) {
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 161-165

```cpp
 161 |       // Compute initial location in logical coordinates
 162 |       cutlass::MatrixCoord tb_offset_A{
 163 |         threadblock_tile_offset.m() * Mma::Shape::kM,
 164 |         0
 165 |       };
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 167-170

```cpp
 167 |       cutlass::MatrixCoord tb_offset_B{
 168 |         0,
 169 |         threadblock_tile_offset.n() * Mma::Shape::kN
 170 |       };
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 172-173

```cpp
 172 |       // Compute position within threadblock
 173 |       int thread_idx = threadIdx.x;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 175-181

```cpp
 175 |       // Construct iterators to A and B operands
 176 |       typename Mma::IteratorA iterator_A(
 177 |         params.params_A,
 178 |         params.ref_A.data(),
 179 |         params.problem_size.mk(),
 180 |         thread_idx,
 181 |         tb_offset_A);
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 183-183

```cpp
 183 |       iterator_A.add_pointer_offset(params.stride_A * batch_idx);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 185-190

```cpp
 185 |       typename Mma::IteratorB iterator_B(
 186 |         params.params_B,
 187 |         params.ref_B.data(),
 188 |         params.problem_size.kn(),
 189 |         thread_idx,
 190 |         tb_offset_B);
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 192-197

```cpp
 192 |       iterator_B.add_pointer_offset(params.stride_B * batch_idx);
 193 | 
 194 |       //
 195 |       // Main loop
 196 |       //
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 199-201

```cpp
 199 |       // Broadcast the warp_id computed by lane 0 to ensure dependent code
 200 |       // is compiled as warp-uniform.
 201 |       int warp_idx = canonical_warp_idx_sync();
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 203-203

```cpp
 203 |       int lane_idx = threadIdx.x % 32;
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 205-205

```cpp
 205 |       Mma mma(shared_storage.main_loop, thread_idx, warp_idx, lane_idx);
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 207-207

```cpp
 207 |       typename Mma::FragmentC accumulators;
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 209-209

```cpp
 209 |       accumulators.clear();
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 212-217

```cpp
 212 |       // Compute threadblock-scoped matrix multiply-add
 213 |       mma(params.gemm_k_iterations, accumulators, iterator_A, iterator_B, accumulators);
 214 | 
 215 |       //
 216 |       // Epilogue
 217 |       //
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 219-223

```cpp
 219 |       OutputOp output_op(params.epilogue);
 220 | 
 221 |       //
 222 |       // Masked tile iterators constructed from members
 223 |       //
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 225-226

```cpp
 225 |       threadblock_tile_offset =
 226 |           threadblock_swizzle.get_tile_offset(params.swizzle_log_tile);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 228-232

```cpp
 228 |       //assume identity swizzle
 229 |       MatrixCoord threadblock_offset(
 230 |         threadblock_tile_offset.m() * Mma::Shape::kM,
 231 |         threadblock_tile_offset.n() * Mma::Shape::kN
 232 |       );
```
**EN:** This block continues the matrix-multiply engine defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的矩阵乘执行引擎相关逻辑。

### Lines 234-241

```cpp
 234 |       // Tile iterator writing to output tile
 235 |       typename Epilogue::OutputTileIterator iterator_C(
 236 |         params.params_C,
 237 |         params.ref_C.data(),
 238 |         params.problem_size.mn(),
 239 |         thread_idx,
 240 |         threadblock_offset
 241 |       );
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 243-243

```cpp
 243 |       iterator_C.add_pointer_offset(params.stride_C * batch_idx);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 245-252

```cpp
 245 |       // Tile iterator writing to output tile
 246 |       typename Epilogue::OutputTileIterator iterator_D(
 247 |         params.params_D,
 248 |         params.ref_D.data(),
 249 |         params.problem_size.mn(),
 250 |         thread_idx,
 251 |         threadblock_offset
 252 |       );
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 254-254

```cpp
 254 |       iterator_D.add_pointer_offset(params.stride_D * batch_idx);
```
**EN:** This block continues wiring together the template types, policies, and helper routines that define the current kernel path.
**CN:** 这一段继续装配当前内核路径所需的模板类型、策略对象和辅助例程。

### Lines 256-260

```cpp
 256 |       Epilogue epilogue(
 257 |         shared_storage.epilogue, 
 258 |         thread_idx, 
 259 |         warp_idx, 
 260 |         lane_idx);
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 262-266

```cpp
 262 |       // run efficient epilogue
 263 |       epilogue(output_op, iterator_D, accumulators, iterator_C);
 264 |     }
 265 |   }
 266 | };
```
**EN:** This block continues the epilogue/output path defined by the surrounding kernel template.
**CN:** 这一段继续实现周围内核模板中的epilogue 输出路径相关逻辑。

### Lines 270-272

```cpp
 270 | } // namespace kernel
 271 | } // namespace gemm
 272 | } // namespace cutlass
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
