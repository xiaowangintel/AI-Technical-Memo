# device_nhwc_padding.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `tools/util/include/cutlass/util/device_nhwc_padding.h`
- **Purpose (EN):** This file declares device nhwc padding for general CUTLASS utility support.
- **目的 (CN):** 该文件声明了面向CUTLASS 通用工具支持的device nhwc padding逻辑。
- **Brief / 简述:** cuda kernels for padding in device memory with NHWC layout.

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
36:  * \brief cuda kernels for padding in device memory with NHWC layout.
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
- **EN:** Supporting logic for the device nhwc padding implementation.
- **CN:** device nhwc padding实现的辅助逻辑。

### Lines 47-49
```cpp
47: /** \brief interface for padding in a device memory tensor with NHWC layout
48:  * \tparam T: data type
49:  */
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 50-55
```cpp
50: template <typename T>
51: void nhwc_padding(cutlass::Tensor4DCoord input_tensor_size,
52:                   cutlass::Tensor4DCoord output_tensor_size,
53:                   TensorRef<T, layout::TensorNHWC> ref_input,
54:                   TensorRef<T, layout::TensorNHWC> ref_output,
55:                   cudaStream_t stream);
```
- **EN:** Supporting logic for the device nhwc padding implementation.
- **CN:** device nhwc padding实现的辅助逻辑。

### Lines 58-66
```cpp
58: template <typename T>
59: __global__ void nhwc_padding_kernel(const int32_t n,
60:                                     const int32_t h,
61:                                     const int32_t w,
62:                                     const int32_t c_in,
63:                                     const int32_t c_out,
64:                                     const T zero,
65:                                     const T *input,
66:                                     T *output){
```
- **EN:** Supporting logic for the device nhwc padding implementation.
- **CN:** device nhwc padding实现的辅助逻辑。

### Lines 68-69
```cpp
68:   const int32_t idx_jump       = blockDim.x * gridDim.x;
69:   const int32_t total_elements = n * h * w * c_out;
```
- **EN:** Declares or updates local/member state such as `idx_jump`, `x`, `total_elements`, `c_out`.
- **CN:** 声明或更新局部/成员状态，例如 `idx_jump`, `x`, `total_elements`, `c_out`。

### Lines 71-71
```cpp
71:   int32_t c_idx, w_idx, h_idx, n_idx, resudial;
```
- **EN:** Declares or updates local/member state such as `resudial`.
- **CN:** 声明或更新局部/成员状态，例如 `resudial`。

### Lines 73-74
```cpp
73:   T value;
74:   for (int32_t idx = blockIdx.x * blockDim.x + threadIdx.x; idx < total_elements; idx += idx_jump) {
```
- **EN:** Declares or updates local/member state such as `value`, `idx`, `x`, `total_elements`.
- **CN:** 声明或更新局部/成员状态，例如 `value`, `idx`, `x`, `total_elements`。

### Lines 76-89
```cpp
76:     c_idx = idx%c_out;
77:     if (c_idx >= c_in){
78:       value = zero;    
79:     }
80:     else{
81:       resudial = idx/c_out;
82:       w_idx = resudial%w;
83:       resudial = resudial/w;
84:       h_idx = resudial%h;
85:       n_idx = resudial/h;	
86:       resudial = ((n_idx * h + h_idx) * w + w_idx) * c_in + c_idx;
87:       value = input[resudial];
88:     }
89:     output[idx] = value;
```
- **EN:** Declares or updates local/member state such as `c_idx`, `c_out`, `value`, `zero`.
- **CN:** 声明或更新局部/成员状态，例如 `c_idx`, `c_out`, `value`, `zero`。

### Lines 90-91
```cpp
90:   }
91: }
```
- **EN:** Supporting logic for the device nhwc padding implementation.
- **CN:** device nhwc padding实现的辅助逻辑。

### Lines 94-94
```cpp
94: // fast kernel for c_in = 3 & c_out = 4
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 95-107
```cpp
95: template <typename Tio, typename Telement, int element_in_Tio>
96: __global__ void nhwc_padding_channel_3To4_kernel(const int32_t n,
97:                                                  const int32_t h,
98:                                                  const int32_t w,
99:                                                  const Tio *input,
100:                                                  Tio *output,
101:                                                  const int32_t max_output_element,
102:                                                  const int32_t max_input_element,
103:                                                  const Tio zero_io,
104:                                                  const Telement zero_element){                                                
105:   __shared__ Tio shm[192];
106:   const int tidx = blockIdx.x * 192 + threadIdx.x;  
107:   const int threadidx = threadIdx.x; 
```
- **EN:** Declares or updates local/member state such as `tidx`, `x`, `threadidx`.
- **CN:** 声明或更新局部/成员状态，例如 `tidx`, `x`, `threadidx`。

