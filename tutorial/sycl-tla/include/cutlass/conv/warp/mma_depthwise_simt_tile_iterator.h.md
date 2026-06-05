# mma_depthwise_simt_tile_iterator.h — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cutlass/conv/warp/mma_depthwise_simt_tile_iterator.h`
- **Purpose (EN):** Describes the lane policy used by warp-level matrix multiply operators targeting SIMT instructions
- **用途 (CN):** 提供面向 矩阵乘加 深度卷积 simt tile 迭代器 的 warp 级数学或变换辅助组件。

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
 32:     \brief Describes the lane policy used by warp-level matrix multiply operators targeting SIMT
 33:       instructions
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

### Lines 38-41
```cpp
 38: #include "cutlass/cutlass.h"
 39: #include "cutlass/array.h"
 40: #include "cutlass/tensor_ref.h"
 41: #include "cutlass/matrix_shape.h"
```
**EN:** Imports direct dependencies used later in the file, including `cutlass.h`, `array.h`, `tensor_ref.h`, `matrix_shape.h`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `cutlass.h`, `array.h`, `tensor_ref.h`, `matrix_shape.h`。

### Lines 43-43
```cpp
 43: #include "cutlass/conv/convolution.h"
```
**EN:** Imports direct dependencies used later in the file, including `convolution.h`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `convolution.h`。

### Lines 45-45
```cpp
 45: #include "cutlass/arch/memory_sm75.h"
```
**EN:** Imports direct dependencies used later in the file, including `memory_sm75.h`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `memory_sm75.h`。

### Lines 47-47
```cpp
 47: #include "cutlass/layout/matrix.h"
```
**EN:** Imports direct dependencies used later in the file, including `matrix.h`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `matrix.h`。

### Lines 49-51
```cpp
 49: #include "cutlass/gemm/gemm.h"
 50: #include "cutlass/gemm/warp/mma_simt_policy.h"
 51: #include "cutlass/gemm/warp/mma_simt_tile_iterator.h"
```
**EN:** Imports direct dependencies used later in the file, including `gemm.h`, `mma_simt_policy.h`, `mma_simt_tile_iterator.h`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `gemm.h`, `mma_simt_policy.h`, `mma_simt_tile_iterator.h`。

### Lines 55-57
```cpp
 55: namespace cutlass {
 56: namespace conv {
 57: namespace warp {
```
**EN:** Opens the namespace scope `cutlass` that contains these definitions.

**CN:** 打开包含这些定义的命名空间作用域 `cutlass`。

### Lines 61-81
```cpp
 61: /// Iterates over operands to warp-level matrix multiply operations targeting SIMT instructions
 62: ///
 63: /// concept: MutableRandomAccessContiguousTileIteratorConcept
 64: ///
 65: template <
 66:   /// Size of the matrix to load (concept: MatrixShape)
 67:   typename Shape_,
 68:   /// Operand identity
 69:   cutlass::gemm::Operand Operand,
 70:   /// Data type of A elements
 71:   typename Element_,
 72:   /// Layout of operand
 73:   typename Layout_,
 74:   /// Shape of the warp in units of thread (concept: MmaSimtPolicy)
 75:   typename Policy_,
 76:   /// Number of partitions along K dimension - used in sliced-K
 77:   int PartitionsK = 1,
 78:   /// Group Size along kPartition - used in sliced-K
 79:   int PartitionGroupSize = 1
 80: >
 81: class DepthwiseMmaSimtTileIterator;
```
**EN:** Declares class `DepthwiseMmaSimtTileIterator`. The nearby comment explains that it serves the surrounding depthwise MMA simt tile iterator logic.

**CN:** 声明类 `DepthwiseMmaSimtTileIterator`，相邻注释说明它服务于周围的 深度卷积 矩阵乘加 simt tile 迭代器 逻辑。

### Lines 85-113
```cpp
 85: /// Specialization for B operands of row-major layouts
 86: ///
 87: /// Concept: MutableRandomAccessContiguousTileIteratorConcept
 88: ///
 89: template <
 90:     /// Size of the matrix to load (concept: MatrixShape)
 91:     typename Shape_,
 92:     /// Data type of A elements
 93:     typename Element_,
 94:     /// Shape of the warp in units of thread (concept: MmaSimtPolicy)
 95:     typename Policy_,
 96:     /// Number of partitions along K dimension
 97:     int PartitionsK,
 98:     /// Group Size along kPartition - used in sliced-K
 99:     int PartitionGroupSize>
100: class DepthwiseMmaSimtTileIterator<Shape_,
101:                                    cutlass::gemm::Operand::kB,
102:                                    Element_,
103:                                    layout::RowMajor,
104:                                    Policy_,
105:                                    PartitionsK,
106:                                    PartitionGroupSize>
107:     : public cutlass::gemm::warp::MmaSimtTileIterator<Shape_,
108:                                                cutlass::gemm::Operand::kB,
109:                                                Element_,
110:                                                layout::RowMajor,
111:                                                Policy_,
112:                                                PartitionsK,
113:                                                PartitionGroupSize> {
```
**EN:** Stores member state such as `Shape_`, `Element_`, `Policy_`, `PartitionsK`, `kB`, `RowMajor` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `Shape_`, `Element_`, `Policy_`, `PartitionsK`, `kB`, `RowMajor` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 115-122
```cpp
115:   using Base = cutlass::gemm::warp::MmaSimtTileIterator<Shape_,
116:                                                cutlass::gemm::Operand::kB,
117:                                                Element_,
118:                                                layout::RowMajor,
119:                                                Policy_,
120:                                                PartitionsK,
121:                                                PartitionGroupSize>;
122:  public:
```
**EN:** Introduces aliases such as `Base` to keep the surrounding template code readable.

**CN:** 引入 `Base` 等别名，以提升周围模板代码的可读性。

### Lines 123-124
```cpp
123:   /// Shape of tile to load (concept: MatrixShape)
124:   using Shape = Shape_;
```
**EN:** Introduces aliases such as `Shape` to keep the surrounding template code readable.

**CN:** 引入 `Shape` 等别名，以提升周围模板代码的可读性。

### Lines 126-127
```cpp
126:   /// Operand tag
127:   static cutlass::gemm::Operand const kOperand = cutlass::gemm::Operand::kB;
```
**EN:** Stores member state such as `kOperand` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `kOperand` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 129-130
```cpp
129:   /// Element type
130:   using Element = Element_;
```
**EN:** Introduces aliases such as `Element` to keep the surrounding template code readable.

**CN:** 引入 `Element` 等别名，以提升周围模板代码的可读性。

### Lines 132-133
```cpp
132:   /// Layout of policy
133:   using Layout = layout::RowMajor;
```
**EN:** Introduces aliases such as `Layout` to keep the surrounding template code readable.

**CN:** 引入 `Layout` 等别名，以提升周围模板代码的可读性。

### Lines 135-136
```cpp
135:   /// Decomposition of elements among threads
136:   using Policy = Policy_;
```
**EN:** Introduces aliases such as `Policy` to keep the surrounding template code readable.

**CN:** 引入 `Policy` 等别名，以提升周围模板代码的可读性。

### Lines 138-139
```cpp
138:   /// TensorRef type for loading element from a tensor
139:   using TensorRef = typename Base::TensorRef;
```
**EN:** Introduces aliases such as `TensorRef` to keep the surrounding template code readable.

**CN:** 引入 `TensorRef` 等别名，以提升周围模板代码的可读性。

### Lines 141-142
```cpp
141:   /// Index type
142:   using Index = typename TensorRef::Index;
```
**EN:** Introduces aliases such as `Index` to keep the surrounding template code readable.

**CN:** 引入 `Index` 等别名，以提升周围模板代码的可读性。

### Lines 144-145
```cpp
144:   /// Long Index type
145:   using LongIndex = typename TensorRef::LongIndex;
```
**EN:** Introduces aliases such as `LongIndex` to keep the surrounding template code readable.

**CN:** 引入 `LongIndex` 等别名，以提升周围模板代码的可读性。

### Lines 147-148
```cpp
147:   /// Coordinate for an element in the tensor
148:   using TensorCoord = typename TensorRef::TensorCoord;
```
**EN:** Introduces aliases such as `TensorCoord` to keep the surrounding template code readable.

