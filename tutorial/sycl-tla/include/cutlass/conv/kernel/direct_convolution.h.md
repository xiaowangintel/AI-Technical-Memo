# direct_convolution.h — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cutlass/conv/kernel/direct_convolution.h`
- **Purpose (EN):** Template for a multi-staged Depthwise Convolution kernel.
- **用途 (CN):** 组合 直接 卷积 对应的内核级卷积逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
```cpp
  1: /***************************************************************************************************
  2:  * Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
  3:  * SPDX-License-Identifier: BSD-3-Clause
  4:  *
  5:  * Redistribution and use in source and binary forms, with or without
  6:  * modification, are permitted provided that the following conditions are met:
  7:  *
  8:  * 1. Redistributions of source code must retain the above copyright notice, this
  9:  * list of conditions and the following disclaimer.
 10:  *
 11:  * 2. Redistributions in binary form must reproduce the above copyright notice,
 12:  * this list of conditions and the following disclaimer in the documentation
 13:  * and/or other materials provided with the distribution.
 14:  *
 15:  * 3. Neither the name of the copyright holder nor the names of its
 16:  * contributors may be used to endorse or promote products derived from
 17:  * this software without specific prior written permission.
 18:  *
 19:  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 20:  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 21:  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 22:  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 23:  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 24:  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 25:  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 26:  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 27:  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 28:  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 29:  *
 30:  **************************************************************************************************/
```
**EN:** Records the BSD-3-Clause license and redistribution conditions for the file.

**CN:** 记录该文件的 BSD-3-Clause 许可和再分发条件。

### Lines 31-33
```cpp
 31: /*! \file
 32:     \brief Template for a multi-staged Depthwise Convolution kernel.
 33: */
```
**EN:** Documents the file's purpose, terminology, and high-level usage.

**CN:** 说明文件的用途、术语以及高层使用方式。

### Lines 35-35
```cpp
 35: #pragma once
```
**EN:** Marks this file as a single-inclusion header with `#pragma once`.

**CN:** 通过 `#pragma once` 把该文件标记为单次包含头文件。

### Lines 37-37
```cpp
 37: #include "cutlass/cutlass.h"
```
**EN:** Imports direct dependencies used later in the file, including `cutlass.h`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `cutlass.h`。

### Lines 39-51
```cpp
 39: #include "cutlass/aligned_buffer.h"
 40: #include "cutlass/array.h"
 41: #include "cutlass/numeric_types.h"
 42: #include "cutlass/matrix_shape.h"
 43: #include "cutlass/platform/platform.h"
 44: #include "cutlass/semaphore.h"
 45: #include "cutlass/tensor_ref.h"
 46: #include "cutlass/layout/tensor.h"
 47: #include "cutlass/gemm/gemm.h"
 48: #include "cutlass/conv/convolution.h"
 49: #include "cutlass/conv/conv2d_problem_size.h"
 50: #include "cutlass/conv/conv3d_problem_size.h"
 51: #include "cutlass/epilogue/threadblock/output_iterator_parameter.h"
```
**EN:** Imports direct dependencies used later in the file, including `aligned_buffer.h`, `array.h`, `numeric_types.h`, `matrix_shape.h`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `aligned_buffer.h`, `array.h`, `numeric_types.h`, `matrix_shape.h`。

### Lines 55-57
```cpp
 55: namespace cutlass {
 56: namespace conv {
 57: namespace kernel {
```
**EN:** Opens the namespace scope `cutlass` that contains these definitions.

**CN:** 打开包含这些定义的命名空间作用域 `cutlass`。

### Lines 61-80
```cpp
 61: /// Parameters structure
 62: template <typename Mma_,                 ///! Threadblock-scoped matrix multiply-accumulate
 63:           typename Epilogue_,            ///! Epilogue
 64:           typename ThreadblockSwizzle_,  ///! Threadblock swizzling function
 65:           conv::Operator ConvOperator,   ///! Convolutional operator (Fprop, Dgrad, Wgrad)
 66:           typename Arguments_,           ///! Kernel Arguments
 67:           typename ConvOutputIteratorParameter_, ///! Output Iterator Params
 68:           typename ConvProblemSize_ = Conv2dProblemSize,  ///! Convolutional operator on 2D or 3D problem
 69:           conv::GroupMode GroupMode_ = conv::GroupMode::kNone,  ///! Group mode
 70:           typename ThreadBlockOutputShape_ = cutlass::conv::TensorNHWCShape<1, 1, 1, 1> >  ///! OutputShape per ThreadBlock
 71: struct DirectConvolutionParams {
 72:   using Mma = Mma_;
 73:   using Epilogue = Epilogue_;
 74:   using EpilogueOutputOp = typename Epilogue::OutputOp;
 75:   using ThreadblockSwizzle = ThreadblockSwizzle_;
 76:   using ThreadBlockOutputShape = ThreadBlockOutputShape_;
 77:   static Operator const kConvolutionalOperator = ConvOperator;
 78:   using ConvProblemSize = ConvProblemSize_;
 79:   using Arguments = Arguments_;
 80:   using ConvOutputIteratorParameter = ConvOutputIteratorParameter_;
```
**EN:** Declares struct `DirectConvolutionParams`. The nearby comment explains that it serves the surrounding direct convolution parameters logic.

