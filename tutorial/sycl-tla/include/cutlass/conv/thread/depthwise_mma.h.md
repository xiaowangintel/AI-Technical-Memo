# depthwise_mma.h — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cutlass/conv/thread/depthwise_mma.h`
- **Purpose (EN):** Templates exposing architecture support for depthwise convolution
- **用途 (CN):** 提供面向 深度卷积 矩阵乘加 的线程级辅助组件。

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
 32:     \brief Templates exposing architecture support for depthwise convolution
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

### Lines 37-42
```cpp
 37: #include "cutlass/cutlass.h"
 38: #include "cutlass/tensor_ref.h"
 39: #include "cutlass/layout/matrix.h"
 40: #include "cutlass/arch/mma.h"
 41: #include "cutlass/gemm/gemm.h"
 42: #include "cutlass/gemm/thread/mma.h"
```
**EN:** Imports direct dependencies used later in the file, including `cutlass.h`, `tensor_ref.h`, `matrix.h`, `mma.h`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `cutlass.h`, `tensor_ref.h`, `matrix.h`, `mma.h`。

### Lines 46-48
```cpp
 46: namespace cutlass {
 47: namespace conv {
 48: namespace thread {
```
**EN:** Opens the namespace scope `cutlass` that contains these definitions.

**CN:** 打开包含这些定义的命名空间作用域 `cutlass`。

### Lines 52-67
```cpp
 52: /// MMA operation
 53: template <
 54:   /// Size of the matrix product (concept: GemmShape)
 55:   typename Shape_,
 56:   /// Number of threads participating
 57:   int kThreads_,
 58:   /// Data type of A elements
 59:   typename ElementA,
 60:   /// Data type of B elements
 61:   typename ElementB,
 62:   /// Element type of C matrix
 63:   typename ElementC,
 64:   /// Inner product operator
 65:   typename Operator
 66: >
 67: struct ElementwiseInnerProduct;
```
**EN:** Declares struct `ElementwiseInnerProduct`. The nearby comment explains that it serves the surrounding elementwise inner product logic.

**CN:** 声明结构体 `ElementwiseInnerProduct`，相邻注释说明它服务于周围的 elementwise inner product 逻辑。

### Lines 70-83
```cpp
 70: /// General implementation
 71: template <
 72:     /// Size of the matrix product (concept: GemmShape)
 73:     typename Shape_,
 74:     /// Data type of A elements
 75:     typename ElementA_,
 76:     /// Data type of B elements
 77:     typename ElementB_,
 78:     /// Element type of C matrix
 79:     typename ElementC_>
 80: struct ElementwiseInnerProduct<Shape_, 1, ElementA_, ElementB_, ElementC_, arch::OpMultiplyAdd> {
 81:   using Shape = Shape_;
 82:   using Operator = arch::OpMultiplyAdd;
 83:   using ElementC = ElementC_;
```
**EN:** Introduces aliases such as `Shape`, `Operator`, `ElementC` to keep the surrounding template code readable.

**CN:** 引入 `Shape`, `Operator`, `ElementC` 等别名，以提升周围模板代码的可读性。

### Lines 85-95
```cpp
 85:   CUTLASS_HOST_DEVICE
 86:   void operator()(Array<ElementC_, Shape::kN> &d,
 87:                   Array<ElementA_, Shape::kN> const &a,
 88:                   Array<ElementB_, Shape::kN> const &b,
 89:                   Array<ElementC_, Shape::kN> const &c) {
 90:     CUTLASS_PRAGMA_UNROLL
 91:     for (int i = 0; i < Shape::kN; ++i) {
 92:       d[i] = a[i] * b[i] + c[i];
 93:     }
 94:   }
 95: };
```
**EN:** Implements the callable operator that performs the block's main action.

**CN:** 实现可调用运算符，执行该代码块的主要动作。

### Lines 98-106
```cpp
 98: /// Specialization of half_t
 99: template <>
100: struct ElementwiseInnerProduct<
101:   gemm::GemmShape<2, 2, 1>,
102:   1,
103:   half_t,
104:   half_t,
105:   half_t,
106:   arch::OpMultiplyAdd> {
```
**EN:** Stores member state such as `half_t` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `half_t` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 108-110
```cpp
108:   using Shape = gemm::GemmShape<2, 2, 1>;
109:   using Operator =  arch::OpMultiplyAdd;
110:   using ElementC = half_t;
```
**EN:** Introduces aliases such as `Shape`, `Operator`, `ElementC` to keep the surrounding template code readable.