**CN:** 引入 `TensorCoord` 等别名，以提升周围模板代码的可读性。

### Lines 150-151
```cpp
150:   /// Thread-level shape of a fragment
151:   using ThreadShape = typename Base::ThreadShape;
```
**EN:** Introduces aliases such as `ThreadShape` to keep the surrounding template code readable.

**CN:** 引入 `ThreadShape` 等别名，以提升周围模板代码的可读性。

### Lines 153-154
```cpp
153:   /// Number of individual loads
154:   using Iterations =  typename Base::Iterations;
```
**EN:** Introduces aliases such as `Iterations` to keep the surrounding template code readable.

**CN:** 引入 `Iterations` 等别名，以提升周围模板代码的可读性。

### Lines 156-157
```cpp
156:   /// Fragment object holding a thread's part of a tile
157:   using Fragment = typename Base::Fragment;
```
**EN:** Introduces aliases such as `Fragment` to keep the surrounding template code readable.

**CN:** 引入 `Fragment` 等别名，以提升周围模板代码的可读性。

### Lines 159-159
```cpp
159:   static_assert(Policy::LaneMmaShape::kN == 1, "Each thread should be 1 element per LDS along the k-dim");
```
**EN:** Defines function `static_assert` for this stage of the convolution workflow.

**CN:** 定义函数 `static_assert`，服务于卷积工作流的这一阶段。

### Lines 161-166
```cpp
161: private:
163:   MatrixCoord lane_offset_;
164:   int channel_idx_;
165:   int base_channel_idx_;
166:   int warps_n_;
```
**EN:** Stores member state such as `lane_offset_`, `channel_idx_`, `base_channel_idx_`, `warps_n_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `lane_offset_`, `channel_idx_`, `base_channel_idx_`, `warps_n_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 168-172
```cpp
168:  public:
170:   /// Default ctor constructs null iterator
171:   CUTLASS_HOST_DEVICE
172:   DepthwiseMmaSimtTileIterator():Base() { }
```
**EN:** Provides constructor-style initialization for `DepthwiseMmaSimtTileIterator`.

**CN:** 为 `DepthwiseMmaSimtTileIterator` 提供构造式初始化逻辑。

### Lines 174-179
```cpp
174:   /// Constructor from TensorRef
175:   CUTLASS_HOST_DEVICE
176:   DepthwiseMmaSimtTileIterator(
177:     TensorRef ref, 
178:     int lane_id
179:   ) : Base(ref, lane_id) {
```
**EN:** Provides constructor-style initialization for `DepthwiseMmaSimtTileIterator`.

**CN:** 为 `DepthwiseMmaSimtTileIterator` 提供构造式初始化逻辑。

### Lines 181-182
```cpp
181:     // compute offset based on thread ID and lane layout
182:     typename Policy::LaneLayout lane_layout = Policy::get_lane_layout();
```
**EN:** Defines function `get_lane_layout` for this stage of the convolution workflow.

**CN:** 定义函数 `get_lane_layout`，服务于卷积工作流的这一阶段。

### Lines 184-188
```cpp
184:     warps_n_ = -1;
185:     channel_idx_ = 0;
186:     base_channel_idx_ = 0;
187:     lane_offset_ = lane_layout.inverse(lane_id) * MatrixCoord(0, Policy::LaneMmaShape::kN);
188:   }
```
**EN:** Defines function `inverse` for this stage of the convolution workflow.

**CN:** 定义函数 `inverse`，服务于卷积工作流的这一阶段。

### Lines 190-192
```cpp
190:   /// Advances an iterator along logical dimensions of matrix in units of whole tiles
191:   CUTLASS_HOST_DEVICE
192:   DepthwiseMmaSimtTileIterator &add_tile_offset(TensorCoord const &coord) {
```
**EN:** Defines function `add_tile_offset` for this stage of the convolution workflow.

**CN:** 定义函数 `add_tile_offset`，服务于卷积工作流的这一阶段。

### Lines 194-196
```cpp
194:     if(warps_n_ == -1){
195:         warps_n_ = coord.column();
196:     }
```
**EN:** Defines function `column` for this stage of the convolution workflow.

**CN:** 定义函数 `column`，服务于卷积工作流的这一阶段。

