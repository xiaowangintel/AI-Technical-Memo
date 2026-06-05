# device_nchw_to_nhwc.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `tools/util/include/cutlass/util/device_nchw_to_nhwc.h`
- **Purpose (EN):** This file declares device nchw to nhwc for general CUTLASS utility support.
- **目的 (CN):** 该文件声明了面向CUTLASS 通用工具支持的device nchw to nhwc逻辑。
- **Brief / 简述:** cuda kernels to transform a device memory tensor from NCHW layout to NHWC layout.

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
36:  * \brief cuda kernels to transform a device memory tensor from NCHW layout to NHWC layout.
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
- **EN:** Supporting logic for the device nchw to nhwc implementation.
- **CN:** device nchw to nhwc实现的辅助逻辑。

### Lines 47-49
```cpp
47: /** \brief interface to transform a device memory tensor from NCHW layout to NHWC layout.
48:  * \tparam T: data type
49:  */
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 50-55
```cpp
50: template <typename T>
51: void nchw_to_nhwc(cutlass::Tensor4DCoord input_tensor_size,
52:                   cutlass::Tensor4DCoord output_tensor_size,
53:                   TensorRef<T, layout::TensorNCHW> ref_input,
54:                   TensorRef<T, layout::TensorNHWC> ref_output,
55:                   cudaStream_t stream);
```
- **EN:** Supporting logic for the device nchw to nhwc implementation.
- **CN:** device nchw to nhwc实现的辅助逻辑。

### Lines 57-70
```cpp
57: template <typename T>
58: __global__ void nchw_to_nhwc_kernel(T *output, 
59:                                     const T *input, 
60:                                     const int n,
61:                                     const int h, 
62:                                     const int w, 
63:                                     const int c) {
64:   const int hw = h*w;
65:   const int chw = c*hw;
66:   __shared__ T shbuf[32 * (32 + 1)]; 
67:   const int32_t tid  = threadIdx.y*blockDim.x + threadIdx.x;
68:   const int32_t wid  = tid / 32;     
69:   const int32_t lid  = tid % 32;     
70:   const int32_t ni   = blockIdx.z;
```
- **EN:** Declares or updates local/member state such as `hw`, `w`, `chw`, `tid`.
- **CN:** 声明或更新局部/成员状态，例如 `hw`, `w`, `chw`, `tid`。

### Lines 71-72
```cpp
71:   const int32_t ci0  = blockIdx.y * 32;
72:   const int32_t hwi0 = blockIdx.x * 32;
```
- **EN:** Declares or updates local/member state such as `ci0`, `hwi0`.
- **CN:** 声明或更新局部/成员状态，例如 `ci0`, `hwi0`。

### Lines 74-87
```cpp
74:   const size_t input_idx = ni * chw + (ci0 + wid) * hw + hwi0;
75:   const T *A = input + input_idx;
76:   if (hwi0 + lid < hw) {
77:     const int lid_x_33 = lid * 33;
78:     if ((ci0 + 32) <= c) {
79:       int ci = wid;  // between 0 and 7
80:       CUTLASS_PRAGMA_UNROLL
81:       for (int cLoopIdx = 0; cLoopIdx < 4; cLoopIdx++) {
82:         shbuf[lid_x_33 + ci] = A[lid];
83:         A                    = &A[8 * hw];
84:         ci += 8;
85:       }
86:     } else {
87:       for (int ci = wid; ci < 32; ci += 8) {
```
- **EN:** Declares or updates local/member state such as `input_idx`, `hwi0`, `A`, `lid_x_33`.
- **CN:** 声明或更新局部/成员状态，例如 `input_idx`, `hwi0`, `A`, `lid_x_33`。

### Lines 88-92
```cpp
88:         if ((ci + ci0) < c) {
89:           shbuf[lid_x_33 + ci] = A[lid];
90:         }
91:         A = &A[8 * hw];
92:       }
```
- **EN:** Declares or updates local/member state such as `A`.
- **CN:** 声明或更新局部/成员状态，例如 `A`。

### Lines 93-95
```cpp
93:     }
94:   }
95:   __syncthreads();
```
- **EN:** Implements `__syncthreads` for this file's main component.
- **CN:** 为该文件的核心组件实现 `__syncthreads`。

### Lines 97-110
```cpp
97:   const int32_t ciOut = ci0 + lid;
98:   output = &output[ni * chw + ciOut];
99:   if (ciOut < c) {
100:     if (hwi0 + 32 < hw) {
101:       int hwI = wid;
102:       CUTLASS_PRAGMA_UNROLL
103:       for (int hwLoopIdx = 0; hwLoopIdx < 4; ++hwLoopIdx) {
104:         output[(hwi0 + hwI) * c] = shbuf[(hwI)*33 + lid];
105:         hwI += 8;
106:       }
107:     } else {
108:       for (int hwI = wid; hwI < 32; hwI += 8) {
109:         if (hwi0 + hwI < hw) {
110:           output[(hwi0 + hwI) * c] = shbuf[(hwI)*33 + lid];
```
- **EN:** Declares or updates local/member state such as `ciOut`, `lid`, `output`, `hwI`.
- **CN:** 声明或更新局部/成员状态，例如 `ciOut`, `lid`, `output`, `hwI`。

### Lines 111-114
```cpp
111:         }
112:       }
113:     }
114:   }
```
- **EN:** Supporting logic for the device nchw to nhwc implementation.
- **CN:** device nchw to nhwc实现的辅助逻辑。

### Lines 115-115
```cpp
115: }
```
- **EN:** Supporting logic for the device nchw to nhwc implementation.
- **CN:** device nchw to nhwc实现的辅助逻辑。

### Lines 117-122
```cpp
117: template <typename T>
118: void nchw_to_nhwc(cutlass::Tensor4DCoord input_tensor_size,
119:                   cutlass::Tensor4DCoord output_tensor_size,
120:                   TensorRef<T, layout::TensorNCHW> ref_input,
121:                   TensorRef<T, layout::TensorNHWC> ref_output,
122:                   cudaStream_t stream) {
```
- **EN:** Supporting logic for the device nchw to nhwc implementation.
- **CN:** device nchw to nhwc实现的辅助逻辑。

### Lines 124-128
```cpp
124:   assert(
125:     input_tensor_size.n() == output_tensor_size.n() &&
126:     input_tensor_size.c() == output_tensor_size.h() &&
127:     input_tensor_size.h() == output_tensor_size.w() &&
128:     input_tensor_size.w() == output_tensor_size.c());
```
- **EN:** Supporting logic for the device nchw to nhwc implementation.
- **CN:** device nchw to nhwc实现的辅助逻辑。

### Lines 130-133
```cpp
130:   int n = output_tensor_size.n();
131:   int h = output_tensor_size.h();
132:   int w = output_tensor_size.w();
133:   int c = output_tensor_size.c();
```
- **EN:** Implements `n` and coordinates helper calls such as `h`, `w`, `c`.
- **CN:** 实现 `n`，并协调调用 `h`, `w`, `c` 等辅助逻辑。

### Lines 135-139
```cpp
135:   dim3 grid((h*w + 31)/32, (c + 31)/32, n);
136:   dim3 block(32, 8);
137:   nchw_to_nhwc_kernel<<<grid, block, 0, stream>>>(ref_output.data(), ref_input.data(), 
138:                                                   n, h, w, c);
139: }
```
- **EN:** Implements `grid` and coordinates helper calls such as `block`, `data`.
- **CN:** 实现 `grid`，并协调调用 `block`, `data` 等辅助逻辑。

### Lines 141-141
```cpp
141: } //namespace cutlass
```
- **EN:** Supporting logic for the device nchw to nhwc implementation.
- **CN:** device nchw to nhwc实现的辅助逻辑。

## Key Concepts / 关键概念
- **Template metaprogramming / 模板元编程**
- **CUDA ecosystem integration / CUDA 生态集成**
- **Tensor manipulation / 张量处理**

## Dependencies / 依赖关系
- **Internal headers / 内部头文件:** `cutlass/cutlass.h`, `cutlass/layout/tensor.h`, `cutlass/numeric_types.h`, `cutlass/tensor_coord.h`, `cutlass/tensor_ref.h`
- **Runtime/backends / 运行时与后端:** `CUDA`
