# tensor_compare.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `tools/util/include/cutlass/util/reference/device/tensor_compare.h`
- **Purpose (EN):** This file declares tensor utilities for device-side reference utilities.
- **目的 (CN):** 该文件声明了面向设备端参考工具的张量工具逻辑。
- **Brief / 简述:** Defines host-side elementwise operations on TensorView.

## Line-by-Line Analysis / 逐行分析
### Lines 1-20
```cpp
1: /***************************************************************************************************
2:  * Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
3:  * Copyright (C) 2025 Intel Corporation, All rights reserved.
4:  * SPDX-License-Identifier: BSD-3-Clause
5:  *
6:  * Redistribution and use in source and binary forms, with or without
7:  * modification, are permitted provided that the following conditions are met:
8:  *
9:  * 1. Redistributions of source code must retain the above copyright notice, this
10:  * list of conditions and the following disclaimer.
11:  *
12:  * 2. Redistributions in binary form must reproduce the above copyright notice,
13:  * this list of conditions and the following disclaimer in the documentation
14:  * and/or other materials provided with the distribution.
15:  *
16:  * 3. Neither the name of the copyright holder nor the names of its
17:  * contributors may be used to endorse or promote products derived from
18:  * this software without specific prior written permission.
19:  *
20:  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
```
- **EN:** License header and copyright terms for the file.
- **CN:** 文件的许可证头和版权条款。

### Lines 21-31
```cpp
21:  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
22:  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
23:  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
24:  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
25:  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
26:  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
27:  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
28:  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
29:  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
30:  *
31:  **************************************************************************************************/
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 32-34
```cpp
32: /* \file
33:   \brief Defines host-side elementwise operations on TensorView.
34: */
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 36-36
```cpp
36: #pragma once
```
- **EN:** Uses include guards or pragmas to ensure the declarations are processed only once.
- **CN:** 使用 include guard 或 pragma，确保这些声明只被处理一次。

### Lines 37-37
```cpp
37: // Standard Library includes
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 38-38
```cpp
38: #include <utility>
```
- **EN:** Includes the main dependencies required by this module; notable headers are `utility`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `utility`。

### Lines 40-40
```cpp
40: // Cutlass includes
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 41-42
```cpp
41: #include "cutlass/cutlass.h"
42: #include "cutlass/relatively_equal.h"
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cutlass/cutlass.h`, `cutlass/relatively_equal.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cutlass/cutlass.h`, `cutlass/relatively_equal.h`。

### Lines 44-44
```cpp
44: #include "cutlass/util/distribution.h"
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cutlass/util/distribution.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cutlass/util/distribution.h`。

