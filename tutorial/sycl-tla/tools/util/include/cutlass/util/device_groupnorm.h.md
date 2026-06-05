# device_groupnorm.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `tools/util/include/cutlass/util/device_groupnorm.h`
- **Purpose (EN):** This file declares device groupnorm for general CUTLASS utility support.
- **目的 (CN):** 该文件声明了面向CUTLASS 通用工具支持的device groupnorm逻辑。
- **Brief / 简述:** cuda kernels to do group norm on a device memory tensor with NHWC layout. The tensor will be divided into [N, H, W, G, C'] and then we do normalization on [H, W, C'].

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
36:  * \brief cuda kernels to do group norm on a device memory tensor with NHWC layout. The tensor will be divided into [N, H, W, G, C'] and then we do normalization on [H, W, C'].
37:  */
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 39-45
```cpp
39: #include "cutlass/cutlass.h"
40: #include "cutlass/layout/tensor.h"
41: #include "cutlass/numeric_types.h"
42: #include "cutlass/tensor_coord.h"
43: #include "cutlass/tensor_ref.h"
44: #include "device_utils.h"
45: #include <cfloat>
```
- **EN:** Includes the main dependencies required by this module; notable headers are `cutlass/cutlass.h`, `cutlass/layout/tensor.h`, `cutlass/numeric_types.h`, `cutlass/tensor_coord.h`, `cutlass/tensor_ref.h`.
- **CN:** 引入该模块所需的主要依赖；较重要的头文件有 `cutlass/cutlass.h`, `cutlass/layout/tensor.h`, `cutlass/numeric_types.h`, `cutlass/tensor_coord.h`, `cutlass/tensor_ref.h`。

### Lines 47-47
```cpp
47: namespace cutlass {
```
- **EN:** Supporting logic for the device groupnorm implementation.
- **CN:** device groupnorm实现的辅助逻辑。

### Lines 49-51
```cpp
49: /** \brief interface to do group norm on a device memory tensor with NHWC layout.
50:  * \tparam T: data type
51:  */
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 52-60
```cpp
52: template <typename T>
53: void groupnorm(cutlass::Tensor4DCoord input_size,
54:                const int num_groups,
55:                const float eps,
56:                TensorRef<T, layout::TensorNHWC> ref_output,
57:                TensorRef<T, layout::TensorNHWC> ref_input,
58:                TensorRef<T, layout::TensorNHWC> ref_gamma,
59:                TensorRef<T, layout::TensorNHWC> ref_beta,
60:                cudaStream_t stream);
```
- **EN:** Supporting logic for the device groupnorm implementation.
- **CN:** device groupnorm实现的辅助逻辑。

### Lines 62-62
```cpp
62: extern __shared__ char groupnorm_shm[];
```
- **EN:** Supporting logic for the device groupnorm implementation.
- **CN:** device groupnorm实现的辅助逻辑。

### Lines 64-68
```cpp
64: // For small prod_dim1_to_last_dim/num_groups, to avoid multiple loads from global memory,
65: // we store the input in the shared memory.
66: // grid(num_groups, dim0)
67: // block(BLOCKSIZE)
68: // BLOCKSIZE * TVecs_PER_THREAD <= prod_dim1_to_last_dim/num_group
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 69-82
```cpp
69: template<typename TVec, typename T, int T_PER_TVec>
70: __global__ void groupnorm_twopass_store_locally(T*          output,
71:                                                 const T*    input,
72:                                                 const T*    gamma,
73:                                                 const T*    beta,
74:                                                 int         num_groups,
75:                                                 int         prod_dim1_to_last_dim,
76:                                                 int         last_dim,
77:                                                 const float eps,
78:                                                 const int   TVecs_PER_THREAD)
79: {
80:     const int   bid               = blockIdx.y;   // index of batch
81:     const int   gid               = blockIdx.x;   // index of group
82:     const int   tid               = threadIdx.x;  // index of thread
```
- **EN:** Declares or updates local/member state such as `bid`, `y`, `gid`, `x`.
- **CN:** 声明或更新局部/成员状态，例如 `bid`, `y`, `gid`, `x`。

