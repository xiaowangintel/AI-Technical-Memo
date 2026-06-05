# UpSampleKernelNEONAntialias.h — Code Analysis / 代码分析

## Source / 来源
- File: `aten/src/ATen/native/cpu/UpSampleKernelNEONAntialias.h`
- Repository: `https://github.com/pytorch/pytorch.git` (`/root/xw/pytorch`, commit `27a4844d7fb`)
- Purpose (EN): This file implements or declares CPU resampling/interpolation kernels and coordinate mapping logic in the ATen native CPU path.
- 用途 (CN): 该文件在 ATen 原生CPU 路径中实现或声明了CPU 重采样/插值 kernel 与坐标映射逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
```cpp
 1: // NEON-optimized uint8 bilinear resize with antialiasing for aarch64.
 2: //
 3: // This is the NEON counterpart of UpSampleKernelAVXAntialias.h.
 4: // It only supports num_channels == 3 with channels-last input.
 5:
 6: #pragma once
 7: #if defined(__aarch64__)
 8:
 9: #include <ATen/core/Tensor.h>
10: #include <arm_neon.h>
11: #include <c10/util/irange.h>
12:
13: #ifndef AT_PER_OPERATOR_HEADERS
14: #include <ATen/Functions.h>
15: #else
16: #include <ATen/ops/empty.h>
17: #endif
```
- EN: This range pulls in required headers, including `ATen/core/Tensor.h`, `arm_neon.h`, `c10/util/irange.h`. It also sets up compile-time guards, feature switches, or platform-specific paths. Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 这一段引入了所需头文件，例如 `ATen/core/Tensor.h`, `arm_neon.h`, `c10/util/irange.h`。 它还建立了编译期开关、特性选择或平台相关分支。 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 19-28
```cpp
19: namespace {
20:
21: // Emulate SSE _mm_madd_epi16: multiply 8 int16 pairs element-wise, then add
22: // adjacent 32-bit products pairwise, producing 4 int32 results.
23: // [a0*b0+a1*b1, a2*b2+a3*b3, a4*b4+a5*b5, a6*b6+a7*b7]
24: inline int32x4_t neon_madd_s16(int16x8_t a, int16x8_t b) {
25:   int32x4_t prod_low = vmull_s16(vget_low_s16(a), vget_low_s16(b));
26:   int32x4_t prod_high = vmull_s16(vget_high_s16(a), vget_high_s16(b));
27:   return vpaddq_s32(prod_low, prod_high);
28: }
```
- EN: The code enters or documents the namespace scope used by ATen native CPU operators. The main symbol in this range is `neon_madd_s16`, which contributes directly to this file's operator logic. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。 这一段的主要符号是 `neon_madd_s16`，它们直接构成本文件的算子逻辑。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 30-40
```cpp
30: // Interpolation horizontal pass: compute x-axis interpolation outputs.
31: //
32: // Input data is interleaved RGB:
33: //   input = [r0, g0, b0, r1, g1, b1, r2, g2, b2, ...]
34: // Weights are float values rescaled to int16:
35: //   weights[i] = [w[i, 0], w[i, 1], ..., w[i, K-1]]
36: // For each output pixel i, we compute:
37: //   oR[i] = r[xmin[i]] * w[i,0] + ... + r[xmin[i]+K-1] * w[i,K-1]
38: //   oG[i] = g[xmin[i]] * w[i,0] + ... + g[xmin[i]+K-1] * w[i,K-1]
39: //   oB[i] = b[xmin[i]] * w[i,0] + ... + b[xmin[i]+K-1] * w[i,K-1]
40: //
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 41-53
```cpp
41: // Processes one row at a time.
42: void NeonResampleHorizontal(const at::Tensor& unpacked_output,
43:                             const at::Tensor& unpacked_input,
44:                             int ksize,
45:                             const std::vector<at::Tensor>& horiz_indices_weights,
46:                             unsigned int horiz_weights_precision) {
47:   const auto* kk = (const int16_t*)(horiz_indices_weights[3].const_data_ptr<double>());
48:
49:   auto xout = unpacked_output.size(2);
50:   auto yin = unpacked_output.size(1);
51:   auto xin = unpacked_input.size(2);
52:   const auto num_channels = unpacked_input.size(0);
53:   TORCH_INTERNAL_ASSERT(num_channels == 3);
```
- EN: The main symbol in this range is `NeonResampleHorizontal`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `NeonResampleHorizontal`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 55-67
```cpp
55:   const int64_t* idx_ptr_xmin = horiz_indices_weights[0].const_data_ptr<int64_t>();
56:   const int64_t* idx_ptr_size = horiz_indices_weights[1].const_data_ptr<int64_t>();
57:
58:   uint8_t* output_p = unpacked_output.data_ptr<uint8_t>();
59:   const uint8_t* input_p = unpacked_input.const_data_ptr<uint8_t>();
60:
61:   auto xout_stride = xout * num_channels;
62:   auto xin_stride = xin * num_channels;
63:
64:   for (int64_t yy = 0; yy < yin; yy++) {
65:     uint8_t* C10_RESTRICT lineOut = output_p + yy * xout_stride;
66:     const uint8_t* C10_RESTRICT lineIn = input_p + yy * xin_stride;
67:     const int32_t initial_val = 1 << (horiz_weights_precision - 1);
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 69-79
```cpp
69:     // Horizontal convolution for a single row.
70:     //
71:     // For each output pixel out_x, computes the weighted sum of ids_size input
72:     // pixels using the weight vector k[]. Uses vld3_u8 to load 8 interleaved RGB
73:     // pixels and deinterleave them into separate R, G, B vectors. This avoids
74:     // the shuffle masks needed by the AVX2 path.
75:     //
76:     // The weight vector is split as: ids_size = num_blocks_8 * 8 + remainder.
77:     // We process 8 weights in a vectorized loop, then handle remaining weights
78:     // in a scalar cleanup loop. Accumulators are kept per-channel in int32 to
79:     // avoid overflow.
```
- EN: The math and shape handling relate to convolution-style operators.
- CN: 这里的计算与形状处理与卷积类算子相关。

### Lines 80-91
```cpp
80:     for (int64_t out_x = 0; out_x < xout; out_x++) {
81:       // ids_min is a byte offset (pre-multiplied by num_channels)
82:       const int64_t ids_min = idx_ptr_xmin[out_x];
83:       const int64_t ids_size = idx_ptr_size[out_x];
84:       const int16_t* k = &kk[out_x * ksize];
85:       const uint8_t* lineIn_min = lineIn + ids_min;
86:
87:       int32x4_t acc_r = vdupq_n_s32(0);
88:       int32x4_t acc_g = vdupq_n_s32(0);
89:       int32x4_t acc_b = vdupq_n_s32(0);
90:
91:       int64_t i = 0;
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 93-104
```cpp
 93:       // Block 8: load 8 RGB pixels with vld3, deinterleave, widen to int16,
 94:       // multiply-accumulate with 8 weights per channel.
 95:       // Each vmlal_s16 accumulates 4 products; calling it on the low and high
 96:       // halves puts all 8 products into 4 int32 lanes, reduced later by
 97:       // vaddvq_s32.
 98:       for (; i + 8 <= ids_size; i += 8) {
 99:         uint8x8x3_t rgb = vld3_u8(lineIn_min + num_channels * i);
100:         int16x8_t weights = vld1q_s16(&k[i]);
101:
102:         int16x8_t r16 = vreinterpretq_s16_u16(vmovl_u8(rgb.val[0]));
103:         int16x8_t g16 = vreinterpretq_s16_u16(vmovl_u8(rgb.val[1]));
104:         int16x8_t b16 = vreinterpretq_s16_u16(vmovl_u8(rgb.val[2]));
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 106-116
```cpp
106:         acc_r = vmlal_s16(acc_r, vget_low_s16(r16), vget_low_s16(weights));
107:         acc_r = vmlal_s16(acc_r, vget_high_s16(r16), vget_high_s16(weights));
108:         acc_g = vmlal_s16(acc_g, vget_low_s16(g16), vget_low_s16(weights));
109:         acc_g = vmlal_s16(acc_g, vget_high_s16(g16), vget_high_s16(weights));
110:         acc_b = vmlal_s16(acc_b, vget_low_s16(b16), vget_low_s16(weights));
111:         acc_b = vmlal_s16(acc_b, vget_high_s16(b16), vget_high_s16(weights));
112:       }
113:
114:       // Block 4: handle 4 pixels that didn't fit in a block of 8
115:       // We also use vld3_u8 here, which still loads 8 pixels, but we only use
116:       // the lower half - so the computation is correct.
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 117-132
```cpp
117:       // On all rows except the last one, reading 8 pixels is safe (the tensors
118:       // are channels-last). But on the last row, we have to be careful not to
119:       // read past the buffer, hence the extra boundary check.
120:       const uint8_t* block_of_4_safe_load_end = input_p + yin * xin_stride - 24;
121:       for (; i + 4 <= ids_size && lineIn_min + num_channels * i <= block_of_4_safe_load_end; i += 4) {
122:         uint8x8x3_t rgb = vld3_u8(lineIn_min + num_channels * i);
123:         int16x4_t weights4 = vld1_s16(&k[i]);
124:
125:         int16x4_t r16 = vget_low_s16(vreinterpretq_s16_u16(vmovl_u8(rgb.val[0])));
126:         int16x4_t g16 = vget_low_s16(vreinterpretq_s16_u16(vmovl_u8(rgb.val[1])));
127:         int16x4_t b16 = vget_low_s16(vreinterpretq_s16_u16(vmovl_u8(rgb.val[2])));
128:
129:         acc_r = vmlal_s16(acc_r, r16, weights4);
130:         acc_g = vmlal_s16(acc_g, g16, weights4);
131:         acc_b = vmlal_s16(acc_b, b16, weights4);
132:       }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 134-146
```cpp
134:       // Horizontal reduction + rounding bias
135:       int32_t sum_r = vaddvq_s32(acc_r) + initial_val;
136:       int32_t sum_g = vaddvq_s32(acc_g) + initial_val;
137:       int32_t sum_b = vaddvq_s32(acc_b) + initial_val;
138:
139:       // Scalar cleanup for remaining pixels
140:       for (; i < ids_size; i++) {
141:         int16_t w = k[i];
142:         const uint8_t* p = lineIn_min + num_channels * i;
143:         sum_r += w * p[0];
144:         sum_g += w * p[1];
145:         sum_b += w * p[2];
146:       }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 148-158
```cpp
148:       // Right-shift and clamp to [0, 255]
149:       uint8_t* out = lineOut + num_channels * out_x;
150:       out[0] = static_cast<uint8_t>(std::clamp(sum_r >> horiz_weights_precision, 0, 255));
151:       out[1] = static_cast<uint8_t>(std::clamp(sum_g >> horiz_weights_precision, 0, 255));
152:       out[2] = static_cast<uint8_t>(std::clamp(sum_b >> horiz_weights_precision, 0, 255));
153:     }
154:   }
155: }
156:
157: // Interpolation vertical pass: compute y-axis interpolation outputs.
158: //
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 159-172
```cpp
159: // Input data is interleaved RGB:
160: //   input = [r0, g0, b0, r1, g1, b1, ...]
161: // For each output row yy, we compute a weighted sum of ids_size input rows:
162: //   oC[x] = C[ymin[yy] * xstride + x] * w[yy,0] + ... + C[(ymin[yy]+K-1) * xstride + x] * w[yy,K-1]
163: // for each channel C in {R, G, B} and each horizontal position x.
164: void NeonResampleVertical(const at::Tensor& unpacked_output,
165:                           const at::Tensor& unpacked_input,
166:                           int ksize,
167:                           const std::vector<at::Tensor>& vert_indices_weights,
168:                           unsigned int vert_weights_precision) {
169:   const auto* kk = (const int16_t*)(vert_indices_weights[3].const_data_ptr<double>());
170:
171:   const int64_t* idx_ptr_xmin = vert_indices_weights[0].const_data_ptr<int64_t>();
172:   const int64_t* idx_ptr_size = vert_indices_weights[1].const_data_ptr<int64_t>();
```
- EN: The main symbol in this range is `NeonResampleVertical`, which contributes directly to this file's operator logic. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 这一段的主要符号是 `NeonResampleVertical`，它们直接构成本文件的算子逻辑。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 174-187
```cpp
174:   uint8_t* output_p = unpacked_output.data_ptr<uint8_t>();
175:   const uint8_t* input_p = unpacked_input.const_data_ptr<uint8_t>();
176:
177:   auto xout = unpacked_output.size(2);
178:   auto yout = unpacked_output.size(1);
179:   const auto num_channels = unpacked_input.size(0);
180:   TORCH_INTERNAL_ASSERT(num_channels == unpacked_output.size(0));
181:
182:   const int64_t data_size = xout * num_channels;
183:
184:   for (const auto yy : c10::irange(yout)) {
185:     const auto* k = &kk[yy * ksize];
186:     auto ids_min = idx_ptr_xmin[yy];
187:     auto ids_size = idx_ptr_size[yy];
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation. Quantization-specific data handling or packed-parameter logic is active in this range.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。 这一段涉及量化数据处理或打包参数逻辑。

### Lines 189-199
```cpp
189:     uint8_t* C10_RESTRICT lineOut = output_p + yy * data_size;
190:     const int32_t initial_val = 1 << (vert_weights_precision - 1);
191:     const int32x4_t initial = vdupq_n_s32(initial_val);
192:
193:     // Vertical convolution for one output row.
194:     // Computes a weighted sum of ids_size input rows for all x positions.
195:     //
196:     // The data is treated as a flat byte array of size xout * num_channels.
197:     // We process 16 bytes at a time in the NEON path, then fall back to scalar
198:     // for the remainder. The vertical pass doesn't need channel deinterleaving
199:     // because the same weight applies to all channels at a given (x, y) position.
```
- EN: The math and shape handling relate to convolution-style operators.
- CN: 这里的计算与形状处理与卷积类算子相关。

### Lines 200-213
```cpp
200:     //
201:     // To process 2 weights at a time, we interleave pixels from two input rows
202:     // using vzipq_u8, then use neon_madd_s16 (which pairs adjacent int16 values,
203:     // multiplies, and adds) so that each pair [pixel_row_i, pixel_row_i+1] is
204:     // multiplied by [w_i, w_i+1] and summed in one operation.
205:     int64_t j = 0;
206:
207:     // Process 16 bytes at a time using 4 accumulators (sss0..sss3),
208:     // each holding 4 int32 partial sums.
209:     for (; j + 16 <= data_size; j += 16) {
210:       int32x4_t sss0 = initial;
211:       int32x4_t sss1 = initial;
212:       int32x4_t sss2 = initial;
213:       int32x4_t sss3 = initial;
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 215-226
```cpp
215:       const uint8_t* lineIn_min = input_p + j + ids_min;
216:
217:       // Process 2 weights at a time by interleaving rows.
218:       // For weights w0, w1 and pixel bytes from row_i and row_{i+1}:
219:       //   mmk = [w0, w1, w0, w1, w0, w1, w0, w1]
220:       //   vzipq interleaves: [r0_0, r1_0, r0_1, r1_1, ...] (from 2 rows)
221:       //   neon_madd_s16 computes: r0_0*w0 + r1_0*w1, r0_1*w0 + r1_1*w1, ...
222:       int64_t i = 0;
223:       for (; i + 1 < ids_size; i += 2) {
224:         int16_t w0 = k[i];
225:         int16_t w1 = k[i + 1];
226:         int16x8_t mmk = {w0, w1, w0, w1, w0, w1, w0, w1};
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 228-242
```cpp
228:         uint8x16_t src1 = vld1q_u8(lineIn_min + i * data_size);
229:         uint8x16_t src2 = vld1q_u8(lineIn_min + (i + 1) * data_size);
230:
231:         uint8x16x2_t interleaved = vzipq_u8(src1, src2);
232:
233:         int16x8_t pix0 = vreinterpretq_s16_u16(vmovl_u8(vget_low_u8(interleaved.val[0])));
234:         int16x8_t pix1 = vreinterpretq_s16_u16(vmovl_u8(vget_high_u8(interleaved.val[0])));
235:         int16x8_t pix2 = vreinterpretq_s16_u16(vmovl_u8(vget_low_u8(interleaved.val[1])));
236:         int16x8_t pix3 = vreinterpretq_s16_u16(vmovl_u8(vget_high_u8(interleaved.val[1])));
237:
238:         sss0 = vaddq_s32(sss0, neon_madd_s16(pix0, mmk));
239:         sss1 = vaddq_s32(sss1, neon_madd_s16(pix1, mmk));
240:         sss2 = vaddq_s32(sss2, neon_madd_s16(pix2, mmk));
241:         sss3 = vaddq_s32(sss3, neon_madd_s16(pix3, mmk));
242:       }
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 244-257
```cpp
244:       // Handle remaining single weight (when ids_size is odd)
245:       for (; i < ids_size; i++) {
246:         int16x8_t mmk = vdupq_n_s16(k[i]);
247:
248:         uint8x16_t src = vld1q_u8(lineIn_min + i * data_size);
249:
250:         int16x8_t pix_lo = vreinterpretq_s16_u16(vmovl_u8(vget_low_u8(src)));
251:         int16x8_t pix_hi = vreinterpretq_s16_u16(vmovl_u8(vget_high_u8(src)));
252:
253:         sss0 = vaddq_s32(sss0, vmull_s16(vget_low_s16(pix_lo), vget_low_s16(mmk)));
254:         sss1 = vaddq_s32(sss1, vmull_s16(vget_high_s16(pix_lo), vget_high_s16(mmk)));
255:         sss2 = vaddq_s32(sss2, vmull_s16(vget_low_s16(pix_hi), vget_low_s16(mmk)));
256:         sss3 = vaddq_s32(sss3, vmull_s16(vget_high_s16(pix_hi), vget_high_s16(mmk)));
257:       }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 259-269
```cpp
259:       // Right-shift accumulators by coefs_precision to convert fixed-point -> int
260:       int32x4_t shift = vdupq_n_s32(-static_cast<int32_t>(vert_weights_precision));
261:       sss0 = vshlq_s32(sss0, shift);
262:       sss1 = vshlq_s32(sss1, shift);
263:       sss2 = vshlq_s32(sss2, shift);
264:       sss3 = vshlq_s32(sss3, shift);
265:
266:       // Narrow int32 -> int16 (with saturation), then int16 -> uint8 (with
267:       // saturation), clamping to [0, 255]
268:       int16x8_t narrow_lo = vcombine_s16(vqmovn_s32(sss0), vqmovn_s32(sss1));
269:       int16x8_t narrow_hi = vcombine_s16(vqmovn_s32(sss2), vqmovn_s32(sss3));
```
- EN: The math and shape handling relate to convolution-style operators.
- CN: 这里的计算与形状处理与卷积类算子相关。

### Lines 271-283
```cpp
271:       // Store 16 output bytes
272:       vst1_u8(lineOut + j, vqmovun_s16(narrow_lo));
273:       vst1_u8(lineOut + j + 8, vqmovun_s16(narrow_hi));
274:     }
275:
276:     // Scalar fallback for remaining bytes
277:     for (; j < data_size; j++) {
278:       int32_t sss = initial_val;
279:       const uint8_t* lineIn_min = input_p + j + ids_min;
280:
281:       for (int64_t i = 0; i < ids_size; i++) {
282:         sss += k[i] * static_cast<int32_t>(lineIn_min[i * data_size]);
283:       }
```
- EN: Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 285-295
```cpp
285:       sss >>= vert_weights_precision;
286:       lineOut[j] = static_cast<uint8_t>(std::clamp(sss, 0, 255));
287:     }
288:   }
289: }
290:
291: // Main entry point for NEON-accelerated uint8 bilinear resize.
292: //
293: // Only supports num_channels == 3 with channels-last memory format.
294: // Mirrors upsample_avx_bilinear_bicubic_uint8 but uses NEON intrinsics and
295: // works directly on interleaved RGB data (no unpack/pack step needed).
```
- EN: This range continues the file's low-level CPU implementation details and operator glue code.
- CN: 这一段继续展开本文件的底层 CPU 实现细节和算子胶水代码。

### Lines 296-311
```cpp
296: //
297: // Weights are computed as int16 values by compute_index_ranges_int16_weights
298: // with align_i32=false (no int32 alignment padding needed, unlike AVX2 which
299: // processes pixels as 4-byte RGBA units).
300: template <typename scale_type, class F>
301: void upsample_neon_bilinear_bicubic_uint8(const at::Tensor& input_,
302:                                           const at::Tensor& output,
303:                                           bool align_corners,
304:                                           const scale_type& scales,
305:                                           bool antialias) {
306:   auto batch_size = input_.size(0);
307:   auto num_channels = input_.size(1);
308:   auto xin = input_.size(3);
309:   auto yin = input_.size(2);
310:   auto xout = output.size(3);
311:   auto yout = output.size(2);
```
- EN: The main symbol in this range is `false`, which contributes directly to this file's operator logic.
- CN: 这一段的主要符号是 `false`，它们直接构成本文件的算子逻辑。

### Lines 313-324
```cpp
313:   if (xin == xout && yin == yout) {
314:     output.copy_(input_);
315:     return;
316:   }
317:
318:   TORCH_INTERNAL_ASSERT(num_channels == 3);
319:   TORCH_INTERNAL_ASSERT(output.is_contiguous(at::MemoryFormat::ChannelsLast));
320:
321:   auto input = input_.contiguous(at::MemoryFormat::ChannelsLast);
322:
323:   auto need_horizontal = xout != xin;
324:   auto need_vertical = yout != yin;
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. The block also assembles or returns the result expected by the surrounding operator code.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 该代码块还会组装或返回外围算子代码所需的结果。

### Lines 326-336
```cpp
326:   int ksize_horiz, ksize_vert;
327:   std::vector<at::Tensor> horiz_indices_weights, vert_indices_weights;
328:   unsigned int horiz_weights_precision, vert_weights_precision;
329:
330:   if (need_horizontal) {
331:     int interp_dim = 3;
332:     std::tie(horiz_indices_weights, ksize_horiz, horiz_weights_precision) = F::compute_index_ranges_int16_weights(
333:         /*input_size=*/xin,
334:         /*output_size=*/xout,
335:         /*stride=*/num_channels,
336:         /*ndims=*/4,
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 337-348
```cpp
337:         /*reshape_dim=*/interp_dim,
338:         /*align_corners=*/align_corners,
339:         /*opt_scale=*/scales[interp_dim - 2],
340:         /*antialias=*/antialias,
341:         /*align_i32=*/false);
342:   }
343:
344:   if (need_vertical) {
345:     int interp_dim = 2;
346:     std::tie(vert_indices_weights, ksize_vert, vert_weights_precision) = F::compute_index_ranges_int16_weights(
347:         /*input_size=*/yin,
348:         /*output_size=*/yout,
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 349-361
```cpp
349:         /*stride=*/num_channels * xout,
350:         /*ndims=*/4,
351:         /*reshape_dim=*/interp_dim,
352:         /*align_corners=*/align_corners,
353:         /*opt_scale=*/scales[interp_dim - 2],
354:         /*antialias=*/antialias,
355:         /*align_i32=*/false);
356:   }
357:
358:   at::Tensor buffer_horiz;
359:   if (need_horizontal && need_vertical) {
360:     buffer_horiz = at::empty({num_channels, yin, xout}, input.options());
361:   }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。

### Lines 363-377
```cpp
363:   for (const auto i : c10::irange(batch_size)) {
364:     at::Tensor input_slice = input[i];
365:
366:     if (need_horizontal) {
367:       at::Tensor horiz_output = need_vertical ? buffer_horiz : output[i];
368:       NeonResampleHorizontal(horiz_output, input_slice, ksize_horiz, horiz_indices_weights, horiz_weights_precision);
369:       if (need_vertical) {
370:         input_slice = horiz_output;
371:       }
372:     }
373:     if (need_vertical) {
374:       NeonResampleVertical(output[i], input_slice, ksize_vert, vert_indices_weights, vert_weights_precision);
375:     }
376:   }
377: }
```
- EN: Conditional branches handle alternate code paths, edge cases, or platform/dtype distinctions. Looping logic performs the repeated per-element, per-channel, or per-kernel computation.
- CN: 条件分支用于处理不同代码路径、边界情况或平台/数据类型差异。 循环逻辑承担重复的逐元素、逐通道或逐 kernel 计算。

### Lines 379-381
```cpp
379: } // anonymous namespace
380:
381: #endif // __aarch64__
```
- EN: It also sets up compile-time guards, feature switches, or platform-specific paths. The code enters or documents the namespace scope used by ATen native CPU operators.
- CN: 它还建立了编译期开关、特性选择或平台相关分支。 代码在这里进入或说明了 ATen 原生 CPU 算子的命名空间作用域。

## Key Concepts / 关键概念

- Quantized arithmetic and packed parameters / 量化算术与打包参数
- Convolution shape/kernel handling / 卷积形状与 kernel 处理
- Low-level memory access / 底层内存访问
- Scalar and dtype abstractions / 标量与数据类型抽象
- Shape/container bookkeeping / 形状与容器管理

## Dependencies / 依赖关系

- ATen headers / ATen 头文件: `ATen/core/Tensor.h`, `ATen/Functions.h`, `ATen/ops/empty.h`
- c10 headers / c10 头文件: `c10/util/irange.h`
- Standard or third-party headers / 标准库或第三方头文件: `arm_neon.h`
- Key helper symbols / 关键辅助符号: `Scalar`