**CN:** 引入 `Shape`, `Operator`, `ElementC` 等别名，以提升周围模板代码的可读性。

### Lines 112-118
```cpp
112:   CUTLASS_HOST_DEVICE
113:   void operator()(
114:     Array<half_t, 2> &d,
115:     Array<half_t, 2> const &a,
116:     Array<half_t, 2> const &b,
117:     Array<half_t, 2> const &c
118:   ) {
```
**EN:** Implements the callable operator that performs the block's main action.

**CN:** 实现可调用运算符，执行该代码块的主要动作。

### Lines 120-120
```cpp
120: #if (defined(__CUDA_ARCH__) && (__CUDA_ARCH__ >= 600))
```
**EN:** Starts a conditional-compilation branch for architecture- or feature-specific code.

**CN:** 开始一个面向特定架构或特性的条件编译分支。

### Lines 122-124
```cpp
122:     __half2 const & A = reinterpret_cast<__half2 const &>(a);
123:     __half2 const & B = reinterpret_cast<__half2 const &>(b);
124:     __half2 const & C = reinterpret_cast<__half2 const &>(c);
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 126-126
```cpp
126:     __half2 tmp_D = __hfma2(A, B, C);
```
**EN:** Defines function `__hfma2` for this stage of the convolution workflow.

**CN:** 定义函数 `__hfma2`，服务于卷积工作流的这一阶段。

### Lines 128-128
```cpp
128:     d = reinterpret_cast<Array<half_t, 2> const &>(tmp_D);
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 130-130
```cpp
130: #else
```
**EN:** Ends or switches a conditional-compilation branch.

**CN:** 结束或切换一个条件编译分支。

### Lines 131-135
```cpp
131:     CUTLASS_PRAGMA_UNROLL
132:     for (int i = 0; i < 2; ++i) {
133:       d[i] = a[i] * b[i] + c[i];
134:     }
135: #endif
```
**EN:** Stores member state such as `d` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `d` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 136-137
```cpp
136:   }
137: };
```
**EN:** Closes the current scope and finishes the surrounding definition block.

**CN:** 关闭当前作用域，并结束外围定义块。

### Lines 141-156
```cpp
141: /// Structure to compute the matrix product
142: template <
143:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
144:   typename Shape,
145:   /// Data type of A elements
146:   typename ElementA,
147:   /// Data type of B elements
148:   typename ElementB,
149:   /// Element type of C matrix
150:   typename ElementC,
151:   /// Concept: arch::OpMultiplyAdd or arch::Mma<>
152:   typename Operator = arch::OpMultiplyAdd,
153:   /// Used for partial specialization
154:   typename Enable = bool
155: >
156: struct DepthwiseDirectConvElementwiseInnerProduct;
```
**EN:** Declares struct `DepthwiseDirectConvElementwiseInnerProduct`. The nearby comment explains that it serves the surrounding depthwise direct conv elementwise inner product logic.

**CN:** 声明结构体 `DepthwiseDirectConvElementwiseInnerProduct`，相邻注释说明它服务于周围的 深度卷积 直接 conv elementwise inner product 逻辑。

### Lines 160-173
```cpp
160: /// Gemplate that handles all packed matrix layouts
161: template <
162:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
163:   typename Shape_,
164:   /// Data type of A elements
165:   typename ElementA_,
166:   /// Data type of B elements
167:   typename ElementB_,
168:   /// Element type of C matrix
169:   typename ElementC_,
170:   /// Operator used to compute GEMM
171:   typename Operator_
172: >
173: struct DepthwiseDirectConvElementwiseInnerProductGeneric {
```
**EN:** Declares struct `DepthwiseDirectConvElementwiseInnerProductGeneric`. The nearby comment explains that it serves the surrounding depthwise direct conv elementwise inner product generic logic.

**CN:** 声明结构体 `DepthwiseDirectConvElementwiseInnerProductGeneric`，相邻注释说明它服务于周围的 深度卷积 直接 conv elementwise inner product generic 逻辑。