### Lines 83-92
```cpp
83:     const int   bdimx             = blockDim.x;
84:     const int   s_reduce_elements = prod_dim1_to_last_dim / num_groups;
85:     const int   v_reduce_elements = s_reduce_elements / T_PER_TVec;
86:     const int   s_group_stride    = last_dim / num_groups;
87:     const int   v_group_stride    = s_group_stride / T_PER_TVec;
88:     const int   offset_of_group   = (bid * prod_dim1_to_last_dim + gid * s_group_stride) / T_PER_TVec;
89:     const TVec* input_TVec_ptr    = (const TVec*)(input) + offset_of_group;
90:     TVec*       output_TVec_ptr   = (TVec*)(output) + offset_of_group;
91:     T*       local_val         = ((T*)groupnorm_shm) + TVecs_PER_THREAD * T_PER_TVec * tid;
92:     float       local_sum[1]      = {0.0f};
```
- **EN:** Declares or updates local/member state such as `bdimx`, `x`, `s_reduce_elements`, `num_groups`.
- **CN:** 声明或更新局部/成员状态，例如 `bdimx`, `x`, `s_reduce_elements`, `num_groups`。

### Lines 94-94
```cpp
94: // load from global memory into shared memory
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 95-95
```cpp
95: #pragma unroll
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 96-104
```cpp
96:     for (int i = 0; i < TVecs_PER_THREAD; i += 1) {
97:         const int current_load_start_idx = (i * bdimx + tid) * T_PER_TVec;
98:         const int offset_in_group =
99:             ((current_load_start_idx / s_group_stride) * last_dim + (current_load_start_idx % s_group_stride))
100:             / T_PER_TVec;
101:         if (current_load_start_idx < s_reduce_elements) {
102:             TVec      tmp_vec          = input_TVec_ptr[offset_in_group];
103:             T*        tmp_vec_ptr      = (T*)(&tmp_vec);
104:             const int local_val_offset = i * T_PER_TVec;
```
- **EN:** Declares or updates local/member state such as `i`, `TVecs_PER_THREAD`, `current_load_start_idx`, `T_PER_TVec`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `TVecs_PER_THREAD`, `current_load_start_idx`, `T_PER_TVec`。

### Lines 105-105
```cpp
105: #pragma unroll
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 106-113
```cpp
106:             for (int j = 0; j < T_PER_TVec; j++) {
107:                 float tmp = static_cast<float>(tmp_vec_ptr[j]);
108:                 local_sum[0] += tmp;
109:                 local_val[local_val_offset + j] = tmp_vec_ptr[j];
110:             }
111:         }
112:     }
113:     __shared__ float s_mean, s_variance;
```
- **EN:** Declares or updates local/member state such as `j`, `T_PER_TVec`, `tmp`, `s_variance`.
- **CN:** 声明或更新局部/成员状态，例如 `j`, `T_PER_TVec`, `tmp`, `s_variance`。

### Lines 115-115
```cpp
115:     // reduction for mean
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 116-125
```cpp
116:     if (bdimx <= 32) {
117:         warpReduceSum<float, 1>(local_sum);
118:     }
119:     else {
120:         blockReduceSum<float, 1>(local_sum);
121:     }
122:     if (tid == 0) {
123:         s_mean = local_sum[0] / s_reduce_elements;
124:     }
125:     __syncthreads();
```
- **EN:** Declares or updates local/member state such as `tid`, `s_mean`, `s_reduce_elements`.
- **CN:** 声明或更新局部/成员状态，例如 `tid`, `s_mean`, `s_reduce_elements`。

