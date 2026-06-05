# tensor_elementwise.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `tools/util/include/cutlass/util/reference/device/kernel/tensor_elementwise.h`
- **Purpose (EN):** This file declares tensor utilities for device-side reference kernels.
- **目的 (CN):** 该文件声明了面向设备端参考内核的张量工具逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20
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
30:  **************************************************************************************************/
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 32-32
```cpp
32: #pragma once
```
- **EN:** Uses include guards or pragmas to ensure the declarations are processed only once.
- **CN:** 使用 include guard 或 pragma，确保这些声明只被处理一次。

### Lines 34-34
```cpp
34: #include <curand_kernel.h>
```
- **EN:** Includes the main dependencies required by this module; notable headers are `curand_kernel.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `curand_kernel.h`。

### Lines 36-36
```cpp
36: #include "cutlass/cutlass.h"
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cutlass/cutlass.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cutlass/cutlass.h`。

### Lines 38-41
```cpp
38: namespace cutlass {
39: namespace reference {
40: namespace device {
41: namespace kernel {
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 43-43
```cpp
43: ////////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 45-45
```cpp
45: /// Kernel to initialize tensor to uniform random distribution
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 46-49
```cpp
46: template <typename T>
47: __global__ void TensorInitializeUniform(
48:     Distribution dist, int64_t seed, int dim_contiguous, int dim_strided, T *tensor, int ldm) {
49:   __shared__ curandState_t rng_state[1024];
```
- **EN:** Implements `TensorInitializeUniform` for this file's main component.
- **CN:** 为该文件的核心组件实现 `TensorInitializeUniform`。

### Lines 51-51
```cpp
51:   uint64_t gtid = threadIdx.x + blockIdx.x * blockDim.x + blockIdx.y * gridDim.x * blockDim.x;
```
- **EN:** Declares or updates local/member state such as `gtid`, `x`.
- **CN:** 声明或更新局部/成员状态，例如 `gtid`, `x`。

### Lines 53-53
```cpp
53:   curand_init(seed, gtid, 0, &rng_state[threadIdx.x]);
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 55-56
```cpp
55:   int c_idx = blockIdx.x * blockDim.x + threadIdx.x;
56:   int s_idx = blockIdx.y * blockDim.x;
```
- **EN:** Declares or updates local/member state such as `c_idx`, `x`, `s_idx`.
- **CN:** 声明或更新局部/成员状态，例如 `c_idx`, `x`, `s_idx`。

### Lines 58-58
```cpp
58:   tensor += s_idx * ldm + c_idx;
```
- **EN:** Declares or updates local/member state such as `c_idx`.
- **CN:** 声明或更新局部/成员状态，例如 `c_idx`。

### Lines 60-62
```cpp
60:   for (int s_offset = 0; s_offset < blockDim.x; ++s_offset, ++s_idx) {
61:     if (s_idx < dim_strided && c_idx < dim_contiguous) {
62:       double range = dist.uniform.max - dist.uniform.min;
```
- **EN:** Declares or updates local/member state such as `s_offset`, `x`, `range`, `min`.
- **CN:** 声明或更新局部/成员状态，例如 `s_offset`, `x`, `range`, `min`。

### Lines 64-64
```cpp
64:       double rnd = curand_uniform(&rng_state[threadIdx.x]);
```
- **EN:** Declares or updates local/member state such as `rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `rnd`。

### Lines 66-66
```cpp
66:       rnd = dist.uniform.min + range * rnd;
```
- **EN:** Declares or updates local/member state such as `rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `rnd`。

### Lines 68-69
```cpp
68:       // Random values are cast to integer after scaling by a power of two to facilitate error
69:       // testing
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 70-75
```cpp
70:       if (dist.int_scale >= 0) {
71:         rnd = double(int(rnd * double(1 << dist.int_scale)));
72:         *tensor = T(rnd / double(1 << dist.int_scale));
73:       } else {
74:         *tensor = T(rnd);
75:       }
```
- **EN:** Declares or updates local/member state such as `rnd`, `tensor`.
- **CN:** 声明或更新局部/成员状态，例如 `rnd`, `tensor`。

### Lines 77-80
```cpp
77:       tensor += ldm;
78:     }
79:   }
80: }
```
- **EN:** Declares or updates local/member state such as `ldm`.
- **CN:** 声明或更新局部/成员状态，例如 `ldm`。

### Lines 82-82
```cpp
82: ///////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 84-84
```cpp
84: /// Kernel to initialize tensor to uniform distribution
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 85-88
```cpp
85: template <typename T>
86: __global__ void TensorInitializeGaussian(
87:     Distribution dist, int64_t seed, int dim_contiguous, int dim_strided, T *tensor, int ldm) {
88:   __shared__ curandState_t rng_state[1024];
```
- **EN:** Implements `TensorInitializeGaussian` for this file's main component.
- **CN:** 为该文件的核心组件实现 `TensorInitializeGaussian`。

### Lines 90-90
```cpp
90:   uint64_t gtid = threadIdx.x + blockIdx.x * blockDim.x + blockIdx.y * gridDim.x * blockDim.x;
```
- **EN:** Declares or updates local/member state such as `gtid`, `x`.
- **CN:** 声明或更新局部/成员状态，例如 `gtid`, `x`。

### Lines 92-92
```cpp
92:   curand_init(seed, gtid, 0, &rng_state[threadIdx.x]);
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 94-95
```cpp
94:   int c_idx = blockIdx.x * blockDim.x + threadIdx.x;
95:   int s_idx = blockIdx.y * blockDim.x;
```
- **EN:** Declares or updates local/member state such as `c_idx`, `x`, `s_idx`.
- **CN:** 声明或更新局部/成员状态，例如 `c_idx`, `x`, `s_idx`。

### Lines 97-97
```cpp
97:   tensor += s_idx * ldm + c_idx;
```
- **EN:** Declares or updates local/member state such as `c_idx`.
- **CN:** 声明或更新局部/成员状态，例如 `c_idx`。

### Lines 99-100
```cpp
99:   for (int s_offset = 0; s_offset < blockDim.x; ++s_offset, ++s_idx) {
100:     if (s_idx < dim_strided && c_idx < dim_contiguous) {
```
- **EN:** Declares or updates local/member state such as `s_offset`, `x`.
- **CN:** 声明或更新局部/成员状态，例如 `s_offset`, `x`。

### Lines 101-102
```cpp
101:       // Random values are cast to integer after scaling by a power of two to facilitate error
102:       // testing
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 104-104
```cpp
104:       double rnd = curand_normal(&rng_state[threadIdx.x]);
```
- **EN:** Declares or updates local/member state such as `rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `rnd`。

### Lines 106-106
```cpp
106:       rnd = dist.gaussian.mean + dist.gaussian.stddev * rnd;
```
- **EN:** Declares or updates local/member state such as `rnd`.
- **CN:** 声明或更新局部/成员状态，例如 `rnd`。

### Lines 108-116
```cpp
108:       if (dist.int_scale >= 0) {
109:         rnd = double(int(rnd * double(1 << dist.int_scale)));
110:         *tensor = T(rnd / double(1 << dist.int_scale));
111:       } else {
112:         *tensor = T(rnd);
113:       }
114:     }
115:   }
116: }
```
- **EN:** Declares or updates local/member state such as `rnd`, `tensor`.
- **CN:** 声明或更新局部/成员状态，例如 `rnd`, `tensor`。

### Lines 118-118
```cpp
118: /// Kernel to initialize tensor to an identity matrix
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 119-122
```cpp
119: template <typename T>
120: __global__ void TensorInitializeLinear(
121:     Distribution dist, int64_t seed, int dim_contiguous, int dim_strided, T *tensor, int ldm) {
122:   __shared__ curandState_t rng_state[1024];
```
- **EN:** Implements `TensorInitializeLinear` for this file's main component.
- **CN:** 为该文件的核心组件实现 `TensorInitializeLinear`。

### Lines 124-124
```cpp
124:   uint64_t gtid = threadIdx.x + blockIdx.x * blockDim.x + blockIdx.y * gridDim.x * blockDim.x;
```
- **EN:** Declares or updates local/member state such as `gtid`, `x`.
- **CN:** 声明或更新局部/成员状态，例如 `gtid`, `x`。

### Lines 126-126
```cpp
126:   curand_init(seed, gtid, 0, &rng_state[threadIdx.x]);
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 128-129
```cpp
128:   int c_idx = blockIdx.x * blockDim.x + threadIdx.x;
129:   int s_idx = blockIdx.y * blockDim.x;
```
- **EN:** Declares or updates local/member state such as `c_idx`, `x`, `s_idx`.
- **CN:** 声明或更新局部/成员状态，例如 `c_idx`, `x`, `s_idx`。

### Lines 131-131
```cpp
131:   tensor += s_idx * ldm + c_idx;
```
- **EN:** Declares or updates local/member state such as `c_idx`.
- **CN:** 声明或更新局部/成员状态，例如 `c_idx`。

### Lines 133-139
```cpp
133:   for (int s_offset = 0; s_offset < blockDim.x; ++s_offset, ++s_idx) {
134:     if (s_idx < dim_strided && c_idx < dim_contiguous) {
135:       *tensor =
136:           dist.linear.offset + dist.linear.delta_row * c_idx + dist.linear.delta_column * s_idx;
137:     }
138:   }
139: }
```
- **EN:** Declares or updates local/member state such as `s_offset`, `x`, `tensor`, `s_idx`.
- **CN:** 声明或更新局部/成员状态，例如 `s_offset`, `x`, `tensor`, `s_idx`。

### Lines 141-141
```cpp
141: /// Kernel to initialize tensor to an identity matrix
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 142-145
```cpp
142: template <typename T>
143: __global__ void TensorInitializeIdentity(
144:     Distribution dist, int64_t seed, int dim_contiguous, int dim_strided, T *tensor, int ldm) {
145:   __shared__ curandState_t rng_state[1024];
```
- **EN:** Implements `TensorInitializeIdentity` for this file's main component.
- **CN:** 为该文件的核心组件实现 `TensorInitializeIdentity`。

### Lines 147-147
```cpp
147:   uint64_t gtid = threadIdx.x + blockIdx.x * blockDim.x + blockIdx.y * gridDim.x * blockDim.x;
```
- **EN:** Declares or updates local/member state such as `gtid`, `x`.
- **CN:** 声明或更新局部/成员状态，例如 `gtid`, `x`。

### Lines 149-149
```cpp
149:   curand_init(seed, gtid, 0, &rng_state[threadIdx.x]);
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

### Lines 151-152
```cpp
151:   int c_idx = blockIdx.x * blockDim.x + threadIdx.x;
152:   int s_idx = blockIdx.y * blockDim.x;
```
- **EN:** Declares or updates local/member state such as `c_idx`, `x`, `s_idx`.
- **CN:** 声明或更新局部/成员状态，例如 `c_idx`, `x`, `s_idx`。

### Lines 154-154
```cpp
154:   tensor += s_idx * ldm + c_idx;
```
- **EN:** Declares or updates local/member state such as `c_idx`.
- **CN:** 声明或更新局部/成员状态，例如 `c_idx`。

### Lines 156-161
```cpp
156:   for (int s_offset = 0; s_offset < blockDim.x; ++s_offset, ++s_idx) {
157:     if (s_idx < dim_strided && c_idx < dim_contiguous) {
158:       *tensor = (c_idx == s_idx ? T(1) : T(0));
159:     }
160:   }
161: }
```
- **EN:** Declares or updates local/member state such as `s_offset`, `x`, `tensor`, `c_idx`.
- **CN:** 声明或更新局部/成员状态，例如 `s_offset`, `x`, `tensor`, `c_idx`。

### Lines 163-163
```cpp
163: ////////////////////////////////////////////////////////////////////////////////////////////////////
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 165-168
```cpp
165: } // namespace kernel
166: } // namespace device
167: } // namespace reference
168: } // namespace cutlass
```
- **EN:** Supporting logic for the tensor utilities implementation.
- **CN:** 张量工具实现的辅助逻辑。

## Key Concepts / 关键概念
- **Template metaprogramming / 模板元编程**
- **Reference implementation / 参考实现**
- **Tensor manipulation / 张量处理**

## Dependencies / 依赖关系
- **Internal headers / 内部头文件:** `cutlass/cutlass.h`
- **External headers / 外部头文件:** `curand_kernel.h`
