# device_nhwc_to_nchw.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `tools/util/include/cutlass/util/device_nhwc_to_nchw.h`
- **Purpose (EN):** This file declares device nhwc to nchw for general CUTLASS utility support.
- **目的 (CN):** 该文件声明了面向CUTLASS 通用工具支持的device nhwc to nchw逻辑。
- **Brief / 简述:** cuda kernels to transform a device memory tensor from NHWC layout to NCHW layout.

## Line-by-Line Analysis / 逐行分析
### Lines 1-20
```cpp
1: /******************************************************************************
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
```
- **EN:** License header and copyright terms for the file.
- **CN:** 文件的许可证头和版权条款。

### Lines 21-30
```cpp
21:  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
22:  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
23:  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
24:  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
25:  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
26:  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
27:  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
28:  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
29:  *
30:  ******************************************************************************/
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 32-32
```cpp
32: #pragma once
```
- **EN:** Uses include guards or pragmas to ensure the declarations are processed only once.
- **CN:** 使用 include guard 或 pragma，确保这些声明只被处理一次。

### Lines 34-37
```cpp
34: /**
35:  * \file
36:  * \brief cuda kernels to transform a device memory tensor from NHWC layout to NCHW layout.
37:  */
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 39-43
```cpp
39: #include "cutlass/cutlass.h"
40: #include "cutlass/layout/tensor.h"
41: #include "cutlass/numeric_types.h"
42: #include "cutlass/tensor_coord.h"
43: #include "cutlass/tensor_ref.h"
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cutlass/cutlass.h`, `cutlass/layout/tensor.h`, `cutlass/numeric_types.h`, `cutlass/tensor_coord.h`, `cutlass/tensor_ref.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cutlass/cutlass.h`, `cutlass/layout/tensor.h`, `cutlass/numeric_types.h`, `cutlass/tensor_coord.h`, `cutlass/tensor_ref.h`。

### Lines 45-45
```cpp
45: namespace cutlass {
```
- **EN:** Supporting logic for the device nhwc to nchw implementation.
- **CN:** device nhwc to nchw实现的辅助逻辑。