### Lines 127-127
```cpp
127:     // reduction for std
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 128-128
```cpp
128:     local_sum[0] = 0.0f;
```
- **EN:** Supporting logic for the device groupnorm implementation.
- **CN:** device groupnorm实现的辅助逻辑。

### Lines 129-129
```cpp
129: #pragma unroll
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 130-133
```cpp
130:     for (int i = 0; i < TVecs_PER_THREAD; i += 1) {
131:         const int current_load_start_idx = (i * bdimx + tid) * T_PER_TVec;
132:         if (current_load_start_idx < s_reduce_elements) {
133:             const int local_val_offset = i * T_PER_TVec;
```
- **EN:** Declares or updates local/member state such as `i`, `TVecs_PER_THREAD`, `current_load_start_idx`, `T_PER_TVec`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `TVecs_PER_THREAD`, `current_load_start_idx`, `T_PER_TVec`。

### Lines 134-134
```cpp
134: #pragma unroll
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 135-148
```cpp
135:             for (int j = 0; j < T_PER_TVec; j++) {
136:                 float tmp = static_cast<float>(local_val[local_val_offset + j]);
137:                 tmp -= s_mean;
138:                 local_sum[0] += tmp * tmp;
139:             }
140:         }
141:     }
142:     if (bdimx <= 32) {
143:         warpReduceSum<float, 1>(local_sum);
144:     }
145:     else {
146:         blockReduceSum<float, 1>(local_sum);
147:     }
148:     if (tid == 0) {
```
- **EN:** Declares or updates local/member state such as `j`, `T_PER_TVec`, `tmp`, `s_mean`.
- **CN:** 声明或更新局部/成员状态，例如 `j`, `T_PER_TVec`, `tmp`, `s_mean`。

### Lines 149-151
```cpp
149:         s_variance = rsqrtf(local_sum[0] / s_reduce_elements + eps);
150:     }
151:     __syncthreads();
```
- **EN:** Implements `rsqrtf` and coordinates helper calls such as `__syncthreads`.
- **CN:** 实现 `rsqrtf`，并协调调用 `__syncthreads` 等辅助逻辑。

### Lines 153-153
```cpp
153:     // normalize
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 154-156
```cpp
154:     const int   gamma_offset_of_group = gid * v_group_stride;
155:     const TVec* gamma_TVec_ptr        = (const TVec*)gamma + gamma_offset_of_group;
156:     const TVec* beta_TVec_ptr         = (const TVec*)beta + gamma_offset_of_group;
```
- **EN:** Declares or updates local/member state such as `gamma_offset_of_group`, `v_group_stride`, `gamma_TVec_ptr`, `beta_TVec_ptr`.
- **CN:** 声明或更新局部/成员状态，例如 `gamma_offset_of_group`, `v_group_stride`, `gamma_TVec_ptr`, `beta_TVec_ptr`。