### Lines 46-46
```cpp
46: #include "tensor_foreach.h"
```
- **EN:** Includes the main dependencies required by this module; notable headers are `tensor_foreach.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `tensor_foreach.h`。

### Lines 48-50
```cpp
48: namespace cutlass {
49: namespace reference {
50: namespace device {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 52-52
```cpp
52: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 54-54
```cpp
54: namespace kernel {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 56-56
```cpp
56: template <typename Element>
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 57-57
```cpp
57: #if defined (CUTLASS_ENABLE_SYCL)
```
- **EN:** Conditional-compilation or macro block keyed on `defined`.
- **CN:** 以 `defined` 为条件的条件编译或宏定义代码块。

### Lines 58-58
```cpp
58: void
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 59-59
```cpp
59: #else
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 60-60
```cpp
60: __global__ void
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 61-61
```cpp
61: #endif
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 62-66
```cpp
62:  BlockCompareEqual(
63:   int *equal, 
64:   Element const *ptr_A,
65:   Element const *ptr_B,
66:   size_t capacity) {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 68-68
```cpp
68:   size_t idx = ThreadIdxX() + BlockDimX() * BlockIdxX();
```
- **EN:** Implements `ThreadIdxX` and coordinates helper calls such as `BlockDimX`, `BlockIdxX`.
- **CN:** 实现 `ThreadIdxX`，并协调调用 `BlockDimX`, `BlockIdxX` 等辅助逻辑。

### Lines 70-70
```cpp
70:   for (; idx < capacity; idx += GridDimX() * BlockDimX()) {
```
- **EN:** Declares or updates local/member state such as `capacity`.
- **CN:** 声明或更新局部/成员状态，例如 `capacity`。

### Lines 72-73
```cpp
72:     Element a = cutlass::ReferenceFactory<Element>::get(ptr_A, idx);
73:     Element b = cutlass::ReferenceFactory<Element>::get(ptr_B, idx);
```
- **EN:** Implements `get` for this file's main component.
- **CN:** 为该文件的核心组件实现 `get`。

### Lines 75-76
```cpp
75:     if (a != b) {
76:       *equal = 0;
```
- **EN:** Declares or updates local/member state such as `equal`.
- **CN:** 声明或更新局部/成员状态，例如 `equal`。

### Lines 78-81
```cpp
78:       return;
79:     }
80:   }
81: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 83-83
```cpp
83: template <typename Element>
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 84-84
```cpp
84: #if defined (CUTLASS_ENABLE_SYCL)
```
- **EN:** Conditional-compilation or macro block keyed on `defined`.
- **CN:** 以 `defined` 为条件的条件编译或宏定义代码块。

### Lines 85-85
```cpp
85: void
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 86-86
```cpp
86: #else
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 87-87
```cpp
87: __global__ void
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 88-88
```cpp
88: #endif
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 89-95
```cpp
89:  BlockCompareRelativelyEqual(
90:   int *equal, 
91:   Element const *ptr_A,
92:   Element const *ptr_B,
93:   size_t capacity,
94:   Element epsilon,
95:   Element nonzero_floor) {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 97-97
```cpp
97:   size_t idx = ThreadIdxX() + BlockDimX() * BlockIdxX();
```
- **EN:** Implements `ThreadIdxX` and coordinates helper calls such as `BlockDimX`, `BlockIdxX`.
- **CN:** 实现 `ThreadIdxX`，并协调调用 `BlockDimX`, `BlockIdxX` 等辅助逻辑。

### Lines 99-99
```cpp
99:   for (; idx < capacity; idx += GridDimX() * BlockDimX()) {
```
- **EN:** Declares or updates local/member state such as `capacity`.
- **CN:** 声明或更新局部/成员状态，例如 `capacity`。

### Lines 101-102
```cpp
101:     Element a = cutlass::ReferenceFactory<Element>::get(ptr_A, idx);
102:     Element b = cutlass::ReferenceFactory<Element>::get(ptr_B, idx);
```
- **EN:** Implements `get` for this file's main component.
- **CN:** 为该文件的核心组件实现 `get`。

### Lines 104-104
```cpp
104:     if (!relatively_equal(a, b, epsilon, nonzero_floor)) {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 105-105
```cpp
105: #ifdef SHOW_DIFF
```
- **EN:** Conditional-compilation or macro block keyed on `SHOW_DIFF`.
- **CN:** 以 `SHOW_DIFF` 为条件的条件编译或宏定义代码块。

### Lines 106-106
```cpp
106:       printf("[%zu]: %f vs %f\n", idx, (double) a, (double) b);
```
- **EN:** Emits debug or diagnostic output so intermediate state can be inspected.
- **CN:** 输出调试或诊断信息，便于检查中间状态。

### Lines 107-107
```cpp
107: #endif
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 108-112
```cpp
108:       *equal = 0;
109:       return;
110:     }
111:   }
112: }
```
- **EN:** Declares or updates local/member state such as `equal`.
- **CN:** 声明或更新局部/成员状态，例如 `equal`。

### Lines 114-114
```cpp
114: template <template <class> class BinaryOp, typename Element>
```
- **EN:** Introduces `BinaryOp`, a type used to support tensor utilities.
- **CN:** 引入 `BinaryOp`，即一个用于支持张量工具的类型。

### Lines 115-115
```cpp
115: #if defined (CUTLASS_ENABLE_SYCL)
```
- **EN:** Conditional-compilation or macro block keyed on `defined`.
- **CN:** 以 `defined` 为条件的条件编译或宏定义代码块。

### Lines 116-116
```cpp
116: void
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 117-117
```cpp
117: #else
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 118-118
```cpp
118: __global__ void
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 119-119
```cpp
119: #endif
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 120-124
```cpp
120: BlockElementwiseOp(
121:   Element *ptr_dst,
122:   Element const *ptr_A,
123:   Element const *ptr_B,
124:   size_t capacity) {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 126-126
```cpp
126:   BinaryOp<Element> bin_op{};
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 128-132
```cpp
128:   const size_t start = ThreadIdxX() + BlockDimX() * BlockIdxX();
129:   const size_t step = GridDimX() * BlockDimX();
130:   for (size_t idx = start; idx < capacity; idx += step) {
131:     Element a = cutlass::ReferenceFactory<Element>::get(ptr_A, idx);
132:     Element b = cutlass::ReferenceFactory<Element>::get(ptr_B, idx);
```
- **EN:** Implements `ThreadIdxX` and coordinates helper calls such as `BlockDimX`, `BlockIdxX`, `GridDimX`.
- **CN:** 实现 `ThreadIdxX`，并协调调用 `BlockDimX`, `BlockIdxX`, `GridDimX` 等辅助逻辑。

### Lines 134-136
```cpp
134:     cutlass::ReferenceFactory<Element>::get(ptr_dst, idx) = bin_op(a, b);
135:   }
136: }
```
- **EN:** Implements `get` and coordinates helper calls such as `bin_op`.
- **CN:** 实现 `get`，并协调调用 `bin_op` 等辅助逻辑。

### Lines 138-138
```cpp
138: } // namespace kernel
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 141-141
```cpp
141: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 143-143
```cpp
143: template<class> class BlockCompareEqualKernelName;
```
- **EN:** Introduces `BlockCompareEqualKernelName`, a type used to support tensor utilities.
- **CN:** 引入 `BlockCompareEqualKernelName`，即一个用于支持张量工具的类型。

### Lines 145-145
```cpp
145: /// Performs a bit-level equality check between two blocks
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 146-153
```cpp
146: template <typename Element>
147: bool BlockCompareEqual(
148:   Element const *ptr_A,
149:   Element const *ptr_B,
150:   size_t capacity,
151:   int grid_size = 0, 
152:   int block_size = 0,
153:   cudaStream_t stream = nullptr) {
```
- **EN:** Declares or updates local/member state such as `grid_size`, `block_size`, `stream`.
- **CN:** 声明或更新局部/成员状态，例如 `grid_size`, `block_size`, `stream`。

### Lines 155-156
```cpp
155:   int equal_flag = 1;
156:   int *device_equal_flag = nullptr;
```
- **EN:** Declares or updates local/member state such as `equal_flag`, `device_equal_flag`, `nullptr`.
- **CN:** 声明或更新局部/成员状态，例如 `equal_flag`, `device_equal_flag`, `nullptr`。

### Lines 158-158
```cpp
158: #if defined (CUTLASS_ENABLE_SYCL)
```
- **EN:** Conditional-compilation or macro block keyed on `defined`.
- **CN:** 以 `defined` 为条件的条件编译或宏定义代码块。

### Lines 159-163
```cpp
159:   device_equal_flag = reinterpret_cast<int*>(compat::malloc(sizeof(int)));
160:   if (device_equal_flag == nullptr) {
161:     throw std::runtime_error("Failed to allocate device flag.");
162:   }
163:   compat::memcpy(device_equal_flag, &equal_flag, sizeof(int));
```
- **EN:** Implements `malloc` and coordinates helper calls such as `runtime_error`, `memcpy`.
- **CN:** 实现 `malloc`，并协调调用 `runtime_error`, `memcpy` 等辅助逻辑。

### Lines 164-164
```cpp
164: #else
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 165-167
```cpp
165:   if (cudaMalloc((void **)&device_equal_flag, sizeof(int)) != cudaSuccess) {
166:     throw std::runtime_error("Failed to allocate device flag.");
167:   }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 169-173
```cpp
169:   if (cudaMemcpy(
170:     device_equal_flag, 
171:     &equal_flag, 
172:     sizeof(int), 
173:     cudaMemcpyHostToDevice) != cudaSuccess) {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 175-176
```cpp
175:     throw std::runtime_error("Failed to copy equality flag to device.");
176:   }
```
- **EN:** Implements `runtime_error` for this file's main component.
- **CN:** 为该文件的核心组件实现 `runtime_error`。

### Lines 177-177
```cpp
177: #endif
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 179-179
```cpp
179:   if (!grid_size || !block_size) {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 180-180
```cpp
180: #if defined (CUTLASS_ENABLE_SYCL)
```
- **EN:** Conditional-compilation or macro block keyed on `defined`.
- **CN:** 以 `defined` 为条件的条件编译或宏定义代码块。

### Lines 181-183
```cpp
181:     block_size = 128;
182:     grid_size = (capacity + block_size - 1) / block_size;
183:     grid_size = (grid_size < 64 ? grid_size : 64); // limit grid size to avoid out_of_resources runtime error.
```
- **EN:** Declares or updates local/member state such as `block_size`, `grid_size`.
- **CN:** 声明或更新局部/成员状态，例如 `block_size`, `grid_size`。

### Lines 184-184
```cpp
184: #else
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 185-185
```cpp
185:     // if grid_size or block_size are zero, query occupancy using the CUDA Occupancy API
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 186-189
```cpp
186:     cudaError_t result = cudaOccupancyMaxPotentialBlockSize(
187:       &grid_size,
188:       &block_size,
189:       reinterpret_cast<void const *>(kernel::BlockCompareEqual<Element>));
```
- **EN:** Implements `cudaOccupancyMaxPotentialBlockSize` for this file's main component.
- **CN:** 为该文件的核心组件实现 `cudaOccupancyMaxPotentialBlockSize`。

### Lines 191-193
```cpp
191:     if (result != cudaSuccess) {
192:       throw std::runtime_error("Failed to query occupancy.");
193:     }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 194-195
```cpp
194:     // Limit block size. This has the effect of increasing the number of items processed by a
195:     // single thread and reduces the impact of initialization overhead.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 196-196
```cpp
196:     block_size = (block_size < 128 ? block_size : 128);
```
- **EN:** Declares or updates local/member state such as `block_size`.
- **CN:** 声明或更新局部/成员状态，例如 `block_size`。

### Lines 197-197
```cpp
197: #endif
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 198-198
```cpp
198:   }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 200-200
```cpp
200: #if defined(CUTLASS_ENABLE_SYCL)
```
- **EN:** Conditional-compilation or macro block keyed on `defined(CUTLASS_ENABLE_SYCL)`.
- **CN:** 以 `defined(CUTLASS_ENABLE_SYCL)` 为条件的条件编译或宏定义代码块。

### Lines 201-204
```cpp
201:   const auto sycl_block = compat::dim3(block_size, 1, 1);
202:   const auto sycl_grid = compat::dim3(grid_size, 1, 1);
203:   compat::launch<kernel::BlockCompareEqual<Element>, BlockCompareEqualKernelName<Element>>(sycl_grid, sycl_block, device_equal_flag, ptr_A, ptr_B, capacity);
204:   compat::wait();
```
- **EN:** Implements `dim3` and coordinates helper calls such as `wait`.
- **CN:** 实现 `dim3`，并协调调用 `wait` 等辅助逻辑。

### Lines 206-206
```cpp
206:   compat::memcpy(&equal_flag, device_equal_flag, sizeof(int));
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 208-208
```cpp
208:   compat::free(reinterpret_cast<void*>(device_equal_flag));
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 209-209
```cpp
209: #else
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 210-211
```cpp
210:   dim3 grid(grid_size, 1, 1);
211:   dim3 block(block_size, 1, 1);
```
- **EN:** Implements `grid` and coordinates helper calls such as `block`.
- **CN:** 实现 `grid`，并协调调用 `block` 等辅助逻辑。

### Lines 213-213
```cpp
213:   kernel::BlockCompareEqual<Element><<< grid, block, 0, stream >>>(device_equal_flag, ptr_A, ptr_B, capacity);
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 215-215
```cpp
215:   cudaStreamSynchronize(stream);
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 217-221
```cpp
217:   if (cudaMemcpy(
218:     &equal_flag, 
219:     device_equal_flag,
220:     sizeof(int), 
221:     cudaMemcpyDeviceToHost) != cudaSuccess) {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 223-223
```cpp
223:     cudaFree(device_equal_flag);
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 225-226
```cpp
225:     throw std::runtime_error("Failed to copy equality flag from device.");
226:   }
```
- **EN:** Implements `runtime_error` for this file's main component.
- **CN:** 为该文件的核心组件实现 `runtime_error`。

### Lines 228-228
```cpp
228:   cudaFree(device_equal_flag);
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 229-229
```cpp
229: #endif
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 231-232
```cpp
231:   return equal_flag;
232: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 234-234
```cpp
234: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 236-236
```cpp
236: template<class> class BlockCompareRelativelyEqualName;
```
- **EN:** Introduces `BlockCompareRelativelyEqualName`, a type used to support tensor utilities.
- **CN:** 引入 `BlockCompareRelativelyEqualName`，即一个用于支持张量工具的类型。

### Lines 238-238
```cpp
238: /// Performs a bit-level equality check between two blocks
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 239-248
```cpp
239: template <typename Element>
240: bool BlockCompareRelativelyEqual(
241:   Element const *ptr_A,
242:   Element const *ptr_B,
243:   size_t capacity,
244:   Element epsilon,
245:   Element nonzero_floor,
246:   int grid_size = 0, 
247:   int block_size = 0,
248:   cudaStream_t stream = nullptr) {
```
- **EN:** Declares or updates local/member state such as `grid_size`, `block_size`, `stream`.
- **CN:** 声明或更新局部/成员状态，例如 `grid_size`, `block_size`, `stream`。

### Lines 250-251
```cpp
250:   int equal_flag = 1;
251:   int *device_equal_flag = nullptr;
```
- **EN:** Declares or updates local/member state such as `equal_flag`, `device_equal_flag`, `nullptr`.
- **CN:** 声明或更新局部/成员状态，例如 `equal_flag`, `device_equal_flag`, `nullptr`。

### Lines 253-253
```cpp
253: #if defined (CUTLASS_ENABLE_SYCL)
```
- **EN:** Conditional-compilation or macro block keyed on `defined`.
- **CN:** 以 `defined` 为条件的条件编译或宏定义代码块。

### Lines 254-258
```cpp
254:   device_equal_flag = reinterpret_cast<int*>(compat::malloc(sizeof(int)));
255:   if (device_equal_flag == nullptr) {
256:     throw std::runtime_error("Failed to allocate device flag.");
257:   }
258:   compat::memcpy(device_equal_flag, &equal_flag, sizeof(int));
```
- **EN:** Implements `malloc` and coordinates helper calls such as `runtime_error`, `memcpy`.
- **CN:** 实现 `malloc`，并协调调用 `runtime_error`, `memcpy` 等辅助逻辑。

### Lines 259-259
```cpp
259: #else
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 260-262
```cpp
260:   if (cudaMalloc((void **)&device_equal_flag, sizeof(int)) != cudaSuccess) {
261:     throw std::runtime_error("Failed to allocate device flag.");
262:   }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 264-268
```cpp
264:   if (cudaMemcpy(
265:     device_equal_flag, 
266:     &equal_flag, 
267:     sizeof(int), 
268:     cudaMemcpyHostToDevice) != cudaSuccess) {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 270-271
```cpp
270:     throw std::runtime_error("Failed to copy equality flag to device.");
271:   }
```
- **EN:** Implements `runtime_error` for this file's main component.
- **CN:** 为该文件的核心组件实现 `runtime_error`。

### Lines 272-272
```cpp
272: #endif
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 274-274
```cpp
274:   if (!grid_size || !block_size) {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 275-275
```cpp
275: #if defined (CUTLASS_ENABLE_SYCL)
```
- **EN:** Conditional-compilation or macro block keyed on `defined`.
- **CN:** 以 `defined` 为条件的条件编译或宏定义代码块。

### Lines 276-278
```cpp
276:     block_size = 128;
277:     grid_size = (capacity + block_size - 1) / block_size;
278:     grid_size = (grid_size < 64 ? grid_size : 64); // limit grid size to avoid out_of_resources runtime error.
```
- **EN:** Declares or updates local/member state such as `block_size`, `grid_size`.
- **CN:** 声明或更新局部/成员状态，例如 `block_size`, `grid_size`。

### Lines 279-279
```cpp
279: #else
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 280-280
```cpp
280:     // if grid_size or block_size are zero, query occupancy using the CUDA Occupancy API
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 281-284
```cpp
281:     cudaError_t result = cudaOccupancyMaxPotentialBlockSize(
282:       &grid_size,
283:       &block_size,
284:       reinterpret_cast<void const *>(kernel::BlockCompareRelativelyEqual<Element>));
```
- **EN:** Implements `cudaOccupancyMaxPotentialBlockSize` for this file's main component.
- **CN:** 为该文件的核心组件实现 `cudaOccupancyMaxPotentialBlockSize`。

### Lines 286-288
```cpp
286:     if (result != cudaSuccess) {
287:       throw std::runtime_error("Failed to query occupancy.");
288:     }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 289-290
```cpp
289:     // Limit block size. This has the effect of increasing the number of items processed by a
290:     // single thread and reduces the impact of initialization overhead.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 291-291
```cpp
291:     block_size = (block_size < 128 ? block_size : 128);
```
- **EN:** Declares or updates local/member state such as `block_size`.
- **CN:** 声明或更新局部/成员状态，例如 `block_size`。

### Lines 292-292
```cpp
292: #endif
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 293-293
```cpp
293:   }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 295-295
```cpp
295: #if defined(CUTLASS_ENABLE_SYCL)
```
- **EN:** Conditional-compilation or macro block keyed on `defined(CUTLASS_ENABLE_SYCL)`.
- **CN:** 以 `defined(CUTLASS_ENABLE_SYCL)` 为条件的条件编译或宏定义代码块。

### Lines 296-297
```cpp
296:   const auto sycl_block = compat::dim3(block_size, 1, 1);
297:   const auto sycl_grid = compat::dim3(grid_size, 1, 1);
```
- **EN:** Implements `dim3` for this file's main component.
- **CN:** 为该文件的核心组件实现 `dim3`。

### Lines 299-301
```cpp
299:   compat::launch<kernel::BlockCompareRelativelyEqual<Element>, BlockCompareRelativelyEqualName<Element>>(sycl_grid, sycl_block, device_equal_flag, ptr_A, ptr_B, capacity,
300:                                                                   epsilon, nonzero_floor);
301:   compat::wait();
```
- **EN:** Implements `wait` for this file's main component.
- **CN:** 为该文件的核心组件实现 `wait`。

### Lines 303-303
```cpp
303:   compat::memcpy(&equal_flag, device_equal_flag, sizeof(int));
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 305-305
```cpp
305:   compat::free(reinterpret_cast<void*>(device_equal_flag));
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 306-306
```cpp
306: #else
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 307-308
```cpp
307:   dim3 grid(grid_size, 1, 1);
308:   dim3 block(block_size, 1, 1);
```
- **EN:** Implements `grid` and coordinates helper calls such as `block`.
- **CN:** 实现 `grid`，并协调调用 `block` 等辅助逻辑。

### Lines 310-317
```cpp
310:   kernel::BlockCompareRelativelyEqual<Element><<< grid, block, 0, stream >>>(
311:     device_equal_flag, 
312:     ptr_A, 
313:     ptr_B, 
314:     capacity, 
315:     epsilon, 
316:     nonzero_floor
317:   );
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 319-319
```cpp
319:   cudaStreamSynchronize(stream);
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 321-325
```cpp
321:   if (cudaMemcpy(
322:     &equal_flag, 
323:     device_equal_flag,
324:     sizeof(int), 
325:     cudaMemcpyDeviceToHost) != cudaSuccess) {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 327-327
```cpp
327:     cudaFree(device_equal_flag);
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 329-330
```cpp
329:     throw std::runtime_error("Failed to copy equality flag from device.");
330:   }
```
- **EN:** Implements `runtime_error` for this file's main component.
- **CN:** 为该文件的核心组件实现 `runtime_error`。

### Lines 332-332
```cpp
332:   cudaFree(device_equal_flag);
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 333-333
```cpp
333: #endif
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 335-336
```cpp
335:   return equal_flag;
336: }
```
- **EN:** Returns the computed value, status, or object to the caller.
- **CN:** 向调用者返回计算得到的值、状态或对象。

### Lines 338-338
```cpp
338: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 340-340
```cpp
340: template<template<class> class, class> class BlockElementwiseOpKernelName; 
```
- **EN:** Introduces `BlockElementwiseOpKernelName`, a type used to support tensor utilities.
- **CN:** 引入 `BlockElementwiseOpKernelName`，即一个用于支持张量工具的类型。

### Lines 342-342
```cpp
342: /// Performs an elementwise function of two blocks
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 343-350
```cpp
343: template <template <class> class BinaryOp, typename Element>
344: void BlockElementwiseOp(
345:   Element *ptr_dst,
346:   Element const *ptr_A,
347:   Element const *ptr_B,
348:   size_t capacity,
349:   int grid_size = 0,
350:   int block_size = 0) {
```
- **EN:** Introduces `BinaryOp`, a type used to support tensor utilities.
- **CN:** 引入 `BinaryOp`，即一个用于支持张量工具的类型。

### Lines 353-353
```cpp
353:   if (!grid_size || !block_size) {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 354-354
```cpp
354: #if defined (CUTLASS_ENABLE_SYCL)
```
- **EN:** Conditional-compilation or macro block keyed on `defined`.
- **CN:** 以 `defined` 为条件的条件编译或宏定义代码块。

### Lines 355-357
```cpp
355:     block_size = 128;
356:     grid_size = (capacity + block_size - 1) / block_size;
357:     grid_size = (grid_size < 64 ? grid_size : 64); // limit grid size to avoid out_of_resources runtime error.
```
- **EN:** Declares or updates local/member state such as `block_size`, `grid_size`.
- **CN:** 声明或更新局部/成员状态，例如 `block_size`, `grid_size`。

### Lines 358-358
```cpp
358: #else
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 359-359
```cpp
359:     // if grid_size or block_size are zero, query occupancy using the CUDA Occupancy API
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 360-363
```cpp
360:     cudaError_t result = cudaOccupancyMaxPotentialBlockSize(
361:       &grid_size,
362:       &block_size,
363:       reinterpret_cast<void const *>(kernel::BlockElementwiseOp<BinaryOp, Element>));
```
- **EN:** Implements `cudaOccupancyMaxPotentialBlockSize` for this file's main component.
- **CN:** 为该文件的核心组件实现 `cudaOccupancyMaxPotentialBlockSize`。

### Lines 365-367
```cpp
365:     if (result != cudaSuccess) {
366:       throw std::runtime_error("Failed to query occupancy.");
367:     }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 369-370
```cpp
369:     // Limit block size. This has the effect of increasing the number of items processed by a
370:     // single thread and reduces the impact of initialization overhead.
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 371-371
```cpp
371:     block_size = (block_size < 128 ? block_size : 128);
```
- **EN:** Declares or updates local/member state such as `block_size`.
- **CN:** 声明或更新局部/成员状态，例如 `block_size`。

### Lines 372-372
```cpp
372: #endif
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 373-373
```cpp
373:   }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 375-375
```cpp
375: #if defined(CUTLASS_ENABLE_SYCL)
```
- **EN:** Conditional-compilation or macro block keyed on `defined(CUTLASS_ENABLE_SYCL)`.
- **CN:** 以 `defined(CUTLASS_ENABLE_SYCL)` 为条件的条件编译或宏定义代码块。

### Lines 376-379
```cpp
376:   const auto sycl_block = compat::dim3(block_size, 1, 1);
377:   const auto sycl_grid = compat::dim3(grid_size, 1, 1);
378:   compat::launch<kernel::BlockElementwiseOp<BinaryOp, Element>, BlockElementwiseOpKernelName<BinaryOp, Element>>(
379:       sycl_grid, sycl_block, ptr_dst, ptr_A, ptr_B, capacity);
```
- **EN:** Implements `dim3` for this file's main component.
- **CN:** 为该文件的核心组件实现 `dim3`。

### Lines 380-380
```cpp
380: #else
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 381-383
```cpp
381:   dim3 grid(grid_size, 1, 1);
382:   dim3 block(block_size, 1, 1);
383:   kernel::BlockElementwiseOp<BinaryOp, Element><<< grid, block >>>(ptr_dst, ptr_A, ptr_B, capacity);
```
- **EN:** Implements `grid` and coordinates helper calls such as `block`.
- **CN:** 实现 `grid`，并协调调用 `block` 等辅助逻辑。

### Lines 384-384
```cpp
384: #endif
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 385-385
```cpp
385: }
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 387-387
```cpp
387: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 389-391
```cpp
389: } // device
390: } // reference
391: } // cutlass
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

## Key Concepts / 关键概念
- **Template metaprogramming / 模板元编程**
- **Structured type design / 结构化类型设计**
- **Conditional compilation / 条件编译**
- **CUDA ecosystem integration / CUDA 生态集成**
- **SYCL interoperability / SYCL 互操作**
- **Reference implementation / 参考实现**

## Dependencies / 依赖关系
- **Internal headers / 内部头文件:** `cutlass/cutlass.h`, `cutlass/relatively_equal.h`, `cutlass/util/distribution.h`
- **External headers / 外部头文件:** `utility`, `tensor_foreach.h`
- **Runtime/backends / 运行时与后端:** `CUDA`, `SYCL`