### Lines 175-176
```cpp
175:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
176:   using Shape = Shape_;
```
**EN:** Introduces aliases such as `Shape` to keep the surrounding template code readable.

**CN:** 引入 `Shape` 等别名，以提升周围模板代码的可读性。

### Lines 178-179
```cpp
178:   /// Data type of operand A
179:   using ElementA = ElementA_;
```
**EN:** Introduces aliases such as `ElementA` to keep the surrounding template code readable.

**CN:** 引入 `ElementA` 等别名，以提升周围模板代码的可读性。

### Lines 181-182
```cpp
181:   /// Data type of operand B
182:   using ElementB = ElementB_;
```
**EN:** Introduces aliases such as `ElementB` to keep the surrounding template code readable.

**CN:** 引入 `ElementB` 等别名，以提升周围模板代码的可读性。

### Lines 184-185
```cpp
184:   /// Element type of operand C
185:   using ElementC = ElementC_;
```
**EN:** Introduces aliases such as `ElementC` to keep the surrounding template code readable.

**CN:** 引入 `ElementC` 等别名，以提升周围模板代码的可读性。

### Lines 187-188
```cpp
187:   /// Underlying mathematical operator
188:   using Operator = Operator_;
```
**EN:** Introduces aliases such as `Operator` to keep the surrounding template code readable.

**CN:** 引入 `Operator` 等别名，以提升周围模板代码的可读性。

### Lines 190-191
```cpp
190:   /// A operand storage
191:   using FragmentA = Array<ElementA, Shape::kMN>;
```
**EN:** Introduces aliases such as `FragmentA` to keep the surrounding template code readable.

**CN:** 引入 `FragmentA` 等别名，以提升周围模板代码的可读性。

### Lines 193-194
```cpp
193:   /// B operand storage
194:   using FragmentB = Array<ElementB, Shape::kN>;
```
**EN:** Introduces aliases such as `FragmentB` to keep the surrounding template code readable.

**CN:** 引入 `FragmentB` 等别名，以提升周围模板代码的可读性。

### Lines 196-197
```cpp
196:   /// C operand storage
197:   using FragmentC = Array<ElementC, Shape::kMN>;
```
**EN:** Introduces aliases such as `FragmentC` to keep the surrounding template code readable.

**CN:** 引入 `FragmentC` 等别名，以提升周围模板代码的可读性。

### Lines 199-206
```cpp
199:   /// Instruction
200:   using MmaOp = cutlass::conv::thread::ElementwiseInnerProduct<
201:     gemm::GemmShape<Shape::kN, Shape::kN, 1>,
202:     1,
203:     ElementA,
204:     ElementB,
205:     ElementC,
206:     Operator>;
```
**EN:** Introduces aliases such as `MmaOp` to keep the surrounding template code readable.

**CN:** 引入 `MmaOp` 等别名，以提升周围模板代码的可读性。

### Lines 213-224
```cpp
213:   /// Computes a matrix product D = A * B + C
214:   CUTLASS_HOST_DEVICE
215:   void operator()(
216:     FragmentC & D,
217:     FragmentA const & A,
218:     FragmentB const & B,
219:     FragmentC const & C) {
220:     Array<ElementC, Shape::kN> *ptr_D = reinterpret_cast<Array<ElementC, Shape::kN> *>(&D);
221:     Array<ElementA, Shape::kN> const *ptr_A =
222:         reinterpret_cast<Array<ElementA, Shape::kN> const *>(&A);
223:     Array<ElementB, Shape::kN> const *ptr_B =
224:         reinterpret_cast<Array<ElementB, Shape::kN> const *>(&B);
```
**EN:** Implements the callable operator that performs the block's main action.

**CN:** 实现可调用运算符，执行该代码块的主要动作。

### Lines 226-226
```cpp
226:     MmaOp mma_op;
```
**EN:** Stores member state such as `mma_op` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `mma_op` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 228-229
```cpp
228:     // Copy accumulators
229:     D = C;
```
**EN:** Stores member state such as `D` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `D` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 231-235
```cpp
231:     // Compute matrix product
232:       CUTLASS_PRAGMA_UNROLL
233:       for (int n = 0; n < Shape::kN / MmaOp::Shape::kN; ++n) {
234:         CUTLASS_PRAGMA_UNROLL
235:         for (int m = 0; m < Shape::kM; ++m) {
```
**EN:** Iterates through the relevant tile or coordinate space and updates local state.