### Lines 157-157
```cpp
157: #pragma unroll
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 158-171
```cpp
158:     for (int i = 0; i < TVecs_PER_THREAD; i += 1) {
159:         const int current_load_start_idx = (i * bdimx + tid) * T_PER_TVec;
160:         const int offset_in_group =
161:             ((current_load_start_idx / s_group_stride) * last_dim + (current_load_start_idx % s_group_stride))
162:             / T_PER_TVec;
163:         const int gamma_offset_in_group = (current_load_start_idx % s_group_stride) / T_PER_TVec;
164:         const int local_val_offset      = i * T_PER_TVec;
165:         if (current_load_start_idx < s_reduce_elements) {
166:             TVec gamma_val     = gamma_TVec_ptr[gamma_offset_in_group];
167:             TVec beta_val      = beta_TVec_ptr[gamma_offset_in_group];
168:             T*   gamma_val_ptr = (T*)(&gamma_val);
169:             T*   beta_val_ptr  = (T*)(&beta_val);
170:             TVec tmp_vec;
171:             T*   tmp_vec_ptr = (T*)(&tmp_vec);
```
- **EN:** Declares or updates local/member state such as `i`, `TVecs_PER_THREAD`, `current_load_start_idx`, `T_PER_TVec`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `TVecs_PER_THREAD`, `current_load_start_idx`, `T_PER_TVec`。

### Lines 172-172
```cpp
172: #pragma unroll
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 173-186
```cpp
173:             for (int j = 0; j < T_PER_TVec; j++) {
174:                 float tmp = (static_cast<float>(local_val[local_val_offset + j]) - s_mean) * s_variance
175:                                 * static_cast<float>(gamma_val_ptr[j])
176:                             + static_cast<float>(beta_val_ptr[j]);
177:                 if (sizeof(T) == sizeof(half)) {
178:                     tmp_vec_ptr[j] = T(__float2half_rn(tmp));
179:                 }
180:                 else {
181:                     tmp_vec_ptr[j] = T(tmp);
182:                 }
183:             }
184:             output_TVec_ptr[offset_in_group] = tmp_vec;
185:         }
186:     }
```
- **EN:** Declares or updates local/member state such as `j`, `T_PER_TVec`, `tmp`, `tmp_vec`.
- **CN:** 声明或更新局部/成员状态，例如 `j`, `T_PER_TVec`, `tmp`, `tmp_vec`。

### Lines 187-187
```cpp
187: }
```
- **EN:** Supporting logic for the device groupnorm implementation.
- **CN:** device groupnorm实现的辅助逻辑。

### Lines 189-194
```cpp
189: // For large prod_dim1_to_last_dim/num_groups,
190: // in which the data cannot be stored locally,
191: // we will load from global memory multiple times,
192: // grid(num_groups, dim0)
193: // block(BLOCKSIZE)
194: // BLOCKSIZE * TVecs_PER_THREAD <= prod_dim1_to_last_dim/num_group
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 195-208
```cpp
195: template<typename TVec, typename T, int T_PER_TVec>
196: __global__ void groupnorm_twopass_multiple_load(T*          output,
197:                                                 const T*    input,
198:                                                 const T*    gamma,
199:                                                 const T*    beta,
200:                                                 int         num_groups,
201:                                                 int         prod_dim1_to_last_dim,
202:                                                 int         last_dim,
203:                                                 const float eps,
204:                                                 const int   TVecs_PER_THREAD)
205: {
206:     const int   bid               = blockIdx.y;   // index of batch
207:     const int   gid               = blockIdx.x;   // index of group
208:     const int   tid               = threadIdx.x;  // index of thread
```
- **EN:** Declares or updates local/member state such as `bid`, `y`, `gid`, `x`.
- **CN:** 声明或更新局部/成员状态，例如 `bid`, `y`, `gid`, `x`。

### Lines 209-217
```cpp
209:     const int   bdimx             = blockDim.x;
210:     const int   s_reduce_elements = prod_dim1_to_last_dim / num_groups;
211:     const int   v_reduce_elements = s_reduce_elements / T_PER_TVec;
212:     const int   s_group_stride    = last_dim / num_groups;
213:     const int   v_group_stride    = s_group_stride / T_PER_TVec;
214:     const int   offset_of_group   = (bid * prod_dim1_to_last_dim + gid * s_group_stride) / T_PER_TVec;
215:     const TVec* input_TVec_ptr    = (const TVec*)(input) + offset_of_group;
216:     TVec*       output_TVec_ptr   = (TVec*)(output) + offset_of_group;
217:     float       local_sum[1]      = {0.0f};
```
- **EN:** Declares or updates local/member state such as `bdimx`, `x`, `s_reduce_elements`, `num_groups`.
- **CN:** 声明或更新局部/成员状态，例如 `bdimx`, `x`, `s_reduce_elements`, `num_groups`。

### Lines 219-219
```cpp
219: #pragma unroll
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 220-227
```cpp
220:     for (int i = 0; i < TVecs_PER_THREAD; i += 1) {
221:         const int current_load_start_idx = (i * bdimx + tid) * T_PER_TVec;
222:         if (current_load_start_idx < s_reduce_elements) {
223:             const int offset_in_group =
224:                 ((current_load_start_idx / s_group_stride) * last_dim + (current_load_start_idx % s_group_stride))
225:                 / T_PER_TVec;
226:             TVec tmp_vec     = input_TVec_ptr[offset_in_group];
227:             T*   tmp_vec_ptr = (T*)(&tmp_vec);
```
- **EN:** Declares or updates local/member state such as `i`, `TVecs_PER_THREAD`, `current_load_start_idx`, `T_PER_TVec`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `TVecs_PER_THREAD`, `current_load_start_idx`, `T_PER_TVec`。

### Lines 228-228
```cpp
228: #pragma unroll
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 229-235
```cpp
229:             for (int j = 0; j < T_PER_TVec; j++) {
230:                 float tmp = static_cast<float>(tmp_vec_ptr[j]);
231:                 local_sum[0] += tmp;
232:             }
233:         }
234:     }
235:     __shared__ float s_mean, s_variance;
```
- **EN:** Declares or updates local/member state such as `j`, `T_PER_TVec`, `tmp`, `s_variance`.
- **CN:** 声明或更新局部/成员状态，例如 `j`, `T_PER_TVec`, `tmp`, `s_variance`。

### Lines 237-237
```cpp
237:     // reduction for mean
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 238-247
```cpp
238:     if (bdimx <= 32) {
239:         warpReduceSum<float, 1>(local_sum);
240:     }
241:     else {
242:         blockReduceSum<float, 1>(local_sum);
243:     }
244:     if (tid == 0) {
245:         s_mean = local_sum[0] / s_reduce_elements;
246:     }
247:     __syncthreads();
```
- **EN:** Declares or updates local/member state such as `tid`, `s_mean`, `s_reduce_elements`.
- **CN:** 声明或更新局部/成员状态，例如 `tid`, `s_mean`, `s_reduce_elements`。

### Lines 249-249
```cpp
249:     // reduction for std
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 250-250
```cpp
250:     local_sum[0] = 0.0f;
```
- **EN:** Supporting logic for the device groupnorm implementation.
- **CN:** device groupnorm实现的辅助逻辑。

### Lines 251-251
```cpp
251: #pragma unroll
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 252-259
```cpp
252:     for (int i = 0; i < TVecs_PER_THREAD; i += 1) {
253:         const int current_load_start_idx = (i * bdimx + tid) * T_PER_TVec;
254:         if (current_load_start_idx < s_reduce_elements) {
255:             const int offset_in_group =
256:                 ((current_load_start_idx / s_group_stride) * last_dim + (current_load_start_idx % s_group_stride))
257:                 / T_PER_TVec;
258:             TVec tmp_vec     = input_TVec_ptr[offset_in_group];
259:             T*   tmp_vec_ptr = (T*)(&tmp_vec);
```
- **EN:** Declares or updates local/member state such as `i`, `TVecs_PER_THREAD`, `current_load_start_idx`, `T_PER_TVec`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `TVecs_PER_THREAD`, `current_load_start_idx`, `T_PER_TVec`。

### Lines 260-260
```cpp
260: #pragma unroll
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 261-274
```cpp
261:             for (int j = 0; j < T_PER_TVec; j++) {
262:                 float tmp = static_cast<float>(tmp_vec_ptr[j]);
263:                 tmp -= s_mean;
264:                 local_sum[0] += tmp * tmp;
265:             }
266:         }
267:     }
268:     if (bdimx <= 32) {
269:         warpReduceSum<float, 1>(local_sum);
270:     }
271:     else {
272:         blockReduceSum<float, 1>(local_sum);
273:     }
274:     if (tid == 0) {
```
- **EN:** Declares or updates local/member state such as `j`, `T_PER_TVec`, `tmp`, `s_mean`.
- **CN:** 声明或更新局部/成员状态，例如 `j`, `T_PER_TVec`, `tmp`, `s_mean`。

### Lines 275-277
```cpp
275:         s_variance = rsqrtf(local_sum[0] / s_reduce_elements + eps);
276:     }
277:     __syncthreads();
```
- **EN:** Implements `rsqrtf` and coordinates helper calls such as `__syncthreads`.
- **CN:** 实现 `rsqrtf`，并协调调用 `__syncthreads` 等辅助逻辑。

### Lines 279-279
```cpp
279:     // normalize
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 280-282
```cpp
280:     const int   gamma_offset_of_group = gid * v_group_stride;
281:     const TVec* gamma_TVec_ptr        = (const TVec*)gamma + gamma_offset_of_group;
282:     const TVec* beta_TVec_ptr         = (const TVec*)beta + gamma_offset_of_group;
```
- **EN:** Declares or updates local/member state such as `gamma_offset_of_group`, `v_group_stride`, `gamma_TVec_ptr`, `beta_TVec_ptr`.
- **CN:** 声明或更新局部/成员状态，例如 `gamma_offset_of_group`, `v_group_stride`, `gamma_TVec_ptr`, `beta_TVec_ptr`。

