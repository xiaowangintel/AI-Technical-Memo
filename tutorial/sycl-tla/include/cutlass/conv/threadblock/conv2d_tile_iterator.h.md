# conv2d_tile_iterator.h — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cutlass/conv/threadblock/conv2d_tile_iterator.h`
- **Purpose (EN):** Template wraps the tile access iterator concept to load whole tiles from tensors in memory used for implicit GEMM convolution.
- **用途 (CN):** 实现面向 二维卷积 tile 迭代器 的线程块 tile 迭代器。

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

### Lines 31-34
```cpp
 31: /*! \file
 32:     \brief Template wraps the tile access iterator concept to load whole tiles from tensors in
 33:       memory used for implicit GEMM convolution.
 34: */
```
**EN:** Documents the file's purpose, terminology, and high-level usage.

**CN:** 说明文件的用途、术语以及高层使用方式。

### Lines 36-36
```cpp
 36: #pragma once
```
**EN:** Marks this file as a single-inclusion header with `#pragma once`.

**CN:** 通过 `#pragma once` 把该文件标记为单次包含头文件。

### Lines 38-48
```cpp
 38: #include "cutlass/cutlass.h"
 39: #include "cutlass/array.h"
 40: #include "cutlass/coord.h"
 41: #include "cutlass/matrix_shape.h"
 42: #include "cutlass/tensor_ref.h"
 43: #include "cutlass/tensor_view.h"
 44: #include "cutlass/layout/pitch_linear.h"
 45: #include "cutlass/layout/tensor.h"
 46: #include "cutlass/layout/matrix.h"
 47: #include "cutlass/conv/convolution.h"
 48: #include "cutlass/conv/conv2d_problem_size.h"
```
**EN:** Imports direct dependencies used later in the file, including `cutlass.h`, `array.h`, `coord.h`, `matrix_shape.h`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `cutlass.h`, `array.h`, `coord.h`, `matrix_shape.h`。

### Lines 52-54
```cpp
 52: namespace cutlass {
 53: namespace conv {
 54: namespace threadblock {
```
**EN:** Opens the namespace scope `cutlass` that contains these definitions.

**CN:** 打开包含这些定义的命名空间作用域 `cutlass`。

### Lines 58-61
```cpp
 58: template <typename TileAccessIterator_>
 59: class TileIterator {
 60: public:
 61:   using TileAccessIterator = TileAccessIterator_;
```
**EN:** Declares class `TileIterator`, a tile iterator component in the convolution stack.

**CN:** 声明类 `TileIterator`，它是卷积栈中的 tile 迭代器 组件。

### Lines 63-77
```cpp
 63:   using Shape = typename TileAccessIterator::Shape;
 64:   using Element = typename TileAccessIterator::Element;
 65:   using Layout = typename TileAccessIterator::Layout;
 66:   using TensorCoord = typename Layout::TensorCoord;
 67:   using ThreadMap = typename TileAccessIterator::ThreadMap;
 68:   using AccessType = typename TileAccessIterator::AccessType;
 69:   using TensorRef = typename TileAccessIterator::TensorRef;
 70:   using Index = typename TileAccessIterator::Index;
 71:   using LongIndex = typename TileAccessIterator::LongIndex;
 72:   static IteratorAlgorithm const kIteratorAlgorithm = TileAccessIterator::kIteratorAlgorithm;
 73:   static StrideSupport const kStrideSupport = TileAccessIterator::kStrideSupport;
 74:   using Params = typename TileAccessIterator::Params;
 75:   static int const kConvDim = TileAccessIterator::kConvDim;
 76:   using ConvProblemSize = typename TileAccessIterator::ConvProblemSize;
 77:   static int const kAccessesPerVector = TileAccessIterator::kAccessesPerVector;
```
**EN:** Introduces aliases such as `Shape`, `Element`, `Layout`, `TensorCoord`, `ThreadMap`, `AccessType` to keep the surrounding template code readable.

**CN:** 引入 `Shape`, `Element`, `Layout`, `TensorCoord`, `ThreadMap`, `AccessType` 等别名，以提升周围模板代码的可读性。

### Lines 79-82
```cpp
 79:   /// Fragment object to be loaded or stored
 80:   using Fragment = cutlass::Array<
 81:     Element, 
 82:     ThreadMap::Iterations::kCount * ThreadMap::kElementsPerAccess>;
```
**EN:** Introduces aliases such as `Fragment` to keep the surrounding template code readable.