**CN:** 遍历相关的 tile 或坐标空间，并更新局部状态。

### Lines 237-239
```cpp
237:           Array<ElementC, MmaOp::Shape::kN> tmpD = ptr_D[m * Shape::kN / MmaOp::Shape::kN + n];
238:           Array<ElementA, MmaOp::Shape::kN> tmpA = ptr_A[m * Shape::kN / MmaOp::Shape::kN + n];
239:           Array<ElementB, MmaOp::Shape::kN> tmpB = ptr_B[n];
```
**EN:** Stores member state such as `ElementC`, `ElementA`, `ElementB` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `ElementC`, `ElementA`, `ElementB` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 241-241
```cpp
241:           mma_op(tmpD, tmpA, tmpB, tmpD);
```
**EN:** Defines function `mma_op` for this stage of the convolution workflow.

**CN:** 定义函数 `mma_op`，服务于卷积工作流的这一阶段。

### Lines 243-243
```cpp
243:           ptr_D[m * Shape::kN / MmaOp::Shape::kN + n] = tmpD;
```
**EN:** Stores member state such as `ptr_D` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `ptr_D` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 245-248
```cpp
245:         }
246:       }
247:   }
248: };
```
**EN:** Closes the current scope and finishes the surrounding definition block.

**CN:** 关闭当前作用域，并结束外围定义块。

### Lines 252-269
```cpp
252: /// Structure to compute the matrix product
253: template <
254:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
255:   typename Shape_,
256:     /// Data type of A elements
257:   typename ElementA_,
258:   /// Data type of B elements
259:   typename ElementB_,
260:   /// Element type of C matrix
261:   typename ElementC_
262: >
263: struct DepthwiseDirectConvElementwiseInnerProduct<
264:   Shape_,
265:   ElementA_,
266:   ElementB_,
267:   ElementC_,
268:   arch::OpMultiplyAdd
269:   > {
```
**EN:** Stores member state such as `Shape_`, `ElementA_`, `ElementB_`, `ElementC_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `Shape_`, `ElementA_`, `ElementB_`, `ElementC_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 270-271
```cpp
270:   /// Size of the Gemm problem - concept: gemm::GemmShape<>
271:   using Shape = Shape_;
```
**EN:** Introduces aliases such as `Shape` to keep the surrounding template code readable.

**CN:** 引入 `Shape` 等别名，以提升周围模板代码的可读性。

### Lines 273-274
```cpp
273:   /// Data type of operand A
274:   using ElementA = ElementA_;
```
**EN:** Introduces aliases such as `ElementA` to keep the surrounding template code readable.

**CN:** 引入 `ElementA` 等别名，以提升周围模板代码的可读性。

### Lines 276-277
```cpp
276:   /// Data type of operand B
277:   using ElementB = ElementB_;
```
**EN:** Introduces aliases such as `ElementB` to keep the surrounding template code readable.

**CN:** 引入 `ElementB` 等别名，以提升周围模板代码的可读性。

### Lines 279-280
```cpp
279:   /// Element type of operand C
280:   using ElementC = ElementC_;
```
**EN:** Introduces aliases such as `ElementC` to keep the surrounding template code readable.

**CN:** 引入 `ElementC` 等别名，以提升周围模板代码的可读性。

### Lines 282-283
```cpp
282:   /// Underlying mathematical operator
283:   using Operator = arch::OpMultiplyAdd;
```
**EN:** Introduces aliases such as `Operator` to keep the surrounding template code readable.

**CN:** 引入 `Operator` 等别名，以提升周围模板代码的可读性。

### Lines 285-287
```cpp
285:   /// A operand storage
286:   using FragmentA =
287:       Array<ElementA, Shape::kMN>;  // output_tile_size per thread * groups_per_thread
```
**EN:** Introduces aliases such as `FragmentA` to keep the surrounding template code readable.

**CN:** 引入 `FragmentA` 等别名，以提升周围模板代码的可读性。