### Lines 283-283
```cpp
283: #pragma unroll
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 284-297
```cpp
284:     for (int i = 0; i < TVecs_PER_THREAD; i += 1) {
285:         const int current_load_start_idx = (i * bdimx + tid) * T_PER_TVec;
286:         if (current_load_start_idx < s_reduce_elements) {
287:             const int offset_in_group =
288:                 ((current_load_start_idx / s_group_stride) * last_dim + (current_load_start_idx % s_group_stride))
289:                 / T_PER_TVec;
290:             const int gamma_offset_in_group = (current_load_start_idx % s_group_stride) / T_PER_TVec;
291:             TVec      gamma_val             = gamma_TVec_ptr[gamma_offset_in_group];
292:             TVec      beta_val              = beta_TVec_ptr[gamma_offset_in_group];
293:             T*        gamma_val_ptr         = (T*)(&gamma_val);
294:             T*        beta_val_ptr          = (T*)(&beta_val);
295:             TVec      tmp_vec               = input_TVec_ptr[offset_in_group];
296:             T*        tmp_vec_ptr           = (T*)(&tmp_vec);
297:             TVec      output_tmp_vec;
```
- **EN:** Declares or updates local/member state such as `i`, `TVecs_PER_THREAD`, `current_load_start_idx`, `T_PER_TVec`.
- **CN:** 声明或更新局部/成员状态，例如 `i`, `TVecs_PER_THREAD`, `current_load_start_idx`, `T_PER_TVec`。

### Lines 298-298
```cpp
298:             T*        output_tmp_vec_ptr = (T*)(&output_tmp_vec);
```
- **EN:** Declares or updates local/member state such as `output_tmp_vec_ptr`.
- **CN:** 声明或更新局部/成员状态，例如 `output_tmp_vec_ptr`。

### Lines 299-299
```cpp
299: #pragma unroll
```
- **EN:** Preprocessor directives configure compilation behavior for this source file.
- **CN:** 预处理指令用于配置该源文件的编译行为。

### Lines 300-313
```cpp
300:             for (int j = 0; j < T_PER_TVec; j++) {
301:                 float tmp =
302:                     (static_cast<float>(tmp_vec_ptr[j]) - s_mean) * s_variance * static_cast<float>(gamma_val_ptr[j])
303:                     + static_cast<float>(beta_val_ptr[j]);
304:                 if (sizeof(T) == sizeof(half)) {
305:                     output_tmp_vec_ptr[j] = T(__float2half_rn(tmp));
306:                 }
307:                 else {
308:                     output_tmp_vec_ptr[j] = T(tmp);
309:                 }
310:             }
311:             output_TVec_ptr[offset_in_group] = output_tmp_vec;
312:         }
313:     }
```
- **EN:** Declares or updates local/member state such as `j`, `T_PER_TVec`, `tmp`, `output_tmp_vec`.
- **CN:** 声明或更新局部/成员状态，例如 `j`, `T_PER_TVec`, `tmp`, `output_tmp_vec`。

### Lines 314-314
```cpp
314: }
```
- **EN:** Supporting logic for the device groupnorm implementation.
- **CN:** device groupnorm实现的辅助逻辑。

### Lines 316-317
```cpp
316: //ref_input & ref_output should be [N, H, W, C]
317: //ref_gamma & ref_beta should be [1, 1, 1, C]
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 318-331
```cpp
318: template <typename T>
319: void groupnorm(cutlass::Tensor4DCoord input_size,
320:                const int num_groups,
321:                const float eps,
322:                TensorRef<T, layout::TensorNHWC> ref_output,
323:                TensorRef<T, layout::TensorNHWC> ref_input,
324:                TensorRef<T, layout::TensorNHWC> ref_gamma,
325:                TensorRef<T, layout::TensorNHWC> ref_beta,
326:                cudaStream_t stream){
327:   const int N = input_size.n();
328:   const int H = input_size.h();
329:   const int W = input_size.w();
330:   const int C = input_size.c();
331:   if (C % num_groups != 0){
```
- **EN:** Declares or updates local/member state such as `N`, `H`, `W`, `C`.
- **CN:** 声明或更新局部/成员状态，例如 `N`, `H`, `W`, `C`。