### Lines 198-200
```cpp
198:     Base::add_tile_offset(coord);
199:     return *this;
200:   }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 202-206
```cpp
202:   /// Loads a fragment from memory at the location pointed to by the iterator. (vector loads)
203:   CUTLASS_HOST_DEVICE
204:   void load_with_pointer_offset(Fragment &frag, Index pointer_offset) const {
205:     Array<Element, Policy::LaneMmaShape::kN> *dst_ptr =
206:         reinterpret_cast<Array<Element, Policy::LaneMmaShape::kN> *>(&frag);
```
**EN:** Loads a fragment or tile from memory into registers or shared memory.

**CN:** 把片段或 tile 从内存加载到寄存器或共享内存。

### Lines 208-211
```cpp
208:     CUTLASS_PRAGMA_UNROLL
209:     for (int k = 0; k < Iterations::kRow; ++k) {
210:       CUTLASS_PRAGMA_UNROLL
211:       for (int n = 0; n < Iterations::kColumn; ++n) {
```
**EN:** Iterates through the relevant tile or coordinate space and updates local state.

**CN:** 遍历相关的 tile 或坐标空间，并更新局部状态。

### Lines 213-216
```cpp
213:         void const *ptr = this->ref_.data() +
214:                           this->ref_.offset({-(channel_idx_ - base_channel_idx_),
215:                                              n * Policy::WarpShape::kColumn}) +
216:                           pointer_offset / Policy::LaneMmaShape::kN;
```
**EN:** Stores member state such as `kN` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `kN` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 218-220
```cpp
218:         // Base_k of a warp +  Base_k of current threads.
219:         int thread_k_base_idx =
220:             warps_n_ * Shape::kColumn / Policy::LaneMmaShape::kN + lane_offset_.column();
```
**EN:** Defines function `column` for this stage of the convolution workflow.

**CN:** 定义函数 `column`，服务于卷积工作流的这一阶段。

### Lines 222-222
```cpp
222:         if (channel_idx_ + k == thread_k_base_idx + n * Policy::WarpShape::kColumn) {
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 223-224
```cpp
223:           // Depthwise kernel would only do computation when channel == k.
224:           // Loads an element when the current computation channel == the k corresponding to this thread.
```
**EN:** Provides commentary that explains the intent of the nearby implementation.

**CN:** 提供说明性注释，解释邻近实现的设计意图。

### Lines 225-226
```cpp
225:           arch::shared_load(dst_ptr[n + k * Iterations::kColumn], ptr);
226:         } else {
```
**EN:** Defines function `shared_load` for this stage of the convolution workflow.

**CN:** 定义函数 `shared_load`，服务于卷积工作流的这一阶段。

### Lines 227-232
```cpp
227:           // Reduce SMEM load
228:           dst_ptr[n + k * Iterations::kColumn].fill(Element(0));
229:         }
230:       }
231:     }
232:   }
```
**EN:** Defines function `fill` for this stage of the convolution workflow.

**CN:** 定义函数 `fill`，服务于卷积工作流的这一阶段。

### Lines 234-238
```cpp
234:   /// Loads a fragment from memory at the location pointed to by the iterator.
235:   CUTLASS_HOST_DEVICE
236:   void load(Fragment &frag) const {
237:     load_with_pointer_offset(frag, 0);
238:   }
```
**EN:** Loads a fragment or tile from memory into registers or shared memory.

**CN:** 把片段或 tile 从内存加载到寄存器或共享内存。

### Lines 240-243
```cpp
240:   /// Notify the iterator which k-group it is currently pointing to.
241:   ///
242:   /// This does not advance the iterator. Rather, it overrides its internal
243:   /// tracking with constant-valued k-group index
```
**EN:** Provides commentary that explains the intent of the nearby implementation.

**CN:** 提供说明性注释，解释邻近实现的设计意图。

### Lines 244-251
```cpp
244:   CUTLASS_DEVICE
245:   void set_kgroup_index(int k_group) {
246:     if(k_group % PartitionGroupSize == 0 && k_group != 0){
247:       base_channel_idx_ = k_group;
248:     }
249:     channel_idx_ = k_group;
250:   }
251: };
```
**EN:** Defines function `set_kgroup_index` for this stage of the convolution workflow.

**CN:** 定义函数 `set_kgroup_index`，服务于卷积工作流的这一阶段。

### Lines 255-282
```cpp
255: template <
256:     /// Size of the matrix to load (concept: MatrixShape)
257:     typename Shape_,
258:     /// Size of filter (concept: gemm::GemmShape<Depth, Height, Width>)
259:     typename FilterShape_,
260:     /// Size of the matrix to load (concept: MatrixShape)
261:     typename ThreadOutputShape_,
262:     /// Size of the matrix to load (concept: MatrixShape)
263:     typename ThreadBlockOutputShape_,
264:     /// Operand identity
265:     cutlass::gemm::Operand Operand,
266:     /// Data type of A elements
267:     typename Element_,
268:     /// Shape of the warp in units of thread (concept: MmaSimtPolicy)
269:     typename Policy_,
270:     /// Iterator algo type
271:     conv::IteratorAlgorithm IteratorAlgorithm = IteratorAlgorithm::kAnalytic,
272:     /// Stride ( MatrixShape<Height, Width> )
273:     typename StrideShape = cutlass::MatrixShape<-1, -1>,   
274:     /// Dilation ( MatrixShape<Height, Width> )
275:     typename DilationShape =  cutlass::MatrixShape<-1, -1>,
276:     /// Activation Shape loaded by threadblock
277:     typename ActivationShape = cutlass::conv::TensorNHWCShape<-1,-1,-1,-1>,
278:     /// Number of partitions along K dimension - used in sliced-K
279:     int PartitionsK = 1,
280:     /// Group Size along kPartition - used in sliced-K
281:     int PartitionGroupSize = 1>
282: class DepthwiseDirect2dConvSimtTileIterator;
```
**EN:** Declares class `DepthwiseDirect2dConvSimtTileIterator`. The nearby comment explains that it serves the surrounding depthwise direct2d conv simt tile iterator logic.

**CN:** 声明类 `DepthwiseDirect2dConvSimtTileIterator`，相邻注释说明它服务于周围的 深度卷积 direct2d conv simt tile 迭代器 逻辑。

### Lines 285-327
```cpp
285: /// Specialization for A operands of row-major layouts
286: ///
287: /// Concept: MutableRandomAccessContiguousTileIteratorConcept
288: ///
289: template <
290:     /// Size of the matrix to load (concept: MatrixShape)
291:     typename Shape_,
292:     /// Size of filter (concept: gemm::GemmShape<Depth, Height, Width>)
293:     typename FilterShape_,
294:     /// Size of the matrix to load (concept: TensorNHWC)
295:     typename ThreadOutputShape_,
296:     /// Size of the matrix to load (concept: TensorNHWC)
297:     typename ThreadBlockOutputShape_,
298:     /// Data type of A elements
299:     typename Element_,
300:     /// Shape of the warp in units of thread (concept: MmaSimtPolicy)
301:     typename Policy_,
302:     /// Iterator algo type
303:     conv::IteratorAlgorithm IteratorAlgorithm,
304:     /// Stride ( MatrixShape<Height, Width> )
305:     typename StrideShape,   
306:     /// Dilation ( MatrixShape<Height, Width> )
307:     typename DilationShape,
308:     /// Activation Shape loaded by threadblock
309:     typename ActivationShape,
310:     /// Number of partitions along K dimension - used in sliced-K
311:     int PartitionsK,
312:     /// Group Size along kPartition - used in sliced-K
313:     int PartitionGroupSize>
314: class DepthwiseDirect2dConvSimtTileIterator<Shape_,
315:                                             FilterShape_,
316:                                             ThreadOutputShape_,
317:                                             ThreadBlockOutputShape_,
318:                                             cutlass::gemm::Operand::kA,
319:                                             Element_,
320:                                             Policy_,
321:                                             IteratorAlgorithm,
322:                                             StrideShape,   
323:                                             DilationShape,
324:                                             ActivationShape,
325:                                             PartitionsK,
326:                                             PartitionGroupSize> {
327:  public:
```
**EN:** Stores member state such as `Shape_`, `FilterShape_`, `ThreadOutputShape_`, `ThreadBlockOutputShape_`, `Element_`, `Policy_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `Shape_`, `FilterShape_`, `ThreadOutputShape_`, `ThreadBlockOutputShape_`, `Element_`, `Policy_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 328-329
```cpp
328:   /// Shape of tile to load (concept: MatrixShape)
329:   using Shape = Shape_;
```
**EN:** Introduces aliases such as `Shape` to keep the surrounding template code readable.

**CN:** 引入 `Shape` 等别名，以提升周围模板代码的可读性。

### Lines 331-332
```cpp
331:   /// Shape of filter (concept: gemm::GemmShape<Depth, Height, Width>)
332:   using FilterShape = FilterShape_;
```
**EN:** Introduces aliases such as `FilterShape` to keep the surrounding template code readable.

**CN:** 引入 `FilterShape` 等别名，以提升周围模板代码的可读性。

### Lines 334-335
```cpp
334:   /// Shape of tile to load (concept: TensorNHWC)
335:   using ThreadOutputShape = ThreadOutputShape_;
```
**EN:** Introduces aliases such as `ThreadOutputShape` to keep the surrounding template code readable.

**CN:** 引入 `ThreadOutputShape` 等别名，以提升周围模板代码的可读性。

### Lines 337-338
```cpp
337:   /// Shape of tile to load (concept: TensorNHWC)
338:   using ThreadBlockOutputShape = ThreadBlockOutputShape_;
```
**EN:** Introduces aliases such as `ThreadBlockOutputShape` to keep the surrounding template code readable.

**CN:** 引入 `ThreadBlockOutputShape` 等别名，以提升周围模板代码的可读性。

### Lines 340-341
```cpp
340:   /// Operand tag
341:   static cutlass::gemm::Operand const kOperand = cutlass::gemm::Operand::kA;
```
**EN:** Stores member state such as `kOperand` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `kOperand` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 343-344
```cpp
343:   /// Element type
344:   using Element = Element_;
```
**EN:** Introduces aliases such as `Element` to keep the surrounding template code readable.

**CN:** 引入 `Element` 等别名，以提升周围模板代码的可读性。

### Lines 346-347
```cpp
346:   /// Layout of policy
347:   using Layout = layout::RowMajor;
```
**EN:** Introduces aliases such as `Layout` to keep the surrounding template code readable.

**CN:** 引入 `Layout` 等别名，以提升周围模板代码的可读性。

### Lines 349-350
```cpp
349:   /// Decomposition of elements among threads
350:   using Policy = Policy_;
```
**EN:** Introduces aliases such as `Policy` to keep the surrounding template code readable.

**CN:** 引入 `Policy` 等别名，以提升周围模板代码的可读性。

### Lines 352-353
```cpp
352:   /// TensorRef type for loading element from a tensor
353:   using TensorRef = TensorRef<Element, Layout>;
```
**EN:** Introduces aliases such as `TensorRef` to keep the surrounding template code readable.

**CN:** 引入 `TensorRef` 等别名，以提升周围模板代码的可读性。

### Lines 355-356
```cpp
355:   /// Index type
356:   using Index = typename TensorRef::Index;
```
**EN:** Introduces aliases such as `Index` to keep the surrounding template code readable.

**CN:** 引入 `Index` 等别名，以提升周围模板代码的可读性。

### Lines 358-359
```cpp
358:   /// Long Index type
359:   using LongIndex = typename TensorRef::LongIndex;
```
**EN:** Introduces aliases such as `LongIndex` to keep the surrounding template code readable.

**CN:** 引入 `LongIndex` 等别名，以提升周围模板代码的可读性。

### Lines 361-362
```cpp
361:   /// Coordinate for an element in the tensor
362:   using TensorCoord = typename TensorRef::TensorCoord;
```
**EN:** Introduces aliases such as `TensorCoord` to keep the surrounding template code readable.

**CN:** 引入 `TensorCoord` 等别名，以提升周围模板代码的可读性。

### Lines 364-369
```cpp
364:   //
365:   // Derived quantities
366:   //
368:   static_assert(!(Shape::kRow % Policy::WarpShape::kRow), 
369:     "The warp-level GEMM M size must be divisible by the number of threads arranged along the M dimension.");
```
**EN:** Defines function `static_assert` for this stage of the convolution workflow.

**CN:** 定义函数 `static_assert`，服务于卷积工作流的这一阶段。

### Lines 371-374
```cpp
371:   static_assert(Shape::kRow > 0, "Shape::kRow must be greater than zero.");
372:   static_assert(Shape::kColumn > 0, "Shape::kColumn must be greater than zero.");
373:   static_assert(Policy::WarpShape::kRow > 0, "Policy::WarpShape::kRow must be greater than zero.");
374:   static_assert(Shape::kRow / Policy::WarpShape::kRow > 0, "Shape::kRow / Policy::WarpShape::kRow must be greater than zero.");
```
**EN:** Defines function `static_assert` for this stage of the convolution workflow.

**CN:** 定义函数 `static_assert`，服务于卷积工作流的这一阶段。

### Lines 376-380
```cpp
376: // Thread-level shape of a fragment
377:   using ThreadShape = MatrixShape<
378:     ThreadOutputShape::kNHW, // Output tile shape Computed by current threads
379:     ThreadOutputShape::kC
380:   >;
```
**EN:** Introduces aliases such as `ThreadShape` to keep the surrounding template code readable.

**CN:** 引入 `ThreadShape` 等别名，以提升周围模板代码的可读性。

### Lines 382-383
```cpp
382:   static_assert(!(ThreadShape::kColumn % Policy::LaneMmaShape::kN), 
383:     "Thread-level GEMM must be divisible by Policy::LaneMmaShape.");
```
**EN:** Defines function `static_assert` for this stage of the convolution workflow.

**CN:** 定义函数 `static_assert`，服务于卷积工作流的这一阶段。

### Lines 385-389
```cpp
385:   /// Number of individual loads
386:   using Iterations = MatrixShape<
387:     ThreadShape::kRow,
388:     ThreadShape::kColumn / Policy::LaneMmaShape::kN
389:   >;
```
**EN:** Introduces aliases such as `Iterations` to keep the surrounding template code readable.

**CN:** 引入 `Iterations` 等别名，以提升周围模板代码的可读性。

### Lines 391-394
```cpp
391:   using ThreadTileCount = MatrixShape<
392:     ThreadBlockOutputShape::kH / ThreadOutputShape::kH,
393:     ThreadBlockOutputShape::kW / ThreadOutputShape::kW
394:   >;
```
**EN:** Introduces aliases such as `ThreadTileCount` to keep the surrounding template code readable.

**CN:** 引入 `ThreadTileCount` 等别名，以提升周围模板代码的可读性。

### Lines 396-397
```cpp
396:   /// Fragment object holding a thread's part of a tile
397:   using Fragment = Array<Element, ThreadShape::kCount>;
```
**EN:** Introduces aliases such as `Fragment` to keep the surrounding template code readable.

**CN:** 引入 `Fragment` 等别名，以提升周围模板代码的可读性。

### Lines 399-402
```cpp
399: protected:
401:   /// Internal reference
402:   cutlass::TensorRef<Array<Element, Policy::LaneMmaShape::kN>, layout::RowMajor> ref_;
```
**EN:** Stores member state such as `Element` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `Element` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 404-407
```cpp
404:   int activation_offset[ThreadOutputShape::kH][ThreadOutputShape::kW][Iterations::kColumn];
405:   int iterator_r_;
406:   int iterator_s_;
407:   int iterator_offset_;
```
**EN:** Stores member state such as `iterator_r_`, `iterator_s_`, `iterator_offset_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `iterator_r_`, `iterator_s_`, `iterator_offset_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 409-410
```cpp
409:   int inc_next_s_ ;
410:   int inc_next_r_ ;
```
**EN:** Stores member state such as `inc_next_s_`, `inc_next_r_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `inc_next_s_`, `inc_next_r_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 412-413
```cpp
412:   MatrixCoord lane_offset_;
413: public:
```
**EN:** Stores member state such as `lane_offset_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `lane_offset_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 415-417
```cpp
415:   /// Default ctor constructs null iterator
416:   CUTLASS_HOST_DEVICE
417:   DepthwiseDirect2dConvSimtTileIterator() { }
```
**EN:** Provides constructor-style initialization for `DepthwiseDirect2dConvSimtTileIterator`.

**CN:** 为 `DepthwiseDirect2dConvSimtTileIterator` 提供构造式初始化逻辑。

### Lines 419-424
```cpp
419:   /// Constructor from TensorRef
420:   CUTLASS_HOST_DEVICE
421:   DepthwiseDirect2dConvSimtTileIterator(
422:     TensorRef ref, 
423:     int lane_id
424:   ) {
```
**EN:** Provides constructor-style initialization for `DepthwiseDirect2dConvSimtTileIterator`.

**CN:** 为 `DepthwiseDirect2dConvSimtTileIterator` 提供构造式初始化逻辑。

### Lines 426-427
```cpp
426:     // compute offset based on thread ID and lane layout
427:     typename Policy::LaneLayout lane_layout = Policy::get_lane_layout();
```
**EN:** Defines function `get_lane_layout` for this stage of the convolution workflow.

**CN:** 定义函数 `get_lane_layout`，服务于卷积工作流的这一阶段。

### Lines 429-430
```cpp
429:     // Set channel offset
430:     lane_offset_ = lane_layout.inverse(lane_id) * MatrixCoord(0, Policy::LaneMmaShape::kN);
```
**EN:** Defines function `inverse` for this stage of the convolution workflow.

**CN:** 定义函数 `inverse`，服务于卷积工作流的这一阶段。

### Lines 432-432
```cpp
432:     ref.add_coord_offset(lane_offset_);
```
**EN:** Defines function `add_coord_offset` for this stage of the convolution workflow.

**CN:** 定义函数 `add_coord_offset`，服务于卷积工作流的这一阶段。

### Lines 434-435
```cpp
434:     ref_.reset(reinterpret_cast<Array<Element, Policy::LaneMmaShape::kN> *>(ref.data()),
435:                ref.stride(0) / Policy::LaneMmaShape::kN);
```
**EN:** Defines function `reset` for this stage of the convolution workflow.

**CN:** 定义函数 `reset`，服务于卷积工作流的这一阶段。

### Lines 437-440
```cpp
437:     iterator_r_ = 0;
438:     iterator_s_ = 0;
439:     iterator_offset_ = 0;
440:   }
```
**EN:** Stores member state such as `iterator_r_`, `iterator_s_`, `iterator_offset_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `iterator_r_`, `iterator_s_`, `iterator_offset_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 442-447
```cpp
442:   /// Adds a pointer offset to internal pointer(s) to advance through memory
443:   CUTLASS_HOST_DEVICE
444:   DepthwiseDirect2dConvSimtTileIterator &add_pointer_offset(LongIndex offset) {
445:     ref_.add_pointer_offset(offset);
446:     return *this;
447:   }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 449-452
```cpp
449:   /// Loads a fragment from memory at the location pointed to by the iterator.
450:   template<typename Params>
451:   CUTLASS_HOST_DEVICE
452:   void setup_initial_status(Params const& params)  {
```
**EN:** Defines function `setup_initial_status` for this stage of the convolution workflow.

**CN:** 定义函数 `setup_initial_status`，服务于卷积工作流的这一阶段。

### Lines 454-455
```cpp
454:     inc_next_s_ = params.inc_next[0];
455:     inc_next_r_ = params.inc_next[1];
```
**EN:** Stores member state such as `inc_next_s_`, `inc_next_r_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `inc_next_s_`, `inc_next_r_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 457-462
```cpp
457:     // Get base HW offset of current threads
458:     int threadgroup = threadIdx.x / (ThreadBlockOutputShape::kC / ThreadOutputShape::kC);
459:     int base_p_ =
460:         (threadgroup / (ThreadTileCount::kColumn)) * ThreadOutputShape::kH;
461:     int base_q_ =
462:         (threadgroup % (ThreadTileCount::kColumn)) * ThreadOutputShape::kW;
```
**EN:** Stores member state such as `base_p_`, `base_q_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `base_p_`, `base_q_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 464-471
```cpp
464:     CUTLASS_PRAGMA_UNROLL
465:     for (int p = 0; p < ThreadOutputShape::kH; ++p) {
466:       CUTLASS_PRAGMA_UNROLL
467:       for (int q = 0; q < ThreadOutputShape::kW; ++q) {
468:         CUTLASS_PRAGMA_UNROLL
469:         for (int col = 0; col < Iterations::kColumn; ++col) {
470:           int base_w = (base_q_ + q) * params.stride[0];
471:           int base_h = (base_p_ + p) * params.stride[1];
```
**EN:** Iterates through the relevant tile or coordinate space and updates local state.

**CN:** 遍历相关的 tile 或坐标空间，并更新局部状态。

### Lines 473-478
```cpp
473:           int offset = base_h * params.activation_tile_w + base_w;
474:           activation_offset[p][q][col] = offset;
475:         }
476:       }
477:     }
478:   }
```
**EN:** Stores member state such as `offset` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `offset` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 481-483
```cpp
481:   /// Advances an iterator along logical dimensions of matrix in units of whole tiles
482:   CUTLASS_HOST_DEVICE
483:   DepthwiseDirect2dConvSimtTileIterator &add_tile_offset(TensorCoord const &coord) {
```
**EN:** Defines function `add_tile_offset` for this stage of the convolution workflow.

**CN:** 定义函数 `add_tile_offset`，服务于卷积工作流的这一阶段。

### Lines 484-487
```cpp
484:     // Set warp row and col start
485:     lane_offset_ = MatrixCoord({lane_offset_.row() + coord.row() * Shape::kRow, lane_offset_.column()});
486:     return *this;
487:   }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 489-496
```cpp
489:   /// Advances an iterator along logical dimensions of matrix in units of whole tiles
490:   CUTLASS_HOST_DEVICE
491:   void advance(int32_t pointer_offset) {
492:     ref_.reset(ref_.data() + pointer_offset / sizeof(Element) / Policy::LaneMmaShape::kN);
493:     iterator_s_ = 0;
494:     iterator_r_ = 0;
495:     iterator_offset_ = 0;
496:   }
```
**EN:** Advances iterator or pipeline state to the next logical position.

**CN:** 将迭代器或流水线状态推进到下一个逻辑位置。

### Lines 498-503
```cpp
498:   /// Advances the iterator along the advance dimension
499:   CUTLASS_HOST_DEVICE
500:   DepthwiseDirect2dConvSimtTileIterator &operator++() {
501:     ++iterator_s_;
502:     if (iterator_s_ < FilterShape::kColumn) {
503:       iterator_offset_ += inc_next_s_;
```
**EN:** Stores member state such as `iterator_s_`, `inc_next_s_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `iterator_s_`, `inc_next_s_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 505-506
```cpp
505:       return *this;
506:     }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 508-508
```cpp
508:     iterator_s_ = 0;
```
**EN:** Stores member state such as `iterator_s_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `iterator_s_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 510-514
```cpp
510:     ++iterator_r_;
511:     if (iterator_r_ < FilterShape::kRow) {
512:       iterator_offset_ += inc_next_r_;
513:       return *this;
514:     }
```
**EN:** Stores member state such as `iterator_r_`, `inc_next_r_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `iterator_r_`, `inc_next_r_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 516-519
```cpp
516:     iterator_r_ = 0;
517:     iterator_offset_ = 0;
518:     return *this;
519:   }
```
**EN:** Stores member state such as `iterator_r_`, `iterator_offset_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `iterator_r_`, `iterator_offset_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 521-523
```cpp
521:   /// Advances the iterator along the advance dimension
522:   CUTLASS_HOST_DEVICE
523:   DepthwiseDirect2dConvSimtTileIterator & operator--() {
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 524-526
```cpp
524:     // Do nothing
525:     return *this;
526:   }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 528-530
```cpp
528:   /// Loads a fragment from memory at the location pointed to by the iterator. (vector loads)
529:   CUTLASS_HOST_DEVICE
530:   void load_with_pointer_offset(Fragment &frag, Index pointer_offset) const {
```
**EN:** Loads a fragment or tile from memory into registers or shared memory.

**CN:** 把片段或 tile 从内存加载到寄存器或共享内存。

### Lines 532-533
```cpp
532:     Array<Element, Policy::LaneMmaShape::kN> *dst_ptr = 
533:       reinterpret_cast<Array<Element, Policy::LaneMmaShape::kN> *>(&frag);
```
**EN:** Stores member state such as `Element` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `Element` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 536-550
```cpp
536:     CUTLASS_PRAGMA_UNROLL
537:     for (int p = 0; p < ThreadOutputShape::kH; ++p) {
538:       CUTLASS_PRAGMA_UNROLL
539:       for (int q = 0; q < ThreadOutputShape::kW; ++q) {
540:         CUTLASS_PRAGMA_UNROLL
541:         for (int n = 0; n < Iterations::kColumn; ++n) {
542:           void const *ptr = ref_.data() +
543:                             ref_.offset({activation_offset[p][q][n] + (iterator_offset_),
544:                                          n * Policy::WarpShape::kColumn}) +
545:                             pointer_offset / Policy::LaneMmaShape::kN;
546:           arch::shared_load(dst_ptr[n + q + p * ThreadOutputShape::kW], ptr);
547:         }
548:       }
549:     }
550:   }
```
**EN:** Iterates across tiles, vectors, or coordinates to update local state and predicates.

**CN:** 遍历 tile、向量或坐标，以更新局部状态和谓词。

### Lines 552-556
```cpp
552:   /// Loads a fragment from memory at the location pointed to by the iterator.
553:   CUTLASS_HOST_DEVICE
554:   void load(Fragment &frag) const {
555:     load_with_pointer_offset(frag, 0);
556:   }
```
**EN:** Loads a fragment or tile from memory into registers or shared memory.

**CN:** 把片段或 tile 从内存加载到寄存器或共享内存。

### Lines 558-560
```cpp
558:   /// Stores a fragment to memory at the location pointed to by the iterator
559:   CUTLASS_HOST_DEVICE
560:   void store_with_pointer_offset(Fragment const &frag, Index pointer_offset) const {
```
**EN:** Stores computed data back to memory.

**CN:** 将计算结果写回内存。

### Lines 561-562
```cpp
561:     // Do nothing at present.
562:   }
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 564-568
```cpp
564:   /// Stores a fragment to memory at the location pointed to by the iterator
565:   CUTLASS_HOST_DEVICE
566:   void store(Fragment const &frag, Index pointer_offset) const {
567:     store_with_pointer_offset(frag, 0);
568:   }
```
**EN:** Stores computed data back to memory.

**CN:** 将计算结果写回内存。

### Lines 570-571
```cpp
570:   CUTLASS_DEVICE
571:   void set_kgroup_index(int k_group) {
```
**EN:** Defines function `set_kgroup_index` for this stage of the convolution workflow.

**CN:** 定义函数 `set_kgroup_index`，服务于卷积工作流的这一阶段。

### Lines 572-574
```cpp
572:     // no operation here
573:   }
574: };
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 577-617
```cpp
577: /// Specialization for A operands of row-major layouts
578: ///
579: /// Concept: MutableRandomAccessContiguousTileIteratorConcept
580: ///
581: template <
582:     /// Size of the matrix to load (concept: MatrixShape)
583:     typename Shape_,
584:     /// Size of filter (concept: gemm::GemmShape<Depth, Height, Width>)
585:     typename FilterShape_,
586:     /// Size of the matrix to load (concept: TensorNHWC)
587:     typename ThreadOutputShape_,
588:     /// Size of the matrix to load (concept: TensorNHWC)
589:     typename ThreadBlockOutputShape_,
590:     /// Data type of A elements
591:     typename Element_,
592:     /// Shape of the warp in units of thread (concept: MmaSimtPolicy)
593:     typename Policy_,
594:     /// Stride ( MatrixShape<Height, Width> )
595:     typename StrideShape_,
596:     /// Dilation ( MatrixShape<Height, Width> )
597:     typename DilationShape_,
598:     /// Activation Shape loaded by threadblock
599:     typename ActivationShape_,
600:     /// Number of partitions along K dimension - used in sliced-K
601:     int PartitionsK,
602:     /// Group Size along kPartition - used in sliced-K
603:     int PartitionGroupSize>
604: class DepthwiseDirect2dConvSimtTileIterator<Shape_,
605:                                             FilterShape_,
606:                                             ThreadOutputShape_,
607:                                             ThreadBlockOutputShape_,
608:                                             cutlass::gemm::Operand::kA,
609:                                             Element_,
610:                                             Policy_,
611:                                             IteratorAlgorithm::kFixedStrideDilation,
612:                                             StrideShape_,
613:                                             DilationShape_,
614:                                             ActivationShape_,
615:                                             PartitionsK,
616:                                             PartitionGroupSize> {
617:  public:
```
**EN:** Stores member state such as `Shape_`, `FilterShape_`, `ThreadOutputShape_`, `ThreadBlockOutputShape_`, `Element_`, `Policy_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `Shape_`, `FilterShape_`, `ThreadOutputShape_`, `ThreadBlockOutputShape_`, `Element_`, `Policy_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 618-619
```cpp
618:   /// Shape of tile to load (concept: MatrixShape)
619:   using Shape = Shape_;
```
**EN:** Introduces aliases such as `Shape` to keep the surrounding template code readable.

**CN:** 引入 `Shape` 等别名，以提升周围模板代码的可读性。

### Lines 621-622
```cpp
621:   /// Shape of filter (concept: gemm::GemmShape<Depth, Height, Width>)
622:   using FilterShape = FilterShape_;
```
**EN:** Introduces aliases such as `FilterShape` to keep the surrounding template code readable.

**CN:** 引入 `FilterShape` 等别名，以提升周围模板代码的可读性。

### Lines 624-625
```cpp
624:   /// Shape of tile to load (concept: TensorNHWC)
625:   using ThreadOutputShape = ThreadOutputShape_;
```
**EN:** Introduces aliases such as `ThreadOutputShape` to keep the surrounding template code readable.

**CN:** 引入 `ThreadOutputShape` 等别名，以提升周围模板代码的可读性。

### Lines 627-628
```cpp
627:   /// Shape of tile to load (concept: TensorNHWC)
628:   using ThreadBlockOutputShape = ThreadBlockOutputShape_;
```
**EN:** Introduces aliases such as `ThreadBlockOutputShape` to keep the surrounding template code readable.

**CN:** 引入 `ThreadBlockOutputShape` 等别名，以提升周围模板代码的可读性。

### Lines 630-631
```cpp
630:   /// Stride ( MatrixShape<Height, Width> )
631:   using StrideShape = StrideShape_;
```
**EN:** Introduces aliases such as `StrideShape` to keep the surrounding template code readable.

**CN:** 引入 `StrideShape` 等别名，以提升周围模板代码的可读性。

### Lines 633-634
```cpp
633:   /// Dilation ( MatrixShape<Height, Width> )
634:   using DilationShape = DilationShape_;
```
**EN:** Introduces aliases such as `DilationShape` to keep the surrounding template code readable.

**CN:** 引入 `DilationShape` 等别名，以提升周围模板代码的可读性。

### Lines 636-637
```cpp
636:   /// Activation Shape loaded by threadblock
637:   using ActivationShape = ActivationShape_;
```
**EN:** Introduces aliases such as `ActivationShape` to keep the surrounding template code readable.

**CN:** 引入 `ActivationShape` 等别名，以提升周围模板代码的可读性。

### Lines 639-640
```cpp
639:   /// Operand tag
640:   static cutlass::gemm::Operand const kOperand = cutlass::gemm::Operand::kA;
```
**EN:** Stores member state such as `kOperand` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `kOperand` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 642-643
```cpp
642:   /// Element type
643:   using Element = Element_;
```
**EN:** Introduces aliases such as `Element` to keep the surrounding template code readable.

**CN:** 引入 `Element` 等别名，以提升周围模板代码的可读性。

### Lines 645-646
```cpp
645:   /// Layout of policy
646:   using Layout = layout::RowMajor;
```
**EN:** Introduces aliases such as `Layout` to keep the surrounding template code readable.

**CN:** 引入 `Layout` 等别名，以提升周围模板代码的可读性。

### Lines 648-649
```cpp
648:   /// Decomposition of elements among threads
649:   using Policy = Policy_;
```
**EN:** Introduces aliases such as `Policy` to keep the surrounding template code readable.

**CN:** 引入 `Policy` 等别名，以提升周围模板代码的可读性。

### Lines 651-652
```cpp
651:   /// TensorRef type for loading element from a tensor
652:   using TensorRef = TensorRef<Element, Layout>;
```
**EN:** Introduces aliases such as `TensorRef` to keep the surrounding template code readable.

**CN:** 引入 `TensorRef` 等别名，以提升周围模板代码的可读性。

### Lines 654-655
```cpp
654:   /// Index type
655:   using Index = typename TensorRef::Index;
```
**EN:** Introduces aliases such as `Index` to keep the surrounding template code readable.

**CN:** 引入 `Index` 等别名，以提升周围模板代码的可读性。

### Lines 657-658
```cpp
657:   /// Long Index type
658:   using LongIndex = typename TensorRef::LongIndex;
```
**EN:** Introduces aliases such as `LongIndex` to keep the surrounding template code readable.

**CN:** 引入 `LongIndex` 等别名，以提升周围模板代码的可读性。

### Lines 660-661
```cpp
660:   /// Coordinate for an element in the tensor
661:   using TensorCoord = typename TensorRef::TensorCoord;
```
**EN:** Introduces aliases such as `TensorCoord` to keep the surrounding template code readable.

**CN:** 引入 `TensorCoord` 等别名，以提升周围模板代码的可读性。

### Lines 663-669
```cpp
663:   //
664:   // Derived quantities
665:   //
667:   static_assert(!(Shape::kRow % Policy::WarpShape::kRow),
668:                 "The warp-level GEMM M size must be divisible by the number of threads arranged "
669:                 "along the M dimension.");
```
**EN:** Defines function `static_assert` for this stage of the convolution workflow.

**CN:** 定义函数 `static_assert`，服务于卷积工作流的这一阶段。

### Lines 671-675
```cpp
671:   static_assert(Shape::kRow > 0, "Shape::kRow must be greater than zero.");
672:   static_assert(Shape::kColumn > 0, "Shape::kColumn must be greater than zero.");
673:   static_assert(Policy::WarpShape::kRow > 0, "Policy::WarpShape::kRow must be greater than zero.");
674:   static_assert(Shape::kRow / Policy::WarpShape::kRow > 0,
675:                 "Shape::kRow / Policy::WarpShape::kRow must be greater than zero.");
```
**EN:** Defines function `static_assert` for this stage of the convolution workflow.

**CN:** 定义函数 `static_assert`，服务于卷积工作流的这一阶段。

### Lines 677-679
```cpp
677:   // Activations loaded by threadblock
678:   static int const ThreadActivationShapeH = (ThreadOutputShape::kH - 1) * StrideShape::kRow +
679:                                             (FilterShape::kRow - 1) * DilationShape::kRow + 1;
```
**EN:** Defines compile-time constants such as compile-time constants that parameterize later logic.

**CN:** 定义 compile-time constants 等编译期常量，用来参数化后续逻辑。

### Lines 681-682
```cpp
681:   static int const ThreadActivationShapeW = (ThreadOutputShape::kW - 1) * StrideShape::kColumn +
682:                                             (FilterShape::kColumn - 1) * DilationShape::kColumn + 1;
```
**EN:** Defines compile-time constants such as compile-time constants that parameterize later logic.

**CN:** 定义 compile-time constants 等编译期常量，用来参数化后续逻辑。

### Lines 684-685
```cpp
684:   using ThreadActivationShape = cutlass::conv::
685:       TensorNHWCShape<1, ThreadActivationShapeH, ThreadActivationShapeW, ThreadOutputShape::kC>;
```
**EN:** Introduces aliases such as `ThreadActivationShape` to keep the surrounding template code readable.

**CN:** 引入 `ThreadActivationShape` 等别名，以提升周围模板代码的可读性。

### Lines 687-690
```cpp
687:   // Thread-level shape of a fragment
688:   using ThreadShape =
689:       MatrixShape<ThreadOutputShape::kNHW,
690:                   ThreadOutputShape::kC>;
```
**EN:** Introduces aliases such as `ThreadShape` to keep the surrounding template code readable.

**CN:** 引入 `ThreadShape` 等别名，以提升周围模板代码的可读性。

### Lines 692-693
```cpp
692:   static_assert(!(ThreadShape::kColumn % Policy::LaneMmaShape::kN),
693:                 "Thread-level GEMM must be divisible by Policy::LaneMmaShape.");
```
**EN:** Defines function `static_assert` for this stage of the convolution workflow.

**CN:** 定义函数 `static_assert`，服务于卷积工作流的这一阶段。

### Lines 695-697
```cpp
695:   /// Number of individual loads
696:   using Iterations =
697:       MatrixShape<ThreadShape::kRow, ThreadShape::kColumn / Policy::LaneMmaShape::kN>;
```
**EN:** Introduces aliases such as `Iterations` to keep the surrounding template code readable.

**CN:** 引入 `Iterations` 等别名，以提升周围模板代码的可读性。

### Lines 699-700
```cpp
699:   using ThreadTileCount = MatrixShape<ThreadBlockOutputShape::kH / ThreadOutputShape::kH,
700:                                       ThreadBlockOutputShape::kW / ThreadOutputShape::kW>;
```
**EN:** Introduces aliases such as `ThreadTileCount` to keep the surrounding template code readable.

**CN:** 引入 `ThreadTileCount` 等别名，以提升周围模板代码的可读性。

### Lines 702-703
```cpp
702:   /// Fragment object holding a thread's part of a tile
703:   using Fragment = Array<Element, ThreadShape::kCount>;
```
**EN:** Introduces aliases such as `Fragment` to keep the surrounding template code readable.

**CN:** 引入 `Fragment` 等别名，以提升周围模板代码的可读性。

### Lines 705-707
```cpp
705:  protected:
706:   /// Internal reference
707:   cutlass::TensorRef<Array<Element, Policy::LaneMmaShape::kN>, layout::RowMajor> ref_;
```
**EN:** Stores member state such as `Element` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `Element` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 709-712
```cpp
709:   Array<Element, Policy::LaneMmaShape::kN>
710:       activation[ThreadActivationShape::kH][ThreadActivationShape::kW][Iterations::kColumn];
711:   int iterator_r_;
712:   int iterator_s_;
```
**EN:** Stores member state such as `Element`, `iterator_r_`, `iterator_s_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `Element`, `iterator_r_`, `iterator_s_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 715-715
```cpp
715:   MatrixCoord lane_offset_;
```
**EN:** Stores member state such as `lane_offset_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `lane_offset_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 717-720
```cpp
717:  public:
718:   /// Default ctor constructs null iterator
719:   CUTLASS_HOST_DEVICE
720:   DepthwiseDirect2dConvSimtTileIterator() {}
```
**EN:** Provides constructor-style initialization for `DepthwiseDirect2dConvSimtTileIterator`.

**CN:** 为 `DepthwiseDirect2dConvSimtTileIterator` 提供构造式初始化逻辑。

### Lines 722-724
```cpp
722:   /// Constructor from TensorRef
723:   CUTLASS_HOST_DEVICE
724:   DepthwiseDirect2dConvSimtTileIterator(TensorRef ref, int lane_id) {
```
**EN:** Provides constructor-style initialization for `DepthwiseDirect2dConvSimtTileIterator`.

**CN:** 为 `DepthwiseDirect2dConvSimtTileIterator` 提供构造式初始化逻辑。

### Lines 725-726
```cpp
725:     // compute offset based on thread ID and lane layout
726:     typename Policy::LaneLayout lane_layout = Policy::get_lane_layout();
```
**EN:** Defines function `get_lane_layout` for this stage of the convolution workflow.

**CN:** 定义函数 `get_lane_layout`，服务于卷积工作流的这一阶段。

### Lines 728-729
```cpp
728:     // Set channel offset
729:     lane_offset_ = lane_layout.inverse(lane_id) * MatrixCoord(0, Policy::LaneMmaShape::kN);
```
**EN:** Defines function `inverse` for this stage of the convolution workflow.

**CN:** 定义函数 `inverse`，服务于卷积工作流的这一阶段。

### Lines 731-731
```cpp
731:     ref.add_coord_offset(lane_offset_);
```
**EN:** Defines function `add_coord_offset` for this stage of the convolution workflow.

**CN:** 定义函数 `add_coord_offset`，服务于卷积工作流的这一阶段。

### Lines 733-734
```cpp
733:     ref_.reset(reinterpret_cast<Array<Element, Policy::LaneMmaShape::kN> *>(ref.data()),
734:                ref.stride(0) / Policy::LaneMmaShape::kN);
```
**EN:** Defines function `reset` for this stage of the convolution workflow.

**CN:** 定义函数 `reset`，服务于卷积工作流的这一阶段。

### Lines 736-738
```cpp
736:     iterator_r_ = 0;
737:     iterator_s_ = 0;
738:   }
```
**EN:** Stores member state such as `iterator_r_`, `iterator_s_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `iterator_r_`, `iterator_s_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 740-745
```cpp
740:   /// Adds a pointer offset to internal pointer(s) to advance through memory
741:   CUTLASS_HOST_DEVICE
742:   DepthwiseDirect2dConvSimtTileIterator &add_pointer_offset(LongIndex offset) {
743:     ref_.add_pointer_offset(offset);
744:     return *this;
745:   }
```
**EN:** Computes and returns the value needed by the surrounding template logic.

**CN:** 计算并返回外围模板逻辑所需的值。

### Lines 747-750
```cpp
747:   /// Loads a fragment from memory at the location pointed to by the iterator.
748:   template <typename Params>
749:   CUTLASS_HOST_DEVICE void setup_initial_status(
750:       Params const &params) {
```
**EN:** Defines function `setup_initial_status` for this stage of the convolution workflow.

**CN:** 定义函数 `setup_initial_status`，服务于卷积工作流的这一阶段。

### Lines 752-757
```cpp
752:     // Get base HW offset of current threads
753:     int threadgroup = threadIdx.x / (ThreadBlockOutputShape::kC / ThreadOutputShape::kC);
754:     int base_h =
755:         (threadgroup / (ThreadTileCount::kColumn)) * ThreadOutputShape::kH * StrideShape::kRow;
756:     int base_w =
757:         (threadgroup % (ThreadTileCount::kColumn)) * ThreadOutputShape::kW * StrideShape::kColumn;
```
**EN:** Stores member state such as `base_h`, `base_w` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `base_h`, `base_w` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 759-765
```cpp
759:     CUTLASS_PRAGMA_UNROLL
760:     for (int h = 0; h < ThreadActivationShape::kH; ++h) {
761:       CUTLASS_PRAGMA_UNROLL
762:       for (int w = 0; w < ThreadActivationShape::kW; ++w) {
763:         CUTLASS_PRAGMA_UNROLL
764:         for (int col = 0; col < Iterations::kColumn; ++col) {
765:           int offset = (base_h + h) * ActivationShape::kW + (base_w + w);
```
**EN:** Iterates through the relevant tile or coordinate space and updates local state.

**CN:** 遍历相关的 tile 或坐标空间，并更新局部状态。

### Lines 767-772
```cpp
767:           void const *ptr = ref_.data() + ref_.offset({offset, col * Policy::WarpShape::kColumn});
768:           arch::shared_load(activation[h][w][col], ptr);
769:         }
770:       }
771:     }
772:   }
```
**EN:** Defines function `shared_load` for this stage of the convolution workflow.

**CN:** 定义函数 `shared_load`，服务于卷积工作流的这一阶段。

### Lines 774-776
```cpp
774:   /// Advances an iterator along logical dimensions of matrix in units of whole tiles
775:   CUTLASS_HOST_DEVICE
776:   DepthwiseDirect2dConvSimtTileIterator &add_tile_offset(TensorCoord const &coord) {
```
**EN:** Defines function `add_tile_offset` for this stage of the convolution workflow.

**CN:** 定义函数 `add_tile_offset`，服务于卷积工作流的这一阶段。

### Lines 777-781
```cpp
777:     // Set warp row and col start
778:     lane_offset_ =
779:         MatrixCoord({lane_offset_.row() + coord.row() * Shape::kRow, lane_offset_.column()});
780:     return *this;
781:   }
```
**EN:** Stores member state such as `lane_offset_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `lane_offset_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 783-789
```cpp
783:   /// Advances an iterator along logical dimensions of matrix in units of whole tiles
784:   CUTLASS_HOST_DEVICE
785:   void advance(int32_t pointer_offset) {
786:     ref_.reset(ref_.data() + pointer_offset / sizeof(Element) / Policy::LaneMmaShape::kN);
787:     iterator_s_ = 0;
788:     iterator_r_ = 0;
789:   }
```
**EN:** Advances iterator or pipeline state to the next logical position.

**CN:** 将迭代器或流水线状态推进到下一个逻辑位置。

### Lines 791-797
```cpp
791:   /// Advances the iterator along the advance dimension
792:   CUTLASS_HOST_DEVICE
793:   DepthwiseDirect2dConvSimtTileIterator &operator++() {
794:     ++iterator_s_;
795:     if (iterator_s_ < FilterShape::kColumn) {
796:       return *this;
797:     }
```
**EN:** Stores member state such as `iterator_s_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `iterator_s_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 799-799
```cpp
799:     iterator_s_ = 0;
```
**EN:** Stores member state such as `iterator_s_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `iterator_s_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 801-804
```cpp
801:     ++iterator_r_;
802:     if (iterator_r_ < FilterShape::kRow) {
803:       return *this;
804:     }
```
**EN:** Stores member state such as `iterator_r_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `iterator_r_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 806-808
```cpp
806:     iterator_r_ = 0;
807:     return *this;
808:   }
```
**EN:** Stores member state such as `iterator_r_` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `iterator_r_` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 810-812
```cpp
810:   /// Advances the iterator along the advance dimension
811:   CUTLASS_HOST_DEVICE
812:   DepthwiseDirect2dConvSimtTileIterator &operator--() {
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 813-815
```cpp
813:     // Do nothing
814:     return *this;
815:   }
```
**EN:** Returns a value that feeds the next stage of the implementation.

**CN:** 返回供下一阶段实现使用的值。

### Lines 817-821
```cpp
817:   /// Loads a fragment from memory at the location pointed to by the iterator. (vector loads)
818:   CUTLASS_HOST_DEVICE
819:   void load_with_pointer_offset(Fragment &frag, Index pointer_offset) const {
820:     Array<Element, Policy::LaneMmaShape::kN> *dst_ptr =
821:         reinterpret_cast<Array<Element, Policy::LaneMmaShape::kN> *>(&frag);
```
**EN:** Loads a fragment or tile from memory into registers or shared memory.

**CN:** 把片段或 tile 从内存加载到寄存器或共享内存。

### Lines 823-830
```cpp
823:     CUTLASS_PRAGMA_UNROLL
824:     for (int p = 0; p < ThreadOutputShape::kH; ++p) {
825:       CUTLASS_PRAGMA_UNROLL
826:       for (int q = 0; q < ThreadOutputShape::kW; ++q) {
827:         CUTLASS_PRAGMA_UNROLL
828:         for (int n = 0; n < Iterations::kColumn; ++n) {
829:           const int h = p * StrideShape::kRow + iterator_r_ * DilationShape::kRow;
830:           const int w = q * StrideShape::kColumn + iterator_s_ * DilationShape::kColumn;
```
**EN:** Stores member state such as `h`, `w` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `h`, `w` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 832-836
```cpp
832:           dst_ptr[n + q + p * ThreadOutputShape::kW] = activation[h][w][n];
833:         }
834:       }
835:     }
836:   }
```
**EN:** Stores member state such as `dst_ptr` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `dst_ptr` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 838-840
```cpp
838:   /// Loads a fragment from memory at the location pointed to by the iterator.
839:   CUTLASS_HOST_DEVICE
840:   void load(Fragment &frag) const { load_with_pointer_offset(frag, 0); }
```
**EN:** Loads a fragment or tile from memory into registers or shared memory.

**CN:** 把片段或 tile 从内存加载到寄存器或共享内存。

### Lines 842-844
```cpp
842:   /// Stores a fragment to memory at the location pointed to by the iterator
843:   CUTLASS_HOST_DEVICE
844:   void store_with_pointer_offset(Fragment const &frag, Index pointer_offset) const {
```
**EN:** Stores computed data back to memory.

**CN:** 将计算结果写回内存。

### Lines 845-846
```cpp
845:     // Do nothing at present.
846:   }
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 848-852
```cpp
848:   /// Stores a fragment to memory at the location pointed to by the iterator
849:   CUTLASS_HOST_DEVICE
850:   void store(Fragment const &frag, Index pointer_offset) const {
851:     store_with_pointer_offset(frag, 0);
852:   }
```
**EN:** Stores computed data back to memory.

**CN:** 将计算结果写回内存。

### Lines 854-855
```cpp
854:   CUTLASS_DEVICE
855:   void set_kgroup_index(int k_group) {
```
**EN:** Defines function `set_kgroup_index` for this stage of the convolution workflow.

**CN:** 定义函数 `set_kgroup_index`，服务于卷积工作流的这一阶段。

### Lines 856-858
```cpp
856:     // no operation here
857:   }
858: };
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 860-862
```cpp
860: } // namespace warp
861: } // namespace conv
862: } // namespace cutlass
```
**EN:** Closes the current scope and finishes the surrounding definition block.

**CN:** 关闭当前作用域，并结束外围定义块。

## Key Concepts / 关键概念
- **EN:** Main role: Describes the lane policy used by warp-level matrix multiply operators targeting SIMT instructions **CN:** 核心作用：提供面向 矩阵乘加 深度卷积 simt tile 迭代器 的 warp 级数学或变换辅助组件。
- **EN:** Key exported symbols include `DepthwiseMmaSimtTileIterator`, `DepthwiseDirect2dConvSimtTileIterator`, `Base`, `Shape`, `Element`, `Layout`. **CN:** 关键导出符号包括 `DepthwiseMmaSimtTileIterator`, `DepthwiseDirect2dConvSimtTileIterator`, `Base`, `Shape`, `Element`, `Layout`。
- **EN:** Heavy use of C++ templates encodes shapes, layouts, datatypes, and architecture choices. **CN:** 大量使用 C++ 模板来编码形状、布局、数据类型和架构选择。
- **EN:** Iterator logic maps convolution tensors onto tiled memory accesses and implicit-GEMM traversal. **CN:** 迭代器逻辑把卷积张量映射为分块内存访问和隐式 GEMM 遍历。
- **EN:** MMA composition is central: the file either defines math operators directly or prepares data for them. **CN:** MMA 组合是核心：该文件要么直接定义数学算子，要么为其准备数据。

## Dependencies / 依赖关系
### Direct Includes / 直接包含
- `cutlass/cutlass.h`
- `cutlass/array.h`
- `cutlass/tensor_ref.h`
- `cutlass/matrix_shape.h`
- `cutlass/conv/convolution.h`
- `cutlass/arch/memory_sm75.h`
- `cutlass/layout/matrix.h`
- `cutlass/gemm/gemm.h`
- `cutlass/gemm/warp/mma_simt_policy.h`
- `cutlass/gemm/warp/mma_simt_tile_iterator.h`

### Internal Relationships / 内部关系
- **EN:** Builds on CUTLASS GEMM shapes, policies, or operators. **CN:** 构建在 CUTLASS GEMM 的形状、策略或算子之上。
- **EN:** Relies on tensor or matrix layout helpers. **CN:** 依赖张量或矩阵布局辅助类型。