**CN:** 声明结构体 `DirectConvolutionParams`，相邻注释说明它服务于周围的 直接 卷积 参数 逻辑。

### Lines 82-85
```cpp
 82:   using ThreadblockShape = typename Mma::Shape;
 83:   static IteratorAlgorithm const kIteratorAlgorithm = Mma::IteratorA::kIteratorAlgorithm;
 84:   static conv::GroupMode const kGroupMode = GroupMode_;
 85:   static int const kStages = Mma::kStages;
```
**EN:** Introduces aliases such as `ThreadblockShape` to keep the surrounding template code readable.

**CN:** 引入 `ThreadblockShape` 等别名，以提升周围模板代码的可读性。

### Lines 87-91
```cpp
 87:   ConvProblemSize problem_size;
 88:   cutlass::gemm::GemmCoord grid_tiled_shape;
 89:   gemm::GemmCoord implicit_gemm_problem_size;
 90:   int swizzle_log_tile;
 91:   int smem_size_;
```
**EN:** Stores member state such as `problem_size`, `grid_tiled_shape`, `implicit_gemm_problem_size`, `swizzle_log_tile`, `smem_size_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `problem_size`, `grid_tiled_shape`, `implicit_gemm_problem_size`, `swizzle_log_tile`, `smem_size_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 93-107
```cpp
 93:   int gemm_k_iterations;
 94:   int gemm_k_iterations_per_channel;
 95:   typename Mma::IteratorA::Params iterator_A;
 96:   typename Mma::IteratorA::Element const *ptr_A;
 97:   typename Mma::IteratorB::Params iterator_B;
 98:   typename Mma::IteratorB::Element const *ptr_B;
 99:   typename Mma::IteratorB::Element *ptr_reordered_B;
100:   typename Epilogue::OutputTileIterator::Params iterator_C;
101:   typename Epilogue::OutputTileIterator::Element *ptr_C;
102:   typename Epilogue::OutputTileIterator::Params iterator_D;
103:   typename Epilogue::OutputTileIterator::Element *ptr_D;
104:   typename EpilogueOutputOp::Params output_op;
105:   int *semaphore;
106:   SplitKMode split_k_mode;
107:   int split_k_slices;
```
**EN:** Stores member state such as `gemm_k_iterations`, `gemm_k_iterations_per_channel`, `iterator_A`, `ptr_A`, `iterator_B`, `ptr_B` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `gemm_k_iterations`, `gemm_k_iterations_per_channel`, `iterator_A`, `ptr_A`, `iterator_B`, `ptr_B` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 113-114
```cpp
113:   CUTLASS_HOST_DEVICE
114:   DirectConvolutionParams() : swizzle_log_tile(0), gemm_k_iterations(0) {}
```
**EN:** Provides constructor-style initialization for `DirectConvolutionParams`.

**CN:** 为 `DirectConvolutionParams` 提供构造式初始化逻辑。

### Lines 117-143
```cpp
117:   CUTLASS_HOST_DEVICE
118:   DirectConvolutionParams(Arguments const &args, int *semaphore = nullptr)
119:       : problem_size(args.problem_size),
120:         implicit_gemm_problem_size(
121:             cutlass::conv::implicit_gemm_problem_size(kConvolutionalOperator, args.problem_size)),
122:         iterator_A(Mma::IteratorA::getParams(args.problem_size, args.ref_A.layout())),
123:         ptr_A(args.ref_A.data()),
124:         iterator_B(Mma::IteratorB::getParams(args.problem_size, args.ref_B.layout())),
125:         ptr_B(args.ref_B.data()),
126:         ptr_reordered_B(args.ref_reordered_B.data()),
127:         iterator_C(ConvOutputIteratorParameter::layout(args.ref_C), args.problem_size),
128:         ptr_C(args.ref_C.data()),
129:         iterator_D(ConvOutputIteratorParameter::layout(args.ref_D), args.problem_size),
130:         ptr_D(args.ref_D.data()),
131:         output_op(args.output_op),
132:         semaphore(semaphore),
133:         split_k_mode(args.split_k_mode),
134:         split_k_slices(args.problem_size.split_k_slices) {
135:     gemm_k_iterations =
136:         depthwise_gemm_k_iterations<ThreadBlockOutputShape::kN,
137:                                     ThreadBlockOutputShape::kH,
138:                                     ThreadBlockOutputShape::kW>(kConvolutionalOperator,
139:                                                                 ThreadblockShape::kK,
140:                                                                 args.problem_size,
141:                                                                 kIteratorAlgorithm,
142:                                                                 kGroupMode,
143:                                                                 ThreadblockShape::kN);
```
**EN:** Provides constructor-style initialization for `DirectConvolutionParams`.

