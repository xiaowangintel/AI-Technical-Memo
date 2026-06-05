# 8x4c1x4-dq-packedA-sse2.h — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/quantized/cpu/qnnpack/src/q8gemm_sparse/8x4c1x4-dq-packedA-sse2.h`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU linear algebra or matrix-multiplication support paths in the ATen native quantized CPU path.
- 用途 (CN): 该文件在 ATen 原生量化 CPU 路径中实现或声明了CPU 线性代数或矩阵乘法支持路径。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28
```cpp
 1: /*
 2:  * Copyright (c) Facebook, Inc. and its affiliates.
 3:  * All rights reserved.
 4:  *
 5:  * This source code is licensed under the BSD-style license found in the
 6:  * LICENSE file in the root directory of this source tree.
 7:  */
 8:
 9: #include <immintrin.h>
10:
11: #include <qnnpack/q8gemm_sparse.h>
12: #include <requantization/runtime-sse2.h>
13:
14: #include "8x4c1x4-packed-sse2.h"
15:
16: #define CONVERT_TO_FP_AND_TRANSPOSE(a, b, c, d, t_a, t_b, t_c, t_d)  \
17:   a_ps = _mm_cvtepi32_ps(a);                                         \
18:   b_ps = _mm_cvtepi32_ps(b);                                         \
19:   c_ps = _mm_cvtepi32_ps(c);                                         \
20:   d_ps = _mm_cvtepi32_ps(d);                                         \
21:   tmp0 = _mm_shuffle_ps(a_ps, b_ps, _MM_SHUFFLE(1, 0, 1, 0));        \
22:   tmp1 = _mm_shuffle_ps(a_ps, b_ps, _MM_SHUFFLE(3, 2, 3, 2));        \
23:   tmp2 = _mm_shuffle_ps(c_ps, d_ps, _MM_SHUFFLE(1, 0, 1, 0));        \
24:   tmp3 = _mm_shuffle_ps(c_ps, d_ps, _MM_SHUFFLE(3, 2, 3, 2));        \
25:   t_a = _mm_shuffle_ps(tmp0, tmp2, _MM_SHUFFLE(2, 0, 2, 0));         \
26:   t_b = _mm_shuffle_ps(tmp0, tmp2, _MM_SHUFFLE(3, 1, 3, 1));         \
27:   t_c = _mm_shuffle_ps(tmp1, tmp3, _MM_SHUFFLE(2, 0, 2, 0));         \
28:   t_d = _mm_shuffle_ps(tmp1, tmp3, _MM_SHUFFLE(3, 1, 3, 1));
```
- EN: This range pulls in required headers, including `immintrin.h`, `qnnpack/q8gemm_sparse.h`, `requantization/runtime-sse2.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段引入了所需头文件，例如 `immintrin.h`, `qnnpack/q8gemm_sparse.h`, `requantization/runtime-sse2.h`。 它还建立了编译期开关、特性选择或平台相关分支。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 30-47
```cpp
30: // KERNEL_NAME and W_INDEX_DTYPE macros are defined in
31: // https://github.com/pytorch/pytorch/blob/master/aten/src/ATen/native/quantized/cpu/qnnpack/src/q8gemm_sparse/8x4c1x4-dq-packedA-sse2.c
32: void KERNEL_NAME(
33:     size_t mr,
34:     size_t nr,
35:     const uint8_t* a_packed,
36:     const uint8_t* packed_w,
37:     const W_INDEX_DTYPE* w_row_ptr,
38:     const W_INDEX_DTYPE* w_block_ids_ptr,
39:     const float* b,
40:     float* c,
41:     size_t c_stride,
42:     size_t output_channel_index,
43:     const struct pytorch_qnnp_conv_dynamic_quantization_params
44:         quantization_params[RESTRICT_STATIC 1]) {
45:   const __m128i va_zero_point = _mm_set1_epi16(quantization_params->input_zero_point);
46:   const __m128 vbias = _mm_load_ps(b);
47:   const __m128i vzero = _mm_setzero_si128();
```
- EN: The main symbol in this range is `pytorch_qnnp_conv_dynamic_quantization_params`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range. The math and shape handling relate to convolution-style operators.
- CN: 这一段的主要符号是 `pytorch_qnnp_conv_dynamic_quantization_params`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。 这里的计算与形状处理与卷积类算子相关。

### Lines 49-71
```cpp
49:   // Packed A format.
50:   // 8kx4m blocks for all blocks given 4 rows (4m) are placed in contiguous memory.
51:   // Original A
52:   // --------- K -----------          -- (K + 4 - 1) / 4 --
53:   // |                     |          |                   |
54:   // |                     |        (M + 8 - 1)/8         |
55:   // |                     | Packed   |                   |
56:   // M                     |  =>      |-------------------|
57:   // |                     |        Thus Packed A has (K + 4 - 1)/4 * (M + 8 -1)/8 blocks
58:   // |                     |
59:   // |---------------------|
60:   //
61:   // Each 8 x 4 blocks is transposed and stored.
62:   // Each of the (K + 4 - 1)/4 blocks for a given group of 8 m blocks
63:   // are stored adjacent in memory
64:   // Thus, each block:
65:   // |----8m-----|----8m-----|
66:   // 4k          |           | .....
67:   // |-----------|-----------|
68:   // This locality helps in loading 8kx8m blocks of activations
69:   // Note when M is not multiple of 8, the rest can contain arbitrary
70:   // data in packed A as we will not be writing those out.
71:   // This will be taken care by just copying the appropriate valid data
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 73-91
```cpp
73:   __m128i vacc_low[4];
74:   __m128i vacc_high[4];
75:   const __m128 vmultiplier =
76:       _mm_loadu_ps(&quantization_params->multipliers[output_channel_index]);
77:   for (int32_t n = 0; n < nr; n++) {
78:     vacc_low[n] = _mm_setzero_si128();
79:     vacc_high[n] = _mm_setzero_si128();
80:     const int16_t b_zero_point =
81:       (int16_t)(uint16_t)quantization_params->kernel_zero_points[
82:       output_channel_index + n];
83:
84:     int32_t num_blocks = w_row_ptr[n+1] - w_row_ptr[n];
85:     // Offset into compressed values.
86:     // w_row_ptr[0] is the block offset in the compressed values.
87:     // Where the corresponding row of the weight matrix starts.
88:     const uint8_t* temp_packed_w = packed_w + w_row_ptr[n] * COL_BLOCK_SIZE;
89:     // Similarly w_row_ptr[0] is also the block offset where
90:     // corresponding row's block column ids start.
91:     // Per row # of block column ids = # of block values
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 92-118
```cpp
 92:     const W_INDEX_DTYPE* temp_w_block_ids_ptr = w_block_ids_ptr + w_row_ptr[n];
 93:     while (num_blocks > 1) {
 94:       // Load two 1x4 uint8 blocks 2 ints
 95:       const uint8_t* b_ptr = temp_packed_w;
 96:       // This is not perf optimal since this will result in
 97:       // register spills. We probably should work with output block
 98:       // of 1x4 instead of 1x8
 99:       // But doing is this way because mostly this how we will
100:       // do it for ARM and this reference code helps establish
101:       // the baseline for functional correctness.
102:       const int16_t b_0 = (int16_t)((uint16_t)(b_ptr[0]));
103:       const int16_t b_1 = (int16_t)((uint16_t)(b_ptr[1]));
104:       const int16_t b_2 = (int16_t)((uint16_t)(b_ptr[2]));
105:       const int16_t b_3 = (int16_t)((uint16_t)(b_ptr[3]));
106:       const int16_t b_4 = (int16_t)((uint16_t)(b_ptr[4]));
107:       const int16_t b_5 = (int16_t)((uint16_t)(b_ptr[5]));
108:       const int16_t b_6 = (int16_t)((uint16_t)(b_ptr[6]));
109:       const int16_t b_7 = (int16_t)((uint16_t)(b_ptr[7]));
110:       // Now we will load 8kx1(broadcast 8) weight values
111:       const __m128i vxb0 = _mm_set1_epi16((b_0 - b_zero_point));
112:       const __m128i vxb1 = _mm_set1_epi16((b_1 - b_zero_point));
113:       const __m128i vxb2 = _mm_set1_epi16((b_2 - b_zero_point));
114:       const __m128i vxb3 = _mm_set1_epi16((b_3 - b_zero_point));
115:       const __m128i vxb4 = _mm_set1_epi16((b_4 - b_zero_point));
116:       const __m128i vxb5 = _mm_set1_epi16((b_5 - b_zero_point));
117:       const __m128i vxb6 = _mm_set1_epi16((b_6 - b_zero_point));
118:       const __m128i vxb7 = _mm_set1_epi16((b_7 - b_zero_point));
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 120-138
```cpp
120:       // Load activation blocks. In this kernel we assume
121:       // a mat is already transposed. K x M
122:       // 1. Load 8 1x8 registers = 8k x 8m
123:
124:       // Load column id of the first 1x4 block
125:       int32_t col_block_id_0 = temp_w_block_ids_ptr[0];
126:       // Load column id of the second 1x4 block
127:       int32_t col_block_id_1 = temp_w_block_ids_ptr[1];
128:       const __m128i va0 =
129:         _mm_loadl_epi64((const __m128i*) (a_packed +
130:             col_block_id_0 * PACKED_A_BLOCK_SIZE + MR * 0));
131:       const __m128i va1 =
132:         _mm_loadl_epi64((const __m128i*) (a_packed +
133:             col_block_id_0 * PACKED_A_BLOCK_SIZE + MR * 1));
134:       const __m128i va2 =
135:         _mm_loadl_epi64((const __m128i*) (a_packed +
136:             col_block_id_0 * PACKED_A_BLOCK_SIZE + MR * 2));
137:       const __m128i va3 =
138:         _mm_loadl_epi64((const __m128i*) (a_packed +
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 139-158
```cpp
139:             col_block_id_0 * PACKED_A_BLOCK_SIZE + MR * 3));
140:       const __m128i va4 =
141:         _mm_loadl_epi64((const __m128i*) (a_packed +
142:             col_block_id_1 * PACKED_A_BLOCK_SIZE + MR * 0));
143:       const __m128i va5 =
144:         _mm_loadl_epi64((const __m128i*) (a_packed +
145:             col_block_id_1 * PACKED_A_BLOCK_SIZE + MR * 1));
146:       const __m128i va6 =
147:         _mm_loadl_epi64((const __m128i*) (a_packed +
148:             col_block_id_1 * PACKED_A_BLOCK_SIZE + MR * 2));
149:       const __m128i va7 =
150:         _mm_loadl_epi64((const __m128i*) (a_packed +
151:             col_block_id_1 * PACKED_A_BLOCK_SIZE + MR * 3));
152:
153:       const __m128i vxa0 =
154:           sub_zero_point(_mm_unpacklo_epi8(va0, vzero), va_zero_point);
155:       const __m128i vxa1 =
156:           sub_zero_point(_mm_unpacklo_epi8(va1, vzero), va_zero_point);
157:       const __m128i vxa2 =
158:           sub_zero_point(_mm_unpacklo_epi8(va2, vzero), va_zero_point);
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 159-178
```cpp
159:       const __m128i vxa3 =
160:           sub_zero_point(_mm_unpacklo_epi8(va3, vzero), va_zero_point);
161:       const __m128i vxa4 =
162:           sub_zero_point(_mm_unpacklo_epi8(va4, vzero), va_zero_point);
163:       const __m128i vxa5 =
164:           sub_zero_point(_mm_unpacklo_epi8(va5, vzero), va_zero_point);
165:       const __m128i vxa6 =
166:           sub_zero_point(_mm_unpacklo_epi8(va6, vzero), va_zero_point);
167:       const __m128i vxa7 =
168:           sub_zero_point(_mm_unpacklo_epi8(va7, vzero), va_zero_point);
169:
170:       // acc += a0 * b0;
171:       __m128i vacc_low_16bits = _mm_mullo_epi16(vxa0, vxb0);
172:       __m128i vacc_high_16bits = _mm_mulhi_epi16(vxa0, vxb0);
173:       vacc_low[n] = _mm_add_epi32(vacc_low[n],
174:         _mm_unpacklo_epi16(vacc_low_16bits, vacc_high_16bits));
175:       vacc_high[n] = _mm_add_epi32(vacc_high[n],
176:         _mm_unpackhi_epi16(vacc_low_16bits, vacc_high_16bits));
177:       // acc += a1 * b1;
178:       vacc_low_16bits = _mm_mullo_epi16(vxa1, vxb1);
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 179-198
```cpp
179:       vacc_high_16bits = _mm_mulhi_epi16(vxa1, vxb1);
180:       vacc_low[n] = _mm_add_epi32(vacc_low[n],
181:         _mm_unpacklo_epi16(vacc_low_16bits, vacc_high_16bits));
182:       vacc_high[n] = _mm_add_epi32(vacc_high[n],
183:         _mm_unpackhi_epi16(vacc_low_16bits, vacc_high_16bits));
184:       // acc += a2 * b2;
185:       vacc_low_16bits = _mm_mullo_epi16(vxa2, vxb2);
186:       vacc_high_16bits = _mm_mulhi_epi16(vxa2, vxb2);
187:       vacc_low[n] = _mm_add_epi32(vacc_low[n],
188:         _mm_unpacklo_epi16(vacc_low_16bits, vacc_high_16bits));
189:       vacc_high[n] = _mm_add_epi32(vacc_high[n],
190:         _mm_unpackhi_epi16(vacc_low_16bits, vacc_high_16bits));
191:       // acc += a3 * b3;
192:       vacc_low_16bits = _mm_mullo_epi16(vxa3, vxb3);
193:       vacc_high_16bits = _mm_mulhi_epi16(vxa3, vxb3);
194:       vacc_low[n] = _mm_add_epi32(vacc_low[n],
195:         _mm_unpacklo_epi16(vacc_low_16bits, vacc_high_16bits));
196:       vacc_high[n] = _mm_add_epi32(vacc_high[n],
197:         _mm_unpackhi_epi16(vacc_low_16bits, vacc_high_16bits));
198:       // acc += a4 * b4;
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 199-225
```cpp
199:       vacc_low_16bits = _mm_mullo_epi16(vxa4, vxb4);
200:       vacc_high_16bits = _mm_mulhi_epi16(vxa4, vxb4);
201:       vacc_low[n] = _mm_add_epi32(vacc_low[n],
202:         _mm_unpacklo_epi16(vacc_low_16bits, vacc_high_16bits));
203:       vacc_high[n] = _mm_add_epi32(vacc_high[n],
204:         _mm_unpackhi_epi16(vacc_low_16bits, vacc_high_16bits));
205:       // acc += a5 * b5;
206:       vacc_low_16bits = _mm_mullo_epi16(vxa5, vxb5);
207:       vacc_high_16bits = _mm_mulhi_epi16(vxa5, vxb5);
208:       vacc_low[n] = _mm_add_epi32(vacc_low[n],
209:         _mm_unpacklo_epi16(vacc_low_16bits, vacc_high_16bits));
210:       vacc_high[n] = _mm_add_epi32(vacc_high[n],
211:         _mm_unpackhi_epi16(vacc_low_16bits, vacc_high_16bits));
212:       // acc += a6 * b6;
213:       vacc_low_16bits = _mm_mullo_epi16(vxa6, vxb6);
214:       vacc_high_16bits = _mm_mulhi_epi16(vxa6, vxb6);
215:       vacc_low[n] = _mm_add_epi32(vacc_low[n],
216:         _mm_unpacklo_epi16(vacc_low_16bits, vacc_high_16bits));
217:       vacc_high[n] = _mm_add_epi32(vacc_high[n],
218:         _mm_unpackhi_epi16(vacc_low_16bits, vacc_high_16bits));
219:       // acc += a7 * b7;
220:       vacc_low_16bits = _mm_mullo_epi16(vxa7, vxb7);
221:       vacc_high_16bits = _mm_mulhi_epi16(vxa7, vxb7);
222:       vacc_low[n] = _mm_add_epi32(vacc_low[n],
223:         _mm_unpacklo_epi16(vacc_low_16bits, vacc_high_16bits));
224:       vacc_high[n] = _mm_add_epi32(vacc_high[n],
225:         _mm_unpackhi_epi16(vacc_low_16bits, vacc_high_16bits));
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 227-249
```cpp
227:       // Now we have 1x8 m acculated 32 bit values in vacc_low[n](4) and vacc_high[n](4)
228:
229:       temp_packed_w = temp_packed_w + COL_BLOCK_SIZE * 2;
230:       temp_w_block_ids_ptr += 2;
231:       num_blocks -= 2;
232:     }
233:     if (num_blocks > 0) {
234:       // Load two 1x4 uint8 blocks 2 ints
235:       const uint8_t* b_ptr = temp_packed_w;
236:       const int16_t b_0 = (int16_t)((uint16_t)(b_ptr[0]));
237:       const int16_t b_1 = (int16_t)((uint16_t)(b_ptr[1]));
238:       const int16_t b_2 = (int16_t)((uint16_t)(b_ptr[2]));
239:       const int16_t b_3 = (int16_t)((uint16_t)(b_ptr[3]));
240:       // Now we will load 8kx1(broadcast 8) weight values
241:       const __m128i vxb0 = _mm_set1_epi16((b_0 - b_zero_point));
242:       const __m128i vxb1 = _mm_set1_epi16((b_1 - b_zero_point));
243:       const __m128i vxb2 = _mm_set1_epi16((b_2 - b_zero_point));
244:       const __m128i vxb3 = _mm_set1_epi16((b_3 - b_zero_point));
245:
246:       // Then load transformed weight blocks
247:       // 1. Load 4 1x8 registers = 4k x 8m
248:       // Thus have 4x8 (4k x 8m) activations a0, a1, a2, a3
249:       // Each a containing 8 m values.
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 251-272
```cpp
251:       // Load column id of the first 1x4 block
252:       int32_t col_block_id_0 = temp_w_block_ids_ptr[0];
253:       const __m128i va0 =
254:         _mm_loadl_epi64((const __m128i*) (a_packed +
255:             col_block_id_0 * PACKED_A_BLOCK_SIZE + MR * 0));
256:       const __m128i va1 =
257:         _mm_loadl_epi64((const __m128i*) (a_packed +
258:             col_block_id_0 * PACKED_A_BLOCK_SIZE + MR * 1));
259:       const __m128i va2 =
260:         _mm_loadl_epi64((const __m128i*) (a_packed +
261:             col_block_id_0 * PACKED_A_BLOCK_SIZE + MR * 2));
262:       const __m128i va3 =
263:         _mm_loadl_epi64((const __m128i*) (a_packed +
264:             col_block_id_0 * PACKED_A_BLOCK_SIZE + MR * 3));
265:       const __m128i vxa0 =
266:           sub_zero_point(_mm_unpacklo_epi8(va0, vzero), va_zero_point);
267:       const __m128i vxa1 =
268:           sub_zero_point(_mm_unpacklo_epi8(va1, vzero), va_zero_point);
269:       const __m128i vxa2 =
270:           sub_zero_point(_mm_unpacklo_epi8(va2, vzero), va_zero_point);
271:       const __m128i vxa3 =
272:           sub_zero_point(_mm_unpacklo_epi8(va3, vzero), va_zero_point);
```
- EN: Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段涉及量化数据处理或打包参数逻辑。

### Lines 274-301
```cpp
274:       // acc += a0 * b0;
275:       __m128i vacc_low_16bits = _mm_mullo_epi16(vxa0, vxb0);
276:       __m128i vacc_high_16bits = _mm_mulhi_epi16(vxa0, vxb0);
277:       vacc_low[n] = _mm_add_epi32(vacc_low[n],
278:         _mm_unpacklo_epi16(vacc_low_16bits, vacc_high_16bits));
279:       vacc_high[n] = _mm_add_epi32(vacc_high[n],
280:         _mm_unpackhi_epi16(vacc_low_16bits, vacc_high_16bits));
281:       // acc += a1 * b1;
282:       vacc_low_16bits = _mm_mullo_epi16(vxa1, vxb1);
283:       vacc_high_16bits = _mm_mulhi_epi16(vxa1, vxb1);
284:       vacc_low[n] = _mm_add_epi32(vacc_low[n],
285:         _mm_unpacklo_epi16(vacc_low_16bits, vacc_high_16bits));
286:       vacc_high[n] = _mm_add_epi32(vacc_high[n],
287:         _mm_unpackhi_epi16(vacc_low_16bits, vacc_high_16bits));
288:       // acc += a2 * b2;
289:       vacc_low_16bits = _mm_mullo_epi16(vxa2, vxb2);
290:       vacc_high_16bits = _mm_mulhi_epi16(vxa2, vxb2);
291:       vacc_low[n] = _mm_add_epi32(vacc_low[n],
292:         _mm_unpacklo_epi16(vacc_low_16bits, vacc_high_16bits));
293:       vacc_high[n] = _mm_add_epi32(vacc_high[n],
294:         _mm_unpackhi_epi16(vacc_low_16bits, vacc_high_16bits));
295:       // acc += a3 * b3;
296:       vacc_low_16bits = _mm_mullo_epi16(vxa3, vxb3);
297:       vacc_high_16bits = _mm_mulhi_epi16(vxa3, vxb3);
298:       vacc_low[n] = _mm_add_epi32(vacc_low[n],
299:         _mm_unpacklo_epi16(vacc_low_16bits, vacc_high_16bits));
300:       vacc_high[n] = _mm_add_epi32(vacc_high[n],
301:         _mm_unpackhi_epi16(vacc_low_16bits, vacc_high_16bits));
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 303-328
```cpp
303:       // Now we have 1x8 m acculated 32 bit values in vacc_low[n](4) and vacc_high[n](4)
304:     }
305:   }
306:
307:   __m128 vout[8];
308:   __m128 a_ps, b_ps, c_ps, d_ps, tmp0, tmp1, tmp2, tmp3;
309:
310:   // Transform low half of 4x8 result
311:   // That is 4x4 block (4n x 4m)
312:   // Convert to FP and transpose: 4m x 4n
313:   CONVERT_TO_FP_AND_TRANSPOSE(vacc_low[0],
314:                               vacc_low[1],
315:                               vacc_low[2],
316:                               vacc_low[3],
317:                               vout[0],
318:                               vout[1],
319:                               vout[2],
320:                               vout[3])
321:   CONVERT_TO_FP_AND_TRANSPOSE(vacc_high[0],
322:                               vacc_high[1],
323:                               vacc_high[2],
324:                               vacc_high[3],
325:                               vout[4],
326:                               vout[5],
327:                               vout[6],
328:                               vout[7])
```
- EN: The math and shape handling relate to convolution-style operators.
- CN: 这里的计算与形状处理与卷积类算子相关。

### Lines 330-348
```cpp
330:   vout[0] = _mm_mul_ps(vmultiplier, vout[0]);
331:   vout[1] = _mm_mul_ps(vmultiplier, vout[1]);
332:   vout[2] = _mm_mul_ps(vmultiplier, vout[2]);
333:   vout[3] = _mm_mul_ps(vmultiplier, vout[3]);
334:   vout[4] = _mm_mul_ps(vmultiplier, vout[4]);
335:   vout[5] = _mm_mul_ps(vmultiplier, vout[5]);
336:   vout[6] = _mm_mul_ps(vmultiplier, vout[6]);
337:   vout[7] = _mm_mul_ps(vmultiplier, vout[7]);
338:
339:   vout[0] = _mm_add_ps(vout[0], vbias);
340:   vout[1] = _mm_add_ps(vout[1], vbias);
341:   vout[2] = _mm_add_ps(vout[2], vbias);
342:   vout[3] = _mm_add_ps(vout[3], vbias);
343:   vout[4] = _mm_add_ps(vout[4], vbias);
344:   vout[5] = _mm_add_ps(vout[5], vbias);
345:   vout[6] = _mm_add_ps(vout[6], vbias);
346:   vout[7] = _mm_add_ps(vout[7], vbias);
347:
348:   float* c0 = c;
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 349-368
```cpp
349:   float* c1 = c0 + c_stride;
350:   if (mr < 2) {
351:     c1 = c0;
352:     vout[1] = vout[0];
353:   }
354:   float* c2 = c1 + c_stride;
355:   if (mr < 3) {
356:     c2 = c0;
357:     vout[2] = vout[0];
358:   }
359:   float* c3 = c2 + c_stride;
360:   if (mr < 4) {
361:     c3 = c0;
362:     vout[3] = vout[0];
363:   }
364:   float* c4 = c3 + c_stride;
365:   if (mr < 5) {
366:     c4 = c0;
367:     vout[4] = vout[0];
368:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 369-388
```cpp
369:   float* c5 = c4 + c_stride;
370:   if (mr < 6) {
371:     c5 = c0;
372:     vout[5] = vout[0];
373:   }
374:   float* c6 = c5 + c_stride;
375:   if (mr < 7) {
376:     c6 = c0;
377:     vout[6] = vout[0];
378:   }
379:   float* c7 = c6 + c_stride;
380:   if (mr < 8) {
381:     c7 = c0;
382:     vout[7] = vout[0];
383:   }
384:
385:   if (nr == 4) {
386:     _mm_storeu_ps(c0, vout[0]);
387:     _mm_storeu_ps(c1, vout[1]);
388:     _mm_storeu_ps(c2, vout[2]);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 389-408
```cpp
389:     _mm_storeu_ps(c3, vout[3]);
390:     _mm_storeu_ps(c4, vout[4]);
391:     _mm_storeu_ps(c5, vout[5]);
392:     _mm_storeu_ps(c6, vout[6]);
393:     _mm_storeu_ps(c7, vout[7]);
394:   } else {
395:     if (nr >= 2) {
396:       _mm_storel_pi((__m64*)c0, vout[0]);
397:       _mm_storel_pi((__m64*)c1, vout[1]);
398:       _mm_storel_pi((__m64*)c2, vout[2]);
399:       _mm_storel_pi((__m64*)c3, vout[3]);
400:       _mm_storel_pi((__m64*)c4, vout[4]);
401:       _mm_storel_pi((__m64*)c5, vout[5]);
402:       _mm_storel_pi((__m64*)c6, vout[6]);
403:       _mm_storel_pi((__m64*)c7, vout[7]);
404:
405:       nr -= 2;
406:
407:       c0 += 2;
408:       c1 += 2;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 409-428
```cpp
409:       c2 += 2;
410:       c3 += 2;
411:       c4 += 2;
412:       c5 += 2;
413:       c6 += 2;
414:       c7 += 2;
415:       vout[0] = _mm_shuffle_ps(vout[0], vout[0], _MM_SHUFFLE(2, 2, 2, 2));
416:       vout[1] = _mm_shuffle_ps(vout[1], vout[1], _MM_SHUFFLE(2, 2, 2, 2));
417:       vout[2] = _mm_shuffle_ps(vout[2], vout[2], _MM_SHUFFLE(2, 2, 2, 2));
418:       vout[3] = _mm_shuffle_ps(vout[3], vout[3], _MM_SHUFFLE(2, 2, 2, 2));
419:       vout[4] = _mm_shuffle_ps(vout[4], vout[4], _MM_SHUFFLE(2, 2, 2, 2));
420:       vout[5] = _mm_shuffle_ps(vout[5], vout[5], _MM_SHUFFLE(2, 2, 2, 2));
421:       vout[6] = _mm_shuffle_ps(vout[6], vout[6], _MM_SHUFFLE(2, 2, 2, 2));
422:       vout[7] = _mm_shuffle_ps(vout[7], vout[7], _MM_SHUFFLE(2, 2, 2, 2));
423:     }
424:     if (nr != 0) {
425:       *c0 = _mm_cvtss_f32(vout[0]);
426:       *c1 = _mm_cvtss_f32(vout[1]);
427:       *c2 = _mm_cvtss_f32(vout[2]);
428:       *c3 = _mm_cvtss_f32(vout[3]);
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 429-435
```cpp
429:       *c4 = _mm_cvtss_f32(vout[4]);
430:       *c5 = _mm_cvtss_f32(vout[5]);
431:       *c6 = _mm_cvtss_f32(vout[6]);
432:       *c7 = _mm_cvtss_f32(vout[7]);
433:     }
434:   }
435: }
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Namespace-scoped helper structure / 命名空间内辅助结构
- Operator-specific control flow / 算子特定控制流

## Dependencies / 依赖关系

- Standard or third-party headers / 标准库或第三方头文件: `immintrin.h`, `qnnpack/q8gemm_sparse.h`, `requantization/runtime-sse2.h`, `8x4c1x4-packed-sse2.h`
- Key helper symbols / 关键辅助符号: `qnnpack`