**CN:** 引入 `Fragment` 等别名，以提升周围模板代码的可读性。

### Lines 84-87
```cpp
 84: private:
 86:   /// Internal state
 87:   TileAccessIterator tile_access_iterator_;
```
**EN:** Stores member state such as `tile_access_iterator_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `tile_access_iterator_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 89-100
```cpp
 89: public:
 91:   /// Constructor
 92:   CUTLASS_HOST_DEVICE
 93:   TileIterator(
 94:     Params const &params,
 95:     ConvProblemSize const &problem_size,
 96:     Element const *ptr,
 97:     int thread_idx,
 98:     MatrixCoord const &threadblock_offset = MatrixCoord()
 99:   ):
100:     tile_access_iterator_(params, problem_size, ptr, thread_idx, threadblock_offset) { }
```
**EN:** Provides constructor-style initialization for `TileIterator`.

**CN:** 为 `TileIterator` 提供构造式初始化逻辑。

### Lines 102-105
```cpp
102:   CUTLASS_HOST_DEVICE
103:   static Params getParams(ConvProblemSize const &problem_size, Layout const &layout) {
104:     return TileAccessIterator::getParams(problem_size, layout);
105:   }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 107-111
```cpp
107:   /// Overrides the internal iteration index
108:   CUTLASS_HOST_DEVICE
109:   void set_iteration_index(Index index) {
110:     tile_access_iterator_.set_iteration_index(index);
111:   }
```
**EN:** Defines function `set_iteration_index` for this stage of the convolution workflow.

**CN:** 定义函数 `set_iteration_index`，服务于卷积工作流的这一阶段。

### Lines 113-117
```cpp
113:   /// Adds a pointer offset in units of Element
114:   CUTLASS_HOST_DEVICE
115:   void add_pointer_offset(LongIndex pointer_offset) {
116:     tile_access_iterator_.add_pointer_offset(pointer_offset);
117:   }
```
**EN:** Defines function `add_pointer_offset` for this stage of the convolution workflow.

**CN:** 定义函数 `add_pointer_offset`，服务于卷积工作流的这一阶段。

### Lines 119-124
```cpp
119:   /// Advances to the next tile in memory.
120:   CUTLASS_HOST_DEVICE
121:   TileIterator &operator++() {
122:     tile_access_iterator_.advance();
123:     return *this;
124:   }
```
**EN:** Advances iterator or pipeline state to the next logical position.

**CN:** 将迭代器或流水线状态推进到下一个逻辑位置。

### Lines 126-132
```cpp
126:   /// Advances to the next tile in memory.
127:   CUTLASS_HOST_DEVICE
128:   TileIterator operator++(int) {
129:     TileIterator self(*this);
130:     operator++();
131:     return self;
132:   }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 134-136
```cpp
134:   /// Loads a fragment from memory
135:   CUTLASS_DEVICE
136:   void load_with_pointer_offset(Fragment &frag, Index pointer_offset) {
```
**EN:** Loads a fragment or tile from memory into registers or shared memory.

**CN:** 把片段或 tile 从内存加载到寄存器或共享内存。

### Lines 138-139
```cpp
138:     frag.clear();
139:     AccessType *frag_ptr = reinterpret_cast<AccessType *>(&frag);
```
**EN:** Defines function `clear` for this stage of the convolution workflow.

**CN:** 定义函数 `clear`，服务于卷积工作流的这一阶段。

### Lines 141-146
```cpp
141:     CUTLASS_PRAGMA_UNROLL
142:     for (int s = 0; s < ThreadMap::Iterations::kStrided; ++s) {
143:       CUTLASS_PRAGMA_UNROLL
144:       for (int c = 0; c < ThreadMap::Iterations::kContiguous; ++c) {
145:         CUTLASS_PRAGMA_UNROLL
146:         for (int v = 0; v < kAccessesPerVector; ++v) {
```
**EN:** Iterates through the relevant tile or coordinate space and updates local state.

**CN:** 遍历相关的 tile 或坐标空间，并更新局部状态。