### Lines 109-110
```cpp
109:   shm[threadIdx.x] = tidx >= max_input_element ? zero_io : input[tidx];  
110:   __syncthreads();
```
- **EN:** Implements `__syncthreads` for this file's main component.
- **CN:** 为该文件的核心组件实现 `__syncthreads`。

### Lines 112-123
```cpp
112:   const int output_offset = blockIdx.x * 256;
113:   const int lower_bound = max_output_element < output_offset + 256 ? max_output_element : output_offset + 256;
114:   for (int i = output_offset + threadidx, j = threadidx ; i < lower_bound ; i+=192, j+=192)
115:   {
116:     const Telement* shm_element = (const Telement*)shm + j*3*element_in_Tio/4;
117:     Telement array[element_in_Tio];
118:     CUTLASS_PRAGMA_UNROLL
119:     for (int k = 0 ; k < element_in_Tio ; k++)
120:       array[k] = ((k+1)%4 == 0) ? zero_element : shm_element[(k > 3) ? (k - 1) : k];
121:     output[i] = *((const Tio *)array);
122:   }
123: }
```
- **EN:** Declares or updates local/member state such as `output_offset`, `lower_bound`, `i`, `j`.
- **CN:** 声明或更新局部/成员状态，例如 `output_offset`, `lower_bound`, `i`, `j`。