### Lines 289-290
```cpp
289:   /// B operand storage
290:   using FragmentB = Array<ElementB, Shape::kN>;  // 1 * groups_per_thread
```
**EN:** Introduces aliases such as `FragmentB` to keep the surrounding template code readable.

**CN:** 引入 `FragmentB` 等别名，以提升周围模板代码的可读性。

### Lines 292-294
```cpp
292:   /// C operand storage
293:   using FragmentC =
294:       Array<ElementC, Shape::kMN>;  // output_tile_size per thread * groups_per_thread
```
**EN:** Introduces aliases such as `FragmentC` to keep the surrounding template code readable.

**CN:** 引入 `FragmentC` 等别名，以提升周围模板代码的可读性。

### Lines 296-296
```cpp
296:   static bool const use_optimized = 0;
```
**EN:** Stores member state such as `use_optimized` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `use_optimized` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 298-302
```cpp
298:   using ArchMmaOperator =  DepthwiseDirectConvElementwiseInnerProductGeneric<Shape,
299:                                                         ElementA,
300:                                                         ElementB,
301:                                                         ElementC,
302:                                                         Operator>;
```
**EN:** Introduces aliases such as `ArchMmaOperator` to keep the surrounding template code readable.

**CN:** 引入 `ArchMmaOperator` 等别名，以提升周围模板代码的可读性。

### Lines 308-314
```cpp
308:   /// Computes a matrix product D = A * B + C
309:   CUTLASS_HOST_DEVICE
310:   void operator()(
311:     FragmentC & D,
312:     FragmentA const & A,
313:     FragmentB const & B,
314:     FragmentC const & C) {
```
**EN:** Implements the callable operator that performs the block's main action.

**CN:** 实现可调用运算符，执行该代码块的主要动作。

### Lines 316-316
```cpp
316:     ArchMmaOperator mma;
```
**EN:** Stores member state such as `mma` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `mma` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 318-318
```cpp
318:     mma(D, A, B, C);
```
**EN:** Defines function `mma` for this stage of the convolution workflow.

**CN:** 定义函数 `mma`，服务于卷积工作流的这一阶段。

### Lines 320-321
```cpp
320:   }
321: };
```
**EN:** Closes the current scope and finishes the surrounding definition block.

**CN:** 关闭当前作用域，并结束外围定义块。

### Lines 323-325
```cpp
323: } // namespace thread
324: } // namespace conv
325: } // namespace cutlass
```
**EN:** Closes the current scope and finishes the surrounding definition block.

**CN:** 关闭当前作用域，并结束外围定义块。

## Key Concepts / 关键概念
- **EN:** Main role: Templates exposing architecture support for depthwise convolution **CN:** 核心作用：提供面向 深度卷积 矩阵乘加 的线程级辅助组件。
- **EN:** Key exported symbols include `ElementwiseInnerProduct`, `DepthwiseDirectConvElementwiseInnerProduct`, `DepthwiseDirectConvElementwiseInnerProductGeneric`, `Shape`, `Operator`, `ElementC`. **CN:** 关键导出符号包括 `ElementwiseInnerProduct`, `DepthwiseDirectConvElementwiseInnerProduct`, `DepthwiseDirectConvElementwiseInnerProductGeneric`, `Shape`, `Operator`, `ElementC`。
- **EN:** Heavy use of C++ templates encodes shapes, layouts, datatypes, and architecture choices. **CN:** 大量使用 C++ 模板来编码形状、布局、数据类型和架构选择。
- **EN:** MMA composition is central: the file either defines math operators directly or prepares data for them. **CN:** MMA 组合是核心：该文件要么直接定义数学算子，要么为其准备数据。

## Dependencies / 依赖关系
### Direct Includes / 直接包含
- `cutlass/cutlass.h`
- `cutlass/tensor_ref.h`
- `cutlass/layout/matrix.h`
- `cutlass/arch/mma.h`
- `cutlass/gemm/gemm.h`
- `cutlass/gemm/thread/mma.h`

### Internal Relationships / 内部关系
- **EN:** This file is mostly self-contained within the CUTLASS convolution stack. **CN:** 该文件在 CUTLASS 卷积栈内部相对自包含。