### Lines 148-148
```cpp
148:           int idx = v + kAccessesPerVector * (c + s * ThreadMap::Iterations::kContiguous);
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 150-157
```cpp
150:           cutlass::arch::global_load<
151:             AccessType,
152:             sizeof(AccessType)
153:           >(
154:             frag_ptr[idx],
155:             tile_access_iterator_.get() + pointer_offset,
156:             tile_access_iterator_.valid()
157:           );
```
**EN:** Stores member state such as `AccessType`, `frag_ptr` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `AccessType`, `frag_ptr` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 159-163
```cpp
159:           ++tile_access_iterator_;
160:         }
161:       }
162:     }
163:   }
```
**EN:** Stores member state such as `tile_access_iterator_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `tile_access_iterator_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 165-170
```cpp
165:   /// Loads a fragment from memory
166:   CUTLASS_DEVICE
167:   void load(Fragment &frag) {
168:     tile_access_iterator_.set_iteration_index(0);
169:     load_with_pointer_offset(frag, 0);
170:   }
```
**EN:** Loads a fragment or tile from memory into registers or shared memory.

**CN:** 把片段或 tile 从内存加载到寄存器或共享内存。

### Lines 172-175
```cpp
172:   CUTLASS_DEVICE
173:   void advance() {
174:     tile_access_iterator_.advance();
175:   }
```
**EN:** Advances iterator or pipeline state to the next logical position.

**CN:** 将迭代器或流水线状态推进到下一个逻辑位置。

### Lines 177-179
```cpp
177:   /// Determines whether the Implicit GEMM can execute the given problem.
178:   CUTLASS_HOST_DEVICE
179:   static Status can_implement(ConvProblemSize const &problem_size) {
```
**EN:** Validates alignment, size, grouping, and feature constraints before launch.

**CN:** 在启动前校验对齐、尺寸、分组和特性约束。

### Lines 181-184
```cpp
181:     // dispatch to iterator implementation
182:     return TileAccessIterator::can_implement(problem_size);
183:   }
184: };
```
**EN:** Validates alignment, size, grouping, and feature constraints before launch.

**CN:** 在启动前校验对齐、尺寸、分组和特性约束。

### Lines 187-191
```cpp
187: // Strided Dgrad Tile Iterator
188: template <typename TileAccessIterator_>
189: class TileIteratorStridedDgrad {
190: public:
191:   using TileAccessIterator = TileAccessIterator_;
```
**EN:** Declares class `TileIteratorStridedDgrad`. The nearby comment explains that it serves the surrounding tile iterator strided data-gradient logic.

**CN:** 声明类 `TileIteratorStridedDgrad`，相邻注释说明它服务于周围的 tile 迭代器 跨步 数据梯度 逻辑。

### Lines 193-206
```cpp
193:   using Shape = typename TileAccessIterator::Shape;
194:   using Element = typename TileAccessIterator::Element;
195:   using Layout = typename TileAccessIterator::Layout;
196:   using TensorCoord = typename Layout::TensorCoord;
197:   using ThreadMap = typename TileAccessIterator::ThreadMap;
198:   using AccessType = typename TileAccessIterator::AccessType;
199:   using TensorRef = typename TileAccessIterator::TensorRef;
200:   using Index = typename TileAccessIterator::Index;
201:   using LongIndex = typename TileAccessIterator::LongIndex;
202:   static IteratorAlgorithm const kIteratorAlgorithm = TileAccessIterator::kIteratorAlgorithm;
203:   static StrideSupport const kStrideSupport = TileAccessIterator::kStrideSupport;
204:   using Params = typename TileAccessIterator::Params;
205:   static int const kConvDim = TileAccessIterator::kConvDim;
206:   using ConvProblemSize = typename TileAccessIterator::ConvProblemSize;
```
**EN:** Introduces aliases such as `Shape`, `Element`, `Layout`, `TensorCoord`, `ThreadMap`, `AccessType` to keep the surrounding template code readable.

**CN:** 引入 `Shape`, `Element`, `Layout`, `TensorCoord`, `ThreadMap`, `AccessType` 等别名，以提升周围模板代码的可读性。

### Lines 208-211
```cpp
208:   /// Fragment object to be loaded or stored
209:   using Fragment = cutlass::Array<
210:     Element, 
211:     ThreadMap::Iterations::kCount * ThreadMap::kElementsPerAccess>;
```
**EN:** Introduces aliases such as `Fragment` to keep the surrounding template code readable.

**CN:** 引入 `Fragment` 等别名，以提升周围模板代码的可读性。

### Lines 213-216
```cpp
213: private:
215:   /// Internal state
216:   TileAccessIterator tile_access_iterator_;
```
**EN:** Stores member state such as `tile_access_iterator_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `tile_access_iterator_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 218-238
```cpp
218: public:
220:   /// Constructor (output gradient (Dy) OperandA ctor)
221:   CUTLASS_HOST_DEVICE
222:   TileIteratorStridedDgrad(
223:     Params const &params,
224:     ConvProblemSize const &problem_size,
225:     Element const *ptr,
226:     int thread_idx,
227:     FastDivmod const &stride_h_divmod, FastDivmod const &stride_w_divmod,
228:     int start_r, int start_s,
229:     MatrixCoord const &threadblock_offset = MatrixCoord()
230:   ):
231:     tile_access_iterator_(
232:       params, 
233:       problem_size, 
234:       ptr, 
235:       thread_idx, 
236:       stride_h_divmod, stride_w_divmod, 
237:       start_r, start_s, 
238:       threadblock_offset) { }
```
**EN:** Provides constructor-style initialization for `TileIteratorStridedDgrad`.

**CN:** 为 `TileIteratorStridedDgrad` 提供构造式初始化逻辑。

### Lines 240-255
```cpp
240:   /// Constructor (filter (w) OperandB ctor)
241:   CUTLASS_HOST_DEVICE
242:   TileIteratorStridedDgrad(
243:     Params const &params,
244:     ConvProblemSize const &problem_size,
245:     Element const *ptr,
246:     int thread_idx,
247:     int start_r, int start_s,
248:     MatrixCoord const &threadblock_offset = MatrixCoord()
249:   ):
250:     tile_access_iterator_(params, 
251:       problem_size, 
252:       ptr, 
253:       thread_idx, 
254:       start_r, start_s, 
255:       threadblock_offset) { }
```
**EN:** Provides constructor-style initialization for `TileIteratorStridedDgrad`.

**CN:** 为 `TileIteratorStridedDgrad` 提供构造式初始化逻辑。

### Lines 257-260
```cpp
257:   CUTLASS_HOST_DEVICE
258:   static Params getParams(ConvProblemSize const &problem_size, Layout const &layout) {
259:     return TileAccessIterator::getParams(problem_size, layout);
260:   }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 263-267
```cpp
263:   /// Adds a pointer offset in units of Element
264:   CUTLASS_HOST_DEVICE
265:   void add_pointer_offset(LongIndex pointer_offset) {
266:     tile_access_iterator_.add_pointer_offset(pointer_offset);
267:   }
```
**EN:** Defines function `add_pointer_offset` for this stage of the convolution workflow.

**CN:** 定义函数 `add_pointer_offset`，服务于卷积工作流的这一阶段。

### Lines 269-274
```cpp
269:   /// Advances to the next tile in memory.
270:   CUTLASS_HOST_DEVICE
271:   TileIteratorStridedDgrad &operator++() {
272:     tile_access_iterator_.advance();
273:     return *this;
274:   }
```
**EN:** Advances iterator or pipeline state to the next logical position.

**CN:** 将迭代器或流水线状态推进到下一个逻辑位置。

### Lines 276-282
```cpp
276:   /// Advances to the next tile in memory.
277:   CUTLASS_HOST_DEVICE
278:   TileIteratorStridedDgrad operator++(int) {
279:     TileIteratorStridedDgrad self(*this);
280:     operator++();
281:     return self;
282:   }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 284-286
```cpp
284:   /// Loads a fragment from memory
285:   CUTLASS_DEVICE
286:   void load_with_pointer_offset(Fragment &frag, Index pointer_offset) {
```
**EN:** Loads a fragment or tile from memory into registers or shared memory.

**CN:** 把片段或 tile 从内存加载到寄存器或共享内存。

### Lines 288-289
```cpp
288:     frag.clear();
289:     AccessType *frag_ptr = reinterpret_cast<AccessType *>(&frag);
```
**EN:** Defines function `clear` for this stage of the convolution workflow.

**CN:** 定义函数 `clear`，服务于卷积工作流的这一阶段。

### Lines 291-294
```cpp
291:     CUTLASS_PRAGMA_UNROLL
292:     for (int s = 0; s < ThreadMap::Iterations::kStrided; ++s) {
293:       CUTLASS_PRAGMA_UNROLL
294:       for (int c = 0; c < ThreadMap::Iterations::kContiguous; ++c) {
```
**EN:** Iterates through the relevant tile or coordinate space and updates local state.

**CN:** 遍历相关的 tile 或坐标空间，并更新局部状态。

### Lines 296-303
```cpp
296:         cutlass::arch::global_load<
297:           AccessType,
298:           sizeof(AccessType)
299:         >(
300:           frag_ptr[c + s * ThreadMap::Iterations::kContiguous],
301:           tile_access_iterator_.get() + pointer_offset,
302:           tile_access_iterator_.valid()
303:         );
```
**EN:** Stores member state such as `AccessType`, `frag_ptr` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `AccessType`, `frag_ptr` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 305-308
```cpp
305:         ++tile_access_iterator_;
306:       }
307:     }
308:   }
```
**EN:** Stores member state such as `tile_access_iterator_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `tile_access_iterator_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 310-315
```cpp
310:   /// Loads a fragment from memory
311:   CUTLASS_DEVICE
312:   void load(Fragment &frag) {
313:     tile_access_iterator_.set_iteration_index(0);
314:     load_with_pointer_offset(frag, 0);
315:   }
```
**EN:** Loads a fragment or tile from memory into registers or shared memory.

**CN:** 把片段或 tile 从内存加载到寄存器或共享内存。

### Lines 317-320
```cpp
317:   CUTLASS_DEVICE
318:   void advance() {
319:     tile_access_iterator_.advance();
320:   }
```
**EN:** Advances iterator or pipeline state to the next logical position.

**CN:** 将迭代器或流水线状态推进到下一个逻辑位置。

### Lines 322-324
```cpp
322:   /// Determines whether the Implicit GEMM can execute the given problem.
323:   CUTLASS_HOST_DEVICE
324:   static Status can_implement(ConvProblemSize const &problem_size) {
```
**EN:** Validates alignment, size, grouping, and feature constraints before launch.

**CN:** 在启动前校验对齐、尺寸、分组和特性约束。

### Lines 326-329
```cpp
326:     // dispatch to iterator implementation
327:     return TileAccessIterator::can_implement(problem_size);
328:   }
329: };
```
**EN:** Validates alignment, size, grouping, and feature constraints before launch.

**CN:** 在启动前校验对齐、尺寸、分组和特性约束。

### Lines 332-334
```cpp
332: } // namespace threadblock
333: } // namespace conv
334: } // namespace cutlass
```
**EN:** Closes the current scope and finishes the surrounding definition block.

**CN:** 关闭当前作用域，并结束外围定义块。

## Key Concepts / 关键概念
- **EN:** Main role: Template wraps the tile access iterator concept to load whole tiles from tensors in memory used for implicit GEMM convolution. **CN:** 核心作用：实现面向 二维卷积 tile 迭代器 的线程块 tile 迭代器。
- **EN:** Key exported symbols include `TileIterator`, `TileIteratorStridedDgrad`, `TileAccessIterator`, `Shape`, `Element`, `Layout`. **CN:** 关键导出符号包括 `TileIterator`, `TileIteratorStridedDgrad`, `TileAccessIterator`, `Shape`, `Element`, `Layout`。
- **EN:** Heavy use of C++ templates encodes shapes, layouts, datatypes, and architecture choices. **CN:** 大量使用 C++ 模板来编码形状、布局、数据类型和架构选择。
- **EN:** Iterator logic maps convolution tensors onto tiled memory accesses and implicit-GEMM traversal. **CN:** 迭代器逻辑把卷积张量映射为分块内存访问和隐式 GEMM 遍历。
- **EN:** Precomputed parameter objects reduce runtime address arithmetic in hot loops. **CN:** 预计算参数对象可以减少热点循环中的运行时地址计算。

## Dependencies / 依赖关系
### Direct Includes / 直接包含
- `cutlass/cutlass.h`
- `cutlass/array.h`
- `cutlass/coord.h`
- `cutlass/matrix_shape.h`
- `cutlass/tensor_ref.h`
- `cutlass/tensor_view.h`
- `cutlass/layout/pitch_linear.h`
- `cutlass/layout/tensor.h`
- `cutlass/layout/matrix.h`
- `cutlass/conv/convolution.h`
- `cutlass/conv/conv2d_problem_size.h`

### Internal Relationships / 内部关系
- **EN:** Depends on architecture-specific intrinsics or tags. **CN:** 依赖体系结构相关的内建操作或标签。