### Lines 332-337
```cpp
332:     printf("[ERROR] C should be a multiple of num_groups.\n");
333:   }
334:   T* output = ref_output.data();
335:   const T* input = ref_input.data();
336:   const T* gamma = ref_gamma.data();
337:   const T* beta = ref_beta.data();
```
- **EN:** Emits debug or diagnostic output so intermediate state can be inspected.
- **CN:** 输出调试或诊断信息，便于检查中间状态。

### Lines 339-352
```cpp
339:   const int dim0 = N;
340:   const int last_dim = C;
341:   const int prod_dim1_to_last_dim = H*W*C;
342:   const int s_reduce_elements = prod_dim1_to_last_dim / num_groups;
343:   const int s_group_stride = last_dim / num_groups;
344:   dim3      grid(num_groups, dim0);
345:   int       threadblock_size = 32;
346:   if (s_group_stride % 2 == 0) {
347:     const int T_PER_TVec = 2;
348:     while (threadblock_size < 1024) {
349:       if (s_reduce_elements / T_PER_TVec / threadblock_size <= 8)
350:         break;
351:         threadblock_size *= 2;
352:       }
```
- **EN:** Declares or updates local/member state such as `dim0`, `N`, `last_dim`, `C`.
- **CN:** 声明或更新局部/成员状态，例如 `dim0`, `N`, `last_dim`, `C`。