### Lines 125-125
```cpp
125: // fast kernel for c_in = 3 & c_out = 8
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 126-138
```cpp
126: template <typename Tio, typename Telement, int element_in_Tio>
127: __global__ void nhwc_padding_channel_3To8_kernel(const int32_t n,
128:                                                  const int32_t h,
129:                                                  const int32_t w,
130:                                                  const Tio *input,
131:                                                  Tio *output,
132:                                                  const int32_t max_output_element,
133:                                                  const int32_t max_input_element,
134:                                                  const Tio zero_io,
135:                                                  const Telement zero_element){                                                
136:   __shared__ Tio shm[192];
137:   const int tidx = blockIdx.x * 192 + threadIdx.x;  
138:   const int threadidx = threadIdx.x; 
```
- **EN:** Declares or updates local/member state such as `tidx`, `x`, `threadidx`.
- **CN:** 声明或更新局部/成员状态，例如 `tidx`, `x`, `threadidx`。

### Lines 140-141
```cpp
140:   shm[threadIdx.x] = tidx >= max_input_element ? zero_io : input[tidx];  
141:   __syncthreads();
```
- **EN:** Implements `__syncthreads` for this file's main component.
- **CN:** 为该文件的核心组件实现 `__syncthreads`。

### Lines 143-148
```cpp
143:   const int output_offset = blockIdx.x * 512;
144:   const int lower_bound = max_output_element < output_offset + 512 ? max_output_element : output_offset + 512;
145:   for (int i = output_offset + threadidx, j = threadidx ; i < lower_bound ; i+=192, j+=192)
146:   {
147:     const Telement* shm_element = (const Telement*)shm + (element_in_Tio == 4 ? j/2 : j)*3;
148:     Telement array[element_in_Tio];
```
- **EN:** Declares or updates local/member state such as `output_offset`, `lower_bound`, `i`, `j`.
- **CN:** 声明或更新局部/成员状态，例如 `output_offset`, `lower_bound`, `i`, `j`。

### Lines 149-149
```cpp
149:     //float
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 150-154
```cpp
150:     if (element_in_Tio == 4){
151:       CUTLASS_PRAGMA_UNROLL
152:       for (int k = 0 ; k < element_in_Tio ; k++)
153:         array[k] = ((j % 2) == 1) ? zero_element : ((k >= 3) ? zero_element : shm_element[k]);
154:     }
```
- **EN:** Declares or updates local/member state such as `element_in_Tio`, `k`.
- **CN:** 声明或更新局部/成员状态，例如 `element_in_Tio`, `k`。

### Lines 155-155
```cpp
155:     //half
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 156-163
```cpp
156:     else{
157:       CUTLASS_PRAGMA_UNROLL
158:       for (int k = 0 ; k < element_in_Tio ; k++) 
159:         array[k] = (k >= 3) ? zero_element : shm_element[k];          
160:     }
161:     output[i] = *((const Tio *)array);
162:   }
163: }
```
- **EN:** Declares or updates local/member state such as `k`, `element_in_Tio`.
- **CN:** 声明或更新局部/成员状态，例如 `k`, `element_in_Tio`。

### Lines 165-175
```cpp
165: template <typename T>
166: void nhwc_padding(cutlass::Tensor4DCoord input_tensor_size,
167:                   cutlass::Tensor4DCoord output_tensor_size,
168:                   TensorRef<T, layout::TensorNHWC> ref_input,
169:                   TensorRef<T, layout::TensorNHWC> ref_output,
170:                   cudaStream_t stream){
171:   assert(
172:     input_tensor_size.n() == output_tensor_size.n() &&
173:     input_tensor_size.h() == output_tensor_size.h() &&
174:     input_tensor_size.w() == output_tensor_size.w() &&
175:     input_tensor_size.c() <= output_tensor_size.c()); 
```
- **EN:** Supporting logic for the device nhwc padding implementation.
- **CN:** device nhwc padding实现的辅助逻辑。

### Lines 177-181
```cpp
177:   int n = input_tensor_size.n();
178:   int h = input_tensor_size.h();
179:   int w = input_tensor_size.w();
180:   int c_in = input_tensor_size.c();
181:   int c_out = output_tensor_size.c();
```
- **EN:** Implements `n` and coordinates helper calls such as `h`, `w`, `c`.
- **CN:** 实现 `n`，并协调调用 `h`, `w`, `c` 等辅助逻辑。

### Lines 183-183
```cpp
183:   //case 1 : channel == 3 padding to 4 or 8
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 184-187
```cpp
184:   if ((c_out == 4 || c_out == 8) && c_in == 3 && (n*h*w % 8 == 0)){
185:     dim3 block(192);
186:     const int nhw = n*h*w;
187:     const int nhwc = nhw*c_in;
```
- **EN:** Declares or updates local/member state such as `c_out`, `c_in`, `nhw`, `w`.
- **CN:** 声明或更新局部/成员状态，例如 `c_out`, `c_in`, `nhw`, `w`。

### Lines 188-188
```cpp
188:     //for half_t
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 189-202
```cpp
189:     if (cutlass::sizeof_bits<T>::value == 16){
190:       const int element_in_Tio = 8;
191:       const int max_input_element = nhwc/element_in_Tio;
192:       const int max_output_element = nhw*c_out/element_in_Tio;
193:       const int4 zero_io = {0, 0, 0, 0};
194:       const half_t zero_element = static_cast<half_t>(0.0f);
195:       dim3 grid((nhwc + 192*element_in_Tio - 1)/(192*element_in_Tio));
196:       if (c_out == 4){
197:         nhwc_padding_channel_3To4_kernel<int4, half_t, element_in_Tio><<<grid, block, 0, stream>>>
198:           (n, h, w,
199:           (const int4 *)ref_input.data(),
200:           (int4 *)ref_output.data(),
201:           max_output_element,
202:           max_input_element,
```
- **EN:** Declares or updates local/member state such as `value`, `element_in_Tio`, `max_input_element`, `max_output_element`.
- **CN:** 声明或更新局部/成员状态，例如 `value`, `element_in_Tio`, `max_input_element`, `max_output_element`。

### Lines 203-206
```cpp
203:           zero_io,
204:           zero_element);
205:       }
206:       else if (c_out == 8){
```
- **EN:** Declares or updates local/member state such as `c_out`.
- **CN:** 声明或更新局部/成员状态，例如 `c_out`。

### Lines 207-216
```cpp
207:         nhwc_padding_channel_3To8_kernel<int4, half_t, element_in_Tio><<<grid, block, 0, stream>>>
208:           (n, h, w,
209:           (const int4 *)ref_input.data(),
210:           (int4 *)ref_output.data(),
211:           max_output_element,
212:           max_input_element,
213:           zero_io,
214:           zero_element);
215:       }
216:     }
```
- **EN:** Supporting logic for the device nhwc padding implementation.
- **CN:** device nhwc padding实现的辅助逻辑。

### Lines 217-217
```cpp
217:     //for float
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 218-231
```cpp
218:     else{
219:       const int element_in_Tio = 4;
220:       const int max_input_element = nhwc/element_in_Tio;
221:       const int max_output_element = nhw*c_out/element_in_Tio;
222:       const float4 zero_io = {0.0f, 0.0f, 0.0f, 0.0f};
223:       const float zero_element = 0.0f;
224:       dim3 grid((nhwc + 192*element_in_Tio - 1)/(192*element_in_Tio));
225:       if (c_out == 4){
226:         nhwc_padding_channel_3To4_kernel<float4, float, element_in_Tio><<<grid, block, 0, stream>>>
227:           (n, h, w,
228:           (const float4 *)ref_input.data(),
229:           (float4 *)ref_output.data(),
230:           max_output_element,
231:           max_input_element,
```
- **EN:** Declares or updates local/member state such as `element_in_Tio`, `max_input_element`, `max_output_element`, `zero_io`.
- **CN:** 声明或更新局部/成员状态，例如 `element_in_Tio`, `max_input_element`, `max_output_element`, `zero_io`。

### Lines 232-235
```cpp
232:           zero_io,
233:           zero_element);
234:       }
235:       else if (c_out == 8){
```
- **EN:** Declares or updates local/member state such as `c_out`.
- **CN:** 声明或更新局部/成员状态，例如 `c_out`。

### Lines 236-246
```cpp
236:         nhwc_padding_channel_3To8_kernel<float4, float, element_in_Tio><<<grid, block, 0, stream>>>
237:           (n, h, w,
238:           (const float4 *)ref_input.data(),
239:           (float4 *)ref_output.data(),
240:           max_output_element,
241:           max_input_element,
242:           zero_io,
243:           zero_element);
244:       }
245:     }
246:   }
```
- **EN:** Supporting logic for the device nhwc padding implementation.
- **CN:** device nhwc padding实现的辅助逻辑。

### Lines 247-247
```cpp
247:   //case 2 : even channel
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 248-252
```cpp
248:   else if ((c_out % 2) == 0 && (c_in % 2) == 0){
249:     int32_t total_elements = n * h * w * c_out / 2;
250:     int block_size = 256;
251:     dim3 grid((total_elements + 255)/256);
252:     dim3 block(block_size);
```
- **EN:** Declares or updates local/member state such as `total_elements`, `block_size`.
- **CN:** 声明或更新局部/成员状态，例如 `total_elements`, `block_size`。

### Lines 253-253
```cpp
253:     //for half_t
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 254-257
```cpp
254:     if (cutlass::sizeof_bits<T>::value == 16){
255:       const __half2 zero  = {0.0f, 0.0f};
256:       nhwc_padding_kernel<<<grid, block, 0, stream>>>(n, h, w, c_in/2, c_out/2, zero, (const __half2*)ref_input.data(), (__half2*)ref_output.data());
257:     }
```
- **EN:** Declares or updates local/member state such as `value`, `zero`.
- **CN:** 声明或更新局部/成员状态，例如 `value`, `zero`。

### Lines 258-258
```cpp
258:     //for float
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 259-263
```cpp
259:     else{
260:       const float2 zero  = {0.0f, 0.0f};
261:       nhwc_padding_kernel<<<grid, block, 0, stream>>>(n, h, w, c_in/2, c_out/2, zero, (const float2*)ref_input.data(), (float2*)ref_output.data());
262:     }
263:   }
```
- **EN:** Declares or updates local/member state such as `zero`.
- **CN:** 声明或更新局部/成员状态，例如 `zero`。

### Lines 264-264
```cpp
264:   //case 3 : odd channel
```
- **EN:** Comment block that explains nearby logic or records contextual notes.
- **CN:** 注释块，用于解释附近逻辑或记录上下文说明。

### Lines 265-273
```cpp
265:   else{
266:     int32_t total_elements = n * h * w * c_out;
267:     int block_size = 256;
268:     dim3 grid((total_elements + 255)/256);
269:     dim3 block(block_size);
270:     const T zero = static_cast<T>(0.0f);
271:     nhwc_padding_kernel<<<grid, block, 0, stream>>>(n, h, w, c_in, c_out, zero, ref_input.data(), ref_output.data());
272:   }
273: }
```
- **EN:** Declares or updates local/member state such as `total_elements`, `c_out`, `block_size`, `zero`.
- **CN:** 声明或更新局部/成员状态，例如 `total_elements`, `c_out`, `block_size`, `zero`。

### Lines 276-276
```cpp
276: } //namespace cutlass
```
- **EN:** Supporting logic for the device nhwc padding implementation.
- **CN:** device nhwc padding实现的辅助逻辑。

## Key Concepts / 关键概念
- **Template metaprogramming / 模板元编程**
- **CUDA ecosystem integration / CUDA 生态集成**
- **Tensor manipulation / 张量处理**

## Dependencies / 依赖关系
- **Internal headers / 内部头文件:** `cutlass/cutlass.h`, `cutlass/layout/tensor.h`, `cutlass/numeric_types.h`, `cutlass/tensor_coord.h`, `cutlass/tensor_ref.h`
- **Runtime/backends / 运行时与后端:** `CUDA`