**CN:** 为 `DirectConvolutionParams` 提供构造式初始化逻辑。

### Lines 145-146
```cpp
145:     gemm_k_iterations_per_channel = implicit_gemm_k_iterations_per_channel(
146:         kConvolutionalOperator, args.problem_size, kIteratorAlgorithm);
```
**EN:** Defines function `implicit_gemm_k_iterations_per_channel` for this stage of the convolution workflow.

**CN:** 定义函数 `implicit_gemm_k_iterations_per_channel`，服务于卷积工作流的这一阶段。

### Lines 148-148
```cpp
148:     ThreadblockSwizzle threadblock_swizzle;
```
**EN:** Stores member state such as `threadblock_swizzle` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `threadblock_swizzle` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 150-154
```cpp
150:     grid_tiled_shape = threadblock_swizzle.get_tiled_shape(
151:         kConvolutionalOperator,
152:         problem_size,
153:         {ThreadblockShape::kM, ThreadblockShape::kN, ThreadblockShape::kK},
154:         args.problem_size.split_k_slices);
```
**EN:** Stores member state such as `kConvolutionalOperator`, `problem_size`, `kM` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `kConvolutionalOperator`, `problem_size`, `kM` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 156-156
```cpp
156:     swizzle_log_tile = threadblock_swizzle.get_log_tile(grid_tiled_shape);
```
**EN:** Defines function `get_log_tile` for this stage of the convolution workflow.

**CN:** 定义函数 `get_log_tile`，服务于卷积工作流的这一阶段。

### Lines 158-160
```cpp
158:     // Dynamic SMEM usage because stride and dilation are runtime params.
159:     smem_size_ = (cutlass::platform::max(iterator_A.activation_size, int(sizeof(typename Epilogue::SharedStorage))) * kStages + iterator_B.filter_size);
160:   }
```
**EN:** Defines function `max` for this stage of the convolution workflow.

**CN:** 定义函数 `max`，服务于卷积工作流的这一阶段。

### Lines 162-163
```cpp
162:   CUTLASS_HOST_DEVICE
163:   int get_smem_size() {
```
**EN:** Defines function `get_smem_size` for this stage of the convolution workflow.

**CN:** 定义函数 `get_smem_size`，服务于卷积工作流的这一阶段。