### Lines 353-355
```cpp
353:     dim3      block(threadblock_size);
354:     const int TVec_PER_THREAD = (s_reduce_elements / T_PER_TVec + threadblock_size - 1) / threadblock_size;
355:     const int shm_size = T_PER_TVec * TVec_PER_THREAD * threadblock_size * sizeof(T);
```
- **EN:** Implements `block` for this file's main component.
- **CN:** 为该文件的核心组件实现 `block`。

### Lines 356-358
```cpp
356:     // for small s_reduce_elements, specific case for H=W=22, C=1280, num_groups=32;
357:     // the size of grid & block may have better choice for different cases.
358:     // ensure shared memory is smaller than 48KB
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 359-372
```cpp
359:     if (std::is_same<T, float>::value){
360:       if (shm_size < 48 * 1024) {
361:         groupnorm_twopass_store_locally<float2, T, T_PER_TVec><<<grid, block, shm_size, stream>>>(
362:           output, input, gamma, beta, num_groups, prod_dim1_to_last_dim, last_dim, eps, TVec_PER_THREAD);
363:       }
364:       else {
365:         groupnorm_twopass_multiple_load<float2, T, T_PER_TVec><<<grid, block, 0, stream>>>(
366:           output, input, gamma, beta, num_groups, prod_dim1_to_last_dim, last_dim, eps, TVec_PER_THREAD);
367:       }
368:     }
369:     else{
370:       if (shm_size < 48 * 1024) {
371:         groupnorm_twopass_store_locally<half2, T, T_PER_TVec><<<grid, block, shm_size, stream>>>(
372:           output, input, gamma, beta, num_groups, prod_dim1_to_last_dim, last_dim, eps, TVec_PER_THREAD);
```
- **EN:** Supporting logic for the device groupnorm implementation.
- **CN:** device groupnorm实现的辅助逻辑。

### Lines 373-377
```cpp
373:       }
374:       else {
375:         groupnorm_twopass_multiple_load<half2, T, T_PER_TVec><<<grid, block, 0, stream>>>(
376:           output, input, gamma, beta, num_groups, prod_dim1_to_last_dim, last_dim, eps, TVec_PER_THREAD);
377:       }
```
- **EN:** Supporting logic for the device groupnorm implementation.
- **CN:** device groupnorm实现的辅助逻辑。

### Lines 378-391
```cpp
378:     }
379:   }
380:   else {
381:     const int T_PER_TVec = 1;
382:     while (threadblock_size < 1024) {
383:       if (s_reduce_elements / T_PER_TVec / threadblock_size <= 8)
384:         break;
385:         threadblock_size *= 2;
386:       }
387:     dim3      block(threadblock_size);
388:     const int TVec_PER_THREAD = (s_reduce_elements / T_PER_TVec + threadblock_size - 1) / threadblock_size;
389:     const int shm_size = T_PER_TVec * TVec_PER_THREAD * threadblock_size * sizeof(T);
390:     if (shm_size < 48 * 1024) {
391:       groupnorm_twopass_store_locally<T, T, T_PER_TVec><<<grid, block, shm_size, stream>>>(
```
- **EN:** Declares or updates local/member state such as `T_PER_TVec`, `break`, `TVec_PER_THREAD`, `threadblock_size`.
- **CN:** 声明或更新局部/成员状态，例如 `T_PER_TVec`, `break`, `TVec_PER_THREAD`, `threadblock_size`。

### Lines 392-397
```cpp
392:         output, input, gamma, beta, num_groups, prod_dim1_to_last_dim, last_dim, eps, TVec_PER_THREAD);
393:     }
394:     else {
395:       groupnorm_twopass_multiple_load<T, T, T_PER_TVec><<<grid, block, 0, stream>>>(
396:         output, input, gamma, beta, num_groups, prod_dim1_to_last_dim, last_dim, eps, TVec_PER_THREAD);
397:     }
```
- **EN:** Supporting logic for the device groupnorm implementation.
- **CN:** device groupnorm实现的辅助逻辑。

### Lines 398-398
```cpp
398:   }
```
- **EN:** Supporting logic for the device groupnorm implementation.
- **CN:** device groupnorm实现的辅助逻辑。

### Lines 400-400
```cpp
400: }
```
- **EN:** Supporting logic for the device groupnorm implementation.
- **CN:** device groupnorm实现的辅助逻辑。

### Lines 402-402
```cpp
402: } //namespace cutlass
```
- **EN:** Supporting logic for the device groupnorm implementation.
- **CN:** device groupnorm实现的辅助逻辑。

## Key Concepts / 关键概念
- **Template metaprogramming / 模板元编程**
- **CUDA ecosystem integration / CUDA 生态集成**
- **Tensor manipulation / 张量处理**
- **Reduction support / 归约支持**

## Dependencies / 依赖关系
- **Internal headers / 内部头文件:** `cutlass/cutlass.h`, `cutlass/layout/tensor.h`, `cutlass/numeric_types.h`, `cutlass/tensor_coord.h`, `cutlass/tensor_ref.h`
- **External headers / 外部头文件:** `device_utils.h`, `cfloat`
- **Runtime/backends / 运行时与后端:** `CUDA`
