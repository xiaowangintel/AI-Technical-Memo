# depthwise_direct_conv_params.h — Code Analysis / 代码分析

## Source / 来源
- **Path:** `include/cutlass/conv/threadblock/depthwise_direct_conv_params.h`
- **Purpose (EN):** Extracts the host-params objects into non-template code.
- **用途 (CN):** 定义 深度卷积 直接 conv 参数 使用的线程块配置辅助组件。

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
 31: /*! 
 32:   \file 
 33:   \brief Extracts the host-params objects into non-template code.
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

### Lines 38-38
```cpp
 38: #define TRACE_CONV_PARAMS_INITIALIZERS_ENABLED 0
```
**EN:** Defines helper macros or compile-time switches used by the header.

**CN:** 定义头文件内部使用的辅助宏或编译期开关。

### Lines 40-46
```cpp
 40: #include "cutlass/cutlass.h"
 41: #include "cutlass/fast_math.h"
 42: #include "cutlass/layout/tensor.h"
 43: #include "cutlass/layout/matrix.h"
 44: #include "cutlass/layout/pitch_linear.h"
 45: #include "cutlass/conv/convolution.h"
 46: #include "cutlass/conv/conv2d_problem_size.h"
```
**EN:** Imports direct dependencies used later in the file, including `cutlass.h`, `fast_math.h`, `tensor.h`, `matrix.h`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `cutlass.h`, `fast_math.h`, `tensor.h`, `matrix.h`。

### Lines 48-50
```cpp
 48: #if TRACE_CONV_PARAMS_INITIALIZERS_ENABLED
 49: #include <fstream>
 50: #endif
```
**EN:** Imports direct dependencies used later in the file, including `fstream`.

**CN:** 引入本文件后续实现要用到的直接依赖，例如 `fstream`。

### Lines 54-56
```cpp
 54: namespace cutlass {
 55: namespace conv {
 56: namespace threadblock {
```
**EN:** Opens the namespace scope `cutlass` that contains these definitions.

**CN:** 打开包含这些定义的命名空间作用域 `cutlass`。

### Lines 60-62
```cpp
 60: /// Parameters structure used for DepthwiseFpropActivationDirect2dConvTileAccessIteratorOptimized
 61: template<typename Layout_ = layout::TensorNHWC >
 62: struct Depthwise2dFpropDirectConvParams;
```
**EN:** Declares struct `Depthwise2dFpropDirectConvParams`. The nearby comment explains that it serves the surrounding depthwise2d forward-propagation direct conv parameters logic.

**CN:** 声明结构体 `Depthwise2dFpropDirectConvParams`，相邻注释说明它服务于周围的 depthwise2d 前向传播 直接 conv 参数 逻辑。

### Lines 64-66
```cpp
 64: /// Parameters structure used for DepthwiseFpropActivationDirect2dConvTileAccessIteratorFixedStrideDilation
 65: template<typename Layout_ = layout::TensorNHWC >
 66: struct Depthwise2dFpropDirectConvActivationIteratorFixedStrideDilationParams;
```
**EN:** Declares struct `Depthwise2dFpropDirectConvActivationIteratorFixedStrideDilationParams`. The nearby comment explains that it serves the surrounding depthwise2d forward-propagation direct conv activation iterator fixed stride dilation parameters logic.

**CN:** 声明结构体 `Depthwise2dFpropDirectConvActivationIteratorFixedStrideDilationParams`，相邻注释说明它服务于周围的 depthwise2d 前向传播 直接 conv 激活 迭代器 固定 stride dilation 参数 逻辑。

### Lines 68-70
```cpp
 68: /// Parameters structure used for DepthwiseFpropFilterDirectConvTileAccessIteratorOptimized
 69: template<typename Layout_ = layout::TensorNHWC >
 70: struct Depthwise2dFpropDirectConvFilterIteratorParams;
```
**EN:** Declares struct `Depthwise2dFpropDirectConvFilterIteratorParams`. The nearby comment explains that it serves the surrounding depthwise2d forward-propagation direct conv filter iterator parameters logic.

**CN:** 声明结构体 `Depthwise2dFpropDirectConvFilterIteratorParams`，相邻注释说明它服务于周围的 depthwise2d 前向传播 直接 conv 滤波器 迭代器 参数 逻辑。

### Lines 74-76
```cpp
 74: /// Parameters structure used for DepthwiseFpropActivationDirect2dConvTileAccessIteratorOptimized
 75: template<>
 76: struct Depthwise2dFpropDirectConvParams<layout::TensorNHWC> {
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 78-78
```cpp
 78:   using Layout = layout::TensorNHWC;
```
**EN:** Introduces aliases such as `Layout` to keep the surrounding template code readable.

**CN:** 引入 `Layout` 等别名，以提升周围模板代码的可读性。

### Lines 80-80
```cpp
 80:   Layout layout;
```
**EN:** Stores member state such as `layout` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `layout` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 82-85
```cpp
 82:   int32_t activation_tile_h;
 83:   int32_t activation_tile_w;
 84:   int32_t activation_tile_hw;
 85:   FastDivmod activation_tile_w_divmod;
```
**EN:** Stores member state such as `activation_tile_h`, `activation_tile_w`, `activation_tile_hw`, `activation_tile_w_divmod` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `activation_tile_h`, `activation_tile_w`, `activation_tile_hw`, `activation_tile_w_divmod` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 87-92
```cpp
 87:   int filter[2];
 88:   int stride[2];
 89:   int dilation[2];
 90:   int inc_next[2];
 91:   FastDivmod pq_divmod;
 92:   FastDivmod q_divmod;
```
**EN:** Stores member state such as `filter`, `stride`, `dilation`, `inc_next`, `pq_divmod`, `q_divmod` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `filter`, `stride`, `dilation`, `inc_next`, `pq_divmod`, `q_divmod` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 94-96
```cpp
 94:   int activation_load_count;
 95:   int activation_storage_elements;
 96:   int activation_size;
```
**EN:** Stores member state such as `activation_load_count`, `activation_storage_elements`, `activation_size` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `activation_load_count`, `activation_storage_elements`, `activation_size` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 101-102
```cpp
101:   CUTLASS_HOST_DEVICE
102:   Depthwise2dFpropDirectConvParams() { }
```
**EN:** Provides constructor-style initialization for `Depthwise2dFpropDirectConvParams`.

**CN:** 为 `Depthwise2dFpropDirectConvParams` 提供构造式初始化逻辑。

### Lines 104-114
```cpp
104:   CUTLASS_HOST_DEVICE
105:   Depthwise2dFpropDirectConvParams(
106:       Conv2dProblemSize const &problem_size,
107:       Layout const &layout,             ///< layout object
108:       MatrixCoord threadblock_shape,    ///< CTA threadblock Shape
109:       Layout::TensorCoord threadblock_output_shape,  ///< Output tile Shape per threadblock
110:       const int element_size_bits,      ///< bits of activation element
111:       const int thread_count,           ///< threads per threadblock
112:       const int thread_count_contiguous, ///< number of threads for continuous dimension
113:       const int element_per_load)       ///< element per each load
114:       : layout(layout) {
```
**EN:** Provides constructor-style initialization for `Depthwise2dFpropDirectConvParams`.

**CN:** 为 `Depthwise2dFpropDirectConvParams` 提供构造式初始化逻辑。

### Lines 116-117
```cpp
116:     filter[0] = problem_size.S;
117:     filter[1] = problem_size.R;
```
**EN:** Stores member state such as `filter` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `filter` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 119-120
```cpp
119:     stride[0] =  problem_size.stride_w;
120:     stride[1] =  problem_size.stride_h;
```
**EN:** Stores member state such as `stride` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `stride` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 122-123
```cpp
122:     dilation[0] = problem_size.dilation_w;
123:     dilation[1] = problem_size.dilation_h;
```
**EN:** Stores member state such as `dilation` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `dilation` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 125-130
```cpp
125:     // Compute activation_tile size per threadblock because stride and dilation are runtime params.
126:     activation_tile_h = (threadblock_output_shape.h() - 1) * problem_size.stride_h +
127:                         (problem_size.R - 1) * problem_size.dilation_h + 1;
128:     activation_tile_w = (threadblock_output_shape.w() - 1) * problem_size.stride_w +
129:                         (problem_size.S - 1) * problem_size.dilation_w + 1;
130:     activation_tile_hw = activation_tile_h * activation_tile_w;
```
**EN:** Stores member state such as `activation_tile_hw` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `activation_tile_hw` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 132-132
```cpp
132:     activation_tile_w_divmod = FastDivmod(activation_tile_w);
```
**EN:** Provides constructor-style initialization for `FastDivmod`.

**CN:** 为 `FastDivmod` 提供构造式初始化逻辑。

### Lines 134-137
```cpp
134:     /// Below two values could not be templatized because the stride and dilation are runtime params
135:     activation_load_count = (thread_count_contiguous * activation_tile_hw + (thread_count - 1)) / thread_count;
136:     activation_storage_elements = activation_load_count * element_per_load * thread_count;
137:     activation_size =  activation_storage_elements * element_size_bits / 8;
```
**EN:** Stores member state such as `activation_storage_elements`, `activation_size` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `activation_storage_elements`, `activation_size` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 139-143
```cpp
139:     // Fastdivmod for output P, Q
140:     int tiles_p =
141:         (problem_size.P + (threadblock_output_shape.h() - 1)) / (threadblock_output_shape.h());
142:     int tiles_q = (problem_size.Q + (threadblock_output_shape.w() - 1)) /
143:                   (threadblock_output_shape.w());
```
**EN:** Defines function `h` for this stage of the convolution workflow.

**CN:** 定义函数 `h`，服务于卷积工作流的这一阶段。

### Lines 145-146
```cpp
145:     pq_divmod = FastDivmod(tiles_p * tiles_q);
146:     q_divmod = FastDivmod(tiles_q);
```
**EN:** Provides constructor-style initialization for `FastDivmod`.

**CN:** 为 `FastDivmod` 提供构造式初始化逻辑。

### Lines 148-149
```cpp
148:     // next S
149:     inc_next[0] = problem_size.dilation_w;
```
**EN:** Stores member state such as `inc_next` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `inc_next` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 150-153
```cpp
150:     // next R
151:     inc_next[1] = (activation_tile_w * problem_size.dilation_h - (problem_size.S - 1) * problem_size.dilation_w);
152:   }
153: };
```
**EN:** Continues the template implementation by wiring helper logic into the surrounding definition.

**CN:** 继续补充模板实现，把辅助逻辑接入外围定义。

### Lines 156-159
```cpp
156: /// Parameters structure used for DepthwiseFpropActivationDirect2dConvTileAccessIteratorFixedStrideDilation
157: template <>
158: struct Depthwise2dFpropDirectConvActivationIteratorFixedStrideDilationParams<layout::TensorNHWC> {
159:   using Layout = layout::TensorNHWC;
```
**EN:** Introduces aliases such as `Layout` to keep the surrounding template code readable.

**CN:** 引入 `Layout` 等别名，以提升周围模板代码的可读性。

### Lines 161-161
```cpp
161:   Layout layout;
```
**EN:** Stores member state such as `layout` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `layout` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 163-164
```cpp
163:   FastDivmod pq_divmod;
164:   FastDivmod q_divmod;
```
**EN:** Stores member state such as `pq_divmod`, `q_divmod` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `pq_divmod`, `q_divmod` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 166-166
```cpp
166:   int activation_size;
```
**EN:** Stores member state such as `activation_size` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `activation_size` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 172-173
```cpp
172:   CUTLASS_HOST_DEVICE
173:   Depthwise2dFpropDirectConvActivationIteratorFixedStrideDilationParams() {}
```
**EN:** Provides constructor-style initialization for `Depthwise2dFpropDirectConvActivationIteratorFixedStrideDilationParams`.

**CN:** 为 `Depthwise2dFpropDirectConvActivationIteratorFixedStrideDilationParams` 提供构造式初始化逻辑。

### Lines 175-184
```cpp
175:   CUTLASS_HOST_DEVICE
176:   Depthwise2dFpropDirectConvActivationIteratorFixedStrideDilationParams(
177:       Conv2dProblemSize const &problem_size,
178:       Layout const &layout,                          ///< Layout object
179:       MatrixCoord threadblock_shape,                 ///< Threadblock Shape
180:       Layout::TensorCoord threadblock_output_shape,  ///< Output tile Shape per threadblock
181:       const int activation_size_                     ///< Activation size loaded by iterator
182:       )
183:       : layout(layout),
184:         activation_size(activation_size_) {
```
**EN:** Provides constructor-style initialization for `Depthwise2dFpropDirectConvActivationIteratorFixedStrideDilationParams`.

**CN:** 为 `Depthwise2dFpropDirectConvActivationIteratorFixedStrideDilationParams` 提供构造式初始化逻辑。

### Lines 185-189
```cpp
185:     // Fastdivmod for output P, Q
186:     int tiles_p =
187:         (problem_size.P + (threadblock_output_shape.h() - 1)) / (threadblock_output_shape.h());
188:     int tiles_q =
189:         (problem_size.Q + (threadblock_output_shape.w() - 1)) / (threadblock_output_shape.w());
```
**EN:** Defines function `h` for this stage of the convolution workflow.

**CN:** 定义函数 `h`，服务于卷积工作流的这一阶段。

### Lines 191-194
```cpp
191:     pq_divmod = FastDivmod(tiles_p * tiles_q);
192:     q_divmod = FastDivmod(tiles_q);
193:   }
194: };
```
**EN:** Provides constructor-style initialization for `FastDivmod`.

**CN:** 为 `FastDivmod` 提供构造式初始化逻辑。

### Lines 198-201
```cpp
198: /// Parameters structure used for DepthwiseFpropFilterDirectConvTileAccessIteratorOptimized
199: template <>
200: struct Depthwise2dFpropDirectConvFilterIteratorParams<layout::TensorNHWC> {
201:   using Layout = layout::TensorNHWC;
```
**EN:** Introduces aliases such as `Layout` to keep the surrounding template code readable.

**CN:** 引入 `Layout` 等别名，以提升周围模板代码的可读性。

### Lines 203-203
```cpp
203:   Layout layout;
```
**EN:** Stores member state such as `layout` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `layout` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 205-205
```cpp
205:   int filter_size;
```
**EN:** Stores member state such as `filter_size` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `filter_size` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 207-207
```cpp
207:   bool is_convolution;
```
**EN:** Stores member state such as `is_convolution` for later iterator, pipeline, or kernel steps.

**CN:** 保存 `is_convolution` 等成员状态，供后续迭代器、流水线或内核步骤使用。

### Lines 212-213
```cpp
212:   CUTLASS_HOST_DEVICE
213:   Depthwise2dFpropDirectConvFilterIteratorParams() {}
```
**EN:** Provides constructor-style initialization for `Depthwise2dFpropDirectConvFilterIteratorParams`.

**CN:** 为 `Depthwise2dFpropDirectConvFilterIteratorParams` 提供构造式初始化逻辑。

### Lines 215-224
```cpp
215:   CUTLASS_HOST_DEVICE
216:   Depthwise2dFpropDirectConvFilterIteratorParams(
217:       Conv2dProblemSize const &problem_size,
218:       Layout const &layout,           ///< Layout object
219:       MatrixCoord threadblock_shape,  ///< Threadblock Shape
220:       const int filter_size_)         ///< Filter size loaded by iterator
221:       : layout(layout),
222:         filter_size(filter_size_),
223:         is_convolution(problem_size.mode == Mode::kConvolution){}
224: };
```
**EN:** Provides constructor-style initialization for `Depthwise2dFpropDirectConvFilterIteratorParams`.

**CN:** 为 `Depthwise2dFpropDirectConvFilterIteratorParams` 提供构造式初始化逻辑。

### Lines 226-228
```cpp
226: }  // namespace threadblock
227: }  // namespace conv
228: }  // namespace cutlass
```
**EN:** Opens the namespace scope `threadblock` that contains these definitions.

**CN:** 打开包含这些定义的命名空间作用域 `threadblock`。

## Key Concepts / 关键概念
- **EN:** Main role: Extracts the host-params objects into non-template code. **CN:** 核心作用：定义 深度卷积 直接 conv 参数 使用的线程块配置辅助组件。
- **EN:** Key exported symbols include `Depthwise2dFpropDirectConvParams`, `Depthwise2dFpropDirectConvActivationIteratorFixedStrideDilationParams`, `Depthwise2dFpropDirectConvFilterIteratorParams`, `Layout`. **CN:** 关键导出符号包括 `Depthwise2dFpropDirectConvParams`, `Depthwise2dFpropDirectConvActivationIteratorFixedStrideDilationParams`, `Depthwise2dFpropDirectConvFilterIteratorParams`, `Layout`。
- **EN:** Heavy use of C++ templates encodes shapes, layouts, datatypes, and architecture choices. **CN:** 大量使用 C++ 模板来编码形状、布局、数据类型和架构选择。
- **EN:** Iterator logic maps convolution tensors onto tiled memory accesses and implicit-GEMM traversal. **CN:** 迭代器逻辑把卷积张量映射为分块内存访问和隐式 GEMM 遍历。
- **EN:** Precomputed parameter objects reduce runtime address arithmetic in hot loops. **CN:** 预计算参数对象可以减少热点循环中的运行时地址计算。

## Dependencies / 依赖关系
### Direct Includes / 直接包含
- `cutlass/cutlass.h`
- `cutlass/fast_math.h`
- `cutlass/layout/tensor.h`
- `cutlass/layout/matrix.h`
- `cutlass/layout/pitch_linear.h`
- `cutlass/conv/convolution.h`
- `cutlass/conv/conv2d_problem_size.h`
- `fstream`

### Internal Relationships / 内部关系
- **EN:** Relies on tensor or matrix layout helpers. **CN:** 依赖张量或矩阵布局辅助类型。