### Lines 164-167
```cpp
164:     // Dynamic Smem Size
165:     return smem_size_;
166:   }
167: };
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 170-173
```cpp
170: template <typename Params_, typename ElementB_>
171: struct ReorderKernel {
172:   using Params = Params_;
173:   using ElementB = ElementB_;
```
**EN:** Declares struct `ReorderKernel`, a reorder kernel component in the convolution stack.

**CN:** 声明结构体 `ReorderKernel`，它是卷积栈中的 reorder 内核 组件。

### Lines 175-175
```cpp
175:   union SharedStorage {};
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 177-177
```cpp
177:   static unsigned int const kReorderKernelThreadPerCTA = 128;
```
**EN:** Stores member state such as `kReorderKernelThreadPerCTA` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `kReorderKernelThreadPerCTA` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 179-180
```cpp
179:   CUTLASS_HOST_DEVICE
180:   ReorderKernel() {}
```
**EN:** Provides constructor-style initialization for `ReorderKernel`.

**CN:** 为 `ReorderKernel` 提供构造式初始化逻辑。

### Lines 182-189
```cpp
182:   CUTLASS_HOST_DEVICE
183:   static dim3 get_grid_shape(Params const &params) {
184:     return dim3{static_cast<unsigned int>(
185:                     (params.problem_size.filter_size() + kReorderKernelThreadPerCTA - 1) /
186:                     kReorderKernelThreadPerCTA),
187:                 1,
188:                 1};
189:   }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 191-192
```cpp
191:   CUTLASS_HOST_DEVICE
192:   static dim3 get_block_shape() { return dim3{kReorderKernelThreadPerCTA, 1, 1}; }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 194-199
```cpp
194:   CUTLASS_HOST_DEVICE
195:   void operator()(Params const &params, SharedStorage &shared_storage) {
196:     int64_t m = static_cast<int64_t>(params.problem_size.groups);
197:     int64_t n = static_cast<int64_t>(params.problem_size.filter_size() / params.problem_size.K);
198:     const ElementB *src_with_type = static_cast<const ElementB *>(params.ptr_B);
199:     ElementB *dst_with_type = static_cast<ElementB *>(params.ptr_reordered_B);
```
**EN:** Implements the callable operator that performs the block's main action.

**CN:** 实现可调用运算符，执行该代码块的主要动作。

### Lines 201-204
```cpp
201:     int64_t linear_index = blockIdx.x * kReorderKernelThreadPerCTA + threadIdx.x;
202:     int64_t index_m = linear_index / n;
203:     int64_t index_n = linear_index % n;
204:     int64_t new_linear_index = index_m + index_n * m;
```
**EN:** Stores member state such as `linear_index`, `index_m`, `index_n`, `new_linear_index` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `linear_index`, `index_m`, `index_n`, `new_linear_index` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 206-211
```cpp
206:     if (linear_index < m * n) {
207:       dst_with_type[new_linear_index] = src_with_type[linear_index];
208:     }
209:     return;
210:   }
211: };
```
**EN:** Stores member state such as `dst_with_type`, `return` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `dst_with_type`, `return` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 215-224
```cpp
215: template <
216:   typename Mma_,                                  ///! Threadblock-scoped matrix multiply-accumulate 
217:   typename Epilogue_,                             ///! Epilogue
218:   typename ThreadblockSwizzle_,                   ///! Threadblock swizzling function
219:   conv::Operator ConvOperator,                    ///! Convolutional operator (Fprop, Dgrad, Wgrad)
220:   typename ConvProblemSize_ = Conv2dProblemSize,  ///! Convolutional operator on 2D or 3D problem
221:   conv::GroupMode GroupMode_ = conv::GroupMode::kNone,    ///! Group mode
222:   typename ThreadBlockOutputShape_ = cutlass::conv::TensorNHWCShape<1, 1, 1, 1>
223: >
224: struct DirectConvolution {
```
**EN:** Declares struct `DirectConvolution`, a direct convolution component in the convolution stack.

**CN:** 声明结构体 `DirectConvolution`，它是卷积栈中的 直接 卷积 组件。

### Lines 226-231
```cpp
226:   using Mma = Mma_;
227:   using Epilogue = Epilogue_;
228:   using EpilogueOutputOp = typename Epilogue::OutputOp;
229:   using ThreadblockSwizzle = ThreadblockSwizzle_;
230:   using ThreadBlockOutputShape = ThreadBlockOutputShape_;
231:   static Operator const kConvolutionalOperator = ConvOperator;
```
**EN:** Introduces aliases such as `Mma`, `Epilogue`, `EpilogueOutputOp`, `ThreadblockSwizzle`, `ThreadBlockOutputShape` to keep the surrounding template code readable.

**CN:** 引入 `Mma`, `Epilogue`, `EpilogueOutputOp`, `ThreadblockSwizzle`, `ThreadBlockOutputShape` 等别名，以提升周围模板代码的可读性。

### Lines 233-237
```cpp
233:   using ElementA = typename Mma::IteratorA::Element;
234:   using LayoutA = typename Mma::IteratorA::Layout;
235:   using ElementB = typename Mma::IteratorB::Element;
236:   using LayoutB = typename Mma::IteratorB::Layout;
237:   using ElementC = typename EpilogueOutputOp::ElementOutput;
```
**EN:** Introduces aliases such as `ElementA`, `LayoutA`, `ElementB`, `LayoutB`, `ElementC` to keep the surrounding template code readable.

**CN:** 引入 `ElementA`, `LayoutA`, `ElementB`, `LayoutB`, `ElementC` 等别名，以提升周围模板代码的可读性。

### Lines 239-240
```cpp
239:   /// Set output tensor C layout
240:   using LayoutC = LayoutA;
```
**EN:** Introduces aliases such as `LayoutC` to keep the surrounding template code readable.

**CN:** 引入 `LayoutC` 等别名，以提升周围模板代码的可读性。

### Lines 242-243
```cpp
242:   using ElementAccumulator = typename EpilogueOutputOp::ElementAccumulator;
243:   using ElementCompute = typename EpilogueOutputOp::ElementCompute;
```
**EN:** Introduces aliases such as `ElementAccumulator`, `ElementCompute` to keep the surrounding template code readable.

**CN:** 引入 `ElementAccumulator`, `ElementCompute` 等别名，以提升周围模板代码的可读性。

### Lines 245-245
```cpp
245:   using WarpMmaOperator = typename Mma::Policy::Operator;
```
**EN:** Introduces aliases such as `WarpMmaOperator` to keep the surrounding template code readable.

**CN:** 引入 `WarpMmaOperator` 等别名，以提升周围模板代码的可读性。

### Lines 247-248
```cpp
247:   using ArchMmaOperator = typename WarpMmaOperator::ArchMmaOperator;
248:   using MathOperator = typename ArchMmaOperator::Operator;
```
**EN:** Introduces aliases such as `ArchMmaOperator`, `MathOperator` to keep the surrounding template code readable.

**CN:** 引入 `ArchMmaOperator`, `MathOperator` 等别名，以提升周围模板代码的可读性。

### Lines 250-251
```cpp
250:   using OperatorClass = typename WarpMmaOperator::OperatorClass;
251:   using ArchTag = typename WarpMmaOperator::ArchTag;
```
**EN:** Introduces aliases such as `OperatorClass`, `ArchTag` to keep the surrounding template code readable.

**CN:** 引入 `OperatorClass`, `ArchTag` 等别名，以提升周围模板代码的可读性。

### Lines 253-255
```cpp
253:   using ThreadblockShape = typename Mma::Shape;
254:   using WarpShape = typename WarpMmaOperator::Shape;
255:   using InstructionShape = typename cutlass::gemm::GemmShape<1, 1, 1>;
```
**EN:** Introduces aliases such as `ThreadblockShape`, `WarpShape`, `InstructionShape` to keep the surrounding template code readable.

**CN:** 引入 `ThreadblockShape`, `WarpShape`, `InstructionShape` 等别名，以提升周围模板代码的可读性。

### Lines 257-259
```cpp
257:   static int const kStages = Mma::kStages;
258:   static IteratorAlgorithm const kIteratorAlgorithm = Mma::IteratorA::kIteratorAlgorithm; 
259:   static StrideSupport const kStrideSupport = Mma::IteratorA::kStrideSupport;
```
**EN:** Defines compile-time constants such as `kStages`, `kIteratorAlgorithm`, `kStrideSupport` that parameterize later logic.

**CN:** 定义 `kStages`, `kIteratorAlgorithm`, `kStrideSupport` 等编译期常量，用来参数化后续逻辑。

### Lines 261-263
```cpp
261:   /// Warp count (concept: GemmShape)
262:   using WarpCount = typename Mma::WarpCount;
263:   static int const kThreadCount = 32 * WarpCount::kCount;
```
**EN:** Introduces aliases such as `WarpCount` to keep the surrounding template code readable.

**CN:** 引入 `WarpCount` 等别名，以提升周围模板代码的可读性。

### Lines 265-267
```cpp
265:   using TensorRefA = typename Mma::IteratorA::TensorRef;
266:   using TensorRefB = typename Mma::IteratorB::TensorRef;
267:   using TensorRefC = cutlass::TensorRef<ElementC, LayoutC>;
```
**EN:** Introduces aliases such as `TensorRefA`, `TensorRefB`, `TensorRefC` to keep the surrounding template code readable.

**CN:** 引入 `TensorRefA`, `TensorRefB`, `TensorRefC` 等别名，以提升周围模板代码的可读性。

### Lines 269-273
```cpp
269:   /// Check iterator A and B convolution dimension are the same and 
270:   // set device::ImplicitGemmConvolution::kConvDim
271:   static_assert(Mma::IteratorA::kConvDim == Mma::IteratorB::kConvDim, 
272:     "Convolution on different different dimensions is not supported");
273:   static int const kConvDim = Mma::IteratorA::kConvDim;
```
**EN:** Defines function `static_assert` for this stage of the convolution workflow.

**CN:** 定义函数 `static_assert`，服务于卷积工作流的这一阶段。

### Lines 275-276
```cpp
275:   /// Conv dimension and problem size structure (Conv2d or Conv3d)
276:   using ConvProblemSize = ConvProblemSize_;
```
**EN:** Introduces aliases such as `ConvProblemSize` to keep the surrounding template code readable.

**CN:** 引入 `ConvProblemSize` 等别名，以提升周围模板代码的可读性。

### Lines 278-278
```cpp
278:   static conv::GroupMode const kGroupMode = GroupMode_;
```
**EN:** Stores member state such as `kGroupMode` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `kGroupMode` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 284-290
```cpp
284:   using ConvOutputIteratorParameter = epilogue::threadblock::ConvOutputIteratorParameter<
285:     LayoutC,
286:     typename Epilogue::OutputTileIterator::Layout, 
287:     TensorRefC,
288:     ConvOperator,
289:     ConvProblemSize
290:     >;
```
**EN:** Introduces aliases such as `ConvOutputIteratorParameter` to keep the surrounding template code readable.

**CN:** 引入 `ConvOutputIteratorParameter` 等别名，以提升周围模板代码的可读性。

### Lines 293-294
```cpp
293:   /// Argument structure
294:   struct Arguments {
```
**EN:** Declares struct `Arguments`. The nearby comment explains that it serves the surrounding arguments logic.

**CN:** 声明结构体 `Arguments`，相邻注释说明它服务于周围的 arguments 逻辑。

### Lines 296-307
```cpp
296:     //
297:     // Data members
298:     //
300:     ConvProblemSize problem_size;
301:     TensorRefA ref_A;
302:     TensorRefB ref_B;
303:     TensorRefB ref_reordered_B;
304:     TensorRefC ref_C;
305:     TensorRefC ref_D;
306:     typename EpilogueOutputOp::Params output_op;
307:     SplitKMode split_k_mode;
```
**EN:** Stores member state such as `problem_size`, `ref_A`, `ref_B`, `ref_reordered_B`, `ref_C`, `ref_D` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `problem_size`, `ref_A`, `ref_B`, `ref_reordered_B`, `ref_C`, `ref_D` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 313-315
```cpp
313:     /// Default ctor
314:     CUTLASS_HOST_DEVICE
315:     Arguments() { }
```
**EN:** Provides constructor-style initialization for `Arguments`.

**CN:** 为 `Arguments` 提供构造式初始化逻辑。

### Lines 317-321
```cpp
317:     CUTLASS_HOST_DEVICE 
318:     Arguments(
319:       ConvProblemSize const & problem_size
320:     ):
321:       problem_size(problem_size) { }
```
**EN:** Provides constructor-style initialization for `Arguments`.

**CN:** 为 `Arguments` 提供构造式初始化逻辑。

### Lines 323-342
```cpp
323:     CUTLASS_HOST_DEVICE
324:     Arguments(
325:       ConvProblemSize const & problem_size,
326:       TensorRefA const & ref_A,
327:       TensorRefB const & ref_B,
328:       TensorRefC const & ref_C,
329:       TensorRefC const & ref_D,
330:       typename EpilogueOutputOp::Params const & output_op,
331:       TensorRefB const & ref_reordered_B = nullptr,
332:       SplitKMode const & split_k_mode = SplitKMode::kSerial
333:     ):
334:       problem_size(problem_size),
335:       ref_A(ref_A),
336:       ref_B(ref_B),
337:       ref_C(ref_C),
338:       ref_D(ref_D),
339:       output_op(output_op),
340:       ref_reordered_B(ref_reordered_B),
341:       split_k_mode(split_k_mode)
342:     {
```
**EN:** Provides constructor-style initialization for `Arguments`.

**CN:** 为 `Arguments` 提供构造式初始化逻辑。

### Lines 344-344
```cpp
344:     }
```
**EN:** Closes the current scope and finishes the surrounding definition block.

**CN:** 关闭当前作用域，并结束外围定义块。

### Lines 346-346
```cpp
346:   };
```
**EN:** Closes the current scope and finishes the surrounding definition block.

**CN:** 关闭当前作用域，并结束外围定义块。

### Lines 348-357
```cpp
348:   using Params =
349:       typename cutlass::conv::kernel::DirectConvolutionParams<Mma,
350:                                                               Epilogue,
351:                                                               ThreadblockSwizzle,
352:                                                               kConvolutionalOperator,
353:                                                               Arguments,
354:                                                               ConvOutputIteratorParameter,
355:                                                               ConvProblemSize,
356:                                                               kGroupMode,
357:                                                               ThreadBlockOutputShape>;
```
**EN:** Introduces aliases such as `Params` to keep the surrounding template code readable.

**CN:** 引入 `Params` 等别名，以提升周围模板代码的可读性。

### Lines 359-359
```cpp
359:   using ReorderKernel = typename cutlass::conv::kernel::ReorderKernel<Params, ElementB>;
```
**EN:** Introduces aliases such as `ReorderKernel` to keep the surrounding template code readable.

**CN:** 引入 `ReorderKernel` 等别名，以提升周围模板代码的可读性。

### Lines 361-365
```cpp
361:   /// Shared memory storage structure
362:   union SharedStorage {
363:     typename Mma::SharedStorage main_loop;
364:     typename Epilogue::SharedStorage epilogue;
365:   };
```
**EN:** Stores member state such as `main_loop`, `epilogue` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `main_loop`, `epilogue` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 371-372
```cpp
371:   CUTLASS_HOST_DEVICE
372:   DirectConvolution() { } 
```
**EN:** Provides constructor-style initialization for `DirectConvolution`.

**CN:** 为 `DirectConvolution` 提供构造式初始化逻辑。

### Lines 374-376
```cpp
374:   /// Executes one ImplicitGEMM
375:   CUTLASS_DEVICE
376:   void operator()(Params const &params, SharedStorage &shared_storage) {
```
**EN:** Implements the callable operator that performs the block's main action.

**CN:** 实现可调用运算符，执行该代码块的主要动作。

### Lines 378-379
```cpp
378:     // Compute threadblock location
379:     ThreadblockSwizzle threadblock_swizzle;
```
**EN:** Stores member state such as `threadblock_swizzle` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `threadblock_swizzle` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 381-382
```cpp
381:     cutlass::gemm::GemmCoord threadblock_tile_idx =
382:         threadblock_swizzle.get_tile_offset(params.swizzle_log_tile);
```
**EN:** Defines function `get_tile_offset` for this stage of the convolution workflow.

**CN:** 定义函数 `get_tile_offset`，服务于卷积工作流的这一阶段。

### Lines 384-386
```cpp
384:     // Early exit if threadblock is out of range
385:     if (params.grid_tiled_shape.m() <= threadblock_tile_idx.m() ||
386:       params.grid_tiled_shape.n() <= threadblock_tile_idx.n()) {
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 388-389
```cpp
388:       return;
389:     }
```
**EN:** Stores member state such as `return` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `return` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 391-399
```cpp
391:     // Compute position within threadblock
392:     int thread_idx = threadIdx.x;
393:     int iterator_column_offset = 0;
394:     int filter_row_offset = 0;
395:     if (kGroupMode != GroupMode::kNone) {
396:       if (kGroupMode == GroupMode::kDepthwise) {
397:         iterator_column_offset += threadblock_tile_idx.n() * Mma::Shape::kN;
398:       }
399:     } 
```
**EN:** Stores member state such as `thread_idx`, `iterator_column_offset`, `filter_row_offset` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `thread_idx`, `iterator_column_offset`, `filter_row_offset` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 401-411
```cpp
401:     // Construct iterators to A and B operands
402:     typename Mma::IteratorA iterator_A(
403:       params.iterator_A,
404:       params.problem_size,
405:       params.ptr_A,
406:       thread_idx,
407:       MatrixCoord(
408:         threadblock_tile_idx.m() + threadblock_tile_idx.k(),
409:         iterator_column_offset
410:       )
411:     );
```
**EN:** Defines function `iterator_A` for this stage of the convolution workflow.

**CN:** 定义函数 `iterator_A`，服务于卷积工作流的这一阶段。

### Lines 413-422
```cpp
413:     typename Mma::IteratorB iterator_B(
414:       params.iterator_B,
415:       params.problem_size,
416:       params.ptr_reordered_B,
417:       thread_idx,
418:       MatrixCoord(
419:         filter_row_offset,
420:         iterator_column_offset
421:       )
422:     );
```
**EN:** Defines function `iterator_B` for this stage of the convolution workflow.

**CN:** 定义函数 `iterator_B`，服务于卷积工作流的这一阶段。

### Lines 424-427
```cpp
424:     // Broadcast the warp_id computed by lane 0 to ensure dependent code
425:     // is compiled as warp-uniform.
426:     int warp_idx = __shfl_sync(0xffffffff, threadIdx.x / 32, 0);
427:     int lane_idx = threadIdx.x % 32;
```
**EN:** Defines function `__shfl_sync` for this stage of the convolution workflow.

**CN:** 定义函数 `__shfl_sync`，服务于卷积工作流的这一阶段。

### Lines 429-434
```cpp
429:     //
430:     // Main loop
431:     //
433:     // Construct thread-scoped matrix multiply
434:     Mma mma(shared_storage.main_loop, thread_idx, warp_idx, lane_idx);
```
**EN:** Defines function `mma` for this stage of the convolution workflow.

**CN:** 定义函数 `mma`，服务于卷积工作流的这一阶段。

### Lines 436-436
```cpp
436:     typename Mma::FragmentC accumulators;
```
**EN:** Stores member state such as `accumulators` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `accumulators` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 438-438
```cpp
438:     accumulators.clear();
```
**EN:** Defines function `clear` for this stage of the convolution workflow.

**CN:** 定义函数 `clear`，服务于卷积工作流的这一阶段。

### Lines 440-444
```cpp
440:     //
441:     // Epilogue
442:     //
444:     EpilogueOutputOp output_op(params.output_op);
```
**EN:** Defines function `output_op` for this stage of the convolution workflow.

**CN:** 定义函数 `output_op`，服务于卷积工作流的这一阶段。

### Lines 446-448
```cpp
446:     // Compute logical position within grid
447:     threadblock_tile_idx =
448:         threadblock_swizzle.get_tile_offset(params.swizzle_log_tile);
```
**EN:** Defines function `get_tile_offset` for this stage of the convolution workflow.

**CN:** 定义函数 `get_tile_offset`，服务于卷积工作流的这一阶段。

### Lines 451-454
```cpp
451:     MatrixCoord threadblock_offset(
452:       threadblock_tile_idx.m() + threadblock_tile_idx.k(),
453:       threadblock_tile_idx.n() * Mma::Shape::kN
454:     );
```
**EN:** Defines function `threadblock_offset` for this stage of the convolution workflow.

**CN:** 定义函数 `threadblock_offset`，服务于卷积工作流的这一阶段。

### Lines 456-463
```cpp
456:     // Tile iterator writing to destination tensor
457:     typename Epilogue::OutputTileIterator iterator_D(
458:       params.iterator_D,
459:       params.ptr_D,
460:       ConvOutputIteratorParameter::extent(params.problem_size),
461:       thread_idx,
462:       threadblock_offset
463:     );
```
**EN:** Defines function `iterator_D` for this stage of the convolution workflow.

**CN:** 定义函数 `iterator_D`，服务于卷积工作流的这一阶段。

### Lines 465-472
```cpp
465:     // Tile iterator reading from source accumulator tensor
466:     typename Epilogue::OutputTileIterator iterator_C(
467:       params.iterator_C,
468:       params.ptr_C,
469:       ConvOutputIteratorParameter::extent(params.problem_size),
470:       thread_idx,
471:       threadblock_offset
472:     );
```
**EN:** Defines function `iterator_C` for this stage of the convolution workflow.

**CN:** 定义函数 `iterator_C`，服务于卷积工作流的这一阶段。

### Lines 475-480
```cpp
475:     // Construct the epilogue
476:     Epilogue epilogue(
477:       shared_storage.epilogue, 
478:       thread_idx, 
479:       warp_idx, 
480:       lane_idx);
```
**EN:** Defines function `epilogue` for this stage of the convolution workflow.

**CN:** 定义函数 `epilogue`，服务于卷积工作流的这一阶段。

### Lines 483-498
```cpp
483:     // Compute threadblock-scoped matrix multiply-add
484:     // Epilogue is fused in the mainloop
485:     mma(params.gemm_k_iterations,
486:         accumulators,
487:         iterator_A,
488:         params.iterator_A,
489:         iterator_B,
490:         params.iterator_B,
491:         accumulators,
492:         epilogue,
493:         output_op,
494:         iterator_D,
495:         iterator_C,
496:         params.split_k_slices);
497:   }
498: };
```
**EN:** Defines function `mma` for this stage of the convolution workflow.

**CN:** 定义函数 `mma`，服务于卷积工作流的这一阶段。

### Lines 502-504
```cpp
502: } // namespace kernel
503: } // namespace conv
504: } // namespace cutlass
```
**EN:** Closes the current scope and finishes the surrounding definition block.

**CN:** 关闭当前作用域，并结束外围定义块。

## Key Concepts / 关键概念
- **EN:** Main role: Template for a multi-staged Depthwise Convolution kernel. **CN:** 核心作用：组合 直接 卷积 对应的内核级卷积逻辑。
- **EN:** Key exported symbols include `DirectConvolutionParams`, `ReorderKernel`, `DirectConvolution`, `Arguments`, `Mma`, `Epilogue`. **CN:** 关键导出符号包括 `DirectConvolutionParams`, `ReorderKernel`, `DirectConvolution`, `Arguments`, `Mma`, `Epilogue`。
- **EN:** Heavy use of C++ templates encodes shapes, layouts, datatypes, and architecture choices. **CN:** 大量使用 C++ 模板来编码形状、布局、数据类型和架构选择。
- **EN:** Iterator logic maps convolution tensors onto tiled memory accesses and implicit-GEMM traversal. **CN:** 迭代器逻辑把卷积张量映射为分块内存访问和隐式 GEMM 遍历。
- **EN:** MMA composition is central: the file either defines math operators directly or prepares data for them. **CN:** MMA 组合是核心：该文件要么直接定义数学算子，要么为其准备数据。

## Dependencies / 依赖关系
### Direct Includes / 直接包含
- `cutlass/cutlass.h`
- `cutlass/aligned_buffer.h`
- `cutlass/array.h`
- `cutlass/numeric_types.h`
- `cutlass/matrix_shape.h`
- `cutlass/platform/platform.h`
- `cutlass/semaphore.h`
- `cutlass/tensor_ref.h`
- `cutlass/layout/tensor.h`
- `cutlass/gemm/gemm.h`
- `cutlass/conv/convolution.h`
- `cutlass/conv/conv2d_problem_size.h`
- `cutlass/conv/conv3d_problem_size.h`
- `cutlass/epilogue/threadblock/output_iterator_parameter.h`

### Internal Relationships / 内部关系
- **EN:** Builds on CUTLASS GEMM shapes, policies, or operators. **CN:** 构建在 CUTLASS GEMM 的形状、策略或算子之上。