### Lines 47-49
```cpp
47: /** \brief interface to transform a device memory tensor from NHWC layout to NCHW layout.
48:  * \tparam T: data type
49:  */
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 50-55
```cpp
50: template <typename T>
51: void nhwc_to_nchw(cutlass::Tensor4DCoord input_tensor_size,
52:                   cutlass::Tensor4DCoord output_tensor_size,
53:                   TensorRef<T, layout::TensorNHWC> ref_input,
54:                   TensorRef<T, layout::TensorNCHW> ref_output,
55:                   cudaStream_t stream);
```
- **EN:** Supporting logic for the device nhwc to nchw implementation.
- **CN:** device nhwc to nchw实现的辅助逻辑。

### Lines 58-64
```cpp
58: template <typename T>
59: __global__ void nhwc_to_nchw_kernel(T *output, 
60:                                     const T *input, 
61:                                     const int n,
62:                                     const int h, 
63:                                     const int w, 
64:                                     const int c) {
```
- **EN:** Supporting logic for the device nhwc to nchw implementation.
- **CN:** device nhwc to nchw实现的辅助逻辑。

### Lines 66-74
```cpp
66:   const int hw = h*w;
67:   const int hwc = hw*c;
68:   __shared__ T shbuf[32 * (32 + 1)]; 
69:   const int32_t tid  = threadIdx.y*blockDim.x + threadIdx.x;
70:   const int32_t wid  = tid / 32; 
71:   const int32_t lid  = tid % 32; 
72:   const int32_t ni   = blockIdx.z;
73:   const int32_t hwi0  = blockIdx.y * 32;  
74:   const int32_t ci0 = blockIdx.x * 32;  
```
- **EN:** Declares or updates local/member state such as `hw`, `w`, `hwc`, `c`.
- **CN:** 声明或更新局部/成员状态，例如 `hw`, `w`, `hwc`, `c`。

### Lines 76-89
```cpp
76:   const size_t input_idx = ni * hwc + (hwi0 + wid) * c + ci0;
77:   const T *A = input + input_idx;
78:   if (ci0 + lid < c) {
79:     const int lid_x_33 = lid * 33;
80:     if ((hwi0 + 32) <= hw) {
81:       int hwi = wid;  // between 0 and 7
82:       CUTLASS_PRAGMA_UNROLL
83:       for (int cLoopIdx = 0; cLoopIdx < 4; cLoopIdx++) { 
84:         shbuf[lid_x_33 + hwi] = A[lid];
85:         A                     = &A[8 * c];
86:         hwi += 8;
87:       }
88:     } else {
89:       for (int hwi = wid; hwi < 32; hwi += 8) { 
```
- **EN:** Declares or updates local/member state such as `input_idx`, `ci0`, `A`, `lid_x_33`.
- **CN:** 声明或更新局部/成员状态，例如 `input_idx`, `ci0`, `A`, `lid_x_33`。

### Lines 90-94
```cpp
90:         if ((hwi + hwi0) < hw) {
91:           shbuf[lid_x_33 + hwi] = A[lid];
92:         }
93:         A = &A[8 * c];
94:       }
```
- **EN:** Declares or updates local/member state such as `A`.
- **CN:** 声明或更新局部/成员状态，例如 `A`。

### Lines 95-97
```cpp
95:     }
96:   }
97:   __syncthreads();
```
- **EN:** Implements `__syncthreads` for this file's main component.
- **CN:** 为该文件的核心组件实现 `__syncthreads`。

### Lines 99-112
```cpp
99:   const int32_t hwiOut = hwi0 + lid;
100:   output = &output[ni * hwc + hwiOut];
101:   if (hwiOut < hw) {
102:     if (ci0 + 32 < c) {
103:       int cI = wid;
104:       CUTLASS_PRAGMA_UNROLL
105:       for (int hwLoopIdx = 0; hwLoopIdx < 4; ++hwLoopIdx) {
106:         output[(ci0 + cI) * hw] = shbuf[(cI)*33 + lid];
107:         cI += 8;
108:       }
109:     } else {
110:       for (int cI = wid; cI < 32; cI += 8) {
111:         if (ci0 + cI < c) {
112:           output[(ci0 + cI) * hw] = shbuf[(cI)*33 + lid];
```
- **EN:** Declares or updates local/member state such as `hwiOut`, `lid`, `output`, `cI`.
- **CN:** 声明或更新局部/成员状态，例如 `hwiOut`, `lid`, `output`, `cI`。

### Lines 113-116
```cpp
113:         }
114:       }
115:     }
116:   }
```
- **EN:** Supporting logic for the device nhwc to nchw implementation.
- **CN:** device nhwc to nchw实现的辅助逻辑。

### Lines 117-117
```cpp
117: }
```
- **EN:** Supporting logic for the device nhwc to nchw implementation.
- **CN:** device nhwc to nchw实现的辅助逻辑。

### Lines 119-124
```cpp
119: template <typename T>
120: void nhwc_to_nchw(cutlass::Tensor4DCoord input_tensor_size,
121:                   cutlass::Tensor4DCoord output_tensor_size,
122:                   TensorRef<T, layout::TensorNHWC> ref_input,
123:                   TensorRef<T, layout::TensorNCHW> ref_output,
124:                   cudaStream_t stream) {
```
- **EN:** Supporting logic for the device nhwc to nchw implementation.
- **CN:** device nhwc to nchw实现的辅助逻辑。

### Lines 126-130
```cpp
126:   assert(
127:     input_tensor_size.n() == output_tensor_size.n() &&
128:     input_tensor_size.h() == output_tensor_size.c() &&
129:     input_tensor_size.w() == output_tensor_size.h() &&
130:     input_tensor_size.c() == output_tensor_size.w());
```
- **EN:** Supporting logic for the device nhwc to nchw implementation.
- **CN:** device nhwc to nchw实现的辅助逻辑。

### Lines 132-135
```cpp
132:   int n = input_tensor_size.n();
133:   int h = input_tensor_size.h();
134:   int w = input_tensor_size.w();
135:   int c = input_tensor_size.c();
```
- **EN:** Implements `n` and coordinates helper calls such as `h`, `w`, `c`.
- **CN:** 实现 `n`，并协调调用 `h`, `w`, `c` 等辅助逻辑。

### Lines 137-140
```cpp
137:   dim3 grid((c + 31)/32, (h*w + 31)/32, n);
138:   dim3 block(32, 8);
139:   nhwc_to_nchw_kernel<<<grid, block, 0, stream>>>(ref_output.data(), ref_input.data(), 
140:                                                   n, h, w, c);
```
- **EN:** Implements `grid` and coordinates helper calls such as `block`, `data`.
- **CN:** 实现 `grid`，并协调调用 `block`, `data` 等辅助逻辑。

### Lines 142-142
```cpp
142: }
```
- **EN:** Supporting logic for the device nhwc to nchw implementation.
- **CN:** device nhwc to nchw实现的辅助逻辑。

### Lines 144-144
```cpp
144: } //namespace cutlass
```
- **EN:** Supporting logic for the device nhwc to nchw implementation.
- **CN:** device nhwc to nchw实现的辅助逻辑。

## Key Concepts / 关键概念
- **Template metaprogramming / 模板元编程**
- **CUDA ecosystem integration / CUDA 生态集成**
- **Tensor manipulation / 张量处理**

## Dependencies / 依赖关系
- **Internal headers / 内部头文件:** `cutlass/cutlass.h`, `cutlass/layout/tensor.h`, `cutlass/numeric_types.h`, `cutlass/tensor_coord.h`, `cutlass/tensor_ref.h`
- **Runtime/backends / 运行时与后端:** `CUDA`
